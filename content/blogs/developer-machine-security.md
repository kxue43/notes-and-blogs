---
author: Ke Xue
title: Developer Machine Security
date: 2025-12-15T19:53:25-05:00
draft: true
layout: docs
description: 
tags: 
- Cyber Security
- macOS
---

The [Shai-Hulud 2.0 attack](https://www.youtube.com/watch?v=lqZo4waMB3c) indicates that the vulnerability of developer machines against
e.g. supply-chain attacks should be taken very seriously.
The following credentials are all easy-to-grab and known targets of Shai-Hulud.

- AWS credentials in the `~/.aws/credentials` file.
- SSH private keys under in the `~/.ssh` folder.
- Configuration files such as `~/.npmrc`, `~/.pip/pip.conf`, etc.
- Environment variables, usually exported in shell start-up scripts such as `.bashrc`.
