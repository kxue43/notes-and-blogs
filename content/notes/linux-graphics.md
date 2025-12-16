---
author: Ke Xue
title: Linux Graphics
date: 2024-11-04T21:50:02-05:00
draft: true
layout: docs
description: General information on Linux graphics.
tags: 
---


The relationship among graphics card, graphics card driver (meaning "kernel mode driver" here),
OpenGL/Vulkan and widget toolkit (GTK, GT) is like the relationship among database engine, PEP-249 package,
SQLAlchemy and package that might conditionally use SQLAlchemy.

Graphics card drivers are classified as "kernel mode drivers" and
"user mode drivers", but the relationship between these two types is
still similar to that between PEP-249 packages and SQLAlchemy. OpenGL and
Vulkan implementations are user mode drivers. Mesa 3D is basically the
collection of user mode drivers for Intel and AMD graphics cards. NVIDIA
proprietary user mode drivers are not part of Mesa 3D though.

References:

- <https://en.wikipedia.org/wiki/Mesa_%28computer_graphics%29%23Software_architecture>
- <https://www.youtube.com/watch?v=CW1CLcT83as>

Kernel mode drivers are kernel modules that interface with the
underlying hardware. As there are many different drivers, there's need
for a uniformization layer, which is served by OpenGL and Vulkan. Not
only that, these APIs are usually used to interact with GPU to achieve
*hardware-accelerated rendering*.


> [!NOTE]
> As a side note, one key difference between X11 and Wayland is that, in Wayland, clients use user mode drivers
> to directly tell GPU to render and only tell the Wayland server that "something is about to render";
> on the other hand, with X11 it is the server that performs the "render calls".

Both OpenGL and Vulkan are API specifications. OpenGL is easier to use.
Vulkan exposes more low-level operations and is more powerful for
programmers that know how to use it. The same names are probably also
used to refer to some widely used implementations of these APIs.
Nonetheless, there are more than one implementations. For example,
Apple's Metal API and Microsoft's Direct3D 12 are both compatible with
Vulkan. On the other hand, Mesa 3D is "an open source implementation of
OpenGL, Vulkan, and other graphics API specifications".

GTK and QT provide library code for creating GUI widgets, but how these
widgets get rendered on the screen probably depends on how the code is
compiled. It might be possible to compile the code so that at runtime
the program probes the OS for available options and choose the right
one. Bottomline, both GTK and QT can work with OpenGL, maybe Vulkan as
well. In addition, both GTK and QT possibly have code that interact with
X11 and Wayland servers. Basically, GTK and GT are much higher-level
abstractions than the underlying protocols like X11, Wayland, OpenGL and
Vulkan.
