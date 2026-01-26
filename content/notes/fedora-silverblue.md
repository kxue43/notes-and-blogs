---
author: Ke Xue
title: Fedora Silverblue Desktop
date: 2025-10-27T13:41:19-04:00
draft: false
layout: docs
description: Set up a Fedora Silverblue Linux desktop as developer machine.
tags:
- Linux
- dev machine
---

This document covers how to set up an x64 Fedora Silverblue as a developer machine.
It is geared towards Go, Python and JavaScript development.

All commands on this page should be executed from the user's home directory.

## Install Nvidia drivers

Follow the [official doc](https://docs.fedoraproject.org/en-US/fedora-silverblue/troubleshooting/#_using_nvidia_drivers).

## Configure terminal

Download [JetBrainsMono Nerd Font](https://github.com/ryanoasis/nerd-fonts/releases/download/v3.4.0/JetBrainsMono.zip).

```bash
mkdir -p ~/.local/share/fonts
unzip ~/Downloads/JetBrainsMono.zip -d ~/.local/share/fonts
rm ~/.local/share/fonts/*.txt ~/.local/share/fonts/*.md
fc-cache -f -v
```

Restart the Terminal app.

Go to Preference. Choose the Gruvbox color palette. Set font to JetBrainsMono Nerd Font Regular, size 16.

## Create a Toolbox container

```bash
toolbox create dev
toolbox enter dev
```

All subsequent commands run in this `dev` container.

## Install essential utilities

```bash
sudo dnf group install development-tools
sudo dnf install rlwrap jq fzf bat
```

## Install coding tools

```bash
sudo dnf install vim-enhanced neovim ripgrep luarocks tmux pre-commit
```

## Install `go`

```bash
sudo dnf install golang
```

## Install `uv`

```bash
curl -LsSf https://astral.sh/uv/install.sh | UV_NO_MODIFY_PATH=1 sh
```

## Install `rustup`

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

## Install `fnm`

`fnm` is a Node versions manager that doesn't cause a noticeable slow down at activation.
The CLI interface is largely similar to that of `nvm`.

```bash
export PATH="$HOME/.cargo/bin:$PATH"
cargo install fnm
```

## Install AWS CLI v2

```bash
sudo dnf install awscli2
```

## Install GitHub CLI

For authentication against GitHub, the most convenient option is to use the GitHub CLI.

```bash
sudo dnf install gh
```

Login immediately.

```bash
gh auth login
```

## Install `shellcheck`

```bash
sudo dnf install shellcheck
```

## Get dot files

```bash
mkdir -p ~/.config
git clone https://github.com/kxue43/dot-files ~/.config/dot-files
~/.config/dot-files/set-up.sh --with=untracked
```

Restart the Terminal app so that Bash start-up files take effect.

## Install Go executables

```bash
go install github.com/kxue43/cli-toolkit/cmd/toolkit@latest
go install github.com/kxue43/cli-toolkit/cmd/toolkit-assume-role@latest
go install github.com/kxue43/cli-toolkit/cmd/toolkit-serve-static@latest
go install github.com/kxue43/cli-toolkit/cmd/toolkit-show-md@latest
go install mvdan.cc/sh/v3/cmd/shfmt@latest
go install golang.org/x/tools/cmd/godoc@latest
go install golang.org/x/pkgsite
go install github.com/air-verse/air@latest
```

## Install Rust executables

```bash
cargo install --locked tree-sitter-cli
```

## Set up NeoVim

```bash
git clone https://github.com/kxue43/nvim-files ~/.config/nvim && nvim
```

After plugin installation finishes, run `:MasonInstallAll` to install all LSPs.

Run `:checkhealth` to see if there are any problems.

## Optional

### Install `pyenv`

`pyenv` is a CLI tool written in shell scripts. It installs multiple versions of Python by downloading and
compiling them from C source code. First we install various tools that are needed to compile Python interpreters.

```bash
sudo dnf update vte-profile  # https://github.com/containers/toolbox/issues/390
sudo dnf install zlib-devel bzip2 bzip2-devel readline-devel sqlite \
sqlite-devel openssl-devel xz xz-devel libffi-devel findutils tk-devel libzstd-devel
```

Now install `pyenv` via `git clone` into the `~/.pyenv` folder.

```bash
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
pushd ~/.pyenv && src/configure && make -C src && popd
```

### Install `pipx` and `poetry`

```bash
sudo dnf install pipx
pipx ensurepath
pipx install poetry
pipx inject poetry poetry-plugin-export
```
