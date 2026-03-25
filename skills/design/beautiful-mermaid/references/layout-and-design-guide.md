# Layout Constraints and Design Principles

ELKレイアウトエンジンの制約事項とダイアグラム設計原則を統合したガイドライン。
generateモードおよびcreateモードでMermaidコードを生成する際に必ず参照する。

## Layout Constraints (IMPORTANT)

beautiful-mermaid uses the ELK layout engine. Complex graph structures cause layout breakage.
Follow these rules to ensure clean rendering:

**Flat flow preferred over subgraphs:**
- Use a flat linear flow (`A --> B --> C`) rather than nesting everything in subgraphs
- Subgraphs are fine for simple grouping, but avoid edges between subgraphs (`subgraph1 --> subgraph2`) as ELK misaligns them
- If using subgraphs, keep to 1-2 levels max, with edges only between nodes inside them

**Node count limits:**
- Flat flow: up to 20 nodes works well
- With subgraphs: keep under 15 nodes total
- If more nodes are needed, split into multiple separate diagrams

**Avoid special characters in labels:**
- Do not use `%`, `+`, `&`, `<`, `>` in node or edge labels -- they cause rendering artifacts
- Use Japanese equivalents instead: `80%以上` -> `80以上`, `A+B` -> `AとB`
- Parentheses `()` in labels are OK

**Subgraph IDs must be ASCII:**
- Use ASCII IDs with Japanese display labels: `subgraph auth["認証フロー"]`
- Do not use Japanese characters in subgraph IDs: ~~`subgraph 認証フロー`~~

**Font:**
- Always use `Noto Sans JP` for Japanese text support (not `Inter`)
- Inter does not include CJK glyphs and causes misalignment

## Diagram Design Principles

When generating diagrams from code analysis:

- Keep node count manageable (under 20 nodes per diagram). Split into multiple diagrams if the system is complex.
- Use meaningful, concise labels derived from actual code names
- Show the most important relationships, not every possible connection
- Add edge labels to clarify the nature of connections (e.g., "HTTP", "gRPC", "imports")
