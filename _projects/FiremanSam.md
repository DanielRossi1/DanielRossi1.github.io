---
layout: page
title: FiremanSam
description: An IoT-based Early Fire Detection System using Edge Machine Learning
img: assets/img/projects/FiremanSam/FiremanSam.jpeg
importance: 3
category: tools
related_publications: false
---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/FiremanSam/FiremanSam.jpeg" title="FiremanSam Architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The distributed architecture of the FiremanSam system, featuring sensor nodes, a central gateway, and cloud integration.
</div>

## Abstract

**FiremanSam** is a distributed Internet of Things (IoT) system designed for the early detection and localization of fire hazards in industrial and residential environments. Moving beyond simple threshold-based alarms, this project implements **Edge Machine Learning** by deploying a Support Vector Machine (SVM) classifier directly on ESP32 microcontrollers. This allows for intelligent multi-sensor fusion (temperature, humidity, gas, and flame) to distinguish between actual fire events and false positives locally, reducing latency and bandwidth usage. The system utilizes a custom communication protocol (OSCUP) for robust node-to-gateway transmission and integrates with Google Firebase for real-time remote monitoring.

## System Architecture

The system is composed of three distinct devices: a Sensing device, an Actuator device, and the Gateway.

### Sensing Device (Edge Nodes)
Each sensor node is an autonomous unit powered by an **ESP32** microcontroller, equipped with:
*   **Environmental Sensors**: DHT11 (Temperature/Humidity), CCS811 (CO/ppm). The first prototype included a flame sensor, an MQ-7 Carbon Monoxide sensor and a temperature sensor.
*   **Edge Inference**: Runs a lightweight SVM model trained on a custom dataset to classify the current environmental state as "Normal" or "Fire".
*   **Data Transfer**: Immediate upload of classified labels on the Firebase database.

### Network Device (Gateway)
A central Gateway node aggregates data from multiple sensor nodes using **OSCUP (Open Source Custom UART Protocol)** over a wireless network. It acts as the bridge between the local sensor and actuators. It hosts a control web page where it allows to create relationships between sensors and actuator, thus routing specific sensors classified labels to their coupled actuator counterparts.

### Actuator Device
A general actuator connected to the same wifi network of the previous devices, receives data from the gateway and takes an action based on the classification status. In our implementation we ringed an alarm when the status is "fire", nothing otherwise.

### Application Layer (Cloud)
The Gateway captures validated classified data from the **Google Firebase Realtime Database**. Always through Firebase it changes the state of the actuator linked to the specific sensor's classified label.

## Implementation Details

### Edge Machine Learning
The core innovation is the deployment of an SVM classifier on the microcontroller. The model was written form scracth in numpy, implementing the pegasus version of SVM. It has been ported later to C++ for the ESP32.
*   **Feature Engineering**: Raw sensor data is pre-processed and normalized.
*   **Inference Engine**: A custom C++ implementation of the SVM decision function allows for millisecond-level classification times on the 240MHz dual-core ESP32.

```cpp
// Snippet: SVM Decision Function on ESP32
float predict(float* features) {
    float decision = bias;
    for (int i = 0; i < NUM_FEATURES; i++) {
        decision += weights[i] * features[i];
    }
    return (decision > 0) ? CLASS_FIRE : CLASS_SAFE;
}
```

### Communication Protocol (OSCUP)
To ensure data integrity between nodes and the gateway, the custom **OSCUP** protocol was developed. It features:
*   **Packet Structure**: Start/End bytes, Payload Length, Command ID, Payload, and Checksum.
*   **Reliability**: Implements ACK/NACK handshaking to guarantee message delivery.

## Key Features

*   **Multi-Sensor Fusion**: Reduces false alarms by correlating data from multiple physical phenomena.
*   **Edge Intelligence**: Decisions are made locally, ensuring functionality even if internet connectivity is lost.
*   **Scalability**: The star topology allows for easy addition of new sensor nodes.
*   **Real-Time Connectivity**: Instant synchronization with cloud services for remote alerting.

## Resources

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
    <div class="repo p-2 text-center">
        <a href="https://github.com/DanielRossi1/FiremanSam" class="btn btn-primary z-depth-1">
            <i class="fab fa-github"></i> View Code
        </a>
    </div>
</div>
