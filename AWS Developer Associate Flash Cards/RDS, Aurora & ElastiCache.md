---
tags:
  - flashcards/aws-databases
---

# RDS, Aurora & ElastiCache

Relational databases, Aurora variants, and caching strategies for DVA-C02.

---

What is the difference between RDS Multi-AZ and read replicas?
?
**Quick:** Multi-AZ is sync standby for HA; read replicas are async copies for scaling reads.<br>
**Deeper:** Multi-AZ: synchronous standby in a different AZ, auto-failover (~60-120 sec), same DNS endpoint, not used for reads (except Multi-AZ Cluster which has 2 readable standbys). Read replicas: async replication, up to 15 (Aurora), used for read scaling, can be cross-region, can be promoted to standalone. Exam trick: "HA" = Multi-AZ; "scale reads" = read replicas.

What is the difference between Aurora Serverless v1 and v2?
?
**Quick:** v1 scales by pause/resume + ACU steps with cold start; v2 scales granularly with no pause.<br>
**Deeper:** v1: discrete ACU steps, can pause to 0 ACU (cold start when waking), supports MySQL 5.6 and PostgreSQL legacy. v2: smooth fractional ACU scaling (as low as 0.5 ACU), no pause to 0 (in production tier), supports recent engine versions, can act as Aurora cluster member. Exam trick: spiky unpredictable workloads = Serverless v2; pure dev/test that idles = v1.

What is Aurora Global Database?
?
**Quick:** Primary region + up to 5 secondary read-only regions with <1 sec replication lag.<br>
**Deeper:** Designed for low-latency global reads and disaster recovery — RPO ~1 sec, RTO ~1 min (managed planned failover). Secondary regions can have their own read replicas. Promote a secondary to primary for cross-region DR. Exam trick: "<5 second RPO across regions" = Aurora Global; cross-region RDS read replica is slower and async per-instance.

What is RDS Proxy?
?
**Quick:** Managed connection pooler that reduces database connection overhead and adds failover.<br>
**Deeper:** Especially valuable for Lambda — each cold start would otherwise open a fresh DB connection. Proxy pools and reuses connections, holds them across function executions. Supports IAM auth, integrates with Secrets Manager, faster failover (~66% reduction). Exam trick: "Lambda + RDS connection exhaustion" = RDS Proxy.

What are parameter groups and option groups?
?
**Quick:** Parameter groups configure engine-level settings; option groups enable engine-specific features.<br>
**Deeper:** Parameter groups: my.cnf-like values (max_connections, slow_query_log). Some need DB reboot to apply. Option groups: add features like Oracle TDE, SQL Server SSAS/SSRS, MariaDB audit plugin — not used for MySQL/PostgreSQL with most options. Exam trick: cannot modify default parameter groups — clone first.

What is the difference between RDS automated backups and snapshots?
?
**Quick:** Automated backups have retention window (1-35 days) and are deleted when DB is deleted; snapshots persist until manually deleted.<br>
**Deeper:** Automated daily snapshot + transaction logs = point-in-time recovery within retention. Manual snapshots are user-created and live until deleted. Both stored in S3. Exam trick: "keep backups after deleting the DB" = take a final snapshot; setting "delete automated backups: no" preserves the auto backups too.

What is the difference between ElastiCache Redis and Memcached?
?
**Quick:** Redis = rich features (HA, persistence, pub/sub, sorted sets); Memcached = simple cache with multithreading.<br>
**Deeper:** Redis: replication, automatic failover (with Multi-AZ), data persistence (RDB/AOF), data types (lists, sets, sorted sets, streams), pub/sub, geospatial, transactions, Lua scripts. Memcached: multi-threaded, sharded, no persistence, no replication, only simple key/value strings. Exam trick: "session store with HA" = Redis; "simple cache with multi-thread scaling" = Memcached.

What is Redis cluster mode enabled vs disabled?
?
**Quick:** Enabled = sharded across multiple shards (horizontal scale); disabled = single shard with replicas.<br>
**Deeper:** Cluster mode enabled: data partitioned across up to 500 shards, each with primary + replicas, scale write throughput. Cluster mode disabled: one shard with up to 5 replicas, all reads can use replicas, simpler. Exam trick: "scale writes beyond a single node" = cluster mode enabled.

What are write-through vs lazy loading caching strategies?
?
**Quick:** Write-through writes to cache AND DB on every write; lazy loading caches only on cache miss.<br>
**Deeper:** Lazy loading (cache aside): app checks cache → on miss, query DB and populate cache; stale data possible, simple. Write-through: every DB write also updates cache; cache always fresh but write latency higher and cache may hold unused data. Often combined with TTL. Exam trick: questions about "stale data risk" = lazy loading; "extra write latency" = write-through.

What are RDS storage types and limits?
?
**Quick:** General-Purpose SSD (gp2/gp3), Provisioned IOPS (io1/io2), Magnetic (legacy).<br>
**Deeper:** gp3 is the new default — better IOPS/throughput baseline than gp2. io1/io2 for high IOPS workloads ($/IOPS). Max storage: 64 TiB (MySQL/Postgres/MariaDB), 16 TiB (Oracle/SQL Server). Storage autoscaling automatically adds capacity. Exam trick: "switching from gp2 to gp3 saves cost while improving perf" — true for many workloads.

What is the RDS read replica replication lag?
?
**Quick:** Async per-engine (typically seconds); monitor with ReplicaLag CloudWatch metric.<br>
**Deeper:** MySQL/MariaDB/PostgreSQL replicas can be promoted to standalone DBs. Cross-region replicas have additional network latency. Lag spikes during heavy write bursts or large transactions. Exam trick: "read after write consistency" requires reading from primary, not replica.

What encryption options does RDS support?
?
**Quick:** KMS encryption at rest, SSL/TLS in transit; Oracle/SQL Server support TDE via options.<br>
**Deeper:** Encryption at rest must be enabled at creation — cannot enable later (workaround: snapshot, copy snapshot with encryption, restore). All replicas of an encrypted DB are encrypted. SSL via downloaded RDS CA bundle; force SSL via parameter group (rds.force_ssl=1 for Postgres). Exam trick: "encrypt existing unencrypted RDS" = snapshot → encrypted copy → restore.

What is Aurora's storage architecture?
?
**Quick:** Distributed shared 6-way replicated storage across 3 AZs, auto-scaling to 128 TiB.<br>
**Deeper:** Decouples compute from storage. Adding read replicas does not require duplicating storage (shared). Survives loss of 2 copies for writes, 3 for reads. Continuous backup to S3 with point-in-time recovery. Exam trick: Aurora replicas have <100 ms lag (vs RDS several seconds) and use shared storage so no replication cost.

What is the difference between RDS and Aurora?
?
**Quick:** RDS is multi-engine managed; Aurora is AWS-built MySQL/PostgreSQL-compatible with better perf/HA.<br>
**Deeper:** Aurora: ~3-5x faster, auto storage scaling, 6-way replication, up to 15 replicas with <100ms lag, Global Database, Serverless. RDS: supports MySQL, PostgreSQL, MariaDB, Oracle, SQL Server. Exam trick: "performance + AWS-native" = Aurora; "Oracle or SQL Server" = RDS only.
