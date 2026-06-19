---
tags:
  - flashcards/messaging
---

# Messaging & Streams

Decoupling, buffering, and getting delivery + ordering right.

---

Queue vs stream — the core distinction?
?
**Quick:** Queue = work done once then gone (SQS/RabbitMQ); stream = durable, replayable, ordered log many consumers read (Kafka).<br>
**Deeper:** A queue is consumed destructively: once a message is acknowledged it is removed, so each message is processed by one worker. A stream is an append-only log persisted on disk; consumers track their own offset (position), so multiple independent consumers can read the same data and you can replay from the past. Choose a queue for task distribution, a stream for event sourcing, multiple subscribers, or reprocessing history.

When do you add a message queue at all?
?
**Quick:** To decouple producer/consumer, absorb spikes (buffer), do async/background work, and smooth load on a slow downstream.<br>
**Deeper:** Decoupling means the producer doesn't call the consumer directly, so the two can scale, deploy, and fail independently. Buffering lets a queue absorb a traffic spike that the consumer couldn't handle in real time, draining it at a sustainable rate. This trades immediate consistency for resilience and throughput, which is why queues suit background jobs (emails, image processing) rather than synchronous request/response paths.

In Kafka, ordering is guaranteed across what scope?
?
**Quick:** Only within a single partition — chosen by the partition key. No global ordering.<br>
**Deeper:** A partition is one ordered, immutable shard of a topic's log; a topic is split into many partitions for parallelism. Kafka hashes the partition key to pick a partition, so all messages with the same key (e.g. a user ID) land in the same partition and stay ordered relative to each other. There is no ordering guarantee across partitions, so if you need a global order you must use a single partition and give up parallel throughput.

Most queues give at-least-once delivery. What must consumers implement?
?
**Quick:** Idempotency/dedup, so reprocessing a duplicate is harmless.<br>
**Deeper:** At-least-once delivery means a message may be delivered more than once (e.g. when an ack is lost and the broker redelivers), so duplicates are expected. Idempotency means applying the same operation twice yields the same result as applying it once. Consumers achieve this by tracking processed message IDs, using upserts, or making operations naturally idempotent — exactly-once is hard, so designing for safe retries is the pragmatic default.

What determines hot-partition risk in Kafka, and one fix?
?
**Quick:** The partition key; fix by adding a random/sub-key suffix and aggregating in two stages.<br>
**Deeper:** A hot partition occurs when a skewed key (e.g. one celebrity user, or a null/default key) routes a disproportionate share of traffic to a single partition, bottlenecking one consumer while others idle. Salting the key with a random suffix spreads that load across many partitions. You then do a two-stage aggregation — partial results per sub-key, then a final merge — to recover the per-key totals the salting broke up.

What is consumer-group rebalancing?
?
**Quick:** Partitions are redistributed across consumers when one joins/leaves; gives scale + fault tolerance but pauses consumption briefly.<br>
**Deeper:** A consumer group is a set of consumers that cooperatively read a topic, with each partition assigned to exactly one member so work is split without duplication. When a member is added, removed, or crashes, the group rebalances by reassigning partitions to the survivors, providing automatic scaling and fault tolerance. During the rebalance consumption stalls (a "stop-the-world" pause), so frequent rebalances hurt throughput and should be minimized.

What is the outbox pattern and what does it solve?
?
**Quick:** Write the event to an "outbox" table in the same DB transaction as the state change, then relay it — avoids the dual-write problem (DB committed but event lost).<br>
**Deeper:** The dual-write problem is when you must update two systems (your DB and a message broker) but can't do so atomically — a crash between them leaves them inconsistent. The outbox pattern sidesteps this: the event row is committed in the SAME transaction as the business change, so they succeed or fail together. A separate relay process (often via change-data-capture) then reads the outbox and publishes to the broker, retrying safely.

Backpressure — what is it and one way to handle it?
?
**Quick:** Consumer can't keep up with producer; handle via bounded buffers, rate limiting producers, or scaling/partitioning consumers.<br>
**Deeper:** Backpressure is the signal a system sends upstream when demand exceeds the rate it can process, preventing unbounded queue growth and out-of-memory crashes. A bounded buffer caps in-flight work and forces producers to slow or block once full; rate limiting throttles producers proactively. Scaling consumers (more workers, more partitions for parallelism) raises throughput so the consumer side can match producer load.

Dead-letter queue — what's it for?
?
**Quick:** Messages that repeatedly fail processing get routed aside for inspection/retry, so one poison message doesn't block the queue.<br>
**Deeper:** A dead-letter queue (DLQ) is a separate queue where messages are sent after exceeding a max-retries threshold. A poison message is one that consistently fails (malformed payload, bug) and, without a DLQ, would be retried forever or block head-of-line processing. Routing it aside keeps the main queue flowing while preserving the failed message for debugging, manual fixes, or later replay.
