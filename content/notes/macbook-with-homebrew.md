---
author: Sato Seinosuke
title: MacBook with Homebrew
date: 2025-10-27T13:36:55-04:00
draft: false
layout: docs
subtitle: Set up an ARM-chip MacBook as developer machine using Homebrew as package manager.
tags:
- macOS
- dev machine
---

## Introduction

This document covers how to set up an ARM64 Macbook as a developer machine.
It is geared towards Go, Python, JavaScript and Java development.

All commands on this page should be executed from the user's home directory.

For using MacPorts instead of Homebrew as the package manager, refer to [MacBook with MacPorts]({{% ref "/notes/macbook-with-macports" %}}).

## Install xcode command line tools

```bash
xcode-select --install
```

## Install homebrew

Use the macOS default Terminal app.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
eval $(/opt/homebrew/bin/brew shellenv)
brew update
brew upgrade
```

Add the following line to both `~/.zshrc` and `~/.bash_profile`. Restart terminal.

```bash
eval $(/opt/homebrew/bin/brew shellenv)
```

## Use the latest version of bash

```bash
brew install bash bash-completion@2
```

Put `bash` inside `/usr/local/bin` and add it to `/etc/shells`.

```bash
pushd /usr/local/bin
sudo ln -s /opt/homebrew/bin/bash
popd
echo '/usr/local/bin/bash' | sudo tee -a /etc/shells
```

Change the default shell for human admin user to `/usr/local/bin/bash`.

```bash
chsh -s /usr/local/bin/bash
```

Change the default shell for `root` to `/bin/bash`.

```bash
sudo chsh -s /bin/bash
```

Restart computer for the default shell change to take effect.

## Install ghostty

Go to the [download page](https://ghostty.org/download). Download the package installer and use it.

From now on perform all CLI operations in Ghostty.

## Install essential utilities

```bash
brew install rlwrap fzf bat tree jq yq
```

## Install coding tools

When coding, run NeoVim inside Tmux sessions.

```bash
brew install git neovim ripgrep luarocks tmux pre-commit
```

## Install go

```bash
brew install go
```

## Install uv

```bash
brew install uv
```

## Install documentation tools

```bash
brew install hugo sass/sass/sass
```

## Install fnm

`fnm` is a Node versions manager that doesn't cause a noticeable slow down at activation.
The CLI interface is largely similar to that of `nvm`.

```bash
brew install fnm
```

## Install aws cli v2

After installation, pin it to avoid unnecessary frequent version upgrade. It's enough to upgrade for every minor
version bump only.

```bash
brew install awscli
brew pin awscli
```

## Install github cli

For authentication against GitHub, the most convenient option is to use the GitHub CLI. To install, run the
following commands.

```bash
brew install gh
```

Login immediately.

```bash
gh auth login
```

## Install shellcheck

```bash
brew install shellcheck
```

## Install rustup

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

{{% setup-gpg install="brew install gnupg pinentry-mac" %}}

{{% install-go-executables %}}

{{% install-rust-executables %}}

{{% install-claude-code %}}

{{% get-substance %}}
