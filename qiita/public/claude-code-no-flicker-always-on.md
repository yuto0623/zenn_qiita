---
title: Claude Codeのちらつきを消す「NO_FLICKER」モードを常時ONにする方法。
tags:
  - Terminal
  - 開発環境
  - ClaudeCode
private: false
updated_at: '2026-04-07T10:59:30+09:00'
id: 6a8a6516e7caaee176d5
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

Claude Code v2.1.89以降で使える `CLAUDE_CODE_NO_FLICKER` モードを常時ONにする方法を紹介します。

## 設定方法

### settings.jsonで設定する

`~/.claude/settings.json` に記述すると、全プロジェクト共通で有効になります。

```json
{
  "env": {
    "CLAUDE_CODE_NO_FLICKER": "1"
  }
}
```

特定のプロジェクトだけで有効にしたい場合は、プロジェクトルートの `.claude/settings.json` に同じ内容を記述します。

## NO_FLICKERモードで何が変わる？

- **ちらつきが消える** 
- **メモリ使用量が一定** 
- **スクロール位置が安定**
- **マウス操作が使える** 

## だがしかし、日本語テキストをコピーすると文字化けする

![日本語テキストの文字化け](https://raw.githubusercontent.com/yuto0623/zenn_qiita/main/images/claude-code-no-flicker-always-on/image.png)
↑「動作確認OKです。何かお手伝いできることはありますか？」をコピーして貼り付けた結果
