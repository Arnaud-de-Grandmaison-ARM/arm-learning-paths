---
title: Overview
weight: 4

### FIXED, DO NOT MODIFY
layout: learningpathall
---

The input and output images are stored in ``ppm`` (portable pixmap) format, with
3 channels (Red, Green and Blue) and 256 color levels each (a.k.a ``RGB8``).

The image are first converted to the ``YUV420`` color space, where the blur or
low light enhancement operations will take place. After the processing is done
the images are converted back to ``RGB8`` and saved in ``ppm`` format.

## Background Blur

blabla

## Low Light Enhancement

blabla