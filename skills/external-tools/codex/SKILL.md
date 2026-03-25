---
name: codex
description: >
  OpenAI Codex CLIを使用したコードレビュー・分析を実行する。使用場面:
  ユーザーが「codexでレビューして」「codexで分析して」「codex exec」のようにCodex
  CLIの実行を明示的に依頼した場合のみ使用。単にエラーログやテキスト中に'codex'という文字列が含まれているだけではトリガーしない。トリガー:
  「codexでレビュー」「codexで分析」「codex
  exec」「/codex」。コードレビューやバグ調査の一般的な依頼にはトリガーしない（copilot-cliとの棲み分け）。

---

# Codex

Codex CLIを使用してコードレビュー・分析を実行するスキル。

## 実行コマンド

codex exec --full-auto --sandbox read-only --cd <project_directory> "<request>"

## パラメータ

| パラメータ | 説明 |
|-----------|------|
| `--full-auto` | 完全自動モードで実行 |
| `--sandbox read-only` | 読み取り専用サンドボックス（安全な分析用） |
| `--cd <dir>` | 対象プロジェクトのディレクトリ |
| `"<request>"` | 依頼内容（日本語可） |

## 使用例

### コードレビュー
codex exec --full-auto --sandbox read-only --cd /path/to/project "このプロジェクトのコードをレビューして、改善点を指摘してください"

### バグ調査
codex exec --full-auto --sandbox read-only --cd /path/to/project "認証処理でエラーが発生する原因を調査してください"

## 実行手順

1. ユーザーから依頼内容を受け取る
2. 対象プロジェクトのディレクトリを特定する
3. 上記コマンド形式でCodexを実行
4. 結果をユーザーに報告
