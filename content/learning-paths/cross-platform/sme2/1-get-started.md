---
title: Get started
weight: 2

### FIXED, DO NOT MODIFY
layout: learningpathall
---

In this section, you will setup the environment needed to develop with SME2.

SME2 hardware is not widely deployed at the time of writing of this learning
path, so you will have to use:

 - a compiler with support for SME2 instructions. [clang](https://www.llvm.org/)
   version >= 18 or [gcc](https://gcc.gnu.org/) versions >= 14. This learning
   path will use ``clang``.

 - an emulator to execute code with those new instructions as they are most
   probably not yet supported by your machine's hardware.
   In this learning path, we will use the
   [FVP](https://developer.arm.com/Tools%20and%20Software/Fixed%20Virtual%20Platforms).

This learning path relies on
[shrinkwrap](https://gitlab.arm.com/tooling/shrinkwrap) to simplify the process
of building our examples and running them on the FVP. ``shrinkwrap`` bundles
together the compiler toolchain as well as the FVP that will be used.
``shrinkwrap`` is making use of ``docker`` so you will need to have it installed
together with a few other programs.

## Prerequisites

Depending on your machine, jump to one of the below sections to install the
required prerequisites before continuing with this learning path:

### Linux

This learning path assumes you are using an Ubuntu 24.04 machine. You may need
to adapt this command line if you are using a different Ubuntu version or a
different distribution.

```BASH
sudo apt update
sudo apt upgrade
sudo apt install build-essential git netcat-openbsd python3 python3-venv telnet docker.io wget tar -y
sudo usermod -aG docker $USER
```

At this point, it's probably safe to restart your machine if the ``apt upgrade``
performed some operations. If you were not part of the ``docker`` group before,
you will need to re-login in order for the group change to be taken into account.

### MacOS

Using the ``brew`` install the prerequisites with:

```BASH
brew install make git netcat python3 telnet
```

Install ``docker`` following the instructions for MacOS on
[docker](https://www.docker.com/).

## Environment

Using ``git``, clone the environment for experimenting with SME2 to a directory
named ``SME2.git``:

```BASH
git clone TODO_SOME_URL SME2.git
```

TODO: give an overview of ``SME2.git``'s content.

```TXT
SME2.git
├── .git/
├── .clang-format
├── .gitignore
├── Makefile
├── README.rst
├── config
│   ├── SME2_LP.yaml
│   └── TarmacTrace.yaml
├── hello.c
├── main.c
├── matmul.h
├── matmul_asm.c
├── matmul_asm_impl.S
├── matmul_intr.c
├── matmul_vanilla.c
├── misc.c
├── misc.h
├── preprocess_l_asm.S
├── preprocess_vanilla.c
├── setup.source_me
└── sme2_check.c
```

Change directory to your checkout and setup the environment:

```BASH
cd SME2.git
source setup.source_me
```

From now on, all instructions assumes your current directory is ``SME2.git`` and
that you have sourced ``setup.source`` to set up the environment.

### Behind the curtain

``setup.source`` will clone
[shrinkwrap](https://gitlab.arm.com/tooling/shrinkwrap) if it has not already
been cloned. ``shrinkwrap`` is a tool to simplify the process of building and
running firmware on Arm Fixed Virtual Platforms (FVP). It provides a number of
configurations that can be used out-of-the-box as well as enabling users to
compose and extend them in a manner that can be easily shared and reused.

It will then create a ``python`` virtual environment and install some modules
required by ``shrinkwrap`` into the virtual environment in order not to pollute
your machine system. And finally, it will add ``shrinkwrap`` to the searching
path and setup some environment variables for ``shrinkwrap``.

## Suggested reading

If you are not familiar with matrix multiplication --- or need a refresh ;-) ---
this [wikipedia article on Matrix
multiplication](https://en.wikipedia.org/wiki/Matrix_multiplication) is a good
start.

If you are not familiar with SVE and / or SME, it is strongly suggested that you
first read some material as this learning path assumes some basic understanding
of those technologies:

 - [Introducing the Scalable Matrix Extension for the Armv9-A
   Architecture](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/scalable-matrix-extension-armv9-a-architecture)
 - [Arm Scalable Matrix Extension (SME) Introduction (part
   1)](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/arm-scalable-matrix-extension-introduction)
 - [Arm Scalable Matrix Extension (SME) Introduction (part
   2)](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/arm-scalable-matrix-extension-introduction-p2)

You are now all set to move to the next chapter !