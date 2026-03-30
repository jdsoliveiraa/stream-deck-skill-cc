---
name: create-plugin
description: Scaffold a new Stream Deck plugin project from scratch. Use when creating a new Stream Deck plugin, starting a new SD project, or bootstrapping a plugin.
argument-hint: <plugin-uuid> [plugin-name]
disable-model-invocation: true
allowed-tools: Bash, Write, Edit, Read, Glob, Grep
---

# Create a New Stream Deck Plugin

You are creating a new Stream Deck plugin from scratch using the Elgato Stream Deck SDK v2.

## Prerequisites Check

First verify the user has the required tools installed:

```bash
node --version  # Must be v20+
streamdeck --version  # Must have @elgato/cli installed
```

If `streamdeck` is not found, install it:
```bash
npm install -g @elgato/cli
```

## Plugin Details

The user wants to create a plugin with UUID: **$ARGUMENTS**

If no UUID was provided, ask the user for:
1. **Plugin UUID** — reverse-DNS format (e.g., `com.mycompany.my-plugin`). Only lowercase alphanumeric (a-z, 0-9), hyphens (-), and periods (.). Cannot change after publication.
2. **Plugin name** — human-readable display name
3. **Author name**
4. **Brief description** of what the plugin does
5. **What actions** should the plugin include initially?

## Scaffolding

Use the Stream Deck CLI to scaffold:

```bash
streamdeck create
```

This runs an interactive wizard. If you cannot run the interactive wizard, create the project manually with this structure:

### Directory Structure

```
<plugin-name>/
├── <uuid>.sdPlugin/
│   ├── bin/                    # Compiled output (generated)
│   ├── imgs/
│   │   ├── actions/
│   │   │   └── <action>/
│   │   │       ├── icon.png        # 20x20 action list icon (@1x)
│   │   │       ├── icon@2x.png     # 40x40 action list icon (@2x)
│   │   │       ├── key.png         # 72x72 key default image (@1x)
│   │   │       └── key@2x.png      # 144x144 key default image (@2x)
│   │   └── plugin/
│   │       ├── category-icon.png       # 28x28 (@1x)
│   │       ├── category-icon@2x.png    # 56x56 (@2x)
│   │       └── marketplace.png         # 288x288 marketplace icon
│   ├── logs/                   # Runtime logs
│   ├── ui/                     # Property inspector HTML files
│   └── manifest.json           # Plugin manifest
├── src/
│   ├── actions/                # Action TypeScript files
│   └── plugin.ts               # Entry point
├── package.json
├── rollup.config.mjs
└── tsconfig.json
```

### manifest.json

Create with this template (adapt to the user's needs):

```json
{
    "$schema": "https://schemas.elgato.com/streamdeck/plugins/manifest.json",
    "UUID": "<PLUGIN_UUID>",
    "Name": "<PLUGIN_NAME>",
    "Version": "0.1.0.0",
    "Author": "<AUTHOR>",
    "Description": "<DESCRIPTION>",
    "Icon": "imgs/plugin/marketplace",
    "CategoryIcon": "imgs/plugin/category-icon",
    "Category": "<PLUGIN_NAME>",
    "CodePath": "bin/plugin.js",
    "SDKVersion": 2,
    "Software": {
        "MinimumVersion": "6.9"
    },
    "OS": [
        {
            "Platform": "mac",
            "MinimumVersion": "10.15"
        },
        {
            "Platform": "windows",
            "MinimumVersion": "10"
        }
    ],
    "Nodejs": {
        "Version": "20",
        "Debug": "enabled"
    },
    "Actions": []
}
```

### src/plugin.ts (Entry Point)

```typescript
import streamDeck from "@elgato/streamdeck";

// Register actions here (import and register each action)

streamDeck.connect();
```

### package.json

```json
{
    "scripts": {
        "build": "rollup -c",
        "watch": "rollup -c -w --watch.onEnd=\"streamdeck restart <PLUGIN_UUID>\""
    },
    "type": "module",
    "devDependencies": {
        "@elgato/cli": "^1.7.0",
        "@rollup/plugin-commonjs": "^28.0.2",
        "@rollup/plugin-node-resolve": "^16.0.0",
        "@rollup/plugin-terser": "^0.4.4",
        "@rollup/plugin-typescript": "^12.1.2",
        "@tsconfig/node20": "^20.1.4",
        "@types/node": "22.13.0",
        "rollup": "^4.32.1",
        "tslib": "^2.8.1",
        "typescript": "^5.7.3"
    },
    "dependencies": {
        "@elgato/streamdeck": "^2.0.1"
    }
}
```

### rollup.config.mjs

```javascript
import commonjs from "@rollup/plugin-commonjs";
import nodeResolve from "@rollup/plugin-node-resolve";
import terser from "@rollup/plugin-terser";
import typescript from "@rollup/plugin-typescript";
import path from "node:path";
import url from "node:url";

const isWatching = !!process.env.ROLLUP_WATCH;
const sdPlugin = "<UUID>.sdPlugin";

/**
 * @type {import('rollup').RollupOptions}
 */
const config = {
    input: "src/plugin.ts",
    output: {
        file: `${sdPlugin}/bin/plugin.js`,
        sourcemap: isWatching,
        sourcemapPathTransform: (relativeSourcePath, sourcemapPath) => {
            return url.pathToFileURL(path.resolve(path.dirname(sourcemapPath), relativeSourcePath)).href;
        }
    },
    plugins: [
        {
            name: "watch-externals",
            buildStart: function () {
                this.addWatchFile(`${sdPlugin}/manifest.json`);
            },
        },
        typescript({
            mapRoot: isWatching ? "./" : undefined
        }),
        nodeResolve({
            browser: false,
            exportConditions: ["node"],
            preferBuiltins: true
        }),
        commonjs(),
        !isWatching && terser(),
        {
            name: "emit-module-package-file",
            generateBundle() {
                this.emitFile({ fileName: "package.json", source: `{ "type": "module" }`, type: "asset" });
            }
        }
    ]
};

export default config;
```

### tsconfig.json

```json
{
    "extends": "@tsconfig/node20/tsconfig.json",
    "compilerOptions": {
        "customConditions": ["node"],
        "noImplicitOverride": true,
        "module": "ES2022",
        "moduleResolution": "Bundler"
    },
    "include": ["src/**/*.ts"],
    "exclude": ["node_modules"]
}
```

## After Scaffolding

1. Run `npm install` to install dependencies
2. Suggest the user add their first action with `/stream-deck:add-action`
3. Run `npm run watch` to start development

## Important SDK Conventions

- Action UUIDs must be prefixed by the plugin UUID (e.g., `com.example.myplugin.my-action`)
- Action UUIDs: lowercase alphanumeric, hyphens, and periods only
- Icon paths in manifest omit file extensions (Stream Deck adds `.png` / `@2x.png` automatically)
- `CodePath` points to the compiled JS entry (usually `bin/plugin.js`)
- `SDKVersion: 2` enables DRM support and latest features
- `Software.MinimumVersion: "6.9"` is recommended minimum for SDK v2
- Node.js version should be `"20"` (or `"24"` for SD 7.3+)
- Version format is `X.Y.Z.W` (four segments)

For full SDK reference, see the docs/ directory in this plugin.
