# Excalidraw to HTML

Draw wireframes in Excalidraw. Let Claude turn them into fully rendered HTML pages.

**The key idea:** instead of screenshotting your drawing and hoping Claude interprets the pixels correctly, this setup gives Claude direct access to the structured JSON — every shape, position, dimension, text label, and connection. No guessing.

## How It Works

```
You draw in Excalidraw  →  MCP reads JSON  →  Claude generates HTML
    (localhost:3000)        (exact data)       (pixel-accurate)
```

1. You draw a wireframe using shapes in a local Excalidraw instance
2. Claude reads the canvas as structured JSON via MCP (or REST API fallback)
3. Claude maps shapes to semantic HTML elements and generates a complete page

A rectangle labeled "Nav" becomes a `<nav>`. Two stacked boxes inside a container become a card layout. An arrow pointing to a diamond becomes a flow step. Claude reads the relationships, not just the shapes.

## What's in This Repo

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Setup instructions — paste "set this up" and your Claude follows these |
| `skill/SKILL.md` | The skill file Claude uses to know the draw → HTML workflow |
| `index.html` | Visual explainer ([GitHub Pages](https://maxkwaaste.github.io/excalidraw-to-html)) |

## Quick Start

1. **Clone this repo** and follow `CLAUDE.md`, or just tell Claude:

> "Clone https://github.com/maxkwaaste/excalidraw-to-html and follow the CLAUDE.md to set everything up"

2. **Start the canvas:** `cd ~/ClaudeCode/mcp_excalidraw && PORT=3000 npm run canvas`

3. **Draw** at http://localhost:3000

4. **Tell Claude:** "Read my Excalidraw canvas and convert it to HTML"

## Requirements

- Node.js 20+
- Claude Code
- Git

## Credits

Built on [yctimlin/mcp_excalidraw](https://github.com/yctimlin/mcp_excalidraw) — the community MCP server that wraps the open-source Excalidraw editor with a REST API and WebSocket sync layer.

## License

MIT
