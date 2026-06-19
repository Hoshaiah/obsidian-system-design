---
tags:
  - flashcards/aws-messaging
---

# SQS, SNS & EventBridge

Queues, pub/sub, event buses, and which to choose when for DVA-C02.

---

What is the difference between SQS Standard and FIFO queues?
?
**Quick:** Standard = at-least-once + best-effort order, unlimited throughput; FIFO = exactly-once + strict order, 300/3000 TPS.<br>
**Deeper:** Standard guarantees at-least-once delivery (duplicates possible) and best-effort ordering. FIFO guarantees exactly-once processing (5-min dedup window) and strict ordering per MessageGroupId. FIFO throughput: 300 msg/s without batching, 3000 msg/s with batching, or 30,000 msg/s with high-throughput mode. Exam trick: FIFO queue names must end in .fifo.

What is the SQS visibility timeout?
?
**Quick:** The window during which a received message is hidden from other consumers.<br>
**Deeper:** Default 30 seconds, range 0 to 12 hours. If consumer doesn't delete the message before timeout, it reappears for other consumers. Too short = duplicates from slow consumers; too long = delayed retries after crashes. Exam best practice: set visibility timeout to ~6x your function timeout when Lambda consumes SQS; extend via ChangeMessageVisibility for long-running work.

What is the difference between long polling and short polling?
?
**Quick:** Long polling waits up to 20 seconds for messages; short polling returns immediately.<br>
**Deeper:** Short polling (default, WaitTimeSeconds=0) samples a subset of servers and may return empty even when messages exist. Long polling (WaitTimeSeconds 1-20) reduces empty responses and API costs. Exam trick: long polling reduces costs and CPU on consumers — always recommended.

What are SQS message retention and size limits?
?
**Quick:** 4 days default (1 min to 14 days); 256 KB max message size.<br>
**Deeper:** Use SQS Extended Client Library for larger payloads — stores body in S3 and queues a pointer. Exam gotcha: message gets deleted after retention expires even if never received. For larger payloads or streaming, use Kinesis or store the blob in S3.

How does SQS DLQ work?
?
**Quick:** A separate queue that captures messages exceeding the maxReceiveCount.<br>
**Deeper:** Source queue's redrive policy specifies the DLQ and maxReceiveCount (e.g., 5 = after 5 failed receives, message moves to DLQ). DLQ must be same type as source (Standard DLQ for Standard, FIFO for FIFO). Exam trick: alarm on DLQ ApproximateNumberOfMessagesVisible to catch poison messages. Use redrive to source to replay fixed messages.

How does SNS fanout pattern work?
?
**Quick:** Publish once to SNS topic; multiple SQS queues subscribed receive the message.<br>
**Deeper:** Each subscribed queue gets its own copy — independent processing, retries, DLQs per queue. Allows decoupling and parallel consumers. Exam trick: "S3 event needs to trigger 3 different processing pipelines" = S3 → SNS → SQS x3 (each pipeline owns its queue). SNS supports message filtering so each subscriber gets only relevant messages.

What protocols can subscribe to an SNS topic?
?
**Quick:** SQS, Lambda, HTTP/S, email, SMS, mobile push, Kinesis Data Firehose.<br>
**Deeper:** HTTP/S subscribers must confirm subscription (handshake). SMS has region/character/quota limits. Lambda subscribers get sync invokes with retries by SNS. Kinesis Data Firehose subscription lets SNS deliver to S3/Redshift/OpenSearch for analytics. Exam trick: "deliver SNS messages to a data lake" = subscribe Firehose, not Lambda batching.

What are SNS FIFO topics?
?
**Quick:** Ordered + dedup pub/sub, mirrors SQS FIFO, must subscribe to SQS FIFO queues.<br>
**Deeper:** Use for strict ordering across multiple consumers (fanout). Same 300/3000 TPS limits, .fifo suffix required. Subscribers can only be SQS FIFO queues (no Lambda direct, no HTTP). Exam gotcha: cannot mix FIFO and Standard between topic and subscriber.

What is EventBridge and how does it differ from SNS?
?
**Quick:** A serverless event bus with rich filtering, schemas, and many SaaS sources; SNS is simpler pub/sub.<br>
**Deeper:** EventBridge has multiple buses (default, custom, partner), rule-based routing with pattern matching on event JSON, 20+ targets (Lambda, SQS, SNS, Step Functions, Kinesis, etc.), built-in archive/replay, and schema registry. Higher latency (~0.5s) and lower throughput than SNS. Exam trick: "match events on JSON content with a complex filter" = EventBridge; "fanout to many SQS" = SNS.

What is an EventBridge rule pattern?
?
**Quick:** JSON pattern that matches events to route them to targets.<br>
**Deeper:** Example: {"source": ["aws.ec2"], "detail-type": ["EC2 Instance State-change Notification"], "detail": {"state": ["stopped"]}}. Supports exact match, prefix, anything-but, numeric ranges, etc. Up to 5 targets per rule. Schedule expressions (cron/rate) replace CloudWatch Events for scheduled invocations. Exam trick: schedule a Lambda every 5 min = EventBridge rate(5 minutes).

What are EventBridge schemas and the schema registry?
?
**Quick:** A catalog of event structures that generates code bindings.<br>
**Deeper:** Auto-discover schemas from events flowing through a bus, then download Java/Python/TypeScript bindings for type-safe handling. Useful for partner integrations and large microservice fleets. Exam trick: discovery turns runtime events into JSON Schema OpenAPI 3 specs.

How does EventBridge archive and replay work?
?
**Quick:** Archive stores events on a bus; replay re-sends them to targets later.<br>
**Deeper:** Configure archive retention (days to indefinite) and an optional filter pattern. Replay specifies time range — useful for testing new rules, recovering from downstream outages. Exam trick: "audit/recover ability for events" = EventBridge archive + replay; SNS has no equivalent.

When should you choose SQS vs SNS vs EventBridge?
?
**Quick:** SQS = point-to-point queue; SNS = simple pub/sub fanout; EventBridge = routing + filtering hub.<br>
**Deeper:** SQS when one consumer per message and you need decoupling/buffering. SNS when many consumers need the same message instantly with low latency. EventBridge when you need content-based routing, multiple sources (including SaaS), or schemas. Exam favorite: cost — SQS cheapest, SNS middle, EventBridge highest per million events but most features.
