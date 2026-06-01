# Material

An additive upgrade layer over [TopbarPlus v3](https://github.com/1ForeverHD/TopbarPlus). Zero original files modified — all new functionality is injected via `IconPatch` at require-time.

![version](https://img.shields.io/badge/version-1.0.0--beta-6C3EF4?style=for-the-badge)
![luau](https://img.shields.io/badge/luau-strict--typed-6C3EF4?style=for-the-badge)
![built by](https://img.shields.io/badge/built%20by-Plinko%20Labs-6C3EF4?style=for-the-badge)

---

## Contents

1. [Installation](#installation)
2. [Architecture Overview](#architecture-overview)
3. [Theme System](#theme-system)
4. [Badge System](#badge-system)
5. [Element Types](#element-types)
6. [Transitions](#transitions)
7. [Event Shorthands](#event-shorthands)
8. [Group API](#group-api)
9. [Overflow Control](#overflow-control)
10. [Full Example](#full-example)
11. [API Reference](#api-reference)

---

## Installation

1. Drop the `Material` folder into `ReplicatedStorage` (or wherever you keep TopbarPlus).
2. Require `Icon` **through** Material — the patch self-applies on first require:

```lua
local Icon = require(game.ReplicatedStorage.TopbarPlus.Icon)
local _    = require(game.ReplicatedStorage.Material.IconPatch)
-- After this point Icon has all Material methods injected.
```

> **Important:** Always require `IconPatch` before creating any icons. The patch is idempotent — requiring it multiple times is safe.

---

## Architecture Overview

```
TopbarPlus (untouched)
    └── Icon.lua          ← original module, zero modifications
Material
    └── IconPatch.lua     ← injects new static + instance methods into Icon
        ├── Theme System  (registerTheme / applyTheme / :theme())
        ├── Badge System  (:badge() / :clearBadge())
        ├── Elements      (Icon.Element constructors)
        ├── Transitions   (:transition() + Icon.Transitions presets)
        ├── Events        (:onSelect / :onDeselect / :onChange)
        ├── Groups        (Icon.group() / :group())
        └── Overflow      (Icon.setOverflowEnabled / :overflow())
```

<!-- SCREENSHOT: architecture diagram or folder structure in Roblox Studio Explorer -->
<!-- INSERT SCREENSHOT HERE -->

---

## Theme System

Register a named theme once, apply it anywhere. Token groups map directly to TopbarPlus's internal modification array — no more stringly-typed arrays.

### Registering a theme

```lua
Icon.registerTheme("Kibishii", {
    Widget = {
        BackgroundColor3       = Color3.fromRGB(12, 12, 14),
        BackgroundTransparency = 0.05,
        BorderSize             = 4,
        MinimumWidth           = 44,
        MinimumHeight          = 44,
    },
    Label = {
        TextColor3 = Color3.new(1, 1, 1),
        TextSize   = 15,
    },
    Badge = {
        BackgroundColor3 = Color3.fromRGB(220, 40, 40),
        TextColor3       = Color3.new(1, 1, 1),
    },
    Dropdown = {
        BackgroundColor3 = Color3.fromRGB(12, 12, 14),
        MaxIcons         = 5,
    },
})
```

### Applying a theme

```lua
-- Apply to ALL icons (sets the base theme for every icon created after this call)
Icon.applyTheme("Kibishii")

-- Apply to ONE icon only (does not affect others)
Icon.new()
    :theme("Kibishii")

-- Apply with inline overrides on top of a registered theme
Icon.new()
    :theme("Kibishii", {
        Widget = { BackgroundColor3 = Color3.fromRGB(30, 10, 60) },
    })
```

### Token groups

| Group | Tokens | Maps to |
|---|---|---|
| `Widget` | `BackgroundColor3`, `BackgroundTransparency`, `BorderSize`, `MinimumWidth`, `MinimumHeight`, `DesiredWidth` | `IconButton`, `Widget` instances |
| `Label` | `TextColor3`, `TextSize`, `FontFace` | `IconLabel` |
| `Badge` | `BackgroundColor3`, `TextColor3`, `TextSize` | `Notice`, `NoticeLabel` |
| `Dropdown` | `BackgroundColor3`, `BackgroundTransparency`, `MaxIcons` | `Dropdown` |
| `Menu` | `MaxIcons` | `Menu` |

<!-- SCREENSHOT: side-by-side of default theme vs Kibishii dark theme applied to a row of icons -->
<!-- INSERT SCREENSHOT HERE -->

---

## Badge System

Replaces the raw `:notify()` bubble with typed, stateful badge modes. Raw `:notify()` still works as before for backward compatibility.

### dot mode
A small presence indicator — no number, just a colored dot.

```lua
icon:badge("dot")
icon:clearBadge()
```

<!-- SCREENSHOT: dot badge on a topbar icon -->
<!-- INSERT SCREENSHOT HERE -->

### count mode
Numeric accumulation with overflow display (`99+`).

```lua
icon:badge("count", 3)            -- set absolute value to 3
icon:badge("count", "+1")         -- increment by 1
icon:badge("count", "+5")         -- increment by 5
icon:badge("count", "+1", {
    clearAfter = 5,               -- auto-clear after 5 seconds
})
```

Count values above 99 automatically display as `99+`.

<!-- SCREENSHOT: count badge showing "3", then "99+" -->
<!-- INSERT SCREENSHOT HERE -->

### custom mode
Full control — image, text, and color.

```lua
icon:badge("custom", {
    image = 12345678,             -- asset ID (number) or rbxassetid:// string
    text  = "NEW",
    color = Color3.fromHex("#6C3EF4"),
})
```

<!-- SCREENSHOT: custom badge with image and text label -->
<!-- INSERT SCREENSHOT HERE -->

### Clearing badges

```lua
icon:clearBadge()   -- removes badge, resets all internal state
```

---

## Element Types

Typed constructors for dropdown and menu children. Every constructor returns a real `Icon` instance — no API surface changes on `:setDropdown()` or `:setMenu()`.

```lua
local E = Icon.Element

icon:setDropdown({
    E.header("Loadouts"),
    E.item({ label = "Berserker",  image = 11111, onSelect = onBerserker }),
    E.item({ label = "Rogue",      image = 22222, onSelect = onRogue }),
    E.divider(),
    E.toggle({ label = "Auto-equip",  value = true,  onChange = onToggle }),
    E.slider({ label = "Volume",      min = 0, max = 1, default = 0.8, step = 0.05, onChange = onVolume }),
    E.display({ label = "Gold",       value = goldValue }),
    E.input({ label = "Search",       placeholder = "item name...", onSubmit = onSearch }),
})
```

### Constructor reference

| Constructor | Config keys | Description |
|---|---|---|
| `E.item(cfg)` | `label`, `image?`, `onSelect?` | Standard selectable row |
| `E.header(text)` | *(string)* | Non-interactive section label |
| `E.divider()` | — | Visual separator line |
| `E.toggle(cfg)` | `label`, `value`, `onChange` | Row with animated boolean toggle |
| `E.slider(cfg)` | `label`, `min`, `max`, `default`, `step?`, `onChange` | Row with draggable value slider |
| `E.display(cfg)` | `label`, `value` | Read-only value row, supports reactive binding |
| `E.input(cfg)` | `label`, `placeholder?`, `onSubmit` | Text input row, fires on Enter |

### Reactive binding for `E.display`

`value` can be a static value, or any object that exposes a `.changed` or `.Changed` signal (compatible with `Ink.Var`, `BindableValue`, `IntValue`, `StringValue`, etc.):

```lua
local goldVar = Instance.new("IntValue")
goldVar.Value = 500

E.display({ label = "Gold", value = goldVar })
-- Updates automatically whenever goldVar.Value changes
```

<!-- SCREENSHOT: dropdown open showing header, items, divider, toggle, slider, display row -->
<!-- INSERT SCREENSHOT HERE -->

<!-- SCREENSHOT: input element with cursor active -->
<!-- INSERT SCREENSHOT HERE -->

---

## Transitions

Override the dropdown open/close tween per-icon or apply a named preset.

### Named presets

```lua
icon:transition(Icon.Transitions.Bouncy)
icon:transition(Icon.Transitions.Smooth)
icon:transition(Icon.Transitions.Snap)
icon:transition(Icon.Transitions.Fade)
icon:transition(Icon.Transitions.None)
icon:transition(Icon.Transitions.Default)
```

### Inline config

```lua
icon:transition({
    open  = { style = Enum.EasingStyle.Spring, direction = Enum.EasingDirection.Out, time = 0.3 },
    close = { style = Enum.EasingStyle.Quint,  direction = Enum.EasingDirection.In,  time = 0.15 },
})
```

### Preset table

| Preset | Open | Close | Character |
|---|---|---|---|
| `Default` | Exponential Out 0.07s | Exponential Out 0s | Stock TopbarPlus feel |
| `Smooth` | Quint Out 0.18s | Quint In 0.12s | Polished, fluid |
| `Bouncy` | Bounce Out 0.35s | Quint In 0.12s | Playful, energetic |
| `Snap` | Linear Out 0.04s | Linear Out 0s | Instant, crisp |
| `Fade` | Quad Out 0.22s | Quad In 0.14s | Soft, subtle |
| `None` | Linear 0s | Linear 0s | Zero animation |

<!-- SCREENSHOT: GIF or sequence of dropdown opening with Bouncy preset -->
<!-- INSERT SCREENSHOT HERE -->

---

## Event Shorthands

Chainable event binding without needing to call `:bindEvent()` directly.

```lua
icon:onSelect(function(icon)
    print("selected:", icon.name)
end)

icon:onDeselect(function(icon)
    print("deselected:", icon.name)
end)

-- Fires on every state transition including "Viewing" (hover)
icon:onChange(function(icon, state)
    print("state changed to:", state)  -- "Selected" | "Deselected" | "Viewing"
end)
```

All three are chainable and can be stacked:

```lua
Icon.new()
    :setLabel("Shop")
    :onSelect(function(icon) openShop() end)
    :onDeselect(function(icon) closeShop() end)
    :onChange(function(icon, state) updateIndicator(state) end)
```

---

## Group API

Groups let you control multiple icons together. An icon registers itself into a named group via `:group("name")` on the icon chain; the group controller is retrieved via `Icon.group("name")`.

### Creating and adding icons

```lua
local shop = Icon.new()
    :setImage(111)
    :setLabel("Shop")
    :group("HUD")          -- registers into group "HUD", returns the icon (chainable)

local inv = Icon.new()
    :setImage(222)
    :setLabel("Inventory")
    :group("HUD")

-- Retrieve the group controller (creates it if needed, same object every time)
local hud = Icon.group("HUD")
```

### Group controller methods

```lua
hud:setEnabled(false)           -- hide all icons in the group
hud:setEnabled(true)            -- show all icons in the group

hud:applyTheme("Kibishii")      -- apply a registered theme to all icons
hud:applyTheme("Kibishii", {    -- apply with per-group overrides
    Widget = { BackgroundColor3 = Color3.fromRGB(20, 20, 24) },
})

hud:exclusiveSelect(true)       -- mutual deselect: selecting one deselects all others
hud:exclusiveSelect(false)      -- allow multiple selected simultaneously

hud:isolate()                   -- shorthand for exclusiveSelect(true)
hud:unisolate()                 -- shorthand for exclusiveSelect(false)

hud:destroy()                   -- destroy all icons in the group and remove the group
```

> Icons automatically remove themselves from their group when destroyed individually. The group persists until `:destroy()` is called on it.

<!-- SCREENSHOT: group of HUD icons, one selected -->
<!-- INSERT SCREENSHOT HERE -->

---

## Overflow Control

TopbarPlus's built-in overflow system collapses icons into a `...` menu when the screen is too narrow. This is useful by default but can break custom layouts — Material gives you full manual control.

### Global kill switch

```lua
-- Disable overflow collapsing entirely for all icons
Icon.setOverflowEnabled(false)

-- Re-enable (default)
Icon.setOverflowEnabled(true)
```

### Per-icon exclusion

```lua
-- Exclude a specific icon from ever being collapsed into overflow
Icon.new()
    :setLabel("Always Visible")
    :overflow(false)        -- this icon will never be moved into the overflow menu

-- Allow an icon to participate in overflow (default)
icon:overflow(true)
```

### Why you need this

On small screens (< ~600px viewport width) TopbarPlus starts moving icons into an overflow `...` button automatically. If your game has a fixed-layout topbar, or if certain icons (health, currency) must always be visible, exclude them:

```lua
-- Always-visible icons
local health = Icon.new():setLabel("HP"):overflow(false)
local gold   = Icon.new():setLabel("Gold"):overflow(false)

-- Collapsible utility icons
local settings = Icon.new():setLabel("Settings")  -- overflows normally
local map      = Icon.new():setLabel("Map")         -- overflows normally
```

<!-- SCREENSHOT: narrow viewport showing overflow "..." button with some icons collapsed -->
<!-- INSERT SCREENSHOT HERE -->

<!-- SCREENSHOT: same viewport with overflow(false) icons staying visible -->
<!-- INSERT SCREENSHOT HERE -->

---

## Full Example

```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local Icon = require(ReplicatedStorage.TopbarPlus.Icon)
local _    = require(ReplicatedStorage.Material.IconPatch)
local E    = Icon.Element

-- ── 1. Register themes ───────────────────────────────────────────────────────

Icon.registerTheme("Dark", {
    Widget  = {
        BackgroundColor3       = Color3.fromRGB(10, 10, 12),
        BackgroundTransparency = 0.1,
        BorderSize             = 3,
    },
    Label   = { TextColor3 = Color3.new(1, 1, 1), TextSize = 14 },
    Badge   = { BackgroundColor3 = Color3.fromHex("#FF3030") },
    Dropdown = { BackgroundColor3 = Color3.fromRGB(10, 10, 12) },
})

Icon.applyTheme("Dark")

-- ── 2. Disable overflow for critical HUD icons ───────────────────────────────

-- ── 3. Build icons ───────────────────────────────────────────────────────────

local currency = Icon.new()
    :setImage(rbxassetid_gold)
    :setLabel("500")
    :overflow(false)                   -- always visible, never collapsed
    :group("HUD")

local menu = Icon.new()
    :setImage(12345)
    :setLabel("Menu")
    :align("Left")
    :transition(Icon.Transitions.Smooth)
    :badge("count", "+1")
    :group("HUD")
    :onSelect(function(icon)
        icon:clearBadge()
    end)
    :setDropdown({
        E.header("Equipment"),
        E.item({ label = "Weapons",    onSelect = function() end }),
        E.item({ label = "Armor",      onSelect = function() end }),
        E.divider(),
        E.toggle({ label = "Auto-sort", value = false, onChange = function(v) print(v) end }),
        E.slider({ label = "Zoom",      min = 1, max = 5, default = 2,
                   onChange = function(v) print(v) end }),
        E.display({ label = "Ping",    value = pingValue }),
        E.input({ label = "Search",    placeholder = "item name...",
                  onSubmit = function(t) searchItems(t) end }),
    })

-- ── 4. Group operations ──────────────────────────────────────────────────────

local hud = Icon.group("HUD")
hud:exclusiveSelect(true)   -- one open at a time
```

---

## API Reference

### Static methods on `Icon`

| Method | Signature | Description |
|---|---|---|
| `Icon.registerTheme` | `(name: string, tokens: table)` | Register a named theme from token groups |
| `Icon.applyTheme` | `(name: string)` | Apply a theme to all icons as the new base |
| `Icon.group` | `(name: string) → GroupController` | Get or create a named group controller |
| `Icon.setOverflowEnabled` | `(bool: boolean)` | Globally enable/disable overflow collapsing |
| `Icon.Transitions` | table | Preset transition configs: `Default`, `Smooth`, `Bouncy`, `Snap`, `Fade`, `None` |
| `Icon.Element` | table | Element constructors: `item`, `header`, `divider`, `toggle`, `slider`, `display`, `input` |

### Instance methods (new in Material)

| Method | Signature | Description |
|---|---|---|
| `:theme` | `(name, overrides?)` | Apply a registered theme, optionally overriding tokens |
| `:badge` | `(mode, value?, options?)` | Set badge: `"dot"`, `"count"`, `"custom"` |
| `:clearBadge` | `()` | Remove badge and reset all badge state |
| `:transition` | `(pair)` | Set open/close tween config or named preset |
| `:onSelect` | `(fn)` | Shorthand for `bindEvent("selected", fn)` |
| `:onDeselect` | `(fn)` | Shorthand for `bindEvent("deselected", fn)` |
| `:onChange` | `(fn)` | Fires on every state change including `"Viewing"` |
| `:group` | `(name)` | Add this icon to a named group (chainable, returns icon) |
| `:standalone` | `()` | Disable auto-deselect when another icon is selected |
| `:overflow` | `(bool)` | Set whether this icon participates in overflow collapsing |

### GroupController methods

| Method | Signature | Description |
|---|---|---|
| `:add` | `(icon)` | Manually add an icon to the group |
| `:setEnabled` | `(bool)` | Show/hide all icons in the group |
| `:applyTheme` | `(name, overrides?)` | Apply a theme to all icons in the group |
| `:exclusiveSelect` | `(bool)` | Enable/disable mutual deselect within the group |
| `:isolate` | `()` | Alias for `exclusiveSelect(true)` |
| `:unisolate` | `()` | Alias for `exclusiveSelect(false)` |
| `:destroy` | `()` | Destroy all icons in the group |

---

Built by [Plinko Labs](https://github.com/NotKisoMomo)
