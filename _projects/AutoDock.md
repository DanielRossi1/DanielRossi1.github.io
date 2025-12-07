---
layout: page
title: AutoDock
description: A Docker-based Framework for Accelerating AI Research on Heterogeneous Embedded Systems
img: assets/img/projects/AutoDock/autodock.jpeg
importance: 1
category: tools
related_publications: false
---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/AutoDock/autodock.jpeg" title="AutoDock Workflow" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The AutoDock development workflow, enabling seamless transition from workstation to embedded edge devices.
</div>

## Abstract

**AutoDock** is a comprehensive Docker-based framework designed to standardize and accelerate the development lifecycle of Artificial Intelligence applications across heterogeneous hardware platforms. By abstracting the complexities of environment configuration, driver management, and device interfacing, AutoDock ensures full reproducibility and portability of research code. The framework features an automated hardware detection engine that dynamically configures container runtimes for **NVIDIA Jetson** (Nano, Orin), **Raspberry Pi**, and standard x86 workstations, facilitating rapid deployment of deep learning models in edge computing scenarios.

## System Architecture

The framework is built upon a modular architecture that decouples the development environment from the underlying hardware infrastructure.

### Automated Device Detection

AutoDock incorporates a bash-based inference engine (`libraries/devices.sh`) that inspects the host's device tree and kernel modules to identify the hardware platform. Upon detection, it automatically:
*   **Configures the Runtime**: Selects the appropriate container runtime (e.g., `nvidia-container-runtime` for Jetson) to enable GPU acceleration.
*   **Mounts Peripherals**: Automatically maps hardware interfaces such as **GPIO**, **I2C**, **SPI**, and **USB** cameras into the container, granting isolated applications direct access to physical sensors.
*   **Optimizes Resources**: Adjusts shared memory segments (`--shm-size`) and process privileges based on the detected capabilities.

### Remote Development Integration

To support modern research workflows, AutoDock integrates seamlessly with **VSCode Remote-Containers**. It provides a `remote.sh` utility that automates the generation of SSH keys and Docker contexts, allowing researchers to develop, debug, and profile code running on remote embedded devices directly from their local IDE, preserving the convenience of a desktop environment while executing on edge hardware.

## Implementation Details

The core of AutoDock is a set of optimized Dockerfiles and shell scripts that handle the dependency graph for AI research.

*   **Base Images**: Utilizes `nvidia/cuda` and `ubuntu` base images, layered with essential libraries for computer vision (OpenCV, FFmpeg) and deep learning (PyTorch, TensorFlow).
*   **Cross-Platform Support**: Maintains specific Dockerfiles for different architectures (`Dockerfile.jetsonOrin`, `Dockerfile.raspberrypi`), ensuring that platform-specific libraries (like `jetpack` or `rpi.gpio`) are correctly installed.
*   **User-Space Mapping**: Automatically maps the host user's UID/GID into the container, preventing file permission issues common in shared research environments.

## Key Features

*   **Hardware Abstraction**: Write code once, deploy on any supported embedded device without modification.
*   **Zero-Config Setup**: Eliminates the need for manual installation of CUDA, cuDNN, and Python environments.
*   **Peripheral Passthrough**: Native access to hardware accelerators and sensors within the isolated container.
*   **Reproducibility**: Guarantees that experiments can be replicated exactly on different machines, addressing a major challenge in scientific research.

## Resources

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
    <div class="repo p-2 text-center">
        <a href="https://github.com/DanielRossi1/AutoDock" class="btn btn-primary z-depth-1">
            <i class="fab fa-github"></i> View Code
        </a>
    </div>
    <div class="repo p-2 text-center">
        <a href="https://danielrossi1.github.io/AutoDock/" class="btn btn-primary z-depth-1">
            <i class="fas fa-book"></i> Documentation
        </a>
    </div>
</div>
