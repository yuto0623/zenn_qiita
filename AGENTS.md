# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## プロジェクト概要

ZennとQiitaの記事を1つのGitHubリポジトリで同時管理するプロジェクト。`zenn-qiita-sync` GitHub Actionにより、Zennの記事をQiitaに自動同期する。

## アーキテクチャ

```
articles/          # Zenn記事（ここに記事を書く）
qiita/public/      # Qiita記事（自動生成、編集不要）
images/            # 画像ファイル
books/             # Zenn本
```

**重要**: 記事は `articles/` ディレクトリにのみ作成・編集する。`qiita/public/` は GitHub Actions により自動生成される。

## 記事のフロントマター

Zenn記事 (`articles/*.md`) のフロントマター形式:

```yaml
---
title: "記事タイトル"
emoji: "🐷"
type: "tech"  # tech: 技術記事 / idea: アイデア
topics: [Topic1, Topic2]
published: true
---
```

## コマンド

```bash
# Zennプレビュー（ローカル確認）
npx zenn preview

# 新規記事作成
npx zenn new:article

# 新規本作成
npx zenn new:book
```

## 自動同期の仕組み

`main` ブランチにpushすると `.github/workflows/publish.yml` が実行され:
1. Zenn記事がQiita形式に変換される
2. `qiita/public/` に同期される
3. Qiita CLIでQiitaに公開される

`QIITA_TOKEN` シークレットがリポジトリに設定されている必要がある。
