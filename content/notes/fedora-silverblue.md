---
author: Sato Seinosuke
title: Fedora Silverblue Desktop
subtitl: Set up a Fedora Silverblue Linux desktop as developer machine.
date: 2025-10-27T13:41:19-04:00
draft: false
layout: docs
tags:
- Linux
- dev machine
---

This document covers how to set up an x64 Fedora Silverblue as a developer machine.
It is geared towards Go, Python and JavaScript development.

All commands on this page should be executed from the user's home directory.

## Install nvidia drivers

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

## Create a toolbox container

```bash
toolbox create dev
toolbox enter dev
```

All subsequent commands run in this `dev` container.

## Install essential utilities

```bash
sudo dnf group install development-tools
sudo dnf install rlwrap jq yq fzf bat tree
```

## Install coding tools

```bash
sudo dnf install vim-enhanced neovim ripgrep luarocks tmux pre-commit
```

## Install go

```bash
sudo dnf install golang
```

## Install uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | UV_NO_MODIFY_PATH=1 sh
```

## Install rustup

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

## Install fnm

`fnm` is a Node versions manager that doesn't cause a noticeable slow down at activation.
The CLI interface is largely similar to that of `nvm`.

```bash
export PATH="$HOME/.cargo/bin:$PATH"
cargo install fnm
```

## Install aws cli v2

```bash
sudo dnf install awscli2
```

## Install github cli

For authentication against GitHub, the most convenient option is to use the GitHub CLI.

```bash
sudo dnf install gh
```

Login immediately.

```bash
gh auth login
```

## Install shellcheck

```bash
sudo dnf install shellcheck
```

{{% get-substance %}}

{{% install-go-executables %}}

{{% install-rust-executables %}}

{{% setup-neovim %}}
