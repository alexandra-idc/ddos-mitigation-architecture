# High-Performance Layer 7 DDoS Mitigation Architecture

This repository documents the design of a high-performance Layer 7 DDoS mitigation architecture using layered filtering techniques.

The architecture focuses on handling high packet-per-second (PPS) attacks by applying filtering at multiple layers of the networking stack.

---

## Documentation

- [Architecture Overview](architecture.md)
- [Packet Processing Pipeline](packet-processing.md)
- [Kernel Networking Tuning](kernel-tuning.md)
- [Threat Model](threat-model.md)
- [Observability](observability.md)

---

## Project Status

Design phase – implementation and benchmarking planned.

Future work includes:

- XDP program implementation
- traffic simulation
- performance benchmarking
- observability metrics
