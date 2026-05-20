---
author: Sato Seinosuke
title: MacBook with Homebrew
date: 2025-10-27T13:36:55-04:00
draft: false
layout: docs
description: Set up an ARM-chip MacBook as developer machine using Homebrew as package manager.
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

## Get dot files

```bash
mkdir -p ~/.config
git clone https://github.com/kxue43/dotfiles ~/.config/dotfiles
~/.config/dotfiles/set-up.sh
```

Restart the terminal so that Bash start-up files take effect.

## Set the .gitconfig.local file

If the global Git user name and email must be different from the personal ones,
set them via the following commands.

```bash
git config --file ~/.gitconfig.local user.name USERNAME
git config --file ~/.gitconfig.local user.email EMAIL
```

To configure commit signing differently from the default, add the following contents to `.gitconfig.local`.
Use the options to turn commit signing on/off or to use a different GPG key.

```ini
[commit]
	gpgsign = true
[tag]
	gpgSign = true
[user]
	signingkey = ${GPG_KEY_ID}
```

If all Git configurations are the same with the personal defaults, symlink `.gitconfig.local` to `.gitconfig.personal`.

```bash
ln -s $HOME/.config/dotfiles/.gitconfig.personal $HOME/.gitconfig.local
```

## Set up GPG to sign git commits

Install `gnupg` and `pinentry-mac`. The former is the GPG software while the latter is a GUI for prompting for passphrases.

```bash
brew install gnupg pinentry-mac
echo "pinentry-program $(which pinentry-mac)" >>  ~/.gnupg/gpg-agent.conf
```

Restart `gpg-agent`.

```bash
gpg-connect-agent reloadagent /bye
```

Enter GPG interactive mode by `gpg --card-edit`, and then enter the `fetch` and `quit` command in order.
The outputs would be something like below.

```
gpg/card> fetch
gpg: requesting key from 'https://github.com/kxue43.gpg'
gpg: key C9EED408F4B6D021: "Sato Seinosuke (kxue43.github.io) <kxue43@gmail.com>" not changed
gpg: Total number processed: 1
gpg:              unchanged: 1

gpg/card> quit
```

Then use `gpg --list-secret-keys` to confirm that the keys have been fetched.
The outputs should be something like below.

```
[keyboxd]
---------
sec>  rsa4096 2025-12-24 [SC]
      5EF2BE73370DCE7E808814DBC9EED408F4B6D021
      Card serial no. = 0006 27538718
uid           [ unknown] Sato Seinosuke (kxue43.github.io) <kxue43@gmail.com>
ssb>  rsa4096 2025-12-24 [E]
```

## Install go executables

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

## Install rust executables

```bash
cargo install --locked tree-sitter-cli
```

## Set up neovim

```bash
git clone https://github.com/kxue43/nvim-files ~/.config/nvim && nvim
```

After plugin installation finishes, run `:MasonInstallAll` to install all LSPs
and `:TSInstallAll` to install all `tree-sitter` parsers.

Run `:checkhealth` to see if there are any problems.

## Install java, raven and gradle

First install [Amazon Corretto 21](https://docs.aws.amazon.com/corretto/latest/corretto-21-ug/downloads-list.html) or another LTS version.
This manual installation is to avoid the Oracle OpenJDK that comes from `brew install openjdk`.

Open the file `~/.${env}.bashrc` (`${env}` depends on environment) and update the default Java version to use.

```bash
#export JAVA_HOME=$(/usr/libexec/java_home -v SET_HERE)
```

Source `~/.bashrc` to refresh settings.

```bash
source ~/.bashrc
```

Manually install Maven and Gradle. Set their desired version numbers first.
Packages are installed in `~/.local/lib`. Symlinks to the binaries are created in `~/.local/bin`.

```bash
export MVN_VERSION=3.9.10
export GRADLE_VERSION=8.14.2

curl -o ~/Downloads/apache-maven-${MVN_VERSION}-bin.zip -L https://dlcdn.apache.org/maven/maven-3/${MVN_VERSION}/binaries/apache-maven-${MVN_VERSION}-bin.zip
unzip -d ~/.local/lib ~/Downloads/apache-maven-${MVN_VERSION}-bin.zip
pushd ~/.local/bin && \
ls ~/.local/lib/apache-maven-${MVN_VERSION}/bin | xargs -I % sh -c "ln -s $HOME/.local/lib/apache-maven-${MVN_VERSION}/bin/%" && \
popd

curl -o ~/Downloads/gradle-${GRADLE_VERSION}-bin.zip -L https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-bin.zip
unzip -d ~/.local/lib ~/Downloads/gradle-${GRADLE_VERSION}-bin.zip
pushd ~/.local/bin && \
ln -s $HOME/.local/lib/gradle-${GRADLE_VERSION}/bin/gradle && \
popd

rm ~/Downloads/apache-maven-${MVN_VERSION}-bin.zip 
rm ~/Downloads/gradle-${GRADLE_VERSION}-bin.zip
```

## Optional

### Install pyenv

`pyenv` is a CLI tool written in shell scripts. It installs multiple versions of Python by downloading and
compiling them from C source code. First we install various tools that are needed to compile Python interpreters.

```bash
brew install openssl readline sqlite3 xz zlib tcl-tk
```

Now install `pyenv` via `git clone` into the `~/.pyenv` folder.

```bash
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
pushd ~/.pyenv && src/configure && make -C src && popd
```

### Install pipx and poetry

```bash
brew install pipx
pipx ensurepath
pipx install poetry
pipx inject poetry poetry-plugin-export
```

### Install groovy

Download a Groovy _SDK bundle_ into the `~/Downloads` folder, e.g. use [this link](https://groovy.apache.org/download.html).
Set the `GROOVY_VERSION` environment variable below to the corresponding version.

```bash
export GROOVY_VERSION=4.0.27

unzip -d ~/.local/lib ~/Downloads/apache-groovy-sdk-${GROOVY_VERSION}.zip
pushd ~/.local/bin && \
ls ~/.local/lib/groovy-${GROOVY_VERSION}/bin | grep -v '\.bat$' | grep -v '\.ico$' | xargs -I % sh -c "ln -s $HOME/.local/lib/groovy-${GROOVY_VERSION}/bin/%" && \
popd

rm ~/Downloads/apache-groovy-sdk-${GROOVY_VERSION}.zip
```

Open `~/.${env}.bashrc`, update the installed Groovy version.

```bash
#export GROOVY_HOME=$HOME/.local/lib/groovy-SET_HERE
```
