---
name: add-property-inspector
description: Add a property inspector (settings UI) to a Stream Deck action. Use when adding settings panels, configuration UI, or user-facing options to actions.
argument-hint: <action-name>
allowed-tools: Bash, Write, Edit, Read, Glob, Grep
---

# Add a Property Inspector to a Stream Deck Action

Property inspectors are HTML-based UI panels that let users configure your action's settings in the Stream Deck app.

## Gather Context

1. Read the `manifest.json` to find the target action and plugin UUID
2. Read the action's TypeScript file to understand its settings type
3. Determine what settings the user wants to expose

From $ARGUMENTS or the user, determine:
- Which action gets the property inspector
- What settings fields to expose (text, dropdown, toggle, color, etc.)

## Step 1: Create the HTML File

Create `<uuid>.sdPlugin/ui/<action-slug>.html`:

```html
<!doctype html>
<html>
  <head lang="en">
    <meta charset="utf-8" />
    <script src="sdpi-components.js"></script>
  </head>
  <body>
    <!-- Each sdpi-item is a row with a label -->
    <!-- The "setting" attribute auto-syncs with action settings -->

    <!-- Text field -->
    <sdpi-item label="Name">
      <sdpi-textfield setting="name" placeholder="Enter a name"></sdpi-textfield>
    </sdpi-item>

    <!-- Dropdown / Select -->
    <sdpi-item label="Mode">
      <sdpi-select setting="mode">
        <option value="option1">Option 1</option>
        <option value="option2">Option 2</option>
      </sdpi-select>
    </sdpi-item>

    <!-- Checkbox / Toggle -->
    <sdpi-item label="Enabled">
      <sdpi-checkbox setting="enabled"></sdpi-checkbox>
    </sdpi-item>

    <!-- Range / Slider -->
    <sdpi-item label="Volume">
      <sdpi-range setting="volume" min="0" max="100" step="1" default="50"></sdpi-range>
    </sdpi-item>
  </body>
</html>
```

### Available sdpi-components

| Component | Tag | Use case |
|-----------|-----|----------|
| Button | `<sdpi-button>` | Trigger actions |
| Checkbox | `<sdpi-checkbox>` | Boolean toggle |
| Checkbox List | `<sdpi-checkbox-list>` | Multiple selections |
| Color Picker | `<sdpi-color>` | Color selection |
| Date | `<sdpi-calendar type="date">` | Date input |
| DateTime | `<sdpi-calendar type="datetime-local">` | Date + time |
| File Picker | `<sdpi-file>` | File selection |
| Password | `<sdpi-password>` | Masked text input |
| Radio List | `<sdpi-radio>` | Single selection from options |
| Range/Slider | `<sdpi-range>` | Numeric range |
| Select/Dropdown | `<sdpi-select>` | Dropdown selection |
| Textarea | `<sdpi-textarea>` | Multi-line text |
| Textfield | `<sdpi-textfield>` | Single-line text |

**Key concept:** The `setting` attribute on each component automatically binds to the action's settings. When the user changes a value, it's persisted and the action's `onDidReceiveSettings` handler fires.

## Step 2: Download the UI Library

Download `sdpi-components.js` into the UI directory:

```bash
curl -o <uuid>.sdPlugin/ui/sdpi-components.js https://sdpi-components.dev/releases/v4/sdpi-components.js
```

Alternatively, reference it remotely (not recommended for production):
```html
<script src="https://sdpi-components.dev/releases/v4/sdpi-components.js"></script>
```

## Step 3: Update the Manifest

Add `PropertyInspectorPath` to the action in `manifest.json`:

```json
{
    "Name": "My Action",
    "UUID": "com.example.plugin.my-action",
    "PropertyInspectorPath": "ui/my-action.html",
    ...
}
```

You can also set `PropertyInspectorPath` at the plugin root level as a default for all actions.

## Step 4: Handle Settings in the Action

Ensure the action's TypeScript file has the matching settings type and handler:

```typescript
import { action, DidReceiveSettingsEvent, KeyDownEvent, SingletonAction, WillAppearEvent } from "@elgato/streamdeck";

type Settings = {
    name: string;
    mode: string;
    enabled: boolean;
    volume: number;
};

@action({ UUID: "com.example.plugin.my-action" })
export class MyAction extends SingletonAction<Settings> {
    override onWillAppear(ev: WillAppearEvent<Settings>): void | Promise<void> {
        // Use settings from ev.payload.settings
        const { name = "Default" } = ev.payload.settings;
        return ev.action.setTitle(name);
    }

    override onDidReceiveSettings(ev: DidReceiveSettingsEvent<Settings>): void | Promise<void> {
        // React to settings changes from the property inspector
        const { name } = ev.payload.settings;
        return ev.action.setTitle(name ?? "Default");
    }

    override async onKeyDown(ev: KeyDownEvent<Settings>): Promise<void> {
        // Access current settings
        const settings = ev.payload.settings;
        // ... action logic using settings
    }
}
```

## Step 5: Advanced Patterns

### Global Settings (plugin-wide, not per-action)

For settings shared across all actions (e.g., API tokens):

In the property inspector HTML:
```html
<sdpi-item label="API Key">
  <sdpi-textfield setting="apiKey" global></sdpi-textfield>
</sdpi-item>
```

The `global` attribute stores the value in global settings instead of action settings.

In the plugin:
```typescript
const globalSettings = await streamDeck.settings.getGlobalSettings<GlobalSettings>();
```

### Custom Communication (sendToPlugin / sendToPropertyInspector)

For complex interactions that go beyond simple settings:

```typescript
// In the action — send data to the property inspector
ev.action.sendToPropertyInspector({ type: "update", data: someData });

// In the action — receive messages from the property inspector
override onSendToPlugin(ev: SendToPluginEvent): void {
    const message = ev.payload;
    // Handle custom message
}
```

### Security Warning

**Never store API keys or secrets in action settings.** Action settings are plain-text and exported with profiles. Use **global settings** for sensitive data (stored securely on the local machine), or use the **Secrets API** for plugin-level secrets distributed with DRM.

## Step 6: Test

1. Run `npm run build` or `npm run watch`
2. In Stream Deck, drag your action to a key
3. Click the action — the property inspector panel should appear on the right
4. Changes to settings should immediately reflect in the action
