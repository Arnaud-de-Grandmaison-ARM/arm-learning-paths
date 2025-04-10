---
title: Run the pipelines
weight: 6

### FIXED, DO NOT MODIFY
layout: learningpathall
---

In the previous section, we have build the AI Camera Pipelines.

## Background Blur

```BASH
cd $HOME/ai-camera-pipelines
bin/cinematic_mode ./resources/test_input2.ppm test_output2.ppm resources/depth_and_saliency_v3_2_assortedv2_w_augment_mobilenetv2_int8_only_ptq.tflite
```

![example image alt-text#center](test_input2.png "Figure 1: Original picture")
![example image alt-text#center](test_output2.png "Figure 2: Picture with blur applied")

## Low light enhancement

```BASH
cd $HOME/ai-camera-pipelines
bin/low_light_image_enhancement resources/test_input2.ppm test_output2_lime.ppm resources/HDRNetLIME_lr_coeffs_v1_1_0_mixed_low_light_perceptual_l2_loss_int8_only_ptq.tflite
```

![example image alt-text#center](test_output2_lime.png "Figure 3: Picture with low light enhancement applied")