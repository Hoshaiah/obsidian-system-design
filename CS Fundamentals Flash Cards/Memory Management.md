---
tags:
  - flashcards/memory-management
---

# Memory Management

Stack/heap, allocators, garbage collection, and the patterns for memory-safe and memory-efficient code.

---

What's the difference between stack and heap allocation?
?
**Quick:** Stack is LIFO per-thread storage, fixed size, fast (bump pointer); heap is shared, dynamic, requires an allocator and is slower.<br>
**Deeper:** Stack allocation is essentially free — just adjusting the stack pointer — and benefits from cache locality. Heap allocation may require lock contention in a global allocator, bookkeeping (size, free list), and risks fragmentation. Lifetimes differ: stack frees on function return; heap requires explicit free or GC. Choosing stack (or arena/region) over heap is the easiest perf win in many systems.

How does mark-and-sweep garbage collection work?
?
**Quick:** Mark phase traces from GC roots flagging live objects; sweep phase reclaims unmarked memory.<br>
**Deeper:** Pauses can be long for large heaps unless made incremental or concurrent. Variants: tri-color marking with write barriers (used in Go, modern Java GCs) makes mark concurrent. Drawbacks vs. reference counting: latency spikes during collection, requires runtime knowledge of pointers. Modern GCs (G1, ZGC, Shenandoah) target sub-millisecond pauses through region-based and concurrent collection.

What is generational garbage collection and why does it work?
?
**Quick:** Most objects die young, so GC focuses on the young generation cheaply and promotes survivors to an old generation collected rarely.<br>
**Deeper:** Based on the weak generational hypothesis. The young gen is small and collected frequently with copying GC (compacting + cheap). The old gen is large but stable, collected with mark-sweep-compact infrequently. Cross-generation references are tracked via card tables or remembered sets. This is why allocating in a tight loop is often fine in Java/Go but reusing buffers still helps for sustained throughput.

How does reference counting work and what are its drawbacks?
?
**Quick:** Each object tracks how many references point to it; freed when count reaches zero.<br>
**Deeper:** Pros: deterministic destruction (RAII-like), low pause time. Cons: can't reclaim cycles without a cycle detector (Python's gc, Swift's weak refs), atomic increments are expensive in multithreaded contexts, memory overhead per object. CPython uses RC + cycle detector. Modern designs use RC for shared resources but RAII or arenas for fast paths.

What is RAII and why does it matter beyond C++?
?
**Quick:** Resource Acquisition Is Initialization ties resource lifetime to object lifetime, ensuring cleanup via destructors.<br>
**Deeper:** Files, locks, sockets, memory — RAII guarantees release even on exceptions. The pattern shows up everywhere: Python's `with`, C#'s `using`, Java's try-with-resources, Rust's Drop. The deeper insight is that explicit `finally` blocks scale badly; tying cleanup to scope is more robust. Rust enforces this at compile time, making leaks structurally rare.

What's the difference between a pointer and a reference?
?
**Quick:** A pointer is a variable holding an address; a reference is an alias for an existing object, typically non-null and non-rebindable.<br>
**Deeper:** In C++, references can't be null and can't be reseated, simplifying reasoning. Java/Python have only references (no raw pointers in normal code). Rust references add compile-time aliasing rules (one mutable XOR many shared). The reference vs pointer distinction is also semantic: references express "borrowed access," pointers express "address arithmetic."

What's a memory leak and how can it happen in a garbage-collected language?
?
**Quick:** Memory that remains allocated despite being logically unneeded; in GC languages it happens via lingering references.<br>
**Deeper:** Common GC-language leaks: caches without eviction, listeners not unregistered, static collections holding entries, ThreadLocals on thread-pool threads, closures capturing big contexts. Native memory leaks (off-heap buffers, JNI) bypass GC entirely. Heap dumps + retained-size analysis (Eclipse MAT, jmap, pprof) are the standard tools.

What is a cache line and why does it matter for performance?
?
**Quick:** A cache line is the unit of memory the CPU loads from RAM (typically 64 bytes); access patterns within a line are nearly free.<br>
**Deeper:** Sequential access reads whole lines into L1, making linear scans 10-100x faster than pointer chasing. Struct-of-arrays vs array-of-structs choices, alignment, and packing matter enormously. False sharing (two threads writing to the same line) causes cache coherency traffic that destroys throughput. "Mechanical sympathy" is the discipline of writing code that matches hardware behavior.

What does memory fragmentation mean and when does it bite?
?
**Quick:** Free memory split into pieces too small to satisfy allocations even though total free space is enough.<br>
**Deeper:** External fragmentation hits long-running native processes (game engines, embedded). Internal fragmentation is wasted space inside an allocated block (slab/sized-class allocators round up). Solutions: compacting GC, arena/region allocators, jemalloc/tcmalloc size classes. Symptom in production: RSS grows even though logical memory usage is stable.

What's the difference between RSS and virtual memory in process stats?
?
**Quick:** Virtual memory is the address space the process can reference; RSS is the physical memory currently mapped in.<br>
**Deeper:** A 100 GB virtual address space may have a 1 GB RSS if most is unmapped or paged out. Memory-mapped files inflate VSZ without using RAM until pages are touched. Container memory limits are typically enforced against RSS + page cache + kernel structures (cgroup memory), which is why monitoring RSS alone misleads.

What is a stack overflow and what are common causes?
?
**Quick:** A thread's call stack exceeded its fixed allocation; common causes are unbounded recursion or huge local arrays.<br>
**Deeper:** Default stack sizes: 8 MB on Linux user threads, 1 MB on Windows, 64-512 KB on green threads. Deep recursion (parsing, graph traversal) can overflow on real-world inputs. Mitigation: iterative algorithms, explicit stacks, tail-call optimization (where supported), or stack-size tuning. Stack canaries detect stack-buffer overflows used in exploits.

What is the difference between malloc, calloc, realloc, and free?
?
**Quick:** malloc allocates uninitialized memory; calloc allocates zeroed; realloc grows/shrinks an existing allocation; free releases it.<br>
**Deeper:** calloc can be cheaper than malloc+memset because the allocator may already know pages are zero (kernel returns zeroed pages on first touch). realloc may move the buffer, invalidating pointers. Modern allocators (jemalloc, tcmalloc, mimalloc) provide thread caches and size classes to reduce contention and fragmentation under load.
