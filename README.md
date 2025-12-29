# Dixon-IoT-Pipeline-Challenges
Dixon-IoT-Pipeline-Challenges/
│
├─ docs/
│   ├─ challenges.md        # Detailed list of all issues encountered
│   ├─ solutions.md         # Step-by-step solutions implemented
│   ├─ architecture.md      # HPC + Kafka + GPU node layout
│   ├─ deployment.md        # How producers are run (systemd, timers)
│   └─ observability.md     # Prometheus + Grafana dashboards
│
├─ examples/
│   └─ producer_example.py  # Minimal example of camera → Kafka pipeline
│
├─ systemd/                 # systemd service & timer files
│   ├─ kafka-camera@.service
│   ├─ kafka-start.service
│   └─ kafka-stop.service
│
└─ README.md
