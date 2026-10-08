# knowledge-base
Keeping a record of what I have studied.

In this README, I will maintain a topic-wise list of articles and papers that I have studied or glossed over, along with a 2–3 line description for each paper.

Within a separate directory for each topic, there will be individual `.md` files for each article/paper with its link and a detailed explanation of that topic.

## Topics

### LLM Serving: Hardware

- [AMD Infinity Context](LLM-Serving-Hardware/AMD-Infinity-Context.md) — Uses a shared, NVMe SSD-backed storage pool connected to GPU servers through RDMA-capable NICs and network switches, enabling KV-cache transfers into GPU HBM without host-DRAM staging where supported. Explores why NVMe provides capacity while NFS, remote DRAM caching, hipFile, and NIC-based data movement determine the access path.
