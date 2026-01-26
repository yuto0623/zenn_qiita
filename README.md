# Zenn & Qiita 記事管理

ZennとQiitaの記事を1つのリポジトリで同時管理するプロジェクトです。

## 仕組み

[zenn-qiita-sync](https://github.com/C-Naoki/zenn-qiita-sync) を使用して、Zennの記事をQiitaに自動同期します。

1. `articles/` に記事を作成
2. `main` ブランチにpush
3. GitHub Actionsが自動で Qiita に同期

## ディレクトリ構成

```
articles/      # Zenn記事（ここに書く）
qiita/public/  # Qiita記事（自動生成）
images/        # 画像
books/         # Zenn本
```

## 使い方

```bash
# プレビュー
npx zenn preview

# 新規記事作成
npx zenn new:article
```

## セットアップ

リポジトリの Settings > Secrets に `QIITA_TOKEN` を設定してください。

## 参考

- [Zenn CLIの使い方](https://zenn.dev/zenn/articles/zenn-cli-guide)
- [zenn-qiita-sync](https://github.com/C-Naoki/zenn-qiita-sync)
- [Zenn vs Qiitaを終わらせに来た](https://zenn.dev/naoki0103/articles/zenn-qiita-sync-workflow)
