# LLM Serving: Hardware

Notes on the hardware and system stack behind LLM inference: GPU HBM, inference runtimes, distributed serving, interconnects, remote memory and storage, and KV-cache data movement.

## Articles

- [AMD Infinity Context](AMD-Infinity-Context.md) — AMD's approach to extending KV-cache capacity beyond local HBM through shared storage and high-performance networking. Covers NFS, RDMA, DRAM, NVMe, hipFile, hipObject, and implications for MoE expert prefetching.
