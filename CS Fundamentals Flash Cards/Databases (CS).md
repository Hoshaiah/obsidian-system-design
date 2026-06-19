---
tags:
  - flashcards/databases-cs
---

# Databases (CS Fundamentals)

The CS-level mental model of relational and NoSQL databases — indexes, transactions, isolation, and the tradeoffs.

---

What does ACID actually mean and what is the role of each property?
?
**Quick:** Atomicity (all-or-nothing), Consistency (constraints preserved), Isolation (concurrent txns don't interfere), Durability (committed data survives crashes).<br>
**Deeper:** Atomicity is implemented via undo logs or write-ahead logging (WAL). Consistency is application-defined and enforced by constraints, not by the DB alone. Isolation has levels (read uncommitted, committed, repeatable read, serializable) trading correctness for throughput. Durability requires fsync to disk; "eventually durable" replication semantics are a common source of data loss confusion.

How does a B-Tree index speed up queries, and what queries can't it accelerate?
?
**Quick:** B-Tree gives O(log n) point lookups and range scans on the indexed columns; can't accelerate non-prefix wildcards or arbitrary function results.<br>
**Deeper:** A B-Tree on (a, b, c) accelerates predicates on `a`, `a AND b`, `a AND b AND c`, and range scans on the last column used — but not predicates on just `b` or `c` alone (the leftmost prefix rule). `LIKE 'foo%'` works; `LIKE '%foo'` doesn't. Functional indexes (`CREATE INDEX ... ON lower(name)`) and partial indexes solve common edge cases.

What's the difference between a B-Tree index and a hash index?
?
**Quick:** B-Tree supports equality and range; hash supports only equality but with O(1) average lookup.<br>
**Deeper:** Postgres hash indexes are now WAL-logged and useful for very high-cardinality equality lookups. Most workloads stick with B-Tree because range scans, ORDER BY, and prefix matches are common. In-memory caches (Redis) use hash for O(1) but lose ordering; LSM-based DBs (RocksDB, Cassandra) use sorted structures for the same reasons B-Trees win on disk.

What are SQL isolation levels and which anomalies do they prevent?
?
**Quick:** Read Uncommitted (allows dirty reads), Read Committed (prevents dirty reads), Repeatable Read (prevents non-repeatable reads), Serializable (prevents phantoms).<br>
**Deeper:** Most production DBs default to Read Committed (Postgres, Oracle) or Repeatable Read (MySQL InnoDB). True Serializable is expensive (SSI in Postgres, 2PL elsewhere). Snapshot Isolation (used by many) prevents most anomalies but allows write skew — two transactions reading consistent snapshots and writing incompatible results. Knowing which anomalies your isolation level allows is critical for correctness.

What is database normalization and why might you denormalize?
?
**Quick:** Normalization decomposes tables to eliminate redundancy and update anomalies; denormalize for read performance.<br>
**Deeper:** 1NF (atomic columns), 2NF (no partial dependencies on composite key), 3NF (no transitive dependencies), BCNF (every determinant is a candidate key). Denormalization (materialized views, redundant columns, embedded documents) trades write complexity and storage for fewer joins and faster reads — common in analytics warehouses and read-heavy services. Document stores are essentially denormalized by design.

What's the difference between INNER, LEFT, RIGHT, and FULL OUTER joins?
?
**Quick:** INNER returns matching rows; LEFT/RIGHT include unmatched rows from one side filled with NULL; FULL OUTER includes unmatched from both.<br>
**Deeper:** Beneath the syntax, the query planner picks algorithms: nested loop (great when one side is small or indexed), hash join (no order needed, equality only), merge join (both sides sorted, supports inequality). Anti-joins and semi-joins (NOT EXISTS / EXISTS) are first-class operators in the planner. EXPLAIN ANALYZE is your friend.

How does MVCC work and what problem does it solve?
?
**Quick:** Multi-Version Concurrency Control gives each transaction a consistent snapshot via row versions, eliminating most read-write blocking.<br>
**Deeper:** Postgres and Oracle keep multiple versions of each row tagged with txn IDs; readers see the version visible at their snapshot, writers create new versions without blocking readers. Cost: bloat from dead tuples needing vacuum/compaction. InnoDB does similar via undo logs. MVCC enables snapshot isolation, the most common default in modern DBs.

When should you reach for NoSQL over a relational database?
?
**Quick:** When you need horizontal scale, flexible schema, or specific access patterns (KV, document, graph, wide-column) that hurt in SQL.<br>
**Deeper:** The honest answer is usually "you don't need NoSQL; you need a tuned Postgres." Real reasons: write throughput beyond a single primary (Cassandra, Dynamo), graph traversals (Neo4j), document-shaped data with no joins (MongoDB), or true global multi-master writes. Many "NoSQL" stories ended in migrating back to SQL when joins and transactions were re-implemented poorly at the app layer.

What is the CAP theorem and what does it actually constrain?
?
**Quick:** In a distributed system facing a network partition, you must choose between consistency and availability.<br>
**Deeper:** CAP is a partition-time choice, not always-on. "CP" systems (etcd, ZooKeeper, HBase) refuse writes during partitions to stay consistent. "AP" systems (Dynamo, Cassandra) accept writes everywhere and reconcile later (vector clocks, last-write-wins, CRDTs). PACELC extends CAP: even without partitions, you trade Latency vs Consistency.

What is a query plan and how do you read one?
?
**Quick:** A query plan is the tree of operators the DB will execute; read with EXPLAIN/EXPLAIN ANALYZE looking at cost, rows, and the chosen join strategy.<br>
**Deeper:** Look for full table scans on big tables (missing or unusable index), bad row estimates (stale stats — run ANALYZE), and join order surprises. The planner uses statistics + cost model; bad stats produce bad plans. Sometimes a hint or rewriting the query (CTE materialization, breaking up OR into UNION) is the fix. Plan stability matters in production.

What's the difference between OLTP and OLAP?
?
**Quick:** OLTP is many small read/write transactions (apps); OLAP is fewer, larger analytical queries over big aggregates (warehouses).<br>
**Deeper:** OLTP favors row-store, narrow indexes, tight latency (Postgres, MySQL). OLAP favors column-store, compression, and vectorized execution (BigQuery, Snowflake, ClickHouse). Hybrid (HTAP) systems try to do both. Most production designs use OLTP for the system of record and ETL/CDC to a separate OLAP store for analytics.

What is the N+1 query problem?
?
**Quick:** Fetching N parents then issuing one query per parent for children — N+1 round trips instead of 1 or 2.<br>
**Deeper:** Common in ORMs with lazy loading. Fix with eager loading (JOIN or IN-clause batching), DataLoader pattern (batch + cache per request), or explicit prefetch. The performance impact in distributed systems is brutal because every extra query adds latency, not just CPU. Always inspect the actual SQL your ORM emits in production-shaped tests.
