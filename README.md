# Claude Bridge

Figma plugin that gives Claude direct access to your Figma file — read and edit designs through natural language.

## How it works

Claude uses **two complementary modes** to work with Figma:

```
┌─────────────────────────────────────────────────────────┐
│                      Claude Code                        │
├────────────────────────┬────────────────────────────────┤
│   Bridge (edits)       │   Figma MCP (builds)           │
│   curl → :3571         │   use_figma (JS in plugin)     │
│   Best for: targeted   │   Best for: creating screens   │
│   edits, variables,    │   and components from scratch  │
│   reading state        │   in one round-trip            │
└────────────┬───────────┴──────────────┬─────────────────┘
             │                          │
             ▼                          ▼
    Bridge Server (:3571)      Figma MCP (mcp.figma.com)
             │                          │
             └──────────┬───────────────┘
                        ▼
                  Figma Plugin API
```

## Setup

### 1. Start the bridge server

```bash
cd server
npm install
node server.js
```

### 2. Load the plugin in Figma

Figma Desktop → **Plugins → Development → Import plugin from manifest** → select `manifest.json`.

Run the plugin — green dot means connected.

### 3. Connect Figma MCP (one-time)

The Figma MCP is pre-configured. On first use Claude will open an OAuth browser flow — no token needed, just approve in your Figma account. After that it's persistent.

### 4. Talk to Claude

Open Claude Code in this project folder. Claude reads `CLAUDE.md` automatically.

```
"Create an auth screen with dark mode support"
"Change all button labels to Sign Up"
"Audit contrast on the current page"
"Add a tablet variant of the dashboard"
```

## When Claude uses which mode

| Task | Mode used |
|------|-----------|
| Build a new screen from scratch | `use_figma` (MCP) |
| Create components with variants | `use_figma` (MCP) |
| Change text / color / variable | Bridge |
| Read structure, audit, inspect | Bridge |
| Apply variables or styles | Bridge |
| Switch dark/light mode on a frame | Bridge |

Both modes target the same Figma file and are fully compatible — Claude switches between them automatically based on the task.

## Development

```bash
npm install
npm run build   # build once
npm run watch   # rebuild on changes
```
