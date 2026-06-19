# 03 — Estimation Cheatsheet

Back-of-envelope math should take 2 minutes and feel automatic. The point isn't precision — it's justifying your scaling decisions ("12K QPS means a single DB won't cut it, so…"). Round aggressively.

---

## Numbers to memorize

**Time**
- 1 day ≈ 86,400 s ≈ **100K s** (round up — makes division trivial)
- 1 month ≈ 2.5M s
- 1 year ≈ 30M s

**QPS shortcut**
- **1M requests/day ≈ ~12 QPS** (1M / 86,400 ≈ 11.6). Memorize this; scale linearly.
- 100M/day ≈ 1.2K QPS · 1B/day ≈ 12K QPS
- Peak ≈ 2–3× average. State your multiplier.

**Powers of 2 → data size**
- 2^10 = 1 thousand → KB
- 2^20 = 1 million → MB
- 2^30 = 1 billion → GB
- 2^40 = 1 trillion → TB
- 2^50 → PB

**Byte sizes (rules of thumb)**
- char ≈ 1 byte · typical metadata row ≈ ~1 KB
- short text post ≈ ~hundreds of bytes · a long one ≈ ~1 KB
- thumbnail ≈ ~10s KB · photo ≈ ~1 MB · minute of video ≈ ~10s MB
- UUID ≈ 16 bytes · timestamp ≈ 8 bytes · int ≈ 4 bytes

**Latency ladder (Jeff Dean, rounded)**
- Memory read ≈ 100 ns
- SSD random read ≈ 100 µs
- Datacenter round trip ≈ 0.5 ms
- Disk (spinning) seek ≈ 10 ms
- Cross-continent round trip ≈ 100–150 ms

Takeaway for interviews: **memory ≫ SSD ≫ network ≫ disk.** Cache to avoid disk + cross-region hops.

**Throughput rough caps (to justify "we need to scale")**
- Single SQL node: ~thousands of writes/sec, more reads with replicas
- Redis: ~100K+ ops/sec per node
- Single server: ~tens of thousands of concurrent connections (websockets) before you shard

---

## The 4 calculations (do these in order, out loud)

**1. QPS**
```
DAU × actions/user/day = requests/day
requests/day × (1 / 100K) = avg QPS
avg QPS × 2–3 = peak QPS
```
Split read vs write QPS — they scale differently.

**2. Storage**
```
writes/day × bytes/write = bytes/day
× 365 × retention years = total
```
Add a fudge for indexes/replication (×2–3) if it matters.

**3. Bandwidth**
```
QPS × bytes/response = bytes/sec
```
Egress (reads) usually dominates. Matters for media/video, less for text.

**4. Memory (for caching)**
```
Cache the hot set. 80/20: cache ~20% of daily reads.
hot items × size/item = cache RAM needed
```
→ decides how many cache nodes.

---

## Worked example — Twitter-ish

- 200M DAU, each posts 0.5 tweets/day, reads 50 tweets/day
- **Writes:** 200M × 0.5 = 100M tweets/day → ÷100K = ~1K write QPS, peak ~3K
- **Reads:** 200M × 50 = 10B reads/day → ÷100K = ~100K QPS, peak ~250K
- **Read:write ≈ 100:1** → screams *read-heavy* → caching + read replicas + fan-out
- **Storage:** 100M tweets × ~300 bytes (incl. metadata) ≈ 30 GB/day → ~11 TB/year text alone

That read:write ratio decides your whole design. **That's the point of the math** — not the digits, the *conclusion you draw* from them. Always say the conclusion out loud.
