---
title: Product Architecture Interview
description: A free course on the interface and domain design round: aggregates, contracts, error models, idempotency, pagination, versioning and webhooks.
last_reviewed: 2026-08-01
---

# Product Architecture Interview

!!! abstract "The contract"

    **After this course you can** take a one-line product prompt, derive a domain model and the interface that falls out of it, specify the error, idempotency and pagination semantics a client can be written against without asking you a question, and plan a breaking change that ships while clients you cannot upgrade keep working.

    **Who it is for**

    - A backend or platform engineer whose round is scored on the shape of the contract rather than on the size of the cluster behind it.
    - An engineer who has shipped internal endpoints and is now asked to design a public or partner-facing interface, where every mistake is permanent.
    - A senior candidate whose feedback says "designed a reasonable system, thin on semantics", meaning the happy path was drawn and the failure, retry and versioning behaviour never was.

    **Who it is not for**

    | If your round is about | Go here instead |
    |---|---|
    | Sharding, replication, consensus, queues, multi-region, cluster scaling | [Modern System Design](modern-system-design.md) |
    | Rendering, components, bundles, accessibility, user interface design | [Frontend System Design](frontend-system-design.md) |
    | A model-backed product: ranking, retrieval, labels, drift, serving cost | [ML System Design](ml-system-design.md) |
    | Retrieval-augmented generation, agents, token cost, evaluation of free text | [Generative AI System Design](generative-ai-system-design.md) |
    | Offline sync, background work, battery, version skew on a device | [Mobile System Design](mobile-system-design.md) |
    | Which feature to build, and why, for whom, at what price | Product sense rounds. Module 1 tells you how to spot one in the first minute, and this course does not train for it |
    | A round inside the next week | [Fast-Track in 48 Hours](system-design-fast-track.md) |

    **Trains for:** the 45 and 60 minute API, interface and domain design round in backend, platform, developer-experience and integrations loops, plus the contract portion of senior and staff distributed-systems loops.

    **Does not train for:** coding rounds, product sense rounds, data modelling for analytics warehouses, or behavioural rounds.

    **Fast path:** already comfortable? Go straight to the [self-assessment rubrics](#11-self-assessment-rubrics). Just want the tables? Go to the [reference block](#16-reference-block).

---

## Prerequisites

Seniority is a weak readiness signal for this round in particular, because plenty of staff engineers have never owned an interface they could not change on Monday. Prerequisite mastery is the signal that works. Answer every Quick check in under a minute. Two or more misses means start with the linked material rather than with Module 1.

| You should already be able to | Where to learn it | Quick check |
|---|---|---|
| Say what a response status commits you to, beyond "it failed" | [RFC 9110, HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) | A client receives 409. Is retrying the byte-identical request useful, and what must the body carry for the client to act? |
| Write a schema for a request body with a required field and a closed set of values | [JSON Schema reference](https://json-schema.org/understanding-json-schema/reference/object) | Which keyword marks a property mandatory, and what does adding one new enum value do to a client compiled against last year's schema? |
| Say where a transaction boundary sits and what falls outside it | [PostgreSQL transaction documentation](https://www.postgresql.org/docs/current/tutorial-transactions.html) | One handler inserts a row and calls a third-party gateway. Which of the two can succeed alone, and what does the caller see when it does? |
| Distinguish authentication from authorisation, and name one grant type | [OAuth 2.0, RFC 6749](https://www.rfc-editor.org/rfc/rfc6749) | Service A calls service B with no human present. Which grant fits, and whose identity is inside the token? |
| Say which operations are safe to repeat after a timeout | [The failure library on the hub](index.md#the-failure-library) | Of `POST /payments`, `PUT /users/7` and `DELETE /files/9`, which can be replayed blindly, and what does the odd one out need? |
| Read a percentile rather than an average, and turn it into a timeout | [Google SRE Book, chapter 6, free online](https://sre.google/sre-book/monitoring-distributed-systems/) | A dependency is 900 ms at p99 and your endpoint budget is 300 ms. Name the only two honest options |
| Model a one-to-many relationship and say where the identifier lives | [SQL question bank](/Interview-Questions/SQL-Interview-Questions/) | Comments hang off a post, replies hang off a comment. Which identifier does a reply row carry, and what can drift if it carries both? |
| Say what a pagination cursor encodes | [System design question bank](/Interview-Questions/System-design/) | A cursor holds what, minimally, so that page two does not repeat a row that page one already returned? |

??? note "Show answers to the quick checks"

    1. No. 409 says the request conflicts with the current state of the target resource, so an identical replay conflicts again until something else changes that state. The body has to carry a machine-readable code plus the field or version that conflicted, otherwise the client can only log the string and give up.
    2. The `required` keyword, which takes an array of property names. Adding an enum value breaks any client that validates against a closed set, which is why response enums are an evolution hazard and why clients must be told in writing to treat unknown values as a default branch.
    3. The gateway call is outside the boundary. The row can commit while the call fails, or the call can succeed while the transaction rolls back, so the caller can end up charged with no record or recorded with no charge. The repair shape is an outbox plus reconciliation, and Module 9 builds it.
    4. Client credentials. The identity inside the token is the calling service, not a user, so authorisation has to be expressed as service identity plus scope. Reusing a user-shaped token for machine traffic is the most common auth mistake in this round.
    5. `PUT` and `DELETE` are idempotent by definition: repeating them leaves the same state, though the second `DELETE` typically answers 404. `POST /payments` is not, so it needs an idempotency key supplied by the client and stored with the payment.
    6. Either the operation stops being synchronous (return a job resource and let the client poll or receive a callback), or the budget is renegotiated. Setting a 300 ms timeout and failing does not meet the budget, it just moves the failure to the client. Module 14 and Module 20 cover both routes.
    7. The reply carries the parent comment identifier. Carrying the post identifier as well is a deliberate denormalisation for query cost, and the thing that can drift is agreement between the two: a reply whose post identifier disagrees with its parent's is unreachable by one query path and visible by the other.
    8. The sort key values of the last row returned, plus the sort direction and a fingerprint of the filters, so the next page resumes after a value rather than after a count. Encoding an offset inside a cursor and calling it a cursor keeps every one of the offset failure modes in Module 8.

---

## Time budget

| Part of the course | Reading | Practice | Spaced review |
|---|---|---|---|
| Modules 1 to 4, scope, clock, domain model, exposure style | 80 to 99 min | included below | included below |
| Modules 5 to 8, style selection, schema, errors, pagination | 110 to 138 min | included below | included below |
| Modules 9 to 11, idempotency, concurrency, versioning | 86 to 108 min | included below | included below |
| Modules 12 to 14, auth, quotas, long-running operations | 68 to 85 min | included below | included below |
| Modules 15 to 18, webhooks, batch, clients, AI interfaces | 79 to 100 min | included below | included below |
| Modules 19 to 21, interface as product, operability, levels | 57 to 70 min | included below | included below |
| Eleven case studies, attempted before reading the arc | included above | 165 to 220 min | included below |
| Twelve practice problems, six blocked and six interleaved | included above | 180 to 264 min | included below |
| Two timed self-mocks scored against the public rubric | included above | 90 to 120 min | included below |
| Five review sessions on days 1, 3, 7, 21 and 60 | 0 | 0 | 180 to 240 min |
| **Total** | **480 to 600 min, so 8 to 10 h** | **435 to 604 min, so 7 to 10 h** | **180 to 240 min, so 3 to 4 h** |

!!! note "How these were computed, so you can argue with them"

    Reading is summed per module from the Module Map below, never guessed for the page as a whole. The twenty-one module ranges there total exactly 480 minutes at the low end and 600 at the high end, which is the 8 to 10 hour figure. Per-module minutes assume 120 to 150 words a minute, a deliberately slow engaged rate, because prose carrying schemas, tables and failure cases is not read at prose speed.

    Practice itemises to three components. Eleven case studies at 15 to 20 minutes each, attempted before you read the arc, gives 165 to 220. Twelve practice problems at 15 to 22 minutes each gives 180 to 264. Two timed self-mocks at 45 to 60 minutes each gives 90 to 120. Those sum to 435 and 604 minutes, so 7.3 to 10.1 hours, published as 7 to 10.

    Review assumes five sessions of 36 to 48 minutes on days 1, 3, 7, 21 and 60, which is 180 to 240 minutes. Every figure carries roughly 15 percent padding, because self-estimates of study time run optimistic. At an hour a day this is two to three weeks. Full time it is three to four days.

---

## Learning objectives

Each objective is tested by exactly one numbered item in the mastery check, and the numbering matches: objective 1 is tested by mastery item 1.

1. **Derive** a resource model from a stated domain in under ten minutes, listing entities, aggregate boundaries and at least three invariants, and **name the single invariant that decides where the transaction boundary sits**.
2. **Choose** resource, procedure or event exposure for a stated operation, **citing the client count and the coupling cost that decides it**, and name one condition under which exposing the same domain two ways is correct rather than lazy.
3. **Design** an error model for a stated interface: a code taxonomy, a retryability signal, and partial-failure semantics for a batch call, such that **a client can branch on every class without parsing a human message**.
4. **Specify** idempotent creation for a stated write: the key, its lifetime, the request fingerprint and the transaction boundary, and **name the exact retry sequence that still duplicates** if the boundary is drawn one line too late.
5. **Compare** offset, cursor and keyset pagination on a collection receiving concurrent inserts and deletes, **naming the anomaly each produces** (duplicate, skip, unstable order) and the depth in pages at which offset stops being defensible.
6. **Plan** a breaking change for clients you cannot force to upgrade, giving the compatibility classification, the signalling mechanism, the migration order, and **the measurement that ends the deprecation window** rather than a date chosen in advance.
7. **Rewrite** a mid-level contract answer into a staff-level one by adding a deprecation path, a quota and metering consequence, a webhook delivery guarantee and a blast-radius statement, **annotating what each addition changed about the decision**.

---

## Warm-up retrieval

Three to five minutes. Answer out loud or in writing before opening anything else. Retrieval beats rereading, and a visible answer converts one into the other.

**Q1.** The hub's delivery clock lists six situations where following the clock is wrong. One of them names this round type explicitly. State it, and state the one-line move it tells you to make instead.

??? note "Show answer"

    Estimation is noise for this prompt. API and product architecture rounds, most frontend component rounds and schema-centric prompts are decided by semantics rather than by volume, so two minutes of invented arithmetic is two minutes of visible ritual. The move is [the one-line offer](index.md#the-one-line-offer): say that sizing will not drive this design, name the thing that will, and invite the interviewer to ask for numbers anyway. That offer protects you if they did want the math.

**Q2.** From the trade-off atlas on the hub, several pages back: state the rule for choosing between a representational style, a remote procedure style and events, and name the quantity that decides it.

??? note "Show answer"

    - Representational when the surface is public or partner-facing, reads are cacheable, and there are many clients you do not control.
    - Procedure style when the call is internal, latency-sensitive and high volume, with a schema both ends own.
    - Events when the producer must not know its consumers, and consumers are allowed to lag.

    The deciding quantities are the count of clients you control, plus per-call payload size and rate. The named failures: events for a request that needs an answer creates a correlation-identifier maze, and procedure calls between two public parties export your internal model to strangers.

**Q3.** The hub works a payments idempotency prompt as its example of skipping estimation. It concludes that exactly one number still deserves to appear. Name that number and say which option it kills.

??? note "Show answer"

    The client's maximum retry horizon, which sets the retention window for idempotency keys. It kills the "keep keys forever" option, because an unbounded key table degrades its own uniqueness index, and it simultaneously kills a very short window, because a client retrying after a long outage would then be charged twice. Storage volume killed nothing, since the key set is bounded by the payment set you already agreed to store. That is the estimation rule working: the number earns its place because an option dies.

---

## Module map

```
+---------------------------------------------------------+
| MODEL                                       M1 to M4    |
| round type and scope, the contract clock, entities,     |
| aggregates and invariants, resource or procedure or     |
| event as three ways to expose one domain                |
+----------------------------+----------------------------+
                             |
                             v
+---------------------------------------------------------+
| CONTRACT                                    M5 to M8    |
| style selection, schema as the design artefact,         |
| error taxonomy and partial failure, pagination on a     |
| collection that changes while you read it               |
+----------------------------+----------------------------+
                             |
              +--------------+--------------+
              v                             v
+-----------------------------+ +-------------------------+
| CORRECTNESS      M9 to M11  | | POLICY      M12 to M14  |
| idempotency keys, entity    | | grants and scopes,      |
| tags and lost updates,      | | quotas and limits,      |
| versioning and deprecation  | | long-running operations |
+--------------+--------------+ +------------+------------+
               |                             |
               +--------------+--------------+
                              v
+---------------------------------------------------------+
| DELIVERY                                   M15 to M18   |
| webhooks and event delivery, batch partial success,     |
| client and network constraints, AI tool contracts       |
+----------------------------+----------------------------+
                             |
                             v
+---------------------------------------------------------+
| OPERATE AND JUDGE                          M19 to M21   |
| interface as product, per-operation indicators and      |
| resilience budgets, level calibration                   |
+---------------------------------------------------------+

Caption: six clusters and their reading order. Model feeds
Contract, because endpoints are derived from a domain rather
than guessed. Contract feeds two parallel clusters:
Correctness, which is what a retrying client does to you, and
Policy, which is what you charge and permit. Both feed
Delivery, the outbound and bulk surfaces. Everything feeds
Operate and Judge, where mid and staff answers separate.
Arrows point from prerequisite to dependent.
```

| Module | What you will be able to do | Time | Skip if |
|---|---|---|---|
| M1 Scope and honesty | Tell inside the first minute whether you are in an interface design round, a product sense round or a distributed systems round wearing this title, and steer back | 15 to 18 min | You have the round format in writing and it matches what you were told |
| M2 The contract clock | Spend your minutes the way this round rewards, with modelling and semantics ahead of boxes and arrows | 15 to 18 min | You have run a timed self-mock on a contract prompt and finished with error semantics stated |
| M3 Domain modelling first | Turn a prompt into entities, aggregates and invariants, and put the transaction boundary where the invariant demands | 30 to 38 min | Never skip. Every case study on this page starts here, and a guessed endpoint list is the most common first failure |
| M4 Resource, procedure or event | Expose one domain three ways and defend the one you picked on client count and coupling | 20 to 25 min | You have shipped the same domain behind two exposure styles and can say why |
| M5 API style selection | Compare representational, graph and binary styles on caching, fetch shape, streaming, tooling and reach, and name each one's characteristic production failure | 28 to 35 min | You can already state the query-cost failure of a graph style and the browser-reach failure of a binary one |
| M6 Contract first | Treat the schema as the design artefact, generate from it, lint it, and state your compatibility rules at schema level | 22 to 28 min | You run schema linting and generation in continuous integration today |
| M7 Error model design | Design a code taxonomy, signal retryability and retry-after, and give batch operations honest partial-failure semantics | 32 to 40 min | Never skip. This is where candidates most visibly fail and it is the cheapest module to convert into signal |
| M8 Pagination | Compare offset, cursor and keyset under concurrent writes, price deep pagination, and decide about total counts | 28 to 35 min | You have debugged a duplicate-row complaint on a paginated feed and can name the anomaly class |
| M9 Idempotency and exactly-once | Specify keys, lifetime, fingerprinting and the transaction boundary, and name the retry that still duplicates | 32 to 40 min | You have implemented an idempotency key store against a payment surface |
| M10 Concurrency and caching in a contract | Use entity tags and conditional requests to make optimistic locking part of the interface, and close the lost update | 24 to 30 min | You can write the exact request and response pair that rejects a stale update, from memory |
| M11 Versioning, evolution, deprecation | Classify a change as breaking or not, signal a deprecation, migrate clients you cannot force to upgrade, and end the window on a measurement | 30 to 38 min | You have run a deprecation to zero traffic on a public surface |
| M12 Authentication and authorisation | Pick a grant for a stated caller, set token lifetimes and rotation, and choose between scopes, roles and attributes | 26 to 32 min | You have designed service-to-service auth and multi-tenant isolation for a real surface |
| M13 Rate limiting and quotas | Compare algorithms on burst behaviour and coordination cost, place limits per key, tenant and operation, and connect quota design to the bill | 22 to 28 min | You operate a quota system and can state what your headers tell a client to do |
| M14 Long-running operations | Model work that outlives a request as a job resource, with progress, cancellation and result retention | 20 to 25 min | You have shipped an operation resource with a cancellation path |
| M15 Webhooks and event delivery | Sign, replay-protect and retry outbound deliveries, define the guarantee, and give consumers a debugging surface | 28 to 35 min | You run an outbound webhook system with a dead-letter path and a redelivery tool |
| M16 Batch, bulk and streaming | Give a bulk call partial-success semantics a client can act on, and price a chatty interface honestly | 15 to 19 min | You can state your bulk endpoint's per-item error shape without looking |
| M17 Client constraints as design input | Shape payloads for slow networks with delta sync and sparse field selection, and say what chattiness costs a phone | 12 to 16 min | You have designed for a client on a hostile network and measured the bytes |
| M18 AI-era interfaces | Design tool schemas as contracts, declare side effects, stream and cancel, and meter cost per call | 24 to 30 min | You have shipped a tool-calling surface with approval gates and per-tenant metering |
| M19 Interface as product | Cut time to first call, design keys and rotation, and communicate a deprecation to people who did not ask for it | 15 to 18 min | You own developer experience metrics for a public interface today |
| M20 Operability and resilience budgets | Define per-operation indicators, propagate deadlines, and set timeout, retry and shedding budgets that agree with your service levels | 24 to 30 min | You can derive a client retry budget from a stated objective without notes |
| M21 Level calibration | Say what your answer is missing to read one level higher | 18 to 22 min | Never skip if you are targeting senior or above |

Low ends total 480 minutes, high ends total 600. That is the 8 to 10 reading hours in the time budget above.

---

## The delivery clock for this round

Most candidates who fail a contract round did not miss a component. They drew a service diagram, listed six endpoint paths, and never said what a client receives when a call times out, arrives twice, or is written against last year's schema. This clock is a budget rather than a script, and it is shaped differently from a generalist one: modelling and semantics take the minutes that a distributed systems round spends on capacity and topology.

One structural difference worth naming immediately: there is no estimation phase. The hub's clock allows two to three minutes for a number that eliminates an option, and in this round that number usually does not exist, for the reason worked through in [the honest skip](index.md#worked-estimate-two-the-honest-skip). Offer it, do not perform it.

### The 45 minute round

| Minutes | Spend it on | You leave this phase with |
|---|---|---|
| 0 to 2 | Restate the prompt as a domain plus its consumers, and name who cannot be upgraded | The interviewer agreeing, or correcting you cheaply |
| 2 to 8 | Clarify: who the clients are, which are external, what is already public, what must never break | Two or three constraints written down and tagged GIVEN or ASSUMED |
| 8 to 16 | The domain model: entities, aggregate boundaries, invariants, and where the transaction sits | A model on the board that an endpoint list can be derived from |
| 16 to 24 | v0 of the contract: the fewest operations the requirements permit, one happy path end to end | A working surface, deliberately missing its hard semantics |
| 24 to 32 | The semantics that break v0: errors, retries and idempotency, and pagination where a collection grows | Named failure behaviour for every operation you drew |
| 32 to 40 | One deep dive, ideally the one the interviewer picks: versioning, webhooks, auth or quotas | Depth in a single area, with the trade-off stated both ways |
| 40 to 45 | Evolution and close: what a breaking change would look like, what you would measure first, what you skipped | An explicit list of what you did not cover and why |

The phases sum to 45 by construction: 2 plus 6 plus 8 plus 8 plus 8 plus 8 plus 5.

### The 60 minute round

| Minutes | Spend it on |
|---|---|
| 0 to 2 | Open and restate as a domain with named consumers |
| 2 to 9 | Clarify, including which clients are unupgradable and which surface is already public |
| 9 to 19 | Domain model: entities, aggregates, invariants, transaction boundaries |
| 19 to 28 | v0 of the contract, happy path only, fewest operations |
| 28 to 38 | Error model, retry semantics and idempotency, worked on one real operation |
| 38 to 46 | Evolution: compatibility classification, versioning scheme, deprecation signalling |
| 46 to 56 | One or two deep dives, different in kind from each other |
| 56 to 60 | Close: trade-offs, what you skipped, what you would measure first |

These sum to 60: 2 plus 7 plus 10 plus 9 plus 10 plus 8 plus 10 plus 4. The extra 15 minutes buys a real evolution block and a second dive. It does not buy more endpoints.

### The first two minutes

Say four sentences, in this order. Rehearse them once so they are automatic and your attention is free for listening.

1. "Let me restate it as a domain: the nouns are X and Y, the operations people care about are A and B, and the consumers are Z." (If you cannot fill the consumer slot, that is your first clarifying question.)
2. "I am going to spend about eight minutes on the model and the invariants before I write a single path, because the endpoints fall out of that." (You just told them your clock, and you justified it.)
3. "I will treat the error and retry semantics as part of the design, not as a detail at the end." (Cheapest senior signal available in this round, and almost nobody says it.)
4. "Stop me if you want depth somewhere specific rather than coverage." (Interviewers take this invitation more often than candidates expect.)

### Declaring what you are skipping

Skipping silently reads as not knowing. Skipping out loud reads as prioritising. The pattern: name it, say why it cannot change a decision in the next twenty minutes, say in one sentence what you would do if it were in scope.

- "I am not designing the storage engine behind this. Every option we have discussed holds the same rows, so the choice cannot change the contract, which is what the round is about."
- "I am parking the graph style. It is a real option, but with two internal consumers and one partner it buys flexibility nobody asked for and costs us a query-cost problem. Flag me if you want it and I will come back with the comparison."
- "I am leaving the sandbox and key-issuing flow out. It matters commercially and it is where developer experience is won, but it does not change the resource model or the error semantics we are working on."

### Checking in without sounding unsure

A weak check-in asks for approval. A strong one offers a fork. Two per 45 minute round is about right, one after the model and one before the deep dive.

| Weak, reads as unsure | Strong, reads as driving |
|---|---|
| "Does the model look right?" | "That is the aggregate boundary. Do you want the write operations next, or the invariant that decides the transaction?" |
| "Should I cover errors?" | "I have assumed clients retry automatically on 5xx. Correct me now if they do not, because it decides whether creation needs an idempotency key." |
| "Is this enough detail?" | "I can take pagination to the concurrent-insert failure modes, or stay wide and cover auth and quotas. Which is more useful?" |
| "Sorry, I am going long on modelling." | "I am two minutes over on the model. Moving to the contract, and I will come back to the invariant on line three." |

### How the split shifts by level

| Phase | Mid | Senior | Staff |
|---|---|---|---|
| Clarify | Same | Same | Longer: asks who owns the surface and what the support cost of a mistake is |
| Domain model | Entities named | Aggregates with invariants stated | The boundary argued against a second plausible one, with the migration cost of getting it wrong |
| Contract v0 | Most of the time here | Balanced | Compressed, because a correct surface is assumed |
| Error and retry semantics | Status codes named | A taxonomy with retryability and partial failure | Written as a client contract, including what an automated client does at 3am |
| Versioning | Mentioned as a version prefix | A compatibility classification with a deprecation plan | The whole answer is framed around clients that cannot be upgraded |
| Quotas, metering, cost | Absent | Rate limits placed and headers named | Quota design tied to pricing, fairness and the bill |
| Operability | A log line | Per-operation indicators | Deadline propagation, retry budgets, shedding order and who gets paged |

The shape: mid level spends its minutes proving a usable interface could exist, senior spends them proving the semantics were chosen rather than defaulted, staff spends them proving the interface survives five years of clients, integrations and pricing changes it does not control.

!!! tip "One note on frameworks, made once"

    Paid interview courses often ship an acronym spine and apply it identically to every case study. Those acronyms belong to their publishers, not to this page. The problem is not the letters, it is that one spine applied unchanged to every prompt produces an answer that sounds recited, and a recited answer gives a grader nothing to grade.

    The problem is not the letters. It is that one spine applied to nine prompts produces an answer that sounds recited, and interviewers who run these rounds weekly screen for that tell. The clock above is a time budget. It never appears as a heading in any case study on this page, and Module 2 teaches when to abandon it.

!!! danger "WHEN TO BREAK THIS"

    The measurement that justifies following the clock: you have run one timed self-mock and finished with a resource model, stated error semantics and a versioning answer, with time left. Below that, the clock is scaffolding you still need. Above it, treat it as advisory. Three situations specific to a contract round where following it is the wrong move:

    **1. The interviewer hands you an existing schema.** "Here is our current interface, we need to add tiered pricing without breaking anyone" has no v0 to draw, because v0 shipped three years ago. The minutes go to the compatibility classification, what is observably in use, the migration order, the dual-serving window, and how the deprecation ends. Drawing a greenfield resource model over a live public surface is the fastest way to fail a staff round in this track.

    **2. The prompt is one operation, not a service.** "Make payment creation idempotent" or "design the error model for this batch endpoint" is a single-block round wearing a design title. There is no model phase and no contract phase to run. Spend the whole clock on the one surface: the key, the fingerprint, the transaction boundary, the retry that still duplicates, and what the client sees in each case. Narrating a scoping phase here reads as a script running against a question nobody asked.

    **3. The round turns out to be a different round.** This happens two ways, and both are recoverable if you name the mismatch in one sentence and ask which they want.

    If minute four is "how would you shard this, and what happens in a region failover", you are in a distributed systems round wearing this title. Say so out loud, switch, and use [Modern System Design](modern-system-design.md) as your material. If minute four is "which of these two features would you build first, and how would you measure it", you are in a product sense round, and no amount of resource modelling rescues it.

    A quieter fourth: read who is in the room. A platform or developer-experience engineer will follow you into error taxonomies, deprecation and time to first call. An integrations engineer will want webhooks, replay and consumer idempotency. Both are fair. Rebalance the deep dives towards whichever they lean into, and say out loud that you are doing it.

## Module 1. Scope and honesty: what this round actually asks for

**Time: 15 to 20 minutes reading, 10 to 15 minutes practice.**

### 1a. Predict first

Three prompts, all taken from the same day of the same loop. Read them before anything else on this page.

| Round | The prompt, as the interviewer says it |
|---|---|
| A | "You have an hour. Design a photo sharing service for 300 million users." |
| B | "Design the interface a third party developer would use to post photos on a user's behalf." |
| C | "Photo upload failures are our top support driver. What would you build next quarter, and how would you know it worked?" |

??? note "Show answer"

    Prediction asked for: which one is this course, and what single artefact does each interviewer want visible at minute 50?

    | Round | Artefact wanted at the end | Scored on | This course |
    |---|---|---|---|
    | A | A box diagram plus capacity numbers | Storage, replication, fan-out, failure modes | No, take [Modern System Design](modern-system-design.md) |
    | B | A domain model plus an operation list with request and response shapes | Aggregates, invariants, errors, idempotency, evolution | Yes, this is the centre |
    | C | A prioritised bet plus a measurement plan | Problem selection, metric choice, sequencing | No, this is a product sense round |

    One of three. The tell is the noun in the prompt. Prompt A names a *system*, prompt B names an *interface*, prompt C names an *outcome*.

    The expensive confusion is between A and B, because both sound like architecture. In A the interesting question is what happens when one machine is not enough. In B the interesting question is what a caller you have never met sends you, and what you send back when it goes wrong. Those are different rubrics with almost no shared vocabulary.

### Three rounds hide behind one course title

The phrase "product architecture interview" is used by different companies for at least three different things. Preparing for the wrong one is the most expensive mistake available here, because it costs weeks and you will not discover it until minute five.

| What the round really is | Opening line sounds like | The hard step | Where it is taught |
|---|---|---|---|
| Interface and domain design | "Design the API for ..." or "Model the domain for ..." | Deciding what the entities are and what an operation must never allow | This page |
| Distributed system design | "Design X for N million users" | Deciding what breaks at what number | [Modern System Design](modern-system-design.md) |
| Product sense plus loose architecture | "Users complain about X, what would you build?" | Deciding what is worth building at all | Not here, and the header says so |

**How to tell inside 90 seconds.** Listen for what the prompt says already exists.

- A consumer exists and is external ("a partner", "a mobile app we do not control", "a third party"): interface design.
- A user count exists and no consumer is named: distributed system design.
- A complaint or a metric exists: product sense.
- Nothing exists and the interviewer says "start wherever you want": ask. One sentence buys you forty minutes.

### The title history, once, and then we move on

This course sits under a title that several publishers use, and the equivalent paid course was previously published under an *API Design Interview* title, wording that still survives in places on its own page. That history is why two syllabi that look unrelated by name cover the same ground. The hub documents the publisher-by-publisher picture with dated primary links in [the naming section](index.md#the-naming-confusion-explained).

The practical consequence for you is not commercial, it is about preparation:

- If a recruiter says "product architecture", assume interface and domain design until told otherwise, because that is what the syllabus under that title has historically contained.
- If a recruiter says "API design", it is the same round. Do not prepare twice.
- If the interviewer opens with a user count and no named consumer, you are in a distributed systems round wearing the wrong name. Say so and steer, using the script in module 2.

### What a contract round is actually scored on

Six things, in rough order of how often they decide the rating.

| Signal | What a strong answer shows | What a weak answer shows |
|---|---|---|
| Domain model | Named aggregates with stated invariants | A list of endpoints invented from the prompt's nouns |
| Operation shape | Each operation maps to a state change the domain allows | Verbs bolted on when the resource model runs out |
| Error semantics | A taxonomy, retryability, and partial failure rules | A shrug and "return 400" |
| Idempotency | A key, a lifetime, and a conflict rule | "The client can retry" |
| Evolution | A named breaking-change policy and a deprecation path | "We would version it" |
| Authorisation | Who can call what, expressed against the domain model | Bearer tokens mentioned once |

The rows are not equally weighted by accident. Domain model and error semantics carry the most, because they are the two a candidate cannot fake by recall.

### The routing diagram

```
              first line of the prompt
                        |
        +---------------+---------------+
        |               |               |
   a consumer      a user count     a complaint
   is named        is given         or a metric
        |               |               |
        v               v               v
+--------------+ +-------------+ +--------------+
| interface    | | distributed | | product      |
| and domain   | | system      | | sense        |
| design       | | design      | | round        |
+--------------+ +-------------+ +--------------+
        |
   public or partner consumer
        |
        v
+--------------------------+
| evolution and versioning |
| become half the round    |
+--------------------------+

Routing the first line of a round to one of three rubrics, and
the extra weight a public consumer adds to a contract round
```

### 1d. Worked example

The recruiter email says: "Onsite for Senior Software Engineer, Platform. Four rounds: Coding (45), Product Architecture (60), Distributed Systems (60), Behavioural (45). The Product Architecture interviewer maintains our public developer platform."

**Block 1. PURPOSE: convert the round list into deliverables, because I rehearse deliverables and not topics.**

Three technical rounds, three different artefacts: working code, a domain model plus a contract, a box diagram plus numbers. I write those three phrases at the top of my plan. Anything I study has to serve one of them, and material that serves none gets cut.

The strongest signal in the whole email is "maintains our public developer platform". A public platform means external callers I cannot upgrade, which promotes versioning, deprecation and error contracts from nice to have into likely probes.

**The error most readers make here** is treating the two 60 minute rounds as one topic and preparing shared material for both. They share almost nothing. One is about aggregates and contracts, the other is about partitions and failure.

!!! note "Say it before you read on"

    Name one design question that would be interesting in the Distributed Systems round and boring in the Product Architecture round. Say it out loud before continuing.

**Block 2. PURPOSE: decide what I will deliberately not prepare, because preparation is subtraction.**

Given a public platform, I drop three things from the contract round entirely: consensus algorithms, cache topology, and anything about cluster sizing. Those belong to the other round and I have already booked time for them there.

That frees my contract round time for five things: aggregate design, error taxonomy, idempotency, pagination stability, and a deprecation path for a client I cannot force to upgrade.

**The error most readers make here** is additive preparation. They keep the general system design syllabus and add contract material on top, then arrive having half learned both.

**Block 3. PURPOSE: buy information cheaply, because two sentences of recruiter reply beat two hours of guessing.**

I send one message: "For the Product Architecture round, is the prompt usually a greenfield interface, or an existing public interface being changed? And should I assume external third party callers or internal service callers?"

Both answers move real hours. Greenfield plus external means I rehearse domain modelling and versioning. Brownfield plus external means I rehearse the breaking change migration path first, because that will be the whole round.

!!! note "Say it before you read on"

    Which of those two answers would cause you to change the *first ten minutes* of your rehearsed opening, and how?

**Block 4. PURPOSE: rehearse the opening, because it is the only fully predictable forty seconds of the round.**

I write and say out loud: "Before I draw any operations I want to spend about eight minutes on the domain: what the entities are, which of them can change independently, and what must never be true. Then I will pick an interface style and sketch six operations, then spend real time on errors and retries. If you would rather I go deep on one area instead, tell me now."

**The error most readers make here** is improvising the opening and then listing endpoints in minute two, which locks in a resource model nobody has justified.

**Result.** The reply was "existing public interface, external callers, and she usually asks how you would remove something". That moved the last two hours of preparation from resource modelling drills to deprecation mechanics, which is exactly where the round went.

### 1e. Fade the scaffold

**Faded example 1.** The email says: "Onsite, Backend Engineer, Payments. Rounds: Coding (45), Product Architecture (60), Behavioural (45). The Product Architecture interviewer works on the merchant integrations team." Blocks 1 to 3 are given. Produce block 4 yourself.

- Block 1: deliverables are code and a contract. Only one design round, so it carries all the design weight.
- Block 2: I drop general resource modelling breadth and concentrate on money semantics: idempotency, retries, reconciliation, webhooks.
- Block 3: I ask whether the prompt is the merchant facing interface or the internal ledger interface.

??? note "Show answer"

    A defensible block 4, shaped by the fact that a payments caller retries and cannot tolerate a double charge:

    "I want to start with the domain, and specifically with what a payment attempt is as distinct from a payment, because that distinction is what makes retries safe. I will spend eight minutes there, then sketch the operations, then spend real time on idempotency keys and on what I return when the downstream gateway times out. Redirect me if you want the ledger side instead."

    The thing that earns credit is naming the attempt versus payment distinction in the first minute. It is the fastest available signal that you have built a payments interface rather than read about one.

**Faded example 2.** The email says: "Virtual onsite, Software Engineer, Developer Platform. Rounds: Product Architecture (60), Distributed Systems (60), Coding (45). The company published a v2 of its public API eighteen months ago and v1 is still live." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: three deliverables. The v1 and v2 detail in the email is not decoration, it is the interviewer's daily problem.
- Block 2: I drop greenfield only material. I keep the breaking change taxonomy, deprecation signalling, and client migration curves.

??? note "Show answer"

    Block 3: the cheap purchase is about the shape of the prompt, not the topic. I ask: "Is the design round usually greenfield, or do candidates get an existing interface plus a change that must ship?" A brownfield prompt means my rehearsal is a migration plan rather than a resource model, and that single answer redirects perhaps a third of the remaining hours.

    Block 4: an opening that works either way. "I will start by writing down the entities and the invariants, because whether this is new or existing, the thing I must not break is an invariant rather than a URL. Then I will sketch operations. If there is an existing contract in play, tell me early, because then I want to spend most of the hour on what changes and how the old callers keep working."

    Performing the greenfield or brownfield selection out loud is the point. In a platform round the selection is half the difficulty.

### 1f. Check yourself

**Q1.** An interviewer opens with: "Our partners integrate with our booking API. Design it." You have 60 minutes. Which of the six scored signals in this module should get the largest single block of your time, and why?

??? note "Show answer"

    Domain model, roughly a fifth of the working minutes. "Partners" means external callers, and every later decision (which operations exist, which errors are possible, what a retry means) is derived from what the entities are and which invariants must hold.

    The second largest block goes to error semantics and idempotency together, because a booking is a non repeatable side effect and partners retry on timeout. If you spend the largest block on the URL layout instead, you have optimised the least load bearing part of the answer.

**Q2.** Ten minutes into a round titled "product architecture", the interviewer asks how you would shard the storage for 40 terabytes of images. What just happened, and what do you do?

??? note "Show answer"

    The round is not the one the title promised, or the interviewer is checking breadth before returning to the contract. Both are common and neither is a trap.

    Do two things. First, answer the question briefly and concretely rather than deflecting, because refusing to engage reads worse than a merely adequate sharding answer. Second, name the boundary out loud: "I can go deeper on storage layout if you want, though I had assumed the interesting part here is the contract the partners see. Which would you rather I spend the next fifteen minutes on?" That sentence hands the routing decision back to the only person who knows the rubric.

### 1g. When not to use this

Round triage is a preparation tool, and it is over applied far more often than under applied.

| Question | Answer |
|---|---|
| The measurement that justifies doing triage at all | You cannot name the artefact any one of your booked rounds wants at minute 50 |
| Threshold below which it is over-engineering | A single round with a written description that already names the format. Triage adds nothing to a description that already says "you will design an API" |
| The cheaper alternative | One message to the recruiter naming the two things you cannot infer. The reply is usually exact and arrives within a day |
| Failure mode of over-applying it | Spending study hours on meta strategy instead of on modules 3 to 11. The entire triage exercise is worth about thirty minutes |

---

## Module 2. The delivery clock for a contract design round

**Time: 15 to 20 minutes reading, 10 to 15 minutes practice.**

### 2a. Predict first

Two candidates get the same 60 minute prompt: "Design the interface a partner uses to manage bookings." Here is how each spent the working minutes.

| Phase | Candidate A | Candidate B |
|---|---|---|
| Scope and clarify | 4 | 5 |
| Domain model and invariants | 2 | 10 |
| Style choice and resource list | 6 | 3 |
| Operation sketch | 26 | 13 |
| Errors and idempotency | 3 | 7 |
| Deep dive chosen by interviewer | 8 | 8 |
| Evolution and deprecation | 1 | 4 |
| Operations and wrap | 2 | 2 |

??? note "Show answer"

    Prediction asked for: both spent 52 working minutes. Which one rates higher, and what specifically did the loser's extra 13 minutes of operation sketching buy?

    Candidate B rates higher, and the interesting part is that A's extra sketching time bought negative value. With two minutes of domain modelling, A entered the sketch without an aggregate list, renamed resources twice mid draw, and produced operations whose invariants were never stated. Time spent drawing a model you have not justified is time spent building something you will later have to defend and cannot.

    The general shape: in a contract round the expensive thinking happens before the first operation is written, and the credit earning thinking happens after the last one. The middle is transcription.

### The budget

The hub owns the shared clock and the working minute deductions ([the clock](index.md#the-clock)). This module only states how a contract round differs, which it does in two ways: there is no capacity estimation phase, and there is a phase the other tracks do not have at all, which is evolution.

Working minutes are 40 in a 45 minute slot and 52 in a 60 minute slot, using the hub's deduction for introductions and closing questions.

| Phase | What ends the phase | 45 min slot | 60 min slot |
|---|---|---|---|
| Scope and consumer | You have named the caller and whether you control them | 4 | 5 |
| Domain model | Aggregates listed, invariants stated out loud | 8 | 10 |
| Style and resource list | One style chosen with a reason, resources named | 2 | 3 |
| Operation sketch | Four to six operations with request and response shapes | 10 | 13 |
| Errors and idempotency | A taxonomy plus a retry rule for every unsafe operation | 6 | 7 |
| Deep dive | One area taken to field level, chosen by the interviewer | 6 | 8 |
| Evolution | A breaking change policy and one migration story | 3 | 4 |
| Wrap | What you skipped, what you would measure first | 1 | 2 |
| **Total** | | **40** | **52** |

The operation sketch number is derived, not chosen. Writing one operation with its path, its request fields, its response fields and its failure cases takes roughly two minutes at whiteboard speed. Six operations therefore cost about twelve to thirteen minutes, which is what the table allocates. If you find yourself needing eleven operations, you have not found the aggregates yet.

```
+-------+----------+-----+-----------+--------+------+-----+--+
| Scope | Domain   |Style| Operations| Errors | Dive | Evo |Wr|
| 5     | 10       | 3   | 13        | 7      | 8    | 4   |2 |
+-------+----------+-----+-----------+--------+------+-----+--+
0       5          15    18          31       38     46   50 52

Working minutes in a 60 minute contract round. The two blocks
either side of the operation sketch are worth more than the
sketch itself
```

### The first two minutes of a contract round

Four sentences, in this order, before you draw anything.

1. Name the consumer: "I am assuming an external partner we cannot force to upgrade. Correct me if it is an internal caller."
2. Name the boundary: "I will design the contract and the domain behind it, not the storage layout, unless you want that."
3. Name the order: "Entities and invariants first, then operations, then errors and retries, then how it changes over time."
4. Offer the trade: "If you would rather I go deep on one of those, say so now and I will drop the others."

Sentence one is the highest value sentence in the round. Internal caller and external caller answers diverge within ten minutes on versioning, error verbosity and authorisation, and guessing wrong wastes half the hour.

### How the split moves by level

Same 52 working minutes, reallocated. Read the columns, not the cells.

| Phase | Mid | Senior | Staff |
|---|---|---|---|
| Scope and consumer | 4 | 5 | 8 |
| Domain model | 8 | 10 | 12 |
| Style and resource list | 4 | 3 | 2 |
| Operation sketch | 18 | 13 | 9 |
| Errors and idempotency | 6 | 8 | 7 |
| Deep dive | 8 | 8 | 8 |
| Evolution | 2 | 4 | 4 |
| Wrap | 2 | 1 | 2 |

A mid level answer is judged on whether a coherent, complete contract appears. A senior answer is judged on the error and retry semantics and on whether the model survives a hostile follow up. A staff answer spends its budget at the two ends: negotiating what the interface should even own, and stating how it will be evolved and operated.

### When to break the clock

Three situations where following the budget is the wrong move.

| Situation | What to do instead | Why |
|---|---|---|
| The interviewer hands you an existing contract and a required change | Collapse scope, domain and style into six minutes and spend twenty on migration | The round is a brownfield evolution round. The model is given |
| The interviewer interrupts the sketch with "what happens if that call times out" | Abandon the remaining operations and go there immediately | They have told you the rubric. Finishing your list instead is the most common way to fail while appearing organised |
| You are twelve minutes in and still arguing about what an entity means | Stop, pick one meaning, write "assumption" next to it, move on | An unresolved definition costs the rest of the hour. A wrong but declared assumption costs nothing if you flag it |

### 2d. Worked example

The prompt: "Design the interface a hotel partner uses to manage room availability and bookings." 60 minutes.

**Block 1. PURPOSE: spend the first five minutes buying constraints, because every later phase is priced by the answers.**

I ask four questions and nothing else: is the partner external and unupgradable, how many partners are there, does availability change from our side as well as theirs, and is a booking cancellable after confirmation.

The answers come back: external, roughly 800 partners, availability changes from both sides, and cancellation is allowed within policy windows. I write those four facts on the board where the interviewer can see them.

**The error most readers make here** is asking questions whose answers do not change the design, for example the expected request volume. In a contract round throughput rarely forks the interface. Bidirectional mutation always does.

!!! note "Say it before you read on"

    Which of those four answers most changes the domain model, and which most changes the operation list? They are not the same answer.

**Block 2. PURPOSE: build the domain before the operations, because the operation list is a consequence and not a choice.**

I name three aggregates: Property, RatePlan, Booking. I state one invariant per aggregate out loud, for example "a Booking never overlaps another Booking on the same room for the same night".

Then I say the sentence that earns the round: "Because availability changes from both sides, the invariant cannot live only in the partner's system, so the operation that consumes inventory has to be ours and has to be conditional."

**The error most readers make here** is jumping from nouns to URLs. "Property, rate plan, booking" becomes three collection paths and the invariant is never mentioned, so the conditional write requirement never surfaces.

**Block 3. PURPOSE: sketch only the operations the invariants require, and stop.**

Five operations, about two minutes each: list properties, replace a rate plan for a date range, read availability, create a booking conditionally, cancel a booking. I write request and response fields for the conditional create only, and say I will expand others on request.

I explicitly do not add search, reporting or a webhook here. I name them as deliberately out of scope in one sentence, so it is clear they were considered rather than forgotten.

!!! note "Say it before you read on"

    Why did the conditional create earn full field detail while the other four got one line each?

**Block 4. PURPOSE: spend the errors block on the two failure cases the interviewer will probe.**

The two are: the partner retries a create after a timeout, and the partner writes availability that conflicts with a booking we already accepted. I give the first an idempotency key with a 24 hour lifetime, and the second a conditional write with a version token plus a specific conflict error.

**The error most readers make here** is describing errors as a status code list rather than as answers to "what does the caller do next".

**Result.** With five minutes left I stated what I skipped (search, reporting, webhooks), named one thing I would measure first (conflict errors per partner per day, because a partner with a rising conflict rate is about to file a support ticket), and answered two follow ups. The interviewer spent the whole deep dive on conditional writes, which is where block 2 had pointed.

### 2e. Fade the scaffold

**Faded example 1.** Prompt: "Design the interface an e-commerce seller uses to manage a product catalogue." 45 minutes, so 40 working minutes. Blocks 1 to 3 are given. Produce block 4.

- Block 1: four questions bought: external sellers, roughly 50,000 of them, images uploaded separately, and catalogue changes are reviewed before going live.
- Block 2: aggregates are Product, Variant, Listing. Invariant: a Listing is never live while its Product is in review.
- Block 3: five operations sketched, with full field detail only on the bulk upsert.

??? note "Show answer"

    Block 4 must be driven by the two facts that block 1 bought: 50,000 sellers and an asynchronous review step.

    The two probable probes are what a seller receives when a bulk upsert partially succeeds, and how a seller learns that a listing left review. So the errors block covers partial success (a per item result array with a stable item identifier and a per item error code, and an overall status that is not "success") and the review transition (a job resource the seller polls, with an explicit terminal state).

    The move that earns credit is refusing to return a single top level failure for a bulk operation. With 50,000 sellers upserting catalogues, an all or nothing bulk operation converts one bad row into a retry loop over thousands of good rows.

**Faded example 2.** Prompt: "Design the interface an internal fraud service exposes to our checkout service." 60 minutes. Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: internal caller, one consumer today and two planned, synchronous, and checkout has a 300 millisecond budget for the whole call.
- Block 2: one aggregate, Assessment. Invariant: an Assessment is immutable once returned, so a later re-score is a new Assessment.

??? note "Show answer"

    Block 3: with one internal consumer and a hard 300 millisecond budget, the operation list is deliberately tiny: create an assessment, fetch an assessment by id, and nothing else. Field detail goes on the create, and the response carries a decision plus a reason code plus the assessment id, because checkout needs to log why. Notice what the internal caller answer removes: no pagination, no partial success, no versioning ceremony.

    Block 4: the interesting failure is not a bad request, it is a slow one. The contract must state what checkout does when fraud does not answer inside the budget, so the error block covers a deadline conveyed by the caller, a fail open or fail closed default stated explicitly, and an immutable assessment id that makes the retry safe to repeat. A candidate who writes a rich twelve code error taxonomy for a single internal consumer has misread the round.

### 2f. Check yourself

**Q1.** You are 30 minutes into a 60 minute contract round and you have five operations sketched but have not said the word "retry". What do you cut to make room, and what do you say while cutting?

??? note "Show answer"

    Cut the remaining operations, not the errors block. Say it out loud: "I have three more operations in mind, list, delete and export, and they are all shaped like the ones I have drawn. I would rather spend the remaining time on what happens when create times out, because that is where this interface can actually lose money."

    Naming the cut is the whole move. Silently dropping the operations reads as running out of material. Announcing the trade reads as prioritisation, and it is the same twelve minutes either way.

**Q2.** The interviewer says nothing for the first fifteen minutes, then asks only about authorisation for the remaining forty. Which phases of the budget do you sacrifice, and which do you protect?

??? note "Show answer"

    Sacrifice the operation sketch and evolution. Protect the domain model, because authorisation is expressed against it: without aggregates you cannot say who may change what, and you end up listing token types instead of permissions.

    The strong recovery sentence bridges from what you already drew: "Permissions here attach to the Property aggregate rather than to the endpoint, so a partner scoped to one property cannot reach another property's bookings even through the shared search operation." That sentence only exists if the domain block happened.

### 2g. When not to use this

| Question | Answer |
|---|---|
| The measurement that justifies rehearsing a clock | You have run a practice round and could not finish, or finished with more than eight minutes spare |
| Threshold below which it is over-engineering | Rounds under 30 minutes. At that length there is only one budget: five minutes on the domain, twenty on operations and errors, five on wrap |
| The cheaper alternative | A three line note on the desk: domain, operations, errors, evolution. The order matters far more than the minute counts |
| Failure mode of over-applying it | Narrating phase names out loud. Interviewers screen for recited templates, and announcing "now entering my error phase" is the tell |

---

## Module 3. Domain modelling first: entities, aggregates and invariants

**Time: 30 to 35 minutes reading, 25 to 35 minutes practice.**

### 3a. Predict first

A candidate proposes this data model for a seat booking system, then proposes one write operation per table.

| Table | Fields |
|---|---|
| event | event_id, name, venue_id, starts_at |
| seat | seat_id, event_id, section, row, number, status |
| hold | hold_id, seat_id, user_id, expires_at |
| order | order_id, user_id, total_cents, status |
| order_seat | order_id, seat_id |

??? note "Show answer"

    Prediction asked for: name one rule this system must obey that no single one of those five write operations can enforce.

    "A seat is held or sold to at most one user at a time." Nothing in the model owns that rule. A `hold` row can be inserted twice for the same `seat_id` unless something forbids it, and `seat.status` can be updated by one request while a `hold` row is created by another.

    The rule spans two tables, so enforcing it requires deciding which unit of data must change atomically. That unit is the aggregate, and choosing it is the single decision this module is about. A model that lists tables without naming that unit has deferred the only hard question in the domain.

### The four words, defined before use

| Term | Definition | Test that distinguishes it |
|---|---|---|
| Entity | A thing with an identity that persists as its fields change | If you replace every field and it is still "the same one", it is an entity |
| Value object | A thing defined entirely by its fields, with no identity | Two with equal fields are interchangeable. Money, a date range, an address |
| Aggregate | A cluster of entities and value objects that changes as one unit, with one entity as its root | Ask "what must be consistent at the instant a write returns" |
| Invariant | A statement that must be true before and after every operation | If it can be false for a second without the system being wrong, it is not an invariant |

Bounded context arrives later in this module, because it only makes sense once aggregates exist.

### The rule that decides aggregate boundaries

One rule, stated as a question: **what must be true the moment the write returns, and what may be repaired later?**

| Consistency requirement | Where it belongs | Cost |
|---|---|---|
| Must hold at the instant of the write | Inside one aggregate, one transaction | Every write to that aggregate serialises |
| May be false for seconds and then repaired | Across aggregates, with an event and a compensating action | Complexity, plus a visible intermediate state |
| May be false for hours | A batch reconciliation job | Cheapest, and honest, if the business tolerates it |

Two consequences follow, and both are load bearing:

- Aggregates reference each other **by identity only**, never by embedded object. If Booking holds a whole Property object, you have merged two transaction boundaries by accident.
- **One aggregate instance per transaction.** If an operation must change two aggregates atomically, either the boundary is wrong or the operation needs a saga with compensation.

### Why the boundary is a throughput decision, not a taste decision

Aggregate size sets your write concurrency, and the arithmetic is short enough to do at a whiteboard.

Inputs, all stated here: a 40,000 seat venue, an on sale burst of 2,000 hold requests per second, and an assumed 5 milliseconds of transaction time per hold. That assumption is reasonable because a hold is one indexed lookup plus one insert on a warm primary, and 5 milliseconds is a conservative round number for that pair.

| Aggregate choice | Serialised writes per second per instance | Instances | System capacity |
|---|---|---|---|
| The whole event | 1 / 0.005 = 200 | 1 | 200 per second |
| A section of 200 seats | 200 | 40,000 / 200 = 200 | 40,000 per second |
| A single seat | 200 | 40,000 | very large, but see below |

The event as aggregate choice is 10 times short of the 2,000 per second requirement, so it is eliminated by arithmetic rather than by opinion. The section choice clears it with a factor of 20 in hand.

The per seat choice looks best on this table and is usually wrong, because "pick any two adjacent seats" then spans two aggregates and needs a saga to stay atomic. The section boundary is the smallest one that still contains the adjacency invariant.

```
+---------------------------------------+
| Aggregate Section 12                  |
|                                       |
|  +----------+  +----------+           |
|  | Seat A12 |  | Seat A13 |  200 seats|
|  | held     |  | free     |           |
|  +----------+  +----------+           |
|                                       |
|  invariant  one holder per seat       |
|  invariant  adjacent pairs stay whole |
+---------------------------------------+
        |  by id only
        v
+-----------------+     +------------------+
| Aggregate Order |     | Aggregate Payment|
| seat ids only   | --> | order id only    |
+-----------------+     +------------------+

Aggregate boundaries for seat booking. Lines crossing a boundary
carry identifiers only, so each box is one transaction
```

### Bounded contexts: the same word meaning two things

A bounded context is a boundary inside which one word has exactly one meaning. It is the answer to the observation that "Order" in fulfilment and "Order" in billing are not the same object and should not share a model.

| Word | Meaning in one context | Meaning in another | What breaks if you merge them |
|---|---|---|---|
| Order | Fulfilment: a set of items to pick and ship | Billing: a set of charges and taxes | A refund changes billing but must not un-ship anything |
| User | Identity: credentials and sessions | Support: a person with a case history | Deleting an identity would delete case history |
| Booking | Partner: an inventory commitment | Guest: a trip with an itinerary | Partner cancellation windows leak into guest facing fields |

The interview relevant consequence: **one contract should serve one context.** When a single operation returns fields from two contexts, you get an object that no team fully owns and that cannot change without cross team agreement. If you catch yourself adding a `billing_status` field to a fulfilment resource, you have crossed a context boundary and should say so out loud.

### Deriving the interface from the model

Once aggregates and invariants exist, most of the operation list is mechanical.

| What you have | What it becomes in the contract |
|---|---|
| Aggregate root | A resource with an identity and a lifecycle |
| Entity inside an aggregate | A subordinate path or an embedded field, not an independent resource |
| Value object | A field or an embedded structure, never a resource |
| Invariant within one aggregate | A validation the write enforces, plus a specific error code |
| Invariant across aggregates | An event, a compensating operation, and a visible intermediate state |
| State transition with a business meaning | A named operation, because "set status to cancelled" hides a rule |

That last row is where the resource style starts to strain, which is exactly what module 4 is about.

### 3d. Worked example

Prompt: "Model the domain behind an interface that lets a partner sell tickets to our events."

**Block 1. PURPOSE: separate entities from value objects, because the split decides what gets an identifier.**

I list candidate nouns from the prompt and from two clarifying answers: Event, Seat, Section, Hold, Order, Money, DateRange, Attendee.

Money and DateRange are value objects. Two amounts of 4500 cents in the same currency are interchangeable, so they need no identity. Everything else has a lifecycle I need to reference later, so it gets an identifier.

**The error most readers make here** is giving Money an id because it lives in its own table. Storage layout is not the domain. If the only reason a thing has an identifier is a foreign key, it is a value object wearing a row.

!!! note "Say it before you read on"

    Is Section an entity or a value object here? State the test you used, not just the answer.

**Block 2. PURPOSE: state invariants before boundaries, because the invariants are the input and the boundaries are the output.**

I write four invariants:

1. A seat has at most one active hold or sale at any instant.
2. A hold expires at a stated time and frees the seat.
3. An order's total equals the sum of its seat prices plus fees.
4. An event's sold count never exceeds its capacity.

Invariant 1 must hold at the instant of the write, because two people cannot receive the same seat. Invariant 4 may lag by seconds, because it is a derived counter and a brief undercount harms nobody.

**The error most readers make here** is treating all four as equally urgent, which forces everything into one aggregate and destroys write throughput.

**Block 3. PURPOSE: choose boundaries using the arithmetic, not the aesthetics.**

Invariant 1 forces seat level atomicity. Adjacency requirements force a boundary larger than one seat. Using the numbers from earlier in this module (2,000 holds per second at peak, 5 milliseconds per transaction, 200 serialised writes per instance), a section of 200 seats gives 200 instances and 40,000 writes per second of headroom, which is twenty times the requirement.

Invariant 4 becomes a projection updated from a SeatSold event, not a field inside the Section aggregate. It is allowed to be a few seconds stale, and I say that out loud with the number attached.

!!! note "Say it before you read on"

    Why is invariant 3 not a reason to merge Order and Section into one aggregate?

**Block 4. PURPOSE: turn the model into an operation list, so the interface is derived rather than invented.**

- Section aggregate: `hold seats` and `release hold` as named operations, because both enforce invariant 1 and both are transitions rather than field edits.
- Order aggregate: create, read, cancel.
- Event: read only through this contract, because partners do not create our events.
- Sold count: a read field on the event, documented as eventually consistent with a stated bound of a few seconds.

**The error most readers make here** is exposing a patch operation on a seat with a writable `status` field. It lets a caller write "sold" with no order behind it, which makes invariant 1 unenforceable at the contract level even if the database still holds.

**Result.** Four invariants produced three aggregates, six operations and one deliberately stale projection. Nothing in the operation list was invented from the prompt's nouns, and every field a caller can write traces to an invariant that permits it.

### 3e. Fade the scaffold

**Faded example 1.** Prompt: "Model the domain behind a food delivery interface used by restaurants." Blocks 1 to 3 are given. Produce block 4, the operation list.

- Block 1: entities are Restaurant, MenuItem, Order, OrderLine, Courier. Value objects are Money, Address, TimeWindow.
- Block 2: invariants are that an Order never contains an unavailable MenuItem at acceptance time, that an accepted Order has exactly one Courier assignment at a time, and that a Restaurant's daily order count is reportable within a minute.
- Block 3: aggregates are Restaurant with its menu, and Order with its lines and its current assignment. The daily count is a projection.

??? note "Show answer"

    Restaurant aggregate: read the menu, replace the menu for a date, mark an item unavailable. The last one is a named operation rather than a field edit, because it must also affect in-flight carts, which is a rule and not a value.

    Order aggregate: accept, reject, assign courier, mark ready, cancel with a reason. All are transitions, not field writes.

    Deliberately absent: any operation that writes the daily count, because it is derived. Also absent: an operation that creates an Order, because in this direction the restaurant receives orders rather than creating them. Noticing who the caller is, and therefore which transitions they are allowed to trigger, is the point of the block.

**Faded example 2.** Prompt: "Model the domain behind a payroll interface a partner uses to run payroll for its own customers." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: entities are Employer, Employee, PayRun, Payslip, Payment. Value objects are Money, TaxCode, PayPeriod.
- Block 2: invariants are that a PayRun is immutable once submitted, that the sum of Payslip nets equals the PayRun total, and that an Employee has exactly one active TaxCode per PayPeriod.

??? note "Show answer"

    Block 3: PayRun with its Payslips is one aggregate, because invariant 2 must hold at the instant of submission and cannot be repaired later without reissuing money. Employer with Employees is a second aggregate, since employee edits do not need to be atomic with a pay run. Payment is a third, referencing PayRun by id, because money movement is a separate lifecycle with its own retries.

    Immutability changes the shape of the whole interface: a correction is a new PayRun that references the prior one, not an edit. That single decision removes the need for optimistic locking on submitted runs and removes an entire class of concurrency errors.

    Block 4: operations are create draft run, add or replace payslips on a draft, submit run (the transition that freezes it), read run, and create a correction run referencing a prior one. Employer side: create and update employee, set tax code for a period. There is deliberately no update-payslip operation on a submitted run, and saying that out loud is what shows the invariant was applied rather than recited.

### 3f. Check yourself

**Q1.** A candidate says "I will make the whole event one aggregate so that seat allocation is simple." Give the one sentence arithmetic rebuttal, then the design change.

??? note "Show answer"

    Arithmetic: one aggregate instance serialises its writes, so at an assumed 5 milliseconds per transaction it sustains 1 / 0.005 = 200 writes per second, and an on sale burst of 2,000 per second is ten times that.

    Change: partition the aggregate along a boundary that still contains the adjacency invariant, for example a section of about 200 seats, giving 200 instances and roughly 40,000 writes per second of headroom. Then move the sold counter out to a projection updated by an event, and state the staleness bound in the contract rather than hiding it.

**Q2.** You are told "an order's total must always equal the sum of its lines, and an order's lines reference products whose prices change daily". Where does the invariant live, and what does that force into the contract?

??? note "Show answer"

    The invariant lives inside the Order aggregate, which means the price must be copied onto the order line at the moment the line is created. Referencing the Product's current price would make the invariant depend on another aggregate that changes independently, so the total would silently drift whenever a price changed.

    In the contract this forces two visible things: the order line response carries its own `unit_price` rather than only a product id, and creating a line takes an expected price, so that a price change between the client reading a catalogue and posting the order returns a specific conflict error instead of quietly charging a different amount.

### 3g. When not to use this

Aggregate modelling has a real cost: it forces eventual consistency between boundaries, and every boundary crossing becomes an event, a compensation and an observable intermediate state.

| Question | Answer |
|---|---|
| The measurement that justifies it | You can name at least one invariant that spans two entities and would be violated by concurrent writes, or one write path whose serialised throughput is within about 3 times of your peak write rate |
| Threshold below which it is over-engineering | A single writer administrative tool, or any domain where peak writes to the busiest entity stay under roughly 20 per second. At that rate a single transaction over five tables is correct and simpler |
| The cheaper alternative | Foreign keys, a unique constraint for each invariant that spans rows, and one database transaction per request. Write down the invariants anyway, because the list is what you will need on the day the load arrives |
| Failure mode of over-applying it | Sagas and compensating actions for consistency the business never required, which converts a five line transaction into a distributed state machine with its own failure modes |

---

## Module 4. Resource, procedure or event: three ways to expose one domain

**Time: 25 to 30 minutes reading, 20 to 25 minutes practice.**

### 4a. Predict first

The same domain change, "a customer cancels an order that has not shipped", exposed three ways.

| Style | The operation as the caller sees it |
|---|---|
| Resource | Write `status = "cancelled"` onto the order resource |
| Procedure | Call `cancelOrder` with an order id and a reason |
| Event | Publish `OrderCancellationRequested` with an order id and a reason, and return an acknowledgement |

??? note "Show answer"

    Prediction asked for: which one makes the precondition "not yet shipped" hardest to express to the caller, and why?

    The resource form. It presents cancellation as a value assignment, so the caller reasonably expects any value to be writable and expects the write to either succeed or fail on validation.

    Two things then go wrong. First, there is no natural place to carry the reason, which the domain requires but which is not a property of the order's status. Second, once the order has shipped, the failure has to be reported as a rejected field write, which reads as "bad input" when the truth is "the state machine forbids this transition right now".

    The event form has the opposite problem: the caller learns nothing about whether the cancellation was allowed, because the acknowledgement only means the request was accepted. That is fine when a human is not waiting, and wrong when one is.

### The three styles, defined

| Style | The unit the caller manipulates | Response means | Natural failure signal |
|---|---|---|---|
| Resource | A named thing with a lifecycle, addressed by identity | The thing now looks like this | The requested state is not reachable |
| Procedure | A named action with arguments and a result | The action completed, here is the result | The action was refused, here is why |
| Event | A statement that something happened or is requested | We recorded your statement | Nothing synchronous. Failure surfaces later |

None of these is a protocol choice. All three can be carried over the same transport, and module 5 handles transport separately. This module is only about which unit you hand the caller.

### The selection rule, in one question per row

Ask these in order and stop at the first row that answers yes.

| Question | If yes, use | Why |
|---|---|---|
| Does the caller need the outcome before it can continue? | Resource or procedure | An event returns an acknowledgement, not an outcome |
| Is the change a state transition with a precondition and a business name? | Procedure | A transition needs a name, arguments and a refusal reason |
| Is the change a full replacement of a thing the caller already holds? | Resource | Replacement is idempotent by nature and easy to cache |
| Do two or more independent systems need to react? | Event | Otherwise the producer accumulates one integration per consumer |
| Would a synchronous failure of a consumer block the caller? | Event | Decoupling is the point, not the transport |

The interview relevant sentence: **a resource model is the default because it is cheap to read, cache and evolve, and every named procedure you add is a small admission that the resource model did not fit.** Adding three or four such admissions is normal and healthy. Adding twenty means the resources were chosen from nouns rather than from aggregates.

### Chattiness has a number attached

Inputs stated here: a mobile order detail screen needs four things (the order, the customer, the line items with product names, the shipment status), and the assumed round trip time to a regional edge is 120 milliseconds. That figure is an assumption, and it is reasonable as a round number for a mobile client on a typical cellular connection to a nearby edge.

| Shape | Round trips on the critical path | Time on the critical path |
|---|---|---|
| Four resource reads, each needing the previous one's id | 4 | 4 x 120 = 480 ms |
| Four resource reads, all independent, issued in parallel | 1 | 120 ms, at four times the connection and authorisation work |
| One procedure call returning the composed view | 1 | 120 ms |

The 480 millisecond figure is what kills naive resource models on mobile, and it comes entirely from *dependency*, not from the number of calls. If the client can issue all four at once, resource style costs one round trip and the argument disappears. The design question is therefore not "how many resources" but "how many of them can the client address without first reading another".

### Events change the integration count, not the latency

Inputs: 3 systems produce order changes, 6 systems need to react (billing, warehouse, email, analytics, fraud, support).

| Shape | Integrations to build and maintain |
|---|---|
| Each producer calls each consumer directly | 3 x 6 = 18 |
| Each producer publishes, each consumer subscribes | 3 + 6 = 9 |

At 6 consumers the broker halves the integration count. At 2 consumers it is 3 x 2 = 6 versus 3 + 2 = 5, which is not worth a broker, an ordering story and a dead letter queue. The crossover in this arithmetic sits at roughly 3 consumers per producer, which is the threshold worth saying out loud rather than asserting that events are cleaner.

```
+----------+   +----------+   +----------+
| Producer |   | Producer |   | Producer |
+----------+   +----------+   +----------+
      |              |              |
      +-------+------+------+-------+
              |             |
              v             v
        +-----------------------+
        | Event log             |
        | ordered per order id  |
        +-----------------------+
           |      |       |
           v      v       v
     +--------+ +-----+ +---------+
     | Billing| |Email| |Analytics|
     +--------+ +-----+ +---------+

Three producers and three consumers through a log. Direct calls
would need 9 links here and 18 at six consumers
```

### The hybrid that is usually correct

Most real contracts are a resource spine with a small number of named transitions and an event stream beside them.

- Resources for the things the caller reads, lists, caches and holds.
- Procedures for transitions that carry preconditions, arguments or side effects the caller must be told about.
- Events for everything the caller does not need to wait for and other systems must observe.

Saying this out loud is worth more than picking one style purely, because a candidate who insists on pure resource modelling has to smuggle transitions in as status writes, and one who insists on pure procedures loses caching and discoverability.

### 4d. Worked example

Prompt: "Design how a partner publishes a job posting on our board. Postings are reviewed before they go live, and three internal systems care when a posting goes live."

**Block 1. PURPOSE: separate the things from the transitions, because that split decides the style per operation.**

The thing is a JobPosting. The transitions are submit for review, approve, reject, publish, expire. Only the first is caller triggered from outside.

I say: "Reads and drafts are resources. Everything with a precondition is a named operation. Going live is an event, because three systems care and none of them can block the partner."

**The error most readers make here** is exposing a `state` field on the posting and letting the partner write it. That hands the state machine to the caller and makes every illegal transition indistinguishable from a typo.

!!! note "Say it before you read on"

    Name the precondition on "submit for review" that a plain field write could not express.

**Block 2. PURPOSE: shape the resource part so the common reads are one round trip.**

The partner's dashboard shows postings with their review state and applicant counts. Rather than a posting read plus a review read plus a counts read, I return review state as an embedded object on the posting and applicant counts as a separate collection, because counts change constantly and would ruin caching on the posting.

Using the round trip arithmetic above, embedding the review state removes one dependent call, which at an assumed 120 milliseconds per round trip is 120 milliseconds off the dashboard's critical path.

**The error most readers make here** is embedding everything, including the volatile counts, which makes the posting representation uncacheable and forces a fresh read on every poll.

**Block 3. PURPOSE: give the transition a name, arguments and a refusal reason.**

`submitForReview` takes a posting id and an optional note. It refuses with a specific code when the posting is missing required fields, and with a different code when it is already under review. Two codes, not one, because the caller's next action differs: fix and resubmit, versus wait.

**The error most readers make here** is a single generic validation error, which forces the partner's engineer to parse a human message to decide whether to retry.

!!! note "Say it before you read on"

    Why does "already under review" deserve its own code rather than being folded into a conflict category?

**Block 4. PURPOSE: publish the event once and let consumers pull what they need.**

`PostingPublished` carries the posting id, the employer id, the publish timestamp and a version. It does not carry the posting body. Consumers read the body if they need it, which keeps the event small and stops the event schema from becoming a second copy of the resource schema that must be evolved in step.

With 3 consumers and 1 producer, direct calls would be 3 links versus 4 with a broker, so the broker is not justified by integration count alone here. It is justified by the requirement that a slow consumer must not block publishing, which is a different argument, and I state which argument I am using.

**Result.** One resource, two named transitions exposed externally, one event. Every style choice traced to a property of the operation rather than to a preference, and the one place where the usual arithmetic did not justify the broker was called out rather than glossed.

### 4e. Fade the scaffold

**Faded example 1.** Prompt: "Design how a warehouse system tells our order service that a shipment left the building. Two other systems (customer email and analytics) also care." Blocks 1 to 3 are given. Produce block 4.

- Block 1: the thing is a Shipment. The transition is dispatch. It carries a carrier, a tracking number and a timestamp.
- Block 2: Shipment is readable by order id, and the tracking number is embedded because it never changes after dispatch.
- Block 3: `markDispatched` is a named operation with two refusal codes, already dispatched and shipment cancelled.

??? note "Show answer"

    Block 4 has to decide between a direct call and an event, and the honest answer is to do the arithmetic out loud: 1 producer and 3 consumers is 3 direct links versus 4 with a broker, so integration count does not justify the broker.

    What does justify it is failure isolation plus ordering. Email and analytics must not be able to fail a dispatch, and analytics needs dispatch and delivery events in order per shipment. So publish `ShipmentDispatched` keyed by shipment id, with order service consuming it as the system of record and the other two as best effort.

    The credit earning detail is naming the partition key. Without "keyed by shipment id" the ordering guarantee is undefined, and a delivery event can be processed before its dispatch.

**Faded example 2.** Prompt: "Design how a customer changes the delivery address on an order that may already be in a picking queue." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: the thing is an Order with a shipping Address value object. The change is not a simple field write, because the picking stage may have already printed a label.
- Block 2: the Address is embedded on the Order, and the Order carries a fulfilment stage field that is read only to the customer.

??? note "Show answer"

    Block 3: this is a procedure, `requestAddressChange`, not a field write, because the outcome depends on a stage the caller cannot see and the answer is not always yes. The response has three outcomes rather than two: applied, rejected because the parcel already left, and accepted-pending because the warehouse must confirm. A field write cannot express the third outcome at all, and that is the whole reason for the style choice.

    Block 4: accepted-pending needs a representation the caller can poll, so the operation returns an AddressChangeRequest resource with its own id and terminal states. It also needs an event, `AddressChangeApplied` or `AddressChangeRejected`, because the label printing service and the customer email service both react. The common wrong answer is to make the operation synchronous and simply return an error when the warehouse is uncertain, which pushes an unbounded wait onto a customer facing screen.

### 4f. Check yourself

**Q1.** A candidate proposes 22 named procedures and no resources for a document management interface. Give the strongest single objection, and the specific thing you would change first.

??? note "Show answer"

    The objection is that reads lose everything: no shared cache, no conditional requests, no discoverable identity, so every client re-fetches full documents and the read path costs bandwidth proportional to poll frequency rather than to change frequency.

    The first change is to give documents and folders resource identity with cacheable reads, then keep procedures only for the transitions that carry preconditions, likely publish, archive, restore and share. That usually collapses 22 procedures to about 6, and the 6 that remain are exactly the ones a reviewer should scrutinise.

**Q2.** Your event stream carries `OrderUpdated` with the full order body. A consumer complains that it cannot tell what changed. What is the design error, and what are the two candidate fixes?

??? note "Show answer"

    The design error is publishing a state dump under an event name. `OrderUpdated` is not a domain event, it is a change notification with the vocabulary of a database trigger, so every consumer has to diff against its own last copy to recover the meaning the producer already knew.

    Fix one: publish specific events (`OrderAddressChanged`, `OrderCancelled`) that name the transition, which is the better answer because it lets each consumer subscribe to only what it handles. Fix two: keep one event but include the transition name and the changed field set, which is cheaper to build and keeps a single schema, at the cost of every consumer filtering. Choose one and say why, rather than listing both.

### 4g. When not to use this

Each style carries its own over-engineering threshold.

| Style | Measurement that justifies it | Below this threshold it is over-engineering | Cheaper alternative |
|---|---|---|---|
| Named procedures | You can point to a transition with a precondition the caller cannot see, or a required argument that is not a property of the thing | A CRUD admin surface where every change is a field the user typed | A resource with a validated write |
| Events | Three or more independent consumers per producer, or a consumer whose failure must not block the producer | One or two consumers you own and deploy together | A direct call, retried, with a timeout |
| Full resource modelling | Clients read the same objects repeatedly and bandwidth or latency shows up in your traces | An interface used by one server side job once an hour | One procedure that returns the composed result |

The measurement that decides between chattiness fixes is specifically **dependent round trips per screen**. Count them in a trace. If the count is 1 or 2, composing endpoints buys nothing and costs you a cache.

---

## Module 5. Style selection: representational, graph and binary

**Time: 25 to 30 minutes reading, 20 to 25 minutes practice.**

### 5a. Predict first

The same list screen, fifty rows, requested three ways. Assume each full entity serialises to 4 kilobytes of JSON, and the screen actually displays four fields per row that total about 400 bytes.

| Request | What comes back |
|---|---|
| A. Representational list read, default representation | 50 full entities |
| B. Graph query naming exactly the four displayed fields | 50 objects with four fields each |
| C. Binary call with a response message defined by a schema | 50 messages with the same four fields |

??? note "Show answer"

    Prediction asked for: rank A, B and C by bytes on the wire, then name the thing A gets in exchange for being the largest.

    Bytes, using the stated inputs: A is 50 x 4 KB = 200 KB. B is 50 x 400 B = 20 KB. C is the same four fields, but field names are replaced by integer tags on the wire, so assume roughly a third of the JSON size for field name heavy payloads, giving about 7 KB. That one-third figure is an assumption and it is reasonable because short field names often cost as much as the values in small objects.

    What A buys with those bytes: a stable, addressable, cacheable representation. A shared cache can serve the same list to many callers, and a conditional request can return "not modified" in a few hundred bytes. B and C are usually sent as opaque request bodies, so an intermediate cache cannot tell two identical queries apart from two different ones without extra work.

    The general shape: the representational style trades bytes for cacheability, the graph style trades cacheability for exact shaping, and the binary style trades reach and debuggability for size and speed.

### The comparison, on the axes that actually decide it

| Axis | Representational | Graph | Binary schema based |
|---|---|---|---|
| Shared caching | Native, on identity and validators | Hard, because the query is in the body | Rare, usually application level |
| Over-fetching | Common, unless you add field selection | Solved by construction | Solved by schema, at the cost of a new schema per shape |
| Under-fetching | Common, shows up as dependent round trips | Solved by construction | Solved by defining a composed message |
| Streaming | Server sent events or long poll, bolted on | Subscriptions, defined but uneven in practice | Native bidirectional streams |
| Browser reach | Anything that speaks HTTP | Anything that speaks HTTP | Usually needs a proxy or a web variant |
| Tooling for partners | Highest, because a shell one-liner works | Medium, needs a client or an explorer | Lowest, needs generated stubs |
| Operational cost | Lowest | Highest, because query cost is unbounded by default | Medium, dominated by schema distribution |

### The bandwidth number, worked

Continuing the inputs above (200 KB versus 20 KB versus 7 KB) and adding one more: an assumed 1.5 megabits per second of effective mobile downlink, which is a deliberately pessimistic round number for a cellular connection under load.

| Payload | Bits | Transfer time at 1.5 Mbit/s |
|---|---|---|
| 200 KB | 200 x 8 = 1,600 kbit | 1,600 / 1,500 = 1.07 s |
| 20 KB | 160 kbit | 0.107 s |
| 7 KB | 56 kbit | 0.037 s |

The gap between A and B is about one second on a screen a user is watching, which is the argument that wins style debates. The gap between B and C is 70 milliseconds, which is below the threshold most users notice, so choosing binary purely for bytes on a user facing list is not justified by this arithmetic. Choose it for streaming or for internal service call volume instead.

### The caching number, worked

Inputs: a list endpoint receiving 1,000 requests per second, of which an assumed 70 percent are repeats of an identical query within any 60 second window. That assumption is reasonable for a popular list that most users see unpersonalised.

- Representational, cacheable at an edge with a 60 second freshness window: the edge serves 700 requests per second and the origin sees 300.
- Graph or binary, not cacheable by an intermediary: the origin sees all 1,000.

That is a factor of 3.3 in origin load from the style choice alone, before any code is written. It is also the honest counterweight to the bandwidth table, and a strong answer states both numbers rather than one.

```
+---------+  identical GETs  +-------+  misses  +--------+
| Clients | ---------------> | Edge  | -------> | Origin |
| 1000 s  |                  | cache |  300 s   |        |
+---------+                  +-------+          +--------+
                                 |
                             700 s served
                             from cache

+---------+  opaque query bodies        +--------+
| Clients | --------------------------> | Origin |
| 1000 s  |                             | 1000 s |
+---------+                             +--------+

Same traffic under a cacheable read style and an opaque query
style. The second diagram has no cache box because there is
nothing an intermediary can key on
```

### Each style's characteristic production failure

This is the part that separates a comparison table from an opinion worth hiring.

| Style | The failure it produces | Why it is characteristic | The control |
|---|---|---|---|
| Representational | Dependent round trips on mobile, then a rushed composite endpoint that duplicates three others | The model is optimised for the server's nouns, not the client's screens | Sparse field selection plus one or two deliberate composite reads |
| Graph | One deeply nested query expanding into thousands of datastore reads and saturating a shard | Query cost is chosen by the caller, and nesting multiplies | Depth limits, a static cost budget per query, and batching at the resolver |
| Binary | Version skew, where a caller with an old generated stub silently drops a field it never learned about | The schema is compiled in, so upgrades are a distribution problem | A schema registry, compatibility checks in the build, and never reusing a retired field tag |

The graph failure deserves the extra sentence: because the caller composes the query, a single client change can multiply datastore load without any server deployment, and the first sign is usually a shard at 100 percent with no correlated release.

### 5d. Worked example

Prompt: "We have a mobile app, a partner API used by 400 integrators, and eleven internal services. Pick the interface styles."

**Block 1. PURPOSE: refuse the premise that this is one decision, because three audiences have three cost functions.**

I say out loud that there are three surfaces and I will decide each separately, then check they can share a domain layer. Deciding one style for all three is the most common wrong answer here, and it is wrong in a specific way: whichever style you pick, one of the three audiences pays.

**The error most readers make here** is choosing a favourite style and then justifying it three times.

!!! note "Say it before you read on"

    For which of the three audiences does browser reach matter least? Say why before reading on.

**Block 2. PURPOSE: price the partner surface first, because it is the one I cannot change later.**

400 integrators means every breaking change is 400 conversations. The dominant costs are tooling, documentation and stability, not bytes. Representational style wins on those axes, and the bandwidth penalty from the earlier table is irrelevant when the caller is a server on a fast link.

I add sparse field selection to the list reads, which recovers most of the over-fetch cost without giving the caller unbounded query power.

**Block 3. PURPOSE: price the mobile surface on latency and battery, not on elegance.**

The mobile app has four dependent reads on its main screen, which the earlier arithmetic prices at 4 x 120 = 480 milliseconds. The two candidate fixes are a graph style, or two deliberate composite reads in the representational style.

I choose the composite reads, and I say why: with one first party client, the flexibility of a graph style buys little, while the unbounded query cost buys a real operational risk. If the app had many screens changing weekly and a separate team shipping them, I would flip that decision, and I say that too.

**The error most readers make here** is adopting a graph style because the over-fetch table looks decisive, without pricing the operational control it now requires: depth limits, cost budgets and resolver batching are not optional extras.

!!! note "Say it before you read on"

    What measurement, taken after launch, would make you change this decision?

**Block 4. PURPOSE: price the internal surface on call volume and schema discipline.**

Eleven services with high call volume, all deployed by us, no browser in the path. Binary schema based calls win: smaller payloads, generated stubs, and native streaming for the two paths that need it.

The cost I accept is schema distribution, so I name the control immediately: a registry, compatibility checks in continuous integration, and a rule that retired field tags are reserved and never reused.

**Result.** Three surfaces, three styles, one domain layer behind them. The answer that scores badly here is not a wrong style, it is a single style asserted for all three without pricing what each audience loses.

### 5e. Fade the scaffold

**Faded example 1.** Prompt: "A dashboard product where every customer configures their own widgets, and each widget needs a different slice of the same twelve entities." Blocks 1 to 3 are given. Produce block 4, the recommendation and its controls.

- Block 1: one audience, first party web client, no partner surface.
- Block 2: widget shapes change per customer, so the server cannot enumerate the needed field sets ahead of time.
- Block 3: bandwidth analysis shows a full-entity approach sends about 8 times the displayed bytes on a typical dashboard.

??? note "Show answer"

    This is the case where a graph style genuinely wins, because the set of required shapes is chosen by customers at runtime and cannot be enumerated by the server. Saying that condition out loud is what earns the credit, since it is the one condition under which caller composed queries beat server defined responses.

    The controls must come in the same breath, or the recommendation is incomplete: a maximum query depth, a static cost estimate rejected above a budget, per customer complexity quotas, persisted queries so that production traffic uses only reviewed shapes, and batching at the resolver so that a nested list does not become one datastore read per row.

    Persisted queries are the highest value control, because they convert an unbounded caller composed surface back into a reviewable finite set while keeping the authoring flexibility.

**Faded example 2.** Prompt: "An internal service that returns a 2 kilobyte risk score object, called 40,000 times per second by four services we own." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: internal only, no browser, we deploy all callers.
- Block 2: the response shape is fixed and small, and the call is on a latency critical path.

??? note "Show answer"

    Block 3: this is the clearest binary case in the module. At 40,000 calls per second and 2 KB per response, the wire cost is 40,000 x 2 KB = 80 MB per second, which is 640 megabits per second of internal traffic. Cutting the encoded size to roughly a third takes it to about 213 megabits per second, and that saving is real infrastructure rather than a rounding error. The three-times figure is the same assumption stated earlier, and it should be labelled as one.

    Block 4: the controls are schema compatibility checks in the build, reserved field tags that are never reused, and a rule that a new required field is never added. The characteristic failure to name is version skew: a caller compiled against an older schema silently drops a field its schema never mentioned, and the bug shows up as missing data rather than as an error. The mitigation worth stating is that additive-only evolution plus a registry check makes skew harmless rather than merely unlikely.

### 5f. Check yourself

**Q1.** Your partner API is representational. An integrator complains that building their dashboard takes nine calls. Give two fixes and state which you would ship first and why.

??? note "Show answer"

    Fix one, sparse field selection on the existing reads, which cuts bytes but not the number of round trips. Fix two, one composite read designed for that dashboard shape, which cuts the round trips from nine to one or two.

    Ship the composite read first, because nine calls is a dependency problem rather than a payload problem, and the arithmetic says so: at an assumed 120 milliseconds per round trip, nine dependent calls cost about 1.1 seconds, which no field selection can recover. Then add field selection as the cheaper follow up, and resist adding a general query language for one integrator's screen.

**Q2.** A team proposes moving a public partner interface from representational to binary to save bandwidth. What is the strongest counter argument, and what measurement would settle it?

??? note "Show answer"

    The strongest counter is that the cost being optimised is not the cost being paid. Partner integrations are dominated by tooling, debugging and migration effort, and a binary style requires every one of the integrators to adopt generated stubs, which is a migration you cannot force and did not need.

    The settling measurement is egress cost and client side latency attributable to payload size, split by caller. If partner egress is a small share of total spend and partner side latency is dominated by their own processing rather than transfer, the change buys nothing that matters. Keep the representational surface, add field selection, and reserve the binary style for the internal calls where volume actually lives.

### 5g. When not to use this

| Style | Measurement that justifies it | Below this threshold it is over-engineering | Cheaper alternative |
|---|---|---|---|
| Graph style | Client shapes are chosen at runtime by users, or more than about five client teams ship independently against one backend | One or two first party clients with stable screens | Two composite reads plus sparse field selection |
| Binary style | Internal call volume where encoded size times rate is a visible share of network cost, or you need bidirectional streaming | Anything a partner integrates with by hand, or fewer than a few thousand calls per second | Representational reads with compression enabled |
| Sparse field selection | Measured over-fetch above roughly 3 times displayed bytes on a bandwidth constrained client | A server to server caller on a fast link | Leave the default representation alone |
| A composite read | Two or more dependent round trips on a user visible path | A single call that already returns what the screen needs | Nothing, do not add it |

The general failure mode is adopting a style for its best case and inheriting its characteristic failure without the matching control. If you cannot name the control in the same sentence as the choice, you are not ready to make the choice.

---

## Module 6. Contract first: the schema as the design artefact

**Time: 20 to 25 minutes reading, 15 to 20 minutes practice.**

### 6a. Predict first

Here is a proposed change to a shipped response schema, written as a diff.

```
  Booking
    id                string
-   guest_name        string
+   guest             object
+     first_name      string
+     last_name       string
    nights            integer
-   status            string
+   status            enum  CONFIRMED HELD CANCELLED PENDING_REVIEW
+   total_amount      Money  required
```

??? note "Show answer"

    Prediction asked for: four changes are shown. Which of them break an existing client, and which break an existing *caller* as opposed to an existing *reader*?

    | Change | Breaks readers | Breaks writers |
    |---|---|---|
    | Replacing `guest_name` with a `guest` object | Yes, the field they parse disappears | Yes, if they send it on create |
    | Adding `PENDING_REVIEW` to the status enum | Yes, for any client that switches exhaustively on the value | No |
    | Constraining `status` from free string to enum | No, if the values are unchanged | Yes, for any writer sending a value outside the set |
    | Adding required `total_amount` | No | Yes, every existing create call now fails validation |

    The lesson under the table: adding a value to an enum is a breaking change for readers even though nothing was removed, which is the single most commonly missed row in this exercise. A new enum value arriving at a client that mapped the old values exhaustively either throws or falls into a default branch that quietly does the wrong thing.

### The two rules that generate every compatibility answer

- **Readers must ignore fields they do not recognise.** A client that fails on an unknown field converts every additive server change into an outage.
- **Writers must never be required to send something new.** A server that adds a required field breaks every caller that has not shipped yet, which is all of them.

Together these give the asymmetry that most compatibility tables merely list: **additions are safe going out and dangerous coming in.** Say it once at the whiteboard and you can derive the rest live.

### The change taxonomy, derived from those two rules

| Change | Safe for readers of a response | Safe for writers of a request |
|---|---|---|
| Add an optional field | Yes | Yes |
| Add a required field | Yes | No |
| Remove a field | No | Yes |
| Rename a field | No | No, it is a remove plus an add |
| Widen a type, integer to number | Usually no, parsers are typed | Yes |
| Narrow a type, string to enum | Yes if values unchanged | No |
| Add an enum value | No | Yes |
| Remove an enum value | Yes | No |
| Make an optional field required in the response | Yes | Not applicable |
| Change the meaning of a field while keeping its name and type | No, and it is the worst one | No |

That last row is the change no automated checker catches. Repurposing `status` from a shipment state to a payment state passes every schema linter and every generated stub, and breaks every consumer at runtime. The control is a review rule, not a tool.

### Why the schema is the artefact and not the documentation

Three consequences follow from writing the schema before the implementation, and each is worth stating in the round.

| Practice | What it buys | What it costs |
|---|---|---|
| One machine readable interface definition as the source of truth | Generated clients, generated server stubs, generated docs, one place to review a change | The definition becomes a bottleneck, and a sloppy one is worse than none |
| Automated linting of the definition | Naming, pagination and error shape consistency across dozens of operations without a human remembering | Rules must be agreed once, which is a real week of argument |
| Compatibility checking in the build | A breaking change fails the pipeline rather than a partner's integration | You must define which direction of compatibility you require per surface |

The direction matters and candidates rarely name it:

- A **public response surface** needs backward compatibility: new servers must keep old readers working.
- A **request surface with many old writers** needs forward compatibility: new servers must keep accepting old requests.
- An **event stream** usually needs both, because consumers and producers upgrade independently and a replayed old event must still parse.

### The number that makes this concrete

Inputs: 5 million mobile installs, and an assumed client upgrade half life of 90 days, meaning half the remaining installs on any version upgrade every 90 days. That assumption is reasonable for an app users open weekly with automatic updates enabled, and it should be labelled as an assumption when you use it.

| Days after release | Fraction still on the old version | Installs |
|---|---|---|
| 90 | 0.5 | 2,500,000 |
| 180 | 0.25 | 1,250,000 |
| 270 | 0.125 | 625,000 |
| 360 | 0.0625 | 312,500 |

A breaking response change shipped six months after the client release strands 1.25 million installs. That number, not a principle, is what justifies the cost of a schema registry and a compatibility gate. It is also the number that decides sunset dates in module 11.

```
+------------------+     +------------------+
| Interface        | --> | Generated server |
| definition file  |     | stubs and types  |
+------------------+     +------------------+
        |    |
        |    +--------> +------------------+
        |               | Generated client |
        |               | libraries        |
        |               +------------------+
        v
+------------------+     +------------------+
| Lint rules       |     | Compatibility    |
| naming shape     |     | check against    |
| pagination       |     | last released    |
+------------------+     +------------------+
        |                        |
        +-----------+------------+
                    v
             +-------------+
             | Build fails |
             | on breach   |
             +-------------+

The definition is the input to generation and to two gates, so a
breaking change stops in the pipeline rather than at a partner
```

### 6d. Worked example

Prompt: "Review this proposed change to our shipped booking schema and decide what ships." The diff is the one at the top of this module.

**Block 1. PURPOSE: classify each change by direction, because a single verdict for the whole diff is always wrong.**

I take the four changes and place each in the taxonomy: the guest split breaks both directions, the enum addition breaks readers, the enum narrowing breaks writers, the required field breaks writers.

I say the count out loud: four changes, three distinct breakage directions, so this cannot ship as one release regardless of how it is versioned.

**The error most readers make here** is verdicting the diff as "breaking" and stopping. The useful output is a per change plan, because two of these four can ship today.

!!! note "Say it before you read on"

    Which two of the four could ship this afternoon with no client work at all?

**Block 2. PURPOSE: convert each breaking change into a non breaking sequence, because the sequence is the actual deliverable.**

- Guest split: add `guest` alongside `guest_name`, populate both, mark `guest_name` deprecated in the definition, remove it only after the sunset date.
- Required `total_amount`: add it as optional, populate it on every response immediately, and require it on requests only for clients that opt into a newer contract.
- Enum narrowing: keep accepting the old free string on writes, reject unknown values only after measurement shows nobody sends them.

**The error most readers make here** is treating "add both fields" as a hack. It is the standard expand and contract sequence, and naming it as deliberate rather than apologetic is part of the answer.

**Block 3. PURPOSE: handle the enum addition, which has no non breaking sequence, by fixing the contract rather than the change.**

`PENDING_REVIEW` cannot be added safely to clients that switch exhaustively. There is no expand and contract path, because the value itself is the change.

So the fix is a documented rule going forward: clients must treat unknown enum values as a stated fallback, and the definition ships a permanent `UNKNOWN` member so that generated code has somewhere to land. For clients already in the wild, I gate the new value behind a request flag until the install base has turned over, using the half life table to pick the date.

!!! note "Say it before you read on"

    Why does adding a permanent `UNKNOWN` member help future changes but not this one?

**Block 4. PURPOSE: leave behind a rule so the next diff does not need this meeting.**

Three lint rules go into the pipeline: no field removals without a deprecation marker older than the sunset window, no new required request fields, and no enum additions on response types that lack a documented fallback.

**Result.** Two changes ship immediately, one ships as an expand and contract sequence with a dated removal, and one is gated behind an install base threshold with a number attached. The output of a good schema review is a schedule, not a verdict.

### 6e. Fade the scaffold

**Faded example 1.** A proposed diff renames `created` to `created_at` across nine response types for consistency, and the lint rule that flagged it is the naming rule the team just adopted. Blocks 1 to 3 are given. Produce block 4.

- Block 1: a rename is a remove plus an add, so it breaks every reader of all nine types.
- Block 2: the expand and contract sequence is to emit both fields, deprecate the old one, and remove it after the sunset window.
- Block 3: with an assumed 90 day upgrade half life and a stated policy of stranding fewer than 5 percent of installs, the removal cannot happen for about 390 days, since 0.5 raised to the power 390 over 90 is roughly 0.05.

??? note "Show answer"

    Block 4 is the decision, and the defensible one is to not ship the rename on existing types at all.

    The reasoning is a cost comparison the earlier blocks make available: the benefit is naming consistency, which is worth roughly nothing to callers who already integrated, and the cost is thirteen months of duplicated fields across nine types plus a removal that will be forgotten. Apply the lint rule to new types only and mark the nine existing ones as grandfathered with a written note.

    The credit earning move is to say that a lint rule is allowed to have exceptions recorded in the definition, and that a rule which forces a thirteen month migration for cosmetics is a rule being applied without judgement.

**Faded example 2.** An event schema needs a new field that consumers must use to route messages correctly. Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: events need both directions of compatibility, because producers and consumers upgrade independently and old events are replayed from the log.
- Block 2: adding an optional field is safe in both directions, but "consumers must use it" is a semantic requirement that no schema rule can enforce.

??? note "Show answer"

    Block 3: the sequence is to add the field as optional, populate it on every new event, and backfill it into the replayable history if the log is compacted and rewritable. If the history cannot be rewritten, consumers need a defined behaviour for events that predate the field, which is a documented default rather than a crash. That default is part of the contract and belongs in the definition, not in a wiki page.

    Block 4: the rollout order is forced by the dependency. Consumers deploy first with support for both the new field and the pre field default, then the producer starts populating it, and only then can any behaviour depend on it. Reversing that order means new events reach old consumers, which is precisely the failure the compatibility rules exist to prevent. State the order explicitly, because "add the field" without an order is the answer that fails in production.

### 6f. Check yourself

**Q1.** A colleague argues that a schema registry with build time compatibility checks is bureaucracy for a five person team. Give the measurement that decides it.

??? note "Show answer"

    The measurement is the number of consumers you cannot deploy in the same release as the producer. If that number is zero, the registry buys nothing, because a broken change fails your own tests before it reaches anyone. Change the schema and move on.

    The moment it is not zero (a mobile app in an app store, a partner, a replayable event log), the calculation flips, and the half life arithmetic gives the size: with 5 million installs and an assumed 90 day half life, a breaking change six months after a release reaches 1.25 million installs. Bureaucracy is the wrong frame; the question is whether any consumer upgrades on a schedule you do not control.

**Q2.** Which single schema change is invisible to every automated compatibility checker, and what is the control?

??? note "Show answer"

    Changing the meaning of a field while keeping its name and type. `status` moving from a shipment state to a payment state, or `amount` changing from including tax to excluding it, passes every structural check and every generated stub, and breaks every consumer at runtime with no error at the boundary.

    The control is human and procedural: a review rule that any semantic change is treated as a rename, so it must ship as a new field with the old one deprecated. The secondary control is a test written from the consumer's perspective, which is what the consumer driven contract testing in module 11 is for.

### 6g. When not to use this

| Practice | Measurement that justifies it | Below this threshold it is over-engineering | Cheaper alternative |
|---|---|---|---|
| A machine readable definition as the source of truth | More than one team writes clients, or any client you cannot deploy with the server | One team, one client, one deploy | Typed handlers plus a generated document from the code |
| Automated compatibility gates | At least one consumer upgrading on a schedule you do not control | All consumers deploy in your pipeline | Integration tests across the two services |
| Lint rules on the definition | More than roughly twenty operations, or more than three people adding operations | A five operation internal service | One page of conventions and a reviewer who reads it |
| A schema registry | Independently deployed producers and consumers, or a replayable log | Request and response only, single consumer | The definition file in the repository, versioned with the code |

The honest threshold: contract first is a tax paid to buy independent deployability. If nothing deploys independently, you are paying the tax and receiving nothing.

---

## Module 7. Error model design: the part candidates visibly fail

**Time: 25 to 30 minutes reading, 20 to 30 minutes practice.**

### 7a. Predict first

The same failure, "the payment method was declined by the issuing bank", returned three ways.

| Version | Response body |
|---|---|
| A | `400` with body `{"error": "Bad Request"}` |
| B | `400` with body `{"message": "Your card was declined. Please try another card."}` |
| C | `402` with body `{"code": "card_declined", "decline_reason": "insufficient_funds", "retryable": false, "message": "The card was declined.", "request_id": "req_8fk2"}` |

??? note "Show answer"

    Prediction asked for: write the branch a client integrator has to code against each of the three, and count the cases where the integrator has to guess.

    | Version | What the integrator can do programmatically | Guesses required |
    |---|---|---|
    | A | Nothing. Show a generic failure | Everything: is it my fault, will a retry help, is the card the problem |
    | B | Show the message and hope it is user appropriate | Whether to retry, and whether the message is safe to show a customer in another language |
    | C | Branch on `code`, decide retry from `retryable`, log `request_id`, tell the customer something specific | None of the four |

    The general shape: an error is not a report of what went wrong, it is an instruction for what the caller does next. Version B is worse than it looks, because it invites integrators to parse a human message, which then cannot be changed without breaking them.

### The three questions every error must answer

| Question | The field that answers it | What goes wrong if it is missing |
|---|---|---|
| Whose fault is it, mine or yours? | A stable machine code, plus a coarse class | The caller retries a request that will never succeed |
| Will retrying help, and when? | A retryability flag and, when known, a retry-after hint | Either no retry on a transient failure, or a retry storm on a permanent one |
| What exactly is wrong, and where? | A per field or per item detail list | The caller shows "invalid request" to a user who typed one bad character |

Two supporting fields carry their weight in every real interface: a request identifier the caller can quote in a support ticket, and a documentation link keyed by the code.

### The taxonomy, on axes rather than as a list

Most error catalogues are a flat list of codes. Build it on three axes instead, because the axes are what a caller branches on.

| Class | Who can fix it | Retry safe | Typical transport status |
|---|---|---|---|
| Malformed input | Caller, by changing the request | No | 400 |
| Authentication missing or invalid | Caller, by getting a token | No | 401 |
| Authorised but not permitted | Caller's administrator | No | 403 |
| Target does not exist | Caller, by using another identifier | No | 404 |
| Precondition or state conflict | Caller, by reading current state first | Sometimes, after re-reading | 409 |
| Semantic validation failure | Caller, by changing values | No | 422 |
| Quota or rate limit | Caller, by waiting | Yes, after the stated delay | 429 |
| Internal fault | Us | Yes, with backoff | 500 |
| Dependency unavailable | Us | Yes, with backoff | 503 |
| Deadline exceeded upstream | Either | Yes, but only if the operation is idempotent | 504 |

The row candidates most often get wrong is 409 versus 422. A conflict means the request is valid but the current state forbids it, so re-reading and retrying can work. A validation failure means the request itself is wrong, so retrying it unchanged never works. Collapsing both into 400 destroys that distinction and is the single most common error model defect.

### Stable codes and unstable messages

| Element | Contract status | Rule |
|---|---|---|
| Machine code, for example `card_declined` | Part of the contract, versioned, never repurposed | Adding one is a breaking change for exhaustive clients, so document a fallback |
| Human message | Explicitly not part of the contract | State in writing that it may change at any time and must not be parsed |
| Field path in details | Part of the contract | Use the same path syntax the request used |
| Transport status | Part of the contract | Never return 200 with an error body |

That last row deserves a sentence. Returning 200 with `{"success": false}` breaks every intermediary that reasons about status: retries, circuit breakers, monitoring, load balancer health logic. It is a self inflicted wound that shows up as invisible failures on dashboards that look green.

### Retry amplification, with the arithmetic

Inputs, stated here: a service receiving 10,000 requests per second, clients configured to retry up to 3 times, and no retry budget.

| Scenario | Attempts per original request | Load on the service |
|---|---|---|
| Healthy, 0 percent failures | 1 | 10,000 per second |
| Degraded, 30 percent of attempts fail | 1 + 0.3 + 0.3^2 + 0.3^3 = 1.42 | 14,200 per second |
| Fully down, 100 percent of attempts fail | 4 | 40,000 per second |
| Fully down, two layers each retrying 3 times | 4 x 4 = 16 | 160,000 per second |

The important row is the last one. The service that just failed now receives sixteen times its normal traffic, which is why a partially degraded dependency so often becomes a total outage. Nothing about this is exotic: it is 4 raised to the number of retrying layers.

Two controls follow directly, and both belong in the contract rather than only in the client library:

- **A retryability flag**, so callers never retry a permanent failure. That removes the 4x on the entire class of caller errors.
- **A retry budget**, for example retries capped at 10 percent of the successful request rate, which bounds amplification at 1.1x regardless of failure rate.

Jitter matters too, but it changes the *shape* of the load rather than the total, so state it as a separate control rather than as the fix.

```
+---------+   1 try  +----------+   1 try   +----------+
| Client  | -------> | Gateway  | --------> | Service  |
+---------+          +----------+           +----------+

fully down with 3 retries at each layer

+---------+ 4 tries  +----------+ 16 tries  +----------+
| Client  | -------> | Gateway  | --------> | Service  |
+---------+          +----------+           +----------+
                                            | 160k per s
                                            +----------+

Retry amplification multiplies per layer. Two retrying layers
turn 10k per second into 160k against a service already failing
```

### Partial failure in batch operations

A batch operation has three possible contracts, and picking one silently is how interfaces get support tickets.

| Contract | Response | When it is right |
|---|---|---|
| All or nothing | One status, one error | The items share an invariant, for example a set of ledger entries that must balance |
| Independent items | Per item status array with a stable client supplied item key | The items are unrelated, for example a catalogue upsert |
| Fail fast | Processed until the first error, then stop, reporting the index | Ordering matters and later items depend on earlier ones |

For the independent case, three details decide whether integrators can actually use it:

- The overall transport status is not 200 with a success body. Use a status that says "this was processed, look inside", and never a status that says "everything worked".
- Every item result carries the client's own key for that item, not just an index, because the caller may have reordered or retried a subset.
- Item errors use the same code vocabulary as single item operations, so the integrator writes one error handler and not two.

### 7d. Worked example

Prompt: "Design the error model for an authorisation call on a payments interface."

**Block 1. PURPOSE: enumerate the failure sources before the codes, because the sources decide the classes.**

Four sources: the caller's request is wrong, the caller's credentials or permissions are wrong, the card or its issuer refuses, and our own path fails including the downstream gateway.

I map them to classes immediately: caller fixable and permanent, caller fixable and permanent, business outcome, and ours plus transient. The third one is the interesting one, and I say why out loud: a decline is not an error in our system, it is a successful call with a negative business result.

**The error most readers make here** is putting declines in the same bucket as validation failures, which makes an integrator treat "insufficient funds" as a bug in their code.

!!! note "Say it before you read on"

    Argue the other side for thirty seconds: what is the case for returning a decline as a 200 with a status field?

**Block 2. PURPOSE: decide the decline representation, since it is the highest volume non success outcome.**

I choose a distinct status for declines rather than folding them into a generic client error, and I carry two fields: a stable top level code, and a narrower `decline_reason` from a documented set.

The reason for two fields rather than one is evolution. Issuers invent new decline reasons regularly, and new reasons must be addable without breaking a client that branches on the top level code. So the top level set is small and frozen, the narrow set is documented as extensible with an explicit fallback value.

**The error most readers make here** is exposing the raw issuer response code, which welds a third party's vocabulary into your contract forever.

**Block 3. PURPOSE: state retryability per class rather than per code, so the caller's logic stays small.**

- Caller errors: `retryable: false`, no retry-after.
- Declines: `retryable: false` at the interface level, even where a human might retry later with another card. The distinction is that our contract cannot promise a different outcome.
- Rate limits: `retryable: true` with an explicit delay.
- Our faults and gateway timeouts: `retryable: true`, and only safe because every authorisation carries an idempotency key, which module 9 covers.

I say the dependency explicitly: retryable on a money moving operation is a lie unless idempotency is designed, and the two features have to ship together.

!!! note "Say it before you read on"

    Why is `retryable: true` on a 504 dangerous without the idempotency work?

**Block 4. PURPOSE: bound the blast radius, because a payments interface degrading is the scenario that matters.**

With clients retrying three times and no budget, the arithmetic above gives 4x amplification per layer against a gateway that is already failing. So the contract states a retry budget, returns retry-after on 429 and 503, and documents jittered backoff as required client behaviour rather than a suggestion.

**Result.** Four classes, roughly nine top level codes, one extensible narrow set, one retryability rule per class, and a stated dependency on idempotency. The thing that earns a senior rating is not the code list, it is naming which codes a client may retry and proving that retrying them is safe.

### 7e. Fade the scaffold

**Faded example 1.** Prompt: "Design the error model for a bulk contact import that accepts up to 10,000 contacts per request." Blocks 1 to 3 are given. Produce block 4.

- Block 1: failure sources are malformed request envelope, per contact validation failures, duplicate detection against existing records, and our own faults.
- Block 2: the envelope failures are all-or-nothing, and the per contact failures are independent.
- Block 3: the response is a per item array keyed by a client supplied `client_ref` on each contact.

??? note "Show answer"

    Block 4 is the retry story, and it is where bulk operations usually break. With 10,000 items and, say, 12 failures, a client that retries the whole request creates 9,988 duplicate operations, so the contract must make the retry cheap and safe.

    Two mechanisms, stated together: an idempotency key on the whole request, so a network level retry returns the original per item result set rather than reprocessing, and per item results specific enough that the client retries only the failed subset. The second one only works if item errors are classified as retryable or not, item by item.

    The common wrong answer is a top level `partial_success: true` flag with no per item detail, which forces the client to diff its input against the resulting records to work out what happened.

**Faded example 2.** Prompt: "Our public interface returns 400 for every client side problem and 500 for everything else. Two integrators have filed tickets about retry behaviour." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: the two ticket patterns are integrators retrying permanent failures forever, and integrators not retrying transient ones at all.
- Block 2: the root cause is that a single status carries three different meanings, so no branch can be written.

??? note "Show answer"

    Block 3: split the 400 by what the caller does next, not by what went wrong internally. Malformed body stays 400, authentication and permission move to 401 and 403, state conflicts move to 409, semantic validation moves to 422, and quota moves to 429 with a delay. Split the 500 into our fault (500), dependency unavailable (503) and deadline exceeded (504), because the third one is the only one where the caller must consider that the work may in fact have happened.

    Block 4: the migration matters as much as the taxonomy, because narrowing statuses breaks integrators who wrote `if status == 400`. Ship the new machine codes inside the existing bodies first, publish the mapping, give a dated window, then change the statuses. The strongest version adds a per integrator dashboard of which codes they receive, so the ones affected can be contacted directly instead of by broadcast email.

### 7f. Check yourself

**Q1.** An interviewer asks why you would ever return 409 rather than 400. Answer in two sentences, then name what the caller does differently.

??? note "Show answer"

    400 says the request itself is wrong, so retrying it unchanged can never succeed. 409 says the request is well formed but the current state forbids it, so re-reading the resource and retrying with fresh state can succeed.

    The caller behaves differently in a way you can point at: on 400 it surfaces an error to whoever composed the request, on 409 it refetches, reapplies its change and retries once or twice, and only then gives up. If both cases return 400, the caller cannot implement the second behaviour at all.

**Q2.** Your service returns `retryable: true` on a 503. A client retries three times with no jitter, and so does the gateway in front of you. Quantify the load on a fully failed service and name the two controls.

??? note "Show answer"

    Each retrying layer multiplies attempts by 4 when every attempt fails, so two layers give 4 x 4 = 16 times the normal load. A service normally receiving 10,000 requests per second sees 160,000 while it is at its weakest.

    Controls: a retry budget capping retries at a fixed fraction of successful traffic, for example 10 percent, which bounds amplification at 1.1x regardless of failure rate, and retries at one layer only, usually the outermost, so the multiplier cannot compound. Jitter is worth adding but it spreads the load rather than reducing it, so naming it as the fix is the wrong answer.

### 7g. When not to use this

| Mechanism | Measurement that justifies it | Below this threshold it is over-engineering | Cheaper alternative |
|---|---|---|---|
| A full code taxonomy with documented fallbacks | More than one consumer team, or any consumer you cannot deploy | One internal caller you own | Five codes and a clear message field |
| Per item batch results | Batch sizes above roughly 50 where partial failure is expected | Batches of 5 that are naturally atomic | All or nothing with a single error |
| Retry-after and retry budgets | You have observed load amplification during an incident, or you have two or more retrying layers | A single client with a strict timeout and no retries | A timeout and one retry, documented |
| An extensible narrow reason set | The reasons come from a third party whose vocabulary changes | Reasons you fully control and rarely add to | A frozen enum with a fallback member |

The threshold worth remembering: an error model earns its complexity from the number of independent teams writing handlers against it. One team, five codes. Four hundred integrators, a taxonomy with a documented fallback and a stability policy.

---

## Module 8. Pagination: stability, cost and the failure nobody tests

**Time: 20 to 25 minutes reading, 15 to 20 minutes practice.**

### 8a. Predict first

A feed sorted newest first, paginated with offset and limit. Page size is 20. A reader takes 3 seconds to read a page before requesting the next. New items arrive at 10 per second.

| Time | Event |
|---|---|
| t = 0 s | Client requests offset 0 limit 20, receives items R1 to R20 |
| t = 0 to 3 s | 30 new items arrive at the head of the feed |
| t = 3 s | Client requests offset 20 limit 20 |

??? note "Show answer"

    Prediction asked for: exactly which items come back at t = 3 seconds, and how many the reader has already seen.

    At t = 3 seconds the list is 30 new items followed by R1, R2 and so on. Positions 21 to 40 therefore hold the last 10 new items (positions 21 to 30) followed by R1 to R10 (positions 31 to 40).

    So the second page contains 10 items the reader already read. The reader sees duplicates, and they are not random: they are exactly the top of page one. Deletion produces the mirror failure. If 10 items above the window are deleted instead, the window slides the other way and 10 items are skipped and never shown.

    The general statement: offset pagination is stable only against a collection that does not change while the reader is paging. That is a property almost no interesting collection has.

### The three schemes, on the axes that decide it

| Property | Offset and limit | Keyset, also called seek | Opaque cursor |
|---|---|---|---|
| Cost of page N | Scans offset + limit rows | Index seek plus limit rows | Index seek plus limit rows |
| Stable under insert | No, duplicates | Yes | Yes |
| Stable under delete | No, skips | Yes | Yes |
| Jump to page 500 | Yes | No | No |
| Total count available | Cheaply looking, expensively in truth | Not by construction | Not by construction |
| Changing sort mid-scan | Allowed and meaningless | Not allowed | Detectable and rejectable |

An opaque cursor is keyset pagination with the state encoded and signed, so the caller cannot construct one by hand. That distinction matters more than it sounds, because a cursor the caller can author is a cursor you can never change.

### The deep pagination number

Inputs: a table with 10 million rows matching the filter, an index that covers the sort, and an assumed 100 nanoseconds to walk one index entry that is already in memory. That assumption is reasonable as a round number for a cached B-tree traversal step, and it is the one number to label if you use it at a whiteboard.

| Request | Index entries touched | Time |
|---|---|---|
| Offset 0, limit 20 | 20 | 2 microseconds |
| Offset 100,000, limit 20 | 100,020 | 10 milliseconds |
| Offset 5,000,000, limit 20 | 5,000,020 | 500 milliseconds |
| Keyset, any page, limit 20 | 20 plus one seek | tens of microseconds |

The shape is the point: offset cost grows linearly with how deep you are, and keyset cost is flat. A crawler walking the whole collection with offsets does total work proportional to the square of the collection size, which is why the endpoint that was fine for a year falls over the week a partner writes an export script.

### The cursor's contents, and why each part is there

| Part | Why it is in the cursor | What breaks without it |
|---|---|---|
| The sort key value of the last row | It is the seek position | Nothing works |
| A unique tiebreaker, usually the primary key | Rows with equal sort keys must have a total order | A batch import with 500 identical timestamps causes an infinite loop or a skipped block |
| The sort direction and field | The server must reject a mid-scan sort change | The caller changes sort and receives an arbitrary mix |
| A hash of the filter arguments | The same reason as sort | The caller changes the filter and pages through a set that never existed |
| An issue timestamp | Cursors must expire, because the data behind them ages out | A cursor from six months ago seeks into a partition that was archived |

The tiebreaker row is the one candidates miss, and it is worth deriving out loud. With a sort on `created_at` alone and 500 rows sharing one timestamp from a bulk import, "give me rows where created_at is less than X" either returns the same block forever or jumps past all 500, depending on the comparison. The fix is comparing the pair, sort key first and identifier second.

```
+-------------------------------------------+
| Page request                              |
|  cursor  sort key 2026-08-01T10 id 8842   |
|  limit   20                               |
+-------------------------------------------+
                  |
                  v
+-------------------------------------------+
| Seek where sort key id pair is less than  |
| the cursor pair  then take 21 rows        |
+-------------------------------------------+
                  |
                  v
+-------------------------------------------+
| Return 20 rows plus next cursor built     |
| from row 20  the 21st row only answers    |
| has more                                  |
+-------------------------------------------+

Keyset paging. Fetching limit plus one row is how has-more is
answered without a count query
```

### Total counts, priced honestly

| Option | Cost | When to offer it |
|---|---|---|
| Exact count of the filtered set | A full index scan of matching rows on every page request | Small or heavily filtered sets only |
| Capped count, for example "1000 or more" | Scan stops at the cap | Almost always the right public default |
| Estimated count from statistics | Near zero | Dashboards and result summaries where being 5 percent wrong is fine |
| No count, just has-more | One extra row fetched per page | Infinite scroll, and any high volume list |

Using the earlier inputs, an exact count over 10 million matching index entries at 100 nanoseconds each costs about 1 second of work per request, which is why exact totals quietly become the most expensive part of a list endpoint. Offering a capped count instead is a one line contract decision that removes that cost entirely.

### 8d. Worked example

Prompt: "Design pagination for comment threads. Comments nest up to three levels, threads on popular posts reach 50,000 comments, and people comment while others are reading."

**Block 1. PURPOSE: pick the sort before the mechanism, because the sort decides what a cursor can even contain.**

Two sorts are wanted: newest first, and a ranked "top" order. Newest first is a stable monotonic key plus an identifier tiebreaker, so keyset works directly.

Ranked order is not stable at all, because a score changes as votes arrive. I name that immediately rather than discovering it in block 3.

**The error most readers make here** is designing one pagination mechanism and assuming both sorts can use it.

!!! note "Say it before you read on"

    What property must a sort key have for keyset pagination to work, in one sentence?

**Block 2. PURPOSE: make the unstable sort stable by freezing something, because you cannot page a moving order.**

For ranked order I snapshot the ranking at first page request and put the snapshot identifier in the cursor. Later pages read against that snapshot, so a comment cannot move from page 4 to page 2 and vanish.

The snapshot expires, and I state the number: an assumed 10 minute lifetime, which is reasonable because it comfortably exceeds a normal reading session while keeping storage bounded. An expired cursor returns a specific error telling the client to restart from page one.

**The error most readers make here** is silently restarting the scan when the snapshot expires, which shows the reader page one again with no explanation.

**Block 3. PURPOSE: handle nesting without turning one screen into hundreds of calls.**

Top level comments paginate with keyset. Each returns its first 3 replies inline plus a reply count and a reply cursor. Deeper replies paginate on demand.

The arithmetic that justifies inlining 3: a page of 20 top level comments where each needs a separate reply call is 21 requests, and at the assumed 120 milliseconds per round trip from module 4 that is 2.5 seconds if serialised. Inlining the first three replies covers the large majority of threads in one request, and I would measure the reply count distribution to confirm the 3 rather than assert it.

!!! note "Say it before you read on"

    Why is a reply *count* on each comment cheap here when a total count on the thread was expensive earlier?

**Block 4. PURPOSE: state the failure modes in the contract, so integrators are not surprised.**

Three statements go in the documentation: cursors expire after 10 minutes and return a named error, new comments arriving during a newest-first scan appear only on a fresh first page, and total thread counts are capped rather than exact above 1,000.

**Result.** Two sorts, two mechanisms, one snapshot with a stated lifetime, and three documented failure behaviours. The senior signal here is not choosing cursors, it is noticing that the ranked sort cannot be paginated by the same means and saying so before being asked.

### 8e. Fade the scaffold

**Faded example 1.** Prompt: "Paginate an audit log that is append only, never deleted, and queried by administrators filtering on actor and date range." Blocks 1 to 3 are given. Produce block 4.

- Block 1: the natural sort is the append sequence, which is monotonic and unique, so it is its own tiebreaker.
- Block 2: keyset on the sequence number, with the filter hash in the cursor.
- Block 3: no snapshot needed, because appends only ever arrive at one end and deletions never happen.

??? note "Show answer"

    Block 4 is the contract statements, and this collection allows unusually strong ones. Because the log is append only, a cursor never expires for correctness reasons, so cursors can be long lived and even bookmarkable, which is a real feature for an audit tool.

    Two things still need stating. Backward paging is meaningful here and should be offered, since an administrator scanning backwards through time is the main use, and that requires the comparison and the result order to be reversed and then re-reversed before returning. And a page of results should carry the sequence range it covers, so an administrator can prove to an auditor that no gap was skipped.

    The trap avoided: offering exact total counts. A filtered count over an audit log grows without bound, and an administrator who sees "12,481,003 results" is no better served than one who sees "more than 1,000".

**Faded example 2.** Prompt: "A partner exports our entire 40 million row catalogue nightly through the public list endpoint." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: today the endpoint uses offset and limit with a page size of 100.
- Block 2: the partner's export therefore issues 400,000 requests, the last of which uses an offset near 40 million.

??? note "Show answer"

    Block 3: price the current design before changing it. Total index entries touched across the whole export is the sum of the offsets, which is roughly 400,000 x 40,000,000 / 2 = 8 x 10^12 entry visits. At the assumed 100 nanoseconds per entry that is about 800,000 seconds of database work for one nightly export, which is nine days of single threaded work and therefore impossible. This is the quadratic cost made concrete, and the arithmetic is the answer.

    Block 4: two changes, and the second matters more. Move to keyset pagination, which makes each page cost 100 rows regardless of depth and brings the export down to 400,000 cheap seeks.

    Then ask whether a list endpoint is the right surface at all: a bulk export job producing a file, or a change feed the partner consumes incrementally, replaces 400,000 requests with one. The answer that earns the most credit is noticing that the correct fix is not pagination but a different operation, since a partner who wants everything nightly is asking for a snapshot rather than a page.

### 8f. Check yourself

**Q1.** Give the two failure modes of offset pagination on a mutating collection, and say which one is worse and why.

??? note "Show answer"

    Inserts above the window cause duplicates: the reader sees items already seen. Deletions above the window cause skips: items are never shown at all.

    Skips are worse, and the reason is detectability. A duplicate is visible to the user and to the client, which can deduplicate by identifier. A skipped item leaves no trace anywhere, so a partner reconciling records against yours finds a gap weeks later with no way to reconstruct what was missed. Silent data loss beats visible duplication on the severity scale every time.

**Q2.** A cursor is built from `created_at` only. A bulk import writes 500 rows with an identical timestamp. What happens, and what is the one line fix?

??? note "Show answer"

    With a strict comparison, the seek jumps past the whole equal-timestamp block, so up to 500 rows are silently skipped. With a non strict comparison, the same block returns forever and the client loops.

    The fix is to make the sort key a pair, the timestamp plus a unique identifier, and to compare the pair rather than the timestamp. In the query this is a row comparison on `(created_at, id)`, and the index must cover both columns in that order or the seek degrades into a scan.

### 8g. When not to use this

| Mechanism | Measurement that justifies it | Below this threshold it is over-engineering | Cheaper alternative |
|---|---|---|---|
| Any pagination at all | Collections that can exceed a few hundred items, or responses above roughly 100 kilobytes | A bounded collection such as a user's payment methods or an account's roles | Return the whole list and say in the documentation that it is bounded |
| Keyset or cursor rather than offset | The collection mutates while being read, or offsets exceed roughly 10,000 | An admin table over 2,000 static rows where jumping to page 40 is a real requirement | Offset and limit, with a documented maximum offset |
| A ranking snapshot | The sort key changes during a normal reading session | Sorts on immutable fields such as creation time | Keyset on the immutable field |
| Bidirectional cursors | Users actually navigate backwards, which you can see in request logs | Infinite scroll in one direction | Forward cursors only |

The strongest form of this answer names when pagination is the wrong tool entirely: a caller who wants the whole collection every night needs an export or a change feed, and paginating them is optimising the wrong operation.

---

## Module 9. Idempotency: making retries safe on operations that move money

**Time: 30 to 35 minutes reading, 25 to 35 minutes practice.**

### 9a. Predict first

A client charges a card. Here is the log from both sides.

| Time | Client | Server |
|---|---|---|
| 12:00:00.000 | Sends create-charge request | |
| 12:00:00.400 | | Receives it, authorises with the gateway |
| 12:00:02.900 | | Gateway confirms, server begins writing the record |
| 12:00:03.000 | Times out at 3 seconds, connection closed | |
| 12:00:03.100 | | Write committed, tries to respond, socket gone |
| 12:00:03.200 | Retries the identical request | |

??? note "Show answer"

    Prediction asked for: without any extra mechanism, what does the customer's statement look like, and what is the smallest piece of information that would have prevented it?

    Two charges. The first succeeded fully on the server and only the response was lost, so the retry is indistinguishable from a genuine second purchase.

    The smallest sufficient piece of information is a caller generated identifier that is the same on the original request and on the retry, and different on a genuine second purchase. Nothing the server can compute on its own can make that distinction, because both requests are byte identical and arrive 200 milliseconds apart, which is also a plausible gap for an impatient human clicking twice.

    That is the whole argument for idempotency keys in one paragraph: the information required lives with the caller, so the contract has to carry it.

### The definitions, kept separate

| Term | Meaning | Where it lives |
|---|---|---|
| Safe | The operation does not change state, so it can be repeated freely | Reads |
| Idempotent | Repeating it leaves the same state as doing it once | Full replacement and deletion, by nature |
| Idempotency key | A caller supplied identifier that makes a non idempotent operation repeatable | Creation and other state advancing operations |
| Effectively once | At-least-once delivery plus deduplication at the receiver | The only achievable form of "exactly once" across a network |

The last row is worth saying out loud in a round. There is no exactly-once delivery over an unreliable network, because the acknowledgement can be the message that is lost. What you build is at-least-once delivery plus a deduplication record, and calling that "exactly once processing" is fine as long as you can say where the deduplication state lives.

### Method level idempotency, and what it does not cover

| Operation | Idempotent by nature | Caveat |
|---|---|---|
| Read | Yes | None |
| Full replacement of a resource at a caller chosen identity | Yes | Only if the body is a complete state, not a delta |
| Delete | Yes, in effect | The second call returns "not found", which callers often treat as an error |
| Create at a server chosen identity | No | This is the case that needs a key |
| Partial update with absolute values | Yes | Setting a field to 5 twice is the same as once |
| Partial update with relative values | No | Incrementing by 5 twice is not the same as once |

The last two rows are the ones that catch people. `balance = 100` is idempotent, `balance += 5` is not, and the two look nearly identical in a request body. If your update format allows relative operations, every one of them needs a key.

### The key's lifecycle

An idempotency key is not a flag, it is a small state machine with a record behind it.

```
             +-----------------+
 first call  | NEW             |
 ----------> | record created  |
             | lock held       |
             +-----------------+
                     |
        +------------+------------+
        |                         |
        v                         v
+----------------+       +------------------+
| COMPLETED      |       | FAILED           |
| response saved |       | terminal error   |
+----------------+       | saved            |
        |                +------------------+
        | replay                  |
        v                         v
+-------------------------------------------+
| Later identical call returns saved         |
| response  concurrent call gets 409 retry   |
+-------------------------------------------+

Key states. The saved response is what makes a replay identical
rather than merely harmless
```

Four rules make it correct, and each maps to a failure it prevents.

| Rule | Failure it prevents |
|---|---|
| Store a fingerprint of the request body with the key | A caller reuses a key with different content and gets someone else's result |
| Same key plus different fingerprint returns a specific error, not the old result | Silent wrong-answer, the worst outcome available |
| A second call while the first is in flight returns a conflict with a retry-after | Two workers both authorising the same charge |
| The key record and the business write commit in one transaction | The charge exists but the key does not, so the retry charges again |

That last rule is the one that separates a correct implementation from a plausible one. If the key is written to a separate store after the charge succeeds, there is a window in which a crash leaves the charge without its key.

### Sizing the key store

Inputs, stated here: 500 state changing writes per second sustained, a 24 hour key lifetime, and an assumed 300 bytes per record. The 300 bytes is a rough sum of a 36 byte key, a 32 byte fingerprint, roughly 200 bytes of saved response and some metadata, and it is reasonable for a small creation response.

| Quantity | Derivation | Result |
|---|---|---|
| Keys per day | 500 x 86,400 | 43.2 million |
| Steady state size at 24 hour retention | 43.2M x 300 bytes | About 13 gigabytes |
| Steady state size at 7 day retention | 13 GB x 7 | About 91 gigabytes |
| Extra write cost | One small write per state changing request | Roughly doubles write count on those paths |

13 gigabytes is small, which is the honest conclusion: idempotency keys are cheap at this scale and the reason people avoid them is complexity, not storage. The number that actually needs care is the lifetime.

**Choosing the lifetime.** It must exceed the caller's maximum retry window. If clients retry with exponential backoff up to 6 hours, a 24 hour lifetime gives a factor of 4 in hand. Setting it to 1 hour would silently reintroduce double charges for any client whose queue backs up, and that failure appears only during an incident, which is exactly when you cannot afford it.

### 9d. Worked example

Prompt: "Design idempotency for a card authorisation operation on a public payments interface."

**Block 1. PURPOSE: decide who generates the key, because getting this wrong makes the whole mechanism useless.**

The caller generates it, once per logical purchase, and reuses it on every retry of that purchase. I state the two failure shapes explicitly: a caller that generates a new key per HTTP attempt gets duplicate charges, and a caller that reuses one key across different purchases gets a stale response for the second purchase.

Because both failures are the caller's to make, the documentation must state the rule in one sentence and the interface must detect the second case.

**The error most readers make here** is letting the server generate the key. A server generated key cannot survive the response being lost, which is the only case that matters.

!!! note "Say it before you read on"

    Why can the server not simply hash the request body and use that as the key?

**Block 2. PURPOSE: pin the conflict semantics, because they are where implementations quietly differ.**

Three cases, three answers:

- Same key, same fingerprint, previous call finished: return the saved response, with a header saying it was a replay.
- Same key, different fingerprint: return a specific error. Never the old response, and never a fresh charge.
- Same key, previous call still in flight: return a conflict with a short retry-after, because a partial result cannot be returned safely.

**The error most readers make here** is returning the saved response for a mismatched fingerprint, which turns a caller bug into a silent wrong answer that reconciliation finds days later.

**Block 3. PURPOSE: make the record and the money movement atomic, because a gap between them recreates the original bug.**

The key record and the authorisation record commit in one transaction in the same store. If the gateway call itself sits outside that transaction, the sequence is: write the key row in state IN_PROGRESS, call the gateway, then commit the outcome onto the same row.

A crash between the gateway call and the commit leaves a row in IN_PROGRESS, so the recovery path must query the gateway by our own reference to learn what happened rather than assuming failure.

**The error most readers make here** is assuming a crashed request did nothing. In payments, the default assumption must be that it may have done everything.

!!! note "Say it before you read on"

    What identifier must be sent to the gateway for that recovery query to be possible?

**Block 4. PURPOSE: size and bound the mechanism, so it is a design rather than an aspiration.**

Using the module's inputs, 500 writes per second with a 24 hour lifetime and 300 byte records gives about 13 gigabytes, which is a single modest store. I set the lifetime at 24 hours against a documented maximum client retry window of 6 hours, and I state that the key is scoped per account so two customers cannot collide.

**Result.** Caller generated key, three conflict cases, one transaction, a stated lifetime with a stated margin, and a recovery path that assumes the work may have happened. The mechanism is roughly fifty lines of server code, and every one of those five decisions is where a candidate can be probed.

### 9e. Fade the scaffold

**Faded example 1.** Prompt: "Add idempotency to an operation that creates a document and also sends an email." Blocks 1 to 3 are given. Produce block 4.

- Block 1: the caller supplies the key, scoped per workspace.
- Block 2: the three conflict cases are handled as above.
- Block 3: the key record and the document write commit together.

??? note "Show answer"

    Block 4 has to deal with the fact that the email is not transactional. Committing the document and the key atomically does nothing for the send, which can happen zero, one or two times independently.

    The correct shape is to make the send a consequence rather than a step: commit the document, the key and an outbox row in one transaction, then have a worker read the outbox and send. The worker is at-least-once, so the email provider needs its own deduplication identifier derived from the outbox row id, which is what converts at-least-once sending into effectively-once delivery.

    The common wrong answer is to send the email inside the request and treat a duplicate as acceptable. Say the cost out loud instead: a retried request without an outbox sends the customer two emails, and the number of retries during an incident is exactly when this is most visible.

**Faded example 2.** Prompt: "A partner reports occasional duplicate refunds. Their client retries on any 5xx, and our refund operation takes an idempotency key." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: the key mechanism exists and appears correct in code review.
- Block 2: duplicates cluster during periods when our database is slow.

??? note "Show answer"

    Block 3: the diagnosis follows from the clustering. If duplicates only appear under load, the likely cause is that the key record and the refund are not committed atomically, or the key is written after the refund, so a timeout between the two leaves a refund with no key and the retry creates a second one. The second candidate is a key lifetime shorter than the partner's retry window, which would show the same clustering because backed up queues retry late.

    Both are testable without guessing: check whether any refund rows exist with no corresponding key record, and compare the age distribution of retries against the configured lifetime.

    Block 4: the fixes differ. For the atomicity bug, move the key write into the same transaction and add a reconciliation job that finds refunds without keys. For the lifetime bug, raise the lifetime above the partner's maximum retry window and publish that window as part of the contract, since a retry window the partner chooses and you do not know is an unpriced dependency.

### 9f. Check yourself

**Q1.** A candidate says "we do not need idempotency keys because our create endpoint uses PUT with a client chosen id, which is idempotent." Is that right? State the condition under which it is.

??? note "Show answer"

    It is right when the body is a complete state rather than a delta, and when the client genuinely chooses the identifier before the first attempt, so a retry writes to the same address. Under those conditions repeating the write leaves the same state, and no key is required.

    It stops being right in three cases: the operation has side effects outside that resource such as charging a card or sending a message, the body contains relative operations such as increment, or the client generates the identifier per attempt rather than per logical operation. The last one is common and turns the whole argument upside down, because a fresh identifier per attempt is exactly a fresh create per attempt.

**Q2.** Size the key store for 2,000 state changing writes per second with a 48 hour lifetime, and state the one number you would check before accepting your own answer.

??? note "Show answer"

    Keys per day is 2,000 x 86,400 = 172.8 million. Over 48 hours that is 345.6 million records, and at the module's assumed 300 bytes per record it is about 104 gigabytes.

    The number to check before accepting it is the assumed 300 bytes, because it is dominated by the saved response body. If responses are 5 kilobytes rather than 200 bytes, the store is roughly 1.8 terabytes instead of 104 gigabytes, which changes the storage decision completely. The fix in that case is to save a response reference rather than the response, and rebuild the body on replay.

### 9g. When not to use this

| Mechanism | Measurement that justifies it | Below this threshold it is over-engineering | Cheaper alternative |
|---|---|---|---|
| Full idempotency keys with saved responses | The operation has an irreversible external effect, and callers retry on timeout | Internal operations whose duplication is detected and repaired by a downstream reconciliation anyway | A unique constraint on a natural business key, which rejects the duplicate with a conflict |
| Saved response replay | Callers need the original result, for example a generated identifier | Callers only need to know it succeeded | Return a conflict on the duplicate and let the caller read the resource |
| An in-flight conflict state | Concurrent duplicate submissions actually occur, visible as two requests with one key inside a second | Single threaded callers | Rely on the unique constraint on the key and return the loser a conflict |
| A dedicated key store | More than roughly a few hundred state changing writes per second, or a retention requirement longer than the main store's | Low volume interfaces | A unique index on a key column in the existing table, expired by a periodic delete |

The cheapest correct answer is worth naming in a round: for many creation operations, a unique constraint on a caller supplied business identifier gives you idempotency with one line of schema and no new store. Reach for the full mechanism when you must return the *original response*, not merely prevent the duplicate.

---

## Module 10. Concurrency and caching inside the contract

**Time: 20 to 25 minutes reading, 15 to 20 minutes practice.**

### 10a. Predict first

Two support agents edit the same customer record. The interface offers a read and a full replacement write, with no version checking.

| Time | Agent A | Agent B | Stored record |
|---|---|---|---|
| 10:00:00 | Reads: phone `555-0100`, tier `standard` | | phone `555-0100`, tier `standard` |
| 10:00:05 | | Reads the same record | phone `555-0100`, tier `standard` |
| 10:00:40 | | Writes tier `premium`, phone unchanged | phone `555-0100`, tier `premium` |
| 10:00:55 | Writes phone `555-0199`, tier unchanged | | ? |

??? note "Show answer"

    Prediction asked for: what is stored at 10:00:56, what did the system tell each agent, and how long until anyone notices?

    Stored: phone `555-0199`, tier `standard`. Agent A's write carried the whole record including the tier value read at 10:00:00, so it overwrote the upgrade with a stale value.

    Both agents were told their write succeeded, which is the actual defect. Nobody was informed of anything, and the tier reverting is invisible until the customer calls about a benefit they were promised, which could be weeks.

    This is the lost update problem, and note what caused it: not a database race, but a contract that offers a full replacement with no way for the caller to say which version it was replacing.

### The vocabulary, defined before use

| Term | Definition |
|---|---|
| Validator | A short token representing a specific version of a representation, returned with reads |
| Strong validator | Changes whenever the bytes change, so it is safe for comparisons and for writes |
| Weak validator | Changes only on semantically significant edits, so it is usable for caching but not for writes |
| Conditional read | A read that says "only send the body if the version differs from the one I hold" |
| Conditional write | A write that says "only apply this if the current version is still the one I read" |
| Optimistic concurrency | Allow concurrent edits, detect the collision at write time, reject the loser |
| Pessimistic concurrency | Take a lock before editing so collisions cannot happen |

### The one mechanism that fixes both problems

A single token, returned on read and echoed on write, solves caching and lost updates at once.

| Direction | The caller sends | The server answers | Effect |
|---|---|---|---|
| Read | The token it already holds, as a not-modified condition | 304 with no body when unchanged | Saves the body bytes |
| Write | The token it read, as a must-match condition | 412 when the current version differs | Prevents the lost update |

Three token sources, and they are not equivalent:

| Source | Cost | Trap |
|---|---|---|
| Hash of the serialised representation | Compute on every read, or cache alongside | Any field ordering change invalidates every token at once |
| A monotonically increasing version integer on the row | One column, nearly free | Must be incremented in the same transaction as the write |
| Last modified timestamp | Free, already there | Resolution is usually one second, so two writes in the same second are indistinguishable |

Timestamps are the trap worth naming in a round. At an assumed one second resolution, any entity edited more than once per second has a validator that cannot distinguish versions, so the conditional write silently permits the lost update it exists to prevent.

```
+--------+  read              +--------+
| Client | <----------------- | Server |
|        |  body plus token 7 |        |
+--------+                    +--------+
     |
     |  write with must match token 7
     v
+--------+                    +--------+
| Server | ----------------->            token is now 8
| checks | current token is 8 | reject |
| token  |                    | 412    |
+--------+                    +--------+

Optimistic concurrency. The rejection is the feature, because the
client learns it was editing a version that no longer exists
```

### How often will the rejection fire?

This decides whether "reject the loser" is a usable design or a support nightmare, and it is derivable.

Inputs: an edit session takes an assumed 30 seconds between the read and the write, which is reasonable for a human filling a form, and other edits to the same entity arrive independently at a rate of λ per second.

Probability that at least one other edit lands in the window is 1 minus e to the power of minus λ times 30, which for small values is close to λ times 30.

| Entity | Edits per hour | λ per second | Collision probability per edit |
|---|---|---|---|
| A typical customer record | 4 | 0.0011 | 30 x 0.0011 = 3.3 percent |
| A busy shared document | 120 | 0.0333 | 1 minus e^-1 = 63 percent |

The design consequence is sharp. At 3.3 percent, rejecting the loser and asking the human to reapply is fine, and at 1,000 edits per day it produces about 33 retries. At 63 percent, rejection is unusable, and the interface needs either field level merge or a genuinely collaborative model with a different contract shape. Choosing between them without this number is guessing.

### The caching number

Inputs: a 200 kilobyte representation, a not-modified response of an assumed 300 bytes of headers, a client polling every 30 seconds, and an assumed 95 percent of polls finding no change. The 95 percent is reasonable for a resource that changes a few times a day and is polled 2,880 times.

| Strategy | Daily bytes for one polling client |
|---|---|
| Unconditional polling | 2,880 x 200 KB = 576 megabytes |
| Conditional polling | 144 x 200 KB plus 2,736 x 300 B = about 29 megabytes |

That is a factor of 20 for one header. What it does not buy is a saved round trip: the client still pays the network time on every poll, so if latency rather than bandwidth is the problem, conditional requests are the wrong tool and a change feed or a push channel is the right one.

### Choosing a conflict policy

| Policy | What the caller sees | When it is right |
|---|---|---|
| Reject the loser | A specific conflict error naming the current version | Collision rate under roughly 5 percent, and a human can reapply |
| Last write wins | Success, and silent data loss | Only for fields where the newest value is definitionally correct, for example a presence heartbeat |
| Field level merge | Success, with a merged result | Fields are independent and the domain allows partial application |
| Explicit merge resource | A conflict object the caller resolves | High value data where losing an edit is unacceptable |

Field level merge deserves a warning: it is only safe when no invariant spans two fields. If a discount percentage and a total must agree, merging them independently produces a record that no single author ever intended, and the invariant check has to run after the merge rather than on either input.

### 10d. Worked example

Prompt: "Design the concurrency story for an interface where support agents and customers both edit an account record."

**Block 1. PURPOSE: measure before choosing, because the policy follows from the collision rate.**

I ask for two numbers: edits per account per hour, and how long an edit session lasts. The answers are roughly 4 per hour on active accounts and about 30 seconds of form filling.

Using the arithmetic above, 30 x 4 divided by 3,600 gives about 3.3 percent. That is low enough that rejecting the loser is acceptable, and I say the number out loud rather than asserting that optimistic concurrency is fine.

**The error most readers make here** is choosing optimistic or pessimistic concurrency from taste. The rate decides it, and the rate is one question away.

!!! note "Say it before you read on"

    At what collision percentage would you stop rejecting the loser, and what would you do instead?

**Block 2. PURPOSE: pick a validator that cannot silently fail.**

I use a version integer on the account row, incremented in the same transaction as every write. I reject last modified timestamps explicitly, because at an assumed one second resolution two writes inside the same second share a validator, and that is precisely the case optimistic concurrency exists to catch.

I return the version on every read and require it on every write. A write without it is rejected rather than allowed, since an optional precondition is one that clients omit under deadline pressure.

**The error most readers make here** is making the precondition optional to be friendly to integrators. That converts a designed safety property into an opt-in, and the callers who skip it are the ones causing the incidents.

**Block 3. PURPOSE: make the rejection actionable, because a bare conflict error moves the problem to the caller.**

The 412 response carries the current version, the fields that changed since the caller's version, and who changed them. With that, a client can show "the tier was changed to premium by another agent 40 seconds ago" instead of "please try again".

**The error most readers make here** is returning the conflict with no current state, which forces a re-read and leaves the user with no idea what happened.

!!! note "Say it before you read on"

    Why is returning the *changed field set* better here than returning the whole current record?

**Block 4. PURPOSE: connect concurrency to caching, since one token serves both.**

The same version token is used as the read validator, so agent dashboards polling every 30 seconds get not-modified responses. Using the module's numbers, that takes one polling client from 576 megabytes a day to about 29.

I state the limit honestly: this saves bytes, not round trips, so if the dashboard needs sub-second freshness the answer is a push channel and not a shorter poll interval.

**Result.** One version integer, required on writes, echoed on reads, a conflict response carrying the diff, and a measured collision rate that justified the policy. Every choice traced to a number the interviewer watched me derive.

### 10e. Fade the scaffold

**Faded example 1.** Prompt: "Add concurrency control to a collaborative pricing sheet where a dozen analysts edit different cells of the same document all afternoon." Blocks 1 to 3 are given. Produce block 4.

- Block 1: measured edits are about 200 per hour on a busy sheet, with sessions of a few seconds.
- Block 2: at a 5 second window and 200 edits per hour, λW is 5 x 200 / 3,600 = 0.28, so collision probability is 1 minus e^-0.28, about 24 percent.
- Block 3: 24 percent rejection would be intolerable for a document analysts edit continuously.

??? note "Show answer"

    Block 4 must change the granularity rather than the policy, and that is the general lesson: when the collision rate is too high, shrink the thing being versioned.

    Version each cell rather than the document. A cell is edited far less often than the document, so with the same 5 second window and, say, 2 edits per hour per cell, λW falls to 5 x 2 / 3,600 = 0.0028, or under one third of one percent. Optimistic concurrency becomes viable again without changing the mechanism at all.

    The cost has to be stated too: per cell versioning means a write carries many preconditions, and any invariant spanning cells (a column that must sum to a total) can no longer be enforced by a single conditional write. That invariant now needs either a document level version alongside the cell versions, or a recomputation after the fact.

**Faded example 2.** Prompt: "Our mobile clients poll a settings resource every 10 seconds and our egress bill is dominated by it." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: the representation is 40 kilobytes and settings change roughly twice a day.
- Block 2: there are 200,000 active clients at any moment.

??? note "Show answer"

    Block 3: price it first. Each client makes 8,640 polls a day, so 200,000 clients make 1.73 billion requests a day, at 40 kilobytes each, which is about 69 terabytes daily. With conditional requests and an assumed 300 byte not-modified response, essentially all of those become 300 bytes, giving about 0.52 terabytes plus a negligible amount for the two real changes. That is a reduction of more than 100 times from one response header and one request header.

    Block 4: then question the poll interval, because 1.73 billion requests a day is still a lot of connections for data that changes twice. Two changes worth proposing: raise the interval and add a freshness lifetime so intermediaries and the client itself can skip requests entirely, and offer a push or long poll channel for the rare change. The correct final answer usually combines both, and the honest note is that conditional requests fixed the bytes while only the interval change fixes the request count.

### 10f. Check yourself

**Q1.** Explain in two sentences why last modified timestamps are unsafe as write preconditions, and give the situation where they are still fine for reads.

??? note "Show answer"

    At an assumed one second resolution, two writes inside the same second produce the same validator, so a conditional write comparing timestamps cannot detect that the entity changed and permits exactly the lost update it was added to prevent.

    They remain fine as read validators on resources that change far less often than once per second, for example a daily generated report or a static configuration document, where the worst case of a stale cache hit is bounded by the same one second window.

**Q2.** An entity has a 40 percent collision rate under optimistic concurrency. Name two structurally different fixes and the cost of each.

??? note "Show answer"

    Fix one: shrink the versioned unit, so each field or sub-object carries its own version. The collision rate falls in proportion to how the edits spread across the smaller units, and the cost is that invariants spanning units can no longer be enforced by one conditional write.

    Fix two: change the write shape from replacement to intent, so the client sends "add 5 to quantity" or "set tier to premium" and the server applies it against current state. Collisions largely disappear because concurrent intents commute, and the cost is that the operations are no longer idempotent, so every one of them now needs an idempotency key from module 9.

### 10g. When not to use this

| Mechanism | Measurement that justifies it | Below this threshold it is over-engineering | Cheaper alternative |
|---|---|---|---|
| Conditional writes with validators | Two or more independent writers per entity, and a measured collision rate above roughly 1 percent | Single writer entities, for example a user editing only their own profile from one device | A plain write, with the write path logged |
| Conditional reads with validators | Polling clients, and a payload above roughly 10 kilobytes | A resource fetched once per session | A freshness lifetime on the response |
| Field level merge | Collision rate above roughly 20 percent with independent fields | Anything below that, where rejection is rarer than a user changing their mind | Reject the loser and show the diff |
| Collaborative editing models | Continuous concurrent editing of the same small region by multiple people | Occasional overlap between two agents | Optimistic concurrency at a finer granularity |

The threshold that matters is the collision rate, and it is cheap to obtain: count writes per entity per hour in your existing logs and multiply by the edit window. Designing a merge strategy before taking that measurement is the classic over-engineering path here.

---

## Module 11. Versioning, evolution and the clients you cannot upgrade

**Time: 25 to 30 minutes reading, 20 to 30 minutes practice.**

### 11a. Predict first

Four changes are proposed for a public interface currently at major version 2, with about 1,200 active integrations.

| Change | Description |
|---|---|
| 1 | Add an optional `preferred_language` field to the customer response |
| 2 | Rename `zip` to `postal_code` across every address |
| 3 | Change the default page size from 20 to 50 |
| 4 | Stop returning cancelled bookings from the default list, requiring an explicit filter |

??? note "Show answer"

    Prediction asked for: which of these require a new major version, and which is the most dangerous even though it looks smallest?

    | Change | Needs a major version | Note |
    |---|---|---|
    | 1 | No | Additive and optional, safe for readers by module 6's rules |
    | 2 | Not necessarily | Ship both fields, deprecate one, remove after the sunset date |
    | 3 | No, but it is still breaking for some callers | A caller that assumed 20 and sized a buffer or a screen now gets 50 |
    | 4 | Yes in effect, and it is the dangerous one | The schema does not change at all, so every automated check passes |

    Change 4 is a semantic change: same fields, same types, different meaning of the collection. A reconciliation script that counts bookings silently starts reporting fewer, and nothing in any pipeline notices. Change 3 is the same class in miniature, which is why default values are part of the contract even though nobody writes them into the schema.

### What this module adds to module 6

Module 6 gave the change taxonomy and the compatibility rules. This module answers a different question: **the change is breaking, it must ship anyway, and some of your callers will never upgrade.**

Four things have to be decided, and candidates typically decide only the first.

| Decision | The common weak answer | What a strong answer adds |
|---|---|---|
| How versions are identified | "Put v2 in the URL" | Which unit is versioned: the whole surface, the operation, or the representation |
| How long the old version lives | "Until people migrate" | A date derived from the migration curve, published in advance |
| How old callers keep working | "They upgrade" | A translation layer, pinning, or dual serving, with an owner |
| How you know a change broke someone | "Tests" | Consumer expectations verified in the provider's build |

### Versioning schemes, priced honestly

| Scheme | What it is good at | What it costs | Where it hurts |
|---|---|---|---|
| Major version in the path | Obvious, cacheable, easy to route and to document | Every version is a whole parallel surface | Callers upgrade all operations at once or not at all |
| Version in a request header | Fine grained, keeps one set of paths | Invisible in a browser and in logs unless you add it | Easy to forget, so a missing header needs a defined default |
| Media type negotiation per representation | Versions the thing that actually changed | Poor tooling support, unfamiliar to most integrators | Support load from confused callers |
| Date based pinning per caller | Each caller moves independently, small diffs | You maintain a chain of transformations between dates | The chain becomes the most complex code you own |
| No version, additive only | Zero migration work forever | You can never remove or rename anything | The surface accumulates dead fields no one dares delete |

The recommendation worth defending: **version the surface coarsely, evolve additively inside it, and treat a major version bump as a rare and expensive event.** A team that bumps a major version to rename a field has chosen the most expensive mechanism for the cheapest problem.

### The sunset date is derived, not chosen

Reusing the migration model from module 6: 5 million installs and an assumed 90 day upgrade half life, plus a stated policy of stranding no more than 5 percent of callers.

Solve for the time at which one half raised to the power of t divided by 90 equals 0.05. That is t equal to 90 times the base-2 logarithm of 20, which is 90 x 4.32, or about 389 days.

| Policy threshold | Time to reach it | Practical sunset |
|---|---|---|
| 50 percent remaining | 90 days | Far too early |
| 10 percent remaining | 90 x 3.32 = 299 days | Aggressive but arguable |
| 5 percent remaining | 389 days | About 13 months |
| 1 percent remaining | 90 x 6.64 = 598 days | About 20 months |

Two consequences to state out loud. First, a 12 month deprecation window is not generosity, it is arithmetic. Second, the tail never reaches zero, so at some point you switch from waiting to contacting the remaining callers individually, which is only possible if you can identify them per caller. That reporting requirement is a design decision made now, not later.

### Keeping old callers working

| Mechanism | How it works | When it is the right one |
|---|---|---|
| Translation at the edge | One current implementation, plus adapters that reshape old requests and responses | The change is structural, for example a field split or a rename |
| Dual serving | Two implementations behind one router | The change is behavioural and cannot be expressed as a transformation |
| Per caller pinning | Each caller records the contract date it was built against | Many small changes over a long period |
| Feature flag per tenant | Behaviour switches per account | Semantic changes such as the cancelled bookings example |

The rule that keeps this from sprawling: **transformation logic lives in exactly one layer, and the core implementation never contains a version check.** Once `if version == 2` appears inside the domain code, every future change costs more, and the code becomes unremovable because nobody can prove which branch is dead.

```
+----------+   +----------+   +----------+
| Caller   |   | Caller   |   | Caller   |
| pinned   |   | pinned   |   | current  |
| 2024-06  |   | 2025-03  |   |          |
+----------+   +----------+   +----------+
      |              |              |
      v              v              |
+-------------+ +-------------+     |
| Adapter to  | | Adapter to  |     |
| current     | | current     |     |
+-------------+ +-------------+     |
      |              |              |
      +------+-------+--------------+
             v
      +---------------+
      | Core domain   |
      | no version    |
      | branches      |
      +---------------+

Adapters absorb every past contract so the core has one shape.
Version checks inside the core are what make removal impossible
```

### Consumer driven contract testing, and what it catches

A schema compatibility check knows the shape of your contract. It does not know which parts anyone uses. Consumer driven contract testing closes that gap: each consumer publishes the requests it makes and the response fields it depends on, and the provider's build verifies every published expectation before release.

| Failure | Caught by schema checks | Caught by consumer contracts |
|---|---|---|
| Removing a field | Yes | Yes, and it names who used it |
| Adding a required request field | Yes | Yes |
| Changing a default value | No | Yes, if a consumer asserted on it |
| Narrowing what a list returns | No | Yes, if a consumer asserted on it |
| A field that no consumer uses at all | No | Yes, and this is the one that lets you delete safely |

That last row is the underrated one. The main reason interfaces accumulate dead fields is that nobody can prove a field is unused. Consumer contracts turn that from a guess into a query.

### 11d. Worked example

Prompt: "We must rename `zip` to `postal_code` across our public interface. There are 1,200 active integrations and we cannot force any of them to change."

**Block 1. PURPOSE: reject the framing before designing, because the first question is whether this ships at all.**

I state the cost in the same breath as the benefit. The benefit is naming consistency for international addresses. The cost, from the arithmetic above, is roughly 13 months of dual fields plus a removal campaign against a long tail.

I then propose the smaller change: apply the new name to new operations and new types only, and grandfather the existing ones with a documented exception. If the interviewer wants the full rename anyway, I continue, having shown that the trade was priced rather than assumed.

**The error most readers make here** is starting with the migration plan. The most senior move available is asking whether a cosmetic change justifies a year of dual maintenance.

!!! note "Say it before you read on"

    Name a version of this change that would justify the 13 months, and say what makes it different.

**Block 2. PURPOSE: choose the mechanism that does not create a second surface.**

I do not bump the major version. A rename is a field level change, and a major bump forces 1,200 integrators to revalidate every operation to receive one renamed field.

Instead: emit both fields on responses, accept either on requests with a documented precedence rule when both are present and differ, and mark `zip` deprecated in the interface definition with a machine readable sunset date.

**The error most readers make here** is not defining what happens when a caller sends both fields with different values. It happens more often than people expect, usually from a client library merging defaults, and an undefined rule becomes an inconsistent one.

**Block 3. PURPOSE: signal the deprecation in ways a machine can act on, not only a human.**

Three signals: a deprecation marker plus a sunset date in the definition file, response headers carrying the same information on requests that touched the deprecated field, and per integrator usage reporting so we know exactly who is still sending it.

The third is the one that converts a broadcast email into a targeted conversation, and it only exists if requests are attributable per caller.

!!! note "Say it before you read on"

    Why is the response header worth adding when the deprecation is already in the definition file?

**Block 4. PURPOSE: derive the removal date and the exit criteria, so the deprecation ends.**

The date comes from the curve: with an assumed 90 day half life, reaching 5 percent takes about 389 days, so the published sunset is 13 months out and stated on the day the deprecation ships.

Exit criteria are two, not one: the date has passed, and per caller usage shows fewer than 5 percent of integrations still sending or reading `zip`. If the second is not met, the response is not an extension by default but a decision to contact the remaining callers, since an indefinite extension is how a deprecation becomes permanent.

**Result.** No major version, dual fields for 13 months, a precedence rule for the ambiguous case, three deprecation signals, a derived date and a named exit condition. The credit is in the date arithmetic and the exit criteria, because those are what make the deprecation finishable.

### 11e. Fade the scaffold

**Faded example 1.** Prompt: "Our default list operation returns cancelled bookings. We want it to exclude them by default." Blocks 1 to 3 are given. Produce block 4.

- Block 1: this is a semantic change with no schema diff, so no automated check catches it.
- Block 2: the affected callers cannot be identified from the schema, only from behaviour.
- Block 3: an explicit `status` filter is added first, so callers can express either intent before the default changes.

??? note "Show answer"

    Block 4 is the rollout, and the shape is forced by the fact that the change is invisible to tooling. Instrument first: log which callers request the list without a status filter and whose responses actually contained cancelled bookings, since those are the only callers who can be affected.

    Then flip per tenant rather than globally, starting with callers whose usage shows they never read a cancelled booking's fields, and announce the default change with a date to the rest. New integrations get the new default immediately, which stops the affected population from growing while the migration runs.

    The wrong answer is a major version bump. Versioning the entire surface to change one default forces 1,200 integrators to revalidate everything, and most of them are not affected at all.

**Faded example 2.** Prompt: "A field we want to delete is used by an unknown number of the 1,200 integrations." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: the schema check cannot tell you who reads a response field, because reading leaves no trace in the request.
- Block 2: consumer driven contracts exist for the 40 internal consumers but not for external integrators.

??? note "Show answer"

    Block 3: for external callers, reading is unobservable, so the usage question has to be made observable. Three options, in increasing cost: offer sparse field selection and see who asks for the field, ship a per caller opt-out and watch who complains, or run a dark period where the field is omitted for a small percentage of responses for a short window and monitor error rates and support contacts. The third gives a real answer and carries real risk, so it needs an announced window and a fast rollback.

    Block 4: the decision rule should be written before the data arrives, or the result will be argued rather than acted on. For example: remove on the sunset date if measured usage is under 5 percent and no caller in the top 50 by volume is affected, otherwise extend once by 90 days with direct outreach to the identified callers. Publishing the rule in advance is what stops a deprecation from being extended forever, one exception at a time.

### 11f. Check yourself

**Q1.** Derive the sunset date for a client population with an assumed 60 day upgrade half life and a policy of stranding no more than 2 percent, then say what you do about the remaining 2 percent.

??? note "Show answer"

    Solve one half raised to t over 60 equals 0.02. That is t equal to 60 times the base-2 logarithm of 50, which is 60 x 5.64, about 338 days, so roughly 11 months.

    For the remaining 2 percent the answer is not waiting, because the tail is exponential and never reaches zero. You need per caller attribution so the remaining integrations can be contacted individually, a decision rule published in advance, and a defined failure behaviour on the sunset date, ideally a clear error naming the change rather than a silent shape difference.

**Q2.** Give one change that a schema compatibility checker passes and a consumer driven contract test catches, and explain the mechanism.

??? note "Show answer"

    Changing a default value, for example a default page size from 20 to 50, or narrowing what a default list returns. The schema is unchanged, so structural checks see nothing.

    The consumer contract catches it because the consumer publishes an actual expectation, for example "when I call this operation with no page size I receive at most 20 items", and the provider's build replays that expectation against the new implementation. The mechanism is that the test asserts on observed behaviour rather than on declared shape, which is exactly the gap semantic changes exploit.

### 11g. When not to use this

| Mechanism | Measurement that justifies it | Below this threshold it is over-engineering | Cheaper alternative |
|---|---|---|---|
| A formal versioning scheme | At least one consumer you cannot deploy, and a change you cannot make additively | All consumers in your own pipeline | Change the interface and deploy everything together |
| Per caller pinning with adapters | Many small breaking changes over years, with callers on widely different vintages | One or two breaking changes ever | A single major version bump with a long sunset |
| Consumer driven contract tests | More than roughly three independently deployed consumers, or semantic changes that schema checks miss | Two services owned by one team | Integration tests in one pipeline |
| A formal deprecation programme with dated headers | External callers, or an install base you cannot force to upgrade | Internal callers you can page | An announcement and a pull request against each consumer |

The honest summary: every mechanism in this module is a tax paid for the right to change something without coordinating with people you do not control. If you can coordinate, do not pay it. If you cannot, the sunset arithmetic tells you exactly how expensive the change is before you commit to it.

## Module 12. Authentication, authorisation and multi-tenancy

**Time: 26 to 32 minutes reading.**

### 12a. Predict first

A token and a route, taken together. Commit to an answer before continuing.

```
+--------------------------------------------------------------+
| ACCESS TOKEN PAYLOAD          |  ROUTE                       |
|  sub:      user_9931          |   GET /tenants/{tenant_id}/  |
|  tenant:   acme               |       documents/{doc_id}     |
|  scope:    admin              |                              |
|  exp:      issued + 90 days   |  handler:                    |
|  no jti, no revocation list   |    load doc by doc_id        |
|                               |    if token.scope == admin:  |
|                               |       return doc             |
+--------------------------------------------------------------+
Caption: a long-lived bearer token and the handler that trusts
the path parameter rather than the token's tenant claim.
```

**Question.** Name the two independent ways a customer of tenant `acme` reads a document belonging to another tenant, and state the worst-case time between "we revoked this token" and "it stops working".

??? note "Show answer"

    Way one, the missing predicate. The handler loads the document by `doc_id` alone and authorises on scope only. Any `acme` admin can pass any `doc_id` and read it. The `tenant_id` in the path is decoration: nothing compares it to `token.tenant`, and nothing compares either of them to the document's owning tenant. This is the single most common cross-tenant bug in real systems, and it survives code review because the route looks scoped.

    Way two, the path as the authority. Even if the handler did compare the document's tenant to the path's `tenant_id`, an `acme` token sent to `/tenants/globex/documents/5` would pass, because the check would be self-consistent and never touch the token. Authorisation derived from a value the caller controls is not authorisation.

    Revocation lag: 90 days. With no `jti` and no revocation list, the only thing that ends a leaked token's life is expiry. A signature check is offline by construction, which is the property you bought, and the price of that property is exactly the token lifetime.

### 12b. Grants, token lifetimes and the two authorisation questions

**Pick a grant for the caller in front of you.**

Terms first. **Authentication** establishes who is calling. **Authorisation** decides what that caller may do. A **principal** is the entity a decision is made about: a user, a service, or a user acting through an application. A **tenant** is the isolation boundary that owns data.

| Grant | Correct when | The identity inside the token | Characteristic misuse |
|---|---|---|---|
| Authorization code with proof key for code exchange | A human is present and the client cannot keep a secret (browser app, mobile app) | The user, plus the client application | Used for a background job, so the job dies when the user's session ends |
| Client credentials | No human is present: service to service, partner batch jobs | The calling service, no user at all | A user-shaped token reused for machine traffic, which then carries a user's permissions into an automated path |
| Device authorisation | Input-constrained clients: a terminal, a set-top box, a command line tool on a headless host | The user, authorised on a second device | Skipped in favour of pasting a long-lived personal token into a script |
| Refresh token with rotation | Any long-lived session that must survive short access token lifetimes | Same as the grant that issued it | Rotation without reuse detection, which throws away the only benefit |
| Static API key | Server-side integrations where the integrator wants one string in a config file | The key, mapped to a tenant and a scope set | Treated as a password and never rotated, because you gave no rotation path |

**The one sentence that separates a senior answer.** Say who the principal is before naming a grant. If the answer is "a service acting on its own behalf", client credentials follows automatically. If the answer is "a service acting on behalf of a user", you need the user's identity to travel with the call, and you must decide whether it travels as a token exchange or as an assertion your service is trusted to make.

**Token lifetime is an arithmetic problem, not a preference.**

Short lifetimes buy revocation speed. They cost refresh traffic. Both sides are computable from two inputs, so this is never a matter of taste.

Stated inputs for the derivation, both labelled: assume 200,000 concurrently active sessions, which is the number an interviewer would give or you would assume aloud, and assume every session refreshes exactly once per access token lifetime.

| Access token lifetime | Refresh operations per second | Worst-case revocation lag |
|---|---|---|
| 5 minutes | 200,000 / 300 = 667 | 5 minutes |
| 15 minutes | 200,000 / 900 = 222 | 15 minutes |
| 60 minutes | 200,000 / 3,600 = 56 | 60 minutes |
| 24 hours | 200,000 / 86,400 = 2.3 | 24 hours |
| 90 days | Negligible | 90 days |

Now the decision falls out. If a refresh is a database write plus a rotation record, 667 writes per second is a real system with its own availability risk sitting in front of every login. 56 per second is not. So a 60 minute access token is the default, and anything shorter needs a reason.

**What to do about the resulting 60 minute lag,** since "we revoked it and it works for an hour" is unacceptable for some operations but fine for most:

| Approach | Revocation lag | Cost |
|---|---|---|
| Wait for expiry | Full token lifetime | Zero |
| Deny-list of revoked token identifiers, checked at the edge | Seconds | One cache lookup per request, sized by revocations per hour times token lifetime, which is small |
| Introspection call on every request | Immediate | A network hop per request, and your auth service becomes a hard dependency of every call |
| Split by sensitivity: long token for reads, short token or step-up for money movement | Seconds where it matters | Two token classes to explain to integrators |

The last row is the opinionated one. Uniform token lifetimes are a false economy: the operations where a 60 minute lag is intolerable are usually under 2 percent of calls, so paying introspection cost on 100 percent of traffic to protect 2 percent is a straight overspend.

**Refresh token rotation, and the part everyone omits.** Rotation means each refresh returns a new refresh token and invalidates the old one. The value is not the rotation, it is the reuse detection: if an already-used refresh token appears, either the client raced itself or the token was stolen, and you cannot tell which. The correct response is to invalidate the whole token family and force re-authentication. Rotation without that response is bookkeeping.

**Scopes, roles and attributes answer different questions.** Three models, and the failure mode of each.

| Model | The decision reads | Wins when | Fails when |
|---|---|---|---|
| Scopes | "This token was granted `invoices:read`" | Third-party delegation, where a user consents to what an app may do | Used as the whole permission model, because a scope cannot say "this invoice", only "invoices" |
| Roles | "This user is a `billing_admin` in this tenant" | Inside a tenant, where administrators think in job functions | Role count grows past what an administrator can reason about, and every exception spawns a new role |
| Attributes | "This user's department matches the document's department and the document is not restricted" | Rules that depend on data, such as ownership, sharing, region or classification | Nobody can answer "who can see this document" without running the engine, and the engine becomes a latency dependency |

**Scope explosion, computed.** With R resource types and A actions per resource, a fully enumerated scope set has R times A members. At 20 resource types and 4 actions that is 80 scopes. A consent screen listing 80 items is a consent screen nobody reads, so the practical ceiling is around 10 to 15 groups: bundle by resource family and by read versus write, and reserve fine-grained control for the roles layer inside the tenant.

**The layering that actually ships.** Scopes bound what the application may attempt. Roles and attributes decide what this principal may do inside the tenant. Both must pass. Keeping them separate is what lets you answer the two different questions a support ticket asks: "why can this app do that" and "why can this person do that".

### 12c. Multi-tenancy: an isolation ladder with prices

Every rung isolates more and costs more. Pick the lowest rung that satisfies a stated requirement, and state the requirement.

| Rung | Blast radius of a missing predicate | Per-tenant cost | Migration and operations cost | Choose it when |
|---|---|---|---|---|
| Shared tables, `tenant_id` column, predicate in application code | Every tenant | Near zero | One schema change for everyone | Default. Most business software lives here |
| Shared tables plus row-level security enforced by the database | Every tenant, but the database refuses to leak without an explicit override | Near zero | Same, plus a session variable discipline | You want a second line of defence that does not depend on code review |
| Schema per tenant | One tenant | Small, until the catalogue is large | Migrations run N times and take N times as long | Tens to low hundreds of tenants with contractual separation |
| Database per tenant | One tenant | Real: connections, backups, upgrades | Migrations are a fleet operation with partial failure | Regulated data, per-tenant encryption keys, or per-tenant restore requirements |
| Cell or stack per tenant group | One cell | Highest | A deployment system, not a schema change | Very large tenants whose load or availability requirements differ in kind |

**The number that moves you up a rung.** Two conditions, either sufficient. First, a contractual or regulatory requirement for physical separation or independent restore. Second, a single tenant whose share of a shared resource is large enough that its worst day is everyone's worst day.

A workable line for the second condition: if one tenant exceeds roughly 20 to 25 percent of a shared pool's capacity, it is no longer a tenant, it is a co-tenant of your infrastructure, and it should be moved to its own cell before its growth makes the move an emergency.

**Make the predicate impossible to forget.** Code review does not scale as a control. Three mechanisms, in increasing strength:

1. A repository layer where every query builder requires a tenant context object to construct, so an unscoped query does not compile.
2. Database row-level security with the tenant set from a connection-level variable, so a forgotten predicate returns zero rows instead of everyone's rows.
3. A test that runs every query in the suite twice, once as tenant A and once as tenant B, and fails if any query returns identical row identifiers.

**The six cross-tenant leak classes,** because a strong answer names them rather than saying "we check permissions":

| Class | How it happens |
|---|---|
| Missing predicate | The handler looks up by primary key and never joins to tenant |
| Cache key without tenant | Two tenants share a memoised response because the key was the resource id |
| Background job running as an operator | A nightly job holds a superuser connection and writes across boundaries |
| Aggregate or reporting query | A dashboard query is written by hand, outside the repository layer that enforces scoping |
| Error message echo | A 409 or a validation error quotes the conflicting record, which belongs to someone else |
| Identifier enumeration | Sequential ids let a caller measure your other tenants' volume even without reading their rows |

The last one is why opaque, non-sequential identifiers on a public surface are a design choice and not a fashion.

```
+-----------+   +--------------+   +---------------------+
| CALLER    |-->| AUTHENTICATE |-->| RESOLVE TENANT      |
| token or  |   | verify sig,  |   | from token claim,   |
| api key   |   | check deny   |   | never from the path |
+-----------+   | list         |   +----------+----------+
                +--------------+              |
                                              v
              +---------------------+  +------------------+
              | POLICY DECISION     |->| DATA ACCESS      |
              | scope allows the    |  | tenant predicate |
              | action, role or     |  | injected by the  |
              | attribute allows    |  | repository layer |
              | this object         |  | not by the query |
              +---------------------+  +------------------+

Caption: one choke point per request. Tenant comes from the
token, the policy decision needs both the scope and the object,
and the data layer adds the tenant predicate itself so that a
handler cannot omit it. The path parameter is validated against
the resolved tenant and is never the source of it.
```

### 12d. Worked example: permissions for a business document surface

The prompt: 3,000 business tenants share a document service. Tenants want per-folder sharing, some want single sign-on, three large customers are asking for "our data in our own database", and a partner integration needs to read documents on behalf of users.

Stated inputs: 3,000 tenants, largest tenant holds 38 percent of all rows, 4 tenants are in regulated industries with restore-in-isolation clauses, 1 partner integration exists today with 3 more in the pipeline.

**Block 1. Purpose: separate the two authorisation questions before touching a rung of the ladder.**

I split the surface into "which application may act" and "which person may act on which object". The partner integration is the first question: it needs delegated access, so it gets scopes and an authorization code grant with the user present at consent time. Per-folder sharing is the second question: it is object-level and data-dependent, so it needs attributes, not scopes.

The error most readers make here is to answer the whole prompt with scopes, because the partner requirement arrived first and scopes are the thing you name when you hear "third party". A scope named `documents:read` cannot express "this folder, shared with this user, until Friday". Two mechanisms, layered, both required to pass.

!!! question "Say it before you read on"

    Before the next block: state, in one sentence, which of the 3,000 tenants forces a decision about the isolation ladder, and why the other 2,996 do not.

**Block 2. Purpose: pick the lowest rung that satisfies a stated requirement.**

The 4 regulated tenants have a restore-in-isolation clause, which shared tables cannot satisfy at any amount of care: you cannot restore one tenant's rows to a point in time from a shared backup without a bespoke extraction. That is a requirement, not a preference, so those 4 get their own databases.

The tenant holding 38 percent of rows is over my 20 to 25 percent line, so it moves to its own cell. Not for security, for blast radius: its index maintenance, its worst query and its migration duration are already everyone else's problem.

The remaining 2,995 stay on shared tables with row-level security. So the answer is a ladder with three rungs occupied at once, and the sentence that makes it credible is that each exception is attached to a named requirement rather than to a size instinct.

**Block 3. Purpose: make the predicate structurally unforgettable, then prove it.**

Tenant comes from the token claim. The path segment is compared to it and a mismatch is a 404, not a 403, because a 403 confirms the resource exists in another tenant and that is an information leak in itself.

Row-level security is enabled with the tenant set from a connection variable at request entry. The repository layer will not construct a query without a tenant context. Then the proof: the differential test from earlier, running every suite query as two tenants and failing on any shared row identifier. Plus one production canary, a synthetic tenant with a single document that no code path should ever return to anyone else, alerted on.

!!! question "Say it before you read on"

    Before the next block: name the one question a support engineer will ask weekly about this design, and say whether anything built so far can answer it.

**Block 4. Purpose: give the sharing model a shape a support engineer can explain.**

Per-folder sharing is attribute-based, which risks the "nobody can answer who can see this" failure. I buy that back with two things: an effective-permissions endpoint that answers "who can read this document and via which grant", and a rule that every grant is materialised as a row with a source (owner, folder share, role, admin override) so the answer is a join rather than an engine replay.

The error most readers make at this step: designing the evaluation and skipping the explanation. In a business product, "why can this person see this file" is a support ticket that arrives weekly, and a permission model with no readable explanation surface converts each one into an engineering escalation.

### 12e. Fade the scaffold

**Faded example 1.** Last block blanked. A machine-to-machine analytics surface: 40 partner services call a metrics endpoint, no humans involved, each partner may read only its own accounts, and partners rotate infrastructure often.

- Block 1: the principal is a service, so client credentials, and no user identity should appear anywhere.
- Block 2: no isolation ladder decision, because partners read aggregates derived from a shared store and none has a restore clause.
- Block 3: the account filter comes from the credential's mapped partner id, and a partner id in the query string is validated against it, never trusted.
- Block 4, credential lifecycle: **you write this**.

??? note "Show answer"

    Because partners rotate infrastructure often, the credential lifecycle is the whole problem. Two active credentials per partner at all times, so rotation is a rollout rather than a cutover: issue the new one, both work, partner deploys, revoke the old one on the partner's signal or after a stated window.

    Access tokens at 60 minutes, from the arithmetic earlier: at 40 partners refreshing hourly the refresh load is trivial, and 60 minutes of revocation lag on read-only aggregates is acceptable. Add a deny list for the case where a partner reports a leak, because the alternative is telling a partner to wait an hour after a credential is posted publicly.

    Two details that separate this from a passing answer: credentials carry a visible prefix so that automated secret scanners can recognise a leaked one, and each credential's last-used timestamp and calling network are visible to the partner, because the partner is the only party who can tell you that a call from a new location is not theirs.

**Faded example 2.** Last two blocks blanked. An internal platform serving 60 engineering teams, one shared datastore, and a new requirement that a compliance team must read every team's data while no team may read another's.

- Block 1: the principal is a human in a team, plus a small set of humans in a cross-cutting role.
- Blocks 2, 3 and 4: **you write these**.

??? note "Show answer"

    **Block 2, ladder.** Stay on shared tables. Sixty internal teams with no restore clause and no outsized consumer do not justify a schema per team, and a schema per team would make the compliance requirement worse, not better, because the cross-cutting read would then have to fan out over 60 schemas.

    **Block 3, enforcement.** Team comes from the identity provider group claim. The compliance role is not a team, so model it as an attribute on the principal that grants read across teams, and make that grant explicit in the policy rather than implicit in a superuser connection. Every cross-team read is logged with the principal, the object and the justification field, because an audit power with no audit trail is the thing an auditor will actually fail you on.

    **Block 4, the part most answers miss.** The compliance role is now the single most valuable credential in the system, so it gets the treatment the rest of the system does not need: short-lived tokens obtained through a step-up, time-boxed grants that expire without action, and an alert on volume rather than on access, because the failure mode is not one improper read, it is a bulk export. State the threshold: an alert if a compliance principal reads more objects in an hour than the historical daily maximum.

### 12f. Check yourself

**Q1.** Why should a request for a resource belonging to another tenant return 404 rather than 403, and when is that rule wrong?

??? note "Show answer"

    A 403 says "this exists and you may not have it", which confirms the existence of another tenant's object and, with sequential identifiers, lets a caller enumerate a competitor's volume. A 404 leaks nothing.

    The rule is wrong inside a single tenant, where a user hitting an object they lack permission for should get a 403 with an explanation, because there the information is not sensitive and the 404 sends a support engineer hunting for a deleted record that is sitting right there. So the rule is: 404 across the tenant boundary, 403 within it. Saying both halves is the answer, saying only the first is a recited rule.

**Q2.** You have 200,000 active sessions and are asked to cut revocation lag from 60 minutes to under 30 seconds. Give two designs and the cost of each, using the arithmetic from this module.

??? note "Show answer"

    Design one, shorten the access token to 30 seconds. Refresh load becomes 200,000 divided by 30, which is about 6,667 operations per second, each a write if rotation is on. That is a large, always-on dependency in front of every request, and it will be the thing that takes the product down. Reject it, but say the number rather than the instinct.

    Design two, keep 60 minute tokens and add a deny list checked at the edge. Size it: if you revoke on the order of a thousand tokens a day, entries live at most one token lifetime, so the working set is roughly a thousand times one twenty-fourth of a day, in the low tens of entries. It fits in memory on every edge node with room to spare. Cost is one lookup per request and a propagation delay of seconds.

    Design three, if the requirement applies only to money movement: leave reads alone and require a step-up token with a 60 second life for the sensitive operations. Cost is scoped to the traffic that needs it.

### 12g. When not to use this

Two components in this module are routinely over-applied. Each gets its own answer.

**Attribute-based policy engines.**

| Question | Answer |
|---|---|
| The measurement that justifies it | Count of permission rules that depend on data about the object or the requester, not just the requester's job function, and the rate at which new exception rules arrive |
| The threshold below which it is over-engineering | Fewer than roughly 10 rules, and no rule that references object attributes. A role table with 5 roles covers it |
| What to do instead | Roles in a table, checked in one function, with a test per role. Add attributes the first time a rule genuinely cannot be expressed as a role, and add them for that rule only |

**Database or schema per tenant.**

| Question | Answer |
|---|---|
| The measurement that justifies it | A written contractual or regulatory clause requiring isolated restore or separate encryption keys, or a single tenant above roughly 20 to 25 percent of a shared pool |
| The threshold below which it is over-engineering | Under about 50 tenants with no such clause and no outsized tenant. At that scale the migration tax dominates every benefit |
| What to do instead | One database, `tenant_id` on every table, row-level security, and the differential test that fails any query returning another tenant's rows |

The honest sentence to say in an interview: "I would stay on shared tables until a specific tenant or a specific clause forces me off, and I would rather spend that budget on making the tenant predicate structurally impossible to omit, because that is where the real incidents come from."
---

## Module 13. Rate limiting and quotas as a product surface

**Time: 22 to 28 minutes reading.**

A rate limit is not a defensive measure that happens to be visible. It is a published promise about how much of your system a client may have, and it is the mechanism through which a free tier, a paid tier and an abuse policy are all expressed. Candidates who treat it as an infrastructure detail lose the whole product half of the answer.

### 13a. Predict first

A limit specification and a client, side by side.

```
+--------------------------------------------------------------+
| POLICY:  1,000 requests per minute per API key               |
| ALGORITHM: fixed window, aligned to the wall clock minute    |
| ON REJECT: 429 with  Retry-After: 60                         |
|                                                              |
| CLIENT:  a nightly sync that wakes at :59.5 and fires its    |
|          whole backlog as fast as the connection allows      |
+--------------------------------------------------------------+
Caption: a fixed-window policy, its rejection response, and a
client whose traffic is timed close to the window boundary.
```

**Question.** What is the largest number of requests this policy can admit in any two-second interval, and what does the `Retry-After: 60` do to a thousand clients that are all rejected at the same moment?

??? note "Show answer"

    Two thousand. The window resets on the wall clock minute, so a client can spend its entire 1,000 in the last half second of one window and its entire next 1,000 in the first half second of the following window. Fixed windows admit up to twice the nominal rate across a boundary, which means the capacity you must actually provision is 2x the number you published. That factor is a property of the algorithm, not of the client's bad manners.

    The fixed `Retry-After: 60` is worse. Every client rejected inside the same window is told to return at the same instant, so the policy manufactures a synchronised retry wave. This is a thundering herd that you built yourself and then documented. The fix is a per-client reset time (the actual instant that client's window refills) plus a jitter instruction, or a value derived from position in a queue rather than a constant.

### 13b. Algorithms, priced on burst, memory and coordination

Six mechanisms. The two questions that separate them: what burst do they admit, and what does one node need to know about the others?

| Algorithm | Burst admitted | State per key | Coordination | Characteristic failure |
|---|---|---|---|---|
| Fixed window | Up to 2x the limit across a boundary | 1 counter | Cheap, one increment | Boundary doubling, plus synchronised resets |
| Sliding window log | Exactly the limit | 1 timestamp per request in the window | Cheap logically, expensive in memory | Memory blows up on high-rate keys |
| Sliding window counter | Close to the limit, within a few percent | 2 counters | Cheap | Approximation is wrong for very bursty traffic |
| Token bucket | The bucket size, on top of the refill rate | 2 numbers (tokens, last refill) | Cheap | A large bucket lets a quiet client hit you with a month of saved credit |
| Leaky bucket as a queue | None, output is smoothed | Queue plus a counter | Cheap | Adds latency, and the queue is a place requests go to expire |
| Concurrency cap | Not applicable, caps in-flight work | 1 counter | Needs release on every exit path | A leaked slot permanently shrinks the limit |

**Memory, derived.** Assume 1 million active keys and a policy of 1,000 requests per minute, both stated by the interviewer or assumed aloud.

- Sliding window log: up to 1,000 timestamps per key. At 16 bytes each that is 16 KB per key, so 16 GB for a million keys. That number alone eliminates the log at this scale.
- Sliding window counter: 2 counters at 8 bytes, so 16 bytes per key, 16 MB for a million keys. Fits in one node's memory with room for the key strings.

The elimination is the point. The log is not "expensive", it is 1,000 times the memory, and stating the ratio is what makes the choice a derivation rather than an opinion.

**Coordination, derived.** Suppose 30 service nodes and a published global limit of 1,000 per minute per key.

| Approach | What each node does | Error |
|---|---|---|
| Local limits, evenly divided | Allows 1,000 / 30 = 33 per minute | A client whose connections land on 3 nodes gets 100, not 1,000. Under-admission is the failure, and it is invisible to you and infuriating to them |
| Central counter | One increment on a shared store per request | Correct, but every request now depends on that store. At 50,000 requests per second that is 50,000 extra operations per second and a new hard dependency |
| Local buckets, periodic sync | Each node holds a share and reconciles every T | Overshoot is bounded by nodes times per-node rate times T |

Take the third row seriously, because it is what most large systems actually do. With a 200 ms sync interval, 30 nodes, and a per-node admission rate of 33 per minute during the drift window, the worst-case overshoot is 30 times 33 times 0.2 seconds divided by 60 seconds, which is about 3 requests. Overshooting a 1,000 per minute limit by 3 is free. That single calculation is how you justify approximate distributed limiting without hand waving.

### 13c. Dimensions, units and what the client is told

**Limit on the right unit.** Rate limiting per request is the default and it is usually wrong on a surface where requests differ in cost by orders of magnitude. If a client may fetch 1 item or 100 items per call, a per-call limit means your cost per unit of admitted work varies by 100x.

The repair is cost units. Publish a weight per operation, derived from measured work rather than invented:

| Operation | Measured work | Weight |
|---|---|---|
| Fetch one record by id | Baseline, 1 index lookup | 1 |
| List 100 records | About 20x the baseline in database time | 20 |
| Full-text search | About 50x the baseline | 50 |
| Bulk write of 100 items | About 100x, and it takes locks | 120 |

Then the published limit is "6,000 units per minute", and a client can spend it on 6,000 cheap reads or 120 searches. Two properties follow: your capacity planning is in one currency, and a client that gets more efficient is rewarded rather than punished for batching.

**Place limits at more than one level.** A single limit per key is the beginner shape. Real surfaces stack them, and each layer answers a different threat.

```
+---------------------------------------------------------+
| EDGE          per source address, per unauthenticated   |
|               route. Stops floods before they cost you  |
+----------------------------+----------------------------+
                             v
+---------------------------------------------------------+
| GATEWAY       per API key, in cost units. This is the   |
|               limit you publish and bill against        |
+----------------------------+----------------------------+
                             v
+---------------------------------------------------------+
| TENANT        per tenant across all its keys, so one    |
|               tenant cannot buy the system with 40 keys |
+----------------------------+----------------------------+
                             v
+---------------------------------------------------------+
| RESOURCE      concurrency cap per expensive backend,    |
|               shared by everyone. The last line         |
+---------------------------------------------------------+

Caption: four stacked limits, outermost first. Each layer
protects something the layer above cannot see. Only the gateway
layer is a product promise; the others are protection, and only
the gateway layer belongs in your published documentation.
```

**What the response must carry.** A 429 with no information is a bug. The client needs three facts to behave: what the limit was, how much is left, and when it refills. The IETF `httpapi` working group has been standardising `RateLimit` and `RateLimit-Policy` fields for exactly this; as of the date I checked, [the document is still an Internet-Draft, not an RFC](https://datatracker.ietf.org/doc/draft-ietf-httpapi-ratelimit-headers/) (checked 2026-08-01). Use the shape, do not claim the standard.

| Signal | Says | Why the client needs it |
|---|---|---|
| Limit and remaining | Your budget and what is left | Lets a client self-throttle before it is rejected, which is what you actually want |
| Reset instant, per client | When this client's budget refills | Prevents the synchronised return wave a constant `Retry-After` creates |
| Which policy was hit | Per key, per tenant or per operation | Otherwise a client optimises the wrong thing for a week |
| Whether the rejection is billable | Quota exhaustion versus overload | Decides whether the fix is upgrading a plan or backing off |

**Three status codes, three different meanings.** Confusing them is one of the fastest ways to give a client the wrong instruction.

- 429: you exceeded a rate or quota that applies to you. Retry after the stated instant. The system is healthy.
- 503: the system is shedding load. Retry with backoff and jitter, and expect the wait to be unrelated to your own usage.
- 402: you are out of purchased quota. Retrying will never work. A human must act.

### 13d. Worked example: quotas for a search surface with a real bill

The prompt: a search endpoint is being opened to external developers. Design the limits and the quota model.

Stated inputs, all of which drive a number later:

| Input | Value | Tag |
|---|---|---|
| Search cost, measured against a simple read | 50x | GIVEN, from profiling |
| Infrastructure cost of the search fleet | 12,000 per month | GIVEN, from the cost dashboard |
| Search fleet capacity at target latency | 400 searches per second | GIVEN, load test |
| Expected developers in year one | 2,000, with the usual heavy tail | ASSUMED, reasonable because it matches the account growth of the existing read-only surface |

**Block 1. Purpose: choose the unit before choosing the algorithm.**

I refuse to publish a limit in requests, because a search costs 50 times a read and both live on this surface. I publish cost units, with search weighted 50 and a plain read weighted 1. Now one number governs both.

The error most readers make here is to design the algorithm first. Algorithm choice is nearly free to change later; the published unit is a contract that is expensive to change, because it appears in every customer's plan and every invoice. Decide the thing that is hard to reverse first.

!!! question "Say it before you read on"

    Before the next block: using the 400 searches per second capacity figure, say what the total unit budget per second is for the whole fleet, and why that number is the ceiling on everything the plan tiers can sum to.

**Block 2. Purpose: derive the tier sizes from capacity, not from a pricing meeting.**

400 searches per second at weight 50 is 20,000 units per second of fleet capacity, which is 1.2 million units per minute. I hold back 30 percent for headroom, spikes and internal traffic, leaving roughly 840,000 units per minute to sell.

If the free tier is 600 units per minute (12 searches a minute, enough to build and demo something), then 2,000 free developers would commit 1.2 million units per minute, which exceeds the entire sellable budget. That kills a naive free tier immediately.

Two repairs, and I would say both aloud. Either the free tier is smaller, say 100 units per minute, so 2,000 developers commit 200,000 units per minute against 840,000 available, or the free tier is oversubscribed deliberately, on the observation that committed limits are not consumed limits. I would take oversubscription with an explicit ratio and a measured backstop, because the alternative makes the free tier useless for evaluation, and an unusable free tier costs conversions.

The measured backstop is the fourth layer in the stack above: a concurrency cap on the search fleet that sheds when actual load approaches capacity, independent of anybody's quota. Quotas are a promise about typical behaviour, and the concurrency cap is what makes the promise safe to oversubscribe.

**Block 3. Purpose: choose the algorithm now that the numbers constrain it.**

Sliding window counter at the gateway, per key, in units. The log is out on memory grounds from the earlier derivation.

Token bucket is tempting, and I reject a large bucket: a bucket of 10 minutes of savings lets a quiet key spend 10 minutes of fleet capacity in one burst, and with a 30 percent headroom margin that is not survivable if a handful of keys do it at once. I allow a small bucket, at most 2x the per-second rate, so an interactive client's natural burstiness is not punished.

Coordination is approximate with a 200 ms sync, justified by the overshoot arithmetic from the previous section: bounded overshoot of a few units against a budget of hundreds of thousands is not worth a central counter on the hot path.

!!! question "Say it before you read on"

    Before the next block: say why the concurrency cap chosen in block 2 is the thing that makes oversubscription safe, rather than the sliding window just chosen in block 3.

**Block 4. Purpose: connect the quota to the bill and to fairness.**

Cost per unit falls out of the given numbers: 12,000 per month for the fleet, and 840,000 sellable units per minute is about 36 billion units per month if fully consumed. At 10 percent average utilisation, which I assume because sellable capacity is a peak figure and average traffic runs far below peak, the realistic consumption is 3.6 billion units, so the infrastructure floor is about 0.33 per million units.

Whatever the price is, it must clear that floor plus support and margin, and the fact that a candidate can produce a floor at all is the signal.

Fairness gets its own mechanism rather than being an emergent property. Per-tenant concurrency caps, set so no single tenant can hold more than a stated share of in-flight search slots, are the cheapest fairness device available: they cost one counter and they bound the damage a single well-behaved-but-large customer does to everyone else's latency.

The error most readers make at this step: reporting quota consumption only when it is exhausted. A developer needs the running total in every response, because the support ticket you are trying to prevent is "my job died at 3am and I do not know why".

### 13e. Fade the scaffold

**Faded example 1.** Last block blanked. A write-heavy webhook ingestion surface accepts events from 5,000 partner systems. Inputs: ingestion capacity 8,000 events per second, each event costs the same, partners are machines and retry automatically on any failure.

- Block 1, unit: events, because all events cost the same, so weights would be ceremony.
- Block 2, tier sizes: 8,000 per second with 30 percent headroom leaves 5,600 sellable, so a flat 1 event per second per partner commits 5,000 against 5,600 and is defensible without oversubscription.
- Block 3, algorithm: token bucket with a 60 event bucket, because partner systems batch naturally and a smoothing queue would add latency to an ingestion path that has none to spare.
- Block 4, what the rejection tells an automatic retrier: **you write this**.

??? note "Show answer"

    The clients are machines that retry automatically, so the rejection response is the only control input you have over aggregate load, and a constant retry hint would synchronise 5,000 partners.

    Return 429 with a per-partner reset instant computed from that partner's own bucket refill, not from a wall clock boundary. Add an explicit instruction to apply full jitter, and state in the documentation that the client should treat the reset instant as the earliest time, not the correct time.

    Then the part that separates a good answer: for an ingestion surface, rejecting is usually worse than accepting and delaying, because the partner's alternative to your queue is its own queue, which is less reliable and invisible to you. So consider accepting with a 202 and a stated processing delay while the partner is within a burst allowance, and reserve 429 for sustained overage. State the difference in the documentation: a 202 means we hold it, a 429 means you hold it.

    Second detail: never return 503 here for quota reasons. A machine client's backoff for 503 assumes your outage will end on its own, and it will escalate to a human far later than it should if the real cause is that the partner is over its plan.

**Faded example 2.** Last two blocks blanked. An internal platform endpoint used by 60 teams, one shared database, no billing relationship at all.

- Block 1, unit: database time in milliseconds, measured per call, because the teams' calls vary by 200x and there is no product tier to simplify it.
- Blocks 2, 3 and 4: **you write these**.

??? note "Show answer"

    **Block 2, sizes.** There is no revenue constraint, so the budget is the database's capacity and the goal is isolation, not monetisation. Divide by criticality, not evenly: interactive request paths get a guaranteed share, batch jobs get whatever is left and are the first shed. Publish the shares, because an internal quota that nobody can see becomes a mystery outage.

    **Block 3, algorithm.** A concurrency cap per team on the database connection pool, not a rate limit. This is the key move: with a shared database, the resource that runs out is in-flight work, and a rate limit in requests per minute does not bound in-flight work at all when one team's queries get 10 times slower. A cap of, say, 4 concurrent connections per team on a 200 connection pool means no team can consume more than 2 percent of the pool, whatever its queries do.

    **Block 4, the product framing even without a bill.** The internal equivalent of a bill is a chargeback report: show each team its consumed database milliseconds weekly, alongside the platform's total cost. Teams optimise what they can see. Also publish the shedding order, so a team knows in advance whether it is in the interactive class or the batch class, because discovering your class during an incident is how platform teams lose trust.

### 13f. Check yourself

**Q1.** Your published limit is 1,000 requests per minute per key, enforced by a fixed window. Capacity planning is done against 1,000. State the bug and the two ways to fix it.

??? note "Show answer"

    The bug: a fixed window admits up to 2x the nominal rate across the boundary, so a key can present 2,000 requests within a couple of seconds and your fleet must survive it. Planning against 1,000 under-provisions by a factor of two for any client that batches near a minute boundary, and clock-aligned windows encourage exactly that.

    Fix one, change the algorithm: a sliding window counter removes the boundary doubling for two counters of memory per key. Fix two, keep the algorithm and plan against 2x, while adding a concurrency cap so the burst is bounded in in-flight terms even if the rate check admits it. The second is legitimate and cheap; what is not legitimate is publishing 1,000, provisioning 1,000, and calling the resulting incident a client problem.

**Q2.** A client asks why it received 429 when its dashboard shows it used only 40 percent of its plan. Give two explanations that are both your fault, not theirs.

??? note "Show answer"

    First, it hit a different layer. The plan quota is the gateway limit, but the request may have been rejected by a per-tenant limit shared with the customer's other keys, or by an edge limit on the source address, or by a per-operation limit that is not shown on the plan page. If the 429 did not say which policy was hit, the client cannot diagnose it and neither can your support team.

    Second, the dashboard is aggregated over a longer window than the limit. A limit of 1,000 per minute displayed as a monthly percentage will always look underused; a client bursting to 1,000 in ten seconds is at 40 percent of the month and 600 percent of the minute. The repair is to show the client the same window the enforcement uses, and to return remaining quota on every response so the two can never disagree.

### 13g. When not to use this

**Distributed exact rate limiting.**

| Question | Answer |
|---|---|
| The measurement that justifies it | The cost of admitting more than the limit. Compute it as overshoot percentage times the marginal cost of the admitted work, per minute |
| The threshold below which it is over-engineering | If bounded overshoot of roughly 10 to 20 percent costs less than the added latency and availability risk of a shared counter on every request, exact limiting is not worth buying. On most surfaces it is not close |
| What to do instead | Local buckets with periodic sync, sized by the overshoot arithmetic in 13b, plus a concurrency cap as the real backstop |

**Rate limiting at all.**

| Question | Answer |
|---|---|
| The measurement that justifies it | Peak observed request rate from a single caller against the capacity of the smallest downstream resource |
| The threshold below which it is over-engineering | An internal surface with a handful of known callers, all under your operational control, whose combined peak is well under capacity. There, a limit mostly generates incidents in which the limit is the cause |
| What to do instead | A concurrency cap on the expensive dependency and an alert on caller-attributed load. Add rate limits when a caller you cannot phone appears |

The line worth saying in an interview: a rate limit protects you from clients, and a concurrency cap protects you from yourself. If you can only have one on an internal surface, take the cap.

---

## Module 14. Long-running operations

**Time: 20 to 25 minutes reading.**

### 14a. Predict first

```
+--------------------------------------------------------------+
| POST /exports                                                |
|   builds a CSV of a tenant's last 24 months of invoices      |
|   returns 200 with the file body                             |
|                                                              |
| MEASURED:  p50  8 s     p95  110 s     p99  240 s            |
| PATH:      client -> CDN -> load balancer -> service         |
| CLIENT:    an SDK with a 60 s default timeout and 3 retries  |
+--------------------------------------------------------------+
Caption: a synchronous export endpoint with its measured latency
distribution, the hops in front of it and the client's timeout
and retry settings.
```

**Question.** Name three independent components that will terminate this request before it finishes, and say what the retry configuration does to the database at p99.

??? note "Show answer"

    Three terminators, any of which fires first: the client's own 60 second timeout, the load balancer's idle timeout (commonly 60 seconds by default in managed load balancers, and worth stating as an assumption rather than a fact unless you can name the configuration), and the content delivery layer or reverse proxy in front, which typically enforces its own response timeout. Any middlebox in the path can end the request, and you control none of them from inside the handler.

    The retry behaviour is the expensive part. At p99 the export takes 240 seconds and the client gives up at 60, retrying three times. The server never learns that the client left, so four full exports run concurrently for one user request, and the fourth starts while the first is still going.

    The database sees 4x the intended load precisely for the heaviest queries, which is a self-amplifying failure: the slower the exports get, the more retries pile on. This is the retry storm class from the hub's failure library, and here it is guaranteed rather than possible.

    The subtle third answer: even if a retry did succeed, the client has no way to tell whether it got a fresh export or a duplicate of work already done, because a synchronous endpoint has no identity for the work.

### 14b. Give the work an identity

The move is to stop returning the result and start returning a resource that represents the work. Everything else follows from that one decision.

| Piece | Shape | Why it exists |
|---|---|---|
| Submission | `POST /exports` returns 202 with the operation's location | The request finishes in milliseconds, so no middlebox can kill it |
| Operation resource | `GET /exports/{id}` returns state, progress, timing and either a result reference or an error object | The work now has an identity, so retries are cheap reads instead of duplicated work |
| Result reference | A separate URL, ideally time limited | Lets the result be served from storage rather than through your application |
| Cancellation | A state transition on the operation, not a delete | Deleting the record loses the audit of what was cancelled and by whom |

**The state machine, and the rule that makes it usable.**

```
   +---------+      +---------+      +-----------+
   | PENDING | ---> | RUNNING | ---> | SUCCEEDED |
   +----+----+      +----+----+      +-----------+
        |                |
        |                +---------> +-----------+
        |                |           |  FAILED   |
        v                v           +-----------+
   +-----------+    +-----------+
   | CANCELLED |    | CANCELLED |    +-----------+
   +-----------+    +-----------+    |  EXPIRED  |
                                     +-----------+

Caption: five states. PENDING and RUNNING are the only states a
client should poll from. SUCCEEDED, FAILED and CANCELLED are
terminal and never change. EXPIRED is reached from SUCCEEDED
when the result is deleted after its retention window, and it is
a separate state so a client can tell "gone" from "never was".
```

The rule: terminal states are immutable. A client that has read SUCCEEDED must never see that operation become FAILED. Without that rule, clients cannot cache the outcome and must poll forever, and reconciliation logic on the client becomes impossible to write correctly.

**Progress that does not lie.** Two options, and one of them is nearly always wrong.

- A percentage. It must be monotonic. A percentage that goes backwards, or that sits at 99 for four minutes, teaches clients to ignore progress entirely.
- Completed and total counts. Better, because the client can compute its own estimate and can display something honest when total is unknown (report completed only, and say total is null rather than guessing).

### 14c. Polling, cancellation and retention, each derived from a number

**Polling cost, derived.** Assume 10,000 export jobs per hour and a mean duration of 5 minutes, both stated by the interviewer or assumed aloud.

| Poll interval | Polls per job | Polls per hour | Sustained rate |
|---|---|---|---|
| 1 second | 300 | 3,000,000 | 833 per second |
| 5 seconds | 60 | 600,000 | 167 per second |
| Adaptive: 1 s, doubling to 30 s | About 13 | 130,000 | 36 per second |

833 requests per second of pure polling for 10,000 useful jobs is a 300:1 waste ratio, and it is entirely under your control through one response field. Return a suggested next-poll delay on every operation read and have the SDK honour it: start at 1 second, double to a 30 second cap, and reset on any state change. That single field cuts the load by more than an order of magnitude, and it is the cheapest thing on this page.

**Cancellation has a guarantee, and you must state which one.**

| Guarantee | What the response means | When to offer it |
|---|---|---|
| Accepted | We recorded the request, the work may still finish | Default, and honest for distributed work |
| Effective | The work is stopped and no side effects will occur after this point | Only when workers check a cancellation flag between every side effect |
| Compensated | The work is stopped and completed effects are reversed | Only when every step has an inverse, which is rare and expensive |

Saying "cancel returns 202 accepted, and the operation moves to CANCELLED only when a worker acknowledges it" is a small sentence that reads as someone who has operated one of these.

**Result retention, derived.** Assume 10,000 exports per day at an average 4 MB, and assume 95 percent of results are downloaded within one hour, which is reasonable because an export is usually triggered by a person who is waiting for it.

- 24 hour retention: 10,000 times 4 MB is 40 GB of live results at any time. Trivial.
- 30 day retention: 1.2 TB, plus the obligation to keep them consistent with deletions and access control changes over that period.

The 24 hour window covers the 95 percent case with a large margin and turns "where is my export from last month" into a one-line answer: re-run it. State the window in the response as an expiry instant, so the client does not have to remember your policy.

### 14d. Worked example: an export operation for a reporting surface

The prompt: reports currently time out. Make them work.

**Block 1. Purpose: establish that the work needs an identity before designing anything else.**

I check one number first: p99 duration against the smallest timeout in the path. Here p99 is 240 seconds and the smallest timeout is 60. The gap is not closable by optimisation of the kind you can promise in an interview, so the operation becomes asynchronous. If p99 had been 4 seconds against a 60 second timeout, I would say out loud that this whole module is unnecessary and keep it synchronous.

The error most readers make: reaching for a job queue because the word "export" appeared. The trigger is the ratio of p99 to the smallest infrastructure timeout, and saying that ratio out loud is what distinguishes a decision from a reflex.

**Block 2. Purpose: make submission safe to retry, which is the whole reason clients duplicated work before.**

`POST /exports` carries an idempotency key. Same key inside the retention window returns the same operation resource, with the same identifier, rather than starting a second export. Now the client's automatic retry is free: it converges on one operation instead of forking four.

I also add request fingerprinting, because the key alone is not enough: if the same key arrives with different parameters, that is a client bug and it must be rejected loudly rather than silently served the first result. The full treatment of keys, fingerprints and the transaction boundary is Module 9's job; here I am only inheriting it.

!!! question "Say it before you read on"

    Before the next block: say why the idempotency key has to be on the submission and not on the result download, in one sentence.

**Block 3. Purpose: bound polling cost and make progress honest.**

Every read of the operation returns state, completed and total row counts, and a suggested delay. The delay starts at 1 second, doubles to a 30 second cap, and resets to 1 second whenever the state changes, so a client sees a fast transition into RUNNING and then relaxes.

I put a hard cap on operation lifetime too: any operation still RUNNING after 30 minutes moves to FAILED with a specific code, because an operation with no upper bound is a resource leak that nobody notices until the worker pool is full of them.

!!! question "Say it before you read on"

    Before the next block: the result is a file sitting in storage after the operation succeeds. Name one thing that can change between success and download that the stored file does not know about.

**Block 4. Purpose: decide what happens after success, including the two failure paths people forget.**

On success the operation carries a result URL valid for the retention window, plus the expiry instant. Downloads are served from object storage, so a 4 MB file never crosses the application tier.

Two forgotten paths, and naming them is where this answer stops sounding like a tutorial:

1. Authorisation changes between submission and download. The result was computed with the submitter's permissions; if those permissions are revoked an hour later, the pre-signed URL still works. So the download goes through an authorisation check at fetch time, or the URL is bound to a short life measured in minutes rather than hours, and the trade-off is stated.
2. The tenant deletes data that the export contains. Now a file in storage outlives a deletion request, which is a compliance problem rather than an engineering one. The retention window has to be short enough that the deletion pipeline can treat exports as a known location, and exports must be enumerable per tenant so they can be swept.

### 14e. Fade the scaffold

**Faded example 1.** Last block blanked. A video transcoding surface: submissions take 2 to 40 minutes, clients are server-side integrations, and results are large.

- Block 1: p99 far exceeds any timeout, so an operation resource is not optional.
- Block 2: submission carries an idempotency key derived from the source asset and the profile, so a retried submission joins the existing job.
- Block 3: progress reported as completed and total segments, with a suggested poll delay, and a hard 90 minute lifetime cap.
- Block 4, how clients learn about completion without polling at all: **you write this**.

??? note "Show answer"

    Because the clients are server-side integrations with public endpoints, offer a callback in addition to polling, and treat it as an optimisation rather than a replacement. The submission accepts a callback URL; on a terminal state, deliver a signed notification carrying the operation identifier and the terminal state, but not the result itself.

    Three details that make this answer more than "add a webhook".

    1. The notification is a hint, not a guarantee. The operation resource stays the source of truth, so a client that misses a callback still converges by polling at a slow interval.
    2. Delivery gets the full retry and signing treatment of Module 15, because a callback lost silently is worse than no callback at all: clients stop polling once you offer one.
    3. Never put the result payload in the callback. It makes deliveries large, retries expensive, and the signature meaningful over data the client may already have.

**Faded example 2.** Last two blocks blanked. A bulk permission change across a large tenant's document tree: 1 to 20 minutes, must be cancellable, partial application is visible to users while it runs.

- Block 1: asynchronous, and unusually the intermediate state is user visible, so progress is a product feature rather than a diagnostic.
- Blocks 2, 3 and 4: **you write these**.

??? note "Show answer"

    **Block 2, submission.** Idempotency key plus a precondition: the operation carries the version of the tree it was computed against, and submission fails if the tree changed. Otherwise two overlapping permission changes interleave and the final state belongs to neither request. This is optimistic concurrency from Module 10, applied to an operation rather than to a resource.

    **Block 3, progress and cancellation.** Progress is completed and total nodes, and because partial application is visible, the client is told explicitly that the change is being applied progressively and is not atomic. Cancellation is offered with the accepted guarantee, and the operation records where it stopped. The important sentence: cancellation here does not undo, it stops, so the response must say that in words, and the operation must expose the boundary so an operator can decide whether to roll forward or write a compensating change.

    **Block 4, retention and audit.** The operation record is not a temporary artefact here, it is an audit record: who changed what permissions, when, over which subtree, with what outcome. So the retention window is governed by the audit requirement (months) rather than by download behaviour (hours), which is the opposite of the export case. Say that difference out loud: retention windows come from the consumer of the record, and the consumer here is a compliance reviewer, not an impatient user.

### 14f. Check yourself

**Q1.** A client polls your operation resource every second for a 10 minute job and complains about being rate limited. Whose bug is it, and what is the one-field fix?

??? note "Show answer"

    Yours. A polling client has no way to know a good interval unless you tell it, and the default any developer writes is a tight loop with a constant sleep. The fix is a suggested delay field on every operation read, honoured by the SDK, starting around a second, doubling to a cap of about 30 seconds, and resetting on state change.

    The second half of the answer: exempt operation reads from the main quota or price them at a much lower weight, because charging a client for the polling your design forces on it converts your architecture choice into their bill. If you do want polling to count, then you owe them a push option.

**Q2.** Your operation resource sometimes moves from SUCCEEDED back to FAILED when a downstream check runs late. Name two client behaviours this breaks.

??? note "Show answer"

    First, caching and stop-polling logic. A client that reads a terminal state records the outcome and stops polling, by design, so it will never observe the later change and will act on a result that your system no longer considers valid. The divergence is silent and permanent.

    Second, downstream side effects. A client that saw SUCCEEDED may have already emailed a user, released an order, or triggered its own workflow. There is no compensation path because the client believed it was reading an immutable fact.

    The repair is not to make the transition faster. It is to keep terminal states immutable and represent late failures as a new operation or a distinct field, for example a verification state alongside the terminal state, so the client can be told there are two facts rather than one fact that changed.

### 14g. When not to use this

| Question | Answer |
|---|---|
| The measurement that justifies it | The ratio of the operation's p99 duration to the smallest timeout anywhere in the request path, including the client's own default, plus the p99.9 tail |
| The threshold below which it is over-engineering | If p99 is under roughly 30 percent of the smallest timeout and the tail is bounded, stay synchronous. An operation resource adds a state machine, a poller, a worker pool, a result store and a retention policy, all of which must be operated |
| What to do instead | Keep it synchronous with a hard server-side deadline that is shorter than every timeout in the path, return a specific error with a retry hint when the deadline is hit, and stream the response if the size rather than the compute is the problem |

Two boundary cases worth naming, because they are where the simple rule stops applying:

- Duration is fine but the result is huge. That is a streaming problem, not an asynchronous one. A streamed response keeps the request short-lived at the byte level and avoids the entire job machinery.
- Duration is fine at p99 but the work is expensive and non-idempotent. Then you may want an operation resource purely for its identity, so that retries converge, even though the latency never required it. Say that reason explicitly, because it is a different justification and interviewers notice when the justification matches the design.
---

## Module 15. Webhooks, event delivery and consistency across a service boundary

**Time: 28 to 35 minutes reading.**

An outbound event is the only part of your interface where you are the client and someone else's availability is your problem. It is also where the two hardest words in this course meet: a delivery guarantee is a distributed systems claim, and you are making it to strangers in writing.

### 15a. Predict first

```
+--------------------------------------------------------------+
| def capture_payment(req):                                    |
|     with db.transaction():                                   |
|         p = payments.insert(state='captured', ...)           |
|                                                              |
|     for attempt in range(3):                                 |
|         r = http.post(merchant_url, body=event(p))           |
|         if r.ok: break                                       |
|         sleep(2 ** attempt)                                  |
|     return 200                                               |
+--------------------------------------------------------------+
Caption: a handler that commits a payment and then delivers the
event inline, retrying three times inside the request.
```

**Question.** Name the two distinct consistency failures this code can produce, and say which of them is fixed by increasing the retry count from 3 to 30.

??? note "Show answer"

    Failure one, the lost event. The transaction commits, then the process is killed: a deploy, an out-of-memory kill, a machine going away. The payment exists and the event never will, because nothing durable ever recorded that the event was owed. No retry count fixes this, because the retry loop dies with the process. This is the half that candidates almost never name.

    Failure two, the phantom event. Reorder the code slightly, as many real handlers do, and deliver before the commit, or commit later in the same handler: now the merchant is told about a payment that gets rolled back. The merchant has released goods against a payment that does not exist in your ledger.

    Increasing retries from 3 to 30 fixes neither. It addresses only the case where the merchant is briefly unavailable and your process stays alive throughout, which is the least interesting failure of the three. Worse, it makes things actively harder: 30 in-request retries hold a worker for minutes, so a single slow merchant consumes your capacity, and now merchant availability is coupled to your ability to accept payments at all.

    The repair is to record the obligation in the same transaction as the state change, and let a separate process do the delivering. That is the outbox, and it is the whole content of the next section.

### 15b. The dual write, the outbox, and the guarantee you publish

**The dual write problem, stated exactly.** You have two systems to update: your database and someone else's endpoint. There is no transaction across both. Four interleavings exist and two of them are wrong.

| Order | Outcome | Client sees |
|---|---|---|
| Commit, then deliver, both succeed | Correct | Correct |
| Commit succeeds, delivery lost (crash or permanent failure) | Missing event | State changed and nobody was told, forever |
| Deliver succeeds, commit rolls back | Phantom event | Told about a thing that never happened |
| Neither happens | Correct | Nothing happened, nothing said |

**The outbox, in three sentences.** In the same transaction that changes state, insert a row into an outbox table describing the event. A separate relay process reads unsent outbox rows, delivers them and marks them sent. Because the state change and the obligation commit atomically, the missing-event and phantom-event rows above become impossible.

What you buy and what you pay:

| Property | Result |
|---|---|
| Delivery guarantee | At least once. The relay can crash after delivering and before marking sent, so duplicates are inherent |
| Ordering | Per aggregate, if the relay reads in insertion order per key. Never global, and never promise global |
| Added latency | The relay poll interval plus delivery time |
| Added failure mode | A stalled relay silently accumulates unsent rows, which is why relay lag is a first-class alert |

**Relay lag, derived.** Assume the relay polls every 200 ms and delivers in batches of 100 per destination. At 10,000 events per hour, that is 2.8 events per second, so a poll finds around 0.6 events and lag is dominated by the poll interval: typically under half a second.

Now assume a burst of 50,000 events in a minute, which is 833 per second: the relay must sustain 833 deliveries per second or the backlog grows, and the lag becomes backlog divided by drain rate. Publish the steady-state number, alert on the derived one.

```
+--------------------+        +---------------------+
| WRITE PATH         |        | RELAY               |
| one transaction:   |  ==>   | reads unsent rows   |
| state change  +    |        | in key order, marks |
| outbox row         |        | sent after ack      |
+--------------------+        +----------+----------+
                                         |
                     +-------------------+------------------+
                     v                                      v
        +------------------------+            +------------------------+
        | PER DESTINATION QUEUE  |            | ATTEMPT LOG            |
        | own retry schedule and |            | every try: time, code, |
        | own concurrency cap    |            | body excerpt, next try |
        +-----------+------------+            +------------------------+
                    |
        +-----------+-----------+
        v                       v
+----------------+   +------------------------+
| CONSUMER       |   | DEAD LETTER            |
| verifies sig,  |   | after the ladder ends, |
| dedupes by id  |   | replayable by the user |
+----------------+   +------------------------+

Caption: the write path commits state and obligation together;
the relay fans obligations into a queue per destination so one
slow consumer cannot starve the others; every attempt is logged
where the consumer can read it; exhausted deliveries land in a
dead letter store that supports replay.
```

**One queue per destination is the structural decision.** With a single shared queue, one merchant that answers in 30 seconds occupies workers that every other merchant needs, and your p99 delivery latency becomes the worst consumer's latency. A queue and a concurrency cap per destination bounds the blast radius of a slow consumer to that consumer.

**The guarantee you publish, in the words a client can act on.** Write these four lines into your documentation, because each one tells a consumer what code to write.

- You will receive every event at least once. Deduplicate on the event identifier, which is stable across retries.
- You may receive an event more than once, including much later. Make your handler idempotent.
- Events for the same object arrive in order under normal operation, and may arrive out of order after a retry. Every event carries a version, so discard any event whose version is older than what you have stored.
- Return 2xx within 5 seconds. Anything else, including a timeout, is a failure and will be retried on the published schedule.

That last line is the one candidates omit, and it is the one that decides consumer architecture: told to answer in 5 seconds, a competent consumer enqueues and returns, rather than processing inline.

### 15c. Retries, signing, and the surface a consumer debugs with

**Deriving a retry ladder rather than picking one.** Consumer failures come in two clusters, and the ladder has to cover both.

| Cluster | Typical duration | What the ladder needs |
|---|---|---|
| Transient: a deploy restart, a brief network fault, a full worker pool | Seconds to a minute | Several attempts inside the first minute |
| Structural: a bad release, an expired certificate, a misconfigured firewall, a person on holiday | Hours to a business day | Sparse attempts stretching past 24 hours |

Cover both with exponential growth. To reach 24 hours from a 15 second first retry with a growth factor of 3, the number of steps is the logarithm base 3 of (86,400 divided by 15), which is about 7.9, so 8 to 10 attempts. A concrete ladder that satisfies both clusters:

| Attempt | Delay after previous | Cumulative age of the event |
|---|---|---|
| 1 | 0 | 0 |
| 2 | 15 s | 15 s |
| 3 | 1 min | 1 min 15 s |
| 4 | 5 min | 6 min 15 s |
| 5 | 15 min | 21 min |
| 6 | 1 h | 1 h 21 min |
| 7 | 3 h | 4 h 21 min |
| 8 | 6 h | 10 h 21 min |
| 9 | 12 h | 22 h 21 min |
| 10 | 24 h | 46 h 21 min |

Ten attempts, a horizon of just over 46 hours, which covers an outage that starts on a Friday evening and is fixed on a Monday morning only if someone is working Saturday. State that limitation out loud rather than pretending the ladder covers everything, and pair it with dead lettering plus replay so a Monday fix is still recoverable.

Add full jitter to every delay. Without it, a consumer that fails at 09:00 gets all of its pending events re-presented at exactly the same offsets, which reproduces the load spike that may have caused the failure.

**Draining a backlog without re-breaking the consumer.** Assume an endpoint receives 10,000 events per hour in steady state, which is 2.8 per second, and it is down for 6 hours. The backlog is 60,000 events.

- Release at your own maximum, say 500 per second: the consumer receives 180 times its normal load and falls over again, converting a 6 hour outage into a repeating one.
- Release at 3 times steady state, 8.3 per second: the backlog drains in 60,000 divided by 8.3, which is about 2 hours, while the consumer handles a load it can plausibly survive.

So the per-destination queue needs a drain rate cap expressed as a multiple of that destination's observed steady rate, not as a global constant. This is the single most valuable operational detail in the module, and almost nobody says it.

**Signing, and the three details that matter.**

| Detail | Rule | Why |
|---|---|---|
| What is signed | A message authentication code over the timestamp joined to the exact raw request body | Signing parsed and re-serialised JSON fails as soon as key order or number formatting differs anywhere in either stack |
| Replay protection | The timestamp is inside the signed material, and the consumer rejects anything older than about 5 minutes | Without a signed timestamp, a captured request can be replayed forever. Five minutes is comfortably above normal clock skew between machines synchronised to network time and network delay, while keeping the replay window short |
| Rotation | Two active signing secrets at once, both published, with the new one used for signing and both accepted for verification during the overlap | Otherwise rotation is a synchronised cutover with every consumer, which is the same problem as an unversioned breaking change |

Send the signature and the timestamp as separate fields, and document the exact string that was signed, byte for byte. A consumer that cannot reproduce your canonical string cannot verify anything, and this is the most common integration support ticket on any webhook product.

**Ordering, honestly.** Global ordering across destinations is not achievable at reasonable cost and is not worth promising. Per-object ordering under normal operation is achievable and is what consumers actually need. Ship both halves: order events per object key in the relay, and put a monotonically increasing version on every event so a consumer that receives them out of order can drop the stale one. The version is what makes the weak guarantee usable.

**The debugging surface, which is half the product.** A webhook system without one generates support tickets forever.

| Surface | Contents |
|---|---|
| Attempt log per event | Every attempt: timestamp, response status, latency, first few hundred bytes of the response body, and the next scheduled attempt |
| Replay control | Redeliver one event, or every failed event in a time range, on demand |
| Endpoint health | Success rate and latency per destination over the last day, visible to the consumer, not just to you |
| Test delivery | Send a synthetic event of any type on demand, so an integrator can develop without producing real payments |
| Automatic disable rule | State it in advance: for example, disable after 24 hours of unbroken failure with at least 100 attempts, notify the technical contact, and require an explicit re-enable |

The disable rule must be published, because a consumer that comes back after a two-day outage and finds its endpoint switched off with no notice will treat that as your bug, and they will be half right.

### 15d. Worked example: payment events to 800 merchant endpoints

The prompt: our payments product needs to tell merchants when a payment is captured, refunded or disputed. Design it.

Stated inputs:

| Input | Value | Tag |
|---|---|---|
| Merchant endpoints | 800 | GIVEN |
| Events per hour, all merchants | 90,000, peaking at 4x on sale days | GIVEN |
| Share of merchants using a hosted platform rather than their own code | 35 percent | GIVEN |
| Merchant endpoints that answer in over 5 seconds at p99 | About 6 percent | GIVEN, from a pilot |

**Block 1. Purpose: write the guarantee before writing any mechanism.**

I start with the sentence I will publish, because it constrains every later choice: at least once, per-payment ordering under normal operation, versioned so stale events can be discarded, 5 second response deadline, and a published retry ladder with dead lettering.

I choose at least once rather than at most once because a missed capture event means a merchant does not ship an order, which is a customer-visible failure, whereas a duplicate is something the merchant can deduplicate with an identifier I supply. That trade is the whole reason the guarantee is what it is, and stating it as a trade rather than as a default is the difference between a senior and a mid answer.

The error most readers make here: promising exactly once. There is no exactly once across a network boundary you do not control. What exists is at least once plus consumer-side deduplication, and calling that "effectively once" is fine as long as you say where the deduplication happens.

!!! question "Say it before you read on"

    Before the next block: given the 5 second deadline I just published, say what architecture I am implicitly requiring of the 800 merchants, and why 6 percent of them are a problem.

**Block 2. Purpose: make the event owed at the moment the state changes.**

The payment state change and an outbox row commit in one transaction. The relay is separate, reads unsent rows in payment-identifier order, and marks sent only after a 2xx.

Two consequences I say out loud. First, duplicates are now inherent, because the relay can deliver and die before marking sent, which is exactly why the published guarantee is at least once. Second, relay lag is now a service level indicator with an alert on it, because a stalled relay is silent: payments keep working, merchants keep receiving nothing, and the first symptom is a support ticket hours later.

**Block 3. Purpose: size the delivery fleet and stop one slow merchant from taxing everyone.**

90,000 events per hour is 25 per second, and the 4x sale-day peak is 100 per second. One queue and a concurrency cap per merchant, so the 6 percent of merchants who answer slowly consume only their own capacity.

Sizing with the slow ones as the constraint: if a slow merchant takes 5 seconds and I allow 4 concurrent deliveries to it, that merchant absorbs 4 workers for its whole backlog. 48 slow merchants (6 percent of 800) times 4 is 192 workers dedicated to slow endpoints. That number is why per-destination caps exist: without them, those 192 worker-slots would be taken from the shared pool at the worst moment, on a sale day, when the fast 94 percent need them most.

Drain caps are set per merchant at 3 times that merchant's observed steady rate, from the backlog arithmetic earlier.

!!! question "Say it before you read on"

    Before the next block: given that the guarantee in block 1 is at least once, say what the payload must carry for a merchant to be able to obey it, and what it must not carry.

**Block 4. Purpose: give the consumer everything it needs to be correct without asking me.**

The payload carries: event identifier (stable across retries, and the deduplication key), event type, a monotonically increasing version per payment, the payment identifier, the timestamp, and a minimal body. Not the full payment object.

That last choice deserves its reason. A minimal payload plus a fetch is more robust than a fat payload, because the fetch returns current state rather than state as of an event that may be 20 hours old, and because it keeps the signed body small. The cost is one extra call per event on the merchant's side, and for merchants who cannot afford that I offer the fat payload as an option with a documented staleness caveat.

The 35 percent of merchants on hosted platforms cannot write custom verification code, so the practical mitigation is an official plugin or a documented no-code path, and, more importantly, that population is the reason the signature scheme must be simple enough to describe in one paragraph.

The error most readers make at this step: designing the producer perfectly and leaving the consumer to figure out idempotency. Publish the consumer's algorithm explicitly: check the signature, check the timestamp is recent, look up the event identifier and return 200 immediately if seen, otherwise enqueue and return 200, then process asynchronously and discard any event whose version is not newer than stored state. Four lines, and they prevent most of your integration tickets.

### 15e. Fade the scaffold

**Faded example 1.** Last block blanked. A calendar product must notify integrators when events change. Inputs: 300 integrators, 2 million calendar changes per day, and a single busy calendar can generate hundreds of changes per minute during a bulk import.

- Block 1, guarantee: at least once, ordered per calendar, versioned, 5 second deadline.
- Block 2, write path: outbox in the same transaction as the calendar mutation.
- Block 3, fleet: 2 million per day is about 23 per second average, and the per-calendar burst is the real problem rather than the average.
- Block 4, how to stop a bulk import from flooding integrators: **you write this**.

??? note "Show answer"

    Collapse rather than deliver. Because the guarantee is per-calendar ordering with versions, consecutive events for the same object can be coalesced into one event carrying the latest version, so a bulk import that touches one calendar 400 times in a minute produces a handful of deliveries rather than 400.

    Make the mechanism explicit: hold events for an object in a short debounce window, for example 2 to 5 seconds, and emit the newest, keeping the count of collapsed changes in the payload so a consumer knows it happened. This is only sound because the payload is a pointer to current state rather than a diff. If events carried diffs, collapsing would lose information and would be forbidden.

    Say that dependency out loud, because it is the reason the payload decision from block 4 of the worked example is load bearing.

    Second half: give integrators a bulk-change event type for imports, so the semantic "many things changed, go resynchronise" is available instead of a flood of individual events. Publishing a coarse event is often better product design than delivering a fine one perfectly.

**Faded example 2.** Last two blocks blanked. An internal service must tell three other internal services when an account is closed. All four services are in one deployment, one region, and there is already a message bus.

- Block 1, guarantee: at least once with per-account ordering, and consumers deduplicate.
- Blocks 2, 3 and 4: **you write these**.

??? note "Show answer"

    **Block 2, write path.** Still an outbox, because the dual write problem does not care that the consumer is internal. The bus is not transactional with your database, so publishing inline has exactly the failure modes from 15a. This is the point most people miss: the outbox is about your database and your publish step, not about who is consuming.

    **Block 3, transport.** Use the existing bus, not a webhook system. Three known consumers inside one deployment do not need signatures, per-destination queues, attempt logs, a replay console or an automatic disable rule. Building an HTTP delivery service here is the clearest over-engineering example in this module.

    **Block 4, the honest recommendation.** Consider not emitting an event at all. With three consumers in one deployment, a synchronous call to each, or a shared closure workflow, may be simpler and gives you an immediate error instead of an eventual one. Events are the right answer when the producer must not know its consumers, and here the producer knows all three by name.

    The strongest version of this answer names the condition under which events do become right: when the consumer list is expected to grow beyond a handful, or when a consumer must be allowed to be down without blocking account closure.

### 15f. Check yourself

**Q1.** Your webhook system delivers at least once and consumers are told to deduplicate by event identifier. A merchant reports being told about a refund it had already processed, three hours late, after which its own state went backwards. Which of your guarantees was insufficient, and what field fixes it?

??? note "Show answer"

    Deduplication by identifier was working: the problem is not a duplicate, it is a stale event arriving after a newer one. A retry from three hours ago is a different event identifier than the more recent update, so deduplication cannot catch it. The insufficient guarantee is ordering: retries break per-object ordering by construction, because a retried delivery is by definition later than the events that followed it.

    The fix is a monotonically increasing version per object, carried in every event, with the published instruction that consumers discard any event whose version is not newer than the state they hold. That converts "we usually deliver in order" into a guarantee consumers can enforce themselves, which is the only kind of ordering guarantee worth publishing over a retrying transport.

**Q2.** Someone proposes removing the outbox because "the message bus has a 99.99 percent publish success rate, so we lose one event in 10,000 and can reconcile those". Give the argument against, and the one condition under which they are right.

??? note "Show answer"

    The argument against: the outbox is not primarily about publish failures, it is about atomicity between the state change and the obligation. A 99.99 percent publish rate says nothing about the interleaving where the process dies between the commit and the publish call, which is caused by deploys and out-of-memory kills rather than by bus availability, and it says nothing about the reverse case where the publish succeeds and the transaction rolls back. Both produce a permanently inconsistent view that no publish reliability figure covers.

    The condition under which they are right: if a reconciliation process already exists, runs frequently, compares source state against consumer state, and repairs differences, then the outbox is a latency optimisation rather than a correctness mechanism, and dropping it is a legitimate trade. Ask whether that reconciler exists, is tested, and is measured. In most systems the honest answer is that it is a script somebody wrote once, which is why the outbox stays.

### 15g. When not to use this

**A webhook delivery system.**

| Question | Answer |
|---|---|
| The measurement that justifies it | Number of consumers you do not deploy, multiplied by their ability to host a reachable endpoint. Plus the cost, to them, of polling at the freshness they need |
| The threshold below which it is over-engineering | Fewer than roughly 10 consumers, all internal, all on an existing message bus. A delivery system is a signing scheme, a scheduler, an attempt log, a dead letter store, a replay console and an on-call rotation |
| What to do instead | Use the bus you already have, keeping the outbox. For a handful of external consumers, publish a pull-based events feed instead: an ordered, cursor-paginated list of events with a retention window |

**The pull alternative, priced.** A consumer polling an events feed every 10 seconds gets at most 10 seconds of staleness and costs 8,640 requests per day. For 50 consumers that is 432,000 requests per day, or 5 per second, which is nothing. In exchange you delete the entire delivery system: no signing, no retries, no dead letters, no disabled endpoints, no replay tooling. The consumer keeps its own cursor, which means missed events are impossible by construction rather than by your diligence.

Choose push when consumers need sub-second freshness, or when there are thousands of them and the polling load becomes real, or when consumers are unsophisticated and a cursor is too much to ask. Choose pull when consumers are few, when they sit behind a firewall, or when they are browsers, which cannot receive a webhook at all. Saying both options out loud, with the poll cost computed, is a stronger answer than designing the push system flawlessly.

---

## Module 16. Batch, bulk and streaming interfaces

**Time: 15 to 19 minutes reading.**

### 16a. Predict first

```
+--------------------------------------------------------------+
| POST /contacts/bulk                                          |
|   body:  { "contacts": [ ...100 objects... ] }               |
|                                                              |
|   200 OK                                                     |
|   { "created": 97 }                                          |
+--------------------------------------------------------------+
Caption: a bulk create endpoint, its request and its entire
response for a request of one hundred items.
```

**Question.** A client sends this and gets that response. List everything it now cannot do.

??? note "Show answer"

    It cannot identify the 97 that were created, so it cannot store their identifiers, link them to its own records, or show them to a user.

    It cannot identify the 3 that failed, so its only options are to give up or to resend all 100, which will either duplicate the 97 or fail again in the same opaque way.

    It cannot know why the 3 failed, so it cannot distinguish a validation error it should fix from a transient error it should retry from a duplicate that is actually fine.

    It cannot resume. A timeout partway through leaves it with no information at all, not even the count.

    And it cannot retry safely, because there is no item-level identity, so the whole call is a coin flip between duplicating work and losing it. A bulk endpoint without per-item results is not a bulk endpoint, it is a batch job with the observability removed.

### 16b. Partial success is a design decision, not an accident

Two honest semantics exist, and you must choose one and say which.

| Semantics | Response | Correct when |
|---|---|---|
| All or nothing | One status for the whole call; nothing is applied if anything fails | The items form one logical unit, for example the lines of one invoice, where a partial invoice is meaningless |
| Per item | A result entry per item, each with its own status, identifier or error | The items are independent, which is the usual case for imports, event ingestion and fan-out reads |

For per-item semantics, the response is an array positionally aligned with the request, and each entry carries the outcome for that item. Return an overall 200 even when some items failed, and say so in the documentation: the call succeeded, some items did not.

Overloading the transport status to mean "some of your items were invalid" leaves clients unable to distinguish that case from a genuine request failure. The WebDAV specification's 207 status exists for this shape, and borrowing it into a JSON product interface tends to confuse more clients than it helps, so state your choice explicitly either way.

Each item entry needs four things:

- The client's own reference for the item, echoed back, so alignment does not depend on array position surviving every proxy and serialiser.
- The created or affected identifier on success.
- A machine-readable error code plus the offending field on failure, following the error taxonomy from Module 7.
- A retryability signal per item, because in one response some failures are permanent (invalid email) and some are transient (a lock timeout).

**Item-level idempotency.** The whole call gets an idempotency key, and each item carries a client reference that is unique within the tenant. On a retry of the same key, return the stored result. On a partial retry of only the failed items, the client references let you detect duplicates for anything that actually succeeded. Without the per-item reference, a partial retry is unsafe and the client's only correct behaviour is to give up.

### 16c. Sizing, chattiness and flow control, each with a number

**Sizing a batch, derived.** Assume per-item work of 20 ms measured, and an endpoint latency budget of 5 seconds.

- Maximum items that fit the budget: 5,000 ms divided by 20 ms, which is 250.
- Publish 100, not 250. The margin absorbs slower items, a cold cache and a busy database, and it leaves room for the budget to be met at p99 rather than at p50.
- State the limit in the documentation and enforce it with a specific error code that names the maximum, because a client that discovers your limit through a 500 will pick a smaller number by trial and error and never revisit it.

**Chattiness, priced.** Assume a client on a link with 80 ms round trip time importing 100 contacts.

| Approach | Wall clock | Notes |
|---|---|---|
| 100 sequential calls | 100 times 80 ms, so 8 seconds, plus server time | Also 100 sets of headers and 100 authorisation checks |
| 100 calls, 8 in parallel | About 1 second | Cheap to implement on a modern client, and often enough |
| 1 bulk call of 100 | 80 ms plus about 2 seconds of server work | One authorisation check, one transaction strategy, one retry decision |

Note what that table says: parallelism recovers most of the benefit without any new endpoint. The bulk endpoint wins decisively on server-side overhead and on giving the client a single retry unit, not on network time alone. Use that reasoning rather than "batching is faster".

**Streaming and backpressure, briefly.** When the result set is large and the client can process incrementally, streaming beats both pagination and bulk. The design rule that matters: a stream must have flow control, or a fast producer will overwhelm a slow consumer's buffers.

Either the transport provides it, or you build acknowledgement windows into the protocol, for example the client acknowledges every N records and the server stops at N unacknowledged. State which of the two you are relying on, because "we stream it" with no flow control is how a consumer runs out of memory.

### 16d. Worked example: a contact import endpoint

The prompt: customers import contact lists from spreadsheets, typically 500 to 50,000 rows.

**Block 1. Purpose: decide whether this is a bulk call at all.**

50,000 rows at 20 ms is 1,000 seconds of work, which is not a request, it is a job. So the top of the range is Module 14's territory: upload the file, receive an operation resource, poll for progress, download a per-row result report.

The 500-row case is 10 seconds of work, still over any comfortable synchronous budget, so the honest answer is that the whole feature is asynchronous and the bulk endpoint exists only for programmatic clients sending small batches.

The error most readers make here: designing one endpoint for the full range. Two different shapes serve two different clients, and pretending otherwise produces a synchronous endpoint that times out for half the users.

!!! question "Say it before you read on"

    Before the next block: say which of the two shapes needs per-item results more urgently, and why.

**Block 2. Purpose: give the small-batch endpoint honest semantics.**

Per item, capped at 100, with a client reference required on every item. Response is an array of results with identifier or error code plus retryable flag, and an overall 200 whenever the request itself was well formed.

I require the client reference rather than making it optional, because optional identity produces exactly the systems that cannot retry safely, and it costs the client nothing at import time.

!!! question "Say it before you read on"

    Before the next block: say what the 50,000-row path must return that a single overall status cannot, and where a client would go to read it.

**Block 3. Purpose: connect the large path to the operation resource without duplicating semantics.**

The file upload returns an operation. Progress is rows completed against rows total. The terminal result is not a summary, it is a downloadable report with one line per input row carrying the same four fields the small-batch response uses. Same semantics, different envelope, which means one documentation page and one mental model for the client.

Retention on that report follows Module 14's rule, and here it should be longer than the export case: the report is how a customer fixes 300 bad rows, and they will do that over days, not hours.

### 16e. Fade the scaffold

**Faded example 1.** Last block blanked. An analytics ingestion endpoint accepts up to 1,000 events per call from mobile clients on poor networks.

- Block 1: bulk is right, because the client is buffering anyway and network cost dominates.
- Block 2: per-item semantics, each event carrying a client-generated identifier for deduplication.
- Block 3, what the response should contain given a mobile client on a poor network: **you write this**.

??? note "Show answer"

    Not a per-item array. This is the exception to the rule, and recognising the exception is the point of the exercise.

    A 1,000-entry result array can be larger than the request that produced it, on a link where bytes are the scarce resource, and the client has nothing useful to do with per-item success entries for analytics events it will never reference again. So return a compact response: the count accepted, and a list of only the rejected items with their client identifiers and error codes. Successes are implied.

    Two supporting decisions: rejected items are almost always a client bug rather than a transient failure, so the client should log and drop rather than retry, and the documentation must say so. And because the client deduplicates using its own identifiers, a retry of the whole batch is safe, which means the client can use a dumb retry policy and stay correct.

**Faded example 2.** Last two blocks blanked. A payments product is asked for a bulk refund endpoint accepting up to 500 refunds in one call.

- Block 1: refunds are independent, so per-item semantics, but money changes the calculus.
- Blocks 2 and 3: **you write these**.

??? note "Show answer"

    **Block 2, semantics with money involved.** Per item, but the call size drops sharply, to something like 50, because a partially applied set of 500 refunds is a reconciliation problem for a human, and smaller units mean smaller messes. Every item carries its own idempotency key, not just the batch, so a client can retry the 3 that failed without any risk of double-refunding the 47 that succeeded. Item-level keys are optional for contacts and mandatory for money.

    **Block 3, the answer that reads as senior.** Consider declining to build it. A bulk refund endpoint is a mechanism for making 500 irreversible money movements with one call and one authorisation check, which is also an excellent description of an incident. The alternatives are an operation resource with an explicit approval step and a preview of the total amount, or a rate-limited single-refund endpoint that a client loops over, which is slower and much harder to misuse.

    If it is built anyway, the controls follow from the risk rather than from the shape: a maximum total value per call, a preview endpoint that returns exactly what would happen without doing it, a distinct scope required for bulk operations, and every bulk call recorded with the principal, the total and the item list in an audit log that support can read.

### 16f. Check yourself

**Q1.** A client complains your bulk endpoint is unreliable: about 1 percent of its calls "half work". Nothing in your logs shows errors. What is the most likely design defect, and what do you change?

??? note "Show answer"

    The endpoint is almost certainly returning a single overall status with no per-item results, so a call in which some items failed validation looks like a success to you and like a mystery to the client. Your logs show no errors because, from the server's point of view, there were none: the request succeeded and some items did not apply.

    The change is per-item results with client references echoed back, plus a documented statement of what an overall 200 means. Then add the operational half: a counter of per-item failures by code, per client, so that "1 percent half works" becomes a graph you can look at before the client tells you. A bulk endpoint whose item-level failures are not counted is unobservable by construction.

**Q2.** Your per-item work is 40 ms and your endpoint budget is 2 seconds. A client asks for a batch limit of 500. What do you tell them?

??? note "Show answer"

    500 items at 40 ms is 20 seconds, which is ten times the budget, so the answer is no with arithmetic attached rather than no with a policy attached. The limit that fits the budget is 2,000 divided by 40, which is 50, and I would publish 25 to leave margin for slow items and a busy database at p99.

    Then offer the alternative rather than stopping at the refusal: if they genuinely need 500 in one unit of work, that is an operation resource with progress and a result report, and it is a better fit for them anyway because it survives their client restarting. The pattern worth internalising is that a batch size request is usually a latency or reliability request in disguise, and the useful move is to ask what they are trying to avoid.

### 16g. When not to use this

| Question | Answer |
|---|---|
| The measurement that justifies a bulk endpoint | Client-observed round trip time multiplied by the typical number of items, compared against the same work in one call, plus your own per-request overhead (authorisation, rate limit check, connection setup) multiplied by the item count |
| The threshold below which it is over-engineering | Typical batches under about 5 items on a link under about 30 ms round trip. There the saving is a fraction of a second and you have doubled your error surface, your idempotency story and your documentation |
| What to do instead | Connection reuse with 4 to 8 parallel single-item calls, which is a few lines in any client library and keeps one simple error model |

The corollary for streaming: do not stream when the result fits comfortably in one response and the client cannot start work until it has all of it anyway. Streaming buys incremental processing, and if the consumer is going to collect the whole set into an array before doing anything, you have added flow control and partial-failure semantics in exchange for nothing.

---

## Module 17. Client constraints as design input

**Time: 12 to 16 minutes reading.**

### 17a. Predict first

```
+--------------------------------------------------------------+
| GET /feed  returns 50 items, 42 fields each                  |
| Measured payload: 180 KB uncompressed, 46 KB gzipped         |
|                                                              |
| CLIENT NETWORK (assumed, stated aloud):                      |
|   throughput 400 kbps = 50 KB/s                              |
|   round trip time 300 ms                                     |
|   the screen renders 6 items and 9 of the 42 fields          |
+--------------------------------------------------------------+
Caption: one response, its measured size compressed and not, and
the network and screen constraints of the client reading it.
```

**Question.** Compute time to first render for this response, then say which single change to the contract reduces it most, and by how much.

??? note "Show answer"

    Time to first render: one round trip of 300 ms to get the first byte, plus 46 KB at 50 KB/s, which is 920 ms of transfer, so about 1.2 seconds before the client has a complete response to parse. That is with compression already applied.

    The single biggest change is not compression, which is already on, and not field selection alone. It is returning 6 items instead of 50, because the screen shows 6. That cuts transfer to roughly 46 KB times 6 divided by 50, about 5.5 KB, which is 110 ms of transfer, so around 410 ms total. Roughly a 3x improvement from one query parameter.

    Field selection compounds it: 9 fields of 42 is about a fifth of the bytes if fields are of similar size, taking the 6-item page from 5.5 KB to roughly 1.1 KB, so about 320 ms total and now dominated entirely by the round trip.

    The lesson is the ordering: cut item count first, then field count, then worry about the encoding. Candidates usually propose the encoding change first because it feels technical, and it is worth the least.

### 17b. Three levers, in the order they pay

| Lever | What it does | Typical saving | Cost to you |
|---|---|---|---|
| Page size that matches the screen | Stops sending items nobody will look at | Proportional: 6 of 50 is an 88 percent cut | None, if pagination already exists |
| Sparse field selection | Client names the fields it needs | Proportional to fields dropped, often 60 to 80 percent | Response caching gets harder, because each field set is a distinct cache entry |
| Delta sync | Send only what changed since a cursor the client holds | Proportional to churn: 3 changed of 200 held is a 98 percent cut | A change log, tombstones for deletes, and a full-resync path |

**Delta sync, derived and bounded.** Assume a client holds 200 entities at about 0.9 KB each, so a full sync is 180 KB, and assume 3 entities changed since the last sync.

- Full sync: 180 KB.
- Delta: 3 times 0.9 KB, so 2.7 KB, plus the cursor overhead. Roughly 66 times less.

Now find where it stops paying. If 30 percent of the 200 entities change between syncs, the delta is 60 items, which is 54 KB against 180 KB: still a 3x saving, but you are now maintaining a change log, tombstones, a cursor per client and a resync path to buy 3x. Around and above that churn level, a conditional full fetch with an entity tag is simpler and nearly as good, because an unchanged collection costs one small response and a changed one costs the full body either way.

**The three things a delta protocol must specify,** all of which get skipped in weak answers:

1. Deletes. A delta that only carries changed rows can never tell a client that something disappeared. You need tombstones with their own retention window, and the window must exceed the longest plausible client absence or you need a forced resync.
2. The resync trigger. When a client's cursor is older than the tombstone window, or the schema changed, the server must be able to say "discard everything and start over" in a single response. Clients that cannot be told to resync accumulate permanent corruption.
3. Ordering and atomicity. If a client applies half a delta and dies, its cursor must not advance. Advance the cursor only after the whole batch is durably applied locally.

### 17c. Chattiness is a property of the contract, not of the client

**Why a chatty interface costs more than its bytes.** This is an engineering point about radios, not a claim about any particular device. A cellular radio does not return to a low-power state the instant a transfer ends; it stays in a higher-power state for a tail period so that a following transfer does not pay the setup cost again.

The consequence for interface design is direct: twenty small requests spread over a minute keep the radio active far longer than one request that transfers the same total bytes, so the energy cost is driven by the number and spacing of requests, not by the byte count alone.

Two design rules follow, and both belong in an interface, not just in a client:

- Give the client one call that returns what a screen needs, rather than making it assemble a screen from five. Screen-shaped responses are a legitimate contract decision, not a leaky abstraction, when your client count is small and known.
- Give the client a way to batch its writes and to defer non-urgent ones, so background telemetry and read receipts can ride along with the next real request instead of waking the radio on their own.

The device-side half of this argument, including background execution limits and battery accounting, belongs to [Mobile System Design](mobile-system-design.md). Here the point is narrower: chattiness is a property of the contract you published, so it is your design problem.

### 17d. Worked example: shaping a feed contract for a slow network

The prompt: our feed is unusable in markets with slow connections. Fix the contract.

**Block 1. Purpose: get a measurement before proposing anything.**

I ask for two numbers, and I refuse to design without them: the measured payload size at p50 and p95, and the client's round trip time distribution in the affected markets. Without those, every proposal is a guess and I cannot say which one wins.

Using the numbers from the pre-question, transfer dominates at 920 ms against a 300 ms round trip, so the priority is bytes rather than call count. If round trip had been 800 ms and payload 5 KB, the priority would invert entirely and I would be merging calls instead of shrinking them.

The error most readers make here: proposing the fix they know best rather than the fix the ratio indicates. State the ratio out loud, then choose.

!!! question "Say it before you read on"

    Before the next block: say what you would change first if the measurement showed 4 KB payloads and a 900 ms round trip, and why it is the opposite of the change above.

**Block 2. Purpose: change the cheapest thing that moves the number.**

Page size becomes a required parameter with a documented maximum and a default that matches a phone screen rather than a desktop. This is a one-line change on the server and an 88 percent cut for the affected clients.

Then sparse field selection, in the restrained form: a small number of named field sets (`compact`, `standard`, `full`) rather than arbitrary field lists. Named sets keep the response cacheable, because there are three cache entries per resource instead of one per combination of 42 fields, and they keep the documentation finite.

!!! question "Say it before you read on"

    Before the next block: name the number you would ask for before deciding whether delta sync belongs here at all, and state the value above which the answer is no.

**Block 3. Purpose: decide about delta sync using the churn number, not enthusiasm.**

I ask what fraction of a client's held items change between typical sessions. If the feed is chronological and mostly append-only, the answer is small and a cursor-based incremental fetch is already what pagination gives me, so I need nothing new. If items are heavily edited or reordered by ranking, churn is high and delta sync would deliver most of the collection anyway while adding tombstones and a resync path.

For a ranked feed, the honest conclusion is usually that delta sync is the wrong tool: the ranking changes for reasons unrelated to the items, so "what changed" is nearly everything. Saying that, and choosing conditional requests plus small pages instead, is a stronger answer than building the more sophisticated thing.

### 17e. Fade the scaffold

**Faded example 1.** Last block blanked. A field service application syncs a technician's job list to a device that spends hours offline in buildings with no coverage.

- Block 1: measure. Jobs per technician per day, about 40, at 2 KB each; sessions are long and offline gaps are hours.
- Block 2: page size is irrelevant here because 40 items is small; the binding constraint is the offline gap, not the byte count.
- Block 3, the sync design and its resync rule: **you write this**.

??? note "Show answer"

    Delta sync wins here for a reason that has nothing to do with bytes: the device is offline for hours and must reconcile on reconnection, so it needs an ordered change log anyway. The saving is a side effect; the requirement is convergence.

    The design: a cursor per device, a change log carrying creates, updates and tombstones, and the cursor advancing only after the whole batch is durably applied on the device. Tombstone retention must exceed the longest plausible offline gap. Since a technician can be off for a weekend plus a public holiday, a retention window of 30 days is generous, cheap, and turns a resync from a routine event into an exception.

    The resync rule, stated explicitly: if the cursor is older than the tombstone window or the schema version changed, the server returns a resync directive and the client discards local state and refetches. And the write half, which this prompt implies and most answers drop: the device also accumulates outbound changes offline, so it needs an outbox with idempotency keys, and the conflict policy has to be stated rather than left to whichever write lands last.

**Faded example 2.** Last two blocks blanked. A public read-only content interface serves 40 million requests a day to browsers worldwide, with a content delivery network in front.

- Block 1: measure. Payloads are 20 KB, round trip is low for most users because of edge caching, and the cache hit rate is 88 percent.
- Blocks 2 and 3: **you write these**.

??? note "Show answer"

    **Block 2, the lever that pays.** Not field selection. With an 88 percent cache hit rate, the dominant variable is the hit rate itself, and sparse field selection multiplies the number of cache entries per resource, which lowers the hit rate and makes the average request slower even though each response is smaller. The lever that pays is anything that raises the hit rate: a small, fixed set of response shapes, no user-specific fields in cacheable responses, and query parameters normalised so that equivalent requests share a cache entry.

    **Block 3, what to do instead of delta sync.** Conditional requests with entity tags, so an unchanged resource costs a small validation response rather than 20 KB, plus cache lifetimes long enough for the edge to serve most traffic without revalidating. Delta sync is meaningless here because browsers hold no durable client state to delta against.

    The general rule this example teaches: every lever in this module is evaluated against the caching layer, not just against the byte count. A change that halves the payload and halves the cache hit rate is usually a loss, and computing it is a two-line argument you can make in an interview.

### 17f. Check yourself

**Q1.** A mobile team asks for a single endpoint that returns everything their home screen needs, arguing it saves five round trips. Give the case for and the case against, and the condition that decides it.

??? note "Show answer"

    For: five round trips at, say, 300 ms each is 1.5 seconds of latency the user experiences directly, plus five sets of headers, five authorisation checks and five wake-ups of the radio. A screen-shaped response removes all of that and is a legitimate contract, not a hack.

    Against: a screen-shaped endpoint couples your interface to one client's layout, so every redesign becomes a contract change. It also fragments caching, because the aggregate response changes whenever any of its five parts changes, and it makes the response the least cacheable object in the system.

    The deciding condition is the number and controllability of clients. With one or two clients you deploy yourself, the coupling is cheap and the latency win is real, so build it, ideally as a composition layer in front of stable resource endpoints rather than as a new domain concept. With many clients you do not control, the coupling is permanent and the correct answer is a general mechanism such as field selection or multiple resource fetches over one connection.

**Q2.** Sparse field selection is proposed for a public interface sitting behind a cache. State the cost in one sentence and the restrained version of the feature.

??? note "Show answer"

    The cost: every distinct field combination is a distinct cache entry, so a feature that lets clients request any subset of 42 fields turns one cached object into an unbounded number of rarely reused ones, and the cache hit rate falls for everybody.

    The restrained version is a small set of named field sets, three or four, each documented and each cached independently. That captures most of the byte saving (the difference between a compact and a full representation is where nearly all the bytes are) while keeping the cardinality of cache entries fixed and small. If a client needs a combination you did not name, that is a signal to add a named set, not to open the parameter.

### 17g. When not to use this

**Delta sync.**

| Question | Answer |
|---|---|
| The measurement that justifies it | The fraction of a client's held entities that change between typical syncs, and the length of the client's typical offline gap |
| The threshold below which it is over-engineering | Churn above roughly 30 percent, or clients that hold no durable state at all. At high churn the delta approaches the full set while you still pay for a change log, tombstones, per-client cursors and a resync path |
| What to do instead | Conditional requests with an entity tag and a full replace on change, with a page size that matches the screen |

**Sparse field selection.**

| Question | Answer |
|---|---|
| The measurement that justifies it | The share of bytes in fields the client never renders, measured on a real response, and the cache hit rate the surface currently enjoys |
| The threshold below which it is over-engineering | Unrendered fields under roughly 30 percent of bytes, or a surface whose performance depends on a high cache hit rate |
| What to do instead | Two or three named representations, chosen so the compact one covers the common screen, and let the cache keep working |
---

## Module 18. AI-era interfaces: tool schemas as contracts

**Time: 24 to 30 minutes reading.**

Everything in this course so far assumed a caller that reads your documentation, compiles against your schema and behaves deterministically. A model-driven caller does none of those things. It reads a schema at runtime, chooses arguments probabilistically, and will happily call your refund endpoint because a user's message contained the word "refund". The contract has to absorb that.

### 18a. Predict first

```
+--------------------------------------------------------------+
| {                                                            |
|   "name": "refund_payment",                                  |
|   "description": "Refund a payment",                         |
|   "parameters": {                                            |
|     "payment_id": { "type": "string" },                      |
|     "amount":     { "type": "number" },                      |
|     "reason":     { "type": "string" }                       |
|   }                                                          |
| }                                                            |
+--------------------------------------------------------------+
Caption: a tool schema exposed to a model that handles customer
support conversations, exactly as written, with no other fields.
```

**Question.** List five things missing from this schema that would matter to a deterministic client and matter far more to a model-driven one. Then name the one addition that most reduces the blast radius of a wrong call.

??? note "Show answer"

    1. No currency. `amount` as a bare number means the model must infer minor units and currency from context, and it will sometimes infer wrong. A refund of "50" against a payment in a currency with three minor units is a different amount than intended.
    2. No idempotency key. The harness will retry on timeout, and there is nothing to make the second call converge on the first, so a network blip becomes a double refund.
    3. No bounds. Nothing caps the amount, and nothing states that the amount cannot exceed the original payment. Validation is the only enforcement you have against a caller that guesses.
    4. No side-effect declaration. Nothing in the schema says this call moves money and cannot be undone, so no harness can treat it differently from a read.
    5. No enumerated reasons. A free-text `reason` will be a novel string every time, which makes downstream reporting impossible and lets the model write a reason that later reads as an admission of liability.

    The single addition that most reduces blast radius is not any of those: it is a required approval gate, expressed in the contract, so the call returns a pending decision rather than moving money, and a human confirms. Everything else narrows the space of wrong calls. The gate makes a wrong call recoverable.

### 18b. Design rules for a schema a probabilistic caller reads

A tool schema is a contract with three unusual properties: the caller reads the description as instructions, the caller cannot be recompiled, and the caller can be steered by the data it processes. Design accordingly.

| Rule | Concretely | Why it matters more here |
|---|---|---|
| Constrain at the schema level | Closed enumerations, numeric minimums and maximums, string patterns, required fields | Validation is the only enforcement. A deterministic client is constrained by its own code; this one is not |
| Name things unambiguously | `amount_minor_units` with an explicit `currency`, never a bare `amount` | The caller resolves ambiguity by guessing, and guesses are not uniformly distributed towards safety |
| Write the description as a specification | State when to call, when not to call, and what a missing argument means | The description is the only documentation the caller reads, and it reads it every time |
| Declare the side effect | An explicit classification on every tool | Lets one harness rule govern many tools, instead of per-tool special cases |
| Keep the surface small | Five tools that compose beat forty that overlap | Selection error grows with the number of similar options, and similar options are how you get the wrong one |
| Return machine-actionable errors | A code, plus what to do next in words the caller can act on | The caller's next move is generated from your error text, so the error is part of the control flow |

**The side-effect classification, which is the load-bearing idea.**

| Class | Definition | Harness rule |
|---|---|---|
| Read only | No state changes anywhere | Call freely, retry freely, cache if useful |
| Reversible write | Changes state, and an inverse operation exists | Call freely, retry with an idempotency key, log for audit |
| Irreversible or external | Moves money, sends a message to a person, changes access, calls a third party | Requires an approval gate or a scoped policy allowance, always idempotency keyed, always audited |

Classify once, in the schema, and the harness enforces the rule uniformly. The alternative, deciding case by case in prompt text, fails the first time someone adds a tool without reading the convention.

**Errors written for a caller that will act on them.** Compare two responses to the same condition:

- Weak: `{"error": "Invalid request"}`. The caller has no next action, so it will retry the identical call, then try a different tool, then apologise to the user.
- Strong: `{"code": "amount_exceeds_payment", "retryable": false, "detail": "amount_minor_units must be <= 4200", "next": "Ask the user whether to refund the full 4200 or a smaller amount."}`.

The `next` field looks unusual to anyone from a traditional interface background, and it is the highest-value field in the response. You are writing to a caller that composes its next action from your text, so tell it what the action is. Keep the human message separate from the machine code, exactly as Module 7 requires, because here the "human" reading the message is another program.

**Prompt injection changes who your caller is.** If the model processes untrusted content, for example a customer's email, that content can attempt to steer the tool calls. Two consequences for interface design, both of which belong in your answer:

- Authorisation is never delegated to the caller's judgement. The tool call executes with a token scoped to the session's actual principal, and the scope is enforced server side, so a successful injection can only do what that user could already do.
- Irreversible tools sit behind gates that the model cannot open. If the model can decide that approval is unnecessary, there is no gate.

### 18c. Cost, streaming and version pinning

**Metering, derived.** Assume input tokens cost 3 per million and output tokens 15 per million. Those two numbers are an illustrative assumption used only to make the arithmetic concrete, chosen because they sit in a plausible range for a capable model; substitute your own and the method is unchanged.

One agent turn with 12,000 input tokens (system prompt, tool schemas, conversation, retrieved context) and 800 output tokens:

- Input: 12,000 divided by 1,000,000, times 3, which is 0.036.
- Output: 800 divided by 1,000,000, times 15, which is 0.012.
- Total: 0.048 per turn.

A support session of 50 turns is 2.40. Now the product consequences fall straight out:

| Consequence | The arithmetic that forces it |
|---|---|
| Per-request quotas are meaningless | One request can cost 100 times another depending on context length. Meter in cost units, exactly as Module 13 argued for search |
| A flat subscription needs a usage cap | At 2.40 a session, a 20 per month plan breaks even at about 8 sessions. That number belongs in the design conversation, not in a later surprise |
| Context length is a product decision | Input dominates here: 0.036 of 0.048 is 75 percent. Trimming retrieved context is the largest single cost lever |
| Usage must be in the response | Return input tokens, output tokens and a cost figure per call, so the client can budget without reverse engineering your bill |

**Retry cost, derived.** If you enforce structured output by validating and retrying, and the failure rate is 8 percent, the expected cost multiplier is about 1 plus 0.08, which is 1.08 for a single retry, plus a long tail if you allow several. Two design responses: prefer schema-constrained generation where the platform offers it, so the failure rate approaches zero without retries, and cap retries at two with an explicit failure to the caller, because an uncapped validate-and-retry loop is an unbounded bill with no error ever surfacing.

**Streaming and cancellation, with the semantics stated.** Streaming a response is easy. The contract question is what a partial stream means.

| Question | The answer that must be in your documentation |
|---|---|
| Is streamed content committed? | No. Streamed tokens are a preview. Nothing is persisted or acted on until a terminal event |
| What ends a stream? | An explicit terminal event carrying the final state and the usage figures. A closed connection is not a terminal event |
| What does cancellation do? | Stops generation. It does not undo tool calls already executed, and the response must say which ones ran |
| Who pays for a cancelled call? | Tokens generated before cancellation. State it, because a client that cancels aggressively expects a refund it will not get |

The trap in this area: allowing a tool call to execute during a stream, then cancelling, then leaving the client unable to find out what happened. The repair is that every tool execution produces an event in the stream and a durable record keyed by the call identifier, so a cancelled or dropped stream can be reconciled afterwards.

**Version pinning, because a model is an undeclared dependency.** Your schema does not change when the underlying model changes, but the distribution of outputs does. A client compiled against your schema can break on a Tuesday because the model got better at something.

So treat the model version as part of your contract: pin it per credential, expose the version in every response, and give clients an explicit opt-in to move. Module 11's compatibility rules then apply to a dimension they were never written for, and saying that out loud is a strong senior signal.

### 18d. Worked example: a refund tool for a support agent

The prompt: our support agent should be able to refund payments on behalf of a customer, from a chat conversation.

Stated inputs: refunds average 42 (currency units), 3 percent of conversations reach a refund, the agent handles 20,000 conversations a day, and current human-agent refund policy allows full refunds under 100 without a supervisor.

**Block 1. Purpose: classify the operation before designing it.**

Refunds move money to a third party and cannot be recalled, so this is the irreversible class. That classification, not the prompt engineering, decides everything downstream: idempotency key mandatory, approval gate required unless a policy allowance covers the case, and every call audited with the conversation identifier.

I mirror the existing human policy rather than inventing one: under 100 proceeds, at or above 100 requires a supervisor. That is not laziness, it is the right instinct in an interview. The organisation has already priced this risk for humans, and matching it makes the automated path auditable against a policy that already exists.

The error most readers make here: designing an approval gate for everything, which sounds safe and destroys the feature. If a human agent can refund 42 without asking anyone, requiring approval for a 42 refund means the automated path is slower than the manual one and nobody uses it.

!!! question "Say it before you read on"

    Before the next block: say who generates the idempotency key for this tool, and why it cannot be the model.

**Block 2. Purpose: write the schema so wrong calls are mostly impossible.**

- `payment_id`: string, pattern constrained to your identifier format, required.
- `amount_minor_units`: integer, minimum 1, maximum supplied dynamically as the payment's remaining refundable amount, required.
- `currency`: not a parameter at all. It is derived server side from the payment. Removing a parameter is the strongest possible constraint, and any argument the server can derive should not be an argument.
- `reason`: closed enumeration of about six values that match the existing support taxonomy, required.
- `idempotency_key`: not a model parameter either. The harness generates it deterministically from the conversation identifier plus the payment identifier plus the amount, so a retried tool call converges and a genuinely second refund attempt does not.

Two of the five original fields have left the schema entirely. That is the pattern: every field you can derive or generate is a field the caller cannot get wrong.

**Block 3. Purpose: put the gate where the policy says, and make the response usable by the caller.**

Under 100: execute, return the refund identifier, the new refundable balance and the usage figures.

At or above 100: return a pending approval object with an identifier and an explicit `next` telling the caller to inform the customer that a supervisor is reviewing it, and not to retry. The approval is a resource in its own right, with its own state machine, which is Module 14's operation resource reused rather than a new concept.

Failure responses carry code, retryable and next. For example, `payment_already_refunded` is not retryable and its `next` instructs the caller to tell the customer the refund is already processed and to give the original refund identifier.

!!! question "Say it before you read on"

    Before the next block: the gate in block 3 produces an approval object. Say which module already built that shape, and why reusing it is better than inventing an approval concept here.

**Block 4. Purpose: meter and bound the cost, and make the whole thing auditable.**

Per-tenant metering in cost units, from the arithmetic above: at 20,000 conversations a day and, say, 12 turns each at 0.048, the daily model cost is around 11,500 currency units. That figure, derived in the room, is what turns "should we cache the system prompt" from an engineering preference into a funded decision.

Audit: every tool call records the conversation identifier, the principal whose token was used, the arguments, the approval decision if any, and the resulting refund identifier. The single most useful field is the principal, because the answer to "who refunded this" must never be "the assistant".

The error most readers make at this final step: treating the model's own transcript as the audit record. A transcript is a narrative and it can be influenced by the content the model processed. The audit record is the server-side log of what the contract actually executed, and only that log has standing.

### 18e. Fade the scaffold

**Faded example 1.** Last block blanked. A tool that searches an internal knowledge base and returns passages to the model.

- Block 1, classification: read only, so no gate, free retries, and caching is allowed.
- Block 2, schema: a query string, a result count with a maximum of 10, and an optional collection filter from a closed enumeration.
- Block 3, response shape: passages with source identifiers and a relevance score, plus usage.
- Block 4, the cost and safety considerations specific to a read-only tool: **you write this**.

??? note "Show answer"

    Cost first, because read-only does not mean cheap. Retrieved passages are input tokens on the next turn, so a tool that returns 10 passages of 800 tokens adds 8,000 tokens, which at the assumed 3 per million is 0.024 per call, half the cost of the whole turn computed earlier. So cap the result count, cap passage length, and return a token estimate in the response so the harness can decide whether to include everything.

    Safety second, and it is the non-obvious half: a read-only tool is the primary injection vector. The passages it returns are untrusted content that will be placed in the model's context, so the response must be structurally marked as data rather than instruction, and the harness must never grant a passage the authority to trigger another tool. State the rule: content returned by a retrieval tool cannot escalate what the session is allowed to do, because authorisation is bound to the session's principal and enforced server side.

    Third, an operational point: return source identifiers so any answer can be traced to the passages that produced it. Without that, you cannot debug a wrong answer and you cannot remove a bad document from circulation.

**Faded example 2.** Last two blocks blanked. A tool that sends an email to a customer on the user's behalf.

- Block 1, classification: irreversible and external, because an email cannot be recalled and it reaches a third party.
- Blocks 2, 3 and 4: **you write these**.

??? note "Show answer"

    **Block 2, schema.** Recipient is not a free-text parameter: it is an identifier for a contact already associated with the conversation, resolved server side. That single choice eliminates the entire class of failures where a model sends a customer's details to an address it read in a document. Subject and body are strings with maximum lengths, template identifier is a closed enumeration where templates exist, and the harness supplies the idempotency key.

    **Block 3, gate.** Always a preview and confirm step, not because of the amount but because the action is externally visible and unrecoverable. The tool returns a draft object with its own identifier; a separate confirmation, coming from the user's action rather than from the model, sends it. The model cannot call the send step directly, which is the structural version of the rule rather than the polite version.

    **Block 4, metering and audit.** Meter on sends, not on drafts, and rate limit sends per conversation and per recipient, because the failure mode here is not cost, it is a loop that emails a customer six times. Audit records the principal, the draft, the final content and the confirming action. And state the retention rule: drafts that are never confirmed expire, because an unconfirmed draft containing customer data is a liability with no owner.

### 18f. Check yourself

**Q1.** Your tool schema has a `confirm: boolean` parameter, and the harness executes the write when the model sets it to true. Explain why this is not an approval gate.

??? note "Show answer"

    Because the caller you are protecting against is the one deciding. The model sets `confirm`, so the gate is a suggestion in the schema rather than a control in the system, and any influence over the model, including untrusted content it processes, can set it to true. A control the untrusted party can operate is not a control.

    A real gate lives outside the model's reach: the tool returns a pending object, and a distinct actor (a person clicking, or a policy engine evaluating server-side rules against the session's principal and the amount) transitions it. The test to apply, and to say out loud, is: if the model were adversarial, could it complete this action alone? If yes, the gate is decorative.

**Q2.** A client complains that identical requests to your model-backed endpoint returned different shapes last Thursday, and your schema has not changed in months. Diagnose it and say what you owe them.

??? note "Show answer"

    The model version changed underneath the contract. Your schema describes the fields, but the distribution of outputs, including whether optional fields get populated and how free-text fields are phrased, is a function of the model, which is an undeclared dependency of your interface.

    What you owe them: the model version in every response so the change is observable, version pinning per credential so a change is opt-in rather than delivered on a Thursday, a stated notice period for moving the default, and treatment of model migrations under the same compatibility discipline as any other breaking change. If you cannot pin, say so explicitly in the contract, because a client that believes it has a stable dependency and does not is worse off than one told the truth.

### 18g. When not to use this

**A tool-calling surface at all.**

| Question | Answer |
|---|---|
| The measurement that justifies it | The share of real user requests that cannot be expressed as a fixed form or workflow, measured on a sample of actual transcripts or tickets rather than imagined |
| The threshold below which it is over-engineering | An operation with fewer than about five parameters and one obvious sequence. A form plus a deterministic endpoint is cheaper, faster, auditable by construction, and does not need an approval gate |
| What to do instead | The ordinary endpoint you would have built anyway, with a fixed user interface in front of it. Add a model layer when the variety of intent, not the novelty of the technology, forces it |

**Streaming, in this context specifically.**

| Question | Answer |
|---|---|
| The measurement that justifies it | Time to last token against the user's tolerance. Streaming buys perceived latency and nothing else |
| The threshold below which it is over-engineering | Responses that complete in under about a second, or any response the client cannot use until it is complete, such as one it will parse as a whole |
| What to do instead | Return the complete response. You avoid terminal-event semantics, cancellation semantics, partial-state reconciliation and a class of client bugs that only appear on slow networks |

---

## Module 19. Interface as product: developer experience

**Time: 15 to 18 minutes reading.**

### 19a. Predict first

```
+--------------------------------------------------------------+
| ONE MONTH OF THE DEVELOPER FUNNEL                            |
|                                                              |
| visited the documentation      1,000                         |
| created an account               420                         |
| obtained an API key              180                         |
| made a first successful call      96                         |
| made a call on a second day       41                         |
+--------------------------------------------------------------+
Caption: five funnel stages with their monthly counts, from the
documentation page through to a second day of usage.
```

**Question.** You have one engineering week. Which step do you attack, and what is the arithmetic that justifies it rather than attacking the biggest absolute drop?

??? note "Show answer"

    Stage conversion rates: 42 percent to an account, 43 percent to a key, 53 percent to a first call, 43 percent to a second day.

    The biggest absolute drop is 1,000 to 420, but that is a documentation and marketing problem, and most of those 580 were never going to build anything. The largest drop you own as an interface team is 420 to 180: 240 people who wanted a key enough to create an account and still did not get one. At 43 percent conversion, this step is also tied for the worst rate in the funnel.

    The one-week fix is usually not a better key page. It is removing the account requirement from the first call: issue a sandbox key instantly with no account, scoped to test data, and require the account only when the developer wants production access. That collapses two stages into one and moves the account creation step to a point where the developer already has something working and a reason to continue.

    The arithmetic that justifies it: if that change lifted the 420 to 180 step to 80 percent, you would have 336 keys instead of 180, and at the unchanged downstream rates, 178 first calls instead of 96 and 76 second days instead of 41. Nearly double the end of the funnel from one change.

### 19b. Metrics that make developer experience an engineering problem

Vague goals produce vague work. Five measurements turn it into a backlog.

| Metric | Definition | Why it drives design |
|---|---|---|
| Time to first successful call | Median minutes from landing on the documentation to a 2xx on a real endpoint | The single best summary of your onboarding, and it is measurable without asking anyone |
| First-call success rate | Share of developers whose first call succeeds rather than 4xx-ing | A low rate means your required fields, auth or examples are wrong, not that developers are careless |
| Error mix in week one | Distribution of error codes per new key over the first seven days | Tells you exactly which part of the contract is misunderstood. One dominant code is a documentation bug with a name |
| Support tickets per 1,000 calls, per client | Normalised so a big integrator does not look worse than a confused one | Finds the parts of the contract that cost humans, which is the expensive kind of defect |
| Share of traffic on a current client library | Calls from a library version released within the last N months | Predicts how much of a future migration you can do by shipping a library update instead of by asking people |

The last metric is strategically the most important and is almost never mentioned in interviews. If 80 percent of your calls come from a current library, a large class of breaking changes becomes a library release plus a deprecation window. If 20 percent do, every change is a broadcast negotiation with thousands of strangers.

**What a client library should own, and what it must not.**

| Owns | Does not own |
|---|---|
| Authentication, including refresh and rotation | Business logic, or opinions about what your resources mean |
| Retries with jittered backoff, honouring your retry-after signals | Hidden caching that makes two identical calls return different freshness |
| Pagination as an iterator, so nobody hand-writes cursor loops | Silent request rewriting the developer cannot see in a log |
| Idempotency key generation for unsafe methods, by default | Swallowing errors to look friendly |
| Typed errors carrying your machine-readable codes | Its own error taxonomy that diverges from the wire contract |

The rule underneath: a library removes work, never information. Any behaviour a developer cannot observe in a log is a behaviour that will cost you a support ticket at 3am.

**Generated or handwritten.**

| Approach | Wins | Costs |
|---|---|---|
| Generated from the schema | Always current, cheap in many languages, forces schema discipline | Reads like generated code, awkward idioms, hard to add retries and pagination helpers cleanly |
| Handwritten | Idiomatic, ergonomic, pleasant to use | Drifts from the contract, and you will staff at most two languages properly |
| Generated core, handwritten surface | Current where it matters, idiomatic where it is seen | Two layers to keep aligned, which needs a test that runs the surface against the generated core |

The third row is the honest recommendation for a surface with real external usage, and naming why (the generated part is where drift is dangerous, the handwritten part is where drift is invisible) is what makes it an answer rather than a preference.

### 19c. Sandboxes, keys and telling people a surface is going away

**The sandbox, and the two features that make it worth building.** A sandbox with realistic data is nice. A sandbox is only valuable if it can produce failures on demand. Two features:

1. Deterministic triggers: documented inputs that force specific outcomes, for example a particular test value that always produces a declined result or a rate limit response. Without triggers, nobody tests their error paths, and their first experience of your error model is in production at their busiest hour.
2. Event triggers: a control to fire any outbound event on demand. Otherwise integrators cannot develop against your webhooks at all without waiting for real activity.

**Keys, and the four properties that matter.**

- A visible, distinctive prefix, so leaked keys can be recognised by automated secret scanners and by humans reading a screenshot.
- Stored hashed, shown once. If you can display an existing key, so can anyone who reaches your database.
- Two active keys per account, always, so rotation is a rollout rather than an outage.
- Scoped and separately revocable, so a key for a reporting script is not also a key that can move money.

**Deprecation communication, ranked by what it actually reaches.** Module 11 built the mechanism; this is the human half. Ordered by observed reach, not by effort:

| Channel | Reaches | Weakness |
|---|---|---|
| Response headers | Every automated client, in the logs of anyone who looks | Nobody looks until something breaks |
| Client library warnings at compile or start time | Developers on a current library | Silent for handwritten clients |
| Dashboard banner scoped to affected keys | Whoever logs in, which is a minority of active integrators | Ignored by unattended systems |
| Email to the technical contact | The share of accounts with a current contact on file, which is never all of them | Filters, staff turnover, shared inboxes |
| Direct outreach to the largest consumers | The traffic that matters, if the concentration is high | Does not scale past a couple of hundred accounts |
| Scheduled brownouts | Everyone still on the old surface, by construction | Generates tickets, which is the point |

The sentence to say: every channel above brownouts is optional for the recipient, so the plan has to assume a residue that only a brownout reaches, and the residue's size is a measurement you already have from your traffic split.

### 19d. Worked example: cutting time to first call from 47 minutes to under 10

The prompt: our integrators say getting started is painful. Fix it.

Stated inputs, measured: median time to first successful call 47 minutes, first-call success rate 61 percent, and the top error code in week one is a missing required header on 38 percent of failing first calls.

**Block 1. Purpose: split the 47 minutes into parts before optimising any of them.**

I instrument the funnel by timestamp: landing to account, account to key, key to first call attempt, first attempt to first success. Suppose it splits as 6, 22, 9 and 10 minutes. Now the target is obvious: 22 minutes between having an account and having a key is a process problem, not a documentation problem.

The error most readers make here: rewriting the getting-started guide, because that is the visible artefact. The guide covers the 9 minute segment. Even perfect writing there cannot reach the 22.

!!! question "Say it before you read on"

    Before the next block: given that 38 percent of failed first calls are a missing header, say which funnel segment that failure lives in and roughly how many minutes it can account for.

**Block 2. Purpose: remove the step rather than speed it up.**

Sandbox keys issued instantly, no account, scoped to test data, rate limited hard, expiring in 14 days. The account requirement moves to the point where the developer asks for production access, which is a moment they are motivated to complete.

Any approval step that remains, for example for production keys on a regulated product, gets a stated turnaround and a status the developer can see. An invisible queue is what turns a 4 hour approval into a lost integrator.

!!! question "Say it before you read on"

    Before the next block: block 2 removed a step from the funnel. Say which of the four measured segments it shortens, and which segment is left as the largest remaining target.

**Block 3. Purpose: make the first call succeed by construction.**

Three changes, in order of value per hour of work:

1. A copy-paste snippet with the developer's own sandbox key already substituted, on the page where the key is issued. This removes the header mistake, because they never type the header.
2. An error response for the missing header that names the header, shows the corrected call, and links to the exact documentation anchor. Errors are documentation that arrives at the moment of need.
3. A first-call telemetry alert: if a new key's first ten calls all fail with the same code, that is a signal for a proactive message, and at low volumes a human can send it.

Recheck the metric afterwards rather than declaring victory: median time to first call, first-call success rate, and the error mix in week one. If the header error drops and a different code rises to the top, the work is not finished, it has moved.

### 19e. Fade the scaffold

**Faded example 1.** Last block blanked. A partner integration surface where every partner must be contractually approved before receiving production credentials, so instant self-service is impossible.

- Block 1: split the funnel, and accept that the approval segment is fixed by legal process rather than by engineering.
- Block 2: since the step cannot be removed, make it parallel rather than sequential.
- Block 3, what "parallel" means concretely: **you write this**.

??? note "Show answer"

    Give the developer a fully functional sandbox on day zero, with no approval at all, and run the contractual approval concurrently with their build. The approval then gates the promotion of a working integration rather than the start of one, which converts a blocking wait into a background process.

    Three supporting details. First, sandbox and production must be identical in contract, including error codes and event shapes, or the parallel work is partly wasted and the developer discovers it at the worst moment. Second, expose the approval as a status resource with stages and a named owner, because an opaque wait produces a support ticket every two days per partner. Third, make promotion a configuration change rather than a rewrite: same code, different credential, so the only thing the approval gates is a key.

**Faded example 2.** Last two blocks blanked. An internal platform interface used by 60 teams, where onboarding a new team takes about two days of a platform engineer's time.

- Block 1: split the two days. Suppose it is 3 hours of access provisioning, 4 hours of questions answered in chat, and the rest waiting.
- Blocks 2 and 3: **you write these**.

??? note "Show answer"

    **Block 2, the removal.** The 4 hours in chat is the metric to attack, and it is a documentation and error-message problem wearing a support costume. Log the questions for a month and count them: a handful of questions will make up most of the volume, and each one is either a missing example or an error message that does not say what to do. Fix those and the chat time falls without anyone writing a guide nobody reads.

    Access provisioning becomes self-service with a policy check, and the waiting is usually a queue for a human, which is removed by making the common case require no human.

    **Block 3, the internal specifics.** Two things differ from an external surface. First, you know every consumer by name, so the equivalent of a funnel metric is a per-team onboarding time you can actually ask about. Second, your leverage is different: you can ship a shared client library into the internal build system and get near-total adoption, which no public surface can do. Use that leverage deliberately, and measure library adoption as your migration capability, because it is what makes future changes cheap.

### 19f. Check yourself

**Q1.** Your library adds automatic retries with backoff. Name the operation class this makes dangerous by default, and the design that makes it safe.

??? note "Show answer"

    Unsafe, non-idempotent writes. A library that retries a payment creation on a timeout will create two payments, and the developer never wrote the retry so they will never suspect it. Automatic retries turn a client convenience into a correctness hazard precisely where correctness matters most.

    The safe design has three parts.

    1. Retry only where the semantics permit it: safe and idempotent methods always, unsafe methods only when an idempotency key is present.
    2. Generate that key automatically in the library for every unsafe request, so the safe path is the default path and the developer gets convergence without knowing the term.
    3. Honour the server's retryability signal from the error model rather than inferring it from the status code, since only the server knows whether a given 500 landed before or after the side effect.

**Q2.** You are asked to support six languages with hand-written libraries. Give the argument for cutting that to two, in terms an interviewer will accept.

??? note "Show answer"

    Cost per language is not the build, it is the perpetual maintenance: every contract change, every deprecation, every security fix, multiplied by six, staffed by people who are usually expert in one of the six. The predictable outcome is two good libraries and four that drift, and a drifted library is worse than none because developers trust it.

    The argument that lands: measure the language split of your actual traffic. If two languages cover 80 percent of calls, staff those two properly and generate the rest from the schema, clearly labelled as generated, with the schema published so anyone can generate their own. You spend the saved effort on the thing that helps all six, which is the contract itself: accurate schemas, honest errors, and a sandbox with deterministic failure triggers.

    The number to state: the migration capability from earlier. Two well-adopted libraries covering 80 percent of traffic buy you more future flexibility than six neglected ones covering 95.

### 19g. When not to use this

| Question | Answer |
|---|---|
| The measurement that justifies investing in developer experience | Number of integrators you do not employ, multiplied by the support cost per integration, plus the share of your roadmap blocked by clients who cannot migrate |
| The threshold below which it is over-engineering | An internal surface with a handful of consuming teams you can walk over to. There, a sandbox environment, a key rotation console and a published funnel metric are ceremony |
| What to do instead | A working example in the repository, an accurate schema, and error messages that name the fix. Those three cover most of the value at a fraction of the cost |

**A separate one for client libraries.**

| Question | Answer |
|---|---|
| The measurement that justifies a hand-written library | External integrators and their language distribution, plus your expected rate of breaking changes |
| The threshold below which it is over-engineering | Under roughly 20 external integrators concentrated in one or two languages, on a surface you expect to change rarely |
| What to do instead | Publish the schema, generate clients, and maintain three or four copy-paste examples per language covering auth, pagination, retries and webhook verification |
---

## Module 20. Operability and resilience budgets for a contract

**Time: 24 to 30 minutes reading.**

A contract is not only a set of shapes. It is a set of promises about time and failure, and those promises have to be measurable by you and derivable by the client. This module is about the indicators a contract needs, which are not the ones a service dashboard usually carries, and about the budgets that make timeouts, retries and shedding agree with each other instead of fighting.

### 20a. Predict first

```
+--------------------------------------------------------------+
| SERVICE DASHBOARD, LAST 30 DAYS                              |
|                                                              |
|   overall availability            99.97 %                    |
|   overall p99 latency               240 ms                   |
|                                                              |
| TRAFFIC MIX                    ERROR RATE BY OPERATION       |
|   GET  /invoices    98 %         GET  /invoices    0.01 %    |
|   POST /payments     2 %         POST /payments    0.80 %    |
+--------------------------------------------------------------+
Caption: an aggregate availability figure alongside the traffic
mix and the per-operation error rates that produce it.
```

**Question.** Reconstruct the 99.97 percent from the mix, then say what a merchant taking 500 payments a day actually experiences, and what the aggregate figure is therefore useless for.

??? note "Show answer"

    Reconstruction: 0.98 times 99.99 percent plus 0.02 times 99.20 percent gives 0.97990 plus 0.01984, which is 0.99974, so 99.97 percent. The aggregate is dominated by the operation that matters least, because availability weighted by traffic is a measure of your read path wearing the whole service's name.

    The merchant's experience: at a 0.8 percent failure rate on payment creation, 500 payments a day produces about 4 failures a day, every day. If those failures are timeouts on a non-idempotent write, each one is also a support conversation about whether the customer was charged.

    What the aggregate is useless for: every decision you would want to make. It cannot tell you whether to page someone, cannot support a per-operation promise to a client, and cannot be compared against any objective a customer cares about. A contract needs per-operation indicators, because a contract is per operation.

### 20b. The indicators a contract needs

Three terms, defined before use. A **service level indicator** is a measurement, for example the fraction of payment creations that succeed. A **service level objective** is a target for that indicator over a window, for example 99.9 percent over 30 days. An **error budget** is the allowed shortfall, which for 99.9 percent over 30 days is 0.1 percent of 43,200 minutes, so 43.2 minutes.

**Per operation, not per service.** Every operation on a public surface gets its own row, and the rows differ in what they even measure.

| Operation class | Availability indicator | Latency indicator | Correctness indicator |
|---|---|---|---|
| Read of a single resource | Non-5xx share | p99 server time | Staleness against the write that preceded it |
| List or search | Non-5xx share | p99, split by page depth | Pagination stability: duplicate or skipped rows per thousand pages |
| Unsafe write | Non-5xx share | p99 to the durable commit | Duplicate creations per million requests |
| Long-running operation | Share reaching a terminal state | Time to terminal state at p95 | Operations stuck past their lifetime cap |
| Outbound event | Share delivered within a stated window | Time from state change to first successful delivery | Events in the dead letter store |

**The five indicators that only exist because you shipped a contract.** These are what separate an operable interface from an operable service, and none of them appears on a default dashboard.

| Indicator | What it catches | Why it belongs to the contract |
|---|---|---|
| 4xx rate by code, by client | One client's 400 spike, which is your bug more often than theirs | A validation rule that surprises a client is a design defect, and this is the only place it shows |
| Schema validation failure rate | A shape mismatch after a release, on either side | Detects breaking changes you did not know you shipped |
| Version and deprecated-surface mix, by client | Who is still on the old thing | This is the measurement that ends a deprecation window, from Module 11 |
| Idempotency replay rate | How often clients are retrying, which is a proxy for how often you time out | A rising replay rate is an early warning that precedes any error rate change |
| Quota rejection rate by tenant | Whether your limits are policy or accident | A tenant hitting limits daily is either mispriced or misdesigned, and both are product problems |

**Correlation, and the support workflow it enables.** Assign a request identifier at the edge if the client did not supply one, echo it in every response including every error body, and log it on every hop. Propagate standard trace context so a single call can be followed across services.

The reason to insist on it in an interview is concrete: a support ticket that contains an identifier can be answered in minutes by anyone, and a ticket that says "it failed around 3pm" costs an engineer an hour. Put the identifier in the error body, not only in a header, because the client's log line usually captures the body and drops the headers.

### 20c. Budgets: deadlines, retries and what you shed first

**Deadline propagation, derived.** Assume a client budget of 800 ms end to end, 60 ms of network round trip, dependency A at p99 120 ms, dependency B at p99 300 ms, and 100 ms of your own work.

- Your server-side budget: 800 minus 60, which is 740 ms.
- Expected consumption: 120 plus 300 plus 100, which is 520 ms, leaving 220 ms of slack.
- Now the case that matters. If A is slow and takes 500 ms, the remaining budget is 740 minus 500 minus 100, which is 140 ms. B needs 300 ms at p99. Starting B is a decision to blow the budget and to spend B's capacity on a request nobody will wait for.

So the rule: pass a remaining-deadline value to every dependency, and refuse to start a call whose typical duration exceeds what is left. Return a deadline-exceeded error immediately instead. This is worth saying precisely, because "we set timeouts" and "we propagate deadlines" are different designs, and only the second stops work that is already doomed.

**Retry amplification, derived.** Assume a partial outage in which 30 percent of requests fail, and every client retries three times.

- Load multiplier: 0.7 for the successes, plus 0.3 times 4 attempts, which is 1.2, giving 1.9 times normal load.
- If the outage was caused by overload, that 1.9x guarantees it persists. This is the metastable failure class from the hub's failure library: the system stays down after the original trigger is gone, because the retries are now the load.

The repair is a retry budget rather than a retry count. Cap retries at, say, 10 percent of the successful request count over a rolling window, measured client side by the library and server side as a check. Then the worst-case amplification is 1.1x rather than 4x, and a client that is failing everything stops retrying altogether, which is exactly the desired behaviour.

Three supporting rules worth stating: jitter every backoff, never retry a non-idempotent write without an idempotency key, and honour the server's retryability signal instead of guessing from the status code.

**Shedding order is part of the contract.** When you must drop work, drop it in a published order, because an unpublished order means every client believes it is in the protected class.

| Class | Examples | Shed at |
|---|---|---|
| Money and safety critical | Payment creation, refunds, authorisation checks | Last, and only with an explicit incident decision |
| Interactive reads | A user is waiting on a screen | Third |
| Background and bulk | Exports, imports, analytics ingestion | Second |
| Optional enrichment | Recommendations, previews, non-essential joins | First, silently, with a degraded response |

Publishing this converts an incident from a mystery into an expectation. It also forces a useful design conversation, because a surface where everything is class one has no shedding strategy at all.

**Alerting on burn rate, derived.** Take a 99.9 percent monthly objective on payment creation. The budget is 0.1 percent of 43,200 minutes, which is 43.2 minutes of failure for the month.

| Alert | Condition | Implied error rate | Meaning |
|---|---|---|---|
| Fast burn | 2 percent of the budget in 1 hour | 0.864 minutes of failure in 60, so about 1.44 percent | Something broke now. Page someone |
| Slow burn | 5 percent of the budget in 6 hours | 2.16 minutes in 360, so about 0.6 percent | Something is degrading. Open a ticket, do not page |

Two alerts, both derived from one objective, and neither is a threshold somebody picked. That derivation, done aloud in about forty seconds, is one of the highest-value things you can do in the last ten minutes of a round.

### 20d. Worked example: making a payments contract operable

The prompt: we have the payments interface from earlier in this course. Make it operable.

**Block 1. Purpose: pick the indicators the contract implies, not the ones the dashboard already has.**

Payment creation: availability as non-5xx share, latency as p99 to durable commit, and correctness as duplicate creations per million. Payment reads: availability, p99, and staleness against the creating write. Webhook delivery: share delivered within 60 seconds and count in dead letter.

The duplicate-creation indicator is the one that separates this answer. Availability and latency are generic; duplicates per million is a measurement of whether the idempotency design from Module 9 is actually working, and it is the number a payments team would ask about first.

The error most readers make here: naming the golden signals and stopping. Every service has traffic, errors, latency and saturation. A contract additionally has correctness properties that only its semantics define, and those are the interesting rows.

!!! question "Say it before you read on"

    Before the next block: say how you would measure duplicate creations per million without a manual audit, using only things the design already stores.

**Block 2. Purpose: set objectives that are derivable from what clients actually need.**

I derive rather than assert. A merchant taking 500 payments a day at a 99.9 percent objective sees an expected 0.5 failures a day, which is roughly one every two days: tolerable if each failure is safely retryable, and not otherwise. That makes the objective conditional on the semantics, which is the point: 99.9 percent with idempotency keys is a much better product than 99.95 percent without them, because the failures are recoverable by the client rather than by a phone call.

So: 99.9 percent monthly on creation, 99.95 percent on reads (cheaper to hold, and reads carry the interactive traffic), and 99 percent of webhooks delivered within 60 seconds. Then the burn-rate alerts follow from the arithmetic in the previous section, without a second discussion.

**Block 3. Purpose: make the budgets agree with each other.**

Client budget 800 ms, so the server budget is 740 ms after network. Deadlines are propagated to the ledger write and to the gateway call, and a call is not started if the remaining budget is under its p99.

Client libraries carry a retry budget of 10 percent of successful requests, so amplification is capped at 1.1x, and every retry carries the idempotency key generated by the library. The server publishes retryability per error code so the library is not guessing.

Shedding order is published: enrichment first, exports second, reads third, payment creation last. The publication matters as much as the mechanism, because an integrator planning a sale day needs to know where they sit.

!!! question "Say it before you read on"

    Before the next block: the retry budget in block 3 caps amplification at 1.1x. Say what the amplification would be with the same failure rate and an uncapped three-retry policy, and why that difference decides whether an incident ends.

**Block 4. Purpose: make one call traceable end to end, because most incidents are one client's problem.**

Every response carries a request identifier, echoed in error bodies. The identifier is stored with the payment, with the idempotency key record, and with every webhook attempt, so a support engineer can go from a merchant's log line to the ledger entry and to the delivery history in one query.

Then the indicator that closes the loop: 4xx rate by code by merchant, alerted per merchant rather than in aggregate. One merchant going from 0 to 400 validation failures an hour is invisible in an aggregate that serves millions of calls, and it is almost always either your release or their release. Catching it before they open a ticket is what an operable contract feels like from the outside.

### 20e. Fade the scaffold

**Faded example 1.** Last block blanked. A search interface with a 99.5 percent monthly objective and a 400 ms p99 target.

- Block 1, indicators: availability, p99 by page depth, and result-set stability under concurrent writes.
- Block 2, objectives: 99.5 percent monthly gives a budget of 0.5 percent of 43,200 minutes, which is 216 minutes.
- Block 3, budgets: a 400 ms p99 target with a 250 ms index call leaves 150 ms, so no serial second index call is affordable.
- Block 4, the alerting derived from that budget: **you write this**.

??? note "Show answer"

    Budget: 216 minutes for the month. Fast burn at 2 percent of the budget in one hour is 4.32 minutes of failure in 60, which is about a 7.2 percent error rate: page. Slow burn at 5 percent in six hours is 10.8 minutes in 360, which is about 3 percent: ticket.

    Note what the looser objective did to the alert thresholds. At 99.9 percent the fast-burn page fired at 1.44 percent errors; at 99.5 percent it fires at 7.2 percent. Same method, five times the tolerance, and no argument about the number, which is the whole reason to derive thresholds from an objective rather than to choose them.

    Second half, specific to search: latency is the failure mode here more than errors are, so add a latency-based indicator to the budget, for example the share of requests served under 400 ms, and burn budget on slow requests as well as failed ones. A search that answers in 4 seconds has failed even though it returned 200.

**Faded example 2.** Last two blocks blanked. An outbound event system with 800 consumers, some of which are frequently unavailable.

- Block 1, indicators: share of events delivered within 60 seconds, relay lag, dead letter count, and per-destination success rate.
- Blocks 2, 3 and 4: **you write these**.

??? note "Show answer"

    **Block 2, objectives.** The interesting decision is whose failures count against your budget. A consumer that is down is not your outage, so the objective must be defined on what you control: events delivered within 60 seconds to endpoints that are answering. Track consumer-caused failures separately and never mix them into your own indicator, or your budget becomes a measure of your customers' uptime and you will page on their problems.

    **Block 3, budgets.** Relay lag gets a hard target, for example under 5 seconds at p99, because it is the part nobody else can see and the failure is silent. Per-destination concurrency caps and drain rates come from Module 15's arithmetic. The retry ladder is itself a budget: a fixed number of attempts over a stated horizon, after which the event dead letters rather than retrying forever.

    **Block 4, what the consumer can see.** The operability surface has two audiences, and the second one is usually forgotten. Consumers need their own success rate, their own attempt log and their own dead letter list, because most incidents in an event system are one consumer's problem and they are the only party who can fix it. Publishing per-destination health to the destination converts your on-call load into their self-service, which is the highest-leverage operability decision in the module.

### 20f. Check yourself

**Q1.** Your objective is 99.95 percent monthly. Derive the error budget in minutes and both burn-rate alert thresholds, showing the arithmetic.

??? note "Show answer"

    Thirty days is 43,200 minutes. A 99.95 percent objective allows 0.05 percent, which is 21.6 minutes of failure for the month.

    Fast burn, 2 percent of the budget in one hour: 0.02 times 21.6 is 0.432 minutes of failure inside 60 minutes, which is an error rate of about 0.72 percent. That is the page.

    Slow burn, 5 percent in six hours: 0.05 times 21.6 is 1.08 minutes inside 360, which is an error rate of about 0.3 percent. That is a ticket.

    The general form worth remembering rather than the numbers: threshold error rate equals the budget fraction times the window's allowed minutes divided by the window length. Say the form, then plug in, and the interviewer sees a method rather than a memorised pair of numbers.

**Q2.** A client asks you to raise your timeout from 2 seconds to 30 because their requests keep failing. Give the answer that is not simply yes or no.

??? note "Show answer"

    Ask what is actually slow, because a 30 second timeout is a request for a different design in disguise. If their calls take 20 seconds because they are fetching 10,000 records a page, the fix is pagination or an operation resource, not a longer timeout. A longer timeout converts a fast failure into a slow one and holds a worker for 30 seconds while doing it, which is capacity taken from every other client.

    The answer with a number in it: at 2 seconds and, say, 200 workers, a fully saturated pool recovers in 2 seconds. At 30 seconds it recovers in 30, so one client's slow calls can occupy your entire capacity for half a minute. That is the cost you are being asked to absorb, and it belongs in the conversation.

    Then offer the real fix: an asynchronous operation for the heavy case, a smaller page for the interactive case, and a deadline the client can pass so they choose their own trade between waiting and failing fast.

### 20g. When not to use this

**Full distributed tracing and multi-window burn-rate alerting.**

| Question | Answer |
|---|---|
| The measurement that justifies it | Request rate on the operation, and the number of network hops behind a single call |
| The threshold below which it is over-engineering | Below roughly 1 request per second on an operation, percentile alerts are noise: an hour holds 3,600 requests, so p99 is 36 samples and a handful of slow calls moves it wildly. Below three hops, a trace tells you what a log line already told you |
| What to do instead | Log-based counters per operation, one availability alert with a generous window, and a weekly review of the error mix by client |

**Per-operation objectives.**

| Question | Answer |
|---|---|
| The measurement that justifies them | Whether different operations have materially different failure consequences for the client, and whether anyone will act differently on the two numbers |
| The threshold below which it is over-engineering | A surface with four operations that all do the same kind of work for the same client. One objective is enough, and four will produce four dashboards nobody reads |
| What to do instead | One objective, plus a per-operation error-rate breakdown you look at during incidents. Add an objective the first time someone wants to make a promise about one operation specifically |

---

## Module 21. Level calibration: mid, senior and staff on one prompt

**Time: 18 to 22 minutes reading.**

Nobody is told the rubric. The comments that come back are things like "solid but not senior" and "good breadth, thin depth", and neither says what to change. This module makes the difference concrete enough to edit your own answer against.

### 21a. Predict first

Three candidates answered the same prompt, "design a webhook system so merchants learn when a payment is captured". One sentence each, taken from the middle of the answer.

```
+--------------------------------------------------------------+
| A. "I will POST the event to the merchant's URL, and retry    |
|     with exponential backoff if it fails."                    |
|                                                               |
| B. "I will commit an outbox row in the same transaction as    |
|     the payment, so a crash cannot lose the event, and a      |
|     relay delivers it at least once with a signed payload."   |
|                                                               |
| C. "Before mechanism: the guarantee is at least once with     |
|     per-payment ordering, and that choice makes merchant      |
|     deduplication mandatory, which changes what we document,  |
|     what we support, and what breaks when we change the       |
|     payload shape in two years."                              |
+--------------------------------------------------------------+
Caption: three single sentences from three answers to the same
webhook design prompt.
```

**Question.** Rank these by level, then state the specific thing each one adds over the one below it. Do not say "more detail".

??? note "Show answer"

    A is mid, B is senior, C is staff, and the additions are specific.

    B adds over A: correctness under crash. A describes the happy path with a retry loop, which fails exactly as Module 15's pre-question showed. B names a failure mode that A's design cannot survive and repairs it structurally rather than by turning a dial.

    C adds over B: consequences beyond the system. C states the guarantee first, then follows it out into the client's obligations, the support cost and the evolution cost. The tell is the phrase "in two years": C is reasoning about a second-order cost that the design decision creates for other people over time.

    What none of them is: C is not more senior because it used bigger words or mentioned the organisation. It is more senior because it converted one technical choice into its consequences for parties who are not in the room, and those consequences are checkable.

### 21b. Three axes that actually separate levels

Not "depth". Three specific axes, and you can self-score on each.

| Axis | Mid | Senior | Staff |
|---|---|---|---|
| Whose problem you solve | The prompt as literally stated | The prompt plus the client's experience of it | The prompt plus the client plus the cost to the organisation over years |
| What you optimise across | Correctness on the happy path | Correctness plus failure behaviour plus operations | All of that plus migration cost, support cost and money |
| What you do when contradicted | Defend, or capitulate instantly | Ask what case they have in mind, then update the design visibly | Name the assumption that differed, restate the trade, and say which decision now flips |

The third row is the one candidates never rehearse and interviewers weight heavily. Being contradicted is a designed part of the round, and the response is scored.

**The five additions that reliably move an answer up a level.** Each is one or two sentences, and each is checkable rather than atmospheric.

| Addition | The sentence, roughly | What it demonstrates |
|---|---|---|
| A deprecation path | "This shape will need to change within two years, so I am adding fields rather than versioning, and here is the measurement that would end the old surface" | You have owned something you could not change on Monday |
| A quota and metering consequence | "This operation costs about fifty times a read, so it is metered in cost units, which also decides how the free tier is sized" | You connect design to the bill |
| A delivery guarantee, stated as a client obligation | "At least once with per-object versioning, which means consumers must deduplicate, so it goes in the documentation and the client library" | You know a guarantee is a promise to someone else, not a property of your code |
| A blast radius statement | "If this dependency is down, payments still succeed and events are delayed; if the ledger is down, we reject rather than accept, and here is why that direction is correct" | You have been on call |
| An explicit skip | "I am not covering the storage engine, because every option holds the same rows and cannot change the contract" | You are prioritising rather than omitting |

**Down-levelling tells, which are cheap to remove.**

| Tell | What the interviewer concludes | Replace with |
|---|---|---|
| Naming a technology before naming a requirement | Pattern matching rather than designing | "The requirement is X, which needs a component that does Y. Options are these two" |
| Drawing boxes before stating the domain model | Guessing endpoints, which this round scores directly | Entities, invariants, then the surface derived from them |
| Error handling arriving at minute 40, if at all | Semantics treated as a detail | Errors in the same breath as the operation they belong to |
| "It depends" with no branch | Avoiding commitment | "It depends on whether clients are internal. If internal, A, because the coupling is cheap. If external, B" |
| Staff signalling without substance: alignment, stakeholders, org processes | Seniority theatre | A migration cost, a support cost, or a number |

That last row deserves emphasis. Talking about influence and organisations does not read as staff in a design round. Naming a five-year cost that the current design creates does.

### 21c. The same prompt at three levels, side by side

Prompt: "Design the interface for a feature flag service."

| Dimension | Mid | Senior | Staff |
|---|---|---|---|
| Model | Flags with values, an endpoint to read them | Flags, environments, targeting rules and an evaluation result as distinct concepts | Adds the invariant that evaluation must be deterministic given a flag version and a context, which is what makes debugging possible at all |
| Surface | `GET /flags` returning all flags | Bulk evaluation for a context, plus a streamed change channel | Evaluation on the client from a downloaded ruleset, because the network is the availability risk, with the trade stated |
| Errors | 404 and 500 | Taxonomy with retryability | The flag service failing must never fail the caller: a stale or default value is the contract, and the error model says so explicitly |
| Staleness | Not mentioned | A polling interval | A stated staleness budget per flag class, with kill switches on a shorter budget than experiments |
| Evolution | Not mentioned | Additive fields | Rule schema evolution: old clients must evaluate new rule types safely, which forces a default-deny or default-ignore decision now |
| Quotas | Not mentioned | Rate limit per key | Evaluation is free and unmetered by design, because a metered flag check makes teams cache badly and the correctness cost exceeds the infrastructure saving |
| Operability | Logs | Availability and latency indicators | Indicator on evaluation disagreement between client and server, which is the only way to catch a rule engine drifting between versions |
| Blast radius | Not mentioned | The service going down | Names that a bad rule push is a bigger risk than downtime, and adds staged rollout of rules plus an instant revert as part of the interface |

Read the staff column carefully: almost every cell is about a failure that happens after launch, to someone else, and can only be prevented by a decision made now. That is the pattern to reproduce.

### 21d. Worked example: editing one answer up two levels

Here is a mid-level answer to the payments webhook prompt, then the edits. This is the exercise to run on your own recorded mocks.

**Block 1. Purpose: capture the mid answer exactly, so the edits are visible.**

"When a payment is captured, we POST a JSON body with the payment details to the merchant's configured URL. If it fails we retry three times with exponential backoff. We use HTTPS so it is secure. If it still fails we log an error."

The error most readers make when self-editing: rewriting from scratch. The point of the exercise is to see which specific additions change the level, and that is only visible if the base survives.

**Block 2. Purpose: add what makes it senior, which is correctness under failure.**

Three edits, each attached to a failure the mid answer cannot survive:

1. Outbox in the same transaction, because a crash between commit and POST loses the event permanently. This one edit is the difference between a design that works and one that works when nothing goes wrong.
2. A published guarantee, at least once with a stable event identifier, plus per-payment versioning, because retries reorder events by construction.
3. Signature over the timestamp and raw body, with a tolerance window, because "we use HTTPS" authenticates your server to the merchant and does nothing to authenticate the delivery to them.

Also replaced: "retry three times" becomes a derived ladder covering both the restart cluster and the business-day cluster, and "log an error" becomes a dead letter store with replay.

!!! question "Say it before you read on"

    Before the next block: of those three edits, say which one an interviewer is most likely to probe with "what if the relay crashes after delivering", and what your answer is.

**Block 3. Purpose: add what makes it staff, which is consequences over time and money.**

Four further edits, matching the five additions from the table above:

1. Evolution: "The payload will need new fields, so it carries a minimal body plus a fetch, and the event type namespace is designed for additive growth. If we ever must remove a field, here is the traffic measurement that would end the old shape."
2. Metering and quotas: "Delivery costs us per attempt, and a merchant that returns 500 for a day costs 10 attempts per event. Per-destination drain caps bound that, and endpoints failing for 24 hours are disabled with published rules."
3. Guarantee as client obligation: "At least once means merchants must deduplicate, so the identifier is stable, the documentation states the algorithm in four lines, and the client library implements it so most merchants never think about it."
4. Blast radius: "If the relay is down, payments still succeed and events are delayed, which is the correct direction. If we inverted it and blocked payments on delivery, one merchant's outage would stop our revenue. I would rather explain a delay than an outage."

Then the closing sentence that is worth its length: "The thing I would measure first is delivery latency at p99 per destination, because a stalled relay is silent and merchants will not tell us for hours."

!!! question "Say it before you read on"

    Before the next block: count the boxes added between the mid version and the staff version. Say what that count implies about how to spend your last four minutes in a round.

**Block 4. Purpose: name what did not change, which is how you avoid mistaking length for level.**

The core design is the same in all three versions: an event goes to a merchant's URL. No box was added for seniority. What changed was the set of futures the answer accounts for, and every addition is checkable by an interviewer. That is the honest definition of the level difference, and it is why padding an answer with components moves it down rather than up.

### 21e. Fade the scaffold

**Faded example 1.** Last block blanked. A mid-level answer to "design pagination for a comments feed": "We use `?page=2&size=20` and return the items plus a total count."

- Block 1: base captured.
- Block 2, the senior edits: keyset or cursor pagination because offset skips and duplicates rows under concurrent inserts and deletes, a stable sort key including a tiebreaker, and dropping or bounding the total count because it is a full scan on a large collection.
- Block 3, the staff edits: **you write these**.

??? note "Show answer"

    Four edits, one per axis.

    Evolution: the cursor is opaque and versioned, so its internal encoding can change without breaking clients, and clients are told in writing never to parse or construct one. That single decision is what lets the sort key change in two years without a migration.

    Cost and metering: deep pagination is priced. State the depth at which it stops being defensible, and either cap it or bill it in cost units, because an unbounded page parameter is an unmetered full scan anyone can trigger.

    Client obligation: state the consistency guarantee. A cursor gives stability relative to the snapshot it encodes, so clients paging over a mutating collection may miss items added before their cursor, and they must be told that rather than discovering it in a bug report.

    Blast radius: a hot comment thread is a hot key. Name it, and say what degrades: the feed serves a slightly stale cached page rather than failing, and the fallback is part of the contract rather than an incident improvisation.

**Faded example 2.** Last two blocks blanked. A mid answer to "design the error model for a bulk import endpoint": "We return 400 with a message saying what went wrong."

- Block 1: base captured.
- Blocks 2 and 3: **you write these**.

??? note "Show answer"

    **Block 2, senior edits.** Per-item results rather than one status, each with a machine-readable code, the offending field, and a retryable flag, because a single 400 for 100 items tells the client nothing it can act on. An overall status that means "the request was well formed" and never doubles as an item verdict. A documented, closed set of codes, because clients branch on codes and a code set that grows silently is a breaking change nobody announced.

    **Block 3, staff edits.** Four, one per axis.

    - Evolution: adding an error code is a compatible change only if clients were told to treat unknown codes as a default branch, so that sentence goes into the published compatibility rules today, not when it first bites.
    - Cost and support: instrument the code mix per client. One dominant code across many clients is a design defect in your validation, and it is cheaper to fix the contract than to answer the tickets.
    - Client obligation: state whether a retry of the whole batch is safe. It is only safe if items carry client references, so the reference becomes required rather than optional.
    - Blast radius: an import that half applies is a data problem for a human, so cap the batch size by the size of the mess a partial application creates, not only by the latency budget.

### 21f. Check yourself

**Q1.** An interviewer says "I would not do it that way" about your choice of cursor pagination. Give the mid, senior and staff responses.

??? note "Show answer"

    Mid: defends the choice, or abandons it immediately. Both read the same way, as an inability to reason under challenge rather than as conviction or flexibility.

    Senior: "What case do you have in mind?" then updates the design visibly. "If your clients need to jump to page 40, keyset cannot do that, and I would add a bounded offset path for that case with a documented depth limit."

    Staff: names the assumption that differed and says which decisions flip. "I assumed clients read sequentially and the collection changes while they read. If clients need random access into a stable snapshot, then the shape changes: I would materialise a snapshot with an identifier, page by offset within it, and set an expiry. That also changes the storage cost and gives me a new thing to meter, so it is a real trade rather than a parameter change."

    The pattern across the three: mid argues about the answer, senior asks about the case, staff surfaces the assumption and prices the alternative.

**Q2.** You have four minutes left and your answer so far is competent but reads as mid. Name the two additions with the best return, and say why those two rather than more components.

??? note "Show answer"

    First, the evolution and deprecation sentence: how this contract changes in two years, with a measurement that ends the old surface rather than a date. It takes about forty seconds and it is the single most reliable senior signal in this round, because it is the thing you can only know from having owned an interface you could not change.

    Second, the blast radius plus the first measurement: what happens to callers when your main dependency is down, which direction you fail in, and the one indicator you would watch on launch day. Another forty seconds, and it demonstrates operational experience without a war story.

    Why not more components: a component adds surface for the interviewer to probe and takes minutes to draw, and an unprobed box scores nothing. Both of these additions are pure signal about how you think beyond the launch, which is exactly the axis the level distinction is drawn on. Adding a cache in the last four minutes adds a box; adding a deprecation measurement adds a level.

### 21g. When not to use this

| Question | Answer |
|---|---|
| The measurement that justifies aiming a level above your own | Whether you can hold two consecutive follow-ups on any topic you raise. Test it before the round: have someone ask "why" twice about each staff-level addition you plan to make |
| The threshold below which it is over-engineering your own answer | If you cannot answer the second follow-up, raising the topic costs more than skipping it. A deprecation plan you cannot defend reads worse than a clean answer that never mentions one |
| What to do instead | Depth in one area you genuinely own, plus the explicit skip: "I would also want a deprecation plan here, and I have not run one on a public surface, so I will not pretend otherwise" |

That last sentence is not a weakness. Interviewers hear invented depth constantly and it is easy to detect, whereas a clean boundary on your own experience plus real depth elsewhere is both rare and credible.

Two more situations where this module's framing should be set aside:

- You are interviewing for the level you already hold and the round is a confirmation rather than a stretch. Optimise for a complete, clean answer with stated trade-offs, not for reaching upward.
- The interviewer has explicitly asked for breadth, for example "walk me through everything you would consider". Depth in one area against that instruction reads as not listening, which is a delivery failure and it is scored as one.

## Case study 1: Payments interface

Most treatments of this prompt draw a box labelled "payment service" and move on. The interesting part is that money makes retries dangerous, and retries are the one thing you cannot stop a client doing. This case study spends its depth on the retry contract and on how the merchant finds out what happened.

### The prompt (payments)

"Design the public interface a merchant integrates to charge a customer's card, learn when the money actually moved, and issue refunds."

### Questions that fork the design (payments)

Ask these because each answer moves a box or changes a resource. Do not recite them.

| Question worth asking | If the answer is A the design becomes | If the answer is B it becomes |
|---|---|---|
| Do you hold the funds, or route to a third-party card gateway? | Route: your interface is a facade, the gateway's latency and outage become yours, and every charge needs a non-terminal state | Hold: you are the ledger of record, and settlement, reconciliation and regulated reporting enter scope |
| Is authorisation separate from capture? | Single step: one charge resource, one terminal transition, a small state machine | Two step: authorise then capture, so you need auth expiry, partial capture, void, and a released-funds event |
| Does the buyer's browser send card data to you, or does the merchant's server? | Merchant server: secret keys, server-to-server only, and card data is in the merchant's compliance scope | Browser: you need a publishable key, a tokenisation endpoint on your own origin, and the card never touches the merchant |
| Single currency or many? | Single: an amount is one integer in minor units | Many: every amount carries a currency, exchange becomes a quoted resource with an expiry, and rounding rules are part of the contract |
| Must the merchant get the outcome in the response? | Synchronous: you inherit the gateway's tail latency in your own p99 and your timeouts become the merchant's timeouts | Asynchronous: charge returns a non-terminal resource, and webhooks stop being a nice-to-have and become load-bearing |
| Are partial and repeated refunds allowed? | Full refund only: refund is a transition on the charge | Partial and repeated: refund is its own collection, with its own retry key and a remaining-refundable invariant to defend |
| How long must one request stay safely retryable? | Minutes: retry keys can live in a cache with a short lifetime | A day or more: retry keys are durable records with retention, storage cost and their own expiry semantics |
| Do merchants reconcile against your data or the gateway's? | Yours: you owe them an immutable, exportable ledger view | The gateway's: you owe them the gateway's identifiers on every object, which constrains your resource shape |

### Requirements (payments)

Functional:

- Create a payment intent for an amount, currency and payment method, carrying a client-supplied retry key.
- Read a payment intent and its ordered history of state transitions.
- Cancel an intent before capture, and capture an authorised intent.
- Create refunds against a captured intent, fully or partially, more than once.
- Deliver state changes to a merchant-supplied endpoint, and let a merchant read missed changes without contacting support.
- List intents and refunds with a cursor that survives concurrent writes.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Charge attempts, average | 150 per second | ASSUMED, a mid-size platform; the interviewer usually supplies this, so ask before assuming |
| Peak multiplier | 3x, so 450 per second | ASSUMED, a single sale day concentrates a day of traffic into a few hours |
| Gateway response time, p99 | 2.5 seconds | ASSUMED, card authorisation crosses several networks you do not control; treat it as an input, not a target |
| Our own overhead, p99 | under 150 ms excluding gateway time | ASSUMED, measured as total span minus the downstream span, so the number is attributable |
| Accept-path availability, monthly | 99.95 percent | ASSUMED, and deliberately set on accepting the intent, not on the money moving |
| Retry key lifetime | 24 hours | ASSUMED, because merchant job schedulers commonly retry on a daily cycle |
| Event delivery | 99.9 percent within 60 seconds, retried up to 72 hours | ASSUMED, 72 hours covers a weekend outage on the merchant side |
| Refund window | 120 days | ASSUMED, card scheme rules keep disputes open for months, so charges stay queryable |
| Contract stability | no breaking change to a released version for 24 months | GIVEN, merchants ship on quarterly cycles and some never upgrade |
| Charge attempts per day | 13.0 million | DERIVED below |
| Requests in flight at peak | 1,125 | DERIVED below |
| Ledger rows per year | 19.0 billion | DERIVED below |

### Estimation that eliminates (payments)

Every line ends with the option it kills. A number that kills nothing is deleted from the answer.

| Calculation | Result | What it rules out |
|---|---|---|
| 150 per second times 86,400 seconds | 13.0 million charge attempts per day, 450 per second at the 3x peak | Rules out sharding the intent store, rules out a write queue in front of the database, and rules out any conversation about write throughput. One primary absorbs 450 inserts per second |
| 450 per second times 2.5 seconds of gateway time | 1,125 requests in flight at peak | Rules out a thread-per-request server with a few hundred workers, and rules out promising a terminal outcome inside the response. The connection pool, not the CPU, is the scarce resource |
| 13.0 million times 1 KB per retry record | 13 GB per day; 23.7 TB if kept for five years | Rules out an in-process map for retry keys, and rules out unbounded retention of records that nobody reads after a day |
| 13.0 million times 3 lifecycle events each | 39 million events per day, 451 per second average, 1,353 per second at peak | Rules out delivering events inline on the request path, which would add every merchant's endpoint latency to your own |
| One merchant holding 5 percent of volume, endpoint down 24 hours | 1.95 million queued events for that merchant alone | Rules out a single global delivery queue, because that backlog sits in front of every other merchant's events |
| 4 double-entry rows per charge times 13.0 million | 52 million rows per day, 19.0 billion per year, 3.8 TB per year at 200 bytes | Rules out keeping the ledger in the same unpartitioned table as operational intent rows, and rules out an in-memory ledger |
| 99.95 percent of 43,200 minutes in a 30-day month | 21.6 minutes of monthly error budget | Rules out defining the objective on the money moving: a single 30 minute gateway outage would exhaust 139 percent of the budget for something you do not operate |
| 13.0 million per day times 120 days of refund window | 1.56 billion intents that must stay online and queryable | Rules out archiving charges to cold storage after 30 days |

!!! tip "Say this out loud in the room"

    "Four hundred fifty writes per second means the write path needs no design at all. What needs design is the 1,125 requests sitting in flight waiting on someone else's network, so that is where I want to spend the time."

### V0, the simplest thing that works (payments)

**Predict first.** Which single property of V0 fails first, and at what number?

??? note "Show answer"

    The open transaction. V0 inserts the retry key and the intent in one transaction that stays open across a 2.5 second gateway call, so at 450 requests per second the pool holds 1,125 connections. The number that matters is not throughput, it is concurrency, and it is set by someone else's latency.

```
+-----------+     +-------------------+     +-----------------+
| Merchant  | --> | Payments API      | --> | Postgres        |
| server    | <-- | key check, charge | <-- | intents         |
+-----------+     | one transaction   |     | retry_keys      |
                  +-------------------+     | ledger_entries  |
                          |                 +-----------------+
                          v
                  +-------------------+
                  | Card gateway      |
                  | third party       |
                  +-------------------+
```

**Caption.** V0 has four components. The merchant's server calls one payments service, which checks the retry key and writes the intent and its ledger entries in a single database transaction, calling the third-party card gateway inside that transaction. The merchant learns the outcome by polling the intent.

Why this is correct at the stated scale:

- 450 inserts per second on one primary is unremarkable. The unique index on (merchant_id, retry_key) gives duplicate protection with no extra component.
- Putting the retry key and the effect in one transaction is the only version of idempotency that is actually safe, because there is no window where one exists without the other.
- No webhook infrastructure. At 39 million events per day a polling endpoint is more work for the merchant but zero work for you, and the merchant already has to poll for anything they missed.

!!! warning "When not to use even this much"

    Below roughly 1 charge per second, and when your gateway responds in under 300 ms, drop the separate retry-key table entirely. Put a unique constraint on the merchant's own order reference and let the database reject the duplicate. The measurement that justifies a dedicated retry-key record is the first support ticket about a double charge, or a client retry rate above about 1 percent of requests.

### Breaking point, then V1 (payments)

**Breaking point one: connection pool exhaustion, at any sustained rate above roughly 120 requests per second.**

What fails: the transaction stays open for the whole gateway call. With a pool of 300 connections and a 2.5 second hold, the service can only complete 300 divided by 2.5, or 120 requests per second. Above that, requests queue for a connection, the merchant's 10 second client timeout fires, and the merchant retries with the same key while the first attempt is still open. The retry blocks on the row lock, consuming a second connection, and the pool collapses.

How you observe it: connection pool wait time at p99, and a counter of concurrent requests arriving on a key that is already locked. The signature is that server-side success rate stays high while merchant-side success rate falls, because the merchant gave up before you answered.

```
+-----------+     +----------------+     +----------------+
| Merchant  | --> | API tier       | --> | Postgres       |
| server    | <-- | write intent   | <-- | intents        |
+-----------+     | return 202     |     | retry_keys     |
                  +----------------+     | outbox         |
                                         +----------------+
                                                 |
                  +----------------+             v
                  | Charge worker  | <-----------+
                  | calls gateway  |
                  +----------------+
                          |
                          v
+-----------+     +----------------+
| Merchant  | <-- | Event          |
| hook URL  |     | dispatcher     |
+-----------+     +----------------+
```

**Caption.** V1 has six components. The API tier writes the intent, the retry key and an outbox row in one short transaction and returns 202 immediately. A separate worker reads the outbox, calls the gateway and records the outcome. An event dispatcher signs and posts state changes to the merchant's endpoint.

V1 changes, each with the observation that justified it:

| Change | Justified by | Cost you accept |
|---|---|---|
| Return 202 with a non-terminal intent | 1,125 in-flight requests against a 300 connection pool | Every merchant integration now has to handle a state that is neither success nor failure, and your documentation carries that burden forever |
| Gateway call moves to a worker | Transaction hold time tracked gateway p99 exactly | The intent can sit in `processing` if the worker dies, so you now owe a sweeper |
| Retry key gains an explicit `reserved` state | Concurrent requests on one key blocking on a row lock | A second concurrent request gets 409 with `Retry-After`, which is a new error the merchant must handle |
| Outbox table instead of publishing directly | Publishing after commit can lose the event if the process dies between the two | The outbox is another table to drain, and its lag is a new thing to alert on |
| Event dispatcher | 1,353 events per second at peak, which cannot ride the request path | Delivery becomes at-least-once, and you must say so before the merchant discovers it |

**Breaking point two: one dead merchant endpoint stalls delivery for everyone, at about 7 dead endpoints.**

What fails: the dispatcher has 200 workers and a 30 second timeout per delivery attempt. A hanging endpoint occupies a worker for the full 30 seconds. If enough traffic is directed at hanging endpoints, throughput falls to 200 divided by 30, or 6.7 attempts per second, against the 1,353 per second the system produces. Seven merchants each holding 5 percent of volume is enough to consume the pool.

How you observe it: worker occupancy grouped by destination, and delivery lag at p99 per destination plotted against the global figure. When one destination's lag and the global lag move together, you have head-of-line blocking, not a capacity problem, and adding workers buys minutes.

V2 changes:

- Delivery is partitioned by destination endpoint, with a per-destination concurrency cap and a per-destination circuit breaker. One merchant's outage is bounded to that merchant's queue.
- Events that exhaust the 72 hour retry window move to a dead-letter store with a replay operation the merchant can trigger themselves.
- A catch-up read, `GET /events?after={cursor}`, lets a merchant who missed deliveries reconcile without opening a ticket. This is the single cheapest support-cost reduction in the design.
- A reconciliation sweeper re-queries the gateway for any intent left in `processing` for more than 5 minutes, because a lost gateway callback otherwise means money moved with no record of it.

!!! warning "When not to use per-destination queues"

    Below roughly 50 destinations, or when no single destination exceeds about 2 percent of event volume, a single queue with a short per-attempt timeout and a circuit breaker is enough. Per-destination queues multiply your queue count by your customer count, which is a real operational cost. The measurement that justifies the split is delivery lag correlating across unrelated destinations.

### Deep dive one: what a retry key actually stores, and for how long

This is the dive the short treatments skip, and it is the most common senior probe on this prompt.

**A retry key has three states, not two.** Candidates almost always design for two.

| State | What the record holds | What a duplicate request gets |
|---|---|---|
| Absent | Nothing | Proceed: insert the key and the effect in one transaction |
| Reserved | Key, request fingerprint, no response yet | 409 with `Retry-After: 1` and a machine-readable code saying the original is in flight |
| Completed | Key, fingerprint, status code, replayable headers, response body | The original response, byte for byte, with a header marking it a replay |

**The fingerprint is what stops the worst bug.** Store a hash of the operation, the key, the authenticated principal and a canonicalised body. If a merchant reuses a key with a different amount, that is a bug in their code, and the only safe response is to refuse it with a distinct error rather than silently replay a charge for the wrong amount.

**Scope the key per merchant, never globally.** A global key space lets one merchant's key collide with another's, which is both a correctness bug and an information leak. The unique index is on the pair, and the pair is what the fingerprint covers.

**Store the key in the same transaction as the effect.** If the key lives in a cache and the intent lives in the database, a crash between the two writes produces either a double charge or a charge nobody can find. Both are worse than the latency you saved.

**Size and lifetime, derived.** From the estimation table, 13 GB of retry records per day. A 24 hour lifetime keeps that at 13 GB live, which one table holds comfortably. Extending to 7 days costs 91 GB and buys protection against a merchant whose weekly reconciliation job replays requests.

| Property | Mechanism | What it prevents |
|---|---|---|
| No duplicate effect | Unique index on (merchant, key), written in the effect's transaction | The double charge, which is the only failure here that reaches a newspaper |
| No silent wrong effect | Request fingerprint compared on replay | A key reused with a changed amount being charged as the original |
| No stuck client | `reserved` state answering 409 with `Retry-After` | A retry blocking on a row lock and consuming a second connection |
| Bounded storage | 24 hour expiry, documented | A table that grows to 23.7 TB holding records nobody reads |

**What retry keys do not give you.** They deduplicate one client's retries of one request. They do nothing about a merchant submitting the same customer order twice with two different keys, which is a different problem and needs a merchant-supplied order reference with its own uniqueness constraint. Saying this unprompted reads as experience.

**Retryability differs by operation, and the contract should say so.** Reading an intent is safe and repeatable. Creating an intent or a refund changes state and needs a key. Cancelling is naturally repeatable because the second cancel finds the intent already cancelled and returns the same representation.

The IETF HTTP API working group has a draft, [The Idempotency-Key HTTP Header Field](https://datatracker.ietf.org/doc/draft-ietf-httpapi-idempotency-key-header/), whose latest revision is 07 from 15 October 2025 and which is currently expired (accessed August 2026). Citing it is useful precisely because it is a draft: there is no standard here, so your design has to state its own rules.

!!! warning "When not to build retry-key infrastructure"

    Below roughly 1 write per second, or when the operation has a natural unique business key you already store, a unique constraint on that key is cheaper and has fewer states. A retry-key table adds a second store, an expiry job and the `reserved` state's error code. The measurement that justifies it is a duplicate-effect incident, or a client retry rate above 1 percent of requests.

### Deep dive two: event delivery as a promise you can be held to

The dives on this page vary by problem. Here delivery is interesting because the merchant's endpoint is a component in your availability that you cannot deploy, monitor or fix.

**Name the guarantee before designing the mechanism.** At-least-once, unordered, with enough information in the payload for the consumer to reorder and deduplicate. Anything stronger is a promise you will break during your next incident.

| Property | What you promise | What the consumer must do |
|---|---|---|
| Delivery | At least once, retried for 72 hours | Deduplicate on the event identifier |
| Ordering | None across events, but a monotonically increasing sequence per intent | Discard an event whose sequence is lower than the one already applied |
| Freshness | The payload is a snapshot at emit time | Re-read the intent before acting on money |
| Authenticity | Signature over the raw body and a timestamp | Verify before parsing, and reject stale timestamps |

**Sign the raw bytes, not the parsed object.** Re-serialising JSON changes whitespace and key order, so a signature computed over a re-encoded body fails for reasons nobody can debug. The signature covers a timestamp concatenated with the exact body, keyed per endpoint, and the header carries a scheme version prefix so you can rotate the algorithm later. RFC 9530, [Digest Fields](https://www.rfc-editor.org/rfc/rfc9530.html) (February 2024, accessed August 2026), defines `Content-Digest` for the integrity half of this.

**Replay protection needs both halves.** The signed timestamp lets the consumer reject anything older than a few minutes. The event identifier lets them reject anything they already applied. Neither alone is sufficient: the timestamp window is wide enough to replay inside, and identifier deduplication alone lets an attacker replay an old signed event forever.

**Thin or fat payloads is a real trade-off, not a style choice.**

| Payload style | Advantage | Cost |
|---|---|---|
| Thin: identifier and type only | No stale data, minimal exposure of details in transit, smallest signature surface | Every event costs the merchant a read, so your read traffic scales with your event traffic |
| Fat: full object snapshot | Merchant can act without a round trip, and can operate through a brief outage of your read path | The snapshot can be stale on arrival, and merchants will act on it anyway |

The design I would defend: fat payloads carrying an explicit object version, plus a documented rule that anything moving money must re-read first. The version is what makes the rule enforceable rather than aspirational.

**The retry schedule is arithmetic, not folklore.** Exponential backoff from 10 seconds with a cap of 1 hour and full jitter reaches 72 hours in about 75 attempts. Full jitter matters because a merchant endpoint recovering from an outage otherwise receives every backlogged event in one synchronised burst, which knocks it over again.

**The debugging surface is the feature.** A per-attempt log showing the request you sent, the response you got and the timestamp, visible to the merchant without a support ticket, removes more support load than any other single thing in this design. It also removes the argument about whether you sent the event.

!!! warning "When not to use webhooks at all"

    If every consumer can poll and each one sees fewer than about 1 event per second, a cursor-based events endpoint is strictly simpler: no signing, no retries, no dead letters, no per-destination queues, and no dependency on the consumer's uptime. The measurement that justifies pushing is a poll rate where over 90 percent of polls return empty, or a freshness requirement tighter than the poll interval you can afford.

### Failure modes and evaluation (payments)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Duplicate effect from a lost response | Merchant times out, retries, and the first attempt also succeeds | Retry key written in the effect's transaction, plus the `reserved` state for concurrent arrivals |
| Head-of-line blocking | One hanging endpoint occupies the shared delivery pool | Partition delivery by destination, cap concurrency per destination |
| Retry storm | Gateway slows, every merchant retries, in-flight count doubles, it slows further | Bounded retries with jitter, a per-merchant retry budget, and load shedding at the gateway edge |
| Metastable failure | After the gateway recovers, the accumulated retry backlog holds load above capacity | Shed load until the backlog drains, and cap retries as a fraction of successful traffic rather than a fixed count |
| Thundering herd | A dead merchant endpoint returns and receives 1.95 million backlogged events at once | Full jitter on backoff, and a per-destination rate cap on drain |
| Stuck state from a lost callback | Gateway charged the card, its callback never arrived, intent sits in `processing` | Reconciliation sweeper that re-queries the gateway after 5 minutes |
| Clock skew | Merchant server clock drifts, every signature is rejected as stale | Publish the tolerance, return a distinct error code naming skew, and include your timestamp in the response |
| Split brain | Two workers pick up the same outbox row and both call the gateway | Row-level claim with a lease, plus the gateway's own retry key derived from the intent identifier |

Service level indicators to publish:

- Accept-path success rate, measured as non-5xx responses on intent creation.
- Our own overhead at p99, computed as total span minus the gateway span, so a slow gateway does not hide a slow service.
- Retry-key conflict rate, split into matching fingerprints (healthy client retries) and mismatched fingerprints (a client bug you should tell them about).
- Fraction of events delivered within 60 seconds, reported globally and per destination.
- Count of intents in `processing` older than 5 minutes.
- Ledger imbalance: the count of transactions whose entries do not sum to zero.

Alert on ledger imbalance immediately at any non-zero value, because it is a correctness signal and not a capacity signal. Alert on stuck intents when the count stays above zero for 15 minutes. Alert on error budget burn rate for the accept path, not raw error counts. Do not page on one destination's delivery lag; that is the merchant's outage, and it belongs on a dashboard.

Re-checking against the non-functional requirements:

| Requirement | Does V2 meet it? |
|---|---|
| 99.95 percent accept-path availability | Yes, because accepting an intent is a short local transaction that does not depend on the gateway |
| Overhead p99 under 150 ms | Yes, the gateway call left the request path in V1; the remaining work is two indexed writes |
| 450 charge attempts per second at peak | Yes, with headroom of more than an order of magnitude on the write path |
| 99.9 percent of events within 60 seconds | Yes for healthy destinations; explicitly not promised for a destination that is down, and the contract says so |
| 24 month contract stability | Only if the non-terminal state was in the resource from day one. Adding it later is the breaking change this design exists to avoid |
| 120 day refund window | Yes, with intents partitioned by month and the oldest partitions kept online |

### Where this answer is a simplification (payments)

- **Card data changes your compliance boundary, not just your code.** If raw card numbers reach your servers, the audit scope covers every system they touch. The usual answer is a browser-side tokenisation endpoint on your own origin so the merchant never holds the number, which turns a two-party integration into a three-party one.
- **Strong customer authentication turns the flow inside out.** A step-up challenge means the intent has to express "the buyer must be sent somewhere", which is a whole extra shape in the resource and a redirect the merchant has to implement. Designing it in later is a breaking change.
- **Disputes are a second state machine with legal deadlines.** Evidence submission, deadlines set by the card scheme, and outcomes that reverse money months later. The refund resource does not model any of it.
- **Reconciliation is the real product.** Merchants care less about the charge call than about whether your numbers match their bank statement, which means exportable, immutable, dated ledger views with documented rounding.
- **Primary sources with dates.** RFC 9110, [HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) (June 2022, accessed August 2026), is the authority on which status codes carry which meaning, including 409 and `Retry-After`.
- **More primary sources.** RFC 9457, [Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html) (July 2023, accessed August 2026), gives you a standard error body so you are not inventing one. The [Idempotency-Key draft](https://datatracker.ietf.org/doc/draft-ietf-httpapi-idempotency-key-header/) (revision 07, 15 October 2025, expired, accessed August 2026) is the closest thing to a shared convention. For the queue and outbox mechanics underneath the dispatcher, see the movement building blocks in [Modern System Design](modern-system-design.md).

### Level delta (payments)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Retry semantics | Mentions an idempotency key on the request | Designs the key record, the fingerprint and the unique index, and puts it in the effect's transaction | Adds the `reserved` state with a derived error code, and names the case retry keys do not solve |
| State model | Charge succeeds or fails | Adds a non-terminal state because the gateway is slow | Puts the non-terminal state in version one specifically because adding it later is the breaking change |
| Estimation | Produces throughput numbers | Uses concurrency, not throughput, and derives 1,125 in flight | Notices the write path needs no design and reallocates the whole budget to delivery and reconciliation |
| Event delivery | Says "we send a webhook" | Signs the raw body, retries with backoff, and states at-least-once | Partitions by destination on a derived head-of-line number, and ships a self-service catch-up read |
| Error model | Returns 400 and a message | Uses a standard problem body with stable machine-readable codes | Separates retryable from terminal in the code itself, and versions the signature scheme for rotation |
| Operations | Mentions monitoring | Publishes latency and success indicators | Adds ledger imbalance as a correctness alert and refuses to page on a merchant's own outage |

What each level added, in one line each:

- **Mid to senior:** every mechanism now has a number behind it, and the failure it prevents is named rather than implied.
- **Senior to staff:** compatibility becomes a design input rather than a later problem, and at least one alert is about correctness rather than availability.

---
## Case study 2: Comment and threading service

This prompt looks like a storage question and is actually a contract question. The first thing that breaks is not the database, it is the promise your list endpoint made to a client that cannot be upgraded. The depth here goes on the tree representation and on the cursor.

### The prompt (comments)

"Design the interface for comments on a post, including replies to replies, for a mobile client that renders a thread and lets people load more."

### Questions that fork the design (comments)

| Question worth asking | If the answer is A the design becomes | If the answer is B it becomes |
|---|---|---|
| Is nesting capped or unbounded? | Capped at two levels: a reply's parent is always a top-level comment, so the read is two flat queries and the storage is trivial | Unbounded: you need a tree representation, a subtree read, and a rule for what the response does when it runs out of depth budget |
| Is the default order chronological or ranked? | Chronological: the sort key is immutable, so a keyset cursor is stable by construction | Ranked: the sort key mutates under the reader, so the cursor must pin something or the contract must admit drift |
| Are deletes hard or soft? | Hard: deleting a parent orphans its subtree, and you owe a rule for what happens to the children | Soft: a tombstone preserves tree shape, the client renders a placeholder, and every cursor stays valid |
| Does the client need an exact reply count? | Approximate: a periodically refreshed estimate with a documented staleness costs almost nothing | Exact: a maintained counter, which becomes a contended row on exactly the posts you care about |
| Can a comment be edited? | Immutable after create: caching is trivial and every page is safe to store forever | Editable: you need entity tags, conditional writes, and an answer to the lost update problem |
| Is per-viewer filtering required, for blocks and mutes? | No: a page is identical for everyone, so it can be cached once and shared | Yes: the result set is per viewer, shared caching dies, and pagination must be computed per session |
| Is live delivery required? | Poll only: one read endpoint and a cursor is the whole contract | Live: you need a subscription channel and a rule for how a pushed comment interacts with a cursor the client already holds |

### Requirements (comments)

Functional:

- Create a comment on a post, optionally as a reply to an existing comment.
- Read a page of top-level comments, each with a bounded preview of its replies.
- Read further replies under one comment, independently paginated.
- Edit a comment, with the client able to detect that it is overwriting someone else's change.
- Soft delete a comment without breaking the shape of the thread.
- Page forwards and backwards through a collection that other people are writing to.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Comments created per day | 20 million | ASSUMED, a large consumer product; ask, because this number sets nothing else on its own |
| Peak multiplier | 3x | ASSUMED, evening peaks concentrate traffic |
| Thread page reads | 6,000 per second average | ASSUMED, only a minority of post viewers open the comments |
| First-page share of reads | 90 percent | ASSUMED, most readers never tap "load more" |
| Concurrently hot posts | 10,000 | ASSUMED, the set of posts receiving traffic in any five second window |
| Freshness of a first page | at most 5 seconds stale | ASSUMED, a comment thread that lags visibly reads as broken |
| First page latency, p99 | under 120 ms server side | ASSUMED, the comment sheet animates open and must be populated before the animation ends |
| Largest single thread | 200,000 comments | ASSUMED, one post per week goes far outside the distribution |
| Maximum reply depth seen | 200 | ASSUMED, people reply to replies as a game once a thread is popular |
| Mobile round trip | 400 ms on a poor connection | ASSUMED, this is the number that makes chattiness expensive |
| Contract stability | pagination contract valid for 24 months | GIVEN, shipped mobile clients cannot be forced to upgrade |
| Writes per second | 231 average, 694 at peak | DERIVED below |
| Rows after three years | 21.9 billion, 8.8 TB | DERIVED below |
| Origin queries per second at peak | about 3,800 | DERIVED below |

### Estimation that eliminates (comments)

| Calculation | Result | What it rules out |
|---|---|---|
| 20 million divided by 86,400 seconds | 231 writes per second average, 694 at the 3x peak | Rules out sharding the write path, rules out a queue in front of inserts, and rules out spending any interview minutes on write throughput |
| 6,000 reads per second at 3x | 18,000 page reads per second at peak | Rules out serving reads directly from one primary, so a cache tier is load-bearing rather than decorative |
| 10,000 hot posts divided by a 5 second cache lifetime, plus 10 percent of 18,000 for deeper pages | 2,000 plus 1,800, about 3,800 origin queries per second | Rules out a cache lifetime longer than 5 seconds, because the freshness requirement already caps it, and rules out claiming the cache removes the need for an index |
| 20 million times 365 times 3 years, at 400 bytes per row | 21.9 billion rows, 8.8 TB | Rules out one unpartitioned table, and rules out any design that holds a whole thread in application memory |
| 200,000 comments with a page size of 50 | 4,000 pages; the last page's offset is 199,950 | Rules out offset pagination outright, because the database reads and discards 199,950 rows to return 50 |
| 50 top-level comments, each needing its own call for replies | 51 requests per screen; at 18,000 screens per second that is 918,000 requests per second | Rules out a contract where the client composes the thread, and rules out "just add another endpoint" as the answer to nesting |
| 51 requests times a 400 ms mobile round trip, even pipelined four at a time | about 5.1 seconds to render one screen | Rules out any contract that needs more than two calls to paint the first screen |
| Depth 200 fetched one level per query at 1 ms each | 200 ms against a 120 ms budget | Rules out per-level fetching, and rules out an unbounded depth contract that you would then have to honour |
| 3 percent of comments edited, of 20 million | 600,000 edits per day, about 7 per second | Rules out a separately partitioned edit-history subsystem. A version column and an entity tag carry this load |

!!! tip "Say this out loud in the room"

    "Two hundred thirty-one writes per second tells me the storage is easy. Fifty-one requests per screen tells me the interface is the problem, so I am going to design the response shape before I design the table."

### V0, the simplest thing that works (comments)

**Predict first.** V0 is fast enough and correct on paper. What does a real user notice first?

??? note "Show answer"

    They post a comment and it is not there when the sheet reloads, and refreshing twice shows two different versions of the same page. The cache is per instance with a 5 second lifetime, so two consecutive requests can land on instances holding different snapshots and the list appears to move backwards.

```
+---------+     +------------------+     +-----------------+
| Client  | --> | Comments API     | --> | Postgres        |
|         | <-- | in-process page  | <-- | comments table  |
+---------+     | cache, 5 s life  |     | id, post_id,    |
                +------------------+     | parent_id, ts   |
                +------------------+     +-----------------+
```

**Caption.** V0 has three components. A client calls one comments service that keeps a small in-process cache of rendered pages for five seconds, and on a miss runs a keyset query against a single Postgres table holding every comment as a row with an optional parent reference.

Why this is correct at the stated scale:

- 694 inserts per second on one primary with two indexes is well inside a single node's range.
- The cache absorbs the first-page traffic. With 10,000 hot posts and a five second lifetime, the origin sees at most 2,000 first-page queries per second no matter how much traffic arrives, because each post can only miss once per lifetime per instance.
- Ordering is chronological, so the cursor is a keyset on the pair (created timestamp, identifier), which is immutable per row. Offset was eliminated above and never appears.
- The tree is an adjacency list: each row stores its parent. At depth three, which covers 95 percent of comments, three queries is acceptable.

!!! warning "When not to use even this much"

    Below about 5 comments per second and 500 page reads per second, remove the in-process cache. It saves under a millisecond per request and introduces the staleness that becomes breaking point one. The measurement that justifies adding it back is database CPU where thread-page queries exceed roughly 30 percent of the total.

### Breaking point, then V1 (comments)

**Breaking point one: non-monotonic reads, at any deployment with more than one instance.**

What fails: a page cached independently on each instance for five seconds. Two consecutive requests from the same client can be served by instances whose entries were populated three seconds apart, so the newer response contains fewer comments than the older one. The author of a new comment sees it missing for up to five seconds. Neither is a capacity problem and neither gets better with more hardware.

How you observe it: an author-visibility indicator, measured as the fraction of comment creations whose next read by the same client contains the new comment, and a synthetic client that asserts the head cursor of a thread never moves backwards. Both fail at any traffic level, which is the tell that this is a contract defect.

The fix is a change to the contract, not a box. Split the collection into a part that never changes and a part that is tiny:

| Part of the collection | Mutability | How it is served |
|---|---|---|
| A page addressed by a cursor | Fixed range of rows; contents change only if a row inside it is edited or deleted | Cached with a long lifetime, revalidated with an entity tag |
| The head of the collection | Changes on every new comment | A small per-post pointer with a one second lifetime, written through on create |
| A newly created comment | Known to its author immediately | Returned in the create response together with the new head pointer, so the client splices locally |

```
+---------+   +---------------+   +-----------+   +-------------+
| Client  |-->| Comments API  |-->| Shared    |-->| Postgres    |
|         |<--| local cache   |<--| cache     |<--| primary     |
+---------+   | 1 s head life |   | pages by  |   +-------------+
              +---------------+   | cursor    |          |
                                  +-----------+          v
                                                  +-------------+
                                                  | Read        |
                                                  | replicas    |
                                                  +-------------+
```

**Caption.** V1 has five components. Pages addressed by a cursor are immutable ranges held in a shared cache, the per-post head pointer is the only hot mutable value and carries a one second lifetime, and page queries go to read replicas while the head pointer is written through on the primary.

V1 changes, each with the observation that justified it:

| Change | Justified by | Cost you accept |
|---|---|---|
| Pages keyed by cursor, cached long | Author-visibility indicator failing at every traffic level | An edit inside an old page must invalidate that page, so you need the entity tag to revalidate |
| Head pointer as a separate tiny value | Head cursor moving backwards on the synthetic client | One more hot key per post, and a stampede risk when it expires |
| Create returns the comment and the new head | Read-your-writes cannot be satisfied by a cache with a lifetime | The client now owns a splice, which is client code you have to specify precisely |
| Read replicas | Origin query rate at 3,800 per second on one primary | Replica lag means a cursor issued from the primary can point past what a replica has, so the head pointer is read from the primary |

**Breaking point two: the client composes the thread, at 50 top-level comments per screen.**

What fails: V1 returns a flat page of comments. To render a screen the client asks for 50 top-level comments, then makes one call per comment for the first few replies. That is 51 requests per screen, 918,000 requests per second at peak, and about 5.1 seconds of wall clock on a 400 ms mobile round trip even with four requests in flight. Separately, a thread nested 200 deep needs 200 sequential queries at the origin, which is 200 ms against a 120 ms budget.

How you observe it: requests per rendered screen, reported per client version, and the ratio of reply-list calls to thread-page calls. When that ratio sits near the page size, the contract is forcing the fan-out. On the origin, watch queries per page render rather than queries per second.

V2 changes:

- Storage moves from an adjacency list to a materialised path, so a whole subtree is one indexed range scan instead of one query per level. The derivation for why the other candidates lose is in deep dive one.
- The thread endpoint returns top-level comments each carrying a bounded reply preview and its own nested cursor, so one call paints the screen and "load more replies" is a second call only when the reader asks.
- Depth is capped at 40 in storage and expressed in the contract as a link that continues the thread from a new root, so an unbounded product decision does not become an unbounded query.
- Reply counts become exact below 1,000 and approximate above it, stated in the response so the client renders "1.2k" honestly rather than guessing.

!!! warning "When not to use a nested response shape"

    If a screen shows at most one level of replies and the page size is under about 10, two flat calls are simpler and cache better than one nested response, because a flat page has one cursor and one entity tag rather than one per item. The measurement that justifies nesting is requests per rendered screen above roughly 3, or a first-screen paint time dominated by round trips rather than by server time.

### Deep dive one: three ways to store a tree, and why the client never sees which one you chose

**The candidates, compared on what this problem makes expensive.**

| Representation | Subtree read | Insert cost | Move a subtree | Depth limit | Storage overhead |
|---|---|---|---|---|---|
| Adjacency list, parent reference per row | One query per level | One row | One row | None | None |
| Recursive query over the adjacency list | One query, but the planner walks level by level | One row | One row | None | None |
| Materialised path, ordered segments per level | One range scan on a prefix index | One row | Rewrite the subtree's paths | Set by path width | Path bytes per row |
| Closure table, one row per ancestor pair | One indexed lookup | One row per ancestor | Rewrite all pairs under the node | None | Very large, see below |
| Nested sets, left and right bounds | One range scan | Renumber roughly half the tree | Cheap | None | Two integers per row |

**Why the closure table loses here, with the arithmetic.** A closure table stores one row per ancestor-descendant pair, so a comment at depth d contributes d plus 1 rows. With a mean depth of 2.5 across 21.9 billion comments, that is 3.5 times 21.9 billion, or 77 billion rows, against a base table of 21.9 billion. You would be paying 3.5 times the storage and 3.5 times the write amplification to make a read fast that a prefix scan already makes fast.

**Why nested sets lose here.** Inserting a node renumbers every node to its right. On a 200,000 comment thread that averages 100,000 row updates per insert, at a peak insert rate of 694 per second. Nested sets are a read-optimised structure for a tree that almost never changes, and a comment thread is the opposite of that.

**Why the recursive query loses at depth.** It is one statement, but the engine still resolves one level at a time, so latency grows with depth rather than with result size. At the measured depth of 200 that is the same 200 ms the estimation section already ruled out.

```
+---------------------------------------------------------+
| comments table, one row per comment                     |
| path = 000a.001f.0003    depth = 3    post_id = 91       |
| subtree read = WHERE path LIKE '000a.001f.%'             |
| index on (post_id, path) makes it one range scan         |
+---------------------------------------------------------+
```

**Caption.** The materialised path stores each comment's ancestors as fixed-width ordered segments in one column, so reading a subtree is a single prefix range scan on an index over the post identifier and the path.

**Path width is a design number, not a default.** Six characters per level, times a depth cap of 40, is 240 bytes plus separators, which sits below the 400 byte row size assumed above. Allowing depth 200 would cost 1,200 bytes of path on a 400 byte row, so the path would be three quarters of the table. The depth cap is what keeps the representation cheap, and it is a product decision you should surface rather than bury.

**The contract lesson, which is the actual point of this dive.** None of this appears in the interface. The client receives an identifier, a parent identifier, a depth and a cursor. It never receives the path. The moment a path appears in a response, clients parse it, and your storage representation has become a public contract you can never change. Expose the shape of the tree, not the encoding of it.

Bill Karwin's *SQL Antipatterns* (Pragmatic Bookshelf, 2010) is the clearest catalogue of these four representations and their costs, and it predates every framework you might otherwise cite for them.

!!! warning "When not to use a materialised path"

    Below roughly 10,000 comments per thread and a depth of 5 or less, an adjacency list with a recursive query is simpler, needs no path maintenance and no depth cap, and moves a subtree with a single row update. The measurement that justifies the change is subtree read latency at p99 exceeding your budget, or round trips per subtree read above about 3.

### Deep dive two: a cursor that survives inserts, deletes and an edit

The dives on this page vary by problem. Here the cursor is the interesting object because the collection is being written to while a client walks it, which is not true of the file listings in the next case study.

**The three failure modes of pagination on a mutating collection.** Name them before proposing a mechanism.

| Failure | What the user sees | Cause |
|---|---|---|
| Duplicates | The same comment on two consecutive pages | Rows inserted ahead of the reader's position shift everything down |
| Skips | A comment that exists is never shown | Rows deleted ahead of the reader's position shift everything up |
| Invalid position | The list restarts or errors halfway | The cursor referenced a row that was hard deleted |

**Keyset is the mechanism, and the tiebreaker is the part people forget.** Order by the pair (created timestamp, identifier) and let the cursor carry both values. Ordering by timestamp alone is ambiguous whenever two comments share a millisecond, and at 694 writes per second that happens constantly, producing duplicates and skips that look random. The final sort key must be unique.

**Make the cursor opaque, and make it carry its own validity.** A cursor is a base64 string over a small structure:

| Field | Why it is there |
|---|---|
| Sort values from the last row returned | The position itself, which is what makes the cursor immune to hard deletes of that row |
| Direction | So one token serves both forwards and backwards paging |
| Filter fingerprint | So a cursor reused with a different sort or filter is rejected instead of silently returning nonsense |
| Schema version | So you can change the encoding without breaking clients holding old tokens |
| Signature | So clients cannot construct cursors, which is what would freeze your internals into the contract |

That structure is roughly 41 bytes before encoding, so cursor size is not a constraint and does not need discussing further. What does need discussing is expiry: state a lifetime, and return a distinct machine-readable code when an expired cursor arrives, using the problem details format from RFC 9457, [Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html) (July 2023, accessed August 2026). A client that cannot tell "your cursor expired" from "you are unauthorised" will retry forever.

**Soft delete is a pagination decision, not a storage preference.** A tombstone keeps the row, so every cursor pointing at or past it stays valid and the tree keeps its shape. Hard deletion is still possible because the cursor stores sort values rather than a row reference, but it creates the skip failure above and it orphans replies. Choose soft delete and say why.

**Edits and the lost update problem.** A comment page cached under a cursor is only safe if edits invalidate it. Serve an entity tag per comment and per page, require a conditional write on edit, and return 412 when the tag does not match. RFC 9110, [HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) (June 2022, accessed August 2026), defines the precondition semantics in section 13, and RFC 9111, [HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111.html) (June 2022, accessed August 2026), defines how the revalidation works.

**Total counts are the expensive part of the contract.** An exact count of a 200,000 comment thread means counting 200,000 rows on every page request. Return a boolean for whether more exist, plus a count that is exact below 1,000 and marked approximate above it. Expressing the approximation in the response is what lets you change how it is computed later.

**Bidirectional paging needs two link relations, not two endpoints.** Emit `next` and `prev` links using the relation registry from RFC 8288, [Web Linking](https://www.rfc-editor.org/rfc/rfc8288.html) (October 2017, accessed August 2026), so a client follows links rather than assembling URLs, and so you can change the URL shape without a breaking change.

!!! warning "When not to use opaque cursors"

    For a collection that is small, bounded and effectively immutable, under roughly 1,000 rows, page numbers are easier for developers to reason about, easier to deep link, and let a user jump to the end. The measurement that justifies cursors is either a p99 query time on the last page that exceeds the first page's, or a synthetic paginator observing any duplicate or skip on a live collection.

### Failure modes and evaluation (comments)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Non-monotonic reads | Independent per-instance caches serve older and newer snapshots in turn | Immutable pages keyed by cursor, one small mutable head pointer |
| Hot key | One post's head pointer takes a large share of cache traffic | Short-lived local copy of the head pointer, so shared-cache load per post is bounded by instance count divided by the lifetime |
| Cache stampede | The head pointer expires and every instance refreshes it together | Refresh slightly early with a random offset, or take a single-flight lease |
| Lost update | Two moderators edit the same comment and one change vanishes | Entity tag plus conditional write, returning 412 |
| Fan-out amplification | 51 requests per screen because the client composes the thread | Server-shaped nested response with a bounded reply preview |
| Unbounded work | A request asks for a subtree at depth 200 | Depth cap of 40 in storage, continuation link in the contract |
| Thundering herd | A post from a very large account fills the hot set in seconds | Admission control on origin queries per post, serving stale pages while a refresh is in flight |
| Cursor invalidation storm | A schema change rejects every cursor in flight | Version in the cursor, and support the previous version for at least one client release cycle |

Service level indicators to publish:

- First page latency at p50 and p99, reported separately from deeper pages, because they are different queries.
- Requests per rendered screen, per client version. This is the indicator that catches a chatty contract before the origin notices.
- Author visibility: the fraction of creates whose next read by the same client includes the new comment.
- Pagination integrity, measured by a synthetic client that walks a live thread and asserts no duplicate and no missing identifier.
- Cursor rejection rate, split by reason: expired, wrong filter fingerprint, bad signature.
- Shared cache hit ratio for pages and for the head pointer, plotted separately.

Alert on pagination integrity failures at any rate above zero, because they indicate a correctness defect rather than load. Alert on author visibility falling below a stated threshold. Alert on error budget burn for the first page. Do not alert on cache hit ratio; it is a diagnostic, and it moves for reasons that are not incidents.

Re-checking against the non-functional requirements:

| Requirement | Does V2 meet it? |
|---|---|
| First page p99 under 120 ms | Yes, one range scan plus a bounded reply preview, served mostly from cache |
| 18,000 page reads per second at peak | Yes, since immutable pages cache indefinitely and only the head pointer is hot |
| At most 5 seconds stale | Yes, and in practice better, because the head pointer's lifetime is one second |
| 200,000 comment thread | Yes for reads and for paging; the exact reply count is explicitly not promised above 1,000 |
| Depth 200 | Not as a single subtree. The contract caps depth at 40 and continues, which is a deliberate simplification you should state out loud |
| 24 month contract stability | Yes, provided the cursor is opaque and versioned from the first release. A cursor clients can parse is the thing that ends this |

### Where this answer is a simplification (comments)

- **Ranking changes everything in this design.** A "top comments" order makes the sort key mutable, which invalidates the keyset cursor's core assumption. The honest options are a pinned ranking snapshot per session or a documented admission that results drift. The equivalent problem for a ranked set is worked in case study 4.
- **Per-viewer filtering kills the shared cache.** Blocks, mutes and moderation states make the page different for every reader, so the immutable-page trick stops working and you cache fragments instead of pages.
- **Moderation is a second write path with different latency requirements.** A removal must take effect in seconds, globally, including in caches you have already served, which is a purge problem this design does not address.
- **Live delivery interacts badly with cursors.** A comment pushed to a client that also holds a cursor can arrive twice or out of order, so the push channel has to carry the same identifiers and the client has to deduplicate.
- **Primary sources with dates.** RFC 9110 ([HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html), June 2022) for preconditions and status codes, RFC 9111 ([HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111.html), June 2022) for revalidation, RFC 8288 ([Web Linking](https://www.rfc-editor.org/rfc/rfc8288.html), October 2017) for pagination link relations, and RFC 9457 ([Problem Details](https://www.rfc-editor.org/rfc/rfc9457.html), July 2023) for the error body. All accessed August 2026. For the tree representations, *SQL Antipatterns* by Bill Karwin (Pragmatic Bookshelf, 2010). For the cache tiering underneath V1, see the storage building blocks in [Modern System Design](modern-system-design.md).

### Level delta (comments)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Pagination | Proposes page and size parameters | Uses a keyset cursor and names duplicates and skips | Makes the cursor opaque, signed and versioned, and explains that a parseable cursor is what ends the contract |
| Tree storage | Picks a parent reference and moves on | Compares representations and picks a materialised path with a depth cap | Shows the 77 billion row arithmetic that kills the closure table, and refuses to put the path in the response |
| Response shape | Returns a flat list | Adds a nested reply preview to cut round trips | Derives 51 requests per screen first, then designs the shape the number demands |
| Consistency | Does not raise it | Notices read-your-writes and returns the new comment on create | Names non-monotonic reads as the defect, and splits the collection into an immutable part and a small hot pointer |
| Counts | Returns a total | Returns a total and admits it is expensive | Ships an exact-below-a-threshold contract so the computation can change later without a client change |
| Operations | Mentions latency monitoring | Publishes first-page and deep-page latency separately | Adds a synthetic paginator as a correctness indicator and alerts on it at any non-zero rate |

What each level added, in one line each:

- **Mid to senior:** the contract now names its own failure modes, and the storage choice has arithmetic behind it.
- **Senior to staff:** the interface is designed from a measured client cost, and the parts of the contract that would prevent future change are removed on purpose.

---
## Case study 3: File and blob service

The interesting thing about this prompt is that the bytes and the metadata want completely different designs, and the contract is where you decide how much of the byte path you are willing to own. The depth here goes on resumability and on delegated access.

### The prompt (files)

"Design the interface an application uses to upload files of any size, download them, attach metadata, and have them expire on a schedule."

### Questions that fork the design (files)

| Question worth asking | If the answer is A the design becomes | If the answer is B it becomes |
|---|---|---|
| Do browsers upload directly, or do the customer's servers? | Servers only: you can require a chunk protocol and assume a client that retries correctly | Browsers: you need cross-origin rules, direct-to-storage uploads, and a client you cannot assume anything about |
| What is the largest file? | Under about 32 MB: one request with the bytes in the body is the entire design | Multiple gigabytes: you need a session resource, server-authoritative offsets and resumability |
| Are files immutable once written? | Immutable: content addressing, indefinite caching and cheap deduplication all fall out | Mutable: versions, entity tags, conditional writes, and a cache invalidation problem on every write |
| Who authorises a download? | Public: an edge cache URL and no per-request decision | Per user: either every byte flows through you, or you delegate with a short-lived token and accept that a leaked token is a leaked file |
| Do you need the bytes before the file is usable, for scanning or transcoding? | No: bytes can go straight to storage and the file is immediately readable | Yes: the file resource needs a non-ready state, and every client must handle it |
| Is deletion immediate or retained? | Immediate: delete is terminal and storage is reclaimed | Retained: delete becomes a state transition, with a trash lifetime, restore, and legal hold as a separate concept |
| Is deduplication required? | No: every upload is a distinct object with its own lifetime | Yes: the content hash becomes identity, you need reference counting, and you have to reason about one tenant learning that another holds a given file |
| Do clients need partial reads? | Whole file only: download is one response | Ranges: you need range request support, and your edge cache and your storage both have to preserve it |

### Requirements (files)

Functional:

- Begin an upload and receive a session that can be resumed after a disconnection.
- Send bytes in pieces, in order, and ask the server where to continue from.
- Complete an upload with an integrity check that fails loudly on corruption.
- Read and update file metadata independently of the bytes.
- Download a whole file or a byte range.
- Delete a file, and configure a rule that expires files after a stated age.
- List files in a container with a cursor.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Launch scope | server-to-server clients, files capped at 32 MB | GIVEN, this is what the first customer needs |
| Known next scope | mobile clients, files up to 2 GB | GIVEN, stated in the same conversation, which is why it belongs in the requirements and not in a later phase |
| Uploads per day | 2 million | ASSUMED, a mid-size storage product; ask, because it sets the metadata scale |
| Mean file size | 4 MB, with a 2 GB tail | ASSUMED, document-heavy workloads sit here and the tail is video |
| Download to upload byte ratio | 10 to 1 | ASSUMED, files are written once and read many times |
| Mobile uplink, median | 3 Mbps | ASSUMED, this is the number that makes large uploads a protocol problem rather than a bandwidth problem |
| Shortest idle timeout in the path | 350 seconds | ASSUMED, a common load balancer default, and you control only some of the hops |
| Bytes one instance can move | 1 Gbps of encrypted transfer | ASSUMED, a single instance saturates on encryption and copying well before it saturates on request handling |
| Time to first byte on download, p99 | under 200 ms | ASSUMED, a download that hesitates reads as a broken link |
| Durability target | eleven nines | ASSUMED, the industry expectation for stored objects, and the reason you do not build this yourself |
| Abandoned upload rate | 5 percent | ASSUMED, people close tabs |
| Ingest bytes | 92.6 MB per second average, 278 MB per second at peak | DERIVED below |
| Egress bytes | 7.4 Gbps average, 22 Gbps at peak | DERIVED below |
| Storage growth | 2.9 PB per year | DERIVED below |

### Estimation that eliminates (files)

| Calculation | Result | What it rules out |
|---|---|---|
| 2 million divided by 86,400 seconds | 23 uploads per second average, 69 at the 3x peak | Rules out sharding the metadata store, rules out a queue on the create path, and rules out spending time on request throughput |
| 2 million times 4 MB, divided by 86,400 | 8 TB per day, 92.6 MB per second average, 278 MB per second at peak | Rules out one instance handling ingest, and confirms that ingest alone is a three instance problem, not an architecture problem |
| 8 TB per day times 10, converted to bits | 80 TB per day, 7.4 Gbps average, 22 Gbps at peak | Rules out proxying downloads: at 1 Gbps per instance that is 22 instances doing nothing but copying bytes, and their latency becomes your metadata latency |
| 2 GB times 8, divided by 3 Mbps | 5,333 seconds, about 89 minutes | Rules out a single-request upload, rules out buffering the body in memory, and rules out any design that assumes one uninterrupted connection |
| 3 Mbps times 350 seconds, divided by 8 | 131 MB | Rules out raising the timeout as the fix, because the 131 MB cliff is set by whichever hop has the shortest timeout, and some of those hops belong to a carrier |
| 8 TB per day times 365 | 2.9 PB per year | Rules out instance-attached block storage, and rules out shipping without a lifecycle rule |
| 32 MB chunk at 3 Mbps versus 1 MB chunk over a 2 GB file | 85 seconds lost per disconnect at 32 MB; 2,048 requests times 50 ms, or 102 seconds of pure overhead, at 1 MB | Rules out both ends of the range, leaving a chunk band of roughly 4 to 32 MB |
| 5 percent of 2 million, at a 2 MB mean partial | 100,000 abandoned sessions per day, about 200 GB of orphaned bytes daily | Rules out upload sessions without an expiry, and rules out reclaiming space manually |
| 23 sessions per second times a 10.7 second mean duration | about 246 concurrent sessions, with tail sessions lasting 89 minutes | Rules out holding session state in instance memory with sticky routing, because a tail session outlives a deploy |
| Eleven nines of durability | Erasure coding across failure domains plus continuous background verification | Rules out storing bytes on disks you operate, for any team that has other work to do |

!!! tip "Say this out loud in the room"

    "Twenty-three uploads per second is nothing. Eighty-nine minutes to send one file is everything. I am going to design the protocol for the slow client, and let the request rate take care of itself."

### V0, the simplest thing that works (files)

**Predict first.** V0 enforces the 32 MB launch cap. What is the first number that tells you the cap has to go?

??? note "Show answer"

    The rejection rate. Once mobile clients ship, a measurable share of upload attempts exceed 32 MB and receive 413. That number is a product signal, and it arrives before any latency or capacity signal does, which is the point: the contract fails before the infrastructure does.

```
+---------+     +------------------+     +----------------+
| Client  | --> | Files API        | --> | Object store   |
|         | <-- | streams body     | <-- | bytes          |
+---------+     | 32 MB limit      |     +----------------+
                +------------------+
                        |
                        v
                +------------------+
                | Postgres         |
                | file metadata    |
                +------------------+
```

**Caption.** V0 has four components. A client sends a whole file in one request to the files service, which streams it into an object store and writes a metadata row in Postgres. Downloads are proxied back through the same service.

Why this is correct at the launch scale:

- 69 uploads per second and 278 MB per second of ingest is three instances of work, comfortably inside a small fleet.
- A 32 MB file at 3 Mbps takes 85 seconds, which fits inside the 350 second idle timeout with margin, so a single request genuinely works.
- The contract says the limit out loud and returns 413 with a machine-readable code. A design that restricts itself honestly is better than one that accepts a request it cannot finish.
- Metadata and bytes are already separate resources, which is the one decision that makes every later version an addition rather than a rewrite.

!!! warning "When not to use even this much"

    If files are under about 1 MB and there are fewer than a few million of them, put the bytes in the database next to the metadata. One store, one transaction, one backup, and no consistency window where a metadata row points at an object that is not there yet. The measurement that justifies a separate object store is database size growth dominated by blob columns, or a restore time that exceeds your recovery target.

### Breaking point, then V1 (files)

**Breaking point one: the upload success cliff at 131 MB.**

What fails: once the limit is raised for mobile clients, uploads succeed for small files and fail for large ones, with the boundary at the file size whose transfer time equals the shortest idle timeout in the path. At 3 Mbps and 350 seconds that is 131 MB. Above it, the connection is closed mid-body, the client retries from zero, and on a 2 GB file the retry also fails, so the file is simply never uploadable.

How you observe it: upload success rate bucketed by file size, on a log scale. A healthy system is flat. This one is flat to 131 MB and then falls off a shelf. Also watch bytes transferred divided by bytes stored: when clients restart from zero, that ratio climbs above 1 and keeps climbing.

```
+---------+     +------------------+     +----------------+
| Client  | --> | Files API        | --> | Postgres       |
|         | <-- | session state    | <-- | uploads        |
+---------+     | offset authority |     | files          |
     |          +------------------+     +----------------+
     |                   |
     |                   v
     |          +------------------+
     +--------> | Object store     |
      chunks    | parts and bytes  |
                +------------------+
```

**Caption.** V1 has four components. The client creates an upload session, sends bytes in chunks, and asks the service for the current offset after any interruption. The service is the sole authority on how many bytes are durable, and it records session state in Postgres rather than in instance memory.

V1 changes, each with the observation that justified it:

| Change | Justified by | Cost you accept |
|---|---|---|
| Upload session resource | Success rate cliff at 131 MB | A second state machine, with its own creation, expiry and garbage |
| Chunks of 4 to 32 MB | 85 seconds lost per disconnect at the top of the band, 102 seconds of overhead at the bottom | More requests per file, and a per-chunk failure path |
| Server-authoritative offset | Clients cannot know what was durably written before the socket died | An extra round trip on every resume, which is cheap against 89 minutes |
| Session state in Postgres | Tail sessions last 89 minutes and outlive a deploy | Session reads and writes now cost a database round trip |
| Whole-file digest at completion | Restart-from-zero clients were silently producing truncated files | Completion is no longer free; you verify before you acknowledge |
| Seven day session expiry | 200 GB per day of orphaned partial bytes | A client that pauses for a week loses its progress, which the contract must state |

**Breaking point two: 22 Gbps of egress through the request tier.**

What fails: proxying downloads needs 22 instances at peak whose only job is copying bytes. Worse, those instances also serve metadata calls, so a handful of 2 GB downloads occupy connections for minutes and small requests queue behind them. Metadata p99 starts tracking download volume, which is a dependency nobody designed and nobody expects.

How you observe it: the fraction of instance CPU spent in encryption and copying, and the correlation between metadata request p99 and concurrent large transfers. If those two lines move together, the byte path and the control path are sharing a resource they should not share.

V2 changes:

- Downloads are delegated. The service issues a short-lived, scope-limited token and the bytes come from an edge cache in front of the object store. The mechanics and the cost of that delegation are deep dive two.
- Range requests are supported end to end, so a client resuming a download does not restart it. RFC 9110, [HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) (June 2022, accessed August 2026), defines the range semantics in section 14.
- Lifecycle rules become a resource on the container rather than a background script, so expiry is inspectable, testable and auditable by the customer.

```
+---------+     +------------------+     +----------------+
| Client  | --> | Files API        | --> | Postgres       |
|         | <-- | issues download  | <-- | files          |
+---------+     | tokens           |     +----------------+
     |          +------------------+
     |
     |          +------------------+     +----------------+
     +--------> | Edge cache       | --> | Object store   |
      bytes     | validates token  | <-- | bytes          |
                +------------------+     +----------------+
```

**Caption.** V2 has five components. The files service issues a short-lived download token and never touches the bytes. The client presents the token to an edge cache, which validates it and reads from the object store on a miss.

!!! warning "When not to delegate the byte path"

    Below roughly 1 Gbps of egress, or when every single access must appear in your own audit log within seconds and be revocable within seconds, keep proxying. Delegation moves your access log into someone else's system and makes revocation a matter of token lifetime rather than a decision. The measurement that justifies delegating is instance CPU spent copying bytes exceeding about 30 percent, or metadata p99 correlating with transfer volume.

### Deep dive one: designing a resumable upload that survives a tunnel

**Start from the failure, not the protocol.** The client is on a train. The socket dies at an unknown byte. The only question the protocol has to answer well is: where do I start again, and who decides?

**The server decides, and this is the whole design.** The client cannot know how many bytes were durably committed, because the bytes it wrote into a socket may have been buffered, discarded or half written. So the session exposes a read that returns the current offset, and the client always asks before resuming. Any protocol where the client asserts the offset is a protocol that produces silently corrupted files.

| Operation | What it does | Why it exists |
|---|---|---|
| Create session | Returns a session identifier, a location, an expiry and the accepted chunk range | Gives the upload an identity that outlives any one connection |
| Read session | Returns the durable offset and the total expected size | The single source of truth for where to resume |
| Append chunk | Sends bytes at a stated offset, conditional on that offset being current | Makes a stale or duplicated chunk fail loudly instead of creating a gap |
| Complete | Supplies the whole-file digest, returns the file resource | The only point at which corruption can be detected |
| Abort | Releases the session and its partial bytes | Lets a client clean up instead of waiting seven days |

**Commit the offset after the bytes are durable, never before.** If the offset advances first and the process dies, the client resumes past a gap and the file is corrupt in a way no retry will fix and no error will report. This ordering is the single most important line in the design and it costs nothing to say.

**Make appends conditional.** A chunk carries the offset it believes it is writing at, and the server rejects it with 409 if that is not the current offset. This turns three separate hazards into one loud error: a duplicated chunk from a retry, a reordered chunk from a client with parallel writers, and a second client appending to the same session.

**Integrity is checked twice, for different reasons.** A per-chunk digest catches a corrupted transfer immediately, so the client re-sends 8 MB instead of 2 GB. A whole-file digest at completion catches an assembly error, which per-chunk digests cannot. RFC 9530, [Digest Fields](https://www.rfc-editor.org/rfc/rfc9530.html) (February 2024, accessed August 2026), defines `Content-Digest` for the chunk and `Repr-Digest` for the assembled representation, so you do not need to invent a header.

**Completion is naturally idempotent here, and that is why it differs from case study 1.** In the payments interface the client had to supply a retry key, because two charge requests are genuinely two intents. Here the session identifier already is the key: completing an existing session twice returns the same file resource, because the session names exactly one outcome. Do not add a second key mechanism when the resource already provides one.

**Expiry is derived, not chosen by taste.** From the estimation table, 200 GB of orphaned partial bytes per day. A seven day expiry caps the leak at about 1.4 TB, which is under 0.05 percent of annual storage, and seven days covers a client that pauses over a weekend. State the number in the contract so a client can plan around it.

**One thing not to expose.** Do not put the object store's own part numbers, part sizes or upload identifiers in your contract. The moment a client sends you a vendor part number, changing storage vendor is a breaking API change. Your session identifier is yours; the vendor's is an implementation detail.

The tus protocol, version 1.0.0, released 25 March 2016 ([tus.io resumable upload protocol](https://tus.io/protocols/resumable-upload), accessed August 2026), defines `Upload-Offset` and `Upload-Length` with almost exactly this shape, and is worth reading as prior art before inventing headers.

!!! warning "When not to use a session protocol"

    If the 99th percentile file size is under about 10 MB and clients are servers on reliable networks, a single request with a retry is simpler and has fewer states. The session adds a resource, an expiry job, an orphan class and a second way to corrupt a file. The measurement that justifies it is upload success rate showing any cliff when bucketed by size, or a bytes-transferred to bytes-stored ratio above about 1.1.

### Deep dive two: who may read these bytes, and for how long

The dives on this page vary by problem. Delegation is interesting here and nowhere else on this page, because this is the only case study where the payload is large enough that carrying it yourself is the dominant cost.

**The delegation ladder, compared on what it costs you.**

| Approach | Revocation latency | Your egress cost | Audit fidelity | Range support |
|---|---|---|---|---|
| Proxy every byte | Immediate, per request | Full, 22 Gbps at peak | Complete, per byte range | You implement it |
| Redirect to a signed storage URL | Only when the token expires | None after the redirect | Only issuance, not use | Storage provides it |
| Edge cache validating a token | Only when the token expires | None, plus a cache hit ratio | Edge logs, on a delay | Edge provides it |
| Public URL with an unguessable name | Never | None | None | Edge provides it |

**A signed token freezes the authorisation decision at issue time.** That is the sentence to say out loud, because everything else follows from it. You cannot revoke a token that is already out; you can only make it short lived, or rotate the signing key and invalidate every outstanding token at once, which is an outage you inflicted on yourself.

**The lifetime is derived from the download, not chosen round.** A 2 GB file at 3 Mbps takes 89 minutes. If the token is validated only when the transfer starts, a five minute lifetime works and the transfer continues past expiry. If it is validated on every range request, a five minute lifetime breaks every resumed download. Pick the first, say which you picked, and document that a client receiving 403 mid-download should request a fresh token rather than surface an error.

**Scope the token as narrowly as the use allows.**

| Token field | Why it is there | What happens without it |
|---|---|---|
| Object identity | The token grants one file, not a prefix | A leaked token becomes a leaked container |
| Method | Read only, never write | A read link becomes an upload endpoint |
| Expiry | Bounds the damage of a leak | The leak is permanent |
| Signing key identifier | Lets you rotate keys per tenant | Rotation invalidates every customer's tokens at once |
| Optional response headers | Fixes the filename and content type | Clients get the storage vendor's defaults, which leak the vendor |

Do not bind the token to a client address. Mobile clients change address on every handoff, and you will have built a system that fails specifically for the users who most need resumability.

**Caching and the identity trap.** If every user receives a uniquely signed URL for the same popular file, the edge cache sees a distinct key per user and the hit ratio goes to zero. This is cache key fragmentation, and it turns a delegated design back into an origin-bandwidth design without anyone noticing. The fix is to keep the token out of the cache key: put it in a header or a cookie the edge validates but does not key on, and let the object path be the key.

**Sharing is a resource, not a URL.** When a customer wants to share a file, create an access grant with its own identifier, expiry, optional password and revocation, and mint tokens from it. An unguessable URL cannot be revoked, cannot be audited and cannot be listed. A grant can be all three, and "show me everything I have shared" is a feature customers ask for within a month.

**What you give up.** Once bytes leave through the edge, your own logs record issuance rather than use, so per-access auditing depends on an export from the edge on a delay. If a compliance requirement says every read must be logged within seconds, that requirement alone rules out delegation, regardless of the bandwidth arithmetic.

!!! warning "When not to issue signed tokens"

    If files are public and small, skip tokens entirely and serve from the edge with a long cache lifetime; a token adds signing, rotation, clock sensitivity and a 403 path for nothing. The measurement that justifies tokens is any requirement that two different users see different results for the same object path.

### Failure modes and evaluation (files)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Silent corruption | An offset advanced before the bytes were durable, so a resume skipped a range | Commit ordering, conditional appends, and a whole-file digest that fails completion |
| Storage leak | Abandoned sessions accumulate partial bytes at 200 GB per day | Seven day expiry with a reclamation job, and orphaned bytes as a published indicator |
| Cache key fragmentation | Per-user signed URLs give every viewer a distinct cache key | Keep the token out of the cache key and let the object path be the key |
| Thundering herd | A token expires for many concurrent viewers and they all request a fresh one at once | Refresh early with a random offset, and make token issuance a cheap local operation |
| Retry storm | Chunk failures trigger immediate client retries, which add load to an already slow path | Bounded retries with full jitter, and a per-session retry budget the server can signal with `Retry-After` |
| Head-of-line blocking | Large transfers occupy connections on instances that also serve metadata | Separate the byte path from the control path, which is exactly what V2 does |
| Clock skew | A client with a fast clock rejects a token it should accept, or presents one the edge considers expired | Publish the tolerance, and return a distinct error code naming skew rather than a generic 403 |
| Hot object | One file is downloaded by an enormous audience at once | Edge caching handles it, but only if the previous cache key problem was solved |

Service level indicators to publish:

- Upload success rate bucketed by file size, on a log scale. This is the indicator that finds the protocol bugs.
- Bytes transferred divided by bytes stored. Above 1.1 means clients are re-sending, which means resumption is not working.
- Resume rate, the fraction of sessions that used the offset read at least once. A value near zero on mobile means clients are not implementing the protocol.
- Completion digest mismatch rate, which should be indistinguishable from zero.
- Time to first byte on download at p50 and p99, split by cache hit and miss.
- Orphaned session bytes, and the age of the oldest unexpired session.
- Token rejection rate, split by expired, wrong scope and bad signature.

Alert on completion digest mismatches at any sustained non-zero rate, because that is corruption and not load. Alert on any size bucket whose upload success rate drops below the fleet rate by a stated margin, which catches a cliff before customers report it. Alert on orphaned bytes growth rather than on the absolute number. Do not alert on cache hit ratio; review it weekly, since it moves with content mix.

Re-checking against the non-functional requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Files up to 2 GB on a 3 Mbps mobile link | Yes, because no single request has to survive 89 minutes |
| 278 MB per second of ingest at peak | Yes, and ingest still passes through the service, which is a deliberate choice so that scanning can be added later |
| 22 Gbps of egress at peak | Yes, and it costs the service nothing, at the price of audit fidelity |
| Time to first byte under 200 ms at p99 | Yes on an edge hit; a miss depends on the object store and should be reported separately |
| Eleven nines durability | Yes, inherited from the object store, which is why you did not build it |
| 2.9 PB per year of growth | Yes, provided lifecycle rules exist and customers actually configure them, which is a product problem as much as a design one |

### Where this answer is a simplification (files)

- **Scanning and transcoding add a non-ready state to every file.** The moment bytes must be processed before they are readable, the file resource needs a state, every client must handle it, and adding it after launch is a breaking change. Design the state in even if the pipeline comes later.
- **Deduplication by content hash leaks information.** If uploading a file that already exists returns instantly, an attacker learns that someone else holds that exact file. The mitigation is per-tenant deduplication, which forfeits most of the saving.
- **Client-side encryption changes who you are.** If the customer holds the keys, you can no longer scan, transcode, deduplicate or generate previews, and range requests interact awkwardly with block ciphers. That is a product decision wearing a cryptography costume.
- **Data residency constrains the byte path.** If bytes may not leave a region, your edge strategy, your replication and your disaster recovery all change, and the contract has to expose which region a container lives in.
- **Egress is the bill.** For most storage products the dominant cost is bytes leaving, not bytes stored, which is why pricing, quotas and cache hit ratio are design inputs rather than finance concerns.
- **Primary sources with dates.** RFC 9110 ([HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html), June 2022) for range requests in section 14 and preconditions in section 13. RFC 9530 ([Digest Fields](https://www.rfc-editor.org/rfc/rfc9530.html), February 2024) for `Content-Digest` and `Repr-Digest`. The [tus resumable upload protocol](https://tus.io/protocols/resumable-upload), version 1.0.0, released 25 March 2016. All accessed August 2026. For the object store internals underneath this design, see the storage building blocks in [Modern System Design](modern-system-design.md).

### Level delta (files)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Large uploads | Suggests chunking the file | Designs a session with a server-authoritative offset and derives the chunk band | Derives the 131 MB cliff from the shortest timeout in the path and designs for the hop nobody controls |
| Integrity | Mentions a checksum | Checks per chunk and again on completion, and says what each catches | Fixes the commit ordering so an offset never advances ahead of durability, and calls that the corruption-prevention step |
| Byte path | Proxies everything | Delegates downloads to a signed URL after computing 22 Gbps | Names the audit and revocation cost of delegation, and states the requirement that would reverse the decision |
| Caching | Says "put a CDN in front" | Sizes the benefit and enables range requests through the edge | Catches cache key fragmentation from per-user tokens before it happens |
| Contract limits | Accepts whatever is sent | Returns 413 with a machine-readable code and documents the cap | Ships the cap deliberately in V0 as an honest contract, and plans the removal path so it is additive |
| Lifecycle | Adds a cleanup script | Derives a seven day session expiry from orphaned bytes per day | Makes lifecycle a customer-visible resource so expiry is testable and auditable rather than a background job |

What each level added, in one line each:

- **Mid to senior:** the protocol is derived from the slow client rather than from the request rate, and every threshold has arithmetic behind it.
- **Senior to staff:** the ordering of durable writes becomes explicit, and the costs that delegation moves onto other teams are named rather than assumed away.

---
## Case study 4: Search service

Candidates answer this prompt with an index and a ranking function. Neither is the interview. The interview is which parts of your engine you accidentally promised to keep forever, and what happens to a client paging through a list that is being rewritten underneath it.

### The prompt (search)

"Design the interface product teams use to search our catalogue: query, filter, facet, sort and page through results."

### Questions that fork the design (search)

| Question worth asking | If the answer is A the design becomes | If the answer is B it becomes |
|---|---|---|
| Is the query one free-text string, or an expression clients construct? | Free text plus named filters: you own the parser, so you can rewrite it whenever you like | An expression grammar: clients build queries, so your grammar is a public contract you can extend but never simplify |
| Is relevance exposed to clients? | Hidden: rank order only, so a model change ships on a Tuesday | Exposed as scores or explanations: the score is now a contract, and every model change is a breaking change |
| Are results the same for everyone? | Global: results cache, pages are shareable, and one computation serves many | Per viewer, because of permissions or personalisation: shared caching dies and paging is computed per session |
| How fresh must the index be? | Minutes: batch builds, stable result sets, easy paging | Seconds: continuous indexing, so the result set changes while a client walks it |
| Do clients need an exact total? | Approximate: cap the count and say so in the response | Exact: you pay a full match count on every query, which dominates cost on broad queries |
| Is deep paging a genuine use case or an artefact? | Genuine, for export: give it a separate interface with different guarantees and a different rate limit | An artefact of bots and of clients working around a missing filter: cap depth and make the cap part of the contract |
| Are facets needed on high-cardinality fields? | Low cardinality only: facets are cheap counting | High cardinality: a facet can cost more than the search, and needs its own budget and its own limits |
| How many client applications depend on this? | One or two, same company, shipped together: you can change the surface with a coordinated release | Dozens, including shipped mobile apps: every surface you expose has to survive independent release cycles |

### Requirements (search)

Functional:

- Search the catalogue with a text query and a set of typed filters.
- Sort by relevance or by a named business ordering.
- Return facet counts for an agreed set of fields.
- Page through results without duplicates or gaps within one session.
- Export a large result set with different guarantees from interactive search.
- Report how many results matched, honestly, including when the answer is capped.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Queries per second | 3,000 average, 9,000 at the 3x peak | ASSUMED, a busy catalogue on a retail evening; ask, because it decides the fan-out |
| Documents indexed | 200 million, 2 KB each | ASSUMED, a large catalogue including variants |
| Index size relative to source | 25 percent, in a range of 20 to 40 percent | ASSUMED, compressed postings plus a modest set of stored fields land in that band |
| Node memory available for page cache | 64 GB | ASSUMED, a commodity search node |
| Document update rate | 2 percent of the corpus per hour | ASSUMED, prices and stock levels move constantly |
| Index freshness target | 5 seconds | ASSUMED, a price change that is invisible for a minute produces a support ticket |
| Search latency, p99 | 250 ms server side | ASSUMED, a search box that hesitates loses the session |
| Faceted share of queries | 30 percent | ASSUMED, facets appear on category pages but not on the type-ahead path |
| Client applications depending on the surface | 40 | ASSUMED, this is what makes an exposed internal a permanent one |
| Contract stability | query surface valid for 24 months | GIVEN, shipped mobile clients cannot be forced to upgrade |
| Concurrent queries in flight at peak | 2,250 | DERIVED below |
| Index footprint | 100 GB, four shards of 25 GB | DERIVED below |
| Document changes between two page fetches | about 2,200 | DERIVED below |

### Estimation that eliminates (search)

| Calculation | Result | What it rules out |
|---|---|---|
| 9,000 queries per second times a 0.25 second budget | 2,250 queries in flight at peak | Rules out a single node, and rules out any per-query operation measured in whole seconds, because 2,250 concurrent units of work is the ceiling on everything |
| 200 million times 2 KB, times 25 percent | 400 GB of source, 100 GB of index | Rules out one 64 GB node serving from memory, so the index is split into four shards of 25 GB that each stay resident |
| Page size 20 at offset 100,000, across 4 shards | each shard collects 100,020 candidates, the coordinator merges 400,080 | Rules out unbounded offset paging, because the work grows with how far the reader has walked rather than with what they asked for |
| A broad query matching 40 million documents, at an assumed 50 million postings scanned per second per core | 0.8 seconds per core just to count | Rules out an exact total on every query inside a 250 ms budget, so counts are capped and the cap is stated in the response |
| A facet on a 5 million value field: 5 million counters at 8 bytes | 40 MB per shard per query; at 2,700 faceted queries per second across 4 shards that is 432 GB per second of allocation | Rules out letting clients choose arbitrary facet fields, and rules out facets without a top-N limit |
| 2 percent of 200 million per hour, over a 2 second gap between page requests | about 2,200 documents changed | Rules out offset paging even at shallow depth, because documents entering above the reader's position push results down and repeat them |
| For a 4 shard request to hit 250 ms at p99, each shard may exceed it with probability at most 1 minus 0.99 to the power one quarter | 0.25 percent, so each shard needs a p99.75 rather than a p99 | Rules out sizing shard latency to the request budget, and forces either a tighter shard budget or hedged requests |
| A 10 minute pinned view holding 2 percent per hour times one sixth of an hour, of a 100 GB index | about 0.33 GB retained per view; 1,000 concurrent views is 333 GB | Rules out unlimited or long-lived pinned views, since the retained segments would exceed three times the index |
| 40 client applications, each depending on engine internals | 40 coordinated releases for one engine upgrade | Rules out passing the engine's native query language through to clients, at any scale, forever |

!!! tip "Say this out loud in the room"

    "Two thousand two hundred documents change between the moment they see page one and the moment they ask for page two. That number, not the index size, is what decides the pagination contract, so I want to design that first."

### V0, the simplest thing that works (search)

**Predict first.** V0 is already sharded because the arithmetic forced it. So where is V0 actually simple?

??? note "Show answer"

    In the contract. V0 offers one text field, a fixed list of filters, one sort, no facets, no exact counts and a hard depth cap. Every simplification is in what the interface promises, not in what the infrastructure runs. That is the right place for a contract round to economise.

```
+-----------+                 +-----------------+
| Catalogue | --------------> | Indexer         |
| database  |                 | batch every 5 m |
+-----------+                 +-----------------+
                                       |
                                       v
+-----------+     +-----------+     +-----------------+
| Client    | --> | Search    | --> | Index shards    |
|           | <-- | API       | <-- | 4 x 25 GB       |
+-----------+     +-----------+     +-----------------+
```

**Caption.** V0 has four components. An indexer reads the catalogue on a five minute cycle and writes four shards of 25 GB each. A search service parses and validates the request, fans out to all four shards, merges the results and returns them.

Why this is correct at the stated scale:

- Four resident shards absorb 9,000 queries per second of fan-out, because each shard sees the full query rate but only a quarter of the index.
- Depth is capped at 1,000 results, so the coordinator merges at most 4,000 candidates rather than 400,080.
- Counts are reported as capped at 10,000, with a field saying so, rather than as an exact number nobody can afford.
- The query surface is one text field plus an enumerated list of filters and sorts. Nothing about the engine appears in the request or the response.

!!! warning "When not to shard at all"

    Below roughly 20 million documents, or any index that fits in one node's page cache with room for merges, a single node with replicas is strictly better: no fan-out, no straggler, no merge, and the request p99 equals the node p99. The measurement that justifies sharding is resident index size approaching available page cache, or a query latency profile dominated by disk reads rather than by work.

### Breaking point, then V1 (search)

**Breaking point one: the result set moves while the client walks it, from the first page onwards.**

What fails: 2,200 documents change between two page requests. A document that becomes newly matching and ranks above the reader's position shifts everything down by one, so an item from page one reappears on page two. A document that stops matching shifts everything up, so an item is never shown. Neither depends on how deep the reader has gone, so the depth cap does not help.

How you observe it: a synthetic client that walks a live index to the depth cap and asserts no repeated and no missing identifier. Report the duplicate rate per hundred results. A second signal is the correlation between that rate and the indexing rate, which is what tells you it is churn and not a bug in the merge.

V1 changes:

| Change | Justified by | Cost you accept |
|---|---|---|
| Cursor carrying the last result's sort values plus a unique tiebreaker | 2,200 documents changing between fetches | The client can no longer jump to an arbitrary page, which some clients will complain about |
| Depth cap stated in the contract, with a distinct error at the boundary | 400,080 candidates merged at offset 100,000 | Someone genuinely needed depth, and now needs a different interface |
| Counts reported as a value plus a relation, "at least 10,000" | 0.8 seconds of counting on a broad query | Clients render an approximate number, which product managers dislike until they see the latency |
| A separate export interface with its own rate limit and its own objective | Deep paging was a real use case for one caller, not a bug | A second surface to document, secure and operate |

**Breaking point two: one slow shard sets the request latency, and one facet can exhaust a node.**

What fails: two independent things, both invisible in average latency.

- Fan-out tail amplification. With four shards, a request is only as fast as its slowest shard. For the request p99 to be 250 ms, each shard must stay under 250 ms 99.75 percent of the time. Shard-level dashboards showing a healthy p99 will look fine while the request p99 sits well above budget.
- Unbounded facet work. A single query faceting a 5 million value field allocates 40 MB per shard. A handful of such queries arriving together drives a shard into garbage collection or out of memory, which then makes it the straggler in every other request it is serving.

How you observe it: the fraction of requests whose latency is set by a single straggler shard, computed as request latency minus the median shard latency; and per-query peak allocation grouped by facet field. If the straggler is always the same shard, it is a hot shard. If it moves, it is contention from expensive queries.

```
+-----------+     +-----------------+     +-----------------+
| Client    | --> | Search API      | --> | Index shards    |
|           | <-- | cost budget and | <-- | 4 x 25 GB,      |
+-----------+     | hedged fan-out  |     | 2 replicas each |
                  +-----------------+     +-----------------+
                          |
                          v
                  +-----------------+
                  | View registry   |
                  | pinned readers  |
                  +-----------------+
```

**Caption.** V2 has four components. The search service estimates the cost of a request before running it, fans out with a hedge to a second replica when a shard is slow, and keeps a registry of pinned readers so a session can hold a stable view of the index for a bounded time.

V2 changes:

- Every shard has two replicas, and the coordinator sends a second copy of the request to the other replica once a shard exceeds the 95th percentile of its own recent latency. That converts a straggler into an extra unit of load rather than a slow response.
- Requests carry an estimated cost, computed from the facet fields requested, the number of filter clauses and the depth. Requests above the budget are rejected with a machine-readable code that names the offending parameter, rather than accepted and allowed to hurt everyone.
- Facet fields come from an allow-list and always carry a top-N limit, so the 40 MB per shard case cannot be requested.
- A view registry holds pinned readers with a lifetime and a concurrency cap, derived from the 0.33 GB per view arithmetic. This is the mechanism behind the stable-paging option in deep dive two.

!!! warning "When not to hedge requests"

    Hedging costs you a duplicate of every slow request, so at a straggler rate above roughly 10 percent it adds more load than it removes and can push a struggling shard over. Below a fan-out of about three shards, the tail amplification is small enough that fixing the slow shard is cheaper than hiding it. The measurement that justifies hedging is a request p99 materially worse than the median shard p99, with no single shard consistently at fault.

### Deep dive one: the query surface you can still change in two years

**There are three levels of query contract, and the choice is irreversible in one direction.**

| Level | What the client sends | Your freedom afterwards | Characteristic failure |
|---|---|---|---|
| Opaque string plus named parameters | `q=running shoes`, `filter=brand:acme`, `sort=price_asc` | You can replace the parser, the analyser and the engine | Clients encode operators into the free-text string to get capabilities you did not give them |
| Typed expression grammar you defined | A small closed language with typed operators and a published field schema | You can add operators; you can never remove one | The grammar grows by request until it is a database query language nobody designed |
| The engine's native query language | Whatever the engine accepts | None. An engine upgrade is a breaking API change across 40 clients | An engine deprecation becomes a company-wide migration |

**The rule that generates the right answer: expose intent, not implementation.** `sort=price_asc` is an intent. `sort=price:asc,_score:desc` is an implementation, and the moment a client sends it you have promised that a thing called `_score` exists and behaves that way. Enumerate sorts by name and you can change what each one means internally.

**Validate strictly, and say why.** Silently ignoring an unknown filter is the worst possible behaviour: a client typo means the request returns unfiltered results, which looks like a relevance problem and gets debugged for a week. Reject unknown fields and unknown operators with a problem document that lists the valid ones, using RFC 9457, [Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html) (July 2023, accessed August 2026).

**Design the evolution path before you need it.**

| Mechanism | What it buys you |
|---|---|
| Additive-only operators, never removals within a version | A client written today keeps working when you add capability tomorrow |
| Per-operator, per-client usage telemetry | Deprecation becomes evidence: you can see that two clients use an operator, and talk to them |
| A strict mode that rejects anything unrecognised | You can discover whether a parameter is used at all before you consider removing it |
| Deprecation and sunset signalling on responses | Clients learn from the wire, not from an email, using RFC 9745 and RFC 8594 |

RFC 9745, [The Deprecation HTTP Response Header Field](https://www.rfc-editor.org/rfc/rfc9745.html) (March 2025, accessed August 2026), and RFC 8594, [The Sunset HTTP Header Field](https://www.rfc-editor.org/rfc/rfc8594.html) (May 2019, accessed August 2026), give you a standard way to say "this is deprecated" and "this stops working on this date" without inventing headers your clients then have to learn.

**Relevance exposure is part of the query surface, not a separate topic.** If you return a numeric score, clients will threshold on it, and your next ranking change will move every threshold at once. Return rank order, and if callers need to know why an item ranked where it did, offer a separate explanation endpoint that is explicitly labelled as diagnostic, unstable and not for production logic.

**The cost of getting this wrong is asymmetric.** Adding an operator later is a day of work. Removing one, across 40 shipped clients on independent release cycles, is a year. Design as if every field you expose is permanent, because for practical purposes it is.

!!! warning "When not to build a query grammar"

    With fewer than about ten filterable fields and a single client team shipping alongside you, named parameters are strictly better: simpler to validate, simpler to document, impossible to abuse. The measurement that justifies a grammar is either distinct filter combinations in production numbering in the hundreds, or a rising rate of queries containing operator syntax inside the free-text field, which is clients telling you what is missing.

### Deep dive two: paging a ranked set that will not hold still

**Why this differs from the cursor dive in case study 2.** There the sort key was a creation timestamp: immutable, owned by the row, and never recomputed. Here the sort key is a relevance score computed at query time by a model that changes, over an index that changes. A cursor that stores "the last row's sort value" is meaningless if the function producing that value is different on the next request. Same word, different problem, different mechanism.

**Offer three named guarantees, and make the client pick.**

| Guarantee | Mechanism | Cost | Who it is for |
|---|---|---|---|
| Best effort | Cursor over the current index; results may shift | None | Ordinary interactive search, which is almost everyone |
| Stable within a session | A pinned reader with a lifetime, created explicitly | 0.33 GB of retained segments per ten minute view | A user comparing results across pages, or a checkout flow |
| Stable until complete | Materialise the matching identifiers once, then page the materialised list | Storage proportional to the result size, and a long-running job | Export, reconciliation, bulk operations |

**The cursor needs a unique final sort key or it is broken.** Ranked results routinely tie: many documents share a score. Without a unique tiebreaker appended to the sort, "everything after score 4.21" is ambiguous and the same document can appear on both sides of the boundary. Append the document identifier as the last sort key, always. Apache Lucene's `IndexSearcher.searchAfter` takes exactly this shape, a previous result used as the position marker, and is documented in the [Lucene 10.0.0 IndexSearcher javadoc](https://lucene.apache.org/core/10_0_0/core/org/apache/lucene/search/IndexSearcher.html) (accessed August 2026).

**Put the ranking model version inside the cursor.** When a model rolls out mid-session, the scores change, and a cursor issued under the old model addresses a position that no longer exists. Detect it and return a distinct error telling the client to restart the list. The alternative is silent duplicates and gaps that nobody can reproduce, because they only happen during a deploy.

**Make the pinned view a resource, not a hidden mode.** Create it, get an identifier and a lifetime, use it on subsequent searches, delete it when finished. Three reasons this beats an implicit session:

- The cost is visible, so you can cap concurrency and expire the oldest, using the 333 GB arithmetic from the estimation table.
- The client's intent is explicit, so you can meter or price it.
- A leaked view is a listable, expirable object rather than an invisible reference holding segments alive.

**Total counts belong in the response shape, not in a number.** Return a structure with a value and a relation, so "12,431 exactly" and "at least 10,000" are the same shape. That one decision lets you change how counting works later without a breaking change, and it stops clients from rendering a precise-looking number you never promised.

**Deep paging gets its own door.** The export interface takes the same query, returns results in an order that is cheap rather than relevant, usually the document identifier, streams them, and has its own rate limit and its own latency objective. Splitting it removes the pressure that would otherwise push you into supporting offset 100,000 on the interactive path.

!!! warning "When not to offer pinned views"

    If the index refreshes less often than a typical session lasts, say a build every fifteen minutes, a plain cursor already gives a stable result set and the pinned view adds a resource, a leak and a capacity ceiling for nothing. The measurement that justifies it is a synthetic paginator observing duplicates or gaps at a rate your product cannot tolerate, at the depths clients actually reach.

### Failure modes and evaluation (search)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Tail amplification | The slowest of four shards sets every request's latency | Hedge to a second replica past a shard's own 95th percentile |
| Hot shard | One shard is consistently the straggler because of a skewed term or an unbalanced split | Rebalance by document count and check term distribution, not just shard size |
| Resource exhaustion | A high-cardinality facet allocates 40 MB per shard per query | Facet allow-list with a top-N limit, and a pre-execution cost estimate |
| Metastable failure | Slow queries cause client retries, which add load, which slow queries further | Reject above a cost budget at admission, and cap retries as a fraction of successful traffic |
| Thundering herd | A segment refresh invalidates per-query caches and every request recomputes together | Stagger refreshes across replicas so they never invalidate in lockstep |
| Cache key fragmentation | Per-viewer permission filters make every result set unique | Cache the expensive shared part, the text match, separately from the cheap per-viewer filter |
| Stale reader leak | Pinned views are created and never deleted, holding segments alive | Lifetime on every view, a concurrency cap, and eviction of the oldest |
| Silent contract drift | An unknown filter is ignored, so results are unfiltered and look merely irrelevant | Strict validation with a problem document naming the unknown field |

Service level indicators to publish:

- Search latency at p50, p95 and p99, reported separately for filtered, broad and faceted queries, because they are different workloads sharing an endpoint.
- Straggler share: the fraction of requests whose latency exceeds the median shard latency by a stated margin.
- Hedge rate, and hedge win rate. A high hedge rate with a low win rate means you are adding load for nothing.
- Pagination integrity from a synthetic client walking a live index to the depth cap.
- Cursor rejection rate, split by expired, model version changed, and filter mismatch.
- Capped-count rate, the share of responses where the total was reported as a lower bound.
- Index freshness lag, measured as the age of the newest document visible to search.
- Open pinned views and the age of the oldest.

Alert on index freshness lag, because a stale index returns confident wrong answers rather than errors and no other indicator catches it. Alert on straggler share and on open pinned views. Alert on error budget burn for the search path. Do not alert on zero-result rate: it is a product metric that moves with the catalogue and with seasonality, and it belongs in a weekly review.

Re-checking against the non-functional requirements:

| Requirement | Does V2 meet it? |
|---|---|
| 9,000 queries per second at peak | Yes, with four shards at two replicas each and admission control on expensive queries |
| p99 under 250 ms | Yes with hedging, which is what closes the gap between the shard budget and the request budget |
| 5 second index freshness | Yes for best-effort search; explicitly not true inside a pinned view, and the contract says so |
| Honest counts | Yes, since the response shape distinguishes an exact value from a lower bound |
| No duplicates or gaps within a session | Yes with a pinned view; best effort otherwise, and that is a named, chosen guarantee rather than an accident |
| 24 month query surface stability | Yes, provided no engine internal was ever exposed. This is the requirement that constrains the most decisions and shows up in none of the boxes |

### Where this answer is a simplification (search)

- **Relevance quality is a separate discipline and a separate interview.** Everything above is the contract around ranking, not the ranking. Feature engineering, training data, offline evaluation and online experimentation belong in [ML System Design](ml-system-design.md).
- **Language handling is not a checkbox.** Analysers, stemming, compound splitting and script handling differ per language, and the choice is baked into the index at build time, which means changing it is a full reindex rather than a configuration change.
- **Query understanding sits in front of everything here.** Spelling correction, entity recognition and intent classification rewrite the query before it reaches the index, and each rewrite is a place where the results the user sees stop corresponding to what they typed.
- **Experimentation makes the contract harder.** If two users get two ranking models, then results are per viewer, shared caching is gone, and cursors must carry the assignment or a user can flip models mid-list.
- **Personalisation collapses the cache and the pagination story at once.** Every mitigation in this design assumes many users share a result set.
- **Primary sources with dates.** The [Lucene 10.0.0 IndexSearcher javadoc](https://lucene.apache.org/core/10_0_0/core/org/apache/lucene/search/IndexSearcher.html) documents `searchAfter` as the deep paging mechanism. RFC 9457 ([Problem Details](https://www.rfc-editor.org/rfc/rfc9457.html), July 2023) for error bodies, RFC 8288 ([Web Linking](https://www.rfc-editor.org/rfc/rfc8288.html), October 2017) for next-page relations, RFC 9745 ([Deprecation](https://www.rfc-editor.org/rfc/rfc9745.html), March 2025) and RFC 8594 ([Sunset](https://www.rfc-editor.org/rfc/rfc8594.html), May 2019) for retirement signalling. All accessed August 2026. For the retrieval fundamentals underneath the index, *Introduction to Information Retrieval* by Manning, Raghavan and Schütze (Cambridge University Press, 2008) remains the standard free text.

### Level delta (search)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Query surface | Accepts a query string and passes it to the engine | Defines named filters and sorts, and validates strictly | Refuses to expose engine internals and shows the 40-client cost of doing so once |
| Pagination | Offers page and size | Uses a cursor with a tiebreaker and caps depth | Names three guarantees, prices each, and makes the client choose |
| Counts | Returns a total | Caps the total and explains the counting cost | Ships a value-plus-relation shape so the computation can change without a client change |
| Latency | Reports an average | Reports p99 and splits it by query class | Derives the per-shard budget from the fan-out and adds hedging with a stated cost |
| Facets | Adds a facets parameter | Limits the fields and the top-N | Computes cost before execution and rejects at admission with a code naming the parameter |
| Evolution | Plans to version the endpoint | Adds additive-only rules and deprecation signalling | Instruments per-operator usage so deprecation is a conversation with two named teams, not a broadcast |

What each level added, in one line each:

- **Mid to senior:** the contract stops leaking the engine, and every limit exists because a number forced it.
- **Senior to staff:** guarantees are named and chosen rather than implied, and the surface is designed around the fact that removal is a hundred times more expensive than addition.

---

## Case study 5: Publish and subscribe service

Every other case study on this page has a client asking a question and getting an answer. This one has consumers who are absent, slow, duplicated and occasionally reading from a week ago. The contract is therefore mostly about position: who remembers where a consumer got to, and what happens when that memory is wrong.

### The prompt (pub/sub)

"Design the interface a team uses to publish messages to a topic and have other teams' services consume them reliably."

### Questions that fork the design (pub/sub)

Ask these because each answer moves a box or changes a resource. Do not recite them.

| Question worth asking | If the answer is A the design becomes | If the answer is B it becomes |
|---|---|---|
| Do consumers pull, or do you push to them? | Pull: consumers control their own rate, a slow consumer only hurts itself, and you need no consumer availability at all | Push: you inherit every consumer's uptime and latency, and flow control becomes your problem rather than theirs |
| Is a message retained after everyone has read it? | No, delete on acknowledgement: storage is bounded by backlog, and a new consumer starts from now | Yes, retain for a window: a consumer can replay, storage is bounded by time, and reprocessing becomes a normal operation |
| Is ordering required, and across what? | None: any consumer instance takes any message, and scaling is a dial | Per key: messages with the same key must be serialised, which couples ordering to partitioning forever |
| Can a subscription filter, or does it take everything? | Everything: the consumer filters, and you move bytes nobody wants | Server-side filter: you evaluate a predicate per message per subscription, which is the cost the estimation below makes real |
| Is duplicate delivery acceptable? | Yes, at least once: the consumer deduplicates, and the design is simple and honest | No: you need producer sequence numbers, transactional writes and a much narrower set of things you can promise |
| Does a consumer ever need to reprocess history? | No: the position is always "now" | Yes: seek by time and by position become part of the contract, and retention becomes a product decision |
| How many subscriptions attach to the busiest topic? | A handful: fan-out is cheap and can happen at publish time | Hundreds: fan-out at publish time multiplies your write volume by the subscriber count |
| Who owns the schema of a message? | The publisher, informally: fast to start, and every consumer breaks on the first change | A registry with compatibility rules: publishing is slower and consumers survive change |

### Requirements (pub/sub)

Functional:

- Publish a message to a topic, optionally with an ordering key.
- Create a subscription to a topic, with an optional server-side filter.
- Consume messages from a subscription, acknowledge them, and resume from the last acknowledged position after a restart.
- Seek a subscription to a timestamp or an earlier position, to reprocess.
- Route messages that fail repeatedly to a dead-letter destination.
- Register a schema for a topic and reject publishes that do not conform.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Publish rate, aggregate | 50,000 messages per second | ASSUMED, an internal event backbone for a mid-size company; ask, because it decides partition counts and nothing else |
| Average message size | 1 KB | ASSUMED, an event envelope with identifiers and a small payload, not a document |
| Topics | 10,000 | ASSUMED, roughly one per service per event type at a few hundred services |
| Subscriptions per topic, average | 5 | ASSUMED, most events have a handful of interested consumers |
| Subscriptions on the busiest topic | 500 | ASSUMED, one central event that most of the company reacts to; this is the number that breaks naive designs |
| Retention | 7 days | ASSUMED, long enough to survive a weekend incident and reprocess after a Monday fix |
| Publish latency, p99 | under 50 ms | ASSUMED, publishing sits inside a request path in many callers, so it must be cheap |
| End-to-end delivery, p99 | under 2 seconds for a healthy consumer | ASSUMED, and explicitly conditional on the consumer keeping up |
| Messages per day | 4.32 billion | DERIVED below |
| Stored bytes at full retention | 30.2 TB | DERIVED below |
| Deliveries per second, aggregate | 250,000 | DERIVED below |

### Estimation that eliminates (pub/sub)

Every line ends with the option it kills. A number that kills nothing is deleted from the answer.

| Calculation | Result | What it rules out |
|---|---|---|
| 50,000 per second times 86,400 seconds | 4.32 billion messages per day, 4.32 TB per day at 1 KB | Rules out one row per message in a relational table, and rules out an index on anything other than position |
| 4.32 TB per day times 7 days | 30.2 TB retained | Rules out holding the retained log in memory, and rules out per-subscription copies of the retained data, which would multiply this by the subscriber count |
| 50,000 messages per second times 5 subscriptions | 250,000 deliveries per second aggregate | Rules out a delivery path that does per-delivery database work, and rules out writing one durable copy per subscription at publish time |
| The busiest topic at 1,000 messages per second times 500 subscriptions | 500,000 deliveries per second from one topic alone, twice the aggregate figure | Rules out fan-out at publish time entirely. One stored copy read by many cursors is the only shape that survives this number |
| 500 subscriptions times 1,000 messages per second times 2 microseconds per filter evaluation | 1.0 second of CPU per second, so one full core doing nothing but filter evaluation for one topic | Rules out unbounded filter expressions, and rules out evaluating filters on the publish path where they would add to the 50 ms budget |
| 50,000 messages per second divided by 10,000 messages per second per partition | 5 partitions minimum; 32 chosen for headroom and rebalancing granularity | Rules out a single ordered log, and fixes the maximum parallelism of any ordered consumer at 32 |
| One consumer down for 1 hour on a 1,000 message per second topic | 3.6 million message backlog, 3.6 GB | Rules out an in-memory delivery buffer per subscription, and rules out a design where a consumer outage costs the publisher anything |
| 1,000 consumer processes polling every 100 ms with no traffic | 10,000 polls per second, almost all empty | Rules out naive short polling. Long polling or a streaming pull is required before consumer count grows |

!!! tip "Say this out loud in the room"

    "Five hundred subscriptions on one topic at a thousand messages a second is half a million deliveries per second from a single topic. That kills fan-out at publish time before I have drawn anything, so the whole design is one stored log and many independent cursors over it."

### V0, the simplest thing that works (pub/sub)

**Predict first.** V0 stores messages in one table and has consumers poll with a cursor. Which limit is hit first: write throughput, storage, or something else entirely?

??? note "Show answer"

    Something else: poll amplification. Write throughput and storage are both survivable for a while. What breaks first is that every consumer polls every topic it cares about on a timer, so idle topics generate constant load. At 1,000 consumers polling every 100 ms, that is 10,000 queries per second returning nothing, before a single message is published.

```
+-----------+     +--------------------+     +-----------------+
| Publisher | --> | Pub/sub API        | --> | Postgres        |
|           | <-- | append, read after | <-- | messages,       |
+-----------+     +--------------------+     | cursors         |
                          |                  +-----------------+
                          |
                  +--------------------+
                  | Consumer           |
                  | polls, commits     |
                  +--------------------+
```

**Caption.** V0 has four components. A publisher appends a message row to Postgres through one service. A consumer polls the same service for rows after its stored cursor position and commits a new cursor when it has processed them.

Why this is correct at a smaller scale than the one stated:

- One durable copy read by many cursors is already the right shape. Everything V1 adds is about making that shape fast, not about changing it.
- A cursor stored server-side, committed by the consumer, is the whole offset model in four columns, and it gives at-least-once delivery with no additional machinery.
- Retention is a delete query on a timestamp. At a few million rows per day that runs in seconds and needs no compaction design.

!!! warning "When not to use even this much"

    Below roughly 100 messages per second with fewer than 10 consumers, do not build a message service at all. A table with a status column and a worker that claims rows is simpler, has one failure mode, and needs no separate operational story. The measurement that justifies a real broker is either a second team wanting the same events, or a poll rate where more than 90 percent of polls return nothing.

### Breaking point, then V1 (pub/sub)

**Breaking point one: the relational log stops keeping up at roughly 5,000 messages per second.**

What fails: two things compound. Every publish is a row insert with an index update, and every consumer read is an index range scan on a table that is simultaneously being deleted from by the retention job. At 5,000 inserts per second with 25,000 reads per second across five subscriptions each, the table's index is contended and vacuum falls behind the delete rate. Publish p99 crosses 50 ms and stays there. Adding replicas does not help, because the write side is the bottleneck.

How you observe it: publish latency at p99 plotted against retention-job activity. The signature is that latency spikes correlate with deletion batches rather than with publish rate, which points at storage reclamation and not at throughput.

```
+-----------+     +---------------------+     +----------------+
| Publisher | --> | Publish API         | --> | Log storage    |
|           | <-- | validate, route,    |     | 32 partitions, |
+-----------+     | append              |     | 7 day segments |
                  +---------------------+     +----------------+
                            |                        |
                            |                        |
                  +---------------------+            |
                  | Schema registry     |            |
                  | compatibility rules |            |
                  +---------------------+            |
                                                     |
+-----------+     +---------------------+            |
| Consumer  | <-- | Subscription API    | <----------+
| streaming | --> | filter, lease, ack  |
+-----------+     +---------------------+
                            |
                            v
                  +---------------------+
                  | Offset store        |
                  | per subscription    |
                  +---------------------+
```

**Caption.** V1 has six components. Publishers append to a partitioned, segment-based log after a schema registry validates the message. Consumers stream from a subscription service that applies filters, leases messages and records acknowledged positions in a separate offset store.

V1 changes, each with the observation that justified it:

| Change | Justified by | Cost you accept |
|---|---|---|
| Append-only segmented log instead of a table | Publish p99 breaking under retention deletes | Retention is now segment deletion, which is coarse: you cannot delete one message, which matters when a message must be erased for legal reasons |
| 32 partitions | 50,000 per second against 10,000 per second per partition | Maximum parallelism for an ordered consumer is now permanently 32, because raising partition count changes key-to-partition mapping |
| Streaming pull instead of short polling | 10,000 empty polls per second at 1,000 consumers | You now hold long-lived connections, so consumer count is a connection budget, and rolling a broker disconnects everyone at once |
| Offsets stored per subscription, separate from the log | 250,000 deliveries per second against one stored copy | The offset store is now the thing whose loss reprocesses a week of data, so it needs stronger durability than the log itself |
| Schema registry on the publish path | Consumers breaking on the first publisher change | Publishing gains a dependency, so the registry has to be more available than the publish path it guards |

**Breaking point two: one slow consumer and one poison message, at 3.6 million messages of backlog.**

What fails: a consumer with per-key ordering processes a partition serially. One message that always fails, for example a payload that triggers a bug, blocks its partition forever. Meanwhile a consumer that is merely slow accumulates 3.6 million messages in an hour, and when it restarts it fetches as fast as it can, saturating its own downstream dependencies and producing a second outage. Neither situation produces an error on your side; both look like a healthy broker with a rising lag number.

How you observe it: per-subscription, per-partition lag in messages and in seconds. The signature of a poison message is lag rising linearly on exactly one partition while others are flat. The signature of a slow consumer is lag rising on all partitions at the same rate.

V2 changes:

- Delivery attempts are counted per message, and a message exceeding a threshold moves to a dead-letter destination with the original message, the error, the attempt count and the subscription. Without the context, a dead-letter queue is a place data goes to be forgotten.
- Ordering is enforced per key rather than per partition where the consumer opts in, so one blocked key does not block the partition. This costs an in-flight key table per consumer and is the single most useful thing you can offer an ordered consumer.
- Flow control is explicit: the consumer declares how many messages or bytes it will accept outstanding, and the broker respects it. A restarting consumer is protected from its own backlog.
- Seek by time and by position become supported operations, with the honest caveat that seeking backwards on a subscription with many consumers reprocesses for all of them.

!!! warning "When not to use a dead-letter destination"

    If failures are transient by nature, for example a downstream timeout, a dead-letter destination just moves the retry decision somewhere nobody is watching. Bounded retry with backoff on the original subscription is better. The measurement that justifies dead-lettering is a message that has failed more than about 10 times with the same error, which is a data problem and not a capacity problem, and it will never succeed no matter how long you retry.

### Deep dive one: who owns the cursor

This is the dive that decides the operational character of the whole system, and it is the one candidates skip by saying "the consumer commits offsets" without saying what that means when it goes wrong.

**Three ownership models, with sharply different failure behaviour.**

| Model | Where position lives | What a crash costs | What it makes hard |
|---|---|---|---|
| Broker tracks per message | Broker, one record per in-flight message | Nothing beyond redelivery of in-flight messages | Throughput: the broker does per-message bookkeeping, which is 250,000 writes per second here |
| Broker tracks a committed position | Broker, one small record per subscription per partition | Redelivery back to the last commit, which may be thousands of messages | Fine-grained retry: you cannot skip one bad message without skipping everything before it |
| Consumer stores position itself | Consumer's own database | Nothing, if committed in the same transaction as the effect | Operations: you cannot see a consumer's lag, so your dashboards go blind |

**The trade-off nobody states out loud: acknowledgement granularity versus throughput.** Per-message acknowledgement lets you skip one poison message and gives exact lag, and it costs a durable write per message. A committed position is one small write per batch, and it forces redelivery of everything after the last commit. Most systems pick the position model and then bolt per-message tracking on for a bounded in-flight window, which is the honest hybrid.

**The commit ordering bug, which is the most common real defect.**

| Order | What happens on a crash between the two steps | Resulting guarantee |
|---|---|---|
| Commit position, then process | The message is never processed and nobody knows | At most once, usually by accident rather than by choice |
| Process, then commit position | The message is processed again after restart | At least once, which is what you should design for |
| Process and commit in one transaction | Neither or both | Effectively once, and only possible when the effect and the offset share a store |

**Effectively once is a property of the consumer, not of the broker.** No broker can give exactly-once end to end, because the effect happens in the consumer's world. What a broker can do is give the consumer a stable message identifier and a monotonic position, so the consumer can deduplicate. Saying this clearly, and refusing to promise exactly-once delivery, is a senior signal. Promising it is a staff-level red flag.

**Lag is two numbers and they answer different questions.**

| Metric | What it tells you | When it misleads |
|---|---|---|
| Lag in messages | How much work remains | Useless for capacity planning when message cost varies wildly |
| Lag in seconds, from the oldest unacknowledged message | How stale the consumer's view is | Reads as zero on an idle topic even if the consumer is dead |

Publish both, and alert on the time-based one with a liveness check underneath it, because the message-based one goes quiet exactly when a topic stops receiving traffic.

**Seeking backwards is a destructive operation with a friendly name.** Moving a subscription to an earlier position reprocesses everything after it, for every consumer on that subscription, with all its side effects. It needs a confirmation, an audit record and, ideally, the ability to seek a copy of the subscription rather than the original. Treating seek as a routine read operation is how a replay becomes an incident.

!!! warning "When not to let the broker own position"

    If the consumer's effect is a write to its own database, storing the position in that same database, in the same transaction, is strictly better: it makes the effect and the position atomic, which no broker-side commit can. The cost is that your lag dashboards go dark, so publish a heartbeat carrying the consumer's position back. The measurement that justifies moving position into the consumer is any duplicate-effect incident that survived deduplication.

### Deep dive two: fan-out arithmetic and where a filter runs

The dives on this page vary by problem. Delivery guarantees to an external endpoint were the second dive for the payments case study, so here the depth goes to the number that actually shapes a broker: what a topic costs when subscriber count is the variable.

**Three fan-out placements, with their cost in this design's numbers.**

| Placement | Copies stored per message | Filter evaluations per second on the busy topic | Fails when |
|---|---|---|---|
| Copy per subscription at publish time | 500 on the busy topic, so 500 KB stored per 1 KB published | 1,000, evaluated once per message at publish | Immediately. 500,000 durable writes per second for one topic |
| One stored copy, filter at delivery time | 1 | 500,000, one per message per subscription | At high subscriber counts, which is exactly the one core of CPU derived above |
| One stored copy, index the filter | 1 | Far fewer, because non-matching subscriptions are never considered | Filters are expressive enough that they cannot be indexed |

**The design I would defend: one stored copy, filters evaluated at delivery, with a restricted filter grammar chosen so that filters can be indexed.** Restricting the grammar is what makes the third row available, and it is a contract decision made on day one, because loosening a filter grammar later is easy and tightening it is breaking.

**A filter grammar that can be indexed looks like this.**

| Allowed | Why it stays cheap |
|---|---|
| Equality on a declared attribute | Groups subscriptions by attribute value, so one lookup finds every matching subscription |
| Set membership on a declared attribute | Same lookup, repeated per value |
| Presence of an attribute | A single boolean index |
| Conjunction of the above | Intersect the candidate sets |

| Forbidden, and the reason |
|---|
| Arbitrary substring or regular expression matching, because it forces evaluation against every subscription |
| Predicates over the message body rather than declared attributes, because the body may be compressed or opaque |
| Disjunction across different attributes, because it defeats the grouping that makes the index useful |

**Attributes must be separate from the payload.** If a filter has to parse the message body, you have coupled routing to schema, and a schema change becomes a routing change. Put filterable attributes in an envelope alongside the payload, cap their number and size, and treat the payload as opaque bytes to the broker. This single decision is what lets you compress or encrypt payloads later without touching routing.

**Fan-out changes what a publish latency budget means.** With one stored copy, the publisher's 50 ms budget covers an append and nothing else, so subscriber count does not affect publish latency at all. That property is worth protecting explicitly, because the first request you will receive is "can you evaluate my filter at publish time so I know it matched", and granting it couples every publisher to every subscriber.

**The reverse question is the one to ask back.** When a team wants 500 subscriptions on one topic, the usual real requirement is 500 different downstream actions, not 500 independent consumers. Often the right answer is one consumer that fans out internally, which turns a broker problem into that team's problem where it can be solved with a switch statement. Saying no to a subscription is a legitimate design move.

!!! warning "When not to offer server-side filters"

    If the average subscription matches more than about half the topic's messages, filtering server-side saves little bandwidth and costs you the evaluation and the grammar. Let the consumer filter. The measurement that justifies server-side filters is a subscription whose match rate is below roughly 10 percent, where you are otherwise moving nine times more bytes than anyone reads.

### Failure modes and evaluation (pub/sub)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Head-of-line blocking | One poison message stalls its partition for every consumer on the subscription | Per-key ordering instead of per-partition, plus a dead-letter destination after a bounded attempt count |
| Thundering herd | A consumer restarts after an hour and fetches 3.6 million messages as fast as it can | Explicit flow control on outstanding messages and bytes, declared by the consumer and enforced by the broker |
| Hot partition | An ordering key with skewed cardinality sends most traffic to one partition | Publish per-partition throughput, and treat a key with more than roughly 5 percent of volume as a design bug in the publisher |
| Metastable failure | A slow consumer causes redelivery, redelivery increases load, which makes it slower | Lease extension while processing rather than a fixed visibility timeout, and cap redelivery as a fraction of throughput |
| Split brain | Two consumer instances both believe they own a partition after a network partition | Fenced leases with a monotonic epoch, so the older owner's commits are rejected |
| Duplicate effect | At-least-once delivery with a non-idempotent consumer | Stable message identifiers, and a documented expectation that consumers deduplicate; never promise exactly once |
| Retention loss | A consumer down for 8 days finds its position deleted | Alert on any subscription whose oldest unacknowledged position is within 24 hours of the retention edge |
| Schema break | A publisher adds a required field and every consumer fails to parse | Registry compatibility check at publish time, rejecting the publish rather than the consumer |

Service level indicators to publish:

- Publish latency at p99, measured separately from schema validation, so a slow registry is visibly distinct from a slow log.
- Per-subscription lag in seconds, from the oldest unacknowledged message, with a liveness heartbeat so an idle topic does not report a healthy zero.
- Per-partition lag, which is what distinguishes a poison message from a slow consumer.
- Redelivery rate as a fraction of deliveries, split by subscription. A rising rate means processing is slower than the lease, not that the broker is unhealthy.
- Dead-letter arrival rate, per subscription, alerted on any non-zero sustained value because it means data is being set aside.
- Distance between the oldest unacknowledged position and the retention edge, in hours.

Alert on distance to the retention edge, because it is the only failure here that destroys data permanently and it always arrives with days of warning. Alert on per-partition lag divergence rather than on total lag, since divergence is the specific signal for a stuck key. Do not page on total lag: it rises for legitimate reasons every deployment.

Re-checking against the non-functional requirements:

| Requirement | Does V2 meet it? |
|---|---|
| 50,000 publishes per second | Yes, at 32 partitions and roughly 1,560 per second per partition, well under the 10,000 per second per partition assumption |
| Publish p99 under 50 ms | Yes, and it stays independent of subscriber count precisely because fan-out was kept off the publish path |
| 250,000 deliveries per second | Yes, from one stored copy read by many cursors. The publish-time fan-out design fails this by a factor of the subscriber count |
| 500 subscriptions on one topic | Yes, with an indexable filter grammar. With unrestricted filters this needs a full core per topic and does not hold as subscriptions grow |
| 7 day retention, 30.2 TB | Yes, as time-based segment deletion. Note that this makes deleting one specific message impossible, which the simplifications section revisits |
| End-to-end p99 under 2 seconds | Yes for a consumer that is keeping up, and explicitly not promised otherwise. The contract says so rather than implying it |

### Where this answer is a simplification (pub/sub)

- **Segment-based retention and the right to erasure are in direct conflict.** You cannot delete one message from an immutable segment. The usual answer is to store payloads encrypted with a per-subject key and delete the key, which turns erasure into key management and changes the whole storage design.
- **Schema evolution deserves its own module, and has one.** Compatibility direction (can new consumers read old messages, or the reverse) determines which changes are allowed, and it differs per topic. See the versioning and evolution material earlier on this page.
- **Multi-region replication changes the ordering promise.** Ordered within a region and eventually consistent across regions is achievable. Global ordering is not, at any reasonable cost, and a consumer that assumes it will be wrong during exactly one incident per year.
- **Transactional publishing is narrower than it sounds.** Publishing atomically with a database write generally means an outbox in that database, not a distributed transaction. That is a pattern in the publisher, not a broker feature, and the payments case study earlier on this page shows the mechanics.
- **Primary sources with dates.** RFC 9457, [Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html) (July 2023, accessed August 2026), gives the error shape for a rejected publish, which matters because a schema rejection needs to name the offending field. RFC 9110, [HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) (June 2022, accessed August 2026), covers the status semantics for the control-plane operations. For the log storage, partitioning and consensus building blocks underneath the broker, see [Modern System Design](modern-system-design.md).

### Level delta (pub/sub)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Fan-out | Says the broker delivers to all subscribers | Derives 500,000 deliveries per second from one topic and stores one copy | Restricts the filter grammar so subscriptions can be indexed, and treats that restriction as a day-one contract decision |
| Position | Says consumers commit offsets | Names the commit ordering bug and picks process-then-commit | Puts position in the consumer's own transaction where the effect allows it, and adds a heartbeat so lag stays observable |
| Guarantees | Says exactly once | Says at least once and requires consumer deduplication | Explains that end-to-end exactness is a consumer property and refuses to promise it in the contract |
| Ordering | Says messages are ordered | Scopes ordering to a partition and links it to key hashing | Offers per-key ordering so one stuck key does not block a partition, and names the in-flight key table as the cost |
| Backlog | Adds retry | Adds a dead-letter destination with attempt counts | Adds consumer-declared flow control, because a recovering consumer is the thing that causes the second outage |
| Operations | Watches lag | Publishes lag in messages and in seconds | Alerts on distance to the retention edge, the only failure here that loses data irreversibly |

What each level added, in one line each:

- **Mid to senior:** guarantees are stated precisely and narrowly, and the delivery cost model is derived rather than assumed.
- **Senior to staff:** the design protects itself from its own consumers, through grammar restrictions, flow control and an alert aimed at irreversible loss.

---
## Case study 6: Calendar and scheduling interface

This is the only prompt on the page where the first breaking point is a correctness bug rather than a capacity limit, and where the thing that breaks your design is a government changing a law. Calendars are where candidates discover that "store everything in UTC" is advice for logs, not for appointments.

### The prompt (calendar)

"Design the interface a client uses to create events, including repeating ones, invite people, and read a month of someone's calendar."

### Questions that fork the design (calendar)

Ask these because each answer moves a box or changes a resource. Do not recite them.

| Question worth asking | If the answer is A the design becomes | If the answer is B it becomes |
|---|---|---|
| Do events repeat? | No: an event is a row with a start and an end, and the whole design is a range query | Yes: an event is a rule plus exceptions plus overrides, and the read path is an expansion, not a lookup |
| Can a caller edit one occurrence of a series? | No: a change applies to the whole series and identity stays simple | Yes: an occurrence needs its own identity, and "this and following" becomes a series split, which is the operation everyone gets wrong |
| Are attendees on the same system, or external? | Internal: invitations are rows, status is a field, and free/busy is a query you control | External: you are implementing an interoperability protocol, and the other side's semantics are now yours |
| Does the product need free/busy across many people? | No: reads are per calendar and partitioning by owner is free | Yes: a scheduling query fans out across every attendee's calendar, and the fan-out width becomes a latency budget |
| Are all-day events a thing? | No: everything is an instant and time zones only affect display | Yes: you now have dates without times, which are a different type and cannot be stored as instants at all |
| Is the event's time fixed to a wall clock or to an instant? | Instant, for example a launch at a precise moment: UTC is correct and time zones are presentation | Wall clock, for example a 9am standup: UTC is wrong, and the zone identifier must be stored with the event |
| Can a series repeat forever? | No, bounded by count or end date: expansion always terminates | Yes: every read must impose its own window, and no query may ask for "all occurrences" |
| Do clients cache the calendar offline? | No: every view is a read | Yes: you need a sync token, tombstones for deletions, and a defined behaviour when the token is too old |

### Requirements (calendar)

Functional:

- Create, read, update and delete an event, with a repeat rule.
- Edit a single occurrence, all occurrences, or this occurrence and all following ones.
- Invite attendees, record their responses, and let an organiser see them.
- Read every occurrence overlapping a time window for one calendar.
- Query combined free and busy periods across a set of calendars.
- Give a client a sync token so it can fetch only what changed since its last read.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Active users | 20 million | ASSUMED, a mid-size workplace suite; ask, because it sets read volume and nothing else |
| Events created per user per week | 5 | ASSUMED, a working calendar with meetings and reminders, not a personal one |
| Calendar view reads | 10,000 per second | ASSUMED, driven by clients refreshing on focus rather than by user actions |
| Fraction of events that repeat | 30 percent | ASSUMED, standups and one-to-ones repeat, most meetings do not |
| Default view window | 5 weeks | ASSUMED, a month grid includes trailing days from adjacent months |
| Maximum attendees per event | 500 | ASSUMED, and published, because an unbounded attendee list makes invitation fan-out unbounded |
| Read latency, p99 | under 200 ms | ASSUMED, a calendar grid is the first paint of the application |
| Scheduling query fan-out | 50 calendars | ASSUMED, the practical width of a "find a time" query |
| Events created per second | 165 | DERIVED below |
| Occurrences if fully materialised | 333 billion rows | DERIVED below |
| Expansions per second on the read path | 1.5 million | DERIVED below |

### Estimation that eliminates (calendar)

Every line ends with the option it kills. A number that kills nothing is deleted from the answer.

| Calculation | Result | What it rules out |
|---|---|---|
| 20 million users times 5 events per week divided by 604,800 seconds | 165 events created per second, 14.3 million per day | Rules out any discussion of write scaling. One primary handles this, so the design budget belongs entirely on the read path |
| 30 percent of 14.3 million per day, retained 2 years | 4.3 million new series per day, roughly 3.1 billion active series | Rules out an index scan across all series for any query, and forces partitioning by calendar owner |
| 3.1 billion series times 104 weekly occurrences over 2 years | 333 billion occurrence rows if every series were materialised | Rules out materialising occurrences as rows. This is the single number that decides the data model |
| 30 active series per user, expanded over a 5 week window, 10,000 reads per second | Roughly 150 occurrences per read, 1.5 million expansions per second | Rules out the objection that expanding on read is too slow. At roughly 2 microseconds per occurrence that is about 3 cores, which is nothing |
| 50 attendees per scheduling query times 150 occurrences each | 7,500 occurrences computed per scheduling query, across 50 partitions | Rules out a synchronous scatter over 50 partitions inside a 200 ms budget without a per-calendar summary to read instead |
| 500 attendees times an invitation write and a notification | 1,000 side effects from one event creation | Rules out doing attendee fan-out inside the create request, and rules out a synchronous notification per attendee |
| 1 percent of 3.1 billion series referencing a zone whose rules changed | 31 million series whose future occurrences move on one time zone database release | Rules out caching expanded occurrences without a cache key that includes the time zone database version |
| 2 daylight saving transitions per year per affected zone | Two dates a year on which a wall-clock series stored as instants is wrong by an hour | Rules out storing recurring wall-clock events as UTC instants, which is the most common wrong answer to this prompt |

!!! tip "Say this out loud in the room"

    "A hundred sixty-five writes per second means the write path is uninteresting. The two numbers I care about are 333 billion rows if I materialise occurrences, which I will not, and 31 million series that move when a government changes a time zone rule, which is why the zone identifier has to be stored on the event."

### V0, the simplest thing that works (calendar)

**Predict first.** V0 stores each event as a start instant, an end instant and a repeat rule, all in UTC. Name the day of the year on which a weekly 9am meeting created in March becomes wrong.

??? note "Show answer"

    The next daylight saving transition in the organiser's zone. Stored as a UTC instant, "9am local on Mondays" was frozen as, for example, 13:00 UTC. After the clocks move, 13:00 UTC is 8am or 10am local, so the meeting drifts by an hour for every occurrence after the transition. Nothing errors. The calendar is simply wrong, twice a year, for a subset of users.

```
+-----------+     +---------------------+     +-----------------+
| Client    | --> | Calendar API        | --> | Postgres        |
|           | <-- | create, expand      | <-- | events (start,  |
+-----------+     | series on read      |     | end, rrule),    |
                  +---------------------+     | attendees       |
                                              +-----------------+
```

**Caption.** V0 has three components. A client calls one calendar service, which stores events with a start instant, an end instant and a repeat rule in Postgres, and expands repeating series in memory when a view window is requested.

Why this is correct at the stated scale, apart from the bug above:

- Expanding on read is the right call and the estimation proves it: 1.5 million occurrences per second is about 3 cores of work. Materialising them would be 333 billion rows.
- 165 writes per second needs no queue, no cache and no sharding. A single partitioned table with an index on owner and time range serves every read.
- Attendees as rows on the event, with a status column, is the entire invitation model when everyone is on your system.

!!! warning "When not to use even this much"

    If no event repeats, delete the rule column and the expansion code. A calendar of one-off events is a range query over a table, and every deep dive below is over-engineering. The measurement that justifies recurrence machinery is the first user creating the same event on consecutive weeks, which happens immediately in a workplace product and may never happen in a booking product.

### Breaking point, then V1 (calendar)

**Breaking point one: the wall-clock bug, hitting on two specific dates per year.**

What fails: a recurring event whose meaning is "9am local" stored as a UTC instant shifts by one hour for every occurrence after a daylight saving transition. It is not a performance failure and no error is logged. If 30 percent of events repeat and roughly half of your users live in a zone that observes daylight saving, a large fraction of recurring meetings move on two days a year, and every one of them becomes a support ticket on the same morning.

How you observe it: you do not, from telemetry. You observe it from a spike in event edits within 48 hours of a transition date, and from tickets. The right instrument is a scheduled correctness check that expands a sample of recurring series across the transition and asserts the local time is unchanged. Building that check is the fix's proof.

```
+-----------+     +---------------------+     +------------------+
| Client    | --> | Calendar API        | --> | Postgres         |
|           | <-- | expand in zone,     | <-- | series (local    |
+-----------+     | apply overrides     |     | start, zone id,  |
                  +---------------------+     | rule)            |
                       |          |           | overrides        |
                       |          |           | attendees        |
                       v          v           +------------------+
            +-------------+  +--------------+
            | Zone rules  |  | Invite       |
            | versioned   |  | fan-out      |
            +-------------+  | worker       |
                             +--------------+
```

**Caption.** V1 has six components. The calendar service stores a series as a local start time plus a time zone identifier plus a rule, expands occurrences using a versioned copy of the zone rules, applies per-occurrence overrides from a separate table, and hands attendee invitations to a background worker.

V1 changes, each with the observation that justified it:

| Change | Justified by | Cost you accept |
|---|---|---|
| Store local start plus zone identifier, not an instant | Two transitions a year silently moving recurring events | Every read now needs zone rules loaded, and "what time is this really" becomes a computation rather than a column |
| A separate overrides table keyed by occurrence identity | Editing one occurrence of a series had nowhere to be stored | Expansion is now rule minus exceptions plus overrides, which is three sources that must agree |
| Zone rules pinned to a named version | 31 million series move on a rules release | You must now decide, deliberately, when to adopt a new zone rules release, and that decision changes people's calendars |
| Invitation fan-out moved to a worker | 1,000 side effects from one 500 attendee event | Event creation returns before attendees are notified, so the organiser can see an event with no attendees for a moment |
| Explicit window required on every occurrence read | A rule with no end has infinite occurrences | Clients that wanted "all occurrences" now get an error, which is the correct answer to an impossible request |

**Breaking point two: the scheduling query fans out over 50 calendars, at 100 queries per second.**

What fails: a "find a time" query reads 50 calendars, each partitioned by a different owner, expands roughly 150 occurrences each, and merges them. That is 50 partition reads inside a 200 ms budget, so the query's latency is the latency of the slowest of 50 reads. At a per-read p99 of 20 ms, the probability that all 50 are inside it is 0.99 to the power of 50, or 60.5 percent, so nearly 40 percent of scheduling queries wait on a slow partition.

How you observe it: scheduling query latency plotted against attendee count. The signature is a curve that bends upward with the number of attendees while single-calendar reads stay flat, which localises the problem to fan-out rather than to expansion.

V2 changes:

- Each calendar maintains a compact free/busy summary for a rolling window, updated when events change. A scheduling query reads 50 small summaries instead of expanding 50 calendars, which turns the read into a bitmap intersection.
- The summary's cache key includes the zone rules version, so a rules release invalidates exactly the summaries that could have moved rather than all of them.
- A sync token per calendar lets clients fetch only changes, including tombstones for deleted occurrences. Without tombstones, a client that cached a cancelled meeting shows it forever.
- Attendee lists above roughly 100 are treated as a different object: the event carries a group reference rather than individual rows, and per-attendee status becomes a summary. Otherwise a 500 attendee event produces 500 rows that every read of the event must load.

!!! warning "When not to precompute free/busy"

    Below roughly 10 attendees per scheduling query, expand on demand. Ten partition reads inside 200 ms is comfortable, and a summary introduces a second copy of the truth that can be stale, which produces the worst possible calendar bug: a time that shows as free and is not. The measurement that justifies a summary is scheduling latency growing with attendee count past your budget, not the existence of the feature.

### Deep dive one: storing a repetition you can still expand in five years

This dive exists because the representation choice is invisible in the interface and decides whether three ordinary product features are possible at all. Candidates who store a list of dates cannot implement any of them.

**Three ways to represent a repeating event.**

| Representation | Storage for a 2 year weekly series | What it makes impossible |
|---|---|---|
| Materialised occurrence rows | 104 rows | Nothing functionally, but 333 billion rows in aggregate, and an unbounded series cannot be stored at all |
| A rule, expanded on read | 1 row | Nothing, provided every read supplies a window. Expansion cost was derived above as about 3 cores |
| A rule plus a materialised near window | 1 row plus roughly 12 rows | Nothing, and it gives fast reads for the common window at the cost of keeping the two in step |

**The design I would defend: a rule expanded on read, with overrides stored separately.** Expansion is cheap, storage is one row, and an unbounded series is representable. RFC 5545, [iCalendar](https://www.rfc-editor.org/rfc/rfc5545.html) (September 2009, accessed August 2026), already defines this shape: a recurrence rule, exception dates that remove occurrences, and additional dates that add them.

**An occurrence needs identity before you can edit one.** The identity is the series identifier plus the occurrence's original start time, which is what iCalendar calls a recurrence identifier. The subtlety: it must be the original start, not the current one, because moving an occurrence must not change its identity or you cannot find it again to move it back.

**The three edit scopes, and what each actually does.**

| Scope the user picks | What the system does | The bug to avoid |
|---|---|---|
| This occurrence | Write an override row keyed by original start | Keying the override by the new start, which makes the override unfindable after a second edit |
| All occurrences | Update the rule on the series | Silently discarding existing overrides, which resurrects meetings the user already moved |
| This and following | Split: end the old series the day before, create a new series from this occurrence | Editing the rule in place, which retroactively changes past occurrences that already happened |

**"This and following" is a split, and the split is where the data model earns its keep.** The old series gets an end date. A new series is created with the new rule. Overrides after the split point move to the new series. Attendee responses have to be decided deliberately: carrying them over is usually right for a time change and usually wrong for a change of content.

**Bound every expansion.** Two rules make this safe: no read may omit a window, and no window may exceed a published maximum, for example two years. A rule that repeats every second for ten years is a valid rule and an unbounded expansion, so also cap the occurrence count per response and return a continuation. This is the difference between a calendar and a denial of service endpoint.

**Cancellation is not deletion.** A cancelled occurrence must remain visible as cancelled to clients that already saw it, or their cached copy shows a meeting that no longer exists. That means an exception date is not enough on its own: the sync feed needs a tombstone carrying the cancelled occurrence's identity.

!!! warning "When not to expand on read"

    If a calendar is read far more often than it changes and the window is always the same, for example a public opening-hours display, materialise. Expansion on every read repeats identical work. The measurement that justifies materialising is expansion CPU becoming a visible fraction of service cost, which at about 3 cores here it is not, or a query pattern that needs occurrences to be indexable, for example "find every meeting longer than an hour next month" across all users.

### Deep dive two: a time zone is a claim about the future, not an offset

The dives on this page vary by problem. The recurrence dive covered representation; this one covers the input that representation depends on, and it is the part of calendars that has no analogue anywhere else on this page.

**There are three kinds of time in a calendar, and they are different types.**

| Kind | Example | Stored as | What breaks if you store it wrong |
|---|---|---|---|
| Instant | A rocket launch, a market open | UTC timestamp | Nothing. This is the case UTC advice was written for |
| Local time in a named zone | A 9am standup in Berlin | Local wall time plus a zone identifier | Recurring occurrences shift by an hour at every transition |
| Floating local time | "Take medication at 08:00", wherever you are | Local wall time, no zone | Stored with a zone, it fires at the wrong hour after the user flies |

**An offset is not a zone.** Plus two hours is a fact about one moment; Europe/Berlin is a rule set that says what the offset will be at every future moment, and that rule set changes. Storing an offset with a future event freezes a prediction that was true when it was made. Storing the identifier keeps the event correct when the prediction changes.

**Zone rules change, and the change is data, not code.** RFC 6557, [Procedures for Maintaining the Time Zone Database](https://www.rfc-editor.org/rfc/rfc6557.html) (February 2012, accessed August 2026), describes how the shared database is maintained under IANA stewardship. Releases happen when governments change rules, sometimes with only weeks of notice. Three consequences you should state:

- Every cached expansion must carry the zone rules version it was computed with, or a release silently leaves wrong data in your cache.
- Server and client may hold different rules versions, so the same series expands differently on each. The server's expansion is authoritative and the response should carry the version used.
- Adopting a new release changes users' calendars. That is a deployment with user-visible consequences, and it deserves the same care as a schema migration.

**Local times that do not exist, and local times that happen twice.** At a spring transition, a wall clock jumps and an hour is missing. At an autumn transition, an hour repeats. Both are real inputs a user can submit.

| Situation | The input | The policy to publish |
|---|---|---|
| Gap | 02:30 on a day the clock jumps from 02:00 to 03:00 | Shift forward by the size of the gap, and say so, so all clients agree |
| Overlap | 02:30 on a day 02:00 to 03:00 happens twice | Take the first occurrence, which is the earlier offset, and say so |

Neither policy is more correct than the other. What matters is that it is written down, because otherwise every client library picks its own and two devices show different times for the same event.

**All-day events are dates, not instants.** A holiday on 1 January is the whole of 1 January wherever the viewer is; converting it to an instant makes it start on 31 December for someone. Store a date, and give it its own type in the contract, because the first client to receive a timestamp for an all-day event will render it in local time and file a bug.

**What the interface should accept and return.** Accept a local time plus a zone identifier for scheduled things, and a plain date for all-day things. Return the same, plus a computed instant as a convenience field marked explicitly as derived. Returning only the instant loses the information needed to expand the series correctly, and clients cannot reconstruct it.

!!! warning "When not to store a zone identifier"

    If every event is genuinely an instant, for example a log of things that happened or a market data feed, a UTC timestamp is correct and a zone identifier is noise that invites incorrect conversion. The test is a question: if the government changed the clock rules tomorrow, should this event's wall-clock time change or its instant stay fixed? Instants keep the instant. Everything a human agreed to attend keeps the wall clock.

### Failure modes and evaluation (calendar)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Silent data corruption | Recurring events shift an hour after a transition because they were stored as instants | Store local time plus zone identifier, and run a scheduled correctness check across the next transition date |
| Stale derived data | A cached expansion computed under old zone rules is served after a rules release | Zone rules version in the cache key, and a targeted invalidation of series in affected zones |
| Version skew | Client and server hold different zone rules and disagree about a meeting time | Return the rules version used, and make the server's expansion authoritative for anything that triggers a notification |
| Fan-out latency | 40 percent of 50 attendee scheduling queries wait on a slow partition | Precomputed free/busy summaries, read as small objects instead of 50 expansions |
| Thundering herd | Every client refreshes its calendar at the top of the hour before meetings start | Jittered client refresh, plus a sync token so a refresh returns almost nothing |
| Unbounded computation | A rule repeating every second with no end date, expanded over a ten year window | Mandatory window on every read, a published maximum window, and a cap on occurrences per response |
| Write amplification | A 500 attendee event writes 500 invitation rows and 500 notifications inside the request | Background fan-out worker, and a group reference instead of individual rows above roughly 100 attendees |
| Lost update | Two organisers edit the same series from two devices | Entity tag per series plus a conditional write, returning 412 rather than merging |

Service level indicators to publish:

- Calendar view latency at p99, split by whether the view contained recurring series, because expansion cost is only visible in that split.
- Scheduling query latency at p99, bucketed by attendee count, since averaging hides the fan-out curve entirely.
- Expansion correctness: the count of sampled recurring series whose local time changes across the next transition, which should be zero.
- Sync token rejection rate, which rises when clients have been offline longer than your change retention and tells you the retention is too short.
- Invitation fan-out lag at p99, measured from event creation to the last attendee notified.
- Zone rules version currently in effect, published as a value, so an incident can be correlated with a rules adoption.

Alert on non-zero expansion correctness failures, because it is the only indicator here that catches silent wrongness. Alert on sync token rejection rate, since it produces a full resynchronisation from every affected client and can turn into a self-inflicted load spike. Do not alert on invitation fan-out lag under a few minutes; it is asynchronous by design and belongs on a dashboard.

Re-checking against the non-functional requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Read p99 under 200 ms | Yes for single-calendar views at roughly 150 expansions each, and yes for scheduling queries once summaries replace live fan-out |
| 10,000 view reads per second | Yes, roughly 3 cores of expansion plus indexed reads partitioned by owner |
| 165 writes per second | Yes, with an order of magnitude of headroom, which is why none of the design effort went here |
| 50 calendar scheduling fan-out | Yes, via summaries. With live expansion, 40 percent of queries miss the latency budget |
| 500 attendee events | Yes, with background fan-out and a group reference. Inline fan-out puts 1,000 side effects inside one request |
| Unbounded series | Yes, because occurrences are never materialised and every read carries a window |
| Correctness across a transition | Yes, and it is the only requirement here that is verified by a scheduled check rather than by a latency number |

### Where this answer is a simplification (calendar)

- **Interoperating with external calendars is a protocol, not a feature.** Once invitations cross systems you inherit another implementation's interpretation of the same standard, including its handling of gaps, overlaps and partial responses. Most real bugs in production calendars live at that boundary.
- **Rooms and equipment are resources with different semantics.** A room cannot decline, cannot double-book, and needs a booking that is exclusive rather than advisory. Modelling a room as an attendee is convenient and wrong in exactly one case, which is the case that matters.
- **Permissions on a calendar are unusually fine-grained.** See free/busy only, see titles, see full details, edit, manage. Free/busy summaries must respect the first of these, which means the summary is not one object but one per visibility level.
- **Notification timing is its own scheduling problem.** A reminder 10 minutes before an occurrence of an unbounded series has no row to hang off, so reminders are computed from the same expansion and must be recomputed when zone rules change.
- **Primary sources with dates.** RFC 5545, [Internet Calendaring and Scheduling Core Object Specification](https://www.rfc-editor.org/rfc/rfc5545.html) (September 2009, accessed August 2026), defines recurrence rules, exception dates and the recurrence identifier this design uses for occurrence identity. RFC 6557, [Procedures for Maintaining the Time Zone Database](https://www.rfc-editor.org/rfc/rfc6557.html) (February 2012, accessed August 2026), covers the stewardship of the zone rules and is the reason zone data has versions at all. RFC 9110, [HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) (June 2022, accessed August 2026), defines the conditional request behaviour behind the entity tag design.

### Level delta (calendar)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Time storage | Stores everything in UTC | Distinguishes instants from wall-clock times and stores a zone identifier for the latter | Adds floating time as a third type and publishes the gap and overlap policies so clients cannot disagree |
| Recurrence | Stores a rule | Expands on read after deriving that materialising is 333 billion rows | Defines occurrence identity by original start, and treats "this and following" as a series split with an override migration |
| Zone data | Does not mention it | Notes that zone rules change and pins a version | Puts the version in every cache key and treats adopting a release as a user-visible deployment |
| Scheduling | Queries each attendee's calendar | Derives the 50 way fan-out cost and precomputes free/busy | Notes that a summary must exist per visibility level, because free/busy is a permission boundary |
| Bounds | Adds pagination | Requires a window on every occurrence read | Publishes a maximum window and an occurrence cap, naming the unbounded rule as a denial of service vector |
| Verification | Adds monitoring | Publishes latency indicators | Adds a scheduled correctness check across the next transition, because this design's worst failure emits no errors |

What each level added, in one line each:

- **Mid to senior:** time stops being one type, and the data model is chosen against a derived storage number rather than by convention.
- **Senior to staff:** the failure that produces no error signal is identified, given an indicator, and given an owner.

---
## Case study 7: Chat interface

Chat is the prompt where candidates draw a socket and consider the design finished. The contract problem is that a chat product has two transports, a live stream and a history read, and a client has to be able to tell whether it has seen everything. Almost every visible chat bug is a disagreement between those two.

### The prompt (chat)

"Design the interface a client uses to send and receive messages in a conversation, load history, and show what has been read."

### Questions that fork the design (chat)

Ask these because each answer moves a box or changes a resource. Do not recite them.

| Question worth asking | If the answer is A the design becomes | If the answer is B it becomes |
|---|---|---|
| One to one only, or groups? | One to one: fan-out is 2, and a conversation is a row with two participants | Groups up to 1,000: fan-out is the design, and membership changes become events in the message stream |
| Does the client keep local history? | No: every open is a read, and the server is the only copy | Yes: you need a resumable stream, a backfill read and tombstones, and the client can be arbitrarily far behind |
| Is delivery status per recipient? | Delivered and read as a single conversation-level marker | Per recipient: the status object grows with group size and the write rate multiplies by membership |
| Can messages be edited or deleted after sending? | No: a message is immutable, so caching and client storage are trivial | Yes: history reads must return revisions and tombstones, and a client that missed the edit shows the old text forever |
| Is ordering per conversation strict? | Best effort by timestamp: simple, and clients occasionally show messages out of order | Strict: you need a server-assigned per-conversation sequence, which makes the conversation a serialisation point |
| Are messages end-to-end encrypted? | No: the server can index, search, moderate and render previews | Yes: the server carries opaque bytes, so search, moderation and previews move to the client and change every other decision |
| How long is history retained? | Forever: storage grows without bound and old data must be tiered | A window: deletion is routine, and clients need to be told what has aged out rather than seeing a gap |
| Is presence required? | No: enormous saving, since presence is usually the highest-volume traffic in a chat system | Yes: status changes fan out to every contact, and the estimation below shows what that costs |

### Requirements (chat)

Functional:

- Send a message to a conversation and have it appear for every participant.
- Receive messages in near real time while connected.
- Read history in a conversation, backwards from any point, with stable ordering.
- Resume after a disconnection without missing or duplicating messages.
- Mark how far a participant has read, and see how far others have read.
- Edit and delete a message, with both reflected in history and in the live stream.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Concurrent connections | 10 million | ASSUMED, a large consumer messaging product; ask, because it sets the gateway tier size and nothing else |
| Messages sent, average | 40,000 per second | ASSUMED, at roughly 4 messages per connected user per hour |
| Peak multiplier | 2.5x, so 100,000 per second | ASSUMED, evening peaks in a consumer product are pronounced but not event-driven |
| Average recipients per message | 3 | ASSUMED, a mix of one-to-one and small groups |
| Largest group | 1,000 members | ASSUMED, and published, because group size is what turns fan-out into a different problem |
| Average message size | 300 bytes | ASSUMED, text with an envelope; media is a reference to the file service, not an inline payload |
| Live delivery, p99 | under 500 ms from send to a connected recipient | ASSUMED, the number below which a conversation feels synchronous |
| History read, p99 | under 300 ms for a page of 50 | ASSUMED, this is the scroll-back interaction |
| Deliveries per second at peak | 300,000 | DERIVED below |
| Messages per day | 3.46 billion | DERIVED below |
| Gateway nodes | 200 | DERIVED below |

### Estimation that eliminates (chat)

Every line ends with the option it kills. A number that kills nothing is deleted from the answer.

| Calculation | Result | What it rules out |
|---|---|---|
| 100,000 messages per second at peak times 3 recipients | 300,000 deliveries per second | Rules out a durable per-recipient write on the delivery path, and rules out any design where sending blocks on recipients |
| 40,000 per second times 86,400 seconds | 3.46 billion messages per day, 1.04 TB per day at 300 bytes | Rules out one unpartitioned table, and rules out keeping all history in a single hot store rather than tiering it |
| 10 million connections divided by 50,000 per gateway node | 200 gateway nodes | Rules out a design where any single process knows where a user is connected, and forces a routing layer between gateways |
| One gateway node failing, holding 50,000 connections | 50,000 simultaneous reconnections, each wanting to know what it missed | Rules out a reconnect path that costs a database query per connection, since a fleet-wide flap would be 10 million queries |
| A 1,000 member group at 10 messages per second | 10,000 deliveries per second from one conversation | Rules out fanning out to a member list held in one process, and makes large groups a routing problem rather than a delivery problem |
| Per-message read receipts in that group, 10 messages per second times 1,000 readers | 10,000 receipt writes per second from one conversation, against 10 message writes | Rules out a read flag per message per user. The receipt traffic is three orders of magnitude larger than the message traffic |
| A read watermark instead: one row per participant per conversation, updated when they catch up | At most 1,000 updates per second for that group, and far fewer once coalesced | Rules out the objection that read state must be per message. A monotonic position carries the same information |
| 10 million users, presence changing every 60 seconds, 50 contacts each | 166,000 status changes per second producing 8.3 million notifications per second | Rules out broadcasting presence to every contact, and rules out treating presence as free. It is larger than the messaging load |

!!! tip "Say this out loud in the room"

    "Read receipts at a thousand writes per message are three orders of magnitude larger than the messages themselves, and presence is bigger than both. So I want to design read state as a position rather than a flag, and I want to ask whether presence is actually required before I put it in."

### V0, the simplest thing that works (chat)

**Predict first.** V0 pushes messages over a socket and stores them for history. What does a client that was disconnected for 30 seconds see when it reconnects, and why?

??? note "Show answer"

    Nothing about the gap. It sees messages from the moment it reconnects onward. Push-only delivery has no notion of what a client missed, so unless the client asks history for the interval it was away, the messages are silently absent. The client cannot even ask correctly, because it does not know what it does not have. That gap is what the first deep dive is about.

```
+-----------+     +---------------------+     +-----------------+
| Client    | <-> | Chat service        | --> | Postgres        |
| socket    |     | socket, fan-out,    | <-- | messages,       |
+-----------+     | history read        |     | conversations,  |
                  +---------------------+     | members         |
                                              +-----------------+
```

**Caption.** V0 has three components. Clients hold sockets to a single chat service, which writes each message to Postgres and pushes it to any recipient connected to the same process. History is a paged read of the same table.

Why this is correct at a much smaller scale:

- One process that holds every connection knows exactly where everyone is, so routing is a hash table lookup and there is no session registry to build.
- Writing the message before pushing it means history is the source of truth and the push is an optimisation, which is the right relationship and stays right through every later version.
- A conversation-scoped index on the message table serves both live and historical reads, so there is one storage design rather than two.

!!! warning "When not to use even this much"

    Below roughly 10,000 concurrent users, and where a few seconds of latency is acceptable, do not open sockets at all. Long polling against the same history read gives you one code path instead of two, no connection state, and no reconnect storm. The measurement that justifies a persistent connection is a poll rate where over 90 percent of polls return empty, or a product requirement for typing indicators, which cannot be polled affordably.

### Breaking point, then V1 (chat)

**Breaking point one: one process cannot hold the connections, at roughly 50,000 concurrent.**

What fails: memory and file descriptors first, then the fan-out. Each connection costs buffers and a descriptor, and a single process holding 50,000 of them is at the edge of comfortable. Past that, the process cannot accept new connections, and the failure is total rather than gradual, because a rejected connection means a user is offline rather than slow. At 10 million connections, this is 200 processes, and the moment there are two, a sender and a recipient may not share one.

How you observe it: connection accept latency and rejection count per node, plus the fraction of messages whose recipients are not on the sending node. The second number goes from zero to nearly one as soon as you add the second node, which is the architectural signal, not a capacity signal.

```
+-----------+     +----------------+     +------------------+
| Client    | <-> | Gateway tier   | <-- | Session registry |
| socket    |     | 200 nodes      | --> | user -> node     |
+-----------+     +----------------+     +------------------+
                     |         |
                     |         v
              +---------------------+     +------------------+
              | Routing bus         | <-- | Message service  |
              | per conversation    |     | assign sequence, |
              +---------------------+     | persist          |
                                          +------------------+
                                                   |
                                                   v
                                          +------------------+
                                          | Message store    |
                                          | partitioned by   |
                                          | conversation     |
                                          +------------------+
```

**Caption.** V1 has six components. Clients connect to one of 200 gateway nodes. A message service assigns a per-conversation sequence number, persists the message, and publishes it on a routing bus. Gateways subscribe for the conversations their connected users belong to, using a session registry to know who is where.

V1 changes, each with the observation that justified it:

| Change | Justified by | Cost you accept |
|---|---|---|
| Gateway tier separate from message logic | 10 million connections over 200 nodes | Connection state is now a distributed problem, and a gateway deploy disconnects 50,000 users at once |
| Server-assigned per-conversation sequence | Two clients cannot agree on order using their own clocks | A conversation is now a serialisation point, so a very busy conversation has a throughput ceiling |
| Routing bus between message service and gateways | A sender and recipient are on different nodes almost always | Delivery is now at-most-once over the bus, and history is the authoritative recovery path |
| Message store partitioned by conversation | 1.04 TB per day | Cross-conversation queries, for example search, become a separate system |
| Session registry | Gateways must know which node holds a user | The registry is now on the delivery path, so its availability is your delivery availability |

**Breaking point two: reconnect storms and receipt amplification, at 50,000 reconnects and 10,000 receipt writes per second.**

What fails: two independent problems arriving together. When a gateway node restarts, its 50,000 clients reconnect within seconds, and if each reconnect issues a history query to find what it missed, that is 50,000 queries in a burst against a store sized for steady state. Separately, per-message read receipts in a 1,000 member group produce 10,000 writes per second against 10 message writes, so read state dominates the write path of the entire product.

How you observe it: for the storm, database query rate correlated with gateway restarts rather than with user activity. For receipts, the ratio of receipt writes to message writes, per conversation size bucket. The signature is that the ratio scales linearly with group size, which is the definition of a design that does not fit large groups.

V2 changes:

- Resume is a cursor, not a query. The client sends the highest per-conversation sequence it holds, and the gateway returns only what follows, from a short in-memory recent window per conversation. Only a client that is further behind than that window falls through to the message store.
- Read state becomes a monotonic watermark per participant per conversation, coalesced client-side so a rapid scroll produces one update rather than fifty.
- Reconnects are jittered by the server: the close frame carries a reconnect delay, spread over a window, so 50,000 clients do not return in the same second.
- Large groups drop per-recipient read state entirely and publish an aggregate count, because 1,000 individual watermarks are not information a user can act on anyway.

!!! warning "When not to run a routing bus"

    Below roughly 3 gateway nodes, route directly: look up the recipient's node in the session registry and send it there. A bus adds a hop, a subscription model and a second failure domain, all to solve a fan-out problem that does not exist at that size. The measurement that justifies a bus is a per-message fan-out that touches more than a handful of nodes, or a subscription set too large to enumerate per message.

### Deep dive one: one message identity across two transports

This is the dive that decides whether the product feels reliable. A chat client receives the same message from a live stream and from a history read, and everything below follows from making those two agree.

**The gap problem, stated precisely.** A client holds messages up to sequence 41 in a conversation. It disconnects. While it is away, sequences 42 to 47 are written. It reconnects and immediately receives sequence 48 from the live stream. The client now has a hole and, without a sequence number, no way to detect it. It renders 48 directly after 41 and the user has lost six messages permanently.

**A per-conversation monotonic sequence is what makes the hole visible.** Not a timestamp, because clocks disagree and two messages can share a millisecond. Not a global sequence, because that would serialise the entire product. A sequence per conversation, assigned by the server at write time, gives every client a simple rule: if the sequence you received is more than one above the sequence you hold, fetch the range in between.

| Property the sequence needs | Why |
|---|---|
| Monotonic within a conversation | Gap detection is a subtraction, not a comparison of timestamps |
| Assigned server side | Two clients cannot agree on a shared counter |
| Dense, with no skipped values | A sparse sequence makes a gap indistinguishable from a hole, so the client cannot tell when it is caught up |
| Returned on both transports | The live frame and the history record must carry the same value, or the client cannot reconcile them |

**Density is the constraint people miss.** If sequences can be skipped, for example because a write was rolled back, the client sees a hole that will never be filled and either retries forever or gives up and shows a gap. Either assign the sequence only after the write commits, or publish a tombstone for any allocated sequence that will never carry a message.

**The client's reconciliation rule, in three lines.** This is worth stating explicitly in the room, because it is the contract:

- On receiving a live message with sequence n, if you hold n minus 1, append it.
- If you hold less than n minus 1, request the range from your highest held sequence to n, then append.
- Deduplicate on the message identifier, because a range fetch and the live stream will overlap.

**Send-side identity is a separate problem with the same shape.** A client that sends a message and loses the connection before the acknowledgement does not know whether it was written. It retries. Without a client-supplied identifier on the send, that is a duplicate message in the conversation, which users see. With one, the server returns the original message and its sequence.

This is the same retry key idea as the payments case study, applied to a much cheaper effect, and it is why the send path needs a client-generated identifier from version one.

**Edits and deletes need their own sequence positions.** An edit is not a mutation of message 42, it is a new event at sequence 49 that refers to 42. Otherwise a client whose history was fetched before the edit has no way to learn about it: history reads return the current text and the client already has an old copy. Treating mutations as events is what makes the stream self-sufficient.

!!! warning "When not to use per-conversation sequences"

    In a conversation with a single writer, for example a system notification channel, the writer's own counter is enough and a server-assigned sequence adds a serialisation point for nothing. The measurement that justifies server-assigned sequences is any conversation with concurrent writers, which is essentially all of chat, or the first user report of messages appearing in the wrong order.

### Deep dive two: read state as a position, not a flag

The dives on this page vary by problem. The comment service dived on paging and the pub/sub service on cursor ownership; this dive is about a product-visible feature whose naive shape is three orders of magnitude too expensive, which is a different failure from either.

**The estimation already killed the naive design.** Ten messages per second in a 1,000 member group, with a read flag per message per member, is 10,000 writes per second against 10. Read state would be 99.9 percent of the write volume of the entire product, to render a small tick mark.

**A watermark carries the same information at one row per participant per conversation.**

| Model | Rows | Write rate in the 1,000 member group | What it can express |
|---|---|---|---|
| Flag per message per participant | Messages times participants | 10,000 per second | Exactly which messages were read, including out of order |
| Watermark per participant per conversation | Participants | Up to 1,000 per second before coalescing, far fewer after | Everything up to position n has been read |
| Aggregate count per message | Messages | 10 per second | How many have read it, not who |

**Monotonic is what makes the watermark safe.** Read state updates arrive out of order over an unreliable network. If an update can move the watermark backwards, a delayed packet marks messages unread again and the user sees a phantom notification. The rule is a single comparison at write time: accept only a position strictly greater than the stored one. Everything else about read receipts is easier once this holds.

**Coalescing happens on the client, and it must.** A user scrolling through a conversation crosses fifty messages in two seconds. Sending fifty updates is the same amplification in a different place. The client sends at most one update per conversation per interval, carrying the highest position reached. The interval is a published number, and it is why the read indicator is allowed to be a second or two behind.

**Delivered and read are different watermarks, and only one of them is cheap.**

| State | Where it is known | Cost |
|---|---|---|
| Delivered | The gateway, when it writes the frame to the socket | Cheap, one update per participant per burst, and it can be batched entirely server side |
| Read | The client, when the message is visible on screen | Requires a client round trip, so it is bounded by the coalescing interval |

**Group size changes the feature, not just its cost.** Per-participant read state in a 1,000 member group is 1,000 rows to load in order to render a number nobody reads individually. Above a threshold, publish a count and drop the identities. This is a product decision that the estimation forces, and saying "above 100 members I would change what the feature shows" is a stronger answer than optimising the storage of something that should not be stored.

**Privacy is a design input here, not a footnote.** If read state can be disabled by a user, the watermark must still be recorded for their own device synchronisation while not being disclosed to others. That means the storage is unchanged and the disclosure is a separate authorisation decision, which is a much better design than not recording it, because the user's own devices need it to agree on what is unread.

!!! warning "When not to store per-participant read state"

    In a one-to-one conversation with no read indicator in the product, do not store it. Two rows per conversation is cheap but the feature is not free: it creates a privacy commitment, a synchronisation path across a user's devices, and a monotonicity invariant to defend. The measurement that justifies it is the product actually rendering the state, and the threshold that justifies dropping identities is a group size above roughly 100.

### Failure modes and evaluation (chat)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Silent gap | A client misses messages during a disconnection and never knows | Dense per-conversation sequences plus a client rule that fetches any range it does not hold |
| Thundering herd | A gateway restarts and 50,000 clients reconnect within a second | Server-supplied jittered reconnect delay in the close frame, plus a recent-message window that answers resumes without touching storage |
| Write amplification | Per-message read receipts produce 1,000 writes per message in a large group | Monotonic watermarks, client-side coalescing, and aggregate counts above a group size threshold |
| Duplicate message | A send is retried after a lost acknowledgement | Client-generated message identifier, with the server returning the original message and its sequence |
| Hot conversation | A very busy group serialises on its sequence assignment | Publish the per-conversation write ceiling, and treat a conversation approaching it as a product problem, not a scaling one |
| Split brain | Two gateways both believe they hold a user's session after a partition | Session registry entries carry a monotonic epoch, and the older gateway's writes are rejected |
| Clock skew | Ordering by client timestamp puts a message before one it replies to | Never order by client time. Client time is display metadata; server sequence is order |
| Metastable failure | Reconnects cause load, which causes disconnects, which cause reconnects | Admission control on the gateway, shedding reconnects with a retry delay rather than accepting and failing |

Service level indicators to publish:

- Send-to-delivery latency at p99 for connected recipients, measured from server receipt to socket write, so a client's slow network is not counted against the service.
- Gap-fill rate: the fraction of client sessions that had to fetch a missing range. A rise means delivery is dropping frames, and it is the earliest signal of a routing problem.
- Reconnect rate per gateway node, which distinguishes a fleet-wide network event from one unhealthy node.
- Duplicate send rate, measured as sends carrying a client identifier already seen, which is a direct measure of acknowledgement loss.
- Read watermark rejection rate, counting updates refused for moving backwards. A non-trivial rate means a client is not coalescing correctly.
- History read latency at p99 for a page of 50, split by whether the page came from the recent window or from storage.

Alert on gap-fill rate, because it is the indicator that catches lost messages, which is the failure users describe as the product being broken. Alert on reconnect rate crossing a fleet-wide threshold, since that is the leading edge of a metastable loop. Do not alert on send-to-delivery latency for disconnected recipients; that is not a service failure and it will train you to ignore the page.

Re-checking against the non-functional requirements:

| Requirement | Does V2 meet it? |
|---|---|
| 10 million concurrent connections | Yes, at 200 gateway nodes of 50,000, with the session registry as the new dependency to keep highly available |
| 300,000 deliveries per second at peak | Yes, since delivery is a socket write off a routing bus, with no durable per-recipient write |
| Live delivery p99 under 500 ms | Yes for connected recipients. Explicitly not promised for a client that is reconnecting, and the contract says so |
| History read p99 under 300 ms | Yes from the recent window; from partitioned storage it depends on how far back the read is, which is why the window exists |
| Largest group of 1,000 | Yes for messages. Per-participant read identities are dropped above roughly 100 members, which is a change to the feature and must be stated |
| 3.46 billion messages per day | Yes, partitioned by conversation, with old partitions tiered. Cross-conversation search is a separate system |
| Resume without loss or duplication | Yes, given dense sequences and deduplication on message identifier. Neither works without the other |

### Where this answer is a simplification (chat)

- **End-to-end encryption invalidates several boxes here.** If the server holds only ciphertext, it cannot generate previews, cannot search, cannot moderate, and cannot render a notification body. Key distribution and multi-device key agreement become the dominant design problem, and the message store becomes almost trivial by comparison.
- **Multi-device is a second synchronisation problem.** One user with four devices needs read state, message state and drafts to agree across all of them, which means the watermark is per user, not per session, and every device needs its own gap-fill.
- **Media is a different service, and this design assumes so.** A message carries a reference into a file service. The upload session, resumability and lifecycle questions are covered by the file and blob case study earlier on this page.
- **Notification delivery to a backgrounded device is not this transport.** It goes through platform push services with their own delivery semantics, payload limits and no ordering guarantee at all, which means the notification and the socket can arrive in either order.
- **Primary sources with dates.** RFC 6455, [The WebSocket Protocol](https://www.rfc-editor.org/rfc/rfc6455.html) (December 2011, accessed August 2026), defines the transport, including the close frame this design uses to carry a reconnect delay. RFC 9110, [HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) (June 2022, accessed August 2026), covers the history read semantics. For the connection tier, routing and partitioning building blocks, see [Modern System Design](modern-system-design.md), and for client-side storage and reconnection behaviour see [Mobile System Design](mobile-system-design.md).

### Level delta (chat)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Transport | Draws a socket and says messages are pushed | Separates gateways from message logic after deriving 200 nodes | Treats push as an optimisation over an authoritative history read, so a lost frame is recoverable by design |
| Ordering | Orders by timestamp | Assigns a server-side per-conversation sequence | Requires density, and names what a sparse sequence does to a client's gap detection |
| Resume | Says the client refetches on reconnect | Sends the client's last sequence and returns the delta | Adds a recent-message window so 50,000 reconnects do not become 50,000 storage queries |
| Read state | Stores a read flag per message | Uses a monotonic watermark after deriving the 1,000 to 1 amplification | Changes what the feature displays above a group size threshold, rather than optimising storage for something nobody reads |
| Duplicates | Does not mention them | Adds a client-generated message identifier | Notes that gap fill and live delivery overlap by design, so deduplication is required on the client too |
| Operations | Watches latency | Publishes delivery latency and reconnect rate | Alerts on gap-fill rate, which is the only indicator that detects lost messages |

What each level added, in one line each:

- **Mid to senior:** the two transports are reconciled by a single explicit ordering mechanism, and every cost is derived from group size rather than assumed.
- **Senior to staff:** the design assumes its own delivery will fail, and the product feature is reshaped where the numbers say the naive version cannot exist.

---
## Case study 8: Ride hailing interface

Two parties act on the same object at the same time, from two devices, on mobile networks, with money attached to the outcome. That is the whole difficulty of this prompt, and it is why the interesting design work is in transitions rather than in resources.

### The prompt (ride hailing)

"Design the interface the rider app and the driver app use to request a ride, follow it, and cancel it."

### Questions that fork the design (ride hailing)

Ask these because each answer moves a box or changes a resource. Do not recite them.

| Question worth asking | If the answer is A the design becomes | If the answer is B it becomes |
|---|---|---|
| Is the price fixed when the ride is requested, or computed at the end? | Quoted upfront: a quote is a resource with an expiry, and the fare is a promise you must honour | Metered: the fare is derived from the trip, disputes are about distance and time, and no quote resource exists |
| Can the rider see the driver's location before pickup? | No: state transitions are the only updates, and the interface is small | Yes: you have a continuous stream to design, and location ingest becomes the dominant traffic in the system |
| Can either side cancel at any time? | Rider only, before assignment: cancellation is a single transition with no race | Both, at any point: two devices race on one object, and cancellation fees make the outcome financially material |
| Is a driver assigned by the system, or do drivers accept offers? | System assigns: one transition, no offer resource, no timeout to design | Drivers accept: an offer is a resource with a lifetime, and every expiry is a re-match |
| Is the ride's state machine public? | Kept internal, exposed as a small set of coarse states: you can change it freely | Exposed in detail: every state is a compatibility commitment, and adding one may break clients |
| Do you need the trip path, or just the endpoints? | Endpoints: location is ephemeral and nothing is retained | Full path: location becomes durable data with retention, privacy and access rules |
| How many riders and drivers share a city at peak? | Hundreds: matching is a scan, and no spatial index is needed | Tens of thousands: matching needs a spatial index and a bounded search, and supply becomes a contended resource |
| Is there a scheduled-in-advance ride? | No: every ride is now, and there is no future state to hold | Yes: a ride exists for hours before it starts, so the state machine gains a whole branch and expiry semantics |

### Requirements (ride hailing)

Functional:

- Request a ride from a pickup to a destination, with an upfront quote.
- Match a driver, offer the ride, and record acceptance or expiry.
- Stream the assigned driver's location to the rider before pickup and during the trip.
- Advance the ride through its lifecycle from both apps, with the server as the authority.
- Cancel from either side, with a defined fee outcome.
- Produce a final fare and a receipt.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Rides per day | 5 million | ASSUMED, a large city network; ask, because it decides nothing about location load, which dominates |
| Online drivers at peak | 500,000 | ASSUMED, and this is the number that actually sizes the system |
| Location ping interval | one every 4 seconds while online | ASSUMED, a compromise between map smoothness and phone battery |
| Average ride duration | 15 minutes | ASSUMED, an urban trip including pickup |
| Cancellation rate | 15 percent of requests | ASSUMED, mostly before pickup, and high enough that cancellation is a main path rather than an edge case |
| Offer acceptance window | 15 seconds | ASSUMED, long enough to react while driving and short enough to re-match |
| State transition latency, p99 | under 300 ms | ASSUMED, both apps show a button that must respond |
| Quote validity | 5 minutes | ASSUMED, long enough to choose, short enough that conditions have not changed |
| Location pings per second | 125,000 | DERIVED below |
| Concurrent rides | 52,083 | DERIVED below |
| Ride requests per second at peak | 174 | DERIVED below |

### Estimation that eliminates (ride hailing)

Every line ends with the option it kills. A number that kills nothing is deleted from the answer.

| Calculation | Result | What it rules out |
|---|---|---|
| 500,000 drivers divided by a 4 second ping interval | 125,000 location updates per second | Rules out writing location to a relational store, and rules out treating location as part of the ride resource |
| 125,000 per second times 86,400 seconds | 10.8 billion location points per day, 1.08 TB at 100 bytes each | Rules out retaining every point durably by default, and makes path retention a deliberate product decision with a cost |
| 5 million rides per day times 15 minutes, divided by 1,440 minutes | 52,083 concurrent rides on average | Rules out a per-ride durable subscription record, and sets the size of the live tracking fan-out |
| 52,083 concurrent rides with a location update every 2 seconds to the rider | 26,000 outbound updates per second | Rules out sending every ingested ping onward. Roughly one in five ingested points reaches a rider, so the ingest and egress paths are different systems |
| 5 million rides divided by 86,400 seconds, times a 3x peak | 58 requests per second average, 174 at peak | Rules out any conversation about scaling the ride request path. It is 700 times smaller than location ingest |
| 5 million rides times 8 lifecycle transitions | 40 million transitions per day, 463 per second | Rules out a durable event per transition being expensive, so a full transition history is affordable and should be kept |
| 15 percent of 5 million | 750,000 cancellations per day, 8.7 per second | Rules out treating cancellation as an exception path. It happens more often than many products' main path |
| A 15 second offer window with 174 requests per second at peak | Up to 2,610 offers outstanding at any moment | Rules out holding offers only in one process's memory, and makes offer expiry a scheduled operation rather than a lazy check |

!!! tip "Say this out loud in the room"

    "Ride requests are 174 per second and location is 125,000 per second, so these are two different systems that happen to share a resource. I want to design the transition contract carefully, because that is where two devices fight, and I want location to never touch the same store."

### V0, the simplest thing that works (ride hailing)

**Predict first.** V0 exposes a ride resource with a status field that either app can write. Name the sequence of two requests that leaves the system in a state neither app expected.

??? note "Show answer"

    The rider sends cancel while the driver sends arrived, within the same second. Both are valid writes to a status field. Whichever lands second wins, so a ride can end up cancelled after arrival or arrived after cancellation, depending on network timing. Neither app can predict which, both will render the wrong screen, and the cancellation fee is decided by a race.

```
+-----------+     +---------------------+     +-----------------+
| Rider app | --> | Ride API            | --> | Postgres        |
| Driver app| <-- | request, set status,| <-- | rides, drivers, |
+-----------+     | store location      |     | locations       |
                  +---------------------+     +-----------------+
```

**Caption.** V0 has three components. Both apps call one ride service, which stores the ride, its status and the latest driver location in Postgres. Riders poll the ride resource to follow the driver.

Why this is correct at a much smaller scale:

- At a few hundred drivers, location in the same store is fine: a few pings per second is nothing, and having one store means the ride and the driver's position are always consistent.
- Polling a ride resource every two seconds is a correct and very simple live-tracking design when concurrent ride count is in the hundreds.
- One writable status field is the smallest possible lifecycle contract, and it is exactly the thing the first breaking point destroys.

!!! warning "When not to use even this much"

    Below roughly 50 concurrent rides, drop the location table and let the driver's app send its position only when the rider's app asks. That is fewer moving parts and better for battery. The measurement that justifies continuous ingest is a rider-visible map, and the threshold that justifies a separate location system is location writes exceeding roughly 10 percent of your total write volume.

### Breaking point, then V1 (ride hailing)

**Breaking point one: location ingest overwhelms the ride store at roughly 5,000 pings per second.**

What fails: each ping is an update to a row with an index on a spatial or driver key. At 5,000 updates per second the table's index churn dominates, and because the same store holds ride rows, ride transitions start queueing behind location writes. The observable symptom is a ride transition p99 that rises with driver count rather than with ride count, which is a strong signal that two unrelated workloads share a resource.

How you observe it: write latency on the ride table broken down by statement type, and the ratio of location writes to ride writes. At 500,000 drivers that ratio is 125,000 to 58, or roughly 2,150 to 1, which is the whole argument for separating them.

```
+------------+ 1  +-------------------+     +------------------+
| Driver app | -> | Location ingest   | --> | Geospatial index |
+------------+    | validate, sample  |     | in memory        |
                  +-------------------+     +------------------+
                            |                        |
                            v                        |
+------------+    +-------------------+     +------------------+
| Rider app  | <- | Tracking stream   |     | Matching service |
+------------+    | per active ride   |     | bounded search   |
      |           +-------------------+     +------------------+
      | 2                                            |
      v                                              v
+-------------------+                       +------------------+
| Ride service      | --------------------> | Postgres         |
| transitions, fare |                       | rides, offers,   |
+-------------------+                       | transitions      |
```

**Caption.** V1 has seven components. Driver location goes to a separate ingest path feeding an in-memory geospatial index, which the matching service searches. A tracking stream pushes the assigned driver's position to the rider. The ride service owns transitions, offers and fare, in Postgres.

V1 changes, each with the observation that justified it:

| Change | Justified by | Cost you accept |
|---|---|---|
| Location leaves the relational store | 2,150 location writes per ride write | Location is now not durable by default, so a path reconstruction after the fact is a separate opt-in pipeline |
| In-memory geospatial index | Matching needs a bounded search over 500,000 positions | The index is rebuildable but not durable, so a total loss means a few seconds of degraded matching |
| Offer becomes its own resource with a 15 second lifetime | Up to 2,610 outstanding offers at peak | Expiry is now a scheduled sweep, and an expired offer must be distinguishable from a declined one in the contract |
| Tracking stream separate from the ride resource | 26,000 outbound location updates per second | Riders now consume two things, a state channel and a location channel, and the two can disagree momentarily |
| Every transition written as an event, not just a status update | 463 transitions per second is cheap and disputes need history | The ride resource now has a history collection, which is a second thing to page and to authorise |

**Breaking point two: concurrent transitions from two devices, at roughly 8.7 cancellations per second.**

What fails: with 750,000 cancellations per day and a 15 minute average ride, some fraction of them are submitted within seconds of a driver-side transition. A last-write-wins status field resolves those by network timing. The financial consequence is direct: whether a cancellation fee applies usually depends on whether the driver had arrived, so a race decides who pays. Support cannot reconstruct the truth because only the final status was stored.

How you observe it: a counter of transition requests rejected or overwritten where two writes to one ride arrived within a small window, split by the pair of transitions involved. The signature is that the pairs cluster: cancel against arrived, and cancel against started, are almost all of it.

V2 changes:

- Transitions become named operations, not writes to a status field. Each carries the state the caller believes the ride is in, and the server rejects the operation with a precise error if the ride has moved on. Last-write-wins is replaced by a conditional transition.
- Each transition carries a client-supplied key so a retry after a lost response is safe, returning the original outcome rather than attempting the transition again.
- The transition event log becomes the source of truth for fee determination, with the server's receive time as the ordering authority, so a dispute is answerable from data.
- The cancellation outcome, including any fee, is computed and returned by the cancel operation itself rather than derived later, so both apps see the same answer at the same moment.

!!! warning "When not to expose named transitions"

    If only one party can ever change the object, a writable status field is simpler and the conditional machinery is over-engineering: there is no race to resolve. The measurement that justifies named transitions is any observed pair of conflicting writes within your network round trip time, or any state change whose outcome affects money.

### Deep dive one: publishing a state machine you will have to change

This dive exists because the states are the interface here. Every screen in both apps is bound to one, and the day you add a state is the day you find out how many clients you cannot upgrade.

**A status field and a transition operation are not the same contract.**

| Design | What the client sends | What breaks under concurrency | What breaks under evolution |
|---|---|---|---|
| Writable status field | The new status | Last write wins, so timing decides the outcome | Nothing, and that is the problem: clients invent transitions you never intended |
| Named transition operation | The verb, plus the state it expects | Nothing: a stale expectation is rejected with a precise error | Adding a verb is additive; removing one is breaking, and you can see who calls it |

**Expected-state on every transition is the cheap version of optimistic concurrency.** The driver app sends "arrive, expecting assigned". If the ride is already cancelled, the server returns a conflict naming the actual state. The app then renders the truth instead of a screen based on a write it assumed succeeded.

RFC 9110, [HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) (June 2022, accessed August 2026), covers the conditional request and 409 semantics this leans on, and RFC 9457, [Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html) (July 2023, accessed August 2026), gives you a body shape that can carry the actual state as a machine-readable field.

**Adding a state is a breaking change unless you designed for it.** Suppose you add "driver waiting at pickup" between arrived and started. Old clients receive an unknown state string and, depending on how they were written, crash, show nothing, or fall into a default branch. Three mechanisms make the addition survivable, and all three must be decided before version one ships:

| Mechanism | What it does | What it costs |
|---|---|---|
| A coarse state alongside the detailed one | Old clients bind to a small, stable set; new clients read the detail | Two representations to keep consistent, forever |
| A documented rule that unknown states are treated as the previous known one | Turns an unknown value into a defined behaviour | Only works if clients actually implemented it, so it must be in the first version of the software development kit |
| Legal transitions delivered as data | The client renders buttons from the server's list rather than from a hardcoded map | The client can no longer work offline without a cached list, and the list becomes a compatibility surface of its own |

**Deliver the allowed next transitions with the ride.** This is the single highest-leverage decision in this case study. If the server says "from here, the allowed operations are cancel and arrive", the client renders exactly those buttons. Adding a state, or changing when cancellation is free, then requires no client release. This is what module 11 on this page calls moving a decision from code you cannot upgrade into data you can.

**The states themselves should be few and coarse in public.** Internally you may have twenty. Externally, publish the ones a client must distinguish to render a screen, and nothing else. Every additional published state is a value that some client will write a conditional against.

**Terminal states must be genuinely terminal.** Once a ride is completed or cancelled, no transition may leave it. If a late-arriving driver-side transition can reopen a completed ride, then receipts, payouts and dispute records all become mutable after the fact. The rule is one line in the server and it prevents an entire class of financial incident.

!!! warning "When not to deliver transitions as data"

    If the client must work with no connectivity, a server-delivered transition list is unusable at exactly the moment it is needed, and a hardcoded map with a cached fallback is better. The measurement that justifies server-delivered transitions is a release cycle you do not control, for example an app store review queue, combined with a policy that changes more often than you can ship.

### Deep dive two: cancellation is a race with money attached

The dives on this page vary by problem. The first dive covered the shape of the state contract; this one covers the single transition where the contract is tested, because it is the only one where two parties want different outcomes and the answer costs someone money.

**Cancellation is four questions, and candidates usually answer only the first.**

| Question | Why it is not obvious |
|---|---|
| Who is allowed to cancel, and when? | A rider cancelling after pickup is a different event from one cancelling before assignment, and the same verb covers both |
| What happens to the driver? | They are released to the pool, but if they had driven eight minutes they have a claim, so release and compensation are separate outcomes |
| Is there a fee, and who decides? | The fee depends on elapsed state, and both apps will display an answer before the server has decided one |
| What if the cancel and another transition cross? | This is the race, and it is the reason the fee cannot be computed client side |

**The race, made concrete.** The fee policy says cancellation is free until the driver arrives. The driver sends arrive at 10:00:03.100. The rider sends cancel at 10:00:03.050, but on a slower network, so it reaches the server at 10:00:03.400. Arrival is recorded first. The rider is charged for cancelling a ride they cancelled before arrival happened.

**Three defensible policies, and the point is to pick one out loud.**

| Policy | Rule | Consequence |
|---|---|---|
| Server receive order | Whichever request arrives first wins | Simple, explainable, and penalises the party on the worse network |
| Client-stamped time with a bounded skew allowance | Order by client timestamp when the two are within a few seconds | Fairer in principle, and requires trusting a device's clock, which is a claim by an interested party |
| Grace window on the fee | Order by server receive, but waive the fee if the cancel arrived within a few seconds of the transition | Costs money, removes almost all disputes, and is the one I would defend |

The third is a product decision that removes an engineering problem. Saying that trade-off out loud, that you can spend a small amount of money to avoid a hard distributed agreement problem, is a staff-level move.

**Never let the client compute the fee.** Both apps will show the user something the moment the button is pressed. If they compute it locally they will sometimes show a different number from the one charged, and every mismatch becomes a support contact. The cancel operation returns the outcome, including the fee, and the app renders what came back rather than what it predicted. An optimistic local render is acceptable only if it is visually marked as pending.

**Cancellation must be idempotent and must return the same answer twice.** A rider on a poor connection presses cancel, sees nothing, and presses again. Without a client-supplied key, the second request either fails with a confusing conflict or, worse, is treated as a second cancellation event. With one, it returns the first outcome unchanged, which is exactly the retry contract from the payments case study applied to a different effect.

**Offer expiry is a cancellation you did not design as one.** When a driver does not accept within 15 seconds, something must transition the ride. If that happens lazily, on the next read, then a driver who accepts at 15.2 seconds may win against an expiry that has not been evaluated. Expiry must be an active, server-timed transition, recorded in the event log like any other, so that acceptance and expiry are ordered by the same authority.

**The event log is what makes any of this defensible afterwards.** A dispute six weeks later is answerable only if you stored every transition with its server receive time, its actor and its resulting state. At 463 transitions per second this costs almost nothing, which is why the estimation section keeps it: the number exists to prove the log is affordable.

!!! warning "When not to charge a cancellation fee at all"

    If the supply side is not compensated for time already spent, a fee is pure friction: it creates disputes, requires a defensible ordering policy and complicates every client. Below roughly a few thousand rides per day, waiving fees entirely is cheaper than building and operating the machinery to decide them. The measurement that justifies a fee is driver time lost to cancellations becoming a visible fraction of driver earnings.

### Failure modes and evaluation (ride hailing)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Lost update | Two devices write the status field and one change disappears | Named transitions carrying the expected state, rejected with a conflict naming the actual state |
| Duplicate effect | A cancel is retried after a lost response and cancels twice | Client-supplied transition key returning the original outcome |
| Clock skew | A device's clock is minutes off and its timestamps decide a fee | Server receive time is authoritative; client time is metadata only, and skew beyond a threshold is reported back |
| Hot partition | One city's drivers concentrate in a small geospatial cell at an event | Cell size adapted to density, and a matching search bounded by candidate count rather than by radius alone |
| Retry storm | A matching slowdown causes apps to resubmit ride requests | Idempotent request keys, plus an explicit pending state so the app has something to render instead of retrying |
| Metastable failure | Offers expire faster than drivers accept, so every ride re-matches repeatedly | Cap re-match attempts, widen the offer window under load rather than shortening it, and shed new requests before degrading in-flight ones |
| Stale location | The map shows a driver where they were 40 seconds ago | Every location carries its capture time, and clients render staleness rather than hiding it |
| Split brain | Two matching workers offer the same driver to two rides | A short lease on the driver taken at offer time, with a monotonic epoch so the older offer cannot win |

Service level indicators to publish:

- Transition latency at p99, per transition type, because accept and cancel have different urgency and averaging them hides both.
- Conflicting transition rate: transitions rejected because the expected state was stale, split by the pair involved. This is the direct measure of how often the race occurs.
- Location freshness at p99, measured as the age of the position rendered to a rider, not as ingest latency.
- Offer acceptance rate and offer expiry rate, since a rising expiry rate is the leading indicator of a matching problem or a supply problem.
- Duplicate transition rate, measured as requests carrying a key already seen, which quantifies acknowledgement loss on mobile networks.
- Fee dispute rate per thousand cancellations, which is the business-visible consequence of the ordering policy.

Alert on conflicting transition rate crossing a threshold, because a jump means either a client regression or a latency problem that is now costing users money. Alert on offer expiry rate, since it degrades the product before any latency indicator moves. Do not alert on location freshness globally: it is dominated by individual devices with poor connectivity and belongs on a dashboard segmented by network type.

Re-checking against the non-functional requirements:

| Requirement | Does V2 meet it? |
|---|---|
| 125,000 location pings per second | Yes, on a separate ingest path into an in-memory index, with nothing durable on the hot path |
| 26,000 outbound tracking updates per second | Yes, and note it is one fifth of ingest, which is why sampling on the way out is part of the design rather than an optimisation |
| 174 ride requests per second at peak | Yes, by three orders of magnitude. This requirement never constrained anything |
| Transition p99 under 300 ms | Yes, one conditional write plus one event append, both indexed and both local |
| 15 second offer window | Yes, with expiry as an actively scheduled transition rather than a lazy check, which is what makes acceptance and expiry orderable |
| 8.7 cancellations per second | Yes, and each one now returns its fee outcome from the server, which is what removes the client-side mismatch |
| Dispute resolution | Yes, from the transition event log at 463 per second. Without the log, this requirement is unmeetable at any cost |

### Where this answer is a simplification (ride hailing)

- **Matching is an optimisation problem, not a nearest-neighbour query.** Real assignment balances waiting riders against expected future supply, and a greedy nearest match is measurably worse for both sides. That work sits behind the interface and constrains how long a request may stay in a pending state.
- **Pricing is a system of its own.** An upfront quote is a prediction with financial risk attached, and the quote resource's expiry exists because the prediction decays. Surge, promotions and taxes all land in the same fare object and each has its own rounding and disclosure rules.
- **Location data is regulated.** Retention, access, redaction and export are legal obligations in many jurisdictions, and the default of not persisting pings is as much a compliance choice as a cost one.
- **Safety features change the state machine.** Emergency assistance, trip sharing and route deviation detection all add states or side channels, and several of them must work when the app is backgrounded, which pulls in platform push behaviour.
- **Primary sources with dates.** RFC 9110, [HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) (June 2022, accessed August 2026), defines the conditional and conflict semantics behind expected-state transitions. RFC 9457, [Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html) (July 2023, accessed August 2026), gives the error body that can carry the actual state machine-readably. For the geospatial indexing and streaming building blocks, see [Modern System Design](modern-system-design.md); for battery and connectivity constraints on both apps, see [Mobile System Design](mobile-system-design.md).

### Level delta (ride hailing)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Lifecycle | Exposes a writable status field | Replaces it with named transitions carrying an expected state | Delivers the allowed transitions as data, so policy changes need no client release |
| Concurrency | Assumes one writer at a time | Names the cancel against arrive race and resolves it by server receive order | Proposes a grace window that spends money to avoid a distributed agreement problem, and says why |
| Location | Stores location on the ride | Separates ingest after deriving 2,150 location writes per ride write | Notices outbound is one fifth of ingest and designs the two paths independently |
| Retries | Does not mention them | Adds a client-supplied key so cancel is idempotent | Extends it to every transition, since mobile networks lose acknowledgements on all of them |
| Evolution | Publishes the full internal state list | Publishes a coarse public set | Defines unknown-state behaviour in the first software development kit, because that rule cannot be added later |
| Evidence | Stores the current status | Stores a transition event log | Derives that the log costs 463 writes per second, making dispute resolution affordable rather than aspirational |

What each level added, in one line each:

- **Mid to senior:** concurrency between two independent actors is treated as the default rather than the exception, and unrelated workloads are separated on a derived ratio.
- **Senior to staff:** the parts of the contract that cannot be changed later, state vocabulary and unknown-value behaviour, are settled in version one, and one hard problem is deliberately traded for a small, bounded cost.

---
## Case study 9: Tool-calling surface for an AI agent

Every other case study on this page has a caller that reads documentation, ships a fix and never calls a removed field again. This one has a caller that reads your schema on every request, may call your tool for reasons you did not anticipate, and cannot be sent a migration email. The contract has to carry more weight because the caller carries less.

### The prompt (tool surface)

"Design the interface through which a language-model agent calls your company's internal tools, safely and within a budget."

### Questions that fork the design (tool surface)

Ask these because each answer moves a box or changes a resource. Do not recite them.

| Question worth asking | If the answer is A the design becomes | If the answer is B it becomes |
|---|---|---|
| Are tools read-only, or do they change state? | Read-only: a wrong call costs tokens and nothing else, and no approval machinery is needed | State-changing: every call needs an effect declaration, an approval policy and a retry contract |
| Is there a human in the loop, always or sometimes? | Always: approval is the default and the design is a queue with a user attached | Sometimes or never: you need a declared risk class per tool, and a policy that decides without a person |
| How many tools exist? | A dozen: send every schema every turn and stop thinking about it | Hundreds: schemas no longer fit a request budget, and tool selection becomes its own subsystem |
| Who owns each tool, one team or many? | One: naming, error shapes and versioning stay consistent by accident | Many: you need a namespace, a schema lint and a registration review, or the surface degrades into inconsistency |
| Can a tool take longer than a model turn? | No: every call is synchronous and the contract is one request and one response | Yes: a call returns a handle, and the agent needs polling, cancellation and a story for results it never collects |
| Is the caller's output ever passed straight to another tool? | No: a person reviews between steps | Yes: one tool's output is the next tool's untrusted input, and injection becomes a first-class threat |
| Is spend bounded per session or per tenant? | Per tenant, loosely: metering is a billing feature | Per session, hard: the budget is enforced mid-run and the agent must be told it is running out |
| Do tools need to be discoverable at runtime? | No: the working set is fixed per agent and can be reviewed | Yes: tools can be added without a deploy, and a schema lint at registration is the only quality gate you have |

### Requirements (tool surface)

Functional:

- Register a tool with a name, a description, an input schema and a declared effect class.
- Return a working set of tool schemas relevant to the current task, rather than every tool.
- Execute a call, validate its arguments against the schema, and return a structured result or a structured error.
- Require approval before executing any call whose declared effect class demands it.
- Start a long-running call, report its progress, and allow cancellation.
- Meter tokens and calls per session, and refuse further work when a budget is exhausted.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Agent sessions per day | 200,000 | ASSUMED, an internal assistant used across a mid-size company; ask, because it sets everything downstream |
| Model turns per session, average | 12 | ASSUMED, a task that needs several tool calls and a summary at the end |
| Registered tools | 400 | ASSUMED, roughly one per service capability at a few hundred services |
| Average schema size | 350 tokens | ASSUMED, a name, a description and five to eight described parameters |
| Tool call latency, p99 | under 2 seconds | ASSUMED, because a slow tool stalls the whole turn and the user is watching |
| Malformed argument rate | 3 percent of calls | ASSUMED, and non-zero by construction: the caller is probabilistic, so this is a design input, not a bug |
| Calls with side effects | 20 percent of calls | ASSUMED, most agent work is reading; the minority that writes carries all the risk |
| Session budget | hard cap, enforced mid-run | GIVEN, because an unbounded loop is the failure that ends the project |
| Tokens per turn if every schema is sent | 140,000 | DERIVED below |
| Tokens per day if every schema is sent | 336 billion | DERIVED below |
| Side-effecting calls per day | 480,000 | DERIVED below |

### Estimation that eliminates (tool surface)

Every line ends with the option it kills. A number that kills nothing is deleted from the answer.

| Calculation | Result | What it rules out |
|---|---|---|
| 400 tools times 350 tokens | 140,000 tokens of schema in every single request | Rules out sending the full catalogue. Before any latency or accuracy argument, this alone exceeds what a request budget can spend on instructions the model mostly will not use |
| 200,000 sessions times 12 turns | 2.4 million model turns per day, roughly 28 per second | Rules out treating each turn as expensive infrastructure work. The turn rate is small; the token volume per turn is the problem |
| 2.4 million turns times 140,000 tokens | 336 billion input tokens per day just to describe tools | Rules out the full catalogue a second time, on cost. Any per-token price makes this the largest line item in the system before a single tool runs |
| A selected working set of 12 tools times 350 tokens | 4,200 tokens per turn, 10.1 billion per day | Rules out the objection that selection is not worth building. It is a 33x reduction in the dominant cost, from one subsystem |
| 3 percent of 2.4 million calls | 72,000 malformed calls per day, each costing a wasted turn plus a retry turn | Rules out returning a plain string error. At that volume the error message is a control signal consumed 72,000 times a day, so its wording is load-bearing |
| 20 percent of 2.4 million calls | 480,000 side-effecting calls per day | Rules out human approval on every write. At roughly 5.5 per second, a queue of people is not a design, so the effect class has to decide most cases without one |
| 12 sequential turns at a 2 second p99 tool call | 24 seconds of tool time in one session, before any model time | Rules out synchronous execution for anything slow, and makes a job handle mandatory for tools that cannot meet the 2 second budget |
| One runaway session calling one tool in a loop at 1 call per second for an hour | 3,600 calls from a single session | Rules out per-tenant-only budgets. A single session can exceed a day of normal usage, so the cap has to be enforced inside the run |

!!! tip "Say this out loud in the room"

    "Four hundred tools at three hundred fifty tokens each is a hundred forty thousand tokens of schema in every request, before the task. So the first design decision is not what a tool looks like, it is which tools the caller is allowed to see right now."

### V0, the simplest thing that works (tool surface)

**Predict first.** V0 sends every tool schema on every turn and executes calls immediately. Which failure appears first: cost, latency, or something the caller does?

??? note "Show answer"

    Cost, by a wide margin, and it appears on day one rather than under load: 140,000 tokens of schema per turn dominates everything else in the system. Accuracy degradation follows, because a caller choosing among 400 similarly described tools picks wrong more often. Latency is the least of the three, since the schemas add to the input, not to the work.

```
+-----------+     +---------------------+     +-----------------+
| Agent     | --> | Tool gateway        | --> | Internal        |
| runtime   | <-- | send all schemas,   | <-- | services        |
+-----------+     | validate, execute   |     | 400 tools       |
                  +---------------------+     +-----------------+
                            |
                            v
                  +---------------------+
                  | Tool registry       |
                  | name, schema, owner |
                  +---------------------+
```

**Caption.** V0 has four components. The agent runtime asks one gateway for the tool catalogue, receives every schema from a registry, and the gateway validates arguments and calls the internal service that implements the tool.

Why this is correct at a much smaller scale:

- With a dozen tools, sending all of them is 4,200 tokens, which is the same as the selected working set at 400 tools. Selection buys nothing until the catalogue is large.
- One gateway is the right shape and stays right through every later version: it is the only place where validation, authorisation, metering and logging can all happen once.
- Immediate execution is correct when every tool is fast and read-only. Everything after this point is a consequence of that stopping being true.

!!! warning "When not to use even this much"

    Below roughly 10 tools, all read-only, all owned by one team, do not build a registry or a gateway. Define the tools in the agent's own code, call the services directly, and keep the schema next to the function it describes. The measurement that justifies a gateway is the second team wanting to publish a tool, or the first side-effecting tool being added.

### Breaking point, then V1 (tool surface)

**Breaking point one: the schema budget, exceeded at roughly 60 registered tools.**

What fails: at 350 tokens per schema, 60 tools is 21,000 tokens of instructions on every request. Nothing errors. Cost rises linearly with the catalogue while usefulness does not, and selection accuracy falls as the number of similar-sounding tools grows: a caller choosing between eleven tools whose descriptions all begin "search for" will sometimes choose the wrong one, and that mistake costs a full turn plus a correction turn.

How you observe it: schema tokens as a fraction of total input tokens, per turn, and the rate of calls to a tool that the session never calls again after receiving its result (a proxy for a wrong first choice). The signature is that both curves bend upward with catalogue size rather than with traffic.

```
+-----------+  1 task  +---------------------+     +----------------+
| Agent     | -------> | Tool gateway        | --> | Tool selector  |
| runtime   | <------- | select, validate,   | <-- | working set    |
+-----------+  2 set   | authorise, execute  |     +----------------+
                       +---------------------+
                          |        |       |
                          v        v       v
              +-------------+ +---------+ +--------------+
              | Registry    | | Meter   | | Internal     |
              | schema lint | | tokens, | | services     |
              +-------------+ | calls   | +--------------+
                              +---------+
```

**Caption.** V1 has six components. The gateway asks a selector for a small working set of tool schemas relevant to the task, validates and authorises each call against the registry, records token and call usage in a meter, and forwards to the internal service that implements the tool.

V1 changes, each with the observation that justified it:

| Change | Justified by | Cost you accept |
|---|---|---|
| Tool selection instead of the full catalogue | 140,000 tokens per turn against 4,200 selected | The selector can omit the tool that was actually needed, which is a new and hard-to-debug failure mode |
| Namespaced tool names and a registration lint | 400 tools from many teams with no shared conventions | Publishing a tool is now slower, and the lint's rules become a contract with the teams who own tools |
| Structured errors instead of strings | 72,000 malformed calls per day, each consuming an error message | You have committed to an error vocabulary that the caller's behaviour now depends on, so changing wording changes behaviour |
| Per-call authorisation at the gateway | The agent acts for a user whose permissions are narrower than the gateway's | Every tool call now carries an identity, and the tool's own permission model has to agree with the gateway's |
| Metering on every call | An unbounded loop from one session | Metering is now on the request path, so its availability is your availability |

**Breaking point two: unsupervised side effects and runaway loops, at 480,000 writes and 3,600 calls per session.**

What fails: two things with one root, which is that nothing in V1 distinguishes a read from an irreversible write. A caller that misreads a result can send an email, cancel an order or delete a record, and the design has no place to say "not this one without asking". Separately, a session that receives a persistent error can retry the same call indefinitely: at one call per second, one session generates 3,600 calls in an hour, which exceeds a normal day's usage for a single user.

How you observe it: for effects, the count of state-changing calls with no corresponding human confirmation, which V1 cannot even measure because the effect is not declared. For loops, the distribution of calls per session, which is sharply bimodal: almost every session is under 30, and the pathological ones are in the thousands.

V2 changes:

- Every tool declares an effect class at registration: read-only, reversible write, or irreversible write. The class is part of the schema, it is required, and it drives policy rather than documentation.
- Irreversible writes require an approval token issued by a policy engine, which may be a human confirmation or a rule, but never a default.
- Every session carries a hard budget in calls and tokens. When it is 80 percent consumed the agent is told so in a result it can act on; at 100 percent further calls are refused with a distinct, terminal error.
- Tools that cannot meet the 2 second budget return a job handle instead of a result, with status, progress and cancellation as operations on that handle.

!!! warning "When not to build tool selection"

    Below roughly 25 tools, selection is over-engineering and actively harmful: it introduces a component that can hide the one tool the task needed, and a wrongly omitted tool is far harder to debug than a wrongly chosen one. 25 tools is under 9,000 tokens of schema. The measurement that justifies selection is schema tokens exceeding roughly 10 percent of your input budget, or a measurable rise in wrong-tool selection as the catalogue grows.

### Deep dive one: a schema read by a caller you cannot debug

This dive exists because the usual instinct, to treat the schema as a serialisation format, is wrong here. The schema is the entire documentation, the entire training signal and the entire error channel for a caller that will never read a guide.

**The description field is executable, not decorative.** In a conventional interface, a field description helps a developer who is also reading examples, tests and a changelog. Here it is the only thing the caller sees. Two rules follow:

| Rule | Why |
|---|---|
| Say when to call the tool, not only what it does | The caller's hard problem is selection among similar tools, not usage once selected |
| Say when not to call it, naming the neighbouring tool | Disambiguation between two similar tools is where wrong choices concentrate |

**Constrain the type system as hard as the domain allows.** Every degree of freedom you leave open is a place the caller can produce something plausible and wrong.

| Instead of | Use | What it prevents |
|---|---|---|
| A free string for a status | An enumeration of the legal values | Invented values that are grammatical and meaningless |
| A string date | A named date format, with the timezone rule stated | Ambiguous dates, which are silently wrong rather than rejected |
| A nested object three levels deep | A flat parameter list | Deep structures raise the malformed-argument rate with no expressive gain |
| One tool with a mode parameter | One tool per intent, named for the intent | A mode parameter hides two different effect classes behind one name |
| An unbounded list | A list with a stated maximum length | A single call that asks for more work than the tool can bound |

**The error message is a control signal consumed 72,000 times a day.** It is read by the caller and it determines what happens next, so it must contain three things: what was wrong, what the legal values are, and whether retrying could succeed. A message that says only "invalid request" produces a retry of the same invalid request, which is how one bad call becomes a loop.

| Error class | What the message must say | What the caller should do |
|---|---|---|
| Schema violation | Which field, and the legal values or format | Fix the argument and retry once |
| Semantic rejection | Why this valid input is not acceptable here | Change approach, do not retry |
| Not found | That the identifier did not resolve, and how to look one up | Call the lookup tool first |
| Transient failure | That the same call may succeed later, with a delay | Retry after the delay, with a bounded count |
| Terminal failure | That retrying will never succeed | Stop, and report to the user |

**Distinguishing retryable from terminal is the single highest-value bit in the whole error design.** Without it the caller cannot tell a temporary outage from a permanent refusal, and its only strategy is to try again. That is the mechanism behind breaking point two.

**Tool results are untrusted input to the next step.** If one tool returns text that another tool's arguments are built from, then anyone who can influence that text can influence the next call. Treat tool output as data, never as instructions: strip or escape anything that looks like a directive, and never let a tool's output alone authorise an effect. Saying this unprompted is the security signal on this prompt.

**Version by adding, never by changing meaning.** Adding an optional parameter is safe. Adding a value to an enumeration is safe. Changing what an existing parameter means, or narrowing what it accepts, is the change that produces confidently wrong calls, because the caller's behaviour is shaped by descriptions it may have been given a long time ago. JSON Schema, at [draft 2020-12](https://json-schema.org/draft/2020-12/release-notes) (accessed August 2026), is the vocabulary to express these shapes in rather than inventing one.

!!! warning "When not to add another tool"

    If a new capability differs from an existing tool only by a parameter value, do not register a second tool: two near-identical descriptions are the exact condition that raises wrong selection. Conversely, if two behaviours have different effect classes, they must be two tools even if the parameters are identical. The measurement that justifies splitting is a wrong-selection rate concentrated on one pair of tools.

### Deep dive two: declaring effects, gating them, and spending a budget you can enforce

The dives on this page vary by problem. The first covered how the caller reads the contract; this one covers what the contract must prevent, which is the part with no equivalent in any other case study here, because no other caller can be talked out of an action mid-run.

**The effect class is a required field, not a documentation convention.**

| Class | Definition | Default policy |
|---|---|---|
| Read-only | Cannot change any state a user could observe | Execute, meter, log |
| Reversible write | Changes state, and a defined operation restores it | Execute, log with an undo reference, notify |
| Irreversible write | Cannot be undone by any operation you expose | Require an approval token before execution |

**Requiring the field at registration is what makes the policy enforceable.** If effect class were optional, most tools would omit it, and the policy engine would have to guess. Requiring it moves the judgement to the person who wrote the tool, at the moment they know the most, and it makes the registry auditable: you can list every irreversible tool.

**Approval is a token, not a prompt.** The gateway does not ask a human; it demands an approval token that a policy engine issues. That engine may consult a human, or a standing rule, or a per-tenant setting. Separating them matters for two reasons: the gateway stays out of the user-experience business, and 5.5 irreversible calls per second is a rate no queue of people can serve, so most approvals must come from rules.

| Approval source | When it fits | The risk it carries |
|---|---|---|
| Human confirmation | High-value, low-frequency, user-visible actions | Approval fatigue: a person who confirms 40 times an hour stops reading |
| Standing rule with bounds | Repetitive actions inside a limit, for example refunds under a threshold | The bound is now the security boundary, so it needs its own review |
| Pre-authorised session scope | The user granted a class of action for this task up front | Scope creep, if the task changes after the grant |

**Cancellation is part of the approval design, not separate from it.** An irreversible call that has been approved but not yet executed must be cancellable; a call that is running must be cancellable if the tool supports it, and the contract must say plainly which tools do. Silence here means the user cannot stop something they have started.

**The budget must be enforced mid-run, and the caller must be told.** Two mechanisms, and both are needed:

| Mechanism | What it does | What it does not do |
|---|---|---|
| A hard cap enforced at the gateway | Refuses calls once the session's limit is reached | Nothing to help the agent finish gracefully, because it arrives as a wall |
| A remaining-budget signal in each result | Lets the agent prioritise and wrap up before the wall | Nothing to stop a caller that ignores it, which is why the hard cap exists too |

**Meter at the gateway, because it is the only place that sees everything.** Per call it should record the session, the tenant, the tool, the effect class, the token counts, the outcome and the duration. That single record answers cost attribution, abuse detection, wrong-selection analysis and incident review, and there is no second place where all six are available.

**The loop is the failure to design against explicitly.** A caller that receives the same terminal error repeatedly and retries is the pathology behind the 3,600 calls per session number. Three defences, in order of how much they help: mark errors terminal so the caller stops; cap consecutive calls to one tool with identical arguments; and cap the session. The first is the cheapest and the one most often missing.

!!! warning "When not to require approval"

    If a write is genuinely reversible and the reversal is exposed as a tool, do not gate it: approval on a reversible action buys little and spends the user's attention, which is the scarce resource that makes approval work at all. The measurement that justifies gating is an action whose reversal requires a support ticket. Gate on irreversibility, not on how important the action feels.

### Failure modes and evaluation (tool surface)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Retry storm | A caller receives a terminal error and repeats the identical call | Mark errors retryable or terminal explicitly, cap identical consecutive calls, and cap the session |
| Wrong tool selection | A near-duplicate description wins and a full turn is wasted | Disambiguating descriptions naming the neighbouring tool, plus a wrong-selection indicator per tool pair |
| Silent omission | The selector leaves out the one tool the task needed | Log the selected working set on every turn, so a failed task can be replayed against the full catalogue |
| Duplicate effect | A call is retried after a timeout and the write happens twice | A client-supplied key per call, with the gateway returning the original result on a repeat |
| Prompt injection through tool output | One tool returns text that shapes the next call's arguments | Treat all tool output as data, never let output alone authorise an effect, and re-check authorisation at execution time |
| Confused deputy | The gateway's own permissions are broader than the user's | Every call carries the user's identity, and authorisation is evaluated against that identity, not the gateway's |
| Approval fatigue | A person confirms so many actions that confirmation stops being a control | Gate only on irreversibility, and move repetitive approvals to bounded standing rules |
| Metering outage | The meter is unavailable and the gateway must decide whether to allow calls | Fail closed for irreversible writes and open for read-only, and state which, because the default decides your risk posture |

Service level indicators to publish:

- Schema tokens as a fraction of input tokens per turn, which is the direct measure of whether selection is still earning its place.
- Malformed argument rate, per tool. A tool far above the 3 percent baseline has a schema problem, not a caller problem.
- Terminal errors followed by an identical retry, as a fraction of terminal errors. This measures whether your error vocabulary is actually being used.
- Irreversible calls executed, split by approval source, which tells you how much of your risk is being decided by rules rather than people.
- Sessions exceeding 80 percent of budget, and sessions hitting the hard cap, reported separately because they mean different things.
- Tool call latency at p99 per tool, since one slow tool stalls every session that touches it.

Alert on identical-retry-after-terminal rate, because it is the leading indicator of the runaway loop and it moves before any budget alert does. Alert on irreversible calls executed without an approval token at any non-zero value, since that is a policy failure rather than a capacity one. Do not alert on malformed argument rate: it is inherent to a probabilistic caller and belongs on a per-tool dashboard where a schema owner will see it.

Re-checking against the non-functional requirements:

| Requirement | Does V2 meet it? |
|---|---|
| 400 registered tools | Yes, with selection. Without it the schema cost alone makes the system unviable regardless of anything else |
| Schema tokens per turn | Yes, 4,200 with a working set of 12, against 140,000 for the full catalogue |
| Tool p99 under 2 seconds | Yes for synchronous tools. Anything slower returns a job handle, which is a change to the tool's contract, not a performance fix |
| 3 percent malformed rate | Accepted rather than met: the design assumes it and spends its effort on making the resulting error useful |
| 480,000 side-effecting calls per day | Yes, with most decided by standing rules. A human-approval-only design fails this by roughly the ratio of 5.5 per second to what people can review |
| Hard session budget | Yes, enforced at the gateway with an advance signal, because a cap with no warning produces abandoned work rather than finished work |
| Cost attribution | Yes, from the per-call meter record, which is also what makes wrong-selection analysis possible |

### Where this answer is a simplification (tool surface)

- **Selection quality is a retrieval problem with its own evaluation.** Choosing 12 tools out of 400 is a ranking task, and it needs precision and recall measured against labelled tasks. Treating it as a lookup is the most common way this design fails quietly. See [ML System Design](ml-system-design.md) for how to evaluate a retrieval component.
- **Injection defence is deeper than escaping.** Content that reaches the caller from documents, web pages or other users can shape behaviour in ways no input filter catches reliably. The durable defences are architectural: least privilege per call, effect declarations, and never letting a model's output alone authorise an irreversible action.
- **Multi-agent surfaces multiply everything here.** When one agent calls another, the second agent's tool budget, identity and effect policy have to compose with the first's, and the naive composition grants the union of both.
- **Evaluation of the whole agent is a different discipline.** Per-tool correctness does not predict task success, and a change that improves one tool's schema can degrade a task that depended on the old ambiguity. See [Generative AI System Design](generative-ai-system-design.md) for the evaluation side.
- **Primary sources with dates.** [JSON Schema draft 2020-12](https://json-schema.org/draft/2020-12/release-notes) (accessed August 2026) is the schema vocabulary this design assumes. RFC 9457, [Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html) (July 2023, accessed August 2026), gives a standard machine-readable error body, which matters more here than usual because the error is consumed by the caller directly. RFC 9110, [HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) (June 2022, accessed August 2026), covers the status semantics for the job-handle operations.

### Level delta (tool surface)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Catalogue | Sends every tool schema | Derives 140,000 tokens per turn and adds selection | Names the new failure selection introduces (silent omission) and logs the working set so it is debuggable |
| Schema design | Writes a JSON schema per tool | Constrains types, uses enumerations, writes descriptions that say when to call | Adds when-not-to-call naming the neighbouring tool, because selection is the caller's hard problem |
| Errors | Returns a message string | Returns structured errors with a field and legal values | Separates retryable from terminal in the code, and connects that bit to the runaway loop it prevents |
| Effects | Documents which tools write | Declares an effect class per tool | Makes the class required at registration and drives policy from it, so the registry is auditable |
| Approval | Asks the user to confirm | Gates irreversible actions | Derives 5.5 per second, moves most approvals to bounded rules, and names approval fatigue as the failure |
| Budget | Adds a rate limit | Caps calls per session | Adds a remaining-budget signal so the agent can finish, and keeps the hard cap because signals can be ignored |

What each level added, in one line each:

- **Mid to senior:** the contract is designed for a caller that cannot read documentation, and every mechanism has a derived number behind it.
- **Senior to staff:** the design assumes its own caller will behave badly, so irreversibility, budgets and error semantics become enforced structure rather than guidance.

---
## Case study 10: Feature flag and configuration service

This is the only system on the page that sits inside every other system's request path. That inversion drives the whole design: a configuration service cannot be a dependency your callers wait on, because then every one of their outages includes yours. The interesting work is in getting the decision to happen without a network call, and in being honest about how stale the answer may be.

### The prompt (flags)

"Design the interface services use to read feature flags and configuration, including gradual rollouts and a kill switch."

### Questions that fork the design (flags)

Ask these because each answer moves a box or changes a resource. Do not recite them.

| Question worth asking | If the answer is A the design becomes | If the answer is B it becomes |
|---|---|---|
| Does evaluation happen in your service or in the caller? | Yours: one place to reason about, and a network call on every decision | Theirs: you ship rules and a library, evaluation is free, and every client version becomes a compatibility surface |
| How fast must a kill switch take effect? | Minutes: polling is enough and the design is simple | Seconds: you need a push channel plus a poll fallback, and the fallback is what actually has to work |
| Does a decision depend on who the user is? | No: a flag is a global boolean and rollout is a deploy | Yes: targeting rules, attributes and bucketing, and the same user must get the same answer everywhere |
| Must the same user get a stable answer over time? | No: any consistent-within-request answer is fine | Yes: bucketing must be a deterministic function of a stable identifier, not of anything server-side |
| What happens when the service is unreachable? | Fail to a default in code: simple, and the default is now a second source of truth | Serve the last known values from local storage: correct, and now staleness is a contract term |
| Are values booleans, or arbitrary configuration? | Booleans: small payloads and a trivial schema | Arbitrary values: types, validation, and a bad value can break every caller at once |
| Who can change a flag, and with what review? | Anyone, immediately: fast, and one typo is a company-wide incident | Reviewed and staged: safer, and now the tool has environments, approvals and an audit trail |
| Do you need to know which flags are still used? | No: flags accumulate forever and nobody dares delete one | Yes: evaluation telemetry flows back, which is a second high-volume pipeline |

### Requirements (flags)

Functional:

- Read the value of a flag for a given evaluation context, from any service instance.
- Define targeting rules: percentage rollouts, attribute matches and explicit overrides.
- Turn a flag off everywhere, quickly, as a kill switch.
- Serve a defined value when the service is unreachable, including at process start.
- Record who changed what and when, and support reverting a change.
- Report which flags are actually being evaluated, so dead flags can be removed.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Service instances | 20,000 | ASSUMED, a few hundred services at moderate replica counts; ask, because it sets distribution load |
| Flags and configuration keys | 5,000 | ASSUMED, and it grows monotonically unless someone measures usage, which is why the last requirement exists |
| Application request rate, aggregate | 100,000 per second | ASSUMED, the traffic that flag evaluation rides on top of |
| Flag evaluations per application request | 10 | ASSUMED, a request path that crosses several services, each checking a couple of flags |
| Kill switch propagation | under 30 seconds to 99 percent of instances | ASSUMED, fast enough to stop an incident, slow enough not to require a push channel you must trust |
| Availability of a decision | 100 percent, including when the service is down | GIVEN, because a configuration service inside the request path must never be able to take callers down |
| Average rule payload per flag | 500 bytes | ASSUMED, a name, a type, a default and a handful of targeting rules |
| Evaluations per second | 1,000,000 | DERIVED below |
| Full payload size | 2.5 MB | DERIVED below |
| Naive polling bandwidth | 1.67 GB per second | DERIVED below |

### Estimation that eliminates (flags)

Every line ends with the option it kills. A number that kills nothing is deleted from the answer.

| Calculation | Result | What it rules out |
|---|---|---|
| 100,000 requests per second times 10 evaluations | 1,000,000 evaluations per second | Rules out a remote call per evaluation, on volume alone. That is ten times the traffic of the system it is supporting, to answer questions whose answers rarely change |
| A 1 millisecond remote evaluation added to a request that makes 10 of them | 10 milliseconds added to every application request, serially | Rules out remote evaluation a second time, on latency, and it rules it out even if the volume were affordable |
| 5,000 flags times 500 bytes | 2.5 MB of rules, the entire decision surface | Rules out the argument that rules are too large to ship to clients. 2.5 MB is a small file, and this is the number that makes local evaluation possible |
| 20,000 instances polling every 30 seconds | 667 requests per second, which is trivial | Rules out any concern about request rate on the distribution path. The request rate is not the problem |
| 667 requests per second times 2.5 MB | 1.67 GB per second, 144 TB per day | Rules out sending the full payload on every poll. The bandwidth, not the request rate, is what breaks naive polling |
| The same poll rate with conditional requests, 99 percent unchanged | 6.7 full payloads per second, about 17 MB per second | Rules out the objection that polling cannot scale. A conditional request turns the previous row into a rounding error |
| 20,000 instances restarting together after a deploy | 20,000 cold fetches of 2.5 MB, 50 GB in a burst | Rules out cold start depending on the service being reachable, and makes a bundled bootstrap file a launch requirement rather than a refinement |
| A 30 second poll interval as the only propagation mechanism | Worst-case kill switch latency of 30 seconds, meeting the requirement exactly with no margin | Rules out a longer interval, and shows that a push channel is an optimisation on top of a poll that already meets the requirement, not a dependency |

!!! tip "Say this out loud in the room"

    "A million evaluations a second is ten times the traffic of the system I am supporting, and the entire rule set is two and a half megabytes. Those two numbers together say the decision has to happen in the caller's process, so my real design problem is distributing a small file quickly and saying honestly how stale it might be."

### V0, the simplest thing that works (flags)

**Predict first.** V0 answers one evaluation per remote call. Which of its two problems, latency or availability, would stop you shipping it even if the volume were free?

??? note "Show answer"

    Availability. Ten milliseconds per request is painful but survivable. A configuration service in the synchronous path means every caller's availability is multiplied by yours: if you are 99.9 percent available, every service that reads a flag is capped below that, and your bad deploy becomes everyone's outage. Latency is a cost; this is a structural defect.

```
+-----------+     +---------------------+     +-----------------+
| Service   | --> | Flag API            | --> | Postgres        |
| instance  | <-- | evaluate one flag   | <-- | flags, rules,   |
+-----------+     | per call            |     | audit           |
                  +---------------------+     +-----------------+
                            |
                            |
                  +---------------------+
                  | Admin console       |
                  | edit, review, audit |
                  +---------------------+
```

**Caption.** V0 has four components. A service instance calls a flag service for each evaluation, which reads rules from Postgres and returns a value. An admin console edits the same rules and writes an audit record.

Why this is correct at a much smaller scale:

- With one service at 100 requests per second and two flags, this is 200 evaluations per second and a few milliseconds of added latency, which nobody will notice.
- One evaluator means one implementation of the rules. Every consistency bug in the rest of this case study comes from having more than one.
- The admin console and the audit trail are correct from the start and never change. It is the read path, not the write path, that gets rebuilt.

!!! warning "When not to use even this much"

    Below roughly 20 flags in a single service, a configuration file deployed with the application is correct, and a flag service is over-engineering: you get atomicity, review and rollback from the deploy pipeline you already run. The measurement that justifies a service is needing to change a value without a deploy, usually first felt during an incident, or a second service needing the same value.

### Breaking point, then V1 (flags)

**Breaking point one: the synchronous dependency, which fails on the first flag-service outage regardless of load.**

What fails: every caller that evaluates a flag on its request path now waits on you. A slow response, not even an error, is enough: a 200 millisecond p99 on the flag service becomes a 2 second addition to a caller making ten evaluations, and their circuit breakers begin opening on their own dependencies. The volume number, 1,000,000 evaluations per second, is the second problem and the more visible one, but the dependency is the one that ends the design.

How you observe it: the correlation between your latency and your callers' error rates. The signature is that a caller's incident timeline starts with a rise in flag evaluation latency several minutes before their own alerts fire, which is the clearest possible statement that you are on their critical path.

```
+-----------+     +----------------------+     +-----------------+
| Service   | --> | Local evaluator      |     | Postgres        |
| instance  | <-- | rules in memory      |     | flags, rules,   |
+-----------+     +----------------------+     | audit           |
                        |          |           +-----------------+
                        | poll     | usage              |
                        |          v                    |
                  +----------------------+     +-----------------+
                  | Rule distribution    | <-- | Admin console   |
                  | versioned snapshots  |     | edit, review    |
                  +----------------------+     +-----------------+
                                |
                                v
                        +----------------+
                        | Usage pipeline |
                        | which flags    |
                        +----------------+
```

**Caption.** V1 has six components. Each service instance holds the rule set in memory and evaluates locally. A distribution service publishes versioned snapshots that instances poll for, and a usage pipeline collects which flags each instance actually evaluated.

V1 changes, each with the observation that justified it:

| Change | Justified by | Cost you accept |
|---|---|---|
| Evaluation moves into the caller's process | 1,000,000 evaluations per second and a synchronous dependency | The evaluation logic now ships as a library, so every client version is a compatibility surface you cannot upgrade on demand |
| Rules distributed as versioned snapshots | 2.5 MB is small enough to ship whole | Instances now disagree during propagation, which is a new and permanent property of the system |
| Polling with a conditional request | 1.67 GB per second of unconditional polling | Freshness is now bounded by the poll interval, and that bound belongs in the contract |
| Usage telemetry flowing back | 5,000 flags that nobody dares delete | A second high-volume pipeline, whose own failure must not affect evaluation |
| Bundled bootstrap snapshot in the client | 20,000 cold starts requesting 50 GB after a deploy | The bundled snapshot ages with the build, so a long-lived build starts from very stale values |

**Breaking point two: instances disagree, and one user sees both answers, at any propagation window.**

What fails: during the 30 second propagation window, some instances hold snapshot 41 and some hold 42. A user whose requests are load balanced across them sees the feature enabled, then disabled, then enabled. For a visual change this is annoying; for a change that alters what is written to a database, it is a correctness bug. The window is not eliminable, so it must be made survivable.

How you observe it: publish the snapshot version each instance holds, and alert on the spread between the oldest and newest across the fleet. The signature of a stuck instance is a version that stops advancing while the fleet moves on, which no aggregate latency metric will show you.

V2 changes:

- Bucketing for percentage rollouts is a deterministic hash of the flag key and a stable user identifier, computed identically in every client. The same user therefore gets the same answer from every instance holding the same snapshot, so the only disagreement left is version skew, not randomness.
- A change to a flag is a new immutable snapshot version, and a snapshot is applied atomically. An instance never holds half of one change, which is what turns a multi-flag edit into a single decision.
- A push channel notifies instances that a new version exists, cutting typical propagation to a second or two. The poll continues underneath it at the same interval, because a push channel is an optimisation and the poll is the guarantee.
- Every evaluation returns the snapshot version alongside the value, so a caller can log which decision came from which rule set. This is what makes an incident reviewable.

!!! warning "When not to add a push channel"

    If the poll interval already meets the propagation requirement, a push channel adds a long-lived connection per instance, a fan-out problem at 20,000 connections, and a second code path that will be less tested than the first. The measurement that justifies push is a propagation requirement tighter than the shortest poll interval your bandwidth allows, which at 17 MB per second of conditional polling is a long way off.

### Deep dive one: where the decision happens decides everything else

This dive comes first because it is the choice made in minute two, and because every remaining question in this case study is downstream of it. Candidates who pick without articulating the trade-off cannot answer the staleness questions that follow.

| Property | Evaluated remotely | Evaluated locally |
|---|---|---|
| Latency per decision | A network round trip, times the number of flags | Microseconds, in process |
| Availability impact on callers | Your availability caps theirs | None: a caller with a snapshot keeps deciding through your outage |
| Freshness | Immediate | Bounded by propagation, and the bound must be published |
| Rule changes | Take effect instantly, everywhere, atomically | Take effect per instance, over a window |
| Logic changes | Ship when you deploy | Ship when every client upgrades, which may be never |
| User attributes | Must be sent to you, including attributes you should not hold | Stay in the caller's process, which is a real privacy advantage |
| Consistency across instances | Perfect | Eventual, and that is a contract term |

**The design I would defend: local evaluation, with the trade-off stated plainly.** The two numbers decide it: 1,000,000 evaluations per second, and 2.5 MB of rules. Local evaluation trades exact freshness for availability and latency, and freshness is the cheaper thing to give up because it can be bounded and published.

**The cost you have accepted is a versioned evaluation library you cannot force anyone to upgrade.** This is the same problem as the brownfield case study, arriving early and by choice. Three mitigations, all decided in version one:

| Mitigation | What it does |
|---|---|
| Rules are declarative data, never code | A new rule type is data the old library can be taught to skip, rather than logic it lacks |
| Unknown rule types are skipped with a defined outcome | An old client meeting a new rule falls through to the flag's default, predictably, rather than failing |
| The snapshot declares a minimum library version | You can detect, per instance, who cannot evaluate the current rules, which is the only way to run a migration |

**Local evaluation changes what "turn it off" means.** A kill switch is no longer an instruction; it is a new snapshot that has to reach 20,000 processes. That reframing is the point: your job during an incident is distribution, not decision, and the indicator you need is the fleet's version spread, not your own error rate.

**The hybrid exists and is usually a mistake.** Evaluating locally but calling out for a few "important" flags reintroduces the synchronous dependency for exactly the flags that matter most during an incident, which is when you are least able to serve it. If a decision genuinely cannot tolerate staleness, it is not a feature flag; it is a query against the system of record.

!!! warning "When not to evaluate locally"

    If evaluation depends on data the caller does not have, for example a fraud score or an entitlement held in another system, local evaluation cannot work and a remote decision service is correct. In that case put it behind a short cache and a fail-open default, and accept that it is on the request path. The measurement that separates the two is whether the evaluation context is fully available in the calling process.

### Deep dive two: staleness as a published number, and what an offline client may do

The dives on this page vary by problem. The first dive established that a local answer may be old; this one is about turning that from an embarrassment into a contract term, which is the part of the design that no other case study on this page needs.

**Say what stale means, in three numbers, and put them in the documentation.**

| Number | Value here | How it is derived |
|---|---|---|
| Propagation target | 30 seconds to 99 percent of instances | The poll interval, which meets the stated requirement with no push channel involved |
| Maximum staleness while healthy | one poll interval plus one publish latency | The worst case is an instance that polled just before a change was published |
| Maximum staleness while unhealthy | unbounded, and the client reports its age | An instance cut off from distribution keeps working with the last snapshot, which is the point |

**Unbounded staleness during an outage is the correct behaviour, and it must be visible.** A client that discards its snapshot when it cannot refresh trades a stale answer for no answer, which is strictly worse: it converts your outage into everyone's. So the client keeps deciding, and it exposes the age of what it is deciding from, so the humans can see it.

**The layered fallback, in the order the client should try it.**

| Layer | Where it comes from | When it is used |
|---|---|---|
| In-memory snapshot | The last successful refresh | Normal operation |
| Local persisted snapshot | Written to disk on each refresh | Process restart while distribution is unreachable |
| Bundled bootstrap snapshot | Baked into the build artefact | First start of a fresh instance with no disk state |
| Per-call default | Supplied by the caller at the evaluation site | No snapshot at all, or a flag absent from the snapshot |

**The per-call default is the most important line of the whole client contract.** Every evaluation supplies the value to use if the flag is unknown. That single rule removes the entire class of failures where a caller crashes because a flag was renamed, deleted or never reached them, and it means a brand-new instance with no data at all still runs.

**Choose the default so that failure is safe, not so that it is convenient.** The default should be the state the system was in before the flag existed. For a kill switch this inverts the usual instinct: the flag should mean "enable the risky thing", defaulting to off, so that an instance which cannot reach you is already in the safe state. A kill switch that defaults to "not killed" fails open at exactly the wrong moment.

**Conditional requests are what make polling affordable, and they are already standard.** The client sends the version it holds; an unchanged snapshot returns a small unchanged response instead of 2.5 MB. That single mechanism turns 1.67 GB per second into roughly 17 MB per second.

RFC 9110, [HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) (June 2022, accessed August 2026), defines entity tags and the 304 response this rests on, and RFC 9111, [HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111.html) (June 2022, accessed August 2026), defines the freshness model. There is no reason to invent a private protocol for this.

**Jitter the poll, or you have built a synchronised herd.** 20,000 instances that start together and poll on a fixed interval will poll together forever. A randomised interval spreads them, and it matters most at the moment you can least afford it, which is a fleet-wide restart.

**Report the age, not just the version.** A dashboard of snapshot versions tells you the spread; a distribution of snapshot age in seconds tells you whether anyone is dangerously stale. The second is the one to alert on, because it is expressed in the unit the contract is written in.

!!! warning "When not to persist a snapshot locally"

    If instances are extremely short lived, for example function invocations lasting under a second, local persistence buys nothing and the bundled bootstrap plus per-call defaults are the whole story. Persisting also creates a stale file that can outlive a flag's deletion. The measurement that justifies persistence is instances that restart more often than they can afford a cold fetch, which at 2.5 MB is a low bar but not zero.

### Failure modes and evaluation (flags)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Thundering herd | 20,000 instances restart together and all fetch the full snapshot | Jittered poll intervals, a bundled bootstrap so cold start needs no fetch, and conditional requests so most fetches are empty |
| Split brain | Half the fleet holds the old snapshot and half the new, and one user sees both | Deterministic bucketing so the only variable is version, atomic snapshot application, and an alert on fleet version spread |
| Stale data | An instance is cut off and decides from a week-old snapshot | Publish snapshot age per instance and alert on the maximum; keep deciding rather than failing, because the alternative is worse |
| Configuration error | A bad value is published and every caller adopts it within 30 seconds | Staged rollout of the change itself, schema validation on publish, and one-click revert to the previous snapshot version |
| Cache stampede | The push channel notifies 20,000 instances at once and all fetch immediately | Random delay before fetching on notification, sized to spread the fleet across several seconds |
| Metastable failure | Distribution is overloaded, clients retry faster, load rises further | Fixed poll interval with jitter and no adaptive speed-up on failure; a failing client must slow down, never accelerate |
| Version skew | An old client library cannot evaluate a new rule type | Declarative rules, defined skip behaviour, and a minimum library version declared in the snapshot so you can find the stragglers |
| Silent unbounded growth | Flags accumulate because nobody knows which are used | Usage telemetry per flag, and a stale-flag report that names an owner |

Service level indicators to publish:

- Snapshot age at the 99th percentile across the fleet, in seconds, which is the indicator written in the same unit as the contract.
- Fleet version spread: the number of distinct snapshot versions in use, which finds a stuck instance that no average will reveal.
- Time from publish to 99 percent of instances holding the new version, measured end to end rather than from your own logs.
- Fraction of polls returning unchanged, which should be near 99 percent. A drop means something is republishing needlessly and your bandwidth is about to follow.
- Evaluations returning the per-call default because the flag was absent, per flag. A rise means a rename or deletion is reaching clients that did not expect it.
- Client library version distribution, since it bounds which rule types you can safely publish.

Alert on snapshot age at the 99th percentile, because it is the direct measure of the promise you made and it catches a distribution failure before anyone reports a stale decision. Alert on fleet version spread persisting beyond the propagation window. Do not alert on your own service's error rate as the primary signal: clients are designed to survive it, and the thing that actually matters is whether they are current.

Re-checking against the non-functional requirements:

| Requirement | Does V2 meet it? |
|---|---|
| 1,000,000 evaluations per second | Yes, and none of them cross a network. This requirement is met by the architecture, not by capacity |
| A decision always available | Yes, through the layered fallback ending in a per-call default. The default is what makes the guarantee absolute rather than probable |
| Kill switch within 30 seconds | Yes by polling alone, and typically within seconds with the push channel. The poll is the guarantee; push is the improvement |
| 20,000 instances | Yes, at 667 conditional requests per second and roughly 17 MB per second of bandwidth |
| 5,000 flags at 2.5 MB | Yes, shipped whole. If the payload grew past roughly 50 MB this design would need per-service scoping instead |
| Audit and revert | Yes, since every change is an immutable snapshot version, which makes revert a publish rather than an edit |
| Knowing which flags are used | Yes, from the usage pipeline, and it is the only requirement here that a purely technical design would have omitted |

### Where this answer is a simplification (flags)

- **Experimentation is a different product with the same plumbing.** An experiment needs randomised assignment, exposure logging, statistical analysis and a stopping rule. Serving a variant is the easy part; deciding what the variant did is the hard part. See [ML System Design](ml-system-design.md) for the analysis side.
- **Client-side flags for browsers and mobile devices break several assumptions here.** You cannot ship 2.5 MB of rules to a phone, the rules themselves may be commercially sensitive, and the device may be offline for days. That is a remote evaluation design with a signed, scoped payload, and it is genuinely different. See [Mobile System Design](mobile-system-design.md).
- **Configuration is not the same thing as feature flags, and merging them is a common regret.** A flag is short lived, boolean and owned by a team; a configuration value may be long lived, typed and load bearing. Sharing one store is convenient and makes it easy to change a database connection string with the tooling built for toggling a button.
- **Flag lifecycle is an organisational problem wearing a technical costume.** Usage telemetry tells you a flag is dead; it does not remove the branch from the code. Without an owner and a removal process, the flag count only grows.
- **Primary sources with dates.** RFC 9110, [HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) (June 2022, accessed August 2026), defines the entity tags and conditional requests that make polling affordable. RFC 9111, [HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111.html) (June 2022, accessed August 2026), defines the freshness model this design's staleness contract borrows its vocabulary from. For the distribution and consensus building blocks under snapshot publication, see [Modern System Design](modern-system-design.md).

### Level delta (flags)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Evaluation site | Calls a flag service per decision | Moves evaluation into the caller after deriving 1,000,000 evaluations per second | Names the cost of that move, an unupgradable client library, and designs declarative rules with defined skip behaviour |
| Availability | Adds retries and a cache | Adds a local snapshot and a fallback chain | Requires a per-call default at every evaluation site, so a client with no data at all still runs |
| Distribution | Polls for updates | Uses conditional requests after deriving 1.67 GB per second | Jitters the poll and keeps it as the guarantee, treating the push channel as an optimisation that may fail |
| Consistency | Says updates are eventual | Names the window and makes snapshots atomic | Makes bucketing deterministic so version skew is the only variable, and alerts on fleet version spread |
| Defaults | Picks a sensible default | Documents the default per flag | Orients defaults so that unreachability lands in the safe state, which inverts the naive kill switch |
| Lifecycle | Adds an audit log | Adds usage telemetry | Treats revert as a publish of a previous immutable version, so rollback is the same operation as any change |

What each level added, in one line each:

- **Mid to senior:** the service is removed from its callers' critical path on a derived number, and distribution cost is engineered rather than assumed.
- **Senior to staff:** the residual uncertainty (skew, staleness, unreachability) is turned into published numbers, safe defaults and indicators someone can act on.

---
## Case study 11: A breaking change on a public interface

Every other case study on this page starts with a blank page. This one starts with twelve thousand integrations you did not write, cannot test, and cannot upgrade, and a change that has to ship anyway. It is the closest of the eleven to what senior engineers actually spend their time on, and the one where the right answer is least likely to be a diagram.

### The prompt (brownfield)

"Our public interface returns an order total as an integer number of cents. We are launching in markets whose currencies do not have two decimal places. Ship the fix without breaking the integrations we cannot upgrade."

### Questions that fork the design (brownfield)

Ask these because each answer moves a box or changes a resource. Do not recite them.

| Question worth asking | If the answer is A the design becomes | If the answer is B it becomes |
|---|---|---|
| Is the change additive, or does it change an existing field's meaning? | Additive: ship a new field, leave the old one, and most of this case study is unnecessary | Meaning changes: old clients will read the old field and get a number that is confidently wrong |
| Can an old client be given a correct value in the old shape? | Yes for some inputs: you can scope the incompatibility to the cases that cannot be represented | No, ever: the old shape must start failing loudly, and the question becomes when and how |
| Do you know which integrations read the affected field? | Yes, from telemetry: you can target, measure and prove | No: every decision below is a guess, so building the measurement comes before building the fix |
| Is there a contract that lets you retire an old shape? | Yes, with a stated notice period: the timeline is a calendar problem | No: the timeline is a negotiation, and you may be maintaining the old shape indefinitely |
| Can a client tell you which shape it wants? | Yes, via a version selector: two shapes coexist and each client chooses | No: you must infer it, and inference is wrong for exactly the clients who are hardest to contact |
| How is traffic distributed across integrations? | Evenly: outreach scales with the number of integrations, which is bad | Concentrated: a small number of integrations carry most traffic, so outreach and risk have different shapes |
| Does the old shape have to keep receiving new features? | Yes: you now maintain two evolving interfaces forever | No: it is frozen at the moment of the split, which is the only version of this that ends |
| Who pays when an unupgraded client breaks? | You, in support and reputation | The client, contractually: which changes the negotiation but not the engineering |

### Requirements (brownfield)

Functional:

- Represent monetary amounts correctly for currencies with zero, two or three decimal places.
- Keep existing integrations working, unchanged, for the full notice period.
- Let a client declare which shape of the interface it wants, explicitly.
- Signal deprecation and planned retirement in a way a machine can read.
- Report, per integration, whether it is still using the old shape.
- Retire the old shape on a stated date, with a defined failure mode for anyone still on it.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Active integrations | 12,000 | ASSUMED, a widely used public interface; ask, because the tail is what makes this hard |
| Request rate | 8,000 per second | ASSUMED, and largely irrelevant to the difficulty, which is a useful thing to notice out loud |
| Share of traffic from the top 100 integrations | 80 percent | ASSUMED, a power law; integration traffic is almost never uniform |
| Notice period before retirement | 12 months | GIVEN, from the published interface policy, which is the only lever that makes retirement possible at all |
| Fields on the affected resource | 30 | ASSUMED, an order with line items, addresses, totals and status |
| Share of orders in non-two-decimal currencies at launch | 4 percent | ASSUMED, initially small and growing, which is why this can be shipped incrementally |
| Expected upgrade rate at 12 months | 97 percent of integrations | ASSUMED, and the missing 3 percent is the entire problem |
| Requests per day | 691 million | DERIVED below |
| Field observations per day if logged naively | 20.7 billion | DERIVED below |
| Integrations still on the old shape at retirement | 360 | DERIVED below |

### Estimation that eliminates (brownfield)

Every line ends with the option it kills. A number that kills nothing is deleted from the answer.

| Calculation | Result | What it rules out |
|---|---|---|
| 8,000 requests per second times 86,400 seconds | 691 million requests per day | Rules out nothing about capacity, and that is the point: this problem is not about load, so no part of the answer should be about scaling |
| 691 million requests times 30 fields | 20.7 billion field observations per day | Rules out logging every field access to answer "who uses this field". The measurement you need cannot be built the obvious way |
| 12,000 integrations times 30 fields, aggregated once per day | 360,000 rows per day | Rules out the objection that field-level usage cannot be measured. Aggregating per integration per day instead of per request is five orders of magnitude cheaper |
| 12,000 integrations minus the top 100 | 11,900 integrations carrying 20 percent of traffic | Rules out a rollout plan based on traffic share alone. Most of your risk by count is in traffic you can barely see |
| 11,900 integrations at 5 minutes of individual outreach each | 992 hours, roughly six months of one person's full time | Rules out manual outreach to the tail, and makes machine-readable deprecation signalling the primary channel rather than a courtesy |
| 3 percent of 12,000 at the end of the notice period | 360 integrations still on the old shape on retirement day | Rules out a clean cutover. Someone will break, so the design question is who, how loudly, and with how much warning |
| 4 percent of orders in currencies the old field cannot represent | 27.6 million requests per day where the old shape is not just outdated but wrong | Rules out leaving the old field populated for those orders. A wrong number is worse than an absent one |
| Two wire shapes over one domain model, across 30 fields and every endpoint | One translation layer, one test matrix doubled, and one date on which half of it is deleted | Rules out branching the service. Two deployed copies of the whole system diverge, and the divergence never converges |

!!! tip "Say this out loud in the room"

    "Eight thousand requests a second is not the problem here. Twelve thousand integrations where the tail of eleven thousand nine hundred carries only a fifth of the traffic is the problem, because outreach scales with count and risk scales with traffic, and those two point in opposite directions."

### V0, the simplest thing that works (brownfield)

**Predict first.** V0 adds a new correctly typed amount field and leaves the old integer field exactly as it is. Name the specific order for which V0 is wrong.

??? note "Show answer"

    An order in a currency with three decimal places, or with none. The old field is an integer number of hundredths. For a three-decimal currency it either loses a digit or is off by a factor of ten, and for a zero-decimal currency it is inflated by a hundred. An old client reads a number that is well formed, plausible and wrong, which is the worst possible outcome and the reason additive-only is not sufficient here.

```
+-----------+     +---------------------+     +-----------------+
| Client    | --> | Order API           | --> | Order service   |
| any age   | <-- | returns both        | <-- | one domain      |
+-----------+     | total_cents and     |     | model, amounts  |
                  | amount object       |     | with currency   |
                  +---------------------+     +-----------------+
```

**Caption.** V0 has three components. One order service holds amounts with an explicit currency and precision. The interface returns both the new amount object and the legacy integer field, so existing clients are untouched.

Why this is correct for most of the traffic:

- For the 96 percent of orders in two-decimal currencies, the legacy field is exactly right, and no client anywhere needs to change. Additive change is free, and free change is the default you should reach for.
- One domain model, two representations, is the shape the whole solution keeps. Everything after this adds control over the second representation rather than replacing the idea.
- It ships in days rather than quarters, which matters: the market launch is not waiting for a migration programme.

!!! warning "When not to do more than this"

    If every value your interface returns can be represented correctly in the old shape, stop here. Adding a version selector, a translation layer and a retirement timeline for a change that breaks nobody is a large cost for no benefit, and it trains clients to expect versioning where none is needed. The measurement that justifies going further is a single case where the old shape must carry a wrong or absent value.

### Breaking point, then V1 (brownfield)

**Breaking point one: the legacy field cannot represent 4 percent of orders, which is 27.6 million requests a day.**

What fails: for a zero-decimal or three-decimal currency, there is no integer number of hundredths that is correct. You have three options and all of them are bad: return a rounded value (silently wrong, and the client charges the wrong amount), return zero (obviously wrong, and clients will treat it as a free order), or omit the field (a client that assumes it is present may crash). The one thing you must not do is the first, because it is the only one where nobody finds out.

How you observe it: you will not, from your own telemetry, because you are returning a well-formed response. It surfaces as a client-reported discrepancy days later. The instrument you need is a synthetic check that requests an order in each supported currency through the old shape and asserts the legacy field is either exactly correct or explicitly absent.

```
+-----------+ shape=  +----------------------+     +----------------+
| Client    | ------> | Edge translation     | --> | Order service  |
| declares  | <------ | legacy <-> current   | <-- | one domain     |
| shape     |         +----------------------+     | model          |
+-----------+                |         |           +----------------+
                             v         |
                  +----------------+  +------------------+
                  | Usage rollup   |  | Deprecation      |
                  | per key, per   |  | signalling       |
                  | field, per day |  | headers, docs    |
                  +----------------+  +------------------+
```

**Caption.** V1 has five components. Clients declare which shape they want. An edge translation layer converts between the legacy wire shape and the current one over a single domain model. A usage rollup records which fields each API key received, and a deprecation signalling component adds machine-readable retirement information to responses.

V1 changes, each with the observation that justified it:

| Change | Justified by | Cost you accept |
|---|---|---|
| Clients declare a shape explicitly | Two shapes cannot both be the default | Clients that declare nothing must be assigned a shape by inference, and inference is wrong for exactly the quiet ones |
| Translation at the edge, one domain model behind | 30 fields and every endpoint | Every new feature must be considered against both shapes until the legacy one is frozen |
| Legacy field omitted, not rounded, for currencies it cannot represent | 27.6 million requests a day where any value would be wrong | Some old clients will break on the missing field, which is exactly the intent: it is a loud failure instead of a silent one |
| Per-key, per-field usage rollup | 11,900 tail integrations you cannot phone | A daily aggregation pipeline, and a privacy review, since usage data is data about your customers |
| Machine-readable deprecation and sunset signalling | 992 hours of manual outreach that will not happen | You have now published a date, and publishing a date you then miss costs more credibility than not publishing one |

**Breaking point two: the tail does not upgrade, leaving 360 integrations on retirement day.**

What fails: nothing technical. Notices are sent, the top 100 integrations upgrade within weeks because they have engineers watching, and the tail does not respond at all. Many are abandoned scripts whose authors have left. At 97 percent adoption you still have 360 live integrations, carrying perhaps 2 percent of traffic and 100 percent of your inability to delete the legacy code path. A cutover on the stated date breaks all of them at once, on the same morning.

How you observe it: the count of distinct API keys that received a legacy-shape response in the last 7 days, plotted weekly against the retirement date. The curve is the whole story: it falls fast, then flattens, and the flat part is what you are actually planning for.

V2 changes:

- Scheduled brownouts: short, announced windows before the retirement date during which the legacy shape returns an error. An integration that is genuinely dead stays silent; one that is alive but unattended produces an alert on someone's side, which is the only way to reach an owner you cannot email.
- The legacy shape is frozen at the split: it receives no new fields, no new endpoints and no new capabilities. This is what turns an unbounded maintenance cost into a bounded one, and it is a stronger migration incentive than any email.
- Retirement is graduated rather than instantaneous: the legacy shape starts returning errors for a rising percentage of requests over the final weeks, so a client that missed everything experiences intermittent failure before total failure.
- A named extension process exists, with a deadline and an owner, for the small number of integrations with a genuine reason. Pretending there will be no exceptions guarantees the exceptions are handled badly.

!!! warning "When not to run brownouts"

    If clients are internal and reachable, a brownout is a hostile way to send a message you could have sent directly. Brownouts exist because you cannot reach the owner, and they cost real user-visible failures. The measurement that justifies one is a set of integrations still active after direct outreach has been attempted and has produced no response.

### Deep dive one: proving that nobody uses it

This dive exists because every decision in the timeline rests on a claim about usage, and most teams cannot support that claim with anything better than a guess. The measurement is cheap once you stop trying to build it per request.

**The naive design is five orders of magnitude too expensive, and the fix is aggregation.**

| Approach | Volume | Verdict |
|---|---|---|
| Log every field of every response | 20.7 billion records per day | Unusable, and it would cost more than the system it measures |
| Sample 1 percent of requests | 207 million records per day | Still large, and worse: sampling misses the rare client, which is precisely the client you need to find |
| Aggregate per key, per field, per day, in the process | 360,000 rows per day | Correct. Every integration is represented, and the volume is trivial |

**Sampling is the trap.** The clients that matter here are the ones that call rarely: a nightly reconciliation job that runs once a day is invisible at a 1 percent sample and will still break on retirement day. Aggregation keeps every key while discarding only the per-request detail you do not need.

**"Received the field" and "depends on the field" are different claims.** Your rollup records what you sent, not what the client parsed. That gap matters in both directions:

| Situation | What the rollup says | The truth |
|---|---|---|
| Client requests the resource and ignores the field | Used | Not depended on. You will over-count, which is the safe direction |
| Client requests the resource only on a quarterly cycle | Unused for 89 days | Depended on. This is why the observation window must exceed the longest plausible call interval |
| Client parses the field strictly and fails on absence | Used | Depended on, and breakage is guaranteed rather than likely |

**Set the observation window from the calling pattern, not from convenience.** A 7 day window misses monthly jobs. A 90 day window catches quarterly ones and is long enough that "no usage in 90 days" is a defensible statement. Publishing the window alongside the claim is what makes the claim auditable.

**Field-level requests are more precise than response-level ones.** If your interface supports sparse field selection, a client that explicitly asks for a field has declared a dependency, which is far stronger evidence than having been sent it by default. That is a real argument for sparse selection in a public interface: it makes deprecation measurable.

**Segment the report by traffic and by count, because they drive different actions.**

| Segment | What it drives |
|---|---|
| High traffic, few integrations | Direct outreach with a named engineer, because the blast radius is large and the owner is reachable |
| Low traffic, many integrations | Signalling, documentation and brownouts, because individual contact does not scale to 11,900 |
| Zero traffic in 90 days | Candidate for retirement with no notice beyond the published policy |

**Give clients their own usage report.** An integration that can see which deprecated fields it is receiving can act without you contacting it. This is the single cheapest lever in the whole migration, and it is the one most often skipped because it looks like a product feature rather than an engineering one.

!!! warning "When not to build usage telemetry"

    Below roughly 20 known integrations with named owners, a spreadsheet and an email thread are better: they carry context a rollup never will, such as who is on holiday and which client is being rewritten anyway. Usage telemetry also creates a data set about your customers that needs a retention policy. The measurement that justifies building it is an integration count above what one person can hold in their head.

### Deep dive two: two wire shapes over one domain model

The dives on this page vary by problem. The first covered how you learn what is safe to change; this one covers how you carry two contracts at once without the code fracturing, which is a problem no greenfield case study on this page has.

**Three ways to run two versions, and only one of them ends.**

| Approach | What it costs | How it ends |
|---|---|---|
| Deploy two copies of the service | Two of everything, and they diverge within one quarter | It does not. Merging them later is a second migration |
| Branch inside business logic on version | Every function grows conditionals, and the conditionals never get deleted | Badly. Removal requires touching every branch and re-testing everything |
| Translate at the edge, one domain model behind | One translation module per legacy shape | Cleanly. Retirement is deleting one module and its tests |

**Translation at the edge is the answer, and its defining property is that the domain model does not know versions exist.** The order service holds an amount with a currency and a precision. The current shape serialises that directly. The legacy shape converts it to an integer number of hundredths, and refuses when it cannot. The moment version logic leaks past the edge, you have chosen the second row without meaning to.

**The translation must be able to fail.** A translator that always produces something is how a silent wrong value ships. The legacy translator has exactly three outcomes: an exact value, an explicit absence, or an error with a machine-readable reason. Rounding is not among them.

**The version selector should be explicit and should not be in the path.** Two options, and the trade-off is worth naming:

| Mechanism | Advantage | Disadvantage |
|---|---|---|
| A version in the path | Visible in every log and every example; impossible to forget | Every resource has two addresses, so links, caches and bookmarks fragment |
| A version declared per request or pinned per key | One address per resource, and a key can be pinned to what it was built against | Invisible in a log line unless you deliberately record it, and easy to omit by accident |

The design I would defend is a per-key pin with a per-request override: an integration is pinned to the shape it was created against, so silence never means change, and a client that wants the new shape can ask for it explicitly before repinning.

**Silence must never mean "give me the newest thing".** A client that sends no version is by definition one that was not written with versions in mind, which makes it the least likely to survive a change. Defaulting the unversioned caller to the latest shape breaks precisely the clients that are least equipped to notice.

**Freeze the legacy shape at the split, and say so.** No new fields, no new endpoints, no new values in existing enumerations. Two consequences follow, and both are good: the translation module stops growing, and the incentive to migrate becomes a feature gap rather than an email.

**Signal deprecation in a way a machine can read.** RFC 9745, [The Deprecation HTTP Response Header Field](https://www.rfc-editor.org/rfc/rfc9745.html) (March 2025, accessed August 2026), defines a response header that states a resource is deprecated and a link relation pointing at migration documentation. RFC 8594, [The Sunset HTTP Header Field](https://www.rfc-editor.org/rfc/rfc8594.html) (May 2019, accessed August 2026), defines a header carrying the date the resource is expected to stop responding. Using both means the information is in every response, not only in an email that was filtered.

**Test the translator against recorded traffic, not against your own examples.** Replay a sample of real legacy requests through the translator and compare responses with what the old system produced, field by field. Differences are either intended (and should be listed) or bugs. Consumer-driven contract tests are the stronger version of this where clients will participate, and recorded traffic is the version that works when they will not.

!!! warning "When not to translate at the edge"

    If the two shapes differ in behaviour rather than in representation, for example the old one is eventually consistent and the new one is not, translation cannot bridge them and pretending it can produces subtle wrongness. In that case the honest move is a separate resource with a new name and no migration path. The measurement that separates the two is whether every legacy field is a pure function of the current domain model.

### Failure modes and evaluation (brownfield)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Silent wrong value | A rounded legacy amount is well formed and incorrect | Translation refuses rather than rounds, plus synthetic checks per currency asserting exactness or absence |
| Unversioned default drift | A client that sent no version is served the new shape and breaks | Pin every key to the shape it was created against; silence never means change |
| Long-tail breakage | 360 integrations fail simultaneously on retirement day | Graduated brownouts and a rising error percentage so failure is experienced before it is total |
| Version leakage | A conditional on version appears inside business logic | Architectural test asserting the version selector is not referenced outside the translation module |
| Measurement blind spot | A quarterly job is invisible in a 7 day usage window | A 90 day observation window, aggregated per key rather than sampled per request |
| Missed deadline | The retirement date slips and clients learn that dates are negotiable | Publish a date only once the usage curve supports it, and treat slipping it as costlier than setting it late |
| Frozen shape thaw | A new feature is added to the legacy shape under commercial pressure | Make the freeze a written policy with a named approver, because the exception request will come from outside engineering |
| Support overload | Brownout windows generate a burst of tickets | Schedule brownouts with support staffed, publish them in advance, and prepare a single linked explanation |

Service level indicators to publish:

- Count of distinct API keys receiving a legacy-shape response in the last 7 days, and separately in the last 90 days. The gap between the two numbers is your quarterly-job population.
- Share of total requests served in the legacy shape, tracked weekly against the retirement date.
- Count of legacy responses where a field was omitted because it could not be represented, per key. This finds clients heading for breakage before the date arrives.
- Translation error rate, split by reason, since a rise in one reason usually means a new currency or a new value reached a translator that cannot express it.
- Support contacts per brownout window, which is the honest measure of what the migration is costing people.
- Number of integrations whose declared shape is inferred rather than explicit, which should trend to zero.

Alert on translation errors with a reason you have not seen before, because it means the domain model has grown something the legacy shape cannot represent and nobody noticed. Alert on any legacy response that produced a rounded value, which should be structurally impossible and therefore indicates a code path that bypassed the translator. Do not alert on legacy traffic share: it is a programme metric that moves over months and belongs on a weekly review, not a pager.

Re-checking against the non-functional requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Correct amounts for all supported currencies | Yes in the current shape; in the legacy shape the field is absent rather than wrong, which is the strongest guarantee available |
| Existing integrations keep working for 12 months | Yes for two-decimal currencies, which is 96 percent of orders. Not for the other 4 percent, and that exception is stated rather than hidden |
| Explicit shape selection | Yes, pinned per key with a per-request override, so an unversioned client never drifts |
| Machine-readable deprecation | Yes, via deprecation and sunset headers on every legacy response, which reaches the 11,900 integrations outreach cannot |
| Per-integration usage reporting | Yes, at 360,000 rows per day, with a 90 day window chosen to catch quarterly callers |
| Retirement on a stated date | Yes, with graduated failure. It does not mean nobody breaks; it means everyone who breaks was warned in a way a machine could read |
| Bounded maintenance cost | Yes, because the legacy shape is frozen. Without the freeze, this requirement is unmeetable at any effort |

### Where this answer is a simplification (brownfield)

- **Money has more corners than decimal places.** Rounding rules, allocation of a remainder across line items, currency conversion timing and the difference between a display amount and a settlement amount are all part of a real fix, and each has its own compatibility story.
- **Contract testing works only with willing counterparties.** Consumer-driven contract tests are the correct answer when your consumers will run them, which internal consumers usually will and 11,900 public integrations will not. Recorded-traffic replay is the fallback and it is weaker.
- **The negotiation is often the real work.** Retirement dates for large integrations are commercial conversations, and the engineering plan has to survive one of them moving. Building the timeline with slack in it is a design decision.
- **Client libraries change the shape of the problem.** If most integrations use a library you publish, you can move the translation into it and retire the wire shape far faster. That option is worth asking about in the first five minutes and is easy to forget.
- **Primary sources with dates.** RFC 9745, [The Deprecation HTTP Response Header Field](https://www.rfc-editor.org/rfc/rfc9745.html) (March 2025, accessed August 2026), and RFC 8594, [The Sunset HTTP Header Field](https://www.rfc-editor.org/rfc/rfc8594.html) (May 2019, accessed August 2026), are the two standards for signalling deprecation and retirement machine-readably. RFC 9457, [Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html) (July 2023, accessed August 2026), gives the error body for a translation refusal, which needs a machine-readable reason. RFC 9110, [HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) (June 2022, accessed August 2026), covers the status semantics for a retired resource.

### Level delta (brownfield)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| First move | Adds the new field alongside the old one | Does that too, then asks which values the old field cannot represent | Notices the answer is 4 percent of orders and makes absence, not rounding, the rule |
| Versioning | Adds a version to the path | Pins a version per key so silence never means change | Keeps version knowledge inside a translation module and adds a test that it never leaks into business logic |
| Evidence | Assumes the field is still used | Measures usage per key per field | Aggregates rather than samples, sets a 90 day window from the calling pattern, and separates received from depended on |
| Communication | Sends an email and updates the docs | Adds deprecation and sunset headers so the signal is machine-readable | Derives that outreach to 11,900 integrations is 992 hours and treats signalling as the primary channel, not a courtesy |
| Endgame | Picks a cutover date | Adds brownouts so unattended clients surface | Freezes the legacy shape so maintenance is bounded, which is a stronger incentive than any notice |
| Honesty | Says nobody will break | Says the tail will break and plans for it | Names the number, 360 integrations, and designs graduated failure so breakage is experienced before it is total |

What each level added, in one line each:

- **Mid to senior:** the change is driven by evidence about real usage rather than by assumptions about it, and the signal reaches clients through a channel that scales.
- **Senior to staff:** the migration is given an end, through a freeze, a measured curve and a failure mode chosen deliberately rather than arrived at by default.

---

## 10. Practice problems

Two sets doing two different jobs. The blocked set rehearses one move at a time with the topic already named, which is how you build the move. The interleaved set hides the topic, so your first job is deciding which move applies. Only the second job is observable in an interview.

Work every problem on paper or out loud before opening an answer. Rubrics, model answers and diagnosed wrong answers for all twelve problems are in [Section 11](#11-self-assessment-rubrics).

### 10a. Blocked set: six problems, same shape as the worked examples

**B1. Derive a resource model and find the invariant that sets the transaction boundary.**

The prompt: "Model the domain for a workshop booking product. Workshops run as scheduled sessions, each session has a fixed capacity, learners hold seats, and a learner may cancel for a full refund until 24 hours before the session starts."

Registration for a popular session opens at a published time. Assume 2,000 requests arrive in the first 5 seconds, because a published opening time concentrates demand the way a ticket release does. Assume a single indexed row update plus commit costs about 2 ms, which is the order of magnitude the [latency ladder](index.md#the-latency-ladder) gives for a small write on local storage.

List the entities, draw the aggregate boundaries, state at least three invariants, and name the one invariant that decides where the transaction boundary sits. Then say what the 2 ms figure rules out.

??? note "Show answer"

    **Entities:** Workshop (the reusable definition), Session (one scheduled run of a workshop, with a capacity and a start time), Seat or Booking (one learner holding one place in one session), Learner, Payment, Waitlist entry.

    **Aggregate boundaries.** Three, not six.

    | Aggregate | Root | What is inside | Why the boundary is here |
    |---|---|---|---|
    | Session | Session | Session, its seats, its capacity counter, its waitlist | The capacity invariant spans the session and its seats, so they commit together |
    | Workshop | Workshop | Title, description, default capacity, publication state | Editing a description must never contend with a seat write |
    | Payment | Payment | Charge, refunds, gateway identifiers | It fails independently, it is a different failure domain, and it settles later |

    Learner is referenced by identifier from Session, never contained. If Learner were inside the Session aggregate, one profile edit would lock a session row.

    **Invariants:**

    1. `seats_taken` never exceeds `capacity` for a session.
    2. One learner holds at most one seat in a given session.
    3. A refund is permitted only while `now` is earlier than `session.start` minus 24 hours.
    4. A waitlist position is unique and dense within a session, so promoting the head does not leave a gap.
    5. A session cannot be cancelled while any seat is in a `payment_pending` state, because the money and the seat would disagree.

    **The invariant that decides the transaction boundary: number 1.** It is the only one that spans two rows which are both written by the same operation. Booking a seat must insert the seat row and increment the session counter in one transaction, which is what makes Session the aggregate root rather than Booking.

    The others do not need a transaction boundary and it is worth saying so out loud:

    - Number 2 is a unique index on the pair (session, learner) inside the same aggregate.
    - Number 3 is a comparison against two immutable values, so it is a pure function of data you already have.
    - Number 4 needs ordering, not atomicity, which a sequence per session gives you.
    - Number 5 spans two aggregates, so it is enforced by a state check plus a compensating path, not by one transaction.

    **What the 2 ms rules out.** One row serialises at 1 divided by 0.002, which is 500 transactions per second. Peak demand is 2,000 requests over 5 seconds, so 400 per second, which is 80 percent of that ceiling.

    That number kills three options at once:

    - It kills putting the payment gateway call inside the seat transaction. A 2 second gateway call takes the ceiling to 0.5 per second, and the queue is 400 deep after one second.
    - It kills sending the confirmation email inside the transaction, for the same reason at a smaller scale.
    - It kills a single global counter shared across sessions, since every session would contend on one row rather than on its own.

    It does not force sharding, a queue, or a reservation service. At 80 percent of a single row's ceiling, for 5 seconds, with a short transaction, the correct design is one transaction and a retry on serialisation conflict.

    !!! warning "When not to keep a capacity counter at all"

        If capacity is not a real constraint, there is no invariant, so there is no aggregate to defend. A webinar with a stated cap of 100,000 and a median of 300 registrations does not need a counter, a lock or a waitlist: count the rows on read. The measurement that justifies the counter is a session whose fill rate reaches its capacity, or a measured contention rate above roughly 10 percent of writes retrying, which is the threshold the [trade-off atlas](index.md#the-trade-off-atlas) uses for optimistic concurrency generally.

**B2. Choose the exposure style on client count, not on taste.**

Your pricing engine computes a price for a basket. Two populations call it.

| Caller | Volume at peak | Payload | Can you redeploy them? |
|---|---|---|---|
| Three internal services, same repository, same release train | 4,000 calls per second | 2 KB request, 1 KB response | Yes, on your own schedule |
| Forty partner integrations, each a company you have never met | 5 calls per second total | Same shapes | No, and some have not shipped in two years |

Choose the exposure style or styles. Compute one number that eliminates a popular answer, and name the condition under which exposing this domain two ways is correct rather than lazy.

??? note "Show answer"

    **First, kill the number that people reach for.** Bandwidth: 4,000 calls per second times 3 KB of request plus response is 12 MB per second, or about 96 Mbit per second. That is unremarkable on any modern network, so bandwidth eliminates nothing and does not belong in the answer.

    **Second, kill the second number people reach for.** Serialisation cost. Assume parsing 2 KB of JSON costs about 20 microseconds and a binary format costs about 4, which is the right order of magnitude for a schema-compiled decoder against a text parser; substitute your own measurement. The saving is 16 microseconds times 4,000, so 64 milliseconds of CPU per second, which is 6.4 percent of one core. That eliminates "we need a binary protocol for efficiency" as an argument at this volume.

    **What is left is the only thing that decides it: who you can redeploy.**

    | Property | Internal three | Partner forty |
    |---|---|---|
    | Clients you control | 3 | 0 |
    | Cost of a breaking change | Three pull requests, one afternoon | A deprecation programme measured in quarters |
    | Right exposure | Procedure style with a schema both ends compile | Representational resources, coarse and stable |

    **The answer: one domain service, two exposures.** A `PriceBasket` procedure for the internal three, and a `POST /quotes` resource returning a quote with an identifier and an expiry for the forty. Both are thin adapters over the same pricing domain service. Neither reimplements a rule.

    **The condition that makes two exposures correct rather than lazy.** Two client populations with different upgrade rights and different change cadences, and a second exposure that is a projection rather than a second implementation. If both adapters are under a hundred lines and neither contains a pricing rule, you have one design with two front doors. The moment a rule appears in only one of them, you have two products, and they will disagree in a way a customer discovers first.

    **The condition that makes it lazy.** Two exposures for one population, or a second exposure created because a team preferred a different tool. That is a maintenance tax with no client on the other end of it.

    !!! warning "When not to expose the same domain twice"

        Below roughly two external consumers, or when every consumer is on the same release train as you, ship one exposure. The second exposure costs a second schema to lint, a second set of error codes to keep consistent, and a permanent risk of drift. The measurement that justifies it is the arrival of the first consumer whose deploy you cannot schedule.

**B3. Give a bulk endpoint honest partial-failure semantics.**

`POST /contacts/bulk` accepts up to 500 contacts in one call. From the single-item endpoint you have measured that about 0.4 percent of submitted contacts fail validation, most often a malformed email address.

A colleague proposes: validate all 500, and if any item fails, reject the whole batch with 400 so the client can fix and resubmit. Compute the number that decides this, then specify the error model: the taxonomy, the retryability signal and the per-item shape.

??? note "Show answer"

    **The number.** If each item independently passes with probability 0.996, a full batch of 500 passes with probability 0.996 to the power 500. That is about 0.135, so roughly 13 percent of batches are clean and 87 percent contain at least one bad item.

    All-or-nothing therefore rejects 87 percent of calls. The client's only recovery is bisecting the batch to find the offender, which converts one call into several and multiplies your load precisely when clients are struggling.

    Shrinking the batch does not save it. At 50 items the clean probability is 0.996 to the power 50, about 0.818, so 18 percent of batches still fail entirely. The design is wrong at every batch size; it is only less obviously wrong at small ones.

    **The taxonomy.** Two levels, because a bulk call has two levels of failure.

    | Level | Meaning | Status | Body |
    |---|---|---|---|
    | Envelope | The call itself was unacceptable: bad token, malformed JSON, over the item limit, over quota | 401, 400, 413, 429 | One problem object, no per-item results |
    | Item | The call was processed and individual items succeeded or failed | 200 | Per-item results array |

    **The per-item shape.** Every item result carries the submitted index, the outcome, and for failures a stable machine-readable code plus a retryability flag.

    ```
    { "accepted": 487, "failed": 13,
      "results": [
        { "index": 0,  "status": "created", "id": "cnt_9f2" },
        { "index": 7,  "status": "failed",
          "code": "invalid_email", "retryable": false,
          "field": "email" },
        { "index": 41, "status": "failed",
          "code": "storage_unavailable", "retryable": true,
          "retry_after_ms": 2000 } ] }
    ```

    **Three rules that make this usable, each of which candidates miss:**

    - **Retryability is a field, not an inference from the status.** Index 7 and index 41 both failed inside a 200 response. One must never be resubmitted, one must be. The status code cannot express that, and a client that guesses will either duplicate contacts or drop them.
    - **The index is the join key.** Return the submitted position, not just the identifier, because failed items have no identifier and the client has to map results back to its own array.
    - **The whole call needs one idempotency key, and each item needs its own natural key.** Otherwise a client retrying the batch after a network timeout re-creates the 487 that succeeded.

    **On 207.** Multi-Status exists and comes from WebDAV, [RFC 4918](https://www.rfc-editor.org/rfc/rfc4918.html) (June 2007, accessed August 2026). It is a defensible choice.

    The reason to prefer a plain 200 is that intermediaries, client libraries and dashboards branch on the status class rather than the exact code. So 207 buys nothing the body does not already carry, and it costs a support conversation with every client whose HTTP library handles unknown 2xx codes badly.

    Use the standard problem body from [RFC 9457](https://www.rfc-editor.org/rfc/rfc9457.html) (July 2023, accessed August 2026) at the envelope level, and your own item shape inside.

    !!! warning "When not to build partial-success semantics"

        If the items in a batch are genuinely one unit of work, all-or-nothing is correct and partial success is a bug. Posting the two sides of a double-entry ledger transaction is one operation wearing a plural name.

        The test: ask whether a client would ever want to keep the successful subset. If the answer is no, use one transaction, one status, and say so in the documentation.

        Partial success also costs an error path per item that has to be tested. Below roughly 10 items per call, a loop over the single-item endpoint is simpler and the client already knows how to handle it.

**B4. Specify idempotent creation, then name the retry that still duplicates.**

`POST /transfers` moves money between two accounts you hold. Facts you have:

- The client library retries up to 4 times with full-jitter backoff at 1, 2, 4, 8 and 16 seconds, so a burst of retries spans at most 31 seconds.
- Clients also run a nightly reconciliation job that resubmits anything it cannot confirm, so a replay can arrive up to 24 hours later.
- You process about 2 million transfers a day.

Specify the key, its lifetime, the fingerprint and the transaction boundary. Then write out the exact retry sequence that still produces two transfers if the boundary is drawn one line too late.

??? note "Show answer"

    **The key.** Client-supplied, opaque, scoped to the pair (tenant, key), never global. A global key space lets one tenant's key collide with another's, which is both a correctness bug and a way to learn that another tenant exists.

    **The lifetime, derived.** The key must outlive the longest replay the client can produce. That is 24 hours for the nightly job plus 31 seconds of in-burst retries, so anything under 24 hours is provably too short. Ship 48 hours, because a job that runs late, a weekend shift or an hour of clock disagreement all sit inside the extra day.

    Storage check: 2 million per day times about 1 KB per key record times 2 days of retention is about 4 GB live. One table holds that without a conversation. Keeping keys for a year would be 730 GB, which is a uniqueness index nobody needs and the reason the window is bounded rather than infinite.

    **The fingerprint.** A hash of the operation name, the key, the authenticated principal and a canonicalised body. On a replay:

    | Situation | Response |
    |---|---|
    | Key absent | Proceed, and write the key with the effect |
    | Key present, fingerprint matches, effect complete | Replay the original response, byte for byte, with a header saying it is a replay |
    | Key present, fingerprint matches, effect still running | 409 with a machine-readable code and `Retry-After`, so the client waits rather than starting a second attempt |
    | Key present, fingerprint differs | Refuse with a distinct code. Never replay, and never process. The client has a bug, and either answer would be wrong |

    **The transaction boundary.** The key row and the transfer row are inserted in one transaction, with a unique index on (tenant, key). One statement, one commit, no window.

    **The retry sequence that still duplicates, when the key is written after the effect.**

    ```
    t=0.00  Request A arrives, key K, amount 500
    t=0.05  BEGIN; insert transfer T1; COMMIT      <- effect durable
    t=0.06  process crashes before writing key K
    t=1.00  Client sees a timeout, retries with key K
    t=1.05  Handler looks up K: absent
    t=1.06  BEGIN; insert transfer T2; COMMIT      <- second transfer
    t=1.07  insert key K, pointing at T2

    Caption: two transfers, one key. The key now protects the
    duplicate and not the original, and no later retry can
    detect the damage.
    ```

    **The mirror-image failure, when the key is written first in its own transaction:**

    ```
    t=0.00  Request A, key K
    t=0.02  insert key K (own transaction, committed)
    t=0.05  transfer insert fails, request returns 500
    t=1.00  Client retries with key K
    t=1.02  Handler finds K, replays a success response
            for money that never moved

    Caption: one key, zero transfers, and a client holding a
    receipt for nothing. This is the worse of the two, because
    it is silent.
    ```

    **The third sequence, which the boundary cannot fix.** Two requests carrying key K arrive one millisecond apart, both find the key absent, and both attempt the insert. The unique index makes one fail with a constraint violation. That is correct but it is not a good client experience, and it is exactly why the key record has a reserved state as well as a completed one. Without the reserved state, the loser gets a raw database error or blocks on a row lock while holding one of your connections.

    !!! warning "When not to build a key store"

        Below roughly one write per second, or when the operation already carries a natural unique business key you store anyway, a unique constraint on that key is cheaper and has fewer states. A key table adds a second write, an expiry job and a new error code your clients must handle. The measurement that justifies it is the first duplicate-effect incident, or a client retry rate above about 1 percent of requests.

**B5. Compare pagination styles on a collection that changes while you read it.**

A collection of 40 million rows, sorted newest first, served 50 per page. Measurements and assumptions:

- Skipping one index entry costs about 0.5 microseconds. That is a few main memory reads, and the [latency ladder](index.md#the-latency-ladder) puts a main memory read at about 100 nanoseconds.
- The read budget for one page is 20 milliseconds.
- New rows arrive at the head at 6 per second; rows are deleted at 2 per second.
- A reader spends about 3 seconds between requesting one page and the next.

Give the depth at which offset stops being defensible, and name the anomaly each style produces.

??? note "Show answer"

    **Offset depth, derived.** Offset makes the database produce and discard every row before the window. At 0.5 microseconds per discarded entry, a 20 millisecond budget buys 20 divided by 0.0005, which is 40,000 discarded rows. At 50 rows a page, that is page 800.

    Beyond that the skip alone exceeds the budget before a single returned row is read. At offset 100,000 the discard is 100,000 times 0.5 microseconds, which is 50 milliseconds, so page 2,000 is two and a half times over budget doing nothing useful. Offset is defensible to roughly page 800 on this collection and indefensible after it.

    Keyset does not have this shape at all. A seek into a balanced index over 40 million rows is about log base 2 of 40 million, which is 25 comparisons, and it costs the same on page 800 as on page 1. That single comparison, 25 against 40,000, is the whole argument.

    **The anomalies, which are the part that gets you hired.**

    | Style | What it encodes | Anomaly under concurrent writes | Frequency here |
    |---|---|---|---|
    | Offset | A count of rows to discard | Duplicates when rows are inserted ahead of your window, skips when rows are deleted behind it | 6 inserts per second over a 3 second gap shifts the window by 18 rows, so 18 of the next 50 rows repeat, which is 36 percent of the page |
    | Offset, deletions | Same | Rows never seen at all | 2 deletes per second over 3 seconds removes 6 rows, so 6 rows slide past the window unread per page turn |
    | Cursor over a non-unique sort key with no tiebreaker | The last sort value | Duplicates and skips at every tie, silently | Any ties at all. Timestamps at second resolution tie constantly |
    | Cursor or keyset over a mutable sort column | The last value of a column that can change | A row that is edited moves behind the cursor and is never returned, or moves ahead and is returned twice | Every update to the sort column of an already-paged row |
    | Keyset over an immutable unique key, or a compound key with a unique tiebreaker | Last sort value plus last identifier | Newly inserted rows ahead of the cursor are simply not in this pass, which is a stable and explainable result | The correct answer here |

    **What the cursor must encode**, minimally: the sort key value of the last row returned, a unique tiebreaker (the row identifier), the sort direction, and a fingerprint of the filters. The fingerprint is what stops a client changing a filter mid-scan and receiving a result set that belongs to neither query.

    Encode it opaquely, as a signed or at least base64 blob, and say in writing that it is opaque. If it is readable, clients will parse it, and then its format is part of your contract forever.

    **Total counts.** An exact count over 40 million rows means visiting 40 million index entries. At the same 0.5 microseconds that is 20 seconds. That eliminates an exact `total` field on every page. Offer `has_more`, or a separately cached estimate with its staleness stated.

    !!! warning "When not to use cursors"

        On a bounded collection, say under a thousand rows, where the product genuinely wants numbered pages and a jump-to-page control, offset is the right answer and a cursor costs you a feature. Cursors also make "sort by a column the user picks" much harder, because each sort order needs its own tiebreaker and index. The measurement that justifies cursors is either a page depth above the number you derived above, or a single complaint about a duplicated or missing row on a live feed.

**B6. Plan a breaking change for clients you cannot force to upgrade.**

Your public interface returns `customer.name` as one string. Product needs `given_name` and `family_name` separately, and needs one value removed from the `status` response enum because it can no longer occur.

Facts: 4,200 API keys made a call in the last 30 days, total traffic is about 500 million calls a month, and 22 handlers touch the `name` field.

Give the compatibility classification, the signalling, the migration order, and the measurement that ends the deprecation window.

??? note "Show answer"

    **Classification first, because the two changes are not the same kind of change.**

    | Change | Class | Why |
    |---|---|---|
    | Adding `given_name` and `family_name` alongside `name` | Compatible | Clients ignore fields they do not know, provided you told them to in writing |
    | Removing `name` | Breaking | Every client reading it fails, and you cannot see which ones do without measuring |
    | Removing a value from a response enum | Breaking in practice | Generated clients compile a closed set. Removing a value breaks nothing today and breaks everything if it ever reappears, so the honest classification is that the enum's contents are frozen |
    | Adding a value to a request enum | Compatible | You are widening what you accept |
    | Adding a value to a response enum | Breaking in practice | A client with a closed parser throws on the new value. This is the one people classify wrong |

    **Signalling.** Three mechanisms, and none of them is a blog post.

    - A `Sunset` header carrying the date on every response that uses the deprecated field, defined in [RFC 8594](https://www.rfc-editor.org/rfc/rfc8594.html) (May 2019, accessed August 2026).
    - A `Deprecation` header naming the field. This has been through the IETF process and you should check its current status before quoting a specification number to anyone.
    - A machine-readable deprecation notice in the schema itself, so generated clients emit a compile-time warning rather than relying on a human reading a header.

    **Migration order.** The order is the answer; the mechanisms are commodity.

    1. **Measure before you announce.** Log field-level access per API key: which keys read `name`, and which write it. Without this you are negotiating with an unknown population, and every subsequent step is guesswork.
    2. **Expand.** Add both new fields, populated, alongside the old one. Nothing breaks and nothing is announced yet.
    3. **Switch the writes.** Accept both shapes on input, prefer the new one, and derive the old one for output. The derivation is deliberately lossy in one direction, and you document which.
    4. **Announce with the measurement attached.** Tell each affected key what it specifically is using, not a generic notice. A key whose owner is told "you call `name` on 14,000 requests a day" migrates; a key whose owner receives a newsletter does not.
    5. **Default the new shape for new keys.** Every integration created after this date never sees the old field, so the affected population is now closed and can only shrink.
    6. **Narrow.** Remove the old field, in a new major version if you have one, behind a per-key flag if you do not.

    **The measurement that ends the window.** Not a date. Two numbers and a list.

    - Calls per day touching the deprecated field, weighted by the commercial weight of the keys making them. Raw call counts flatter you, because one large integration can be 0.001 percent of keys and 40 percent of revenue.
    - The count of distinct keys that have made zero deprecated calls for 30 consecutive days, which is the only evidence that a client actually migrated rather than paused.
    - A contact list of every remaining key with a named owner on their side.

    Worked: suppose at month six the deprecated field carries 0.02 percent of 500 million monthly calls, which is 100,000 calls a month, from 38 distinct keys. Thirty-eight keys is a list one person can work through at 8 conversations a day in about 5 days. That is the moment the window can end, and the reason it can end is that the remaining population is small enough to talk to individually, not that six months elapsed.

    **The cost of never ending it**, which is the sentence that reads as staff level: dual serving puts a branch in each of the 22 handlers that touch the field, doubles the test matrix on those handlers, and every future change to customer naming has to be made twice. That is the ongoing price of the compatibility promise, and it is why the window needs an exit condition rather than a hope.

    !!! warning "When not to run a deprecation programme at all"

        If every consumer is inside your own deployment boundary, this apparatus is waste. Change the field, fix the callers, ship it in one change. The measurement that justifies the programme is the existence of one consumer you cannot redeploy on your own schedule. Below that, the programme's real cost is that it makes every change take a quarter, which teaches your team not to make changes.

### 10b. Interleaved and cumulative set: six problems, topic not stated

These are unlabelled on purpose. Decide which tool applies before you start designing. Several reach back to material that is not on this page: the [trade-off atlas](index.md#the-trade-off-atlas) and the [failure library](index.md#the-failure-library) on the hub, [Modern System Design](modern-system-design.md) for what sits behind the contract, [ML System Design](ml-system-design.md) for a model-backed surface, and the [system design question bank](/Interview-Questions/System-design/) for the storage and partitioning background.

Twenty to thirty minutes each, out loud, with a timer.

**I1.** We have an internal orders service. Three consumers, all in this repository, all deployed by us on the same release train. A team member has proposed adding URI versioning with a v1 and v2 router, plus a six-month deprecation policy for every change. Design it.

**I2.** Here is what runs today: a public interface, 9,000 API keys active in the last 30 days, one flat limit of 100 requests per minute per key enforced with a fixed window at the minute boundary, no plans and no metering. Product wants three paid tiers with different limits, plus per-call billing on one expensive endpoint, live in a quarter, and no existing integration may break.

**I3.** Our p99 for `POST /orders` is 140 milliseconds and has not moved in a month. Three partners opened tickets this week saying our interface times out. Our error rate dashboard is green. Diagnose it and say what you would change.

**I4.** We are going to expose our ranking model to partners as an endpoint. The model is retrained weekly and the ranking changes when it is. Design the contract.

**I5.** A partner reports that they received the same event three times, and that one event arrived out of order relative to another for the same object. Our dispatcher's dashboards are green and its delivery lag is under a second. What do you tell them, and what do you change?

**I6.** Next quarter we are partitioning the orders table by tenant. What changes in the public contract, and what would you do this quarter to make the answer be "nothing"?

!!! warning "Honesty note about the interleaved set"

    You will score lower here than on the blocked set, and it will feel like the reading did not work. The gap is the design. The blocked set measures whether you can execute a move once somebody has named it; the interleaved set measures whether you can pick one, which is the only thing an interviewer can observe. A drop of one or two rubric levels between the two sets is the expected outcome, not a signal to reread the modules.

---

## 11. Self-assessment rubrics

### The master rubric

Every problem in Section 10 is scored on the same four criteria, with named levels rather than numbers. A visible score crowds out the comment, and the comment is the part that changes your next answer.

| Criterion | Developing | Adequate | Strong | Exceptional |
|---|---|---|---|---|
| Problem framing and scoping | Starts listing endpoint paths before naming an entity. Restates the prompt back as the requirements | Asks questions, but no answer would move a resource, a status code or a boundary | Every question has two branches and the chosen branch is stated. Names who cannot be upgraded before designing anything | Names the consumer nobody mentioned (the internal batch job, the support tool, the partner's nightly reconciliation) and designs for it |
| Design and trade-off reasoning | One surface, presented as the answer. No alternative named | Alternatives named, chosen by habit or by what the last team did | Each choice is settled by a number or an invariant stated in front of the interviewer, and the rejected option is named with the condition under which it wins | Identifies which decision is expensive to reverse (identifier format, enum contents, the presence of a non-terminal state) and sequences the design around it |
| Depth in at least one area | Path level everywhere. Errors are "we return 400" | Goes one layer deeper when pushed | Volunteers one dive that reaches implementation detail: the key record's states, the cursor's contents, the field-level access log behind a deprecation | The dive includes how it fails silently, what would detect it, and what the recovery costs the client rather than only what it costs you |
| Communication and drive | Waits to be asked what comes next | Answers well, does not move the conversation | States the plan, keeps the clock, offers forks instead of asking for approval | Absorbs a hostile follow-up by changing the design and naming which earlier claim is now void |

Score all four on every problem. Two Strongs and two Adequates is a passing senior answer. Four Adequates with nothing Strong is the profile most often written up as "hire, one level down", because nothing in it was memorable.

### Rubric notes, blocked set

??? note "B1 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks whether capacity is a real constraint before designing a counter | Assumes a waitlist because the prompt used the word |
    | Trade-offs | Uses the 500 per second row ceiling to eliminate three options, not to justify one | Computes the ceiling and never refers to it again |
    | Depth | Says which invariants do not need a transaction, and why | Lists invariants without saying which one is load-bearing |
    | Communication | States the aggregate boundary as a decision with a cost, then moves | Narrates the modelling process rather than its output |

    Model answer: three aggregates. Session owns its seats, its capacity counter and its waitlist, because `seats_taken` never exceeding `capacity` is the one invariant that spans two rows written by the same operation, which makes Session the root and puts the transaction there. Workshop is separate so a description edit never contends with a seat write. Payment is separate because it fails independently and settles later.

    The other invariants are cheaper than they look. One learner per session is a unique index inside the aggregate. The 24 hour refund rule is a comparison of two values I already hold. Waitlist density needs ordering, which a per-session sequence gives me. Only capacity needs atomicity.

    On numbers: 2,000 requests in 5 seconds is 400 per second against a single-row ceiling of 500 per second at 2 milliseconds a transaction. That is 80 percent utilisation, which is survivable for a short burst with a retry on serialisation conflict, and it kills putting the payment call or the email inside that transaction outright.

    Wrong answer: "Booking is the aggregate root, and a background job reconciles overbooking." Reveals that the candidate has moved a correctness invariant into an eventual process. Overbooking is discovered after a learner has a confirmation, so the compensation is a human apology rather than a rollback. If the product genuinely accepts overbooking, that is a stated product decision with a stated rate, not a side effect of a modelling choice.

    Wrong answer: "Put a distributed lock service in front of the session." Reveals machinery chosen by association. The contended resource is one row in one database that already has row locks and a transaction. The lock service adds a network hop inside the critical section and a new way to fail, to defend an invariant the database was already defending.

    Wrong answer: an entity list with no aggregate boundaries: Workshop, Session, Seat, Learner, Payment, Waitlist, all peers. Reveals modelling as vocabulary rather than as boundary-drawing. Without boundaries there is nothing to say about transactions, and every subsequent question about concurrency has no anchor.

    Compare your process: did you find the invariant that spans two rows, or did you decide the boundary first and then look for support?

??? note "B2 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Separates the two client populations in the first sentence | Treats "the callers" as one group |
    | Trade-offs | Computes bandwidth and serialisation, then deletes both because neither eliminates anything | Cites efficiency with no number |
    | Depth | States what makes two exposures legitimate and what makes them two products | Says "we will do both" with no condition |
    | Communication | Names the deciding quantity out loud before choosing | Presents a conclusion and defends it afterwards |

    Model answer: the deciding quantity is the count of clients I can redeploy. Internally that is three, all on my release train, so a breaking change costs three pull requests and one afternoon. Externally it is forty companies I have never met, some of whom have not shipped in two years, so a breaking change costs a quarter.

    Bandwidth is 4,000 times 3 KB, about 12 MB per second, which decides nothing. Serialisation saving from a binary format is about 16 microseconds per call times 4,000, which is 64 milliseconds of CPU per second, about 6 percent of a core. That also decides nothing, and I would say so rather than let it sit in the answer.

    So: a procedure-style call for the internal three with a schema both ends compile, and a `POST /quotes` resource returning a quote with an identifier and an expiry for the forty. Both are thin adapters over one pricing domain service. Two exposures are correct when the populations have different upgrade rights and the second is a projection; they are lazy when a pricing rule lives in only one of them, at which point they are two products that will disagree in front of a customer.

    Wrong answer: "Use a graph style so every consumer takes only the fields it needs." Reveals a solution to a fetch-shape problem the prompt does not have. One operation returning one price has no over-fetching to solve, and the graph style would hand forty external parties the ability to compose queries whose cost you have to bound.

    Wrong answer: "Binary everywhere, it is faster." Reveals efficiency asserted rather than measured. Six percent of a core is the entire prize, and the price is browser reach, harder debugging for forty partners, and a schema compiler in every partner's build.

    Wrong answer: "One representational interface for everybody, internal callers included." Reveals a rule applied without a cost. It is not wrong in the way the previous two are, and at three internal callers it is defensible. What makes it Developing rather than Adequate is naming no cost: you have just made your internal callers pay a public contract's stability tax on a surface you could change in an afternoon.

    Compare your process: did you delete a number that eliminated nothing, or did you leave it in the answer because you had computed it?

??? note "B3 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks whether the 500 items are one unit of work before designing partial success | Assumes partial success because the endpoint is plural |
    | Trade-offs | Computes 0.996 to the power 500 and uses it to kill all-or-nothing at every batch size | Argues from principle that partial success is better |
    | Depth | Makes retryability an explicit field and explains why the status cannot carry it | Returns a list of error messages with no codes |
    | Communication | States what the client does with each item class, in the client's words | Describes the response shape without saying who acts on it |

    Model answer: about 87 percent of 500-item batches contain at least one bad item, from 0.996 to the power 500 being roughly 0.135. All-or-nothing therefore rejects most calls and pushes clients into bisecting batches, which multiplies my load exactly when clients are struggling. At 50 items it is still 18 percent, so the design is wrong at every size.

    Two levels of failure. The envelope fails as a whole for bad tokens, malformed bodies, oversized batches and quota, and gets a single problem object. Items fail individually inside a 200, each carrying its submitted index, a stable code, a retryable flag and, where relevant, a retry delay.

    Retryability has to be a field. Inside one 200 response, `invalid_email` must never be resubmitted and `storage_unavailable` must be. No status code can express both at once, and a client that guesses either duplicates records or silently drops them. The batch also needs one idempotency key, so that a client retrying after a network timeout does not re-create the items that already succeeded.

    Wrong answer: "Return 400 with the list of failures, so the client knows something went wrong." Reveals a status chosen for the human reading the log rather than for the machine branching on it. A 4xx tells the client the request was not processed, which is false: 487 contacts were created, and the client will now retry the whole batch.

    Wrong answer: "Return 207 Multi-Status, that is what it is for." Reveals a specification recalled without its consequences. It is a reasonable choice, but intermediaries and client libraries branch on the class rather than the code, so it buys nothing the body does not already carry and costs a support conversation with every partner whose HTTP stack handles unknown 2xx codes badly.

    Wrong answer: "Return the identifiers that succeeded, and the client can diff." Reveals the join key being dropped. Failed items have no identifier, so the client must map results back to its own array by position, and a diff cannot tell an item that failed apart from one that was never sent.

    Compare your process: did you compute the probability before you formed an opinion about partial success?

??? note "B4 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Derives the key lifetime from the client's longest replay, not from a round number | Picks 24 hours because it sounds standard |
    | Trade-offs | Names what the key does not solve as well as what it does | Presents the key as exactly-once delivery |
    | Depth | Writes the failing timeline with timestamps and names which row committed when | Says "keep them in the same transaction" without showing the failure |
    | Communication | Distinguishes the loud failure from the silent one and says which is worse | Treats both orderings as equally wrong |

    Model answer: the key is client-supplied, opaque and scoped to (tenant, key). Lifetime is driven by the longest replay the client can produce, which is a nightly reconciliation job at 24 hours plus 31 seconds of in-burst retries, so under 24 hours is provably too short and I ship 48 to absorb a late job or a weekend. Storage is about 2 million a day times 1 KB times 2 days, so roughly 4 GB live, which needs no discussion.

    The fingerprint is a hash of the operation, key, principal and canonicalised body. Matching fingerprint plus completed effect replays the stored response. Matching plus in-flight returns 409 with a retry delay. Differing fingerprint is refused with a distinct code and never processed, because both replaying and processing would be wrong.

    The boundary: the key row and the transfer row are inserted in one transaction with a unique index on (tenant, key). If the key is written after the effect, this duplicates: request A commits transfer T1, the process dies before the key is written, the client retries at one second, finds no key, and commits transfer T2. Two transfers, one key, and the key now protects the duplicate rather than the original.

    The mirror image is worse because it is silent: key committed first, effect fails, client retries, receives a replayed success for money that never moved. And neither ordering handles two requests arriving a millisecond apart, which is what the reserved state is for.

    Wrong answer: "Store the key in a cache with a short expiry, it is faster." Reveals the boundary being traded for latency in exactly the place where the boundary is the whole point. A crash between the cache write and the database write yields either a double transfer or a phantom receipt, and you have saved a millisecond.

    Wrong answer: "The key makes the operation exactly-once." Reveals a guarantee claimed that no single service can provide across a network boundary. It deduplicates one client's retries of one request. It does nothing about the same customer instruction submitted twice under two different keys, which needs a business-level uniqueness constraint, and saying that unprompted is the senior tell.

    Wrong answer: "Return 200 with the original response on any replay, regardless of body." Reveals a missing fingerprint. A client that reuses a key with a changed amount then gets a receipt for the wrong amount, silently, and it will be discovered during a reconciliation weeks later.

    Compare your process: did you write a timeline with timestamps, or did you assert that the ordering matters?

??? note "B5 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks whether the collection mutates while it is read before choosing a style | Chooses cursors as a default with no anomaly named |
    | Trade-offs | Derives page 800 from a budget and a per-row skip cost | Says deep offsets are slow, with no threshold |
    | Depth | Names the tiebreaker and the filter fingerprint inside the cursor, and says the cursor is opaque | Says "use a cursor" and stops |
    | Communication | Quantifies the duplicate rate at 36 percent rather than calling it "possible duplicates" | Lists anomalies as risks with no frequency |

    Model answer: at 0.5 microseconds per discarded index entry and a 20 millisecond budget, offset buys 40,000 discarded rows, so page 800 at 50 per page. Page 2,000 spends 50 milliseconds discarding before it reads anything useful. Keyset seeks in about 25 index comparisons on 40 million rows and costs the same on page 800 as on page 1, which is the whole argument in one comparison.

    Under concurrent writes, offset produces duplicates from inserts ahead of the window and skips from deletes behind it. Six inserts a second across a 3 second page turn shifts the window by 18 rows, so 36 percent of the next page repeats. Two deletes a second means 6 rows slide past unread per page turn.

    A cursor over a non-unique sort key duplicates and skips at every tie, which timestamps at second resolution produce constantly, so the cursor carries the sort value plus the row identifier as a tiebreaker, the direction, and a fingerprint of the filters. A cursor over a mutable sort column has a different anomaly: an edited row moves behind the cursor and is never returned. That is the one worth naming out loud, because it is invisible to the client.

    Exact totals are out: 40 million index entries at 0.5 microseconds is 20 seconds. I would ship `has_more` and, if the product insists, a cached estimate with its staleness stated.

    Wrong answer: "Cursors always, offsets never." Reveals a rule without a threshold. On a bounded collection with a jump-to-page control, offset is correct and a cursor removes a product feature. The rule is only useful with the depth number attached.

    Wrong answer: "Encode the offset inside a base64 string and call it a cursor." Reveals the mechanism being copied without the property. Every offset anomaly survives verbatim, and now the client cannot see that it has an offset, so nobody debugs it.

    Wrong answer: "Add an index and deep offsets will be fine." Reveals a misdiagnosis of where the cost is. The index is already being used; the cost is producing and discarding entries before the window, and no index removes work you have asked the database to skip.

    Compare your process: did you attach a frequency to each anomaly, or did you list them as things that can happen?

??? note "B6 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Measures field-level usage before announcing anything | Announces a date and then discovers who is affected |
    | Trade-offs | Classifies the enum change as breaking in practice and explains the closed-parser reason | Calls additive changes safe without qualification |
    | Depth | Gives the migration order and says why measurement is step one | Lists mechanisms (headers, versions) with no sequence |
    | Communication | Ends the window on a measurement plus a contact list, not a date | Sets a six-month window and calls it a plan |

    Model answer: adding the two new fields is compatible. Removing `name` is breaking. Removing a value from a response enum is breaking in practice, because generated clients compile closed sets, and by the same reasoning adding a value to a response enum is breaking too, while adding one to a request enum is safe. That asymmetry is the part most candidates get backwards.

    Order: measure field-level access per key first, because otherwise every later step is guesswork. Then expand, populating both shapes. Then accept both on input and prefer the new one. Then announce per key, telling each owner what they specifically call and how often, because a personalised notice migrates people and a newsletter does not. Then default new keys to the new shape, which closes the affected population. Then narrow.

    The window ends on measurement, not a date: deprecated calls per day weighted by the commercial weight of the keys making them, the count of keys at zero deprecated calls for 30 consecutive days, and a contact list with a named owner on each remaining key.

    At month six, 0.02 percent of 500 million calls is 100,000 a month from 38 keys, and 38 owners is five days of conversations for one person. The window ends because the remaining population is small enough to talk to, not because six months elapsed.

    The cost I would state out loud: dual serving puts a branch in each of the 22 handlers touching this field and doubles their test matrix, and that is the standing price of the promise.

    Wrong answer: "Ship v2 and leave v1 running." Reveals versioning used as a substitute for migration. Two versions with no exit condition is two products, the old one accumulates the clients who never move, and the next change has to be made twice.

    Wrong answer: "Announce a six-month deprecation and remove it on the date." Reveals a plan with no feedback. If 400 keys are still calling on the date, you either break them or you extend, and extending after a public date is worse than never having set one.

    Wrong answer: "It is additive, so it is safe: we add the new fields and quietly stop populating `name`." Reveals a misclassification dressed as caution. A field that exists but is empty is worse than a removed field, because clients fail at the point of use rather than at parse time, and your logs show a healthy 200.

    Compare your process: did measurement come before announcement, or did you write the announcement first?

### Rubric notes, interleaved set

??? note "I1 rubric, model answer and wrong answers (the answer is not to add it)"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks who the consumers are and finds that all three are redeployable | Starts choosing between URI, header and media-type versioning |
    | Trade-offs | Prices the machinery against a saving of three pull requests | Prices only the safety, never the cost |
    | Depth | Names what versioning would actually own: a router, two handler sets, a doubled test matrix, and a policy that slows every change | Names memory or code duplication as the only cost |
    | Communication | Declines, gives the threshold at which the answer flips, and offers the cheaper thing to do now | Either builds it or dismisses the proposal as premature |

    Model answer: I would not add it, and here is the arithmetic. There are three consumers, all in this repository, all on my release train. A breaking change costs three pull requests and one afternoon. Versioning machinery costs a router, two live handler sets, a doubled test matrix on every versioned path, and a policy that puts six months between wanting a change and making it.

    The threshold at which the answer flips is the arrival of the first consumer I cannot redeploy on my own schedule: a mobile binary in a store, a partner, a customer's script, or a pipeline owned by another team with its own release train. Not a consumer count, a consumer type.

    What I would do this quarter instead, three things:

    - Consumer-driven contract tests in continuous integration, so a breaking change fails the build of the consumer that would break, rather than being caught by a policy document.
    - Expand and contract as the standard shape for every change.
    - A schema linter in the build that classifies each diff as compatible or breaking, so the discussion happens at review time rather than in an incident.

    Those three cost about a week, they keep the afternoon-sized change afternoon-sized, and they are exactly the groundwork you want in place on the day the answer does flip.

    Wrong answer: designing the versioning scheme competently, as asked. Reveals a candidate who optimises whatever they are pointed at. This prompt exists to see whether you will produce a number that contradicts the request.

    Wrong answer: "No, that is premature optimisation." Reveals the right instinct with no evidence. A refusal without arithmetic scores the same as compliance without arithmetic, because neither shows a decision process.

    Wrong answer: "Add it now, because we will go public eventually." Reveals designing for an unstated future. Name the date the surface goes public and the lead time to add versioning, and the argument becomes legitimate. Without those two numbers it is a preference, and it costs the team velocity every week until the date arrives.

    Compare your process: did you state the condition that would change your mind, in the same breath as the refusal?

??? note "I2 rubric, model answer and wrong answers (brownfield)"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Establishes that no existing key may notice, and makes that the first design constraint | Designs the three tiers first and handles migration later |
    | Trade-offs | Ships metering before pricing, because you cannot price what you have not measured | Picks a limiter algorithm as the opening move |
    | Depth | Names the fixed-window boundary effect with a number, and connects the quota headers to what a client does at 3am | Says "use a token bucket" with no burst analysis |
    | Communication | States explicitly what is not changing, and in what order the three pieces ship | Presents a target architecture with no sequence |

    Model answer, in shipping order.

    First, the constraint that dominates everything: 9,000 existing keys must see identical behaviour on the day this ships. So every existing key is grandfathered onto a `legacy` plan whose limits are exactly today's limits. Plans become a resource attached to the key, and the default for a key with no plan is `legacy`. Nothing else about tiering can be designed until that is true.

    Second, meter before you price. Ship read-only metering for a full billing cycle: counted calls per key per operation, emitted as an event stream, aggregated daily, exposed to the customer before it is ever charged for. You cannot set a tier boundary without the distribution of current usage, and you cannot defend the first invoice without the customer having seen the counter for a month first. This is also where you discover that one key is 40 percent of the expensive endpoint.

    Third, the limiter. The current fixed window at the minute boundary has a named failure: every one of 9,000 clients resets at the same instant, so they burst together. That is [thundering herd](index.md#the-failure-library), and the measurement that shows it is the ratio of the peak second to the mean second inside a minute. A token bucket or a sliding window removes the shared boundary; a token bucket also lets you sell burst capacity as a tier feature, which is the reason to prefer it here.

    Fourth, the headers, from day one and on every response including successes: the limit, the remaining allowance, the reset instant, and on a 429 a `Retry-After`. Header semantics are a contract, so shipping them before the tiers exist means the tier launch changes numbers rather than shapes. A 429 without `Retry-After` guarantees a retry storm from well-meaning clients.

    Where the limit lives: the volumetric part at the edge, keyed on the API key, and the plan-aware part at the service, because plan, balance and per-tenant quota are business state the edge does not hold. That is the split in the [trade-off atlas](index.md#the-trade-off-atlas), and both halves are needed here.

    What I am not changing: no resource shapes, no status codes on existing operations, no authentication. The only new failure a legacy key can encounter is one it could already encounter.

    Wrong answer: "Add plans, set the new limits, and email everyone about the change." Reveals the brownfield constraint being treated as a communications problem. Some fraction of 9,000 keys is a script written by a contractor who left, and its behaviour on a new 429 is unknown and untested.

    Wrong answer: "Use a distributed rate limiter with exact global counting across all edge nodes." Reveals coordination bought without a reason. Exact counting costs a synchronous coordination step on every request to enforce a limit whose business purpose tolerates a few percent of error. Approximate per-node budgets with periodic reconciliation are the normal answer, and the exception is a hard legal or financial cap.

    Wrong answer: "Bill from the load balancer logs." Reveals metering treated as an analytics problem. Logs are sampled, dropped under load and not durable enough to invoice from, and the first billing dispute is unanswerable. Metering that produces money is a durable event stream with the same care as a ledger.

    Compare your process: did you protect the 9,000 existing keys before you designed anything new?

??? note "I3 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Notices that green dashboards measure what you answered, not what the client experienced | Takes the p99 at face value and looks for a client bug |
    | Trade-offs | Separates the candidate causes with a distinguishing signal for each | Concludes "network problem" and proposes retries |
    | Depth | Proposes deadline propagation and a retry budget, and names what each prevents | Proposes a timeout increase |
    | Communication | Gives the partner something actionable today (a correlation identifier) as well as a fix | Explains internal metrics to an external customer |

    Model answer: the first thing to say is that a p99 measured on requests you answered cannot see the requests you never answered. Timeouts, connection queue time and abandoned requests are usually outside that number, so the dashboard and the tickets can both be honest.

    Candidate causes, each with a signal that separates it:

    - **Client timeout below your tail.** Their timeout may be 100 milliseconds against a 140 millisecond p99 and a much worse p99.9. Signal: the reported failure rate matches the fraction of your distribution above their configured timeout, and the failures are spread evenly rather than clustered.
    - **Retry storm.** Their client retries three times, your gateway retries twice, so one logical call becomes six. Signal: your request count per logical operation exceeds one, and load rises while success falls. This is straight out of the [failure library](index.md#the-failure-library) and the fix is one retry layer with a budget, not a bigger pool.
    - **Head-of-line blocking on a shared connection pool.** One slow downstream call holds connections that fast requests then queue behind. Signal: queue wait time rises while service time does not.
    - **No deadline propagation.** You keep working on a request the client abandoned, spending capacity on answers nobody will read. Signal: work completing successfully after the client's connection is gone.
    - **Gray failure on one instance.** One host passes health checks and serves slowly, so it stays in rotation. Signal: latency by instance, not in aggregate, shows one outlier.

    What I would change: measure at the edge including queue and connection time, publish a per-operation objective rather than a service-wide one, propagate a deadline from the inbound request into every downstream call, cap retries as a fraction of base traffic, and return a `request_id` on every response and every error so a partner can quote one instead of describing symptoms. The last item is the cheapest and it usually shortens the next ticket from days to minutes.

    Wrong answer: "Ask the partners to increase their timeout." Reveals a fix that moves the cost to the customer and hides the tail. It also makes a future incident worse, because longer client timeouts mean more concurrent requests held during a slowdown.

    Wrong answer: "Add retries on our side so transient failures are absorbed." Reveals retries added without a budget, which is the direct route to the storm that may already be happening.

    Wrong answer: "The dashboard is green, so this is a partner problem." Reveals a service level defined by what is convenient to measure. The objective the customer cares about is measured at their edge, and a per-operation indicator that excludes timeouts is describing your comfort rather than their experience.

    Compare your process: did you question what the dashboard measures before you questioned the partners?

??? note "I4 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks what the partner does with the output, because that decides what the contract may promise | Designs a request and response shape immediately |
    | Trade-offs | Refuses to expose raw scores and gives the reason in terms of cross-version comparability | Exposes scores because they are available |
    | Depth | Treats a retrain as a compatibility event with the same apparatus as a schema change | Treats the model as an implementation detail |
    | Communication | States the determinism guarantee in one sentence a partner can build against | Leaves determinism unstated |

    Model answer: the contract has to survive the model changing weekly, so the first decision is what the response is allowed to mean.

    - **Ranks, not scores.** A score is comparable only within one model version. If I publish scores, partners will threshold them, cache them, and compare last week's to this week's, and the first retrain silently changes what their threshold means. I publish an ordered list plus an opaque `model_version` on every response. If a partner genuinely needs a magnitude, it is a calibrated probability with a documented meaning, which is a much larger commitment and is priced accordingly.
    - **Determinism, stated precisely.** Same input, same model version, same output. Nothing is promised across versions. That single sentence is the whole thing partners need, and almost nobody writes it down.
    - **A retrain is a compatibility event.** Not a breaking change to the schema, but a change to behaviour that clients depend on. It gets a version stamp, a changelog entry, an opt-in window on a pinned version for partners who need to re-certify, and a maximum pin duration so pinning does not become a second product I maintain forever. This is the same apparatus as B6, applied to behaviour rather than to fields.
    - **Explanation fields are a permanent commitment.** If I return reasons, partners build user interfaces on them, and the reason taxonomy is now frozen against model changes that would alter it. Ship reasons only if the taxonomy is stable independently of the model.
    - **A degradation path is a contract decision.** When the model is unavailable I can return 503, or a popularity ordering with a flag saying it is a fallback. The second is usually better for the partner and it must be declared, because an undeclared fallback is indistinguishable from a bad model.
    - **Metering.** Ranking calls cost real compute, so the call is counted, the count is visible to the partner, and the quota is per key. The design lives in [ML System Design](ml-system-design.md) for the model side, and here for the surface.

    Wrong answer: "Return the raw scores, the partner can decide what to do with them." Reveals an internal representation exported without a stability promise. It is the interface equivalent of publishing a database column, and every retrain becomes an incident you did not schedule.

    Wrong answer: "Version the endpoint per model, `/v1/rank`, `/v2/rank` weekly." Reveals versioning applied without a cost model. Fifty-two live versions a year, each of which is a model artefact you must keep serving, is a commitment nobody would sign if it were stated in those terms.

    Wrong answer: "It is a model, so results vary. We will say so in the documentation." Reveals a guarantee replaced by a disclaimer. Partners still need to know whether identical input gives identical output within a version, because caching, testing and support all depend on it.

    Compare your process: did you ask what the partner does with the ordering before you designed the response?

??? note "I5 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Recognises that the system is behaving as designed and the failure is in the documented guarantee | Opens an investigation into the dispatcher |
    | Trade-offs | Prices ordered delivery per destination and shows it does not fit | Promises ordering to close the ticket |
    | Depth | Gives the consumer a stable event identifier plus a per-object version so stale updates can be dropped | Says "consumers should be idempotent" with no mechanism |
    | Communication | Answers the partner with a guarantee statement, not with an apology | Escalates internally without changing the contract |

    Model answer: nothing is broken. At-least-once delivery with retries produces duplicates by construction, and parallel delivery workers plus per-attempt retries reorder by construction. The green dashboard is correct. What failed is that the guarantee was never written down, so the partner inferred a stronger one.

    What I owe them, and would ship:

    - **A written guarantee.** At-least-once, no global ordering, delivered within a stated window under normal conditions, retried for a stated horizon. One paragraph, in the documentation, versioned.
    - **A stable `event_id`**, identical across every redelivery of the same event, so a consumer can deduplicate with a single unique index on their side. Without this the advice to be idempotent is not actionable.
    - **A per-object monotonic `sequence` plus the resource version in the payload**, so a consumer receiving an older update after a newer one can drop it. Ordering then becomes the consumer's cheap local decision rather than my expensive global promise.
    - **A catch-up read**, so a consumer who suspects a gap fetches from a cursor rather than opening a ticket.

    Why I will not promise ordering: ordered delivery per destination means one in-flight delivery at a time to that destination. At 200 milliseconds per delivery that is 5 events per second per destination. A partner generating 50 events per second would accumulate 45 per second of backlog, which is 162,000 an hour, so the promise fails on the first busy day and fails invisibly. If one partner truly needs ordering, it is per object rather than global, it is priced, and it is capped.

    Wrong answer: "Add a deduplication cache on our side so we only send each event once." Reveals a guarantee promised across a boundary that cannot provide it. You cannot know whether a delivery was received; you only know whether you got a response. Deduplication belongs on the side that can observe the effect.

    Wrong answer: "Switch to exactly-once delivery." Reveals a phrase used as a design. Across an HTTP boundary to a consumer you do not operate, the achievable pair is at-least-once plus an idempotent consumer, which is exactly the atlas row on delivery semantics.

    Wrong answer: "Tell them to check the timestamp and ignore older events." Reveals a fix that depends on two machines agreeing about time. Clock skew is in the failure library for a reason, and a per-object sequence you generate is free and correct.

    Compare your process: did you check what you had promised before you checked what you had built?

??? note "I6 rubric, model answer and wrong answers (brownfield)"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Reframes the question as "what has already leaked into the contract" | Designs the partitioning scheme |
    | Trade-offs | Identifies which leaks are compatible to fix now and breaking to fix later | Treats all the changes as equally urgent |
    | Depth | Names identifier format and cursor opacity as the two things to change this quarter | Names only the query changes |
    | Communication | Gives a sequenced plan whose whole purpose is that the answer becomes "nothing" | Lists impacts without an order |

    Model answer: the goal is that the answer next quarter is "nothing changed", and the work to make that true happens now, because every item on the list is compatible today and breaking after the split.

    What has leaked, and what it costs later:

    | Leak | Why it breaks after the split | Fix now, compatible |
    |---|---|---|
    | Sequential integer identifiers | A global sequence across partitions is a coordination point, and per-partition sequences collide | Issue opaque string identifiers now, accept both on input, stop publishing the integer |
    | Readable cursors encoding an offset or a global position | There is no global position after the split | Make cursors opaque and signed now, and say in writing that they are opaque |
    | Cross-tenant list endpoints with a global sort | Becomes a scatter-gather with a merge, and page depth becomes expensive | Require a tenant scope on list operations now, while it can be added as a default rather than a requirement |
    | Batch endpoints accepting items from several tenants in one transaction | The transaction now spans partitions | Give the batch partial-success semantics now, as in B3, which is a compatible addition |
    | An exact `total` on list responses | A count becomes a fan-out | Replace with `has_more` now, deprecate `total` on the schedule from B6 |
    | Read-your-writes across tenants | Different partitions, different replication lag | Document the scope of the read-your-writes promise now, before the promise gets narrower |

    Sequencing: identifiers and cursors first, because both need a full deprecation window and both are currently cheap. Then the list scoping and the batch semantics, which are additive. Then the count. The partitioning work itself is a [Modern System Design](modern-system-design.md) problem and I would not spend interview minutes on the hash-against-range choice unless asked; the contract question is entirely about what the old shape promised that the new one cannot keep. For the storage-side background, the [system design question bank](/Interview-Questions/System-design/) covers the partitioning trade directly.

    Wrong answer: "Nothing changes, partitioning is an implementation detail." Reveals a boundary assumed rather than checked. Sequential identifiers, global sorts and exact counts are all implementation details that have already become promises.

    Wrong answer: "We will version the interface at the same time as the split." Reveals two large risks coupled into one release. The partition migration and a version migration have independent failure modes and independent rollbacks, and combining them means neither can be rolled back alone.

    Wrong answer: "Add a tenant identifier to every path now and be done." Reveals the easy half of the answer. Path scoping does not fix opaque identifiers, cursor contents, counts or the batch transaction boundary, and those are the ones that are compatible today and breaking later.

    Compare your process: did you list what the old contract promised, or what the new database looks like?

---

## 12. Mastery check and remediation map

Seven short-answer items, one for each learning objective in Section 4, in the same order. Objective 1 is tested by M1, objective 2 by M2, and so on to objective 7 and M7. Write or say your answer before opening the block, because producing the answer is what builds the retrieval path and recognising a correct one does not.

**M1** (Objective 1, derive a resource model and locate the transaction boundary). Domain: an organisation buys a subscription with a fixed number of seats. Members occupy seats, seats can be reassigned between members, and the subscription bills monthly in advance. List the entities, draw the aggregate boundaries, state at least three invariants, and name the single invariant that decides where the transaction boundary sits.

??? note "Show answer"

    **Entities:** Organisation, Subscription, Plan, Seat, Member, Assignment (a member occupying a seat over an interval), Invoice, Payment method.

    **Aggregates:** three.

    | Aggregate | Root | Contains | Why |
    |---|---|---|---|
    | Subscription | Subscription | Seat count, seats, current assignments, plan reference | The seat invariant spans the subscription and its assignments |
    | Member | Member | Identity, contact details, organisation reference | Edited on its own schedule and referenced by identifier, never contained |
    | Billing | Invoice | Line items, amounts, period, payment attempts | Fails and retries independently, and settles after the fact |

    **Invariants:**

    1. The count of active assignments never exceeds the subscription's seat count.
    2. A member holds at most one seat in a given subscription.
    3. Assignment intervals for one seat never overlap, so the seat's history is a partition of time.
    4. An invoice line for a period never covers a seat count higher than the count in force during that period.
    5. Reducing the seat count is refused while assignments exceed the new count, rather than silently evicting someone.

    **The invariant that decides the boundary: number 1.** Assigning a seat writes the assignment row and reads or updates the subscription's seat usage, and both must be true at the same instant, which puts the transaction on the Subscription aggregate and makes Subscription the root.

    Why the others do not move it:

    - Number 2 is a unique index inside the same aggregate.
    - Number 3 is an exclusion constraint on one seat, still inside the aggregate.
    - Number 4 spans Subscription and Billing, which are separate aggregates, so it is enforced by recording the seat count on the invoice line at generation time rather than by a transaction across both.
    - Number 5 is a precondition check, not an atomicity requirement.

    The sentence that reads as senior: "Billing is deliberately outside the transaction, because an invoice that fails must not roll back a seat assignment a customer has already been told about."

**M2** (Objective 2, choose the exposure style). An operation recalculates a customer's loyalty tier. The checkout path calls it about 900 times a second from two internal services on your release train. A partner customer-relationship system needs each customer's tier once a day, for about 2 million customers, and there are 60 such partners. Choose the exposure or exposures, cite the quantities that decide it, and name one condition under which exposing this domain two ways is correct rather than lazy.

??? note "Show answer"

    **The quantities that decide it:**

    | Quantity | Internal | Partner |
    |---|---|---|
    | Clients you can redeploy | 2 | 0 |
    | Call rate | 900 per second, synchronous, inside a latency budget | 2 million per day per partner, tolerant of hours |
    | Coupling cost of a change | Two pull requests | A deprecation programme |

    **Choice.** Three exposures over one domain service, and each is justified by a different quantity.

    - A procedure-style call for the checkout path. Two clients you control, latency-sensitive, high rate, a schema both ends compile.
    - A representational read for partners: `GET /customers/{id}/tier` for single lookups.
    - An event stream, `tier.changed`, for the daily bulk case. Two million individual reads per partner per day is 23 per second per partner sustained, and 60 partners is 1,380 per second of polling to observe changes that affect a small fraction of customers. The event stream turns that into one delivery per actual change, which is the number that eliminates polling.

    **The condition that makes multiple exposures correct.** Different client populations with different upgrade rights and different freshness needs, where each exposure is a thin projection over one domain service and none of them contains a tier rule. Adding a rule to only one is what turns three front doors into three products.

    **The condition that makes it lazy.** Three exposures for one population, or a second exposure created because a team preferred different tooling. That is a maintenance tax with nobody on the other end.

**M3** (Objective 3, design the error model). For an interface with external clients, give the error taxonomy, the retryability signal, and the partial-failure semantics for a batch operation, such that a client can branch on every class without parsing a human message. Then name the one thing in your error body that a client must never branch on.

??? note "Show answer"

    **Taxonomy.** Every error carries a stable machine-readable code. The status class tells an intermediary what happened; the code tells the client what to do.

    | Class | Status | Retryable | What a correct client does |
    |---|---|---|---|
    | Malformed request | 400 | No | Fix the payload. Never retry |
    | Unauthenticated | 401 | Once, after refreshing | Refresh the token, retry exactly once, then stop |
    | Forbidden | 403 | No | Surface to a human. A retry cannot change permissions |
    | Not found | 404 | No | Stop, unless the resource is known to be newly created and eventually visible, which the body must say |
    | Conflict with current state | 409 | Only if the body says so | Read the current state, decide, then act. An identical replay conflicts again |
    | Precondition failed | 412 | After re-reading | Re-fetch, re-apply, resubmit with the new validator |
    | Semantically invalid | 422 | No | Fix the data |
    | Rate limited | 429 | Yes | Wait for `Retry-After`, then retry with jitter |
    | Internal error | 500 | Yes | Bounded retries with full jitter |
    | Unavailable | 503 | Yes | Wait for `Retry-After` if present, otherwise back off |
    | Upstream timeout | 504 | Only if the operation is idempotent or carries a key | Retry with the same idempotency key, never without one |

    **The retryability signal.** An explicit boolean field plus an optional delay, not an inference from the status. Two errors in the same 409 class can differ: a conflicting concurrent write is worth retrying after a re-read, and a reused idempotency key with a changed body never is. The status cannot express that difference, so the body must.

    **Partial failure in a batch.** Envelope-level failures (bad token, malformed body, over the item limit, over quota) get a 4xx and no item results. Item-level outcomes ride inside a 200 with a results array carrying the submitted index, a per-item status, a stable code and the per-item retryable flag. The client resubmits only the retryable subset, under the same idempotency key.

    **What a client must never branch on: the human message.** It is for a developer reading a log, it is subject to translation, wording changes and length limits, and every change to it becomes a breaking change the moment a client parses it. Say so in the documentation, and keep the code stable even when you rewrite the sentence.

    !!! warning "When not to build a full taxonomy"

        An internal service with two callers in your own repository does not need eleven codes and a retryability field. Return the status, a short code and a message, and spend the time elsewhere. The measurement that justifies the full model is the first client you cannot patch the same day, or the first support ticket whose answer is "what does that error mean".

**M4** (Objective 4, specify idempotent creation). For a write that must not happen twice, specify the key, its lifetime, the request fingerprint and the transaction boundary. Then name the exact retry sequence that still duplicates if the boundary is drawn one line too late, and say which of the two ordering mistakes is worse.

??? note "Show answer"

    **Key:** client-supplied, opaque, unique per (tenant, key), enforced by a unique index. Never a global key space, because collisions across tenants are both a correctness bug and an information leak.

    **Lifetime:** derived from the longest replay the client can produce, not chosen for tidiness. Take the client's retry burst (seconds) and the slowest automated replay (a nightly or weekly reconciliation job), and set the window beyond the larger one. A daily job means the window must exceed 24 hours, and 48 gives room for a late run.

    **Fingerprint:** a hash of the operation, the key, the authenticated principal and a canonicalised body, stored with the key. Match plus completed effect replays the stored response. Match plus in-flight returns 409 with a retry delay. Mismatch is refused with a distinct code and never processed.

    **Boundary:** the key row and the effect row commit in one transaction. Not two transactions, not a cache plus a database.

    **The sequence that still duplicates, when the key is written after the effect:**

    ```
    t=0.00  Request A arrives with key K
    t=0.05  effect committed
    t=0.06  process dies before the key row is written
    t=1.00  client times out, retries with key K
    t=1.05  no key found, second effect committed
    Result: two effects, one key, no way to detect it later
    Caption: the key ends up protecting the duplicate, not
    the original.
    ```

    **The mirror image, when the key is committed first, is worse.** Key committed, effect fails, client retries, and the client receives a replayed success for something that never happened. The first mistake produces a duplicate that reconciliation can find, because two effects exist and both are visible. The second produces a claim with no effect behind it, which nothing on your side can detect, and which surfaces weeks later as a customer dispute.

    **Neither ordering handles concurrency**, which is the third answer worth volunteering: two requests with the same key a millisecond apart both find nothing, and the unique index makes one lose. That is why the key record needs a reserved state answering 409 rather than a raw constraint error or a row lock that holds a connection.

**M5** (Objective 5, pagination under concurrent writes). Compare offset, cursor and keyset pagination on a collection receiving concurrent inserts and deletes. Name the anomaly each produces, state what a correct cursor encodes, and give the method for computing the page depth at which offset stops being defensible.

??? note "Show answer"

    | Style | Anomaly under concurrent writes | Why |
    |---|---|---|
    | Offset | Duplicates and skips | An insert ahead of the window shifts every later row forward, so the next page repeats rows. A delete behind the window shifts rows back, so rows are never returned |
    | Cursor over a non-unique sort key | Duplicates and skips at ties, silently | Resuming "after value V" cannot distinguish rows sharing V, so it either repeats them or drops them |
    | Cursor or keyset over a mutable sort column | A row is missed entirely or returned twice | An edit moves the row across the cursor position while you are paging |
    | Keyset over an immutable sort key plus a unique tiebreaker | New rows inserted ahead of the cursor are absent from this pass | Stable and explainable. It is the correct default |

    **What the cursor encodes:** the sort key values of the last row returned, a unique tiebreaker (usually the row identifier), the sort direction, and a fingerprint of the filters that produced the result set. Opaque, and documented as opaque, otherwise clients parse it and its format becomes part of your contract.

    **The method for the offset depth.** Two inputs and one division.

    1. Measure or assume the cost of skipping one index entry.
    2. State the read budget for a page.
    3. Divide the budget by the per-row skip cost to get the maximum discarded rows, then divide by the page size to get the page number.

    Worked with the page numbers from B5: at 0.5 microseconds per skipped entry and a 20 millisecond budget, that is 40,000 rows, which at 50 per page is page 800. Beyond it, the skip alone exceeds the budget before a useful row is read. The comparison that settles the argument: a keyset seek on 40 million rows is about 25 index comparisons and is the same cost on page 800 as on page 1.

    **The frequency, which is what makes it real.** With 6 inserts a second and a 3 second page turn, the window shifts by 18 rows, so 36 percent of a 50-row page repeats. This is not a rare race, it is the normal behaviour of a live feed.

**M6** (Objective 6, plan a breaking change). For clients you cannot force to upgrade, give the compatibility classification, the signalling mechanism, the migration order, and the measurement that ends the deprecation window. Name the one classification most candidates get backwards.

??? note "Show answer"

    **Classification.**

    | Change | Class |
    |---|---|
    | Adding an optional response field | Compatible, provided clients were told in writing to ignore unknown fields |
    | Adding an optional request field with a default | Compatible |
    | Adding a value to a request enum | Compatible: you are widening what you accept |
    | Adding a value to a response enum | Breaking in practice |
    | Removing or renaming a field | Breaking |
    | Changing a type, a unit or a precision | Breaking, and the worst kind, because it parses successfully |
    | Tightening validation on an existing field | Breaking |
    | Changing a default | Breaking for clients relying on the old one, and invisible in your logs |
    | Making an optional field required | Breaking |

    **The one most candidates get backwards:** adding a value to a response enum. It feels additive and therefore safe. Generated clients compile closed sets, so a new value throws at parse time in code you did not write and cannot see. The rule to state: response enums are frozen unless every client was told in writing to treat unknown values as a default branch, and you can show that they do.

    **Signalling.** A `Sunset` header carrying the date on every affected response ([RFC 8594](https://www.rfc-editor.org/rfc/rfc8594.html), May 2019, accessed August 2026), a deprecation marker in the schema so generated clients warn at build time, and a per-key notice that names what that key specifically calls. Not a blog post, and not a newsletter.

    **Migration order.** Measure field-level usage per key. Expand, adding the new shape alongside the old. Accept both on input, prefer the new. Announce per key with their own numbers attached. Default new keys to the new shape, which closes the population. Narrow.

    **The measurement that ends the window.** Deprecated calls per day weighted by the commercial weight of the keys making them, the count of keys at zero deprecated calls for 30 consecutive days, and a contact list with a named owner per remaining key. The window ends when the remaining population is small enough to talk to individually, not when a pre-chosen date arrives. A date with no measurement leaves you choosing between breaking clients and extending publicly, and the second is worse than never having announced a date.

**M7** (Objective 7, mid to staff). Name the four additions that convert a mid-level contract answer into a staff-level one on this page, and say what each one changes about a decision rather than what it adds to the diagram.

??? note "Show answer"

    - **A deprecation path.** Changes the shape of version one. Once you commit to migrating clients you cannot redeploy, you put the non-terminal state, the opaque identifier and the open enum into the first release, because each is compatible today and breaking later. It moves compatibility from a future problem to a design input.
    - **A quota and metering consequence.** Changes what the interface is allowed to offer. A call that costs real compute gets counted, and the counter is visible to the customer before it is charged for. It also decides granularity: an operation you cannot meter cheaply is an operation you cannot price, which sometimes changes the operation.
    - **A webhook delivery guarantee.** Changes what consumers must build. Saying at-least-once with no global ordering forces you to ship a stable event identifier and a per-object sequence, and it forecloses a promise you could not have kept. It converts a support conversation into a documented contract.
    - **A blast-radius statement.** Changes the failure design. Naming who is affected by one bad deploy, one poisoned key or one dead consumer endpoint is what produces per-destination queues, per-tenant limits and a degradation path, rather than a single shared pool that fails for everyone at once.

    Each of the four turns a claim into a commitment somebody can be held to. That, rather than extra components, is what the level difference is made of.

**Threshold.** Five of seven, with M4 and M6 among them. Those two are weighted because a candidate who cannot specify idempotency to the transaction line, or cannot end a deprecation on a measurement, has described an interface rather than designed one, and both are the probes senior interviewers reach for first.

| Missed item | Reread | Redo |
|---|---|---|
| M1 | Module 3 on domain modelling, plus the requirements and V0 blocks of the payments and comment threading case studies | Problem B1, then model a domain of your own in ten minutes with a timer, ending on the invariant that sets the boundary |
| M2 | Module 4 on resource, procedure or event, Module 5 on style selection, and the style row of the [trade-off atlas](index.md#the-trade-off-atlas) | Problem B2, then redo it with the partner count changed to two and see which decisions flip |
| M3 | Module 7 on error model design, and Module 16 on batch partial success | Problem B3, then write the taxonomy table from memory and check it against the reference block |
| M4 | Module 9 on idempotency, and deep dive one of the payments case study | Problem B4, then write both failing timelines from memory with timestamps |
| M5 | Module 8 on pagination, and the pagination deep dive in the search service case study | Problem B5, then recompute the offset depth with a 5 millisecond budget and a 200 row page |
| M6 | Module 11 on versioning and deprecation, and the brownfield case study end to end | Problem B6, then problem I6, and diff the two lists of what leaked into the contract |
| M7 | Module 21 on level calibration, plus the level delta block of any two case studies | Problem I2 at mid level, then again at staff level, and annotate line by line what the second one added |

!!! warning "Scoring well right now is a weak signal"

    Passing this immediately after reading mostly measures that the page is still in working memory. The signal that predicts interview performance is the delayed check in Section 15, taken a week later with the page closed. If you score seven of seven today and four of seven next Tuesday, the honest reading of your ability is four.

---

## 13. Common mistakes

Technical mistakes cost you a follow-up. Delivery mistakes cost you the round. The last five rows are delivery, they appear in debrief notes constantly, and they are the cheapest thing on this page to fix.

| The mistake | Why candidates make it | What the interviewer concludes | What to say instead |
|---|---|---|---|
| Listing endpoint paths before naming an entity | Paths are the visible part of an interface and they feel like progress | Guesses interfaces rather than deriving them, and will invent a resource for every screen | "Before any paths: the entities are Session, Seat and Learner, the aggregate boundary is around Session because capacity is the invariant, and the operations fall out of that" |
| Designing only the happy path | The happy path is what a tutorial shows, and it is what the prompt literally asked for | Has never operated a surface where an automated client retried at 3am | "Here is the create call. Now the three things that go wrong with it: a duplicate arrival, a partial batch, and a stale write. Each has a code and a retry rule" |
| Returning 400 with a human message and nothing else | It is what most internal services do, so it looks normal | Clients of this interface will parse English to make decisions | "Every error carries a stable code and a retryable boolean. The message is for a developer reading a log, and I would document that clients must not branch on it" |
| Treating an idempotency key as a header you accept | The header is the visible part, and the storage is not | Has read about idempotency and never implemented it | "The key is a row with a fingerprint and three states, written in the same transaction as the effect. Here is the retry sequence that duplicates if it is not" |
| Claiming exactly-once across a network boundary | The phrase is common and sounds like a stronger guarantee | Does not know which guarantees are achievable, which is dangerous in a payments or messaging context | "At-least-once delivery plus an idempotent consumer. I will ship a stable event identifier so their deduplication is a unique index, and I will write the guarantee down" |
| Paginating with offset on a live feed | It is what the database tutorial showed and it works in development | Will ship duplicate and missing rows and treat the bug reports as a mystery | "Six inserts a second across a three second page turn repeats 36 percent of the next page. Keyset with a tiebreaker, and the cursor is opaque" |
| Versioning as a prefix with no migration plan | The prefix is the part that appears in documentation | Will accumulate live versions and never retire one | "The prefix is the cheap part. The plan is measure, expand, dual-serve, announce per key, close the population, narrow, and here is the measurement that ends it" |
| Adding a value to a response enum and calling it additive | Additive is a real category and this feels like it | Will break generated clients and will not find out from their own logs | "Adding to a request enum is safe. Adding to a response enum breaks closed parsers, so response enums are frozen unless every client was told to default on unknown values" |
| Skipping quotas because the prompt did not mention money | Quotas feel like an operations concern rather than a design one | Cannot connect an interface to its bill or to fair use between tenants | "This endpoint is the expensive one, so it is metered per key and counted before it is charged for, and the limit is expressed in headers on every response" |
| Designing before clarifying (delivery) | Silence feels like failure and drawing feels like progress | Will build the wrong thing quickly and confidently | "Two questions first, because they change the shape: who are the consumers, and which of them can I not redeploy?" |
| Reciting a framework's phases out loud (delivery) | A memorised spine feels like structure under pressure | Recited an acronym rather than reasoning, which interviewers who run this round weekly screen for | Announce the decision, never the phase: "I am putting the transaction on Session, because capacity is the invariant that spans two rows" |
| Monologuing through the operation list (delivery) | Reciting a known sequence feels safe and fills the time | Cannot collaborate, and gave the interviewer no opening to steer | "That is the write path. Do you want the error semantics next, or the evolution story?" |
| Defending a choice instead of updating it (delivery) | Reading a follow-up as an attack rather than as information | Will not update a design in a review either, which is expensive on a real team | "You are right that a client can reuse a key with a different body. That changes the fingerprint from an optimisation to a requirement. Let me redo the replay rules" |
| Running out of time before saying anything about evolution (delivery) | The resource model is the comfortable part and it expands to fill the hour | No evidence the candidate has ever lived with an interface after shipping it | At minute 32: "I want eight minutes for evolution and failure behaviour, so I am stopping the model here and will come back if there is time" |

---

## 14. Interview-day checklist

One screen. Print it or keep it in a second window. Nothing here is new.

**Minute budget**

| Phase | 45 minute round | 60 minute round |
|---|---|---|
| Restate as a domain with named consumers | 0 to 2 | 0 to 2 |
| Clarify, including who cannot be upgraded | 2 to 8 | 2 to 9 |
| Domain model: entities, aggregates, invariants | 8 to 16 | 9 to 19 |
| Contract v0: fewest operations, one happy path | 16 to 24 | 19 to 28 |
| Errors, retries, idempotency, pagination | 24 to 32 | 28 to 38 |
| Evolution: classification, signalling, migration | folded into the dive | 38 to 46 |
| One deep dive (two if 60 minutes) | 32 to 40 | 46 to 56 |
| Close: trade-offs, what you skipped, what you would measure | 40 to 45 | 56 to 60 |

**The first four sentences**

1. "Let me restate it as a domain: the nouns are X and Y, the operations people care about are A and B, and the consumers are Z."
2. "I will spend about eight minutes on the model and the invariants before I write a path, because the operations fall out of that."
3. "I will treat error and retry semantics as part of the design rather than as a detail at the end."
4. "Stop me if you want depth somewhere specific rather than coverage."

**Five clarifying questions that generalise across prompts**

1. Who are the consumers, and which of them can I not redeploy on my own schedule?
2. What already exists and is public, and what has someone already promised will never change?
3. Does the caller need the outcome in the response, or is an acknowledgement enough?
4. Do clients retry automatically, and on what?
5. What must never happen twice, and what must never be lost?

**Drawing order**

```
DRAW IN THIS ORDER. WITHHOLD BOXES UNTIL A NUMBER FORCES ONE.

    1                    2                    3
 +--------------+   +---------------+   +---------------+
 | ENTITIES     |   | OPERATIONS    |   | ERROR AND     |
 | and the      |-> | fewest that   |-> | RETRY RULES   |
 | aggregate    |   | satisfy the   |   | per operation |
 | boundary     |   | requirements  |   |               |
 +--------------+   +---------------+   +---------------+
                                                |
                                                v
                                        +---------------+
                                        | 4, EVOLUTION  |
                                        | what breaks   |
                                        | a live client |
                                        +---------------+

 NOT YET: service boxes, databases, caches, queues, gateways,
 load balancers. Add one only when a number you stated needs it.
 Caption: model first, then operations, then semantics, then
 evolution. Infrastructure is the last thing you draw in this
 round, not the first.
```

Write the operations as a table with columns Operation, Request, Response, Errors, Retry-safe. That table is the artefact the interviewer remembers, and it is the one that makes the error column impossible to skip. Erasing a path you invented in minute four reads as guessing; adding one in minute twenty-two because an invariant demanded it reads as design.

**Three recovery scripts**

| Situation | Say this |
|---|---|
| You are stuck | "Let me state the invariant I am defending and the two ways to defend it. A puts it in one transaction and costs contention on one row. B accepts a window and needs a repair path. I take A because the invariant is about money, and I will flag it as reversible if contention shows up." |
| The interviewer disagrees | "That is a case I did not price. If a client reuses a key with a different body, my replay rule is wrong, not just incomplete. Let me change the fingerprint from optional to required and redo the table." |
| You are running out of time | "I have about six minutes. I would rather spend them on what a breaking change looks like here than on finishing this dive, because I think that is the higher-value part. Say if you would prefer otherwise." |

**Before you finish**

- State the two trade-offs you actually made and what each one cost.
- State what you deliberately skipped, one sentence each, with the reason it could not change a decision.
- Name the single indicator you would watch in week one and the value that would make you roll back.
- Name the degradation path: what a client receives when a dependency you do not own is unavailable.
- Name the decision that is most expensive to reverse. In this round it is almost always the identifier format, the contents of an enum, or the absence of a non-terminal state.

---

## 15. Study schedule and spaced review

### 15a. Three plans, each defined by what it drops

Day 1 is whatever day you start. The skip column is the hard part to write and the useful part to read: a plan that tells you to do everything is not a plan.

**One week (about 9 to 12 working hours)**

| Day | Do | Skip |
|---|---|---|
| 1 | Modules 1 to 3, then problem B1 | Nothing yet. Module 3 is the one module that changes every later answer |
| 2 | Modules 4 and 5, then problem B2, then the payments case study end to end with a timer | The style comparison beyond representational against procedure. You need the trade, not the taxonomy |
| 3 | Modules 6 and 7, then problem B3 | Module 6's generation and linting tooling detail. Read the compatibility rules and move on |
| 4 | Modules 8 and 9, then problems B4 and B5 | Module 10 on entity tags, unless your target surface is edit-heavy |
| 5 | Module 11, then problem B6, then the brownfield case study | Modules 12 and 13, unless your loop is platform or developer-experience shaped |
| 6 | Modules 15 and 20, then problems I3 and I5 with a timer | Modules 14, 16, 17 and 18. Each is real and none of them is where you lose the round |
| 7 | Module 21, then Section 12, then Section 14. One timed self-mock against the master rubric | Everything unreached. Do not start new material the day before a round |

**One weekend (about 5 to 7 working hours)**

- Saturday morning: Modules 3, 7 and 9. Domain modelling, error models and idempotency. These three change more answers per minute read than anything else on the page.
- Saturday afternoon: one case study end to end, out loud, with a timer. Take payments if you are targeting senior or above, because retries and delivery guarantees are where those rounds go. Take comment threading if you are earlier in your career, because the modelling is the interesting part and the semantics are gentler.
- Sunday morning: Modules 8 and 11 (pagination and versioning), then problems B5, B6 and I1.
- Sunday afternoon: Sections 12, 13 and 14. Rehearse the first four sentences until they are automatic.
- Skip entirely: Modules 1, 2, 5, 6, 10, 12 to 19, every second deep dive, and ten of the eleven case studies. One case study worked properly beats five skimmed, and the difference is visible within two minutes of an interview starting.

**One evening (about 2 to 3 hours)**

- 0:00 to 0:25: Section 14 and Module 1. Get the clock, the opening and the ability to tell which round you are in.
- 0:25 to 1:10: Module 3 and Module 7. The domain model and the error taxonomy are the two fastest visible upgrades to a contract answer, and almost nobody arrives with the second.
- 1:10 to 2:00: one case study, requirements and V0 only, no deep dives, out loud.
- 2:00 to 2:30: Section 13 once, then Section 14 again.
- Skip: every other module, every deep dive, the mastery check, and every practice problem except B4. If your round is tomorrow, extra breadth will not be retrievable under pressure and a rehearsed opening will be.

### 15b. Review schedule

| When | What | Minutes |
|---|---|---|
| Day 1 | The review cards below, once through | 10 |
| Day 3 | Cards, plus redo one blocked problem you got wrong | 20 |
| Day 7 | The delayed self-check in 15d, closed book, then cards | 30 |
| Day 21 | Cards, plus one interleaved problem you have not attempted | 40 |
| Day 60 | One full case study out loud with a timer, then cards | 60 |

Consistency beats the exact intervals by a wide margin. A session on day 4 and another on day 9 is worth far more than a perfect schedule abandoned after the first miss. If you skip a checkpoint, do the next one on the day you remember rather than restarting the sequence.

### 15c. Review cards

One fact or one decision per card, nothing you could not already explain. Copy this block into whatever tool you use. The format is prompt, two colons, answer.

```
Q :: Which invariant decides where the transaction boundary sits?
A :: The one that spans two rows written by the same operation.
     The others become indexes, constraints or state checks.

Q :: What decides resource against procedure against event?
A :: The count of clients you can redeploy, plus rate and
     payload. Not efficiency, which usually decides nothing.

Q :: Why must retryability be a field and not the status code?
A :: Two errors in one class differ. Inside a 200 batch, a bad
     email must never be resent and a storage failure must.

Q :: What does an idempotency key record hold?
A :: Key, request fingerprint, state (absent, reserved,
     completed), and the stored response for replay.

Q :: What sets the idempotency key lifetime?
A :: The client's longest replay horizon. A nightly job means
     the window must exceed 24 hours.

Q :: The retry sequence that duplicates when the key is late?
A :: Effect commits, process dies before the key is written,
     client retries, second effect commits. Two effects, one key.

Q :: Which key ordering mistake is worse, and why?
A :: Key first, then effect. It returns a receipt for something
     that never happened, and nothing on your side detects it.

Q :: How do you find the depth where offset stops being defensible?
A :: Page read budget divided by the per-row skip cost gives the
     rows you may discard, then divide by page size.

Q :: What must a cursor encode?
A :: Last sort values, a unique tiebreaker, the direction, and a
     fingerprint of the filters. Opaque, and documented as opaque.

Q :: Which enum change is breaking, and which is safe?
A :: Adding to a request enum is safe. Adding to or removing
     from a response enum breaks closed parsers.

Q :: What ends a deprecation window?
A :: A measurement, not a date. Deprecated calls weighted by
     client value, plus a contact list with named owners.

Q :: Why can you not promise ordered webhook delivery?
A :: Ordering means one in-flight delivery per destination. At
     200 ms per attempt that is 5 events per second, per client.

Q :: What do you owe a webhook consumer so idempotency is possible?
A :: A stable event id across redeliveries, a per-object
     sequence, and a catch-up read from a cursor.

Q :: What is the first thing to fix before a tenant partition?
A :: Identifier format and cursor opacity. Both are compatible
     changes today and breaking changes afterwards.

Q :: Four things a staff contract answer adds over a mid one?
A :: A deprecation path, a quota and metering consequence, a
     stated delivery guarantee, and a blast-radius statement.
```

### 15d. Delayed self-check

Answer these from memory a week after you finish, with this page closed. Write the answers down before you check anything.

1. Take a domain you have not modelled here, for example a warehouse reserving stock against orders. Give the entities, the aggregate boundaries, three invariants, and the one that decides the transaction boundary.
2. For one write in that domain, specify the idempotency key, its lifetime with the derivation, the fingerprint and the transaction boundary. Then write the retry timeline that duplicates if the boundary is drawn one line too late.
3. Name one change to that interface that is breaking, one that is compatible, and one that most people classify wrong. Then state the measurement that would end its deprecation window.

Twenty minutes, no notes. If you can do all three, the material is retrievable under pressure. If you cannot, the step where you stalled is the one to redo, not the whole page.

---

## 16. Reference block

!!! info "This section teaches nothing new"

    Everything below appeared earlier with its derivation. This is the version to reread the morning of the round. If a row here is the first place you have met an idea, go and read the module it came from rather than memorising the row.

### 16a. Decision tables

**Exposure style**

| If | Choose | Because | Do not choose it when |
|---|---|---|---|
| The surface is public or partner-facing and there are clients you cannot redeploy | Representational resources, coarse and stable | Caching, tooling and a shape that survives clients you never meet | The operation is a verb with no noun behind it, in which case forcing a resource produces a fake one |
| The caller is internal, on your release train, latency-sensitive and high rate | Procedure style with a schema both ends compile | A breaking change costs pull requests rather than a quarter | Any consumer sits outside your deploy boundary, at which point you have exported your internal model |
| The producer must not know its consumers and consumers may lag | Events | The producer's release cadence stops being everyone's problem | The caller needs an answer, where events produce a correlation-identifier maze |
| Clients need many different field subsets from one graph and you control the query cost budget | Graph style | Removes over-fetching for genuinely varied clients | Consumers are external and unbounded, where query cost becomes their choice and your bill |
| Two populations with different upgrade rights over one domain | Two thin exposures over one domain service | Each population gets the coupling it can afford | A rule would live in only one adapter, which makes two products |

**Pagination**

| If | Choose | Anomaly you accept | Below this it is over-engineering |
|---|---|---|---|
| Bounded collection, under roughly a thousand rows, product wants numbered pages | Offset | Duplicates and skips if it mutates, which at this size is usually visible and tolerable | Never over-engineering. Cursors here remove a product feature |
| Live collection with concurrent inserts, sorted on an immutable key | Keyset with a unique tiebreaker | New rows ahead of the cursor are absent from this pass | Under the offset depth you derived, offset still works and is simpler |
| Sort column can be edited | Keyset over an immutable key plus a separate filter | You give up sorting by the mutable column, or you accept missed rows | Where the sort column is genuinely immutable, this restriction costs you nothing |
| Client needs to resume days later | Cursor with an explicit expiry and a documented behaviour on expiry | A stale cursor must fail loudly rather than silently reset to page one | Where sessions are short, an expiry is a state you did not need |

**Error model**

| If | Return | Retryable | The thing candidates get wrong |
|---|---|---|---|
| The request conflicts with current state | 409 plus a code plus the conflicting field or version | Only if the body says so | Replaying an identical request, which conflicts again |
| A validator failed on a conditional write | 412 | After re-reading | Treating it as a server error |
| Quota or rate exceeded | 429 plus `Retry-After` | Yes | Omitting `Retry-After`, which guarantees a retry storm |
| The operation timed out upstream | 504 | Only with an idempotency key | Retrying a non-idempotent write with no key |
| Some items in a batch failed | 200 plus per-item results with index, code and retryable | Per item | Returning 4xx for the whole batch, which is false when most items succeeded |

**Idempotency**

| If | Do | The measurement that justifies it |
|---|---|---|
| The write has a natural unique business key you already store | A unique constraint on that key | Nothing. This is the cheap default |
| The write has no natural key and clients retry | A client-supplied key row with a fingerprint, written in the effect's transaction | A duplicate-effect incident, or a client retry rate above about 1 percent of requests |
| Concurrent arrivals on one key are possible | Add a reserved state answering 409 with `Retry-After` | Any observation of two requests on one key arriving inside one request duration |
| The client has an automated replay job | Set the key lifetime beyond that job's period | The job's schedule, stated by the client |

**Versioning and change**

| Change | Class | What you do |
|---|---|---|
| New optional response field | Compatible | Ship it, having told clients in writing to ignore unknown fields |
| New value in a request enum | Compatible | Ship it |
| New or removed value in a response enum | Breaking in practice | Treat response enums as frozen, or prove every client defaults on unknown values |
| Removed, renamed or retyped field | Breaking | Expand, dual-serve, announce per key, close the population, narrow |
| Tighter validation, new required field, changed default | Breaking | Same programme. Changed defaults are the most invisible of the three |
| Behaviour change with no schema change (a retrained model, a reordered list) | Breaking in practice for anyone depending on it | Version-stamp the behaviour and give a pinning window with a maximum duration |

**Delivery guarantees for outbound events**

| You promise | You must ship | It costs |
|---|---|---|
| At-least-once, no ordering | A stable event identifier, a per-object sequence, a catch-up read | Consumers must deduplicate, and you must document that |
| At-least-once, ordered per object | Single in-flight delivery per object key | Throughput per key becomes one over the delivery latency |
| Ordered globally per destination | Single in-flight delivery per destination | At 200 ms per attempt, 5 events per second per destination, so it fails on the first busy customer |
| Exactly-once across an HTTP boundary | Nothing, because it is not available | Promising it costs you the incident where it is discovered |

### 16b. The numbers this course uses, and where each comes from

Inputs marked assumed are chosen because they are typical order-of-magnitude figures for the scenario as stated. Every derived line stays correct as a method when you substitute the interviewer's number.

| Number | Derivation | What it eliminates |
|---|---|---|
| 500 writes per second on one contended row | 1 divided by an assumed 2 ms per indexed row update plus commit, which is the order of magnitude the hub's latency ladder gives for a small local write | Any design that puts a remote call inside that transaction: a 2 second gateway call takes the ceiling to 0.5 per second |
| 400 requests per second at a published opening time | 2,000 requests assumed to arrive in the first 5 seconds | Nothing on its own. Paired with the 500 per second ceiling it kills the email and the payment call inside the seat transaction |
| 13 percent of 500-item batches are clean | 0.996 to the power 500, from an assumed per-item failure rate of 0.4 percent measured on the single-item endpoint | All-or-nothing batch rejection, which would fail 87 percent of calls |
| 18 percent of 50-item batches contain a failure | 0.996 to the power 50 | Shrinking the batch as a fix, since the design is wrong at every size |
| 31 seconds of client retry burst | 1 plus 2 plus 4 plus 8 plus 16 seconds of full-jitter backoff over 4 retries | A key lifetime measured in seconds |
| 48 hour idempotency key lifetime | A 24 hour reconciliation job plus the 31 second burst, rounded up so a late job still lands inside the window | Both a short cache-based window and unbounded retention |
| 4 GB of live key records | 2 million writes a day times an assumed 1 KB per record times 2 days of retention | An in-process map for keys, and any argument that retention is a storage problem |
| 40,000 rows of affordable offset discard | A 20 millisecond page budget divided by an assumed 0.5 microseconds per skipped index entry, itself a few main memory reads at about 100 ns each | Offset beyond page 800 at 50 rows a page |
| 50 milliseconds at offset 100,000 | 100,000 times 0.5 microseconds | Deep offset pagination on a large collection, since the skip alone is over budget |
| about 25 index comparisons for a keyset seek | log base 2 of 40 million | The claim that deep pages are inherently expensive |
| 36 percent of a page repeats | 6 inserts per second times a 3 second page turn is 18 rows, over a 50 row page | Offset on a live feed, and the habit of calling duplicates a rare race |
| 20 seconds for an exact total | 40 million index entries at 0.5 microseconds | An exact `total` field on every page response |
| 12 MB per second of pricing traffic | 4,000 calls per second times 3 KB of request plus response | Bandwidth as an argument for a binary protocol at this volume |
| 6.4 percent of one core saved by a binary format | An assumed 16 microsecond parse saving per 2 KB call times 4,000 calls per second, giving 64 ms of CPU per second | Efficiency as the reason to pick an exposure style here |
| 1,380 polls per second to observe tier changes | 2 million customers per partner per day divided by 86,400 is about 23 per second, times 60 partners | Polling as the partner integration, in favour of an event stream |
| 100,000 deprecated calls a month from 38 keys | 0.02 percent of 500 million monthly calls, measured at month six | Ending a deprecation on a date rather than on a measurement |
| 5 days to contact the remaining population | 38 keys at an assumed 8 conversations a day for one person | The claim that a per-key migration conversation does not scale, at this population size |
| 15,000 requests per second of theoretical limit ceiling | 9,000 keys times 100 requests per minute is 900,000 per minute, divided by 60 | Sizing the limiter from the ceiling rather than from measured usage |
| 5 events per second per destination under ordered delivery | 1 divided by an assumed 200 ms per delivery attempt | Promising ordered webhook delivery |
| 162,000 events per hour of backlog | A partner generating 50 events per second against 5 delivered, times 3,600 | The idea that ordering fails gracefully. It fails invisibly and permanently |

### 16c. ASCII glyph legend

```
ASCII CONVENTIONS USED IN EVERY DIAGRAM ON THIS PAGE

 +--------+   solid box: a component you run and can page an
 | NAME   |   engineer about
 +--------+

 +========+   double edge: an aggregate boundary. Everything
 | NAME   |   inside commits in one transaction
 +========+

 +- - - - +   dashed edge: a party you do not operate, whose
 | NAME   |   latency and outage become your contract's problem
 +- - - - +

   --->       synchronous call, the caller is waiting
   ==>        asynchronous or later delivery, nobody is waiting
   <-->       two representations of one domain that must agree
   |   v      vertical continuation of the same path

 Caption: box style says whether you operate it and whether it
 is one transaction; arrow style says whether a caller is
 waiting on the reply.
```

### 16d. One-page summary cards

Eleven cards, one per case study, in the order the case studies appear.

**Card 1. Payments interface**

| Field | Content |
|---|---|
| Prompt | Design the interface a merchant integrates to charge a card, learn when money moved, and refund |
| Aggregate and invariant | Payment intent owns its state transitions and its retry key. The invariant is that one key produces at most one effect |
| Number that decides | 450 charges per second at peak times a 2.5 second gateway response is 1,125 requests in flight, which the connection pool cannot hold |
| V0 | One service, one transaction covering the key, the intent and the gateway call, merchant polls for the outcome |
| Breaking point | Connection pool exhaustion above roughly 120 per second, from a 300 connection pool divided by a 2.5 second hold |
| V1 | Accept and return a non-terminal state immediately, move the gateway call to a worker, publish state changes through an outbox |
| Deep dive one | What a retry key record stores, its three states, and how its lifetime is derived from client replay behaviour |
| Deep dive two | Event delivery as a promise: signing, at-least-once, retry horizon, and a self-service catch-up read |
| Failure class | Duplicate effect from a lost response, head-of-line blocking on one dead destination, retry storm on gateway slowdown |
| Staff delta | The non-terminal state present in version one because adding it later is the breaking change, plus ledger imbalance as a correctness alert |

**Card 2. Comment and threading service**

| Field | Content |
|---|---|
| Prompt | Design the interface for comments with replies, on posts that can attract very large threads |
| Aggregate and invariant | Comment is its own aggregate, referencing its parent by identifier. The invariant is that a reply's parent exists and belongs to the same post |
| Number that decides | An assumed worst-case thread of 500,000 comments against a 50 row page is 10,000 pages, which kills offset immediately and kills any response that embeds a subtree |
| V0 | A flat collection per post, sorted newest first, keyset paginated, with a parent identifier on each row |
| Breaking point | Rendering a nested thread needs one request per level; at an assumed depth of 8 that is 8 round trips per screen |
| V1 | Materialised path or an ordering key that encodes ancestry, so one query returns a contiguous slice of a subtree in display order |
| Deep dive one | Hierarchy representation: adjacency list against materialised path, compared on read cost, write cost and the cost of moving a subtree |
| Deep dive two | Stable ordering through a mutating collection: what the cursor encodes when the sort is by score rather than by time, and why score sorting forces a snapshot |
| Failure class | Hot key on a viral post, and unbounded fan-out when a client requests a deep subtree |
| Staff delta | A stated maximum depth and a stated maximum page span in the contract, so the expensive query is impossible rather than discouraged |

**Card 3. File and blob service**

| Field | Content |
|---|---|
| Prompt | Design the interface for uploading, downloading and managing large files |
| Aggregate and invariant | Upload session owns its parts. The invariant is that a file is visible only when every part it claims is durable |
| Number that decides | A 5 GB upload on an assumed 10 Mbit per second mobile link is about 67 minutes, which eliminates a single request and forces resumable parts |
| V0 | One `PUT` with the bytes in the body, metadata in a separate resource, a direct download |
| Breaking point | Any interruption restarts the whole transfer; at an assumed 2 percent failure rate per 10 minutes, a 67 minute upload succeeds about 25 percent of the time |
| V1 | An upload session resource with numbered parts, a committed offset a client can query, an expiry on the session, and a completion step that flips visibility |
| Deep dive one | The resumable protocol: what the client asks for after a crash, why the server owns the committed offset, and how a session expires without orphaning storage |
| Deep dive two | Lifecycle and deletion semantics: soft delete, retention windows, and what a pre-signed download link commits you to after the object is deleted |
| Failure class | Orphaned parts consuming storage forever, and a signed link outliving the permission that granted it |
| Staff delta | The expiry and the storage reclamation path designed in version one, plus a stated maximum link lifetime tied to the permission model |

**Card 4. Search service**

| Field | Content |
|---|---|
| Prompt | Design the search interface over a large, frequently updated corpus |
| Aggregate and invariant | The query is a value, not an entity. The invariant is that one result page never contradicts another page of the same result set |
| Number that decides | At an assumed 30 percent of queries reaching page two and 3 percent reaching page ten, deep result access is rare enough that a bounded result window is a product decision rather than a limitation |
| V0 | `GET /search?q=` returning a ranked page with an opaque cursor and no total |
| Breaking point | Ranking changes between page requests, so page two omits or repeats results the user already saw |
| V1 | A short-lived result set snapshot addressed by an identifier inside the cursor, with a stated expiry and a defined behaviour when it expires |
| Deep dive one | Result set stability: snapshot against re-query, what expires, and what the client sees when it does |
| Deep dive two | Exposing ranking without exposing scores: why raw scores are not comparable across index versions, and what a partner may build against instead |
| Failure class | Cursor expiry treated as an error by clients, and unbounded deep paging used as a scraping vector |
| Staff delta | A stated maximum result depth with the product reasoning behind it, plus a version stamp so a re-index is a declared behaviour change |

**Card 5. Publish and subscribe service**

| Field | Content |
|---|---|
| Prompt | Design the interface for topics, subscriptions and message delivery |
| Aggregate and invariant | Subscription owns its offset. The invariant is that acknowledging a message never moves the offset past a message that was not delivered |
| Number that decides | If a consumer processes 500 messages a second and a redelivery window is 30 seconds, a crash replays up to 15,000 messages, which forces consumer-side idempotency into the contract rather than into a footnote |
| V0 | Topics, push subscriptions, at-least-once delivery, server-managed acknowledgement |
| Breaking point | A slow consumer holds unacknowledged messages, and the backlog grows without any client-visible signal |
| V1 | Explicit flow control, a visible backlog metric per subscription, a dead-letter destination after a stated attempt count, and a replay operation |
| Deep dive one | Who owns the offset: server-managed acknowledgement against client-managed position, and what each makes possible and impossible |
| Deep dive two | Dead-letter design as a client-facing feature: how a message gets there, how it is inspected, and how it is replayed without a support ticket |
| Failure class | Poison message blocking a partition, and unbounded queue growth converting a throughput deficit into permanent delay |
| Staff delta | The delivery guarantee, the retention window and the replay path all stated in the contract, plus a per-subscription blast radius so one bad consumer cannot stall the topic |

**Card 6. Calendar and scheduling interface**

| Field | Content |
|---|---|
| Prompt | Design the interface for events, recurrence, invitations and conflicts |
| Aggregate and invariant | Event series owns its recurrence rule and its exceptions. The invariant is that an exception always refers to an instance the rule actually generates |
| Number that decides | A daily recurrence with no end date over a 10 year query window is 3,650 instances per series, which eliminates materialising instances at write time for open-ended rules |
| V0 | Single events with explicit start and end times, invitations as a list of attendees with a status each |
| Breaking point | A recurring series edited "for this instance only" has nowhere to live, and a series edited "from here on" splits identity |
| V1 | Series plus rule plus a sparse exception set, with instances expanded at read time inside a bounded window, and a stable identifier per instance |
| Deep dive one | Time correctness: storing a local time plus a zone identifier rather than an instant for future events, and why a daylight saving transition changes the answer |
| Deep dive two | The invitation state machine across parties: whose copy is authoritative, what a decline changes, and what happens when two attendees edit at once |
| Failure class | Clock and zone-database skew between client and server, and an unbounded expansion request on an open-ended rule |
| Staff delta | The bounded expansion window written into the contract, and a stated rule for which party's edit wins, rather than last write wins by accident |

**Card 7. Chat interface**

| Field | Content |
|---|---|
| Prompt | Design the interface for a chat product: live messages, history and read state |
| Aggregate and invariant | Conversation owns its message sequence. The invariant is that a message identifier, once assigned, never changes and never reorders |
| Number that decides | A client offline for a day in a conversation receiving 2 messages a minute is 2,880 messages to reconcile, which forces a cursor-based catch-up read rather than a full history fetch |
| V0 | A socket for live delivery and a paginated history read, with the server assigning identifiers and order |
| Breaking point | Messages composed offline have no server identifier, so the client cannot display them consistently or deduplicate on reconnect |
| V1 | A client-generated idempotency key per message, a server-assigned sequence, and a read-state cursor per participant |
| Deep dive one | Splitting the contract by transport: what the socket promises, what the resource read promises, and why the second must be able to reconstruct anything the first delivered |
| Deep dive two | Read state as a per-participant cursor rather than a per-message flag, and what that removes from the write path |
| Failure class | Duplicate messages from offline retries, and thundering herd on mass reconnect after a socket tier restart |
| Staff delta | Reconnect behaviour specified as part of the contract, including jitter expected of clients, plus the guarantee that history alone can rebuild client state |

**Card 8. Ride hailing interface**

| Field | Content |
|---|---|
| Prompt | Design the interface for requesting a ride, tracking it, and cancelling |
| Aggregate and invariant | Trip owns its state machine. The invariant is that every transition is one of a declared set, and that a terminal state is final |
| Number that decides | Location at 1 update per second for an assumed 40 minute trip is 2,400 updates, which eliminates delivering location through the same resource as the trip |
| V0 | Create a trip, poll it for state, cancel it before pickup |
| Breaking point | Polling at 1 second per client for live location multiplies read load by trip duration, and still shows a stale position |
| V1 | The trip resource carries state and legal transitions; location moves to a separate stream with its own lifetime, cost and cancellation |
| Deep dive one | Exposing a state machine honestly: publishing the legal transitions, refusing illegal ones with a distinct code, and versioning the state set as a frozen enum |
| Deep dive two | Cancellation semantics across parties: who may cancel in which state, what it costs, and how a cancellation racing an acceptance is resolved |
| Failure class | Split brain between rider and driver views of the trip state, and clock skew on client-supplied timestamps |
| Staff delta | The state set treated as frozen from version one, with the fee consequence of each cancellation window written into the contract rather than into a support macro |

**Card 9. Tool-calling surface for an AI agent**

| Field | Content |
|---|---|
| Prompt | Design the tool interface an autonomous agent calls on a user's behalf |
| Aggregate and invariant | Each tool call is its own record with its own approval state. The invariant is that a side-effecting tool never executes without a recorded approval |
| Number that decides | An agent that retries a failed call twice, at an assumed 3 tool calls per task, produces up to 9 invocations per task, so every side-effecting tool needs a key rather than a hope |
| V0 | A schema per tool, synchronous execution, a result or an error |
| Breaking point | A non-deterministic caller resubmits a semantically identical call with a different body, so body-hash deduplication does not fire |
| V1 | Declared side-effect class per tool, a caller-supplied idempotency key required on any mutating tool, an approval gate for the classes that need one, and per-call cost metering |
| Deep dive one | The schema as the contract for a caller that cannot read documentation: closed enums, no free-text fields where a set will do, and error messages written to be actionable by a machine |
| Deep dive two | Cost and cancellation: metering per call, streaming partial results, and what a cancelled call commits you to having already done |
| Failure class | Runaway retry loops from an agent that treats every error as retryable, and poison inputs that fail identically forever |
| Staff delta | Side-effect declaration and approval as schema-level fields rather than documentation, plus a per-tenant spend cap with a stated behaviour when it is reached |

**Card 10. Feature flag and configuration service**

| Field | Content |
|---|---|
| Prompt | Design the interface applications use to read feature flags and configuration |
| Aggregate and invariant | A flag owns its rules and its default. The invariant is that evaluation for a given key and subject is deterministic within a configuration version |
| Number that decides | 20,000 application instances polling every 30 seconds is 667 requests per second for data that changes a few times a day, which eliminates polling as the primary path |
| V0 | A read endpoint returning the full configuration document, polled on an interval |
| Breaking point | The service being unavailable takes every application with it, because a flag read is on the startup path |
| V1 | A streamed or long-polled change feed with a version, a local cache with a documented staleness bound, and a required client-side default for every flag |
| Deep dive one | Where evaluation happens: server-side evaluation leaks subject attributes to you and centralises correctness, client-side evaluation ships rules and keeps working offline |
| Deep dive two | Staleness and offline behaviour as a contract: the maximum staleness you promise, what the client does past it, and why a flag with no local default is a hidden hard dependency |
| Failure class | Cold-start collapse when every instance restarts and polls at once, and gray failure where a stale configuration serves for hours unnoticed |
| Staff delta | The blast radius of one bad flag change stated explicitly, with a staged rollout in the contract and a stated maximum staleness rather than an implied one |

**Card 11. Brownfield: a breaking change on a live public interface**

| Field | Content |
|---|---|
| Prompt | Here is our current public interface. We need a change that breaks it, and thousands of clients cannot be upgraded |
| Aggregate and invariant | Nothing new is modelled. The invariant is that no request that worked yesterday fails today |
| Number that decides | 0.02 percent of 500 million monthly calls is 100,000 calls from 38 keys, and 38 owners is about 5 days of conversations at 8 a day, which is what makes the window closable |
| V0 (what exists) | A live interface, an unknown usage distribution per field, and a change request with a date attached |
| Breaking point | The date arrives with hundreds of keys still calling the old shape, so you either break them or extend publicly |
| V1 | Measure field-level usage per key, expand, dual-serve, announce per key with their own numbers, default new keys to the new shape, then narrow |
| Deep dive one | Classifying the change: which parts are compatible, which are breaking, and why a new response enum value belongs in the second group |
| Deep dive two | Ending the window on a measurement: the weighted traffic threshold, the 30 day zero-call rule, and the contact list with named owners |
| Failure class | A silently empty field that returns 200, and a deprecation extended in public because the exit condition was a date |
| Staff delta | The ongoing cost of dual serving stated in handlers and test cases, so ending the window is argued as an engineering cost rather than as tidiness |

---

## 17. What this course does not cover

Naming our limits is cheaper than pretending we have none, and it stops you studying the wrong thing.

| Not covered | Why | Where to go instead |
|---|---|---|
| Distributed systems fundamentals: partitioning, replication, consensus, multi-region, capacity | Assumed as background here. A contract sits on top of them, and this round does not score them | Our [Modern System Design](modern-system-design.md) course, and Designing Data-Intensive Applications, Martin Kleppmann, 1st edition, O'Reilly, 2017 |
| Product sense: which feature to build, for whom, at what price | A genuinely different round with a different rubric. The course header says so, and Module 1 tells you how to spot it in the first minute | Ask your recruiter which round you are in, in writing. It is the cheapest question you will ask all quarter |
| Domain-driven design as a discipline: bounded contexts across an organisation, context maps, strategic design | We use aggregates and invariants because they decide transaction boundaries in an interview. The organisational half is a career, not a section | Domain-Driven Design, Eric Evans, Addison-Wesley, 2003, and Implementing Domain-Driven Design, Vaughn Vernon, Addison-Wesley, 2013 |
| Security engineering beyond grants and scopes: threat modelling, cryptographic protocol design, key management at depth | We cover what a contract must state. Doing security properly here would be worse than sending you to people who do it full time | The OWASP API Security Top 10, 2023 edition, published free at owasp.org, and your own security team before anything ships |
| Specification tooling in detail: which schema language, which generator, which linter, which mock server | Product names and their defaults move faster than a page can be reviewed, and a design answer should not turn on a brand | The current documentation for whichever you use, checked on the day. The compatibility rules in Module 6 are tool-independent on purpose |
| Frontend and mobile consumption of these interfaces: caching layers, offline stores, rendering | Real, adjacent and large. We cover only what the contract must offer a constrained client | Our [Frontend System Design](frontend-system-design.md) and [Mobile System Design](mobile-system-design.md) courses |
| Model quality behind a model-backed endpoint: training, evaluation, drift, serving cost | We design the surface over the model, not the model. The two rounds are scored differently | Our [ML System Design](ml-system-design.md) course |
| Token budgets, retrieval augmentation and evaluation of free-text output | Module 18 covers the contract for a tool-calling surface. The system behind it is a separate arc | Our [Generative AI System Design](generative-ai-system-design.md) course |
| Integration patterns catalogued at implementation depth: message routing, transformation, channel patterns | We teach the decision and the guarantee. The catalogue form is useful separately | Enterprise Integration Patterns, Hohpe and Woolf, Addison-Wesley, 2003 |
| Operating a service: on-call practice, incident command, postmortem culture | We cover the indicators and budgets a designer must state. Operating is a job | The Site Reliability Engineering book, O'Reilly, 2016, readable free at sre.google, and Release It!, Michael Nygard, 2nd edition, Pragmatic Bookshelf, 2018 |
| Organisational design around interfaces: team boundaries, ownership models, platform funding | It decides more about real interfaces than any technique here, and it is not what this round asks | Team Topologies, Skelton and Pais, IT Revolution, 2019, and Building Microservices, Sam Newman, 2nd edition, O'Reilly, 2021 |
| Regulated domains: payment card compliance, financial reporting, health data rules | Jurisdiction-specific and consequential. Generic advice would be worse than none | Your legal and compliance function, plus the relevant regulator's own published guidance for your sector |
| Behavioural rounds and company-specific loop formats | Loop structures change per team and per quarter, and unverifiable claims about them are exactly what we refuse to publish | Ask your recruiter for the current format, in writing |

!!! note "On anything we cite"

    We name the edition and the year so you can tell what has aged. If a link here is dead or a newer edition exists, open an issue and it is fixed in the next review pass rather than sitting behind an unverifiable "recently updated" badge.

---

## 18. Provenance

**Last reviewed:** 1 August 2026. A page whose last-reviewed date is more than nine months old is flagged for review rather than quietly left in place.

**Changelog, most recent first**

| Date | Change |
|---|---|
| August 2026 | First publication: twenty-one modules, eleven case studies with the full arc and level deltas, twelve practice problems with public rubrics, and a mastery check mapped one to one against the seven learning objectives |
| August 2026 | The error model, idempotency and deprecation material was promoted from subsections into full modules after review, because those three are where candidates visibly fail and where competing material is thinnest |
| August 2026 | Every figure in Sections 10, 12 and 16 re-derived from inputs stated in the same section; figures that could not be derived were relabelled as assumptions with the reason each is reasonable, and the ones that eliminated no design option were deleted |

**Found a problem?** Open an issue at [github.com/datascienceinterviews/datascienceinterviews.github.io/issues](https://github.com/datascienceinterviews/datascienceinterviews.github.io/issues). Corrections, disputed numbers, dead links and better derivations are all in scope. Errors get fixed in days, and the changelog above records what changed.

All content on this page is original. Research informed which topics belong here; no prose, framework, acronym, example, table or diagram is derived from any paid course. Primary sources are cited inline with their edition and year, and any claim we could not source is either labelled as an assumption or is not on the page.
