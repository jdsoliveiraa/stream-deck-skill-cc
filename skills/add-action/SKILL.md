---
name: add-action
description: Add a new action to an existing Stream Deck plugin. Use when adding actions, buttons, keys, dials, or new functionality to a Stream Deck plugin.
argument-hint: <action-name> [type:key|dial|both]
allowed-tools: Bash, Write, Edit, Read, Glob, Grep
---

# Add a New Action to a Stream Deck Plugin

You are adding a new action to an existing Stream Deck plugin project.

## Gather Information

First, read the existing `manifest.json` to understand the plugin's UUID and current actions:

1. Find the manifest: look for `*.sdPlugin/manifest.json`
2. Read `src/plugin.ts` to see how existing actions are registered
3. Read any existing actions in `src/actions/` for patterns

Then determine from the user (or from $ARGUMENTS):
- **Action name** — human-readable (e.g., "Toggle Mute")
- **Action identifier** — will be `<plugin-uuid>.<action-slug>` (e.g., `com.example.plugin.toggle-mute`)
- **Controller type** — `Keypad` (key/button), `Encoder` (dial on SD+), or both
- **What should the action do?** — what happens on press/rotation/etc.
- **Does it need settings?** — configurable options for the user
- **Number of states** — 1 (default) or 2 (for toggle actions like on/off, mute/unmute)

## Step 1: Create the Action TypeScript File

Create `src/actions/<action-slug>.ts`:

### For a Key Action (Keypad):

```typescript
import streamDeck, { action, KeyDownEvent, SingletonAction, WillAppearEvent } from "@elgato/streamdeck";

@action({ UUID: "<PLUGIN_UUID>.<ACTION_SLUG>" })
export class <ActionClassName> extends SingletonAction<Settings> {
    /**
     * Called when the action appears on the Stream Deck canvas.
     */
    override onWillAppear(ev: WillAppearEvent<Settings>): void | Promise<void> {
        // Initialize the action display
    }

    /**
     * Called when the user presses the key.
     */
    override async onKeyDown(ev: KeyDownEvent<Settings>): Promise<void> {
        // Handle key press
        // Access settings: ev.payload.settings
        // Update settings: await ev.action.setSettings({ ... })
        // Set title: await ev.action.setTitle("text")
        // Show alert on error: await ev.action.showAlert()
    }
}

type Settings = {
    // Define your settings shape here
};
```

### For a Dial Action (Encoder — Stream Deck +):

```typescript
import streamDeck, { action, DialDownEvent, DialRotateEvent, SingletonAction, TouchTapEvent, WillAppearEvent } from "@elgato/streamdeck";

@action({ UUID: "<PLUGIN_UUID>.<ACTION_SLUG>" })
export class <ActionClassName> extends SingletonAction<Settings> {
    override onWillAppear(ev: WillAppearEvent<Settings>): void | Promise<void> {
        // Set initial feedback on the touch strip
        return ev.action.setFeedback({
            title: "My Action",
            value: "0"
        });
    }

    /**
     * Called when the dial is pressed.
     */
    override async onDialDown(ev: DialDownEvent<Settings>): Promise<void> {
        // Handle dial press
    }

    /**
     * Called when the dial is rotated.
     */
    override async onDialRotate(ev: DialRotateEvent<Settings>): Promise<void> {
        // ev.payload.ticks — positive = clockwise, negative = counter-clockwise
    }

    /**
     * Called when the touch strip area is tapped.
     */
    override async onTouchTap(ev: TouchTapEvent<Settings>): Promise<void> {
        // Handle touch tap
    }
}

type Settings = {
    // Define your settings shape here
};
```

### For a Two-State Toggle Action:

```typescript
import streamDeck, { action, KeyDownEvent, SingletonAction, WillAppearEvent } from "@elgato/streamdeck";

@action({ UUID: "<PLUGIN_UUID>.<ACTION_SLUG>" })
export class <ActionClassName> extends SingletonAction<Settings> {
    override onWillAppear(ev: WillAppearEvent<Settings>): void | Promise<void> {
        // State 0 = off, State 1 = on (auto-toggled by Stream Deck)
    }

    override async onKeyDown(ev: KeyDownEvent<Settings>): Promise<void> {
        const isOn = ev.payload.state === 1;
        // React to the current state
        // Stream Deck automatically toggles between states for 2-state actions
    }
}

type Settings = {};
```

## Step 2: Register the Action

Edit `src/plugin.ts` to import and register the new action:

```typescript
import { <ActionClassName> } from "./actions/<action-slug>";

streamDeck.actions.registerAction(new <ActionClassName>());
```

**Important:** All actions must be registered BEFORE `streamDeck.connect()`.

## Step 3: Update the Manifest

Add the action to the `Actions` array in `manifest.json`:

### For a Key action:

```json
{
    "Name": "<Action Display Name>",
    "UUID": "<PLUGIN_UUID>.<ACTION_SLUG>",
    "Icon": "imgs/actions/<action-slug>/icon",
    "Tooltip": "<Brief description of what this action does>",
    "Controllers": ["Keypad"],
    "States": [
        {
            "Image": "imgs/actions/<action-slug>/key",
            "TitleAlignment": "bottom"
        }
    ]
}
```

### For a Dial action:

```json
{
    "Name": "<Action Display Name>",
    "UUID": "<PLUGIN_UUID>.<ACTION_SLUG>",
    "Icon": "imgs/actions/<action-slug>/icon",
    "Tooltip": "<Brief description>",
    "Controllers": ["Encoder"],
    "Encoder": {
        "layout": "$A1",
        "TriggerDescription": {
            "Rotate": "Adjust",
            "Push": "Toggle",
            "Touch": "Select"
        }
    },
    "States": [
        {
            "Image": "imgs/actions/<action-slug>/key"
        }
    ]
}
```

### For both Key + Dial:

Set `"Controllers": ["Keypad", "Encoder"]` and include both `States` and `Encoder` properties.

### For a two-state toggle:

```json
{
    "Name": "<Action Display Name>",
    "UUID": "<PLUGIN_UUID>.<ACTION_SLUG>",
    "Icon": "imgs/actions/<action-slug>/icon",
    "Controllers": ["Keypad"],
    "States": [
        {
            "Image": "imgs/actions/<action-slug>/state0",
            "Name": "Off"
        },
        {
            "Image": "imgs/actions/<action-slug>/state1",
            "Name": "On"
        }
    ]
}
```

## Step 4: Create Placeholder Icons

Create the icon directories and note which images are needed:

```
imgs/actions/<action-slug>/
├── icon.png        # 20x20 — shown in action list
├── icon@2x.png     # 40x40 — retina action list
├── key.png         # 72x72 — default key image
└── key@2x.png      # 144x144 — retina key image
```

**Icon path rules:** Manifest paths omit the file extension. Stream Deck auto-resolves `.png` and `@2x.png` variants.

## Step 5: Suggest Next Steps

After adding the action:
1. If the action needs user-configurable settings, suggest `/stream-deck:add-property-inspector`
2. Run `npm run build` or `npm run watch` to test
3. The action should appear in Stream Deck under the plugin's category

## Available Events on SingletonAction

| Event | When it fires |
|-------|--------------|
| `onWillAppear` | Action appears on canvas (page nav, startup) |
| `onWillDisappear` | Action leaves canvas |
| `onKeyDown` | Key is pressed down |
| `onKeyUp` | Key is released |
| `onDialDown` | Dial is pressed (SD+) |
| `onDialUp` | Dial is released (SD+) |
| `onDialRotate` | Dial is rotated (SD+) |
| `onTouchTap` | Touch strip is tapped (SD+) |
| `onDidReceiveSettings` | Settings updated from property inspector |
| `onPropertyInspectorDidAppear` | User opens the action's settings panel |
| `onPropertyInspectorDidDisappear` | User closes the settings panel |
| `onSendToPlugin` | Message received from property inspector |
| `onTitleParametersDidChange` | User changes title in SD app |

## Available Action Commands

| Command | What it does |
|---------|-------------|
| `ev.action.setTitle(title)` | Set the key title text |
| `ev.action.setImage(image)` | Set the key image (base64 or path) |
| `ev.action.setSettings(settings)` | Persist action settings |
| `ev.action.getSettings()` | Retrieve action settings |
| `ev.action.showAlert()` | Flash yellow warning triangle |
| `ev.action.showOk()` | Flash checkmark |
| `ev.action.setFeedback(feedback)` | Update touch strip layout values (SD+) |
| `ev.action.setFeedbackLayout(layout)` | Change the touch strip layout (SD+) |
| `ev.action.setTriggerDescription(desc)` | Set dial interaction hints (SD+) |

For complete reference, see the docs/ directory.
