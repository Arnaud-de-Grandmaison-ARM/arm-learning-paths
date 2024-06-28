---
title: Check your environment
weight: 3

### FIXED, DO NOT MODIFY
layout: learningpathall
---

In this section, you will check that the environment needed to develop with SME2
just works. This will also serve as your first hands on the environment

## Compile the examples

TODO: get rid of the ``--runtime`` arg. This however requires LLVM_ET to be
shipped in the docker image.

First, compile the example codes:

```BASH
shrinkwrap --runtime=docker-local build SME2.yaml
```

TODO: explain ``shrinkwrap``'s output and where the binaries and other files
such as listings, memory maps and so on can be found.

## Basic Sanity checks

Without arguments, ``shrinkwrap`` will run the ``hello`` program. This program
source code is in ``hello.c`` and is looking like:

```C
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    printf("Hello, world !\n");
    return EXIT_SUCCESS;
}
```

Run ``shrinkwrap`` with:

```BASH
shrinkwrap --runtime=docker-local run SME2.yaml
```

You should get an output similar to:

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

The important line is ``[   fvp ] Hello, world !``, which means the "hello
world" program did execute perfectly.

You now know that generic code can be compiled and run on the FVP.

## SME2 checks

You will now run the ``sme2_check`` program, which checks that SME2 works as
expected, in the compiler and in the FVP. The source code is found in
``sme2_check.c``:

```C
#include <stdio.h>
#include <stdlib.h>

#ifdef __ARM_FEATURE_SME2
#include <arm_sme.h>
#else
#error __ARM_FEATURE_SME2 is not defined
#endif

#define get_cpu_ftr(regId, feat, msb, lsb)                                     \
    ({                                                                         \
        unsigned long __val;                                                   \
        __asm__("mrs %0, " #regId : "=r"(__val));                              \
        printf("%-20s: 0x%016lx\n", #regId, __val);                            \
        printf("  - %-10s: 0x%08lx\n", #feat,                                  \
               (__val >> lsb) & ((1 << (msb - lsb)) - 1));                     \
    })

int main(int argc, char *argv[]) {
    get_cpu_ftr(ID_AA64PFR0_EL1, SVE, 35, 32);
    get_cpu_ftr(ID_AA64PFR1_EL1, SME, 27, 24);

    int n = 0;
#ifdef __ARM_FEATURE_SVE
    n = svcntb() * 8;
#endif
    if (n) {
        printf("SVE is available with length %d\n", n);
    } else {
        printf("SVE is unavailable.\n");
        exit(EXIT_FAILURE);
    }

    printf("Checking has_sme: %d\n", __arm_has_sme());
    printf("Checking in_streaming_mode: %d\n", __arm_in_streaming_mode());

    printf("Starting streaming mode...\n");
    __asm__("smstart");

    printf("Checking in_streaming_mode: %d\n", __arm_in_streaming_mode());

    printf("Stopping streaming mode...\n");
    __asm__("smstop");

    printf("Checking in_streaming_mode: %d\n", __arm_in_streaming_mode());

    return EXIT_SUCCESS;
}
```

The ``sme2_check`` program will display the content of some system registers
from the CPU and will then check if SVE is available, then if SME is available,
and finally will switch into streaming monde and back from streaming mode. The
``__ARM_FEATURE_SME2`` and ``__ARM_FEATURE_SVE`` macros are provided by the
compiler when it targets an SME capable target, which is specified with the
``-march=armv9.4-a+sme2`` command line option to ``clang`` in file ``Makefile``.
The ``arm_sme.h`` include file is part of the Arm C library extension
([ACLE](https://arm-software.github.io/acle/main/)). The ACLE provides types and
function declarations to let C/C++ programmers exploit the Arm architecture. We
will use the SME related part of the library, but it also provides support for
Neon or other Arm architectural extensions.

```BASH
shrinkwrap --runtime=docker-local run SME2.yaml --rtvar PROGRAM='${artifact:SME2_CHECK}'
```

The output you get should be similar to:

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

You have checked that code can be compiled and executed with full SME2 support:
you are now all set to move to the next chapter !