# Napoleon UI Library

A premium, runtime-loaded Roblox Lua UI framework designed for sophisticated script hubs. Loaded dynamically via `HttpGet`, Napoleon provides a highly customizable windowed interface with nested sections, intelligent tabs, collapsible groups with internal tabbing, and an extensive widget toolkit — all backed by a robust configuration save/load system.

---

## Table of Contents

- [Loading](#loading)
- [Design Philosophy](#design-philosophy)
- [Icons](#icons)
- [Window](#window)
  - [Window Configuration](#window-configuration)
  - [Window Methods](#window-methods)
  - [ESP Preview System](#esp-preview-system)
- [Section](#section)
- [Tab](#tab)
- [Group](#group)
- [Group Tabs](#group-tabs)
- [Controls](#controls)
  - [Toggle](#toggle)
  - [Button](#button)
  - [Label](#label)
  - [Divider](#divider)
  - [Dropdown](#dropdown)
  - [Dynamic Dropdown](#dynamic-dropdown)
  - [Multi-Select Dropdown](#multi-select-dropdown)
  - [Slider](#slider)
  - [Keybind](#keybind)
  - [Keybind Toggle](#keybind-toggle)
  - [Text Input](#text-input)
  - [Textbox](#textbox)
  - [Color Picker](#color-picker)
- [Configuration System](#configuration-system)
- [Advanced Patterns](#advanced-patterns)
- [Full Production Example](#full-production-example)
- [API Quick Reference](#api-quick-reference)
- [Changelog](#changelog)

---

## Loading

```lua
-- Production source (always up-to-date)
local librarySource = game:HttpGet("https://raw.githubusercontent.com/VoidDeveloper67/Nepo/refs/heads/main/tuf%2B3.txt")
local Library = loadstring(librarySource)()
```

### Development Workflow

For rapid iteration during development, use a local source file. This eliminates network latency and allows instant visual feedback on changes.

```lua
local Library

-- Attempt local development source first
local success, localSource = pcall(function()
    return readfile("neposource.lua")
end)

if success then
    Library = loadstring(localSource)()
    warn("[Napoleon] Loaded from local development source")
else
    local hostedSource = game:HttpGet("https://raw.githubusercontent.com/VoidDeveloper67/Nepo/refs/heads/main/tuf%2B3.txt")
    Library = loadstring(hostedSource)()
    warn("[Napoleon] Loaded from hosted source")
end
```

---

## Design Philosophy

Napoleon UI is built around **compositional hierarchy** and **declarative configuration**:

- **Window** → Global container and theming engine
- **Sections** → Logical feature categories (Combat, Visuals, Movement, etc.)
- **Tabs** → Sub-features within a section
- **Groups** → Visual panels that organize controls into columns
- **Group Tabs** → Further subdivision within a group for complex features
- **Controls** → Interactive widgets that drive script behavior

Every control supports **silent updates** (bypassing callbacks), **auto-generated configuration flags**, and **runtime value inspection**.

---

## Icons

Napoleon accepts icon references in multiple formats for maximum flexibility:

```lua
-- All of these are valid and equivalent
Icon = "rbxassetid://7734053495"
Icon = 7734053495
Icon = "7734053495"
```

### Icon Management Best Practices

Centralize your icons in a dedicated table to maintain consistency and simplify updates:

```lua
local Icons = {
    -- Combat
    Crosshair   = "rbxassetid://7733765307",
    Target      = "rbxassetid://7734053495",
    Sword       = "rbxassetid://7733942651",
    Shield      = "rbxassetid://7733955511",

    -- Visuals
    Eye         = "rbxassetid://7734020488",
    Palette     = "rbxassetid://7734068321",
    Box         = "rbxassetid://7733917120",

    -- Utility
    Settings    = "rbxassetid://7734068321",
    Play        = "rbxassetid://7734021700",
    Save        = "rbxassetid://7734037533",
    Bell        = "rbxassetid://7733992358",

    -- Navigation
    Home        = "rbxassetid://7733970061",
    ChevronRight= "rbxassetid://7733715400",
    CornerDown  = "rbxassetid://7733765140",
}
```

> **Recommendation:** The Lydie/Lucide Roblox icon export (773.../774... asset ID range) provides crisp 256×256 vector icons that scale flawlessly across all display resolutions.

---

## Window

### `Library.new(options)` → `Window`

The Window is the root container. It manages global theming, input handling, notifications, and the configuration subsystem.

```lua
local Window = Library.new({
    Name                = "Napoleon // Script Hub",
    AccentColor         = Color3.fromRGB(99, 130, 255),
    BackgroundColor     = Color3.fromRGB(8, 8, 12),
    SecondaryColor      = Color3.fromRGB(15, 15, 22),
    TextColor           = Color3.fromRGB(245, 245, 250),
    SubTextColor        = Color3.fromRGB(140, 140, 155),
    KeyDurationText     = "Key: Lifetime",
    ShowStatusStrip     = true,
    Motion              = "Smooth",
    AutoConfig          = true,
    ShowAutoSaveToggle  = true,
})
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `Name` | `string` | `"Napoleon"` | Window title displayed in the title bar |
| `AccentColor` | `Color3` | `Color3.fromRGB(99, 130, 255)` | Primary highlight color for active states, sliders, and indicators |
| `BackgroundColor` | `Color3` | `Color3.fromRGB(8, 8, 12)` | Main window background |
| `SecondaryColor` | `Color3` | `Color3.fromRGB(15, 15, 22)` | Panel backgrounds, group containers |
| `TextColor` | `Color3` | `Color3.fromRGB(245, 245, 250)` | Primary text color |
| `SubTextColor` | `Color3` | `Color3.fromRGB(140, 140, 155)` | Descriptions, placeholders, inactive text |
| `KeyDurationText` | `string` | `nil` | Optional status text shown in the title bar |
| `ShowStatusStrip` | `boolean` | `false` | Whether to display the bottom status strip |
| `Motion` | `string` | `"Normal"` | Animation style: `"Normal"`, `"Smooth"`, or `"Instant"` |
| `AutoConfig` | `boolean` | `false` | Automatically save configuration on every control change |
| `ShowAutoSaveToggle` | `boolean` | `false` | Show a toggle in the UI to enable/disable `AutoConfig` |

---

### Window Methods

#### `Window:SetToggleKey(keyCode: Enum.KeyCode)`

Binds a keyboard key to toggle the window visibility.

```lua
Window:SetToggleKey(Enum.KeyCode.RightShift)
-- or
Window:SetToggleKey(Enum.KeyCode.Insert)
```

---

#### `Window:SetAccentColor(color: Color3)`

Dynamically updates the accent color across all UI elements in real-time.

```lua
Window:SetAccentColor(Color3.fromRGB(255, 107, 107))  -- Coral red
```

---

#### `Window:Notify(options)`

Dispatches a toast notification with customizable duration, icon, and styling.

```lua
Window:Notify({
    Title       = "Configuration Saved",
    Description = "Your settings have been persisted to slot 'PvP-Config'.",
    Duration    = 4,
    Icon        = Icons.Save,
})
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Title` | `string` | Yes | Bold heading text |
| `Description` | `string` | Yes | Body content |
| `Duration` | `number` | Yes | Display duration in seconds |
| `Icon` | `string` | No | Roblox asset ID for the notification icon |

---

#### `Window:SaveConfig(configName: string)` → `boolean, string?`

Serializes all control states with explicit or auto-generated flags to persistent storage.

```lua
local success, errorMessage = Window:SaveConfig("aggressive-pvp")
if success then
    Window:Notify({
        Title = "Success",
        Description = "Configuration 'aggressive-pvp' saved.",
        Duration = 3,
        Icon = Icons.Save,
    })
else
    Window:Notify({
        Title = "Save Failed",
        Description = errorMessage or "Unknown error occurred.",
        Duration = 5,
    })
end
```

---

#### `Window:LoadConfig(configName: string)` → `boolean, string?`

Restores all control states from a named configuration profile.

```lua
local success, errorMessage = Window:LoadConfig("aggressive-pvp")
if not success then
    warn("[Napoleon] Failed to load config:", errorMessage)
end
```

---

#### `Window:Destroy()`

Completely destroys the window instance, cleans up all connections, and releases memory.

```lua
Window:Destroy()
```

---

## ESP Preview System

Napoleon includes a built-in ESP (Extra Sensory Perception) preview renderer for visual configuration without entering gameplay.

#### `Window:SetESPPreviewEnabled(enabled: boolean)`

Toggles the ESP preview overlay.

```lua
Window:SetESPPreviewEnabled(true)
```

#### `Window:SetESPPreviewData(data: table)`

Updates the preview with target simulation data.

```lua
Window:SetESPPreviewData({
    Name            = "TargetPlayer",
    Item            = "Enchanted Sword",
    Distance        = 142,
    HealthPercent   = 0.65,
    Color           = Color3.fromRGB(99, 130, 255),
    ShowBox         = true,
    ShowName        = true,
    ShowHealth      = true,
    ShowDistance    = true,
    ShowHighlight   = true,
})
```

| Field | Type | Description |
|-------|------|-------------|
| `Name` | `string` | Player name label |
| `Item` | `string` | Equipped item display |
| `Distance` | `number` | Distance in studs |
| `HealthPercent` | `number` | 0.0 to 1.0 health ratio |
| `Color` | `Color3` | ESP highlight color |
| `ShowBox` | `boolean` | Render bounding box |
| `ShowName` | `boolean` | Render name tag |
| `ShowHealth` | `boolean` | Render health bar |
| `ShowDistance` | `boolean` | Render distance text |
| `ShowHighlight` | `boolean` | Render glow/chams effect |

---

## Section

### `Window:CreateSection(options)` → `Section`

Sections are top-level navigation categories displayed in the sidebar. They group related tabs under a unified header with an icon.

```lua
local CombatSection = Window:CreateSection({
    Name         = "Combat",
    Icon         = Icons.Sword,
    FallbackIcon = Icons.Shield,  -- Used if primary icon fails to load
})

local VisualSection = Window:CreateSection({
    Name = "Visuals",
    Icon = Icons.Eye,
})

local MiscSection = Window:CreateSection({
    Name = "Miscellaneous",
    Icon = Icons.Settings,
})
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | `string` | Yes | Display label in the sidebar |
| `Icon` | `string` | Yes | Primary Roblox asset ID |
| `FallbackIcon` | `string` | No | Backup icon if primary fails |

---

## Tab

### `Section:AddTab(options)` → `Tab`

Tabs live inside sections and represent distinct feature sets. Each tab displays a name, description, and icon.

```lua
local AimbotTab = CombatSection:AddTab({
    Name         = "Aimbot",
    Description  = "Precision targeting with prediction and smoothing.",
    Icon         = Icons.Crosshair,
    FallbackIcon = Icons.Target,
})

local ESP_Tab = VisualSection:AddTab({
    Name        = "ESP",
    Description = "Entity highlighting and information overlays.",
    Icon        = Icons.Eye,
})
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | `string` | Yes | Tab label |
| `Description` | `string` | Yes | Subtitle shown below the tab name |
| `Icon` | `string` | Yes | Tab icon asset ID |
| `FallbackIcon` | `string` | No | Fallback icon |

---

## Group

### `Tab:AddGroup(options)` → `Group`

Groups are visual panels that contain controls. They can be positioned in left or right columns to create balanced layouts.

```lua
local MainGroup = AimbotTab:AddGroup({
    Name = "Core Settings",
    Side = "Left",
    Icon = Icons.Crosshair,
})

local AdvancedGroup = AimbotTab:AddGroup({
    Name = "Advanced",
    Side = "Right",
    Icon = Icons.Settings,
})
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | `string` | Yes | Group heading |
| `Side` | `"Left" \| "Right"` | Yes | Column placement |
| `Icon` | `string` | Yes | Group header icon |

---

## Group Tabs

### `Group:AddTab(options)` → `GroupTab`

**Advanced Feature:** Splits a group into internal tabbed pages. Ideal for complex features with many controls (e.g., Aimbot with separate pages for Targeting, Smoothing, and Checks).

```lua
-- Always guard against older library versions
local TargetingPage = Group
local SmoothingPage = Group
local ChecksPage = Group

if Group.AddTab then
    TargetingPage = Group:AddTab({
        Name    = "Targeting",
        Icon    = Icons.Crosshair,
        Default = true,  -- This tab opens by default
    })

    SmoothingPage = Group:AddTab({
        Name = "Smoothing",
        Icon = Icons.CornerDown,
    })

    ChecksPage = Group:AddTab({
        Name = "Checks",
        Icon = Icons.Shield,
    })
end

-- Add controls to specific sub-tabs
TargetingPage:AddToggle({
    Name = "Enabled",
    Default = false,
    Callback = function(v) end,
})

ChecksPage:AddToggle({
    Name = "Team Check",
    Default = true,
    Callback = function(v) end,
})
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | `string` | Yes | Tab label |
| `Icon` | `string` | Yes | Tab icon |
| `Default` | `boolean` | No | Whether this tab is active by default |

---

## Controls

All controls are created via methods on `Group` or `GroupTab` instances.

---

### Toggle

### `Group:AddToggle(options)` → `Toggle`

A binary on/off switch with optional configuration flagging.

```lua
local aimEnabled = MainGroup:AddToggle({
    Name     = "Aimbot Enabled",
    Default  = false,
    Flag     = "Combat.Aimbot.Enabled",
    Callback = function(value)
        -- value: boolean
        print("Aimbot state:", value)
    end,
})
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | `string` | Yes | Label text |
| `Default` | `boolean` | No | Initial state (default: `false`) |
| `Flag` | `string` | No | Config key; auto-generated if omitted |
| `Callback` | `function(boolean)` | No | Fired on state change |

**Methods:**

```lua
aimEnabled:Set(true)           -- Enable with callback
aimEnabled:Set(true, true)     -- Enable silently (no callback)
aimEnabled:Get()               -- Returns current boolean
```

---

### Button

### `Group:AddButton(options)`

A clickable action button with optional icon.

```lua
MainGroup:AddButton({
    Name     = "Force Target Refresh",
    Icon     = Icons.Play,
    Callback = function()
        -- Execute immediate action
        print("Target list refreshed")
    end,
})
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | `string` | Yes | Button label |
| `Icon` | `string` | No | Button icon |
| `Callback` | `function()` | Yes | Click handler |

---

### Label

### `Group:AddLabel(options)`

Static text display with optional word wrapping.

```lua
MainGroup:AddLabel({
    Text = "Status: Scanning for targets...",
    Wrap = true,
})
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Text` | `string` | Yes | Display text |
| `Wrap` | `boolean` | No | Enable word wrapping |

---

### Divider

### `Group:AddDivider()`

Visual separator line for organizing controls into logical regions.

```lua
MainGroup:AddDivider()
```

---

### Dropdown

### `Group:AddDropdown(options)` → `Dropdown`

Single-selection dropdown with configurable options.

```lua
local targetPriority = MainGroup:AddDropdown({
    Name     = "Priority Mode",
    Options  = {"Closest", "Lowest HP", "Highest HP", "Distance"},
    Default  = "Closest",
    Flag     = "Combat.Aimbot.Priority",
    Callback = function(value)
        -- value: string (selected option)
        print("Priority set to:", value)
    end,
})
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | `string` | Yes | Label |
| `Options` | `table` | Yes | Array of string options |
| `Default` | `string` | No | Initially selected option |
| `Flag` | `string` | No | Config key |
| `Callback` | `function(string)` | No | Selection change handler |

**Methods:**

```lua
targetPriority:Set("Lowest HP")        -- Change selection
targetPriority:Set("Lowest HP", true)  -- Change silently
targetPriority:Get()                   -- Returns selected string
targetPriority:UpdateOptions({"New A", "New B"})  -- Replace all options
```

---

### Dynamic Dropdown

Dropdowns can auto-refresh their options from a provider function — perfect for dynamic game data like player lists or quest names.

```lua
MainGroup:AddDropdown({
    Name            = "Active Quest",
    OptionsProvider = function()
        -- Return fresh data every refresh cycle
        return getActiveQuests() or {"No quests available"}
    end,
    AutoRefresh     = true,
    RefreshInterval = 1.0,  -- Refresh every second
    Callback        = function(selectedQuest)
        print("Selected quest:", selectedQuest)
    end,
})
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `OptionsProvider` | `function()` | Yes | Returns a table of strings |
| `AutoRefresh` | `boolean` | No | Enable automatic refreshing |
| `RefreshInterval` | `number` | No | Seconds between refreshes |

---

### Multi-Select Dropdown

### `Group:AddMultiDropdown(options)` → `MultiDropdown`

Select multiple options simultaneously. Returns an array of selected values.

```lua
local espLayers = ESP_Group:AddMultiDropdown({
    Name     = "ESP Layers",
    Options  = {"Box", "Name", "Health", "Distance", "Weapon", "Tracer"},
    Default  = {"Box", "Name"},
    Flag     = "Visuals.ESP.Layers",
    Callback = function(values)
        -- values: table of selected strings
        print("Active layers:", table.concat(values, ", "))
    end,
})
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | `string` | Yes | Label |
| `Options` | `table` | Yes | Available options |
| `Default` | `table` | No | Initially selected options |
| `Flag` | `string` | No | Config key |
| `Callback` | `function(table)` | No | Change handler |

**Methods:**

```lua
espLayers:Set({"Box", "Health", "Tracer"})
espLayers:Set({"Box"}, true)  -- Silent
espLayers:Get()               -- Returns table of selected strings
espLayers:UpdateOptions({"Box", "Name", "Health"})  -- Update available options
```

---

### Slider

### `Group:AddSlider(options)` → `Slider`

Numeric input with draggable handle, min/max constraints, and optional suffix.

```lua
local fovSlider = MainGroup:AddSlider({
    Name      = "Field of View",
    Min       = 10,
    Max       = 360,
    Default   = 120,
    Increment = 5,
    Suffix    = "°",
    Flag      = "Combat.Aimbot.FOV",
    Callback  = function(value)
        -- value: number
        print("FOV:", value)
    end,
})
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | `string` | Yes | Label |
| `Min` | `number` | Yes | Minimum value |
| `Max` | `number` | Yes | Maximum value |
| `Default` | `number` | No | Starting value |
| `Increment` | `number` | No | Step size (default: `1`) |
| `Suffix` | `string` | No | Text appended to value display |
| `Flag` | `string` | No | Config key |
| `Callback` | `function(number)` | No | Change handler |

**Methods:**

```lua
fovSlider:Set(180)              -- Animate to value
fovSlider:Set(180, true)        -- Instant jump (no animation)
fovSlider:Set(180, true, true)  -- Instant and silent
fovSlider:Get()                 -- Returns current number
```

---

### Keybind

### `Group:AddKeybind(options)` → `Keybind`

Keyboard input binding with Hold or Toggle behavior modes.

```lua
local aimKey = MainGroup:AddKeybind({
    Name            = "Aim Key",
    Default         = Enum.KeyCode.LeftAlt,
    Mode            = "Hold",  -- "Hold" or "Toggle"
    Callback        = function(active)
        -- active: boolean
        -- "Hold": true while key is pressed
        -- "Toggle": true when toggled on
        print("Aim active:", active)
    end,
    ChangedCallback = function(newKey)
        -- newKey: Enum.KeyCode
        print("Rebound to:", newKey.Name)
    end,
})
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | `string` | Yes | Label |
| `Default` | `Enum.KeyCode` | No | Initial key binding |
| `Mode` | `string` | No | `"Hold"` or `"Toggle"` |
| `Callback` | `function(boolean)` | No | State change handler |
| `ChangedCallback` | `function(Enum.KeyCode)` | No | Rebind handler |

---

### Keybind Toggle

### `Group:AddKeybindToggle(options)` → `KeybindToggle`

Combines a keybind with an integrated toggle switch. The keybind only functions when the toggle is enabled.

```lua
local flyBind = MiscGroup:AddKeybindToggle({
    Name            = "Fly Bind",
    Default         = Enum.KeyCode.F,
    ToggleDefault   = false,
    Callback        = function()
        -- Fires when key is pressed (if toggle is on)
        print("Fly activated")
    end,
    ToggleCallback  = function(enabled)
        -- Fires when toggle is switched
        print("Fly mode:", enabled)
    end,
    ChangedCallback = function(newKey)
        print("Fly key changed to:", newKey.Name)
    end,
})
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | `string` | Yes | Label |
| `Default` | `Enum.KeyCode` | No | Initial key |
| `ToggleDefault` | `boolean` | No | Initial toggle state |
| `Callback` | `function()` | No | Key press handler |
| `ToggleCallback` | `function(boolean)` | No | Toggle change handler |
| `ChangedCallback` | `function(Enum.KeyCode)` | No | Rebind handler |

---

### Text Input

### `Group:AddTextInput(options)` → `TextInput`

Live-updating text field that fires on every character change.

```lua
local configName = UtilityGroup:AddTextInput({
    Name        = "Profile Name",
    Placeholder = "my-profile",
    Default     = "default",
    Callback    = function(value)
        -- value: string
        print("Profile name:", value)
    end,
})
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | `string` | Yes | Label |
| `Placeholder` | `string` | No | Hint text when empty |
| `Default` | `string` | No | Pre-filled value |
| `Callback` | `function(string)` | No | Input change handler |

**Methods:**

```lua
configName:SetValue("aggressive-pvp")
configName:SetValue("stealth", true)  -- Silent
configName:Get()                       -- Returns current string
```

---

### Textbox

### `Group:AddTextbox(options)` → `Textbox`

Text input with optional Enter-only submission mode. Supports multi-line scenarios.

```lua
local webhookInput = UtilityGroup:AddTextbox({
    Title       = "Discord Webhook",
    Placeholder = "https://discord.com/api/webhooks/...",
    Default     = "",
    EnterOnly   = true,
    Callback    = function(text, enterPressed)
        -- text: string
        -- enterPressed: boolean
        if enterPressed then
            validateWebhook(text)
        end
    end,
})
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Title` | `string` | Yes | Label above input |
| `Placeholder` | `string` | No | Hint text |
| `Default` | `string` | No | Pre-filled value |
| `EnterOnly` | `boolean` | No | `true` = callback only on Enter press |
| `Callback` | `function(string, boolean)` | No | Input handler |

---

### Color Picker

### `Group:AddColorPicker(options)` → `ColorPicker`

Full RGB color selector with live preview.

```lua
local accentPicker = ThemeGroup:AddColorPicker({
    Name     = "Accent Color",
    Default  = Color3.fromRGB(99, 130, 255),
    Callback = function(color)
        -- color: Color3
        Window:SetAccentColor(color)
    end,
})
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | `string` | Yes | Label |
| `Default` | `Color3` | No | Initial color |
| `Callback` | `function(Color3)` | No | Color change handler |

**Methods:**

```lua
accentPicker:Set(Color3.fromRGB(255, 0, 0))
accentPicker:Set(Color3.fromRGB(0, 255, 0), true)  -- Silent
accentPicker:Get()  -- Returns current Color3
```

---

## Configuration System

Napoleon automatically tracks all controls that have explicit `Flag` values or auto-generated flags. This enables seamless save/load functionality across sessions.

### How Flags Work

- **Explicit flags:** You define the key (e.g., `"Combat.Aimbot.FOV"`)
- **Auto-generated flags:** Created from control position and name if omitted
- **Scope:** Dot notation recommended for organization (`Category.Feature.Setting`)

### Save/Load Patterns

```lua
-- Save current state
local success = Window:SaveConfig("my-preset")

-- Load previous state
local success = Window:LoadConfig("my-preset")

-- Auto-save on every change (set during window creation)
local Window = Library.new({
    AutoConfig = true,  -- Automatically persists to default config
})
```

### Configuration Best Practices

```lua
-- Use descriptive, hierarchical flag names
Flag = "Combat.Aimbot.Enabled"
Flag = "Combat.Aimbot.FOV"
Flag = "Combat.Aimbot.Priority"
Flag = "Visuals.ESP.Box.Enabled"
Flag = "Visuals.ESP.Box.Color"
Flag = "Movement.Fly.Speed"
Flag = "Movement.Fly.Keybind"

-- Create a config management UI
local ConfigGroup = MiscTab:AddGroup({
    Name = "Configuration",
    Side = "Left",
    Icon = Icons.Save,
})

local configInput = ConfigGroup:AddTextInput({
    Name     = "Config Name",
    Default  = "default",
    Flag     = "System.Config.Name",
})

ConfigGroup:AddButton({
    Name = "Save Config",
    Icon = Icons.Save,
    Callback = function()
        local name = configInput:Get()
        local ok = Window:SaveConfig(name)
        Window:Notify({
            Title = ok and "Saved" or "Error",
            Description = ok and "Config '" .. name .. "' saved." or "Failed to save config.",
            Duration = 3,
            Icon = ok and Icons.Save or Icons.Shield,
        })
    end,
})

ConfigGroup:AddButton({
    Name = "Load Config",
    Icon = Icons.Play,
    Callback = function()
        local name = configInput:Get()
        local ok = Window:LoadConfig(name)
        Window:Notify({
            Title = ok and "Loaded" or "Error",
            Description = ok and "Config '" .. name .. "' loaded." or "Failed to load config.",
            Duration = 3,
        })
    end,
})
```

---

## Advanced Patterns

### Pattern 1: Conditional UI Based on Toggle State

```lua
local aimEnabled = MainGroup:AddToggle({
    Name    = "Aimbot Enabled",
    Default = false,
    Flag    = "Combat.Aimbot.Enabled",
})

-- Controls that only make sense when aimbot is on
local fovSlider = MainGroup:AddSlider({
    Name     = "FOV",
    Min      = 10,
    Max      = 360,
    Default  = 120,
    Flag     = "Combat.Aimbot.FOV",
})

-- Link visibility or sensitivity to the toggle
aimEnabled:OnChanged(function(enabled)
    -- You can implement visual feedback or logic gating here
    print("Aimbot state changed:", enabled)
end)
```

### Pattern 2: Dynamic Control Updates

```lua
local weaponDropdown = MainGroup:AddDropdown({
    Name    = "Weapon Type",
    Options = {"Sword", "Gun", "Magic"},
    Default = "Sword",
})

local damageSlider = MainGroup:AddSlider({
    Name    = "Damage Multiplier",
    Min     = 1,
    Max     = 10,
    Default = 1,
})

-- Update slider constraints based on weapon selection
weaponDropdown:OnChanged(function(weapon)
    if weapon == "Gun" then
        damageSlider:Set(5)
        -- In a real implementation, you might update Min/Max here
    end
end)
```

### Pattern 3: Cross-Control Synchronization

```lua
local espColor = ESP_Group:AddColorPicker({
    Name     = "ESP Color",
    Default  = Color3.fromRGB(99, 130, 255),
    Flag     = "Visuals.ESP.Color",
    Callback = function(color)
        Window:SetAccentColor(color)
    end,
})

-- Button to apply ESP color to UI accent
ESP_Group:AddButton({
    Name = "Sync to UI",
    Callback = function()
        Window:SetAccentColor(espColor:Get())
        Window:Notify({
            Title = "Synced",
            Description = "ESP color applied to UI accent.",
            Duration = 2,
        })
    end,
})
```

### Pattern 4: Batch Configuration Presets

```lua
local function applyAggressivePreset()
    aimEnabled:Set(true, true)
    fovSlider:Set(180, true, true)
    priorityDropdown:Set("Lowest HP", true)
    teamCheck:Set(false, true)
    visCheck:Set(false, true)

    Window:Notify({
        Title       = "Preset Applied",
        Description = "Aggressive PvP configuration loaded.",
        Duration    = 3,
        Icon        = Icons.Sword,
    })
end

local function applyStealthPreset()
    aimEnabled:Set(true, true)
    fovSlider:Set(60, true, true)
    priorityDropdown:Set("Closest", true)
    teamCheck:Set(true, true)
    visCheck:Set(true, true)

    Window:Notify({
        Title       = "Preset Applied",
        Description = "Stealth configuration loaded.",
        Duration    = 3,
        Icon        = Icons.Shield,
    })
end

PresetsGroup:AddButton({
    Name = "Aggressive",
    Icon = Icons.Sword,
    Callback = applyAggressivePreset,
})

PresetsGroup:AddButton({
    Name = "Stealth",
    Icon = Icons.Shield,
    Callback = applyStealthPreset,
})
```

---

## Full Production Example

```lua
--[[
    Napoleon UI Library - Production Example
    A complete, feature-rich hub implementation demonstrating
    all major features and best practices.
--]]

-- ============================================================
-- 1. LOADING
-- ============================================================
local Library

local success, localSource = pcall(function()
    return readfile("neposource.lua")
end)

if success then
    Library = loadstring(localSource)()
    warn("[Napoleon] Development mode loaded")
else
    local source = game:HttpGet("https://raw.githubusercontent.com/VoidDeveloper67/Nepo/refs/heads/main/tuf%2B3.txt")
    Library = loadstring(source)()
    warn("[Napoleon] Production mode loaded")
end

-- ============================================================
-- 2. ICON REGISTRY
-- ============================================================
local Icons = {
    Crosshair   = "rbxassetid://7733765307",
    Target      = "rbxassetid://7734053495",
    Sword       = "rbxassetid://7733942651",
    Shield      = "rbxassetid://7733955511",
    ShieldCheck = "rbxassetid://7733955511",
    Eye         = "rbxassetid://7734020488",
    Palette     = "rbxassetid://7734068321",
    Settings    = "rbxassetid://7734068321",
    Play        = "rbxassetid://7734021700",
    Save        = "rbxassetid://7734037533",
    Bell        = "rbxassetid://7733992358",
    Home        = "rbxassetid://7733970061",
    CornerDown  = "rbxassetid://7733765140",
    Box         = "rbxassetid://7733917120",
}

-- ============================================================
-- 3. WINDOW INITIALIZATION
-- ============================================================
local Window = Library.new({
    Name                = "Napoleon // PvP Suite",
    AccentColor         = Color3.fromRGB(99, 130, 255),
    BackgroundColor     = Color3.fromRGB(8, 8, 12),
    SecondaryColor      = Color3.fromRGB(15, 15, 22),
    TextColor           = Color3.fromRGB(245, 245, 250),
    SubTextColor        = Color3.fromRGB(140, 140, 155),
    KeyDurationText     = "Key: Lifetime Access",
    ShowStatusStrip     = true,
    Motion              = "Smooth",
    AutoConfig          = false,
    ShowAutoSaveToggle  = true,
})

Window:SetToggleKey(Enum.KeyCode.RightShift)

-- ============================================================
-- 4. SECTIONS
-- ============================================================
local CombatSection = Window:CreateSection({
    Name = "Combat",
    Icon = Icons.Sword,
})

local VisualSection = Window:CreateSection({
    Name = "Visuals",
    Icon = Icons.Eye,
})

local MiscSection = Window:CreateSection({
    Name = "System",
    Icon = Icons.Settings,
})

-- ============================================================
-- 5. COMBAT TAB
-- ============================================================
local AimbotTab = CombatSection:AddTab({
    Name        = "Aimbot",
    Description = "Precision targeting with prediction.",
    Icon        = Icons.Crosshair,
})

-- Left Column: Core Settings
local CoreGroup = AimbotTab:AddGroup({
    Name = "Core",
    Side = "Left",
    Icon = Icons.Crosshair,
})

-- Group Tabs for organization
local TargetingPage = CoreGroup
local SmoothingPage = CoreGroup
local ChecksPage = CoreGroup

if CoreGroup.AddTab then
    TargetingPage = CoreGroup:AddTab({
        Name    = "Targeting",
        Icon    = Icons.Target,
        Default = true,
    })

    SmoothingPage = CoreGroup:AddTab({
        Name = "Smoothing",
        Icon = Icons.CornerDown,
    })

    ChecksPage = CoreGroup:AddTab({
        Name = "Checks",
        Icon = Icons.ShieldCheck,
    })
end

-- Targeting Controls
local aimEnabled = TargetingPage:AddToggle({
    Name     = "Enabled",
    Default  = false,
    Flag     = "Combat.Aimbot.Enabled",
    Callback = function(v)
        print("[Aimbot] Enabled:", v)
    end,
})

local aimMode = TargetingPage:AddDropdown({
    Name     = "Target Mode",
    Options  = {"Closest", "Lowest HP", "Highest HP", "Distance"},
    Default  = "Closest",
    Flag     = "Combat.Aimbot.Mode",
    Callback = function(v)
        print("[Aimbot] Mode:", v)
    end,
})

local fovSlider = TargetingPage:AddSlider({
    Name      = "Field of View",
    Min       = 10,
    Max       = 360,
    Default   = 120,
    Increment = 5,
    Suffix    = "°",
    Flag      = "Combat.Aimbot.FOV",
})

-- Smoothing Controls
local smoothSlider = SmoothingPage:AddSlider({
    Name      = "Smoothing",
    Min       = 0,
    Max       = 1,
    Default   = 0.15,
    Increment = 0.01,
    Suffix    = "",
    Flag      = "Combat.Aimbot.Smoothing",
})

local predictionToggle = SmoothingPage:AddToggle({
    Name     = "Movement Prediction",
    Default  = true,
    Flag     = "Combat.Aimbot.Prediction",
})

-- Checks Controls
checksTeam = ChecksPage:AddToggle({
    Name     = "Team Check",
    Default  = true,
    Flag     = "Combat.Aimbot.TeamCheck",
})

checksVis = ChecksPage:AddToggle({
    Name     = "Visibility Check",
    Default  = true,
    Flag     = "Combat.Aimbot.VisCheck",
})

checksWall = ChecksPage:AddToggle({
    Name     = "Wall Check",
    Default  = false,
    Flag     = "Combat.Aimbot.WallCheck",
})

-- Right Column: Keybinds & Actions
local ActionGroup = AimbotTab:AddGroup({
    Name = "Actions",
    Side = "Right",
    Icon = Icons.Settings,
})

ActionGroup:AddKeybind({
    Name     = "Aim Key",
    Default  = Enum.KeyCode.LeftAlt,
    Mode     = "Hold",
    Callback = function(active)
        if active then
            print("[Aimbot] Key held")
        end
    end,
})

ActionGroup:AddDivider()

ActionGroup:AddButton({
    Name     = "Apply Aggressive",
    Icon     = Icons.Sword,
    Callback = function()
        aimEnabled:Set(true, true)
        fovSlider:Set(180, true, true)
        aimMode:Set("Lowest HP", true)
        checksTeam:Set(false, true)
        checksVis:Set(false, true)
        Window:Notify({
            Title       = "Preset Applied",
            Description = "Aggressive configuration loaded.",
            Duration    = 3,
            Icon        = Icons.Sword,
        })
    end,
})

ActionGroup:AddButton({
    Name     = "Apply Stealth",
    Icon     = Icons.Shield,
    Callback = function()
        aimEnabled:Set(true, true)
        fovSlider:Set(60, true, true)
        aimMode:Set("Closest", true)
        checksTeam:Set(true, true)
        checksVis:Set(true, true)
        Window:Notify({
            Title       = "Preset Applied",
            Description = "Stealth configuration loaded.",
            Duration    = 3,
            Icon        = Icons.Shield,
        })
    end,
})

-- ============================================================
-- 6. VISUALS TAB
-- ============================================================
local ESPTab = VisualSection:AddTab({
    Name        = "ESP",
    Description = "Entity highlighting and overlays.",
    Icon        = Icons.Eye,
})

local ESP_Group = ESPTab:AddGroup({
    Name = "Layers",
    Side = "Left",
    Icon = Icons.Box,
})

local espLayers = ESP_Group:AddMultiDropdown({
    Name     = "Active Layers",
    Options  = {"Box", "Name", "Health", "Distance", "Weapon", "Tracer"},
    Default  = {"Box", "Name"},
    Flag     = "Visuals.ESP.Layers",
    Callback = function(v)
        print("[ESP] Layers:", table.concat(v, ", "))
    end,
})

local espColor = ESP_Group:AddColorPicker({
    Name     = "Highlight Color",
    Default  = Color3.fromRGB(99, 130, 255),
    Flag     = "Visuals.ESP.Color",
    Callback = function(color)
        Window:SetAccentColor(color)
    end,
})

ESP_Group:AddButton({
    Name     = "Preview ESP",
    Icon     = Icons.Eye,
    Callback = function()
        Window:SetESPPreviewEnabled(true)
        Window:SetESPPreviewData({
            Name          = "TargetPlayer",
            Distance      = 156,
            HealthPercent = 0.75,
            Color         = espColor:Get(),
            ShowBox       = true,
            ShowName      = true,
            ShowHealth    = true,
            ShowDistance  = true,
        })
    end,
})

-- ============================================================
-- 7. SYSTEM TAB (Config Management)
-- ============================================================
local ConfigTab = MiscSection:AddTab({
    Name        = "Configuration",
    Description = "Save and load settings profiles.",
    Icon        = Icons.Save,
})

local ConfigGroup = ConfigTab:AddGroup({
    Name = "Profiles",
    Side = "Left",
    Icon = Icons.Save,
})

local configInput = ConfigGroup:AddTextInput({
    Name        = "Profile Name",
    Placeholder = "my-preset",
    Default     = "default",
    Flag        = "System.Config.Name",
})

ConfigGroup:AddButton({
    Name     = "Save Profile",
    Icon     = Icons.Save,
    Callback = function()
        local name = configInput:Get()
        local ok = Window:SaveConfig(name)
        Window:Notify({
            Title       = ok and "Saved" or "Failed",
            Description = ok and "Profile '" .. name .. "' saved." or "Save error.",
            Duration    = 3,
            Icon        = Icons.Save,
        })
    end,
})

ConfigGroup:AddButton({
    Name     = "Load Profile",
    Icon     = Icons.Play,
    Callback = function()
        local name = configInput:Get()
        local ok = Window:LoadConfig(name)
        Window:Notify({
            Title       = ok and "Loaded" or "Failed",
            Description = ok and "Profile '" .. name .. "' loaded." or "Load error.",
            Duration    = 3,
            Icon        = Icons.Play,
        })
    end,
})

ConfigGroup:AddDivider()

ConfigGroup:AddButton({
    Name     = "Reset to Defaults",
    Icon     = Icons.Bell,
    Callback = function()
        aimEnabled:Set(false, true)
        fovSlider:Set(120, true, true)
        aimMode:Set("Closest", true)
        smoothSlider:Set(0.15, true, true)
        checksTeam:Set(true, true)
        checksVis:Set(true, true)
        checksWall:Set(false, true)
        espLayers:Set({"Box", "Name"}, true)
        espColor:Set(Color3.fromRGB(99, 130, 255), true)
        Window:Notify({
            Title       = "Reset",
            Description = "All settings restored to defaults.",
            Duration    = 3,
        })
    end,
})

-- ============================================================
-- 8. BOOT NOTIFICATION
-- ============================================================
Window:Notify({
    Title       = "Napoleon Loaded",
    Description = "PvP Suite initialized. Press RightShift to toggle.",
    Duration    = 5,
    Icon        = Icons.Home,
})
```

---

## API Quick Reference

### Hierarchy

| Method | Returns | Purpose |
|--------|---------|---------|
| `Library.new(opts)` | `Window` | Create the hub window |
| `Window:CreateSection(opts)` | `Section` | Add a sidebar category |
| `Section:AddTab(opts)` | `Tab` | Add a feature tab |
| `Tab:AddGroup(opts)` | `Group` | Add a panel container |
| `Group:AddTab(opts)` | `GroupTab` | Add internal group tabs |

### Window Methods

| Method | Purpose |
|--------|---------|
| `Window:SetToggleKey(key)` | Bind show/hide key |
| `Window:SetAccentColor(color)` | Update theme color |
| `Window:Notify(opts)` | Show toast notification |
| `Window:SaveConfig(name)` | Persist control states |
| `Window:LoadConfig(name)` | Restore control states |
| `Window:Destroy()` | Cleanup and remove |
| `Window:SetESPPreviewEnabled(b)` | Toggle ESP preview |
| `Window:SetESPPreviewData(data)` | Update preview data |

### Controls

| Method | Returns | Purpose |
|--------|---------|---------|
| `Group:AddToggle(opts)` | `Toggle` | On/off switch |
| `Group:AddButton(opts)` | — | Action button |
| `Group:AddLabel(opts)` | — | Static text |
| `Group:AddDivider()` | — | Visual separator |
| `Group:AddDropdown(opts)` | `Dropdown` | Single selection |
| `Group:AddMultiDropdown(opts)` | `MultiDropdown` | Multi selection |
| `Group:AddSlider(opts)` | `Slider` | Numeric input |
| `Group:AddKeybind(opts)` | `Keybind` | Key binding |
| `Group:AddKeybindToggle(opts)` | `KeybindToggle` | Key + toggle |
| `Group:AddTextInput(opts)` | `TextInput` | Live text field |
| `Group:AddTextbox(opts)` | `Textbox` | Enter-submitted text |
| `Group:AddColorPicker(opts)` | `ColorPicker` | Color selector |

### Control Methods

```lua
-- Toggle, Dropdown, MultiDropdown, TextInput, ColorPicker
control:Set(value, silent)
control:Get()

-- Slider
slider:Set(value, instant, silent)
slider:Get()

-- Dropdown, MultiDropdown
dropdown:UpdateOptions(options)
```

---

## Changelog

### v2.0.0 — Napoleon Release
- **Rebranded** from Flow UI to Napoleon UI Library
- **New source URL:** `https://raw.githubusercontent.com/VoidDeveloper67/Nepo/refs/heads/main/tuf%2B3.txt`
- **Group Tabs:** Added internal tabbing within groups for complex UIs
- **ESP Preview:** Built-in ESP simulation for visual configuration
- **Dynamic Dropdowns:** Auto-refreshing options via `OptionsProvider`
- **Enhanced Configuration:** Auto-save toggle and improved flag management
- **Silent Mode:** All setters support silent updates to prevent callback loops
- **Motion System:** Configurable animation styles (`Normal`, `Smooth`, `Instant`)

### v1.x — Flow UI Legacy
- Initial windowed hub framework
- Sections, tabs, and groups
- Basic control toolkit
- Configuration save/load

---

## Notes

- **Group Tabs** require the latest source version. Always use `if Group.AddTab then` guards for backward compatibility.
- **Icons:** The Lydie/Lucide 773.../774... asset ID range provides the most reliable, high-resolution icons.
- **Configs:** Controls without explicit `Flag` values receive auto-generated keys based on their hierarchy path and name.
- **Silent Mode:** Pass `true` as the second argument to any `Set` method to suppress callbacks and prevent infinite loops.
- **Development:** Use a local `neposource.lua` file during development for instant iteration, falling back to the hosted source for distribution.

---

*Built for precision. Designed for dominance. — Napoleon UI Library*
