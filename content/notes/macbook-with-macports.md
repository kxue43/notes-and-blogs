---
author: Sato Seinosuke
title: MacBook with MacPorts
subtitle: Set up an Intel-chip MacBook as developer machine using MacPorts as package manager. 
date: 2025-10-27T13:40:57-04:00
draft: false
layout: docs
tags:
- macOS
- dev machine
---

## Introduction

This document covers how to set up an Intel-chip (x64) Macbook as a developer machine, using MacPorts as the package manager.

This is intended for old Macbooks whose OS has dropped out of Homebrew Tier 1 support.
For example, when macOS Tahoe was released, Ventura dropped to Homebrew Tier 3 support.
A 2017 Macbook Pro cannot upgrade beyond Ventura, so it has to use MacPorts.
Since Tahoe is the last macOS that supports x64, eventually all Intel-chip Macbooks have to use MacPorts instead of Homebrew.

All commands on this page should be executed from the user's home directory.

See [MacBook with Homebrew]({{% ref "/notes/macbook-with-homebrew" %}}) for setting up an ARM64 MacBook using Homebrew as the package manager.

## Install xcode command line tools

```bash
xcode-select --install
```

## Install macports

Use the GUI installer downloaded from the [MacPorts website](https://www.macports.org/install.php).
Choose to use the "default prefix" during installation.

Use the macOS default Terminal app. Add the following line to both `~/.zshrc` and `~/.bash_profile`. Restart terminal.

```bash
export PATH="/opt/local/bin:/opt/local/sbin:$PATH"
```

Perform initial `selfupdate`.

```bash
sudo port selfupdate
```

## Use the latest version of bash

```bash
sudo port install bash bash-completion
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
sudo port install curl jq yq rlwrap fzf bat tree
```

## Install coding tools

When coding, run NeoVim inside Tmux sessions.

```bash
sudo port install git neovim ripgrep lua-luarocks tmux pre-commit
```

## Install go

```bash
sudo port install go
```

## Install uv

```bash
sudo port install uv
```

## Install fnm

`fnm` is a Node versions manager that doesn't cause a noticeable slow down at activation.
The CLI interface is largely similar to that of `nvm`.

```bash
sudo port install fnm
```

## Install aws cli v2

```bash
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
rm AWSCLIV2.pkg
```

## Install github cli

For authentication against GitHub, the most convenient option is to use the GitHub CLI. To install, run the
following commands.

```bash
sudo port install gh
```

Login immediately.

```bash
gh auth login
```

## Install shellcheck

```bash
sudo port install shellcheck
```

{{% get-substance %}}

{{% setup-gpg install="sudo port install gnupg2 pinentry-mac" %}}

{{% install-go-executables %}}

{{% install-rust-executables %}}
