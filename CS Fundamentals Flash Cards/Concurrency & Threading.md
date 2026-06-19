---
tags:
  - flashcards/concurrency
---

# Concurrency & Threading

The primitives, hazards, and mental models for correct concurrent code.

---

What is a race condition and why is it hard to test?
?
**Quick:** A race condition is when correctness depends on the timing or interleaving of concurrent operations on shared state.<br>
**Deeper:** Races are nondeterministic, so they hide in development and surface under load. Tools: TSan/Helgrind for data race detection, fuzzing schedulers (rr, Loom for Java/Rust), invariant checks. The deeper fix is design: prefer immutability, message passing, or per-thread state. "I added a mutex and the bug went away" usually means you added a different bug.

What's the difference between a mutex and a semaphore?
?
**Quick:** A mutex is mutual exclusion (one owner at a time, ownership tracked); a semaphore is a counter for available resources.<br>
**Deeper:** Mutexes enforce ownership (often only the locker can unlock, with priority inheritance), suited to protecting critical sections. Counting semaphores model resource pools (N permits to use a resource). Binary semaphores look like mutexes but lack ownership semantics, which is why they're bad for mutual exclusion but good for signaling between threads.

What are the four conditions for deadlock and how do we prevent them?
?
**Quick:** Mutual exclusion, hold-and-wait, no preemption, circular wait; prevent by breaking any one — most commonly via global lock ordering.<br>
**Deeper:** Beyond ordering, use try-lock with timeout and backoff, lock hierarchies (verified at runtime), or eliminate locks via lock-free or actor designs. Detection in production: deadlock detectors in JVMs and DBs find waits-for cycles and break them by killing one party. Always document lock-acquisition order in code with multiple locks.

What is an atomic operation and when do you need one?
?
**Quick:** An atomic op completes indivisibly from other threads' perspective; needed whenever multiple threads touch the same memory without locks.<br>
**Deeper:** Hardware provides CAS (compare-and-swap), fetch-and-add, etc. Compose them with memory ordering (acquire/release/seq-cst) to build lock-free queues, counters, and reference counts. Beware: atomic doesn't mean wait-free or contention-free; a CAS loop under contention is no faster than a mutex, sometimes slower.

What is a memory model and why should application developers care?
?
**Quick:** A memory model defines what reads can see after concurrent writes, including reordering allowed by compiler and CPU.<br>
**Deeper:** Without a memory model, "obvious" code can break: writes from one thread may be reordered or invisible. C++/Java/Go/Rust each define ordering primitives (volatile/atomic/memory_order). x86 is relatively strong (TSO); ARM is weak and exposes more reordering, which is why some bugs only appear on Apple Silicon servers. Always reason about happens-before, not wall-clock.

What's the difference between concurrency and parallelism?
?
**Quick:** Concurrency is dealing with many things at once (composition of independent computations); parallelism is doing many things at once (simultaneous execution).<br>
**Deeper:** A single core can be concurrent (cooperative multitasking, async I/O) but not parallel. Parallelism requires multiple cores. The distinction shapes design: an async server is concurrent and might be single-threaded; a worker pool is parallel. Rob Pike's "Concurrency is not parallelism" lecture is the canonical reference.

How do async/await and coroutines differ from OS threads?
?
**Quick:** Coroutines are cooperative, scheduled in userspace by the runtime; threads are preemptive, scheduled by the kernel.<br>
**Deeper:** Async tasks are cheap (KB of stack, microseconds to switch) so you can have millions; OS threads are expensive (MBs of stack, microseconds plus cache pollution). Tradeoff: cooperative scheduling means a CPU-bound task can block the loop ("don't block the event loop"), and the colored-functions problem (async infects callers). Go's goroutines and Java virtual threads hide the seam by combining both models.

What is the actor model and where does it shine?
?
**Quick:** Actors are isolated state machines that communicate only via async messages; great for distributed systems and fault isolation.<br>
**Deeper:** Erlang/Elixir/Akka popularized this. No shared mutable state means no data races by construction. Failures isolate per-actor with supervision trees ("let it crash"). Tradeoffs: message-passing overhead, harder to reason about ordering, and back-pressure must be designed explicitly. The cloud equivalent is durable workflow engines and event-driven microservices.

What's the difference between optimistic and pessimistic concurrency control?
?
**Quick:** Pessimistic locks upfront assuming conflicts; optimistic proceeds without locks and detects conflicts at commit.<br>
**Deeper:** Optimistic (CAS, version numbers, MVCC) wins when conflicts are rare; cheap path, retry path for rare cases. Pessimistic (mutexes, SELECT FOR UPDATE) wins when conflicts are common; pay the lock cost once instead of repeated retries. Web apps often use optimistic at the HTTP layer (If-Match etags) and pessimistic in the DB hot path.

What is a producer-consumer pattern and how is it implemented correctly?
?
**Quick:** Producers add work to a bounded buffer; consumers remove and process; synchronization handles full/empty conditions.<br>
**Deeper:** Implemented with a thread-safe queue (lock + condition variables, or lock-free MPMC like Disruptor). Bounded buffers create back-pressure, preventing producer overrun. Get this wrong and you get unbounded memory growth or live-lock. In async systems, the equivalent is reactive streams (Project Reactor, RxJava) with explicit demand signaling.

What is false sharing and how do you avoid it?
?
**Quick:** Two cores writing to different variables in the same cache line invalidate each other's cache; pad to separate lines.<br>
**Deeper:** Cache lines are typically 64 bytes. If two threads increment counters that happen to live next to each other in memory, every increment causes coherence traffic, devastating performance. Fix: align hot fields to cache line boundaries (`alignas(64)`, padding, Java `@Contended`). Common pitfall in concurrent counters and queue head/tail pointers.

What is the difference between lock-free and wait-free data structures?
?
**Quick:** Lock-free guarantees system-wide progress; wait-free guarantees per-thread progress in bounded steps.<br>
**Deeper:** Most "lock-free" structures (queues, stacks) use CAS retry loops, so a thread could theoretically livelock under extreme contention. Wait-free structures (rare in practice) guarantee no thread starves. The real benefit of lock-free design is composability and fault tolerance: a thread crashing while holding a mutex deadlocks the system; a thread crashing mid-CAS doesn't.
