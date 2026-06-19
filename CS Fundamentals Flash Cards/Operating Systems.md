---
tags:
  - flashcards/operating-systems
---

# Operating Systems

How the OS abstracts CPU, memory, and I/O — and what leaks through to your code.

---

What's the difference between a process and a thread?
?
**Quick:** Processes have isolated address spaces; threads share memory within a process.<br>
**Deeper:** Process creation is expensive (new page tables, file descriptors, PID), but isolation gives fault containment and security. Threads share heap, code, and file descriptors but have their own stack and registers, so context switches are cheaper and communication is just reading shared memory. The cost is that any thread can corrupt the shared state, which is why concurrent programming is hard. Modern systems often use process-per-tenant isolation (Chrome, nginx workers) combined with thread pools inside.

What happens during a context switch and why is it expensive?
?
**Quick:** The kernel saves the current thread's registers and state, picks a new thread, and restores its state; cost dominated by cache and TLB pollution.<br>
**Deeper:** Direct cost is a few microseconds (save ~16-32 registers, swap stack pointer, possibly kernel mode transition). The hidden cost is colder caches: instruction cache, data cache, and TLB are all loaded with the new thread's working set, causing tens of microseconds of slowdown afterward. This is why thread pools, batching, and io_uring exist — to amortize switches.

How does virtual memory work and what is the role of the MMU?
?
**Quick:** Each process sees its own contiguous address space; the MMU translates virtual addresses to physical frames using page tables, with the TLB caching translations.<br>
**Deeper:** Pages (typically 4 KB) let the OS lazily allocate, swap to disk, share read-only code, and implement copy-on-write fork. A TLB miss can cost 100+ cycles as the hardware walks multi-level page tables. Huge pages (2 MB / 1 GB) reduce TLB pressure for memory-heavy workloads like databases. Page faults that hit disk are millions of cycles, which is why working-set size matters so much.

What is a page fault and what are the different kinds?
?
**Quick:** A page fault occurs when access targets a page not currently in physical memory; types include minor, major, and invalid.<br>
**Deeper:** Minor faults (soft) just need the OS to map a page that's already in RAM (e.g., shared library, COW). Major faults (hard) require disk I/O — extremely expensive, often >1 ms. Invalid faults are segfaults (SIGSEGV). Page fault rate is a key health metric; sustained major faults indicate memory pressure or thrashing where the system spends more time paging than working.

What are the four necessary conditions for deadlock?
?
**Quick:** Mutual exclusion, hold-and-wait, no preemption, and circular wait — all four must hold.<br>
**Deeper:** Break any one to prevent deadlock. Common strategies: impose a global lock ordering (prevents circular wait), use try-lock with backoff, or allow preemption (e.g., database transaction abort). Detection-based recovery uses wait-for graphs to find cycles. Livelock is the cousin where threads keep responding to each other but make no progress.

What's the difference between user mode and kernel mode, and why do syscalls cost so much?
?
**Quick:** User mode has restricted privileges; kernel mode has full hardware access. Syscalls switch modes, saving state and crossing a security boundary.<br>
**Deeper:** A syscall traps into the kernel, saves user state, validates arguments (defensively, against malicious input), executes, and returns. The cost (~100s of ns to microseconds) plus side effects (CPU pipeline flush, sometimes TLB) is why high-perf systems batch syscalls (writev, sendmmsg) or bypass the kernel (DPDK, io_uring, eBPF).

What are the main scheduling algorithms and their tradeoffs?
?
**Quick:** FCFS (simple, unfair), Round Robin (fair, latency-friendly), SJF (optimal turnaround, starvation risk), MLFQ (general-purpose), CFS (Linux, weighted fair).<br>
**Deeper:** Real systems use multi-level feedback queues (MLFQ) or fair queueing variants. Linux CFS picks the task with the smallest vruntime, giving proportional CPU share by nice value. Real-time schedulers (SCHED_FIFO, SCHED_DEADLINE) bypass fairness for latency guarantees. Tradeoffs cluster around throughput vs latency vs fairness; you almost never get all three.

What's the difference between blocking, non-blocking, and asynchronous I/O?
?
**Quick:** Blocking waits in the syscall; non-blocking returns EAGAIN if not ready; async hands off and notifies on completion.<br>
**Deeper:** Blocking I/O is simplest but ties up a thread per connection. Non-blocking + readiness notification (select/poll/epoll/kqueue) drives event loops in nginx, Node.js, and Netty. True async (io_uring, Windows IOCP) returns completion events for the whole operation, including buffer ownership, enabling kernel-side batching and zero syscall hot paths.

What IPC mechanisms exist and what are their tradeoffs?
?
**Quick:** Pipes, sockets, shared memory, message queues, signals — fastest is shared memory; most flexible is sockets.<br>
**Deeper:** Pipes and Unix domain sockets are byte streams with kernel-managed buffering, simple to use but require copying. Shared memory avoids the copy but requires explicit synchronization (semaphores/futexes). Message queues add structure (POSIX mq, Mach ports). Signals are minimal and async-safe but limited to small set; never carry significant data over them.

How does a mutex differ from a spinlock and when do you use each?
?
**Quick:** A mutex puts waiters to sleep (context switch); a spinlock busy-waits. Spinlock wins when expected wait is shorter than a context switch.<br>
**Deeper:** Spinlocks are essential in kernel code where you can't sleep (interrupt context) and for very short critical sections on multicore. Userspace mutexes typically spin briefly before sleeping (adaptive). Holding a spinlock too long wastes CPU; holding a mutex too long causes pile-ups. Real-world systems use reader-writer locks, sequence locks, or RCU for read-mostly workloads.

What is copy-on-write (COW) and where does it appear?
?
**Quick:** Pages are shared and marked read-only; on write, the kernel copies the page lazily.<br>
**Deeper:** Powers fork() so creating a new process is cheap (no eager copy of the entire address space). Also underlies snapshots in btrfs/ZFS, container image layers, and immutable persistent data structures. Downside: write-heavy workloads after fork cause page fault storms; that's why databases often use a daemon process model instead of forking workers.

What is thrashing and how do you detect it?
?
**Quick:** Thrashing is when the system spends most of its time paging instead of executing; detect via high page fault rate and low CPU utilization.<br>
**Deeper:** Happens when total working set exceeds physical RAM; processes fight for pages and constantly evict each other. The OOM killer or admission control (working-set model, page-fault frequency) is needed to break the cycle. Modern cloud workloads often see thrashing-like symptoms when containers exceed memory limits and get swapped or killed.
