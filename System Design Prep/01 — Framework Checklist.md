# 01 — Framework Checklist

The spine you run on every problem. This is the Hello Interview "Delivery" framework — keep it, just make it automatic. Time budget assumes a 45-min interview.

> Say the step names out loud as you go. Interviewers grade structure as much as content. Narrating "let me start with requirements" signals seniority.

---

## 1. Requirements (~5 min)

**Functional** — what the system *does*. Get 3–4 core actions, no more. Ask, don't assume.
- "What are the main things a user can do?"
- Write them as verbs: post a tweet, view timeline, follow a user.
- **Explicitly cut scope out loud:** "I'll skip auth, analytics, and DMs unless you want them."

**Non-functional** — the *qualities*. This is where you earn senior points.
- Scale: DAU, read:write ratio, peak QPS?
- Latency targets (reads usually < 200ms)
- Consistency vs availability — **pick one per feature and justify it** (this is the CAP call)
- Durability, security, where relevant

> Don't let requirements sprawl. 5 min, lock it, move on. You can revisit.

- [ ] Stated 3–4 functional requirements
- [ ] Stated the key non-functional ones (scale, latency, consistency)
- [ ] Explicitly scoped things OUT

---

## 2. Core Entities (~2 min)

The nouns. User, Tweet, Follow, Like. Just name them — you'll detail schema later. This anchors the API.

- [ ] Listed the 3–6 core entities

---

## 3. API / Interface (~5 min)

One endpoint per functional requirement. Keep it boring REST unless there's a reason.
```
POST /tweets        { text }            -> tweetId
GET  /feed?cursor=  &limit=             -> Tweet[]
POST /users/:id/follow
```
- Use **cursor pagination**, not offset, for feeds (say why: stable under inserts).
- Note auth via header, not in the body — gestures at security without rabbit-holing.

- [ ] One endpoint per functional requirement
- [ ] Pagination strategy named for any list endpoint

---

## 4. High-Level Design (~10–15 min)

Draw the boxes that satisfy the functional requirements. Client → API gateway / LB → services → data stores. Walk one request through end to end, out loud.

- Start simple. A single service + single DB is a fine v1. **Resist premature scaling.**
- Name your data store and **justify it in one line** (see [[02 — Building Blocks Cheatsheet]]).
- Get a working system on the board before you optimize anything.

- [ ] Boxes satisfy every functional requirement
- [ ] Walked one request end-to-end out loud
- [ ] Picked + justified the primary data store

---

## 5. Deep Dives (~10–15 min) — **where you win or lose**

The interviewer probes. Or you drive: "The interesting part here is X — let me go deep." Pick the 1–2 hardest components and reason about tradeoffs.

For each deep dive, hit the trifecta:
1. **The problem** ("this read path won't scale because…")
2. **2+ options** with tradeoffs
3. **A pick** + the conditions under which you'd switch

Tie back to the non-functional requirements from step 1. "We said reads must be < 200ms, so here I'd add a cache and accept staleness because…"

- [ ] Drove at least one deep dive yourself
- [ ] Gave ≥2 options + tradeoffs + a justified pick
- [ ] Connected the choice back to a stated requirement

---

## Throughout
- Manage your own time — glance at the clock, don't let requirements eat 15 min.
- Think out loud. Silence reads as being stuck.
- It's a conversation, not a monologue. Check in: "Does that tradeoff make sense, or want me to go deeper on the queue?"
- When you don't know something, say what you'd measure or how you'd decide — that beats bluffing.
