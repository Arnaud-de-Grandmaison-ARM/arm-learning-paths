---
title: Deploy !
weight: 11

### FIXED, DO NOT MODIFY
layout: learningpathall
---

In this chapter, we will look into how to best deploy our project,
that is how to make it easy for our users to use the `Matrix` library
on their platform, and how to empower our users to contribute to the
library by publishing the project on github for example.

## Add deployment helpers with CMake

We have heavily benefited from how easy it was to use
[GoogleTest](https://github.com/google/googletest) and
[benchmark](https://github.com/google/benchmark) as dependencies,
so let's do the same favor to our users.

## Publish on github

Deploying the project on a public infrastructure is an important step for
others to be able to start reusing it, but also contributing to it.

On top of making the repository public, github actions can be setup to
perform regression testing on several platforms.

## What have we achieved so far ?
