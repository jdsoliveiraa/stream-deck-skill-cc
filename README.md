# stream-deck

A Claude Code plugin for building Elgato Stream Deck plugins with the SDK v2.

## What it does

Provides skills, an agent, and hooks to guide you through the full Stream Deck plugin development lifecycle — from scaffolding to publishing on the Elgato Marketplace.

## Skills

| Skill | Command | Description |
|-------|---------|-------------|
| **create-plugin** | `/stream-deck:create-plugin <uuid>` | Scaffold a new Stream Deck plugin project |
| **add-action** | `/stream-deck:add-action <name> [type]` | Add actions (key, dial, toggle) to an existing plugin |
| **add-property-inspector** | `/stream-deck:add-property-inspector <action>` | Create settings UI with sdpi-components |
| **build** | `/stream-deck:build [build\|pack\|validate\|release]` | Build, package, validate, or prepare for release |
| **dev-mode** | `/stream-deck:dev-mode [start\|debug\|restart\|stop]` | Development mode with hot-reload and debugging |
| **sdk-reference** | *(auto-loaded)* | Claude automatically consults the full SDK docs when relevant |

## Agent

**stream-deck-dev** — A specialized agent with deep knowledge of the Stream Deck SDK v2. Preloaded with the sdk-reference skill for accurate code generation.

## Bundled Documentation

- 38 pages of official Elgato SDK v2 documentation (introduction, guides, references, CLI, WebSocket API)
- 11 real-world sample files from Elgato's official `streamdeck` and `streamdeck-plugin-samples` repositories
- Complete manifest TypeScript type definition and JSON schema reference

## Installation

### From the Anthropic marketplace (Claude Code CLI)

```
/plugin install stream-deck
```

### From the Anthropic marketplace (Claude.ai / Desktop app)

1. Go to **Settings > Plugins**
2. Search for `stream-deck`
3. Click **Install**

### From GitHub (manual)

```bash
git clone https://github.com/jdsoliveiraa/stream-deck-skill-cc.git
claude --plugin-dir ./stream-deck-skill-cc
```

### Local development

If you cloned the repo locally, pass it directly:

```bash
claude --plugin-dir /path/to/stream-deck-skill-cc
```

## Requirements

For plugin development (not for this Claude Code plugin itself):

- [Node.js](https://nodejs.org/) v20+
- [Stream Deck](https://www.elgato.com/downloads) v6.9+
- Stream Deck CLI: `npm install -g @elgato/cli`
- A Stream Deck device (or [Stream Deck Mobile](https://www.elgato.com/stream-deck-mobile))

## Quick Start

1. Install the plugin (see [Installation](#installation) above)
2. Run `/stream-deck:create-plugin com.yourname.my-plugin`
3. Follow the guided scaffolding
4. Run `/stream-deck:add-action my-first-action` to add functionality
5. Run `/stream-deck:dev-mode start` to begin development with hot-reload

## License

MIT
