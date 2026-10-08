# AMD Infinity Context

**Article:** [AMD Infinity Context](https://rocm.blogs.amd.com/software-tools-optimization/amd-infinity-context/README.html)

## Overview

AMD Infinity Context extends the effective capacity available for LLM context/KV-cache data beyond a GPU's local HBM by using shared storage and high-performance network transfers. It targets long-context inference and reuse of previously computed KV-cache blocks.

## Serving software stack

- **vLLM:** Inference engine; executes prefill and decode, handles batching, and manages GPU-side KV cache.
- **llm-d:** Optional distributed inference coordination, including cache-aware routing and prefill/decode disaggregation.
- **Kubernetes:** Optional infrastructure orchestration, responsible for deploying and managing GPU workloads.
- **LMCache:** KV-cache storage, offload, and reuse across tiers.
- **NIXL:** Data-transfer abstraction for moving inference state across memory/storage systems.
- **hipFile / hipObject:** AMD GPU-oriented file and object storage I/O interfaces.

vLLM can run directly across multiple GPUs and nodes without llm-d or Kubernetes. Those layers address cluster orchestration and request routing, rather than transformer execution itself.

## NFS, TCP, RDMA, and storage tiers

These are different abstractions:

| Term | Meaning |
| --- | --- |
| NFS | Network File System: a protocol for accessing files hosted on another machine |
| TCP | Reliable, ordered network transport |
| RDMA | Network data transfer with reduced CPU intervention and memory copying |
| DRAM | Volatile memory; may cache file data on the NFS server |
| NVMe SSD | Persistent backing storage, typically larger and cheaper per GB than DRAM |
| NIC | Network interface connecting a GPU node or storage server to the network |

NFS can operate over TCP or RDMA. NFS does not have its own special memory: **NFS server memory** means DRAM on the machine serving the filesystem.

### Cache hit in server DRAM

```
NFS server DRAM -> RDMA-capable NIC -> network -> GPU NIC -> GPU HBM
```

This path does not need an NVMe SSD read. GPU-direct transfers require compatible hardware, memory registration, and software support.

### Cache miss in server DRAM

```
NVMe SSD -> server buffers/DRAM -> storage NIC -> network -> GPU NIC -> GPU HBM
```

The SSD provides capacity and persistence. If the entire working set can be kept in remote DRAM, the SSD need not be on the serving critical path; a purpose-built RDMA memory service could even avoid filesystem overhead.

## Why specialized NICs?

RDMA-capable NICs already reduce CPU overhead and host-memory copies during transfers. A SmartNIC/DPU could additionally offload storage-protocol work, manage caching, or schedule prefetches. But it does **not** make SSD media intrinsically faster, nor does it remove network-bandwidth constraints.

## hipFile vs hipObject

- **hipFile:** File-oriented I/O between storage/filesystems and GPU memory, with accelerated direct paths when supported.
- **hipObject:** Object-oriented access (for example, S3-compatible object storage), with accelerated transfers on supported backends.
- **S3:** An object storage API/service, not a specific physical memory technology; its backend may use SSDs or other media.

## Hardware research implications

A remote KV-cache tier reduces pressure on GPU HBM but adds network/storage transfers. Its usefulness depends on reuse, bandwidth, latency, and whether retrieval is faster than recomputation.

For MoE expert weights, the same hierarchy can be considered, but fetching an expert reactively may be too slow. Research opportunities include expert prediction, prefetch scheduling, expert cache replacement, overlap with GPU computation, and NIC-side data movement. KV-cache locality and MoE expert locality are related but distinct problems.

## Key takeaway

**NFS = remote file access; DRAM/NVMe = where data resides; TCP/RDMA = how it travels; NIC = network hardware; HBM = GPU working memory.** The critical performance question is which tier and link limits the transfer, not simply whether the data is accessed through NFS.
