# Dixon IoT Lab Pipeline: System Architecture

This document describes the **hardware, network, and software architecture** for the multi-camera RTSP → Kafka pipeline deployed in the Dixon IoT Lab.

---

## 1. High-Level Architecture

         ┌─────────────────────────────┐
         │     RTSP Cameras (40)       │
         │  6 AM → 6 PM Streaming     │
         └─────────────┬─────────────┘
                       │
                       ▼
      ┌──────────────────────────────────┐
      │      HPC Cluster (4 Nodes)       │
      │                                  │
      │ ┌───────────────┐  ┌───────────┐│
      │ │ HPC-GPU Node  │  │ HPC-CPU-1 ││
      │ │  L40 GPU      │  │  CPU Only ││
      │ │ 8–12 cameras │  │ 10 cameras││
      │ └───────────────┘  └───────────┘│
      │ ┌───────────┐  ┌─────────────┐ │
      │ │ HPC-CPU-2 │  │ HPC-CPU-3   │ │
      │ │ 10 cams   │  │ 8–10 cams   │ │
      │ └───────────┘  └─────────────┘ │
      └───────────────┬────────────────┘
                      │
                      ▼
         ┌─────────────────────────────┐
         │       Kafka Cluster         │
         │  3 Brokers / 1 Zookeeper    │
         │  Topics: Camera Frames      │
         └─────────────┬─────────────┘
                       │
                       ▼
         ┌─────────────────────────────┐
         │ Prometheus + Grafana        │
         │  Metrics: FPS, Latency,    │
         │  Dropped Frames, Uptime    │
         └─────────────────────────────┘

---

## 2. Node Roles

| Node       | Role                        | Assigned Cameras | Notes |
|------------|----------------------------|----------------|-------|
| HPC-GPU    | FFmpeg GPU decoding         | 8–12           | L40 GPU, CUDA acceleration, reduced CPU load |
| HPC-CPU-1  | FFmpeg CPU decoding         | 10             | CPU-only, balanced load |
| HPC-CPU-2  | FFmpeg CPU decoding         | 10             | CPU-only, balanced load |
| HPC-CPU-3  | FFmpeg CPU decoding         | 8–10           | CPU-only, balanced load |

> GPU node handles streams that require high FPS and heavy encoding. CPU nodes handle remaining cameras. Node distribution prevents bottlenecks and OOM crashes.

---

## 3. Data Flow

1. **Cameras** → produce RTSP streams.
2. **Producer scripts** (Python + FFmpeg) read RTSP streams.
   - GPU node uses CUDA-accelerated FFmpeg.
   - CPU nodes use software decoding.
3. **Python producer** encodes frames to JPEG.
4. **Frames + timestamp** sent to Kafka topics.
   - Kafka optimization: `acks=1`, `retries=5`, `compression_type='lz4'`.
5. **Consumers** (optional) read frames for:
   - AI inference
   - Dashboard visualization
   - Storage
6. **Metrics exported** to Prometheus:
   - FPS per camera
   - Kafka latency
   - Dropped frames
   - Producer uptime
7. **Grafana dashboards** visualize pipeline health and performance.

---

## 4. Scheduling & Automation

- Producers run **6 AM → 6 PM** using:
  - systemd timers OR
  - cron jobs
- systemd ensures:
  - Automatic restart on crash
  - Resource limits (CPU, Memory)
  - Logging via `journalctl`

---

## 5. Observability Layer

- **Prometheus exporter** in Python exposes metrics per producer.
- **Grafana dashboard** shows:
  - Frames/sec per camera
  - Dropped frames
  - Kafka send latency
  - FFmpeg restart count
  - Node-level CPU/GPU usage
- Alerts can be configured for:
  - Producer downtime
  - FPS below threshold
  - Kafka broker issues

---

## 6. Notes on Scaling

- Each HPC node can run multiple producers.
- GPU node limited to **8–12 streams** to avoid OOM.
- CPU nodes can scale based on available cores and memory.
- Night-time streams can be throttled or disabled to save resources.

---

This architecture ensures:
- **High throughput** for 40 cameras
- **Automated, reliable execution**
- **Resource optimization** (CPU/GPU)
- **Full observability** via Prometheus + Grafana
