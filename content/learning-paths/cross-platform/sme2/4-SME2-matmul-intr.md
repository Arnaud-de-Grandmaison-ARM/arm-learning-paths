---
title: SME2 intrinsics matrix multiplication
weight: 5

### FIXED, DO NOT MODIFY
layout: learningpathall
---

In this chapter, you will write an SME2 optimized matrix multiplication in C
using the intrinsics provided by the compiler.

## Matrix multiplication with SME2 intrinsics

### Description

*Intrinsics*, also know known as *compiler intrinsics* or *intrinsic functions*
are functions available to the application developers that the compiler has an
intimate knowledge of. This enables the compiler to either translate that
function to a very specific instruction and/or to perform specific
optimizations.

You can lean more about intrinsics on this [wikipedia
article](https://en.wikipedia.org/wiki/Intrinsic_function).

All Arm specific intrinsics are specified in the
[ACLE](https://github.com/ARM-software/acle) --- Arm C language extension. ACLE
is supported by the main compilers, most notably [GCC](https://gcc.gnu.org/) and
[clang](https://clang.llvm.org).

TODO

### Implementation

Compile and run it with:

```BASH
shrinkwrap --runtime=null build SME2.yaml
shrinkwrap run SME2.yaml --rtvar PROGRAM=build/shrinkwrap/package/SME2/sme2_matmul_intr
```

which should output something similar to:

```TXT
TODO
```
