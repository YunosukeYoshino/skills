---
name: harness-auditor
description: >
  Harness Engineering健全性を決定論的シェルコマンドで検査する。
  CLAUDE.md肥大化、Hooks設定、リンター、pre-commit、ADR、テスト、設定保護を
  exit codeベースで判定しHTMLレポートをブラウザで表示する。
  Use PROACTIVELY when: "harness audit", "harness check",
  "ハーネス監査", "ハーネスチェック", "開発基盤チェック", "健全性チェック".
---

# Harness Engineering Audit

全チェックはexit code (0=pass) で判定する。LLMの主観的評価は行わない。

## Workflow

1. 設定パスを解決する
2. 8つのチェックを順番に実行し、exit codeを記録する
3. 結果JSONを生成し、`scripts/generate-report.sh` でHTMLレポートを生成する
4. `open` コマンドでブラウザに表示する

## 設定パス解決

```bash
SETTINGS="$(test -f .claude/settings.json && echo .claude/settings.json || echo ~/.claude/settings.json)"
```

## Checks

| # | Check | Command | Pass条件 |
|---|-------|---------|----------|
| 1 | CLAUDE.md行数 | `test "$(wc -l < CLAUDE.md 2>/dev/null \|\| echo 999)" -le 200` | exit 0 (推奨<=50) |
| 2 | AGENTS.md行数 | `test -z "$(find . -name AGENTS.md -exec sh -c '[ "$(wc -l < "$1")" -gt 200 ] && echo "$1"' _ {} \;)"` | exit 0 (全て<=200) |
| 3 | Hooks 3種 | `jq -e '.hooks \| (has("PreToolUse") and has("PostToolUse") and has("Stop"))' "$SETTINGS"` | exit 0 |
| 4 | リンター設定 | `find . -maxdepth 2 \( -name biome.json -o -name pyproject.toml -o -name .golangci.yml -o -name Cargo.toml \) \| grep -q .` | exit 0 |
| 5 | Pre-commit | `find . -maxdepth 1 \( -name lefthook.yml -o -name .pre-commit-config.yaml \) -o -type d -name .husky \| grep -q .` | exit 0 |
| 6 | ADR | `find . -path '*/adr/*.md' -not -path '*/node_modules/*' \| grep -q .` | exit 0 |
| 7 | テスト | `find . \( -name '*.test.*' -o -name '*.spec.*' -o -name '*_test.go' \) -not -path '*/node_modules/*' \| grep -q .` | exit 0 |
| 8 | 設定保護Hook | `jq -e '[.hooks.PreToolUse[]?.hooks[]? \| .command // ""] \| any(test("biome\|eslint\|golangci\|pyproject\|deny\|block"))' "$SETTINGS"` | exit 0 |

## Report Generation

各チェック実行後、結果をJSON配列として一時ファイルに書き出す:

```json
[
  {"id":1,"name":"CLAUDE.md行数","exit_code":0,"action":"-"},
  {"id":2,"name":"AGENTS.md行数","exit_code":1,"action":"200行以下に分割"}
]
```

FAILには修正アクションを1行で併記、PASSのactionは`"-"`。

レポート生成と表示:

```bash
SKILL_DIR="$(dirname "$(readlink -f "$0" 2>/dev/null || echo ~/.claude/skills/harness-audit/scripts)")"
REPORT="/tmp/harness-audit-$(date +%Y%m%d-%H%M%S).html"
bash ~/.claude/skills/harness-audit/scripts/generate-report.sh /tmp/harness-audit-results.json "$REPORT"
open "$REPORT"
```
