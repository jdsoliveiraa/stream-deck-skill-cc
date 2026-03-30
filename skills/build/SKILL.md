---
name: build
description: Build, pack, validate, or prepare a Stream Deck plugin for distribution. Use when building, packaging, validating, or distributing a Stream Deck plugin.
argument-hint: [build|pack|validate|release]
disable-model-invocation: true
allowed-tools: Bash, Read, Edit, Glob, Grep
---

# Build / Pack / Validate a Stream Deck Plugin

Manage the build lifecycle of a Stream Deck plugin project.

## Determine the Command

Based on $ARGUMENTS (default: `build`):

- **build** — Compile TypeScript and bundle with Rollup
- **pack** — Package the plugin into a `.streamDeckPlugin` installer file
- **validate** — Validate the manifest and plugin structure
- **release** — Full pipeline: validate, build, pack

## Build

Compile the plugin source:

```bash
npm run build
```

This runs `rollup -c` which:
1. Compiles TypeScript from `src/`
2. Bundles into `*.sdPlugin/bin/plugin.js`
3. Resolves Node.js dependencies

### Watch Mode (Development)

For live-reload during development:

```bash
npm run watch
```

This watches for changes in `src/` and `manifest.json`, rebuilds, and restarts the plugin in Stream Deck automatically.

## Validate

Run the Stream Deck CLI validator:

```bash
streamdeck validate <UUID>.sdPlugin
```

This checks:
- manifest.json structure and required fields
- Action UUID format and uniqueness
- Icon file references
- OS compatibility declarations
- SDK version compatibility

### Manual Validation Checklist

Also verify:
1. **manifest.json** has `$schema` set to `https://schemas.elgato.com/streamdeck/plugins/manifest.json`
2. **Version** format is `X.Y.Z.W` (four segments)
3. **All action UUIDs** are prefixed by the plugin UUID
4. **Icon paths** in manifest don't include file extensions
5. **Icon files** exist at referenced paths (with `.png` extension)
6. **Required icon sizes**:
   - Action icon: 20x20 (@1x), 40x40 (@2x)
   - Key image: 72x72 (@1x), 144x144 (@2x)
   - Category icon: 28x28 (@1x), 56x56 (@2x)
   - Marketplace icon: 288x288
7. **CodePath** points to a valid compiled JS file
8. **All actions registered** in `src/plugin.ts` before `streamDeck.connect()`

## Pack

Package for distribution:

```bash
streamdeck pack <UUID>.sdPlugin
```

This:
1. Validates the plugin
2. Bundles the `.sdPlugin` directory contents
3. Outputs a `.streamDeckPlugin` installer file

### Ignoring Files

Create a `.sdignore` file (same syntax as `.gitignore`) to exclude files from the package:

```
# .sdignore
*.map
*.ts
node_modules/
logs/
.DS_Store
```

## Release Pipeline

Full release sequence:

```bash
# 1. Validate
streamdeck validate <UUID>.sdPlugin

# 2. Build production
npm run build

# 3. Pack
streamdeck pack <UUID>.sdPlugin

# 4. Test the packaged plugin
# Double-click the .streamDeckPlugin file to install in Stream Deck
```

### DRM Protection (Marketplace)

For plugins distributed on the Elgato Marketplace:
- Set `SDKVersion` to `3` in manifest
- Set `Software.MinimumVersion` to `"6.9"` or higher
- Use `@elgato/streamdeck` v2+
- Upload to Maker Console (DRM applied during processing)
- Don't modify distributed files at runtime (integrity checking)
- Don't read manifest.json at runtime (it's a protected asset)

## Troubleshooting Build Issues

| Issue | Fix |
|-------|-----|
| `Cannot find module '@elgato/streamdeck'` | Run `npm install` |
| Plugin doesn't appear in SD | Restart Stream Deck app, check elevated privileges |
| Changes not reflecting | Use `streamdeck restart <uuid>` or `npm run watch` |
| `streamdeck` command not found | Run `npm install -g @elgato/cli` |
| Validation errors | Run `streamdeck validate` and fix reported issues |
| Icon not showing | Check path in manifest has no extension, file exists as .png |
