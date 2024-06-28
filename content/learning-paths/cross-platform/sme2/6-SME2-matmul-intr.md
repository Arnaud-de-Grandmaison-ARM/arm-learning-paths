---
title: SME2 intrinsics matrix multiplication
weight: 7

### FIXED, DO NOT MODIFY
layout: learningpathall
---

In this chapter, you will write an SME2 optimized matrix multiplication in C
using the intrinsics provided by the compiler.

## Matrix multiplication with SME2 intrinsics

*Intrinsics*, also know known as *compiler intrinsics* or *intrinsic functions*
are functions available to the application developers that the compiler has an
intimate knowledge of. This enables the compiler to either translate that
function to a very specific instruction and/or to perform specific
optimizations.

You can lean more about intrinsics on this [wikipedia
article](https://en.wikipedia.org/wiki/Intrinsic_function).

Using intrinsics allows the programmer to use the very specific instructions
needed to achieve the required performance while writing in C all the mundane
code (loops, ...). This gives performance close to what can be reached with hand
written assembly and is way more maintainable and portable !

All Arm specific intrinsics are specified in the
[ACLE](https://github.com/ARM-software/acle) --- Arm C language extension. ACLE
is supported by the main compilers, most notably [GCC](https://gcc.gnu.org/) and
[clang](https://clang.llvm.org).

## Implementation

The assembly version is very low level, and does not deal properly with the SME
state. In a real world large scale software, the program will move back and
forth from streaming mode, and some streaming mode routines will call other
streaming mode routines, which means that some state (including the ZA storage)
needs to be saved and restored. This is hopefully defined in the ACLE and
supported by the compiler: the programmer *just* has to annotate the function
with some keywords and set up some registers (see function ``setup_sme`` in
``misc.c`` for an example). See [Introduction to streaming and non-streaming
mode](https://arm-software.github.io/acle/main/acle.html#controlling-the-use-of-streaming-mode)
for further details.

Here again, a top level function named ``matmul_intr`` in ``matmul_intr.c``
will be used to stitch together the preprocessing and the multiplication:

```C
__arm_new("za") __arm_locally_streaming void matmul_intr(
    uint64_t M, uint64_t K, uint64_t N, const float *restrict matLeft,
    const float *restrict matRight, float *restrict matLeft_mod,
    float *restrict matResult) {
    preprocess_l_intr(M, K, matLeft, matLeft_mod);
    matmul_intr_impl(M, K, N, matLeft_mod, matRight, matResult);
}
```

Note the ``__arm_new("za")`` and ``__arm_locally_streaming`` that will make the
compiler save the ZA storage so we can use it without destroying its content if
it was still in use by one of the callers.

### Matrix preprocessing

```C
void preprocess_l_intr(
    uint64_t M, uint64_t K, const float *restrict a,
    float *restrict a_mod) __arm_streaming __arm_inout("za") {
    const uint64_t SVL = svcntsw();
    const uint64_t M_mod = SVL * (M / SVL + (M % SVL != 0 ? 1 : 0));
    uint64_t row = 0;
    svbool_t MDim_predicate = svwhilelt_b32(row, M);
    // The outer loop, iterating over rows (M dimension)
    do {
        uint64_t col = 0;
        svcount_t KDim_predicate = svwhilelt_c32(col, K, 2);
        // The inner loop, iterating on columns (K dimension).
        do {
            // Load-as-rows loop
            for (uint64_t trow = 0; trow < SVL; trow += 4) {
                svcount_t pn10 =
                    svpsel_lane_c32(KDim_predicate, MDim_predicate, trow + 0);
                svcount_t pn11 =
                    svpsel_lane_c32(KDim_predicate, MDim_predicate, trow + 1);
                svcount_t pn12 =
                    svpsel_lane_c32(KDim_predicate, MDim_predicate, trow + 2);
                svcount_t pn13 =
                    svpsel_lane_c32(KDim_predicate, MDim_predicate, trow + 3);

                const uint64_t tile_UL_corner = (row + trow) * K + col;
                svfloat32x2_t z20_z28 = svld1_x2(pn10, &a[tile_UL_corner]);
                svfloat32x2_t z21_z29 = svld1_x2(pn11, &a[tile_UL_corner + K]);
                svfloat32x2_t z22_z30 =
                    svld1_x2(pn12, &a[tile_UL_corner + 2 * K]);
                svfloat32x2_t z23_z31 =
                    svld1_x2(pn13, &a[tile_UL_corner + 3 * K]);

                svfloat32x4_t z20_z21_z22_z23 =
                    svcreate4(svget2(z20_z28, 0), svget2(z21_z29, 0),
                              svget2(z22_z30, 0), svget2(z23_z31, 0));
                svfloat32x4_t z28_z29_z30_z31 =
                    svcreate4(svget2(z20_z28, 1), svget2(z21_z29, 1),
                              svget2(z22_z30, 1), svget2(z23_z31, 1));
                svwrite_hor_za32_f32_vg4(
                    /* tile: */ 0, /* slice: */ trow, z20_z21_z22_z23);
                svwrite_hor_za32_f32_vg4(
                    /* tile: */ 1, /* slice: */ trow, z28_z29_z30_z31);
            }

            // Read-as-columns and store loop
            const uint64_t dest_0 = row * K + col * SVL;
            const uint64_t dest_1 = dest_0 + SVL * SVL;
            for (uint64_t tcol = 0; tcol < SVL; tcol += 4) {
                svcount_t pn10 =
                    svwhilelt_c32(dest_0 + tcol * SVL, K * M_mod, 4);
                svcount_t pn11 =
                    svwhilelt_c32(dest_1 + tcol * SVL, K * M_mod, 4);
                svfloat32x4_t z0_z3 =
                    svread_ver_za32_f32_vg4(/* tile: */ 0, /* slice: */ tcol);
                svfloat32x4_t z4_z7 =
                    svread_ver_za32_f32_vg4(/* tile: */ 1, /* slice: */ tcol);
                svst1(pn10, &a_mod[dest_0 + tcol * SVL], z0_z3);
                svst1(pn11, &a_mod[dest_1 + tcol * SVL], z4_z7);
            }

            // Update column counter.
            col += SVL;
            KDim_predicate = svwhilelt_c32(col, K, 2);
            // I would like to use this instead:
            //} while (svptest_first(svptrue_b32(),
            // svreinterpret_b(KDim_predicate)));
        } while (col < K);

        // Update row counter.
        row += SVL;
        MDim_predicate = svwhilelt_b32(row, M);
        // I would like to use this instead:
        // } while (svptest_first(svptrue_b32(), MDim_predicate));
    } while (row < M);
}
```

Note that ``preprocess_l_intr`` has been annotated with ``__arm_streaming``
(because this function is using streaming instructions) and
``__arm_inout("za")`` (because this function reuses the ZA storage from its
caller).

TODO: describe what the code does.

### Outer-product multiplication

```C
void matmul_intr_impl(
    uint64_t M, uint64_t K, uint64_t N, const float *restrict matLeft_mod,
    const float *restrict matRight,
    float *restrict matResult) __arm_streaming __arm_inout("za") {
    const uint64_t SVL = svcntsw();

    // Build the result matrix tile by tile.
    // TODO: process 2x2 tiles at the same time, like in the asm code.
    for (uint64_t row = 0; row < M; row += SVL) {
        svbool_t MDim_predicate = svwhilelt_b32(row, M);
        for (uint64_t col = 0; col < N; col += SVL) {
            svbool_t NDim_predicate = svwhilelt_b32(col, N);

            // Outer product + accumulation
            svzero_za();
            const uint64_t matLeft_pos = row * K;
            const uint64_t matRight_UL_corner = col;
            for (uint64_t k = 0; k < K; k++) {
                svfloat32_t zL =
                    svld1(MDim_predicate, &matLeft_mod[matLeft_pos + k * SVL]);
                svfloat32_t zR = svld1(NDim_predicate,
                                       &matRight[matRight_UL_corner + k * N]);
                svmopa_za32_m(0, MDim_predicate, NDim_predicate, zL, zR);
            }

            // Store ZA.
            const uint64_t result_tile_UL_corner = row * N + col;
            for (uint64_t trow = 0; trow < SVL; trow += 4) {
                // Load 4 lines from ZA
                svfloat32x4_t zr = svread_hor_za32_f32_vg4(
                    /* tile: */ 0, /* slice: */ trow);

                // And store them to matResult.
                svst1(NDim_predicate,
                      &matResult[result_tile_UL_corner + (trow + 0) * N],
                      svget4(zr, 0));
                svst1(NDim_predicate,
                      &matResult[result_tile_UL_corner + (trow + 1) * N],
                      svget4(zr, 1));
                svst1(NDim_predicate,
                      &matResult[result_tile_UL_corner + (trow + 2) * N],
                      svget4(zr, 2));
                svst1(NDim_predicate,
                      &matResult[result_tile_UL_corner + (trow + 3) * N],
                      svget4(zr, 3));
            }
        }
    }
}
```

TODO: describe !

### Compile and run

The main function is exactly the same that was used for the assembly version,
with the ``IMPL`` macro defined to be ``intr`` in the ``Makefile``.

First, make sure that the ``sme2_matmul_intr`` executable is up to date:

```BASH
docker run --rm -v "$PWD:/home/ubuntu/work" -w /home/ubuntu/work sme2-environment make sme2_matmul_intr
```

Then execute ``sme2_matmul_intr`` on the FVP:

```BASH
docker run --rm -v "$PWD:/home/ubuntu/work" -w /home/ubuntu/work sme2-environment ./run-fvp.sh sme2_matmul_intr
```

which should output something similar to:

```TXT
SME2 Matrix Multiply fp32 *intr* example with args 125 35 70
Matrix preprocessing: PASS !
Matrix multiplication: PASS !

Info: /OSCI/SystemC: Simulation stopped by user.
```
