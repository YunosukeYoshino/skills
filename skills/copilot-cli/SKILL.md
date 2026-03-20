---
name: copilot-cli
description: "GitHub Copilot CLIを使用してバグ調査・エラー原因分析・コードベース調査を実行する。使用場面: (1) バグ・エラーの原因調査、(2) 再現手順の仮説立案、(3) ログ/設定/依存関係の確認、(4) 原因候補の確度付き分析、(5) リファクタリング提案、(6) コードレビュー。トリガー: copilot, 調査して, 原因を調べて, デバッグして, /copilot"
---

# Investigating with Copilot CLI

GitHub Copilot CLIを使用してコードの調査・デバッグ・分析を実行するスキル。

## 実行コマンド

```bash
copilot -p "<request>" \
  --silent \
  --no-ask-user \
  --allow-all-tools \
  --add-dir <project_directory> \
  --disallow-temp-dir \
  --available-tools bash \
  --available-tools rg \
  --available-tools glob \
  --available-tools view \
  --disable-builtin-mcps \
  --model gpt-5.3-codex
```

## パラメータ

| パラメータ | 説明 |
|-----------|------|
| `-p "<request>"` | 調査内容・依頼（日本語可） |
| `--silent` | **エージェントの最終応答のみ出力**（MCPログ・進捗・使用統計を抑制） |
| `--no-ask-user` | エージェントがユーザーに質問しない（自律実行） |
| `--allow-all-tools` | 非インタラクティブモードでツールを確認なしで実行（`-p` 使用時に必須） |
| `--add-dir <dir>` | 対象プロジェクトのディレクトリ（通常は `.`） |
| `--disallow-temp-dir` | 一時ディレクトリの使用禁止（安全な読み取り専用分析） |
| `--available-tools <tool>` | **ツールごとに個別フラグで指定**（カンマ区切り不可）。`--allow-all-tools` と組み合わせると列挙ツールのみ使用可 |
| `--disable-builtin-mcps` | 組み込み GitHub MCP サーバーを無効化（読み取り専用分析では不要、ノイズ削減） |
| `--model gpt-5.3-codex` | 使用モデルの指定 |

> **重要**: `--available-tools` はカンマ区切りでの一括指定不可。ツールごとに個別フラグを使う。
> 例: `--available-tools bash --available-tools rg` （`--available-tools bash,rg` は NG）

> **`--silent` について**: MCPツール呼び出しの進捗ログ・使用統計などを抑制し、エージェントの最終回答テキストのみを stdout に出力する。スクリプトや結果のパイプ処理に有効。

## 使用例

### バグ・エラー調査（推奨プロンプトパターン）

```bash
copilot -p "認証処理でエラーが出る。再現手順の仮説を立て、ログ/設定/依存関係を確認するために必要なコマンドを実行し、原因候補を確度付きで出して。" \
  --silent \
  --no-ask-user \
  --allow-all-tools \
  --add-dir . \
  --disallow-temp-dir \
  --available-tools bash \
  --available-tools rg \
  --available-tools glob \
  --available-tools view \
  --disable-builtin-mcps \
  --model gpt-5.3-codex
```

### コードレビュー

```bash
copilot -p "このプロジェクトのコードをレビューして、改善点・セキュリティ上の問題を指摘してください。" \
  --silent \
  --no-ask-user \
  --allow-all-tools \
  --add-dir . \
  --disallow-temp-dir \
  --available-tools bash \
  --available-tools rg \
  --available-tools glob \
  --available-tools view \
  --disable-builtin-mcps \
  --model gpt-5.3-codex
```

### 依存関係・設定の確認

```bash
copilot -p "依存関係のバージョン競合や設定ミスがないか確認し、問題があれば原因と対処法を示してください。" \
  --silent \
  --no-ask-user \
  --allow-all-tools \
  --add-dir . \
  --disallow-temp-dir \
  --available-tools bash \
  --available-tools rg \
  --available-tools glob \
  --available-tools view \
  --disable-builtin-mcps \
  --model gpt-5.3-codex
```

### 結果を変数・ファイルに受け取る

```bash
# 変数に格納
result=$(copilot -p "テストの失敗原因を調べて" --silent --no-ask-user --allow-all-tools --add-dir . --disable-builtin-mcps)

# ファイルに出力
copilot -p "コードレビュー結果を出して" --silent --no-ask-user --allow-all-tools --add-dir . --disable-builtin-mcps > review.md
```

## 実行手順

1. ユーザーから調査内容・依頼を受け取る
2. 対象プロジェクトのディレクトリを特定する（不明な場合は `.` を使用）
3. 上記コマンド形式で Copilot CLI を実行
4. 結果をユーザーに報告

## 効果的なプロンプトパターン

デバッグ・調査時は以下の構造でプロンプトを組み立てると精度が高い:

```
{症状の説明}。{仮説立案の依頼}、{確認事項（ログ/設定/依存関係など）}を確認するために必要なコマンドを実行し、{出力形式（原因候補を確度付きで出す など）}。
```

## 出力制御チートシート

| やりたいこと | フラグ |
|-------------|--------|
| MCPログ・進捗を消してクリーンな出力にする | `--silent` |
| ファイルログも無効化する | `--log-level none` |
| GitHub MCP サーバーを無効化する | `--disable-builtin-mcps` |
| 特定 MCP サーバーだけ無効化する | `--disable-mcp-server <name>` |
