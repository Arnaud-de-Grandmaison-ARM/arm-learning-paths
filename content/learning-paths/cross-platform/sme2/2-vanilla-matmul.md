---
title: Vanilla matrix multiplication
weight: 3

### FIXED, DO NOT MODIFY
layout: learningpathall
---

In this chapter, you will write a by-the-book matrix multiplication in C.

## Vanilla matrix multiplication

### Algorithm description

The vanilla matrix multiplication operation takes two input matrices, A [Al
lines x Ac columns] and B [Bl lines x Bc columns], to produce an output matrix C
[Cl lines x Cc columns]. The operation consists in iterating on each row of A
and column of B, multiplying each element of the A row with its corresponding
element in the B column then summing all these products. This implies that the
A, B and C matrices have some constraint on their dimensions:

- A's number of columns need to match B's number of lines: Ac == Bl
- C will have dimensions Cl == Al and Cc == Bc

### C implementation

A literal implementation of the above algorithm can be found in ``matmul.c``:

```C
void matmul(    uint64_t nbr_l, uint64_t nbc_l, uint64_t nbc_r,
                const float * restrict a, const float * restrict b, float * restrict c)
{
    for (uint64_t m=0; m<nbr_l; m++) {
        for (uint64_t n=0; n<nbc_r; n++) {

            float acc = 0.0;

            for (uint64_t k=0; k<nbc_l; k++)
                acc += (float) a[m*nbc_l + k] * (float) b[k*nbc_r + n];

            c[m*nbc_r + n] = acc;
        }
    }
}
```

In this learning path, the matrices are layed in memory as a contiguous
sequences of elements, in row major order. The ``matmul`` function performs the
algorithm that was described above. The pointers to A, B and C have been
annotated as ``restrict``, which informs the compiler that the memory areas
designated by those pointer do not alias (i.e. they do not overlap in any way),
so the compiler does not need to insert extra instructions to deal with those
cases. The pointers to A and B are marked as ``const`` because those 2 matrices
are not modified by ``matmul``.

Compile and run it with:

```BASH
shrinkwrap --runtime=null build SME2.yaml
shrinkwrap run SME2.yaml --rtvar PROGRAM=build/shrinkwrap/package/SME2/sme2_matmul_1
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
[   fvp ] Matrix Multiply fp32 example
[   fvp ]
[   fvp ] Info: /OSCI/SystemC: Simulation stopped by user.
[   fvp ]
[   fvp ] --- FVP_Base_RevC_2xAEMvA statistics: -----------------------------------------
[   fvp ] Simulated time                          : 0.028000s
[   fvp ] User time                               : 0.070036s
[   fvp ] System time                             : 0.004807s
[   fvp ] Wall time                               : 0.083411s
[   fvp ] Performance index                       : 0.34
[   fvp ] cluster0.cpu0                           :  33.59 MIPS (     2801499 Inst)
[   fvp ] Memory highwater mark                   : 0x2a5e2000 bytes ( 0.662 GB )
[   fvp ] -------------------------------------------------------------------------------
```
