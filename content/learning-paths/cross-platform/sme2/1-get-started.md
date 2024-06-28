---
title: Get started
weight: 2

### FIXED, DO NOT MODIFY
layout: learningpathall
---

In this section, you will setup the environment needed to develop with SME2.

## Before you begin

SM2 hardware is not widely deployed at the time of writing, so you will have to
use:

 - [QEMU](https://www.qemu.org/) in order to execute those new instructions. A
   fairly recent version of `QEMU` is required to run the example code described
   here (TODO: which version ?).
 - a compiler with support for SME2 instructions. [Clang](https://www.llvm.org/)
   version >= 18 or [gcc](https://gcc.gnu.org/) versions >= 14

This learning path assumes you are using an Arm based machine with Ubuntu 24.04
installed. You can install the required tools with:

```BASH
sudo apt install build-essential clang ninja-build -y
```

### MacOS

At the time of writing, `QEMU` user-space emulation is not supported on MacOS.
This can be worked around by using `QEMU` system emulation mode and compiling
the example codes to baremetal with the [LLVM embedded toolchain for
Arm](https://github.com/ARM-software/LLVM-embedded-toolchain-for-Arm).

### Windows

TBD

## Suggested reading

If you are not familiar with SVE and / or SME, it is suggested that you first
read some material :

 - [Introducing the Scalable Matrix Extension for the Armv9-A
   Architecture](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/scalable-matrix-extension-armv9-a-architecture)
 - [Arm Scalable Matrix Extension (SME) Introduction (part
   1)](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/arm-scalable-matrix-extension-introduction)
 - [Arm Scalable Matrix Extension (SME) Introduction (part
   2)](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/arm-scalable-matrix-extension-introduction-p2)