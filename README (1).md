# MethaneUI / NH Library — Full API Docs

Internal notes for every feature in the current `Library.lua` build.

```lua
local Library = loadstring(game:HttpGet("YOUR_RAW_URL"))()
-- after load: getgenv().Library / _G.Library
```

---

## Bootstrap / paths

| Field | Default | Notes |
|-------|---------|--------|
| `Library.Directory` | `"MethaneUI"` | Root workspace folder |
| `Library.Brand` | `"methane"` | Config brand segment |
| `Library.GameName` | `"test"` | Config game segment |
| `Library.Folders.Assets` | `"/Assets"` | Fonts / assets |
| `Library.Folders.Configs` | `"/methane/cfgs/test"` | Rebuilt by `EnsureConfigFolders` |
| `Library.FontSize` | `9` | Default UI text size |
| `Library.MenuKeybind` | key string (default X / UI Bind) | Toggle UI |
| `Library.Theme` | Preset table | Live colors |
| `Library.Flags` | `{}` | All element flags |
| `Library.Animation` | `{ Time=0.3, Style="Quint", Direction="Out" }` | Tweens |

```lua
Library.Brand = "methane"
Library.GameName = "test"
Library:EnsureConfigFolders()
-- MethaneUI/
--   Assets/
--   methane/cfgs/test/   ← .json configs
```

```lua
Library:Exit()  -- destroy UI, threads, connections, clear getgenv().Library
```

---

## Icons (Lucide)

Uses Footagesus Icons (`Main-v2`), type `lucide`.

```lua
Library:ResolveIcon("crosshair")            -- name → rbxassetid
Library:ResolveIcon("rbxassetid://123")     -- passthrough
Library:ResolveIcon("lucide:eye")           -- strips prefix
```

---

## Theme keys

`Background`, `Outline`, `Border`, `Border 2`, `Light Border`, `Accent`, `Risky`, `Text`, `Inactive Text`, `Section`, `Element`, `Hovered Element`

```lua
Library.Theme.Accent = Color3.fromRGB(152, 188, 255)
Library:ChangeTheme("Accent", Library.Theme.Accent)
```

Also editable under **Settings → Other → Theming**.

---

## Configs (disk)

```lua
local json = Library:GetConfig()       -- flags + layout JSON string
Library:LoadConfig(json)
Library:GetConfigsList(dropdown)      -- refresh dropdown from disk
```

Files: `MethaneUI/{Brand}/cfgs/{GameName}/{Name}.json`

**Settings → Configs** UI:
- Dropdown of saved configs  
- Name textbox  
- Create / Delete / Load / Save  

---

## Share strings (no server)

Format: **`MTH` + base64(config JSON)**  
(HEX fallback encode if no base64 API on the executor)

```lua
local code, err = Library:ExportShareString()
-- code == "MTH...."

local ok, err = Library:ImportShareString(code)
```

**Settings → Configs → Share** (right column):
- **Export Share String** — builds `MTH…`, copies to clipboard  
- **Share string** textbox — paste  
- **Load Share String** — decode + `LoadConfig`  

### Length
Strings are long offline — the payload *is* the full config (flags + layout).  
Shorter codes need a host (upload JSON, return id). Optional future shrink: drop Layout, only non-defaults, or compress.

### Safety
- Does **not** expose Library source, loadstrings, cookies, or executor data  
- Only flag values + layout positions  
- If a user puts a webhook/token in a textbox and exports, that value is inside the string  
- Spotify token file is separate (`MethaneUI/token.txt`), not in share blob  
- Works when the script is **obfuscated** (same GetConfig / LoadConfig / encode path)

---

## Widgets

Create anytime; register into Settings → Widgets with `RegisterSettingsWidget`.

### Watermark
```lua
local W = Library:Watermark({ Name = "Methane" })
W:SetDynamicTextProvider(function(fps) return ("Methane | %dfps"):format(fps) end)
W:SetText / SetName / SetVisibility / Center / GetBounds
```

### KeybindList
```lua
local K = Library:KeybindList({ Name = "Keybinds" })
K:Add(key, name, mode)
K:SetVisibility / Center / SetText
```

### ESPPreview
```lua
local P = Library:ESPPreview({ Name = "ESP Preview" })
P:SetVisibility(true)
-- AddObject / BuildFromModel inside source
```

### TargetIndicator
```lua
local T = Library:TargetIndicator()
T:SetTarget(instOrNil)
T:AddItem("line")
T:SetVisibility / Center / SetPosition / GetBounds / ResetConnection
```

### RadarWidget
```lua
local R = Library:RadarWidget({ Name = "Radar" })
R:SetRange / SetSweep / SetHeading
R:Upsert(key, data) / Remove / Clear
R:SetFooter / SetText / SetVisibility / Center / GetBounds
```

### ConsoleLogger
```lua
local L = Library:ConsoleLogger({
  Name = "Console",
  Callback = function(text, log) log:AddOutput(text) end,
})
L:AddOutput / AddWarning / AddError / Clear
L:SetCommandCallback / SetVisibility / Center / SetText
```

### ModeratorList
```lua
local M = Library:ModeratorList({ Name = "Moderators" })
local e = M:Add("Player", "reason")
e:Set("reason") e:Remove()
M:Remove / Clear / SetVisibility / Center / SetText
```

### StatListWidget
```lua
local S = Library:StatListWidget({ Name = "Stats" })
S:SetLines({ "Kills: 0", "Deaths: 0" })
S:SetVisibility(true)
```

### ChargeShotWidget
```lua
local C = Library:ChargeShotWidget({ Name = "Charge Shot" })
C:SetRange(min, max) / SetValue / SetAlpha / SetFillColor
C:SetVisibility / Center / SetText / SetPosition
```

### InventoryViewer / SpotifyPlayer / Playerlist
```lua
Library:InventoryViewer({ Name = "Inventory" })
Library:SpotifyPlayer()
Library:Playerlist({ Name = "Players" })
-- :SetVisibility etc.
```

### RegisterSettingsWidget
```lua
Library:RegisterSettingsWidget({
  Name = "Watermark",
  Default = true,
  Callback = function(v) Watermark:SetVisibility(v) end,
  Settings = function(section, toggle) end, -- optional nested gear
})
```

---

## Window

```lua
local Window = Library:Window({
  Title = "Methane",                         -- floating header (centered, TextSize 13)
  ButtonName = "Main UI",                    -- dock label
  BrandName = "Methane",                     -- stored; panel text hidden
  BrandAlign = "Center",                     -- Left | Center | Right
  Logo = "rbxassetid://72404794660074",      -- or numeric id → rbxassetid
})
```

Panel brand row = **logo only**, **30×30**, centered (no “METHANE” text on the panel).

```lua
Window:SetOpen(bool)
Window:Center()
Window:SetTitle("Methane")
Window:SetDockText("Main UI")
Window:SetBrandName / SetBrandLogo / SetBrandAlign
Window:AddHeaderButton({ Text, Active, Callback })
Window:CreateSettingsPage()
Window:Page({ Name = "Main" })
```

---

## Page → SubPage → Section

```lua
local Page = Window:Page({ Name = "Main" })
Page:Turn(true)

local Sub = Page:SubPage({
  Name = "Combat",          -- internal
  Icon = "crosshair",       -- lucide or rbxassetid://  (icons ONLY on strip)
})
-- max 5 sub-tabs per page
-- selected icon = Accent, idle = Text

local Sec = Sub:Section({ Name = "Aimbot", Side = 1 })  -- 1 left, 2 right
```

---

## Elements

### Toggle
```lua
local T = Sec:Toggle({
  Name = "Enable",
  Flag = "Aim",
  Default = false,
  Risky = false,            -- red label only (dim off / bright on); NO hover popup
  Tooltip = "optional",     -- skipped when Risky
  Callback = function(v) end,
})
T:Set / SetVisibility / SetText
T:Keybind({ Flag, Mode = "Toggle"|"Hold"|"Always", Default = Enum.KeyCode.E, Callback })
T:Colorpicker({ Flag, Default = Color3, Callback = function(c, a) end })
local Opts = T:Settings()   -- gear panel (section-like)
Opts:Toggle / Slider / Dropdown / Label / ...
```

### Button
```lua
Sec:Button({ Name = "Run", Tooltip, Callback })
-- :Press / SetVisibility / SetText
```

### Slider
```lua
Sec:Slider({
  Name = "FOV", Flag = "FOV",
  Default = 90, Min = 1, Max = 360,
  Decimals = 0,             -- STEP size: 0 = int, 0.01 = hundredths
  Suffix = "°",
  Callback = function(v) end,
})
-- value label always on the row (not clipped)
-- :Set / GetSize / SetVisibility / SetText
```

### Dropdown
```lua
Sec:Dropdown({
  Name = "Hit", Flag = "Hit",
  Items = { "Head", "Torso" },
  Default = "Head",         -- or list if Multi
  Multi = false,
  Callback = function(v) end,
})
-- :Set / Add / Remove / Refresh / SetOpen / SetVisibility / SetText
```

### Label
```lua
Sec:Label({ Name = "Color" }):Colorpicker({ Flag, Default, Callback })
Sec:Label({ Name = "Bind" }):Keybind({ Flag, Mode, Default, Callback })
-- :SetText / SetVisibility
```

### Textbox
```lua
Sec:Textbox({
  Name = "Webhook", Flag = "WH",
  Placeholder = "https://...", Default = "",
  Callback = function(text) end,
})
-- :Set / SetText / SetVisibility
```

### Notification
```lua
Library:Notification("msg", 3, Library.Theme.Accent)
```

---

## Colorpicker

SV palette + hue + alpha, plus under the alpha bar:
- **Hex** `#RRGGBB` (FocusLost applies)
- **R / G / B** 0–255 (FocusLost applies)

Right-click chip → copy / paste color.  
Flag shape: `{ Color, Alpha, HexValue, Transparency }`.

```lua
cp:Set(Color3, alpha)
cp:SetOpen(bool)
cp:Update()
```

---

## Keybind

Modes: `Toggle` | `Hold` | `Always`  
Shows on KeybindList when configured.

```lua
kb:Set(Enum.KeyCode.E)
kb:SetMode("Hold")
kb:Press(bool)
```

---

## Settings page (`CreateSettingsPage`)

| Sub-tab | Icon | Content |
|---------|------|---------|
| Configs | `folder` | Disk configs + **Share** (MTH export/import) |
| Other | `settings` | Theming, UI Bind, blur/snow, Unload, animation speed, Widgets |

---

## Flags

| Element | `Library.Flags[Flag]` |
|---------|------------------------|
| Toggle | `boolean` |
| Slider | `number` |
| Dropdown | `string` or `string[]` |
| Textbox | `string` |
| Keybind | `{ Key, Mode, ... }` |
| Colorpicker | `{ Color, Alpha, HexValue, Transparency }` |

`LoadConfig` reapplies via internal `SetFlags`.

---

## Layout persistence

```lua
Library:RegisterLayout(id, { Instance, SaveSize, MinimumSize })
Library:GetLayoutConfig()
Library:ApplyLayoutConfig(t)
```

Included in GetConfig / share strings (unless you slim later).

---

## Background

```lua
Library:SetBackgroundBlurEnabled(bool)
Library:SetBackgroundSnowEnabled(bool)
Library:SetBackgroundEffectsVisible(bool, instant)
```

---

## Utility

```lua
Library:Round(n, step)           -- step 0 → integer
Library:SafeCall(fn, ...)
Library:Thread(fn)
Library:Connect(signal, fn)
Library:Tween / Fade / FadeDescendants
Library:MakeDraggable / MakeResizeable
Library:IsMouseOverFrame / IsClipped
Library:OnHover(enter, leave)
Library:AddToTheme / ChangeItemTheme
Library:BindToWindowVisibility(fn)
Library:OpenConfirmDialog({ Title, Message, ConfirmText, CancelText, AccentColor, Callback })
```

---

## Tree

```text
Library
└─ Window
   ├─ Logo 30×30 (centered, no panel title text)
   ├─ Floating header title
   ├─ Pages (text tabs)
   │  └─ SubPages ≤5 (lucide icons only)
   │     └─ Section Side 1|2
   │        ├─ Toggle [+Keybind][+Colorpicker][+Settings]
   │        ├─ Slider / Dropdown / Button / Label / Textbox
   │        └─ …
   ├─ Dock
   └─ CreateSettingsPage → Configs (disk + MTH share) | Other
Widgets…
```

---

## Minimal example

```lua
local Library = loadstring(game:HttpGet("RAW"))()

Library.Brand = "methane"
Library.GameName = "test"
Library:EnsureConfigFolders()

local Window = Library:Window({
  Title = "Methane",
  ButtonName = "Main UI",
  BrandName = "Methane",
  BrandAlign = "Center",
  Logo = "rbxassetid://72404794660074",
})

local Page = Window:Page({ Name = "Main" })
local Sub = Page:SubPage({ Name = "Combat", Icon = "crosshair" })
local Sec = Sub:Section({ Name = "Aimbot", Side = 1 })

local t = Sec:Toggle({ Name = "Enable", Flag = "Aim", Default = false, Callback = print })
t:Keybind({ Flag = "AimKey", Mode = "Hold", Default = Enum.KeyCode.E })
local opts = t:Settings()
opts:Slider({ Name = "FOV", Flag = "FOV", Default = 90, Min = 1, Max = 360, Decimals = 0, Suffix = "°" })

Window:CreateSettingsPage()
Library:Notification("Methane Loaded", 3, Library.Theme.Accent)
```

---

## Gotchas

1. Slider **`Decimals` = step**, not digit count (`0` ints, `0.01` hundredths).  
2. **Risky** = red text only; never attaches the old RISKY tooltip.  
3. SubPage = icons only; max **5**; use lucide names or `rbxassetid://`.  
4. Share strings start with **`MTH`**; long by design without a server.  
5. Share ≠ source leak; only settings. Obfuscation is fine.  
6. Keep secrets out of flagged textboxes if you export shares.  
7. This build includes patched sub-tabs, logo brand, Round fix, hex/RGB colorpicker, MTH share — keep this file if upstream raw is older.
