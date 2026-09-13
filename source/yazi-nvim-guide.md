# Yazi + Neovim: A Practical Guide

Tailored for Arch/CachyOS with `yay`, `kitty`, and `fish`.

---

## 1. Installation

```fish
# Yazi (you already have this installed)
yay -S yazi

# Neovim (latest stable)
yay -S neovim

# Recommended companion tools yazi/nvim lean on
yay -S ripgrep fd fzf unzip poppler ffmpeg 7zip jq imagemagick chafer 2>/dev/null
yay -S ripgrep fd fzf unzip poppler ffmpeg p7zip jq imagemagick
```

- `ripgrep`/`fd` — fast search, used by yazi's filter and by nvim plugins like Telescope.
- `poppler`, `ffmpeg`, `imagemagick` — image/video/PDF previews in yazi.
- `unzip`/`p7zip` — archive preview & extraction in yazi.

Check versions:
```fish
yazi --version
nvim --version
```

---

## 2. Config File Locations

| Tool | Config root |
|---|---|
| Yazi | `~/.config/yazi/` |
| Neovim | `~/.config/nvim/` |

Yazi's main files:
```
~/.config/yazi/
├── yazi.toml     # general settings (layout, preview, opener rules)
├── keymap.toml   # keybindings
├── theme.toml    # colors/icons
└── init.lua      # plugin loading / yazi-side Lua config
```

Neovim (if you go with a Lua-based setup, which is standard now):
```
~/.config/nvim/
├── init.lua
└── lua/
    ├── config/     # options, keymaps, autocmds
    └── plugins/    # one file per plugin (if using lazy.nvim)
```

---

## 3. Yazi Keybinds

Yazi ships with sane defaults in `keymap.toml`. Here are the ones you'll use constantly:

### Navigation
| Key | Action |
|---|---|
| `h` / `j` / `k` / `l` | Left (parent dir) / down / up / right (enter dir) |
| `Arrow keys` | Same as hjkl |
| `g g` | Go to top |
| `G` | Go to bottom |
| `g h` | Go to home directory |
| `g c` | Go to `~/.config` |
| `g r` | Go to root `/` |
| `.` | Toggle hidden files |

### Selection
| Key | Action |
|---|---|
| `Space` | Toggle selection on current item |
| `v` | Enter visual (select) mode |
| `V` | Enter visual (unset) mode |
| `Ctrl+a` | Select all |
| `Ctrl+r` | Invert selection |

### File operations
| Key | Action |
|---|---|
| `y` | Yank (copy) |
| `x` | Cut |
| `p` | Paste |
| `P` | Paste (overwrite) |
| `d` | Move to trash (needs `trash-cli`) |
| `D` | Permanently delete |
| `a` | Create file/dir (trailing `/` = dir) |
| `r` | Rename |
| `.` | Toggle hidden |
| `c c` | Copy file name |
| `c p` | Copy absolute path |
| `c d` | Copy directory path |

### Opening
| Key | Action |
|---|---|
| `Enter` (or `l` on a file) | Open with default opener |
| `o` | Open with... (choose app) |
| `O` | Open with... (interactive, shows all matches) |

### Tabs & panes
| Key | Action |
|---|---|
| `t` | New tab |
| `1`-`9` | Jump to tab N |
| `[` / `]` | Switch tab left/right |
| `Ctrl+u` / `Ctrl+d` | Preview pane scroll up/down |

### Search & filter
| Key | Action |
|---|---|
| `/` | Filter current dir by name |
| `f` (in filter) | Fuzzy filter as you type |
| `s` | Search (via `fd`) |
| `S` | Search (via `rg`, content search) |

### Misc
| Key | Action |
|---|---|
| `~` | Show help / keymap cheatsheet |
| `q` | Quit |
| `Ctrl+z` | Suspend |
| `:` | Command mode (run a yazi command) |

---

## 4. Customizing Yazi Keybinds

Edit `~/.config/yazi/keymap.toml`. Example — rebind `Enter` to always open in nvim for text files, or add a custom shell command:

```toml
[[manager.prepend_keymap]]
on = ["<C-e>"]
run = "shell 'nvim \"$@\"' --confirm"
desc = "Open selected file(s) in Neovim"
```

That `<C-e>` example opens the hovered/selected file directly in nvim from yazi — extremely useful.

---

## 5. Yazi Openers (`yazi.toml`)

Control what app handles what filetype:

```toml
[opener]
edit = [
    { run = 'nvim "$@"', desc = "Edit in Neovim", block = true },
]
open = [
    { run = 'xdg-open "$@"', desc = "Open" },
]

[open]
rules = [
    { mime = "text/*", use = "edit" },
    { name = "*.md", use = "edit" },
    { mime = "image/*", use = "open" },
    { name = "*", use = ["edit", "open"] },
]
```

`block = true` matters — it tells yazi to suspend and hand the terminal to nvim, then return to yazi when you quit.

---

## 6. Integrating Yazi and Neovim

There are two directions of integration you'll want:

### A. Open Neovim from Yazi
Already covered above — either use `Enter`/default opener rules, or a dedicated keybind like `<C-e>`.

### B. Open Yazi from inside Neovim (as a file picker)
Use the **`yazi.nvim`** plugin (by mikavilpas). This is the modern standard — better than netrw or even Telescope's file picker for browsing.

Using `lazy.nvim`:
```lua
-- lua/plugins/yazi.lua
return {
  "mikavilpas/yazi.nvim",
  event = "VeryLazy",
  dependencies = { "folke/snacks.nvim" }, -- optional, for floating window
  keys = {
    {
      "<leader>-",
      mode = { "n", "v" },
      "<cmd>Yazi<cr>",
      desc = "Open yazi at the current file",
    },
    {
      "<leader>cw",
      "<cmd>Yazi cwd<cr>",
      desc = "Open yazi in nvim's working directory",
    },
    {
      "<c-up>",
      "<cmd>Yazi toggle<cr>",
      desc = "Resume the last yazi session",
    },
  },
  opts = {
    open_for_directories = true, -- lets you `nvim .` and it opens yazi instead of netrw
    keymaps = {
      show_help = "<f1>",
    },
  },
}
```

With `open_for_directories = true`, running `nvim ~/git/taskplus` (or `nvim .`) drops you straight into yazi instead of netrw.

Inside the yazi-in-nvim floating window: `Enter` on a file opens it as a normal nvim buffer, and you're back in your editor.

---

## 7. Neovim: Core Keybinds (Vim motions you'll actually use)

Assuming default/near-default keymaps (`<leader>` is usually `Space` in modern configs):

### Modes
| Key | Action |
|---|---|
| `i` / `a` | Insert before/after cursor |
| `I` / `A` | Insert at line start / end |
| `o` / `O` | New line below/above + insert |
| `v` / `V` / `Ctrl+v` | Visual / Visual line / Visual block |
| `Esc` or `jk` (if remapped) | Back to normal mode |

### Movement
| Key | Action |
|---|---|
| `h j k l` | Left/down/up/right |
| `w` / `b` / `e` | Next word / back word / end of word |
| `0` / `^` / `$` | Line start / first non-blank / line end |
| `gg` / `G` | File start / file end |
| `{` / `}` | Prev/next paragraph |
| `Ctrl+d` / `Ctrl+u` | Half page down/up |
| `%` | Jump to matching bracket |
| `f{char}` / `t{char}` | Jump to / until char on line |

### Editing
| Key | Action |
|---|---|
| `dd` | Delete line |
| `yy` | Yank (copy) line |
| `p` / `P` | Paste after/before |
| `u` / `Ctrl+r` | Undo / redo |
| `.` | Repeat last change |
| `ciw` / `diw` / `yiw` | Change/delete/yank inner word |
| `ci"` / `ci(` etc. | Change inside quotes/brackets |
| `>>` / `<<` | Indent / unindent line |
| `J` | Join line below to current |

### Search & replace
| Key | Action |
|---|---|
| `/pattern` / `?pattern` | Search forward/backward |
| `n` / `N` | Next/prev match |
| `:%s/old/new/g` | Replace all in file |
| `:%s/old/new/gc` | Replace all with confirmation |
| `*` | Search word under cursor |

### Windows/buffers (native)
| Key | Action |
|---|---|
| `:sp` / `:vsp` | Horizontal/vertical split |
| `Ctrl+w` then `h/j/k/l` | Move between splits |
| `Ctrl+w` `q` | Close split |
| `:bn` / `:bp` | Next/prev buffer |
| `:ls` | List buffers |

---

## 8. Neovim: Modern Plugin-Driven Workflow

If you don't already have a distro-config (LazyVim, NvChad, kickstart.nvim), and you want something you fully understand, `kickstart.nvim` is the best starting point — single-file, heavily commented, no magic.

Core plugin stack most modern configs converge on:

| Purpose | Plugin |
|---|---|
| Plugin manager | `lazy.nvim` |
| Fuzzy finder | `telescope.nvim` (or `fzf-lua`) |
| File tree (if not using yazi.nvim) | `nvim-tree.lua` or `neo-tree.nvim` |
| Syntax highlighting | `nvim-treesitter` |
| LSP config | `nvim-lspconfig` + `mason.nvim` (installs LSPs) |
| Autocompletion | `nvim-cmp` or `blink.cmp` |
| Git signs/diff | `gitsigns.nvim` |
| Status line | `lualine.nvim` |
| Comment toggling | `Comment.nvim` (or built-in `gc` in nvim 0.10+) |
| Fast motion | `flash.nvim` |

Typical Telescope keybinds (very common convention):
| Key | Action |
|---|---|
| `<leader>ff` | Find files |
| `<leader>fg` | Live grep (project-wide search, via ripgrep) |
| `<leader>fb` | Find open buffers |
| `<leader>fh` | Search help tags |

LSP keybinds (once `nvim-lspconfig` is set up):
| Key | Action |
|---|---|
| `gd` | Go to definition |
| `gr` | Find references |
| `K` | Hover documentation |
| `<leader>rn` | Rename symbol |
| `<leader>ca` | Code action |
| `[d` / `]d` | Prev/next diagnostic |

---

## 9. A Realistic Daily Workflow

1. Open terminal (kitty) in your project root.
2. `yazi` — browse the tree, preview files, delete/rename/move as needed.
3. Land on the file you want → `Enter`/`<C-e>` → opens in nvim.
4. Inside nvim: `<leader>-` (yazi.nvim) any time you need to jump back into yazi without losing your buffer — e.g., to grab a file into a split, or check what's in a sibling directory.
5. `<leader>ff` / `<leader>fg` (Telescope) for anything you can search by name/content instead of browsing.
6. `:qa` when done, or `Ctrl+z`/`fg` to suspend/resume if you need to shell out.

---

## 10. Quick Reference Cheat-Sheet (print-worthy)

**Yazi essentials:** `hjkl` nav · `Space` select · `y`/`x`/`p` copy/cut/paste · `d` trash · `a` new · `r` rename · `Enter`/`o` open · `/` filter · `s`/`S` search name/content · `.` hidden files · `t` new tab · `~` help

**Neovim essentials:** `i`/`Esc` insert/normal · `dd`/`yy`/`p` cut/copy/paste line · `u`/`Ctrl+r` undo/redo · `ciw`/`di(` change/delete inside · `:%s/a/b/g` replace all · `/` search · `Ctrl+w hjkl` split nav · `gd`/`K`/`<leader>ca` LSP

---

## 11. Where to go deeper

- Yazi docs: https://yazi-rs.github.io/docs/
- Yazi keymap defaults source: check `~/.config/yazi/keymap.toml` after install (it's fully commented)
- `yazi.nvim`: https://github.com/mikavilpas/yazi.nvim
- kickstart.nvim: https://github.com/nvim-lua/kickstart.nvim
- `:help` inside nvim is genuinely the best reference — `:help motion.txt`, `:help lsp`, etc.
