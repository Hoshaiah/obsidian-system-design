---
tags:
  - flashcards/dilemmas
---


The comparison calls interviewers love to probe. For each: when to pick each side, and the switch condition. Saying "X when…, Y when…, I'd switch if…" reads as senior.

---

## Data

SQL vs NoSQL — when each?
?
**Quick:** SQL for correctness, joins, flexible/unknown queries, transactions. NoSQL for horizontal write scale on a *known* access pattern. Switch to NoSQL when one access pattern outgrows a single SQL primary.<br>
**Deeper:** SQL (relational) databases enforce schemas, ACID transactions, and ad-hoc joins, making them ideal when relationships and correctness matter. NoSQL stores (key-value, document, wide-column) drop joins/strict schemas to scale writes horizontally across nodes, but optimize for one predictable access pattern. Choose SQL by default; switch to NoSQL once a single access pattern's write/throughput volume exceeds what one SQL leader can sustain.
<!--SR:!2026-06-14,1,230-->

Normalization vs denormalization — when each?
?
**Quick:** Normalize for write integrity + no duplication (OLTP). Denormalize to avoid expensive joins on hot read paths. Switch to denorm when reads dominate and joins are the bottleneck.<br>
**Deeper:** Normalization splits data into related tables so each fact lives once, preventing update anomalies — great for write-heavy transactional systems. Denormalization deliberately duplicates data (precomputed/joined rows) so reads avoid multi-table joins. Start normalized; denormalize specific read paths when reads vastly outnumber writes and join cost becomes the latency bottleneck.
<!--SR:!2026-06-14,1,230-->

Strong vs eventual consistency — when each?
?
**Quick:** Strong for money, inventory, auth, uniqueness. Eventual for feeds, likes, view counts, analytics. Decide *per feature*, not per system.<br>
**Deeper:** Strong consistency guarantees every read sees the most recent write (linearizable), at the cost of latency and availability during partitions. Eventual consistency lets replicas diverge briefly and converge later, buying availability and scale. Apply strong consistency only where correctness is non-negotiable (payments, stock, identity) and accept eventual elsewhere — the choice is made feature by feature.
<!--SR:!2026-06-14,1,230-->

Optimistic vs pessimistic locking — when each?
?
**Quick:** Optimistic (version/CAS) under low contention — cheap, retries rare. Pessimistic (locks) under high contention where retries would thrash.<br>
**Deeper:** Optimistic locking assumes conflicts are rare: it reads a version number and only commits if it's unchanged (compare-and-swap), retrying on conflict. Pessimistic locking acquires an exclusive lock up front so others must wait, preventing conflicts but blocking throughput. Use optimistic when contention is low so retries are rare; switch to pessimistic when contention is high enough that constant retries would waste more work than the locks.
<!--SR:!2026-06-14,1,230-->

Blob store vs DB for files — when each?
?
**Quick:** Object store (S3) for the bytes, always at scale; DB only for the metadata + URL. Never put large binaries in the primary DB.<br>
**Deeper:** An object/blob store (S3, GCS) is purpose-built for cheap, durable, scalable storage of large binary files served directly or via CDN. A relational DB is for structured queryable records — storing large BLOBs bloats it, slows backups, and wastes expensive storage. The standard pattern: keep file bytes in the object store and store only metadata plus the object URL/key in the DB.
<!--SR:!2026-06-14,1,230-->

## Replication & Scale

Replication vs sharding — what does each buy?
?
**Quick:** Replication scales *reads* + adds redundancy/failover. Sharding scales *writes* + storage. They're complementary — most big systems do both.<br>
**Deeper:** Replication copies the same data to multiple nodes, so reads can fan out and a failed node has standbys for failover. Sharding partitions different data across nodes by a shard key, so each node owns a slice of the writes and storage. They solve different bottlenecks — read scale/HA vs write/storage scale — so large systems typically shard and then replicate each shard.
<!--SR:!2026-06-16,3,250-->

Vertical vs horizontal scaling — when each?
?
**Quick:** Vertical first (simplest) until cost/ceiling/SPOF bites. Horizontal when you need beyond one box or HA — accept the coordination/consistency cost.<br>
**Deeper:** Vertical scaling adds CPU/RAM/disk to a single machine — no app changes, but bounded by the largest box and still a single point of failure. Horizontal scaling adds more machines, giving near-unlimited capacity and redundancy at the price of coordination, data partitioning, and consistency challenges. Scale up first for simplicity; switch to scaling out when you hit the single-box ceiling/cost or need high availability.
<!--SR:!2026-06-14,1,230-->

Leader-follower vs leaderless (quorum) — when each?
?
**Quick:** Leader-follower for simple strong-ish writes + read scaling (most SQL). Leaderless (Dynamo/Cassandra, R+W>N) for high availability + write scale, tolerating tunable consistency.<br>
**Deeper:** Leader-follower routes all writes through one leader that replicates to followers, giving a simple write path and read scaling but a write SPOF and failover lag. Leaderless replication lets any replica accept writes and uses quorum reads/writes (R+W>N) to stay consistent, maximizing availability and write throughput. Choose leader-follower for straightforward, mostly-strong workloads; choose leaderless when you need always-on availability and can tune consistency per query.
<!--SR:!2026-06-14,1,230-->

Consistent hashing vs modulo hashing — why prefer consistent?
?
**Quick:** Modulo rehashes ~everything when node count changes. Consistent hashing moves only ~1/N of keys — pick it whenever the node set changes dynamically.<br>
**Deeper:** Modulo hashing maps a key with `hash(key) % N`, so changing N (adding/removing a node) remaps nearly every key, causing a massive reshuffle. Consistent hashing places nodes and keys on a ring so adding/removing a node only relocates the keys between adjacent points (~1/N). Prefer consistent hashing whenever membership changes dynamically — caches, sharded stores, distributed systems with scaling/failures.
<!--SR:!2026-06-16,3,250-->

## Async & Delivery

Queue vs stream (SQS vs Kafka) — when each?
?
**Quick:** Queue for one-time task processing, simple decoupling, per-message ack. Stream for durable, replayable, ordered, multi-consumer event logs + analytics.<br>
**Deeper:** A queue (SQS) delivers each message to one consumer, which acks and removes it — ideal for distributing tasks and decoupling producers from workers. A stream/log (Kafka) is an append-only, retained, ordered log that many independent consumer groups can read and replay from their own offsets. Use a queue for transient work distribution; use a stream when you need durable history, replay, ordering, or multiple downstream consumers.
<!--SR:!2026-06-14,1,230-->

Fan-out on write vs read (feeds) — the tradeoff?
?
**Quick:** Write (push): cheap reads, expensive writes, breaks for celebrities. Read (pull): cheap writes, expensive reads. Hybrid: push for normal users, pull for celebrities, merge at read.<br>
**Deeper:** Fan-out on write pushes a new post into every follower's precomputed feed at write time — reads are a simple lookup but writes explode for users with millions of followers. Fan-out on read assembles a feed by querying authors at read time — writes are trivial but reads are expensive. The hybrid pushes for normal-sized accounts and pulls for celebrities, merging the two at read time to bound both costs.
<!--SR:!2026-06-14,1,230-->

Saga vs 2PC — why prefer saga?
?
**Quick:** Saga = compensating actions, non-blocking, scales across services. 2PC blocks participants and has a coordinator SPOF. Prefer saga for microservices.<br>
**Deeper:** A saga breaks a distributed transaction into local steps, each with a compensating action to undo it if a later step fails — no locks held across services, so it scales. Two-phase commit (2PC) has a coordinator that locks all participants through prepare/commit phases, blocking resources and stalling everyone if the coordinator dies. In a microservices world prefer sagas; reserve 2PC for tightly-coupled systems that need atomic guarantees and can tolerate blocking.
<!--SR:!2026-06-14,1,230-->

Sync vs async processing — when each?
?
**Quick:** Sync when the caller needs the result now and it's fast. Async (queue) for slow/spiky/retryable work or fan-out — return a job id and process in background.<br>
**Deeper:** Synchronous processing makes the caller wait for the result inline — simple and correct when the work is fast and the answer is needed immediately. Asynchronous processing accepts the request, returns a job id, and does the work via a queue/worker — absorbing spikes, enabling retries, and decoupling slow tasks. Go sync for fast must-have-now results; go async for slow, bursty, retryable, or fan-out work.
<!--SR:!2026-06-14,1,230-->

## Transport & APIs

REST vs gRPC vs GraphQL — when each?
?
**Quick:** REST for public, cacheable, simple CRUD. gRPC for fast internal service-to-service (binary, streaming). GraphQL when clients need flexible field selection / to avoid over/under-fetching.<br>
**Deeper:** REST uses HTTP verbs over JSON resources — ubiquitous, cacheable, and easy for public APIs. gRPC uses Protobuf over HTTP/2 for compact, fast, strongly-typed RPC with streaming, ideal between internal services. GraphQL exposes a single endpoint where clients specify exactly which fields they need, eliminating over/under-fetching but adding server complexity. Pick REST for public CRUD, gRPC for high-performance internal calls, GraphQL when diverse clients need flexible, precise queries.
<!--SR:!2026-06-14,1,230-->

Long polling vs WebSocket vs SSE — when each?
?
**Quick:** Long polling for simple/occasional updates. WebSocket for bidirectional real-time (chat, games). SSE for one-way server→client streams (feeds, notifications).<br>
**Deeper:** Long polling holds an HTTP request open until data arrives, then reconnects — simple and firewall-friendly but inefficient for frequent updates. WebSocket opens a persistent full-duplex TCP connection for low-latency two-way traffic. Server-Sent Events (SSE) is a one-way HTTP stream from server to client with auto-reconnect. Use long polling for occasional updates, WebSocket when both sides push constantly, SSE for server-driven one-way streams.
<!--SR:!2026-06-14,1,230-->

Offset vs cursor pagination — why cursor?
?
**Quick:** Cursor is stable under inserts/deletes and O(1)-ish; offset shifts results and gets slow at high offsets. Use cursor for feeds/infinite scroll.<br>
**Deeper:** Offset pagination uses `LIMIT/OFFSET`, which must scan and skip rows (slow at large offsets) and shifts every page when rows are inserted/deleted. Cursor pagination passes a pointer (e.g. the last seen id/timestamp) and seeks directly past it, giving stable, fast results regardless of inserts. Prefer cursor for large or live datasets like feeds and infinite scroll; offset is fine only for small, static, page-numbered lists.
<!--SR:!2026-06-14,1,230-->

## Infra

L4 vs L7 load balancer — when each?
?
**Quick:** L4 (TCP) for raw speed and protocol-agnostic routing. L7 (HTTP) when you need content-based routing, TLS termination, or path/host rules.<br>
**Deeper:** An L4 load balancer routes by IP/port at the transport layer without inspecting payloads — extremely fast and protocol-agnostic. An L7 load balancer parses HTTP, enabling routing by path/host/header, TLS termination, and request-aware features at higher CPU cost. Use L4 for raw throughput or non-HTTP protocols; use L7 when you need content-based routing or HTTP-aware features.
<!--SR:!2026-06-14,1,230-->

Monolith vs microservices — the real tradeoff?
?
**Quick:** Monolith: simpler deploy/debug, fine until org/scale forces splits. Microservices: independent scaling/teams, at the cost of network calls, distributed-txn pain, and ops overhead. Don't start micro without a reason.<br>
**Deeper:** A monolith is one deployable unit — simple to develop, test, and debug, with in-process calls and easy transactions, but it scales as a whole and can bottleneck large teams. Microservices split the system into independently deployable services, enabling per-service scaling and team autonomy, but introduce network latency, distributed transactions, and operational complexity. Start with a monolith and split into services only when org size or scaling needs justify the overhead.
<!--SR:!2026-06-14,1,230-->

CDN vs cache — what's the difference in role?
?
**Quick:** CDN caches static/edge content geographically near users. App cache (Redis) holds dynamic/hot data near the service. Use both: CDN for assets, Redis for query results.<br>
**Deeper:** A CDN is a network of geographically distributed edge servers that cache static assets close to users to cut latency and offload origin. An application cache (Redis/Memcached) sits near your services and holds hot dynamic data like query results or sessions in memory. They serve different layers — CDN at the edge for assets, app cache in the data center for dynamic data — so production systems use both.
<!--SR:!2026-06-14,1,230-->

---

## Data Modeling & Storage (essentials)

ACID vs BASE — what mindset does each represent?
?
**Quick:** ACID = strict correctness, transactions, immediate consistency (SQL). BASE = Basically Available, Soft state, Eventual consistency — trade strictness for availability + scale (NoSQL).<br>
**Deeper:** ACID (Atomicity, Consistency, Isolation, Durability) is the relational mindset: transactions either fully succeed or roll back, with immediate consistency. BASE (Basically Available, Soft state, Eventual consistency) is the NoSQL mindset: stay available and scalable by letting state be temporarily inconsistent and converge later. Choose ACID when correctness must be guaranteed at write time; BASE when availability and horizontal scale outweigh immediate consistency.
<!--SR:!2026-06-14,1,230-->

OLTP vs OLAP — when each?
?
**Quick:** OLTP = many small fast reads/writes, transactional (your app DB). OLAP = few huge analytical scans/aggregations (warehouse). Don't run heavy analytics on your OLTP store — replicate to OLAP.<br>
**Deeper:** OLTP (Online Transaction Processing) handles high volumes of small, fast, concurrent transactions — the live application database. OLAP (Online Analytical Processing) runs a few large, complex scans and aggregations over historical data — the analytics warehouse. Keep them separate: replicate or ETL OLTP data into an OLAP store so heavy analytical queries don't degrade transactional performance.
<!--SR:!2026-06-14,1,230-->

Row store vs column store — when each?
?
**Quick:** Row store for transactional point reads/writes of whole records (OLTP). Column store for analytics scanning a few columns over many rows — better compression + aggregation (OLAP).<br>
**Deeper:** A row store keeps each record's fields contiguously, so reading or writing a whole record is cheap — ideal for transactional point access. A column store keeps each column contiguously, so scanning a few columns across millions of rows is fast and compresses well — ideal for analytics. Use row stores for OLTP record-level access; column stores for OLAP scans and aggregations.
<!--SR:!2026-06-14,1,230-->

Single-leader vs multi-leader replication — when each?
?
**Quick:** Single-leader: one write path, simple, no write conflicts (most apps). Multi-leader: writes in multiple regions for latency/availability, but you must resolve write conflicts.<br>
**Deeper:** Single-leader replication funnels all writes through one node that streams to followers — simple, with no write conflicts, but the leader is a write bottleneck/SPOF. Multi-leader replication accepts writes at several leaders (often one per region) for lower write latency and regional availability, but concurrent writes to the same data create conflicts you must resolve. Default to single-leader; adopt multi-leader when you genuinely need multi-region write locality and can handle conflict resolution.
<!--SR:!2026-06-14,1,230-->

Synchronous vs asynchronous replication — the tradeoff?
?
**Quick:** Sync: follower confirms before commit → no data loss on failover, but higher latency. Async: commit immediately → fast, but a crashed leader can lose un-replicated writes.<br>
**Deeper:** Synchronous replication waits for a follower to acknowledge each write before the leader commits, guaranteeing the data survives a leader failure at the cost of added write latency. Asynchronous replication commits on the leader and ships changes to followers in the background — fast, but writes not yet replicated are lost if the leader crashes. Choose sync when zero data loss is required; async when latency/throughput matters more than a small failover risk.
<!--SR:!2026-06-14,1,230-->

Hash sharding vs range sharding — when each?
?
**Quick:** Hash: even distribution, no hot spots, but range queries scatter. Range: efficient range scans + ordering, but risks hot shards on sequential keys.<br>
**Deeper:** Hash sharding assigns rows by `hash(key)`, spreading load evenly but scattering any range query across all shards. Range sharding assigns contiguous key ranges to each shard, making range scans and ordering cheap but risking a hot shard when keys are sequential (e.g. timestamps all hitting the newest shard). Pick hash for uniform point-lookup load, range when you query by ranges and can avoid monotonic keys.
<!--SR:!2026-06-16,3,250-->

Schema-on-write vs schema-on-read — when each?
?
**Quick:** Schema-on-write (SQL): validate structure up front, reliable queries. Schema-on-read (data lake): store raw, interpret later — flexible for evolving/unknown data.<br>
**Deeper:** Schema-on-write enforces a defined structure when data is inserted, so queries are reliable and type-safe but ingestion is rigid. Schema-on-read stores raw data as-is and applies structure only when reading it, giving flexibility for evolving or unknown formats at the cost of query-time parsing and weaker guarantees. Use schema-on-write for well-understood transactional data; schema-on-read for raw lakes and exploratory or fast-changing data.
<!--SR:!2026-06-14,1,230-->

Materialized view vs compute-on-read — the tradeoff?
?
**Quick:** Materialized view precomputes + stores results → fast reads, stale until refreshed, extra storage. Compute-on-read is always fresh but pays cost every query. Precompute hot, expensive queries.<br>
**Deeper:** A materialized view precomputes a query's result and persists it, so reads are instant but the data is stale until the next refresh and uses extra storage. Compute-on-read runs the query live every time — always fresh but paying the full cost on each request. Materialize hot, expensive queries whose staleness is tolerable; compute on read for cheap queries or when absolute freshness is required.
<!--SR:!2026-06-14,1,230-->

Auto-increment vs UUID vs Snowflake ID — when each?
?
**Quick:** Auto-increment: compact, ordered, but reveals counts + needs a single source (bad for sharding). UUID: globally unique, generate anywhere, but random (index bloat). Snowflake: unique + roughly time-ordered + distributed — best of both.<br>
**Deeper:** Auto-increment IDs are small and sequential but require a central counter and leak record counts, making them poor for distributed/sharded writes. UUIDs are 128-bit values generable anywhere with no coordination, but their randomness scatters index inserts and bloats indexes. Snowflake IDs encode a timestamp plus machine/sequence bits, giving distributed generation, uniqueness, and rough time-ordering — use them when you need decentralized, sortable unique IDs at scale.
<!--SR:!2026-06-14,1,230-->

Event sourcing vs state-oriented storage — the tradeoff?
?
**Quick:** Event sourcing stores the log of changes (full audit, replay, time-travel) but is complex to query. State storage keeps only current state — simple, but history is lost.<br>
**Deeper:** Event sourcing persists an append-only log of every change and derives current state by replaying events, giving full audit history, time-travel, and replay at the cost of complex querying and rebuild logic. State-oriented storage saves only the latest value, which is simple to read and query but discards how the data got there. Use event sourcing when history/audit/replay is core; use state storage when only the current value matters.
<!--SR:!2026-06-14,1,230-->

CQRS vs single model — when each?
?
**Quick:** CQRS splits read and write models (optimize/scale each independently) — use when read and write loads/shapes diverge sharply. Single model otherwise; CQRS adds sync complexity.<br>
**Deeper:** CQRS (Command Query Responsibility Segregation) uses separate models for writes (commands) and reads (queries), each independently optimized and scaled, but you must keep the read side synced with the write side. A single model serves both reads and writes with one schema — simpler and consistent by construction. Adopt CQRS only when read and write workloads or shapes diverge sharply enough to justify the sync overhead.
<!--SR:!2026-06-14,1,230-->

Data warehouse vs data lake — the difference?
?
**Quick:** Warehouse: structured, schema-on-write, query-ready (BI/SQL). Lake: raw, any format, schema-on-read, cheap bulk storage (ML/exploration). Many use both (lakehouse).<br>
**Deeper:** A data warehouse stores cleaned, structured data with a defined schema, optimized for fast BI/SQL analytics. A data lake stores raw data in any format cheaply, applying structure only at read time, ideal for ML and exploration. Warehouses trade ingestion flexibility for query readiness; lakes trade query convenience for cheap flexible storage — and the lakehouse pattern blends both.
<!--SR:!2026-06-14,1,230-->

## Caching & Read Path (essentials)

Cache-aside vs read-through — the difference?
?
**Quick:** Cache-aside: app checks cache, on miss loads DB + populates (app owns logic). Read-through: cache itself loads from DB on miss (library/provider owns it). Cache-aside is the common interview default.<br>
**Deeper:** In cache-aside, the application checks the cache, and on a miss it reads the DB and writes the value back into the cache itself — the app controls the caching logic. In read-through, the cache layer sits in front of the DB and transparently fetches and stores on a miss, so the app just queries the cache. Cache-aside is the flexible, common default; read-through centralizes the logic when the cache provider supports it.
<!--SR:!2026-06-14,1,230-->

Write-around vs write-back caching — when each?
?
**Quick:** Write-around: write straight to DB, skip cache (avoids polluting cache with write-once data). Write-back: write cache now, flush to DB later (fast, risk of loss on crash).<br>
**Deeper:** Write-around writes new data directly to the DB and bypasses the cache, avoiding caching data that may never be read again (it's cached only on a later read). Write-back writes to the cache immediately and asynchronously flushes to the DB later, giving very fast writes but risking data loss if the cache crashes before the flush. Use write-around for write-once/rarely-read data; write-back for write-heavy workloads that can tolerate some durability risk.
<!--SR:!2026-06-14,1,230-->

TTL expiry vs explicit invalidation — the tradeoff?
?
**Quick:** TTL: simple, self-healing, but serves stale data up to the TTL. Explicit invalidation: always fresh, but you must reliably catch every write path (easy to miss one).<br>
**Deeper:** TTL expiry sets a lifetime on cache entries so they auto-expire — simple and self-healing, but readers may see stale data until the TTL lapses. Explicit invalidation deletes/updates the cache entry on every write, keeping data fresh but requiring you to instrument every code path that mutates the data, and a single missed path serves stale data indefinitely. Use TTL when bounded staleness is acceptable; explicit invalidation when freshness is critical and write paths are controlled.
<!--SR:!2026-06-14,1,230-->

Local (in-process) cache vs distributed cache — when each?
?
**Quick:** Local: nanosecond access, no network, but per-instance + inconsistent across nodes. Distributed (Redis): shared + consistent, but a network hop. Often layer both (L1 local, L2 Redis).<br>
**Deeper:** A local in-process cache lives in the app's memory — fastest possible access with no network hop, but each instance has its own copy that can diverge across nodes. A distributed cache (Redis) is a shared store all instances read, giving a single consistent view at the cost of a network round-trip. Use local for ultra-hot read-mostly data, distributed for shared state — and often layer them (L1 local, L2 Redis).
<!--SR:!2026-06-14,1,230-->

Bloom filter vs direct lookup — what does the filter buy?
?
**Quick:** A Bloom filter answers "definitely not present / maybe present" in tiny space → skip expensive disk/DB lookups for absent keys. No false negatives, some false positives.<br>
**Deeper:** A Bloom filter is a compact probabilistic bit-array that tests set membership: it can say "definitely not present" or "maybe present," never giving false negatives but allowing some false positives. A direct lookup hits the actual store every time, which is correct but expensive for keys that don't exist. Put a Bloom filter in front to cheaply short-circuit lookups for absent keys, falling back to the real store only when the filter says "maybe."
<!--SR:!2026-06-14,1,230-->

## Communication & APIs (essentials)

Push vs pull — when each?
?
**Quick:** Push: server sends on change → low latency, but wasteful if clients idle + needs connection state. Pull: client polls → simple + scalable, but latency + wasted empty polls.<br>
**Deeper:** In a push model the server proactively sends data the moment it changes, minimizing latency but requiring it to track client connections and risking wasted sends to idle clients. In a pull model clients poll the server on an interval — simple and stateless to scale, but updates lag the poll interval and many polls return nothing. Push when low latency and active clients matter; pull when simplicity and statelessness win and some delay is fine.
<!--SR:!2026-06-14,1,230-->

Request-response vs event-driven — when each?
?
**Quick:** Request-response when caller needs an answer now and coupling is fine. Event-driven (emit events, consumers react) for decoupling, fan-out, and async workflows.<br>
**Deeper:** Request-response is synchronous: the caller sends a request and blocks for a direct reply, which is simple but couples the caller to the callee's availability. Event-driven has producers emit events that any number of consumers react to independently, enabling decoupling, fan-out, and asynchronous workflows at the cost of harder end-to-end tracing. Use request-response when you need an immediate answer; event-driven for decoupled, multi-consumer, async processing.
<!--SR:!2026-06-14,1,230-->

Orchestration vs choreography — the difference?
?
**Quick:** Orchestration: a central coordinator drives the steps (visible, easier to debug, but a coupling/SPOF point). Choreography: services react to each other's events (decoupled, but flow is implicit + hard to trace).<br>
**Deeper:** In orchestration a central coordinator explicitly invokes each step in a workflow, making the flow visible and easy to debug but creating a coupling point and potential bottleneck/SPOF. In choreography each service reacts to events emitted by others with no central controller, maximizing decoupling but leaving the overall flow implicit and hard to trace. Use orchestration for complex flows needing visibility/control; choreography for loosely-coupled, autonomous services.
<!--SR:!2026-06-14,1,230-->

API gateway vs direct-to-service — what does the gateway add?
?
**Quick:** A single entry point for auth, rate limiting, routing, TLS, aggregation — at the cost of one more hop + a component to scale. Direct calls are simpler internally but push cross-cutting concerns into each service.<br>
**Deeper:** An API gateway is a single front door that centralizes cross-cutting concerns — authentication, rate limiting, routing, TLS termination, and response aggregation — but adds a hop and a component you must scale and keep highly available. Direct-to-service calls skip that hop and are simpler internally, but each service must then implement those concerns itself. Use a gateway when you want centralized policy and a unified client surface; direct calls for simple internal meshes.
<!--SR:!2026-06-16,3,250-->

Synchronous RPC vs async messaging — when each?
?
**Quick:** Sync RPC for immediate results + simple flow, but caller is coupled to callee availability/latency. Async messaging for decoupling, buffering spikes, and retryable work.<br>
**Deeper:** Synchronous RPC has the caller invoke a remote method and wait for the result — simple and immediate, but the caller's latency and success depend on the callee being up and fast. Async messaging sends the request to a broker/queue and processes it later, decoupling caller from callee, absorbing traffic spikes, and enabling retries. Use sync RPC when you need the answer now; async messaging for decoupling, buffering, and resilient background work.
<!--SR:!2026-06-14,1,230-->

Client-side vs server-side load balancing — the difference?
?
**Quick:** Client-side: caller picks an instance from a registry (no extra hop, but logic in every client). Server-side: a dedicated LB fronts the pool (centralized, language-agnostic, one more hop).<br>
**Deeper:** Client-side load balancing has the caller fetch the instance list from a service registry and pick one itself, avoiding an extra hop but embedding balancing logic in every client/language. Server-side load balancing puts a dedicated LB in front of the pool, centralizing the logic and keeping clients dumb at the cost of one more network hop. Choose client-side for low-latency internal meshes; server-side for centralized, language-agnostic control.
<!--SR:!2026-06-14,1,230-->

Service mesh vs in-app libraries — the tradeoff?
?
**Quick:** Mesh (sidecar) handles retries/mTLS/observability outside the app → language-agnostic, ops-managed, but infra complexity. Libraries are simpler to start but must be reimplemented per language.<br>
**Deeper:** A service mesh runs a sidecar proxy alongside each service to handle retries, mTLS, and observability transparently, so the logic is language-agnostic and centrally managed but adds significant infrastructure complexity. In-app libraries bake those same concerns into the application code — simpler to start with but must be reimplemented and kept consistent across every language and service. Use a mesh for large polyglot fleets; libraries for small or single-language systems.
<!--SR:!2026-06-16,3,250-->

Webhooks vs polling — when each?
?
**Quick:** Webhooks: provider pushes on event → real-time, efficient, but you must expose + secure an endpoint and handle retries/dedup. Polling: you pull on a schedule → simple, but latency + wasted calls.<br>
**Deeper:** Webhooks have the provider POST to your endpoint the moment an event occurs — real-time and efficient, but you must expose and secure a public endpoint and handle retries, duplicates, and ordering. Polling has you query the provider on a schedule — simple and fully under your control, but updates lag the interval and most polls return nothing. Use webhooks for timely, high-volume events; polling for simplicity or when you can't host an endpoint.
<!--SR:!2026-06-15,2,230-->

Backend-for-frontend vs one shared API — when each?
?
**Quick:** BFF: a tailored API per client (web/mobile) → optimal payloads, but more services to maintain. Shared API: one surface, simpler, but clients over/under-fetch.<br>
**Deeper:** A backend-for-frontend (BFF) provides a dedicated API per client type (web, mobile, etc.), tailoring payloads and aggregation to each — optimal but multiplying services to build and maintain. A single shared API exposes one surface to all clients, which is simpler to run but forces clients to over- or under-fetch since the shape can't suit everyone. Use a BFF when client needs diverge significantly; a shared API when they're similar enough.
<!--SR:!2026-06-14,1,230-->

Stateful vs stateless services — why prefer stateless?
?
**Quick:** Stateless: any instance serves any request → trivial horizontal scaling + failover (push state to Redis/DB). Stateful only when in-memory locality is essential (and then you need sticky routing + replication).<br>
**Deeper:** A stateless service keeps no per-client state in memory, so any instance can serve any request — making horizontal scaling, load balancing, and failover trivial (session/state is externalized to Redis/DB). A stateful service holds state locally, requiring sticky routing and replication to survive restarts and rebalancing. Prefer stateless and externalize state; go stateful only when in-memory locality (e.g. for performance) is genuinely essential.
<!--SR:!2026-06-14,1,230-->

## Networking & Transport (essentials)

TCP vs UDP — when each?
?
**Quick:** TCP: reliable, ordered, connection-based (web, APIs, DB). UDP: fast, connectionless, lossy (video/voice, gaming, DNS) where speed beats guaranteed delivery.<br>
**Deeper:** TCP is a connection-oriented protocol that guarantees ordered, reliable, retransmitted delivery via handshakes and acks — used for web, APIs, and databases. UDP is connectionless and fire-and-forget: no ordering, retries, or delivery guarantee, but minimal overhead and latency — used for video/voice, gaming, and DNS. Choose TCP when correctness and ordering matter; UDP when low latency matters more than occasional loss.
<!--SR:!2026-06-14,1,230-->

HTTP/1.1 vs HTTP/2 vs HTTP/3 — the progression?
?
**Quick:** 1.1: one in-flight request per connection (head-of-line blocking). 2: multiplexed streams over one TCP conn + header compression. 3: same over QUIC/UDP, removing TCP head-of-line blocking.<br>
**Deeper:** HTTP/1.1 allows only one outstanding request per connection, so a slow response blocks those behind it (head-of-line blocking) and clients open many connections. HTTP/2 multiplexes many concurrent streams over a single TCP connection with header compression, but a lost TCP packet still stalls all streams. HTTP/3 runs over QUIC (on UDP), giving independent streams so packet loss only stalls its own stream — eliminating TCP-level head-of-line blocking.
<!--SR:!2026-06-14,1,230-->

Forward proxy vs reverse proxy — the difference?
?
**Quick:** Forward proxy sits in front of *clients* (egress control, anonymity). Reverse proxy sits in front of *servers* (LB, TLS, caching, hides backends). Interviews usually mean reverse.<br>
**Deeper:** A forward proxy sits between clients and the internet, acting on behalf of clients for egress filtering, caching, or anonymity. A reverse proxy sits in front of servers, acting on their behalf for load balancing, TLS termination, caching, and hiding the backend topology. When system-design interviews say "proxy" they usually mean a reverse proxy fronting your services.
<!--SR:!2026-06-16,3,250-->

Pull CDN vs push CDN — when each?
?
**Quick:** Pull: CDN fetches from origin on first miss, then caches → easy, self-managing, good for large catalogs. Push: you upload to the CDN ahead of time → control over what's cached, good for large infrequently-changing files.<br>
**Deeper:** A pull CDN lazily fetches content from your origin on the first request and caches it, so it's self-managing and ideal for large catalogs where you can't predict demand. A push CDN requires you to proactively upload content to edge nodes, giving precise control over what's cached and when — better for large, infrequently-changing files you want pre-positioned. Use pull for big, dynamic catalogs; push for curated, stable assets.
<!--SR:!2026-06-14,1,230-->

DNS round-robin vs load balancer — the difference?
?
**Quick:** DNS round-robin spreads clients across IPs cheaply but has no health awareness + caching delays failover. A real LB does health checks + instant rerouting. Use DNS for coarse geo/region, LB for instance-level.<br>
**Deeper:** DNS round-robin returns rotating IPs to spread clients across servers — cheap and simple, but it's blind to server health and DNS caching means failed nodes keep getting traffic until TTLs expire. A load balancer actively health-checks backends and instantly reroutes around failures. Use DNS for coarse geographic/region steering and a real LB for fine-grained, health-aware instance-level distribution.
<!--SR:!2026-06-14,1,230-->

Edge compute vs central compute — when each?
?
**Quick:** Edge: run logic near users (low latency, offload origin) for simple, latency-sensitive work. Central: heavy/stateful/coordinated work in core data centers. Push read/transform to the edge, keep source-of-truth central.<br>
**Deeper:** Edge compute runs lightweight logic on servers geographically near users, cutting latency and offloading the origin — best for simple, stateless, latency-sensitive transforms. Central compute runs in core data centers where you have full resources, shared state, and coordination — best for heavy, stateful, or consistency-critical work. Push reads and transforms to the edge while keeping the authoritative source-of-truth and heavy processing central.
<!--SR:!2026-06-16,3,250-->

## Resilience & Reliability (essentials)

Retry vs circuit breaker — how do they combine?
?
**Quick:** Retry handles transient blips (with backoff + jitter). Circuit breaker stops calling a failing dependency for a cooldown → prevents retry storms from worsening an outage. Use both together.<br>
**Deeper:** Retries re-attempt a failed call (with exponential backoff and jitter) to ride out transient glitches. A circuit breaker tracks failure rates and, once a threshold is crossed, "opens" to fail fast for a cooldown period instead of hammering a dependency that's already down. Together they handle blips via retries while the breaker prevents those retries from becoming a storm that deepens an outage.
<!--SR:!2026-06-14,1,230-->

Active-active vs active-passive — the tradeoff?
?
**Quick:** Active-active: all regions serve traffic → full utilization + instant failover, but needs conflict handling/sync. Active-passive: standby idle until failover → simpler, but wasted capacity + failover lag.<br>
**Deeper:** Active-active runs all regions/nodes serving live traffic simultaneously, giving full capacity utilization and near-instant failover, but it requires data sync and conflict resolution across sites. Active-passive keeps a standby idle until the primary fails, which is simpler with no conflicts but wastes the standby's capacity and incurs failover lag. Choose active-active for max availability and utilization; active-passive for simplicity when some downtime on failover is acceptable.
<!--SR:!2026-06-14,1,230-->

Rate limiting vs load shedding — the difference?
?
**Quick:** Rate limiting caps a *client's* request rate (fairness, abuse). Load shedding drops requests when the *system* is overloaded (self-protection), often by priority. Both keep you up under pressure.<br>
**Deeper:** Rate limiting caps how many requests a given client/key may send in a window, enforcing fairness and blocking abuse regardless of system health. Load shedding kicks in only when the system itself is overloaded, dropping or rejecting requests (often lowest-priority first) to protect overall stability. Rate limiting is per-client policy; load shedding is system-wide self-defense — and both keep you available under pressure.
<!--SR:!2026-06-14,1,230-->

Graceful degradation vs hard failure — why prefer degradation?
?
**Quick:** Degrade: serve a reduced experience (stale cache, hidden feature, default value) instead of erroring → preserves core function during partial outages. Hard fail only when correctness forbids a fallback.<br>
**Deeper:** Graceful degradation responds to a partial outage by serving a reduced experience — stale cache, a hidden non-critical feature, or a sensible default — so the core function keeps working. Hard failure returns an error and stops, which is only appropriate when serving anything but the correct answer would be wrong (e.g. a payment). Prefer degradation to preserve availability; hard-fail only where correctness forbids a fallback.
<!--SR:!2026-06-14,1,230-->

Bulkhead isolation vs shared pool — what does bulkhead buy?
?
**Quick:** Separate resource pools (threads/conns) per dependency so one slow/failing dependency can't exhaust everything → failures stay contained. Shared pool is simpler but lets one bad actor sink the ship.<br>
**Deeper:** Bulkhead isolation gives each dependency its own resource pool (threads, connections), so a slow or failing dependency can only exhaust its own pool while the rest keep working — failures stay contained. A shared pool lets all dependencies draw from one set of resources, which is simpler but means one misbehaving dependency can starve everything else. Use bulkheads to isolate critical paths from noisy or unreliable dependencies.
<!--SR:!2026-06-14,1,230-->

Fail-fast (timeout) vs unbounded wait — why fail fast?
?
**Quick:** A bounded timeout frees the caller's resources and lets retries/fallbacks kick in; unbounded waits cascade — callers pile up and the whole chain hangs. Always set timeouts on remote calls.<br>
**Deeper:** Fail-fast means setting a bounded timeout on remote calls so a stuck dependency releases the caller's thread/connection promptly, freeing it to retry or fall back. An unbounded wait holds resources indefinitely while waiting on a hung dependency, so callers pile up and the failure cascades through the whole chain. Always set timeouts on remote calls to contain failures rather than letting them propagate.
<!--SR:!2026-06-14,1,230-->

Idempotent vs non-idempotent operations — why does it matter for retries?
?
**Quick:** Idempotent ops (PUT, set-state, dedup-keyed) are safe to retry after a timeout. Non-idempotent ops (naive POST/charge) can double-apply — make them idempotent with a key before allowing retries.<br>
**Deeper:** An idempotent operation produces the same result whether applied once or many times (PUT, set-to-value, or anything keyed by a dedup token), so retrying after an ambiguous timeout is safe. A non-idempotent operation (a naive POST or charge) applies again on each call, so a retry can double-charge or double-create. Before enabling retries, make risky operations idempotent — typically with an idempotency key the server dedups on.
<!--SR:!2026-06-14,1,230-->

Shallow vs deep health checks — the tradeoff?
?
**Quick:** Shallow ("am I up?") is cheap but misses broken dependencies. Deep ("can I reach DB/cache?") catches real failures but can cascade (one slow dep marks everything unhealthy). Shallow for liveness, deep for readiness.<br>
**Deeper:** A shallow health check just confirms the process is running and responding — cheap, but it passes even when a critical dependency is broken. A deep health check verifies downstream dependencies (DB, cache) too, catching real failures but risking cascades where one slow dependency marks every instance unhealthy at once. Use shallow checks for liveness (is the process alive?) and deep checks for readiness (can it actually serve?).
<!--SR:!2026-06-14,1,230-->

## Consistency & Coordination (essentials)

Linearizability vs serializability — the difference?
?
**Quick:** Linearizability = single-object real-time recency (reads see the latest write). Serializability = multi-object transactions appear to run in *some* serial order. Strict serializable = both.<br>
**Deeper:** Linearizability is a single-object guarantee about real-time recency: once a write completes, every later read sees it, as if there were one copy. Serializability is a multi-object transaction guarantee: concurrent transactions produce a result equivalent to *some* serial order, but with no real-time ordering promise. Strict serializability combines both — serial-order isolation plus real-time recency.
<!--SR:!2026-06-14,1,230-->

Last-write-wins vs vector clocks — when each?
?
**Quick:** LWW: pick the latest timestamp on conflict → simple, but silently drops concurrent writes. Vector clocks: detect true causality/concurrency so you can merge or surface conflicts — correct but heavier.<br>
**Deeper:** Last-write-wins resolves conflicts by keeping the write with the highest timestamp — trivial to implement, but it silently discards concurrent updates and depends on clock accuracy. Vector clocks track per-node version counters so the system can tell whether two writes are causally ordered or truly concurrent, letting you merge or surface conflicts correctly at the cost of extra metadata. Use LWW when occasional lost updates are tolerable; vector clocks when you must preserve concurrent writes.
<!--SR:!2026-06-16,3,250-->

Distributed lock vs lease — the difference?
?
**Quick:** A plain distributed lock can be held forever if the holder dies. A lease is a lock with a TTL that auto-expires → safer against crashes, but you must finish (or renew) before it lapses.<br>
**Deeper:** A plain distributed lock grants exclusive access until explicitly released, so if the holder crashes without releasing, it can block everyone indefinitely. A lease is a lock with a TTL that automatically expires, so a dead holder's grip is released after the timeout — but the live holder must complete or renew its work before the lease lapses, or another worker may proceed concurrently. Prefer leases to survive crashes; size the TTL and renew carefully.
<!--SR:!2026-06-14,1,230-->

Strong vs approximate counters — how do you scale counts?
?
**Quick:** Exact strong counts don't scale on hot items. For likes/views use approximate/eventually-consistent counters (sharded counters, async aggregation, HyperLogLog) and reconcile.<br>
**Deeper:** A strong (exact) counter serializes every increment through one record, which becomes a contention hotspot on viral items. Approximate counters spread load via sharded sub-counters, asynchronous aggregation, or probabilistic structures like HyperLogLog for unique counts, trading exactness for scale. For high-volume likes/views, use approximate eventually-consistent counters and reconcile periodically; reserve exact counters for low-volume or correctness-critical counts.
<!--SR:!2026-06-14,1,230-->

Quorum read vs single-replica read — the tradeoff?
?
**Quick:** Quorum (R+W>N) guarantees you read the latest write but costs latency + availability. Single-replica read is fast/cheap but may be stale. Tune R/W per read's freshness need.<br>
**Deeper:** A quorum read contacts enough replicas that, with R+W>N, the read set overlaps the last write set and is guaranteed to include the newest value — correct but slower and less available (it needs multiple replicas up). A single-replica read hits just one node, which is fast and cheap but may return stale data. Tune R and W per query: high R for reads that need freshness, low R where speed beats recency.
<!--SR:!2026-06-14,1,230-->

Batch vs stream processing — when each?
?
**Quick:** Batch: process large bounded data on a schedule → high throughput, high latency (reports, ETL). Stream: process unbounded events continuously → low latency (fraud, metrics, real-time feeds).<br>
**Deeper:** Batch processing runs over a large, bounded dataset on a schedule, maximizing throughput but delivering results with high latency — good for reports and ETL. Stream processing handles an unbounded flow of events continuously, giving low end-to-end latency at lower per-event throughput — good for fraud detection, live metrics, and real-time feeds. Use batch when freshness can wait and volume is large; stream when results must be near-real-time.
<!--SR:!2026-06-14,1,230-->

## Processing & Scaling (essentials)

Lambda vs Kappa architecture — the difference?
?
**Quick:** Lambda runs parallel batch + speed layers (accurate but two codebases). Kappa uses one stream-processing pipeline for both real-time and reprocessing (simpler, replay from the log).<br>
**Deeper:** Lambda architecture maintains two paths — a batch layer for accurate, complete results and a speed layer for low-latency approximations — merged at query time, which is robust but duplicates logic across two codebases. Kappa architecture uses a single stream-processing pipeline for both live and historical data, reprocessing by replaying the event log, which simplifies maintenance. Choose Kappa when one stream pipeline suffices; Lambda when batch and real-time genuinely need separate treatment.
<!--SR:!2026-06-14,1,230-->

Static vs auto-scaling — when each?
?
**Quick:** Static (fixed capacity): predictable load, simpler, no cold-start risk, but you pay for peak. Auto-scaling: follow demand → cost-efficient for spiky load, but watch scale-up lag + thrashing.<br>
**Deeper:** Static scaling provisions a fixed capacity sized for peak — simple, predictable, and free of cold-start/scale-up lag, but you pay for that peak even when idle. Auto-scaling adds and removes instances based on demand, saving cost on spiky or variable load, but it must contend with scale-up latency and the risk of thrashing (rapid up/down flapping). Use static for steady, predictable load; auto-scaling for bursty or variable traffic.
<!--SR:!2026-06-14,1,230-->

Functional vs horizontal partitioning — the difference?
?
**Quick:** Functional: split by feature/table onto different DBs (users DB, orders DB) — easy first step, bounded by the biggest single table. Horizontal (sharding): split rows of one table across nodes — scales a single huge dataset.<br>
**Deeper:** Functional partitioning splits data by feature or table onto separate databases (a users DB, an orders DB) — an easy early scaling step, but ultimately bounded by the largest single table that still lives on one node. Horizontal partitioning (sharding) splits the rows of one table across many nodes by a shard key, scaling a single enormous dataset beyond one machine. Functionally partition first; shard when an individual table outgrows a single node.
<!--SR:!2026-06-14,1,230-->
