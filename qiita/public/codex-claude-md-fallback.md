---
title: CodexでAGENTS.mdがないときにCLAUDE.mdを読む設定を知った
publication_name: aun_phonogram
private: false
tags:
  - Codex
  - ClaudeCode
  - 開発環境
updated_at: '2026-02-24T03:12:06.751Z'
id: null
organization_url_name: null
slide: false
---

## はじめに

`AGENTS.md` がない場合に別ファイル名をフォールバックできる設定を初めて知りました。

結論、`~/.codex/config.toml` にこれを書くだけです。

```toml
# ~/.codex/config.toml
project_doc_fallback_filenames = ["CLAUDE.md"]
```

これで、既存プロジェクトが `CLAUDE.md` 運用でも、Codexが指示を読んでくれます。
`AGENTS.md` を `CLAUDE.md` へのシンボリックリンクにしなくても、`project_doc_fallback_filenames` の設定だけで `CLAUDE.md` を読み込ませられることを知りました。

## どんなときに便利？

- すでに `CLAUDE.md` を各ディレクトリで運用している
- Claude CodeとCodexを併用している

## 読み込み順（ここが大事）

Codexは各ディレクトリで次の順に探します。

1. `AGENTS.md`
2. `project_doc_fallback_filenames` に並べた名前（例: `CLAUDE.md`）

つまり、`AGENTS.md` があればそちらが優先され、ないときだけ `CLAUDE.md` が使われます。

## 設定例

```toml
# ~/.codex/config.toml
project_doc_fallback_filenames = ["CLAUDE.md"]
```

## 注意点

- 設定変更後はCodexを再起動（または新規セッション開始）

## 参考

- https://developers.openai.com/codex/guides/agents-md
