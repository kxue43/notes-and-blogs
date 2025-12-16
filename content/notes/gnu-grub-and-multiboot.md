---
author: Ke Xue
title: GNU GRUB and Multiboot
date: 2024-11-04T21:45:41-05:00
draft: false
layout: docs
description: Set up Linux multiboot using GNU GRUB.
tags: 
- Linux
---

## Bootloader

The [Bootloader Wiki page](https://en.wikipedia.org/wiki/Bootloader) contains good information
about bootloaders in general.

According to the Wiki page, BIOS and UEFI are first-stage bootloaders.
They start the second-stage bootloader such as GNU GRUB. Second-stage
bootloaders "are not themselves operating systems, but are able to load
an operating system properly and transfer execution to it; the operating
system subsequently initializes itself and may load extra device
drivers. The second-stage boot loader does not need drivers for its own
operation, but may instead use generic storage access methods provided
by system firmware such as the BIOS or Open Firmware, though typically
with restricted hardware functionality and lower performance".

## GNU GRUB

According to the [GNU GRUB Wiki page](https://en.wikipedia.org/wiki/GNU_GRUB),
"GNU GRUB is a boot loader package from the GNU Project. GRUB is the reference
implementation of the Free Software Foundation's Multiboot
Specification, which provides a user the choice to boot one of multiple
operating systems installed on a computer or select a specific kernel
configuration available on a particular operating system's
partitions".

On a PC with dual-boot Debian 12 and Fedora 39, the `/boot/efi` folder is shared by the two systems
because it mounts the `sda1` partition. The rest of the `/boot` folder are on each system's partition,
hence different across systems. For example, `sudo ls -l /boot` shows the following on Debian 12:

```
-rw-r--r-- 1 root root   259453 Feb  1 03:05 config-6.1.0-18-amd64
drwx------ 4 root root     4096 Dec 31  1969 efi
drwxr-xr-x 5 root root     4096 Feb 25 22:29 grub
-rw-r--r-- 1 root root 60072437 Feb 26 08:43 initrd.img-6.1.0-18-amd64
-rw-r--r-- 1 root root       83 Feb  1 03:05 System.map-6.1.0-18-amd64
-rw-r--r-- 1 root root  8152768 Feb  1 03:05 vmlinuz-6.1.0-18-amd64
```

while it shows the following on Fedora 39:

```
-rw-r--r--. 1 root root    263616 Oct  5 20:00 config-6.5.6-300.fc39.x86_64
-rw-r--r--. 1 root root    269338 Feb 16 19:00 config-6.7.5-200.fc39.x86_64
-rw-r--r--. 1 root root    269378 Feb 22 19:00 config-6.7.6-200.fc39.x86_64
drwx------. 4 root root      4096 Dec 31  1969 efi
drwx------. 3 root root      4096 Feb 28 10:44 grub2
-rw-------. 1 root root 115794971 Feb 25 16:14 initramfs-0-rescue-71a739e90e9c4c3db10b1b9a885ae3a7.img
-rw-------. 1 root root  40516005 Feb 25 16:14 initramfs-6.5.6-300.fc39.x86_64.img
-rw-------. 1 root root  78123593 Feb 26 07:46 initramfs-6.7.5-200.fc39.x86_64.img
-rw-------. 1 root root  78123299 Feb 28 07:46 initramfs-6.7.6-200.fc39.x86_64.img
drwxr-xr-x. 3 root root      4096 Feb 25 16:12 loader
lrwxrwxrwx. 1 root root        45 Oct 31 21:11 symvers-6.5.6-300.fc39.x86_64.xz -> /lib/modules/6.5.6-300.fc39.x86_64/symvers.xz
lrwxrwxrwx. 1 root root        45 Feb 25 16:32 symvers-6.7.5-200.fc39.x86_64.xz -> /lib/modules/6.7.5-200.fc39.x86_64/symvers.xz
lrwxrwxrwx. 1 root root        45 Feb 28 07:46 symvers-6.7.6-200.fc39.x86_64.xz -> /lib/modules/6.7.6-200.fc39.x86_64/symvers.xz
-rw-------. 1 root root   8706706 Oct  5 20:00 System.map-6.5.6-300.fc39.x86_64
-rw-r--r--. 1 root root   8851363 Feb 16 19:00 System.map-6.7.5-200.fc39.x86_64
-rw-r--r--. 1 root root   8850731 Feb 22 19:00 System.map-6.7.6-200.fc39.x86_64
-rwxr-xr-x. 1 root root  14536168 Feb 25 16:13 vmlinuz-0-rescue-71a739e90e9c4c3db10b1b9a885ae3a7
-rwxr-xr-x. 1 root root  14536168 Oct  5 20:00 vmlinuz-6.5.6-300.fc39.x86_64
-rwxr-xr-x. 1 root root  14794568 Feb 16 19:00 vmlinuz-6.7.5-200.fc39.x86_64
-rwxr-xr-x. 1 root root  14794568 Feb 22 19:00 vmlinuz-6.7.6-200.fc39.x86_64
```

We see that Debian uses `initrd` while Fedora uses `initramfs`. The `vmlinuz-<version>-***` are compressed
Linux kernel image files. The naming schemas are slightly different across distros, but they probably
all contain the same kernel-only (no kernel modules) code for the same version.

According to [Fedora doc](https://docs.fedoraproject.org/en-US/fedora/latest/system-administrators-guide/kernel-module-driver-configuration/Manually_Upgrading_the_Kernel/#sec-Verifying_the_Initial_RAM_Disk_Image),
the job of `initramfs` files is "to preload the block device modules,
such as for IDE, SCSI or RAID, so that the root file system, on which
those modules normally reside, can then be accessed and mounted". This
is in line with Wikipedia's description of GRUB. GRUB is "to actually
boot an operating system by configuring it and starting the kernel". It
does this in a way that requires awareness of "the underlying file
systems, so kernel images are configured and accessed using their actual
file paths". This is probably why `initramfs` files contain and load
drivers (block device modules). `initrd` files on Debian should do the
same thing. Therefore, each pair of `vmlinuz` and `initramfs`/`initrd`
files with the same version number corresponds to one entry in the GRUB
boot menu. Typically, a Linux distro keeps three versions of Linux
kernels on disk, so that users can log into an older version if a kernel
upgrade causes problems.

## Multiboot

The GNU GRUB bootloader is a package of the OS. Therefore, there is no
single bootloader that belongs to all systems. Whichever system is the
first in the boot sequence, its GRUB is the bootloader used. From
[Fedora doc](https://docs.fedoraproject.org/en-US/fedora/latest/system-administrators-guide/kernel-module-driver-configuration/Manually_Upgrading_the_Kernel/#sec-Verifying_the_Initial_RAM_Disk_Image)
it seems that the `grub.cfg` file is the source of entries in the boot menu
(on Debian the full path is `/boot/grub/grub.cfg`; on Fedora it's `/boot/grub2/grub.cfg`).
The contents of this file are the result of running a CLI tool, with inputs from the kernel image files located in `/boot`,
template files located in `/etc/grub.d` and custom settings in `/etc/default/grub`. On Debian this tool is `update-grub`.
On Fedora it's `grub2-mkconfig`. After installing a new kernel version via the distro's package manager,
this CLI tool is automatically run so that boot menu entries get updated.

On multiboot machines, the `grub.cfg` file typically contains explicit entries for *the other distros*.
For example, in Debian's `grub.cfg`, there are entries for "Fedora", and vice versa.
Here lies the twist when upgrading kernel version of one distro on a multiboot machine.

Suppose a machine has dual-boot Debian 12 and Fedora 39, and Debian is the first in the boot sequence,
so its bootloader is used. After upgrading kernel version on Fedora, the new version is not available in Debian's boot menu,
but is if switching to using Fedora's own bootloader. This is because Debian's `grub.cfg` file should be updated
after the kernel update on the Fedora side. Otherwise it's not aware of the new version without "probing".
The other way around is probably also true, unless Fedora does something smarter here.

## Update `grub.cfg`

First make sure the settings in `/etc/default/grub` are still desired. On Debian, `GRUB_DISABLE_OS_PROBER`
should be set to `false` in order for it to "probe" for other distros on the same multiboot machine.

On Debian, run:

```bash
sudo update-grub
```

On Fedora, run:

```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
```
