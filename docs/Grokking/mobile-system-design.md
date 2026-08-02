---
title: Mobile System Design Interview
description: A free course on designing the mobile client as a distributed node: budgets, offline sync, media pipelines, push, battery, version skew, release health.
last_reviewed: 2026-08-01
---

# Mobile System Design Interview

!!! abstract "The contract"

    **After this course you can** take a one-line mobile prompt, draw a client that keeps working on a hostile network and through process death, attach a measurable budget to every constraint it must hold, name the number at which it breaks, and say how a release you cannot recall would tell you it had.

    **Who it is for**

    - An iOS or Android engineer with roughly 2 to 8 years of shipping apps, facing a 45 or 60 minute design round.
    - A backend or full-stack engineer moving onto a client team, who can design the service but stalls on process death, version skew and battery.
    - A senior mobile candidate whose feedback keeps saying "described screens, not a system", and who cannot tell which minutes went to the wrong thing.

    **Who it is not for**

    | If your round is about | Go here instead |
    |---|---|
    | A browser application, or one interactive component with rendering and bundle budgets | [Frontend System Design](frontend-system-design.md) |
    | The server side: services, stores, pipelines, capacity, consistency, consensus | [Modern System Design](modern-system-design.md) |
    | The API contract itself: resources, error models, pagination semantics, idempotency, webhooks, versioning | [Product Architecture](product-architecture.md) |
    | Choosing a model, labels, ranking metrics or a training loop, even for an on-device model | [ML System Design](ml-system-design.md) |
    | A feature built on a foundation model: retrieval, agents, tokens, cost | [Generative AI System Design](generative-ai-system-design.md) |
    | Writing Swift or Kotlin against a timer, or implementing a screen live | [Data structures and algorithms bank](/Interview-Questions/data-structures-algorithms/) |
    | A round inside the next week | [Fast-Track in 48 Hours](system-design-fast-track.md) |

    **Trains for:** the 45 and 60 minute mobile system design round in iOS, Android and cross-platform loops, the component and SDK variant of that round, and the design portion of senior and staff mobile loops.

    **Does not train for:** language or framework trivia screens, live UI implementation, algorithm rounds, product-sense rounds, or behavioural rounds.

    **Fast path:** already comfortable? Go straight to the [self-assessment rubrics](#11-self-assessment-rubrics). Just want the tables? Go to the [reference block](#16-reference-block).

---

## Prerequisites

Seniority is a weak readiness signal, and a job title is worse. Prerequisite mastery is the signal that works. Answer every Quick check in under a minute. Two or more misses means start with the linked material rather than with Module 1.

| You should already be able to | Where to learn it | Quick check |
|---|---|---|
| Say what the operating system can do to your process while your code is not running | [Android activity lifecycle](https://developer.android.com/guide/components/activities/activity-lifecycle), [Apple app life cycle](https://developer.apple.com/documentation/uikit/managing-your-app-s-life-cycle) | Your app is uploading a 40 MB video and the user switches away. Name two different things the OS can do next, and say which one loses the upload |
| Convert a payload size and a link rate into wall-clock seconds without a calculator | [Estimation anchors on the hub](index.md#anchors-worth-memorising) | A screen fetches 1.4 MB. Usable throughput is 400 kbit per second. Roughly how many seconds until the last byte lands? |
| Turn a refresh rate into a per-frame time budget | [The latency ladder on the hub](index.md#the-latency-ladder) | At 120 hertz, how many milliseconds does one frame get, and what is left for your code if the system compositor takes 3 ms? |
| Read a cache and conditional-request header set and say what the client does next | [RFC 9111, HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111.html) | A response carries `max-age=0` and an `ETag`. What does the client send on the next read, and what does the server return if nothing changed? |
| Say what makes a retried write safe to repeat | [Hub glossary](index.md#glossary) | Your client retries a POST after a timeout. Name the one field that makes the duplicate harmless, and say who generates it and when |
| Size a decoded image in bytes from its pixel dimensions | [Estimation anchors on the hub](index.md#anchors-worth-memorising) | A 4032 by 3024 photo, decoded at 4 bytes per pixel. How many megabytes, and how many screens' worth is that at 1080 by 1920? |
| Read a percentile on a fleet of devices rather than an average | [Android app startup time](https://developer.android.com/topic/performance/vitals/launch-time), [Google SRE Book, chapter 6](https://sre.google/sre-book/monitoring-distributed-systems/) | Cold start p50 is 900 ms and p90 is 3.2 s. Who is in that p90, and would shaving 100 ms off the p50 path move it? |
| Say why a client defect cannot be fixed the way a server defect can | [Hub glossary, blast radius](index.md#glossary) | You merge the fix now. Roughly what share of installs runs it an hour later, and what does that force you to have built beforehand? |

??? note "Show answers to the quick checks"

    1. It can suspend the process (your upload stalls and may resume) or terminate it outright to reclaim memory (in-memory upload state is gone). Termination is the one that loses data, and it loses it silently unless the upload was written to durable storage before it started.
    2. 1.4 MB is about 11.2 megabits. 11,200 kbit divided by 400 kbit per second is 28 seconds. That is one screen, on one request, with no round trips counted. It is the number that kills a design that fetches everything up front.
    3. 1000 divided by 120 is about 8.3 ms per frame. Minus 3 ms of compositor cost leaves roughly 5.3 ms for layout, decode and draw on the main thread. Any single main-thread job longer than that drops a frame.
    4. `max-age=0` means the stored copy must be revalidated before reuse, so the client sends the same request with `If-None-Match` carrying the ETag. If nothing changed the server returns 304 with no body, which costs a round trip but not the payload.
    5. An idempotency key, generated by the client before the first attempt and reused unchanged on every retry of that same logical write. Generating it per attempt is the classic bug: every retry then looks like a new request.
    6. 4032 times 3024 is 12,192,768 pixels, times 4 bytes is about 48.8 MB. A 1080 by 1920 screen is 2,073,600 pixels, about 8.3 MB. So one full-resolution decode is roughly six screens of memory, which is why decoding at display size rather than source size is the whole game.
    7. Low-tier devices: less RAM, slower storage, more thermal throttling, and more likely to be starting cold because the OS evicted the process. Shaving 100 ms off a path that all devices run moves p50 and barely moves p90, because the p90 is dominated by work that scales with CPU and disk speed.
    8. Close to zero. Review, staged rollout and user update behaviour mean the fix reaches the fleet over days, and some installs never take it. That forces the fix path to exist before the incident: remote config, a server-side kill switch, and a response shape the old app version can still parse.

---

## Time budget

| Part of the course | Reading | Practice | Spaced review |
|---|---|---|---|
| Modules 1 to 3, scope, the six axes, budgets | 65 to 85 min | 9 to 15 min | included below |
| Modules 4 to 7, client shape, backend for frontend, networking, pagination | 114 to 150 min | 12 to 20 min | included below |
| Modules 8 to 11, images, uploads, push and transports, sync | 153 to 206 min | 12 to 20 min | included below |
| Modules 12 to 15, storage tiers, battery, concurrency, platform variation | 96 to 126 min | 12 to 20 min | included below |
| Modules 16 to 20, security, release, SDK design, mechanics, levels | 112 to 153 min | 15 to 25 min | included below |
| Eleven case studies, attempted before reading each arc | included above | 154 to 198 min | included below |
| Two practice sets, eleven problems | included above | 156 to 212 min | included below |
| Two timed self-mocks, scored against the public rubric | 0 | 110 to 150 min | included below |
| Five review sessions on days 1, 3, 7, 21 and 60 | 0 | 0 | 180 to 240 min |
| **Total** | **540 to 720 min, so 9 to 12 h** | **480 to 660 min, so 8 to 11 h** | **180 to 240 min, so 3 to 4 h** |

!!! note "How these were computed, so you can argue with them"

    Reading is summed per module from the Module Map table below, never guessed for the page as a whole. The twenty module ranges total 540 minutes at the low end and 720 at the high end, which is the 9 to 12 hour figure. Per-module minutes assume 120 to 150 words a minute, a deliberately slow engaged rate, because dense technical prose with diagrams and byte arithmetic is not read at prose speed.

    Practice itemises to five components that sum to the column above:

    | Practice component | Unit cost | Count | Total |
    |---|---|---|---|
    | Attempt each case study before reading its arc | 14 to 18 min | 11 | 154 to 198 min |
    | Faded examples inside the modules | 3 to 5 min | 20 | 60 to 100 min |
    | Blocked practice set | 12 to 16 min | 5 | 60 to 80 min |
    | Interleaved and cumulative set | 16 to 22 min | 6 | 96 to 132 min |
    | Timed self-mock plus rubric scoring | 55 to 75 min | 2 | 110 to 150 min |
    | **Sum** | | | **480 to 660 min** |

    The five faded-example rows in the main table (9 to 15, 12 to 20, 12 to 20, 12 to 20, 15 to 25) are that 60 to 100 minute line split across the module clusters. Review assumes five sessions of 36 to 48 minutes, which is the 180 to 240 minute column.

    Every figure is padded by roughly 15 percent, because self-estimates of study time run optimistic. Minutes are the authoritative unit here; a single headline number would be more marketable and less true. At an hour a day this is three weeks, full time it is four days.

---

## Learning objectives

Each objective is tested by exactly one numbered item in Section 12, and the numbering matches: objective 1 is tested by item 1, objective 7 by item 7.

1. **Classify** a mobile prompt inside the first two minutes as client-scope, component or SDK scope, or a server round wearing a mobile title, and **state the single sentence that steers a misfiled round** without contradicting the interviewer.
2. **Derive**, for a stated device tier and network profile, a budget set covering cold start in milliseconds, bytes on the wire for the first screen, and peak resident memory, showing each arithmetic step, and **name the design option that each number eliminates**.
3. **Design** a paginated and cached list path over a feed that mutates while the user scrolls, **naming the pagination scheme chosen and the exact user-visible defect** (duplicate, skip or reordering) that each rejected scheme would have produced.
4. **Specify** an offline write path for a stated connectivity profile, giving the durable queue, the retry policy with jitter, the idempotency key and the conflict rule, and **state what the user sees at each of send, retry, conflict and permanent failure**.
5. **Choose** a transport from push, polling, server-sent events, websockets and a persistent broker for a stated update rate and latency need, **citing the messages-per-connection-per-hour figure that decides it** and the battery cost the rejected option would have carried.
6. **Diagnose** a described client regression (cold start, dropped frames, memory kill, battery drain, lost upload) into exactly one named cause, and **name the single on-device measurement that confirms it**, in under three minutes.
7. **Rewrite** a mid-level mobile answer into a staff-level one by adding version skew across app versions you cannot force-upgrade, a server-side kill switch, one release health metric with its rollback threshold, and one degradation path, annotating what each addition changed about the decision.

---

## Warm-up retrieval

Three to five minutes. Answer out loud or in writing before opening anything. Retrieval beats rereading, and a visible answer converts one into the other.

**Q1.** From the hub's round router: name two signals inside the first minute that tell you this is a client design round rather than a generalist distributed-systems round.

??? note "Show answer"

    First, the artefact you are asked to draw lives on a device: a screen, an app, a library that ships inside someone else's binary. Second, the first follow-up is about a constraint the server does not have, such as what happens offline, what happens when the process is killed, or what the old app version does with the new response. A third, weaker signal: the interviewer asks who owns the release, which is a question about deployment latency and only matters on the client.

**Q2.** From the trade-off atlas on the hub, several pages back: state the decision rule for long polling versus websockets versus server-sent events, and name the number that decides it.

??? note "Show answer"

    Long polling when updates are rare and the path must stay plain HTTP. Websockets when traffic is frequent, bidirectional and low latency, and you can operate sticky connections. Server-sent events for server-to-client text with free reconnect. The deciding numbers are messages per connection per minute and concurrent connections against per-connection memory. Getting it wrong either way has a name: websockets everywhere creates connection state you must shed and rebalance, and long polling at a high message rate burns one request per update.

**Q3.** From the hub's failure library: name the failure class where a fleet of clients all reconnect at the same instant, and give the design change that prevents it.

??? note "Show answer"

    Thundering herd, which on a client fleet usually arrives as mass reconnect after a disconnect or as a shared cron minute. The prevention is a per-client random offset: jittered backoff on reconnect, and schedules spread by a random offset rather than aligned to the top of the hour. The mobile-specific twist is that you cannot patch the clients that are causing it, so the jitter has to be in the version already shipped or the server has to shed load and stagger the reconnect itself.

---

## Module map

```
+--------------------------------+
| FRAME               M1 to M3   |
| scope, the six axes, budgets   |
| cold start, frames, bytes, RAM |
+---------------+----------------+
                |
                v
+--------------------------------+
| CLIENT SHAPE        M4 to M7   |
| layers, backend for frontend,  |
| networking, pagination         |
+---------------+----------------+
                |
                v
+--------------------------------+   +----------------------+
| DATA IN AND OUT     M8 to M11  |   | THE DEVICE           |
| images, uploads, push and      |==>| M12 storage tiers    |
| transports, the sync engine    |   | M13 battery and life |
+---------------+----------------+   | M14 threads, memory  |
                |                    | M15 platform gaps    |
                |                    +-----------+----------+
                |                                |
                v                                v
+-------------------------------------------------------+
| SHIP AND DEFEND                        M16 to M20     |
| security and privacy, release health and kill         |
| switches, library and SDK design, interview           |
| mechanics, level calibration                          |
+-------------------------------------------------------+

Caption: the five clusters of this course and their reading order.
Frame feeds Client Shape, because you cannot pick an architecture
before you have a budget to hold. Client Shape feeds Data In And
Out, the media, transport and sync work. That cluster feeds The
Device, which is where every one of those choices is paid for in
storage, battery, threads and platform differences. Both feed
Ship And Defend, where mid and staff answers separate. Arrows
point from prerequisite to dependent.
```

| Module | What you will be able to do | Time | Skip if |
|---|---|---|---|
| M1 Scope and honesty | Tell inside the first minute whether you are in a client round, a component round or a server round with a mobile title, and steer back in one sentence | 12 to 16 min | You have your exact round format in writing and it matches what you practise |
| M2 The six axes | Name the six ways a client differs from a service (runtime, network, deployment latency, interaction model, fragmentation, privacy posture) and say which axis a given prompt turns on | 18 to 24 min | Never skip. Every budget and every trade-off later on hangs off one of these six, and it costs twenty minutes |
| M3 Budgets with numbers | Derive cold start, frame, memory, bytes-per-screen and battery budgets from a stated device tier, and delete any number that eliminates nothing | 35 to 45 min | You can already state what each of your numbers ruled out, every time |
| M4 Client architecture | Separate a client into layers, pick a presentation pattern and say where it stops scaling, and treat navigation as an architectural layer rather than a detail | 35 to 45 min | You have modularised a real app and can state the boundary that stopped a build-time regression |
| M5 Backend for frontend | Decide what belongs on the server, shape a response for one screen, and version a contract across app versions you cannot force-upgrade | 22 to 30 min | You own a mobile-facing endpoint and have already shipped a backward-compatible shape change |
| M6 Networking under constraint | Choose a protocol, coalesce and prioritise requests, classify retries and add jitter, and apply backpressure when the device is the bottleneck | 35 to 45 min | You have debugged a client retry storm and can name the tier where you capped it |
| M7 Pagination done properly | Pick offset, page number, cursor or keyset for a mutating feed, and name the duplicate, skip or reorder each rejected option produces | 22 to 30 min | You have shipped a keyset-paginated feed and can explain what happens when an item is deleted mid-scroll |
| M8 Media in | Design an image pipeline end to end: decode and downsample, bitmap budgets, three cache tiers with eviction, cancellation on recycle, prefetch ahead of scroll | 38 to 52 min | You have written or replaced an image loader and can state its memory ceiling from memory |
| M9 Media out | Design chunked resumable upload that survives process death, with a durable queue, compression decisions, progress and recovery after a force-quit | 32 to 42 min | You have shipped background upload and can describe what your app does when the user kills it mid-send |
| M10 Push and real-time | Run a token lifecycle with dead-token reaping, reason about best-effort delivery, priority, collapse keys and time to live, and choose a transport with battery attached | 38 to 52 min | You operate a push pipeline and can state your token churn rate and your badge consistency rule |
| M11 The sync engine | Build an outbox, choose between operation log and state sync, handle tombstones and clock skew, and work a CRDT example rather than naming one | 45 to 60 min | You have shipped multi-device sync and can describe a convergence bug you fixed |
| M12 Storage and caching on device | Place data across key-value, embedded database, secure storage and memory cache, set eviction, and say what survives a reinstall | 22 to 30 min | You can already state which of your app's stores is backed up, which is encrypted and which is disposable |
| M13 Lifecycle, background and battery | Schedule deferrable work inside OS-imposed limits, adapt in low-power mode, and attribute a drain to a subsystem with a measurement | 28 to 35 min | You have chased a real battery regression to a named subsystem |
| M14 Concurrency and memory | Keep the main thread clear, structure cancellation, respond to memory warnings, and find a leak with a profiler rather than by reasoning | 26 to 35 min | You profile routinely and have fixed a leak found from a heap dump rather than from a guess |
| M15 Platform variation | Name the real primitive on each platform for the same job, and say where the cross-platform answer differs rather than pretending it does not | 20 to 26 min | You have shipped the same feature on both platforms and can list where the designs had to diverge |
| M16 Security and privacy | Place secrets and tokens correctly, decide on pinning, design a permission request with a denial fallback, and design as if the client is compromised | 28 to 35 min | You have handled a token refresh race and a permission denial path in production |
| M17 Release engineering | Design flags, deterministic bucketing, offline-durable telemetry, staged rollout, forced update, kill switches and release health metrics with thresholds | 26 to 37 min | Never skip if you are targeting senior or above. This is the cluster that separates a client engineer from a client system designer |
| M18 Library and SDK design | Design a public API surface, threading contract, error model, initialisation cost, binary size budget and a versioning policy for code inside someone else's binary | 26 to 37 min | You maintain a published SDK and can state its cold-start cost and its binary size delta |
| M19 Interview mechanics | Run the clock, draw the client diagram in an order the interviewer can follow, and descope at the twenty-five minute mark without sounding lost | 16 to 22 min | You have run at least one timed self-mock and finished with time left |
| M20 Level calibration | Say what your answer is missing to read one level higher | 16 to 22 min | Never skip if you are targeting senior or above |

Low ends total 540 minutes, high ends total 720. That is the 9 to 12 reading hours in the time budget above.

---

## The delivery clock for this round

The hub owns the [shared clock](index.md#the-clock) used across all seven courses. This section is the mobile variant of it, and the shape is genuinely different: fewer minutes on capacity, more on what happens when the network, the OS or the release process takes something away from you. Read the hub's version once; do not read it twice.

Most candidates who fail a mobile round did not lack a fact. They drew a screen, a view model and a repository, then answered every follow-up inside that picture, never committing to a budget, an offline behaviour, or a plan for the version of the app that is already in the store.

### The 45 minute round

| Minutes | Spend it on | You leave this phase with |
|---|---|---|
| 0 to 2 | Restate the prompt as a client boundary: what runs on device, what the server owes you | The interviewer agreeing, or correcting you cheaply |
| 2 to 8 | Clarify: offline expectations, multi-device, device tier floor, which platform, who owns the API | Three or four constraints tagged GIVEN or ASSUMED |
| 8 to 12 | Budgets, only where a number eliminates an option: bytes per screen, memory ceiling, start target | One or two decisions already made, not a page of arithmetic |
| 12 to 20 | v0 drawn end to end: screen, data layer, cache, network, storage | A correct client at the stated scale, on the board |
| 20 to 29 | The breaking point, then v1 | A named budget, the number it breaches, and the one layer you added |
| 29 to 39 | One deep dive, ideally the one the interviewer picks | Depth in one area, with a trade-off stated both ways |
| 39 to 45 | Failure modes, what you would instrument, what you skipped and why | An explicit list of what you did not cover, and one first measurement |

The phases sum to 45 by construction: 2 plus 6 plus 4 plus 8 plus 9 plus 10 plus 6.

### The 60 minute round

| Minutes | Spend it on |
|---|---|
| 0 to 2 | Open, and restate as a client boundary |
| 2 to 10 | Clarify: offline, multi-device, device floor, platform, API ownership, release cadence |
| 10 to 15 | Budgets that eliminate |
| 15 to 24 | v0 end to end |
| 24 to 35 | Breaking point and v1 |
| 35 to 51 | Two deep dives, different in kind from each other |
| 51 to 58 | Offline and failure behaviour, telemetry, release health, cost on the device |
| 58 to 60 | Close: trade-offs, what you skipped, what you would measure first |

These sum to 60: 2 plus 8 plus 5 plus 9 plus 11 plus 16 plus 7 plus 2. The extra fifteen minutes buys a second deep dive and a real operations section. It does not buy more boxes on the client diagram.

### The first two minutes

Say four sentences, in this order. Rehearse them once so they are automatic and you can spend your attention on listening.

1. "Let me restate the boundary: the client owns X, the server owes me Y, and the contract between them is Z." (If you cannot fill all three slots, that is your first clarifying question.)
2. "I am going to spend about six minutes on constraints and budgets, then draw the client end to end, then break it on purpose." (You just told them your clock, which reads as control.)
3. "I will assume this has to work with no network and survive the process being killed, unless you tell me otherwise." (Naming those two up front is the cheapest senior signal in this round.)
4. "Stop me if you want depth somewhere specific rather than coverage." (Interviewers take this invitation more often than candidates expect.)

### Declaring what you are skipping

Skipping silently reads as not knowing. Skipping out loud reads as prioritising. The pattern is: name it, say why it cannot change a decision in the next twenty minutes, then say in one sentence what you would do if it were in scope.

- "I am not designing the server's storage layer. Whether the feed is in one shard or fifty does not change anything on the device, and the tight constraint here is bytes on the wire per screen."
- "I am skipping accessibility of the individual cells, which matters and is real work. In scope it would be a focus order, a content description per element and a reflow test at the largest font size."
- "I will not design the analytics schema. Nothing in this client changes based on the event names, because the deciding constraint is the durable queue and its flush policy, and I will design that instead."

### Checking in without sounding unsure

A weak check-in asks for approval. A strong one offers a fork.

| Weak, reads as unsure | Strong, reads as driving |
|---|---|
| "Does that make sense?" | "That is the read path. Do you want the write path next, or the cache eviction policy behind it?" |
| "Is this the architecture you wanted?" | "I have assumed a single device per account. Correct me now if it is multi-device, because that decides whether I need a sync engine or just a cache." |
| "Should I go deeper?" | "I can go one level into the upload resumption, or stay wide and cover release health. Which is more useful to you?" |
| "Sorry, I am rambling." | "I am three minutes over on the cache tiers. Moving to the offline queue." |

### How the split shifts by level

| Phase | Mid | Senior | Staff |
|---|---|---|---|
| Clarify and scope | Same | Same | Longer: challenges whether the feature belongs on the client at all |
| Budgets | Full, shown step by step | Trimmed to the numbers that decide the data layer | Often one number, then a support cost or a device-tier consequence |
| Offline and process death | Named as a concern | A designed path with a user-visible state at each stage | The constraint the rest of the design is built around |
| v0 and v1 | Most of the time here | Balanced | Compressed, because it is assumed correct |
| Deep dives | One, chosen by the interviewer | Two, one chosen by the candidate and defended | Two, plus the migration path from the app already in the store |
| Release and operations | Mentioned at the end | Flags plus staged rollout, with a stated rollback criterion | Woven throughout: version skew, kill switch, release health thresholds, who gets paged |

The rough shape: mid level spends its minutes proving a working client could be built, senior spends them proving each constraint was chosen deliberately, staff spends them proving the design is reachable from the binary that is already installed on millions of devices and survivable when it is wrong.

!!! tip "One note on frameworks, made once"

    Educative's paid courses each ship an acronym spine and apply it identically to every case study; the hub [lists them with dated sources](index.md#the-shared-design-framework). Those are theirs, not ours.

    The problem is not the letters. It is that one spine applied to six prompts produces an answer that sounds recited, and interviewers who run these rounds weekly now screen for that tell. The clock above is a time budget. It never appears as a heading in any case study on this page, and Module 1 teaches when to abandon it.

!!! danger "When to break this clock"

    The measurement that justifies following the clock: you have run one timed self-mock and finished with a client on the board, a failure path stated, and time left. Below that, the clock is scaffolding you still need. Above it, treat it as advisory. Three situations specific to this round where following it is the wrong move:

    **1. The interviewer is running a server round under a mobile title.** If minute five is "how would you shard the feed store", the round has drifted, and the drift is not always deliberate.

    Steer once, explicitly and cheaply: "Happy to go there. Before I do, is this round scored on the client or on the service? I want to spend the minutes where you are grading." Then follow whatever they say, and follow it fully. Fighting to get back to the device after they have answered is the most expensive thing you can do in a 45 minute round.

    **2. The prompt is a component or an SDK, not an app.** "Design an image loading library", "design an analytics SDK", "design a feature flag client". There is no product surface to clarify and no screen to draw, so the clarify and v0 phases collapse. The minutes go instead to the public API surface, the threading contract, the error model, initialisation cost on startup, binary size, and the versioning policy for code that ships inside a binary you do not control.

    Drawing a three-layer app architecture for a library prompt is the fastest way to signal you did not hear the question.

    **3. The prompt is a regression in a shipped app.** "Cold start got 40 percent worse after a feature merge, diagnose and fix it." There is no greenfield v0, so drawing one wastes ten minutes. The minutes go to the measurement plan, what you would bisect and how, the trace you would read and what you expect to see in it, the fix, and the flag it hides behind while it rolls out.

    A fourth, quieter one: read who is in the room. A platform specialist will follow you into decode costs, main-thread scheduling and OS background limits; an architect will follow you into layering, contracts and rollout. Both are asking a fair question. Rebalance the two deep dives towards whichever one they lean into, and say out loud that you are doing it.

## Module 1. Scope and honesty: what a mobile design round actually tests

**Time: 20 to 30 minutes reading, 10 to 15 minutes practice.**

### 1a. Predict first

Five opening lines, all delivered in a slot the recruiter labelled "Mobile System Design".

| # | The opening line |
|---|---|
| A | "Design a photo sharing app." Nothing more is said. |
| B | "Design offline mode for a note taking app that syncs across three devices." |
| C | "We store five hundred million photos. How would you shard and serve them globally?" |
| D | "Design an image loading library that a hundred teams inside this company would depend on." |
| E | "Cold start went from 1.2 seconds to 1.9 seconds after last month's merge. Diagnose it and design the fix." |

??? note "Show answer"

    Prediction asked for: which line is a backend round wearing a mobile title, which line is ambiguous until you ask, and which three are unambiguously client work.

    | # | What it really is | Your first move |
    |---|---|---|
    | A | Ambiguous. A photo sharing app contains a feed client, a camera pipeline, a fan-out service and a photo store | Ask which surface, do not guess |
    | B | Client design, sync flavoured. The hard part is convergence and conflict on a device that goes offline | Start with the local store and the outbox |
    | C | Backend design under a mobile title. Sharding a photo store has no client in it | Offer the client half explicitly |
    | D | Component or library design. Real, common, and the paid competition ignores it | Design an API surface, not an app |
    | E | Brownfield client performance. Scored on measurement discipline, not on boxes | Ask for the trace before proposing a cause |

    The tell for C is that nothing in the prompt has a user watching it. The tell for D is a consumer that is another engineer. The tell for E is a named regression with a number attached.

### 1b. The four prompt shapes and what each one scores

A mobile design round is not one question type. Four shapes recur, and they reward different behaviour.

| Shape | The deliverable | Dominant risk in a weak answer | Minutes go to |
|---|---|---|---|
| Product surface ("design the feed screen of X") | A client architecture plus the data and state flow behind one or two screens | Drawing a server topology and never opening the device | Local store, cache tiers, pagination, render cost |
| Component or SDK ("design an image loader, an analytics SDK") | A public API, a threading contract, an error model, a resource budget | Designing an app instead of a library | API surface, lifecycle, back pressure, binary size |
| Offline or sync ("make it work on a plane") | A write path that survives process death and reconciles | Hand waving "we queue it" with no durability story | Outbox, conflict policy, idempotency, convergence |
| Brownfield diagnosis ("cold start regressed") | A measurement plan, then a fix with a rollback | Guessing a cause before asking for data | Instrumentation, bisection, budget enforcement |

**The asymmetry that defines this round.** A backend design round rewards you for scaling up. A mobile design round rewards you for degrading down. The client is the only node in the system with a human watching it, running on hardware you did not choose, on a network you cannot see, inside a process the operating system may terminate without asking.

**What the rubric usually contains.** Interviewers vary, but the recurring columns are close to these seven.

| Rubric column | A weak answer looks like | A strong answer looks like |
|---|---|---|
| Scoping | Starts drawing at minute two | States the surface and the boundary before any box |
| Client architecture | Names a pattern acronym and stops | Names where the pattern stops scaling and what replaces it |
| Data and state flow | One arrow labelled "API" | A source of truth, a cache policy, and an invalidation rule |
| Failure and offline | "We show an error" | Named states, retry policy, and what the user sees for each |
| Performance budget | "It should be fast" | A number per phase that sums to a stated target |
| Platform specifics | Vendor neutral to the point of vagueness | The real primitive on at least one platform, named |
| Communication and drive | Waits to be asked | Runs the clock, checks in, states what was skipped |

### 1c. Why some interviewers run a backend round under a mobile title

This is common enough that having a script for it is worth more than another architecture pattern.

| Cause | What you observe | What it means for you |
|---|---|---|
| Shared question bank across a whole engineering org | The prompt is a product name with server-shaped follow-ups | The interviewer may be happy either way, so you may choose |
| The interviewer is a backend or infrastructure engineer covering a mobile loop | Follow-ups about replicas, sharding, consistency | They will score what they can evaluate, so give them a contract and a client |
| The role really is mobile plus services | The job description mentions API ownership | Do not fight it, split the time and say you are splitting it |
| The prompt is genuinely ambiguous | "Design a photo sharing app" | It is your job to disambiguate, not theirs |

**The steering move, in three escalating steps.** Use the smallest one that works.

1. The boundary sentence, said once, in the first two minutes. "I will treat the server as one box with a contract on it, and spend most of the time on the device: local storage, sync, rendering and failure. Tell me now if you would rather I go the other way."
2. The split offer, if the follow-ups keep pulling server side. "I can give you ten minutes on how the feed service fans out, or ten minutes on how the client stays correct when that service is slow. Which is more useful to you?"
3. The dual-track answer, if steering fails twice. Answer the server question competently in two minutes, then attach a client consequence to it. "If the fan-out is asynchronous then the client can read its own write before the server has it, so I need a local echo and a reconciliation rule. Here is that rule."

!!! tip "The sentence that fails"

    "I am a mobile engineer, so I do not really do backend." This reads as a boundary on your competence rather than on the question. Redirect the question, never your own scope.

```
+----------------------------+      +---------------------------+
| DEVICE  33 of 38 minutes   |      | SERVER  5 of 38 minutes   |
| screens and frame budget   |      | one box with a contract   |
| local store and caches     | <==> | shape of the response     |
| outbox and sync            |      | version and compatibility |
| permissions and secrets    |      | push service integration  |
+----------------------------+      +---------------------------+

Figure how a 38 working minute mobile round splits across the boundary
```

### 1d. Worked example: the round opens with "Design a photo sharing app"

Forty five minutes, so 38 working minutes after greetings and closing questions. The prompt is one product noun and nothing else.

**Block 1. PURPOSE: turn a product name into a surface small enough to design.**

I do not ask "what features". That invites a list I cannot fit in 38 minutes. I ask which single user journey we are designing, and I offer three, because offering choices is faster than open questions.

"This product is at least four systems. I can design the feed client, the capture and upload pipeline, or the direct messaging client. Which one do you want?" I get: the feed client, including the images.

**The error most readers make here** is accepting the whole product and then triaging silently. Triage that happens in your head is invisible, and invisible triage scores as omission.

!!! note "Say it before you read on"

    Say why offering three named surfaces beats asking "which features matter most". One of them costs the interviewer effort and one does not.

**Block 2. PURPOSE: put the boundary on the record before I draw anything.**

I say the boundary sentence from 1c, then I make it concrete: "The server will be one box that returns a page of feed items and signed image URLs. Everything else I say will be on the device."

The interviewer nods. That nod is worth minutes: it is now agreed that I am not evasive about the server, I am scoped.

**Block 3. PURPOSE: absorb the first server pull without losing the boundary.**

At minute nine the interviewer asks how the feed is ranked. I do not refuse and I do not spend five minutes there.

"Ranking is server side. Assume a scored list per user, recomputed on write plus a request-time blend. What that costs me on the client is that the list is not stable between requests, so my pagination cannot be offset based and my cache key has to include a ranking session. Can I take that consequence and move on?"

Two sentences of server, one sentence of client consequence, and a request to move. This is the dual-track answer used early, before it is needed.

**The error most readers make here** is either stonewalling ("that is backend") or disappearing into ranking for six minutes. Both cost the same score, for opposite reasons.

!!! note "Say it before you read on"

    Name the one client design decision that the answer "ranking is recomputed at request time" just forced. If you cannot, reread the last paragraph before continuing.

**Block 4. PURPOSE: pre-commit the two areas that will carry the depth score.**

With 38 minutes I get one broad pass and two deep areas, not five. I choose the image pipeline and pagination stability, and I say so out loud at minute twelve: "I am going to go deep on the image memory and cache tiers, and on pagination under a mutating list. I am deliberately not covering direct messages, stories, or the capture pipeline."

**The error most readers make here** is choosing deep dives implicitly by running out of time in whichever area they happened to reach at minute thirty. Choosing out loud converts a time-out into a decision.

**Result.** By minute twelve the round has a named surface, an agreed boundary, one absorbed server question, and two declared deep dives. Nothing has been drawn yet, and that is fine: everything drawn after this point is defensible.

### 1e. Fade the scaffold

**Faded example 1.** The prompt is "Design a collaborative document editor" in a 45 minute mobile round. Blocks 1 to 3 are given. You produce block 4.

- Block 1: I offer three surfaces (the editor screen, the document list and offline availability, the presence and collaboration layer). I am given the editor plus offline.
- Block 2: I put the boundary on the record. The server is one box exposing document snapshots and an operation stream.
- Block 3: at minute eight the interviewer asks how conflicts are resolved server side. I give two sentences on server-side merge, then the client consequence: local edits must be representable as operations with stable identifiers so that a rebase is possible.
- Block 4: **you pre-commit the two deep areas and say what you are skipping.**

??? note "Show answer"

    A defensible block 4: "I am going deep on the local document model and the outbox that survives a force quit, and on how the editor stays at frame budget while an operation stream is applying remote edits. I am deliberately skipping the comment system, printing and export, and the sharing permission model."

    Why those two: this prompt's distinctive risk is that the client is a writer, not a reader. A feed client can drop a write and lose nothing. An editor that drops a write loses the user's work, which is the one failure users never forgive. The second dive is chosen because remote operations arrive on the same main thread that is servicing typing, which is a genuine frame budget conflict rather than a generic performance mention.

    The most common weak answer here picks "sync conflict resolution" and "offline storage" as the two dives. Those are the same dive twice, and the round ends with nothing said about rendering at all.

**Faded example 2.** The prompt is "Design a ride hailing app" in a 60 minute round with an interviewer from a backend platform team. Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: I offer the rider trip screen, the driver client, or the map and location pipeline. I am given the rider trip screen.
- Block 2: boundary stated. The server is one box exposing trip state transitions and a driver location stream.

??? note "Show answer"

    Block 3: the server pull will come, and from a platform interviewer it will be about the dispatch matching algorithm. The absorb-and-return move: "Matching is a server-side assignment problem over nearby drivers, and for the client the only thing that matters is that assignment is not instantaneous and can be revoked. That means my client is a state machine with a pending state that can go backwards, not a linear progress bar. Here is that state machine."

    Note what happened: a backend question was answered in one sentence and then converted into the single most important client artefact in this prompt.

    Block 4: with 52 working minutes I can afford three declared areas rather than two. I choose the trip state machine including revocation and race conditions, the location stream consumption at a battery budget, and degradation when the network drops mid trip. I skip payment, ratings, and the driver client entirely, and I say so.

### 1f. Check yourself

**Q1.** Ten minutes into a mobile round, the interviewer has asked three consecutive questions about database replication. You have already used the boundary sentence once. What do you do, and what do you not do?

??? note "Show answer"

    Do: escalate to step two, the split offer, and make it a real choice with a time attached. "I can spend ten minutes on the replication story or ten on how the client behaves when a replica is stale and it reads its own write back as missing. Which do you want?"

    Also do: if they choose replication, answer it properly and keep attaching client consequences, because a stale read is a client-visible bug and you can own that framing.

    Do not: repeat the boundary sentence a second time. Repeating it turns a scoping move into a refusal, and refusal is the one behaviour in this situation that reliably costs the round.

**Q2.** You are given the prompt "Design an analytics SDK". Name two things you would design that you would not design for a product surface prompt, and one thing that becomes far more important.

??? note "Show answer"

    Two things unique to the component shape: a public API surface with an explicit threading contract (which methods may be called from any thread, which are safe during application startup), and an initialisation cost budget, because a hundred teams depending on you means your startup work lands on every one of their cold starts.

    What becomes far more important: version and binary compatibility. An app surface can be rewritten in one release. A library with a hundred consumers cannot break a signature without a coordinated migration, so the error model and the extension points have to be right on the first release.

### 1g. When not to use this

Round triage and boundary setting are cheap, but they are not free and they can be over-applied.

| Question | Answer |
|---|---|
| The measurement that justifies doing this | The prompt names a product rather than a surface, or the first two follow-ups are server shaped |
| Scale threshold below which it is over-engineering | A prompt that already names one screen and one behaviour ("design the offline inbox"). Restating the boundary there wastes 60 seconds and reads as stalling |
| The cheaper alternative | One sentence: "I will assume the server is a given contract unless you want otherwise." Then start |
| Failure mode of over-applying | Spending five of 38 minutes negotiating scope. Scoping is worth two to three minutes; past that it is avoidance |

---

## Module 2. Six axes that make mobile different

**Time: 25 to 35 minutes reading, 15 to 20 minutes practice.**

### 2a. Predict first

Here is a plan written by a competent backend engineer for a new feature: a live vote count on a video screen.

```
1 client opens a websocket on screen entry
2 server pushes every vote increment as it happens
3 client updates the number on each message
4 on error client reconnects immediately
5 config for the feature is read from a server flag at app launch
6 rollback plan is revert the server deploy

Figure a server shaped plan for a client feature
```

??? note "Show answer"

    Prediction asked for: name the four lines that are wrong specifically because this runs on a phone, and say which axis each one violates.

    | Line | What breaks | Axis |
    |---|---|---|
    | 2 and 3 | A popular video can produce hundreds of increments per second. Each one is a main thread update, so at 120 hertz you have about 6 milliseconds of app budget per frame and you will miss it | Interaction model |
    | 1 and 4 | A socket held open on a screen keeps the radio out of its low power state, and immediate reconnect on a flaky link becomes a reconnect storm on the server and a battery hole on the device | Runtime and network |
    | 5 | The flag is read once at launch, so a process that stays alive for days never picks up the kill switch, and a process that is killed and restarted may pick it up at a random moment | Runtime |
    | 6 | Reverting the server deploy does not remove the client code that opens the socket. The client half of this feature ships on a release cycle you do not control | Deployment latency |

    Line 5 is the one most readers miss. It looks like configuration hygiene. It is actually the difference between a two minute incident and a thirty day one.

### 2b. The six axes

Every difference that matters in a mobile design round reduces to one of six. Naming the axis out loud in the round is worth more than reciting a platform detail, because it tells the interviewer you have a model rather than a list.

| Axis | The constraint in one line | What it forces into your design |
|---|---|---|
| Runtime | Your process is a guest. It can be suspended, frozen or terminated at any moment, without a callback you can rely on | Durability of in-flight work, state restoration, and no assumption that anything is still running |
| Network | Bandwidth, latency and reachability all change while a single request is in flight | Timeouts as budgets, retry with jitter, resumability, and a defined behaviour for every failure |
| Deployment latency | Server fixes ship in minutes. Client fixes ship in weeks, to a population that partially never upgrades | Remote configuration, kill switches, and forward compatible contracts |
| Interaction model | A human is watching a specific pixel and the frame deadline is single digit milliseconds | Main thread discipline, budgets per phase, and cancellation as a first class operation |
| Ecosystem fragmentation | The same binary runs on hardware separated by roughly an order of magnitude in every resource | Device tiers, adaptive behaviour, and testing on the worst tier you support |
| Privacy posture | Data on the device is subject to user granted permissions, store policy and local law, and can be revoked at any time | Permission denial as a normal path, on-device processing where possible, and a working degraded mode |

### 2c. Deployment latency, derived

This is the axis that most changes how a senior candidate answers, so it is worth doing the arithmetic once and carrying the result.

Assume the following, each labelled and each conservative for a large consumer app.

| Input | Assumed value | Why this is reasonable |
|---|---|---|
| Time to build, test and submit a hotfix | 1 day | A release branch, a build, and a regression pass compress to about a day when a team is trying |
| Store review before availability | 1 day | Reviews are typically same day to two days for an update to an existing app |
| Staged rollout to full availability | 5 days | Teams ramp 1, 10, 50 then 100 percent with soak time between steps |
| Users on the newest version after 7 days | 60 percent | Auto update is common but not instant, and devices update on charge and wifi |
| Users on the newest version after 30 days | 90 percent | The curve flattens after the first fortnight |
| Users who never take the update | 3 to 5 percent | Old operating systems, storage pressure, disabled auto update |

Now the derivation.

- Time until the fix exists on any device: 1 plus 1 plus 5 equals **7 days**.
- Time until the fix reaches 90 percent of users: **about 30 days** from the moment the bug was found.
- A server side fix, from decision to full traffic behind an existing flag: **minutes**. Take 10 minutes as the working number.
- Ratio of client fix reach time to server fix time: 30 days is 43,200 minutes, divided by 10 equals **about 4,300 times slower**.
- And the tail never closes. At 3 percent permanently stale and 10 million installs, **300,000 devices** run the broken code forever.

**The design consequence, stated as a rule.** Any behaviour that you might need to change in under a week must be controllable from the server. That is what remote configuration is for, and it is why a kill switch is a design element rather than an operational nicety.

```
BUG FOUND
   |
   +--> server side path
   |      flag flip  -->  full traffic in 10 minutes
   |
   +--> client side path
          build 1 day > review 1 day > rollout 5 days
             > 60 percent at 7 days > 90 percent at 30 days
             > 3 percent never

Figure the two repair paths available after a bug is found
```

### 2d. Worked example: applying the six axes to one feature

The feature: show a live vote count on the video screen, the same feature from 2a. I will pass it through each axis and let each one change the design.

**Block 1. PURPOSE: pick the transport by asking what the user can actually perceive, not by asking what is real time.**

A human cannot read a number that changes hundreds of times a second. I decide the user needs the count to feel live, which I define as updated at most once per second and correct within a few seconds. That single definition removes the per-event stream.

I choose a server push that carries no payload plus a poll, or a stream that the server already coalesces to one message per second per client. I will take the coalesced stream.

**The error most readers make here** is treating "real time" as a requirement rather than as a word the product manager used. Ask what interval is perceptible and the whole transport question usually collapses.

!!! note "Say it before you read on"

    Say which of the six axes forced the coalescing decision, and which second axis it also happens to fix for free.

**Block 2. PURPOSE: make the runtime axis explicit by writing down what happens when the process is not running.**

Three states, three behaviours.

| Process state | What the stream does | What the user sees |
|---|---|---|
| Foreground, screen visible | Stream open, one update per second | A live number |
| Backgrounded, process alive | Stream closed within a few seconds | Number frozen at last value |
| Terminated by the operating system | Nothing. There is no code of mine running | On relaunch, fetch once, then reopen |

The important line is the third. I do not get a reliable callback before termination, so anything I needed to persist must already be persisted. For a read only counter that is trivial. I note it because in module 9 the same axis is what makes an upload queue hard.

**Block 3. PURPOSE: bound the render cost so the interaction axis cannot be violated later.**

One update per second, one text node changed, on a screen that is also decoding video. The frame budget at 120 hertz is 1000 divided by 120, which is 8.3 milliseconds per frame, and I will reserve a quarter of it for the system compositor and other work, leaving about 6 milliseconds for my code.

A single number update is far inside that. But I add one rule: the update must not trigger a layout pass on the whole screen, because a full layout on a video screen with overlays is the thing that will cost milliseconds, not the text change itself.

**The error most readers make here** is optimising the network and never asking what the update costs on the main thread. The frame budget is the constraint the user actually feels.

!!! note "Say it before you read on"

    Before block 4, say what you would put in the design so that this feature can be turned off in ten minutes rather than thirty days.

**Block 4. PURPOSE: buy back the deployment latency I do not have.**

Three server controlled values, read at launch and refreshed on foreground, with safe defaults compiled in.

| Control | Values | What it buys |
|---|---|---|
| Feature enabled | on, off | The ten minute kill switch derived in 2c |
| Update interval | 1 to 30 seconds | Back off under server load without a client release |
| Transport | stream, poll | Fall back to polling if the stream tier is unhealthy |

Defaults compiled into the binary are off, 5 seconds, and poll. If configuration cannot be fetched at all, the app is in its safest state rather than its most expensive one.

**The error most readers make here** is a kill switch that is only read at cold start. A process can stay alive for days, so the flag must also be re-read on foreground, and the feature must check it at the point of use rather than only at initialisation.

**Result.** The feature now has a perceptual budget rather than a real time claim, defined behaviour in all three process states, a frame budget rule, and a ten minute off switch. Four of the six axes appear explicitly. Fragmentation and privacy did not bind here, and saying that out loud is itself a scoring move.

### 2e. Fade the scaffold

**Faded example 1.** The feature is "show the user's step count from the health sensor on the home screen, updated as they walk". Blocks 1 to 3 are given. Produce block 4.

- Block 1: perceptible interval is coarse. A step count that updates every 10 to 30 seconds is indistinguishable from one that updates continuously, so I read a batched aggregate rather than subscribing per step.
- Block 2: runtime. The process is usually not running while the user walks, so the sensor data must be recorded by the operating system's own store and read on foreground, not accumulated in my memory.
- Block 3: interaction. One text update per 10 seconds is free. No frame budget concern.
- Block 4: **you handle the remaining two axes.**

??? note "Show answer"

    The two remaining axes are privacy and fragmentation, and both bind hard here.

    Privacy: health data requires an explicit user grant that can be revoked at any time, including while the app is backgrounded. So permission denied is a normal state, not an error state. The home screen needs a designed appearance for granted, denied, and not yet asked, and the not yet asked state must never be a system prompt fired on first launch, because a prompt with no context is denied and denial is often permanent. Ask in context, after the user taps the step card.

    Fragmentation: not every device has the sensor, and some report it through a coprocessor with different latency and different history depth. So the design needs a capability check before the permission request, and a home screen layout that omits the card entirely when the capability is absent rather than showing an empty one.

    The most common weak answer treats permission denial as an error toast. A toast is what you show for a transient failure. A revoked permission is a stable state of the world and needs a stable presentation.

**Faded example 2.** The feature is "let the user reply to a message directly from the notification". Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: there is no perceptible interval question. The interaction is a single user initiated send.
- Block 2: runtime. The reply is handled in a short lived, restricted execution context that the operating system grants for a limited window, and the main app process may not be running at all.

??? note "Show answer"

    Block 3, the interaction and network axes: the execution window is short and not negotiable, so the reply cannot depend on a full application startup, a dependency graph build, or a session refresh round trip. The correct shape is: write the message into the durable outbox with an idempotency key, hand it to a background transfer mechanism owned by the operating system, and return immediately. The user sees the reply as sent because it is durably recorded, not because the server acknowledged it.

    Block 4, deployment and fragmentation: the notification reply surface differs between platforms in what it allows, and older operating system versions may not support it at all. So the server must not assume the capability exists. The push payload carries the data the reply action needs, the client advertises its capability at registration time, and the whole action sits behind a remote flag because a bug in a code path that runs outside your normal process is exactly the kind of bug you will want off in ten minutes.

### 2f. Check yourself

**Q1.** Using the assumptions in 2c, a bug is found on day zero that corrupts a local cache. The team ships a client fix immediately. On day 10, roughly what fraction of the installed base is still running the broken code, and what does that imply about the server side of the fix?

??? note "Show answer"

    From 2c: the fix is not on any device until day 7 (1 build, 1 review, 5 rollout). Adoption reaches 60 percent at 7 days after availability, so on day 10 only 3 days of adoption have elapsed. Interpolating the stated curve, well under half the base has it, so roughly 60 to 75 percent of installs are still broken on day 10.

    The implication: the client fix is necessary but it is not the incident response. The incident response has to be server side, which means the server must be able to make the broken code path harmless. Concretely, that is a flag that stops the client writing to that cache, or a response field that instructs clients to purge and bypass it. If neither exists in the currently shipped binary, the correct lesson is that the previous release should have shipped one.

**Q2.** Which single axis explains why an "immediate reconnect on socket error" loop is worse on mobile than on a desktop web client? Name the two distinct costs.

??? note "Show answer"

    The network axis, because reachability changes underneath a live connection rather than being binary and stable.

    Cost one, on the device: each reconnect attempt wakes the radio and holds it in a high power state, so a tight loop on a link that is down converts a connectivity problem into a battery problem within minutes.

    Cost two, on the server: when a shared network segment recovers, every client that was looping reconnects in the same instant. That is a thundering herd, and it arrives at the one tier least able to absorb it, because connection setup is more expensive than a request. The fix for both is the same: exponential backoff with full jitter, plus a cap, plus not reconnecting at all while the operating system reports no route.

### 2g. When not to use this

The six axis pass is a framing tool. It is a checklist, and checklists get recited.

| Question | Answer |
|---|---|
| The measurement that justifies a full six axis pass | The feature crosses a process boundary, holds a connection, writes user data, or requires a permission. If it does none of those, it does not need the pass |
| Scale threshold below which it is over-engineering | A purely presentational change (a new layout for existing data) touches one axis at most. Running all six on it burns three minutes for nothing |
| The cheaper alternative | Ask two questions: what happens if the process dies right now, and how do I turn this off in ten minutes. Those two cover the majority of the value |
| Failure mode of over-applying | Reciting six headings per component in an interview. Name the axis only where it changes the design, and say explicitly which axes did not bind |

---

## Module 3. Non-functional requirements with budgets

**Time: 35 to 45 minutes reading, 20 to 30 minutes practice.**

### 3a. Predict first

Here is a non-functional requirements block a candidate wrote in a real-shaped round.

```
NON FUNCTIONAL REQUIREMENTS
  app should start fast
  scrolling should be smooth
  should not use too much memory
  should work on low end devices
  should be battery efficient
  should work offline

Figure a requirements block with no design consequence
```

??? note "Show answer"

    Prediction asked for: how many design decisions can be made from this block, and what is the minimum edit that makes it useful?

    Zero decisions can be made from it. Every line is a direction, not a threshold, and a direction cannot rule anything out. "Fast" does not tell you whether you may build the dependency graph eagerly at launch. "Smooth" does not tell you whether a 4 millisecond main thread operation is acceptable.

    The minimum edit is a number and a condition per line: fast becomes "time to first content under 1.5 seconds on a mid tier device from a cold process", smooth becomes "no frame over 8 milliseconds of application work at 120 hertz", memory becomes "resident set under 180 megabytes on the feed screen".

    The tell that a candidate has done this before is not the specific number. It is that each number is attached to a device tier and a screen, because a number without a tier is not measurable.

### 3b. Tiers, cold start and the frame budget

#### Device tiers, because a number without a tier is meaningless

Every budget in this module is per tier. Define the tiers first, in the round, in about twenty seconds.

| Tier | Assumed hardware | Assumed share of a global consumer base |
|---|---|---|
| Low | 2 to 3 gigabytes of memory, no high performance core, slow storage | 30 percent |
| Mid | 4 to 6 gigabytes, two performance cores, decent storage | 45 percent |
| High | 8 gigabytes or more, several performance cores, fast storage | 25 percent |

!!! note "These shares are an assumption, and here is why they are reasonable"

    A global consumer app sees a long tail of older and cheaper hardware, so the low tier cannot be a rounding error. A business or developer facing app skews high and the shares invert. The number you must defend in an interview is not 30 percent, it is that you asked which distribution you are designing for and then set budgets against the worst tier you promised to support.

**The rule that follows.** Budgets are set at the tier boundary you commit to, not at the average. If you support the low tier, the low tier number is the requirement and the high tier number is a bonus. Averages hide the users who churn.

#### Cold start, decomposed until it sums

Two terms, defined before use.

| Term | Definition |
|---|---|
| Cold start | The process does not exist. The operating system creates it, the runtime initialises, then your code runs |
| Warm start | The process exists but the screen must be recreated |
| Time to initial display | From launch intent to the first frame with your own pixels on it, even if the content is placeholders |
| Time to full display | From launch intent to the first frame where the primary content is real and usable |

**Where the targets come from.** Use the classic response time thresholds from Nielsen, *Usability Engineering* (Academic Press, 1993): about 0.1 seconds feels instantaneous, about 1 second keeps the user in flow, and about 10 seconds loses their attention entirely. Those are perception facts, not platform facts, so they are a legitimate basis for a target.

| Tier | Time to initial display target | Time to full display target | Reasoning |
|---|---|---|---|
| High | 0.6 seconds | 1.0 second | Stay inside the flow threshold end to end |
| Mid | 0.9 seconds | 1.5 seconds | First pixels inside flow, content just outside it |
| Low | 1.4 seconds | 2.5 seconds | Accept a visible wait, stay far from the attention threshold |

**Now decompose the mid tier budget so it sums.** A target you cannot attribute is a target you cannot defend when it regresses.

| Phase | Budget in milliseconds | What lives here |
|---|---|---|
| Process creation and runtime init | 200 | Operating system work plus runtime startup. Mostly not yours |
| Library and dependency graph init | 250 | Every library that initialises eagerly at launch |
| First screen construction and layout | 300 | View or scene creation, first layout pass |
| First frame draw | 150 | Rasterise and present |
| **Time to initial display subtotal** | **900** | 200 plus 250 plus 300 plus 150 |
| Read cached content from disk | 200 | The reason a disk cache exists |
| Network fetch, only on cache miss | 400 | Budgeted, not guaranteed |
| **Time to full display total** | **1500** | 900 plus 200 plus 400 |

**What this table buys you in the round.** Three separate arguments, all derivable from it.

- The 250 millisecond library budget is the whole reason lazy initialisation is an architecture decision rather than a micro-optimisation. Ten libraries at 25 milliseconds each consumes it entirely.
- The 200 millisecond disk read is why the design must render from cache first and reconcile with the network afterwards. If you wait for the network you have spent 400 of your 600 remaining milliseconds on a resource you do not control.
- On the low tier, assume every phase is roughly 1.6 times slower (2.5 divided by 1.5 from the target table). 900 milliseconds of initial display becomes about 1,440, which is exactly the low tier target, with no slack. That is how you show the interviewer that the low tier is not an afterthought.

#### The frame budget, and why 60 hertz thinking is now wrong

The refresh rate sets a hard deadline. Miss it and the previous frame is shown again, which the user perceives as a stutter.

| Refresh rate | Frame period | Reserve 25 percent for system and compositor | Application budget |
|---|---|---|---|
| 60 hertz | 1000 divided by 60 equals 16.7 milliseconds | 4.2 | **12.5 milliseconds** |
| 90 hertz | 1000 divided by 90 equals 11.1 milliseconds | 2.8 | **8.3 milliseconds** |
| 120 hertz | 1000 divided by 120 equals 8.3 milliseconds | 2.1 | **6.2 milliseconds** |

!!! note "The 25 percent reserve is an assumption, and here is why it is reasonable"

    Your application is not the only thing that must complete inside the frame period. The compositor, the input pipeline and other processes all need cycles, and the scheduler does not hand you the core for free. A quarter is a working reserve that keeps you honest. Measure your own and replace it. What must not happen is designing against the full period as though it were yours.

**The consequence, derived.** Take a single main thread operation that costs 8 milliseconds, for example parsing a moderate JSON payload on the main thread.

- At 60 hertz it consumes 8 divided by 12.5 equals 64 percent of the budget. Tight but survivable.
- At 120 hertz it consumes 8 divided by 6.2 equals 129 percent of the budget. Every frame it runs in is dropped.

So the same code is acceptable on a 2018 device and janky on a 2024 flagship. This inverts the usual intuition that newer hardware forgives more, and it is worth saying out loud in a round.

**Jank arithmetic.** A 100 millisecond main thread block at 120 hertz drops 100 divided by 8.3, which is about 12 consecutive frames. Users reliably perceive anything above roughly two dropped frames during a scroll or animation, so a single 100 millisecond block is not a small problem.

```
120 HERTZ FRAME  8300 microseconds per frame

+-------------------------------------+--------+
| application work  6200 microseconds | system |
+-------------------------------------+--------+
0                                  6200     8300

one main thread parse of 8000 microseconds
+------------------------------------------+
| overruns the budget so the frame is lost |
+------------------------------------------+

Figure the application share of a 120 hertz frame and one overrun
```

### 3c. Memory, bytes per screen and battery

#### Memory, and the kill you cannot catch

Two numbers matter and they are different.

| Number | What it is | Why you care |
|---|---|---|
| Heap or allocation limit | A per process cap enforced by the runtime or the operating system | Exceeding it terminates you immediately, usually with a crash report |
| Resident footprint | Total physical memory attributed to your process | Determines your rank in the queue when the system needs memory back |

**Bitmap arithmetic, because images dominate.** A decoded bitmap costs width times height times bytes per pixel. At 4 bytes per pixel (8 bits each for red, green, blue and alpha):

- A full screen image at 1080 by 2400 pixels: 1080 times 2400 times 4 equals 10,368,000 bytes, about **10.4 megabytes**.
- A 12 megapixel camera photo at 4000 by 3000: 4000 times 3000 times 4 equals **48 megabytes** decoded, from a source file that might be 3 megabytes on disk.
- Assume a 192 megabyte heap cap on a mid tier device. That is 192 divided by 10.4, about **18 full screen bitmaps**, and exactly **4 camera photos** before you are dead.

The gap between a 3 megabyte file and a 48 megabyte decode is the single most important number in module 8, and it is why "the image is small" is never an answer.

**Why footprint matters even when you never crash.** When memory is scarce the system reclaims it by terminating background processes, and larger processes are reclaimed first. So an app with a large footprint is killed while backgrounded and pays a full cold start on return.

Derive the user visible cost with the module 3c numbers: a warm resume renders in roughly 200 milliseconds (screen recreation only), a cold start costs 1,500 milliseconds on mid tier. That is a **7.5 times** penalty, and the user attributes it to the app being slow, not to memory.

**A defensible budget block for a feed screen, mid tier.**

| Component | Budget | Derivation |
|---|---|---|
| Application code, runtime, libraries | 60 megabytes | Assumption, measured once and then held |
| Image memory cache | 24 megabytes | One eighth of a 192 megabyte heap, a conventional and defensible split |
| Visible bitmaps not in cache | 30 megabytes | About three full screen images in flight |
| Everything else (view hierarchy, database pages, network buffers) | 40 megabytes | Assumption |
| **Total** | **154 megabytes** | Leaves about 38 megabytes of headroom under the 192 cap |

#### Bytes per screen, and the battery cost of a chatty client

**Bytes per screen, derived for a feed.** Assume a feed screen shows 8 cards, each with one square image, on a device with a 1080 pixel wide display.

| Choice | Pixels per image | Bytes at 0.17 bytes per pixel | 8 cards plus 50 kilobytes of JSON | Time at 1 megabit per second |
|---|---|---|---|---|
| Serve the full resolution 1080 by 1080 | 1,166,400 | about 198 kilobytes | about 1.63 megabytes | about 13 seconds |
| Serve 540 by 540 and upscale | 291,600 | about 50 kilobytes | about 450 kilobytes | about 3.6 seconds |
| Serve 540 by 540 plus a 32 by 32 preview first | as above plus 1 kilobyte each | about 51 kilobytes | about 458 kilobytes | preview in under 0.5 seconds |

!!! note "The 0.17 bytes per pixel figure is an assumption, and here is why it is reasonable"

    Photographic content encoded at a mid quality setting in a modern lossy format commonly lands between 0.1 and 0.25 bytes per pixel. Take the middle. What matters for the design is the ratio: halving each dimension quarters the pixel count and therefore roughly quarters the bytes. That ratio is arithmetic, not an assumption.

The elimination this performs: 13 seconds to fill a screen is not a slow experience, it is an abandoned session. So serving one image size for all surfaces is ruled out, and the design needs server side variants keyed by the rendered size. That is a requirement, derived, in four lines.

**Battery, taught as a mechanism rather than an invented milliwatt figure.** The dominant client controllable cost is the radio, because of tail energy: after a transmission the radio stays in a high power state for a few seconds before stepping down, in case more data arrives.

Assume a 5 second tail and 1 second of active transmission per request. This is a modelling assumption; the exact tail is network and generation dependent, but the existence of a multi-second tail is the mechanism that matters.

- Ten requests spaced 20 seconds apart: each pays 1 second active plus 5 seconds tail, so 10 times 6 equals **60 radio-seconds**.
- The same ten requests coalesced into one burst: 2 seconds active plus 5 seconds tail equals **7 radio-seconds**.
- Ratio: **about 8.6 times** less radio time for identical payload.

That single derivation justifies request coalescing, batched telemetry, and deferring non-urgent work to a moment when the radio is already awake. It is also why a background poll every 60 seconds is one of the most expensive things a mobile app can do.

**Relative subsystem cost, ordered rather than numbered.** Continuous location at high accuracy and continuous camera or video encoding sit at the top, sustained radio use next, then screen at high brightness, then general processing. Any absolute number here would be device specific and would age badly, so what you state in a round is the ordering plus the measurement you would run: an on-device power profile with the feature on and off, on the same device, at the same brightness.

### 3d. Worked example: writing the budget block for a prompt in three minutes

The prompt: "Design the client for a short video feed." I have about three minutes of my 38 for non-functional requirements, and I want every line to eliminate something.

**Block 1. PURPOSE: commit to a tier before any number, so the numbers are measurable.**

I say: "I will design for the mid tier as the default and treat the low tier as a supported degradation, assuming a global consumer distribution. If this is a market where the low tier is the majority, tell me and I will move the budgets."

**The error most readers make here** is quoting a single number with no tier attached. It is not wrong, it is unfalsifiable, and unfalsifiable numbers score as decoration.

!!! note "Say it before you read on"

    Say why committing to a tier is a requirements decision rather than a testing decision.

**Block 2. PURPOSE: set the three budgets that will actually constrain the architecture.**

| Budget | Value | What it eliminates |
|---|---|---|
| Time to first video frame, cold start, mid tier | 1.5 seconds | Rules out building the full dependency graph before the player exists, and rules out waiting for the feed API before showing anything |
| Application work per frame at 120 hertz | 6 milliseconds | Rules out decoding thumbnails on the main thread and rules out any synchronous disk read during scroll |
| Resident footprint on the feed screen, mid tier | 180 megabytes | Rules out holding more than a small window of decoded video buffers, which in turn decides the prefetch depth |

Each of these is one of the derivations above, restated for this prompt. Nothing new was invented.

**Block 3. PURPOSE: convert the byte budget into a product decision the interviewer can push back on.**

Using the byte arithmetic in 3c: at 1 megabit per second, 1.5 seconds of budget buys about 190 kilobytes. So the first video segment must be under roughly 150 kilobytes after protocol overhead, which forces a low bitrate initial segment and an adaptive ladder that steps up after playback starts.

I say the consequence out loud: "This means the first two seconds of video will visibly be lower quality on a slow connection, and I am choosing that over a spinner. That is a product trade-off and I would want it confirmed."

**The error most readers make here** is presenting a technical budget without surfacing the product consequence. The budget is only interesting because it forces someone to choose between a blurry start and a blank screen.

!!! note "Say it before you read on"

    Say what measurement would tell you that this trade-off was the wrong one.

**Block 4. PURPOSE: name the battery budget in a way that can be checked.**

Continuous video decode plus sustained network is close to the worst combination a phone can run, so I do not promise a number I cannot derive. I promise a method: "I would set the budget as a percentage of battery per hour of active viewing, measured on a fixed device at fixed brightness, and treat a regression above a set threshold as release blocking. I would also add a low power mode behaviour: drop the prefetch depth from three items to one and cap the bitrate ladder."

**The error most readers make here** is inventing a milliwatt figure. Stating the measurement and the degradation policy is stronger than a number you cannot defend, and interviewers notice the difference.

**Result.** Four budgets, each attached to a tier, each ruling out at least one design option, and one of them explicitly handed back to the product as a trade-off. That block takes about three minutes to say and it makes every later decision in the round defensible.

### 3e. Fade the scaffold

**Faded example 1.** The prompt is "Design the client for a map application". Blocks 1 to 3 are given. Produce block 4.

- Block 1: I commit to the mid tier, and I note that maps are used outdoors where the network is worst, so I will use a 0.5 megabit per second assumption rather than 1.
- Block 2: budgets are time to first map tile under 1.2 seconds from a cached region, 6 milliseconds of application work per frame during a pan, and 200 megabytes resident with tiles included.
- Block 3: byte budget. At 0.5 megabits per second, 1.2 seconds buys about 75 kilobytes, which is roughly three vector tiles at an assumed 25 kilobytes each, so the first view must be served from the on-device tile cache and the network path is for the surrounding ring only.
- Block 4: **you write the battery budget and the degradation policy.**

??? note "Show answer"

    A defensible block 4: continuous location is the top of the subsystem ordering in 3c, so it is the budget that matters and it must be expressed as a policy rather than a number.

    "I would budget location updates by mode. While the user is actively panning, no continuous location is needed at all, only the last known fix."

    "While navigating, I need high accuracy continuous updates and I accept the cost, but I would tie the update interval to speed: at walking pace a fix every 3 to 5 seconds is indistinguishable from continuous, and it cuts the sensor duty cycle by several times against a once per second fix. In low power mode I drop to the coarser provider and stop prefetching the surrounding tile ring."

    The measurement: battery percentage per hour, on a fixed device, in three scenarios (idle map open, panning, navigating), compared release over release.

    The most common weak answer says "we will use location efficiently". That has the same information content as the block in 3a.

**Faded example 2.** The prompt is "Design a messaging client". Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: mid tier default, and I flag that messaging is used on every tier including the oldest devices, so the low tier is a first class target rather than a degradation.
- Block 2: time to conversation list under 1.0 seconds cold on mid tier, because the list must come entirely from local storage; 6 milliseconds per frame at 120 hertz while scrolling a conversation; 150 megabytes resident.

??? note "Show answer"

    Block 3, the byte and network budget with its product consequence: a conversation list rendered from local storage costs zero network bytes, which is the entire reason the 1.0 second target is achievable while the map prompt needed 1.2. The network budget is therefore not about first render at all, it is about the delta sync that follows. I would budget the foreground sync at under 50 kilobytes for a typical session, which forces a change token or cursor based delta rather than refetching conversation state.

    The product consequence to surface: messages sent while offline will appear as sent locally before the server has them, so the interface needs a visible per message state (pending, sent, delivered) and I need the product to agree that a pending message is shown in the timeline rather than hidden.

    Block 4, battery: the dominant cost is maintaining a connection for delivery. I would not hold a socket open in the background at all, and would instead rely on the platform push service to wake the app, using the notify then fetch pattern from module 10. Foreground holds a socket, background does not. The measurement is battery percentage per hour with the app backgrounded and idle, which should be indistinguishable from the app not being installed. Any measurable difference there is a bug, not a trade-off.

### 3f. Check yourself

**Q1.** Your feed screen currently does a 5 millisecond main thread JSON parse per card as cards come into view. The team ships a device with a 120 hertz display. Nothing in your code changed. Using the frame budget table in 3b, quantify what happened and name the fix that does not require making the parse faster.

??? note "Show answer"

    Before: at 60 hertz the application budget is 12.5 milliseconds, so a 5 millisecond parse consumes 40 percent. Frames were tight but not dropped.

    After: at 120 hertz the budget is 6.2 milliseconds, so the same 5 millisecond parse consumes 81 percent. Add any layout or draw work on the same frame and you exceed the period, so the frame is dropped. A user scrolling past 10 cards now sees up to 10 dropped frames.

    The fix that does not touch parse speed: move the parse off the main thread and make it happen before the card is visible, driven by the prefetch window rather than by the view becoming visible. The card then binds to an already parsed object, which is a few tens of microseconds. This is the same structural move as image prefetch in module 8: the cost did not shrink, it moved off the critical frame.

**Q2.** A reviewer says your 24 megabyte image cache budget is arbitrary. Defend it from stated inputs, then state the condition under which you would change it.

??? note "Show answer"

    Derivation from the memory section in 3c: assume a 192 megabyte heap cap on the target mid tier device, and allocate one eighth, giving 24 megabytes.

    At the feed's rendered size, a card image at 540 by 540 pixels and 4 bytes per pixel costs 540 times 540 times 4, which is about 1.17 megabytes. So 24 divided by 1.17 is about 20 cached bitmaps, and with 8 cards visible per screen that is about two and a half screens of scroll history. That is enough to make a short scroll back instant, which is the actual product goal.

    When I would change it: if measurement shows the cache hit rate on scroll back is below roughly 80 percent, the window is too small and I would raise it, provided the total footprint from 3c still fits under the cap with headroom. If background termination rate rises after raising it, that is the signal that footprint, not hit rate, is now the binding constraint, and I would lower it again and shift the burden to the disk cache.

### 3g. When not to use this

Budget blocks are the highest value three minutes in a mobile round, and they are still possible to over-apply.

| Question | Answer |
|---|---|
| The measurement that justifies a full budget block | The prompt has a rendering surface, a cold start, or a network dependency on the critical path. Almost every product surface prompt does |
| Scale threshold below which it is over-engineering | A component or library prompt (design an image loader) needs a resource budget and an initialisation cost, but does not need a cold start decomposition, because you do not own the host application's launch |
| The cheaper alternative | Three numbers with tiers attached: time to content, per frame application budget, resident footprint. That is 80 percent of the value in 45 seconds |
| Failure mode of over-applying | Producing a twelve row budget table and then never referring to it again. A budget earns its place only when you later say "I cannot do that, it costs 4 milliseconds and I have 6" |
| The second failure mode | Numbers with no tier and no derivation. These are worse than no numbers, because they invite a follow-up you cannot answer |

---

## Module 4. Client architecture

**Time: 35 to 45 minutes reading, 20 to 25 minutes practice.**

### 4a. Predict first

Here is the responsibility list of one screen class, taken from a real shaped codebase.

```
FeedScreen
  holds   the list of items currently shown
  does    fetch a page from the network
  does    parse the response
  does    decide which items are advertisements
  does    write items to the local database
  does    handle pull to refresh
  does    handle a like tap and post it
  does    format timestamps for display
  does    emit analytics events
  does    show error alerts

Figure one screen class holding ten responsibilities

```

??? note "Show answer"

    Prediction asked for: a second entry point is added, a deep link that opens the same feed filtered to one author. Which responsibility forces the first rewrite, and why is it not the network code?

    State ownership forces the rewrite. The list of items lives inside the screen object, so its lifetime is the screen's lifetime. The deep link creates a second screen instance, which means a second fetch, a second cache, and two copies of the same item that can disagree after a like.

    It is not the network code, because a fetch function is trivially callable from two places. The thing that does not survive duplication is mutable state with no owner outside the view.

    The second answer worth having: the like tap. It mutates state that two screens now display, so without a shared source of truth the user likes a post on the deep linked screen and sees it unliked when they go back. That bug is architectural, and no amount of care in the network layer prevents it.

### 4b. Layers, and the one rule that makes them real

Three layers, defined before use, with an explicit dependency direction.

| Layer | Owns | Must not contain | How you test it |
|---|---|---|---|
| Presentation | View hierarchy, gesture handling, and a state holder that exposes one state object per screen | Networking, database access, business rules | Render a state object, assert what is on screen |
| Domain | Use cases, entities, invariants, policy such as "an item is visible if" | Any platform type, any view type, any storage type | Plain unit tests with no platform runtime |
| Data | Repositories, remote and local sources, mappers between wire and domain shapes | Anything about how the data is displayed | Fake the sources, assert the repository's contract |

**The rule that makes layering more than folder names.** Dependencies point inward only. Presentation may depend on domain. Data may depend on domain. Domain depends on nothing. If the domain layer imports a view type or a network type, the layering is decorative.

**Why this matters more on a client than on a server.** A server process usually handles one request and exits the code path. A client holds live state across a user session that spans minutes, survives configuration changes, and must be restorable after the process is killed. State without an owner outside the view is the root cause of most client bugs.

#### The user interface pattern family, and where each one stops

Four points on a spectrum. Each is a legitimate answer at some size, and each has a specific breaking point. Naming the breaking point is what separates a senior answer from a pattern name.

| Pattern | The shape | Stops scaling when | The symptom you observe |
|---|---|---|---|
| Screen object does everything | View and logic in one class | More than about two asynchronous sources, or a second entry point to the same data | The bug in 4a: two instances of the same data disagreeing |
| Presenter drives a view interface | Presenter calls imperative methods on a view interface | The number of view methods grows with the number of states rather than the number of widgets | An interface with 20 methods such as showLoading, hideLoading, showEmpty, and a state you can reach where two are true at once |
| Observable state per field | The state holder exposes several independent observable values | Two observables can be inconsistent for a moment, and that moment is renderable | A frame where the spinner is hidden but the list is still empty |
| Single immutable state plus a reducer | One state object per screen, events in, state and effects out | The single object grows so large that any change re-renders everything | Frame budget misses during typing, because one keystroke rebuilds the whole screen |

**The rule that fixes the middle two.** The view is a function of one state value. If two pieces of state can disagree, make them one piece of state with a shape that cannot express the disagreement. A sealed set of states (Loading, Empty, Content with items, Error with a reason) makes "spinner visible and content visible" unrepresentable rather than merely unlikely.

**The fix for the fourth.** Do not abandon the single state. Slice it. The view subscribes to the derived slice it needs, and rerender is scoped to the slice that changed. This is a memoisation problem, not an architecture problem.

#### Unidirectional data flow

Three terms, defined before use.

| Term | Definition |
|---|---|
| State | An immutable value that fully describes what should be on screen right now |
| Event | Something that happened: a user action, or a result arriving |
| Effect | A side effect requested by the reducer, such as "start a fetch", executed outside the reducer |

The loop, and the reason effects are separated from the reducer: a reducer that performs a network call is not testable without a network, and a reducer that returns a description of a network call is testable with an equality assertion.

```
     +--------+   event    +----------+   effect   +---------+
     |  VIEW  | =========> | REDUCER  | =========> | EFFECTS |
     |        |            | pure     |            | network |
     |        | <========= | function |            | storage |
     +--------+   state    +----------+            +---------+
                                |                        |
                                +<=======================+
                                    result event

Figure the unidirectional loop with effects outside the pure reducer
```

#### Dependency injection, priced against the cold start budget

Dependency injection means an object receives its collaborators rather than constructing or locating them. Three ways to do it, priced against the 250 millisecond library and graph budget from module 3.

| Approach | How the graph is built | Cost at launch | When it is the right answer |
|---|---|---|---|
| Manual constructor injection | You write the wiring by hand | Near zero, and visible in a stack trace | Small apps, libraries, and any code with fewer than roughly 50 wired types |
| Compile time generated graph | A tool generates the wiring at build time | Near zero at launch, moved to build time | Large apps where the graph is big enough to be error prone by hand |
| Runtime reflective container | The container resolves types by inspection when asked | Proportional to graph size, paid at launch | Rarely correct on a client. Convenient, and it lands in your worst budget |

**The derivation that decides it.** Assume a reflective container resolves a binding in 0.2 milliseconds on a mid tier device. This is a modelling assumption; measure yours. With 400 bindings resolved eagerly at launch, that is 400 times 0.2 equals **80 milliseconds**, which is 32 percent of the entire 250 millisecond library budget from module 3, spent on wiring rather than on anything the user asked for.

Two consequences, both worth stating in a round.

- Prefer a graph resolved at build time, because the cost then lands on your build machine rather than on the user's cold start.
- Make everything lazy by default. A binding that is never used on the launch path should cost nothing at launch, whichever approach you chose.

#### Modularisation, and the threshold where it pays

Splitting an application into build modules buys three things: parallel and incremental builds, enforced dependency direction (a module simply cannot import what it does not declare), and ownership boundaries that survive team growth.

It costs configuration, a public and internal split for every type crossing a boundary, and a new class of problem where a change requires touching four modules.

**The threshold, derived from build time.** Assume a full build takes 8 minutes, an engineer makes 20 incremental builds a day, and a single module incremental build takes 45 seconds against 4 minutes for a monolithic incremental build.

- Saving per build: 4 minutes minus 45 seconds equals about **3.25 minutes**.
- Per engineer per day: 20 times 3.25 equals **65 minutes**.
- With 3 engineers: about 3 hours a day recovered, which pays for the modularisation work within weeks.
- With 1 engineer: about 1 hour a day, and the boundary maintenance cost is now a meaningful fraction of the saving.

So the honest threshold is roughly **three or more engineers touching the same codebase daily**, or a monolithic incremental build already over about two minutes. Below that, modularise to enforce layering only if you have already seen the layering violated.

### 4c. The layered picture, with state ownership marked

```
+---------------------------------------------------------+
| PRESENTATION   views  gestures  one state object         |
+---------------------------------------------------------+
              |  events go down and state comes back up
              v
+---------------------------------------------------------+
| DOMAIN   use cases  entities  invariants  no platform    |
+---------------------------------------------------------+
              |  calls go down and results come back up
              v
+---------------------------------------------------------+
| DATA   repository owns the truth  remote and local       |
+---------------------------------------------------------+
              |
              v
+-------------------+       +-----------------------------+
| REMOTE SOURCE     |       | LOCAL SOURCE  database      |
+-------------------+       +-----------------------------+

Figure three layers with dependencies pointing inward and truth in data

```

### 4d. Worked example: one screen, three asynchronous sources

The screen is a feed. Three sources feed it: a cached page from the local database, a network refresh, and a live stream of like counts. The user can also like a post, which is a write.

**Block 1. PURPOSE: name the single source of truth before naming any pattern.**

I ask one question of the design: after a like, which component holds the answer to "is this post liked"? If the answer is "the screen", I have the 4a bug. If the answer is "the repository", every screen showing that post agrees for free.

So the repository owns item state and exposes it as an observable query over the local database. The network writes into the database. The screen reads only from the database. The network never talks to the screen.

**The error most readers make here** is choosing a presentation pattern first. The presentation pattern is downstream of where the truth lives, and if the truth lives in the view, no pattern rescues it.

!!! note "Say it before you read on"

    Say what happens to the like state on a configuration change or a low memory recreation under each of the two options above.

**Block 2. PURPOSE: collapse three asynchronous sources into one state so an impossible screen cannot be rendered.**

Three independent observables can produce a frame where refreshing is false, items is empty and error is null, which is a blank screen with no explanation. I make the state one sealed value instead.

| State | Fields | What the screen shows |
|---|---|---|
| Loading | none | Skeleton placeholders |
| Content | items, isRefreshing, staleAsOf | The list, with an optional refresh indicator |
| Empty | reason | An explanatory empty state, not a spinner |
| Error | reason, cachedItems or none | Cached content plus a retry affordance, or a full error screen |

Note that Content carries isRefreshing rather than there being a separate refreshing flag. That is the whole trick: refreshing is only meaningful when there is content, so the type says so.

**The error most readers make here** is keeping a boolean per concern because it is easy to write. The cost arrives later as a bug report with a screenshot nobody can reproduce.

**Block 3. PURPOSE: model the optimistic write as a state, not a mutation.**

A like tap must feel instant, so I write it locally first. But a local write that is indistinguishable from a confirmed one is a lie the user will eventually catch.

So the database row carries the like value plus a pending marker. The repository exposes the merged view (local value wins), and the outbox owns the send. Three outcomes.

| Outcome | What the repository does | What the user sees |
|---|---|---|
| Server confirms | Clear the pending marker | Nothing changes, which is correct |
| Server rejects with a permanent error | Revert the local value, clear pending | The like visibly undoes, with a one line reason |
| Network unavailable | Leave pending, outbox retries | The like stays, because it will be sent |

!!! note "Say it before you read on"

    Before block 4, say why the pending marker has to be in the database rather than in memory. Module 2's runtime axis has the answer.

**Block 4. PURPOSE: pick the injection and module boundary from the test I want to be able to write.**

The test I want: give the domain layer a fake repository, apply a like event, assert the emitted state sequence, with no platform runtime and no database. That test is only possible if the use case depends on a repository interface, not a class.

So: constructor injection of an interface, the implementation bound in a data module, the domain module depending on nothing. With three engineers on this codebase, the modularisation threshold from 4b is met, so I split presentation, domain and data into build modules and let the build enforce what a code review otherwise has to.

**The error most readers make here** is introducing interfaces everywhere as a reflex. An interface earns its place when it has a second implementation, and a fake used in a test is a legitimate second implementation. An interface with exactly one implementation and no test double is overhead.

**Result.** The truth lives in one place, the screen cannot render an impossible combination, the optimistic write is honest about its own uncertainty, and the architecture is justified by a test rather than by a diagram.

### 4e. Fade the scaffold

**Faded example 1.** The screen is a chat conversation: a local message history, an incoming message stream, a typing indicator stream, and an outgoing send. Blocks 1 to 3 are given. Produce block 4.

- Block 1: truth is the local message database. Incoming messages are written to it, never delivered to the view directly.
- Block 2: one sealed state. Content carries messages, a connection banner value, and the typing indicator, because a typing indicator is only meaningful with content on screen.
- Block 3: an outgoing message is inserted immediately with status Pending, moves to Sent then Delivered, or to Failed with a retry affordance.
- Block 4: **you choose the injection approach and the module boundaries, justified by a test.**

??? note "Show answer"

    A defensible block 4. The test I want is: given a fake message repository and a fake clock, send a message, simulate the send failing twice then succeeding, and assert the status sequence is Pending, Pending, Sent with exactly one row in the database. That single test names three seams.

    | Seam | Why it must be injected |
    |---|---|
    | Message repository | So the test runs with no database or network |
    | Clock | Because retry backoff is time based and a test must not sleep |
    | Identifier generator | So the message and its idempotency key are deterministic in a test |

    The clock and the identifier generator are the two most commonly missed. Both are hidden global state, and both make the test flaky rather than failing, which is worse.

    Modules: presentation, domain and data as before, plus one detail. The typing indicator is ephemeral and never persisted, so it belongs to a separate lightweight source that the presentation layer combines, and it must not pass through the message repository. Putting an ephemeral signal into the persistence path is the most common structural error in this prompt.

**Faded example 2.** The screen is a photo picker: the device photo library, an in-progress selection, and an upload triggered on confirm. Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: truth for the library is the operating system's own photo store, which I query but do not copy. Truth for the selection is a state holder scoped to the picker flow, not to a single screen.
- Block 2: one sealed state with PermissionRequired, Loading, Content carrying a page of asset references plus the selected set, and Denied carrying whether the denial is permanent.

??? note "Show answer"

    Block 3, the write as a state: confirming a selection must not copy bytes on the main thread and must not block the user interface at all. I insert one durable upload record per selected asset, in state Queued, holding the asset reference and an idempotency key, and I dismiss the picker immediately. The upload itself is module 9's problem, and handing it off cleanly is the point.

    The distinctive risk here, and the reason this prompt is not just the chat prompt again: the asset reference may become invalid. The user can delete the photo from the library between confirming and uploading. So Queued must be able to transition to a terminal Unavailable state that is not a retryable failure, and the interface must be able to say so.

    Block 4, injection and modules: the seams are the photo library source (so the test does not need a device library), the clock, and the upload queue. The photo library source is the interesting one, because it is the only place in this feature that touches a platform permission, so isolating it behind an interface also isolates every permission code path into one testable class.

### 4f. Check yourself

**Q1.** A colleague proposes exposing four separate observable values from the state holder: items, isLoading, errorMessage and isEmpty. Give the concrete failing frame this permits, and the smallest change that makes it impossible rather than unlikely.

??? note "Show answer"

    The failing frame: a refresh completes with zero results. items updates to empty, isLoading updates to false, isEmpty has not been recomputed yet because it is derived from a different emission. For one frame the screen shows no list, no spinner, no error and no empty state. That is a blank screen, and it is the hardest class of bug to reproduce because it depends on emission ordering.

    The smallest change: make it one value with a sealed set of cases. Empty stops being a boolean and becomes a case, so it cannot be out of step with items, because it is the same value. Note the shape of the fix: the goal is not to be careful about ordering, it is to remove the ordering.

**Q2.** Your team has one engineer, a 90 second incremental build and 40 wired types. A reviewer says the app should be split into eight modules with a generated dependency graph. Using the thresholds in 4b, what do you say?

??? note "Show answer"

    Say no, with the arithmetic. The modularisation threshold is roughly three engineers or an incremental build over two minutes; this codebase has one engineer and 90 seconds, so the daily saving is small and the boundary maintenance cost is paid every day by the one person who would benefit. Forty wired types is well inside the range where manual constructor injection is readable and free at launch, and a generated graph solves an error rate that has not appeared yet.

    What I would do instead, because the reviewer is not wrong about the underlying worry: enforce the dependency direction with a lint rule or a package structure check today, which costs an afternoon. That captures the architectural benefit of modules without the build system cost, and it leaves the module split available later when the engineer count crosses the threshold.

### 4g. When not to use this

The layered plus unidirectional design is a good default, and defaults get applied where they do not pay.

| Question | Answer |
|---|---|
| The measurement that justifies the full structure | Two or more asynchronous sources feeding one screen, or the same data reachable from two entry points, or a state related bug that survived a code review |
| Scale threshold below which it is over-engineering | A single screen reading one endpoint with no writes and no cache. A screen object with a fetch and a state value is correct there, and a domain layer with one pass-through use case per call is ceremony |
| The cheaper alternative | Keep the sealed state object (it is nearly free and prevents the impossible frame), skip the separate domain layer, and let the screen talk to a repository directly |
| Failure mode of over-applying | A use case class per endpoint that does nothing but forward. Reviewers see it, it reads as cargo cult, and it makes navigation through the codebase slower for no test benefit |
| The tell in an interview | Naming a pattern without naming where it stops. Anyone can name three patterns. The score is in the breaking point |

---

## Module 5. Backend for frontend for mobile

**Time: 25 to 35 minutes reading, 15 to 20 minutes practice.**

### 5a. Predict first

Here is a feed response and the client code that consumes it.

```
GET  feed  returns
  items
    id
    author id
    author name
    text
    media key
    ranking score
    experiment bucket
    flags bitmask

client
  if appVersion is 3 or higher
      build image url with layout v2
  else
      build image url with layout v1

Figure a response that leaks the server model and a version ladder

```

??? note "Show answer"

    Prediction asked for: what breaks when version 4 ships, and what do the ranking score and experiment bucket fields cost you?

    The version ladder is the visible break. Version 4 adds a third branch, and now the server must reason about three client behaviours it cannot see. Worse, the branch is on the client, so the server cannot change the layout rule at all without a release. From module 2 that is 30 days, not 10 minutes.

    The flags bitmask is a slower break. A bitmask is a shared numeric contract with no room for a client that does not recognise a bit, so the first time the server sets bit 9, older clients either ignore it silently or misinterpret it. There is no default case in a bitmask.

    The ranking score costs you two things. It is server internal, so shipping it invites a client to sort by it, and once one client sorts by it the server can never change its scale. It is also a data exposure: a score is a signal about the ranking model, sitting on a device you do not control.

    The experiment bucket costs you the same way plus one more: it means the client now contains experiment logic, which puts experiment changes on the 30 day clock.

### 5b. The placement rule, derived rather than asserted

There is one rule, and it falls straight out of module 2c.

> Anything you might need to change in less than a week belongs on the server. Anything that must work with no server belongs on the client.

Apply it, and most placement arguments resolve without opinion.

| Concern | Where it belongs | The reason, in one line |
|---|---|---|
| Ranking, ordering, eligibility | Server | Changes weekly or faster, and the 30 day client clock cannot follow it |
| Business rules and access decisions | Server | Change rate plus the fact that a client can be modified by its user |
| Experiment assignment | Server, with the variant sent as data | Assignment logic on the client freezes the experiment design for a release cycle |
| Copy and strings that vary by experiment | Server, with a compiled in fallback | Same reason, plus the fallback keeps the app usable offline |
| Layout, gestures, animation | Client | The frame budget from module 3 is 6 milliseconds and a round trip is not available inside it |
| Offline behaviour, outbox, conflict resolution | Client | By definition the server is unreachable when it matters |
| Secrets and signing keys | Server | Assume the client is compromised. Module 16 covers this in detail |
| Date, number and name formatting | Client | Locale, time zone and accessibility text size are device properties, not account properties |
| Pagination cursors | Server, opaque to the client | An opaque cursor lets the server change the pagination strategy without a client release |

**What a backend for frontend actually is.** A service that exists to serve one client family, sitting in front of the domain services, owned by the client team, and free to change shape as fast as the client needs. It is not a gateway and it is not a caching layer, though it may do both.

**The one honest reason it exists.** Domain services are shared and therefore slow to change. Client screens are unshared and change constantly. Putting the fast changing shaping code behind an interface the client team owns is the whole idea.

### 5c. Response shaping, as a spectrum with a version skew cost

Four levels. Each is correct somewhere. The cost that rises down the list is version skew: the number of app versions in the wild that must all correctly render whatever the server sends.

| Level | Server returns | Client responsibility | Skew risk | Use when |
|---|---|---|---|---|
| 0 Domain entities | Normalised objects that mirror the domain | All shaping, joining and formatting | Low | Many heterogeneous clients, or a public interface |
| 1 Screen shaped view model | Fields named for what the screen shows, already joined and filtered | Bind fields to views | Medium | One or two first party clients, fast product iteration |
| 2 Typed component list | An ordered list of typed components, each with its data | Render known component types, skip unknown ones | High | You need to reorder or A/B test screen composition without a release |
| 3 Full layout tree | Layout, styling and behaviour as data | A generic renderer | Very high | Rarely worth it outside a few specific surfaces |

**When not to use each, stated plainly.**

- **Level 0 when not:** when a screen needs six calls to render, because each call costs a round trip. On a 150 millisecond cellular link, six sequential calls is 900 milliseconds of pure latency, which is most of the 1.5 second budget from module 3.
- **Level 1 when not:** when three different screens need almost the same data. You end up with three near duplicate responses and a shaping service that changes on every screen tweak.
- **Level 2 when not:** when you have fewer than roughly two years of supported client versions or no real need to reorder remotely. The renderer, the unknown component handling and the analytics plumbing are weeks of work, and they buy flexibility you are not using.
- **Level 3 when not:** almost always for a general product surface. You are rebuilding a layout engine over a network, you lose platform accessibility behaviour unless you reimplement it, and every rendering bug becomes a data bug that you cannot reproduce locally.

**The rule that makes level 2 survivable.** Unknown component types must be skipped silently and counted, never rendered as an error and never crashed on. The count is the important half: without telemetry on skipped components, a server change that strands 20 percent of clients is invisible.

### 5d. Worked example: shaping the feed response across three live app versions

The situation: versions 8.0, 8.1 and 8.2 are all in the wild, holding roughly 15, 25 and 60 percent of traffic. 8.2 shipped a new card type with a video preview. The product wants to launch a poll card next week.

**Block 1. PURPOSE: replace version numbers with capabilities, so the server stops guessing.**

The client sends what it can render, not what it is. One header or field at request time.

```
client sends
  supported cards   text  photo  video  poll
  supported media   avif  webp  jpeg
  screen width dp   412
  pixel density     3
  locale            en GB
  low data mode     false

Figure a capability declaration replacing a version number ladder

```

Now the server never needs a version table. A client that does not know polls simply does not list poll, and it is not sent one. When 8.3 adds polls, the same server code serves it with no change.

**The error most readers make here** is sending the app version and building the capability mapping on the server. That works until a version is hotfixed, or a build is region specific, or a client disables a feature at runtime because a flag turned it off. Capabilities are a property of the running client, and versions are only a proxy for them.

!!! note "Say it before you read on"

    Say how the capability approach changes the launch date of the poll card, compared with a version ladder. Use the numbers from module 2c.

**Block 2. PURPOSE: choose a shaping level for this specific screen, and say what it costs.**

The feed needs remote reordering and new card types without releases, so this is level 2, a typed component list. I say the cost out loud rather than pretending it is free.

| What level 2 costs here | The mitigation I am committing to |
|---|---|
| Unknown component types will reach old clients | Skip and count, with an alert if the skip rate on any version exceeds 1 percent |
| Analytics must work for components the client does not recognise | Impression logging is keyed by an opaque component identifier the server supplies, not by a client side type |
| A malformed component can blank a screen | Each component is parsed independently. A parse failure drops one card, never the page |

That third row is the one that separates a considered answer from an enthusiastic one. Whole page parsing of a heterogeneous list is the most common way this design fails in production.

**Block 3. PURPOSE: write the compatibility rules as rules, so they can be reviewed.**

Five rules, and the whole contract discipline is in them.

1. Fields are added, never removed and never repurposed. A field name means one thing forever.
2. Enumerated values are open. Every client ships a default case, and the server does not send a new value until the population that has the default case is large enough (see block 4).
3. Nothing server internal crosses the boundary. No ranking scores, no experiment names, no internal identifiers with meaning.
4. Cursors and tokens are opaque strings. The client stores and returns them and never parses them.
5. Every response carries a server supplied schema or configuration version, so telemetry can attribute a rendering bug to a server change rather than a client release.

**The error most readers make here** is treating "additive only" as the whole policy. Additive only is necessary and insufficient. Rule 2 is the one that actually bites: adding a value to an existing enumeration is additive and it still breaks every client that has an exhaustive switch on it.

!!! note "Say it before you read on"

    Say which of the five rules the flags bitmask in 5a violates, and whether the violation is fixable without a client release.

**Block 4. PURPOSE: decide when it is safe to send the new thing, using the adoption curve.**

The poll card requires clients to declare the poll capability, which lands in 8.3. From module 2c: about 60 percent of users are on the newest version at 7 days after availability and 90 percent at 30 days.

| Launch strategy | Reach at launch | Cost |
|---|---|---|
| Send polls the day 8.3 is available | Near zero, rising slowly | None technically, but the product sees a launch that does not launch |
| Wait 7 days after full availability | About 60 percent | The other 40 percent see a feed with a gap where a poll would be, unless the server substitutes |
| Substitute a photo card for clients without the capability | 100 percent | The server maintains two renderings of the same content, which is real work |

I choose substitution, because a feed with a hole in it is a worse product than a feed with an older card type, and because substitution is the mechanism that lets us stop caring about the 3 to 5 percent who never upgrade.

**The error most readers make here** is planning around a forced upgrade. Compute the cost first: at 10 million installs and a 3 percent permanent tail, forcing an upgrade below 8.3 strands **300,000 users**. Forced upgrade is a security tool, not a product convenience.

**Result.** The server no longer knows or cares about app versions, the shaping level is chosen with its costs named and mitigated, five reviewable compatibility rules exist, and the launch plan is built from the adoption curve rather than from hope.

### 5e. Fade the scaffold

**Faded example 1.** The surface is a settings screen whose rows differ by account type, region and several experiments. Blocks 1 to 3 are given. Produce block 4.

- Block 1: capabilities. The client declares which row control types it can render (toggle, chooser, link, destructive action) and its accessibility text scale.
- Block 2: shaping level 2, a typed list of rows, chosen because settings composition changes constantly and per region, and because a settings screen is exactly a list of typed components.
- Block 3: the five rules apply unchanged, with one addition: a destructive row must carry a server supplied confirmation string, so a new destructive action can never ship without its warning.
- Block 4: **you decide the rollout plan for a new row control type.**

??? note "Show answer"

    A defensible block 4. A new control type (say a slider) is exactly the enumeration problem in rule 2, so the sequence is fixed and it has three steps.

    1. Ship a client release that declares the slider capability and has a default case for unknown row types that skips and counts. Send nothing new yet.
    2. Wait until the population that declares the capability crosses the threshold you are willing to serve. Using module 2c, that is about 7 days after full availability for 60 percent and about 30 days for 90 percent.
    3. Serve sliders only to clients declaring the capability, and serve the previous control (a chooser with three fixed values) to everyone else.

    The substitution decision differs from the feed case, and the reason is worth saying: in a feed, a missing card is invisible to the user. In settings, a missing row is a setting the user cannot change, which is a support ticket and possibly a compliance problem. So settings requires substitution, where the feed merely prefers it.

**Faded example 2.** The surface is a checkout screen. Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: capabilities are the available payment method types, the supported authentication challenge types, and whether the client can present a web view for a redirect flow.
- Block 2: shaping level 1, a screen shaped view model, deliberately not level 2. Checkout has a small, slow changing set of surfaces and a high correctness bar, and a generic renderer makes correctness harder to reason about.

??? note "Show answer"

    Block 3, the compatibility rules with the checkout specific additions. The five rules hold, plus three that exist because money is involved.

    | Extra rule | Why checkout needs it |
    |---|---|
    | Amounts are integer minor units plus a currency code, never a formatted string and never a floating point number | Formatting is a client concern (rule from 5b) and floating point rounding on money is a defect class, not a style choice |
    | Every state changing call carries a client generated idempotency key, and the server returns the original result for a repeat | The client will retry, because module 6 says so, and a double charge is unrecoverable trust damage |
    | An unknown payment method or challenge type is a hard stop with a clear message, never a skip | This is the exact inversion of the feed rule, and the inversion is the point |

    Block 4, rollout. A new payment method must not be offered until the client can complete it end to end, so there is no substitution option and no partial launch. The plan is: ship the capability, wait for the capability declaring population to cross the threshold, then enable server side per region with a kill switch. The kill switch matters more here than anywhere else in this module, because a broken payment method costs revenue per minute and the client fix is 30 days away.

    The most common weak answer applies the feed's skip and count rule to checkout. Silently skipping a payment method the user expected is a worse failure than an honest error.

### 5f. Check yourself

**Q1.** A teammate proposes returning `displayPrice: "£12.99"` from the server so that all clients format identically. Give two concrete failures, and state the one case where their proposal is actually right.

??? note "Show answer"

    Failure one, accessibility and locale: the device owns the locale, the region format and the accessibility settings. A user with a British account travelling with a device set to another locale, or a user whose screen reader announces currency differently, gets the server's guess rather than their own settings. Formatting is a device property (5b).

    Failure two, arithmetic: a client that shows a subtotal, applies a discount, or animates a value cannot do any of it with a string, so you will end up sending the number as well and now there are two sources of truth that can disagree.

    Where they are right: when the value is not really a price but a piece of copy with legal or promotional wording, for example "from £12.99 per month, cancel anytime". That is a string, it is subject to legal review, and it must change without a client release. Send it as a string, and send the numeric amount separately for anything that computes.

**Q2.** You are on shaping level 2 and telemetry shows 4 percent of sessions skipping at least one component. Name the two most likely causes and the different fix for each.

??? note "Show answer"

    Cause one: a server change started sending a component type that a still large population does not declare. This is a rollout error against block 4, and the fix is immediate and server side: stop sending that type to clients that do not declare it, then re-launch once the declaring population crosses the threshold. Ten minutes, not 30 days.

    Cause two: the component is declared but its payload changed shape, so parsing fails for a subset. This is a rule 1 violation (a field was repurposed or made non-optional) and the fix is also server side: restore the previous shape and add the new field alongside. The tell that distinguishes the two is whether the skips correlate with the component type or with the client version. Type correlated means cause two, version correlated means cause one.

    Both fixes are server side, and that is the argument for level 2 rather than against it. On level 1 with client side branching, neither fix would be available for 30 days.

### 5g. When not to use this

A backend for frontend is a service, with a team, an on-call rotation and a deployment pipeline. That is a real cost.

| Question | Answer |
|---|---|
| The measurement that justifies building one | A screen needs three or more calls to render, or the client team is blocked on another team's release cycle to change a response shape, or two client platforms are diverging in what they need |
| Scale threshold below which it is over-engineering | One client platform, one team owning both sides, and fewer than about ten endpoints. A service in front of a service, deployed by the same people, adds a hop and an on-call rotation for nothing |
| The cheaper alternative | Add a screen shaped endpoint to the existing service, or add sparse field selection to the existing one. Both give you response shaping without a new deployable |
| Failure mode of over-applying | The backend for frontend becomes a second place where business rules live, because it was easier to add the rule there. Now the rule exists twice and the copies drift |
| The shaping level failure mode | Reaching for level 3 because it sounds impressive. Say what it costs (accessibility, local reproduction, a rendering bug becoming a data bug) and choose level 2 unless the surface really is layout driven |

---

## Module 6. Networking under constraint

**Time: 35 to 45 minutes reading, 20 to 30 minutes practice.**

### 6a. Predict first

Here is the network configuration of a real shaped client.

```
request timeout        30 s
max retries            5
backoff                1  2  4  8  16 s
retry on               any failure
concurrent requests    unlimited
deduplication          none
idempotency keys       none

Figure a network configuration with four independent defects

```

??? note "Show answer"

    Prediction asked for: the worst case time before the user is told anything, one user visible failure, and one server visible failure.

    Worst case time: 6 attempts at up to 30 seconds each is 180 seconds, plus the backoff sum of 1 plus 2 plus 4 plus 8 plus 16 equals 31 seconds. That is **211 seconds**, about three and a half minutes, before the user sees an error. No user waits that long, so in practice the user force quits and you lose the session with no telemetry.

    User visible failure: retry on any failure plus no idempotency keys means a send that timed out after the server processed it is retried, producing a duplicate. On a message that is annoying. On a payment it is unrecoverable.

    Server visible failure: no jitter means every client that failed during the same incident retries at the same instants, so the recovering server is hit by synchronised waves at 1, 3, 7, 15 and 31 seconds. That is a retry storm, and it can keep a server that was about to recover permanently down.

    Fourth defect, easy to miss: unlimited concurrency means a user's tap on a detail screen queues behind whatever background prefetching was already in flight, on a connection with limited parallelism. The user pays for work they did not ask for.

### 6b. Protocol choice, priced in round trips

Three terms first. **Round trip time** is one message to the server and back. **Head of line blocking** is one stalled item preventing later items from being processed. **Connection migration** is a connection surviving a change of network interface or address.

| Protocol | What it buys on a phone | What it costs | When not to use it |
|---|---|---|---|
| HTTP over TCP, one connection per request | Simplicity, works everywhere | A full handshake per request, and no multiplexing | Any screen making more than one call, which is nearly all of them |
| HTTP with multiplexing over one TCP connection | Many concurrent requests on one handshake, header compression | A lost packet stalls every stream on that connection, because the stall is below HTTP in the stack | Lossy cellular links where loss is common enough that the shared stall dominates |
| HTTP over a datagram transport with per stream loss recovery | No transport level head of line blocking, connection migration across a network change, and resumption with no round trip | Not available on every network path, and some middleboxes block it | Networks that block it. You must be able to fall back |
| Long lived socket both ways | Server can push, no per message request overhead | Holds the radio, needs its own reconnection, heartbeat and ordering logic | Anything that can tolerate a few seconds of delay. Use push plus fetch instead (module 10) |
| Remote procedure call over multiplexed HTTP with a schema | Generated clients, compact binary payloads, streaming | A schema and toolchain, and constrained browser reach | Small surfaces where a schema is more overhead than the payload it saves |

**The handshake arithmetic that decides connection reuse.** Assume a 150 millisecond round trip time on a mobile network, which is a reasonable middle for cellular.

| Situation | Round trips before the first byte of your request is sent | Time |
|---|---|---|
| New connection, transport handshake plus modern encryption handshake | 2 | 300 milliseconds |
| Resumed encrypted connection over the same transport | 1 | 150 milliseconds |
| Resumed connection on a transport that supports zero round trip resumption | 0 | 0 milliseconds |
| Connection already open and reused | 0 | 0 milliseconds |

Now apply it. A screen that makes 4 sequential calls, each on a fresh connection, spends 4 times 300 equals **1,200 milliseconds** on handshakes alone, which exceeds the entire 900 millisecond initial display budget from module 3. The same 4 calls on one reused multiplexed connection spend **0**.

That derivation eliminates two designs at once: it rules out one connection per request, and it rules out sequential dependent calls where a single shaped response would do (which is the argument that module 5 made from the other side).

**The mobile specific point worth saying out loud.** A phone changes network interface mid session, walking out of a building from wifi to cellular. A connection identified by an address dies at that moment and every in-flight request fails. A connection identified by a connection identifier can migrate and survive. That is not a micro-optimisation, it is the difference between a visible failure and no event at all, and it happens to every user every day.

### 6c. Quality of service tiers, retries, and deduplication

#### Tiers, because a single queue makes the user wait for the prefetcher

Define the tiers by who is waiting, then give each its own limits.

| Tier | Example | Concurrency | Timeout | Retries | On metered network | In low power mode |
|---|---|---|---|---|---|---|
| User blocking | Tap opens a detail screen | 4 | 10 seconds | 2 | Yes | Yes |
| User visible | Pull to refresh the feed | 4 | 15 seconds | 3 | Yes | Yes |
| Deferrable | Outbox send, telemetry batch | 2 | 30 seconds | Until success, with a cap | Yes | Deferred |
| Opportunistic | Image prefetch, cache warming | 2 | 20 seconds | 1 | Only small payloads | No |

**The derivation that justifies separate pools.** Assume one shared pool with 4 concurrent slots, and 20 opportunistic prefetches already queued, each taking 200 milliseconds.

- Time to drain the queue: 20 divided by 4, times 200 milliseconds, equals **1 second**.
- So a user tap can wait up to 1 second before its request even starts, on top of its own latency.
- With separate pools, the user blocking tier has its own 4 slots and starts immediately.

One second of pure queueing is worth roughly the entire first render budget from module 3, which is why this is an architecture decision and not a tuning knob.

#### The retry taxonomy

Retrying the wrong thing is worse than not retrying.

| What happened | Retry | How | Why |
|---|---|---|---|
| No network route reported by the operating system | No | Wait for the connectivity callback | Retrying with no route is pure battery cost and cannot succeed |
| Name resolution or connect failure | Yes | Full jitter backoff | Transient by nature |
| Server returned 500, 502, 503, 504 | Yes | Full jitter, honour any retry-after hint | Transient by contract |
| Server returned 429 | Yes | Honour retry-after, otherwise jitter with a longer base | You are being told to slow down |
| Read timeout after the request was fully sent | Only with an idempotency key | Same key, full jitter | The server may already have applied it |
| Server returned 400, 401, 403, 404, 422 | No | Surface it | A retry cannot fix a request the server understood and rejected |
| Server returned 401 specifically | Once, after a token refresh | Refresh, then one retry | Distinguish "refresh and retry" from "retry", and never loop |

**Full jitter, and why it is not optional.** The rule is: wait a random duration between zero and the current backoff ceiling, rather than waiting the ceiling.

Derive the effect. Assume 10,000 clients failed during the same incident, and the fourth backoff step has a ceiling of 8 seconds.

- Without jitter, all 10,000 retry within the same few tens of milliseconds. The recovering server sees an instantaneous 10,000 request spike.
- With full jitter over an 8 second window divided into 100 millisecond buckets, there are 80 buckets, so the expected arrivals per bucket are 10,000 divided by 80, which is **125**.
- That is an 80 times reduction in peak, from arithmetic alone, with no coordination between clients.

**The retry budget, which is the part most candidates miss.** Cap retries as a fraction of successful requests over a rolling window, for example 10 percent. When the budget is exhausted, fail immediately without retrying.

Why: during a total outage there are no successes, so the budget is zero, so clients stop retrying entirely. Without a budget, a full outage means every client sends 1 attempt plus 5 retries, which is **6 times normal load** aimed at a server trying to come back. The budget converts a self-inflicted denial of service into a graceful stop.

**Deadlines beat retry counts.** Give the operation a total budget and let it fit as many attempts as it can.

Worked: a 10 second user deadline, 1 second base backoff, doubling, with a 2.5 second per attempt timeout. Attempt 1 at t equals 0, fails at 2.5. Backoff up to 1 second, attempt 2 at about 3.5, fails at 6. Backoff up to 2 seconds, attempt 3 at about 8, must complete by 10. So **three attempts fit**, and the fourth is not started because it cannot finish. The user is told at 10 seconds, not at 211.

#### Deduplication, coalescing and idempotency

Three different mechanisms that are often confused.

| Mechanism | Where it lives | What it prevents |
|---|---|---|
| In-flight coalescing | Client, keyed by a canonical request signature | Two callers issuing the same GET at the same time |
| Result caching with a freshness window | Client | The same GET being reissued a second later |
| Idempotency key | Client generated, honoured by the server | A retried write being applied twice |

**Coalescing, quantified.** On a feed screen with 8 cards by 3 distinct authors, naive avatar loading issues 8 requests. Coalescing on the signature (method, path, sorted query, body hash) collapses them to **3**, a 62 percent reduction, with no cache and no server change.

**Idempotency keys, and the one rule that makes them work on mobile.** The key is generated when the operation is created, not when the request is sent, and it is stored with the operation in durable storage.

The reason is module 2's runtime axis: if the key is generated at send time, a process killed between two send attempts generates a new key on relaunch, and the server sees two distinct operations. A key persisted with the operation survives process death, which is the only case that matters.

#### The request pipeline

```
+-----------+   +-------------+   +---------------+
| CALL SITE |==>| DEDUPE      |==>| TIER QUEUE    |
| use case  |   | in flight   |   | blocking 4    |
|           |   | map by sig  |   | visible   4   |
+-----------+   +-------------+   | deferrable 2  |
                                  | opportunist 2 |
                                  +---------------+
                                          |
                                          v
+-------------+   +-------------+   +---------------+
| RESULT      |<==| RETRY       |<==| CONNECTION    |
| or typed    |   | taxonomy    |   | pool reused   |
| error       |   | jitter      |   | multiplexed   |
+-------------+   | budget      |   +---------------+
                  +-------------+

Figure the path of one request through dedupe tier retry and pool

```

### 6d. Worked example: the network path of a like tap

The operation: the user taps like. It must feel instant, must not double apply, and must survive the app being killed.

**Block 1. PURPOSE: classify the operation before choosing any policy, because the tier decides everything else.**

Is the user waiting on the network? No. Module 4's optimistic write already put the like on screen from local state. So this is not user blocking, it is **deferrable**: it must eventually happen, and nobody is watching the clock.

That single classification gives me the concurrency limit (2), the timeout (30 seconds), the retry policy (until success with a cap), and the low power behaviour (defer). None of those were separate decisions.

**The error most readers make here** is classifying by importance rather than by who is waiting. A like is important and nobody is waiting for it. A prefetch is unimportant and nobody is waiting for it. Importance decides how hard you retry, waiting decides the queue.

!!! note "Say it before you read on"

    Say what changes about every one of those four settings if the product decides the like button must show a confirmed state before the animation completes.

**Block 2. PURPOSE: make the operation durable and identified before it is allowed near the network.**

Order matters and it is counterintuitive: persist first, send second.

1. Write the intent to the outbox table: entity identifier, desired value, a generated idempotency key, created timestamp, attempt count, state Pending.
2. Update the local item so the interface shows the like.
3. Only then enqueue a send.

If the process dies between steps 1 and 3, the outbox still holds the intent and the send is retried on relaunch with the same key. If the key had been generated at step 3, the relaunch would generate a second key and the server would see two likes.

**The error most readers make here** is generating the identifier in the network layer. It looks like a networking concern and it is a durability concern.

**Block 3. PURPOSE: derive the retry policy from a deadline instead of picking a retry count.**

A like has no user deadline, but it does have a usefulness horizon. I set 24 hours: after that, the intent is stale enough that silently dropping it is better than applying it.

| Window | Backoff behaviour | Attempts, approximately |
|---|---|---|
| First 60 seconds | Full jitter, base 1 second, doubling to a 30 second ceiling | 5 to 6 |
| Next 24 hours | Full jitter with a 15 minute ceiling, only when connectivity is reported | About 96 |
| After 24 hours | Stop, mark the row Expired, revert the local value, count it | 0 |

The connectivity gate is what makes this cheap. From the retry taxonomy, no route means do not attempt, so a device offline for 8 hours performs zero attempts rather than 32 wasted radio wakes.

!!! note "Say it before you read on"

    Say why the retry budget from 6c matters even though this single operation retries at most about a hundred times over a day.

**Block 4. PURPOSE: handle the rapid double tap, which is a deduplication problem and not a retry problem.**

The user taps like, unlike, like within a second. Three intents, one final truth.

The outbox is keyed by (entity identifier, operation type), not by intent instance. A new intent for the same key **replaces** the pending row rather than appending to it, provided the existing row has not started sending. If it has started, the new intent is queued behind it and the idempotency key is regenerated, because it is genuinely a second operation.

The result: three taps produce one request in the common case, and never produce an inconsistent final state.

**The error most readers make here** is treating this as debouncing in the interface. A debounce on the button hides the symptom and does nothing for a tap on one screen followed by a tap on a duplicate of that screen, or for two intents that survive a process restart. The collapse belongs in the durable queue.

**Result.** One classification decision produced four policies, durability came before transmission, the retry policy came from a stated horizon rather than a habit, and the double tap was solved where the state lives rather than where the finger lands.

### 6e. Fade the scaffold

**Faded example 1.** The operation is prefetching the next screen of feed images. Blocks 1 to 3 are given. Produce block 4.

- Block 1: nobody is waiting, and it is not important, so this is opportunistic: 2 concurrent, 20 second timeout, 1 retry, suspended in low power mode, and restricted on a metered network to small payloads.
- Block 2: nothing is persisted. A prefetch that is lost to a process kill is simply not done, and re-deriving it on relaunch is cheaper than storing it.
- Block 3: retries are capped at 1 because a prefetch that fails is worth less than the radio time a second attempt costs. The deadline is the moment the user scrolls past the item, at which point the request is cancelled.
- Block 4: **you handle deduplication and cancellation.**

??? note "Show answer"

    Deduplication: coalesce on the canonical request signature, because a prefetch for an image and a real display request for the same image are the same GET. The correct behaviour when a display request arrives for an in-flight prefetch is not to issue a second request, it is to **promote** the in-flight one to the user blocking tier and attach the second caller to it. That promotion is the interesting half, and most answers miss it.

    Cancellation: it must be driven by the scroll position, and it must reach the socket rather than merely dropping the callback. A cancelled request whose bytes still arrive has spent the radio time anyway, so an uncancelled prefetch is a battery cost with no benefit at all. Concretely: cancel when the item leaves the prefetch window in either direction, and cancel everything on screen exit.

    The subtlety worth naming: cancelling a request that is 90 percent downloaded wastes more than finishing it. A reasonable rule is to let a request finish if its response is nearly complete, and to cancel it otherwise.

**Faded example 2.** The operation is submitting a payment. Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: the user is watching a spinner and will not leave the screen, so this is user blocking. Concurrency is irrelevant (there is one), timeout is 10 seconds per attempt.
- Block 2: persist first. The payment intent, its idempotency key and its state go into durable storage before the request is issued, because the user may force quit at exactly the wrong moment and the app must be able to ask the server what happened.

??? note "Show answer"

    Block 3, retry policy from a deadline. The deadline is what the user will tolerate while staring at a spinner, so 15 seconds total, not the 24 hours a like gets. With a 10 second per attempt timeout, that is one attempt plus at most one retry, and the retry only happens for connect failures and 5xx.

    The read timeout row of the taxonomy is the critical one here: if the request was fully sent and the response never arrived, the server may have taken the payment. Retrying with the same idempotency key is safe and is the correct behaviour. Retrying without one is a double charge.

    Block 4, deduplication and the part that only payments need: after the deadline expires with no answer, the client must not report failure. It must report **unknown** and move to a reconciliation state, polling a status endpoint keyed by the same idempotency key with backoff. The interface says "confirming your payment", not "payment failed".

    Two diagnosed wrong answers. First, showing failure on timeout, which trains users to pay twice. Second, treating the retry as a fresh submission with a new key, which is how a double charge happens even with idempotency implemented, because the key was scoped to the request instead of to the intent.

### 6f. Check yourself

**Q1.** Your client has 50,000 active devices. A backend deploy causes 90 seconds of 503 responses. With the 6a configuration (5 retries, backoff 1, 2, 4, 8, 16, no jitter, no budget), estimate the load pattern the recovering servers see. Then estimate it with full jitter and a 10 percent retry budget.

??? note "Show answer"

    With the 6a configuration: all 50,000 clients that were mid request fail at roughly the same moment, so they retry together at about t plus 1, t plus 3, t plus 7, t plus 15 and t plus 31 seconds. Five synchronised spikes of up to 50,000 requests each, arriving in windows of tens of milliseconds. Total extra requests: 50,000 times 5 equals **250,000** on top of normal traffic, concentrated into five instants. The 90 second outage plausibly extends well past 90 seconds because each spike can re-trigger the failure.

    With full jitter: the same 250,000 requests spread across the backoff windows. The largest window is 0 to 31 seconds, so peak instantaneous load drops by roughly the ratio of the window to the spike width, which is two orders of magnitude.

    With a 10 percent retry budget as well: during the outage the success count is near zero, so the budget empties after roughly the first wave. Clients then fail fast instead of retrying, and the extra load collapses toward **50,000 total** rather than 250,000. The user experience is worse per request and the system recovers far faster, which is the correct trade.

**Q2.** A screen makes 5 calls to render. Using the round trip arithmetic in 6b, quantify the cost of making them sequentially on fresh connections against making them concurrently on one reused connection, at 150 milliseconds round trip time and 100 milliseconds of server time each. Then say which module 5 decision this argues for.

??? note "Show answer"

    Sequential on fresh connections: each call costs 2 round trips of handshake (300 milliseconds) plus 1 round trip of request and response (150) plus 100 milliseconds of server time, which is 550 milliseconds. Five of those in sequence is **2,750 milliseconds**.

    Concurrent on one reused, multiplexed connection: handshake already paid, so the cost is 150 milliseconds of round trip plus 100 milliseconds of server time, and the five overlap. Total is about **250 milliseconds**, roughly 11 times better.

    Even the good case is not free, though, and that is the point: 250 milliseconds is a quarter of the 900 millisecond initial display budget from module 3, spent on a screen that could have been one request. This argues for the module 5 decision to move from level 0 domain entities toward a level 1 screen shaped response, which collapses 5 calls to 1 and removes the multiplexing question entirely.

### 6g. When not to use this

The full pipeline (tiers, dedupe, budgets, deadlines) is the right default for a consumer app at scale, and it is genuinely too much for some clients.

| Question | Answer |
|---|---|
| The measurement that justifies the full pipeline | More than about 10 concurrent request sources, or any background work competing with user initiated work, or a retry related incident that has already happened once |
| Scale threshold below which it is over-engineering | An app with under about 20 endpoints, no prefetching and no background sync. There is nothing for the tiers to separate, because everything is user blocking |
| The cheaper alternative | Three things, in this order: a total deadline instead of a retry count, full jitter, and idempotency keys on writes. Those three are perhaps 40 lines and they prevent the two failures that actually reach production |
| Failure mode of over-applying | Four queues configured for an app that only ever has one request in flight, and a retry budget that fires during normal operation because the success window is too small to be statistically meaningful at low traffic |
| The specific over-engineering tell | Building a custom transport or a bespoke connection pool. Use the platform stack and configure it. Almost nobody needs to write this layer, and interviewers know it |

---

## Module 7. Pagination done properly

**Time: 25 to 35 minutes reading, 15 to 25 minutes practice.**

### 7a. Predict first

A client paginates a feed with offset and limit. One item is inserted at the head between the two requests.

```
t0   server list       A B C D E F G H
     client requests   limit 4  offset 0
     client receives   A B C D

t1   item Z is inserted at the head
     server list       Z A B C D E F G H

t2   client requests   limit 4  offset 4
     client receives   D E F G

Figure an offset paginated list with one insertion between two pages

```

??? note "Show answer"

    Prediction asked for: which item does the user see twice, and which item would they never see if an item had been deleted instead of inserted?

    **Duplicate.** At t2 the list is Z A B C D E F G H, so indices 4 through 7 are D, E, F and G. The client already holds A, B, C and D, so **D appears twice**. Every insertion above the read position shifts the whole window down by one and duplicates one item per insertion.

    **Skip.** Run it the other way. Delete A between the requests, so the list becomes B C D E F G H. Indices 4 through 7 are now F, G and H. The client holds A, B, C and D, so **E is never fetched by any request**. It is not late, it is gone from this session entirely.

    The general statement, which is the thing worth carrying: with offset pagination, every insertion above the cursor duplicates one item and every deletion above the cursor skips one. The rate of both is exactly the mutation rate above your read position, so a fast moving feed corrupts every page.

### 7b. The four schemes, compared on the things that actually differ

Terms first.

| Term | Definition |
|---|---|
| Offset | Skip this many rows, then return the next N |
| Page number | The same as offset, computed as page minus one, times page size |
| Cursor | An opaque token issued by the server that encodes a position. The client stores it and returns it unread |
| Keyset (also called seek) | Ask for rows strictly after a given sort key value, using the sort key itself as the position |

| Property | Offset | Page number | Cursor | Keyset |
|---|---|---|---|---|
| Stable when items are inserted above | No | No | Yes | Yes |
| Stable when items are deleted above | No | No | Yes | Yes |
| Stable when items are reordered | No | No | Only with a snapshot | No |
| Cost at deep positions | Grows with offset | Grows with offset | Constant | Constant |
| Can jump to an arbitrary page | Yes | Yes | No | No |
| Server can change strategy without a client release | No | No | Yes | No |
| Bidirectional (page backwards) | Yes | Yes | Only if designed for it | Yes, with a reversed comparison |

**The deep page cost, derived.** A database asked for offset 100,000 with limit 20 must produce and discard 100,000 rows before returning 20. It reads roughly **100,020** index entries. The keyset form, "where sort key is less than the last value I saw", seeks directly and reads **20**. That is about a **5,000 times** difference in work, and it grows without bound as the user scrolls.

**Keyset needs a unique sort key, and here is why.** Sort by a timestamp with second granularity and three items share a timestamp. The page boundary lands in the middle of that group. The next request asks for items strictly before that timestamp, so the other two items in the group are skipped. Ask for items at or before it, and the item you already have is duplicated.

The fix is a compound key: sort by (timestamp, identifier) and compare the pair. The identifier breaks every tie, so the ordering is total and the boundary is exact.

**Cursor is not an alternative to keyset, it is a wrapper around it.** A cursor is usually a keyset position with a signature and a version stamp, encoded so the client cannot read it. That opacity is the whole benefit: the server can switch from a timestamp key to a score key, or add a snapshot identifier, without a client release. From module 2, that is worth 30 days.

#### The four failure modes, named

| Failure mode | What the user sees | Root cause |
|---|---|---|
| Duplicate | The same post twice, sometimes hundreds of rows apart | Insertion above the read position with a position based scheme |
| Skip | Nothing. That is what makes it dangerous | Deletion above the read position |
| Unstable ordering | Items visibly jump between requests | The sort key is recomputed per request, as with a ranked feed |
| Phantom end of list | Scrolling stops early | A page returned only items the client already had, the client deduplicated them all, and treated an empty result as the end |

The fourth is the subtle one and it is common. Two rules prevent it.

1. End of list is a server assertion, not a client inference. The server returns an explicit "no more" signal, and the client never concludes the end from a short or empty page.
2. If a page deduplicates to zero items, the client requests the next page automatically rather than stopping, with a small cap on consecutive empty pages so a broken cursor cannot loop.

#### Ranked feeds, where even keyset is not enough

A ranked feed recomputes scores per request, so the sort key itself changes between page one and page two. Keyset on a moving score is unstable by construction.

The fix is a **ranking session**: the server materialises an ordered list of identifiers once, stores it, and paginates within it. The cursor becomes (session identifier, position).

Cost it, so that the answer is a design rather than a wish. Assume 10 million daily active users, 3 sessions each per day, a materialised list of 500 identifiers, 8 bytes per identifier, and a 30 minute session lifetime.

- Bytes per session: 500 times 8 equals **4 kilobytes**.
- Sessions per day: 10 million times 3 equals **30 million**.
- Concurrently live sessions: a day holds 48 half hour windows, so about 30 million divided by 48, which is **625,000**.
- Memory at any instant: 625,000 times 4 kilobytes equals about **2.5 gigabytes**.

Two and a half gigabytes in a cache tier is unremarkable, so the design is affordable, and now you can defend it with arithmetic rather than with confidence. If the number had come out at 2.5 terabytes, the correct answer would be to materialise only the first 100 identifiers and re-rank beyond that, accepting instability deep in the feed where almost nobody scrolls.

### 7c. The client half, which is where most of the bugs live

The server can be perfect and the list can still be wrong. Five client rules.

| Rule | Why |
|---|---|
| Deduplicate by stable identifier at merge time, always | Even a correct cursor scheme produces overlap after a refresh, and this is one line of code |
| Refresh and append are different operations with different merge rules | Append adds to the tail. Refresh replaces the head and must not clear the list, or the user loses their scroll position |
| Detect the gap after a refresh | If the refreshed head does not overlap the cached list at all, there is a hole. Either clear the list or insert an explicit gap marker the user can tap |
| Anchor scroll position to an item identifier, not a pixel offset | Prepending items shifts every pixel offset, and the user's reading position jumps |
| Persist the cursor with the cached page | Otherwise a process death mid-scroll restarts the feed from the top, which is module 2's runtime axis again |

```
CACHED       A B C D E F
REFRESH      Z Y A B
overlap at A so prepend Z Y and keep scroll anchored on D

CACHED       A B C D E F
REFRESH      Q R S T
no overlap so there is a hole between T and A

+-------------------------------+
| Q R S T                       |
| = = = gap  tap to load = = =  |
| A B C D E F                   |
+-------------------------------+

Figure refresh merged with overlap and refresh that reveals a hole

```

### 7d. Worked example: pagination for a ranked feed with pull to refresh

The prompt: an infinite feed, ranked server side, with pull to refresh and occasional real time insertions at the head.

**Block 1. PURPOSE: choose the scheme from the ordering property, not from habit.**

I ask one question: is the ordering stable between requests? The answer is no, because ranking is recomputed. That single fact eliminates offset, page number and plain keyset in one move, and it leaves cursor plus a ranking session.

I say the elimination out loud, because eliminating three options with one question is exactly what the estimation discipline is for.

**The error most readers make here** is reaching for cursor pagination because it is the known good answer, without noticing that a cursor over an unstable ordering is still unstable. The cursor is necessary and it is not sufficient.

!!! note "Say it before you read on"

    Say what specifically would be unstable about a cursor that encoded the ranking score of the last item seen.

**Block 2. PURPOSE: design the cursor so the server keeps its freedom.**

The cursor is an opaque string. Inside it, and invisible to the client, the server puts the ranking session identifier, the position in the materialised list, and a schema version.

| Property of this cursor | What it buys |
|---|---|
| Opaque | The server can change the entire scheme later without a client release |
| Carries a session identifier | Page 2 comes from the same materialised ordering as page 1 |
| Carries a version stamp | A cursor issued by an old server can be recognised and rejected cleanly rather than misread |
| Has an expiry | A cursor resumed after two days points at a session that no longer exists, and the server can say so |

The expiry needs a defined behaviour, not just a rejection. On an expired cursor the server starts a fresh session and returns a flag telling the client this is a discontinuity, so the client can clear rather than append.

**Block 3. PURPOSE: define refresh separately from append, because they are not the same request.**

| Operation | Request | Server behaviour | Client merge |
|---|---|---|---|
| Initial load | No cursor | Materialise a new session, return page 1 plus a cursor | Replace the list |
| Append | Cursor | Continue in the same session | Append, deduplicate by identifier |
| Pull to refresh | No cursor, plus the identifier of the newest item held | New session, return the head | Prepend if it overlaps, otherwise show a gap marker |
| Real time insert | A push, then a fetch of the head | Same as refresh | Show a "new posts" pill rather than moving content under the user's thumb |

That last row is a product decision with a technical cause. Inserting content above the reading position while the user is reading moves what they are looking at, so the correct behaviour is to hold the new items and offer them.

**The error most readers make here** is one endpoint with one merge rule for both refresh and append. The list then either loses scroll position on every refresh or accumulates duplicates on every insertion.

!!! note "Say it before you read on"

    Say why the refresh request sends the newest held identifier, when the server is going to materialise a new session anyway.

**Block 4. PURPOSE: state the client rules that make the correct server behaviour survive contact with a real device.**

Four commitments, all from 7c: deduplicate on merge, anchor the scroll to an item identifier, persist the cursor with the cached page so a process kill resumes rather than restarts, and never infer the end of the list from an empty page.

I add one measurement so the design is checkable in production: count duplicate identifiers observed per session and the rate of gap markers shown. Both should be near zero, and a rise in either is a pagination regression that no crash reporter will ever tell you about.

**The error most readers make here** is shipping pagination with no telemetry. Duplicates and skips are silent, users rarely report them, and without a counter you hear about them from a screenshot on social media.

**Result.** One question eliminated three schemes, the cursor is opaque so the server keeps its freedom, refresh and append are different operations with different merge rules, and two counters make the whole thing observable.

### 7e. Fade the scaffold

**Faded example 1.** The prompt is paginating chat message history, scrolling upward into the past, while new messages arrive at the bottom. Blocks 1 to 3 are given. Produce block 4.

- Block 1: the ordering is stable, because messages are ordered by a server assigned sequence number that never changes. That admits plain keyset and there is no need for a ranking session.
- Block 2: the cursor is a keyset position over (sequence number), which is already unique, so no tie breaker is needed. Pagination is bidirectional: give me 50 before sequence X for history, give me everything after sequence Y for new messages.
- Block 3: history load and live tail are different operations. History appends upward and never touches the tail. The live tail appends downward and never touches history.
- Block 4: **you state the client rules and the telemetry.**

??? note "Show answer"

    Client rules, with the chat specific differences called out.

    | Rule | Chat specific note |
    |---|---|
    | Deduplicate by identifier | The identifier must be the server sequence, and locally sent messages need a client identifier that the server echoes so the optimistic row is replaced rather than duplicated |
    | Anchor scroll | Anchor to a message identifier, and when prepending history keep the anchored message at the same screen position. This is the single most visible bug in chat clients |
    | Persist the cursor | Persist the whole message window, not just the cursor, because chat must open instantly from local storage (module 3 gave a 1.0 second target) |
    | End of list is a server assertion | Here it is genuinely reachable, because conversations have a beginning, and the server must say so rather than the client inferring it from a short page |

    The extra rule chat needs and the feed did not: **gap detection by sequence continuity**. Because sequence numbers are dense, the client can detect a hole arithmetically. If it holds sequences 100 to 150 and 180 to 200, it knows 151 to 179 are missing and can fetch exactly that range. A ranked feed cannot do this, because its ordering has no arithmetic.

    Telemetry: count detected sequence gaps that persist after a fetch, and count optimistic messages that were never reconciled with a server sequence. Both should be zero.

**Faded example 2.** The prompt is paginating search results where the user can change the sort order mid scroll. Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: ordering is stable for a fixed query and sort, but a sort change is a different ordering entirely, so it is a new list rather than a continuation.
- Block 2: the cursor encodes the query, the sort, a result set identifier and a position, and it expires. Any change to query or sort invalidates the cursor by construction, because they are inside it.

??? note "Show answer"

    Block 3, operations: there are three, not two. Initial search, append, and **re-sort**, which is deliberately not a refresh. Re-sort discards the entire list, resets the scroll to the top, and starts a new result set. Trying to preserve position across a sort change is a bad idea: there is no meaningful mapping between position 40 in one ordering and any position in another, and attempting one produces a list the user cannot trust.

    There is no pull to refresh at all in this design, because search results are a snapshot of a query rather than a stream, and adding refresh invites exactly the mutation problems this prompt does not otherwise have. Saying "I am deliberately not adding refresh here" is a scoring move.

    Block 4, client rules and telemetry: deduplicate as always, do not persist the cursor beyond the session (a search result set from yesterday is not worth resuming, unlike a feed position), and cancel in-flight append requests immediately when the sort changes, or a late response will append results from the previous ordering into the new list. That last one is the distinctive bug in this prompt, and it is a cancellation problem rather than a pagination problem.

    Telemetry: count appends that arrived after a sort change and were discarded. A non-zero count is correct and expected. A zero count means the cancellation path is never exercised, which usually means it is not wired up.

### 7f. Check yourself

**Q1.** A feed serves 20 items per page with offset pagination. Measurement shows an average of 2 insertions above the read position per page request. A user scrolls 10 pages. Estimate how many duplicates they see, and say why the equivalent skip count is more damaging even when it is smaller.

??? note "Show answer"

    Each insertion above the read position shifts the window by one and duplicates one item. At 2 insertions per request over 10 requests, the user sees roughly **20 duplicates** across 200 items, about 10 percent of the feed. That is visible and it looks like a broken product.

    Skips are more damaging at any count for two reasons. First, they are invisible: a user cannot report a post they never saw, so the bug does not appear in support tickets and only shows up as a content distribution anomaly nobody attributes to pagination. Second, the cost is asymmetric on the business side. A duplicate wastes a slot. A skip means content that was ranked worth showing was never shown, so creators lose reach and the ranking system is being evaluated on impressions it never actually delivered.

**Q2.** Your cursor currently encodes a plain timestamp of the last item. An engineer proposes adding a secondary sort by popularity for a new surface. What breaks, and what is the minimal change?

??? note "Show answer"

    What breaks: the cursor is a keyset position over a single column, and the new ordering is over two columns. A position expressed as "before timestamp T" does not identify a position in a (popularity, timestamp) ordering, so the second page of the new surface is arbitrary. Worse, if the cursor is transparent (a raw timestamp the client can read), old clients may keep constructing cursors themselves and there is no way to stop them.

    Minimal change, in order: make the cursor opaque first, so this class of change stops requiring client releases. Then encode the full compound key inside it, (popularity, timestamp, identifier), with the identifier as the tie breaker so the ordering is total. Then version the cursor so a cursor issued under the old scheme is recognised and answered with a clean restart signal rather than being misinterpreted.

    The lesson to carry: the opacity of the cursor is not a detail, it is the thing that makes every later ordering change cheap.

### 7g. When not to use this

Cursor pagination with ranking sessions is correct for a large mutating feed and is genuine over-engineering for many lists.

| Question | Answer |
|---|---|
| The measurement that justifies cursor plus session | The collection mutates above the read position during a session, or the deep page query time is already visible in server latency percentiles |
| Scale threshold below which it is over-engineering | A collection under roughly 200 items that fits in one or two requests. Fetch it whole, cache it, and sort on the client. No pagination scheme at all is the correct design |
| Second threshold | An immutable or slow moving list (a user's own order history, a settings catalogue). Offset is fine, and page numbers may even be a product requirement |
| The cheaper alternative | Keyset over a compound key with a unique tie breaker. It is a few lines, it fixes duplicates and skips, and it does not need a session store. Add the session only when the ordering itself is unstable |
| Failure mode of over-applying | A ranking session store with a 2.5 gigabyte footprint protecting a list that changes twice a day. Measure the mutation rate above the read position before building it |
| The client side failure mode | Correct server pagination with no client deduplication. The cheapest fix in this entire module is the merge time deduplication, and it is the one most often missing |

---

## Module 8. Media in: the image pipeline

**Time: 35 to 45 minutes reading, 20 to 30 minutes practice.**

### 8a. Predict first

Here is how a list row loads its image, and what the user does.

```
row binds
    imageView load  url
    url points at the 4000 by 3000 original
    no target size passed
    no cancellation when the row is recycled
    memory cache    none
    disk cache      none

user flings the list past 60 rows in 3 seconds

Figure a naive image load in a recycling list during a fast scroll

```

??? note "Show answer"

    Prediction asked for: peak decoded memory, requests issued, and bytes transferred. Use 4 bytes per pixel and assume each original file is 3 megabytes on disk.

    **Decoded memory per image:** 4000 times 3000 times 4 equals 48,000,000 bytes, about **48 megabytes**. With three decodes in flight, that is 144 megabytes, which exceeds the 192 megabyte mid tier heap assumption from module 3 once the rest of the app is counted. The app does not get slow, it gets terminated.

    **Requests:** 60 rows in 3 seconds with no cancellation is **60 requests** in flight or queued, at **20 per second**, none of which are for a row the user is still looking at.

    **Bytes:** 60 times 3 megabytes equals **180 megabytes** downloaded, for images displayed for roughly 50 milliseconds each. At the 1 megabit per second assumption from module 3, that is 1,440 seconds of transfer requested in a 3 second window, so in practice the queue never drains and the visible rows load last.

    The fourth failure, the one users actually report: because requests complete out of order and the row was recycled, the wrong photo appears in the wrong row.

### 8b. The pipeline, stage by stage

Terms first.

| Term | Definition |
|---|---|
| Decode | Turning compressed bytes into a bitmap in memory, one entry per pixel |
| Downsample | Decoding at a reduced resolution so the bitmap is never bigger than it needs to be |
| Transform | Work applied after decode: crop, rounded corners, blur, colour conversion |
| Target | The view or surface that asked for the image and may be recycled before it arrives |

The stages, in order, with the decision that lives at each.

| Stage | What happens | The decision |
|---|---|---|
| 1 Key | Build a cache key from source plus target size plus transform chain | Get this wrong and everything downstream is wrong |
| 2 Memory cache | Look for a decoded, transformed bitmap | Size it against the heap budget, not by feel |
| 3 Disk cache | Look for the encoded bytes | Store encoded, not decoded. Decoded on disk is huge and not portable |
| 4 Network | Fetch, in the opportunistic or user blocking tier from module 6 | Coalesce concurrent requests for the same key |
| 5 Decode and downsample | Produce a bitmap no larger than the target needs | This is where the 48 megabytes becomes 1 megabyte |
| 6 Transform | Apply crop and shape | Do it before caching, so it is not repeated per bind |
| 7 Deliver | Check the target still wants this key, then set it | Skip this and you get the wrong photo in the recycled row |

**The cache key is the design.** The key must contain the source identifier, the target dimensions in pixels, and a stable description of the transform chain.

- Key on the URL alone and a 96 pixel avatar shares an entry with a 1080 pixel full screen render. One of them is wrong, and which one depends on scroll order.
- Include the transforms and a rounded thumbnail no longer collides with a square one.
- Two caches, two key shapes: the **disk** cache holds encoded bytes keyed by the source URL, because the bytes are the same whatever you will do with them. The **memory** cache holds decoded bitmaps keyed by source plus size plus transforms.

#### Downsampling, with the arithmetic

A 4000 by 3000 source, needed at 540 by 540 physical pixels for a feed card.

| Approach | Decoded pixels | Bytes at 4 per pixel | Ratio |
|---|---|---|---|
| Decode fully, then scale down for display | 12,000,000 | **48 megabytes** | 1 |
| Decode at a sample factor of 4 (1000 by 750) | 750,000 | **3 megabytes** | 16 times smaller |
| Decode at a sample factor of 4 then scale to exactly 540 by 540 | 291,600 | **1.17 megabytes** | 41 times smaller |

The sample factor is chosen as the largest power of two that still leaves both dimensions at or above the target. Here 4000 divided by 540 is 7.4 and 3000 divided by 540 is 5.6, so the largest safe power of two is **4**, giving 1000 by 750, both still above 540.

Never pick a factor that takes a dimension below the target, because upscaling a too-small bitmap is visible immediately and is the most common downsampling bug.

**Pixel format.** Four bytes per pixel gives 8 bits each of red, green, blue and alpha. Two bytes per pixel drops alpha and reduces colour precision, halving memory. It is a real option for opaque photographic thumbnails and a bad one for gradients or anything with transparency, where banding is obvious. State the trade rather than defaulting silently.

#### Sizing the three tiers, from the module 3 budget

| Tier | Size | Derivation |
|---|---|---|
| Memory | **24 megabytes** | One eighth of the 192 megabyte mid tier heap, the budget already set in module 3 |
| Disk | **150 megabytes** | See below |
| Network | Long lived, via immutable content addressed URLs | Costs nothing on the device and removes revalidation round trips |

**What 24 megabytes of memory cache actually holds.** At 1.17 megabytes per feed bitmap, 24 divided by 1.17 is about **20 bitmaps**. With 8 cards visible per screen, that is **2.5 screens** of scroll history, which is enough that a short scroll back is instant and a long one is not. That is the correct trade: instant scroll back is a nice-to-have, and termination from footprint is not.

**What 150 megabytes of disk cache actually holds.** At the module 3 figure of about 50 kilobytes per 540 by 540 encoded image, 150 megabytes holds about **3,000 images**. Assume a user views 5 screens per session at 8 images per screen, 3 sessions a day, so 120 images a day. That is **25 days** of history, far more than anyone scrolls back through, so the size is generous and could be halved without a measurable hit rate change.

Two disk cache rules that are not about size. First, store it in a location the operating system is allowed to purge, because a cache that the system cannot reclaim under storage pressure becomes the user's reason to delete your app. Second, write through a journal, so a process killed mid write leaves a recoverable cache rather than a corrupt entry.

#### Eviction, and the rule that prevents a crash

| Tier | Policy | The rule most designs miss |
|---|---|---|
| Memory | Least recently used, evicting by **byte size** rather than entry count | A bitmap currently attached to a visible view must never be evicted and recycled. Track in-use entries separately, or you will free a bitmap that is being drawn |
| Disk | Least recently used with a size cap and a maximum age | Enforce the cap on a background thread with a hysteresis band (trim to 80 percent when you hit 100 percent), or you trim on every insert |
| Both | Coalesce concurrent requests for the same key | Eight visible rows requesting the same avatar must produce one fetch and one decode, not eight. This is the module 6 dedupe applied to the media path |

Also respond to the system memory warning: drop the memory cache to a small fraction rather than clearing it entirely, because clearing it guarantees a full reload storm the moment the user scrolls.

#### Placeholders and prefetch, both derived

**Placeholders.** Embed a tiny preview in the list response itself, so a card is never blank. At 32 by 32 pixels the encoded preview is on the order of a kilobyte.

- 8 cards times 1 kilobyte equals **8 kilobytes** added to a 50 kilobyte page response, about **16 percent** larger.
- In exchange, first paint has real colour and shape in every card at the moment the list appears, with zero extra requests.

That is one of the best trades available on a feed, and the arithmetic is what makes it defensible rather than a matter of taste.

**Prefetch distance, derived.** Assume a fast scroll of 2,000 pixels per second and a card height of 400 pixels, so the user passes **5 cards per second**. Assume fetch plus decode takes 300 milliseconds, during which the user passes **1.5 cards**.

- Minimum useful prefetch distance: 2 cards.
- Add margin for latency variance: **4 cards ahead**.
- Speculative cost if the user stops immediately: 4 times 50 kilobytes equals **200 kilobytes** wasted, which is acceptable on an unmetered network and worth capping to 2 cards on a metered one.

Prefetch runs in the opportunistic tier from module 6, so it never competes with a request the user is waiting for.

### 8c. The pipeline as a picture

```
key = source id + target pixel size + transform chain
        |
        v
+---------------------------------------------------+
| MEMORY CACHE   24 MB   LRU by bytes   hit=deliver  |
+---------------------------------------------------+
        | miss
        v
+---------------------------------------------------+
| DISK CACHE   150 MB   encoded bytes   hit=decode   |
+---------------------------------------------------+
        | miss
        v
+---------------------------------------------------+
| NETWORK   opportunistic tier   coalesce same key   |
+---------------------------------------------------+
        |
        v
+---------------------------------------------------+
| DECODE and DOWNSAMPLE then TRANSFORM then cache    |
+---------------------------------------------------+
        |
        v
+---------------------------------------------------+
| DELIVER only if the target still wants this key    |
+---------------------------------------------------+

Figure the three tier image path with delivery guarded by the key

```

### 8d. Worked example: the image path for one feed card

The prompt: a feed of cards, each with one photo, on the mid tier device from module 3.

**Block 1. PURPOSE: write the cache key first, because every later decision depends on it.**

The key is source identifier plus target pixel size plus an ordered transform description. Concretely, a card image at 540 by 540 with a rounded corner transform is a different entry from the same photo at 96 by 96 for the author avatar.

I say why out loud: the memory cache stores decoded bitmaps, and a decoded bitmap has exactly one size, so a key that omits the size describes something that does not exist.

**The error most readers make here** is keying on the URL because that is what the disk cache does. The two tiers store different things and therefore need different keys, and conflating them means either wrong sized images or a memory cache that never hits.

!!! note "Say it before you read on"

    Say what would visibly go wrong if the transform chain were left out of the memory key, on a screen showing the same photo as both a rounded avatar and a square card.

**Block 2. PURPOSE: size the tiers from the budget I already committed to, not from a default.**

From module 3 I own 24 megabytes of image memory on a 192 megabyte heap. From 8b that is about 20 bitmaps at 1.17 megabytes each, which is 2.5 screens.

I state the consequence I am accepting: scrolling back more than about two screens will re-decode from the disk cache, which costs roughly the decode time rather than a network round trip. That is a deliberate trade of scroll-back smoothness against termination risk, and termination is worse (module 3 priced it at a 7.5 times penalty).

**The error most readers make here** is sizing the memory cache as a fraction of available memory measured at runtime. Available memory is high right after launch and low later, so the cache grows into exactly the memory the system was about to want back.

**Block 3. PURPOSE: keep the decode off the frame budget.**

Module 3 gives 6.2 milliseconds of application work per frame at 120 hertz. A decode of a 1000 by 750 intermediate is far longer than that, so the decode happens on a background thread, always, with no exception for cache hits from disk.

Two consequences I state:

- The bind path must be synchronous and cheap: set the placeholder, start the request, return. A synchronous disk read during bind is enough to drop frames on its own.
- The memory cache lookup may happen on the main thread, because it is a hash lookup of tens of microseconds, and making it asynchronous would cause a visible flash of placeholder on an image that was already in memory.

That second point is a real distinction and it is worth making: the fast path is synchronous, everything else is not.

!!! note "Say it before you read on"

    Say why making the memory cache lookup asynchronous produces a visible flicker, given that the lookup is fast either way.

**Block 4. PURPOSE: make cancellation and prefetch behave like one system.**

Every request carries the target's generation token. When a row is recycled the token changes, which does three things: the in-flight request is cancelled if it is not nearly complete, any arriving result is discarded rather than delivered, and the slot is freed for the newly bound row.

Prefetch uses the derived window from 8b: 4 cards ahead in the scroll direction, opportunistic tier, cut to 2 on a metered connection, suspended entirely in low power mode. When a prefetched request is still in flight and the row becomes visible, it is promoted to the user blocking tier rather than re-issued, which is the promotion rule from module 6.

**The error most readers make here** is cancelling only the callback and not the transfer. The bytes still arrive, so the radio time is spent and the battery cost of a fling through 60 rows is unchanged. Cancellation must reach the transport to be worth anything.

**Result.** The key is defined before the caches, the tiers are sized from a budget set two modules earlier, the decode never touches the frame budget, and cancellation and prefetch are the same mechanism seen from two directions.

### 8e. Fade the scaffold

**Faded example 1.** The surface is a full screen photo viewer with pinch to zoom, opened from the feed. Blocks 1 to 3 are given. Produce block 4.

- Block 1: the key now includes a zoom tier, because a zoomed region needs a different decode than the fitted view. Tiers are fit-to-screen, 2 times and 4 times.
- Block 2: memory sizing changes. One fitted full screen bitmap at 1080 by 2400 is 10.4 megabytes (module 3), so the 24 megabyte budget holds two of them plus nothing else. The viewer therefore gets its own smaller cache holding the current image and its immediate neighbours, and the feed's cache is left alone.
- Block 3: decode stays off the main thread, and the fitted image is shown first while a higher tier decodes in the background.
- Block 4: **you design cancellation and prefetch for the swipe between photos.**

??? note "Show answer"

    Prefetch: the user swipes left and right through the set, so prefetch is bidirectional and shallow. One neighbour each side is enough, because a swipe takes roughly 300 milliseconds and the fitted decode fits inside that. Prefetching three each side costs 6 times 10.4 megabytes of potential decoded memory for no perceptible gain, and would breach the heap.

    Prefetch what, exactly: the encoded bytes into the disk cache, not the decoded bitmap into memory. This is the key difference from the feed case. Decoding ahead is what blows the heap here, whereas in the feed the bitmaps were small enough that decoding ahead was fine.

    Cancellation: a zoom tier decode must be cancelled the moment the user zooms back out or swipes away, and it must be cancellable mid decode, not only before it starts. A 4 times tier decode of a large photo can take hundreds of milliseconds, so a non-cancellable decode blocks the pool while the user is already looking at the next photo.

    The subtle rule: on leaving the viewer, drop the zoom tiers immediately but keep the fitted bitmap, because the user is returning to a feed that will want a smaller version of the same photo and the fitted one can be downscaled cheaply.

**Faded example 2.** The surface is a chat list showing 40 small avatars, many of them repeated because the same people appear in several conversations. Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: key is source plus 96 by 96 pixels plus a circular crop transform. Small and uniform.
- Block 2: sizing is different in kind. At 96 by 96 and 4 bytes per pixel, one avatar is 96 times 96 times 4 equals 36,864 bytes, about 36 kilobytes. A 4 megabyte avatar cache holds about 110 avatars, which is more distinct people than a chat list shows, so the entire working set fits in memory.

??? note "Show answer"

    Block 3, decode and the frame budget: a 36 kilobyte decode is fast, but 40 of them during a fling still add up, so decode stays off the main thread. The more important point here is that the working set fits entirely in memory, so after the first load there are no decodes at all during scroll. That changes the design goal from "make decode cheap" to "make sure the first load is complete and correct", which is the opposite emphasis from the feed.

    Block 4, cancellation, prefetch and the distinctive one: **deduplication by identity rather than by request**. The same person appears in 6 conversations, so 6 rows request the same key simultaneously. Coalescing collapses those to one fetch and one decode, and all 6 rows receive the same bitmap instance. Six rows sharing one bitmap is safe here precisely because the memory cache must not recycle a bitmap that is in use, which is the eviction rule from 8b earning its place.

    Prefetch is nearly pointless here and saying so is the right answer: the whole list is small, the images are tiny, and the correct move is to warm the entire avatar set once on conversation list load rather than to prefetch along a scroll direction. Cancellation still matters for the initial burst, but recycling churn is not the problem it was in the feed.

    The most common weak answer copies the feed design wholesale, including a 4 card prefetch window and a 24 megabyte cache, for a working set that is under 4 megabytes.

### 8f. Check yourself

**Q1.** Your feed shows 8 cards per screen and users report the app being killed in the background. Memory profiling shows 24 megabytes in the image cache and 90 megabytes in bitmaps that are not in the cache. Explain what is most likely happening and give the fix.

??? note "Show answer"

    Ninety megabytes outside the cache means bitmaps are being decoded at the wrong size, retained by views, or both. Two likely causes.

    Cause one, and the most common: the decode is not downsampled to the target. From 8b, a full decode of a 4000 by 3000 source is 48 megabytes against 1.17 megabytes at the card size, a factor of 41. Two undownsampled images account for the whole 90 megabytes. The fix is to pass the target size into the decode, which is stage 5 of the pipeline.

    Cause two: recycled views hold strong references to their bitmaps, so bitmaps for rows the user scrolled past hundreds of rows ago are still reachable and cannot be collected. The fix is to clear the image on recycle, which is the same generation token mechanism as cancellation.

    How to tell them apart in one step: check the dimensions of the retained bitmaps. If they are 4000 by 3000, it is cause one. If they are 540 by 540 and there are 70 of them, it is cause two.

**Q2.** A teammate proposes caching decoded bitmaps on disk to avoid re-decoding. Evaluate it with numbers from this module.

??? note "Show answer"

    Size: a decoded 540 by 540 bitmap is 1.17 megabytes against about 50 kilobytes encoded, roughly **23 times larger**. The 150 megabyte disk budget would hold about 128 images instead of 3,000, so the disk hit rate collapses and most requests go to the network. Trading a decode for a network round trip on a mobile link is the wrong direction.

    Correctness: a raw bitmap on disk is tied to a pixel format and a target size, so it is invalidated by a screen density change, a display size change, or a change to the card layout. The encoded bytes are valid for all of those.

    Where the idea is actually right: a small number of images that are expensive to decode and shown at exactly one size on the launch path, for example an onboarding illustration. There, pre-decoding into a compact format is a legitimate cold start optimisation, and the correct place for it is the application binary rather than a disk cache.

    So the answer is no for the general path, with a named exception, and the reason is a 23 times storage ratio rather than an opinion.

### 8g. When not to use this

Almost nobody should write this pipeline from scratch, and saying so is a strong answer rather than a weak one.

| Question | Answer |
|---|---|
| The measurement that justifies a full three tier pipeline | Images on a scrolling surface, or any decode over about 1 megabyte, or an observed background termination rate that correlates with image heavy screens |
| Scale threshold below which it is over-engineering | A screen with one or two static images of known size. Load them, keep a reference, done. No tiers, no prefetch, no eviction policy |
| The cheaper alternative | A maintained image loading library. It already implements keys, tiers, eviction, cancellation and downsampling. Your job in a real project is to configure the sizes from your own budget, not to reimplement it |
| When writing your own is justified | A component or library prompt that explicitly asks for it, or a genuinely unusual requirement such as a custom decoder or an encrypted cache. Otherwise, integrating and tuning beats building |
| Failure mode of over-applying | A prefetch window of 10 cards on a metered connection, which spends the user's data allowance on content they never see. Prefetch depth is derived from scroll velocity and latency, not chosen for comfort |
| The tell in an interview | Naming a library and stopping. The score is in the numbers: the decode arithmetic, the tier sizing from a heap budget, and the cancellation rule. Those transfer whether you wrote the library or configured it |

---

## Module 9. Media out: uploads that survive the real world

**Time: 30 to 40 minutes reading, 20 to 25 minutes practice.**

### 9a. Predict first

Here is how an app uploads a photo, and the network it runs on.

```
user picks a 4 MB photo and taps send
app holds the bytes in memory
app posts the whole file in one request
app drives progress from bytes written to the socket
app retries the whole request twice on failure
app cancels the upload if the user leaves the screen

uplink is 1 megabit per second
the link drops for about 5 seconds twice a minute

Figure a single shot upload on a link that drops twice a minute

```

??? note "Show answer"

    Prediction asked for: the chance one attempt succeeds, the expected bytes sent per successful upload, and what happens when the user force quits.

    **Time for one attempt.** 4 megabytes is 32 megabits, and at 1 megabit per second that is **32 seconds** of continuous transfer.

    **Chance one attempt succeeds.** Two drops per 60 seconds is a rate of 1 drop per 30 seconds. Over a 32 second window the expected number of drops is 32 divided by 30, about **1.07**. Modelling drops as independent arrivals, the chance of zero drops is e to the power of minus 1.07, about **0.34**. So roughly **two thirds of attempts fail**.

    **Overall.** One attempt plus two retries is three tries, so the chance all three fail is 0.66 cubed, about **0.29**. Nearly a third of uploads fail entirely, and the user is told after roughly 96 seconds of waiting.

    **Expected bytes.** Every failure restarts from byte zero, so the expected data sent per success is about 4 megabytes divided by 0.34, roughly **11.8 megabytes**, about three times the file size. On a metered plan the user pays three times over for one photo.

    **Force quit.** The bytes are in memory and the progress is in memory, so everything is lost. There is no record that the user ever tried to send anything, which is the failure they will describe as "the app ate my photo".

### 9b. Resumable chunked upload, and how to size the chunk

A resumable upload is a three phase protocol. Each phase exists for a reason worth naming.

| Phase | Request | What it establishes |
|---|---|---|
| 1 Create | Send metadata: total size, content hash, content type, an idempotency key | The server allocates a session and returns a session identifier. Nothing has been sent yet, so this is cheap to retry |
| 2 Transfer | Send byte ranges against the session | The server acknowledges the highest **contiguous committed offset**, which is the only authority on progress |
| 3 Complete | Ask the server to finalise the session | The server verifies the hash and returns the created resource identifier |
| Resume | Query the session | The server replies with its committed offset. Transfer continues from there |

**The rule that makes the whole thing work.** The server's committed offset is the truth, and the client's recorded offset is a hint. A process can die after the bytes were sent and before the client wrote down that it sent them, so the client must always ask before resuming.

#### Chunk size, derived rather than chosen

Two forces pull in opposite directions.

| Force | Effect of a large chunk | Effect of a small chunk |
|---|---|---|
| Loss of work on failure | A failed 5 megabyte chunk wastes up to 5 megabytes | A failed 256 kilobyte chunk wastes at most 256 kilobytes |
| Per chunk overhead | Few requests, little overhead | One round trip and a set of headers per chunk |

Price both on the 9a network (1 megabit per second, 1 drop per 30 seconds) with a 150 millisecond round trip.

| Chunk size | Transfer time | Chance that chunk survives | Round trip overhead for a 4 MB file |
|---|---|---|---|
| 5 megabytes | 40 seconds | e to the minus 1.33, about **0.26** | 1 chunk, 0.15 seconds, 0.4 percent |
| 1 megabyte | 8 seconds | e to the minus 0.27, about **0.76** | 4 chunks, 0.6 seconds, 1.9 percent |
| 256 kilobytes | 2 seconds | e to the minus 0.067, about **0.94** | 16 chunks, 2.4 seconds, 7.5 percent |

**Expected total bytes sent, which is the number that decides it.** With resumability only the failed chunk is resent, so expected bytes are roughly the file size divided by the per chunk success probability.

- Single shot: 4 divided by 0.34 equals **11.8 megabytes**.
- 1 megabyte chunks: 4 divided by 0.76 equals **5.3 megabytes**.
- 256 kilobyte chunks: 4 divided by 0.94 equals **4.3 megabytes**, but with 7.5 percent overhead added back.

**The rule to carry.** Choose a chunk that takes roughly 5 to 10 seconds to transfer at the measured uplink, clamped to a sensible range such as 256 kilobytes to 8 megabytes. Measure throughput during the upload and resize the next chunk. That single adaptive rule beats any fixed constant, because the same app runs on a 100 megabit per second home connection and a 0.3 megabit per second rural cellular link.

#### The durable queue, because memory is not a place to keep a user's photo

One row per upload, in the local database, written **before** any bytes move.

| Column | Why it must be persisted |
|---|---|
| Local asset reference | The bytes stay where they are. Copying a 200 megabyte video to make a private copy is often unacceptable |
| Idempotency key | Generated at intent creation (module 6), so a relaunch reuses it rather than creating a second upload |
| Session identifier | Without it a relaunch cannot resume and must restart from zero |
| Total size and content hash | Lets the server verify, and lets the client detect that the source changed underneath it |
| Committed offset | A hint only. Always re-verified against the server on resume |
| State, attempt count, next attempt time | The retry policy from module 6, made durable |
| Created at | Drives expiry, so a queue cannot grow without bound |

The state machine, with terminal states that are honest about the different ways an upload ends.

```
+--------+     +-----------+     +-----------+     +----------+
| QUEUED |====>| PREPARING |====>| UPLOADING |====>| FINISHED |
+--------+     | compress  |     | chunks    |     +----------+
                +-----------+     +-----------+
                                    |  |  |
              +---------------------+  |  +-------------------+
              v                        v                      v
        +-----------+           +-------------+        +-------------+
        | UNAVAIL   |           | EXPIRED     |        | FAILED      |
        | source is |           | past the    |        | server said |
        | gone      |           | horizon     |        | no          |
        +-----------+           +-------------+        +-------------+

Figure the upload state machine with three distinct terminal failures

```

Three terminal failures rather than one, because they need different interfaces: source gone means offer to pick again, expired means offer to retry manually, and rejected means show the server's reason.

### 9c. Process death, compression and honest progress

#### Surviving process death and backgrounding

Two mechanisms, used at different times.

| Situation | Mechanism | Why |
|---|---|---|
| App is in the foreground | Upload in your own process | You get fine grained progress, adaptive chunk sizing, and immediate cancellation |
| App is backgrounded or being terminated | Hand the transfer to the operating system's background transfer service | It continues while your process does not exist, and it wakes or relaunches you on completion |

The background service imposes real constraints that are worth naming in a round: the payload usually has to be a file on disk rather than a stream you generate, you get no arbitrary code during the transfer, and the operating system decides when it runs. So the handoff has to write the current chunk to a file and record enough state that a relaunched process can pick up the result.

**Reconciliation on launch**, which is the piece most designs forget. For every non-terminal row: ask the server for the session's committed offset, compare it to the local hint, and continue. Do not assume the local hint is right and do not assume it is wrong.

#### Client side compression, decided by arithmetic

A 12 megapixel photo, 4000 by 3000, roughly 3.5 megabytes as captured.

| Option | Resulting size | Transfer at 1 megabit per second | On device cost |
|---|---|---|---|
| Send the original | 3.5 megabytes | **28 seconds** | 0 |
| Resize to a 2048 pixel long edge, re-encode | 2048 times 1536 equals 3.15 megapixels, at the 0.17 bytes per pixel assumption from module 3, about **535 kilobytes** | **4.3 seconds** | Assume 400 milliseconds of decode, resize and encode on a mid tier device |

Spend 400 milliseconds of processor time to save about 24 seconds of radio time. From module 3, sustained radio use is one of the dominant battery terms, so this is a win on time, on data allowance and on battery at once. It is close to the clearest trade in this whole course.

**When not to compress**, which matters just as much.

| Situation | Why compression is wrong |
|---|---|
| The product promises originals (a photo backup service) | Re-encoding destroys the thing the user is paying for |
| The source is already small, under roughly 200 kilobytes | Re-encoding can make it larger and always loses quality |
| The source is already in an efficient modern codec at a sensible resolution | The saving is small and the quality loss is not |
| The server needs the original for its own processing | You cannot un-compress later |

A workable rule: compress when the predicted output is under about 60 percent of the input and the product does not promise originals. Predict from the pixel arithmetic above rather than by encoding and checking, so you do not pay the processor cost to discover it was not worth paying.

**Video, where the arithmetic is more dramatic.** A 60 second clip captured at 4K and roughly 50 megabits per second is 60 times 50 divided by 8, which is **375 megabytes**. At 1 megabit per second that is 3,000 seconds, about **50 minutes**.

Transcoded to 1080p at 8 megabits per second it is 60 megabytes, about **8 minutes** of transfer, against perhaps 30 to 60 seconds of on device encoding. The transcode is obviously correct. The honest caveat is that on device encoding is heavy enough to be worth deferring until the device is charging if the upload is not urgent.

#### Progress that does not lie

Drive progress from the **committed offset the server acknowledged**, not from bytes written to the socket. Bytes written include bytes buffered in the operating system that may never arrive, which is why a naive progress bar reaches 100 percent and then restarts.

Report progress as the maximum committed offset ever observed, so it is monotone. A progress bar that goes backwards is read as a bug even when the underlying behaviour is correct.

### 9d. Worked example: uploading a photo attached to a post

The prompt: the user picks a photo, writes a caption, and taps post. The post must appear immediately in their own feed.

**Block 1. PURPOSE: decide compression before anything else, because it changes every number downstream.**

The photo is 3.5 megabytes at 4000 by 3000, and the product does not promise originals. Using the compression arithmetic in 9c, resizing to a 2048 pixel long edge yields about 535 kilobytes for about 400 milliseconds of processor time, turning a 28 second transfer into a 4.3 second one.

I decide to compress, and I state the consequence for chunking: a 535 kilobyte payload at 1 megabit per second is about 4.3 seconds of transfer, which is already inside the 5 to 10 second target band, so **this upload is a single chunk** and the chunking machinery only engages for larger media.

**The error most readers make here** is designing the chunking protocol first and then discovering that the typical payload fits in one chunk. Compress first, then size the chunks against what is actually being sent.

!!! note "Say it before you read on"

    Say what would change about the chunking decision if this were a photo backup product that promised originals.

**Block 2. PURPOSE: make the post durable before the user's attention moves on.**

Order, and it is the same order as module 6's like tap for the same reason.

1. Write the post row locally: caption, local asset reference, idempotency key, state Queued.
2. Insert the post into the local feed immediately, marked pending, so it appears in the user's own feed.
3. Dismiss the composer.
4. Only now start preparing and uploading.

The user's post exists from step 1 onward, whatever happens to the process, the network or the app.

**The error most readers make here** is keeping the composer on screen until the upload finishes. That converts a background operation into a modal wait, and it is why users think uploads are slow when the actual constraint is that they were prevented from doing anything else.

**Block 3. PURPOSE: define what happens at each of the four ways this can go wrong.**

| Failure | Detection | Behaviour | What the user sees |
|---|---|---|---|
| Transient network failure | Connect failure or 5xx (module 6 taxonomy) | Resume from the server's committed offset with jittered backoff | Nothing. The pending post stays pending |
| Process killed mid upload | Non-terminal row found at launch | Query the session offset, resume | Nothing, provided the resume happens before they notice |
| Source photo deleted by the user | Asset reference no longer resolves | Terminal state Unavailable | The pending post shows a clear message and an option to pick another photo |
| Server rejects the content | A 4xx that is not retryable | Terminal state Failed with the server's reason | The reason, plus remove or edit options |

Note the deliberate asymmetry: two of these are invisible to the user by design, and two must be visible immediately. Deciding which failures deserve the user's attention is a design decision, not an implementation detail.

!!! note "Say it before you read on"

    Say why the process-kill case must query the server rather than trusting the offset the client last wrote down.

**Block 4. PURPOSE: set the expiry horizon and the progress rule, so the queue cannot rot.**

Expiry: 7 days. After that, an unposted post is more likely to confuse than to please, and the session on the server has probably expired anyway. On expiry the row moves to Expired, the local post is removed from the feed with a notice, and the event is counted.

Progress: driven by the committed offset, monotone, and reported at most a few times a second so that progress updates do not themselves cost frames (module 3's budget is 6.2 milliseconds and a progress update that triggers a layout pass is not free).

**The error most readers make here** is an unbounded queue. Rows accumulate for months, the reconciliation on launch grows linearly, and cold start regresses for a reason nobody can find. Every durable queue needs a horizon.

**Result.** Compression decided by arithmetic, durability before transmission, four failure paths with two of them deliberately invisible, and a bounded queue with honest progress.

### 9e. Fade the scaffold

**Faded example 1.** The prompt is uploading a 200 megabyte video from the share sheet. Blocks 1 to 3 are given. Produce block 4.

- Block 1: transcode on device, using the 9c arithmetic. 200 megabytes at 1 megabit per second is 1,600 seconds, about 27 minutes. Transcoding to a lower bitrate brings it to roughly 35 megabytes and about 5 minutes of transfer, for perhaps 40 seconds of encoding.
- Block 2: durable first. The row exists before the transcode starts, because a transcode is long enough that the process can die during it.
- Block 3: chunking now matters. At 35 megabytes and a target of 5 to 10 seconds per chunk on a 1 megabit per second link, chunks are about 1 megabyte, so roughly 35 chunks, resized adaptively as measured throughput changes.
- Block 4: **you handle backgrounding, expiry and progress.**

??? note "Show answer"

    Backgrounding is the whole answer here, because a 5 minute upload will not finish in the foreground. The design is a handoff: while the app is in the foreground, upload in process with adaptive chunk sizing. On backgrounding, write the remaining work as files and hand the transfer to the operating system's background transfer service, recording the handoff in the row so a relaunched process knows the transfer is not its own.

    The constraint that shapes it: the background service will not run your adaptive sizing logic, so the chunk size at handoff is fixed for the remainder unless the app returns to the foreground. Choose a slightly conservative size at handoff, because you lose the ability to shrink it after a failure.

    Expiry is longer than the photo case: 14 days rather than 7, because a video is a larger investment by the user and re-recording is often impossible. The transcoded intermediate must be deleted on any terminal state, or the cache directory accumulates hundreds of megabytes per abandoned upload, which is a storage complaint that reads as a bug.

    Progress: still the committed offset, but the user also needs progress during the transcode, which is a separate phase with its own percentage. Showing a single bar that covers both phases is honest only if the phases are weighted by their actual duration, so 40 seconds of transcode against 300 seconds of upload means the transcode is about 12 percent of the bar. An unweighted two phase bar that sits at 50 percent for 5 minutes is the common mistake.

**Faded example 2.** The prompt is uploading crash reports and diagnostic logs. Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: compress aggressively, because logs are text and compress by roughly an order of magnitude, and nobody needs the original bytes.
- Block 2: durable, but with a different urgency. The crash report must be written to disk from inside a crashing process, so it is written by a minimal handler with no allocation, and uploaded on the **next** launch rather than the current one.

??? note "Show answer"

    Block 3, failure handling, and it inverts almost every rule from the photo case. Nothing here is user visible, so every failure is silent. There is no Unavailable state, because the source is a file this app wrote. A server rejection is dropped and counted rather than surfaced. The only failure that matters is silent accumulation, and the guard is a hard cap on the on-disk report directory (say 20 megabytes), oldest evicted first.

    Block 4, expiry and scheduling. Expiry is short: a crash report older than 7 days is worth much less than a fresh one, and a diagnostic log older than 48 hours is usually worthless. More importantly, this is the deferrable tier from module 6, so it uploads only on an unmetered network by default and never in low power mode.

    The distinctive rule, and the reason this is not the photo prompt again: the upload must be **rate limited at the source**. A crash loop can generate thousands of reports in minutes, so the client samples (keep the first N per signature per hour) and the server can tell the client to stop with a server side flag. Without both, a bad release turns a crash into a self-inflicted traffic incident on top of the crash.

### 9f. Check yourself

**Q1.** Your uploads use 8 megabyte chunks. Field telemetry shows a median uplink of 0.8 megabits per second and a connection interruption roughly every 45 seconds. Compute what is happening and give the new chunk size.

??? note "Show answer"

    Transfer time per chunk: 8 megabytes is 64 megabits, divided by 0.8 megabits per second equals **80 seconds**.

    Chance a chunk survives: with one interruption per 45 seconds, the expected interruptions in 80 seconds is 80 divided by 45, about 1.78. The chance of zero is e to the minus 1.78, about **0.17**. So roughly five in six chunks fail, and each failure discards up to 8 megabytes.

    Expected bytes per megabyte delivered: 1 divided by 0.17 is about **5.9 times** the file size. Most users will never complete an upload of any size.

    New chunk size, from the 5 to 10 second rule: at 0.8 megabits per second, 8 seconds of transfer is 6.4 megabits, which is **800 kilobytes**. Round to 512 kilobytes to leave margin. Chance of survival becomes e to the minus 5.1 divided by 45, about 0.89, and expected bytes fall to about 1.12 times the file size. The fix is a constant, and it is worth roughly a five times reduction in data sent.

**Q2.** A colleague says resumable upload is unnecessary because the network layer already retries. Give the specific case that defeats retry, and the smallest addition that fixes it.

??? note "Show answer"

    The case: the process is terminated. Retry lives in memory, in the network layer, inside a process the operating system can kill at any moment (module 2's runtime axis). When that happens the retry state is gone, the socket is gone, and the bytes already sent are gone with it. Retry handles a failed request; it does not handle a failed process.

    A second case worth naming: retry restarts from byte zero, so on the 9a network the expected data sent is roughly three times the file size. Retry makes the upload eventually succeed and makes it expensive.

    The smallest addition that fixes both: a durable row containing the idempotency key and the session identifier, plus a resume query on launch. That is a table and one request. You do not need the adaptive chunk sizing, the background handoff or the compression pipeline to get most of the benefit, and saying which part is the minimum is what distinguishes a considered answer.

### 9g. When not to use this

Full resumable upload with a durable queue is correct for user generated media and is too much for many payloads.

| Question | Answer |
|---|---|
| The measurement that justifies it | A payload whose transfer time at the median observed uplink exceeds roughly 10 seconds, or an observed upload failure rate above a few percent, or content the user cannot recreate |
| Scale threshold below which it is over-engineering | Payloads under about 1 megabyte on a typical connection, which transfer in a few seconds. A single request with a deadline and an idempotency key is correct there |
| The cheaper alternative | Durable intent plus idempotency key plus a plain retry. That handles process death and duplicate application, which are the two failures that actually hurt, without any chunking protocol |
| When to skip even the durable row | Genuinely disposable payloads such as an analytics batch that can be regenerated or dropped. Persisting them costs storage and reconciliation time for no user benefit |
| Failure mode of over-applying | A chunking protocol with adaptive sizing for a 40 kilobyte avatar upload, where the session creation round trip costs more than the payload |
| Second failure mode | A durable queue with no expiry horizon. It grows forever, and its launch reconciliation quietly becomes a cold start regression |

---

## Module 10. Push and real-time delivery

**Time: 30 to 40 minutes reading, 15 to 25 minutes practice.**

### 10a. Predict first

Here is the push design of a messaging app.

```
server sends one push per chat message
payload carries   sender name  message text  thread id  unread count
priority          high  for every message
collapse key      none
time to live      28 days
token store       tokens are inserted  never removed
silent push       one per message  to sync state in the background

Figure a push design with six independent defects

```

??? note "Show answer"

    Prediction asked for: name at least four defects and say which one the user notices first.

    | Defect | Consequence |
    |---|---|
    | Message text in the payload | The content passes through and is stored by a third party service, appears on a lock screen, and is truncated by the payload size limit, which is small (a few kilobytes) |
    | Unread count in the payload | It is computed at send time and delivered later, possibly much later given the 28 day time to live, so the badge is wrong the moment two devices are involved |
    | No collapse key | A device offline for an hour comes back and receives 50 separate notifications at once |
    | High priority for everything | Each one wakes the device from a low power state, which is a battery cost, and platforms throttle apps that mark everything urgent |
    | 28 day time to live | A message delivered 3 weeks later is not a late notification, it is a bug. The user has long since read it on another device |
    | Tokens never removed | Dead tokens accumulate forever. See the arithmetic in 10c |

    The one the user notices first is the collapse key defect, because 50 notifications arriving simultaneously is immediate, visible and infuriating. The one that costs the most engineering time to fix later is the payload carrying content, because removing it changes the client, the server and the notification presentation together.

### 10b. What push actually guarantees, which is almost nothing

Push is a best effort hint delivery service operated by the platform. Write the guarantees down honestly, because every design error in this area comes from assuming one it does not have.

| Property | Reality |
|---|---|
| Delivery | Best effort. A message can be dropped and you are not told |
| Ordering | None. Two pushes can arrive in either order |
| Latency | Unbounded. Usually seconds, sometimes much longer on a sleeping device |
| Receipt | You learn that the service accepted it, not that the device got it, unless the app acknowledges from its own code |
| Duplication | Possible. The same logical event can arrive twice |
| Availability to your app | Revocable at any moment by the user turning notifications off, or by the operating system restricting a rarely used app |
| Payload size | Small, on the order of a few kilobytes |

**The consequence, and it is the single most useful sentence in this module.** Push is a hint, not a transport. The pattern that follows is **notify then fetch**: the payload carries an event type and the minimum identifier needed to act, and the client fetches authoritative state from your own API.

Price it, because the extra request looks like a cost until you count properly.

| Approach | Pushes for 50 messages received while offline | Requests | Correctness |
|---|---|---|---|
| Content in the payload | 50 delivered at once | 0 | Wrong counts, truncated text, content on a third party |
| Notify then fetch, no collapse key | 50 delivered at once | 1, if the client coalesces | Correct, but 50 notifications |
| Notify then fetch, collapse key per conversation | 1 per conversation | 1 delta sync | Correct, and the user sees one summary per conversation |

The third row costs one request and buys correct counts, correct ordering, full message text, and no user content sitting on a service you do not run.

```
+--------+  event  +---------+  hint   +--------+  fetch  +-----+
| SERVER |========>| PUSH    |========>| CLIENT |========>| API |
| writes |         | SERVICE |         | wakes  |<========|     |
| message|         | best    |         | and    |  truth  +-----+
+--------+         | effort  |         | fetches|
                   +---------+         +--------+
                                            |
                                            v
                                    +----------------+
                                    | LOCAL STORE    |
                                    | is the truth   |
                                    | shown to user  |
                                    +----------------+

Figure notify then fetch with the local store as the displayed truth

```

### 10c. Tokens, priority, collapse and time to live

#### Token lifecycle

A push token identifies an app installation on a device for the push service. It is not a user identifier and it is not stable.

| Event | Effect on the token |
|---|---|
| App installed and permission granted | A token is issued |
| Platform rotates it | A new token arrives, usually with a callback |
| App reinstalled | New token, old one becomes invalid |
| Device restored from a backup onto new hardware | The restored token is invalid and must not be used |
| App data cleared | New token |
| User revokes notification permission | Token becomes undeliverable |

**The registration record, and the one design rule.** Key the server record by **installation identifier**, not by token. A token rotation then updates a row rather than inserting one. Key by token and every rotation creates a duplicate row, so a single device accumulates entries and receives the same notification several times.

A workable record: installation identifier, user identifier, current token, platform, app version, declared capabilities (module 5), last successful delivery, last app open.

#### Dead token reaping, with the arithmetic

Assume 10 million installs and that **1.5 percent of installations per month** produce a token invalidation through reinstall, restore, data clearing or permission revocation.

!!! note "Why 1.5 percent per month is a reasonable working assumption"

    It is the combined rate of several independent events (device replacement, reinstall, restore, permission change), each individually small, on a consumer app with normal churn. Measure your own rate from delivery failures; the point of the number here is that even a small monthly rate compounds.

- Dead tokens per month: 10,000,000 times 0.015 equals **150,000**.
- After 24 months with no reaping: 150,000 times 24 equals **3.6 million** dead rows.
- As a fraction of the live base: 3.6 million against 10 million live, so about **26 percent of every fan-out is wasted**.
- At one notification per user per day, that is **3.6 million wasted sends per day**, plus the latency cost of a fan-out that is a quarter larger than it needs to be.

**Two reaping mechanisms, both needed.**

1. **Synchronous.** The push service reports per token that a token is unregistered or invalid. Delete or tombstone that row in the same operation that sent the batch. This catches the majority and costs nothing extra.
2. **Time based sweep.** No successful delivery and no app open for 90 days, so mark the installation dormant and stop sending. Keep the row for analytics rather than deleting it, so you can measure how large the dormant population is.

Why 90 days for the sweep: it is long enough that a normal user on holiday or on a second device is not reaped, and short enough that the dormant set does not dominate the fan-out. It is a policy choice, so state it as one and say what you would measure to tune it, namely the rate at which reaped installations later return.

#### Priority

| Priority | What the platform does | Cost | Correct use |
|---|---|---|---|
| High | Wakes the device promptly even from a low power state | Battery, and platforms throttle apps that overuse it | Something the user would consider urgent: an incoming call, a direct message, a security alert |
| Normal | May be delayed and batched until the device is next awake | Low | Everything else: reactions, digests, background sync hints |

**The rule.** High priority requires a user visible notification that a reasonable user would call urgent. Using high priority to drive a silent background sync is the classic abuse, and the penalty is that the platform throttles your app, which removes the mechanism you were relying on.

#### Collapse key and time to live

A **collapse key** tells the service that only the most recent undelivered message with that key needs to be delivered. A **time to live** tells it how long to hold an undelivered message before dropping it.

Collapse key granularity is a product decision. Use the level the user perceives.

| Granularity | Effect after an hour offline | Verdict |
|---|---|---|
| None | 50 notifications | Wrong |
| One key for the whole app | 1 notification, and the user has no idea which conversations were active | Too coarse |
| One key per conversation | One notification per active conversation, each showing the latest message | Right for chat |

Keep the number of distinct collapse keys modest, because services impose limits on how many pending keys they will track per device, and exceeding the limit degrades in ways you cannot observe from the client.

**Time to live, chosen by a single question:** after how long would delivering this message be a bug?

| Content | Time to live | Reasoning |
|---|---|---|
| Incoming call invitation | 30 to 60 seconds | A phone that rings for a call that ended ten minutes ago is a defect, not a late notification |
| Chat message | 4 to 24 hours | Still meaningful tomorrow, and the next foreground fetch will catch it anyway |
| Silent sync hint | 0, meaning deliver now or drop | It is a hint. The next foreground reconciles |
| Promotional message | Hours, never days | A promotion delivered after the offer ends is worse than no promotion |

#### Silent push, and the rule that makes it safe

A silent push wakes the app briefly with no user visible notification. Platforms budget and throttle these: the number per app per period is capped, and an app that receives them and does nothing useful, or whose user rarely opens it, gets throttled further.

**The design rule that makes silent push safe to use.** Every silent push must be redundant. The app must reach exactly the same state on its next foreground without it. If missing a silent push causes a correctness bug, the design is wrong, because the platform is entitled to drop every one of them.

Good uses, all of which satisfy that rule: warm a cache so the app opens fresh, invalidate a cache entry, and trigger an outbox flush that would have happened on foreground anyway.

#### Badge and count consistency

Counts must be sent as **absolute values computed by the server**, never as increments.

Derive why. Assume 5 percent of pushes are not delivered.

- With increments, after 100 events the expected error is 5 events, and the error is permanent because there is nothing to correct it against. Errors accumulate over weeks.
- With absolute values, the error is bounded by the staleness of the last delivered push, and the next successful delivery corrects it completely.

Even absolute values are stale at delivery, so the authoritative count comes from your own sync response on foreground. The badge is a best effort approximation that self corrects, which is the most that push semantics allow.

#### The transport decision, in one table

| Transport | Latency | Battery | Works with the app backgrounded | Ordering | Use when |
|---|---|---|---|---|---|
| Platform push | Seconds, unbounded tail | Very low | Yes, this is the point | None | Anything that can tolerate seconds and must reach a sleeping device |
| Polling on an interval | Half the interval on average | Poor, from the radio tail in module 3 | Only via scheduled background work | Not applicable | A last resort, or slow moving data on a long interval |
| Server sent events | Sub second | Moderate, holds a connection | No | Per stream | Foreground only, server to client, and you want plain HTTP |
| Web socket | Sub second | Moderate to poor | No | Per connection | Foreground, bidirectional, chat and presence |
| Lightweight publish and subscribe protocol | Sub second | Better than a plain socket, designed for constrained devices | No, unless the platform keeps it alive | Per topic, per quality of service level | Many topics, constrained devices, or an existing broker |

**The combination that is almost always right on a phone:** platform push when backgrounded, a live connection only while in the foreground, and a reconciling fetch on every foreground transition regardless of what arrived.

### 10d. Worked example: delivery for a chat client

The prompt: messages must feel instant when the app is open, must arrive when it is closed, and counts must be right across three devices.

**Block 1. PURPOSE: decide what the push carries, because that decision constrains everything else.**

Notify then fetch. The payload carries the event type, the conversation identifier, a monotonically increasing server sequence number, and a short localised preview string produced by the server for the notification text only.

Why a preview string rather than the message text: the notification must be readable before the fetch completes, so something has to be displayable immediately. Making it a server produced display string rather than the raw content means it can be truncated, localised and redacted according to the user's own lock screen preference, which the server knows and the payload does not have room to reason about.

**The error most readers make here** is either putting the whole message in the payload (which fails on size, privacy and staleness) or putting nothing in it (which means a notification cannot be shown until a network fetch succeeds, and that fetch may fail).

!!! note "Say it before you read on"

    Say why the sequence number is in the payload when the client is going to fetch anyway.

**Block 2. PURPOSE: get the token lifecycle right, since it is the part that silently rots.**

Register by installation identifier, not by token. Update on every rotation callback and on every app launch, because a launch is a free opportunity to correct drift.

Reap on both mechanisms from 10c: synchronously on an invalid token result, and by a 90 day dormancy sweep. Using the 10c arithmetic, doing neither would leave about 26 percent of the fan-out wasted within two years.

Multi device: the fan-out targets every live installation for the user, so the send path is a query over installations, not a single token lookup. That is exactly why keying by installation matters.

**The error most readers make here** is treating the token as the user's address. A user has several devices, each device's token changes over time, and a token that worked yesterday can be invalid today.

**Block 3. PURPOSE: set priority, collapse and time to live per message class rather than globally.**

| Message class | Priority | Collapse key | Time to live |
|---|---|---|---|
| Direct message | High | Conversation identifier | 12 hours |
| Group message in a muted conversation | Normal | Conversation identifier | 12 hours |
| Reaction to a message | Normal | Conversation identifier | 4 hours |
| Typing indicator | Never sent by push at all | Not applicable | Not applicable |
| Incoming call | High | Call identifier | 45 seconds |

The typing indicator row is the interesting one. It belongs to the foreground connection only, because a typing indicator delivered by push is both useless (it will arrive after the typing stopped) and expensive (it would wake the device for nothing).

!!! note "Say it before you read on"

    Say why the incoming call time to live is 45 seconds and not the 12 hours a message gets.

**Block 4. PURPOSE: make counts correct across three devices, given that push guarantees nothing.**

Three rules, and the third is the one that makes it actually work.

1. The badge value is an absolute count computed by the server at send time.
2. On every foreground transition the client fetches the authoritative unread state and overwrites the badge, whatever push said.
3. Read state is a synchronised value, not a local one. Reading on the phone must clear the count on the tablet, so the read receipt is a write that goes through the same outbox as a message (module 6), and other devices discover it through the same notify then fetch path.

Silent push carries the read state change to the other devices, and by the 10c rule it is redundant: if the silent push is dropped, the other device corrects itself on its next foreground.

**The error most readers make here** is treating read state as a local user interface concern. It is shared state across devices, and it is the single most visible correctness bug in multi device messaging.

**Result.** The push carries a hint plus just enough to render a notification, tokens are keyed and reaped so the fan-out stays honest, delivery parameters vary by message class, and counts converge through a fetch rather than depending on delivery.

### 10e. Fade the scaffold

**Faded example 1.** The prompt is order status for a food delivery app: confirmed, being prepared, on the way, arriving now, delivered. Blocks 1 to 3 are given. Produce block 4.

- Block 1: notify then fetch, with the payload carrying the order identifier, the new status, and a status timestamp. The status is included because the notification text depends on it and must render offline.
- Block 2: token lifecycle as in 10d, with one addition: an order is short lived, so a failed delivery during an active order is worth escalating to a fallback channel rather than merely retrying.
- Block 3: priority is high for arriving now and delivered, normal for confirmed and being prepared. Collapse key is the order identifier for all of them. Time to live is 5 minutes for arriving now, 1 hour for the rest.
- Block 4: **you handle out of order delivery and the interface consequence.**

??? note "Show answer"

    Push has no ordering (10b), so "on the way" can arrive after "arriving now". The status timestamp in the payload is what makes this solvable: the client keeps the highest timestamp it has seen for the order and **discards any push with an older one**. A status can only move forward.

    The collapse key makes this mostly moot for delivered notifications, since collapsing keeps the latest, but collapsing only applies to messages still undelivered at the service. Two pushes both in flight can still arrive out of order, so the client side ordering guard is required rather than optional.

    The interface consequence, and the reason this prompt is not the chat prompt again: an order status is a **monotone state machine** displayed as progress, so an out of order update would visibly move the progress backwards. The design must make backwards movement unrepresentable, which is the same lesson as module 4's sealed state applied to a different problem.

    One case the guard must allow through: cancellation, which is a legitimate exit from any state. So the rule is monotone within the happy path plus a terminal cancellation that can arrive at any point, not simply "timestamps must increase".

**Faded example 2.** The prompt is a breaking news alert sent to 20 million devices at once. Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: the content is public, short, and identical for everyone, so this is the one case where putting the headline in the payload is correct rather than lazy.
- Block 2: tokens as before, and the dormancy sweep matters much more here, since a 26 percent wasted fan-out on a 20 million device send is over 5 million pointless deliveries per alert.

??? note "Show answer"

    Block 3, priority, collapse and time to live: high priority is defensible for genuine breaking news and indefensible for editorial pushes, so the class must be enforced at the sending tool rather than left to whoever writes the alert. Collapse key is a single news key, so a device that was offline gets the latest headline and not six stale ones. Time to live is 30 minutes, because a breaking alert delivered two hours late is a bug by the 10c test.

    Block 4, the distinctive problem: **the fetch stampede**. If every device performs notify then fetch, 20 million devices fetch within the delivery window. If delivery spreads over 60 seconds, that is over 300,000 requests per second aimed at whatever endpoint the alert points to.

    Three mitigations, in order of value.

    | Mitigation | Effect |
    |---|---|
    | Put the content in the payload so no fetch is needed at all | Removes the stampede entirely, and is available here only because the content is public and small |
    | If a fetch is needed for the article body, delay it by a jitter derived from a hash of the installation identifier, spread over several minutes | Turns a 60 second spike into a multi minute plateau, deterministically and with no coordination |
    | Serve the article from a cache tier with a content addressed immutable URL | The fan-out never reaches an origin at all |

    The most common weak answer is to design the push carefully and never notice that the client behaviour it triggers is a self inflicted load test on your own infrastructure.

### 10f. Check yourself

**Q1.** Your app has 5 million installs and no dead token reaping after 18 months, with an invalidation rate of 1.5 percent of installs per month. Quantify the waste, then say which of the two reaping mechanisms you would ship first and why.

??? note "Show answer"

    Dead tokens per month: 5,000,000 times 0.015 equals 75,000. Over 18 months that is 75,000 times 18 equals **1.35 million** dead rows against 5 million live, so about **21 percent** of every fan-out is wasted. At one notification per user per day that is 1.35 million pointless send attempts a day.

    Ship the synchronous mechanism first. It requires no new infrastructure (the invalid token result already comes back in the send response), it is a delete on a row you already have, and it catches the majority of cases: reinstall, data clear and permission revocation all surface as invalid token results. The dormancy sweep needs a job, a policy decision on the threshold, and a definition of last app open, and it catches the smaller residual population of devices that are simply gone.

    One caution worth naming: process invalid token results carefully against rotation. A token that was replaced by a rotation you have already recorded must not cause you to delete the installation row, or you unregister a device that is perfectly healthy.

**Q2.** A colleague proposes using a silent push after every server side data change to keep the client's cache warm, at roughly 40 per user per day. Evaluate it.

??? note "Show answer"

    Two problems, one of policy and one of design.

    Policy: silent pushes are budgeted and throttled by the platform, and 40 a day per user is well into the territory where throttling is likely. The consequence is not that 40 becomes 30; it is that the platform may deprioritise your app's silent pushes generally, which damages the uses that actually mattered.

    Design: the proposal violates the redundancy rule from 10c. If the cache is only warm because a silent push arrived, then a user whose pushes were throttled gets a systematically worse experience, and that degradation is invisible to you because you never learn that the push was dropped.

    What I would do instead: coalesce. One silent push at most every 15 minutes, carrying a change token rather than a change, and only when there is actually something to fetch. That is at most 96 a day in theory and far fewer in practice, and each one triggers a delta sync that would have been correct anyway on the next foreground.

    Then measure the thing that matters: the proportion of foregrounds that find the cache already fresh, with and without the silent push. If that number does not move, the mechanism is not paying for itself.

### 10g. When not to use this

Push is cheap to add and expensive to get right, and there are cases where the right answer is less of it.

| Question | Answer |
|---|---|
| The measurement that justifies push at all | The user needs to know something while your app is not running, and the value of knowing it decays in minutes to hours. If it decays in days, an in-app surface is enough |
| Scale threshold below which the full apparatus is over-engineering | An app with one notification type and a few thousand installs. Token reaping arithmetic at that size is a rounding error, and one priority and one time to live is fine |
| The cheaper alternative | Notify then fetch with a single collapse key and a sensible time to live. That is perhaps a tenth of the work and removes the majority of the defects listed in 10a |
| When to use a live connection instead | Only while the app is in the foreground and the interaction genuinely needs sub second latency. Holding a connection in the background is both restricted by the platform and expensive by module 3's radio arithmetic |
| When to use polling instead | Rarely. A 60 second background poll costs roughly 8.6 times the radio time of the equivalent coalesced work (module 3), and it is worse in latency too. Polling is defensible only on a long interval for slow data with no push entitlement |
| Failure mode of over-applying | Sending a push for every event because the mechanism exists. Notification fatigue causes users to disable notifications entirely, and that is irreversible in practice. The correct measurement is not delivery rate, it is the rate at which notifications are disabled |

---

## Module 12. Storage and caching tiers on device

**Time: 45 to 55 minutes reading, 30 to 40 minutes practice.**

Every mobile app has four or five storage tiers whether anyone designed them or
not. This module makes the choice explicit, sizes each tier from a device number
rather than a guess, and answers the question interviewers use to separate
levels: what exactly survives, and what exactly does not.

### 12a. Predict first

Read this artefact and commit to an answer before continuing.

```
+---------------------------------------------------------------+
| STORAGE AUDIT of a shipping feed app                          |
|   key-value prefs  refresh_token, user_id, and one cached     |
|                    feed JSON blob of 180 KB                   |
|   cache directory  decoded image files, 640 MB, no cap        |
|   app data dir     feed.sqlite, 40 MB, included in backups    |
|   memory cache     up to 100 decoded full-screen bitmaps      |
| Device facts                                                  |
|   heap ceiling for this process  256 MB                       |
|   screen 1080 x 2340 pixels, 4 bytes per pixel                |
+---------------------------------------------------------------+
Caption: four storage tiers as actually configured, with the two
device numbers that decide whether the memory tier is legal.
```

**Question.** Compute how many images the user can scroll past before the
process is killed, and separately name the tier that leaks a secret onto a
different device.

??? note "Show answer"

    **The kill.** One full-screen bitmap is 1080 x 2340 x 4 = 10,108,800 bytes,
    so 10.1 MB. The heap ceiling is 256 MB, so 256 / 10.1 = 25.3 bitmaps fills
    it. The cache is configured for 100, meaning it will try to hold 1,010 MB,
    about four times the ceiling. The process dies after roughly 25 full-bleed
    images, which on a feed is 25 scrolls, so under a minute of use.

    The fix is not a smaller count. It is decoding to the display size rather
    than the source size. A cell that is 1080 x 600 needs 1080 x 600 x 4 = 2.59
    MB, so the same 100 entries need 259 MB, which is still too many, but at 30
    entries it is 78 MB and defensible. Count-based limits on bitmaps are a trap:
    the limit must be in bytes.

    **The leak.** The refresh token sits in plain key-value preferences, which
    live in the app data directory and are therefore inside the device backup.
    Restore that backup onto another device and the token travels with it. A
    long-lived credential must be in the platform key store, marked as
    non-migratable, and excluded from backup.

    **A bonus you may have caught.** The 180 KB JSON blob in preferences is
    loaded synchronously and wholly on first access on both major platforms, so
    every cold start parses 180 KB before it can read `user_id`. Preferences are
    for scalars.

**The tier table, which is the module in one screen.**

| Tier | Read latency | Survives process death | Survives OS storage pressure | In device backup | Right contents |
|---|---|---|---|---|---|
| In-memory cache | Nanoseconds to microseconds | No | Not applicable | No | Decoded objects you can rebuild |
| Key-value preferences | Sub-millisecond after a whole-file load | Yes | Yes | Yes, by default | Scalars and flags, kilobytes total |
| Embedded database | 0.1 to 5 ms per query | Yes | Yes | Configurable | Queryable structured data |
| Files in the cache directory | Milliseconds | Yes | No, the OS may delete them | No | Anything refetchable |
| Files in the app data directory | Milliseconds | Yes | Yes | Configurable | User-authored data not yet synced |
| Platform key store | Milliseconds, includes a crypto operation | Yes | Yes | No, and it should stay that way | Keys, tokens, credentials |

**One rule that resolves most arguments: nothing on the device is durable.** The
OS may clear caches, the user may clear data, a restore may or may not carry a
tier, and reinstall behaviour differs by platform and changes between versions.
Design so the only durable tier is the server, and treat every local tier as an
optimisation that can vanish between two launches. Then reinstall behaviour
becomes a detail rather than a correctness dependency.

**Classify data before choosing a tier.** Four durability classes cover almost
everything, and the class determines the tier mechanically.

| Class | Definition | Tier | On loss |
|---|---|---|---|
| A, reconstructible | The server can regenerate it | Cache directory, capped | Refetch, show a placeholder |
| B, user-authored and unsynced | Exists only here until the outbox drains | App data directory, backed up, plus the outbox | Real data loss. This class is why backups matter |
| C, secret | A credential or key | Platform key store, excluded from backup | Re-authenticate |
| D, ephemeral interface state | Scroll position, expanded rows, draft text | Memory, plus the platform's saved-state mechanism | The user loses their place, which is a quality bug |

Class D deserves a sentence because candidates skip it. Your process can be
killed while backgrounded and recreated on the same screen. If interface state
lives only in memory, the user returns to the top of the feed with a cleared
form, and the crash reporter shows nothing because there was no crash.

**Sizing the memory tier from the frame budget, not from a round number.**

- Cell image at display size 1080 x 600 x 4 bytes = 2.59 MB
- Heap ceiling assumed 256 MB for a mid-tier device, which is why we design for
  it rather than for the flagship in your pocket
- Interface, framework and application objects, assume they take half the heap,
  leaving 128 MB
- Give the bitmap cache at most a quarter of what remains: 32 MB
- 32 / 2.59 = 12 cells, which is roughly two screens of scroll history

Twelve is a small number, and that is the honest result. It says the memory tier
is a scroll-reversal optimisation, not a browsing history, and that the disk tier
is what makes scrolling back feel instant.

**Sizing the disk tier from the working set.**

- Images viewed per session, assumed from scroll analytics: 200
- Encoded bytes per image at the delivered resolution, assumed: 150 KB
- One session's working set: 200 x 150 KB = 30 MB
- Cap at two sessions, so 60 MB, because re-viewing content older than the
  previous session is rare on a feed

Then measure the hit-rate curve and move the cap to the knee. A cap chosen from
a curve is defensible in a design review. A cap of "512 MB because that seemed
fine" is the thing an interviewer probes.

**Eviction, compared on what each one actually optimises.**

| Policy | Optimises | Fails when | Cost |
|---|---|---|---|
| Least recently used | Temporal locality, which is what scrolling produces | A single long scroll evicts everything the user is about to scroll back to | One list operation per access |
| Least frequently used | Repeatedly viewed items | New items never accumulate frequency, so fresh content is evicted immediately | A counter per entry, plus ageing |
| Size aware, cost per byte | Total refetch cost rather than entry count | Needs a refetch-cost estimate you probably do not have | A comparator over two fields |
| Time to live only | Freshness | Says nothing about space, so it is not an eviction policy at all | Trivial |

The working answer is two caps and two policies: least recently used for space,
and a per-entry validator for freshness. They answer different questions, and
conflating them produces caches that serve stale data and evict fresh data at
the same time.

**Embedded database facts that change a design.**

- Batch writes in one transaction. Assume a durable commit costs 2 ms. One
  thousand individual inserts is 1,000 commits, so about 2 seconds. The same
  thousand inserts inside one transaction is one commit, so about 2 ms. This is
  a thousandfold difference and it is one line of code.
- Every index you add is paid on every write to that table. Add indexes from
  measured query plans, not from intuition.
- Never migrate a cache. If a schema change affects a table holding class A
  data, drop the table and refetch. Migration code is the most-bug-prone code in
  a client, and writing it for data the server already holds is wasted risk.
- Whole-database encryption is paid on every page read, so measure your query
  latency with it on before adopting it. Encrypting only the sensitive columns is
  often the better trade, and it is a decision, not an omission.

```
+---------------------------------------------------------------+
| MEMORY CACHE      capped in bytes, dies with the process      |
+---------------------------------------------------------------+
              |  miss                          ^ promote
              v                                |
+---------------------------------------------------------------+
| DISK CACHE DIR    capped, OS may evict, refetchable only      |
+---------------------------------------------------------------+
              |  miss
              v
+---------------------------------------------------------------+
| APP DATA DIR      user-authored, backed up, never auto-evicted|
+---------------------------------------------------------------+
              |  secrets only
              v
+---------------------------------------------------------------+
| KEY STORE         keys and tokens, excluded from backup       |
+---------------------------------------------------------------+
Caption: four on-device tiers ordered by durability. A miss falls
to the tier below, and only the bottom two hold anything whose
loss the user would notice.
```

### 12d. Worked example

**The prompt.** "Plan the on-device storage for a feed app that also lets people
write and post. It must open to content with no network."

Inputs stated first:

| Input | Value | Status |
|---|---|---|
| Target device heap ceiling | 256 MB | ASSUMED, a mid-tier device from a few years ago, because that is where the process kills happen |
| Screen | 1080 x 2340 | GIVEN |
| Feed items cached for offline open | 200 | DERIVED below |
| Images viewed per session | 200 | ASSUMED, from scroll depth analytics |
| Encoded image bytes | 150 KB | ASSUMED, at the delivered resolution for this screen width |
| Drafts held per user | up to 5 | ASSUMED, a composer where drafts are rare but precious |

#### Block A. Purpose: assign every piece of data a durability class

I write the class table before I name a single storage technology, because the
class decides the technology.

| Data | Class | Tier | Justification in one line |
|---|---|---|---|
| Feed item metadata | A | Embedded database in the cache directory | The server can regenerate all of it |
| Decoded images | A | Memory, then cache directory files | Pure optimisation, refetchable |
| Draft posts | B | App data directory plus outbox | Exists nowhere else until it is sent |
| Refresh token | C | Key store, excluded from backup | A credential, and long lived |
| Scroll position, expanded state | D | Memory plus saved instance state | Cheap to lose, but visibly annoying |
| Feature flag values | A with a compiled default | Preferences, small scalars | Must have a value before any network answers |

I say the interesting consequence out loud: the feed database goes in the cache
directory, not the app data directory. It can be evicted by the OS under storage
pressure and that is correct, because the alternative is an app that consumes
gigabytes of a user's phone to hold content the server already has.

**The error most readers make here.** Putting everything in one database in the
app data directory. That mixes class A and class B, so the cache is backed up
(wasting the user's backup quota on refetchable data) and the drafts inherit the
cache's eviction and migration behaviour.

!!! tip "Self-explanation prompt"

    Before Block B, say what breaks if drafts are stored in the cache directory
    with the feed, and describe the support ticket the user files.

#### Block B. Purpose: size the memory tier so the process survives

From the device facts: at 1080 x 600 display size and 4 bytes per pixel, one
decoded cell image is 2.59 MB. I budget 32 MB, derived earlier as a quarter of
the half of the heap that is not already interface objects, giving 12 entries.

I then check that 12 is enough for the actual interaction. During a fling the
list binds maybe 3 cells per frame and the user reverses direction within a
screen or two. Twelve entries covers roughly two screens, so scroll reversal
hits memory and longer journeys hit disk. That is the intended behaviour, and I
state it as intended rather than letting it look like an accident.

Two hard rules I attach:

- The cap is in bytes, never in entries, because bitmap size varies by cell type
  and a count-based cap silently becomes a 10x memory swing.
- On a memory-pressure callback, the memory tier is cleared entirely and first,
  before anything else is dropped, because it is the only tier whose loss costs
  nothing but a disk read.

**The error most readers make here.** Sizing the memory cache from the source
image dimensions. A 4000 x 3000 source decodes to 48 MB. Decoding at source size
and then scaling for display is the single most common cause of client
out-of-memory kills, and it is invisible on a developer's flagship device.

#### Block C. Purpose: size and cap the disk tier from the working set

Two separate caches, because they have different lifetimes:

| Cache | Contents | Cap | Eviction |
|---|---|---|---|
| Image files | Encoded bytes at delivered resolution | 60 MB | Least recently used, plus a 30 day maximum age |
| Feed database | 200 items of metadata | 200 items, roughly 400 KB at an assumed 2 KB per item | Trim to 200 newest on each successful refresh |

The 60 MB comes from the working set arithmetic: one session is 200 x 150 KB =
30 MB, and two sessions of history is 60 MB. The 200 items comes from the
offline requirement: opening to content means the last two screens plus enough
to scroll for a minute, and 200 items at an assumed 8 items per screen is 25
screens, which is generous.

I also state what I will measure to revise these: cache hit rate against cap, at
30, 60 and 120 MB. If hit rate at 120 MB is within 2 points of 60 MB, the extra
60 MB of the user's phone buys nothing and I keep the smaller cap.

**The error most readers make here.** Letting the image cache grow without a
cap because "the OS will clear it". The OS clears it at its convenience, usually
when the device is already nearly full, and until then your app is the one
showing up at the top of the storage settings screen. Users uninstall for this.

#### Block D. Purpose: decide the secret tier and the backup contract

Three decisions, each with the reason attached:

- The refresh token goes in the key store, flagged so it does not migrate to a
  restored device. A credential that survives a restore turns a stolen backup
  into a session.
- The short-lived access token lives in memory only. It expires in minutes, so
  persisting it buys nothing and adds a place it can leak from.
- Backup inclusion is explicit per directory. Class B data (drafts) is backed
  up. Class A data (feed cache, images) is excluded. Class C never.

I say the number that makes this concrete: excluding a 60 MB image cache from
backup saves the user 60 MB of backup quota per device, per backup, to protect
data the server can send again in seconds.

**The error most readers make here.** Treating "encrypted" as a property of the
file rather than of the key. If the encryption key sits next to the encrypted
file in the same backed-up directory, you have added a compute cost and no
security. The key store exists precisely so the key lives somewhere the file
does not.

#### Block E. Purpose: define the three destructive events before someone else does

| Event | What must happen | Why |
|---|---|---|
| Log out | Clear key store entries, drop the feed database, clear image caches, keep nothing user-identifying. Drafts are the hard case: prompt or upload before clearing | The next user of this device must not see the previous user's content |
| User clears app data | The app must cold start into a signed-out state with no crash | This path is almost never tested and it is the top source of "app won't open" reviews |
| Reinstall | Assume nothing survives, including the key store, regardless of what the platform documentation implies today | Behaviour differs by platform and version. An app that depends on it breaks on an OS update you do not control |

The draft case is the one worth arguing in an interview. Silently deleting
unsynced user writing on logout is data loss caused by a design decision.
Uploading it first, or blocking logout behind a clear prompt, is a product
decision that an engineer should raise, and raising it is a senior signal.

**The error most readers make here.** Implementing logout as "delete the token".
The session ends, the cached content does not, and the next sign-in on that
device shows a different user's feed for the seconds before a refresh lands.

### 12e. Fade the scaffold

**Fade 1: offline map regions.** Users download a city region for offline use.
Tiles are large, immutable once downloaded, and a region can be hundreds of
megabytes. Same five-block shape, last block blank.

- Block A, durability classes: downloaded tiles are class B even though the
  server can regenerate them, because the user explicitly asked for them and
  re-downloading a 400 MB region on a metered connection is a real cost. Style
  and search caches remain class A.
- Block B, memory tier: hold decoded tiles for the visible viewport plus one
  ring. At an assumed 256 x 256 tile and 4 bytes per pixel, one tile is 262 KB,
  and a viewport plus ring of 25 tiles is 6.6 MB. Cap in bytes, clear on memory
  warning.
- Block C, disk tier: regions go in the app data directory, excluded from
  backup because they are huge and refetchable, with per-region accounting so
  the user can see and delete each region by name.
- Block D, secrets and backup: no secrets beyond the API credential. Exclude
  every region from backup. Store the region manifest, not the tiles, in a small
  database so the app can list regions before touching the tile store.
- Block E, destructive events: **you write this block.**

??? note "Show answer"

    Log out: keep the regions. They are not user-identifying content and
    re-downloading hundreds of megabytes because someone switched accounts is a
    worse outcome than a shared tile cache. Clear only search history and any
    saved places, which are user data.

    User clears app data: every region is gone at once, potentially several
    gigabytes, and the app must detect this on next launch rather than crashing
    when it opens a missing tile file. The rule is that the manifest and the
    tiles must be validated against each other at startup, cheaply: check the
    region directory exists and its file count matches the manifest, and if not,
    mark the region as not downloaded rather than half present.

    Reinstall: assume everything is gone. The important design consequence is
    that the download must be resumable and the manifest must be reconstructible
    from the server, so a user who reinstalls can re-download only what they had
    rather than choosing regions again from scratch.

    The one extra event this design has that the feed did not: the OS reporting
    low storage. Because regions are class B, the app must never silently delete
    them. It should surface a screen listing regions by size and last use and let
    the user choose. Deleting a user's explicit download to save space is the
    kind of decision that generates one-star reviews.

**Fade 2: a podcast client with offline episode downloads.** Episodes are large
audio files, users subscribe to shows, and playback position must survive
everything. Last two blocks blank.

- Block A, durability classes: episode audio is class B (explicitly downloaded).
  Show and episode metadata is class A. Playback position is class B and tiny,
  and it is the single most emotionally important byte in the product.
- Block B, memory tier: almost nothing. Audio streams from disk, so the memory
  tier holds artwork only, capped in bytes at an assumed 8 MB, which at 1080 x
  1080 x 4 = 4.4 MB per full-size image means artwork must be decoded at display
  size or the cap holds fewer than two images.
- Block C, disk tier: **you write this block.**
- Block D, secrets and backup: **you write this block.**

??? note "Show answer"

    Block C, disk tier. Two stores with different rules. Episode audio goes in
    the app data directory, excluded from backup, with a per-show quota and an
    automatic cleanup policy the user configures: delete after played, or keep
    the newest N per show.

    Derive the default rather than picking one. At an assumed 45 MB per
    hour-long episode and a user subscribed to 10 shows publishing weekly,
    keeping 3 per show is 30 x 45 MB = 1.35 GB, which is too much to impose by
    default. Keeping 1 per show is 450 MB, which is defensible with an obvious
    control to raise it.

    Metadata goes in a small embedded database in the cache directory, trimmed to
    an assumed 100 episodes per subscribed show. It is class A, so it is never
    migrated: on a schema change, drop and refetch.

    Block D, secrets and backup. The subscription list and playback positions are
    the only things the user would grieve, and both are tiny: 10 shows and an
    assumed 500 tracked positions at 24 bytes each is under 15 KB. So they sync
    to the server on every change and are also included in device backup, which
    is the one case in this course where belt and braces is cheap enough to be
    obviously right.

    Audio files are excluded from backup, without exception. Including 1.35 GB of
    refetchable audio in a user's cloud backup quota is a hostile act, and it is
    the kind of default that gets an app singled out in storage settings.

    Playback position deserves one more line: write it on a timer, not only on
    pause. If the process is killed during playback, an unwritten position means
    the user restarts a 60 minute episode from the beginning. An assumed 10
    second write interval costs 6 writes per minute of tiny rows and bounds the
    loss at 10 seconds.

### 12f. Check yourself

**Q1.** Explain, with a number, why a bitmap memory cache must be capped in
bytes rather than in entries, using a screen of 1080 x 2340 and a mix of
full-bleed cells and 120 pixel tall thumbnails.

??? note "Show answer"

    A full-bleed cell decoded at 1080 x 2340 x 4 is 10.1 MB. A thumbnail at 1080
    x 120 x 4 is 0.52 MB. The ratio is about 19 to 1. A cache capped at 30
    entries therefore holds anywhere between 15.6 MB and 303 MB depending purely
    on what the user happened to scroll past.

    That means an entry cap does not bound memory at all. It bounds a proxy for
    memory that varies by a factor of nineteen on this screen, and the worst case
    is the one where the user is most engaged. A byte cap of 32 MB holds 3
    full-bleed cells or 61 thumbnails, and in both cases the process survives.

**Q2.** Your feed database lives in the cache directory. A teammate reports that
after an OS storage cleanup, the app opens to an empty feed instead of the
user's cached content, and wants to move the database to the app data directory.
Give the argument for and against, and state what you would actually do.

??? note "Show answer"

    For the move: the offline-open requirement is a product promise, and a tier
    the OS can delete cannot back a promise. Moving it makes the promise real.

    Against: the database is class A, entirely reconstructible, and moving it to
    the app data directory means the app now holds content the server already has
    in a tier that only grows and is never reclaimed. At 200 items and an assumed
    2 KB each it is only 400 KB, but the same argument will be used next quarter
    for the 60 MB image cache, and then the app is a storage hog.

    What I would do: move only the metadata database, which is 400 KB, and keep
    images in the cache directory. Then offline open shows text and layout
    immediately with placeholders where images were evicted, which satisfies the
    product promise at 400 KB rather than 60 MB.

    I would also add the measurement that settles the question: how often does
    the OS actually clear our cache directory, per user per month. If the answer
    is under an assumed 1 percent, the whole conversation was about a rare event
    and the cheap fix is correct.

### 12g. When not to use this

**An embedded relational database.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You have queries with predicates, sorts or joins, AND the collection exceeds a few thousand rows, AND you have measured that filtering in memory misses the frame budget |
| Scale threshold below which it is over-engineering | Under roughly 1,000 rows with key-only access, a single serialised file loaded into a map is faster, simpler, and has no migration surface. The whole file at an assumed 400 bytes per row is 400 KB, which parses in a few milliseconds |
| Cheaper alternative | One file per collection, written atomically, held in memory while the screen is open. You lose partial updates and gain a system nobody has to migrate |

**A three-tier cache (memory, disk, network).**

| Question | Answer |
|---|---|
| Measurement that justifies it | Measured repeat-access rate on the same objects within a session above roughly 20 percent, AND a decode or parse cost above 1 ms per object, which is what makes the memory tier worth its complexity |
| Scale threshold | For objects that are cheap to rebuild from disk (small JSON, a few hundred bytes), the memory tier saves microseconds and costs you an invalidation bug. Two tiers is the right answer more often than people admit |
| Cheaper alternative | Disk plus the platform HTTP cache, with correct validators on the server. You get revalidation, storage limits and eviction from code you did not write and do not maintain |

**Whole-database encryption at rest.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The database holds class C or regulated data, AND you have measured p95 query latency with encryption enabled against your frame budget rather than assuming it is free |
| Scale threshold | If the sensitive data is a handful of fields, encrypting those columns with a key-store key costs a fraction of the read path and gives the same protection for the data that matters |
| Cheaper alternative | Do not store it. A design that keeps the sensitive value on the server and holds only a reference is stronger than any on-device encryption, because the attacker's best case is an identifier |

**A custom eviction policy.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You have a hit-rate curve against cap for at least two policies on real traffic, and the custom policy beats least recently used by a margin you can state |
| Scale threshold | Below roughly a 100 MB cache, the difference between policies is worth a few percent of hit rate, which is worth less than the bugs. Least recently used plus a byte cap is the answer |
| Cheaper alternative | Least recently used for space and a per-entry validator for freshness, which are separate concerns and should stay separate mechanisms |

---

## Module 13. Lifecycle, background execution and battery

**Time: 50 to 60 minutes reading, 35 to 45 minutes practice.**

The operating system is an adversary with good intentions. It will suspend your
process, defer your work, batch your alarms and kill you for memory, and it will
do none of it at a time you chose. This module turns that into arithmetic: a
battery budget, a wakeup allowance, and a job classification that follows from
both.

### 13a. Predict first

Read this artefact and commit to an answer before continuing.

```
+---------------------------------------------------------------+
| BACKGROUND SYNC v0                                            |
|   A periodic job every 15 minutes, all day                    |
|   Each run holds a wake lock, opens a connection, fetches     |
|   20 KB, writes the database, releases                        |
|   Active work per run: 1.2 seconds                            |
| Assumed device and radio facts                                |
|   Battery 3000 mAh at 3.85 V                                  |
|   Cellular radio holds a high-power state about 10 s after    |
|   the last packet, then steps down                            |
|   Average power while the radio is up: 500 mW                 |
+---------------------------------------------------------------+
Caption: a fifteen-minute periodic sync, with the two radio facts
that decide what it actually costs.
```

**Question.** Compute the share of a full battery this consumes per day, then
name which of its costs dominates and what that implies for the fix.

??? note "Show answer"

    **Battery energy.** 3.0 Ah x 3.85 V = 11.55 Wh. In joules: 11.55 x 3600 =
    41,580 J.

    **Per run.** The radio is up for the 1.2 s of work plus the roughly 10 s
    tail, so 11.2 s at 0.5 W = 5.6 J.

    **Per day.** 24 x 4 = 96 runs. 96 x 5.6 = 537.6 J. As a share: 537.6 /
    41,580 = 1.29 percent of a full charge, every day, for background sync alone.

    **What dominates.** The tail. Ten of the 11.2 seconds, so 89 percent of the
    energy, is spent after the payload has already arrived. That single fact
    rewrites the optimisation: shrinking the payload from 20 KB to 2 KB changes
    almost nothing, while halving the number of wakeups halves the cost. Move
    from 96 wakeups to 6 opportunistic ones and the cost is 6 x 5.6 = 33.6 J,
    which is 0.08 percent.

    **The bonus answer.** On a modern OS this job will not run every 15 minutes
    anyway. Periodic work is deferred into maintenance windows, so any product
    behaviour that depends on a 15 minute freshness guarantee is already false in
    the field and only looks correct on a plugged-in test device.

**The process state ladder, and what each step takes away.**

| State | What you have | What is taken |
|---|---|---|
| Foreground, visible | Full CPU, network, sensors | Nothing, but you are on a frame budget |
| Foreground-adjacent, user-visible task | Continued execution while a visible indicator shows | Requires declaring a purpose, and the user can see and stop it |
| Backgrounded, recently | A short grace window measured in tens of seconds | Long-running work. Assume 25 seconds and design to need less |
| Backgrounded, idle | Scheduled callbacks only, at the OS's convenience | Timers, sockets, sensors, and any notion of "now" |
| Suspended | Nothing runs | Everything. Your sockets are dead and your timers did not fire |
| Killed | Nothing exists | All in-memory state. Your next code runs in a fresh process |

Two consequences to state in an interview, because they are the ones candidates
miss:

- Every background unit of work must be idempotent and resumable, because it will
  be killed mid-run and re-run later. A job that assumes it finishes is a job
  that corrupts something eventually.
- There is no reliable "run at time T" for a backgrounded app. You express
  constraints and the OS chooses. Design the product around "eventually, and
  batched with other work", or move the work to the server and push the result.

**Classify the work first. The mechanism follows.**

| Class | Definition | Mechanism | Battery posture |
|---|---|---|---|
| User-visible now | The user is watching a progress indicator | Foreground task with a visible indicator | Expensive and honest. The user sees the cost |
| User-initiated, deferrable | The user asked for it, it may complete later | Constrained deferrable job, expedited if the product needs minutes not hours | Moderate. Attach constraints such as network type |
| Opportunistic | Nobody asked. Prefetch, cleanup, telemetry flush | Deferrable job with charging or idle constraints, or ride an existing wakeup | Should be nearly free or it should not exist |
| Server-side | It does not need the device at all | Do it on the server and push the result | Zero device cost |

The fourth row is the senior move. A large fraction of background work on
clients exists because someone put a loop on the device instead of asking the
server to compute the answer and notify. "Move it to the server" should be your
first proposal, and the reason to reject it (privacy, offline capability, data
the server does not have) should be a stated reason rather than an omission.

**Riding an existing wakeup, priced.** The 5.6 J figure was for an isolated
wakeup that pays the full radio promotion and tail. If your work runs while the
radio is already up (a push just arrived, the user just foregrounded the app,
another app woke the radio and the OS batched you alongside), you pay only the
active seconds.

- Isolated wakeup, 1.2 s of work: 11.2 s of radio at 0.5 W = 5.6 J
- Riding an existing wakeup, same work: 1.2 s at 0.5 W = 0.6 J
- Ratio: 9.3 to 1

That ratio is the entire argument for coalescing, and it is why "sync on push
arrival" beats "sync every fifteen minutes" even when the push carries no data.

**The four drain sources a client actually controls.**

| Source | Lever | Typical mistake |
|---|---|---|
| Wakeups | Fewer, coalesced, constrained | A timer per feature, each with its own schedule |
| Radio time | Batch requests, compress, avoid chatty protocols, cancel aggressively | Six parallel small requests where one batched request would do |
| Location fixes | Lower accuracy, larger distance filter, region monitoring instead of continuous | Continuous high-accuracy updates left running after the screen that needed them is gone |
| CPU seconds | Do less, do it off the main thread but do not do it twice, cache decode results | Re-parsing the same payload on every screen entry |

Relative cost, so you can prioritise. Treat this ordering as an assumption to be
confirmed on your own hardware, but it is stable enough to reason with: for the
same wall-clock second, the screen at high brightness costs the most, then the
cellular radio in its high-power state, then continuous satellite location, then
wifi, then a busy CPU core. The ordering matters more than the exact numbers,
because it tells you that a feature which keeps the screen on is a battery
feature before it is anything else.

**Low-power mode: a policy table, not a flag check.** When the OS signals a
constrained state, decide in advance what degrades and in what order.

| Degrade this | To this | User-visible cost |
|---|---|---|
| Prefetch depth | Next screen only, instead of next five | Slightly slower scroll into new content |
| Media autoplay | Off | The user taps to play |
| Telemetry flush | Batched to the next foreground session | Delayed dashboards, no user impact |
| Location accuracy | Coarse, larger distance filter | Less precise position on a map |
| Background upload | Deferred until charging or wifi | Photos post later, with a visible state |
| Animation richness | Reduced | Flatter transitions |

State the ordering rule too: degrade things the user cannot see before things
they can, and never degrade a thing the user explicitly asked for. Deferring a
user's photo upload because the battery is low is defensible only if the app
says so on screen.

**Measuring it, with the limits of each method.**

| Method | Sees | Cannot see | Use it for |
|---|---|---|---|
| On-device battery attribution | Per-app energy over hours, by subsystem | Which of your features caused it | A first triage: is it us at all |
| Power measurement rig | Instantaneous draw at millisecond resolution | Real user behaviour and network conditions | Comparing two implementations of the same job |
| Field A/B on drain per foreground hour | Real behaviour, real networks, real devices | Small effects, without enough users | Shipping decisions |
| Wakeup and job counters in your own telemetry | Exactly how often your jobs ran and how long they held resources | Energy, directly | Everyday regression detection. This is the cheapest and most useful one |

Size the A/B before running it. Assume drain per foreground hour has a standard
deviation of 8 percentage points across users, and you want to detect a 1 point
regression at conventional power:

- n per arm = 2 x sigma squared x (z_alpha + z_beta) squared / delta squared
- = 2 x 64 x (1.96 + 0.84) squared / 1
- = 2 x 64 x 7.84 = 1,004 users per arm

So roughly a thousand users per arm detects a one point change. If your rollout
stage is smaller than that, your battery A/B is decorative, and knowing that
before you run it is the difference between a measurement and a ritual.

```
+---------------------------------------------------------------+
| WHAT IS THE WORK FOR                                          |
+-------+-----------------+---------------+---------------------+
        |                 |               |
        v                 v               v
+---------------+ +---------------+ +-----------------------+
| USER WATCHING | | USER ASKED,   | | NOBODY ASKED          |
| visible task  | | can wait      | | prefetch, cleanup     |
| show progress | | constrained   | | ride an existing      |
| and a stop    | | deferrable    | | wakeup or drop it     |
+---------------+ +-------+-------+ +-----------+-----------+
                          |                     |
                          v                     v
                  +---------------------------------------+
                  | CAN THE SERVER DO IT INSTEAD          |
                  | if yes, do it there and push          |
                  +---------------------------------------+
Caption: classify background work by who is waiting for it, then
ask whether the device needs to be involved at all.
```

### 13d. Worked example

**The prompt.** "Our messaging client is being blamed for battery drain. Give it
a background design that fits inside 1 percent of a battery per day."

Inputs, stated before any design:

| Input | Value | Status |
|---|---|---|
| Battery | 3000 mAh at 3.85 V, so 41,580 J | ASSUMED, a mid-range phone, chosen because complaints come from mid-range devices |
| Isolated radio wakeup, 1 s of work | 5.6 J | DERIVED in section 13a from the tail assumption |
| Work riding an existing wakeup | 0.6 J per second of work | DERIVED, same power without the tail |
| Messages received per day | 200 | ASSUMED, an active user |
| Push notifications delivered per day | 200 | GIVEN by the messaging product |
| Media uploads per day | 5 | ASSUMED |

#### Block A. Purpose: turn the budget into a wakeup allowance

One percent of 41,580 J is 415.8 J per day. That is the entire background
allowance, and every job has to be paid out of it. I split it before designing
anything, because an unsplit budget is always overspent:

| Purpose | Share | Joules |
|---|---|---|
| Message delivery and fetch | 55 percent | 229 J |
| Media upload | 25 percent | 104 J |
| Token refresh, config, telemetry | 10 percent | 42 J |
| Headroom for the unforeseen | 10 percent | 42 J |

Now convert to units I can design with. At 5.6 J for an isolated wakeup, the
whole 415.8 J buys 74 isolated wakeups a day. At 0.6 J per second of work riding
an existing wakeup, the same budget buys nearly 700 seconds of work. The design
question is therefore not "how much work" but "how much of the work can ride
something that was going to happen anyway".

**The error most readers make here.** Starting from the feature list and adding
up. Start from the budget and divide it. A budget that is allocated before
design forces the trade-offs into the open, and the interviewer can see you
making them.

!!! tip "Self-explanation prompt"

    Before Block B, say why 200 pushes a day does not automatically mean 200
    isolated wakeups, and what property of push delivery makes that true.

#### Block B. Purpose: classify every background job against the ladder

| Job | Class | Mechanism | Cost per day |
|---|---|---|---|
| Fetch message body after a push | User-visible soon | Rides the push wakeup: radio already up | 200 x 0.3 s x 0.5 W = 30 J |
| Send read receipts | Opportunistic | Coalesced, piggybacked on the next outgoing request | Approximately 0 J |
| Upload media the user sent | User-initiated, deferrable | Constrained job, expedited, resumable | 5 x 8 s x 0.5 W = 20 J |
| Refresh auth token | Opportunistic | On next foreground, not on a timer | Approximately 0 J |
| Flush telemetry | Opportunistic | Charging or wifi constraint | 2 x 5.6 = 11.2 J |
| Prefetch avatars and link previews | Opportunistic | Rides the push wakeup, capped per day | 20 x 0.5 s x 0.5 W = 5 J |

Total: 30 + 20 + 11.2 + 5 = 66.2 J, which is 0.16 percent of the battery, well
inside the 1 percent budget. The headroom is deliberate: field conditions are
worse than assumptions, and radios in poor signal draw more for longer.

The row that carries the design is the first one. Because a push already wakes
the device and brings the radio up, fetching the message body in that window
costs 0.3 s of active radio rather than a full 11.2 s cycle. Two hundred fetches
cost 30 J instead of 1,120 J. That is the difference between a 0.07 percent
feature and a 2.7 percent feature, and it is entirely about when, not what.

**The error most readers make here.** Designing a polling loop as a fallback
"in case push fails" and leaving it always on. A safety-net poll every 15
minutes costs 537 J a day, which is more than the entire budget, to cover a
failure mode that occurs on a small fraction of devices. Make the fallback
conditional: poll only after an assumed 6 hours with no push received, which
costs 4 wakeups a day in the worst case.

#### Block C. Purpose: attach every remaining job to an existing wakeup

Four wakeup sources already exist. I list them and assign work to each, so no
job invents a schedule of its own:

| Existing wakeup | Work that rides it |
|---|---|
| Push arrival | Message body fetch, avatar prefetch, read-receipt flush |
| App foregrounded | Token refresh, config fetch, full sync, telemetry flush |
| Device charging and on wifi | Media upload retries, cache trimming, database maintenance |
| User-initiated send | Any pending read receipts and delivery acknowledgements |

The rule I state: no feature gets its own timer. New background work must name
which existing wakeup it rides, or justify a new one against the budget. This is
a process rule as much as a technical one, and it is the only thing that stops
background cost from creeping back up over four quarters.

**The error most readers make here.** Treating each feature's background work as
independently reasonable. Every one of six features scheduling "just a small
15 minute job" produces 576 wakeups a day. The cost is superlinear in team size
unless someone owns the total.

!!! tip "Self-explanation prompt"

    Say why "charging and on wifi" is a much stronger constraint than "on wifi",
    in terms of the joule arithmetic from Block A.

#### Block D. Purpose: define the degradation ladder before the OS forces it

Three constrained states, each with a defined behaviour:

| Signal | Behaviour |
|---|---|
| Low-power mode on | Drop avatar prefetch entirely, defer media upload to charging, halve telemetry flush frequency, keep message fetch untouched |
| Metered or roaming connection | Defer media upload with a visible "waiting for wifi" state, fetch message text but not media, drop prefetch |
| Storage low | Trim caches aggressively, refuse new prefetch, keep unsent user content |

The invariant across all three: message delivery, which is the product, never
degrades. Everything else is negotiable. Writing the invariant down first makes
the rest of the table fall out mechanically, and it gives you a one-sentence
answer when the interviewer asks what you would cut.

**The error most readers make here.** Deferring the user's own outgoing media
silently. If the app decides to hold a photo until wifi, the user must see that
decision on the message row. A silent deferral looks exactly like a bug, and
users retry, which costs more energy than sending it would have.

#### Block E. Purpose: instrument so the number can be defended

Four counters, emitted with the telemetry that already flushes opportunistically:

| Counter | Why it earns its place |
|---|---|
| Background wakeups per day, by trigger | The single number the whole budget is expressed in |
| Radio-seconds per day, estimated from request start and end times | A proxy for the dominant cost that needs no special platform support |
| Job runs, completions and kills, by job name | Kills mean non-resumable work is being wasted and repeated |
| Drain per foreground hour, from the platform battery signal | The user-facing outcome, for the A/B |

I set the regression alert on wakeups per day rather than on battery, because
wakeups move immediately when someone adds a timer, while battery drain is noisy
and lagging. Assume a baseline of 12 wakeups a day per user at p50; alert when
p50 exceeds 18, which is a 50 percent increase and far outside ordinary
behavioural variation.

**The error most readers make here.** Reporting only the mean. Background cost
is a tail problem: a small cohort with a retry loop, a poison outbox entry or a
failing job burns hundreds of wakeups a day while the mean barely moves. Report
p50, p95 and p99, and alert on p95.

### 13e. Fade the scaffold

**Fade 1: a fitness app that records a workout.** Location is sampled during a
run, the screen is usually off, and the recording must survive the app being
backgrounded for an hour. Same five-block shape, last block blank.

- Block A, budget: this is user-initiated and user-visible, so the budget is far
  larger than 1 percent. State it as a product number: an assumed 8 percent of
  battery per hour of recording is acceptable for a workout tracker because the
  user chose to start it and can see it running.
- Block B, classification: location sampling is a user-visible foreground task
  with a persistent indicator, not deferrable work. Uploading the finished
  workout is user-initiated deferrable. Fetching route suggestions is
  opportunistic and should be off during recording.
- Block C, ride existing wakeups: nothing rides anything here, because the
  recording itself is the continuous cost. The lever is sampling rate: at an
  assumed 1 Hz high-accuracy sampling versus 0.2 Hz with a 10 metre distance
  filter, the second sends the location subsystem to sleep between fixes and cuts
  its share substantially. Measure both on a rig.
- Block D, degradation: on low-power mode, keep recording at reduced sampling
  and say so on screen. Never silently stop a recording, because losing a
  user's marathon is unrecoverable and unforgivable.
- Block E, instrumentation: **you write this block.**

??? note "Show answer"

    The counters differ from the messaging case because the failure mode differs.
    Here the thing that ruins a user's day is a truncated or lost recording, so
    the instrumentation is about integrity first and energy second.

    Counter one: sample gap distribution during a recording, specifically the
    count of gaps longer than an assumed 30 seconds. A gap means the OS suspended
    the sampling or the process was killed, and it shows up as a straight line
    across a map.

    Counter two: recordings that ended without an explicit user stop, as a share
    of all recordings. This is the process-death rate, and it is the number to
    put in front of a product manager, because it converts directly into support
    tickets.

    Counter three: bytes and rows of recording data flushed to durable storage
    per minute, plus the age of the newest unflushed point. This tells you the
    bound on how much a kill can destroy. Write at least once every assumed 10
    seconds so a kill loses at most 10 seconds of route.

    Counter four, the energy one: battery percentage delta per recorded hour, by
    sampling configuration, so the sampling-rate decision in Block C is backed by
    field data rather than the rig.

**Fade 2: an email client with three configured accounts.** Users expect new
mail to appear without opening the app, two of the accounts support push and one
does not. Last two blocks blank.

- Block A, budget: assume a 1.5 percent daily background allowance, slightly
  above the messaging client because mail is a background-first product, so
  415.8 J becomes 624 J.
- Block B, classification: pushed accounts are the messaging case and cost
  almost nothing. The non-push account requires polling, which is the entire
  problem, and it must be budgeted separately and visibly.
- Block C, ride existing wakeups: **you write this block.**
- Block D, degradation: **you write this block.**

??? note "Show answer"

    Block C. The two push accounts already generate wakeups, so the polled
    account rides them: whenever a push arrives for account A, also poll account
    C if it has not been polled in the last assumed 10 minutes. At an assumed 60
    pushes a day across the two push accounts, that alone gives the polled
    account far more freshness than a standalone schedule would, at 0.6 J per
    second of work rather than 5.6 J per wakeup.

    Then add a floor for the case where the push accounts are quiet. Poll on
    foreground, on charging, and on an assumed 4 hourly deferrable job as a
    backstop. Six standalone wakeups a day at 5.6 J is 33.6 J, which is 5 percent
    of the 624 J budget for the worst case, and the arithmetic is what makes the
    decision defensible rather than arbitrary.

    Say the honest thing too: the polled account will be less fresh than the
    pushed ones, sometimes by hours, and the app should show when each account
    last synced rather than pretending they are equivalent.

    Block D. The degradation ladder for mail differs from messaging because the
    payload is heavy. On low-power mode: fetch headers only, no bodies, no
    attachments, no images. On metered connections: same, plus defer attachment
    prefetch entirely. Under both: keep the polled account on its foreground and
    charging triggers only, dropping the 4 hourly backstop, because that backstop
    is the most expensive line in the budget and the least valuable when the user
    is conserving power.

    The invariant to state: sending never degrades. A queued outgoing message
    keeps its expedited path in every constrained state, because a user who
    pressed send and later discovers it sat in a queue for six hours has been
    failed in a way no battery saving justifies.

### 13f. Check yourself

**Q1.** A colleague proposes replacing your push-triggered fetch with a 5 minute
polling job "so freshness is predictable". Using the joule figures from this
module, compute the daily cost and state the freshness the poll actually
delivers.

??? note "Show answer"

    A 5 minute poll is 288 runs a day. Each is an isolated wakeup at 5.6 J, so
    288 x 5.6 = 1,612.8 J, which against 41,580 J is 3.9 percent of a full
    battery per day. That is roughly twenty-four times the 66.2 J the entire
    background design cost, to serve one feature.

    The freshness it delivers is not 5 minutes. Backgrounded periodic work is
    deferred into OS maintenance windows, so in the field the interval stretches,
    often to tens of minutes or hours on an idle device. So the proposal spends
    3.9 percent of the battery to buy a predictability that the operating system
    does not grant. The correct response is not "no", it is "here are both
    numbers, and here is the fallback poll that costs 4 wakeups a day and only
    runs when push has been silent for 6 hours".

**Q2.** Your background job is killed roughly 30 percent of the time before it
completes. Name the two design properties the job must have, and describe what
goes wrong for a media upload job that has neither.

??? note "Show answer"

    The two properties are idempotence and resumability. Idempotence means
    re-running the job after a partial run produces the same end state, which
    requires a stable operation identifier the server can deduplicate on.
    Resumability means the job records progress durably as it goes, so the next
    run continues rather than restarts.

    A media upload with neither behaves badly in two ways at once.

    Without resumability, a 20 MB upload killed at 80 percent restarts from
    zero. At a 30 percent kill rate per attempt the expected number of attempts
    is 1 / 0.7 = 1.43, and each failed attempt has already burned on average half
    the payload, so you pay roughly 1.43 x 20 MB of transfer for 20 MB of
    content, plus the radio time.

    Without idempotence, an upload that actually completed but whose
    acknowledgement was lost is retried, and the recipient sees the photo twice.

    The fix for both is small: chunked upload with a server-issued session
    identifier, a stored offset after each chunk, and the same session identifier
    presented on every retry.

### 13g. When not to use this

**A background job at all.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You can name a user-visible outcome that fails without it, AND the work cannot be done on the server, AND you have the joule cost from a wakeup count |
| Scale threshold below which it is over-engineering | If the work is only ever consumed when the user opens the app, doing it on foreground is free and always fresh. A large share of background jobs in shipping apps compute things nobody reads before they are recomputed |
| Cheaper alternative | Do it on foreground entry, with a skeleton screen for the first few hundred milliseconds. Users accept a brief load far more readily than they accept battery drain |

**Push-triggered background fetch.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Content must be present before the user opens the app, AND you have measured how often a silent push actually results in the fetch running, which is well below 100 percent |
| Scale threshold | If the payload fits in the push itself, assume a couple of kilobytes, then send it in the push and skip the fetch. Below that size the round trip is pure overhead |
| Cheaper alternative | Put the content in the notification and fetch on tap. The user who never opens the notification cost you nothing at all |

**A polling fallback.**

| Question | Answer |
|---|---|
| Measurement that justifies it | A measured share of devices that receive no pushes over a long window, for example above 1 percent of daily actives with zero deliveries in 24 hours |
| Scale threshold | Under that, a permanent poll spends a percentage point of every user's battery to serve a rounding error of users. Make it conditional and per-device instead of global |
| Cheaper alternative | Poll only on foreground plus a long backstop that arms itself after a silence threshold, so the cost is paid by the affected devices only |

**A foreground task with a visible indicator.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The user started it, expects it to finish now, and would be surprised by a delay. Uploads over an assumed 30 seconds and active recordings qualify |
| Scale threshold | For work under a few seconds, the ordinary background grace window after the app leaves the screen is enough, and a persistent indicator for a two second task is noise the user learns to ignore |
| Cheaper alternative | A deferrable job with an expedited hint, plus a state on the relevant row in the interface so the user can see it is pending without a system-level indicator |

---

## Module 14. Concurrency and memory

**Time: 45 to 55 minutes reading, 35 to 45 minutes practice.**

A client has one thread that is allowed to touch the screen and a hard deadline
that arrives 60, 90 or 120 times a second. Every concurrency decision in a
mobile app is downstream of that. This module turns "do work off the main
thread" into arithmetic, then does the same for memory.

### 14a. Predict first

Read this artefact and commit to an answer before continuing.

```
+---------------------------------------------------------------+
| FEED ROW BINDING, as written today                            |
|   onBind(row):                                                |
|     bytes = readFileFromCache(row.id)          3 ms           |
|     model = parseJson(bytes)                   4 ms           |
|     applyValuesToViews(model)                  1 ms           |
| Facts measured on the target device                           |
|   Display refresh rate 90 Hz                                  |
|   During a fling the list binds 3 new rows per frame          |
|   Measure, layout and draw cost 2 ms on top of binding        |
+---------------------------------------------------------------+
Caption: a row binding that does disk and parse work inline, with
the refresh rate and bind rate that turn it into dropped frames.
```

**Question.** Compute the frame rate the user actually sees during a fling, and
name the second failure this code will produce that a frame trace will not show.

??? note "Show answer"

    **Frame budget.** At 90 Hz the display asks for a frame every 1000 / 90 =
    11.1 ms.

    **Work per frame.** Three rows at (3 + 4 + 1) = 8 ms each is 24 ms, plus 2
    ms of measure, layout and draw, so 26 ms.

    **Delivered rate.** 26 / 11.1 = 2.34, and a frame cannot be delivered
    part-way through a refresh interval, so each produced frame spans 3
    intervals. The user sees 90 / 3 = 30 frames per second during the fling,
    with the characteristic uneven judder of a variable overrun.

    **The second failure.** The 3 ms disk read is a median. Storage latency has a
    long tail, especially on a nearly full device or while another app is
    writing. Assume a p99 of ten times the median, so 30 ms for one read. That
    single row turns a 26 ms frame into a 53 ms frame, which is a visible hitch
    rather than a smooth 30 fps. Frame-rate averages hide this completely, which
    is why the metric to track is the count of frames over budget, not the mean.

    If storage stalls badly enough that the main thread blocks for seconds, the
    OS reports an application-not-responding event, and that arrives as a
    stability metric rather than a performance one, which is why nobody connects
    it to this function.

**What must, must not, and may run on the main thread.**

| Work | Main thread | Why |
|---|---|---|
| Mutating views, layout, animation callbacks | Must | The interface toolkit is not thread safe by design |
| Reading a value already in memory | Fine | Nanoseconds, and moving it costs a hop |
| File or database access | Never | Median is milliseconds, tail is tens of milliseconds |
| JSON, protocol buffer or XML parsing | Never | Scales with payload, and payloads grow silently |
| Image decode | Never | Tens of milliseconds and megabytes of allocation |
| Cryptography, compression | Never | CPU bound, and the tail depends on device class |
| Regular expressions over long strings | Never | Innocent looking and occasionally catastrophic |
| Object allocation in a bind function | Avoid | Allocation churn causes collector pauses inside the frame |

**Frame budgets, with the headroom rule.**

| Refresh rate | Interval | Usable budget at 70 percent | What the other 30 percent is |
|---|---|---|---|
| 60 Hz | 16.7 ms | 11.7 ms | System compositing, other processes, thermal variation |
| 90 Hz | 11.1 ms | 7.8 ms | Same, but you have 33 percent less of it |
| 120 Hz | 8.3 ms | 5.8 ms | Same again, and now a single allocation pause matters |

The 70 percent figure is an operating convention rather than a law, and here is
why it is reasonable: your process does not own the device, and a budget set at
100 percent of the interval is a budget that is exceeded whenever anything else
happens. Setting it at 70 percent means ordinary system noise does not produce a
dropped frame.

**The rule that fixes most feed performance: bind is not the place to compute.**
Move work to whichever of these is earliest and cheapest:

| Move it to | Example | What it costs |
|---|---|---|
| Server | Preformatted display strings, precomputed aspect ratios | Response size, and a contract you must version |
| Ingest time, off the main thread | Parse once into a display model when the page arrives | Memory for the parsed model |
| Background prefetch, ahead of scroll | Decode the next screen of images | Wasted work if the user stops scrolling |
| Bind time | Only assigning already-computed values to views | Nothing, if the previous rows did their job |

Applying that to the artefact: parse the page once when it arrives, hold display
models in memory, and binding becomes the 1 ms assignment step. Work per frame
drops from 26 ms to 3 x 1 + 2 = 5 ms, which fits inside the 7.8 ms usable budget
at 90 Hz with room left.

**Sizing thread pools, rather than picking eight.**

| Pool | Size rule | Worked | Why |
|---|---|---|---|
| CPU bound | Number of performance cores, minus one | Assume 8 cores, so 6 | More threads than cores adds context switching, not throughput |
| Network | Little's law: concurrency = throughput x latency | 20 requests per second x 0.2 s = 4 | Beyond this, requests queue in your pool instead of in flight |
| Disk | 1 to 2 | 2 | Storage is a single device. Extra threads reorder into worse access patterns |
| Database writes | Exactly 1, a serial queue | 1 | Concurrent writers serialise on a lock anyway, and a serial queue makes it explicit and debuggable |

Little's law is worth stating out loud in an interview because it converts a
guess into a derivation: if you want to sustain 20 requests per second and each
takes 200 ms, you need 4 in flight. Asking for 32 in-flight requests does not
make the network faster, it makes 28 of them wait somewhere less visible.

**Structured concurrency, in one paragraph.** Tie every task to the lifetime of
the thing that asked for it: a screen, a row, a request. When the screen goes
away, its scope is cancelled and every child task is cancelled with it. The
benefit is not elegance, it is that you can no longer forget, because forgetting
requires deliberately launching outside the scope.

**Cancellation is cooperative, which is the trap.** A cancelled task does not
stop. It is marked cancelled and stops at the next point that checks. A tight
decode loop or a synchronous socket read checks nothing and runs to completion.
Two practical rules:

- Check for cancellation between units of work, and make the units small enough
  that the check happens within a frame or two.
- Pass cancellation into libraries that support it (request cancellation on the
  networking client, decoder abort flags) rather than relying on your own loop.

**Pricing cancellation.** During a fast fling a feed can start decoding an image
for every row it binds. Assume 200 rows fly past, 20 of which stop on screen, and
each decode costs 15 ms of CPU and allocates 2.6 MB.

- Without cancellation: 200 x 15 ms = 3.0 s of CPU and 520 MB of allocation
  churn, all of it competing with the frames the user is watching
- With cancellation on recycle: roughly 20 useful decodes, so 0.3 s and 52 MB
- Ratio: 10 to 1, and the wasted 90 percent lands precisely when the device is
  busiest

**Memory: the numbers that decide a design.**

| Fact | Consequence |
|---|---|
| Bitmaps are width x height x bytes per pixel, with no compression in memory | A 4000 x 3000 photo is 48 MB decoded. Always decode to display size |
| The heap ceiling is per process and lower than device RAM | A 2 GB device may give your process a few hundred megabytes before killing it |
| Collector pauses land inside frames | Allocation in a bind function is a frame-rate bug, not a memory bug |
| The OS kills by a score that rises with your footprint | Being the biggest background app means being killed first, which shows up as slow resume, not as a crash |

**Five leaks that account for most client memory bugs.**

| Leak | Mechanism | Detection |
|---|---|---|
| A destroyed screen retained by a listener | The screen registered a callback and never removed it | Watch destroyed screens, force a collection, dump if still reachable |
| A static or singleton cache holding view objects | Convenient at write time, permanent at run time | Grep for statics holding interface types, then heap diff |
| A closure capturing the screen | An asynchronous callback keeps the whole object graph alive | Same watcher, plus reference-path inspection |
| An unbounded collection | A list that only ever appends, such as an in-memory event log | Growth over a long session, visible in an allocation timeline |
| Native memory not released | Buffers, decoded frames or handles held by non-managed code | Resident set grows while the managed heap does not |

The detection pattern worth naming: after a screen is destroyed, hold a weak
reference to it, run a collection, and if it is still reachable, capture the
reference path. This finds the first three rows automatically and is cheap
enough to leave on in development builds for everyone.

**Responding to memory pressure, as a ladder.**

| Pressure level | Drop | Keep |
|---|---|---|
| Mild | The in-memory bitmap cache, entirely | Parsed display models, screen state |
| Moderate | Parsed models for screens not on top of the stack | The visible screen and unsynced user data |
| Severe | Everything reconstructible, including the visible screen's off-viewport data | Unsynced user data, always |

The invariant is the last cell: never discard class B data (user-authored and
unsynced, from Module 12) to relieve memory pressure. Everything else is a cache
and can be rebuilt.

```
+-------------------+   post results only   +-------------------+
| MAIN THREAD       | <==================== | CPU POOL, size 6  |
| views, layout     |                       | parse, decode     |
| assign values     |                       | compress          |
+-------------------+                       +-------------------+
        ^                                            ^
        | cancel on destroy                          |
+-------+-----------+                       +--------+----------+
| SCREEN SCOPE      | ====================> | NETWORK POOL, 4   |
| owns every task   |   launch and cancel   | in-flight limit   |
+-------------------+                       +-------------------+
        |
        v
+-------------------+                       +-------------------+
| DISK POOL, size 2 | ====================> | DB WRITE QUEUE, 1 |
| file reads        |                       | serial, batched   |
+-------------------+                       +-------------------+
Caption: one screen scope owns every task it starts, four pools are
sized by what they contend for, and only assignment happens on the
main thread.
```

### 14d. Worked example

**The prompt.** "Our feed drops frames on mid-range devices and runs out of
memory on 2 GB devices. Redesign the concurrency and memory model."

Inputs before any change:

| Input | Value | Status |
|---|---|---|
| Target refresh rate | 90 Hz, so 11.1 ms per frame | GIVEN |
| Usable budget at 70 percent | 7.8 ms | DERIVED |
| Rows bound per frame during a fling | 3 | GIVEN, measured |
| Current bind cost | 8 ms per row | GIVEN, measured |
| Heap ceiling on the target device | 256 MB | ASSUMED, a 2 GB device typically grants a few hundred megabytes per process |
| Cell image at display size | 1080 x 600 x 4 = 2.59 MB | DERIVED |

#### Block A. Purpose: measure before changing anything

I ask for three artefacts before I touch code, and I say why each one is
necessary rather than nice:

| Artefact | Question it answers |
|---|---|
| A frame trace during a fling | Which function is inside the over-budget frames, rather than which function is slow in isolation |
| An allocation timeline for the same fling | Whether the drops are compute or collector pauses, because the fixes are different |
| A heap dump after visiting and leaving the feed ten times | Whether the memory problem is footprint or a leak, because those are also different fixes |

I state the discriminator up front: if frame drops line up with collection
events, the problem is allocation churn and the fix is to stop allocating in
bind. If they line up with long function bodies, the problem is compute and the
fix is to move it. Guessing between those two is how a week disappears.

**The error most readers make here.** Optimising the function that looks
expensive. The function inside the dropped frames is often not the slowest
function in the app, and the two are found by different tools.

!!! tip "Self-explanation prompt"

    Before Block B, say why a heap dump taken after ten visits is more useful
    than one taken after a single visit, and what specifically you compare.

#### Block B. Purpose: empty the bind path

Current cost per frame: 3 x 8 + 2 = 26 ms against a 7.8 ms usable budget, so I
need to remove roughly 21 ms.

| Work | Moved to | New bind cost |
|---|---|---|
| Disk read of the cached row | Page load, on the disk pool | 0 ms |
| JSON parse | Page load, on the CPU pool, producing display models | 0 ms |
| Text measurement for variable-height rows | Precomputed with the display model | 0 ms |
| Assign values to views | Stays on the main thread | 1 ms |

New cost per frame: 3 x 1 + 2 = 5 ms, inside the 7.8 ms budget with 2.8 ms of
headroom. That headroom is not slack, it is the allowance for the p99 cases,
and I say so.

The memory consequence has to be checked in the same breath, because I have
traded time for space. A display model at an assumed 1.5 KB, for 200 cached
items, is 300 KB. That is negligible, so the trade is free. If the model had
been 100 KB per item I would have had to hold models only for a window around
the viewport, and that is a different design.

**The error most readers make here.** Moving the parse to a background thread
but leaving it in bind. Now bind returns immediately, the row renders empty, and
the content arrives a frame or two later, so the user sees flickering
placeholders during every fling. Off the main thread is not the same as out of
the bind path.

#### Block C. Purpose: size the pools and serialise the database

Four pools, each sized by what it contends for rather than by taste:

| Pool | Size | Derivation |
|---|---|---|
| CPU (parse, decode) | 6 | 8 cores assumed, minus one for the main thread and one for system work |
| Network | 4 | Little's law: 20 requests per second x 0.2 s of latency |
| Disk | 2 | One storage device. Extra threads produce worse access patterns |
| Database writes | 1, serial | Writers serialise on a lock regardless, so make it visible |

I add one rule that matters more than the sizes: image decode gets its own
bounded queue with a depth cap, an assumed 8 entries, and newest-first ordering.
During a fling the newest request is for a row the user is about to see, and the
oldest is for a row that has already left the screen. A first-in-first-out decode
queue spends the whole fling decoding rows nobody will look at.

**The error most readers make here.** Using one large shared pool for
everything. A burst of image decodes then starves the network calls that feed
the next page, so the list runs out of content precisely when the user is
scrolling fastest. Pools are not just about thread count, they are about
isolation.

!!! tip "Self-explanation prompt"

    Say why newest-first ordering on the decode queue is correct here, and name a
    screen where it would be exactly wrong.

#### Block D. Purpose: make cancellation actually happen

Three scopes, each tied to a lifetime the framework already manages:

| Scope | Cancelled when | Owns |
|---|---|---|
| Screen | The screen is destroyed | Page fetches, prefetch, anything not tied to a row |
| Row | The row is recycled | That row's image decode and any per-row fetch |
| Request | The response arrives or times out | Retries and the parse of that response |

The row scope is the one that pays. Using the earlier arithmetic, cancelling on
recycle takes a fling from 200 decodes to roughly 20, which is 3.0 s of CPU down
to 0.3 s and 520 MB of churn down to 52 MB.

I check the cooperative-cancellation trap explicitly: the decoder must be given
an abort signal, and the loop that resizes must check between rows of pixels,
not only at the end. A cancelled task that runs to completion has cost exactly
as much as an uncancelled one and is harder to reason about.

**The error most readers make here.** Cancelling the wrong scope. Cancelling
page fetches on row recycle means fast scrolling repeatedly kills the request
that would have supplied the next page, and the list stalls at the bottom. Match
each task to the lifetime that genuinely bounds it.

#### Block E. Purpose: bound memory and detect the leak

Two mechanisms, one for footprint and one for leaks.

Footprint: the bitmap cache is capped in bytes at 32 MB, derived in Module 12
from the heap ceiling, and the pressure ladder drops it first. Display models
are held for a window of an assumed 200 items and trimmed on each refresh.

Leaks: a weak-reference watcher registers every destroyed screen and row
holder, forces a collection after an assumed 5 second delay, and captures a
reference path if the object is still reachable. In development builds this
fails the test suite. In release builds it reports a counter, not a dump,
because a heap dump on a user's device is both slow and full of their data.

The one number I would put on a dashboard: process kills while backgrounded, as
a share of sessions. It converts memory footprint into a user-visible outcome,
namely how often reopening the app restarts it from cold instead of resuming.

**The error most readers make here.** Shipping the leak detector's full dump
capability to production. It captures user data, it takes seconds, and it
usually triggers on the slowest devices. Report the fact of the leak in
production and capture the detail in development.

### 14e. Fade the scaffold

**Fade 1: a map screen that decodes and composites tiles.** Tiles arrive over
the network, decode to 256 x 256 x 4 = 262 KB each, and the visible viewport
plus one ring is 25 tiles. Same five-block shape, last block blank.

- Block A, measure first: frame trace during a pan and a pinch-zoom, allocation
  timeline across a zoom level change (the worst case, because every tile is
  replaced at once), heap dump after ten screen visits.
- Block B, empty the draw path: tiles are decoded on the CPU pool and uploaded
  as textures ahead of being needed. Drawing assigns already-decoded tiles only.
  A zoom level change must reuse the previous level's tiles scaled, so the screen
  is never blank while the new level decodes.
- Block C, pool sizing: network pool sized by Little's law at an assumed 30 tile
  requests per second and 150 ms latency, so 4 to 5 in flight. Decode pool of 6.
  Decode queue ordered by distance from the viewport centre, not by arrival.
- Block D, cancellation: tiles outside the viewport plus ring are cancelled on
  every pan step. During a fast pan this is the dominant saving, exactly as row
  recycle was in the feed.
- Block E, memory and leak detection: **you write this block.**

??? note "Show answer"

    Footprint first. The viewport plus ring is 25 tiles at 262 KB, so 6.6 MB
    resident for what is visible. A cache of two additional zoom levels for
    smooth zooming triples that to roughly 20 MB, which is defensible against a
    256 MB ceiling. Cap in bytes at 24 MB and evict by distance from the current
    viewport and zoom level, not by recency, because recency on a map means the
    tiles you just panned away from and will not return to.

    The map-specific hazard is native memory. Texture memory usually lives
    outside the managed heap, so the managed heap can look flat while the process
    resident size climbs to a kill. The counter to watch is process resident size,
    not heap size, and the leak signature is textures that are never released
    when their tile is evicted. Tie texture lifetime to the cache entry
    explicitly rather than to garbage collection.

    Leak detection follows the same weak-reference watcher pattern, applied to
    the map screen and to any overlay or annotation objects, which are the usual
    culprits because they are registered with the map and often not removed.

    The dashboard number changes too: for a map, track frames over budget during
    pan and zoom separately, because zoom is the operation that replaces every
    tile at once and it is where the design either holds or does not.

**Fade 2: a chat screen that decrypts each message before rendering.** Messages
are end-to-end encrypted, decryption costs an assumed 0.4 ms per message on the
target device, and a conversation opens showing the last 50 messages. Last two
blocks blank.

- Block A, measure first: trace the open of a long conversation, which is the
  worst case, and separately trace scrolling upward through history. Check
  whether frame drops coincide with collections, because a per-message
  decryption that allocates a new buffer each time is an allocation problem
  wearing a compute costume.
- Block B, empty the bind path: 50 messages at 0.4 ms is 20 ms of decryption on
  open. That is a visible stall on a screen the user just tapped into. Decrypt
  off the main thread, in a batch, before the screen presents, and cache the
  plaintext in memory keyed by message identifier so scrolling back never
  decrypts twice.
- Block C, pool sizing: **you write this block.**
- Block D, cancellation: **you write this block.**

??? note "Show answer"

    Block C, pool sizing. Decryption is CPU bound, so it belongs on the CPU pool
    of 6, but with one important restriction: use a single serial queue per
    conversation. Decrypting messages out of order is possible for some schemes
    and impossible for others, because ratcheting schemes derive each message key
    from the previous state. Stating that constraint is the point of this block:
    the pool size is decided by the cryptographic protocol, not by the core count.

    So the shape is a serial queue per conversation, and at most a few
    conversations decrypting at once, which in practice means a bounded pool of 2
    to 3 with per-conversation serialisation inside it. Database writes stay on
    the single serial writer from the feed design.

    Block D, cancellation. Cancelling decryption is subtly dangerous. If the
    scheme advances state as it decrypts, an aborted decryption can leave the
    session state inconsistent, and the user experiences that as messages that
    can never be read. The rule is that a decryption unit is not cancellable once
    started, and the cancellation happens at the queue level: drop queued items
    for conversations the user has left, but let the in-flight item finish.

    What is cancellable is everything around it: media downloads for messages
    scrolled past, link preview fetches, and avatar loads all cancel on recycle
    exactly as in the feed. Separating "cancellable because it is a cache fill"
    from "not cancellable because it advances protocol state" is the senior
    distinction here.

### 14f. Check yourself

**Q1.** A colleague raises the disk pool from 2 threads to 12 to "speed up
loading". Predict what happens to median and p99 read latency and explain the
mechanism in two sentences.

??? note "Show answer"

    Median latency stays roughly the same or worsens slightly, and p99 gets
    markedly worse. The storage device serves one request at a time internally,
    so twelve threads do not create twelve parallel reads, they create a deeper
    queue with more interleaving, which destroys sequential access patterns and
    adds queueing delay on top of service time.

    The second-order effect matters more: twelve blocked threads hold twelve
    stacks and twelve sets of buffers, and on a memory-constrained device that
    raises footprint for no throughput. The correct lever is fewer and larger
    reads, not more concurrent small ones.

**Q2.** You cancel an image decode when a row is recycled, but the trace shows
the decode still running to completion 40 ms later. Give the mechanism and the
smallest fix.

??? note "Show answer"

    Cancellation is cooperative. Marking the task cancelled sets a flag; the code
    stops only when it reaches a point that checks the flag. A decode implemented
    as one call into a decoder library has no such point between its start and
    its end, so it runs to completion regardless.

    The smallest fix is to give the decoder an abort signal it checks internally,
    which most decoding interfaces expose in some form, or to decode in slices
    (rows or tiles of the image) with a cancellation check between slices. If
    neither is possible, the fallback is admission control rather than
    cancellation: do not start a decode for a row unless it has been on screen
    for at least one frame, which during a fast fling prevents most of the work
    from starting at all.

### 14g. When not to use this

**Custom thread pools.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You can show one class of work starving another, for example page fetches queued behind image decodes during a fling, in a trace |
| Scale threshold below which it is over-engineering | An app with a handful of concurrent operations per screen gains nothing from four pools. The platform's default dispatchers for CPU and IO work are correct until you can show contention |
| Cheaper alternative | The platform's two standard dispatchers plus an explicit in-flight limit on the networking client, which is one configuration line and covers the common starvation case |

**Structured concurrency scoping down to the row level.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Wasted work during fast scrolling is measurable: count decodes started versus decodes whose result is displayed. Below roughly a 3 to 1 ratio the scoping is not paying |
| Scale threshold | On a short list that does not recycle, such as a settings screen, per-row scopes add ceremony to solve a problem that cannot occur |
| Cheaper alternative | Screen-level scope only, plus an image library that already cancels on view detach, which is standard behaviour in the mature ones |

**Object pooling to avoid allocation.**

| Question | Answer |
|---|---|
| Measurement that justifies it | An allocation timeline showing collection pauses inside dropped frames, attributable to a specific allocation site in a hot loop |
| Scale threshold | Modern collectors handle short-lived small objects cheaply. Below a few thousand allocations per frame, pooling trades a real correctness risk (objects used after return to the pool) for an unmeasurable gain |
| Cheaper alternative | Hoist the allocation out of the loop, reuse a single buffer owned by the caller, or precompute the value at load time so the hot path allocates nothing |

**A leak watcher running in production builds.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Background process kills are a meaningful share of sessions and development builds do not reproduce the leak, so you need field signal |
| Scale threshold | If your kill rate is already low, the watcher costs a forced collection per screen destruction on every user's device to find nothing |
| Cheaper alternative | Report the counter only, never the dump, and sample it at an assumed 1 percent of sessions. You get the trend without the cost or the privacy exposure |

---

## Module 15. Platform variation

**Time: 35 to 45 minutes reading, 25 to 35 minutes practice.**

Most course material on this topic is either a list of API names with no
consequence, or a shrug that says the platforms are basically the same. Both are
useless in an interview. This module states the four differences that change an
architecture, treats everything else as naming, and gives the cross-platform
answer honestly.

### 15a. Predict first

Read this artefact and commit to an answer before continuing.

```
+---------------------------------------------------------------+
| DESIGN DOC EXCERPT, submitted for review                      |
|   1. A background service syncs every 15 minutes on both      |
|      platforms.                                               |
|   2. We hold a websocket open while the app is backgrounded   |
|      so messages arrive instantly.                            |
|   3. Tokens go in the platform preferences store, which is    |
|      equivalent on both sides.                                |
|   4. The same permission prompt copy ships on both, shown at  |
|      first launch.                                            |
+---------------------------------------------------------------+
Caption: four cross-platform claims from a design review, three
of which do not survive contact with an operating system.
```

**Question.** For each numbered claim, say whether it holds, and for the ones
that do not, name the architectural consequence rather than the API.

??? note "Show answer"

    **Claim 1, false on both, for different reasons.** Neither platform
    guarantees a 15 minute cadence for a backgrounded app. Both defer periodic
    work into batched maintenance opportunities, and the deferral grows with how
    little the user opens your app. The consequence is architectural: any product
    promise phrased as "within 15 minutes" has to move to a server-side push, or
    be restated as "next time you open the app".

    **Claim 2, false.** A backgrounded process is suspended, and a suspended
    process has no sockets. The consequence is that instant delivery must come
    from the platform push service, which means the message content, the
    ordering, and the fetch-on-arrival path all become part of the design rather
    than an implementation detail of the socket you were going to keep open.

    **Claim 3, false in the way that matters.** Both platforms have a simple
    preferences store and a separate hardware-backed key store, and only the
    latter is appropriate for a credential. The consequence is that token
    storage, backup exclusion and logout clearing become a shared requirement
    with two different implementations, not one shared line of code.

    **Claim 4, true as written and wrong as a design.** You can show the same
    prompt at first launch on both. The consequence is that you will get your
    lowest possible grant rate on both, because you asked before the user has any
    reason to say yes. Permission timing is a product decision that is
    cross-platform, and the platforms differ in what a denial means afterwards.

**The four differences that change an architecture.** Everything else on the
comparison tables below is naming, and naming is worth thirty seconds in an
interview. These four are worth ten minutes.

| Difference | Why it changes the design |
|---|---|
| Background execution model | Decides whether a feature can exist on the device at all, or must move to the server plus push |
| Process death and state restoration | Decides where interface state lives and whether your screens must be reconstructible from a serialisable bundle |
| Permission granularity and prompt policy | Decides how many distinct denial states your feature must handle, and whether a partial grant exists |
| Delivery and update mechanism | Decides how fast you can fix a bug, whether you can roll back, and therefore how much you must put behind a remote flag |

**Execution and scheduling.**

| Concern | One platform | The other | Cross-platform stacks |
|---|---|---|---|
| Deferrable background work | A job scheduling framework with constraints (network, charging, idle), persisted across reboot | A registered background task with a system-chosen opportunity, plus a separate longer processing task | Wrap both. The shared part is the job definition and constraints, never the scheduling |
| Long running with the user watching | A foreground service with a declared type and a persistent notification | A background task assertion for a bounded window, plus a background transfer service for uploads and downloads | Must be native on both. This is where every wrapper leaks |
| Guaranteed periodic execution | Not guaranteed | Not guaranteed | Not guaranteed. Stop designing around it |
| Reaction to process suspension | Process may be killed and later recreated with saved state | Process is suspended, and may be terminated without further code running | Identical requirement: all work is idempotent and resumable |

**Storage, security and privacy.**

| Concern | One platform | The other | Cross-platform stacks |
|---|---|---|---|
| Simple scalars | A key-value preferences store, whole-file load on first access | A user defaults store with similar semantics | Wrap trivially. This one really is naming |
| Structured local data | An embedded SQL database, usually behind an object mapping layer | An embedded SQL database, usually behind an object graph framework | Shared schema definition, platform-specific driver |
| Secrets | A hardware-backed key store, keys can require biometric authentication | A keychain with device-only accessibility classes and a secure element | Never share. The accessibility and migration semantics differ and both matter |
| Backup inclusion | Per-directory and per-file rules, configurable | Per-file exclusion attributes and directory conventions | Must be set natively, and it is the most commonly missed configuration in the whole list |

**Networking, push and delivery.**

| Concern | One platform | The other | Cross-platform stacks |
|---|---|---|---|
| Push transport | A vendor push service with a device token, priorities and collapse keys | A vendor push service with a device token, priorities and collapse identifiers | The registration and token lifecycle must be native. The routing logic on your server is shared |
| Data-only push | Delivered subject to app standby state, may be delayed or dropped | Delivered as a content-available notification, with a per-app budget and no delivery guarantee | Best effort on both. Design for repair, not delivery |
| Background transfer | Enqueued through the work scheduler, or a foreground service for user-visible transfers | A system-managed session that continues after the app exits and relaunches the app on completion | Native on both, and this is a common reason a wrapper needs a native module |
| Version rollout control | Staged rollout by percentage, halt and resume | Phased release over days, with a pause | Same product need, different granularity. Your flag system must not depend on either |

**Interface, state and concurrency.**

| Concern | One platform | The other | Cross-platform stacks |
|---|---|---|---|
| Declarative interface | A modern declarative toolkit alongside an older view system | A modern declarative toolkit alongside an older view controller system | The whole point of the stack is to replace both |
| Concurrency primitive | Structured coroutines with scopes, plus executors | Structured tasks with actors, plus dispatch queues | Usually an event loop with an isolate or worker model. The threading contract differs and it leaks into every native module |
| Process death restoration | Explicit saved state bundles, and the system will recreate your screen | State restoration hooks, and the system may relaunch into a restored scene | Both require the same design property: every screen reconstructs from serialisable state |
| Navigation | A navigation component with a back stack that the system also manipulates | A navigation stack plus system-level gestures | A shared router that must still respect two different system back behaviours |

**The cross-platform column, without the marketing.** Three honest statements:

- The shared layer that pays is the one with no operating system in it: models,
  the sync engine, validation, the retry and backoff policy, feature flag
  evaluation, and the display-model transformation from Module 14.
- The layer that does not pay is anything touching background execution, push
  registration, secure storage, permissions, or system interface conventions.
  Every serious cross-platform app writes native code for these, and the
  question is only whether you admit it in the design.
- The costs to price are startup time, binary size, and the debugging surface of
  the bridge between the two worlds. Ask for measurements of all three from the
  team proposing the stack, and if none exist, that is the finding.

```
+---------------------------------------------------------------+
| SHARED CORE, no operating system calls                        |
| models, sync engine, retry policy, flag evaluation, display   |
| model transforms, validation                                  |
+-------------------------------+-------------------------------+
                                |
        +-----------------------+-----------------------+
        v                                               v
+----------------------------+          +----------------------------+
| PLATFORM SHELL A           |          | PLATFORM SHELL B           |
| scheduling, push, keystore |          | scheduling, push, keychain |
| permissions, navigation    |          | permissions, navigation    |
| interface toolkit          |          | interface toolkit          |
+----------------------------+          +----------------------------+
Caption: the sharing line sits exactly where operating system calls
begin. Everything above it is one implementation, everything below
is two.
```

### 15d. Worked example

**The prompt.** "One team has to ship an offline-capable notes client on both
platforms. Decide what is shared and what is not."

Inputs, stated first:

| Input | Value | Status |
|---|---|---|
| Engineers available | 6 | GIVEN |
| Target: feature parity within one release cycle | Yes | GIVEN |
| Estimated total code | 60,000 lines | ASSUMED, typical for a notes client with sync, from comparable scope |
| Split of that code | 45 percent core logic, 35 percent interface, 20 percent platform integration | ASSUMED, and the point of Block A is to test the assumption rather than accept it |
| Cold start budget | 500 ms on a mid device | GIVEN, from the non-functional requirements |

#### Block A. Purpose: draw the sharing line by asking one question per component

The question is not "can this be shared" but "does this call the operating
system". I go component by component:

| Component | Calls the OS | Decision |
|---|---|---|
| Note model, validation, search index | No | Shared |
| Sync engine: outbox, tokens, conflict rules | No | Shared, and this is the most valuable share because it is the hardest code to write twice |
| Networking policy: retry, backoff, idempotency keys | No | Shared |
| Feature flag evaluation and bucketing | No | Shared |
| Local database schema and queries | Partly | Schema shared, driver native |
| Background scheduling | Yes | Native, two implementations |
| Push registration and token lifecycle | Yes | Native |
| Secure storage | Yes | Native |
| Permissions | Yes | Native |
| Interface | Yes, deeply | Decided in Block B |

Result: the 45 percent core is genuinely shared, the 20 percent platform layer
is genuinely not, and the 35 percent interface layer is the actual decision.
That framing turns a religious argument into a scoped one.

**The error most readers make here.** Debating the framework before drawing the
line. The line is a property of the app, not of the framework, and once it is
drawn the framework choice is mostly about the interface layer.

!!! tip "Self-explanation prompt"

    Before Block B, say why the sync engine is a better sharing candidate than
    the interface even if both are the same number of lines.

#### Block B. Purpose: decide the interface layer by pricing it, not by taste

Two candidate shapes, priced against the inputs:

| Shape | Duplicated work | Costs to check |
|---|---|---|
| Shared core, native interface on each side | 35 percent of 60,000 = 21,000 lines written twice | Two interface skill sets on a team of 6, and parity drift between platforms |
| Shared core and shared interface via a cross-platform toolkit | Close to zero interface duplication | Startup time, binary size, and native modules for the 20 percent platform layer regardless |

The measurements I would demand before choosing the second row, with the
threshold attached:

- Cold start delta from the runtime. Budget is 500 ms, so a runtime that adds
  more than an assumed 150 ms (30 percent of the budget) has to be justified by
  something bigger than code sharing.
- Binary size delta. Set a budget in advance, for example an assumed 8 MB, and
  hold it, because size affects install conversion on constrained networks.
- Frame timing on the target mid device for the scrolling list, measured, not
  demonstrated on a flagship.

With 6 engineers and a parity requirement, I would lean to the shared interface
and hold it to those three numbers, and I would say plainly that if any of the
three fails, native interfaces are the answer and the core sharing still stands.
Notice what did not decide it: nobody's preference.

**The error most readers make here.** Treating cross-platform as all or
nothing. The core sharing decision and the interface sharing decision are
independent, and the first one is nearly always worth taking.

#### Block C. Purpose: fork the four architecture-changing differences explicitly

These get named as separate work items with owners, because they are where
"shared" quietly becomes wrong:

| Difference | What forks | What stays shared |
|---|---|---|
| Background execution | Scheduling API, constraint expression, foreground task versus background assertion | The job definitions themselves, their idempotency requirements, and the retry policy |
| Process death and restoration | The save and restore hooks, and the state bundle format | The rule that every screen reconstructs from a serialisable state object |
| Permissions | Prompt invocation, partial grant states, settings deep links | The decision of when to prompt, the pre-prompt copy, and the denial fallback behaviour |
| Delivery and updates | Rollout mechanics and review timing | The flag system, the forced-update policy, and the minimum supported version contract |

The right-hand column is the one to say out loud. Each of these differences
forks an implementation while leaving a policy shared, and separating policy
from mechanism is what keeps two platform builds behaving the same.

**The error most readers make here.** Abstracting the platform difference away
behind a single interface such as `Scheduler.schedule(job, every: 15.minutes)`.
The abstraction implies a guarantee neither platform gives, so it teaches every
future engineer something false. Abstract at the level of "run this eventually,
when there is network", which both platforms can honour.

!!! tip "Self-explanation prompt"

    Say what goes wrong six months later when a new engineer calls that
    `every: 15.minutes` API, and how the honest abstraction would have prevented
    it.

#### Block D. Purpose: define the contract between core and shell

Four rules, each preventing a specific failure:

| Rule | Prevents |
|---|---|
| The core never blocks and never calls back on a thread of its own choosing. It takes an executor from the shell | Interface updates arriving on the wrong thread, which is a crash on both platforms |
| The core exposes state as an observable snapshot, not as events the shell must accumulate | Divergence between the two shells, which is how parity bugs are born |
| All platform capabilities enter the core as injected interfaces (clock, storage, network, key store) | Untestable core code, and a core that cannot run in a unit test on a build machine |
| The core has no notion of "now" except through the injected clock | Time-dependent sync bugs that only reproduce at midnight in one time zone |

Rule three has a payoff worth quantifying: with those interfaces injected, the
whole sync engine runs in a plain test process with a fake clock and a fake
network. A conflict-resolution suite that would take minutes on two device farms
runs in seconds on a build machine, which is what makes it get run.

**The error most readers make here.** Letting the shell reach into the core's
storage directly for a quick fix. Once that happens twice, the sharing line is
gone and every future change costs two implementations plus a coordination
problem.

#### Block E. Purpose: state the testing and release consequence

| Consequence | Detail |
|---|---|
| Core tests are platform independent and fast | Run on every commit, with a fake clock and simulated network conditions |
| Shell tests are integration tests on real devices | Fewer of them, concentrated on the four forked differences: scheduling, restoration, permissions, updates |
| Release trains can be independent per platform, but the flag system must be shared | A feature flagged on for one platform and not the other is normal, and the flag definition must live in one place |
| The minimum supported version contract is shared | Because the server must serve both, the compatibility window is set by the slower-adopting platform |

The last row is the one candidates miss. Your server contract's compatibility
window is not a per-platform decision. It is set by whichever platform's users
update most slowly, and that is a fact you look up rather than assume.

**The error most readers make here.** Testing the forked layer only on the
platform the author works on. The four architecture-changing differences are
exactly the places where a bug is invisible from the other side, and they need
explicit device coverage on both.

### 15e. Fade the scaffold

**Fade 1: a camera-heavy app shipping on both platforms.** Capture, on-device
processing, and upload. Same five-block shape, last block blank.

- Block A, sharing line: the processing pipeline definition, the upload queue,
  the retry policy and the metadata model are shared. Capture, camera
  configuration, hardware capability queries and the media library are native,
  because camera stacks differ far more than storage does.
- Block B, interface decision: the capture screen is native on both regardless
  of the rest, because it is latency-sensitive, gesture-heavy and full of
  platform-specific hardware behaviour. Everything after capture (review, edit,
  share) is a candidate for sharing.
- Block C, forked differences: background upload is the big one. It must
  continue after the app leaves the screen, which means a foreground task on one
  platform and a system-managed transfer session on the other, with the same
  resumable protocol underneath.
- Block D, core to shell contract: the core owns the upload state machine and
  exposes a snapshot per asset (queued, uploading, failed, done). The shell owns
  progress display and any system-level indicator.
- Block E, testing and release consequence: **you write this block.**

??? note "Show answer"

    Core tests: the upload state machine runs with a fake network that can drop
    at any byte offset, so resumability is a unit test rather than a field
    discovery. Add a test that kills and recreates the state machine mid-upload,
    since process death is the normal case here, not the exception.

    Device tests concentrate on the two things that cannot be faked: camera
    capture across a matrix of devices, and background upload continuation. The
    second needs a real device test that backgrounds the app, waits, and verifies
    completion, on both platforms, because it is the single most likely thing to
    silently regress with an OS update.

    Release consequence: capture code is the highest-risk code in the app because
    a camera bug is unrecoverable for the user in the moment. It gets a slower
    staged rollout than the rest, and it needs a remote kill switch that falls
    back to the system camera. The rollout stage cannot be shared between
    platforms because the underlying device populations differ.

    One more: the media library permission has different partial-grant states on
    the two platforms, including a limited selection mode. That means the app
    needs a designed experience for partial access, not an error state, and it
    must be tested as a first-class path on both.

**Fade 2: a payments software development kit shipped to third-party apps on
both platforms.** Last two blocks blank.

- Block A, sharing line: the protocol, the state machine, the error taxonomy and
  the retry policy are shared. Everything the host app can see (the public API
  shape, the threading contract, the error type) is deliberately native and
  idiomatic on each platform, because an SDK that feels foreign is an SDK people
  avoid.
- Block B, interface decision: any prebuilt screens are native. A cross-platform
  runtime inside someone else's app is a large imposition of binary size and
  startup cost on a host that did not choose it.
- Block C, forked differences: **you write this block.**
- Block D, core to shell contract: **you write this block.**

??? note "Show answer"

    Block C, forked differences. Two of the four matter enormously here and two
    barely at all. Background execution matters: the SDK must never schedule its
    own background work in a host app's process without the host's explicit
    consent, because it spends the host's battery budget and the host gets the
    blame. Delivery matters in reverse: the SDK cannot ship its own updates, so
    every behaviour that might need changing must be server-controlled, and the
    SDK must tolerate running at a version two years old.

    Process death matters moderately: a payment flow interrupted by process death
    must resume or fail cleanly, never leave a charge in an unknown state, so the
    state machine persists its progress. Permissions barely matter, since a
    payments SDK should request nothing.

    Block D, core to shell contract. The contract runs in the opposite direction
    from an app: the host app is the caller, so the SDK's public surface is the
    contract, and it must state its threading behaviour explicitly. Callbacks
    return on a caller-supplied executor, or on the main thread by documented
    default, never on an internal thread the host cannot reason about.

    The shared core must not appear in the public API at all. No shared type,
    no cross-platform runtime type, and no third-party type crosses the boundary,
    because every type you expose becomes a versioning obligation and a potential
    conflict with the host's own dependencies. The public surface is
    platform-idiomatic types only, with the shared core hidden behind them.

### 15f. Check yourself

**Q1.** An interviewer works on the platform you do not. Give the two sentences
you would say in the first minute, and explain what each one buys you.

??? note "Show answer"

    Sentence one: "I will name the mechanism generically and give the primitive I
    have used, and please tell me if the equivalent on your side behaves
    differently." This buys permission to be specific without
    pretending to expertise you do not have, and it invites the interviewer into
    a collaboration rather than a quiz.

    Sentence two: "The four places where the platforms genuinely change the
    architecture are background execution, process death, permission granularity
    and update delivery, so I will be explicit at each of those and treat the rest
    as naming." This buys the frame for the whole round: it tells the interviewer
    you have a model of the differences rather than a vocabulary list, and it
    means every later platform-specific question lands somewhere you have already
    marked.

**Q2.** A team proposes moving to a cross-platform stack to halve their client
work. Name the three measurements you would ask for and give the threshold you
would set on each, using a 500 ms cold start budget.

??? note "Show answer"

    One, cold start delta attributable to the runtime, measured on the target
    mid-tier device, not a flagship. Threshold: no more than an assumed 150 ms,
    which is 30 percent of the 500 ms budget. Anything larger means every future
    startup optimisation is fighting the framework.

    Two, binary size delta. Threshold: set in advance, for example an assumed 8
    MB, because install size affects completion of downloads on slow or metered
    connections and the effect is largest exactly where growth matters most.

    Three, frame timing for the heaviest scrolling surface in the app, on the
    same target device, reported as frames over budget rather than as a mean.
    Threshold: no worse than the existing native implementation.

    Then the honest addition: the claim of halving the work is already wrong,
    because the 20 percent platform integration layer stays native either way and
    the core logic could have been shared without changing the interface stack.
    The real question is only whether the interface layer, roughly a third of the
    code, is worth those three costs.

### 15g. When not to use this

**A cross-platform interface framework.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Interface code is a large share of your total, parity drift between platforms is causing real bugs, and the three measurements above all pass on your target device |
| Scale threshold below which it is over-engineering | For an app with one or two screens per platform, or for a latency-critical surface such as camera capture or a map, the runtime cost buys nothing you needed |
| Cheaper alternative | Share the core logic only, which captures most of the duplicated effort without touching the rendering path, and keep interfaces native |

**A shared cross-platform core module.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The core logic is genuinely complex (a sync engine, a state machine, a pricing calculation) and has been reimplemented divergently at least once |
| Scale threshold | If the core is thin, for example fetch and display, sharing adds a build system and a bridging layer to save a few hundred lines |
| Cheaper alternative | A shared schema and a shared contract test suite that both native implementations must pass. You get behavioural parity without shared binaries |

**An abstraction over platform background scheduling.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You have more than a handful of background jobs and their constraints are genuinely the same on both platforms |
| Scale threshold | With two or three jobs, the abstraction costs more than the duplication and hides the platform semantics that matter |
| Cheaper alternative | A shared job definition (name, constraints, idempotency key, retry policy) as plain data, executed by two native schedulers. Shared policy, native mechanism |

**A single unified permission facade.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Your app requests four or more permissions and you keep rewriting the same prompt-and-fallback logic |
| Scale threshold | Below three permissions, the facade obscures partial-grant states that differ per platform, and partial grants are exactly the states that produce bugs |
| Cheaper alternative | A shared decision table (when to prompt, what the pre-prompt says, what the denial fallback is) with native call sites, so the policy is shared and the states stay visible |

---

## Module 16. Security and privacy

**Time: 50 to 60 minutes reading, 35 to 45 minutes practice.**

The client runs on hardware the attacker may own, in a process they can inspect,
on a network they may control. Every good client security answer starts by
saying which attacker it is defending against, because the controls that stop
one do nothing against another.

### 16a. Predict first

Read this artefact and commit to an answer before continuing.

```
+---------------------------------------------------------------+
| CLIENT SECURITY, as proposed                                  |
|   refresh token stored in plain key-value preferences         |
|   access token lifetime 30 days                               |
|   TLS pinned to the leaf certificate of the api host          |
|   that leaf certificate is rotated every 90 days              |
|   feature gate: if device is not jailbroken, allow transfers  |
| Release facts                                                 |
|   store review takes 2 days                                   |
|   update adoption 50 percent in 7 days, 90 percent in 30 days |
|   5,000,000 installed users                                   |
+---------------------------------------------------------------+
Caption: five client security choices, plus the release facts that
turn one of them into an outage.
```

**Question.** Compute how many users lose all access at the next certificate
rotation even if you plan perfectly, and separately name what the jailbreak gate
actually prevents.

??? note "Show answer"

    **The outage.** Pinning to a leaf means every rotation invalidates the pinned
    value, so a new pin must reach every device before the rotation. Best case,
    you ship the new pin 30 days ahead. At 90 percent adoption in 30 days, 10
    percent of users are still on the old pin on rotation day. Ten percent of
    5,000,000 is 500,000 people whose app cannot reach the API at all, and cannot
    be fixed remotely because the fix travels over the connection that is broken.

    Ship it only 7 days ahead and adoption is 50 percent, so 2,500,000 users are
    locked out. There is no partial degradation here: pinning failure is total.

    **What the jailbreak gate prevents.** Nothing that matters. It is a boolean
    computed inside a process the attacker controls, on a device they own. Anyone
    capable of modifying the app flips it, and tooling to do so is widely
    available. It stops a curious user, not an attacker, and worse, it creates a
    false sense that a client-side check is a control.

    The real controls for the same concern are server side: transaction limits,
    step-up authentication for risky actions, anomaly detection, and a
    platform-issued attestation that the server verifies rather than the client
    asserts.

**Start with the threat model, because it decides everything else.**

| Attacker | Capability | What actually defends against them | What does not |
|---|---|---|---|
| Passive network observer | Reads traffic | Transport encryption | Anything on the device |
| Active interceptor with a trusted root installed on the device | Reads and modifies traffic | Certificate or public key pinning | Transport encryption alone |
| Thief with a locked device | Physical possession | Device encryption, keys bound to hardware and to user authentication | App-level passcodes |
| Thief with an unlocked device | Full app access | Session timeout, re-authentication for sensitive actions | Storage encryption, since the app is already unlocked |
| Malware or another app on the device | Reads shared storage, clipboards, logs, screenshots | Key store for secrets, no sensitive data in logs or clipboard, screen capture flags | Obfuscation |
| The user, modifying your app | Full control of the process | Server-side enforcement, verified attestation, rate limits | Client-side checks of any kind |
| Compromised backend or insider | Sees whatever you sent | Data minimisation, on-device processing, short retention | Nothing on the client, after the fact |

The row that changes an interview is the sixth. Say plainly that the client is
untrusted, that every rule the business depends on is enforced server side, and
that client-side checks exist to improve the experience of honest users and to
raise the cost for casual attackers. That single framing separates a security
answer from a list of features.

**Secure storage, and what each tier resists.**

| Tier | Resists | Does not resist |
|---|---|---|
| Plain preferences or a plain file | Nothing beyond ordinary app sandboxing | Backup extraction, a rooted device, another app with elevated access |
| File encrypted with a key stored beside it | Casual inspection | Anyone who reads the same directory, which is the same attacker |
| Hardware-backed key store, device-only | Backup extraction and transfer to another device | An attacker with the device unlocked and the app running |
| Hardware-backed key requiring user authentication | The above, plus use while the device is locked or by another process | A user who authenticates, which includes a coerced user |

The design rule that follows: the sensitivity of the data decides the tier, and
the tier decides the failure you accept. Write both down. "Refresh token, in the
hardware key store, device-only, so a stolen backup is useless and a stolen
unlocked phone is not" is a complete sentence with a stated residual risk.

**Token lifetimes, derived rather than picked.** A stolen access token is usable
until it expires, so the expected exposure from a single theft is roughly half
the lifetime. Choose the lifetime from the value at risk per unit time:

| Product | Value at risk | Access token lifetime | Reasoning |
|---|---|---|---|
| Banking or payments | A transaction, immediately | 5 to 15 minutes | Expected exposure of a few minutes, and a refresh is cheap because the app is online when it matters |
| Social or messaging | Message history and identity | 1 to 24 hours | Frequent refresh costs battery and fails offline, and the loss is not monetary |
| Media playback | Access to content | Hours to days | Content is licensed, not owned, and revocation happens at a different layer |

The refresh token is the long-lived secret, so it gets the strongest storage and
the strictest handling:

- Rotate on every use, so a refresh token is single use.
- Detect reuse. If a rotated token is presented twice, the family is compromised.
  Revoke every descendant and force re-authentication.
- Bind to the device where the platform allows it, so a stolen token cannot be
  replayed from elsewhere.
- Clear on logout, on account switch, and on detected compromise.

**Pinning, done in a way that cannot brick your app.** Five rules, each of which
prevents a specific incident:

| Rule | Prevents |
|---|---|
| Pin the public key (the subject public key info), not the certificate | A routine certificate renewal with the same key from breaking every client |
| Pin the issuing intermediate or a long-lived key, not a short-lived leaf | The 90 day rotation problem computed in 16a |
| Ship at least two pins, including a backup key that is not yet in use | A forced key change with no ability to reach clients |
| Give the pin set an expiry date after which pinning is skipped | Turning a certificate incident into a permanent outage for old app versions |
| Roll out in report-only mode first, and watch the failure rate | Discovering a misconfigured pin from support tickets rather than from telemetry |

There is one more, and it is the detail that marks a senior answer: the kill
switch for pinning cannot be delivered over the pinned connection. If pinning
breaks, that channel is exactly what is unavailable. Fetch the pin configuration
from a separate host with its own pin set, or sign the configuration and allow
it to arrive through the push payload, which travels over the platform's
connection rather than yours.

**Permission models compared, at the level that changes a design.**

| Property | One platform | The other | Design consequence |
|---|---|---|---|
| When you may ask | At the point of use, and repeatedly until the user declines twice | At the point of use, once. A second ask is not shown | On both, treat every prompt as the only one you will get |
| Partial grants | Coarse location as an alternative to precise, and one-time grants | Limited photo selection, reduced accuracy location, one-time grants | Partial access is a first-class state, not an error path |
| After denial | The system stops showing the prompt, you must send the user to settings | Same outcome, different route | You need a designed screen that explains what is lost and links to settings |
| Background variants | Background location is a separate, later, stricter grant | Similar escalation from in-use to always | Never ask for the background variant first. Earn it after the in-use feature has demonstrated value |

**The prompt policy that produces the best outcome.**

- Ask at the moment the user is trying to do the thing that needs it, never on
  first launch. A prompt at launch has no context, so the rational answer is no.
- Show a pre-prompt of your own first, explaining the value and offering a way
  out. This costs you nothing (a user who declines your screen was going to
  decline the system prompt) and it preserves the one system prompt you get.
- Instrument three numbers: prompt shown, granted, and feature completed. The
  third is the one that matters, because a high grant rate on a feature nobody
  finishes is not a win.
- Test the denial path as a primary path, not as an error.

**Denial fallbacks, so the feature degrades instead of dying.**

| Permission denied | Fallback that keeps the feature usable |
|---|---|
| Precise location | Ask for a postcode or city, or use coarse location with a wider result radius |
| Notifications | An in-app inbox with a visible unread state, plus an email channel if the user opted in |
| Camera | Choose from the photo library instead |
| Photo library, full | Use the limited-selection picker, and never nag for full access |
| Contacts | A share link the user can send through any app they like |
| Background location | A manual check-in button, with an honest explanation of what is less convenient |

**Tamper resistance, priced honestly.** Client-side integrity checks raise the
cost of attack; they do not prevent it. Budget effort by what the attacker
gains:

| Control | Raises attacker cost by | Real value |
|---|---|---|
| Root or jailbreak detection | Minutes | Deters casual modification, nothing more |
| Code obfuscation | Hours to days | Slows reverse engineering of a protocol |
| Anti-debugging and integrity self-checks | Days | Meaningful only alongside server-side enforcement |
| Platform attestation, verified server side | Substantially, because the signal is signed by hardware you do not control | The only client integrity signal a server should weight |
| Server-side limits, anomaly detection, step-up authentication | Not applicable, it changes what an attacker can achieve rather than what they can do | The actual control |

**Privacy as design, not as a policy document.**

- Minimise: every field you do not send is a field that cannot leak, cannot be
  subpoenaed, and cannot be mis-retained.
- Process on device where you can, and say what that costs (battery, model size,
  and a slower path to changing the logic).
- Redact by construction: a logger that refuses to accept token or credential
  types is better than a review that hopes nobody logged one.
- Watch the accidental channels: clipboard contents, screenshots in the task
  switcher, keyboard caches on sensitive fields, and crash reports containing
  request bodies.
- Retention: decide how long each on-device store keeps data and enforce it in
  code, because "until the user clears it" is not a retention policy.

```
+---------------------------------------------------------------+
| WHAT THE ATTACKER CONTROLS                                    |
+-----------------+-----------------+-------------------------+
| THE NETWORK     | THE DEVICE      | YOUR APP PROCESS        |
| use TLS, pin    | key store,      | assume total control    |
| the public key, | device-only     | enforce server side,    |
| ship 2 pins     | keys, auth gate | verify attestation      |
+-----------------+-----------------+-------------------------+
        |                 |                     |
        v                 v                     v
+---------------------------------------------------------------+
| WHAT THE SERVER MUST STILL ENFORCE ALONE                      |
| limits, authorisation, anomaly detection, step-up auth        |
+---------------------------------------------------------------+
Caption: three attacker positions with the control that answers
each, and the row that no client-side control can substitute for.
```

### 16d. Worked example

**The prompt.** "Design the security posture for a banking client: storage,
tokens, transport, and what happens on a compromised device."

Inputs stated first:

| Input | Value | Status |
|---|---|---|
| Installed users | 5,000,000 | GIVEN |
| Update adoption | 50 percent in 7 days, 90 percent in 30 | GIVEN |
| Sensitive action | A transfer above a threshold | GIVEN |
| Session expectation | Users tolerate re-authentication for transfers, not for checking a balance | ASSUMED, and it is a product research question I would confirm |
| Certificate rotation | Leaf every 90 days, intermediate far less often | GIVEN |

#### Block A. Purpose: decide what the client can enforce at all

I split every rule into "client can enforce" and "server must enforce", before
designing a single control:

| Rule | Enforced where | Client's role |
|---|---|---|
| Transfer limits | Server | Show the limit, prevent obvious mistakes early |
| Authorisation for an account | Server | Never render an account the token does not grant |
| Device integrity | Server, from a verified attestation | Collect the attestation, do not judge it |
| Session freshness | Both | Client enforces an idle timeout for the experience, server enforces token expiry for real |
| Screenshot and clipboard restrictions | Client | Genuine value against other apps and shoulder surfing |
| Storage encryption | Client | Genuine value against a stolen device and backup extraction |

I say the framing sentence out loud: the client's security job is to protect the
honest user's data on the device and to avoid handing an attacker anything
useful, while every rule the bank depends on is enforced somewhere the attacker
cannot reach.

**The error most readers make here.** Presenting client controls as the security
design. An interviewer hearing "we detect jailbreak and obfuscate the binary"
with no server-side story concludes the candidate has not thought about who the
attacker is.

!!! tip "Self-explanation prompt"

    Before Block B, say why an idle timeout enforced only on the client is still
    worth building, given that an attacker can remove it.

#### Block B. Purpose: set token lifetimes from the value at risk

| Token | Lifetime | Storage | Derivation |
|---|---|---|---|
| Access token | 10 minutes | Memory only | Expected exposure from a theft is about 5 minutes. Refreshing every 10 minutes while active costs one small request, which rides an existing wakeup |
| Refresh token | 30 days, rotated on every use | Hardware key store, device-only, non-migratable | Long enough that a monthly user does not re-authenticate, single use so a stolen copy is detectable |
| Step-up token for transfers | Single transaction, minutes | Memory only | The highest-value action gets the shortest window, which is the whole point of step-up |

Reuse detection is the mechanism that makes 30 days acceptable: because each
refresh token is single use, a second presentation of the same token means
either a race or a theft. The server revokes the entire token family and forces
re-authentication. I would tune for a small number of false positives from
retries rather than tolerate a silent compromise, and I would make the client
handle a forced re-authentication gracefully because it will happen.

**The error most readers make here.** Persisting the access token "so the app
opens faster". It saves one request, adds a persistent copy of a credential, and
undoes the entire reason for a short lifetime.

#### Block C. Purpose: choose storage tiers and the authentication gate

| Data | Tier | Reason |
|---|---|---|
| Refresh token | Hardware key store, device-only, requires user authentication to use | An unlocked stolen phone still cannot mint new sessions without the user's biometric or passcode |
| Cached balances and recent transactions | Encrypted database with a key in the key store, not requiring authentication | Needs to render immediately on open. It is disclosure risk, not transaction risk |
| Payee list | Same as balances | Same reasoning, and it is genuinely useful offline |
| Full statements | Not cached at all | Fetch on demand. The convenience is small and the disclosure risk is large |
| Anything in a log or crash report | Nothing sensitive, enforced by a logger that rejects sensitive types | Crash reports leave the device and are read by many people |

I state the residual risk explicitly, because a security design without one is
incomplete: with an unlocked phone, an attacker sees balances and payees. They
cannot transfer, because that needs a step-up authentication bound to the
hardware key. That is the line I have deliberately drawn, and I would confirm it
against the bank's risk appetite.

**The error most readers make here.** Gating everything behind biometric
authentication. Requiring authentication to see a balance trains users to
authenticate constantly, which makes the prompt meaningless at the moment it
matters. Gate the action, not the view.

!!! tip "Self-explanation prompt"

    Say why requiring biometric authentication for every screen actually reduces
    security, in terms of what the user learns.

#### Block D. Purpose: pin transport without creating an outage

| Decision | Value | Why |
|---|---|---|
| What is pinned | The public key of the issuing intermediate, plus a backup key | Survives leaf rotation entirely, which removes the 90 day cliff |
| Number of pins | 2 minimum, one in use and one held in reserve | An emergency key change does not require an app update |
| Pin set expiry | 180 days from build date | An abandoned old version stops pinning rather than stops working |
| Rollout | Report-only for one full release cycle, measuring the failure rate | Corporate networks and interception proxies are the usual surprise |
| Kill switch delivery | A signed configuration fetched from a separate host with its own pins, plus a push-delivered override | The switch must not depend on the connection it is disabling |

Now the arithmetic that justifies it. Under the artefact's original design, a
rotation lockout hits 10 percent of 5,000,000 users, so 500,000 people, even
with a 30 day lead. Under this design, leaf rotation is invisible because the
intermediate key is unchanged, and even an intermediate change is covered by the
backup pin without any app update at all.

**The error most readers make here.** Treating pinning as a security control
with no availability cost. Pinning is the one client-side control that can take
your entire user base offline, and it must be designed with the failure mode in
front of you.

#### Block E. Purpose: handle the compromised device and the privacy surface

| Signal | Response |
|---|---|
| Platform attestation fails or is absent | Do not block the app. Send the signal to the server, which lowers transfer limits and requires step-up more often |
| Device is rooted by the user's own choice | Same path. Many developers and enthusiasts run modified devices and are not attackers |
| Screenshot or screen recording detected on a sensitive screen | Obscure sensitive values and log the event. Blocking is not always possible and should not be relied upon |
| Task switcher preview | Obscure the snapshot on backgrounding, which is a small change that prevents balances appearing in a screenshot the OS takes |
| Clipboard | Mark account numbers as sensitive where the platform allows, and never auto-copy a credential |

Privacy decisions, stated as design choices rather than compliance text:

- Do not send device identifiers you do not use. Every identifier collected is
  an obligation.
- Redact request and response bodies from crash reports by default, and allowlist
  specific fields rather than denylisting sensitive ones.
- Set a retention policy per on-device store: transactions cached for an assumed
  90 days, then trimmed, because the app is not an archive.

**The error most readers make here.** Blocking the app on a failed integrity
check. It converts a probabilistic signal into a binary outcome, locks out
legitimate users on unusual devices, and gives an attacker a clear oracle to
test their bypass against. Feed the signal into a risk score instead.

### 16e. Fade the scaffold

**Fade 1: a health app storing records offline.** Records are highly sensitive,
must be readable offline, and are shared with a clinician through a code. Same
five-block shape, last block blank.

- Block A, what the client can enforce: the client protects data at rest and in
  the interface. Access control over who may read a record is enforced server
  side, and the sharing code is validated there, never locally.
- Block B, token lifetimes: access token of an assumed 30 minutes, since the
  value at risk is disclosure rather than money movement and clinicians use the
  app in sessions. Refresh token 14 days, rotated, in the hardware key store,
  requiring user authentication because the data is sensitive enough to justify
  the friction.
- Block C, storage tiers: the record database is encrypted with a key store key
  gated on user authentication, and the key is released for the session so
  offline reading works after one unlock. Attachments are stored as encrypted
  files, excluded from backup.
- Block D, pinning: public key pinning on the intermediate, two pins, expiry,
  report-only first. Identical to the banking case, because the transport threat
  does not vary by industry.
- Block E, compromised device and privacy surface: **you write this block.**

??? note "Show answer"

    Compromised device: the response differs from banking because there is no
    transaction to limit. The lever is disclosure, so a failed attestation should
    restrict bulk operations (exporting all records, generating a new sharing
    code) while leaving ordinary reading alone. Blocking reading would deny a
    patient their own medical data on their own device, which is the wrong
    outcome even under a real compromise.

    Screenshots and the task switcher matter more here than in banking, because a
    single screen can disclose a diagnosis. Obscure the snapshot on backgrounding
    and flag record screens against capture where the platform supports it.

    Privacy is the dominant axis. Minimise hard: the server does not need the
    device model, the advertising identifier, or a precise location, and
    collecting any of them creates an obligation with no product benefit.
    Telemetry must never carry a record identifier, and screen names in analytics
    must not encode a condition, which is a subtle leak: an event named
    "viewed_diabetes_summary" is a diagnosis in your analytics pipeline.

    Retention: records the user has not opened in an assumed 12 months are
    dropped from the local cache and refetched on demand, which bounds what a
    device compromise exposes without changing the everyday experience.

**Fade 2: a consumer app adding a peer-to-peer money transfer feature.** Last
two blocks blank.

- Block A, what the client can enforce: nothing about the money. Every limit,
  every eligibility rule and every fraud decision is server side. The client
  contributes signals and a good experience.
- Block B, token lifetimes: the existing social session stays long lived, but
  the transfer feature gets a separate step-up token, minutes long, obtained
  through a re-authentication that is bound to hardware.
- Block C, storage tiers: **you write this block.**
- Block D, pinning and transport: **you write this block.**

??? note "Show answer"

    Block C, storage tiers. The correct answer is mostly "store less". Do not
    cache a balance, a card number, or a full transaction history locally, because
    the feature is used occasionally and the disclosure risk is continuous. Cache
    only what makes the feature usable: a recent recipients list, which is
    contact-derived and moderately sensitive, in the encrypted database.

    The step-up token lives in memory and never touches disk. Any payment
    instrument reference is an opaque server-issued token, never a card number,
    so the worst case of a full device compromise is a reference that is useless
    without the server's authorisation.

    One genuinely important detail: the transfer feature must not inherit the
    social app's logging conventions. Amounts, recipients and instrument
    references are excluded from analytics and crash reports by type, enforced in
    the logger rather than by review.

    Block D, pinning and transport. Pin the payment endpoints, and consider not
    pinning the rest. This is a real trade rather than a purity question: pinning
    the whole app raises the blast radius of a pinning mistake to the entire
    product, while pinning only the payment host limits an incident to a feature
    that can be disabled remotely.

    That gives a clean failure story: if the payment pin fails, the feature shows
    an unavailable state and the rest of the app keeps working, and the remote
    kill switch that disables the feature travels over the unpinned social API.
    The kill switch is again not on the pinned channel, which is the rule from
    the module applied to a split-host design.

### 16f. Check yourself

**Q1.** Your app pins correctly, and support reports that users on a particular
corporate network cannot sign in. Give the mechanism, and say what your
report-only rollout would have told you.

??? note "Show answer"

    The corporate network runs an interception proxy that terminates TLS and
    re-signs traffic with a root certificate installed on managed devices. That
    is exactly the attacker profile pinning defends against, so pinning is doing
    its job and the corporate proxy looks identical to an active interceptor.

    A report-only rollout would have shown a small, geographically or
    organisationally clustered population of pin validation failures before you
    enforced anything. That gives you the choice in advance: accept the breakage
    and document it, add an enterprise configuration that disables pinning for
    managed deployments, or provide a supported path for those users. Discovering
    it after enforcement means the affected users cannot sign in to report it
    through the app.

**Q2.** A product manager asks for a "remember me" feature that keeps users
signed in for a year. Give your counter-proposal with the specific token design
and one number that shows the exposure you removed.

??? note "Show answer"

    Counter-proposal: keep the user signed in for a year at the interface level,
    but do not keep a single credential alive for a year. Use a refresh token
    with a 30 day sliding window that rotates on every use, stored in the
    hardware key store as device-only and requiring user authentication for use,
    plus a short access token in memory.

    Exposure removed, stated numerically: a stolen credential with a one year
    lifetime has an expected usable window of about six months. With a 30 day
    sliding, single-use refresh token, expected exposure falls to about 15 days,
    and in practice much less because the legitimate device's next refresh
    triggers reuse detection and revokes the family. That is roughly a twelvefold
    reduction in exposure while the user still experiences a year of not signing
    in, which is what the product manager actually asked for.

### 16g. When not to use this

**Certificate or public key pinning.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You handle credentials, money or regulated data, AND you have an operational process that can rotate pins with a lead time longer than your slowest 10 percent of update adoption |
| Scale threshold below which it is over-engineering | For a content app with no user secrets beyond a session token, ordinary transport security plus short token lifetimes covers the realistic threat. Pinning adds an outage mode with no matching gain |
| Cheaper alternative | Short-lived tokens, server-side anomaly detection on unusual client behaviour, and transport security with certificate transparency monitoring on the server side |

**Biometric or passcode gating on app open.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The default screen exposes data that would harm the user if seen by whoever holds their unlocked phone, such as balances or medical records |
| Scale threshold | If the sensitive action is rare, gate the action rather than the app. Gating open costs every user a prompt on every launch to protect a screen most of them are not worried about |
| Cheaper alternative | Obscure sensitive values by default with a tap to reveal, and require authentication only for the sensitive action |

**Root and jailbreak detection.**

| Question | Answer |
|---|---|
| Measurement that justifies it | It feeds a server-side risk score alongside platform attestation, and you have decided what a positive signal changes |
| Scale threshold | As a gate on functionality, never. It is trivially bypassed by anyone who matters and it blocks legitimate users on modified devices |
| Cheaper alternative | Platform attestation verified server side, weighted into a risk decision, with limits and step-up authentication as the actual response |

**On-device encryption of the whole local database.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The database holds sensitive or regulated data, AND you have measured query latency with encryption enabled against the frame budget from Module 14 |
| Scale threshold | If the data is public content the server also serves anonymously, encryption protects nothing and costs every read. Below any sensitive field, skip it |
| Cheaper alternative | Encrypt the specific sensitive columns with a key store key, or better, do not store the sensitive data locally at all and hold a server-issued reference instead |

---

## Module 17. Release engineering

**Time: 50 to 60 minutes reading, 40 to 50 minutes practice.**

You cannot patch a client. Whatever ships is what runs, on devices you do not
control, for months. Everything in this module exists to buy back the control
you lost at submission: flags to change behaviour without a build, deterministic
bucketing so experiments mean something, durable telemetry so you can see what
happened, and a rollout ladder that limits how many people a mistake reaches.

### 17a. Predict first

Read this artefact and commit to an answer before continuing.

```
+---------------------------------------------------------------+
| EXPERIMENT SETUP, as configured                               |
|   Remote config fetched at launch, blocking up to 5 s         |
|   Variant chosen server side, per request, from a random      |
|   number                                                      |
|   Exposure logged when the config fetch completes             |
|   Telemetry posted fire and forget, dropped when offline      |
|   Rollout was 50 percent, raised to 70 percent on day 3       |
| Facts                                                         |
|   Cold start budget 500 ms                                    |
|   Only 20 percent of users reach the screen the flag changes  |
+---------------------------------------------------------------+
Caption: an experiment configuration with four independent defects,
plus the two facts that let you size two of them.
```

**Question.** Name three separate reasons the experiment result is unreadable,
and compute the cold start damage.

??? note "Show answer"

    **One, assignment is not stable.** A per-request random number means the same
    user can receive the treatment on one request and the control on the next.
    There is no treatment group, only a population that got a blend, so any
    measured difference is diluted toward zero by construction.

    **Two, exposure is logged in the wrong place.** Everyone who launches is
    counted as exposed, but only 20 percent reach the screen the flag changes. A
    genuine 5 percent improvement among people who see the feature appears as
    0.05 x 0.20 = 1 percent across the logged population, so the experiment needs
    roughly 25 times the sample to detect the same real effect.

    **Three, missing data is correlated with the treatment.** Fire-and-forget
    telemetry drops events when offline or when the app is killed. If the
    treatment makes the app crash more or stalls it on poor networks, the events
    that vanish are exactly the ones describing the harm, so the treatment looks
    better than it is.

    **A fourth, for credit.** Raising the rollout from 50 to 70 percent mid-flight
    mixes cohorts: the added users have less exposure time and, because they
    arrived later, a different usage profile. If the bucketing scheme also
    reassigns on a percentage change, some existing users switch arms, which
    destroys the comparison outright.

    **Cold start damage.** A blocking config fetch adds the network round trip to
    startup. At an assumed p50 of 300 ms the budget of 500 ms is already 60
    percent consumed by one fetch, and at the p99 the 5 second timeout applies,
    so p99 cold start is at least 5 seconds against a 500 ms budget, a factor of
    ten. With no network at all, every launch waits the full 5 seconds and then
    proceeds, which users read as a frozen app.

**Four kinds of flag, four lifetimes, four owners.**

| Kind | Purpose | Lifetime | Removal rule |
|---|---|---|---|
| Release flag | Ship code dark, turn it on when ready | Weeks | Removed in the release after full rollout. Non-negotiable |
| Kill switch | Turn a feature off in an incident | Years | Kept, reviewed annually, tested by actually flipping it |
| Experiment flag | Decide between variants | The experiment plus its analysis window | Removed when the decision is made, and the losing code deleted |
| Entitlement or configuration | Plan tier, region, regulatory rule | Permanent | Not a flag at all. It is configuration and belongs in a different system |

**Flag hygiene, with the arithmetic that forces it.** Assume a team adds 8 flags
a quarter and removes 2. After two years that is 48 stale flags. Nobody tests
that surface, because the combinations are astronomically many, and the honest
practice is to test the default path and the fully-on path only. That is exactly
why every flag needs an owner, an expiry date and an automatic reminder, and why
a flag past expiry should fail a build rather than generate a ticket nobody
reads.

**The evaluation contract, which fixes the startup defect.**

- Flags are evaluated locally, from a cached snapshot on disk, in microseconds.
- Every flag has a compiled-in default, used when no snapshot exists. The first
  launch after install must be correct with no network at all.
- The snapshot is refreshed asynchronously, never on the startup critical path.
- Refreshed values apply on the next launch by default. Only flags explicitly
  declared live-safe apply mid-session, because a variant that changes under the
  user's fingers breaks both the experience and the experiment.

**Deterministic bucketing, in two stages.** The bucketing function is:

```
+-------------------------------------------------------------+
| bucket(salt, unit) = hash(salt + ":" + unit) mod 10000       |
|                                                             |
| eligible if  bucket(experiment_salt, unit) < rollout x 100  |
| variant   if bucket(variant_salt, unit) < 5000 then A else B|
+-------------------------------------------------------------+
Caption: two independent hashes. The first decides who is in the
experiment, the second decides which arm, so raising the rollout
adds users without moving anyone already assigned.
```

Why two stages rather than one is the point of the design. With a single hash,
raising the rollout from 50 to 70 percent shifts the boundary and can move users
across arms. With two, eligibility grows monotonically and every already-eligible
user keeps their arm forever, which is what makes a ramp safe mid-experiment.

The salt matters for a second reason: a different salt per experiment makes the
buckets independent, so a user who was in the treatment of experiment one is not
systematically in the treatment of experiment two. Reusing one salt across
experiments correlates them and quietly ruins both.

**Choosing the unit.**

| Unit | Stable across | Breaks on | Use when |
|---|---|---|---|
| Install identifier | Sessions, updates | Reinstall, second device | Pre-login experiments, onboarding |
| Account identifier | Devices, reinstalls | Not available before sign-in | Anything measured per person |
| Session | Nothing useful | Always | Never for experiments. Fine for load shedding |

The login transition is the trap. A user bucketed on install identifier before
sign-in and on account identifier afterwards can change arm mid-session. Two
defensible fixes: analyse only post-login exposures for account-level
experiments, or record the install-to-account link and keep the original
assignment for the life of the experiment. Pick one and say which.

**Detecting a broken split before reading any metric.** A sample ratio mismatch
means the observed split differs from the intended one, and it invalidates
everything downstream. Derive the tolerance:

- Total exposed users: 200,000, intended 50/50, so expected 100,000 per arm
- Standard deviation of a count under a fair split: sqrt(n x 0.25) =
  sqrt(50,000) = 223.6
- Observed 101,200 versus 98,800: the deviation is 1,200, which is 1,200 / 223.6
  = 5.4 standard deviations

Five standard deviations is not a fluctuation, it is a bug: a filter applied to
one arm, a crash that removes users from one arm before exposure, or a
bucketing mistake. The operating rule is to check this first and refuse to read
the metrics if it fails.

**Exposure logging, stated precisely.** Log an exposure at the line where the
flag value changes what the user gets, once per session, carrying the flag name,
the variant, and the configuration version. Not at fetch, not at app start, and
not on every evaluation. The dilution arithmetic in 17a is the whole
justification, and it is the single most common experimentation bug in mobile
clients.

**Offline-durable telemetry, sized.**

| Decision | Value | Derivation |
|---|---|---|
| Events per session | 40 | ASSUMED, a moderately instrumented app. Count yours |
| Bytes per event | 300 | ASSUMED, a name, a timestamp, and a handful of properties |
| Per session | 12 KB | 40 x 300 |
| Seven offline days at 5 sessions per day | 420 KB | 35 sessions x 12 KB |
| Queue cap | 5 MB | Roughly twelve weeks of the above, which comfortably exceeds any plausible offline period |
| Batch size | 50 events, about 15 KB, compressed to roughly 3 KB | One request per 50 events instead of 50 requests |

Four properties the queue needs, each preventing a specific failure:

- Durable and transactional: an event written and the app killed a millisecond
  later must still be there. Otherwise your crash-adjacent events are the ones
  you lose.
- Deduplicated by an event identifier generated at creation. Retries after an
  ambiguous timeout are normal, and double counting corrupts every rate.
- Priority classes with a drop policy: when the cap is reached, drop the lowest
  priority oldest first, and never drop crash, stability or billing events.
- Clock corrected: send the device's wall clock, the device's monotonic uptime,
  and the age of each event at send time, so the server can reconstruct ordering
  even when the device clock is wrong or was changed while offline.

**Staged rollout, with the hold time derived rather than guessed.** The question
a stage answers is: has this build made stability worse. Size it.

- Baseline crash-free sessions: 99.6 percent, so p1 = 0.004
- A regression worth catching: 99.3 percent, so p2 = 0.007
- Sample per arm at conventional power: (1.96 + 0.84) squared x (p1(1 - p1) +
  p2(1 - p2)) / (p2 - p1) squared
- = 7.84 x (0.003984 + 0.006951) / 0.000009
- = 7.84 x 0.010935 / 0.000009 = 9,525 sessions

So roughly 9,500 sessions in the new build are needed to see a 0.3 point crash
regression. At 1,000,000 daily sessions:

| Stage | Sessions per day | Time to 9,500 | Hold |
|---|---|---|---|
| 1 percent | 10,000 | About 23 hours | 24 hours |
| 5 percent | 50,000 | About 4.6 hours | 12 hours, to cross a daily cycle |
| 20 percent | 200,000 | About 1.1 hours | 12 hours |
| 50 percent | 500,000 | Under 30 minutes | 24 hours, to include a weekend or weekday change |

The holds beyond the detection time exist for a different reason, and saying so
is what makes the ladder defensible: usage patterns vary by hour and by day, and
a bug that only appears on the commute or on a weekend needs a clock, not a
sample size.

**Forced update and version skew.**

| Mechanism | When it is right | Cost |
|---|---|---|
| Soft prompt, dismissible | Routine encouragement, two or more versions behind | Near zero, and adoption gains are small |
| Soft prompt, blocking once per week | An important fix, and you accept some annoyance | Some churn among infrequent users |
| Hard block | A security fix or a server contract you genuinely cannot keep serving | Every user below the floor is locked out until they update |

Size the hard block before proposing it. At 90 percent adoption in 30 days, a
hard block 30 days after release still locks out 10 percent of installed users,
and those users are disproportionately on old devices or constrained networks.
The server contract's compatibility window is therefore a business decision with
a number attached: supporting one more version costs you a code path, and
dropping it costs you a measurable share of users.

**Release health, and what to alert on.**

| Metric | Compare against | Alert |
|---|---|---|
| Crash-free sessions and crash-free users | The previous release at the same stage | A drop beyond the detection threshold you derived |
| Application-not-responding or hang rate | Previous release | Any increase, since these are usually main-thread regressions |
| Cold start p50 and p90 | Previous release, per device tier | A p90 regression above an assumed 10 percent |
| Network error rate by endpoint | Previous release | A rise that is not explained by a server incident |
| Adoption curve | The historical curve | A stall, which usually means a store issue rather than a code issue |

Compare against the previous release rather than an absolute target, because
absolute crash rates vary with device mix and season, and a relative comparison
at the same rollout stage controls for both.

### 17d. Worked example

**The prompt.** "Design the flag, experiment and telemetry stack for a 1 million
daily active user app."

Inputs first:

| Input | Value | Status |
|---|---|---|
| Daily active users | 1,000,000 | GIVEN |
| Sessions per user per day | 1 | ASSUMED, so 1,000,000 daily sessions, which keeps the rollout arithmetic simple and is conservative |
| Cold start budget | 500 ms | GIVEN |
| Baseline crash-free sessions | 99.6 percent | ASSUMED, a typical healthy consumer app. Use your own |
| Events per session | 40 at 300 bytes | ASSUMED, from the instrumentation plan |

#### Block A. Purpose: fix the evaluation path before designing anything clever

The startup rule comes first because everything else depends on it: flags
evaluate from a local snapshot, synchronously, in microseconds, with compiled
defaults. The network refresh happens after the first frame, on a background
thread, and applies at the next launch unless the flag is declared live-safe.

| Path | Latency | Behaviour with no network |
|---|---|---|
| Read cached snapshot | Under 1 ms | Works, using the last known values |
| No snapshot, first launch after install | 0 ms | Works, using compiled defaults |
| Refresh | Off the critical path | No effect on this launch |

I say the consequence out loud: the app's first launch after install must be
correct with the defaults compiled into the binary, which means every flag's
default is a product decision made at build time, not an accident.

**The error most readers make here.** Blocking on a config fetch "just for a
moment" to avoid a flash of the wrong experience. The moment is a network round
trip, so it is 300 ms at the median and a timeout at the tail, and it turns a
configuration system into the largest single contributor to cold start.

!!! tip "Self-explanation prompt"

    Before Block B, say what has to be true about a flag for it to be safe to
    apply mid-session, and give one example of each answer.

#### Block B. Purpose: bucket so a ramp cannot corrupt the experiment

Two hashes, as in the diagram above: one for eligibility against the rollout
percentage, one for the arm. Unit is the account identifier for anything
measured per person, and the install identifier for onboarding experiments that
happen before sign-in.

I check the ramp property explicitly, because that is what the artefact got
wrong. Going from 50 to 70 percent changes only the eligibility test, and the
eligibility hash for a given user is fixed, so every user who was already in
stays in with the same arm. The added 20 percent are new, and I flag them as a
separate cohort for analysis rather than pooling them.

Then the guard: a sample ratio mismatch check runs before any metric is read. At
200,000 exposed users the standard deviation of an arm count is 223.6, so I set
the automated check to refuse the analysis at more than 4 standard deviations,
which is about a 900 user imbalance, and to page the owner rather than silently
reporting a result.

**The error most readers make here.** Bucketing on a session or a device where
the analysis is per person. The arithmetic then treats one heavy user's twenty
sessions as twenty independent observations, which understates variance and
manufactures significant results out of nothing.

#### Block C. Purpose: make telemetry survive the conditions you are measuring

| Property | Decision | Derivation or reason |
|---|---|---|
| Storage | Durable local queue, written in the same transaction as nothing else | An event must survive a kill one millisecond after it is recorded |
| Cap | 5 MB | 12 KB per session, 5 sessions a day, so 5 MB is about twelve weeks offline |
| Drop policy | Lowest priority, oldest first. Never drop stability or billing events | The events you would drop under pressure are the ones describing the pressure |
| Dedup | Event identifier assigned at creation, server deduplicates | Retries are normal and double counting corrupts every rate |
| Batching | 50 events or 5 minutes of foreground, whichever first, plus a flush on backgrounding | 15 KB per batch, roughly 3 KB compressed, one request instead of 50 |
| Clock | Send device wall clock, monotonic uptime, and per-event age | Lets the server reconstruct order despite a wrong or changed device clock |
| Battery | Low priority events wait for an existing wakeup, from Module 13 | Telemetry should not be a wakeup source of its own |

The exposure event is special and gets the highest non-crash priority, because
an experiment with missing exposures is not an experiment.

**The error most readers make here.** Holding the queue in memory and writing it
on backgrounding. Process death does not call your backgrounding hook reliably,
so the sessions that end in a kill lose all their events, and those are exactly
the sessions where something went wrong.

!!! tip "Self-explanation prompt"

    Say why dropping the oldest events under pressure is usually right, and name
    one event type where dropping the newest would be better.

#### Block D. Purpose: derive the rollout ladder from detection power

Using the derivation above, 9,525 sessions in the new build detects a 0.3 point
crash regression:

| Stage | Daily sessions in stage | Detection time | Hold | Why the hold is longer |
|---|---|---|---|---|
| 1 percent | 10,000 | 23 hours | 24 hours | Nothing to add, detection is the constraint |
| 5 percent | 50,000 | 4.6 hours | 12 hours | Covers a full day and night usage cycle |
| 20 percent | 200,000 | 1.1 hours | 12 hours | Same reason, plus time for support signal to arrive |
| 50 percent | 500,000 | 27 minutes | 24 hours | Crosses a weekday or weekend boundary |
| 100 percent | 1,000,000 | 14 minutes | Monitor for 7 days | Long-tail issues on rare devices |

Halt criteria are written before the rollout starts, not decided during it:

- Crash-free sessions below the previous release by more than 0.3 points at any
  stage: halt and roll back.
- Hang or not-responding rate up at all: halt, because these are main-thread
  regressions and they get worse with slower devices, which arrive later in the
  rollout.
- A single new crash signature above an assumed 0.05 percent of sessions: halt.
- Support contact rate up by an assumed 20 percent: investigate before advancing.

**The error most readers make here.** Advancing on a schedule instead of on a
signal. A rollout that moves to the next stage every 12 hours regardless of what
the numbers say is a countdown timer, not a safety mechanism.

#### Block E. Purpose: decide what you do when the flag is not enough

Two mechanisms, with the boundary between them stated:

| Situation | Mechanism |
|---|---|
| A feature is harmful and can be turned off | Kill switch, applied live, no update needed. This must be tested by flipping it in production on a schedule |
| The client is producing bad data or bad requests but the feature cannot simply be off | Remote configuration that changes behaviour: lower a sampling rate, raise a backoff, disable a code path |
| The build is unsalvageable and the store version cannot be recalled | Halt rollout, promote the previous version where the platform allows, and use a soft update prompt to move users forward |
| The client version is incompatible with a required change | Hard block, sized against adoption, with a working update path and a clear message |

Sizing the last row honestly: a hard block 30 days after release still strands
10 percent of installs, which at 1,000,000 daily actives is 100,000 people. That
number belongs in the proposal, because it converts an engineering convenience
into a business decision.

**The error most readers make here.** Building a kill switch and never testing
it. A switch that has not been flipped in production is a switch that has a
50 percent chance of not working during the incident that needs it, and the
incident is the worst time to discover the flag never reached the evaluation
path.

### 17e. Fade the scaffold

**Fade 1: a pricing experiment on a subscription paywall.** Three price points,
revenue is the metric, and the finance team needs the result to be defensible.
Same five-block shape, last block blank.

- Block A, evaluation path: prices come from the cached snapshot with compiled
  defaults, and the price shown must never change mid-session. This flag is not
  live-safe, because a price that changes while a user is deciding is both a bad
  experience and a possible compliance problem.
- Block B, bucketing: account identifier where available, install identifier
  before sign-in, with pre-login exposures excluded from the revenue analysis. A
  dedicated salt, because correlating price with any other experiment would make
  both unreadable.
- Block C, telemetry: paywall view, price shown, purchase attempted and purchase
  completed all become high priority durable events. Purchase events are never
  dropped, and they are reconciled against the store's server notifications
  rather than trusted from the client.
- Block D, rollout: this is an experiment rather than a release, so the ladder is
  about revenue risk rather than crash detection. Start at an assumed 5 percent
  and size the duration by the metric's variance, not by crash power, because
  revenue per user is far noisier than a crash rate.
- Block E, what you do when the flag is not enough: **you write this block.**

??? note "Show answer"

    The kill switch here is not "turn the feature off", because the paywall
    cannot be off. It is "return every eligible user to the control price", which
    must be a single configuration change that applies at next launch and is
    tested before the experiment starts.

    Second, a hard rule that is specific to pricing: a user who has already seen a
    price must keep seeing it for the duration of their decision window. So the
    revert applies to new exposures only, and existing exposures are honoured
    until an assumed 7 day window expires. Silently changing a quoted price is
    the kind of thing that generates complaints and regulatory attention rather
    than a support ticket.

    Third, client telemetry is not the source of truth for revenue. Purchases are
    confirmed by server-to-server notifications from the store, and the client
    event is used only to attribute the purchase to a variant. If the two
    disagree, the server wins and the discrepancy rate is itself a monitored
    metric, because a rising discrepancy means the attribution is broken.

    Finally, the unrecoverable case: if the experiment is found to have charged a
    group incorrectly, no flag fixes it. The response is a refund process and a
    server-side reconciliation, which is why the experiment design should have
    limited exposure to 5 percent and why the finance team was in the room.

**Fade 2: a feature flag system delivered as an SDK to third-party apps.** Host
apps embed it, and you cannot see their code. Last two blocks blank.

- Block A, evaluation path: identical rules, enforced harder. Evaluation is
  synchronous from a local snapshot with host-supplied defaults, and the SDK
  must never block the host's startup. If the host has no snapshot, evaluation
  returns the default the host passed at the call site, which means the host's
  code is correct even if your SDK never initialises.
- Block B, bucketing: the SDK cannot choose the unit, because only the host has
  a notion of a user. The host supplies the unit identifier, and the SDK provides
  the hash and the two-stage scheme, plus a documented guarantee that the same
  unit and salt always produce the same bucket in every SDK version.
- Block C, telemetry: **you write this block.**
- Block D, rollout: **you write this block.**

??? note "Show answer"

    Block C, telemetry. The SDK is spending someone else's battery and data
    allowance, so exposure events must be small, batched, and attached to an
    existing wakeup rather than scheduling their own. Give the host explicit
    control: a configurable flush policy, an option to supply their own transport
    so events ride their existing telemetry pipeline, and an option to disable
    collection entirely while still evaluating flags.

    Two hard rules. The queue is capped in bytes at a small default, an assumed
    1 MB, because the host's storage is not yours to fill. And the SDK must never
    collect anything about the host app beyond what is needed for exposure
    attribution, because the host's privacy disclosures now include your
    behaviour and a surprise there is a serious problem for them.

    Block D, rollout. You have no rollout control at all, which is the defining
    constraint. Your code ships when the host ships, which may be months after
    you release, and multiple versions of your SDK run in the field
    simultaneously and indefinitely.

    So the rollout mechanism moves server side: new evaluation behaviour is
    introduced behind a server-controlled capability that old SDK versions ignore
    and new ones honour, and it is ramped by SDK version and host application
    identifier. The bucketing function specifically can never change, because a
    host running version 1 and a host running version 5 must place the same user
    in the same bucket, or a cross-app experiment silently splits.

    The rollback story is the same shape: you cannot recall a version, so every
    behaviour that might need reverting must be server-switchable, and the SDK
    must degrade to its previous behaviour rather than to an error when a
    capability is withdrawn.

### 17f. Check yourself

**Q1.** Your experiment shows a 2 percent lift with a comfortable significance
level, and then you notice the treatment arm has 3 percent more users than the
control. State what you do and why, with the arithmetic for 200,000 exposed
users.

??? note "Show answer"

    You void the result and investigate the split before looking at the metric
    again. At 200,000 exposed users, an even split expects 100,000 per arm with a
    standard deviation of sqrt(200,000 x 0.25) = 223.6. A 3 percent excess is
    about 3,000 users, which is 13 standard deviations. That is not chance under
    any threshold anyone uses.

    The reason it voids the lift, rather than merely being untidy, is that
    whatever mechanism added users to one arm probably also selected them. Common
    causes: a crash in the control arm removing users before their exposure is
    logged, a filter applied in one arm only, or a bucketing bug. In every one of
    those, the arms are no longer comparable populations, so the 2 percent may be
    entirely an artefact of who is in each group.

**Q2.** A teammate wants to apply remote config changes immediately rather than
at next launch, because "waiting is a worse experience". Give the two categories
of flag where they are right and the general rule you would write down.

??? note "Show answer"

    They are right for kill switches and for operational parameters. A kill
    switch that waits for the next launch is useless during an incident, and a
    backoff multiplier or sampling rate that only takes effect tomorrow does not
    help the server that is overloaded today. Both share a property: applying
    them mid-session cannot produce an inconsistent experience, because they
    remove behaviour or change a rate rather than swapping an interface.

    The general rule: a flag may apply live only if flipping it at an arbitrary
    moment leaves the interface and the in-progress user task coherent. Anything
    that changes layout, copy, pricing, or the shape of a flow applies at next
    launch, because the alternative is a user watching a screen rearrange itself
    mid-decision and an experiment where the exposure record no longer describes
    what the user saw.

### 17g. When not to use this

**A remote configuration system.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You have shipped at least one incident that a flag would have contained, or you run experiments, and you can name an owner for the flag inventory |
| Scale threshold below which it is over-engineering | A small app with weekly releases and few users can change behaviour by shipping. Below roughly a fortnightly release cadence, the flag system costs more than the delay it removes |
| Cheaper alternative | A single server-provided JSON document with a handful of values, cached on disk, with compiled defaults. It is a day of work and it covers kill switches, which is the case that actually matters |

**Client-side experimentation infrastructure.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You have enough traffic to detect the effects you care about. Compute it: an effect of 1 percent on a metric with your observed variance needs a specific sample, and if you do not have it, the infrastructure will produce noise |
| Scale threshold | Below roughly tens of thousands of exposed users per arm, most experiments on typical product metrics are underpowered, and running them teaches the team to trust noise |
| Cheaper alternative | Staged rollout with careful before-and-after monitoring, plus qualitative research. Weaker causally, honest about it, and far cheaper |

**A staged rollout ladder with derived holds.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Enough daily sessions that a 1 percent stage reaches your detection threshold in a reasonable time. Below that, the early stages detect nothing and merely delay |
| Scale threshold | At an assumed 10,000 daily sessions, a 1 percent stage is 100 sessions a day, which detects nothing. Start at 20 or 50 percent and rely on monitoring plus a fast rollback |
| Cheaper alternative | Two stages, an internal or beta channel and then everyone, with the same halt criteria written down. The criteria matter more than the number of stages |

**A hard forced update.**

| Question | Answer |
|---|---|
| Measurement that justifies it | A security issue, or a server contract you genuinely cannot keep serving, AND you have computed how many installs the block strands at your adoption curve |
| Scale threshold | For anything short of that, the cost is direct: 10 percent of installs at 30 days, and those users skew toward old devices and constrained networks |
| Cheaper alternative | Keep serving the old contract behind a translation layer on the server, and use a soft prompt. The server-side code path is usually cheaper than the lost users |

---

## Module 18. Library and SDK design

**Time: 45 to 55 minutes reading, 35 to 45 minutes practice.**

Component-scale prompts are common in mobile rounds and are where most
candidates are least prepared. The shift is that your code runs inside someone
else's process, on their startup budget, their battery, their crash rate and
their support queue, and you cannot see their code. Every rule below follows
from that.

### 18a. Predict first

Read this artefact and commit to an answer before continuing.

```
+---------------------------------------------------------------+
| PROPOSED SDK SURFACE                                          |
|   Analytics.init(context, apiKey)   static, called at start   |
|      opens a database             8 ms                        |
|      reads a config file          4 ms                        |
|      starts a network request     async                       |
|   Analytics.track(name)             returns void              |
|   Analytics.setListener(l)          l is invoked on an        |
|                                     internal worker thread    |
|   The public Event type exposes a third-party JSON type       |
| Host facts                                                    |
|   Host cold start budget 500 ms, currently at 430 ms          |
|   Storage p99 latency is about 10x the median                 |
+---------------------------------------------------------------+
Caption: a proposed public surface with its startup cost, plus the
two host facts that decide whether it is acceptable.
```

**Question.** Compute the p99 startup contribution, and name the two support
tickets that arrive in the first week.

??? note "Show answer"

    **Startup.** The main-thread work is 8 + 4 = 12 ms at the median, taking the
    host from 430 ms to 442 ms, inside the 500 ms budget. At the p99 of storage
    latency, the same work is 80 + 40 = 120 ms, taking the host to 550 ms, which
    is 10 percent over budget. That is a tail regression the SDK author will
    never see on their own device and the host team will attribute to their own
    code.

    **Ticket one: "your SDK crashes our app".** The listener is invoked on an
    internal worker thread. The host does the obvious thing and updates a view
    inside the callback, which is a crash on both platforms. The SDK is
    technically not at fault and is entirely to blame, because it never stated a
    threading contract and never let the caller choose.

    **Ticket two: "we cannot upgrade our JSON library".** Exposing a third-party
    type in a public API drags that dependency into every host, so the host's own
    version must match yours. The moment two SDKs disagree about a shared
    dependency version, the host cannot build, and they will remove whichever SDK
    is easier to remove.

    **A third, for credit.** `track` returns void, so when events do not arrive
    the host has no way to tell whether the SDK rejected them, queued them, or
    dropped them. The result is a support thread that cannot be resolved by
    either side.

**The five budgets a host lends you, and what to ask for.**

| Budget | Ask for | Why this number is reasonable |
|---|---|---|
| Main thread at startup | Under 10 ms | With four SDKs at 10 ms each, that is 40 ms, which is 8 percent of a 500 ms budget |
| Steady-state memory | Under 1 MB | The host's heap ceiling is shared, and being a memory contributor makes them a kill candidate |
| Binary size | Under an assumed 300 KB | Install size affects download completion on constrained networks, and hosts audit it |
| Background wakeups | Zero of your own | From Module 13, a wakeup costs about 5.6 J. Ride the host's activity instead |
| Crash budget | Zero | An SDK crash is indistinguishable from a host crash in the host's dashboards |

**Public API rules, each preventing a named failure.**

| Rule | Prevents |
|---|---|
| One entry point, configured with an options object | An expanding set of static setters that must be called in a particular order |
| No required initialisation on the startup path. Record the config, defer the work | The p99 startup regression computed above |
| Calls before initialisation are safe and buffered, not crashes | A crash caused by the host's own initialisation ordering, which you cannot control |
| Double initialisation is idempotent | Crashes in hosts that initialise from two entry points, which is common in apps with multiple launch paths |
| Never expose a third-party type in a public signature | Version conflicts that make your SDK unremovable-but-unupgradable |
| Every operation that can fail returns something inspectable | Unresolvable support threads |

**The threading contract, written as a table in your documentation.**

| Method | Safe on the main thread | Blocks | Callback arrives on |
|---|---|---|---|
| `configure(options)` | Yes | No | Not applicable |
| `track(event)` | Yes | No, enqueues and returns | Not applicable |
| `flush(completion)` | Yes | No | The executor supplied in options, defaulting to main |
| `identity(user)` | Yes | No | Not applicable |
| `diagnostics()` | Yes | No, reads an in-memory snapshot | Not applicable |

Two rules to state out loud: nothing in the public surface blocks the calling
thread, and no callback arrives on a thread the caller did not choose. Providing
a default of "main thread" and an option to supply an executor covers both the
casual integrator and the sophisticated one.

**The error model, in three classes.**

| Class | Example | Response |
|---|---|---|
| Programmer error | Configuring with an empty key, calling with a null event name | Assert loudly in debug builds, degrade silently in release. Never crash a user's app for a developer's mistake |
| Expected runtime condition | Offline, queue full, rate limited | A typed result the caller can inspect, plus a documented default behaviour |
| Internal failure | Your own database is corrupt | Contain it. Disable the affected subsystem, report to your own telemetry, keep the host running |

The invariant that summarises all three: an SDK may fail, and may say it failed,
but must never take the host process down. A host that has been crashed once by
a vendor SDK removes it, and no amount of feature work wins them back.

**Extensibility: exactly three seams, and the reason to stop there.**

| Seam | Why it earns its place |
|---|---|
| Network transport | Hosts have their own client with their own interceptors, proxies, pinning and metrics |
| Storage location and encryption | Hosts have policies about where data lives and whether it is encrypted |
| Clock and logger | Makes your library testable, and lets the host route your logs into their system |

Every additional seam is a permanent commitment: it appears in the public
surface, it constrains your internals forever, and it must keep working across
every future version. Say no by default and add a seam when two separate hosts
have asked for the same one.

**Initialisation cost, designed down to nothing.**

- At `configure`, store the options and return. Do no disk or network work.
- Schedule the real setup (open storage, load the queue, read config) on a
  background thread after the host's first frame, or lazily on first use,
  whichever comes first.
- Buffer calls that arrive before setup completes in a small bounded in-memory
  list, an assumed 100 entries, then hand them to the queue once it exists.
- Measure the result the way a host would: cold start with and without the SDK,
  on a mid-tier device, reported at p50 and p90.

That takes the artefact's 12 ms median and 120 ms tail on the main thread to
effectively zero, and the cost is a few hundred bytes of buffer.

**Binary size, which hosts really do audit.**

| Lever | Typical effect | Cost to you |
|---|---|---|
| Do not depend on a general purpose serialisation or networking library | Often the single largest saving, since these are the heaviest transitive dependencies | Hand-written encoding for a small schema |
| Split optional features into separate modules | Hosts pay only for what they use | More artefacts to version and test |
| Keep the public surface small | Less code that cannot be stripped by dead code elimination | Fewer conveniences |
| Ship a size report with every release | Turns a suspicion into a number | A build step |

**Versioning and deprecation, as a written contract.**

- Semantic versioning applies to the public surface, not to your internals, and
  that distinction must be documented or every patch release becomes a debate.
- Binary compatibility matters where hosts consume prebuilt artefacts, and it is
  broken by changes that look harmless in source, such as adding a parameter with
  a default or changing a type in a signature.
- Deprecation runs on a clock: mark deprecated with a replacement named in the
  message, keep it working for an assumed two minor versions or six months
  whichever is longer, then remove in a major version.
- Never expose a third-party type. If you must interoperate, provide a small
  adapter module that hosts opt into.

**Surviving a hostile host.** Your code will run under all of these, and each one
should be a test:

| Condition | Required behaviour |
|---|---|
| Called before initialisation | Buffer or no-op, never crash |
| Initialised twice, from two entry points | Idempotent |
| Process killed mid-operation | Durable queue, resumable, idempotent send |
| Device clock set to last year | Events still ordered correctly using monotonic uptime |
| No network for a week | Bounded queue, graceful drop policy, no growth |
| Host calls from many threads at once | Thread safe, documented, and tested under contention |
| Host on a two year old SDK version | Server still accepts its payloads |

**Diagnosability inside a host you cannot debug.** Provide a `diagnostics()`
call returning a snapshot: SDK version, initialisation state, queue depth,
oldest queued event age, last error, and last successful send. Add a verbose
logging toggle and an in-memory ring buffer of the last 100 operations. This
turns a week of email into one screenshot, and it is the single highest-leverage
feature for support cost.

```
+---------------------------------------------------------------+
| PUBLIC SURFACE   configure, track, flush, identity, diagnose  |
| no blocking, no third-party types, caller chooses the thread  |
+-------------------------------+-------------------------------+
                                |
                                v
+---------------------------------------------------------------+
| DEFERRED CORE    starts after first frame or on first use     |
| bounded buffer > durable queue > batcher > transport seam     |
+-------------------------------+-------------------------------+
                                |
        +-----------------------+-----------------------+
        v                       v                       v
+----------------+   +--------------------+   +------------------+
| STORAGE SEAM   |   | CLOCK AND LOGGER   |   | HOST TRANSPORT   |
| location, key  |   | injectable, fake   |   | optional, host   |
| supplied       |   | in tests           |   | client if given  |
+----------------+   +--------------------+   +------------------+
Caption: a thin non-blocking public surface over a core that starts
late, with exactly three replaceable seams and nothing else.
```

### 18d. Worked example

**The prompt.** "Design the public API for an analytics SDK that third-party
apps will embed."

Inputs first:

| Input | Value | Status |
|---|---|---|
| Host cold start budget | 500 ms | GIVEN |
| Startup allowance for this SDK | Under 10 ms of main thread | ASSUMED, derived from four SDKs sharing 8 percent of the budget |
| Events per session | 40 at 300 bytes | ASSUMED, from Module 17 |
| Queue cap | 1 MB | ASSUMED, deliberately smaller than a first-party app's, because the storage is the host's |
| Binary size budget | 300 KB | ASSUMED, and stated in the release notes so hosts can hold you to it |

#### Block A. Purpose: write the surface backwards from the caller's worst day

I start from the integration failures I expect and design the surface so each
one is impossible:

| Feared failure | Surface decision |
|---|---|
| "Your SDK slowed our startup" | `configure` stores options and returns. No disk, no network |
| "Your SDK crashed us" | Calls before configure are buffered. Double configure is idempotent. No exception escapes the public surface |
| "Events are missing and we cannot tell why" | `diagnostics()` returns queue depth, oldest event age, last error, last successful send |
| "We cannot upgrade our dependencies" | No third-party type appears in any public signature |
| "It sends data we did not expect" | Collection is explicit. There is no automatic screen or interaction capture unless the host enables it |

The resulting surface is five calls: `configure`, `track`, `identity`, `flush`,
`diagnostics`. I say the number out loud, because a small surface is a design
claim rather than an omission, and it is the thing a reviewer can check.

**The error most readers make here.** Designing the surface from the feature
list. The feature list produces one method per capability and a static setter
for every option. Designing from the caller's failures produces a surface that
is small because each addition has to defend itself.

!!! tip "Self-explanation prompt"

    Before Block B, say why buffering calls made before `configure` is better
    than throwing, given that throwing would find the host's bug faster.

#### Block B. Purpose: state the threading contract as part of the API

Documented in a table shipped with the SDK, and enforced in code:

| Guarantee | Mechanism |
|---|---|
| No public method blocks the caller | `track` writes to an in-memory ring and returns. The durable write happens on the SDK's own serial queue |
| Callbacks arrive on the caller's chosen executor | `configure` takes an optional executor. The default is the main thread, documented |
| The SDK is safe to call from any thread | One serial queue owns all mutable state. Public methods only enqueue |
| The SDK never touches the main thread on its own initiative | Setup runs after first frame on a background thread |

The serial queue is the whole concurrency design, and it is deliberately boring.
A single owner thread for mutable state removes an entire class of bugs that
would otherwise be reported as flakiness by hosts you cannot debug.

**The error most readers make here.** Offering a synchronous variant "for
convenience". Every synchronous method in an SDK eventually gets called on the
main thread by a host under deadline pressure, and then your library owns their
application-not-responding rate.

#### Block C. Purpose: define what fails, how it is visible, and what never happens

| Situation | Behaviour | Visible how |
|---|---|---|
| Empty or malformed key | Assert in debug, disable collection in release | `diagnostics().lastError`, plus a single log line |
| Queue full | Drop by priority, oldest first within class | A counter in diagnostics, and a dropped-events figure sent with the next batch |
| Storage unavailable or corrupt | Fall back to an in-memory queue with a smaller cap, rebuild storage on next launch | Diagnostics state changes to degraded |
| Network failing | Exponential backoff with jitter, capped at an assumed 30 minutes | Last successful send timestamp |
| Any internal exception | Caught at the public boundary, reported to our own telemetry, never rethrown | Counter only |

The dropped-events figure is worth its own sentence: sending the count of what
you dropped, with the next successful batch, means the host's analysis can tell
the difference between "nothing happened" and "we lost the data". Silent loss
makes every downstream number untrustworthy.

**The error most readers make here.** Rethrowing an internal error to be
"transparent". Transparency for an SDK means a value the caller can inspect, not
an exception crossing into code that has no idea what to do with it.

!!! tip "Self-explanation prompt"

    Say why reporting the number of dropped events is more valuable than raising
    the queue cap, and what you would do if the dropped count were consistently
    above zero.

#### Block D. Purpose: budget startup, memory and binary size, then measure

| Budget | Design that meets it | How it is measured |
|---|---|---|
| Under 10 ms main thread at startup | `configure` does nothing but store options, roughly microseconds. Setup deferred | Cold start of a sample host with and without the SDK, p50 and p90, mid-tier device |
| Under 1 MB steady memory | Bounded in-memory ring of 100 events, single batch buffer, no caches | Heap snapshot of the sample host with the SDK idle for 5 minutes |
| Under 300 KB binary | No general purpose serialisation dependency, hand-written encoder for a fixed schema, optional features in separate modules | A size report generated on every release build |
| Zero background wakeups | Flush on host foreground, backgrounding, and when the host's own network activity is detected | Wakeup counter in the sample host over 24 hours |

I state the trade I made: hand-writing the encoder costs me a day and some
ongoing care, and it buys the largest single line item in the size budget. That
is a defensible engineering trade for a library and a bad one for an app, and
the difference is that a library's cost is multiplied by every host.

**The error most readers make here.** Measuring size and startup on the SDK in
isolation. The number that matters is the delta in a real host application with
its own dependencies, because dependency overlap can make your contribution
smaller or much larger than it looks alone.

#### Block E. Purpose: write the versioning contract before the first release

| Rule | Detail |
|---|---|
| What semantic versioning covers | The public surface listed in the documentation. Internals may change in any release |
| Binary compatibility | Maintained within a major version for hosts consuming prebuilt artefacts |
| Deprecation clock | Deprecated with a named replacement, supported for two minor versions or six months, whichever is longer, removed only in a major version |
| Server compatibility | The ingestion endpoint accepts payloads from every SDK version released in the last assumed 24 months. This is a server cost you accept when you ship a library |
| Third-party types | Never in a public signature. Adapters ship as separate optional modules |

The server compatibility row is the one candidates forget, and it is the largest
long-term cost of shipping an SDK. Hosts upgrade slowly, some never, so your
ingestion path carries every payload shape you have ever released. Deciding the
window explicitly, and instrumenting the share of traffic from each version, is
how you eventually retire one.

**The error most readers make here.** Treating a major version bump as a way to
clean up. Hosts do not upgrade major versions on your schedule, so a major
version that drops old behaviour splits your user base into two populations you
must both support, which is more work than the cleanup saved.

### 18e. Fade the scaffold

**Fade 1: an image loading library.** Hosts pass a URL and a view, and expect a
correctly sized image with caching and cancellation. Same five-block shape, last
block blank.

- Block A, surface backwards from failures: the feared failures are memory
  kills, wrong images appearing in recycled views, and jank. So the surface takes
  the target view (to derive size and to key cancellation), the request is
  cancelled automatically when the view is recycled or detached, and decoding
  always targets the view's measured size rather than the source dimensions.
- Block B, threading contract: `load` is main-safe and returns immediately.
  Decode happens on a bounded pool. The completion callback is delivered on the
  main thread by default, because it will be used to update a view.
- Block C, error model: a missing image is an expected condition and produces a
  placeholder plus an inspectable result, never an exception. A decode failure
  disables that entry rather than the library.
- Block D, budgets: memory cache capped in bytes at a fraction of the host's
  heap that the host can configure, disk cache capped and in the cache
  directory, zero background work, and no general purpose dependencies.
- Block E, versioning contract: **you write this block.**

??? note "Show answer"

    The public surface is small and the risky part is the configuration types:
    the transformation interface, the cache interface and the transport interface
    are seams, and seams are permanent. Semantic versioning covers those
    interfaces as strictly as it covers the methods, because a host implementing
    a custom transformation is broken by an added method on that interface.

    So the versioning rule that matters here: seams evolve by adding new
    interfaces, never by adding requirements to existing ones. If a
    transformation needs a new capability, it becomes an optional secondary
    interface that the library checks for, so existing implementations keep
    compiling and running.

    Binary compatibility deserves emphasis because image libraries are usually
    consumed as prebuilt artefacts and often sit in the dependency graph twice via
    other libraries. Exposing no third-party types is not merely tidy here, it is
    what prevents the diamond dependency that stops a host from building at all.

    Finally, an image library has an unusual deprecation hazard: the disk cache
    format. Changing it invalidates every user's cache, which produces a burst of
    network traffic on upgrade day across every host. The rule is that the cache
    format is versioned and a new version starts empty rather than migrating, and
    the release notes state the expected one-time traffic increase so hosts are
    not surprised.

**Fade 2: a feature flag SDK for third-party hosts.** Evaluation must work
offline and must never block the host. Last two blocks blank.

- Block A, surface backwards from failures: the feared failures are a blocked
  startup, a wrong value before the snapshot loads, and inconsistent values
  within a session. So evaluation is synchronous against a local snapshot, every
  call site passes its own default, and values are frozen for the session unless
  the flag is declared live-safe.
- Block B, threading contract: evaluation is main-safe and lock-free against an
  immutable snapshot swapped atomically. Refresh happens on a background thread.
  Change callbacks arrive on a caller-supplied executor.
- Block C, error model: **you write this block.**
- Block D, budgets: **you write this block.**

??? note "Show answer"

    Block C, error model. The defining property is that evaluation cannot fail.
    There is no error return on `boolValue(forKey:default:)`, because a host that
    has to handle an error at every call site will handle it badly. Every failure
    resolves to the caller's default, and the fact that a default was used is
    recorded in diagnostics rather than returned.

    Failures that do need visibility: a refresh that has not succeeded for a long
    time, a snapshot that failed to parse, and an authentication failure. These
    surface through `diagnostics()` and an optional delegate, so an integrator
    who wants to alert can, and one who does not is never interrupted. The
    snapshot age is the single most useful field, because a flag system serving
    three-week-old values is broken in a way nothing else reveals.

    One hard rule: a parse failure of a new snapshot must keep the previous
    snapshot, never fall back to empty. Falling back to empty means every flag
    reverts to its compiled default simultaneously across every device that
    fetched the bad payload, which converts a formatting mistake into a
    product-wide behaviour change.

    Block D, budgets. Startup is the whole game: `configure` reads nothing.
    The snapshot loads on a background thread, and until it does, evaluation
    returns the caller's defaults. That is correct rather than merely fast,
    because the host's code stated its default at the call site.

    Memory: the snapshot is small, an assumed 200 flags at 100 bytes is 20 KB, so
    hold it fully in memory as an immutable map. Binary: no serialisation
    dependency, a compact hand-parsed format. Network: one small conditional
    request per foreground session at most, with a validator so an unchanged
    snapshot costs a few hundred bytes. Wakeups: none of your own, ever, because
    a flag SDK that wakes a host's process to refresh flags is spending the
    host's battery on your convenience.

### 18f. Check yourself

**Q1.** Your SDK exposes `Result<Void, AnalyticsError>` from `track`. A host
reports that their code is now cluttered with error handling for a call they do
not care about. Give your two options and say which you would pick and why.

??? note "Show answer"

    Option one: make `track` return nothing and move all failure information into
    `diagnostics()` and an optional delegate. Option two: keep the result but make
    it discardable so callers who ignore it produce no warning, and document that
    ignoring it is the expected default.

    I would pick option one. `track` is a fire-and-record call made from many
    places, and its failures are aggregate rather than per-call: the interesting
    facts are "the queue is full" and "nothing has sent for six hours", which are
    properties of the SDK, not of the individual event. Returning a per-call
    result puts a decision at every call site that no caller can meaningfully
    make, which is the definition of a bad API. The counts belong in diagnostics
    and in the dropped-events figure sent with the next batch.

**Q2.** A host asks you to add an interface so they can supply their own event
serialisation. Give the questions you would ask and the criterion you would use
to decide.

??? note "Show answer"

    Questions: what problem does the current format cause them, is it size, a
    compliance requirement, or compatibility with an internal pipeline, and could
    a transport seam solve it instead by letting them send the payload wherever
    they like in whatever wrapper they want.

    The criterion is that a seam is a permanent, versioned part of the public
    surface, so it needs to be justified by more than one host and by a need that
    cannot be met by an existing seam. If the real requirement is "our data must
    go to our own endpoint", the transport seam already covers it. If the
    requirement is "our security team must review the exact bytes", documentation
    and a debug mode that prints the payload covers it.

    If two independent hosts need genuinely different on-the-wire encoding, I
    would add it, and I would add it as a new optional interface rather than as a
    parameter on an existing one, so no existing implementation breaks.

### 18g. When not to use this

**Shipping an SDK at all.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Multiple integrators would otherwise reimplement non-trivial protocol logic, such as durable queueing, retries and idempotency, and get it wrong differently each time |
| Scale threshold below which it is over-engineering | If your integration is one HTTP call with a header, an SDK is a maintenance obligation that replaces ten lines of the host's code with a dependency, a version, and a support channel |
| Cheaper alternative | A documented endpoint, a small sample project, and a copy-and-paste snippet. Hosts prefer it more often than SDK authors expect |

**An extensibility seam.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Two or more independent hosts have asked for the same replacement, and no existing seam covers it |
| Scale threshold | With one requester, a configuration option or a documented workaround is cheaper than a permanent interface you can never change |
| Cheaper alternative | An option in the configuration object, or a targeted callback. Both can be removed in a major version. An interface implemented by hosts effectively cannot |

**Your own thread pool inside an SDK.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You have concurrent CPU-bound work, such as decoding, and can show contention with a single serial queue in a real host |
| Scale threshold | For queue-and-send workloads, one serial queue is sufficient and removes an entire class of concurrency bugs the host will report as flakiness |
| Cheaper alternative | A single serial queue for state, plus the host's own executor when one is supplied. Fewer threads in someone else's process is a feature |

**Automatic instrumentation that captures screens and interactions.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Hosts explicitly ask for it, and you can enumerate exactly what is captured and let them exclude views and fields |
| Scale threshold | Off by default, always. Capturing by default puts data in your pipeline that the host never disclosed to their users, which is their legal exposure and your reputational one |
| Cheaper alternative | Explicit instrumentation with good ergonomics, plus an opt-in automatic mode with a documented capture list and exclusion controls |

---

## Module 19. Interview mechanics

**Time: 35 to 45 minutes reading, 30 to 40 minutes practice.**

The hub page owns the general delivery clock for system design rounds. This
module is the client-specific version: what to draw, in what order, what to
leave off, and the exact sentence that turns "I am running out of time" from a
failure into a demonstration of judgement.

### 19a. Predict first

Read this artefact and commit to an answer before continuing.

```
+---------------------------------------------------------------+
| TRANSCRIPT TIMELINE, 45 minute mobile round                   |
|   00 to 18   clarifying questions, fourteen of them           |
|   18 to 38   one diagram drawn, 22 boxes, no numbers anywhere |
|   38 to 43   interviewer asks about offline behaviour.        |
|              Candidate answers "we would cache it"            |
|   43 to 45   candidate asks their questions                   |
| The score sheet has five dimensions                           |
|   framing, budgets, architecture, depth, communication        |
+---------------------------------------------------------------+
Caption: how forty-five minutes were actually spent, next to the
five things the interviewer has to write a score against.
```

**Question.** Name the dimension that was already unrecoverable at minute 25,
say why, and explain how the diagram made it worse rather than better.

??? note "Show answer"

    **Depth, and it was gone before the candidate noticed.** A genuine deep dive
    needs an uninterrupted stretch of roughly 12 to 15 minutes: state the
    mechanism, put numbers on it, name what breaks, and show the alternative you
    rejected. At minute 25 the diagram was still being drawn, so no 12 minute
    block remained. When the interviewer finally asked about offline at minute
    38, the only available answer was a claim.

    **Budgets scored zero as well**, for a simpler reason: no number appears
    anywhere in the transcript. A client round has four cheap numbers available
    from the first minute (start target, frame budget, bytes per screen, daily
    battery share) and stating them costs thirty seconds.

    **Why the diagram hurt.** Twenty-two boxes is twenty-two obligations. Every
    box invites a question, and the candidate has implicitly promised to be able
    to defend all of them. Breadth also crowds out the only thing that
    demonstrates level, which is depth in one place. Eight boxes plus one
    expanded region scores higher than twenty-two boxes, every time.

    **The clarifying block is the other clue.** Eighteen minutes and fourteen
    questions is 40 percent of the round spent without the design changing
    shape. Clarifying questions earn their place by forking the design, and
    fourteen of them that did not fork anything is a checklist being recited.

**The client round's minute budget, and why it differs from a backend one.** A
backend round spends heavily on capacity estimation and storage topology. A
client round spends that time on state, lifecycle and constraint budgets,
because those are where client systems actually fail.

| Minutes (45) | Minutes (60) | Phase | What is produced |
|---|---|---|---|
| 0 to 2 | 0 to 2 | Frame | One sentence on what you think is being asked, and the confirmation |
| 2 to 7 | 2 to 9 | Requirements and budgets | Four functional bullets and four numbers |
| 7 to 11 | 9 to 14 | The contract | What the client consumes from the server, and the shape of a page |
| 11 to 20 | 14 to 25 | Architecture | Eight boxes, drawn in the fixed order below |
| 20 to 33 | 25 to 45 | Deep dive | One dive at 45 minutes, two at 60 |
| 33 to 40 | 45 to 54 | Failure and degradation | What breaks, how it degrades, what you would measure |
| 40 to 45 | 54 to 60 | Close | What you skipped and why, then questions |

**The drawing order, which is the highest-leverage habit in this module.** Draw
in this sequence, narrating as you go, and stop at eight boxes.

| Order | Box | Why it comes here |
|---|---|---|
| 1 | The surface: the screen and its states (loading, empty, error, offline, partial) | It is the only thing the user experiences, and listing states early prevents the classic "you only designed the happy path" |
| 2 | The data layer boundary: what the interface consumes | Forces you to name a display model, which is where the frame budget is won |
| 3 | The source of truth: the local store | Says out loud that the interface reads from local storage, never from the network |
| 4 | The sync and network edge: outbox, fetch, push | The place every offline question will land |
| 5 | Constraint annotations on the diagram: process death, background limits, memory ceiling | Three short labels that show you design for the operating system, not against it |
| 6 | The deep dive, expanded to one side | Everything above stays small so this can be large |

What to leave off, deliberately and out loud: dependency injection, class names,
the backend's internal services, analytics plumbing, and any box whose only role
is to show that you have heard of it.

**The opening, four sentences.**

- "I read this as a client design problem: the interesting parts are state,
  offline behaviour and the budgets, and I will treat the server as a contract
  we can negotiate."
- "I will spend about five minutes on requirements and numbers, ten on the
  architecture, then take one area deep, and leave the last five for failure
  modes and what I skipped."
- "Two clarifying questions that change the design, then I will start drawing."
- "Stop me if you want the time spent differently."

The fourth sentence is the one people leave out and it is the cheapest point on
the page: it makes the plan negotiable, which turns any later redirection into
collaboration rather than correction.

**Checkpoints, one sentence each, roughly every ten minutes.**

| Moment | Sentence |
|---|---|
| After requirements | "Those are the four numbers I will design against. Anything you would change before I draw?" |
| After the architecture | "That is the whole client in eight boxes. I want to go deep on the sync engine, unless media is more interesting to you." |
| At the descope point | "Two areas left. Here is why I would pick one." |
| Before the close | "I have five minutes. I will spend it on failure modes and then tell you what I skipped." |

**The four numbers to state in the first five minutes.** All four come from
earlier modules and none requires a calculator:

| Number | Typical framing |
|---|---|
| Cold start target by device tier | "Under 500 ms on a mid device, which is the tier where the complaints come from" |
| Frame budget | "11.1 ms at 90 Hz, and I design to 70 percent of that" |
| Bytes per screen on a slow network | "A screen of feed should be under an assumed 300 KB, because at 200 kbit that is 12 seconds" |
| Background battery share per day | "Under 1 percent, which at 41,580 J is about 74 isolated radio wakeups" |

Stating even two of these separates you immediately, because most candidates
design a client with no quantities in it at all.

**Descoping at minute 25, which is a skill rather than an apology.** The rule:
at minute 25 of a 45 minute round, count the areas you still want to cover. If
it is more than one, you are over-scoped and you say so.

The sentence: "I have about fifteen minutes of real content left. There are two
areas I could take deep: the sync engine and the media pipeline. The sync engine
carries the correctness risk, because a wrong conflict rule loses user data,
while the media pipeline carries a performance risk that shows up in a profiler
and is fixable later. I will go deep on sync and give you the media pipeline in
three sentences."

Three things happen in that sentence: you name the choice, you justify it by
consequence rather than preference, and you deliver the discarded area in
summary so it is not simply missing.

**Recovery scripts, verbatim enough to use.**

| Situation | Say |
|---|---|
| Stuck | "Let me state what I do know and where the uncertainty is. I know the write path has to be durable before the interface confirms. What I am unsure about is whether the merge rule should be per field or per row, so let me reason through both quickly." |
| The interviewer disagrees | "That is a better constraint than the one I assumed. Let me redo the affected part: if writes can be lost on reconnect, then the outbox needs a retention policy, which changes the storage tier." |
| The interviewer is running a backend round under a mobile title | "Happy to design the service side. Before I do, is it useful if I spend five minutes on the client contract, since that is where I would normally push back on the response shape? If you would rather I stay server side, I will." |
| You are behind on time | "I am about five minutes behind my plan. I am going to skip the contract sketch and go straight to the architecture, and I will state the contract assumptions as I draw." |
| Asked about the platform you do not use | "On the side I have shipped, the primitive is X and it behaves like this. I would expect the equivalent on your platform to differ in whether the OS can kill the process mid-task, which changes whether I need resumability. Does it?" |

**How the split changes with level.**

| Level | Spends more time on | Spends less time on |
|---|---|---|
| Mid | Architecture and the contract | Failure modes, and choosing between areas |
| Senior | The deep dive and failure modes | Drawing, because the diagram is smaller |
| Staff | Framing, sequencing and what to skip | Drawing, and enumerating components |

```
+---------------------------------------------------------------+
| 1 SURFACE     screen states: loading, empty, error, offline   |
+-------------------------------+-------------------------------+
                                |
+-------------------------------v-------------------------------+
| 2 DISPLAY MODELS   precomputed, bind assigns only             |
+-------------------------------+-------------------------------+
                                |
+-------------------------------v-------------------------------+
| 3 LOCAL STORE     source of truth for the interface           |
+---------------+-------------------------------+---------------+
                |                               |
+---------------v-------+           +-----------v---------------+
| 4 OUTBOX, SYNC EDGE   |           | 5 CONSTRAINTS             |
| push, pull, retry     |           | process death, bg limits, |
|                       |           | memory ceiling            |
+-----------------------+           +---------------------------+
Caption: the drawing order for a client round. Five regions, drawn
top to bottom, with the deep dive expanded beside whichever region
the problem makes interesting.
```

### 19d. Worked example

**The prompt.** "Design an offline-capable notes client." Forty-five minutes,
narrated by the clock.

#### Block A. Purpose: minutes 0 to 7, buy the frame and put numbers on the table

I open with the four sentences, then ask exactly two clarifying questions,
chosen because each one forks the design:

| Question | If A | If B |
|---|---|---|
| Is a note edited by one person on several devices, or shared with others live? | Per-field state sync with version checks, which is a week of work | Operation log with rebase, which is a month, and I would say so |
| What is the longest offline period we support? | 24 hours means a small tombstone horizon and a simple token | 30 days means tombstone retention and a full-resync path, from Module 11 |

Then the four numbers, spoken in about thirty seconds: cold start under 500 ms
on a mid device, 11.1 ms frame budget at 90 Hz designed to 7.8, a note list page
under 100 KB, and background sync under 1 percent of battery which is about 74
isolated wakeups.

**The error most readers make here.** Asking clarifying questions that do not
fork anything ("how many users?" in a client round). The test for a good
clarifying question is that you can state, before asking, what you would draw
differently under each answer. If you cannot, do not ask it.

!!! tip "Self-explanation prompt"

    Before Block B, say which of the two clarifying questions you would drop if
    you only had time for one, and what you would assume instead.

#### Block B. Purpose: minutes 7 to 20, draw in order and stop at eight boxes

I draw the five regions from the diagram above, narrating each as I go, and I
say the two sentences that carry the most weight:

- "The interface reads only from the local store. The network never appears in a
  screen's read path."
- "A user's edit is durable in one transaction with an outbox row before the
  interface says saved."

Constraint annotations go on as short labels rather than boxes: process death
next to the local store, background limits next to the sync edge, and memory
ceiling next to the display models.

At minute 20 I stop drawing and check in: "That is the whole client. I would go
deep on the sync engine, because that is where user data can be lost. Unless you
would prefer the editor or the search index."

**The error most readers make here.** Continuing to draw because there is more
to draw. The diagram is not the answer, it is the map for the deep dive, and
every box after the eighth costs a minute you needed later.

#### Block C. Purpose: minutes 20 to 33, one deep dive with numbers in it

Sync engine, at the depth of Module 11 and no deeper:

| Minute | Content |
|---|---|
| 20 to 23 | The unit of change: per field, with the table of which fields can conflict |
| 23 to 26 | Outbox and idempotency keys generated at enqueue, with the retry duplication failure named |
| 26 to 29 | Change token pull, with the delta versus full arithmetic: 920 bytes steady state against 2 MB |
| 29 to 31 | Conflict resolution by version integer, and the sentence "I will not order anything by device clock" |
| 31 to 33 | Tombstone horizon matched to the 30 day offline promise, at 240 KB per user |

Two decisions I say out loud with the alternative I rejected: I chose state sync
with a per-field rule table over an operation log for everything, because only
the body field needs merging; and I chose server-assigned versions over a
logical clock, because every mutation passes through the server anyway.

**The error most readers make here.** Going deep on the area they find most
comfortable rather than the area the problem makes risky. If asked why you chose
this dive, "it is where data loss lives" is an answer, and "it is what I know
best" is not.

!!! tip "Self-explanation prompt"

    Say what you would have deep dived on instead if the clarifying answer had
    been "notes are shared and edited live by several people", and why the choice
    changes.

#### Block D. Purpose: minutes 33 to 40, failure, degradation and measurement

Three failures, each with the observation that detects it:

| Failure | Detected by |
|---|---|
| A poison outbox entry that never drains | p99 oldest outbox age above 24 hours |
| Tombstone horizon shorter than a client's absence, resurrecting deleted notes | Full-resync rate, plus a resurrection counter on apply |
| Process death mid-apply, leaving the token ahead of the data | Token and data written in one transaction, and a startup consistency check |

Degradation ladder: on low-power mode, defer non-urgent pushes and stop
prefetching attachments; on a metered connection, sync text but not
attachments; under storage pressure, trim caches and never touch unsynced notes.

The one measurement I would put first: outbox depth percentiles, because it is
the single number that turns a class of silent sync failures into an alert.

**The error most readers make here.** Listing failure modes without the
detection. A failure you cannot observe is a failure you will hear about from a
review, and interviewers hear the difference immediately.

#### Block E. Purpose: minutes 40 to 45, close by naming what you skipped

The close is three sentences and a question:

- "I skipped the editor and the search index. Search matters at scale and would
  be a local index rebuilt incrementally on write, which I would size before
  committing."
- "The biggest risk in what I designed is the conflict rule on the body field.
  If I had another week I would instrument concurrent-edit frequency before
  building any merge machinery, because that number decides whether the operation
  log is worth it."
- "The first thing I would measure in production is oldest-outbox-age at p99."
- "What would you have wanted me to spend more time on?"

The middle sentence is the one that reads as senior: naming your own design's
weakest point, with the measurement that would resolve it, is a stronger signal
than any additional component.

**The error most readers make here.** Using the last five minutes to add
components. The close is for judgement, not coverage, and an interviewer with a
score sheet is writing in the communication and depth boxes during exactly these
minutes.

### 19e. Fade the scaffold

**Fade 1: a 60 minute round, "design a photo upload client".** Same shape,
stretched to the 60 minute budget, last block blank.

- Block A, minutes 0 to 9: frame, then two forking questions: are uploads
  expected to complete while the app is closed (which forces a system-managed
  transfer path), and can the server accept chunked resumable uploads (which
  decides whether a failed upload restarts). Numbers: an assumed 4 MB per photo,
  a target of 95 percent of uploads completing without user intervention, a
  1 percent daily battery share, and a cold start budget of 500 ms.
- Block B, minutes 9 to 25: draw the five regions, with the upload queue as the
  centre. Annotate process death next to the queue, because it is the constraint
  that shapes everything here.
- Block C, minutes 25 to 45, two deep dives: resumable chunked upload with
  offsets stored durably per chunk, and the background continuation path with
  its platform fork from Module 15. Numbers: at a 30 percent kill rate per
  attempt, expected attempts are 1 / 0.7 = 1.43, and without resumability that
  is roughly 1.43 uploads of transfer for one upload of content.
- Block D, minutes 45 to 54: failures are duplicate uploads from retried
  sessions, a queue that grows without bound on a long trip, and silent
  deferral. Detections are server-side duplicate rate by session identifier,
  queue depth percentiles, and completion rate within 24 hours.
- Block E, minutes 54 to 60, the close: **you write this block.**

??? note "Show answer"

    Skipped, named honestly: on-device compression policy, editing, and the
    gallery experience. One sentence each: compression is a quality-versus-bytes
    decision I would drive from measured upload success rates by network type
    rather than from a default quality setting.

    Biggest risk in what was designed: the background continuation path, because
    it is the piece that differs most between platforms and is the most likely to
    break silently on an OS update. The measurement that would resolve it is a
    scheduled device test that backgrounds the app mid-upload and verifies
    completion, run on every release on both platforms.

    First thing to measure in production: share of uploads completing without the
    user reopening the app, split by network type and by platform. That single
    number is the product promise and it is the one that regresses invisibly.

    Then the question back: "Would you have wanted the compression policy instead
    of the background path? I picked the background path because a failed upload
    is a lost memory and a badly compressed one is only a worse photo."

**Fade 2: a 45 minute component prompt, "design an image loading library".**
Component prompts spend their time differently: less architecture, more public
surface and more behaviour under stress. Last two blocks blank.

- Block A, minutes 0 to 6: frame it explicitly as a component round, because the
  time split changes. Two forking questions: is this for our app only or for
  third parties (which decides how much of Module 18 applies), and do we control
  the image server (which decides whether resolution negotiation is available).
  Numbers: 2.59 MB per decoded cell image, a 32 MB byte cap, 12 entries.
- Block B, minutes 6 to 18: draw the request lifecycle rather than an
  architecture: request keyed by view plus size, memory lookup, disk lookup,
  network fetch, decode at target size, deliver on main, cancel on recycle. Six
  boxes, and the cancellation path drawn explicitly because it is where the
  design is won.
- Block C, minutes 18 to 32, the deep dive: **you write this block.**
- Block D, minutes 32 to 40, failure and measurement: **you write this block.**

??? note "Show answer"

    Block C, the deep dive. Take memory and cancellation together, since they are
    the same problem seen twice. Decode at the view's measured size, not the
    source size, and show the arithmetic: a 4000 x 3000 source is 48 MB decoded
    against a 2.59 MB target, which is the difference between a working library
    and one that kills the host process.

    Then cancellation with its number: during a fling of 200 rows where 20 stop
    on screen, cancelling on recycle takes 200 decodes down to roughly 20, which
    is 3.0 s of CPU down to 0.3 s and 520 MB of allocation churn down to 52 MB.
    Add the cooperative-cancellation trap: the decoder needs an abort signal or a
    sliced decode loop, otherwise the cancelled work runs to completion anyway.

    Close the dive with the queue policy, which is the detail that shows depth:
    newest-first ordering on the decode queue with a bounded depth, because
    during a fling the newest request is for a row about to appear and the oldest
    is for a row already gone.

    Block D, failure and measurement. Failures: the wrong image appearing in a
    recycled view (a keying bug, detected by an assertion in debug that the
    delivered request identifier matches the view's current one), an unbounded
    disk cache filling the device, and a decode stampede when many identical URLs
    are requested at once.

    The stampede deserves its mechanism: coalesce identical in-flight requests so
    the second and later callers attach to the first rather than starting their
    own fetch and decode. Without it, a grid of the same placeholder image
    decodes it once per cell.

    Measurements: cache hit rate by tier, frames over budget during a fling,
    peak resident memory during a scroll of 500 rows, and decodes started versus
    decodes displayed, which is the direct measure of whether cancellation is
    working.

### 19f. Check yourself

**Q1.** At minute 30 of a 45 minute round the interviewer asks a question that
would take ten minutes to answer properly. Give the sentence you say and explain
what it protects.

??? note "Show answer"

    "That is a ten minute answer and I have fifteen minutes left, so let me give
    you the two minute version and you can tell me whether to expand it: the
    mechanism is X, the number that decides it is Y, and the thing that breaks is
    Z. If that is the most interesting area, I will spend the rest of the time
    there and skip the failure modes section."

    What it protects: the close. Without it you either give a rushed shallow
    answer and lose the depth score, or you give the full answer and lose the
    last five minutes where the judgement signals live. Making the trade explicit
    also hands the choice to the interviewer, who knows which dimension they
    still need to score.

**Q2.** An interviewer with a backend background keeps steering a mobile round
toward server sharding. Give your two-step response and say what each step is
for.

??? note "Show answer"

    Step one, comply and do it well, briefly: "The write path here is a
    per-user change feed, so I would partition by user identifier, which keeps a
    user's changes on one partition and makes the change token a per-partition
    sequence." Two or three sentences. This shows you can hold the conversation
    they want and removes any suspicion that you are avoiding it.

    Step two, offer the trade explicitly: "I am happy to keep going server side.
    I would flag that the client-side risks in this design are larger than the
    partitioning ones, specifically the conflict rule and the background upload
    path, so if you want signal on client depth I would spend the next ten
    minutes there. Your call."

    Step one buys credibility, step two recovers the dimension you are actually
    being scored on. Doing step two without step one reads as deflection, which
    is why the order matters.

### 19g. When not to use this

**The minute budget as written.**

| Question | Answer |
|---|---|
| Measurement that justifies it | A 45 or 60 minute round with a single open prompt and an interviewer who expects you to drive |
| Scale threshold below which it is over-engineering | In a 30 minute screen, this budget does not fit. Compress to: 2 minutes framing, 5 requirements, 8 architecture, 10 one dive, 5 close, and say the plan aloud so the compression is visible |
| Cheaper alternative | Ask at the start how they want the time spent. Many interviewers have a fixed agenda, and following theirs beats following yours |

**The fixed drawing order.**

| Question | Answer |
|---|---|
| Measurement that justifies it | An application-scale prompt where the client architecture is the subject |
| Scale threshold | For a component prompt, the order is wrong: draw the request lifecycle and the public surface instead, because there is no application architecture to show |
| Cheaper alternative | Draw whatever the first deep question is about, then fill in around it. The order exists to prevent aimlessness, not to be obeyed |

**The descope script.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You have more than one remaining area at the two-thirds mark, which is minute 25 of 45 or minute 40 of 60 |
| Scale threshold | If you are on plan with one area left, do not descope. Announcing a cut you did not need reads as anxiety rather than judgement |
| Cheaper alternative | A one-line checkpoint: "I am on time, going into the sync dive now." It gives the interviewer the same chance to redirect at a fraction of the words |

---

## Module 20. Level calibration

**Time: 35 to 45 minutes reading, 30 to 40 minutes practice.**

Levels are not distinguished by how many components appear. They are
distinguished by evidence, by anticipated failure, by the scope of what the
candidate treats as changeable, and by what they deliberately leave out. This
module shows the same prompt answered three ways with the deltas annotated.

### 20a. Predict first

Read these three excerpts, all answering "how would you handle images in the
feed?", and commit to an answer before continuing.

```
+---------------------------------------------------------------+
| EXCERPT X                                                     |
| I would use an image loading library with a memory and disk   |
| cache, and load off the main thread so scrolling stays        |
| smooth.                                                       |
+---------------------------------------------------------------+
| EXCERPT Y                                                     |
| Cells are 1080 by 600, so a decoded bitmap is 2.59 MB. On a   |
| 256 MB heap I cap the memory cache at 32 MB in bytes, which   |
| is 12 entries, about two screens. The rest comes from a 60 MB |
| disk cache sized from a 200 image session working set. I      |
| decode at display size and cancel on view recycle.            |
+---------------------------------------------------------------+
| EXCERPT Z                                                     |
| Same sizing as Y. The larger issue is that the server sends   |
| one resolution to every screen, so we pay bytes and decode    |
| for pixels nobody sees. That is a response contract change    |
| and a migration for old clients, so I would ship the client   |
| caps now and take the contract change next quarter.           |
+---------------------------------------------------------------+
Caption: three answers to the same question, differing in what
each one treats as changeable.
```

**Question.** Rank the three by level and name the single sentence in each that
decides the ranking.

??? note "Show answer"

    **X is a mid answer.** The deciding sentence is the whole thing: it names a
    pattern with no quantity attached. It is not wrong, and an interviewer cannot
    distinguish it from an answer given by someone who has read a blog post. The
    missing element is evidence.

    **Y is a senior answer.** The deciding sentence is "a decoded bitmap is 2.59
    MB, so I cap the memory cache at 32 MB in bytes, which is 12 entries". The
    cap is derived from two device facts, the unit is chosen deliberately (bytes,
    not entries), and the consequence is stated (two screens of history). Y also
    names the two mechanisms that actually prevent the failure: decode at display
    size, and cancel on recycle.

    **Z is a staff answer.** The deciding sentence is "that is a response
    contract change and a migration for old clients, so I would ship the client
    caps now and take the contract change next quarter." Z has changed the
    boundary of the problem from the client to the contract, priced the change in
    organisational terms, and sequenced it. Note what Z did not do: add
    components. Z has fewer moving parts than Y, plus a plan.

**The four dimensions that actually separate levels.**

| Dimension | Mid | Senior | Staff |
|---|---|---|---|
| Scope treated as changeable | The client code | The client and its contract with the server | The contract, the roadmap, and which team pays |
| Evidence | Names patterns | Derives numbers from stated inputs | Derives numbers, then names the measurement that would revise them |
| Failure anticipation | Handles errors that are asked about | Names failure modes and their detection unprompted | Names the failure the organisation will actually hit, which is often process rather than code |
| What is left out | Nothing, tries to cover everything | Skips deliberately and says so | Skips deliberately, says so, and sequences the skipped work |

**The counter-intuitive fact worth internalising: staff answers usually contain
fewer components.** A mid answer adds a cache, a queue and a worker because each
is defensible in isolation. A staff answer removes two of them and explains what
measurement would bring them back. Interviewers read component count as
enthusiasm and restraint as judgement.

**Behaviour by phase, so you can locate yourself.**

| Phase | Mid | Senior | Staff |
|---|---|---|---|
| Framing | Starts designing after one or two questions | Asks two questions that fork the design and says how | States the interpretation, the risk, and what would change the interpretation |
| Budgets | Rarely states numbers | States four numbers and designs against them | States four numbers and says which one is the binding constraint |
| Architecture | Many boxes, evenly weighted | Eight boxes, with weight where the risk is | Six boxes, plus what was intentionally not built |
| Deep dive | Goes deep where comfortable | Goes deep where the risk is, with numbers | Goes deep where the risk is, and names the alternative rejected and why |
| Failure | Lists errors | Lists failure classes with detection | Lists failure classes, detection, and the one that will actually happen given the team |
| Trade-offs | Presents a choice as obvious | Presents both sides with a number | Presents both sides, picks one, and states the condition under which they would switch |
| Communication | Answers questions | Drives, checks in, adjusts | Drives, negotiates scope, and makes the interviewer's job easy |

**Anti-signals: the answers that use senior vocabulary at a mid depth.**

| Said | Why it reads as weaker, not stronger |
|---|---|
| "I would use a conflict-free replicated data type" with no mention of metadata growth or the server's role | Names a solution family without its cost, which suggests the term was learned rather than used |
| "We need backpressure" with no queue, no depth and no drop policy | Backpressure is a mechanism with a number. Without one it is a word |
| "Eventual consistency is fine here" without saying what the user sees during the window | The interesting part is the user-visible interval, and skipping it skips the design |
| "We would shard the client database" | Applies a server concept to a single-user device, which suggests pattern matching rather than reasoning |
| Naming five caching layers | Every layer is an invalidation bug. Count is not depth |

### 20d. Worked example

**The prompt.** "Design a chat client that works offline." The same prompt,
answered three ways.

#### Block A. Purpose: the mid answer, and exactly what it is missing

"I would keep messages in a local database and show them from there. When you
send a message it goes in the database and a background worker sends it. If
there is no network it retries later. I would use push notifications for
incoming messages, and cache the last few hundred messages per conversation so
you can read offline. For conflicts, the server timestamp wins."

What is right: local-first reads, a background sender, push for delivery. This
is a competent answer and it would pass a bar for mid.

| Missing | Consequence |
|---|---|
| No numbers anywhere | The budgets dimension scores zero |
| "Retries later" with no policy | No backoff, no cap, no poison-entry handling, so a single bad message can block a queue |
| "Server timestamp wins" applied to everything | Read state and message ordering need different rules, and last write wins destroys the first |
| Durability of the send is implied, not designed | The transaction boundary between the local write and the outbox row is the whole reliability story |

**The error most readers make here.** Reading the mid answer and concluding it
is wrong. It is not wrong, it is unevidenced, and the fix is not more components
but more derivation.

!!! tip "Self-explanation prompt"

    Before Block B, pick one sentence in the mid answer and write the senior
    version of it by attaching a number and a consequence.

#### Block B. Purpose: the senior answer, and the exact deltas it adds

"Messages are local-first: a send writes the message row and an outbox row in
one transaction, with an idempotency key generated at enqueue so a retry after
an ambiguous timeout does not duplicate. The interface renders from the local
store only.

Delivery rides push. A push wakes the radio, so fetching the body in that window
costs about 0.3 seconds of radio time rather than a full 11.2 second wakeup
cycle: 200 pushes a day is roughly 30 J against a 415 J daily background budget,
which is 1 percent of a 3000 mAh battery.

Ordering does not use device clocks. The server assigns a per-conversation
sequence, and read state syncs as a last-read sequence merged by maximum, which
is monotone so it needs no conflict resolution at all. Unread counts are computed
locally from that sequence rather than synced, because a synced count is a second
source of truth that drifts from the messages.

Failure modes I would watch: a poison outbox entry, detected by p99 oldest
outbox age above 24 hours; duplicate sends, detected by the server's
idempotency-key hit rate; and silent push loss, which is expected, so the design
repairs on next sync rather than assuming delivery."

| Delta over mid | Dimension it scores |
|---|---|
| Transaction boundary named explicitly | Depth |
| Idempotency key at enqueue, with the failure it prevents | Depth and failure anticipation |
| Battery arithmetic tied to the push design | Budgets |
| Monotone merge for read state, with "no conflict resolution needed" | Trade-off reasoning |
| Three failure modes each with a detection | Failure anticipation |

**The error most readers make here.** Believing the senior answer is longer.
It is roughly the same length. It replaced vague clauses with derived ones,
which is a substitution rather than an addition.

#### Block C. Purpose: the staff answer, and why it has fewer parts

"Everything in the senior answer stands, with three changes.

First, I would not build the local search index in version one. It is the
component teams always add here, it costs an incremental rebuild on every write,
and I have no evidence that offline search is used. I would ship without it and
instrument in-app search frequency, and build it when that number crosses a
threshold we agree now.

Second, the highest risk in this design is not technical. Three teams touch the
message path, and the idempotency key contract has to hold across all of them.
I would write the key semantics into the API contract with a test both sides run,
because the failure mode is a client team and a server team each assuming the
other deduplicates.

Third, sequencing. The outbox and idempotency work is the first release because
it is the correctness floor. Multi-device read state is second. The encrypted
attachment path is third and is a quarter of work on its own, so I would not
promise it in the same release, and I would say that to the product owner in
those words."

| Delta over senior | Dimension it scores |
|---|---|
| Removes a component, with the measurement that would add it back | Restraint, and evidence |
| Names an organisational failure mode and a mechanism against it | Failure anticipation at the right scope |
| Sequences the work across releases with reasons | Scope treated as changeable |
| Commits to a conversation with the product owner | Communication and drive |

**The error most readers make here.** Trying to reach staff by adding
sophistication. The staff answer here contains no technique the senior answer
lacked. It removed something, named a cross-team failure, and put the work in
an order.

!!! tip "Self-explanation prompt"

    Say which of the three staff moves you could make in your next interview
    without knowing anything new, and what stops most candidates from making it.

#### Block D. Purpose: name the answers that look senior and are not

| Answer | What the interviewer hears |
|---|---|
| "I would use an operation-based CRDT for the message log" | Message ordering is server-assigned in every chat design that has a server. This applies heavy machinery to a solved problem, which signals pattern matching |
| "We need a circuit breaker on the send path" | A circuit breaker protects a shared server from a stampede. On a single client, backoff plus a cap is the mechanism, and naming the wrong one shows the concept was borrowed |
| "Everything is eventually consistent so it converges" | Convergence is a property you engineer per field. Asserting it as a blanket property skips the design |
| "I would cache aggressively at every layer" | Every layer is an invalidation bug and a memory cost. Aggressive is not a design parameter |
| Reciting a framework acronym as section headings | The structure is doing the thinking, which is exactly what the level ladder is trying to detect |

The repair for each is the same shape: state the mechanism, attach a number,
name what it costs, and say when you would not use it. That is the move this
whole course trains.

**The error most readers make here.** Assuming the interviewer will not probe.
Every one of these phrases invites the follow-up that exposes it, and the
follow-up arrives within one question.

#### Block E. Purpose: three behaviours that raise your level in the next round

None of these require new knowledge, which is the point.

| Behaviour | How to do it tomorrow |
|---|---|
| Attach a number to every claim | Before saying "we would cache", compute the entry size and the cap. Two device facts and one multiplication is enough |
| Name one thing you will not build, and the measurement that would change your mind | Pick the component you are least sure about and say "I would not build this yet, and here is what would make me" |
| State the failure that happens because of how teams work, not how code works | Contracts assumed differently by two teams, a flag nobody removed, a kill switch never tested |

The first behaviour moves mid to senior. The second and third move senior toward
staff. All three are available to a candidate who has read this course and has
not shipped anything new since.

**The error most readers make here.** Performing a level above your experience.
An interviewer probes the claim, and a sequencing plan that cannot survive one
follow-up about who owns the server contract reads worse than an honest senior
answer. Reach one step, not two.

### 20e. Fade the scaffold

**Fade 1: "design an image loading library".** Mid and senior answers given,
staff answer blank.

- Mid: "A memory cache and a disk cache, load on a background thread, show a
  placeholder, cancel when the view is reused." Correct shape, no quantities, no
  mechanism for the two failures that matter.
- Senior, on sizing: "Decode at the view's measured size, because a 4000 by 3000
  source is 48 MB decoded against a 2.59 MB target. Cap the memory cache in
  bytes at 32 MB, which is 12 entries on a 256 MB heap."
- Senior, on wasted work: "Cancel on recycle, which during a 200 row fling takes
  200 decodes down to about 20, so 3.0 s of CPU becomes 0.3 s. Coalesce
  identical in-flight requests so a grid of the same URL decodes once.
  Newest-first decode queue with a bounded depth, because during a fling the
  newest request is the one about to be visible."
- Staff: **you write this answer.**

??? note "Show answer"

    Everything in the senior answer, plus three moves that change the scope.

    One, change the boundary. The largest available win is not in the library, it
    is that the server sends one resolution for every screen. Negotiating a
    resolution parameter in the image URL or the response contract cuts bytes and
    decode cost together, and it benefits every client. That is a cross-team
    change with a migration for old clients, so I would ship the client-side caps
    first and open the contract conversation in parallel rather than blocking on
    it.

    Two, remove something. I would not build a custom eviction policy. Least
    recently used with a byte cap is within a few points of anything cleverer at
    this scale, and the difference is worth less than the bugs. The measurement
    that would change my mind is a hit-rate curve against cap for two policies on
    real traffic.

    Three, name the organisational failure. A shared image library ends up owned
    by nobody and configured differently by each feature team, and the failure
    that follows is a memory cap raised locally by one team to fix their screen,
    which then kills the process on everyone else's. The mechanism against it is
    that the cap is owned centrally and configurable only per device tier, with
    the override requiring a review.

    Sequencing: caps and cancellation first, because they stop process kills.
    Coalescing second. Resolution negotiation third, in parallel with a
    server-side owner. Custom eviction never, unless the curve says otherwise.

**Fade 2: "design an analytics SDK for third-party apps".** Mid answer given,
senior and staff blank.

- Mid: "A public track method, an in-memory queue flushed every 30 seconds, an
  init call in the host's startup that sets up the database and fetches the
  config, and a listener for delivery callbacks."
- Senior: **you write this answer.**
- Staff: **you write this answer.**

??? note "Show answer"

    Senior. The surface is five calls and nothing blocks. `configure` stores the
    options and returns in microseconds; setup runs after the host's first frame
    or on first use, because 8 ms of database open plus 4 ms of config read is 12
    ms at the median and 120 ms at a p99 storage tail, against a host budget of
    500 ms that is already at 430.

    The queue is durable, not in memory, because process death is the normal case
    and the events lost would be exactly the ones describing a crash. Cap it at
    1 MB, which at 40 events per session and 300 bytes each is many weeks of
    offline use, and drop lowest priority oldest first while never dropping
    stability events.

    Batch 50 events into roughly 3 KB compressed, and flush on foreground, on
    backgrounding, and on existing host network activity. Never on a wakeup of
    our own, because an isolated radio wakeup costs about 5.6 J of someone
    else's battery.

    Threading contract stated in the documentation: no public method blocks, all
    callbacks arrive on a caller-supplied executor defaulting to main, and one
    serial queue owns all mutable state. No third-party type appears in a public
    signature, because that pins the host's dependency versions.

    Staff. All of that, plus four moves.

    One, scope. The costly long-term commitment is not the client, it is that the
    ingestion endpoint must accept every payload shape ever released, because
    hosts upgrade slowly and some never do. I would set that window explicitly at
    an assumed 24 months, instrument traffic share by SDK version, and put the
    retirement of a version on a roadmap rather than discovering the cost later.

    Two, removal. No automatic screen and interaction capture in version one.
    It is the feature that sells the SDK and it puts data in our pipeline that
    the host never disclosed to their users, which is their legal exposure. Ship
    explicit instrumentation with good ergonomics, and add an opt-in automatic
    mode later with a documented capture list.

    Three, the organisational failure mode. The thing that kills an SDK is a
    single crash attributed to us in a host's dashboard, because the host removes
    it and never returns. So the invariant is that no exception crosses the public
    boundary, and I would enforce it with a boundary test rather than a code
    review convention.

    Four, sequencing and support cost. Diagnostics ship in version one, not later:
    a snapshot with queue depth, oldest event age, last error and last successful
    send turns a week of email with a host we cannot debug into one screenshot.
    That is the highest return item in the whole plan, and it is the one that
    always gets deferred.

### 20f. Check yourself

**Q1.** A candidate gives an answer with twelve components, all correctly named,
and no numbers. Another gives six components with three derived numbers and one
component explicitly not built. Say how each is scored on the four dimensions
and which one you would hire for a senior role.

??? note "Show answer"

    Twelve components, no numbers: scope is the client only, evidence is absent,
    failure anticipation is untested because breadth left no room to probe, and
    nothing was left out. That scores as mid regardless of how correct the names
    are, because the interviewer cannot distinguish familiarity from judgement.

    Six components, three numbers, one deliberate omission: evidence is present
    and checkable, the omission demonstrates restraint and gives the interviewer
    a place to probe productively, and the smaller surface leaves time for depth.
    That is a senior signal even if it covers less ground.

    I would hire the second for senior. The first candidate's answer is
    unfalsifiable: every component might be right or might be recited, and there
    is no way to tell. The second candidate has given me three numbers I can
    challenge and one decision I can push on, which is what an interview is for.

**Q2.** You are interviewing for senior and you find yourself with fifteen
minutes left and no numbers stated. Give the recovery move and the two numbers
you would reach for first in a client round.

??? note "Show answer"

    The recovery move: stop adding to the design and retrofit the budgets out
    loud. "I have been designing without stating my budgets, so let me anchor
    two of them now and check the design against them." Then do it, and let the
    numbers change something. A number that changes nothing is decoration; a
    number that resizes a cache or removes a component is evidence.

    The two numbers to reach for first, because both are available from device
    facts alone: the decoded image size for your cell dimensions, which
    immediately produces a memory cap and often removes a component, and the
    frame budget at the target refresh rate with the 70 percent headroom rule,
    which immediately tells you what may not happen in a bind function.

    A third if there is time: the background battery share, since one wakeup at
    about 5.6 J against 41,580 J of battery converts any polling proposal into a
    percentage on the spot.

### 20g. When not to use this

**Performing a level above your experience.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You have actually done the thing: shipped the migration, owned the contract, sequenced the roadmap. Then describe it |
| Scale threshold below which it is over-engineering | If you cannot survive two follow-up questions on who owned the change and what went wrong, the claim costs more than it earns. Reach one step, not two |
| Cheaper alternative | Give an excellent answer at your level with numbers and one deliberate omission. A strong senior answer beats a thin staff impression in every rubric |

**The level ladder as a universal map.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The company publishes an engineering ladder whose language matches these dimensions, which many do |
| Scale threshold | At a small company the boundaries collapse: one engineer owns the contract, the roadmap and the code, so the scope dimension does not separate anyone |
| Cheaper alternative | Ask what the role's scope is in the first two minutes, and calibrate the answer to that rather than to a generic ladder |

**Deliberate omission as a technique.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You can name the measurement that would bring the omitted component back, and the threshold |
| Scale threshold | Omitting something the prompt explicitly asked for is not restraint, it is not answering. Restraint applies to components you introduced, not to requirements you were given |
| Cheaper alternative | Build it briefly and say it is the part you are least confident about, with the measurement you would run. That is honest and scores on the same dimension |

**Optimising for the level ladder at all during the round.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You have a specific weak dimension identified from past feedback, and you are practising it deliberately |
| Scale threshold | Mid-round, thinking about the ladder costs attention you need for the design. Practise the behaviours until they are habits, then forget the ladder exists |
| Cheaper alternative | Three habits, rehearsed until automatic: attach a number, name one thing you will not build, and state a failure mode with its detection. The ladder takes care of itself |

## Case study 1: News feed client

A feed looks like the easy prompt and it is not. The server side of a feed is covered in [Modern System Design](modern-system-design.md) and the ranking side in [ML System Design](ml-system-design.md). This case study assumes both exist and asks a different question: what does the phone have to do so that a person on a train, on a three year old handset, with a screen reader on, gets a feed that does not stutter, does not repeat itself and does not eat their data plan.

### The prompt (news feed client)

"Design the mobile client for our social feed: infinite scroll, images, likes and comments. It has to feel instant on a mid range Android phone with a bad connection."

### Questions that fork the design (news feed client)

Each row changes a box on the whiteboard. Ask four or five of these, not all seven, and say why you picked those.

| Question worth asking | If A the design becomes | If B it becomes |
|---|---|---|
| Is the feed chronological or ranked per session? | Chronological: ordering is a function of (created_at, id), so the client can use a keyset cursor, merge new items locally and reconcile after an offline period | Ranked: the order is a server-side session artefact, the client must carry an opaque cursor, must never re-sort, and cannot merge a local insert into the right position |
| Must the app show content when it opens with no network? | No: an in-memory page list plus the platform HTTP response cache is enough, and you ship no database | Yes: you need a durable local store, an explicit staleness label in the UI, and a decision about how old is too old to show |
| Do likes and comments have to survive the app being killed mid-action? | Best effort: fire the request, retry in memory, forget on process death | Durable: an on-disk outbox, idempotency keys, and a reconciliation pass on next launch, which is a whole subsystem |
| What is the lowest device tier we support? | Recent mid tier: you can size caches from an 8 GB device and target a 120 hertz frame budget | Three year old 3 GB device: the memory ceiling sets the image cache size, the frame budget is 60 hertz, and image variants must be requested at the device's own width |
| Is inline video autoplay in scope? | No: one media pipeline, images only | Yes: a player pool with a hard cap, decoder sessions as a scarce resource, and video prefetch competing with image prefetch for the same bytes |
| Is a formal accessibility conformance level a requirement? | No stated target: you still do the basics but nothing forces layout decisions | Yes: 200 percent text scaling changes cell heights, which changes how many items fit a screen, which changes prefetch distance, which is now a derived number rather than a constant |
| Can the server describe new post types without an app release? | No: the client renders a closed set of types and a new type needs a store rollout, which takes weeks to reach most users | Yes, server driven layout: the renderer becomes generic, and version skew between server payloads and installed app versions becomes a first class failure mode |

!!! tip "Say this out loud in the room"

    "I am going to ask about the lowest device tier before I ask about anything else, because on a mobile client the memory ceiling is what sizes the caches, and the caches are most of the design."

### Requirements (news feed client)

Functional:

- Open to feed content from local storage, then refresh, without a blank screen in between.
- Scroll indefinitely, fetching further pages before the reader reaches the end.
- Render text posts, single image posts and image plus caption posts.
- Like, save and comment, with the action visible immediately and durable across process death.
- Operate fully with a screen reader, and reflow correctly at 200 percent text scale.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Low device tier | 720 by 1600 pixels, 3 GB RAM, 60 hertz | ASSUMED; this is the shape of a phone that is three years old at the time of writing, and the interviewer usually has an opinion, so ask |
| High device tier | 1080 by 2400 pixels, 8 GB RAM, 120 hertz | ASSUMED, for the same reason |
| Frame budget | 16.7 ms at 60 hertz, 8.3 ms at 120 hertz | DERIVED: 1000 divided by the refresh rate |
| Process memory budget | 200 MB low tier, 400 MB high tier | ASSUMED; device makers configure per app limits, so treat these as placeholders and read the real limit at runtime |
| Cold start to first content | under 1.5 s at p90 on the low tier | ASSUMED; it is roughly the point at which a person stops watching the screen and looks away |
| Reader pace | one screen every 2.5 s while browsing, 20 screens per session | ASSUMED; this is the number that sets prefetch distance, so measure it in your own product rather than inheriting it |
| Poor network | 1.5 Mbps effective, 300 ms round trip | ASSUMED; a moving train on a congested cell, which is the worst case people actually complain about |
| Image memory cache | 50 MB low tier, 100 MB high tier | DERIVED below |
| Bytes per screen | 110 KB low tier, 250 KB high tier | DERIVED below |
| Visible duplicate or skipped items | zero | GIVEN by the product; a feed that repeats itself is the most reported feed bug there is |

### Estimation that eliminates (news feed client)

Every line ends with the option it kills. Assume a feed post cell occupies about 41 percent of screen height (a 4 by 3 image at full width plus roughly 120 pixels of header and action row), so about 2.5 posts are visible per screen.

| Calculation | Result | What it rules out |
|---|---|---|
| 1080 by 810 pixels at 4 bytes per pixel for a decoded 32 bit bitmap | 3.5 MB per image in memory, high tier | Rules out caching decoded bitmaps for a whole 20 item page: that is 70 MB, more than the entire high tier image budget |
| 720 by 540 at 4 bytes per pixel | 1.6 MB per image, low tier | Rules out requesting one image size for all devices: the low tier device would hold 3.5 MB bitmaps to draw 720 pixels of them |
| 200 MB process budget times 25 percent, divided by 1.6 MB | 31 bitmaps on the low tier | Rules out sizing the image cache by item count. A text post costs kilobytes and an image post costs megabytes, so the cache must be bounded in bytes |
| 31 bitmaps divided by 2.5 posts per screen | about 12 screens of images held in memory | Rules out reaching disk for a short scroll back, and rules out the belief that you need a memory cache larger than the reader's working set |
| 1080 by 810 pixels at an assumed 0.9 bits per pixel encoded | 98 KB per image on the wire, high tier | Rules out nothing yet, but it is the input to the next three lines. Measure your own corpus; modern codecs at the same visual quality land lower |
| 720 by 540 at 0.9 bits per pixel | 44 KB per image, low tier | Rules out serving high tier variants to low tier devices: it is 2.2 times the bytes for pixels the panel cannot show |
| 2.5 images per screen times 98 KB, plus 2.5 times 1.5 KB of post JSON | about 250 KB per screen, high tier | Rules out a per screen budget target that ignores which tier you are on |
| 250 KB per screen divided by 1.5 Mbps | 1.33 s to fetch a screen on the bad network | Rules out fetching on demand at the scroll boundary: the reader advances every 2.5 s and would see a spinner half the time |
| Prefetch 2 screens ahead at 1.33 s per screen | 5 s of cover against a 1.33 s fetch | Rules out prefetch distance 1 (no cover for a stall) and rules out distance 6, see next line |
| Prefetch distance d over a 20 screen session wastes d divided by (20 plus d) | 9 percent waste at d equals 2, 23 percent at d equals 6 | Rules out aggressive prefetch. Three times the waste to buy cover the reader's pace does not need |
| 20 post page at 1.5 KB per post, on a 300 ms round trip link | 30 KB, 160 ms transfer plus 300 ms round trip equals 460 ms | Rules out 5 post pages: four requests cost 1.2 s of pure round trips to deliver the same 20 posts |
| 100 post page at 1.5 KB per post | 150 KB, 800 ms transfer plus 300 ms round trip | Rules out very large pages: 8 screens is 20 posts, so 80 percent of a 100 post page is metadata the session never reaches |
| Decode of 0.39 megapixels at an assumed 35 megapixels per second on one slow core | 11 ms per low tier image | Rules out decoding on the main thread: 11 ms out of a 16.7 ms frame, and 11 out of 8.3 ms is already over budget at 120 hertz |
| A fling covering 5 screens in 1 s, at 2.5 posts per screen | 12.5 decodes per second, 138 ms of CPU per second | Rules out an unbounded decode pool: the work is bursty, so it needs a bounded queue with cancellation, not more threads |
| Process start 350 ms, first layout 250 ms, cached page read 150 ms, first image decode 60 ms | 810 ms to first content from disk | Rules out nothing on its own. It leaves 690 ms of the 1.5 s budget, which is the slack the next line spends |
| The same start, but blocking the first frame on the network: 350 plus 250 plus 600 (cold handshake, 2 round trips) plus 460 (page) plus 590 (three low tier images) | 2.25 s | Rules out a network blocking first frame. Cached content must render before the refresh returns |

!!! note "These assumptions are knobs, not facts"

    The 0.9 bits per pixel, the 35 megapixels per second decode rate and the 200 MB process budget are planning assumptions. Substitute measurements from your own devices and your own image corpus. What transfers is the structure: bitmap memory scales with displayed pixels and not with file size, and the memory ceiling is what sizes every cache on the page.

### V0, the simplest thing that works (news feed client)

**Predict first.** At the stated scale, which single component of the simplest possible feed client fails first, and what number tells you?

??? note "Show answer"

    Not the network layer and not the list widget. The image path fails first, and the number is 11 ms of decode inside a 16.7 ms frame. Everything else in a feed client is cheap; images are the only thing on the screen that costs megabytes of memory and tens of milliseconds of CPU per item.

```
+-------------+   +------------------+   +-----------------+
| Feed screen |-->| Feed repository  |-->| HTTP client     |
| list widget |<--| page list in RAM |<--| disk resp cache |
+-------------+   +------------------+   +-----------------+
       |                                         |
       v                                         v
+-------------+                           +-----------------+
| Image view  |-------------------------->| Image CDN       |
| bitmap LRU  |<--------------------------| sized variants  |
+-------------+                           +-----------------+
```

**Caption.** V0 has four client components: a list widget, a repository holding the fetched pages in memory, an HTTP client whose disk response cache provides the only persistence, and an image view backed by a byte bounded bitmap cache that fetches sized variants from a content delivery network.

Why this is correct at the stated scale:

- One reader, one session, 20 screens. There is no throughput problem on the device and no concurrency problem beyond the decode pool.
- The platform HTTP response cache gives you offline first paint for free. A stored response for the first feed page is read from disk at launch, which covers the "no blank screen" requirement without a database.
- Sized variants from the delivery network remove the largest single waste (2.2 times the bytes) with a query parameter rather than a client pipeline.
- Every box you do not draw is a box you do not have to migrate, monitor or explain when it corrupts.

!!! warning "When not to build a local database for a feed"

    Below roughly 5,000 locally retained items, and when no user action needs to survive process death, the platform HTTP response cache plus an in memory page list is the right answer. A database adds a schema, a migration path, a background thread, and a class of bugs where the UI and the store disagree. The measurement that justifies it is either a durable write requirement (an outbox) or an offline session share above roughly 5 percent of launches, measured with a launch time connectivity flag.

### Breaking point, then V1 (news feed client)

**Breaking point one: scroll jank, visible at any fling above about 3 screens per second.**

What fails. Decode runs on whichever thread called the image loader. At 12.5 decodes per second and 11 ms each, any frame that contains a decode misses its 16.7 ms budget. Worse, a recycled cell keeps its in flight request, so the reader sees the wrong photo appear for one frame before the right one lands.

How you observe it. Record per frame duration during scroll only, and publish two numbers: the share of frames over budget, and p99 frame duration. Segment both by device tier. If the over budget share rises with image density per screen rather than with total scroll distance, the decode path is the cause and adding threads will not fix it.

V1 makes the media path explicit:

```
+-------------+   +------------------+   +-----------------+
| Feed screen |-->| Feed repository  |-->| Paged fetcher   |
| list widget |   | page list in RAM |   | cursor based    |
+-------------+   +------------------+   +-----------------+
       |                                         |
       v                                         v
+-------------+   +------------------+   +-----------------+
| Image req   |-->| Decode pool      |-->| Bitmap LRU      |
| keyed to    |   | bounded, cancels |   | 50 MB, bytes    |
| view holder |   | on recycle       |   | not item count  |
+-------------+   +------------------+   +-----------------+
                            |
                            v
                  +------------------+   +-----------------+
                  | Encoded disk     |<--| Image CDN       |
                  | cache 200 MB     |   | sized variants  |
                  +------------------+   +-----------------+
```

**Caption.** V1 keys every image request to the view holder that asked for it, decodes on a bounded pool that cancels on recycle, and adds a disk tier holding encoded bytes so a re-decode does not become a re-download.

What each addition buys, with the measurement that justifies it:

| Addition | What it fixes | Measurement that justifies it |
|---|---|---|
| Request keyed to the view holder | The wrong photo flashing in a recycled cell | Any non zero count of "delivered to a recycled holder" events, which you can only see if you log them |
| Bounded decode pool with cancellation | 138 ms per second of bursty CPU landing on the main thread | Share of frames over budget during scroll, segmented by images per screen |
| Byte bounded bitmap cache | Out of memory kills when a page of image posts arrives | Out of memory kill rate per 1,000 sessions, and peak resident memory at p95 |
| Encoded disk cache, 200 MB | A scroll back beyond 12 screens turning into a re-download | Image cache hit ratio split by tier, memory and disk plotted separately |
| Cursor based paged fetcher | See breaking point two | Duplicate and skipped item counts per session |

!!! warning "When not to add a three tier image cache"

    If your feed is text first, with images on fewer than about one item in ten, a single byte bounded memory cache plus the HTTP response cache is enough. The disk tier costs you an eviction policy, a size budget the user can see in their storage settings, and a cache poisoning failure mode.

    The measurement that justifies the third tier is a memory cache hit ratio below roughly 70 percent at your measured scroll back distance, plotted against cache size, so you can see whether more memory would have been the cheaper fix.

**Breaking point two: the feed repeats itself, at roughly 6.5 percent of items on a busy feed.**

What fails. Offset pagination asks for items 20 to 39 of a list that is being edited underneath it. Assume 4 new items reach the head of the feed per minute at peak, and that a page of 20 items is 8 screens, which is 20 s of reading.

Between two page requests, 1.3 new items appear at the head. Every one of them shifts the offset by one, so the next page repeats 1.3 items out of 20, which is 6.5 percent. Deletions produce the mirror bug: items the reader never sees.

How you observe it. The client already knows every item id it has rendered. Count ids that appear twice in a session, and count gaps by comparing the ids the server returned against the ids the client already held. Publish both per session. Nobody finds this bug from server logs.

V2 makes the feed a real local store:

```
+-------------+   +------------------+   +-----------------+
| Feed screen |-->| Local store is   |<--| Sync worker     |
| list widget |<--| the only source  |   | cursor + delta  |
+-------------+   | of truth on disk |   +-----------------+
                  +------------------+            |
                          |                       v
+-------------+   +------------------+   +-----------------+
| Outbox      |-->| Action sender    |-->| Feed API        |
| durable, id |   | idempotency key  |   | opaque cursor   |
+-------------+   +------------------+   +-----------------+
```

**Caption.** V2 makes an on disk store the single source of truth that the screen reads from, fills it through a sync worker that pages with an opaque cursor, and sends user actions through a durable outbox carrying an idempotency key per action.

| V2 change | Justified by | Cost you accept |
|---|---|---|
| Opaque cursor from the server | 6.5 percent duplicate rate under offset paging | The client can no longer reason about position, so "jump to top" becomes a server call |
| Local store as the only read path | Two sources of truth (memory list and disk) disagreeing after a background refresh | A schema, migrations, and a background write path that must never block a frame |
| Durable outbox with idempotency keys | Likes lost on process death, and double likes on retry | The UI must model three states per action (pending, confirmed, failed) instead of two |
| Delta sync rather than page replacement | A full page replacement resets scroll position and loses the reader's place | The server must be able to express edits and deletes, not just pages |

!!! warning "When not to make the local store the source of truth"

    A read only feed with no offline requirement and no durable actions does not need this. Single source of truth costs you a schema, a migration for every shape change, and the discipline that no screen may read from the network directly. The measurement that justifies it is either a non zero durable action requirement or a measured rate of UI and store disagreement, which shows up as items that reappear after being hidden.

### Deep dive one: the bitmap budget, or why a feed dies of images

This dive is chosen because the arithmetic above put every constraint in the image path. The chat, maps and video clients later on this page each have a different dominant cost, so their dives go elsewhere.

**The four sizes of one photograph.** Candidates conflate them, and every real bug lives in the gap between two of them.

| Size | What it is | Low tier value | Why it matters |
|---|---|---|---|
| Source | The original upload | 4000 by 3000, several MB | Never send this to a phone; it is the single most common cause of an out of memory crash |
| Encoded variant | The bytes on the wire | 720 by 540, 44 KB | Sets your bandwidth and disk cache cost |
| Decoded bitmap | Pixels in memory | 720 by 540 by 4 bytes, 1.6 MB | Sets your memory cache cost, and it is 36 times the encoded size |
| Displayed | Pixels the panel lights up | 720 by 540 | Any decoded pixel beyond this is memory you paid for and nobody saw |

**The rule that follows.** Decode to the displayed size, not to the source size. If the view is 720 pixels wide and the variant is 1080, downsample during decode rather than after it, because decoding at full size and then scaling allocates the 3.5 MB you were trying to avoid.

**Sizing the cache from the ceiling, not from a round number.**

1. Read the process memory limit at runtime rather than hard coding it. A 3 GB device and an 8 GB device do not get the same budget, and a hard coded 50 MB is either wasteful or fatal depending on which one runs it.
2. Give the bitmap cache a quarter of the budget. That leaves room for the list widget, the store, the networking stack and whatever the platform allocates on your behalf.
3. Bound it in bytes. An item count is meaningless when one item is 4 KB of text and the next is 1.6 MB of photograph.
4. Evict least recently used, but pin the items currently attached to a visible view. An eviction that frees a bitmap the screen is drawing produces a blank cell.
5. Drop to a smaller budget on a memory warning from the OS, and log that it happened. A memory warning you did not respond to becomes a process kill you cannot explain.

**The prefetch window, derived rather than guessed.**

| Input | Value | Source |
|---|---|---|
| Reader pace | one screen per 2.5 s | ASSUMED, and it is the first thing to measure in your own product |
| Screen fetch time on a bad link | 1.33 s high tier, 0.59 s low tier | DERIVED from bytes per screen and 1.5 Mbps |
| Cover needed | one fetch plus one stall | Design choice: survive a single stall without a spinner |
| Window | 2 screens, which is 5 items at 2.5 posts per screen | DERIVED |
| Waste at that window | 9 percent of session bytes | DERIVED from d divided by (20 plus d) |

At 200 percent text scale the post cell grows from about 41 percent to about 49 percent of screen height, so posts per screen falls from 2.5 to about 2.0. A window expressed as "5 items" now covers 2.5 screens instead of 2. Express the window in screens and convert using measured cell heights; a constant item count silently changes meaning when the reader changes their font size.

**Cancellation is the part people skip.** When a cell is recycled, three things must happen in order, and getting the order wrong produces the flashing wrong image:

1. Detach the request token from the view holder, so a late completion has nowhere to deliver.
2. Cancel the network fetch if it has not started transferring, and let it finish into the disk cache if it has, because a half fetched image is wasted bytes and a fully fetched one is a future hit.
3. Cancel the decode, which is the expensive part, and the only one that is definitely worth stopping.

**A worked cost comparison, so you can defend a number.** Two designs for the same 20 screen session on the low tier:

| Design | Bytes fetched | Peak bitmap memory | Frames over budget |
|---|---|---|---|
| One variant for all devices, decode on demand, no cancellation | 20 screens times 245 KB equals 4.9 MB | 2.5 visible times 3.5 MB equals 8.8 MB plus uncancelled decodes in flight | High: every frame with a decode |
| Sized variants, bounded pool, cancel on recycle, 2 screen prefetch | 22 screens times 110 KB equals 2.4 MB | Bounded at 50 MB by construction | Low: decodes never touch the frame thread |

The second design fetches half the bytes and bounds the memory, and the reason is not cleverness. It is that the first design decoded pixels nobody displayed.

!!! warning "When not to build your own image pipeline"

    If your app shows fewer than about 20 images per session and none of them are full width, use the platform image loading facility and stop. A hand rolled pipeline costs you a decode pool, a two tier cache, a cancellation protocol and a lifetime of edge cases around recycling.

    The measurement that justifies building or adopting a full pipeline is a memory profile where decoded bitmaps exceed roughly 20 percent of process memory, or an out of memory kill rate above roughly 1 per 1,000 sessions on your lowest tier.

### Deep dive two: pagination on a feed that is editing itself

The second dive goes somewhere unrelated to the first on purpose. The image dive was about a resource ceiling. This one is about correctness under concurrent mutation, and it ends somewhere candidates rarely go: what a reordering feed does to a screen reader.

**Four pagination schemes, compared on what breaks.**

| Scheme | Request shape | Duplicates | Skips | Works on a ranked feed |
|---|---|---|---|---|
| Offset | skip 20, take 20 | Yes, one per head insert between pages | Yes, one per head delete | No |
| Page number | page 3, size 20 | Same as offset, it is offset wearing a hat | Same as offset | No |
| Keyset on a sort key | after (created_at, id) | No, if the sort key is stable and unique | No, unless the item itself is deleted | Only if the ranking is a stable sort key, which ranked feeds are not |
| Opaque server cursor | after "eyJ2IjoxLCJz..." | No, the server owns the session's materialised order | No | Yes, this is what a ranked feed needs |

**Why keyset is not enough for a ranked feed.** Keyset pagination works because the sort key totally orders the collection and does not change. A ranking score changes every time the model runs. If the client pages on score, an item whose score rose between page one and page two appears twice, and an item whose score fell disappears. The fix is not a better client cursor. It is that the server materialises the order once per feed session and hands the client a position in that materialisation.

**The three failure modes the cursor does not fix, and what does.**

| Failure | What the reader sees | Client side fix |
|---|---|---|
| Deleted item still in the cursor's materialised page | A post that no longer exists, or a blank cell | Tombstone handling in the local store, and a diff on apply rather than an append |
| Item edited after it was rendered | Stale caption or like count | Version field per item; apply the higher version and leave scroll position alone |
| Cursor expired after a long background period | An error, or worse, a silent restart from the top | Detect the expiry, keep showing the local store, and refresh from the top only when the reader is at the top |

**The rule for applying a page.** Never replace the list. Diff the incoming page against the local store by item id, then apply inserts, updates and deletes. Replacement is what resets scroll position, and losing a reader's place is a more damaging bug than showing them a slightly stale like count.

**What a mutating list does to a screen reader.** This is where a strong answer separates itself.

- A screen reader user's reading cursor sits on one item. If the list re-sorts underneath, the cursor is now somewhere unrelated, and there is no visual context to recover from.
- Inserting at the head while the reader is at item 40 moves everything down by one. Sighted readers do not notice; a reader navigating by position does.
- An infinite list has no end, so "how far through am I" has no answer unless you supply one.

The mechanics the W3C authoring guidance sets out for this pattern, in the terms it uses:

| Concern | Mechanism | Consequence if you skip it |
|---|---|---|
| Position in a set that has no end | Per item position and set size attributes | The reader is told "item, item, item" with no sense of progress |
| Content arriving while the reader is inside the list | Mark the region busy during the update, and clear it when done | Partial announcements, or announcements interleaved with the item being read |
| Which item is the anchor for loading more | Load and unload based on which item holds focus, not on pixel scroll position | Content unloads under the reading cursor, which throws focus to the top of the page |
| Naming each item | Each item labelled by its own heading rather than by its body | Every item announces as a wall of text with no way to skim |

**The design consequence.** If loading is driven by focus rather than by pixel offset, then your prefetch trigger has two implementations: a scroll position trigger for pointer and touch input, and a focus position trigger for assistive technology and keyboard input. Candidates who have shipped this know it; candidates who have not assume one trigger covers both.

**Do not unload aggressively.** Recycling views is fine. Removing items from the underlying collection to save memory is not, because the reading cursor may be inside the range you removed. Bound memory with the bitmap cache, which is where the megabytes actually are, and leave the item metadata in place. At 1.5 KB per item, 2,000 retained items cost 3 MB, which is 1.5 percent of the low tier budget.

!!! warning "When not to use an opaque server cursor"

    A chronological feed with a stable, unique sort key should use keyset pagination instead. Keyset lets the client reason about position, merge locally created items into the right place, and resume after an offline period without a server round trip. An opaque cursor takes all three away and adds a server side session that can expire. The measurement that justifies the opaque cursor is a ranking function that reorders items between requests, which you can detect by paging twice with the same parameters and diffing the results.

### Failure modes and evaluation (news feed client)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Thundering herd | A push notification to everyone drives millions of clients to foreground and refresh in the same 10 s | Jittered refresh on foreground, and a server side ability to tell clients to back off; the client must honour a retry after hint |
| Retry storm | The feed API slows, every client retries three times, effective load triples, it slows further | Bounded retries with full jitter, a per session retry budget expressed as a fraction of successful requests, and no retry at all on 4xx |
| Cache stampede at the delivery network | A new image variant size ships in an app release and every client requests a size that has never been generated | Roll out new variant sizes behind a flag to a small share first, and pre-generate the variant server side before the flag opens |
| Hot key | One viral post's image is requested by every client at once | This is a delivery network concern, not a client one, but the client contributes by requesting a stable variant URL so the edge can actually cache it |
| Process kill under memory pressure | The reader switches apps, comes back, and the feed has restarted at the top | Persist scroll anchor (the id of the top visible item) on every stop, and restore by id rather than by index |
| Clock skew | Items sorted by a client supplied timestamp appear in the wrong order when the device clock is wrong | Sort by server assigned ordering only; use the device clock for display only, never for ordering |
| Version skew | A server driven post type ships before the app version that renders it has reached most installs | Every unknown type renders a defined fallback cell, and the server is told the client's capability version so it can avoid sending types the client cannot draw |
| Metastable failure | After an outage, restored clients all sit in a tight retry loop that keeps the API above capacity | Exponential backoff with a cap and jitter, plus a client honoured server side load shedding signal |

Service level indicators to publish, all measured on the device:

- Cold start to first content at p90, segmented by device tier and by whether the launch had a network.
- Share of frames over budget during scroll, and p99 frame duration, segmented by device tier.
- Feed page latency at p95 including client retries, which is not the same number as the server's latency.
- Image time to display at p95, and image failure rate.
- Duplicate item count and skipped item count per session.
- Outbox drain latency at p95, and outbox permanent failure rate.
- Out of memory kill rate and crash free session rate per 1,000 sessions.
- Bytes per session, split into bytes rendered and bytes prefetched but never seen.

Alert on: crash free sessions and application not responding rate by app version and device tier during a staged rollout; the over budget frame share crossing its threshold on any tier; and any non zero duplicate item rate, because the correct value is zero and a small non zero value is the early warning.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Cold start under 1.5 s at p90 | Yes at 810 ms from the local store, and only because the network refresh does not block the first frame |
| No blank screen on open | Yes, the store is read first and the refresh applies as a diff |
| Zero duplicates or skips | Yes with an opaque cursor plus diff on apply; measured client side, not asserted |
| Durable likes and comments | Yes through the outbox, with idempotency keys so a retry after process death does not double count |
| Memory within the tier budget | Yes, the bitmap cache is bounded in bytes at 25 percent of a runtime read limit |
| Screen reader operation and 200 percent scale | Partly: the item and set attributes and the busy state are the mechanism, but the prefetch trigger must also follow focus, and that is easy to regress |

### Where this answer is a simplification (news feed client)

- **The HTTP response cache is doing more work in V0 than it should.** A per user, per session ranked feed is not a cacheable resource in the terms of [RFC 9111, HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111) (June 2022; checked 1 August 2026). V0 leans on it for offline first paint, which works in practice and is honest to admit is a hack rather than a design.
- **The feed accessibility pattern is specified, not invented here.** The [W3C ARIA Authoring Practices Guide feed pattern](https://www.w3.org/WAI/ARIA/apg/patterns/feed/) (checked 1 August 2026) is the source for position and set size attributes, the busy state during updates, and the rule that loading follows focus rather than scroll offset. Read it before you design an infinite list.
- **Conformance targets are a real requirement with layout consequences.** [WCAG 2.2](https://www.w3.org/TR/WCAG22/) reached W3C Recommendation on 12 December 2024 (checked 1 August 2026). Reflow and target size criteria change cell heights and hit areas, which is why the prefetch window on this page is derived from measured cell heights rather than fixed.
- **Transport matters more than this page admits.** A lossy mobile link punishes head of line blocking, and connection migration across a Wi-Fi to cellular handoff is a transport feature, not an application one. See [RFC 9000, QUIC](https://www.rfc-editor.org/rfc/rfc9000) (May 2021) and [RFC 9114, HTTP/3](https://www.rfc-editor.org/rfc/rfc9114) (June 2022), both checked 1 August 2026.
- **What this page skips.** Ad insertion (which breaks every assumption about a stable item list), video autoplay policy, image codec negotiation across device generations, and the ranking system itself. Ranking is [ML System Design](ml-system-design.md); the fan out service behind the cursor is [Modern System Design](modern-system-design.md).
- **The hardest real problem is not on this page.** It is that a feed client is where six teams ship features into one scroll surface, and no single owner controls the memory budget. That is an organisational problem wearing an architecture costume, and the technical mechanism that helps is a per feature memory and bytes budget enforced in continuous integration.

### Level delta (news feed client)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Draws a list, a repository and an API | Asks for the lowest device tier before drawing anything | Computes 1.6 MB per bitmap in the first three minutes and states that the whole design is a memory budget with a network attached |
| Image handling | Names a well known image loading library | Explains sized variants, a memory and disk cache, and cancellation on recycle | Derives the cache size from a runtime memory limit, shows the four sizes of one photograph, and names decode at display size as the single highest value rule |
| Pagination | Proposes page number or offset | Proposes a cursor and says why offset duplicates | Computes the 6.5 percent duplicate rate from an insert rate, then shows that keyset also fails on a ranked feed and explains why the server must materialise the order |
| Applying a page | Replaces the list | Appends the page | Diffs by id, handles tombstones and version fields, and names scroll position loss as a worse bug than staleness |
| Accessibility | Mentions content descriptions | Adds labels, focus order and dynamic type support | Notes that loading must follow focus rather than scroll offset, that unloading items can throw the reading cursor, and that 200 percent text scale changes the derived prefetch window |
| Durability | Fires the like request | Retries the like request | Designs the outbox with idempotency keys, models three action states in the UI, and says what reconciliation does on next launch |
| Measurement | "We would test on a low end device" | Names frame drops and crash rate | Publishes over budget frame share segmented by tier, duplicate rate per session, and bytes prefetched but never seen, then gates the rollout on them |

What each level added:

- **Mid to senior.** The design stops being a diagram of layers and starts being derived from a device tier, a network speed and a reader's pace.
- **Senior to staff.** The candidate computes the memory ceiling before choosing a component, treats correctness under concurrent mutation as a first class problem, and turns accessibility from a checklist into a constraint that changes a number elsewhere in the design.

---
## Case study 2: Chat client

The feed client's dominant cost was memory. The chat client's dominant cost is wakeups, and its dominant risk is a message the user believes they sent. The backend fan out for chat is in [Modern System Design](modern-system-design.md); everything below assumes it exists and asks what the device has to guarantee.

### The prompt (chat client)

"Design the mobile chat client: one to one and group conversations, messages arrive in order and never twice, it works offline, and the same account is signed in on a phone, a tablet and a laptop."

### Questions that fork the design (chat client)

| Question worth asking | If A the design becomes | If B it becomes |
|---|---|---|
| Is the content end to end encrypted, or only encrypted in transit? | Transport only: the server can order, deduplicate, search and fan out, and the client is a thin renderer of a server owned log | End to end: the server cannot read or reorder content, the client owns key state, search must run locally, and history for a new device becomes a client to client transfer problem |
| What is the largest group we support? | 50 members: encrypting once per recipient device is affordable and you never build anything cleverer | 5,000 members: per device fan out from the sender is a multi megabyte upload per message, so you need a shared group key distributed out of band |
| Do messages have to arrive while the app is backgrounded? | Foreground only: one socket, no push, and the whole delivery problem is a connection state machine | Background: push wakes the app, payload size limits apply, silent wakes are rationed by the OS, and the client must fetch after waking rather than trust the payload |
| Is ordering global per conversation, or only causal? | Server assigned sequence per conversation: gap detection is a subtraction, and backfill is a range request | Causal only: the client orders with a logical clock, tolerates a reorder window, and must decide what to render while it waits |
| Are typing indicators and read receipts in scope? | No: traffic is message shaped, bursty and rare | Yes: high frequency, low value events that dominate radio wakeups unless coalesced, and read state has to converge across three devices |
| Does full history sync to a newly added device? | No: a new device starts empty, and the entire key backup problem disappears | Yes: either a device to device transfer or an encrypted server backup, and the backup key becomes the weakest link in the whole system |
| Can a message be edited or deleted for everyone? | No: the conversation is an append only log and the local store is trivial | Yes: messages are mutable entities with tombstones, the store needs an update path, and you need a rule for an edit that arrives before the original |

!!! tip "Say this out loud in the room"

    "Whether this is end to end encrypted changes almost every box, so I want to settle it before I draw anything. If it is, the interesting work moves onto the device."

### Requirements (chat client)

Functional:

- Compose and send a message that appears in the conversation immediately, before any network call completes.
- Deliver every message exactly once to the reader's eye, in a stable order per conversation.
- Queue sends while offline and deliver them after the app has been killed and relaunched.
- Converge read state, delivery state and message content across three signed in devices.
- Search local history without a network call.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Messages sent per user per day | 40 | ASSUMED; ask, because it changes the local store size and nothing else |
| Messages received per user per day | 120 | ASSUMED; people receive more than they send because groups multiply reads |
| Devices per account | 3 | GIVEN in the prompt |
| Local history retained | 200,000 messages | ASSUMED; roughly four years at the rates above, and users are furious when history disappears |
| Bytes per stored message | 420 (120 of text, 300 of envelope and indexes) | ASSUMED; measure your own envelope, it is usually larger than people guess |
| Tap to visible "sending" state | under 50 ms | DERIVED: roughly three frames at 60 hertz, which is the threshold at which a tap stops feeling connected to its result |
| Send to delivered, both parties online | under 1 s at p95 | ASSUMED; it is the point at which a conversation stops feeling live |
| Duplicate messages visible to a user | zero | GIVEN by the product |
| Sends lost to process death | zero | GIVEN by the product; this is the requirement that forces the outbox |
| Battery budget for messaging while backgrounded | under 1 percent of the battery per day | ASSUMED; a messaging app that shows up in the OS battery screen gets uninstalled |

### Estimation that eliminates (chat client)

Assume a 4,000 mAh battery at 3.85 V, which is 15.4 Wh, or 55,440 J. Assume a cellular radio that stays in a connected state for 10 s after any transmission, drawing 800 mW there against 10 mW idle. Both are planning assumptions; measure the real values on your target devices.

| Calculation | Result | What it rules out |
|---|---|---|
| 200,000 messages times 420 bytes | 84 MB of local history | Rules out holding history in memory, and rules out a plain file. It also rules out panic: 84 MB is nothing on a modern phone |
| 84 MB plus an assumed 40 percent full text index overhead | 118 MB total on disk | Rules out server side search as a requirement driver. Local search over 118 MB is an embedded database feature, not a project |
| 160 message deliveries per device per day times 300 bytes | 48 KB per device per day | Rules out justifying a binary wire format on bandwidth. If you want one, justify it on parse cost or wakeups instead, and say so |
| One isolated radio wakeup: 10 s tail times 0.79 W | 7.9 J per wakeup | Rules out reasoning about network cost in bytes. On a phone the cost of a transfer is dominated by the tail, not the payload |
| Polling every 60 s while backgrounded: 1,440 wakeups times 7.9 J | 11,376 J, which is 20.5 percent of the battery per day | Rules out background polling entirely, at any interval under about 20 minutes |
| Push delivery at an assumed 1 J per wake and fetch, times 160 | 160 J, which is 0.29 percent of the battery per day | Rules out a client owned background socket. The OS already holds one connection for push; sharing it is two orders of magnitude cheaper |
| Foreground socket heartbeat every 30 s, against an assumed 800 mW screen | Heartbeat energy is a rounding error while the screen is on | Rules out heartbeat interval tuning as a foreground concern. It matters only for long lived background sessions |
| AES-GCM at an assumed 1 GB per second with hardware acceleration, on a 120 byte message | 0.12 microseconds | Rules out symmetric encryption as a latency or battery concern. Anyone who says encryption is why chat is slow has not measured it |
| 5,000 member group times 3 devices, one envelope each at 300 bytes | 15,000 envelopes, 4.5 MB uploaded per message sent | Rules out per device sender fan out for large groups. This single number is why shared group keys exist |
| The same group with a shared sender key: one ciphertext uploaded, key distribution only on membership change | 300 bytes per message | Rules out the claim that end to end encryption cannot scale to large groups; it rules in the membership change cost instead |
| 84 MB of history transferred to a new device at an assumed 20 Mbps | 34 s | Rules out a blocking spinner at sign in. History sync must be background, resumable, and ordered most recent first |
| 10 million clients reconnecting after a server restart, spread over 300 s of jitter | 33,000 reconnects per second | Rules out fixed reconnect delay. Without jitter the same event is 10 million in the first second, which is a self inflicted denial of service |

!!! note "Where these numbers come from"

    The battery, radio and cryptography rates above are planning assumptions chosen to be conservative and easy to substitute. The structure is what transfers: a phone pays for connections, not for bytes; symmetric cryptography is free; and group fan out cost is a multiplication you must do out loud.

### V0, the simplest thing that works (chat client)

**Predict first.** Before reading further: at 40 sends per day, what is the first thing that goes wrong in the simplest possible chat client?

??? note "Show answer"

    Not throughput and not battery. The first failure is a message the user believes they sent. The app is killed by the OS while a send is in flight, the retry lived in memory, and the message is gone with no error and no trace. Every other feature on this page is negotiable; that one is not, which is why the outbox is in V0 and not in V1.

```
+-----------+   +------------------+   +------------------+
| Chat UI   |-->| Outbox table     |-->| Connection       |
| composer  |   | durable, keyed   |   | socket if fore   |
+-----------+   +------------------+   +------------------+
      |                  |                      |
      |                  |                      v
+-----------+   +------------------+   +------------------+
| Message   |<--| Apply inbound    |<--| Push wake, then  |
| store     |   | dedupe by id     |   | fetch since ack  |
+-----------+   +------------------+   +------------------+
```

**Caption.** V0 has six client components: a composer, a durable outbox table keyed by a client generated identifier, a socket used only while the app is in the foreground, a push wake path that triggers a fetch rather than trusting the payload, an inbound apply step that deduplicates by identifier, and a local message store that the screen reads from.

Why this is correct at the stated scale:

- 40 sends and 160 receives per day is no throughput problem at all. Every design decision here is about durability and wakeups.
- The outbox is in V0 because "sends lost to process death: zero" is a stated requirement, and no amount of retry logic in memory satisfies it.
- Push triggers a fetch rather than carrying content, so the design does not depend on payload size limits and does not leak message text into a third party notification service.
- The store is the only read path, so the UI has one source of truth from the first version and never grows a second.

!!! warning "When not to hold a socket at all"

    If your product's messages are not conversational (order confirmations, alerts, a support thread with hour scale replies), do not open a socket even in the foreground. Push plus a fetch on foreground covers it, and you avoid a connection state machine, a heartbeat, a reconnect policy and a reconnect storm. The measurement that justifies a socket is a p95 send to delivered requirement under a few seconds while both parties are actively looking at the screen.

### Breaking point, then V1 (chat client)

**Breaking point one: gaps and duplicates, visible the first time a connection drops mid conversation.**

What fails. The client deduplicates by identifier, so it never shows the same message twice, but it has no way to know that a message it never received exists. The reader sees a reply to something they cannot see. The mirror bug is a send whose acknowledgement was lost: the client retries, the server accepts it as new, and the recipient sees the message twice.

How you observe it. Give every message a server assigned per conversation sequence number. The client keeps the highest contiguous sequence it holds. Any received sequence more than one above it is a gap; count gaps per session and publish the number. Duplicates are detected the same way: a sequence you already hold arriving again.

V1 makes delivery verifiable:

```
+-----------+   +------------------+   +------------------+
| Chat UI   |-->| Outbox: pending  |-->| Sender: idem key |
| composer  |   | sent, acked, dead|   | bounded retries  |
+-----------+   +------------------+   +------------------+
      |                  |                      |
      |                  |                      v
+-----------+   +------------------+   +------------------+
| Message   |<--| Gap detector     |<--| Receiver: seq n  |
| store     |   | backfill range   |   | ack highest held |
+-----------+   +------------------+   +------------------+
```

**Caption.** V1 gives the outbox an explicit state machine, attaches an idempotency key to every send so a retry is not a new message, and adds a gap detector on the receive side that requests the missing sequence range rather than waiting for it to arrive.

| V1 change | Justified by | Cost you accept |
|---|---|---|
| Idempotency key per send | Duplicate sends after a lost acknowledgement | The server must keep a key to sequence mapping for at least the client's maximum retry window |
| Server assigned per conversation sequence | Gaps are undetectable without one | The server owns ordering, so a truly offline peer to peer mode is off the table |
| Gap detector with a range backfill | A reply arriving before the message it replies to | The client needs a "fetching earlier messages" state in the UI, and must decide whether to render the reply meanwhile |
| Explicit outbox states | The "sending forever" spinner with no way out | Four states to render instead of two, and a product decision about when to give up |
| Bounded retries with full jitter | 10 million clients reconnecting in the same second | Some sends fail permanently and the user must be told, which is a design surface nobody wants to build |

!!! warning "When not to add per conversation sequence numbers"

    If the product tolerates rare reordering and never shows replies or threads, sequence numbers add server side state per conversation and a backfill endpoint for a problem the reader will not notice. The measurement that justifies them is a non zero observed gap rate, which you can only get by adding sequence numbers, so the honest version is: add them if the product has any reply, quote or thread feature, because those make a gap visible.

**Breaking point two: three devices that disagree, visible on the first day of use.**

What fails. The phone marks a conversation read; the tablet still shows a badge. The laptop sends a message; the phone never shows it in the sent list. Delivery to devices is treated as a fan out of messages, but read state, deletions, mutes and the user's own sends are account level events that every device needs and no device produces on the wire.

How you observe it. On each device, checksum the conversation state (last read sequence, message count, unread count) and report it. A cross device audit that compares checksums for the same account is the only way this surfaces before a support ticket. Publish the disagreement rate.

V2 introduces an account level event stream:

```
+-----------+   +------------------+   +------------------+
| Device A  |-->| Account event    |<--| Device B         |
| own cursor|   | log, ordered,    |   | own cursor       |
+-----------+   | per account      |   +------------------+
      |         +------------------+           |
      |                  |                     |
+-----------+   +------------------+   +------------------+
| Local     |   | Device C         |   | Key + device     |
| store A   |   | own cursor       |   | registry         |
+-----------+   +------------------+   +------------------+
```

**Caption.** V2 gives the account a single ordered event log that carries messages, read markers, deletes and the user's own sends. Each device consumes it independently at its own cursor, and a device registry tracks which devices exist and hold which keys.

| V2 change | Justified by | Cost you accept |
|---|---|---|
| Account level ordered event log | Read state and self sends have no other delivery path | Every device now has a cursor that can fall behind, and a device that has been offline for months has a long replay |
| Self fan out (your send goes to your other devices too) | The laptop's message never appearing on the phone | Your own devices count towards group fan out arithmetic, which matters at 5,000 members |
| Device registry | You cannot encrypt to a device you do not know about | Adding or removing a device becomes a distributed operation with a security consequence, covered in deep dive two |

### Deep dive one: the outbox, from tap to acknowledged

This dive is chosen because the requirement "zero sends lost to process death" is the only hard guarantee on the page, and it is the thing candidates most often hand wave. The feed client's dives were about memory and about mutation under pagination; this one is about durability across a process boundary.

**The state machine, and what each transition costs.**

| State | Meaning | Enters when | Leaves when |
|---|---|---|---|
| Composed | Written to the outbox and to the store, rendered with a clock icon | The user taps send, before any network call | The sender picks it up |
| In flight | Handed to the transport with an idempotency key | A connection is available and this is the oldest pending item in the conversation | An acknowledgement or an error arrives, or the attempt times out |
| Acked | The server has assigned a sequence number | Acknowledgement received | Never; this is terminal for the send path |
| Retrying | Waiting out a backoff | A retryable error or a timeout | The backoff expires, or the retry budget is exhausted |
| Failed | The user must be told | The retry budget is exhausted, or a permanent error arrives | The user retries manually, or deletes it |

**The ordering rule that is easy to get wrong.** Send one message per conversation at a time. If you send three in parallel and the second fails, the first and third arrive and the reader sees a conversation with a hole in it that fills in later. Serialising per conversation costs latency only when there is a queue, and a queue only exists when the network is bad, which is exactly when ordering matters most.

**Writing before sending, and why the order is not negotiable.**

1. Write the message to the outbox and the store inside one transaction, with a client generated identifier.
2. Only then update the UI. The UI reads from the store, so this is automatic if the store is the single source of truth.
3. Only then attempt the network call.

Reverse steps one and three and you have built the bug: the process dies between the network call and the write, and now the server has a message the client has never heard of. The user's device shows nothing; the recipient sees the message. That asymmetry is far worse than a message that simply failed.

**Idempotency keys, sized honestly.** The key is the client generated identifier. The server stores a mapping from key to assigned sequence for a retention window. That window must exceed the client's maximum retry span. If the client retries for up to 24 hours (a plausible offline period), a one hour server side key retention silently converts a retry into a duplicate. State the window out loud in the interview; it is the kind of cross boundary contract that separates a senior answer.

**The four retry classes, because "retry with backoff" is not an answer.**

| Class | Example | Action |
|---|---|---|
| Transient network | No route, timeout, connection reset | Retry with exponential backoff and full jitter, no user visible error until the budget is spent |
| Server transient | 500, 503, or an explicit retry after hint | Retry, and honour the hint rather than your own schedule |
| Client permanent | Message too large, blocked recipient, invalid conversation | Do not retry. Move to failed and tell the user what to change |
| Ambiguous | The request was sent and the response never arrived | Retry with the same idempotency key. This is the whole reason the key exists |

**Surviving process death.** The outbox is a table, so it survives by construction. Two things still need designing:

- On launch, scan the outbox for items in "in flight" and move them back to "retrying". A message left in flight forever is the "sending" spinner that never resolves, and it is the single most common chat client bug report.
- Schedule a deferrable background job to drain the outbox, so a message composed offline sends before the user next opens the app. Deferrable, not immediate, because at 7.9 J per isolated wakeup, draining eagerly for one queued message is the wrong trade.

**Editing and deleting a queued message.** The user writes a message offline, then deletes it before it sends. If the outbox row is simply removed, and the send had actually reached the server, the message exists for the recipient and not for the sender. The correct behaviour is to keep the row, mark it cancelled, and let the sender decide: if it has not been dispatched, drop it; if dispatch is ambiguous, send it and immediately send a delete. Candidates who name this have shipped chat.

**A worked trace, so the states are concrete.** A user on a train sends three messages while the connection is dead, then walks into signal:

| Time | Event | Outbox state | What the user sees |
|---|---|---|---|
| 0 s | Tap send, message A | A: composed | A with a clock icon, immediately |
| 3 s | Tap send, B and C | A, B, C: composed | Three messages, three clocks |
| 5 s | Sender attempts A, times out | A: retrying, B and C: composed | No change, which is deliberate |
| 65 s | Signal returns, A dispatched, acked seq 41 | A: acked, B: in flight | A gets a tick, B and C still show clocks |
| 66 s | B acked seq 42, C dispatched | B: acked, C: in flight | Ticks appear in order |
| 67 s | C acked seq 43 | All acked | Conversation is consistent, and order is preserved |

Note what did not happen: B and C were never dispatched in parallel with A, so no hole could appear.

!!! warning "When not to build a durable outbox"

    If a failed action is cheap to redo and the user would notice its absence immediately, an in memory retry plus an error message is enough. A durable outbox costs a table, a state machine, a background drain job, a launch time recovery scan and a cancellation protocol.

    The measurement that justifies it is any non zero rate of user actions lost to process termination, which you only see if you log a "process died with pending work" event on the next launch. For chat the answer is always yes; for a settings toggle it is always no.

### Deep dive two: what multi device costs once the content is encrypted

The second dive deliberately leaves the durability topic behind. This one is about a cost that only appears when you combine two requirements that each look reasonable alone: end to end encryption, and the same account on three devices.

**The core fact.** Encryption is to a device, not to a person. Three devices per account means a two person conversation has six endpoints, and every message is encrypted separately for each of the five that are not the sender.

**How the cost scales, computed rather than asserted.**

| Conversation | Recipient devices | Envelopes per message | Bytes uploaded per message |
|---|---|---|---|
| One to one, 3 devices each | 5 (their 3 plus your other 2) | 5 | 1.5 KB |
| Group of 50, 3 devices each | 149 | 149 | 45 KB |
| Group of 5,000, 3 devices each | 14,999 | 14,999 | 4.5 MB |

At 45 KB per message a 50 person group is affordable on a phone. At 4.5 MB it is not: on the 1.5 Mbps link from case study 1 that is 24 s to send one message, and the sender pays it every time.

**The shape of the fix, and its cost.** Encrypt the message once under a group key, and distribute that key to each device using the pairwise sessions you already have. Then:

| Property | Per device fan out | Shared group key |
|---|---|---|
| Bytes per message sent | Grows with member count | Constant |
| Cost on membership change | Zero | One key distribution to every remaining device |
| Forward secrecy after a member leaves | Automatic, they simply stop receiving | Requires rotating the key, or the leaver can read future messages |
| Sender attribution | Inherent, each envelope is from you | Requires a per sender signing key inside the group |

The trade is explicit: constant per message cost bought with a per membership change cost. In a group that churns every few minutes, that is a bad trade. In a group of 5,000 that churns daily, it is the only trade.

**The three things a new device forces you to answer.**

1. **Which devices exist.** A registry the server holds, that every sender reads before encrypting. If it is wrong or stale, either a device silently cannot decrypt, or a device you removed can still read.
2. **How the user notices a new device.** The registry is the security boundary, so a server that silently adds a device can read everything from then on. The mitigation is a safety number or key fingerprint the user can compare, plus a visible notification when the device set changes.
3. **What history the new device gets.** This is the hard one, and it has three answers, none of them free.

| History approach | Mechanism | What it costs |
|---|---|---|
| Nothing | New device starts empty | Users hate it, and it makes a new phone feel like a new account |
| Device to device transfer | Existing device sends the 84 MB store over a local link | Both devices must be present and awake; 34 s at 20 Mbps, longer over a local link; and the old device may be lost, which is the case you actually need to cover |
| Encrypted server backup | The store is encrypted under a key derived from a user secret and stored server side | The backup key becomes the weakest link. A short user chosen code is guessable; a long random key gets lost, and losing it means losing the history |

**Where the client work actually goes.** Once content is opaque to the server, four features move onto the device:

| Feature | Server side version | Client side version | Client cost |
|---|---|---|---|
| Search | Server index over all history | Local full text index over 118 MB | Index build on first sync, and incremental updates on every message |
| Unread counts | Server counts | Client counts from its own store, converged through the account event log | Diverges whenever a device is behind, which is why the audit metric exists |
| Media thumbnails | Server generates | Client generates on receive | Decode and resize on a background thread, at the cost from case study 1 |
| Link previews | Server fetches | Client fetches, which leaks the link to the preview target | A privacy decision, and a per conversation setting in most real products |

**The cryptographic cost, in perspective.** A symmetric encryption of a 120 byte message is 0.12 microseconds. A ratchet step is an elliptic curve operation, on the order of tens of microseconds. Even at 15,000 recipient devices, the arithmetic is not the problem; the 4.5 MB upload is. Say this explicitly in an interview, because the reflex answer is that encryption is expensive, and the measurement says otherwise.

!!! warning "When not to use end to end encryption on the client"

    If the product's value depends on server side processing of content (search across an organisation's messages, moderation, retention for compliance, or an assistant that reads the thread), end to end encryption removes the feature, not just the convenience. It costs you a device registry, a key backup story, local search, client side previews and a permanent support burden when someone loses every device at once.

    The measurement that justifies it is a threat model in which the service operator is in scope, which is a product decision and not an engineering one.

### Failure modes and evaluation (chat client)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Duplicate delivery | A lost acknowledgement turns one send into two messages | Idempotency key per send, with a server retention window that exceeds the client's maximum retry span |
| Message loss on process kill | The user sees a sent message that no recipient received, or vice versa | Write to the outbox before dispatch, and rescan for stranded in flight rows on every launch |
| Thundering herd | A chat service restart drops 10 million connections and every client reconnects at once | Exponential backoff with full jitter spread over minutes, and a server signal that can widen the spread |
| Retry storm | Every queued send retries the moment connectivity returns, on every device at once | Per conversation serialisation, a per session retry budget, and a cap on concurrent in flight sends |
| Clock skew | A device with a wrong clock stamps its message in the past and it sorts above the conversation | Order by server assigned sequence only; render the device clock nowhere except as a display hint |
| Split brain across devices | Read state, mutes and deletes disagree between phone and tablet | An account level ordered event log with a per device cursor, plus a cross device state checksum audit |
| Dead push tokens | Messages silently stop arriving after a reinstall or a restore | Re-register the token on every launch, and reap tokens server side on the first permanent rejection |
| Badge drift | The notification badge and the in app unread count disagree | Derive both from the same store value, and never increment a badge from a push payload |
| Metastable failure | After an outage, replay of every device's backlog keeps the service above capacity | Prioritise recent conversations first, cap replay batch size, and let the client take backlog in bounded chunks |

Service level indicators, measured on the device:

- Send to acknowledged latency at p50 and p95, split by whether a socket was already open.
- Tap to visible sending state at p99, which should be a frame count and not a network number.
- Outbox drain latency at p95, measured from composition to acknowledgement including offline time.
- Permanent send failure rate per 1,000 sends.
- Gap rate and duplicate rate per conversation per session, both of which should be zero.
- Cross device state disagreement rate, from the checksum audit.
- Background energy attributed to the app per day, and wakeups per day.
- History sync completion rate and time to first usable conversation on a new device.

Alert on: any non zero permanent send failure rate above a small threshold; gap rate above zero on any app version; wakeups per day crossing the budget after a release, because a background regression is invisible until it reaches the OS battery screen.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Immediate send feedback under 50 ms | Yes, because the store write is local and the UI reads the store |
| Exactly once to the reader's eye | Yes: idempotency keys stop duplicates on send, sequence numbers detect gaps on receive |
| Zero sends lost to process death | Yes, and only because the write precedes the dispatch |
| Convergence across three devices | Yes through the account event log, and the checksum audit is how you know rather than hope |
| Local search with no network | Yes at 118 MB of store and index, which the estimation showed is affordable |
| Under 1 percent of the battery per day | Yes on push at 0.29 percent; no on any polling design, which is why polling is not on the diagram |
| Send to delivered under 1 s at p95 | Yes when a socket is open; when the app is backgrounded this becomes a push latency number you do not control, and you should say so |

### Where this answer is a simplification (chat client)

- **Multi device session management is a specified problem, not an improvised one.** [The Sesame Algorithm](https://signal.org/docs/specifications/sesame/) (Marlinspike and Perrin, revision 2, 14 April 2017; checked 1 August 2026) sets out the device registry, the active and inactive session handling, and the failure cases (simultaneous initiation, restored backups, erased devices) that this page compresses into one table.
- **The ratchet itself has moved.** [The Double Ratchet Algorithm](https://signal.org/docs/specifications/doubleratchet/) (Perrin, Marlinspike and Schmidt, revision 4, 4 November 2025; checked 1 August 2026) now includes post quantum extensions. If you cite forward secrecy in an interview, cite the version you actually read.
- **Transport choice deserves more than one row.** [MQTT Version 5.0](https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html) (OASIS Standard, 7 March 2019; checked 1 August 2026) is the standard worth reading if you propose a lightweight publish and subscribe transport, and [RFC 9000, QUIC](https://www.rfc-editor.org/rfc/rfc9000) (May 2021; checked 1 August 2026) is the reason connection migration across a network handoff is a transport property rather than an application one.
- **Push delivery semantics are best effort, and the page treats them as reliable.** Real push services drop, collapse and delay. The design above survives it only because push triggers a fetch rather than carrying content, which means a dropped push costs latency and not correctness. That is a deliberate design property and worth stating as one.
- **What this page skips.** Media send (resumable upload with its own durable queue), voice and video calls, message reactions and their convergence rules, disappearing messages, and the moderation and abuse surface, which is genuinely harder than everything above when the operator cannot read content.
- **The organisational reality.** Three platforms implement this state machine independently, and they diverge. The durable fix is a shared specification with a conformance test suite that all three run, not three careful teams.

### Level delta (chat client)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Draws a socket, a server and a database | Asks whether it is end to end encrypted before drawing | Asks the same, then computes the 5,000 member fan out at 4.5 MB per message in the first five minutes and lets that shape the whole answer |
| Send path | Calls the API and shows a spinner | Adds a retry with backoff and an offline queue | Writes to the outbox before dispatch, names the process death asymmetry, serialises per conversation, and specifies the idempotency key retention window as a cross boundary contract |
| Ordering | "The server orders it" | Server sequence numbers per conversation | Adds gap detection with a range backfill, and refuses to sort by any device clock |
| Multi device | Mentions sync | Proposes a per device cursor | Designs an account level event log carrying read state, deletes and self sends, and adds a cross device checksum audit because divergence is otherwise invisible |
| Battery | "We use push, not polling" | Explains that push shares the OS connection | Derives 20.5 percent per day for polling against 0.29 percent for push from a wakeup tail cost, and uses the number to close the discussion |
| Encryption | "It is end to end encrypted" | Explains pairwise sessions per device | Shows the fan out arithmetic, moves to a shared group key, names the membership change cost and the forward secrecy consequence, and states that the cryptography is free and the upload is not |
| History on a new device | Not mentioned | Suggests a server backup | Lays out the three options with their costs, and names the backup key as the weakest link in the entire system |
| Failure handling | "Retry" | Backoff with jitter | Four named retry classes with different actions, a launch time recovery scan for stranded sends, and a cancellation protocol for a queued message the user deleted |

What each level added:

- **Mid to senior.** The design acquires a transport rationale and a queue, and stops assuming the network works.
- **Senior to staff.** The candidate treats durability across a process boundary as the hard requirement, derives the battery and fan out numbers rather than asserting them, and designs the observability (gap rate, checksum audit) that would let anyone else find these bugs later.

---
## Case study 3: Maps client

The feed client was bounded by memory and the chat client by wakeups. The maps client is bounded by two things at once: a rendering budget during a gesture, and a power budget during a two hour drive. It is also the first prompt on this page where the user explicitly asks for content to be stored before it is needed, which turns a cache into a product feature.

### The prompt (maps client)

"Design the mobile maps client: pan and zoom a world map, search for a place, turn by turn navigation, and let the user download a city so it works with no signal."

### Questions that fork the design (maps client)

| Question worth asking | If A the design becomes | If B it becomes |
|---|---|---|
| Raster tiles or vector tiles? | Raster: the client is a textured quad grid, there is no styling engine, and offline storage grows enormously because every zoom level is a separate download | Vector: the client gains a renderer, a glyph atlas and a style runtime, but a single download serves many zoom levels and restyling needs no new bytes |
| Is turn by turn navigation in scope? | No: location can be sampled coarsely and rarely, and the battery section of this design mostly disappears | Yes: continuous location, screen on for long periods, route matching every fix, and power becomes the dominant non functional requirement |
| Is routing computed on the server or on the device? | Server: one small request per trip, but a dead network mid trip means no reroute when the driver misses a turn | On device: the offline pack must include a routing graph, which is a different artefact from tiles and roughly as large |
| How is an offline region chosen? | User drawn box: sizes are unbounded, so you owe the user a size estimate before they commit | Fixed city packs: predictable sizes, server built and cacheable, wasteful at region boundaries and useless for a rural trip |
| Is live traffic in scope? | No: tiles are near static and can be cached for weeks | Yes: a second overlay with a refresh cadence measured in minutes, so the cache policy splits into two policies with different expiry rules |
| Does location leave the device? | No: matching and search happen locally, and the privacy story is simple | Yes: an upload pipeline with batching, a retention policy, and a permission prompt whose denial you must design a working fallback for |
| Does navigation continue with the screen off or the app backgrounded? | Foreground only: OS lifecycle is easy and the power budget is dominated by the display | Background: a persistent notification, an OS entitlement, and throttling rules you must design around rather than fight |
| What is the storage floor we support? | 128 GB device: offline packs are effectively unconstrained | 32 GB device with 2 GB free: the pack size ceiling picks your maximum zoom level, which picks your tile format |

!!! tip "Say this out loud in the room"

    "I want to settle raster against vector early, because it decides the offline pack size by roughly a factor of thirty, and the offline requirement is the one thing in this prompt that is not negotiable."

### Requirements (maps client)

Functional:

- Pan, zoom and rotate a world map at the display refresh rate, with no blank areas during a normal gesture.
- Search for a place by name and show it on the map.
- Provide turn by turn guidance with rerouting when the user leaves the route.
- Download a user chosen region for offline use, with a size shown before the download starts.
- Continue navigation through a total loss of network inside a downloaded region.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Display | 1080 by 2400 pixels, 60 hertz | ASSUMED, matching the high tier from case study 1 |
| Frame budget during a gesture | 16.7 ms | DERIVED from 60 hertz |
| Blank tile frames during a normal pan | zero | GIVEN by the product; a grey checkerboard is the single most complained about maps defect |
| Battery for a 45 minute navigation session | under 15 percent | ASSUMED; it is roughly the point at which a driver without a charger starts worrying |
| Offline pack for a metropolitan area | under 200 MB | ASSUMED; it has to fit alongside photographs on a phone that is already full |
| Offline region area | 40 km by 40 km | ASSUMED; a metropolitan area plus its commuter ring |
| Free storage floor | 2 GB | ASSUMED; the low end of what a nearly full 32 GB phone offers |
| Tile memory ceiling | under 30 MB of texture | DERIVED below |
| Location fix rate during navigation | 1 Hz | ASSUMED; below this, turn announcements arrive late at motorway speeds |

### Estimation that eliminates (maps client)

The tile pyramid gives you almost all of these for free. At zoom level z the world is divided into 4 to the power z tiles, and the earth's circumference is about 40,075 km, so a tile spans 40,075 divided by 2 to the power z kilometres at the equator.

| Calculation | Result | What it rules out |
|---|---|---|
| 4 to the power 14 tiles, at an assumed 12 KB per vector tile | 268 million tiles, 3.2 TB for the world at zoom 14 | Rules out any notion of shipping or downloading a global map. Offline must mean a region |
| 40,075 divided by 2 to the power 16 | 0.61 km per tile at zoom 16 | Rules out nothing yet; it is the input to the next two lines |
| 40 km divided by 0.61, squared, for a 40 by 40 km box at zoom 16 | 66 by 66, which is 4,356 tiles | Rules out reasoning about offline packs in vague terms. This is a countable number |
| The full pyramid from zoom 0 to 16 is the top level count times (1 plus one quarter plus one sixteenth and so on), which converges to four thirds | 5,808 tiles, 70 MB at 12 KB each | Rules out the fear that vector offline packs are enormous. 70 MB fits the 200 MB budget with room for the routing graph |
| The same box in raster to zoom 18: 40,075 divided by 2 to the power 18 is 0.153 km, so 262 by 262 tiles, times four thirds, at an assumed 25 KB per raster tile | 91,525 tiles, 2.29 GB | Rules out raster tiles for offline entirely. It exceeds the 2 GB free storage floor for one city |
| An assumed 500,000 road segments in a 40 by 40 km metropolitan area at an assumed 60 bytes per segment | 30 MB of routing graph | Rules out treating the offline pack as "just tiles". Routing data is a comparable cost and a separate artefact |
| 1080 divided by 256, times 2400 divided by 256, rounded up | 5 by 10, which is 50 tiles on screen | Rules out per tile network fetches during rendering: 50 tiles is already 50 potential requests for one still frame |
| 50 on screen tiles plus a one tile border, so 7 by 12, at 256 by 256 by 4 bytes of texture | 84 tiles, 22 MB of GPU texture | Rules out an unbounded texture cache. Two screens of margin would already be 44 MB, which is a quarter of the low tier process budget from case study 1 |
| A fling covering 2 screens in 300 ms, at 50 tiles per screen | 100 new tiles in 300 ms, or 333 tile fetches per second | Rules out fetching tiles on demand during a gesture. The renderer must draw something it already has |
| An assumed 40 mW for the satellite receiver at 1 Hz, against an assumed 800 mW display and 600 mW for rendering | Location is 40 of 1,440 mW, or 2.8 percent of navigation draw | Rules out "turn off the satellite receiver to save battery during navigation". The screen and the renderer are the cost |
| 15.4 Wh of battery divided by 1.44 W | 10.7 hours of continuous navigation | Rules out treating a 45 minute commute as a power problem, and rules in the long road trip and the thermal question |
| Background location at 1 Hz with the screen off: 0.04 W over 24 hours, against 15.4 Wh | 0.96 Wh, which is 6.2 percent of the battery per day | Rules out continuous background location as a default. It forces a duty cycle or an OS level significant change trigger |
| Uploading a location batch every 60 s at an assumed 7.9 J per isolated radio wakeup | 474 J per hour, which is 0.86 percent of the battery per hour | Rules out per minute uploads. Batch to 5 minutes (0.17 percent per hour) or piggyback on a connection that is already open |

!!! note "What is assumed and why"

    Tile sizes, segment counts and component power draws above are planning assumptions, chosen because they are easy to replace with measurements from your own tiles and your own devices. The pyramid arithmetic is not an assumption: 4 to the power z tiles and a four thirds pyramid factor follow from the tiling scheme itself, and they are the numbers that do the eliminating.

### V0, the simplest thing that works (maps client)

**Predict first.** With a plain tile cache and a plain map view, what is the first thing a user will complain about, and what is the number behind it?

??? note "Show answer"

    Grey squares during a fling. The number is 333 tile fetches per second: a fling asks for 100 tiles in 300 ms, and no network can answer that. The fix is not a faster network or a bigger cache, it is that the renderer must be able to draw an approximation of a tile it does not have, which is a rendering decision rather than a caching one.

```
+-------------+   +------------------+   +-----------------+
| Map view    |-->| Tile source      |-->| Tile CDN        |
| GPU quads   |<--| memory LRU 22 MB |<--| /z/x/y vector   |
+-------------+   +------------------+   +-----------------+
       |                   |
       |                   v
+-------------+   +------------------+
| Location    |   | Disk tile cache  |
| provider    |   | LRU, 200 MB      |
| 1 Hz        |   +------------------+
+-------------+
```

**Caption.** V0 has five components: a map view drawing textured quads, a tile source with a memory cache sized to the on screen working set, a disk cache holding encoded tiles under a least recently used policy, a content delivery network serving vector tiles by zoom, column and row, and a location provider sampling at 1 Hz.

Why this is correct at the stated scale:

- Vector tiles let one downloaded zoom level serve several displayed zoom levels, which is what keeps 70 MB from becoming 2.29 GB.
- A 22 MB memory cache is exactly the on screen working set plus a one tile border, derived rather than guessed.
- A disk cache with a least recently used policy is genuinely enough for a user who stays in one city and has signal.
- Nothing here needs a database, a background job, or a download manager, and each of those is a subsystem you would otherwise have to justify.

!!! warning "When not to use vector tiles"

    If your map is a static illustration (a store locator, a delivery tracking screen, a fixed campus map), raster tiles at two or three zoom levels are simpler, need no renderer, no font atlas and no style runtime, and add far less to your binary size. The measurement that justifies vector is either an offline pack requirement, or a styling requirement that would otherwise force a re-download of every tile. Below roughly three supported zoom levels and with no offline requirement, vector tiles are over engineering.

### Breaking point, then V1 (maps client)

**Breaking point one: the fling, at 333 tile requests per second.**

What fails. During a fling the viewport crosses 100 tile boundaries in 300 ms. Every one of them is a cache miss on the first pass through a new area. The renderer has nothing to draw, so it draws the background colour, and the user sees a grey checkerboard sweeping across the screen. Requests queued during the fling arrive after it ends, so the network work was mostly wasted too.

How you observe it. Count frames in which any on screen tile slot had no content, and publish that as a share of gesture frames. Separately, count tile requests issued during a gesture that completed after the gesture ended: that is your wasted request rate. If the two move together, this is the failure, and a bigger cache will not fix a first visit.

V1 changes what the renderer is allowed to draw:

```
+-------------+   +------------------+   +-----------------+
| Map view    |-->| Tile pyramid     |-->| Request queue   |
| draws best  |   | parent fallback  |   | priority by     |
| available   |<--| child upgrade    |<--| viewport dist   |
+-------------+   +------------------+   +-----------------+
       |                   |                      |
       |                   v                      v
+-------------+   +------------------+   +-----------------+
| Location    |   | Disk tile cache  |   | Tile CDN        |
| provider    |   | LRU, 200 MB      |<--| vector tiles    |
+-------------+   +------------------+   +-----------------+
```

**Caption.** V1 lets the renderer draw a scaled up parent tile whenever the exact tile is missing, then swaps in the child tile when it arrives, and puts every tile request through a priority queue ordered by distance from the viewport centre with cancellation for requests that leave the screen.

| V1 change | Justified by | Cost you accept |
|---|---|---|
| Parent tile fallback | Zero blank frames is a stated requirement and 333 requests per second cannot be served | The map is briefly blurry rather than briefly blank, and you must decide how many levels up you will scale before giving up |
| Priority queue by viewport distance | The centre of the screen matters more than the corner and both were equal before | A scheduler with a comparator that someone will get wrong during a refactor, so it needs a test |
| Cancellation when a tile leaves the viewport | Wasted requests completing after the gesture | A cancelled in flight transfer is wasted bandwidth, so cancel the decode always and the transfer only if it has not started |
| Bounded upload of textures per frame | Uploading 100 textures in one frame blows the 16.7 ms budget on its own | Tiles appear over two or three frames instead of one, which is invisible to the eye and visible in the frame timing chart |

!!! warning "When not to add a tile request scheduler"

    A map that does not support free panning (a fixed route overview, a single pin) has no gesture to schedule for. The scheduler costs a priority comparator, a cancellation protocol and a concurrency limit, and it earns nothing when the viewport does not move. The measurement that justifies it is a wasted request rate above roughly 20 percent during gestures, which you can only see if you tag requests with the gesture that produced them.

**Breaking point two: the network dies mid trip and the cache turns out not to be an offline map.**

What fails. A least recently used disk cache holds whatever the user happened to pan across, at whatever zoom they happened to be at, and evicts it when something else needs the space. It does not hold the roads two towns ahead, it holds no routing graph, and it will happily evict the region the user downloaded last night because they browsed somewhere else this morning.

How you observe it. Run a chaos test: start navigation, disable the network at a random point in the route, and record whether the app can reach the destination without a blank map and without losing the ability to reroute. Publish that as an offline route completion rate. It will be far lower than anyone expects, and it is the only honest measurement of an offline claim.

V2 separates pinned content from cached content:

```
+-------------+   +------------------+   +-----------------+
| Map view    |-->| Tile lookup:     |-->| Pinned region   |
| draws best  |   | pinned first,    |   | store, no evict |
| available   |<--| then LRU cache   |<--| tiles + routing |
+-------------+   +------------------+   +-----------------+
       |                   |                      |
       |                   |                      |
+-------------+   +------------------+   +-----------------+
| Route engine|   | Disk tile cache  |   | Download job    |
| on device   |   | LRU, evictable   |   | resumable, est  |
+-------------+   +------------------+   +-----------------+
```

**Caption.** V2 adds a pinned region store that the eviction policy may never touch, holding both tiles and a routing graph, filled by a resumable background download job that shows a size estimate before it starts, and an on device route engine that can reroute with no network.

| V2 change | Justified by | Cost you accept |
|---|---|---|
| Pinned store separate from the cache | The cache evicted the thing the user explicitly asked to keep | Two stores with two lookup paths, and a user visible storage number you now owe them |
| Routing graph in the pack | Tiles alone let you see the map and not navigate it | 30 MB on top of 70 MB, and a second artefact with its own version and update cadence |
| Resumable download job | A 100 MB download over a mobile link will be interrupted | Range requests, a job store, and a resume point per artefact |
| Size estimate before download | The user cannot consent to a number they were not shown | The estimate must be honest, which is deep dive one |
| On device routing | A server reroute needs a network the user has just lost | A routing engine in the app, larger binary size, and route quality that may differ from the server's |

### Deep dive one: sizing an offline region before the user taps download

This dive is chosen because it is the only place in this prompt where the client owes the user a promise about the future. The feed dive was a memory ceiling, the chat dives were durability and key distribution; this one is about an estimate that has to be right before any work happens.

**What is actually in a pack.** Candidates say "the tiles". A pack has at least four artefacts, and three of them are not tiles.

| Artefact | Size for a 40 by 40 km region | Why it is separate |
|---|---|---|
| Vector tiles, zoom 0 to 16 | 70 MB | DERIVED above from 5,808 tiles at 12 KB |
| Routing graph | 30 MB | DERIVED above from 500,000 segments at 60 bytes |
| Search index for places in the region | an assumed 15 MB | Offline search is a separate requirement and people forget it until the demo |
| Style, fonts and sprite atlas | an assumed 8 MB | Shared across regions, so it is a fixed cost paid once, not per region |
| Total, first region | 123 MB | DERIVED, and inside the 200 MB budget |
| Total, each additional region | 115 MB | DERIVED, because the style is already there |

**Why the estimate must come from the server.** The client knows the bounding box. It does not know how many tiles in that box are non empty, and tile size varies by roughly an order of magnitude between an ocean tile and a dense city tile. An estimate computed as "tile count times average size" is wrong by a factor of three in both directions.

The server has the tile index, so it can sum actual byte counts for the requested box. The client asks, the server answers, and the user consents to a real number rather than a guess.

**The zoom level decision, which is the whole size lever.**

| Maximum packed zoom | Tiles in the box | Pack tile size | What the user loses |
|---|---|---|---|
| 14 | 289 top level, 385 with the pyramid | 4.6 MB | House numbers and small street names, because over zooming vector data four levels degrades label placement badly |
| 15 | 1,089 top level, 1,452 with the pyramid | 17 MB | Some minor road detail at high zoom |
| 16 | 4,356 top level, 5,808 with the pyramid | 70 MB | Little, in practice, because vector data over zooms cleanly by two levels |
| 17 | 17,424 top level, 23,232 with the pyramid | 279 MB | Nothing, and it costs four times the bytes for it |

The four times jump per level is not a surprise, it is the definition of the pyramid: each level has four times the tiles of the one above. Say that out loud, because it makes the zoom ceiling a one line decision rather than a judgement call. Zoom 16 is the answer here because it is the largest level that fits the 200 MB budget with the routing graph included.

**Over zoom is the property that makes vector worth its complexity.** A vector tile is geometry, so the renderer can draw zoom 16 data at zoom 20 by scaling the geometry and re-running label placement. A raster tile is pixels, so the same operation is a blur. This is why the raster comparison needed zoom 18 in the pack and the vector one did not, and it is where the 70 MB against 2.29 GB gap comes from.

**Designing the download job.**

1. Ask the server for a size estimate for the exact bounding box, and show it with the device's free storage next to it.
2. Refuse to start if the estimate exceeds free storage minus a safety margin. Running a device out of storage mid download is a failure that damages more than your app.
3. Download in ordered chunks: routing graph first, then tiles from the region centre outward. If the user cancels at 60 percent, an outward ordering means they have a usable smaller region rather than a useless scattered one.
4. Use range requests so an interrupted transfer resumes rather than restarts. This is a property of the transfer, defined in [RFC 9110, HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110) (June 2022; checked 1 August 2026).
5. Verify each artefact before marking the pack complete. A half written routing graph that the app trusts is worse than no pack, because it fails in the middle of a drive.
6. Record the actual size against the estimate and report the ratio. If your estimates are systematically 30 percent low, you want to know that from telemetry and not from reviews.

**Updating a pack.** Maps change. A pack downloaded in March is wrong by September, and the user will not re-download 123 MB to fix it.

| Update strategy | Bytes | Failure mode |
|---|---|---|
| Full re-download | 115 MB per region | Users on metered connections never update, so their maps rot |
| Per tile conditional requests | 5,808 conditional requests, most returning not modified | The requests themselves are the cost: 5,808 round trips at 300 ms with modest concurrency is minutes of work and thousands of wakeups |
| Server computed diff manifest | One request returning a list of changed tiles, then only those tiles | The server must retain enough version history to diff against whatever the client has, which bounds how stale a pack may be before it needs a full refresh |

The third is the only one that works, and the constraint it introduces is worth naming: the server can only diff against versions it still retains, so a pack older than the retention window must be re-downloaded in full. Tell the user that before they hit it.

!!! warning "When not to build offline regions"

    If your users are in continuous coverage and the product is not used while travelling, a well tuned tile cache with a generous size gives most of the benefit for none of the cost. Offline regions cost a second store, an eviction exemption, a download manager, a size estimate contract with the server, an update diff protocol and a storage settings screen.

    The measurement that justifies it is the share of sessions with a connectivity gap longer than the cache can cover, measured by logging network state transitions during map use, not by intuition about what users want.

### Deep dive two: the location duty cycle and the degradation ladder

The second dive moves from storage to power, which is the other constraint this prompt imposes. It also answers a question the estimation section deliberately left hanging: if the satellite receiver is only 2.8 percent of navigation draw, why does anyone talk about location and battery in the same sentence?

**Because the receiver is cheap only when it is already running.** The costs that matter are elsewhere:

| Cost | When it applies | Approximate size |
|---|---|---|
| Receiver acquiring a first fix from cold | After minutes without a fix, with no assistance data | Seconds of high draw, and the assistance data download that shortens it |
| Receiver tracking, already locked | Continuous navigation | 40 mW, which is the number from the estimation table |
| Keeping the process alive to receive fixes | Background tracking | The OS must not suspend you, which is a far larger cost than the receiver |
| Waking the radio to upload fixes | Any design that ships location off the device | 7.9 J per isolated wakeup, which dominates everything above |
| Keeping the screen on | Any foreground navigation | 800 mW, which is twenty times the receiver |

The design consequence is direct: optimise the wakeups and the screen, not the receiver. A design that samples at 1 Hz and uploads once every five minutes costs less than a design that samples at 0.2 Hz and uploads every fix.

**Accuracy tiers, and what each one is for.**

| Tier | Typical mechanism | Use it for | Do not use it for |
|---|---|---|---|
| Coarse, network derived | Nearby cell and wireless network identifiers | "Which city am I in", weather, a default search area | Anything with a turn in it |
| Balanced | Fused sensors, receiver duty cycled | Recording a walk, showing a blue dot on a browsed map | Motorway navigation, where a late fix is a missed junction |
| High, continuous | Receiver at 1 Hz plus inertial sensors | Turn by turn guidance | Anything backgrounded and indefinite |
| Event driven | Significant change or geofence triggers, handled by the OS | "Tell me when I am near the shop" | Anything needing a trace of where the user went |

**The degradation ladder.** State this as an ordered list in an interview, because it demonstrates that you know a design has to keep working when the environment refuses to cooperate.

1. **Battery below a threshold, or the OS enters a low power mode.** Drop the fix rate from 1 Hz to 0.2 Hz and lean harder on inertial sensors and map matching between fixes. Guidance quality degrades gently; announcements come slightly later.
2. **Screen off during navigation.** Keep the fix rate, because the announcements are the product now, but stop rendering. Rendering was 600 mW of the 1,440 mW total, so this alone extends the session by roughly 70 percent.
3. **No network.** Keep navigating from the pinned region. Disable live traffic, and say so in the interface rather than showing stale traffic as if it were current.
4. **Permission downgraded to approximate.** Turn by turn is impossible. Offer route overview and step by step directions the user follows manually, and explain exactly what precise location would restore. Never simply fail.
5. **Permission denied entirely.** The map still works. Search still works. Ask the user to drop a start pin. A maps app that shows an empty screen because it cannot locate the user has failed a design test, not a permission test.
6. **Thermal throttling.** Reduce the render rate first (a map at 30 frames per second is fine), then reduce the fix rate. Never drop announcements, which are the only part of navigation that is safety relevant.

**Map matching is what buys you the lower fix rate.** Between fixes, the client knows the road geometry, the last heading and the last speed. Projecting forward along the matched road segment is far more accurate than interpolating between two raw fixes, and it is what makes a 0.2 Hz design usable at all. It also fixes the urban canyon case where the raw fix jumps to a parallel street, because a jump that would require leaving the road network is rejected rather than rendered.

**Uploading location, if the product requires it.**

| Decision | Choice | Why |
|---|---|---|
| Batch size | 5 minutes of fixes, or 300 fixes at 1 Hz | 12 wakeups per hour instead of 60, which is 0.17 percent of battery per hour instead of 0.86 |
| Flush triggers | Batch full, app backgrounded, trip ended, or a connection is already open for another reason | Piggybacking on an existing connection makes a batch free, because the tail is already being paid |
| Durability | On disk queue, same shape as the chat outbox | A dropped trip trace is a support ticket, and the queue is a solved problem from case study 2 |
| Precision | Truncate coordinates to the precision the feature actually needs | You cannot leak precision you never collected, and this is the cheapest privacy control there is |
| Retention | Delete on the device once acknowledged, and state a server retention period | Location history is the most sensitive data most apps hold |

**What the whole navigation session costs, recomputed.** With the screen on at 800 mW, rendering at 600 mW, the receiver at 40 mW and uploads batched at 5 minutes, total draw is about 1.44 W plus a small upload term. Over a 45 minute drive that is 1.08 Wh out of 15.4 Wh, which is 7 percent, comfortably inside the stated 15 percent budget.

Turn the screen off and it falls to roughly 3 percent. Upload every fix instead of batching and you add roughly 0.86 percent per hour, which is small here and is the number that ruins a background tracking feature.

!!! warning "When not to run continuous high accuracy location"

    If the feature only needs to know that the user arrived somewhere, use the OS's event driven geofence or significant change triggers instead. Continuous tracking costs you a foreground service, a persistent notification, an entitlement review, 6.2 percent of the battery per day and a privacy posture you must defend. The measurement that justifies continuous location is a feature that needs the path and not just the endpoints, which is a product question you should ask before you design anything.

### Failure modes and evaluation (maps client)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Thundering herd | A style or tile version bump invalidates every client's cache at once and the delivery network takes the full miss rate | Version the tile URL, roll the new version to a small share first, and let old tiles stay valid rather than purging |
| Cache stampede | Many clients request the same newly published tile before the edge has it | This is an edge concern, but the client contributes by using stable, cacheable tile URLs and by not appending per user parameters |
| Background job killed | The OS terminates the offline download at 70 percent | Resumable range requests plus a persisted resume point per artefact, and a job that reschedules rather than restarts |
| Storage exhausted mid download | The device fills up and the pack is half written | Check free storage against the estimate before starting, reserve space if the platform allows it, and verify artefacts before marking a pack usable |
| Permission revoked mid session | The user downgrades to approximate location while navigating | Detect the change, degrade to step by step directions, and tell the user what changed rather than silently getting worse |
| Clock skew | Tile expiry is evaluated against a wrong device clock and every tile looks stale, or none do | Use server supplied validators rather than absolute expiry where possible, and treat a wildly wrong clock as a signal to revalidate |
| Metastable failure | After a network outage, every queued tile request retries at once and the queue never drains | Bound the queue, drop rather than retry tiles whose viewport has moved on, and use full jitter on the retries you keep |
| Thermal throttling | A phone in a windscreen mount in the sun drops to a fraction of its frame rate | Reduce render rate before fix rate, and never drop voice guidance |
| Silent data rot | A downloaded pack is six months old and routes the user down a road that is now closed | Diff based updates, an age indicator on each pack, and a hard limit beyond which the pack must be refreshed |

Service level indicators:

- Blank tile frame share during gestures, and p99 frame duration during gestures.
- Wasted tile request rate, defined as requests completing after their viewport left the screen.
- Offline route completion rate under injected network loss, measured as a scheduled test rather than in production.
- Pack download success rate, resume rate, and estimate to actual size ratio.
- Battery draw per navigation hour, segmented by screen on and screen off.
- Wakeups per hour attributable to location upload.
- Time to first fix at p50 and p95, segmented by cold and warm start.
- Reroute latency at p95, split by on device and server routing.

Alert on: pack download success rate dropping after a release; estimate to actual ratio drifting beyond a band, because it means the user is consenting to a wrong number; and battery draw per navigation hour crossing its budget on any device family.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Zero blank tile frames during a pan | Yes, because the renderer draws a scaled parent rather than nothing; the cost is a brief blur |
| 60 hertz during a gesture | Yes, provided texture uploads are bounded per frame; that bound is a real constraint and is easy to regress |
| Under 200 MB per metropolitan pack | Yes at 123 MB for the first region and 115 MB after, with zoom 16 as the ceiling |
| Navigation continues with no network | Yes inside a pinned region, with on device routing and rerouting; no outside it, and the user must be told where the boundary is |
| Under 15 percent battery for 45 minutes | Yes at roughly 7 percent with the screen on, and about 3 percent with it off |
| Tile memory under 30 MB | Yes at 22 MB for the on screen set plus a one tile border, which is a derived bound and not a tuning knob |
| Works on a device with 2 GB free | Yes for one pack; two packs at 115 MB each still fit, but the app owes the user a storage screen that says so |

### Where this answer is a simplification (maps client)

- **The tile pyramid arithmetic is the only part of this that is not an assumption.** Four to the power z tiles per level and a four thirds pyramid factor follow from the tiling scheme. Everything else (tile byte sizes, segment counts, power draws) is a placeholder for your own measurements, and this page labels them as such wherever they appear.
- **Resumable download is an HTTP property, not a feature you invent.** Range requests and conditional requests are defined in [RFC 9110, HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110) (June 2022; checked 1 August 2026), and cache behaviour in [RFC 9111, HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111) (June 2022; checked 1 August 2026). If you propose a bespoke chunking protocol in an interview, be ready to say what it buys over these.
- **Transport matters for a client that changes networks constantly.** A car drives between cells and in and out of wireless coverage. Connection migration is a transport level property, see [RFC 9000, QUIC](https://www.rfc-editor.org/rfc/rfc9000) (May 2021; checked 1 August 2026).
- **What this page skips.** Map data licensing (which constrains what you may cache and for how long, and is often the real reason offline packs expire), address geocoding, indoor positioning, the accessibility of a map surface (which is genuinely hard and under solved), lane guidance, and speed limit data quality.
- **Routing quality is its own discipline.** An on device engine that produces a different route from the server one is a support problem, and reconciling them is harder than either engine. The honest interview answer is that on device routing is a degraded mode with a stated quality gap, not a replacement.
- **The hardest real problem is freshness.** A downloaded region is a snapshot of a world that keeps changing. Everything on this page about diff updates is the easy half; deciding how stale is too stale, and what to do when a user's only pack is beyond that line while they are offline, is a product decision with no comfortable answer.

### Level delta (maps client)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Draws a map view, a tile cache and a tile server | Asks raster against vector before drawing | Asks the same, then does the pyramid arithmetic on the whiteboard and shows a factor of thirty between the two answers for the offline pack |
| Rendering under gesture | "Cache the tiles" | Adds a prefetch ring around the viewport | Computes 333 requests per second during a fling, concludes no caching strategy can win, and makes the renderer draw a scaled parent tile instead |
| Offline | "We download the tiles" | Adds a pinned store separate from the cache | Enumerates four artefacts in a pack, derives 123 MB, picks zoom 16 from a four times per level cost curve, and makes the server compute the estimate because the client cannot |
| Download job | Downloads with a progress bar | Adds resume and a background job | Orders the download centre outward so a cancel leaves something usable, verifies artefacts before use, and reports estimate against actual as a telemetry metric |
| Battery | "Location drains the battery" | Duty cycles the fixes | Shows the receiver is 2.8 percent of navigation draw, redirects the effort to the screen and the upload wakeups, and derives the batching interval from a 7.9 J tail cost |
| Degradation | Not addressed | Handles permission denial | Presents an ordered degradation ladder covering low power, screen off, no network, approximate permission, denied permission and thermal throttling, with what is dropped at each step |
| Updates | Not addressed | Re-downloads the pack | Compares full re-download, conditional per tile and a server diff manifest, and names the version retention window as the constraint the third one introduces |
| Measurement | "We would test it" | Frame rate and crash rate | Offline route completion under injected network loss, blank tile frame share during gestures, and estimate to actual size ratio as a consent metric |

What each level added:

- **Mid to senior.** The design separates the thing the user asked to keep from the thing the cache happens to hold, and stops treating a gesture as a network problem.
- **Senior to staff.** The candidate uses the pyramid arithmetic to make the format choice obvious, moves the power conversation to where the power actually goes, and designs for the environment refusing to cooperate rather than for the happy path.

---
## Case study 4: Video client

The three clients before this one spent bytes to render something the user had already asked for. A video client spends bytes on the user's behalf, in advance, on a connection they may be paying for by the gigabyte, for content they may abandon after eight seconds. That is what makes it a different problem rather than a bigger one.

### The prompt (video client)

"Design the mobile client for our video app: playback starts fast, does not stall, does not burn the user's data plan, and an episode can be downloaded for a flight."

### Questions that fork the design (video client)

| Question worth asking | If A the design becomes | If B it becomes |
|---|---|---|
| Live or on demand? | On demand: the whole quality ladder exists before playback starts, so you can prefetch, download and buffer as far ahead as you like | Live: the buffer ceiling is your latency requirement, you cannot buffer ahead of the encoder, and segment durations shrink, which changes every number below |
| Is content protected by digital rights management? | No: segments are ordinary cacheable files and an offline download is a file copy | Yes: license acquisition sits on the startup critical path, offline licenses carry their own expiry, and key handling varies by device in ways you will discover in the field |
| Long form player or short form vertical feed? | Long form: startup happens once per session and the bitrate controller has minutes to converge | Short form: a new stream every few seconds, so startup is the entire design and prefetching the next items dominates the byte budget |
| May we prefetch over cellular? | Yes, within a cap: you need a byte budget and per network accounting | Wi-Fi only: prefetch becomes a scheduled background job rather than a scroll time decision, and the cellular experience is entirely reactive |
| How many codecs and containers must we support? | One: a single ladder and a high edge cache hit rate | Several: the ladder multiplies, edge cache hit rate falls because requests split across variants, and device capability detection becomes a real component you must keep current |
| What stall rate is acceptable? | About 1 percent of playback time: a conservative ladder and generous buffers | Near zero: you buy it with either lower average quality or higher startup latency, and you must say which one you are spending |
| Must a download finish while the app is in the background, and survive a reboot? | No: downloads are foreground only and the design is much smaller | Yes: a durable job store, resumable ranges, an OS download facility or a foreground service, and license lifetime management across days |

!!! tip "Say this out loud in the room"

    "Before I draw anything I want to know whether this is short form or long form, because in short form the startup path is the product and in long form it is a one time cost. They are not the same design."

### Requirements (video client)

Functional:

- Start playback from a tap on a title, with the first frame visible inside a stated budget.
- Adapt quality to the available throughput without the user noticing the mechanism.
- Respect a per network quality and prefetch policy the user can change.
- Download a title for offline playback, resumable, surviving app termination and reboot.
- Play a downloaded title with no network at all, including its licence.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Quality ladder | 240p at 0.3, 360p at 0.7, 480p at 1.2, 720p at 2.5, 1080p at 4.5 Mbps | ASSUMED; a five rung ladder with roughly a factor of 1.8 between rungs, which is a common shape. Ask, because the ladder is usually already decided |
| Segment duration | 4 s | ASSUMED; the trade is startup latency against request overhead and encoding efficiency |
| Tap to first frame | under 1.0 s at p75 | ASSUMED; it is roughly where a person starts wondering whether the tap registered |
| Rebuffer ratio | under 0.5 percent of playback time | ASSUMED; state it as a ratio, not a count, because a count punishes long sessions |
| Buffer target, steady state | 30 s | DERIVED below from an assumed outage distribution |
| Cellular data per hour, default | under 1.2 GB | DERIVED below |
| Offline download storage floor | 8 GB free | ASSUMED; consistent with the nearly full phone from case study 3 |
| Battery for a 2 hour viewing session | under 30 percent | ASSUMED; a film should not need a charger |

### Estimation that eliminates (video client)

Assume the same 15.4 Wh battery and the same 10 s radio tail at 800 mW as case study 2, and the same 300 ms round trip bad network as case study 1.

| Calculation | Result | What it rules out |
|---|---|---|
| 4 s segment at 4.5 Mbps | 2.25 MB per segment | Rules out nothing yet; it is the input to the next two lines |
| 2.25 MB at an available 3 Mbps | 6 s to fetch the first 1080p segment | Rules out starting at the top rung. You cannot meet a 1 s startup budget with a 6 s first fetch |
| 4 s segment at 0.7 Mbps, fetched at 3 Mbps | 0.35 MB, 0.93 s | Rules out a 4 s first segment even at a low rung: 0.93 s consumes the entire startup budget on its own |
| A 1 s first segment at 0.7 Mbps, fetched at 3 Mbps | 87.5 KB, 233 ms | Rules out uniform segment duration. The first segment should be short even if the rest are not |
| Startup budget: manifest 150 ms, licence 200 ms, first segment 350 ms, decoder init 100 ms, on a warm connection | 800 ms | Rules out sequencing the licence after the first segment. There is no room for it later |
| The same startup on a cold connection: add an assumed 250 ms for name resolution and handshake | 1,050 ms | Rules out a cold connection on the play path. Pre-warm from the details screen, before the tap |
| Assumed cellular outage distribution on a train: median 4 s, p95 25 s | A 10 s buffer survives the median and not the p95 | Rules out a 10 s buffer target on mobile, which is a common desktop default |
| A 30 s buffer against the same distribution | Survives p95 | Rules out both a small buffer and an unbounded one; see the next line for the ceiling |
| Buffering 120 s ahead at 4.5 Mbps, with an assumed 25 percent of sessions abandoned in the first 2 minutes | 67 MB fetched, of which a quarter is wasted on average | Rules out "buffer as much as the network allows" on a metered connection |
| 90 minutes at 4.5 Mbps | 3.04 GB for one film | Rules out defaulting to the top rung on cellular. Three films exhaust an assumed 10 GB monthly plan |
| 60 minutes at 2.5 Mbps | 1.13 GB per hour | Rules out a 1.2 GB per hour cellular budget at any rung above 720p, which is exactly why the requirement is set there |
| 45 minute episode at 2.5 Mbps | 843 MB | Rules out downloading a season at the top rung by default: nine episodes fills the 8 GB storage floor |
| Continuous streaming: assumed 800 mW screen, 200 mW hardware decoder, 800 mW radio held active | 1.8 W, so 8.6 hours of battery | Rules out radio power as the top lever, and rules in the next line as the way to attack it |
| Burst fetch: 30 s of 2.5 Mbps content is 9.4 MB, fetched at 20 Mbps in 3.75 s, plus a 10 s tail, so the radio is active 13.75 s in every 30 | Radio averages 0.37 W instead of 0.8 W, total 1.37 W, so 11.2 hours | Rules out continuous low rate streaming as a transfer strategy. Burst and sleep buys roughly 30 percent more session |
| 2 hour session at 1.37 W against 15.4 Wh | 2.74 Wh, which is 18 percent of the battery | Rules out anxiety about the 30 percent budget, and confirms that the screen, not the network, is what you would attack next |

!!! note "Substitute your own ladder and your own radio"

    Every power figure and every ladder rung above is a planning assumption. What transfers is the method: compute the first segment fetch time before choosing a startup rung, compute gigabytes per hour before choosing a cellular default, and compute radio duty cycle before claiming any transfer strategy is efficient.

### V0, the simplest thing that works (video client)

**Predict first.** With a fixed quality rung and a fixed buffer, what is the first complaint, and which number produces it?

??? note "Show answer"

    Stalling, and the number is the gap between the fixed rung and the slowest connections in your population. A fixed 2.5 Mbps rung stalls for every session whose sustained throughput is below 2.5 Mbps, which on mobile is a large minority. The mirror complaint arrives from the other tail: users on fast connections watching 720p on a 1080p panel and wondering why it looks soft.

```
+-------------+   +------------------+   +-----------------+
| Player      |-->| Manifest fetch   |-->| CDN             |
| surface     |   | pick fixed rung  |   | manifest        |
+-------------+   +------------------+   +-----------------+
       |                   |
       |                   v
+-------------+   +------------------+   +-----------------+
| Hardware    |<--| Segment fetcher  |<--| CDN             |
| decoder     |   | keep 2 ahead     |   | segments        |
+-------------+   +------------------+   +-----------------+
```

**Caption.** V0 has five components: a player surface, a manifest fetch that picks one fixed quality rung, a segment fetcher that keeps two segments ahead of the playhead, a content delivery network serving both, and the device's hardware decoder.

Why this is correct at the stated scale:

- One stream at a time, one user. There is no concurrency problem and no capacity problem on the device.
- Segments are ordinary cacheable objects, so the edge does the heavy lifting and the client stays simple.
- A hardware decoder means playback costs 200 mW rather than a software decode that would dominate the power budget and heat the device.
- Two segments ahead is 8 s of buffer, which is wrong, and the point of V0 is to be wrong in a way you can measure rather than wrong in a way you guessed at.

!!! warning "When not to use adaptive streaming at all"

    If your videos are short (under about 30 s), always played to completion, and always over the same network class (an onboarding clip, a product demo, a sticker animation), a single progressive file at one quality is simpler, starts faster and caches perfectly at the edge. Adaptive streaming costs you a manifest, a segment fetcher, a rung controller, per rung encoding cost and a much harder debugging story.

    The measurement that justifies it is a rebuffer ratio above your threshold at a fixed rung, or a throughput distribution in your population wide enough that no single rung serves both ends.

### Breaking point, then V1 (video client)

**Breaking point one: a fixed rung stalls at one end of the population and wastes at the other.**

What fails. Assume a median mobile throughput of 3 Mbps with a tenth percentile near 0.8 Mbps. A fixed 2.5 Mbps rung cannot be sustained by the bottom fifth of sessions, so those sessions stall repeatedly. Lower the rung to serve them and the top of the distribution watches 240p on a 1080p panel.

How you observe it. Publish rebuffer ratio (stalled seconds divided by played seconds) and average delivered bitrate, both segmented by network class and by throughput decile. A rebuffer ratio that is concentrated in the bottom deciles is this failure. A rebuffer ratio spread evenly across deciles is a different problem, usually the buffer target rather than the rung.

V1 makes quality a control loop:

```
+-------------+   +------------------+   +-----------------+
| Player      |-->| Rung controller  |-->| Segment fetcher |
| surface     |   | throughput and   |   | fetch next rung |
+-------------+   | buffer occupancy |   +-----------------+
       |          +------------------+            |
       |                   |                      v
+-------------+   +------------------+   +-----------------+
| Hardware    |<--| Buffer, 30 s     |<--| CDN             |
| decoder     |   | target, on disk  |   | segments        |
+-------------+   +------------------+   +-----------------+
```

**Caption.** V1 adds a rung controller fed by two independent signals, a measured throughput estimate and the current buffer occupancy, and raises the buffer target from 8 s to 30 s so a p95 outage does not become a stall.

| V1 change | Justified by | Cost you accept |
|---|---|---|
| Rung controller | A fixed rung cannot serve a distribution with a five to one spread | A control loop that can oscillate, which is breaking point two |
| 30 s buffer target | A 10 s buffer does not survive a 25 s p95 outage | 16.9 MB held at the top rung, and more bytes wasted when a session is abandoned |
| Disk backed buffer | 30 s at 4.5 Mbps is too large to hold comfortably in the process memory budget from case study 1 | A file lifecycle to manage, and cleanup on abnormal termination |
| Short first segment | 0.93 s for a 4 s first segment eats the whole startup budget | The encoding pipeline must produce a differently sized first segment, which is a server side ask you should name |
| Pre-warmed connection from the details screen | 250 ms of handshake on the play path | A connection opened for a play that may never happen, which is a small waste to remove a large latency |

!!! warning "When not to raise the buffer target"

    On an unmetered, stable connection with no abandonment (a set top style session), a larger buffer is nearly free and you should take it. On mobile it is not free: every buffered second on a cellular link is a byte the user may pay for and may never watch. The measurement that justifies a given target is your own outage duration distribution, specifically the percentile you have decided to survive. Picking 30 s because another product picked 30 s is not an answer.

**Breaking point two: the controller oscillates, and the user notices the mechanism.**

What fails. A controller driven only by a throughput estimate reacts to every fluctuation. It steps up, the higher rung consumes more than the link provides, the estimate falls, it steps down, the buffer recovers, the estimate rises, it steps up again. The viewer sees quality flicker every few seconds, which reads as a broken player even when no stall occurs. The same controller is at its worst during startup, when it has no throughput history at all.

How you observe it. Count rung switches per minute of playback and publish the distribution, not the mean. Then correlate the switch rate with abandonment. A session with eight switches per minute and no stalls can be worse for the viewer than a session with one stall.

V2 separates the phases and adds the byte budget:

```
+-------------+   +------------------+   +-----------------+
| Player      |-->| Startup phase    |-->| Steady phase    |
| surface     |   | low rung, short  |   | buffer led with |
+-------------+   | segment, ramp up |   | hysteresis band |
       |          +------------------+   +-----------------+
       |                   |                      |
+-------------+   +------------------+   +-----------------+
| Download    |   | Byte budget per  |   | Prefetch        |
| manager     |-->| network class    |<--| scheduler       |
| durable job |   | and per session  |   | next items      |
+-------------+   +------------------+   +-----------------+
```

**Caption.** V2 splits the controller into a startup phase and a steady phase with different rules, and adds a shared byte budget that both the prefetch scheduler and the offline download manager must draw from, so the two cannot separately decide to spend the user's data plan.

### Deep dive one: choosing the next rung without the viewer noticing

This dive is chosen because the control loop is where a video client is genuinely different from every other client on this page. The earlier dives were about memory, durability, key distribution, pack sizing and power. This one is about a feedback system whose output the user can see.

**Two signals, two failure modes.**

| Signal | What it tells you | How it fails alone |
|---|---|---|
| Throughput estimate | What the link recently delivered | Noisy, and it is a lagging measure of a quantity that changes faster than you can measure it. Optimistic after a burst from an edge cache, pessimistic after one slow segment |
| Buffer occupancy | How much time you have before a stall | Slow to react at startup, when the buffer is empty by definition and says "panic" regardless of the link |

Neither is sufficient. The workable design uses buffer occupancy as the primary control in steady state, because it is the quantity you actually care about, and uses the throughput estimate as a ceiling so you never request a rung the link demonstrably cannot carry.

**The steady state rule, stated as a mapping.**

| Buffer occupancy | Action | Reason |
|---|---|---|
| Below 10 s | Step down one rung immediately, ignore hysteresis | A stall is worse than any quality drop, and this is the only place where reacting fast is right |
| 10 to 20 s | Hold the current rung | The dead band. Doing nothing is a decision, and it is the one that stops oscillation |
| 20 to 30 s | Step up one rung only if the throughput estimate supports the higher rung with an assumed 1.4 times margin, and only if the last switch was more than an assumed 15 s ago | The margin and the dwell time are the two knobs that control the switch rate |
| Above 30 s | Hold the rung and stop fetching until occupancy falls below the target | This is what turns continuous streaming into burst and sleep, worth roughly 30 percent of session battery from the estimation table |

**Why the dead band matters, with the arithmetic.** Suppose the link delivers exactly 2.9 Mbps and the ladder has rungs at 2.5 and 4.5. Without a dead band, the controller sits at 2.5, sees spare capacity, steps to 4.5, drains the buffer at 1.6 Mbps of deficit, and steps back down when the buffer falls.

At a 30 s buffer, the round trip takes about 19 s down and however long it takes to refill. That is roughly three switches per minute, indefinitely, on a link that never changed. The dead band and the 1.4 times margin both exist to make the controller refuse a rung it cannot hold.

**The startup phase, which needs different rules.**

1. Start at a rung whose first segment fetch fits the startup budget. From the estimation table, at 3 Mbps that is 0.7 Mbps with a 1 s first segment, giving 233 ms.
2. Use the previous session's throughput as a prior if it is recent and on the same network class, but cap how much it may raise the starting rung. A prior from home Wi-Fi must not choose the starting rung on a train.
3. Fetch the licence in parallel with the manifest, not after it. It is 200 ms of the 800 ms budget and it has no dependency on the manifest contents in the common case.
4. Ramp up only once the buffer has reached the dead band. Ramping while the buffer is filling is how you produce a stall in the first ten seconds, which is the single most abandonment inducing moment in a session.
5. Render the first frame as soon as it decodes, before the buffer target is met. Startup is measured to first frame, not to steady state.

**How you evaluate a change to any of this.** This is where candidates stop and where the interesting part starts.

| Metric | What it catches | Why it is not enough alone |
|---|---|---|
| Rebuffer ratio | Stalls | A controller that always picks the lowest rung has a perfect rebuffer ratio and a terrible product |
| Average delivered bitrate | Quality | A controller that always picks the top rung looks excellent here and stalls constantly |
| Rung switches per minute | Oscillation | A controller that never switches scores perfectly and cannot adapt |
| Startup time to first frame | The first impression | Improving it by starting lower trades against average bitrate |
| Time spent at the highest rung the link could sustain | Efficiency, and it is the one that is hard to game | Requires you to estimate what the link could have sustained, which is itself uncertain |

Publish all five, and refuse to accept a change that improves one by degrading another without an explicit product decision about the trade. That refusal is the senior behaviour here.

**A worked comparison, so the trade is concrete.** Three controllers on the same assumed session (a 30 minute view on a link that delivers 3 Mbps for 20 minutes, then 1 Mbps for 5, then 3 Mbps again):

| Controller | Rebuffer seconds | Mean rung | Switches |
|---|---|---|---|
| Fixed 2.5 Mbps | 5 minutes at 1 Mbps cannot sustain 2.5, so it stalls repeatedly: roughly 180 s | 2.5 Mbps | 0 |
| Throughput only, no dead band | Near 0, because it reacts fast | About 2.4 Mbps | Roughly 3 per minute, so about 90 |
| Buffer led with a dead band and a 30 s target | Near 0: the 30 s buffer absorbs the transition, and the step down happens once | About 2.3 Mbps | About 2 for the whole session |

The third gives up a small amount of mean bitrate and removes 88 switches. That trade is almost always correct, and being able to state it as a trade rather than a win is the point.

!!! warning "When not to use buffer led control"

    In low latency live streaming the buffer target is your latency requirement, often a few seconds, so there is no dead band to work with and the controller must be throughput led with a much faster reaction. In short form feeds the session is shorter than the controller's convergence time, so the rung is effectively chosen once at startup and the design effort belongs in prefetch instead.

    The measurement that tells you which regime you are in is median session duration against your controller's convergence time. If the session is shorter, the controller barely matters.

### Deep dive two: spending bytes the viewer may never watch

The second dive leaves the control loop entirely. It is about a decision no other client on this page has to make in the same form: how many bytes to fetch for content the user has not asked for, on a connection they may be paying for, when the probability they watch it is well under one.

**The three ways a video client spends speculative bytes.**

| Mechanism | Bytes at risk | Probability they are used |
|---|---|---|
| Buffer ahead within the playing item | Buffer target times the current rung, so 16.9 MB at 30 s and 4.5 Mbps | High, but not one: an abandoned session wastes the whole buffer |
| Prefetch of the next item in a browse surface | First few seconds of each candidate | Low, and it is a product of how many candidates you prefetch |
| Offline download | The whole title, so 843 MB for an episode | High, because the user asked, but not one: plans change and licences expire |

**The accounting that makes this decidable.** Define wasted bytes as bytes fetched and never rendered, and publish it as a ratio of total bytes fetched, segmented by network class. Without that number every prefetch argument is an opinion.

| Prefetch policy | Bytes fetched per browse session | Wasted ratio | When it is right |
|---|---|---|---|
| None | 0 | 0 percent | Metered connection, or a browse surface where the user picks deliberately from a long list |
| First 2 s of the top candidate only | An assumed 1 candidate times 2 s at 0.7 Mbps, so 175 KB | Depends only on how often the top candidate is chosen | Almost always defensible, because it is small |
| First 2 s of the top 5 candidates | 875 KB | At least 80 percent, because at most one is played | Unmetered connections with a strong ranking signal |
| First 10 s of the top 5 candidates | 4.4 MB | At least 80 percent | Rarely, and never on cellular |

The pattern is that prefetch cost scales with candidates times seconds, and the wasted ratio is bounded below by (candidates minus 1) divided by candidates. That bound is not something you can engineer away; it is a property of prefetching more than one thing.

**Network class is a first class input, not a checkbox.**

| Class | Prefetch | Buffer target | Default rung | Reason |
|---|---|---|---|---|
| Unmetered, fast | Aggressive | 30 s or more | Highest the panel can show | Bytes are nearly free and the only cost is battery |
| Unmetered, slow | Conservative | 30 s | Whatever the controller picks | The buffer is protecting against the link, not the plan |
| Metered, fast | Top candidate only | 30 s, but see below | Capped at 720p by default | 1.13 GB per hour against 3.04 GB per hour is the whole argument |
| Metered, and the user has set a saver preference | None | 15 s | Capped at 480p | The user has told you their preference and overriding it is a trust failure |
| Roaming | None | 15 s | Lowest usable | Roaming is where a single session can cost real money |

Note the buffer target on a metered fast connection. A large buffer is good for stalls and bad for abandonment waste. At 25 percent abandonment in the first two minutes, a 30 s buffer wastes on average 25 percent of 30 s of content, which at 2.5 Mbps is 2.3 MB per abandoned session. That is small enough to accept, and computing it is what lets you accept it rather than argue about it.

**Radio duty cycle, which is the reason burst and sleep wins.** From the estimation table, fetching 30 s of content in 3.75 s and then idling leaves the radio active 13.75 s in every 30, against 30 in every 30 for a continuous trickle. That is 0.37 W against 0.8 W, and it is worth roughly 2.6 hours of extra session time.

The design rule that follows: fetch in bursts up to the buffer target, then stop entirely, rather than pacing fetches to match the playback rate. Pacing feels elegant and is the wrong answer on a phone.

**Offline downloads, which are prefetch with the probability set close to one.**

| Design decision | Choice | Why |
|---|---|---|
| Quality selection | Ask, and show the size for each option before the tap | 843 MB against 3.04 GB is a decision only the user can make, and they need the number to make it |
| Storage check | Refuse to start if the estimate exceeds free space minus a margin | Same rule as the maps pack in case study 3, and for the same reason |
| Resumability | Range requests per segment, with a persisted completion set | A segmented format gives you resumability almost for free; use it rather than inventing chunking |
| Ordering | In playback order | A partial download is watchable from the start, which a scattered one is not |
| Background execution | A durable job store plus the platform's background transfer facility | The app will be terminated during a 843 MB download, and a job that cannot survive that is not a download manager |
| Licence | Acquire and persist the offline licence at download time, not at play time | The user will press play on a plane. A licence fetched at play time needs a network they do not have |
| Licence expiry | Show the expiry in the interface, and refresh it opportunistically while online | A download that silently expired the day before a flight is the worst failure this feature has |
| Cleanup | Delete on watch completion if the user opted in, and always on sign out | Storage the user cannot find and cannot free generates uninstalls |

**The failure that catches everyone.** A user downloads five episodes on Wi-Fi, boards a plane, and playback fails because the licences were acquired with a 24 hour window and the flight is on day three. Nothing about the bytes was wrong. The design fix is to treat licence lifetime as part of the download artefact: show it, refresh it whenever the app is online and a download exists, and warn before it lapses.

!!! warning "When not to prefetch at all"

    If your browse surface has more than a handful of plausible next items, or your users are predominantly on metered connections, prefetch is a cost with a bounded upside and an unbounded downside in trust. It costs bytes the user pays for, battery, edge cache pressure and a wasted byte metric you must then defend.

    The measurement that justifies it is a startup time improvement large enough to move completion rate in an experiment, measured against the wasted byte ratio it produced. If you cannot show both numbers, do not ship the prefetch.

### Failure modes and evaluation (video client)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Rebuffer storm on failover | An edge fails, every client's throughput estimate collapses at once, and they all step down together | Per client jittered retry to an alternate edge, and a controller that treats a sudden collapse as a possible failover rather than a link change |
| Licence server outage | Playback stops even though the bytes are cached locally, which reads to the user as "the app is broken" | Persist licences with a lifetime longer than a plausible outage, and degrade to already licensed content rather than failing the whole app |
| Thundering herd at a scheduled release | Everyone presses play in the same minute, and the edge sees a cold cache for a large catalogue item | Stagger availability slightly, pre-warm the edge, and let the client's startup rung be conservative during the first minutes of a release |
| Metastable oscillation | The controller and the network settle into a cycle that neither escapes | The dead band and the minimum dwell time are exactly the damping that prevents this, and they must be tested against a link that sits between two rungs |
| Storage exhausted mid download | A 843 MB download fills the device | Check free space against the estimate, verify after completion, and never leave a partial file that the player believes is complete |
| Decoder resource exhaustion | Several player instances exist (a feed with autoplay plus a foreground player) and the device runs out of decoder sessions | A player pool with a hard cap, and explicit release on detach; this is the same discipline as the bitmap cache in case study 1 |
| Thermal throttling | A long session in a warm room drops the frame rate and the decoder struggles | Cap the rung by sustained thermal state, and prefer a lower resolution over a dropped frame rate |
| Clock skew | An offline licence appears expired because the device clock is wrong | Use a monotonic device counter alongside wall clock where the licence system allows, and revalidate rather than hard failing when the clock looks implausible |
| Version skew | A new ladder rung or codec ships server side before most installs can decode it | Client advertises its decode capability, the manifest is filtered server side, and an unknown rung is skipped rather than attempted |

Service level indicators:

- Rebuffer ratio (stalled seconds divided by played seconds), segmented by network class and throughput decile.
- Startup time to first frame at p75 and p95, split by warm and cold connection.
- Rung switches per minute, published as a distribution.
- Mean delivered bitrate, and time spent at the highest sustainable rung.
- Wasted byte ratio: bytes fetched and never rendered, segmented by mechanism (buffer, prefetch, download).
- Download success rate, resume rate, and licence expiry incidents per 1,000 downloads.
- Battery draw per playback hour, and radio active duty cycle.
- Playback failure rate by cause, with licence failures separated from network failures, because they need different fixes.

Alert on: rebuffer ratio crossing its threshold in any network class, not in aggregate; licence failure rate above zero, since it is a total failure for the affected user; and wasted byte ratio on metered connections crossing its budget after a release, which is how a prefetch regression is caught before it reaches a billing complaint.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Tap to first frame under 1.0 s at p75 | Yes at 800 ms with a warm connection, a short first segment and a parallel licence fetch; no on a cold connection, which is why the connection is pre-warmed |
| Rebuffer ratio under 0.5 percent | Yes with a 30 s buffer and buffer led control, given the assumed outage distribution; the honest answer is that it depends on that distribution and you should measure your own |
| Under 1.2 GB per hour on cellular | Yes at a 720p cap, which delivers 1.13 GB per hour; no at 1080p, which is 2.03 GB per hour |
| Resumable download surviving termination and reboot | Yes through a durable job store, per segment range requests and a persisted completion set |
| Offline playback with no network | Yes, provided the licence was acquired at download time and has not expired, which is the failure most designs forget |
| Under 30 percent battery for 2 hours | Yes at roughly 18 percent with burst and sleep fetching; about 24 percent with a continuous trickle, which is why the fetch strategy is part of the answer |

### Where this answer is a simplification (video client)

- **Segmented adaptive delivery is specified, and the specifications are worth reading.** [RFC 8216, HTTP Live Streaming](https://www.rfc-editor.org/rfc/rfc8216) (Informational, August 2017; checked 1 August 2026) is the readable one. The other major family is MPEG-DASH, standardised as ISO/IEC 23009-1; cite it by number rather than by a summary, and check which edition you mean.
- **Range requests and conditional requests are how resumable downloads work.** See [RFC 9110, HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110) (June 2022; checked 1 August 2026), and [RFC 9111, HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111) (June 2022; checked 1 August 2026) for why segment URLs must be stable if you want the edge to help you.
- **Transport choice affects stalls more than most client code does.** Loss on a mobile link and a network handoff mid segment are transport problems: see [RFC 9000, QUIC](https://www.rfc-editor.org/rfc/rfc9000) (May 2021) and [RFC 9114, HTTP/3](https://www.rfc-editor.org/rfc/rfc9114) (June 2022), both checked 1 August 2026.
- **The rung controller here is deliberately simple.** Published work on rate adaptation covers buffer based control, model predictive approaches and learned controllers, and the comparisons are sensitive to the trace set used. Treat any single published result as evidence about a trace set, not about your users.
- **What this page skips.** Encoding ladder design (per title and per scene encoding change every number in the estimation table), subtitle and caption delivery and their accessibility requirements, audio track switching, live streaming with its entirely different buffer economics, and content protection robustness levels that vary by device.
- **The organisational reality.** The client controller, the encoding ladder and the delivery network are usually owned by three different teams, and each can improve its own metric while degrading the product. The only durable fix is a shared, published quality of experience metric set that all three are measured against, which is why the evaluation table above lists five metrics and insists none of them is sufficient alone.

### Level delta (video client)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Draws a player, a manifest and a delivery network | Asks live against on demand, and short form against long form, before drawing | Asks the same, then computes a 6 s first segment at the top rung and uses it to rule out the obvious startup design in the first five minutes |
| Startup | "It loads the video" | Picks a low starting rung | Itemises an 800 ms budget across manifest, licence, first segment and decoder init, parallelises the licence, shortens the first segment, and pre-warms the connection from the previous screen |
| Adaptation | "It adapts to bandwidth" | Uses a throughput estimate | Uses buffer occupancy as the primary signal with throughput as a ceiling, and derives the dead band from a worked oscillation example on a link between two rungs |
| Buffer | "Buffer a bit ahead" | Sets a 30 s target | Derives the target from an outage duration distribution, then bounds it from above using abandonment rate and wasted bytes |
| Data cost | "We reduce quality on cellular" | Caps the rung on cellular | Computes 3.04 GB against 1.13 GB per hour, sets the cap from the stated budget, and puts prefetch and download on one shared byte budget so they cannot each spend it |
| Prefetch | Not mentioned, or "prefetch the next one" | Prefetches a few seconds of the top candidate | Shows the wasted ratio is bounded below by (candidates minus 1) over candidates, and refuses to ship prefetch without both a startup gain and a waste number |
| Battery | "Video uses battery" | Notes the screen dominates | Derives burst and sleep from a 10 s radio tail, shows 1.37 W against 1.8 W, and converts it into 2.6 hours of session |
| Offline | "Download the file" | Resumable background download | Treats the licence as part of the artefact, shows expiry in the interface, refreshes it opportunistically, and names the day three flight failure before the interviewer does |
| Evaluation | "We measure buffering" | Rebuffer ratio and startup time | Publishes five metrics, states that each is gameable alone, and refuses a change that trades one for another without an explicit product decision |

What each level added:

- **Mid to senior.** The design gains a control loop and a network policy, and the quality question stops being "what bitrate" and becomes "what does the link support".
- **Senior to staff.** The candidate treats adaptation as a feedback system with a damping requirement, puts every speculative byte on one accounted budget, and designs the measurement set so that no single team can win their metric at the product's expense.

---

Four device assumptions run through every case study below. They are placeholders, not facts. Say that out loud in the room, then offer to re-run the arithmetic with the interviewer's numbers. A number you label as a placeholder and offer to measure reads as engineering; the same number stated as fact reads as recall.

| Assumption | Value | Why it is reasonable |
|---|---|---|
| Reference mid-tier device | 6 CPU cores, 4 GB RAM, 1080 by 2400 display, 60 Hz | This is where cheap-to-midrange phones cluster. Read your own fleet's median from your crash and vitals reporting on day one |
| Battery cell | 4,000 mAh at 3.85 V, so 15.4 Wh | Milliamp-hours alone cannot be turned into watts. The watt-hour figure is what every battery calculation on this page divides into |
| Cellular round trip | 120 ms at p50, 600 ms at p95 | A working mobile connection with ordinary congestion. Your real user monitoring will disagree, and that disagreement is the point |
| Cellular downlink throughput | 3 Mbit per second, so 375 KB per second | A conservative real-world figure rather than a marketing peak |
| Radio high-power tail | about 10 seconds at about 0.7 W after a transmission, so about 7 J per isolated send | The exact numbers vary by radio and carrier. The shape is what the designs depend on: a fixed energy cost per transmission that does not scale with bytes |
| HTTPS request overhead | about 600 bytes | Request and response headers plus record framing on an already warm connection. Compressing a payload below this figure buys nothing |
| App heap ceiling before the OS kills the process | 256 MB | A placeholder for a mid-tier device. Read the real value from the platform at runtime rather than hard-coding it |

The eight deep dives on this page deliberately do not repeat each other. Two are about a state machine under concurrency, two about an energy and latency budget, two about a public contract, and two about durability on a device that dies without warning. Where two case studies touch the same mechanism, for example a write-ahead log, they reach opposite conclusions about durability settings, and the page says why.

---

## Case study 5: Ride hailing client

This is the first prompt on the page where the client holds state that money depends on, and where two humans are looking at two phones that must agree. That is the whole difficulty. The maps, the routing and the dispatch algorithm are not your problem in a client round, and saying so early buys you the time to do the part that is.

### The prompt (ride hailing client)

"Design the mobile apps for a ride hailing service: the rider requests a trip, and the driver runs it from accept through to drop off."

### Questions that fork the design (ride hailing client)

| Question worth asking | If the answer is A the design becomes | If the answer is B the design becomes |
|---|---|---|
| One app with a role switch, or two separate apps? | One app: a shared trip state layer and one release train, but every driver-only permission is declared for riders too, which costs you install conversion and a store review argument | Two apps: the driver app is tuned for an eight hour foreground session with the screen on, and the rider app is tuned for cold start, and they can ship on different cadences |
| Is the server the only authority on trip state? | Yes: the client is a projection, every user action is a proposal, and the hard work is reconciling an optimistic local view, which is deep dive one | No, the client may commit locally: you now own conflict resolution between two devices and a server, which is a research problem you should refuse in an interview and say why |
| May the driver app hold location in the background all shift? | Yes, with a foreground service or always-permission: the pipeline is a long-lived component with a durable buffer and an OS-visible notification | No, foreground only: you must design gap detection and backfill, and tell the product owner that arrival estimates degrade whenever the driver checks a message |
| How fresh must the driver's position look to the rider? | One update per second, animated: high downlink, client-side interpolation along the road, and the map becomes your frame budget problem | One update every five seconds: cheap, but you must interpolate anyway or the pin teleports and users read teleporting as a broken app |
| Does the driver work where the network does not? | Rarely: you can treat offline as an error state with a retry banner | Routinely, in tunnels and rural gaps: the local trip log becomes durable storage of record, trip completion must commit offline, and fares are provisional until acknowledged |
| Is payment authorised at request or at drop off? | At request: the failure mode is a stranded authorisation on a cancelled trip, which is a refund problem | At drop off: the failure mode is a completed trip with no payment, so the client must guarantee that the completion event eventually reaches the server, which changes the durability requirement |
| How many app versions must the server tolerate at once? | Two: you can change the state machine in a release | Twenty over three years: transitions must be additive only, unknown states must render as something safe, and forced upgrade becomes a design feature rather than an emergency lever |
| Does the driver's phone measure the billable distance? | No, the server derives it from received points: tampering is a server problem and the client stays simple | Yes: you have a fraud surface, and you need monotonic timestamps, signed batches and a server-side plausibility check, which is a different case study |

### Requirements (ride hailing client)

Functional, rider app:

- Request a trip for a pickup and destination, and see matching progress within one screen transition.
- See the assigned driver, the vehicle, and a position that moves plausibly between updates.
- Cancel, and see the cancellation policy and any fee before confirming.
- Resume into the correct trip state after the app is killed and relaunched.

Functional, driver app:

- Go online and offline, and see why the app believes it is online.
- Receive an offer with a visible countdown, accept or decline, and never see an offer that has already expired.
- Move through arrive, start and end, including when the network is absent.
- Emit location while online, buffer it when offline, and lose no point that belongs to a trip.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Drivers online at peak in one city | 8,000 | ASSUMED; a mid-size market. Ask, because the number changes nothing on the client and everything on the ingest tier |
| Concurrent trips at peak in that city | 1,500 | ASSUMED; roughly one in five online drivers carrying a rider |
| Location sample rate while on a trip | 1 per second | ASSUMED; matches what turn-by-turn navigation needs to look smooth |
| Location sample rate while idle and online | 1 per 4 seconds | ASSUMED; dispatch needs a driver located to within a block, not a lane |
| Time from tap to a visible state change | under 400 ms | ASSUMED; the threshold at which a tap stops feeling connected to the screen |
| Offer countdown | 15 seconds | GIVEN by the product |
| Driver position staleness on the rider's map | under 6 seconds at p95 | ASSUMED; derived against the upload cadence below |
| Client and server trip state divergence at trip end | zero | GIVEN; this is the requirement that money depends on |
| Time to converge after reconnecting | under 10 seconds | ASSUMED |
| Driver battery drain while online | under 12 percent per hour | ASSUMED; an eight hour shift needs a car charger anyway, but above this figure a charger cannot keep up |
| Durable local trip log retention | 72 hours | ASSUMED; covers a weekend-long backend incident |
| Cellular bytes per driver hour | under 3 MB | ASSUMED; drivers pay for their own data and notice |

### Estimation that eliminates (ride hailing client)

| Calculation | Result | What it rules out |
|---|---|---|
| 1 point per second for a 20 minute trip | 1,200 points per trip | Rules out one request per point at the API level: 1,200 requests per trip per driver, and 1,500 concurrent trips would be 1,500 requests per second of pure per-point chatter from one city |
| A packed point: latitude and longitude deltas 2 bytes each, time delta 1 byte, speed, bearing and accuracy quantised to 1 byte each | about 8 bytes after the first point, 40 bytes for a self-contained point | Rules out compressing the payload as the first optimisation. A 4 second batch is 4 points, so 40 bytes of body against 600 bytes of request overhead |
| On-trip upload every 4 seconds: 900 uploads per hour times (600 bytes overhead plus 160 bytes body) | 684 KB per driver hour | Rules out bandwidth as the binding constraint. You are at 23 percent of the 3 MB budget, so the design is not decided by bytes |
| Same traffic sent one point per second: 3,600 uploads times 640 bytes | 2.3 MB per driver hour | Rules out per-point upload on request count and server load, but note that it does not breach the byte budget either. The reason to batch is not bytes |
| Radio energy at 1 send per second against a 10 second high-power tail | the radio never returns to idle, so 0.7 W continuously, which is 4.5 percent of the battery per hour | Rules out the belief that batching from 1 second to 4 seconds saves battery. Both cadences pin the radio high. This single line is worth writing on the board |
| Radio energy at 1 send per 30 seconds: 120 sends times 7 J | 840 J, so 0.23 Wh, so 1.5 percent per hour | Rules out short batching for the idle state, and proves the energy win only arrives once the gap exceeds the radio tail. That threshold is about 15 seconds, and it is the real design boundary |
| On-trip power: screen and map 0.8 W plus high accuracy positioning 0.35 W plus radio 0.7 W (each ASSUMED, measure yours) | 1.85 W, so 12.0 percent per hour | Rules out any second continuous consumer in the driver app while on a trip. The budget is exactly spent |
| Idle power: screen 0.8 W plus balanced positioning 0.1 W plus radio 0.23 W | 1.13 W, so 7.3 percent per hour | Rules out treating idle and on-trip as one mode. Half a shift idle and half on trip averages 9.7 percent per hour, which is the only way the 12 percent figure is met |
| 8,000 drivers at 1 per 4 seconds plus 1,500 trips at 1 per second | 2,000 plus 1,500, so 3,500 points per second per city | Rules out the client being the scaling worry, and rules out sampling idle drivers at 1 Hz, which would be 8,000 points per second for positions no rider is watching |
| 72 hours times 3,600 points per hour times 40 bytes | 10.4 MB | Rules out an in-memory-only buffer, since the buffer must survive process death, and rules out worrying about disk, since 10 MB is nothing |
| 400 ms tap-to-state budget against a 600 ms p95 round trip | the budget is smaller than one network round trip | Rules out waiting for the server before changing the screen. Optimistic local transitions are forced by arithmetic, not by taste, and that is what makes deep dive one necessary |
| 15 second offer countdown minus two p95 round trips | about 13.8 seconds of actual human decision time | Rules out fetching trip detail after the offer arrives. The offer payload must be self-sufficient |

### V0, the simplest thing that works (ride hailing client)

**Predict first: at how many concurrent trips does polling for trip state stop being the right answer?**

??? note "Show answer"

    Later than most candidates say, and for a reason that is not load. At 1,500 concurrent trips polling every 2 seconds you generate 750 requests per second, which one modest service handles. What breaks first is the driver's battery and the rider's perception.

    Polling every 2 seconds pins the radio in its high-power state exactly as a 1 Hz stream would, so polling costs the same energy as streaming while delivering worse freshness. The trigger to move is not a request-per-second number, it is the first product requirement that needs a server-initiated event, such as an offer that must reach a driver within a second.

```
+----------+   create, poll trip state   +---------------+
| Rider    | <-------------------------> | Trip service  |
| app      |                             | holds all     |
+----------+                             | state         |
                                         |               |
+----------+   poll offers, post points  |               |
| Driver   | <-------------------------> |               |
| app      |                             +---------------+
+----------+
```

**Caption.** V0 has three parts: a rider app that creates a trip and polls for its state, a driver app that polls for offers and posts location on a timer, and one trip service that holds every piece of state. Neither client stores anything that outlives the process.

Why this is genuinely correct at a real scale:

- One source of truth, so there is no reconciliation code and no class of divergence bugs.
- A support engineer can answer "what state is this trip in" by reading one row.
- Every client screen is a pure function of the last response, which is the easiest thing in the world to test.
- Version skew is trivial, because the client holds no state whose shape can drift.

!!! warning "When not to move off polling and server-only state"

    Below roughly 200 concurrent trips in a city with reliable coverage, polling every 2 seconds is the correct design and a push channel is over-engineering. The measurement that justifies moving is the distribution of tap-to-state latency at p95 against your 400 ms target, plus the count of offers that expire before the driver sees them.

    If p95 tap-to-state is already inside budget because your network is good, and offer expiry is near zero, the cheaper fix is to shorten the poll interval on the two screens that need it rather than to build a channel, an outbox and a reconciliation protocol.

### Breaking point, then V1 (ride hailing client)

**What fails first: the 400 ms tap-to-state budget, on every tap.** A round trip at p95 is 600 ms, and the state change cannot precede the response. How you observe it: instrument from the touch-up event to the first frame that renders a different state, and plot p50 and p95 by connection type. If p95 sits above 400 ms and tracks your network latency curve rather than your CPU curve, the fix is architectural, not a matter of shaving code.

**What fails second: process death eats the send queue.** The OS reclaims the driver app while it is backgrounded at a traffic light. Any points and any state proposal held in memory are gone. How you observe it: give each driver session a monotonic sequence number on every emitted point, and let the server compute the gaps. A gap rate that spikes in the minute after a backgrounding event is process death, not coverage.

**What fails third: the same tap becomes two trips.** A rider taps request, the response is slow, they tap again. How you observe it: count trips created per unique client request identifier. Any value above one is a duplicate, and duplicates in this product mean a charged cancellation.

V1 makes the client a projection with an outbox:

```
+-----------+     +------------------+     +---------------+
| Actions   | --> | Outbox, durable  | --> | Trip service  |
| propose   |     | idempotency key  |     | authority     |
+-----------+     +------------------+     +---------------+
                                                   |
+-----------+     +------------------+     +---------------+
| UI reads  | <-- | Trip state       | <-- | Push channel  |
| one model |     | machine, local   |     | server events |
+-----------+     +------------------+     +---------------+

+--------------+     +---------------------+
| Point buffer | --> | Batching uploader   |
| durable ring |     | every 4 s on a trip |
+--------------+     +---------------------+
```

**Caption.** V1 splits the client into three paths: the UI reads a single local trip state machine, user actions write idempotency-keyed proposals into a durable outbox that retries until acknowledged, and location points land in a durable ring buffer drained by a batching uploader. Server events arrive on a push channel and are the only thing allowed to move the machine into a terminal state.

What each addition buys, and what it costs:

| Addition | Buys | Costs |
|---|---|---|
| Local state machine | Tap-to-state under 40 ms, because the transition is local | A second definition of the state machine that can drift from the server's |
| Durable outbox with idempotency keys | Duplicate trips go to zero, and a killed process resumes its intent | Retry logic, key lifetime, and a server contract you now depend on |
| Durable point ring buffer | Gaps become coverage-only, not process-death-related | Disk writes on a hot path, so group commit matters |
| Push channel | Offers arrive in under a second, and the radio is not woken by polling | Connection lifecycle, reconnect storms, and a whole class of "connected but not receiving" bugs |

!!! warning "When not to add a durable outbox"

    An outbox earns its place when a user action has a financial or safety consequence and the process can die between intent and acknowledgement. Below that bar, in-memory retry with a visible error is simpler and more honest. The measurement that justifies it is the duplicate-per-key rate and the rate of actions lost to backgrounding. If both are under about one in ten thousand actions, an outbox adds a durable store, a key lifetime policy and a resume path to fix nothing you can see.

### Breaking point, then V2 (ride hailing client)

**What fails: out-of-order and stale server events reverse the trip.** A push arrives late, after a newer one, and the client moves the trip backwards from "on trip" to "arrived". How you observe it: log every applied transition with the server's own ordering token, then count transitions where the applied token is lower than the previously applied one. That count should be zero and will not be.

**What also fails: 8,000 drivers reconnect at the same instant.** A network blip or a backend deploy drops every connection in the city, and every client reconnects immediately, then requests a full state refresh. This is a thundering herd, and it is self-inflicted. How you observe it: plot connection establishment rate per second, and look for a spike whose height equals your online device count.

V2 adds three mechanisms, and each one is small:

| Mechanism | What it does | Why it is not optional |
|---|---|---|
| Server-assigned state epoch | Every server view of a trip carries a monotonically increasing epoch. The client discards any event whose epoch is not greater than the last applied | Push channels do not guarantee order, and neither does a retried request. Without an epoch you cannot tell late from new |
| Reconciliation fetch on reconnect | On every reconnect the client fetches the authoritative state for its active trip and any outbox entries still unacknowledged, then applies the epoch rule | A push channel that was down cannot tell you what you missed. Only a fetch closes the gap |
| Jittered reconnect with a full-state budget | Reconnect after a random delay drawn from a growing window, and the reconciliation fetch is one small request, not a full history | Turns a herd into a ramp. The jitter window is sized from the herd: 8,000 devices spread over 60 seconds is 133 reconnections per second, which a normal service absorbs |

!!! warning "When not to add epochs and reconciliation"

    If every state change is client-initiated and acknowledged in the same response, there is no ordering problem and an epoch is ceremony. Epochs earn their place the moment a second actor, a server or another device, can change the same object. The measurement that justifies them is the count of applied-out-of-order transitions described above. If it is zero over a week of real traffic with real network conditions, you do not have this problem yet.

### Deep dive one: the trip state machine, and the four races that actually happen

Most candidates draw a state machine and move on. The state machine is not the interesting part. The interesting part is that two devices and a server all hold a copy of it, and only one of them is right.

**The states, and who may propose what.**

| State | Rider may propose | Driver may propose | Server only |
|---|---|---|---|
| Idle | Request | Go online, go offline | |
| Requesting | Cancel | | Assign, no supply |
| Assigned | Cancel | Decline, arrived | Reassign, driver cancelled |
| Arrived | Cancel | Start | Timeout |
| On trip | | End | Force end |
| Ended, unsynced | | | |
| Completed | | | Settle |
| Cancelled | | | Settle fee |

Two rules fall out of that table and they carry the whole design:

- A client may propose a transition and render it immediately, but only into a non-terminal state. Completed and Cancelled are server-only, because money settles there.
- "Ended, unsynced" is a real state with its own screen, not an error. A driver who ends a trip in a tunnel is in a state the product must name, and hiding it behind a spinner is how you get a driver who taps end five times.

**The epoch rule, in four lines.**

- Every server state carries (trip identifier, epoch, state).
- The client applies an event only if its epoch is strictly greater than the last applied epoch for that trip.
- A local optimistic transition does not increment the epoch. It sets a pending flag beside the last applied server state.
- Any server event clears the pending flag, whether it confirms the proposal or contradicts it.

That last line is the one candidates miss. A contradiction must clear the optimism, or the UI shows a proposal that the server already rejected, forever.

**Race one: the double request.** The rider taps request twice, or taps once on a network that retries under them.

| Step | What happens | Why it is safe |
|---|---|---|
| Client mints a request key before any input or output | A UUID stored in the outbox row | The key exists before the network does, so a process death between tap and send loses the intent, not the identity |
| Both attempts carry the same key | Server creates on first, returns the existing trip on second | The client cannot tell which attempt won, and does not need to |
| Key lifetime is stated in the contract | The server retains it longer than the client's outbox retention | An expired key that the client still holds turns a retry into a second trip |

The interviewer's follow-up is usually about the key's lifetime. The honest answer is that it is a contract, that the client's retention must be strictly shorter than the server's, and that this is one of the few places where a client and a server share a constant.

**Race two: cancel versus assign.** The rider taps cancel in the same 300 ms in which dispatch assigns a driver.

- The client renders "Cancelling", not "Cancelled". It is a pending flag over the Requesting state.
- The server resolves the race. If the assign committed first, the response says Assigned with a cancellation fee, and the client must render that without accusing the user of anything.
- The wrong design shows "Cancelled" optimistically, then flips to "Driver on the way". Users read that as the app trying to charge them, and you will read about it publicly.

**Race three: two drivers accept the same offer.** Both taps are inside the countdown, both are valid.

- The server does a compare-and-set on the offer's epoch. One wins.
- The loser must receive a specific outcome, "offer taken", not a generic failure. A generic failure teaches drivers that accepting is unreliable, and they start tapping earlier and harder on every offer, which makes the race more common.
- The client must also expire the offer locally, on a monotonic clock rather than a wall clock, so a device with a skewed or user-adjusted clock cannot show an expired offer as live.

**Race four: ending a trip with no network.**

| Step | Client behaviour | Consequence to be honest about |
|---|---|---|
| Driver taps End with no connectivity | Move to "Ended, unsynced", stop location sampling, write the terminal proposal to the outbox | The fare is provisional. Show it as provisional; do not show a final number you cannot honour |
| Rider's app has no matching event | Rider still sees "On trip" | The two devices genuinely disagree, and no client-side trick fixes it. The server closes it when the driver reconnects |
| Reconnect | Outbox drains, server settles, both clients receive the same epoch | Convergence is at reconnect, and the 10 second requirement is measured from reconnect, not from the tap |

**Version skew: what a client does with a state it has never heard of.** With twenty live app versions, the server will eventually send a state your build does not know.

- Every state in the wire format carries a category: pending, active, terminal.
- An unknown state renders from its category plus a server-supplied display string, and disables every action.
- The client never invents a terminal state, and never maps an unknown state onto the nearest one it knows, which is the bug that ends trips that are still running.

!!! warning "When not to allow optimistic local transitions"

    Optimism is worth it only when the round trip is visible to a human and the action is reversible. For anything that settles money, or that a second party acts on, render a pending state and wait. The measurement that justifies optimism is your p95 tap-to-state figure against the 400 ms threshold. If your users are on good networks and p95 is 250 ms, optimism buys nothing and costs you every reconciliation bug described above.

### Deep dive two: the location pipeline, priced in joules rather than bytes

This dive exists because the intuitive answer, batch harder to save battery, is wrong here, and the arithmetic in the estimation table proves it. Deep dive one was about correctness under concurrency; this one is about energy, which is the axis a backend engineer never has to think about.

**The result to lead with.** Uploading once per second and uploading once per four seconds cost the same energy, because both keep the radio inside its high-power tail. Batching only saves energy when the gap between transmissions exceeds the tail, which is about 15 seconds. On a trip you cannot wait 15 seconds, because the rider's map would be stale beyond the 6 second requirement. Therefore, on a trip, radio energy is the price of the product, and every saving must come from the idle state.

**The sampling policy, by context.**

| Context | Sample rate | Accuracy tier | Upload cadence | Why |
|---|---|---|---|---|
| Offline | none | none | none | No consumer exists. Any sampling here is a bug, and it is the one users notice in their battery settings |
| Idle, online, screen off | 1 per 4 seconds | balanced, network-derived | every 30 seconds | Dispatch needs a block, not a lane, and 30 seconds clears the radio tail |
| Offer pending | 1 per 2 seconds | high | every 4 seconds | The offer's estimated pickup time is being computed from this |
| En route to pickup | 1 per second | high | every 4 seconds | The rider is watching a pin move |
| On trip | 1 per second | high | every 4 seconds | Route, fare and safety all read this |
| Stopped, speed under 1 m/s for 60 seconds | 1 per 10 seconds | balanced | every 30 seconds | A parked car generates identical points. This is the single largest saving available and it costs nothing |
| Battery under 15 percent or the OS is in a low-power mode | halve every rate, widen every cadence to 30 seconds | balanced | every 30 seconds | Degrade before the OS degrades you, and tell the driver you did it |

The stopped-detection row deserves a number. A driver waiting five minutes at a pickup emits 300 points at 1 Hz, of which 299 are the same point. Dropping to 1 per 10 seconds emits 30. That is 90 percent of the samples in a common situation, removed by one predicate.

**The buffer, and why it is durable.**

| Property | Choice | Reason |
|---|---|---|
| Structure | Fixed-size ring on disk, records of equal width | Equal-width records make the ring arithmetic trivial and make a torn write detectable by length |
| Size | 72 hours at 40 bytes per point, so about 10.4 MB | Sized from the retention requirement, and small enough that the ceiling is never the interesting failure |
| Commit | Group commit every 20 points, or on any state transition | One disk sync per 20 seconds on a trip rather than one per second |
| Durability level | Relaxed: a crash may lose the last group | Location is a stream with redundancy. Losing the last 20 points loses 20 seconds of a route the server can interpolate. Compare case study 6, where the same mechanism is configured for full durability because the lost record is an order |
| Ordering | Monotonic sequence number per session, never reused | Lets the server compute loss as gaps, which is the only honest way to measure your own reliability |

**Gaps are data, not absence.** When the client reconnects after a tunnel it uploads the buffered points, and it also uploads an explicit marker for any range it never sampled, for example because the process was dead. The server must be able to distinguish "the driver did not move" from "I have no idea where the driver was". Without the marker, an arrival estimator draws a straight line through a gap and confidently reports a driver who was never on that road.

**The rider side: making 4 second updates look like motion.**

| Technique | What it does | What it costs | When it is wrong |
|---|---|---|---|
| Straight-line interpolation | Animates the pin between the last two known points | Nothing | Shows the car driving through buildings, which reads as a fake |
| Snap to route geometry | Animates along the polyline the driver is following | One route fetch, already needed for the estimate | Wrong the moment the driver deviates, and it hides the deviation |
| Bounded extrapolation | Continues along the route for at most one update interval past the last point | A rule and a timer | Beyond one interval it is fiction, so it must stop |
| Explicit staleness affordance | After a stated silence, dim the pin and label it as last seen | A design conversation | Never wrong, and it is the mechanism that keeps the other three honest |

The senior move is the fourth row. Every animation technique lies a little; the design decision is how long you let it lie before you say so. A rider who is told the position is 40 seconds old trusts the app more than a rider watching a smoothly animated fiction.

!!! warning "When not to build an adaptive sampler"

    A fixed rate is correct when the app is only in the foreground and sessions are short, for example a food delivery tracking screen the user watches for two minutes. The adaptive ladder above costs you a state machine, a set of thresholds nobody can tune without field data, and a class of bugs where the app is stuck in a degraded mode.

    The measurement that justifies it is battery drain per online hour, segmented by state. If idle drain is already inside budget, do the stopped-detection row alone and stop there: it is one predicate and it delivers most of the win.

### Failure modes and evaluation (ride hailing client)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Thundering herd | A deploy or a network blip drops 8,000 connections, all of which reconnect and request state at once | Jittered reconnect over a window sized from the fleet, and a reconciliation fetch that is small by construction |
| Retry storm | The trip service slows, clients time out and retry, and the added load keeps it slow | Retries carry the idempotency key so they are cheap to reject, plus exponential backoff with jitter and a client-side cap on concurrent in-flight proposals |
| Split brain | The rider sees "On trip", the driver sees "Ended, unsynced", and both are correct views of different information | Named as a real state rather than hidden, with convergence at reconnect and a stated 10 second target measured from reconnect |
| Clock skew | A device with a wrong wall clock expires an offer early or shows a countdown that jumps | Every countdown and every timeout runs on a monotonic clock; wall clock timestamps sent to the server are accompanied by a monotonic uptime delta so the server can correct them |
| Metastable failure | The uploader backlog grows faster than it drains after an outage, so every device is permanently behind | Bounded upload concurrency, a drain rate that is measured, and a rule that on-trip points jump the queue ahead of historical backlog |
| Process death | The driver app is reclaimed mid-trip and loses in-memory intent | Durable outbox and durable ring buffer, plus a resume path that is exercised by a test that actually kills the process |
| Stale but plausible UI | The map animates a driver who has not reported a position in two minutes | The staleness affordance, driven by a watchdog rather than by the absence of a callback |

Service level indicators to publish:

- Tap-to-state latency at p50 and p95, per action type, measured from touch-up to first differing frame.
- Duplicate trips per thousand request keys, which must be zero and is the first number to check after any outbox change.
- Location gap rate per driver hour, computed server-side from sequence gaps, split into coverage gaps and process-death gaps.
- Driver position staleness on the rider's map at p95, measured as the age of the newest point rendered.
- Applied-out-of-order transition count, which should be zero once epochs exist.
- Battery drain per online hour at p50 and p90, segmented by state, since a single mean hides the app being stuck in high-accuracy mode.
- Age of the oldest unsynced terminal state across the fleet, which is your real convergence metric.

Alert on: any non-zero duplicate trip rate; unsynced terminal states older than a stated threshold; reconnect rate spikes above a multiple of baseline; and process-death gap rate rising after a release, because that is how you catch a memory regression before the crash rate does.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Tap to visible state under 400 ms | Yes, at about 40 ms, because the transition is local. The cost is the entire reconciliation apparatus in deep dive one |
| Driver position staleness under 6 seconds at p95 | Yes on good coverage: a 4 second upload cadence plus one network leg. No in a tunnel, and the design tells the rider so rather than pretending |
| Zero divergence at trip end | Yes, because terminal states are server-only and the client cannot mint one |
| Converge within 10 seconds of reconnect | Yes, and the metric is defined from reconnect rather than from the user's tap, which is the honest definition |
| Battery under 12 percent per hour | Just barely on trip, at 12.0 percent by the estimation table, and comfortably when the shift is averaged. This is the requirement most likely to fail in the field, so it is the one to instrument first |
| Under 3 MB per driver hour | Yes, at about 0.7 MB on trip. Bytes were never the constraint |
| 72 hour durable log | Yes, at 10.4 MB |

### Where this answer is a simplification (ride hailing client)

- **Android developer documentation, "About background location and battery life"** at https://developer.android.com/develop/sensors-and-location/location/battery, checked 1 August 2026, states the platform guidance behind the sampling ladder above: pass the largest workable interval, use a maximum update delay several times the interval to allow batched delivery, and prefer balanced or low-power accuracy outside the foreground. The page frames accuracy, frequency and latency as the three levers, which is exactly the structure of the context table in deep dive two.
- **Android developer documentation, "Task Scheduling", the persistent work guide** at https://developer.android.com/develop/background-work/background-tasks/persistent, checked 1 August 2026, describes work that survives process restarts and device reboots by being stored in an on-device database, with configurable retry backoff. That is the sanctioned mechanism for draining an outbox after a kill, and it is why the outbox is durable rather than a retry loop in memory.
- **Apple developer documentation, "Background Tasks"** at https://developer.apple.com/documentation/backgroundtasks, checked 1 August 2026, is the reference for the equivalent deferred and refresh scheduling on that platform. Read it before assuming the two platforms give you the same guarantees, because they do not.
- **RFC 9000, "QUIC: A UDP-Based Multiplexed and Secure Transport", May 2021** at https://www.rfc-editor.org/rfc/rfc9000.txt, checked 1 August 2026, defines connection migration in Section 9, where a connection survives a change of network path by using connection identifiers rather than an address pair. A moving vehicle changes cell and sometimes changes network, so this is directly load-bearing for whether a reconnect is even needed.
- **Dispatch and matching are not in this answer at all.** Which driver receives which offer, and how the market clears, is a backend problem covered in the ride hailing case study on [modern-system-design.md](modern-system-design.md). If the interviewer steers there, they may be running a backend round under a mobile title, and it is worth naming that gently.
- **Arrival estimates are a modelling problem.** The client consumes an estimate and animates it; producing one from traffic, historical speeds and the driver's own trace is covered in the delivery time prediction case study on [ml-system-design.md](ml-system-design.md).
- **The idempotency contract is a server-side design in its own right.** Key scope, retention, and what the server returns for a replayed key are covered properly in [product-architecture.md](product-architecture.md). The client depends on that contract and cannot fix a server that gets it wrong.
- **Map rendering is a whole other budget.** Tile fetching, vector rendering and the frame cost of an animated pin are the subject of the maps client case study elsewhere on this page, and they compete for the same 16 ms frame and the same battery.
- **Safety, insurance and regulatory trip recording** impose retention and evidentiary requirements that make some of the relaxed durability choices above unacceptable in specific jurisdictions. That is a legal input to a technical decision, and the honest interview answer is to ask rather than to guess.

### Level delta (ride hailing client)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Draws a client, a server and a websocket, then starts on the state machine | Starts at polling with server-only state and names what breaks | Asks who is authoritative and how many app versions are live before drawing anything, because those two answers decide whether the client may hold state at all |
| Optimistic UI | "We update the UI immediately for responsiveness" | Renders locally and reconciles on the server response | Derives it from a 400 ms budget against a 600 ms p95 round trip, then bounds it: optimism only into non-terminal states, because terminal states settle money |
| Ordering | Not raised | Mentions that events can arrive out of order | Puts a server-assigned epoch on every state, states the discard rule, and adds the line most people miss: a contradicting event must clear the optimistic flag |
| Location | "Send location every few seconds" | Adaptive rates by trip state, batched uploads | Shows that 1 Hz and 4 second batching cost identical energy because of the radio tail, locates the real threshold at about 15 seconds, and concludes that on-trip radio cost is unavoidable so the saving must come from idle |
| Gaps | Treats missing data as missing | Buffers durably and uploads on reconnect | Sends explicit gap markers so the server can distinguish a stationary driver from an unknown one, and names the arrival estimator that would otherwise draw a confident straight line |
| Races | Handles the double tap with a disabled button | Adds an idempotency key | Enumerates four races with their resolutions, and notices that the key's lifetime is a shared constant between client and server, with the client's retention strictly shorter |
| Version skew | Assumes one version | Adds a version field | Makes states carry a category so an unknown state renders safely, and states the rule that a client never maps an unknown state onto the nearest known one |
| Honesty | Claims the design handles offline | Notes some cases need reconnection | Names "Ended, unsynced" as a first-class product state with its own screen and a provisional fare, and says out loud that the two devices genuinely disagree until reconnect |

What each level added:

- **Mid to senior:** the design starts small and grows against named failures, and the location pipeline stops being a timer and becomes a policy that varies with context.
- **Senior to staff:** every choice acquires an arithmetic justification, the intuitive energy answer is disproved rather than repeated, and the ugliest state in the product is volunteered with a screen designed for it instead of being hidden behind a spinner.

---

## Case study 6: Trading client

Case study 5 had one hard number, the tap-to-state budget, and one hard rule, that money settles on the server. This prompt has a hard number on every path and a rule that is stricter than anything above it: a duplicate order is not a bug you fix in the next release, it is a headline. The two deep dives are chosen accordingly, one on a latency budget you can defend with measurements, one on integrity when the process can die mid-submit.

### The prompt (trading client)

"Design the mobile app for a retail brokerage: a live watchlist, a symbol detail screen with a chart, and order entry."

### Questions that fork the design (trading client)

| Question worth asking | If the answer is A the design becomes | If the answer is B the design becomes |
|---|---|---|
| Retail or professional users? | Retail: a few hundred milliseconds of price staleness is acceptable, and conflating updates is normal and correct | Professional: the active symbol may not be conflated at all, glass-to-glass moves under 100 ms, and the whole rendering approach changes |
| How many symbols stream at once? | Twenty visible: a per-symbol subscription is fine and the client can hold every one in memory | A two hundred symbol watchlist scrolled quickly: subscriptions follow the viewport, the server conflates, and subscription churn becomes its own problem |
| May the client compute numbers the user trades on, such as buying power? | No: every figure is a server value, which costs round trips and simplifies correctness enormously | Yes, for display: you now need a reconciliation rule and a policy for what happens when the client's arithmetic and the server's disagree, which will happen |
| Can orders be cancelled and replaced from the app? | No, submit only: order entry is fire and confirm, and the state machine has four states | Yes: pending-cancel and pending-replace are real states, a cancel can lose a race with a fill, and the UI must express both outcomes |
| Are market data entitlements per user and per symbol? | No: quotes are quotes, and you can cache freely | Yes: entitlements gate every subscription, delayed data is a fallback rather than an error, and you may not share a cached quote across users |
| Must the app hold up at the market open? | No, average load is the design point | Yes: you design for a burst that is roughly twenty times steady state, and the burst is where every queue you did not bound becomes visible |
| Continuous markets or session-based? | Continuous: no session state, no closed rendering path | Sessions: pre-market and post-market flags, a market-closed screen, and a daily reset that invalidates half your caches at a known instant, which is a scheduled thundering herd |
| Do fills arrive by push notification? | No: the app is the only channel and must be open | Yes: push is best-effort, so the app must never treat the absence of a push as the absence of a fill, and every resume path re-queries |

### Requirements (trading client)

Functional:

- A watchlist of symbols with a live price, change, and a sparkline.
- A symbol detail screen with an intraday chart and a depth or last-trade view.
- Order entry for market and limit orders, with cancel and replace.
- A positions and orders screen whose contents survive a kill and relaunch.
- A staleness indicator on every displayed price, so the user can tell live from frozen.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Symbols visible on one watchlist screen | 20 | GIVEN by the layout: about eight rows visible plus overscan |
| Watchlist size | 200 | ASSUMED; ask, because it decides whether subscriptions follow the viewport |
| Tick rate per liquid symbol at the open | 50 per second | ASSUMED, and clearly a placeholder. A liquid large-cap prints frequently at the open; measure your own feed before trusting this |
| Tick rate per symbol in steady state | 2 per second | ASSUMED, on the same basis |
| Glass-to-glass price update | under 250 ms at p95, from server publish to the frame that shows it | ASSUMED for retail; a professional product would set this four times lower |
| Order submit acknowledgement visible | under 1,000 ms at p95 | ASSUMED |
| Frame budget | 16 ms at 60 Hz, 11 ms at 90 Hz, 8 ms at 120 Hz | GIVEN by the display refresh rate, and matching the published platform thresholds cited at the end of this case study |
| Duplicate orders | zero | GIVEN. This is the requirement that outranks every other one on the page |
| Cold start to a usable watchlist | under 2 seconds on the reference device | ASSUMED, and well inside the published excessive-cold-start threshold cited below |
| Cellular bytes per hour of active watching | under 20 MB | ASSUMED; users watch markets on trains |
| Stale price detection | a price with no update and no heartbeat for 5 seconds is visibly marked | ASSUMED, and it is the single most important safety feature in the product |

### Estimation that eliminates (trading client)

| Calculation | Result | What it rules out |
|---|---|---|
| 20 visible symbols times 50 ticks per second at the open | 1,000 updates per second | Rules out one UI update per tick. At 60 frames per second that is 16.7 updates inside a single frame, so 16 of every 17 would be overwritten before a pixel changed |
| A packed tick: symbol id 4 bytes, price 4, size 4, timestamp 8, flags 1 | 21 bytes | Rules out a text encoding for the hot path: the same tick as a small object with a symbol string is about 90 bytes, and the ratio matters four lines below |
| 1,000 ticks per second times 21 bytes times 3,600 | 75.6 MB per hour | Rules out sending every tick to the device at the open. It breaches the 20 MB budget by 3.8 times, and this is before framing overhead |
| Server-side conflation to 10 updates per second per symbol: 20 times 10 times 21 bytes times 3,600 | 15.1 MB per hour | Rules out client-only conflation. The bytes must be dropped upstream of the radio, because a byte the device receives has already cost the energy |
| 200 symbol watchlist at 2 ticks per second in steady state | 400 updates per second for 180 rows nobody can see | Rules out subscribing to the whole watchlist. Subscriptions follow the viewport plus a small overscan, which is 90 percent less traffic for identical visible behaviour |
| Glass-to-glass budget: network one way 60 ms, decode and dispatch 5 ms, conflation window 100 ms, wait for the next frame up to 16 ms, render 8 ms | 189 ms, leaving 61 ms of headroom against 250 ms | Rules out a conflation window wider than about 150 ms, and rules out decoding on the main thread: 5 ms of decode is a third of a 60 Hz frame, every frame |
| 1,000 ms order budget against a 600 ms p95 round trip | room for exactly one round trip | Rules out a pre-flight buying-power check before submit. Two round trips is 1,200 ms at p95, so validation must be pre-fetched and re-checked server-side |
| Reconnect after a 30 second tunnel: 200 symbols times a 40 byte snapshot | 8 KB | Rules out replaying the tick log across the gap, which would be 30 seconds times 400 ticks per second times 21 bytes, so 252 KB of data that is entirely superseded |
| One in-flight order, one process death, one unknown outcome | one query is required, and it must be by an identifier the client minted before sending | Rules out resubmitting on resume. A resubmitted market order can double a position, and no amount of user apology fixes that |
| Chart: one intraday minute bar per minute for a 6.5 hour session, at 5 numbers of 4 bytes | 390 bars, 7.8 KB | Rules out streaming raw ticks to draw an intraday chart. Bars are two orders of magnitude cheaper and are what the chart actually renders |

### V0, the simplest thing that works (trading client)

**Predict first: what actually breaks first if you poll quotes every three seconds, load or perception?**

??? note "Show answer"

    Perception, and it breaks in a specific and dangerous way rather than a gradual one. Load is trivial: 20 symbols in one batched request every 3 seconds is one request per user per 3 seconds.

    The danger is that a three second old price looks exactly like a live one. A user places a limit order against a number that moved before they tapped, and the app gave no signal. The first thing to build is therefore not a stream, it is the staleness indicator, and it is worth saying so in the room because it inverts the expected answer.

```
+-----------+   batched poll, 20 symbols   +----------------+
| Client    | <--------------------------> | Quote service  |
| watchlist |                              +----------------+
+-----------+
      |  submit with client order id
      |
+----------------+     +------------------+
| Order service  | --> | Order status by  |
| authority      |     | client order id  |
+----------------+     +------------------+
```

**Caption.** V0 polls one batched quote request for the visible symbols every three seconds, and submits orders with a client-minted identifier to an order service that can be queried by that same identifier. There is no stream, no local store, and no conflation.

Why this is correct at a real scale:

- The client order identifier is present from the first line of code, so the integrity requirement is met before any of the performance work exists.
- One request per three seconds costs one radio wake-up per three seconds, which is the same energy as a stream and far less code.
- Every screen is a function of the last response, so there is no store to corrupt and no ordering to get wrong.
- The staleness rule is trivial to implement, because the client knows exactly when the last response arrived.

!!! warning "When not to move off polling quotes"

    If the product is a long-term investing app where users check a balance twice a day, polling is correct forever and a streaming stack is a large amount of code that improves a number nobody feels. The measurement that justifies streaming is the distribution of time between a price change on the venue and the user seeing it, weighted by whether the user acted.

    If almost nobody places an order within ten seconds of opening a symbol, they are not trading against the tick, and the cheaper move is a shorter poll on the symbol detail screen alone.

### Breaking point, then V1 (trading client)

**What fails: the open.** At 50 ticks per second per symbol, polling shows the user one of every 150 prices, and the 3 second staleness is now larger than the interval in which the price meaningfully moves. How you observe it: log, for each rendered price, the venue timestamp it came from, and plot the age distribution. If the p95 age exceeds the interval in which the price typically moves by more than the spread, the display is misleading rather than merely late.

**What also fails: bytes and wake-ups on the symbol detail screen.** A user watching one symbol at a one-second poll wakes the radio 3,600 times an hour for data that is mostly unchanged.

V1 replaces polling with a stream and adds the client machinery that a stream requires:

```
+------------+   +---------------+   +------------------+
| Socket in  |-->| Decode, off   |-->| Tick router      |
| conflated  |   | the UI thread |   | last value per   |
+------------+   +---------------+   | symbol           |
                                     +------------------+
                                              |
                                              |
+------------+   +---------------+   +------------------+
| Rendered   |<--| Frame flush   |<--| Dirty symbol set |
| rows       |   | one per vsync |   | bounded          |
+------------+   +---------------+   +------------------+
```

**Caption.** V1 receives server-conflated updates on one connection, decodes them off the UI thread, writes last-known values into a per-symbol store, marks the symbol dirty, and flushes all dirty symbols once per display frame rather than once per tick.

Why each box is there, in one line each:

- Decoding off the UI thread, because the estimation table shows 5 ms of decode inside a 16 ms frame.
- A last-value store, because a price is state, not an event, and state has exactly one current value.
- A dirty set, because the flush must be proportional to the number of changed visible symbols, not to the number of ticks received.
- A single flush per frame, because the display cannot show more than one value per frame no matter how many arrive.

!!! warning "When not to build a tick router and frame flush"

    Below roughly 10 updates per second across the whole screen, direct binding from the network callback to the view is simpler and behaves identically. The measurement that justifies the router is dropped frames during a burst, attributed to the update path. If your frame timing during the open is clean without it, the router is a store, a scheduler and a threading contract bought to fix a graph that was already flat.

### Breaking point, then V2 (trading client)

**What fails: the burst arrives anyway.** Server-side conflation smooths the average and does nothing for a coordinated event, for example an index-wide move in which all 20 visible symbols update in the same 100 ms window. The client queue grows, the device falls behind, and the prices on screen are old while the socket is busy.

How you observe it: measure the depth of the inbound queue at the moment of each frame flush, and the age of the oldest undelivered update. A queue that recovers is fine. A queue whose depth trends upward across a minute is a metastable failure in progress, and the device will not recover on its own because it is spending its time on updates that are already superseded.

**What also fails: the process dies with an order in flight.** The user taps submit on the underground, the app is killed, and on relaunch the app knows nothing.

V2 adds three things:

| Addition | Mechanism | Why it is the right shape |
|---|---|---|
| Adaptive client conflation with a safe drop policy | The per-symbol store holds one value, so intermediate prices are dropped by construction. Event streams, such as a trade tape, get a bounded queue with an explicit drop counter rather than silent loss | Dropping an intermediate price loses nothing, because the newer price supersedes it. Dropping a trade print loses a row the user can count, so it must be counted and disclosed |
| A behind-indicator driven by queue age | If the oldest undelivered update exceeds a threshold, the UI shows that it is catching up | The failure the user must never suffer is a screen that looks live and is not. Telling them is cheap and it is the honest move |
| A durable order journal with a reconciliation ladder | The client order identifier and intent are written to disk before the request leaves, and every resume path walks unresolved records | The only defence against an unknown outcome is an identifier that survives the process, which is deep dive two |

!!! warning "When not to add client-side adaptive conflation"

    If the server already conflates and your measured queue depth never exceeds a handful of updates even at the open, a second conflation layer adds a tuning knob, a drop counter and a class of "why did that print not appear" support tickets. The measurement that justifies it is queue depth at flush time at p99 during the highest-volume five minutes of the week. Below about five, skip it.

### Deep dive one: the glass-to-glass budget, and measuring it without lying to yourself

The budget from the estimation table sums to 189 ms against a 250 ms objective. That is arithmetic. The dive is about the two harder questions: which stage do you actually control, and how do you measure a one-way latency between two clocks that disagree.

**Stage by stage, and who owns it.**

| Stage | Budget | Owner | What you can do about it |
|---|---|---|---|
| Venue to your server | not in the budget | Upstream | Nothing on the client. Do not accept it into your SLI, or you will be paged for someone else's link |
| Server publish to device receive | 60 ms | Network | Choose a transport that survives a path change, keep the connection warm, and place the edge closer. Beyond that it is physics |
| Decode and dispatch | 5 ms | You | A compact binary encoding and a decode that allocates nothing per tick. This is the cheapest stage to ruin and the cheapest to fix |
| Conflation window | 100 ms | You | The single largest controllable term. Halving it doubles the update rate and the byte cost, so it is a dial with a price |
| Wait for the next frame | up to 16 ms | The display | Unavoidable. Flush on the frame callback, not on a timer, or you pay this twice |
| Layout and draw | 8 ms | You | Update a value, not a layout. A price change that triggers a re-layout of a row is the difference between 1 ms and 8 ms |
| **Sum** | **189 ms** | | **61 ms of headroom, which is the only reason the budget survives one slow stage** |

**How to measure it honestly.** You cannot measure a one-way latency between a server clock and a device clock, because the offset between them is unknown and drifts. Candidates who claim otherwise are subtracting two numbers that are not comparable.

- Measure the client-side segment absolutely: from the moment bytes arrive at the socket to the frame callback that commits the change. One clock, one process, no skew.
- Measure the network segment as half of a round trip taken on the same connection, and state the assumption that the path is roughly symmetric, which is often false on cellular uplinks.
- Report the two separately, and never add them into a single number that pretends to be measured end to end.
- Anchor on the venue timestamp carried in the tick for staleness, not for latency. Age relative to a venue timestamp is comparable across devices even when the absolute offset is unknown, because every device compares against the same source.

**State versus events, the distinction the whole dive rests on.**

| Kind | Example | Correct buffer | What conflation does | What loss means |
|---|---|---|---|---|
| State | Last price, best bid, best ask, position quantity | One slot per key | Drops superseded values, which is free | Nothing, provided the newest arrives |
| Events | Trade prints on a tape, order status transitions, fills | Bounded queue with a counter | Nothing may be conflated | A missing row the user can notice, so it must be counted and shown |
| Derived | Percent change, sparkline, profit and loss | Recomputed from state | Not applicable | Recomputed on the next state update |

The senior sentence is: conflation is safe for state and unsafe for events, and the bug is always someone treating an event stream as state because it arrived on the same socket.

**Main thread discipline, in concrete rules.**

- No decode, no parse, no allocation of collection objects on the UI thread. The 5 ms decode budget is spent on a background thread.
- One immutable snapshot per frame, published to the UI thread with a single reference assignment.
- A price change updates a text value only. If it changes a size, a colour resource lookup, or anything that reflows a row, the 8 ms draw budget becomes the 16 ms frame budget.
- Flashing a cell on change is a real requirement and a real cost. Animate with a property that does not trigger layout, and cap the number of simultaneous flashes, because 20 rows flashing at once is 20 animations competing for the same frame.

!!! warning "When not to spend effort on the latency budget at all"

    If the product serves long-horizon investors, a 2 second update is indistinguishable from a 200 ms one for the decision being made, and the whole budget above is expensive theatre. The measurement that justifies the work is the distribution of time between a user opening a symbol and placing an order. If the median is minutes, spend the effort on the staleness indicator and the order path instead, both of which matter at every latency.

### Deep dive two: order integrity when the process can die between intent and acknowledgement

This dive is deliberately not another latency budget. It is the one part of the app where being late is fine and being wrong is not, and it uses the opposite durability setting from the location buffer in case study 5, which is the point of putting them on the same page.

**The framing.** The client sends a submit and the process dies before any response arrives. Three worlds are consistent with that observation: the request never left, the request arrived and was accepted, or the request arrived and was rejected. The client cannot distinguish them from local evidence, ever. Every mechanism below exists to make that ignorance survivable rather than to eliminate it.

**Write-ahead, in order.**

| Step | What is written | Why the order matters |
|---|---|---|
| 1 | Mint a client order identifier | It must exist before any input or output, so that a crash at any later point leaves an identity to query with |
| 2 | Write a journal record: identifier, symbol, side, quantity, order type, limit price, state SUBMITTING, and a monotonic timestamp | The record is the intent. If it is not on disk before the request goes out, a crash leaves an order that may exist on the server and nowhere on the client |
| 3 | Sync the journal | Without the sync, a power loss can lose the record while the request still reached the server |
| 4 | Send the request | Everything after this point is recovery, not prevention |
| 5 | On response, update the record to ACCEPTED or REJECTED with the server's order identifier | The server identifier is what support and the user will see |

Step 3 is where this design and case study 5 diverge. The location ring buffer is configured for relaxed durability, because losing the last 20 points loses 20 seconds of a route the server can interpolate. The order journal is configured for full durability, because losing the last record loses the only local evidence that an order might exist. Same mechanism, opposite setting, and the reason is the cost of losing one record.

**The reconciliation ladder, run on every launch and every foreground.**

| Journal state found | Action | What the user sees |
|---|---|---|
| SUBMITTING | Query the order service by client order identifier | A row marked "checking", never a row marked "sent" |
| Query returns accepted or working | Update the journal, adopt the server state | The live order, with its server identifier |
| Query returns filled or partially filled | Update the journal, and refresh positions before showing anything | The fill, and a position that already includes it |
| Query returns rejected | Update the journal with the reason | The rejection reason, which is the one place a raw server message is better than a friendly one |
| Query returns not found | Stop. Do not resubmit | "We could not confirm this order", with a link to the orders screen and no automatic action |

The "not found" row is the whole dive. Not found is ambiguous: the request may never have arrived, or it may have arrived and the query index may be lagging behind the write path. Automatic resubmission on that ambiguity is how a user ends up with two positions. The correct behaviour is to surface the ambiguity, offer a manual retry that reuses the same identifier, and let the user decide.

**The contract the client depends on.** None of the above works without three server-side guarantees, and naming them is what separates a client engineer who has shipped this from one who has read about it:

- The same client order identifier submitted twice returns the same order, and never creates a second one.
- The identifier is scoped per account, so two accounts cannot collide.
- The server retains the identifier strictly longer than the client's journal retention, which here is 72 hours. An expired identifier that the client still holds turns a recovery query into a new order.

**Cancel and replace, where the state machine earns its keep.**

| State | Legal next states | The race |
|---|---|---|
| Working | Pending-cancel, pending-replace, partially filled, filled, expired | None |
| Pending-cancel | Cancelled, filled, partially filled then cancelled | A cancel can lose to a fill that was already in flight. Both outcomes are correct |
| Pending-replace | Replaced, filled at the original terms, rejected | A replace is a cancel and a new order, so it has two ways to fail |
| Filled | terminal | |
| Cancelled | terminal | |

Two rules follow:

- Never render "Cancelled" optimistically. Render "Cancelling". A user who is told an order is cancelled and then sees a fill has been lied to about their own money.
- Optimism is allowed in exactly one place: a newly submitted order may appear in the list immediately, greyed, labelled as sending, and excluded from every total. It is a receipt for a tap, not a claim about the market.

**A worked recovery, narrated.** I am on the underground. I tap submit on a market order for 10 shares. The journal record hits disk and is synced, and the request goes out into no signal. The OS reclaims the app while I am reading something else. I resurface, open the app, and the launch path finds one SUBMITTING record.

- Before reading on, say why the query must be by the client identifier rather than by "my most recent order". Because the most recent order might be an older one that succeeded, and adopting it would attach my recovery to the wrong object.
- The query returns not found. The app shows "could not confirm", and does not resubmit.
- Thirty seconds later a fill push arrives, because the request had in fact reached the server before the radio died and the index was simply behind. The app reconciles by identifier, the row becomes a fill, and the position updates.
- Before reading on, say what would have happened if the app had auto-retried at the not-found step. It would have submitted a second market order, and I would own 20 shares instead of 10, at two different prices, and the app would have caused it.

!!! warning "When not to build the journal and the ladder"

    A paper trading mode, a watch-only app, or any product where the user cannot transact does not need any of this, and building it costs a durable store, a synchronous write on a user-facing path, and a recovery path that must be tested by actually killing the process. The measurement that justifies it is simply whether an unacknowledged request can have a financial effect. If it can, the bar is not a metric, it is a yes.

### Failure modes and evaluation (trading client)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Stale but plausible display | The socket is dead, the last price is from four minutes ago, and it looks exactly like a live price | A per-symbol watchdog driven by heartbeats, not by the absence of ticks. No update and no heartbeat within the stated window means the price is visibly marked stale |
| Thundering herd | The market opens, or a deploy drops every connection, and the whole user base reconnects and requests snapshots at once | Jittered reconnect, snapshot-then-stream rather than replay, and a subscription set that is the viewport rather than the watchlist |
| Metastable failure | The inbound queue grows faster than the device drains it during a burst, and never recovers because the device is busy on superseded data | One slot per state key, bounded queues for events with an explicit drop counter, and a behind-indicator when the oldest undelivered update ages past a threshold |
| Retry storm on submit | Submits time out, clients retry, the added load extends the timeouts | Retries reuse the client order identifier so they are cheap to reject, with backoff, jitter, and a hard cap of one in-flight submit per identifier |
| Split brain on order state | The app shows working, the venue shows filled | The server is authoritative for every order state, push is treated as a hint that triggers a query rather than as truth, and every foreground triggers a reconciliation |
| Clock skew | A device with a wrong clock computes a negative price age and renders a price as fresh forever | Staleness is computed against venue timestamps carried in the data, and all local timers run on a monotonic clock |
| Session boundary herd | Every client invalidates its day's data at the same instant when the session rolls | Spread the invalidation over a window, and make the rollover a server-pushed event rather than a client-side wall-clock trigger |
| Poison update | One malformed message causes the decoder to throw, and the connection restarts, forever | Decode failures are counted and skipped, not fatal. A connection that restarts more than a stated number of times in a minute stops and reports |

Service level indicators to publish:

- Client-segment glass-to-glass latency at p50 and p95, socket arrival to frame commit, which is skew-free by construction.
- Rendered price age at p95, computed against venue timestamps.
- Staleness watchdog trip rate per session, split by network type. This is the honest reliability number for the whole product.
- Dropped event count by stream, which must be visible in telemetry even when it is invisible in the UI.
- Inbound queue depth at flush time, at p99, over the busiest five minutes of the week.
- Order submit acknowledgement latency at p95, and the rate of unknown-outcome orders per thousand submits.
- Duplicate orders per thousand submits, which is a release blocker at any non-zero value.
- Frame drop rate during the first two minutes of the session, since that is the worst case and an average across the day hides it.

Alert on: any duplicate order; unknown-outcome rate above a stated floor; watchdog trip rate rising after a release; and queue depth trending across a burst rather than recovering.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Glass-to-glass under 250 ms at p95 | Yes on the budget, at 189 ms with 61 ms of headroom, and the client-owned portion is 13 ms of it. The rest is network and the conflation dial |
| Zero duplicate orders | Yes, provided the server honours the identifier contract. The client cannot deliver this alone and should say so |
| Order acknowledgement under 1,000 ms at p95 | Yes, at one round trip, which is only possible because validation was pre-fetched |
| 20 MB per hour | Yes, at 15.1 MB at the open with viewport subscriptions, and far less in steady state. This was the number that forced server-side conflation |
| Cold start under 2 seconds | Not answered by this design. It is decided by initialisation work and by how much of the watchlist is cached, which is a separate module of this course |
| Visible staleness within 5 seconds | Yes, and it is the one mechanism that keeps every other approximation on this page safe |

### Where this answer is a simplification (trading client)

- **Android developer documentation, "Slow rendering"** at https://developer.android.com/topic/performance/vitals/frozen, checked 1 August 2026, gives the frame windows this case study budgets against: about 16 ms at 60 Hz, 11 ms at 90 Hz and 8 ms at 120 Hz, with frames over 700 ms classed as frozen and an unresponsive UI thread beyond five seconds producing an application-not-responding report. It also states that overrunning the window causes the frame to be dropped rather than shown late, which is why the flush is aligned to the frame callback.
- **Android developer documentation, "App startup time"** at https://developer.android.com/topic/performance/vitals/launch-time, checked 1 August 2026, defines cold, warm and hot start and publishes the thresholds at which each is treated as excessive: 5 seconds cold, 2 seconds warm, 1.5 seconds hot. The 2 second cold-start requirement above is deliberately stricter than the platform's excessive threshold, and the page is the right place to argue that with a product owner.
- **SQLite, "Write-Ahead Logging"** at https://sqlite.org/wal.html, checked 1 August 2026, documents the durability trade this dive depends on: with synchronous set to NORMAL the write-ahead log is not synced on every commit, so a power loss can lose recent transactions, whereas FULL syncs on commit. That is precisely why the order journal uses the stricter setting and the location buffer in case study 5 does not.
- **RFC 9000, "QUIC: A UDP-Based Multiplexed and Secure Transport", May 2021** at https://www.rfc-editor.org/rfc/rfc9000.txt, checked 1 August 2026, defines connection migration in Section 9, which is what allows a stream to survive a user walking from a train onto the street without a full reconnect and a fresh snapshot for every symbol.
- **Market data entitlement and licensing** is a legal and commercial system that constrains caching, sharing and even screenshotting, and it is frequently the reason a design that is technically obvious is not permitted. In an interview, ask; do not design around it silently.
- **Order routing, risk checks and the venue gateway** live entirely on the server. A client round should not attempt them, and the exchange-side design is closer to the payments and ledger case study on [modern-system-design.md](modern-system-design.md).
- **The idempotency contract this design leans on** is designed properly in [product-architecture.md](product-architecture.md), including key scoping, retention and what a replayed key must return. The client is a consumer of that contract and inherits its bugs.
- **Charting at depth** is its own engineering problem: level-of-detail selection, incremental bar updates, and gesture handling that must not fight the scroll container. The frontend treatment of a heavy visualisation surface is in [frontend-system-design.md](frontend-system-design.md).
- **Nothing here is investment advice or a regulatory design.** Record keeping, suitability checks, disclosures and audit retention change what the client may show and store, and they vary by jurisdiction.

### Level delta (trading client)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Draws a websocket and a list, and starts on the chart | Starts at polling, then derives why it fails at the open | Asks whether the users are retail or professional and whether entitlements are per symbol, because both answers change the transport and the caching rules before a box is drawn |
| Update path | Binds the socket callback to the row | Coalesces updates and flushes on a timer | Flushes on the frame callback, keeps one slot per state key, and separates state from events with the rule that conflation is safe for one and unsafe for the other |
| Bytes | Not raised | Notes that JSON is heavy and switches to a binary encoding | Shows that 1,000 ticks per second is 75.6 MB per hour, that this breaches the budget by 3.8 times, and that the drop must happen server-side because a received byte has already cost radio energy |
| Latency measurement | "We measure end to end latency" | Adds timestamps at each stage | Points out that a one-way latency between two clocks is not measurable, reports the client segment absolutely and the network segment as half a round trip, and uses venue timestamps for staleness rather than for latency |
| Order safety | Disables the button while submitting | Adds a client order identifier and retries | Writes and syncs the journal before the request leaves, refuses to auto-resubmit on a not-found result, names the ambiguity that makes not-found dangerous, and states the three server guarantees the client depends on |
| Cancel semantics | Shows cancelled on tap | Shows a pending state | Enumerates the outcomes of a cancel that loses to a fill, and rules that optimism is permitted only for a greyed sending row excluded from every total |
| Failure the user feels | Not raised | Adds a connection indicator | Identifies the frozen-but-plausible screen as the worst failure in the product, builds a heartbeat watchdog rather than relying on the absence of ticks, and makes the trip rate a published reliability metric |
| Honesty | Claims zero duplicate orders | Notes the server must be idempotent | Says the client cannot deliver the zero-duplicate requirement alone, states the identifier retention relationship between client and server, and names it as a shared constant |

What each level added:

- **Mid to senior:** the update path becomes a pipeline with a store rather than a callback, and the order path acquires an identity that survives a retry.
- **Senior to staff:** the numbers decide where work happens (server-side conflation, because energy is spent on receipt), the measurement methodology is corrected rather than assumed, and the design's dependence on a server contract is stated instead of quietly assumed.

---

## Case study 7: Image loading library

This is a component prompt, not an app prompt, and it is scored differently. Nobody will ask you how it scales to ten million users. They will ask what the function signature is, what happens when the caller cancels, and who owns the bitmap after you hand it over. The two deep dives reflect that: one on memory arithmetic, which is the physics of the problem, and one on the public contract, which is the product.

If the interviewer says "design an image loading library" and you spend twelve minutes drawing a content delivery network, you have answered a different question. Say in the first minute that you are treating the origin as given and designing the on-device component, and ask them to redirect you if that is wrong.

### The prompt (image loading library)

"Design an image loading library for a mobile app: given a URL and a place to put it, show the image."

### Questions that fork the design (image loading library)

| Question worth asking | If the answer is A the design becomes | If the answer is B the design becomes |
|---|---|---|
| A general-purpose library for other teams, or an in-house component for one app? | General: the public surface, binary size, threading contract and versioning policy are most of the work, and half the internals can be naive | In-house: you can hard-code the transport, the cache policy and the image sizes, and delete the entire extensibility layer |
| Do you control the image origin? | Yes: request the right variant by size and the on-device downsampling problem largely disappears, at the cost of a variant matrix and cache fragmentation upstream | No: the origin serves one large original, so decode-time downsampling is mandatory and is the first thing you build |
| Is the main surface a scrolling list or a full-screen viewer? | List: cancellation, view recycling and prefetch dominate, and memory is spent on many small bitmaps | Viewer: progressive decode, zoom tiling and one very large bitmap dominate, and the cache policy is almost irrelevant |
| Are transformations part of the surface? | No: the URL is the cache key, and everything is simple | Yes: the key is no longer the URL, which is the single most consequential decision in the whole design and the one candidates get wrong |
| Must images be available offline? | No: the platform HTTP cache may be sufficient and you can ship without a disk layer | Yes: the disk layer becomes a store with an explicit retention policy, and eviction becomes a product decision rather than an implementation detail |
| Who owns the threading contract? | The library: callers get results on the main thread and never think about it, but composing with the caller's own concurrency is awkward | The caller supplies a dispatcher: testable and composable, at the cost of a larger surface and more ways to hold it wrong |
| Is binary size budgeted? | No: you can bundle decoders and support formats the platform lacks | Yes: you use platform decoders only, lose some format coverage, and must say so in the documentation rather than in a bug report |
| Do animated images have to work? | No: one decode path, one bitmap, done | Yes: decoding becomes streaming and stateful, memory becomes a function of frame size times buffered frames, and it deserves its own subsystem |

### Requirements (image loading library)

Functional:

- Load an image from a URL into a target, with a placeholder while loading and a distinct error state.
- Cancel automatically when the target is recycled or detached, without the caller writing cancellation code.
- Apply transformations such as resize, crop, rounded corners or blur, with correct caching.
- Prefetch images that are about to be needed, at a lower priority than visible work.
- Serve from a memory cache, then a disk cache, then the network, and report which tier answered.
- Allow the caller to substitute the network layer and the decoder without forking the library.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Memory cache ceiling | 25 percent of the app's available heap | ASSUMED; derived into a byte figure below. The percentage is the contract, the byte figure is computed at runtime |
| Disk cache ceiling | the smaller of 250 MB and 2 percent of free space | ASSUMED; a library that fills a user's phone will be uninstalled and blamed |
| Memory-tier hit to pixels | inside one frame, so 8 ms at 120 Hz | DERIVED from the frame budget cited at the end of this case study |
| Disk-tier hit to pixels | under 100 ms | ASSUMED; slower than a frame but faster than a user notices as a load |
| Main thread work per load call | under 1 ms | ASSUMED; a 12 row screen binding at once must not spend a frame in the library's own bookkeeping |
| Decodes on the main thread | zero | GIVEN; this is a rule, not a target, and the arithmetic below is why |
| Binary size added | under 300 KB | ASSUMED; ask, because for some teams this is the deciding criterion |
| Initialisation cost at process start | under 5 ms | ASSUMED; a library that does input or output in its initialiser is a cold start regression nobody attributes to it |
| Cancellation to network stop | under 50 ms | ASSUMED; a fling past 100 rows must not keep 100 sockets busy |
| Wrong image delivered to a recycled target | zero | GIVEN; it is the defining bug of this component |

### Estimation that eliminates (image loading library)

| Calculation | Result | What it rules out |
|---|---|---|
| A full-screen image on the reference device: 1080 times 2400 times 4 bytes per pixel | 10.4 MB decoded | Rules out reasoning about images by their file size. A 200 KB file is a 10.4 MB object once decoded, and the heap only counts the second number |
| Twelve such bitmaps held for a scrolling list | 124 MB against a 256 MB heap | Rules out caching decoded bitmaps at source resolution. Half your heap for twelve list rows is an out-of-memory kill waiting for a second screen |
| The same image decoded for a 360 by 240 list thumbnail: 360 times 240 times 4 | 345 KB, so 30 times smaller | Rules out "decode once, scale when drawing". The scaling must happen during decode, because after decode the memory is already spent |
| A 4,000 by 3,000 camera photo at 4 bytes per pixel | 48 MB decoded | Rules out decoding without reading the dimensions first. Three of these is an immediate process kill on the reference device |
| Memory cache at 25 percent of a 256 MB heap, divided by 345 KB per thumbnail | 64 MB, so about 185 thumbnails, which is roughly 15 screens of a 12 row list | Rules out sizing the cache by entry count. 185 thumbnails and 6 full-screen images are the same count and differ by 20 times in bytes |
| Disk cache of 250 MB divided by an ASSUMED 120 KB average encoded image | about 2,080 images | Rules out treating disk as unlimited history. That is a few days of ordinary browsing, so eviction order is a product-visible decision |
| A fast fling passes about 10 rows per second, so 100 rows in 10 seconds, of which about 8 are still visible at the end | 92 of 100 requests are waste | Rules out fire-and-forget requests. Cancellation is not an optimisation in this component, it is the architecture |
| Four parallel fetches on a 375 KB per second link, at 120 KB per image | 0.32 seconds per image, so 12.5 images per second | Rules out unbounded parallelism, which splits the same bandwidth and delays every image equally, and rules out a single fetcher, which serialises a visible row behind an off-screen prefetch |
| Decode and downsample of a 120 KB image at an ASSUMED 15 ms on one mid-tier core, times 12 visible rows | 180 ms, which is 11 dropped frames at 60 Hz | Rules out main thread decode arithmetically rather than by dogma, and shows that two decode threads bring the same work to 90 ms of wall clock |
| Expected time to pixels with a 60 percent memory hit, a 60 percent disk hit on the remainder, and ASSUMED costs of 0.1 ms, 20 ms and 400 ms | 0.06 plus 4.8 plus 64, so 68.9 ms | Rules out nothing yet, but it is the baseline for the next two rows |
| The same with the memory hit raised to 70 percent | 0.07 plus 3.6 plus 48, so 51.7 ms | Rules out nothing on its own. A ten point memory hit gain buys 17 ms |
| The same with the memory hit left at 60 percent and the disk hit raised to 80 percent | 0.06 plus 6.4 plus 32, so 38.5 ms | Rules out "make the memory cache bigger" as the first move. A twenty point disk gain buys 30 ms, nearly twice as much, because it removes network calls rather than disk reads |

That last group is the one to say out loud. The instinct is to grow the memory cache, and the arithmetic says the disk cache is where the leverage is, because only the disk tier removes a 400 ms network call.

### V0, the simplest thing that works (image loading library)

**Predict first: at what point is the correct answer "do not build this library"?**

??? note "Show answer"

    When the app shows a small, bounded set of images that are known in advance, and the platform's own image view plus the HTTP cache already meets the frame budget. A settings screen with a dozen icons does not need a request pipeline, a two-level key or a bitmap pool.

    The measurement that decides it is frame timing on the busiest image screen with the naive implementation in place. If it is clean, everything below is code you will maintain forever to fix a flat graph. The instinct that a scrolling list always needs a library is wrong at small scale, and saying so is a point in your favour rather than a dodge.

```
+----------+   +--------------------+   +---------------+
| load(url,|-->| Memory cache, LRU  |-->| Deliver to    |
| target)  |   | sized in bytes     |   | target        |
+----------+   +--------------------+   +---------------+
                     | miss
                     |
              +----------------+
              | Fetch, decode  |
              | downsample, on |
              | a small pool   |
              +----------------+
```

**Caption.** V0 is four parts: an entry point taking a URL and a target, a byte-sized least-recently-used memory cache, a small worker pool that fetches and decodes at the target's size, and a delivery step. The cache key is the URL plus the requested size.

Why this is correct for a real app:

- Sizing the cache in bytes rather than entries is the one non-obvious decision, and it is free to make at the start and painful to retrofit.
- Decoding at the target size is what keeps the 10.4 MB figure from ever appearing.
- The pool is bounded, so a fling cannot start a hundred decodes.
- There is no disk layer, no transformation, no priority and no extensibility, and for many apps none of them is ever needed.

!!! warning "When not to add a disk cache"

    If the images are small, the origin is fast, and the platform's HTTP cache already stores the encoded bytes, a second disk layer duplicates that storage and doubles your eviction bugs. The measurement that justifies your own disk tier is the share of loads that reach the network on a warm start, and the byte cost of those loads per session. If the platform cache is already answering most of them, spend the effort on the two-level key instead, because that is what the platform cache cannot do for you.

### Breaking point, then V1 (image loading library)

**What fails first, and it fails on the first fast fling: the wrong image appears.** A row is recycled while its load is in flight, the completion arrives, and the library writes an image into a target that now represents a different item. How you observe it: in debug builds, assert that the key attached to the delivered result equals the target's current key; in production, sample the same comparison and count mismatches. This bug is invisible to crash reporting and extremely visible to users.

**What fails second: duplicate work for the same image.** Two visible rows reference the same URL and you fetch and decode it twice. How you observe it: count network fetches per distinct key per session, which should be close to one.

**What fails third: every cold start re-downloads everything.** Without a disk tier, the memory cache dies with the process. How you observe it: bytes downloaded in the first ten seconds after a cold start, compared with the same window after a warm start.

V1 introduces identity, deduplication, and a real cache hierarchy:

```
+-----------+   +----------------+   +-----------------+
| Request   |-->| In-flight map  |-->| Memory cache    |
| with key  |   | join duplicates|   | decoded bitmaps |
+-----------+   +----------------+   +-----------------+
      |                                       | miss
      |                                       |
+-----------+   +----------------+   +-----------------+
| Target    |   | Priority queue |<--| Disk cache      |
| token     |   | visible first  |   | encoded bytes   |
+-----------+   +----------------+   +-----------------+
                        |                     | miss
                        |                     |
                 +--------------+   +-----------------+
                 | Decode pool  |<--| Network fetch   |
                 +--------------+   +-----------------+
```

**Caption.** V1 gives every request a key and every target a monotonically increasing token, joins duplicate in-flight requests on the key, and orders work in a priority queue so visible requests precede prefetch. Lookups walk memory, then disk for encoded bytes, then network, and results are delivered only if the target's token still matches.

The target token is the fix for the wrong-image bug, and it is worth stating precisely:

- Every target holds a token that increments on every new load call against it.
- A request captures the token at submission.
- On completion, if the target's current token differs from the captured one, the result is discarded rather than delivered.
- Discarding must also release the bitmap back to whoever owns it, or you have swapped a visual bug for a memory leak.

!!! warning "When not to build in-flight deduplication"

    Deduplication pays when the same key is genuinely requested concurrently, for example an avatar repeated down a comment thread. If your measured duplicate-fetch rate per session is under a few percent, the join map buys almost nothing and costs you a shared-state concurrency problem plus a hard question about what cancellation means when two callers share one fetch. Measure first; the answer varies enormously by product.

### Breaking point, then V2 (image loading library)

**What fails: the cache key stops meaning what it says.** The moment a caller asks for the same URL blurred, or circle-cropped, or at a different size, one key maps to several different pixel buffers. Whichever arrives first wins and every later request gets the wrong variant. How you observe it: a bug report saying an avatar is round in one screen and square in another, on the same build, intermittently.

**What also fails: memory pressure kills the process rather than the cache.** The library holds 64 MB happily until the app opens a camera, and then the OS reclaims the whole process. How you observe it: the ratio of background kills to foreground crashes, and whether kills correlate with image-heavy screens.

**And: prefetch starves visible work.** A prefetch of twenty upcoming rows occupies the pool while the row under the user's thumb waits. How you observe it: time to pixels for visible requests, segmented by whether prefetch was active.

V2 separates keys, adds ownership, and gives prefetch its own budget:

| Addition | Mechanism | Why it is the right shape |
|---|---|---|
| Two-level key | A source key identifies the encoded bytes on disk and is derived from the URL alone. A request key identifies the decoded bitmap in memory and is derived from the source key plus the target size, the scale mode and every transformation in order | The disk tier stores one copy of the original, and the memory tier stores as many variants as are in use. Collapsing them into one key is the single most common design failure here |
| Bitmap ownership rule | The library owns every bitmap it delivers, callers must not retain one past the target's next load, and a pool reuses buffers of matching dimensions | Reuse cuts allocation and collection pressure sharply, and it introduces a class of aliasing bugs that only an explicit ownership rule in the public documentation can prevent |
| Memory pressure hook | On a system trim signal, evict in tiers: first the pool, then non-visible cache entries, then everything except bitmaps currently attached to an attached target | An eviction that reclaims a bitmap being drawn is worse than the kill it was trying to avoid |
| Prefetch as a priority class | Prefetch requests never occupy more than a stated share of the pool, and are pre-empted by any visible request | Prefetch is speculative by definition, so it must always yield to work the user is waiting for |

!!! warning "When not to pool bitmaps"

    Pooling is worth it when you are allocating many bitmaps of identical dimensions repeatedly, which is exactly a fixed-height scrolling list and almost nothing else. In a viewer, or in a grid with varied sizes, the pool hit rate collapses and you have added an ownership contract, a recycle path and a family of use-after-recycle bugs for no measurable gain.

    The measurement that justifies it is allocation-attributed pause time during a fling. If it is a small fraction of your frame time, skip the pool and keep the simpler ownership story.

### Deep dive one: bitmap memory arithmetic and a cache size that is not a guess

Every other decision in this component is downstream of one equation, and most candidates never write it down.

**The equation.** Decoded bytes equals width in pixels times height in pixels times bytes per pixel. The file size does not appear in it.

| Pixel format | Bytes per pixel | 360 by 240 thumbnail | 1080 by 2400 full screen | What you give up |
|---|---|---|---|---|
| Four channels, eight bits each | 4 | 345 KB | 10.4 MB | Nothing. This is the default |
| Packed 16 bit colour without alpha | 2 | 173 KB | 5.2 MB | Visible banding on gradients, and no transparency. Acceptable for photographic thumbnails, wrong for illustrations |
| Platform-managed hardware buffer | 4, but outside the app's own heap | 345 KB off-heap | 10.4 MB off-heap | Direct pixel access, some transformation paths, and a portion of your ability to reason about it |
| Half-float wide gamut | 8 | 691 KB | 20.7 MB | Nothing visually. It doubles your memory, so it is a per-image decision, not a default |

The row that matters in interviews is the third. Moving bitmaps outside the app's own heap changes which limit you hit, not how much memory the device has. A candidate who says it "solves memory" has not thought about the device.

**Downsampling at decode, not after.** The correct sequence has three steps, and skipping the first is the classic out-of-memory bug:

1. Read the image header only, obtaining width, height and format, without allocating pixels.
2. Compute a sampling factor from the source dimensions and the target dimensions, respecting whatever granularity the platform decoder supports, which on one major platform is powers of two.
3. Decode with that factor, producing a bitmap at or slightly above the target size, and finish with an exact scale only if the residual error is visible.

Worked, with numbers: a 4,000 by 3,000 source for a 360 by 240 target. The ratio is 11.1 horizontally and 12.5 vertically. Taking the smaller power of two that still exceeds the target on both axes gives a factor of 8, producing 500 by 375, which is 750 KB rather than 48 MB. That is a factor of 64 saved by reading a header first.

**Sizing the memory cache, with the derivation.**

| Step | Value | Source |
|---|---|---|
| Heap ceiling on the reference device | 256 MB | Read at runtime, never hard-coded |
| Share taken by the image cache | 25 percent, so 64 MB | ASSUMED. A quarter of the heap is a common share for a component whose job is caching, and it leaves the app three quarters for everything else |
| Typical list thumbnail | 345 KB | From the equation above |
| Entries held | about 185 | 64 MB divided by 345 KB |
| Screens of scrollback covered | about 15 | 185 divided by 12 visible rows |

Fifteen screens is a defensible answer to "how big should the cache be", because it is expressed in the unit the user experiences: how far back can they scroll before an image reloads. A cache sized in megabytes answers a question nobody asked.

**Where to spend the next hour of engineering time.** The expected-time rows in the estimation table gave 68.9 ms at the baseline, 51.7 ms for a ten point memory hit gain, and 38.5 ms for a twenty point disk hit gain. The conclusion generalises:

- The memory tier removes a 20 ms disk read.
- The disk tier removes a 400 ms network call.
- Therefore, until your disk hit rate is high, every hour spent tuning the memory cache is worth roughly a twentieth of an hour spent on the disk cache.

That is the kind of sentence that changes a roadmap, and it comes from three multiplications.

**Memory pressure, in tiers.** When the system asks for memory back, the order is not arbitrary:

| Tier | What is released | Why it is safe |
|---|---|---|
| 1 | The bitmap pool | Pure optimisation, nothing is displaying it |
| 2 | Cache entries whose targets are detached | The user is not looking at them, and a reload costs a disk read |
| 3 | Cache entries for the currently visible screen | Almost never correct; a reload here is a visible flash |
| Never | Bitmaps attached to an attached, drawing target | Releasing one produces a blank or, worse, a recycled buffer being drawn |

!!! warning "When not to cache decoded bitmaps at all"

    If every image on screen is displayed exactly once and never returns, for example a one-shot onboarding illustration or a full-screen photo viewer that always moves forward, a decoded cache holds memory that will never be hit. The measurement that justifies it is memory-tier hit rate. Below about 20 percent, delete the tier, keep the disk tier, and give the memory back to the app.

### Deep dive two: the public surface, the cancellation contract and the threading contract

This dive is deliberately not a data path. In a component round the contract is the deliverable, and every dive on this page differs from the others because the prompt differs: case study 5 dived on concurrency and energy, case study 6 on measurement and durability, and this one on an interface, because that is what a library is. If an interviewer asked for two data-path dives here, I would push back and say the interesting risk lives in the surface.

**The surface, as a small closed set of concepts.**

| Concept | What it is | Why it must be public |
|---|---|---|
| Request | An immutable description: source, target size, scale mode, transformations, priority, placeholder, error image | Callers need to build one without a target, for prefetch |
| Target | Somewhere the result goes, with a token and a lifecycle the library can observe | Cancellation is derived from the target's lifecycle, so a target cannot be an opaque callback |
| Key | The two-level identity described in deep dive one | Callers must be able to invalidate a specific variant, which means they must be able to name it |
| Fetcher | Bytes for a source, given a request | Substituting the transport is the most common integration need |
| Decoder | A bitmap for bytes, given a request | Format support is where teams need to extend without forking |
| Transformation | A pure function from bitmap to bitmap, contributing a stable identifier to the key | If a transformation cannot name itself, it cannot be cached correctly. This is the requirement people forget |
| Disposable | The handle a load returns | Callers who use an unusual target need a way to cancel manually |

**The cancellation contract, layer by layer.** "Cancel" means four different things depending on where the request is, and a library that documents only one of them will be misused.

| Where the request is | What cancel guarantees | What it does not guarantee |
|---|---|---|
| Queued, not started | It never starts, immediately | Nothing else needed |
| Network in flight | The socket is released within the stated 50 ms | That no bytes were transferred, or that the origin was not charged for the request |
| Decoding | The decode is abandoned at the next checkpoint | Immediate return of the thread, because a platform decoder call is often not interruptible mid-frame |
| Delivering | The result is discarded and any bitmap is returned to the pool | That no work was done. The work is complete; you are throwing it away |

Two consequences worth stating in the room:

- Partially downloaded bytes should be written to the disk cache when the transport supports resumption, because a cancelled load during a fling is often re-requested seconds later when the user scrolls back.
- A cancelled decode that has already completed should still populate the memory cache. Discarding a finished bitmap because the target went away wastes the most expensive step you performed.

**The threading contract, stated as rules rather than as prose.**

- The entry point never blocks the calling thread, and never performs input or output on it. Its work is a key computation, a cache probe and an enqueue, budgeted at under 1 ms.
- A memory-tier hit may deliver synchronously, on the calling thread, within the same frame. This is the fast path that makes scrolling smooth, and losing it to a uniform "always asynchronous" rule is a real regression.
- Every other delivery happens on the main thread, and the library states which one, because a caller mutating a view from a background thread is a crash the caller will blame on you.
- Decoders and fetchers run on library-owned pools whose sizes are documented and configurable. The default fetch concurrency is four, derived in the estimation table from bandwidth rather than chosen for elegance.
- The caller may supply the dispatchers. This exists for testability more than for flexibility: a test that cannot make the library synchronous is a flaky test.

**The error model.** Returning a single generic failure object pushes classification onto every caller, and every caller will do it differently and wrongly.

| Failure kind | Retryable | What a caller should do |
|---|---|---|
| Source not found, permanently | No | Show the error image, do not retry, and do not log an error every time the row is bound |
| Transport failure | Yes, with backoff | Retry when connectivity returns |
| Decode failure on well-formed input | No | Show the error image and report it, because it usually means an unsupported format |
| Decode refused, image exceeds the configured pixel ceiling | No | This is a defence, not a bug. Report the dimensions |
| Cancelled | Not applicable | Do nothing at all, and above all do not report it as an error, because during a fling it is the majority outcome |

That final row prevents a specific real incident: a library that reports cancellations as errors turns one fast scroll into hundreds of error events, which is a self-inflicted denial of service on your own analytics pipeline. That is the same overhead problem case study 8 is entirely about.

**Extensibility without breaking compatibility.**

| Mechanism | Who implements it | Compatibility consequence |
|---|---|---|
| A registry mapping a source type to a fetcher | The caller implements, the library dispatches | You can add built-in types forever. This is the safe direction |
| An interface the caller implements | The caller | Every method you add later breaks every implementation. Add methods with defaults or not at all |
| A configuration builder | The library | Safe to extend, and the reason builders outlive constructors in long-lived libraries |
| A public concrete class with public fields | The library | Effectively frozen on release. Avoid |

**Initialisation cost.** The library must do nothing at process start beyond registering a lazily created singleton. Reading the disk cache index, probing free space or opening a database at start is a cold-start regression that no product team will ever attribute to the image library, which means it will never be fixed. Under the 5 ms budget, the honest implementation does its index read on first use, on a background thread, with lookups queueing behind it.

!!! warning "When not to make any of this pluggable"

    For an in-house component with one caller, every extension point above is a cost with no benefit: more code, more indirection, and a surface you cannot change once a second team adopts it. The measurement that justifies pluggability is the number of distinct teams integrating and the number of source types they need. With one team and one type, expose a single function and keep the internals free to change, which is worth more than any extension point.

### Failure modes and evaluation (image loading library)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Identity failure | The wrong image appears in a recycled row after a fast fling | Target tokens captured at submission and re-checked at delivery, plus a debug assertion and a sampled production counter |
| Out-of-memory kill | The process dies on an image-heavy screen, often after the user opened another app | Decode-time downsampling, a byte-sized cache, a pixel ceiling per image, and tiered response to trim signals |
| Decode bomb | A hostile or accidental 30,000 pixel image arrives and decoding it is a kill | Read the header first and refuse anything above a configured pixel ceiling, as a defence rather than a bug |
| Thundering herd | A cache is cleared, or a build invalidates keys, and every visible row requests the origin at once | Bounded fetch concurrency plus in-flight join, which together turn a herd into a queue |
| Cache stampede | Many requests miss the same key simultaneously before any populates it | The in-flight map is the fix, and it must be checked before the queue, not after |
| Priority inversion | Prefetch of twenty upcoming rows starves the row under the user's thumb | Prefetch capped as a share of the pool and pre-empted by visible work |
| Metastable queue growth | A fast fling enqueues faster than the pool drains, and the backlog never clears | A bounded queue that drops the oldest non-visible request, since a request for a row the user passed 10 seconds ago has no value |
| Disk corruption | A truncated cache entry decodes to garbage or throws on every load | Length-prefixed entries with a checksum, and a corrupt entry is deleted and refetched rather than surfaced |
| Self-inflicted telemetry flood | Cancellations reported as errors turn one fling into hundreds of events | Cancellation is not an error, stated in the error model and enforced by a test |

Service level indicators to publish:

- Time to pixels at p50 and p95, segmented by answering tier, because one blended number hides everything interesting.
- Memory-tier and disk-tier hit rates, tracked separately, since the arithmetic above says they are worth very different amounts.
- Decode time at p95 by source dimensions, which is how you catch an origin that started serving larger images.
- Wrong-image rate from the key comparison, which should be exactly zero.
- Out-of-memory kill rate on image-heavy screens, compared against the app baseline.
- Bytes downloaded per session attributable to images, split by cold and warm start.
- Frame drop rate during a fling on the busiest list, which is the number the user actually feels.
- Library initialisation time at p95, because this is the metric that protects you from being blamed for cold start.

Alert on: any non-zero wrong-image rate; disk hit rate falling after a release, which usually means a key changed and silently invalidated the whole cache; and decode time at p95 rising, which usually means the origin changed, not your code.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Memory tier inside one frame | Yes, because a memory hit is permitted to deliver synchronously on the calling thread |
| Disk tier under 100 ms | Yes on the assumed 20 ms read and decode, with generous margin. The assumption is the weak part and should be measured per device tier |
| Zero main thread decodes | Yes, by rule, and the 180 ms arithmetic is why the rule is not negotiable |
| Zero wrong images | Yes, by target token, and it is asserted in debug and sampled in production rather than assumed |
| Memory ceiling at 25 percent of heap | Yes, computed at runtime, plus tiered trim response. The pool and off-heap buffers must be counted or the ceiling is fiction |
| Cancellation within 50 ms | Yes for queued and network stages. Decode cancellation is best-effort and the contract says so instead of pretending |
| Under 1 ms of main thread work per call | Yes, provided key computation is cheap. A transformation whose identifier is expensive to compute breaks this quietly |
| Initialisation under 5 ms | Yes, with a lazy index read. This is the requirement most easily broken by a well-meaning later change |

### Where this answer is a simplification (image loading library)

- **Android developer documentation, "Loading Large Bitmaps Efficiently"** at https://developer.android.com/topic/performance/graphics/load-bitmap, checked 1 August 2026, describes exactly the header-first sequence used in deep dive one: decode with bounds only to read dimensions without allocating pixels, compute a sample size, then decode again. It notes that the sample size is a power of two and gives a worked example in which a 2048 by 1536 image at a factor of four occupies about 0.75 MB instead of about 12 MB, and it recommends using an established library rather than writing this yourself.
- **Android developer documentation, "Slow rendering"** at https://developer.android.com/topic/performance/vitals/frozen, checked 1 August 2026, is the source of the frame windows this case study budgets against, including the point that overrunning the window drops the frame rather than showing it late.
- **Colour management is missing here.** Wide-gamut and high dynamic range sources need a colour space decision at decode time, and getting it wrong is a subtle, hard-to-report bug where images look slightly wrong on some devices only.
- **Progressive and streaming decode** lets a large image appear at low quality quickly and sharpen. It changes the delivery contract from one result to several, which ripples through the entire public surface designed above.
- **Animated formats** need a frame scheduler, a memory model based on frame size times buffered frames, and a policy for what happens when several animations are visible at once. That is a subsystem, not a decoder.
- **Zoom and pan on very large images** needs tiling, which is closer to the maps client case study elsewhere on this page than to anything here.
- **The origin usually wins.** Serving pre-sized variants and a modern compression format from the origin removes more device work than any client optimisation in this case study. If you control the origin and are not doing this, do it before you tune a cache. The origin-side design is a content delivery problem covered in [modern-system-design.md](modern-system-design.md).
- **The component-versus-application framing** is worth studying in its own right, because the same split exists in web interviews. The component-scale treatment is in [frontend-system-design.md](frontend-system-design.md).

### Level delta (image loading library)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Draws a cache, a network layer and a decoder | Asks whether this is a library for other teams or an in-house component | Says in the first minute that the origin is treated as given and the deliverable is the on-device contract, then asks to be redirected if that is the wrong scope |
| Memory | "Images use a lot of memory, so we cache carefully" | Downsamples at decode and sizes the cache in bytes | Writes width times height times bytes per pixel on the board, derives 10.4 MB for one full-screen bitmap and 124 MB for twelve, then converts the cache size into screens of scrollback |
| Cache key | Uses the URL | Adds the target size to the key | Splits into a source key for encoded bytes on disk and a request key for the decoded variant in memory, and names transformation identity as a requirement on the transformation interface |
| Cancellation | Cancels on view recycle | Explains the recycled-view bug and fixes it with a tag | Captures a monotonic token at submission, re-checks at delivery, returns the discarded bitmap to the pool, and specifies what cancel guarantees at each of four layers |
| Where to optimise | "Make the cache bigger" | Tracks hit rates for both tiers | Shows that twenty points of disk hit rate is worth nearly twice ten points of memory hit rate, because only the disk tier removes a 400 ms network call |
| Concurrency | Uses a thread pool | Bounds it and prioritises visible work | Derives a fetch concurrency of four from bandwidth, gives prefetch its own capped share, and names priority inversion as the failure that appears without it |
| Public surface | Describes a load function | Lists the extension points | Classifies each extension point by whether the library or the caller implements it, and states which direction is safe to extend after release |
| Errors | Returns a generic failure | Distinguishes network from decode failures | Declares retryability per failure kind and rules that cancellation is not an error, then names the telemetry flood that follows from getting that one row wrong |
| Honesty | Claims the design prevents out-of-memory kills | Notes that large images are still a risk | Adds a pixel ceiling as an explicit defence, and states that off-heap bitmaps change which limit you hit rather than removing the limit |

What each level added:

- **Mid to senior:** the component acquires identity and bounds, and the cache stops being a dictionary and becomes a policy with a size derived from something.
- **Senior to staff:** the arithmetic decides where effort goes rather than intuition, the public contract is treated as the actual deliverable, and the failures that only appear at the boundary between library and caller are named before the interviewer finds them.

---

## Case study 8: Analytics SDK

The second component prompt on this page, and it is graded on a dimension the previous three were not: your code runs inside somebody else's app, so your crash is their crash, your cold start cost is their review score, and your battery drain is their uninstall. An analytics SDK is the clearest example in mobile of a component whose primary non-functional requirement is that nobody notices it.

The deep dives here are durability and statistics, deliberately different from case study 7's memory and contract dives, because the hard part of this component is not its surface (it is three methods) but what happens to a recorded event between the call and the warehouse.

### The prompt (analytics SDK)

"Design an analytics SDK that other teams embed in their apps to record product events."

### Questions that fork the design (analytics SDK)

| Question worth asking | If the answer is A the design becomes | If the answer is B the design becomes |
|---|---|---|
| First-party, inside apps you own, or third-party, inside customers' apps? | First-party: you can ship a heavier client, coordinate releases, and fix a bad version quickly | Third-party: the overhead budget is contractual, you cannot force an upgrade, and a remote kill switch stops being a nicety |
| Do these events drive billing or revenue reporting? | No, product analytics only: sampling and a stated loss bound are fine, and this changes almost every decision below | Yes: you need at-least-once delivery, server-side deduplication and a reconciliation report, and you may not sample at all |
| Is there a real-time dashboard requirement? | No: upload opportunistically on charge and unmetered networks, at almost no battery cost | Yes, within a minute: batches shrink, radio wake-ups multiply, and the battery budget becomes the binding constraint |
| Does the SDK ever see personal data? | No, and you can prove it with a schema allowlist | Yes: consent gating goes in front of the queue, redaction happens on the record path, and deletion by identifier becomes a design requirement rather than a policy document |
| Who defines the event schema? | The app team, free-form: fast to adopt, and the warehouse fills with unusable high-cardinality properties within a quarter | The SDK, from a generated schema: usable data, at the cost of code generation, versioning, and a slower path to instrumenting anything |
| Must identity survive a reinstall? | No: identity is per install, which is cheap, private, and loses returning-user analysis | Yes: you need a durable identifier, which is a privacy and platform-policy decision long before it is a technical one |
| May the SDK do work at process start? | Yes: simplest, and you own a slice of every cold start forever | No, lazy only: you protect cold start and lose the earliest events, which are the ones that describe launch |
| How many SDK versions are live at once? | Two: the wire format is changeable | Twenty across three years: the wire format is permanent, every event carries a schema version, and the server parses all of them until the last device dies |

### Requirements (analytics SDK)

Functional:

- Record an event with a name and typed properties, from any thread, without blocking the caller.
- Attach session, app, device and identity context automatically so callers do not repeat it.
- Persist events so that an event the SDK accepted survives process death.
- Batch and upload with retry, backoff and jitter, honouring server-directed pacing.
- Sample by event type, with the rate carried on every event so the warehouse can reweight.
- Gate everything behind a consent state that can change at any moment, including retroactively.
- Provide a debug mode that shows what would be sent, without sending it.
- Support deletion of a user's data by identifier.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Main thread time per record call | under 50 microseconds | ASSUMED; 50 microseconds is 0.6 percent of an 8 ms frame, so ten calls in one frame cost under 1 percent of it |
| Cold start cost added | under 5 ms | ASSUMED; measured against the published excessive cold-start threshold cited at the end of this case study |
| Steady-state memory | under 2 MB | ASSUMED |
| Disk ceiling | 20 MB | ASSUMED; large enough for days of backlog, small enough that no user notices |
| Battery cost | under 0.5 percent of a full charge per day | ASSUMED, and converted into a hard upload ceiling below |
| Events per user per day | 300 | ASSUMED; ask, because a video app and a banking app differ by an order of magnitude |
| Event size | about 400 bytes as text, about 120 bytes packed | ASSUMED |
| Delivery guarantee | at-least-once, loss under 0.5 percent, duplicates under 0.1 percent after server deduplication | ASSUMED |
| Crash safety | an event the SDK has accepted survives process death | GIVEN; without this the durable queue has no purpose |
| Binary size added | under 200 KB | ASSUMED; for a third-party SDK this is frequently the criterion that decides adoption |
| Crashes attributable to the SDK | zero | GIVEN. A crash in an analytics library is an outage in someone else's product |

### Estimation that eliminates (analytics SDK)

| Calculation | Result | What it rules out |
|---|---|---|
| 300 events per day times 400 bytes | 120 KB per device per day of raw text | Rules out the user's data plan as a concern, and sets up the next line |
| The same compressed at an ASSUMED 5 to 1, which is realistic for repetitive structured text | about 24 KB per device per day | Rules out sending packed binary as the first optimisation. Compression already beats it, and text is far easier to debug |
| One upload per event: 300 requests times 600 bytes of overhead | 180 KB of overhead against 24 KB of payload, so 7.5 times more overhead than content | Rules out per-event upload decisively. This one line is the entire argument for batching |
| Batching at 50 events: 6 uploads per day times 600 bytes | 3.6 KB of overhead, so 15 percent of payload | Rules out nothing yet, but it is the reference point |
| Batching at 20 events: 15 uploads times 600 bytes | 9 KB, so 37 percent of payload | Rules out batch sizes below about 20, since overhead then dominates the thing you are paying to send |
| Batching at 100 events: 3 uploads times 600 bytes | 1.8 KB, so 7 percent of payload | Rules out the idea that bigger is always better: past about 100 the return flattens while the loss window on a crash keeps growing |
| Battery: a 0.5 percent share of 15.4 Wh | 77 mWh per day | Sets the ceiling used in the next two lines |
| Six uploads per day at 7 J each of radio energy | 42 J, so 11.7 mWh, which is 15 percent of the budget | Rules out radio wake-ups as the dominant cost at this cadence, and rules out the reflex that analytics is a battery problem at low volume |
| Solving for the ceiling: 77 mWh divided by 1.94 mWh per upload | about 40 uploads per day | Rules out any design that flushes more than roughly every 36 minutes on average, which in turn rules out a naive 30 second flush timer, at 2,880 uploads per day |
| Central processing cost: 300 calls times 50 microseconds | 15 ms of CPU per device per day | Rules out the record path as a battery concern, which is the opposite of most people's instinct and worth saying |
| Write amplification with one sync per event, at an ASSUMED 4 KB minimum physical write | 300 syncs and 1.2 MB of physical writes for 120 KB of logical data | Rules out syncing per event. Group commit at 20 events gives 15 syncs a day for the same data |
| 10 million daily active users times 6 uploads | 60 million requests per day, so 694 per second average and about 2,100 at a 3 times peak | Rules out a fixed wall-clock upload schedule. Every device flushing at a local midnight is a self-inflicted thundering herd on a service you also own |
| Sampling: detecting a 1 percent relative change on a 10 percent base rate needs 16 times 0.1 times 0.9 divided by 0.001 squared | about 1.44 million observations per arm | Rules out nothing yet; it is the denominator for the next line |
| At 1 percent uniform sampling of 10 million users, you observe about 100,000 per arm, giving a detectable difference of the square root of (16 times 0.09 divided by 100,000) | 0.0038 absolute, so a 3.8 percent relative change | Rules out uniform 1 percent sampling for funnel work, and simultaneously rules out 100 percent sampling of high-volume events like scroll. The only answer left is per-event-type rates |
| A four-step funnel with independent 10 percent sampling at each step | 0.1 to the fourth power, so 1 in 10,000 complete funnels observed | Rules out sampling each event independently, by four orders of magnitude. Bucketing must be consistent per user, which is deep dive two |

### V0, the simplest thing that works (analytics SDK)

**Predict first: what fraction of events does an in-memory queue actually lose, and what would you measure to find out?**

??? note "Show answer"

    You cannot know without an independent counter, which is the real answer. The naive estimate is that you lose whatever is buffered when the process dies, so the loss rate is roughly the share of sessions that end in a background kill or a crash multiplied by the average fraction of a batch outstanding. At an ASSUMED 8 percent of sessions ending in a kill and half a batch outstanding on average, that is about 4 percent of events.

    The way to measure it rather than assume it is a monotonic sequence number per install: the server sees the gaps, and gaps are the only honest loss metric, because a lost event by definition cannot report itself.

```
+-----------+   +--------------------+   +----------------+
| record()  |-->| In-memory queue    |-->| POST batch     |
| any thread|   | flush at 50 or on  |   | to collector   |
+-----------+   | background         |   +----------------+
                +--------------------+
```

**Caption.** V0 is three parts: a record call that appends to an in-memory queue from any thread, a flush trigger at 50 events or when the app goes to the background, and one batched upload. Nothing is written to disk and nothing survives the process.

Why this is correct for many products:

- It satisfies the 50 microsecond and 5 ms budgets trivially, because it does almost nothing.
- Six uploads a day is 15 percent of the battery budget, so the energy question is already answered.
- A product analytics use case that tolerates a few percent loss gets accurate rates, because loss is roughly uniform across event types.
- There is no disk format to version, corrupt, migrate or fill.

!!! warning "When not to add durability"

    If your events measure rates and proportions rather than totals, and the loss is uncorrelated with the thing you are measuring, a durable queue buys precision you cannot use. The measurement that justifies it is the sequence-gap loss rate combined with a check for correlation: if loss is concentrated in the events immediately preceding a crash, it is not uniform, and every crash-adjacent funnel you analyse is biased. That correlation, not the raw loss rate, is what makes the disk layer necessary.

### Breaking point, then V1 (analytics SDK)

**What fails: loss is not uniform, it is exactly where you need the data.** The events immediately before a crash or a background kill are the ones an engineer investigating that crash wants, and they are precisely the ones an in-memory queue loses. How you observe it: compare the count of session-start events with session-end events, and separately plot the sequence-gap rate against time-to-process-death. A gap rate that spikes in the final seconds of a session is the signature.

**What also fails: consent arrives after the events do.** A user withdraws consent and you have a queue of events in memory with no policy for them.

**And: one badly behaved caller can take the app down.** A loop that records 50,000 events fills memory in seconds.

V1 makes the pipeline durable, sampled and gated:

```
+-----------+   +----------------+   +------------------+
| record()  |-->| Consent gate   |-->| Sample decision  |
| any thread|   | drop or allow  |   | per event type   |
+-----------+   +----------------+   +------------------+
                                              |
                                              |
+-----------+   +----------------+   +------------------+
| Uploader  |<--| Durable queue  |<--| Enrich, validate |
| batch, jit|   | group commit   |   | against schema   |
+-----------+   +----------------+   +------------------+
```

**Caption.** V1 puts four stages between the record call and the network: a consent gate that drops before anything is stored, a per-event-type sampling decision made at record time, enrichment and schema validation, and a durable append-only queue drained by a batching uploader with jittered backoff.

Two ordering decisions in that diagram are load-bearing:

- The consent gate is in front of the queue, not in front of the upload. If it sat in front of the upload, a withdrawal of consent would leave a disk full of data you must now delete, which is a worse problem than the one you solved.
- The sampling decision is made at record time, not at upload time. Deciding later means you paid the disk write for events you discard, and it makes the sampling rate ambiguous for any event that spans a configuration change.

!!! warning "When not to sample at all"

    If your daily event volume is small enough that full collection is affordable at every layer, sample nothing. Sampling costs you a rate to carry, a reweighting step in every query, a consistency requirement across funnel steps, and a permanent source of confusion when someone forgets to divide. The measurement that justifies sampling is warehouse cost per event type against the analytical value of that type. High-volume, low-value events such as scroll positions are worth sampling. Purchases never are.

### Breaking point, then V2 (analytics SDK)

**What fails: the collector is down for six hours.** Every device retries, the queue grows, and at 300 events a day the disk ceiling is not the immediate problem, but a chattier host app at 30,000 events a day fills 20 MB in under two days. Then the recovery is worse than the outage: every device in the world uploads its entire backlog the moment the service returns, which is a retry storm you built yourself.

How you observe it: queue depth at p99 across the fleet, and the ratio of upload attempts to successes over time. A ratio that climbs while depth climbs is a metastable failure, and the fleet will not recover without pacing.

**What also fails: a single malformed event blocks a batch forever.** The collector rejects the batch with a client error, the SDK retries the identical batch, and it is rejected again, permanently.

**And: property cardinality quietly destroys the warehouse.** One caller sets a property to a unique identifier, and a month later a single event type has 40 million distinct values in one column.

V2 adds five bounded mechanisms:

| Addition | Mechanism | Why it is the right shape |
|---|---|---|
| Bounded queue with a named drop policy | The queue has a hard byte ceiling. Events carry a priority class, and drops happen within the lowest class first | Unbounded queues do not fail gracefully, they fail as a disk-full dialog in someone else's app |
| The drop counter is itself an event | A critical-class event records how many were dropped, by class and by reason | Silent loss is the failure mode that makes every downstream number wrong without anyone knowing |
| Batch quarantine and bisection | A batch rejected with a client error is split. Halves that succeed proceed, and the single offending event is dropped with a counter | Turns a permanent block into one lost event plus a diagnostic |
| Server-directed pacing and a kill switch | The collector returns an upload interval and, if needed, a stop instruction. Recovery is spread over a window, not a moment | You cannot upgrade the installed base, so the only control you have over old versions is one they ask for |
| Schema allowlist with cardinality caps | Unknown event names and unknown properties are rejected at record time, with a counter. Properties exceeding a distinct-value cap are truncated to a bucket | Governance at the edge is the only governance that scales, because rejecting at the warehouse means you already paid to transport it |

!!! warning "When not to add a bounded queue with drop policies"

    If your host app records tens of events per session and uploads on every foreground, the queue never approaches its ceiling and the drop machinery is dead code that still has to be tested. The measurement that justifies it is queue depth at p99 across the fleet during your worst collector incident, which you should simulate rather than wait for. If depth never exceeds a few hundred events, keep a simple ceiling that drops the oldest and move on.

### Deep dive one: the durable queue, group commit, and what at-least-once actually costs

This dive uses the same mechanism as the order journal in case study 6 and configures it in the opposite direction, which is the clearest illustration on this page that a durability setting is a consequence of what one lost record costs, not a house style.

**Choosing the store.**

| Option | Crash safety | Delete-after-ack cost | Corruption recovery | Concurrency | Verdict |
|---|---|---|---|---|---|
| Append-only file with length-prefixed, checksummed records | Good, if you truncate a bad tail | Rewrite or compact, which is the weak point | Truncate at the first bad record | One writer, one reader | Best when events are uniform and deletion is by prefix |
| Embedded key-value store | Good | Cheap, delete by key | Depends on the engine | Usually good | Good, and it hides the durability knobs you actually want to control |
| Embedded relational database with a write-ahead log | Good, and configurable | Cheap, delete by identifier range | Mature and well documented | Excellent | Best default, because the durability setting is explicit and documented |

**The durability setting, stated as a decision rather than a default.** A write-ahead log can either sync on every commit or defer syncing, and the second option means a power loss can lose recent transactions in exchange for far fewer disk operations.

| Component | Setting | Reasoning |
|---|---|---|
| Order journal, case study 6 | Sync on every commit | One lost record is one order whose existence is unknowable. The cost of a sync is irrelevant against that |
| Analytics queue, this case study | Defer the sync | One lost record is one event out of hundreds, in a system that already tolerates 0.5 percent loss. Paying 300 syncs a day for it is a battery and wear cost with no analytical benefit |

Say that comparison out loud in an interview. It shows that you chose a setting rather than inherited one.

**Group commit, derived.** From the estimation table, one sync per event is 300 syncs and 1.2 MB of physical writes per day for 120 KB of logical data. Committing in groups of 20 gives 15 syncs a day.

The cost is the loss window: a crash loses up to 19 events instead of 1. Against a 0.5 percent loss budget and 300 events a day, 19 events is 6.3 percent of a day, which is why the group must also be flushed on a timer and on any lifecycle transition, not on the count alone.

**What record() actually promises.** This is a documentation decision that prevents an entire category of support tickets:

- record() returns after the event is in an in-memory buffer, not after it is on disk. It is non-blocking by contract.
- The buffer is committed to disk on a count trigger, a short timer, and every lifecycle transition that gives you warning.
- A caller who needs more than that calls flush(), which is explicitly documented as blocking and as something to call rarely.
- The SDK never promises that an event reaches the server. Nothing on a device can promise that.

**Crash recovery.**

| Situation | Detection | Response |
|---|---|---|
| Torn final record | Length prefix does not match the bytes present, or the checksum fails | Truncate at the last good record. Never fail the whole file |
| Whole-file corruption | The header or index is unreadable | Move the file aside, start a new one, and record a critical event with the size of what was lost |
| Clock moved backwards between runs | The stored monotonic sequence exceeds what the wall clock suggests | Trust the sequence, not the clock. Send both timestamps and let the server decide |

**At-least-once, and the duplicates it guarantees.** You must not delete an event before the server acknowledges it, so an acknowledgement lost on the return path produces a re-upload of events the server already has.

- At an ASSUMED 0.5 percent acknowledgement loss rate and 50 events per batch, about 0.5 percent of events arrive twice.
- Server-side deduplication on the pair (install identifier, sequence number) is a constant-time check against a bounded recent window, which is affordable at 694 requests per second.
- Exactly-once on the client is not available. The device cannot distinguish a lost request from a lost acknowledgement, which is the same ignorance as the trading client's unknown order outcome, with far lower stakes.

**Sequence numbers are the most valuable field in the payload.** A monotonic per-install sequence, never reused, gives you three things for four bytes:

- Deduplication, as above.
- A loss metric, because gaps that never fill in are events that never arrived. Nothing else can measure your loss, since a lost event cannot report its own absence.
- Ordering that survives a wrong device clock, which matters because device wall clocks are user-adjustable and frequently wrong.

**Backpressure and the drop policy, honestly.**

| Policy | What it biases toward | When it is right |
|---|---|---|
| Drop the newest | Preserves the start of the session, so funnels stay analysable | Product analytics, where funnel entry matters |
| Drop the oldest | Preserves the most recent events, which describe the current problem | Debugging and diagnostics, where the last minute is the interesting one |
| Drop by class | Preserves purchases and errors, drops verbose telemetry | Almost always correct, and it is why events carry a class at all |

The recommendation is classes first, then drop the newest within the verbose class, and never drop the critical class. If the critical class alone would breach the ceiling, that is not a drop policy problem, it is an instrumentation problem, and the SDK should say so with a counter rather than absorb it.

!!! warning "When not to build a durable queue"

    Covered in the V0 breaking point, and worth repeating with its measurement: the justification is not the loss rate, it is whether loss correlates with what you are measuring. Uniform loss of 4 percent barely affects a rate. Loss concentrated before crashes destroys crash analysis. Measure the correlation, not the average, and if there is none, keep the in-memory queue and spend the effort on schema quality instead.

### Deep dive two: sampling and privacy that keep the numbers usable

This dive is statistics and policy rather than mechanism, and it is the one place on this page where the correct design decision is decided by an error bar. It pairs with deep dive one the way measurement paired with durability in case study 6, but the subject is different: there, the question was whether an event happened; here, it is what a surviving event lets you conclude.

**The failure that motivates everything: independent sampling destroys funnels.** From the estimation table, four funnel steps each sampled independently at 10 percent yields complete funnels for 1 in 10,000 users rather than 1 in 10. The fix is not a bigger sample, it is a consistent one.

- Compute a bucket from a stable hash of the install identifier, once, at start.
- An event is kept if its bucket falls under the rate configured for its event type.
- Because the bucket is the same for every event from that install, a user who is in the sample is in it for every step of the funnel.
- The hash must include a per-event-type salt only when you deliberately want independent samples, which is rare and should be justified in writing.

**Rates are per event type, and the reason is arithmetic.**

| Event type | Volume per user per day | Suggested rate | Why |
|---|---|---|---|
| Purchase or other revenue event | under 1 | 100 percent | Sampling money is never worth the saving, and it makes reconciliation impossible |
| Error and crash-adjacent breadcrumbs | 2 | 100 percent | You need the rare ones, which are the entire point |
| Screen view | 40 | 25 percent | Common enough that a quarter still resolves small differences |
| Scroll or gesture telemetry | 200 | 1 to 5 percent | The highest volume and lowest per-event value on the list |
| Session start and end | 4 | 100 percent | These are the denominators of every other rate, so sampling them makes everything else harder |

**Every event carries the rate it was sampled at.** Without it, a rate change looks exactly like a traffic change, and someone will announce a 75 percent drop in engagement that is actually a configuration change. With it, the warehouse divides by the rate to get an estimate, and it can also compute an interval.

**The error bar, derived.** A count of k events observed at rate r estimates a true count of k divided by r. Treating the sampling as a binomial process, the standard error of that estimate is approximately the square root of k times (1 minus r), divided by r, which for a small r is close to the square root of k divided by r.

- 100 events observed at 1 percent estimate 10,000, with a standard error near 1,000.
- 10,000 events observed at 1 percent estimate 1 million, with a standard error near 10,000, which is 1 percent.
- Therefore reweighted counts below roughly a few hundred observed events must never be reported as a point number, and a dashboard that does so will manufacture false trends every week.

**Where the rate lives.** The rate must be server-controlled, versioned, and shipped as configuration rather than compiled into the SDK.

- Server-controlled, because you cannot upgrade the installed base to fix a rate that is too high.
- Versioned, because a query that spans a rate change must be able to weight each period correctly.
- Never changed silently, because the warehouse needs the change as data, not as a note in someone's calendar.

**Privacy, as a set of placements rather than a policy statement.**

| Decision | Placement | Why here and not elsewhere |
|---|---|---|
| Consent gate | In front of the queue | Nothing is stored that a withdrawal would force you to delete |
| Property allowlist | On the record path | Free-form string properties are where personal data leaks, and the leak must be stopped before it is durable |
| Redaction of known patterns | On the record path | Same reason. Redacting at upload leaves the raw value on the user's disk |
| Identity resolution | Server side | The device should not hold a mapping between identities it does not need |
| Deletion by identifier | Server side, keyed by whatever the SDK sends | You cannot delete by an identifier you never stored, which is a real tension: privacy-minimising identity makes deletion requests harder to honour |

**Identifier tiers, and what each one buys and costs.**

| Tier | Survives | Enables | Costs |
|---|---|---|---|
| Session | Nothing | Within-session funnels | No returning-user analysis at all |
| Install | App restarts and reboots | Retention and cohort analysis for that install | Resets on reinstall, which understates retention |
| Account | Reinstall and device change | True cross-device analysis | It is personal data, with everything that follows |
| Device-level advertising identifier | Reinstall | Attribution across apps | Platform policy, permission prompts, and a user-visible reset control |

The honest interview answer is that install-scoped identity is the default, account-scoped is added only where the product genuinely needs cross-device analysis, and anything device-level is a business decision with a legal owner who is not you.

**On-device aggregation, the option nobody proposes.** For the highest-volume, lowest-value event types you can send counts per hour instead of events.

- Bytes fall by roughly the ratio of events to distinct counters, which for scroll telemetry can be a hundredfold.
- You lose sequencing, funnels, and any ability to debug a specific session.
- It is the right answer for a metric like "how many times was this control tapped today" and the wrong answer for anything you will ever want to investigate.

!!! warning "When not to enforce a schema at the SDK"

    A schema allowlist slows every new instrumentation down to the speed of a release train, which for a third-party SDK is the customer's release train, not yours. If your host app is small and one team owns all instrumentation, a documented convention plus a warehouse-side check is cheaper and nearly as effective.

    The measurement that justifies edge enforcement is the count of distinct event names and the maximum property cardinality in your warehouse. Past a few hundred names or a property with millions of distinct values, the convention has already failed.

### Failure modes and evaluation (analytics SDK)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Thundering herd | Every device flushes at the same local wall-clock time, or the whole fleet retries the instant an outage ends | Jittered intervals drawn from a window, plus server-directed pacing on recovery. Never a fixed clock trigger |
| Retry storm | Uploads fail, clients retry immediately, and the added load keeps the collector down | Exponential backoff with jitter, a cap on attempts before a long sleep, and honouring a server-supplied retry interval |
| Metastable failure | Queue depth grows faster than the drain rate and never recovers, because every device is spending its uploads on backlog | A bounded queue, a drop policy with counters, and a pacing instruction that lets the fleet drain over hours rather than seconds |
| Poison event | One malformed event causes a permanent client-error rejection, and the same batch retries forever | Quarantine and bisect the batch, drop the single offender, and count it |
| Silent loss | Events vanish and nothing reports it, because a lost event cannot report itself | Per-install monotonic sequence numbers, with loss computed server-side from gaps |
| Double counting | A configuration change to a sampling rate is not recorded, so reweighted numbers shift for no visible reason | The rate travels on every event, the rate configuration is versioned, and rate changes are themselves recorded |
| Unbounded cardinality | One property accumulates millions of distinct values and one warehouse table becomes unusable | Distinct-value caps applied at record time with bucketing, plus a rejected-property counter |
| Clock skew | Device timestamps are wrong, sometimes by years, and sorting by them scrambles sessions | Send the device wall clock, a monotonic uptime delta, and let the server stamp receipt time. Order by sequence, not by clock |
| Disk exhaustion | The queue fills the ceiling and the host app starts failing its own writes | A hard byte ceiling checked before every commit, with drops counted rather than the write attempted |
| Host app harm | The SDK crashes, or adds 200 ms to cold start, and the host team removes it | Every public entry point catches and counts rather than throwing, initialisation is lazy and off the main thread, and the SDK publishes its own cold-start contribution |

Service level indicators to publish, and note that half of them are about the host app rather than about the data:

- Sequence-gap loss rate per install, which is the only honest loss metric available.
- Duplicate rate after server deduplication.
- End-to-end event latency at p50 and p95, from record time to warehouse availability.
- Queue depth at p99 across the fleet, and the age of the oldest queued event.
- Drop counts by class and by reason, published as a first-class metric rather than a log line.
- SDK-attributed cold start contribution at p95, measured by the host app with and without initialisation.
- SDK-attributed crash rate, which must be indistinguishable from zero.
- Uploads and bytes per device day, which is the battery proxy the estimation table is built on.
- Rejected event and property counts by rule, which is how you find an instrumentation mistake in days rather than quarters.

Alert on: any SDK-attributed crash; drop counts crossing a stated floor; loss rate rising after an SDK release; and queue depth trending upward across the fleet, which is the early warning for a collector problem that has not yet paged anyone.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Under 50 microseconds on the main thread per call | Yes, if enrichment and validation happen off the calling thread. Doing schema validation inline breaks this quietly, which is why it sits after the queue boundary in the diagram |
| Under 5 ms of cold start | Yes, with lazy initialisation and a background index read. This is the requirement most likely to regress silently, so it is published as an indicator |
| Under 0.5 percent battery per day | Yes, at 6 uploads for 15 percent of the budget, with the derived ceiling of about 40 uploads per day as the guardrail |
| Loss under 0.5 percent | Yes on the durable path, and measured rather than assumed, since gaps are visible server-side |
| Duplicates under 0.1 percent | Only after server-side deduplication. The client alone delivers about 0.5 percent, and saying so is more useful than claiming otherwise |
| 20 MB disk ceiling | Yes, enforced before every commit, with drops counted |
| Crash safety for accepted events | Yes, within the group commit window, which is bounded by a timer and by lifecycle transitions as well as by count |
| Zero SDK-attributed crashes | This is a discipline, not a mechanism: catch at every entry point, count, and never throw into a host app that did not ask for it |

### Where this answer is a simplification (analytics SDK)

- **SQLite, "Write-Ahead Logging"** at https://sqlite.org/wal.html, checked 1 August 2026, documents the durability setting this dive turns into a decision: with synchronous set to NORMAL the log is not synced on every commit, so a power loss can lose recent transactions, while FULL syncs on commit. It also describes automatic checkpointing at roughly a thousand pages and warns that continuous readers can starve a checkpoint, which is a real hazard for a queue that is read constantly by an uploader.
- **Android developer documentation, "Task Scheduling", the persistent work guide** at https://developer.android.com/develop/background-work/background-tasks/persistent, checked 1 August 2026, describes the sanctioned way to run work that survives process exit and reboot, backed by an on-device database, with configurable exponential backoff. That is the mechanism the uploader should be built on rather than a custom timer, and the page is explicit that it is not intended for in-process work that can safely be dropped.
- **Android developer documentation, "App startup time"** at https://developer.android.com/topic/performance/vitals/launch-time, checked 1 August 2026, publishes the thresholds beyond which cold, warm and hot starts are treated as excessive: 5 seconds, 2 seconds and 1.5 seconds. It also separates time to initial display from time to full display, which is the distinction to use when arguing about how much of a cold start an SDK may consume.
- **Apple developer documentation, "Background Tasks"** at https://developer.apple.com/documentation/backgroundtasks, checked 1 August 2026, is the reference for the equivalent deferred scheduling on that platform, and the two platforms do not offer the same guarantees. Read both before promising a host team an upload cadence.
- **Consent frameworks vary by jurisdiction and by platform**, and an SDK that ships worldwide inherits all of them. Treat consent as an input with several states rather than a boolean, and expect that at least one market requires a different default.
- **Instrumentation quality dominates transport quality.** The most common reason analytics data is unusable is not loss or duplication, it is that two teams named the same concept differently. That is a governance problem, and the schema allowlist above is only the enforcement end of it.
- **Multiple processes, app extensions and embedded web views** all record events and all have their own lifecycles. A single-process design is a teaching simplification that a good interviewer will probe within ten minutes.
- **The server side of this system** is a high-volume ingestion and aggregation pipeline with late events and deduplication, which is the metrics pipeline and ad click aggregation territory of [modern-system-design.md](modern-system-design.md).
- **Experiment assignment is a different component** with a much stricter consistency requirement than analytics, even though it is often bundled into the same SDK. Deterministic bucketing for experiments is treated separately in the feature flag SDK case study elsewhere on this page.

### Level delta (analytics SDK)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Draws a queue and an uploader | Asks whether the events drive billing, because that decides the delivery guarantee | Asks whether this runs in someone else's app, and reframes the whole design around the fact that the primary requirement is being unnoticeable |
| Batching | "We batch to save battery and data" | Picks a batch size and a flush trigger | Shows that per-event upload is 7.5 times more overhead than payload, that batch sizes below 20 are dominated by overhead, and that the returns flatten past 100 while the crash loss window keeps growing |
| Battery | Asserts that uploads cost battery | Reduces the number of uploads | Converts 0.5 percent of a 15.4 Wh cell into 77 mWh, prices one upload at about 1.94 mWh, and derives a hard ceiling of about 40 uploads per day that eliminates a 30 second flush timer outright |
| Durability | Adds a database | Explains group commit | States the durability setting as a decision, contrasts it with the order journal in case study 6, and prices the loss window at 19 events against a 0.5 percent budget |
| Loss measurement | "We assume delivery is reliable" | Compares counts of paired events | Adds a monotonic per-install sequence and notes that gaps are the only honest loss metric, because a lost event cannot report its own absence |
| Sampling | Samples everything at one rate | Uses different rates per event type | Shows that independent per-event sampling yields 1 in 10,000 complete four-step funnels, requires consistent bucketing from a stable hash, and puts the rate on every event so reweighting is possible |
| Reporting sampled data | Divides by the rate | Notes that small counts are noisy | Gives the standard error as the square root of the observed count divided by the rate, and rules that reweighted counts under a few hundred observations may not be reported as point numbers |
| Privacy | Mentions consent | Gates uploads on consent | Puts the gate in front of the queue so that a withdrawal does not leave data on disk, and names the real tension that deletion by identifier requires storing the identifier |
| Failure containment | Assumes the collector is up | Adds retry with backoff | Adds batch bisection for poison events, server-directed pacing so the fleet drains over hours, and a kill switch, on the grounds that the installed base cannot be upgraded |
| Honesty | Claims exactly-once delivery | Says at-least-once with server deduplication | States that the client alone delivers about 0.5 percent duplicates, that exactly-once is unavailable on a device, and that the same ignorance appears in case study 6 with much higher stakes |

What each level added:

- **Mid to senior:** the pipeline acquires stages with a defensible order, and delivery becomes a stated guarantee rather than an assumption.
- **Senior to staff:** the battery budget and the statistics both produce hard ceilings that eliminate designs before they are drawn, the SDK's obligations to its host app are treated as first-class requirements, and the limits of what a device can promise are stated rather than glossed.

## 10. Practice problems

Two sets, and they are not interchangeable. The blocked set rehearses the arithmetic and the moves you have just watched, with the topic already named. The interleaved set hides the topic, so your first job is to work out which budget the prompt is really about. That first job is the one a mobile design round scores.

Work everything on paper, or out loud with a timer, before opening an answer. Rubrics, model answers and diagnosed wrong answers for all eleven problems are in Section 11.

### 10a. Blocked set: five problems, same shape as the worked examples

**B1. Split a cold start budget and find what to cut.**

Product wants p90 cold start, measured from process creation to a usable first screen, under 1,200 ms on your mid-tier reference device. Cold means after a force stop or a reboot, not a warm resume.

Measured on that device, with the startup trace:

| Phase | p90 |
|---|---|
| Process creation plus runtime init | 260 ms |
| Class loading and dependency graph construction | 240 ms |
| Four SDK initialisers on the main thread | 60, 40, 90, 30 ms |
| Config file read from disk on the main thread | 180 ms |
| First layout and first frame | 160 ms |
| Blocking feed fetch before the screen is usable | 900 ms |

State the total, name the two designs the arithmetic eliminates, and give the ordered fix list with the number each step buys.

??? note "Show answer"

    Total, serial: 260 plus 240 plus 220 (the four initialisers sum to 60 plus 40 plus 90 plus 30) plus 180 plus 160 plus 900 equals 1,960 ms. You are 760 ms over a 1,200 ms target.

    On-device work, excluding the network: 260 plus 240 plus 220 plus 180 plus 160 equals 1,060 ms.

    Eliminated, one: "make the API faster" as the primary fix. Even at a network cost of zero you are at 1,060 ms against a 1,200 ms target, so a perfect backend leaves 140 ms of margin on the reference device and less on the tier below it. Server latency is worth fixing, but it cannot be the plan.

    Eliminated, two: treating the feed fetch as part of startup at all. The 900 ms has to leave the critical path, which means the first usable screen is rendered from the previous session's cached feed and the network call becomes a refresh behind it. This is a product conversation as much as an engineering one, because it changes what the user sees first.

    Ordered fixes, cheapest and most reversible first:

    | Step | Buys | Running total |
    |---|---|---|
    | Render from the cached feed, refresh behind it | 900 ms | 1,060 ms |
    | Defer three of four SDK initialisers past first frame; only the crash reporter (60 ms) must run first | 160 ms | 900 ms |
    | Move the config read off the main thread and off the startup path | 180 ms | 720 ms |

    Why 720 ms and not 1,199 ms: assume the low tier runs roughly 1.6 times slower than the mid tier on the same work, which is the order of magnitude you should confirm by running the same trace on your own low-tier device. 720 times 1.6 is 1,152 ms, still inside 1,200. A design that only just clears the target on the reference device fails on the device most of your users hold.

    How you would observe it: p90 of the platform's own startup trace, segmented by device tier and by cold against warm, with a regression gate in the release pipeline. A mean cold start number hides exactly the tail this exercise is about.

**B2. Fit list binding into a frame budget.**

Your feed scrolls on a device whose display runs at 90 Hz. Measured on that device: binding one list item costs 2.4 ms on the main thread, and during a fast fling three new items are bound in a single frame. Layout and draw of the whole visible list costs 4.5 ms.

Give the frame budget, say what happens, and give the change that fixes it.

??? note "Show answer"

    Frame budget: 1000 divided by 90 equals 11.1 ms. At 60 Hz it would be 16.7 ms and at 120 Hz it would be 8.3 ms, which is why the budget is stated at the device's actual refresh rate rather than at 60.

    You do not own the whole 11.1 ms. Assume you own about 70 percent of it, with the rest going to input handling, the system compositor and the render thread. That assumption is worth stating out loud and measuring; the conclusion below survives anything above roughly 60 percent. Your share is about 7.8 ms.

    What happens: three items at 2.4 ms is 7.2 ms, plus 4.5 ms of layout and draw, equals 11.7 ms of main thread work against 7.8 ms of budget. Every fling that pulls three items into view drops a frame. Even one item is 2.4 plus 4.5 equals 6.9 ms, which leaves 0.9 ms of margin.

    Eliminated: micro-optimising the bind. Making bind 10 percent faster takes three items from 7.2 to 6.5 ms, still over budget with layout included. A 10 percent win on the wrong number is not a fix.

    The change: bind must become a set of view writes and nothing else. Formatting, date arithmetic, string concatenation, text measurement and any parsing move to a background thread when the page arrives, and the result is stored on the item model. Assume that drops bind to 0.6 ms, which you verify rather than assume. Three items then cost 1.8 plus 4.5 equals 6.3 ms, inside 7.8.

    Second change, independent of the first: prefetch ahead of the scroll so at most one item is bound per frame in the common case. That gives 0.6 plus 4.5 equals 5.1 ms and leaves headroom for the frame where a bitmap also arrives.

    The sentence that reads as senior: "this bug only exists on the 90 Hz device, because the same code has 16.7 ms at 60 Hz and never misses." Frame budgets are the one place where a newer, more expensive phone is the harder target.

**B3. Bytes per screen on a slow network.**

A feed screen shows 8 cards. Each card carries about 1.2 KB of JSON and one image the server currently returns at 220 KB. The client prefetches two screens ahead. The target network is 400 kbps effective downlink with a 300 ms round trip.

Compute time to a complete screen, and decide what changes.

??? note "Show answer"

    One screen: 8 times 1.2 KB equals 9.6 KB of JSON, plus 8 times 220 KB equals 1,760 KB of images. Call it 1,770 KB, about 1.77 MB.

    Time: 1.77 MB times 8 equals 14.2 Mbit. At 0.4 Mbit per second that is 35.4 seconds for one screen. With two screens prefetched you have 5.31 MB in flight, about 106 seconds.

    Eliminated, one: prefetching two screens ahead on this network. The prefetch competes for the same 400 kbps as the screen the user is looking at, so it makes the visible screen slower in exchange for content the user may never reach.

    Eliminated, two: shipping one image size to every device. A card image displayed at 360 by 200 density-independent pixels on a 3x device is 1,080 by 600 actual pixels.

    Assume 0.12 bytes per pixel at acceptable quality with a modern lossy codec. That is an assumption you must measure on your own imagery, because photographic and graphic content compress very differently.

    1,080 times 600 times 0.12 equals about 78 KB. Eight of those is 624 KB, which is 5.0 Mbit, about 12.5 seconds. That is a 2.8 times improvement for a server-side change with no client release.

    What the ordering buys you. Fetch the JSON first: 9.6 KB is 0.077 Mbit, about 0.19 seconds of transfer plus one 300 ms round trip, so a text-complete screen inside about half a second. Then request images in view order, and cancel any request whose view has scrolled off and been recycled.

    The prefetch policy that survives contact with this network: prefetch the next two images, not the next two screens, and only once the visible screen's requests have completed. On an unmetered fast connection you can raise both numbers, which means the prefetch depth is a remote-config value and not a constant.

**B4. Size the image cache tiers.**

Your process gets about 256 MB on the mid-tier device before the operating system starts reclaiming it. You budget one eighth of that for decoded bitmaps. Card images display at 1,080 by 600 pixels in 32-bit colour, and the source images on the server are 4,032 by 3,024.

How many images fit in memory, what does that force, and what is the trap in the source resolution?

??? note "Show answer"

    Memory budget: 256 divided by 8 equals 32 MB. One eighth is a starting allocation, not a law; the number that matters is that you picked a fraction deliberately and can say why.

    Decoded size per card image: 1,080 times 600 times 4 bytes equals 2,592,000 bytes, about 2.59 MB. Note that this is 33 times the 78 KB that crossed the network in B3. Encoded size and decoded size are different quantities and confusing them is the single most common mobile memory bug.

    Capacity: 32 MB divided by 2.59 MB equals 12 images, which is about one and a half screens of 8 cards.

    What that forces: the memory tier cannot survive a scroll back of more than roughly a screen and a half, so a disk tier is not optional. Store encoded bytes on disk, not decoded bitmaps. 200 MB of disk at 78 KB per image holds about 2,564 images, roughly 320 screens. The tax is decode time on a disk hit; assume 8 ms per image on the mid tier, measured, which is inside the 11.1 ms frame budget from B2 only if the decode happens off the main thread.

    The trap: decoding the source at full resolution. 4,032 times 3,024 times 4 bytes equals 48,771,072 bytes, about 48.8 MB, for an image that occupies 2.59 MB once drawn. One such decode is one and a half times your entire bitmap budget, and two concurrent ones will get the process killed. Downsample during decode by asking the decoder for a sample size that lands at or just above the display size, and cap the largest dimension you will ever decode.

    Eliminated: a memory-only cache, and full-resolution decode. Both are eliminated by arithmetic, not by taste.

**B5. Choose a transport for a chat client.**

10 million daily active devices. Messages must appear within 2 seconds while the app is in the foreground, and within about a minute while it is backgrounded. Assume a cellular radio holds a high-power state for roughly 8 seconds after a transfer completes. That tail is real, it is carrier and device specific, and the design conclusion below holds for any tail above about 2 seconds.

Compare a 30 second poll, a persistent socket and push, on server load and on radio time. Then state what you would ship.

??? note "Show answer"

    Poll at 30 seconds, server side: 86,400 divided by 30 equals 2,880 polls per device per day. Across 10 million devices that is 2.88e10 requests per day, which is 2.88e10 divided by 86,400 equals about 333,000 requests per second, sustained, the overwhelming majority returning nothing.

    Poll at 30 seconds, device side: 2,880 wakeups times 8 seconds of radio tail equals 23,040 seconds, which is 6.4 hours of elevated radio state per device per day. It also misses the 2 second foreground requirement by up to 30 seconds.

    Eliminated: polling as the primary transport, on two independent grounds. Either one is enough; having both is what makes the elimination safe to state quickly.

    The figure that actually decides the transport is messages per connection per hour. Assume an active conversation delivers 30 messages an hour to a device. Polling every 30 seconds spends 120 requests an hour to carry 30 messages, so three quarters of the requests are empty. Below roughly 1 message per connection per hour, polling wins outright because a held connection and its heartbeats cost more than the occasional request. Above roughly 10 an hour, a held connection wins. Between the two, the deciding factor is latency, not cost.

    Persistent socket, foreground: one connection per foregrounded device, with a heartbeat every 30 to 60 seconds so that carrier and corporate middleboxes do not silently drop an idle connection. It meets the 2 second requirement with room.

    Sizing it: assume peak foreground concurrency is 8 percent of daily actives, a ratio you should replace with your own measurement. That is 800,000 concurrent sockets, and at an assumed 10 KB of kernel plus application state per connection it is 8 GB across the fleet, which is a handful of machines rather than a design problem.

    The real costs are the reconnect storm after a server restart, and the fact that a heartbeat while backgrounded pays the same radio tail as a poll.

    Push, backgrounded: the operating system will not let you hold a socket open indefinitely in the background, and push is the only transport with an OS-level guarantee that you get woken. It is best effort. Design it as a signal to sync, never as the message carrier, because a dropped or collapsed push must not lose a message.

    What I would ship: socket in the foreground, push as the background wakeup signal, one delta sync on resume that reconciles whatever push dropped, and a low-frequency poll as the safety net for the case where push has been silent.

    Pricing the safety net: one poll every 15 minutes is 96 polls per device per day, so 10e6 times 96 divided by 86,400 equals about 11,000 requests per second across the fleet, thirty times cheaper than the 30 second poll. On the device it is 96 times 8, so 768 seconds, about 13 minutes of radio tail a day.

### 10b. Interleaved and cumulative set: six problems, topic not stated

These are unlabelled on purpose. Decide which budget or which failure class applies before you start designing. Several reach back to material that is not on this page: the [trade-off atlas](index.md#the-trade-off-atlas) and the [failure library](index.md#the-failure-library) on the hub, the [Modern System Design course](modern-system-design.md) for the server side of every one of these prompts, the [ML System Design course](ml-system-design.md) where a model is involved, and the [system design question bank](/Interview-Questions/System-design/).

Twenty to thirty minutes each, out loud, with a timer running.

**I1.** A team member has proposed adding a full sync engine to our app: an operation log, tombstones, hybrid logical clocks and conflict resolution, so that the user's saved-items list works offline. The list holds at most a few hundred rows per user and only that user ever writes to it. Design it.

**I2.** Here is what ships today: a five-year-old app in one module, a REST API that returns the full object graph per screen, images served at source resolution straight from blob storage, and a release train that ships every two weeks with no staged rollout. p90 cold start on the mid tier moved from 1,300 ms to 1,820 ms after last quarter's merge of a partner SDK, and crash-free sessions fell from 99.7 percent to 99.4 percent. You have two quarters and no headcount increase. Plan it.

**I3.** Our messaging app sometimes shows a message twice, and sometimes shows two messages in the wrong order. It only happens to users who have the app on two devices. The transport is at-least-once and the message list is ordered by the timestamp the sending device wrote. Diagnose it and change the design.

**I4.** Product wants an on-device model that reorders the cached feed while the user scrolls, refreshed weekly. Design the client half: how the model reaches the device, where it runs, what the frame budget allows, and what happens on a device where it will not fit.

**I5.** Every release, our client error rate runs 5 to 8 times baseline for about two hours and then settles on its own. Nobody rolls back and nothing is obviously broken afterwards. Name the class and design the change.

**I6.** Our analytics SDK is embedded by 40 internal teams. Startup regressed by 90 ms across every app after we added a synchronous flush of the pending event queue during initialisation, and one team reports that our background upload starves their image prefetch on cellular. Redesign the SDK's contract.

!!! warning "Honesty note about the interleaved set"

    You will score lower here than on the blocked set, and it will feel like the reading did not take. The gap is the design. The blocked set measures whether you can execute a move once somebody has named it; the interleaved set measures whether you can pick one, which is the only part an interviewer can observe. A drop of one or two rubric levels between the two sets is the expected outcome, not a signal to reread the page.

---
## 11. Self-assessment rubrics

### The master rubric

Every problem in Section 10 is scored on the same four criteria, with named levels rather than numbers. A visible score crowds out the comment, and the comment is the part that changes your next answer.

| Criterion | Developing | Adequate | Strong | Exceptional |
|---|---|---|---|---|
| Problem framing and scoping | Starts drawing client layers before asking what the screen has to do. Restates the prompt as the requirements | Asks questions, but no answer would move a budget, a box or a byte count | Asks which device tier, which network and which refresh rate the target is stated against, and says how each answer forks the design | Names the budget nobody mentioned (memory ceiling, battery, bytes on a metered connection) and proposes the number to use until someone corrects it |
| Design and trade-off reasoning | One client architecture, presented as the answer. No alternative named | Alternatives named, chosen by platform habit or by what is fashionable | Each choice is settled by a number computed in front of the interviewer, and the rejected option is named with the condition under which it wins | Identifies the decision that is expensive to reverse on a client you cannot force-upgrade (wire contract, local schema, permission prompt) and sequences around it |
| Depth in at least one area | Box level everywhere. Names libraries instead of mechanisms | Goes one layer deeper when pushed | Volunteers one dive to implementation detail: decode and downsample, the outbox state machine, the reconnect ladder, the cursor's tie-break column | The dive includes how it fails silently on a user's device, what telemetry would show it, and what the fix costs given the release cadence |
| Communication and drive | Waits to be asked what comes next | Answers well, does not move the conversation | States the plan, keeps the clock, offers forks instead of asking permission | Absorbs a hostile follow-up by changing the design and naming which earlier claim is now void, including "that measurement was on the wrong tier" |

Score all four on every problem. Two Strongs and two Adequates is a passing senior answer. Four Adequates with nothing Strong is the profile most often written up as "hire, one level down", because nothing was memorable.

### Rubric notes, blocked set

??? note "B1 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks which device tier the 1,200 ms is stated against, and whether cold means after force stop or after reboot | Accepts "cold start" without asking what it is measured from |
    | Trade-offs | Separates on-device work from network before proposing anything | Proposes a faster API first |
    | Depth | Orders the fixes by cost and reversibility, and leaves headroom for the tier below | Lists fixes with no order and no arithmetic |
    | Communication | States the low-tier multiplier as an assumption and says it must be measured | Presents 720 ms as if it applied to every device |

    Model answer: the serial total is 1,960 ms against a 1,200 ms target. Splitting it, on-device work is 1,060 ms and the blocking fetch is 900 ms. That kills the API-latency theory immediately: a network of zero still leaves 1,060 ms, so the fetch has to leave the critical path rather than get faster.

    In order: render the previous session's cached feed and refresh behind it, which recovers 900 ms; defer three of the four SDK initialisers past first frame, keeping only the crash reporter, which recovers 160 ms; move the config read off the main thread, which recovers 180 ms. That lands at 720 ms.

    I want that much headroom because the low tier runs roughly 1.6 times slower on the same work, so 720 becomes about 1,152 ms there. A design that only just clears on the reference device fails on the device most users hold.

    I would gate this in the release pipeline on p90 by device tier, because a mean hides exactly the tail we are budgeting.

    Wrong answer: "Add a splash screen so it feels faster." Reveals a confusion between perceived and measured start. It also usually makes the measured number worse, because the splash is one more thing to inflate and draw before the first useful frame.

    Wrong answer: "Move everything to background threads." Reveals no model of the dependency order. Class loading and the dependency graph are on the critical path because the first screen needs them; moving work to a background thread that the first frame then waits on has moved the cost, not removed it.

    Wrong answer: "1,960 ms, so we need a faster device tier as the target." Reveals a habit of renegotiating the requirement instead of designing to it. Fair as a last resort, and unrecoverable as a first move.

    Compare your process: did you split on-device work from network before you proposed a single fix, or after?

??? note "B2 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks the display refresh rate before computing anything | Assumes 16.7 ms because 60 Hz is the number people memorise |
    | Trade-offs | Reserves a share of the frame for the system and says why | Allocates the entire 11.1 ms to application code |
    | Depth | Names precisely what leaves the bind path: formatting, parsing, text measurement | Says "optimise the bind" |
    | Communication | Points out that the newer device is the harder target | Treats the fast device as the easy case |

    Model answer: at 90 Hz the frame budget is 1000 divided by 90, so 11.1 ms, and I assume I own about 70 percent of it, roughly 7.8 ms, with the rest going to input, the compositor and the render thread. Three binds at 2.4 ms plus 4.5 ms of layout and draw is 11.7 ms, so every fling that pulls in three items drops a frame.

    Shaving the bind is not the fix; a 10 percent win still leaves me over. The fix is to make bind a set of view writes only, moving formatting, date arithmetic and text measurement onto a background thread when the page arrives and caching the result on the item model.

    If that takes bind to 0.6 ms, three items cost 6.3 ms. Prefetching ahead of the scroll so that at most one item binds per frame gets me to 5.1 ms and leaves room for the frame where a bitmap also lands.

    Wrong answer: "Use a faster list component." Reveals a substitution of a library for a measurement. The 2.4 ms is being spent inside your bind callback, and no list implementation can make your string formatting cheaper.

    Wrong answer: "Load fewer items per page." Reveals a confusion between page size and per-frame work. Binding happens as views scroll into view regardless of how many items arrived in the last response.

    Wrong answer: "It is 11.1 ms and we use 11.7 ms, so we are 0.6 ms over." Reveals that the system's share was never reserved. Correct arithmetic on the wrong budget produces a fix that is measured as insufficient two weeks later.

    Compare your process: did you ask for the refresh rate, or did you reach for 16.7 ms?

??? note "B3 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Converts the screen into bytes before discussing any technique | Names compression, caching and prefetch as a list |
    | Trade-offs | Shows that prefetch competes with the visible screen for the same link | Treats prefetch as free because it is in the background |
    | Depth | Derives the resized image size from display pixels and a stated bytes-per-pixel assumption | Says "resize images" with no target size |
    | Communication | Makes prefetch depth a remote-config value rather than a constant | Picks one prefetch depth for all networks |

    Model answer: one screen is 9.6 KB of JSON plus 1,760 KB of images, about 1.77 MB, which at 0.4 Mbit per second is 35.4 seconds. Two screens of prefetch triples that to about 106 seconds and, worse, the prefetch shares the link with the screen the user is staring at.

    The largest single win is server-side resizing. The card displays at 1,080 by 600 actual pixels; at an assumed 0.12 bytes per pixel, which I would measure on our own imagery, that is about 78 KB, so eight cards are 624 KB and about 12.5 seconds. Then I order the work: JSON first, so a text-complete screen appears in under half a second including one round trip, then images in view order, cancelling on view recycle.

    Prefetch becomes the next two images rather than the next two screens, and only after the visible screen is done. Depth and quality both move with the connection class, so they are remote config, not constants.

    Wrong answer: "Add a CDN." Reveals a latency fix applied to a bandwidth problem. A closer edge cuts the 300 ms round trip, not the 14.2 Mbit you still have to move through a 400 kbps link.

    Wrong answer: "Cache aggressively." Reveals a fix aimed at the second visit. The number in the prompt is first paint on a cold cache, which is the case that decides whether the user stays.

    Wrong answer: "Use a more efficient image format and we are done." Reveals a single-lever answer. Format matters, and it is inside the 0.12 bytes per pixel assumption, but 220 KB was mostly wrong resolution rather than wrong codec.

    Compare your process: did you compute seconds, or did you stop at megabytes?

??? note "B4 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Separates encoded bytes from decoded bytes in the first sentence | Uses one number for both |
    | Trade-offs | Lets the 12-image capacity decide that a disk tier exists | Adds a disk cache because caches usually have tiers |
    | Depth | Computes the 48.8 MB full-resolution decode and names the downsample as the fix | Mentions out-of-memory as a risk with no number |
    | Communication | Says which fraction of process memory was chosen and why it is a choice | States one eighth as if it were a rule |

    Model answer: 32 MB of bitmap budget, and each card image costs 1,080 times 600 times 4 bytes, so 2.59 MB decoded, which is 33 times its 78 KB encoded size. That is 12 images, about a screen and a half, so the memory tier alone cannot survive a scroll back and a disk tier is forced rather than chosen.

    Disk stores encoded bytes: 200 MB at 78 KB each is about 2,564 images, roughly 320 screens, and the tax is a decode on every disk hit, assume 8 ms on this tier, which must be off the main thread to stay inside the 11.1 ms frame budget.

    The trap is decoding the 4,032 by 3,024 source at full size: 48.8 MB of transient allocation for an image that occupies 2.59 MB on screen, so one decode is one and a half times the whole bitmap budget. I downsample during decode to the nearest size at or above display size, and I cap the maximum dimension I will ever decode regardless of what the server sends.

    Wrong answer: "Use soft or weak references so the collector handles it." Reveals a preference for a mechanism over a budget. It converts a predictable eviction policy into an unpredictable one and makes the frame timing depend on collector behaviour.

    Wrong answer: "Store decoded bitmaps on disk so we skip the decode." Reveals that the disk cost was never computed. Decoded frames are 2.59 MB each, so 200 MB of disk holds 77 images instead of 2,564, and you have traded a 33 times capacity loss for 8 ms.

    Wrong answer: "The server should never send 4,032 by 3,024." Reveals a client design that trusts the server. True and worth saying, and you still cap the decode, because you cannot force-upgrade the client that meets a bad response in two years.

    Compare your process: did you write down the decoded size before you talked about eviction?

??? note "B5 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Separates the foreground requirement from the background one immediately | Designs one transport for both states |
    | Trade-offs | Kills polling on two independent grounds, server load and radio tail | Rejects polling as "inefficient" with no number |
    | Depth | Names push as best effort and makes it a sync trigger rather than a carrier | Treats push as guaranteed delivery |
    | Communication | Keeps a low-frequency poll as a named safety net and prices it | Presents a single transport with no fallback |

    Model answer: polling every 30 seconds is 2,880 polls per device per day, which is about 333,000 requests per second across 10 million devices, and 2,880 times an 8 second radio tail is 6.4 hours of elevated radio per device per day. Either number kills it on its own, and it still misses the 2 second target by up to 30 seconds.

    Stated as the ratio that generalises: an active conversation is about 30 messages per connection per hour, and a 30 second poll spends 120 requests an hour to carry them. Below roughly 1 message per connection per hour a poll is genuinely cheaper than a held connection; above roughly 10 an hour it is not.

    Foreground, I take a persistent socket with a 30 to 60 second heartbeat so middleboxes do not drop it. At an assumed 8 percent peak foreground concurrency that is 800,000 connections, which is a fleet sizing exercise rather than a design problem. The real costs are the reconnect storm on a server restart, which I handle with jittered backoff and a connect budget, and the fact that heartbeats pay the radio tail.

    Backgrounded, push is the only wakeup the operating system guarantees, and it is best effort. So push carries a signal, never the message, and the client responds by running one delta sync that reconciles whatever was dropped or collapsed. As a safety net I poll once every 15 minutes when backgrounded and push has been silent: 96 polls per device per day, about 11,000 requests per second across the fleet and about 13 minutes of radio tail.

    Wrong answer: "Use a websocket for everything." Reveals no model of background execution limits. The operating system will suspend the process and the socket dies; a design that assumes otherwise fails silently on exactly the users who complain that messages are late.

    Wrong answer: "Push notifications, that is what they are for." Reveals push treated as a delivery guarantee. Push is rate limited, can be collapsed, can be dropped when the device is offline past the time to live, and the user can revoke the permission entirely, which is a design branch and not an edge case.

    Wrong answer: "Poll faster in the foreground and slower in the background." Reveals a single mechanism tuned rather than a decision made. It preserves the two costs that eliminated polling and buys nothing that a socket does not do better.

    Compare your process: did you compute requests per second before you named a technology?

### Rubric notes, interleaved set

??? note "I1 rubric, model answer and wrong answers (the answer is not to add it)"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Establishes that there is exactly one writer per record before touching conflict resolution | Starts choosing a conflict resolution strategy |
    | Trade-offs | Prices the permanent operational surface of a sync engine against a few hundred rows | Prices only the offline capability gained |
    | Depth | Names what you would still build (durable outbox, server version, full refetch) and what you would not | Refuses without proposing anything |
    | Communication | Gives the threshold at which the answer flips | Either builds it or dismisses the proposal |

    Model answer: I would not build it, and here is the arithmetic. A few hundred rows at an assumed 200 bytes each is about 60 KB. A full state refetch on open is therefore 60 KB, which on the 400 kbps link from B3 is 1.2 seconds, and on any better link it is invisible. Delta sync exists to avoid moving a dataset that is expensive to move, and this one is not.

    Conflict resolution exists to reconcile concurrent writes to the same record from different writers. Here the only writer is the user, and the only concurrency is the same user on two devices, which a server-assigned version plus a rejected-write retry handles completely.

    What I would build instead: a durable outbox so writes made offline survive process death, a client-generated operation identifier used as the idempotency key on the server, a monotonic server version per record so a stale write is rejected rather than silently applied, and a full refetch on open with the response cached for the offline read path. That is one table and one worker.

    What the proposal costs if we accept it: an operation log with its own compaction and retention policy, tombstones with a retention window that must exceed the longest offline period we support, hybrid logical clocks that every future contributor has to reason about, and a class of bug that only reproduces on a user's device after a specific interleaving. All of that ships in a binary we cannot recall.

    I would revisit when either condition holds: two writers genuinely edit the same record concurrently (shared lists, collaborative notes), or the dataset grows past roughly ten times this size so that a full refetch stops fitting the bytes-per-screen budget.

    Wrong answer: designing the sync engine competently, as asked. Reveals a candidate who optimises whatever they are pointed at. This prompt exists to see whether you will produce a number that contradicts the request.

    Wrong answer: "No, that is over-engineering." Reveals the right instinct with no evidence. A refusal without arithmetic scores the same as compliance without arithmetic, because neither shows a decision process.

    Wrong answer: "Use last write wins on the device clock, it is simpler." Reveals a serious misconception. Device clocks are user-settable and drift, so last write wins on a device timestamp is a silent data-loss mechanism, not a simplification. If you want simple, the server assigns the version.

    Compare your process: did you say the threshold at which you would change your mind?

??? note "I2 rubric, model answer and wrong answers (brownfield)"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Separates the two regressions, start time and crash rate, and asks whether they share a cause | Treats "the app got worse" as one problem |
    | Trade-offs | Buys the ability to measure and to roll back before buying any architecture | Proposes modularisation in the first quarter |
    | Depth | Specifies the staged rollout, the kill switch and the halt criterion in numbers | Says "we will add feature flags" |
    | Communication | States explicitly what is not being changed in two quarters and why | Presents a target architecture with no order |

    Model answer, in the order I would ship it.

    First, buy observability and a stop button, because right now a bad release runs for two weeks. Two weeks of work: staged rollout at 1 percent, 10 percent, 50 percent and 100 percent with an automatic halt on crash-free sessions dropping more than 0.2 points against the previous release; a remote kill switch for the partner SDK specifically; and a cold-start regression gate in the pipeline at p90 by device tier. None of this changes the architecture and all of it changes what the next incident costs.

    Second, attack the two regressions with the measurement, not with a theory. The partner SDK is the obvious suspect for both numbers, and the kill switch makes that a one-line experiment rather than a release. Apply the B1 method: split on-device work from network, defer every initialiser that is not the crash reporter, and move disk reads off the startup path.

    Third, the images. Source-resolution images from blob storage is a server-side change with no client release, and from B3 it is the largest byte win available. Add a resizing layer with device-size variants and cap the client's decode dimensions so an old client meeting an old response cannot allocate 48 MB.

    Fourth, and only fourth, the API shape. Full object graph per screen is a bytes problem and a coupling problem. Introduce one endpoint shaped for one screen, versioned, alongside the existing one, and move a single screen to it. This is the step that teaches you what a full backend-for-frontend would cost.

    What I am deliberately not doing in two quarters: modularising the monolith, migrating the UI framework, or introducing a sync engine. Each is a multi-quarter project whose benefit is developer velocity, and neither of the two numbers I was given is a velocity number.

    The order is not arbitrary. Rollout control first because it is the cheapest and most reversible and it bounds the damage of everything after it. The API last because it is the decision that is expensive to reverse on clients we cannot force-upgrade.

    Wrong answer: "Rewrite it in the current architecture." Reveals a template answer. It changes everything at once, has no intermediate state to measure, and it will not fit two quarters with no new headcount.

    Wrong answer: "Roll back the partner SDK." Reveals a fix without a diagnosis. It may be right, and until the kill switch exists you cannot test it, and if the partnership is contractual "remove it" is not available.

    Wrong answer: "Modularise so builds get faster." Reveals an engineering preference presented as a user outcome. Build time is a real cost and it is not one of the two numbers on the table.

    Compare your process: did you buy the ability to stop a bad release before you designed anything?

??? note "I3 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Notices that "only two-device users" points at ordering and identity, not at the transport | Investigates the network |
    | Trade-offs | Separates the duplicate problem from the ordering problem; they have different fixes | Proposes one fix for both symptoms |
    | Depth | Names the device wall clock as the root cause of the ordering bug and gives the replacement | Says "use timestamps properly" |
    | Communication | Says which fix is client-only and which needs a wire contract change | Presents both as one change |

    Model answer: two symptoms, two causes, and the two-device detail is the clue for both.

    Duplicates. At-least-once transport means the same message can be delivered more than once, which is expected and is not the bug. The bug is that the client has no stable identity to deduplicate on.

    The fix is a client-generated identifier assigned when the message is created, carried through send, retry and delivery unchanged, and used as the primary key of the local message table. Retries then collapse by construction. If the identifier is assigned by the server on receipt, a retried send creates a second message, which is exactly what a user with a flaky connection sees.

    Ordering. Sorting by the sending device's wall clock is the root cause. Device clocks are user-settable, drift, and jump when the time zone or the network time source changes, so two devices ordering the same conversation will disagree, and one device with a clock 40 seconds fast pins its own messages to the top forever.

    The replacement is a server-assigned monotonic sequence per conversation, which is the simple answer, or a hybrid logical clock if messages must be orderable before the server has seen them. The client keeps the device timestamp for display only and never for sort.

    What each fix costs. The identifier is a wire contract change: the server must accept a client-supplied identifier and treat a repeat as the same message, which means an idempotency window on the server side and a client that is old enough to not send one must keep working. The ordering fix is mostly server-side, plus a client migration that re-sorts the local store on upgrade.

    Multi-device convergence, which is the part the prompt is really testing: each device applies the same server sequence, so both converge on the same order without talking to each other. Local unsent messages sort after everything acknowledged, in outbox order, and visibly move into place when acknowledged. Showing that movement is better than hiding it, because the alternative is an order that silently changes on refresh.

    The class from the [failure library](index.md#the-failure-library): clock skew, with at-least-once delivery amplifying it into visible duplicates.

    Wrong answer: "Make the transport exactly-once." Reveals a belief that a delivery guarantee can be bought. At-least-once plus an idempotency key is what exactly-once effects are made of, and the key has to be generated where the intent originates, which is the sending device.

    Wrong answer: "Sync the device clock with a time server." Reveals a fix that is not enforceable. You cannot make a user's clock correct, and even a correct clock does not give you a total order across devices.

    Wrong answer: "Deduplicate by message text and timestamp." Reveals a heuristic standing in for identity. Two identical short messages sent deliberately are indistinguishable from a duplicate, and users do send "ok" twice.

    Compare your process: did the two-device detail change your hypothesis, or did you start with the network?

??? note "I4 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks what the model is for and whether reordering on every scroll is the requirement or a proposal | Accepts "on every scroll" as a constraint |
    | Trade-offs | Prices the inference against the frame budget from B2 before choosing where it runs | Chooses on-device because it sounds better for privacy |
    | Depth | Specifies model delivery as a versioned artefact with staged rollout and a size gate per tier | Says "download the model" |
    | Communication | Names the fallback for devices that cannot run it, and treats that as the default path | Treats the no-model device as an error case |

    Model answer: first, the frame budget decides where this runs. From B2 I own about 7.8 ms of a 90 Hz frame and layout plus draw already takes 4.5 ms, so any inference on the main thread is out.

    Reordering on every scroll event is out too: scroll events fire far more often than the list changes, and reordering visible rows under the user's finger is a usability defect regardless of cost. What the product wants is a reordered page, so I score once per page of candidates on a background thread, off the scroll path, and swap the new order in at a stable point.

    Sizing the work: assume a page of 50 candidates and a per-candidate inference cost of 0.2 ms measured on the mid tier, which is 10 ms per page. That is off the main thread and happens once per page, so it is invisible. If the measured cost were ten times that, the answer becomes score the next page while the current one is being read, which is a prefetch problem rather than a latency problem.

    Model delivery: the model is a versioned artefact, downloaded separately from the app binary, so that shipping a new model does not need a store release and a bad model can be reverted in minutes.

    That means a version stamp logged with every prediction, a signature check on the artefact, a staged rollout by cohort, and a size gate. An assumed 8 MB artefact is fine on the mid tier and is a real cost on a metered connection, so it downloads on unmetered networks by default with a bounded metered fallback.

    The device where it will not fit: this is the default path, not the error path. The client ships with the server-supplied order and uses it whenever the model is absent, stale beyond a threshold, or too slow on this device measured at first run. That single decision converts a hard dependency into a soft one.

    What I would take from the [ML System Design course](ml-system-design.md): the training and evaluation loop, the fact that the on-device model is a distilled or quantized version of a server model with a measured quality gap, and the point that logging predictions with the model version is what makes any of this debuggable later.

    Wrong answer: "Run it on the server, clients are slow." Reveals that the offline requirement was dropped. The prompt says the feed is cached, which means the reorder has to work with no network, or it is not the feature that was asked for.

    Wrong answer: "Ship the model inside the app binary." Reveals no model of the release cycle. A weekly refresh through a store release is not weekly, and a bad model then needs a new release to remove.

    Wrong answer: "Reorder on every scroll callback as requested." Reveals compliance over judgement. Say why it cannot work, in milliseconds, and offer the version that delivers the same product outcome.

    Compare your process: did you compute against the frame budget before you decided where inference runs?

??? note "I5 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Ties the spike to the release rather than to traffic, and uses the self-recovery as evidence | Investigates backend capacity |
    | Trade-offs | Enumerates three mechanisms and gives a distinguishing measurement for each | Names one cause and stops |
    | Depth | Adds the mobile-specific step: rollout is a percentage of installs, not a percentage of requests | Fixes it as a generic deploy problem |
    | Communication | Proposes one measurement before three changes | Proposes three changes at once |

    Model answer: the class is a release-triggered thundering herd, and the fact that it settles in about two hours with no intervention is the strongest clue. Nothing is permanently broken; something is being done once per updated install and then not again.

    Three candidate mechanisms, each with a signal that separates it:

    - A first-run migration. The new binary migrates the local database or rewrites a cache on first open, and it fails on a subset of device states. Signal: errors concentrate in the first session after upgrade, cluster by previous app version, and the stack points at storage.
    - A synchronised cold call. Every updated client fetches config, refreshes a token or re-registers a push token on first launch, all within the same window as the store rollout. Signal: a matching spike in one backend endpoint, and errors that are timeouts rather than exceptions.
    - Version skew at the contract. New clients call a field or endpoint that a fraction of the backend fleet does not serve yet, or old clients meet a response shaped for the new one. Signal: errors correlate with backend instance version, not with device state, and they stop when the backend rollout finishes.

    Measure before changing: error rate split by previous app version, by first-session against later sessions, and by backend build. One release of instrumentation tells you which of the three it is, and the three fixes are different.

    The fix set, in order: make first-run work idempotent, resumable and off the critical path; jitter every first-launch network call across a window rather than firing on open; and stage the rollout by install percentage with an automatic halt on crash-free sessions, so a bad release reaches 1 percent of installs rather than all of them.

    The mobile-specific point most candidates miss: a server rollout can be halted in seconds, and a client rollout cannot. Once a binary is on a device you own that code until the user updates, so the kill switch has to be inside the shipped binary and controlled remotely. Designing the off switch is part of designing the feature.

    Wrong answer: "Add more backend capacity for release day." Reveals no mechanism. It also hides a client bug behind a server cost you now pay every release.

    Wrong answer: "It recovers on its own, so it is acceptable." Reveals a missing user model. Two hours of a 5 to 8 times error rate on every release is an availability cost paid on schedule, and it quietly teaches the team to ship less often.

    Wrong answer: "Release at night." Reveals a workaround rather than a design. Store rollouts are not instantaneous or time-zone aligned, so the herd arrives whenever devices choose to update.

    Compare your process: did the self-recovery change your hypothesis, or did you ignore it?

??? note "I6 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Treats the 40 host apps as the customer and their budgets as the requirement | Designs the SDK around its own convenience |
    | Trade-offs | Gives the SDK an explicit cost budget and a documented threading contract | Promises to be "lightweight" |
    | Depth | Names quality-of-service tiers so the SDK's traffic yields to the host's | Says "upload in the background" |
    | Communication | States what the public API guarantees and what it deliberately does not | Lists features |

    Model answer: an SDK's requirements are the host app's budgets. I would write them down as numbers first: under 5 ms of main thread time during initialisation, zero disk or network on the calling thread, a bounded on-disk queue, and a stated binary size cost. Those become the contract, and they are tested in the SDK's own pipeline the same way a product tests cold start.

    The 90 ms regression: a synchronous flush during initialisation put disk and network on the host's startup path. Initialisation becomes a cheap object construction that registers a listener and returns; the queue opens lazily on the first event, on a background dispatcher the SDK owns; and the flush is scheduled work rather than an inline call. If the host needs a flush before termination it calls one, explicitly, and that call is documented as blocking.

    The starvation complaint: the SDK and the host are competing for one radio and one link. Analytics is the lowest quality-of-service tier there is. It should upload only when the connection is unmetered or when the queue is near its cap, coalesce into batches sized to fill one radio wake rather than dribbling events, and defer entirely while the host reports itself as latency sensitive. That last part needs a public hook, because the SDK cannot see the host's image prefetch.

    The rest of the contract, stated once so nobody has to guess: events are best effort with an at-least-once delivery guarantee and a client-generated event identifier for deduplication; the disk queue has a fixed byte cap with oldest-first eviction and the eviction count is itself reported; errors are surfaced through a single callback and never thrown into the caller's thread; and the public surface is additive-only within a major version, because 40 teams upgrade on 40 schedules.

    What I would not do: sample by default without telling the host, or start a background thread at class-load time. Both are the kinds of thing that make teams pin an old version forever.

    Wrong answer: "Make initialisation asynchronous." Reveals a partial fix. If the first event still touches disk on the caller's thread, or if the async initialiser is awaited before the first screen, the 90 ms moves rather than disappears.

    Wrong answer: "Let each team configure everything." Reveals a contract pushed onto the customer. Forty teams will pick forty configurations and the support cost lands back on you; pick defaults that meet the budgets and expose two knobs.

    Wrong answer: "Upload immediately so we do not lose events." Reveals an SDK optimising its own metric at the host's expense. Analytics completeness is worth less than the host app's scroll, and saying that out loud is the sign you have designed a library rather than a feature.

    Compare your process: did you write the host's budgets down as numbers before you designed anything?

---
## 12. Mastery check and remediation map

Seven short-answer items, one for each learning objective in Section 4, in the same order. Write or say your answer before opening the block. Producing the answer is what builds the retrieval path; recognising a correct answer does not.

**M1** (Objective 1, classify the scope and steer a misfiled round). Name the three scopes a mobile prompt can have, give the tell for each inside the first two minutes, and give the single sentence you say when the round is a server round wearing a mobile title.

??? note "Show answer"

    | Scope | The tell in the first two minutes | What the round then rewards |
    |---|---|---|
    | Client scope | The artefact is an app or a screen, and the first follow-up is about offline, process death or an old app version | Budgets, sync, transports, release control |
    | Component or SDK scope | The artefact ships inside somebody else's binary, or is one widget: an image loader, an analytics SDK, a download manager | Public API surface, threading contract, initialisation cost, binary size, versioning |
    | Server round with a mobile title | The prompt names a service, a store or a fan-out, and the follow-ups are about capacity, consistency or partitioning | The [Modern System Design](modern-system-design.md) material, not this page |

    The steering sentence, which agrees before it redirects: "Happy to design the service side, and I want to check the emphasis first: shall I spend most of the time on the backend, or on the client contract, the offline behaviour and what an old app version does with the new response?"

    Why it works: it does not contradict the interviewer, it offers them the fork rather than making it, and it names the three things you are strongest at so that a "do both" answer still lands where you want it. If they say backend, believe them and design the backend.

**M2** (Objective 2, a budget set that eliminates). Reference tier: a mid-range phone with a 90 Hz display, about 256 MB of process headroom, on a 400 kbps link with a 300 ms round trip. The prompt is a photo feed. Give cold start, bytes on the wire for the first screen, and peak resident memory, showing each step, then name one number you would delete.

??? note "Show answer"

    Frame budget first, because it constrains everything else: 1000 divided by 90 equals 11.1 ms, and assume you own about 70 percent, so 7.8 ms. Eliminates: image decode on the main thread, since a single decode of a card image is around 8 ms on this tier.

    Cold start: take 1,200 ms p90 as the target unless product states otherwise, and say it is an assumption. The low tier runs roughly 1.6 times slower, so on-device startup work has to be under 1,200 divided by 1.6, which is 750 ms, for the low tier to clear the same bar. Eliminates: any blocking network call before the first usable frame, because the fetch alone is a round trip plus payload and there is no 750 ms budget that survives it.

    Bytes for the first screen: 8 cards at about 1.2 KB of JSON is 9.6 KB, which is 0.077 Mbit and lands in about 0.19 seconds plus one 300 ms round trip. Images at 1,080 by 600 pixels and an assumed 0.12 bytes per pixel are about 78 KB each, so 624 KB, which is 5.0 Mbit and 12.5 seconds. Eliminates: any design that waits for images before showing a screen, and prefetching a second screen while the first is still loading.

    Peak resident memory: 256 MB of headroom, one eighth allocated to bitmaps is 32 MB, and each decoded card image is 1,080 times 600 times 4 bytes, so 2.59 MB. That is 12 images. Eliminates: a memory-only image cache, and full-resolution decode, since a 4,032 by 3,024 source decodes to 48.8 MB, which is one and a half times the entire bitmap budget.

    The number I would delete: monthly active users. It does not move the frame budget, the startup budget, the byte budget or the memory ceiling, all of which are per-device quantities. It earns a place later if it sizes the push fan-out or the socket concurrency, and not before.

**M3** (Objective 3, pagination over a mutating feed). A feed is ordered newest first. While the user reads page 1, twelve items are inserted at the head and three are deleted from page 2. Name the scheme you would ship and the exact user-visible defect each rejected scheme produces.

??? note "Show answer"

    Ship: keyset pagination on the sort key plus a unique tie-break, so the cursor is the last returned item's (created_at, id) pair and page 2 asks for rows strictly less than it. The tie-break column is not optional.

    | Rejected scheme | The exact defect, given this mutation |
    |---|---|
    | Offset or limit and skip | Twelve inserts at the head push everything down by twelve, so page 2 re-serves the last twelve rows of page 1. The user sees twelve duplicates |
    | Offset, on the deletes | Three deletions pull rows up by three, so three rows that would have started page 2 are never served. The user sees a silent skip, which is worse than a duplicate because nothing looks wrong |
    | Page number | Same as offset, and it additionally invites the client to cache "page 3" as a stable identity, which it is not |
    | Cursor with no tie-break | Two items sharing a `created_at` value straddle the boundary, so one is served twice or not at all depending on the comparison operator. It reproduces once a week and never on your machine |
    | Any scheme, if the server reranks between requests | Every scheme breaks, because the sort order itself moved. Duplicates and skips both appear and the cursor is meaningless |

    Two design consequences that come with the choice:

    - New items at the head are simply not in a downward scan, so they need a deliberate product answer: a "new posts" affordance at the top rather than a silent reorder under the user's finger. Deciding this out loud is what separates a pagination answer from a pagination fact.
    - If the server reranks, the cursor must carry a ranking snapshot identifier so the whole scroll session reads one consistent ordering, and the snapshot needs a retention window.

    The cache decision that makes duplicates invisible: the local store is keyed by item identifier, so a re-served row upserts instead of appending. That does not excuse offset pagination, and it does mean a single duplicated row never reaches the screen.

**M4** (Objective 4, the offline write path). A user posts a comment on a connection that drops every couple of minutes. Give the durable queue, the retry policy, the idempotency key and the conflict rule, and say what the user sees at send, retry, conflict and permanent failure.

??? note "Show answer"

    Durable queue. The user action writes two rows in one local transaction: the optimistic comment row and the outbox row that will send it. One transaction, because a process kill between the two leaves either a comment that never sends or a send with no comment, and both are bugs a user reports as "it disappeared".

    Retry policy, classified rather than uniform:

    | Response | Treatment |
    |---|---|
    | Network error, timeout, 429, 5xx | Retry with exponential backoff and full jitter, capped at about 5 minutes between attempts |
    | 4xx other than 408 and 429 | Permanent. Do not retry, surface it, and keep the local row |
    | Success | Replace the local row with the server's, keeping the local identifier so the list does not jump |

    Full jitter matters more on a client fleet than on a server: when a regional outage ends, every device in that region has the same backoff schedule, so an unjittered retry is a self-inflicted thundering herd against a service that has just come back.

    Idempotency key. Generated on the device when the comment is created, stored on the outbox row, and reused unchanged on every attempt of that same logical write. Generating it per attempt is the classic bug that produces duplicate posts. The server retains keys for at least the client's total attempt deadline, so if the client gives up after 24 hours the server keeps keys longer than that.

    Conflict rule. A comment is append-only, so there is no conflict; say that explicitly rather than designing one. For an edit, the server assigns a monotonic version per record, a write carrying a stale version is rejected with the current state attached, and the client keeps the losing text as a local draft rather than destroying it.

    What the user sees:

    | Moment | What is on screen |
    |---|---|
    | Send | The comment appears immediately in a visibly pending state. A state, not a spinner, because it may last minutes |
    | Retry | Still pending, with no error text. Only after a threshold, say 30 seconds, does a "waiting for connection" line appear, so a two second blip produces no alarming UI |
    | Conflict | The server state is shown, and the local version is offered back as a draft with an explicit action. Nothing is silently discarded |
    | Permanent failure | An explicit failed state with the reason and a retry action, and the row stays in the list. A row that vanishes reads as data loss even when nothing was lost |

**M5** (Objective 5, transport choice). A live score screen updates about 6 times a minute during a match, watched in the foreground, and must also alert the user at match start while backgrounded. Choose the transport, cite the messages-per-connection-per-hour figure that decides it, and name the battery cost the rejected option carried.

??? note "Show answer"

    The deciding figure: 6 per minute is 360 messages per connection per hour. Below roughly 1 message per connection per hour a poll is genuinely cheaper than holding a connection; above roughly 10 an hour a held connection wins. At 360 the question is settled before any other consideration.

    Choose: server-sent events over the existing HTTP stack for the foreground stream. The traffic is server to client only and it is text, so a websocket buys a client-to-server channel you would not use, and server-sent events give you automatic reconnect with a last-event identifier, which is exactly the resume semantics a match stream needs.

    Rejected, polling, with its battery cost. To hit 10 second freshness you poll 360 times an hour. Assume the radio holds a high-power state for about 8 seconds after each transfer: a poll every 10 seconds never lets the radio drop out of that state, so you pay a continuously powered radio and receive nothing between goals. You have bought the cost of a persistent connection without getting one.

    Rejected, websockets. Correct when the client also sends: chat, presence, collaborative editing, a game. It costs bidirectional connection state, sticky routing and a rebalancing problem when a server restarts. None of that is paid for by a score feed.

    Rejected, a persistent broker such as MQTT. It wins with many topics per device, retained last-known values on subscribe, and a very small wire format on a constrained link, which is why it turns up in vehicles and sensors. Below roughly a handful of topics per device it is a broker you operate and a second authentication path you maintain, for nothing.

    Backgrounded: push, with a collapse key so a burst of goals collapses to the latest state, and a time to live short enough that a goal alert cannot arrive after the match has finished. Push is best effort, so the screen still reconciles with one fetch on resume.

**M6** (Objective 6, diagnose a client regression). Users report the app is slow to open, but only sometimes, and mostly after they have used other apps. Cold start p90 has not moved. Name the one cause and the one on-device measurement that confirms it.

??? note "Show answer"

    Cause: the process is being terminated by the operating system to reclaim memory, so sessions the user expects to be a resume are cold starts instead. The startup path did not get slower; more of the population is running it. Peak resident memory is the regression, not startup code.

    The one measurement that confirms it: the ratio of cold starts to warm starts, segmented by device memory class, together with the reason the operating system reports for the process ending. If the cold-start share rose while cold-start p90 held flat, the diagnosis is memory footprint. If the share held and p90 rose, it is startup work.

    The general case, one row per regression class:

    | Symptom | Named cause | The single on-device measurement that confirms it |
    |---|---|---|
    | Cold start regressed | Eager initialisation on the main thread, or a disk read on the startup path | A startup trace with per-section timing, at p90, split by device tier |
    | Dropped frames while scrolling | Main-thread work inside bind, or a synchronous decode | A frame timing histogram plus a main-thread trace captured during a fling, not at rest |
    | Memory kill | Peak resident memory, usually a full-resolution decode or an unbounded cache | Peak memory sampled just before the kill, plus the OS-reported termination reason |
    | Battery drain | Radio duty cycle from uncoalesced requests, or an un-duty-cycled sensor or location stream | Radio-on and wakelock time attributed per subsystem over a fixed window, with the app in a known state |
    | Lost upload | In-memory upload state destroyed when the process was terminated | The count of outbox rows that never reached a terminal state, correlated with process termination reasons |

    The discipline the item is testing: one cause and one measurement. A candidate who lists five possible causes and no measurement has described a search space, not a diagnosis.

**M7** (Objective 7, mid to staff). Name the four additions that convert a mid-level mobile answer into a staff-level one on this page, and say what each one changes about a decision.

??? note "Show answer"

    - **Version skew across app versions you cannot force-upgrade.** Changes the wire contract decision. Once you state that the oldest version you support is, say, eighteen months old, you can no longer remove or repurpose a field, so a new screen shape ships as an additional response alongside the old one rather than as a replacement, and the deprecation gets a date.
    - **A server-side kill switch.** Changes what you are allowed to ship at all. It converts a two-week rollback into a two-minute one, which means the switch has to be in the binary before the feature is. Saying this out loud reorders the plan: you build the switch in the release before the feature.
    - **One release health metric with a rollback threshold.** Names the metric, the number and the window at which the rollout halts without a meeting: for example, crash-free sessions dropping more than 0.2 points against the previous release measured over the first 4 hours at 10 percent. Committing to it forces the metric to exist before the ramp rather than after the incident.
    - **One degradation path.** States what the screen shows when the network, the config service, the push token or the on-device model is unavailable: the cached page, the server-supplied order, the last known state with its age shown. It converts a hard dependency into a soft one and is usually the cheapest availability win in the whole design.

    Each of the four turns a claim into a decision somebody can be held to. That, rather than extra components, is what the level difference is made of.

**Threshold.** Five of seven, with M2 and M4 among them. Those two are weighted because a candidate who cannot derive a budget set or specify an offline write path has described a client rather than designed one, and every case study on this page leans on both.

| Missed item | Reread | Redo |
|---|---|---|
| M1 | Module 1 on scope, and Module 19 on interview mechanics | Take three prompts from Section 10 and write the steering sentence for each in one minute |
| M2 | Module 3 on budgets, plus the memory arithmetic in Module 8 | Problems B1, B3 and B4 out loud, naming the eliminated option after every number |
| M3 | Module 7 on pagination, plus Module 12 on storage tiers for the local upsert | Draw the cursor, tie-break column and "new items" affordance for the news feed client case study |
| M4 | Module 6 on networking and retries, and Module 11 on the sync engine | Problem I3, then redo the chat client case study's offline send queue from the prompt alone |
| M5 | Module 10 on push and real-time, plus the transport rows of the [trade-off atlas](index.md#the-trade-off-atlas) | Problem B5, then redo it with the update rate changed to one message an hour and see which decisions flip |
| M6 | Module 13 on battery, Module 14 on concurrency and memory, and the hub's [failure library](index.md#the-failure-library) | Problems I5 and I2, then write the five-row regression table from memory |
| M7 | Module 5 on backend for frontend, Module 17 on release engineering, Module 20 on level calibration | Problem I2, then answer it again at staff level and diff the two answers line by line |

!!! warning "Scoring well right now is a weak signal"

    Passing this immediately after reading mostly measures that the page is still in working memory. The signal that predicts interview performance is the delayed check in Section 15, taken a week later with the page closed. If you score seven of seven today and four of seven next Tuesday, the honest reading of your ability is four.

---

## 13. Common mistakes

Technical mistakes cost you a follow-up. Delivery mistakes cost you the round. The last four rows are the ones that appear in debrief notes, and they are the cheapest to fix.

| The mistake | Why candidates make it | What the interviewer concludes | What to say instead |
|---|---|---|---|
| Drawing the client's layers as the answer | Architecture patterns are the part that was studied, and they are easy to draw | Has learned a diagram, not a system. Cannot be asked anything the diagram does not contain | "Before the layers: the budgets are 11.1 ms a frame, 1,200 ms cold start on the mid tier and about 32 MB of bitmaps. Here is what each one already rules out" |
| Treating 16.7 ms as the frame budget | 60 Hz is the number everyone memorised | Has not shipped against a modern display, and will size work for the wrong device | "This device is 90 Hz, so the frame is 11.1 ms and I own about 7.8 ms of it once the compositor is paid" |
| Confusing encoded and decoded image size | Both are called "the image" | Will ship an app that gets killed on the tier most users hold | "78 KB on the wire, 2.59 MB decoded, a 33 times expansion. The 2.59 MB is what the memory budget is spent on" |
| Treating push as guaranteed delivery | It arrives reliably enough during development | Has not operated a push pipeline, and will lose messages in production | "Push is best effort and can be collapsed or dropped past its time to live, so it triggers a sync rather than carrying the payload" |
| Sorting by the device's clock | The timestamp is right there on the message | Has not seen a two-device ordering bug, and will ship one | "Device clocks are user-settable, so ordering comes from a server-assigned sequence and the device time is display only" |
| Generating the idempotency key per attempt | The key is created where the request is built | Does not have a model of what a retry is | "The key is created with the intent, stored in the outbox, and reused unchanged for every attempt of that same write" |
| Offset pagination on a feed that mutates | It is the pagination everyone learned first | Will ship duplicates and, worse, silent skips | "Twelve inserts at the head make page 2 repeat twelve rows. Keyset on the sort key plus a unique tie-break, and here is what the tie-break is for" |
| Designing as if the app can be patched | Server habits transfer without anyone noticing | Will ship a defect with no way to stop it | "The fix reaches close to zero percent of installs in the first hour, so the kill switch ships in the release before the feature" |
| Proposing a sync engine for single-writer data | Sync is the impressive-sounding part of the syllabus | Adds machinery by association rather than by measurement | "One writer, a few hundred rows, about 60 KB. A durable outbox and a server version cover it. I would revisit at concurrent multi-writer editing" |
| Designing before clarifying (delivery) | Silence feels like failure, and drawing feels like progress | Will build the wrong thing quickly and confidently | "Two questions first, because they change the shape: which device tier and network am I designing for, and what has to work with no connection?" |
| Monologuing through the client stack (delivery) | Reciting a known sequence feels safe and fills time | Cannot collaborate, and the interviewer has no opening to steer | "That is the read path. Do you want me deeper on the cache tiers, or on what happens when the process is killed mid-write?" |
| Refusing to say "I do not know" (delivery) | Believing a visible gap is a scoring event | Bluffs, which is expensive on a team and easy to detect | "I have not shipped on that platform. What I know is the constraint it imposes, and here is how I would find out which primitive to use" |
| Defending a choice instead of updating it (delivery) | Reading a follow-up as an attack rather than as information | Will not update a design in a code review either | "You are right that a collapse key loses the intermediate states. That changes the payload, not just the transport. Let me redo that part" |
| Running out of time inside the architecture section (delivery) | Layers are comfortable and offline behaviour is not | No sense of the clock, and no evidence they can ship | At minute 25: "I want to leave ten minutes for offline behaviour and release control, so I will close the architecture here and come back if there is time" |

---
## 14. Interview-day checklist

One screen. Print it or keep it open in a second window. Nothing here is new.

**Minute budget**

| Phase | 45 minute round | 60 minute round |
|---|---|---|
| Restate as a client boundary | 0 to 2 | 0 to 2 |
| Clarify: offline, multi-device, device floor, platform, API owner | 2 to 8 | 2 to 10 |
| Budgets, only where a number eliminates something | 8 to 12 | 10 to 15 |
| V0 drawn end to end | 12 to 20 | 15 to 24 |
| Breaking point, then V1 | 20 to 29 | 24 to 35 |
| One deep dive (two if 60 minutes) | 29 to 39 | 35 to 51 |
| Offline behaviour, failure modes, telemetry, release health | 39 to 45 | 51 to 58 |
| Close: trade-offs, what you skipped, first measurement | included above | 58 to 60 |

**The first four sentences**

1. "Let me restate the boundary: the client owns X, the server owes me Y, and the contract between them is Z."
2. "I will spend about six minutes on constraints and budgets, then draw the client end to end, then break it on purpose."
3. "I am assuming this has to work with no network and survive the process being killed, unless you tell me otherwise."
4. "Stop me if you want depth somewhere specific rather than coverage."

**Five clarifying questions that generalise across mobile prompts**

1. What is the device tier floor and the network I should design for, and is the target stated at p50 or p90?
2. What has to work with no connection, and for how long offline?
3. Is this one device per account or several, and can two of them write the same thing?
4. Who owns the API, and how old is the oldest app version we still support?
5. What is the release cadence, and is there already a remote kill switch I can rely on?

**Whiteboard drawing order**

```
DRAW IN THIS ORDER. WITHHOLD THE REST UNTIL A BUDGET FORCES IT.

   1             2              3              4
 +--------+   +---------+   +----------+   +---------+
 | SCREEN |<->| STATE   |<->| REPO     |<->| NETWORK |
 |        |   | HOLDER  |   | one box  |   | CLIENT  |
 +--------+   +---------+   +----------+   +---------+
                                 |
                                 | 5, after a byte or offline number
                                 v
                            +----------+
                            | LOCAL    |
                            | STORE    |
                            +----------+
                                 |
                                 | 6, once a write must survive a kill
                                 v
                            +----------+
                            | OUTBOX   |
                            +----------+

 NOT YET: cache tiers, image pipeline, sync engine, push, flags,
 background scheduler, second platform, on-device model.
 Caption: four boxes across first, then the local store, then the
 outbox, and nothing else until a stated budget forces it.
```

Draw the read path left to right first, because it is what the prompt asked about. Add the local store when a byte count or an offline requirement forces it, and the outbox when a write has to survive process death. Erasing a box you drew in minute three reads as guessing; adding one in minute twenty-two after a number reads as design.

**Three recovery scripts**

| Situation | Say this |
|---|---|
| You are stuck | "Let me state the budget I am designing against and the two options I can see: A keeps everything on device and costs memory, B fetches on demand and costs bytes on a bad link. I will take A because the offline requirement is hard, and I will flag it as reversible." |
| The interviewer disagrees | "That is a case I did not price. If the process is killed there, the in-memory queue is gone, so I would move the queue to disk before the first byte goes out rather than change the transport. Does that address it?" |
| You are running out of time | "I have about six minutes. I would rather spend them on what happens offline and how a bad release gets stopped than on finishing this dive, because I think that is the higher-value part. Say if you would prefer otherwise." |

**Before you finish**

- State the two trade-offs you actually made and what each one cost.
- State what you deliberately skipped, in one sentence each.
- Name the release health metric you would watch and the number that halts the rollout.
- Name the degradation path: what the screen shows with no network, no config service and no push token.
- Name the decision that is most expensive to reverse, which on a client is almost always the wire contract or the local schema.

---

## 15. Study schedule and spaced review

### 15a. Three plans, each defined by what it drops

Day 1 is whatever day you start. The skip column is the part of this section that is hard to write and the part worth reading.

**One week (about 8 to 11 working hours)**

| Day | Do | Skip |
|---|---|---|
| 1 | Modules 1 to 3, then problems B1 and B2 | Nothing yet |
| 2 | Modules 4 to 7, then the news feed client case study end to end | Its second deep dive; come back only if time allows |
| 3 | Modules 8 and 9, then problems B3 and B4 | Module 15 on platform variation unless you are interviewing cross-platform |
| 4 | Modules 10 and 11, then the chat client case study, then problem B5 | The maps and video client case studies |
| 5 | Modules 12 to 14, then problems I3 and I5 | Module 16 on security unless your target team owns authentication |
| 6 | Modules 17 and 18, the analytics SDK case study, then the first timed self-mock scored against Section 11 | Modules 19 and 20 for now; you will read them tomorrow |
| 7 | Modules 19 and 20, the second timed self-mock, then Section 12, then Section 14 | Everything unreached. Do not start new material the day before a round |

**One weekend (about 5 to 7 working hours)**

- Saturday morning: Modules 2, 3 and 7. The six axes, the budget set and pagination affect every prompt on this page and every prompt you will be asked.
- Saturday afternoon: one case study end to end, out loud, with a timer. Take the news feed client if you are early career, the chat client otherwise, because sync and ordering are where senior rounds go.
- Sunday morning: Modules 10 and 11 (transports and the sync engine), then problems B5 and I3.
- Sunday afternoon: Module 17, then one timed self-mock, then Sections 12, 13 and 14. Rehearse the first four sentences until they are automatic.
- Skip entirely: Modules 4, 15, 16 and 18, the second timed self-mock, every second deep dive, and ten of the eleven case studies. One case study done properly beats five skimmed, and the difference shows inside two minutes of a real round.

**One evening (about 2 to 3 hours)**

- 0:00 to 0:25, Section 14 and Module 1. Get the clock, the opening and the steering sentence.
- 0:25 to 1:05, Module 3 and Module 10. The budget set and the transport decision are the two fastest visible upgrades to a mobile answer.
- 1:05 to 2:00, one case study, requirements and v0 only, no deep dives.
- 2:00 to 2:30, Section 13 once, then Section 14 again.
- Skip: every other module, every deep dive, the mastery check, both self-mocks, and all interleaved problems except I1. If your round is tomorrow, extra breadth will not be retrievable under pressure and a rehearsed opening will be.

### 15b. Review schedule

| When | What | Minutes |
|---|---|---|
| Day 1 | The review cards below, once through | 10 |
| Day 3 | Cards, plus redo one blocked problem you got wrong | 20 |
| Day 7 | The delayed self-check in 15d, closed book, then cards | 30 |
| Day 21 | Cards, plus one interleaved problem you have not attempted | 40 |
| Day 60 | One full case study out loud with a timer, then cards | 60 |

Consistency beats the exact intervals by a wide margin. A session on day 4 and day 9 is worth far more than a perfect schedule abandoned after the first miss. If you skip a checkpoint, do the next one on the day you remember rather than restarting the sequence.

### 15c. Review cards

One fact or one decision per card. Copy this block into whatever tool you use. The format is prompt, two colons, answer.

```
Q :: Frame budget at 60, 90 and 120 hertz, and how much is yours?
A :: 16.7, 11.1 and 8.3 ms. Assume you own about 70 percent once
     input, compositor and render thread are paid.

Q :: Decoded bytes for a W by H image in 32-bit colour?
A :: W x H x 4. A 1080 x 600 card image is 2.59 MB decoded.

Q :: Decoded against encoded size for a typical card image?
A :: About 33 times. 78 KB on the wire, 2.59 MB in memory. The
     memory budget is spent on the second number.

Q :: What decides push against polling against a held connection?
A :: Messages per connection per hour. Under about 1, poll wins.
     Over about 10, a held connection wins.

Q :: What makes a retried write safe to repeat?
A :: An idempotency key generated with the intent, stored in the
     outbox, and reused unchanged on every attempt.

Q :: Why is the device clock never the sort key?
A :: It is user-settable and drifts, so two devices disagree.
     Order comes from a server-assigned sequence.

Q :: Offset pagination on a feed with 12 inserts at the head?
A :: Page 2 repeats 12 rows. Deletions cause silent skips, which
     are worse because nothing looks wrong.

Q :: What does a keyset cursor need besides the sort key?
A :: A unique tie-break column, or rows with equal sort values
     straddle the page boundary and one is lost or repeated.

Q :: What share of installs runs your fix an hour after merge?
A :: Close to zero. That is why the kill switch ships in the
     release before the feature does.

Q :: What is a push notification, precisely?
A :: A best-effort wakeup signal, collapsible and expirable. It
     triggers a sync; it never carries the only copy.

Q :: Which two rows are written in one transaction on an offline write?
A :: The optimistic local row and the outbox row. Splitting them
     loses one or the other when the process is killed.

Q :: Why does a client fleet need jitter more than a server fleet?
A :: When an outage ends, every device shares the same backoff
     schedule, and you cannot patch them.

Q :: Cost of decoding a 4032 x 3024 photo at full size?
A :: 4032 x 3024 x 4 = about 48.8 MB, roughly one and a half
     times a 32 MB bitmap budget. Downsample during decode.

Q :: When is a sync engine over-engineering?
A :: Single writer, small dataset. A durable outbox plus a
     server-assigned version covers it.

Q :: Four things a staff mobile answer adds over a mid one?
A :: Version skew across unupgradable clients, a kill switch, a
     release health metric with a rollback threshold, and a
     degradation path.
```

### 15d. Delayed self-check

Answer these from memory a week after you finish, with this page closed. Write the answers down before you check anything.

1. Take a mobile prompt you have not studied here. Derive its budget set (frame, cold start, bytes for the first screen, peak memory) and name what each number eliminates.
2. Draw its offline write path end to end, and say what the user sees at send, retry, conflict and permanent failure.
3. Name the release health metric you would gate the rollout on, the threshold that halts it, and the degradation path when the feature's backing service is down.

Twenty minutes, no notes. If you can do all three, the material is retrievable under pressure. If you cannot, the step where you stalled is the one to redo, not the whole page.

---
## 16. Reference block

!!! info "This section teaches nothing new"

    Everything below appeared earlier with its derivation. This is the version to reread the morning of the round. If a row here is the first place you have met an idea, go and read the module it came from rather than memorising the row.

### 16a. Decision tables

**Transport**

| If | Choose | Cost you must say out loud | Below this it is over-engineering |
|---|---|---|---|
| Under about 1 message per connection per hour, and the path must stay plain HTTP | Poll on a schedule, coalesced with other deferrable work | Freshness is bounded by the interval, and every poll pays a radio wake plus its tail | Nothing. This is already the cheap option; the mistake is polling faster, not polling |
| Server to client only, text, more than about 10 messages per connection per hour, foreground | Server-sent events | One held connection per foregrounded device, and a reconnect policy you own | Under about 10 messages an hour, where a poll costs less than the held connection |
| Bidirectional, frequent, latency sensitive, foreground | Websocket | Connection state, sticky routing, rebalancing on restart, and a reconnect storm to jitter | When the client never sends. You are paying for a channel you do not use |
| Many topics per device, retained last-known values, very constrained link | Persistent broker such as MQTT | A broker to operate and a second authentication path to maintain | Below roughly a handful of topics per device, where one stream carries everything |
| The app is backgrounded, or not running at all | Push, as a wakeup signal | Best effort: collapsible, expirable, revocable by the user, and never the only copy | Never skip it for background delivery. It is the only OS-guaranteed wakeup you have |

**Pagination over a list that mutates**

| If | Choose | The defect you avoid | Do not choose it when |
|---|---|---|---|
| The list is append-only at the head and sorted by a stable key | Keyset on the sort key plus a unique tie-break | Duplicates from head inserts and silent skips from deletions | The server reranks between requests, in which case no cursor is stable without a snapshot |
| The server reranks the list per request | Keyset plus a ranking snapshot identifier carried in the cursor | An ordering that changes under the user's finger | The snapshot cannot be retained long enough for a realistic scroll session |
| The list is genuinely static, such as an archived export | Offset is acceptable | Nothing, because nothing mutates | Anything a user can write to while reading |
| The consumer needs a jump-to-page control | Offset or page number, with the instability stated to the product owner | Nothing. This is a product requirement overriding a correctness preference | You could have satisfied the requirement with a filter or a search instead |

**Local storage tier**

| If | Choose | What it costs | Do not choose it when |
|---|---|---|---|
| A handful of small scalars: flags, last sync token, user preference | Key-value store | No query capability, and it is usually loaded wholesale | The data grows with the user's content |
| Structured content the user reads offline, queried by more than one key | Embedded database | Schema migrations you must run on a device you cannot debug | The dataset is small enough to refetch in under a second on the target link |
| Tokens, keys, anything an attacker on the device must not read | Platform secure storage | A slower API and platform-specific behaviour around backup and restore | The value is public. Secure storage is not a general-purpose cache |
| Decoded images and short-lived derived state | Bounded in-memory cache with explicit eviction | It is gone on process death, by design | You need it after a restart, which means it belongs on disk as encoded bytes |
| Large binary content: media, downloads, encoded image bytes | Disk cache in a directory the OS may reclaim | The OS can delete it without telling you, so it must be re-derivable | It is the only copy of something the user created |

**Sync strategy**

| If | Choose | The number that decides it | Do not choose it when |
|---|---|---|---|
| One writer, small dataset | Full refetch on open, plus a durable outbox for writes | Dataset bytes against the bytes-per-screen budget. Under roughly 100 KB, refetching is invisible | Two devices write the same record concurrently |
| One writer, large dataset | Delta sync with a change token, plus tombstones for deletions | Tombstone retention must exceed the longest offline period you support | You cannot bound how long a client may be offline, in which case add a full-resync fallback |
| Several writers, last writer acceptable | Server-assigned monotonic version, stale writes rejected with the current state | Conflict rate per record per day. If it is near zero, this is enough | Losing a writer's text silently is unacceptable, which it usually is |
| Several writers, no acceptable loser | Operation log with a merge rule, or a conflict-free replicated data type | Operations per record per session, and the size of the metadata each type carries | One writer. The metadata and the reasoning cost are permanent and the benefit is zero |

**Where a computation runs**

| If | Choose | Because | Do not choose it when |
|---|---|---|---|
| It writes a view property and nothing else | Main thread | It has to be there, and it costs under a millisecond | It parses, formats, measures text or touches disk |
| It is per-item work that scales with page size | Background thread at page arrival, result cached on the model | The frame budget is 8 to 17 ms and item binding must be a set of assignments | The result depends on layout that has not happened yet |
| It needs data the device does not have, or compute the device cannot afford | Server | The device pays bytes instead of milliseconds, and you can change it without a release | The feature must work offline, which is usually the actual requirement |
| It must work offline and the model fits the tier | On-device, off the interaction path, versioned separately from the binary | A new model ships without a store release, and a bad one reverts in minutes | The device cannot run it, which is the default path and needs a server-supplied fallback |

### 16b. The numbers this course uses, and where each comes from

Every row below appeared with its derivation in Sections 10 to 15. Numbers introduced inside the modules and the case studies carry their derivations there. Inputs marked assumed are chosen because they are typical for the scenario as stated; every derived line survives as a method when you substitute your own measurement.

| Number | Derivation | What it eliminates |
|---|---|---|
| 16.7, 11.1 and 8.3 ms frame budgets | 1000 divided by 60, 90 and 120 | Quoting one frame budget for every device |
| About 7.8 ms of frame budget that is yours at 90 Hz | 11.1 ms times an assumed 70 percent application share, the rest going to input, compositor and render thread | Synchronous image decode on the main thread, at an assumed 8 ms per decode on the mid tier |
| 1,960 ms serial cold start | 260 plus 240 plus 220 plus 180 plus 160 plus 900, each measured per phase | Nothing on its own; it earns its place next to the split below |
| 1,060 ms of on-device cold start work | The same sum with the 900 ms network fetch removed | A faster API as the primary cold start fix, since a network of zero still misses a 1,200 ms target on the tier below |
| 750 ms of on-device budget | A 1,200 ms target divided by an assumed 1.6 times low-tier multiplier | A design that only just clears on the reference device |
| 1.77 MB for one feed screen | 8 cards times 1.2 KB of JSON plus 8 times 220 KB of images | Nothing on its own; pair it with the link rate |
| 35.4 seconds to load that screen | 1.77 MB times 8 bits per byte, divided by 0.4 Mbit per second | Waiting for images before showing a screen, and prefetching a second screen while the first is loading |
| 78 KB per resized card image | 1,080 by 600 display pixels times an assumed 0.12 bytes per pixel, which you must measure on your own imagery | Shipping one image size to every device |
| 2.59 MB decoded per card image | 1,080 times 600 times 4 bytes at 32-bit colour | Sizing a bitmap cache from encoded bytes |
| 32 MB bitmap budget | 256 MB of process headroom on the mid tier, with one eighth allocated deliberately | Unbounded in-memory caches |
| 12 images resident | 32 MB divided by 2.59 MB | A memory-only image cache, since 12 is about a screen and a half |
| 48.8 MB for a full-resolution decode | 4,032 times 3,024 times 4 bytes | Decoding at source resolution, since one decode is one and a half times the whole bitmap budget |
| 2,564 images on a 200 MB disk tier | 200 MB divided by 78 KB of encoded bytes | Storing decoded frames on disk, which would hold 77 |
| 2,880 polls per device per day | 86,400 seconds divided by a 30 second interval | Nothing yet; see the two rows below |
| About 333,000 requests per second | 2,880 polls times 10 million devices, divided by 86,400 | A 30 second poll as the primary chat transport, on server cost |
| 6.4 hours of elevated radio per device per day | 2,880 polls times an assumed 8 second radio tail | The same poll, independently, on battery |
| About 11,000 requests per second for the safety net | 96 polls a day times 10 million devices, divided by 86,400 | The claim that any polling at all is unaffordable |
| 360 messages per connection per hour | 6 updates a minute times 60 minutes | Polling for a live score screen, since above about 10 an hour a held connection wins |
| 800,000 concurrent sockets | 10 million daily actives times an assumed 8 percent peak foreground share | The claim that socket concurrency is the hard part of a chat client |
| About 60 KB for a saved-items list | A few hundred rows at an assumed 200 bytes each | A sync engine with an operation log for single-writer data |
| 10 ms to score a page on device | 50 candidates times an assumed 0.2 ms per candidate on the mid tier | Running inference on the main thread, and running it per scroll event |

### 16c. ASCII glyph legend

```
ASCII CONVENTIONS USED IN EVERY DIAGRAM ON THIS PAGE

 +--------+   solid box: code inside the app binary, which you
 | NAME   |   cannot change without a release
 +--------+

 +========+   double edge: durable state on the device that
 | NAME   |   survives process death: local store, outbox,
 +========+   disk cache, secure storage

 +- - - - +   dashed edge: something outside the binary that can
 | NAME   |   change without a release: server, remote config,
 +- - - - +   push service, feature flag definitions

   --->       a user is waiting for this to complete
   ==>        deferred or scheduled work, nobody is waiting
   <-->       two copies of one contract that must agree
   - ->       best effort, may never arrive
   |   v      vertical continuation of the same path

 Caption: box style says whether you can change the thing
 without shipping a release; arrow style says whether a user
 is waiting on it.
```

### 16d. One-page summary cards

Eleven cards, one per case study, in the order the case studies appear.

**Card 1. News feed client**

| Field | Content |
|---|---|
| Prompt | Design the client for an infinitely scrolling feed |
| Budget that decides | Bytes for the first screen on a 400 kbps link, and the frame budget at the device's refresh rate |
| Number that decides | 1.77 MB of unresized screen is 35.4 seconds; resized to 78 KB an image it is 12.5 seconds, and JSON alone lands in under half a second |
| V0 | One request per page, render text on arrival, load images in view order, memory cache only |
| Breaking point | Scrolling back re-downloads everything, and a fling drops frames; observe as image cache hit rate near zero and a frame timing histogram with a tail during flings |
| V1 | Three cache tiers (memory, disk, network) with cancellation on view recycle, keyset pagination with a tie-break, and precomputed bind data |
| Deep dive one | Pagination stability: what twelve head inserts and three deletions do to each scheme, and why the local store is keyed by item identifier |
| Deep dive two | The image pipeline: decode and downsample, the 33 times encoded-to-decoded expansion, eviction, and prefetch depth as remote config |
| Failure class | Memory kill from full-resolution decode, and prefetch starving the visible screen on a slow link |
| Staff delta | Prefetch depth and image quality behind remote config, a byte budget enforced in the release pipeline, and a degradation path to the cached page with its age shown |

**Card 2. Chat client**

| Field | Content |
|---|---|
| Prompt | Design a messaging client that works on two devices and on a bad connection |
| Budget that decides | Message visible within 2 seconds foreground, about a minute backgrounded, on a battery that must last the day |
| Number that decides | A 30 second poll is 2,880 requests and 6.4 hours of radio per device per day, which eliminates polling twice over |
| V0 | Single device, socket in the foreground, messages ordered by server receipt, no local queue |
| Breaking point | The user sends with no connection and the message vanishes; observe as sent-message counts that do not reconcile with server receipts |
| V1 | Durable outbox with a client-generated identifier, socket foreground, push as a background wakeup, one delta sync on resume |
| Deep dive one | Multi-device ordering: why the device clock is not the sort key, and what a server-assigned per-conversation sequence buys |
| Deep dive two | The transport ladder and its handoffs: socket, push, delta sync on resume, and a low-frequency poll as the named safety net |
| Failure class | Clock skew amplified by at-least-once delivery into visible duplicates, and a reconnect thundering herd after an outage |
| Staff delta | Idempotency key retention on the server sized to the client's attempt deadline, a kill switch on the socket that falls back to push plus sync, and a stated rollback threshold on message delivery latency |

**Card 3. Maps client**

| Field | Content |
|---|---|
| Prompt | Design a map client with offline regions and live positioning |
| Budget that decides | Battery over a long session, and disk for offline regions |
| Number that decides | Location update rate against battery: the design question is the duty cycle, not the accuracy setting |
| V0 | Fetch tiles on demand, cache them on disk, request location at the highest accuracy while the map is visible |
| Breaking point | A long session drains the battery and an offline region download has no bound; observe as radio-on and location time attributed per subsystem over a fixed window |
| V1 | Tile pyramid with a bounded disk budget, offline regions downloaded on unmetered networks with a size estimate shown before download, and location duty-cycled to the zoom level and movement rate |
| Deep dive one | The tile pyramid: how the zoom level and viewport determine the tile set, and how an offline region is sized and bounded before the user commits |
| Deep dive two | Location duty cycling: measuring drain per subsystem, degrading in low-power mode, and what accuracy the screen actually needs at each zoom |
| Failure class | Battery drain from an un-duty-cycled sensor, and disk exhaustion from unbounded tile caching |
| Staff delta | A per-feature battery budget with an owner, the offline region size shown before the download starts, and a degradation path to the last cached tiles with a stale indicator |

**Card 4. Video client**

| Field | Content |
|---|---|
| Prompt | Design a video playback client with adaptive bitrate and offline download |
| Budget that decides | Time to first frame, rebuffer rate, and bytes on a metered connection |
| Number that decides | Buffer depth in seconds against the link rate: the buffer must cover the worst gap you expect, and every second of buffer is bytes you may never play |
| V0 | One quality level, buffer a fixed number of seconds, play |
| Breaking point | The link rate moves and playback stalls or the quality is needlessly low; observe as rebuffer events per playback hour split by connection class |
| V1 | A bitrate ladder with a switch policy driven by measured throughput and buffer depth, plus a separate download path for offline |
| Deep dive one | The buffering ladder: what triggers a switch up and down, why the two thresholds differ, and why switching on a single measurement oscillates |
| Deep dive two | Prefetch and download policy on a metered connection, including what happens when the user leaves mid-download and the process is killed |
| Failure class | Oscillating quality from a switch policy with no hysteresis, and a download that cannot resume after process death |
| Staff delta | The ladder and its thresholds behind remote config, rebuffer rate as the release health metric with a rollback threshold, and a degradation path to the lowest rung rather than to a stall |

**Card 5. Ride hailing client**

| Field | Content |
|---|---|
| Prompt | Design the rider client for the request, match and trip flow |
| Budget that decides | Correctness of the trip state under process death, and location cost over a trip |
| Number that decides | The number of states in the trip machine and the transitions that are not idempotent, because each one is a duplicate charge or a lost trip |
| V0 | Request a ride, poll for status, show the driver on a map |
| Breaking point | The process is killed mid-trip and the client comes back not knowing what happened; observe as sessions that resume into an inconsistent state, counted explicitly |
| V1 | An explicit trip state machine persisted on every transition, server as the source of truth, and a reconciliation fetch on every resume |
| Deep dive one | The trip state machine: every transition, which are client-initiated, and what the client shows when it does not know |
| Deep dive two | Race conditions at the edges: cancel racing with match, double request from a retried tap, and the idempotency key that makes both harmless |
| Failure class | Split state between client and server after process death, and a duplicate request from a retried write with no key |
| Staff delta | Reconciliation on resume treated as the normal path rather than recovery, a kill switch on the new matching flow, and a degradation path that shows the last known state with its age |

**Card 6. Trading client**

| Field | Content |
|---|---|
| Prompt | Design a trading client with live prices and order entry |
| Budget that decides | Render cost under a high-rate update stream, and correctness of an order under an ambiguous timeout |
| Number that decides | Updates per second against the frame budget: above roughly one update per frame per row, you are rendering states no human can see |
| V0 | Stream every tick, update the row on arrival, submit orders over plain requests |
| Breaking point | A volatile minute produces more updates than frames and the main thread saturates; observe as dropped frames correlated with message rate, not with user input |
| V1 | Coalesce ticks into one render per frame per row, and an order path with a client-generated key, an explicit pending state and a reconciliation query |
| Deep dive one | Tick coalescing: buffering updates and applying the latest per row per frame, and why dropping intermediate states is correct here and wrong in a chat |
| Deep dive two | Order integrity under an ambiguous timeout: the key, the pending state, the query that resolves it, and what the user is allowed to do while it is unresolved |
| Failure class | Main thread saturation from an unthrottled stream, and a duplicate order from a retried submit |
| Staff delta | A stated maximum render rate with the product consequence spelled out, a kill switch on the streaming transport that falls back to a slower refresh, and an explicit rule for what the client shows when it cannot confirm an order |

**Card 7. Image loading library**

| Field | Content |
|---|---|
| Prompt | Design an image loading library for other teams to embed |
| Budget that decides | The host app's bitmap budget and frame budget, not yours |
| Number that decides | 2.59 MB decoded against 78 KB encoded, and a 32 MB budget that holds 12 images |
| V0 | Fetch, decode, set on the view, keep a memory cache |
| Breaking point | Recycled views show the wrong image and the process is killed on a large source; observe as memory kills concentrated on the low tier and mismatched images reported as a visual bug |
| V1 | Request keyed to the view with cancellation on recycle, decode with downsampling to display size, three tiers with explicit eviction |
| Deep dive one | The public API and threading contract: what is guaranteed about which thread a callback arrives on, what cancellation means, and how the caller expresses a target size |
| Deep dive two | Decode and downsample: reading the source dimensions before allocating, choosing a sample size, and reusing buffers rather than allocating per image |
| Failure class | Memory kill from full-resolution decode, and stale results delivered to a recycled view |
| Staff delta | A stated memory ceiling the host can set, a binary size budget, and an additive-only public surface within a major version because forty teams upgrade on forty schedules |

**Card 8. Analytics SDK**

| Field | Content |
|---|---|
| Prompt | Design an analytics SDK embedded by many first-party apps |
| Budget that decides | Under 5 ms of main thread at initialisation, zero disk or network on the caller's thread, a bounded disk queue, a stated binary size |
| Number that decides | The 90 ms startup regression caused by one synchronous flush during initialisation |
| V0 | Buffer events in memory, upload on a timer |
| Breaking point | Events are lost on process death and the upload competes with the host's own traffic; observe as event counts that do not reconcile with session counts |
| V1 | Durable disk queue with a byte cap and oldest-first eviction, lazy open on first event, batched upload at the lowest quality-of-service tier |
| Deep dive one | The durable queue: byte cap, eviction policy, and reporting the eviction count so silent loss becomes visible loss |
| Deep dive two | Quality-of-service tiers and the initialisation contract, including the hook that lets the host say it is latency sensitive right now |
| Failure class | Host startup regression from work on the caller's thread, and radio contention starving the host's own requests |
| Staff delta | The host's budgets written down as the SDK's tested requirements, defaults that meet them without configuration, and a documented deprecation policy for the public surface |

**Card 9. File download manager**

| Field | Content |
|---|---|
| Prompt | Design a download manager with resumability and background completion |
| Budget that decides | Completing across process death and OS background limits, on a link that drops |
| Number that decides | The expected uninterrupted window on the target link against the transfer time, which is what forces chunking rather than one request |
| V0 | One request, write to disk, show progress |
| Breaking point | The connection drops at 90 percent and the whole transfer restarts; observe as bytes transferred divided by bytes delivered, which should be near 1 and will not be |
| V1 | Ranged requests with a persisted offset, integrity verification on completion, and work scheduled through the OS's own deferrable-work mechanism |
| Deep dive one | Resumability: range requests, the validator that proves the remote file did not change mid-download, and where the offset is persisted so a kill costs one chunk |
| Deep dive two | Background execution limits: what the OS will and will not let you finish, how work is expressed as constraints rather than as a thread, and what the user sees when the OS defers it |
| Failure class | Lost transfer state on process termination, and a corrupted file from a source that changed between chunks |
| Staff delta | A queue with an explicit concurrency and priority policy, a kill switch that pauses all downloads server-side, and a degradation path that keeps partial files rather than deleting them |

**Card 10. Feature flag SDK**

| Field | Content |
|---|---|
| Prompt | Design a feature flag client with offline evaluation |
| Budget that decides | Evaluation with no network, and agreement with the server's answer for the same user |
| Number that decides | Staleness in minutes that the product will accept, which decides refresh cadence and whether evaluation is local or remote |
| V0 | Fetch flag values on launch, hold them in memory, evaluate by lookup |
| Breaking point | First launch has no values and the app behaves as if every flag is off, and a user is bucketed differently on two devices; observe as flag exposure counts that disagree with the server's assignment counts |
| V1 | Bundled defaults for first run, a cached snapshot on disk, deterministic local bucketing from a stable identifier, and a refresh with a staleness bound |
| Deep dive one | Deterministic bucketing: hashing a stable identifier with the flag key so client and server agree without talking, and what changes when the identifier changes |
| Deep dive two | Version skew of flag definitions: what an old app does with a flag it has never heard of, and what a new app does when the server still serves the old schema |
| Failure class | Cold-start divergence between local defaults and server values, and an exposure logged for a user who never saw the feature |
| Staff delta | Exposure logging as a first-class durable event, a kill switch that is itself not behind a flag, and a stated rule that a flag whose value cannot be fetched fails to the safe side |

**Card 11. Brownfield: cold start regressed 40 percent**

| Field | Content |
|---|---|
| Prompt | Cold start got 40 percent worse after a feature merge. Diagnose it and design the fix |
| Budget that decides | p90 cold start by device tier, not the mean and not p50 |
| Number that decides | The split between on-device work and network on the critical path, because it decides whether the fix is code or contract |
| V0 (what exists) | A single startup path with every initialiser eager on the main thread and a blocking first fetch |
| Breaking point | The regression is already shipped, so the first constraint is that you cannot patch the installed fleet quickly; observe as p90 by tier and by app version |
| V1 | Deferred initialisation with an explicit dependency order, the first fetch removed from the critical path, and a regression gate in the release pipeline |
| Deep dive one | The measurement and bisection plan: which trace, which percentile, which segment, and what you expect to see if each hypothesis is true |
| Deep dive two | Deferred initialisation: what must run before the first frame, what can run after, and the dependency ordering that makes deferral safe rather than a crash |
| Failure class | Eager main-thread initialisation, and a memory footprint increase that shows up as more cold starts rather than slower ones |
| Staff delta | A cold start budget enforced as a merge gate, a kill switch on the merged feature so the hypothesis can be tested without a release, and the explicit statement of what is not being changed this quarter |

---
## 17. What this course does not cover

Naming our limits is cheaper than pretending we have none, and it stops you studying the wrong thing. Where we point at a book or a standard we name the edition and the year, so you can tell what has aged.

| Not covered | Why | Where to go instead |
|---|---|---|
| Writing Swift, Kotlin, Dart or Objective-C, and language or framework trivia | A different round with a different rubric. Nothing here improves by knowing a language feature | Your platform's own language reference, plus the [data structures and algorithms bank](/Interview-Questions/data-structures-algorithms/) for the coding round |
| Implementing a screen: layout systems, animation craft, gesture recognisers, custom drawing | This course designs the system behind the screen. Building the screen is a live coding round, not a design one | Your platform's own user interface documentation, checked on the day, because these APIs move faster than a page can be reviewed |
| The server side of every prompt on this page: services, stores, partitioning, consensus, capacity | Assumed as the other half of the contract rather than taught here | Our [Modern System Design](modern-system-design.md) course, and Designing Data-Intensive Applications, Martin Kleppmann, 1st edition, O'Reilly, 2017 |
| The API contract itself: resource modelling, error taxonomies, webhook delivery, deprecation policy | We consume a contract and say what a client needs from it. Designing the contract is its own round | Our [Product Architecture](product-architecture.md) course |
| Choosing a model, designing labels, ranking metrics, training loops, even for an on-device model | We design how a model reaches a device, where it runs and what happens without it. What the model is remains a different discipline | Our [ML System Design](ml-system-design.md) course |
| Browser applications and web component design | Different runtime, different budgets, and a deployment model where you can patch in minutes | Our [Frontend System Design](frontend-system-design.md) course |
| Accessibility implementation depth: authoring patterns per control, focus order, screen reader behaviour | We name it as a scope decision you should declare out loud. Doing it properly is a track, not a paragraph, and it is largely platform specific | The Web Content Accessibility Guidelines 2.2, published as a W3C Recommendation in October 2023, for the principles, plus your platform's own accessibility documentation for the implementation |
| Offensive mobile security: reverse engineering, runtime tampering, obfuscation product selection | Genuinely important, adversarial, and it changes faster than a reviewed page can track | The OWASP Mobile Application Security project (the verification standard and the testing guide); check the current version number on the day, because it is revised on its own schedule |
| Privacy law and platform policy specifics: consent regimes, data residency, store review rules | Jurisdiction-specific and revised without notice. Generic advice here would be worse than none | Your own legal and policy function, and the store's current published policy, read on the day you need it |
| Cryptographic implementation: choosing primitives, key derivation, protocol design | We say where secrets belong and what pinning costs. Implementing cryptography is a specialist job and a common source of shipped defects | Your platform's own cryptography library documentation, and a specialist review before shipping |
| Cross-platform framework internals: rendering pipelines, bridge costs, plugin ecosystems | We give a column for where the cross-platform answer differs, and no more. The internals move fast enough that a dated page misleads | The framework's own documentation and release notes, with the version you are on written down |
| Games, real-time graphics, augmented reality, and wearable or automotive form factors | Different budgets, different input models, and different interview loops | Vendor documentation for the specific platform, and a specialist course if that is your target role |
| Per-platform profiling tool walkthroughs | We say which measurement confirms which diagnosis. Which button to press changes with every tool release | Your platform's own performance tooling documentation, checked on the day |
| Experiment design beyond deterministic bucketing and staged rollout | We teach enough to bucket consistently and to halt a rollout. Sizing and reading experiments is a field | Trustworthy Online Controlled Experiments, Kohavi, Tang and Xu, Cambridge University Press, 2020 |
| Resilience patterns at library depth: circuit breakers, bulkheads, load shedding | We use the client-side subset (retry classification, jitter, backpressure). The full catalogue is server-shaped | Release It!, Michael Nygard, 2nd edition, Pragmatic Bookshelf, 2018 |
| Behavioural rounds, team matching, levelling and compensation | Loop structures change per team and per quarter, and unverifiable claims about them are exactly what we refuse to publish | Ask your recruiter for the current format, in writing |

!!! note "On anything we cite"

    We name the edition and the year so you can tell what has aged. If a link here is dead, a newer edition exists, or a standard has been superseded, open an issue and it is fixed in the next review pass rather than sitting behind an unverifiable "recently updated" badge.

---

## 18. Provenance

**Last reviewed:** 1 August 2026. A page whose last-reviewed date is more than nine months old is flagged for review rather than quietly left in place.

**Changelog, most recent first**

| Date | Change |
|---|---|
| August 2026 | First publication: twenty modules, eleven case studies with the full arc and level deltas, eleven practice problems with public rubrics, and a mastery check mapped one to one against the seven learning objectives |
| August 2026 | Reference block separated from the teaching sections after review, so a reader returning for one decision table does not have to scroll through prose to reach it |
| August 2026 | Every figure in Sections 10 to 16 re-derived from inputs stated in the same section; figures that could not be derived were relabelled as assumptions with the reason each is reasonable, and the ones that eliminated no design option were deleted |

**Found a problem?** Open an issue at [github.com/datascienceinterviews/datascienceinterviews.github.io/issues](https://github.com/datascienceinterviews/datascienceinterviews.github.io/issues). Corrections, disputed numbers, dead links and better derivations are all in scope. Errors get fixed in days, and the changelog above records what changed.

All content on this page is original. Research informed which topics belong here; no prose, framework, acronym, example, table or diagram is derived from any paid course. Primary sources are cited inline with their edition and year, and any claim we could not source is either labelled as an assumption or is not on the page.
