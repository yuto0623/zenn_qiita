---
title: "Convexの基本機能まとめ  TypeScriptで書けるリアルタイムBaaS"
emoji: "🚀"
type: "tech"
topics: [Convex, Nextjs, TypeScript, BaaS]
published: true
publication_name: "aun_phonogram"
---

## はじめに

最近、リアルタイム同期を使ったWebアプリを作るために調べていて、ConvexというBaaSを知りました。TypeScriptで書けて、フロントまで型が通る設計がすごく良さそうだったので、実際に触ってみて基本機能をまとめました。

## Convexとは

ざっくり言うと「TypeScriptで書ける、リアルタイム同期がデフォルトのBaaS」です。

- **DBからフロントまで型が通る**：スキーマ定義がそのままクライアントの型になる
- **リアルタイム同期がデフォルト**：`useQuery` で取得したデータは自動で再購読される
- **サーバー関数をTypeScriptで書く**：`convex/` ディレクトリに関数を置くだけでデプロイされる
- **バックエンドの部品が一通り揃っている**：DB・認証・ファイル・Cron・Webhook

FirebaseやSupabaseと似た立ち位置ですが、**TypeScript前提で設計されている**のがいちばんの特徴です。

## セットアップ

Next.jsプロジェクトにConvexを入れるには、以下のコマンドを実行します。

```bash
npm install convex
npx convex dev
```

初回は Convex にログインして、プロジェクトを作成する流れになります。`convex/` ディレクトリと `.env.local` が自動生成されます。

クライアント側は `ConvexProvider` でラップします。

```tsx
// app/ConvexClientProvider.tsx
"use client";

import { ConvexProvider, ConvexReactClient } from "convex/react";
import { ReactNode } from "react";

const convex = new ConvexReactClient(process.env.NEXT_PUBLIC_CONVEX_URL!);

export function ConvexClientProvider({ children }: { children: ReactNode }) {
  return <ConvexProvider client={convex}>{children}</ConvexProvider>;
}
```

`app/layout.tsx` でこのProviderを噛ませれば準備完了です。

## 基本機能の紹介

ここからがメインです。Convexの基本機能を順に見ていきます。

### Schema（スキーマ定義）

`convex/schema.ts` にテーブル定義を書きます。ここで定義した型が、そのままサーバー関数やクライアントの型として使われます。

```ts
// convex/schema.ts
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  tasks: defineTable({
    text: v.string(),
    isCompleted: v.boolean(),
    userId: v.string(),
  }).index("by_user", ["userId"]),
});
```

`index()` を定義しておくと、クエリでその順序・絞り込みを効率的に使えるようになります。

### Query（読み取り関数）

`query` は**リアルタイム購読される読み取り専用関数**です。データが変わると、`useQuery` で購読しているクライアントに自動でプッシュされます。

```ts
// convex/tasks.ts
import { query } from "./_generated/server";
import { v } from "convex/values";

export const list = query({
  args: { userId: v.string() },
  handler: async (ctx, { userId }) => {
    return await ctx.db
      .query("tasks")
      .withIndex("by_user", (q) => q.eq("userId", userId))
      .collect();
  },
});
```

クライアント側はこれだけです。

```tsx
"use client";
import { useQuery } from "convex/react";
import { api } from "@/convex/_generated/api";

export function TaskList({ userId }: { userId: string }) {
  const tasks = useQuery(api.tasks.list, { userId });
  if (tasks === undefined) return <div>Loading...</div>;
  return <ul>{tasks.map((t) => <li key={t._id}>{t.text}</li>)}</ul>;
}
```

WebSocketで繋がっているので、他のユーザーが `tasks` テーブルを更新すれば、このコンポーネントは再レンダリングされます。**自分でリアルタイムの配線を一切書かなくていい**のがConvexの強みです。

### Mutation（書き込み関数）

`mutation` は書き込み用の関数。**1つのMutation内の処理は自動でトランザクショナル**に実行されます。

```ts
// convex/tasks.ts
import { mutation } from "./_generated/server";
import { v } from "convex/values";

export const add = mutation({
  args: { text: v.string(), userId: v.string() },
  handler: async (ctx, { text, userId }) => {
    return await ctx.db.insert("tasks", {
      text,
      isCompleted: false,
      userId,
    });
  },
});
```

クライアント側は `useMutation` で呼び出します。

```tsx
const addTask = useMutation(api.tasks.add);
await addTask({ text: "Convexの記事を書く", userId });
```

Mutationが走ると、そのテーブルを購読している全Queryが自動で再評価されます。

### Action（外部API呼び出し）

Mutation/Queryは「Convex内のDBだけを触る決定的な関数」として設計されているので、外部APIを叩く処理は `action` に分けます。

```ts
// convex/ai.ts
import { action } from "./_generated/server";
import { v } from "convex/values";

export const summarize = action({
  args: { text: v.string() },
  handler: async (ctx, { text }) => {
    const res = await fetch("https://api.openai.com/v1/chat/completions", {
      method: "POST",
      headers: { Authorization: `Bearer ${process.env.OPENAI_API_KEY}` },
      body: JSON.stringify({ /* ... */ }),
    });
    const data = await res.json();
    // DB書き込みはMutation経由で
    await ctx.runMutation(api.tasks.saveSummary, { summary: data.choices[0].message.content });
  },
});
```

ポイントは、**ActionからDBを触りたいときは `ctx.runMutation` / `ctx.runQuery` を経由する**こと。Action自体はDBに直接アクセスできない代わりに、外部APIなどの非決定的な処理を安全に扱えます。

### File Storage（ファイルアップロード）

ファイルアップロードも組み込みです。流れは「アップロード用URLを発行 → クライアントがそこにPOST → storageIdをDBに保存」の3ステップ。

```ts
// convex/files.ts
import { mutation } from "./_generated/server";

export const generateUploadUrl = mutation(async (ctx) => {
  return await ctx.storage.generateUploadUrl();
});
```

クライアント側:

```tsx
const generateUploadUrl = useMutation(api.files.generateUploadUrl);

async function upload(file: File) {
  const url = await generateUploadUrl();
  const result = await fetch(url, { method: "POST", body: file });
  const { storageId } = await result.json();
  // storageIdをDBに保存する
}
```

保存したファイルは `ctx.storage.getUrl(storageId)` でダウンロードURLを取得できます。

### Auth（認証）

認証はClerk・Auth0などの外部IDプロバイダと連携する設計です。ClerkとConvexは公式で統合されていて、いちばん楽です。

```ts
// convex/auth.config.ts
export default {
  providers: [
    {
      domain: process.env.CLERK_JWT_ISSUER_DOMAIN,
      applicationID: "convex",
    },
  ],
};
```

サーバー関数側では `ctx.auth.getUserIdentity()` で認証済みユーザー情報が取れます。

```ts
export const myTasks = query({
  args: {},
  handler: async (ctx) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error("Unauthorized");
    return await ctx.db
      .query("tasks")
      .withIndex("by_user", (q) => q.eq("userId", identity.subject))
      .collect();
  },
});
```

クライアント側は `ConvexProviderWithClerk` に差し替えるだけで、認証トークンの受け渡しは自動です。

### Scheduled Functions / Cron（スケジューラ）

「1時間後にこの関数を実行」「毎日朝9時にこれを実行」といった定期・遅延実行も組み込み機能です。

```ts
// convex/crons.ts
import { cronJobs } from "convex/server";
import { internal } from "./_generated/api";

const crons = cronJobs();

crons.daily(
  "daily cleanup",
  { hourUTC: 0, minuteUTC: 0 },
  internal.tasks.cleanup,
);

export default crons;
```

遅延実行はMutation/Actionの中から呼び出せます。

```ts
await ctx.scheduler.runAfter(60 * 1000, internal.tasks.notify, { taskId });
```

外部のCronサービスや別ワーカーを用意する必要がないのは地味に便利です。

### HTTP Actions（Webhook受け口）

外部サービスからのWebhookを受けたいときは `httpAction` を使います。

```ts
// convex/http.ts
import { httpRouter } from "convex/server";
import { httpAction } from "./_generated/server";

const http = httpRouter();

http.route({
  path: "/stripe-webhook",
  method: "POST",
  handler: httpAction(async (ctx, request) => {
    const payload = await request.json();
    // 署名検証してDBに反映
    await ctx.runMutation(internal.payments.handleWebhook, { payload });
    return new Response(null, { status: 200 });
  }),
});

export default http;
```

デプロイするとHTTPSのエンドポイントが払い出されるので、そのURLをStripeなどに登録すればOKです。

## まとめ

Convexの基本機能をひととおり紹介しました。

- **Schema / Query / Mutation / Action** の4つが中心
- **File / Auth / Scheduler / HTTP Action** で周辺も揃う
- TypeScriptで全部書ける & フロントまで型が通る
- リアルタイム同期がデフォルトで動く

「Next.jsアプリのバックエンドでやりたいこと」がだいたい1箇所で完結するのがすごく楽でした。リアルタイム機能も、自分で書かなくていいのは大きいです。

次はこのリアルタイム同期を使って、複数ブラウザで同じキャンバスを共有できる[シンプルなホワイトボードアプリ](https://zenn.dev/aun_phonogram/articles/convex-whiteboard)を作ってみます。

https://zenn.dev/aun_phonogram/articles/convex-whiteboard