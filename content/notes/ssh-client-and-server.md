---
author: Ke Xue
title: SSH Client and Server
date: 2024-11-05T21:31:53-05:00
draft: false
layout: docs
description: Set up SSH client and server.
tags: 
- SSH
- Linux
- macOS
---

This page describes how to set up a Linux machine as an SSH server and remote in from macOS.

## SSH Server on Linux

First install OpenSSH server on the specific Linux distro if it's not already present.

Then check if the `sshd` service is enabled and active.

```bash
sudo systemctl status sshd
```

If not, enable and activate it by the following.

```bash
sudo systemctl enable sshd
sudo systemctl start sshd
```

## SSH Client on macOS

The [Shai-Hulud 2.0 attack](https://www.youtube.com/watch?v=lqZo4waMB3c) indicates that the vulnerability of developer machines against
e.g. supply-chain attacks should be taken very seriously.
In particular, SSH private key files stored in the `~/.ssh` folder should no longer be considered secure enough.
Therefore, we should use SSH key pairs with FIDO support only. Below are how to set up such keys on macOS.

Obviously, we should first obtain a good secure key with FIDO support, e.g. YubiKey 5c.

Then, install the following dependencies, as the builtin `ssh` binary on macOS doesn't work with FIDO secure keys out of the box.

```bash
brew install libfido2 openssh
```

Make sure both `ssh` and `ssh-keygen` point to binaries installed by Homebrew via the `openssh` package.
On MacBook with ARM chips, they are if they lie in the `/opt/homebrew/bin` folder.

```bash
which ssh
which ssh-keygen
```

First generate an SSH key pair with FIDO support. When prompted, touch the secure key.
Passphrase is optional. Substitute comment with the appropriate contents.

```bash
ssh-keygen -t ed25519-sk -C "REPLACE_HERE"
```

Add the *public key* to the SSH server, this time using password authentication of the user.

```bash
ssh-copy-id -i $PUBLIC_KEY_PATH ${USER}@${HOST}
```

Put the following section in `~/.ssh/config`. Replace `<ALIAS>` with an easy-to-remember alias.
Replace other `<***>` with the values of the same-named environment variables above.
Note that `IdentityAgent none` is critical as it forces the Homebrew SSH to handle the authentication itself rather than delegating it to the (incompatible) Apple agent.

```
Host <ALIAS>
    User <USER>
    HostName <HOST>
    IdentityAgent none
    IdentityFile <PUBLIC_KEY_PATH>
```

Remote into the SSH server with the following command. Touch the secure key when prompted.

```bash
ssh ${USER}@${ALIAS}
```

## Optionally Disable SSH Password Authentication

After setting up SSH Key authentication for one user, we can optionally disable SSH password authentication
on the server altogether.

Open the file `/etc/ssh/sshd_config`.
Find the line with:

```
#PasswordAuthentication yes
```

Change it to:

```
PasswordAuthentication no
```

Then restart the `sshd` service.

```bash
sudo systemctl restart sshd
```
