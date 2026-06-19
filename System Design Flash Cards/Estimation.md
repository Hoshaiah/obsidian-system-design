---
tags:
  - flashcards/estimation
---

# Estimation

Back-of-envelope in 2 minutes. The point is the *conclusion you draw*, not the digits.

---

1M requests/day ≈ how many QPS?
?
**Quick:** ~12 QPS (1M ÷ 86,400 ≈ 11.6). Memorize 1M/day ≈ ~12 QPS.<br>
**Deeper:** QPS = queries (requests) per second. There are ~86,400 seconds in a day (≈10^5), so any "per day" figure divided by ~100,000 gives average QPS. This is the anchor number to derive others from — e.g. 100M/day ≈ 1,200 QPS — and you then multiply by a 2–3× peak factor for worst-case sizing.

How do you get peak QPS from average?
?
**Quick:** Multiply by 2–3× and state the multiplier out loud.<br>
**Deeper:** Average QPS spreads load evenly across all 86,400 seconds, but real traffic clusters in peak hours (lunchtime, evenings, time zones). The peak-to-average ratio captures this burstiness; a 2–3× rule of thumb is standard for consumer apps. You size capacity for peak, not average, or the system falls over during the busiest window.

What's the *point* of estimation in an interview?
?
**Quick:** To justify a scaling decision, e.g. "100:1 read:write → read-heavy → cache + replicas." Always say the conclusion.<br>
**Deeper:** Estimation is a means to an architectural decision, not arithmetic for its own sake. A read:write ratio tells you whether to invest in caching and read replicas (read-heavy) or in write batching/sharding (write-heavy); a bandwidth number tells you whether you need a CDN. State the number, then state what it makes you do.

Powers of 2: 2^10, 2^20, 2^30, 2^40 map to what units?
?
**Quick:** KB, MB, GB, TB (≈ thousand, million, billion, trillion bytes).<br>
**Deeper:** Each power of 2^10 (1024) is one binary order of magnitude and lines up closely with a decimal order of ~1000. So 2^10 ≈ 10^3 (KB), 2^20 ≈ 10^6 (MB), 2^30 ≈ 10^9 (GB), 2^40 ≈ 10^12 (TB). Knowing these lets you convert record counts × bytes into storage tiers instantly without precise math.

Latency ladder, fastest first: memory, SSD, datacenter round-trip, disk seek.
?
**Quick:** Memory ~100ns < SSD ~100µs < DC round-trip ~0.5ms < disk seek ~10ms.<br>
**Deeper:** Latency is the time to complete a single operation, and each tier here is roughly 100–1000× slower than the one above it. A datacenter round-trip is a network hop within one region; a disk seek is the mechanical head-move on a spinning HDD (the slowest). The ladder explains why caching in memory and avoiding random disk I/O dominate performance design.

Rough size of one typical row / small record for storage math?
?
**Quick:** ~1 KB. Tweets/metadata ~hundreds of bytes; round to 1 KB for speed.<br>
**Deeper:** A "record" is one logical entry — a row, a document, a message — and 1 KB is a deliberate round-up that keeps mental math simple and slightly conservative. Text-only records (a tweet, a user profile row) are usually a few hundred bytes; media or large blobs are sized separately. Use 1 KB as the default multiplier in records/day × bytes/record calculations.

A char/byte is 1B — how big is a UUID, a typical URL?
?
**Quick:** UUID ~16B (36 chars as string); URL ~100B. Use these for ID/storage estimates.<br>
**Deeper:** A UUID (universally unique identifier) is 128 bits = 16 bytes in binary form, though it's 36 characters when written as a hyphenated hex string. ASCII/UTF-8 stores one character per byte, so a URL of ~100 characters is ~100 bytes. These constants let you size index columns, key spaces, and link-storage tables quickly.

Seconds in a day, for QPS math?
?
**Quick:** ~86,400 ≈ 10^5. So X/day ÷ 10^5 ≈ X×10⁻⁵ QPS.<br>
**Deeper:** 86,400 = 24 × 60 × 60, and rounding it to 10^5 (100,000) is the trick that makes daily-to-QPS conversion a one-step shift of the decimal. The small over-estimate of the denominator makes your QPS slightly conservative, which is fine for sizing. This single approximation underpins nearly every back-of-envelope traffic calculation.

How do you estimate storage for N years?
?
**Quick:** records/day × bytes/record × 365 × years, then ×(1 + replication/index overhead). State the overhead factor.<br>
**Deeper:** Raw data size is daily write volume scaled out over the retention period (days × years). Real systems store more than raw bytes: replication keeps multiple copies for durability (e.g. 3× in many distributed stores) and indexes/metadata add further overhead. Multiply by a stated factor (often ~1.5–3×) so your capacity number reflects what actually lands on disk.

How do you estimate read bandwidth?
?
**Quick:** QPS × response size. E.g. 10K QPS × 1 KB = 10 MB/s — then decide if a CDN/cache is needed.<br>
**Deeper:** Bandwidth is data moved per unit time, so read throughput is request rate (QPS) times the average payload returned per request. Comparing that figure to what a single server or link can sustain tells you whether to offload static/popular content to a CDN (edge cache) or add caching layers. Always pair the number with the offloading decision it drives.
