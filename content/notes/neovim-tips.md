---
author: Ke Xue
title: NeoVim Tips
date: 2025-11-13T15:41:14-05:00
draft: false
layout: docs
description: Miscellaneous tips for NeoVim. 
tags:
- NeoVim
---

## Color palette

When using NvChad, color palette is provided by [NvChad/base46](https://github.com/NvChad/base46).

Themes define the actual RGB hex value of each "abstract color".
Integration maps "abstract colors" to highlight groups of NeoVim and various plugins.

To override highlight group colors, do it in `chadrc.lua`.

After update, run `:lua require("base46").load_all_highlights()` to re-compute the color files.
Otherwise, colors won't update even after restarting NeoVim.

`:help nvui.base46` shows the `NvChad/base46` help page.

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

## Inspect a user-created command

```
:verbose command MasonInstallAll
```

This shows where the command is installed. It reveals that `:MasonInstallAll` is defined in [NvChad/ui] under [nvchad/au.lua].

[NvChad/ui]: https://github.com/NvChad/ui
[nvchad/au.lua]: https://github.com/NvChad/ui/blob/v3.0/lua/nvchad/au.lua
