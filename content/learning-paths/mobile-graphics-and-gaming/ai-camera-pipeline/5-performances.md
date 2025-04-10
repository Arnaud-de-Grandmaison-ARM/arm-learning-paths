---
title: Performances
weight: 7

### FIXED, DO NOT MODIFY
layout: learningpathall
---

The application that have been formerly built have a *benchmark* mode that will
run the core function multiple times in a hot loop:

- ``ai-camera-pipelines/bin/cinematic_mode_benchmark``
- ``ai-camera-pipelines/bin/low_light_image_enhancement_benchmark``

KleidiCV improves the performances of the camera pipelines by providing OpenCV
with computation kernels optimized for the Arm processors.

## Performances with KleidiCV

 By default, the openCV library is built with KleidiCV support, so let's measure
 the performances of the applications that we have already built:

 ```BASH
$ bin/cinematic_mode_benchmark 20 resources/depth_and_saliency_v3_2_assortedv2_w_augment_mobilenetv2_int8_only_ptq.tflite
INFO: Created TensorFlow Lite XNNPACK delegate for CPU.
Total run time over 20 iterations: 1985.99 ms

$ bin/low_light_image_enhancement_benchmark 20 resources/HDRNetLIME_lr_coeffs_v1_1_0_mixed_low_light_perceptual_l2_loss_int8_only_ptq.tflite
INFO: Created TensorFlow Lite XNNPACK delegate for CPU.
Total run time over 20 iterations: 52.3448 ms
```

It can be seen from above that:
- ``cinematic_mode_benchmark`` performed 20 iterations in 1985.99 ms,
- ``low_light_image_enhancement_benchmark`` performed 20 iterations in 52.3448 ms.

## Performances without KleidiCV

The support for KlediCV in OpenCV can be optionnaly turned off by editing the
``WITH_KLEIDICV`` parameter in ``external/opencv/CMakeLists.txt``.

```TXT
set(WITH_KLEIDICV "OFF" CACHE INTERNAL "")
```

Edit the ``external/opencv/CMakeLists.txt`` as indicated to turn off the the
KleidiCV support and rebuild the applications as shown in the build section.

Now run again the benchmarks:

```BASH
$ bin/cinematic_mode_benchmark 20 resources/depth_and_saliency_v3_2_assortedv2_w_augment_mobilenetv2_int8_only_ptq.tflite
INFO: Created TensorFlow Lite XNNPACK delegate for CPU.
Total run time over 20 iterations: 2027.11 ms

$ bin/low_light_image_enhancement_benchmark 20 resources/HDRNetLIME_lr_coeffs_v1_1_0_mixed_low_light_perceptual_l2_loss_int8_only_ptq.tflite
INFO: Created TensorFlow Lite XNNPACK delegate for CPU.
Total run time over 20 iterations: 75.8808 ms
```

Let's put together all those number in a simple table in order to compare them easily:

| Benchmark                                 | Without KleidiCV | With KleidiCV |
|-------------------------------------------|------------------|---------------|
| ``cinematic_mode_benchmark``              | 2027.11 ms       | 1985.99 ms    |
| ``low_light_image_enhancement_benchmark`` | 75.8808 ms       | 52.3448 ms    |

As can be seen, the blur pipeline (``cinematic_mode_benchmark``) does benefit
marginally from KleidiCV whereas low light enhancement got almost a 30% boost.

## Future performances uplift with SME2

A nice benefit of using KleidiCV (or KleidiAI) is that whenever the hardware
will add support for new and more powerful instructions, the applications will
be able to get a performance uplift without requiring complex software changes
--- KleidiCV and KleidiAI operate as abstraction layers that will be able to
build on hardware improvements to boost future performances. An example of such
a performance boost *for free* will take place in a couple months from now when
processors implementing SME2 will become available.