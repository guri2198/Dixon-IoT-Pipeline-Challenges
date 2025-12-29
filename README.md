# Dixon-IoT-Pipeline-Challenges

<img width="1100" height="605" alt="Screenshot from 2025-12-29 15-42-52" src="https://github.com/user-attachments/assets/76c3c965-d673-4ff9-bd17-f6073eb2ffd5" />

# Dixon IoT Pipeline: Challenges & Solutions

## Overview
This repository documents the key challenges encountered while building the Dixon IoT Lab's **Kafka-based RTSP camera pipeline**, and the production-grade solutions implemented to overcome them. 

It is meant as a **technical reference and case study** for engineers dealing with high-throughput camera streaming, HPC nodes, and GPU-accelerated pipelines.

---

## Challenges Encountered
- **Kafka broker / ISR instability**  
  - Single broker crashes, data transmission stops
- **FFmpeg backpressure / incomplete frames**  
  - Frame drops when CPU overloaded
- **Tyron machine OOM & process hangs**  
  - Night-time camera streams overloading GPU memory
- **Manual script management**  
  - 40+ camera scripts run individually → human error
- **Night-time low visibility**  
  - Cameras producing near-zero value frames
- **Observability gaps**  
  - No metrics on FPS, frame drops, Kafka latency

---

## Implemented Solutions
- **Single Python script for all cameras**  
  - Configurable per camera via arguments & config files
- **systemd process management**  
  - Auto-restart crashed producers
  - Resource limits (CPU, memory)
  - Time-based start/stop (6 AM → 6 PM)
- **GPU acceleration for FFmpeg on L40 node**  
  - Reduced CPU load
  - Stable streaming of multiple cameras
- **Kafka optimization**  
  - `acks=1`, `retries=5`, `compression_type='lz4'`
- **Prometheus + Grafana monitoring**  
  - Per-camera FPS, latency, dropped frames, producer uptime
- **Node distribution strategy**  
  - 1 GPU node (8–12 cams)
  - 3 CPU nodes (rest of the cameras)

---

## Repo Structure
