---
tags:
  - flashcards/framework
---

# Framework & Delivery

The structure interviewers grade as much as content. Make it automatic.

---

What are the 5 steps of the design framework, in order?
?
**Quick:** Requirements (functional + non-functional) → Core entities → API → High-level design → Deep dives.<br>
**Deeper:** Functional requirements are what the system does (features/user actions); non-functional requirements are quality constraints like scale, latency, availability, and consistency. Core entities are the main data objects (e.g., User, Post); the API is the contract clients call; the high-level design lays out the major components and data flow; deep dives drill into the hard parts. Following this order keeps you from designing before you know what you're building.

How should you split your time in a 45-min interview?
?
**Quick:** ~5 min requirements, ~5 min API/entities, ~10–15 min high-level design, ~10–15 min deep dives, buffer for wrap-up.<br>
**Deeper:** The exact minutes flex with the prompt, but the ratio is the point — most candidates over-spend on requirements/high-level boxes and run out of time for deep dives, which is where senior signal lives. Keep an eye on the clock and explicitly time-box so you always reach at least one deep dive.

What separates a mid from a senior *requirements* phase?
?
**Quick:** Nailing non-functional reqs (scale, latency, an explicit consistency-vs-availability call per feature) and scoping things OUT loud.<br>
**Deeper:** Non-functional requirements are quality attributes rather than features — throughput, response time, durability, and where you sit on the consistency/availability spectrum (the CAP-theorem tradeoff that under a network partition you can guarantee strong consistency or availability, not both). Senior candidates make that call per feature instead of system-wide, and explicitly declare what's out of scope so the design stays focused and tractable.

Which step actually decides the outcome at mid/senior level?
?
**Quick:** The deep dives — anyone can draw boxes; going 3 layers deep on one component with real tradeoffs is the differentiator.<br>
**Deeper:** A deep dive is a focused investigation of one component or bottleneck — e.g., how a feed is fanned out, how a counter stays accurate under concurrency, or how a cache is kept fresh. The high-level design proves you know the pieces; the deep dive proves you can reason about failure modes, contention, and tradeoffs, which is the actual signal interviewers weight at senior level.

For each deep dive, what trifecta do you deliver?
?
**Quick:** (1) the problem, (2) two+ options with tradeoffs, (3) a justified pick + when you'd switch.<br>
**Deeper:** This structure mirrors how real architecture decisions are documented (e.g., an ADR — Architecture Decision Record). Naming the problem first frames why it's hard; presenting multiple options with tradeoffs shows breadth; and committing to a choice with the conditions under which you'd reverse it shows you understand that designs are context-dependent, not absolute.

Why narrate the step names out loud ("now let me size this")?
?
**Quick:** Interviewers grade structure; narrating signals seniority and keeps you from skipping steps under pressure.<br>
**Deeper:** Interviews are partly observed performance — the interviewer is scoring your process, not just the final diagram. Saying the step name aloud makes your reasoning legible, invites course-correction early, and acts as a checklist that stops you from jumping straight to implementation details and forgetting requirements or API.

What two numbers should you always derive in non-functional reqs?
?
**Quick:** Scale (DAU/QPS) and the read:write ratio — they drive every later decision (cache? replicas? shard?).<br>
**Deeper:** DAU is daily active users and QPS is queries (requests) per second; together they size the load. The read:write ratio tells you whether the workload is read-heavy or write-heavy: read-heavy systems benefit from caching and read replicas, while write-heavy systems push you toward sharding, write-optimized stores, and async processing. These two numbers anchor capacity estimates so design choices are grounded rather than arbitrary.

What's the first thing to do after hearing the prompt?
?
**Quick:** Repeat it back and ask 2–3 clarifying questions to bound scope before designing anything.<br>
**Deeper:** Prompts are intentionally vague to test whether you'll define the problem before solving it. Restating confirms shared understanding, and targeted questions (expected scale, which features matter, who the users are) bound the scope so you don't waste time designing for requirements that don't exist. Jumping straight into boxes is a classic junior tell.
