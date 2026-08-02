---
title: Modern System Design Interview Course
description: A free course on deriving a distributed system from its stated constraints, naming the number at which it breaks, and defending the simplest design.
last_reviewed: 2026-08-01
---

# Modern System Design Interview

!!! abstract "The contract"

    **After this course you can** take a one-line prompt, derive a design from its stated constraints, name the number at which that design breaks, and defend the smallest version that still meets the requirement against a hostile follow-up.

    **Who it is for**

    - A backend or infrastructure engineer with roughly 2 to 8 years of shipping services, facing a generalist design round.
    - An engineer who can build the system but stalls when asked to choose out loud between two designs.
    - A senior candidate whose feedback keeps saying "good breadth, no depth" and who cannot tell which minute they lost.

    **Who it is not for**

    | If your round is about | Go here instead |
    |---|---|
    | Ranking, retrieval, training pipelines, feature stores | [ML System Design](../ml-system-design/) |
    | LLM products, RAG, agents, inference serving | [Generative AI System Design](../generative-ai-system-design/) |
    | An app client, offline sync, battery, push | [Mobile System Design](../mobile-system-design/) |
    | A browser application or a single component | [Frontend System Design](../frontend-system-design/) |
    | An API contract, error model, versioning, webhooks | [Product Architecture](../product-architecture/) |
    | An interview inside the next week | [Fast-Track in 48 Hours](system-design-fast-track.md) |

    **Trains for:** the 45 and 60 minute generalist distributed-systems design round, and the architecture portion of senior and staff loops.

    **Does not train for:** coding rounds, low-level object design rounds, written architecture take-homes, or behavioural rounds.

    **Fast path:** already comfortable? Go straight to the [self-assessment rubrics](#11-self-assessment-rubrics). Just want the tables? Go to the [reference block](#16-reference-block).

---

## Prerequisites

Seniority is a poor readiness signal. Prerequisite mastery is a good one. Answer every Quick check in under a minute. Two or more misses means start with the linked material, not with Module 1.

| You should already be able to | Where to learn it | Quick check |
|---|---|---|
| Trace one HTTP request from client to database and back, naming each hop | [System design question bank](/Interview-Questions/System-design/) | Name the three hops between a browser and a row in a database, in order |
| Say what an index costs on the write path | [PostgreSQL indexes documentation](https://www.postgresql.org/docs/current/indexes.html) | A table has three secondary indexes. One row insert touches how many index structures, at minimum? |
| Divide traffic per day into traffic per second in your head | This page, Module 4 | 8.6 million requests a day is roughly how many per second? |
| Read a percentile, not an average | [Google SRE Book, chapter 6, free online](https://sre.google/sre-book/monitoring-distributed-systems/) | Your p99 is 400 ms and you fan out to five services in parallel. Is the combined p99 better or worse than 400 ms, and why? |
| Compute what a cache hit ratio does to origin load | [MDN HTTP caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching) | Front door takes 10,000 requests per second at a 90 percent hit ratio. What does the origin see? |
| Say what a queue's consumer lag means | [Apache Kafka design documentation](https://kafka.apache.org/documentation/#design) | Lag is climbing in a straight line. What is the one comparison that tells you whether adding consumers fixes it? |
| Write SQL that joins two tables and groups | [SQL question bank](/Interview-Questions/SQL-Interview-Questions/) | Which side of a left join keeps its rows when the other side has no match? |
| Draw a five-box diagram on a whiteboard without redrawing it twice | Any whiteboard, ten minutes of practice | Draw login: browser, service, database, and the two arrows. Did you leave room on the right? |

??? note "Show answers to the quick checks"

    1. Browser to load balancer or gateway, gateway to application service, service to database. Anything more is an optimisation you have not justified yet.
    2. Four. The table's own storage plus three secondary indexes. This is the whole argument against adding indexes casually.
    3. 8,600,000 divided by 86,400 seconds is close to 100 per second. Memorise 86,400 and stop multiplying.
    4. Worse. If each call independently has a 1 percent chance of exceeding 400 ms, the chance that none of five does is 0.99 to the fifth power, about 0.951, so about 4.9 percent of requests exceed it. Fan-out converts a p99 into roughly a p95.
    5. 10 percent of 10,000, so 1,000 per second. The cache is doing a factor of ten, and losing it means the origin must survive ten times its normal load.
    6. Arrival rate against total consumer service rate. If arrival exceeds service, add consumers. If they are equal and lag is still growing, you have a partition skew or a poison message, and more consumers change nothing.
    7. The left side. Unmatched right-side columns come back null.
    8. There is no answer key. If you redrew it, that is the practice, and it is the cheapest hour in this course.

---

## Time budget

| Part of the course | Reading | Practice | Spaced review |
|---|---|---|---|
| Modules 1 to 5, framing and properties | 2.9 to 3.7 h | 1.5 to 2.0 h | included below |
| Modules 6 to 9, building blocks | 4.3 to 5.5 h | 2.0 to 3.0 h | included below |
| Modules 10 to 16, hard parts and judgement | 4.8 to 5.8 h | 1.5 to 2.5 h | included below |
| Eight case studies and both practice sets | included above | 5.0 to 8.5 h | included below |
| Five review sessions on days 1, 3, 7, 21 and 60 | 0 | 0 | 4.0 to 5.0 h |
| **Total** | **12 to 15 h** | **10 to 16 h** | **4 to 5 h** |

!!! note "How these were computed, so you can argue with them"

    Reading is summed per module from the Module Map table below, not guessed for the page as a whole. That table's minutes total 720 at the low end and 900 at the high end, which is exactly the 12 to 15 hour range. Per-module minutes assume 120 to 150 words a minute, a deliberately slow engaged rate, because dense technical prose with diagrams is not read at prose speed.

    Practice assumes 20 to 30 minutes attempting each of the 8 case studies before reading the arc, which is 160 to 240 minutes, plus the 11 practice problems at the minutes each set states: 5 to 15 for each of the five blocked problems and 20 to 30 for each of the six interleaved, which is 145 to 255 minutes.

    Those two lines sum to 305 to 495 minutes, the 5.0 to 8.5 hours in the case-study row. Timed self-mock work is the day 7, 21 and 60 review sessions, counted in the review column rather than twice.

    Review assumes five sessions of 45 to 60 minutes. Every per-item figure above is padded by roughly 15 percent, because self-estimates of study time run optimistic.

    A single number would be more marketable and less true. At an hour a day this is two to three weeks. Full time it is a hard week.

---

## Learning objectives

Each objective is tested by exactly one numbered item in the mastery check, and the numbering matches: objective 1 is tested by mastery item 1.

1. **Produce** five clarifying questions from a one-line prompt in under four minutes, where every question has two plausible answers that lead to visibly different architectures, and no question survives that fails that fork test.
2. **Convert** a stated traffic figure and retention window into storage, bandwidth and machine-count estimates, showing each arithmetic step, and **state for every number the design option it eliminates**, deleting any number that eliminates nothing.
3. **Draw** a v0 design that satisfies the stated requirements using the fewest components the requirements permit, then **name** the specific metric and the specific threshold at which it fails, plus the one component you add next.
4. **Choose** between two replication or partitioning options for a given read-to-write ratio and data size, **citing the number that decides it** and the condition under which the rejected option would win.
5. **Diagnose** a described production symptom into exactly one named failure class from the failure library, and name the design change that removes it, in under two minutes and without proposing a retry.
6. **Defend** a single-node or single-region design against an interviewer pushing for distribution, citing the throughput or data-size threshold below which distribution costs more than it buys.
7. **Rewrite** a mid-level answer into a staff-level answer by adding blast radius, migration order, cost per unit of traffic and one operability commitment, annotating what each addition changed about the decision.

---

## Warm-up retrieval

Three to five minutes. Answer out loud or in writing before opening anything. Retrieval beats rereading, and a visible answer converts one into the other.

**Q1.** From the hub's archetype router: name two signals inside the first minute that tell you this is a generalist distributed-systems round and not a product architecture round.

??? note "Show answer"

    First, the prompt is stated in users, requests or data volume rather than in callers and resources. Second, the first follow-up targets capacity or failure rather than error codes, pagination or versioning. A third weaker signal: the interviewer hands you a drawing surface and expects boxes, rather than expecting a request and response body.

**Q2.** From the trade-off atlas on the hub, several pages back: state the decision rule for putting work behind a queue instead of doing it synchronously, and name the number that decides it.

??? note "Show answer"

    Do it synchronously when the caller needs the result to render its next screen and the work fits inside the latency budget you have left. Move it to a queue when the work would consume more than roughly a fifth of the remaining budget, or when its failure should not fail the caller's request. The deciding number is the end-to-end p99 budget minus the time already spent upstream. If that leftover is 40 ms and the work takes 300 ms, the decision is made for you.

**Q3.** The hub states plainly what a written course cannot do. What are the two things, and what does this section substitute for them?

??? note "Show answer"

    It cannot give you a live interviewer who adapts follow-ups to your specific weak answer, and it cannot give you calibrated human judgement of your delivery. The substitutes are public four-criterion rubrics on every practice problem, interviewer script packs you can hand to a friend, timed drills, and level deltas that show what a staff answer added. Substitutes, not replacements. Do at least one mock with a human.

---

## Module map

```
+--------------------------------+
| FRAME              M1 to M4    |
| scope, clock, requirements,    |
| estimation that eliminates     |
+---------------+----------------+
                |
                v
+--------------------------------+
| PROPERTIES         M5          |
| availability math, durability, |
| consistency spectrum           |
+---------------+----------------+
                |
                v
+--------------------------------+    +---------------------+
| BUILDING BLOCKS    M6 to M9    |    | HARD PARTS          |
| request path, storage,         |==> | M10 multi-region    |
| movement, coordination         |    | M11 failure library |
+---------------+----------------+    +----------+----------+
                |                                |
                v                                v
+-------------------------------------------------------+
| JUDGEMENT                              M12 to M16     |
| operability, cost, simplicity, brownfield, levels      |
+-------------------------------------------------------+

Caption: the five clusters of this course and their reading order.
Frame feeds Properties, which feeds Building Blocks. Building Blocks
feeds Hard Parts. Both feed Judgement, which is where mid-level and
staff-level answers separate. Arrows point from prerequisite to
dependent; nothing points backwards.
```

| Module | What you will be able to do | Time | Skip if |
|---|---|---|---|
| M1 Scope and honesty | Name which round you are in within the first minute, and steer back when the title and the questions disagree | 15 to 20 min | You have already been told your exact round format in writing |
| M2 The delivery clock | Spend 45 or 60 minutes on a budget instead of on whatever the interviewer last asked | 25 to 30 min | Never skip. This is the module that changes outcomes fastest |
| M3 Requirements that change the design | Ask five questions whose answers fork the architecture, and drop the ones that do not | 40 to 50 min | You already scope by branch table and can show one you wrote |
| M4 Estimation as elimination | Size a system in four minutes, and offer or decline estimation without losing credibility | 40 to 50 min | You can already state what a number ruled out, every time |
| M5 Non-functional characteristics as mechanisms | Turn "highly available" into a number, a mechanism and a cost | 55 to 70 min | You can derive the availability of two components in series and in parallel from memory |
| M6 Building blocks I, the request path | Choose between DNS, anycast, layer 4 and layer 7 balancing, and compare four rate limiters on memory and burst | 65 to 80 min | You operate load balancers and gateways as your day job |
| M7 Building blocks II, storage | Pick a replication topology and a partition key, and state what each costs on read and on write | 70 to 90 min | You have shipped a resharding and can describe its rollback plan |
| M8 Building blocks III, movement | Choose queue versus log versus pub-sub, and say precisely where exactly-once is a lie | 65 to 85 min | You have run a stream processor in production with watermarks |
| M9 Building blocks IV, coordination | Explain Raft at interview depth, use leases and idempotency keys, and say why you rarely implement consensus | 60 to 75 min | You can explain why a leader lease needs a clock bound |
| M10 Multi-region and geo-replication | Pick active-active or active-passive from a latency and residency budget, and name the conflict rule | 50 to 60 min | Your target companies are single-region and you have confirmed it |
| M11 The failure library | Name a production symptom's failure class and the design change that removes it | 50 to 60 min | Never skip. This is the highest-yield module for senior candidates |
| M12 Observability and operability | Ship SLIs, alerts and trace propagation as part of the design, and keep metric cardinality bounded | 40 to 50 min | You have written an SLO document that survived a review |
| M13 Cost as a constraint | Attach a cost per million requests to an architectural choice and argue it without sounding cheap | 30 to 40 min | You already do capacity and cost planning for a service |
| M14 Simplicity | Defend a single-node answer with a threshold, and delete a box you cannot justify | 45 to 55 min | Never skip. Interviewers report this as the rarest strong signal |
| M15 Brownfield design | Plan dual writes, shadow traffic, backfill order, deprecation and blast radius for a running system | 45 to 55 min | You are interviewing at mid level only, and the loop is greenfield |
| M16 Level calibration | Say what your answer is missing to read one level higher | 25 to 30 min | Never skip if you are targeting senior or above |

Low ends total 720 minutes, high ends total 900. That is the 12 to 15 reading hours in the time budget above.

Method note for the Time column: it is reading only, computed per module at 120 to 150 words a minute. Each of Modules 1 to 8 repeats its reading range inline under its own heading, unchanged, and adds a practice range. The inline practice ranges for Modules 1 to 5 sum to 90 to 120 minutes, which is the 1.5 to 2.0 hours in the practice column above.

---

## The delivery clock for this round

Most candidates who fail a design round did not lack a fact. They ran out of minutes with no design on the board, or they filled 45 minutes with breadth and never went deep enough for the interviewer to grade anything. The clock is a budget, not a script.

### The 45 minute round

| Minutes | Spend it on | You leave this phase with |
|---|---|---|
| 0 to 2 | Restate the prompt, name the two things you will optimise for | The interviewer agreeing, or correcting you cheaply |
| 2 to 8 | Clarifying questions, then requirements written down | Functional list, plus two or three numbers tagged GIVEN or ASSUMED |
| 8 to 12 | Estimation, only where a number will eliminate an option | One or two decisions already made, not a page of arithmetic |
| 12 to 20 | v0 drawn: the fewest boxes the requirements permit | A correct design at the stated scale, on the board |
| 20 to 30 | The breaking point, then v1 | A named metric, a threshold, and the component you added |
| 30 to 40 | One deep dive, ideally the one the interviewer picks | Depth in a single area, with a trade-off stated both ways |
| 40 to 45 | Failure modes, what you would measure first, what you skipped | An explicit list of what you did not cover and why |

The phases sum to 45 by construction: 2 plus 6 plus 4 plus 8 plus 10 plus 10 plus 5.

### The 60 minute round

| Minutes | Spend it on |
|---|---|
| 0 to 2 | Open and restate |
| 2 to 10 | Clarify and write requirements |
| 10 to 15 | Estimation that eliminates |
| 15 to 24 | v0 |
| 24 to 36 | Breaking point and v1 |
| 36 to 52 | Two deep dives, different in kind from each other |
| 52 to 58 | Failure modes, SLIs, operability |
| 58 to 60 | Close: trade-offs, what you skipped, what you would measure first |

The extra 15 minutes buys one more deep dive and a real operability section. It does not buy more boxes. These sum to 60: 2 plus 8 plus 5 plus 9 plus 12 plus 16 plus 6 plus 2.

### The first two minutes

Say four sentences, in this order. Rehearse them once so they are automatic and you can spend attention on listening.

1. "Let me restate it: you want X for roughly Y users, and the part that matters most is Z." (Restate, and put a number in it even if you invented the number.)
2. "I am going to spend about five minutes on requirements, then draw the simplest thing that works, then break it on purpose." (You just told them your clock, which reads as control.)
3. "Two things I will optimise for: A and B. I will trade away C to get them." (Naming what you sacrifice is the single cheapest senior signal available.)
4. "Stop me if you want depth somewhere specific rather than coverage." (This is an invitation, and interviewers take it more often than candidates expect.)

### Declaring what you are skipping

Skipping silently reads as not knowing. Skipping out loud reads as prioritising. The pattern is: name it, say why it is out of scope, say what you would do if it were in scope, in one sentence.

- "I am not designing auth here. It is a solved shape and it does not interact with the write hot spot, which is the interesting part. If you want it, it is a token check at the gateway."
- "I am skipping the analytics path. It is asynchronous and it never blocks a user request, so it cannot change any decision we make in the next twenty minutes."
- "I will not size the CDN. Nothing in this design changes based on whether it is 2 or 20 terabytes a day."

### Checking in without sounding unsure

The difference is that a weak check-in asks for approval and a strong one offers a fork.

| Weak, reads as unsure | Strong, reads as driving |
|---|---|
| "Does that make sense?" | "That is the write path. Do you want the read path next, or the failure behaviour?" |
| "Is this what you wanted?" | "I have assumed reads dominate writes ten to one. Correct me now if that is wrong, because it decides the storage choice." |
| "Should I go deeper?" | "I can go one level into partitioning, or stay wide and cover the queue. Which is more useful to you?" |
| "Sorry, I am rambling." | "I am four minutes over on requirements. Moving to the diagram." |

### How the split shifts by level

| Phase | Mid | Senior | Staff |
|---|---|---|---|
| Requirements and clarifying | Same | Same | Longer: challenges whether the stated problem is the real one |
| Estimation | Full, shown step by step | Trimmed to the numbers that decide something | Often one number, then a cost or headcount consequence |
| v0 and v1 | Most of the time here | Balanced | Compressed, because it is assumed correct |
| Deep dives | One, chosen by interviewer | Two, one chosen by the candidate and defended | Two, plus the migration path to get there from today |
| Failure and operability | Mentioned at the end | A real section with alerts and SLIs | Woven throughout, plus blast radius and rollback |

The rough shape: mid level spends its minutes proving the design is correct, senior spends them proving the trade-off was chosen, staff spends them proving the design is reachable, operable and affordable.

!!! tip "One note on frameworks, made once"

    Educative's paid courses use acronym spines: RESHADED in the modern course, SCADET in mobile, REDCAAP in generative AI. Those are theirs, not ours. The problem is not the letters, it is that applying the same acronym identically to nine prompts produces an answer that sounds recited, and interviewers now screen for exactly that tell. The clock above is a time budget, not a spine. It never appears as a heading in any case study on this page, and Module 2 teaches when to abandon it.

!!! danger "When to break this clock"

    The measurement that justifies following the clock: you have run a timed self-mock and finished with a design on the board and time left. Below that, the clock is scaffolding you still need. Above it, treat it as advisory. Three situations where following it is the wrong move:

    **1. The interviewer opens with a depth question.** If minute three is "how would you store the mapping?", they have told you they want one deep area, not coverage. Drop estimation entirely, sketch a three-box v0 in ninety seconds, and spend the round where they pointed. Coverage you were not asked for is the most expensive thing you can buy with those minutes.

    **2. Estimation would be noise.** Internal tooling for 200 employees, or a prompt with no stated scale and no scale-sensitive component, does not need a capacity section. Offer once: "I can size this if it will drive a decision, otherwise I will assume it fits on one machine and move on." If they say move on, move on and never return to it.

    **3. The system already exists.** In a brownfield prompt the requirements are fixed and the interesting minutes are elsewhere. Replace the requirements and v0 phases with: what does the current system do, what is failing and at what number, what is the smallest change, and in what order do you ship it without a big-bang cutover. Drawing a greenfield v0 for a system that is already running is the most common way to fail a staff round.

    A fourth, quieter one: if the interviewer has gone silent and is typing, they are usually grading drive rather than content. Keep narrating decisions out loud and do not wait for permission to continue.

## Module 1. Scope and honesty: which round is this

**Time: 15 to 20 minutes reading, 10 to 15 minutes practice.**

### 1a. Predict first

Read these five opening lines. Each is something an interviewer has plausibly said in the first minute of a round.

| # | The opening line |
|---|---|
| A | "Design a service that shortens long URLs. Take it as deep as you like." |
| B | "Here is our ingestion service. It falls over every Monday at 09:00. Walk me through what you would change." |
| C | "Design the public interface a partner integration would call to submit a refund." |
| D | "Design the classes for a parking garage, then write the interfaces." |
| E | "Design the system that decides which posts appear at the top of the feed." |

??? note "Show answer"

    Prediction asked for: which of these does this course train you to answer, and which belong to a different round with a different scoring rubric?

    | # | Round | Does this course train it |
    |---|---|---|
    | A | Distributed system design, greenfield | Yes, this is the centre of the course |
    | B | Distributed system design, brownfield | Yes, and it is the round most courses skip |
    | C | Interface and domain design | Partly: the request path and storage modules transfer, the contract modules do not |
    | D | Object and class design, often with code | No. Different rubric, different preparation |
    | E | Machine learning system design | Partly: serving, storage and movement transfer, the modelling and metric work does not |

    The tell for D is the word "classes" and the request for code. The tell for E is that the hard part is a decision quality problem, not a capacity problem. The tell for C is that the deliverable is a contract, not a topology.

### The archetypes, and what each one actually scores

A "system design interview" is not one round. At least six distinct rounds use the phrase, and they score different things. Preparing for the wrong one is the most expensive mistake available to you, because it costs weeks, not minutes.

| Archetype | The deliverable the interviewer wants | What gets you a strong rating | Where this course helps |
|---|---|---|---|
| Distributed backend, greenfield | A topology plus the reasoning that produced it | Deriving the design from stated constraints, and naming what breaks next | Fully |
| Distributed backend, brownfield | A change plan for a running system | Blast radius, migration order, rollback | Fully, in the brownfield module |
| Interface and domain design | A resource model, error semantics, versioning plan | Idempotency, pagination stability, compatibility | Partly |
| Machine learning system design | A training and serving loop with metrics | Label delay, evaluation, feedback loops | Partly |
| Object and class design | Classes, interfaces, sometimes compiling code | Cohesion, extension points, invariants | No |
| Infrastructure deep dive | One component's internals | Depth, mechanism, measured trade-offs | Fully, in the building block modules |

**How to tell which round you are in, inside 90 seconds.** Listen for the noun in the prompt.

- A product name or user-facing verb ("shorten", "feed", "chat") means backend design.
- The words "interface", "endpoint", "partner", "SDK" mean contract design.
- The words "predict", "rank", "detect", "recommend" mean machine learning design.
- The words "class", "object", "implement" mean object design.
- A named running system plus a symptom means brownfield.

**The rounds disagree with their titles more often than anyone admits.** A recruiter's calendar invite says "System Design" and the interviewer runs a deep dive on their own team's queue. Your response is not to complain, it is to re-aim in one sentence.

!!! tip "The re-aim sentence"

    "Before I start drawing, can I check what you want most from the next forty minutes: breadth across the whole system, or depth on one part of it? I will spend the time differently."

    Asked once, in the first two minutes, this reads as professionalism. Asked at minute twenty, it reads as being lost.

**On the name of this course category.** Several paid products ship under nearly identical "Grokking" titles from different vendors, and at least one sits on a URL history that makes the two easy to confuse. That is a purchasing problem, not a learning problem. Check the vendor name on the checkout page. We mention it once and move on.

### The routing diagram

```
        prompt heard in the first minute
                     |
       +-------------+-------------+
       |             |             |
  product noun   contract noun  model noun
       |             |             |
       v             v             v
 +-----------+  +-----------+  +-----------+
 | backend   |  | interface |  | ML system |
 | design    |  | design    |  | design    |
 +-----------+  +-----------+  +-----------+
       |
   is a running system named
       |
   +---+----+
   |        |
   v        v
+--------+ +----------+
| brown  | | green    |
| field  | | field    |
+--------+ +----------+

Figure routing the first minute of a round to one of five archetypes
```

### Worked example: a recruiter email arrives

The email says: "You have a 60 minute System Design round with a senior engineer from the Payments Platform team. Please be ready to discuss scalability."

**Block 1. PURPOSE: extract the scoring surface from the words I was actually given.**

I have three signals: 60 minutes, Payments Platform, and the word scalability. The team name is the strongest of the three. A payments platform team spends its days on idempotency, ledger correctness and third-party failure, so a payments interviewer usually probes correctness under retry before they probe throughput. The word "scalability" is recruiter boilerplate and I weight it near zero.

!!! note "Say it before you read on"

    Why did I weight the team name above the word "scalability"? Say your reason out loud before continuing.

**The error most readers make here** is treating recruiter adjectives as requirements. "Scalability" appears in almost every invite. It carries no information, so it cannot change my preparation.

**Block 2. PURPOSE: decide what I will deliberately not prepare.**

Given a payments team and 60 minutes, I drop object design drills and machine learning material entirely. I also drop deep video and media pipeline material. That frees roughly the whole of my preparation time for three things: exactly-once effects, ledger and reconciliation, and gateway failure handling.

**The error most readers make here** is additive preparation: adding payments material on top of everything else and running out of hours. Preparation is subtraction.

**Block 3. PURPOSE: buy information cheaply before the round.**

I send the recruiter one message: "Is the round greenfield design, or a walkthrough of an existing system? And is the interviewer looking for breadth or one deep dive?" This is a normal question, and recruiters usually answer it.

!!! note "Say it before you read on"

    Name the one piece of information in that reply that would change the most about how I spend the 52 working minutes.

**Block 4. PURPOSE: write the opening thirty seconds that steers a mislabelled round.**

I write and rehearse two sentences. "I would like to spend the first five minutes on requirements and the numbers that matter, then draw the smallest thing that works, then break it deliberately. If you would rather go straight to depth on one area, tell me now and I will restructure."

**The error most readers make here** is improvising the opening. The opening is the only part of the round that is fully predictable, so it is the only part worth memorising.

**Result.** The reply came back: "Existing system, they want to see how you would extend it." That single sentence moved my preparation from greenfield drills to the brownfield checklist: dual writes, backfill, shadow traffic and rollback.

### 1e. Fade the scaffold

**Faded example 1.** The invite says: "45 minute technical design round, Search Infrastructure team, please be prepared to write on a shared document." Work through blocks 1 to 3, then produce block 4 yourself.

- Block 1: the signals are 45 minutes, Search Infrastructure, and a text-only tool.
- Block 2: a search infrastructure team probes index build cost, freshness lag, and query fan-out. I drop transactional and payments material.
- Block 3: I ask the recruiter whether the round is component depth or end-to-end.
- Block 4: **you write the opening thirty seconds.**

??? note "Show answer"

    A defensible block 4, adapted to a text-only tool because no one can read your ASCII art while you type it:

    "Since we are in a document rather than a whiteboard, I will keep the diagram to a short indented list and spend the drawing time talking instead. I plan to do five minutes on requirements, then the simplest index and query path that works, then break it on freshness. Stop me if you would rather go deep on the indexing side."

    The tool constraint is a real design input for the round. Candidates who ignore it lose minutes typing box drawings that nobody reads.

**Faded example 2.** The invite says: "60 minutes with a staff engineer, topic to be chosen on the call." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: the signals are 60 minutes, staff level, and an unspecified topic.
- Block 2: an unspecified topic means I cannot subtract by domain, so I subtract by archetype instead and prepare the general arc rather than any one product.

??? note "Show answer"

    Block 3: the cheap information purchase is different here. There is nothing to ask the recruiter about the topic, so I ask about the format instead: "Will the topic be chosen from my background or from their team's domain?" If it comes from my background, I prepare the two systems on my resume in depth, because those will be chosen and the follow-ups will be hostile.

    Block 4: the opening sentence has to handle a topic I have not seen. "Give me sixty seconds to restate the problem in my own words and list what I think the two hardest parts are, then I will tell you which one I want to spend most of our time on." A staff interviewer is scoring your ability to select the hard part, so the opening should visibly perform that selection.

### 1f. Check yourself

**Q1.** An invite says "System Design" and the first line of the round is "Design the retry and error semantics our mobile clients see." Which archetype are you in, and what is the first thing you change about your plan?

??? note "Show answer"

    Contract design, not topology design. The words are error semantics and clients. The first change is to move most of the minute budget from boxes and capacity to the error taxonomy, retry safety per operation, and version skew across clients you cannot force-upgrade. Drawing a load balancer here scores nothing.

**Q2.** You are twenty minutes into a round you believed was greenfield, and the interviewer says "assume all of that already exists in production today". What just changed?

??? note "Show answer"

    It became a brownfield round. Every subsequent proposal now needs a migration story: how the change ships without downtime, what runs in parallel, what the rollback is, and what the blast radius is if it is wrong. A design that would have scored well as greenfield scores poorly here if you present it as a replacement rather than as a sequence.

### 1g. When not to use this

Round triage is a preparation tool, not a ritual. It is over-applied more often than under-applied.

| Question | Answer |
|---|---|
| What measurement justifies doing triage | You are preparing for two or more loops, or you cannot name your round's archetype from the invite |
| Threshold below which it is over-engineering | One round, one written round list from the recruiter that already names the format. Triage adds nothing |
| Cheaper alternative | Send one message to the recruiter. Two sentences of reply beat an hour of inference |
| The failure mode of over-applying it | Spending preparation hours on meta-strategy instead of on the arc in modules 3 to 8. Triage is worth about 30 minutes total |

---

## Module 2. The delivery clock

**Time: 25 to 30 minutes reading, 15 to 20 minutes practice.**

### 2a. Predict first

Here is a real-shaped timeline from a 45 minute round that did not pass.

```
minute 0    greeting and introductions
minute 2    interviewer states the prompt
minute 3    candidate draws client box balancer box service box
minute 9    interviewer asks what the write volume is
minute 11   candidate gives first requirement
minute 18   candidate adds a queue a cache and a wide column store
minute 26   interviewer asks how a top ten list is produced
minute 31   candidate says we could shard that
minute 38   interviewer stops the design and asks for questions
minute 45   end

Figure a minute by minute trace of a round that was lost early
```

??? note "Show answer"

    Prediction asked for: at which minute was this round already lost?

    Minute 3. One minute after the prompt, the candidate started drawing. Everything after that is a consequence: the requirement at 00:11 arrives after the topology it was supposed to justify, the three components at 00:18 are unmotivated because no number ruled anything out, and the vague answer at 00:31 is what a design with no data model produces.

    Note what is not the cause: the candidate was not short of knowledge. They named a queue, a cache and a wide column store. Rounds are lost on sequencing far more often than on vocabulary.

### The budget, and where it comes from

A 45 minute slot is not 45 minutes of design. Subtract the parts you do not control.

| Slot overhead | Minutes | Why |
|---|---|---|
| Greeting and introductions | 2 | Standard and unavoidable |
| Your questions at the end | 5 | Interviewers protect this and cutting it looks incurious |
| **Working minutes left** | **38** | 45 minus 7 |

The same subtraction on a 60 minute slot leaves 52 working minutes (60 minus 2 minus 6, where the longer slot usually gets a slightly longer question block).

Allocate the working minutes to phases that produce score. Each figure below is a target with roughly plus or minus 2 minutes of slack, and each column sums to the working total.

| Phase | 45 min round | 60 min round | What it buys |
|---|---|---|---|
| Clarify and scope | 6 | 8 | Removes the wrong problem before you solve it |
| Requirements as numbers | 3 | 4 | Gives every later choice something to point at |
| Estimation, only if it eliminates | 2 | 3 | Rules out designs, or is skipped out loud |
| Entities and one interface sketch | 4 | 5 | Prevents the vague answer at minute 31 |
| V0 diagram, smallest correct thing | 5 | 6 | Demonstrates restraint, which is scored |
| Breaking point and V1 | 7 | 9 | The core of a senior rating |
| One deep dive, chosen deliberately | 7 | 11 | The core of a staff rating |
| Failure modes and operability | 3 | 4 | Separates a design from a drawing |
| Wrap and trade-off statement | 1 | 2 | Leaves the interviewer with your summary |
| **Total** | **38** | **52** | |

**Three checkpoints, and the sentence to say at each.** Say these out loud. They buy you correction opportunities and they read as control, not doubt.

| Clock | Sentence | What it protects |
|---|---|---|
| Minute 8 of 45 | "Here is what I think we are building and the three numbers that matter. Anything you would change before I draw?" | Stops you solving the wrong problem |
| Minute 20 of 45 | "That is the simplest version and I believe it is correct at the volumes we said. I am going to break it now." | Signals deliberate restraint rather than ignorance |
| Minute 33 of 45 | "I have about five minutes. I can go deeper here or cover failure modes. Which is more useful?" | Hands the interviewer the last allocation decision |

**How the split shifts by level.** The total is fixed, so every addition is a subtraction.

| Level | Takes minutes from | Spends them on |
|---|---|---|
| Mid | Deep dive | V0 correctness, data model, being explicitly right at the stated scale |
| Senior | V0 | Breaking point, V1, and one mechanism explained to its internals |
| Staff | Estimation and V0 | Framing the problem differently, cost per unit of traffic, migration order, and naming what they chose not to build |

!!! tip "The one acronym note in this course"

    Paid courses in this category sell a memorised sequence, usually as an acronym such as RESHADED or SCADET, and then apply it identically to every prompt. Two problems. First, interviewers now screen for the recited-template tell, and a candidate who runs the same eight headings on a rate limiter and on a video platform sounds like a form being filled in.

    Second, an acronym encodes an order but not a budget, so it does not tell you what to cut when you are eleven minutes behind. We teach a budget and then teach when to break it.

### The clock as a diagram

```
45 MINUTE ROUND  each block is one minute

+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|intro| clarify + numbers |  V0  | break to V1 |deep dive|
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
 0    2                  13     18            25        32

+--+--+--+--+--+--+
| fail | wrap | Q |
+--+--+--+--+--+--+
32    35    36   45

checkpoints
  minute 8  = confirm the problem
  minute 20 = declare V0 done and break it
  minute 33 = offer the interviewer the last choice

Figure the 45 minute budget with three spoken checkpoints
```

### Worked example: running the clock on a small prompt

The prompt at minute 2: "Design a service that stores and serves user avatar images."

**Block 1. PURPOSE: spend the first six minutes buying constraints, not drawing.**

I ask four questions and stop. How many users and how often do they change an avatar. What sizes do clients request. Is the image public or access controlled. Does an upload have to be visible immediately or is a few seconds acceptable. I get: 50 million users, an avatar changed on average once a year, four rendered sizes, public, and a few seconds is fine.

I say the derived shape out loud: this is an extremely read heavy, almost immutable, small object workload with a tiny write rate.

!!! note "Say it before you read on"

    Before reading block 2, say why "changed once a year" is the single most design-relevant answer of the four.

**The error most readers make here** is asking questions that cannot fork the design, such as "what is the tech stack". A question earns its 30 seconds only if two different answers produce two different drawings.

**Block 2. PURPOSE: convert answers into two numbers that eliminate options.**

Writes: 50 million changes per year. A year is about 3 times 10 to the 7 seconds, so that is under 2 uploads per second. That number eliminates every write-scaling mechanism in my vocabulary. No sharded write path, no ingestion queue for throughput reasons.

Reads: assume 20 avatar views per active user per day and 10 million daily active users, so 200 million reads per day. Divide by roughly 10 to the 5 seconds per day: about 2,000 reads per second. Both assumptions are labelled as assumptions and stated as such in the round.

That read number matters because 2,000 requests per second of small immutable public objects is exactly what a content delivery network exists for, and it eliminates the design where the application servers serve image bytes.

**Block 3. PURPOSE: draw the smallest correct thing and say that it is deliberate.**

Upload path: client to service, service writes the original to a blob store, enqueues nothing, and generates the four sizes inline because 2 uploads per second times four resizes is trivial work. Read path: client to content delivery network, cache miss to blob store origin, with the object key derived from user id and a version counter.

I say: "That is the whole design at these numbers. Adding a resize queue or a metadata cache now would be adding a box I cannot justify with a measurement."

**The error most readers make here** is adding a transcode queue reflexively because media plus queue is a memorised pairing. At two uploads per second, the queue's only real function is retry, and a retry column in the database does that more cheaply.

!!! note "Say it before you read on"

    Say what specific measurement would justify adding the resize queue later.

**Block 4. PURPOSE: break my own design on the clock, before the interviewer does.**

At minute 20 I break it: the version counter in the object key is what makes cache invalidation free, so what happens if a user changes their avatar twice in a second and the two resize jobs race? I add a compare-and-set on the version and let the later version win, and I note that the loser's objects are garbage that a lifecycle rule deletes after 30 days.

Then I spend minutes 25 to 32 on the one deep dive that this problem actually makes interesting: cache key design and the moderation hold, where an avatar must be pulled from the edge within seconds of a takedown decision. That is a purge path, and it is the only place where the cheap design genuinely strains.

### 2e. Fade the scaffold

**Faded example 1.** Same clock, 60 minute round. Prompt: "Design a service that lets employees search a company document store." You get blocks 1 to 3, then produce block 4.

- Block 1: I buy constraints. I learn there are 200,000 employees, 40 million documents, an average document of 40 kilobytes, permissions are per document, and freshness of five minutes is acceptable.
- Block 2: numbers. 40 million times 40 kilobytes is about 1.6 terabytes of raw text, which fits on one machine's disk and rules out a distributed index for size reasons alone. Assume 5 searches per employee per working day, so 1 million searches per day, about 12 per second at the daily average and perhaps 60 per second at peak if peak is 5 times average.
- Block 3: V0 is one search index, one crawler that pulls document changes every five minutes, and permission filtering applied at query time.
- Block 4: **you break it and choose the deep dive.**

??? note "Show answer"

    The break: permission filtering at query time is the failure. If a user can see 0.1 percent of 40 million documents, a query that matches 10,000 documents may have zero visible results after filtering, and the index has no way to know how deep to search. This is the classic post-filtering blow-up, and it appears at a specific number: when the visible fraction falls below roughly 1 percent, top-k retrieval stops returning a full page.

    V1: move permissions into the index as access control terms attached to each document, so filtering happens during matching rather than after it. Cost: a permission change now requires reindexing every affected document, which is why the deep dive is not sharding.

    The deep dive this problem makes interesting: permission change propagation. How many documents does one group membership change touch, how long does the reindex take, and what does a user see during the gap. That is a far better use of nine minutes than a generic sharding discussion.

**Faded example 2.** 45 minute round. Prompt: "Design a service that emails a receipt after every purchase." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: I learn there are 2 million purchases per day, receipts must never be sent twice, a delay of up to two minutes is acceptable, and a third-party email provider is used.
- Block 2: 2 million per day is about 20 per second on average. Assume a peak of 5 times average, so 100 per second. That eliminates any throughput-driven architecture and puts the entire difficulty on duplicate suppression and provider failure.

??? note "Show answer"

    Block 3, V0: the purchase transaction writes a row to an outbox table in the same database transaction as the purchase itself. A worker polls the outbox, calls the email provider with an idempotency key equal to the purchase id, and marks the row sent. No message broker, because 100 per second of durable work is comfortably inside what a single relational table plus a couple of workers handles, and the transactional outbox is the only cheap way to make "purchase recorded" and "receipt queued" atomic.

    Block 4, the break and the dive: the break is the provider. If the provider returns a timeout, the worker does not know whether the mail was sent. Retrying risks a duplicate, not retrying risks a missing receipt.

    The deep dive is therefore delivery semantics: idempotency keys accepted by the provider, a sent-state machine with a pending-unknown state, and a reconciliation job that queries the provider's own message log for the ambiguous window. Choosing that dive over "how would you scale the workers" is what separates a senior answer here, because the worker count is trivially derivable and the ambiguity is not.

### 2f. Check yourself

**Q1.** You are at minute 24 of a 45 minute round and you have not drawn anything yet, because clarification kept opening new questions. What do you do?

??? note "Show answer"

    Cut, out loud, and cut requirements rather than the design. Say: "I am going to freeze scope here and design only the read path for a single region, and I will note the three things I am leaving out." Then draw V0 in four minutes. An unfinished drawing plus a stated cut scores far above a complete requirements list with no design. Silence while you recover is the worst option, because the interviewer cannot tell recovery from being stuck.

**Q2.** Why does the budget put more staff-level minutes into framing than into the deep dive, when depth is what staff engineers are known for?

??? note "Show answer"

    Because at staff level the deep dive is assumed and the framing is the differentiator. A staff candidate is scored on whether they chose the right problem: which part is actually risky, what is not worth building, what it costs, and how it ships without breaking the running system. Depth still has to appear, but depth without selection reads as a strong senior answer, not a staff one.

### 2g. When not to use this

The clock is scaffolding. Scaffolding that stays up forever becomes the building's problem.

| Question | Answer |
|---|---|
| What measurement justifies drilling the clock | Run three timed mocks. If you fail to reach a deep dive in two of the three, the clock is your bottleneck |
| Threshold below which it is over-engineering | If you already finish with five or more minutes spare, stop timing yourself and spend the hours on depth instead |
| Cheaper alternative | A single sticky note with three checkpoint minutes on it, and nothing else |

**Three situations where following the clock is the wrong move.**

| Situation | The tell | What to do instead |
|---|---|---|
| The interviewer wants one deep dive | They interrupt your V0 with an internals question in the first ten minutes | Abandon the breadth phases. Give them 25 minutes of depth and 5 minutes of context |
| Estimation would be noise | Both candidate designs work at any volume you could plausibly assume | Say "I can size this if it will change our choice, otherwise I will skip it", then skip it |
| The schema is the whole problem | The prompt is about relationships, permissions or money | Spend 12 minutes on entities and invariants. Boxes and arrows are the cheap part here |
---

## Module 3. Requirements that change the design

**Time: 40 to 50 minutes reading, 20 to 25 minutes practice.**

### 3a. Predict first

A candidate produced this requirements list in the first five minutes of a round on a social product.

| # | Stated requirement |
|---|---|
| 1 | The system should be highly available |
| 2 | Users can follow other users |
| 3 | p99 read latency stays under 200 milliseconds |
| 4 | The system should be scalable |
| 5 | A new post is visible to followers within 5 seconds |
| 6 | The most followed account has about 40 million followers |
| 7 | EU user data stays in the EU |

??? note "Show answer"

    Prediction asked for: which of these seven change a drawing, and which are noise?

    | # | Changes the design | Why |
    |---|---|---|
    | 1 | No | Every system claims this. It changes nothing until it becomes a number and a failure behaviour |
    | 2 | Yes | It sets the fan-out shape, which decides push versus pull |
    | 3 | Yes | It sets a latency budget that a synchronous multi-hop read path cannot meet |
    | 4 | No | Pure noise. Delete it |
    | 5 | Yes | A 5 second staleness budget permits asynchronous fan-out and rules out read-time joins across shards |
    | 6 | Yes | 40 million followers on one account is the single most design-changing fact on the list |
    | 7 | Yes | Residency forces regional partitioning of storage and constrains where the write goes |

    Four of seven earn their place. Requirement 6 is the one most candidates never ask for, and it is the one that decides the architecture.

### The fork test

A clarifying question is worth asking only if two plausible answers produce two different drawings. Everything else is a checklist recital, and interviewers recognise it instantly.

Ask this before every question: **if the answer is A I draw X, if the answer is B I draw Y.** If you cannot fill both halves, do not ask.

**The question classes that actually fork designs.** Five to seven of these, chosen for the prompt, is a full clarification phase.

| Class | The question | If A the design becomes | If B the design becomes |
|---|---|---|---|
| Volume and ratio | Reads per write | Read heavy: caches, denormalised read models, precompute | Write heavy: append-only logs, batched writes, no read cache |
| Freshness budget | How stale may a read be | Seconds tolerated: asynchronous fan-out, materialised views | Must be immediate: read from the source of truth, no view |
| Fan-out shape | What does the largest account or group look like | Uniform: any partitioning works | Heavy tail: hybrid push and pull, hot key handling |
| Correctness class | Is a duplicate acceptable | Yes: at-least-once and move on | No: idempotency keys, dedupe store, reconciliation |
| Query shape | Point lookup, range scan, top-k or full text | Point: key-value store | Range or top-k: ordered index, or a separate serving store |
| Retention | How long is data kept and can it be deleted | Short with a time to live: cheap storage, no compaction worry | Forever with deletion rights: tombstones, crypto shredding, audit |
| Tenancy | Do tenants share storage | Shared: cheap, noisy neighbour risk | Isolated: per tenant shards, higher fixed cost |
| Geography | Where do users and data live | One region: skip the hardest module in this course | Multi region: latency budget, conflict policy, residency |
| Client | Can every client be upgraded | Yes: evolve freely | No: version skew, compatibility windows, server-side shaping |
| Degradation | What does a user see when a dependency is down | Blank or error acceptable: simple | Must degrade gracefully: defaults, cached fallbacks, fail open |

**Turning an adjective into a mechanism.** Interviewers state adjectives. Your job is to convert.

| They said | You say back | The design consequence |
|---|---|---|
| Highly available | "Can I take that as 99.9 percent monthly, so about 43 minutes down, and what does a user see during that time" | Decides redundancy and degradation, not just replica count |
| Fast | "Fast for whom: p50 or p99, and on which operation" | Decides whether a synchronous multi-hop path is legal |
| Real time | "Sub second, or a few seconds" | Decides push versus poll, and streaming versus batch |
| Consistent | "If two users read at the same instant, must they see the same thing" | Decides quorum, leader routing, or nothing at all |
| Secure | "Which threat: another tenant, an insider, or the internet" | Decides isolation boundary, not a checklist of encryption |

**Tag every requirement.** Write GIVEN if the interviewer said it, ASSUMED if you invented it, DERIVED if it follows from the other two. This single habit prevents the most common credibility failure, which is quietly promoting your own guess into a fact and then defending it.

### One question, drawn as a fork

```
       how stale may a follower feed be

       +---------------------------+
       |                           |
   under 1 second              5 seconds ok
       |                           |
       v                           v
+----------------+        +------------------+
| read time      |        | write time       |
| gather from    |        | fan out into per |
| followed users |        | user feed store  |
+----------------+        +------------------+
       |                           |
       v                           v
+----------------+        +------------------+
| cost moves to  |        | cost moves to    |
| every read     |        | every write and  |
| fan out is n   |        | storage grows    |
+----------------+        +------------------+

Figure one clarifying question forking a feed into two architectures
```

### Worked example: turning a vague prompt into forks

The prompt: "Design a notification system."

**Block 1. PURPOSE: find out what the system is actually for, because the phrase covers three different products.**

I ask one question first: "Is this in-app notifications, push and email delivery to devices, or the alerting layer that decides what is worth notifying about?" The answer is push and email delivery for a consumer app.

That one question just removed two thirds of the possible designs. Notice it forked before any number was discussed.

**The error most readers make here** is starting with capacity questions. Volume clarifies size, but product scope clarifies which system you are drawing, and drawing the wrong system at the right scale scores zero.

!!! note "Say it before you read on"

    Say why "which of these three products is it" outranks "how many notifications per day" as a first question.

**Block 2. PURPOSE: ask the five questions that fork the delivery design, and state each fork out loud.**

| My question | Answer I got | The fork I stated |
|---|---|---|
| Is a duplicate notification acceptable | No for transactional, yes for marketing | Two paths: transactional gets dedupe keys, marketing does not |
| How late can a notification be | Transactional under 30 seconds, marketing hours | One low latency path, one batch path, not one path with priorities bolted on |
| What is the biggest single send | A campaign to 20 million users | The write path must absorb a burst 1,000 times the mean, so a buffer is not optional |
| Do users have per category preferences | Yes, and they change rarely | Preferences are a read-mostly lookup, cacheable, not a join on the hot path |
| Must we prove a notification was delivered | Yes for transactional | Per message state machine and a receipt store, which is more expensive than the send itself |

**Block 3. PURPOSE: convert the fan-out answer into the one number that decides the topology.**

A 20 million user campaign against a steady state I assume to be 50 million notifications per day is the whole story. 50 million per day divided by roughly 100,000 seconds per day is about 500 per second average. One campaign is 20 million messages. If the product wants it delivered in 10 minutes, that is 20 million divided by 600 seconds, about 33,000 per second, which is 66 times the average rate.

That ratio, not the absolute volume, is what forces a decoupled queue plus a rate-shaped sender. I say the ratio out loud, because the ratio is the argument.

!!! note "Say it before you read on"

    Before block 4, say which requirement you would drop first if the interviewer told you the campaign must go out in 60 seconds instead of 10 minutes.

**Block 4. PURPOSE: state what I am deliberately leaving out, and why.**

I say: "I am not designing the alerting or triggering logic, I am not designing the mobile client's display, and I am treating the push provider as an external dependency with its own failure modes. If you want any of those instead, say so now and I will trade one of them for the delivery path."

**The error most readers make here** is leaving scope implicit and then being surprised when the interviewer asks about the part they silently dropped. Stating the cut converts an omission into a decision.

### 3e. Fade the scaffold

**Faded example 1.** Prompt: "Design a system to store and query application logs." Blocks 1 to 3 given, produce block 4.

- Block 1: I ask whether this is for debugging by engineers, for compliance retention, or for alerting. The answer is engineer debugging with a 30 day window.
- Block 2: the forks. Query shape is text search and time range, not point lookup, which rules out a plain key-value store. Writes hugely outnumber reads. Loss of a small fraction of logs during an incident is acceptable, which is a very large permission.
- Block 3: numbers. Assume 2,000 hosts each emitting 100 lines per second at 200 bytes, so 200,000 lines per second and 40 megabytes per second, about 3.5 terabytes per day and roughly 100 terabytes for 30 days after assuming compression halves it.

??? note "Show answer"

    Block 4, the deliberate cut: "I am not designing the agent on each host beyond its buffering behaviour, I am not doing log-based alerting, and schema and parsing are out of scope unless you want them. Given 100 terabytes and a debugging use case, I am also going to design tiered retention: hot searchable storage for 3 days, cheap object storage with slower search for the remaining 27, and I will tell you the number that justifies that split."

    The tiering is the point. A 30 day requirement plus a stated permission to lose a fraction during incidents means the expensive index does not need to cover 30 days, and that single decision may cut the storage bill by most of an order of magnitude. State it as a decision, not as an assumption.

**Faded example 2.** Prompt: "Design a coupon redemption service." Blocks 1 and 2 given. Produce blocks 3 and 4.

- Block 1: it is redemption at checkout, not coupon creation or targeting.
- Block 2: the forks I asked about are whether a coupon can be used twice (no), whether coupons are per user or global (both exist), and what happens if the service is down at checkout (checkout must proceed without the discount).

??? note "Show answer"

    Block 3, the number that decides it: I ask for the worst case, and I get "a single global coupon with 10,000 uses, promoted on television". That is a single hot key with a hard counter and a burst of concurrent decrements.

    If I assume the promotion drives 50,000 redemption attempts in the first 10 seconds, that is 5,000 attempts per second against one row. The absolute traffic is small. The contention on one key is the entire problem, and it eliminates any design that treats redemption as an ordinary write.

    Block 4, the deliberate cut and the degradation rule: "I am not designing coupon issuance or fraud scoring. I am designing redemption as a conditional decrement with an idempotency key per checkout attempt, and because you said checkout must proceed if we are down, I will make the coupon service fail open with the discount omitted rather than fail closed. That means over-redemption is impossible but under-redemption is possible, and I would rather explain a missed discount than a negative balance."

    Naming which direction the system fails in is the senior move here. Most candidates say "it should be highly available" and never state which error they prefer.

### 3f. Check yourself

**Q1.** An interviewer says "assume standard requirements". What do you do?

??? note "Show answer"

    You state them instead of asking for them, and you tag them. "Then I will assume 10 million daily active users, a read to write ratio of about 50 to 1, a few seconds of staleness is acceptable, and single region. Those are my assumptions, all four are cheap to change, and I will call out any place where one of them is doing real work in the design." This is faster than negotiating and it keeps you honest, because a tagged assumption can be corrected mid-round without embarrassment.

**Q2.** Give one clarifying question that fails the fork test, and rewrite it so it passes.

??? note "Show answer"

    Fails: "What database should we use?" Neither answer changes what you draw next, and it asks the interviewer to do your job.

    Passes: "Are queries point lookups by identifier, or range scans over time?" Point lookups let me pick a key-value store and partition by identifier. Range scans force an ordered index and a partitioning key that keeps the range on one node, which is a completely different drawing.

### 3g. When not to use this

| Question | Answer |
|---|---|
| What measurement justifies more clarification | You have asked fewer than five questions and at least one plausible answer would still change the drawing |
| Threshold below which it is over-engineering | You have asked five questions and the last two did not fork anything. Stop and draw |
| Cheaper alternative | State assumptions out loud and tag them ASSUMED. It costs 20 seconds and can be corrected later |
| Hard stop | Minute 8 of a 45 minute round. Past that, clarification is avoidance |

---

## Module 4. Estimation as elimination

**Time: 40 to 50 minutes reading, 20 to 25 minutes practice.**

### 4a. Predict first

Here is an estimation block of the kind that appears in most study material.

```
daily active users            100 million
requests per user per day     20
requests per day              2 billion
requests per second           23148
peak requests per second      46296
storage per record            1 kilobyte
daily storage                 2 terabytes
five year storage             3650 terabytes
bandwidth                     23 megabytes per second

Figure a typical estimation block with nine computed numbers
```

??? note "Show answer"

    Prediction asked for: how many of these nine numbers changed a design decision?

    Zero, as written. Not one of them is followed by a sentence that rules an option out. That is the defect. The block also carries false precision: 23,148 requests per second is a number computed to five significant figures from an input that was a guess to one.

    Repaired, the same block would be three numbers, each with its consequence: "about 20,000 requests per second at the mean, which rules out serving from a single node; about 2 terabytes per day, which rules out keeping the full dataset in memory; peak is only twice the mean, which rules out an ingestion buffer sized for bursts."

    The rule for this course: **a number appears only if the next sentence uses it to eliminate an option, or it is labelled as an assumption you will revisit.**

### The arithmetic, done the way it survives a whiteboard

**Work in powers of ten, and round aggressively.** Precision you did not have in the input cannot appear in the output.

| Conversion | Use this | Why it is safe |
|---|---|---|
| Seconds in a day | 100,000 (true value 86,400) | 16 percent high, which is inside the error of any traffic guess, and it makes division trivial |
| Seconds in a month | 2.5 million | 30 times 86,400 is 2.59 million |
| Seconds in a year | 30 million | 365 times 86,400 is 31.5 million |
| Bytes to terabytes | 1 million million | Keep everything in scientific notation until the last step |
| 1 gigabit per second | 125 megabytes per second | Divide by 8 |
| Requests per day to per second | Divide by 100,000 | 1 million per day is 10 per second |

The last row is the single most useful fact in this module. **One million events per day is ten per second.** Most candidates who cannot estimate under pressure simply do not have this anchored.

**The three quantities worth computing, and nothing else.**

| Quantity | Formula | What it typically eliminates |
|---|---|---|
| Requests per second at peak | daily volume divided by 100,000, times a peak factor you state | Single node serving, or the need for any distribution at all |
| Bytes stored per unit time | records per day times bytes per record | Memory-only designs, single node storage, retention policy |
| Working set size | hot fraction times total records times bytes | Whether a cache is one node or a tier |

**Peak factors are assumptions and must be labelled.** A consumer product with one time zone might peak at 3 to 5 times its daily mean; a global product with flat traffic might peak at 1.5. State which you assume and why. If the design does not change between a factor of 2 and a factor of 5, say so and stop refining.

**Hardware reference points are also assumptions.** Never quote a benchmark you cannot source. Instead, state the shape of the machine you are assuming and invite correction.

| Assumption to state | A reasonable stated value | Why it is reasonable to assume |
|---|---|---|
| Memory on one server | 64 to 256 gigabytes | This is the ordinary range of rentable instances, so it is what a reviewer will picture |
| Local solid state storage per server | Single digit terabytes | Same reason: it matches commonly rented instance families |
| Network per server | Single digit gigabits per second | Same reason |
| Random reads per second from a network attached volume | Thousands unless provisioned higher | Provisioned throughput is a purchasing decision, so say you would measure it |

Say the sentence: "I am assuming a machine with roughly 128 gigabytes of memory. If your machines are much larger, this next conclusion changes." That is honest, it is checkable, and it protects you from a follow-up you cannot win.

**The graceful skip.** Estimation is not always worth its minutes.

> "I can size this if it will change what we build. My instinct is that both designs on the table work at any volume we would plausibly assume, so unless you want the numbers I will spend the time on the failure modes instead."

Interviewers who wanted estimation will ask for it. Interviewers who did not just got three minutes back, and you signalled that you know why the exercise exists.

### The elimination funnel

```
+------------------------------------------+
| stated inputs   users rate size retention |
+------------------------------------------+
                    |
                    v
+------------------------------------------+
| three quantities   qps   bytes   hot set  |
+------------------------------------------+
                    |
      +-------------+-------------+
      |                           |
      v                           v
+------------------+     +--------------------+
| option survives  |     | option eliminated  |
| keep it on board |     | say it out loud    |
+------------------+     +--------------------+
                    |
                    v
+------------------------------------------+
| if nothing was eliminated delete the math |
+------------------------------------------+

Figure estimation as a filter on options not as arithmetic
```

### Worked example: sizing a link shortener so that it removes boxes

GIVEN by the interviewer: 500 million new links created per month, read to write ratio about 100 to 1, links must work for at least 5 years.

**Block 1. PURPOSE: get the write rate, and use it to delete machinery.**

500 million per month divided by 2.5 million seconds per month is 200 writes per second.

Eliminated by that number: a sharded write path, an ingestion queue justified by throughput, and any discussion of write batching. 200 small writes per second is inside what one primary database node handles, so if I later shard, the reason will be storage size, not write rate. I say exactly that sentence, because it pre-empts the follow-up.

!!! note "Say it before you read on"

    Say what would have to be true for 200 writes per second to still require sharding.

**The error most readers make here** is computing the write rate and then adding a queue anyway, because writes plus scale is a memorised pairing. The number you just computed forbids it.

**Block 2. PURPOSE: get the storage curve, and use it to choose the storage class.**

Bytes per record: 7 bytes of short code, an assumed 100 bytes of average destination URL, 8 bytes of owner id, 8 bytes of creation timestamp, 8 bytes of expiry. That is 131 bytes of payload. I assume total on-disk cost is about 500 bytes per record once the primary key index, a secondary index on owner, and per-row overhead are counted, roughly four times payload.

500 million records per month times 500 bytes is 250 gigabytes per month, so 3 terabytes per year and 15 terabytes over the 5 year requirement.

Eliminated by that number: keeping the whole mapping in memory. At an assumed 128 gigabytes of usable memory per cache node, 15 terabytes is over 100 nodes of pure cache, which is an absurd bill for a redirect. Also eliminated: a single node that holds everything comfortably, not because 15 terabytes cannot fit on one machine, but because restoring 15 terabytes from a backup at an assumed 200 megabytes per second takes about 21 hours, and that is the number I would actually cite in the round.

**Block 3. PURPOSE: get the read rate, and decide whether a cache is justified rather than assumed.**

100 to 1 against 200 writes per second is 20,000 reads per second.

Now the elimination that matters. Reads are point lookups on a 15 terabyte dataset, so unless the working set fits in memory, each read is a random disk read. 20,000 random reads per second against a volume I assumed provides low thousands of operations per second means the disk is the wall. That justifies a cache, with a measurement rather than a reflex.

**Block 4. PURPOSE: size the cache, and use the size to eliminate a whole tier.**

Assume link traffic is heavily skewed and that 80 percent of reads hit links created in the last 30 days, and within those, 10 percent of the links serve most of the traffic. Links in 30 days: 500 million. Ten percent is 50 million. At 150 bytes per cached entry, that is 7.5 gigabytes.

Eliminated by that number: a distributed cache tier. 7.5 gigabytes fits in memory on a single cache node with room to spare, so I start with a small replicated cache, not a sharded one. I state the skew as an assumption and name the metric that would falsify it: cache hit ratio below about 80 percent in production means the skew assumption was wrong and the tier gets revisited.

!!! note "Say it before you read on"

    Say which of the four blocks above would change most if the interviewer said links expire after 24 hours.

**Block 5. PURPOSE: check bandwidth, find that it eliminates nothing, and delete it.**

20,000 reads per second times an assumed 500 byte redirect response including headers is 10 megabytes per second, about 80 megabits per second. That rules nothing out at all: it is inside a single machine's network capacity.

So I say one sentence and move on: "Bandwidth is not a constraint here, so I will not spend time on it." Deleting a number out loud is worth more than computing three more.

**The error most readers make here** is presenting every computed number regardless of consequence, which turns the estimation phase into a recital and burns four of the thirty eight working minutes.

### 4e. Fade the scaffold

**Faded example 1.** A photo product. GIVEN: 10 million photos uploaded per day, each stored at 4 sizes, average total 2 megabytes per photo across all sizes, kept forever. Reads are 30 times writes. Produce the last block.

- Block 1, write rate: 10 million per day divided by 100,000 is 100 uploads per second. That eliminates a sharded upload path and any throughput-driven ingestion queue.
- Block 2, storage: 10 million times 2 megabytes is 20 terabytes per day, so about 7 petabytes per year. That eliminates any block storage or database-resident design and forces an object store.
- Block 3, read rate: 100 times 30 is 3,000 reads per second of large immutable objects.
- Block 4: **you decide what the read number eliminates, and size the edge cache.**

??? note "Show answer"

    3,000 reads per second times an assumed 200 kilobytes per served image is 600 megabytes per second, about 4.8 gigabits per second. That eliminates serving images from application servers, because it is several machines' worth of network capacity spent on bytes that never change. It justifies a content delivery network on bandwidth grounds alone, which is a stronger argument than latency.

    Edge cache sizing: assume 90 percent of reads target photos uploaded in the last 7 days. 7 days of uploads is 70 million photos, and if the commonly requested size is an assumed 200 kilobytes, the recent set is 14 terabytes. That does not fit in one cache node's memory, so unlike the shortener, this one genuinely needs a distributed edge tier with disk-backed caching. Note that the same reasoning produced opposite answers in two problems, which is the point of doing the arithmetic rather than memorising the conclusion.

**Faded example 2.** A chat product. GIVEN: 500 million messages per day, average 200 bytes of text, history retained forever, a user opens the app and reads the last 50 messages of each of 5 conversations. Produce the last two blocks.

- Block 1, write rate: 500 million divided by 100,000 is 5,000 messages per second. Assume a peak factor of 3 for a single dominant time zone, so 15,000 per second at peak. That still does not eliminate a single primary for raw insert rate, but it is close enough that I would not build on one.
- Block 2, storage: 500 million times an assumed 500 bytes stored per message once identifiers, timestamps and indexes are counted, is 250 gigabytes per day, about 90 terabytes per year.

??? note "Show answer"

    Block 3, read rate and shape: assume 100 million daily active users each opening the app 10 times a day, so 1 billion sessions a day, and each session reads 5 conversations times 50 messages. That is 250 billion message reads per day, which divided by 100,000 is 2.5 million message reads per second.

    That number eliminates any design that reads messages individually. It forces the read unit to be a contiguous page of one conversation, which in turn forces the partition key to be the conversation and the sort key to be the message sequence, so 50 messages come back in one range read.

    Block 4, what the numbers jointly decide: 90 terabytes per year that is written once, read heavily for hours and then almost never, plus a range-read access pattern, is the exact shape a log-structured store with range scans is built for.

    It also justifies tiering by age, since the read rate on messages older than 30 days is a tiny fraction of the total. The elimination sentence is: "I am not going to keep 5 years of history on the same storage class as today's messages, because the read rate difference between them is at least two orders of magnitude and I would be paying for the difference."

### 4f. Check yourself

**Q1.** You compute 4,000 requests per second for a service. State two different designs this number eliminates, and one it does not.

??? note "Show answer"

    Eliminated: a design that requires per-request coordination through a single strongly consistent counter, since that becomes the whole system's throughput ceiling. Also eliminated: any claim that this needs global distribution for throughput reasons, because 4,000 requests per second of ordinary work is a handful of machines.

    Not eliminated: whether you need a cache. That depends on what each request does, the size of the working set, and the latency budget, none of which the request rate alone tells you. Request rate is the most quoted and least decisive of the three quantities.

**Q2.** An interviewer asks for capacity estimates on a design where both candidate options are a single service with a relational database, differing only in schema. What do you say?

??? note "Show answer"

    Offer the skip, with the reason. "Both options here are one service and one database, so a capacity estimate will not separate them. What would separate them is the query shape at the ninety ninth percentile, so I would rather estimate rows touched per query than requests per second. Shall I do that instead?" You have declined the ritual, named what actually decides the question, and offered a better substitute in one breath.

### 4g. When not to use this

| Question | Answer |
|---|---|
| What measurement justifies estimating | Two designs are on the table and you can name the threshold at which the answer flips |
| Threshold below which it is over-engineering | The two designs differ by less than one order of magnitude in every quantity you could compute |
| Cheaper alternative | State the single number you would measure first in production, and move on |
| The tell you have over-applied it | You have four numbers on the board and have not deleted a single option |

---

## Module 5. Non-functional characteristics as mechanisms

**Time: 55 to 70 minutes reading, 25 to 35 minutes practice.**

### 5a. Predict first

A checkout service is promised at 99.99 percent availability. Its request path calls six internal services in sequence, each independently rated at 99.95 percent.

??? note "Show answer"

    Prediction asked for: what availability can checkout actually deliver, and how far is it from the promise?

    Serial dependencies multiply. Six services at 99.95 percent each means six unavailability budgets of 0.05 percent stacking up. For small numbers the product is close to the sum, so total unavailability is about 6 times 0.05, which is 0.30 percent, giving roughly 99.70 percent.

    Converting to time: a 30 day month is 43,200 minutes. 0.30 percent of that is about 130 minutes per month. The promise of 99.99 percent allowed 4.3 minutes. The design is off by a factor of 30, and no amount of replication inside any one of the six services fixes it, because the arithmetic is about the length of the chain, not the health of the links.

    The design move is to shorten the critical path, not to add nines to its members.

### Availability, as arithmetic you can do on a whiteboard

**The conversion table.** A 30 day month is 43,200 minutes. Every figure below is that number times the unavailability fraction.

| Availability | Unavailable per month | Unavailable per year |
|---|---|---|
| 99 percent | 432 minutes | 3.65 days |
| 99.9 percent | 43.2 minutes | 8.76 hours |
| 99.95 percent | 21.6 minutes | 4.38 hours |
| 99.99 percent | 4.32 minutes | 52.6 minutes |
| 99.999 percent | 26 seconds | 5.26 minutes |

Two things fall out immediately. First, 99.99 percent means you cannot afford a single unplanned human intervention in a month, since paging a human and having them act takes longer than 4.32 minutes. That is why four nines is an automation claim, not a redundancy claim. Second, the difference between three and four nines is a factor of ten in cost and effort, so it must be justified by a stated business consequence.

**Series and parallel.**

| Shape | Formula | Consequence |
|---|---|---|
| A depends on B depends on C | multiply the availabilities | Every added hop on the critical path lowers the ceiling |
| Two independent replicas, either suffices | 1 minus the product of the failure rates | Two 99 percent replicas give 99.99 percent if failures are independent |

**Independence is the assumption that fails.** Replicas share a control plane, a configuration push, a deployment pipeline, a certificate, a DNS zone and often a region. A shared dependency turns a parallel calculation back into a serial one. Treat the parallel formula as an upper bound and say so, because interviewers probe exactly here.

**The design move is dependency reduction.** For each dependency on the critical path, ask: can it be removed, cached, defaulted, or made asynchronous?

| Technique | What it does to the arithmetic | Cost |
|---|---|---|
| Remove from critical path | Deletes its term entirely | Feature loss or asynchronous behaviour |
| Serve from cache on failure | Its term becomes near 1, at the cost of staleness | Stale data, and a cache that must be warm |
| Default value on failure | Its term becomes near 1 | The default must be safe in the worst case |
| Make it asynchronous | Moves it off the read path | Needs durable handoff, or the work is lost |
| Hedge across two providers | Its term becomes the product of both failures | Double integration cost and reconciliation |

### Durability is a different question from availability

| Property | Question it answers | What it is bought with |
|---|---|---|
| Availability | Can I reach it right now | Redundancy on the serving path, failover, degradation |
| Durability | Will it still exist tomorrow | Replica count, spread across failure domains, backups, verification |

A system can be highly durable and frequently unavailable, or highly available and lose data. Say which one the prompt cares about. Do not quote a vendor's durability figure you cannot source; instead state the mechanism: how many copies, in how many independent failure domains, and how you would detect that a copy has silently rotted.

### The consistency spectrum, with the mechanism that buys each level

Strong versus eventual is not a spectrum, it is a two item list, and it is the reason so many candidates cannot answer a follow-up. Here is the usable version, ordered from strongest.

| Level | What a client observes | Mechanism that provides it | What it costs |
|---|---|---|---|
| Linearizable | Every read sees the latest completed write, as if one copy existed | Consensus, or reads routed through a leader with a lease | Cross node round trip on every operation, unavailable during partition on the minority side |
| Sequential | All clients see operations in the same order, not necessarily real time order | Total order broadcast | Same ordering cost, weaker real time guarantee |
| Causal | Effects never appear before their causes | Version vectors or dependency tracking on writes | Metadata per item, and it grows with the number of writers |
| Read your writes | A client always sees its own writes | Sticky routing to a replica, or a token carrying the write position | Session state, and a fallback when the sticky target is gone |
| Monotonic reads | A client never sees time go backwards | Pin a session to one replica, or carry the last seen version | Same as above |
| Bounded staleness | Reads are behind by at most a stated bound | Replication lag monitoring plus refusal to serve when lag exceeds the bound | Availability drops when lag spikes, which is a deliberate trade |
| Eventual | Replicas converge if writes stop | Asynchronous replication | Anomalies visible to users, and a conflict policy is mandatory |

**The interview move.** Say which level each operation needs, not which level the system has. In a payments product the balance decrement is linearizable and the transaction list is read your writes. Those are different mechanisms in the same service, and saying so is worth more than any single choice.

### Latency, percentiles and fan-out amplification

Averages hide the failure. Use percentiles, and know what happens when a request fans out.

If one backend serves a request in under 100 milliseconds 99 percent of the time, and a request must contact **n** independent backends and wait for all of them, the chance that every one is fast is 0.99 to the power n.

| Fan-out n | Probability all n are fast | Fraction of requests hitting a slow backend |
|---|---|---|
| 1 | 0.99 | 1 percent |
| 10 | 0.90 | about 10 percent |
| 100 | 0.37 | about 63 percent |

At a fan-out of 100, the ninety ninth percentile of one backend has become the median experience of the user. This effect and the mitigations for it, including sending a duplicate request and cancelling the loser, are described in Dean and Barroso, "The Tail at Scale", Communications of the ACM, February 2013 (<https://cacm.acm.org/research/the-tail-at-scale/>).

Design consequences that follow directly:

- Reduce fan-out before optimising any single backend.
- Set per-dependency timeouts below the total budget, and make the budget explicit in the design.
- Prefer returning a partial answer over waiting for the slowest shard, when the product allows it.

### Failure models, named

You cannot design against a failure you have not named. Use these five and say which ones you are handling.

| Model | What happens | Typical design response |
|---|---|---|
| Crash stop | A node stops and does not return | Health checks, replica promotion |
| Crash recovery | A node stops and comes back with old state | Fencing tokens, epoch numbers, refuse stale writes |
| Omission | Messages are dropped in one direction | Timeouts plus idempotent retry |
| Partition | Two halves are alive and cannot talk | Choose which side keeps serving, and say so |
| Gray or limping | A node is alive, slow, and passing health checks | Latency-based ejection, outlier detection, request hedging |

Gray failure is the one candidates forget and the one that causes the worst incidents, because every automated system believes the node is healthy while every user experiences it as broken.

### The availability chain, drawn

```
+--------+   +--------+   +--------+   +--------+
| client |-->|  edge  |-->|checkout|-->|  auth  |
+--------+   +--------+   +--------+   +--------+
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
        +----------+    +----------+    +----------+
        | pricing  |    | catalog  |    |  fraud   |
        +----------+    +----------+    +----------+
              |
              v
        +--------------+
        | card gateway |
        +--------------+

critical path is every box a failure of which fails checkout
each box on that path multiplies into the achievable number

Figure the checkout critical path where availability is multiplied
```

### Worked example: turning "highly available and fast" into mechanisms

The prompt: a checkout service. The interviewer says it must be "highly available and fast".

**Block 1. PURPOSE: convert two adjectives into numbers with consequences attached.**

I ask what a minute of checkout downtime costs, and get "a lot, but we have never measured it". So I propose a target instead: 99.95 percent monthly, which is 21.6 minutes. I say why I did not propose 99.99: four nines allows 4.3 minutes a month, which is less than the time to page a human, so it is a commitment to fully automated recovery of every failure mode including the third-party card gateway, which we do not control.

For latency I propose p99 under 800 milliseconds for the checkout call, and I state the split I intend to defend: 200 milliseconds of my own work and 600 milliseconds of dependency budget.

**The error most readers make here** is accepting "highly available" and writing 99.99 percent because it sounds strong. A number you cannot mechanically achieve is worse than a lower number you can defend.

**Block 2. PURPOSE: compute the achievable number from the dependency chain, before designing anything.**

Assumed ratings, stated as assumptions: auth 99.99, pricing 99.95, catalog 99.9, fraud 99.9, card gateway 99.9. Serial unavailability is 0.01 plus 0.05 plus 0.1 plus 0.1 plus 0.1, which is 0.36 percent, so about 99.64 percent, or roughly 155 minutes per month.

The target was 21.6 minutes. I am off by a factor of 7 before I have drawn a single box, and I say this out loud, because it reframes the entire round: this is not a scaling problem, it is a dependency problem.

!!! note "Say it before you read on"

    Say which two dependencies you would attack first, and why the ones you picked are the cheapest to remove.

**Block 3. PURPOSE: shorten the critical path rather than add nines to its members.**

| Dependency | Decision | New contribution |
|---|---|---|
| Catalog | Remove from path: the cart already holds the item snapshot taken at add-to-cart time | 0 |
| Fraud | Fail open under a risk cap: if fraud scoring is unavailable, accept orders below an assumed value threshold and queue them for asynchronous review | near 0 |
| Pricing | Keep, but serve from a short-lived local cache on failure, accepting bounded staleness of a few minutes | near 0 |
| Auth | Keep on the path. A checkout that authenticates nobody is not a checkout | 0.01 percent |
| Card gateway | Integrate a second provider and fail over | product of two failure rates |

New serial unavailability: 0.01 percent from auth, plus the gateway leg. If both gateways are at 99.9 percent and their failures are independent, that leg contributes 0.001 times 0.001, which is negligible. Total is about 0.01 percent, so roughly 99.99 percent on paper.

I immediately discount it. Independence between two payment providers is plausible but not guaranteed, since both may depend on the same card networks, and my own service and its data store are not in the estimate at all. I claim 99.95 percent with a mechanism behind it, not 99.99 percent from arithmetic.

**The error most readers make here** is treating the parallel formula as a fact. Say the independence assumption out loud every single time you use it.

**Block 4. PURPOSE: state consistency per operation and pick the failure direction.**

| Operation | Level needed | Mechanism | Failure direction if I cannot meet it |
|---|---|---|---|
| Reserve inventory | Linearizable on the item key | Conditional update on a single owning partition | Fail closed: refuse the order rather than oversell |
| Charge the card | Effect exactly once | Idempotency key per checkout attempt plus a state machine with an unknown state | Fail closed, then reconcile |
| Write order history | Read your writes | Route the confirmation read to the primary or carry a write token | Fail open: show a pending state |
| Loyalty points | Eventual | Asynchronous event, retried | Fail open: award later |

Saying "linearizable here, eventual there, and here is why" is the difference between a recited consistency definition and a design.

!!! note "Say it before you read on"

    Before reading the fade, say what a user sees during the 21.6 minutes a month this design is allowed to be down, and whether that experience is acceptable.

### 5e. Fade the scaffold

**Faded example 1.** A media playback service. Target proposed at 99.9 percent monthly. Critical path: edge, playback session service, entitlement check, licence server, content delivery network. Assumed ratings: edge 99.99, session 99.95, entitlement 99.95, licence 99.9, delivery network 99.99. Produce the last block.

- Block 1: numbers. 0.01 plus 0.05 plus 0.05 plus 0.1 plus 0.01 is 0.22 percent, so about 99.78 percent, which is 95 minutes a month against a 43 minute budget.
- Block 2: the licence server dominates at 0.1 percent, contributing nearly half the total.
- Block 3: **you shorten the path and state the consistency and failure direction per operation.**

??? note "Show answer"

    Shortening: issue licences with a lifetime of an assumed 24 hours and cache them on the device, so the licence server is off the path for any user who played something in the last day. If an assumed 80 percent of sessions are from such users, the licence term drops to 0.2 times 0.1 percent, which is 0.02 percent. Entitlement can be attached to the session token at login with a short expiry, removing it from the per-play path as well.

    New total: 0.01 plus 0.05 plus 0.02 plus 0.01, about 0.09 percent, roughly 99.91 percent, which meets the target with almost no margin. I would say that out loud rather than claiming comfort.

    Consistency and failure direction: entitlement revocation is the interesting case. Caching a licence for 24 hours means a cancelled subscription can keep playing for up to a day. That is a bounded staleness decision, and it must be stated as a product choice with a number attached, not hidden. For high value content the bound would be minutes, and the availability arithmetic would then have to be paid for a different way.

**Faded example 2.** A ride dispatch service, target 99.95 percent. Critical path: mobile edge, dispatch, driver location store, map routing provider, pricing, payments pre-authorisation. Produce the last two blocks.

- Block 1: assumed ratings are edge 99.99, dispatch 99.95, location store 99.95, routing provider 99.9, pricing 99.95, pre-authorisation 99.9. That sums to 0.01 plus 0.05 plus 0.05 plus 0.1 plus 0.05 plus 0.1, which is 0.36 percent, about 99.64 percent, or 155 minutes a month against a 21.6 minute budget.

??? note "Show answer"

    Block 2, where the budget goes: the two third-party-ish legs, routing and pre-authorisation, contribute 0.2 of the 0.36. Attacking dispatch or the location store first would be optimising the wrong term.

    Block 3, shortening the path: routing can be removed from the dispatch decision by falling back to straight-line distance when the provider is unavailable, which produces slightly worse matches rather than no rides. Pre-authorisation can move off the critical path entirely for riders with a payment method that has succeeded recently, with the authorisation done asynchronously and the ride cancelled later if it fails.

    Both are fail-open choices, and both need a stated cap: no fallback matching beyond an assumed 5 kilometre radius, and no deferred authorisation for accounts younger than an assumed 30 days.

    Block 4, consistency per operation: the assignment of a driver to a rider must be linearizable on the driver key, because two riders assigned the same driver is a visible product failure and a support cost.

    Driver location updates are the opposite: they are eventual, lossy by design, and the newest value wins, since a location that is 3 seconds stale is worth more than a consistent one that is 30 seconds stale. Naming those two as opposite ends of the spectrum inside the same system is the answer this problem is really asking for.

### 5f. Check yourself

**Q1.** A candidate says "we will add a replica, so availability goes from 99 to 99.99 percent". What is the strongest objection?

??? note "Show answer"

    The parallel formula assumes independent failures, and replicas are rarely independent. They usually share a deployment pipeline, a configuration source, a control plane, a certificate authority and often a rack or region. A bad configuration push or an expired certificate takes both out simultaneously, and those are common causes of real outages.

    The honest phrasing is: "two replicas raise the ceiling to 99.99 percent for uncorrelated hardware failure, and do nothing for correlated software failure, which is the larger cause. Here is what I would do about the correlated part."

**Q2.** Which is worse for a user: a service that is unavailable for 40 minutes once a month, or one that is unavailable for 80 seconds every day? Both compute to about 99.9 percent.

??? note "Show answer"

    It depends entirely on the product, which is the point: a single availability number does not capture user experience. For a batch or back office system, thirty daily blips are almost invisible and one long outage is a serious incident. For an interactive consumer product, a daily failure trains users to distrust the product, while a single monthly outage is forgivable.

    The design response is to specify frequency and duration separately, and to say which of the two the design optimises. Interviewers rate this observation highly because it shows you have operated something.

### 5g. When not to use this

**Availability targets above three nines.**

| Question | Answer |
|---|---|
| What measurement justifies it | A costed consequence of downtime per minute, and an existing on-call rotation already meeting the tier below |
| Threshold below which it is over-engineering | If nobody can state the cost of a minute of downtime, do not design for four nines. Design for three, measure, and revisit |
| Cheaper alternative | Three nines plus a well tested degradation path. Graceful degradation buys more perceived reliability per unit of effort than an extra nine |

**Multi-zone and multi-region redundancy.**

| Question | Answer |
|---|---|
| What measurement justifies it | Correlated failure history: a zone level event has actually cost you more than your error budget, or residency law requires it |
| Threshold below which it is over-engineering | A service with fewer than a few thousand requests per second and no regulatory driver. Multi-region roughly doubles infrastructure and adds a conflict policy you must maintain forever |
| Cheaper alternative | Multiple zones in one region, plus tested restore from backup with a stated recovery time objective |

**Linearizable operations.**

| Question | Answer |
|---|---|
| What measurement justifies it | A concrete anomaly with a cost: overselling, double spending, split assignment |
| Threshold below which it is over-engineering | Any operation where a stale read produces a mildly wrong display and no financial or safety consequence |
| Cheaper alternative | Read your writes for the acting user, eventual for everyone else. It covers most of the perceived correctness at a fraction of the coordination cost |
---

## Module 6. Building blocks I, the request path

**Time: 65 to 80 minutes reading, 30 to 45 minutes practice.**

### 6a. Predict first

```
+--------+   +-----+   +---------+   +--------+   +--------+
| client |-->| DNS |-->| region  |-->|  L7    |-->| service|
|        |   | TTL |   | entry   |   | proxy  |   | pool   |
|        |   | 300 |   | address |   |        |   |        |
+--------+   +-----+   +---------+   +--------+   +--------+

health check runs every 10 seconds and needs 3 failures
DNS record is withdrawn when the region is marked unhealthy

Figure a DNS based failover path with a five minute record lifetime
```

??? note "Show answer"

    Prediction asked for: the region dies completely at time zero. How long until essentially no user traffic reaches it?

    Add the terms.

    | Term | Time | Why |
    |---|---|---|
    | Detection | 30 seconds | 3 failures at a 10 second interval |
    | Withdrawal and propagation | seconds to tens of seconds | The authoritative record changes and caches begin to expire |
    | Record lifetime | up to 300 seconds | A resolver that fetched the record one second before the failure keeps it for its full life |
    | Clients that ignore the lifetime | unbounded | Some runtimes and libraries cache resolved addresses for the life of the process |

    Best case is around a minute, typical is five to six minutes, and the long tail is unbounded because you do not control client caching behaviour.

    The design conclusion: **name resolution is a traffic steering mechanism, not a failover mechanism.** If the requirement is failover in seconds, the mechanism has to live below the name: a shared address announced from several locations, or a proxy layer that reroutes without the client re-resolving anything.

### The path, component by component

Every request crosses the same five layers. Each layer has one job, one characteristic failure, and one threshold below which it should not exist.

| Layer | Its one job | Characteristic failure |
|---|---|---|
| Name resolution | Turn a name into an address, with policy | Stale caches during failover |
| Global steering | Choose which location serves the user | Sending users to a far region during a flap |
| Local balancing | Choose which instance serves the request | Even distribution onto an uneven backend |
| Gateway | Apply cross-cutting policy once | Becoming a place where business logic accumulates |
| Edge cache | Serve bytes without touching the origin | Cache key mistakes that leak or fragment |

**Name resolution and global steering.** Two mechanisms, often confused.

| Mechanism | How it steers | Failover speed | Cost |
|---|---|---|---|
| Policy in the name system | Returns different addresses by geography, weight or health | Bounded below by the record lifetime, plus client caching | Low, and it works everywhere |
| One address announced from many places | Network routing carries the user to the nearest announcement | Seconds to tens of seconds, as routes converge | Requires address space and network operations, and route changes can break long lived connections |

A software load balancer designed to keep connections mapped to the same backend while the backend set changes is described in Eisenbud and others, "Maglev: A Fast and Reliable Software Network Load Balancer", USENIX NSDI 2016 (<https://www.usenix.org/conference/nsdi16/technical-sessions/presentation/eisenbud>). The reason that paper matters for interviews is the problem it names: when the set of servers changes, a naive hash reshuffles every connection, and long lived connections die.

**Layer 4 versus layer 7 balancing.**

| | Layer 4 | Layer 7 |
|---|---|---|
| Decides using | Addresses and ports | Path, headers, method, cookies |
| Can do | Fast forwarding, connection level failover | Routing by path, retries, header rewriting, response caching |
| Cannot do | Anything content aware | Cheap forwarding, since it must parse |
| Cost per request | Very low | Higher: parsing, and usually a second connection |
| Use when | Raw throughput and protocol agnosticism matter | You need routing, retries or policy per request |

**Balancing algorithms, compared on the property that actually matters.**

| Algorithm | Behaviour | Fails when |
|---|---|---|
| Round robin | Even request counts | Requests have very different costs |
| Weighted round robin | Even counts scaled by capacity | Weights are static and reality is not |
| Least connections | Sends to the least busy | A stuck backend holds few connections and looks idle |
| Least latency, moving average | Sends to the fastest recently | Can stampede onto a newly healthy node |
| Two random choices, pick the better | Near optimal balance with almost no state | Very small backend pools |
| Consistent hash on a key | The same key lands on the same backend, cache friendly | Skewed keys create a hot backend |

Consistent hashing distributes keys with minimal movement when the backend set changes, but it says nothing about load. Adding a per-backend capacity limit that forces overflow to the next backend keeps both properties, an approach set out in Mirrokni, Thorup and Zadimoghaddam, "Consistent Hashing with Bounded Loads", 2016 (<https://arxiv.org/abs/1608.01350>).

**Health checks, and the way they cause outages.**

| Choice | Upside | The trap |
|---|---|---|
| Shallow check, process is alive | Cannot cascade | Misses a node that is up and broken |
| Deep check, calls dependencies | Catches real brokenness | A dependency outage marks every node unhealthy at once, and the balancer has nowhere to send traffic |
| Passive checks from real traffic | Free, and reflects reality | Slow to react on low traffic paths |

The mandatory mitigation for deep checks is a fail-open rule: if more than an assumed 50 percent of backends are marked unhealthy simultaneously, ignore the checks and keep serving, because the far more likely explanation is that the check itself is wrong. Google's SRE material discusses the general shape of this problem under overload and cascading failure (<https://sre.google/sre-book/handling-overload/>, 2016).

**Gateway.** One place to apply authentication, rate limiting, request identifiers, routing and payload limits. Its real risk is not performance, it is ownership: gateways accumulate per-endpoint special cases until nobody can change them. State the rule out loud in the round: only policy that is genuinely common to all services lives here.

**Edge caching: push versus pull.**

| | Pull | Push |
|---|---|---|
| How content arrives | First request for an object fetches it from the origin | You upload the object to the edge before it is requested |
| First request | Slow, and a burst of misses can hammer the origin | Fast |
| Best for | Large catalogues where most objects are cold | Small sets of predictable, high value objects, and event driven spikes |
| Operational cost | Low | You now own distribution and invalidation |

Cache offload is arithmetic: origin requests equal total requests times one minus the hit ratio. At 20,000 requests per second and a 95 percent hit ratio, the origin sees 1,000 per second. Improving the ratio to 99 percent takes the origin to 200 per second, a five times reduction from a four point change. That non-linearity is why cache key design is worth interview minutes.

**Cache key design, the three mistakes.**

- Including a value that varies per user, such as a session identifier, which reduces the hit ratio to near zero.
- Excluding a value that changes the response, such as an accept-language header, which serves the wrong content.
- Using a mutable path with a short lifetime instead of an immutable versioned path, which forces you to build a purge system you did not need.

**Rate limiting algorithms, compared on memory, burst and coordination.** These numbers are derived from stated inputs, not measured.

Assume a limit of 1,000 requests per minute per key and 5 million active keys.

| Algorithm | State per key | Memory at 5 million keys | Burst behaviour |
|---|---|---|---|
| Fixed window counter | 1 counter, 8 bytes | 40 megabytes plus key overhead | Allows up to 2 times the limit across a window boundary |
| Sliding log | 1 timestamp per request, 8 bytes each, up to 1,000 | up to 40 gigabytes | Exact, no boundary artefact |
| Sliding window counter | 2 counters, 16 bytes | 80 megabytes plus key overhead | Approximates the log within a few percent |
| Token bucket | tokens plus last refill time, 16 bytes | 80 megabytes plus key overhead | Allows a burst up to bucket capacity, then a steady rate |
| Leaky bucket as a queue | queue of pending requests | Depends on queue depth | Removes bursts entirely by delaying them |

The sliding log row is the whole argument. Exactness costs three orders of magnitude more memory than the approximation, which is why almost nobody runs it above trivial limits. Stripe published a description of running token bucket and concurrency limiting in a shared cache in production (<https://stripe.com/blog/rate-limiters>, 2017).

**Coordination cost is the second axis, and the one candidates forget.**

| Placement | Accuracy | Cost |
|---|---|---|
| Central store, one call per request | Exact against the global limit | One network round trip per request, and the store must sustain the full request rate |
| Local per node, limit divided by node count | Wrong whenever traffic is uneven across nodes | Free |
| Local with leases from a central store | Overshoot bounded by nodes times lease size | Central load reduced by the lease size |

### The request path, drawn

```
+--------+    +-----------+    +-------------+
| client |--->| name and  |--->| edge cache  |
|        |    | steering  |    | and TLS     |
+--------+    +-----------+    +-------------+
                                     |
                                 miss only
                                     v
                               +-------------+
                               |  gateway    |
                               | auth limit  |
                               +-------------+
                                     |
                                     v
                               +-------------+
                               | L7 balancer |
                               +-------------+
                               |      |      |
                               v      v      v
                          +------+ +------+ +------+
                          | svc  | | svc  | | svc  |
                          +------+ +------+ +------+

Figure the five layers a request crosses before reaching a service
```

### Worked example: choosing a rate limiter with numbers, not taste

GIVEN: a public interface serving 200,000 requests per second at peak, 5 million active keys, 50 gateway nodes, a published limit of 1,000 requests per minute per key with a permitted burst of 50, and a p99 latency budget of 800 milliseconds end to end.

**Block 1. PURPOSE: eliminate algorithms on memory before discussing anything else.**

From the table above: the sliding log needs up to 8 kilobytes per key at this limit, so 5 million keys is up to 40 gigabytes of state. That exceeds an assumed 128 gigabyte cache node once replication and overhead are counted, and it buys exactness nobody asked for.

Token bucket needs 16 bytes of state plus key overhead. Assume 64 bytes of total overhead per entry in a real cache, so 80 bytes per key, giving 400 megabytes for 5 million keys. That is a rounding error on one node.

I choose token bucket, and I say why in one sentence: the published contract mentions a burst of 50, and token bucket is the only algorithm in the table that expresses a burst allowance as a first-class parameter rather than as an accident of window boundaries.

!!! note "Say it before you read on"

    Say why the permitted burst of 50 in the contract, on its own, eliminates the fixed window counter.

**The error most readers make here** is choosing the sliding log because it is described as accurate. Accuracy is not the requirement. The requirement is a published contract with a burst allowance, and 40 gigabytes is the price of an accuracy nobody can observe.

**Block 2. PURPOSE: decide where the state lives, using the request rate rather than a preference.**

Central placement means the limiter store handles 200,000 operations per second, one per request. Assume a single cache node comfortably serves around 100,000 simple operations per second, an assumption I would verify before building anything on it. Then a central design needs at least 2 shards, and I would run 4 for headroom and failure tolerance. Sharding is trivial here because the natural shard key is the limiter key itself, and there are no cross-key operations.

Local placement means each of the 50 nodes enforces 1,000 divided by 50, which is 20 requests per minute. That is only correct if each key's traffic is spread evenly across all 50 nodes. It is not: connections are long lived, and a single client with one connection hits one node and gets 20 requests per minute against a published 1,000. That is a contract violation, not an approximation, so local-only is eliminated.

**Block 3. PURPOSE: check whether the network hop I just added is affordable.**

Assume a same-zone round trip to the cache of 1 millisecond at the median and 3 milliseconds at the ninety ninth percentile, which is a figure I would measure rather than trust. Against an 800 millisecond p99 budget, 3 milliseconds is 0.4 percent. The hop is affordable, and I say the percentage rather than the milliseconds, because the percentage is the argument.

The real risk is not latency, it is coupling: if the limiter store is unavailable, does the interface stop? I choose fail open with a local fallback bucket, because rejecting all traffic to protect a limit is a worse outcome than briefly allowing more traffic than the contract. I state the cap on that decision: fail open only while fewer than an assumed 5 percent of requests are affected, and alert immediately.

!!! note "Say it before you read on"

    Say which product would make the opposite choice, failing closed, and what property of that product forces it.

**Block 4. PURPOSE: reduce the central load by an order of magnitude if the numbers demand it.**

If the interviewer raises traffic to 2 million requests per second, the central design needs 20 or more shards, and that is enough operational weight to reconsider. The lease design fixes it: each gateway node requests a block of 100 tokens for a key and spends them locally.

Central operations drop from one per request to one per 100 requests, so 2 million per second becomes 20,000 per second, which is a single shard again.

The price is bounded overshoot: in the worst case every one of the 50 nodes holds an unspent lease of 100 tokens, so up to 5,000 extra requests could be admitted for a key before the leases expire. Against a limit of 1,000 per minute, that is a real overshoot, so I shrink the lease to 10 tokens, which caps overshoot at 500 and still cuts central load by ten times.

**The error most readers make here** is presenting leases as strictly better. They trade exactness for load, and the trade must be quantified: overshoot equals node count times lease size, and that formula is the sentence to say out loud.

### 6e. Fade the scaffold

**Faded example 1.** Decide whether to put a content delivery network in front of an interface. GIVEN: 8,000 requests per second, responses average 40 kilobytes, 70 percent of responses are identical for all users and change at most daily, the rest are per user. Produce the last block.

- Block 1, bytes: 8,000 times 40 kilobytes is 320 megabytes per second, about 2.6 gigabits per second. That is more than a single machine's assumed network capacity, so serving all of it from origin machines means buying machines for bytes rather than for work.
- Block 2, cacheable share: 70 percent of requests are shareable, so a well designed cache key could take origin traffic to 2,400 requests per second and origin bytes to about 0.8 gigabits per second.
- Block 3: **you decide, and state what would make you not do it.**

??? note "Show answer"

    Do it, but for the bytes, not for the latency. The argument is that 2.6 gigabits per second of largely identical responses is bandwidth you are paying twice for, and moving 70 percent of it to an edge cuts origin egress by roughly two thirds and cuts origin machine count by a similar factor if you were provisioning for bandwidth.

    What would make me not do it: if the shareable fraction were below roughly 20 percent, the hit ratio would be too low to change the origin's provisioning, and I would be adding an invalidation problem for a 20 percent saving. The other blocker is per-user data in a shareable response, which is a correctness risk, so I would insist that the shareable responses carry no user-specific fields at all rather than trying to vary the cache key on identity.

    The measurement I would name first: the cache hit ratio and the origin egress in gigabits, side by side, weekly.

**Faded example 2.** Choose a balancing algorithm. GIVEN: a service with an in-process cache where a hit costs 1 millisecond and a miss costs 40 milliseconds, 24 instances, and a key space where the top 20 keys account for an assumed 30 percent of traffic. Produce the last two blocks.

- Block 1: round robin gives every instance a full copy of the working set, so cache memory is multiplied by 24 and the hit ratio is set by per-instance memory rather than by total memory.

??? note "Show answer"

    Block 2, why consistent hashing is the candidate: hashing on the cache key sends the same key to the same instance, so each instance caches roughly one twenty fourth of the key space and the effective cache size is the sum across instances rather than the smallest of them. With a 40 to 1 miss penalty, hit ratio dominates latency, so this is worth real complexity.

    Block 3, why plain consistent hashing then fails, and the fix: the top 20 keys carry 30 percent of traffic, and hashing sends each of them to exactly one instance. A single instance can receive several of them and be driven far above its share while others idle.

    The fix is a per-instance load cap that overflows excess requests for a hot key to the next instance in the ring, which keeps most of the cache locality and bounds the imbalance. The alternative fix, which is cheaper and often sufficient, is to replicate only the known hot keys onto every instance and hash everything else, so 20 keys are duplicated and the rest stay partitioned.

    The measurement that decides between them: per-instance request rate at the ninety ninth percentile against the mean. If the ratio exceeds about 1.5, the imbalance is real and worth fixing; below that, do nothing.

### 6f. Check yourself

**Q1.** Your deep health check calls the database. The database has a brief incident. What does the balancer do, and what should it have done?

??? note "Show answer"

    Every backend fails its check simultaneously, so the balancer marks the entire pool unhealthy and has nowhere to send traffic. A partial database problem becomes a total outage, and recovery is delayed because the pool stays out even after the database returns, until checks pass again.

    The correct behaviour is a fail-open threshold: when more than an assumed half of the pool is unhealthy at once, disregard the checks and keep routing, because a correlated failure of every backend is far more likely to be a broken check or a shared dependency than genuinely dead servers. Also separate the readiness signal, which controls traffic, from the dependency health signal, which should page a human rather than remove capacity.

**Q2.** Why does putting a rate limiter in the gateway not protect a downstream service from overload?

??? note "Show answer"

    Because a per-key limit constrains individuals, not aggregates. Ten thousand keys each perfectly inside their limit can still exceed what the service behind them can serve. Rate limiting is a fairness and abuse mechanism. Overload protection is a separate mechanism: concurrency limits, queue depth limits with rejection, and load shedding that drops the cheapest requests first. Saying that these are two different problems with two different mechanisms is one of the higher-value distinctions available in this round.

### 6g. When not to use this

**Content delivery network.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Origin egress in gigabits per second, and the fraction of responses that are byte-identical across users |
| Threshold below which it is over-engineering | Under roughly 1 terabyte of egress per month, or a shareable fraction below about 20 percent |
| Cheaper alternative | Correct cache headers plus versioned immutable asset paths, which many hosts already serve efficiently |

**Dedicated rate limiting service.**

| Question | Answer |
|---|---|
| Measurement that justifies it | A recorded incident caused by one caller, or a published contract with quotas |
| Threshold below which it is over-engineering | A single internal service with trusted callers and no published quota |
| Cheaper alternative | A maximum concurrency setting on the server and a connection limit at the proxy |

**API gateway as a separate tier.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Three or more services duplicating the same authentication and limiting logic |
| Threshold below which it is over-engineering | One or two services. The gateway becomes an extra hop and a shared failure domain for no gain |
| Cheaper alternative | A shared library, or the reverse proxy you already run |

**Global steering across regions.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Measured user latency by region showing a gap that matters to the product, or a residency requirement |
| Threshold below which it is over-engineering | A single region product with users concentrated in one geography |
| Cheaper alternative | One region plus an edge cache, which recovers most of the latency for static and cacheable content at a fraction of the complexity |

---

## Module 7. Building blocks II, storage

**Time: 70 to 90 minutes reading, 35 to 50 minutes practice.**

### 7a. Predict first

```
table  orders
  order id        primary key
  customer id     index 1
  status          index 2
  created at      index 3
  merchant id     index 4

write rate 2000 inserts per second
each order row is about 300 bytes

Figure a table with four secondary indexes and its insert rate
```

??? note "Show answer"

    Prediction asked for: what does one insert actually cost, and what is the real write rate on storage?

    One logical insert is five physical writes: the row plus one entry in each of the four indexes. At 2,000 inserts per second that is 10,000 index updates per second on top of the row writes.

    Then the storage engine adds its own multiplier. On a page-oriented engine, each of those five writes dirties a page, and a page is far larger than the entry, so a 30 byte index entry can cost an assumed 8 kilobyte page write, plus the write-ahead log entry. On a log-structured engine, the writes are sequential and cheap on arrival, but compaction rewrites the same data several times later.

    The design conclusion: **an index is a write cost you pay on every insert to make one query fast.** Four indexes is four taxes. In a round, when you add an index, say which query it serves and what it costs the write path, or an interviewer will assume you do not know it costs anything.

### Replication, three topologies and the arithmetic of each

| Topology | How writes happen | Wins when | Characteristic failure |
|---|---|---|---|
| Single leader | All writes go to one node, replicas follow | Almost always, at almost every scale | Failover window, and stale reads from replicas |
| Multi leader | Writes accepted in several places | Multi region write locality, or offline clients | Conflicts, which you must resolve explicitly |
| Leaderless quorum | Clients write to several nodes, read from several | High write availability, tolerant of node loss | Anomalies that need repair, and tuning that looks simple and is not |

**Synchronous, asynchronous and the middle.**

| Mode | Data loss on leader failure | Write latency cost |
|---|---|---|
| Asynchronous | Everything not yet replicated, equal to the current lag | None |
| Synchronous to all replicas | None | Slowest replica sets your latency, and one slow replica stalls all writes |
| Synchronous to at least one, asynchronous to the rest | None, if the synchronous replica survives | One extra round trip, and the fastest replica sets the pace |

The third row is the default worth defending in a round. It buys a recovery point objective of zero for single node failure at the cost of one round trip, and it avoids the availability trap where any slow replica blocks every write.

**Quorum arithmetic.** With N replicas, W nodes acknowledging a write and R nodes consulted on a read, overlapping quorums require W plus R to be greater than N.

| N | W | R | Property | Tolerates |
|---|---|---|---|---|
| 3 | 2 | 2 | Overlap guaranteed | 1 node down for both reads and writes |
| 3 | 3 | 1 | Fast reads | 0 nodes down for writes |
| 3 | 1 | 3 | Fast writes | 0 nodes down for reads |
| 5 | 3 | 3 | Overlap guaranteed | 2 nodes down |

The caveat interviewers probe: overlap guarantees that a read touches a node that saw the write, not that the read returns the newest value, unless the client resolves versions. Concurrent writes still need a conflict rule. The design that popularised this configuration space, along with virtual nodes and version vectors, is DeCandia and others, "Dynamo: Amazon's Highly Available Key-value Store", SOSP 2007 (<https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf>).

**Replication lag, as a number you can compute.** If a replica falls behind during an incident, catch-up time is backlog divided by the surplus apply rate.

Worked: writes arrive at 2,000 per second, a replica can apply 5,000 per second, and it was disconnected for 10 minutes. Backlog is 2,000 times 600, which is 1.2 million writes. Surplus is 5,000 minus 2,000, which is 3,000 per second. Catch-up is 1.2 million divided by 3,000, which is 400 seconds.

If instead the apply rate were 2,500 per second, catch-up would be 2,400 seconds, six times longer for a replica that is only twice as slow. That non-linearity is why replica headroom matters more than replica count.

### Partitioning, and the choice that is hard to reverse

| Strategy | Good for | Bad for |
|---|---|---|
| Hash of a key | Even spread, point lookups | Range scans, which must touch every partition |
| Range of a key | Range scans and time windows | Hot spots, since recent data is one partition |
| By tenant or entity group | Isolation, simple reasoning, cheap transactions inside a group | Very uneven tenants, and cross-tenant queries |

**Choosing the partition key is the highest leverage decision in the storage module.** State the rule: the partition key should be the value present in the overwhelming majority of your queries. If your dominant query is "the last 50 messages in conversation X", the partition key is the conversation, and the sort key is the sequence number, so the answer is one contiguous read.

**Secondary indexes across partitions, the two shapes.**

| Shape | Where the index lives | Read cost | Write cost |
|---|---|---|---|
| Local, partitioned by the same key as the data | Beside the data | Must query every partition and merge, so cost grows with partition count | One local write |
| Global, partitioned by the indexed value | Somewhere else | One partition | A second write in a different partition, which is a distributed write with its own failure modes |

The interview sentence: a global secondary index converts every insert into a multi-partition write, so either you accept that the index is eventually consistent with the row, or you buy a transaction. Most systems choose the former and most candidates never say it.

**Rebalancing.** Never partition by hash modulo the node count. Adding one node to N changes the destination of almost every key. Two workable options:

| Approach | Movement when a node is added | Complexity |
|---|---|---|
| Fixed large partition count, assigned to nodes | Whole partitions move, roughly one Nth of the data | Low, and it is the common default |
| Consistent hashing with virtual nodes | Roughly one Nth of the keys move | Moderate, and it also gives locality benefits |

**Hot keys.** Partitioning spreads keys, not traffic. One key can exceed one partition's capacity by itself.

| Remedy | How it works | Price |
|---|---|---|
| Read replica of the hot partition | Extra copies serve reads | Only helps reads, and adds staleness |
| Key splitting, appending a suffix | One logical key becomes k physical keys | Reads must gather k parts, and writers must pick a suffix |
| Request coalescing at the caller | Concurrent identical reads share one fetch | Only helps a burst of identical reads |
| Short lived local cache | Absorbs repeat reads near the client | Staleness bounded by the lifetime you choose |

Detection first: you cannot fix a hot key you cannot see, so per-key traffic sampling or a top-k sketch belongs in the design, not in the postmortem.

### Key-value internals, and the amplification that decides the choice

Two families. Define both before comparing.

- **Page-oriented tree.** Data lives in fixed-size pages arranged as a balanced tree. Updates modify pages in place, protected by a write-ahead log.
- **Log-structured merge tree.** Writes go to an in-memory table and a sequential log. When the memory table fills, it is written out as a sorted immutable file. Background compaction merges files into larger ones. This structure is described in Chang and others, "Bigtable: A Distributed Storage System for Structured Data", USENIX OSDI 2006 (<https://www.usenix.org/legacy/event/osdi06/tech/chang.html>).

| Property | Page-oriented tree | Log-structured merge tree |
|---|---|---|
| Write path | Random page updates plus a log | Sequential appends |
| Write amplification | Page size divided by row size, plus the log. An assumed 8 kilobyte page for a 200 byte row is about 40 times, plus roughly 2 times for the log | Roughly the level fanout times the number of levels. At a fanout of 10 and 4 levels, about 40 times, paid later during compaction |
| Read path for a point lookup | Tree height random reads, typically 3 or 4 | One check per level, cut down by per-file summaries |
| Space amplification | Low, but fragmentation exists | Higher, since obsolete versions live until compaction |
| Latency profile | Predictable | Spiky, because compaction competes with foreground work |
| Range scans | Excellent, the tree is ordered | Good, but a merge across files is required |

**Per-file summaries are what make the log-structured read path viable.** A probabilistic filter answers "this file definitely does not contain this key" using an assumed 10 bits per key for roughly a 1 percent false positive rate. For 1 billion keys that is 10 billion bits, about 1.25 gigabytes of memory. That number is worth knowing, because it tells you when the filters stop fitting in memory, and when they stop fitting, point lookups start touching every level.

**The honest interview answer.** At the point rates most prompts describe, both families work. The deciding factors are the read pattern, whether the workload is update-heavy or append-heavy, and what your team can operate under pressure. Say that instead of reciting a preference, then name the one number that would change your mind: sustained write throughput per node beyond what random page updates can absorb.

### Blob storage, and why the database is the wrong place

| Where the bytes live | Consequence |
|---|---|
| In a database row | Backups, replication and cache all now carry bytes that never get queried, and the row cache hit ratio collapses |
| In an object store, with a key in the row | The database stays small and fast, and bytes are served by something built for bytes |

The failure mode that made this a research topic is small-file metadata cost: with billions of small objects, the metadata lookups needed to find a file can dominate the read itself. The response of packing many small objects into large append-only volumes with a compact in-memory index is described in Beaver and others, "Finding a Needle in Haystack: Facebook's Photo Storage", USENIX OSDI 2010 (<https://www.usenix.org/legacy/event/osdi10/tech/full_papers/Beaver.pdf>).

### Partitions and replicas, drawn

```
        key space split into partitions

 +--------+  +--------+  +--------+  +--------+
 |  p 0   |  |  p 1   |  |  p 2   |  |  p 3   |
 +--------+  +--------+  +--------+  +--------+

        each partition has one leader and two followers

 +---------------+  +---------------+  +---------------+
 |   node A      |  |   node B      |  |   node C      |
 | p0 leader     |  | p1 leader     |  | p2 leader     |
 | p1 follower   |  | p2 follower   |  | p3 follower   |
 | p3 follower   |  | p0 follower   |  | p1 follower   |
 +---------------+  +---------------+  +---------------+

 losing node A promotes p0 on the node holding its follower
 adding node D moves whole partitions not individual keys

Figure partitions spread across nodes with leaders and followers
```

### Worked example: choosing storage for the link shortener

Carried forward from module 4: 200 writes per second, 20,000 reads per second, 15 terabytes over 5 years, point lookups by short code, a 7.5 gigabyte hot set, and records that are written once and essentially never updated.

**Block 1. PURPOSE: let the access pattern pick the shape before any product name is mentioned.**

Every hot-path read is a point lookup by short code. There are no range scans, no joins and no multi-row transactions on the hot path. That shape is a key-value lookup, so the primary key is the code and the partition key is the code, hashed.

I explicitly refuse to add a secondary index on owner to the hot store. The owner-facing dashboard is a different access pattern with a different latency budget, so it gets its own read model fed asynchronously. Adding a global secondary index would make every creation a two-partition write to serve a query that runs a few times per user per month.

!!! note "Say it before you read on"

    Say what the owner dashboard costs if you serve it by scanning the main store instead, and why that is still acceptable at these numbers.

**The error most readers make here** is designing one store that serves every query. Two access patterns with different latency budgets and different volumes are two stores, and saying so is a senior move, not an extravagance.

**Block 2. PURPOSE: size the nodes using the recovery window, not the capacity sheet.**

15 terabytes fits on a small number of machines, so raw capacity does not force a node count. The operational constraint does. Assume restore runs at 200 megabytes per second, which is a figure I would verify. One hour of restore time is 720 gigabytes. If I want a node restorable in about 3 hours, nodes hold about 2 terabytes, so 15 terabytes needs 8 partitions of primary data.

With three copies of each partition, that is 45 terabytes of raw storage. I state the recovery time objective the choice implies: losing one node costs a promotion in seconds and a rebuild measured in hours, and the rebuild is the number that matters because a second failure during the rebuild is the scenario that loses data.

**Block 3. PURPOSE: choose the engine family using the workload's actual shape.**

The workload is append-only with no updates and no deletes until expiry. That is close to the ideal case for a log-structured engine: compaction has almost nothing to reclaim, so the usual write amplification argument barely applies, and point lookups are one file check per level with per-file filters in memory.

Filter memory check: 30 billion records over 5 years would be a problem, but this dataset is 30 billion records only if you keep every link forever at this rate. Recompute: 500 million per month times 60 months is 30 billion records, so at an assumed 10 bits per key the filters alone are 37.5 gigabytes spread over 8 nodes, about 4.7 gigabytes per node. That fits, but it is not free, and it is the number I would monitor.

**Block 4. PURPOSE: state the caching layer as a consequence of the earlier numbers rather than as a habit.**

From module 4, the hot set is 7.5 gigabytes. That fits on one cache node, so I use a small replicated cache in front of the store rather than a sharded tier, and I set the entry lifetime to an assumed 24 hours because records are immutable, which makes invalidation a non-problem.

The one invalidation case is deletion or a takedown, which is rare and can afford an explicit purge. I name the metric that governs the tier: cache hit ratio. Below about 80 percent the skew assumption from module 4 was wrong, and the correct response is to re-derive the hot set rather than to add nodes.

!!! note "Say it before you read on"

    Say which single new requirement would invalidate the whole of this storage design, and why.

### 7e. Fade the scaffold

**Faded example 1.** Choose the storage layout for a user timeline read as "the newest 30 items for user X, then page backwards". GIVEN: 200 million users, an assumed average of 500 timeline items per user, an item reference of 40 bytes, and 40,000 timeline reads per second. Produce the last block.

- Block 1, shape: the query is a range scan inside one user, newest first, so partition by user and sort by descending sequence.
- Block 2, size: 200 million times 500 times 40 bytes is 4 terabytes of references, which is small enough that references, not full items, is clearly the right unit to store.
- Block 3: **you decide the engine family and the caching, and name what breaks.**

??? note "Show answer"

    Engine family: a range scan inside a partition suits an ordered store, and both families do this well, so the deciding factor is the write pattern. Timelines are append-heavy with occasional deletes when a source item is removed, which favours a log-structured store, with the caveat that deletions become tombstones that a range scan must skip.

    If an assumed 1 percent of items are deleted, a 30 item page may need to read 31 or 32 entries, which is fine. If a user's sources are mostly deleted accounts, the same scan reads thousands of tombstones, which is the failure to name.

    Caching: the newest page is what almost every read wants, so cache the first page per active user. If an assumed 20 million users are active daily and a page of 30 references is 1.2 kilobytes, that is 24 gigabytes, which is a small sharded tier rather than a single node.

    What breaks: a user who follows a very high volume source floods their timeline, so the write cost per source item scales with follower count. That is the push versus pull decision, and it belongs in the movement module rather than here.

**Faded example 2.** Choose storage for an inventory service. GIVEN: 5 million distinct items, 3,000 stock decrements per second at peak, and a single flash-sale item that receives an assumed 40 percent of all decrements during a sale. Produce the last two blocks.

- Block 1, shape: the operation is a conditional decrement on one key, which must be linearizable per item, so the partition key is the item and every decrement must be routed to that partition's leader.

??? note "Show answer"

    Block 2, the hot key arithmetic: 40 percent of 3,000 is 1,200 conditional writes per second against a single key on a single leader. Serialised updates on one row also serialise their commits, so this is a queue on one lock, and the ninety ninth percentile latency will be set by contention rather than by disk. This is the number that eliminates the plain single-row design.

    Block 3, the fix and its price: split the hot item's stock into k sub-counters, each holding an equal share, and route a decrement to a randomly chosen sub-counter. At k equal to 20, each sub-counter takes about 60 writes per second, which is ordinary.

    The price is that stock is now approximate at the boundary: a sub-counter can be empty while others have stock, so a customer can be told the item is out of stock while 15 units remain elsewhere. The fix for that is a fallback that retries against another sub-counter on failure, which costs an extra round trip only in the endgame of a sale.

    The measurement that justifies the split: per-key write rate at peak. Below roughly 100 conditional writes per second on one key, do not split, because you would be trading an exact counter for an approximate one to solve a problem you do not have.

### 7f. Check yourself

**Q1.** A candidate proposes a global secondary index so a rare admin query is fast. What do you say?

??? note "Show answer"

    Name the cost in write-path terms: every insert becomes a write in two partitions, so the write path gains a second failure mode and either becomes eventually consistent with the row or requires a cross-partition transaction. Then compare frequencies: the index is paid on every write and used a few times a day.

    The cheaper answer is an asynchronously maintained read model, or accepting a slow scan for a query nobody waits on. Trading a permanent write-path tax for a rare read is the exact shape of decision interviewers want you to refuse.

**Q2.** Your replicas are asynchronous and a user reports that their own edit disappeared after saving. What happened, and what is the smallest fix?

??? note "Show answer"

    The write went to the leader and the subsequent read was served by a replica that had not yet applied it, so the user failed to read their own write. The smallest fix is not stronger consistency everywhere.

    It is read-your-writes for the acting user only: after a write, route that user's reads to the leader for a short window, or carry the write position in the session and require any replica serving them to have reached it. That is a session-scoped guarantee costing one token, rather than a system-wide guarantee costing a round trip on every read.

### 7g. When not to use this

**Sharding a database.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Working set no longer fits in memory and p99 read latency has degraded, or write throughput is at the ceiling of one primary, or one node's dataset makes the restore window unacceptable |
| Threshold below which it is over-engineering | Under roughly 1 terabyte per node and a few thousand writes per second. Sharding costs you cross-shard queries, cross-shard transactions, rebalancing and a much harder operational story |
| Cheaper alternative | A larger machine, read replicas for read scaling, archiving cold rows to another table or store, and removing an index |

**Read replicas.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Read load is the bottleneck and the workload tolerates staleness measured in the current replication lag |
| Threshold below which it is over-engineering | When the primary is under an assumed 50 percent utilisation, or when every read must be current anyway |
| Cheaper alternative | A cache with an explicit lifetime, which gives more relief per unit of effort and makes the staleness bound visible instead of implicit |

**Leaderless quorum store.**

| Question | Answer |
|---|---|
| Measurement that justifies it | A requirement for writes to survive the loss of any node or zone without a failover pause, on a workload whose conflicts have an obvious resolution rule |
| Threshold below which it is over-engineering | Any workload with read-modify-write operations, counters or uniqueness constraints. You will rebuild coordination on top and get a worse version of it |
| Cheaper alternative | Single leader with semi-synchronous replication and automated failover, which covers the vast majority of prompts |

**A separate object store for files.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Byte volume that distorts database backups, replication or cache hit ratio, which usually shows up above a few tens of gigabytes of blobs |
| Threshold below which it is over-engineering | A few gigabytes of small attachments in a product with one deployment. The extra service, credentials and lifecycle rules cost more than they save |
| Cheaper alternative | Store the bytes in the database, and revisit when backup duration becomes the complaint |
---

## Module 8. Building blocks III, movement

**Time: 65 to 85 minutes reading, 30 to 45 minutes practice.**

### 8a. Predict first

```
producers  20000 events per second steady

+--------------------------------------------+
|  log topic   24 partitions                 |
+--------------------------------------------+
        |            |             |
        v            v             v
   +---------+  +---------+   +---------+
   | consumer|  | consumer|   | consumer|
   | group   |  | group   |   | group   |
   +---------+  +---------+   +---------+

one consumer instance processes 500 events per second
the team plans to run 60 instances

Figure a log topic with 24 partitions and a plan to run 60 consumers
```

??? note "Show answer"

    Prediction asked for: what throughput does this actually achieve, and what happens over the next hour?

    In a partitioned log, a partition is assigned to at most one consumer in a group, so parallelism is capped by partition count. 24 partitions times 500 events per second is 12,000 per second. The other 36 instances sit idle, holding no partition and consuming budget.

    Arrival is 20,000 per second, so lag grows at 8,000 per second. Over one hour that is 8,000 times 3,600, which is 28.8 million events of backlog. If each event is an assumed 300 bytes, that is about 8.6 gigabytes of retained backlog, which is fine for disk and fatal for an in-memory queue.

    The fix is more partitions, not more consumers. Raising to 60 partitions gives 30,000 per second of capacity, a surplus of 10,000 per second, so the 28.8 million backlog clears in 2,880 seconds, about 48 minutes. Note the asymmetry worth saying out loud in a round: **the backlog took one hour to build and takes forty eight minutes to clear even after the fix.** Recovery is never instant, and repartitioning a live topic is itself an operation with an ordering cost.

### Queue or log: they are not the same thing

| | Queue | Partitioned log |
|---|---|---|
| Unit of progress | Per message acknowledgement | An offset per partition per consumer group |
| After consumption | Message is removed | Record stays until retention expires |
| Replay | Not possible once acknowledged | Rewind the offset and reprocess |
| Ordering | Usually none, or per group with extra work | Total order within a partition |
| Parallelism | Add consumers freely | Capped by partition count |
| Multiple independent readers | Needs a copy per consumer | Natural: each group has its own offset |
| Best at | Work distribution with uneven task cost | Event distribution, replay, several downstream users |

Choose a queue when the message is a task to be done once by someone. Choose a log when the message is a fact that several systems care about and someone will want to reprocess it. Saying which of those two the payload is, before naming any technology, is the move that separates a designed answer from a named one.

### Delivery semantics, stated precisely

| Semantic | What the broker does | What the application must do |
|---|---|---|
| At most once | Acknowledge before processing | Accept loss |
| At least once | Acknowledge after processing | Make processing idempotent, because duplicates are certain |
| Effectively once | At least once, plus deduplication or transactional commits | Own a deduplication key or a transactional sink |

**Where exactly-once is a lie.** A broker can make delivery and offset commitment atomic inside its own system. Apache Kafka's transactional producer and consumer offsets do this, as described in Confluent's write-up of the design (<https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/>, 2017). What no broker can do is make an external side effect atomic with a message acknowledgement. If your handler charges a card, sends an email or calls a partner, then a crash between the side effect and the acknowledgement is indistinguishable from a crash before the side effect.

The honest formulation to say in a round: **at-least-once delivery plus an idempotent effect equals exactly-once outcomes, and that is the only kind you can actually build.**

**Sizing a deduplication store, because "we will deduplicate" is not a design.** Deduplication needs a key, a window and memory.

Worked: 500 transactional messages per second, a 24 hour window, and an assumed 32 bytes per key plus 32 bytes of entry overhead. Keys in the window are 500 times 86,400, which is 43.2 million. At 64 bytes each that is about 2.8 gigabytes, which fits on one node.

Now change the input to 20,000 messages per second: 1.73 billion keys, about 110 gigabytes, which does not. At that point you either shorten the window to one hour, which cuts it to 4.6 gigabytes, or you move deduplication into the sink by making the write itself conditional on the key.

That derivation is the answer. The window length is a design parameter you choose from a memory budget, not a detail.

### Ordering, and the requirement people over-state

Global ordering across a distributed stream costs a single serialisation point, which is a throughput ceiling. Almost no product needs it.

| Requirement as stated | What is usually meant | Mechanism |
|---|---|---|
| Messages must be in order | Messages about the same entity must be in order | Partition by entity id, order within partition |
| The whole system must agree on order | Causally related events must not invert | Causal metadata, or a single partition per causal chain |
| Nothing may be processed out of order | Handlers must be commutative or the effect must be idempotent | Version numbers on the record, and last-writer-wins by version |

**The cost of ordering by entity is a hot partition.** If one entity produces a disproportionate share of events, its partition is a single consumer's problem. The remedy is the same as the hot key remedy in the storage module, and the trade is identical: split the key and give up strict ordering for that entity, or keep ordering and accept the ceiling.

### Backpressure, and the failure it prevents

**Little's Law.** For a stable system, the number of items in flight equals arrival rate times the average time each spends in the system. Written as L equals lambda times W.

Use it in two directions.

- Forward: at 20,000 arrivals per second and 50 milliseconds of processing, in-flight work is 20,000 times 0.05, which is 1,000 items. That is your concurrency requirement.
- Backward: if the latency budget for time spent in the queue is 2 seconds and arrivals are 20,000 per second, the maximum tolerable queue depth is 40,000. **A queue longer than that is not a buffer, it is a latency violation you have not noticed yet.**

**The three responses to overload, and when each is right.**

| Response | Behaviour | Right when |
|---|---|---|
| Block the producer | Producer slows to consumer speed | Producers are internal and can wait |
| Reject with a retry signal | Producer is told to back off | Producers are external and can be told to try later |
| Drop the lowest value work | Some work is discarded deliberately | The work is sheddable, such as telemetry or non-critical notifications |

The response that is never right is an unbounded buffer. It converts a throughput problem into a memory exhaustion problem and delays the failure until it is unrecoverable.

**Retry storms and metastable failure.** When a system slows, clients retry, which raises load, which slows it further. The system can stay stuck in the degraded state even after the original trigger is removed, because retries alone now sustain the overload. This class is named and analysed in Bronson and others, "Metastable Failures in Distributed Systems", HotOS 2021 (<https://sigops.org/s/conferences/hotos/2021/papers/hotos21-s11.pdf>).

Three mechanisms, all cheap, all expected in a senior answer.

| Mechanism | What it does |
|---|---|
| Exponential backoff with randomisation | Spreads retries in time instead of synchronising them, as described in Amazon's write-up on backoff and jitter (<https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/>, 2015) |
| Retry budget as a fraction of traffic | Caps total retry load, so retries can never become the majority of requests |
| Circuit breaking | Stops calling a dependency that is failing, giving it room to recover |

**Dead letter handling.** A message that fails repeatedly must leave the main path, or it blocks a partition forever. State three things in the design: the attempt count that triggers diversion, where diverted messages go, and who looks at them. A dead letter queue nobody reads is a silent data loss mechanism with extra steps.

### Event time, watermarks and the completeness knob

Two clocks exist. Processing time is when your system saw the event. Event time is when it happened. They diverge because of mobile clients, retries, network delays and offline buffering.

A watermark is an assertion: "I believe I have now seen all events with an event time before T." Everything downstream of that assertion is a trade.

| Choice | Effect on completeness | Effect on latency |
|---|---|---|
| Advance the watermark aggressively | More events arrive late and are excluded or corrected | Results published sooner |
| Hold the watermark back | Fewer late events | Every result waits for the slowest producer |
| Allow lateness, then emit corrections | Complete eventually | Downstream consumers must handle restatements |

**The design question is not "how do I handle late data", it is "who consumes the correction".** If the downstream is a dashboard, corrections are fine. If the downstream is a billing invoice already sent to a customer, they are not, and the design must close the window with a hard cutoff and a documented policy for what happens to events arriving afterwards.

**Windowing types, in one line each.**

| Window | Shape | Typical use |
|---|---|---|
| Tumbling | Fixed size, non overlapping | Per minute counts |
| Sliding | Fixed size, overlapping | Moving averages and rate alerts |
| Session | Gap defined, variable length | User activity sessions |

### Movement, drawn

```
+----------+     +---------------------------+
| producer |---->| durable log               |
+----------+     | partition by entity id    |
                 +---------------------------+
                    |          |          |
                    v          v          v
              +---------+ +---------+ +---------+
              |consumer | |consumer | |consumer |
              +---------+ +---------+ +---------+
                    |          |          |
                    +----------+----------+
                               |
                     +---------+---------+
                     |                   |
                     v                   v
              +-------------+     +-------------+
              | sink with   |     | dead letter |
              | idempotent  |     | after n     |
              | write       |     | attempts    |
              +-------------+     +-------------+

Figure a log with per entity partitioning an idempotent sink and a
dead letter path
```

### Worked example: the notification campaign, sized properly

Carried forward from module 3. GIVEN: 50 million notifications per day steady, about 500 per second; one marketing campaign of 20 million messages that the product wants delivered in 10 minutes; transactional notifications must arrive within 30 seconds; duplicates are unacceptable for transactional and tolerable for marketing.

ASSUMED, and stated as such: the push provider accepts 5,000 messages per second for this account; each send is about 40 milliseconds of mostly waiting on the network; each message is about 200 bytes on the wire.

**Block 1. PURPOSE: test the stated requirement against the dependency before designing anything.**

The campaign wants 20 million messages in 600 seconds, which is 33,333 per second. The provider accepts 5,000 per second. At that rate 20 million messages take 4,000 seconds, which is about 67 minutes.

So the requirement is not achievable, and no amount of internal architecture changes that. I say this immediately, because it is the single most valuable sentence available in this problem: "the 10 minute target requires roughly seven times the provider throughput we have, so before I design anything I want to know whether we can buy more provider capacity, spread across several sender accounts, or relax the target."

**The error most readers make here** is designing a beautifully scalable internal pipeline that ends at a dependency doing 5,000 per second. Always compute the slowest link first.

!!! note "Say it before you read on"

    Say what you would do if the interviewer replies "assume we cannot change the provider or the target".

**Block 2. PURPOSE: size the buffer from the mismatch, which decides the buffer's technology.**

Suppose the campaign is enqueued as fast as we can generate it and drained at 5,000 per second. Peak backlog is roughly 20 million minus what drains during generation. If generation takes 600 seconds, drain removes 3 million in that time, so the peak backlog is about 17 million messages.

At 200 bytes each, that is 3.4 gigabytes. Eliminated by that number: an in-memory queue, and any broker configuration that keeps the backlog in memory. Required: a durable, disk-backed log with retention comfortably above the drain time, which here needs to cover at least 67 minutes and which I would set to hours for safety.

**Block 3. PURPOSE: prove that one shared pipe violates the transactional requirement, using arithmetic rather than intuition.**

If transactional and marketing messages share one first-in-first-out pipe, a transactional message enqueued at the peak sits behind 17 million marketing messages. At a drain rate of 5,000 per second, its wait is 17 million divided by 5,000, which is 3,400 seconds, about 57 minutes. The requirement was 30 seconds. The violation is a factor of over 100.

Therefore the two classes get separate topics and separate consumer groups with separate provider quota, not a priority field on one queue. I say why explicitly: a priority field inside one partitioned log does not help, because a consumer reads its partition in order and cannot skip ahead to a high priority message sitting behind a million low priority ones.

!!! note "Say it before you read on"

    Say what would change if the provider allowed us to reserve a guaranteed 500 per second for transactional traffic within the same account.

**Block 4. PURPOSE: make duplicates impossible where they are unacceptable, and size the machinery that does it.**

Transactional messages carry a deduplication key equal to the triggering event identifier. Before sending, the worker performs a conditional insert of that key with a 24 hour lifetime, and only sends if the insert succeeded. Sizing, from the section above: 500 per second times 86,400 seconds is 43.2 million keys, and at an assumed 64 bytes per entry that is about 2.8 gigabytes, which fits on a single node with replication.

Marketing messages get no deduplication store, because the volume would be 40 times larger for a class where duplicates are tolerated. Choosing different guarantees for different classes of the same message type, and justifying it with a memory figure, is the depth this problem rewards.

**Block 5. PURPOSE: state the overload policy and the failure handling before being asked.**

| Situation | Policy | Why |
|---|---|---|
| Provider returns rate limit errors | Back off with randomised delay and reduce concurrency, do not retry immediately | Synchronised retries are how a rate limit becomes an outage |
| Provider times out with no answer | Treat as unknown, do not resend transactional without checking the deduplication key | The key makes an ambiguous send safe to retry |
| Backlog exceeds an assumed 2 hours of drain time | Stop accepting new campaigns and alert | A queue that can never drain is a failed system that looks healthy |
| A message fails 5 times | Divert to dead letter storage with the failure reason, and page nobody, but report daily | Poison messages must not block a partition |

**The error most readers make here** is treating the dead letter path as a formality. State the trigger count, the destination and the human process, or it is a silent loss channel.

### 8e. Fade the scaffold

**Faded example 1.** An event pipeline counts video views per minute. GIVEN: 40,000 events per second at peak, each event 250 bytes, a consumer instance handles an assumed 2,000 events per second, and the current topic has 16 partitions. Produce the last block.

- Block 1, capacity: 16 partitions times 2,000 is 32,000 per second, against 40,000 arriving. Lag grows at 8,000 per second.
- Block 2, the backlog if it runs for 30 minutes: 8,000 times 1,800 is 14.4 million events, about 3.6 gigabytes at 250 bytes each.
- Block 3: **you fix it, and state the recovery time and the risk of the fix.**

??? note "Show answer"

    The fix is partitions, not instances. Going to 32 partitions gives 64,000 per second of capacity, a surplus of 24,000 per second, so 14.4 million clears in 600 seconds, about 10 minutes.

    The risk of the fix is the part most candidates miss. Increasing partition count changes which partition a key hashes to, so events for the same video can appear in two partitions during and after the change. Any per-key ordering guarantee is broken across that boundary, and a per-key aggregation that assumed one partition per key can now double count or split counts.

    The safe versions are to over-provision partitions at creation time, or to run a new topic alongside the old one and cut over consumers once the old topic is drained.

    The design lesson to state out loud: partition count is closer to a schema decision than to a scaling knob, so choose it for the traffic you expect in a year, not the traffic you have.

**Faded example 2.** Ad click accounting. GIVEN: 30,000 clicks per second, each click must be billed exactly once, clients can be offline so events arrive with event times up to 12 hours old, and an invoice is generated daily. Produce the last two blocks.

- Block 1, the semantic: billing means the effect must happen once, so the design is at-least-once delivery plus an idempotent write keyed on the click identifier, not a broker feature.

??? note "Show answer"

    Block 2, the deduplication budget: 30,000 per second over a 12 hour window is 30,000 times 43,200, which is 1.3 billion keys. At an assumed 64 bytes per entry that is about 83 gigabytes, which is a sharded store, not a single node.

    The cheaper alternative is to push deduplication into the sink: make the aggregate store accept a conditional insert of the click identifier as part of the same write that increments the counter, so the deduplication state and the counter share a partition and a transaction, and the separate store disappears.

    Block 3, the watermark and the invoice boundary: a 12 hour lateness window against a daily invoice means the invoice cannot be generated at midnight for the day just ended, because up to 12 hours of that day's clicks may not have arrived.

    Two defensible policies. Either delay invoicing by 12 hours and publish complete numbers, or invoice on time and carry late clicks into the next period as an explicit adjustment line. Both are correct engineering; only one is correct for a given finance team, and asking which is the point. What is not acceptable is silently dropping late events, because the money is missing and nobody sees it.

    The measurement to name: the fraction of events arriving after the watermark, tracked daily. If it is under an assumed 0.1 percent, the adjustment approach is cheap; if it is several percent, the delayed invoice is the only honest option.

### 8f. Check yourself

**Q1.** A design says "we use a broker with exactly-once delivery, so we do not need idempotency". What is wrong?

??? note "Show answer"

    The broker's guarantee covers its own boundary: reading a record, producing derived records and committing the offset can be made atomic within that system. It cannot cover an effect outside the system, such as a charge, an email or a call to a partner. A crash after the external effect and before the offset commit produces a repeat. The correct statement is that the handler must be idempotent for any external effect, and the broker's transactional feature removes duplicates only for sinks inside its own transactional scope.

**Q2.** Your consumer lag is flat at 4 million events and not growing. Is the system healthy?

??? note "Show answer"

    It is stable but not healthy. Flat lag means throughput now matches arrivals, so nothing is getting worse, but there is a permanent delay equal to backlog divided by drain rate. At an assumed 20,000 per second of drain, 4 million events is 200 seconds of delay, so every consumer of that output is looking at data more than three minutes old.

    If the product promised near real time, this is an ongoing violation that no alert on growth would catch. Alert on lag in seconds, not lag in messages, because seconds is the unit the requirement was written in.

### 8g. When not to use this

**A message broker at all.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Peak to mean arrival ratio above roughly 3, or work that must survive the caller going away, or several independent consumers of the same event |
| Threshold below which it is over-engineering | Under roughly 1,000 jobs per second with a peak to mean ratio near 1, where the work fits inside the request's latency budget |
| Cheaper alternative | A table used as an outbox in the same transaction as the business write, polled by a worker. It gives durability, retries and ordering per row without a new system to operate |

**A stream processing framework.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Windowed aggregation over event time with lateness, at a volume where a periodic batch job cannot finish inside its own interval |
| Threshold below which it is over-engineering | Anything a five minute batch query can compute. Streaming buys freshness and costs you state management, restarts and a much harder debugging story |
| Cheaper alternative | A scheduled query over the raw events, with the schedule set to the freshness the product actually needs |

**Exactly-once machinery.**

| Question | Answer |
|---|---|
| Measurement that justifies it | An effect with a cost per duplicate: money moved, a message sent to a person, an item shipped |
| Threshold below which it is over-engineering | Counters and metrics where a fraction of a percent of double counting is invisible, and telemetry that is already sampled |
| Cheaper alternative | A natural idempotency key on the write, which is usually already available as an existing identifier |

**Priority lanes and separate topics.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Compute the worst case wait for the urgent class behind the bulk class. If it exceeds the urgent class requirement, you need separation |
| Threshold below which it is over-engineering | When the backlog never exceeds a few seconds of drain time, one pipe is simpler and correct |
| Cheaper alternative | Rate limiting the bulk producer so it cannot build a backlog large enough to delay urgent work |

## Module 9. Building blocks IV: coordination

**Coordination** means getting more than one machine to agree on one fact when
machines die and messages arrive late. It is the most over-reached-for building
block in the interview and the one candidates describe most vaguely.

### 9a. Pre-question

Read this design fragment and commit to an answer before continuing.

```
+--------------------------------------------------------------+
| Three app servers, one Postgres, one Redis.                   |
| Every 60s each server runs the same job sweep:                |
|   1. SET sweeplock <id> NX PX 30000     (Redis lock, 30s TTL) |
|   2. SELECT jobs WHERE due_at < now() AND state = 'pending'   |
|   3. call the email provider for each row                     |
|   4. UPDATE jobs SET state = 'sent'                           |
| Measured: the lock is held by exactly one server 99.9% of     |
| sweeps. Volume: 10,000 due jobs per hour.                     |
+--------------------------------------------------------------+
Caption: a lock-based job sweep across three servers, with its
measured mutual-exclusion rate and its job volume.
```

**Question.** Roughly how many duplicate-eligible emails per day does the 0.1%
figure expose you to, and does adding a second Redis node reduce it?

??? note "Show answer"

    10,000 per hour is 240,000 per day. A 0.1% mutual-exclusion failure rate
    exposes on the order of 240 jobs per day to a double send. Treat that as an
    exposure ceiling, not a prediction: the real count depends on how much of the
    send window overlaps.

    A second Redis node makes it worse. More replicas means more ways for the two
    copies to disagree during a failover, and a lock built on asynchronous
    replication can be handed to two holders.

    The number to attack is not 99.9%.
    It is the fact that a duplicate holder can produce a duplicate side effect at
    all. Put a unique constraint on the send, keyed by job id, and the lock stops
    being a correctness mechanism and becomes what it always was: an efficiency
    mechanism that stops three servers doing the same work.

### 9b. Explanation in small steps

**The one question that routes everything.** Before you name Raft, ask: does
this decision have to survive the death of the machine that made it, and is
there any repair possible afterwards if two machines make it at once?

| Answer | What you need | Typical cost |
|---|---|---|
| No survival needed, repair is cheap | Nothing. Retry and reconcile | Zero |
| Survival needed, repair is possible | Single primary plus idempotency keys plus a reconciler | One table, one cron |
| Survival needed, repair impossible | A consensus group or a database that already runs one | A quorum, a latency floor, an on-call surface |

Most interview prompts land in row two. Candidates answer as if they were all in
row three.

**Terms, defined before use.**

| Term | Definition |
|---|---|
| Quorum | The smallest set of replicas that must acknowledge before a decision counts. For 2f+1 replicas it is f+1 |
| Term (Raft) | A monotonically increasing election epoch. At most one leader per term |
| Commit index | The highest log position a majority has durably stored |
| Lease | A lock with an expiry that the holder must renew, so a dead holder releases without anyone contacting it |
| Fencing token | A monotonic number issued with a lock, checked by the resource, so a stale holder's writes are rejected |
| Clock skew bound | The maximum difference between two nodes' wall clocks that your infrastructure will guarantee. If nobody can name it, you do not have one |
| Idempotency key | A client-generated identifier for one logical intent, reused across every retry of that intent |

**Quorum arithmetic you can do at the whiteboard.** With N replicas, a majority
quorum is floor(N/2)+1, and the group survives N minus quorum failures.

| N | Quorum | Failures tolerated | Availability at per-node 99% |
|---|---|---|---|
| 3 | 2 | 1 | 0.99^3 + 3(0.99^2)(0.01) = 0.99970 |
| 5 | 3 | 2 | previous terms plus 10(0.99^3)(0.01^2) = 0.99999 |
| 7 | 4 | 3 | Gain under one extra nine, latency now waits on the 4th slowest |

Convert the right column to time: 0.99970 leaves 2.6 hours of expected downtime
per year (0.0003 times 8,760), and 0.99999 leaves about 5 minutes. That is the
argument for five nodes and the argument against seven in one table.

**Raft at the depth an interview actually rewards.** You do not need the paper's
proofs. You need these five facts and the consequences.

| Fact | Consequence you should state |
|---|---|
| One leader per term, elected by majority vote | Two leaders cannot both commit, so split brain is contained rather than prevented |
| A write is committed when a majority has it durably | Write latency has a floor of one round trip to the median replica plus one fsync |
| Followers reject log entries that do not match at the previous index | Log divergence is repaired by truncation, not by merge |
| Elections need a randomised timeout longer than a round trip | A network blip longer than the election timeout costs you a leaderless window |
| Reads from the leader are only linearizable with a lease or a quorum round trip | "Read from the leader" is not automatically correct |

Paxos differs mainly in shape: single-decree Paxos agrees on one value and
Multi-Paxos adds a stable leader to skip a phase. Raft folds the leader into the
protocol from the start. For an interview, the honest sentence is that Raft is
easier to reason about and both give you the same guarantee, and the interesting
question is not which one but whether you should be running either.

**The latency floor, derived.** Assume three replicas across three availability
zones in one region with a 1 ms zone-to-zone round trip, and an NVMe fsync of
about 0.5 ms. Both assumptions are conservative and you should measure yours.

- One commit needs the leader's fsync plus one follower's ack: about 1.5 ms.
- A chain of writes that each depend on the previous one therefore tops out near
  1,000 / 1.5, roughly 650 per second.
- Batching 100 entries per commit gives about 65,000 entries per second at the
  same 1.5 ms latency. Throughput comes from batching. Latency does not improve.

Now move the replicas to three regions with a 70 ms round trip. One commit is
about 70 ms, so a dependent chain runs at 14 per second. This single derivation
kills most "just make it globally consistent" answers.

**Leases and the safe action window.** A lease of length L is not safe for L
seconds of work. If your clock skew bound is e and the maximum delay on the
renewal response is d, the holder may only act for L minus e minus d.

- L = 15 s, e = 200 ms, d = 2 s gives a 12.8 s safe window.
- Renew at L/3, so 5 s, which buys two renewal attempts before expiry.

**Fencing tokens close the gap leases leave.** A paused holder that wakes after
expiry still believes it holds the lease. Have the lock service hand out an
increasing integer with every grant, attach it to every write, and have the
storage layer refuse any write whose token is below the highest it has seen.

This is the difference between "the lock is probably right" and "a wrong holder
cannot corrupt anything". Martin Kleppmann's 2016 write-up on distributed
locking is the clearest public treatment of the failure this prevents:
[how to do distributed locking](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html)
(accessed August 2026).

**Clocks.** Three kinds, and only one is safe to order events with.

| Clock | Orders events correctly | Use it for |
|---|---|---|
| Wall clock (NTP) | No, skew is unbounded unless someone guarantees a bound | Human-readable timestamps, TTLs with slack |
| Lamport clock | Yes for causally related events, no total order across concurrent ones | Cheap causality, version counters |
| Hybrid logical clock | Yes, and stays close to wall time | Ordering across nodes when you also want readable timestamps |

A hybrid logical clock stores physical milliseconds in the high bits and a
counter in the low bits, bumping the counter when two events share a
millisecond. It guarantees that if A happened before B then hlc(A) is less than
hlc(B), which a raw wall clock does not. The original description is Kulkarni et
al., *Logical Physical Clocks and Consistent Snapshots in Globally Distributed
Databases*, 2014 ([paper](https://cse.buffalo.edu/tech-reports/2014-04.pdf),
accessed August 2026).

**Skew, quantified.** Suppose your platform team guarantees clocks within plus
or minus 10 ms and your cluster takes 50,000 writes per second. Then 10 ms times
50,000, that is 500 writes, sit inside the ambiguity window at any moment. Any
rule of the form "the later timestamp wins" silently picks a winner among those
500. If your platform team cannot state a bound at all, timestamp ordering is
not a design, it is a coin flip.

**Idempotency keys, the pattern that replaces most coordination.**

| Rule | Why |
|---|---|
| The client generates the key, once per intent, and reuses it on every retry | A server-generated key changes per retry and does nothing |
| Store key, request fingerprint, state, response, created_at, with a unique index on key | The unique index is what actually enforces once-ness |
| Three states: NEW, IN_PROGRESS, DONE | IN_PROGRESS is what makes concurrent retries safe, not just sequential ones |
| Same key, different fingerprint returns 422 | That is a client bug and hiding it costs you a week of debugging later |
| TTL exceeds the longest retry horizon, including a human clicking again | A key that expires before the retry is not an idempotency key |

Sizing, derived: 1,000 writes per second times 86,400 seconds is 86.4 M rows per
day. A 7 day retention gives 605 M rows. At 200 bytes per row that is about
121 GB plus index, which fits comfortably on one node. If your write rate is 20x
that, revisit, because 2.4 TB of hot key lookups is a different design.

Stripe's public API documentation is a good reference for the client-facing
contract this produces:
[idempotent requests](https://docs.stripe.com/api/idempotent_requests)
(accessed August 2026).

### 9c. ASCII diagram

```
       +------------------+        +------------------+
       |  CLIENT          |        |  LOCK SERVICE    |
       |  key = intent id |        |  grants token N  |
       |  retries reuse   |        |  N increases     |
       +---------+--------+        +---------+--------+
                 |                           |
                 v                           v
       +--------------------------------------------------+
       |  SERVICE                                          |
       |  1. insert key (unique index) -> NEW or DONE      |
       |  2. attach token N to every write                 |
       +---------------------+----------------------------+
                             |
                             v
       +--------------------------------------------------+
       |  STORAGE                                          |
       |  rejects any write with token < highest seen      |
       +--------------------------------------------------+

Caption: two independent safety mechanisms. The unique index on the
idempotency key stops a repeated intent; the fencing token stops a
stale lock holder. Neither one substitutes for the other.
```

### 9d. Worked example: charge a card exactly once

**The prompt.** "Users check out. We call an external payment gateway. Make sure
nobody is ever charged twice, and nobody is ever silently not charged."

Inputs I am given or will assume, stated up front:

| Input | Value | Status |
|---|---|---|
| Average checkout rate | 100 per second | GIVEN |
| Peak checkout rate | 300 per second | GIVEN |
| Gateway p99 latency | 4 s | GIVEN |
| Our timeout on the gateway | 10 s | DERIVED, see block C |
| Gateway calls ending in an ambiguous timeout | 0.2% | ASSUMED. Pull the real number from your own logs; the design changes shape at 0.001% and at 1% |

#### Block A. Purpose: decide whether this needs agreement at all

I start by asking whether two concurrent writers could produce something no
repair can fix. They can: two captures against the same card. So this is not
row one of my routing table.

Then I ask whether repair is possible. It is: a refund exists, and the gateway
exposes a lookup by reference. So this is row two, not row three. That single
observation removes leader election, Raft, and any distributed lock from my
answer, and I say so out loud rather than letting the interviewer wonder if I
forgot them.

!!! tip "Self-explanation prompt"

    Before reading Block B, say in one sentence why "repair is possible" is the
    fact that eliminates consensus here, and name one payment-adjacent operation
    where repair is not possible.

**The error most readers make here.** Reaching for a distributed lock on the
user id. It does not help. The lock protects the window between two of your
processes, but the duplicate charge you actually see in production comes from
one process retrying after a timeout, and a lock the same process re-acquires
is no barrier at all.

#### Block B. Purpose: make a client retry cost nothing

The client generates an idempotency key when the user presses Pay, and reuses it
for every retry of that press. Pressing Pay again generates a new key, because
that is a new intent.

Server side, in one transaction:

```
+-------------------------------------------------------------+
| INSERT INTO idem(key, fingerprint, state) VALUES (k, f, NEW) |
|   conflict on key ->                                         |
|      state = DONE        -> return stored response           |
|      state = IN_PROGRESS -> return 409, Retry-After: 2       |
|      fingerprint differs -> return 422                       |
|   no conflict ->                                             |
|      proceed, then write state = DONE with the response      |
+-------------------------------------------------------------+
Caption: the four outcomes of an idempotency key insert, and the
response each one produces.
```

Sizing this table with the numbers above: 100 per second average times 86,400 is
8.64 M rows per day. Payments need a long horizon because a human may retry days
later, so I keep 30 days: 259 M rows at roughly 300 bytes is about 78 GB. That
fits one node with room, so I do not shard it, and I say that explicitly because
restraint is a scored behaviour.

**The error most readers make here.** Letting the server mint the key. Then
every retry looks like a new intent and the table becomes an audit log with no
safety property.

#### Block C. Purpose: survive the ambiguous timeout

This is the block that separates a real answer from a recited one. Our timeout
fires and we do not know whether the gateway charged the card.

I set the timeout at 10 s rather than close to the 4 s p99, because a timeout
below the gateway's own tail converts slow-but-successful calls into ambiguous
ones, and ambiguity is the expensive state. At 0.2% ambiguity and 8.64 M calls
per day, that is 17,280 ambiguous charges per day. Nobody resolves 17,280 cases
by hand, so automation is forced by the number, not by taste.

The resolution path:

1. Before calling the gateway, write an intent row with our own reference id and
   state PENDING. This must be durable before the network call, otherwise a
   crash loses the fact that we tried.
2. Send the reference id to the gateway as its own idempotency key, so the
   gateway deduplicates on our behalf.
3. On timeout, leave the row PENDING and return "processing" to the user.
4. A reconciler polls the gateway by reference id, on a backoff, and resolves
   PENDING to CAPTURED or FAILED.
5. Anything still PENDING after the gateway's own settlement horizon goes to a
   human queue with the reference id attached.

!!! tip "Self-explanation prompt"

    Say why step 1 has to be durable before step 2, and what specifically breaks
    if you write the intent row after the gateway call returns.

**The error most readers make here.** Retrying the gateway immediately on
timeout without sending the same gateway-side key. That is the mechanism that
produces the double charge you were hired to prevent.

#### Block D. Purpose: name what still breaks, and price the fix

| Residual risk | Frequency | What I do |
|---|---|---|
| Gateway lookup itself is down during reconciliation | Rare, correlated with the outage that caused the timeouts | Exponential backoff with jitter, and a hard cap on PENDING age before the human queue |
| Two of our processes handle the same key concurrently | Bounded by the IN_PROGRESS state | Accept, because the 409 makes it a client retry, not a duplicate charge |
| Idempotency table lost | Catastrophic | Same durability class as the ledger, so it rides the same replication and backup |
| Clock used to expire keys is skewed | Bounded by TTL slack | TTL is 30 days, skew is milliseconds, so no action |

I close by naming what I deliberately did not build: no distributed lock, no
consensus group, no leader election. One unique index and one reconciler carry
the whole correctness argument, and the operational surface is two objects
instead of a quorum.

### 9e. Faded examples

**Fade 1: exactly one order-confirmation email.** Same shape, last block blank.

- Block A, decide whether agreement is needed: two sends produce a duplicate
  email. Repair is a social apology, not a system action, but the cost is low.
  Row two of the routing table.
- Block B, make retries free: unique index on (order_id, template_version) in an
  outbox table, inserted in the same transaction as the order.
- Block C, survive the ambiguous send: the mail provider accepts a caller
  message id, so send it and treat a timeout as PENDING; the provider's events
  webhook resolves it.
- Block D, what still breaks and what it costs: **you write this block.**

??? note "Show answer"

    Residual risks. The webhook may never arrive, so a sweeper marks anything
    PENDING for more than one hour as UNKNOWN and re-sends only if the provider
    lookup says nothing was accepted. The outbox row and the order commit
    together, so a crash between them is impossible, which is the whole reason
    for the outbox rather than a direct call inside the request.

    Duplicate email
    on a genuine provider-side retry is accepted, because the cost is one
    confusing message and the alternative is a coordination protocol with an
    on-call rota. Cost: one table, one sweeper, one webhook endpoint.

**Fade 2: one nightly report per tenant across twelve app servers.** Last two
blocks blank.

- Block A, decide whether agreement is needed: two reports for one tenant means
  two emails and double compute. Repair is possible and cheap.
- Block B, make retries free: **you write this block.**
- Block C, survive the ambiguous run: **you write this block.**

??? note "Show answer"

    Block B. Insert a row into report_run with a unique index on
    (tenant_id, report_date) before doing any work. Whichever server wins the
    insert runs the report; the other eleven get a conflict and stop. No lock
    service, no leader election, and the database you already run is the
    arbiter. The winning server holds no lock, so a slow server cannot be fenced
    out mid-run, which is why the next block matters.

    Block C. The winner may crash after inserting and before finishing. Store a
    heartbeat_at column updated every 30 s. A sweeper reclaims any row whose
    heartbeat is older than 5 minutes by bumping an attempt counter, which both
    permits a retry and bounds it. Report generation writes to a temp location
    and renames on completion, so a partial run is never visible. Attempts above
    three go to a human queue rather than looping forever, because an infinite
    retry of a poisoned tenant is a self-inflicted outage.

### 9f. Check yourself

**Q1.** Your Raft group has five members across three availability zones with a
1 ms inter-zone round trip. A product manager asks for a globally consistent
counter readable from Sydney with a 40 ms p99. What do you tell them?

??? note "Show answer"

    The counter's writes commit in the region hosting the quorum, so a Sydney
    reader pays the one-way network cost to that region regardless of how fast
    the quorum is. If the group lives in Virginia, the great-circle distance to
    Sydney is roughly 15,500 km, and at 5 microseconds per km one way that is a
    77 ms floor before any processing.

    The 40 ms p99 is not achievable with a
    globally consistent read. Offer two alternatives: a local read of a
    bounded-staleness replica with the staleness stated in the API, or move the
    quorum to the region that has the strict requirement and let the others
    read stale.

**Q2.** A candidate proposes fencing tokens but the storage layer is a plain S3
compatible object store with no compare-and-set. What breaks, and what is the
cheapest repair?

??? note "Show answer"

    Fencing only works if the resource enforces it. An object store that
    unconditionally accepts a PUT cannot reject a stale token, so a paused
    holder overwrites the newer object. Cheapest repairs, in order of effort:
    write to a token-suffixed key and have readers resolve the highest token; or
    use the conditional-write primitive if the store offers one; or put a small
    database in front to hold the current token and the pointer, which is one
    extra component but restores the invariant.

### 9g. When not to use this

**Consensus (Raft, Paxos, or a hosted equivalent).**

| Question | Answer |
|---|---|
| Measurement that justifies it | You can show state where two concurrent writers produce a result no reconciliation can repair, AND a failover gap of 30 to 60 seconds is more expensive than a permanent quorum |
| Scale threshold below which it is over-engineering | Under roughly 1,000 writes per second against the coordinated object, with an availability target of 99.9% (43 minutes per month), a single primary plus a standby plus a written promotion runbook is cheaper and has fewer failure modes |
| Cheaper alternative | A unique constraint in the database you already run, an advisory lock, a single-partition log for ordering, or idempotency keys plus a reconciler |

**Distributed locks.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Duplicated work is wasting a measurable amount of capacity, for example more than 10% of a worker fleet, and the duplicated work has no side effect that a unique constraint could catch |
| Scale threshold | Below a fleet where duplicated work costs less than one instance, skip the lock and let all workers race on a unique index |
| Cheaper alternative | Unique index plus insert-wins, or partitioning work by a hash of the key so each worker owns a disjoint slice and no lock is needed |

**Hybrid logical clocks.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You have observed ordering anomalies attributable to wall-clock skew, or you need cross-node causal ordering and cannot route writes through a single sequencer |
| Scale threshold | Single-region, single-primary systems get a total order from the primary for free. Adding an HLC there is pure complexity |
| Cheaper alternative | A monotonic sequence from one database, or a version counter per entity with compare-and-set |

---

## Module 10. Multi-region and geo-replication

### 10a. Pre-question

```
+-------------------------------------------------------------+
| Current: one region, us-east. All users, all data.           |
| Measured, last 30 days:                                      |
|   traffic from Europe          18%                           |
|   p99 page latency, US users   210 ms                        |
|   p99 page latency, EU users   540 ms                        |
|   requests per page load       4 sequential API calls        |
|   TLS + TCP handshakes per new connection   2 round trips    |
| Target: p99 under 300 ms everywhere.                         |
+-------------------------------------------------------------+
Caption: current single-region latency by geography, with the
per-page request structure that determines how the gap arises.
```

**Question.** Before adding a region, how much of the 330 ms EU gap can you
recover without moving any data, and what does that imply about the decision?

??? note "Show answer"

    Virginia to Ireland is roughly 5,500 km. At 5 microseconds per km one way,
    that is a 27.5 ms one-way floor and a 55 ms round trip; real fibre routes run
    about 1.3 to 1.5 times great circle, so budget 75 to 85 ms round trip.

    Four sequential API calls cost four round trips, so about 300 to 340 ms of
    the EU page latency is round trips alone. Terminating TLS at an edge point of
    presence near the user removes the 2 handshake round trips (about 150 ms) and
    keeps a warm pooled connection back to origin. Collapsing four sequential
    calls into one aggregated call removes three more round trips.

    Doing both plausibly lands EU under 300 ms with zero data movement. The
    implication: latency alone rarely justifies a second region until the request
    path is already flat and edge-terminated. Bring the measurement, not the
    instinct.

### 10b. Explanation in small steps

**Only three things force a second region.** Say which one you are serving, out
loud, in the first minute.

| Forcing requirement | What it demands | What it does not demand |
|---|---|---|
| Latency | Compute and read-path data near the user | Writes near the user |
| Availability (survive losing a whole region) | A second copy of state, capacity to absorb the load, and a rehearsed failover | Active-active |
| Data residency (law or contract) | Storage and processing inside a legal boundary | Anything about latency |

Candidates who conflate these produce active-active designs for a latency
problem and pay conflict resolution costs forever.

**The distance table you can derive at the whiteboard.** Light in fibre travels
at about 200,000 km per second, so 5 microseconds per km one way and 10 per km
round trip. Multiply the great-circle distance, then add 30 to 50% for real
routing and switching.

| Pair | Great circle | RTT floor | Budget in practice |
|---|---|---|---|
| Virginia to Oregon | approx 3,900 km | 39 ms | 60 to 70 ms |
| Virginia to Ireland | approx 5,500 km | 55 ms | 75 to 85 ms |
| Virginia to Singapore | approx 15,500 km | 155 ms | 210 to 250 ms |
| Ireland to Singapore | approx 10,800 km | 108 ms | 150 to 180 ms |

Consequence, stated as a rule: no synchronous cross-region dependency belongs in
a user-facing request path with a p99 budget under about 400 ms, because one
crossing plus your own processing already consumes most of it.

**The three topologies, and the one most people mean.**

| Topology | Who accepts writes | Conflict risk | Failover | Typical use |
|---|---|---|---|---|
| Active-passive | One region | None | Promote standby, RPO equals replication lag, RTO is minutes | Availability requirement only |
| Home-region partitioned (read local, write home) | Every region, but only for entities it owns | None within an entity | Move a home region, slow but safe | Latency plus residency |
| Full active-active | Every region for everything | Guaranteed, needs a merge rule | Instant | Only when losing zero writes matters more than simple semantics |

The middle row is what most production "active-active" systems actually are, and
naming it precisely is a senior signal. Each user, tenant or account has a home
region recorded in a small global routing table. Reads are served locally from a
replica. Writes are proxied to the home region.

**Conflict resolution, compared honestly.**

| Strategy | Correct when | Fails when | Cost |
|---|---|---|---|
| Last writer wins on wall clock | Updates are idempotent and rare | Concurrent updates inside the skew window vanish silently | Free, and free is what it is worth |
| Last writer wins on hybrid logical clock | You need a total order and can accept losing one side | Same silent loss, but at least ordering is causal | Small |
| Version vectors, surface conflicts to the app | The app has a real merge rule | Vector size grows with writer count | Medium |
| CRDT (counters, sets, registers, text) | The data type genuinely has a commutative merge | Anything with an invariant like "balance must not go negative" | High, and the type must fit |
| Single writer per entity | Almost always, for business entities | Truly global objects, for example a global unique username | Low, this is the default you should reach for |

The pattern worth internalising: pick single-writer-per-entity first, and only
move down this table for the specific fields that genuinely need concurrent
writes from multiple regions.

**What cannot be partitioned by region.** Every design has a small set of
globally unique things. Name them early.

- Global uniqueness constraints, for example usernames or short codes.
- Cross-tenant aggregates, for example a global leaderboard or a total count.
- Anything a regulator requires to be a single authoritative record.

Standard treatments: allocate ranges of ids per region so uniqueness is local
(a region owns codes ending in its digit), or accept a single global service for
the tiny slice that needs it and give it a generous latency budget because it is
off the hot path.

**Residency changes the data model, not just the deployment.** If EU personal
data must not leave the EU, then a global secondary index containing that data
is illegal, a global cache of user objects is illegal, and cross-region lookups
must return opaque references that the owning region resolves. Interviewers
rarely raise this; raising it yourself is a strong signal.

**Cost, derived.** Assume 20,000 writes per second globally, 500 bytes per
replicated record including framing, and an inter-region transfer price of
$0.02 per GB. The price is an assumption: cloud list prices for inter-region
transfer sat in the low cents per GB range through 2026, and you should
substitute today's figure from your provider's page.

- Bytes: 20,000 times 500 B is 10 MB per second, 864 GB per day, about 26 TB per
  month.
- Active-passive, one direction: 26,000 GB times $0.02 is about $520 per month.
- Three-region mesh: each region generates a third of the writes and ships to
  two others, so 2 times 26 TB total, about $1,040 per month.

That is not the expensive part, and saying so is the point. The expensive parts
are the extra full copies of storage (a 50 TB dataset at $0.08 per GB-month is
$4,000 per copy per month, so two extra copies is $8,000), and the headroom:
each region must be able to absorb the others' traffic, so a two-region pair
running at 50% utilisation costs twice a single region running at 100%.

### 10c. ASCII diagram

```
     EU USER                            US USER
        |                                   |
        v                                   v
  +-------------+                     +-------------+
  | EDGE POP EU |                     | EDGE POP US |
  | TLS ends    |                     | TLS ends    |
  +------+------+                     +------+------+
         |                                   |
         v                                   v
  +---------------------+          +---------------------+
  | REGION eu-west      |          | REGION us-east      |
  | read replica        |<========>| read replica        |
  | writes for EU-homed |  async   | writes for US-homed |
  +----------+----------+          +----------+----------+
             |                                |
             +--------------+-----------------+
                            v
                  +-------------------+
                  | HOME REGION TABLE |
                  | user -> region    |
                  | tiny, cached      |
                  +-------------------+

Caption: home-region routing. Edges terminate TLS near the user, each
region owns writes for the users homed there, replication is
asynchronous, and one small global table records ownership.
```

### 10d. Worked example: take a photo-sharing app into Europe

**The prompt.** "We run in us-east only. Leadership says we are going global.
Design it."

#### Block A. Purpose: find out which requirement is actually forcing this

I do not start drawing. I ask which of the three forcing requirements applies,
because they produce different architectures.

The interviewer says: EU is 18% of traffic, EU users complain about speed, and
legal has raised residency for EU accounts. That is two forcing requirements
(latency and residency) and not the third (there is no stated requirement to
survive losing a whole region). So I am building home-region partitioning, not
full active-active, and I say that now so the rest of the conversation has a
spine.

I also state what I am ruling out and why: full active-active would let an EU
user write to us-east, which defeats the residency requirement and buys nothing,
since a user's writes come from one place at a time.

**The error most readers make here.** Accepting "we are going global" as a
requirement. It is a business statement. The design forks completely on which of
the three it means, and asking costs eight seconds.

#### Block B. Purpose: choose the partition axis, and name what will not partition

The axis is the account. Every account gets a home_region on creation, chosen by
signup IP with a manual override. Photos, comments, likes and the account record
all live in the home region.

What does not partition, and my answer for each:

| Global thing | Why it resists | What I do |
|---|---|---|
| Username uniqueness | One namespace by definition | Small global service, single primary in us-east, off the hot path, 250 ms budget is fine for signup |
| The follow graph across regions | An EU user follows a US user | Store each edge in the follower's home region, and fetch the followee's public profile from its region |
| Photo blobs viewed globally | A public photo has a worldwide audience | Blob in the home region, served through the CDN, which is where the global read scale actually lives |

!!! tip "Self-explanation prompt"

    Before Block C, say why storing the follow edge in the follower's region
    rather than the followee's is the right call for a feed read, and what it
    costs on the write side when a celebrity gains a million followers.

**The error most readers make here.** Partitioning by data type instead of by
entity owner. "Photos in EU, comments in US" creates a cross-region join on
every page load and violates residency at the same time.

#### Block C. Purpose: design the cross-region read with a stated staleness budget

An EU user opens a US user's profile. Options, and the numbers that choose:

| Option | EU p99 added | Staleness | Residency risk |
|---|---|---|---|
| Synchronous fetch from us-east | 75 to 85 ms per crossing | Zero | None, if only public data crosses |
| Async-replicated read replica of public profiles in EU | About 2 ms | Replication lag | None, if only public fields replicate |
| CDN-cached rendered profile fragment | Sub-millisecond on hit | Cache TTL | None |

I choose replicated public profiles plus CDN fragments, with a stated staleness
budget of 60 seconds for profile fields and 5 seconds for follower counts. I put
the number in the API contract, because "eventually consistent" without a number
is not a design, it is a shrug.

The residency line is drawn here: only public fields replicate. Email, phone,
precise location and private albums never leave the home region, so the EU
replica of a US profile carries display name, avatar URL and counts, nothing
else.

!!! tip "Self-explanation prompt"

    State the one measurement you would put on a dashboard to prove the 60 second
    staleness budget is being met, and what you would alert on.

**The error most readers make here.** Replicating the whole row because it is
easier, then discovering during the legal review that the design is illegal and
the schema has to be split anyway.

#### Block D. Purpose: cost the move and rehearse the failure

Costs, using the assumption prices above and a stated dataset of 50 TB:

| Line | Derivation | Monthly |
|---|---|---|
| EU compute | 18% of traffic, but provisioned for 30% to absorb growth and spikes | 30% of current compute |
| EU storage | EU accounts are 18% of 50 TB, that is 9 TB, at $0.08 per GB-month | approx $720 |
| Replicated public profiles | Assume 2% of the dataset is public profile fields, so 1 TB replicated both ways, at $0.08 | approx $160 |
| Cross-region transfer | Public profile deltas, assume 1 MB per second, 2.6 TB per month, at $0.02 per GB | approx $52 |

Total marginal storage and transfer is under $1,000 per month, which reframes
the conversation: the cost of this project is engineering time and operational
complexity, not infrastructure. Saying that out loud is a senior move.

Failure rehearsal, which is the block candidates skip:

- eu-west loses its database primary. EU users cannot write. US users are
  unaffected because they are homed elsewhere. Blast radius is 18% of write
  traffic, not 100%, and that containment is the main non-latency benefit.
- The home-region routing table is unavailable. Every request now cannot find
  its home. This is the new single point of failure I created, so it gets a
  read-only local cache in every region with a long TTL, and it is small enough
  (one row per account, tens of bytes) to hold entirely in memory.
- Replication lag to the EU public-profile replica exceeds 60 seconds. Serve
  stale and emit a metric, do not fail the request, because a stale display name
  is not an incident.

### 10e. Faded examples

**Fade 1: add a second region to a payments ledger.** Last block blank.

- Block A, forcing requirement: availability. The business states a target of
  surviving a full region loss with an RPO under 5 seconds.
- Block B, partition axis: none. A ledger has a global invariant (debits equal
  credits per account) and accounts are not geographically owned, so this is
  active-passive, not partitioned. Writes go to one region.
- Block C, cross-region read path: read replicas serve balance queries with a
  stated staleness of 5 seconds; anything that decides a transaction reads from
  the primary region.
- Block D, cost and failure rehearsal: **you write this block.**

??? note "Show answer"

    Cost. A standby region that accepts no traffic still needs the full dataset
    and enough compute to take over, so budget roughly one extra full copy of
    storage and 60 to 100% of primary compute, since a standby scaled to 30%
    cannot absorb a failover. That is close to a doubling, and the honest framing
    is that availability is the expensive requirement, not latency.

    Failure rehearsal. RPO under 5 seconds with asynchronous replication means
    you must alert when lag exceeds 2 seconds, because at failover time anything
    unreplicated is lost money. Rehearse the promotion quarterly against real
    traffic, measure RTO, and publish it. Decide in advance who declares a
    failover and on what evidence, because the expensive failure is a
    split-brain caused by promoting while the old primary is still writing.
    Fence it: the old primary must be provably stopped, or the promotion waits.

**Fade 2: add a second region for a session store.** Last two blocks blank.

- Block A, forcing requirement: latency. Session validation happens on every
  request, and a cross-region hop on every request is unaffordable.
- Block B, partition axis: **you write this block.**
- Block C, cross-region read path: **you write this block.**

??? note "Show answer"

    Block B. Do not partition and do not replicate. Sessions are derivable state
    with a short life, so the cheapest correct design is a signed, self-contained
    token validated locally in every region with no store at all, plus a small
    revocation list replicated everywhere. The partition axis question dissolves
    because the data stops being shared.

    Block C. The revocation list is the only thing that crosses regions. Size it:
    assume 10 M active users, a 0.1% daily revocation rate, and a 24 hour token
    lifetime, giving 10,000 entries, which is well under a megabyte and fits in
    memory in every region.

    Replicate it with a few seconds of lag and accept
    that a revoked token may work for a few extra seconds, or shorten the token
    lifetime to 15 minutes so the exposure window is bounded by the refresh
    instead. State which one you chose and why.

### 10f. Check yourself

**Q1.** Your interviewer says "make it active-active so we get better
availability". What is the one number you ask for, and how does the answer
change your design?

??? note "Show answer"

    Ask for the availability target as a time budget, not a percentage: how many
    minutes per month may we be down. 99.9% is 43 minutes per month, which a
    multi-AZ single-region deployment with a rehearsed failover usually meets.
    99.99% is 4.3 minutes per month, which a single region generally cannot meet
    because one bad deploy or control-plane incident exceeds it.

    Under 43 minutes, argue for multi-AZ and a documented runbook. Under 4.3
    minutes, you need a second region, but note that active-passive with a fast
    rehearsed failover meets an availability target; active-active is what you
    need when the RTO must be near zero, which is a stricter and more expensive
    requirement.

**Q2.** An EU user and a US user both edit the same shared album title within
50 ms of each other, and your regions are 80 ms apart. What does each
conflict-resolution strategy produce, and which do you ship?

??? note "Show answer"

    Neither region can have seen the other's write, so this is a genuine
    concurrent update. Last-writer-wins on wall clock silently drops one title
    and, because the writes fall inside the clock skew window, you cannot even
    predict which. Version vectors detect the conflict and hand it to the app,
    which for a title means showing the user a choice, which is bad product for a
    trivial field. A register CRDT still has to break the tie, so it degrades to
    last-writer-wins with a defensible rule.

    Ship single-writer: the album has an owner, the owner's home region accepts
    the title write, and the other region proxies. One 80 ms write penalty on a
    rare operation is much cheaper than a conflict model applied to every field.

### 10g. When not to use this

**A second region.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Far-geography p99 exceeds the SLO by more than the one-way network floor to your region even after edge TLS termination and request-path flattening, AND that geography is above roughly 15% of traffic; OR the availability budget is under about 26 minutes per year (99.995%) and single-region incidents already exceed it; OR a residency rule applies |
| Scale threshold below which it is over-engineering | Under about 10% of traffic from the far geography, with a 99.9% target (43 minutes per month), a second region is over-engineering |
| Cheaper alternative | Edge TLS termination (removes 2 round trips, roughly 150 ms at 75 ms RTT), request-path flattening, a CDN for static and cacheable fragments, and in-region multi-AZ redundancy |

**Full active-active with conflict resolution.**

| Question | Answer |
|---|---|
| Measurement that justifies it | A measured RTO requirement near zero, or a workload where a single entity genuinely receives concurrent writes from multiple regions (rare outside collaborative editing and shopping carts) |
| Scale threshold | If more than about 95% of entities are written from one region in any given hour, home-region partitioning captures nearly all the benefit with none of the conflict cost. Measure this before choosing |
| Cheaper alternative | Home-region partitioning with a documented home-move procedure, plus active-passive for the small set of genuinely global objects |

**Cross-region synchronous replication.**

| Question | Answer |
|---|---|
| Measurement that justifies it | An RPO of zero that is contractually or legally required, and a write path that can absorb a full inter-region round trip in its p99 budget |
| Scale threshold | If the write path's p99 budget is under about 400 ms, or the dependent write rate is above roughly 1,000 per second, synchronous cross-region replication will not fit (see the Raft latency derivation in Module 9) |
| Cheaper alternative | Asynchronous replication with a measured and alerted lag target, plus an outbox so nothing is lost between the transaction and the replica |

---

## Module 11. The failure library

Every named failure below is a shape, not a story. Learn the shape, the
amplification factor, the design decision that prevents it, and the signal that
proves it is happening. Naming the shape in an interview is worth more than
describing the symptom.

### 11a. Pre-question

```
+-------------------------------------------------------------+
| Service A calls Service B. Both are healthy at t=0.          |
|   A: 5,000 rps, timeout on B = 2 s, retries = 3, no jitter   |
|   B: capacity 6,000 rps, p99 latency 40 ms                   |
| At t=1 min, B's database gets slow. B's p99 goes to 3 s.     |
| At t=6 min, the database is fixed. B's p99 would be 40 ms    |
|   again if the offered load were 5,000 rps.                  |
+-------------------------------------------------------------+
Caption: a two-service call chain with its retry policy, capacity
and the timeline of a transient dependency slowdown.
```

**Question.** What is the offered load on B at t=2 min, and does B recover on
its own at t=6 min?

??? note "Show answer"

    At t=2 min every call to B exceeds the 2 s timeout, so A makes the original
    request plus 3 retries: 4 times 5,000, that is 20,000 rps offered against
    6,000 rps of capacity. B is 3.3 times over.

    B does not recover at t=6 min. B is now saturated by retries alone, so its
    latency stays above 2 s, so A keeps retrying, so the offered load stays at
    20,000 rps. The trigger has been removed and the system stays down. That is
    the defining property of a metastable failure: a sustaining feedback loop
    that outlives its cause. Recovery requires shedding load below the point
    where the loop breaks, which here means dropping offered load under 6,000
    rps, not waiting.

### 11b. Explanation in small steps

**The taxonomy that matters: does it self-heal when the trigger goes away?**

| Class | Self-heals | Escape route |
|---|---|---|
| Transient overload | Yes | Wait |
| Amplified overload | Usually | Wait plus capacity |
| Metastable | No | Shed load below the recovery threshold, then restore |

Only the third class needs a kill switch, and it is the class candidates never
name. The formal treatment is Bronson et al., *Metastable Failures in
Distributed Systems*, HotOS 2021
([paper](https://sigops.org/s/conferences/hotos/2021/papers/hotos21-s11.pdf),
accessed August 2026).

**The library.** Each row gives the amplification factor you can compute.

| Name | Trigger | Amplification | Prevented by |
|---|---|---|---|
| Retry storm | A dependency slows past the caller's timeout | 1 + retries, so 4x at 3 retries | Retry budget as a fraction of request rate, not a per-request count |
| Metastable failure | Any amplifier plus a sustaining loop | Unbounded in time, not in magnitude | A load shedder that can drop below the recovery threshold |
| Cache stampede | A hot key expires while under load | concurrency = rps times backend latency | Single-flight (one fill per key), or serve stale while refreshing |
| Thundering herd | Clients synchronise on a common clock or a common restart | All clients in one interval | Full jitter on every timer |
| Hot key or hot shard | Skewed key popularity | popular key share divided by uniform share | Split the key, replicate it, or cache in front |
| Split brain | Network partition plus automatic promotion | Two authorities, silent divergence | Quorum, fencing tokens, and a promotion gate |
| Cascading timeout | Caller timeout shorter than callee timeout | Work performed with nobody waiting for it | Deadline propagation, timeouts strictly decreasing down the chain |
| Queue backlog spiral | Arrival rate above service rate for long enough | Latency grows without bound; queue depth is the leading signal | Bounded queues plus shed on full, never unbounded |
| Poison pill | One message that crashes every consumer | Entire consumer fleet | Attempt counter and a dead-letter path |
| Fallback amplification | Fallback path shares capacity with the primary | Fallback fails exactly when it is needed | Fallbacks must use a disjoint resource or not exist |

**Derivation 1: the retry budget.** A per-request retry count of 3 sets a worst
case of 4x offered load, and your fleet is usually provisioned at 50 to 70%
utilisation, so 4x guarantees overload.

Instead, cap retries as a fraction of
the request rate with a client-side token bucket: allow retries up to 10% of the
current request rate. Worst case is then 1.1x, and healthy-state retries still
work because the bucket is nearly always full. AWS publishes a clear treatment
of this and of jitter:
[timeouts, retries, and backoff with jitter](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/)
(accessed August 2026).

**Derivation 2: cache stampede concurrency.** One key, 10,000 requests per
second, a 50 ms backend query. When the key expires, every request in the next
50 ms misses: 10,000 times 0.05 is 500 concurrent backend queries for one key.
If the backend allows 100 concurrent queries, you have just built an outage from
a cache expiry. With single-flight, the number is 1. With serve-stale-while-
revalidate, the number is 1 and no request waits.

**Derivation 3: jitter.** One million mobile clients poll every 300 seconds. If
they synchronise (a push restart, a midnight boundary, a deploy), the peak is
1,000,000 requests in roughly one second. With full jitter, meaning the next
interval is drawn uniformly from 0 to 300 s, the arrivals spread across 300 s
and the peak is about 3,300 per second. That is a 300x reduction from one line
of client code, which is why jitter is the highest-leverage item in this module.

**Derivation 4: hot key.** 1,000 shards, uniform traffic means each shard sees
0.1% of load. One celebrity account draws 5% of reads. At 200,000 reads per
second globally, that key's shard sees 10,000 reads per second against a shard
capacity of, assume, 2,000. It is 5x over while 999 shards idle. Fixes, in
increasing cost: cache the key at the edge with a short TTL; replicate the key
across N shards and read from a random one; split the key by suffix and merge on
read.

**Derivation 5: timeout ordering.** If the gateway times out at 1 s and the
service behind it times out at 2 s against the database, then between 1 s and
2 s the service is doing work no one will read, holding a connection and a
thread. Under load this is pure waste that deepens the overload. The rule:
timeouts strictly decrease along the call chain, and each hop passes its
remaining deadline down so the callee can abandon early.

**Little's law, the one formula worth memorising.** L = lambda times W. Items in
the system equals arrival rate times time in system.

- 5,000 requests per second at 40 ms means 200 requests in flight. A thread pool
  of 100 is already the bottleneck.
- The same 5,000 per second at 2 s means 10,000 in flight. Nothing you own has
  10,000 threads, so the queue is what actually absorbs it, and an unbounded
  queue converts an overload into an unbounded latency.

**The one rule that prevents most of this.** Every queue has a bound and a
shed policy. An unbounded queue does not prevent failure, it converts a fast
failure into a slow one and hides it from your latency alerts until the memory
runs out.

### 11c. ASCII diagram

```
   OFFERED LOAD                       CAPACITY LINE
        |                                   |
  4x  --+-------- retry storm --------------+---- overload
        |            |                      |
  1.1x--+---- retry budget caps here -------+---- safe
        |
        +--> sustaining loop? --> yes --> METASTABLE
        |                                (no self-heal)
        +--> sustaining loop? --> no  --> TRANSIENT
                                         (waits it out)

   ESCAPE FROM METASTABLE:
   +------------+   +--------------+   +----------------+
   | shed load  |-->| below recov. |-->| restore slowly |
   | (drop 80%) |   | threshold    |   | (ramp 10%/min) |
   +------------+   +--------------+   +----------------+

Caption: retry amplification against capacity, the test that decides
whether a failure self-heals, and the three-step escape from a
metastable state.
```

### 11d. Worked example: the feed service that would not come back

**The prompt.** "Your feed service p99 went from 80 ms to 30 s during a database
incident. The database was repaired 40 minutes ago. The feed service is still at
30 s. Diagnose and fix it, live."

Facts I have on the dashboard:

| Signal | Value |
|---|---|
| Feed service request rate at the load balancer | 8,000 rps, normal |
| Feed service request rate arriving at the database | 31,000 qps |
| Database p99 | 1.2 s (was 40 ms before the incident, is 40 ms in a test query now) |
| Feed pod count | 200, all at 95% CPU |
| Connection pool wait time | 28 s |
| Errors returned to users | 4% timeouts, 96% slow successes |

#### Block A. Purpose: decide which class of failure this is, before touching anything

The trigger is gone and the system has not recovered. That single fact moves
this out of "wait it out" and into metastable, and it changes what I do: no
amount of patience helps, and adding capacity may not help either, because
capacity that gets consumed by the loop just feeds the loop.

The proof is in the two rate numbers: 8,000 requests in, 31,000 queries out.
Roughly 3.9 queries per request against a baseline I expect to be near 1. That
ratio is the amplifier, and finding the amplifier is the whole first move.

**The error most readers make here.** Scaling up. Doubling pods doubles the
connection demand on a database that is already the constraint, which deepens
the loop. Under metastable conditions, scale-up is often the wrong direction.

!!! tip "Self-explanation prompt"

    Before Block B, say why the 8,000-in versus 31,000-out ratio is more
    diagnostic than the 30 s p99, and what a ratio of exactly 1.0 would have told
    you instead.

#### Block B. Purpose: identify the sustaining loop, not just the amplifier

The 3.9x could be retries or it could be a cache that stopped working. I check
cache hit rate: it fell from 94% to 11% during the incident and has not
recovered.

That is two loops stacked, and I name both:

1. Retry loop. Timeouts fire, clients retry 3 times, load quadruples.
2. Cache loop. Slow backends mean fills time out, so nothing repopulates, so
   every request goes to the database, so the database stays slow, so fills keep
   timing out. The cache cannot refill under load, which is the classic
   cold-cache metastable trap.

The second loop is the one that survives a retry fix, so I attack it too.

**The error most readers make here.** Finding the retry storm, fixing it, and
declaring victory. The cache loop alone keeps a system down.

#### Block C. Purpose: escape the state, in the order that actually works

Three actions, in this sequence, and the sequence matters:

| Step | Action | Expected effect | How I verify |
|---|---|---|---|
| 1 | Shed aggressively: reject 80% of requests at the gateway with a 503 and a Retry-After | Offered load to the database falls under the recovery threshold | Database p99 falls under 100 ms within a minute |
| 2 | Let the cache refill under the reduced load, with single-flight enabled so one fill per key | Hit rate climbs back toward 94% | Hit rate crosses 70% before I proceed |
| 3 | Restore traffic in ramps of 10% per minute | Load returns without re-entering the loop | If p99 rises above 200 ms at any step, hold the ramp |

Shedding 80% of traffic during an incident feels wrong and is correct. Serving
20% of users well beats serving 100% of users a 30 s timeout, and the ramp is
what stops the loop restarting the moment the gate opens.

!!! tip "Self-explanation prompt"

    Say why step 2 has to complete before step 3 begins, and what specifically
    happens if you restore full traffic to a cache at 11% hit rate.

**The error most readers make here.** Shedding 20% instead of 80%. The shed must
take offered load below the recovery threshold, and the recovery threshold is
well below normal capacity because a cold cache multiplies per-request cost.

#### Block D. Purpose: make the state unreachable, and price each fix

| Fix | Prevents | Cost | Priority |
|---|---|---|---|
| Retry budget: client token bucket at 10% of request rate | Retry storm, caps amplification at 1.1x | One library change, one metric | 1, highest ratio of prevention to effort |
| Single-flight on cache fills | Stampede and the refill trap | Small, in the cache client | 2 |
| Serve stale while revalidating, with a stale ceiling of 10 minutes | Cache loop entirely, since a slow backend no longer empties the cache | Requires a staleness decision from product | 3 |
| Deadline propagation with strictly decreasing timeouts | Cascading timeout waste | Cross-team, touches every service | 4 |
| Automatic load shedder at a defined queue depth | Removes the need for a human to decide to shed | Medium, plus a rehearsal, plus the risk of shedding when it should not | 5 |

I close by naming the SLI that would have caught this in the first two minutes:
queries-per-request as a ratio, alerted when it exceeds 1.5. That single derived
metric distinguishes "the database is slow" from "we are amplifying", and it is
the thing I would add before any of the fixes above.

### 11e. Faded examples

**Fade 1: the midnight spike.** Last block blank.

- Block A, classify: every night at 00:00 UTC the API sees a 40x spike lasting
  90 seconds, then normal. It self-heals, so it is not metastable. The shape is
  a thundering herd from clients whose daily sync is scheduled on a wall-clock
  boundary.
- Block B, find the amplifier: 2 M devices, all firing a daily sync at 00:00.
  Compressed into 90 s, that is roughly 22,000 rps against a normal 550 rps.
- Block C, escape now: raise the rate limit temporarily and let the queue absorb
  it, because it self-heals and the spike is short.
- Block D, make it unreachable and price each fix: **you write this block.**

??? note "Show answer"

    Primary fix: full jitter on the client's daily timer, drawing the next fire
    time uniformly across the 24 hour window rather than at a fixed hour. Spread
    over 86,400 s, 2 M devices give about 23 rps of sync traffic, a 950x
    reduction in peak. Cost: one client change, but it ships on the app release
    cycle, so it takes weeks to reach most devices and you cannot force it.

    Bridge fix while the client rolls out: have the server return a
    Retry-After-style next-sync-at value with jitter applied server-side, so
    control moves to a surface you can change today. Cost: one field, one client
    behaviour that already exists in most HTTP clients.

    Do not fix by provisioning for the peak. Provisioning for 22,000 rps to serve
    550 rps of steady load is a 40x over-provision that costs money every hour of
    every day to absorb 90 seconds of self-inflicted load.

**Fade 2: the checkout that dies on Black Friday.** Last two blocks blank.

- Block A, classify: at 3x normal load, checkout latency goes from 200 ms to
  9 s and stays there until the fleet is restarted. It does not self-heal, so it
  is metastable.
- Block B, find the amplifier and the loop: **you write this block.**
- Block C, escape now: **you write this block.**

??? note "Show answer"

    Block B. Look for the ratio first: calls-per-request to each dependency,
    compared with the healthy baseline. The typical finding here is an unbounded
    in-process queue: at 3x load, arrival rate exceeds service rate, so by
    Little's law the in-flight count grows without bound and time-in-system grows
    with it. The sustaining loop is that longer queueing pushes requests past the
    client timeout, the client retries, and the retry enters the same unbounded
    queue. The restart works because it empties the queue, which is a diagnostic
    clue, not a fix.

    Block C. Bound the queue immediately (reject when depth exceeds arrival rate
    times the target latency, so at 5,000 rps and a 200 ms target the bound is
    1,000) and shed on full with a 503.

    Then shed at the gateway until offered
    load falls under measured service rate, and ramp back. Verify with queue
    depth, not latency: depth is the leading indicator and falls first. Do not
    restart the fleet, because a restart discards in-flight work and hides the
    fact that the queue bound is the actual defect.

### 11f. Check yourself

**Q1.** A service has a 99.99% success rate and its client retries once on
failure. A colleague argues that retries make the effective success rate
99.999999%. What is wrong with that reasoning?

??? note "Show answer"

    It assumes failures are independent, and the failures that matter are not.
    Independent failures (a single dropped packet, one unlucky pod) do compose
    that way, and a retry genuinely helps. Correlated failures (the dependency is
    overloaded, a bad deploy, a partition) affect the retry exactly as much as
    the original, so the retry buys nothing and adds load precisely when the
    system is least able to take it.

    The correct framing: retries convert independent failures into successes and
    convert correlated failures into amplification. That is why the control is a
    retry budget as a fraction of request rate, which is generous during
    independent failures (rare, so the bucket is full) and restrictive during
    correlated ones (common, so the bucket drains).

**Q2.** Name the leading indicator you would alert on for a queue backlog
spiral, and say why the obvious latency alert is too late.

??? note "Show answer"

    Alert on queue depth, or equivalently on the ratio of arrival rate to service
    rate crossing 1.0 for a sustained window. By Little's law, time in system
    equals items divided by arrival rate, so latency is a consequence of depth
    and lags it. By the time p99 latency crosses your alert threshold, the queue
    already holds the work that will keep it crossed for minutes, and any action
    you take has to drain a backlog rather than prevent one.

    A useful pairing: alert on depth for the page, and on latency for the SLO
    burn. One tells you to act, the other tells you it already cost the user.

### 11g. When not to use this

**Automatic load shedding.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You have observed at least one metastable event, or your traffic has a measured burst factor above about 3x that exceeds provisioned capacity, and manual response time exceeds the time to full saturation |
| Scale threshold | Below a few thousand requests per second on a fleet that can autoscale within the time it takes to saturate, a bounded queue plus a fast 503 on full is enough; a separate shedding tier is over-engineering |
| Cheaper alternative | Bounded queues, a hard concurrency limit per process, and a documented manual gateway switch that on-call is trained to flip |

**Circuit breakers.**

| Question | Answer |
|---|---|
| Measurement that justifies it | A dependency whose failures are measurably correlated and long-lived (minutes, not seconds), where continuing to call it wastes capacity you need for other paths |
| Scale threshold | With a single dependency and one caller, a timeout plus a retry budget already caps the waste. Breakers earn their keep when one caller fans out to several dependencies and must protect the healthy ones |
| Cheaper alternative | Timeouts plus retry budget plus bounded concurrency per dependency. A concurrency limit alone gives you most of a breaker's isolation with none of its state machine or its half-open tuning |

**Fallback paths.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The fallback uses a genuinely disjoint resource (different data store, different region, precomputed static content) and you have tested it under the load it would receive |
| Scale threshold | Any fallback that shares a database, a network path or a thread pool with the primary is not a fallback. Below that bar, do not build one |
| Cheaper alternative | Degrade explicitly: return a smaller, cheaper response (fewer feed items, no personalisation, cached-only) rather than routing to a second full implementation you never exercise |

---

## Module 12. Observability and operability as design output

A design you cannot operate is not a design. This module treats the SLI set, the
alert set, the label schema and the debug path as artefacts you draw on the
whiteboard, in the same pass as the boxes.

### 12a. Pre-question

```
+--------------------------------------------------------------+
| Proposed metric, one per HTTP request:                        |
|   http_request_duration_seconds                               |
| Labels:                                                       |
|   route          (200 distinct values)                        |
|   status_code    (8 distinct values)                          |
|   pod            (30 pods)                                    |
|   region         (3 regions)                                  |
|   customer_id    (proposed by a stakeholder, 1,000,000 values)|
| Scrape interval: 15 s.  Histogram: 12 buckets.                |
+--------------------------------------------------------------+
Caption: a proposed request-duration metric with its label
cardinality, scrape interval and bucket count.
```

**Question.** How many time series does this produce with and without
customer_id, and what does that do to storage per day?

??? note "Show answer"

    Series count is the product of label cardinalities times the bucket count.

    Without customer_id: 200 times 8 times 30 times 3 is 144,000 label
    combinations, times 12 buckets is 1,728,000 series.

    With customer_id: multiply by 1,000,000, giving 1.728 trillion series. That
    is not a large bill, it is an impossible system.

    Storage, without customer_id: at 15 s scrape there are 5,760 samples per
    series per day, so 1,728,000 times 5,760 is about 9.95 billion samples per
    day. At an assumed 2 bytes per compressed sample (a common figure for
    delta-of-delta and XOR compression on regular numeric series, and one you
    should verify against your own store), that is about 20 GB per day, or
    roughly 600 GB per month before downsampling.

    The design rule that follows: unbounded labels go on traces and logs, which
    are sampled, never on metrics, which are not.

### 12b. Explanation in small steps

**Four definitions, in dependency order.**

| Term | Definition |
|---|---|
| SLI | A ratio: good events divided by valid events, measured where the user experiences it |
| SLO | A target for an SLI over a window, for example 99.9% of valid requests under 300 ms over 28 days |
| Error budget | The permitted bad fraction: 100% minus the SLO, expressed as time or as a count |
| Burn rate | How fast you are consuming the budget relative to spending it evenly across the window |

**Error budget as time, derived.** A 30 day month is 43,200 minutes.

| SLO | Bad fraction | Budget per 30 days |
|---|---|---|
| 99% | 0.01 | 432 minutes (7.2 hours) |
| 99.9% | 0.001 | 43.2 minutes |
| 99.95% | 0.0005 | 21.6 minutes |
| 99.99% | 0.0001 | 4.32 minutes |
| 99.999% | 0.00001 | 26 seconds |

Use this table to end arguments. When someone asks for five nines, show them 26
seconds per month and ask which deploy fits in it.

**Burn-rate alerting, derived.** Alerting on "error rate above 1%" pages you for
things that do not threaten the objective and misses slow bleeds. Alert on the
fraction of budget consumed instead.

- A one hour window is 1/720 of a 30 day month.
- Burning at 14.4 times the even rate for one hour consumes 14.4/720, that is
  2%, of the month's budget.
- Burning at 6 times for six hours consumes 6 times 6/720, that is 5%.
- Burning at 1 time for the whole month consumes exactly 100%, by definition.

So: page on a fast burn (14.4x over 1 hour, confirmed by a 5 minute window to
avoid single-scrape noise), and open a ticket on a slow burn (3x over 24 hours).
The canonical public treatment is the Google SRE Workbook chapter on
[alerting on SLOs](https://sre.google/workbook/alerting-on-slos/)
(accessed August 2026).

**Two method families, and when each applies.** RED covers request-driven
services: Rate, Errors, Duration. USE covers resources: Utilisation, Saturation,
Errors. Brendan Gregg's published description of the USE method is at
[usemethod](https://www.brendangregg.com/usemethod.html) (accessed August 2026).

| You are looking at | Use | Because |
|---|---|---|
| An HTTP or RPC service | RED | The user's experience is a request outcome |
| A CPU, disk, connection pool, thread pool | USE | The failure mode is exhaustion, and saturation leads latency |
| A queue or stream | Both, plus lag | Consumer lag is the one signal that predicts everything else |

Saturation is the item people omit and the one that pages earliest. Connection
pool wait time, queue depth and thread pool queue length are all saturation
signals that rise before latency does.

**Choose SLIs from the user journey, not from the architecture.** For each
journey, one availability SLI and one latency SLI. Two rules:

- Measure as close to the user as you can afford. A server-side 200 that the
  client never received is not a good event.
- Count valid events carefully. Requests from a load test, from a scanner, or
  with a malformed body are not valid events, and including them either hides
  real failures or manufactures fake ones.

**Cardinality budget, as a design constraint.** Give yourself a number before
you design the labels, the same way you give yourself a latency budget.

| Rule | Practical form |
|---|---|
| Set a series budget per service | For example 50,000 series, which at 15 s scrape and 2 bytes per sample is about 580 MB per day |
| Never label with an unbounded set | No user id, request id, URL with ids in it, error message, or full stack |
| Normalise route labels | /users/12345/posts becomes /users/{id}/posts, or your 200-route label becomes millions |
| Put high-cardinality context on traces and exemplars | Traces are sampled, so cardinality costs you nothing there |

**Tracing, and the two sampling strategies.**

| Strategy | How it works | Good at | Bad at |
|---|---|---|---|
| Head sampling | Decide at the first service, propagate the decision | Cheap, simple, consistent across the trace | At 1%, you almost never capture the rare slow trace you need |
| Tail sampling | Buffer the full trace, decide after it completes | Keeps all errors and all slow traces | Needs a buffering tier and memory proportional to trace duration times rate |

A pragmatic default: head sample at a low rate for the baseline, and force-sample
every trace that errors or exceeds a latency threshold, using a sampling
decision propagated in the trace context so the whole trace is kept.

**Trace propagation is a design constraint, not a library setting.** Every hop
that drops the context breaks the trace: a queue that does not carry headers, a
thread pool that does not propagate context, a third party that strips unknown
headers. Draw the propagation path on the diagram and mark where it breaks.

**The operability block every design should end with.** Four items, in order:

1. Three SLIs with their measurement point.
2. Two alerts: one fast-burn page, one slow-burn ticket.
3. One dashboard that answers "is it us or a dependency" in under 30 seconds.
4. One debug path: given a user complaint with a timestamp, the exact steps to
   find their request.

### 12c. ASCII diagram

```
  USER JOURNEY                 SIGNALS PRODUCED
  +---------------+
  | open feed     |---> SLI: good = 200 in < 300 ms
  +-------+-------+           valid = authenticated GET /feed
          |
          v
  +---------------+
  | GATEWAY       |---> RED: rate, errors, duration (low card)
  |               |---> trace starts here, id propagates down
  +-------+-------+
          |
          v
  +---------------+     +----------------+
  | FEED SERVICE  |---->| CACHE          |--> USE: hit rate,
  |               |     |                |    eviction, memory
  +-------+-------+     +----------------+
          |
          v
  +---------------+
  | DATABASE      |---> USE: pool wait, saturation, replica lag
  +---------------+

  ALERTS: fast burn 14.4x/1h -> page | slow burn 3x/24h -> ticket

Caption: signals mapped onto one user journey. RED at request
boundaries, USE at resources, one SLI defined at the user-visible
edge, and the two burn-rate alerts derived from it.
```

### 12d. Worked example: make the URL shortener operable

**The prompt.** "You designed a URL shortener. Now tell me how you run it."
Given: 10,000 redirects per second, 100 creates per second, redirect p99 target
50 ms, business impact is that a failed redirect is a lost click.

#### Block A. Purpose: pick SLIs from what the user does, not from what we built

Two journeys, so two SLI pairs. I refuse to write an SLI for the cache or the
database, because those are causes, not user experiences.

| Journey | Availability SLI | Latency SLI | Measured where |
|---|---|---|---|
| Follow a short link | HTTP 301 or 302 divided by all valid GET /{code} | Fraction under 50 ms | CDN or edge log, because that is what the user actually got |
| Create a short link | HTTP 201 divided by all valid authenticated POST | Fraction under 400 ms | Gateway |

Valid events exclude requests for codes that never existed (those correctly
return 404 and are not our failure) and exclude synthetic load-test traffic by
user agent. I say the exclusions out loud, because a candidate who defines valid
events precisely is signalling operational experience.

**The error most readers make here.** Defining the SLI on the server's own
response code. If the edge returns 502 because our origin was slow, our origin
logs a success and our SLI stays green while every user fails.

!!! tip "Self-explanation prompt"

    Before Block B, say why "redirect availability" and "create availability"
    should have different SLO targets here, using the business impact given
    above.

#### Block B. Purpose: set targets the business can defend, then derive the alerts

Redirects lose money per failure and are read-only, so the target is high:
99.95%, which is 21.6 minutes per 30 days. Creates are retried by a human and
happen 100x less often, so 99.9% (43.2 minutes) is honest and cheaper.

Alerts derived from 99.95% on redirects:

| Alert | Condition | Budget consumed if it fires and lasts | Action |
|---|---|---|---|
| Fast burn page | Error ratio above 14.4 times 0.0005, that is 0.72%, over 1 hour, confirmed over 5 minutes | 2% per hour | Page on-call |
| Slow burn ticket | Error ratio above 3 times 0.0005, that is 0.15%, over 24 hours | 10% per day | File a ticket in business hours |

Notice that the fast-burn page fires at 0.72% errors and the slow-burn ticket at
0.15%. Both are derived from one SLO, not chosen by feel, and that derivation is
the thing to show.

**The error most readers make here.** Setting a static "alert if errors above
1%" threshold. At 10,000 rps, 1% is 100 failures per second, and a 99.95% SLO is
blown in under four minutes at that rate. The static threshold is far too loose
for the fast case and silent for the slow bleed.

#### Block C. Purpose: design the label schema inside a stated cardinality budget

Budget: 50,000 series for this service. Now I spend it.

| Metric | Labels | Series |
|---|---|---|
| redirect_requests_total | status (4), region (3), edge_pop (40) | 480 |
| redirect_duration_seconds | region (3), edge_pop (40), 12 buckets | 1,440 |
| create_requests_total | status (6), region (3) | 18 |
| create_duration_seconds | region (3), 12 buckets | 36 |
| cache_operations_total | op (3), result (2), region (3) | 18 |
| db_pool_wait_seconds | region (3), 12 buckets | 36 |

Total is about 2,000 series, comfortably inside 50,000, which leaves room for
the metrics I have not thought of yet. Explicitly banned as labels: the short
code (unbounded), the destination URL (unbounded), the user id (unbounded). All
three go on traces and on sampled logs instead, where a 1% sample makes them
free.

!!! tip "Self-explanation prompt"

    Say what you would lose by banning the short code as a label, and how you
    would still answer the question "which code is generating all our traffic".

**The error most readers make here.** Adding the short code as a label because
hot-key detection sounds like a metrics problem. It is a top-K problem: solve it
with a sampled top-K stream or a periodic log aggregation, not with a time
series per code.

#### Block D. Purpose: write the debug path before the incident

Given a user saying "my link was broken around 14:32 UTC", the on-call steps:

1. Dashboard panel one: SLI for redirects, last 6 hours, split by region and
   edge point of presence. Answers "was it everyone or one place".
2. Dashboard panel two: dependency panel, cache hit rate and database pool wait,
   same window. Answers "is it us or a dependency" in under 30 seconds, which is
   the stated requirement.
3. If it was localised, query sampled logs for the short code in the window.
   Codes are on logs, not metrics, which is exactly why the sampling rate for
   error traces is 100%.
4. Pull the trace by id from the log line, and read the span durations.

I finish by naming the one derived metric I would add beyond the standard set:
database queries per redirect. Its healthy value is near 0.06 (a 94% cache hit
rate), and an alert at 0.3 catches a cache degradation before latency moves.

### 12e. Faded examples

**Fade 1: operability for a queue-based image worker.** Last block blank.
Given: 500 images per second arriving, target is 95% processed within 60 s.

- Block A, SLIs from the journey: freshness, meaning the fraction of images
  whose end-to-end age at completion is under 60 s, measured from the enqueue
  timestamp, not the dequeue timestamp. Plus a success ratio on processing.
- Block B, targets and alerts: 95% within 60 s over 28 days. Fast burn at 14.4x
  over one hour pages; oldest-unprocessed-message age over 120 s pages
  immediately, because that is the leading signal.
- Block C, label schema inside budget: image_type (6), status (4), region (3),
  plus a duration histogram, staying near 300 series. No image id, no user id,
  no filename as labels.
- Block D, the debug path: **you write this block.**

??? note "Show answer"

    Given "my upload from 09:14 never appeared": first panel is consumer lag and
    oldest-message age, which distinguishes a stalled consumer from a single
    failed item. Second panel is dead-letter queue depth and its rate, which
    catches a poison pill. If both are healthy, the item failed individually, so
    query sampled logs by user id in the window and pull the trace from the log
    line, which is why the enqueue span must carry the trace context into the
    queue message headers.

    The step people forget: verify the message was ever enqueued. Instrument the
    producer with its own counter, because a failure to enqueue is invisible in
    every consumer-side metric and is a common cause of "it just vanished".

**Fade 2: operability for a shared cache tier.** Last two blocks blank.
Given: 200,000 operations per second, 40 nodes, used by 6 services.

- Block A, SLIs: the cache has no direct user journey, so the honest position is
  that it gets no SLO of its own; it gets USE signals and its effect appears in
  the SLOs of the six services that depend on it.
- Block B, targets and alerts: **you write this block.**
- Block C, label schema inside budget: **you write this block.**

??? note "Show answer"

    Block B. Alert on saturation and on the dependent services' derived ratios,
    not on hit rate in isolation. Concretely: page on memory utilisation above
    85% (eviction rate rises non-linearly past that), page on connection or
    command queue depth above the value implied by Little's law for your latency
    target, and ticket on hit rate dropping more than 10 points below its 7 day
    baseline for any one service. A global hit rate hides one service's collapse
    behind five healthy ones, which is why the baseline comparison is per
    service.

    Block C. Labels: node (40), service (6), operation (4), result (3). That is
    2,880 series for counters plus a per-service latency histogram at 6 times 12,
    that is 72 more. Banned: cache key, key prefix if prefixes are user-derived,
    and error message text. For hot-key detection use a sampled top-K over keys
    emitted as a periodic log record, not as labels.

### 12f. Check yourself

**Q1.** Your service reports 99.98% availability from its own metrics, but users
report frequent failures. Name two structural reasons the number can be right
and useless.

??? note "Show answer"

    First, the measurement point. Server-side success does not mean the user
    received a response. Failures in the load balancer, at the edge, in TLS
    negotiation, or on the client's network never reach your counters. Moving the
    SLI to the edge log, or to client-reported telemetry, usually drops the
    reported number and makes it true.

    Second, the definition of valid events. If health checks, retries, cheap
    static asset requests or a chatty polling endpoint dominate the denominator,
    then a small number of failures on the expensive endpoint that users actually
    care about is diluted to invisibility. Split the SLI per journey and the
    dilution disappears.

**Q2.** A stakeholder asks for 99.999% availability. Give the two-sentence reply
that moves the conversation forward rather than refusing.

??? note "Show answer"

    "That is 26 seconds of downtime per 30 days, which means no deploy, no
    config change and no dependency incident may ever produce a user-visible
    error for more than about four seconds a week. Can we look at which specific
    user journey needs that, because we can usually give one critical path a much
    higher target cheaply while the rest sit at 99.9%."

    The move is to convert the percentage into time, then to narrow the scope
    from the whole system to a journey. That reframing is nearly always accepted,
    and it is a senior signal because it treats reliability as a budget to
    allocate rather than a number to argue about.

### 12g. When not to use this

**Distributed tracing.**

| Question | Answer |
|---|---|
| Measurement that justifies it | A user request touches four or more services, or median incident diagnosis time exceeds about 30 minutes with the question "which hop was slow" still unanswered |
| Scale threshold | With one or two services, structured request logs carrying a correlation id give you the same answer. Below three hops, tracing is over-engineering |
| Cheaper alternative | A correlation id generated at the edge, logged by every component, plus per-dependency latency histograms in each service |

**Tail sampling.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Head sampling at your affordable rate misses the traces you need, provable by counting how many of last month's investigations found no sampled trace for the reported request |
| Scale threshold | Under roughly 1,000 requests per second, sample at 100% of errors and 10% of successes and skip the buffering tier entirely |
| Cheaper alternative | Head sampling plus a forced-sample flag set whenever a request errors or exceeds a latency threshold, propagated through the trace context |

**A dedicated metrics platform per team.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The shared platform's ingest limit or query latency is measurably blocking incident response, with dates and incident numbers |
| Scale threshold | Below a few million active series per team, a shared platform with per-team quotas is cheaper in both money and staffing |
| Cheaper alternative | A per-team series budget with a chargeback report, plus recording rules that pre-aggregate the expensive queries |

---

## Module 13. Cost as a first-class constraint

### 13a. Pre-question

```
+--------------------------------------------------------------+
| Service, steady state:                                        |
|   10,000 requests per second                                  |
|   5 ms of CPU per request                                     |
|   2 KB response body, served to the public internet           |
|   target CPU utilisation 60%                                  |
| Assumed list prices (substitute your provider's current page):|
|   compute        $0.04 per vCPU-hour                          |
|   egress         $0.08 per GB to the internet                 |
+--------------------------------------------------------------+
Caption: a steady-state service with its CPU cost per request,
response size and the two unit prices needed to cost it.
```

**Question.** Which is larger, the compute bill or the egress bill, and by how
much?

??? note "Show answer"

    Compute. 10,000 requests per second times 5 ms is 50 CPU-seconds of work per
    second, so 50 vCPUs busy. At 60% target utilisation you provision 50 divided
    by 0.6, that is about 84 vCPUs. Over a 730 hour month: 84 times $0.04 times
    730 is about $2,453.

    Egress. A 30 day month is 2,592,000 seconds, so 10,000 rps is 25.92 billion
    requests. At 2 KB each that is 51,840 GB, and at $0.08 per GB that is about
    $4,147.

    Egress is about 1.7 times compute. This is the single most useful reflex in
    cost work: for read-heavy services with real payloads, the bytes usually cost
    more than the CPU, and the cheapest optimisation is nearly always to send
    fewer bytes (compression, field pruning, caching at the edge) rather than to
    tune the code.

### 13b. Explanation in small steps

**Assumed unit prices used throughout this module.** These are order-of-magnitude
list prices in the range cloud providers published through 2026. They are
labelled assumptions, and the point is the method, not the digits. Substitute
your provider's current page before quoting any of this in a real design review.

| Resource | Assumed price |
|---|---|
| vCPU | $0.04 per vCPU-hour |
| Memory | $0.005 per GB-hour |
| Block storage (SSD) | $0.08 per GB-month |
| Object storage (standard) | $0.021 per GB-month |
| Object storage (infrequent access) | $0.010 per GB-month |
| Inter-zone transfer | $0.01 per GB |
| Inter-region transfer | $0.02 per GB |
| Internet egress | $0.08 per GB |
| CDN egress | $0.02 per GB |

Two structural facts worth more than the numbers: moving bytes usually costs
more than storing them for a month, and every price above is per unit, so the
only way to argue about cost is to know your units per request.

**Express everything as a unit cost.** Three units cover almost every prompt.

| Unit | Formula | Use it to |
|---|---|---|
| Cost per 1,000 requests | (monthly compute + egress + per-request storage IO) divided by (monthly requests / 1,000) | Compare two designs for the same feature |
| Cost per user per month | Total monthly cost divided by monthly active users | Argue against a feature, or for a price |
| Cost per GB stored per month, by tier | Storage price times replication factor times retention | Decide retention and tiering |

From the pre-question: $6,600 per month over 25.92 million thousands of requests
is about $0.00025 per 1,000 requests. That number is more useful than the total
because it survives a traffic forecast.

**The four cost shapes, and what each one implies.**

| Shape | Grows with | Cheapest lever | Trap |
|---|---|---|---|
| Compute | Requests times CPU per request | Reduce work per request, raise utilisation target | Autoscaling that never scales down |
| Egress | Requests times bytes per response | Compress, prune fields, cache at the edge | Chatty internal calls crossing zones |
| Storage | Data times replication times retention | Retention policy, tiering, compression | Keeping three replicas of data you never read |
| Coordination and cross-zone chatter | Internal calls times bytes times cross-zone probability | Zone-aware routing, batching | Invisible in every dashboard until the bill arrives |

**Derivation: the cross-zone tax.** Three availability zones, callers pick
targets uniformly, so 2 out of 3 calls leave the zone. Take the pre-question
service: 25.92 billion requests per month, 3 internal calls each, 4 KB of
request plus response per internal call.

- Internal bytes: 25.92e9 times 3 times 4 KB is about 311 TB per month.
- Crossing a zone boundary: two thirds, so about 207 TB.
- At $0.01 per GB: 207,000 GB times $0.01 is about $2,070 per month.

That is 84% of the compute bill, generated by nothing the architecture diagram
shows. Zone-aware routing (prefer a same-zone replica, fall back across zones)
removes most of it and costs one load-balancer setting. This is the highest
value-per-word cost point you can make in an interview.

**Derivation: when a cache pays for itself.** A cache is worth its cost when the
avoided backend work exceeds the cache's own bill.

- Backend read cost: assume a database read consumes 2 ms of CPU. At $0.04 per
  vCPU-hour, one CPU-second is $0.0000111, so one read is about $0.0000000222.
- 10,000 reads per second for a month is 25.92 billion reads, costing about $576
  in database CPU.
- A 64 GB cache node costs, at the memory price, 64 times $0.005 times 730, that
  is about $234 per month, and you need at least two for redundancy, so $468.

So a 94% hit rate saves about $541 of database CPU and costs $468. On money
alone it is roughly break-even, and the honest conclusion is that caches are
almost never justified by cost. They are justified by latency and by protecting
a backend that cannot be scaled horizontally. Saying this correctly separates
you from candidates who add a cache reflexively and from candidates who claim it
saves money it does not save.

**Derivation: retention as the dominant storage lever.** 500 GB of new data per
day, 3 replicas, block storage at $0.08 per GB-month.

| Retention | Data at steady state | Monthly cost |
|---|---|---|
| 30 days | 500 times 30 times 3, that is 45 TB | 45,000 times $0.08, about $3,600 |
| 90 days | 135 TB | about $10,800 |
| 365 days | 547 TB | about $43,800 |

Now tier: keep 30 days hot on block storage and the rest on object storage at
$0.021 with a single replica handled by the service.

- Hot: 45 TB at $0.08 is $3,600.
- Cold, 335 days at 500 GB per day is 167 TB, at $0.021 is about $3,507.
- Total about $7,100 against $43,800, a 6x reduction, for the cost of one
  lifecycle policy and a slower read path for old data.

**How to argue for a cheaper design without sounding cheap.** Four moves, in
order of effectiveness:

| Move | Sentence shape |
|---|---|
| Convert to a unit the business already uses | "This is 4 cents per active user per month, against an ARPU of $2" |
| Attach the cost to a requirement, not to taste | "The 90 day retention costs $7,200 a month. If the actual need is 30 days for debugging and a yearly export for audit, it is $3,600 plus a batch job" |
| Offer the expensive version as an option with a trigger | "I would start with one region and revisit at 15% EU traffic, which is the point where the latency SLO fails" |
| Name what you are buying, not what you are saving | "Spending $2,000 a month on the second region buys a 4 minute RTO instead of a 45 minute one" |

The failure mode to avoid is proposing the cheap design without the number, which
reads as timidity, or proposing the expensive one without the number, which reads
as inexperience.

### 13c. ASCII diagram

```
  COST OF ONE REQUEST, DECOMPOSED
  +--------------------------------------------------------+
  | CPU        5 ms  x $0.04/vCPU-hr / 3600 = $0.000000056  |
  | EGRESS     2 KB  x $0.08/GB             = $0.00000016   |
  | INTERNAL   3 calls x 4 KB x 2/3 x $0.01 = $0.00000008   |
  | STORAGE IO amortised per request        = small         |
  +--------------------------------------------------------+
        |            |               |
        v            v               v
  +-----------+ +-----------+ +---------------+
  | fix by    | | fix by    | | fix by        |
  | less work | | fewer     | | zone-aware    |
  | per req   | | bytes     | | routing       |
  +-----------+ +-----------+ +---------------+

Caption: one request's cost split into three components with the
lever that reduces each. Egress dominates CPU by roughly 3x here, and
invisible internal chatter is comparable to CPU.
```

### 13d. Worked example: cost the news feed fan-out decision

**The prompt.** "Push or pull for the feed? Justify it with numbers."

Given and assumed inputs, stated before any arithmetic:

| Input | Value | Status |
|---|---|---|
| Registered users | 100 M | GIVEN |
| Daily active users | 30 M | GIVEN |
| Posts per day | 1 M | GIVEN |
| Median followers per poster | 200 | GIVEN |
| Feed opens per DAU per day | 4 | GIVEN |
| Feed entries kept per user | 500 | ASSUMED, because two screens of scroll at 25 items is 50 and 10 pages of history is a generous ceiling |
| Feed entry row size | 200 bytes | ASSUMED, post id, author id, timestamp, score, flags |
| Largest account's followers | 50 M | GIVEN |

#### Block A. Purpose: cost the push design at the median, where it looks good

Fan-out on write. Every post is copied into each follower's feed list.

- Feed inserts: 1 M posts times 200 followers is 200 M inserts per day.
- Average insert rate: 200 M divided by 86,400 is about 2,315 per second. Peak at
  3x is about 7,000 per second, which one well-provisioned cluster handles.
- Storage: 30 M active users times 500 entries times 200 bytes is 3 TB. At $0.08
  per GB-month with 3 replicas, that is 3,000 times $0.08 times 3, about $720 per
  month.
- Read cost: a feed open is one range scan of a contiguous list. 30 M DAU times 4
  opens is 120 M reads per day, about 1,390 per second.

The push design's read path is nearly free, which is why it wins by default.

**The error most readers make here.** Stopping here. The median is not where
this design fails, and an interviewer who hears only the median number concludes
you have not built one of these.

!!! tip "Self-explanation prompt"

    Before Block B, predict which single input in the table above breaks the push
    design, and by roughly what factor.

#### Block B. Purpose: find the input that eliminates push, and quantify it

The 50 M follower account. Assume it posts 5 times a day.

- Inserts from that one account: 50 M times 5 is 250 M per day.
- The entire rest of the system generates 200 M per day.

One user generates more write load than every other user combined. Worse, the
insert burst is not spread: a post fans out to 50 M rows, and at 7,000 inserts
per second of spare capacity that single post takes about two hours to deliver,
which is a product failure, not just a cost one.

This number eliminates pure push. It does not eliminate push for everyone, which
is the distinction that matters.

**The error most readers make here.** Concluding "so use pull". Pull for
everyone costs a fan-in query across 200 followees on every feed open: 120 M
opens per day times 200 lookups is 24 billion lookups per day, about 278,000 per
second, which is roughly 200x the push design's read rate. Pull fixes the tail
and breaks the median.

#### Block C. Purpose: cost the hybrid and find the threshold that defines it

Hybrid: push for normal accounts, pull for accounts above a follower threshold T.
Merge at read time.

Choose T by cost. Pushing one post to F followers costs F inserts. Pulling that
author at read time costs one extra lookup per feed open by each of the F
followers, and each follower opens 4 times a day.

- Push cost per post: F inserts.
- Pull cost per post-day: F followers times 4 opens is 4F lookups.

At first glance pull looks worse, but the pull lookup is one query against the
author's own timeline, cached and shared, whereas the push insert is F distinct
row writes to F distinct partitions. Assume a cached author-timeline read is 20
times cheaper than a partitioned insert (an assumption you should replace with a
measurement, but the direction is robust because one is a cache hit and the other
is a durable write).

- Push: F insert-units.
- Pull: 4F divided by 20, that is 0.2F insert-units, per day of posting.

So pull is cheaper per follower once the account posts often enough that its
posts are re-read many times, and push is cheaper for accounts whose posts are
read few times. The practical threshold is where the burst matters rather than
the average: set T so that a single post's fan-out completes inside the freshness
budget. At 7,000 inserts per second of headroom and a 30 second freshness target,
T is 7,000 times 30, that is 210,000 followers. Round to 100,000 for safety.

Result: accounts under 100,000 followers are pushed, which covers the
overwhelming majority of posts; accounts over it are pulled and merged at read
time. The threshold was derived from a latency budget and a measured insert rate,
not chosen because it sounded round.

!!! tip "Self-explanation prompt"

    Say what happens to T if the freshness target tightens from 30 seconds to 5
    seconds, and what that implies about which lever product owns.

**The error most readers make here.** Presenting the hybrid without a threshold.
"Push for normal users, pull for celebrities" is a shape, not a design. The
number, and where the number came from, is the answer.

#### Block D. Purpose: state the total, the unit cost, and what would change it

| Line | Derivation | Monthly |
|---|---|---|
| Feed storage | 3 TB times 3 replicas at $0.08 per GB-month | about $720 |
| Feed write compute | 7,000 peak inserts per second, assume 0.5 ms CPU each, so 3.5 vCPUs at peak, provision 12 for headroom and failover | about $350 |
| Feed read compute | 1,390 reads per second, assume 1 ms CPU, so 1.4 vCPUs, provision 6 | about $175 |
| Egress on feed responses | 120 M opens per day times 25 KB is 3 TB per day, 90 TB per month, at $0.02 through a CDN | about $1,800 |

Total is roughly $3,000 per month, or $0.0001 per active user per month. The
striking result is that egress on the response bodies dominates, so the highest
leverage cost optimisation is not the fan-out strategy at all: it is shrinking
the 25 KB feed response, for example by returning ids and letting the client
fetch cached post bodies from the CDN.

I would say exactly that in the interview, because arriving at "the thing we
spent twenty minutes arguing about is not the expensive part" is a stronger
finish than defending the hybrid.

### 13e. Faded examples

**Fade 1: cost a video transcode pipeline.** Last block blank.
Given: 10,000 uploads per day, average 8 minutes, 5 output renditions,
transcoding runs at roughly 2x realtime per vCPU (assumption, replace with a
measurement for your codec and preset).

- Block A, cost the obvious component: CPU-minutes per upload is 8 minutes times
  5 renditions divided by 2x realtime, that is 20 CPU-minutes. Times 10,000
  uploads is 200,000 CPU-minutes per day, about 3,333 CPU-hours per day, or
  100,000 per month. At $0.04 that is about $4,000 per month.
- Block B, find the input that changes the answer: the 5 renditions. Dropping to
  3 by using per-title adaptive ladders cuts transcode cost by 40% and affects
  only a minority of devices.
- Block C, cost storage and delivery: 10,000 videos per day at, assume, 300 MB
  across all renditions is 3 TB per day, 90 TB per month accumulating. Delivery
  at, assume, 20 views per video times 300 MB of one rendition is far larger than
  storage.
- Block D, total, unit cost and what would change it: **you write this block.**

??? note "Show answer"

    The unit that matters is cost per video-hour delivered, not cost per upload,
    because delivery scales with popularity and transcode does not. With
    delivery at 20 views times, assume, 150 MB of the selected rendition, that is
    3 GB per video, 30 TB per day, 900 TB per month, at CDN $0.02 per GB it is
    about $18,000 per month. Delivery is roughly 4.5 times transcode.

    What would change it: a viewing distribution shift. If views are Zipfian and
    the top 1% of videos take most traffic, then transcoding the long tail into 5
    renditions is waste. Transcode lazily, producing only the base rendition on
    upload and the higher ladders on first demand, and the transcode bill falls
    with the tail while delivery is unchanged. State the trigger: lazy transcode
    is worth it once the fraction of videos with under 5 views exceeds roughly
    half.

**Fade 2: cost an idempotency table at payments scale.** Last two blocks blank.
Given: 100 writes per second average, 300 peak, 30 day retention, 300 byte rows,
plus one index on the key.

- Block A, cost the obvious component: 100 times 86,400 is 8.64 M rows per day,
  30 days is 259 M rows, at 300 bytes is 78 GB, plus an index at roughly 30% is
  about 101 GB. With 3 replicas at $0.08 per GB-month, that is about $24 per
  month. The storage is free in any practical sense.
- Block B, find the input that changes the answer: **you write this block.**
- Block C, cost the thing that is not storage: **you write this block.**

??? note "Show answer"

    Block B. Retention is not the input that matters; the write rate against the
    unique index is. Every request performs an indexed insert before doing any
    work, so the table is on the critical path of 100% of payment traffic. If the
    rate rises 20x to 2,000 per second, you are at 172 M rows per day and the
    hot index no longer fits in memory, at which point insert latency degrades
    and the idempotency table becomes the bottleneck of the payment system.

    Block C. The real cost is latency and blast radius, not dollars. Budget it:
    an indexed insert plus commit at, assume, 2 ms adds 2 ms to every payment,
    which is fine against a 4 second gateway call. The failure cost is total: if
    this table is unavailable, no payment can start. So it inherits the payment
    database's availability target and its replication, and the honest sentence
    is that this component costs $24 a month in storage and the same on-call
    burden as the ledger.

### 13f. Check yourself

**Q1.** A colleague proposes moving from three replicas to two to save a third
of the storage bill on a 200 TB dataset. Compute the saving and give the
counter-argument in the same breath.

??? note "Show answer"

    Saving: 200 TB is 200,000 GB. One replica at $0.08 per GB-month is $16,000,
    so dropping from three to two saves about $16,000 per month.

    Counter-argument, with its own number: with three replicas and a majority
    quorum you survive one node loss with no availability impact and no data
    loss. With two, any single node loss leaves you with no redundancy, so a
    second failure during the rebuild window loses data.

    Rebuilding 200 TB at,
    assume, 1 GB per second takes about 56 hours, and that is 56 hours of
    single-copy exposure every time a node dies. The correct framing is a trade
    of $16,000 per month against a 56 hour exposure window at each failure, which
    is a business decision, not an engineering one, and it belongs to whoever
    owns the durability requirement.

**Q2.** Your design serves 1 KB JSON responses at 50,000 requests per second to
the public internet. Name the single change with the largest cost effect and
estimate it.

??? note "Show answer"

    Put a CDN in front and make the responses cacheable, even briefly. Derivation:
    50,000 rps over a 30 day month is 129.6 billion requests, at 1 KB that is
    129,600 GB, so 129.6 TB. At internet egress of $0.08 per GB that is about
    $10,368 per month. Through a CDN at $0.02 per GB it is about $2,592, and any
    cache hit rate above zero also removes origin compute.

    Second-largest change, worth naming: compression. JSON typically compresses
    well, so assume a 4x reduction, which cuts the same bill by roughly 75%
    again. The two together plausibly take a $10,000 monthly egress line under
    $1,000, which is a much larger effect than anything you can do to the
    application code.

### 13g. When not to use this

**Detailed cost modelling in an interview.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The prompt contains a cost constraint, or two candidate designs are functionally equivalent and cost is the discriminator |
| Scale threshold | If the system serves under roughly 100 requests per second, or the whole thing runs on two instances, the cost difference between any two reasonable designs is under a few hundred dollars a month and modelling it is theatre |
| Cheaper alternative | One sentence naming the dominant cost line and the lever, for example "this is egress dominated, so I would cache at the edge before I optimise anything else" |

**Reserved capacity and commitment discounts.**

| Question | Answer |
|---|---|
| Measurement that justifies it | A trailing 90 day utilisation floor that you are confident persists, and an architecture that is not about to change shape |
| Scale threshold | Below roughly a year of stable workload, or during a migration, commitments lock you into the design you are about to replace |
| Cheaper alternative | Right-size first, raise the utilisation target, and shut down non-production environments outside working hours, which typically removes a larger fraction than any discount |

**Chargeback and per-team cost attribution.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Infrastructure spend is material relative to engineering headcount cost, and at least one team's spend is growing faster than its traffic |
| Scale threshold | Under about 10 teams, a monthly report with the top 10 line items produces the same behaviour change without the tagging programme |
| Cheaper alternative | Tag the top spending resources only, publish a monthly ranked list, and let social pressure do the work |

---

## Module 14. Simplicity and scale thresholds

Interviewers report that their favourite moment is a candidate removing a box.
No vendor teaches restraint, because complexity fills pages. This module gives
you the thresholds that make restraint defensible rather than lazy.

### 14a. Pre-question

```
+--------------------------------------------------------------+
| Proposed design for an internal job board:                    |
|   React app -> API gateway -> 4 microservices                 |
|   -> Kafka -> stream processor -> Elasticsearch               |
|   -> sharded Postgres (8 shards) -> Redis cluster (6 nodes)   |
| Stated requirements:                                          |
|   500 new job posts per day                                   |
|   200,000 searches per day                                    |
|   3,000 companies, 400,000 candidate accounts                 |
|   search results may be up to 60 seconds stale                |
+--------------------------------------------------------------+
Caption: a proposed 8-component architecture next to the four
requirement numbers it is meant to satisfy.
```

**Question.** Convert the two traffic numbers into per-second rates, then say
how many of the eight components the requirements actually justify.

??? note "Show answer"

    500 posts per day is 0.006 writes per second. 200,000 searches per day is
    2.3 per second average; even a 10x daily peak concentration is 23 per second.

    A single Postgres instance handles both by four orders of magnitude, and full
    text search over 3,000 companies' worth of postings fits a built-in text
    index. The justified design is: web app, one API process, one Postgres, one
    read replica for safety. That is three components, not eight.

    Kafka, the stream processor, Elasticsearch, the 8 shards and the 6-node Redis
    cluster are all unjustified by the stated numbers. The correct interview
    behaviour is not to build the small design silently; it is to build it, then
    name the number at which each removed component becomes necessary.

### 14b. Explanation in small steps

**What one machine actually does.** These are assumptions, deliberately
conservative, and you should replace them with a measurement from your own
hardware. They exist so you can do arithmetic rather than gesture.

| Capability | Assumed capacity on one modern server | Why this is reasonable |
|---|---|---|
| Primary-key reads from a cached working set | 50,000 per second | Page-cache hits with a modern storage engine, on a machine with dozens of cores |
| Write transactions with group commit on NVMe | 5,000 to 15,000 per second | Bounded by fsync batching, not by CPU |
| Sequential scan from NVMe | 2 to 5 GB per second | Consumer NVMe read bandwidth, several drives striped |
| Working set held in RAM | up to 1 to 2 TB | Single-socket servers with a terabyte or more of memory are ordinary rentals now |
| HTTP requests handled by one application process fleet | 10,000 to 50,000 per second per few vCPUs, depending entirely on work per request | Derived from CPU per request, not a fixed number |

Two numbers to keep: 50,000 reads per second and 10,000 writes per second on a
single node. Almost every interview prompt is under one of those.

**The threshold table.** For each component, the measurement that justifies it
and the threshold below which it is over-engineering.

| Component | Add it when | Over-engineering below | Cheaper thing first |
|---|---|---|---|
| A cache | Measured p99 exceeds the SLO and profiling attributes it to repeated identical reads, or the backend is at a utilisation you cannot raise | About 5,000 reads per second on a database with a cached working set | An index, a smaller result set, a connection pool fix |
| Read replicas | Read traffic alone pushes the primary above 60% utilisation | About 20,000 reads per second | Query tuning, then a bigger primary |
| Sharding | Working set exceeds RAM on the largest instance available, or write rate exceeds what one primary can fsync | About 10,000 writes per second, or about 1 TB of hot data | Vertical scaling, archiving cold rows, partitioning within one instance |
| A queue | The write path can tolerate delay AND the producer's rate exceeds the consumer's sustained rate for meaningful periods | About 1,000 async operations per second, or bursts under 5x | A background thread with a bounded in-process queue, or a database-backed outbox table |
| A stream processor | You need windowed aggregation over a rate that a periodic batch job cannot keep up with | Under about 10,000 events per second, or when a 1 minute batch is acceptable | A cron job over a table, running every minute |
| A dedicated search index | Query patterns need relevance ranking, fuzzy matching or faceting that a database text index cannot express, at a corpus above roughly 10 M documents | Under about 1 M documents with simple keyword search | The database's built-in full text index |
| A second region | See Module 10 | Under 10% far-geography traffic, 99.9% target | Edge termination and CDN |
| A service split | Two parts have genuinely different scaling curves, deploy cadences or failure domains, and the team boundary already exists | Under about 15 engineers on the codebase | A module boundary inside one deployable |

**The interaction cost of every box you add.** N components have N times N minus
1, all over 2, pairwise interfaces that can fail. This is a combinatorial
heuristic, not a measurement, and it is honest as a heuristic:

| Components | Pairwise interactions |
|---|---|
| 3 | 3 |
| 5 | 10 |
| 8 | 28 |
| 12 | 66 |

Going from 3 boxes to 8 multiplies the interaction surface by roughly 9. Not all
pairs actually talk, but every stateful box independently adds a deploy, a
version, a backup, a restore rehearsal, a capacity model and an on-call
runbook. That list is the real cost, and reciting it is how you defend removing
a box without sounding like you cannot build one.

**Vertical before horizontal, stated as a rule.** The largest single instances
available on major clouds carry hundreds of vCPUs and a terabyte or more of
memory. Doubling an instance takes a maintenance window. Sharding takes a
quarter and is permanent. Exhaust the reversible option first.

**How to defend a single-node answer when the interviewer expects a cluster.**
The script has four beats, and the order matters:

1. State the number. "At 23 searches per second, one Postgres is at under 1% of
   its capacity."
2. State the trigger for the next step. "I would add a read replica when reads
   push the primary past 60% utilisation, which at this query mix is about
   20,000 per second, so roughly 900 times current traffic."
3. State the reversibility. "A read replica is a day of work and is reversible.
   Sharding is a quarter and is not, so it comes much later."
4. Offer the harder version explicitly. "If you would like, I can design the
   version at 100 times this traffic, which is where the shape genuinely
   changes."

Beat four is the one that converts a restraint answer from a risk into a
strength, because it proves you chose simplicity rather than defaulting to it.

**The two failure modes of restraint.** Be honest about both.

| Failure mode | What it looks like | Guard |
|---|---|---|
| Under-designing | Simple design that violates a stated non-functional requirement, for example no redundancy when the SLO needs it | Re-check the design against every non-functional requirement before you present it |
| Restraint as a hiding place | "I would keep it simple" with no numbers, used to avoid a topic you are unsure of | Every restraint claim must carry the threshold number that would change it |

### 14c. ASCII diagram

```
  TRAFFIC ---------------------------------------------->
  1/s        100/s      5k/s      20k/s     10k w/s    +

  +---------+
  | 1 app   |
  | 1 db    |
  +----+----+
       |
       +----------> +-----------+
                    | + replica |
                    | (reads    |
                    |  > 60%)   |
                    +-----+-----+
                          |
                          +--------> +------------+
                                     | + cache    |
                                     | (p99 miss) |
                                     +-----+------+
                                           |
                                           +-----> +---------+
                                                   | + shard |
                                                   | (RAM or |
                                                   |  fsync) |
                                                   +---------+

Caption: the escalation ladder. Each rung names the measurement that
promotes you to it, and nothing is added without crossing the
threshold on the axis above.
```

### 14d. Worked example: the internal job board from the pre-question

**The prompt.** "Design an internal job board." The interviewer has drawn
nothing and is waiting.

#### Block A. Purpose: convert requirements into rates before drawing anything

I write four numbers on the board before a single box:

| Requirement | Rate | What it rules out |
|---|---|---|
| 500 posts per day | 0.006 writes per second | Any write-scaling machinery: sharding, write queues, CQRS |
| 200,000 searches per day | 2.3 per second average, assume 10x peak concentration so 23 per second | A search cluster, a cache tier, read replicas for load |
| 400,000 accounts, 3,000 companies | Assume 2 KB per account and 50 KB per company with postings, so under 1 GB | Any storage distribution |
| 60 second staleness allowed | Batch is permitted | Stream processing |

Every one of those numbers eliminates something. That is the standard from the
estimation module and it applies here with more force, because the elimination
is the answer.

**The error most readers make here.** Drawing the boxes first and sizing after.
Once a Kafka box is on the whiteboard, removing it feels like a retreat, so you
defend it. Size first and the box never gets drawn.

!!! tip "Self-explanation prompt"

    Before Block B, say which of the four numbers you would ask the interviewer
    to confirm first, and why that one carries the most design risk.

#### Block B. Purpose: build the smallest design that satisfies every stated requirement

```
  +---------+     +-------------+     +----------------+
  | BROWSER |---->| APP SERVER  |---->| POSTGRES       |
  |         |     | (2 procs,   |     | primary        |
  |         |     |  1 vCPU ea) |     | + text index   |
  +---------+     +------+------+     +--------+-------+
                         |                     |
                         |                     v
                         |            +----------------+
                         |            | READ REPLICA   |
                         +----------->| (failover, not |
                                      |  load)         |
                                      +----------------+
       +-------------------------------------------+
       | OBJECT STORE: resumes and company logos    |
       +-------------------------------------------+

Caption: the smallest design meeting all four requirements. Two app
processes for redundancy, one Postgres with a text index, one replica
present for failover rather than for read capacity.
```

Justification, requirement by requirement:

- 0.006 writes per second against roughly 10,000 available: 0.00006% utilised.
- 23 searches per second against roughly 50,000 cached reads: under 0.05%.
- Under 1 GB of data on a machine with hundreds of gigabytes of RAM: the entire
  dataset is resident, so every query is a memory hit.
- Full text search over a corpus this small is a built-in index, and the 60
  second staleness allowance is not even needed, because the index is
  synchronous.

The replica is present for a reason I state explicitly: not for read load, which
is negligible, but so that a lost primary is a promotion rather than a restore
from backup. That is a redundancy requirement, not a scale one, and separating
those two justifications is the point of the block.

**The error most readers make here.** Adding the replica and then claiming it
helps with read scale. It does not here, and claiming it does tells the
interviewer you attach reasons to components after choosing them.

#### Block C. Purpose: name the trigger for every component I did not add

This is the block that converts restraint into expertise. For each thing the
interviewer might have expected:

| Not added | Trigger that would add it | Distance from today |
|---|---|---|
| Redis cache | Primary CPU above 60% attributable to repeated reads, roughly 5,000 reads per second at this query mix | about 200x |
| Read replicas for load | Reads above roughly 20,000 per second | about 900x |
| Elasticsearch | Corpus above roughly 1 M documents, or a requirement for faceting and relevance tuning the text index cannot express | about 30x on corpus, or a product decision at any size |
| Kafka | More than roughly 1,000 asynchronous events per second, or a second consumer that needs replay | about 100x |
| Sharding | Working set above RAM (roughly 1 TB) or writes above roughly 10,000 per second | more than 1,000x |
| Service split | The codebase exceeds roughly 15 engineers, or two parts diverge in deploy cadence | organisational, not technical |

Notice that one trigger, Elasticsearch, is a product decision rather than a scale
one. Saying so is important: not every component is justified by traffic, and
pretending otherwise is its own kind of dishonesty.

!!! tip "Self-explanation prompt"

    Pick the Kafka row and say what you would build instead if you needed
    asynchronous work tomorrow at 5 events per second.

**The error most readers make here.** Listing what was not added without the
trigger numbers. "We do not need Kafka yet" is an opinion. "We need Kafka above
roughly 1,000 events per second and we are at 5" is an engineering position.

#### Block D. Purpose: check the simple design against every non-functional requirement

This is the guard against under-designing, and I run it out loud:

| Non-functional requirement | Does the design meet it | Evidence |
|---|---|---|
| Availability | Yes, if the target is 99.9% | Two app processes, replica for failover, and a 43 minute monthly budget accommodates a promotion |
| Durability | Yes | Replica plus point-in-time backups; I state the RPO as the backup interval |
| Latency | Yes | All queries hit resident memory; the dominant cost is network, not the database |
| Security | Needs work the traffic numbers do not reveal | Resumes are personal data, so the object store is private with signed URLs and a short expiry |
| Operability | Yes | Three components means three runbooks; the SLI is search success and post success |

The security row is deliberately the one that fails. Small designs are not
automatically complete designs, and finding the requirement your simplicity did
not address is what stops this module becoming an excuse.

### 14e. Faded examples

**Fade 1: an internal analytics dashboard.** Last block blank.
Given: 40 internal users, 6 dashboards, each running 12 queries, refreshed every
5 minutes during business hours, over a 2 TB event table.

- Block A, convert to rates: 40 users times 6 dashboards times 12 queries every
  300 seconds is at most 9.6 queries per second, and in practice far less
  because users do not all watch all dashboards. The 2 TB table is the only
  interesting number.
- Block B, smallest design: a columnar store or a partitioned table with
  pre-aggregated rollups computed by a 5 minute cron. No streaming, no cache, no
  separate serving layer, because the refresh cadence already permits batch.
- Block C, triggers for what was not added: streaming if the freshness
  requirement drops under about 1 minute; a serving cache if concurrent query
  count pushes the warehouse past its slot limit; a separate warehouse if
  analytics queries start affecting production.
- Block D, check against non-functional requirements: **you write this block.**

??? note "Show answer"

    Availability: internal dashboards at business hours only, so a 99.5% target
    is honest and a single node with backups meets it. Say the number rather than
    silently under-designing.

    Durability: the rollups are derived, so losing them costs one cron run, not
    data. The source event table is the thing that needs backups, and its RPO
    should match the upstream retention.

    Latency: 5 minute refresh means a 10 second query is acceptable, which is
    what makes the batch rollup design viable at 2 TB.

    Security and privacy: this is the row that fails again. Event tables usually
    carry user identifiers, so access control per dashboard and column-level
    masking for personal fields are required, and neither is implied by any
    traffic number. Add a retention policy too, because keeping raw events
    forever is both a cost and a compliance exposure.

**Fade 2: a scheduling service for internal meeting rooms.** Last two blocks
blank.
Given: 5,000 employees, 300 rooms, roughly 4,000 bookings per day, availability
checks roughly 40,000 per day.

- Block A, convert to rates: 4,000 bookings per day is 0.05 writes per second;
  40,000 checks per day is 0.5 reads per second, with a peak near the top of each
  hour, so assume 20x concentration giving 10 per second. 300 rooms times a year
  of slots is a small table.
- Block B, smallest design: **you write this block.**
- Block C, triggers for what was not added: **you write this block.**

??? note "Show answer"

    Block B. One application process pair and one Postgres. The interesting part
    is not scale, it is the double-booking invariant: enforce it with an
    exclusion constraint on (room_id, time range) so the database rejects
    overlapping bookings, rather than with a lock or a read-then-write check.
    That is the same lesson as Module 9, arriving from a different direction: the
    constraint is the coordination mechanism.

    Block C. A cache: never at 10 reads per second; the trigger would be roughly
    5,000 reads per second. A queue: only if booking triggers slow side effects
    such as calendar invitations and video links, in which case an outbox table
    polled every second is sufficient at 0.05 writes per second, and a real
    broker is warranted above roughly 1,000 events per second.

    Sharding: never,
    since the entire dataset is measured in megabytes. Distributed locking: never,
    because the exclusion constraint already provides the guarantee, and adding a
    lock would create a second, weaker source of truth.

### 14f. Check yourself

**Q1.** An interviewer says "assume this needs to handle a million users". What
do you ask before changing your design, and why does the question matter more
than the number?

??? note "Show answer"

    Ask for the rate, not the population. A million registered users tells you
    nothing; a million users with 10% daily active and 20 actions per day is
    2 M actions per day, which is 23 per second average and perhaps 200 per second
    at peak. That is still one database.

    The question matters because population and rate differ by three or four
    orders of magnitude, and every component threshold in this module is defined
    on rate or on working-set size. A candidate who redesigns on hearing "a
    million users" has revealed that they scale on vocabulary rather than on
    arithmetic.

**Q2.** You have proposed a single-node design. The interviewer looks
unconvinced and says "what if it goes down?" Give the answer that neither
capitulates nor dismisses.

??? note "Show answer"

    Separate the two requirements the question conflates. Redundancy is not
    scale: I already have a standby replica and a documented promotion, so a node
    loss costs an RTO I can state, for example 2 to 5 minutes with automated
    promotion. That answers availability without changing the topology.

    Then state the budget: if the target is 99.9%, that is 43 minutes a month and
    a 5 minute promotion fits comfortably. If the target is 99.99%, that is 4.3
    minutes and my promotion does not fit, so I would move to a managed
    multi-availability-zone cluster with automatic failover. Offering the specific
    number at which your answer changes is what converts the exchange from a
    defence into a design conversation.

### 14g. When not to use this

**Restraint as a default posture.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You have converted every stated requirement into a rate or a size, and the simple design clears each one with at least a 10x margin |
| Scale threshold above which restraint is the wrong answer | Once any single component is above roughly 50% of its assumed single-node capacity at current traffic, the simple design has no headroom for growth or for a bad day, and arguing for it is under-designing |
| Cheaper alternative to a full simple design | If you have not sized anything, do not claim simplicity. Say "I need one number before I can tell you whether this is one box or twenty" and ask for it |

**The single-node capacity assumptions in this module.**

| Question | Answer |
|---|---|
| Measurement that justifies using them | You are doing order-of-magnitude elimination in an interview and no measured figure is available |
| Threshold below which they are misleading | Any decision with real money or a real migration attached. These numbers vary by more than 10x with row size, index count, query shape and hardware |
| Cheaper alternative | A 30 minute load test against a representative dataset, which produces a number specific to your workload and settles the argument permanently |

**Removing a component from a running system just because it is under threshold.**

| Question | Answer |
|---|---|
| Measurement that justifies removal | The component's cost (money plus on-call plus deploy friction) exceeds its benefit, and you can show current traffic is under the trigger by a wide margin with no forecast crossing it |
| Threshold below which removal is not worth it | If the component is stable, cheap and already operated, removing it has a migration cost and a risk, and greenfield thresholds do not apply to brownfield systems. See Module 15 |
| Cheaper alternative | Freeze it: stop adding dependencies on it, document the trigger for removal, and revisit when it next demands work |

---

## Module 15. Brownfield design

Every paid case study starts from a blank page. Several real staff-plus rounds
hand you a running system with users on it and ask what you would do next. The
skills barely overlap: greenfield rewards the best end state, brownfield rewards
the safest ordered path to it.

### 15a. Pre-question

```
+--------------------------------------------------------------+
| Running system, 3 years old:                                  |
|   one Postgres primary, 4 TB, 8,000 writes/s peak             |
|   one table, events, is 3.1 TB of the 4 TB                    |
|   p99 write latency rose from 12 ms to 210 ms over 6 months   |
|   replica lag peaks at 45 s during the nightly batch          |
|   14 services write to this database directly                 |
|   no downtime window; the product is used 24/7 globally       |
+--------------------------------------------------------------+
Caption: a running single-database system with its size, write rate,
latency trend, replica lag and the number of direct writers.
```

**Question.** Which single fact in that box most constrains the plan, and what
does it rule out?

??? note "Show answer"

    The 14 direct writers. Every other fact describes a load problem with known
    remedies; that one describes a coupling problem that makes any remedy slow.
    You cannot change the schema, move a table or introduce a new store without
    coordinating 14 deployables owned by an unknown number of teams.

    It rules out any plan whose first step is a schema change or a data move. The
    first step has to be to interpose a layer you control, so that later steps
    touch one codebase instead of fourteen. It also reframes the estimate: the
    work is bounded by organisational coordination, not by bytes, which is
    exactly the kind of observation a staff-level round is testing for.

### 15b. Explanation in small steps

**The ordering rule that governs everything.** Make the new system correct
before it is authoritative, and make it authoritative before it is exclusive.
Every migration below is that rule expanded.

**The migration ladder.** Six rungs, in order. Skipping a rung is where outages
come from.

| Rung | What you do | What it proves | Reversible |
|---|---|---|---|
| 1. Shadow read | Send a copy of every read to the new system, discard the result, compare asynchronously | The new system returns the same answers | Fully, it serves nobody |
| 2. Dual write | Write to old and new, old remains authoritative | The new system can absorb production write load and stay consistent | Fully, stop writing to new |
| 3. Backfill | Copy historical data into the new system while dual write continues | The new system has complete data | Fully, delete and retry |
| 4. Cutover reads | Serve a growing percentage of reads from the new system | Users are unaffected by the new read path | Yes, flip the percentage back |
| 5. Cutover writes | New system becomes authoritative; old is written for safety only | The new system is the source of truth | Costly but possible while the old path still receives writes |
| 6. Decommission | Stop writing the old path, freeze, then delete | Done | No |

**Dual write is not atomic, and this is the fact candidates miss.** Writing to
two systems in one request is two independent failures. Three honest options:

| Option | Guarantee | Cost | When to pick |
|---|---|---|---|
| Write both in the request, log divergence | None. Either can fail | Lowest | Never for authoritative data; acceptable for a shadow that a reconciler repairs |
| Transactional outbox: write the row and an outbox record in one local transaction, a relay ships the outbox to the new system | At-least-once delivery, ordered per key | One table, one relay process | The default, and the answer to give |
| Change data capture from the old system's replication log | At-least-once, no application change, captures the 14 writers you cannot modify | A CDC pipeline to run | When you cannot change every writer, which is exactly the pre-question scenario |

The outbox works when you own the writers. CDC works when you do not. Naming
which constraint you are under, and picking accordingly, is the senior move.

**Backfill arithmetic.** Backfill is bounded by the write capacity you are
willing to spend on it, not by how fast you can read.

- Suppose the target can absorb 100,000 writes per second and you allocate 20%
  to backfill, so 20,000 per second.
- 6 billion rows divided by 20,000 is 300,000 seconds, that is 3.5 days of
  continuous running.
- If you may only backfill in a 6 hour nightly window: 6 times 3,600 times
  20,000 is 432 M rows per night, so about 14 nights.

Two rules that fall out. First, backfill in key order with a checkpoint so a
crash resumes rather than restarts. Second, the backfill must never overwrite a
newer live write: either write with an "insert if not exists" semantic, or
compare versions, or backfill only rows whose id is below the watermark you
recorded when dual write started.

**Shadow traffic and the acceptance gate.** Shadow comparison without a gate is
a science project. Define the gate before you start.

| Gate element | Example value | Why |
|---|---|---|
| Sample size | 10 M compared requests | Enough to see a 1 in 100,000 defect at least a hundred times |
| Mismatch ceiling | Under 0.01% unexplained | Some mismatch is legitimate (timestamps, ordering of equal-scored results) |
| Explained mismatch requirement | Every mismatch class categorised and either fixed or accepted in writing | An uncategorised 0.005% is a defect you have not found yet |
| Latency requirement | New path p99 within 110% of old | A correct but slower system is not a migration, it is a regression |
| Duration | At least one full weekly cycle | Weekend traffic has a different shape and finds different bugs |

**Blast radius, made explicit.** For every step, write down three numbers before
you take it.

| Dimension | Question | Example |
|---|---|---|
| Traffic fraction | What share of requests can this break | 1% canary |
| Population | Who exactly is exposed | Internal users, then one small region, then everyone |
| Reversibility window | How long until this cannot be undone | Reversible until the old path stops receiving writes |

The third one is the one people forget, and it is the reason cutover of writes
is scheduled for a Tuesday morning and not a Friday evening.

**Deprecation, and the one-client problem.** Removing the old path is the step
that never finishes. What actually works:

1. Instrument the old path per caller, so you know exactly who is left.
2. Publish the removal date and the migration guide at the same time.
3. Introduce scheduled brownouts: return errors for 5 minutes at a known time,
   then 30 minutes, then an hour. A brownout finds the callers who ignored the
   email and produces a list, not a surprise.
4. When one caller remains, go and do their migration for them. It is cheaper
   than another quarter of carrying the old path.

**Buying time is a legitimate first move.** Interviewers reward a candidate who
separates the cheap reversible relief from the expensive terminal change.

| Relief | Typical effort | Typical headroom bought |
|---|---|---|
| Add the missing index, fix the N+1 query | Days | Often 2 to 10x on the affected path |
| Vertical scale the primary | A maintenance window | 2 to 4x |
| Move the heaviest read traffic to a replica | Days | Removes read load entirely from the primary |
| Archive or partition out cold rows | Weeks | Shrinks the working set, sometimes dramatically |
| Batch or debounce a chatty writer | Days | Directly proportional to the batching factor |

### 15c. ASCII diagram

```
  STEP 1 SHADOW          STEP 2 DUAL WRITE      STEP 3 BACKFILL
  +--------+             +--------+             +--------+
  | CLIENT |             | CLIENT |             | CLIENT |
  +---+----+             +---+----+             +---+----+
      |                      |                      |
      v                      v                      v
  +--------+  copy    +--------------+       +--------------+
  | OLD    |------>   | OLD (auth.)  |       | OLD (auth.)  |
  | (auth.)|  compare |   + outbox   |       |   + outbox   |
  +--------+   async  +------+-------+       +------+-------+
      |                      | relay                | relay
      v                      v                      v
  +--------+             +--------+     +---------------------+
  | NEW    |             | NEW    |<----| BACKFILL, id < mark |
  | (dark) |             | (dark) |     | 20k rows/s, resumes |
  +--------+             +--------+     +---------------------+

  STEP 4 READ CUTOVER: 1% -> 10% -> 50% -> 100%, flag-controlled
  STEP 5 WRITE CUTOVER: new authoritative, old still written
  STEP 6 DECOMMISSION: brownouts, then freeze, then delete

Caption: the migration ladder drawn as three states plus three
scripted steps. The old system stays authoritative through step 4, so
everything up to that point is reversible by a flag.
```

### 15d. Worked example: the write hot spot, next two quarters

**The prompt.** "Here is the system in the pre-question box. You have two
quarters. What do you do?"

#### Block A. Purpose: measure before moving, and refuse to plan without the breakdown

I do not accept "the database is slow" as a diagnosis. Three measurements first,
and I name what each would change:

| Measurement | If it shows A | If it shows B |
|---|---|---|
| Write latency split by table and by statement | One table dominates, so the fix is local | Spread evenly, so the fix is the instance or the storage |
| Time in commit versus time in lock waits versus time in index maintenance | Index maintenance dominates, so drop or defer indexes | Lock waits dominate, so the fix is a hot row or a long transaction |
| Write rate per calling service | One service is 70% of writes, so fix one caller | Even spread, so it is genuinely aggregate load |

Suppose the answers come back: the events table is 78% of write time, index
maintenance is over half of that (there are 6 indexes on events), and one
telemetry service is 62% of the write rate.

That reshapes the whole plan. This is not a "we outgrew Postgres" problem; it is
a "one service is writing telemetry into our transactional database with six
indexes on it" problem.

**The error most readers make here.** Jumping straight to sharding or to a new
data store. Sharding a table whose load comes 62% from one misplaced caller
carries all the cost and fixes the wrong thing.

!!! tip "Self-explanation prompt"

    Before Block B, say what you would do differently if the third measurement
    had come back evenly spread across all 14 services.

#### Block B. Purpose: buy time with the cheapest reversible change

Quarter one, first six weeks. Nothing here is a migration.

| Action | Derivation of the expected effect | Reversible |
|---|---|---|
| Drop 3 of the 6 indexes on events after confirming no query uses them | Index maintenance is over half of 78% of write time; removing half the indexes plausibly removes a quarter of total write time | Yes, recreate them |
| Batch the telemetry service's writes from 1 row per event to 500 per statement | 62% of 8,000 writes per second is about 5,000; batching 500 to 1 makes that about 10 statements per second, so total statement rate falls from 8,000 to about 3,010 | Yes, flag it off |
| Move the nightly batch's reads to the replica and give it its own replica | Replica lag peaks at 45 s during the batch, which is the batch competing with replication apply | Yes |

Expected result, stated as a prediction I can be held to: write p99 returns
toward the 12 ms baseline and the system has roughly 2.5x headroom on statement
rate. That buys the time to do the real work carefully instead of under
pressure.

**The error most readers make here.** Treating quick wins as beneath a design
answer. In a staff round, sequencing the reversible relief before the irreversible
migration is the answer, and skipping it reads as an inability to prioritise.

#### Block C. Purpose: design the terminal state and pick the migration mechanism

Terminal state: events leaves the transactional database entirely and lands in a
store designed for append-heavy, time-ordered data with a retention policy. The
transactional database keeps the roughly 0.9 TB of actual business entities and
returns to being small.

The mechanism is decided by one constraint from the pre-question: 14 direct
writers, which I do not control.

| Candidate mechanism | Verdict |
|---|---|
| Transactional outbox in each writer | Rejected: 14 codebases, unknown owners, a quarter of coordination before the first row moves |
| Change data capture from the Postgres write-ahead log | Chosen: no writer changes, captures all 14, ordered per key, and it is a component I own end to end |
| Application-level dual write in a new shared library | Rejected: still requires 14 deploys, and version skew means partial coverage for months |

CDC is the answer here specifically because of the coupling fact, and I say the
causal link out loud, since the same problem with 2 writers I own would get the
outbox.

Backfill sizing, using the module's method: 3.1 TB of events at, assume, 400
bytes per row is about 7.75 billion rows. If the new store absorbs 100,000
writes per second and I allocate 20% to backfill, that is 20,000 per second, so
about 387,500 seconds, roughly 4.5 days of continuous backfill. I record the
maximum event id at the moment CDC starts and backfill only below it, so live
CDC writes and the backfill can never fight.

!!! tip "Self-explanation prompt"

    Say why recording a watermark id at CDC start is safer than comparing
    timestamps to decide which rows the backfill should skip.

**The error most readers make here.** Choosing the mechanism from a preference
about elegance rather than from the constraint. The outbox is the better pattern
in the abstract and the wrong one here.

#### Block D. Purpose: sequence the two quarters with explicit gates

| Weeks | Step | Gate to proceed | Rollback |
|---|---|---|---|
| 1 to 6 | Relief work from Block B | Write p99 under 30 ms, statement rate under 4,000 per second | Recreate indexes, disable batching flag |
| 7 to 10 | Stand up the new store; CDC to it; shadow-read comparison on the 5 heaviest queries | 10 M compared reads, under 0.01% unexplained mismatch, over one full week | Turn off CDC, delete the store |
| 11 to 17 | Backfill below the watermark, resumable, throttled to 20% of target capacity | Row counts and checksums match per day-partition | Delete and rerun |
| 18 to 20 | Read cutover behind a flag: 1%, 10%, 50%, 100%, each step held for 48 hours | SLO burn rate unchanged at each step | Flip the flag back, instantly |
| 21 to 24 | Write cutover: new store authoritative, Postgres still receiving CDC-derived writes for one month | One week at 100% with no incident | Repoint writes; this is the last reversible point |
| 25 to 26 | Deprecate: per-caller instrumentation, published date, brownouts at 5 min, 30 min, 1 hour | Zero callers for two weeks | None after deletion, so a final export goes to cold storage first |

Two things I flag as risks with owners, because a plan without named risks is a
wish:

- The events table's schema is used by ad hoc analytics queries nobody has
  inventoried. Mitigation: instrument reads for four weeks before week 18, and
  publish the caller list.
- The 4.5 day backfill will not run cleanly on the first attempt. Mitigation: the
  checkpointed, resumable design, plus a rehearsal on one day-partition in week
  10 rather than discovering the problems in week 11.

Finally, what I am explicitly not doing in these two quarters: not sharding, not
splitting the monolith, not introducing a second region. Each is defensible
later, none is on the critical path of the stated problem, and saying which
attractive work you are declining is a staff-level behaviour.

### 15e. Faded examples

**Fade 1: move sessions from sticky in-memory state to a shared store.** Last
block blank.

- Block A, measure first: what fraction of requests currently benefit from
  stickiness, what is the session size distribution, and how often does a node
  restart evict live sessions. Suppose sessions average 4 KB, there are 2 M
  concurrent, and node restarts log out roughly 30,000 users per deploy.
- Block B, buy time cheaply: reduce deploy-time impact with connection draining
  and a rolling deploy, which cuts the 30,000 without changing architecture.
- Block C, terminal state and mechanism: a signed self-contained token with a
  small replicated revocation list, or a shared store holding 2 M times 4 KB,
  which is 8 GB and fits in memory on one node pair. Dual-read from both during
  transition: check the shared store first, fall back to the local session, and
  write only to the shared store.
- Block D, sequence with gates: **you write this block.**

??? note "Show answer"

    Week 1 to 2: deploy the shared store and write sessions to both places, with
    reads still local. Gate: store write success above 99.99% and added p99 under
    2 ms.

    Week 3: read from the shared store with a fallback to local, behind a
    per-request flag at 1%, then 10%, then 100%, holding 24 hours at each. Gate:
    no rise in re-authentication rate, which is the SLI that would show a lost
    session.

    Week 4: turn off local session storage and remove stickiness from the load
    balancer. Gate: one week at 100% with no fallback hits, proven by a counter
    on the fallback path, which is the specific instrumentation that makes this
    step safe. Rollback: re-enable stickiness, which is a load-balancer setting
    and takes seconds, so this migration stays reversible until the local session
    code is deleted, which happens a month later.

**Fade 2: split a shared orders table that three teams write to.** Last two
blocks blank.

- Block A, measure first: per-team write rate and query patterns, which columns
  each team actually reads, and how many queries join across the columns the
  teams would like to separate. Suppose 40% of queries join across the proposed
  boundary, which is the number that decides whether the split is viable at all.
- Block B, buy time cheaply: add the missing composite indexes and move reporting
  reads to a replica, buying enough headroom to plan properly.
- Block C, terminal state and mechanism: **you write this block.**
- Block D, sequence with gates: **you write this block.**

??? note "Show answer"

    Block C. With 40% of queries crossing the boundary, a physical split into
    separate databases converts those into cross-service calls with no
    transaction, which is a large product risk.

    The honest terminal state is
    therefore not a split into three databases. It is: one database, three
    schemas, with each team's writes going through a service interface it owns,
    and the cross-boundary queries replaced by explicit read APIs. That preserves
    the ability to split physically later, once the join fraction is driven down,
    and it makes the join fraction visible as a metric to drive down.

    Block D. Weeks 1 to 4: introduce the three service interfaces with the
    existing table behind them, and route each team's writes through its own
    interface. Gate: zero direct writes remaining, proven by a per-caller counter
    at the database.

    Weeks 5 to 10: replace cross-boundary queries with read
    APIs, one at a time, measuring the join fraction weekly. Gate: join fraction
    under 5%. Only then is a physical split a reasonable next proposal, and it
    is a separate decision with its own justification, not a foregone conclusion.
    The rollback at every step is a routing flag, because the data never moved.

### 15f. Check yourself

**Q1.** Why is dual write from the application usually worse than change data
capture, and name the one situation where the reverse is true.

??? note "Show answer"

    Application dual write performs two independent operations with no shared
    transaction, so any crash or error between them leaves the two systems
    divergent, and the divergence is silent unless you build a reconciler anyway.
    It also requires changing every writer, which is slow when writers are owned
    by other teams. CDC reads the source system's own replication log, so it
    inherits the source's ordering and durability, needs no writer changes, and
    captures writers you did not know about.

    The reverse is true when the new system needs data the old system does not
    store. CDC can only ship what was written. If the target needs an enriched
    event (a computed score, a joined attribute, a field the old schema lacks),
    the application knows it and the log does not, so a transactional outbox
    carrying the enriched record is the right mechanism.

**Q2.** You are at week 20 of the plan above, 100% of reads on the new store, and
a defect appears that has been silently corrupting 0.3% of rows since the
backfill. What do you do, and what does the plan's design give you?

??? note "Show answer"

    The plan gives you two things: Postgres is still authoritative for writes
    until week 21, and the backfill was checkpointed and resumable below a
    recorded watermark. So the recovery is to flip the read flag back to the old
    path (seconds, no data loss, users unaffected), fix the defect, and re-run
    the backfill for the affected key range only, using the checkpoints to bound
    the rerun rather than repeating 4.5 days.

    The general lesson worth stating: the value of keeping the old system
    authoritative through the read cutover is not that you expect a rollback, it
    is that it converts a data-corruption incident into a configuration change.
    That is why rung 5 is separate from rung 4 and why nobody should do them in
    the same week.

### 15g. When not to use this

**A full migration ladder.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The system serves live users with an availability target you would breach by taking downtime, and the data set is large enough that a copy cannot complete inside any window you could negotiate |
| Scale threshold below which it is over-engineering | Under roughly 100 GB and with a negotiable maintenance window, an offline copy plus a verify plus a cutover takes an evening and costs a fraction of a six-rung migration |
| Cheaper alternative | Announce a two hour window, stop writes, copy, verify checksums, switch, keep the old data for a month |

**Change data capture.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You cannot change all the writers, or there are more than about five of them across team boundaries, or you need historical ordering the application cannot reproduce |
| Scale threshold | With one or two writers you own, a transactional outbox is simpler, has no extra infrastructure, and carries richer records |
| Cheaper alternative | Transactional outbox, or for low-rate data a periodic query on an updated_at column with a watermark, which needs nothing but a cron |

**Shadow traffic comparison.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The new implementation is a rewrite rather than a re-host, so behavioural equivalence is genuinely uncertain, and the cost of a wrong answer reaching a user is high |
| Scale threshold | If the new system runs the same code against the same schema (a re-host or a version upgrade), shadow comparison finds nothing and costs a doubling of load. Skip it and canary instead |
| Cheaper alternative | A canary at 1% with automatic rollback on SLO burn, plus a replay of a recorded request corpus in a test environment |

---

## Module 16. Level calibration

The single most requested thing candidates ask for, and the thing paid courses
do not supply, is what a staff answer adds. This module answers it as observable
behaviour rather than as vocabulary, then makes you produce the three versions
yourself.

### 16a. Pre-question

```
+--------------------------------------------------------------+
| Three answers to "how do we keep the feed fast?"              |
|                                                               |
| A: "Add a Redis cache in front of the feed query, with a      |
|     5 minute TTL, and shard by user id."                      |
|                                                               |
| B: "Feed p99 is 900 ms and 700 ms of that is the fan-in       |
|     query. A cache fixes it only if the hit rate exceeds      |
|     about 85%, which needs a 5 minute TTL. That makes the     |
|     feed 5 minutes stale, so I would confirm the freshness    |
|     requirement before choosing between the cache and         |
|     precomputing the feed on write."                          |
|                                                               |
| C: "Feed latency has three causes and only one is worth       |
|     fixing this half. I would spend the quarter on the        |
|     fan-in query, defer the cache, and put a freshness SLO    |
|     in place so the next person has a number to design        |
|     against instead of an opinion."                           |
+--------------------------------------------------------------+
Caption: three answers to the same question, differing in what they
do with the measurement rather than in which components they name.
```

**Question.** Rank A, B and C by level, and name the specific behaviour that
separates each from the one below it.

??? note "Show answer"

    A is mid, B is senior, C is staff, and the separator is not the components.
    All three could end up building the same cache.

    A to B: B measured, and used the measurement to derive a condition (85% hit
    rate) which exposed a trade-off (5 minutes of staleness) which produced a
    question for the requirement owner. A asserted a solution with no number
    behind it.

    B to C: C chose what not to do and made the decision durable. It ranked three
    causes, allocated a quarter, deferred a defensible option explicitly, and
    left behind an artefact (a freshness SLO) that changes how future decisions
    get made. C is answering "what should this team do" while B is answering
    "what is the right design".

    The trap: C without B underneath it is not a staff answer, it is hand-waving.
    Staff answers contain the senior answer and then add to it.

### 16b. Explanation in small steps

**What levels are not.** They are not vocabulary, not the number of components,
and not the obscurity of the technology named. A candidate who says "Raft" and
cannot derive the write latency has shown less than one who says "one primary
with failover" and states the RTO.

**The behaviour table.** Read this as a ladder where each column contains the
one before it.

| Dimension | Mid | Senior | Staff |
|---|---|---|---|
| Requirements | Works correctly from the requirements given | Asks the questions that change the design, and shows the fork | Challenges whether the stated problem is the real one, and reframes with evidence |
| Numbers | Estimates when asked | Every number eliminates an option | Names which number the whole decision hinges on, and says how to get it |
| Design | Produces a correct design | Chooses between designs and names the deciding measurement | Designs the sequence of designs over time, and what triggers each transition |
| Trade-offs | Names trade-offs when prompted | Volunteers them, with the threshold that flips each | Prices them in money, time and team capacity |
| Failure | Handles the failures asked about | Names failure classes unprompted and their prevention | Designs the operational contract: SLOs, blast radius, rollback, who is paged |
| Scope | Answers the question | Manages the clock and descopes deliberately | Says what will not be done this year and why, and who owns the risk |
| Uncertainty | Says "I do not know" | Says "I do not know, here is how I would find out" | Says "we cannot know this yet, here is the cheapest experiment and the decision it unblocks" |
| Organisation | Not expected | Notices when a design implies a team boundary | Treats team boundaries, migration cost and on-call load as design inputs |

**Two failure modes, symmetric and equally costly.**

| Failure mode | What it sounds like | Why it fails |
|---|---|---|
| Overshooting | A candidate interviewing at mid who talks about org design and quarterly sequencing but cannot size the database | The interviewer concludes you have read about the job rather than done it. Depth in one place beats breadth everywhere |
| Undershooting | A strong senior candidate who answers the literal prompt perfectly and never volunteers a trade-off | The interviewer has no evidence for the higher level, and calibrates down to what they observed |

The rule: perform your level plus one, and only in the areas where you actually
have depth. Signal the higher level by volunteering a trade-off and a threshold,
not by widening the topic list.

**The observable proxies you can self-score.** Record yourself, then count.

| Proxy | Mid range | Senior range | Staff range |
|---|---|---|---|
| Sentences containing a number that eliminates an option | 1 to 3 | 4 to 8 | 4 to 8, plus at least one about cost or time |
| Unprompted trade-offs, each with a threshold | 0 to 1 | 3 or more | 3 or more, at least one priced |
| Times you said what you were not doing and why | 0 | 1 to 2 | 3 or more, including one attractive option declined |
| Clarifying questions that visibly forked the design | 1 to 2 | 3 to 5 | 3 to 5, at least one challenging the premise |
| Depth dives you drove yourself, not the interviewer | 0 to 1 | 1 to 2 | 2, one of them operational rather than architectural |

These ranges are a self-calibration heuristic derived from the behaviour table
above, not measured data from real loops. Use them to notice a pattern in your
own recordings, not as a score.

**What the level delta is not allowed to be.** Adding boxes. If your staff
version of an answer has more components than your senior version, you have
probably confused seniority with complexity. In practice, staff answers often
have fewer components and more constraints.

### 16c. ASCII diagram

```
                        +-------------------------+
   STAFF                | sequence over time,     |
                        | cost, org, what NOT to  |
                        | do, decision artefacts  |
                        +-----------+-------------+
                                    | contains
                        +-----------v-------------+
   SENIOR               | chooses between designs |
                        | volunteers trade-offs   |
                        | numbers that eliminate  |
                        +-----------+-------------+
                                    | contains
                        +-----------v-------------+
   MID                  | correct design from     |
                        | given requirements      |
                        +-------------------------+

   FAILURE: reaching the top box without the two below it reads as
   hand-waving, and is scored lower than a clean MID answer.

Caption: levels as containment, not as separate skills. Each level
includes everything below it, and skipping a layer is penalised.
```

### 16d. Worked example: "design a notification service", three times

**The prompt.** "Design a service that sends notifications to our users across
push, email and SMS." Given: 20 M users, 50 M notifications per day, three
channels, and a stated requirement that a user must never receive the same
notification twice.

#### Block A. Purpose: fix the level signature before answering

I decide, before speaking, which behaviours I intend to show. For a senior
target that means at least three unprompted trade-offs with thresholds, at least
four eliminating numbers, and one deliberate descope. Naming the target to
myself stops the answer drifting into a topic tour.

The base arithmetic, which every version shares:

- 50 M per day is 579 per second average. Assume a 5x peak concentration because
  notification traffic follows waking hours, giving about 2,900 per second.
- 2,900 per second is comfortably inside one queue and a modest worker fleet, so
  scale is not the interesting part of this problem. Deduplication and channel
  failure are.

That last sentence is the hinge, and every level uses it differently.

**The error most readers make here.** Treating 50 M per day as a big number. It
is 579 per second. Candidates who fail to convert spend the round designing for
scale that does not exist and never reach the actual difficulty.

#### Block B. Purpose: write the mid answer, and mark its ceiling

> "Notifications are produced by services publishing events. I would put them on
> a queue, with a worker fleet consuming and dispatching to a channel adapter for
> push, email and SMS. Each notification gets an id, and I store delivered ids in
> a table with a unique constraint so a retry cannot deliver twice.
>
> User channel preferences live in a preferences table the worker reads. Failed
> sends are retried with exponential backoff, and after five failures they go to
> a dead letter queue. I would shard the queue by user id so ordering per user is
> preserved."

This is a correct design. It satisfies the stated requirements, it names the
deduplication mechanism, and the components are appropriate. Nothing in it is
wrong.

Its ceiling: every number in the prompt was accepted without being used. It does
not say why five retries rather than three, what the per-channel failure profile
is, whether ordering per user is actually required, or what happens when the SMS
provider is down for an hour. It answers the question asked.

!!! tip "Self-explanation prompt"

    Before Block C, pick the single sentence in the mid answer that a senior
    version would attack first, and say what number it needs.

#### Block C. Purpose: add the senior delta, and mark its ceiling

The senior version keeps everything above and adds four things. I mark each with
the behaviour it demonstrates.

> "At 579 per second average and 2,900 at peak, this is one queue and roughly 20
> workers, so I will not spend time on scaling and will spend it on delivery
> semantics instead. [eliminating number, deliberate descope]
>
> The three channels have different failure profiles, and that changes the
> design. Push is cheap and fails fast; SMS costs real money per message and its
> providers have minute-scale outages; email has delayed bounces that arrive
> hours later. So I would not have one worker pool: I would have one per channel,
> with its own concurrency limit and its own retry budget, so an SMS provider
> outage cannot consume the workers that deliver push. [unprompted trade-off]
>
> On the never-twice requirement, the unique constraint stops our duplicates but
> not the provider's. If we time out on an SMS send, we do not know whether it
> went.
>
> I would pass our notification id to the provider as its idempotency key
> where supported, and where it is not, decide per channel: for push, resend,
> because a duplicate is nearly free; for SMS, do not resend, because a duplicate
> costs money and annoys the user more than a miss. That is a product decision
> and I would confirm it. [trade-off with a threshold, and a question for the
> requirement owner]
>
> Ordering per user is worth questioning. Sharding the queue by user id costs us
> head-of-line blocking: one user's stuck SMS delays their push. If ordering is
> not actually required, I would drop the sharding and gain isolation.
> [challenging a choice with the cost that decides it]"

Its ceiling: it is still answering "what is the right design". It does not say
what gets built first, what it costs, who operates it, or what happens in the
second quarter.

**The error most readers make here.** Adding the trade-offs as a list at the end
rather than at the decision points. A trade-off stated where the decision is made
reads as reasoning; the same trade-off recited at the end reads as a checklist.

#### Block D. Purpose: add the staff delta, and name what it costs

The staff version contains the senior one and adds sequence, cost, operational
contract and refusal.

> "Before the design, one reframe. The requirement is never send the same
> notification twice, but the failure users complain about is usually
> notification volume, not duplication. I would want to know whether the actual
> pain is duplicates or a lack of batching and rate limiting per user, because
> those are different systems and one of them is much cheaper. Assuming
> duplication is genuinely the requirement, here is the plan. [reframing with
> evidence, then proceeding rather than stalling]
>
> Cost first, because it decides the shape. At 50 M per day, if 5% go to SMS at,
> assume, $0.005 per message, that is 2.5 M messages, so $12,500 per day, roughly
> $375,000 a month. Push and email together are a rounding error against that.
>
> So the expensive component in this system is not any box on my diagram, it is
> the SMS channel, and the highest-value engineering is anything that reduces
> unnecessary SMS: per-user batching, channel fallback ordering, and a preference
> default that prefers push when the app is installed. I would build that in
> quarter one. [priced trade-off, and it changes what gets built]
>
> Operational contract. One SLI per channel: fraction of notifications delivered
> within their channel's target, which is 30 seconds for push, 5 minutes for
> email, 60 seconds for SMS. Fast-burn page at 14.4 times the budget over an
> hour. The one alert that actually matters is per-channel queue age, because
> that predicts a breach before it happens. On call is one rotation, and a
> provider outage must degrade to another channel automatically rather than
> paging a human. [operational design as an output]
>
> Sequence. Quarter one: single worker pool, push and email only, the
> deduplication table, and the cost controls. SMS is deliberately last because it
> is the expensive channel and I want the batching working before it is switched
> on. Quarter two: per-channel pools, provider fallback, delayed bounce handling.
> [sequence with a reason]
>
> What I am not doing: no multi-region, no separate template service, no
> user-facing notification centre. Each is defensible and none is on the critical
> path of the stated problem. If the notification centre matters more than
> deduplication, tell me and I will re-plan. [refusal with an offer]"

Notice what the staff version did not do: it added no components. It removed one
(the per-channel pools moved to quarter two), added constraints and a sequence,
and priced the thing nobody had priced.

!!! tip "Self-explanation prompt"

    Identify the one sentence in the staff answer that would be a mistake at mid
    level, and say why the same sentence is a strength at staff level.

**The error most readers make here.** Performing the staff answer without the
mid answer underneath. If asked how deduplication works and you cannot describe
the unique constraint, the reframing and the cost model read as evasion.

### 16e. Faded examples

**Fade 1: level up an answer to "design a URL shortener".** Last block blank.

- Block A, level signature and base arithmetic: 100 M new links per year is 3.2
  per second, and 10,000 redirects per second is the read side. Reads exceed
  writes by roughly 3,000 to 1, which is the number the whole design hangs on.
- Block B, mid answer and its ceiling: base62 encoding of a counter or a hash,
  a key-value store mapping code to URL, a cache in front, a 301 redirect.
  Correct. Ceiling: does not say why base62 over a hash, does not size the
  keyspace, does not mention that 301 is cached by browsers forever and therefore
  breaks analytics and link editing.
- Block C, senior delta and its ceiling: 7 characters of base62 is 62 to the 7th,
  about 3.5 trillion codes, which at 100 M per year lasts far longer than the
  product will. Choose 302 over 301 specifically because the analytics
  requirement needs each click to reach us, and name the cost, which is one extra
  round trip per click forever. Volunteer the custom-alias collision path.
  Ceiling: still a design, not a plan.
- Block D, staff delta and what it costs: **you write this block.**

??? note "Show answer"

    Price the read path, since it dominates: 10,000 redirects per second is 25.9
    billion per month, and at a small response through a CDN the egress is
    modest, but the origin request rate is not, so the highest-leverage decision
    is the cache TTL at the edge, which trades link-edit latency against origin
    load. State that trade explicitly and pick a number, for example 60 seconds.

    Add the operational contract: redirect availability is the SLI, measured at
    the edge, target 99.95%, and the alert is on edge-observed error ratio, not
    on origin health, because origin can be healthy while users fail.

    Add sequence and refusal: quarter one is the redirect path and the analytics
    pipeline as a fire-and-forget stream, because analytics must never be able to
    slow a redirect. Not doing: custom domains, link expiry policies, or a
    multi-region write path. Name the abuse problem as a real risk with an owner,
    since a shortener is a phishing vector and that is a product and trust cost,
    not an architecture one.

**Fade 2: level up an answer to "how would you monitor this system?"** Last two
blocks blank.

- Block A, level signature and base arithmetic: the honest observation is that
  most candidates answer this with a tool list, so the differentiator is deriving
  the alerts from an SLO rather than naming dashboards.
- Block B, mid answer and its ceiling: metrics for rate, errors and duration;
  logs; traces; dashboards per service; alert on error rate and on high latency.
  Correct and complete as a list. Ceiling: no thresholds derived from anything,
  and no user journey named.
- Block C, senior delta and its ceiling: **you write this block.**
- Block D, staff delta and what it costs: **you write this block.**

??? note "Show answer"

    Block C. Define the SLI at the user-visible edge for one named journey, set
    an SLO with a time budget (99.9% is 43 minutes per 30 days), and derive the
    two alerts from the burn rate rather than from a static threshold: page at
    14.4x over an hour, ticket at 3x over a day.

    Volunteer the cardinality
    trade-off: high-cardinality context goes on traces, which are sampled, and
    never on metrics, which are not. Name saturation signals (queue depth, pool
    wait) as the leading indicators that let you act before latency moves.
    Ceiling: still describes a good monitoring design rather than a plan for
    getting there or a judgement about what it is worth.

    Block D. Price it: a series budget per service with the storage derivation
    attached, and the point that a metrics bill scales with series count times
    retention rather than with traffic, so the retention policy is a real
    decision and not an afterthought.

    Add the organisational output: an SLO is
    only useful if someone is empowered to stop feature work when the budget is
    exhausted, so name that policy as the actual deliverable and the dashboards
    as its by-product. Sequence it: one journey with one SLO and two alerts this
    quarter, rather than instrumenting everything. Refuse something: no per-team
    metrics platform, no tail sampling yet, with the trigger for each stated.

### 16f. Check yourself

**Q1.** A candidate's answer contains Raft, CRDTs, multi-region active-active and
a service mesh. The interviewer scores it below a candidate who used one database
and one queue. Give two reasons that is a correct scoring decision.

??? note "Show answer"

    First, vocabulary without derivation is unverified. If the candidate cannot
    state Raft's write latency floor for their replica placement, or which
    specific field needs a CRDT and why single-writer would not do, then each
    term is a claim they cannot support, and an interviewer's job is to
    distinguish claims from knowledge.

    Second, the components carry costs the candidate did not pay for. Multi-region
    active-active implies a conflict model, a residency position, roughly doubled
    infrastructure and a failover rehearsal. An answer that adds the component
    without the obligations shows the candidate has not operated one. The simpler
    answer, if it satisfies the stated non-functional requirements, is
    demonstrably sufficient, and sufficiency plus stated triggers beats a longer
    parts list.

**Q2.** You are interviewing at senior level and you finish your design with 12
minutes left. Name the highest-value thing to do with those minutes, and the
common thing candidates do instead.

??? note "Show answer"

    Highest value: drive a deep dive yourself, on the part of the system this
    specific problem makes interesting, and make it an operational or failure
    dive if your design dive is already done. Say "I would like to spend the
    remaining time on X, because it is where this design is most likely to break"
    and then do it. Volunteering the dive is itself the senior signal, because
    the alternative is waiting to be asked.

    What candidates do instead: recap the design, or add components. The recap
    adds no information the interviewer lacks, and adding components at minute 45
    means adding them without the derivation you had time for earlier, which
    lowers rather than raises the score.

### 16g. When not to use this

**Performing above your interviewing level.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You can produce the level below it fully, on demand, under follow-up questioning, in the same session |
| Threshold below which it is over-reach | If you cannot size the database, derive the alert thresholds or state the failure modes, then organisational and sequencing talk is a liability, not an asset. Fix the layer below first |
| Cheaper alternative | Pick one area where you genuinely have depth and drive it hard. One real deep dive outscores four gestures at breadth |

**The self-scoring proxy counts.**

| Question | Answer |
|---|---|
| Measurement that justifies using them | You have recorded at least three practice rounds and want to compare yourself against yourself over time |
| Threshold below which they mislead | A single session. Counts vary with the prompt, the interviewer's style and how much they interrupt, so one round tells you nothing |
| Cheaper alternative | One question after each practice round: which sentence did I say that could only have been said by someone who has operated a system like this |

**Level deltas as a template to recite.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You are preparing, and you want to see the same problem from three altitudes to find your own ceiling |
| Threshold below which it is harmful | In the live round. Announcing "and here is the staff-level view" is the memorised-template tell that interviewers now screen for |
| Cheaper alternative | Let the level show through behaviour: volunteer a threshold, price one trade-off, name one thing you are not doing. Never narrate the level itself |

!!! note "Why the eight case studies share their section headings"

    The arc headings below are identical in all eight, so you can jump to the breaking point or the level delta in any of them without hunting. The two deep dives in each are named for the thing that case study actually makes interesting, and no two of the sixteen are alike.

    Uniform navigation, unique content: that is the split, and it is deliberate. If you ever find yourself narrating these heading names in a live round, you are reciting a recipe, which is the exact tell Module 2 tells you to avoid.

## Case study 1: URL shortener

Most published treatments give this prompt one lesson, produce a base62 encoder, and stop. The interesting part is the key space, and that is exactly the part the short version skips. This case study spends its depth there.

### The prompt (shortener)

"Design a service that turns a long URL into a short link, and sends anyone who visits the short link to the original."

### Clarifiers that fork the design (shortener)

Ask these because each answer changes a box on the whiteboard. Do not recite them.

| Question worth asking | If the answer is A the design becomes | If the answer is B the design becomes |
|---|---|---|
| Are short codes user-chosen, or system-generated? | System-generated: you own the whole key space and can pick a bijection over a counter | User-chosen aliases allowed: you need a uniqueness check, a reserved-word blocklist, and a shared namespace with generated codes |
| Do links expire or get deleted? | Never expire: keys are write-once and you never reclaim key space | Expire or delete: you need a reclaim policy, tombstones, and a rule about whether a reclaimed code may be reissued |
| Is per-click analytics a product feature? | No analytics: you can return HTTP 301 and let browsers cache the redirect away | Yes analytics: you must return 302 or 307 so every click reaches you, which multiplies your read traffic |
| Is the same long URL allowed to produce two different short codes? | Deduplicate: one row per long URL, so you need an index on a hash of the long URL | Do not deduplicate: writes are pure inserts, no read-before-write, and per-user analytics stay separable |
| Single region or global? | Single region: one primary, one cache tier, simple | Global: redirect reads served from regional replicas, writes still single-homed, and you must decide the staleness a fresh link may show |
| Is the short code allowed to be guessable? | Guessable is fine: a dense counter-derived space is cheapest | Must be unguessable: you need a sparse space, which changes the code length and rules out a plain counter |
| What is the read to write ratio? | Near 1:1: skip the cache tier entirely | 100:1 or higher: the whole design is a read path, and the write path is almost an afterthought |

### Requirements (shortener)

Functional:

- Create a short code for a submitted URL, and return it.
- Resolve a short code to its URL and redirect.
- Record a click event per resolution.
- Support optional user-chosen aliases.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| New links per day | 10 million | ASSUMED, a mid-size public shortener; the interviewer usually supplies this, so ask |
| Read to write ratio | 100 to 1 | ASSUMED, because a link is created once and clicked many times |
| Redirect availability | 99.99 percent monthly | GIVEN in the prompt framing; a dead redirect breaks every page that embeds it |
| Redirect latency, server side | under 50 ms at p99 | ASSUMED; the redirect is a hop the user did not ask for, so it should be invisible next to page load |
| Link retention | 5 years minimum | ASSUMED; links get embedded in printed material and archived pages |
| Redirect requests per second at peak | 35,000 | DERIVED below |
| Total stored rows after 5 years | 18 billion | DERIVED below |

### Numbers that eliminate options (shortener)

Every line below ends with the option it kills. If a number kills nothing, it does not belong in the answer.

| Calculation | Result | What it rules out |
|---|---|---|
| 10,000,000 writes per day divided by 86,400 seconds | 116 writes per second average, 350 at a 3x peak | Rules out any write-path sharding, any distributed write coordinator, and any queue on the create path. One relational primary absorbs 350 inserts per second |
| 116 writes per second times 100 | 11,600 reads per second average, 35,000 at a 3x peak | Rules out serving redirects from the primary alone at p99, and rules out doing analytics writes synchronously on the redirect path |
| 10,000,000 per day times 365 times 5 years | 18.25 billion rows | Rules out an in-memory-only store and rules out a single unpartitioned table for the long term |
| 18.25 billion rows times 500 bytes per row | 9.1 TB | Rules out holding the full mapping in RAM, so the design needs a disk-backed store with a cache in front |
| 62 to the power 6 equals 5.68e10; 18.25e9 divided by that | 32 percent of the key space consumed | Rules out 6-character codes: at one third occupancy, random generation collides constantly and even a counter leaks how close you are to exhaustion |
| 62 to the power 7 equals 3.52e12; 18.25e9 divided by that | 0.52 percent of the key space consumed | Rules out 8 characters as needless length. 7 characters is the answer |
| 35,000 redirects per second times 500 bytes of response | 17.5 MB per second, about 140 Mbps | Rules out a CDN justified on bandwidth. If you want a CDN here, justify it on geographic latency instead, and say so |
| Hottest 100 million codes times 250 bytes cached | 25 GB of cache | Rules out a large cache cluster. Two nodes plus a replica hold the working set |

!!! tip "Say this out loud in the room"

    "Three hundred fifty writes per second means I am not going to shard the write path, and I want to be explicit that I checked rather than assumed."

### V0: the smallest design that meets the requirements (shortener)

??? note "Predict first: which single component in V0 fails first, and at what number?"

    The in-process cache. It is per-instance and it is empty after every deploy, so the number that matters is not steady-state traffic, it is the first thirty seconds after a rollout, when all 35,000 reads per second reach Postgres at once.

```
+-----------+      +------------------+      +----------------+
| Browser   | ---> | App server       | ---> | Postgres       |
|           | <--- | in-process LRU   | <--- | links table    |
+-----------+      | 200k entries     |      | code PK        |
                   +------------------+      | url longtext   |
                        |                    +----------------+
                        v
                   +------------------+
                   | Click log file   |
                   | appended locally |
                   +------------------+
```

**Caption.** V0 has three components. The browser calls a stateless app server that keeps a small in-process cache; on a miss it does a primary-key lookup in one Postgres instance; click events are appended to a local log file and shipped out of band.

Why this is correct at the stated scale:

- Writes are 350 per second at peak. A single Postgres primary with one unique index does this without effort.
- Reads are 35,000 per second at peak. Under the Zipf-like click distribution assumed above (a small head of links takes most clicks, which is what link sharing produces), a 200,000-entry in-process cache absorbs roughly 90 percent, leaving 3,500 primary-key lookups per second on the primary. That is within one primary's range for point reads on a hot index.
- Short codes come from a Postgres sequence, passed through a bijection so they are not sequential in the output. Details in the first deep dive.

!!! warning "When not to use even this much"

    Below about 5 writes per second and 500 reads per second, drop the in-process cache. At that rate the cache saves under one millisecond per request and adds a stale-entry failure mode you now have to reason about after every deploy. The measurement that justifies adding it back is a database CPU profile where point lookups exceed 30 percent of primary CPU.

### Breaking points, and what each version adds (shortener)

**Breaking point one: the deploy cliff, at any traffic above roughly 8,000 reads per second.**

What fails: the in-process cache is per instance and per process lifetime. A rolling deploy replaces every instance, every cache starts empty, and for the next thirty seconds the full 35,000 reads per second lands on the primary. Connection pool wait time goes vertical, and redirect p99 goes from 8 ms to multiple seconds.

How you observe it: cache hit ratio plotted against deploy markers, and database connection pool wait time at p99. If those two lines are correlated with your release timeline, you have found it. This is a thundering herd, not a capacity problem, and adding database CPU does not fix it.

```
+---------+   +-------------+   +-----------+   +-------------+
| Browser |-->| App server  |-->| Shared    |-->| Postgres    |
|         |   | local LRU   |   | cache     |   | primary     |
+---------+   | 1 sec TTL   |   | 25 GB     |   +-------------+
              +-------------+   +-----------+          |
                     |                                 v
                     v                          +-------------+
              +-------------+                   | Read        |
              | Click       |                   | replicas    |
              | event queue |                   +-------------+
              +-------------+
```

**Caption.** V1 keeps the tiny local cache but puts a shared cache tier behind it, moves reads to replicas, and moves click events onto a queue instead of a local file.

V1 changes, each with the observation that justified it:

| Change | Justified by | Cost you accept |
|---|---|---|
| Shared cache tier, 25 GB | Cache hit ratio collapsing at deploys | A new component that can fail, and cache and database can now disagree |
| Local cache kept, with a 1 second TTL | Hot-key protection, see deep dive two | Up to 1 second of staleness on link edits |
| Read replicas | Primary CPU dominated by point reads | Replication lag means a link can 404 for a second after creation. Fix by writing the new code into the cache on create |
| Click events onto a queue | Local log files are lost when an instance is replaced | Click counts become eventually consistent, which is fine, and you should say so before the interviewer asks |

**Breaking point two: 18 billion rows and 9.1 TB, reached in year five.**

What fails: index maintenance and operational recovery, not query latency. A 9 TB single-instance database has a restore time measured in many hours, and your 99.99 percent monthly availability budget is 4.3 minutes. One restore blows a year of budget.

How you observe it: track time-to-restore from your most recent backup as a monthly drill, and plot it against your error budget. Also watch write latency p99 creeping upward as the index depth grows.

V2 splits the store by responsibility:

- The code-to-URL mapping moves to a hash-partitioned key-value store keyed by short code. It is a pure point-lookup workload, it never needs a join, and hash partitioning by code gives even distribution for free.
- The relational database keeps only user-owned metadata: who created a link, campaign tags, custom alias ownership. That is a small fraction of the rows.
- Click events land in a columnar analytics store via the queue, partitioned by day, with a retention policy.

!!! warning "When not to use a partitioned key-value store here"

    Below about 500 million rows, or below roughly 1 TB, this is over-engineering. A single primary with read replicas is operationally simpler and you can restore it inside your error budget. The measurement that justifies the move is a restore-time drill that exceeds your monthly downtime allowance, not a row count someone found impressive.

### Deep dive one: generating the key

This is the dive that the one-lesson version skips, and it is where a senior answer separates from a recited one.

**The candidate strategies, compared on what actually matters.**

| Strategy | Collision behaviour | Coordination needed | Leaks information | Handles custom alias |
|---|---|---|---|---|
| Hash the URL, truncate to 7 base62 chars | Collides badly, see arithmetic below | None | No | Needs a shared uniqueness check anyway |
| Global counter, base62 encoded | Never collides | One allocator | Yes, codes are sequential and enumerable | Yes, with a shared unique index |
| Counter plus a bijective scramble | Never collides | One allocator | No, output looks random | Yes, same index |
| Random 7 chars with a uniqueness check | Rare collisions, retried | None | No | Yes |
| Pre-generated key pool handed out in blocks | Never collides | A key service | No | Yes |
| Snowflake-style 64-bit ID | Never collides | Node ID assignment | Time ordering leaks | Encodes to 11 chars, too long |

**Why hash-and-truncate fails, with the arithmetic.** With a key space of 62 to the 7 equals 3.52e12 and 1.825e10 keys in use, the expected number of colliding pairs is roughly n squared divided by 2N, which is (1.825e10 squared) divided by (2 times 3.52e12), or about 47 million collisions. Truncated hashing is not collision-free at this scale, so you need a uniqueness check on insert regardless. Once you have that check, the hash bought you nothing.

**The design I would defend.** A single allocator row hands out blocks of 100,000 consecutive counter values. Each app instance holds a block in memory and consumes it locally. Fleet-wide at 350 writes per second, the allocator sees one allocation every 285 seconds, so it is nowhere near a bottleneck and a brief allocator outage is survivable because instances still hold unconsumed blocks.

The raw counter is then passed through a bijection over the 7-character space before it is shown to anyone. Multiply the counter by a fixed odd constant modulo 62 to the 7, or run a small Feistel network over the same domain. Both are reversible, both preserve the never-collides property, and both make consecutive counters produce codes that share no visible structure.

What each property buys you:

| Property | Mechanism | What it prevents |
|---|---|---|
| Never collides | Counter is the source of truth, bijection is one-to-one | Retry loops on insert, and the read-before-write they force |
| Not enumerable | Bijection scrambles adjacency | A scraper walking your entire link set by incrementing a code |
| Gaps are harmless | Unconsumed block values are simply lost on restart | Any need to make block allocation transactional with the insert |
| Custom aliases coexist | Same table, same unique index on code | Two users being handed the same code by different paths |

**Keep the unique index even though the counter guarantees uniqueness.** If the allocator ever splits its brain and hands the same block to two instances, the index turns a silent data-corruption incident into a visible insert error. The index costs you nothing on a workload of 350 inserts per second, and it is the difference between a page and a postmortem.

**The blocklist.** A 7-character base62 space will generate offensive strings. Filter generated codes against a blocklist before issuing them, and skip forward when one hits. This is a real product requirement that almost no candidate mentions, and mentioning it reads as experience.

!!! warning "When not to use a key allocation service"

    Below roughly 50 writes per second on a single database, use the database sequence directly plus the bijection. A separate key service adds a component whose outage stops every write, in exchange for removing a bottleneck you do not have. The measurement that justifies the split is sequence contention appearing in your database wait-event profile.

### Deep dive two: the read path under a Zipf distribution

The dives on this page vary by problem. Here the read path is interesting because a single link can go viral, which is a hot-key problem that does not arise in the crawler or the chat system.

**Hot key.** One link in a widely shared post can take most of your 35,000 reads per second. In a hash-partitioned cache, one key lives on one node, so that node absorbs the whole spike while its peers idle.

The fix is the small local cache with a short time to live, and the number is easy to derive. With 200 app instances and a 1 second local time to live, the shared cache sees at most 200 requests per second for that key regardless of how popular the link becomes, because each instance can only miss once per second. User traffic can grow by a factor of a thousand and the shared cache load for that key does not move.

**Cache stampede.** When the shared cache entry expires, every instance misses at the same moment and they all query the database together. Two mitigations, and you should name the trade-off:

| Mitigation | How it works | Cost |
|---|---|---|
| Probabilistic early expiry | Each reader independently decides to refresh slightly before the expiry, with a random offset | Slightly more database reads in steady state |
| Lease or single-flight | The first miss takes a token and refreshes; others wait or serve stale | Adds a coordination point, and a bug there stalls readers |

Facebook's team described the lease approach for exactly this problem in [Scaling Memcache at Facebook](https://www.usenix.org/conference/nsdi13/technical-sessions/presentation/nishtala) (Nishtala and others, USENIX NSDI 2013, accessed August 2026), which is worth reading as the primary source rather than a summary.

**Negative lookups.** Scanners probe random 7-character codes. Every one is a cache miss followed by a database read that finds nothing. At even a modest probe rate this can exceed your legitimate traffic. Two answers:

- Cache the negative result with a short time to live, say 30 seconds. Cheap, effective, and bounded.
- Put a Bloom filter of all issued codes in front. With 1.825e10 codes at a 1 percent false positive rate, the filter needs about 9.6 bits per element, which is 21.9 GB. That is larger than your hot cache, for a problem that negative caching already solves. Bloom filters (Bloom, Communications of the ACM, 1970) are the right tool when the negative set is unbounded and the positive set is small, which is the opposite of this situation.

**301 versus 302, which is a traffic decision and not a trivia question.** A 301 is a permanent redirect and browsers cache it, so repeat clicks never reach you. A 302 or 307 keeps every click on your servers. The semantics are defined in RFC 9110 (June 2022).

If analytics is a product feature, you must use 302 or 307, and that decision is what produced your 35,000 reads per second in the first place. If it is not, 301 can remove most of your read traffic, and it also removes your ability to ever disable a malicious link.

!!! warning "When not to use a two-tier cache"

    Below roughly 2,000 reads per second, or when your traffic has no hot key (measure this: plot requests per second for your top key against your total), a single shared cache tier is enough. The local tier adds a second staleness window that shows up as "I edited the link and it did not change" support tickets. The measurement that justifies it is a single cache key exceeding about 5 percent of one cache node's capacity.

### Failure modes and the operating contract (shortener)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Thundering herd | Empty local caches after a deploy send all reads to the database | Warm the cache before an instance takes traffic, and stagger the rollout |
| Cache stampede | A hot key expires and every instance refreshes it at once | Probabilistic early expiry, or a lease on refresh |
| Hot key | One viral link saturates one cache node | Local cache with a short time to live, bounding per-key shared cache load |
| Retry storm | The database slows, clients retry, load doubles, it slows further | Bounded retries with jitter, and a circuit breaker on the database call |
| Split brain | Two allocator instances hand out the same counter block | The unique index turns silent duplication into a loud insert failure |
| Metastable failure | After a database blip, the retry backlog keeps load above capacity even after the original cause is gone | Shed load at the gateway, and cap the retry budget as a fraction of successful traffic |

Service level indicators to publish:

- Redirect success rate, measured as non-5xx responses on the resolve path.
- Redirect server-side latency at p50 and p99.
- Shared cache hit ratio, and local cache hit ratio, plotted separately.
- Create success rate, and key block allocation latency.
- Click event pipeline lag.

Alert on error budget burn rate for redirect availability, not on raw error count. A 4.3 minute monthly budget means a 2 percent error rate for 10 minutes is an incident and a 0.001 percent rate all month is not.

Re-checking against the non-functional requirements:

| Requirement | Does V2 meet it? |
|---|---|
| 99.99 percent redirect availability | Yes, provided the read path degrades to serving stale cache entries when the store is unavailable. State that degradation explicitly |
| Redirect p99 under 50 ms | Yes at a 90 percent-plus cache hit ratio; the miss path is a single point lookup |
| 35,000 reads per second at peak | Yes, and the local cache tier means the number could grow tenfold without a shape change |
| 5 year retention, 9.1 TB | Yes in the partitioned store; no in a single primary, which is why V2 exists |

### Where this answer is a simplification (shortener)

- **Abuse is the actual hard problem.** A shortener is a phishing tool by default. Real services check destinations against reputation feeds at create time and re-check them later, because a link can be created benign and repointed. That means the mapping is mutable, which invalidates the "write-once" simplification the whole design leans on.
- **Custom domains multiply everything.** Customers want links on their own domain, which means per-domain certificate issuance and renewal, and it turns your single hostname into thousands. The ACME protocol (RFC 8555, March 2019) is the mechanism.
- **Bots inflate analytics.** Link preview crawlers from chat and social apps fetch every link that gets pasted, often several times. Your click counts are wrong unless you filter them, and filtering them is an ongoing arms race.
- **Deletion is a legal requirement, not a feature.** Click logs contain IP addresses. Retention policy and deletion requests are design inputs.
- **Further reading with dates.** Instagram's engineering post on sharding and ID generation (2012) is the clearest short primary source on ID design trade-offs; search for it by title, since the post has moved hosts. Twitter's [archived Snowflake repository](https://github.com/twitter-archive/snowflake) (first released 2010, repository archived September 2021, accessed August 2026) documents the time-ordered alternative. RFC 9110 (June 2022) is the authority on redirect semantics.

### Level delta: the same prompt at three levels (shortener)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Key generation | Base62 encoding of an auto-increment ID | Counter plus a bijection, and explains why hash-truncate needs a uniqueness check anyway | Adds block allocation, the blocklist, and keeps the unique index specifically as a corruption tripwire |
| Estimation | Produces numbers | Produces numbers and uses each to remove an option | Notices that 350 writes per second means the write path needs no design at all, and reallocates the remaining time to the read path |
| Cache | Proposes a cache | Sizes it at 25 GB from the working set, and picks an eviction policy | Adds the local tier with a derived per-key ceiling, and names the staleness ticket it will generate |
| Failure handling | Mentions retries | Names thundering herd and cache stampede with mitigations | Names metastable failure and states the load-shedding rule that stops it |
| Redirect code | Says 301 or 302 without a reason | Picks 302 because analytics is a requirement | Points out that this one choice sets the read traffic for the whole system, and offers 301 as a cost lever if analytics is dropped |
| Scope discipline | Adds components when unsure | Justifies each component with a measurement | Removes the CDN after showing bandwidth is 140 Mbps, and says a CDN would need a latency argument instead |

What each level added, in one line each:

- **Mid to senior:** every number now eliminates something, and every box now has a measurement behind it.
- **Senior to staff:** the operational consequences enter the design (restore time against error budget, corruption tripwires, load shedding), and at least one component gets removed on evidence.

---

## Case study 2: Web crawler

### The prompt (crawler)

"Design a web crawler that fetches a large portion of the public web and stores the pages for a search index."

### Clarifiers that fork the design (crawler)

| Question worth asking | If the answer is A the design becomes | If the answer is B the design becomes |
|---|---|---|
| Is this a one-time crawl or continuous? | One-time: the frontier drains and the job ends, no re-crawl scheduler | Continuous: you need per-URL change-rate estimation and a re-crawl priority, which is a second scheduling problem |
| Do pages need JavaScript executed? | Static HTML only: one HTTP fetch per page, cheap | Rendered: a headless browser per page, which changes cost per page by roughly two orders of magnitude and dominates the capacity plan |
| Is the target the whole web or a defined set of sites? | Defined set: politeness is simple, the frontier fits in memory, and host diversity is your constraint | Open web: you need trap detection, per-domain budgets, and a frontier that does not fit in memory |
| Do we store raw pages or only extracted text? | Raw: 25 TB at our scale, object store required | Extracted text only: roughly one fifth the bytes, but you cannot re-parse later without re-crawling |
| How fresh must popular pages be? | Days: a single-priority frontier is fine | Minutes for a subset: you need a separate high-frequency lane, which is a different system from the bulk crawl |
| Who owns the politeness policy? | Fixed global delay: simple, and needlessly slow on large sites | Adaptive per host: delay derived from observed response time, which needs per-host state and a feedback loop |
| Are we allowed to crawl at all? | Public content only, robots respected: standard design | Sites with terms restrictions or login walls: this becomes a legal and access-management problem before it is an engineering one |

### Requirements (crawler)

Functional:

- Fetch pages starting from a seed list, and discover new URLs from fetched pages.
- Respect robots exclusion rules and per-host rate limits.
- Store fetched content and its metadata.
- Avoid fetching the same content twice, whether the duplication is by URL or by content.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| First milestone | 10 million pages in 7 days | ASSUMED; every crawler starts with a milestone, and designing for the end state first is the classic mistake here |
| Target scale | 1 billion pages in 30 days | ASSUMED; this is a mid-size index, not a general web index |
| Politeness | at most 1 request per host per 2 seconds by default | ASSUMED; conservative, and the default most operators would accept without complaint |
| robots.txt cache lifetime | at most 24 hours | GIVEN by RFC 9309 (September 2022), which says crawlers should use standard HTTP caching and should not use a cached copy beyond 24 hours |
| Stored bytes per page after compression | 25 KB | ASSUMED; roughly 100 KB of HTML at about 4 to 1 text compression |
| Politeness violations | zero tolerated | GIVEN; a violation is how you get blocked and how you get complained about |
| Crawl rate at target | 386 pages per second | DERIVED below |

### Numbers that eliminate options (crawler)

| Calculation | Result | What it rules out |
|---|---|---|
| 1e9 pages divided by 30 days of seconds (2.592e6) | 386 pages per second | Rules out nothing yet; it is the input to everything below |
| 386 per second times 0.5 second mean fetch latency, by Little's law | 193 requests in flight | Rules out a large cluster. Two hundred concurrent sockets is one machine's worth of I/O, so the machine count must be justified by state size, not by fetch throughput |
| 386 per second at 1 request per host per 2 seconds | at least 772 distinct hosts in flight | Rules out a single global FIFO frontier. The frontier must be organised by host or you cannot reach the target rate at all |
| 1e9 pages times 10 outlinks each | 1e10 discovered URLs | Rules out treating discovered URLs as a small side table |
| 1e10 URLs times 100 bytes | 1 TB of frontier | Rules out an in-memory frontier |
| 1e10 URL hashes times 8 bytes | 80 GB seen-set | Rules out an exact in-memory set on one machine, and forces the choice explored in deep dive two |
| 1e9 pages times 25 KB | 25 TB | Rules out local disks and rules out a relational store for page bodies. Object store |
| 386 per second times 100 KB on the wire | 38.6 MB per second, about 309 Mbps | Rules out a multi-datacentre crawl justified on bandwidth. One well-connected site is enough |
| Milestone: 1e7 pages divided by 7 days of seconds (604,800) | 16.5 pages per second | Rules out the entire distributed design for the first milestone. This is the number that justifies V0 |

### V0: the smallest design that meets the requirements (crawler)

??? note "Predict first: at the milestone rate of 16.5 pages per second, what is the first thing that stops working when you push toward 386?"

    Not the network and not the CPU. The seen-set and the frontier, both of which are held in memory in V0. At the milestone they are 800 MB and 10 GB; at target they are 80 GB and 1 TB. State size, not throughput, is what forces the distributed design.

```
+-----------+     +----------------------+     +--------------+
| Seed list | --> | Crawler process      | --> | Object store |
+-----------+     | frontier by host     |     | page bodies  |
                  | seen-set in memory   |     +--------------+
                  | robots cache         |
                  | fetch parse extract  |
                  | discovered URLs feed |
                  | back into own queue  |
                  +----------------------+
```

**Caption.** V0 is a single process holding a per-host frontier, an in-memory seen-set and a robots cache; it fetches, parses, writes bodies to an object store, and feeds discovered URLs back into its own frontier.

Why this is correct at the milestone:

- 16.5 pages per second times 0.5 second latency is about 9 requests in flight. A single process handles that without breaking a sweat.
- The seen-set at 1e8 URLs times 8 bytes is 800 MB, which fits in memory.
- Storage at 1e7 pages times 25 KB is 250 GB, which is one bucket and no partitioning decision.
- Politeness is trivially correct because one process owns every host, so there is no coordination question at all.

That last point is the one to say out loud. Politeness is a global constraint, and V0 satisfies it for free by having a single owner. Every later version has to work to keep that property.

!!! warning "When not to distribute this"

    Below roughly 50 pages per second and 100 million discovered URLs, a single machine with a disk-backed frontier is the correct answer, and adding a cluster costs you the free politeness guarantee. The measurement that justifies distribution is resident memory for the seen-set plus frontier exceeding what one machine can hold, or a restart recovery time you cannot tolerate.

### Breaking points, and what each version adds (crawler)

**Breaking point one: state size, at roughly 100 million pages crawled.**

What fails: the in-memory seen-set crosses available RAM. You observe it as resident set size climbing linearly with pages crawled, then the page fault rate spiking and the crawl rate collapsing. A restart at this point loses all frontier state, and re-deriving it means re-crawling.

V1 shards by registrable domain:

```
+--------------------+   +--------------------+   +------------+
| Router             |-->| Fetcher shard 1    |-->| Object     |
| hashes registrable |   | owns those hosts   |   | store      |
| domain to a shard  |   | frontier on disk   |   | page       |
| takes back every   |   | bloom plus KV seen |   | bodies     |
| discovered URL the |   +--------------------+   +------------+
| shards emit        |   +--------------------+
+--------------------+-->| Fetcher shard N    |
                         | same structure     |
                         +--------------------+
```

**Caption.** V1 routes every discovered URL to the shard that owns its registrable domain; each shard keeps its own disk-backed frontier and seen-set, and all shards write bodies to a shared object store.

The critical design choice: shard by **registrable domain**, not by hostname and not by full URL.

| Sharding key | Politeness consequence |
|---|---|
| Full URL hash | Every shard fetches from every host, so the per-host rate limit needs cross-shard coordination on every request. Unworkable |
| Hostname | A site with wildcard subdomains spreads across all shards, and each shard independently rate-limits, so the origin server receives N times the intended rate |
| Registrable domain | One shard owns all hosts under a domain, so the rate limit stays a local decision with no coordination |

That property, keeping politeness node-local, is what makes the shard boundary correct. The IRLbot work ("IRLbot: Scaling to 6 Billion Pages and Beyond", WWW 2008) documents the subdomain explosion problem in detail and is the primary source worth citing here.

**Breaking point two: the frontier stops being about capacity and starts being about quality, at roughly 1 billion discovered URLs.**

What fails: a small number of hosts consume a large fraction of the frontier. Infinite calendars, session identifiers in query strings, and faceted-search parameter combinations generate unbounded URL spaces. You observe it as a pages-per-host distribution where the top host holds a double-digit percentage of pending URLs, and a rising content-duplicate rate.

V2 adds three things:

- A URL canonicaliser applied before the seen-set check, so parameter noise collapses.
- A per-domain budget: a cap on pending URLs and on pages per crawl cycle, so no single domain can starve the frontier.
- A near-duplicate detector, covered in deep dive two.

!!! warning "When not to add per-domain budgets"

    On a defined-site crawl of under 10,000 known hosts, budgets add tuning surface for a problem you can see and fix by hand. The measurement that justifies them is a single registrable domain exceeding a few percent of total frontier depth.

### Deep dive one: politeness and frontier scheduling

Two constraints pull against each other. Politeness says wait between requests to the same host. Throughput says never leave a fetch slot idle. Frontier design is the resolution.

**The scheduling invariant, with the number.** To keep F fetch slots busy with a per-host delay of d seconds and a mean fetch latency of L seconds, the number of hosts you must be actively tracking is at least F times d divided by L. With F equal to 200, d equal to 2 seconds and L equal to 0.5 seconds, that is 800 hosts. If your frontier holds URLs from fewer than 800 ready hosts, you cannot hit 386 pages per second no matter how much hardware you add.

**The structure that satisfies it.**

```
+--------------------+       +---------------------------+
| Priority intake    | ====> | Per host queues           |
| new and recrawl    |       | one FIFO per active host  |
+--------------------+       +---------------------------+
                                        |
                                        v
                             +---------------------------+
                             | Ready heap keyed by       |
                             | next allowed fetch time   |
                             +---------------------------+
                                        |
                                        v
                             +---------------------------+
                             | Fetcher slot pool         |
                             | 200 concurrent            |
                             +---------------------------+
```

**Caption.** URLs enter a priority intake, are placed into a queue per active host, and each host queue registers in a heap ordered by the earliest time that host may next be fetched; fetchers pull whichever host is due.

This two-level shape (a priority-ordered intake feeding per-host queues) is the structure described in the Mercator paper (Heydon and Najork, 1999). It is worth naming the source rather than presenting it as novel, and it is worth noting the critique: Mercator's fixed queue count assumes a roughly stable set of active hosts, which is a poor fit for a crawl where host popularity shifts hourly. A dynamically sized host-queue map with eviction of idle hosts is the modern adaptation.

**Adaptive politeness beats a fixed delay.** A fixed 2 second delay is simultaneously too fast for a struggling shared-hosting server and too slow for a large site that would happily serve you fifty requests per second. Set the delay as a multiple of the observed response time:

| Observed p50 response from host | Delay at a multiplier of 10 | Effective rate |
|---|---|---|
| 50 ms | 0.5 s | 2 pages per second |
| 200 ms | 2 s | 0.5 pages per second |
| 2 s | 20 s | 0.05 pages per second |

The multiplier is a single tunable, and the mechanism is self-correcting: a host you are hurting slows down, which slows you down further. That feedback loop is the whole point.

**robots.txt handling, precisely.** RFC 9309 (September 2022) standardised the exclusion protocol. Two operational points candidates usually miss:

- The 24 hour cache ceiling means robots fetches are themselves a traffic source. At 1 million active domains, that is 1 million fetches per day, which is 11.6 per second, about 3 percent of your crawl budget. Derivable, and worth stating.
- A robots fetch that fails with a server error is not permission to crawl. Treat 5xx as "disallow, retry later"; treat 404 as "allow". Getting this backwards is how crawlers earn bans.
- Crawl-delay is not part of RFC 9309. If you honour it, say that you are honouring a non-standard extension.

**DNS is a real bottleneck.** At 386 pages per second across hundreds of distinct hosts, uncached DNS resolution becomes a per-request dependency with variable latency. Run a caching resolver inside the crawler, honour record TTLs, and floor very short TTLs so a misconfigured domain cannot force a lookup per fetch.

!!! warning "When not to build a two-level frontier"

    Below roughly 20 pages per second with fewer than 10,000 seed hosts, a single priority queue plus a map of last-fetched timestamps per host is enough, and it is far easier to debug. The measurement that justifies the two-level structure is fetcher slot idle time above about 20 percent while the frontier is non-empty, which means you are blocked on politeness, not on work.

### Deep dive two: deduplication at three levels

The dives differ from the URL shortener's deliberately: there the interesting read-path property was a hot key, here it is that most of your discovered work is work you have already done. Duplicate detection is the crawler's dominant cost.

**Level one: URL identity.** Canonicalise before hashing. Lowercase the scheme and host, drop the default port, remove the fragment, resolve dot segments, sort query parameters, and strip known tracking parameters. Then hash the result. Skipping canonicalisation multiplies your seen-set and your frontier by the number of parameter permutations a site emits.

**The seen-set decision, and the counterintuitive part.** The instinct is a Bloom filter. Size it and see:

| Target false positive rate | Bits per element | Total for 1e10 URLs |
|---|---|---|
| 1 percent | 9.6 | 12.0 GB |
| 0.1 percent | 14.4 | 18.0 GB |

Going from 1 percent to 0.1 percent costs 6 GB, which is cheap. But look at what a false positive means here: the filter says "seen" when the URL is new, so you silently never crawl that page. At 1 percent over 2 billion genuinely new URLs, that is 20 million pages you will never index and never notice.

Now the part that is usually stated backwards. A Bloom filter saves disk lookups only for items that are **absent**. On the web, most discovered links point to pages you have already seen, so most checks return "maybe present" and still require the exact lookup. If 80 percent of your 1e10 checks are for already-seen URLs, the filter eliminates lookups for the other 20 percent and leaves 8 billion disk reads intact. The usual pitch for a Bloom filter does not apply to this workload.

**The design that actually works: batch and merge.** Buffer discovered URL hashes, and once per interval sort them and merge against the sorted on-disk seen-set.

| Approach | Work for 8e9 seen-set checks | Notes |
|---|---|---|
| Random point lookups at 100 microseconds each | 8e5 seconds, about 9 days | ASSUMED device latency, typical of an NVMe random read; the ratio is what matters, not the exact figure |
| Sequential merge of an 80 GB sorted set at 500 MB per second | 160 seconds | ASSUMED sequential throughput, deliberately conservative for NVMe |

The merge turns a random-I/O problem into a sequential one, and the cost of that is latency: a discovered URL is not known to be new until the next merge runs. For a crawler that is a perfectly acceptable trade, and saying so is the answer.

**Level two: exact content identity.** Hash the normalised body after stripping whitespace and boilerplate. At 1e9 pages times 8 bytes, the content-hash set is 8 GB and fits in memory. This catches mirrors, and it catches the duplicate URLs that canonicalisation missed, which is most of them.

**Level three: near duplicates.** Templated pages differ in a byline or a timestamp and are otherwise identical. Exact hashing misses all of them. The standard method is a 64-bit simhash with a Hamming distance threshold, described in "Detecting Near-Duplicates for Web Crawling" (Manku, Jain and Das Sarma, WWW 2007).

The retrieval trick is a pigeonhole argument you can derive in the room. Split the 64-bit fingerprint into 4 blocks of 16 bits. If two fingerprints differ in at most 3 bits, those 3 bits cannot cover all 4 blocks, so at least one block matches exactly. Build 4 tables, each keyed on a different block, and a candidate lookup is 4 exact lookups.

Cost: 4 tables times 1e9 entries times 16 bytes per entry equals 64 GB. That is affordable, and the derivation is what makes it a senior answer rather than a recalled fact.

!!! warning "When not to use simhash"

    Below roughly 10 million documents, exact content hashing plus manual inspection of the top duplicate clusters is cheaper and far more debuggable. Simhash carries real tuning cost (token weighting, threshold selection, and a threshold that is wrong in both directions on different site types), and that cost is not repaid until duplicate density measurably degrades index quality. The measurement that justifies it is a near-duplicate rate you can demonstrate on a sample, not an assumption that the web is duplicative.

### Failure modes and the operating contract (crawler)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Retry storm | A host returns 503 under load, your fetcher retries, and your crawler becomes the outage | Treat 429 and 503 as politeness signals: multiply the host delay, do not retry immediately |
| Thundering herd | Robots caches for a large hosting provider all expire in the same minute | Jitter the robots cache lifetime per host |
| Hot shard | One registrable domain with 100 million URLs pins one fetcher shard | Per-domain frontier budget, plus a rebalance that moves the domain, not the URLs |
| Split brain | A shard-map change is applied unevenly, so two shards own a domain and each applies its own rate limit | Version the shard map, refuse to fetch under an unconfirmed map version, and alert on any host fetched by two shards |
| Crawler trap | A calendar generates a new URL per day forever | Depth limits per host, URL pattern detection, and a hard budget |
| Amplification abuse | A page links to 100,000 URLs on a third-party host, using your crawler as a load generator | Cap discovered URLs accepted per source page, and rate-limit by target host regardless of source |

Service level indicators:

- Pages fetched per second, against the target rate.
- Fetch outcome distribution by status class, watched for shifts rather than absolute values.
- **Politeness violations per host per minute.** This is the indicator that protects your ability to keep crawling at all, and its target is zero.
- Fetcher slot idle time while the frontier is non-empty, which is the direct signal that host diversity is too low.
- Frontier depth and its growth rate versus drain rate.
- Duplicate rate at each of the three levels, tracked separately.

Alert on any non-zero politeness violation, on frontier growth exceeding drain for a sustained window, and on fetch success rate dropping by more than a set margin from its trailing baseline.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| 1e9 pages in 30 days | Yes, if host diversity exceeds 772 ready hosts; that is the binding constraint, not hardware |
| Zero politeness violations | Yes by construction, because domain ownership is exclusive per shard, and the shard-map versioning is what keeps it true during a resize |
| robots respected within 24 hours | Yes, at a cost of about 3 percent of the crawl budget |
| No page fetched twice | Not exactly. The batch merge means a URL discovered twice within one merge interval can be fetched twice. State this as an accepted, bounded cost |

### Where this answer is a simplification (crawler)

- **JavaScript rendering changes the cost model completely.** A headless browser render costs roughly two orders of magnitude more CPU than an HTTP fetch and parse. If a meaningful share of your target corpus requires rendering, the capacity plan is a rendering plan, and the fetch tier stops being interesting.
- **Sitemaps are a better discovery channel than link following** for large sites, and they carry change hints. The sitemaps protocol at sitemaps.org (0.90, in use since 2005) is the reference.
- **Crawl budget is a published concept on the receiving side.** Google documents crawl capacity limit and crawl demand for large sites in [Optimize your crawl budget](https://developers.google.com/search/docs/crawling-indexing/large-site-managing-crawl-budget) (Google Search Central, page dated July 2026, accessed August 2026), which is useful for understanding what site operators expect from a crawler.
- **The legal surface is real.** Terms of service, jurisdiction, and the wave of AI-crawler restrictions mean "the public web" is a shrinking and contested set. Cloudflare wrote on 1 July 2025 that it was changing its default to block AI crawlers that do not pay for content ([Content Independence Day: no AI crawl without compensation](https://blog.cloudflare.com/content-independence-day-no-ai-crawl-without-compensation/), accessed August 2026), which materially changes what a new crawler can reach.
- **You may not need to crawl at all.** Common Crawl publishes monthly archives of billions of pages under an open licence. For many products, consuming that corpus is the correct engineering answer, and proposing it shows judgement rather than laziness.
- **Primary sources worth reading:** the Mercator paper (1999) for frontier architecture, IRLbot (WWW 2008) for domain-level scaling and trap resistance, Manku et al. (WWW 2007) for near-duplicate detection, and RFC 9309 (September 2022) for exclusion semantics.

### Level delta: the same prompt at three levels (crawler)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Starting point | Draws the distributed cluster immediately | Starts at a milestone and justifies each shard by a state-size measurement | Names politeness as a global invariant that V0 satisfies for free, and treats preserving it as the design's main job |
| Sharding key | Hashes the URL | Shards by hostname | Shards by registrable domain and explains the subdomain-explosion failure that hostname sharding causes |
| Deduplication | Proposes a Bloom filter | Sizes the Bloom filter and picks a false positive rate | Shows the filter helps the wrong side of this workload, and proposes the batch merge with the sequential versus random comparison |
| Politeness | Fixed delay per host | Adaptive delay derived from response time | Adds the shard-map versioning that keeps politeness correct during a rebalance, which is the only time it actually breaks |
| Traps | Not mentioned | Depth limits | Per-domain budget plus link-amplification caps, framed as protecting third parties from your system |
| Scope | Builds everything | Defers rendering and re-crawl | Asks whether Common Crawl removes the need to build this at all, and prices both paths |

What each level added:

- **Mid to senior:** state size replaces throughput as the reason for every component, and politeness becomes a derived constraint rather than a policy footnote.
- **Senior to staff:** the design is evaluated for its effect on systems it does not own, and the build-versus-consume question is asked before the architecture.

---

## Case study 3: News feed

### The prompt (feed)

"Design the home timeline for a social network, so that a user opening the app sees recent posts from the accounts they follow."

### Clarifiers that fork the design (feed)

| Question worth asking | If the answer is A the design becomes | If the answer is B the design becomes |
|---|---|---|
| Chronological or ranked? | Chronological: a cursor on time plus identifier is stable, and pagination is a solved problem | Ranked: the list reorders between page requests, so you need session-pinned materialisation, covered in deep dive two |
| What is the follower count distribution? | Bounded, say under 10,000 followers: pure fan-out on write works and you are done | Heavy tailed with accounts above a million followers: you need the hybrid, and the threshold is derivable |
| Must a post be visible to all followers within seconds? | Minutes acceptable: fan-out can be lazy and cheap | Seconds: fan-out capacity per author becomes the binding constraint and sets the hybrid threshold |
| Can the feed include content the user does not follow? | Follow graph only: the candidate set is a fan-in over followees | Recommended content included: this is a retrieval and ranking system, and the fan-out question becomes a minor subsystem |
| What fraction of accounts are active? | Most accounts active: fan-out to all followers | A large inactive tail: fan-out only to recently active followers, plus a backfill path for returners |
| Are deletes and privacy changes retroactive? | Best effort: feeds can briefly show a deleted post | Strict: every materialised feed entry needs a filter at read time, which reintroduces a per-post check you were trying to avoid |
| One feed, or several surfaces? | One: a single materialisation | Several (home, profile, notifications): materialisation cost multiplies, and you should ask whether they can share one store |

### Requirements (feed)

Functional:

- Return a page of recent posts from accounts the user follows.
- Support paging backward through the feed without duplicates or gaps.
- Reflect a new post from a followed account within a stated freshness window.
- Stop showing posts from an account after an unfollow.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Daily active users | 300 million | ASSUMED; the interviewer usually gives a user count, so ask before assuming |
| Feed opens per user per day | 5 | ASSUMED; a conservative figure for a social product |
| Share of users who post, and posts each | 1 percent, 2 posts per day | ASSUMED; posting is far rarer than reading in every public description of social usage |
| Mean followers per posting account | 200 | ASSUMED for the median; the tail is the whole problem and is handled separately |
| Feed read latency | under 200 ms at p99, server side | ASSUMED; the feed is the app's first screen |
| Freshness, publish to visible | under 30 seconds at p99 | ASSUMED, and this number sets the hybrid threshold, so it is worth negotiating in the room |
| Peak feed reads per second | 52,000 | DERIVED below |

### Numbers that eliminate options (feed)

| Calculation | Result | What it rules out |
|---|---|---|
| 300e6 users times 5 opens divided by 86,400 | 17,400 feed reads per second, 52,000 at 3x peak | Rules out any per-read fan-in that touches hundreds of rows, once combined with the followee count below |
| 1 percent of 300e6 times 2 posts, divided by 86,400 | 70 posts per second, 210 at 3x peak | Rules out treating the write path as a scaling problem. Seventy writes per second is nothing |
| Reads divided by writes | 250 to 1 | Rules out pull-only as the default. When reads outnumber writes 250 to 1, precomputing at write time is the cheaper side |
| 70 posts per second times 200 followers | 14,000 feed inserts per second, 42,000 at peak | Rules out worrying about the median author. Fan-out at the median is a rounding error |
| An account with 50 million followers, at 20,000 inserts per second of per-author budget | 2,500 seconds, about 42 minutes | Rules out pure push. The last follower sees the post 42 minutes late, blowing the 30 second freshness requirement by two orders of magnitude |
| Freshness budget 30 s times per-author budget 20,000 per second | 600,000 followers | This is the push-versus-pull threshold, derived rather than guessed |
| 300e6 users times 500 feed entries times 32 bytes of identifiers | 4.8 TB | Rules out nothing on its own, but see the next line |
| 300e6 times 500 times 1 KB of full post content | 150 TB | Rules out storing post bodies in the feed. Store identifiers, hydrate at read |
| 52,000 reads per second times 40 hydration lookups, batched 20 per call | 104,000 batched cache calls per second | Rules out a single cache node, and rules out issuing one lookup per item |
| 60 percent of accounts inactive for 30 days, times 14,000 inserts per second | 8,400 wasted inserts per second and 60 percent of 4.8 TB | Rules out unconditional fan-out. Fan out to active followers only, and backfill returners |

### V0: the smallest design that meets the requirements (feed)

??? note "Predict first: at what feed read rate does the pull design stop working, given a median of 100 followees?"

    Around 1,000 feed reads per second. That is 100,000 index probes per second on the posts table, which is roughly where a single well-indexed relational primary tops out for this access pattern. Below that number, pull is not just acceptable, it is better, because there is no fan-out lag and no materialised state to repair.

```
+---------+     +--------------+     +----------------------+
| Client  | --> | Feed service | --> | Posts table          |
+---------+     |              |     | index author id time |
                |              |     +----------------------+
                |              |     +----------------------+
                |              | --> | Follows table        |
                +--------------+     | index follower id    |
                                     +----------------------+
```

**Caption.** V0 reads the caller's followee list, then runs one indexed range query per followee merged by time, with no precomputed feed state anywhere.

Why this is correct at the stated smaller scale:

- No materialised state means no fan-out lag, no repair jobs, and no divergence between the feed and the truth.
- An unfollow takes effect instantly and for free.
- A deleted post disappears everywhere immediately, satisfying the strict-privacy branch of the clarifier table without any extra machinery.
- Storage is one copy of each post.

!!! warning "When not to move off pull"

    Below roughly 1,000 feed reads per second, or when the median followee count is under about 50, fan-out on write is over-engineering. It buys read latency you are not short of, and it costs you a whole class of consistency bugs. The measurement that justifies the move is feed read p99 climbing with followee count, visible as a plot of latency against the followee-count percentile.

### Breaking points, and what each version adds (feed)

#### Breaking point one, then V1 (feed)

**What fails: 17,400 reads per second times 200 followees is 3.48 million index probes per second.**

How you observe it: plot feed read latency at p99 against the user's followee-count percentile. Pull degrades linearly with followee count, so the curve is a straight line upward, and the users at the ninetieth percentile of followee count are the ones filing complaints. That shape is the signature of a fan-in problem and it distinguishes this from a general capacity shortage.

V1 pushes at write time:

```
+---------+  +--------------+  +-----------------+
| Author  |->| Post service |->| Posts store     |
+---------+  +--------------+  +-----------------+
                    |
                    v
             +--------------+     +-----------------+
             | Fanout queue |---->| Fanout workers  |
             +--------------+     +-----------------+
                                          |
                                          v
                                  +-----------------+
+---------+   +--------------+    | Feed lists      |
| Reader  |-->| Feed service |--->| one per user    |
+---------+   | hydrate ids  |    +-----------------+
              +--------------+
```

**Caption.** V1 writes each post once to the posts store, then a queue of fan-out workers appends the post identifier to one materialised list per active follower; readers do one range read on their own list and hydrate identifiers into content.

A feed read is now one range read plus a batched hydration, independent of followee count. That is the whole point: the cost moved from a variable the reader controls to a variable the author controls.

!!! warning "When not to add the fan-out queue"

    If posts per second times mean followers is under roughly 1,000 inserts per second, do the fan-out inline in the write request and skip the queue. A queue adds lag you must monitor and a backlog you must drain. The measurement that justifies it is post-creation p99 being dominated by fan-out time.

#### Breaking point two, then V2 (feed)

**What fails: the account with 50 million followers, at 42 minutes of fan-out.**

How you observe it: fan-out job duration is bimodal, with a tight cluster under a second and a sparse tail in the thousands of seconds. Feed freshness measured at the follower level shows a systematic bias where followers written late in the fan-out see the post minutes after followers written early. Meanwhile, ordinary authors' fan-out jobs queue behind the celebrity's and their freshness degrades too, which is the part that makes this urgent.

V2 is the hybrid:

| Author's follower count | Delivery | Read cost |
|---|---|---|
| Below 600,000 | Push into follower feed lists | One range read |
| At or above 600,000 | Not pushed; the author's own post list is read at feed time | One extra range read per celebrity followee, batched |

The threshold of 600,000 is derived, not chosen: freshness budget of 30 seconds multiplied by the 20,000 inserts per second a single author's fan-out may consume without starving everyone else. If the interviewer relaxes freshness to 5 minutes, the threshold moves to 6 million and the hybrid may become unnecessary. Say that out loud; it demonstrates that the number is a lever and not a fact.

The read-side merge cost is bounded because a user follows few very large accounts. Assume a ceiling of 50 celebrity followees per user (ASSUMED; it is bounded by how many accounts above 600,000 followers exist and how many any one person follows). Fifty range reads issued as one batched call adds a single round trip.

!!! warning "When not to build the hybrid"

    If your largest account has under about 100,000 followers, the hybrid is pure complexity: at 20,000 inserts per second per author, that account's fan-out finishes in 5 seconds, inside the freshness budget. The measurement that justifies the hybrid is the fan-out duration distribution developing a tail that crosses your freshness service level objective.

### Deep dive one: the fan-out cost model as follower count grows

**The crossover is a property of an edge, not of a user.** This is the framing that makes the whole trade-off collapse into one comparison. Consider a single follow edge from author A to follower B:

- Push costs one insert every time A posts.
- Pull costs one range read every time B opens the feed.

So push is cheaper on that edge exactly when **A's posts per day is less than B's feed opens per day**. Everything else in this section is a consequence.

| Author profile | Posts per day | Follower opens per day | Cheaper on that edge |
|---|---|---|---|
| Typical person | 2 | 5 | Push, by 2.5x |
| Prolific news account | 50 | 5 | Pull, by 10x |
| Dormant account followed by actives | 0.1 | 5 | Push, by 50x |
| Any author, follower who never opens | 2 | 0 | Pull, infinitely. This is the inactive-user case |

Checking the celebrity with the same rule: an account posting 20 times a day to 50 million followers generates 1 billion inserts per day under push. Those 50 million followers collectively open the feed 250 million times per day, so pull costs 250 million reads. Pull is 4 times cheaper on steady-state volume, and it is unboundedly better on freshness, because there is no fan-out to wait for.

**Total cost as a function of follower count.** Hold posts per day at 2 and follower opens at 5, and vary F:

| Followers F | Push inserts per day | Pull reads per day | Push fan-out duration at 20,000 per second |
|---|---|---|---|
| 200 | 400 | 1,000 | 0.02 s |
| 10,000 | 20,000 | 50,000 | 1 s |
| 600,000 | 1,200,000 | 3,000,000 | 30 s, at the freshness limit |
| 50,000,000 | 100,000,000 | 250,000,000 | 2,500 s, far past the limit |

Read the table carefully: push stays cheaper on raw operation count at every row, including the last. Push does not lose on cost, it loses on **latency and on burst isolation**. That distinction is the senior-level point, and most treatments get it wrong by claiming push is too expensive for celebrities.

**The three fan-out variants, side by side.**

| Property | Push | Pull | Hybrid |
|---|---|---|---|
| Feed read cost | One range read | One read per followee | One range read plus a bounded merge |
| Write cost | Follower count | One | Follower count below the threshold |
| Freshness | Fan-out duration | Immediate | Immediate for celebrities, fan-out duration for everyone else |
| Unfollow correctness | Needs a filter or a repair | Correct for free | Needs a filter on the pushed portion only |
| Storage | One entry per follower | Zero | Proportional to non-celebrity edges |
| Operational surface | Queue lag, backlog, repair jobs | None | Both, plus a threshold to tune |

**Fan out to active followers only.** With 60 percent of accounts inactive for 30 days, skipping them removes 8,400 inserts per second and 60 percent of the 4.8 TB. The returning-user path costs almost nothing: at an assumed 100,000 returns per day times 200 followees, that is 20 million range reads per day, which is 231 per second. Two orders of magnitude cheaper than what you saved. This is the highest-return optimisation on the page and it takes one predicate.

!!! warning "When not to use the hybrid at all"

    Under about 100 posts per second with no account above 100,000 followers, run pure push with active-follower filtering and stop. The hybrid's read-time merge is a permanent complexity tax on every feed read. The measurement that justifies it is fan-out duration crossing your freshness objective, which is a number you can watch rather than predict.

### Deep dive two: pagination and duplicate suppression on a mutating list

The dives vary by problem on purpose. The crawler's second dive was deduplication of content; here it is deduplication of what the reader has already seen, which is a different problem with a different economy.

**Chronological feeds paginate cleanly.** A cursor of (timestamp, post identifier) is stable because new items arrive at the head, never in the middle. Page two is "everything older than this cursor". Offset pagination is wrong here for the usual reason: an insert at the head shifts every offset, so the reader sees an item twice.

**Ranked feeds break that guarantee.** If the ranking changes between the request for page one and the request for page two, items move across the page boundary. The reader sees duplicates, gaps, or both, and neither is recoverable at the client.

The fix is to freeze the ranking for the session:

| Element | Design | Cost |
|---|---|---|
| Session feed | On the first request of a session, materialise a ranked list of about 500 identifiers keyed by user and session | 5 million concurrent sessions times 500 times 8 bytes equals 20 GB |
| Cursor | An offset into the frozen list, not into the live data | Cheap, and it makes page two a slice |
| Head refresh | New posts since session start are prepended as a separate lane the client can pull explicitly | Keeps freshness without disturbing the frozen ordering |
| Expiry | Session feeds expire after roughly 30 minutes of inactivity | Bounds the 20 GB above |

The 20 GB figure is what makes this feasible; if it had come out at 20 TB, the answer would have to be an approximation instead.

**Suppressing already-seen items across sessions.** Exact tracking is too expensive: 300 million users times 1,000 recent post identifiers times 8 bytes is 2.4 TB. A per-user Bloom filter of 1,000 items at a 1 percent false positive rate needs about 9,600 bits, which is 1.2 KB per user, or 360 GB in total. That fits.

The trade-off is worth stating precisely, because it is the reason a Bloom filter is right here and wrong in the crawler's seen-set. A false positive suppresses a post the user has not seen. The user never learns that the post existed, so the error is invisible and costs one item in a hundred.

In the crawler, the same error silently drops a page from the index. It is equally invisible, but there it accumulates permanently into a measurable coverage gap. Same mechanism, different acceptability, decided by whether the loss is recoverable.

**Unfollow and delete correctness on a pushed feed.** Materialised identifiers can outlive the reason they were written. Two options:

| Approach | Mechanism | Cost |
|---|---|---|
| Filter at read | Check each hydrated post against the current follow graph and visibility rules | Adds a check to every read; this is unavoidable if privacy changes must be strict |
| Repair at write | On unfollow, enqueue a job to remove that author's entries from that follower's list | Cheap on average, unbounded for an unfollow of a heavily posted account |

Filtering at read is usually right, because you are already hydrating each post and the visibility check rides along on data you fetched anyway.

!!! warning "When not to build session-pinned feeds"

    If the feed is chronological, do not build this. A time-plus-identifier cursor is correct, stateless, and free. The measurement that justifies session pinning is a duplicate-item rate per session that you can actually observe in client telemetry, not a theoretical concern about reordering.

### Failure modes and the operating contract (feed)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Hot key | A viral post's content row is hydrated 52,000 times per second from one cache shard | Local cache tier with a short time to live, bounding per-key shared cache load |
| Thundering herd | A major event makes everyone refresh in the same 10 seconds | Client-side jittered refresh, and a stale-while-revalidate policy on the feed list |
| Metastable failure | Fan-out lag grows, workers retry, retries add load, lag grows further and does not recover after the trigger passes | Cap the retry budget as a fraction of successful throughput, and shed the lowest-priority fan-out first |
| Cache stampede | A celebrity post expires from the hydration cache | Probabilistic early expiry on hot content keys |
| Split brain on the follow graph | An unfollow is applied on one replica while fan-out reads another, so the post is delivered anyway | Read-time visibility filter makes this self-correcting; that is a second reason to prefer filtering over repair |
| Clock skew | Posts sorted by wall clock across shards interleave incorrectly | Sort by a monotonic post identifier with an embedded timestamp, never by a server's local clock |

Service level indicators:

- Feed read latency at p50 and p99, segmented by followee-count percentile. The segmentation is the point; an unsegmented p99 hides the fan-in problem entirely.
- Freshness, measured as publish time to visible time at the p99 follower.
- Fan-out queue lag, at p50 and p99, with the celebrity path excluded so it does not mask ordinary regressions.
- Empty feed rate, which catches materialisation failures that latency metrics miss.
- Duplicate item rate per session.

Alert on fan-out lag p99 crossing the freshness objective, on empty feed rate deviating from its baseline, and on feed read p99 for the top followee-count decile rather than for the whole population.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Feed read p99 under 200 ms | Yes: one range read plus one batched hydration plus a bounded celebrity merge |
| Freshness under 30 s at p99 | Yes by construction; the threshold of 600,000 followers was derived from this number |
| 52,000 reads per second at peak | Yes, and it scales with feed-list shards rather than with followee counts |
| Unfollow takes effect | Yes, via the read-time filter, with a short window where a post is fetched and then filtered |
| 4.8 TB of feed state | Reduced to roughly 40 percent of that by active-follower fan-out |

### Where this answer is a simplification (feed)

- **The ranking is the product.** Everything above is the delivery substrate. A real home timeline is a candidate generation and ranking system with integrity filters, diversity constraints and advertising insertion, and those decide the user experience far more than fan-out does.
- **"Feeding Frenzy: Selectively Materializing Users' Event Feeds"** (Silberstein and colleagues, SIGMOD 2010) is the primary academic treatment of the push, pull and hybrid decision, and it formalises the per-edge crossover argument used above. It is the single best thing to read after this page.
- **Facebook's TAO paper**, [TAO: Facebook's Distributed Data Store for the Social Graph](https://www.usenix.org/conference/atc13/technical-sessions/presentation/bronson) (Bronson and others, USENIX ATC 2013, accessed August 2026), describes the graph store behind this class of read path and is the clearest published source on hydration at scale.
- **LinkedIn published a description of FollowFeed** on its engineering blog in 2016 that argues explicitly for the pull side of the trade-off, which is a useful counterweight to the assumption that push always wins. Search for it by title; the post has moved between blog paths since publication.
- **Twitter's timeline architecture talk**, [Timelines at Scale](https://www.infoq.com/presentations/Twitter-Timeline-Scalability/) (Raffi Krikorian, QCon San Francisco 2012, published on InfoQ April 2013, accessed August 2026), is the widely cited primary account of a production hybrid.
- **Multi-surface reality.** Home feed, profile, search and notifications all need similar materialisation, and the cost of building four of these separately is the actual architectural problem inside a large company.

### Level delta: the same prompt at three levels (feed)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Draws fan-out on write immediately | Starts at pull and shows the number where it fails | Asks whether the feed is ranked before designing anything, because that answer changes the pagination design entirely |
| Hybrid threshold | "Celebrities are special" | Picks a threshold and defends it | Derives 600,000 from the freshness objective and the per-author budget, then offers to move it by renegotiating freshness |
| Cost argument | "Push is too expensive for celebrities" | Compares push and pull operation counts | Shows push is still cheaper on operations and loses on latency and burst isolation, correcting the common claim |
| Inactive users | Not mentioned | Filters fan-out to active followers | Prices the returner backfill at 231 reads per second to prove the filter is safe, rather than asserting it |
| Pagination | Offset | Cursor | Session-pinned materialisation for the ranked case, with the 20 GB sizing that makes it viable |
| Consistency | Not raised | Mentions eventual consistency | Chooses read-time filtering over write-time repair and gives split brain as the second, independent reason |
| Observability | Latency and errors | Adds fan-out lag | Segments feed latency by followee-count percentile, because the unsegmented number hides the failure mode being designed against |

What each level added:

- **Mid to senior:** the design is derived from a read-to-write ratio and a measured degradation curve instead of from a remembered pattern.
- **Senior to staff:** thresholds become negotiable levers tied to stated objectives, a widely repeated claim gets corrected with arithmetic, and the metric definitions are designed alongside the system.

---

## Case study 4: Chat and presence

### The prompt (chat)

"Design a messaging system supporting one-to-one and group conversations, with delivery receipts and online presence, across a user's multiple devices."

### Clarifiers that fork the design (chat)

| Question worth asking | If the answer is A the design becomes | If the answer is B the design becomes |
|---|---|---|
| Is the content end-to-end encrypted? | Server-readable: the server can deduplicate, search, moderate and re-deliver history to a new device | End-to-end: the server stores opaque blobs, cannot search or moderate content, and a new device generally cannot receive old history |
| How many devices per user? | One: a session is a connection, and read state is a single cursor | Several: read state is per device aggregated to per user, and the session registry maps a user to a set |
| What is the largest group? | Hundreds: per-conversation ordering with a single writer is trivially cheap | Hundreds of thousands: a single-writer conversation becomes a hot partition and ordering must relax |
| Is message ordering required to be identical for all participants? | Per conversation only: a sequence number from the partition owner gives it for free | Causally consistent across conversations: you need vector or hybrid logical clocks, and it is a much larger system |
| How much history is retained? | Days: storage is a rounding error and catch-up is bounded | Forever: 730 TB per year at our scale, with tiering and a cold path |
| Is presence a core feature or a nicety? | Nicety: poll it lazily when a contact is on screen | Core, with sub-second accuracy: presence traffic exceeds message traffic by two orders of magnitude, as derived below |
| Must delivery be exactly once? | At least once with client deduplication: simple and robust | Exactly once end to end: you are promising something the network cannot give you; reframe it as idempotent delivery |

### Requirements (chat)

Functional:

- Send a message to a one-to-one or group conversation, and deliver it to every participant's devices.
- Report sent, delivered and read states to the sender.
- Show whether a contact is online.
- Let a device that has been offline catch up without losing or duplicating messages.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Daily active users | 100 million | ASSUMED; ask, because this number sets the connection tier |
| Messages sent per user per day | 40 | ASSUMED; messaging is high frequency compared with posting |
| Share of users connected at peak | 30 percent | ASSUMED; a chat app holds a connection while the app is foregrounded or a push socket is alive |
| Contacts per user | 200 | ASSUMED; used only for the presence calculation |
| Send to delivered latency | under 500 ms at p99 for an online recipient | ASSUMED; above this, conversation feels broken |
| Message durability | no acknowledged message is ever lost | GIVEN; this is the one property users notice immediately |
| Duplicate delivery | permitted, bounded, and invisible to the user | GIVEN by the at-least-once branch; state it as a design property, not a bug |
| Peak messages per second | 139,000 | DERIVED below |

### Numbers that eliminate options (chat)

| Calculation | Result | What it rules out |
|---|---|---|
| 100e6 users times 40 messages divided by 86,400 | 46,000 messages per second, 139,000 at 3x peak | Rules out a single message store partition, and rules out any per-message coordination across the cluster |
| 46,000 times (0.95 one-to-one plus 0.05 group of 50) | 159,000 deliveries per second | Rules out modelling a group message as N separate stored messages. Store once per conversation, deliver by reference |
| 30 percent of 100e6 | 30 million concurrent connections | Rules out running connection handling inside the same process as message storage |
| 30e6 connections times 10 KB per idle connection | 300 GB of connection state fleet-wide | ASSUMED per-connection cost for a tuned event-driven server with small buffers; rules out holding all connections on a handful of machines |
| 300 GB at 500,000 connections per node | about 60 gateway nodes | Rules out a fixed small gateway tier and forces a session registry, since no single node knows where a recipient is |
| 100e6 users times 20 connectivity transitions per day, divided by 86,400 | 23,000 presence transitions per second | ASSUMED transition count, because mobile networks flap constantly; on its own this looks manageable |
| 23,000 transitions times 200 contacts | 4.6 million presence events per second | Rules out broadcasting presence to all contacts. This is 100 times the message rate, so presence would be the dominant workload in the entire system |
| 4e9 messages per day times 500 bytes | 2 TB per day, 730 TB per year | Rules out a single-tier hot store, and forces time-partitioned storage with tiering |
| 50 percent of recipients offline, times 4e9 messages, times 500 bytes | 1 TB of pending state | Rules out a separate per-recipient offline queue. The conversation log is already durable, so make it the queue and delete a component |
| 46,000 messages per second across 4,096 conversation partitions | 11 messages per second per partition | Rules out any objection to single-writer ordering per conversation. It is nearly free |

### V0: the smallest design that meets the requirements (chat)

??? note "Predict first: V0 has no message queue. Should it? Say why before reading on."

    No. A queue would be a second durable store holding the same messages the conversation log already holds, with its own retention and its own failure modes. The log is ordered, durable and replayable by cursor, which is everything a queue would have provided. Adding a queue here is the most common unnecessary box in this problem.

```
+----------+     +----------------------+     +----------------+
| Device   | <=> | Gateway plus chat    | --> | Messages store |
| websocket|     | connections in RAM   |     | key conv id    |
+----------+     | assigns seq per conv |     | plus seq       |
                 +----------------------+     +----------------+
                          |
                          v
                 +----------------------+
                 | Push provider        |
                 | for offline devices  |
                 +----------------------+
```

**Caption.** V0 holds websocket connections in one process, assigns a per-conversation sequence number on write, appends to a messages store keyed by conversation and sequence, and asks a push provider to wake devices that are not connected.

Correct at: 50,000 concurrent connections (about 500 MB of connection state at the assumed 10 KB each) and 500 messages per second, which is a trivial insert rate.

Three properties V0 already has, which is why the later versions are additions rather than rewrites:

- **Ordering is solved.** One partition owner assigns the sequence number, so there is one writer per conversation and therefore a total order, with no consensus protocol involved.
- **The offline queue is solved.** A device that reconnects sends its last known sequence number and asks for everything after it. There is nothing to queue.
- **Duplicates are solved at the client.** The client generates a message identifier before sending; the server enforces uniqueness of that identifier within the conversation, and the client discards anything it already has.

!!! warning "When not to separate the gateway from the chat service"

    Below roughly 100,000 concurrent connections, keep them in one process. The split costs you a network hop on every delivery and a session registry that is now a hard dependency for message delivery. The measurement that justifies the split is connection memory or file-descriptor pressure forcing you to scale the connection tier independently of the write tier.

### Breaking points, and what each version adds (chat)

#### Breaking point one, then V1 (chat)

**What fails: 30 million connections cannot live on one node, and once they are spread, the sender's gateway does not know which gateway holds the recipient.**

How you observe it: connection count per node against its memory ceiling, then, after a naive horizontal split, a delivery success rate that falls to roughly one over N as messages land on gateways that do not hold the recipient.

```
+---------+   +-------------+     +------------------+
| Device  |<=>| Gateway     | --> | Chat service     |
+---------+   | holds 500k  |     | assigns seq      |
              | connections |     | per conversation |
              +-------------+     | routes delivery  |
                    |             | back to gateways |
                    |             +------------------+
                    v                 |          |
                    |                 v          v
             +---------------------------+  +--------------+
             | Session registry          |  | Messages     |
             | user to device to gateway |  | store        |
             | written by the gateways   |  | partitioned  |
             | read by the chat service  |  | by conv id   |
             +---------------------------+  +--------------+
```

**Caption.** V1 separates a stateful gateway tier holding connections from a stateless chat service that assigns sequence numbers, writes to a partitioned message store, and routes delivery back through gateways; a session registry, written by gateways and read by the chat service, maps each user and device to the gateway currently holding it.

| Change | Justified by | Cost you accept |
|---|---|---|
| Gateway tier separated | 60 nodes of connections, scaled independently of write capacity | An extra hop on every delivery |
| Session registry | Delivery needs to find the recipient's gateway | A hot lookup on every delivery, and a component whose staleness causes missed pushes |
| Store partitioned by conversation | 139,000 messages per second at peak | Cross-conversation queries become expensive, which is fine because nobody needs them |

!!! warning "When not to add a session registry"

    With a single gateway node, or with consistent hashing from user identifier directly to gateway, you do not need one. Direct hashing is simpler but it moves every affected user's connection whenever the gateway fleet resizes. The measurement that justifies the registry is reconnection churn during deploys exceeding what your clients tolerate.

#### Breaking point two, then V2 (chat)

**What fails: the second device.**

How you observe it: unread counts that differ between a user's phone and laptop, messages that stay bold on one device after being read on another, and support reports of "my desktop is missing messages from yesterday". None of these show up in latency or error metrics, which is why this failure is usually found by users rather than by dashboards.

V2 makes the device, not the user, the unit of state:

- The session registry maps a user to a set of (device, gateway) pairs.
- Read state becomes a cursor per (device, conversation), with the per-user state derived from the set.
- Delivery targets every currently connected device, and every disconnected device catches up by cursor on reconnect.

!!! warning "When not to build per-device sync"

    For a single-device product with under about 10,000 concurrent users, one connection plus "give me everything since sequence N" on reconnect is the whole design. Per-device cursors and receipt aggregation are over-engineering until you can measure a second device per user in your own telemetry.

### Deep dive one: delivery ordering

Chat makes ordering interesting in a way the other three problems on this page do not: three different orderings exist and users notice when they disagree.

| Ordering | Defined by | Who observes it |
|---|---|---|
| Send order | The sending client's own actions | The sender, who expects their messages to stay in the order they typed them |
| Arrival order | When the server accepted each message | Nobody directly, but it decides the sequence number |
| Display order | How the client sorts what it holds | Every participant, and it is the only one that matters to the product |

**The mechanism: one writer per conversation.** The conversation identifier is the partition key, the partition owner assigns a strictly increasing sequence number, and that gives a total order within the conversation without a consensus protocol. At 4,096 partitions and 46,000 messages per second, each partition sees 11 messages per second, so the single-writer constraint costs nothing.

Global ordering across conversations is both expensive and useless, and saying so is a better answer than designing it. No user can observe the relative order of two messages in two different conversations well enough for it to matter.

**Push is a hint; pull by cursor is the authority.** This is the simplification that removes an entire class of problems:

| Without it | With it |
|---|---|
| The push stream must be reliable and ordered, so you need per-recipient acknowledgement and retransmission | The push carries "there is something new in conversation C"; the client fetches by cursor |
| A dropped push means a lost message | A dropped push means a slightly delayed message; the next push, or the next foreground event, repairs it |
| Reconnection needs a replay protocol | Reconnection is the same cursor fetch as everything else |

**Client-side ordering rules, stated exactly:**

- Sort by server sequence number, never by any clock. A client wall clock is attacker-controlled and routinely wrong.
- Show the sender's own message optimistically at the tail, marked pending, then re-sort when the server returns its sequence number. Users tolerate a message moving; they do not tolerate a delay before it appears.
- Deduplicate on the client-generated message identifier. Both a network retry and a session-registry split brain produce a second copy, and both are handled by the same three lines of client code.

**Where this model does not stretch.** Causal ordering across conversations, the case where someone says "check the group chat" and the referenced message has not arrived yet, is not solved by per-conversation sequence numbers. The Matrix specification models room history as a directed acyclic graph of events precisely to handle ordering under partition, and Lamport's "Time, Clocks, and the Ordering of Events in a Distributed System" (Communications of the ACM, 1978) is the foundational reading.

!!! warning "When not to use single-writer per-conversation sequencing"

    For a broadcast channel with a million members and a high message rate, the single writer becomes a hot partition. There, relax to per-sender ordering with a client-side merge, and accept that two participants may see two senders' messages in different relative orders. The measurement that justifies relaxing is a single conversation partition exceeding a meaningful fraction of one node's write capacity.

### Deep dive two: multi-device sync and offline catch-up

The second dive differs from the news feed's pagination dive because the constraint is different: there the list was mutating under the reader, here the list is stable and the reader is intermittent across several devices.

**The wrong model and the right one.**

| Model | Storage | Failure mode |
|---|---|---|
| A queue per device holding undelivered copies | Messages duplicated per device, plus 1 TB of pending state | Server must track per-device delivery, and a new device has nothing |
| One log per conversation, one cursor per device | Messages stored once; cursors are 100e6 users times 3 devices times 50 active conversations times 24 bytes, which is 360 GB | Catch-up cost must be bounded, see below |

The cursor model stores messages once and makes every device's state a small pointer. The 360 GB figure is what makes it obviously correct compared with duplicating the message body per device.

**Bounding catch-up.** A device offline for three days must not replay three days. Naive catch-up across 50 conversations at 500 messages each is 25,000 messages, which at 500 bytes is 12.5 MB on a mobile connection at app start. Instead:

| Rule | Value | Result |
|---|---|---|
| Conversations synced eagerly on reconnect | 20 most recently active | Bounds the breadth |
| Messages per conversation on catch-up | 100, newest first | Bounds the depth |
| Everything else | Fetched on demand when opened | Moves cost to the moment the user shows intent |

That is 2,000 messages, about 1 MB, which is a reasonable cold start. The remaining history stays reachable by cursor.

**Receipt aggregation, stated as rules because ambiguity here ships bugs.**

| Receipt | Rule | Why the alternative fails |
|---|---|---|
| Sent | The server has durably stored the message and assigned a sequence number | Anything earlier is a lie the sender will catch |
| Delivered | Any one of the recipient's devices has acknowledged receipt | Requiring all devices means a laptop left closed at home suppresses receipts forever |
| Read | Any one device has advanced its read cursor past the message | Same failure, and per-device read state is not something the sender should see |

**Push notifications are a separate, lossy channel.** The mobile push services are best effort by design and support collapse keys and time-to-live, both documented by their owners: Apple in [Sending notification requests to APNs](https://developer.apple.com/documentation/usernotifications/sending-notification-requests-to-apns) (Apple Developer Documentation, accessed August 2026) and Google in [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging) (accessed August 2026), which documents collapsible messages and message lifespan. A push receipt is not a delivery receipt. Treating it as one produces the bug where a message is marked delivered while the phone is in a tunnel with the app killed.

**The new-device problem, which encryption makes unsolvable rather than hard.** With server-readable content, a new device fetches history by cursor and it just works. With end-to-end encryption the server holds ciphertext addressed to key material the new device does not have, so history cannot be handed over by the server.

Signal's [Sesame specification](https://signal.org/docs/specifications/sesame/) (revision 2, April 2017, accessed August 2026) describes the multi-device session model, and it is the reason several encrypted products simply start a new device with no history. Say this in the room if the encryption clarifier came back as end-to-end; it demonstrates that you traced a requirement through to a product consequence.

!!! warning "When not to store per-device cursors"

    If the product has one device per account, store one cursor per user and delete the device dimension entirely. It halves the state and removes receipt aggregation completely. The measurement that justifies the device dimension is observed multi-device usage in your own telemetry, not an assumption that users will want it.

### Failure modes and the operating contract (chat)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Thundering herd | A gateway node dies and 500,000 clients reconnect within seconds, each issuing a catch-up query | Jittered reconnect backoff on the client, plus a connection admission rate limit at the gateway |
| Metastable failure | Reconnect storms slow catch-up queries, timeouts trigger more reconnects, and the system stays down after the original cause is gone | Shed catch-up requests before shedding sends, and cap concurrent catch-up per gateway |
| Split brain | Two gateways both believe they hold a user's session, so the message is delivered twice | Acceptable by design: the client deduplicates on message identifier. Say this out loud rather than designing it away |
| Hot partition | A single very large group concentrates writes on one partition owner | Relax to per-sender ordering for conversations above a size threshold |
| Retry storm | A send times out client-side while succeeding server-side, and the client retries | The client message identifier makes the retry idempotent; without it, retries create duplicates that survive deduplication |
| Clock skew | Messages sorted by device clock appear years out of order after a device sets its clock wrong | Never sort by clock; sequence number is the ordering authority and the timestamp is display metadata |
| Cache stampede | Presence for a popular contact expires and every viewer refreshes at once | Presence subscriptions with a shared refresh, plus a short randomised time to live |

Service level indicators:

- Send to delivered latency at p50 and p99, for online recipients only, since offline recipients measure the push provider rather than your system.
- Connection establishment success rate, and reconnect rate as a leading indicator of gateway trouble.
- Catch-up query latency at p99, which is the metric that goes bad first during a reconnect storm.
- Presence staleness at p99.
- Duplicate delivery rate, which should be small and non-zero. A reported zero means the metric is broken, because at-least-once delivery is the stated design.
- Message durability violations, target zero, measured by client-side gap detection in sequence numbers.

Alert on send to delivered p99, on gateway reconnect rate exceeding its baseline, on catch-up p99, and on any detected sequence gap.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Send to delivered under 500 ms at p99 | Yes for connected devices: one registry lookup, one gateway hop, one socket write |
| No acknowledged message lost | Yes, because acknowledgement follows durable write and assignment of a sequence number, and every reader is cursor-driven |
| Duplicates bounded and invisible | Yes, via client-side deduplication on the client-generated identifier |
| 139,000 messages per second at peak | Yes at 4,096 partitions, which is 34 per second per partition at peak |
| Presence without dominating the system | Yes, only by abandoning broadcast presence; subscribe-on-view is the design, and it is the direct consequence of the 4.6 million per second number |
| 730 TB per year | Requires time-partitioned storage with a cold tier; this is the requirement V2 addresses least completely, and saying so is better than pretending |

### Where this answer is a simplification (chat)

- **End-to-end encryption changes the system, not a component.** The server cannot search, cannot moderate content, cannot deduplicate by content, and cannot serve history to a new device. Signal's [Double Ratchet specification](https://signal.org/docs/specifications/doubleratchet/) and [Sesame](https://signal.org/docs/specifications/sesame/) (both accessed August 2026) are the primary sources, and RFC 9420 (July 2023) standardises Messaging Layer Security for group encryption at scale.
- **Storage engines matter more than the diagram suggests.** Discord published two detailed accounts of exactly this workload, [How Discord Stores Billions of Messages](https://discord.com/blog/how-discord-stores-billions-of-messages) (January 2017) and [How Discord Stores Trillions of Messages](https://discord.com/blog/how-discord-stores-trillions-of-messages) (March 2023), both accessed August 2026, and together they are the best public description of message-store evolution under real growth.
- **The connection tier is its own specialty.** Very high connection counts per machine are achievable, but treat any specific figure you have heard as evidence that a heavily tuned stack reached it once, not as a number you can assume. Size the connection tier from your own measured memory per idle connection.
- **Mobile networks are the real adversary.** Connections drop constantly, radio wake-ups cost battery, and the client's retry policy affects your server load more than any server-side choice.
- **Interoperability is now a regulatory input.** The EU Digital Markets Act obligations that took effect for designated gatekeepers in March 2024 require messaging interoperability, which changes the protocol surface from an internal choice to an external contract.
- **Matrix publishes its full specification** at spec.matrix.org, including the event graph and state resolution, and it is the most detailed openly readable design for federated chat ordering.

### Level delta: the same prompt at three levels (chat)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Message transport | Adds a message queue between sender and recipient | Uses the conversation log as the queue and explains why the extra component is redundant | Names the removal explicitly as the design's main simplification and shows it also solves offline catch-up |
| Ordering | "Use timestamps" | Per-conversation sequence numbers from a single writer | Adds that global ordering is unobservable and therefore not worth building, and states where single-writer breaks |
| Duplicates | Tries to guarantee exactly once | At least once plus client deduplication | Treats duplicate delivery as a published design property with an SLI, and points out a zero reading means the metric is broken |
| Presence | Broadcast to contacts | Notices the volume is large | Derives 4.6 million events per second, shows it is 100 times the message rate, and redesigns to subscribe-on-view before writing any other box |
| Multi-device | Not addressed | Per-device cursors | Defines receipt aggregation rules explicitly, because the ambiguity is where the bugs are, and traces the encryption clarifier through to no history on a new device |
| Reconnect | Not addressed | Backoff with jitter | Names metastable failure and specifies that catch-up sheds before sends, with a per-gateway concurrency cap |
| Honesty | Claims the design meets everything | Notes eventual consistency | States that 730 TB per year is the requirement the design covers least well, before the interviewer finds it |

What each level added:

- **Mid to senior:** components get removed rather than added, and the delivery guarantee is stated as something achievable rather than as a wish.
- **Senior to staff:** one estimate (presence volume) reorders the whole design, ambiguous product semantics get pinned down as explicit rules, and the weakest part of the answer is volunteered rather than defended.

## Case study 5: A video platform

### The prompt (video)

"Design a video platform: people upload videos, and other people watch them."

### Clarifiers that fork the design (video)

Ask these to change the architecture, not to fill two minutes. Each row has a different answer on each side.

| Question worth asking | If the answer is A, the design becomes | If the answer is B, it becomes |
|---|---|---|
| Is this user uploads at arbitrary length, or a curated catalogue? | A, user uploads: the transcode fleet is the dominant cost and publish latency is a product promise | B, catalogue: encoding is a one-time offline job and the whole interview is delivery |
| On demand, or live? | A, on demand: transcode is a batch job that may retry | B, live: transcode is a streaming pipeline with a glass-to-glass budget and no retries |
| Is publish latency measured in minutes or hours? | A, minutes: priority lanes and partial publish, where the lowest rung goes live first | B, hours: one batch pass at the cheapest encoder settings, no lanes |
| Do we keep the uploaded master forever? | A, keep: cold storage dominates the storage bill but re-encoding to a new codec is possible | B, discard after ladder: cheaper, and a codec migration means re-uploading |
| Is playback gated by entitlement or digital rights management? | A, gated: a token service sits on the hot path and the cache key must include entitlement | B, public: segments are immutable public objects and cache 100 percent |
| What share of watch time lands on the most popular 1 percent of videos? | A, heavy skew: the edge cache carries the load and origin stays small | B, flat: origin egress dominates and you need regional origins, not just a cache |
| Is the audience one country or global? | A, one: single region origin, one content delivery network | B, global: regional origins, and storage placement becomes a latency decision |

### Requirements (video)

Functional:

- Resumable upload of a source file.
- Transcode to a multi-rung bitrate ladder plus a manifest.
- Publish, then adaptive-bitrate playback with seek.
- Delete a video, including from the edge.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Uploads per day | 500,000 | GIVEN |
| Average source length | 10 minutes | ASSUMED, because a platform mixing short and long form sits in this range and the answer is insensitive to 8 versus 12 |
| Daily active viewers | 100,000,000 | GIVEN |
| Watch time per active viewer | 40 minutes per day | ASSUMED, chosen so the reader can rescale linearly |
| Publish latency | p50 under 10 minutes, p95 under 30 minutes | GIVEN |
| Playback start time | p95 under 2 seconds | GIVEN |
| Acknowledged uploads lost | zero | GIVEN |
| Average delivered bitrate | 2.5 Mbps | ASSUMED, a blend of mobile and desktop rungs; stated separately so it can be changed |

### Numbers that eliminate options (video)

Every number below is followed by what it removes from the table.

1. Upload arrivals: 500,000 divided by 86,400 seconds is 5.8 per second; at an assumed 3x diurnal peak that is 17 per second. **Eliminates** sharding the upload metadata database and putting a queue in front of the upload API. Seventeen inserts per second is one modest database.
2. Source minutes per day: 500,000 uploads times 10 minutes is 5,000,000 source-minutes.
3. Transcode compute: ASSUME 4 CPU-minutes to produce the whole five-rung ladder from one source-minute, because each rung encodes faster than real time on a single core at these resolutions and five rungs sum to a small multiple of real time. That is 20,000,000 CPU-minutes per day, or 20,000,000 divided by 1,440, about 13,900 cores running continuously. **Eliminates** transcoding inside the upload API process, and eliminates transcoding on demand at playback time.
4. Output bytes: a ladder of 0.3, 0.7, 1.2, 2.5 and 4.5 Mbps sums to 9.2 Mbps, which is 9.2 times 60 divided by 8, or 69 MB per source-minute. Times 5,000,000 gives 345 TB per day, about 126 PB per year. **Eliminates** a single hot storage tier and eliminates generating every rung for every video regardless of demand.
5. Delivery: 100,000,000 viewers times 40 minutes is 4,000,000,000 watch-minutes per day. At 2.5 Mbps that is 18.75 MB per watch-minute, so 75 PB per day, which is 868 GB per second, about 6.9 Tbps averaged. **Eliminates** serving playback from origin under any plausible origin footprint.
6. Origin egress after caching: at a 95 percent edge hit ratio origin carries 5 percent of 6.9 Tbps, about 345 Gbps. At 90 percent it is 690 Gbps. **Eliminates** treating hit ratio as an implementation detail: five points of hit ratio doubles the origin tier.
7. Read to write ratio: 4,000,000,000 watch-minutes against 5,000,000 source-minutes is 800 to 1. **Eliminates** any design that makes the read path more complex to shave latency off the write path.

!!! tip "The offer to make in the room"

    Say the ratio out loud before the arithmetic: "this is an 800 to 1 read to write system, so I am going to spend most of my time on delivery unless you want the pipeline." That single sentence buys you the interviewer's routing decision and costs fifteen seconds.

### V0: the smallest design that meets the requirements (video)

```
+----------+     +---------------+
| Uploader |---->| Upload API    |----+
+----------+     +---------------+    |
                                      |
+----------+     +---------------+    |
| Viewer   |---->| Play API      |----+
+----------+     +---------------+    |
                                      |
       +------------------------------+------------------+
       | Metadata database: videos, job state, manifests |
       +------------------------------+------------------+
                                      |
       +------------------------------+------------------+
       | Object store: source master and ladder segments |
       +------------------------------+------------------+
                                      |
       +------------------------------+------------------+
       | Transcoder worker pool, polls job state in the  |
       | metadata database                               |
       +-------------------------------------------------+
```

Caption: V0 for a video platform. The upload API writes the master to the object store and a pending row to the metadata database. A worker pool polls that table, reads the master, writes ladder segments back to the same bucket, and marks the row published. The play API returns a manifest whose segment URLs point straight at the object store. Five boxes, no queue, no content delivery network.

This satisfies every functional requirement. It is correct up to two computable limits.

- **Transcode limit.** One 32-core worker gives 32 times 1,440, or 46,080 CPU-minutes per day. Divided by the assumed 4 CPU-minutes per source-minute, that is 11,520 source-minutes, or about 1,150 uploads per day at 10 minutes each.
- **Delivery limit.** ASSUME the object store front end gives you 10 Gbps, which is 1.25 GB per second. One concurrent viewer at 2.5 Mbps consumes 0.3125 MB per second. That is about 4,000 concurrent viewers.

Below roughly 1,000 uploads a day and 4,000 concurrent viewers, adding a queue, a job orchestrator or a cache is work you cannot justify with a measurement.

### Breaking points, and what each version adds (video)

**Break one, at about 1,150 uploads per day.** The polling worker pool cannot drain the backlog, so job age grows without bound and p95 publish latency crosses 30 minutes. You observe it as *oldest pending job age*, not as queue depth: depth is ambiguous when workers are scaling, age is not.

**V1: split the pipeline from the API and chunk the work.**

```
+----------+    +------------+    +-------------------+
| Uploader |--->| Upload API |--->| Object store      |
+----------+    +-----+------+    | master            |
                      |           +-------------------+
                      |
                +-----+------+    +-------------------+
                | Job queue, |--->| Segmenter: cut    |
                | durable    |    | master into parts |
                +------------+    +---------+---------+
                                            |
                +---------------------------+---------+
                | Encoder workers, one task per part  |
                | per rung, autoscaled on job age     |
                +---------------------------+---------+
                                            |
                +---------------------------+---------+
                | Stitcher and packager: manifests,   |
                | then mark published                 |
                +-------------------------------------+
```

Caption: V1 transcode pipeline. The upload API only stores the master and enqueues one job. A segmenter cuts the master at keyframe boundaries, encoder workers process parts in parallel across every rung, and a stitcher assembles segments plus manifests before publishing.

Chunking is what buys the latency target: a 10 minute video split into 30 second parts is 20 parallel tasks per rung, so wall-clock publish time falls to roughly the time for one part plus fixed overhead, instead of the time for the whole file.

**Break two, at about 4,000 concurrent viewers.** Object store egress saturates. You observe it as segment time-to-first-byte climbing and the player's rebuffer ratio rising; playback start p95 crosses 2 seconds well before the store returns errors.

**V2: put a cache hierarchy in front, and make the objects cacheable.**

```
+--------+   +----------+   +--------------+   +-----------+
| Player |-->| Edge     |-->| Origin       |-->| Object    |
|        |   | cache    |   | shield cache |   | store     |
+--------+   +----------+   +--------------+   +-----------+
     |
     +-----> +--------------------------------------------+
             | Play API: manifest, entitlement, URL signer |
             +--------------------------------------------+
```

Caption: V2 delivery path. Players fetch immutable segments through an edge cache backed by a shared origin shield, so a cold segment is fetched from the object store once for the whole edge fleet rather than once per edge. Manifests come from the play API on a short time to live; segments never change and carry a long one.

**Break three, at catalogue scale.** From the sizing, full ladders for everything cost 126 PB per year, and most of it is never requested. You observe it as *bytes stored per byte delivered*, sliced by video age. When the tail rung of a two-year-old video has zero requests in 90 days, you are paying rent on nothing.

**V3: generate cold rungs on demand and tier the storage.** Keep the top two rungs and the lowest rung eagerly. Package rare rungs just in time on the first request, accepting a slower first play for a video nobody is watching. Move masters to archival storage after 30 days.

### Boxes not to add without a measurement (video)

| Component | The measurement that justifies it | Below this, it is over-engineering |
|---|---|---|
| Durable job queue | Oldest pending job age exceeds a third of your publish latency budget | Under a few hundred uploads a day, a status column and a polling worker is simpler and easier to debug |
| Chunked parallel encode | Single-file wall-clock encode time exceeds the p95 publish target | Videos under about 2 minutes finish inside the budget whole; chunking adds keyframe alignment bugs for nothing |
| Content delivery network | Origin egress above roughly a third of your provisioned link, or segment time-to-first-byte above 200 ms outside your region | A few thousand concurrent viewers in one region are served fine by the object store directly |
| Origin shield tier | Edge-to-origin request rate for the same object exceeds the edge count during a popularity spike | With one edge region, a shield is a second hop that adds latency and buys nothing |
| Just-in-time packaging | More than about 30 percent of stored rendition bytes have zero requests in 90 days | Small catalogues cost less to store eagerly than to build the packaging service |

### Deep dive one: chunk boundaries and the encoding ladder

This dive earns its place because chunked encoding is where a video pipeline is uniquely hard: the parallelism is only correct if every worker agrees on where the cuts are.

**Why cuts must be keyframe aligned.** A decoder can only start at a keyframe. If part boundaries land at different frames on different rungs, a player that switches rungs mid-stream has no common resume point and either stalls or shows a glitch.

**The rule.** Force a fixed group-of-pictures length, ASSUME 2 seconds, on every rung, and cut parts at multiples of it. Two seconds is a reasonable default because it bounds switch latency to one segment while keeping keyframe overhead small; measure the bitrate cost on your own content before committing.

| Choice | Publish latency effect | Quality and cost effect |
|---|---|---|
| Long parts, 5 minutes | Little parallelism, wall clock near serial | Encoder sees more context, slightly better rate control |
| Short parts, 10 seconds | High parallelism, wall clock near the fixed overhead | More per-task overhead, more keyframes, visible cost per rung |
| Parts at 30 seconds, keyframes at 2 seconds | 20-way parallelism on a 10 minute source | The default worth defending |

**The stitching trap.** Rate control decisions do not carry across an independently encoded part, so a two-pass encoder that targets an average bitrate per part will produce a stream whose bitrate wobbles at part boundaries. State the mitigation: pass a fixed quantizer target rather than a bitrate target per part, then check the assembled stream's peak bitrate against the rung's ceiling.

??? note "Check yourself: why can the segmenter not simply cut every 30 seconds by byte offset?"

    Because a byte offset lands in the middle of a compressed frame and almost never on a keyframe. The segmenter must parse the container and cut on a frame boundary that is a keyframe in the source, or re-encode a lead-in so it can create one.

**The ladder itself.** A fixed ladder wastes bits on simple content and starves complex content. Netflix published in December 2015 that it replaced a fixed ladder with a per-title ladder derived from running many encodes and choosing points on the resulting quality curve ([Per-Title Encode Optimization](https://netflixtechblog.com/per-title-encode-optimization-7e99442b62a2), Netflix Technology Blog, December 2015). Naming that trade in the interview, and then saying you would ship a fixed ladder first because per-title costs several extra encodes per video, is a stronger answer than proposing per-title immediately.

### Deep dive two: cache hit ratio as the economic centre of the design

The second dive is deliberately not another pipeline topic. Delivery is where the money is at 800 to 1, and hit ratio is the one variable that moves it.

**What determines hit ratio:**

| Lever | Effect on hit ratio | Cost of pulling it |
|---|---|---|
| Immutable segment URLs with a long time to live | Large positive, segments never revalidate | Requires content-addressed or version-suffixed paths, so a re-encode changes the URL |
| Cache key hygiene, stripping query parameters used for analytics | Large positive, prevents key fragmentation | Analytics must move to a separate beacon call |
| Signed URLs with per-user tokens | Large negative if the token is in the cache key | Put the token in a header or validate at the edge instead of fragmenting the key |
| Request coalescing at the edge | Positive during spikes only | Adds a small queueing delay on the first miss |
| Origin shield | Reduces origin fill by roughly the edge count for a spiking object | One extra hop, about 10 to 30 ms on a miss |

**The arithmetic to say out loud.** From the sizing, 6.9 Tbps of delivery at 95 percent hit is 345 Gbps of origin fill. Adding a shield in front of, ASSUME, 30 edge regions turns 30 independent cold fetches for a newly viral segment into one, which is the difference between a manageable spike and an origin brownout.

??? note "Check yourself: playback start p95 is 2 seconds and you are at 1.9. Which single change helps most?"

    Shorten the first segment. Start time is dominated by manifest fetch plus the first segment download at the initial rung. Making the first segment 1 second and starting on a low rung, then switching up, cuts the critical path more than any cache change, because the manifest and first segment are usually a cache miss for a first-time viewer anyway.

### Failure modes and the operating contract (video)

| Failure class | How it shows up here | Signal to alert on |
|---|---|---|
| Thundering herd | A video goes viral; every edge misses the same segment simultaneously and hits origin at once | Origin fill requests per unique object per minute |
| Retry storm | A malformed source crashes an encoder, the task requeues, the pool burns capacity on one poison video | Task attempts per job, alert above 3 |
| Metastable failure | Backlog grows, autoscaler adds workers, workers contend on the object store, throughput drops, backlog grows faster | Ratio of completed tasks to started tasks, falling below 1 for more than 10 minutes |
| Hot key | One manifest for a live premiere is requested by millions in the same second, and it has a short time to live by design | Manifest requests per second per video |
| Clock skew | Signed segment URLs rejected at the edge because edge and signer clocks disagree | 403 rate on segment requests, sliced by edge region |

**Service level indicators to publish:** publish latency p50 and p95, playback start time p95, rebuffer ratio, segment error rate, edge hit ratio, oldest pending job age.

**Re-check against the requirements.**

- Publish p95 under 30 minutes is met by chunked encode plus autoscaling on job age; the failure mode that threatens it is the poison-pill retry storm, so the attempt cap is a requirement, not a nicety.
- Playback start p95 under 2 seconds is met by the edge cache plus a short first segment; it is threatened by manifest time to live choices, so manifest latency gets its own indicator.
- Zero acknowledged uploads lost is met by acknowledging only after the master is durably written, which is why the upload API writes to the object store before it writes the metadata row.

### Where this answer is a simplification (video)

- Hardware encoding changes the compute arithmetic entirely. Google published a paper on deploying custom video transcoding hardware at warehouse scale ([Warehouse-scale video acceleration: co-design and deployment in the wild](https://dl.acm.org/doi/10.1145/3445814.3446723), ASPLOS, April 2021). If your interviewer works on video, expect the CPU-minutes assumption to be challenged.
- Shot-aware encoding splits on scene changes rather than fixed intervals, which interacts badly with the fixed-interval chunking above. Netflix published its approach in March 2018 ([Optimized shot-based encodes](https://netflixtechblog.com/optimized-shot-based-encodes-now-streaming-4b9464204830), Netflix Technology Blog).
- The manifest formats are standards with real constraints. HTTP Live Streaming is specified in [RFC 8216](https://www.rfc-editor.org/rfc/rfc8216) (IETF, August 2017); the common segment container that lets one set of segments serve both major streaming formats is the Common Media Application Format, ISO/IEC 23000-19.
- Delivery at this scale is often not a rented content delivery network. Netflix documents its own appliance programme publicly at [Open Connect](https://openconnect.netflix.com/). Saying "at this scale you eventually build the cache" is fine; saying every large platform does is not.

### Level delta: the same prompt at three levels (video)

| Dimension | Mid | Senior | Staff |
|---|---|---|---|
| Framing | Lists upload, transcode, store, deliver and draws all four immediately | States the 800 to 1 read to write ratio first and asks which half to spend time on | Asks what the business is optimising: publish latency, cost per delivered hour, or catalogue breadth, and designs to that |
| Numbers | Estimates storage | Derives 13,900 cores and 6.9 Tbps, and uses each to remove an option | Adds cost per delivered hour and per published hour, and shows which architectural change moves each |
| Progression | Presents the finished architecture with a queue and a delivery network already drawn | Starts at V0, names the 1,150 uploads per day limit, then adds the queue | Also names what to remove: eager full ladders for the cold tail, and the second manifest format nobody uses |
| Depth | Mentions adaptive bitrate | Explains keyframe alignment and why chunk boundaries must match across rungs | Explains the rate-control discontinuity at part boundaries and the fixed-quantizer mitigation, and how it would be measured |
| Failure | Says "add retries" | Names the poison-pill retry storm and the attempt cap, plus the thundering herd and the shield | Names the metastable backlog, defines the completed-to-started ratio as the alert, and specifies the load shedding rule that breaks the loop |
| Ownership | Not mentioned | Notes that the pipeline and delivery are separate on-call surfaces | Draws the contract between them: the pipeline's output is an immutable, versioned segment set, so delivery never needs a pipeline deploy to roll back |

The mid answer is not wrong; it is undifferentiated. The senior delta is derivation and one genuine depth area. The staff delta is a stated objective function, a deletion, and an interface between teams.

---

## Case study 6: Ride hailing

### The prompt (ride hailing)

"Design the backend for a ride hailing app: riders request a ride and we assign them a nearby driver."

### Clarifiers that fork the design (ride hailing)

| Question worth asking | If the answer is A, the design becomes | If the answer is B, it becomes |
|---|---|---|
| One rider per trip, or shared rides? | A, one rider: matching is a nearest-driver selection | B, shared: matching becomes a routing problem with detour limits, and it needs an optimiser, not a query |
| Is the marketplace city-scoped or borderless? | A, city: shard by city, no cross-shard matching, no border logic | B, borderless: geographic partitioning with explicit handling of queries crossing shard edges |
| Does the driver app stay in the foreground? | A, yes: a fixed 4 second ping cadence is dependable | B, no: the operating system throttles background location, so cadence is adaptive and staleness is a first-class variable |
| May dispatch take 5 seconds to decide? | A, yes: batch the requests into a short window and solve assignment globally | B, no, under a second: greedy nearest only, and you accept worse global assignment |
| Can a driver hold two live offers at once? | A, no: you need a per-driver lease, and matching contains a serialised critical section | B, yes, broadcast and first accept wins: simpler server, worse driver experience, and an accept race to resolve |
| Do we need the full location trace for pricing or disputes? | A, yes: an archive path off the ingest pipeline | B, no: locations are ephemeral with a 60 second time to live, memory only |
| Does pricing consume the same supply data? | A, yes: the location pipeline needs a second consumer, not a second pipeline | B, no: pricing runs on trip outcomes and the pipeline stays single-purpose |

### Requirements (ride hailing)

Functional: driver goes online and streams location; rider requests a ride; the system offers the trip to one driver at a time; driver accepts or times out; trip state machine runs to completion; trip is archived.

| Requirement | Value | Tag |
|---|---|---|
| Concurrent online drivers at peak | 1,000,000 | GIVEN |
| Location ping cadence | every 4 seconds | GIVEN |
| Ride requests at peak | 5,000 per second | GIVEN |
| Request to first offer sent | p95 under 2 seconds | GIVEN |
| A driver holding two live offers | never | GIVEN |
| Location staleness at read time | p99 under 10 seconds | DERIVED: a 4 second cadence plus one missed ping plus a 2 second pipeline budget |
| Ping payload | 100 bytes | ASSUMED: driver id, latitude, longitude, heading, speed, accuracy and a timestamp fit comfortably in that |
| Candidates evaluated per request | 20 | ASSUMED: enough to survive several declines without widening the radius |

### Numbers that eliminate options (ride hailing)

1. Location write rate: 1,000,000 drivers divided by 4 seconds is 250,000 writes per second. **Eliminates** writing each ping into an indexed relational table, and eliminates a general purpose spatial database as the hot store.
2. Location bandwidth: 250,000 times 100 bytes is 25 MB per second, about 200 Mbps. **Eliminates** worrying about bandwidth at all. This number exists to stop you designing a compression scheme: the cost here is index maintenance and fan-out, not bytes on the wire.
3. Working set: 1,000,000 drivers times 100 bytes is 100 MB. **Eliminates** a distributed store for current position. Every driver's current location fits in one machine's memory many times over, so partitioning is for throughput and availability, never for capacity.
4. Dispatch read rate: 5,000 requests per second times 20 candidates is 100,000 candidate evaluations per second. **Eliminates** calling a road-network routing service synchronously per candidate: at an assumed 20 ms per call that is 2,000 concurrent calls sustained, which is a service you would have to build and staff. Prefilter by straight-line distance, then route only the top few.
5. Write to read ratio: 250,000 against 5,000 is 50 to 1, write heavy. **Eliminates** any index that does expensive work per update to make reads cheaper. You need an update that is a constant-time cell reassignment.
6. Archive volume: 250,000 pings per second times 100 bytes is 2.16 TB per day raw. **Eliminates** keeping raw traces in the serving store. Archive compressed and downsampled, off the hot path.

### V0: the smallest design that meets the requirements (ride hailing)

```
+-----------+    +----------------+
| Driver    |--->| Location API   |----+
| app ping  |    +----------------+    |
+-----------+                          |
                                       |
+-----------+    +----------------+    |
| Rider app |--->| Dispatch API   |----+
+-----------+    +--------+-------+    |
                          |            |
      +-------------------+------------+-------------+
      | In-memory store: driver id to cell, cell to  |
      | driver set, 60 second time to live per key   |
      +----------------------------------------------+
                          |
      +-------------------+--------------------------+
      | Relational database: trips, users, offers    |
      +----------------------------------------------+
```

Caption: V0 for ride hailing. Location pings update two keys in a single in-memory store: the driver's cell and the cell's member set. Dispatch reads the cells covering the request circle, sorts by straight-line distance, and writes the trip and offer rows to a relational database. Four boxes, no queue, no shards.

Correct up to a computable limit. ASSUME a single-threaded in-memory store sustains 100,000 simple operations per second, which is a deliberately conservative planning number for one core doing set membership work. Dispatch consumes 5,000 requests times 4 cell reads, or 20,000 operations per second, leaving 80,000 for writes. Each driver writes 0.25 operations per second, so 80,000 divided by 0.25 is 320,000 concurrent drivers.

Below about 300,000 concurrent drivers in one city cluster, sharding the location store is work you cannot justify.

### Breaking points, and what each version adds (ride hailing)

**Break one, at about 320,000 concurrent drivers.** The location store saturates one core. You see command latency p99 climb first, then staleness: instrument staleness as *now minus the ping timestamp at the moment dispatch reads it*, as a histogram, because that is the number in the requirements and store latency is only a proxy for it.

**V1: partition the location store geographically, and batch the ingest.**

```
+-----------+   +---------------------+
| Driver    |-->| Ingest tier:        |
| app pings |   | validate, buffer    |
+-----------+   | 1 second per shard  |
                +----------+----------+
                           |
   +-----------+-----------+-----------+
   |           |           |           |
+--+-----+ +---+----+ +----+---+ +-----+--+
| Shard  | | Shard  | | Shard  | | Shard  |
| region | | region | | region | | region |
| A cells| | B cells| | C cells| | D cells|
+--------+ +--------+ +--------+ +--------+
   |           |           |           |
   +-----------+-----+-----+-----------+
                     |
      +--------------+----------------+
      | Dispatch service: routes the  |
      | query circle to owning shards |
      +-------------------------------+
```

Caption: V1 location pipeline. A stateless ingest tier buffers pings for one second per destination shard and applies them as one batched write, cutting round trips by the batch factor. Each shard owns a contiguous set of geographic cells and holds them in memory. Dispatch computes which cells the query circle covers and fans out only to the shards owning them.

Partitioning geographically rather than by driver id hash is the decision worth defending: a radius query touches a contiguous region, so geographic ownership makes the fan-out one or two shards instead of all of them.

**Break two, double offers.** Two dispatch instances handling two riders in the same neighbourhood both select the same driver within the same few hundred milliseconds. You see it as a rising rate of "offer rejected, driver already engaged" and as riders who see a driver assigned then unassigned. It is not rare: at 5,000 requests per second in dense areas, candidate overlap is the normal case.

**V2: a per-driver offer lease, owned by the shard that owns the driver.** Dispatch must acquire the lease before it sends an offer. The lease has a time to live slightly longer than the driver's accept window, so a crashed dispatcher cannot strand a driver.

**Break three, the scarcity spiral.** When supply is short, every request finds no candidates, widens its radius, scans more cells, costs more CPU, takes longer, and riders retry. Load rises because the system is failing, which makes it fail more. That is a metastable failure, and it does not clear when the original trigger goes away. You see it as *requests per completed match* climbing and mean scan radius climbing together.

**V3: admission control.** Cap the scan radius. Return "no drivers nearby" quickly rather than scanning further. Add a rider-side queue with a stated position so retries stop being client-driven. Shed the lowest-value request class first.

### Boxes not to add without a measurement (ride hailing)

| Component | The measurement that justifies it | Below this, it is over-engineering |
|---|---|---|
| Sharded location store | Store CPU above 60 percent, or staleness p99 above half the budget | Under about 300,000 concurrent drivers, one memory store is faster and far easier to reason about |
| Ingest batching tier | Round trips per second per shard above what the shard can accept, at the same payload volume | If pings already fit, batching adds up to a second of staleness for no gain |
| Per-driver offer lease | Measured double-offer rate above roughly 0.1 percent of offers | If dispatch runs as a single instance per city, mutual exclusion is free and a lease is a distributed lock you do not need |
| Batch matching window | Assignment quality measurably worse than a solver would give on the same candidate set | In thin markets there is usually one plausible driver, so batching only adds delay |
| Stream platform on the ingest path | A second independent consumer of the location feed exists today | With one consumer, a queue between two services you own is a hop that can fail |

### Deep dive one: the location update pipeline

Chosen because 250,000 writes per second against a 100 MB working set is an unusual shape: it is a throughput problem with no capacity problem, and that inverts the usual instincts.

**Why the pipeline is not a message queue on the hot path.** Dispatch needs the *current* value, not the history. A log gives ordering and replay, which cost latency and buy nothing for a value that is obsolete in 4 seconds. The design instead writes directly to the owning shard and forks a sampled copy into a stream for the archive and for supply aggregation.

```
+--------+   +---------+   +----------------+
| Driver |-->| Ingest  |-->| Location shard |
+--------+   +----+----+   | current value  |
                  |        +----------------+
                  |
                  +------->+----------------+
                           | Stream: trace  |
                           | archive and    |
                           | supply metrics |
                           +----------------+
```

Caption: the fork. The synchronous path updates only the current value in memory. The asynchronous path carries a copy to durable storage and to the aggregation jobs, so a stream outage degrades pricing and analytics but never dispatch.

**Adaptive cadence, and why it is a server decision.** A stationary driver at a rank does not need a 4 second ping. Send the cadence down to the client as policy, and have the client fall back to a fixed rate if it cannot reach the server.

| Driver state | Cadence | Effect on the 250,000 per second figure |
|---|---|---|
| On a trip, moving | 4 seconds | Unchanged, this is the case the requirement was written for |
| Online, idle, stationary over 2 minutes | 30 seconds | Removes most idle drivers from the write budget |
| Online, idle, moving | 8 seconds | Halves the idle moving population's writes |

If ASSUME 50 percent of online drivers are idle and stationary at any moment, moving them to 30 seconds cuts total writes from 250,000 to about 141,000 per second. That is a bigger win than any storage engine choice, and it costs one policy field.

??? note "Check yourself: why measure staleness at read time rather than logging ingest lag?"

    Because the requirement is about the value dispatch actually used. Ingest lag misses a driver whose phone lost signal for 20 seconds: the pipeline is healthy and the data is stale. Measuring at read time captures client-side gaps, network gaps and pipeline gaps in one number.

### Deep dive two: dispatch matching and the offer lease

This dive is a coordination problem, deliberately different from the throughput problem above. The interesting part is not finding nearby drivers; it is committing to one.

**Greedy versus a batch window.**

| Approach | Latency | Assignment quality | When to choose it |
|---|---|---|---|
| Greedy nearest, decide immediately | Lowest, meets a 2 second p95 easily | Locally good, globally poor: an early rider takes the driver a later, closer rider needed | Thin markets, or a hard sub-second requirement |
| Batch window of 3 to 5 seconds, then solve | Spends most of the 2 second budget, so the window must fit inside it | Better: solve the assignment across all requests and candidates in the window | Dense markets where several riders and drivers overlap in every window |

DoorDash published in February 2020 that its dispatch system batches deliveries and assignments rather than assigning each order on arrival ([Next-Generation Optimization for Dasher Dispatch](https://careersatdoordash.com/blog/next-generation-optimization-for-dasher-dispatch-at-doordash/)). Cite it as their published choice for their problem, not as a universal law: their batching tolerance comes from food preparation time, which a ride hailing rider does not have.

**The lease protocol, stated precisely:**

1. Dispatch selects a candidate and asks the owning shard to acquire the driver's lease with a time to live of the accept window plus a margin.
2. The shard grants the lease only if none is held. It is the single writer for that driver, so this is a local compare and set, not a distributed lock.
3. Dispatch sends the offer. On accept, it converts the lease into a trip assignment. On decline or timeout, it releases the lease and takes the next candidate.
4. If dispatch dies, the lease expires and the driver becomes available again.

**What breaks it.** If the shard's ownership of a driver moves during a rebalance, two shards can briefly both believe they own that driver, and both can grant a lease. That is split brain, and it is why ownership handoff must be a fenced operation: the new owner takes over only with a monotonically increasing ownership term, and leases granted under an older term are rejected.

??? note "Check yourself: the accept window is 15 seconds and the lease time to live is 20 seconds. A dispatcher crashes right after sending an offer. What does the driver see, and what does the rider see?"

    The driver still sees the offer and can accept, but the accepting write finds no live dispatcher to complete the trip, so the accept fails and the offer disappears at 15 seconds. The rider sees no assignment for 20 seconds until the lease expires and another dispatcher retries. The fix is to make the accept path write to the shard rather than back to a specific dispatcher, so any dispatcher can finish the trip.

### Failure modes and the operating contract (ride hailing)

| Failure class | How it shows up here | Signal to alert on |
|---|---|---|
| Hot key or hot shard | An airport or stadium cell holds a disproportionate share of drivers and every query hits one shard | Per-shard operations per second divided by fleet median, alert above 4 |
| Split brain | Two shards both grant a lease during ownership handoff | Lease grants rejected by ownership term, and observed double offers |
| Thundering herd | A deploy restarts the ingest tier and a million driver apps reconnect in the same few seconds | Connection establishment rate, alert on a step change |
| Retry storm | Rider clients retry on timeout during a slow period, multiplying the load that caused the slowness | Requests per completed match |
| Metastable failure | The scarcity spiral above, which persists after the trigger clears | Mean candidate scan radius trended against match rate |
| Clock skew | A driver's phone clock is minutes off, so staleness computed from the device timestamp is nonsense | Share of pings with a device timestamp outside a 60 second window of server time |

**Indicators to publish:** request to offer p95, location staleness p99 at read time, offers per completed match, double-offer rate, per-shard utilisation, lease acquisition failure rate.

**Re-check against the requirements.**

- The 2 second p95 survives only if the batch matching window is under about 1 second, which is why the window is a tuned number and not a constant.
- The never-two-offers requirement is met by the lease plus fenced ownership handoff; without fencing it is met only most of the time, and the interviewer should hear you say that.
- The 10 second staleness p99 is met by the 4 second cadence plus a 1 second ingest batch, with 5 seconds of margin for a missed ping, and adaptive cadence deliberately spends part of that margin on idle drivers where staleness does not matter.

### Where this answer is a simplification (ride hailing)

- Cell systems are a real engineering choice with published rationale. Uber open-sourced H3, a hexagonal hierarchical grid, and published why hexagons beat squares for movement analysis in June 2018 ([H3: Uber's Hexagonal Hierarchical Spatial Index](https://www.uber.com/en-US/blog/h3/)).
- Trip state is more than a state machine. Uber published a ground-up rewrite of its fulfilment platform describing the consistency problems in trip state in 2021 ([Uber's Fulfillment Platform](https://www.uber.com/blog/fulfillment-platform-rearchitecture-part-1/)).
- Getting an offer to a phone in under a second is its own system. Uber described its real-time push platform in 2022 ([Uber's Real-Time Push Platform](https://www.uber.com/blog/real-time-push-platform/)). The interview answer says "push the offer"; the real system has delivery receipts, reconnection and fallbacks.
- Straight-line distance is a poor proxy for arrival time across a river or a highway with no exit. Real dispatch scores candidates with a road-network estimate, which is why the sizing above rules out calling it for all 20 candidates and forces a two-stage filter.

### Level delta: the same prompt at three levels (ride hailing)

| Dimension | Mid | Senior | Staff |
|---|---|---|---|
| Framing | Draws driver app, rider app, matching service, database | Asks whether dispatch may take seconds, because that single answer decides greedy versus batch | Asks what the marketplace is optimising: rider wait, driver utilisation, or completion rate, and notes the design differs for each |
| Numbers | Says location writes are heavy | Derives 250,000 writes per second and, crucially, the 100 MB working set, then uses the second number to refuse a distributed store | Adds the adaptive cadence calculation showing a policy change beats an infrastructure change, with the cost saved |
| Progression | Presents shards and a stream platform immediately | Starts at one memory store, computes the 320,000 driver limit, then shards | Also names what to delete: the stream platform on the hot path, and the separate pricing pipeline that duplicates the same feed |
| Depth | "Use a lock so two riders do not get the same driver" | Specifies the lease with a time to live, owner-local compare and set, and the release path on decline | Specifies fenced ownership handoff with a monotonic term, and states the exact window in which the guarantee is violated without it |
| Failure | Mentions retries and timeouts | Names the scarcity spiral as metastable and proposes radius capping | Defines requests per completed match as the alert, sets the shedding order across request classes, and states the recovery procedure that does not depend on load falling |
| Evolution | Not mentioned | Notes that shards can be added | Sequences the migration from one store to shards with dual writes and a shadow read comparison, and names the blast radius of a bad cell assignment |

The senior delta is the 100 MB observation and the lease protocol. The staff delta is the fencing term, the deletion, and a migration plan for a system that is already running.

---

## Case study 7: Proximity search

### The prompt (proximity)

"Design a service that finds places near me, for example every restaurant within five kilometres."

### Clarifiers that fork the design (proximity)

| Question worth asking | If the answer is A, the design becomes | If the answer is B, it becomes |
|---|---|---|
| Do the indexed entities move? | A, no: the index is an offline artefact rebuilt on a schedule | B, yes: this is the ride hailing problem, and the answer is a mutable in-memory cell index |
| Fixed radius, or user-chosen? | A, fixed set of radii: precompute a result set per cell | B, arbitrary radius: compute a cell cover per query, which is the general case |
| Are there filters, such as open now, price and category? | A, yes: geography becomes candidate generation and a scoring stage follows | B, no: the geo index returns the final answer and is the whole system |
| Ranked by distance, or by blended relevance? | A, distance: the index ordering is the output ordering | B, blended: results must be re-sorted after retrieval, and pagination stability becomes a problem |
| Nearest N, or all within R? | A, nearest N: cost varies wildly with local density, so you expand rings | B, all within R: bounded work per cell, but an unbounded result set downtown |
| How many entities, and how fast do they change? | A, 200 million changing slowly: a periodic full rebuild is viable | B, live inventory changing per minute: incremental updates are mandatory and the structure choice changes |
| Is coverage uniform or city-skewed? | A, skewed: fixed-size grid cells overflow downtown and an adaptive structure wins | B, uniform: a flat grid is simpler and correct |

### Requirements (proximity)

Functional: search by latitude, longitude, radius and filters; return a ranked, paginated list; create, update and delete a place.

| Requirement | Value | Tag |
|---|---|---|
| Indexed places | 200,000,000 | GIVEN |
| Peak query rate | 50,000 per second | GIVEN |
| Latency | p99 under 100 ms | GIVEN |
| Updates | 500,000 per day | GIVEN |
| Freshness, create to visible | under 1 hour | GIVEN |
| Indexed bytes per place | 1 KB | ASSUMED: name, address, category set, hours, rating and identifiers fit in a kilobyte of indexed fields |
| Geo tuple bytes per place | 20 bytes | DERIVED: an 8 byte identifier plus two 4 byte projected coordinates plus 4 bytes of packed flags |

### Numbers that eliminate options (proximity)

1. Document size: 200,000,000 times 1 KB is 200 GB. **Eliminates** holding full documents in memory on every node.
2. Geo index size: 200,000,000 times 20 bytes is 4 GB. **Eliminates** partitioning the geometry for capacity. The entire spatial index fits in memory on one ordinary machine, so any sharding you propose must be justified by throughput, not size.
3. Query cost: ASSUME a five kilometre query touches a few hundred candidates and costs 5 ms of CPU, made up of a cell cover computation plus roughly 10 microseconds per candidate to filter and score. A 16 core node at 60 percent utilisation gives 9.6 core-seconds per second, so 9.6 divided by 0.005 is about 1,900 queries per second per node. **Eliminates** a single-node deployment: 50,000 divided by 1,900 is 27 nodes.
4. Replication versus sharding: 27 nodes each holding the full 4 GB index is cheaper in engineering time than sharding 4 GB across 27 machines, and it removes cross-shard fan-out from the latency budget entirely. **Eliminates** geographic sharding of the read fleet at this data size.
5. Update rate: 500,000 per day is 5.8 per second. **Eliminates** an incremental-update-first design, provided a full rebuild is fast enough to meet the one hour freshness target. That proviso is the whole of the next deep dive.
6. Density skew: ASSUME the densest 0.1 percent of cells contain 10 percent of places, because commercial activity concentrates in city centres; measure your own distribution before trusting this. That means a dense cell holds about 200,000,000 times 0.10 divided by (number of cells times 0.001) places, which for a one million cell grid is 20,000 places in one cell. **Eliminates** any fixed per-cell allocation, and eliminates scanning a whole cell without a cap.

### V0: the smallest design that meets the requirements (proximity)

```
+---------+    +------------------+    +------------------+
| Client  |--->| Search API       |--->| Relational DB    |
| lat lon |    | bounding box     |    | with a spatial   |
| radius  |    | then filter      |    | index on point   |
+---------+    +------------------+    +------------------+
```

Caption: V0 for proximity search. One stateless API converts the radius into a bounding box, issues a single spatial index query against the primary database, filters and sorts the results in the database, and returns a page. Three boxes, no cache, no separate index service.

This satisfies every functional requirement including filters and pagination, because the database already does both. It is correct up to one computable limit: ASSUME a bounding-box query with filters over 200,000,000 rows costs 20 ms of database CPU, which is a reasonable planning figure for an index descent plus a few hundred row fetches on warm pages. With 8 usable cores, 8 divided by 0.020 is about 400 queries per second.

Below roughly 400 queries per second, building a separate index service is a second copy of your data to keep correct, for no measured benefit.

### Breaking points, and what each version adds (proximity)

**Break one, at about 400 queries per second.** Database CPU saturates, p99 crosses 100 ms, and because the same node serves writes, update latency degrades at the same time. You observe it as a latency-versus-throughput curve that bends sharply, and as lock wait time rising alongside it.

**V1: move the geometry into an immutable read-side artefact.**

```
+---------+    +----------------+    +-------------------+
| Client  |--->| Router: hash   |--->| Query node: memory|
+---------+    | to any replica |    | mapped geo index  |
               +----------------+    | plus doc store    |
                                     +-------------------+
+-----------------+    +-------------------------------+
| Relational DB   |--->| Builder job: sort places into |
| source of truth |    | cell order, emit one artefact |
+-----------------+    +-------------------------------+
```

Caption: V1 for proximity search. The relational database stays the source of truth for writes. A builder job periodically reads all places, sorts them into spatial cell order and emits a single immutable file. Query nodes memory-map that file, so a query is a binary search plus a contiguous scan, and adding capacity is copying a file to another machine.

The property that makes this cheap: the artefact is immutable, so replicas need no coordination, no replication protocol and no consistency argument beyond "which version am I serving".

**Break two, the dense cell.** A five kilometre radius downtown covers cells holding, from the sizing, on the order of 20,000 places. Scanning and filtering 20,000 candidates at 10 microseconds each is 200 ms, twice the budget, while the same query in a suburb takes 3 ms. You observe it by slicing the latency histogram by result-set size or by city, never by looking at the global p99, which averages the two populations into a number that describes nobody.

**V2: adapt the cell size to density, and cap the work per cell.** Split any cell above a threshold count into four children at build time, so cells hold a bounded number of places regardless of geography. Store each cell's places pre-sorted by a static quality score, so a query that only needs the top 20 can stop early instead of scanning the cell.

**Break three, freshness for a small class of change.** A restaurant closing permanently must disappear faster than the rebuild cycle. You observe it as complaints, which is the worst kind of observation, so instrument it: seed canary records and measure time from write to visible.

**V3: a delta overlay.** Query nodes consult a small in-memory layer of changes since the last build, merged into results at query time and folded into the next artefact. Sized from the update rate: 5.8 per second times 3,600 is about 21,000 changes per hour, which at 1 KB each is 21 MB, small enough to hold and ship continuously.

### Boxes not to add without a measurement (proximity)

| Component | The measurement that justifies it | Below this, it is over-engineering |
|---|---|---|
| Separate index service | Database CPU above 60 percent at peak, or write latency degraded by read load | Under a few hundred queries per second, the spatial index in your primary database is correct and free |
| Memory-mapped immutable artefact | Rebuild time comfortably inside the freshness budget, and a read fleet that needs to scale independently | If updates must be visible in seconds, an immutable artefact fights you, and a mutable index is the right shape |
| Adaptive cell splitting | Latency p99 in the densest slice more than 3x the median slice | Uniform data does not overflow cells, and adaptive splitting adds build complexity and a second lookup path |
| Delta overlay | Measured canary visibility lag exceeds the freshness requirement | If the rebuild already fits the budget, an overlay is a second code path for results and a second source of bugs |
| Result cache | Repeat rate of identical query keys above roughly 20 percent | Continuous coordinates make cache keys nearly unique; caching them fills memory and hits nothing |

### Deep dive one: choosing the structure by measuring rebuild cost

The two case studies before this one dived on throughput and coordination. This one dives on an index build, because here the data is nearly static and the read fleet is stateless, which makes build cost, not query cost, the deciding variable. Say that reasoning out loud in the interview; it is the part that shows judgement.

**The candidates:**

| Structure | Build shape | Query shape | Update shape |
|---|---|---|---|
| Geohash or a similar linear cell key | Compute a 64 bit key per point, then sort | Compute the cell cover of the circle, then a range scan per cell key prefix | Insert into a sorted structure, or rebuild |
| Quadtree | Insert points one at a time, splitting nodes above a capacity | Descend to the covering nodes, then collect | Cheap in place, the structure's main selling point |
| Hierarchical spherical cells | Same as the linear key, with better cell shapes near the poles and a better cell cover algorithm | Same, with fewer cells needed to cover a circle | Same as the linear key |

**The benchmark to run, and the numbers to beat.** These constants are assumptions; the point of the dive is that you go and measure them on your data rather than argue from taste.

- Linear cell key: ASSUME 200 ns to compute a key and 100 ns per element amortised for a parallel sort, because both are cache-friendly sequential passes. For 200,000,000 points that is 200,000,000 times 300 ns, about 60 seconds, plus the time to read 4 GB.
- Quadtree: ASSUME 1 microsecond per insert, because at a depth around 14 each insert is a chain of pointer dereferences that miss cache. For 200,000,000 points that is 200 seconds, and the output is a pointer graph that cannot be memory-mapped or copied as bytes.

**The decision that follows.** A one minute rebuild against a one hour freshness requirement means a full rebuild every 15 minutes costs under 7 percent of one machine's time. The quadtree's advantage is cheap in-place insertion, and at 5.8 updates per second we do not need cheap in-place insertion. So the linear cell key wins, not because it is a better structure in the abstract, but because this workload never uses the quadtree's advantage.

**Reverse the workload and the answer reverses.** At 500,000 updates per *second* instead of per day, rebuilding is impossible, and a mutable in-place structure is the only option. State that inversion; it proves the choice was reasoned rather than recited.

**The boundary problem, which every cell scheme has.** Two points either side of a cell edge are neighbours in reality and far apart in key space. The fix is not to pick a cleverer curve, it is to query the cell cover of the circle rather than a single cell, which means computing every cell the circle intersects and scanning each. The cost of a scheme is therefore how many cells it takes to cover a circle at a given precision.

??? note "Check yourself: your cell cover for a 5 km circle returns 40 cells at the chosen level, and 4 cells one level up. Which do you use?"

    It depends on which is cheaper: 40 cells at the fine level means 40 range scans over tightly bounded sets, while 4 cells at the coarse level means 4 scans over sets roughly 16 times larger, plus discarding many points outside the circle. Measure both. In dense areas the fine cover usually wins because the wasted candidates dominate; in sparse areas the coarse cover wins because per-scan overhead dominates.

### Deep dive two: pagination that stays stable across an index swap

The second dive is about the read contract, not the index, because that is where this design's immutability bites. A client paging through results holds a cursor while the fleet swaps from artefact version 99 to version 100.

**What goes wrong:**

| Symptom | Cause | Who notices |
|---|---|---|
| A place appears on page 1 and again on page 2 | Ranking changed between requests, pushing an item down past the page boundary | Users, immediately |
| A place is never shown | Ranking change pushed an item up past the boundary the client already passed | Nobody, which is worse |
| Page 3 is empty | The client's routing landed on a replica running an older artefact with fewer results | Users, and it looks like a bug in the client |

**The fix, in three parts:**

1. Make the cursor carry the artefact version, not just an offset. A request with version 99 is routed to a replica still serving 99, or is told to restart from page 1 with a clear signal.
2. Retire an artefact version only after a grace period longer than a realistic paging session, ASSUME five minutes, so version-pinned cursors almost never fail.
3. Make the sort key total. Ties broken by place identifier, so two replicas building the same artefact produce the same order and a rebuild does not reshuffle equal-scoring results.

**The trade you are making.** Version-pinned cursors mean a paging user sees a consistent but slightly stale view, and a place that closed two minutes ago stays on their page 4. That is the correct trade for a browse experience and the wrong one for live inventory, which is why the clarifying table asks about inventory up front.

??? note "Check yourself: why not solve this by making page size large enough that nobody pages?"

    Because it moves the cost rather than removing it. A 200 result page in a dense area is exactly the 200 ms scan from break two, paid by every user including the 80 percent who never scroll. Deep pagination is rare, so pay for it only when it happens.

### Failure modes and the operating contract (proximity)

| Failure class | How it shows up here | Signal to alert on |
|---|---|---|
| Cache stampede | The whole fleet swaps artefact version at once, every page fault is cold, and p99 spikes for a minute | Page fault rate at swap, and swap-time latency spike duration |
| Thundering herd | 27 nodes pull the same 4 GB artefact from the same store simultaneously | Artefact fetch concurrency, alert above a small multiple |
| Hot key | A cell covering a stadium or a transport hub is in the cover of a large share of queries | Per-cell scan count divided by median |
| Version skew | Half the fleet on version 99, half on 100, and a paging client sees duplicates | Distinct artefact versions live in the fleet, alert above 2 |
| Silent staleness | The builder fails, the last good artefact keeps serving, and nothing errors | Index age in minutes, alert above twice the build interval |

**Indicators to publish:** query p99 sliced by result-set size, index age, canary visibility lag, swap duration, distinct live versions, empty-result rate by region.

**Re-check against the requirements.**

- The 100 ms p99 holds only after adaptive cell splitting, because break two showed the dense slice at 200 ms; the global p99 was never the right measurement.
- The one hour freshness target is met by the rebuild alone at 15 minute cadence, with the delta overlay as insurance for the class of change where an hour is too slow.
- The 50,000 query per second target is met by 27 replicas from the sizing, plus headroom, and the design deliberately did not shard, which is the sentence to say when the interviewer asks how you would shard.

### Where this answer is a simplification (proximity)

- The cell schemes are real, documented libraries. Geohash was published by Gustavo Niemeyer in 2008 and remains the simplest scheme to explain on a whiteboard. Google's [S2 geometry library](http://s2geometry.io/) documents a spherical cell hierarchy with better cell shapes and a cell cover algorithm; Uber's [H3](https://www.uber.com/en-US/blog/h3/) (June 2018) uses hexagons.
- Production search engines usually do not use any of the three. Apache Lucene indexes points with a block k-d tree, derived from the Bkd-tree ([Bkd-Tree: A Dynamic Scalable kd-Tree](https://users.cs.duke.edu/~pankaj/publications/papers/bkd-sstd.pdf), Procopiuc, Agarwal, Arge and Vitter, SSTD 2003). If your interviewer expects you to reach for a search engine, this is the structure underneath it.
- The spatial index in a relational database is a real answer with a real ceiling. The [PostGIS documentation](https://postgis.net/docs/using_postgis_dbmanagement.html) describes its generalised search tree indexing, and V0 above is not a straw man.
- Filters change everything. A high-selectivity filter such as "open now, vegan, rated above 4.5" is often better answered by filtering first and checking geography second, which inverts the whole design. Real systems keep both paths and choose per query from estimated selectivity.

### Level delta: the same prompt at three levels (proximity)

| Dimension | Mid | Senior | Staff |
|---|---|---|---|
| Framing | Proposes a geohash index immediately | Asks whether entities move and whether filters exist, because both change the structure | Asks what the product does with the results, since browse and dispatch have opposite freshness contracts |
| Numbers | Estimates total storage | Derives the 4 GB geo index and uses it to refuse sharding, then derives 27 replicas from query cost | Adds the rebuild benchmark and shows the structure choice follows from the update rate, then states the update rate at which it flips |
| Progression | Draws the index service first | Starts with the database spatial index, computes 400 queries per second, then extracts the read path | Also names what not to build: a result cache, and the sharding the interviewer expects |
| Depth | "Geohash is better than quadtree because it is simpler" | Compares them on build, query and update cost, and names the boundary problem and the cell cover fix | Specifies the benchmark to run, names the constants as assumptions, and states the workload inversion that reverses the answer |
| Read contract | Not mentioned | Notes that ranking must be stable | Specifies version-pinned cursors, the grace period, the total sort order, and the staleness this trades for |
| Operations | "Add monitoring" | Slices latency by density and alerts on index age | Alerts on distinct live versions and canary visibility lag, and treats a silently stale index as the most dangerous outcome because it does not error |

The senior delta is the two refusals: no sharding, and no cache. The staff delta is the falsifiable benchmark and the read contract nobody else brought up.

---

## Case study 8: Typeahead suggestions

### The prompt (typeahead)

"Design search autocomplete: as the user types, show the top suggestions."

### Clarifiers that fork the design (typeahead)

| Question worth asking | If the answer is A, the design becomes | If the answer is B, it becomes |
|---|---|---|
| Global popularity, or personalised? | A, global: one shared index, cacheable at every layer | B, personalised: a per-user blend at query time, and the edge cache hit ratio collapses to near zero |
| Prefix matching only, or typo tolerant? | A, prefix: an ordered structure and a simple lookup | B, typo tolerant: edit-distance search, a different index, and roughly an order of magnitude more work per query |
| How fresh must a newly trending term be? | A, 15 minutes: a streaming counter and a delta layer | B, next day: a nightly batch build and nothing else |
| Is the suggestion set a closed catalogue or open user queries? | A, closed: a bounded corpus, everything precomputable | B, open: an unbounded tail, so thresholding and abuse filtering become part of the build |
| Must unsafe suggestions be suppressed? | A, yes: a moderation gate becomes a build-blocking step with a human in it | B, no: the build is fully automatic, which is faster and riskier |
| One locale or many? | A, one: a single index | B, many: an index per locale, multiplying memory and build time by the locale count |
| Is the latency budget end to end, including the network? | A, yes, under 100 ms end to end: client debounce and client-side caching become part of the architecture | B, server-side only: the client is out of scope and the request rate is whatever it is |

### Requirements (typeahead)

Functional: given a prefix, return the top 10 suggestions by score; ingest query logs; refresh ranking; suppress blocked terms; support several locales.

| Requirement | Value | Tag |
|---|---|---|
| Searches per day | 5,000,000,000 | GIVEN |
| Server-side typeahead budget | 100,000 queries per second at peak | GIVEN |
| Server latency | p99 under 25 ms | GIVEN |
| Trending term visible within | 15 minutes | GIVEN |
| Indexed suggestions across all locales | 100,000,000 | ASSUMED: a thresholded head of the query distribution, sized below |
| Average suggestion length | 25 characters | ASSUMED, so the string table can be sized |
| Requests per search from a naive client | 10 | ASSUMED: one request every two typed characters on a 20 character query |

### Numbers that eliminate options (typeahead)

1. Naive request rate: 5,000,000,000 searches times 10 requests is 50,000,000,000 requests per day, or 578,000 per second averaged, and roughly 1,200,000 at an assumed 2x peak. The budget is 100,000. **Eliminates** a client that calls the server on typing. The client is now part of the architecture, not a consumer of it.
2. Client policy: a 200 ms debounce plus caching of prefixes already answered brings this to ASSUME 3 requests per search, because most keystrokes fall inside a debounce window and extensions of an answered prefix can be filtered locally. That is 5,000,000,000 times 3 divided by 86,400, about 174,000 per second averaged.

   **Eliminates** the claim that we are done: even with debounce we are above the 100,000 budget at average, so the client must also stop requesting past a prefix length where the suggestion set has converged.
3. Corpus: ASSUME the top 10,000,000 distinct queries cover 90 percent of volume, because search query distributions are heavily skewed with a very long tail; measure the coverage curve on your own logs. Extending to 100,000,000 covers the useful tail across locales. **Eliminates** indexing every distinct query string ever seen, which is unbounded and mostly typos.
4. Index memory: ASSUME 8 trie nodes per suggestion after prefix sharing, and 70 bytes per node made up of 40 bytes for ten 4 byte identifiers, roughly 24 bytes of amortised child references and a 4 byte score. That is 100,000,000 times 8 times 70 bytes, about 56 GB. **Eliminates** a single-machine index, and therefore forces partitioning.
5. Query CPU: ASSUME 20 microseconds per lookup, a walk of a handful of nodes plus copying ten identifiers. A 16 core node at 60 percent gives 9.6 core-seconds per second, so 9.6 divided by 0.00002 is about 480,000 queries per second per node. **Eliminates** partitioning for throughput. One node could serve the entire query budget if the index fitted. Partitioning here is purely a memory decision, and saying that out loud separates you from candidates who shard reflexively.
6. Counting pipeline: 5,000,000,000 divided by 86,400 is about 58,000 events per second into the aggregation. **Eliminates** exact per-key counters in a transactional store, and points at a streaming aggregate or a sketch.
7. Rebuild transfer: a 56 GB artefact rebuilt every 15 minutes is 56 GB per 900 seconds per replica, about 62 MB per second continuously per replica. **Eliminates** meeting the 15 minute freshness target by rebuilding. The base index rebuilds nightly, and freshness comes from somewhere else.
8. Delta size: ASSUME 100,000 trending terms per 15 minute window at 100 bytes each is 10 MB, which distributed to 30 replicas is 300 MB per window, about 0.3 MB per second. **Eliminates** any concern about delta distribution cost, which is why the delta layer is the right answer to point 7.

### V0: the smallest design that meets the requirements (typeahead)

```
+--------+    +------------------+    +-------------------+
| Client |--->| Suggest API      |--->| Relational DB     |
| prefix |    | prefix range     |    | terms table with  |
+--------+    | scan, order by   |    | an index on term  |
              | count, limit 10  |    +---------+---------+
              +------------------+              |
                                                |
              +---------------------------------+------+
              | Nightly job: aggregate query logs into |
              | the terms table                        |
              +----------------------------------------+
```

Caption: V0 for typeahead. One API issues a prefix range scan against an ordered index on the term column, sorts the matches by count and returns ten. A nightly job recomputes counts from the query logs. Three boxes, no trie, no cache, no streaming.

It is correct up to a limit set by the ordering problem: the index is ordered by term, but the answer is ordered by count, so every matching row must be read before the top 10 is known. ASSUME 5 ms per query on a corpus of 1,000,000 terms, since a typical prefix matches a few thousand rows on warm pages. With 8 usable cores, 8 divided by 0.005 is about 1,600 queries per second.

Below a million terms and a couple of thousand queries per second, a trie is a data structure you are maintaining for pleasure.

### Breaking points, and what each version adds (typeahead)

**Break one, the short prefix.** At corpus growth, single-character prefixes match a large share of the table. ASSUME the letter "a" starts 4 percent of terms: at 100,000,000 terms that is 4,000,000 rows to read and sort for one keystroke. You observe it by slicing latency by prefix length; the curve is monotone and the one and two character buckets are the entire tail.

**V1: precompute the top ten at every prefix.**

```
+--------+   +--------------+   +----------------------+
| Client |-->| Suggest API  |-->| In-memory trie: each |
| with   |   | routes by    |   | node holds the top   |
| cache  |   | prefix       |   | ten for that prefix  |
+--------+   +--------------+   +----------+-----------+
                                           |
             +-----------------------------+----------+
             | Nightly builder: counts to trie file   |
             +----------------------------------------+
```

Caption: V1 for typeahead. Each trie node stores the answer for its prefix, so a lookup is a walk of at most the prefix length and a copy of ten identifiers, independent of how many terms share the prefix. The database becomes an input to the build rather than a participant in the query.

The client now also carries a debounce and a local prefix cache, per the sizing. That is not an optimisation; it is the only way the request rate fits the budget.

**Break two, memory.** From the sizing, 56 GB of precomputed nodes does not fit one machine with headroom, and adding a locale adds its share again. You observe it as resident set size approaching the machine limit, and as build output size growing per release.

**V2: partition by prefix bucket, weighted by traffic.**

```
+--------+   +----------------+   +------------+ +-----------+
| Client |-->| Router: first  |-->| Partition  | | Partition |
+--------+   | two characters |   | a-b, hot,  | | c-f, cold |
             | to partition   |   | 6 replicas | | 2 replicas|
             +-------+--------+   +------------+ +-----------+
                     |
             +-------+------------------------------------+
             | Assignment map, rebuilt when traffic skew  |
             | crosses a threshold                        |
             +--------------------------------------------+
```

Caption: V2 for typeahead. A router maps a prefix to the partition that owns it. Partitions are not equal-sized: assignment is weighted by measured traffic, so a hot two-character bucket gets its own partition and more replicas, while cold buckets share. Because a prefix maps to exactly one partition, there is no fan-out and no scatter-gather latency.

**Break three, freshness.** A nightly build cannot show a term that started trending at 09:00 until tomorrow, and the requirement is 15 minutes. You observe it with canary terms: inject a synthetic term into the log stream and measure time to first appearance.

**V3: a streaming counter plus a delta layer, with a moderation gate.** Counts flow from a streaming aggregate into a small delta index shipped every few minutes. Query nodes merge the base trie's top ten with the delta's candidates for the same prefix and re-sort. The moderation gate sits between the counter and the delta, because the delta is exactly the path a manipulation campaign would use.

### Boxes not to add without a measurement (typeahead)

| Component | The measurement that justifies it | Below this, it is over-engineering |
|---|---|---|
| Precomputed trie | Latency in the shortest-prefix slice above the budget, at the current corpus size | Under about a million terms, an ordered index and a limit clause meets a 25 ms budget |
| Prefix partitioning | Index memory above what one node holds with headroom for the build and the delta | If the index fits, partitioning adds a router, an assignment map and a rebalancing procedure for nothing |
| Traffic-weighted assignment | Requests per second on the busiest partition above 3x the fleet median | Uniform traffic does not need weighting, and the map becomes a thing to keep correct |
| Streaming counter and delta layer | Measured canary lag above the freshness requirement | If nightly meets the product requirement, the streaming path is a second ranking system with its own drift |
| Edge cache for suggestions | Repeat rate of identical prefix keys above roughly 40 percent, which needs non-personalised results | Personalised suggestions make keys nearly unique, so the cache costs memory and returns misses |

### Deep dive one: partitioning a trie, and the memory versus latency dial

Chosen because typeahead has an unusual property the sizing exposed: one node has enough CPU for the entire query load and not enough memory for the index. Every partitioning decision here is therefore about bytes, and the interesting questions are what to store and what to recompute.

**What to partition on.**

| Key | Fan-out per query | Balance | Verdict |
|---|---|---|---|
| Hash of the full term | Every partition, because a prefix does not determine a hash | Perfect | Wrong: a prefix query becomes a scatter-gather across the fleet |
| First character | One partition | Poor: the busiest letters carry many times the median | Workable only with weighting |
| First two characters, assigned by measured traffic | One partition | Good, and adjustable without a code change | The default worth defending |
| Locale, then first two characters | One partition | Good, and it isolates a locale's build | Right when locales have separate corpora and separate moderation rules |

**The dial: how deep to precompute.** Storing the top ten at every node is what makes lookups constant-ish. It is also what costs 56 GB.

| Precompute depth | Memory effect | Latency effect |
|---|---|---|
| Top ten at every node | Full 56 GB | Lookup is a walk plus a copy, roughly 20 microseconds |
| Top ten only to depth 4, traverse below | ASSUME depth 4 nodes are about 15 percent of all nodes, so roughly 8 GB | Deeper prefixes traverse a subtree, but subtrees below depth 4 are small because the corpus narrows fast |
| No precompute, traverse always | Smallest, roughly the string table plus structure | Short prefixes traverse enormous subtrees, which is exactly break one returning |

The shape of the trade is the lesson: precomputation is most valuable where subtrees are largest, which is at shallow depths, and least valuable where they are small, which is deep. So precompute shallow and traverse deep, and you keep most of the latency benefit for a fraction of the memory.

??? note "Check yourself: why is a hot partition here less dangerous than a hot shard in the ride hailing case?"

    Because this partition is read-only. A hot read partition is fixed by adding replicas, which is copying a file. A hot write shard in the ride hailing case owns mutable state and a lease protocol, so you cannot fix it by adding copies; you have to move ownership, and moving ownership is where the split brain risk lives.

### Deep dive two: keeping the ranking fresh without rebuilding

The second dive is about the update path, not the serving structure, because the sizing showed the base rebuild cannot meet the freshness target and the delta can. The two dives are deliberately on opposite sides of the same system.

**Counting at 58,000 events per second.** Exact counts for 100,000,000 keys in a transactional store is the option the sizing removed. Two workable approaches:

| Approach | Memory | Error | When to pick it |
|---|---|---|---|
| Partitioned streaming aggregate, count per key per window | Proportional to distinct keys in the window | Exact within the window | When the window's distinct key count is manageable and you want auditable numbers |
| Count-min sketch plus a heavy-hitter structure | Fixed and small, independent of key count | Overestimates by a bounded amount with a stated probability | When you only need the head, which is exactly what a delta layer needs |

The count-min sketch is described in [An Improved Data Stream Summary: The Count-Min Sketch and its Applications](https://dl.acm.org/doi/10.1016/j.jalgor.2003.12.001) (Cormode and Muthukrishnan, Journal of Algorithms, 2005). Saying the name is not the point; saying that it overestimates and never underestimates, so it is safe for finding trending terms and unsafe for billing, is the point.

**Merging base and delta at query time.** The node holds the base top ten for the prefix and a delta map of terms whose score moved. The merge is: take the base ten, add any delta terms matching the prefix, re-score everything with the delta's counts where present, sort, return ten. The delta is small enough from the sizing that this stays inside the 25 ms budget.

**The swap, and why it is staggered.** Replacing a 56 GB memory-mapped artefact means the new file's pages are cold. If the fleet swaps together, every node takes its page-fault penalty at the same moment and the fleet-wide p99 spikes. Stagger the swap across replicas, and pre-warm by replaying a sample of recent prefixes against the new artefact before it takes traffic.

**The moderation gate.** Counts alone are manipulable: a small number of automated clients can push a term into the head. Two mitigations, both cheap:

1. Count distinct users per term, not raw occurrences, so volume from one source does not move the ranking.
2. Require a term to survive a blocklist and a policy check before it can enter the delta. Google publishes a policy describing categories of predictions it removes ([How Google autocomplete works in Search](https://blog.google/products/search/how-google-autocomplete-works-search/), Google, April 2018, and the accompanying [autocomplete policies](https://support.google.com/websearch/answer/7368877)). The design consequence is that the delta path has a gate in it, so its freshness is bounded by how fast the gate runs.

??? note "Check yourself: a term appears in the delta, then disappears at the next nightly build. What happened?"

    The nightly build recomputed the term's score over a long window, where a 15 minute spike is negligible. The delta and the base are measuring different things: recent velocity versus sustained popularity. Either accept that trending terms fade, which is usually correct, or make the base build use a decayed count so a real shift in popularity carries through.

### Failure modes and the operating contract (typeahead)

| Failure class | How it shows up here | Signal to alert on |
|---|---|---|
| Cache stampede | The fleet swaps artefacts together and every node page-faults at once | Latency spike duration at swap, and page fault rate |
| Thundering herd | A client release removes the debounce and the request rate multiplies overnight | Requests per search, which is the ratio that matters, not raw queries per second |
| Retry storm | Clients retry timed-out suggestion calls, adding load during the exact period the service is slow | Retry rate; the correct policy for typeahead is to never retry and fail silently |
| Hot key or hot partition | A single trending prefix concentrates traffic on one partition | Partition queries per second divided by fleet median |
| Poisoned ranking | Coordinated traffic pushes an unwanted term into the head | Delta entries rejected by the gate, and any delta entry with an anomalous unique-user to occurrence ratio |
| Silent staleness | The streaming counter dies, the base index keeps serving, and nothing errors | Delta age and canary visibility lag |

**Indicators to publish:** p99 sliced by prefix length, requests per search, suggestion acceptance rate, canary visibility lag, delta age, partition skew ratio, swap duration.

**Re-check against the requirements.**

- The 25 ms p99 holds because the lookup is a bounded walk plus a small merge, and because there is no scatter-gather; if partitioning had used a hash, this requirement would fail.
- The 100,000 query per second budget is met only with the client debounce and prefix cache from the sizing, which means the client team owns part of this service level and should be told so.
- The 15 minute freshness target is met by the delta path, and its real bound is the moderation gate, so gate latency is a published indicator rather than an internal detail.

### Where this answer is a simplification (typeahead)

- A pointer trie is rarely the production structure. Lucene stores term dictionaries in finite state transducers, which compress shared suffixes as well as shared prefixes; Michael McCandless described the implementation in [Using Finite State Transducers in Lucene](https://blog.mikemccandless.com/2010/12/using-finite-state-transducers-in.html) (2010). That change alone can move the 56 GB estimate by a large factor, which is why the estimate is labelled as an assumption.
- Typo tolerance is a separate system. Prefix matching cannot answer a query with a transposition in the first two characters, and bolting edit distance onto a trie changes both the memory and the latency arithmetic.
- Personalisation destroys the caching story. Every layer above the index, including the client cache and any edge cache, assumes the answer depends only on the prefix. Adding personal history means merging a per-user candidate set at query time and accepting a much lower hit ratio.
- Suggestion quality is a ranking problem, not a counting problem. Raw popularity ignores whether the suggestion led to a good outcome. Real systems rank on downstream engagement, which turns this into a machine learning system with training data, evaluation and a feedback loop, and the feedback loop is self-reinforcing because suggested terms get typed more.

### Level delta: the same prompt at three levels (typeahead)

| Dimension | Mid | Senior | Staff |
|---|---|---|---|
| Framing | Proposes a trie in the first minute | Asks about personalisation and typo tolerance first, because both invalidate the trie plan | Asks what the suggestion is for: query completion, product discovery or navigation, since each changes the ranking objective and therefore the pipeline |
| Numbers | Estimates corpus size | Derives 578,000 requests per second from a naive client and uses it to move work to the client | Also derives that one node has enough CPU and not enough memory, and makes that asymmetry the organising principle of the design |
| Progression | Draws trie, partitions and streaming pipeline at once | Starts with an ordered index, shows the short-prefix failure, then adds precomputation | Also names what to remove: the edge cache once personalisation is on the roadmap, and the streaming path if the product does not need 15 minute freshness |
| Depth | "Shard the trie" | Compares partition keys, rejects hashing because it forces scatter-gather, and weights assignment by traffic | Adds the precompute-depth dial with the memory and latency numbers, and explains why shallow precomputation captures most of the benefit |
| Ranking | "Rank by frequency" | Separates base popularity from recent velocity and merges at query time | Names the manipulation risk, specifies unique-user counting and a gate, and states that the gate bounds the freshness service level |
| Ownership | Not mentioned | Notes the client must debounce | States that the client owns part of the service level, publishes requests-per-search as the shared indicator, and makes a client release that changes it a reviewable change |

The senior delta is rejecting hash partitioning for a stated reason and separating popularity from velocity. The staff delta is naming the CPU-versus-memory asymmetry as the design's organising fact, and turning a client behaviour into a contractual indicator.

## 10. Practice problems

Two sets, and they are not interchangeable. The blocked set rehearses the moves you just watched. The interleaved set makes you pick the move, which is the thing an interviewer actually scores.

### 10a. Blocked set: same shape as the worked examples

Work each one on paper before opening the answer. Five to fifteen minutes each.

**B1. Code length for a link shortener.**
A shortener is told to expect 200 million new links per day and to keep every link forever. Codes are drawn at random from a 62 character alphabet. Is a 6 character code enough? What is the shortest length that survives ten years, and what is the first collision-related problem you will hit before you run out of space?

??? note "Show answer"

    Space at length 6: 62^6 = about 5.7e10 codes. At 2e8 per day that is 5.7e10 / 2e8 = 285 days. Six characters is eliminated by a single division.

    Length 7: 62^7 = about 3.5e12. 3.5e12 / 2e8 = 17,500 days = about 48 years. Length 7 survives ten years with room. Length 8 buys 62 times more and costs one byte per record, so it is defensible but not required by this requirement.

    The problem you hit first is not exhaustion, it is collision rate on insert. Ten years in you have 7.3e11 links against 3.5e12 slots, so roughly 1 in 4.8 random draws collides. Random-generate-and-check stops being one round trip and becomes several. That is the number that pushes you toward a pre-allocated counter or a key-generation service rather than reject sampling.

**B2. Rate limiter choice on memory.**
You must enforce 100 requests per minute per user for 10 million monthly active users, with at most 2 million distinct users active in any minute. Compare a sliding window log against a token bucket on memory. Which do you take, and what do you give up?

??? note "Show answer"

    Sliding window log stores one timestamp per request in the window. Worst case per user is 100 timestamps. At 8 bytes each that is 800 bytes of payload per user, plus per-entry overhead in whatever structure you use. For 2 million active users: 2e6 x 800 bytes = 1.6 GB of payload before overhead.

    Token bucket stores two numbers per user: current tokens and last refill time. Call it 16 bytes of payload. For 2 million active users: 2e6 x 16 = 32 MB.

    A 50x memory difference is what decides it. Take the token bucket. What you give up is exactness at the window edge: a token bucket permits a full burst of 100 immediately after a quiet minute, which a log would not. If the requirement is "never more than 100 in any rolling 60 seconds" you have to say out loud that the bucket does not meet it and offer the sliding window counter (two counters plus a weighted blend) as the middle option at about 32 bytes per user.

**B3. Availability math for a new dependency.**
Your service currently meets 99.95 percent monthly availability. A colleague proposes adding a synchronous call to an internal fraud service that publishes a 99.9 percent target. What happens to your number, and what are your two options?

??? note "Show answer"

    A 30 day month is 30 x 24 x 60 = 43,200 minutes. Your current budget: 0.05 percent of 43,200 = 21.6 minutes per month.

    Serial dependencies multiply. 0.9995 x 0.999 = 0.9985, which is 0.15 percent unavailable, or 64.8 minutes per month. You would triple your error budget by adding one box.

    Option one: make the call non-blocking. Fail open with a default decision, and reconcile asynchronously. Availability returns to 99.95 percent and you have accepted a fraud risk window that someone must price.

    Option two: keep it synchronous and renegotiate the service level objective for your service down to 99.85 percent. That is a product decision, not an engineering one.

    The answer that fails is "we will add retries". Retries against a hard-down dependency convert unavailability into latency and, if unbounded, into a retry storm.

**B4. Partitioning under a write hot spot.**
A table is partitioned by `tenant_id`. One tenant now produces 45 percent of all writes and its partition is saturated. You cannot ask the tenant to slow down. What do you change, and what does the change cost you?

??? note "Show answer"

    The shape of the fix is to make the partition key carry entropy that the hot tenant does not control: composite key of `tenant_id` plus a bucket, where the bucket is `hash(record_id) mod N` for that tenant only. Writes for the hot tenant now spread across N partitions.

    The cost is on the read path. Any query that used to be a single-partition scan by tenant becomes a scatter across N partitions plus a merge. You have converted a write problem into a fan-out read problem, so N should be the smallest number that clears the write ceiling, not a round number like 64.

    Say the second half out loud. Candidates who name only the split, and not the read cost, read as having memorised a trick rather than made a trade.

    The alternative worth naming and rejecting: move the hot tenant to its own dedicated cluster. That is correct when the tenant is large enough to justify separate operations, and it is over-engineering below roughly one tenant per operator on call.

**B5. Delivery semantics and idempotency key storage.**
A payment worker consumes from an at-least-once queue at 500 messages per second. You de-duplicate with an idempotency key. How long must you retain keys, and how much storage is that?

??? note "Show answer"

    Retention is set by the longest possible redelivery gap, not by a comfortable round number. Bound it with three inputs, and state each as an assumption: the queue's maximum visibility timeout or retry window (assume 6 hours), the maximum client retry window (assume 24 hours, because mobile clients retry after reconnect), and your worst tolerated replay after an incident (assume 72 hours). Retention is the maximum, so 72 hours.

    Storage: 500 per second x 86,400 = 4.32e7 keys per day, x 3 days = 1.3e8 keys. At 64 bytes per entry (key plus a small result pointer) that is 1.3e8 x 64 = about 8.3 GB. That fits in one memory-resident store with replication, which eliminates the "we need a distributed database for dedupe keys" branch.

    The number that changes the design is the 72 hours, not the 500 per second. If the replay requirement were 30 days you would be at 83 GB and would move keys to disk with a time-to-live index.

### 10b. Interleaved set

These are not labelled by topic. Decide first which tool applies. Four of the six send you back to a page you should already have read: the [trade-off atlas](index.md#the-trade-off-atlas) and the [failure library](index.md#the-failure-library) on the hub, the [system design question bank](/Interview-Questions/System-design/), and the [forward deployed engineer question bank](/Interview-Questions/Forward-Deployed-Engineer/). Open those pages while you work; the point of an interleaved set is that the tool is not on the screen you just read. Twenty to thirty minutes each, out loud, with a timer.

**I1.** Our internal analytics dashboard shows yesterday's revenue correctly, but during peak hours the "last hour" panel is 40 minutes stale. It is fed by a nightly batch job plus a streaming topper. Diagnose and propose a fix. Use the batch-versus-stream row of the [trade-off atlas on the hub](index.md#the-trade-off-atlas): name the quantity that row says decides it, then say who in this organisation owns that quantity.

**I2.** Here is what we run today: one region, one Postgres primary with two read replicas, an application tier behind a load balancer, and a nightly analytics export. Writes to the orders table have grown roughly sixfold in a year and write p99 crossed 400 milliseconds last month. Plan the next two quarters.

**I3.** We serve 300 requests per second from three application servers. The p99 is 40 milliseconds. The dominant query is a primary-key lookup in Postgres that the database reports at about 3 milliseconds. A team member has proposed putting Redis in front of it. Design that cache.

**I4.** Design the aggregation path for ad click counting, where finance needs hourly totals that do not change after they are published, and roughly 2 percent of click events arrive up to 6 hours late. Before you start, read the stream processing prompt in the [system design question bank](/Interview-Questions/System-design/), take its stated scale requirements as GIVEN, and say which single one of them changes your watermark and lateness decision, and which change nothing.

**I5.** A payments endpoint called from a mobile app is double charging approximately 1 request in 20,000. The endpoint already accepts an idempotency key. Find the bug and change the design. Then read the prompt on making a customer pipeline safe to re-run (incremental loads, change data capture and idempotency) in the [forward deployed engineer question bank](/Interview-Questions/Forward-Deployed-Engineer/), and say which of its idempotency arguments survive when the effect is an external charge rather than a row you own.

**I6.** Our feed service holds a 90 millisecond p50 all day. Every deploy, the p99 goes to 4 seconds for about eight minutes, then recovers on its own. Name the failure class from the [failure library on the hub](index.md#the-failure-library), give the design change, and say which of that library's prevention rows you would deliberately not add here and why.

!!! warning "Honesty note about the interleaved set"

    You will score lower here than on the blocked set, and it will feel worse. That gap is the point: the blocked set tests whether you can execute a move you have been handed, and the interleaved set tests whether you can choose one, which is the only skill the interview measures. A drop of one or two rubric levels between the two sets is the expected result, not evidence that the reading failed.

---

## 11. Self-assessment rubrics

### The master rubric

Every problem above is scored on the same four criteria. Levels are named, not numbered, because a visible number crowds out the comment.

| Criterion | Developing | Adequate | Strong | Exceptional |
|---|---|---|---|---|
| Problem framing and scoping | Starts designing before asking anything. Requirements are restated, not derived | Asks questions, but the answers would not change any box in the design | Every question asked has two branches, and the chosen branch is stated | Names the requirement that is missing from the prompt and proposes the number to use until someone corrects it |
| Design and trade-off reasoning | One design, presented as the answer. Alternatives absent | Alternatives named, chosen on preference or familiarity | Each choice is decided by a stated number or measurement, and the rejected option is named | Identifies the one decision that is expensive to reverse and sequences the design around it |
| Depth in at least one area | Stays at box level everywhere | Goes one layer deeper when prompted | Volunteers one deep dive and reaches implementation detail without being pushed | Deep dive includes how it fails, how it is observed, and what the recovery costs |
| Communication and drive | Interviewer has to ask what happens next | Answers well, does not advance the conversation | States the plan, keeps the clock, checks in without seeming unsure | Handles a hostile follow-up by updating the design and saying which earlier claim changed |

Score each problem on all four. Two Strongs and two Adequates is a passing senior answer. Four Adequates with no Strong is the profile that most often gets a "hire at a lower level" outcome, because nothing stood out.

### Rubric notes and answers, blocked set

??? note "B1 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks whether links expire, because expiry changes the space requirement more than length does | Accepts "forever" without noticing it was assumed |
    | Trade-offs | Uses the division to eliminate 6 characters explicitly | Picks 7 because it is a common answer |
    | Depth | Reaches the collision-rate-on-insert consequence | Stops at total capacity |
    | Communication | States the ten year horizon as an assumption before computing | Presents 48 years as a fact |

    Model answer: 62^6 = 5.7e10, divided by 2e8 per day = 285 days, so 6 is eliminated. 62^7 = 3.5e12, divided by 2e8 = 17,500 days = about 48 years, so 7 is sufficient. The binding problem before exhaustion is that random generation collides more often as the space fills, so at roughly 20 percent occupancy the generate-and-check loop needs multiple attempts and the write path gets slower over years. That pushes toward pre-allocated ranges.

    Wrong answer: "Use a UUID, they never collide." Reveals a misconception that the constraint is uniqueness. The constraint is length, because the artefact is a short link a human retypes. A 128 bit identifier encoded in base62 is 22 characters.

    Wrong answer: "Hash the URL with MD5 and take the first 7 characters." Reveals that truncation was not analysed. Truncating to 7 base62-equivalent characters gives you the same 3.5e12 space with worse distribution guarantees and no collision handling, and it also makes the same URL from two users share a code, which breaks per-user analytics.

    Wrong answer: "Add a character when we run out." Reveals no model of migration cost. Variable-length codes are fine, but the candidate has to say that existing codes keep working and that the decoder is length-agnostic, otherwise this is a breaking change to every published link.

    Compare your process: did you divide before you chose, or choose and then justify?

??? note "B2 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks whether the limit must be exact or approximate before comparing | Assumes exactness silently |
    | Trade-offs | Computes both memory figures and lets the 50x gap decide | Names both algorithms without sizing either |
    | Depth | Reaches the burst behaviour difference at the window edge | Treats the algorithms as equivalent |
    | Communication | States what is given up, unprompted | Presents the bucket as strictly better |

    Model answer: log is 2e6 x 100 x 8 bytes = 1.6 GB of payload; bucket is 2e6 x 16 bytes = 32 MB. Take the bucket, and say aloud that it permits a 100 request burst immediately after an idle minute. If the policy is a hard rolling window, offer the sliding window counter at roughly 32 bytes per user as the compromise, and say that it is approximate at the boundary by design.

    Wrong answer: "Use a distributed rate limiter with strong consistency across all edges." Reveals a misread of what rate limiting is for. Coordination cost per request is the dominant term, and the requirement did not ask for exactness across regions. Below roughly ten edge locations, per-node limits with a shared periodic reconciliation are cheaper and adequate.

    Wrong answer: "Store the counters in Postgres." Reveals no model of write amplification. Every request becomes a write. At even 10,000 requests per second that is 10,000 writes per second of pure overhead against a database sized for real work.

    Compare your process: did you reach for the algorithm you know best, or the one the memory number selected?

??? note "B3 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks what happens to a request if fraud is unreachable, before doing arithmetic | Treats it as a pure availability sum |
    | Trade-offs | Presents fail-open and renegotiated SLO as the two real options | Offers only "add retries" |
    | Depth | Converts percentages to minutes per month | Leaves it in percentages |
    | Communication | Names the fraud risk window as a product decision with an owner | Decides the risk unilaterally |

    Model answer: 0.9995 x 0.999 = 0.9985, so 21.6 minutes of monthly budget becomes 64.8 minutes. Either make the call asynchronous with a fail-open default and reconcile afterwards, which restores 99.95 percent and creates a bounded fraud exposure window, or accept 99.85 percent and go tell whoever owns the number. Retries do not help against a dependency that is down.

    Wrong answer: "Cache the fraud decision." Reveals a misunderstanding of the dependency. Fraud decisions are per transaction, so there is nothing to cache except a per-user reputation, which is a different and much weaker signal. Say that out loud rather than presenting the cache as equivalent.

    Wrong answer: "Add a circuit breaker, so availability is unaffected." Reveals a confusion between latency protection and correctness. A circuit breaker prevents cascading timeout, but you still have to say what the request does while the breaker is open. That decision, not the breaker, is the answer.

    Compare your process: did you convert to minutes per month, or argue about percentages?

??? note "B4 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks which reads must stay single-partition before choosing N | Chooses a split with no read model |
    | Trade-offs | Names the scatter-gather cost in the same breath as the split | Presents the split as free |
    | Depth | Chooses N from the write ceiling, and says how to migrate live | Says "shard it more" |
    | Communication | Names dedicated-cluster isolation as the alternative and rejects it with a threshold | Ignores alternatives |

    Model answer: composite key of tenant plus `hash(record_id) mod N`, applied only to the hot tenant so every other tenant keeps single-partition reads. Pick the smallest N that puts the hot tenant's write rate under the per-partition ceiling you measured. Migrate with dual writes, backfill, then flip reads. The cost is that this tenant's list queries become an N-way scatter and merge, which needs a bounded page size or it becomes the next incident.

    Wrong answer: "Switch the partition key to a hash of the record id for everyone." Reveals the misconception that a hot partition is a key-design problem for all keys. It fixes one tenant and degrades every other tenant's reads permanently.

    Wrong answer: "Put a cache in front of it." Reveals a read-path reflex applied to a write-path problem. Caching does not reduce write pressure on the saturated partition.

    Compare your process: did you say the read cost without being asked?

??? note "B5 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks who retries and over what window before choosing retention | Picks 24 hours because it is a day |
    | Trade-offs | Shows that 8 GB fits in memory and eliminates the distributed store | Sizes nothing |
    | Depth | Names what is stored beside the key (the result, so a replay returns the same response) | Stores only the key |
    | Communication | Labels all three windows as assumptions | States them as facts |

    Model answer: retention equals the largest of the queue redelivery window, the client retry window and the incident replay window. With 6, 24 and 72 hours assumed, take 72. 500 per second x 86,400 x 3 = 1.3e8 keys, x 64 bytes = about 8.3 GB, which fits one replicated in-memory store. Store the response alongside the key so a duplicate returns the original result rather than a bare "already processed".

    Wrong answer: "Use exactly-once delivery from the broker." Reveals belief in a guarantee the network cannot provide end to end. Brokers offer exactly-once processing within their own boundary; the moment you call an external payment gateway you are back to at-least-once plus idempotency.

    Wrong answer: "Keep keys forever, storage is cheap." Reveals no cost model. At 4.32e7 keys per day this grows by about 2.8 GB per day, so a year is a terabyte of pure dedupe state with no requirement asking for it.

    Compare your process: did retention come from a requirement, or from a habit?

### Rubric notes and answers, interleaved set

??? note "I1 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks what "stale" means to the user: missing events, or a late watermark | Starts proposing Kafka |
    | Trade-offs | Separates ingestion lag from aggregation window closure | Treats all lag as one number |
    | Depth | Reaches watermarks and allowed lateness, and what the panel should display while a window is open | Adds hardware |
    | Communication | Proposes a measurement before a change | Proposes a rewrite |

    Model answer: the first move is to split the 40 minutes into its parts, because they have different fixes. Measure event time to ingest time (producer and broker lag), ingest time to processing time (consumer lag), and processing time to window close (the watermark and allowed lateness setting).

    If the dominant term is consumer lag, you have a partition or parallelism problem. If it is window closure, no amount of capacity helps, because the pipeline is deliberately waiting for late events. The user-facing fix in that case is to render partial windows as partial, with a clear "still filling" state, rather than to shorten the wait and publish numbers that later change.

    The atlas row on batch versus stream says the deciding quantity is the freshness requirement in minutes, stated by whoever consumes the output. Here that owner is the person reading the panel, not the pipeline team, and until they name the number, "40 minutes stale" is not yet a defect. Getting that sentence said is often the whole fix, because the answer is sometimes that 40 minutes is fine and the label is what is wrong.

    Wrong answer: "Move everything to streaming and delete the batch job." Reveals the misconception that lag is caused by batch. Batch is producing the correct number; the panel that is wrong is the streaming one.

    Wrong answer: "Scale the consumers." Reveals that the candidate did not decompose the latency. If closure is the dominant term, this changes nothing and costs money.

    Compare your process: did you measure before you designed?

??? note "I2 rubric, model answer and wrong answers (brownfield)"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks what the write growth is made of before proposing anything | Proposes microservices in the first minute |
    | Trade-offs | Sequences changes by blast radius and reversibility | Presents a target architecture with no order |
    | Depth | Specifies the migration mechanics: dual write, backfill, shadow read, cutover, rollback trigger | Says "migrate to a new database" |
    | Communication | Gives a quarter-by-quarter plan with a stated exit criterion per step | Gives an architecture with no calendar |

    Model answer, in the order I would actually do it. Quarter one, buy headroom without changing shape: find what the 6x is made of (more orders, or more writes per order such as status updates and audit rows), move append-only side tables out of the hot path into an events table or a queue-fed consumer, add missing indexes, and reduce write amplification from triggers and hot updated columns. This is reversible and takes weeks. Measure again.

    Quarter two, only if p99 is still over budget: introduce a partitioned store for the orders write path behind a dual write, backfill history, run shadow reads to compare, then cut over reads per tenant with a documented rollback. Name what you are not doing: not splitting the monolith, not going multi-region, because neither addresses a write hot spot and both multiply the blast radius while you are already unhealthy.

    Wrong answer: "Split the monolith into services." Reveals a template answer. Service boundaries do not reduce writes to a table; the same rows get written from a different process.

    Wrong answer: "Add read replicas." Reveals a read-path reflex. There are already two, and replicas do not absorb writes.

    Wrong answer: "Move to a NoSQL store next sprint." Reveals no migration model. Without dual writes, backfill and a shadow read comparison, this is a rewrite with an unspecified rollback.

    Compare your process: did you order the work by reversibility, or by how interesting it was?

??? note "I3 rubric, model answer and wrong answers (do not add that)"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks what problem the cache is meant to solve, and finds there is not one | Starts choosing an eviction policy |
    | Trade-offs | Prices the new failure modes against a 3 millisecond saving | Prices only the latency win |
    | Depth | Names invalidation, stampede and split-brain staleness as the costs | Names only memory cost |
    | Communication | Declines the change without dismissing the person who proposed it | Either builds it silently or mocks the idea |

    Model answer: I would not add it, and here is the arithmetic. The query is 3 milliseconds inside a 40 millisecond p99, so the theoretical ceiling on the improvement is under 8 percent of tail latency, and only on cache hits. Against that, a cache adds: a second source of truth that can be stale, an invalidation path that must be written and tested, a stampede risk on cold start or eviction of a hot key, and one more thing that is down at 3 a.m.

    At 300 requests per second on a primary-key lookup, Postgres is already serving this from its own buffer cache. What I would do instead is confirm where the 40 milliseconds actually goes with a trace, because if the query is 3 milliseconds then 37 milliseconds is somewhere else and that is where the win is. I would revisit the cache when the database shows sustained CPU above roughly 60 percent attributable to this query, or when read volume grows about tenfold.

    Wrong answer: designing the cache as asked, competently. Reveals that the candidate optimises what they are pointed at. Interviewers use this prompt specifically to see whether you will push back with numbers.

    Wrong answer: "No, caching is premature optimisation." Reveals the right instinct with no evidence. A refusal without arithmetic scores the same as compliance without arithmetic.

    Compare your process: did you say the threshold at which you would change your mind?

??? note "I4 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Establishes that "does not change after publication" is the hard requirement | Optimises for freshness |
    | Trade-offs | Separates the fast approximate path from the slow correct path | Tries to make one path do both |
    | Depth | Handles the 6 hour lateness with an explicit restatement or correction mechanism | Drops late events |
    | Communication | States which number the dashboard shows and which number finance uses | Conflates the two consumers |

    Model answer: two consumers with two different requirements means two paths, and saying so early is most of the answer. The dashboard path is a streaming aggregation with a short watermark, labelled approximate, refreshed continuously. The finance path closes an hour only after the lateness bound has passed, so publication for hour H happens at H plus 6 hours plus a margin.

    On the borrowed numbers: from the question bank's stream processing prompt, the only stated requirement that moves this design is the freshness target, because it decides whether the dashboard path exists at all. The throughput and event-rate figures change partition and consumer counts and change nothing about correctness, so quoting them here would be arithmetic that eliminates nothing.

    Events carry a click identifier and are de-duplicated on it, because at-least-once transport plus retries is the normal case. The aggregate is then a distinct count over the identifier rather than an incrementing counter, which is what makes the result reproducible from the raw log. If a correction arrives after publication, it goes into an adjustment row for the next period rather than mutating a published hour, because finance systems reconcile on immutable periods.

    Wrong answer: "Use exactly-once processing so we do not need dedupe." Reveals the same guarantee confusion as B5. Exactly-once within one engine does not survive a producer retry across a network boundary.

    Wrong answer: "Wait 6 hours for everything." Reveals no read of the two consumers. The dashboard becomes useless and someone will build a shadow pipeline.

    Compare your process: did you notice there were two consumers before you drew a box?

??? note "I5 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks who generates the key and when, which is where the bug usually is | Assumes the key is correct and blames the database |
    | Trade-offs | Distinguishes a key-generation bug from a key-store race | Proposes locks immediately |
    | Depth | Reaches the check-then-act race and names the atomic fix | Adds a mutex with no scope |
    | Communication | Gives the 1 in 20,000 number a plausible mechanism | Hand-waves "a race condition" |

    Model answer: 1 in 20,000 is a rate low enough to be a narrow window, which points at check-then-act rather than at a missing key. Two candidate mechanisms, both testable.

    First, key generation: if the client regenerates the key on retry (for example the key is derived from a timestamp, or the app restarts and creates a new one), then the two requests are genuinely distinct to the server and no server-side fix helps. That is a client contract bug, fixed by generating the key once per user intent and persisting it across process death.

    Second, the store: if the server reads the key, sees nothing, and then writes, two concurrent retries both read nothing. The fix is a single atomic conditional insert that also records the in-progress state, so the second request blocks or returns the in-flight result. Both need the retention window from B5 to be at least the client retry window, or a late retry finds no record and re-charges.

    On the borrowed page: the re-runnable pipeline argument is that a replay is safe when every write is keyed and the sink applies a key once. That argument survives here only for the rows you own.

    It fails for the charge, because the provider's ledger is a sink you do not control, so safety has to come from the provider's own idempotency key rather than from your write being repeatable. That is the difference between an internal pipeline you can re-run and an external effect you can only ask to be applied once.

    Wrong answer: "Add a database transaction." Reveals a vague grasp of isolation. Unless the insert is a conditional write on a unique key, two transactions can still both find nothing under read-committed.

    Wrong answer: "Rate limit retries from the client." Reveals treating a correctness bug as a load problem. It lowers the incidence and leaves the double charge in place.

    Compare your process: did you look at the client before the server?

??? note "I6 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Ties the spike to the deploy event, not to traffic | Investigates traffic patterns |
    | Trade-offs | Distinguishes cold cache from connection storm from JIT or warmup effects | Names one cause and stops |
    | Depth | Names the self-recovery as the diagnostic clue and reasons from it | Ignores that it recovers alone |
    | Communication | Proposes one measurement that separates the candidate causes | Proposes three changes at once |

    Model answer: the class is a load-induced spike with a self-healing tail. Recovering without intervention rules out a permanent regression and points at warmup. Three candidates, each with a distinguishing signal:

    - Cold local cache: new instances start empty, so instance-level hit rate drops to near zero and climbs back as it fills.
    - Connection storm: new instances open their pools at once, so the database shows a connection spike and queueing.
    - Runtime warmup: CPU per request is elevated on new instances only, and falls without any cache filling.

    Measure per-instance cache hit rate, per-instance CPU and database connection count during the next deploy, then fix the one that shows up: pre-warm the cache before taking traffic, ramp traffic to new instances instead of switching at once, or stagger the rollout so only a fraction of capacity is cold at any moment. The general lesson is that a deploy is a load event, and capacity that is correct in steady state can be insufficient during the transition.

    In the hub's failure library this is cold-start collapse, and its prevention row already names rolling restarts and cache warming. The prevention row to reject out loud is load shedding: shedding during a deploy hides a self-inflicted event behind dropped user requests, and the trigger for adding it is a peak that exceeds provisioned capacity, which is not what is happening here.

    Wrong answer: "Increase instance count." Reveals no model of the mechanism. More cold instances makes a cold-start spike worse, not better.

    Wrong answer: "It fixes itself, so it is not a problem." Reveals a missing user model. Eight minutes of 4 second p99 on every deploy is an availability cost paid every release.

    Compare your process: did the fact that it self-recovers change your hypothesis, or did you ignore it?

---

## 12. Mastery check and remediation map

Seven short-answer items, one per learning objective from Section 4. Write your answer before opening the block. No multiple choice: producing the answer is the thing that builds the retrieval path.

**M1** (Objective 1, clarifying questions that fork the design). You are asked to design a notification system. Give two clarifying questions where each possible answer changes a box on the diagram, and say what changes.

??? note "Show answer"

    Question: is delivery best effort or must every notification eventually arrive? If best effort, a fire-and-forget publish with no per-user state is enough. If guaranteed, you need a durable per-user outbox, delivery receipts and a redelivery worker, which is three extra boxes and a storage sizing exercise.

    Question: can the same notification be sent to a user twice? If yes, at-least-once transport with no dedupe. If no, you need an idempotency key per notification per device and a retention window for it.

    A question that does not qualify: "how many users are there?" It is worth asking, but by itself it changes sizing, not shape.

**M2** (Objective 2, estimates that eliminate). A number is only allowed in your answer if it does one specific thing. What, and what do you say when it cannot?

??? note "Show answer"

    A number is allowed if the next sentence uses it to rule out a design option. If a number rules nothing out, it is arithmetic performed for its own sake and it costs you time you needed for a deep dive.

    When you cannot make it eliminate anything, say so and offer: "I can size this if it will drive a decision, otherwise I would rather spend the time on the write path." Interviewers who want the estimation will ask for it, and you have just shown you know why it exists.

**M3** (Objective 3, v0 and its breaking point). Define a breaking point in the form an interviewer wants to hear. Give an example in that form.

??? note "Show answer"

    The form is: what fails, at what number, and how you would see it. All three parts, or it is a guess.

    Example: "This single primary handles writes fine to roughly the point where sustained write throughput saturates its disk. I would see it as replication lag climbing on the replicas and write p99 rising while CPU stays flat, and I would put the alert on replication lag rather than on CPU."

    An answer that fails: "It will not scale." No component, no number, no observation.

**M4** (Objective 4, choosing replication or partitioning by the number that decides it). A service holds 4 TB and takes 1,000 writes per second at peak, with 50 reads for every write. Choose between one primary with read replicas and hash partitioning across shards. Name the number that decides it, and the condition under which the rejected option would win.

??? note "Show answer"

    The two numbers are 1,000 writes per second and, at 50 reads per write, 50 x 1,000 = 50,000 reads per second, both measured against what one node does. Assume a single primary on local SSD sustains a few thousand small writes per second; that assumption is reasonable because each write is one row plus its indexes, and the disk rather than the network is the limit.

    At 1,000 per second you are inside that ceiling, and 4 TB fits one node's disk, so nothing in the stated load forces a partition key.

    Take one primary plus read replicas. Reads are the pressure and replicas absorb reads: the replica count is 50,000 divided by the per-replica read ceiling you measure. The price you say out loud is replica lag, so the choice is incomplete without a staleness budget on the read path.

    The rejected option wins when the write rate approaches the primary's ceiling, because a replica adds no write capacity at all: every replica replays the same write stream. It also wins when the data outgrows one node's disk, or when restoring a single 4 TB node takes longer than the recovery target. The deciding number is write throughput or data size, never the read rate.

    An answer that fails: "4 TB is large, so shard it." Size on its own eliminated nothing here, and it commits you to the decision that is hardest to reverse.

**M5** (Objective 5, failure diagnosis). A service is healthy, then a brief dependency blip occurs, and afterwards the service stays broken at the same traffic level it handled a minute earlier. Name the class and the two design changes that prevent it.

??? note "Show answer"

    Metastable failure: the system has entered a state where the work generated by the failure sustains the failure, so removing the original trigger does not recover it. Retries are usually the amplifier, because each failed request produces more load than a successful one.

    Prevention: cap total retry work with a retry budget (a fraction of successful traffic, not a per-request count) plus jittered backoff, and shed load at admission so the queue cannot grow beyond what the service can drain. Recovery usually requires shedding traffic deliberately, which is why the runbook matters as much as the design.

**M6** (Objective 6, defending a single-node or single-region design). Your design is one region and one primary, serving 3,000 requests per second against 200 GB. The interviewer says "that will not scale, distribute it." Give the answer that holds, and the threshold at which you would change your mind.

??? note "Show answer"

    Answer with the rates first, because "will not scale" is not a measurement. The shape is: at 3,000 requests per second against 200 GB, this is one node well inside its ceiling and a working set that fits memory on a commodity machine, so distribution buys headroom I do not need yet.

    Then price what distribution costs, because that is what makes the defence credible rather than stubborn: a partition key chosen now and expensive to reverse, coordination on every operation that touches two keys, split brain as a new failure mode, and, if the push is multi-region, a synchronous write that cannot beat the round trip between the regions.

    For regions 6,000 km apart, light in fibre travels at roughly 200,000 km per second, so 2 x 6,000 / 200,000 is about 60 ms added to every write, before any work is done.

    Then give the threshold. I would partition when sustained write throughput passes roughly half of one node's measured ceiling, or when the working set stops fitting memory, or when the data no longer fits one node with headroom to rebuild it. I would go multi-region when the availability requirement is higher than one region survives, or when data residency law forces it, and not because traffic grew.

    Two answers that fail: agreeing and sharding, which trades a working design for an unmeasured one; and "one node is fine", with no number and no threshold, which is the same missing evidence pointed the other way.

**M7** (Objective 7, brownfield sequencing and the level delta). You are given a running system and a new constraint. State the ordering rule for the work, and state what a staff-level answer adds over a senior one.

??? note "Show answer"

    Ordering rule: cheapest and most reversible first, and re-measure between steps. Each step needs an exit criterion so you know whether to proceed, and a rollback that does not depend on the step having gone well.

    Staff delta: a senior answer designs the target state correctly. A staff answer also sequences the migration against organisational reality (who is on call, what else ships that quarter), names the blast radius of each step, names what will be deprecated and when, and states which decision is expensive to reverse so that it is made last, not first.

**Threshold.** Five of seven, with M3 and M5 among them. M3 and M5 are weighted because breaking points and failure classes are what separate a described architecture from a designed one.

| Missed item | Reread | Redo |
|---|---|---|
| M1 | Module 3, requirements that change the design, and the clarifying branch tables in the URL shortener and chat case studies | Rewrite the branch table for the chat case study from the prompt alone, then compare |
| M2 | Module 4, estimation as elimination | Problems B1 and B2, aloud, saying the eliminated option after every number |
| M3 | Module 14 on simplicity, plus the v0 and breaking point blocks in the news feed and web crawler case studies | Problem I2, and state each breaking point as failure, number, observation |
| M4 | Module 7, building blocks II, storage: the three replication topologies and the partition-key section | Problem B4, then redo the storage choice in the URL shortener case study and state the condition under which the rejected option wins |
| M5 | Module 11, the failure library | Problems I6 and I1 |
| M6 | Module 14 on simplicity and scale thresholds, plus Module 10 for the single-region half of the argument | Problem I3, then defend the V0 in the web crawler case study against a push to distribute, with your threshold said out loud |
| M7 | Module 15 on brownfield design and Module 16 on level calibration | Problem I2, then answer it again at staff level and diff the two |

!!! warning "Scoring well right now is a weak signal"

    Passing this immediately after reading mostly measures that the page is still in working memory. The signal that predicts interview performance is the delayed check in Section 15, taken a week later with the page closed. If you score seven of seven today and four of seven next Tuesday, the honest reading is four.

---

## 13. Common mistakes catalogue

Technical mistakes cost you a follow-up. Delivery mistakes cost you the round. The last four rows are the ones that show up in debrief notes.

| The mistake | Why candidates make it | What the interviewer concludes | What to say instead |
|---|---|---|---|
| Adding a cache before showing a read bottleneck | Caching is the most rehearsed move in every prep resource | Reaches for components by habit, will add cost and staleness to production for no measured reason | "Reads are 3 milliseconds today, so a cache buys under 8 percent of tail. I would add one when database CPU attributable to this query passes about 60 percent" |
| Sharding in the first five minutes | Sharding signals sophistication and the prompt said "scale" | Cannot tell a hundred writes per second from a hundred thousand | "One primary handles this write rate. Here is the number at which it stops, and here is what I would do at that point" |
| Claiming exactly-once delivery | The phrase exists, so it sounds like a setting | Has not worked a delivery-semantics bug | "At-least-once transport plus an idempotency key, with the key retained for the longest retry window, which gives exactly-once effects" |
| Estimating with numbers that decide nothing | Frameworks teach estimation as a mandatory stage | Performs ritual rather than reasoning, and burned six minutes | "That comes to roughly 12 terabytes a year, which rules out keeping it all in memory and keeps blob storage in play" |
| Naming a database by brand instead of by property | Brands are easier to recall than access patterns | Chose the tool before knowing the query | "I need range scans by user and time with high write throughput, so I want a partitioned log-structured store. Several products fit" |
| Ignoring the write path of the read design | Reads are easier to draw | Half a design | "Before I go further on serving, here is what a write does, in order, and where it can fail halfway" |
| Presenting a finished architecture in one move | It looks confident and rehearsed | Cannot show derivation, so the reasoning cannot be assessed | "Here is the simplest thing that meets these requirements. Now let me break it deliberately" |
| Designing before clarifying (delivery) | Silence feels like failure, drawing feels like progress | Will build the wrong thing quickly and confidently | "Two questions first, because the answers change the shape: is this read heavy or write heavy, and does a stale read hurt anyone?" |
| Monologuing for ten minutes (delivery) | Fear that stopping invites a hard question | Cannot collaborate, and the interviewer has no way to steer you off a cliff | "That is the storage layer. Do you want me to go deeper here, or move to how it fails?" |
| Refusing to say "I do not know" (delivery) | Belief that admitting a gap is a scoring event | Bluffs, which is expensive on a real team | "I have not implemented Raft. What I know is what it guarantees and what it costs. Here is how I would decide whether we need it" |
| Defending a choice instead of updating it (delivery) | Reading a follow-up as an attack rather than as data | Will not change a design in a code review either | "You are right that this breaks under a hot key. That changes my partition choice. Let me redraw that part" |
| Running out of time with no conclusion (delivery) | Losing track while deep in one component | No sense of the clock, which is a proxy for scope control at work | At minute 35: "I want to leave five minutes to cover failure modes and what I skipped, so I will close this dive here" |

---

## 14. Interview-day checklist

One screen. Print it or keep it open in a second window. Nothing here is new; it is the operating manual.

**Minute budget**

| Phase | 45 minute round | 60 minute round |
|---|---|---|
| Clarify and scope | 0 to 5 | 0 to 6 |
| Requirements, and estimate only if it eliminates | 5 to 9 | 6 to 12 |
| V0, the simplest correct design | 9 to 15 | 12 to 20 |
| Breaking point, then V1 | 15 to 25 | 20 to 32 |
| Deep dive (one, or two if 60 minutes) | 25 to 36 | 32 to 48 |
| Failure modes, SLIs, one alert | 36 to 42 | 48 to 55 |
| Trade-offs, what you skipped, questions | 42 to 45 | 55 to 60 |

**The first four sentences**

1. "Let me make sure I have the problem: you want X for Y users, and the thing that matters most is Z."
2. "I have two or three questions whose answers would change the design, then I will sketch the simplest version that works."
3. "I plan to spend about ten minutes on requirements and a first design, then break it deliberately, then go deep on one part."
4. "Stop me at any point if you want me somewhere else."

**Five clarifying questions that generalise**

1. What is the read to write ratio, roughly, and which one is growing faster?
2. Who is hurt if a read is thirty seconds stale, and how badly?
3. Is there a hot entity: one user, tenant, key or region that is far larger than the rest?
4. What is the availability target, and is it the same for reads and writes?
5. Is this greenfield, or is something running today that I have to keep working?

**Whiteboard drawing order**

```
DRAW IN THIS ORDER. LEAVE THE DOTTED BOXES OFF UNTIL ASKED.

  1                2                 3
 +--------+      +----------+      +-----------+
 | CLIENT | ---> | SERVICE  | ---> | PRIMARY   |
 |        |      | (one box)|      | DATASTORE |
 +--------+      +----------+      +-----------+
                      |
                      | 4 (only after a number forces it)
                      v
                 +----------+
                 | QUEUE +  |
                 | WORKER   |
                 +----------+

 NOT YET: cache, CDN, shard map, second region, service split.
 Caption: the four boxes to draw first, and the five to withhold.
```

Leave off, until a stated number forces it: cache, CDN, shard map, second region, search index, service decomposition. Adding a box after you have justified it reads as design. Erasing a box you drew in minute three reads as guessing.

**Three recovery scripts**

| Situation | Say this |
|---|---|
| You are stuck | "Let me state what I am optimising for and list the two options I see, then pick one and move: option A trades X for Y, option B the reverse. I will take A because of Z, and I will flag it as reversible." |
| The interviewer disagrees | "That is a case I did not price. Let me take it seriously for a second: if that happens, this part breaks because of X. So I would change Y. Does that address it?" |
| You are running out of time | "I have about six minutes. I want to spend them on how this fails and what I would measure, rather than finishing this dive, because I think that is the higher-value part. Say if you would rather I finish here." |

**Before you finish**

- State the two trade-offs you actually made, and what each one cost.
- State what you deliberately skipped and why, in one sentence each.
- Name the single thing you would measure first in week one, and what number would make you change the design.
- Name the decision that is most expensive to reverse.
- Ask one question that shows you were designing for their system, not a generic one.

---

## 15. Study schedule and spaced review

### 15a. Three plans, each defined by what it drops

Day 1 is whatever day you start. The skip lists are the important part of this section.

**One week (about 8 to 11 working hours)**

| Day | Do | Skip |
|---|---|---|
| 1 | Modules 1 to 4, then problems B1 and B2 | Nothing yet |
| 2 | Modules 5 to 7, then the URL shortener case study end to end | The second deep dive in the case study |
| 3 | Modules 8 and 9, then problem B5 | Paxos beyond what Raft gives you; read the leases part only |
| 4 | News feed and chat case studies, v0 and breaking points only | Both sets of deep dives; you will come back if time allows |
| 5 | Modules 11 and 12, then problems I6 and I1 | Module 13 on cost |
| 6 | Module 15 on brownfield, then problem I2 with a timer | Module 10 on multi-region unless your target company is multi-region |
| 7 | Module 16 on level calibration, then Section 12 mastery check, then Section 14 | Everything you have not reached; do not start new material the day before |

**One weekend (about 5 to 7 working hours)**

- Saturday morning: Modules 2, 3 and 4 (clock, clarifying, estimation). These are the highest-yield hours on the page because they affect every prompt.
- Saturday afternoon: one full case study end to end, out loud, with a timer. Pick URL shortener if you are early career, news feed otherwise.
- Sunday morning: Module 11 (failure library) and Module 14 (simplicity), then problems I3 and I6.
- Sunday afternoon: Section 12, Section 13, Section 14. Rehearse the first four sentences until they are automatic.
- Skip entirely: Modules 9, 10 and 13, all second deep dives, and five of the eight case studies. One case study done properly beats five skimmed.

**One evening (about 2 to 3 hours)**

- 0:00 to 0:30, Section 14 and Module 2. Learn the clock and the opening.
- 0:30 to 1:15, Module 4 and problems B1 and B3. Estimation that eliminates is the fastest visible upgrade to an answer.
- 1:15 to 2:15, one case study, v0 and one breaking point only.
- 2:15 to 2:45, Section 13, read once, then Section 14 again.
- Skip: every module, every deep dive, the mastery check, and all interleaved problems except I3. If your round is tomorrow, extra breadth will not be retrievable under pressure and rehearsed delivery will be.

### 15b. Review schedule

| When | What | Minutes |
|---|---|---|
| Day 1 | The review cards below, once through | 10 |
| Day 3 | Cards, plus redo one blocked problem you got wrong | 20 |
| Day 7 | The delayed self-check in 15d, closed book, then cards | 30 |
| Day 21 | Cards, plus one interleaved problem you have not attempted | 40 |
| Day 60 | One full case study out loud with a timer, then cards | 60 |

Consistency beats the exact intervals by a wide margin. A card session on day 4 and day 9 is worth far more than a perfect schedule you abandon after the first miss. If you miss a checkpoint, do the next one on the day you remember rather than restarting.

### 15c. Review cards

One fact or one decision per card. Copy this block into whatever tool you use. Format is prompt, then two colons, then answer.

```
Q :: A number is allowed in an answer only if it does what?
A :: Rules out a design option in the very next sentence.

Q :: What are the three parts of a breaking point?
A :: What fails, at what number, and how you would observe it.

Q :: Two serial dependencies at 99.95 and 99.9 percent give what?
A :: 0.9985, which is 64.8 minutes of a 43,200 minute month.

Q :: What converts at-least-once delivery into exactly-once effects?
A :: An idempotency key retained for the longest retry window.

Q :: What sets idempotency key retention?
A :: The largest of broker redelivery, client retry and replay windows.

Q :: Failure class where removing the trigger does not recover?
A :: Metastable failure, usually sustained by retry amplification.

Q :: What caps retry amplification?
A :: A retry budget as a fraction of successful traffic, plus jitter.

Q :: What does splitting a hot partition cost?
A :: Reads for that key become an N-way scatter and merge.

Q :: Token bucket versus sliding window log, what decides it?
A :: Memory per active user, roughly 16 bytes against 800.

Q :: What does a token bucket permit that a rolling window forbids?
A :: A full burst immediately after an idle period.

Q :: When is a queue over-engineering?
A :: When the downstream is healthy and the user waits anyway, at a
     few hundred writes per second a polled table is enough.

Q :: What must you say before the round ends, about operations?
A :: One SLI as good over valid events, one page-worthy alert, one
     thirty-second health number.

Q :: Ordering rule for brownfield work?
A :: Cheapest and most reversible first, re-measure between steps.

Q :: What does a staff answer add over a senior one?
A :: Migration sequencing, blast radius, deprecation, and making the
     irreversible decision last.

Q :: A deploy causes an eight minute p99 spike that self-heals. Class?
A :: Warmup, so measure per-instance cache hit rate, CPU and
     connection count before changing anything.
```

### 15d. Delayed self-check

Answer these from memory, a week after you finish, with this page closed. Write the answers down before checking anything.

1. Take any prompt you have not studied here. Produce three clarifying questions where each answer changes a box, and say what changes.
2. Draw a v0 for it, then state one breaking point in the three-part form, then draw the v1 that answers it.
3. Name two failure classes that v1 is exposed to, the design decision that prevents each, and the alert you would page on.

If you can do all three in twenty minutes without notes, the material is retrievable under pressure. If you cannot, the specific step where you stalled is the one to redo, not the whole page.

---

## 16. Reference block

!!! info "This section teaches nothing new"

    Everything below appeared earlier with its derivation. This is the version you come back to the morning of the round. If a row here is the first time you have met an idea, go and read the module it came from rather than memorising the row.

### 16a. Decision tables

**Replication topology**

| If | Choose | Because | Do not choose it when |
|---|---|---|---|
| Reads dominate, stale reads are tolerable | Single leader, async replicas | Cheapest, no write coordination | A read-your-own-write path exists and you have not routed it to the leader |
| Writes must survive leader loss with zero data loss | Single leader, at least one sync replica | Bounded durability guarantee | Write latency budget cannot absorb a cross-zone round trip |
| Writes originate in two regions and both must accept during a partition | Multi leader or leaderless | Availability under partition | You have no conflict resolution the product can live with |
| One dataset, low write rate, high read fan-out | Read replicas plus a cache | Simplest thing that works | Total data fits on one node and read rate is under a few thousand per second, in which case add neither |

**Partitioning strategy**

| If | Choose | Cost you must say out loud |
|---|---|---|
| Access is by a single entity id and no ranges | Hash of the id | Range scans become scatter-gather |
| Access is by entity plus a time range | Composite: entity for placement, time for ordering | Old partitions go cold and uneven |
| One key produces a large share of writes | Composite with a bucket, applied to that key only | That key's reads become an N-way merge |
| Data volume fits one node and will for two years | Do not partition | None, and this is the answer more often than candidates say |

**Delivery semantics**

| Requirement | Mechanism | What it actually costs |
|---|---|---|
| Best effort, loss acceptable | Fire and forget publish | Nothing, and no receipts to debug with |
| Every message eventually processed | At-least-once plus consumer idempotency | Dedupe storage and a retention decision |
| Effect applied once, transport unreliable | Idempotency key with stored result | A conditional atomic write on the hot path |
| Ordered per entity | Partition by entity key, one consumer per partition | Head-of-line blocking behind one slow message |

**Cache placement**

| Symptom | Cache to add | Threshold below which it is over-engineering |
|---|---|---|
| Repeated identical reads of large static objects | CDN, pull-through | Under roughly a terabyte a month of egress on objects with low reuse |
| Repeated identical queries with a slow origin | Read-through cache keyed by query | Origin query under about 5 milliseconds and origin CPU under about 60 percent |
| Expensive derived views recomputed per request | Materialised view refreshed on write | Recomputation under the latency budget at peak |
| Same object requested by thousands at once | Request coalescing at the origin, not a bigger cache | Fewer than a few hundred concurrent misses on one key |

### 16b. The numbers this course uses, and where each comes from

Inputs marked "assumed" are chosen because they are typical order-of-magnitude figures for the scenario as stated, and every derived line stays correct as a method when you substitute the interviewer's number.

| Number | Derivation | What it eliminates |
|---|---|---|
| 43,200 minutes in a month | 30 x 24 x 60, with a 30 day month assumed for arithmetic that a person can do aloud | Arguing availability in percentages, where 99.9 and 99.95 sound similar |
| 86,400 seconds in a day | 24 x 60 x 60 | Guessing per-second rates from daily totals |
| 3.5e12 seven-character codes | 62^7 for a base62 alphabet | Six-character codes at any rate above about 1e8 per day |
| 48 years of code space | 3.5e12 divided by 2e8 new links per day (rate assumed from the prompt) | Eight-character codes as a requirement rather than a preference |
| 386 pages per second | 1e9 pages (assumed target) divided by 30 days x 86,400 seconds | A single-host fetcher, and any design without at least a few hundred hosts in flight |
| 2.25 GB per hour of 1080p | 5 Mbit/s (assumed steady bitrate) x 3600 s / 8 bits per byte | Storing all renditions on block storage; blob storage is forced |
| 610 m by 1.2 km geohash cell at length 6 | 30 bits split 15 lat and 15 lon: 180 degrees / 2^15 = 0.0055 degrees latitude, and 360 / 2^15 = 0.011 degrees longitude, at about 111 km per degree near the equator | Fixed-precision geohashing alone for a dense city, where one cell holds far too many entities |
| 16 bytes per token bucket entry | Two values, current tokens and last refill timestamp, 8 bytes each | Sliding window logs at 800 bytes per user for millions of users |
| 800 bytes per sliding window log entry | 100 requests per window (from the stated limit) x 8 byte timestamps | Exact rolling windows when memory is the binding constraint |
| 250,000 location writes per second | 1e6 active drivers (assumed) divided by a 4 second update interval (assumed) | A single indexed table as the location write target |
| 1,000,000 row writes per celebrity post | One row per follower, at the stated follower count | Pure fan-out on write above roughly the follower count where publish latency exceeds the product's tolerance |
| 64 bytes per idempotency record | Key plus a small pointer to the stored result | A distributed database for dedupe state when the total is single-digit gigabytes |

### 16c. ASCII glyph legend

```
ASCII CONVENTIONS USED IN EVERY DIAGRAM ON THIS PAGE

 +--------+   solid box: a process you run and can page on
 | NAME   |
 +--------+

 +========+   double edge: stateful store, survives restart
 | NAME   |
 +========+

   --->       request path, the caller waits for the reply
   ==>        async path, the caller does not wait
   <-->       session held open in both directions
   |   v      vertical continuation of the same path

 Caption: box style says whether a component holds state, and
 arrow style says whether the caller blocks on the reply.
```

### 16d. One-page summary cards

**Card 1. URL shortener**

| Field | Content |
|---|---|
| Prompt | Design a service that turns a long URL into a short one and redirects |
| Number that decides | 62^7 = 3.5e12 codes against the daily creation rate |
| V0 | One service, one relational table, codes drawn from a pre-allocated counter range |
| Breaking point | Redirect read rate saturates the primary; observe as read latency rising with flat write rate |
| V1 | Read replicas plus an edge cache on the redirect, writes unchanged |
| Deep dive one | Key generation: counter ranges against random-and-check against hash truncation, decided on collision rate as occupancy grows |
| Deep dive two | Redirect path caching, including negative caching so scans for missing codes do not reach the database |
| Failure class to name | Hot key when one link goes viral, plus cache stampede when it expires |
| Staff delta | Abuse scanning before publish, expiry as a storage lever, analytics decoupled so a redirect never waits on a write |

**Card 2. Web crawler**

| Field | Content |
|---|---|
| Prompt | Design a crawler that fetches a billion pages a month and keeps them fresh |
| Number that decides | 386 pages per second, against a politeness rule of one request per host per second |
| V0 | One frontier queue, a fetcher pool, a parser, a document store |
| Breaking point | Politeness serialises per host, so throughput is bounded by hosts in flight, not by fetcher count |
| V1 | Per-host queues with a scheduler that maintains hundreds of hosts in flight and enforces per-host delay |
| Deep dive one | Frontier scheduling: priority by change rate, per-domain budgets, and robots handling with a cache |
| Deep dive two | Deduplication: URL normalisation for near-identical links, then content shingling for identical bodies behind different URLs |
| Failure class to name | Crawler traps producing infinite URL spaces, and head-of-line blocking behind one slow host |
| Staff delta | Recrawl policy derived from observed change rate per site, and an explicit politeness and legal posture |

**Card 3. News feed**

| Field | Content |
|---|---|
| Prompt | Design a home feed showing recent posts from accounts a user follows |
| Number that decides | Row writes per post equals follower count, so one post from a million-follower account is a million writes |
| V0 | Pull on read: query the accounts you follow, merge, sort, return |
| Breaking point | Users who follow thousands of accounts blow the read latency budget; observe as p99 correlated with followee count |
| V1 | Fan-out on write into per-user feed stores, with the read reduced to one range scan |
| Deep dive one | The hybrid threshold: derive the follower count above which fan-out on write costs more than merging at read, and handle those accounts by pull |
| Deep dive two | Feed store layout and truncation: how many entries per user, eviction, and what a backfill costs when someone follows a large account |
| Failure class to name | Celebrity hot key, and thundering herd when a popular feed entry expires from cache |
| Staff delta | A publish latency budget that the fan-out must fit inside, and the cost per feed read stated in currency |

**Card 4. Chat and presence**

| Field | Content |
|---|---|
| Prompt | Design one-to-one and group messaging with delivery receipts and presence |
| Number that decides | Concurrent connections divided by connections a node can hold, which sets node count and therefore routing |
| V0 | One connection server holding sessions, one message store, direct delivery when both parties are connected |
| Breaking point | More than one connection node means the sender's node does not hold the recipient's session |
| V1 | A session registry plus inter-node routing, and a durable per-recipient inbox for offline delivery |
| Deep dive one | Ordering and multi-device sync: per-conversation sequence numbers rather than wall clock, and how a second device catches up |
| Deep dive two | The offline queue: retention, receipts, and de-duplication when a client reconnects and replays |
| Failure class to name | Duplicate delivery on reconnect, clock skew if ordering uses timestamps, presence flapping on mobile networks |
| Staff delta | Presence fan-out costed separately, because subscriptions to presence usually exceed message volume, and what end-to-end encryption removes from the server's options |

**Card 5. Video platform**

| Field | Content |
|---|---|
| Prompt | Design upload, processing and playback for user-generated video |
| Number that decides | 2.25 GB per hour of 1080p source, multiplied by the rendition ladder |
| V0 | Direct upload to blob storage, one transcode worker, playback served from the same blob store |
| Breaking point | Time from upload to publish grows with queue depth; observe as transcode backlog, not as request latency |
| V1 | Chunked upload, segment-parallel transcode, and a delivery path fronted by a CDN with an adaptive bitrate ladder |
| Deep dive one | The upload pipeline: resumable chunks, integrity checks, and what happens when the client dies at 80 percent |
| Deep dive two | The delivery path: segment sizing, ladder selection, and what a premiere event does to cache fill |
| Failure class to name | Retry storm on upload from mobile clients, and cache miss storm at a scheduled release |
| Staff delta | Storage tiering by observed popularity, ladder trimming for long-tail content, and cost per stored hour against cost per delivered hour |

**Card 6. Ride hailing**

| Field | Content |
|---|---|
| Prompt | Design driver location tracking and rider-to-driver matching |
| Number that decides | 250,000 location writes per second from a million active drivers updating every 4 seconds |
| V0 | One service writing driver positions to an indexed table, matching by bounding-box query |
| Breaking point | The index cannot absorb the write rate; observe as write latency rising while query rate is flat |
| V1 | Split the paths: last-known position in an in-memory geospatial structure for matching, durable history written asynchronously for billing and analytics |
| Deep dive one | The location pipeline: update frequency as a tunable, adaptive rates when a driver is idle, and what precision the matcher actually needs |
| Deep dive two | Dispatch matching: greedy assignment against short batching windows, and the fairness and latency trade-off between them |
| Failure class to name | Hot cell in a dense area, and split brain assigning two riders to one driver |
| Staff delta | Matching framed as an optimisation over a window with explicit fairness constraints, and per-city isolation so one market cannot take down another |

**Card 7. Proximity search**

| Field | Content |
|---|---|
| Prompt | Design "find the nearest N places within R kilometres" |
| Number that decides | A length-6 geohash cell is about 610 m by 1.2 km near the equator, which sets how many entities land in one cell |
| V0 | Indexed latitude and longitude columns with a bounding-box query and a distance filter |
| Breaking point | Dense areas put far too many rows in one cell while rural cells are empty; observe as p99 by city, not overall |
| V1 | A spatial index that adapts to density, with the cell size chosen per region rather than globally |
| Deep dive one | Geohash against quadtree, decided on index rebuild cost and on how the structure handles skew, not on elegance |
| Deep dive two | Border correctness: why a single-cell lookup misses near neighbours across a boundary, and the neighbour-cell expansion that fixes it |
| Failure class to name | Hot cell during an event, and stale index if entities move faster than the rebuild cadence |
| Staff delta | Separating a mostly static place index from a fast-moving object index, because they have opposite rebuild economics |

**Card 8. Typeahead suggestions**

| Field | Content |
|---|---|
| Prompt | Design search suggestions that appear as the user types |
| Number that decides | The end-to-end budget per keystroke, which has to cover network round trip plus lookup |
| V0 | One in-memory trie with the top completions precomputed at each node, rebuilt nightly |
| Breaking point | The trie no longer fits one node, or the rebuild cadence is slower than the rate at which popular queries change |
| V1 | Trie partitioned by prefix, with a streaming counter feeding incremental top-k updates |
| Deep dive one | Prefix partitioning and the load skew it creates, since prefixes are not uniformly popular and the split must follow traffic, not the alphabet |
| Deep dive two | Ranking updates: batch rebuild against streaming counts, and how fast a breaking-news query must be able to enter the suggestions |
| Failure class to name | Hot prefix shard, and stale suggestions after a traffic spike the batch job has not seen |
| Staff delta | The tension between personalisation and cacheability stated as a number, plus per-market indexes and the cost of spell correction on the hot path |

---

## 17. What this course does not cover

Naming our limits is cheaper than pretending we have none, and it saves you from studying the wrong thing.

| Not covered | Why | Where to go instead |
|---|---|---|
| Coding and algorithms rounds | A different skill with a different practice loop, and it would double the page without improving a design answer | The coding sections of this site, and any standard problem set |
| Machine learning system design | Ranking, retrieval, training and serving deserve their own arc | Our Machine Learning System Design Interview course in this section |
| Frontend, mobile and interface contract rounds | Each has a distinct scoring rubric that a backend distributed-systems page teaches badly | Our Frontend, Mobile and Product Architecture courses in this section |
| Storage engine internals below the access-pattern level | We stop where the interview stops. B-tree and log-structured merge implementation detail rarely changes a design answer | Database Internals, Alex Petrov, 1st edition, O'Reilly, 2019 |
| Formal treatment of consensus, proofs and model checking | Interview depth is what the protocol guarantees and what it costs, not a proof | In Search of an Understandable Consensus Algorithm, Ongaro and Ousterhout, USENIX ATC 2014, with the extended version at raft.github.io |
| Distributed systems theory in general | We teach decisions, not derivations, and the standard reference is better than anything we would write | Designing Data-Intensive Applications, Martin Kleppmann, 1st edition, O'Reilly, 2017. A second edition has been in preparation, so check its status before buying |
| Stream processing beyond interview depth | Watermarks and windowing appear here as design constraints, not as an engine tutorial | Streaming Systems, Akidau, Chernyak and Lax, 1st edition, O'Reilly, 2018 |
| Running the systems you design | Operating is a career, not a chapter. We cover only the operability a designer must state | Site Reliability Engineering, Beyer, Jones, Petoff and Murphy, O'Reilly, 2016, and The Site Reliability Workbook, 2018, both readable free at sre.google/books |
| Resilience patterns as an engineering practice | We name the failure classes; the practice of building for them is a book | Release It!, Michael Nygard, 2nd edition, Pragmatic Bookshelf, 2018 |
| Cloud provider specifics, pricing and managed service selection | Prices and service names change faster than we can review a page, and a design answer should not depend on a brand | Provider documentation and calculators, checked on the day you need them |
| Security engineering, threat modelling and cryptography | Adjacent and large. We mention isolation and abuse only where a design decision turns on it | The provider and standards documentation for the specific control you need |
| Behavioural rounds and company-specific loop formats | Loop structures change per team and per quarter, and unverifiable claims about them are exactly what we refuse to publish | Ask your recruiter for the current format, in writing |
| Verified analyses of specific databases under partition | We do not run these tests, so we do not report their results | Jepsen reports at jepsen.io, each dated and versioned against the software tested |
| A broad link farm of prompts | Coverage without derivation is what we are trying to beat | The System Design Primer and awesome-scalability, both free community repositories on GitHub that we do not control or endorse |

!!! note "On anything we cite"

    We name the edition and the year so you can tell what has aged. If a link here is dead or a newer edition exists, open an issue and it gets fixed in the next review pass rather than sitting stale behind an unverifiable "updated recently" badge.

---

## 18. Provenance

**Last reviewed:** August 2026. A page whose last-reviewed date is more than nine months old is flagged for review rather than quietly left in place.

**Changelog, most recent first**

| Date | Change |
|---|---|
| August 2026 | First publication of the full sixteen-module course with eight case studies, public rubrics for every practice problem, and the level delta blocks |
| August 2026 | Reference block split out from the teaching sections after review, so returning readers do not have to scroll through prose to reach a decision table |
| August 2026 | Every figure in Sections 10 and 16 re-derived from stated inputs, and inputs that could not be derived were relabelled as assumptions with the reason they are reasonable |

**Found a problem?** Open an issue at [github.com/datascienceinterviews/datascienceinterviews.github.io/issues](https://github.com/datascienceinterviews/datascienceinterviews.github.io/issues). Corrections, disputed numbers, dead links and better derivations are all in scope. Errors get fixed in days, and the changelog above records what changed.

All content on this page is original. Research informed which topics belong here; no prose, framework, example, table or diagram is derived from any paid course. Primary sources are cited inline with their edition and year, and any claim we could not source is either labelled as an assumption or is not on the page.
