---
layout: page
title: EdgePowerMeter
description: A High-Precision Power Profiling Tool for Edge AI Benchmarking
img: assets/img/projects/EdgePowerMeter/edgepowermeter.jpeg
importance: 2
category: tool
related_publications: false
---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/edgepowermeter_setup.jpg" title="EdgePowerMeter Setup" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The EdgePowerMeter hardware interface connected to an NVIDIA Jetson Nano for real-time power analysis.
</div>

## Abstract

**EdgePowerMeter** is a specialized hardware-software instrument designed for the precise measurement and analysis of power consumption in embedded AI systems. Addressing the need for granular energy profiling in edge computing, this tool combines a custom-designed ESP32-C3 based acquisition board with a high-throughput Python desktop application. It enables researchers to correlate power usage (Watts) with model inference performance (FPS), deriving critical efficiency metrics such as **Joules per Inference** and **FPS/Watt**, which are essential for optimizing neural networks on battery-constrained devices.

## System Architecture

The system operates on a master-slave architecture where the embedded probe acquires telemetry data and streams it to a host workstation for visualization and logging.

### Hardware Acquisition Layer
The sensing unit is built around the **INA226**, a high-precision current and power monitor with an I2C interface.
*   **Microcontroller**: **ESP32-C3** RISC-V MCU, chosen for its low power consumption and native USB-Serial capabilities.
*   **Sampling Rate**: Configured for high-frequency sampling to capture transient power spikes typical of GPU workload bursts.
*   **Communication**: Utilizes a high-speed UART link (921600 baud) to transmit voltage, current, and power readings with minimal latency.

### Software Analysis Suite
The host application is developed in **Python** using the **PySide6 (Qt)** framework, providing a responsive GUI for real-time data visualization.
*   **Real-Time Plotting**: Visualizes power consumption waveforms dynamically.
*   **Metric Integration**: Can ingest external performance logs (e.g., from a running AI model) to compute efficiency metrics in real-time.
*   **Data Export**: Supports logging to CSV and JSON formats for post-hoc analysis in tools like MATLAB or Pandas.

## Implementation Details

### Firmware (C++)
The firmware is optimized for throughput. It continuously polls the INA226 registers and formats the data into a lightweight binary or text protocol.
```cpp
// Snippet: High-speed serial transmission
void loop() {
    float busVoltage = ina.readBusVoltage();
    float current_mA = ina.readShuntCurrent();
    float power_mW = ina.readBusPower();
    
    Serial.printf("%.3f,%.3f,%.3f\n", busVoltage, current_mA, power_mW);
    // Minimal delay to maximize sampling rate
}
```

### Desktop Application (Python)
The desktop app uses multi-threading to decouple serial data acquisition from UI rendering, ensuring smooth performance even at high data rates.
```python
# Snippet: Data Acquisition Thread
class Worker(QThread):
    def run(self):
        while self.running:
            line = self.serial_port.readline().decode('utf-8')
            voltage, current, power = parse_data(line)
            self.data_signal.emit(voltage, current, power)
```

## Experimental Utility

This tool has been instrumental in benchmarking various edge accelerators, including the NVIDIA Jetson family and Raspberry Pi, allowing for:
*   **Thermal Throttling Analysis**: Observing power drops correlated with thermal events.
*   **Model Quantization Impact**: Quantifying the energy savings of INT8 vs FP16 inference.
*   **Idle vs Load Profiling**: Accurately measuring the baseline power consumption of carrier boards.

## Resources

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
    <div class="repo p-2 text-center">
        <a href="https://github.com/DanielRossi1/EdgePowerMeter" class="btn btn-primary z-depth-1">
            <i class="fab fa-github"></i> View Code
        </a>
    </div>
</div>
