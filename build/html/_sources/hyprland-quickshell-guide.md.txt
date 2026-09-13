# Hyprland (Lua config) + Quickshell: A Practical Guide

Updated for the Lua configuration introduced in Hyprland 0.55. Syntax here comes from the official example config (`hyprwm/Hyprland/example/hyprland.lua`).

---

## 1. Lua vs the legacy .conf

Hyprland gained Lua configuration in April 2026, shipping in 0.55. Lua is optional: if there's no `hyprland.lua`, the old `hyprland.conf` loads as before. If `hyprland.lua` does exist, it's loaded instead.

Two things that trip people up:

- The check for which config to use happens **once at startup**. So `hyprctl reload` won't switch formats — you need a full restart (log out/in, or reboot).
- If you delete `hyprland.conf` and it keeps regenerating, that's the same issue. Quit and start Hyprland fully rather than reloading.

If you're migrating an existing `.conf`, `hyprmorph` (on LuaRocks) is a converter that preserves comments and order, warns about ambiguous constructs, and carries unsafe lines forward as Lua comments for manual review.

---

## 2. File Locations

| Tool | Config root |
|---|---|
| Hyprland | `~/.config/hypr/hyprland.lua` |
| Quickshell | `~/.config/quickshell/` |

The official example explicitly recommends splitting the config into multiple files — create them separately and pull them in with `require("myColors")`.

A sensible split:
```
~/.config/hypr/
├── hyprland.lua      # entry point, requires the rest
├── monitors.lua
├── binds.lua
├── rules.lua
└── colors.lua
```

---

## 3. The `hl` API — Core Functions

Everything is exposed through the global `hl` table.

| Function | Purpose |
|---|---|
| `hl.config({...})` | Set config variables (general, decoration, input, misc, layouts) |
| `hl.bind(key, dispatcher, opts)` | Keybinding |
| `hl.monitor({...})` | Monitor setup |
| `hl.env(name, value)` | Environment variable |
| `hl.exec_cmd(cmd)` | Run a command |
| `hl.on(event, fn)` | Event hook (e.g. autostart) |
| `hl.window_rule({...})` | Window rule |
| `hl.layer_rule({...})` | Layer rule |
| `hl.workspace_rule({...})` | Workspace rule |
| `hl.curve(name, {...})` | Animation curve (bezier or spring) |
| `hl.animation({...})` | Animation definition |
| `hl.device({...})` | Per-device input config |
| `hl.gesture({...})` | Touchpad gestures |
| `hl.permission(path, kind, action)` | Permissions |

---

## 4. Monitors

```lua
hl.monitor({
    output = "",
    mode = "preferred",
    position = "auto",
    scale = "auto",
})
```

Explicit multi-monitor setup, one call per output:
```lua
hl.monitor({ output = "DP-1",     mode = "2560x1440@144", position = "0x0",    scale = 1 })
hl.monitor({ output = "HDMI-A-1", mode = "1920x1080@60",  position = "2560x0", scale = 1 })
```

Get exact output names with `hyprctl monitors`.

---

## 5. Look and Feel

Config values are nested Lua tables now, not flat `key = value` lines:

```lua
hl.config({
    general = {
        gaps_in = 5,
        gaps_out = 20,
        border_size = 2,
        col = {
            active_border = { colors = {"rgba(33ccffee)", "rgba(00ff99ee)"}, angle = 45 },
            inactive_border = "rgba(595959aa)",
        },
        resize_on_border = false,
        allow_tearing = false,
        layout = "dwindle",
    },
    decoration = {
        rounding = 10,
        rounding_power = 2,
        active_opacity = 1.0,
        inactive_opacity = 1.0,
        shadow = {
            enabled = true,
            range = 4,
            render_power = 3,
            color = 0xee1a1a1a,
        },
        blur = {
            enabled = true,
            size = 3,
            passes = 1,
            vibrancy = 0.1696,
        },
    },
    animations = {
        enabled = true,
    },
})
```

Note the gradient border syntax — a table of colors plus an angle, much cleaner than the old string form.

---

## 6. Keybindings

`hl.bind(keyString, dispatcher, opts)`. Key combos are a single string joined with `+`:

```lua
local mainMod = "SUPER"
local terminal = "kitty"
local fileManager = "dolphin"
local menu = "hyprlauncher"

hl.bind(mainMod .. " + Q", hl.dsp.exec_cmd(terminal))
hl.bind(mainMod .. " + C", hl.dsp.window.close())
hl.bind(mainMod .. " + E", hl.dsp.exec_cmd(fileManager))
hl.bind(mainMod .. " + V", hl.dsp.window.float({ action = "toggle" }))
hl.bind(mainMod .. " + R", hl.dsp.exec_cmd(menu))
hl.bind(mainMod .. " + P", hl.dsp.window.pseudo())
hl.bind(mainMod .. " + J", hl.dsp.layout("togglesplit"))
```

### Dispatchers (`hl.dsp.*`)

| Dispatcher | Purpose |
|---|---|
| `hl.dsp.exec_cmd(cmd)` | Run a command |
| `hl.dsp.window.close()` | Close active window |
| `hl.dsp.window.float({ action = "toggle" })` | Toggle floating |
| `hl.dsp.window.pseudo()` | Pseudo-tile |
| `hl.dsp.window.move({ workspace = n })` | Move window to workspace |
| `hl.dsp.window.drag()` | Mouse drag move |
| `hl.dsp.window.resize()` | Mouse drag resize |
| `hl.dsp.focus({ direction = "left" })` | Move focus |
| `hl.dsp.focus({ workspace = n })` | Switch workspace |
| `hl.dsp.workspace.toggle_special("magic")` | Scratchpad |
| `hl.dsp.layout("togglesplit")` | Layout command |
| `hl.dsp.exit()` | Exit Hyprland |

### Focus movement
```lua
hl.bind(mainMod .. " + left",  hl.dsp.focus({ direction = "left" }))
hl.bind(mainMod .. " + right", hl.dsp.focus({ direction = "right" }))
hl.bind(mainMod .. " + up",    hl.dsp.focus({ direction = "up" }))
hl.bind(mainMod .. " + down",  hl.dsp.focus({ direction = "down" }))
```

### Workspaces — where Lua actually pays off

Instead of twenty near-identical lines, loop:

```lua
for i = 1, 10 do
    local key = i % 10  -- 10 maps to key 0
    hl.bind(mainMod .. " + " .. key, hl.dsp.focus({ workspace = i }))
    hl.bind(mainMod .. " + SHIFT + " .. key, hl.dsp.window.move({ workspace = i }))
end
```

### Scratchpad
```lua
hl.bind(mainMod .. " + S", hl.dsp.workspace.toggle_special("magic"))
hl.bind(mainMod .. " + SHIFT + S", hl.dsp.window.move({ workspace = "special:magic" }))
```

### Mouse binds
Mouse binds use an opts table with `mouse = true` rather than a separate `bindm` keyword:
```lua
hl.bind(mainMod .. " + mouse_down", hl.dsp.focus({ workspace = "e+1" }))
hl.bind(mainMod .. " + mouse_up",   hl.dsp.focus({ workspace = "e-1" }))
hl.bind(mainMod .. " + mouse:272", hl.dsp.window.drag(),   { mouse = true })
hl.bind(mainMod .. " + mouse:273", hl.dsp.window.resize(), { mouse = true })
```

### Media keys
The old `bindl`/`binde` variants are opts now — `locked = true` (works while locked), `repeating = true` (fires on hold):
```lua
hl.bind("XF86AudioRaiseVolume", hl.dsp.exec_cmd("wpctl set-volume -l 1 @DEFAULT_AUDIO_SINK@ 5%+"), { locked = true, repeating = true })
hl.bind("XF86AudioLowerVolume", hl.dsp.exec_cmd("wpctl set-volume @DEFAULT_AUDIO_SINK@ 5%-"), { locked = true, repeating = true })
hl.bind("XF86AudioMute", hl.dsp.exec_cmd("wpctl set-mute @DEFAULT_AUDIO_SINK@ toggle"), { locked = true, repeating = true })
hl.bind("XF86MonBrightnessUp",   hl.dsp.exec_cmd("brightnessctl -e4 -n2 set 5%+"), { locked = true, repeating = true })
hl.bind("XF86MonBrightnessDown", hl.dsp.exec_cmd("brightnessctl -e4 -n2 set 5%-"), { locked = true, repeating = true })

-- requires playerctl
hl.bind("XF86AudioPlay", hl.dsp.exec_cmd("playerctl play-pause"), { locked = true })
hl.bind("XF86AudioNext", hl.dsp.exec_cmd("playerctl next"), { locked = true })
hl.bind("XF86AudioPrev", hl.dsp.exec_cmd("playerctl previous"), { locked = true })
```

### Bind handles
`hl.bind` returns a handle you can toggle at runtime — useful for modal/contextual binds:
```lua
local closeWindowBind = hl.bind(mainMod .. " + C", hl.dsp.window.close())
closeWindowBind:set_enabled(false)
```

---

## 7. Autostart

Autostart is an event hook now, not `exec-once`:

```lua
hl.on("hyprland.start", function()
    hl.exec_cmd("quickshell")
    hl.exec_cmd("nm-applet")
    hl.exec_cmd("hyprpaper")
end)
```

---

## 8. Environment Variables

```lua
hl.env("XCURSOR_SIZE", "24")
hl.env("HYPRCURSOR_SIZE", "24")
```

---

## 9. Input & Devices

```lua
hl.config({
    input = {
        kb_layout = "us",
        kb_variant = "",
        kb_options = "",
        follow_mouse = 1,
        sensitivity = 0,   -- -1.0 to 1.0
        touchpad = {
            natural_scroll = false,
        },
    },
})

hl.gesture({
    fingers = 3,
    direction = "horizontal",
    action = "workspace",
})

hl.device({
    name = "epic-mouse-v1",
    sensitivity = -0.5,
})
```

---

## 10. Window Rules

Rules take a `match` table instead of the old comma-separated string, and each rule can be named:

```lua
hl.window_rule({
    name = "suppress-maximize-events",
    match = { class = ".*" },
    suppress_event = "maximize",
})

hl.window_rule({
    name = "float-pavucontrol",
    match = { class = "^(pavucontrol)$" },
    float = true,
})

hl.window_rule({
    name = "fix-xwayland-drags",
    match = {
        class = "^$",
        title = "^$",
        xwayland = true,
        float = true,
        fullscreen = false,
        pin = false,
    },
    no_focus = true,
})
```

Like binds, rules return handles: `local r = hl.window_rule({...})` then `r:set_enabled(false)`.

Layer rules follow the same shape:
```lua
hl.layer_rule({
    name = "no-anim-overlay",
    match = { namespace = "^my-overlay$" },
    no_anim = true,
})
```

Workspace rules:
```lua
hl.workspace_rule({ workspace = "w[tv1]", gaps_out = 0, gaps_in = 0 })
```

Find a window's class with `hyprctl activewindow` while it's focused.

---

## 11. Animations

Curves are defined by name, then referenced by animations. Both bezier and spring types:

```lua
hl.curve("easeOutQuint",  { type = "bezier", points = { {0.23, 1}, {0.32, 1} } })
hl.curve("almostLinear",  { type = "bezier", points = { {0.5, 0.5}, {0.75, 1} } })
hl.curve("quick",         { type = "bezier", points = { {0.15, 0}, {0.1, 1} } })
hl.curve("easy", { type = "spring", mass = 1, stiffness = 71.2633, dampening = 15.8273644 })

hl.animation({ leaf = "global",    enabled = true, speed = 10,   bezier = "default" })
hl.animation({ leaf = "windows",   enabled = true, speed = 4.79, spring = "easy" })
hl.animation({ leaf = "windowsIn", enabled = true, speed = 4.1,  spring = "easy", style = "popin 87%" })
hl.animation({ leaf = "fade",      enabled = true, speed = 3.03, bezier = "quick" })
hl.animation({ leaf = "workspaces", enabled = true, speed = 1.94, bezier = "almostLinear", style = "fade" })
```

---

## 12. Layouts

```lua
hl.config({ dwindle   = { preserve_split = true } })
hl.config({ master    = { new_status = "master" } })
hl.config({ scrolling = { fullscreen_on_one_column = true } })
```

---

## 13. Permissions

Permission changes require a Hyprland restart and are not applied on-the-fly, for security reasons.

```lua
hl.config({
    ecosystem = {
        enforce_permissions = true,
    },
})

hl.permission("/usr/(bin|local/bin)/grim", "screencopy", "allow")
hl.permission("/usr/(lib|libexec|lib64)/xdg-desktop-portal-hyprland", "screencopy", "allow")
hl.permission("/usr/(bin|local/bin)/hyprpm", "plugin", "allow")
```

---

## 14. LSP Setup (worth doing)

Since configs are Lua now, you get real editor tooling. Hyprland ships an `hl.meta.lua` stub — leave it in its stubs directory and point `.luarc.json` at that directory so your LSP resolves `hl.*` completions and catches typos before you restart the compositor.

---

## 15. `hyprctl` Commands

Unchanged by the Lua migration:

| Command | What it does |
|---|---|
| `hyprctl reload` | Reload config (does NOT switch .conf ↔ .lua) |
| `hyprctl monitors` | List monitors + resolution/refresh |
| `hyprctl clients` | List open windows + properties |
| `hyprctl activewindow` | Info on focused window (get class/title for rules) |
| `hyprctl dispatch workspace 3` | Jump to workspace 3 |
| `hyprctl keyword <option> <value>` | Change a value at runtime |

---

## 16. Quickshell Basics

Quickshell is QML-based — a tree of `.qml` files starting from `shell.qml`. Unaffected by the Hyprland Lua change.

```
~/.config/quickshell/
├── shell.qml           # entry point
├── modules/
│   ├── Bar.qml
│   ├── Workspaces.qml
│   ├── Clock.qml
│   └── SystemTray.qml
└── services/
```

Launch:
```bash
qs -c ~/.config/quickshell/shell.qml
```

From Hyprland autostart:
```lua
hl.on("hyprland.start", function()
    hl.exec_cmd("quickshell")
end)
```

### Core concepts
- **`PanelWindow`** — anchors a bar/panel to a screen edge.
- **`Variants`** — instantiate one bar per monitor automatically.
- **Process / `Quickshell.execDetached`** — shell out for data (`pactl`, `nmcli`, `/sys` for battery).
- Hot reload on file save — no restart while iterating.

### Minimal bar skeleton
```qml
import Quickshell
import QtQuick

PanelWindow {
    anchors {
        top: true
        left: true
        right: true
    }
    height: 32

    Rectangle {
        anchors.fill: parent
        color: "#1e1e2e"

        Text {
            anchors.centerIn: parent
            text: "hello quickshell"
            color: "white"
        }
    }
}
```

### Debugging
Run `qs` in a foreground kitty window while iterating — QML errors print straight to that terminal. Move it into autostart only once it's stable.

---

## 17. Reference

- Lua config announcement: https://hypr.land/news/26_lua/
- Official example config: https://github.com/hyprwm/Hyprland/blob/main/example/hyprland.lua
- Wiki (configuring): https://wiki.hypr.land/Configuring/Start/
- Quickshell docs: https://quickshell.outfoxxed.me/
- `hyprmorph` (.conf → .lua converter): https://luarocks.org/modules/junaga/hyprmorph
