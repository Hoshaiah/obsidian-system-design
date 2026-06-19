---
tags:
  - flashcards/caching
---

# Caching & Performance

Where reads get fast — and the hazards that bite you.

---

Name the main cache-read pattern and its two hazards.
?
**Quick:** Cache-aside (lazy loading); hazards are stale data (you own invalidation/TTL) and cache stampede/thundering herd on expiry.<br>
**Deeper:** Cache-aside means the app checks the cache first and, on a miss, loads from the DB and populates the cache itself — the cache is "to the side," not in the write path. Because the app controls writes, you must explicitly invalidate or set a TTL, so reads can return stale data until then. A stampede (thundering herd) happens when a popular key expires and many concurrent requests all miss at once, hammering the DB simultaneously.

How do you prevent a cache stampede?
?
**Quick:** Request coalescing/locks (one fetch fills the cache), staggered TTLs/jitter, or background refresh before expiry.<br>
**Deeper:** Request coalescing (a.k.a. single-flight) lets only one request acquire a lock to recompute the value while others wait for that single fill, instead of all hitting the DB. Adding random jitter to TTLs prevents many keys from expiring at the same instant. Background or "early" refresh proactively recomputes a value before it expires so the cache is never empty under load.

Beyond GET/SET, name three Redis capabilities worth citing.
?
**Quick:** Sorted sets (feeds/leaderboards), TTL keys (rate limiting/sessions), pub/sub + distributed locks.<br>
**Deeper:** Sorted sets store members ranked by a score, ideal for leaderboards or time-ordered feeds with O(log n) range queries. TTL (time-to-live) keys auto-expire, which is how you build session stores and sliding-window rate limiters. Pub/sub provides lightweight messaging, and patterns like Redlock use atomic SET-with-expiry to implement distributed locks across multiple app servers.

What's a hot key and how do you mitigate it?
?
**Quick:** One key getting disproportionate traffic; mitigate with replication of that key, client/local caching, or key splitting.<br>
**Deeper:** A "hot key" is a single cache/DB key (e.g. a celebrity's profile) drawing so much traffic that the one node holding it saturates while others idle. Fixes: replicate the key across nodes so reads spread, cache it in-process on each app server to skip the network, or split it into sub-keys (key1, key2…) that the client picks among. The same idea applies to hot partitions in sharded stores.

Three cache eviction policies and the default to name?
?
**Quick:** LRU (default — evict least recently used), LFU (least frequently used), FIFO. Say LRU unless asked.<br>
**Deeper:** Eviction policies decide what to drop when the cache is full. LRU (Least Recently Used) evicts the entry untouched for the longest time, which suits most access patterns where recent data is reused. LFU (Least Frequently Used) tracks hit counts and is better when popularity is stable; FIFO (First In First Out) just evicts the oldest insertion regardless of usage.

What does a CDN cache, and what's its main win?
?
**Quick:** Static/edge content (images, video, JS) near users; cuts latency + origin load. Mention it for any read-heavy, geo-distributed read path.<br>
**Deeper:** A CDN (Content Delivery Network) is a globally distributed set of edge servers (points of presence) that cache copies of content close to users geographically. Serving from a nearby edge cuts round-trip latency and offloads requests from your origin servers. It's best for static or rarely-changing assets, though edge caching of dynamic responses is possible with careful TTLs and invalidation.

Write-through vs write-back caching in one line?
?
**Quick:** Write-through = write cache + DB synchronously (safe, slower); write-back = write cache now, DB later (fast, risk of loss).<br>
**Deeper:** In write-through, every write updates the cache and the backing store together before returning, keeping them consistent at the cost of write latency. In write-back (write-behind), the write hits only the cache and is flushed to the DB asynchronously, which is fast but loses data if the cache node dies before flushing. The choice trades durability against write performance.

What problem does a cache NOT solve?
?
**Quick:** Write scaling and strong consistency — caches speed reads and can serve stale data.<br>
**Deeper:** A cache is a read accelerator: it stores copies of data to avoid recomputing or re-fetching, so it does nothing to increase write throughput, which still bottlenecks on the underlying store (use sharding/partitioning for that). Because cached copies can lag the source of truth, caches inherently weaken consistency — readers may see stale values until invalidation or TTL expiry. If you need strong consistency, read from the authoritative store.

Where can caching live in the stack (name the layers)?
?
**Quick:** Client/browser → CDN → API/in-process → distributed cache (Redis) → DB query cache. Cache as close to the user as correctness allows.<br>
**Deeper:** Each layer intercepts reads earlier in the path: browser/client caches (HTTP cache, local storage) avoid network calls entirely; the CDN serves edge content near users; in-process/application caches live inside the app server for sub-millisecond hits; a distributed cache like Redis is shared across servers; and the DB's own query/buffer cache is the last stop. Caching nearer the user is faster but harder to invalidate, so push it only as far as your consistency requirements allow.
