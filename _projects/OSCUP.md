---
layout: page
title: OSCUP
description: Open Source Custom UART Protocol for Reliable Embedded Communication
img: assets/img/projects/OSCUP/OSCUP.jpg
importance: 6
category: Master's
related_publications: false
---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/OSCUP/OSCUP.jpg" title="OSCUP Packet Structure" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The OSCUP packet frame structure designed for minimal overhead and maximum data integrity.
</div>

## Abstract

**OSCUP (Open Source Custom UART Protocol)** is a lightweight, binary communication protocol designed specifically for resource-constrained embedded systems. It addresses the common need for a reliable, standardized method of data exchange between microcontrollers (e.g., Arduino, ESP32, STM32) and host computers or other nodes over asynchronous serial interfaces (UART). Unlike raw ASCII streams, OSCUP provides framing, error detection, and command multiplexing, ensuring data integrity in noisy electrical environments.

## Protocol Specification

The protocol is defined by a strict packet structure that ensures synchronization and validity.

### Packet Structure
Each frame consists of the following fields:

| Field | Size (Bytes) | Description |
| :--- | :---: | :--- |
| **START** | 1 | Start-of-Frame delimiter (e.g., `0xAA`) |
| **LEN** | 1 | Length of the payload |
| **CMD** | 1 | Command ID for multiplexing different data types |
| **PAYLOAD** | N | Variable length data (0-255 bytes) |
| **CRC** | 1 | Cyclic Redundancy Check (or Checksum) for error detection |
| **END** | 1 | End-of-Frame delimiter (e.g., `0xBB`) |

### Reliability Mechanisms
*   **Framing**: The unique START and END bytes allow the receiver to resynchronize with the data stream if a byte is dropped or corrupted.
*   **Error Detection**: A checksum (XOR or CRC8) is calculated over the payload. Packets with mismatched checksums are automatically discarded.
*   **Handshaking**: The protocol supports an optional ACK/NACK mechanism, where the receiver confirms successful receipt of critical messages, enabling automatic retransmission.

## Implementation Details

The reference implementation provides a C++ library for embedded devices and a Python library for host applications.

### C++ Implementation (Embedded)
Designed for efficiency, the C++ library uses a finite state machine (FSM) to parse incoming bytes without blocking the main loop.

```cpp
// Snippet: OSCUP State Machine
void parseByte(uint8_t byte) {
    switch (state) {
        case WAIT_START:
            if (byte == START_BYTE) state = WAIT_LEN;
            break;
        case WAIT_LEN:
            payloadLen = byte;
            state = WAIT_CMD;
            break;
        // ... (CMD, PAYLOAD, CRC states)
        case WAIT_END:
            if (byte == END_BYTE && calcCRC() == receivedCRC) {
                processPacket();
            }
            state = WAIT_START;
            break;
    }
}
```

## Key Features

*   **Low Overhead**: Minimal header and footer bytes maximize the effective data throughput.
*   **Platform Agnostic**: Written in standard C/C++ and Python, compatible with any architecture supporting UART.
*   **Extensible**: The 1-byte Command ID allows for up to 256 distinct message types, making it suitable for complex control systems.

## Resources

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
    <div class="repo p-2 text-center">
        <a href="https://github.com/DanielRossi1/OSCUP" class="btn btn-primary z-depth-1">
            <i class="fab fa-github"></i> View Code
        </a>
    </div>
</div>
