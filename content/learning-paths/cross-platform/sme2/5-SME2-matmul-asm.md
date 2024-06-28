---
title: SME2 assembly matrix multiplication
weight: 6

### FIXED, DO NOT MODIFY
layout: learningpathall
---

In this chapter, you will use an SME2 optimized matrix multiplication written directly in assembly.

## Matrix multiplication with SME2 in assembly

### Description

This learning path reuse the assembly version provided in the [SME programmer's
guide](https://developer.arm.com/documentation/109246/0100/matmul-fp32--Single-precision-matrix-by-matrix-multiplication)
where you will find a high level and an in-depth description of the 2 steps
performed. The assembly versions have been modified so they coexist nicely with
the intrinsic versions. In this learning path, the ``preprocess`` function is
defined in ``preprocess_l_asm.S`` and the outer-product based matrix
multiplication is in ``matmul_asm_impl.S``. Those 2 functions have been stitched
together in ``matmul_asm.c`` with the same prototype than the reference
implementation of matrix multiplication, so that a top-level ``matmul_asm`` can
be called from the ``main`` function:

```C
void matmul_asm(uint64_t M, uint64_t K, uint64_t N,
                const float *restrict matLeft, const float *restrict matRight,
                float *restrict matLeft_mod, float *restrict matResult) {
    __asm volatile(""
                   :
                   :
                   : "p0", "p1", "p2", "p3", "p4", "p5", "p6", "p7", "p8", "p9",
                     "p10", "p11", "p12", "p13", "p14", "p15", "z0", "z1", "z2",
                     "z3", "z4", "z5", "z6", "z7", "z8", "z9", "z10", "z11",
                     "z12", "z13", "z14", "z15", "z16", "z17", "z18", "z19",
                     "z20", "z21", "z22", "z23", "z24", "z25", "z26", "z27",
                     "z28", "z29", "z30", "z31");

    preprocess_l_asm(M, K, matLeft, matLeft_mod);
    matmul_asm_impl(M, K, N, matLeft_mod, matRight, matResult);

    __asm volatile(""
                   :
                   :
                   : "p0", "p1", "p2", "p3", "p4", "p5", "p6", "p7", "p8", "p9",
                     "p10", "p11", "p12", "p13", "p14", "p15", "z0", "z1", "z2",
                     "z3", "z4", "z5", "z6", "z7", "z8", "z9", "z10", "z11",
                     "z12", "z13", "z14", "z15", "z16", "z17", "z18", "z19",
                     "z20", "z21", "z22", "z23", "z24", "z25", "z26", "z27",
                     "z28", "z29", "z30", "z31");
}
```

Note the use of the ``__asm`` statement forcing the compiler to save the SVE/SME registers.

The high-level ``matmul_asm`` function is called from ``main.c``:

```C
#include "matmul.h"
#include "misc.h"

#include <inttypes.h>
#include <math.h>
#include <stdio.h>
#include <stdlib.h>

#ifndef __ARM_FEATURE_SME2
#error __ARM_FEATURE_SME2 is not defined
#endif

#ifndef IMPL
#error matmul implementation selection macro IMPL is not defined
#endif

#define STRINGIFY_(I) #I
#define STRINGIFY(I) STRINGIFY_(I)
#define FN(M, I) M##I
#define MATMUL(I, M, K, N, mL, mR, mM, m) FN(matmul_, I)(M, K, N, mL, mR, mM, m)

// Assumptions:
// nbr in matLeft (M): any
// nbc in matLeft, nbr in matRight (K): any K > 2
// nbc in matRight (N): any

int main(int argc, char **argv) {

    /* Size parameters */
    uint64_t M, N, K;
    if (argc >= 4) {
        M = strtoul(argv[1], NULL, 0);
        K = strtoul(argv[2], NULL, 0);
        N = strtoul(argv[3], NULL, 0);
    } else {
        /* Default: 125x35x70 */
        M = 125;
        K = 35;
        N = 70;
    }

    printf("\nSME2 Matrix Multiply fp32 *%s* example with args %lu %lu %lu\n",
           STRINGIFY(IMPL), M, K, N);

    setup_sme();

    const uint64_t SVL = svcntsw();

    /* Calculate M of transformed matLeft.  */
    const uint64_t M_mod = SVL * (M / SVL + (M % SVL != 0 ? 1 : 0));

    float *matRight = (float *)malloc(K * N * sizeof(float));

    float *matLeft = (float *)malloc(M * K * sizeof(float));
    float *matLeft_mod = (float *)malloc(M_mod * K * sizeof(float));
    float *matLeft_mod_ref = (float *)malloc(M_mod * K * sizeof(float));

    float *matResult = (float *)malloc(M * N * sizeof(float));
    float *matResult_ref = (float *)malloc(M * N * sizeof(float));

#ifdef DEBUG
    initialize_matrix(matLeft, M * K, LINEAR_INIT);
    initialize_matrix(matRight, K * N, LINEAR_INIT);
    initialize_matrix(matLeft_mod, M_mod * K, DEAD_INIT);
    initialize_matrix(matResult, M * N, DEAD_INIT);

    print_matrix(M, K, matLeft, "matLeft");
    print_matrix(K, N, matRight, "matRight");
#else
    initialize_matrix(matLeft, M * K, RANDOM_INIT);
    initialize_matrix(matRight, K * N, RANDOM_INIT);
#endif

    MATMUL(IMPL, M, K, N, matLeft, matRight, matLeft_mod, matResult);

    // Compute the reference values with the vanilla implementations.
    matmul(M, K, N, matLeft, matRight, matResult_ref);
    preprocess_l(M, K, matLeft, matLeft_mod_ref);

    unsigned error = compare_matrices(K, M_mod, matLeft_mod_ref, matLeft_mod,
                                      "Matrix preprocessing");
    if (!error)
        error = compare_matrices(M, N, matResult_ref, matResult,
                                 "Matrix multiplication");

    free(matRight);

    free(matLeft);
    free(matLeft_mod);
    free(matLeft_mod_ref);

    free(matResult);
    free(matResult_ref);

    return error ? EXIT_FAILURE : EXIT_SUCCESS;
}
```

The actual matrix multiplication implementation is provided thru the ``IMPL``
macro. Depending on the ``M``, ``K``, ``N`` dimension parameters, ``main`` will
allocate memory for all the matrices and initialize  ``matLeft`` and
``matRight`` with random data. It will then run the matrix multiplication from
``IMPL`` and compute the reference values for the preprocessed matrix as well as
the result matrix. It then compares the actual values to the reference values
and report errors (if any). Last, all the memory is de-allocated before exiting
the program with a success or failure return code.

### Compile and run it

```BASH
shrinkwrap --runtime=docker-local build SME2.yaml
shrinkwrap --runtime=docker-local run SME2.yaml --rtvar PROGRAM='${artifact:SME2_MATMUL_ASM}'
```

which should output something similar to:

```TXT
Press '^]' to quit shrinkwrap.
All other keys are passed through.
Environment ip address: 172.17.0.2.

[   fvp ] terminal_0: Listening for serial connection on port 5000
[   fvp ] terminal_1: Listening for serial connection on port 5001
[   fvp ] terminal_3: Listening for serial connection on port 5002
[   fvp ] terminal_2: Listening for serial connection on port 5003
[   fvp ]
[   fvp ] SME2 Matrix Multiply fp32 *asm* example with args 125 35 70
[   fvp ] Matrix preprocessing: PASS !
[   fvp ] Matrix multiplication: PASS !
[   fvp ]
[   fvp ] Info: /OSCI/SystemC: Simulation stopped by user.
[   fvp ]
[   fvp ] --- FVP_Base_RevC_2xAEMvA statistics: -----------------------------------------
[   fvp ] Simulated time                          : 0.026200s
[   fvp ] User time                               : 0.097437s
[   fvp ] System time                             : 0.018030s
[   fvp ] Wall time                               : 0.136308s
[   fvp ] Performance index                       : 0.19
[   fvp ] cluster0.cpu0                           :  19.23 MIPS (     2620572 Inst)
[   fvp ] Memory highwater mark                   : 0x2d171000 bytes ( 0.705 GB )
[   fvp ] -------------------------------------------------------------------------------
```
