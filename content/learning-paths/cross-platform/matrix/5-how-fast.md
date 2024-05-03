---
title: How fast is our code ?
weight: 6

### FIXED, DO NOT MODIFY
layout: learningpathall
---

In this chapter, we will use google micro-benchmarking
[benchmark](https://github.com/google/benchmark) framework to measure the
performances of the core functions of our library.

Before one can optimize a piece of code, it's important to be able to assess
the current performances, and measure the delta improvements (or regressions)
of the complete library. It's just way to easy to make one specific function
faster, while regressing several others without noticing it right away ---
unless *some* form of testing is in place. On this aspect, micro-benchmarking
is to performance what unit-testing is to functional correctness.

## Setup Google benchmark

## Add benchmark

## Measure

## What have we achieved so far ?

CMake has enabled us to easily add performance testing, again in a platform
agnostic way --- for the platform supported by the
[benchmark](https://github.com/google/benchmark) framework.