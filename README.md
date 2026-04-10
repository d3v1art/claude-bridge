# Claude Bridge

A Figma plugin that connects Claude (or any external tool) to your Figma file via a local bridge server. Send commands over HTTP — Claude reads and edits your design in real time.

## How it works

```
Claude / external tool
        |
  HTTP POST /command
        |
  Bridge Server (localhost:3571)
        |
    WebSocket
        |
  Figma Plugin (Claude Bridge)
        |
    Figma API
```

1. The **bridge server** runs locally and exposes a small HTTP API.
2. The **Figma plugin** connects to it over WebSocket and stays open while you work.
3. Any tool (Claude Code, scripts, etc.) sends a JSON command to the server — the plugin executes it inside Figma and returns the result.

## Setup

### 1. Start the bridge server

```bash
cd server
npm install
node server.js
```

Server starts on `http://localhost:3571`.

### 2. Load the plugin in Figma

In Figma desktop: **Plugins → Development → Import plugin from manifest** → select `manifest.json`.

Run the plugin — it will connect automatically and show a green dot when ready.

### 3. Send commands

```bash
# Check connection status
curl http://localhost:3571/status

# Send a command
curl -X POST http://localhost:3571/command \
  -H "Content-Type: application/json" \
  -d '{"action": "get_selection"}'
```

## API

`POST /command` — send any action to Figma. Body is a JSON object with an `action` field and optional params.

`GET /status` — returns `{"connected": true/false}`.

### Supported actions

| Action | Description |
|--------|-------------|
| `get_selection` | Get currently selected nodes |
| `get_node` | Get node by ID |
| `get_children` | Get direct children of a node |
| `get_tree` | Get node tree (with depth control) |
| `get_page_tree` | Get full page tree |
| `get_page_nodes` | Get top-level nodes on current page |
| `get_parent` | Get parent chain of a node |
| `get_text` | Get text content of a text node |
| `get_text_style` | Get typography style of a text node |
| `get_fills` | Get fill colors (with parent chain) |
| `get_variables` | Get all local variable collections |
| `get_variable` | Get a single variable by ID |
| `get_local_components` | List all components and component sets |
| `find_all_instances` | Find all component instances |
| `get_all_texts` | Get all text nodes in a scope |
| `get_screenshot` | Export a node as base64 PNG |
| `get_annotations` | Read plugin/shared data from a node |
| `audit_contrast` | Find WCAG contrast failures in a scope |
| `rename` | Rename a node |
| `move` | Set absolute position |
| `move_by` | Move relative (dx/dy) |
| `resize` | Set width/height |
| `set_visible` | Show/hide a node |
| `set_text` | Update text content |
| `set_fill` | Set solid fill color |
| `set_stroke` | Set stroke color and weight |
| `set_corner_radius` | Set corner radius |
| `set_opacity` | Set opacity |
| `set_effect` | Add drop shadow or other effect |
| `set_font` | Set font family, size, line height |
| `set_layout` | Configure auto-layout on a frame |
| `create_frame` | Create a new frame |
| `create_rectangle` | Create a rectangle |
| `create_text` | Create a text layer |
| `create_text_style` | Create a local text style |
| `create_instance` | Instantiate a component |
| `create_component_from_node` | Convert a node into a component |
| `combine_as_variants` | Combine components into a variant set |
| `add_component_property` | Add a property to a component |
| `set_property_reference` | Link a layer to a component property |
| `get_component_properties` | Get component property definitions |
| `create_variable_collection` | Create a variable collection |
| `create_variable` | Create a variable in a collection |
| `duplicate` | Clone a node |
| `reparent` | Move a node to a different parent |
| `delete_node` | Delete a node |
| `batch` | Execute multiple commands in one call |

### Example: create a frame with text

```json
{
  "action": "batch",
  "commands": [
    {
      "action": "create_frame",
      "name": "Card",
      "width": 320,
      "height": 200,
      "x": 0,
      "y": 0,
      "fill": { "r": 0.97, "g": 0.97, "b": 0.97 },
      "cornerRadius": 12
    },
    {
      "action": "create_text",
      "text": "Hello from Claude",
      "fontSize": 18,
      "x": 24,
      "y": 24
    }
  ]
}
```

## Development

```bash
npm install
npm run build      # build plugin once
npm run watch      # rebuild on changes
```

Built with [Svelte](https://svelte.dev/) (plugin UI) and [esbuild](https://esbuild.github.io/) (plugin code).
