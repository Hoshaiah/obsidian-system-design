# 04 — Problems & Deep Dives

What to actually design each day. For each: the prompt, requirements to target, the deep dives that matter, and the trap most candidates fall into. Run each through The Loop in [[00 — 7-Day Plan]].

> Don't read the deep-dive notes before your cold attempt. Attempt first, *then* check yourself against these.

---

## Day 1 — URL Shortener (or Rate Limiter)
*Easy on purpose. You're grooving the framework, not being challenged.*

**Functional:** shorten a long URL → short code; redirect short → long.
**Non-functional:** read-heavy (100:1+), low-latency redirects, high availability, codes don't collide.

**Deep dives:**
- **Code generation:** counter + base62 (no collisions, but a sequential global counter is a bottleneck → pre-allocate ranges per server) vs hash + truncate (collisions → check-and-retry). Know both, pick counter+base62 and say why.
- **Redirect:** 301 (permanent, cached by browser — kills your analytics) vs 302 (temporary — every hit reaches you). Pick based on whether you need click tracking.
- **Read scaling:** cache hot URLs; the mapping is immutable so caching is trivial (no invalidation).

**Trap:** over-engineering a 5-line problem. Show restraint — it's a signal.

---

### Rate Limiter (alternative Day 1)
**Algorithms:** token bucket (allows bursts, most common), leaky bucket (smooths), fixed window (simple, boundary spikes), sliding window log/counter (accurate, more memory).
**Deep dive:** distributed rate limiting — counters in Redis (atomic INCR + TTL), the race between check-and-increment (→ Lua script / atomic op), where to enforce (API gateway).

---

## Day 2 — Twitter Timeline / News Feed
*The canonical read-heavy problem.*

**Functional:** post a tweet; follow users; view home timeline (tweets from people you follow, reverse-chron).
**Non-functional:** ~100:1 read:write, timeline loads < 200ms, availability > consistency (a few seconds stale is fine).

**Deep dives:**
- **Fan-out on write (push):** on post, write the tweet into every follower's precomputed timeline (in Redis). Reads are dirt cheap. *Cost:* writes explode for high-follower accounts; wasted work for inactive followers.
- **Fan-out on read (pull):** build the timeline at read time by querying everyone you follow. Cheap writes. *Cost:* expensive, slow reads.
- **The celebrity problem:** push breaks for accounts with millions of followers (one tweet = millions of writes). → **Hybrid:** push for normal users, pull for celebrities, merge at read time. This is the senior answer.
- **Timeline cache:** Redis sorted set per user, capped to ~800 entries.

**Trap:** picking push or pull and not handling the celebrity case. The hybrid is the whole point.

---

## Day 3 — Ad Click Aggregator
*Write-heavy, data-intensive. Your turf — go deep.*

**Functional:** ingest click events; query aggregated metrics (clicks per ad per minute/hour) with low latency.
**Non-functional:** very high write throughput, near-real-time queries, accuracy matters (it's billing), handle late/duplicate events.

**Deep dives:**
- **Ingestion:** events → Kafka (buffer + durability + replay). Partition by ad_id.
- **Stream processing:** Flink/Spark Streaming with **tumbling windows** (e.g. 1-min) to aggregate.
- **Exactly-once / dedup:** events arrive at-least-once → dedup by event_id, or idempotent aggregation. Don't double-bill.
- **Hot partition:** a viral ad floods one partition → key by `ad_id + random suffix` (sub-partition), aggregate in two stages.
- **Late events / watermarks:** events arrive out of order → watermarks decide when a window is "done"; allow a grace period.
- **Lambda / kappa:** fast streaming path for real-time + a **batch reconciliation** job over raw events for correctness. Mention reconciliation — it's the thing billing systems actually do, and you'd know that from a payments background.

**Trap:** ignoring duplicates and late events. For a *billing* system that's disqualifying — and it's exactly where your fintech instinct shines.

---

## Day 4 — WhatsApp / Notification System
*Real-time, stateful connections.*

**Functional:** send/receive 1:1 messages; delivery + read receipts; online/offline presence; group chat (stretch).
**Non-functional:** low latency, message ordering, deliver to offline users, durability (don't lose messages).

**Deep dives:**
- **Connection layer:** persistent **websockets** to connection servers. Users on different servers → a routing/registry layer (which server holds which user) + a message bus between them.
- **Delivery to offline users:** store-and-forward — persist undelivered messages in a per-user inbox (a queue / Cassandra), flush on reconnect, then push via APNS/FCM.
- **Ordering:** per-conversation sequence numbers or timestamps; "order only guaranteed within a conversation."
- **Delivery guarantees:** ACKs at each hop (sent → delivered → read); client retries with message IDs → dedup.
- **Presence:** heartbeats + TTL in Redis; last-seen. Note presence is expensive at scale and often eventually consistent.

**Trap:** treating it as stateless request/response. The whole challenge is *persistent connection state* and offline delivery.

---

## Day 5 — Payment System  ⭐ your edge
*Where your payments background makes you the best candidate in the pool. Go deep on purpose.*

**Functional:** charge a customer; handle async settlement from a processor; track payment state; refunds.
**Non-functional:** **strong consistency** (it's money), exactly-once (never double-charge), auditability, durability. Availability matters but correctness wins ties.

**Deep dives:**
- **Idempotency:** client sends an idempotency key → dedup table → at-most-once charge even under retries/timeouts. This is *the* payments answer; state it first.
- **Double-entry ledger:** every transaction = balanced debits/credits, append-only, immutable. Source of truth for audit + reconciliation. (You can speak to this from real experience.)
- **State machine:** payment status (created → pending → succeeded/failed → refunded); illegal transitions rejected.
- **Async settlement:** processor confirms later via webhook → **outbox pattern** / reliable event handling so you don't lose or double-apply confirmations.
- **Distributed transactions:** prefer a **saga** (compensating actions) over 2PC across services; explain why 2PC is avoided (blocking, coordinator SPOF).
- **Reconciliation:** periodic batch reconciles your ledger against the processor's records — catches drift. Most candidates have never heard of this. You've lived it.

**Trap (for others, not you):** treating money like any CRUD object. Your job is to make the consistency, idempotency, and reconciliation story so concrete the interviewer realizes you've shipped this.

**If the role isn't fintech:** swap in **Typeahead / Search Autocomplete** — trie structure, top-k per prefix, Elasticsearch, prefix caching, debouncing. You know ES, so it's still a strength. But keep payments ready; if they say "design anything you've built," reach for it.

---

## Day 6 — Mocks
Pick any two you *didn't* drill (e.g. **Dropbox/file storage** and **Uber/proximity**) so you're forced to improvise with the framework rather than recite. The goal is testing the *process* under pressure, not recall.

- **Dropbox:** chunking, dedup by content hash, sync/conflict resolution, metadata vs blob split.
- **Uber:** geospatial indexing (geohash / quadtree / S2), matching, real-time location updates.

Record yourself. Watch it back. Note every stall and every place you went shallow → flashcards.
