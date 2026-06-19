# System Design — 7-Day Plan

**Goal:** Not to *learn* system design this week. To make *driving a design conversation out loud, on a timer* automatic. You already know the pieces (Kafka, Redis, Elasticsearch, CAP, sharding). This week is reps, not new theory.

**The ratio:** ~70% active output, ~30% input. If you catch yourself watching a video for more than ~30 min in a sitting, stop and go talk at a whiteboard.

**Companion notes:**
- [[01 — Framework Checklist]] — the spine you run on every problem
- [[02 — Building Blocks Cheatsheet]] — components + the one tradeoff that matters for each
- [[03 — Estimation Cheatsheet]] — numbers to have memorized for back-of-envelope
- [[04 — Problems & Deep Dives]] — what to actually design each day
- [[05 — Flashcards]] — spaced repetition deck (Day 1 onward)

---

## The Loop (run this on EVERY problem)

1. **Cold attempt, out loud, on a timer (~40 min).** Excalidraw or whiteboard. Talk as if someone's in the room. Don't pause to look things up — if you freeze, you found a gap. Keep going.
2. **Review against reference** (HI writeup or Alex Xu vol. 1/2). Mark every spot where you hand-waved, froze, or missed a tradeoff.
3. **Redo the weak section out loud, corrected.**
4. **Distill** the reusable pattern + write 5–10 flashcards of the *specific* tradeoffs you missed.
5. **Space it** — those flashcards go into tomorrow's warm-up.

Two non-negotiables: **talk out loud** and **use a timer.** Silent reading trains nothing the interview measures.

---

## Daily 5-Hour Template

- [ ] **0:30** — Warm-up: yesterday's flashcards (spaced repetition)
- [ ] **2:30** — One problem through the full Loop
- [ ] **1:00** — Second pass, or a deep dive on one component
- [ ] **0:45** — Distill into notes + write new flashcards
- [ ] **0:15** — Buffer

Protect the out-loud blocks. If you fade, cut passive review — never the practice.

---

## Day 1 — Foundation
- [ ] Rewrite [[01 — Framework Checklist]] in your own words as a one-page checklist
- [ ] Consolidate [[02 — Building Blocks Cheatsheet]] (you know most of this — this is recall practice, not learning)
- [ ] Drill 4 estimations from [[03 — Estimation Cheatsheet]]: QPS, storage, bandwidth, server count
- [ ] Run ONE easy problem slowly, no timer, to groove the structure: **URL shortener** or **rate limiter** (see [[04 — Problems & Deep Dives]])
- [ ] Seed [[05 — Flashcards]] into your spaced-repetition plugin; do a first pass

## Day 2 — Read-heavy / Feeds
- [ ] Loop: **Design Twitter timeline / news feed**
- [ ] Deep dive: fan-out on write vs read, celebrity problem, timeline cache
- [ ] Warm-up: Day 1 flashcards
- [ ] Write new cards

## Day 3 — Write-heavy / Data-intensive
- [ ] Loop: **Ad click aggregator** (your turf — push the deep dives hard)
- [ ] Deep dive: stream processing, tumbling windows, exactly-once, hot partitions, batch reconciliation
- [ ] Warm-up: Day 1–2 flashcards

## Day 4 — Real-time / Messaging
- [ ] Loop: **WhatsApp** or **notification system**
- [ ] Deep dive: websockets, presence, delivery guarantees, ordering, offline delivery, APNS/FCM
- [ ] Warm-up: Day 1–3 flashcards

## Day 5 — Your Edge: Financial Systems
- [ ] Loop: **Payment system** (idempotency, ledger, exactly-once, reconciliation)
- [ ] This is where your payments background lets you dominate a deep dive most candidates fumble. Go deep on purpose.
- [ ] If target role isn't fintech: do **typeahead/search** instead (you know Elasticsearch), keep payments in your back pocket
- [ ] Warm-up: Day 1–4 flashcards

## Day 6 — Mocks (highest-leverage day)
- [ ] Two full 45-min mocks, end to end, out loud, **recorded**
- [ ] HI's AI mock, or a peer
- [ ] Review brutally: where did you stall? where did you go shallow? what did you forget to say out loud?
- [ ] Cards for every miss

## Day 7 — Taper
- [ ] Spaced repetition: ALL flashcards
- [ ] One light timed run on a problem you feel weakest on
- [ ] Re-skim all notes
- [ ] **No new material.** Sleep well — a rested brain interviews better than a crammed one.

---

## On the paralysis
You don't have time to be comprehensive, and trying to be is the thing freezing you. This plan deliberately skips problems. Four archetypes deep beats twelve archetypes shallow — the patterns transfer. Pick this, stop evaluating alternatives, and the indecision dissolves because there's nothing left to decide.

**If you take one thing:** the *deep dive* is where mid/senior candidates win or lose. Anyone can sketch boxes. Far fewer can go three layers into one component and reason about tradeoffs out loud. Your production + on-call experience is the raw material for exactly that.
