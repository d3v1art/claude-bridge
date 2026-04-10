# Claude Bridge

This project includes a local bridge server that lets you control Figma directly.

## Connecting to Figma

If the user wants to read or edit their Figma file, use the bridge:

**Check connection first:**
```bash
curl http://localhost:3571/status
```

If `connected: true` — send commands. If not — ask the user to start the server (`node server/server.js`) and open the Claude Bridge plugin in Figma.

**Send a command:**
```bash
curl -s -X POST http://localhost:3571/command \
  -H "Content-Type: application/json" \
  -d '{"action": "ACTION_NAME", ...params}'
```

## Available actions

**Reading:**
- `get_selection` — what's selected right now
- `get_node` — node by ID (`nodeId`)
- `get_children` — direct children (`nodeId`)
- `get_tree` — node subtree (`nodeId`, `depth`)
- `get_page_nodes` — top-level nodes on current page
- `get_text` — text content (`nodeId`)
- `get_text_style` — typography (`nodeId`)
- `get_fills` — fill colors with parent chain (`nodeId`)
- `get_variables` — all local variable collections
- `get_local_components` — all components and component sets
- `find_all_instances` — all component instances
- `get_all_texts` — all text nodes in scope (`scopeId`)
- `get_screenshot` — export as base64 PNG (`nodeId`, `scale`)
- `audit_contrast` — WCAG contrast failures (`scopeId`)

**Editing:**
- `rename` — rename node (`nodeId`, `name`)
- `move` — set position (`nodeId`, `x`, `y`)
- `resize` — set size (`nodeId`, `width`, `height`)
- `set_visible` — show/hide (`nodeId`, `visible`)
- `set_text` — update text (`nodeId`, `text`)
- `set_fill` — set fill color (`nodeId`, `color: {r,g,b}`, `opacity`)
- `set_stroke` — set stroke (`nodeId`, `color`, `weight`)
- `set_corner_radius` — corner radius (`nodeId`, `radius`)
- `set_opacity` — opacity (`nodeId`, `opacity`)
- `set_font` — typography (`nodeId`, `family`, `style`, `size`)
- `set_layout` — auto-layout (`nodeId`, `mode`, `gap`, `padding`)
- `set_effect` — add shadow/effect (`nodeId`, `effectType`, ...)

**Creating:**
- `create_frame` — new frame (`name`, `width`, `height`, `x`, `y`, `fill`, `cornerRadius`)
- `create_rectangle` — new rectangle
- `create_text` — new text layer (`text`, `fontSize`, `x`, `y`, `fill`)
- `create_instance` — instantiate component (`componentId`, `parentId`, `x`, `y`)
- `create_component_from_node` — convert node to component (`nodeId`)
- `duplicate` — clone node (`nodeId`, `x`, `y`)

**Structure:**
- `reparent` — move to new parent (`nodeId`, `newParentId`)
- `delete_node` — delete (`nodeId`)
- `batch` — multiple commands in one call (`commands: [{action, ...params}]`)

## Colors

Colors are passed as normalized floats `{r, g, b}` in range 0–1, not 0–255.

```json
{ "r": 1, "g": 0, "b": 0 }  // red
{ "r": 0.2, "g": 0.6, "b": 1 }  // blue
```

## Typical workflow

1. `get_selection` or `get_page_nodes` — understand what's on the canvas
2. `get_tree` with `depth: 2-3` — explore structure of a specific frame
3. Make targeted edits with specific actions
4. Use `batch` for multiple changes at once
