---
layout: page
title: TakuNet
description: An Energy-Efficient CNN for Real-Time Inference on Embedded UAV Systems
img: assets/img/projects/TakuNet/TakuNet_Architecture.png
importance: 1
category: research
related_publications: rossi2025takunet
---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/TakuNet/TakuNet_Architecture.png" title="TakuNet Architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The TakuNet macro-architecture featuring the Stem, TakuBlock, and DownSampler modules.
</div>

## Abstract

**TakuNet** is a lightweight Convolutional Neural Network (CNN) specifically designed for real-time inference on embedded Unmanned Aerial Vehicle (UAV) systems, particularly in emergency response scenarios {% cite rossi2025takunet %}. The architecture balances high accuracy with low computational complexity, enabling efficient deployment on edge devices with limited power and processing capabilities. Published at the **2025 IEEE/CVF Winter Conference on Applications of Computer Vision Workshops (WACVW)**, TakuNet addresses the critical need for energy-efficient computer vision in autonomous search and rescue operations.

## Methodology

The TakuNet architecture follows a modular design optimized for hardware acceleration. It is constructed using a 4-stage macro-architecture that progressively increases channel width while reducing spatial resolution.

### Architecture Design

The network is composed of the following key components:

*   **Stem Block**: The initial processing unit responsible for rapid spatial downsampling and preliminary feature extraction.
*   **TakuBlock**: The core building block of the network. It utilizes **depth-wise separable convolutions** to drastically reduce the number of parameters and floating-point operations (FLOPs) compared to standard convolutions. Each block incorporates **skip connections** (residual learning) to facilitate gradient flow and prevent degradation during training.
*   **DownSampler**: Specialized modules placed between stages to reduce feature map resolution efficiently.

### Model Configuration

The specific configuration of TakuNet used in the publication is defined by the following hyperparameters:
*   **Depths**: `[5, 5, 5, 4]` (Number of blocks in each of the 4 stages)
*   **Widths**: `[40, 80, 160, 240]` (Number of channels in each stage)

This configuration was empirically selected to maximize the accuracy-latency trade-off on embedded accelerators.

## Installation and Usage

The official implementation leverages **Docker** to ensure reproducibility across different hardware platforms.

### Setup

```bash
# Clone the repository
git clone https://github.com/DanielRossi1/TakuNet.git

# Build the Docker container
cd TakuNet/docker
./build.sh

# Run the container
./run.sh
```

### Training and Inference

The framework provides a unified interface for training and inference through the `launch.sh` script:

```bash
cd src
# Launch training with default configuration
./launch.sh TakuNet
```

For deployment on edge devices (e.g., NVIDIA Jetson Orin Nano), the model supports export to **TensorRT** engines for maximum performance.

## Implementation Details

The project is implemented using **PyTorch Lightning**, ensuring code modularity, reproducibility, and scalable training.

*   **Training Strategy**: The model is trained using the **RMSprop** optimizer coupled with a **StepLR** learning rate scheduler to ensure stable convergence.
*   **Deployment Pipeline**: To achieve real-time performance on edge devices, the training pipeline includes export utilities for optimized inference engines:
    *   **NVIDIA Jetson Orin Nano**: The model is converted to **TensorRT** engines, utilizing **FP16** precision to leverage the Tensor Cores on the Orin architecture.
    *   **Raspberry Pi**: For CPU-bound edge devices, the model is exported to **ONNX** format and executed via **ONNX Runtime**.

## Experimental Results

TakuNet was evaluated on the **AIDER** (Aerial Image Dataset for Emergency Response) and **AIDER-V2** datasets.

*   **Inference Speed**: On the NVIDIA Jetson Orin Nano, TakuNet achieves an inference throughput of over **650 FPS** (Frames Per Second) in FP16 mode.
*   **Efficiency**: The model demonstrates a superior balance of accuracy and computational cost compared to general-purpose lightweight backbones like MobileNetV3 and EfficientNet-Lite when applied to the specific domain of aerial emergency imagery.

## Resources

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
    <div class="repo p-2 text-center">
        <a href="https://github.com/DanielRossi1/TakuNet" class="btn btn-primary z-depth-1">
            <i class="fab fa-github"></i> View Code
        </a>
    </div>
    <div class="repo p-2 text-center">
        <a href="https://ieeexplore.ieee.org/document/10972615" class="btn btn-primary z-depth-1">
            <i class="fas fa-file-pdf"></i> Read Paper
        </a>
    </div>
</div>
