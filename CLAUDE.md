# Excalidraw to HTML — Setup Instructions for Claude

These instructions tell Claude Code how to set up the full Excalidraw MCP → HTML pipeline on any Mac. Follow them step by step.

## What This Does

Gives you a local Excalidraw drawing canvas that Claude can read as structured JSON (element types, positions, dimensions, text, connections). No screenshots, no image interpretation — Claude reads exact data and converts drawings into fully rendered HTML pages.

## Prerequisites

- Node.js 20+ and npm
- Claude Code installed
- Git

## Setup Steps

### 1. Clone the Excalidraw MCP server

```bash
cd ~/ClaudeCode  # or wherever the user keeps projects
git clone https://github.com/yctimlin/mcp_excalidraw.git
cd mcp_excalidraw
npm ci
npm run build
```

### 2. Register the MCP server with Claude Code

```bash
claude mcp add excalidraw --scope user \
  -e EXPRESS_SERVER_URL=http://localhost:3000 \
  -e ENABLE_CANVAS_SYNC=true \
  -- node $(pwd)/dist/index.js
```

### 3. Install the skill

Copy the skill file so Claude knows the draw-to-HTML workflow:

```bash
mkdir -p ~/.claude/skills/excalidraw-to-html
cp /path/to/this/repo/skill/SKILL.md ~/.claude/skills/excalidraw-to-html/SKILL.md
```

Or if cloned from GitHub:

```bash
mkdir -p ~/.claude/skills/excalidraw-to-html
cp ~/ClaudeCode/excalidraw-to-html/skill/SKILL.md ~/.claude/skills/excalidraw-to-html/SKILL.md
```

### 4. Start the canvas server

```bash
cd ~/ClaudeCode/mcp_excalidraw
PORT=3000 npm run canvas
```

This starts the Excalidraw editor at **http://localhost:3000**.

### 5. Verify

Open http://localhost:3000 in a browser. Draw something. Then test the API:

```bash
curl -s http://localhost:3000/api/elements | python3 -c "
import json, sys
data = json.load(sys.stdin)
print(f'Elements on canvas: {len(data.get(\"elements\", []))}')
"
```

If it prints a count, you're good.

## How to Use

1. Open http://localhost:3000 and draw a wireframe using shapes (rectangles, text, diamonds, arrows, ellipses)
2. Wait 2 seconds for auto-sync, or click "Sync to Backend"
3. Tell Claude: "Read my Excalidraw canvas and convert it to HTML"
4. Claude reads the JSON via MCP or REST API and generates a fully rendered HTML page

## Tips for Best Results

- **Use shapes, not freehand.** Rectangles, text labels, arrows, diamonds, and ellipses give Claude exact typed data. Freehand paths are just raw point arrays.
- **Add text labels.** A rectangle labeled "Nav" becomes a `<nav>`. Without labels, Claude has to guess.
- **Nest shapes for hierarchy.** A big rectangle containing smaller ones = a container with child components.
- **Use arrows for flow.** Claude reads arrow bindings to understand which shapes connect.

## Important Notes

- The canvas server stores data **in memory**. Restarting it clears everything. Export before stopping.
- Auto-sync has a **1200ms debounce**. Wait a couple seconds after drawing before asking Claude to read.
- MCP tools (`describe_scene`, `export_scene`, etc.) only work after restarting Claude Code (they load at session start). The REST API (`curl localhost:3000/api/elements`) works immediately as a fallback.
- This uses [yctimlin/mcp_excalidraw](https://github.com/yctimlin/mcp_excalidraw) (community). The official Excalidraw MCP is write-only — it can't read your drawings back.

## Multi-Machine Setup

If you run the canvas on Machine A and Claude Code on Machine B (e.g., via Tailscale):

```bash
# On Machine B, register MCP pointing to Machine A's IP
claude mcp add excalidraw --scope user \
  -e EXPRESS_SERVER_URL=http://<machine-a-ip>:3000 \
  -e ENABLE_CANVAS_SYNC=true \
  -- node /path/to/mcp_excalidraw/dist/index.js
```

## Quick Reference

| What | Where |
|------|-------|
| Drawing canvas | http://localhost:3000 |
| REST API | http://localhost:3000/api/elements |
| MCP server name | `excalidraw` |
| Skill location | `~/.claude/skills/excalidraw-to-html/SKILL.md` |
| Canvas server start | `cd ~/ClaudeCode/mcp_excalidraw && PORT=3000 npm run canvas` |
