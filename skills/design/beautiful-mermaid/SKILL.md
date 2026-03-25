---
name: beautiful-mermaid
description: >
  Mermaid記法でSVGダイアグラムを生成し、READMEやMarkdownに埋め込む。3モード:
  `render`（既存Mermaidコード→テーマ付きSVG）、`generate`（コードベース分析→自動生成）、`create`（自然言語→ダイアグラム）。トリガー:
  「Mermaid」「マーメイド」「Mermaidで図を作って」「フローチャート」「可視化」「アーキテクチャ図」「シーケンス図」「ER図」「クラス図」「状態遷移図」。draw.io形式が指定された場合はdrawio-diagram-forgeスキルを使用すること（このスキルはMermaid記法専用）。
  or architecture that would benefit from visual representation.
---

# Beautiful Mermaid

Render Mermaid diagrams as beautifully themed SVG files using the `beautiful-mermaid` library.
Output is optimized for embedding in README.md and other Markdown documents.

## Language

Mermaid diagram内のノードラベル、エッジラベル、サブグラフタイトルは全て日本語で記述する。
ユーザーへの説明やMarkdown埋め込みスニペットのalt textも日本語で書く。

例:
- Node: `A[ユーザー入力]` (not `A[User Input]`)
- Edge: `A -->|認証成功| B` (not `A -->|Auth Success| B`)
- Subgraph: `subgraph auth["認証フロー"]` (ASCII ID + Japanese label)
- Alt text: `![ユーザー登録フロー](docs/diagrams/user-registration.svg)`

技術用語（API, DB, HTTP, gRPC等）はそのまま使って良い。

## Prerequisites

The project must have `beautiful-mermaid` installed:

```bash
npm install beautiful-mermaid
# or
bun add beautiful-mermaid
```

If not installed, install it before proceeding.

## Modes

This skill operates in three modes, selected by argument:

| Mode | Argument | Description |
|------|----------|-------------|
| **Render** | `render` | Take existing Mermaid code and render it as a themed SVG |
| **Generate** | `generate` | Analyze codebase and auto-generate architecture diagrams |
| **Create** | `create` | Convert natural language description into a Mermaid diagram |

If no argument is given, ask the user which mode they want, or infer from context.

---

## Mode: Render

Take existing Mermaid source code and produce a themed SVG file.

### Steps

1. Get the Mermaid source from the user (inline, file path, or clipboard)
2. Ask for theme preference if not specified (default: `github-light`). Read `references/theme-reference.md` for available themes and selection logic.
3. Generate and execute a render script. Read `references/render-options-and-troubleshooting.md` for render option details if non-default settings are needed.
4. Save SVG to the project and provide the Markdown embed snippet

### Render Script Template

Create a temporary script at the project root and run it with `bun`:

```typescript
import { renderMermaidSVG, THEMES } from 'beautiful-mermaid'
import { writeFileSync } from 'fs'

const source = `
<MERMAID_SOURCE_HERE>
`

const theme = THEMES['<THEME_NAME>'] // or custom { bg, fg } object
const svg = renderMermaidSVG(source.trim(), {
  ...theme,
  padding: 40,
  font: 'Noto Sans JP',
})

writeFileSync('<OUTPUT_PATH>', svg)
console.log('SVG saved to <OUTPUT_PATH>')
```

Run with bun (required -- the package exports raw .ts files):
```bash
bun run render-mermaid.tmp.ts && rm render-mermaid.tmp.ts
```

### Output

- Save SVG to `docs/diagrams/<name>.svg` (create directory if needed)
- Print the Markdown embed snippet:
  ```markdown
  ![<diagram description>](docs/diagrams/<name>.svg)
  ```

---

## Mode: Generate

Analyze the codebase and automatically generate architecture or flow diagrams.

### Steps

1. Identify what to visualize. Ask if unclear. Common targets:
   - **Directory structure** - module dependency graph
   - **API flow** - request lifecycle through middleware/handlers
   - **Data model** - ER diagram from schema definitions
   - **State machine** - state transitions from state management code
   - **Class hierarchy** - inheritance and composition relationships
   - **Component tree** - React/Vue component hierarchy

2. Read relevant source files to understand the structure

3. Compose Mermaid source code that accurately represents the codebase:
   - Use `graph TD/LR` for architecture and flow diagrams
   - Use `erDiagram` for data models
   - Use `classDiagram` for class hierarchies
   - Use `sequenceDiagram` for request/response flows
   - Use `stateDiagram-v2` for state machines
   - Read `references/mermaid-syntax-reference.md` for supported shapes, edge types, and directions.
   - Read `references/layout-and-design-guide.md` for ELK layout constraints and design principles. This is critical for clean rendering.

4. Read `references/theme-reference.md` for theme selection. Render using the same approach as Render mode.

5. Save and provide the embed snippet

---

## Mode: Create

Convert a natural language description into a Mermaid diagram.

### Steps

1. Understand what the user wants to visualize from their description
2. Select the most appropriate diagram type:
   - **Flowchart** for processes, workflows, decision trees
   - **Sequence** for time-ordered interactions between actors
   - **Class** for OOP structures, interfaces, relationships
   - **ER** for database schemas, entity relationships
   - **State** for state machines, lifecycle flows
3. Write clean Mermaid source that captures the description.
   - Read `references/mermaid-syntax-reference.md` for supported shapes, edge types, and directions.
   - Read `references/layout-and-design-guide.md` for ELK layout constraints and design principles. This is critical for clean rendering.
4. Read `references/theme-reference.md` for theme selection. Render and save using the same approach as Render mode.
5. Show the Mermaid source to the user so they can iterate

---

## References

| File | Content | When to Read |
|------|---------|--------------|
| `references/mermaid-syntax-reference.md` | Mermaid構文リファレンス（形状、エッジ、方向） | generate/createモードでMermaidコードを生成する際 |
| `references/layout-and-design-guide.md` | ELKレイアウト制約とデザイン原則 | generate/createモードでMermaidコードを生成する際（必須） |
| `references/theme-reference.md` | テーマ一覧、カスタムテーマ、選択ロジック | 全モードでテーマを選択する際 |
| `references/render-options-and-troubleshooting.md` | レンダリングオプションとトラブルシューティング | レンダリング設定の調整時またはエラー発生時 |

---

## Output Convention

- Directory: `docs/diagrams/` (relative to project root)
- Naming: `<descriptive-name>.svg` (kebab-case)
- Always provide the Markdown embed snippet after generating

### For GitHub README Compatibility

GitHub renders SVG files referenced in Markdown. The generated SVGs are self-contained
(fonts imported via Google Fonts URL in the SVG style block) and render correctly on GitHub.

If the user needs the diagram inline in a README that's already being edited,
insert the image reference at the appropriate location.

---

## Examples

**Render existing code:**
```
/beautiful-mermaid render

graph TD
  A[User Request] --> B[API Gateway]
  B --> C{Auth?}
  C -->|Yes| D[Service]
  C -->|No| E[401 Error]
```

**Generate from codebase:**
```
/beautiful-mermaid generate
src/ディレクトリのモジュール依存関係を可視化して
```

**Create from description:**
```
/beautiful-mermaid create
ユーザーがログインしてからダッシュボードを表示するまでのフローを作って
```
