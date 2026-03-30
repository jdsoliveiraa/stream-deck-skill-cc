---
name: dev-mode
description: Start Stream Deck plugin development mode with hot-reload, debugging, and live testing. Use when starting development, debugging, or testing a Stream Deck plugin.
argument-hint: [start|debug|restart|stop|link|unlink|list]
disable-model-invocation: true
allowed-tools: Bash, Read, Glob, Grep
---

# Stream Deck Plugin Development Mode

Manage the development lifecycle of your Stream Deck plugin.

## Commands

Based on $ARGUMENTS (default: `start`):

### start — Start Development with Hot-Reload

```bash
npm run watch
```

This watches `src/` and `manifest.json` for changes, auto-rebuilds, and restarts the plugin in Stream Deck. Press Ctrl+C to stop.

### debug — Attach a Debugger

Stream Deck plugins support Node.js debugging via the inspector protocol.

**VS Code:**
1. Open Quick Open (Cmd+P / Ctrl+P)
2. Type `> Debug: Attach to Node Process`
3. Select the `node20` process

**Or configure in manifest** (fixed port):
```json
{
    "Nodejs": {
        "Version": "20",
        "Debug": "12345"
    }
}
```

Then use VS Code launch config:
```json
{
    "type": "node",
    "request": "attach",
    "name": "Attach to Stream Deck Plugin",
    "port": 12345
}
```

By default, Stream Deck auto-assigns an available port. Set `"Debug": "enabled"` for auto-assignment, or a port number for a fixed port.

### restart — Restart the Plugin

```bash
streamdeck restart <plugin-uuid>
```

Useful after manual builds or manifest changes.

### stop — Stop the Plugin

```bash
streamdeck stop <plugin-uuid>
```

### link — Link a Development Plugin

```bash
streamdeck link <path-to-sdPlugin-directory>
```

Creates a symlink from your development `.sdPlugin` directory to the Stream Deck plugins folder. This is how the scaffolded project works — your dev directory IS the installed plugin.

### unlink — Unlink a Development Plugin

```bash
streamdeck unlink <plugin-uuid>
```

Removes the symlink without deleting your source files.

### list — List Installed Plugins

```bash
streamdeck list
```

Shows all installed plugins (both linked development plugins and installed ones).

### dev — Enable Developer Mode

```bash
streamdeck dev
```

Toggles developer mode in Stream Deck, enabling additional debugging features.

### config — View Configuration

```bash
streamdeck config
```

Shows the Stream Deck CLI configuration (plugins directory path, etc.).

## Plugin Logs

Stream Deck plugin logs are written to:

```
<uuid>.sdPlugin/logs/
```

Use the `streamDeck.logger` in your plugin code:

```typescript
import streamDeck from "@elgato/streamdeck";

streamDeck.logger.info("Plugin started");
streamDeck.logger.warn("Something unusual");
streamDeck.logger.error("An error occurred", error);
streamDeck.logger.debug("Debug info");
streamDeck.logger.trace("Detailed trace");
```

Default log level is `INFO`. Set via manifest or environment for more verbose output.

## Development Tips

1. **Use `npm run watch`** — it's the fastest development loop
2. **Check logs** when something doesn't work — they're in `*.sdPlugin/logs/`
3. **Use `streamDeck.logger`** — not `console.log` (which goes to stdout, not the log file)
4. **Debug with VS Code** — attach to the Node.js process for breakpoints
5. **Restart Stream Deck app** if the plugin disappears after an update
6. **Use two states** for toggle actions — SD auto-toggles for you
7. **Test on actual hardware** — simulators don't cover all edge cases
