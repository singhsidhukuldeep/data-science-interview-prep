---
title: "System Design Interview Fast-Track in 48 Hours"
description: An hour-by-hour 48 hour plan for a design round days away, built around what to skip and which deeper course to open once the round is over.
last_reviewed: 2026-08-01
---

# System Design Interview: Fast-Track in 48 Hours

!!! abstract "The contract"

    **After this course you can** walk into a 45 minute design round two days from now with a minute budget you have rehearsed, eight building blocks you can defend, one deep dive you can hold for eight minutes, and a written list of what you deliberately did not prepare.

    **Who it is for**

    - An engineer whose round is in two to five days, who has already booked the slot and cannot move it.
    - Someone who has designed systems at work but has not sat a design round in over a year.
    - Someone who opened a full course, saw thirty hours of reading, and closed it again.

    **Who it is not for**

    | If this is you | Go here instead |
    |---|---|
    | You have two weeks or more before the round. Say it plainly: cramming loses to a full path, and every hour you spend here is an hour of worse material | [Modern System Design](modern-system-design.md) |
    | Your round is a ranking, retrieval or prediction problem | [ML System Design](ml-system-design.md) |
    | Your round is a product built on a foundation model | [Generative AI System Design](generative-ai-system-design.md) |
    | Your round is an app client, offline sync or battery | [Mobile System Design](mobile-system-design.md) |
    | Your round is a browser application or one component | [Frontend System Design](frontend-system-design.md) |
    | Your round is an API contract, error model or webhook | [Product Architecture](product-architecture.md) |

    **Trains for:** the 45 or 60 minute generalist distributed-systems round, at mid and senior level, when you have 48 wall-clock hours and nothing else.

    **Does not train for:** staff-plus architecture loops, take-home designs, coding rounds, or any round where the prompt is a model, a phone or a browser.

    **The promise, stated as a limit.** This sprint omits a great deal on purpose. It omits consensus internals, storage engine internals, multi-region write models, cost modelling, observability design, brownfield migration sequencing and level calibration. The next section names every omission and where it lives. If reading that table makes you want the omitted material, you do not want this page, you want [Modern System Design](modern-system-design.md).

    **Fast path:** need the plan now? Go to [the triage diagnostic](#the-triage-diagnostic). Need only the tables? Go to [the reference block](#16-reference-block).

---

## What this sprint deliberately skips

This is the most useful table on the page, so it comes first rather than last. Everything below is real interview material that a 48 hour plan cannot buy you. Reading the list is how you decide whether the trade is acceptable.

| Omitted topic | Why it is out of a 48 hour plan | Where it is covered |
|---|---|---|
| Consensus internals: Raft, leases, fencing tokens | Takes hours to hold and appears as a follow-up in a minority of rounds. A candidate who says "I would use a managed coordination service and here is what I depend on it for" loses almost nothing | [Modern, Module 9](modern-system-design.md#module-9-building-blocks-iv-coordination) |
| Storage engine internals: LSM trees, compaction, write amplification | Deep, slow to learn, and rarely decides an interview answer at mid or senior level | [Modern, Module 7](modern-system-design.md#module-7-building-blocks-ii-storage) |
| Multi-region write models and data residency | Only bites when the prompt names a regulator or a recovery time objective in minutes. You can scope it out in one sentence | [Modern, Module 10](modern-system-design.md#module-10-multi-region-and-geo-replication) |
| Cost per million requests as an argument | A genuine staff signal, and unreachable in two days without practice at attaching money to boxes | [Modern, Module 13](modern-system-design.md#module-13-cost-as-a-first-class-constraint) |
| Observability designed in, not bolted on: SLI choice, cardinality budgets | Cheap to fake badly, expensive to learn properly. The sprint teaches one line about what you would measure first, and stops | [Modern, Module 12](modern-system-design.md#module-12-observability-and-operability-as-design-output) |
| Brownfield sequencing: dual writes, shadow traffic, backfill order, rollback | Needs its own worked examples. If your loop is known to be brownfield, this page is the wrong page | [Modern, Module 15](modern-system-design.md#module-15-brownfield-design) |
| Level calibration: rewriting a mid answer into a staff answer | Requires you to have a stable mid-level answer first, which is what these 48 hours are for | [Modern, Module 16](modern-system-design.md#module-16-level-calibration) |
| Six of the eight flagship case studies: web crawler, video platform, ride hailing, proximity search, typeahead, plus both extra dives per prompt | Three prompts practised to a breaking point beat eight prompts read passively | [Modern, case studies](modern-system-design.md#case-study-2-web-crawler) |
| The full trade-off atlas, twenty-two decisions | The sprint carries eight of them. The rest are reference you can read after the round | [Hub, trade-off atlas](index.md#the-trade-off-atlas) |
| The full failure library, fifteen classes with triggers and preventions | The sprint meets classes as they arise inside three case studies and two drills, never as a catalogue you study, so coverage is uneven and the classes your three prompts never raise stay unknown to you | [Hub, failure library](index.md#the-failure-library) |
| Interviewer script packs and a second and third self-mock | Requires a willing human and more hours than remain | [Modern, section 15](modern-system-design.md#15-study-schedule-and-spaced-review) |

!!! warning "What the omissions actually cost you"

    Two things, and it is fair to name them. First, a follow-up that lands on an omitted topic will get a shallower answer from you than it would have after a full path. Second, the sprint gives you one strong deep dive rather than a choice of four, so an interviewer who steers you away from your dive will find thinner ground.

    Both are survivable. The honest-gap script in the delivery clock section is what you use, and it costs less than bluffing. What is not survivable is arriving with no clock, no opening, and no rehearsed prompt, which is where most of the 48 hours go.

---

## Prerequisites

Five rows, not eight, because a reader with 48 hours cannot act on a longer list. Answer each Quick check in under a minute. Two or more misses and the honest advice is to move your round if you can, and to read [Modern System Design](modern-system-design.md) if you cannot.

| You should already be able to | Where to learn it | Quick check |
|---|---|---|
| Trace one HTTP request from client to database and back | [System design question bank](/Interview-Questions/System-design/) | Name the three hops between a browser and a row in a database, in order |
| Divide a daily count into a per-second rate in your head | [Hub, estimation anchors](index.md#anchors-worth-memorising) | 8.6 million requests a day is roughly how many per second? |
| Read a percentile rather than an average | [Google SRE Book, chapter 6, free online](https://sre.google/sre-book/monitoring-distributed-systems/) | Your p99 is 300 ms and you call four services in parallel. Is the combined p99 better or worse, and why? |
| Say what a cache hit ratio does to origin load | [MDN HTTP caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching) | The edge takes 20,000 requests per second at a 95 percent hit ratio. What does the origin see? |
| Draw six boxes on a shared canvas without redrawing them | Any drawing tool, ten minutes | Draw login: browser, gateway, service, database, and the arrows. Did you leave room on the right? |

??? note "Show answers to the quick checks"

    1. Browser to load balancer or gateway, gateway to application service, service to database. Anything else is an optimisation you have not justified yet.
    2. 8,600,000 divided by 86,400 is close to 100 per second. Memorise 86,400 and stop multiplying.
    3. Worse. If each of four calls independently exceeds 300 ms one percent of the time, the chance none does is 0.99 to the fourth power, about 0.961, so roughly 3.9 percent exceed it. Parallel fan-out converts a p99 into about a p96.
    4. Five percent of 20,000, so 1,000 per second. The edge is doing a factor of twenty, and losing it means the origin must survive twenty times its normal load.
    5. No answer key. If you redrew it, that is the practice, and it is the cheapest ten minutes in this sprint.

---

## Time budget

The sprint is 48 wall-clock hours. Working hours are the part you spend at a desk. The two sums below are the whole method: working plus non-working equals 48, and the seven blocks in the plan sum to 48.

| Part of the sprint | Reading | Practice | Review and taper |
|---|---|---|---|
| Hours 0 to 10: triage, delivery clock, toolkit, estimation drills | 3.0 to 3.5 h | 4.0 to 5.0 h | 0.5 h |
| Hours 10 to 18: cards either side of the first sleep | 0 | 0 | 0.5 h |
| Hours 18 to 34: three case studies, three timed drills, two deep dives | 2.5 to 3.0 h | 10.0 to 10.5 h | 0.5 h |
| Hours 34 to 42: weak-spot repair, then the second sleep | 0 | 0.5 h | 0.5 to 1.0 h |
| Hours 42 to 48: two interleaved drills, checklists, summary cards | 0.5 h | 1.5 to 2.0 h | 1.0 h |
| **Total working** | **6.0 to 7.0 h** | **16.0 to 18.0 h** | **3.0 to 3.5 h** |

| Line | Hours |
|---|---|
| Working total, the three columns above summed | 25.0 to 28.5 |
| Non-working: two sleeps of about 7 hours, meals, travel, dead time | 19.5 to 23.0 |
| **Wall clock** | **48.0** |

!!! note "How these were computed, so you can argue with them"

    Reading is summed per block, not guessed for the page. Reading assumes 120 to 150 words a minute, which is deliberately slow, because dense prose with diagrams is not read at prose speed.

    Practice is itemised: three case studies attempted cold for 20 minutes each (60 min), the same three rewritten for 10 minutes each after reading (30 min), five timed drills at 25 minutes each (125 min), six estimation drills at 5 minutes each (30 min), two opening rehearsals at 6 minutes (12 min), four deep-dive rehearsals at 8 minutes (32 min), and one 30 minute repair session. Those itemised activities total 319 minutes.

    The remaining 641 to 761 minutes are unstructured drawing, rewriting and re-attempting, mostly inside blocks 18 to 34. Itemised plus unstructured is 960 to 1,080 minutes, which is the 16.0 to 18.0 hours above.

    Review is five sessions: 30 minutes in block 0 to 10, 30 minutes split either side of the first sleep, 30 minutes inside blocks 18 to 34, 30 to 60 minutes before the second sleep, and 60 minutes in the taper. That is 180 to 210 minutes, the 3.0 to 3.5 hours above. Every figure is padded by roughly 15 percent, because self-estimates of study time run optimistic.

    A single number would be more marketable and less true. The three tracks in the triage land at 27.5, 27.0 and 25.0 working hours, all inside the range above.

---

## Learning objectives

Six objectives. Each is tested by exactly one numbered item in the [mastery check](#12-mastery-check-and-remediation-map), and the numbering matches: objective 1 is tested by mastery item M1.

1. **Route** yourself to one of three sprint tracks using the eight-question triage, and **state** which two blocks your track expands and which two it cuts, with the reason for each.
2. **Deliver** the first two minutes of a 45 minute round as four sentences that restate the prompt with a number, name the hard parts, scope something out, and ask one forking question, without naming a phase out loud.
3. **Produce** two numbers for any prompt in under three minutes, each followed by the design option it removes, and **decline** estimation in one sentence when no option would die.
4. **Draw** a v0 using only components from the eight-item toolkit, using the fewest the written requirements permit, then **name** the metric and the threshold at which it breaks and how you would observe it.
5. **Sustain** one deep dive for eight minutes on a prompt of your choice, naming a mechanism, a number derived in front of the interviewer, and one failure class named anywhere in this sprint.
6. **Convert** a scored self-mock into a next-block reading list using the remediation map, and **name** the sibling course that repairs each weakness after the round.

---

## Warm-up retrieval

Three to five minutes, before you open anything else. Answer out loud. If all three are blank, that is data for the triage, not a reason to stop.

**Q1.** The hub states one hard rule about when a number is allowed to appear in your answer. State it.

??? note "Show answer"

    A number may appear only if the next sentence uses it to rule out a design option. If nothing is ruled out, delete the number. The one exception is a number explicitly labelled an assumption because a later step depends on it, and that one carries a sentence saying why the assumption is reasonable.

**Q2.** In the hub's 45 minute clock, more than half the working minutes sit after one particular event. Which event, and which two phases are widest?

??? note "Show answer"

    After the first diagram. The two widest blocks are the breaking point with V1 (8 minutes) and the deep dive (9 minutes), against 8 minutes for drawing V0. Candidates who fail on delivery usually spend that second half adding boxes instead of stressing the ones already drawn.

**Q3.** Reaching back to the hub's failure library: name the class where a system stays broken after the original trigger is gone, and the two design changes that prevent it.

??? note "Show answer"

    Metastable failure. The degraded state generates its own load, usually through retries, so removing the trigger does not recover it. Prevention: load shedding at admission, and bounded queues that reject rather than buffer. Recovery normally requires shedding traffic deliberately, which is why the runbook matters as much as the design.

---

## The triage diagnostic

Two readers with the same deadline should not follow the same hours. Answer eight questions yes or no, count the yes answers, and read your track off the diagram.

1. Have you sat at least one real system design interview, not a mock, in the last 18 months?
2. Can you state a minute budget for a 45 minute round right now, without looking it up?
3. In the last month, have you drawn a system for another person on a whiteboard or shared canvas?
4. Can you convert 400 million requests per day into requests per second in your head, right now?
5. Can you name three failure classes and the design change that removes each?
6. Have you operated a service in production that had a paging alert attached to it?
7. Can you name the number that decides between adding a cache and adding a read replica?
8. In your last design discussion at work, did you remove a component rather than add one?

??? note "Show answer: the factual questions, so you can score 4, 5 and 7 honestly"

    **Q4.** 400,000,000 divided by 86,400 is about 4,630 per second average. If you needed paper, score it no.

    **Q5.** Any three of: retry storm, cache stampede, thundering herd, hot key, metastable failure, head-of-line blocking. Each needs its removal: one retry layer with a budget and jitter; single-flight refresh; jittered wakeups; a local cache tier or a split key; load shedding with bounded queues; separate queues by cost class. Naming three without their fixes scores no.

    **Q7.** The achievable hit rate on the hot subset. A cache earns its place when a small subset serves most reads and seconds of staleness are acceptable; below roughly an 80 percent hit rate you have added a hop and a staleness bug for nothing, and a replica is the better tool. That threshold is a labelled assumption, reasonable because below it the miss path still carries most of the traffic.

```
+---------------------------------------------+
| Count the yes answers from the eight above  |
+---------------------------------------------+
   |
   +-> +--------------------------------------+
   |   | 0 to 2 yes = Track A, first timer    |
   |   | more clock and toolkit, one dive     |
   |   +--------------------------------------+
   |
   +-> +--------------------------------------+
   |   | 3 to 5 yes = Track B, rusty          |
   |   | the published plan, unchanged        |
   |   +--------------------------------------+
   |
   +-> +--------------------------------------+
       | 6 to 8 yes = Track C, out of practice|
       | skim blocks 1 and 2, all the reps    |
       +--------------------------------------+
```

Caption: the triage router. Your yes count selects one of three tracks. Track A buys framing and vocabulary, Track B runs the published plan unchanged, and Track C skips most of the teaching and spends the hours on timed repetitions instead.

### Working hours per block, by track

Same 48 wall-clock hours, allocated differently. Unused block time is rest, not spare study time.

| Block | Wall clock | Track A | Track B | Track C |
|---|---|---|---|---|
| Hours 0 to 4, set up and the clock | 4 h | 3.5 | 3.0 | 2.0 |
| Hours 4 to 10, toolkit and estimation | 6 h | 5.5 | 5.0 | 3.5 |
| Hours 10 to 18, sleep and cards | 8 h | 0.5 | 0.5 | 0.5 |
| Hours 18 to 26, three prompts at sprint pace | 8 h | 7.0 | 7.0 | 7.0 |
| Hours 26 to 34, deep dives and mock | 8 h | 6.5 | 7.0 | 7.0 |
| Hours 34 to 42, repair, then sleep | 8 h | 1.0 | 1.0 | 1.5 |
| Hours 42 to 48, taper | 6 h | 3.5 | 3.5 | 3.5 |
| **Working total** | **48 h wall** | **27.5** | **27.0** | **25.0** |

What each track changes, and why:

- **Track A expands** hours 0 to 4 and 4 to 10, because a first timer's biggest loss is a round with no structure and no vocabulary. **It cuts** hours 26 to 34 to one deep dive instead of two, because a second dive you cannot hold adds nothing.
- **Track B** runs the plan as published. It is the calibration point for the other two.
- **Track C cuts** hours 0 to 4 and 4 to 10 to skim reading, because the material is already held. **It expands** hours 34 to 42 and the drill count, because its failure mode is a rusty performance, not a missing schema.

---

## The 48 hours, block by block

```
+------+--------+--------+--------+--------+--------+------+
| Set  | Tools  | SLEEP  | Sprint | Dives  | SLEEP  |Taper |
| up   | + est  | + card | 3 cases| + mock | + fix  |+ mock|
| 4 h  | 6 h    | 8 h    | 8 h    | 8 h    | 8 h    | 6 h  |
+------+--------+--------+--------+--------+--------+------+
0      4        10       18       26       34       42    48
```

Caption: the shape of the sprint in wall-clock hours. Two of the seven blocks are sleep, which is 16 of the 48 hours, and they are not negotiable. The heaviest work sits in the two eight hour blocks between the sleeps, and the final block is a taper rather than a push.

### Anchoring the blocks to real clock times

Hour 0 is 48 hours before your round starts. One worked anchoring, which is the case where the blocks land perfectly:

| Block | If the round is Thursday at 14:00 |
|---|---|
| 0 to 4 | Tuesday 14:00 to 18:00 |
| 4 to 10 | Tuesday 18:00 to Wednesday 00:00 |
| 10 to 18 | Wednesday 00:00 to 08:00, sleep |
| 18 to 26 | Wednesday 08:00 to 16:00 |
| 26 to 34 | Wednesday 16:00 to Thursday 00:00 |
| 34 to 42 | Thursday 00:00 to 08:00, sleep |
| 42 to 48 | Thursday 08:00 to 14:00 |

If your round is not at 14:00, the sleep blocks will not land on your night. Shift blocks 10 to 18 and 34 to 42 onto whatever eight hour window contains your normal sleep, and absorb the difference in the neighbouring blocks. Never move sleep to make room for study.

### Hours 0 to 4: buy a clock and an opening

| | |
|---|---|
| **Goal** | Leave this block able to say the first four sentences of a round out loud, from memory, on any prompt |
| **Read** | The triage above; [the hub's clock](index.md#the-clock); [the first two minutes](index.md#the-first-two-minutes); [stating what you are not covering](index.md#stating-what-you-are-deliberately-not-covering); [Modern, Module 2](modern-system-design.md#module-2-the-delivery-clock) |
| **Drill** | Score the triage. Then record yourself on your phone doing the four-sentence opening for "design a link shortener". Listen back. Do it again. Six minutes each, twice |
| **Hard stop** | At hour 4 you stop reading clock material for the rest of the sprint, even if the opening still feels rough. It gets fixed by repetition in later blocks, not by more reading |

### Hours 4 to 10: the toolkit and numbers that eliminate

| | |
|---|---|
| **Goal** | Leave able to write the eight building blocks and six numbers from memory, and to produce one eliminating number in under 90 seconds |
| **Read** | [The minimum viable toolkit](#the-minimum-viable-toolkit) on this page; [the hub on estimation](index.md#capacity-estimation-that-earns-its-place); [the anchors](index.md#anchors-worth-memorising) and [the latency ladder](index.md#the-latency-ladder); [Modern, Module 4](modern-system-design.md#module-4-estimation-as-elimination) |
| **Drill** | The [six estimation drills](#1h-six-estimation-drills-five-minutes-each), five minutes each. Then close the page and write the eight blocks, six numbers and five patterns on paper. Check. Rewrite the ones you missed |
| **Hard stop** | No storage internals, no consensus, no multi-region. If you are reading about compaction at hour 9, you have left the plan and the plan is what you are buying |

### Hours 10 to 18: sleep, and this block is load-bearing

| | |
|---|---|
| **Goal** | Consolidate what blocks 1 and 2 put in. This block does more for recall than a third study block would |
| **Read** | The [review cards](#16d-review-cards) for 15 minutes before sleeping, and the same cards for 15 minutes on waking. Nothing else |
| **Drill** | None, deliberately. The absence is the design |
| **Hard stop** | If you are awake at hour 15 studying, you have converted hours 18 to 26 into low-value time. The plan assumes about seven hours of sleep here. Stop |

### Hours 18 to 26: three prompts, requirements to first breaking point

| | |
|---|---|
| **Goal** | Leave with three prompts you have taken from a blank page to a named breaking point, twice each |
| **Read** | The [three sprint case studies](#sprint-case-study-1-url-shortener) on this page, one at a time, after attempting each |
| **Drill** | For each prompt: 20 minutes attempting it cold on paper, then read the arc, then 10 minutes rewriting your version. Then run [timed drills D1 and D2](#timed-drills) at 25 minutes each |
| **Hard stop** | No deep dives in this block. If one prompt exceeds 55 minutes including reading, abandon it and move on. Three shallow completions beat one polished one |

### Hours 26 to 34: two deep dives, at real depth

| | |
|---|---|
| **Goal** | Leave able to hold one dive for eight minutes without notes, with a mechanism, a derived number and a failure class |
| **Read** | The two flagship sections your triage points at. Track A reads one, not two |
| **Drill** | [Timed drill D3](#timed-drills) at 25 minutes. Then explain each chosen dive aloud for 8 minutes to an empty room or a recording, then again after checking what you dropped |
| **Hard stop** | Two dives, not four. A third dive costs you the rehearsal time that makes the first two usable, and interviewers grade one deep answer above three shallow ones |

### Hours 34 to 42: repair, then sleep

| | |
|---|---|
| **Goal** | Turn drill scores into one hour of targeted repair, then stop taking on anything new |
| **Read** | Your own rubric scores from D1 to D3, then the one row of the [remediation map](#12-mastery-check-and-remediation-map) with your lowest score |
| **Drill** | 30 minutes re-answering only the lowest-scoring segment, not the whole drill. Then 30 to 60 minutes on the review cards. Then sleep for about seven hours |
| **Hard stop** | Nothing new after hour 36. New material introduced this late cannot be retrieved under pressure and it displaces material that can |

### Hours 42 to 48: taper

| | |
|---|---|
| **Goal** | Arrive rested with three summary cards in working memory and a checklist you have actually read |
| **Read** | The [three summary cards](#16e-one-page-summary-cards), the [interview-day checklist](#14-interview-day-checklist), the review cards once |
| **Drill** | [Timed drills D4 and D5](#timed-drills), the interleaved pair, 25 minutes each. Then stop |
| **Hard stop** | At hour 46 close everything. The last two hours are food, a walk, and setting up the room and the drawing tool. No new prompt, no new page, no last article |

---

## The minimum viable toolkit

This is the only teaching module in the sprint. Everything in it is chosen because it appears across most generalist prompts, and everything that appears in only one prompt has been cut.

### 1a. Predict first

Before reading on, commit to an answer. A candidate draws a load balancer, an application tier, a cache, a queue and a database, then reads the requirements. Which of those five boxes is most likely to be the one they cannot justify when asked, and why that one?

??? note "Show answer"

    The cache, and the reason is arithmetic rather than taste. A load balancer and an application tier are justified by having more than one process. A database is justified by needing state. A queue is justified as soon as one piece of work can be slower than the response.

    A cache needs a measured hit rate on a hot subset plus a tolerance for staleness, and neither exists before the requirements are written. It is also the most rehearsed move in prep material, so it is the box interviewers probe first.

### 1b. The eight building blocks

Eight, not twenty. Each row carries the measurement that justifies it and the threshold below which it is over-engineering.

| Block | Reach for it when | When not to use it |
|---|---|---|
| Stateless service tier behind a load balancer | You need more than one process, or you need to deploy without downtime | Below a few hundred requests per second, two instances and DNS is the whole answer, and the balancer is a component that can fail on its own |
| Relational primary with read replicas | Query shapes are still moving, you need joins, and reads outnumber writes | When reads are not the pressure. Replicas add read capacity and lag, never write capacity, so a write bottleneck is untouched by them |
| Shared cache tier in front of the store | A small subset serves most reads and seconds of staleness are acceptable | Below roughly an 80 percent achievable hit rate on the hot subset, you have added a hop and a staleness bug for nothing. Use a replica |
| Queue between a fast path and slow work | The caller needs an acknowledgement, not the result, and the work completes within a stated delay | When the caller cannot render its next screen without the result. Then a queue does not remove the latency, it hides the failure |
| Object store for large values, with a row that points at it | Values exceed a few hundred kilobytes and are served by URL | For small values always read alongside their row. Blobs in the database bloat backups, caches and replication instead |
| Hash partitioning by an entity key | One node can no longer hold the data or absorb the writes | Before you have measured either. The partition key is the hardest decision on the page to reverse, so it is the last one to make, not the first |
| Edge cache or CDN for cacheable reads | Users are geographically spread and the object is reusable across them | When you justified it on bandwidth and your egress is tens of megabits per second. Justify it on geography and first-byte latency, or drop it |
| Idempotency key plus at-least-once transport | A retry could produce a second real-world effect, such as a second charge | When the effect is a blind overwrite of a whole row by key, which is already idempotent. Adding keys there is machinery with no failure to prevent |

### 1c. The toolkit assembled

```
+---------+    +-------------+    +---------------+
| Client  |--->| App tier    |--->| Primary store |
+---------+    | stateless   |    | + replicas    |
               +-------------+    +---------------+
                  |     |                ^
                  v     v                |
          +---------+ +---------+  +-----------+
          | Cache   | | Queue   |->| Worker    |
          | hot set | | bounded |  | slow work |
          +---------+ +---------+  +-----------+
                          |
                          v
                   +---------------+
                   | Object store  |
                   | large values  |
                   +---------------+
```

Caption: the seven boxes most generalist prompts are built from, with the eighth (partitioning) being a property of the primary store rather than a box. The client calls a stateless app tier, which reads through a cache to a primary store with replicas, pushes slow work onto a bounded queue for a worker, and keeps large values in an object store. Almost every v0 in this sprint is a subset of this picture.

### 1d. Worked example

Prompt: "Design an internal document store for a company of 4,000 employees. People upload files and search them by title."

I narrate the decisions in the order I would say them.

**Block one, purpose: establish that scale is not the problem here.** 4,000 employees is GIVEN. Assume each does 20 document operations a working day, which is reasonable because a document tool is used in bursts rather than continuously. That is 80,000 operations a day, and 80,000 divided by 86,400 is under 1 per second. At a 5x peak that is under 5 per second.

That single number eliminates the load balancer as a scaling argument, eliminates the cache, eliminates partitioning, and eliminates the queue for throughput reasons. The most common error at this step is to compute the number and then draw the boxes anyway.

??? note "Before reading on, say why block one had to come first"

    Because every box in blocks two and three is justified or eliminated by that rate. If you draw first and estimate second, the estimate can only confirm what you drew, and the interviewer can see that happening.

**Block two, purpose: pick the smallest correct storage shape.** Files are large and served by URL, so they go to an object store. Metadata (title, owner, timestamps, object key) is small, relational and queried by title, so it goes in one relational primary. That is two boxes, and the toolkit row for the object store is what justifies the split.

The common error here is putting file bytes in the database because it is one fewer box. It is one fewer box and it doubles your backup size, poisons your cache and slows your replication.

**Block three, purpose: find the one thing that is actually slow.** Extracting text from a PDF for search takes seconds. The caller does not need it to render the upload confirmation. That is exactly the queue row: acknowledgement now, work completes within a stated delay.

So the design is: app tier, object store, relational primary, one bounded queue, one worker. Five boxes, no cache, no partitioning, no CDN.

??? note "Before reading on, say what the interviewer is most likely to push on"

    The search itself. Title search on 4,000 employees' documents is a relational query with an index; full text search over extracted content is a different tool. The move is to state the fork out loud rather than to pick silently: "title only is an index on one column, full text over extracted content is a search index, and I would want to know which the product means before I add a fourth store."

**Block four, purpose: name the breaking point before being asked.** The first thing to break is not throughput, it is the object store permission model, at the moment one document must be visible to some employees and not others. Observation: a support ticket, not a graph. That is worth saying, because naming a non-capacity breaking point is uncommon and it reads as experience.

### 1e. Fade the scaffold

**Faded one.** Prompt: "Design a service that emails a receipt after every purchase." Purchases are 300,000 a day (GIVEN). Blocks one and two are given; produce block three.

- Block one: 300,000 divided by 86,400 is about 3.5 per second, at a 4x peak about 14 per second. Rules out partitioning, rules out a cache, rules out horizontal anything for throughput.
- Block two: purchases already exist in a relational store. Receipts need a record of what was sent, so one small table.
- Block three: ?

??? note "Show answer"

    A bounded queue plus a worker, and an idempotency key on the send. The email provider is a remote dependency that can be slow or down, and the purchase must commit whether or not the email goes out, so the send cannot be synchronous with the purchase write.

    The key detail most answers miss: the queue message must be written in the same transaction as the purchase, otherwise a crash between the two produces a purchase with no receipt or a receipt with no purchase. That is the outbox pattern, and it is the fourth of the five patterns below.

    The idempotency key is the receipt id, because at-least-once delivery means the worker will occasionally process the same message twice and a customer must not get two receipts.

**Faded two.** Prompt: "Design a leaderboard for a mobile game with 2 million daily players." Block one is given; produce blocks two and three.

- Block one: assume each player finishes 10 sessions a day, which is reasonable for a casual mobile title played in short bursts. 2,000,000 times 10 is 20 million score writes a day, which is 231 per second average and about 900 at a 4x peak. Reads: assume each player checks the board 5 times a day, so 10 million reads a day, 116 per second, 460 at peak.
- Blocks two and three: ?

??? note "Show answer"

    Block two, storage shape: the top 100 is one small sorted structure that fits in memory easily (100 entries), and it is read far more than it changes. Per-player best scores are 2 million rows of maybe 50 bytes, which is 100 MB, so one relational primary holds everything with room to spare. Nothing here forces partitioning.

    Block three, the interesting part: 900 writes per second against a single sorted structure is a lock convoy waiting to happen, because every write contends on the same object. The fix is that only a score above the current hundredth place can change the top 100, so the app tier reads the current cut-off and discards the rest without touching the shared structure. That converts 900 contended writes per second into a handful.

    The common wrong answer is to add a queue in front of the leaderboard. It removes the latency from the caller and keeps the contention, so the convoy just happens in the worker instead.

### 1f. Check yourself

**C1.** A prompt gives you 40 reads per second and 6 writes per second. Which of the eight blocks are you allowed to draw, and what do you say when the interviewer asks where the cache is?

??? note "Show answer"

    You are allowed a service tier and a store. Nothing else in the stated load justifies a box. If large files are involved, the object store row justifies itself on value size rather than on rate.

    What to say: "At 40 reads a second the database is not the bottleneck, so a cache would buy latency I have not been asked for and add a staleness window I would then have to reason about. I would add one when the profile shows this query taking a large share of database CPU. Say the word if you want me to design it anyway."

**C2.** Name the one block in the toolkit whose wrong use is most expensive to reverse, and say why.

??? note "Show answer"

    Hash partitioning, because the partition key gets embedded in every query, every migration and every operational runbook. Changing it later means moving all the data while serving traffic. Every other block on the list can be removed in a deploy: a cache can be bypassed, a queue can be drained and the work inlined, a CDN can be pointed away from.

### 1g. When not to use this

The toolkit itself has a threshold. Do not reach for it when the prompt is not a service.

| Situation | Why the toolkit misfires | Use instead |
|---|---|---|
| The prompt is an API contract, error model or webhook | Boxes are not the answer; resources, idempotency semantics and versioning are | [Product Architecture](product-architecture.md) |
| The prompt is a browser screen or one component | The budget is bytes shipped and first paint, not queries per second | [Frontend System Design](frontend-system-design.md) |
| The prompt is an app client with offline sync | The client is the distributed node, and the constraints are battery, network and version skew | [Mobile System Design](mobile-system-design.md) |
| The prompt is a model that ranks, scores or detects | Labels, metrics and serving decide the answer, and a cache in front of a stale model is worse than no cache | [ML System Design](ml-system-design.md) |
| The whole system fits on one machine by inspection | The correct answer is one box, and drawing seven signals you cannot tell scales apart | Say the threshold out loud and stop |

### 1h. Six estimation drills, five minutes each

Do these in block 4 to 10. Write the arithmetic and the eliminated option before opening any answer.

**E1.** A product does 250 million API calls a day. Average per second, and what does it rule out?

??? note "Show answer"

    250,000,000 divided by 86,400 is about 2,894 per second. Assume a 3x peak, which sits inside the hub's 2x to 5x band for a diurnal consumer product, giving about 8,680 per second.

    It rules out a single application process, so you need a service tier with more than one instance. It does not rule out a single primary store, because you have not yet said what fraction of those calls touch the store or whether they are cacheable. Saying that second half out loud is what separates a number that eliminates from a number that impresses.

**E2.** You store 5 million rows a day of about 400 bytes each, for 2 years. Size it and eliminate something.

??? note "Show answer"

    5,000,000 times 400 bytes is 2,000,000,000 bytes, so 2 GB a day. Two years is 730 days, so 2 times 730 equals 1,460 GB, about 1.46 TB.

    Rules out holding the dataset in memory. Does not rule out one node's disk, because 1.46 TB fits an ordinary SSD with margin, so partitioning is not yet justified by size. What might justify it is restore time, and that is a different measurement.

**E3.** The availability target is 99.95 percent, measured monthly. Minutes of downtime, and what does it rule out?

??? note "Show answer"

    30 days times 24 hours times 60 minutes is 43,200 minutes a month. 0.05 percent of 43,200 is 21.6 minutes.

    Rules out any design whose recovery involves a restore that takes longer than 21.6 minutes, because one such event spends the entire monthly budget. That is usually the real argument for replicas, and it is a much better argument than "for scale".

**E4.** A read fans out to 6 services in parallel, each with a p99 of 200 ms. What happens to the combined p99?

??? note "Show answer"

    If each call independently exceeds 200 ms one percent of the time, the chance that none of six does is 0.99 to the sixth power, about 0.941. So about 5.9 percent of requests exceed 200 ms, and the combined p99 is worse than any single dependency's p99.

    Rules out setting a user-facing p99 budget equal to a dependency's p99. Either cut the fan-out, or budget each dependency well below the user-facing target, or hedge the slowest call.

**E5.** 12,000 reads per second at a 92 percent cache hit ratio. What does the origin see, and what does a deploy do?

??? note "Show answer"

    Steady state: 8 percent of 12,000 is 960 per second at the origin. Cold cache after a deploy: the full 12,000 per second, which is 12.5 times the steady-state load.

    Rules out sizing the origin for steady state alone. Either warm the cache before an instance takes traffic, or stagger the rollout, or size the origin for a stated fraction of cold traffic and say which fraction.

**E6.** Two regions 8,000 km apart. What does a synchronous cross-region write cost, and what does that rule out?

??? note "Show answer"

    Light in fibre travels at roughly 200,000 km per second. A round trip is 2 times 8,000 divided by 200,000, which is 0.08 seconds, so about 80 ms before any routing, queueing or work.

    Rules out strong cross-region consistency on any write path with a sub-100 ms budget. It does not rule out multi-region reads, which is the distinction worth saying: "I can serve reads from both regions today; making writes strongly consistent across them costs 80 ms per write, so I would need to hear that the requirement is worth that."

### 1i. The five patterns

Five, and each carries its own threshold.

| Pattern | What it does | When not to use it |
|---|---|---|
| Cache-aside with single-flight refresh | The first miss on a key refreshes it while other readers wait or serve stale, so an expiry does not become a stampede | Below roughly 2,000 reads per second, or with no hot key at all. A single shared cache is enough, and the coordination point can stall readers if it has a bug |
| Fan-out on write, with a read-time merge for large producers | Precompute the common case, merge the expensive minority at read time | When every producer has a small audience. Pure push is simpler, and the hybrid path costs you a second code path you must keep correct forever |
| Outbox: write the event in the same transaction as the state change | Removes the dual-write gap where a crash leaves the store and the queue disagreeing | When there is only one store being written. Adding an outbox to a single-store write is machinery guarding a gap that does not exist |
| Bounded queues plus load shedding at admission | Converts an unbounded latency failure into a visible rejection you can alert on | Until peak exceeds provisioned capacity often enough that the event has a name. Below that, autoscaling with headroom is simpler and fails less strangely |
| Timeouts that shrink with depth, one retry layer, full jitter | Stops a slow leaf from exhausting every caller above it, and stops retries from multiplying load | Never skip the timeouts. Do skip the circuit breaker until you have more than one remote dependency on the path and have watched one stay slow for over a minute |

---

## The delivery clock, compressed

The full version, including how the split moves by level and six situations where following it is wrong, is on [the hub](index.md#the-clock). This is the one screen you memorise.

| Phase | 45 min round | You leave with |
|---|---|---|
| Restate, name the hard parts, scope out, ask one question | 0 to 2 | Agreement, or a cheap correction |
| Clarify and write requirements down | 2 to 9 | A functional list plus two or three numbers, tagged |
| Estimation, or the one-line skip | 9 to 11 | One or two options already dead |
| V0, fewest boxes the requirements permit | 11 to 19 | A correct small design, drawn |
| Breaking point, then V1 | 19 to 27 | What fails, at what number, how you see it |
| One deep dive | 27 to 36 | A mechanism, a derived number, a failure class |
| Failure modes, what you skipped, what you would measure | 36 to 40 | A named class and one metric |

The budget assumes a 45 minute slot loses about 5 minutes to introductions, tooling and your own closing questions, leaving 40 working minutes. That deduction is an assumption, and it is reasonable because nearly every remote round spends time on hellos and screen sharing.

### The opening, as four sentences

Restate with a number. Name the two or three hard parts. State what you are dropping. Ask the one question whose answer changes the design most. Under sixty seconds spoken, and no phase name in it.

> "So: a service that takes a long URL, returns a short one and redirects on lookup, at something like a hundred million redirects a day. The hard parts look like read latency on the redirect, key generation without collisions, and one link going viral. I am dropping analytics dashboards and custom domains unless you want them. Before I draw: do links ever expire, because that decides whether I need a deletion path at all?"

### Three recovery scripts

| Situation | Say this |
|---|---|
| You are stuck | "I am going to state my assumption and move, because I would rather be corrected than stall. I am assuming reads dominate at roughly a hundred to one. If that is wrong, the whole read path changes and I want to know now." |
| The interviewer disagrees | "You are right that this breaks under a hot key. That changes my partition choice, so let me redraw that part." |
| You are running out of time | At minute 34: "I want to keep five minutes for failure modes and what I skipped, so I am closing this dive here." |

!!! danger "WHEN TO BREAK THIS"

    1. **The interviewer opens with the deep dive.** If the prompt is "assume the feed exists, shard it", there is no V0 phase. Spend the whole clock on the question asked.
    2. **The interviewer interrupts.** That is the highest-value signal in the round, because it tells you what is being graded. Follow it immediately, and never say "I will get to that later".
    3. **You are visibly behind at minute 20.** Do not compress every remaining phase equally. Drop the deep dive, say you are dropping it, and protect the failure-modes block, which is where senior signal concentrates.

---

## Sprint case study 1: URL shortener

Done at sprint pace: about 25 minutes of thinking, not 90. The full nine-part arc is here, compressed, and the section ends by naming what a fuller answer adds.

### The prompt (shortener)

"Design a service that turns a long URL into a short link, and sends anyone who visits the short link to the original."

### Clarifiers that fork the design (shortener)

| Question worth asking | If A, the design becomes | If B, the design becomes |
|---|---|---|
| Are codes system-generated or user-chosen? | Generated: you own the key space and can use a counter plus a scramble | User-chosen allowed: you need a uniqueness check, a reserved-word blocklist and a shared namespace |
| Is per-click analytics a product feature? | No: you can return 301 and browsers cache the redirect away | Yes: you must return 302 or 307, so every click reaches you and your read traffic is what it is |
| Do links expire? | Never: keys are write-once, no reclaim policy | Yes: you need tombstones and a rule on whether a code may be reissued |
| Single region or global? | Single: one primary plus a cache | Global: regional read replicas, single-homed writes, and a stated staleness for fresh links |
| What is the read to write ratio? | Near 1 to 1: skip the cache tier | 100 to 1 or higher: the whole design is a read path |

### Requirements (shortener)

Functional: create a code for a URL and return it; resolve a code and redirect; record a click per resolution.

| Non-functional | Value | Tag |
|---|---|---|
| New links per day | 2 million | ASSUMED, a mid-size product. The interviewer usually supplies this, so ask |
| Read to write ratio | 200 to 1 | ASSUMED, because a link is created once and shared many times |
| Redirect availability | 99.95 percent monthly | ASSUMED; a dead redirect breaks every page that embeds it |
| Redirect latency, server side | under 50 ms at p99 | ASSUMED; the redirect is a hop the user did not ask for |
| Retention | 3 years | ASSUMED; links get embedded in archived pages |
| Stored row size | about 200 bytes | ASSUMED: a 7 byte code, roughly 120 bytes of URL, an 8 byte owner id, an 8 byte timestamp, plus index and row overhead |

### Numbers that eliminate (shortener)

| Calculation | Result | What it rules out |
|---|---|---|
| 2,000,000 divided by 86,400, then 4x for peak | 23 writes per second average, 92 at peak | Rules out write-path sharding, a write coordinator, and any queue on the create path. One primary absorbs 92 inserts per second |
| 23 times 200 | 4,600 reads per second average, 18,400 at peak | Rules out serving redirects from the primary alone at p99, and rules out writing click events synchronously on the redirect path |
| 2,000,000 times 365 times 3 | 2.19 billion rows | Rules out an in-memory-only store |
| 2.19 billion times 200 bytes | 438 GB | Rules out partitioning for size. 438 GB fits one SSD with margin, so any shard would be for blast radius, not capacity |
| Assume 5 percent of codes take most clicks, so 109.5 million times 200 bytes | 21.9 GB | Rules out a cache cluster. Two nodes hold the hot set. The 5 percent is an assumption, reasonable because link sharing produces a heavy popularity skew |

### V0: the smallest thing that works (shortener)

??? note "Predict first: which component in V0 fails first, and at what number?"

    The primary store, and not on writes. At 92 writes per second it is idle on the write path. It fails on 18,400 point reads per second at peak, and you will see it as primary CPU dominated by the resolve query while the write rate stays flat.

```
+---------+     +---------------+     +----------------+
| Browser |---->| App tier      |---->| Postgres       |
|         |<----| stateless     |<----| code -> url    |
+---------+     +---------------+     | 438 GB, PK     |
                                      +----------------+
```

Caption: V0 has three boxes. A browser calls a stateless app tier, which does one primary-key lookup in a single relational store and returns a redirect. No cache, no queue, no partitioning, because nothing in the stated numbers justifies them yet.

!!! warning "When not to use even this much"

    Below roughly 5 writes and 500 reads per second, drop the separate app tier count to two instances and stop. At that rate you are designing for a load you have not got, and every extra box is a failure mode you now own.

### The breaking point, then V1

**Breaking point: 18,400 reads per second at peak.** What fails: primary CPU saturates on point lookups, connection pool wait climbs, and redirect p99 goes from single-digit milliseconds to seconds. How you observe it: query-level CPU attribution on the resolve statement, plotted against pool wait time at p99. This is capacity, not a herd.

```
+---------+   +-----------+   +---------+   +-------------+
| Browser |-->| App tier  |-->| Cache   |-->| Postgres    |
|         |   | local LRU |   | 22 GB   |   | primary +   |
+---------+   | 1 s TTL   |   +---------+   | replicas    |
              +-----------+                 +-------------+
                    |
                    v
              +-----------+
              | Click     |
              | queue     |
              +-----------+
```

Caption: V1 adds a 22 GB shared cache sized from the working set, a tiny per-instance cache with a one second time to live, read replicas, and a queue for click events. Five boxes, each added because a number above demanded it.

| Change | Justified by | Cost you accept |
|---|---|---|
| Shared cache, 22 GB | The 21.9 GB working set calculation | A component that can fail, and cache and store can now disagree |
| Local cache, 1 second time to live | Hot-key protection, derived in the first dive | Up to one second of staleness on link edits |
| Read replicas | Primary CPU dominated by point reads | Replication lag, so a brand new link can 404 briefly. Fix by writing the code into the cache on create |
| Click events onto a queue | Synchronous click writes double the work on the read path | Click counts become eventually consistent. Say so before you are asked |

### Deep dive one: sizing the cache and the hot-key ceiling

The dive is chosen because this prompt's whole interest is the read path. The two other sprint case studies dive elsewhere on purpose.

The working set is 21.9 GB, derived above. That fits two cache nodes with a replica, so the sizing question is settled in one line. The interesting number is the hot-key ceiling.

One viral link can take a large share of 18,400 reads per second. In a hash-partitioned cache that key lives on one node, so that node absorbs the spike while its peers idle. The local per-instance cache is what bounds it: with 80 app instances and a one second time to live, each instance can miss at most once per second for that key, so the shared cache sees at most 80 requests per second for it no matter how popular the link becomes.

The general form is worth memorising, because it generalises to every hot-key problem: shared-tier load per key equals instance count divided by local time to live. Raise the time to live to cut load, and pay for it in staleness.

### Deep dive two: the redirect status code is a traffic decision

A 301 is permanent and browsers cache it, so repeat clicks never reach you. A 302 or 307 keeps every click on your servers. Redirect semantics are defined in RFC 9110 (June 2022).

That one choice produced the 18,400 reads per second in the first place. If analytics is dropped, 301 removes most of your repeat read traffic, and with it the cache tier, the replicas and the queue. It also removes your ability to ever disable a malicious link, which is the argument against it.

Saying this out loud is worth more than the cache sizing, because it shows you can see a product decision setting an infrastructure bill.

### Failure modes and what you would measure (shortener)

| Class | How it shows up here | What you do |
|---|---|---|
| Cache stampede | A hot key expires and every instance refreshes together | Single-flight refresh, or a random offset on the time to live |
| Hot key | One viral link saturates one cache node | The local tier with a short time to live, bounding per-key load at instance count divided by the time to live |
| Thundering herd | Empty local caches after a rolling deploy | Stagger the rollout, and warm before taking traffic |
| Retry storm | The store slows, clients retry, offered load rises as capacity falls | One retry layer, a retry budget as a fraction of base traffic, full jitter |

Publish three indicators and no more: redirect success rate, redirect server-side p99, and shared cache hit ratio. Alert on error budget burn for redirect availability, not on raw error count. The 99.95 percent target is 21.6 minutes a month, derived as 0.05 percent of 43,200 minutes.

Re-check against the requirements: p99 under 50 ms holds at a high hit ratio because the miss path is one point lookup; 18,400 reads per second holds and could grow tenfold without a shape change, because the local tier caps per-key load; 438 GB holds on one primary, so the retention requirement needs no partitioning.

### Where this is a simplification (shortener)

- **Abuse is the actual hard problem.** A shortener is a phishing tool by default, so real services check destinations at create time and re-check later. That makes the mapping mutable, which breaks the write-once assumption the design leans on.
- **Bots inflate click counts.** Link preview crawlers fetch every pasted link, often repeatedly.
- **Key generation is skipped here on purpose.** The sprint uses a counter passed through a reversible scramble and moves on. The full treatment, including why hash-and-truncate needs a uniqueness check anyway, the collision arithmetic and the block allocator, is in [Modern, case study 1](modern-system-design.md#case-study-1-url-shortener).

### What a fuller answer adds (shortener)

Key space arithmetic and code length; a key allocation service and when not to build one; negative caching against scanners; custom domains and certificate issuance; and the deletion path that retention law requires. Read the deep version after the round.

### Level delta (shortener)

| Dimension | Mid | Senior | Staff |
|---|---|---|---|
| Estimation | Produces the numbers | Uses each number to kill an option | Notices 92 writes per second means the write path needs no design at all, and reallocates the time |
| Cache | Proposes one | Sizes it at 22 GB from the working set | Adds the local tier with the derived per-key ceiling, and names the staleness ticket it will generate |
| Status code | Says 301 or 302 | Picks 302 because analytics is required | Points out that this choice sets the entire read bill, and offers 301 as a cost lever |
| Failure handling | Mentions retries | Names stampede and hot key with fixes | Names the retry budget as a fraction of base traffic, not a per-request count |

Mid to senior: every number now removes an option and every box has a measurement behind it. Senior to staff: a product decision is priced, and at least one component is removed on evidence.

---

## Sprint case study 2: home timeline

Chosen second because it is the prompt where the toolkit's fan-out pattern earns its place, and because its interesting number is a crossover rather than a capacity.

### The prompt (timeline)

"Design the home timeline for a social app: the list of recent posts from the accounts a user follows."

### Clarifiers that fork the design (timeline)

| Question worth asking | If A, the design becomes | If B, the design becomes |
|---|---|---|
| Is the timeline chronological or ranked? | Chronological: a merge over recent posts, and the design is about fan-out | Ranked: a retrieval and scoring pipeline, and this is the wrong course. Route to [ML System Design](ml-system-design.md) |
| How fresh must a new post be for followers? | Minutes: fan-out can lag and batch | Seconds: fan-out throughput becomes a hard constraint and sets your crossover |
| What is the largest follower count? | Thousands: pure fan-out on write is fine | Millions: you need a read-time merge for the largest accounts |
| Can the timeline be capped? | Yes, most recent few hundred: storage is bounded per user | No, infinite scroll to the beginning: you need a fallback read path anyway |
| Are edits and deletes visible immediately? | Eventually: precomputed rows can carry stale copies | Immediately: precompute ids only, hydrate content at read time |

### Requirements (timeline)

Functional: post; read the timeline in reverse chronological order; page backwards through it.

| Non-functional | Value | Tag |
|---|---|---|
| Daily active users | 50 million | GIVEN in the prompt framing |
| Timeline reads per user per day | 8 | ASSUMED; a habitual app is opened a handful of times a day |
| Posts per user per day | 0.2 | ASSUMED; most users read far more than they post |
| Median accounts followed | 200 | ASSUMED; typical of a mature consumer graph |
| Freshness budget for a new post | 30 seconds | ASSUMED; a post that takes minutes to appear reads as broken |
| Timeline read p99 | under 200 ms server side | ASSUMED; it is the first screen of the app |

### Numbers that eliminate (timeline)

| Calculation | Result | What it rules out |
|---|---|---|
| 50,000,000 times 8, divided by 86,400, then 3x peak | 4,630 reads per second average, 13,900 at peak | Rules out serving each read with a scatter-gather over followed accounts, as the next row shows |
| 13,900 peak reads times 200 followed accounts | 2.78 million index probes per second | Rules out pull at read entirely. This is the number the whole design turns on |
| 50,000,000 times 0.2, divided by 86,400, then 3x peak | 116 writes per second average, 348 at peak | Rules out any write-path complexity for throughput reasons. The write path is cheap; its fan-out is not |
| 348 peak posts times 200 followers | 69,600 timeline inserts per second at peak | Rules out a relational primary for timeline rows. This is a wide-column or key-value workload |
| One post by an account with 10 million followers, at 69,600 inserts per second | 144 seconds of fan-out for one post | Rules out pure fan-out on write, because 144 seconds exceeds the 30 second freshness budget |
| Freshness budget 30 s times 69,600 inserts per second | 2.09 million followers | The crossover: above this follower count, an account cannot be fanned out inside the budget, so it must be merged at read time |

That last row is the point of the whole case study. The threshold is derived from your own freshness budget and your own fan-out throughput, not memorised.

### V0: the smallest thing that works (timeline)

??? note "Predict first: what is the first thing that breaks in a pull-at-read design?"

    Not storage and not the post write. The timeline read, because each read touches one index range per followed account. The 2.78 million probes per second above is the failure, and you can see it as read p99 rising in proportion to how many accounts a user follows.

```
+--------+    +-------------+    +------------------+
| Client |--->| Feed API    |--->| posts table      |
|        |<---| merge top50 |<---| idx author,time  |
+--------+    +-------------+    +------------------+
```

Caption: V0 is three boxes. The client asks the feed API, which reads the most recent posts for each followed author from one index and merges them in memory. This is correct for a small graph and is the design you should draw first even though you already know it breaks.

!!! warning "When not to move off V0"

    If the median user follows under about 20 accounts, pull at read is the right answer forever. 13,900 reads times 20 is 278,000 probes per second, which one replicated store handles. Precomputing timelines below that point buys you a second copy of every post and a backfill on every schema change.

### Breaking point, then V1 and V2 (timeline)

**Breaking point one: 2.78 million index probes per second.** Observation: timeline p99 correlated with followed-account count per user, while the store's CPU is dominated by index scans rather than by writes.

```
+--------+   +-----------+   +------------------+
| Author |-->| Post svc  |-->| posts table      |
+--------+   +-----------+   +------------------+
                  |
                  v
           +--------------+      +----------------+
           | Fan-out      |----->| timelines      |
           | bounded queue|      | one row per    |
           +--------------+      | user, capped   |
                                 +----------------+
                                        ^
+--------+   +-----------+              |
| Reader |-->| Feed API  |--------------+
|        |   | + merge   |
+--------+   | big posts |
             +-----------+
```

Caption: V1 and V2 together. A post is written once, then a bounded fan-out queue writes a capped list of post ids into each follower's timeline row, and the feed API reads one row per request. V2 is the extra arrow: accounts above the derived crossover skip fan-out, and the feed API merges their recent posts at read time.

**Breaking point two: one account above 2.09 million followers.** What fails is the freshness budget, not the system. Observation: alert on oldest-message age in the fan-out queue, never on queue depth, because depth tells you nothing about how late the last user will be.

| Change | Justified by | Cost you accept |
|---|---|---|
| Precomputed timeline rows, capped at 500 ids | 2.78 million probes per second at read | A second copy of the id list, and a backfill whenever the row format changes |
| Store post ids, not post bodies | Edits and deletes must not leave stale copies in millions of rows | One extra hydration read per timeline load, served from cache |
| Bounded fan-out queue | Fan-out is bursty and must not consume the store | Backpressure, so a burst delays fan-out rather than breaking the write path |
| Read-time merge above 2.09 million followers | The 144 second fan-out exceeds a 30 second budget | A second code path that must stay correct forever, which is a real cost worth naming |

### Deep dive one: the crossover, derived live

This dive exists because the crossover is the one number in the prompt an interviewer cannot look up, and deriving it in front of them is the strongest thing you can do here.

The chain is four steps and takes about ninety seconds on a whiteboard.

- Peak posts per second, from daily active users and posts per user: 348.
- Median fan-out per post, from the median follower count: 200, giving 69,600 inserts per second at peak.
- The freshness budget, stated as a requirement: 30 seconds.
- Multiply: 30 times 69,600 is 2.09 million followers. Above that, this account cannot be fanned out in time.

What to say next matters as much as the number: "so the hybrid boundary is not a magic constant, it is my freshness budget times my fan-out throughput. If the product accepts five minutes of lag for celebrity posts, the boundary moves by a factor of ten and I may not need the second path at all."

### Deep dive two: paging a list that changes while you read it

Chosen because it is the failure users actually report, and because it is a different class from anything in case study 1.

Offset pagination breaks on a mutating list. If three posts arrive between page one and page two, offset 50 now points three items further back, so the reader sees three duplicates. If posts are deleted, they skip items instead.

The fix is a cursor: an opaque token encoding the sort key of the last item returned, which for a timeline is the pair (created at, post id). The post id breaks ties when two posts share a timestamp, which happens more often than people expect at 348 writes per second.

| Approach | Behaviour on a mutating list | When it is still fine |
|---|---|---|
| Offset and limit | Duplicates on insert, skips on delete | A list that does not change, such as an archive |
| Cursor on (created at, post id) | Stable, because the position is a value not a count | The normal answer here |
| Snapshot the id list per session | Perfectly stable, and goes stale | Sessions short enough that staleness is invisible |

### Failure modes and what you would measure (timeline)

| Class | How it shows up here | What you do |
|---|---|---|
| Unbounded queue growth | Fan-out falls behind and timelines silently go stale by hours | Alert on oldest-message age, autoscale consumers on age, cap the age after which work is dropped |
| Hot key | One account's timeline row is read far more than any other | Cache the hydrated page, and read that row from a replica set |
| Head-of-line blocking | One 2 million follower fan-out job stalls every ordinary post behind it | Separate queues by cost class, so small fan-outs never wait behind large ones |
| Thundering herd | A scheduled push wakes millions of clients into the timeline read at once | Jitter the schedule per client rather than sending on a shared minute |

Indicators: timeline read p99, fan-out oldest-message age, and hydration cache hit ratio. Re-check: 200 ms p99 holds because a read is one row plus a cached hydration; 13,900 reads per second holds because it is one row read per request; the 30 second freshness holds for everyone below the crossover and is met by the merge path above it.

### Where this is a simplification (timeline)

- **Ranking changes everything.** Most real timelines are ranked, which turns this into a retrieval and scoring problem. That is a different course: [ML System Design](ml-system-design.md).
- **The follow graph is its own system.** Reading a user's followers at fan-out time is a scatter-gather over a graph store, and it is the second hardest thing here.
- **The full fan-out cost model,** including the arithmetic of when precomputation stops paying and how the two paths are kept consistent, is in [Modern, case study 3](modern-system-design.md#case-study-3-news-feed).

### What a fuller answer adds (timeline)

The cost model in money per million timeline reads; duplicate suppression across pages; the follow-graph store; backfill order when the timeline row format changes; and the multi-region read story. All of it is post-round reading.

### Level delta (timeline)

| Dimension | Mid | Senior | Staff |
|---|---|---|---|
| Fan-out choice | Names push or pull | Picks a hybrid and says why | Derives the crossover from the freshness budget and offers to move the budget instead |
| Timeline row | Stores post bodies | Stores ids and hydrates | Names the backfill cost of the row format as the reason ids win |
| Queue | Adds a queue | Bounds it and alerts on age not depth | Separates queues by cost class and names head-of-line blocking as the reason |
| Paging | Uses offset | Uses a cursor | Explains the tie-break on post id and the support tickets its absence produces |

Mid to senior: the design stops being a shape and starts being a set of numbers. Senior to staff: the requirement itself becomes negotiable, and operational consequences drive the structure.

---

## Sprint case study 3: chat with delivery receipts

Chosen third because it is the only prompt in the sprint where the interesting resource is connection state rather than storage or fan-out, so its dives cannot repeat the first two.

### The prompt (chat)

"Design one-to-one and small-group messaging with delivery receipts, for a mobile and web client."

### Clarifiers that fork the design (chat)

| Question worth asking | If A, the design becomes | If B, the design becomes |
|---|---|---|
| Is history stored forever or is this ephemeral? | Stored: message storage sizing decides the store choice | Ephemeral: the design is almost entirely a connection problem |
| Group size ceiling? | Small, say 100: fan-out per message is trivial | Large, say 100,000: this becomes a broadcast problem closer to case study 2 |
| Must ordering be consistent across devices? | Best effort: timestamps are fine and the design shrinks | Consistent: you need a per-conversation sequence, which is the second dive |
| Is end-to-end encryption required? | No: the server can index and search content | Yes: the server stores opaque blobs, no server-side search, and key distribution becomes the hard part |
| Do clients need offline catch-up? | No: a client that misses a message misses it | Yes: you need durable per-conversation cursors and a resume path |

### Requirements (chat)

Functional: send a message to a conversation; deliver to all participants' connected devices; store history; report delivered and read receipts.

| Non-functional | Value | Tag |
|---|---|---|
| Daily active users | 10 million | GIVEN in the prompt framing |
| Messages sent per active user per day | 40 | ASSUMED; messaging is bursty and heavy for active users |
| Fraction of users connected at peak | 30 percent | ASSUMED; a single-region consumer app concentrates use in waking hours |
| Message record size | about 400 bytes | ASSUMED: roughly 200 bytes of body, a 16 byte message id, an 8 byte conversation id, an 8 byte sender id, an 8 byte timestamp, plus envelope and index overhead |
| Delivery latency to a connected device | under 500 ms at p99 | ASSUMED; above this the conversation feels broken |
| Per-connection memory | about 20 KB | ASSUMED; socket buffers plus a small session record. Reasonable because the per-connection state here is an identity and a cursor, not a buffer of history |

### Numbers that eliminate (chat)

| Calculation | Result | What it rules out |
|---|---|---|
| 10,000,000 times 40, divided by 86,400, then 3x peak | 4,630 messages per second average, 13,900 at peak | Rules out any per-message coordination or consensus. This is an ordinary write rate |
| 10,000,000 times 30 percent | 3 million concurrent connections at peak | Rules out one connection server, and rules out keeping session state only inside the connection process |
| 3,000,000 times 20 KB | 60 GB of connection state | Rules out holding all sessions on one node, and sizes the gateway tier |
| 3,000,000 connections divided by an assumed 50,000 per node | 60 gateway nodes | Rules out treating gateways as ordinary stateless instances. The 50,000 is an assumption, reasonable because it keeps per-node connection memory near 1 GB (50,000 times 20 KB) and blast radius at under 2 percent of connections |
| 400,000,000 messages per day times 400 bytes | 160 GB per day, 58 TB per year | Rules out a single relational primary for history, and rules out keeping all of it hot |
| 3,000,000 connected clients polling once every 2 seconds, against 13,900 useful messages per second | 1.5 million polls per second | Rules out polling at this scale, since over 99 percent of responses are empty. This is what justifies persistent connections |

### V0: the smallest thing that works (chat)

??? note "Predict first: why is polling the right V0 even though you already know it breaks?"

    Because it is correct, it is three boxes, and it makes the breaking point measurable. Drawing a gateway tier before showing that polling fails means the interviewer never sees you derive anything. The number that kills polling, 1.5 million requests per second of mostly empty responses, is more persuasive than the assertion that chat needs sockets.

```
+--------+    +---------------+    +-----------------+
| Client |--->| Message API   |--->| messages table  |
| polls  |<---| poll for new  |<---| conv id, seq    |
| 2 s    |    +---------------+    +-----------------+
+--------+
```

Caption: V0 is three boxes. Clients poll a stateless message API every two seconds, which reads any messages newer than the client's cursor from one store keyed by conversation and sequence. Correct, simple, and it fails at a number you can state.

!!! warning "When not to move off V0"

    Below roughly 20,000 concurrent users, polling every few seconds is 10,000 requests per second of cheap reads, and a persistent connection tier is a large operational cost for a latency improvement nobody asked for. The measurement that justifies sockets is poll traffic exceeding the useful traffic by a large factor, plus a delivery latency requirement that polling cannot meet.

### Breaking point, then V1 (chat)

**Breaking point: 1.5 million polls per second for 13,900 useful messages.** What fails is efficiency and latency together: over 99 percent of poll responses are empty, and a two second poll interval cannot meet a 500 ms delivery target. Observation: ratio of empty to non-empty poll responses, plotted against delivery latency.

```
+--------+   +--------------+   +------------------+
| Client |==>| Gateway tier |-->| Message service  |
| socket |   | 60 nodes     |   | assigns seq      |
+--------+   +--------------+   +------------------+
                   ^                     |
                   |                     v
             +-------------+     +------------------+
             | Session map |     | Message store    |
             | user -> gw  |     | conv id, seq     |
             +-------------+     +------------------+
```

Caption: V1 has five boxes. Clients hold a persistent socket to one of 60 gateway nodes; a session map records which gateway holds each user; the message service assigns a per-conversation sequence number and writes to a partitioned store; delivery is a lookup in the session map followed by a push to the right gateway.

| Change | Justified by | Cost you accept |
|---|---|---|
| Persistent connections, 60 gateway nodes | 1.5 million polls per second for 13,900 useful messages | Connection state you must now shed and rebalance, which is a genuinely new class of operational work |
| Session map, user to gateway | Delivery needs to find a connection without broadcasting to 60 nodes | A store on the delivery path, so it needs to be fast and its staleness needs a fallback |
| Per-conversation sequence number | Ordering must be consistent across a user's devices | One assignment point per conversation, which is the second dive |
| Partitioned message store by conversation id | 58 TB per year | Cross-conversation queries become scatter-gather, which is fine because nobody asks for them |

**Second breaking point: a gateway node restarts and drops 50,000 connections at once.** All of them reconnect immediately, and 60 nodes worth of coordinated restarts is a herd that can take the tier down. Observation: reconnect rate per second against deploy markers. The fix is full jitter on reconnect plus resume-from-sequence so a reconnect is cheap.

### Deep dive one: where session state lives, and how a message finds a connection

Chosen because connection routing is the thing this prompt has that neither of the other two does.

Three options, and the choice is a trade between a lookup and a broadcast.

| Option | Delivery cost | Failure behaviour | When it wins |
|---|---|---|---|
| Session map in a shared fast store | One lookup per delivery | A stale entry sends to the wrong gateway, so the gateway must reject and the sender must retry once | The default at this scale |
| Broadcast to all gateways | Zero lookups, 60 messages per delivery | No stale state to go wrong | Only with a small gateway tier, say under 10 nodes |
| Consistent hash of user id to gateway | Zero lookups, computed | Every node change remaps a fraction of users, and a user must reconnect to the right node | When connections are cheap to move and the tier size is stable |

The number that decides it: gateway node count. At 60 nodes, broadcast costs 60 times the delivery work for zero lookups, which is a bad trade. At 5 nodes it is a good one, and saying that threshold out loud is the point.

The stale-entry case is the detail worth naming: the session map is updated on connect and on disconnect, and disconnects are not always clean. So the gateway must verify it actually holds the connection and reject if not, and the sender falls back to storing the message for offline delivery. Without that check you get silent message loss, which is the worst failure in a chat system.

### Deep dive two: ordering, and why wall-clock timestamps are the wrong tool

Chosen because it is a correctness question rather than a capacity one, which makes it a different shape from every other dive in this sprint.

Two devices send into the same conversation at nearly the same moment. If each message carries the sending server's wall-clock timestamp and clients sort by it, two servers whose clocks differ by even tens of milliseconds can produce different orders on different devices. Ordinary clock drift is enough. That is the clock skew failure class.

The fix is a monotonic sequence number assigned per conversation by whichever component owns that conversation's writes. Clients sort by sequence, not by time, and the timestamp becomes display metadata only.

| Property | Mechanism | What it prevents |
|---|---|---|
| Same order on every device | Per-conversation sequence, assigned once | Two devices showing the same two messages in different orders |
| Gap detection | Client notices a missing sequence number | Silent loss, because the client can ask for the gap explicitly |
| Cheap offline catch-up | Client stores its last sequence per conversation | Replaying the whole history on reconnect |
| Receipts are just cursors | Delivered and read are sequence numbers per participant | A separate receipt record per message per participant |

That last row is the compression worth showing: a read receipt is not a message, it is a number, so a participant reading 400 messages produces one update rather than 400.

!!! warning "When not to use a per-conversation sequence"

    If the product accepts best-effort ordering, which some notification-style feeds do, the sequence assignment is a single ordering point per conversation that you now have to keep available. Below a few messages per second per conversation with no multi-device requirement, sorting by server timestamp is genuinely enough, and the sequence is machinery guarding a bug nobody would see.

### Failure modes and what you would measure (chat)

| Class | How it shows up here | What you do |
|---|---|---|
| Thundering herd | A gateway restart sends 50,000 clients into an instant reconnect | Full jitter on reconnect, staggered deploys, resume from a sequence number rather than a full sync |
| Clock skew | Two devices show the same conversation in different orders | Sort by sequence, never by wall-clock time |
| Head-of-line blocking | One large group's fan-out delays one-to-one messages behind it | Separate delivery queues by conversation size class |
| Gray failure | A gateway holds connections but stops delivering, and health checks still pass | Health checks that exercise a real delivery path, and outlier ejection on delivery latency |

Indicators: delivery latency from send acknowledgement to device receipt at p99, connection count per gateway, reconnect rate, and undelivered message age. Re-check: 500 ms p99 holds because delivery is one session lookup plus one push; 3 million connections hold across 60 nodes at the assumed 50,000 each; 58 TB per year holds in a partitioned store and not in a single primary.

### Where this is a simplification (chat)

- **End-to-end encryption changes the whole shape.** The server stores opaque blobs, cannot search, and key distribution becomes the hard problem.
- **Group membership changes mid-conversation** raise questions about history visibility that this arc skips entirely.
- **Multi-device sync and offline catch-up** get one line here and a full treatment in [Modern, case study 4](modern-system-design.md#case-study-4-chat-and-presence).

### What a fuller answer adds (chat)

Presence as its own subsystem with its own fan-out problem; push notification delivery to a device that is not connected; the media path for images and video; and a migration story for the sequence assignment when a conversation gets too hot.

### Level delta (chat)

| Dimension | Mid | Senior | Staff |
|---|---|---|---|
| Transport | Says WebSockets | Derives them from 1.5 million polls per second | Names the operational cost of connection state as the thing being bought, not the latency |
| Ordering | Uses timestamps | Uses a per-conversation sequence | Names clock skew as the class and shows receipts collapsing into cursors |
| Session routing | Assumes a lookup works | Handles the stale entry and the fallback | Gives the node-count threshold at which broadcast beats a lookup |
| Deploys | Not mentioned | Staggers restarts | Sizes the reconnect herd from connections per node and jitters it |

Mid to senior: each component is derived from a number rather than recognised from a pattern. Senior to staff: the failure of the mechanism, not the mechanism, drives the design.

---

## Timed drills

Five prompts. Three are the same shape as the case studies; two are deliberately different and will feel worse. Do D1 to D3 in blocks 18 to 34 and D4 to D5 in the taper.

### The stopwatch protocol

Set an actual timer. Speak out loud, do not think silently, because the round is a spoken performance and silent practice does not train it.

| Minutes | Do this |
|---|---|
| 0 to 4 | Restate with a number, name the hard parts, scope something out, ask one forking question |
| 4 to 7 | Write requirements down, tagged GIVEN, ASSUMED or DERIVED |
| 7 to 9 | Two numbers, each followed by the option it kills, or one sentence declining |
| 9 to 15 | Draw v0 with the fewest toolkit boxes the requirements permit |
| 15 to 21 | Name the breaking point (what, at what number, how observed), then V1 |
| 21 to 25 | One failure class, one metric you would watch, one thing you skipped and why |

Those six segments sum to 25 minutes. Score yourself against the rubric immediately, while you still remember what you said.

### The rubric

| Criterion | Below bar | At bar | Strong | Standout |
|---|---|---|---|---|
| Problem framing and scoping | Started drawing before clarifying | Asked questions, but some changed nothing | Every question forked the design, and something was scoped out explicitly | Renegotiated a requirement and showed what that bought |
| Design and trade-off reasoning | Components appeared without justification | Each box has a reason | Each box has a measurement, and one box was removed | Named the threshold at which the rejected option would win |
| Depth in one area | Everything at the same shallow depth | One area went deeper than the rest | One area held for six minutes with a mechanism and a number | The dive was chosen because this problem made it interesting, and that choice was stated |
| Communication and drive | Monologued or waited to be asked | Steady narration, some check-ins | Announced decisions rather than phases, and drove the clock | Handled a disagreement by updating the design rather than defending it |

### D1: rate limiter for a public API

??? note "Show model answer sketch"

    Clarify first: per API key, per IP or per tenant plan? Is the limit shared across a fleet, or per instance? Is a burst allowed? Those three fork the design completely.

    Numbers that eliminate: assume 20,000 requests per second across 100,000 keys. A counter per key per window is 100,000 small records, which is well under a gigabyte, so a distributed counter store is not forced by size. It is forced, if at all, by the need for one shared view across instances.

    V0: token bucket held per instance, with the limit divided by instance count. Two boxes. It is wrong at the edges (a client hitting one instance repeatedly gets a fraction of its quota) and correct enough for a generous limit.

    Breaking point: uneven client-to-instance distribution, visible as clients being limited well below their stated quota. V1 moves the counter to a shared fast store with an atomic increment, and keeps a small local bucket to absorb bursts without a round trip per request.

    Failure class: hot key, when one large tenant's counter lives on one node. Fix by sharding that tenant's counter into N sub-counters, each with one Nth of the quota.

    What you skipped: business-state limits (plan, balance) belong at the service, not the edge, and you should say which half of the limiting lives where.

??? note "Show two wrong answers and what each reveals"

    **Wrong answer one: "use a sliding window log."** It is a real algorithm, and it stores a timestamp per request. At 20,000 requests per second with a one minute window that is 1.2 million timestamps in memory at all times, for a decision that a token bucket makes with two numbers. The misconception is choosing the most precise algorithm rather than the one whose memory cost is bounded.

    **Wrong answer two: putting the limiter only at the edge.** The misconception is treating rate limiting as purely volumetric. A request that is cheap to receive and expensive to serve, such as an export, is not controlled by a request-count limit at all, and per-tenant quotas need business state the edge does not have.

### D2: metrics ingestion for 200,000 hosts

??? note "Show model answer sketch"

    Clarify: how many series per host, what resolution, and what retention? Those three multiply into the only number that matters.

    Numbers that eliminate: assume 500 series per host at 10 second resolution. 200,000 times 500 is 100 million series, and at one point per 10 seconds that is 10 million points per second. That single number rules out any per-point row in a relational store and rules out synchronous indexing.

    Assume 16 bytes per point uncompressed. 10 million times 16 is 160 MB per second, which is 13.8 TB a day. That rules out full-resolution retention beyond days without downsampling, and downsampling therefore becomes a requirement rather than an optimisation.

    V0: agents push to an ingest tier that appends to a columnar time-series store partitioned by time. Three boxes.

    Breaking point: cardinality. One label with unbounded values, such as a request id, multiplies series without bound, and you observe it as series count growing faster than host count. V1 adds a cardinality budget enforced at ingest, which rejects rather than accepts, plus downsampled rollups written at write time.

    Failure class: backpressure collapse, when the ingest tier accepts more than the store can absorb. Bounded queues and rejection at admission.

??? note "Show two wrong answers and what each reveals"

    **Wrong answer one: "shard by host id."** It spreads writes evenly, which is real, but every dashboard query is "this metric across all hosts", so it turns each read into a fan-out over every shard. The misconception is partitioning for write balance without checking the read shape.

    **Wrong answer two: no cardinality limit, on the grounds that storage is cheap.** Storage is cheap and index memory is not. The misconception is treating a time-series store as a log store, when the cost is dominated by distinct series rather than by points.

### D3: typeahead suggestions

??? note "Show model answer sketch"

    Clarify: are suggestions personalised, and how fresh must new terms be? Personalisation moves this toward ranking and out of this course; freshness decides whether the index is rebuilt or updated.

    Numbers that eliminate: assume 20 million searches a day and 4 keystrokes per search that trigger a request. 80 million requests a day is 926 per second average, about 3,700 at a 4x peak. That does not rule out much, which is the point: this problem is not decided by throughput.

    Assume 10 million distinct prefixes worth caching at 200 bytes of serialised suggestions each. That is 2 GB, which rules out any argument for a distributed cache tier. One node holds the whole answer set.

    V0: a precomputed prefix-to-suggestions map in memory, served directly. Two boxes. This is the case where the simplest design is the right one, and saying so is the signal.

    Breaking point: freshness, not scale. A trending term must appear within minutes, and a full rebuild takes longer. V1 adds a small recent-terms overlay merged at query time on top of the periodically rebuilt map.

    Failure class: cold-start collapse, when a restart leaves an empty map and every request falls through to a slow path. Load the map before taking traffic.

??? note "Show two wrong answers and what each reveals"

    **Wrong answer one: designing a distributed trie in the first five minutes.** The 2 GB figure says one node holds everything. The misconception is reaching for the structure the topic is famous for instead of sizing the data.

    **Wrong answer two: rebuilding the index on every new term.** It gives perfect freshness and makes the rebuild the bottleneck. The misconception is treating freshness as binary rather than as a budget that an overlay can meet cheaply.

### D4: a running photo service, and a new constraint

**Prompt.** "We have a photo service on one relational primary at 60 percent CPU, serving 3,000 requests per second, with images in an object store. Product wants a public API with per-tenant quotas shipping next quarter. What changes?"

This one is brownfield and interleaved, and it draws on the toolkit, the hub's [trade-off atlas](index.md#the-trade-off-atlas) and the rate limiter drill above.

??? note "Show model answer sketch"

    The first move is to refuse to redesign. V0 already exists and works. The question is what the new constraint changes, and the answer is mostly not the storage.

    What changes: an authenticated API surface with keys, per-tenant quota enforcement, and a way to attribute cost and load to a tenant. What does not change: the object store, the image path, and the existing web traffic.

    The number that matters is headroom. At 60 percent CPU you have roughly 40 percent, so the honest question is what fraction of 3,000 requests per second the new API adds. If the answer is unknown, that is the first thing to instrument, not the first thing to design.

    Ordering, cheapest and most reversible first: add per-tenant attribution to existing metrics; add API keys and authentication; add quota enforcement in the service where tenant state lives; only then consider a read replica if attribution shows API reads growing.

    The blast radius sentence is what earns the round: a quota bug that rejects real traffic is worse than no quota, so quotas ship in a log-only mode first, and the switch to enforcement is a separate change with its own rollback.

??? note "Show two wrong answers and what each reveals"

    **Wrong answer one: proposing a migration to a partitioned store.** Nothing in the prompt says the store is the problem, and 60 percent CPU with an unknown growth rate is not evidence. The misconception is that a brownfield prompt is an invitation to redesign, when it is a test of restraint and sequencing.

    **Wrong answer two: putting quotas at the edge.** Per-tenant quotas depend on plan and balance, which the edge does not have. The misconception is copying the rate limiter answer without noticing the constraint changed from volumetric to business-state.

### D5: the answer is no

**Prompt.** "The team wants to add a message broker and a service mesh to an internal tool with 400 users and one deploy a week. Make the call."

??? note "Show model answer sketch"

    The correct answer is do not add either, and the way you say it decides whether it reads as wisdom or as obstruction.

    Size it first, because the argument has to be a measurement. 400 users at even 100 actions a day is 40,000 actions a day, which is under 1 per second. At that rate a broker moves work that a function call already completes inside the request, and a mesh manages traffic between services that could be one process.

    Then price the cost, which is where the strong version of this answer lives: each is a component with its own failure modes, its own upgrade cadence, and its own on-call knowledge, carried by a team whose actual bottleneck is one deploy a week.

    Then give the threshold that would change your mind, because that is what stops it sounding like a reflex. The broker earns its place when a request path contains work that cannot finish inside the response and whose failure must not fail the caller. The mesh earns its place when you have enough services that per-service retry, timeout and identity configuration has become a real maintenance cost, which is not at two services.

    Then offer the cheaper thing: a database-backed job table with a worker is a queue that this team can debug, and it can be replaced by a broker later without changing the calling code.

??? note "Show two wrong answers and what each reveals"

    **Wrong answer one: agreeing, because these are standard components.** The misconception is that popularity is evidence. Nothing in the prompt names a bottleneck, and adding two components with no measurement is exactly the pattern-matching interviewers screen for.

    **Wrong answer two: refusing without a threshold.** "That is over-engineering" with no number is the same missing evidence pointed the other way, and it reads as inflexible rather than as measured.

!!! note "An honesty note about D4 and D5"

    These two will feel harder than D1 to D3 and you will score lower on them. That is deliberate. D1 to D3 are the same shape as the case studies you just read, so they measure whether you can repeat a pattern. D4 and D5 hide which tool applies, which is what a real prompt does. A lower score on interleaved practice is the normal signal, not a sign the sprint failed.

---

## The night before

Six lines. Do these at hour 46, then stop.

1. Confirm the round: time zone, joining link, drawing tool, and whether it is 45 or 60 minutes. Open the drawing tool once and draw one box in it.
2. Print or open the [interview-day checklist](#14-interview-day-checklist) and the [three summary cards](#16e-one-page-summary-cards) in a second window.
3. Say the four-sentence opening out loud once, on a prompt you have not practised. Do not evaluate it.
4. Write your one deep dive on a card in three words, so you can see it while you talk.
5. Set out water, paper and a pen. Paper matters even in a remote round, for the arithmetic.
6. Sleep. If you have to choose between a seventh study hour and a seventh hour of sleep, the sleep wins and it is not close.

!!! warning "What not to do tonight"

    Do not open a new prompt, a new article or a new course. Material learned in the last few hours before a round cannot be retrieved under pressure, and the attempt displaces material that could have been. The single most common self-inflicted failure in a sprint is a panic hour at midnight.

---

## The morning of

Five lines, in this order.

1. Eat something. A design round is 45 minutes of continuous talking.
2. Read the three summary cards once, at reading speed. Do not re-derive anything.
3. Say the opening out loud twice, once on a prompt you know and once on one you do not.
4. Read the three recovery scripts. These are the highest-value forty seconds of the morning, because they are what you will actually need.
5. Ten minutes before: close every tab except the drawing tool and the checklist. Stand up, walk once around the room, sit down.

In the first minute of the round, ask two ordinary scheduling questions if nobody has said: is this 45 or 60 minutes, and would they like you to drive or to be interrupted. Both change how you spend the clock, and both are free to ask.

---

## After the interview

The sprint bought you a performance. It did not buy you depth, and the gap is exactly the omission table at the top of this page. Convert it while the round is still fresh.

**Within two hours of the round,** write down: every question you could not answer, every number you invented, and the one moment you felt the interviewer lose interest. That document is worth more than any course, because it is the only genuinely personalised study plan you will get.

| What went wrong in your round | Read this next | Start at |
|---|---|---|
| You could not size anything, or your numbers decided nothing | [Modern System Design](modern-system-design.md) | [Module 4, estimation as elimination](modern-system-design.md#module-4-estimation-as-elimination) |
| You drew a correct design and could not say what breaks | [Modern System Design](modern-system-design.md) | [Module 14, simplicity and scale thresholds](modern-system-design.md#module-14-simplicity-and-scale-thresholds) |
| A production-symptom question exposed you | [Modern System Design](modern-system-design.md) | [Module 11, the failure library](modern-system-design.md#module-11-the-failure-library) |
| The follow-ups were about labels, metrics or serving a model | [ML System Design](ml-system-design.md) | Its module map |
| The prompt was a product on a foundation model | [Generative AI System Design](generative-ai-system-design.md) | Its module map |
| The interviewer kept steering to the client, offline or battery | [Mobile System Design](mobile-system-design.md) | Its module map |
| The prompt was one screen or one component | [Frontend System Design](frontend-system-design.md) | Its module map |
| The questions were about resources, errors, idempotency and versioning | [Product Architecture](product-architecture.md) | Its module map |
| You had breadth and no depth anywhere | [Modern System Design](modern-system-design.md) | Any two case studies, done fully, including both deep dives |

### The delayed self-check

Answer these from memory a week after the round, with this page closed. If you can, the sprint became knowledge rather than a performance.

??? note "Show the three questions"

    1. Derive the crossover follower count above which fan-out on write cannot meet a stated freshness budget, from inputs you choose.
    2. A system stays degraded after a brief dependency blip is over. Name the class and the two design changes that remove it.
    3. Give the threshold, as a measurement, at which you would add a cache, and the cheaper thing you would do below it.

---

## 12. Mastery check and remediation map

Six short-answer items, one per learning objective. Write your answer before opening the block.

**M1** (Objective 1, routing yourself). You scored 7 yes answers on the triage. Which two blocks does your track expand, which two does it cut, and why each?

??? note "Show answer"

    Seven yes answers is Track C, strong but out of practice. It cuts hours 0 to 4 and 4 to 10 to skim reading, because the clock and the toolkit are already held and rereading them is comfort rather than progress. It expands hours 34 to 42 and the drill count in hours 18 to 34, because Track C's failure mode is a rusty performance rather than a missing schema, and only timed repetition fixes that.

    The general rule is worth stating: the triage routes by weakness, not by seniority, so two people with the same title and the same deadline can land in different tracks.

**M2** (Objective 2, the opening). Give the four sentences for a prompt you have not practised: "design a service that stores and serves user avatars."

??? note "Show answer"

    A correct answer has four moves and no phase names. For example:

    > "So: users upload an image, we store it, and we serve it back on every page that shows a profile, so reads massively outnumber writes. The hard parts look like the read volume, resizing into several variants, and what happens when a user replaces an image that is cached everywhere. I am dropping moderation and face detection unless you want them. Before I draw: do we serve one size or several, because that decides whether resizing is on the write path or the read path?"

    Check yourself on all four: restated with a directional number or ratio, named the hard parts, scoped something out, asked one question that forks the design. An answer that fails names a phase ("first I will do requirements") or asks a question whose answer changes nothing.

**M3** (Objective 3, numbers that eliminate). A prompt gives you 90 million events a day. Produce one number that eliminates something, then give the sentence you would use if nothing could be eliminated.

??? note "Show answer"

    90,000,000 divided by 86,400 is about 1,042 per second average. At a 3x peak, about 3,100 per second. That rules out a single application process, and it does not rule out a single primary store, because at roughly 3,000 small appends per second one node is still inside its range. Saying both halves is the answer; saying only the first is the number without the elimination.

    The decline sentence: "I can size this if it will drive a decision. Right now I think the consistency requirement decides the design rather than the request rate. Want me to run the numbers anyway?" Interviewers accept this more often than candidates expect, and the offer protects you if they did want the math.

**M4** (Objective 4, v0 and its breaking point). A prompt states 1,200 reads per second, 40 writes per second, 300 GB of data, and a requirement that a write is visible to the writer immediately. Draw the v0 in words, then name the breaking point in the required form.

??? note "Show answer"

    V0 is two boxes plus the client: a stateless service tier and one relational primary. No cache, because 1,200 reads per second is inside one primary's range and a cache would add a staleness window that directly conflicts with the read-your-writes requirement. No replicas, for the same reason: replica lag breaks the stated requirement unless the writer's own reads are pinned to the primary. No partitioning, because 300 GB fits one node with margin.

    The breaking point, in the required three parts: what fails is the primary's read capacity, at the point where sustained read throughput approaches its measured ceiling for this query mix; you observe it as connection pool wait time at p99 rising while write rate stays flat; and the next component is a read replica plus an explicit rule that the writer's reads go to the primary.

    An answer that fails: "it will not scale." No component, no number, no observation.

**M5** (Objective 5, holding a dive). Pick one of the three sprint case studies and give the three things your eight-minute dive must contain, with a concrete example of each.

??? note "Show answer"

    A mechanism, a number derived in front of the interviewer, and a named failure class.

    Taking the timeline: the mechanism is fan-out on write with a read-time merge above a threshold; the number is the crossover, derived as freshness budget times fan-out throughput, so 30 seconds times 69,600 inserts per second is 2.09 million followers; the failure class is head-of-line blocking, where one enormous fan-out job stalls ordinary posts behind it, removed by separating queues by cost class.

    A dive that fails contains only the mechanism. A dive with a mechanism and a number but no failure mode reads as a design that has never been operated.

**M6** (Objective 6, converting scores into a plan). You scored Below bar on "Depth in one area" across all five drills, and At bar elsewhere. State your next block, and the sibling course you open after the round.

??? note "Show answer"

    Next block: not more prompts. More prompts repeat the weakness. Take one prompt you have already done and rehearse its single dive aloud for eight minutes, twice, without notes, then check what you dropped the second time. That is the remediation row for depth, and it is a rehearsal task rather than a reading task.

    After the round: [Modern System Design](modern-system-design.md), specifically two full case studies including both deep dives each, because the depth gap is a practice gap and reading modules will not close it.

**Threshold.** Five of six, with M3 and M4 among them. Those two are weighted because eliminating numbers and stated breaking points are what separate a described architecture from a designed one, and they are also the two things a 48 hour sprint can genuinely install.

| Missed item | Reread | Redo |
|---|---|---|
| M1 | [The triage diagnostic](#the-triage-diagnostic) and the per-track table | Re-score the triage honestly and rewrite your own block allocation |
| M2 | [The delivery clock, compressed](#the-delivery-clock-compressed) | Record the opening on three unpractised prompts, six minutes each |
| M3 | [The six estimation drills](#1h-six-estimation-drills-five-minutes-each) and [the hub on estimation](index.md#capacity-estimation-that-earns-its-place) | Drills E1, E2 and E6, aloud, saying the eliminated option after every number |
| M4 | [The eight building blocks](#1b-the-eight-building-blocks) and every V0 block in the three case studies | Timed drill D1, stopping at minute 21 |
| M5 | The two deep dives in whichever case study you know best | Rehearse one dive aloud for eight minutes, twice, and diff the two attempts |
| M6 | [After the interview](#after-the-interview) and the rubric | Re-score two completed drills against the rubric and write the one-line plan each score implies |

!!! warning "Scoring well right now is a weak signal"

    Passing this an hour after reading mostly measures that the page is still in working memory. The signal that predicts performance is the delayed self-check in the previous section, taken a week later with the page closed.

---

## 13. Common mistakes catalogue

Technical mistakes cost a follow-up. Delivery mistakes cost the round. The last four rows are the delivery ones, and they are where a sprint candidate loses most often.

| The mistake | Why candidates make it | What the interviewer concludes | What to say instead |
|---|---|---|---|
| Cramming a new topic in the final six hours | Panic reads as productivity | Nothing, because it never surfaces. The cost is the material you did prepare and could not retrieve | Nothing. Close the tab and go to the taper block |
| Adding a cache with no read bottleneck shown | It is the most rehearsed move in every prep resource | Reaches for components by habit | "Reads are 1,200 a second, so a cache buys latency I was not asked for. I would add one when this query passes a large share of database CPU" |
| Partitioning in the first five minutes | Sharding sounds senior and the prompt said "scale" | Cannot tell a hundred writes per second from a hundred thousand | "One primary handles this write rate. Here is the number at which it stops and what I do then" |
| Claiming exactly-once delivery | The phrase exists, so it sounds like a setting | Has not debugged a delivery-semantics bug | "At-least-once transport plus an idempotency key retained for the longest retry window, which gives exactly-once effects" |
| Reciting a memorised architecture for a famous prompt | The sprint rehearsed three prompts, so pattern-matching is tempting | Cannot derive, only recall, and the follow-ups will show it immediately | "This looks like the timeline problem, so let me check whether the same numbers hold here before I reuse the shape" |
| Designing before clarifying (delivery) | Silence feels like failure and drawing feels like progress | Will build the wrong thing quickly and confidently | "Two questions first, because the answers change the shape: read heavy or write heavy, and does a stale read hurt anyone?" |
| Monologuing for ten minutes (delivery) | Fear that stopping invites a hard question | Cannot collaborate, and cannot be steered off a cliff | "That is the storage layer. Deeper here, or move to how it fails?" |
| Refusing to say "I do not know" (delivery) | Belief that a named gap is a scoring event | Bluffs, which is expensive on a real team | "I have not operated a consensus system. I know what it guarantees and what it costs, and here is how I would decide whether we need one" |
| Defending a choice instead of updating it (delivery) | Reading a follow-up as an attack rather than as data | Will not change a design in code review either | "You are right that this breaks under a hot key. That changes my partition choice, so let me redraw" |

---

## 14. Interview-day checklist

One screen. Nothing here is new.

**Minute budget, 45 minute round.** Open 0 to 2. Clarify and requirements 2 to 9. Numbers 9 to 11. V0 11 to 19. Breaking point and V1 19 to 27. One dive 27 to 36. Failure modes and wrap 36 to 40.

**The first four sentences.** Restate with a number. Name the hard parts. Say what you are dropping. Ask the one question that forks the design.

**Five clarifying questions that generalise across prompts**

1. Read heavy or write heavy, and roughly what ratio?
2. Does a stale read hurt anyone, and how stale is acceptable in seconds?
3. What is the largest single entity: the biggest tenant, the biggest producer, the hottest key?
4. Is this greenfield, or is there a running system I should not break?
5. What would make this a failure in production, as opposed to wrong on the whiteboard?

**Drawing order.** Client, then the one service that does the work, then the store. Stop. Leave the right third of the canvas empty, because that is where V1 goes. Do not draw a load balancer, a cache or a queue until a number on the board demands it.

**Three recovery scripts.** Stuck: state the assumption and move. Disagreed with: update the design out loud. Out of time at minute 34: close the dive and protect the wrap.

**Before you finish, three sentences.** Name one trade-off you made and its cost. Name what you skipped and why. Name the one thing you would measure first in production.

---

## 15. If you have less than 48 hours

The sprint has branches. Each one names what it drops and what that costs.

| You have | Do these blocks | What you drop | What that costs |
|---|---|---|---|
| 24 hours | Hours 0 to 10 in full, one sleep, then a compressed 18 to 34 with two case studies instead of three, then the taper | The third case study, one deep dive, drills D3 to D5 | You arrive with one dive instead of two, and no interleaved practice, so an unfamiliar prompt will feel unfamiliar |
| 12 hours | The clock, the toolkit, the six estimation drills, one case study, drills D1 and D5 | Both sleeps as scheduled blocks, two case studies, all deep dives | You have structure and vocabulary but no depth anywhere. Plan to say "I would go deeper here if we had time" and mean it |
| 6 hours | The clock, the four-sentence opening rehearsed three times, the eight building blocks, one case study read not attempted | Everything else | You are buying delivery only. That is still the single highest-yield six hours available, because most rounds are lost on structure rather than on facts |
| 2 hours | The four-sentence opening, the five clarifying questions, the drawing order, the three recovery scripts | Everything else | You will not derive anything, but you will not freeze, and you will not draw seven boxes before asking a question |

The pattern across all four branches is the same, and it is the argument of this whole page: delivery is bought first because it is cheapest to install and most expensive to lack. Depth is bought last because it takes the longest and is worth the most.

---

## 16. Reference block

Lookup material only. Nothing here is explained for the first time.

### 16a. The eight decisions this sprint carries

The full set of twenty-two is in [the hub's trade-off atlas](index.md#the-trade-off-atlas). These eight cover most sprint prompts.

| Decision | Choose A when | Choose B when | The number that decides it |
|---|---|---|---|
| A: cache. B: read replica | A small hot subset serves most reads and seconds of staleness are fine | Reads spread across the whole dataset, or the reader needs read-your-writes | Achievable hit rate on the hot subset, assumed to stop paying below about 80 percent |
| A: normalise. B: denormalise | Writes are frequent and a shared entity must stay correct | Reads dominate and the read shape is stable | Reads per write for that entity, assumed to favour denormalising above roughly 100 to 1 |
| A: synchronous. B: asynchronous | The caller cannot proceed without the result | The caller needs only an acknowledgement | The endpoint's tail latency budget against the p99 of the inlined work |
| A: fan-out on write. B: on read | Producers have small audiences | A few producers have very large audiences | Freshness budget times fan-out throughput, which gives your own crossover |
| A: single region. B: multi region | Users are concentrated and an hours-long regional outage is survivable | A regulator requires residency, or recovery time is minutes | Recovery time objective in minutes, and cross-region round trip against the write budget |
| A: hash partitioning. B: range | Access is by exact key and you want even spread | Queries are ordered scans over time or sequence | Fraction of queries that are range scans |
| A: at-least-once plus idempotency. B: transactional | The effect can be deduplicated by a key you control | The effect is external and cannot be undone | Cost of one duplicate against the throughput cost of transactional commit |
| A: vertical scaling. B: horizontal | Well below the biggest instance available, stateful workload, small team | Within a small factor of the biggest instance, or failure isolation is needed | Peak utilisation as a fraction of the largest single node you can buy |

### 16b. The six numbers, with derivations

| Number | Value | Derivation |
|---|---|---|
| Seconds in a day | 86,400 | 24 times 60 times 60 |
| Daily count to per second | divide by 86,400 | 1 million a day is about 11.6 per second; 1 billion a day is about 11,600 |
| Peak multiplier | assume 2x to 5x average | Traffic concentrates in waking hours for a single-region consumer product, so peak hour carries several times the mean |
| Small row size | 150 to 500 bytes | A short text record is about 300 bytes: 280 characters mostly one byte each, plus three 8 byte fields |
| Latency ladder | memory 100 ns, SSD 100 us, disk 9 ms, same datacentre under 1 ms, intercontinental about 80 ms | Memory to SSD is about 1,000x; SSD to spinning disk about 90x; disk to intercontinental about 9x. The 80 ms is 8,000 km of fibre at roughly 200,000 km per second, doubled for real routes |
| Monthly downtime budget | 99.9 percent is 43 min, 99.95 is 21.6 min, 99.99 is 4.3 min | 30 times 24 times 60 is 43,200 minutes; take 0.1, 0.05 and 0.01 percent of it |

### 16c. ASCII glyph legend

| Glyph | Meaning |
|---|---|
| `+---+` | A component or box |
| `--->` | A request or data flow, in the direction of the arrow |
| `<---` | A response |
| `==>` | A persistent connection rather than a request |
| Text inside a box | The component name and its one distinguishing property |
| The line under a diagram | The caption, which carries the same information as the drawing |

### 16d. Review cards

Copy this block. One fact or one decision per card.

```
1. A number may appear only if the next sentence kills an option.
2. Seconds in a day: 86,400. Peak is 2x to 5x average.
3. Cache earns its place above roughly an 80 percent hit rate.
4. Replicas add read capacity and lag, never write capacity.
5. Fan-out crossover = freshness budget x fan-out throughput.
6. Hot key ceiling = instance count / local time to live.
7. Partition key is the hardest decision to reverse. Make it last.
8. Queue = acknowledgement now. It hides failure, not latency.
9. Outbox = event written in the same transaction as the state.
10. Order by sequence, never by wall-clock time.
11. Alert on oldest-message age, never on queue depth.
12. Metastable = the degraded state generates its own load.
13. Breaking point = what fails, at what number, how observed.
14. 99.95 percent monthly = 21.6 minutes of downtime.
15. Say what you skipped, before the interviewer asks.
```

### 16e. One-page summary cards

**Card 1, URL shortener**

- Inputs: 2 million links a day, 200 to 1 reads, 3 year retention, 200 byte rows.
- 92 writes per second at peak rules out write sharding. 18,400 reads per second at peak rules out serving from the primary. 438 GB rules out partitioning for size. A 21.9 GB working set sizes the cache at two nodes.
- V0: client, app tier, one store. V1 adds a 22 GB cache, a one second local tier, replicas and a click queue.
- Dive: hot-key ceiling equals instance count over local time to live. Second dive: 302 versus 301 sets the entire read bill.
- Failures: cache stampede, hot key, thundering herd, retry storm.

**Card 2, home timeline**

- Inputs: 50 million daily users, 8 reads and 0.2 posts each, 200 median followers, 30 second freshness.
- 13,900 peak reads times 200 followed accounts is 2.78 million probes per second, which kills pull at read. 348 peak posts times 200 followers is 69,600 inserts per second. Crossover is 30 times 69,600, so 2.09 million followers.
- V0: pull at read. V1: precomputed capped id lists. V2: read-time merge above the crossover.
- Dive: derive the crossover live. Second dive: cursor on (created at, post id).
- Failures: unbounded queue growth, hot key, head-of-line blocking, thundering herd.

**Card 3, chat**

- Inputs: 10 million daily users, 40 messages each, 30 percent connected at peak, 400 byte records.
- 3 million connections at 20 KB is 60 GB, so 60 gateway nodes at an assumed 50,000 each. 1.5 million polls per second for 13,900 useful messages kills long polling. 58 TB a year kills a single relational primary.
- V0: polling. V1: gateway tier, session map, per-conversation sequence, partitioned store.
- Dive: session map versus broadcast, decided by node count. Second dive: sequence numbers, because clock skew reorders devices, and receipts collapse into cursors.
- Failures: thundering herd on reconnect, clock skew, head-of-line blocking, gray failure.

---

## 17. What this course does not cover

Beyond the omission table at the top, which lists what a 48 hour budget cuts from a generalist round, these are outside the page entirely.

| Not covered | Why | Where to go |
|---|---|---|
| Any round that is not generalist distributed systems | The toolkit misfires on models, phones, browsers and API contracts | The five sibling courses linked in the header |
| Behavioural and hiring-manager rounds | A different skill with a different apparatus | Not on this site |
| Coding and low-level object design rounds | Unrelated to this page's method | [DSA question bank](/Interview-Questions/data-structures-algorithms/) |
| Salary, levelling and offer negotiation | Out of scope, and we have nothing auditable to add | Not on this site |
| Company-specific interview formats | We do not publish claims about a named company's internal process without that company's own published writing | Ask your recruiter directly. It is an ordinary question |

Free material we do not control, with editions and dates, for after the round:

- *Designing Data-Intensive Applications*, Martin Kleppmann, O'Reilly, 2017. The single best book for the material this sprint omits, particularly replication, partitioning and consistency.
- *Site Reliability Engineering*, Google, O'Reilly 2016, [free online at sre.google/sre-book](https://sre.google/sre-book/table-of-contents/). Chapters 4 to 6 are the source for service levels and alerting.
- *The Site Reliability Workbook*, Google, O'Reilly 2018, [free online at sre.google/workbook](https://sre.google/workbook/table-of-contents/). The practical companion, including error budget policy.
- RFC 9110, HTTP Semantics, June 2022, for redirect and caching semantics.
- *In Search of an Understandable Consensus Algorithm*, Ongaro and Ousterhout, USENIX ATC 2014, if you want the consensus material this sprint drops.
- The `donnemartin/system-design-primer` repository on GitHub is a widely used free community resource. It is a living repository with no fixed edition, so check its recent commit history before trusting any specific figure in it.

---

## 18. Provenance

**Last reviewed:** 1 August 2026. Any page in this section whose last-reviewed date passes nine months is flagged for review rather than quietly left in place.

**Changelog, most recent first**

| Date | Change |
|---|---|
| August 2026 | First publication: the omission table, the eight-question triage with three tracks, the seven-block hour-by-hour plan, the eight-block toolkit, three sprint case studies, five timed drills and the post-round routing table |
| August 2026 | Every number on this page re-derived from inputs stated in the same section; the fan-out crossover and the hot-key ceiling changed from stated constants into formulas the reader computes from their own budget |
| August 2026 | Working hours reconciled against wall-clock hours so the seven blocks sum to 48 and the three tracks land inside the published 25.0 to 28.5 hour band |

**Found a problem?** Open an issue or a pull request at [github.com/datascienceinterviews/datascienceinterviews.github.io/issues](https://github.com/datascienceinterviews/datascienceinterviews.github.io/issues). Disputed numbers, dead links, better derivations and missing counter-arguments are all in scope.

All content on this page is original. Research informed which topics belong here; no prose, framework, acronym, example, table or diagram is derived from any paid course. Primary sources are cited inline with their edition and year, and any claim we could not source is either labelled as an assumption or is not on the page.
