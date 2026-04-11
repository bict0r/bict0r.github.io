+++
title = 'Building a GitHub Dark Color Palette for the Risotto Hugo Theme'
date = 2026-04-11T00:00:00-05:00
draft = false
description = "How I mapped the GitHub Dark terminal colorscheme to the base16 framework to create a custom palette for the Risotto Hugo theme — with the file available to download."
tags = ["hugo", "risotto", "css", "theming", "base16"]
+++

The [Risotto](https://github.com/joeroe/risotto) Hugo theme uses the [base16](https://github.com/chriskempson/base16) color framework — a system that defines 16 named CSS variables, each with a specific semantic role. Any terminal colorscheme that follows the base16 spec can be dropped straight into Risotto with no extra work.

I wanted my site to match the GitHub Dark theme I use in my terminal, so I built a custom palette from scratch. Here's how it works and how you can use it.

---

## How Risotto loads color palettes

Risotto looks for palette files in two places, in order:

1. `static/css/palettes/<name>.css` in your project root
2. `static/css/palettes/<name>.css` inside the theme itself

Set the palette name in `hugo.toml`:

```toml
[params.theme]
  palette = "base16-dark-custom"
```

Placing your file in your project's `static/` directory means it overrides the theme's built-in palettes and won't get wiped out when you update the theme submodule.

---

## What the base16 variables mean

Each variable has a defined role. The ones that matter most for a readable site:

| Variable | Role |
|----------|------|
| `--base00` | Page background |
| `--base01` | Lighter background — panels, sidebars |
| `--base02` | Code block background |
| `--base03` | Comments, muted text |
| `--base04` | Secondary foreground |
| `--base05` | Default body text |
| `--base06` | Light foreground |
| `--base07` | Pure white |
| `--base08` | Red — errors, deletions |
| `--base09` | Orange — numbers, constants |
| `--base0A` | Yellow — classes |
| `--base0B` | Green — strings |
| `--base0C` | Cyan — regex, support |
| `--base0D` | Blue — functions, links |
| `--base0E` | Purple — keywords |
| `--base0F` | Bright red — deprecated |

Risotto maps these to semantic aliases in `colours.css`:

```css
--bg:       var(--base00);   /* page background */
--inner-bg: var(--base02);   /* code block background */
--fg:       var(--base05);   /* body text */
--muted:    var(--base03);   /* comments, nav prompts */
--link:     var(--base0D);   /* links */
```

`--base02` is especially important — it's the code block background. If you accidentally set it to a light color, your code text becomes unreadable.

---

## Mapping GitHub Dark to base16

I pulled the colors from the [GitHub Dark Alacritty config](https://github.com/nicowillis/alacritty-github-theme). Terminal colorschemes define colors in two groups: normal (0–7) and bright (8–15). The base16 mapping is:

| base16 | Terminal slot | GitHub Dark value | Notes |
|--------|--------------|-------------------|-------|
| `--base00` | background | `#010409` | Deepest background |
| `--base01` | — | `#0d1117` | GitHub's main dark canvas |
| `--base02` | selection bg | `#161b22` | GitHub's inset code canvas |
| `--base03` | bright black | `#6e7681` | Comments, muted UI |
| `--base04` | normal white | `#b1bac4` | Secondary text |
| `--base05` | foreground | `#e6edf3` | Body text |
| `--base06` | — | `#e6edf3` | Same as base05 |
| `--base07` | bright white | `#ffffff` | Pure white |
| `--base08` | normal red | `#ff7b72` | Errors |
| `--base09` | normal yellow | `#d29922` | Constants, numbers |
| `--base0A` | bright yellow | `#e3b341` | Classes |
| `--base0B` | normal green | `#3fb950` | Strings |
| `--base0C` | normal cyan | `#39c5cf` | Regex, support |
| `--base0D` | normal blue | `#58a6ff` | Functions, links |
| `--base0E` | normal magenta | `#bc8cff` | Keywords |
| `--base0F` | bright red | `#ffa198` | Deprecated |

---

## The palette file

```css
/* GitHub Dark
 * Source: terminalcolors.com/themes/github/dark/
 * Maps Alacritty terminal colors to base16 roles
 */

:root {
    --base00: #010409; /* background */
    --base01: #0d1117; /* lighter background — panels */
    --base02: #161b22; /* code block background */
    --base03: #6e7681; /* comments — bright black */
    --base04: #b1bac4; /* secondary text — normal white */
    --base05: #e6edf3; /* default foreground */
    --base06: #e6edf3; /* light foreground */
    --base07: #ffffff; /* white */
    --base08: #ff7b72; /* red — errors, deletions */
    --base09: #d29922; /* orange — numbers, constants */
    --base0A: #e3b341; /* yellow — classes */
    --base0B: #3fb950; /* green — strings */
    --base0C: #39c5cf; /* cyan — regex, support */
    --base0D: #58a6ff; /* blue — functions, links */
    --base0E: #bc8cff; /* purple — keywords */
    --base0F: #ffa198; /* bright red — deprecated */
}
```

---

## Using it in your own Risotto site

1. Save the CSS above to `static/css/palettes/github-dark.css` in your Hugo project root
2. In `hugo.toml`, set:

```toml
[params.theme]
  palette = "github-dark"
```

3. Run `hugo server` to preview.

You can also [download the file directly](/css/palettes/base16-dark-custom.css) — that's the exact file this site uses.

---

## Adapting it to other colorschemes

The same process works for any terminal theme. If your terminal emulator (Alacritty, Kitty, WezTerm) has a config file with hex colors, map them like this:

- `colors.primary.background` → `--base00`
- `colors.primary.foreground` → `--base05`
- `colors.normal.*` → `--base08` through `--base0C` (syntax colors)
- `colors.bright.black` → `--base03` (comments/muted)
- `colors.bright.white` → `--base07`

The tricky one is `--base02` — don't use your terminal's selection background color for this. Use the darkest sensible inset background instead, something a few shades lighter than `--base00`.
