---
tags:
  - flashcards/scaling
---

# Scaling & Distribution

How you grow past one box — and the new bug classes each move introduces.

---

Leader-follower replication scales what, and introduces what bug class?
?
**Quick:** Scales reads + gives failover; introduces replication lag → stale follower reads and read-your-own-writes problems.<br>
**Deeper:** In leader-follower (primary-replica) replication, all writes go to one leader and are streamed asynchronously to read-only followers, so you can fan reads out across replicas and promote a follower if the leader dies. Replication lag is the delay before a write reaches a follower; a client reading a stale follower may miss its own just-committed write (the read-your-own-writes anomaly). Mitigations include reading from the leader for recent writes, sticky routing, or monitoring lag.

Sharding scales what, and what's the hardest decision?
?
**Quick:** Scales writes + storage horizontally; the shard-key choice is the whole game (bad key → hot shards, hard cross-shard queries).<br>
**Deeper:** Sharding (horizontal partitioning) splits a dataset across independent nodes so each handles a slice of the writes and storage, breaking the single-leader write ceiling. The shard key is the field used to decide which shard a row lives on; a skewed key concentrates traffic on a hot shard, and queries that don't include the key must scatter-gather across all shards. Picking it is hard because it's expensive to change once data is distributed.

What makes a good shard key?
?
**Quick:** High cardinality + even access distribution + matches your most common query, so load spreads and queries hit one shard.<br>
**Deeper:** Cardinality is the number of distinct values; high cardinality gives the system enough buckets to spread data finely. Even access distribution means no single value (e.g. one celebrity user) draws a disproportionate share of traffic and creates a hot shard. Aligning the key with frequent query predicates lets those queries route to a single shard instead of fanning out, which is far cheaper.

What problem does consistent hashing solve?
?
**Quick:** Minimizes data movement when nodes are added/removed (vs naive modulo, which rehashes almost everything).<br>
**Deeper:** With naive `hash(key) % N` sharding, changing N (adding/removing a node) changes almost every key's assignment, forcing a massive reshuffle. Consistent hashing maps both keys and nodes onto a ring; a key belongs to the next node clockwise, so adding/removing a node only relocates the keys in that one arc (~1/N of the data). Virtual nodes (many ring points per physical node) smooth out uneven load.

What do virtual nodes add to consistent hashing?
?
**Quick:** Each physical node owns many points on the ring → smoother load balance and less skew when nodes join/leave.<br>
**Deeper:** With one ring point per node, the arcs between nodes vary wildly in size, so some nodes own much more data than others. Virtual nodes (vnodes) place many ring positions per physical node, averaging out arc sizes so load is distributed more evenly. They also let heterogeneous hardware carry proportional load (give a beefier box more vnodes) and spread a departing node's data across many survivors instead of dumping it all on one neighbor.

Vertical vs horizontal scaling — the tradeoff?
?
**Quick:** Vertical = bigger box, simple but capped + SPOF; horizontal = more boxes, scales further but adds coordination/consistency complexity.<br>
**Deeper:** Vertical scaling (scaling up) adds CPU/RAM/disk to a single machine — no app changes needed, but you hit a hardware ceiling and that one machine is a single point of failure (SPOF: a component whose failure takes down the whole system). Horizontal scaling (scaling out) adds more machines, giving near-unlimited headroom and redundancy, but now you must handle data partitioning, replication, and distributed consistency. Most large systems scale up first for simplicity, then out when forced.

What does a load balancer give you beyond distribution?
?
**Quick:** Health checks/failover, TLS termination, and a single stable entry point; pick L4 (fast, TCP) vs L7 (content-aware routing).<br>
**Deeper:** Beyond spreading requests, a load balancer probes backends with health checks and stops routing to unhealthy ones (failover), and it can terminate TLS so backends skip encryption overhead. It also presents one stable virtual IP/DNS name, decoupling clients from the changing set of instances. L4 (transport layer) balances on IP/port — fast but blind to content; L7 (application layer) inspects HTTP, enabling path/header-based routing, sticky sessions, and per-route policies at higher cost.

How do you keep services stateless, and why bother?
?
**Quick:** Push session/state to a shared store (Redis/DB); statelessness lets any instance handle any request → trivial horizontal scaling.<br>
**Deeper:** A stateless service keeps no client-specific data in local memory between requests, so externalizing session/state to a shared store (Redis, a database) means every instance is interchangeable. That lets a load balancer send any request to any instance without sticky sessions, and instances can be added, removed, or restarted freely. It is the foundation of elastic autoscaling and fast failure recovery, since losing one node loses no user state.

What's a quorum (R + W > N) for?
?
**Quick:** In leaderless replication, requiring overlapping read/write replica sets guarantees reads see the latest write while staying available.<br>
**Deeper:** In leaderless replication (e.g. Dynamo-style), data is stored on N replicas; a write waits for W acknowledgements and a read queries R replicas. When R + W > N, the read set and write set must overlap by at least one node, so a read is guaranteed to touch a replica holding the newest write. Tuning R and W trades latency and availability against consistency (e.g. W=1 is fast writes but weaker guarantees).

Single point of failure — how do you talk about removing one?
?
**Quick:** Replicate it + add automatic failover; name the component (DB primary, LB, coordinator) and how traffic reroutes.<br>
**Deeper:** A single point of failure (SPOF) is any component with no redundancy whose failure halts the system. You remove one by adding redundancy (replicas, standby instances, multiple AZs) plus a mechanism to detect failure and automatically shift traffic to a healthy unit. In an interview, identify the specific SPOF (database primary, load balancer, coordinator/leader), then describe the failover path — promote a replica, use a redundant LB pair with a floating IP, or run leader election.
