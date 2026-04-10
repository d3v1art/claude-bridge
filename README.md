# Claude Bridge

Figma plugin that gives Claude direct access to your Figma file — read and edit designs through natural language.

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

### 3. Talk to Claude

Open Claude Code in this project folder. Claude will read `CLAUDE.md` automatically and know how to use the bridge.

Just ask:
> "What's on the current page?"
> "Find all detached components"
> "Change the button text to Sign Up"

## How it works

```
Claude Code → HTTP → Bridge Server (:3571) → WebSocket → Figma Plugin → Figma API
```

## Development

```bash
npm install
npm run build   # build once
npm run watch   # rebuild on changes
```
