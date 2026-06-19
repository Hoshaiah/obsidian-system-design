---
tags:
  - flashcards/theory
---

# Consistency & Theory

The vocabulary that lets you make — and defend — the C-vs-A call per feature.

---

State CAP precisely.
?
**Quick:** Under a network partition you must choose Consistency or Availability. Partitions are inevitable, so CAP is really a C-vs-A choice *when one occurs*.<br>
**Deeper:** CAP says a distributed system can guarantee at most two of Consistency (every read sees the latest write), Availability (every request gets a non-error response), and Partition tolerance (the system keeps working when the network drops messages between nodes). Because partitions cannot be avoided in real networks, P is non-negotiable, so the real decision is C-vs-A *during* a partition: refuse/stale-block to stay consistent (CP), or keep serving possibly-stale data to stay available (AP).

What does PACELC add over CAP?
?
**Quick:** Else (no partition) you still trade Latency vs Consistency — it covers normal operation, not just failures.<br>
**Deeper:** CAP only describes behavior during a network Partition (choose Consistency or Availability). PACELC extends it: if Partitioned → C-vs-A, Else (normal operation) → Latency-vs-Consistency. The insight is that even with a healthy network, synchronously keeping replicas consistent costs latency, so systems like Dynamo/Cassandra (PA/EL) favor low latency while Spanner-style systems (PC/EC) favor consistency.

Strong vs eventual vs causal consistency?
?
**Quick:** Strong: every read sees the latest write. Eventual: replicas converge over time. Causal: related events seen in the right order.<br>
**Deeper:** Strong (linearizable) consistency makes the system behave as if there is one copy with a single global order, so a read never returns stale data — but it costs coordination and latency. Eventual consistency only promises that, absent new writes, all replicas eventually agree; reads may be stale in the meantime (common in AP stores). Causal consistency is the useful middle ground: it preserves cause-and-effect ordering (e.g. you always see a reply after its original comment) without forcing a total global order, so it is cheaper than strong but avoids the "out of order" anomalies of pure eventual.

Define idempotency and why it matters.
?
**Quick:** Same request applied twice = same result; the antidote to at-least-once delivery + client retries. Essential for payments.<br>
**Deeper:** An operation is idempotent if applying it N times has the same effect as applying it once (e.g. "set balance to 100" or a create keyed by an idempotency key). It matters because reliable systems use at-least-once delivery — messages and client retries can duplicate a request — so without idempotency a retried "charge $50" could double-charge. The usual implementation is a unique idempotency key the server records, deduplicating repeats.

What are ACID transactions, briefly?
?
**Quick:** Atomicity, Consistency, Isolation, Durability — all-or-nothing, valid state, isolated concurrency, survives crash. The SQL guarantee.<br>
**Deeper:** Atomicity means a transaction either fully commits or fully rolls back — no partial writes. Consistency means it moves the database from one valid state to another, honoring constraints. Isolation means concurrent transactions don't see each other's uncommitted intermediate state (tunable via isolation levels). Durability means once committed, the data survives crashes (typically via a write-ahead log). These are the strong guarantees relational databases provide within a single node/cluster.

What is read-your-own-writes consistency?
?
**Quick:** A user always sees their own updates immediately; common fix is routing that user's reads to the leader or a sticky replica.<br>
**Deeper:** Read-your-writes is a session/client-centric consistency guarantee: after you submit a write, your subsequent reads reflect it, even if other users still see stale data. It matters in read-replica setups where reads normally hit lagging followers — you'd post a comment and not see it. Typical fixes are routing a user's reads to the leader for a short window after they write, or pinning the session to a replica known to have caught up.

What's the dual-write problem?
?
**Quick:** Writing to two systems (DB + queue) without a transaction → one can succeed and the other fail, causing drift. Fix with the outbox pattern.<br>
**Deeper:** A dual write is when one operation must update two independent systems (e.g. commit a row in the database and publish an event to Kafka) with no shared transaction. If the first succeeds and the second fails (crash, network blip), the systems diverge permanently. The outbox pattern fixes it by writing the event into the same database transaction as the data (an "outbox" table), then a separate relay reliably publishes those rows, making the two writes atomic via the one ACID transaction.

Optimistic vs pessimistic locking — when each?
?
**Quick:** Optimistic (version check on commit) for low contention; pessimistic (lock up front) for high contention where retries would thrash.<br>
**Deeper:** Pessimistic locking acquires a lock before touching data, blocking other writers until you commit — safe but it serializes access and can deadlock or stall. Optimistic locking takes no lock; it reads a version/timestamp, does the work, and on commit checks the version is unchanged, retrying if someone else wrote first. Optimistic wins when conflicts are rare (the happy path is lock-free), while pessimistic wins under heavy contention where constant optimistic retries would waste work.

What is a saga and what's it for?
?
**Quick:** A sequence of local transactions with compensating actions to undo on failure — keeps consistency across services without a distributed lock.<br>
**Deeper:** A saga manages a business process spanning multiple services, where each service does its own local ACID transaction and publishes an event to trigger the next step. Since there's no global transaction across services, if a later step fails you run compensating transactions to semantically undo the earlier ones (e.g. refund a charge). Sagas come in choreography (services react to each other's events) and orchestration (a central coordinator drives the steps), trading distributed-transaction locking for eventual consistency.

What is exactly-once delivery, really?
?
**Quick:** Practically unachievable end-to-end; you get at-least-once delivery + idempotent processing, which is effectively-once.<br>
**Deeper:** True exactly-once delivery is impossible across an unreliable network because the sender can't tell a lost message from a lost acknowledgment, so it must either risk dropping (at-most-once) or risk duplicating (at-least-once). The practical pattern is at-least-once delivery combined with idempotent consumers (dedup by message/idempotency key), giving "effectively-once" processing where duplicates are harmless. Some systems (e.g. Kafka transactions) offer exactly-once *semantics* within their own boundary, but that's scoped, not a universal end-to-end guarantee.
