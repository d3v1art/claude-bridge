# Claude Bridge

This project includes a local bridge server that lets you control Figma directly.

## Connecting to Figma

The bridge server runs at **http://localhost:3571**. Before any Figma operation, check its status and start it if needed:

```bash
curl -s http://localhost:3571/status
```

**If the request fails (server not running)** — start it:
```bash
node /Users/cutpixel/Work/GIT/Claude\ Bridge/server/server.js &
```
Then wait 1–2 seconds and check status again.

**If `connected: false`** (server up but Figma plugin not open) — ask the user to open the Claude Bridge plugin in Figma. You cannot proceed without the plugin connected.

**If `connected: true`** — ready, send commands.

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

**Variables & modes:**
- `update_variable` — изменить значение переменной (`variableId`, `values: { modeId: value }`)
- `delete_variable` — удалить переменную (`variableId`)
- `apply_variable` — привязать переменную к свойству ноды (`nodeId`, `property`, `variableId`)
- `detach_variable` — отвязать переменную от свойства (`nodeId`, `property`)
- `get_variable_bindings` — посмотреть все привязки переменных на ноде (`nodeId`)
- `switch_mode` — переключить режим на фрейме (`nodeId`, `collectionId`, `modeId`)
- `reset_mode` — сбросить явный режим, вернуться к родительскому (`nodeId`, `collectionId`)

**Audit:**
- `audit_missing_components` — detached instances без main component
- `audit_hardcoded_colors` — заливки/обводки не из переменных
- `audit_detached_styles` — текстовые слои без text style
- `audit_empty_frames` — фреймы без содержимого
- `audit_contrast` — нарушения контраста WCAG
- `audit_all` — все проверки за один запрос (возвращает сгруппированный отчёт)

**Search:**
- `find_nodes` — find by name/text/type (`name`, `text`, `textContains`, `type`, `scopeId`, `limit`)

**Pages:**
- `get_pages` — list all pages
- `switch_page` — switch to page (`pageId`)
- `create_page` — create new page (`name`, `index?`)
- `delete_page` — delete page (`pageId`)
- `rename_page` — rename page (`pageId`, `name`)

**Viewport & selection:**
- `scroll_to_node` — zoom to node (`nodeId`)
- `set_selection` — select nodes (`nodeIds`)
- `notify` — show toast in Figma (`message`, `error?`)

**Layer order:**
- `bring_to_front` — bring to front (`nodeId`)
- `send_to_back` — send to back (`nodeId`)
- `reorder` — set exact index in parent (`nodeId`, `index`)

**Grouping:**
- `group` — group nodes (`nodeIds`, `name?`, `parentId?`)
- `ungroup` — ungroup (`nodeId`)

**Styles:**
- `get_local_styles` — all local paint/text/effect/grid styles
- `apply_paint_style` — apply color style (`nodeId`, `styleId`, `target?`)
- `apply_text_style` — apply text style to text node (`nodeId`, `styleId`)
- `apply_effect_style` — apply effect style (`nodeId`, `styleId`)
- `create_paint_style` — create color style (`name`, `color`, `opacity?`)
- `create_effect_style` — create effect style (`name`, `effects`)

**Sizing, rotation & blend:**
- `set_sizing` — layout sizing mode (`nodeId`, `axis`, `mode: FIXED|HUG|FILL`)
- `set_blend_mode` — blend mode (`nodeId`, `blendMode`)
- `rotate` — rotation in degrees (`nodeId`, `angle`)
- `set_constraints` — layout constraints (`nodeId`, `horizontal`, `vertical`) — values: `MIN|MAX|STRETCH|CENTER|SCALE`
- `get_constraints` — read constraints (`nodeId`)

**Fills stack:**
- `set_fills` — replace entire fills array (`nodeId`, `fills`)
- `add_fill` — append a fill (`nodeId`, `fill`)
- `remove_fill` — remove fill by index (`nodeId`, `index?`)

**Component instances:**
- `reset_instance` — reset all overrides (`nodeId`)
- `detach_instance` — detach from component, returns frame (`nodeId`)

**Export:**
- `export_svg` — export node as SVG string (`nodeId`)

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
2. `find_nodes` — locate specific elements by name, text, or type
3. `get_tree` — inspect structure of a specific node if needed
4. Make targeted edits with specific actions
5. Use `batch` for multiple changes at once
