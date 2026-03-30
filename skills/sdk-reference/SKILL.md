---
name: sdk-reference
description: Look up Stream Deck SDK documentation, API reference, manifest schema, events, commands, CLI usage, WebSocket API, property inspector components, touch strip layouts, and plugin guidelines. Use when answering questions about the Stream Deck SDK, looking up API details, checking manifest fields, or finding how to use SDK features.
user-invocable: false
---

# Stream Deck SDK v2 Reference

You have access to the complete Elgato Stream Deck SDK v2 documentation in the `docs/` directory of this plugin (relative to this skill: `${CLAUDE_SKILL_DIR}/../..`). Use these files to answer questions accurately.

## Documentation Index

Read the relevant file(s) from the docs/ directory based on the user's question:

### Introduction
- `docs/01-getting-started.md` — Prerequisites, CLI setup, scaffolding, file structure
- `docs/02-your-first-changes.md` — Modifying actions, hot-reload, debugging
- `docs/03-plugin-environment.md` — Architecture (Node.js backend + Chromium frontend), runtime versions, manifest structure, lifecycle
- `docs/04-distribution.md` — DRM, packaging with `streamdeck pack`, publishing to Marketplace

### Plugin Guides
- `docs/05-actions.md` — Action types (Key/Dial), UUIDs, registering actions, handling events, accessing visible actions, full events + commands reference
- `docs/06-keys.md` — Key actions, states, multi-state toggles, titles, images, setImage/setTitle commands
- `docs/07-dials.md` — Dial/encoder actions (SD+), touch strip layouts ($X1,$A0,$A1,$B1,$B2,$C1), custom layouts, setFeedback, trigger descriptions
- `docs/08-settings.md` — Action settings vs global settings, reading/writing, type safety, Zod validation, security considerations
- `docs/09-property-inspectors.md` — HTML UI panels, sdpi-components library, all UI components (textfield, select, checkbox, range, etc.)
- `docs/10-devices.md` — Device enumeration, device types, connection events
- `docs/11-profiles.md` — Bundled profiles, switching profiles, device types for profiles
- `docs/12-embedded-resources.md` — getResources/setResources, embedding files with actions
- `docs/13-system.md` — System events, openUrl, platform info
- `docs/14-deep-linking.md` — Deep-link URLs (`streamdeck://...`), receiving deep-links in plugins
- `docs/15-app-monitoring.md` — ApplicationsToMonitor, detecting when apps launch/quit
- `docs/16-secrets.md` — Plugin secrets API for DRM-protected plugins
- `docs/17-logging.md` — streamDeck.logger API, log levels, log file locations
- `docs/18-localization.md` — i18n, translation files, localized strings

### Style Guide
- `docs/19-code-linting.md` — ESLint config for SD plugins
- `docs/20-plugin-guidelines.md` — Marketplace submission guidelines, UX requirements

### References
- `docs/21-manifest.md` — **Complete manifest.json reference** — all fields, TypeScript type declaration, JSON schema URL, examples (basic, dial, profiles, app monitoring)
- `docs/22-touch-strip-layout.md` — Custom layout JSON format, layout items, canvas coordinates
- `docs/23-changelog.md` — SDK version history and changes

### WebSocket API (low-level)
- `docs/25-websocket-plugin.md` — Raw WebSocket events sent/received by the plugin process
- `docs/26-websocket-ui.md` — Raw WebSocket events for property inspectors
- `docs/27-websocket-changelog.md` — WebSocket protocol version history

### CLI Reference
- `docs/30-cli-intro.md` — Stream Deck CLI overview
- `docs/31-cli-create.md` — `streamdeck create` command
- `docs/32-cli-config.md` — `streamdeck config` command
- `docs/33-cli-dev.md` — `streamdeck dev` command
- `docs/34-cli-link.md` — `streamdeck link` command
- `docs/35-cli-list.md` — `streamdeck list` command
- `docs/36-cli-pack.md` — `streamdeck pack` command
- `docs/37-cli-restart.md` — `streamdeck restart` command
- `docs/38-cli-stop.md` — `streamdeck stop` command
- `docs/39-cli-unlink.md` — `streamdeck unlink` command
- `docs/40-cli-validate.md` — `streamdeck validate` command

### Upgrading
- `docs/24-upgrading-v2.md` — Migration guide from SDK v1 to v2

## Quick Reference

### Manifest TypeScript Type (Key Fields)

```typescript
type Manifest = {
    Actions: {
        Controllers?: ["Encoder" | "Keypad", ("Encoder" | "Keypad")?];
        DisableAutomaticStates?: boolean;
        Encoder?: {
            Icon?: string;
            layout?: string; // "$A0" | "$A1" | "$B1" | "$B2" | "$C1" | "$X1" | custom .json
            TriggerDescription?: { LongTouch?: string; Push?: string; Rotate?: string; Touch?: string; };
        };
        Icon: string;
        Name: string;
        PropertyInspectorPath?: string;
        States: { Image: string; Name?: string; ShowTitle?: boolean; TitleAlignment?: "bottom"|"middle"|"top"; }[];
        Tooltip?: string;
        UUID: string;
        VisibleInActionsList?: boolean;
    }[];
    Author: string;
    Category?: string;
    CategoryIcon?: string;
    CodePath: string;
    Description: string;
    Icon: string;
    Name: string;
    Nodejs?: { Debug?: string; Version: "20" | "24"; };
    OS: { MinimumVersion: string; Platform: "mac" | "windows"; }[];
    Profiles?: { AutoInstall?: boolean; DeviceType: number; Name: string; Readonly?: boolean; }[];
    SDKVersion: 2 | 3;
    Software: { MinimumVersion: string; };
    UUID: string;
    Version: string;  // format: "X.Y.Z.W"
};
```

### JSON Schema URL
```
https://schemas.elgato.com/streamdeck/plugins/manifest.json
```

### SingletonAction Events

| Event | Description |
|-------|-------------|
| onWillAppear | Action appears on canvas |
| onWillDisappear | Action leaves canvas |
| onKeyDown | Key pressed |
| onKeyUp | Key released |
| onDialDown | Dial pressed (SD+) |
| onDialUp | Dial released (SD+) |
| onDialRotate | Dial rotated (SD+) |
| onTouchTap | Touch strip tapped (SD+) |
| onDidReceiveSettings | Settings changed from UI |
| onDidReceiveResources | Resources updated |
| onPropertyInspectorDidAppear | PI opened |
| onPropertyInspectorDidDisappear | PI closed |
| onSendToPlugin | Message from PI |
| onTitleParametersDidChange | User changed title settings |

### Action Commands

| Command | Description |
|---------|-------------|
| setTitle(title) | Set key title |
| setImage(image) | Set key image |
| setState(state) | Set action state index |
| setSettings(settings) | Persist settings |
| getSettings() | Retrieve settings |
| setResources(resources) | Set embedded resources |
| getResources() | Get embedded resources |
| showAlert() | Flash warning icon |
| showOk() | Flash checkmark |
| setFeedback(feedback) | Update touch strip (SD+) |
| setFeedbackLayout(layout) | Change touch strip layout (SD+) |
| setTriggerDescription(desc) | Set dial hints (SD+) |
| sendToPropertyInspector(payload) | Send message to PI |

### Global API (streamDeck.*)

| Namespace | Key Methods |
|-----------|-------------|
| streamDeck.actions | registerAction(), forEach(), onWillAppear/etc. |
| streamDeck.devices | forEach(), onDeviceDidConnect/Disconnect |
| streamDeck.settings | getGlobalSettings(), setGlobalSettings(), onDidReceiveGlobalSettings() |
| streamDeck.system | openUrl(), onDidReceiveDeepLink(), onApplicationDidLaunch/Terminate |
| streamDeck.profiles | switchToProfile() |
| streamDeck.logger | info(), warn(), error(), debug(), trace() |
| streamDeck.connect() | Start the plugin (call after registering all actions) |

## Official Sample Plugins

Real-world examples from the `docs/samples/` directory:

### Cat Keys (Key Action + Property Inspector + Fetch API)
- `docs/samples/cat-keys-action.ts` — RandomCat action: fetches a cat image from API, sets it as key image, auto-updates with timer
- `docs/samples/cat-keys-plugin.ts` — Plugin entry: sets log level, registers action, connects
- `docs/samples/cat-keys-manifest.json` — Manifest with PropertyInspectorPath, Keypad controller
- `docs/samples/cat-keys-package.json` — Package.json with exact dependency versions
- `docs/samples/cat-keys-rollup.config.mjs` — Rollup config with commonjs, terser, module package emit
- `docs/samples/cat-keys-tsconfig.json` — tsconfig extending @tsconfig/node20
- `docs/samples/cat-keys-ui.html` — Property inspector with sdpi-checkbox

### Layouts (Dial/Encoder Actions for Stream Deck +)
- `docs/samples/layouts-dial-action.ts` — BuiltInLayout action: onDialRotate, setFeedbackLayout, setFeedback with all built-in layouts
- `docs/samples/layouts-manifest.json` — Manifest with Encoder controllers, layout definitions, TriggerDescription

### SDK Source (TypeScript API)
- `docs/samples/sdk-singleton-action.ts` — Full SingletonAction class with all event method signatures
- `docs/samples/sdk-exports.ts` — Main SDK exports: streamDeck object with all namespaces (actions, devices, i18n, info, logger, profiles, settings, system, ui, connect)

## How to Use This Reference

When the user asks about any Stream Deck SDK topic:
1. Identify which doc file(s) are relevant from the index above
2. Read the specific file(s) from the docs/ directory
3. Answer using the actual documentation content
4. Include code examples from the docs when helpful
