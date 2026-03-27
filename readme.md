# Skills

Curated agent skills for Claude Code / Codex.

## Install

```bash
# As Claude plugin
/plugin marketplace add YunosukeYoshino/skills

# Or via npx
npx skills add YunosukeYoshino/skills --skill '*' -g -a claude-code -a codex -y

# Specific skill
npx skills add YunosukeYoshino/skills --skill <name> -g -a claude-code -a codex -y
```

## Use in Codex

Codex plugin publishing to the official directory is not self-serve yet, so the
current path is local plugin usage after cloning this repo.

This repo includes `.codex-plugin/plugin.json` at the root, so the cloned repo
itself can be treated as the plugin root.

Create `.agents/plugins/marketplace.json` inside the cloned repo and point the
plugin entry at `./`:

```json
{
  "name": "local-repo",
  "plugins": [
    {
      "name": "yunosuke-skills",
      "source": {
        "source": "local",
        "path": "./"
      },
      "policy": {
        "installation": "AVAILABLE",
        "authentication": "ON_INSTALL"
      },
      "category": "Productivity"
    }
  ]
}
```

Restart Codex after adding the marketplace file and verify that the plugin
appears in the local marketplace.

## Available Skills

### design

| Skill | Description |
|-------|-------------|
| beautiful-mermaid | Mermaid記法でSVGダイアグラムを生成し、READMEやMarkdownに埋め込む。3モード: `render`（既存Mermaidコード→テーマ付きSVG）、`generate`（コードベース分析→自動生成）、`create`（自然 |

### engineering

| Skill | Description |
|-------|-------------|
| tdd-twada | t.wada流テスト駆動開発。Red-Green-Refactorサイクルで機能を実装。トリガー: 「TDDで実装して」「テスト駆動で」「テストファーストで」「TDD」「Red-Green-Refactor」。TypeScript/Java |

### external-tools

| Skill | Description |
|-------|-------------|
| codex | OpenAI Codex CLIを使用したコードレビュー・分析を実行する。使用場面: ユーザーが「codexでレビューして」「codexで分析して」「codex exec」のようにCodex CLIの実行を明示的に依頼した場合のみ使用。単に |
