# LLM Serving: Hardware

Notes on the hardware and system stack behind LLM inference: GPU HBM, inference runtimes, distributed serving, interconnects, remote memory and storage, and KV-cache data movement.

## Articles

- [AMD Infinity Context](AMD-Infinity-Context.md) — Explains the shared NVMe SSD-backed storage architecture: storage servers and GPU nodes connect via RDMA NICs and Ethernet switches, enabling direct-to-HBM KV-cache transfers when supported. Distinguishes the SSD backing tier from NFS-server DRAM caching, and examines NFS-over-RDMA, hipFile, and implications for MoE expert prefetching.
