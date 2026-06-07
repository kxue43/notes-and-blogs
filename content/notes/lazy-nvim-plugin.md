---
author: Sato Seinosuke
title: Switching lazynvim plugin to a fork
date: 2026-06-07T18:56:05-04:00
draft: false
layout: docs
description: 
tags:
- NeoVim
---

## Background

The original `nvim-treesitter/nvim-treesitter` was archived April 3 2026.
The community fork `neovim-treesitter/nvim-treesitter` is a true Git continuation
(shares full commit history up to the archival point) and is the recommended replacement.

## What changed in the fork

- **New required dependency:** `neovim-treesitter/treesitter-parser-registry`.
  It provides `require('treesitter-registry')`, used internally for parser discovery — must be listed in `dependencies`.

- **No `ensure_installed` in `setup()`:** the fork's `setup()` only accepts `install_dir`;
  `ensure_installed` is silently ignored by the plugin itself

- **NvChad compatibility:** NvChad's `TSInstallAll` command reads `opts.ensure_installed` directly from the lazy.nvim spec
(not from the plugin's own setup), then calls `require("nvim-treesitter").install(list)`.
That API still exists in the fork, so NvChad integration works unchanged.

## How to switch in lazy.nvim

Change the plugin name string and add the registry dependency **everywhere** the plugin is referenced:

```lua
-- lua/plugins/init.lua
{
  "neovim-treesitter/nvim-treesitter",           -- was nvim-treesitter/nvim-treesitter
  dependencies = { "neovim-treesitter/treesitter-parser-registry" },
...
}

-- lua/plugins/telescope.lua (or any file with nvim-treesitter as a dep)
{
"nvim-telescope/telescope.nvim",
dependencies = { "neovim-treesitter/nvim-treesitter" },  -- update here too
...
}
```

After saving, close Neovim, delete the old clone, and reopen:

```bash
rm -rf ~/.local/share/nvim/lazy/nvim-treesitter
# reopen nvim — lazy.nvim auto-installs from the fork
```

### The lazy.nvim "last fragment wins" gotcha

lazy.nvim resolves a plugin's source URL by iterating all spec fragments for that plugin in processing order
and taking the **last non-nil url** (`plugin.url = fragment.url or plugin.url`).
Plugin files within an import are processed **alphabetically**.

Consequence: if `telescope.lua` (t) contains `dependencies = { "nvim-treesitter/nvim-treesitter" }`
and `init.lua` (i) contains `"neovim-treesitter/nvim-treesitter"`,
the telescope dependency is processed last and silently wins — reverting the URL back to the original repo.

**Rule:** when switching a plugin's source URL via the name string,
update **every** occurrence of the old name across all spec files,
including `dependencies` lists in unrelated plugin specs.
