---
tags:
  - flashcards/databases
---

# Databases & Storage

Pick the store, then justify it in one line. Most interviews need only a handful of these.

---

What's your default data store, and when do you move off it?
?
**Quick:** Start SQL (correctness, joins, flexible queries, transactions); move to NoSQL when a specific access pattern needs horizontal write throughput/scale SQL can't give.<br>
**Deeper:** Relational databases give you ACID transactions (atomic, consistent, isolated, durable) and ad-hoc joins, so they handle evolving query needs well. Vertical scaling (a bigger box) eventually caps out, so you shard or switch to NoSQL only when a known, high-volume access pattern demands horizontal write scaling. The trigger is a concrete bottleneck, not a vague "we might need scale."

When would you reach for Cassandra (wide-column)?
?
**Quick:** Write-heavy, horizontally scalable, time-series/event data, tunable (eventual) consistency, query patterns known up front.<br>
**Deeper:** Wide-column stores group columns into families and partition rows by a partition key across many nodes, so writes spread evenly and scale linearly. Cassandra is masterless (every node accepts writes) with tunable consistency, where you trade latency for how many replicas must acknowledge a read/write. You model tables around your queries in advance because ad-hoc joins and flexible querying are not supported.

When a document store (MongoDB/DynamoDB)?
?
**Quick:** Flexible/nested schema, single-entity access by key, fast iteration; weak at cross-entity joins.<br>
**Deeper:** Document stores keep self-contained JSON-like documents, so nested data lives together and is fetched in one read by its key. The schema-on-read model lets fields vary between documents, which suits rapidly changing apps. The cost is that relationships spanning documents require app-side joins or denormalization, since the engine won't join for you.

Where do blobs (images/video) live, and where does their metadata live?
?
**Quick:** Blob in object store (S3); metadata + URL in your DB. Never store large binaries in the primary DB.<br>
**Deeper:** A blob (binary large object) is unstructured data like an image or video; object stores such as S3 serve it cheaply over HTTP and scale independently. Storing binaries in the primary DB bloats rows, inflates backups, and wastes its buffer cache. Keep only the pointer (URL/key) plus searchable metadata in the DB, often using pre-signed URLs so clients upload/download directly.

What is a write-ahead log (WAL) and why does it matter?
?
**Quick:** Append-only log written before applying changes; gives durability + crash recovery, and is the basis for replication.<br>
**Deeper:** The WAL records each mutation to a sequential on-disk log before the change touches the main data pages, so a crash can be recovered by replaying the log. Sequential appends are far faster than random page writes, which also helps throughput. Replicas consume the same log stream to stay in sync, making the WAL the foundation of both durability and replication.

What's an LSM-tree and which DBs use it?
?
**Quick:** Log-structured merge tree: buffer writes in memory, flush sorted segments, compact later → fast writes. Cassandra, RocksDB, LevelDB.<br>
**Deeper:** Writes first land in an in-memory table (memtable), then get flushed to immutable on-disk sorted files (SSTables); background compaction merges them. This makes writes sequential and fast, at the cost of read amplification (a key may live across several segments, mitigated by Bloom filters). Contrast with B-trees, which update pages in place and favor reads.

B-tree vs LSM-tree in one line?
?
**Quick:** B-tree = read-optimized, in-place updates (most SQL); LSM = write-optimized, append + compact (write-heavy NoSQL).<br>
**Deeper:** A B-tree is a balanced, sorted tree of pages updated in place, giving predictable point and range reads, which is why most relational engines use it. An LSM-tree buffers writes and appends sorted segments, trading some read cost (multiple segments per key) for high write throughput. Choose by workload: read-heavy and transactional leans B-tree, write-heavy and append-mostly leans LSM.

What does an index cost you?
?
**Quick:** Faster reads on indexed columns, but slower writes and more storage — index only what you query.<br>
**Deeper:** An index is an auxiliary sorted structure (usually a B-tree) that lets the engine find rows without scanning the whole table. Every insert, update, or delete must also maintain each index, so writes get slower and disk usage grows. Add indexes for the columns you actually filter, sort, or join on, and drop ones the query planner never uses.

How do you support full-text search?
?
**Quick:** Elasticsearch / inverted index alongside the primary DB; sync via change stream/CDC. Don't full-text search in SQL at scale.<br>
**Deeper:** An inverted index maps each term to the list of documents containing it, enabling fast relevance-ranked text queries that SQL `LIKE '%...%'` scans can't do efficiently. You run a dedicated engine (Elasticsearch) beside the source-of-truth DB and keep it fresh with Change Data Capture (CDC), which streams the DB's row changes to downstream consumers. This keeps search fast while the primary DB stays authoritative.

What's a secondary index in a sharded DB and its catch?
?
**Quick:** An index on a non-shard-key field; querying it may require scatter-gather across all shards (slow).<br>
**Deeper:** Sharding partitions data by a shard key, so a query on that key hits exactly one shard. A secondary index covers a different field, so the matching rows can live on any shard, forcing a scatter-gather that fans the query out to every shard and merges results. Global secondary indexes can mitigate this but add write overhead and consistency lag.
