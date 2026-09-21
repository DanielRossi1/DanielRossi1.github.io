---
layout: page
title: TakuNetV2
description: Energy-Efficient Models for Real-Time Aerial Disaster Response and Monitoring on Edge Devices
img: assets/img/projects/TakuNetV2/TakuNetV2.png
importance: 1
category: research
related_publications: rossi2026takunet
---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/TakuNetV2/TakuNetV2.png" title="TakuNetV2 Architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The TakuNetV2 macro-architecture: a dense two-convolution stem, four stages of TakuBlockV2 each closed by a Downsampler, and a global average pooling head.
</div>

## Abstract

**TakuNetV2** is the journal extension of [TakuNet]({{ '/projects/TakuNet/' | relative_url }}) (WACVW 2025), published in **Image and Vision Computing** {% cite rossi2026takunet %}. It presents TakuNet as a _family_ of ultra-lightweight convolutional networks for real-time aerial image classification on resource-constrained embedded devices, in the context of disaster response and wide-area monitoring from UAVs. The target regime is deliberately extreme: **under 100K parameters**, deployable on a Raspberry Pi or a UAV-grade NPU, and fast enough to process the video stream on board.

The work was carried out in collaboration with **EBV Elektronik** and **System Electronics**, and includes an evaluation on the Astrial platform, a Hailo-8 based single-board computer designed for UAV deployment.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/TakuNetV2/overview.png" title="TakuNetV2 Overview" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    On-board aerial classification for disaster response: the video stream is classified on the UAV rather than offloaded.
</div>

## From V1 to V2

TakuNetV1 is built almost entirely on depth-wise convolutions. That keeps it tiny, but depth-wise kernels process each channel independently, so **no cross-channel mixing happens inside the blocks at all**: the network's entire ability to combine channels is delegated to the Downsampler closing each stage. This becomes the dominant error source as the number of semantic classes grows, and it is exactly what V2 attacks.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/TakuNetV2/TakuNetV1.png" title="TakuNetV1" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    TakuNetV1, the WACVW 2025 baseline, ~37K parameters, built almost entirely on depth-wise convolutions. Compare it with the V2 architecture at the top of the page: V2 restores cross-channel mixing inside the blocks for ~8K additional parameters.
</div>

Both networks share the same skeleton (a stem, four processing stages each closed by a Downsampler, and a global average pooling head) and differ in what happens inside:

|                          | TakuNetV1                    | TakuNetV2                                                                                                           |
| ------------------------ | ---------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **Stem**                 | full conv → depth-wise conv  | two cascaded full convs, `3 → C/2 → C`                                                                              |
| **Block**                | one residual 3×3 depth-wise  | grouped 1×1 compression + two parallel depth-wise branches (`d=1`, `d=2`) fused by sum, concatenated back, residual |
| **Downsampler grouping** | `⌊(Cin+Cdense)/4⌋` groups    | `⌊(Cin+Cdense)/8⌋` groups                                                                                           |
| **Normalization**        | GRN after every Downsampler  | GRN removed (square roots and norms are slow)                                                                       |
| **Last-stage pooling**   | fixed-kernel average pooling | `AdaptiveAvgPool2d` (one architecture for 224/240/256 inputs)                                                       |
| **Activation**           | ReLU6                        | LeakyReLU                                                                                                           |
| **Refiner**              | present                      | removed, absorbed into the stages                                                                                   |
| **Stage depths**         | `[5, 5, 5, 4]`               | `[3, 3, 5, 4]`                                                                                                      |

Three design points carry the accuracy gain:

- **A dense stem buys channel context, not capacity.** The grouped 1×1 opening each TakuBlockV2 mixes only within its own group. If the preceding stem operator were depth-wise, each compressed channel could only ever see a fixed subset of stem channels. With a _full_ second stem convolution, every stem output already combines all intermediate channels. Dense convolutions also map far better onto NPUs and GPUs than fragmented depth-wise ops.
- **Compression _before_ spatial processing.** The grouped 1×1 halves the channels on which the depth-wise convolutions run, inverting the inverted bottleneck of MobileNetV2/V3, which expands first and pays for it in intermediate memory and bandwidth.
- **Two spatial scales at once, for free.** The dilated branch (`d=2`) widens the receptive field to an effective 5×5 with _identical_ parameter count. Fusing the two branches by summation rather than concatenation means each channel carries both responses without doubling the channel count. That matters in aerial imagery, where the structural edges of a collapsed building and the spatial extent of a flood must be recognised at the same time.

## Results

Weighted F1, parameters, memory footprint and FLOPs, restricted to the **ultra-lightweight regime (< 100K parameters)**. The comparison against MobileNets, EfficientNet, ShuffleNet, ViT variants, ResNet-50 and VGG16 is in the paper.

**AIDER**, 5 classes, 240×240

| Model            |        F1 | Params | Memory (MB) |  FLOPs |
| ---------------- | --------: | -----: | ----------: | -----: |
| EmergencyNet     |     0.936 | 90,963 |        0.36 | 77.34M |
| TinyEmergencyNet |     0.895 | 39,334 |        0.16 | 36.30M |
| TakuNetV1 (fp16) | **0.943** | 37,685 |        0.15 | 35.93M |
| TakuNetV2        |     0.936 | 45,945 |        0.18 | 51.80M |

**AIDERv2**, 4 classes, 224×224

| Model            |        F1 | Params | Memory (MB) |  FLOPs |
| ---------------- | --------: | -----: | ----------: | -----: |
| EmergencyNet     |     0.955 | 90,704 |        0.36 | 61.96M |
| TinyEmergencyNet |     0.882 | 39,075 |        0.16 | 31.57M |
| TakuNetV1 (fp32) |     0.957 | 37,444 |        0.15 | 31.38M |
| TakuNetV2        | **0.959** | 45,704 |        0.18 | 45.23M |

**CLRS**, 25 classes, 256×256

| Model            |        F1 | Params | Memory (MB) |  FLOPs |
| ---------------- | --------: | -----: | ----------: | -----: |
| EmergencyNet     |     0.815 | 96,143 |        0.38 | 82.31M |
| TinyEmergencyNet |     0.710 | 44,514 |        0.18 | 42.63M |
| TakuNetV1 (fp16) |     0.805 | 42,505 |        0.17 | 40.95M |
| TakuNetV2        | **0.838** | 50,765 |        0.20 | 59.07M |

The gap widens with the number of classes, which is exactly the prediction made by the cross-channel-mixing argument: on 25-class CLRS, V2 gains more than three F1 points over V1.

## Embedded Performance

All measurements at batch size 1, averaged over 2,500 runs. `Power` is total board consumption during inference. `δ-Power` is the power drawn _above_ the platform's idle consumption, so `FPS/δ-Power` isolates the energy cost of the model from that of the board.

| Platform          | Model     |    FPS | Power (W) | FPS/δ-Power |
| ----------------- | --------- | -----: | --------: | ----------: |
| Raspberry Pi 5    | TakuNetV1 |  141.0 |      8.16 |       28.50 |
| Raspberry Pi 5    | TakuNetV2 |  101.5 |      8.00 |       21.20 |
| Jetson Orin Nano  | TakuNetV1 |  655.3 |      10.0 |      119.15 |
| Jetson Orin Nano  | TakuNetV2 |  560.3 |      10.0 |      101.87 |
| Astrial (Hailo-8) | TakuNetV1 | 1116.0 |      4.84 |     1312.94 |
| Astrial (Hailo-8) | TakuNetV2 |  358.0 |      4.46 |      761.70 |

**Reading the numbers.** TakuNetV1 dominates where operation-count minimality is what the hardware rewards. V2 trades some of that for accuracy and behaves best on hardware with high arithmetic intensity, still running at 560 FPS on the Orin Nano while being the most accurate model of the group on two of the three datasets.

The Hailo-8 row is the exception and is worth an explanation: the Dataflow Compiler splits TakuNetV2 into a **two-context execution plan** while V1 compiles to a single context, and every context switch costs a PCIe round trip. This is not about model size, given that the chip runs ResNet-50 in one context, but about topology: the parallel dual-branch depth-wise paths and the multi-input concatenation of TakuBlockV2 produce a dataflow graph the current compiler cannot route within a single on-chip allocation. It is a toolchain limitation, it does not occur on any other platform tested, and 358 FPS still exceeds any real-time requirement for aerial classification.

A side measurement drove one design choice, the latency of common activations on a Raspberry Pi 4 over a `(10000, 100)` tensor:

|   ReLU |  ReLU6 |  LeakyReLU |  PReLU |     ELU |    GELU |    SELU |
| -----: | -----: | ---------: | -----: | ------: | ------: | ------: |
| 30.08s | 30.84s | **30.87s** | 32.87s | 168.56s | 401.49s | 169.00s |

## The Astrial Platform

<div class="row justify-content-center">
    <div class="col-sm-6 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/TakuNetV2/astrial.png" title="Astrial board" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The Astrial single-board computer used for the NPU evaluation.
</div>

[Astrial](https://www.systemelectronics.ai/en/astrial-IMX8MP-H8) is an AI-driven single-board computer developed by System Electronics for embedded and UAV applications. It pairs an NXP i.MX8M Plus with a **Hailo-8 NPU** rated at 26 TOPS (int8) and 32 MB of on-chip memory, adds an NXP SE050 Secure Element, and is qualified for the −40 °C to 85 °C range with a custom 50×40×20 mm aluminium heatsink weighing 64 g. Mass, volume and thermal footprint matter directly on a UAV, where an asymmetric heatsink alters the moment of inertia and exposed cooling surfaces add drag.

## Datasets and Reproducibility

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/TakuNetV2/AIDER.jpg" title="AIDER samples" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Sample images from the AIDER dataset (fire, flood, collapsed building, traffic incident, normal).
</div>

The three benchmarks are AIDER ([Zenodo 3888300](https://zenodo.org/records/3888300)), AIDERv2 ([Zenodo 10891054](https://zenodo.org/records/10891054)) and CLRS ([HuggingFace](https://huggingface.co/datasets/jonathan-roberts1/CLRS)). The release includes training, evaluation, ONNX export, embedded inference scripts and the trained checkpoints.

The reference environment is Python 3.11, PyTorch 2.2.2 + CUDA 12.1 and PyTorch Lightning 2.2.5. The repository ships with [AutoDock]({{ '/projects/AutoDock/' | relative_url }}), which detects the host device and builds the matching image (desktop with CUDA/TensorRT, Raspberry Pi, Jetson Nano, Jetson Orin):

```bash
git clone https://github.com/DanielRossi1/TakuNetV2.git
cd TakuNetV2/AutoDock
./build.sh
./run.sh -d /path/to/data
```

## Resources

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
    <div class="repo p-2 text-center">
        <a href="https://github.com/DanielRossi1/TakuNetV2" class="btn btn-primary z-depth-1">
            <i class="fab fa-github"></i> View Code
        </a>
    </div>
    <div class="repo p-2 text-center">
        <a href="https://codeberg.org/danielrossi/TakuNetV2" class="btn btn-primary z-depth-1">
            <i class="fas fa-code-branch"></i> Codeberg Mirror
        </a>
    </div>
    <div class="repo p-2 text-center">
        <a href="https://doi.org/10.1016/j.imavis.2026.106151" class="btn btn-primary z-depth-1">
            <i class="fas fa-file-pdf"></i> Read Paper
        </a>
    </div>
</div>
