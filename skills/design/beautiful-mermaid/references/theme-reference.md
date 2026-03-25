# Theme Reference

テーマ一覧、カスタムテーマの定義方法、テーマ選択ロジックを統合したリファレンス。
全3モード（render/generate/create）でテーマを選択する際に参照する。

## Built-in Themes (15 total)

| Theme | Type | Best For |
|-------|------|----------|
| `github-light` | Light | Light-mode README |
| `github-dark` | Dark | Dark-mode README |
| `tokyo-night` | Dark | Developer-focused docs |
| `catppuccin-mocha` | Dark | Modern dark aesthetic |
| `catppuccin-latte` | Light | Soft light aesthetic |
| `nord` | Dark | Scandinavian minimal |
| `nord-light` | Light | Clean documentation |
| `dracula` | Dark | High contrast |
| `zinc-light` | Light | Minimal, no accent |
| `zinc-dark` | Dark | Minimal, no accent |
| `solarized-light` | Light | Warm documentation |
| `solarized-dark` | Dark | Warm dark docs |
| `one-dark` | Dark | Atom-style |
| `tokyo-night-storm` | Dark | Deeper Tokyo Night |
| `tokyo-night-light` | Light | Light Tokyo variant |

## Custom Themes

For custom colors, pass a `DiagramColors` object instead of a theme name:

```typescript
const svg = renderMermaidSVG(source, {
  bg: '#0f0f0f',
  fg: '#e0e0e0',
  accent: '#ff6b6b',  // arrows, highlights
  muted: '#666666',   // secondary text
})
```

## Theme Selection Logic

- Default: `github-light` (white background, clean and readable)
- Dark mode README: use `github-dark`
- Technical docs: use `nord-light` or `catppuccin-latte`
- If the user explicitly requests dark theme: use `github-dark` or `tokyo-night`
- For transparent backgrounds (works on any theme): add `transparent: true`
