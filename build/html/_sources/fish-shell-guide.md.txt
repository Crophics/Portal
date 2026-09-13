# Fish Shell: Config, Functions, and Abbreviations

---

## 1. Config File Locations

```
~/.config/fish/
├── config.fish          # main config, sourced on every shell start
├── functions/           # one file per function, autoloaded by filename
├── completions/         # custom tab-completions, autoloaded by filename
└── conf.d/               # snippets auto-sourced on startup, load order = filename
```

Reload config without restarting terminal:
```fish
source ~/.config/fish/config.fish
```

---

## 2. Fish vs Bash — Key Syntax Differences

| Bash | Fish |
|---|---|
| `export VAR=value` | `set -x VAR value` |
| `VAR=value` (local) | `set VAR value` |
| `$VAR` | `$VAR` (same) |
| `if [ "$x" = "y" ]; then ... fi` | `if test "$x" = "y" ... end` |
| `for i in 1 2 3; do ... done` | `for i in 1 2 3 ... end` |
| `func() { ... }` | `function func ... end` |
| `$(cmd)` command substitution | `(cmd)` |
| `&&` / `||` | `and` / `or` (also supports `&&`/`||` in recent fish) |
| `.bashrc` | `config.fish` |
| `alias x='cmd'` | `alias x='cmd'` (works, but see abbreviations below — usually better) |

No `[[ ]]` or `[ ]` test brackets — fish uses `test`:
```fish
if test -f myfile.txt
    echo "exists"
end
```

---

## 3. Variables

```fish
set name "Julian"                # local/shell variable
set -x EDITOR nvim                # exported (environment) variable
set -U fish_greeting ""           # universal variable — persists across sessions/shells
set -e VARNAME                    # erase a variable
```

| Scope flag | Meaning |
|---|---|
| (none) | Local to current block/function |
| `-g` | Global — visible in current shell session |
| `-x` | Exported — visible to child processes |
| `-U` | Universal — persists across all fish sessions, saved to disk |

Lists are native:
```fish
set fruits apple banana cherry
echo $fruits[1]        # apple (fish is 1-indexed)
echo $fruits[-1]       # cherry (negative indexing works)
for f in $fruits
    echo $f
end
```

Path manipulation — fish treats `$PATH` as a list automatically:
```fish
fish_add_path ~/.local/bin
```
This is the correct/modern way to add to PATH persistently (writes a universal variable), rather than manually `set -x PATH ...` in config.fish.

---

## 4. Functions

Functions are the fish equivalent of both bash functions and (often) aliases with logic.

Define inline (session-only):
```fish
function hello
    echo "Hello, $argv[1]!"
end
```

Save permanently:
```fish
funcsave hello
```
This writes it to `~/.config/fish/functions/hello.fish`, autoloaded from then on — you can also just create that file directly.

### Example function with argument handling
```fish
function mkcd --description "mkdir then cd into it"
    mkdir -p $argv[1]
    cd $argv[1]
end
```
`$argv` is the args list; `$argv[1]` first arg, `$argv` alone iterates all.

### Useful function features
```fish
function greet
    if test (count $argv) -eq 0
        echo "Hello, world!"
    else
        echo "Hello, $argv[1]!"
    end
end
```

List all defined functions:
```fish
functions
```

Edit a function in your $EDITOR:
```fish
funced hello
```

Remove a function:
```fish
functions -e hello
rm ~/.config/fish/functions/hello.fish   # if saved to disk
```

---

## 5. Abbreviations (fish's power feature — prefer these over aliases)

Abbreviations expand in-place as you type, so the full command shows in your history and you can edit it before running. Aliases just run a fixed replacement silently.

```fish
abbr -a gs git status
abbr -a gc git commit
abbr -a gco git checkout
abbr -a ll  ls -la
abbr -a ..  cd ..
```

Type `gs` + space/enter → it expands to `git status` in the command line before executing.

Persist automatically — `abbr -a` writes to universal variables, no need to funcsave. To make them explicit in your config anyway (for portability/version control), add them to `config.fish`:
```fish
abbr -a gs git status
abbr -a gc git commit
abbr -a gp git push
abbr -a gl git pull
```

List all abbreviations:
```fish
abbr --list
abbr --show
```

Remove one:
```fish
abbr -e gs
```

### Abbreviations with a function (dynamic expansion)
```fish
abbr -a gcm --set-cursor='%' 'git commit -m "%"'
```
`%` marks where the cursor lands after expansion.

---

## 6. Aliases (simpler, but abbreviations are usually better)

```fish
alias ll 'ls -la'
```
Aliases in fish are actually implemented as functions under the hood. Not saved automatically — add to `config.fish` or `funcsave` if you want persistence.

---

## 7. Prompt Customization

Fish's prompt is just a function: `fish_prompt`.

```fish
funced fish_prompt
```

If you're using a prompt framework (Starship, Tide, etc. — common on Arch/Hyprland setups), it hooks into this automatically. Check what you have:
```fish
functions fish_prompt
```

Right-side prompt (optional, shows e.g. time/git status on the right):
```fish
funced fish_right_prompt
```

---

## 8. Autocompletion

Fish auto-generates completions for most CLI tools that ship man pages, and many tools (like `yazi`, `fzf`) ship their own fish completion scripts.

Custom completion example — save to `~/.config/fish/completions/mytool.fish`:
```fish
complete -c mytool -s v -l verbose -d "Enable verbose output"
complete -c mytool -s h -l help -d "Show help"
```

Regenerate fish's completion cache from man pages:
```fish
fish_update_completions
```

---

## 9. History

| Command | What it does |
|---|---|
| `history` | Show command history |
| `history search <term>` | Search history |
| `history delete --exact "cmd"` | Delete a specific entry |
| `Ctrl+R` | Interactive history search (if bound; fzf integration often remaps this) |
| `↑` / `↓` | Step through history, filtered by what you've typed so far |

---

## 10. Useful Built-ins & Idioms

```fish
type -a cmdname          # what a command resolves to (function/alias/binary), all matches
which cmdname             # path to binary (like bash's which)
command cmdname           # bypass functions/abbrs, run the real binary
set -q VARNAME; and echo "set"   # check if var is set
string split ',' "a,b,c"  # fish's native string manipulation (replaces cut/awk for simple cases)
string match -r 'pattern' $str
math "2 + 2"               # built-in calculator, no need for bc/expr
```

`string` and `math` are fish builtins worth knowing — they replace a lot of what you'd reach for `sed`/`awk`/`bc` for in bash one-liners.

---

## 11. conf.d/ for Modular Config

Instead of piling everything into `config.fish`, drop snippets into `conf.d/`:
```
~/.config/fish/conf.d/
├── 00-env.fish        # env vars, PATH
├── 10-abbr.fish        # abbreviations
└── 20-prompt.fish      # prompt setup
```
Loaded in filename order at shell start — useful for keeping things organized instead of one giant `config.fish`, especially as your dotfiles grow (relevant since you're already tracking configs in git repos).

---

## 12. Reference

- Official docs: https://fishshell.com/docs/current/
- `help` inside fish opens the docs in browser: just run `help` or `help <topic>`
- Interactive tutorial: `fish_config` opens a browser-based config UI (themes, functions, abbreviations, colors) — worth trying once
