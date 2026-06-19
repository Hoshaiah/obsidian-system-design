---
tags:
  - flashcards
---

# 05 — Flashcards

**Format:** works with the **Obsidian Spaced Repetition** plugin (st3v3nmw). `::` = single-line card; a lone `?` on its own line = multi-line card (question above, answer below). The `#flashcards` tag (in frontmatter above) makes the whole file a deck. No plugin? Just cover the answer and recite.

Start these Day 1. Add your own as you hit gaps — the cards *you* write from *your* misses are worth 10× any pre-made ones.

---

## The Framework

What are the 5 steps of the design framework, in order?
?
Requirements (functional + non-functional) → Core entities → API → High-level design → Deep dives.

In requirements, what separates a mid from a senior answer?::Nailing the non-functional requirements — scale, latency, and an explicit consistency-vs-availability call per feature — plus scoping things OUT loud.

Which framework step decides the interview outcome at mid/senior level?::The deep dives — anyone can draw boxes; going 3 layers deep on one component with real tradeoffs is the differentiator.

For each deep dive, what trifecta should you deliver?::(1) the problem, (2) two+ options with tradeoffs, (3) a justified pick + when you'd switch.

Why cursor pagination over offset for feeds?::Cursor is stable under inserts/deletes and O(1)-ish; offset shifts results and gets slow at high offsets.

---

## Estimation

1M requests/day ≈ how many QPS?::~12 QPS (1M ÷ 86,400).

How do you get peak QPS from average?::Multiply by 2–3× and state the multiplier.

2^30 corresponds to what size unit?::GB (≈ 1 billion bytes).
<!--SR:!2026-06-14,1,230-->

Order these by latency, fastest first: disk seek, memory read, datacenter round-trip, SSD read.::Memory (~100ns) < SSD (~100µs) < DC round-trip (~0.5ms) < disk seek (~10ms).

What's the *point* of back-of-envelope math in an interview?::Not the digits — the conclusion you draw (e.g. "100:1 read:write → this is read-heavy → cache + replicas"). Always say the conclusion out loud.

---

## Storage Choices

Default data store, and when do you move off it?::Start SQL (correctness, joins, flexible queries); move to NoSQL when a specific access pattern needs horizontal write throughput or scale SQL can't give.

When would you pick Cassandra (wide-column)?::Write-heavy, horizontally scalable, time-series/event data, tunable (eventual) consistency, query patterns known up front.

Where do blobs (images/video) go, and where does their metadata go?::Blob in object store (S3); metadata + URL in your DB.

SQL vs NoSQL in one sentence?::SQL for correctness + flexible queries; NoSQL for horizontal scale on a known access pattern.
<!--SR:!2026-06-16,3,250-->

---

## Caching

Name the main cache-read pattern and its main hazard.::Cache-aside; hazards are stale data (you own invalidation/TTL) and cache stampede / thundering herd on expiry.

Beyond GET/SET, name three Redis capabilities worth citing.::Sorted sets (feeds/leaderboards), TTL keys (rate limiting), pub/sub + distributed locks.

What's a hot key and how do you mitigate it?::A single key getting disproportionate traffic; mitigate with replication of that key, local caching, or key splitting.

---

## Messaging & Streams

Queue vs stream — the distinction?::Queue = work done once then gone (SQS/Rabbit); stream = durable, replayable, ordered log many consumers read (Kafka).

In Kafka, ordering is guaranteed across what scope?::Only within a single partition — chosen by the partition key.

Most queues give at-least-once delivery. What must consumers therefore implement?::Idempotency / dedup, so reprocessing a duplicate is harmless.

What determines hot-partition risk in Kafka, and one fix?::The partition key; fix by adding a random/sub-key suffix and aggregating in two stages.

---

## Scaling Primitives

Leader-follower replication scales what, and introduces what bug class?::Scales reads + gives failover; introduces replication lag → stale follower reads, read-your-own-writes issues.

Sharding scales what, and what's the hardest decision?::Scales writes + storage horizontally; the shard-key choice is the whole game (bad key → hot shards, cross-shard queries get hard).

What problem does consistent hashing solve?::Minimizes how much data moves when nodes are added/removed (vs naive modulo rehashing everything).

---

## Theory

State CAP precisely.::Under a network partition you must choose Consistency or Availability. Partitions are inevitable, so it's C-vs-A when one occurs.

What does PACELC add over CAP?::Else (no partition) you still trade Latency vs Consistency — covers normal operation, not just failures.

Strong vs eventual vs causal consistency?::Strong: every read sees latest write. Eventual: replicas converge over time. Causal: related events seen in correct order.

Define idempotency and why it matters.::Same request applied twice = same result; antidote to at-least-once delivery + client retries. Essential for payments.

---

## Feed Design

Fan-out on write vs read — the tradeoff?::Write (push): cheap reads, expensive writes, breaks for celebrities. Read (pull): cheap writes, expensive reads.

What's the celebrity problem and the standard fix?::One post by a high-follower account = millions of timeline writes. Fix: hybrid — push for normal users, pull for celebrities, merge at read time.

---

## Real-time / Messaging Systems

How do you deliver a chat message to an offline user?::Store-and-forward: persist to a per-user inbox, flush on reconnect, push via APNS/FCM.

What transport for real-time chat, and what state problem does it create?::Persistent websockets; creates connection-state routing — a registry of which server holds which user + a bus between servers.

How is message ordering typically guaranteed in chat?::Per-conversation sequence numbers/timestamps — ordering guaranteed only within a conversation.

---

## Payments  ⭐

First thing you say when designing a payment charge?::Idempotency key → dedup table, so retries/timeouts never double-charge (at-most-once).

What is a double-entry ledger and why use it?::Append-only, immutable, balanced debits/credits per transaction; the auditable source of truth enabling reconciliation.

Saga vs 2PC for distributed transactions — why prefer saga?::Saga uses compensating actions, non-blocking; 2PC blocks and has a coordinator SPOF.

What is reconciliation in a payment system?::A periodic batch comparing your ledger against the processor's records to catch drift/discrepancies.

How do you reliably handle async settlement webhooks?::Outbox pattern / reliable event handling + idempotent processing, so confirmations aren't lost or double-applied.
