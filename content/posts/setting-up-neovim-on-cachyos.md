+++
title = 'Setting Up Neovim as a Development Environment on CachyOS'
date = 2026-04-01T12:00:00-05:00
draft = false
description = "How I configured Neovim with LSP, Treesitter, and a few essential plugins to replace a full IDE on my CachyOS machine."
tags = ["linux", "neovim", "cachyos", "tooling", "editor"]
+++

After years of using VS Code I decided to try going all-in on Neovim. This is what my setup looks like after a few weeks of tweaking.

## Why Neovim

VS Code works fine but it feels heavy on a terminal-first workflow. I spend most of my time in a terminal anyway, so having the editor live there too reduces context switching. Neovim with a good config is also noticeably faster to open and navigate.

## Installing

On CachyOS (Arch-based) the latest stable Neovim is in the main repos:

```bash
sudo pacman -S neovim
```

If you want nightly builds:

```bash
yay -S neovim-nightly-bin
```

## Config structure

I use `lazy.nvim` as the plugin manager. My config lives at `~/.config/nvim/` with this layout:

```
nvim/
├── init.lua
└── lua/
    ├── plugins/
    │   ├── lsp.lua
    │   ├── treesitter.lua
    │   └── ui.lua
    └── settings.lua
```

## Essential plugins

| Plugin | Purpose |
|--------|---------|
| `nvim-lspconfig` | Language server protocol support |
| `nvim-treesitter` | Syntax highlighting and text objects |
| `telescope.nvim` | Fuzzy finder for files, grep, buffers |
| `nvim-cmp` | Autocompletion |
| `gitsigns.nvim` | Git diff indicators in the gutter |

## LSP setup

Install language servers through Mason:

```bash
:MasonInstall lua-language-server pyright typescript-language-server
```

Then wire them up in `lsp.lua`:

```lua
local lspconfig = require('lspconfig')

lspconfig.pyright.setup({})
lspconfig.ts_ls.setup({})
lspconfig.lua_ls.setup({
  settings = {
    Lua = { diagnostics = { globals = { 'vim' } } }
  }
})
```

## What I'm still figuring out

- Debugger integration with `nvim-dap` — works but the config is involved
- A good workflow for markdown/notes that doesn't feel slower than VS Code

Overall the switch has been worth it. The startup time and keyboard-driven navigation make it feel like the editor gets out of the way.
