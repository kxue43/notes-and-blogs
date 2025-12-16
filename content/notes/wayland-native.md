---
author: Ke Xue
title: Wayland Native
date: 2024-11-05T19:39:48-05:00
draft: true
layout: docs
description: Running Linux apps with native Wayland.
tags: 
- Linux
---

Some applications do not start in Wayland native but use Xwayland by default, e.g. VS Code and the Flathub
version of Thunderbird. Xwayland is an Xorg emulation layer of Wayland. It provides applications access
to Xorg libraries, allowing programs that only run on Xorg to run on Wayland.

VS Code is an Electron app --- the visual presentation is handled by Electron, which in turn depends on Chromium.
By default, when running Debian 12 or Fedora 39 with Wayland, VS Code uses Xwayland instead of Wayland native.
This is probably caused by the underlying Chromium component.

Running VS Code with Xwayland in a Wayland session on a machine with NVIDIA drivers is not the most smooth experience,
but running it in Wayland native makes things much better.

This page describes how to detect if a GUI app is running Xwayland, and how to run some apps in Wayland native.

## Detect Xwayland Applications Visually

Run the following command to start `xeyes` from a terminal. A pair of eyes will appear in a small window.

```bash
xeyes
```

When moving the mouse inside another app window, if the eyes are following the cursor, it means the app is running
with Xwayland. Otherwise it's running Wayland native.

To end `xeyes`, simply hit `Ctrl` + `C` in the terminal window.

On Debian 12, `xeyes` comes with the `x11-apps` package.

Reference:

- <https://wiki.archlinux.org/title/wayland#Detect_Xwayland_applications_visually>

## VS Code

The following command from a terminal starts VS Code in Wayland native.

```bash
code --enable-features=UseOzonePlatform --ozone-platform=wayland
```

The command line flags are not parsed by `code` but forwarded to Electron/Chromium, as the stdout messages suggest.

As a side effect, this causes some glitches for the VS Code icon in the dock --- it's a different icon from the app
itself and cannot be pinned to dock.

In ArchLinux, there is a way to use the flags by putting them in a config file. On Debian/Fedora that approach
doesn't seem to exist, but we can put the command above as an alias for `code` in `.zshrc`.

## Flathub Thunderbird

Thunderbird is not built upon Electron so the method for VS Code does not apply.

To configure the Flathub version of Thunderbird, run the following command.

```bash
flatpak override --user --env=MOZ_ENABLE_WAYLAND=1 --socket=wayland org.mozilla.Thunderbird
```
