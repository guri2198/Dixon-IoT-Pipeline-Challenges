# Dixon IoT Lab: Pipeline Solutions

This document outlines the production-grade solutions implemented to address the challenges identified in `challenges.md`.

---

## 1. Unified Python Producer
- Replaced 40 separate scripts with **one configurable producer**.
- Each camera instance started with:
  ```bash
  python producer.py --camera-id D1Cam01 --rtsp <RTSP_URL> --topic D1Cam01
