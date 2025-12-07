---
layout: page
title: MultiVision
description: A Multi-Stage Perception System for Autonomous Driving Environments
img: assets/img/projects/MultiVisionSystem/MultiVisionSystem.jpg
importance: 5
category: Master's
pdf: assets/pdf/MultiVisionSystem.pdf
---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/MultiVisionSystem/MultiVisionSystem.jpg" title="MultiVision Output" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Real-time perception output showing lane boundaries, detected vehicles, and recognized traffic signs.
</div>

## Abstract

**MultiVision** (also known as **ComputerVisionProject**) is a comprehensive Advanced Driver Assistance System (ADAS) developed for the **Computer Vision and Cognitive Systems** course at the **University of Modena and Reggio Emilia**. The project addresses the core challenges of autonomous driving perception: lane keeping, obstacle detection, and traffic rule compliance. By integrating classical computer vision algorithms with state-of-the-art deep learning models, the system provides a robust understanding of the vehicle's environment. The pipeline combines a geometric lane detection algorithm, a **YOLOv7**-based object detector, and a feature-matching traffic sign recognizer into a unified framework capable of real-time performance on standard automotive hardware.

## Methodology

The perception pipeline is divided into three parallel modules that fuse their outputs into a single environmental model. The system leverages both classical computer vision techniques for geometric tasks and deep learning for semantic understanding.

### 1. Classical Techniques

#### Camera Calibration
To ensure geometric accuracy, the system first corrects lens distortion, specifically the barrel distortion common in wide-angle automotive cameras. We employed **Zhang's technique**, acquiring dozens of images of a chessboard pattern to estimate the intrinsic camera parameters and unwarp the input frames.

#### Speed Limit Detection and Classification
This module identifies speed limit signs and classifies their value.
*   **ROI Selection**: To improve performance, the search area is restricted by cropping 5% from the top/bottom and 50% from the left, focusing on the right side of the road where signs are typically located.
*   **Detection**:
    *   **Color Segmentation**: The image is converted to **HSV** color space. Two masks are applied to isolate red hues (ranges [0, 10] and [160, 179]), which are then combined to segment the sign's border.
    *   **Background Removal**: A bitwise AND operation removes non-red background elements.
    *   **Thresholding**: The red channel is extracted and thresholded using **Otsu's method** (values [200, 255]), followed by morphological closing and dilation to reduce noise.
    *   **Localization**: The largest contour is identified as the candidate sign. The **Hough Circle Transform** is then used to refine the exact location and bounding box of the circular sign.
*   **Classification**:
    *   **Feature Extraction**: The candidate region is resized to 100x100 pixels and binarized. **SIFT (Scale-Invariant Feature Transform)** descriptors are extracted to capture local invariant features.
    *   **Matching**: These descriptors are matched against a reference database of speed limit signs using **FLANN-based KNN (k-Nearest Neighbors)**.
    *   **Validation**: A match is considered valid only if the best result is $\ge 125\%$ of the second-best result and exceeds a minimum threshold $k$. This method proved superior to Template Matching (sensitive to rotation) and ORB (prone to false positives on edges).

#### Adaptive Lane Assistant
The lane detection system is designed to be robust against noise (e.g., grass, cracks) and varying lighting conditions.
*   **Preprocessing**:
    *   **Grass Removal**: Green hues are filtered out using HSV masking to remove roadside vegetation noise.
    *   **Adaptive Smoothing**: The image is converted to grayscale, and the mean luminance is calculated. Based on this global luminance and the local luminance of the lower image section, the system selects one of **11 configuration presets**. These presets adjust the parameters of a **Bilateral Filter**, which smooths noise while preserving lane edges (unlike a Gaussian filter).
*   **Edge Detection**: An adaptive **Canny edge detector** is applied, with thresholds dynamically set based on the luminance analysis.
*   **Line Extraction**: A bitwise mask isolates the road surface. The **Probabilistic Hough Transform** is then applied to the left and right halves of the image separately to identify lane markings.
*   **Warning System**: The system monitors the angle ($\theta$) of the detected lines. If the vehicle drifts too close to a lane boundary, the visualization turns red and a warning is issued.

### 2. Deep Learning Techniques

#### YOLOv7 Object Detection
For detecting dynamic agents (vehicles, pedestrians, cyclists), the system deploys **YOLOv7**, a state-of-the-art real-time object detector.
*   **Dataset**: The model was fine-tuned on the **BDD100K** dataset, a large-scale diverse driving dataset. The team focused on 10 relevant classes (car, bus, truck, rider, etc.) and filtered out irrelevant ones.
*   **Training**: Fine-tuning was performed on images resized to 640x640, with extensive data augmentation (Mosaic, Mixup, color jitter, fog/rain simulation). The training ran for 50 epochs on a multi-GPU server (5x NVIDIA RTX 3090), taking approximately 4.5 hours.
*   **Performance**: The model achieves real-time inference speeds (>30 FPS) on standard hardware, balancing accuracy and latency.

### 3. Hardware & Future Developments
*   **Hardware**: Data acquisition was performed using an **e-con Systems See3CAM_CU27** (Sony Starvis sensor) mounted on a custom 3D-printed support.
*   **Tracking**: A **Kalman Filter** is implemented to track detected objects across frames, predicting their future trajectories to smooth detection jitter.
*   **Distance Estimation**: A monocular distance estimation algorithm approximates the distance to objects based on known reference sizes and focal length.

## Installation and Usage

The project provides a Docker-based environment for easy reproduction and deployment.

### Setup

```bash
# Clone the repository
git clone https://github.com/DanielRossi1/ComputerVisionProject.git
cd ComputerVisionProject

# Install dependencies (Conda recommended)
conda env create -f environment.yml
conda activate cvp
```

### Running the Pipeline

The system can be executed on images, videos, or live camera feeds.

**Inference on Video:**
```bash
python main.py -t -tv -sv -td '/path/to/video.mp4' -w 'weights/yolov7.pt' -fl 'output_video.avi'
```

**Real-Time Camera Inference:**
```bash
python camera.py -sc 'live' -w 'weights/yolov7.pt' --verbose --track --confidence 0.85
```

### Key Arguments
*   `-t` / `--track`: Enable object tracking.
*   `-ln` / `--lane-assistant`: Enable lane detection.
*   `-j` / `--jetson`: Optimize for NVIDIA Jetson Nano (disables some heavy modules).
*   `-v` / `--verbose`: Visualize bounding boxes and lane lines on the output frame.

## Resources

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
    <div class="repo p-2 text-center">
        <a href="https://github.com/DanielRossi1/ComputerVisionProject" class="btn btn-primary z-depth-1">
            <i class="fab fa-github"></i> View Code
        </a>
    </div>
    <div class="repo p-2 text-center">
        <a href="{{ site.baseurl }}/assets/pdf/MultiVisionSystem.pdf" class="btn btn-primary z-depth-1">
            <i class="fas fa-file-pdf"></i> Read Report
        </a>
    </div>
</div>
