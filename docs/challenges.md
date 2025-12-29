# Dixon IoT Lab: Pipeline Challenges

This document summarizes the key challenges encountered while building the multi-camera RTSP → Kafka pipeline in the Dixon IoT Lab.

---

## 1. Kafka Broker / ISR Issues
- Single broker crashes or becomes unavailable.
- Data transmission stops until manual intervention.
- Leader elections caused temporary downtime.
- ISR (In-Sync Replica) inconsistencies under high throughput.

**Impact:**  
Frame loss, interrupted pipelines, unreliable streaming for consumers.

---

## 2. FFmpeg Backpressure & Frame Drops
- Some frames were not produced consistently.
- CPU-heavy nodes caused FFmpeg to lag behind real-time streaming.
- Timeout or incomplete frames occurred, triggering pipeline restarts.

**Impact:**  
Lost or delayed frames, lower average FPS, corrupted payloads.

---

## 3. Tyron Machine OOM / Hangs
- Python producers on Tyron (L40 GPU node) occasionally crashed due to GPU memory exhaustion.
- Heavy multi-stream decoding caused system-level hangs.
- Night-time camera streams produced frames of little value, but still consumed GPU/CPU resources.

**Impact:**  
Producer crashes, process hangs, reboot required.

---

## 4. Manual Script Management
- 40 separate Python scripts, each run manually.
- High risk of human error when starting/stopping.
- No automated start/stop schedule.

**Impact:**  
Unpredictable uptime, inconsistent streaming, increased maintenance effort.

---

## 5. Night-time Low Visibility
- Cameras produced frames with minimal visual information during the night.
- Streaming these frames consumed resources unnecessarily.

**Impact:**  
Wasted compute resources, higher GPU/CPU load, unnecessary Kafka traffic.

---

## 6. Observability Gaps
- No metrics for:
  - Frames per second (FPS)
  - Kafka send latency
  - Dropped frames
  - Producer uptime
- Difficulty diagnosing bottlenecks or failures.

**Impact:**  
Troubleshooting was slow and reactive; no proactive monitoring.

---

## 7. GPU / CPU Resource Allocation
- Without controlled distribution, some nodes overloaded.
- GPU node under heavy load caused slower encoding/decoding for multiple cameras.
- CPU nodes occasionally bottlenecked under high frame rate streams.

**Impact:**  
Unstable frame production, inconsistent Kafka throughput.

---

