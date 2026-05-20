---
author: Sato Seinosuke
title: Neovim tips
date: 2025-11-13T15:41:14-05:00
draft: false
layout: docs
description: Miscellaneous tips for NeoVim. 
tags:
- NeoVim
---

## Global find-and-replace in neovim (no lsp)

Use Telescope → quickfix → `:cdo` when there is no LSP refactoring support (e.g. shell codebases).

1. `<leader>fw` — open live grep and search for the pattern

2. In Telescope results, `<C-q>` to send all matches to the quickfix list
   - Or `<Tab>` to select specific entries, then `<M-q>` to send only those

3. Run the substitution:

   ```vim
   :cdo s/old/new/g | update
   ```

   - `:cdo` runs the command once **per match line** — only the lines ripgrep found,
     so other occurrences in the same file are not touched
   - Use `:cfdo %s/old/new/g | update` instead if you want a whole-file substitution
     on each matched file (e.g. pattern is already precise enough)
   - `| update` saves each file after substitution
   - Add the `c` flag (`s/old/new/gc`) for per-match confirmation
   - Use `\b` for whole-word matching: `s/\bold\b/new/g`

4. `:cclose` to dismiss the quickfix window

### Searching within the current file only

Use `:vimgrep` with `%` instead of Telescope:

```vim
:vimgrep /pattern/ % | copen
```

Then proceed with `:cdo s/old/new/g | update` as above. Note that `:cdo`/`:cfdo` require a non-empty quickfix list — always populate it first.
`copen` opens the quickfix list windown. Otherwise it's not shown.

## Inspect a user-created command

```
:verbose command MasonInstallAll
```

This shows where the command is installed. It reveals that `:MasonInstallAll` is defined in [NvChad/ui] under [nvchad/au.lua].

[NvChad/ui]: https://github.com/NvChad/ui
[nvchad/au.lua]: https://github.com/NvChad/ui/blob/v3.0/lua/nvchad/au.lua

## Color palette

When using NvChad, color palette is provided by [NvChad/base46](https://github.com/NvChad/base46).

Themes define the actual RGB hex value of each "abstract color".
Integration maps "abstract colors" to highlight groups of NeoVim and various plugins.

To override highlight group colors, do it in `chadrc.lua`.

After update, run `:lua require("base46").load_all_highlights()` to re-compute the color files.
Otherwise, colors won't update even after restarting NeoVim.

`:help nvui.base46` shows the `NvChad/base46` help page.

## lazy.nvim: opts function vs. table when overriding a plugin spec

When overriding a plugin that uses `opts = function() return ... end` (e.g. NvChad's `nvim-tree` spec), the function form is the *parent* in the metatable chain.
`lazy.nvim` resolves specs recursively from parent to child: the parent's function is called first to produce a baseline `opts` table, then the child's `opts` are merged on top via deep merge.

The subtlety: if the *child* spec uses `opts = function() return {...} end`, the returned table **replaces** whatever the parent produced (the incoming accumulated opts are passed as an argument but can be ignored).
Using a plain `opts = {}` table in the child avoids this — it always merges on top of the parent's result.

Practical rule: to extend a plugin's defaults, use `opts = {}` in your override spec. Only use the function form if you intentionally want to discard upstream defaults.

## Poke at the actual runtime path

```
:lua vim.tbl_map(print, vim.opt.rtp:get())
```

We get the following output.

```
~/.config/nvim
~/.local/share/nvim/lazy/lazy.nvim
~/.local/share/nvim/lazy/conform.nvim
~/.local/share/nvim/lazy/nvim-lspconfig
~/.local/share/nvim/lazy/indent-blankline.nvim
~/.local/share/nvim/lazy/gitsigns.nvim
~/.local/share/nvim/lazy/nvim-autopairs
~/.local/share/nvim/lazy/LuaSnip
~/.local/share/nvim/lazy/friendly-snippets
~/.local/share/nvim/lazy/blink.cmp
~/.local/share/nvim/lazy/nvim-web-devicons
~/.local/share/nvim/lazy/nvim-tree.lua
~/.local/share/nvim/lazy/ui
~/.local/share/nvim/lazy/which-key.nvim
~/.local/share/nvim/lazy/nvim-treesitter
~/.local/share/nvim/lazy/NvChad
/opt/homebrew/Cellar/neovim/0.11.5/share/nvim/runtime
/opt/homebrew/Cellar/neovim/0.11.5/lib/nvim
~/.local/state/nvim/lazy/readme
~/.local/share/nvim/lazy/indent-blankline.nvim/after
```

`lazy.nvim` seems to have a special Lua [package loader] that goes through all folders
in the above list and looks at the `lua` subfolder within. Every `require`'d Lua modules
is from such a `lua` subfolder. 

The order is important.

`~/.config/nvim` is first, so my configurations always have the highest priority.
Then all the `~/.local/share/nvim/lazy/*` are plugin folders, each corresponding
to the project root of the plugin's GitHub repo.

`/opt/homebrew/Cellar/neovim/0.11.5/share/nvim/runtime` and `/opt/homebrew/Cellar/neovim/0.11.5/lib/nvim` are like the "standard library" of NeoVim.

The rest are some `**/after` folders that serve a special purpose.

[package loader]: https://www.reddit.com/r/neovim/comments/14bglt6/i_dont_understand_lua_modules/
