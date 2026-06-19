---
tags:
  - flashcards/computer-architecture
---

# Computer Architecture

How modern CPUs actually execute your code — pipelines, caches, branch prediction, and what makes hot loops fast.

---

What is the cache hierarchy and why does it exist?
?
**Quick:** L1/L2/L3 caches sit between the CPU and DRAM; each level is larger but slower, exploiting locality to hide DRAM latency.<br>
**Deeper:** Typical numbers: L1 ~1 ns and 32-64 KB per core, L2 ~3-5 ns and 256 KB-1 MB, L3 ~10-20 ns shared across cores (tens of MB), DRAM ~100 ns. DRAM is 100x slower than L1, so a cache miss can stall the CPU for hundreds of cycles. This is why algorithm choice is increasingly about access patterns over Big-O — a linear scan that fits in cache beats a "smarter" pointer-chasing structure.

What is CPU pipelining and what hazards stall it?
?
**Quick:** Pipelining overlaps instruction fetch, decode, execute, memory, writeback stages so multiple instructions are in flight; hazards (data, control, structural) cause stalls.<br>
**Deeper:** Modern x86/ARM cores are 14-20 stage superscalar out-of-order pipelines issuing multiple instructions per cycle. Hazards: data (dependency on previous result), control (unpredicted branch), structural (resource conflict). Mitigations: out-of-order execution, register renaming, speculative execution. Spectre/Meltdown exploited the gap between architectural and microarchitectural state.

How does branch prediction work and how do you optimize for it?
?
**Quick:** The CPU predicts the outcome of conditional branches to keep the pipeline full; mispredictions flush the pipeline costing 10-20 cycles.<br>
**Deeper:** Modern predictors are sophisticated (TAGE, perceptron) and learn patterns over time. Unpredictable branches (sorted vs random data) can be 5x slower. Optimization techniques: branchless code (cmov, bitwise tricks), sorting input to make branches predictable, profile-guided optimization (PGO) so hot branches are arranged for fall-through. The famous "sorted array" StackOverflow question illustrates this.

What is the difference between instruction-level parallelism (ILP) and SIMD?
?
**Quick:** ILP executes independent scalar instructions concurrently within a core; SIMD applies one instruction to a vector of data.<br>
**Deeper:** ILP is exploited transparently by the out-of-order engine if you have independent ops. SIMD (SSE/AVX, NEON, SVE) requires explicit vectorization — by the compiler (auto-vec on tight loops) or by hand (intrinsics). 4-16x speedups are common for math-heavy code. Modern workloads (ML, image processing, JSON parsing with simdjson) lean heavily on SIMD.

What is the MMU and what does the TLB cache?
?
**Quick:** The MMU translates virtual to physical addresses using page tables; the TLB caches recent translations to avoid walking page tables.<br>
**Deeper:** Page-table walks can cost 100+ cycles; a TLB hit is essentially free. TLBs are small (64-2048 entries) and split by privilege/page-size. TLB misses dominate workloads with large random memory footprints. Huge pages (2 MB / 1 GB) reduce TLB pressure dramatically — databases and ML inference engines often enable them.

What's the difference between big-endian and little-endian, and where does it matter?
?
**Quick:** Little-endian stores least-significant byte first (x86, ARM default); big-endian stores most-significant first (network byte order).<br>
**Deeper:** Endianness only matters when bytes cross a boundary: file formats, network protocols, memory-mapped hardware. Always use ntohl/htonl or explicit serialization. Modern CPUs are overwhelmingly little-endian, so the issue mostly arises with legacy protocols (TCP/IP headers) or cross-platform data files (use a canonical format like Protobuf).

What is locality of reference, and what's the difference between spatial and temporal locality?
?
**Quick:** Temporal: recently accessed data is likely accessed again soon. Spatial: nearby data is likely accessed soon.<br>
**Deeper:** Caches exploit both: a fetched line stays for temporal reuse and contains neighbors for spatial reuse. Algorithm design that prioritizes locality (blocked matrix multiplication, cache-oblivious algorithms, struct-of-arrays for hot fields) often beats algorithms with better Big-O on real hardware. Pointer chasing is the enemy of locality.

What's the difference between CISC and RISC architectures?
?
**Quick:** CISC (x86) has variable-length, complex instructions; RISC (ARM, RISC-V) has fixed-length, simple instructions.<br>
**Deeper:** The line blurred long ago: modern x86 decodes complex instructions into RISC-like micro-ops internally. Practical implications today: RISC instruction decoding is simpler and more power-efficient (why ARM dominates mobile and is taking servers), but x86 retains a binary compatibility moat. Apple Silicon proved RISC at the high end is competitive on perf-per-watt.

How does cache coherence work in a multicore system?
?
**Quick:** Coherence protocols (MESI etc.) ensure each cache line has one writer at a time and propagate invalidations to other caches.<br>
**Deeper:** When one core writes to a cache line, all other copies must be invalidated (write-invalidate) before the write completes architecturally. This is what makes false sharing expensive: even non-conflicting writes to the same line generate coherence traffic. NUMA adds another wrinkle: cross-socket coherence is much slower than intra-socket.

What is speculative execution and how was it exploited by Spectre/Meltdown?
?
**Quick:** CPUs execute instructions ahead of confirming branches/permissions and roll back if wrong; side effects (cache state) leak data.<br>
**Deeper:** Spectre tricks branch prediction into speculatively accessing secret memory and leaks it via cache timing. Meltdown bypasses kernel/user separation similarly. Mitigations (KAISER, retpolines, microcode) cost 5-30% on syscall-heavy workloads. The lesson: microarchitectural state is observable and must be considered for security, not just architectural state.

What is NUMA and why does it matter on multi-socket servers?
?
**Quick:** Non-Uniform Memory Access: each CPU socket has local DRAM that's faster than remote DRAM on another socket.<br>
**Deeper:** Crossing NUMA nodes adds ~50-100% latency. The OS scheduler tries to keep threads near their memory, but it can guess wrong. NUMA-aware allocation (numactl, libnuma, JVM flags) and pinning threads to nodes can yield large speedups for memory-bound workloads. Cloud VMs increasingly expose NUMA topology.

What's the difference between a register, a stack variable, and a heap variable in terms of access speed?
?
**Quick:** Registers are zero-latency; stack is L1-fast (typically resident); heap depends on cache state and may miss to DRAM.<br>
**Deeper:** Compilers aggressively try to keep hot variables in registers via register allocation. Stack frames are tiny and hot in L1. Heap accesses depend on the data layout — adjacent allocations land near each other, but allocations spread over time fragment cache locality. This is why arena allocators and object pooling can yield 5-10x perf wins in hot paths.
