---
title: "【体験談】Amazonを装ったフィッシング詐欺に引っかかりそうになった話"
emoji: "🙌"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [メール, security, 詐欺]
published: true
---

## はじめに

先日、Amazon を装ったフィッシング詐欺メールに引っかかりそうになってしまいました。幸いカード情報を抜かれる前に気づくことができましたが、一歩間違えれば大きな被害を受けていたかもしれません。同じような被害に遭わないよう、この体験を共有したいと思います。

## 事の発端：怪しいメールの受信

ある日、スマートフォンに Amazon からと思われるメールが届きました。内容は「Amazon プライムの月額会費（税込 600 円）の決済処理が完了できませんでした」というものでした。

実は、最近 Amazon プライムを解約したばかりだったため、「解約手続きに何か問題があったのかもしれない」と思い、このメールを信じてしまいました。

![](/images/amazon-phishing-scam/img.jpg)

## 詐欺サイトへの誘導

メールに記載されていた「今すぐ更新する」ボタンをクリックしてしまいました。すると、Amazon のログイン画面そっくりのページに飛びました。

![](/images/amazon-phishing-scam/img2.jpg)

ここですぐ気づくべきだったのですが、時間が無いこともあり冷静な判断ができず、**アカウント情報（メールアドレスとパスワード）を入力してログインしてしまいました。**

## 危険な瞬間：カード情報入力画面

ログイン後、次に表示されたのはクレジットカード情報の入力画面でした。ここで初めて「何かおかしい」と感じました。

**この時点で詐欺だと気づき、カード情報は入力せずにページを閉じました。**

## 気づくべきだった警告サイン

後から振り返ると、以下の点で詐欺だと気づけるはずでした。

### 1. メールアドレスの確認不足

送信者のメールアドレスをよく確認すべきでした。正規の Amazon からのメールではなかったです。

### 2. URL ドメインの確認不足

ブラウザのアドレスバーを見ると、`mgzn0.com` というドメインでした。Amazon の正規サイトは `amazon.co.jp` のはずです。

## 被害を最小限に抑えるためにとった対策

カード情報は入力しませんでしたが、アカウント情報を入力してしまったため、以下の対策を取りました：

1. **Amazon アカウントのパスワード変更**
2. **二段階認証の設定確認**
3. **アカウントの不正利用がないかチェック**
4. **同じパスワードを使用している他のサービスのパスワード変更**

## フィッシング詐欺を見分けるポイント

今回の経験から学んだ、フィッシング詐欺を見分けるポイントをまとめます：

### メール編

- **送信者のメールアドレスを確認**：正規ドメインかチェック
- **緊急性を煽る内容に注意**：「今すぐ」「期限迫る」などの表現
- **リンクにマウスオーバー**：実際の飛び先 URL を確認

### Web サイト編

- **URL のドメインを確認**：正規サイトのドメインと一致するか
- **SSL 証明書の確認**：鍵マークがあるか、証明書は正しいか
- **サイトの作りを確認**：デザインや文章におかしい点はないか

## まとめ

今回は幸いカード情報を抜かれる前に気づくことができましたが、一歩間違えれば深刻な被害を受けていたかもしれません。

**重要なのは、メールや URL を慎重に確認する習慣を身につけることです。**

- メールアドレス・ドメインの確認
- 緊急性を煽る内容への警戒
- 個人情報入力前の再確認

皆さんも同様の詐欺メールを受信した際は、必ず送信者と URL を確認してから行動するようにしてください。「少しでも怪しい」と思ったら、一度ブラウザを閉じて、公式サイトから直接アクセスすることをお勧めします。

---

_この記事が、フィッシング詐欺被害の防止に少しでも役立てば幸いです。_
