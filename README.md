# Embedded Predictive Maintenance System (STM32 + Edge Analytics)

This project implements vibration-based anomaly detection for rotating machinery 
directly on a microcontroller. The device captures real-time accelerometer data, 
extracts physics-based signal features, and classifies machine health states 
without cloud or external compute resources.

The system is designed for industrial monitoring where network coverage, latency, 
and power constraints make cloud solutions impractical.

---

## System Overview

Sensor (Accelerometer / Vibration Input)  
→ ADC / DMA circular buffer  
→ Sliding windows  
→ Feature extraction (RMS, variance, peak)  
→ Anomaly classifier  
→ Status output (Normal / Warning / Fault)

The pipeline minimizes CPU load and maximizes uptime for battery-powered deployments.

---

## Hardware

- MCU: STM32 L4 series (Cortex-M4F DSP support)
- ADC input: vibration sensor / MEMS accelerometer
- Sampling: continuous acquisition via DMA
- Output: UART logs or GPIO trigger

Any MCU with DMA-capable ADC/peripheral can be used.

---

## Firmware Pipeline

### 1. Data Acquisition via DMA
Vibration input is sampled continuously at a fixed rate (1–5 kHz typical).
Samples are streamed into a DMA circular buffer.

The MCU wakes only on:
- half-buffer interrupt
- full-buffer interrupt

Benefits:
- deterministic timing
- no data loss
- reduced CPU utilization

---

### 2. Windowing
Samples are processed in fixed-length windows.

Example configuration:
- Window length: 256–512 samples
- Overlap: 20–50%

Smaller windows → faster reaction  
Larger windows → more stability and frequency information

---

### 3. Feature Extraction (Time-Domain)
Each window produces a small feature vector:

- RMS (energy)
- Variance (vibration intensity)
- Peak-to-peak (fault events)

These metrics map directly to physical behavior:

- Normal bearings → low RMS, stable variance
- Wear progression → mild variance increase
- Fault → spikes + energy bursts

---

### 4. Edge Classification
Instead of large ML models, the system applies a simple threshold + rule-based classifier.

- Lightweight
- Explainable
- No runtime allocation
- Robust to sensor drift

This design is preferred in industrial embedded systems.

---

## Performance Metrics

- RAM usage: ~1–3 KB
- Flash usage: < 25 KB
- Latency: low ms-level
- Sampling: deterministic with DMA
- MCU power: sleep between DMA interrupts

This enables 24/7 condition monitoring.

---

## Engineering Rationale

### Why DMA?
Continuous sampling without ISR overload.
Predictable data pipeline under high sampling rate.

### Why time-domain features?
FFT or neural networks add complexity but do not improve reliability for simple fault detection.

Time-domain features:
- scale linearly
- require minimal RAM
- map to physics
- run in constant time

### Why rule-based vs ML?
Industrial environments prioritize:
- deterministic behavior
- debug simplicity
- safety compliance
- long-term stability

False positives matter more than academic accuracy.

---

## Deployment Use Cases

- Motor bearing monitoring  
- CNC spindle vibration tracking  
- Gearbox condition assessment  
- Pump/rotor health monitoring  
- Factory floor IoT sensors

The design fits:

- Industrial IoT
- Smart manufacturing
- Remote installations
- Battery-powered nodes

---

## Notes on Real Deployment

- Sensor mounting position affects signal quality
- Ambient noise must be profiled
- High sampling requires stable clock
- Device should handle vibration bursts gracefully
- Logging must be rate-limited to avoid UART bottlenecks

---

## Disclaimer
This repository documents the design, architecture, and engineering decisions made for an embedded predictive maintenance device.  
Proprietary firmware and client-specific code are not included.

---


