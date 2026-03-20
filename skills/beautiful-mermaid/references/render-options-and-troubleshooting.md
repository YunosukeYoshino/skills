# Render Options and Troubleshooting

レンダリングオプション一覧とトラブルシューティング情報。
レンダリング実行時の設定確認やエラー発生時に参照する。

## Render Options

| Option | Default | Description |
|--------|---------|-------------|
| `padding` | `40` | Canvas padding in px |
| `nodeSpacing` | `24` | Horizontal spacing between nodes |
| `layerSpacing` | `40` | Vertical spacing between layers |
| `font` | `Noto Sans JP` | Font family (CJK support required) |
| `transparent` | `false` | Transparent background |

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|---------|
| `Cannot find module 'beautiful-mermaid'` | Not installed | Run `npm install beautiful-mermaid` |
| SVG appears blank | Invalid Mermaid syntax | Validate syntax before rendering |
| Fonts not rendering on GitHub | Expected behavior | GitHub sanitizes font imports; layout is preserved |
| `bun` not found | bun not installed | See https://bun.sh for installation instructions |
