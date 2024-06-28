---
title: SME2 assembly matrix multiplication
weight: 4

### FIXED, DO NOT MODIFY
layout: learningpathall
---

In this chapter, you will write an SME2 optimized matrix multiplication in assembly.

## Matrix multiplication with SME2 in assembly

### Description

TODO: Talk about matrix transposition, a necessary step to use SME2 and why.

TODO: describe matrix multiplication with SME2.

There is a a high level + in-depth description of the 2 steps in the [SME programmer's guide](https://developer.arm.com/documentation/109246/0100/matmul-fp32--Single-precision-matrix-by-matrix-multiplication)

### Assembly implementation

The SME2 matrix multiplication steps are in ``preprocess_l_asm.S`` and ``matmul_asm.S``. Those files define 2 functions: ``preprocess_l_asm`` and ``matmul_asm`` that are used in ``main_asm.c``:

```C
    __asm volatile ("" : : :"p0", "p1", "p2", "p3", "p4", "p5", "p6", "p7",
                         "p8", "p9", "p10", "p11", "p12", "p13", "p14", "p15",
                         "z0", "z1", "z2", "z3", "z4", "z5", "z6", "z7",
                         "z8", "z9", "z10", "z11", "z12", "z13", "z14", "z15",
                         "z16", "z17", "z18", "z19", "z20", "z21", "z22", "z23",
                         "z24", "z25", "z26", "z27", "z28", "z29", "z30", "z31");

    preprocess_l_asm(M, K, matLeft, matLeft_mod);
    matmul_asm(M, K, N, matLeft_mod, matRight, matResult_opt);

    __asm volatile ("" : : :"p0", "p1", "p2", "p3", "p4", "p5", "p6", "p7",
                         "p8", "p9", "p10", "p11", "p12", "p13", "p14", "p15",
                         "z0", "z1", "z2", "z3", "z4", "z5", "z6", "z7",
                         "z8", "z9", "z10", "z11", "z12", "z13", "z14", "z15",
                         "z16", "z17", "z18", "z19", "z20", "z21", "z22", "z23",
                         "z24", "z25", "z26", "z27", "z28", "z29", "z30", "z31");
```

Note the use of the ``__asm`` statement forcing the compiler to save the SVE/SME registers.

Compile and run it with:

```BASH
shrinkwrap --runtime=null build SME2.yaml
shrinkwrap run SME2.yaml --rtvar PROGRAM=build/shrinkwrap/package/SME2/sme2_matmul_asm
```

which should output something similar to:

```TXT
Press '^]' to quit shrinkwrap.
All other keys are passed through.
Environment ip address: 172.17.0.2.

[   fvp ] terminal_0: Listening for serial connection on port 5000
[   fvp ] terminal_1: Listening for serial connection on port 5001
[   fvp ] terminal_2: Listening for serial connection on port 5002
[   fvp ] terminal_3: Listening for serial connection on port 5003
[   fvp ]
[   fvp ] SME2 Matrix Multiply fp32 example
[   fvp ] PASS
[   fvp ]
[   fvp ] Info: /OSCI/SystemC: Simulation stopped by user.
[   fvp ]
[   fvp ] --- FVP_Base_RevC_2xAEMvA statistics: -----------------------------------------
[   fvp ] Simulated time                          : 0.031725s
[   fvp ] User time                               : 0.115192s
[   fvp ] System time                             : 0.010602s
[   fvp ] Wall time                               : 0.149217s
[   fvp ] Performance index                       : 0.21
[   fvp ] cluster0.cpu0                           :  21.31 MIPS (     3179640 Inst)
[   fvp ] Memory highwater mark                   : 0x2a713000 bytes ( 0.663 GB )
[   fvp ] -------------------------------------------------------------------------------
```
