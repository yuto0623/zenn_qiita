---
title: Next.jsとConvexで作る極限までシンプルなリアルタイムホワイトボード
publication_name: aun_phonogram
private: false
tags:
  - Convex
  - Nextjs
  - TypeScript
  - React
updated_at: '2026-04-27T07:39:36.338Z'
id: null
organization_url_name: null
slide: false
---

## はじめに

https://zenn.dev/aun_phonogram/articles/convex-basics

[前回の記事](https://zenn.dev/aun_phonogram/articles/convex-basics)でConvexの基本機能をひととおり触ったので、次は実際に何か作ってみようということで、**リアルタイム共有できるホワイトボードアプリ**を作ってみました。

ただし普通に作ると機能が膨らみがちなので、今回は「**極限までシンプル**」を目標にしています。

- 部屋なし・認証なし・1枚の共有ボード
- ペンの色も太さも固定
- 機能は「描く」「全消し」だけ

それでも、Convexのおかげでリアルタイム同期は何も書かずに動きます。

## どこまで作るか

仕様をひとことで言うと、こうです。

- ブラウザでマウス／タッチで線を描ける
- 描いた線は別のブラウザにも自動で反映される
- 「全消し」ボタンで全員のキャンバスがクリアされる

**割り切りポイント**として、「描いている途中の線」は他ユーザーには表示しません。`pointerup`(指やペンを離した瞬間)に1ストロークまとめてConvexに送ります。これで通信回数が劇的に減って実装もシンプルになります。

「ペンの動きそのものをリアルタイム共有」したい場合は点ごとにMutationを飛ばす設計になりますが、今回はそこまでやりません。

## セットアップ

### Next.jsプロジェクトを作る

まだ無ければ、まずNext.jsのプロジェクトを用意します。

```bash
npx create-next-app@latest convex-whiteboard
cd convex-whiteboard
```

App Router・TypeScript・ESLintは有効でOKです(以降のコードはApp Router前提です)。

### Convexを入れる

```bash
npm install convex
npx convex dev
```

初回は Convex にログインを求められて、プロジェクトを新規作成する流れになります。完了すると、

- `convex/` ディレクトリ(ここにサーバー関数を書く)
- `.env.local`(以下の3つの環境変数が自動で書き込まれる)
  - `CONVEX_DEPLOYMENT`
  - `NEXT_PUBLIC_CONVEX_URL`
  - `NEXT_PUBLIC_CONVEX_SITE_URL`

が生成されます。`npx convex dev` はこのまま起動しっぱなしにしておきます(`convex/` 以下を保存するたびに自動デプロイしてくれます)。

### ConvexProviderでラップする

クライアントから Convex を叩くために、Provider を作ります。

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

`app/layout.tsx` でこの Provider を噛ませます。

```tsx
// app/layout.tsx
import { ConvexClientProvider } from "./ConvexClientProvider";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ja">
      <body>
        <ConvexClientProvider>{children}</ConvexClientProvider>
      </body>
    </html>
  );
}
```

ここまでで、クライアントから `useQuery` / `useMutation` が使える状態になりました。

## Schema

`strokes`(ストローク)テーブルを1つだけ作ります。1ストローク = 点の配列、です。

```ts
// convex/schema.ts
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  strokes: defineTable({
    points: v.array(v.object({ x: v.number(), y: v.number() })),
  }),
});
```

## サーバー関数

読み取り(`list`)・追加(`add`)・全消し(`clear`)の3つだけ。

```ts
// convex/strokes.ts
import { mutation, query } from "./_generated/server";
import { v } from "convex/values";

export const list = query({
  args: {},
  handler: async (ctx) => {
    return await ctx.db.query("strokes").collect();
  },
});

export const add = mutation({
  args: {
    points: v.array(v.object({ x: v.number(), y: v.number() })),
  },
  handler: async (ctx, { points }) => {
    await ctx.db.insert("strokes", { points });
  },
});

export const clear = mutation({
  args: {},
  handler: async (ctx) => {
    const all = await ctx.db.query("strokes").collect();
    for (const s of all) {
      await ctx.db.delete(s._id);
    }
  },
});
```

`clear` は全件取得 → ループで削除しているだけです。Convexは1つのMutation内が自動でトランザクショナルなので、途中で失敗したら全部巻き戻ります。

## クライアント

ここがメインです。といっても150行くらいです。

```tsx
// app/Whiteboard.tsx
"use client";

import { useEffect, useRef, useState } from "react";
import { useQuery, useMutation } from "convex/react";
import { api } from "@/convex/_generated/api";

type Point = { x: number; y: number };

export function Whiteboard() {
  const strokes = useQuery(api.strokes.list);
  const addStroke = useMutation(api.strokes.add);
  const clearStrokes = useMutation(api.strokes.clear);

  const canvasRef = useRef<HTMLCanvasElement>(null);
  const [drawing, setDrawing] = useState<Point[] | null>(null);

  // 確定済みストローク + 描画中ストロークを毎回まとめて再描画
  useEffect(() => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    const ctx = canvas.getContext("2d");
    if (!ctx) return;

    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.lineWidth = 2;
    ctx.lineCap = "round";
    ctx.strokeStyle = "#222";

    const drawStroke = (points: Point[]) => {
      if (points.length < 2) return;
      ctx.beginPath();
      ctx.moveTo(points[0].x, points[0].y);
      for (let i = 1; i < points.length; i++) {
        ctx.lineTo(points[i].x, points[i].y);
      }
      ctx.stroke();
    };

    strokes?.forEach((s) => drawStroke(s.points));
    if (drawing) drawStroke(drawing);
  }, [strokes, drawing]);

  const getPoint = (e: React.PointerEvent<HTMLCanvasElement>): Point => {
    const rect = e.currentTarget.getBoundingClientRect();
    return { x: e.clientX - rect.left, y: e.clientY - rect.top };
  };

  return (
    <div>
      <canvas
        ref={canvasRef}
        width={800}
        height={500}
        style={{ border: "1px solid #ccc", touchAction: "none" }}
        onPointerDown={(e) => {
          e.currentTarget.setPointerCapture(e.pointerId);
          setDrawing([getPoint(e)]);
        }}
        onPointerMove={(e) => {
          if (!drawing) return;
          setDrawing([...drawing, getPoint(e)]);
        }}
        onPointerUp={async () => {
          if (drawing && drawing.length > 1) {
            await addStroke({ points: drawing });
          }
          setDrawing(null);
        }}
      />
      <div style={{ marginTop: 8 }}>
        <button onClick={() => clearStrokes()}>全消し</button>
      </div>
    </div>
  );
}
```

ポイントを順に説明します。

### 1. 描画中はローカルstateに溜める

`onPointerDown` で `drawing` ステートを `[最初の点]` で初期化、`onPointerMove` で点を足していきます。この間、Convexには何も送りません。

### 2. 指を離した瞬間に1ストロークごとMutation

`onPointerUp` で `addStroke` を呼びます。ここではじめてConvexにデータが渡るので、この瞬間に他ユーザーのキャンバスにも線が現れます。

### 3. `useQuery` の結果が変われば自動で再描画

`useEffect` の依存配列に `strokes` が入っているので、誰かが線を引くたびにこのコンポーネントは再レンダリングされ、キャンバスが描き直されます。**WebSocketの配線も差分計算もこちらでは一切書いていません。**

### 4. `touchAction: "none"`

スマホでスクロールしないように指定しておきます。これがないとスマホで線が引けません。

## ページに組み込む

あとは適当なページから呼ぶだけです。

```tsx
// app/page.tsx
import { Whiteboard } from "./Whiteboard";

export default function Page() {
  return (
    <main style={{ padding: 24 }}>
      <h1>Realtime Whiteboard</h1>
      <Whiteboard />
    </main>
  );
}
```

## 動作確認

`npx convex dev` と `npm run dev` を走らせて、ブラウザを2つ並べて開きます。片方で線を引くと、もう片方にもパッと線が現れる、はずです。

![ホワイトボードに棒人間を描いた完成イメージ](https://raw.githubusercontent.com/yuto0623/zenn_qiita/main/images/convex-whiteboard/image2.png)

ストロークを引き終わってから反映されるので、若干「ピッ」と出てくる感じになりますが、これが今回の割り切りです。

### ダークモードで線が見えないとき

OSをダークモードにしていると、`create-next-app` のデフォルトの `app/globals.css` がそれに追従して背景が暗くなり、`strokeStyle: "#222"` のペンが背景に溶けてほぼ見えなくなります(下のような状態になります)。

![ダークモードでペンの線が背景に溶けてしまっている状態](https://raw.githubusercontent.com/yuto0623/zenn_qiita/main/images/convex-whiteboard/image.png)

手っ取り早く回避するなら、`app/globals.css` の `prefers-color-scheme: dark` ブロックをコメントアウトしてしまうのが楽です。

```css
/* app/globals.css */
/* @media (prefers-color-scheme: dark) {
  :root {
    --background: #0a0a0a;
    --foreground: #ededed;
  }
} */
```

これでライトモード固定になり、ペンの線がちゃんと見えるようになります。

## まとめ

- スキーマ1テーブル + Query/Mutation 3つ + クライアント1ファイル、これだけでリアルタイム共有のホワイトボードができる
- 「描画中の点をリアルタイム共有しない」と決めるだけで通信量も実装量も激減する
- WebSocketの配線・購読管理は何も書いていないのに動く

ここから拡張するなら、ペンの色や太さの選択、ボード(部屋)のID化、Clerk認証で「自分のボード一覧」、`undo` あたりが自然な次の一歩だと思います。とはいえ、**このちょっとのコードで動く**というのが、Convexを触っていて一番気持ちよかったポイントでした。
