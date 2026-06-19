# 02 — Building Blocks Cheatsheet

For each: *what it's for* + *the one tradeoff that matters in an interview.* You know most of this — treat it as recall drilling. Cover the right column, recite from the left.

---

## Routing / Edge

**Load balancer** — distributes traffic across servers. L4 (transport, fast, dumb) vs L7 (HTTP-aware, can route by path/header, does TLS termination). *Tradeoff:* L7 is smarter but adds latency + is a heavier box. Default to L7 at the app tier.

**API gateway** — single entry: auth, rate limiting, routing, request aggregation. *Tradeoff:* one more hop + a potential bottleneck/SPOF, but centralizes cross-cutting concerns. Worth it once you have >1 service.

**CDN** — caches static (and cacheable dynamic) content near users. *Tradeoff:* huge latency win for reads, but cache invalidation is hard — stale content vs origin load. TTL + versioned URLs are your levers.

---

## Storage

**SQL (Postgres/MySQL)** — relational, ACID, joins, strong consistency. *Tradeoff:* vertical scaling ceiling; sharding is painful. **Default choice** — reach for it unless you can name why not. A single Postgres handles more than people think (tens of thousands of simple reads/sec with replicas).

**NoSQL — key-value (DynamoDB, Redis)** — fast lookups by key, horizontal scale. *Tradeoff:* you design around access patterns up front; no flexible queries/joins. Great for sessions, feeds, counters.

**NoSQL — document (MongoDB)** — flexible schema, nested docs. *Tradeoff:* easy to start, but weak cross-document consistency and you can denormalize yourself into update hell.

**NoSQL — wide-column (Cassandra)** — write-optimized, horizontally scalable, tunable consistency. *Tradeoff:* eventual consistency by default, no joins, query patterns locked at table-design time. Good for time-series, write-heavy logs/events.

**Blob store (S3)** — large unstructured objects (images, video, backups). *Tradeoff:* cheap + durable + scalable, but high per-object latency and no query. Store the blob in S3, the metadata + URL in your DB.

**SQL vs NoSQL, the one-liner:** start SQL for correctness and flexible queries; move to NoSQL when a specific access pattern needs horizontal scale or write throughput SQL can't give. Say *which* table and *why.*

---

## Speed

**Cache (Redis/Memcached)** — in-memory, sub-ms reads. Patterns: cache-aside (most common), write-through, write-back. *Tradeoff:* you trade freshness for speed — now you own invalidation, TTLs, and the thundering-herd / cache-stampede problem. Also a hot-key risk.

**Redis specifically** — also gives you sorted sets (leaderboards, feeds), TTL keys (rate limiting), pub/sub, distributed locks. Mention the data structure, not just "Redis."

---

## Decoupling / Throughput

**Message queue (SQS, RabbitMQ)** — async work, smooths spikes, decouples producer/consumer. *Tradeoff:* buys you resilience + buffering at the cost of end-to-end latency and at-least-once delivery (→ you must handle dedup/idempotency).

**Event stream / log (Kafka)** — ordered, replayable, multi-consumer, high throughput. *Tradeoff:* more ops complexity than a queue, but you get replay, ordering within a partition, and fan-out. Partition key choice determines ordering + hot-partition risk. **Order is only guaranteed within a partition.**

**Queue vs stream:** queue = work to be done once then gone; stream = durable event log many consumers read, replayable.

---

## Search / Specialized

**Search index (Elasticsearch)** — full-text, fuzzy, ranking, typeahead. *Tradeoff:* not your source of truth — it's a denormalized read-optimized copy you keep in sync (via CDC / dual write / stream), so it's eventually consistent.

**Object/graph for relationships (Neo4j)** — when "friends of friends" queries dominate. *Tradeoff:* niche; only raise if the problem is relationship-traversal-heavy.

---

## Scaling Primitives

**Replication** — copies of data. Leader-follower: writes to leader, reads from followers. *Tradeoff:* scales reads + gives failover, but follower reads are stale (replication lag) → read-your-own-writes problems.

**Sharding / partitioning** — split data across nodes by a key. *Tradeoff:* scales writes + storage horizontally, but cross-shard queries/transactions get hard, and a bad shard key → hot shards. Choosing the shard key is the whole game — say it out loud.

**Consistent hashing** — distributes keys so adding/removing a node moves minimal data. Raise it when you discuss sharding or distributed caches.

---

## Theory you'll get quizzed on

**CAP** — under a network **P**artition you must choose **C**onsistency or **A**vailability. Partitions are inevitable, so it's really C-vs-A *when one happens.* Banking → C. Social feed → A.

**PACELC** — extends CAP: *if* Partition → C vs A; **E**lse (normal operation) → **L**atency vs **C**onsistency. Captures the everyday tradeoff, not just failure mode.

**Consistency models** — strong (every read sees the latest write) vs eventual (replicas converge over time) vs causal (related events seen in order). Name the one each feature needs.

**Idempotency** — same request applied twice = same result. The antidote to at-least-once delivery and client retries. Idempotency key → dedup table. **Critical for payments.**
