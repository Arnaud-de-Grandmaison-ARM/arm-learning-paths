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
   [FVP](https://developer.arm.com/Tools%20and%20Software/Fixed%20Virtual%20Platforms)
   and [QEMU](https://www.qemu.org/) are the two usual choices. At the time of
   writing this learning path, ``QEMU`` does not yet have SME2 support, so you
   will be using an ``FVP`` instead.

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

Next, install the [LLVM embedded
toolchain](https://github.com/ARM-software/LLVM-embedded-toolchain-for-Arm/releases)
at version 18.1.3 for your machine (``Aarch64`` or ``x86_64``). Using an Arm machine as an example:

```BASH
wget https://github.com/ARM-software/LLVM-embedded-toolchain-for-Arm/releases/download/release-18.1.3/LLVM-ET-Arm-18.1.3-Linux-AArch64.tar.xz
tar xfJ LLVM-ET-Arm-18.1.3-Linux-AArch64.tar.xz
export LLVM_ET_INSTALL_DIR=$PWD/LLVM-ET-Arm-18.1.3-Linux-AArch64
```

At this point, it's probably safe to restart your machine if the ``apt upgrade``
performed some operations. If you were not part of the ``docker`` group before,
you will need to re-login in order for the change to be taken into account.

### MacOS

Using the ``brew`` install the prerequisites with:

```BASH
brew install make git netcat python3 telnet
```

Install ``docker`` following the instructions for MacOS on
[docker](https://www.docker.com/).

Next, install the [LLVM embedded
toolchain](https://github.com/ARM-software/LLVM-embedded-toolchain-for-Arm/releases)
at version 18.1.3 by downloading the ``dmg`` file and running it.

### Windows

TBD

## Environment

Using ``git``, clone the environment for experimenting with SME2 to a directory
named ``SME2.git``:

```BASH
git clone TODO_SOME_URL SME2.git
```

And change directory to your checkout and setup the environment:

```BASH
cd SME2.git
source setup.source_me
```

From now on, all instructions assumes your current directory is ``SME2.git`` and
that you have sourced ``setup.source`` to set up the environment.

### Behind the curtain

This learning path makes use of
[shrinkwrap](https://gitlab.arm.com/tooling/shrinkwrap) to simplify the process
of building our examples and running them on the FVP.

``setup.source`` will clone
[shrinkwrap](https://gitlab.arm.com/tooling/shrinkwrap) if it has not already
been cloned. It will then create a ``python`` virtual environment and install
some modules required by ``shrinkwrap``. And finally, it will add ``shrinkwrap``
to the searching path and setup some environment variables for ``shrinkwrap``.

## Test your environment

You should now check that your SME2 environment works.

TODO: get rid of the ``--runtime`` arg. This however requires LLVM_ET to be
shipped in the docker image.

First, compile the example codes:

```BASH
shrinkwrap --runtime=null build SME2.yaml
```

### Basic Sanity checks

```BASH
shrinkwrap run SME2.yaml
```

This will a default sample "hello wold" program (which prints *Hello
world !*) with the FVP and should provide an output similar to:

```TXT
Press '^]' to quit shrinkwrap.
All other keys are passed through.
Environment ip address: 172.17.0.2.

[   fvp ] terminal_0: Listening for serial connection on port 5000
[   fvp ] terminal_1: Listening for serial connection on port 5001
[   fvp ] terminal_2: Listening for serial connection on port 5002
[   fvp ] terminal_3: Listening for serial connection on port 5003
[   fvp ] Hello, world !
[   fvp ]
[   fvp ] Info: /OSCI/SystemC: Simulation stopped by user.
[   fvp ]
[   fvp ] --- FVP_Base_RevC_2xAEMvA statistics: -----------------------------------------
[   fvp ] Simulated time                          : 0.000080s
[   fvp ] User time                               : 0.004645s
[   fvp ] System time                             : 0.001310s
[   fvp ] Wall time                               : 0.009364s
[   fvp ] Performance index                       : 0.01
[   fvp ] cluster0.cpu0                           :   0.91 MIPS (        8484 Inst)
[   fvp ] Memory highwater mark                   : 0x2a19a000 bytes ( 0.658 GB )
[   fvp ] -------------------------------------------------------------------------------
```

You now know that generic code can be compiled and run on your system.

### SME2 checks

```BASH
shrinkwrap run SME2.yaml --rtvar PROGRAM=build/shrinkwrap/package/SME2/sme2_check
```

This runs a simple program that checks if the compiler was able to use SVE /
SME2 instructions, and that the FVP models those instructions. The output you
get should be similar to:

```TXT
Press '^]' to quit shrinkwrap.
All other keys are passed through.
Environment ip address: 172.17.0.2.

[   fvp ] terminal_0: Listening for serial connection on port 5000
[   fvp ] terminal_1: Listening for serial connection on port 5001
[   fvp ] terminal_2: Listening for serial connection on port 5002
[   fvp ] terminal_3: Listening for serial connection on port 5003
[   fvp ] ID_AA64PFR0_EL1     : 0x1101101131111112
[   fvp ]   - SVE       : 0x00000001
[   fvp ] ID_AA64PFR1_EL1     : 0x0000101002000001
[   fvp ]   - SME       : 0x00000002
[   fvp ] SVE is available with length 512
[   fvp ] Checking has_sme: 1
[   fvp ] Checking in_streaming_mode: 0
[   fvp ] Starting streaming mode...
[   fvp ] Checking in_streaming_mode: 1
[   fvp ] Stopping streaming mode...
[   fvp ] Checking in_streaming_mode: 0
[   fvp ]
[   fvp ] Info: /OSCI/SystemC: Simulation stopped by user.
[   fvp ]
[   fvp ] --- FVP_Base_RevC_2xAEMvA statistics: -----------------------------------------
[   fvp ] Simulated time                          : 0.000200s
[   fvp ] User time                               : 0.012575s
[   fvp ] System time                             : 0.014961s
[   fvp ] Wall time                               : 0.038825s
[   fvp ] Performance index                       : 0.01
[   fvp ] cluster0.cpu0                           :   0.52 MIPS (       20200 Inst)
[   fvp ] Memory highwater mark                   : 0x2a5dd000 bytes ( 0.662 GB )
[   fvp ] -------------------------------------------------------------------------------
```

## Suggested reading

If you are not familiar with matrix multiplication --- or need a refresh ;-) ---
this [wikipedia article on Matrix
multiplication](https://en.wikipedia.org/wiki/Matrix_multiplication) is a good
start.

If you are not familiar with SVE and / or SME, it is strongly suggested that you first
read some material :

 - [Introducing the Scalable Matrix Extension for the Armv9-A
   Architecture](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/scalable-matrix-extension-armv9-a-architecture)
 - [Arm Scalable Matrix Extension (SME) Introduction (part
   1)](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/arm-scalable-matrix-extension-introduction)
 - [Arm Scalable Matrix Extension (SME) Introduction (part
   2)](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/arm-scalable-matrix-extension-introduction-p2)

You are now all set to move to the next chapter !