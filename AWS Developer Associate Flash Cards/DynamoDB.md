---
tags:
  - flashcards/aws-dynamodb
---

# DynamoDB

Keys, capacity, indexes, streams, and the modeling tricks DVA-C02 loves to test.

---

What is the difference between partition key and sort key?
?
**Quick:** Partition key distributes items across partitions; sort key orders items within a partition.<br>
**Deeper:** Primary key is either partition key alone or partition+sort (composite). Two items can share a partition key only if they differ in sort key. Query operation requires the partition key and can filter on sort key with conditions like begins_with, between, >. Exam trick: Scan reads all items; Query is targeted and efficient — always prefer Query.

How do you calculate RCUs and WCUs?
?
**Quick:** 1 WCU = 1 write/sec up to 1 KB; 1 RCU = 1 strongly-consistent read/sec up to 4 KB (or 2 eventually-consistent).<br>
**Deeper:** Reads: round item size UP to nearest 4 KB; eventually consistent reads cost half the RCUs of strongly consistent. Transactional reads/writes cost 2x. Example: read a 6 KB item eventually consistent = ceil(6/4)/2 = 1 RCU. Writes: round UP to nearest 1 KB. Exam favorite: 10 strongly-consistent reads/sec of 8 KB items = 10 * (8/4) = 20 RCUs.

When should you use on-demand vs provisioned capacity?
?
**Quick:** On-demand for unpredictable/spiky traffic; provisioned for steady, predictable workloads.<br>
**Deeper:** On-demand auto-scales instantly, costs ~6-7x more per request than provisioned but no capacity planning. Provisioned is cheaper if you know your traffic, supports auto scaling, and reserved capacity for further discount. Exam trick: "new application with unknown traffic" = on-demand; "steady production workload, cost-optimized" = provisioned with auto scaling.

What is the difference between GSI and LSI?
?
**Quick:** GSI = different partition key, any time, eventually consistent; LSI = same partition key + different sort key, at table creation, strong consistency available.<br>
**Deeper:** LSI must be created with the table and is limited to 5 per table; shares table's partition. GSI can be added/removed anytime, has its own provisioned capacity, and only supports eventual consistency. Exam gotcha: LSI cannot be added after the table exists — if the question says "we already have a table," the answer is GSI.

What is DynamoDB Accelerator (DAX)?
?
**Quick:** Fully-managed, in-memory write-through cache for DynamoDB.<br>
**Deeper:** Microsecond reads, item cache (5 min default) and query cache (5 min default). Sits in your VPC. Compatible with DynamoDB API — minimal code change (replace client). Exam trick: DAX is for read-heavy workloads with the same items repeatedly. It does NOT help with write performance or with diverse, unique reads. Use ElastiCache for arbitrary caching beyond DynamoDB.

What are DynamoDB Streams and what do they capture?
?
**Quick:** Time-ordered log of item-level changes, retained 24 hours.<br>
**Deeper:** View types: KEYS_ONLY, NEW_IMAGE, OLD_IMAGE, NEW_AND_OLD_IMAGES. Lambda integrates as event source mapping, processes shards in order. Exam use cases: cross-region replication, analytics pipelines, triggering side effects. Kinesis Data Streams for DynamoDB is the alternative with 1-year retention and Kinesis tooling.

How does DynamoDB TTL work?
?
**Quick:** Auto-deletes items after a Unix epoch timestamp attribute expires, free of WCU cost.<br>
**Deeper:** Define a TTL attribute name; items typically deleted within 48 hours of expiration (not real-time). Deletes appear in Streams with userIdentity = dynamodb (good for archival to S3). Exam trick: TTL is best-effort delayed cleanup, not a precise timer. For exact expiration semantics, check timestamps in your application.

What are DynamoDB transactions and their cost?
?
**Quick:** ACID across up to 100 items / 4 MB; cost 2x normal RCU/WCU.<br>
**Deeper:** TransactWriteItems and TransactGetItems for all-or-nothing operations including across tables in the same region/account. Each item in a transactional read/write consumes 2 RCUs or 2 WCUs. Exam gotcha: transactions double cost — only use when you need atomicity. Conditional writes (single-item) are still cheaper than transactions.

What is the difference between strong and eventual consistency?
?
**Quick:** Strong returns latest write; eventual may return stale data but is half the cost and faster.<br>
**Deeper:** Default reads are eventually consistent. Pass ConsistentRead=true on GetItem/Query/Scan for strong consistency (costs 2x RCU). GSI reads are ALWAYS eventually consistent — no strong-consistent option. Exam trick: "GSI + strong consistency" is a wrong answer; choose LSI if you need strong consistency on alternate sort.

What causes DynamoDB throttling and how do you fix it?
?
**Quick:** Hot partitions or exceeding provisioned capacity; fix with better key design or higher capacity.<br>
**Deeper:** ProvisionedThroughputExceededException means a single partition is overwhelmed (3,000 RCU / 1,000 WCU per partition cap) or table capacity exhausted. Fixes: spread writes across more partition keys (add random suffix / write sharding), use exponential backoff with jitter, enable auto scaling, or switch to on-demand. Adaptive capacity helps but is not instant.

What is single-table design and why do experts recommend it?
?
**Quick:** Store multiple entity types in one DynamoDB table for efficient Query access.<br>
**Deeper:** Reduces costs (fewer tables to provision), enables single-query joins via shared partition key, and matches DynamoDB's access-pattern-first modeling. Uses generic PK/SK like PK="USER#123", SK="ORDER#456". Exam gotcha: this is a design pattern, not an AWS feature — but DVA-C02 sometimes tests "minimize cost and queries" scenarios where one table is the answer.

What are DynamoDB batch operation limits?
?
**Quick:** BatchGetItem: 100 items / 16 MB; BatchWriteItem: 25 items / 16 MB.<br>
**Deeper:** Batches are NOT atomic — partial failures return UnprocessedItems which you must retry. BatchWriteItem does not support UpdateItem or conditional writes (only PutItem and DeleteItem). Exam trick: if you need atomic multi-item operations, use Transactions, not Batch.

What are conditional writes used for?
?
**Quick:** Updates/puts/deletes that only succeed if a condition expression evaluates true.<br>
**Deeper:** Used for optimistic locking (attribute_exists, attribute_not_exists, version checks). Failed conditional write still consumes WCU. Exam pattern: "ensure no overwrite of existing item" = ConditionExpression "attribute_not_exists(pk)". Way cheaper than transactions for single-item atomicity.

What are DynamoDB item and attribute size limits?
?
**Quick:** Max item size 400 KB including attribute names and values.<br>
**Deeper:** Includes binary length after UTF-8 encoding. Exam gotcha: for larger blobs, store in S3 and put the S3 key in DynamoDB. Max partition key 2,048 bytes; sort key 1,024 bytes. Local secondary index: 10 GB per partition key value limit (GSI has no such limit).
