# Material

An additive upgrade layer over [TopbarPlus v3](https://github.com/1ForeverHD/TopbarPlus). Zero original files modified -- all new functionality is injected via `IconPatch`.

![version](https://img.shields.io/badge/version-1.0.0--beta-6C3EF4?style=for-the-badge)
![luau](https://img.shields.io/badge/luau-strict--typed-6C3EF4?style=for-the-badge)
![built by](https://img.shields.io/badge/built%20by-Plinko%20Labs-6C3EF4?style=for-the-badge)

---

## Installation

Drop `Material` into your game. Require `Icon`:

```lua
local Icon  = require(path.Icon)
```
---

## Theme System

Register a named theme once, apply it anywhere. Tokens map to the internal TopbarPlus modification array automatically -- no more stringly-typed arrays.

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

-- apply to all icons (sets the base theme)
Icon.applyTheme("Kibishii")

-- apply to one icon only
Icon.new()
    :theme("Kibishii")

-- apply with inline overrides on top
Icon.new()
    :theme("Kibishii", {
        Widget = { BackgroundColor3 = Color3.fromRGB(30, 10, 60) },
    })
```

**Available token groups:** `Widget`, `Label`, `Badge`, `Dropdown`, `Menu`

---

## Badge System

Replaces the raw `:notify()` bubble with typed badge modes. `:notify()` still works as before.

```lua
-- dot -- small presence indicator, no number
icon:badge("dot")

-- count -- numeric accumulation
icon:badge("count", 3)          -- set to 3
icon:badge("count", "+1")       -- increment by 1
icon:badge("count", "+1", { clearAfter = 5 })  -- auto-clear after 5s

-- custom -- image + text
icon:badge("custom", {
    image = 12345678,
    text  = "NEW",
    color = Color3.fromHex("#6C3EF4"),
})

-- clear
icon:clearBadge()
```

---

## Element Types

Typed constructors for dropdown and menu children. Every constructor returns a real Icon instance -- no API changes needed on `:setDropdown()` or `:setMenu()`.

```lua
local E = Icon.Element

icon:setDropdown({
    E.header("Loadouts"),
    E.item({ label = "Berserker", image = 11111, onSelect = onBerserker }),
    E.item({ label = "Rogue",     image = 22222, onSelect = onRogue }),
    E.divider(),
    E.toggle({ label = "Auto-equip",  value = true,  onChange = onToggle }),
    E.slider({ label = "Volume",      min = 0, max = 1, default = 0.8, step = 0.05, onChange = onVolume }),
    E.display({ label = "Gold",       value = goldValue }),   -- reactive binding supported
    E.input({ label = "Search",       placeholder = "item name...", onSubmit = onSearch }),
})
```

| Constructor | Description |
|---|---|
| `Element.item(cfg)` | Standard selectable row |
| `Element.header(text)` | Non-interactive section label |
| `Element.divider()` | Visual separator line |
| `Element.toggle(cfg)` | Row with animated boolean toggle |
| `Element.slider(cfg)` | Row with draggable value slider |
| `Element.display(cfg)` | Read-only value row -- reactive binding via `.changed` or `.Changed` signal |
| `Element.input(cfg)` | Text input row, fires `onSubmit(text)` on Enter |

`Element.display` reactive binding is compatible with Ink.Var, BindableValue, and any object with a `.changed` or `.Changed` signal.

---

## Transitions

Override the dropdown open/close tween per icon, or use a named preset.

```lua
-- named preset
icon:transition(Icon.Transitions.Bouncy)

-- inline config
icon:transition({
    open  = { style = Enum.EasingStyle.Spring, direction = Enum.EasingDirection.Out, time = 0.3 },
    close = { style = Enum.EasingStyle.Quint,  direction = Enum.EasingDirection.In,  time = 0.15 },
})
```

**Presets:** `Default`, `Smooth`, `Bouncy`, `Snap`, `Fade`, `None`

---

## Event Shorthands

```lua
icon:onSelect(function(icon)   print("selected")   end)
icon:onDeselect(function(icon) print("deselected") end)
icon:onChange(function(icon, state) print(state)   end)
```

`:onSelect` and `:onDeselect` are direct shorthands for `:bindEvent`. `:onChange` fires on every state transition including `"Viewing"`.

---

## Group API

```lua
-- add an icon to a named group (chainable, stays on the icon chain)
local shop = Icon.new()
    :setImage(111)
    :setLabel("Shop")
    :group("HUD")

local inv = Icon.new()
    :setImage(222)
    :setLabel("Inventory")
    :group("HUD")

-- retrieve the group controller
local hud = Icon.group("HUD")

hud:setEnabled(false)              -- hide all
hud:setEnabled(true)               -- show all
hud:applyTheme("Kibishii")         -- bulk theme
hud:exclusiveSelect(true)          -- mutual deselect
hud:destroy()                      -- destroy all
```

---

## Full Example

```lua
local Icon = require(game.ReplicatedStorage.TopbarPlus.Icon)
local _    = require(game.ReplicatedStorage.Material.IconPatch)
local E    = Icon.Element

Icon.registerTheme("Dark", {
    Widget  = { BackgroundColor3 = Color3.fromRGB(10, 10, 12), BackgroundTransparency = 0.1 },
    Label   = { TextColor3 = Color3.new(1, 1, 1) },
    Badge   = { BackgroundColor3 = Color3.fromHex("#FF3030") },
})

Icon.applyTheme("Dark")

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
        E.slider({ label = "Zoom",      min = 1, max = 5, default = 2, onChange = function(v) print(v) end }),
    })
```

---

Built by [Plinko Labs](https://github.com/NotKisoMomo)
