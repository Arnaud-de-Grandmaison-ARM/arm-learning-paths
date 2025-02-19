---
title: KleidiAI
weight: 7

### FIXED, DO NOT MODIFY
layout: learningpathall
---


The LLM part of the Voice Assistant, is using
[Llama.cpp](https://github.com/ggml-org/llama.cpp). LLM inference is a very
computation intensive task and has thus been heavily optimized within Llama.cpp
for various platforms --- including Arm.

Speech recognition is also a part that is computation intensive, and it has also
been optimized for Arm processors.

## KleidiAI

This application uses [KleidiAI library](https://gitlab.arm.com/kleidi/kleidiai)
by default for optimized performance on Arm processors.

[KleidiAI](https://gitlab.arm.com/kleidi/kleidiai) is an open-source library
that provides optimized performance-critical routines, also known as
micro-kernels, for artificial intelligence (AI) workloads tailored for Arm®
CPUs.

These routines are tuned to exploit the capabilities of specific Arm hardware
architectures, aiming to maximize performance.

The KleidiAI library has been designed for ease of adoption into C or C++
machine learning (ML) and AI frameworks. Specifically, developers looking to
incorporate specific micro-kernels into their projects can only include the
corresponding ``.c`` and ``.h`` files associated with those micro-kernels and a
common header file.

### Compare the performance without KleidiAI

By default, the Voice Assistant is built with KleidiAI support on Arm platforms,
but this can be disabled should you want to compare the performances to a raw
implementation.

You can disable the KleidiAI support at build times in Android Studio by adding
the ``-PkleidiAI=false`` to the graddle invocation. You can also edit the top
level ``gradle.properties`` file and add ``kleidiAI=false`` at the end of it.

### Why use KleidiAI ?

A nice benefit of using KleidiAI is that it enables the developer to work at a
somehow high-level, leaving the KleidiAI library to select the best
implementation at runtime to perform the computation in the most effective way
on the current target. This in itself is a great win, because a significant
amount of work went into optimizing those micro-kernels, but where it becomes
extremely powerful is when newer version of the architecture become available: a
simple update of the KleidiAI used by the Voice Assistant will give it
automatically access to newer hardware features when they become available. An
example of such a feature deployment is happening with SME2, which means in some
near future the Voice Assitant will be able to benefit from improved
performances (on the devices which have implemented SME2) with no further effort
required.
