---
layout: page
title: BrainForce
description: A Wireless Brain-Computer Interface (BCI) for Assistive Device Control
img: assets/img/projects/BrainForce/BrainForce.avif
importance: 8
category: educational
related_publications: false
---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/BrainForce/BrainForce.avif" title="BrainForce Headset" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The custom-built BrainForce EEG headset, featuring a dual-channel sensor array and wireless transmission module.
</div>

## Abstract

**BrainForce** is a Brain-Computer Interface (BCI) system developed to explore the potential of non-invasive neural control for assistive technology. The project focuses on the acquisition and processing of Electroencephalography (EEG) signals to translate cerebral activity into control commands for external devices. Specifically, the system detects the modulation of **Alpha waves (8-12 Hz)** in the visual cortex—associated with relaxation and eye closure—to trigger actuation in a remote-controlled vehicle. This proof-of-concept demonstrates a low-cost, accessible pathway for developing neural prosthetics for individuals with severe motor disabilities.

## Methodology

The system operates on a closed-loop architecture comprising signal acquisition, spectral analysis, and wireless actuation.

### Signal Acquisition
*   **Sensors**: Utilizes the **ADS1292R**, a medical-grade analog front-end (AFE) for biopotential measurements, configured for 2-channel EEG acquisition.
*   **Placement**: Electrodes are positioned over the occipital lobe (visual cortex) to maximize the detection of Alpha rhythms.
*   **Hardware**: A custom-fabricated headset houses the sensors, an **Arduino UNO** for local processing, and a Bluetooth module for telemetry.

### Signal Processing Pipeline
The core logic relies on Frequency Domain analysis to identify brain states.
1.  **Sampling**: Raw EEG data is sampled at 250 Hz.
2.  **Windowing**: A Hamming window is applied to the time-series buffer to reduce spectral leakage.
3.  **Fast Fourier Transform (FFT)**: The **arduinoFFT** library computes the power spectrum of the signal.
4.  **Feature Extraction**: The system calculates the Power Spectral Density (PSD) specifically within the Alpha band (8-12 Hz).
5.  **Thresholding**: A dynamic moving average threshold determines if the user is in a "Relaxed" (High Alpha) or "Active" (Low Alpha) state.

## Implementation Details

### Hardware Construction
The headset was engineered from recycled materials to demonstrate cost-effectiveness.
*   **Chassis**: Thermo-molded plastic from repurposed computer cases.
*   **Shielding**: Coaxial cabling used for all electrode connections to minimize 50Hz mains hum and electromagnetic interference.
*   **Power**: Independent 9V battery supply to isolate the user from mains voltage for safety.

### Control Logic
```cpp
// Snippet: Alpha Wave Detection Logic
void loop() {
    // Perform FFT on the sample buffer
    FFT.Windowing(vReal, SAMPLES, FFT_WIN_TYP_HAMMING, FFT_FORWARD);
    FFT.Compute(vReal, vImag, SAMPLES, FFT_FORWARD);
    FFT.ComplexToMagnitude(vReal, vImag, SAMPLES);

    // Extract power in Alpha band (8-12 Hz)
    double alphaPower = 0;
    for (int i = 8; i <= 12; i++) {
        alphaPower += vReal[i];
    }

    // Trigger actuation if threshold exceeded
    if (alphaPower > THRESHOLD) {
        sendBluetoothCommand(MOVE_FORWARD);
    } else {
        sendBluetoothCommand(STOP);
    }
}
```

## Key Features

*   **Non-Invasive**: Requires no surgical intervention, using dry or wet surface electrodes.
*   **Wireless**: Bluetooth connectivity allows for tether-free operation, essential for practical usability.
*   **Real-Time Processing**: Optimized FFT algorithms enable instantaneous feedback and control.
*   **Standalone**: The headset performs all signal processing on-board, removing the need for a tethered PC.

## Resources

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
    <div class="repo p-2 text-center">
        <a href="https://www.hackster.io/onedead_match/brainforce-fd2aab" class="btn btn-primary z-depth-1">
            <i class="fas fa-project-diagram"></i> View Project
        </a>
    </div>
</div>
