# Sphinx + MyST (Markdown) + Furo Theme

---

## 1. Installation

```bash
pip install sphinx myst-parser furo --break-system-packages
```

Or in a venv (better for a project you'll keep building):
```fish
python -m venv .venv
source .venv/bin/activate.fish
pip install sphinx myst-parser furo
```

If you go the venv route, `sphinx-build`/`make html` must also run from inside the activated venv.

---

## 2. Project Structure

`sphinx-quickstart` sets this up. A typical split-source layout:

```
docs/
├── Makefile
├── make.bat
├── build/
└── source/
    ├── conf.py          # all configuration
    ├── index.rst        # or index.md — root doc, holds the toctree
    ├── _static/         # custom CSS, images, logos
    └── _templates/
```

Build:
```bash
make html          # from the docs/ root
# or explicitly:
sphinx-build -b html source build
```

Output lands in `build/html/index.html`.

---

## 3. Enabling Markdown with MyST

In `conf.py`:

```python
extensions = [
    "myst_parser",
]
```

That's it. Adding the extension causes all `.md` documents to be parsed as MyST — you don't need a `source_suffix` mapping for the basic case.

Both parsers coexist: `.md` files are parsed as MyST, `.rst` files still go through the reStructuredText parser as normal. You can mix both in the same project and the same `toctree`.

If you *do* want to be explicit (or you're adding a non-standard extension), the dict form still works:
```python
source_suffix = {
    ".rst": "restructuredtext",
    ".md": "markdown",
}
```

---

## 4. MyST Syntax

MyST is CommonMark plus extensions for the things Sphinx needs — roles and directives.

### Directives
Triple-backtick fence with the directive name in braces:

````markdown
```{note}
This renders as a Sphinx note admonition.
```

```{warning}
And this as a warning.
```

```{code-block} python
:linenos:

def hello():
    print("hi")
```
````

### Roles
Inline, with braces before backticks:

```markdown
{doc}`other-page`
{ref}`my-label`
{py:class}`mypackage.MyClass`
```

### Toctree in Markdown
Your `index.md` (if you convert `index.rst`):

````markdown
# My Project

```{toctree}
:maxdepth: 2
:caption: Contents

installation
usage
api
```
````

Note the leading `:option: value` lines — that's how MyST expresses what rST writes as `:maxdepth: 2`.

### Cross-reference targets
```markdown
(my-label)=
## A header

Link to it: {ref}`my-label`
```

---

## 5. MyST Optional Extensions

MyST ships several syntax extensions, off by default. Enable in `conf.py`:

```python
myst_enable_extensions = [
    "colon_fence",       # ::: fences as alternative to ```
    "deflist",           # definition lists
    "linkify",           # bare URLs become links
    "substitution",      # variable substitution
    "tasklist",          # - [ ] checkboxes
    "dollarmath",        # $inline$ and $$block$$ math
    "attrs_inline",      # inline attributes
]
```

`linkify` needs an extra dependency:
```bash
pip install linkify-it-py
```

### colon_fence
Worth enabling — lets you write directives with `:::` instead of backticks, which avoids nesting headaches when a directive contains a code block:

```markdown
:::{note}
This works, and you can put ```python blocks inside without fence conflicts.
:::
```

All MyST config variables are prefixed `myst_` — that's the pattern to look for in the docs.

---

## 6. Furo Theme

```python
html_theme = "furo"
```

That single line is usually enough — Furo's design goal is minimal config, and it gives you light/dark mode following the reader's system preference, responsive layout, and search out of the box.

Furo uses calendar versioning (e.g. `2025.12.19`), MIT licensed.

### Theme options

```python
html_theme_options = {
    "light_logo": "logo-light.png",
    "dark_logo": "logo-dark.png",
    "sidebar_hide_name": False,
    "navigation_with_keys": True,
    "announcement": "New release: 2.0.0 is out!",
    "source_repository": "https://github.com/Crophics/Portal",
    "source_branch": "main",
    "source_directory": "docs/source/",
}
```

| Option | Effect |
|---|---|
| `light_logo` / `dark_logo` | Logo per color scheme (files go in `_static/`) |
| `sidebar_hide_name` | Hide project name in sidebar (useful if logo says it) |
| `announcement` | Site-wide banner at top of every page; accepts HTML |
| `navigation_with_keys` | Arrow-key page navigation |
| `source_repository` / `source_branch` / `source_directory` | Powers the "Edit this page" link |

Only options documented by Furo are supported — it does **not** inherit the options from Sphinx's `basic` theme (`nosidebar`, `sidebarwidth`, etc. won't do anything).

### Colors

Furo is built on CSS variables. Override them via `light_css_variables` / `dark_css_variables`:

```python
html_theme_options = {
    "light_css_variables": {
        "color-brand-primary": "#7C4DFF",
        "color-brand-content": "#7C4DFF",
        "color-admonition-background": "orange",
    },
    "dark_css_variables": {
        "color-brand-primary": "#B388FF",
        "color-brand-content": "#B388FF",
    },
}
```

How light/dark works: Furo is light by default and switches to dark when the browser requests it via `prefers-color-scheme: dark`. Dark mode **inherits** the light-mode variable definitions and only overrides specific values — so anything you set in `light_css_variables` carries into dark unless you override it again.

**Gotcha worth knowing:** typos in the `*_css_variables` dicts are silently ignored — no error, no warning. If a color change "doesn't work," check your spelling first.

The full variable list is in Furo's source under `src/furo/assets/styles/variables`.

### Code block styling
Furo doesn't handle syntax highlighting itself — Sphinx does. Configure it with:
```python
pygments_style = "default"
pygments_dark_style = "monokai"
```

### Custom CSS
```python
html_static_path = ["_static"]
html_css_files = ["custom.css"]
```
Put `custom.css` in `source/_static/`.

### Per-page tweaks
Furo reads file-wide metadata. To hide the right-hand "Contents" sidebar on a specific page, set `hide-toc` at the page level. (It's already hidden automatically on pages with no inner headings.)

---

## 7. A Working `conf.py`

```python
project = "Portal"
copyright = "2026, Julian"
author = "Julian"

extensions = [
    "myst_parser",
]

myst_enable_extensions = [
    "colon_fence",
    "deflist",
    "tasklist",
]

templates_path = ["_templates"]
exclude_patterns = ["_build", "Thumbs.db", ".DS_Store"]

html_theme = "furo"
html_static_path = ["_static"]

html_theme_options = {
    "sidebar_hide_name": False,
    "navigation_with_keys": True,
    "light_css_variables": {
        "color-brand-primary": "#7C4DFF",
        "color-brand-content": "#7C4DFF",
    },
}
```

---

## 8. Common Build Errors

| Error | Cause |
|---|---|
| `NameError: name 'myst_parser' is not defined` | Missing quotes — needs `["myst_parser"]`, not `[myst_parser]` |
| `Could not import extension myst_parser` | Package not installed in the environment Sphinx is running from (very common with venvs, and on Read the Docs if it's not in your requirements file) |
| Theme silently falls back to `alabaster` with no error | `furo` isn't installed — Sphinx doesn't always error loudly on a missing theme |
| Color/variable change does nothing | Typo in `*_css_variables` — silently ignored by design |
| `Unknown directive` | Directive belongs to an extension you haven't added to `extensions` |

Run with `-W` to turn warnings into errors when you want a strict build:
```bash
sphinx-build -W -b html source build
```

---

## 9. Useful Extras

| Extension | Purpose |
|---|---|
| `sphinx.ext.autodoc` | Pull docstrings from Python source |
| `sphinx.ext.napoleon` | Google/NumPy docstring styles |
| `sphinx.ext.intersphinx` | Cross-link to other projects' docs |
| `sphinx_design` | Cards, grids, tabs, dropdowns |
| `sphinx_copybutton` | Copy button on code blocks |
| `sphinx-autobuild` | Live-reloading dev server |

`sphinx-autobuild` is the quality-of-life one:
```bash
pip install sphinx-autobuild
sphinx-autobuild source build/html
```
Serves on localhost and rebuilds on save.

If you add `sphinx_design` alongside MyST, its docs recommend enabling `colon_fence`:
```python
extensions = ["myst_parser", "sphinx_design"]
myst_enable_extensions = ["colon_fence"]
```

---

## 10. Reference

- MyST-Parser docs: https://myst-parser.readthedocs.io/
- MyST config reference (all `myst_` options): https://myst-parser.readthedocs.io/en/latest/configuration.html
- Furo customisation: https://pradyunsg.me/furo/customisation/
- Sphinx docs: https://www.sphinx-doc.org/
