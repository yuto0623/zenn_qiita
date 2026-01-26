---
title: Claude Codeで1日でアプリを作ってApp Store公開
tags:
  - AppStore
  - reactnative
  - 個人開発
  - expo
  - ClaudeCode
private: false
updated_at: '2026-01-26T22:11:41+09:00'
id: ba709fd35f1984402c93
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

「アプリを作ってストアに公開する」という一連の流れを勉強したくて、Claude Codeを使って1日でアプリを作り、App Storeに公開しました。

結論から言うと、**Claude Codeを使えばアプリ開発がびっくりするほど簡単**でした。設計書を渡せば一発で実装、プライバシーポリシーも作成、エラーが出ても即座に解決。ほとんどの作業をAIに任せられました。

作ったのは「**感情だけを記録する日記アプリ**」という、文章を一切書かない感情記録アプリです。

https://github.com/yuto0623/wordless-diary
https://apps.apple.com/jp/app/%E6%84%9F%E6%83%85%E3%81%A0%E3%81%91%E3%82%92%E8%A8%98%E9%8C%B2%E3%81%99%E3%82%8B%E6%97%A5%E8%A8%98%E3%82%A2%E3%83%97%E3%83%AA/id6757698845

## 作ったアプリ：感情だけを記録する日記アプリ

コンセプトは「**文章を書かせない日記**」。ChatGPTに「1日で作れるシンプルなアプリのアイデア」を相談して決めました。

- 1日1回、感情を選ぶだけ
- 理由は記録しない
- 過去は修正できない
- 感情の推移だけが残る

シンプルなアプリにしたのは、**ストア公開までの流れを学ぶことが目的**だったからです。機能を絞ることで、申請に必要な作業に集中できました。

![感情選択画面](https://raw.githubusercontent.com/yuto0623/zenn_qiita/main/images/claude-code-one-day-app/img1.png)
*Today画面：6つの感情から今日の気持ちを選ぶだけ*

![振り返り画面](https://raw.githubusercontent.com/yuto0623/zenn_qiita/main/images/claude-code-one-day-app/img2.png)
*振り返り画面：カレンダーで感情の推移を確認できる*

### 技術スタック

- **Expo (React Native)** - クロスプラットフォーム開発
- **TypeScript** - 型安全な開発
- **expo-sqlite** - ローカルデータベース
- **EAS Build / Submit** - ビルドとストア申請

バックエンドなし、アカウント機能なし、完全ローカル完結の構成です。

## 開発からストア公開までの流れ

### Phase 1: 設計

まずClaude Codeと一緒に設計書を作成しました。

```markdown
# Wordless-Diary 設計書（MVP）

## コンセプト
文章を一切書かせない。1日1回、感情を選ぶだけのミニマル日記アプリ。

## 画面構成
- Today画面：感情選択UI
- 振り返り画面：カレンダー表示

## データ設計
- SQLiteでローカル保存
- 1日1回の記録制約
```

最初に設計書を作っておくことで、Claude Codeへの指示が明確になり、実装がスムーズに進みました。

### Phase 2: 実装

#### プロジェクトセットアップ

```bash
npx create-expo-app wordless-diary
cd wordless-diary
npx expo install expo-sqlite expo-haptics
```

#### アプリの実装

Claude Codeに以下のように指示しました。

```
この設計書を元にReact Nativeで実装して
```

これだけで、一発でアプリ全体を実装してくれました。

- 感情選択のグリッドUI
- カレンダー表示
- SQLiteでのデータ保存

全部自動で生成されます。自分でコードを書く必要はありませんでした。

### Phase 3: ストア公開準備

ここからがストア公開の学びポイントです。

#### 1. app.jsonの設定

[app.json](https://docs.expo.dev/versions/latest/config/app/)はExpoプロジェクトの設定ファイルです。アプリ名、バージョン、アイコン、iOS/Android固有の設定などをここで定義します。

<details><summary>app.jsonの例</summary>


```json
{
  "expo": {
    "name": "感情だけを記録する日記アプリ",
    "slug": "wordless-diary",
    "version": "1.0.0",
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.yuto0623.wordlessdiary",
      "infoPlist": {
        "ITSAppUsesNonExemptEncryption": false
      }
    },
    "android": {
      "package": "com.yuto0623.wordlessdiary"
    }
  }
}
```

</details>

#### 2. EAS設定

```bash
npm install -g eas-cli
eas login
eas build:configure
```

[eas.json](https://docs.expo.dev/build/eas-json/)はEAS CLIの設定ファイルです。`eas build:configure`で自動生成され、開発・プレビュー・本番用のビルドプロファイルを定義できます：

<details><summary>eas.jsonの例</summary>


```json
{
  "cli": {
    "version": ">= 16.28.0",
    "appVersionSource": "remote"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal"
    },
    "production": {
      "autoIncrement": true
    }
  },
  "submit": {
    "production": {}
  }
}
```

</details>

#### 3. プライバシーポリシーの作成

App Store申請には**プライバシーポリシーのURL**が必須です。

これもClaude Codeに「このアプリ用のプライバシーポリシーを作って」と頼んだら、すぐに作成してくれました。GitHub Pagesで公開してます。

<details><summary>プライバシーポリシーの例</summary>


```html
<!-- docs/privacy-policy.html -->
<h1>プライバシーポリシー</h1>
<h2>収集するデータ</h2>
<p>本アプリは、以下のデータをお使いの端末内にのみ保存します：</p>
<ul>
  <li>記録した日付</li>
  <li>選択した感情</li>
  <li>記録した日時</li>
</ul>
<p><strong>これらのデータは端末内にのみ保存され、外部サーバーへの送信は一切行いません。</strong></p>
```

</details>

データを収集しないシンプルなアプリでも、プライバシーポリシーは必要です。

#### 4. ストア用アセットの準備

ここもAIに頼りました。

- **アプリアイコン**: AI画像生成で作成（1024x1024px）
- **スクリーンショット**: 各デバイスサイズ用に複数枚
- **アプリの説明文**: AIに生成してもらった

### Phase 4: ビルドと申請

#### ビルド

```bash
# iOS用ビルド
eas build --platform ios --profile production

# Android用ビルド
eas build --platform android --profile production
```

初回は Apple Developer Program への登録と、証明書の設定が必要です。EASが対話形式で案内してくれます。

#### 申請

```bash
# App Store Connect へ提出
eas submit --platform ios

# Google Play へ提出
eas submit --platform android
```

## ストア公開で学んだこと

### 意外と公開作業のほうがめんどくさい

アプリの実装はClaude Codeが一瞬でやってくれましたが、ストア公開の準備は地味に手間がかかりました。

- App Store Connectでの設定項目が多い
- スクリーンショットを各デバイスサイズ用に用意
- プライバシーポリシーのURLを用意
- 年齢制限やカテゴリの設定

コードを書く時間より、これらの作業の方が時間がかかった印象です。

### ハマったポイント

1. **証明書周り** - EASを使えば自動で管理してくれるが、初回は理解に時間がかかった
2. **スクリーンショット** - 各デバイスサイズ用に用意する必要がある
3. **Google Playのクローズドテスト要件** - 公開前に12人以上のテスターによるクローズドテストが必要。これがまだ完了していないのでAndroid版は未公開

### Claude Codeが役立った場面

- 全部！

## まとめ

Claude Codeを使うことで、**1日でアプリ開発からストア公開まで**を完了できました。

正直、ここまで簡単だとは思っていませんでした。自分がやったことは：

- ChatGPTにアプリのアイデアを相談
- Claude Codeに設計書を作ってもらう
- 設計書を渡してアプリを実装してもらう
- プライバシーポリシーを作ってもらう
- EASのコマンドを実行してビルド・申請

**コードはほぼ書いていません。** Claude Codeが全部やってくれました。

Expo + EAS + Claude Codeの組み合わせは、アプリ開発のハードルを劇的に下げてくれます。「アプリ開発に興味はあるけど難しそう」と思っている方は、ぜひ試してみてください。思っている以上に簡単です。

## リンク

**App Store**
https://apps.apple.com/jp/app/%E6%84%9F%E6%83%85%E3%81%A0%E3%81%91%E3%82%92%E8%A8%98%E9%8C%B2%E3%81%99%E3%82%8B%E6%97%A5%E8%A8%98%E3%82%A2%E3%83%97%E3%83%AA/id6757698845

**GitHub**
https://github.com/yuto0623/wordless-diary
