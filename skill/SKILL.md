---
name: excalidraw-to-html
description: Read structured Excalidraw canvas data via MCP or REST API and convert drawings into fully rendered HTML pages. Uses JSON element data (not screenshots) for accurate conversion.
---

# Excalidraw to HTML Skill

## When to Use

When the user draws something in Excalidraw (localhost:3000) and wants it converted to a rendered HTML page.

## Canvas Access

- **Drawing URL:** http://localhost:3000
- **REST API:** http://localhost:3000/api/elements
- **MCP tools (when loaded):** `describe_scene`, `export_scene`, `query_elements`

## Workflow

### Step 1: Read the canvas

If MCP tools are loaded, use `describe_scene` or `export_scene`.

If MCP tools aren't available (mid-session add), fall back to the REST API:

```bash
curl -s http://localhost:3000/api/elements | python3 -c "
import json, sys
data = json.load(sys.stdin)
elements = data.get('elements', [])
print(f'Total elements: {len(elements)}')
for el in elements:
    t = el.get('type')
    x, y = round(el.get('x', 0)), round(el.get('y', 0))
    w, h = round(el.get('width', 0)), round(el.get('height', 0))
    text = el.get('text', '')
    stroke = el.get('strokeColor', '')
    bg = el.get('backgroundColor', '')
    bound = el.get('boundElements', [])
    contained = el.get('containerId', '')
    start = el.get('startBinding', None)
    end = el.get('endBinding', None)
    deleted = el.get('isDeleted', False)
    if deleted: continue
    label = f' text=\"{text}\"' if text else ''
    print(f'  {t}: pos=({x},{y}) size={w}x{h} stroke={stroke} bg={bg}{label}')
    if bound: print(f'    boundElements: {json.dumps(bound)}')
    if contained: print(f'    containerId: {contained}')
    if start: print(f'    startBinding: {json.dumps(start)}')
    if end: print(f'    endBinding: {json.dumps(end)}')
"
```

### Step 2: Interpret the layout

Map Excalidraw elements to semantic HTML:

| Excalidraw element | HTML interpretation |
|---|---|
| Rectangle (large, contains others) | Container, section, card wrapper |
| Rectangle (small, inside another) | Card, content block, nav item |
| Rectangle with text | Labeled section, button, heading area |
| Diamond | Decision point, CTA, evaluation step |
| Ellipse | Avatar, icon placeholder, status indicator |
| Arrow (connecting shapes) | Flow or relationship between elements |
| Text (standalone) | Heading, label, paragraph |
| Freedraw | Decorative -- interpret intent, don't reproduce paths |
| Line | Divider, separator |

**Spatial rules:**
- Elements stacked vertically inside a container = column layout (flex-direction: column)
- Elements side by side = row layout (flex-direction: row)
- Element positions (x, y) determine relative placement
- Element sizes (width, height) determine proportions
- Arrow bindings (startBinding, endBinding) show which shapes connect

### Step 3: Generate HTML

- Build a self-contained HTML file with inline CSS
- Match the spatial layout from the drawing
- Add meaningful content that fits the diagram's structure
- Include hover states, transitions, responsive breakpoints
- If the user specifies a brand or site, use those tokens (colors, fonts, spacing)
- Open the file in browser with `open <path>`

### Step 4: Auto-sync reminder

After reading, remind user: edits auto-sync with 1200ms debounce. If they just drew something, wait 2 seconds or click "Sync to Backend" before reading.

## Important Notes

- Canvas server stores everything in memory. Restart = data gone. Always offer to export before stopping.
- `freedraw` elements are raw point arrays -- interpret the intent rather than reproducing the path literally.
- If no text labels exist on shapes, ask the user what each shape represents OR infer from context.
- The official Excalidraw MCP (`excalidraw/excalidraw-mcp`) is write-only. Only `yctimlin/mcp_excalidraw` supports reading drawings back.
