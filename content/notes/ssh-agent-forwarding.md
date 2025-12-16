---
author: Ke Xue
title: SSH Agent Forwarding
date: 2025-12-15T20:39:48-05:00
draft: true
layout: docs
description: 
tags: 
---

## Use SSH Agent Forwarding to Allow `git clone` on Server

When cloning a private GitHub repository over SSH *on the server*, the server needs access to the SSH private key.
It is not secure to place the SSH private key file directly on the server. The better option is
to enable SSH agent forwarding on the client and forward the private key from client to server when they are connected.

For example, to set up agent forwarding for `github.com` put the following contents in `~/.ssh/config`:

```
Host <server_ip_v4>
    ForwardAgent yes
Host github.com
    User git
    AddKeysToAgent yes
    IdentityFile <path_to_private_key_file>
```

The `ForwardAgent yes` option shares the client's private keys with the server whenever there is a connection,
so should be used with caution. In particular, it is best to use specific IP addresses or host names
in the first rule, instead of using wildcard.

## References

- [OpenSSH configuration](https://help.ubuntu.com/community/SSH/OpenSSH/Configuring)
- [SSH Key authentication](https://code.visualstudio.com/docs/remote/troubleshooting#_quick-start-using-ssh-keys)
- [Troubleshooting SSH agent forwarding](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/using-ssh-agent-forwarding#troubleshooting-ssh-agent-forwarding)
