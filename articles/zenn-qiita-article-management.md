---
title: "ZennとQiitaの記事の同時管理"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Zenn,ZennCLI,Qiita,QiitaCLI]
published: false
---
## はじめに
私は最近ZennとQiitaの記事を投稿し始めました。
どちらも素晴らしいプラットフォームですが、記事を両方に投稿する際はコピペして少し修正しないといけないので管理が大変になります。
そこで、ZennとQiitaの記事を1つのGitHubリポジトリで管理する方法を見つけましたので共有したいと思います。

## 方法
C-Naokiさんが制作されている[zenn-qiita-sync](https://github.com/C-Naoki/zenn-qiita-sync
)を使用してZennとQiitaの記事を1つのGitHubリポジトリで管理します。

https://github.com/C-Naoki/zenn-qiita-sync

[zenn-qiita-sync](https://github.com/C-Naoki/zenn-qiita-sync
)の使い方はC-Naokiさんご本人が書かれた「[Zenn vs Qiitaを終わらせに来た
](https://zenn.dev/naoki0103/articles/zenn-qiita-sync-workflow)」の記事で使い方等の手順が分かるので参考にしてみてください。

https://zenn.dev/naoki0103/articles/zenn-qiita-sync-workflow

## 最後に
このツールを試す際に、同じ投稿を複数投稿してしまうなどしてしまいました。
同じ投稿が表示されてしまった方など荒らしてしまい申し訳ございません。
