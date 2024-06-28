---
title: Outer product
weight: 5

### FIXED, DO NOT MODIFY
layout: learningpathall
---

In this section, you will learn how the outer product can be used to improve matrix multiplication.

TODO: stuff !

TODO: Talk about matrix transposition, a necessary step to use SME2 and why.

TODO: describe matrix multiplication with SME2.

TODO: use here the reference implementation for ``preprocess``.

```C
void preprocess_l(uint64_t nbr, uint64_t nbc, const float *restrict a,
                  float *restrict a_mod) {
    const uint64_t SVL = svcntsw();

    // For all blocks of SVL x SVL data
    for (uint64_t By = 0; By < nbr; By += SVL)
        for (uint64_t Bx = 0; Bx < nbc; Bx += SVL) {
            // For this block of data
            const uint64_t dest = By * nbc + Bx * SVL;
            for (uint64_t j = 0; j < SVL; j++)
                for (uint64_t i = 0; i < SVL && (Bx + i) < nbc; i++)
                    a_mod[dest + i * SVL + j] =
                        By + j < nbr ? a[(By + j) * nbc + Bx + i] : 0.0;
        }
}
```