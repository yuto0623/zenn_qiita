---
title: "Remotionのスキルを使って動画作成してみた"
emoji: "🎬"
type: "tech"
topics: [Remotion, React, ClaudeCode, 動画作成]
published: false
---

## はじめに

Claude Codeには「スキル」という仕組みがあり、特定のフレームワークやツールのベストプラクティスを学習させることができます。今回は、Remotion公式が提供しているスキルを使って、以前書いた「[Claude Codeで1日でアプリを作ってApp Store公開](https://zenn.dev/aun_phonogram/articles/claude-code-one-day-app)」の記事を動画にしてみました。

https://zenn.dev/aun_phonogram/articles/claude-code-one-day-app

結論から言うと、**「この記事を動画にして」と伝えるだけで42秒の紹介動画が完成**しました。

https://youtu.be/NGItbY0dPNE

## Remotionとは

Remotionは、Reactを使ってプログラマティックに動画を作成できるフレームワーク。

- React コンポーネントで動画を作成
- TypeScript対応
- プレビュー機能付き
- MP4などに書き出し可能

## セットアップ

### Claude Codeにスキルを追加

まず、Claude CodeにRemotionのベストプラクティスを学ばせるためにスキルを追加。

```bash
npx skills add remotion-dev/skills
```

これで、Claude CodeがRemotionの正しい書き方を把握した状態で開発をサポートしてくれる。

### プロジェクト作成

```bash
# Remotionプロジェクトの作成
npx create-video@latest
```

## 実際にやったこと

### ブログ記事から動画を生成

「この記事をRemotionのスキルを使って動画にしたい」とClaude Codeに伝えたら、**それだけで動画ができた**。

- 既存のブログ記事（[Claude Codeで1日でアプリを作ってApp Store公開](https://zenn.dev/aun_phonogram/articles/claude-code-one-day-app)）を渡した
- Claude Codeがスキルの知識を使ってRemotionのコードを自動生成
- プレビューで確認 → そのまま動画として出力

## 使用した機能

Claude Codeに生成されたコードについて聞いてみたところ、以下のような構成になっていました。

### コンポーネント構成

6つのシーンに分けて構成されています：

| シーン | 内容 | 長さ |
|---|---|---|
| TitleScene | オープニング | 5秒 |
| AppIntroScene | アプリ紹介 | 6秒 |
| PhaseScene ×4 | 開発の4フェーズ説明 | 各5秒 |
| ClaudeCodeScene | Claude Codeの活躍場面 | 6秒 |
| SummaryScene | まとめ | 8秒 |
| EndingScene | 締めくくり | 5秒 |

### アニメーション

- **`spring()`**: バネのような自然な動きでフェードイン・スケールイン
- **`interpolate()`**: フレームの進行度を0〜1に変換して滑らかな変化
- **カスケード効果**: 要素を順番に表示するためのフレームオフセット

```tsx
// springの使用例
const scale = spring({
  frame,
  fps,
  config: { damping: 100 },
});

// interpolateの使用例
const opacity = interpolate(frame, [0, 30], [0, 1]);
```

### ビジュアル

- グラデーション背景
- グラスモーフィズム（`backdropFilter: blur`）
- 絵文字を効果的に使ったアイコン表現
- Claudeのブランドカラー（`#D97757`）をアクセントに使用

## 学んだこと

### スキルの威力

Remotionのスキルを入れた状態でClaude Codeに動画作成を依頼すると：

- Remotionのベストプラクティスに沿ったコードが生成される
- `spring()`や`interpolate()`などのアニメーション関数が適切に使われる
- シーンの分割やSequenceの使い方も正しく理解している

### Remotion自体の学び

- **フレームベースの考え方**: 秒ではなくフレーム（30fpsなら1秒=30フレーム）で考える
- **Reactの知識がそのまま活きる**: コンポーネント分割、propsの受け渡しなど
- **プレビューが便利**: `npm start`でリアルタイムプレビューしながら調整できる

## まとめ

Claude Codeのスキル機能 + Remotionの組み合わせで、**ブログ記事から動画を自動生成**できました。

動画編集ソフトを使わずに、テキストベースで動画を作れるのはエンジニアにとって嬉しいポイント。しかもReactの知識がそのまま活かせるので、学習コストも低いです。

プロダクトの紹介動画やデモ動画を作りたいときに、ぜひ試してみてください。

## リンク

**動画作成に使った記事**

https://zenn.dev/aun_phonogram/articles/claude-code-one-day-app

**以前の記事で作ったアプリ**

https://apps.apple.com/jp/app/%E6%84%9F%E6%83%85%E3%81%A0%E3%81%91%E3%82%92%E8%A8%98%E9%8C%B2%E3%81%99%E3%82%8B%E6%97%A5%E8%A8%98%E3%82%A2%E3%83%97%E3%83%AA/id6757698845

