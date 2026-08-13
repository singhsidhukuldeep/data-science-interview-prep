---
title: Frontend System Design Interview
description: A free course on both frontend design rounds: whole applications and single components, with rendering, state, accessibility and byte budgets as engineering.
last_reviewed: 2026-08-01
---

# Frontend System Design Interview

!!! abstract "The contract"

    **After this course you can** take a one-line frontend prompt, tell inside a minute whether you are being asked for an application or for one component, derive a design whose rendering strategy, state split, accessibility contract and byte budget each rest on a number you can recompute out loud, and name what breaks first on a mid-range phone.

    **Who it is for**

    - A product engineer with roughly 2 to 8 years of shipping browser interfaces, facing a 45 or 60 minute frontend design round.
    - A full stack or backend engineer who can draw the server side confidently and then stalls when the whiteboard is a screen: render strategy, cache keys, focus order, bytes.
    - A senior candidate whose feedback keeps saying "listed libraries" or "no numbers", and who cannot tell which minutes were spent on the wrong thing.

    **Who it is not for**

    | If your round is about | Go here instead |
    |---|---|
    | A native iOS or Android client: battery, background execution, over-the-air release, version skew | [Mobile System Design](mobile-system-design.md) |
    | Services, stores, queues, sharding, consensus, multi-region: the parts behind the network call | [Modern System Design](modern-system-design.md) |
    | The contract itself: domain model, resources, error taxonomy, idempotency, pagination, versioning | [Product Architecture](product-architecture.md) |
    | The quality of what the screen shows: ranking, recommendation, retrieval, abuse scoring | [ML System Design](ml-system-design.md) |
    | A product built on a foundation model: evaluation, guardrails, tokens and cost | [Generative AI System Design](generative-ai-system-design.md) |
    | A round inside the next week | [Fast-Track in 48 Hours](system-design-fast-track.md) |

    **Trains for:** the 45 and 60 minute frontend design round in web and product engineering loops, in both of the formats it ships as (an application, or a single component), plus the design portion of senior and staff frontend loops.

    **Does not train for:** frontend coding rounds (implement a debounce, build this from a screenshot), CSS layout screens, framework trivia, product sense rounds, or behavioural rounds.

    **Fast path:** already comfortable? Go straight to the [self-assessment rubrics](#11-self-assessment-rubrics). Just want the decision tables and the budgets? Go to the [reference block](#16-reference-block).

---

## Prerequisites

Seniority is a weak readiness signal in this round, because frontend experience varies enormously in what it touched: two engineers with the same years can have shipped very different surfaces. Prerequisite mastery is the signal that works. Answer every Quick check in under a minute. Two or more misses means start with the linked material rather than with Module 1.

| You should already be able to | Where to learn it | Quick check |
|---|---|---|
| Read a network waterfall and say which request delayed the first paint | [Chrome DevTools network reference](https://developer.chrome.com/docs/devtools/network/) | A stylesheet in the head, and a script at the end of the body carrying `defer`. Which one holds up the first paint, and why? |
| Say what blocks the browser main thread, and what the user sees while it is blocked | [Interaction to Next Paint, web.dev](https://web.dev/articles/inp) | A click handler runs for 300 ms. Describe what the user sees, and name the metric that records it |
| Convert compressed bytes into download seconds on a stated connection | [Estimation anchors on the hub](index.md#anchors-worth-memorising) | 600 KB of compressed JavaScript over an effective 1.6 Mbps connection. Roughly how many seconds before parsing even starts? |
| State what an HTTP cache directive does from origin to browser | [RFC 9111, HTTP Caching](https://httpwg.org/specs/rfc9111.html) | `Cache-Control: max-age=31536000, immutable` on a file named `app.a1b2c3.js`. What must be true about your build for that to be safe? |
| Name the role, the keyboard behaviour and the focus destination for a custom control | [W3C ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/patterns/) | A custom dropdown is open. Which key closes it, and where does focus go afterwards? |
| Read a percentile instead of an average | [Google SRE Book, chapter 6, free online](https://sre.google/sre-book/monitoring-distributed-systems/) | Interaction latency is 40 ms at p50 and 900 ms at p95. Is the complaining population 5 percent of users? |
| Trace one request from a browser through a CDN to an origin | [System design question bank](/Interview-Questions/System-design/) | Name the three hops in order, and say which one you can change without shipping a deploy |
| Say what causes a component to re-render, and what stops one | This page, Module 8 | A parent re-renders. Under what condition does its child not, and what is the commonest way that condition is accidentally broken? |

??? note "Show answers to the quick checks"

    1. The stylesheet. CSS is render-blocking: the browser will not paint until it has built the style rules, because it cannot know the final appearance without them. `defer` postpones script execution until after parsing, so it does not hold the first paint, although it can still delay the page becoming interactive.
    2. The page appears frozen. Nothing repaints, hover and focus styles do not update, and the browser cannot process the next input until the handler returns. The metric is Interaction to Next Paint, which measures from the input to the next frame the user actually sees, so all 300 ms lands inside it.
    3. 1.6 Mbps is 1,600,000 bits per second, so divide by 8 to get 200,000 bytes per second, which is roughly 200 KB per second. 600 divided by 200 is about 3 seconds of pure download, before parse, compile, execute, and before any request that this script goes on to make.
    4. The filename must change whenever the bytes change, which is what content hashing does, and nothing may ever serve different bytes under the same name. 31,536,000 seconds is 365 times 86,400, so one year. Get this wrong and a user holds a stale file for a year with no way for you to recall it.
    5. Escape closes it, and focus returns to the control that opened it. If focus lands on the document body instead, a keyboard user is dumped to the top of the page and has to tab all the way back, which is the single most common focus bug in custom widgets.
    6. No. At least 5 percent of interactions take 900 ms or more, which is not the same as 5 percent of users. Assume interactions are independent for a moment: a session with 20 interactions hits at least one slow one with probability 1 minus 0.95 to the power 20, which is about 64 percent. The assumption is rough, but it makes the point that the affected population is far larger than the percentile number.
    7. Browser to CDN edge, edge to origin on a miss, origin to whatever it reads from. The hop you can change without a deploy is the edge: cache keys, time to live, purge, headers and redirects are all configuration, which is why the edge is where you fix things at three in the morning.
    8. The child does not re-render when its inputs are unchanged and it is memoised, or in a fine-grained reactive framework when it did not read the value that changed. The commonest way to break it is passing a freshly created object, array or inline function as a prop, which is a new reference on every render, so the equality check always fails and the memoisation does nothing.

---

## Time budget

| Part of the course | Reading | Practice | Spaced review |
|---|---|---|---|
| Modules 1 to 3, which round, the clock, interface budgets | 60 to 78 min | 20 to 30 min | included below |
| Modules 4 to 5, rendering strategy and the hydration boundary | 70 to 94 min | 15 to 20 min | included below |
| Modules 6 to 7, component API design and accessibility | 90 to 120 min | 15 to 20 min | included below |
| Modules 8 to 10, state, caching and delivery, real-time | 110 to 146 min | 20 to 30 min | included below |
| Modules 11 to 13, performance, design systems, micro-frontends | 95 to 128 min | 20 to 30 min | included below |
| Modules 14 to 18, security, locales, testing, observability, levels | 115 to 154 min | 18 to 32 min | included below |
| Eight case studies, attempted before reading each arc | included above | 120 to 160 min | included below |
| Both practice sets, eleven problems | included above | 195 to 250 min | included below |
| Five review sessions on days 1, 3, 7, 21 and 60 | 0 | 0 | 180 to 240 min |
| **Total** | **540 to 720 min, so 9 to 12 h** | **423 to 572 min, so 7 to 10 h** | **180 to 240 min, so 3 to 4 h** |

!!! note "How these were computed, so you can argue with them"

    Reading is summed per module from the Module Map below, never guessed for the page as a whole. The eighteen module ranges total 540 minutes at the low end and 720 at the high end, which is the 9 to 12 hour figure. Per-module minutes assume 120 to 150 words a minute, a deliberately slow engaged rate, because prose carrying diagrams, decision tables and arithmetic is not read at prose speed.

    Practice itemises to three components that sum to the total:

    - Faded examples inside the eighteen modules, 6 to 9 minutes each, so 108 to 162 minutes. That is the six module rows above.
    - Eight case studies, 15 to 20 minutes attempting each before reading its arc, so 120 to 160 minutes.
    - Practice sets, five blocked problems at 15 to 20 minutes each (75 to 100) plus six interleaved problems at 20 to 25 minutes each (120 to 150), so 195 to 250 minutes.

    Adding those three gives 423 to 572 minutes, which is the practice column total.

    Review is five sessions of 36 to 48 minutes, which is 180 to 240 minutes. Every figure carries roughly 15 percent padding, because self-estimates of study time run optimistic. Minutes are the authoritative unit here: one headline number would be more marketable and less true. At an hour a day this is three weeks, full time it is four days.

---

## Learning objectives

Each objective is tested by exactly one numbered item in the mastery check, and the numbering matches: objective 1 is tested by mastery item 1.

1. **Classify** a frontend prompt as application-scale or component-scale within the first sixty seconds, **naming the two signals in the interviewer's wording that decided it**, and stating the single clarifying question that settles a tie.
2. **Choose** a rendering strategy from client, server, static, incremental, streaming and islands for a stated first-render budget, personalisation level and device profile, **naming the number that decides it** and the specific cost the rejected strategy would have carried.
3. **Derive** a delivery budget from a stated device and network profile, giving JavaScript bytes allowed on the critical path, a first-render target and an interaction-response target, **showing the arithmetic that converts bytes into milliseconds** at the stated bandwidth.
4. **Specify** the public interface of an interactive component, covering controlled versus uncontrolled state, the state machine, and the keyboard and focus contract, **to the point where a second engineer could implement it without asking you a behavioural question**.
5. **Separate** a screen's state into client, server-cache and shared categories, and **state for every server-cache entry its cache key, its staleness tolerance in seconds and the event that invalidates it**.
6. **Diagnose** a described interaction-latency regression into exactly one named cause from long task, render cascade, unshipped code split, blocking third party or layout thrash, and **name the single measurement that confirms it**, in under three minutes.
7. **Rewrite** a mid-level frontend answer into a staff-level one by adding a performance budget enforced in continuous integration, a conformance target with its testing method, a rollout and kill-switch plan, and one degradation path, **annotating what each addition changed about a decision already made**.

---

## Warm-up retrieval

Three to five minutes. Answer out loud or in writing before you open anything else. Retrieval beats rereading, and a visible answer quietly converts one into the other.

**Q1.** From the hub's round router: name two signals inside the first minute that tell you this is a frontend design round rather than a generalist distributed-systems round wearing a frontend title.

??? note "Show answer"

    First, the artefact you are asked to draw is a screen or a widget, not a service, a store or a pipeline. Second, the follow-ups are about what the user sees and when: first paint, scroll behaviour, what happens on a slow connection, what a keyboard user does.

    A third, weaker signal is that the interviewer accepts an API as a given rather than asking you to design it. When the follow-ups instead go to replication, sharding or consistency, you are in a backend round, and Module 1 has the script for steering back.

**Q2.** From the trade-off atlas on the hub, several pages back: state the client rendering versus server rendering rule, and name the two numbers that decide it.

??? note "Show answer"

    Client rendering when the page is highly interactive after load, sessions are long, and the audience is on capable devices. Server rendering when first paint or crawlability matters, or when the audience is on low-end devices and slow networks.

    The two deciding numbers are the first contentful paint budget in milliseconds and the JavaScript bytes shipped on the critical path. The failure each way: client rendering ships a blank screen to a slow phone, and server rendering moves per-request CPU cost onto a fleet you now have to size and pay for.

**Q3.** The hub states one hard rule about when a number may appear in your answer, and one situation in which capacity math is theatre. State both, and say what replaces server-side request-rate math in a client-side round.

??? note "Show answer"

    The rule: a number may appear only if the next sentence uses it to rule out a design option, and if nothing is eliminated the number is deleted. The exception is a number explicitly labelled an assumption because a later step depends on it, carrying one sentence on why it is reasonable.

    Capacity math is theatre when the prompt is semantics-centric or fits on one machine by inspection. In a client-side round, the equivalent gate is the device and network budget: bytes shipped on the critical path, main-thread milliseconds, memory ceiling, and what the interface does with no connection.

---

## Module map

```
+--------------------------------------------+
| FRAME                             M1 to M3 |
| which of the two rounds you are in, the    |
| clock, and budgets for an interface        |
+------+----------------------+--------------+
       |                      |
       v                      v
+------------------+   +---------------------+
| RENDER           |   | COMPONENT           |
| M4 strategy      |   | M6 public API and   |
| M5 server and    |   |    state machine    |
|    client line   |   | M7 accessibility    |
|                  |   |    contract         |
+------+-----------+   +------+--------------+
       |                      |
       +--------------+-------+
                      v
+--------------------------------------------+
| DATA                             M8 to M10 |
| state split three ways, HTTP and browser   |
| caching, transports and offline queues     |
+---------------------+----------------------+
                      v
+--------------------------------------------+
| SPEED AND REUSE                 M11 to M13 |
| metrics, budgets in CI, design systems,    |
| micro-frontends and when to refuse them    |
+---------------------+----------------------+
                      v
+--------------------------------------------+
| HARDEN AND JUDGE                M14 to M18 |
| browser security, locales, testing and     |
| rollout, observability, level calibration  |
+--------------------------------------------+

Caption: six clusters and their reading order. Frame splits into
two branches because this course trains two round formats: Render
is the application branch, Component is the widget branch. Both
rejoin at Data, because state and caching decide either one. Data
feeds Speed and Reuse, which feeds Harden and Judge, where mid
and staff answers separate. Arrows run prerequisite to dependent.
```

| Module | What you will be able to do | Time | Skip if |
|---|---|---|---|
| M1 Scope and honesty | Tell inside the first minute whether the round is application-scale or component-scale, and steer back when the title and the questions disagree | 12 to 16 min | Your recruiter told you the format in writing and the interviewer stuck to it |
| M2 The delivery clock | Run two different minute budgets, one per format, and switch between them mid-round without visibly restarting | 20 to 26 min | You have run a timed self-mock in both formats and finished each with time left |
| M3 Requirements for an interface | Turn a vague screen prompt into a functional list plus budgets: first render, interaction response, bytes, conformance level, devices, locales | 28 to 36 min | Never skip. Every case study starts here, and this is where the round is usually lost |
| M4 Rendering strategy | Choose among client, server, static, incremental, streaming and islands on a stated budget, and say what the rejected option would have cost | 42 to 56 min | You have shipped at least two of these strategies and can state the first-paint number that separates them |
| M5 The server and client boundary | Say what actually ships to the device, spot a server data waterfall in a component tree, and flatten it | 28 to 38 min | You have profiled and fixed a server-side fetch waterfall yourself |
| M6 Component design | Design a component's public interface, decide controlled versus uncontrolled, and draw the state machine underneath it | 45 to 60 min | Never skip if any of your target companies runs the component format, which most do |
| M7 Accessibility as engineering | Attach an authoring pattern, a focus plan and a testing method to every interactive control you draw | 45 to 60 min | You have shipped a widget against a published authoring pattern and tested it with a screen reader |
| M8 State, split three ways | Sort a screen's state into client, server-cache and shared, and give every cached entry a key, a staleness tolerance and an invalidation trigger | 40 to 54 min | You already draw this split by habit and can name your invalidation triggers from memory |
| M9 Data fetching and caching | Design the delivery path end to end: HTTP cache semantics, content-hashed assets, CDN keys and purge, browser storage tiers, service worker strategy | 38 to 50 min | You have designed a purge strategy and know what your storage tiers cost on a cold start |
| M10 Network and real-time | Pick a transport on message rate and direction, then design reconnection, ordering, deduplication and an offline outbound queue | 32 to 42 min | You operate a live-updating interface and can state its reconnect and dedup rules |
| M11 Performance | Name which metric blames which cause, split code by route and component, virtualize a large list, and enforce a budget in continuous integration | 45 to 60 min | You have attributed a long task to a specific line and shipped the fix, with a before and after number |
| M12 Design systems | Design token architecture and component versioning, and theme without a flash of the wrong colours | 28 to 38 min | You maintain a component library with a published deprecation policy |
| M13 Micro-frontends, honestly | Compare integration strategies on bundle duplication, version skew, routing ownership and error isolation, and refuse them out loud when the honest answer is no | 22 to 30 min | You have already said no to this once, with a reason that was not aesthetic |
| M14 Security in the browser | Name the browser attack classes, and attach the specific header, attribute or policy that closes each one | 28 to 38 min | You have written a content security policy that survived contact with a real third-party script |
| M15 Internationalisation | Design message formats, plurals, right-to-left mirroring, locale-aware formatting and time zone correctness into the component API rather than around it | 22 to 28 min | You have shipped a right-to-left locale and a date across a time zone boundary |
| M16 Testing and rollout | Decide what to unit, integration and end-to-end test in an interface, add accessibility and visual assertions to continuous integration, and design a kill switch | 28 to 38 min | You own a test pyramid you designed and a flag you have actually killed in production |
| M17 Observability | Design real user monitoring with sampling and attribution, error boundaries, source maps in production, and state the privacy cost of session replay | 20 to 28 min | You have diagnosed a production-only frontend bug from your own telemetry |
| M18 Level calibration | Say what your answer is missing to read one level higher | 17 to 22 min | Never skip if you are targeting senior or above |

Low ends total 540 minutes, high ends total 720. That is the 9 to 12 reading hours in the time budget above.

---

## The delivery clock for this round

This page takes a position on how the round is lost, and its whole structure follows from that position. The failure is rarely a missing fact. It is naming libraries, drawing a component tree that would fit almost any prompt, and never committing to a rendering strategy, a byte budget, a keyboard model or a way of being wrong. This round is also the only one in the section with two genuinely different clocks, because "design a news feed" and "design an autocomplete" want their minutes spent in different places.

!!! tip "Which clock am I on?"

    Application-scale prompts name a product or a screen: a feed, a dashboard, a checkout. Component-scale prompts name a widget and expect an interface: an autocomplete, a data grid, a modal. If the prompt is ambiguous, ask once, in one sentence: "Do you want the whole surface, or do you want me to design this as a reusable component with a public API?" That question costs eight seconds and redirects the next forty minutes.

### The 45 minute application round

| Minutes | Spend it on | You leave this phase with |
|---|---|---|
| 0 to 2 | Restate the prompt as a screen and a user job, and name which format you think you are in | Agreement, or a cheap correction |
| 2 to 8 | Clarify, then write the functional list plus budgets: first render, interaction response, bytes, conformance level, device and locale range | Three or four numbers tagged GIVEN or ASSUMED |
| 8 to 11 | Sizing that eliminates: bytes on the critical path, rows on screen at once, messages per second | One or two options already dead |
| 11 to 19 | v0 drawn: rendering strategy, component tree, data flow, the one API shape you need | A correct interface at the stated budget, on the board |
| 19 to 28 | The breaking point, then v1 | A named metric, a threshold, and the one thing you changed |
| 28 to 38 | One deep dive, ideally the one the interviewer picks | Depth in a single area, with the trade-off stated both ways |
| 38 to 45 | Accessibility contract, degradation path, what you would measure first, what you skipped | An explicit list of what you left out, and why |

The phases sum to 45 by construction: 2 plus 6 plus 3 plus 8 plus 9 plus 10 plus 7.

### The 45 minute component round

Same slot, very different shape. There is no capacity phase and no architecture diagram. The public interface is the artefact, and roughly half the clock belongs to behaviour rather than structure.

| Minutes | Spend it on | You leave this phase with |
|---|---|---|
| 0 to 2 | Restate the component as a contract: what it renders, what it owns, what the caller owns | A scope both of you agree is a component, not an app |
| 2 to 7 | Clarify usage sites, data volume, controlled or uncontrolled, framework, conformance target | The three constraints that will decide the API |
| 7 to 14 | The public interface: inputs, events, slots, imperative handles, and the controlled decision | A signature on the board that a caller could read |
| 14 to 21 | The state machine and the keyboard model, drawn as states and transitions | Every key accounted for, and focus destinations named |
| 21 to 29 | The data path: fetching, debounce, cancellation, caching, error and empty states | A race condition named and closed |
| 29 to 37 | Accessibility contract plus one performance concern taken to real depth | One area where you clearly went deeper than a checklist |
| 37 to 45 | Testing plan, how the API evolves without breaking callers, what you skipped | A versioning story, which almost nobody offers |

These sum to 45: 2 plus 5 plus 7 plus 7 plus 8 plus 8 plus 8.

### The 60 minute rounds

| Minutes | Application-scale | Component-scale |
|---|---|---|
| 0 to 2 | Open and restate | Open and restate as a contract |
| 2 to 10 | Clarify plus the full budget list | Clarify, 2 to 8 |
| 10 to 14 | Sizing that eliminates | Public interface, 8 to 17 |
| 14 to 23 | v0 end to end | State machine and keyboard model, 17 to 25 |
| 23 to 34 | Breaking point and v1 | Data path, 25 to 34 |
| 34 to 50 | Two deep dives, different in kind | Accessibility contract at depth, 34 to 44 |
| 50 to 57 | Accessibility, security, observability, rollout | One performance dive, 44 to 54 |
| 57 to 60 | Close: trade-offs, skips, first measurement | Testing, versioning, deprecation, skips, 54 to 60 |

The application column sums to 60: 2 plus 8 plus 4 plus 9 plus 11 plus 16 plus 7 plus 3. The component column also sums to 60: 2 plus 6 plus 9 plus 8 plus 9 plus 10 plus 10 plus 6. In both cases the extra 15 minutes buys depth and a rollout story. It does not buy more boxes.

### The first two minutes

Four sentences, in this order. Rehearse them once so they run without attention and you can spend yours on listening.

1. "Let me restate it: users do X on this surface, and the thing I think is hard is Y." (If you cannot name a hard part, that is your first clarifying question.)
2. "Before I draw: do you want the whole surface, or this as a reusable component with a public API?" (This is the highest-value eight seconds in the round.)
3. "I am going to spend about six minutes on requirements and budgets, then draw it, then break it on purpose." (You just told them your clock, which reads as control.)
4. "Stop me if you want depth somewhere specific rather than coverage." (Interviewers take this invitation more often than candidates expect.)

### Declaring what you are skipping

Skipping in silence reads as not knowing. Skipping out loud reads as prioritising. The pattern: name it, say why it cannot change a decision in the next twenty minutes, say in one sentence what you would do if it were in scope.

- "I am not designing the API. I will assume a cursor-paginated endpoint, and I will state what I need from it: stable ordering under inserts, and a total count only if the design turns out to need one."
- "I am skipping the theming layer. It does not touch the rendering strategy or the data path, and at this scope it is a token naming exercise rather than an architecture one."
- "I will cover analytics in one line: it goes on an idle callback, and it is the first thing I delete if the third-party byte budget is blown. Tell me if you want more."

### Checking in without sounding unsure

A weak check-in asks for reassurance. A strong one offers a fork, so the interviewer routes you and you keep driving.

| Weak, reads as unsure | Strong, reads as driving |
|---|---|
| "Does that make sense?" | "That is the render path. Do you want the state split next, or the offline behaviour?" |
| "Is this the component you meant?" | "I have assumed this renders in at most three places. Correct me now if it is a design-system primitive, because that changes the whole API." |
| "Should I go deeper?" | "I can go one level into list virtualization, or stay wide and cover accessibility and rollout. Which is more useful?" |
| "Sorry, I am rambling." | "I am two minutes over on the keyboard model. Moving to the data path." |

Two check-ins in a 45 minute round is about right: one after v0, one before the deep dive. Past four, you have handed over the driving seat, which is the thing being graded.

### How the split shifts by level

| Phase | Mid | Senior | Staff |
|---|---|---|---|
| Clarify and budgets | A functional list | A functional list plus three budgets carrying numbers | Negotiates which budget is the real constraint, and who owns it |
| Sizing | Usually skipped | Bytes on the critical path, rows on screen at once | One number, then its team or cost consequence |
| Rendering strategy | Names one, usually the one they used last | Chooses on a first-render budget and a personalisation level | Chooses, then names the migration path from what exists today |
| v0 and v1 | Most of the time lands here | Balanced | Compressed, because correctness is assumed |
| Accessibility | Mentioned near the end | An authoring pattern plus focus behaviour, per control | A conformance target, a testing method, and who is accountable for it |
| Deep dives | One, chosen by the interviewer | Two, one chosen by the candidate and defended | Two, plus rollout, kill switch and the regression they expect |
| Close | A recap | Trade-offs plus one thing they would measure | What was skipped, the first measurement, and the budget enforced in CI |

Read the columns rather than the cells. Mid level spends its minutes proving a working interface could be built. Senior spends them proving the rendering, state and accessibility decisions were chosen deliberately against numbers. Staff spends them proving the design is reachable from the codebase that exists, operable when a third-party script goes slow, and cheap enough that nobody reverts it next quarter.

!!! tip "One note on frameworks, made once"

    Paid interview courses often ship an acronym spine and apply it identically to every case study. Those acronyms belong to their publishers, not to this page. The problem is not the letters, it is that one spine applied unchanged to every prompt produces an answer that sounds recited, and a recited answer gives a grader nothing to grade.

    The problem is not the letters. It is that one spine applied to a dozen prompts produces an answer that sounds recited, and it fails hardest here, because an autocomplete and a checkout flow genuinely do not want the same phases. Everything above is a time budget. It never appears as a heading in any case study on this page, and Module 1 teaches when to abandon it.

!!! danger "WHEN TO BREAK THIS"

    The measurement that justifies following a clock: you have run one timed self-mock and finished with a design on the board, an accessibility contract stated, and time left over. Below that, the clock is scaffolding you still need. Above it, treat it as advisory. Three situations specific to this round where following it is the wrong move:

    **1. The interviewer starts drawing servers.** If minute six is "and how would you shard that", you are in a backend round that was scheduled under a frontend title. Do not fight it and do not pretend the client is the whole system. Say one sentence: "Happy to design the service side too. I will keep the client contract as the anchor and work backwards." Then run the [Modern System Design](modern-system-design.md) clock instead, and come back to bytes and focus order at the end if minutes remain.

    **2. The format flips mid-round.** You opened application-scale and at minute twenty they say "actually, just design the autocomplete inside it". Switch clocks out loud rather than quietly: "That is a component question, so I am going to spend the rest on its public interface, its state machine and its keyboard model." Then abandon the remaining application phases entirely.

    Continuing to draw the page architecture after that pivot is the fastest way to look like you are executing a script rather than listening.

    **3. The screen already exists.** "This page got slow after a dependency upgrade, diagnose it" has no v0 to draw, because v0 shipped months ago. Replace the requirements and v0 phases with four questions: what regressed, in which metric, at which percentile, and what changed in the same window. Then spend the clock on attribution, the smallest fix, how you would verify it, and what you would add to continuous integration so it cannot happen again.

    A quieter fourth: read who is in the room. A design-systems engineer will follow you into API surface, theming and deprecation and lose interest in the CDN. A performance engineer will do the opposite. Both are asking a fair question. Rebalance the deep dives towards whichever one they lean into, and say out loud that you are doing it.

## Module 1. Scope and honesty: the two question formats

**Time: 20 to 30 minutes reading, 10 to 15 minutes practice.**

### 1a. Predict first

Four interviewers open four rounds. All four invites said "Frontend System Design". Read the opening lines before anything else on this page.

| Prompt | The interviewer's first sentence |
|---|---|
| P1 | "Design the news feed for our social product. Assume the API already exists." |
| P2 | "Design an autocomplete we could put in the design system and hand to twelve teams." |
| P3 | "Design a photo sharing product, front to back." |
| P4 | "Checkout got 400 ms slower after last month's dependency upgrade. Walk me through what you do." |

??? note "Show answer"

    Prediction asked for: for each prompt, what artefact is due at minute 40, and which of the two formats is it?

    | Prompt | Format | The artefact due at the end |
    |---|---|---|
    | P1 | Application scale | A client architecture: rendering mode, state split, caching, list strategy, plus the budgets you designed against |
    | P2 | Component scale | A public API surface, a state machine, an accessibility contract, and an edge-case list. Almost no boxes |
    | P3 | Neither, as stated | "Front to back" puts a server rubric in the room. Ask which half is scored before you draw anything |
    | P4 | Application scale, brownfield | A measurement plan and one diagnosed cause. An architecture drawn before a measurement is a wrong answer here |

    The tell in P1 is the second sentence. Handing you the API fixes the boundary and tells you the entire round happens inside the client. The tell in P2 is the word "twelve teams", which converts an implementation into a contract you cannot change later without breaking consumers.

    P4 is the one candidates misread most. They hear "slower" and start proposing code splitting. Nothing in the prompt says the regression is bytes. Proposing a fix before naming the measurement is the failure mode this whole round exists to detect.

### The two formats, side by side

One title covers two rounds that share almost no rubric. Preparing only for one is the most expensive mistake available on this track, because it costs weeks and is invisible until the round starts.

| Dimension | Application scale | Component scale |
|---|---|---|
| What the prompt names | A product surface: a feed, a dashboard, a checkout, a mail client | A widget or a primitive: autocomplete, data grid, dialog, file uploader, date picker |
| What is normally given | The API, or permission to invent it | Nothing. You define the interface, which is the point |
| What you draw first | A boundary diagram: what runs on the server, what ships, what caches where | A prop table and a state chart. Boxes and arrows are close to useless |
| Where the minutes go | Rendering strategy, state split, caching, list performance, failure states | API surface, state ownership, keyboard and focus model, edge cases |
| The number that dominates | Bytes on the critical path, interaction latency, first render | Count of consumers, count of reachable configurations, count of interactive states |
| What earns a strong rating | Deriving the rendering and caching choice from a stated budget | Refusing to add a prop, and naming the state the widget must not be able to reach |
| The most common failure | Naming a framework instead of a constraint | Designing the happy path and never opening the keyboard model |

### Two formats you should also expect, and neither is on the invite

- **The disguised backend round.** The interviewer keeps asking about the database behind the feed. The rubric in their head is server capacity. Steer once, explicitly, then follow: "I can design the client, or I can design the service behind it. Which is scored today?" If they want the service, [Modern System Design](modern-system-design.md) is the material, and saying so out loud costs you nothing.
- **The diagnostic round.** You are handed a running system and a regression, as in P4. There is no greenfield design phase. The deliverable is an ordered measurement plan, one cause, and the smallest change that moves the number.

!!! warning "The sentence that decides the format, and how to force it early"

    You get one cheap move at minute one: state which format you think you are in and offer the other. "I read this as a component design problem, so I am going to spend most of the time on the public API and the interaction model rather than on architecture boxes. Tell me now if you would rather see the application around it."

    Interviewers answer this almost every time. Twenty seconds spent here protects the remaining thirty-nine minutes. Candidates who skip it routinely deliver the wrong artefact and never find out why the feedback said "did not go deep".

### How to tell inside sixty seconds

Four tells, in order of reliability.

1. **Reuse language.** "Design system", "component library", "used by N teams", "reusable", "publish it as a package". Any one of these makes it component scale, because the deliverable is a contract other people depend on.
2. **The article and the noun.** "The feed", "our checkout" is a specific product surface, so application scale. "A date picker", "an uploader" is a category, so component scale.
3. **Whether the data source is described.** If the interviewer describes endpoints, payload shapes or an existing gateway, they have drawn the boundary for you and everything interesting is inside the client.
4. **Whether a constraint arrives unprompted.** "Our users are on low-end Android devices" or "this has to work at 200 rows per second" signals application scale with a budget already chosen for you.

### The routing diagram

```
          first two sentences of the prompt
                        |
        +---------------+---------------+
        |                               |
  names a product              names a widget or
  surface or a page            a reusable primitive
        |                               |
        v                               v
+------------------+          +--------------------+
| application      |          | component scale    |
| scale            |          | API + state chart  |
+------------------+          +--------------------+
        |
   does a system
   already exist
        |
   +----+-----+
   |          |
   v          v
+--------+ +----------------+
| green  | | diagnostic:    |
| field  | | measure first  |
+--------+ +----------------+

Caption: routing the opening of a frontend round to one of three
deliverables. Reuse language sends you right; an existing system
sends you to the diagnostic branch, where measurement precedes design.
```

### On which companies favour which format

We are not going to publish a company-to-format table, and the reason is the point of this section.

| Why not | Consequence for you |
|---|---|
| The format is chosen by the team and often by the individual interviewer, not by the company | A table that is right for one org inside a company is wrong for the next org over |
| Nothing about internal round design is published by the companies that run these loops | Any table we wrote would be crowdsourced hearsay presented as fact, which is exactly the failure mode this site exists to avoid |
| Round formats change between hiring cycles | A table would rot silently, and you would find out during the round |

What you can do instead, all of it checkable by you:

- **Read the job description for structural words.** "Design system", "component platform", "web platform", "developer experience", "UI infrastructure" predict component scale. "Product engineer", "growth", "feature team", "consumer web" predict application scale. This is inference about the work, not a claim about a company.
- **Ask the recruiter two questions.** "Is the frontend design round scoped to the client, or does it include the service behind it?" and "Is it usually a product surface or a single component?" Both are ordinary scheduling questions and recruiters answer them.
- **Read the interviewer's public profile if you are given a name.** Someone who works on a component library is unlikely to hand you a feed prompt.

### 1d. Worked example

The invite says: "Frontend Engineer, four rounds. Frontend System Design (60 min), Coding: DOM and async (45 min), Frontend Deep Dive (45 min), Behavioural (45 min). Your design interviewer works on the Web Platform team."

**Block 1. PURPOSE: convert a round list into a list of artefacts, because artefacts are what I can rehearse.**

Three technical rounds, three different outputs. The design round wants a design artefact. The coding round wants working code against a DOM API. The deep dive wants spoken mechanism at the level of "what does the browser actually do here". I write three words at the top of my plan: artefact, code, mechanism.

The strongest signal in the email is "Web Platform team". That is infrastructure, not product. It shifts my expectation toward component scale, or toward an application prompt where the interesting part is the boundary rather than the feature.

**The error most readers make here** is reading "Frontend Engineer" and rehearsing a news feed, because a feed is the prompt that free material practises most. The team name overrides the job title every time.

!!! note "Say it before you read on"

    Name one thing a Web Platform interviewer is likely to push on that a product-team interviewer usually will not. Say it out loud before continuing.

**Block 2. PURPOSE: decide what I will deliberately not prepare, because preparation is subtraction.**

Given a platform team, I drop three areas entirely: marketing-page search visibility, analytics dashboards, and anything that turns into a product-sense conversation. I keep and deepen four: public API design for a component, the accessibility contract, bundle and boundary economics, and version skew across consumers.

That reallocation is roughly six hours of study moved, not added. A platform interviewer asking "what happens when a consumer upgrades and their markup breaks" is a question no amount of feed practice answers.

**The error most readers make here** is additive preparation: they bolt component material on top of a full application syllabus and arrive having half-rehearsed both.

**Block 3. PURPOSE: buy information cheaply, because two sentences from a recruiter beat two hours of inference.**

I send one message: "For the 60 minute design round, is it usually a product surface or a single reusable component? And is the service behind it in scope, or is the round scoped to the client?"

If the answer is "single component", my whole rehearsal changes: I practise drafting a prop table under time pressure, not drawing architecture. If the answer is "product surface, client only", I practise the rendering and caching derivation instead.

!!! note "Say it before you read on"

    Which single sentence in a recruiter's reply would most change how you spend the last four hours of preparation? Answer before reading block 4.

**Block 4. PURPOSE: rehearse an opening that works for either format, because the email did not settle it.**

I write and say out loud: "Before I start, one question that changes how I spend the hour: is this a product surface, where I should design the client architecture and its budgets, or a single reusable component, where I should design the public API and the interaction model? I have a plan for both. If you have no preference I will assume the first and say what I am dropping."

**The error most readers make here** is improvising the opening, then discovering at minute twenty that they drew architecture for a component prompt.

**Result.** The reply was: "Component prompt, and she cares a lot about accessibility." That moved the final hours from rendering strategy to focus management and the authoring patterns in Module 7, which is exactly where the round went.

### 1e. Fade the scaffold

**Faded example 1.** The invite says: "Senior Frontend Engineer, Payments. Rounds: Frontend System Design (60), Coding (60), Behavioural (45). The design interviewer is the tech lead for the checkout surface." Blocks 1 to 3 are given. Produce block 4.

- Block 1: artefacts are a design artefact, code, and a behavioural narrative. "Checkout surface" is a product surface, so application scale is the default assumption.
- Block 2: I drop design-system governance, micro-frontends and heavy visualisation. I keep form correctness, error and retry semantics, idempotent submission from the client, and third-party script risk on a payment page.
- Block 3: I ask whether the payment provider integration is in scope, because an embedded provider frame changes the security and state model completely.

??? note "Show answer"

    A defensible block 4. What earns credit is that the opening names correctness before performance, which is the inversion a payments interviewer is listening for.

    "Restating it: the checkout surface, client side, with a payment step whose failure is expensive. I think the hard parts are submission exactly once from a client that can be reloaded mid-flow, error states that are recoverable rather than dead ends, and keeping third-party scripts off the critical path. I am going to spend more time than usual on state and failure and less on rendering strategy, because on a checkout the rendering choice is not what loses money. Stop me if you want the opposite emphasis."

    The specific move that separates this from a generic opening is refusing the usual performance-first script and saying why. On most surfaces first render dominates. On a payment step, a double submission dominates.

**Faded example 2.** The invite says: "Frontend Engineer, Data Platform UI. Rounds: Frontend System Design (45), Coding (45), Behavioural (45). No further detail." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: three artefacts. Only 45 minutes for design, so about 40 working minutes, which means one deep dive rather than two.
- Block 2: "Data Platform UI" points at grids, charts and large result sets, so I keep virtualization, interaction latency under load, and streaming or paginated result handling. I drop marketing-page concerns entirely.

??? note "Show answer"

    Block 3: the cheap purchase is about scale, not topic. I ask: "Roughly what result sizes does this UI handle, hundreds of rows or millions?" The answer changes the entire round. Hundreds of rows means the design problem is interaction and correctness. Millions means the design problem is windowing, incremental data transfer and never holding the full set in memory, and the deep dive is decided before I walk in.

    Block 4: an opening that puts the scale question on the record in the first thirty seconds. "I want to pin one number before I design: the largest result set a user can put on screen. If it is thousands I will keep the design simple and spend the time on interaction correctness. If it is millions, the design is a windowed view over a cursor and almost every other choice follows from that. Which one is it?"

    Asking for one number and stating how each answer forks the design is the whole move. It is the same discipline as estimation in a backend round, done with the one quantity that actually decides a data-heavy UI.

### 1f. Check yourself

**Q1.** An interviewer says: "Design a file uploader." You ask no questions and start drawing a client, a chunking layer and a server. What did you just get wrong, and what was the one question worth asking?

??? note "Show answer"

    You assumed application scale on a prompt that is more often component scale. "A file uploader" is an indefinite article plus a widget category, which is tell number two. If it is a component prompt, the server side you just drew is out of scope, and every minute spent on it is a minute not spent on the API, the progress and cancellation model, and the accessibility of a drop zone.

    The one question: "Is this a reusable component other teams will consume, or the upload feature inside one product?" If reusable, the first artefact is a prop table plus the state chart for a single file, and resumability becomes a contract question ("who owns the retry, us or the consumer?") rather than a server design.

**Q2.** Twenty minutes into an application-scale round, the interviewer says "assume everything renders instantly and the network is free". What just happened, and what do you do?

??? note "Show answer"

    They removed performance from the rubric. What is left is state correctness, data modelling on the client, failure states and interaction design. This is a gift, because it is one of the clearest scope signals available.

    Do two things. Say the reframe out loud so it is on the record: "Then the interesting problems are what the client considers the source of truth, what happens when two tabs disagree, and what the user sees in each failure state." Then delete your prepared bundle and rendering material. Continuing to volunteer byte budgets after that sentence reads as not listening, which is a delivery failure and is scored as one.

### 1g. When not to use this

Format triage is a preparation tool, and it is over-applied more often than under-applied.

| Question | Answer |
|---|---|
| The measurement that justifies doing triage at all | You have two or more loops booked, or you cannot name the deliverable of your design round from its description |
| The threshold below which it is over-engineering | One loop, with an invite that already names the format and the interviewer's team. Triage adds nothing on top of that |
| The cheaper alternative | One recruiter message with the two questions in this module. Reply time is usually a day and the information is exact, where inference is a guess |
| The failure mode of over-applying it | Spending study hours on meta-strategy instead of Modules 3 to 9. The whole triage exercise is worth about thirty minutes, once |
| What to do if triage is genuinely impossible | Prepare the component format. It is the smaller body of material, it is the one most easily left unprepared, and application-scale prompts reward component thinking more than the reverse |

---

## Module 2. The delivery clock for a frontend round

**Time: 25 to 35 minutes reading, 15 to 20 minutes practice.**

### 2a. Predict first

Below is the generalist clock from [the hub](index.md#the-clock), for 40 working minutes.

| Phase | Minutes |
|---|---|
| Scope and clarify | 5 |
| Requirements written down | 4 |
| Estimation gate | 2 |
| V0 design | 8 |
| Breaking point and V1 | 8 |
| Deep dive | 9 |
| Failure modes and wrap | 4 |

??? note "Show answer"

    Prediction asked for: which phases are close to worthless in a component-scale round, and what has to replace them?

    Two phases collapse. **Estimation** goes to zero: there is no traffic volume inside one widget, and inventing a request rate for a date picker is visible ritual. **V0 as a box diagram** also goes to near zero: the smallest thing that works for a component is an API signature, which takes ninety seconds to write, not eight minutes to draw.

    What replaces them, and why each is worth more minutes than the phase it displaces:

    | Replaces | New phase | Why it is worth the minutes |
    |---|---|---|
    | Estimation | Consumers and use cases | Consumer count is the quantity that decides how much configurability the API needs, exactly as request rate decides a backend design |
    | Part of V0 | Public API surface | The API is the irreversible artefact. Internals can be rewritten; a shipped prop cannot be removed without breaking consumers |
    | Part of V0 | State machine and interaction model | Interactive widgets fail on unreachable and illegal states, not on architecture |
    | Part of deep dive | Accessibility contract | It is a functional requirement with a testable pass bar, so it belongs in the main clock and not in an optional dive |

    If you predicted "estimation goes away", that is the easy half. The harder half is noticing that the V0 phase does not shrink because component design is easier. It shrinks because the artefact is textual, and the time moves to the two things text hides: states and keys.

### The two clocks

Both clocks assume a 45 minute slot loses about 5 minutes to introductions, tooling and your questions at the end, leaving 40 working minutes, and a 60 minute slot loses about 8, leaving 52. Those two deductions are assumptions, and they are reasonable because nearly every remote round spends time on hellos, screen sharing and closing questions. Adjust once you see how your interviewer runs the first two minutes.

**Application-scale clock.**

| Phase | What ends the phase | 40 working min | 52 working min |
|---|---|---|---|
| Scope and format check | You have said which format you are in and named what is out | 3 | 4 |
| Requirements and budgets | A visible functional list plus two or three numbers | 6 | 7 |
| Data and API boundary | Agreement on what the server gives you and in what shape | 4 | 5 |
| V0: rendering and state | The simplest rendering mode and state split that satisfies the budgets, drawn | 8 | 10 |
| Breaking point and V1 | You name what fails, at what number, and change exactly one thing | 8 | 11 |
| Deep dive | One area at implementation depth, two if you have 52 | 7 | 10 |
| Failure, measurement and wrap | Named failure states, what you would instrument, what you skipped | 4 | 5 |
| **Total** | | **40** | **52** |

The sums: 3 plus 6 plus 4 plus 8 plus 8 plus 7 plus 4 is 40, and 4 plus 7 plus 5 plus 10 plus 11 plus 10 plus 5 is 52.

**Component-scale clock.** Same working minutes, spent on entirely different things.

| Phase | What ends the phase | 40 working min | 52 working min |
|---|---|---|---|
| Consumers and use cases | Three named call sites, at least one of them awkward | 4 | 5 |
| Public API surface | A prop table on the board, with owned state marked | 7 | 9 |
| State machine and interaction | Every state, every transition, and the illegal ones named | 7 | 9 |
| Accessibility contract | Roles, keys, focus rules and announcements, as a testable list | 6 | 8 |
| Edge cases and failure | Async, empty, error, long content, disabled, controlled and uncontrolled | 7 | 9 |
| Performance and bundle | Where the cost is, and what the component refuses to do | 4 | 6 |
| Versioning and wrap | What breaks a consumer, and what you skipped | 5 | 6 |
| **Total** | | **40** | **52** |

The sums: 4 plus 7 plus 7 plus 6 plus 7 plus 4 plus 5 is 40, and 5 plus 9 plus 9 plus 8 plus 9 plus 6 plus 6 is 52.

### The shape, drawn

```
Application scale, 40 working minutes
+-----+--------+-----+--------+--------+-------+-----+
|Scope|Reqs+bud|API  | V0     | Break  | Dive  |Wrap |
| 3   | 6      | 4   | 8      | 8      | 7     | 4   |
+-----+--------+-----+--------+--------+-------+-----+
0     3        9     13       21       29      36    40

Component scale, 40 working minutes
+------+--------+--------+------+--------+----+-----+
|Users | API    | States | A11y | Edges  |Perf|Ver  |
| 4    | 7      | 7      | 6    | 7      | 4  | 5   |
+------+--------+--------+------+--------+----+-----+
0      4        11       18     24       31   35    40

Caption: the two clocks over the same 40 working minutes. The
application clock puts its two widest blocks after the first
drawing. The component clock has no single dominant block, and
spends 20 of 40 minutes on states, accessibility and edges.
```

### The first two minutes, per format

Four sentences, in order, before you draw or write anything.

**Application scale.**

1. Restate the surface with one number attached, so a mismatch surfaces now rather than at minute thirty.
2. Name the two or three things you believe are actually hard, in client terms: bytes, interaction latency, state ownership, offline, accessibility.
3. State what you are dropping, and why it cannot change a decision in the next twenty minutes.
4. Ask the one clarifying question whose answer changes the design most, usually about device class, data volume or personalisation.

**Component scale.**

1. Name the consumers: "I am designing this for other teams, so the API is the thing I cannot take back."
2. Name the three call sites you will design against, including one awkward one.
3. State the two things you will treat as functional requirements rather than extras: the keyboard model and the controlled or uncontrolled contract.
4. Ask whether the component is styled or headless, because that decision changes the entire API.

### Checking in without handing over the pen

A weak check-in asks for reassurance. A strong one offers a routing decision and keeps you driving.

| Weak | What the interviewer hears | Stronger |
|---|---|---|
| "Does that make sense?" | You cannot self-assess | "That is the render and cache story. Next I would do the list at 10,000 rows, unless you want the offline behaviour." |
| "Should I talk about accessibility?" | You think it is optional | "I am treating the keyboard model as a functional requirement, so it is coming in the next block rather than at the end." |
| "Is this API okay?" | You want approval for a contract | "Two of these props overlap. I would rather ship one enum than two booleans. Do you want the reasoning, or shall I just make the call?" |
| "Sorry, I am going too fast" | You are apologising for pace | "I am moving fast on purpose so there is time for failure states. Stop me anywhere." |

Two check-ins in a 40 minute round is about right: one after the first artefact, one before the deep dive. More than four and the interviewer is driving, which is the thing being tested.

### How the split shifts by level (component-scale round)

Same 52 working minutes, allocated differently.

| Phase | Mid | Senior | Staff |
|---|---|---|---|
| Scope and format check | 3 | 4 | 7 |
| Requirements and budgets | 6 | 7 | 8 |
| Boundary or API surface | 6 | 9 | 9 |
| V0 or state machine | 14 | 10 | 7 |
| Breaking point and V1 | 10 | 11 | 9 |
| Deep dive | 9 | 8 | 7 |
| Failure, measurement and wrap | 4 | 3 | 5 |

Read the columns, not the cells. A mid answer is scored on whether a correct client appears at all. A senior answer is scored on the breaking point and one area of real depth. A staff answer spends at both ends: negotiating what is worth building, and stating how the thing is measured, rolled out and deprecated.

### 2d. Worked example

Sixty minute round, so 52 working minutes. The interviewer says: "Design an autocomplete component. We would put it in our shared library."

**Block 1. PURPOSE: pick the clock out loud, because the wrong clock produces the wrong artefact and I cannot recover 20 minutes.**

I hear "shared library", which is tell number one from Module 1. I say: "I am treating this as a component design problem, so I will spend most of the time on the public API, the interaction states and the accessibility contract, and only a few minutes on performance. If you would rather see the search architecture behind it, say so now."

That sentence takes twelve seconds and buys the whole plan.

**The error most readers make here** is starting with debouncing. Debouncing is a four-line implementation detail. Leading with it signals that you think this is a coding question.

!!! note "Say it before you read on"

    The interviewer says "actually, also assume the search service is yours to design". How many of the 52 minutes should move, and out of which phase? Answer before reading block 2.

**Block 2. PURPOSE: spend the first five minutes buying constraints, because a component API is decided by its consumers, not by its features.**

I ask for three call sites and I insist on an awkward one: a plain search box, a multi-select tag input inside a form, and an address field that must submit on selection. Three call sites with different shapes is exactly what forces the API to be an inversion-of-control design rather than a pile of booleans.

I write the three on the board. Everything I design from here has to serve all three or explicitly refuse one.

**Block 3. PURPOSE: protect the accessibility block by scheduling it early, because it is the block that gets eaten when time runs short.**

I say: "I am going to do the keyboard and screen reader contract at minute eighteen, before edge cases, because it constrains the API. If I leave it to the end it becomes a list of promises."

This is a scheduling decision that reads as seniority, and it is also true: the announcement model decides whether the component owns a status region, which is a prop and therefore an API change.

!!! note "Say it before you read on"

    Name one API decision that changes if the component owns its own live region rather than requiring the consumer to provide one. Answer before reading block 4.

**Block 4. PURPOSE: reserve the last six minutes for versioning, because on a shared component that is the only part nobody else will cover.**

At minute 46 I stop adding and say: "Last thing. Three changes to this API would break consumers silently rather than loudly: changing the default of a boolean, tightening a prop type, and changing what the component announces. I would ship the first behind a codemod, the second as a major, and the third never without a flag."

**Result.** In a real round this ordering produces an artefact the interviewer can grade against every criterion. The candidate who spends minute 46 adding a "fuzzy matching" feature has a longer feature list and a worse rating, because features are the cheap part and a contract is the expensive part.

### 2e. Fade the scaffold

**Faded example 1.** Forty-five minute round, so 40 working minutes. Prompt: "Design the notifications panel for our web app." Blocks 1 to 3 are given. Produce block 4.

- Block 1: product surface plus a definite article, so application scale. I say so, and name that the API is assumed to exist unless told otherwise.
- Block 2: I buy two numbers, notifications per user per day and whether unread count must be exact, because those two decide polling versus a persistent connection and whether the count is derived or stored.
- Block 3: V0 is a paginated list fetched on open, with the unread count from a cheap endpoint. No real-time transport yet.

??? note "Show answer"

    Block 4 is the breaking point plus V1, and the credit is in naming the number, not the component.

    "V0 breaks when the product wants the badge correct without a page reload. At one notification per user per hour, polling every 60 seconds is 24 requests per user per day for one useful update, which is wasteful but survivable."

    "At ten per hour with a badge expected to update within five seconds, polling costs 720 requests per user per day and still misses the latency target, so that is where I switch to a server-push transport. I would change exactly one thing: the count and the newest item arrive over a stream, and the list itself stays a plain paginated fetch on open."

    The move that separates this from a template answer is refusing to convert the whole panel to real time. Only the badge has a latency requirement, so only the badge changes transport.

**Faded example 2.** Sixty minute round. Prompt: "Design a data grid for our internal tools team." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: "for our internal tools team" is reuse language, so component scale. I say so and name the awkward call site early: a grid embedded inside another scrolling container.
- Block 2: three consumers, an admin table of 200 rows, a log viewer of 2 million rows, and an editable pricing sheet of 5,000 cells.

??? note "Show answer"

    Block 3: schedule the state machine and accessibility before edge cases, and say why. "Selection semantics and the keyboard model are the two things that differ between those three consumers, so I am doing them at minute eleven. The log viewer wants no selection at all, the admin table wants row selection with shift-range, and the pricing sheet wants cell focus with arrow keys. If I design the API before I resolve that, I will invent a selection prop that fits one of the three."

    Block 4: reserve the last six minutes for what the component refuses to do. "This grid will not own sorting, filtering or data fetching. It takes rows and a row count and asks for a window. That refusal is the reason it can serve 200 rows and 2 million rows with one API, and it is the decision I would defend hardest, because every request to add server-side sorting inside the grid is a request to make it useless for one of the three consumers."

    Naming what the component will not do is the component-scale equivalent of removing a box in a backend round, and it is the strongest available signal at senior level.

### 2f. Check yourself

**Q1.** You are at minute 26 of 40 in an application-scale round and you have only just finished V0. Which phase do you sacrifice and which do you protect?

??? note "Show answer"

    Sacrifice the deep dive down to a single stated choice, and protect the failure and measurement block. Concretely: instead of seven minutes on virtualization internals, spend ninety seconds saying which dive you would do and what you expect to find, then use the remaining time on failure states and instrumentation.

    The reason is where the signal concentrates. A partial deep dive still reads as depth if you name the trade-off. A round that ends with no failure states and no measurement plan reads as a candidate who has never operated a UI in production, and that judgement is much harder to reverse.

    Say the trade explicitly: "I am behind. I am going to skip the virtualization internals and instead spend the last eight minutes on what breaks and how I would know, because I think that is worth more to you."

**Q2.** A component-scale interviewer interrupts at minute 8 with "what happens if the consumer renders two of these on the same page?" Do you follow the clock or the question?

??? note "Show answer"

    Follow the question, immediately and completely. An interruption is the highest-value signal in the round because it tells you exactly what is being graded, and this particular question is about global state leakage: duplicate element ids, a single shared portal container, a document-level keydown listener, and a live region that both instances write into.

    Answer it in full, then re-anchor the clock in one sentence: "That pushes me to make every id instance-scoped and to make the live region a prop rather than something we own. I will fold that into the API table I was building, and carry on there." Saying "I will get to that later" is the single most damaging sentence available in any design round.

### 2g. When not to use this

| Question | Answer |
|---|---|
| The measurement that justifies following a clock | You have run one timed self-mock and did not finish with an artefact and a failure list. Until then, the clock is scaffolding you still need |
| The threshold below which it is over-engineering | A round shorter than 30 minutes, or a screening call. There is no room for seven phases in 25 minutes; use three: constraints, one artefact, one failure story |
| The cheaper alternative | A two-line rule: first artefact on the board by the one-third mark, first failure named by the two-thirds mark. That captures most of the value of the whole table |
| Situation one where following it is wrong | The interviewer opens with the deep dive ("assume the app exists, make the list scroll at 120 Hz"). There is no V0 phase. Spend the whole clock on the one question |
| Situation two where following it is wrong | The prompt is a diagnostic. Requirements and V0 already exist, and the minutes belong to measurement, hypothesis and the smallest change |
| Situation three where following it is wrong | The interviewer is silent and taking notes. That usually means they want coverage, not a conversation. Widen slightly and check in once, rather than going deep on your own choice |

---

## Module 3. Requirements for a user interface

**Time: 30 to 40 minutes reading, 20 to 25 minutes practice.**

### 3a. Predict first

A candidate writes this requirements list in the first five minutes of a round. Read it before continuing.

| # | What the candidate wrote |
|---|---|
| R1 | "The feed shows posts and lets the user like and comment" |
| R2 | "It should be fast" |
| R3 | "We will use server rendering with streaming" |
| R4 | "It should be accessible" |
| R5 | "Support mobile" |

??? note "Show answer"

    Prediction asked for: which of these five are requirements, and what is wrong with each of the others?

    | # | Verdict | Why |
    |---|---|---|
    | R1 | A requirement, but incomplete | It names the ideal state and no other. A UI feature has five states, and four of them are missing |
    | R2 | Not a requirement | "Fast" is not testable. A requirement is a number plus the condition it holds under |
    | R3 | Not a requirement at all | It is a solution. Writing a solution into the requirements list removes the decision the round exists to test |
    | R4 | Not a requirement | Accessible is not a level. Without a conformance level and a test method, nobody can say whether it was met |
    | R5 | Not a requirement | "Mobile" is neither a device class nor a network. It is a word that lets everyone in the room believe something different |

    R3 is the expensive one. A candidate who writes the rendering strategy into the requirements has skipped the derivation and can now only defend a choice they never made. That is pattern matching rather than derivation, and it is very hard to recover from once the word is on the board.

### Functional requirements for a user interface

The unit of a functional requirement here is not a feature. It is a capability plus every state that capability can be in. Every asynchronous UI has five, and candidates routinely write one.

| State | What it means | The question it forces |
|---|---|---|
| Ideal | Data arrived, is non-empty, and the user can act | What is the primary action, and how fast must it acknowledge? |
| Empty | The request succeeded and returned nothing | Is this a new user, a filtered-out result, or a deleted entity? All three look identical and need different copy |
| Loading | The request is in flight | Skeleton, spinner or nothing? What if it resolves in 50 ms and the spinner flashes? |
| Partial | Some data arrived, some failed or is still coming | Which parts of the screen are independently useful, and which must not render half-populated? |
| Error | The request failed | Is it retryable, and does the user lose work if they retry? |

Write the list as capability plus states, and you get requirements that decide architecture:

- "The feed renders the first ten posts, with each post independently able to fail without blanking the page." That sentence rules out one giant query and points at per-item boundaries.
- "Liking acknowledges within 100 ms whether or not the server has responded." That sentence forces optimistic updates and therefore a rollback path.
- "A user who reloads mid-comment does not lose the draft." That sentence forces local persistence and rules out holding drafts only in component state.

### The five budget families

A budget is a number plus the condition under which it must hold. Without the condition it is a slogan.

#### Budget 1: interaction latency

Two different clocks, and candidates conflate them constantly.

| Clock | The number | Where it comes from |
|---|---|---|
| Frame budget at 60 Hz | 16.7 ms per frame | 1000 ms divided by 60 frames |
| Frame budget at 120 Hz | 8.3 ms per frame | 1000 ms divided by 120 frames |
| Your share of a 60 Hz frame | roughly 10 to 12 ms | Assume the browser needs 4 to 6 ms of the frame for style, layout, paint and compositing on a mid-tier device. Reasonable because that work happens on every frame regardless of your code, and it scales with DOM size |
| Acknowledgement of a discrete input | under about 100 ms | The classic human-computer interaction response bands (roughly 0.1 s for "immediate", 1 s for "uninterrupted flow", 10 s for "attention lost") come from the psychology literature of the 1960s to 1990s and have been the working convention ever since. Treat as a convention, not a measurement |

The practical requirement that follows: **acknowledge in under 100 ms, complete when you can, and show progress if completion exceeds about one second.** Acknowledging is not the same as completing, and separating them is what lets you design optimistic updates without lying about them.

#### Budget 2: bundle bytes

Bytes are only meaningful against a network. Pick the network first, then the budget follows by division.

| Input | Value | Why this value |
|---|---|---|
| Assumed effective downlink | 1.6 Mbit/s, which is 200 KB/s | An assumption standing in for a congested mobile connection. Reasonable because it is far below a good connection and far above an unusable one, so it produces a budget that is tight but not absurd |
| Target: JavaScript transferred on the critical path | 200 KB compressed | Chosen so transfer alone costs 1.0 s: 200 KB divided by 200 KB/s |
| Uncompressed equivalent | roughly 600 KB | Assume about 3:1 compression on JavaScript text. Reasonable because JavaScript is highly repetitive text and text compressors do well on it |

Two things that budget does not include, and both are usually larger than the transfer on a low-end device:

- **Parse and compile**, which scales with the uncompressed size, not the compressed size. That is why the 600 KB number matters more than the 200 KB one.
- **Execution**, meaning module evaluation, framework boot and hydration, which is Module 5.

!!! warning "Do not quote a milliseconds-per-kilobyte figure"

    You will see numbers like "N ms of parse per KB on a mid-range phone". Those come from specific devices, specific engines and specific years, and they age badly. In a round, say the shape and name the measurement: "parse and compile scale with uncompressed bytes, so I would measure boot time on the lowest device tier we support and set the budget from that measurement rather than from a published figure."

    That answer is stronger than a quoted number, and it cannot be wrong.

#### Budget 3: first render

First render is a sum, so it can be budgeted as one. Every row is an assumption you state, and the total is the requirement.

| Component of first render | Assumed cost | Where it comes from |
|---|---|---|
| Connection setup, warm | 0 ms | Assume the connection is reused for a repeat visit |
| Connection setup, cold | about 90 ms | Assume a 30 ms round trip, one for DNS if uncached, one for TCP, one for TLS with session resumption |
| Time to first byte from a CDN hit | 30 ms | One round trip at the assumed 30 ms |
| Time to first byte from an origin render | 150 ms | 30 ms network plus an assumed 120 ms of server data fetching and rendering |
| HTML transfer, 40 KB document | 200 ms | 40 KB divided by 200 KB/s |
| Render-blocking CSS, 30 KB | 150 ms | 30 KB divided by 200 KB/s, and it blocks first paint by definition |

A budget written from those rows: "First contentful paint under 500 ms on the assumed network for a cold visit to a cacheable page." Then check it: 90 plus 30 plus 200 is 320, leaving 180 ms of headroom for CSS, which forces the CSS budget to about 36 KB. That is a requirement that decides something.

#### Budget 4: accessibility conformance

State a level and a test method, or it is not a requirement.

| Level | What it is | When to state it |
|---|---|---|
| Level A | The minimum conformance level of the Web Content Accessibility Guidelines | Almost never sufficient as a target. Stating A signals you have not been asked to meet a real bar |
| Level AA | The level normally written into procurement and policy | The default requirement to state in an interview, and the one to design against |
| Level AAA | The highest level, and not achievable for all content | Cite it only for specific criteria you deliberately adopt, never as a blanket target |

The requirement sentence: "Level AA conformance for the flows in scope, verified by a keyboard-only pass, an automated rule run in continuous integration, and one screen reader pass per interactive component." Module 7 makes each of those testable. The specification itself is published by the W3C at [w3.org/TR/WCAG22](https://www.w3.org/TR/WCAG22/).

#### Budget 5: supported devices and networks

"Mobile" is not a requirement. A device tier is, because each tier changes a different number.

| Tier | What it stands for | The budget it changes |
|---|---|---|
| Low | Several-year-old phone, limited memory, thermally throttled, slow storage | Execution and hydration time, and memory ceiling for lists and images |
| Mid | Recent mid-range phone or an older laptop | The frame budget under interaction, because this tier is where jank first appears |
| High | Current flagship phone or a desktop | Almost nothing. Designing for this tier is how teams ship software that only works for the people who built it |

The rule worth stating out loud: **the low tier decides the JavaScript budget, and the network decides the byte budget.** They are different constraints and they are often in tension, because the cheapest fix for slow execution is to ship less code, which also fixes bytes, while the cheapest fix for a slow network is caching, which does nothing for execution.

### Tagging, so the interviewer can audit you

Every requirement carries one of three tags. This is the same discipline the other tracks use, and it matters more here because frontend numbers are so easy to invent.

| Tag | Meaning | Example |
|---|---|---|
| GIVEN | The interviewer said it | "Users are mostly on low-end Android devices" |
| ASSUMED | You chose it, and you said why in one sentence | "200 KB/s effective downlink, standing in for a congested mobile network" |
| DERIVED | It follows arithmetically from a GIVEN or an ASSUMED | "36 KB CSS budget, from a 500 ms paint target minus 320 ms of connection, TTFB and HTML" |

### The budget waterfall, drawn

```
Cold visit, cacheable page, 200 KB per second, 30 ms round trip
+--------------------------------------------------+
| target: first contentful paint under 500 ms      |
+--------------------------------------------------+
   |
   +-> +----------------------+  90 ms  DNS+TCP+TLS
   |   | connection setup     |
   |   +----------------------+
   +-> +----------------------+  30 ms  CDN hit
   |   | time to first byte   |
   |   +----------------------+
   +-> +----------------------+ 200 ms  40 KB at 200 per s
   |   | HTML transfer        |
   |   +----------------------+
   +-> +----------------------+ 180 ms  what is left
       | CSS budget = 36 KB   |
       +----------------------+

Caption: a first-paint budget as a sum. Three costs are assumed
and stated; the fourth is derived by subtraction, which is what
turns a target into a byte limit somebody can enforce.
```

### 3d. Worked example

The prompt: "Design the product listing page for our e-commerce site. A large share of our growth is in markets with slow networks and inexpensive phones."

**Block 1. PURPOSE: turn the prompt into capabilities and their states, because a feature list hides four fifths of the work.**

I write three capabilities and their states, not five features:

- Browse a category: ideal is a grid of products; empty is "no products in this category" versus "your filters removed everything", and those need different copy and different recovery actions; loading is a skeleton grid at the correct dimensions so nothing shifts; partial is prices missing while images are present, which I decide is not allowed to render because a price that appears late is worse than a price that appears with the card; error is retryable in place.
- Filter and sort: the interesting state is partial, because the old results are still on screen while new ones load. I decide the old list stays visible and dims, rather than being replaced by a skeleton.
- Add to basket: acknowledgement under 100 ms regardless of server latency, which forces an optimistic update and a rollback path.

**The error most readers make here** is listing features and moving straight to architecture. The five-state pass takes ninety seconds and produces three architectural constraints, all of which I just derived rather than asserted.

!!! note "Say it before you read on"

    The partial state for filtering is a design decision, not a technical one. Say out loud what the alternative is, and one reason a shopping site might prefer it.

**Block 2. PURPOSE: pick the two budgets that will actually decide the architecture, because five budgets on a whiteboard is a checklist and two is a design.**

The prompt gave me one thing: slow networks and inexpensive phones. That is GIVEN. So the two budgets that decide everything here are bytes on the critical path and execution time on the low tier. Interaction latency matters, but nothing in this page is animation-heavy. Accessibility conformance is a requirement I will state and design to, but it does not fork the architecture between two candidate designs, so it is not one of the two.

I say this out loud, because choosing which budgets matter is the judgement being scored: "There are five things I could budget. Two of them decide this design, and I am going to name the other three and then not spend time on them."

**Block 3. PURPOSE: derive the byte budget rather than assert it, because an asserted number cannot survive one follow-up question.**

- ASSUMED: 200 KB/s effective downlink, standing in for a congested mobile network.
- ASSUMED: a first meaningful render target of 1.5 s on that network, which is the point at which a shopper on a category page is still in flow.
- DERIVED: 1.5 s at 200 KB/s is 300 KB total on the critical path, for everything: HTML, CSS, fonts and any blocking JavaScript.
- DERIVED: allocating 40 KB HTML plus 30 KB CSS plus 30 KB of a subsetted font leaves 200 KB, which is the entire critical-path JavaScript budget, before any product images.
- The elimination: 200 KB compressed is not enough for a client-rendered page that also has to fetch its data after boot, because the data round trip lands after the JavaScript finishes transferring. So client-only rendering is out, and I have ruled it out with arithmetic instead of taste.

**The error most readers make here** is producing a byte number and never using it. A number that eliminates nothing should be deleted. This one killed a rendering strategy in the same breath, which is what earns it its place.

!!! note "Say it before you read on"

    Product images are outside the 300 KB critical path in the arithmetic above. Say why that is legitimate, and what you must do to keep it legitimate.

**Block 4. PURPOSE: write the accessibility and device rows as testable statements, because "accessible" and "mobile" are the two words most likely to be waved at and never checked.**

- "Level AA conformance for browse, filter and add-to-basket, verified by a keyboard-only pass on each flow, automated rules in continuous integration on every pull request, and one screen reader pass per new interactive component."
- "Low tier is the budget-setting device. Boot to interactive on the low tier is measured on a real device in the lab, and the number that comes out of that measurement is the budget, replacing any figure assumed here."
- "The grid reflows to a single column without horizontal scrolling at 400 percent zoom, which is a stated conformance requirement and also happens to be the same code path as the narrow viewport."

**Result.** Four blocks, roughly six minutes spoken. What is on the board is a functional list with states, two budgets with derivations, three eliminated options and a test method. That is a requirements phase an interviewer can grade. A feature list with "should be fast" underneath is not.

### 3e. Fade the scaffold

**Faded example 1.** Prompt: "Design the inbox view of a web email client. Most usage is on desktop, on good networks." Blocks 1 to 3 are given. Produce block 4.

- Block 1: capabilities are list messages, read a message, and compose. The interesting states are partial (the list is present while a message body is still loading) and error (a send that failed after the compose window closed).
- Block 2: the two deciding budgets are interaction latency, because this is a keyboard-heavy application used all day, and memory, because a long-lived tab accumulates.
- Block 3: DERIVED, from a target of 100 ms acknowledgement on arrow-key navigation between messages, the row change must not depend on a network request, which means the list rows carry enough data to render a header immediately.

??? note "Show answer"

    Block 4 is the testable device, accessibility and longevity row set. The credit is in noticing that "desktop, good networks" does not remove requirements, it changes which ones bind.

    - "Level AA for list, read and compose, plus a stated commitment beyond AA on one criterion: every action reachable by keyboard has a documented shortcut, because this is a professional tool used all day and keyboard operation is a primary path rather than an accommodation."
    - "Supported device tier is mid and above, which I am stating explicitly so that nobody later assumes low-tier phones are in scope. The consequence I accept: the JavaScript budget is set by memory and long-session behaviour, not by transfer."
    - "Longevity requirement, because this tab stays open for eight hours: memory after four hours of normal use, measured in a soak test, must not exceed the memory after ten minutes by more than a stated factor. Without this requirement, nothing in the design prevents an unbounded message cache."

    The move worth copying is inventing the fourth budget family that this specific product needs. Five families is a starting set, not a fixed list, and a memory-over-time budget is exactly what a long-lived application requires and a shopping site does not.

**Faded example 2.** Prompt: "Design the analytics dashboard for our internal teams. Data volume is large and the users are on company laptops." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: capabilities are pick a time range, view several charts, and drill into one. Partial is the dominant state, because charts resolve at different speeds and the dashboard must be useful before all of them arrive.
- Block 2: the two deciding budgets are interaction latency during range changes, and the volume of data crossing the network per view.

??? note "Show answer"

    Block 3, the derivation. ASSUMED: eight charts on a dashboard, and an unaggregated series of 100,000 points per chart at 16 bytes per point in JSON, which is 1.6 MB per chart.

    DERIVED: 8 charts times 1.6 MB is 12.8 MB per view, and on a good office connection assumed at 2 MB/s that is 6.4 s of transfer before anything renders. The elimination: shipping raw series to the client is dead, and it died on arithmetic rather than opinion.

    DERIVED consequence: aggregation happens server side, the client requests points at the resolution it can display, and a chart 800 pixels wide asks for at most about 800 points, which is 12.8 KB per chart and 102 KB per view, a factor of 125 smaller.

    Block 4, the testable rows. "Level AA, and specifically every chart has a non-visual equivalent, because a chart image with no accessible representation fails outright and is the single most common conformance failure in dashboards. The equivalent is a data table with the same numbers, reachable from the chart."

    Plus: "Device tier is mid and above; the binding budget is main-thread time during a range change, measured as the longest task, because a dashboard that freezes for 400 ms on every filter change is unusable regardless of how fast it loaded."

### 3f. Check yourself

**Q1.** An interviewer says "assume a fast network and modern devices". Which of the five budget families still bind, and what should you say?

??? note "Show answer"

    Interaction latency and accessibility conformance still bind fully. Bundle bytes and first render relax but do not vanish, because execution cost survives a fast network and a modern device still has a frame budget of 16.7 ms. Device tier has been given to you, so it stops being a decision.

    What to say: "Then bytes stop deciding the design, so I will not spend minutes on transfer arithmetic. Two things still bind: work per interaction, because a fast network does not buy frames, and conformance, because that is a functional requirement and not a performance one. I will set the budget at roughly 10 ms of my own work per frame during interaction and design against that."

    The reason this is the right answer rather than "great, no budgets" is that removing the network reveals the constraint that most desktop applications actually fail on, which is main-thread work.

**Q2.** You write "first contentful paint under 500 ms" as a requirement. Under what conditions, and what makes this number checkable rather than decorative?

??? note "Show answer"

    A latency requirement without a condition set is not testable. It needs four things attached, and each of them is a decision:

    - **Which network and device.** 200 KB/s and the low tier, or a good connection and a laptop, produce budgets that differ by an order of magnitude.
    - **Which percentile.** A median is nearly meaningless for load performance, because the users who leave are in the tail. State p75 or p95, and state which.
    - **Cold or warm.** A cold cache visit and a repeat visit are different systems. Most 500 ms claims quietly assume warm.
    - **Lab or field.** A lab number is reproducible and comparable across pull requests. A field number is real. You need both, and they will disagree, and the disagreement is information rather than a bug.

    So the checkable version reads: "First contentful paint under 500 ms at p75, cold cache, on the low device tier at 200 KB/s, measured in the lab on every pull request, with field p75 reported weekly and allowed to be higher."

### 3g. When not to use this

Formal budgets are a component like any other, and they have a scale below which they cost more than they return.

| Question | Answer |
|---|---|
| The measurement that justifies a full budget set | You have more than one team shipping into the same page, or you have already had one regression that nobody noticed until a user complained |
| The threshold below which it is over-engineering | One team, one surface, fewer than roughly ten pull requests a week. A full five-family budget with continuous integration enforcement will be ignored, and an ignored budget is worse than none because it teaches people that the check is noise |
| The cheaper alternative | One number, enforced. Pick the single quantity that would have caught your last regression, usually compressed bytes on the critical path, put a hard limit in continuous integration, and add families only when a second regression class appears |
| What breaks if you skip budgets entirely | Nothing, for a while. Then bytes and main-thread work grow monotonically, because every individual addition is small and nobody owns the total. The failure is slow, invisible and expensive to reverse |
| The over-application failure mode | Budgets nobody can meet, so the team routinely overrides them. A budget that is overridden twice has become a comment. Raise it to a number you will actually enforce and then hold it |

---

## Module 4. Rendering strategy

**Time: 35 to 45 minutes reading, 25 to 30 minutes practice.**

### 4a. Predict first

Four surfaces from the same company. Read the descriptions before continuing.

| Surface | Description |
|---|---|
| S1 | Marketing landing page. Identical for every visitor. Changes when marketing edits it, a few times a week |
| S2 | Logged-in analytics dashboard. Every pixel depends on who you are. Heavy interaction after load |
| S3 | Product detail page. Identical for everyone except a price that varies by region and a "recently viewed" strip |
| S4 | Documentation site. Identical for everyone. Thousands of pages. Mostly reading, with a search box |

??? note "Show answer"

    Prediction asked for: which rendering mode does each take, and which single quantity decided it?

    | Surface | Mode | The quantity that decided it |
    |---|---|---|
    | S1 | Static, built ahead of time | The shared fraction is 100 percent, so per-request rendering buys nothing and costs origin CPU on every visit |
    | S2 | Client rendering, or server rendering only of the shell | The shared fraction is near zero, so no cached HTML is reusable and the interactive fraction is near 100 percent |
    | S3 | Static or incremental for the shared shell, with the two varying pieces filled in separately | The shared fraction is high but not 1, so the design question is how to keep the page cacheable despite two personalised holes |
    | S4 | Static, with a caveat about build time | Shared fraction is 100 percent, but page count times build time per page is the number that decides whether a full build is still feasible |

    The quantity in every row is the same one: **what fraction of the bytes are identical for all users**. That fraction, and how interactive the page is after it arrives, decide the mode. Framework names decide nothing, which is why naming one in the first minute is the classic tell.

    If you answered "server rendering" for S1 because server rendering is good for search visibility, notice what you gave up: an origin render per visit for a document that never differs. Static output is also fully indexable, so the search argument did not distinguish the options at all.

### Six modes, defined by where and when the HTML is produced

Define these precisely before comparing them, because most disagreements about rendering are actually disagreements about definitions.

| Mode | Where HTML is produced | When | What the browser receives first |
|---|---|---|---|
| Client rendering | In the browser | After JavaScript downloads and executes | A near-empty shell |
| Server rendering | On the server | On every request | A complete document |
| Static generation | On the server | At build time, once per route | A complete document, from a cache |
| Incremental regeneration | On the server | At build time, then again in the background on a staleness rule | A complete document, from a cache, possibly stale |
| Streaming server rendering | On the server | On every request, flushed in pieces as data resolves | The head and shell immediately, then the rest |
| Islands or partial hydration | On the server, at build or request time | Either | A complete document, where only marked subtrees get JavaScript |

Two of these are frequently misdescribed:

- **Incremental regeneration is not faster for the user.** It produces exactly the same cached document as static generation. What it changes is build time and staleness, which are operational properties, not user-facing latency.
- **Streaming does not reduce total time.** It reorders it. The first paint moves earlier; the last byte arrives at roughly the same moment.

### The comparison, with the arithmetic shown

All six numbers below come from one set of stated inputs, so you can recompute or disagree with any of them.

| Input | Value | Status |
|---|---|---|
| Round trip to the edge | 30 ms | ASSUMED, a short-haul connection to a nearby point of presence |
| Effective downlink | 200 KB/s | ASSUMED, carried over from Module 3 |
| Time to first byte, cache hit | 30 ms | DERIVED, one round trip |
| Time to first byte, origin render | 150 ms | ASSUMED, 30 ms network plus 120 ms of server data work |
| Full HTML document | 40 KB, so 200 ms of transfer | ASSUMED size, DERIVED transfer at 200 KB/s |
| Shell-only HTML | 5 KB, so 25 ms of transfer | ASSUMED size, DERIVED transfer |
| Application JavaScript | 200 KB compressed, so 1000 ms of transfer | ASSUMED size, DERIVED transfer |
| Islands JavaScript | 40 KB compressed, so 200 ms of transfer | ASSUMED, one fifth of the app because only marked subtrees ship |
| Hydration execution, full app | 300 ms | ASSUMED for a low-tier device. Replace with your own measurement, always |
| Hydration execution, islands | 60 ms | ASSUMED, scaled with shipped code |
| Client data fetch after boot | 150 ms | DERIVED, 30 ms network plus 120 ms server, the same work the origin render did |

One more assumption, stated because it changes every result: **resources transfer serially over one saturated pipe.** Real browsers overlap downloads, so real numbers are better than these. The ordering between strategies survives the simplification, which is what the comparison is for.

| Mode | Content painted | Interactive | The arithmetic |
|---|---|---|---|
| Static | 230 ms | 1530 ms | 30 plus 200 to paint; plus 1000 of JavaScript plus 300 of hydration |
| Incremental regeneration | 230 ms | 1530 ms | Identical to static by construction. Only freshness and build cost differ |
| Server rendering | 350 ms | 1650 ms | 150 plus 200 to paint; plus 1000 plus 300 |
| Streaming server rendering | 55 ms shell, 325 ms content | 1625 ms | Shell at 30 plus 25; content once data resolves at 150, plus 175 for the remaining 35 KB |
| Client rendering | 55 ms shell, 1355 ms content | 1355 ms | Shell at 55; JavaScript done at 1055; boot 150; data 150 |
| Islands | 230 ms | 490 ms | Paint as static; plus 200 of JavaScript plus 60 of hydration |

Four conclusions fall straight out of the table, and each one eliminates something:

1. **Client rendering paints content 5.9 times later than static** (1355 against 230). That rules it out for any surface where the content is the product and the network is slow.
2. **Client rendering reaches interactive 175 ms earlier than static** (1355 against 1530). That rules out the claim that server rendering is universally faster. It is faster to content and slower to interactive, because hydration is pure added work.
3. **Streaming buys 295 ms of first paint and 25 ms of content** (55 against 350, and 325 against 350). That rules out streaming as a fix for slow data; it is a fix for a blank screen while data is slow.
4. **Islands reach interactive 1040 ms earlier than static** (490 against 1530), on identical HTML. That rules out hydration cost as an unavoidable price and moves it into Module 5, where it is a design variable.

### The two numbers that actually decide

Everything above collapses into a two-by-two. Compute the two fractions from the page in front of you.

- **Shared fraction:** the proportion of the rendered output that is byte-identical for every user. Count regions on the page, not lines of code.
- **Interactive fraction:** the proportion of the rendered output that needs event handlers or client state.

| | Low interactive fraction | High interactive fraction |
|---|---|---|
| **High shared fraction** | Static or incremental. Ship almost no JavaScript | Static or streaming, plus islands. The document is cacheable, the widgets are not |
| **Low shared fraction** | Server rendering, streaming if data is slow | Client rendering, or server-rendered shell plus client content. Cached HTML has no value here |

### The strategies drawn against a common clock

```
Times in ms. One 200 KB per second pipe, 30 ms round trip, low tier.

+---------------------+  +----------------------------+
| static HTML         |  | JS done 1230, hydrate 300  |
| content paints 230  |  | interactive at 1530        |
+---------------------+  +----------------------------+

+---------------------+  +----------------------------+
| islands HTML        |  | JS done 430, hydrate 60    |
| content paints 230  |  | interactive at 490         |
+---------------------+  +----------------------------+

+---------------------+  +----------------------------+
| client shell        |  | JS 1055, boot 1205         |
| paints at 55        |  | content and ready at 1355  |
+---------------------+  +----------------------------+

+---------------------+  +----------------------------+
| server HTML         |  | JS done 1350, hydrate 300  |
| content paints 350  |  | interactive at 1650        |
+---------------------+  +----------------------------+

Caption: the same page under four strategies, from the inputs in
the table above. Reading down the left column shows the content
race; reading down the right column shows the interactivity race,
and the two have different winners.
```

### The decision table you can redraw in a minute

| If this is true | Choose | And accept |
|---|---|---|
| Output is identical for all users and changes rarely | Static | A build step, and a deploy to change content |
| Identical for all users but there are too many pages to build, or content changes hourly | Incremental regeneration | Serving stale documents for up to the staleness window you choose |
| Output depends on the request, and the data is fast | Server rendering | Origin CPU on every request, and a cache you now have to reason about |
| Output depends on the request, and some data is slow | Streaming server rendering | A more complex error story, because part of the response is already sent when the failure happens |
| Output is almost entirely per-user and interaction-heavy after load | Client rendering | A slow first content paint on slow networks, and search visibility you must solve separately |
| The document is mostly static but has a few genuinely interactive regions | Islands | A build system that can split hydration, and a discipline about what counts as an island |

### 4d. Worked example

The prompt: "We have three surfaces on one domain: a marketing site, a product catalogue, and a logged-in account area. Pick the rendering strategy for each and defend it."

**Block 1. PURPOSE: refuse the single answer, because the prompt is testing whether I treat rendering as a per-surface decision or a company-wide religion.**

I say this first, out loud: "There is no one answer for the domain. These three surfaces have different shared fractions, so they get different strategies, and the interesting part is what they share underneath rather than what they share at render time."

That sentence is the whole answer to a common trap. Interviewers ask this exact question to see whether a candidate says "we would use server rendering" and stops.

**The error most readers make here** is picking one mode and then arguing every surface into it. The cost is invisible in the round and enormous in reality: the account area gets a server render it does not benefit from, and the marketing site gets hydration it does not need.

!!! note "Say it before you read on"

    The three surfaces will share a design system and probably a router. Say one thing that becomes harder when the same component library is used under three different rendering strategies.

**Block 2. PURPOSE: compute the shared fraction for each surface, because that number, not taste, selects the mode.**

- Marketing: every region identical for every visitor, so the shared fraction is 1. Interactive regions are a navigation menu and a form. Islands over static.
- Catalogue: the page is a grid of products, a filter panel and a header. ASSUMED, and I say this is an assumption I want checked: prices vary by region and nothing else varies per user. If that holds, the shared fraction is roughly 1 per region rather than per user, so the page is cacheable per region and the personalised part is a small strip.
- Account: nothing is shared. Shared fraction is 0.

**Block 3. PURPOSE: turn the catalogue's "almost shared" into a concrete design, because that is the only hard one and the other two are now decided.**

Two candidate designs for the catalogue, and I want to eliminate one with a number rather than a preference.

- Design A: server render per request with the price already substituted. Cost: an origin render on every request, and a cache key that must include region.
- Design B: static or incremental per region, with the personalised strip fetched by the client after paint.

The eliminating number is the region count. ASSUMED at 12 regions and 50,000 products. Design B produces 12 times 50,000 documents, which is 600,000 cached objects, and that is fine for a content delivery network and impossible to build eagerly at deploy. So the answer inside Design B is incremental regeneration rather than a full static build, and I have just derived why incremental exists rather than reciting it.

**The error most readers make here** is choosing incremental regeneration because it sounds modern. It is a response to exactly one condition, which is a page count large enough that a full build is not viable, and that condition should be a number you say out loud.

!!! note "Say it before you read on"

    Before reading the last block, say why the "recently viewed" strip must not be server rendered into the cached document, in one sentence about cache keys.

**Block 4. PURPOSE: state the cost I am accepting, because a rendering choice with no stated cost reads as a preference rather than a decision.**

For each surface, one sentence of accepted cost:

- Marketing on islands: a build system that can split hydration, and the discipline to keep the island count low. If everything becomes an island, this is client rendering with extra steps.
- Catalogue on incremental regeneration: users can see a stale price for up to the staleness window. I would set that window from how often prices actually change, and if the business says prices change per minute, incremental regeneration is the wrong answer and I would move the price out of the cached document entirely.
- Account on client rendering: a slow first paint on poor networks, which I accept because the user is logged in and returning, so the JavaScript is very likely cached from a previous visit. That last clause is the actual justification, and it is worth saying, because it is why the same choice would be wrong on a first-visit surface.

**Result.** Three different strategies, each derived from one measurable property of the surface, each with a stated cost and one explicit assumption flagged for the interviewer to correct.

### 4e. Fade the scaffold

**Faded example 1.** Prompt: "Design the rendering strategy for a job board. Listings are public, searchable, and change hourly. Logged-in users see saved-job markers on each listing." Blocks 1 to 3 are given. Produce block 4.

- Block 1: this is one surface with two audiences, not two surfaces, so the question is how to keep one cached document serving both.
- Block 2: the shared fraction is high. The only per-user element is a marker on each card, which is a small amount of data with a large blast radius, because it appears on every item.
- Block 3: the design is a cached document for the listing content, with saved-job state fetched separately after paint and merged into the already-rendered cards.

??? note "Show answer"

    Block 4 states the cost and the failure mode of the split, which is the part that turns a split into a decision rather than a habit.

    "The cost I accept is a visible late change: the saved markers appear after the page has painted, so a returning user sees the page shift state once. I would make that acceptable by reserving the marker's space in the cached HTML so nothing moves, and by rendering the un-saved state as the default rather than rendering nothing, so the late arrival is a fill rather than an insertion."

    "The failure mode is the marker request failing while the page succeeds. The design has to decide what that looks like, and I would show the default un-saved state with no error, because a broken marker is not worth an error banner on a page that is otherwise fine. That is a product decision and I would flag it as one rather than making it silently."

    "The staleness window comes from the hourly change rate: regenerating on a five-minute rule means a listing can be up to five minutes out of date, which is well inside the hourly change cadence, so the window is cheap. If listings could be withdrawn instantly for legal reasons, that argument collapses and I would need a purge on withdrawal rather than a time window."

**Faded example 2.** Prompt: "Our documentation site has 40,000 pages and the full build now takes 50 minutes. Writers complain they cannot see changes quickly. What do you change?" Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: this is a build-time problem wearing a rendering-strategy costume. Nothing about the user-facing latency is broken.
- Block 2: DERIVED, 50 minutes over 40,000 pages is 75 ms of build time per page, which is not obviously wasteful. The problem is the total, not the per-page cost.

??? note "Show answer"

    Block 3, the options and the elimination. Three candidates: make the per-page build faster, build fewer pages per deploy, or stop building ahead of time. The number that eliminates: writers care about time from edit to visible, and one writer edits one page. Making the whole build 30 percent faster takes 50 minutes to 35, which does not change a writer's experience at all. So optimisation of the per-page cost is eliminated by the fact that it does not touch the quantity anyone complained about.

    The remaining two are the real fork. Incremental regeneration renders a page on first request after it is invalidated, so the writer's edit is visible after one request rather than after a full build, and the other 39,999 pages are untouched. Server rendering with a long cache time gets the same result and adds an origin dependency for a corpus that never varies per user.

    Block 4, the accepted cost. "Incremental regeneration means the first visitor to a regenerated page pays the render cost, roughly the 75 ms per page we measured plus data fetching. I accept that because it lands on one request per edit rather than on every reader."

    "The operational cost I accept is a new failure mode: if the regeneration path breaks, the site keeps serving stale pages and nobody notices, because the symptom is silence. So I would alert on the age of the oldest served document rather than on regeneration errors, since an error rate of zero is exactly what a completely broken regeneration path also produces."

    The move worth copying is refusing to treat a build-time complaint as a rendering-mode question until the arithmetic showed which quantity was actually broken.

### 4f. Check yourself

**Q1.** A colleague says "we should use streaming server rendering, it will make the page faster". What is the one question that tells you whether they are right?

??? note "Show answer"

    "Which number are you trying to move, first paint or content complete?" Streaming moves the first one and barely touches the second, as the worked table shows: 55 ms against 350 ms for the shell, but 325 ms against 350 ms for content.

    So streaming is right when the page has a slow data dependency and a useful shell, meaning there is something worth showing before the data arrives: a header, a navigation, a skeleton with correct dimensions. It is close to worthless when the page is one block of data with nothing meaningful above it, because then the shell is a blank rectangle and you have made a blank screen arrive faster.

    The follow-up worth volunteering is the cost: once you have flushed the head, you cannot change the status code or send a redirect, so error handling has to move into the stream as rendered content. That is a real complexity increase and it is the reason not to stream a page that does not need it.

**Q2.** Your page is static, paints at 230 ms, and reaches interactive at 1530 ms. A stakeholder wants interactive under 700 ms. Which of the inputs from the table can you change, and which change is the largest single win?

??? note "Show answer"

    From the arithmetic, interactive is 230 plus JavaScript transfer plus hydration execution. Three levers exist and they are not equal.

    - **Reduce JavaScript transferred.** The 1000 ms transfer term dominates. Halving to 100 KB removes 500 ms, landing at 1030 ms, which still misses.
    - **Reduce hydration execution.** The 300 ms term is the smallest of the three, so even removing it entirely leaves 1230 ms. This is why "optimise hydration" alone never hits an aggressive target.
    - **Reduce what hydrates at all.** The islands row is the same page at 490 ms, because it cuts both terms at once: 40 KB instead of 200 KB, and 60 ms instead of 300 ms. That is the only single change in the table that reaches the goal.

    The general lesson is that the two terms are correlated, since hydration cost scales with the code you shipped. Attacking them separately produces two modest wins; attacking the shipped surface attacks both, which is exactly what Module 5 is about.

### 4g. When not to use this

Each mode is a component, and each has a threshold below which it is over-engineering.

| Mode | The measurement that justifies it | Below this threshold it is over-engineering | The cheaper thing to do instead |
|---|---|---|---|
| Server rendering | Measured first content paint on your target network exceeds your budget, and the shared fraction is low enough that a cache cannot help | A logged-in tool where users return daily with warm caches, or an internal application on a fast network. You are adding an origin render and a cache layer to fix a number nobody is missing | Client rendering with a cached application shell, plus a skeleton that matches the final layout so nothing shifts |
| Static generation | The output is identical for all users, and page count times per-page build time fits in your acceptable deploy time | Fewer than roughly a hundred pages that change several times an hour. The build becomes the bottleneck for content edits, and you have traded a fast page for a slow team | Server rendering with a long cache time at the edge, which gives the same delivered document without a build step |
| Incremental regeneration | A full build no longer fits your deploy window, measured, and staleness of the window you would choose is acceptable to the business | A build under a few minutes. The extra failure mode, silently serving stale pages, costs more than the build time it saves | Keep building everything, and make the build faster or run it more often |
| Streaming server rendering | You have measured a data dependency that delays the whole document, and there is genuinely useful content above it | Data that resolves in tens of milliseconds. You have added a complicated error path to save a few frames | Server render normally, and move the slow dependency out of the request path with a cache |
| Islands or partial hydration | Measured interactive time is dominated by hydration, and the interactive fraction of the page is low | A page where most regions are interactive. Splitting a mostly-interactive page into islands produces coordination overhead plus the original cost | Ship less JavaScript. Route-level splitting gets a large share of the benefit with none of the architectural commitment |
| Client rendering | The shared fraction is near zero and users return often enough that the bundle is cached | A public, first-visit-heavy, content-led page. You are shipping a blank screen to the people you most need to reach | Static or server rendering for the entry document, and keep the client-heavy behaviour behind the login |

---

## Module 5. Hydration cost and the server-client boundary

**Time: 30 to 40 minutes reading, 20 to 30 minutes practice.**

### 5a. Predict first

A page is composed of four components on the server. One of them, the like button, needs an onClick handler. The others are pure output.

| Component | What it does | Needs client behaviour? |
|---|---|---|
| ProductPage | Fetches the product, lays out the page | No |
| PriceBlock | Formats a number with a currency | No |
| Description | Renders sanitised HTML | No |
| LikeButton | Toggles a like, optimistically | Yes |

??? note "Show answer"

    Prediction asked for: which of these four ship JavaScript to the device, and what else ships that is not code?

    Code: only LikeButton and everything it imports. The other three are executed on the server and never sent, which is the entire point of the boundary. But the answer people miss is the second half.

    Three things ship besides the code:

    | What ships | Why | The trap |
    |---|---|---|
    | LikeButton's own code, plus its transitive imports | It must run on the device | If LikeButton imports a formatting library that PriceBlock also uses, that library is now client code. The boundary is contagious downward through imports |
    | The props passed into LikeButton, serialised | The device must reconstruct the component with the same inputs | Passing a whole product object when the button needs one id sends the entire object over the wire, twice: once as HTML, once as serialised props |
    | A description of the server-rendered output around it | The client runtime must place the interactive piece inside the server output without discarding it | This payload is proportional to the tree, not to the interactive part, which is why "only one client component" does not mean "almost no bytes" |

    If you predicted "only the button", you got the code answer right and missed the data answer, which is the one that shows up as an unexplained 60 KB in a production trace.

### What hydration actually costs

Hydration means walking the server-produced markup on the device, recreating the component tree in memory, and attaching event listeners so the markup becomes interactive. Three separate costs, and they are usually discussed as one.

| Cost | Scales with | How to reduce it | How to observe it |
|---|---|---|---|
| Transfer of the framework and component code | Compressed bytes on the critical path | Fewer client components, smaller dependencies, route-level splitting | Network panel, compressed transfer size of scripts |
| Execution: tree reconstruction and listener attachment | Uncompressed code size and node count | Fewer hydrated nodes; islands; deferring non-visible regions | A long-task profile immediately after load |
| The duplicated data payload | The data you passed across the boundary | Pass identifiers rather than objects; render text on the server rather than passing the values that produce it | Search the response body for a value you know is on screen, and count how many times it appears |

### The duplicated data payload, with arithmetic

This is the cost that surprises people, so derive it once.

| Input | Value | Status |
|---|---|---|
| Products in the list | 40 | ASSUMED, one screen plus one scroll |
| Serialised size of one product, as JSON | 400 bytes | ASSUMED: an id, a title, a price, a currency, an image URL and a short blurb |
| Rendered markup for one product card | 1.5 KB | ASSUMED: a card with wrapper elements, class attributes and text |

Now compare three designs. All three display the same 40 products.

| Design | Bytes of product data on the wire | The arithmetic |
|---|---|---|
| Client renders the list from JSON | 16 KB | 40 times 400 bytes |
| Server renders the list, no interactivity | 60 KB | 40 times 1.5 KB |
| Server renders the list, and the whole list is a client component | 76 KB | 60 KB of markup plus 16 KB of serialised props for the same data |

The third row is the one to internalise. Making the list interactive did not add 16 KB of behaviour; it added 16 KB of **data you had already sent**, expressed a second way. The fix is not compression. It is moving the boundary.

- If only the like button is interactive, pass it a product id, which is roughly 8 bytes rather than 400. Across 40 cards that is 320 bytes of serialised props instead of 16 KB, a factor of 50.
- If the card must be interactive as a whole, ask what the client actually needs. A card that only needs a title and an id to work does not need the blurb passed to it, even if the blurb is displayed, because the server already rendered the blurb into markup.

!!! warning "The rule that follows"

    Send across the boundary the smallest input the client behaviour needs, not the object that produced the output. Displayed values that the client never reads should be rendered on the server and never serialised.

    This is the frontend version of "do not select star". It costs nothing to apply, and it is one of the few places where a design decision made in a whiteboard round maps to an exact byte count.

### Waterfalls in server data fetching

Server components that fetch data compose beautifully and serialise accidentally. If a component awaits data, and its child awaits data that depends on the parent's result, the two requests happen in sequence.

| Input | Value | Status |
|---|---|---|
| Latency of one server-side data call | 80 ms | ASSUMED, a same-region service call including its own database work |
| Nesting depth of the product page | 4 levels: page, panel, list, item detail | ASSUMED, from the component tree above |

- Serial: 4 times 80 ms is **320 ms** added to time to first byte, which delays every user-visible number downstream.
- Parallel: all four start at once, so the total is the slowest, **80 ms**.
- The saving is 240 ms, which is larger than the entire hydration execution term from Module 4.

```
+---------------------------+  +---------------------------+
| nested awaits             |  | hoisted awaits            |
| page waits 80             |  | all four start at 0       |
| then panel waits 80       |  | slowest returns at 80     |
| then list waits 80        |  | total 80 ms               |
| then item waits 80        |  |                           |
| total 320 ms              |  |                           |
+---------------------------+  +---------------------------+

Caption: the same four data calls, sequenced by component nesting
on the left and started together on the right. The 240 ms
difference is a code structure decision, not an infrastructure one.
```

### Four ways to flatten a waterfall, ranked by what they cost you

| Technique | What it does | When it is the right one | What it costs |
|---|---|---|---|
| Hoist the fetch | Move all data fetching to the highest common ancestor and pass results down | The children's queries do not depend on each other's results | Couples the ancestor to what its descendants need, which is a real modularity loss |
| Start then await | Begin every request, keep the promises, and await them where the values are used | Requests are independent but the values are consumed in different places | Nothing structural, but it is easy to reintroduce the waterfall with one careless await |
| Request-level deduplication and caching | The same call made in three components resolves to one request | The same entity is needed in several places | You must be sure the cache key is complete, or two different requests collapse into one wrong answer |
| Suspense boundaries plus streaming | Let the slow branch resolve after the rest of the document has been sent | One branch is genuinely slow and the rest of the page is useful without it | The error path gets harder, because the response has already started |

Deduplication has its own arithmetic. If three components each fetch the same user record at 80 ms, that is 240 ms and three origin hits without deduplication, and 80 ms with one hit. The saving is exactly the same shape as request deduplication in Module 8, which is why the two modules use the same vocabulary.

### 5d. Worked example

The prompt: "Our product page renders on the server. Time to first byte is 520 ms and the page ships 240 KB of JavaScript. Cut both."

**Block 1. PURPOSE: split the 520 ms into terms before touching anything, because two numbers with different causes are hiding inside one measurement.**

I ask for one thing: the server-side timing breakdown. ASSUMED, in the absence of it, that the page does four nested data calls at roughly 80 ms and about 200 ms of rendering and framework work, which sums to 520.

That decomposition is the whole first move. 320 ms of the 520 is sequencing, which is free to fix. 200 ms is actual work, which is not. I say out loud that I would rather have the real breakdown, and that I will design against the assumption until I do.

**The error most readers make here** is treating time to first byte as one number and proposing a cache. A cache in front of a 520 ms render hides the waterfall from you permanently, and you will pay for it the first time the cache misses under load.

!!! note "Say it before you read on"

    Say why fixing the waterfall before adding a cache is the correct order, in one sentence about what a cache does to your ability to observe the problem.

**Block 2. PURPOSE: flatten the waterfall, because it is the largest term and the cheapest to remove.**

Three of the four calls are independent: the product, the reviews summary and the shipping estimate. Only the fourth, related products, depends on the product's category. So the honest structure is three in parallel at 80 ms, then one dependent call at 80 ms, giving 160 ms rather than 320 ms.

That takes time to first byte from an assumed 520 to 360 ms with no infrastructure change, no cache and no new dependency. I would then ask whether the related-products call can be made independent by keying on the URL slug rather than on the fetched category, which would take it to 80 ms and the total to 280 ms.

**Block 3. PURPOSE: attack the 240 KB by moving the boundary, not by splitting the bundle, because splitting defers cost and moving the boundary deletes it.**

I ask which regions on the page need client behaviour. ASSUMED answer, and it is the usual one: an image gallery, an add-to-basket button, a quantity stepper and a reviews accordion. Everything else is output.

So the page is not an interactive application with a few static parts. It is a document with four widgets. That inverts the design: the default becomes server-only, and each of the four is a marked client region with a stated reason for being one.

The number I would put on the board: if the four widgets and their dependencies come to 50 KB, the page ships 50 KB instead of 240 KB. The 190 KB difference is not deferred to a later route, it is not sent.

**The error most readers make here** is reaching for lazy loading first. Lazy loading a 240 KB bundle still transfers 240 KB to a user who scrolls, and it adds loading states to regions that never needed them.

!!! note "Say it before you read on"

    Name one region on a product page that looks interactive and usually is not, and say what makes it look interactive.

**Block 4. PURPOSE: check the serialised payload, because I have just made four client components and each one is a boundary I can leak data through.**

The reviews accordion is the risk. If it receives the full review objects as props so it can expand them, the review text ships twice: once as rendered markup inside the collapsed panel, and once as serialised props. ASSUMED, 20 reviews at 600 bytes is 12 KB of duplication.

Two fixes, and I would state both and pick: pass only the ids and fetch the bodies on expand, which costs a request and a loading state, or render the bodies on the server inside the accordion as children and let the client component control only the open state, which costs nothing and is the right answer here because the bodies are already being sent as markup.

**Result.** Time to first byte from 520 ms to about 280 ms, JavaScript from 240 KB to about 50 KB, and a boundary rule that stops the number climbing back. Two of the three came from structure rather than from any new technology.

### 5e. Fade the scaffold

**Faded example 1.** Prompt: "Our dashboard's first byte is 700 ms. Profiling shows six data calls, all nested, at roughly 100 ms each, plus 100 ms of render." Blocks 1 to 3 are given. Produce block 4.

- Block 1: 6 times 100 plus 100 is 700, which matches, so the model is right and the waterfall is 600 of the 700.
- Block 2: four of the six are independent; two depend on the account's tier, which is returned by the first call.
- Block 3: restructured as first call at 100 ms, then five in parallel at 100 ms, giving 200 ms plus 100 ms of render, which is 300 ms.

??? note "Show answer"

    Block 4 is the boundary and payload check, and the interesting move is noticing that a dashboard inverts the product-page conclusion.

    "A dashboard is the opposite shape to a product page: most regions are interactive, so islands would fragment it without saving much. The boundary decision here is not which components are client components, it is which data crosses. Six calls returning full result sets, then serialised into client charts, means every number is on the wire twice."

    "So I would pass the client chart components the already-aggregated series rather than the raw rows, and render the summary tiles entirely on the server since they are text. If a tile shows a single figure, the client never needs the series that produced it. That is the same rule as the product page, applied to a page where the code split does not help."

    "One thing I would measure before committing: whether the 100 ms of render is dominated by the charts. If it is, moving the aggregation to the server has just moved cost from the client to the server, and I need to know that before I claim a win."

**Faded example 2.** Prompt: "We moved a list to a server-rendered boundary and the page got 40 KB bigger. Explain, and fix it." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: bytes going up after moving work to the server is the signature of duplicated data, not of extra code.
- Block 2: the list is 60 items, each item is a client component so it can handle a click, and each receives the whole item object.

??? note "Show answer"

    Block 3, the arithmetic. ASSUMED 60 items at roughly 650 bytes of serialised props is about 39 KB, which matches the observed 40 KB almost exactly, and that match is the confirmation that the diagnosis is right rather than plausible. The markup was already being sent; the props are pure addition.

    Block 4, the fix and its cost. "Move the boundary down. The item does not need to be a client component; only the clickable element inside it does. If that element receives an id rather than an object, 60 items times 8 bytes is 480 bytes instead of 39 KB, which is a factor of about 80."

    "The cost I accept is that the click handler now has less context, so if the interaction needs the item's title for an optimistic update, I have to either pass that one field, which is 60 times perhaps 40 bytes and still 30 times smaller, or read it from the already-rendered markup, which I would not do because it couples behaviour to presentation."

    "The check that this worked is not the bundle report, because no code changed. It is the response body size, and I would add an assertion on it in continuous integration, because this regression is invisible in every dashboard a frontend team normally watches."

### 5f. Check yourself

**Q1.** A teammate says "server components mean we ship less JavaScript, so the page will be faster". Under what condition is that false?

??? note "Show answer"

    It is false whenever the data crossing the boundary is larger than the code you removed. From the arithmetic above, one list can add 39 KB of serialised props while removing perhaps 10 KB of component code, so the transfer goes up and the page gets slower on a slow network.

    It is also false in a second way: shipping less code does not reduce time to first byte, and if the move to server rendering introduced a data waterfall, first byte can rise by hundreds of milliseconds. That is a strictly larger regression than the bundle saving, because it delays everything downstream.

    The honest statement is narrower: server-first boundaries reduce shipped code, and they reduce total transfer only if you also control what crosses the boundary. The measurement that settles it is total response bytes plus time to first byte, before and after, not the bundle report.

**Q2.** You have a page with a 1.2 s server data dependency for one panel, and the rest of the page is ready immediately. Give two designs and the condition that picks between them.

??? note "Show answer"

    **Design A, stream it.** Send the document without the panel, mark the panel's place, and flush its markup when the data resolves at 1.2 s. The user sees a complete page at first byte and one region fills in.

    **Design B, move it off the server request entirely.** Render a placeholder, and let the client fetch the panel after load. The server request never waits.

    The condition that picks between them is whether the panel's content must be present for a crawler or for a user with JavaScript disabled or failing. If it must, stream it, because streamed markup is in the document. If it need not, the client fetch is simpler, has an easier error path, and takes the slow dependency off your server's request budget entirely.

    A secondary condition worth naming: if the 1.2 s call is slow because of load rather than distance, streaming keeps a server request open for 1.2 s per visitor, which consumes connection and memory capacity on your fleet. At high traffic that operational cost can decide it on its own.

### 5g. When not to use this

| Question | Answer |
|---|---|
| The measurement that justifies moving the boundary at all | A profile showing hydration execution as a top main-thread cost after load, or a response body where you can find the same displayed value twice |
| The threshold below which it is over-engineering | A page under roughly 50 KB of client JavaScript that reaches interactive inside your budget. Splitting it into server and client regions adds a mental model and a class of bugs for a saving nobody can measure |
| The cheaper alternative | Route-level code splitting plus deleting one large dependency. Both are reversible in an afternoon; a boundary redesign is not |
| When not to flatten a waterfall | When the calls genuinely depend on each other. Forcing parallelism by pre-fetching things you might not need trades latency for wasted origin load, and on a low hit rate that is a bad trade |
| When not to stream | When the slow dependency is not needed in the document. Client-fetching it is simpler and moves the cost off your fleet |
| The over-application failure mode | A component tree where every leaf is a separately-fetched, separately-streamed boundary. You have rebuilt a waterfall out of suspense boundaries and made it much harder to see |

---

## Module 6. Component design: the public API surface

**Time: 35 to 45 minutes reading, 25 to 35 minutes practice.**

### 6a. Predict first

A team ships this component API to twelve consuming teams. Read the prop list before continuing.

| Prop | Type |
|---|---|
| items | array |
| value | string |
| onChange | function |
| isSearchable | boolean |
| isMulti | boolean |
| isClearable | boolean |
| isDisabled | boolean |
| placeholder | string |
| renderOption | function |
| menuPlacement | string |
| className | string |

??? note "Show answer"

    Prediction asked for: what breaks first, and what is the number that predicts it?

    The number is the count of reachable configurations from the boolean props. Four independent booleans give 2 to the power of 4, which is **16 combinations**, and the component must behave correctly in all of them or document which are illegal. In practice the team tested perhaps five.

    What breaks first, in order:

    | Break | Why |
    |---|---|
    | A combination nobody considered | `isMulti` plus `isClearable` raises a question the API never answered: does clear remove one value or all of them? Two consumers assume different answers |
    | The type of `value` | With `isMulti` true, `value` is logically an array, but the signature says string. Every consumer casts, and the cast is the bug |
    | `renderOption` becomes the escape hatch for everything | Consumers who cannot get what they need from props render arbitrary markup, and the component loses control of its own accessibility semantics |
    | Removing anything | Twelve consumers depend on the surface, so every prop is now permanent unless you ship a major version and a migration |

    The design fix is not fewer features. Replace independent booleans with a single enum where the options are mutually exclusive, and give multi-select its own component rather than a flag, because its value type differs. Four booleans at 16 combinations become one `selectionMode` of two values plus two genuinely independent flags, which is 2 times 2 times 2, or **8**, and none of them contradictory.

### The API surface is the only irreversible artefact

Internals can be rewritten on any Tuesday. A prop that twelve teams import cannot be removed without a coordinated migration, so the API deserves most of the round's minutes.

| Property of the API | Why it is expensive to get wrong | The cheap version of the decision |
|---|---|---|
| Every prop is a permanent promise | Removal requires a major version and consumer work | Ship the smallest surface that serves your three named call sites, and add on request |
| Defaults are behaviour | Changing a default silently changes every consumer's product | Treat a default change as a breaking change, because it is one that no compiler catches |
| Types are a contract | Widening is safe, narrowing breaks builds | Start narrow. Widening later is a minor version |
| What the component announces to assistive technology | It is user-visible behaviour that no test in the consumer's repository asserts | Write it down as part of the API and version it like a prop |

### Controlled and uncontrolled, precisely

The distinction is about who owns the value, and confusion here produces the most common bug class in component libraries.

| | Uncontrolled | Controlled |
|---|---|---|
| Who owns the current value | The component, in its own state | The consumer, passed in on every render |
| What the consumer writes | A `defaultValue`, and optionally an `onChange` to observe | A `value`, and an `onChange` that must update it or nothing happens |
| Reading the value | Requires a ref or the change callback | Trivial, the consumer already has it |
| Validation, formatting, forcing a value | Hard or impossible from outside | Natural |
| Typical failure | The consumer cannot reset or preset it from outside | The consumer forgets to update state, and the field appears frozen |

Three rules that resolve nearly every real case:

1. **Support both, with the standard pair.** `value` plus `onChange` for controlled, `defaultValue` for uncontrolled, and the presence of `value` selects the mode.
2. **Fix the mode at mount and never switch.** A component that starts uncontrolled and receives a `value` later will discard user input at an unpredictable moment. Warn loudly in development.
3. **Never make `onChange` optional in controlled mode.** A controlled component with no handler is a read-only field pretending to be editable, which reads as a bug to every user who tries it.

### Compound components and inversion of control

Both exist to solve the same problem: consumers need arrangements you did not anticipate. They solve it at different costs.

| Pattern | What it looks like | Use it when | What it costs |
|---|---|---|---|
| Props for everything | One component, many props | Two or three call sites that differ only in content | Prop growth is superlinear in the number of variations, which is what the pre-question showed |
| Compound components | A parent plus named parts the consumer arranges | Consumers need to reorder, wrap or style parts independently | Invalid arrangements become expressible, and the parent-to-part communication is implicit, so mistakes fail at runtime rather than at build time |
| Inversion of control | The consumer supplies a render function or a slot | The variation is in how one item is rendered, not in the widget's behaviour | The consumer writes more code and can violate the accessibility contract unless you constrain what they receive |
| Headless plus a styled preset | Behaviour and accessibility with no markup opinion, plus a default styled build on top | You serve teams with genuinely different visual systems | Two artefacts to maintain and version, and adopters must write markup, which many teams will do worse than you would |

The rule that stops this becoming an ideology: **escalate only when a real call site forces you to.** Start with props. Move to compound when a consumer needs to reorder parts. Move to a render function when the variation is in one item's content. Go headless only when the visual systems genuinely differ, because headless moves the accessibility burden to consumers who will not carry it as carefully as you would.

### The layered component

```
+------------------------------------------+
| styled preset: tokens, spacing, variants |
+------------------------------------------+
                   |
+------------------------------------------+
| accessibility layer: roles, keys, focus  |
+------------------------------------------+
                   |
+------------------------------------------+
| behaviour core: state machine, no markup |
+------------------------------------------+

Caption: the three layers of an interactive component. Consumers
may replace the top layer. Replacing the middle layer is what
happens when a library goes headless, and it is why headless
libraries must publish the contract the middle layer enforced.
```

### The accessibility contract, written as API

The contract has three parts, and stating them separates a component that is accessible from one that says it is.

| Part | Example, for a combobox | Why it belongs in the API |
|---|---|---|
| What the component guarantees | Roles and relationships are applied; arrow keys move the active option; Escape closes; focus stays on the input; the active option is announced | Consumers must be able to rely on it without reading the source |
| What the component requires from the consumer | An accessible label, and unique option ids or a key function | If the consumer omits it, the component cannot be conformant, so it must be a required prop rather than a documentation note |
| What the component forbids | Overriding role, aria-expanded, aria-activedescendant or tabindex through prop spreading | Without this, a consumer can silently break conformance in a way no test in your repository catches |

Module 7 makes each guarantee testable. The point here is narrower: these are API decisions, not implementation details, because changing any of them changes what consumers must do.

### 6d. Worked example

The prompt: "Design a select component for our shared library."

**Block 1. PURPOSE: get three call sites on the board before designing anything, because an API designed from imagination serves an imaginary consumer.**

I ask for three, and I push for an awkward one. What I write down:

- A country picker in a settings form: single value, searchable, hundreds of options.
- A status filter on a table toolbar: single value, six options, must render a coloured dot next to each.
- An assignee picker in a comment box: multiple values, avatars, and it must submit on selection with no confirm step.

Those three already contain the design. The second forces custom option content. The third forces a different value type and a different close behaviour. If I design only for the first, I will invent booleans later to retrofit the other two.

**The error most readers make here** is starting from the props. Props are the output of this exercise, not the input.

!!! note "Say it before you read on"

    Two of these three call sites can share one component. One probably should not. Say which, and the property that separates it.

**Block 2. PURPOSE: decide state ownership before shape, because the ownership question changes the type signature and the type signature is the hardest thing to change later.**

Two pieces of state exist and they have different owners.

- The **selected value** is the consumer's. It is form data, it is submitted, it survives the component. So `value` plus `onChange`, with `defaultValue` for the uncontrolled case.
- The **open state, the search query and the active option** are the component's. No consumer has a legitimate reason to own them, and exposing them invites consumers to drive the widget from outside and break the keyboard model.

I make one deliberate exception: `open` and `onOpenChange` as optional controlled props, because the table toolbar case may need to close the menu when a filter is applied elsewhere. I note that this is the one place I am widening the surface, and I say why.

**Block 3. PURPOSE: pick the extension mechanism from the call sites rather than from preference, because this is where component libraries acquire their permanent complexity.**

The second call site needs custom option content. Three ways to allow it, and I eliminate two:

- Add `optionColor` and `optionIcon` props: eliminated, because it solves exactly this case and the next request adds two more props. This is the boolean explosion in a different costume.
- Go fully headless: eliminated as over-correction, since the library exists so twelve teams do not each implement a keyboard model.
- A `renderOption` function receiving the option plus its state: chosen. It solves the general case with one prop.

The constraint that makes it safe: `renderOption` returns content, and the component owns the element that carries the role, the id and the selection state. The consumer cannot spread props onto the option root. That single restriction is what stops the escape hatch from becoming an accessibility hole.

**The error most readers make here** is handing the consumer the whole option element. It looks more flexible and it silently transfers the conformance burden to twelve teams.

!!! note "Say it before you read on"

    Before the last block, say what `renderOption` must receive besides the option itself, and why the component cannot leave it out.

**Block 4. PURPOSE: name the illegal states and the versioning consequences, because that is the part nobody else in the loop will cover.**

Illegal states I would make unrepresentable rather than documented:

- Multi-select and single-select are different components, not a flag, because their `value` types differ. This removes an entire class of casting bugs and is the single strongest decision in the design.
- Open with no options and no empty message: the empty message gets a default rather than being optional, so the state cannot be reached.
- Controlled `value` supplied after mounting uncontrolled: warn in development and keep the original mode, because switching discards user input.

Versioning consequences I would state up front: changing the default of `closeOnSelect`, tightening the type of `value`, or changing what the component announces are all breaking changes even though only one of them fails a build. I would ship the first with a codemod, the second as a major, and the third never without a flag.

**Result.** An API of roughly seven props plus a render function, two components rather than one, three named illegal states, and a written statement of what breaks consumers. That is a stronger artefact than a twelve-prop component that covers more cases.

### 6e. Fade the scaffold

**Faded example 1.** Prompt: "Design a modal dialog for the shared library." Blocks 1 to 3 are given. Produce block 4.

- Block 1: three call sites are a confirm dialog with two buttons, a form dialog that must not close on outside click while dirty, and a full-screen media viewer on small screens.
- Block 2: `open` and `onOpenChange` are the consumer's, because opening is caused by consumer events. Focus location and scroll locking are the component's, and no consumer should own them.
- Block 3: compound parts (a root, a trigger, a content region, a title and a description) rather than a `title` prop, because the form case needs arbitrary content and the media case needs no title chrome at all.

??? note "Show answer"

    Block 4, illegal states and versioning. The strong move here is noticing that a dialog's illegal states are accessibility states.

    "Three states I make unrepresentable. First, content with no accessible name: the title part is required, and if a consumer genuinely has no visible title they must pass a label, so the component cannot render nameless."

    "Second, more than one dialog owning focus: opening a second while one is open either stacks with the previous one made inert, or is refused, and I would pick stacking with inert underneath and say so. Third, dismissal disabled entirely: Escape always closes, because a dialog a keyboard user cannot leave is a trap. The dirty-form case gets an `onDismissRequest` that can show a confirm, not a way to ignore Escape."

    "Versioning: changing what happens on outside click is a breaking change even though it is a behaviour and not a signature, because two of my three call sites depend on opposite answers. So it is an explicit prop from version one rather than a default I might revisit."

**Faded example 2.** Prompt: "Design a toast or notification component." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: call sites are a success confirmation after saving, an error with a retry action, and an upload progress toast that updates in place over 30 seconds.
- Block 2: the queue is the component's, since two teams firing toasts must not each manage stacking. The content is the consumer's. Dismissal timing is shared, which is the tension in this design.

??? note "Show answer"

    Block 3, the extension mechanism. The awkward call site is the third: a toast that mutates rather than being replaced. That forces an imperative API, which is unusual and worth defending. "A toast is triggered by an event, not rendered by a tree, so the API is a function call that returns a handle, and the handle can update or dismiss. Trying to express this declaratively means the consumer owns an array of toasts, and then two teams own two arrays and the stacking is wrong."

    Block 4, illegal states and versioning. "Auto-dismiss plus an action is illegal in my design: a toast with a retry button does not auto-dismiss, because a timed dismissal races the user reaching for the button, and on a slow reader or a screen reader user it always wins. So the presence of an action sets the dismissal to manual, and I would rather enforce that than document it."

    "The second illegal state is an unbounded queue. Thirty toasts stacked is not a design, it is an outage. I cap the visible count and collapse the rest into a count, and I would pick the cap from the tallest supported viewport rather than choosing a round number."

    "Versioning: the announcement politeness is API. A success toast is polite, an error is assertive, and changing that later changes what interrupts a screen reader user mid-sentence. It goes in the contract, not in the implementation."

### 6f. Check yourself

**Q1.** A consumer asks you to add a `hideLabel` boolean to your input component. What do you ask, and what are the two possible right answers?

??? note "Show answer"

    Ask what they are actually trying to do, because `hideLabel` is a solution rather than a requirement. There are two common underlying needs and they have different answers.

    - **They want the label visually hidden but still present for assistive technology**, usually in a compact toolbar. The right answer is to support it, because removing the label entirely would be a conformance failure and consumers will do exactly that if you refuse. Name it for what it does, something like `labelVisibility`, and keep the label required.
    - **They want no label at all**, because a nearby heading or an icon explains the field. The right answer is to refuse the prop and require `aria-label` or a labelled-by reference instead, so the component still cannot render nameless.

    The general principle is that a boolean named after a visual outcome usually hides a semantic decision. Asking one question converts a prop request into a design decision, and it is a cheap habit that keeps a surface small.

**Q2.** Your component has six boolean props. What is the number you would put on the board to argue for a redesign, and what would you replace them with?

??? note "Show answer"

    Six independent booleans give 2 to the power of 6, which is **64 reachable configurations**. That is the number to say out loud, because it converts a vague sense of complexity into a testing obligation nobody is meeting. If your test suite covers eight, you have tested one eighth of the surface consumers can express.

    The replacement is to sort the six into groups by whether they are genuinely independent:

    - Mutually exclusive booleans become one enum. Three exclusive booleans at 8 combinations become one prop with 3 values, and the illegal combinations stop existing rather than being documented.
    - Booleans whose true state changes the type of another prop become a separate component. That is the multi-select case, and it removes a cast rather than a combination.
    - What remains, usually one or two, stays boolean, and that is fine.

    Applied to six, a typical result is one enum of three, one separate component, and two booleans, which is 3 times 2 times 2, or **12** configurations rather than 64. The reduction is the argument.

### 6g. When not to use this

| Pattern | The measurement that justifies it | The threshold below which it is over-engineering | The cheaper alternative |
|---|---|---|---|
| A shared component at all | The same widget has been implemented independently at least three times, and at least two of them are wrong in the same way | Two call sites in one application. Copying is cheaper than abstracting, and the second copy is what teaches you the right abstraction | Copy it, and write down the differences you observe. Abstract on the third |
| Compound components | A consumer has a legitimate need to reorder or wrap parts, which you can name | All consumers use the same arrangement. You have added invalid-arrangement bugs and lost build-time checking for nothing | Props, with one slot for the one region that varies |
| Inversion of control | Two or more consumers need different content in the same structural position | One consumer needs custom content. Give that one a slot rather than converting the API to render functions | A single, narrowly typed slot prop |
| Headless | Two or more consuming teams have genuinely incompatible visual systems, and you can point at both | One design system. Headless doubles your artefacts and pushes accessibility work onto teams who will do it less carefully | A styled component with a token layer, which absorbs most theming needs |
| Controlled and uncontrolled both | Consumers need to preset, reset or validate from outside, which you can name a call site for | Nobody has asked. Supporting both doubles the state paths and creates the mode-switching bug class | Uncontrolled with a change callback, which covers most form usage |
| Versioning policy and codemods | More than roughly five consuming teams, or consumers you cannot upgrade yourself | One or two consumers inside the same repository. You can change the call sites in the same pull request | Change it and fix the call sites. Do not build a deprecation process for two imports |

---

## Module 7. Accessibility as engineering

**Time: 40 to 50 minutes reading, 25 to 35 minutes practice.**

### 7a. Predict first

A team ships a "custom dropdown". Here is what it is made of, described in plain terms.

| Piece | How it is built |
|---|---|
| The closed control | A styled container element with a click handler and the current value as text |
| The list | A container element rendered when open, with one child per option |
| An option | A child element with a click handler that sets the value and closes the list |
| Styling | The container loses its outline on focus, because the designer disliked it |

??? note "Show answer"

    Prediction asked for: list what a keyboard-only user cannot do, and what a screen reader user is told.

    Keyboard, in order of severity:

    | Cannot | Why |
    |---|---|
    | Reach the control at all | A generic container is not focusable. Nothing puts it in the tab order |
    | Open it | A click handler alone does not fire for Enter or Space on a non-button |
    | Move between options | Arrow keys do nothing, because nothing implements the movement |
    | Close it without choosing | Escape does nothing |
    | See where they are | The outline was removed, so even the parts that do receive focus are invisible |

    Screen reader: the control announces its text and nothing else. No role, so it is not identified as a control. No expanded state, so opening it is silent. No relationship between the control and the list, so the options are unassociated content that may not be discoverable at all. Selecting an option changes text somewhere on the page and nothing is announced.

    The important part of this answer is that **none of these are fixed by adding ARIA attributes to the same markup**. Roles describe; they do not implement. Every keyboard row above requires code. That is the sentence this whole module exists to make concrete, and it is the fastest way to separate an engineer who has built a widget from one who has read about one.

### Conformance is a claim, so state it precisely

The Web Content Accessibility Guidelines define success criteria at three levels: A, AA and AAA. Level AA is the level normally written into policy and procurement, so it is the default target to state in a round. The specification is published by the W3C at [w3.org/TR/WCAG22](https://www.w3.org/TR/WCAG22/).

Three properties of a conformance claim that candidates routinely get wrong:

- **Conformance applies to whole pages, not to components.** A component cannot be conformant by itself, because criteria such as focus order and page structure are properties of the page it sits in. A component can be built so that a conformant page is achievable, which is what a component library actually ships.
- **Conformance applies to complete processes.** If checkout has five steps and step four is unusable, the process fails even if four pages pass.
- **Anything on the page can break it.** A third-party widget that traps focus fails the page it is on, regardless of the quality of your own code.

The four principles are worth keeping in your head as a triage ladder rather than as a recitation, because they sort bugs by who is affected and what the fix costs.

| Principle | The question it asks | A typical failure |
|---|---|---|
| Perceivable | Can the information be received at all? | An icon-only button with no accessible name; text on a background with insufficient contrast |
| Operable | Can every action be performed by every input method? | A menu that only opens on hover; a drag interaction with no keyboard equivalent |
| Understandable | Is the behaviour predictable and the error recoverable? | A form that submits on change; an error that says "invalid" with no indication of which field |
| Robust | Will assistive technology receive the right information? | State conveyed only by a class name, so nothing announces the change |

### The authoring pattern, per interactive component

Each interactive widget has an expected structure, key set and focus rule. Consumers of your component and users of your product both rely on the expectation. This table is the working set for an interview.

| Component | Structure and state | Keys that must work | Focus rule | What must be announced |
|---|---|---|---|---|
| Button | Use the native button element | Enter and Space activate | Focus stays on the button | Its name, and its pressed state if it is a toggle |
| Link | Use the native anchor with a destination | Enter activates | Focus stays | Its name, which must make sense out of context |
| Disclosure | A button that controls a region, with an expanded state | Enter and Space toggle | Focus stays on the button | Name plus expanded or collapsed |
| Tabs | A tab list, tabs, and panels associated to their tab | Arrow keys move between tabs, Home and End jump to the ends | Roving: one tab stop for the whole list | Selected state, and position such as three of five |
| Menu button | A button that opens a menu of actions | Enter, Space or Down opens; arrows move; Escape closes; typing jumps | Focus moves into the menu, and returns to the button on close | Menu opened, item names, submenu presence |
| Combobox | A text input plus a popup list, with an expanded state and an active option | Down opens and moves; Up moves back; Enter selects; Escape closes then clears | Focus stays on the input; the active option is referenced rather than focused | Expanded state, the active option, and the number of results when it changes |
| Listbox | A container of options with a selected state | Arrows move, Home and End jump, typing jumps, Space toggles in multi-select | Roving, or an active-descendant reference from the container | Option name, selected state, position in set |
| Modal dialog | A dialog with an accessible name, and everything else made inert | Escape closes; Tab cycles within | Focus moves in on open, is trapped, and is restored on close | The dialog's name on open |
| Tooltip | Supplementary text tied to its trigger | Escape dismisses; it must appear on focus, not only on hover | Focus never moves into it | The text, as part of the trigger's description |
| Status message | A region that exists before the message does | None; it is not focusable | Focus does not move | The message, without stealing focus |

!!! warning "The rule that precedes all of these"

    If a native element does the job, use it and stop. A native button is focusable, is activated by Enter and Space, is announced with the right role, works with voice control and forms, and costs nothing. Every custom replacement re-implements all of that, usually incompletely.

    The strongest thing you can say in an accessibility discussion is "we do not need a custom widget here". It is faster to build, cheaper to maintain and impossible to get subtly wrong.

### Focus management, the three mechanisms

**Mechanism one: the trap.** A modal dialog must contain focus, because a user who tabs out lands on content that is visually behind an overlay. Trapping is three parts: move focus in on open, cycle at the boundaries, and make the rest of the page inert so that assistive technology cannot reach it either. Hiding content visually is not enough, because a screen reader reads the accessibility tree rather than the pixels.

**Mechanism two: restoration.** On close, focus returns to whatever opened the dialog. The case everyone forgets: the trigger no longer exists, because the dialog deleted the row it lived in. Falling back to the document body drops the user at the top of the page and loses their position entirely, so the fallback should be the nearest stable ancestor, such as the list that contained the deleted row.

**Mechanism three: roving tabindex.** In a composite widget the whole widget is one tab stop and arrow keys move inside it. The arithmetic makes the case:

| Design | Tab stops to cross a 20 by 10 grid | Consequence |
|---|---|---|
| Every cell focusable | 200 | Tabbing past the grid to reach the next control takes 200 key presses |
| Roving tabindex | 1 | One press to enter, arrows to move, one press to leave |

The alternative to roving is an active-descendant reference, where focus stays on a container and the container names which child is currently active. Choose between them on one question.

| | Roving tabindex | Active descendant |
|---|---|---|
| Where the operating system focus is | On the active child | On the container, never moving |
| Best for | Grids, toolbars, tab lists, anything where the child is a real control | Comboboxes, where focus must stay in a text input so typing keeps working |
| Common failure | Forgetting to update which child is the single tab stop, so focus is lost after re-render | The referenced id does not exist after filtering, so nothing is announced |

```
+----------------------+
| trigger button       |
| focus starts here    |
+----------------------+
           |
           v
+----------------------+
| dialog opens         |
| focus moves inside   |
| rest of page inert   |
+----------------------+
           |
     +-----+-----+
     |           |
     v           v
+-----------+ +--------------------+
| Tab       | | Escape closes      |
| cycles    | | focus returns to   |
| inside    | | the trigger        |
+-----------+ +--------------------+
                       |
                       v
        +-----------------------------+
        | trigger no longer exists    |
        | focus goes to the list that |
        | contained it, not the body  |
        +-----------------------------+

Caption: the focus lifecycle of a modal dialog, including the
case that is usually missed, where the element that opened the
dialog was removed by the action taken inside it.
```

### Accessible names, and the icon-button trap

Every interactive element needs a name that assistive technology can read. The sources are consulted in a precedence order, and knowing the order explains most naming bugs.

1. An explicit labelled-by reference to other elements on the page.
2. An explicit label attribute carrying a string.
3. A native association, such as a form label bound to its control.
4. The element's own text content, for elements that take their name from content, such as buttons and links.
5. A title attribute, which is a last resort and is not exposed to all users.

Two consequences worth stating out loud in a round:

- **A labelled-by reference overrides visible text.** So an icon button with visible text plus a stale label attribute announces the stale string, and no visual review will ever catch it.
- **An icon button has no text content**, so it has no name at all unless you provide one. This is the single most common naming failure, and it is invisible in every screenshot.

### Live regions, used sparingly

A live region announces content changes without moving focus. Three rules cover nearly all correct use.

- **The region must exist in the document before the text changes.** Inserting a live region and its content at the same moment frequently announces nothing, because there was no region to observe.
- **Polite is the default.** Assertive interrupts whatever is being read, so reserve it for errors and events the user must handle now.
- **Announce a summary, not a payload.** "12 results" is useful. Marking a whole result list as live means every keystroke re-announces the entire list, which is worse than silence because the user cannot escape it.

Two adjacent behaviours belong here because they are functional requirements rather than polish:

- **Reduced motion.** When the user has asked the operating system for reduced motion, shorten or remove transitions, disable parallax and stop anything that moves without being asked. The correct response is not "delete all animation": a shortened transition still communicates that something changed.
- **Error announcement in forms.** Bind the error text to the field so it is read when the field is reached, keep it present rather than transient, and never convey the error state with colour alone. On submit, move focus to a summary that lists the failures and links to each field, because a user who cannot see the page has no other way to find out how many problems exist.

### Testing: sort by what a machine can decide

Percentages of "issues caught by automation" circulate widely and depend entirely on the sample. Sorting by decidability is more useful and cannot go out of date.

| Tier | Examples | Who checks it |
|---|---|---|
| Machine-decidable | Image with no alternative text attribute; form control with no programmatic label; contrast of solid text on a solid background; duplicate element ids; invalid ARIA attribute or value; a positive tabindex | An automated rule set, in continuous integration, failing the build |
| Partly decidable | Focus order plausibility; whether a visible focus indicator exists; link text that is generic; whether a live region exists at all | A tool can flag candidates; a human decides |
| Not machine-decidable | Whether alternative text conveys the right meaning; whether an announcement is comprehensible; whether the keyboard model matches the expected pattern; whether an error message tells the user what to do | A person, using a keyboard and a screen reader |

The practical consequence is the acceptance test to state in a round: automated rules gate every pull request and catch the first tier only; a keyboard-only pass is required for every new interactive flow; one screen reader pass is required for every new interactive component; and a zoom and reflow check is required for every new layout.

**The five-minute manual pass**, which is what to describe when asked how you would verify a design:

1. Put the mouse away. Tab through the whole flow and complete it.
2. At every stop, check that the focus indicator is visible against the actual background.
3. In every composite widget, use the arrow keys and Escape, not just Tab.
4. Zoom to 400 percent and check that content reflows without a horizontal scrollbar.
5. Turn on a screen reader and complete one task, listening specifically at the moments where state changes.

### 7d. Worked example

The prompt: "Make our custom combobox conformant. It is the one from the pre-question."

**Block 1. PURPOSE: define the interaction model in plain language before touching any attribute, because the attributes describe a model that must already exist.**

I write the model as sentences, not as markup: focus lives in the text input at all times; typing filters; Down opens the list and moves the active option; Up moves back; Enter selects the active option and closes; Escape closes without selecting, and a second Escape clears the input; clicking outside closes; the input's value is not overwritten by the active option while browsing.

That last clause is the one people miss, and it is a design decision rather than an accessibility one: browsing with arrows must not destroy what the user typed.

**The error most readers make here** is opening the accessibility work by adding roles. Roles applied to a widget with no interaction model produce something that announces itself correctly and still does not work.

!!! note "Say it before you read on"

    Say why focus must stay on the input rather than moving to the options, in one sentence about what would stop working.

**Block 2. PURPOSE: map the model onto roles, relationships and state, because that is what makes the working widget legible to assistive technology.**

Four mappings, each traceable to a sentence from block 1:

- The input is a combobox that controls the list, and it carries an expanded state. That comes from "Down opens the list".
- The list is a listbox, and each child is an option with a selected state. That comes from "Enter selects the active option".
- The input names its active option by reference, using active descendant rather than moving focus. That comes from "focus lives in the input at all times".
- A status region reports the result count when filtering changes it. That comes from "typing filters", which is a change a sighted user sees and a screen reader user otherwise does not.

**Block 3. PURPOSE: handle the focus and announcement edge cases, because this is where a widget that passes a demo fails in production.**

- Filtering to zero results: the active descendant reference now points at an id that no longer exists. Clear the reference and announce "no results" through the status region, rather than leaving a dangling reference which silently announces nothing.
- Closing on selection: focus stays on the input, and the value updates. Announcing the selection is handled by the input's own value change, so a second announcement through the status region would say everything twice.
- The list scrolls: the active option must be scrolled into view as arrows move, or a sighted keyboard user loses their position even though the widget is technically correct.
- Announcement volume: I announce the count on a debounce rather than on every keystroke, because a user typing six characters should not hear six counts.

**The error most readers make here** is announcing more, on the assumption that more speech is more accessible. Verbosity is its own failure: an interface that narrates everything is harder to use than one that narrates only what changed.

!!! note "Say it before you read on"

    Before the last block, say what should happen when the user types a character while the list is open and an option is active. Two behaviours are defensible; name one and its consequence.

**Block 4. PURPOSE: write the tests, because an accessibility claim with no test is a promise that decays with the next refactor.**

Four tests, one per tier plus the manual gate:

- Automated rules in continuous integration, asserting the machine-decidable tier on a rendered page containing the component in both open and closed states. The closed state alone would miss every relationship.
- Unit or integration tests asserting the key model directly: Down opens, arrows move the active reference, Escape closes, Enter selects, and the input value survives browsing. These are ordinary tests, and they are the ones that catch the regression a year later.
- An assertion that the status region exists before the first filter, since a region created at announcement time frequently announces nothing.
- A manual gate in the pull request template: one keyboard-only pass and one screen reader pass, recorded, for any change to this component.

**Result.** The widget now works before it announces, announces what it does, and has tests that fail if either half breaks. Note the order: model, then semantics, then edge cases, then tests. Reversing the first two is the most common way this work is done badly.

### 7e. Fade the scaffold

**Faded example 1.** Prompt: "Our data table has sortable column headers built as clickable header cells. Make it conformant." Blocks 1 to 3 are given. Produce block 4.

- Block 1: the model is that each sortable header is a control, activating it cycles ascending, descending and possibly none, and the current sort is a property of the column rather than of the page.
- Block 2: the header cell carries the sort state, and the control inside it is a real button so that Enter and Space work without re-implementation.
- Block 3: edge cases are the initial unsorted state, which must be distinguishable from ascending; and the announcement on sort, which must say what happened without reading the whole table.

??? note "Show answer"

    Block 4, the tests. The interesting move is that the most valuable test here is not an accessibility test at all.

    "Automated rules will catch a header cell with no accessible name and a control with an invalid state value, and nothing else in this list. So the tests that matter are behavioural: activating a header with the keyboard changes the sort, the sort state on the header matches the rendered order, and only one column carries a sort state at a time. That last one is the bug that actually ships, and it is a state-machine bug rather than an accessibility bug."

    "The manual gate is one screen reader pass specifically on the sort action, because the question a tool cannot answer is whether the announcement is useful. 'Sorted ascending by price' is useful. 'Button pressed' is technically correct and worthless."

    "One thing I would deliberately not test automatically: whether the announcement text is good. Asserting an exact string in a test freezes the copy and gives false confidence. I would rather have a human check it once per change."

**Faded example 2.** Prompt: "Design the accessibility contract for a drag-to-reorder list." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: dragging is a pointer gesture with no keyboard equivalent by default, so an alternative path is not an enhancement, it is the requirement.
- Block 2: the model is that each item can be picked up, moved and dropped, and every one of those has a keyboard equivalent: a key to pick up, arrows to move, a key to drop, Escape to cancel and restore.

??? note "Show answer"

    Block 3, the focus and announcement edge cases. "Reordering moves the item, so focus must move with it or the user loses their place. Every move needs an announcement of the new position, and 'moved to position 3 of 8' is the useful form because position alone is meaningless without the total. Cancel must restore both the order and the focus, which means the original index has to be stored at pick-up rather than recomputed."

    "The hardest edge case is the list changing underneath a drag, for example another user reordering it in a collaborative document. I would freeze the list for the duration of a keyboard drag and apply remote changes on drop, because a moving target cannot be operated with arrow keys at all."

    Block 4, the tests. "The behavioural tests are the contract: pick up, move, drop and cancel all work from the keyboard, and cancel restores exactly. The announcement test asserts that a region exists and receives one message per move, not the message text. The manual gate is a full keyboard reorder of a five-item list, which takes about thirty seconds and catches everything that matters."

    "I would also test the thing most likely to be wrong under time pressure: that the pointer path and the keyboard path go through the same state machine. Two implementations of reordering will diverge, and the keyboard one will be the one that rots, because nobody uses it during development."

### 7f. Check yourself

**Q1.** A designer asks you to remove focus outlines because they look bad on the brand's buttons. What do you say, and what do you actually ship?

??? note "Show answer"

    Do not argue about whether the indicator is needed, because that argument concedes that it is a style question. Reframe it as a requirement with room for design: the indicator must exist, be visible against the actual background it sits on, and be visible in every state. What it looks like is entirely open.

    What to ship: a focus style that appears for keyboard interaction and not for mouse clicks, which removes the complaint in almost every case, since designers are usually reacting to outlines appearing on click. Then design the indicator properly: it can be a ring, an offset, a background shift or a border, and it can use brand colour, as long as it remains visible on every background it appears on, including the dark section of the page everyone forgets.

    The sentence that ends the discussion productively: "The indicator is not optional, but its appearance is entirely yours. Let us design one we both like, because the default is what we get if we do not."

**Q2.** Your automated accessibility rules pass on every page. What can you claim, and what can you not?

??? note "Show answer"

    You can claim that the machine-decidable tier is clean: no missing alternative text attributes, no unlabelled form controls, no duplicate ids, no invalid ARIA values, no insufficient contrast on solid backgrounds. That is real, it is worth having, and it prevents a large class of regressions cheaply.

    You cannot claim conformance, and you cannot claim usability. Nothing in that run tells you whether the alternative text says the right thing, whether the keyboard model matches the expected pattern for the widget, whether focus goes somewhere sensible after a deletion, whether announcements are comprehensible, or whether a process can be completed end to end without a mouse.

    The honest phrasing for a stakeholder: "Automated checks are a linter. They catch the mistakes that are decidable from the markup. The claim that the product is usable comes from the keyboard pass and the screen reader pass, and those are the ones that need people and time in the plan."

### 7g. When not to use this

| Thing | The measurement that justifies it | The threshold below which it is over-engineering | The cheaper alternative |
|---|---|---|---|
| Building a custom widget at all | The native element genuinely cannot do it, and you can name what it cannot do | Any case where a native select, button, dialog or details element covers the need. A custom widget is hundreds of lines that the platform already ships correctly | Use the native element and style what is stylable. Accept the parts that are not |
| ARIA attributes | You have built a widget the platform has no element for, and the interaction model already works | Adding roles to native elements. A role on a native button either does nothing or breaks it. Wrong ARIA is measurably worse than none, because it lies | Delete the attributes and use the correct element |
| A live region | A visual change carries information and no focus moves to it | Changes the user caused and can see the result of, such as a value they just typed. Announcing those doubles the speech for no gain | Nothing. Silence is correct when the change is already conveyed by focus or by the value |
| A focus trap | A true modal, where content behind must not be reachable | A non-modal popover or a dropdown. Trapping focus in a non-modal makes it impossible to leave, which is worse than the problem | Close on outside interaction and on Escape, and leave the tab order alone |
| A full manual audit per pull request | The change touches an interactive component or a new flow | A copy change or a colour token update. A full audit for every change trains the team to skip audits | The five-minute keyboard pass, and a full audit at flow level rather than per change |
| An accessibility statement claiming conformance | You have tested complete processes, with people, and can name the exceptions | An untested claim. A public conformance claim you cannot support is a legal and reputational exposure, not a marketing line | State what you have tested and what you know is outstanding. Specific and partial beats broad and unverified |

---

## Module 8. State, split three ways

**Time: 35 to 45 minutes reading, 25 to 35 minutes practice.**

### 8a. Predict first

A team's global client store has six slices. Read them before continuing.

| Slice | What it holds |
|---|---|
| S1 | Whether the sidebar is collapsed |
| S2 | The current user's profile, fetched from the server |
| S3 | The list of products for the current search |
| S4 | The theme, light or dark |
| S5 | Whether the "delete account" confirm dialog is open |
| S6 | The draft text of a comment being written |

??? note "Show answer"

    Prediction asked for: which of these do not belong in a global client store, and why?

    | Slice | Verdict | Reason |
    |---|---|---|
    | S1 | Belongs, or belongs in the URL | Distant components read it (the layout and the toggle button), so local state would need to be lifted anyway |
    | S2 | Does not belong | Something outside this tab owns the truth. Putting it in a client store means you now hand-write fetching, caching, staleness and invalidation that a server-cache layer does for free |
    | S3 | Does not belong, twice over | It is server data, and it is derived from the query, which is in the URL. Storing it globally creates two sources of truth for what the user searched for |
    | S4 | Belongs | Genuinely global client state that must also persist across sessions |
    | S5 | Does not belong | Only the dialog and its trigger care. Global dialog state is how two dialogs end up open at once |
    | S6 | Does not belong in a global store, but does need persistence | It is local to one composer, but it must survive a reload, which is a storage question rather than a scope question |

    The pattern in the wrong answers: **four of the six were put in a global store for four different reasons, and only one of them was about sharing.** S2 and S3 are there because the team had no server-cache layer, so the global store absorbed the job. That is the single most common state architecture mistake, and it is what this module fixes.

### The three kinds, and the test that sorts them

| | Client and UI state | Server cache state | Shared client state |
|---|---|---|---|
| Origin | Created here by the user or the interface | A copy of data owned by a server | Created here, read by distant parts |
| Who owns the truth | This tab | The server | This tab, or the device |
| What correctness means | It matches what the user did | It is fresh enough for the decision being made | All readers see the same value |
| How it becomes wrong | It does not, unless there is a bug | Time passes, or somebody else changes it | Two copies drift |
| What invalidates it | Unmount, navigation, reload | Time, a mutation you performed, an event from the server | Nothing; it is authoritative locally |
| Where it should live | Component state, or a state machine | A dedicated server-cache layer with keys and staleness rules | A small global store, or the URL, or persistent storage |
| Examples | Whether a menu is open, the active tab, a partially typed input | A product list, the current user, permissions | Theme, session, feature flags, locale |

**The sorting test**, three questions in order. The first that answers yes decides.

1. **Does anything outside this tab own the truth?** Then it is server cache state, and it needs a key and a staleness rule.
2. **Would two distant components disagree if this were local?** Then it is shared client state.
3. Otherwise it is local, and it should live in the component that owns the interaction.

There is a fourth question worth asking before all of them, because it removes work rather than assigning it: **should this survive a reload, be shareable as a link, and respond to the back button?** If yes, it belongs in the URL, and the URL then becomes the input to the server-cache key rather than a second copy of it.

### Server cache state, mechanically

This is the part that is easy to use through a library and hard to derive from scratch, so it is worth deriving once. Five mechanisms, each with arithmetic.

#### Cache keys

The key is the complete input to the query. Anything that changes the response and is not in the key is a bug waiting for a user to find.

- A list query taking a search term, a sort, a page number, a page size and a facet set has **five key components**. Omitting the facet set means two different filtered results share one entry, and the second reader sees the first reader's data.
- Keys must be **normalised** before comparison: object keys sorted, strings trimmed, numbers not compared to their string forms. Otherwise two logically identical requests miss the cache, and the hit rate quietly halves.
- **Do not put in the key** anything that does not change the response. Adding a timestamp or a render count to a key produces a cache with a hit rate of zero, which is a very expensive way to disable a feature you are still paying to maintain.

#### Deduplication

Multiple components asking for the same key within a short window should produce one request.

| Input | Value | Status |
|---|---|---|
| Avatar components on screen, all for the same user | 12 | ASSUMED, a comment thread |
| Round trip per request | 60 ms | ASSUMED, same figure as elsewhere on this page |
| Concurrent connections available | 6 | ASSUMED, a conservative per-origin limit |

Without deduplication: 12 requests over 6 connections is 2 rounds, so about **120 ms** and 12 origin hits. With deduplication: **one** request, 60 ms, one origin hit. The saving is small on latency and large on origin load, which is the honest way to argue for it.

#### Staleness and revalidation

Two separate timers, and conflating them is the usual bug.

| Timer | What it controls | What happens when it expires |
|---|---|---|
| Staleness window | How long a cached value is served without checking | The next read triggers a background revalidation, and the stale value is still shown |
| Retention window | How long an unused entry is kept in memory | The entry is discarded, so the next read shows a loading state instead of stale data |

Derive the staleness window from the data, not from a habit. If a value changes at most once an hour, a five-minute window bounds staleness at five minutes, which is well inside the change rate.

- ASSUMED: a 12-minute session in which 20 components mount and each would otherwise fetch on mount.
- Without a staleness window: 20 requests.
- With a five-minute window: at most **3** fetch events in 12 minutes, since 12 divided by 5 rounds up to 3.
- That is a factor of roughly 7 in origin load, bought with a bounded 5 minutes of staleness.

Revalidation triggers each carry a cost, so choose them per query rather than globally:

| Trigger | Costs | Worth it when |
|---|---|---|
| On mount | One request per mount, unless the staleness window suppresses it | Almost always, with a sensible window |
| On window focus | A burst when a user returns to the tab | Data that changes while the user is away and matters immediately, such as notifications |
| On reconnect | A burst across every active query at once | Nearly always correct, but stagger it or you have built a thundering herd against your own origin |
| On an interval | Continuous load proportional to open tabs | Only for data with a genuine freshness requirement, and prefer server push over a short interval |
| After a mutation | One request per affected key | Always, and this is the one to get right, since it is what makes the interface feel correct |

#### Normalisation

The same entity appearing in several query results is the source of the classic bug: liking a post updates it in the feed and not in the profile list, because they are two independent cached responses.

| Approach | What it costs | Adopt it when |
|---|---|---|
| Per-query caching, invalidate related keys after a mutation | You must know which keys are related, and that knowledge is easy to get wrong | The default. It is simple and it is correct as long as the key list is maintained |
| A normalised entity cache | A schema, an identity function per type, and a much larger mental model | The same entity appears in many independent queries and must stay consistent across all of them |

The threshold worth stating: if you can name the affected keys after a mutation in one line, per-query invalidation is sufficient. When that list becomes long enough that people get it wrong, normalisation starts to pay.

#### Optimistic updates and rollback

Three steps, in order, and the third is the one that gets skipped.

1. **Snapshot** the current cached value for every key you are about to change.
2. **Apply** the predicted result immediately, so the interface acknowledges within the 100 ms budget from Module 3.
3. **Reconcile**: on success, replace the prediction with the server's answer, because the server may have changed something you did not predict. On failure, restore the snapshot and tell the user.

Optimistic updates are only safe under three conditions, and saying them is what separates a considered answer from a habit:

- The result is **predictable** locally. A like toggles; a document merge does not.
- Failure is **rare**, so the interface is not routinely lying.
- Failure is **reversible without loss**. Rolling back a like is fine. Rolling back "message sent" after the user closed the composer is not.

### The three stores, drawn

```
+-------------------+   +--------------------+  +-------------+
| URL               |   | server cache       |  | local state |
| query, filters,   |-->| key = URL inputs   |  | open, hover |
| page, selected id |   | staleness window   |  | draft text  |
+-------------------+   +--------------------+  +-------------+
        |                     |         |
        v                     v         v
+-------------------+   +--------------------+
| shared client     |   | mutation           |
| theme, session,   |   | invalidates named  |
| feature flags     |   | keys, then refetch  |
+-------------------+   +--------------------+

Caption: the URL feeds the server cache key, so there is one
source of truth for what is being viewed. Mutations invalidate
keys rather than writing into the cache by hand, and local state
never crosses into either store.
```

### 8d. Worked example

The prompt: "Design the state for a filterable, paginated table of orders, where a row can be marked as refunded from a detail panel."

**Block 1. PURPOSE: sort every piece of state before designing any of it, because the sorting decides the architecture and takes ninety seconds.**

Running the test on each piece:

| State | Question that answered yes | Home |
|---|---|---|
| Search text, status filter, date range, sort, page | Should it survive a reload and be shareable? | The URL |
| The orders for the current view | Does something outside this tab own it? | Server cache, keyed by the URL inputs |
| The selected order's detail | Owned outside; also derived from an id in the URL | Server cache, keyed by order id |
| Whether the detail panel is open | Derivable from whether an id is in the URL | Nothing. Do not store it |
| Which rows are checked | Local, unless it must survive navigation | Component state |
| The refund confirm dialog | Only the panel cares | Component state |

The move worth noticing is the fourth row. Panel-open state is not stored at all, because it is a function of the URL. Deleting state is the cheapest correctness improvement available, and it is the move an answer reaches for last, because adding a store feels like design and removing one does not.

**The error most readers make here** is starting with the store and asking what to put in it. Start with the state and ask where each piece belongs, and the store ends up small.

!!! note "Say it before you read on"

    Putting the filters in the URL has a cost as well as a benefit. Name the cost, in one sentence about what happens on every keystroke in the search box.

**Block 2. PURPOSE: design the key before the fetching, because the key is what makes every later behaviour correct or wrong.**

The key for the list is the tuple of search text, status, date range, sort, page and page size. Six components, all from the URL. Normalisation rules I state explicitly: trim the search text, treat empty string and absent as the same value so they do not create two entries, and sort the status array so that two orderings of the same filter share one entry.

The key for the detail is just the order id, which means the detail is shared between the table and any other place that shows one order, at no extra cost.

**Block 3. PURPOSE: choose the staleness and revalidation policy per query, because a global policy is always wrong for at least one of them.**

- The list: orders arrive continuously, but a table being read does not need second-level freshness. ASSUMED staleness window of 30 seconds, revalidate on focus, no interval. Justification: a user returning to the tab wants current data, and a user staring at the table is not making decisions that turn on a 30 second difference.
- The detail: staleness of zero while the panel is open, because the user is about to act on it, and acting on a stale refund state is exactly the failure this design must avoid.

I say the asymmetry out loud, because it is the decision: reading is cheap and stale is fine, acting is expensive and stale is not.

**The error most readers make here** is one global staleness setting. It is either too aggressive for the list, so the origin carries pointless load, or too relaxed for the detail, so a refund is issued against stale state.

!!! note "Say it before you read on"

    Before the last block, say which of the two queries you would refetch on reconnect, and why refetching both at once could be a problem at scale.

**Block 4. PURPOSE: design the mutation, because refunding is where correctness, optimism and invalidation all meet.**

Refund is a case where I would **not** be optimistic, and saying why is the point: the result is not locally predictable (a refund can be declined by the payment provider), and the failure is not cheap to reverse in the user's mind, because showing "refunded" and then withdrawing it is worse than a two second wait.

So: a pending state on the button, a disabled control to prevent double submission, an idempotency key on the request so a retry cannot double-refund, and on success, invalidate two keys, the detail for that order and the list key. I would invalidate rather than write the new value into the cache by hand, because the server may have changed the order's status, its timeline and its totals, and hand-writing one field leaves the other two stale.

**Result.** The URL owns what is being viewed, the cache owns what the server said with a per-query staleness rule, local state owns two booleans, and the mutation invalidates rather than patches. Nothing in this design is a global store, which is the outcome the sorting test produces most of the time.

### 8e. Fade the scaffold

**Faded example 1.** Prompt: "Design the state for a chat window: a message list, a composer, and a read receipt sent when messages become visible." Blocks 1 to 3 are given. Produce block 4.

- Block 1: the conversation id is in the URL; the message list is server cache state with a push channel updating it; the composer draft is local but must survive a reload; scroll position is local and must not be stored globally.
- Block 2: the key is the conversation id plus a cursor, and the cache holds pages that must merge rather than replace, since a new page must not evict the previous one.
- Block 3: staleness is effectively infinite while the push channel is connected, and collapses to zero on reconnect, because the channel is the freshness mechanism and the cache is only correct while it is connected.

??? note "Show answer"

    Block 4 is the mutation, and sending a message is the textbook case where optimism is correct.

    "Sending is optimistic, and it meets all three conditions: the result is predictable, because the message is the one I just composed; failure is rare; and failure is reversible without loss, provided I keep the text. I insert the message with a client-generated id and a pending state, and on acknowledgement I replace it with the server's version, which carries the real id and the real timestamp."

    "Two details that decide whether this is right or subtly wrong. First, the client-generated id must be what the server echoes back, or I cannot match the acknowledgement to the optimistic message and it will appear twice, which is the most common chat bug in existence. Second, a failure keeps the message in the list marked as failed with a retry, rather than removing it and restoring the composer, because silently moving text back into the composer looks like data loss to the user."

    "The read receipt is a different shape: it is fire and forget, it is not user-visible if it fails, and it should be batched. Sending one request per message that scrolls into view turns a fast scroll into dozens of requests, so I would coalesce over a short window and send the highest message id seen, which is one request and is idempotent by construction."

**Faded example 2.** Prompt: "Our app keeps everything in one global store and the team says it is slow and hard to reason about. What do you change first?" Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: two complaints, two different causes. "Slow" is usually re-render scope. "Hard to reason about" is usually that server data lives in a client store with hand-written fetching.
- Block 2: I would measure before moving anything, and the measurement is which components re-render on an unrelated update, since a single store with a coarse subscription re-renders everything.

??? note "Show answer"

    Block 3, the sequencing. "I would not do a rewrite. The first change is to move server data out of the store and into a cache layer with keys and staleness, one query at a time, starting with the noisiest. That is incremental, reversible, and it deletes hand-written loading and error state from every component that used it, which is where most of the 'hard to reason about' comes from."

    "The second change is to delete derived state. Anything computed from other state should be computed, not stored, and stored derived state is where the inconsistencies live. This usually removes a surprising fraction of the store with no behaviour change."

    Block 4, the correct answer to the third request. "The third change is usually to add nothing. Once server data and derived state are gone, what remains in a typical application is a handful of genuinely global values: session, theme, flags, locale. That does not need the machinery it is currently sitting in, and the honest recommendation is often to keep the existing store for those few values rather than migrating them, because a migration with no user-visible benefit is hard to justify and easy to abandon halfway."

    "The measurement that tells me I am done is the one from the start: unrelated components no longer re-render on an update, and a new feature can be added without touching a global file. If either is still true after the first two changes, I have the wrong diagnosis and I would go back to measuring rather than continuing to move code."

### 8f. Check yourself

**Q1.** A colleague proposes caching every query for 24 hours to reduce origin load. What do you ask, and what is the risk?

??? note "Show answer"

    Ask one question per query: what is the longest a user can act on this data before it costs something? That converts a global setting into a per-query decision, which is the only form in which it can be right.

    The risk has two parts. The obvious one is stale data driving an action, such as a refund issued against an order status from yesterday. The less obvious one is that a long staleness window plus revalidation-on-focus produces a burst of requests every time users return to their tabs, so the origin load moves rather than disappearing, and it moves into a spike.

    The stronger proposal is to segment: long windows for data that genuinely changes rarely (a country list, a permissions set), short windows for anything the user acts on, and no caching at all for anything where a stale read is a correctness failure rather than a cosmetic one.

**Q2.** You have a list query cached under key A and a detail query under key B. A mutation changes the entity. Which do you invalidate, and what goes wrong if you patch instead?

??? note "Show answer"

    Invalidate both, and prefer invalidation to patching.

    Patching means writing the change into each cached entry by hand, and it goes wrong in three ways.

    - The server may change fields you did not predict, such as a status timeline, a total or an updated timestamp, so the patched entry becomes a mixture of real and guessed data with no way to tell which is which.
    - The list entry's position may change if the list is sorted or filtered by the field you altered, and patching in place leaves a row sitting in the wrong place.
    - Every new query that shows the same entity is a new place to remember to patch, so the correctness of the cache degrades with every feature added.

    The exception worth naming: patching is right during an optimistic update, because the whole point is to show a predicted value before the server answers. Even then, the reconciliation step replaces the patch with the server's response, so the patch is temporary by design rather than a permanent cache-management strategy.

### 8g. When not to use this

| Thing | The measurement that justifies it | The threshold below which it is over-engineering | The cheaper alternative |
|---|---|---|---|
| A dedicated server-cache layer | You have written loading, error and refetch logic by hand in more than roughly five components | An application with two or three data reads. The layer costs a dependency and a mental model to save code you have not written yet | Fetch in the component, and lift when a second component needs the same data |
| A global client store | Two or more distant components read the same non-server value, and lifting it would cross more than about three levels | Everything else. Most state is local, and most global stores are full of things that were never shared | Component state, lifted to the nearest common ancestor |
| A normalised entity cache | The same entity appears in several independent queries and you have shipped at least one consistency bug because of it | A handful of queries where you can name the affected keys after a mutation in one line | Per-query caching with an explicit invalidation list |
| Optimistic updates | An interaction that must acknowledge inside about 100 ms and whose result is locally predictable | Anything the server can decline, anything with side effects the user cannot see, anything rare enough that the latency is not a complaint | A pending state on the control. It is honest, it is two lines, and users accept it |
| Persisting cache to storage | Repeat visits are frequent and the data is expensive to fetch | A single-session tool. You have added a versioning and migration problem to save one request | Nothing. Fetch it again |
| Refetch on an interval | The data has a stated freshness requirement that no event can satisfy | Data that changes on user action only. An interval turns an idle tab into constant origin load | Refetch after the mutation that changes it, and on focus |

---

## Module 9. Data fetching and caching: HTTP semantics to browser storage

**Time: 35 to 45 minutes reading, 20 to 30 minutes practice.**

### 9a. Predict first

Three responses from the same application. Read the headers before continuing.

| Response | Cache-Control header |
|---|---|
| The HTML document at the site root | `no-cache` |
| `app.a91f3c.js`, a content-hashed bundle | `max-age=86400` |
| `/api/orders?page=1` | `max-age=300, public` |

??? note "Show answer"

    Prediction asked for: what happens on a reload, on a repeat visit after a deploy, and what is wrong with each.

    | Response | What actually happens | The problem |
    |---|---|---|
    | HTML with `no-cache` | Stored, but revalidated on every use, so every visit costs a round trip that usually returns 304 | This is correct, and it is the one people misread. `no-cache` means "check before using", not "do not store". `no-store` is the one that refuses to store |
    | Hashed bundle with `max-age=86400` | Cached for a day, then revalidated | Wasteful and pointless. The filename contains a content hash, so the bytes at this URL can never change. Any revalidation is a guaranteed 304, so the correct value is a year plus `immutable` |
    | API response with `max-age=300, public` | Cached for five minutes by the browser and by any shared cache, including a CDN | `public` on a per-user response is a data leak: a shared cache can serve one user's orders to another. This is the most serious of the three by a wide margin |

    Ranking those by severity is the point of the question. Row two wastes a round trip per asset per day. Row three can serve one customer's data to another, and it is a header, not a code change, which is how it escapes code review.

### The cache chain, in order

A request passes through several caches, and the order decides which header matters where.

```
+-----------+   +--------------+   +---------------+
| memory    |-->| service      |-->| HTTP cache    |
| cache in  |   | worker if    |   | on disk       |
| this tab  |   | registered   |   | shared tabs   |
+-----------+   +--------------+   +---------------+
                                          |
                                          v
                +--------------+   +---------------+
                | origin       |<--| CDN edge      |
                | server       |   | shared by all |
                +--------------+   +---------------+

Caption: the path of one request. The service worker sits in
front of the HTTP cache, so its code can override every header
below it, which is why a bad service worker is the stickiest
failure in frontend delivery.
```

Two consequences worth stating in a round:

- **A service worker overrides your headers.** If a worker serves from its own store, `no-store` on the response does not save you. The worker is code, and code beats configuration.
- **The CDN is shared and the browser cache is not.** Anything marked as cacheable by a shared cache must be identical for every user who could receive it, which is the rule row three of the pre-question broke.

### Response directives, and the bug each one causes when confused

The rules are specified in RFC 9111, the HTTP caching specification, at [rfc-editor.org/rfc/rfc9111](https://www.rfc-editor.org/rfc/rfc9111.html).

| Directive | What it actually means | The confusion, and the bug it causes |
|---|---|---|
| `max-age=N` | Fresh for N seconds in any cache | Confused with a guarantee. A cache may still evict it early; caching is a permission, not a promise |
| `s-maxage=N` | Fresh for N seconds in shared caches only, overriding max-age there | Omitted, so the CDN and the browser are forced onto the same lifetime, which they rarely want |
| `no-cache` | Store it, but revalidate before every use | Read as "do not store". It is the correct header for an entry document, and using `no-store` there instead throws away a cheap 304 path |
| `no-store` | Do not write it to any cache | Applied broadly out of caution, which disables revalidation as well and forces a full transfer every time |
| `private` | Only a browser cache may store it | Omitted on per-user responses, which permits a shared cache to store them. This is the leak |
| `must-revalidate` | Once stale, do not serve it; go and check | Assumed to be the default. Without it, a cache is permitted to serve stale content in some conditions |
| `immutable` | The bytes at this URL will never change, so do not revalidate even on reload | Applied to a URL without a content hash, which makes a stale asset unfixable except by changing the URL |
| `stale-while-revalidate=N` | Serve the stale copy for up to N seconds while refreshing in the background | Confused with max-age. It does not extend freshness; it removes the wait from the request that discovers staleness |
| `stale-if-error=N` | Serve a stale copy for up to N seconds if the origin errors | Rarely used, and it is one of the cheapest availability wins available to a static site |

### Validators, and what a 304 still costs

A validator lets a cache ask "has this changed?" and receive a small answer instead of the whole body. An entity tag is an opaque token for a version; a last-modified date is a coarser fallback with one-second resolution.

A 304 saves bytes. It does not save the round trip, and on a page with many assets the round trips are the cost.

| Input | Value | Status |
|---|---|---|
| Subresources on the page | 40 | ASSUMED, scripts, styles, fonts and images |
| Round trip | 60 ms | ASSUMED, consistent with the rest of this page |
| Concurrent connections | 6 | ASSUMED, a conservative per-origin limit |

- Revalidating all 40: 40 divided by 6 rounds up to 7 batches, so 7 times 60 ms is **420 ms** of pure "nothing has changed".
- With content hashes and `immutable`: **0 ms**, because no request is made at all.

That 420 ms is the entire argument for content hashing, and it is derivable in ten seconds on a whiteboard.

### Content hashing, and the two-tier rule

Every static delivery strategy reduces to two tiers.

| Tier | Example | Headers | Why |
|---|---|---|---|
| Content-addressed assets | `app.a91f3c.js`, `logo.4b2e91.svg` | `max-age=31536000, immutable` | The URL changes when the bytes change, so the old URL never needs to be corrected. 31536000 is 365 times 86400 |
| The pointer document | `index.html`, or the server-rendered entry | `no-cache`, or a short max-age with revalidation | It is the only thing that names the current asset URLs, so it must be checked. Caching it long is how users get a stale application indefinitely |

The consequence people miss: **with content hashing you almost never purge**. Purging exists for the pointer document and for API responses. If your deploy process includes purging your JavaScript, either you are not content hashing or you are purging out of superstition, and both are worth saying out loud.

One real hazard to name: a deploy that removes old asset files breaks any user still holding the old pointer document, because their HTML references URLs that no longer exist. Keep the previous few builds' assets available, and the length of that window is a number you should choose rather than inherit.

### CDN cache keys, and why Vary is dangerous

The cache key is what the CDN uses to decide whether two requests are the same request. By default it is roughly the method, the host and the path with its query string. Anything else you add multiplies the number of stored variants.

The hit rate model is simple enough to do out loud. For a URL receiving N requests per freshness window, with K distinct key variants, roughly K of those requests miss and the rest hit, so the hit rate is about 1 minus K over N.

| Scenario | K | N | Approximate hit rate |
|---|---|---|---|
| One variant | 1 | 300 | 99.7 percent |
| Split by three device classes | 3 | 300 | 99 percent |
| Split by locale, 12 locales | 12 | 300 | 96 percent |
| Vary on the full user agent string | effectively thousands | 300 | Near zero. Every request is a miss |

That is the whole argument against varying on a user agent string: the key space is unbounded, so the cache stops being a cache while continuing to cost money and add a hop. Varying on a small, normalised set is fine, and the discipline is to normalise at the edge into a few classes before the key is computed.

Purging strategies, compared:

| Strategy | Granularity | Use it for | Cost |
|---|---|---|---|
| Purge by URL | One object | A single corrected document | Requires knowing every affected URL, which is rarely true |
| Purge by tag or surrogate key | Everything sharing a tag | "All pages showing product 41", tagged at response time | You must attach tags on the way out, which is a small change with a large payoff |
| Purge by prefix | A path subtree | A section-wide content change | Blunt; easy to invalidate far more than intended |
| Soft purge | Marks stale rather than deleting | Almost always preferable | Requires stale-while-revalidate support, and gives you a cache that never goes cold at once |

Prefer soft purge. A hard purge of a popular key sends every waiting request to the origin at the same instant, which is the cache stampede described in the [failure library](index.md#the-failure-library) on the hub.

### Browser storage tiers

Six places to put bytes on a device, and choosing wrongly is a common source of both jank and data loss.

| Tier | Typical capacity | Persistence | Access cost | Sent to server | Good for | The trap |
|---|---|---|---|---|---|---|
| In-memory (a variable) | Bounded by device memory | Until reload | Free | No | Anything within one page view | Unbounded growth in a long-lived tab |
| Cookies | About 4 KB per cookie, per the cookie specification | Until expiry | Free to read | **Yes, on every matching request** | Session identifiers and nothing else | Every kilobyte is uploaded on every request |
| localStorage | A few megabytes in practice, per origin | Until cleared | **Synchronous, blocks the main thread** | No | Small, rarely-read preferences | A large read on the critical path is a long task, and it is string-only so it costs a parse too |
| sessionStorage | Same order as localStorage | Until the tab closes | Synchronous | No | Per-tab flow state, such as a wizard step | Same blocking cost, plus the surprise that duplicating a tab copies it |
| IndexedDB | Quota-based, typically a share of free disk | Until cleared or evicted under pressure | Asynchronous, structured | No | Offline data, large caches, message queues | An involved API, and the browser may evict it when disk is low unless persistence is requested |
| Cache Storage | Quota-based, same pool | Until cleared or evicted | Asynchronous, stores whole responses | No | Assets and responses managed by a service worker | It is separate from the HTTP cache, so it is a second copy with its own lifetime rules |

Two derivations that settle common arguments:

**Cookies are an upload tax.** Assume 4 KB of cookies on the origin, 60 requests during a page load, and an uplink of 1 Mbit/s, which is 125 KB/s. Then 4 KB times 60 is 240 KB uploaded, and 240 divided by 125 is about **1.9 seconds** of uplink time spread across the load, on a connection whose downlink you were already fighting for. That is the argument for keeping cookies to a session identifier and serving static assets from a cookie-free origin.

**localStorage is a main-thread cost.** It is synchronous, so a read blocks everything, including rendering the frame that is currently in flight. Reading a 2 MB JSON blob from it during startup is both a synchronous disk read and a parse, on the thread that is trying to paint. IndexedDB does the same work off the main thread, which is the whole reason to prefer it for anything larger than a preference.

### 9d. Worked example

The prompt: "Write the caching plan for our application: the HTML entry, the hashed bundles, user avatars, and a list endpoint."

**Block 1. PURPOSE: classify each resource by who it varies for, because that single property decides every header that follows.**

| Resource | Varies by | Therefore |
|---|---|---|
| HTML entry | Nobody, but it points at the current build | Shared-cacheable, but must be revalidated |
| Hashed bundles | Nobody, ever, by construction | Shared-cacheable forever |
| Avatars | The user depicted, not the user requesting | Shared-cacheable, and content-addressable if the URL includes a version |
| List endpoint | The requesting user | Private, or not cached at all |

Avatars are the interesting row and the one people get wrong in both directions. The image of user 12 is the same bytes for everyone who views it, so it is shared-cacheable. What is private is which avatars a given user sees, and that is a property of the page, not of the image.

**The error most readers make here** is classifying by "is it user data" rather than by "does the response body vary by requester". Those are different questions and only the second one decides the headers.

!!! note "Say it before you read on"

    Say what must be true about an avatar URL before you can mark it immutable, and what breaks if it is not true.

**Block 2. PURPOSE: assign headers, and justify each number rather than copying a convention.**

- HTML entry: `no-cache`. Every visit costs one round trip, assumed 60 ms, and usually returns a 304. I accept that 60 ms because it is what guarantees a user gets the current build immediately after a deploy, and because it is the pointer for everything else.
- Hashed bundles: `max-age=31536000, immutable`, which is 365 times 86400 seconds. The `immutable` part is what removes revalidation even on an explicit reload, and it is what turns the 420 ms of 304s calculated earlier into zero.
- Avatars with a version in the path: `max-age=31536000, immutable, public`. Public is correct here and would be a leak on the list endpoint, and being able to say why is the point.
- List endpoint: `private, no-store` if the data is sensitive, or `private, max-age=30` if a 30 second window is acceptable and I can name what a user could do wrong with 30 second old data. I would ask, not assume.

**Block 3. PURPOSE: design the deploy behaviour, because a caching plan that has not thought about deploys is half a plan.**

- Assets from the previous builds stay available for a stated window. ASSUMED at seven days, on the reasoning that a tab left open over a weekend plus a day of slack is the realistic worst case, and the storage cost of a few extra builds is negligible next to a broken session.
- The entry document is not purged, because `no-cache` already revalidates it. Nothing else needs purging, since everything else is content addressed.
- A running client that fails to load a hashed chunk after a deploy needs a recovery path, because that failure is real if the window expires: detect the load failure and reload the page once, which fetches a fresh entry document and therefore fresh URLs.

**The error most readers make here** is stopping at headers. The deploy interaction is where caching plans fail in production, and it is invisible in a header table.

!!! note "Say it before you read on"

    Before the last block, say why "reload the page once" needs the word "once", and what happens without it.

**Block 4. PURPOSE: decide the storage tiers, because the same plan has to say where anything kept on the device lives.**

- Session identifier in a cookie, marked as HTTP-only and same-site, and nothing else in cookies. Justification is the 240 KB upload derivation: every extra kilobyte in a cookie is paid on every request.
- User preferences such as theme in localStorage, because they are tiny, needed synchronously before first paint to avoid a flash, and losing them is harmless.
- Anything larger, such as a cached list for offline reading, in IndexedDB, because the read must not block the main thread.
- No copy of the API responses in Cache Storage unless there is a service worker with a stated offline requirement, because a second cache with its own lifetime is a consistency problem I would rather not own.

**Result.** Four resource classes, four header sets each justified by a property of the resource, a deploy window with a stated reason, a recovery path for the failure that window does not cover, and a storage assignment derived from two costs rather than from habit.

### 9e. Fade the scaffold

**Faded example 1.** Prompt: "Our CDN hit rate is 40 percent and we do not know why." Blocks 1 to 3 are given. Produce block 4.

- Block 1: a low hit rate is a key problem, a freshness problem or a traffic problem, and they are distinguishable. Key problems show as many variants of one URL; freshness problems show as short lifetimes; traffic problems show as low request counts per URL, where nothing is wrong at all.
- Block 2: the request logs show one URL with thousands of key variants, which points at a key that includes something it should not.
- Block 3: the responses carry a `Vary` on a header that differs per browser build, so the key space is effectively unbounded and the hit-rate model gives approximately zero for that URL.

??? note "Show answer"

    Block 4, the fix and the cost of getting it wrong in the other direction.

    "The fix is to normalise before the key is computed: classify the incoming header into a small set at the edge, two or three classes at most, and vary on the class rather than the raw string. With three classes and an assumed 300 requests per URL per freshness window, the model gives about 99 percent instead of near zero."

    "The cost of overcorrecting is worse than the current state, so I would name it explicitly. If I remove the variance entirely and the response genuinely differs by that dimension, the CDN will serve one class's response to another class. That is the same class of bug as marking a per-user response public, and it is silent. So the sequence is: prove the response actually differs, define the classes, verify that responses within a class are byte-identical, and only then change the key."

    "The measurement that tells me I am done is the hit rate on that URL specifically, not the site-wide average, because a site-wide average hides exactly this problem behind a large number of already-healthy URLs."

**Faded example 2.** Prompt: "A user reports seeing an old version of the application even after refreshing. Diagnose." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: refreshing normally revalidates the entry document, so if a refresh does not fix it, something is serving the old entry document or overriding the entry document entirely.
- Block 2: the candidates are an entry document with a long max-age, an `immutable` directive on a non-hashed URL, a CDN that has cached the entry document with a long shared lifetime, or a service worker serving the application shell from its own store.

??? note "Show answer"

    Block 3, the elimination. "These are distinguishable with two observations. First, check the response headers on the entry document from a fresh request: a long max-age or `immutable` there is the answer, and it is a configuration fix. Second, check whether a service worker is registered for the origin: if it is, its code is in front of every header below it, and no header change will help until the worker is updated."

    "The order matters, because the service worker case is the one that cannot be fixed by shipping a new deploy. A worker that serves the shell from its store will keep doing so until it is updated or unregistered, and its update check may itself be cached. That is why a bad worker is the stickiest failure in delivery."

    Block 4, the fix and the prevention. "For the header case: `no-cache` on the entry document, and `immutable` only on URLs that contain a content hash. For the worker case: ship a worker update, and make sure the worker's own script is served with a short lifetime so the update is discoverable at all."

    "The prevention worth building is a kill switch, decided before it is needed. A worker should check a small, uncacheable configuration value on activation and unregister itself if told to. Without that, the only remedy for a broken worker is asking every affected user to clear site data, which for a consumer product is not a remedy at all."

### 9f. Check yourself

**Q1.** Explain the difference between `no-cache` and `no-store` to a colleague, and say when each is right.

??? note "Show answer"

    `no-cache` permits storage and requires revalidation before every use. The cached copy is kept, and the cache asks the origin whether it is still valid, so a 304 costs one round trip and no body transfer. This is the correct default for an entry document: the user always gets the current version, at the cost of one small round trip.

    `no-store` forbids storage anywhere. Every request is a full transfer, and there is no validator exchange because there is nothing stored to validate. It is correct for genuinely sensitive responses where a copy on disk is itself the risk, such as a banking statement on a shared machine.

    The practical failure is applying `no-store` where `no-cache` was meant, out of an instinct that both mean "always fresh". They both give freshness; only one of them also gives you cheap 304s. On a large document that difference is a full body transfer on every visit, and it is invisible in every functional test, because the page is correct and simply slower.

**Q2.** You add a `Vary: Accept-Language` to responses for a site with 12 locales. What happens to the CDN, and what should you do instead if the hit rate matters?

??? note "Show answer"

    The key space for each URL multiplies by the number of distinct header values received, and browsers send full preference lists rather than a single locale, so the practical variant count is far above 12. Using the model above with 300 requests per URL per freshness window, even a clean 12 variants gives about 96 percent, while an unbounded set of preference strings can approach zero.

    Two better designs, and both make the locale part of the identity of the resource instead of a hidden dimension of the key:

    - **Put the locale in the URL**, as a path segment or a subdomain. Each locale becomes its own resource with its own clean cache entry, links are shareable and unambiguous, and there is no `Vary` at all.
    - **Normalise at the edge**, mapping the incoming preference list to one of your 12 supported locales before the key is computed, and vary on that normalised value.

    The first is preferable when the locale is user-visible and should be linkable, which is nearly always true for content sites. The second is preferable when the locale must be inferred and the URL must stay stable, which is more common for an application shell.

### 9g. When not to use this

| Thing | The measurement that justifies it | The threshold below which it is over-engineering | The cheaper alternative |
|---|---|---|---|
| A CDN at all | Users are geographically spread, or origin egress and load are measurable costs | A single-region internal tool with a few hundred users. You have added a cache layer, a purge process and a debugging surface for a latency saving nobody notices | Serve from the origin with correct cache headers, which is most of the benefit for one-region traffic |
| Content hashing plus immutable | You have more than a handful of static assets and repeat visitors | A page with three assets that change weekly. A one-day max-age is fine and needs no build tooling | A modest max-age and a query-string version on the few assets you have |
| Custom CDN cache keys | You have proven the response genuinely differs by the dimension, and you have normalised it into a few classes | Any case where you have not measured the hit rate per URL. Adding key dimensions on suspicion is how a 99 percent hit rate becomes 40 percent | One key, and move the varying dimension into the URL instead |
| Tag-based purging | Content changes must appear faster than your shortest safe max-age | Content that changes on deploy. Your deploy already changes the hashed URLs, so there is nothing to purge | Shorter max-age on the few documents that change, plus stale-while-revalidate |
| A service worker | You have a stated offline requirement, or a measured repeat-visit cost that HTTP caching cannot address | Wanting faster repeat visits. HTTP caching already does that, and a worker adds the stickiest failure mode in frontend delivery | Correct cache headers, which are configuration rather than code, and cannot strand a user |
| IndexedDB | You are storing more than a few hundred kilobytes, or reads are on a path where blocking would be visible | A theme preference and a dismissed-banner flag. The API cost is real and the data is tiny | localStorage, read once at startup |
| Storing anything in cookies beyond a session id | The value must be read by the server on every request | Anything the client alone reads. From the derivation above, 4 KB across 60 requests is about 1.9 seconds of uplink on a 1 Mbit/s connection | localStorage or IndexedDB, which cost zero bytes on the wire |

---

## Module 10. Network and real-time

**Time: 32 to 42 minutes reading, 20 to 30 minutes practice.**

Real-time answers fail in a recognisable way. The transport is named in the first sentence, and everything that decides whether the screen actually works is skipped. Direction, message rate and what the interface does while the connection is gone are what choose a transport. The transport itself decides very little.

### 10a. Predict first

Read this specification and commit to an answer before continuing.

| Property | Value |
|---|---|
| Direction of messages | Server to client, apart from one subscribe on open |
| Instruments streamed | 500 |
| Updates per instrument | About 10 a second |
| Rows visible at once | 40 |
| Current design | One WebSocket, every message written to state on arrival |

**Question.** Name the number that eliminates this design before you reach the transport at all, and say what breaks first on the device.

??? note "Show answer"

    Two numbers, both derivable from the table above.

    **Message rate.** 500 instruments times 10 updates a second is 5,000 messages a second. At 60 frames a second a frame is 16.7 ms, so 5,000 divided by 60 is about 83 messages arriving inside every frame. If each one writes state and schedules a render, the loop is being asked for 5,000 renders a second and will deliver at most 60. The main thread saturates, and rising input latency is what the user notices first.

    **Bytes.** Assume 200 bytes per message on the wire, which is a small JSON object with an instrument id, a price and a timestamp. 5,000 times 200 bytes is 1 MB a second. Against the 200 KB per second profile this page uses elsewhere, that is five times the entire downlink, so the design fails at the requirements step rather than at the transport step.

    **What is actually wrong.** 40 rows are visible out of 500 instruments, so 92 percent of the messages change nothing the user can see. The fixes in order of value: subscribe to what is visible, conflate on the server, coalesce per frame on the client. Swapping WebSocket for something else fixes none of the three.

    The transport question that does matter here is smaller than it looks. The direction is one way, so a bidirectional channel is buying nothing and charging you connection state you have to shed and rebalance.

### Four transports, compared on what actually differs

The primary sources, so you can check any of this rather than take it from a table: WebSocket is [RFC 6455](https://www.rfc-editor.org/rfc/rfc6455.html), published December 2011, which is also where the ping and pong control frames are defined. Server-sent events are specified in the [HTML standard](https://html.spec.whatwg.org/multipage/server-sent-events.html), which defines `EventSource`, the reconnection behaviour, the `retry` field and the `Last-Event-ID` request header. WebTransport is a [W3C Candidate Recommendation Snapshot dated 30 July 2026](https://www.w3.org/TR/webtransport/), which describes multiple streams and unreliable datagrams.

| Transport | Direction | Reconnect and resume | Ordering | What it costs you |
|---|---|---|---|---|
| Long polling | Down, on a request the client keeps reissuing | Nothing to reconnect, because every message is a fresh request | Per response, and a gap exists between responses that the server must buffer across | A request per batch, plus latency of up to one round trip on every message |
| Server-sent events | Down only | In the client already: automatic retry, plus `Last-Event-ID` so the server can replay from where you stopped | Ordered within one stream | Text only, so binary payloads need encoding. One connection per tab, which matters more on HTTP/1.1 than on HTTP/2 |
| WebSocket | Both | Nothing. Reconnection, resume tokens and heartbeats are all yours to write | Ordered within one connection, and nothing is ordered across a reconnect | Connection state on the server, a handshake, and every liveness mechanism written by hand |
| WebTransport | Both, over HTTP/3 | Yours to write, as with WebSocket | Ordered within a stream, unordered across streams, and datagrams may be lost or reordered | The newest of the four, so an answer that picks it owes a fallback path and a reason the fallback is acceptable |

The row that changes most answers is the second one. Server-sent events give you the two things teams most often get wrong when they hand-roll them, automatic reconnection and a resume point, and they give them away for free. A WebSocket gives you neither and is chosen anyway, because "live" and "bidirectional" have become the same word.

### The transport decision, drawn

```
+------------------------------------------+
| Does the client send messages up too,    |
| on the same channel and at message rate? |
+------+---------------------------+-------+
       | no                        | yes
       v                           v
+----------------+     +------------------------+
| updates faster |     | WebSocket. You now own |
| than one every |     | reconnect, heartbeat,  |
| ten seconds?   |     | resume and shedding    |
+---+--------+---+     +------------------------+
    | yes    | no
    v        v
+---------+ +------------------+
| server- | | poll on a timer  |
| sent    | | and stop calling |
| events  | | it real time     |
+---------+ +------------------+

Caption: the transport routing question, top to bottom. The first
fork is direction, the second is rate. Long polling is where the
left leaf goes when server-sent events cannot be used at all, and
WebTransport is the right leaf when you need unreliable datagrams
and can afford to carry a fallback for clients without it.
```

Two clarifications the diagram deliberately leaves out. A client that sends one message on open, or one message per user action, is not sending "at message rate", so it stays on the left branch and posts its writes over ordinary HTTP. And a screen with several live topics does not need several connections: multiplex topics over one stream and give each topic its own sequence.

### Reconnection: backoff, jitter, and why jitter is the load-bearing half

Backoff without jitter is a synchronised retry. Every client that dropped at the same instant comes back at the same instant, which is how a recovering server is knocked over by its own users.

| Attempt | Ceiling from doubling a 1 s base | Delay actually used, with full jitter |
|---|---|---|
| 1 | 1 s | Uniform random in 0 to 1 s |
| 2 | 2 s | Uniform random in 0 to 2 s |
| 3 | 4 s | Uniform random in 0 to 4 s |
| 4 | 8 s | Uniform random in 0 to 8 s |
| 5 | 16 s | Uniform random in 0 to 16 s |
| 6 and after | 30 s, the cap | Uniform random in 0 to 30 s |

The arithmetic that justifies the cap, with the inputs stated. ASSUME 200,000 connected clients, the same population the market data problem in the interleaved set uses, and ASSUME a restart drops all of them at once.

Retrying on a fixed one second delay puts 200,000 attempts into one second. Spreading the same attempts uniformly across a 30 second cap gives about 6,700 a second, a factor of 30 lower, and the price is that the unluckiest client waits 30 seconds. Pick the cap from what a user will tolerate staring at a stale screen, then check the arrival rate it implies.

Not every disconnect deserves a retry, and treating them alike is a real bug.

| Why the connection ended | Correct response |
|---|---|
| Transport error, or the device reports itself offline | Retry with backoff and jitter. Reset the attempt counter after a connection survives a stated interval, not on connect, or a flapping link resets it forever |
| Server closed with a normal close code | Retry with backoff. This is a deploy or a rebalance, and it is the case jitter exists for |
| Authentication or authorisation rejected | Do not retry the socket. Refresh the credential first, and if that fails, stop and say so on screen |
| Server closed with a policy or protocol error | Stop. Retrying a message the server refuses to accept is an infinite loop with a network bill |

One interface rule worth stating out loud, because it is cheap and almost always missing: do not show a "reconnecting" banner on the first failure. A one second blip that recovers on attempt one should be invisible. Show the banner from the second failed attempt, and show a stale-data timestamp rather than a spinner, because a spinner claims progress you cannot promise.

### Heartbeats and liveness

A closed connection is easy. A dead connection that nobody has closed is the problem, and it is common: a phone that loses radio sends no close frame, and an intermediary can drop an idle connection without telling either end. The socket stays open in your code and delivers nothing forever.

So liveness is an application concern, with two numbers.

- **Send interval.** ASSUMED at 20 s here. Choose it below the shortest idle timeout on your own path, measured rather than inherited, because that timeout belongs to whatever proxies and gateways your traffic actually crosses.
- **Liveness deadline.** Two missed beats plus a margin, so about 45 s at a 20 s interval. Below two missed beats you will tear down healthy connections on ordinary jitter.

The cost is a wakeup, not bytes. At a 20 s interval a client sends 3 beats a minute, so 180 an hour. At 5 s it sends 720 an hour, four times the radio wakeups to detect death four times sooner. On a phone that is a battery decision, so state the trade rather than picking 5 seconds because it feels responsive.

Two details that separate a designed answer from a remembered one. First, heartbeats must run in both directions: a server that only listens cannot reclaim the socket of a client that vanished, and connection slots are the resource that runs out. Second, server-sent events reconnect automatically but do not detect a silently dead connection, so a stream that has delivered nothing past your deadline should be closed and reopened by your own code.

### Ordering and deduplication on the client

A single connection preserves the order of what it carries. That guarantee ends at the edges, and every edge is where the bugs live.

| Where order breaks | Why | The mechanism that fixes it |
|---|---|---|
| Across a reconnect | The new connection knows nothing about the old one | A per-topic sequence number, and a resume request carrying the last applied value |
| Between the snapshot fetch and the live stream | Two independent responses race | Subscribe first, buffer arriving messages, then fetch the snapshot and drop buffered messages at or below its sequence |
| Between an optimistic local write and the server copy | The local item exists before the server has an id for it | A client-generated id sent with the write and echoed back, so the local item is replaced rather than joined by a twin |
| Across several streams or datagrams | Independent streams are independently ordered by design | Order inside a topic only, and never assume two topics are comparable |

Three rules that make the receive path boring, which is the goal.

- **Detect gaps, do not paper over them.** If an arriving sequence is greater than the last applied plus one, messages were missed. Refetch the snapshot and resubscribe. Applying the message anyway leaves a client that is silently wrong, which is worse than a visible reload.
- **Deduplicate against a bounded set.** Keep seen message ids in a fixed-size ring, sized from the rate and the resume window. At an assumed 60 chat messages an hour and a 30 minute session, 30 ids covers the session, so a 200 entry ring is generous and cannot grow. An unbounded seen-set is a memory leak that only appears in long sessions, which is exactly where nobody tests.
- **Prefer idempotent application.** "Set price to 41.20 at version 9" survives a duplicate. "Increment unread by one" does not. Where the payload is a delta by nature, carry a version and ignore anything at or below the version already applied.

### The receive path, drawn

```
+----------+   +-----------+   +-----------+   +-----------+
| socket   |-->| drop ids  |-->| sequence  |-->| coalesce  |
| message  |   | already   |   | check,    |   | by key    |
| arrives  |   | seen      |   | gap alarm |   | into a    |
|          |   |           |   |           |   | buffer    |
+----------+   +-----------+   +-----------+   +-----+-----+
                                                     |
                                     once per frame  v
                                          +---------------------+
                                          | apply to state and  |
                                          | render visible rows |
                                          +---------------------+

Caption: one message from the socket to the screen. Deduplication
comes before the sequence check so a replayed message after a
resume does not raise a false gap alarm. Everything to the left of
the buffer runs per message; everything to the right runs at most
once per frame, which is what stops the message rate from setting
the render rate.
```

### Coalescing, batching, and the difference

Coalescing drops superseded messages. Batching keeps all of them and applies them together. Choosing the wrong one either loses data or fails to reduce anything.

| What the message means | Can a later one replace it? | Strategy |
|---|---|---|
| A value at a point in time, such as a price or a courier position | Yes | Coalesce by key, keep the last, flush once per frame |
| A membership or presence state | Yes | Coalesce by key, and accept that intermediate states are never shown |
| An event in a log, a chat message, a notification | No | Batch, apply the whole batch in one write, never drop |
| A delta that depends on the previous value | No | Batch, and apply in sequence order or not at all |

The saving is worth computing rather than asserting. From the predict-first numbers, 5,000 messages a second coalesced per frame at 60 frames a second is at most 60 state writes a second instead of 5,000, which is about 83 times fewer renders. Coalescing by key also collapses the 40 visible instruments to at most 40 changed values a frame, so the render is bounded by what is on screen rather than by what the market is doing.

One frame is the upper bound on flush rate, not the right answer everywhere. A number a person reads rather than watches move can flush on a longer interval, say once per 100 ms, which is another factor of six and is often more legible because the digits stop flickering. Pick the interval from what the value is for, then say which one you picked and why.

### Backpressure when the server outruns the render loop

Backpressure means deciding, deliberately, what happens when input arrives faster than you can consume it. Doing nothing is also a decision, and the thing it decides is that the buffer grows until the tab dies.

Four levers, cheapest first.

1. **Filter at the source.** Subscribe to the visible set and resubscribe on scroll. In the predict-first problem this is 40 of 500 instruments, so it removes 92 percent of the traffic and the bytes with it. It is the cheapest lever and the one most often skipped, because it needs a server that accepts a subscription list.
2. **Conflate on the server.** Send at most one update per key per interval, say one per 100 ms. This is the only lever that saves the bytes as well as the work, and from the arithmetic above the bytes were the binding constraint, so on that problem it is not optional.
3. **Coalesce per frame on the client.** No server change needed, so it is available immediately. It saves renders but still pays the download, the parse and the deduplication for every message.
4. **Shed.** If the pending buffer exceeds a stated bound, stop applying, refetch a snapshot and resubscribe. Shedding is what keeps a slow client correct rather than merely behind.

The measurement that tells you which lever you need is the depth of the pending buffer sampled over a window. A depth that returns to zero every frame is healthy. A depth that trends upward over five seconds means the client is losing, and long task count and interaction latency will confirm it a moment later. If parsing is the cost rather than rendering, move the socket and the parse into a worker and post only the coalesced result to the main thread.

### 10d. Worked example

The prompt: "Design the live-updating part of an order tracking screen: the courier's position on a map, the order state, and a chat thread with the courier."

**Block 1. PURPOSE: separate the streams by direction and by what a lost message costs, because those two properties decide everything downstream.**

| Stream | Direction | Rate | Cost of a lost message | Cost of a duplicate |
|---|---|---|---|---|
| Courier position | Down | About 1 a second while moving | Nothing. The next one supersedes it | Nothing |
| Order state | Down | A handful per order | High. The user waits on a state that never arrives | Nothing, if it carries a version |
| Chat | Both | Bursty, zero for minutes then several in seconds | Unacceptable | Visible and embarrassing |

**The error most readers make here** is picking one transport because the three streams share one screen. They share a screen and nothing else. The position stream can lose messages all day, and the chat send path cannot lose one ever, which is a durability requirement rather than a transport requirement.

!!! note "Say it before you read on"

    Say which of the three streams justifies a bidirectional channel, and then say whether that justification survives the sentence "the send must work when the tab is closed".

**Block 2. PURPOSE: choose transports, and state what each choice costs rather than only what it buys.**

- All three receive paths ride one server-sent events stream carrying three topics, each with its own sequence. Reconnection and the resume point are already in the client, and the direction is one way. Cost: text only, so any binary payload is encoded, and one connection per tab.
- Chat sends are ordinary HTTP requests carrying a client-generated idempotency key, not socket frames. A send has to survive the tab closing, and a socket cannot promise that. Cost: a request per message instead of a frame, which at this rate is nothing.
- What this rejects, out loud: a WebSocket for everything. It buys bidirectionality that only chat wants, and chat is better served by a request that can be queued, retried and deduplicated by a key the server already needs.

**Block 3. PURPOSE: design the receive path, because ordering and duplication are settled here or not at all.**

- Each topic carries a monotonic sequence. On reconnect the client resumes from the last applied sequence per topic. A gap triggers a snapshot refetch for that topic only, not for the screen.
- Deduplication by message id against a 200 entry ring per topic, sized from an assumed 60 chat messages an hour against a 30 minute session, which is about 30 ids.
- Position messages coalesce by key and flush once per frame, so a burst after a tunnel paints one marker move rather than forty.
- Order state applies by version, and any version at or below the applied one is ignored. This is what makes a replay after resume harmless.

**The error most readers make here** is applying messages the instant they arrive because the code is shorter. It is shorter until the first reconnect, at which point the difference between a resume and a replay becomes a user-visible duplicate.

!!! note "Say it before you read on"

    Before the last block, say what the screen should show when the stream has been down for 40 seconds, and why a spinner is the wrong answer.

**Block 4. PURPOSE: design the failure behaviour, because that is the part of a live screen a user actually experiences.**

- Reconnect with full jitter, doubling from a 1 s base to a 30 s cap, with the attempt counter reset only after a connection has survived 60 s. Authentication failures do not retry the stream, they refresh the credential once and then surface an error.
- Heartbeat every 20 s in both directions, connection declared dead at 45 s. Assumptions as derived above: two missed beats plus a margin.
- Chat sends queue in IndexedDB, so they survive a closed tab. Each entry holds the idempotency key, the payload, an attempt count and a created time. Entries older than a stated window surface to the user as failed rather than retrying forever.
- Degradation, stated before it is asked for: if the stream cannot be established at all, poll order state every 15 s, and replace the live map marker with the last known position plus the time it was recorded. A stale position labelled stale is useful. A stale position pretending to be live is a support ticket.

**Result.** Three streams separated by direction and loss cost, one transport chosen per direction with its cost stated, a receive path with sequences, gap detection and bounded deduplication, a send path that survives the tab, and a degradation mode that is honest about staleness. Nothing here required a WebSocket, and being able to say why is the point.

### 10e. Fade the scaffold

**Faded example 1.** Prompt: "Our live dashboard freezes for about two seconds every time the market opens." Blocks 1 to 3 are given. Produce block 4.

- Block 1: a freeze is main-thread occupancy, so the candidates are parse cost, state write cost, render cost or a layout loop, and they are distinguishable in one profile.
- Block 2: the profile shows thousands of short tasks rather than one long one, each a message parse followed by a state write followed by a render.
- Block 3: the rate at open is measured at roughly 4,000 messages a second against a screen showing 30 rows, so almost every message changes nothing visible.

??? note "Show answer"

    Block 4, the fix in the order that costs least, and the measurement for each step.

    "First, coalesce on the client, because it needs no server change and it is available this week. Buffer arriving messages into a map keyed by instrument and flush once per animation frame. At 60 frames a second that is at most 60 state writes a second instead of 4,000, and the freeze should become a busy but responsive screen. The measurement is long task count during the first ten seconds of open, before and after."

    "Second, filter at the source, because 30 rows are visible out of the full set and the rest are being paid for in bytes, parse and deduplication. Resubscribe on scroll with a small overscan so a fast scroll does not blank. The measurement is bytes received during open, which should fall roughly in proportion to the fraction of instruments dropped."

    "Third, conflate on the server, one update per instrument per 100 ms. This is the only step that also removes the download, so I would ask for it even though it is the slowest to land. The measurement is again bytes at open, and I would expect it to bound the rate at ten a second per instrument regardless of how the market behaves."

    "What I would not do first is change the transport. Nothing in the evidence points at the transport, and swapping it would cost weeks and move no number in the profile. If the client is still behind after all three steps, the next move is parsing in a worker, and the signal for that is a profile where parse dominates the coalesced write."

**Faded example 2.** Prompt: "Users report seeing their own chat messages twice after driving through a tunnel." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: a duplicate after a connectivity gap is either a resend that the server accepted twice, or a single server-side message that the client rendered alongside its own optimistic copy.
- Block 2: the server log shows two stored messages with different ids and identical bodies, seconds apart, both from the same device.

??? note "Show answer"

    Block 3, the elimination. "Two stored messages with different ids means this is not a client rendering bug, it is a duplicated write. The device sent the message, lost the radio before the response arrived, treated the failure as unsent, and sent it again on reconnect. The server had already stored the first one and had no way to know the second was the same intent."

    "The distinguishing observation is the ambiguous failure. A request that fails after the server committed is indistinguishable at the client from one that failed before, so a retry is correct behaviour and duplication is the consequence. That means the fix cannot live in the retry logic. It has to live in the identity of the write."

    Block 4, the fix and what it also fixes. "Generate an idempotency key on the client when the message is composed, not when it is sent, and store it with the queued item. The server treats that key as unique per conversation and returns the existing message on a repeat rather than storing a second one. Retries then become free, which is what lets the outbound queue be aggressive."

    "The same key solves the optimistic rendering problem, which is the other way this bug appears. The local item is keyed by the client id, and the server echoes that id back, so the arriving copy replaces the local one instead of appearing beside it. Without the echo, matching is done on body and timestamp, which fails the moment someone sends the same word twice."

    "The prevention is a rule rather than a patch: every write that a client may retry carries a client-generated identity, and every list that merges local and remote items matches on that identity. Stated once in the design, it removes a whole class of report."

### 10f. Check yourself

**Q1.** Your reconnect logic doubles from one second with no upper bound and no randomness. Describe both failure modes, and say what you would change.

??? note "Show answer"

    The first failure is synchronisation. Clients that dropped together retry together, so a server that has just come back receives its entire population in one instant. With an assumed 200,000 connected clients, a fixed one second delay is 200,000 attempts in a second, which is likely to knock the server down again and produce a second synchronised wave.

    Full jitter, choosing a delay uniformly between zero and the current ceiling, spreads the same attempts across the window. At a 30 second cap that is about 6,700 a second, thirty times lower.

    The second failure is the missing cap. Unbounded doubling reaches ten minutes by attempt sixteen, so a user whose connection recovered minutes ago sits in front of a stale screen because the client is still asleep. The cap should come from how long a user will tolerate stale data, and then be checked against the arrival rate it implies.

    A third detail worth naming because it is usually wrong: reset the attempt counter after a connection has been healthy for a stated interval, not on connect. A flapping link connects constantly, and resetting on connect turns your backoff into a fixed one second retry exactly when the network is worst.

**Q2.** After a reconnect, a client shows one message twice. Name the two independent mechanisms that both had to be absent, and say which one you would add first.

??? note "Show answer"

    Mechanism one is resume rather than replay. If the client reconnects and the server sends everything from the start of the session, the client receives messages it already applied. A per-topic sequence, sent on reconnect and honoured by the server, means the stream restarts at the right point. Server-sent events give this shape for free through the `Last-Event-ID` header, and a hand-rolled WebSocket has to be told to do it.

    Mechanism two is deduplication at apply time. Even with resume, a replay window is normal: the server may send a small overlap deliberately, because an overlap is safe and a gap is not. So the client keeps a bounded set of applied ids and drops anything it has already seen.

    Add deduplication first. It is local, it needs no protocol change, and it makes the client correct against any server behaviour including a server that deliberately overlaps. Resume is the larger win because it also removes the wasted transfer, but it needs both ends to agree, and a client that is correct only when the server behaves is not correct.

    The trap to name: the deduplication set must be bounded. An unbounded set is invisible in every test and a slow leak in the long sessions where live features are actually used.

### 10g. When not to use this

| Thing | The measurement that justifies it | The threshold below which it is over-engineering | The cheaper alternative |
|---|---|---|---|
| A persistent connection at all | The update rate, times the staleness the product actually requires, makes polling either too slow or too chatty | Data that changes a few times an hour, or a screen a user looks at for ten seconds. A poll on a 30 s timer is two lines and cannot strand anyone | Poll on a timer, and refetch on the tab becoming visible again |
| A WebSocket rather than server-sent events | The client genuinely sends at message rate, not one write per user action | Any one-way feed. You have taken on reconnection, resume and heartbeats to gain a direction you do not use | Server-sent events, which ship reconnection and a resume header already |
| WebTransport | You need unreliable datagrams or independent streams, and you can name what you gain by dropping ordering | Anything that a single ordered stream serves. Two transports to maintain is the real cost, not the API | WebSocket, or server-sent events if the direction allows |
| An offline outbound queue in storage | Writes must survive a closed tab, and you can point at the user journey where that happens | A desktop tool on an office network. An in-memory retry with a visible failed state is honest and far smaller | Retry in memory, and show the send as failed with a retry control |
| Per-frame coalescing | A profile shows state writes or renders exceeding one per frame during a normal burst | Under roughly one message a second. A coalescer adds a buffer and a flush path for no measurable saving | Apply on arrival, and revisit when the profile says otherwise |
| Client-side gap detection and resume | Missing a message leaves the client silently wrong rather than briefly stale | A feed where the next message supersedes the last, such as a position or a price. Resubscribing and refetching is simpler and always correct | Refetch a snapshot on reconnect and discard the question |
| An application heartbeat | You have seen connections that deliver nothing and never close, or your path crosses intermediaries you do not control | A short-lived connection inside one page view, where a dead connection is indistinguishable from the page ending | The transport's own close handling, plus a refetch when the tab becomes visible |

---

## Module 11. Performance

**Time: 45 to 55 minutes reading, 35 to 45 minutes practice.**

Performance answers fail in a specific way: the candidate lists metrics, then
lists optimisations, and never connects one to the other. The connective tissue
is that each metric blames a different part of the page, and each optimisation
moves exactly one of them.

### 11a. Predict first

Read this artefact and commit to an answer before continuing.

```
+---------------------------------------------------------------+
| Product page. Two measurements taken the same week.           |
|                                                               |
| LAB, one run, desktop, wired, cold cache:                     |
|   TTFB 180 ms   FCP 0.9 s   LCP 1.2 s   TBT 40 ms   CLS 0.00  |
|                                                               |
| FIELD, 28 days, all users:                                    |
|   LCP p50 1.4 s   LCP p75 4.1 s   LCP p95 9.6 s               |
|   INP p75 480 ms                  CLS p75 0.19                |
|                                                               |
| Team's conclusion: "the lab says we are fast, field data is   |
| noisy, we will keep optimising LCP."                          |
+---------------------------------------------------------------+
Caption: one lab run and one field distribution for the same
page, with the team's reading of them.
```

**Question.** Name what the p50 to p75 gap tells you that the lab run cannot,
and say which metric the team should work on first and why it is not LCP.

??? note "Show answer"

    The p50 at 1.4 s is close to the lab's 1.2 s, so the lab run is not wrong. It
    is a measurement of one point in the distribution: a fast device on a fast
    network with a warm origin. It answers "is the page capable of being fast",
    which is a real but narrow question.

    The p50 to p75 jump from 1.4 to 4.1 s is a 2.9x step across one quartile.
    That shape is a population split, not noise: some identifiable group of
    sessions is doing something structurally different. Candidates are device
    class, network class, cold versus warm cache, a specific route variant, a
    logged in versus logged out template, or a third party script that only loads
    for some users. The next action is to segment, not to optimise.

    The metric to work on first is INP at 480 ms. Two reasons. First, LCP p75 at
    4.1 s and INP p75 at 480 ms are both poor, but INP measures the response to a
    deliberate user action, so every one of those 480 ms events is a user who
    tapped and waited. Second, the CLS p75 of 0.19 is a strong hint at the same
    root cause as the INP number: late arriving content that both shifts layout
    and occupies the main thread.

    The general rule the artefact is teaching: the lab tells you whether a fix
    worked. The field tells you what to fix. Using either for the other job wastes
    a quarter.

### 11b. The metric set, and what each one blames

Define the terms as you introduce them. All of these are measured from
navigation start unless stated.

| Metric | What it measures | What it blames | Typical single cause |
|---|---|---|---|
| TTFB, time to first byte | Request start to first byte of the document | Network, redirects, server render, CDN miss | A redirect chain, or an uncached server rendered page |
| FCP, first contentful paint | To the first text or image painted | Render blocking resources in the head | A synchronous stylesheet or blocking font |
| LCP, largest contentful paint | To the paint of the largest above the fold element | Discovery and priority of one specific element | A hero image loaded by JavaScript rather than in the markup |
| CLS, cumulative layout shift | Unexpected movement of visible content | Unreserved space | An image without dimensions, or an injected banner |
| INP, interaction to next paint | Worst (near worst) input to paint across the visit | Main thread occupancy and handler cost | A long task from hydration or a third party |
| TBT, total blocking time | Lab only: main thread time in tasks over 50 ms | Same as INP, before any user exists | A large JavaScript bundle evaluating |

**The blame column is the point.** If LCP is bad, do not "optimise the page".
Find the LCP element, then ask three questions in order: was it discoverable in
the initial HTML, was it given a high enough priority, and was it large in bytes.
Those three cover the large majority of LCP problems and they are answered by
inspection, not by a tool.

**TBT is a lab proxy for INP, not the same metric.** TBT is measured with no user
present, so it can only see potential blocking. A page with a 900 ms hydration
task has terrible TBT and may have fine INP if the user does not interact during
hydration. Use TBT as the CI gate, because it is deterministic. Use INP as the
truth, because it has users in it.

### 11c. Lab versus field, stated as a division of labour

| Property | Lab | Field |
|---|---|---|
| Population | One device, one network, your choice | Real distribution of devices, networks, caches |
| Determinism | High, with the variance controls below | Low, it is a distribution |
| Answers | "Did this change help?" | "What should I change?" |
| Blind to | Everything about your actual users | Anything you have not shipped yet |
| Cost | Minutes per run | Weeks to accumulate a stable quartile |

**Making a lab run deterministic enough to gate on.** A single lab run has
variance measured in tens of percent. Three controls, in order of value:

- Run N times and take the median. N = 5 is the usual compromise: the median of
  5 is much more stable than a single run and costs 5 minutes rather than 20.
- Pin the CPU throttle and the network profile explicitly, and record them in the
  result, so a result from a faster CI machine is not compared with an older one.
- Gate on two consecutive failing builds rather than one, which turns a 10
  percent per run false alarm rate into about 1 percent.

### Percentiles, and how much data a percentile needs

Averages hide the population you care about. Concrete distribution:

- 70 percent of sessions at 700 ms, 30 percent at 4,000 ms
- Mean = 0.7 x 700 + 0.3 x 4,000 = 490 + 1,200 = 1,690 ms
- p75 = 4,000 ms

The mean reads as a mildly slow page. The p75 reads as a page where three users
in ten wait four seconds. Both numbers are correct. Only one describes a user.

**How many samples does a p75 need?** This is derivable and almost nobody
derives it, so it is a strong thing to be able to do at a whiteboard.

The count of samples above the 75th percentile is binomial with n trials and
p = 0.25.

- Standard deviation of that count = sqrt(n x 0.25 x 0.75) = sqrt(0.1875 n)
- n = 1,000: sd = sqrt(187.5) = 13.7 samples
- Two standard deviations is 27 samples out of 1,000, which is 2.7 percentile
  points. So your "p75" is really somewhere between the p72.3 and the p77.7.

To convert that into milliseconds you need the local shape of your own
distribution. If your measured values between p72 and p78 span 2,200 to 2,500
ms, then the 2.7 point wobble is about 135 ms of noise on a p75 near 2,350 ms,
which is roughly 6 percent.

**The operational consequence.** A dashboard cell with 1,000 samples cannot
detect a 5 percent regression. Either accumulate more samples per cell, or
compare over longer windows, or accept that you are watching for large moves
only. Say which one you chose.

### Code splitting, priced

Two assumptions, both stated because everything below depends on them.

- **Effective throughput 1.6 Mbps = 200 KB/s.** This is a planning figure for a
  congested mobile connection. It is reasonable because it sits well below a
  clean 4G link and well above a genuinely bad one, so it will not flatter you.
- **JavaScript main thread cost of roughly 1 ms per KB of uncompressed source on
  a mid tier phone**, covering parse, compile and initial evaluation. Compressed
  JavaScript expands by roughly 3.3x, so 1 KB compressed is about 3.3 ms.
  Measure your own with a long task recording before quoting it.

Now price a bundle.

| Quantity | Value | Derivation |
|---|---|---|
| Main bundle, compressed | 420 KB | GIVEN by the build |
| Transfer time | 2.1 s | 420 / 200 |
| Uncompressed | 1,386 KB | 420 x 3.3 |
| Main thread time | 1.39 s | 1,386 x 1 ms |
| Total before interactive | 3.5 s | 2.1 + 1.39 |

Now split by route. The checkout route needs 160 KB of the 420 KB.

- Saved bytes: 260 KB
- Saved transfer: 260 / 200 = 1.3 s
- Saved main thread: 260 x 3.3 = 858 ms
- Total saved: about 2.2 s, and roughly 40 percent of that saving is CPU, not
  network

That CPU share is the part people forget. On a fast network the same split still
saves 858 ms, which is why "our users are on wifi" is not a reason to skip it.

**Route splitting versus component splitting.** They are not the same technique
at different granularity, they have different failure modes.

| | Route split | Component split |
|---|---|---|
| Trigger | Navigation, which the user expects to take time | Interaction or viewport, which the user expects to be instant |
| Waterfall risk | Low, the route chunk is requested as soon as the route is known | High, the chunk cannot be requested until the parent has executed |
| Break even size | Almost always worth it | Only above roughly 20 to 30 KB compressed |
| Mitigation | Prefetch on link hover or viewport | Preload the chunk at idle after first paint |

**The component split break even, derived.** A dynamic import costs one
additional round trip that cannot start until its parent chunk runs. Assume a
150 ms round trip on mobile, which is an ordinary figure for a warm connection
to a CDN edge.

- A 20 KB compressed chunk transfers in 20 / 200 = 100 ms
- It costs 150 ms of round trip that the parent bundle would not have paid
- Net: you spent 150 ms to save 100 ms of transfer plus 66 ms of CPU
  (20 x 3.3), so it is roughly break even at 20 KB and clearly positive above
  about 40 KB

Below that, splitting a component is a loss dressed as an optimisation. This is
the number to quote when someone proposes lazy loading every modal in the app.

### A budget you can actually enforce

A performance budget fails in CI for one of three reasons: it is measured on the
wrong thing, it is flaky, or nobody can get an exception so they disable it.
Design against all three.

```
+---------------------+     +----------------------+
| BUILD               | ==> | BYTE BUDGET          |
| bundler stats file  |     | per route entry      |
| per route entry     |     | compressed bytes     |
+---------------------+     +----------+-----------+
                                       |
+---------------------+                |  fail or pass
| LAB RUN x5          | ==> +----------v-----------+
| fixed CPU throttle  |     | TIMING BUDGET        |
| fixed network       |     | median TBT, median   |
| median of 5         |     | LCP, 2 builds in a   |
+---------------------+     | row before failing   |
                            +----------+-----------+
                                       |
                            +----------v-----------+
                            | EXCEPTION PATH       |
                            | labelled PR, expiry  |
                            | date, owner          |
                            +----------------------+
Caption: two gates with different determinism, and the escape
hatch that keeps the gate from being deleted.
```

**What to put a number on.**

| Budget | Value type | Why this one |
|---|---|---|
| Compressed bytes per route entry graph | Hard fail | Deterministic, attributable to a diff, and it is the input to both transfer and CPU cost |
| Count of separate blocking requests in the head | Hard fail | A cheap proxy for render blocking waterfalls |
| Median lab TBT over 5 runs | Fail on two consecutive builds | Catches "same bytes, more work" regressions that bytes miss |
| Third party bytes, tracked separately | Hard fail, separate owner | Otherwise a marketing tag consumes the application team's headroom silently |
| Field p75 INP and LCP | Alert, never a build gate | Field data lags the deploy by days, so it cannot block a build |

**Set the initial number from where you are, not from a target.** Take the
current value, add 5 percent headroom, and freeze it. A budget set at an
aspirational number fails on day one and is disabled by day three. Ratchet it
down when a real improvement lands, in the same pull request that lands the
improvement.

### 11d. Worked example

**The prompt.** "Our product detail page felt fine last quarter. Field INP p75
went from 210 ms to 480 ms over six weeks with no single obvious release.
Diagnose and design the fix."

Inputs, stated before I touch anything:

| Input | Value | Status |
|---|---|---|
| Page views per day | 900,000 | GIVEN |
| Routes with meaningful traffic | 12 | GIVEN |
| RUM sampling rate | 100 percent for vitals | GIVEN |
| INP p75 now, was | 480 ms, 210 ms | GIVEN |
| Mid tier phone CPU cost per KB of uncompressed JavaScript | about 1 ms | ASSUMED, stated in the code splitting section, verify with one long task recording before quoting |

#### Block A. Purpose: let the metric name the suspect before touching code

INP moved, LCP did not. That single fact eliminates a large space of causes. INP
is input to next paint, so it blames one of three things: the event handler
itself, the main thread being occupied when the input arrives, or the rendering
work triggered by the handler.

I ask for the attribution breakdown that a modern RUM library records with each
INP sample:

| Component | Meaning | What it would implicate |
|---|---|---|
| Input delay | Time from input to handler start | Main thread was busy with someone else's work |
| Processing time | Handler execution | Our own handler got expensive |
| Presentation delay | Handler end to next paint | Rendering or layout cost, or a large re-render |

I do not guess between them. I say that the next thirty seconds of the interview
depend on which of the three grew, and I ask.

**The error most readers make here.** Jumping to "we should code split". Code
splitting reduces startup work. If the regression is in presentation delay from
an over broad re-render, splitting changes nothing and you have spent a sprint.

!!! tip "Self-explanation prompt"

    Before reading Block B, say which of the three attribution components would
    most likely also have degraded LCP, and use that to reason about why LCP
    holding steady is informative.

#### Block B. Purpose: split the population instead of chasing the average

Suppose attribution says input delay grew from 60 ms to 300 ms. Main thread
contention. Now I segment before optimising, because "the main thread is busy"
has a dozen owners.

I cut the INP p75 by four dimensions, and I check each cell has enough samples to
be readable. From 11d, a cell needs roughly 1,000 samples for a p75 with about
2.7 percentile points of wobble.

- 900,000 views per day
- Cut by 12 routes and 3 device classes = 36 cells
- 900,000 / 36 = 25,000 per cell per day, which is 25x the minimum. Comfortable
- Adding a fourth cut of 4 network classes gives 144 cells and 6,250 each, still
  fine
- Adding a fifth cut of 20 countries gives 2,880 cells and 312 each, which is
  below the readable threshold. I stop at four dimensions and say why

This arithmetic is the difference between a segmentation plan and a wish. It also
tells me the honest limit of what my data can answer.

**The error most readers make here.** Slicing until something looks bad. With
enough thin cells, something always looks bad, and it is noise. Decide the
readable cell size first, then slice.

#### Block C. Purpose: attribute the cost to a specific line of the dependency graph

The segmentation says the regression is on all routes, all devices, worst on the
slowest device class. All routes means shared code, not route code.

I compare the shared bundle across the six weeks:

| Week | Shared bundle compressed | Delta |
|---|---|---|
| Week 0 | 310 KB | baseline |
| Week 6 | 402 KB | +92 KB |

92 KB compressed is about 304 KB uncompressed, about 304 ms of main thread on
the assumed mid tier phone. That is in the right order of magnitude for a 240 ms
input delay regression, so the hypothesis survives arithmetic.

Then I find where the 92 KB went, using the build's module attribution rather
than intuition:

| Source of growth | Compressed | Note |
|---|---|---|
| A date library pulled in transitively by an analytics wrapper | 38 KB | Nobody imported it deliberately |
| A charting library imported at module scope for one rarely used panel | 34 KB | Static import, so it is in the shared graph |
| Genuine feature code | 20 KB | Keep it |

**The error most readers make here.** Reading total bundle size and stopping.
The actionable unit is the module and its importer. "Our bundle grew" is not a
finding. "An analytics wrapper transitively imports a date library" is.

#### Block D. Purpose: choose the cuts that pay, and price each one

Three candidate fixes, priced against the 1.6 Mbps and 1 ms per KB assumptions
stated in the code splitting section.

| Fix | Bytes removed | Transfer saved | CPU saved | Cost to do it |
|---|---|---|---|---|
| Replace the date library with the platform Intl APIs | 38 KB | 190 ms | 125 ms | Two days, plus a formatting audit |
| Convert the chart panel to a dynamic import | 34 KB | 170 ms | 112 ms | Half a day, and it is well above the 20 KB break even |
| Split the 20 KB of feature code by route | 20 KB | 100 ms | 66 ms | Two days, and it is at the break even, so I decline |

I take the first two, which is 72 KB, about 360 ms of transfer and about 238 ms
of CPU on the slow device class. The predicted input delay improvement is in the
same range as the observed regression, which is the check that I am fixing the
thing I diagnosed.

I decline the third explicitly and say why, because declining a plausible
optimisation with a number is a stronger signal than accepting all three.

!!! tip "Self-explanation prompt"

    Say why the chart panel being a static import at module scope mattered more
    than its size, and what you would have concluded if it had been 8 KB instead
    of 34 KB.

**The error most readers make here.** Treating byte savings and CPU savings as
one number. They land on different users. On a fast network with a slow phone
the CPU saving is the entire benefit, and that population is large.

#### Block E. Purpose: make the fix permanent with a gate that will not be flaky

The regression took six weeks and no single release, which means it was 15
merges each adding a few kilobytes. A one time fix will be undone by the same
process. So the deliverable is a gate, not a diff.

| Gate | Threshold | Determinism handling |
|---|---|---|
| Shared bundle compressed bytes | Current value after the fix, 330 KB, plus 5 percent = 347 KB | Fully deterministic from the stats file, hard fail |
| Route entry bytes, per route | Current plus 5 percent each | Deterministic, hard fail |
| Lab TBT on the product detail page | Median of 5 runs, fixed CPU throttle | Fail only on two consecutive builds, which turns a roughly 10 percent per run false alarm rate into roughly 1 percent |
| Third party bytes | Separate budget, separate owner in the code owners file | Hard fail, and it routes to the team that added the tag |

Plus one non gate: a weekly report of the largest module additions by compressed
bytes, sent to the team, because the failure mode here was invisibility rather
than negligence.

**The error most readers make here.** Gating on field INP. Field data arrives
days after the deploy and is a distribution, so it cannot fail a build. Field
data alerts. Lab data gates.

### 11e. Fade the scaffold

**Fade 1: an internal analytics dashboard.** 4,000 daily users, all on company
issued laptops on office wifi, one route, heavy client side charting over tables
of up to 50,000 rows. Complaint is "filtering feels laggy". Same five block
shape, last block blank.

- Block A, let the metric name the suspect: INP again, but the attribution here
  is likely presentation delay rather than input delay, because the handler
  triggers a re-render of a large table. Confirm before acting.
- Block B, split the population: with 4,000 users on one route there are no
  meaningful segments and, by the percentile arithmetic above, a p75 on 4,000
  samples per day has about
  sqrt(0.1875 x 4000) = 27 samples of standard deviation, so 2 sd is 54 out of
  4,000, only 1.4 percentile points. The data is readable in aggregate but there
  is nothing to slice. Segment by row count in the current view instead, which is
  the variable that actually differs.
- Block C, attribute the cost: not bytes. On fast laptops and fast wifi a 400 KB
  bundle costs about 0.13 s of transfer and, at roughly a third of mid tier phone
  cost, about 0.44 s of CPU once. The recurring cost is per interaction: 50,000
  rows re-rendered on every keystroke of the filter input.
- Block D, choose the fixes and price them: virtualise the table so only the
  roughly 40 visible rows render, which turns a 50,000 element render into 40.
  Debounce the filter input at about 150 ms so a 10 character query does 1 render
  rather than 10. Move the filter computation off the main thread only if
  profiling shows the computation, not the render, dominates.
- Block E, make it permanent: **you write this block.**

??? note "Show answer"

    The byte budget from the worked example is close to worthless here, and
    saying so is the point of this fade. Bytes are not the constraint on office
    wifi with laptops. Gating on bytes would give a green build while filtering
    stayed laggy.

    The gate that matches the diagnosis is an interaction cost gate. Build a
    deterministic lab scenario: load a fixture with 50,000 rows, type a 10
    character query with fixed timing, measure total main thread time in tasks
    over 50 ms during the interaction window. That is TBT scoped to an
    interaction rather than to page load.

    Threshold: current post fix value plus 5 percent, median of 5 runs, fail on
    two consecutive builds. Same determinism machinery, completely different
    measurement.

    Add a second gate that is cheap and catches the specific regression class:
    assert that the number of rendered row elements after filtering stays under
    about 100. It is a structural assertion rather than a timing one, so it never
    flakes, and it fails the moment someone removes virtualisation.

    The transferable lesson: the budget must measure the thing that was slow. A
    performance budget copied from a public web page to an internal tool
    typically measures the wrong axis entirely.

**Fade 2: a marketing landing page.** Server rendered, mostly static, one hero
image, a video embed, three third party tags (consent, analytics, a chat
widget). Field LCP p75 is 4.8 s, CLS p75 is 0.31, INP p75 is 140 ms. Last two
blocks blank.

- Block A, let the metric name the suspect: LCP and CLS are both bad while INP is
  fine, which is the opposite pattern to the worked example. That combination
  points at content arriving late and shifting layout, not at main thread
  occupancy.
- Block B, split the population: check whether LCP p75 differs between users who
  accepted and declined the consent dialog, because the dialog gates the tags and
  therefore creates two structurally different pages.
- Block C, attribute the cost: **you write this block.**
- Block D, choose the fixes and price them: **you write this block.**

??? note "Show answer"

    Block C. For LCP, the first question is which element is the LCP element, and
    on a page like this it is almost always the hero image. Then the three
    questions in order from 11b: is it discoverable in the initial HTML, is it
    prioritised, and is it large.

    Common finding: the hero is set as a CSS background image or injected by a
    script, so it is not discoverable by the preload scanner and its request does
    not start until the stylesheet or script has been fetched and evaluated. That
    single fact can add a full round trip plus stylesheet download to LCP, on the
    order of 1 to 2 s on a mobile connection.

    For CLS, attribute per shifting element rather than per page. The three
    candidates are: the hero without width and height attributes so the browser
    cannot reserve space, the consent banner injected after first paint, and the
    chat widget appearing at a fixed position that pushes content. Each has a
    different fix, and a single CLS number tells you none of this.

    The tool answer is that both attributions come from the browser's own
    performance entries: the LCP entry names the element, and each layout shift
    entry names the sources that moved. Recording those in RUM is what makes
    field data actionable rather than merely alarming.

    Block D. Priced fixes, and the ordering matters because two of them are
    nearly free.

    First, put the hero in the markup as an image element with explicit width and
    height and a high fetch priority. Cost: hours. Effect: removes the discovery
    round trip from LCP, and removes the hero's contribution to CLS entirely
    because space is reserved. This is the single largest item and it is not an
    architecture change.

    Second, reserve space for the consent banner and the chat widget with a
    fixed height container rendered in the initial HTML. Cost: hours. Effect:
    removes their CLS contribution. Note it does not make them faster, and that
    is fine, because CLS is about unreserved space, not about speed.

    Third, the video embed. A third party video embed typically ships hundreds of
    kilobytes of player JavaScript before the user has expressed any intent to
    watch. Replace it with a poster image plus a click to load, which defers the
    entire player. Price it: if the embed is 300 KB compressed, deferring saves
    1.5 s of transfer at 200 KB/s and about 1 s of CPU at the stated assumption,
    on every visit, for the large majority of visitors who never press play.

    Fourth, and stated as a decision rather than a fix: the chat widget belongs to
    another team and has its own byte budget with its own owner. I would not
    absorb its cost into the page team's budget. That is an organisational move,
    and on a marketing page it is usually the one that actually holds.

### 11f. Check yourself

**Q1.** A colleague reports that a change improved lab LCP from 2.4 s to 1.6 s,
measured once before and once after. Field LCP p75 is unchanged after two weeks.
Give two distinct explanations and say how you would tell them apart.

??? note "Show answer"

    Explanation one: the lab result is noise. A single run has variance in the
    tens of percent, so a 0.8 s move on one measurement each side is within the
    range a page can produce with no change at all. Test it by rerunning both
    versions five times each and comparing medians.

    Explanation two: the improvement is real but applies to a part of the
    population that does not move the p75. If the change helps only users with a
    warm cache, or only fast devices, it improves the p25 and p50 while leaving
    the slow tail that sets the p75 untouched. Test it by looking at the field
    p25 and p50 as well as the p75, and by segmenting on cache state and device
    class.

    A third possibility worth naming: the field metric has not accumulated enough
    post change data. A 28 day trailing window two weeks after a change is still
    half old data, so the observed p75 is a blend. Check the window definition
    before concluding anything.

    The distinguishing move overall is to look at more of the distribution. A
    single percentile cannot separate "no effect" from "effect on a group this
    percentile does not describe".

**Q2.** Your team proposes lazy loading every dialog in the application, of which
there are 30, averaging 12 KB compressed each. Evaluate with numbers.

??? note "Show answer"

    Total bytes involved: 30 x 12 = 360 KB compressed, which sounds like a large
    win. It is not, for two reasons.

    First, the break even derived earlier. A dynamic import costs a round trip,
    assume 150 ms, and it cannot start until the parent chunk has executed. A
    12 KB chunk transfers in 12 / 200 = 60 ms and costs about 40 ms of CPU
    (12 x 3.3).

    So each split spends 150 ms of latency at the moment the user opens the
    dialog in order to save 100 ms at startup. Every one of these 30 splits is a
    net loss at the point of interaction, which is exactly where INP is measured.

    Second, the savings are not additive in the way the proposal assumes. If a
    typical session opens 3 dialogs, the user still downloads 36 KB, now in 3
    separate round trips at the worst possible moment, and the startup saving is
    324 KB rather than 360 KB.

    The better version: group the 30 dialogs into two or three chunks by
    likelihood of use in a session, giving chunks of roughly 100 to 150 KB that
    are well above break even, and prefetch the most likely chunk at idle after
    first paint. That captures most of the startup saving with one round trip
    that is already complete before the user clicks.

    The answer to give, in one line: the right granularity for a split is set by
    the round trip cost, not by the module boundary.

### 11g. When not to use this

**Component level code splitting.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The chunk is above roughly 30 KB compressed, AND it is not needed by a typical session, AND you have somewhere to prefetch it that is not on the interaction path |
| Scale threshold below which it is over-engineering | Below about 20 KB compressed the round trip cost exceeds the transfer plus CPU saving, so the split makes the interaction slower while making a dashboard number better |
| Cheaper alternative | Group rarely used modules into one or two lazily loaded chunks, prefetched at idle. One round trip instead of many, and the same startup saving |

**A field RUM pipeline with attribution.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You have at least roughly 1,000 samples per day in the cells you intend to read, so a p75 is stable to a few percentile points, AND you have someone who will act on a segment level finding |
| Scale threshold | Below a few thousand daily sessions, per segment percentiles are noise and you are building a dashboard nobody can act on. Under one route and one device class, a lab run answers the same questions |
| Cheaper alternative | A scheduled lab run on your three most important routes, median of 5, charted over time. It will not tell you about your users, and it will catch regressions |

**A performance budget enforced in continuous integration.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You can point to a regression that arrived incrementally over multiple merges rather than in one identifiable change, which is the failure mode a gate catches and code review does not |
| Scale threshold | With one or two engineers and a weekly release, a chart someone looks at is enough. A gate has an ongoing maintenance cost in false alarms and exception handling |
| Cheaper alternative | Record bundle bytes per build into a chart and post the weekly delta to the team channel. Visibility fixes most incremental drift, and it never blocks a release at 5 pm on a Friday |

**Virtualisation of a list.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Rendered element count is high enough to show in an interaction profile, typically above roughly 1,000 elements, AND the interaction cost is in rendering rather than in data fetching |
| Scale threshold | Under a few hundred rows, virtualisation costs you correct find in page behaviour, correct scrollbar sizing, simple keyboard navigation and accessible row indexing, in exchange for milliseconds |
| Cheaper alternative | Paginate, or cap the visible set with a "show more" control. Both keep the platform behaviours that virtualisation breaks and both are a fraction of the code |

---

## Module 12. Design systems

**Time: 30 to 40 minutes reading, 20 to 30 minutes practice.**

A design system interview question is rarely "build a button". It is "40
engineers across 6 apps need to ship a rebrand in one quarter". That is a
versioning, distribution and governance problem wearing a visual costume.

### 12a. Predict first

Read this artefact and commit to an answer before continuing.

```
+---------------------------------------------------------------+
| Component library, 30 components, shipped 18 months ago.      |
| Inside Button.css, colours written as raw hex:                |
|   background 1E5AA8   text FFFFFF   border 17457E             |
| Inside Card.css:                                              |
|   background FFFFFF   border D7DBE0                           |
| 27 of the 30 components contain literal hex values.           |
|                                                               |
| New requirement: ship a dark theme and a high contrast theme. |
+---------------------------------------------------------------+
Caption: a component library with colour values written directly
into component styles, and the requirement that breaks it.
```

**Question.** Count the edits the naive fix requires, then say why the usual
advice ("extract the colours into variables") is only half the fix.

??? note "Show answer"

    The naive count: 27 components with literal values, and each needs its values
    overridden per theme. If each component averages 4 colour declarations, that
    is 27 x 4 = 108 declarations, times 2 new themes, so 216 new declarations
    spread across 27 files.

    Extracting to variables fixes the mechanics and not the problem. Suppose you
    replace every hex with a variable named after the colour: `--blue-600`,
    `--grey-200`. Now the dark theme has to redefine `--blue-600` as something
    that is not blue 600, or every component that used it needs a different
    variable. The first is a lie in the name, the second is the same 27 file edit
    you were avoiding.

    The missing half is a naming layer that describes role rather than
    appearance. A button referencing `--action-background` and a card referencing
    `--surface` can both be re-themed by editing one file of role definitions,
    because the theme is now expressed in the vocabulary the theme actually
    varies.

    The total number of values does not shrink much. What changes is where they
    live: 45 role definitions per theme in one reviewable file, versus 108
    declarations scattered across 27 components. Concentration, not reduction, is
    the win, and saying that honestly is stronger than claiming a saving that is
    not there.

### 12b. Token architecture in three layers

Define each layer by who is allowed to reference it.

| Layer | Example names | Referenced by | Changes when |
|---|---|---|---|
| Primitive | `blue-600`, `grey-200`, `space-4`, `radius-2` | Only the semantic layer | The brand palette itself changes, rarely |
| Semantic | `surface`, `surface-raised`, `text-primary`, `border-subtle`, `action-bg`, `danger-text` | Components, and application code | A theme is added or a role is redefined |
| Component | `button-bg-hover`, `input-border-invalid` | Only that one component | That component's design changes |

**The single rule that makes the architecture work.** A component may never
reference a primitive. If it does, the primitive's name has become part of the
component's contract, and the theme layer can no longer intervene.

Enforce it mechanically: a lint rule that fails on any primitive token name
appearing outside the semantic token file. Without enforcement this rule decays
in about two quarters, because referencing `grey-200` directly is always the
fastest way to close one ticket.

**Counting a realistic system.**

| Layer | Count | Derivation |
|---|---|---|
| Primitive colours | 132 | 12 hues x 11 steps, a common shape because 11 steps gives usable contrast pairs |
| Primitive spacing, radius, type | about 30 | A geometric spacing scale of 8 to 10 steps, 4 radii, 8 type sizes, 4 weights |
| Semantic colour roles | about 45 | Surfaces, text, borders, actions, feedback states, each with a small number of variants |
| Component tokens | 0 to 180 | 30 components x up to 6, and most components should need none |

That last row is a judgement: component tokens are an escape hatch, not a tier
you populate on principle. A component that needs six of its own tokens is
usually a component whose semantic roles are missing.

```
+-------------------+
| PRIMITIVES        |
| blue-600 grey-200 |
| space-4 radius-2  |
+---------+---------+
          | referenced only by
          v
+-------------------+     +----------------------+
| SEMANTIC ROLES    | <== | THEME FILES          |
| surface           |     | light, dark, contrast|
| text-primary      |     | each redefines the   |
| action-bg         |     | same 45 role names   |
+---------+---------+     +----------------------+
          | referenced by
          v
+-------------------+
| COMPONENTS        |
| Button Card Input |
| no hex, no        |
| primitive names   |
+-------------------+
Caption: the reference direction between token layers, and where
a theme intervenes. Arrows point from definition to consumer.
```

### 12c. Theming without a flash

The flash of the wrong theme happens because the server does not know the user's
preference at the moment it produces the HTML. Four ways out, and they differ on
a dimension people forget: cache friendliness.

| Approach | Flash | Works without JavaScript | CDN cacheable | Supports explicit user override |
|---|---|---|---|---|
| CSS media query on the colour scheme preference only | None | Yes | Yes, one cached document | No, only the system setting |
| Blocking inline script in the head that reads storage and sets a class on the root element | None | No | Yes, one cached document | Yes |
| Framework effect after hydration sets the class | Yes, one full paint of the wrong theme | No | Yes | Yes |
| Server reads a cookie and renders the class into the HTML | None | Yes | No, the document is now user specific | Yes |

**The trade nobody states.** The server cookie approach is the most robust and it
makes the document uncacheable at the shared cache layer, because two users now
get different bytes. On a page whose main performance asset is a fully cached
HTML document, that is a real cost.

Three ways to have both, in increasing complexity:

- Vary the cached document on a single small cookie whose value has only two or
  three possibilities. Cache hit rate falls by the number of variants, not by the
  number of users.
- Keep the document shared and cached, and set the theme class from a blocking
  inline script. Cost: about 1 ms of parser blocking and a synchronous storage
  read, and it does not work with JavaScript disabled.
- Ship both themes as CSS that responds to a single attribute, and let an edge
  worker inject the attribute from the cookie into an otherwise cached document.
  Cost: an edge compute dependency.

**Measure the flash before you pay for any of this.** The framework effect
approach produces a visible wrong theme for the interval between first paint and
hydration completing. If that interval is 80 ms on your fastest population and
600 ms on your slowest, the slowest population sees a clearly perceptible flash
and the fastest does not. That distribution decides how much you should spend.

### Variant design, or how a component API stops scaling

The prop explosion is arithmetic. A button with 5 intents, 3 sizes and 2
emphases has 5 x 3 x 2 = 30 combinations. Almost certainly fewer than 30 are
designed, so the API allows states nobody has drawn.

| Approach | Legal states | Failure |
|---|---|---|
| One boolean per look: `isPrimary`, `isDanger`, `isGhost` | 2^3 = 8, of which 5 are contradictory | The type system permits `isPrimary` and `isDanger` together and the CSS resolves it by accident of order |
| One enumerated prop per axis | Full cross product, 30 | Permits undesigned combinations, which then appear in a screenshot and become expected |
| Enumerated axes plus an explicitly sanctioned combination table | Only what is designed, say 11 | More work up front, and a type error when someone reaches for combination 12, which is the moment to have the design conversation |

Prefer the third. The value is not correctness for its own sake: it is that the
type error routes an undesigned request to a designer instead of to a CSS
override in a product repo.

**When props are the wrong tool entirely.** If a component accumulates props
that only exist to reach inside it (`showHeaderIcon`, `headerIconSize`,
`footerAlignment`), the component wants composition instead. Expose named slots
or compound children, and let the consumer place elements. The signal to watch
for is a prop whose only job is to conditionally render another component.

### Versioning and deprecation of a library

Semantic versioning was designed for programmatic contracts and a component
library also has a visual contract, which semver has no slot for. State a policy
explicitly, because "is this a breaking change" is the question that consumes
the most time in a design system team.

| Change class | Example | Version bump | Extra requirement |
|---|---|---|---|
| API breaking | A required prop is renamed | Major | Ship a codemod in the same release |
| Visual breaking | Layout moves by more than about 2 px, or a semantic token's meaning changes | Major | Before and after images in the release notes |
| Visual additive | A new variant, a new component | Minor | None |
| Visual refinement | A sub pixel or sub 2 px adjustment, a transition duration | Patch | None, but log it, because someone's screenshot tests will fail |
| Internal | Refactor with identical rendered output | Patch | Verified by visual regression, not by assertion |

**The 2 px threshold is a choice, not a law.** The reason it is reasonable: below
about 2 px, consumers' visual regression tests will pass under ordinary
anti aliasing tolerance, and above it they will fail. Setting the policy
threshold at the point where it becomes visible to your consumers' tooling is
what makes it enforceable. Pick your own from your own tolerance settings.

**A deprecation clock with dates, not intentions.**

1. Release N: the replacement ships. The old component still works, unchanged.
2. Release N: documentation marks the old one deprecated and links the
   replacement. A codemod ships with it if the change is mechanical.
3. Release N+1: a development only console warning fires once per component per
   session. Never in production, because it is noise in a consumer's error
   tracking that they cannot fix on your schedule.
4. Removal: no earlier than the next major, and no earlier than 6 months after
   step 1, whichever is later.

The "whichever is later" clause is what makes it credible. A team that ships
majors monthly would otherwise give consumers four weeks.

**Adoption measured, not assumed.** Two numbers, both from a static scan of
consumer repositories:

- Share of component instances imported from the library versus locally
  reimplemented. Falling share means the library is not meeting a need, and the
  reimplementations tell you which.
- Count of consumer stylesheets that target the library's internal class names.
  Each one is a bug report you never received, and each one breaks on your next
  internal refactor. This number is the single best early warning that your API
  is too closed.

### 12d. Worked example

**The prompt.** "We have a 30 component library used by 6 applications. We need
to ship a high contrast theme for an accessibility commitment in one quarter,
and the brand team wants a colour refresh in the same window."

Inputs, stated before I touch anything:

| Input | Value | Status |
|---|---|---|
| Components | 30 | GIVEN |
| Consumer applications | 6, each with its own release train | GIVEN |
| Components with literal colour values | 27 | GIVEN |
| Engineers on the design system team | 3 | GIVEN |
| Quarter | 12 weeks | GIVEN |
| Time to convert one component to semantic tokens | 2 to 4 hours including review and visual regression | ASSUMED, from the work being mechanical but requiring a design decision per ambiguous value |

#### Block A. Purpose: separate the two requests, because they are not the same work

The brand refresh and the high contrast theme sound like one project and they
are not. I split them in the first two minutes:

- A brand refresh changes primitive values. If the token architecture is right,
  it is an edit to one primitives file.
- A new theme changes semantic role definitions. It is a new theme file plus,
  critically, decisions about what each role means in that theme.

Both are blocked by the same prerequisite: 27 components must stop containing
literal values. That prerequisite is the project. The two requests are each about
a day of work once it is done.

Saying this reframes the estimate from "two projects in a quarter" to "one
migration plus two small deliverables", which is both more honest and more
achievable.

**The error most readers make here.** Starting with the theme file. You can write
a beautiful dark theme and it will change nothing, because 27 components are not
listening.

!!! tip "Self-explanation prompt"

    Before reading Block B, say what you would do differently if only 3 of the
    30 components contained literal values instead of 27, and why the answer is
    not simply "the same thing but faster".

#### Block B. Purpose: size the migration before committing to a date

- 27 components x 2 to 4 hours = 54 to 108 hours
- 3 engineers, and assume 50 percent of their time goes to the migration because
  support and bug fixes do not stop. That is 60 hours per week x 0.5 x 3 = 60
  hours per week of migration capacity: 3 engineers x 20 hours = 60 hours per
  week
- 54 to 108 hours of work at 60 hours per week is 1 to 2 weeks

That result is suspiciously comfortable, which is the moment to look for what the
estimate excludes. Three things it excludes:

| Excluded | Real cost |
|---|---|
| Deciding the 45 semantic role names | 1 to 2 weeks with design, and it is the actual bottleneck |
| High contrast theme decisions per role | Contrast ratios must be verified per role pair, and some designs need structural changes such as adding borders where colour alone carried meaning |
| Consumer migration across 6 apps | Not our hours, but it is our calendar |

Revised: roughly 2 weeks of naming, 2 weeks of migration, 2 weeks of theme
authoring and contrast verification, leaving 6 weeks of buffer for consumer
adoption in a 12 week quarter. That is a plan. The first number was an estimate.

**The error most readers make here.** Estimating the mechanical work and
presenting it as the project estimate. The mechanical work is almost never the
long pole in a design system migration. Naming and agreement are.

#### Block C. Purpose: design the semantic layer against the hardest theme first

I author the role names against the high contrast theme, not against the current
light theme. The reason is that high contrast is the most constrained: it forces
every role pair to satisfy a contrast requirement, and it exposes any place where
colour alone was carrying meaning.

Concretely, this catches things like:

- A disabled state expressed only as reduced opacity. In high contrast that must
  become a distinct border or pattern, so the role set needs a
  `border-disabled`, which nobody would have invented from the light theme.
- A selected row expressed as a pale tint. High contrast needs an explicit
  selected border role.
- An error field expressed only as a red border. That is a colour only signal in
  any theme and it needed an icon and text regardless. The theme work surfaced an
  existing accessibility defect.

I say that last one out loud, because "this migration will surface accessibility
defects that exist today" sets an expectation that protects the schedule later.

**The error most readers make here.** Deriving semantic roles from the current
light theme by mechanical extraction. You get names like `surface-f5f5f5`, which
is a hex value with extra steps, and the constrained themes then have no
vocabulary for the distinctions they need.

!!! tip "Self-explanation prompt"

    Say why designing against the most constrained theme first is the same
    argument as designing an API against its most demanding consumer, and name one
    case where the argument would not hold.

#### Block D. Purpose: get 6 independent applications onto it without a flag day

Six consumer apps with six release trains means there is no date on which
everyone upgrades. So the migration must be non breaking at every step.

The sequence, and why each step is safe:

| Step | Change | Why consumers are unaffected |
|---|---|---|
| 1 | Add semantic tokens with values identical to today's literals | Purely additive. Rendered output is byte identical |
| 2 | Convert components to reference semantic tokens | Visual regression proves identical output. No API change, so it is a patch release |
| 3 | Ship theme files as an opt in attribute on the root element | Absent the attribute, nothing changes |
| 4 | Ship the brand refresh as new primitive values | This one is a visible change, so it is a major, with before and after images |
| 5 | Consumers adopt the major on their own schedule | Their choice, their train |

Steps 1 through 3 are all non breaking, which means the entire enabling work can
land without coordinating six teams. Only step 4 needs a conversation, and by
then the mechanism is proven.

**The error most readers make here.** Bundling the enabling refactor with the
visible change in one major release. It maximises the blast radius of the riskiest
step and it makes any visual bug ambiguous between the refactor and the rebrand.

#### Block E. Purpose: keep it from decaying after the quarter

Three mechanisms, each cheap, each addressing a specific decay path.

| Decay path | Mechanism |
|---|---|
| Components drift back to literals under deadline pressure | A lint rule that fails on hex values and primitive token names inside component files. Mechanical, not social |
| Consumers copy components into their own repos | Track the import share number defined in the versioning section monthly, and treat a fall as a product signal rather than a compliance problem |
| Consumers override internal class names | Scan for it, and treat each occurrence as a missing API. Add the slot or prop they were reaching for |

I explicitly do not propose a review process or a governance council, because
those depend on attention and the mechanisms above do not.

**The error most readers make here.** Ending at the migration. A design system
that is correct on the day it ships and unmaintained after is worse than no
system, because six teams have now coupled to it.

### 12e. Fade the scaffold

**Fade 1: adding a second brand.** The same library must serve two products with
different palettes, different type scales and different border radii, sharing all
behaviour. Same five block shape, last block blank.

- Block A, separate the requests: two brands is a primitives problem if the two
  brands agree on roles, and a semantics problem if they do not. Check first
  whether both brands have the same concept of, say, a raised surface. If yes,
  one semantic layer with two primitive sets. If no, you have two design systems
  sharing components, which is a much larger claim.
- Block B, size it: the type scale and radius differences mean primitives beyond
  colour must also be tokenised. Count them: about 30 non colour primitives from
  12b, and any component with a hard coded pixel value needs the same treatment
  the colours got.
- Block C, design against the hardest case: the brand with the larger type scale,
  because larger type is what breaks fixed height components, truncation and
  dense layouts. Designing against the compact brand first hides every overflow
  bug.
- Block D, adopt without a flag day: brand two is purely additive, so every step
  is non breaking by construction. The risk is not release management, it is that
  brand two's components look subtly wrong in ways brand one never revealed.
- Block E, keep it from decaying: **you write this block.**

??? note "Show answer"

    The decay path here is different from the theme case and that is the point.
    With one brand, decay looks like literals creeping back. With two brands,
    decay looks like brand specific conditionals appearing inside shared
    components: a check for which brand is active, then two code paths.

    Mechanism one: a lint rule that fails on any reference to a brand identifier
    inside a component file. The brand must reach the component only through
    token values, never through a conditional. This is the two brand analogue of
    the no primitives rule.

    Mechanism two: run the visual regression suite for both brands on every
    change, not just the default. Otherwise brand two regresses silently and is
    discovered by its product team, which is the fastest way to lose their trust
    in the shared library.

    Mechanism three: measure the ratio of brand specific tokens to shared ones. A
    rising ratio means the two brands are diverging and the shared library is
    becoming a costly fiction. Set a threshold in advance, for example above
    about 30 percent brand specific, and agree what you will do when it is
    crossed, which is usually to split the libraries rather than to fight it.

    The honest version of this answer includes that last sentence. Deciding in
    advance what evidence would make you abandon the shared library is more
    convincing than asserting it will work.

**Fade 2: deprecating a widely used component.** A `Dropdown` component used in
about 400 places across 6 apps must be replaced by a `Select` with a different
API and a different accessibility model. Last two blocks blank.

- Block A, separate the requests: there are two here, an API migration and a
  behaviour change. The accessibility model change means the replacement is not
  a drop in even where the API matches, so a codemod can move the call sites but
  cannot verify the result.
- Block B, size it: 400 call sites. A codemod handles the mechanical share, and
  the estimate turns on what fraction is mechanical. If 80 percent are simple
  usages, the codemod moves 320 and 80 need a human, at say 30 minutes each,
  which is 40 hours spread across 6 teams.
- Block C, design against the hardest case: **you write this block.**
- Block D, adopt without a flag day: **you write this block.**

??? note "Show answer"

    Block C. The hardest case for a select style component is not the longest
    option list, it is the one with a custom option renderer, asynchronous
    loading, multiple selection and a form validation relationship. Find the two
    or three most complex existing usages across the 6 apps before designing the
    replacement API, and design against those.

    The reason this matters more than usual: a component with 400 call sites has
    a long tail of usages the library team has never seen. If the new API cannot
    express the hardest three, those teams will not migrate, and you will support
    both components indefinitely, which is the actual failure mode of most
    deprecations.

    The accessibility model is the second hard case and it belongs in the API.
    If the old component managed focus one way and the new one uses a different
    interaction pattern, then keyboard behaviour changes for users of every one
    of the 400 sites. That must be in the release notes as a behaviour change,
    not buried as an improvement.

    Block D. The sequence, and the deprecation clock from the versioning section
    with dates attached.

    Release N: `Select` ships alongside `Dropdown`. Both are supported. A codemod
    ships in the same release and handles the mechanical usages. Documentation
    marks `Dropdown` deprecated with a link and a migration guide for the three
    hard cases identified in Block C.

    Release N+1: a development only warning fires once per session per component.
    Not in production. At the same time, publish the adoption number per consumer
    app, because visibility is what moves the last 20 percent.

    Removal: the next major and no sooner than 6 months. With 6 independent
    release trains, some app will be mid quarter on something else at any given
    date, so the "whichever is later" clause is doing real work here.

    One addition specific to 400 call sites: do not remove `Dropdown` on schedule
    if adoption is under, say, 90 percent. Removing it on principle converts your
    deadline into six teams' emergency. Publish the number, extend once with a
    new date, and escalate through the consumer teams' own planning rather than
    through a breaking release.

### 12f. Check yourself

**Q1.** A consumer team reports that your patch release broke their screenshot
tests. Your change was a 1 px adjustment to a card's internal padding. Who is
wrong, and what do you change?

??? note "Show answer"

    Nobody is wrong, and that is why this needs a written policy rather than a
    judgement call. Their tests are doing exactly what screenshot tests do, and
    your change is exactly what a patch is meant to permit.

    From the change class table above, a sub 2 px adjustment is a patch and
    should be logged in the release notes even though it is not breaking. If your
    release notes did not mention it, that is the defect: the fix is to log every
    visual change regardless of size, so consumers can attribute a failing
    screenshot to a known cause in under a minute.

    The second thing to change is the threshold conversation. If several consumers
    run screenshot tests with tight tolerance, then your 2 px policy line is in
    the wrong place for your actual consumers. Move it down or ask them to raise
    tolerance, and write down which.

    The wrong answer is to promise that patches never change pixels. That
    promise makes every visual refinement a major release, and the library
    freezes within two quarters.

**Q2.** Your library defines 180 component level tokens across 30 components.
Give the two most likely diagnoses and what each implies.

??? note "Show answer"

    Diagnosis one: the semantic layer is missing roles that components genuinely
    need, so each component invented its own. The evidence is repetition, meaning
    the same concept appearing under different component prefixes, for example
    `button-border-focus`, `input-border-focus` and `card-border-focus` all
    holding the same value. The fix is to promote the repeated concept into a
    semantic role and delete the component tokens.

    Diagnosis two: component tokens are being used as a public override surface,
    because consumers had no other way to customise. The evidence is that
    consumers set them from outside the library. That is not a token problem, it
    is an API problem: the component needs a slot, a variant or a prop, and the
    token is a workaround with no type checking and no versioning.

    Both diagnoses are found the same way: group the 180 tokens by the value they
    hold and by who reads them. Tokens sharing a value across components are
    diagnosis one. Tokens read from outside the library are diagnosis two.

    An average of 6 component tokens per component is the signal that something
    is systematically wrong, because most components should need zero.

### 12g. When not to use this

**A three layer token architecture.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You have more than one theme, or you can name a concrete second theme with a date, AND components are consumed by teams who cannot all be edited in one commit |
| Scale threshold below which it is over-engineering | Under roughly 15 components with a single theme in a single repository, two layers (primitives plus components) is enough, and the semantic layer is naming work with no consumer |
| Cheaper alternative | Primitives plus a lint rule banning literal hex in component files. You get the concentration benefit without inventing a role vocabulary you cannot yet validate |

**A separately versioned, published component library.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Three or more consumer applications with genuinely independent release trains, so there is no date on which everyone can adopt a change |
| Scale threshold | With one or two applications in one repository, publishing adds version negotiation, changelog discipline and a deprecation clock to solve a problem you do not have. Ship from head |
| Cheaper alternative | A shared directory in the monorepo with ownership rules and a build that fails on unauthorised cross module imports. Same discipline, no version skew |

**A codemod shipped with a breaking change.**

| Question | Answer |
|---|---|
| Measurement that justifies it | More than roughly 40 call sites across repositories you do not own, AND the transformation is mechanically decidable from the syntax alone |
| Scale threshold | Under about 40 call sites, a well written migration note and a find and replace is faster than writing, testing and supporting a codemod |
| Cheaper alternative | A precise migration guide with before and after examples for the three most common shapes, plus a lint rule that flags the old usage with the replacement in the message |

**A high contrast theme as a separate theme.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Your default themes cannot meet the contrast requirement without changing the brand, AND you have users who have asked for it or a commitment that names it |
| Scale threshold | If your default theme already meets the required contrast ratios for text and interface components, a separate theme duplicates maintenance for no user gain. Fix the default instead |
| Cheaper alternative | Raise contrast in the default theme and support the operating system's forced colours mode, which requires that you never encode meaning in colour alone. That work is required anyway |

---

## Module 13. Micro-frontends, honestly

**Time: 30 to 40 minutes reading, 20 to 30 minutes practice.**

This module is deliberately structured to help you say no. Micro-frontends solve
an organisational problem at a technical cost, and the interview signal is
whether you can price that cost rather than whether you can name three
integration strategies.

### 13a. Predict first

Read this artefact and commit to an answer before continuing.

```
+---------------------------------------------------------------+
| One page composes the shell plus 3 of 4 deployed apps.        |
| Compressed bytes each app ships, by part:                     |
|                                                               |
|   SHELL     framework 45  router 12  design system 60         |
|   CATALOG   framework 45  router 12  design system 60  app 90 |
|   CART      framework 45  router 12  design system 58  app 40 |
|   REVIEWS   framework 44  router 12  design system 60  app 35 |
|                                                               |
| The user loads SHELL + CATALOG + CART + REVIEWS.              |
+---------------------------------------------------------------+
Caption: four independently built bundles broken into shared
libraries and unique application code.
```

**Question.** Compute the duplicated bytes, convert them to seconds using
200 KB/s of throughput and 1 ms of main thread per KB of uncompressed source at
3.3x expansion, then name the one number in the table that blocks the obvious
fix.

??? note "Show answer"

    Totals: shell 117, catalog 207, cart 155, reviews 151. Sum = 630 KB
    compressed.

    Unique application code: 90 + 40 + 35 = 165 KB. One copy of the shared
    libraries: 45 + 12 + 60 = 117 KB. So the theoretical floor is 165 + 117 =
    282 KB, and the duplication is 630 - 282 = 348 KB.

    Transfer: 348 / 200 = 1.74 s. Main thread: 348 x 3.3 = 1,148 KB uncompressed
    at 1 ms per KB = 1.15 s. Total cost of duplication: about 2.9 s, of which
    about 40 percent is CPU that a fast network does not help with.

    The blocking number is the 44 in the reviews row. Three apps are on framework
    version A and one is on version B. Sharing a singleton across apps requires
    them to agree on a version, so deduplication is blocked until the reviews
    team ships an upgrade. Their release train is, by the whole premise of this
    architecture, not yours to schedule.

    That is the honest shape of the trade. The duplication is not a bug in the
    tooling, it is the price of the independence the architecture was adopted to
    buy, and it is paid by the user on every page load.

### 13b. Five integration strategies, compared on what actually differs

Define the terms. **Integration** here means how code from independently
deployed teams ends up executing on one page. **Version skew** means two apps
needing different versions of the same library at the same time.

| Strategy | Deploy independence | Bundle duplication | Skew tolerance | Error isolation | Routing owner |
|---|---|---|---|---|---|
| Build time packages: each team publishes a package, one app builds them | None, a release requires a shell rebuild | None, one dependency graph | Low, one version wins | None, one crash takes the page | Shell |
| Server or edge composition: fragments assembled into one HTML response | High, per fragment | High unless assets are coordinated | High, fragments are separate documents until merged | Good for render errors, poor for runtime errors | Shell or edge |
| Iframes | Very high | Very high, nothing shared | Total, apps share nothing | Excellent, a crash is contained by the browser | Complex, needs message passing |
| Runtime module federation: apps expose modules loaded at runtime | High | Configurable, shared singletons deduplicate | Medium, shared modules need compatible ranges | Only what you build | Shell, by convention |
| Web component wrappers | High | High unless the framework is shared | High, each element brings its own | Medium, errors escape the element | Shell |

**The pattern in that table.** Every column trades against isolation. The
strategies that isolate best (iframes) duplicate most and communicate worst. The
strategies that share best (build time packages) remove the deploy independence
that was the reason to do this at all.

**The dishonest middle.** Runtime federation with aggressive singleton sharing
markets itself as having both. It does not. It converts the duplication cost
into a coordination cost, which is less visible and often larger.

**Quantify the coordination cost.** With N apps and S shared singletons, every
singleton upgrade requires agreement across N teams, so the coupling surface is
N x S pairs.

- 4 apps, 3 singletons (framework, router, design system) = 12 pairs
- A framework major upgrade needs 4 teams to schedule work in the same window
- The team that cannot schedule it blocks the other three

That last line is where the architecture is decided. If your organisation can
coordinate 4 teams on a version bump, you probably did not need micro-frontends.
If it cannot, your singletons will skew and you will pay the duplication anyway.

### 13c. The three problems that are not about bundling

**Routing ownership.** The URL is a shared global namespace, so it needs an
explicit owner and an explicit contract.

```
+-------------------------------------------------------------+
| SHELL owns the prefix table and nothing below it            |
|   /catalog/*   ==> CATALOG app                              |
|   /cart/*      ==> CART app                                 |
|   /reviews/*   ==> REVIEWS app                              |
|   /            ==> SHELL home                               |
+------------------------------+------------------------------+
                               |
        +----------------------+----------------------+
        v                      v                      v
+---------------+     +---------------+     +---------------+
| CATALOG owns  |     | CART owns     |     | REVIEWS owns  |
| every path    |     | every path    |     | every path    |
| under its     |     | under its     |     | under its     |
| prefix, and   |     | prefix        |     | prefix        |
| may not read  |     |               |     |               |
| others        |     |               |     |               |
+---------------+     +---------------+     +---------------+
Caption: a prefix table owned by the shell, with each app owning
its own subtree and no app reading another app's paths.
```

Two rules that prevent most routing incidents:

- An app may only construct links into another app through a shell provided
  function, never by string concatenation. Otherwise a prefix change in one team
  breaks links in three others silently.
- Deep links must work on a cold load. If arriving directly at
  `/reviews/product/17/page/3` requires the catalog app to have run first, you
  have a hidden dependency that only appears in shared links and search results.

**Cross app authentication, and the isolation myth.** State this plainly in an
interview because it is where confident answers go wrong.

| Claim | Reality |
|---|---|
| "Each app is isolated" | Same origin apps share one JavaScript context, one DOM, one cookie jar and one storage area. Isolation is organisational, not technical |
| "We store the token per app" | Any script on the page can read any storage on that origin. There is one blast radius |
| "An untrusted third party app is safe in a mount point" | It is not. Untrusted code needs a cross origin iframe, which is a different architecture |

The workable pattern for trusted first party apps: the shell holds the token in
a closure in memory and exposes an async `getToken()` through the mount
contract. Apps never touch storage. This does not create a security boundary,
and its actual benefit is that refresh, expiry and logout have exactly one
implementation instead of four.

**Error isolation, built rather than assumed.** Four mechanisms, cheap, in the
order they pay off:

| Mechanism | Catches |
|---|---|
| An error boundary per mount point with an app scoped fallback | Render errors in one app, keeping the rest of the page usable |
| A timeout on loading a remote entry, say 3 s, with a fallback | A slow or dead remote turning into an indefinitely blank region |
| A shell side circuit breaker per app, opened after repeated failures | Repeated retries of a remote that is down for everyone |
| A build identifier per app reported with every error | Attribution, which is otherwise impossible when four deploy trains overlap |

The timeout number is a choice and here is the reason 3 s is defensible: it is
above the point where a slow but functioning CDN edge would resolve, and below
the point where a user concludes the page is broken. Set yours from your own
remote entry latency distribution.

### 13d. Worked example

**The prompt.** "We have one large e-commerce front end. Four teams work in it.
Releases are weekly and every release requires all four teams to sign off. The
vice president wants micro-frontends. Design it, or tell them not to."

Inputs, stated before I touch anything:

| Input | Value | Status |
|---|---|---|
| Teams working in the repository | 4 | GIVEN |
| Engineers total | 22 | GIVEN |
| Release cadence today | Weekly, coordinated | GIVEN |
| Time from merge to production | 3 days | GIVEN |
| Rollbacks in the last quarter caused by another team's change | 5 of 7 | GIVEN |
| Current bundle, compressed | 470 KB | GIVEN |
| Framework, router and design system share of that | 117 KB | GIVEN |

#### Block A. Purpose: find out what problem is actually being solved

I do not design anything yet. I ask what improves if this works, and I try to get
a number for it.

| Candidate problem | Does this architecture fix it | Evidence in the inputs |
|---|---|---|
| Coordinated release train | Yes, this is the one it fixes | 3 day merge to production, weekly coordinated release |
| Cross team breakage | Partly. It converts build time breakage into runtime breakage | 5 of 7 rollbacks were another team's change |
| Build times | Sometimes, and a build cache or project references usually does it more cheaply | Not stated, so I ask |
| Technology diversity | Yes, at a cost most teams should refuse | Not requested, so I do not design for it |
| Bundle size | No. It makes this strictly worse | 117 KB of shared libraries would be duplicated per app |

The strongest number here is 5 of 7 rollbacks caused by another team. That is a
real coupling cost. I say out loud that it is also the number most likely to get
worse, because a runtime integration failure is harder to catch before
production than a build failure.

**The error most readers make here.** Designing the architecture that was asked
for. The question contains "or tell them not to" and the interviewer put it there
on purpose. An answer that never evaluates the premise scores lower than one that
evaluates it and then proceeds.

!!! tip "Self-explanation prompt"

    Before reading Block B, decide which single additional number you would ask
    for that could flip your recommendation, and say which way it would flip it.

#### Block B. Purpose: price the cheaper alternative honestly first

The cheaper alternative is a modular monolith with enforced boundaries. I price
both against the actual complaint, which is the coordinated release.

| Option | What it costs | What it delivers |
|---|---|---|
| Enforced module boundaries plus code ownership plus independent test suites | Days, mostly configuration | Faster review, clear blame, no runtime cost. Does not decouple the release |
| Trunk based with per module feature flags and independent enablement | Weeks, plus flag hygiene | Decouples enabling a feature from releasing code. Most of the benefit sought |
| Micro-frontends with runtime federation | A quarter, plus permanent operational surface | Decouples the deploy itself, and adds 348 KB of duplication in the artefact above |

The middle option deserves the emphasis. The complaint is that a release requires
four sign offs. Flags decouple "my code is live" from "my feature is on", which
removes most of the sign off pressure without splitting the deployment unit.

I would recommend the middle option, and I say what would change my mind: if
teams need to deploy at genuinely different cadences (one team shipping hourly,
another quarterly for regulatory review), flags do not help and separate
deployment units do.

**The error most readers make here.** Presenting the cheap option as a strawman.
Price it properly, including what it does not solve, or the recommendation is not
credible.

#### Block C. Purpose: design it properly if the premise survives

Assume the interviewer says the regulatory case is real: one team ships weekly,
one ships after a compliance review that takes a month. Now separate deployment
units are justified, and I design.

Choices, each with the reason:

| Decision | Choice | Reason |
|---|---|---|
| Integration | Runtime module federation | Keeps a single page application experience and permits sharing. Iframes would isolate better and break shared navigation and modals |
| Shared singletons | Framework and router only | These must be single instances for correctness. The design system is shared but not forced to a single version |
| Design system versioning | Ranged, with a supported window of two majors | The design system duplicating costs 60 KB per skewed app, which I would rather pay than block a compliance release |
| Routing | Shell owns the prefix table, apps own their subtrees | The contract from 13c |
| Auth | Shell holds the token, exposes `getToken()` | One implementation of refresh and logout |
| Error handling | Boundary per mount, 3 s remote timeout, circuit breaker, build id in every error report | The four mechanisms from 13c |

I explicitly bound the number of apps at four and say why: the coordination
surface is N x S, so adding apps grows coupling linearly while adding no new
independence for the teams that already have it.

**The error most readers make here.** Sharing everything as a singleton because
duplication looks bad on a chart. Forcing the design system to one version
recreates the coordinated release you just spent a quarter escaping, in a place
where it is harder to see.

!!! tip "Self-explanation prompt"

    Say why the framework must be a forced singleton while the design system need
    not be, in terms of what breaks if two versions are present at once.

#### Block D. Purpose: make the runtime failure modes visible before they happen

Splitting the deployment unit converts a class of build errors into a class of
production errors. I name them and design for each.

| Failure | Trigger | Design response |
|---|---|---|
| Remote entry unavailable | An app's CDN path 404s after a bad deploy | 3 s timeout, cached last known good entry, app scoped fallback UI |
| Contract drift | Shell passes a prop the app no longer reads | A versioned mount contract, with the version asserted at mount time and logged |
| Skew crash | Two design system versions load and one writes global styles | Scope styles per app, and treat any global selector in a shared package as a defect |
| Silent regression | One app doubles its bundle | Per app byte budget with per app ownership, from Module 11 |
| Unattributable error | An error in a shared library, four possible callers | Build identifier per app in every error report and every performance sample |

The last row is the one that is usually discovered too late. With one deployment
unit, an error's build identifier tells you exactly what code is running. With
four, you need four, reported together, or your error tracking becomes guesswork
during an incident.

**The error most readers make here.** Treating observability as a later phase.
In this architecture attribution is the product of the design, not an add on: if
the build identifiers are not in the contract from day one, they arrive after the
first incident that needed them.

### 13e. Fade the scaffold

**Fade 1: an internal admin console.** Three teams, 9 engineers total, one
deploy pipeline, no regulatory constraint, current bundle 280 KB compressed, and
the stated complaint is "our builds take 4 minutes". Same four block shape, last
block blank.

- Block A, find the real problem: the complaint is build time, and the input
  contains no release coupling and no independence requirement. Build time is a
  build tooling problem. Ask what the 4 minutes consists of before anything else.
- Block B, price the cheaper alternative: incremental builds, a shared build
  cache and project references typically address this at a cost of days. A
  micro-frontend split addresses it at the cost of a quarter and adds runtime
  failure modes that a 9 person team now has to operate.
- Block C, design it if the premise survives: the premise does not survive. Nine
  engineers across three teams is below the threshold where deploy independence
  buys anything, and the console is internal so the 100 KB or more of duplication
  is genuinely cheap. The recommendation is no.
- Block D, make the failure modes visible: **you write this block.**

??? note "Show answer"

    The honest answer to Block D is that this block changes shape when the
    recommendation is no, and pretending otherwise would be padding. There are no
    runtime integration failure modes to design for, because there is no runtime
    integration.

    What replaces it is the failure mode of the recommendation itself. The
    predictable one: the team asks again in six months, having grown, and nobody
    remembers why the answer was no. So write down the threshold that would
    change it, with numbers, in a document that will still exist.

    A defensible threshold set for this team: more than about 25 engineers
    touching one deployment unit, OR two teams that genuinely cannot release on
    the same cadence, OR a measured coupling cost such as more than a third of
    rollbacks being caused by another team's change. Any one of the three, not
    all three.

    The second failure mode: the build cache work does not happen either, and the
    original complaint stays. Attach the build time fix to the same decision so
    the answer is "no, and here is the thing we are doing instead, with an owner
    and a date". A recommendation to do nothing is rarely accepted, and it is
    rarely what you meant.

**Fade 2: unwinding an existing split.** An organisation has 6 micro-frontends,
built 3 years ago by a team that has since left. Two are actively developed, four
change roughly once a year. Page load ships 5 copies of a framework. Last two
blocks blank.

- Block A, find the real problem: the architecture is being paid for by every
  user on every page load, and the independence it buys is used by 2 of the 6
  apps. The mismatch between where the cost falls and where the benefit lands is
  the finding.
- Block B, price the alternative: merging the 4 low change apps into one
  deployment unit removes 4 remote loads and 3 duplicate framework copies. At
  45 KB per copy that is 135 KB compressed, about 0.68 s of transfer at 200 KB/s
  plus about 0.45 s of CPU at the stated assumption. The 2 active apps keep their
  independence.
- Block C, design the target state: **you write this block.**
- Block D, make the failure modes visible: **you write this block.**

??? note "Show answer"

    Block C. The target is a hybrid rather than a purity move in either
    direction, and being willing to say that is the point of this fade.

    The four low change apps merge into one deployment unit with enforced module
    boundaries and code ownership, so their teams keep review control without
    keeping a deploy pipeline each. The two active apps stay as separate remotes.
    That is 2 remotes plus 1 merged unit plus the shell, down from 6 plus shell.

    Shared singletons: framework and router forced to one version across the
    remaining remotes, which is now a coordination across 2 teams rather than 6,
    so it is achievable. Design system ranged over two majors as before.

    Sequencing matters because there is no owning team: merge the four quiet apps
    first, since they change least and therefore carry the least migration risk
    and give the largest byte win. Do not start by upgrading the framework
    everywhere, which is the tempting first move and the one most likely to
    stall.

    Block D. The dominant failure mode of an unwind with no original owners is
    undocumented coupling: something in one of the quiet apps depends on a global
    that the shell happens to set, or on load order, and nobody knows.

    Design for discovery rather than for prevention. Merge one app at a time
    behind a flag that routes a percentage of traffic to the merged build, with
    the old remote still deployed. Compare error rate and the two vitals that
    would move (INP, because the merged bundle changes main thread work, and LCP,
    because it changes the request waterfall) between the two arms. Keep the old
    remote until the arm has been at 100 percent for a full release cycle.

    Two specific things to instrument during the unwind: any access to a global
    variable not declared in the mount contract, logged rather than blocked, and
    duplicate framework instance detection, which is a two line check that
    catches the most confusing class of bug in this whole architecture.

    And write the reason for the hybrid down. Hybrids are the state most likely
    to be "cleaned up" by the next arriving engineer in either direction.

### 13f. Check yourself

**Q1.** A team proposes micro-frontends and says the main benefit is that each
team can pick its own framework. Give the strongest version of their argument and
then your response.

??? note "Show answer"

    The strongest version: framework choice is a hiring and productivity
    argument, not a technical preference. A team that is expert in one framework
    and forced into another ships more slowly and hires from a smaller pool. If
    an acquisition brought in a team with a working product in a different
    framework, rewriting it is months of work delivering zero user value, and
    running it as is delivers value on day one.

    The response, in three parts. First, price it: each additional framework is a
    duplicated runtime on any page where both are present, typically tens of
    kilobytes compressed plus its share of CPU, paid by every user of that page.
    If the frameworks never co-occur on one page, the cost is much lower and the
    argument is much stronger, so ask about co-occurrence before arguing.

    Second, name the hidden costs: two accessibility implementations, two design
    system ports, two sets of testing infrastructure, two upgrade treadmills, and
    the fact that engineers can no longer move between teams, which is the thing
    that made the hiring argument work.

    Third, offer the narrow version: framework diversity as a migration mechanism
    with an end date, or as an integration mechanism for an acquisition, is
    reasonable. As a permanent standing policy it multiplies the platform team's
    work by the number of frameworks and that team is usually already the
    constraint.

**Q2.** Your shell forces a single version of the design system. A team blocks a
release because a design system major broke their app. What does this tell you
about the architecture, and what would you change?

??? note "Show answer"

    It tells you the architecture has re-created the coordinated release it was
    adopted to remove, at the design system boundary. Deploy independence that
    depends on all teams accepting the same library version at the same time is
    not deploy independence.

    The diagnosis is that a forced singleton is only correct for libraries where
    two instances are actually incorrect: the framework (two instances of a
    component runtime break context and state), and the router (two instances
    fight over the URL). A design system is styles and components, and two
    versions coexisting is visually inconsistent, not broken.

    The change: move the design system to a supported range, for example any of
    the current or previous major, with styles scoped per app so two versions
    cannot collide through global selectors. Price it: 60 KB compressed per
    skewed app in the artefact at the top of this module, about 0.3 s of transfer
    plus 0.2 s of CPU. Pay that to unblock a release, and treat it as temporary
    by tracking how long any app stays on the older major.

    Add one guardrail so the range does not become permanent: report the version
    each app loads at runtime, and set an internal target such as no app more than
    one major behind for more than a quarter. Visible skew gets fixed. Invisible
    skew becomes the new baseline.

### 13g. When not to use this

**Micro-frontends at all.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Two or more teams that genuinely cannot release on the same cadence, for example one blocked by a compliance review, OR a measured coupling cost such as a large share of rollbacks caused by another team's change, AND more than roughly 25 engineers in one deployment unit |
| Scale threshold below which it is over-engineering | Under about 20 to 25 engineers, or where the actual complaint is build time or review latency, the runtime cost and the permanent operational surface exceed any benefit. The user pays the duplication and gets nothing |
| Cheaper alternative | A modular monolith: enforced module boundaries with a build that fails on unauthorised cross module imports, code ownership per directory, independent test suites, and per module feature flags so enabling a feature is decoupled from releasing code |

**Forced singleton sharing of a library.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Two instances of the library are incorrect, not merely wasteful. The test: can you name a concrete bug that appears with two instances (context is lost, the URL is fought over, a global registry is duplicated) |
| Scale threshold | For a library where duplication is only a size cost, forcing one version reintroduces coordinated releases. At 60 KB compressed, about 0.3 s of transfer, it is usually cheaper to duplicate than to block a team |
| Cheaper alternative | A supported version range spanning two majors, styles and globals scoped per app, and a dashboard of which app is on which version so skew stays visible |

**Iframes as the integration strategy.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You are hosting code you do not control or do not trust, so you need a real security boundary. Same origin composition provides none |
| Scale threshold | For first party apps built by colleagues, iframes cost you shared modals, shared focus management, correct scroll behaviour, deep linking and a coherent design, in exchange for isolation you do not need |
| Cheaper alternative | Same origin composition with an error boundary per mount point. You get containment of crashes, which is the part first party teams actually need, without the interaction penalties |

**A shell provided authentication service.**

| Question | Answer |
|---|---|
| Measurement that justifies it | More than one app needs the same credential, and refresh, expiry and logout logic would otherwise be implemented more than once. Count the implementations you have today |
| Scale threshold | With one application, this is a module, not a service, and wrapping it in a mount contract adds indirection with no consumer |
| Cheaper alternative | A shared authentication package consumed at build time. Same single implementation, no runtime contract to version, and it fails at compile time rather than at mount time |

---

## Module 14. Security in the browser

**Time: 45 to 55 minutes reading, 30 to 40 minutes practice.**

Browser security questions are graded on whether you can reason about a trust
boundary, not on whether you can list acronyms. The organising question for the
whole module: which code executes in my origin, and what can it reach?

### 14a. Predict first

Read this artefact and commit to an answer before continuing.

```
+---------------------------------------------------------------+
| Comment rendering, in production:                             |
|   el.innerHTML = renderMarkdown(comment.body)                 |
|                                                               |
| Response header on every page:                                |
|   Content-Security-Policy: default-src 'self';                |
|     script-src 'self' 'unsafe-inline' cdn.partner.example     |
|                                                               |
| Session credential: a signed token in localStorage,           |
|   attached by the client as an Authorization header.          |
+---------------------------------------------------------------+
Caption: a rendering sink, a content security policy, and a
credential storage choice, all shipped together.
```

**Question.** Rank the three by contribution to the worst case outcome, and say
specifically what the policy header is worth against this particular defect.

??? note "Show answer"

    Rank: the innerHTML assignment is the vulnerability, the localStorage token
    sets the blast radius, and the policy is a control that has been configured
    into irrelevance.

    The sink. Markdown renderers commonly pass through raw HTML unless configured
    not to, so a comment containing an image tag with an error handler attribute
    becomes executing script. Assigning to innerHTML is the moment untrusted text
    becomes markup, and no amount of validating the input earlier helps, because
    the danger is created at the sink rather than at the source.

    The policy. `'unsafe-inline'` in the script directive permits exactly the
    payloads this defect produces: inline event handler attributes and inline
    script elements. With it present and no nonce, the policy blocks nothing
    relevant. Separately, allowlisting a partner host means any script that host
    will serve becomes executable in your origin, which is a standing dependency
    on someone else's hosting hygiene.

    The credential. A token in localStorage is readable by any script running on
    the origin, so a successful injection exfiltrates a bearer credential the
    attacker can then use from their own machine for the life of the token. With
    an HttpOnly cookie the attacker can still act as the user while the page is
    open, which is bad, but they cannot carry the credential away, which is the
    difference between an incident and a breach.

    The transferable point: a policy that allows inline script is not a mitigation
    with a gap, it is an absent mitigation with a header.

### 14b. Cross-site scripting, organised by sink

**Definition.** Cross-site scripting is any path by which text controlled by an
attacker becomes code executing in your origin.

Three delivery routes:

| Type | Where the payload lives | Typical vector |
|---|---|---|
| Stored | Your database | A comment, a profile name, a file name |
| Reflected | The request | A query parameter echoed into the response |
| DOM based | Neither, it never reaches the server | A hash fragment read by client code and written to a sink |

**The sinks are the short list worth memorising.** Everything else is a path to
one of these.

| Sink | Becomes dangerous when |
|---|---|
| `innerHTML`, `outerHTML`, `insertAdjacentHTML` | Any untrusted substring, always |
| A framework's raw HTML escape hatch | Same, and it looks deliberate, so it survives review |
| `href` or `src` set from data | The value can start with a script scheme |
| `eval`, `Function`, `setTimeout` with a string | Any untrusted substring |
| Injecting a `script` or `style` element | Always |
| Server side template interpolation into a script block | The escaping context is JavaScript, not HTML, and HTML escaping is the wrong function |

**The defence order that actually works.**

1. Default to text. Frameworks that escape by default are doing the heavy
   lifting. The remaining risk is concentrated in the escape hatches, which is a
   list you can enumerate and lint for.
2. Sanitise at the sink, not at the source. Input validation is worth doing for
   other reasons and it does not solve this, because the same stored value may be
   rendered into HTML, into an attribute, into a URL and into JavaScript, and each
   needs a different treatment.
3. Use a vetted sanitiser, never a regular expression. Parsing HTML with a
   pattern fails in ways that are well known to attackers and surprising to
   everyone else.
4. Enforce mechanically with a Trusted Types policy where supported, which
   converts every dangerous sink into a type error unless the value passed through
   an approved factory. This turns an audit into a compile time and runtime
   guarantee.

### 14c. Content security policy, and how to roll one out

A policy is a per response header that tells the browser what sources are
allowed for each resource type.

| Directive | What it constrains | Highest value use |
|---|---|---|
| `script-src` | Where script may come from | The whole point of a policy for most sites |
| `object-src` | Plugin content | Set to `'none'` unconditionally |
| `base-uri` | The document base element | Set to `'self'`, it blocks a base tag injection that redirects every relative script URL |
| `frame-ancestors` | Who may frame you | The clickjacking control |
| `form-action` | Where forms may submit | Blocks an injected form exfiltrating input |
| `require-trusted-types-for` | Enforces Trusted Types on sinks | The mechanical version of the sink discipline above |

**Allowlists versus nonces.** An allowlist of hosts is easy to write and weak,
because you inherit every script any allowlisted host will serve, including old
library versions and any endpoint that reflects a caller supplied callback name.

A nonce based policy is stronger: the server generates a random value per
response, the policy accepts scripts carrying it, and injected markup cannot
guess it. Add `'strict-dynamic'` so scripts loaded by a trusted script are also
trusted, which is what makes a nonce policy survive contact with a real bundler.

Three constraints to state honestly:

- The nonce must be per response and unguessable, which means the HTML document
  cannot be a statically cached file without an edge step to inject it.
- If a nonce is present, browsers that support nonces ignore `'unsafe-inline'`,
  so it can stay for older clients without weakening the modern ones. Say this,
  because it is the sentence that unblocks most rollouts.
- Any inline event handler attribute in your own markup will break. That is the
  actual migration cost, and it is worth counting before promising a date.

**A rollout plan with numbers.**

| Phase | Duration | What you do | What tells you to move on |
|---|---|---|---|
| 1. Report only | 2 weeks | Ship the policy in report only mode with a reporting endpoint | Report volume stabilises and you can classify every distinct violation |
| 2. Triage | 1 week | Group reports by directive and blocked source. Expect the large majority to be browser extensions and injected consumer software, which you cannot fix and must not allowlist | You have a list of violations caused by your own code |
| 3. Fix | Varies | Remove inline handlers, add the nonce, move to strict-dynamic | Report volume from your own origin approaches zero |
| 4. Enforce | Ongoing | Switch to enforcing mode, keep the reporting endpoint | Alert on any rise, which now means a real change or a real attack |

The extension noise in phase 2 is the part that surprises people and it is why
report only must run long enough to characterise the baseline. Skipping straight
to enforcement breaks pages for a population you cannot identify.

### Request forgery, framing and cross-origin rules

**Cross-site request forgery, stated as a mechanism.** The browser attaches
cookies to requests based on the destination, not on who initiated them. So a
form on an attacker's page can cause an authenticated state changing request to
your origin. The attacker cannot read the response, which is why forgery targets
writes rather than reads.

| Defence | How it works | Where it falls short |
|---|---|---|
| `SameSite=Lax` cookies | The cookie is withheld on cross-site subrequests and on cross-site form posts, but sent on top level navigations by GET | Any state changing GET endpoint is still exposed. A compromised subdomain is same-site, so it is not defended |
| `SameSite=Strict` | Withheld on all cross-site requests including top level navigation | Breaks inbound links from email and search into authenticated pages, so it usually needs a second, session-lite cookie |
| Synchroniser token | Server issues a per session token, the form submits it, the server compares | Requires server state or a signed token, and it must be scoped per session |
| Double submit cookie | A token in a cookie is echoed in a header or body and compared | A subdomain that can set cookies on the parent domain can forge both halves. Sign the token to fix that |
| Custom request header on a JSON API | A cross-origin request carrying a custom header is not simple, so it triggers a preflight that your server can decline | Only holds if every state changing endpoint refuses requests without it, including any that accept form encoded bodies |

Two rules that make this simple: never make a state changing endpoint reachable
by GET, and set `SameSite` explicitly rather than relying on a default.

**Clickjacking.** An attacker frames your page transparently over their own
interface so a click lands on your control. The defence is `frame-ancestors` in
the policy header, with the legacy frame options header alongside it for older
clients. Framing defences are per response, so any endpoint that returns HTML
needs them, including error pages.

**Cross-origin resource sharing, and what it is not.** State this precisely
because it is the most misunderstood item in the module.

| Claim | Correct |
|---|---|
| It protects your API | No. It relaxes the browser's own restriction on reading cross-origin responses. A non browser client is unaffected |
| A wildcard origin is fine for public data | Yes, for genuinely public data, and it cannot be combined with credentials |
| Reflecting the request origin is a convenient shortcut | It is equivalent to a wildcard that also works with credentials, which is the worst configuration available |
| A preflight is a security control | It is a compatibility mechanism that also gives your server a chance to decline. Treat it as an opportunity, not a guarantee |

**Cookie attributes, each with the attack it addresses.**

| Attribute | Addresses |
|---|---|
| `Secure` | The cookie travelling over a plaintext connection |
| `HttpOnly` | Script reading the credential after an injection |
| `SameSite` | Cross-site request forgery, partially, per the table above |
| `__Host-` name prefix | A subdomain setting or overwriting a cookie the parent relies on |
| `Path` and `Domain` | Over-broad scope, which widens every other risk |
| Explicit `Max-Age` | Sessions living far longer than intended |

**Token storage, priced against the two threats.**

| Storage | Readable by script | Sent automatically | Consequence under injection |
|---|---|---|---|
| `localStorage` | Yes | No | Credential exfiltrated, usable off the machine until expiry |
| `sessionStorage` | Yes | No | Same, bounded by tab lifetime |
| In memory only | Yes, within the same context | No | Attacker acts while the page is open, cannot easily persist |
| `HttpOnly` cookie | No | Yes, so forgery defence is required | Attacker acts as the user through your page, cannot read the credential |

The honest summary: no storage choice survives script execution in your origin
unharmed. The choice determines whether an injection is an active session
compromise or a portable credential theft, and that is worth choosing
deliberately.

### 14d. Worked example

**The prompt.** "Our analytics product lets customers write dashboard
descriptions in markdown and embed an image by URL. Enterprise customers also
want to embed our dashboards in their own intranet pages. Design the browser
security posture."

Inputs, stated before I touch anything:

| Input | Value | Status |
|---|---|---|
| Untrusted content authored by | Customer users, rendered to other users in the same organisation | GIVEN |
| Session credential today | Signed token in localStorage | GIVEN |
| Third party scripts on the page | A support widget and an analytics tag | GIVEN |
| Customers wanting to frame our pages | About 40 enterprise accounts | GIVEN |
| Inline event handler attributes in our markup | Unknown | To be counted, and it sets the policy migration cost |

#### Block A. Purpose: draw the trust boundary before choosing any control

I state what executes in my origin and what it can reach, because every later
decision follows from this one picture.

| Code in my origin | Trust | Can reach |
|---|---|---|
| Our application code | Trusted | Everything on the origin |
| The support widget and analytics tag | Third party, and fully privileged | Everything on the origin, including the session credential |
| Markdown authored by a customer user | Untrusted, and currently rendered through a raw HTML sink | Everything, if it executes |

The uncomfortable row is the second. Two third party scripts already have the
same power as our own code, so any control I design must account for scripts I
did not write and cannot audit. This is the sentence that justifies the effort in
the later blocks.

**The error most readers make here.** Treating third party tags as outside the
threat model because they are from reputable vendors. Reputation is not a
boundary. The boundary is the origin, and they are inside it.

!!! tip "Self-explanation prompt"

    Before reading Block B, say which of the three rows would change if the
    markdown were rendered inside a cross-origin iframe, and what new problem that
    would create.

#### Block B. Purpose: fix the sink, because the policy cannot save a broken sink

Order of work, and the reason for the order:

1. Configure the markdown renderer to not pass through raw HTML. This removes the
   large majority of payloads at almost no cost.
2. Sanitise the rendered output with a vetted sanitiser before it reaches the DOM,
   because renderer configurations regress and a defence that depends on one
   option staying set is not a defence.
3. Constrain the image URL: allow only http and https schemes, and reject
   anything else. A raw URL from a user reaching an `src` attribute is a separate
   sink from the markdown body.
4. Enable Trusted Types with a single approved policy that runs the sanitiser, so
   any new code path that reaches a raw HTML sink fails loudly rather than
   silently shipping.

Step 4 is what makes this durable. Steps 1 to 3 fix today's code. Step 4 fixes
next year's code, written by someone who has not read this design.

**The error most readers make here.** Sanitising on write, when the comment is
stored. The same stored value will later be rendered into HTML, into an attribute
and possibly into a URL, and each context needs different treatment. Sanitise
where the context is known, which is the sink.

#### Block C. Purpose: reduce what a successful injection is worth

Even with the sinks fixed, I assume an injection eventually happens, because two
third party scripts are already privileged. So I reduce the payoff.

| Change | Effect | Cost |
|---|---|---|
| Move the session credential from localStorage to an `HttpOnly`, `Secure`, `SameSite=Lax` cookie with the `__Host-` prefix | An injection can act, but cannot exfiltrate a portable credential | Now needs a forgery defence, since the cookie is automatic |
| Add a custom header requirement on every state changing endpoint | Cross-origin requests carrying it are preflighted and declined | Every client must set it, including any server to server caller, which must be checked |
| Ensure no state changing endpoint accepts GET | Removes the `SameSite=Lax` gap | An audit, plus a lint or route level assertion to keep it true |
| Shorten token lifetime and add refresh | Bounds the window of a stolen session | A refresh flow, and care that the refresh endpoint itself is forgery resistant |

I note the trade explicitly: moving to cookies swaps an exfiltration risk for a
forgery risk, and forgery has better established defences. That is why the trade
is worth making, and saying it that way shows the reasoning rather than the
conclusion.

**The error most readers make here.** Adding the cookie and forgetting that the
credential is now attached automatically to every request the browser makes to
the origin, including ones initiated by other sites. The forgery defence is not
optional after this change, it is created by it.

!!! tip "Self-explanation prompt"

    Say why `SameSite=Strict` would be the wrong choice for this product, using
    the fact that customers link to dashboards from email and chat.

#### Block D. Purpose: ship a policy that is worth its header bytes

Target policy, with a reason for each part:

- `script-src` with a per response nonce plus `'strict-dynamic'`, so our bundler
  loaded chunks work and injected markup cannot execute.
- `object-src 'none'` and `base-uri 'self'`, both unconditional and both cheap.
- `require-trusted-types-for 'script'`, which pairs with Block B step 4.
- `form-action 'self'`, so an injected form cannot post user input elsewhere.
- `frame-ancestors` handled in Block E, because it is the enterprise embedding
  requirement rather than a policy detail.

Rollout, with the four phases and their durations from earlier in this module:
report only for 2 weeks, a triage week, fixes, then enforce. I set the
expectation up front that most report only volume will be browser extensions,
which we neither fix nor allowlist.

The migration cost I flagged as unknown in the inputs is now the thing to
measure first: count inline event handler attributes and inline script blocks in
our own templates. That count is the schedule.

**The error most readers make here.** Adding a nonce while leaving
`'unsafe-inline'` out for older clients and then panicking when a legacy browser
breaks. Keep `'unsafe-inline'` alongside the nonce: nonce aware browsers ignore
it, and older ones fall back to the weaker but functional behaviour.

#### Block E. Purpose: allow enterprise framing without allowing everyone

40 enterprise accounts want to frame our dashboards, so `frame-ancestors 'none'`
is not available. Three options, and I pick one:

| Option | Behaviour | Why I did or did not choose it |
|---|---|---|
| `frame-ancestors` listing every customer domain | Per response header built from the account's configured domains | Chosen. It is explicit, per account, and revocable |
| A wildcard for framing | Anyone can frame us | Rejected. It reintroduces clickjacking against every account to serve 40 |
| A dedicated embed endpoint on a separate origin, with a narrow interface and no session cookie | Framing is allowed only there, and the embedded view has its own scoped credential | The right answer at larger scale, and more work than the requirement justifies at 40 accounts. I name it as the next step |

Two additions that make the chosen option safe:

- Domains are configured per account and validated, not free text, because a
  header built from unvalidated customer input is an injection into a security
  control.
- Any interactive control inside the embedded view that performs a destructive
  action requires a confirmation step, because framing defences protect the
  accounts that configured correctly and a confirmation protects the rest.

**The error most readers make here.** Treating framing as a single global
setting. It is a per response decision and it can be per account, which is what
turns an impossible requirement into a configuration.

### 14e. Fade the scaffold

**Fade 1: a marketing site adding a tag manager.** The marketing team wants a
third party tag manager so they can add scripts without engineering involvement.
The site has a newsletter form and no login. Same five block shape, last block
blank.

- Block A, draw the trust boundary: a tag manager is a mechanism for injecting
  arbitrary script into your origin, controlled by whoever has access to its
  console. The trust boundary now includes that vendor's access control and every
  marketing person with a login.
- Block B, fix the sink: the sink here is the tag manager itself, and it cannot
  be sanitised. The equivalent move is to constrain what it can add: use the
  vendor's own restrictions where they exist, and put the tag manager on a
  separate subdomain so cookies scoped with the `__Host-` prefix are not shared.
- Block C, reduce what an injection is worth: there is no session credential, so
  the valuable target is the newsletter form input and any payment or lead data.
  `form-action 'self'` becomes a high value directive here, because the realistic
  attack is an injected form or an altered form action exfiltrating submissions.
- Block D, ship a policy: a nonce policy is in direct tension with a tag manager,
  because the whole product is inline injection. The honest options are to give
  the tag manager a nonce (which grants it full privilege, so the policy protects
  against everything except the largest risk) or to refuse the tag manager.
- Block E, the framing and embedding question: **you write this block.**

??? note "Show answer"

    The framing question inverts here, and noticing that inversion is the point.
    On a marketing site the risk is not that others frame you, it is small but
    real: a competitor or phishing page framing your pricing page to lend
    credibility, or a clickjacking overlay on the newsletter form to harvest
    submissions.

    Set `frame-ancestors 'none'` on every HTML response including error pages,
    with the legacy frame options header alongside. There is no embedding
    requirement, so this costs nothing, and unlike the worked example there is no
    per account exception to manage.

    The more interesting part of this block is the reverse direction: what the
    marketing site frames. Tag managers commonly add embedded video, chat widgets
    and survey tools, each of which is a third party iframe. Those are far safer
    than third party script because they run in their own origin, so the guidance
    to the marketing team is concrete and useful: prefer any integration offered
    as an iframe over the same integration offered as a script.

    Add the `sandbox` attribute with the narrowest set of permissions each embed
    actually needs, and `allow` only the capabilities it requires. Most embeds
    request more than they use, and nobody checks.

    Finally, the governance answer, which is the one that actually holds: require
    that any new tag goes through a change record with a named owner and a
    review, and enforce it by keeping the tag manager's publish permission with
    engineering while leaving edit permission with marketing. This is an
    organisational control for a risk that has no technical control, and saying so
    is more honest than pretending a policy header solves it.

**Fade 2: migrating an application from token in storage to cookie.** A shipped
single page application with a separate API origin, 200,000 daily users, moving
from a bearer token in localStorage to an HttpOnly cookie. Last two blocks blank.

- Block A, draw the trust boundary: the application origin and the API origin are
  different, which is the fact that makes this migration non trivial. A cookie
  set on the API origin is third party to the application in some browser
  configurations, so the migration may require moving the API under a shared
  parent domain.
- Block B, fix the sink: unchanged from the current posture, and worth auditing
  during the migration because the whole justification for the work is reducing
  injection payoff.
- Block C, reduce what an injection is worth: this is the migration itself.
  `HttpOnly`, `Secure`, `SameSite=Lax`, `__Host-` prefix where the path and
  domain constraints allow it, and a short lifetime with a refresh endpoint.
- Block D, ship a policy: **you write this block.**
- Block E, the cross-origin and forgery consequences: **you write this block.**

??? note "Show answer"

    Block D. The policy work is largely unchanged by the credential migration and
    should not be bundled with it, which is the first thing to say. Two risky
    migrations in one release makes every incident ambiguous.

    That said, one directive becomes more valuable after the migration and one
    becomes less. `form-action 'self'` becomes more valuable, because the
    credential is now attached automatically, so an injected form posting to an
    attacker's origin would carry it. Directives aimed at preventing exfiltration
    of storage contents become less relevant, because there is nothing valuable
    in storage any more.

    Sequence: ship the policy in report only before the credential migration, so
    the baseline of violations is established against the known good version. If
    you enforce a new policy and migrate credentials in the same week, every
    report is ambiguous.

    Block E. Two consequences, and the second is the one that catches teams.

    First, forgery defence is now mandatory, per the worked example. Every state
    changing endpoint needs the custom header requirement or a token, no state
    changing endpoint may be reachable by GET, and the refresh endpoint needs the
    same protection as everything else.

    Second, cross-origin credential handling. Because the application and API are
    on different origins, the browser will not attach the cookie to the API
    request unless the request is made with credentials included and the API
    responds with an explicit allowed origin (never a wildcard, which is
    prohibited with credentials) and the header that permits credentials. Every
    such request becomes preflighted, which adds a round trip.

    Price that round trip, because it is the hidden cost of the migration: at an
    assumed 150 ms round trip, every distinct preflighted endpoint pays 150 ms on
    its first call. The mitigation is a preflight cache duration set on the API
    responses, which trades a small revocation delay for removing the repeat cost.

    The alternative that removes the whole problem: move the API to a subdomain of
    the application's parent domain so requests are same-site. That is an
    infrastructure change and it is usually the right one, and it is worth naming
    even if this organisation cannot do it this quarter.

### 14f. Check yourself

**Q1.** A colleague argues that input validation on the server makes output
sanitisation unnecessary, since nothing dangerous is ever stored. Give the
technical reason they are wrong and one operational reason.

??? note "Show answer"

    Technical reason: dangerous is a property of the destination, not of the
    value. The same stored string may be rendered as HTML body text, into an
    attribute value, into a URL, into a JavaScript string literal and into a CSS
    value, and each of those contexts has a different escaping function.

    A validator at the input has no way to know which contexts a value will
    reach, so it either rejects legitimate content (an apostrophe in a name, a
    plus sign in a search) or lets through something that is harmless in one
    context and executable in another.

    A concrete case: a value that is perfectly safe as text becomes an attack when
    interpolated into an attribute without quoting, because it can close the
    attribute and add an event handler. No input validator can prevent that
    without banning ordinary characters.

    Operational reason: the input validator and the renderer change on different
    schedules and are usually owned by different people. A new renderer added next
    quarter inherits the assumption silently, and the assumption is invisible in
    the new code. Sanitising at the sink puts the control next to the risk, where
    the person adding the sink can see it.

    The strongest version of the response: do both, for different reasons. Input
    validation is for data quality and for limiting what enters your system.
    Output encoding is for safety, and only output encoding is a security control.

**Q2.** Your policy is enforcing with a nonce and strict-dynamic. An engineer
reports that a new feature does not work and asks to add `'unsafe-eval'`.
Diagnose and give two alternatives.

??? note "Show answer"

    The diagnosis is almost always one of three things. A templating or expression
    library that compiles strings into functions at runtime. A validation or query
    library that builds functions from schema strings. Or a bundler configuration
    producing a development build in production, since some development tooling
    evaluates code from strings.

    The first question is which of the three, because the third is a build
    configuration bug and not a policy question at all. Check the deployed bundle
    before discussing the policy.

    Alternative one: move the compilation to build time. Most template and schema
    libraries offer a precompile step precisely because of this constraint. This
    is usually a configuration change and it also improves startup cost, because
    runtime compilation is main thread work.

    Alternative two: replace the library with one that does not construct
    functions from strings. For validation and templating there is normally a
    close equivalent, and the size difference is usually small compared with the
    cost of weakening the policy for the entire origin.

    Why not just add the directive: `'unsafe-eval'` applies to the whole origin,
    not to the one feature that needs it, so it re-enables a class of payload
    everywhere in exchange for one feature. The scope mismatch is the argument,
    and it is the same argument as `'unsafe-inline'` in the opening artefact.

### 14g. When not to use this

**A strict nonce based content security policy.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You render content you did not author, or you load third party script, AND you can count the inline handlers in your own markup so the migration has a real estimate |
| Scale threshold below which it is over-engineering | A fully static site with no user content, no third party script and no authentication gains little from a nonce policy, and it cannot serve a per response nonce from a cached file without adding edge compute |
| Cheaper alternative | `object-src 'none'`, `base-uri 'self'`, `frame-ancestors`, and `form-action 'self'`. Four directives, no per response state, no migration, and they close real attacks |

**Trusted Types enforcement.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You have more than a handful of raw HTML sinks, or more than one team writing code that could add one, so an audit will not stay true |
| Scale threshold | In a small codebase with two known sinks and a lint rule that fails on them, enforcement adds a policy to maintain and a class of runtime failure to handle for little marginal safety |
| Cheaper alternative | A lint rule banning the dangerous sinks with an allowlist of reviewed exceptions, plus a single shared helper that sanitises. Most of the guarantee, none of the runtime surface |

**Moving a credential from storage into an HttpOnly cookie.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Your token is long lived or refreshable, so exfiltration yields lasting access, AND you have third party script on the page or user authored content, so injection is a live risk |
| Scale threshold | If tokens are short lived, narrowly scoped and the origin loads no third party script, the difference between the two options shrinks and the migration cost (forgery defence, cross-origin credential handling, preflights) may exceed the gain |
| Cheaper alternative | Keep the token in memory only, never in persistent storage, and re-obtain it on load from a same-site session cookie. You get the non-portability benefit without changing every request path |

**A separate embed origin for third party framing.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Enough embedding customers that maintaining a per account domain allowlist is operationally painful, or embedded views that need a credential you do not want to be the full session |
| Scale threshold | At a few dozen accounts, a per response `frame-ancestors` header built from validated per account configuration is simpler, revocable and adequate |
| Cheaper alternative | Per account `frame-ancestors` with validated domains, plus a confirmation step on destructive actions inside the embedded view |

---

## Module 15. Internationalisation

**Time: 30 to 40 minutes reading, 20 to 30 minutes practice.**

Internationalisation appears in frontend rounds as a constraint on an otherwise
familiar design, and it is a good discriminator because the wrong answers are
confidently wrong. Putting strings in files is the obvious half of the problem.
What a plural rule does to a component API is the half that decides the design.

### 15a. Predict first

Read this artefact and commit to an answer before continuing.

```
+---------------------------------------------------------------+
| Message assembly:                                             |
|   t("You have ") + count + t(" new messages")                 |
| Locales shipped: en, de, ar, pl, ja                           |
| Bundle: all 5 locales, all 800 messages, in the main chunk    |
| Dates rendered with: date.toLocaleDateString()                |
| Future events stored as: 2026-11-01 09:30 with offset +01:00  |
+---------------------------------------------------------------+
Caption: four internationalisation decisions in one screen of a
shipped application.
```

**Question.** Name a distinct defect in each of the four lines, and put a number
on the bundle one.

??? note "Show answer"

    Assembly by concatenation. Three defects at once. Word order differs by
    language, so a translator cannot reorder the number relative to the words.
    Plural agreement is impossible: Arabic has six plural categories and Polish
    has four, so no pair of fixed fragments can be correct for all counts. And the
    translator sees "You have " with no context, which produces wrong translations
    even where the grammar happens to work.

    All locales in the main chunk. Assume 800 messages at an average 60 bytes,
    which gives about 48 KB per locale before compression. Five locales is about
    240 KB where a user needs 48 KB, so about 192 KB is waste. At 200 KB/s that is
    roughly 0.96 s of transfer, plus parse cost, paid by every user.

    Formatting without an explicit locale. `toLocaleDateString()` with no argument
    uses the runtime's locale, not the locale the user selected in your
    application. A user who set the interface to German on an English configured
    device gets English dates inside a German page, which looks like a bug and is
    hard to reproduce on a developer machine.

    Storing an offset for a future event. An offset is a fact about a moment, not
    a rule about a place. If the zone transitions between now and November, or a
    government changes the rules, the stored offset is wrong and the event moves.
    Store the instant in UTC for the past, and local wall time plus a zone
    identifier for the future.

### 15b. Messages, plurals and why concatenation cannot be patched

**A message format** is a small language for describing a translatable string
with placeholders, plural selection and gender selection inside it, so the
translator controls the whole sentence.

The critical property is that selection lives inside the message, not in the
calling code. The moment your component decides which of two strings to use, you
have hard-coded a two-category plural rule into logic that cannot be translated.

**Plural categories, which is the fact that settles the argument.** The Unicode
locale data defines up to six categories: zero, one, two, few, many, other.
Languages use different subsets.

| Language | Categories used | Count |
|---|---|---|
| Japanese, Chinese, Korean | other | 1 |
| English, German | one, other | 2 |
| Polish | one, few, many, other | 4 |
| Russian | one, few, many, other | 4 |
| Arabic | zero, one, two, few, many, other | 6 |

Two consequences that shape the API:

- A component that accepts `singular` and `plural` props has a two category
  assumption in its signature. It cannot be fixed by translation, only by an API
  change.
- The categories are not about the number's size. Polish uses "few" for counts
  ending in 2, 3 or 4 except in the teens, so the rule is arithmetic on the
  number, which is exactly why it belongs in locale data rather than in your code.

**Markup inside translated text.** Never split a sentence to wrap part of it in a
link or bold. Put a tag placeholder in the message and let the rendering layer
supply the element. Splitting produces the same word order problem as
concatenation and it produces translation fragments no translator can work with.

**Context and disambiguation.** The same source word can need different
translations, for example a noun and a verb spelled identically in English.
Message identifiers should carry a context note, and the extraction tooling
should surface it. This is cheap to build in and impossible to retrofit once
translators have delivered.

### 15c. Direction, and what mirrors

**Definitions.** A right to left locale renders text from right to left and, by
convention, mirrors the whole interface layout. **Logical properties** describe
positions relative to the reading direction (start and end) instead of the screen
(left and right).

The mechanical change is to stop naming physical sides.

| Physical | Logical |
|---|---|
| `margin-left` | `margin-inline-start` |
| `padding-right` | `padding-inline-end` |
| `left: 0` | `inset-inline-start: 0` |
| `text-align: left` | `text-align: start` |
| `border-left` | `border-inline-start` |

Set `dir` on the document from the active locale and the browser does the rest
for anything expressed logically. That is the whole technique, and it is why
adopting logical properties before you have a right to left locale is cheap
insurance rather than speculative work.

**What does not mirror, which is the part people get wrong.**

| Element | Mirrors | Reason |
|---|---|---|
| Layout, navigation, icons that imply direction (back, next) | Yes | They follow reading order |
| Numbers and their digits | No | Digits render left to right within the number |
| Phone numbers, credit card numbers, currency amounts | No | Structured numeric sequences keep their order |
| Media playback timelines and progress | Usually no | Time is conventionally rendered left to right regardless of locale |
| Charts with a time axis | Usually no | The convention follows the data, and mirroring confuses more than it helps |
| Logos and brand marks | No | They are images with a fixed form |
| Code and terminal output | No | Code is left to right |

**Bidirectional isolation.** When a user supplied name in one direction is
embedded in a sentence in the other, the text can visually reorder in ways that
change meaning. Wrap user supplied strings in an isolation element or apply
isolation styling. This is a two character fix for a class of bug that is very
hard to diagnose from a screenshot.

### Formatting, time and bundle shape

**Use the platform formatters, and always pass the locale explicitly.** The
browser ships locale data for number, currency, date, time, relative time,
sorting, list joining and plural rules. Reimplementing any of these is both
larger and less correct.

| Need | Formatter |
|---|---|
| Numbers, percentages, currency, units | Number formatting with explicit locale and options |
| Absolute dates and times | Date and time formatting with an explicit time zone |
| "3 days ago" | Relative time formatting, which handles the category rules |
| Sorting names or strings | Collation, because code point order is wrong in most languages |
| "a, b and c" | List formatting, which differs by locale in separator and conjunction |
| Choosing a plural category | Plural rules, if you are building the message layer yourself |

**Time, stated as three rules.**

1. A past or fixed instant is stored as an instant in UTC. Rendering applies a
   zone at display time.
2. A future event tied to a place is stored as local wall time plus a zone
   identifier, never as an offset, because the offset for that place on that date
   is a prediction that can change.
3. A recurring event is stored as a rule plus a zone, and expanded at read time.
   Materialising future occurrences bakes in offsets that may change.

The single test question that catches most errors: if a government changes its
rules next year, does the stored value still mean the same thing? An instant does
for the past. An offset does not for the future.

**Bundle shape, with the arithmetic.** Take the artefact's 800 messages at
about 60 bytes each, so roughly 48 KB per locale.

| Strategy | Bytes the user downloads | Requests | Note |
|---|---|---|---|
| All locales in the main chunk | 240 KB for 5 locales | 0 extra | The waste is 4 locales the user will never read |
| One chunk per locale | 48 KB | 1 extra | The obvious fix, and it makes the locale a build output |
| Per locale and per route | Say 6 KB for the initial route | 1 extra, more on navigation | Worth it when messages are large relative to code |
| Per locale, loaded at the edge into the document | 48 KB, no extra request | 0 extra | Removes the round trip, at the cost of an uncacheable or varied document |

At 5 locales, splitting saves 192 KB, about 0.96 s of transfer. At 40 locales the
unsplit option is 1.9 MB and the question stops being interesting. The threshold
where splitting starts to pay is roughly where the saved bytes exceed the round
trip cost, which by the arithmetic in Module 11 is around 30 KB, or roughly two
locales' worth of messages here.

**Fallback chain.** Define it explicitly: requested locale, then the base language
without the region, then the default. A missing message renders the fallback, and
in development it should render loudly so it is caught before release.

**Pseudo-localisation for layout.** Generate a fake locale that expands every
string by a fixed factor and adds accents, then run your visual tests against it.
A 35 percent expansion is a reasonable planning figure because several European
languages run substantially longer than English for short interface strings.
Measure your own from delivered translations once you have them.

### 15d. Worked example

**The prompt.** "Our product ships in English only. We are launching in Germany,
Poland, Japan, Saudi Arabia and Brazil in two quarters. Design the front end
work."

Inputs, stated before I touch anything:

| Input | Value | Status |
|---|---|---|
| Messages in the product | about 800 | GIVEN |
| Current message mechanism | A flat key to string map, with runtime concatenation in about 60 places | GIVEN |
| Locales at launch | en, de, pl, ja, ar, pt-BR | GIVEN, so 6 including English |
| Right to left locales | 1 (Arabic) | DERIVED from the list |
| Average message size | 60 bytes | ASSUMED, typical for interface strings. Measure yours from the existing file |
| Stylesheet declarations naming a physical side | Unknown | To be counted, it sets the direction work estimate |

#### Block A. Purpose: separate the work that is per locale from the work that is once

I split the project in the first two minutes, because the two halves have
completely different shapes and completely different owners.

| Work | Type | Scales with |
|---|---|---|
| Message format migration, removing concatenation | Once | Number of call sites, not number of locales |
| Logical properties in the stylesheet | Once | Size of the stylesheet |
| Direction handling and bidi isolation | Once, triggered by having any right to left locale | Number of components with directional layout |
| Formatter adoption, explicit locale everywhere | Once | Number of formatting call sites |
| Bundle splitting by locale | Once | Nothing, it is a build change |
| Translation and review | Per locale | Number of locales x number of messages |
| Layout verification | Per locale, cheaply for most, expensively for Arabic | Number of locales |

The useful conclusion: five of the seven rows do not get more expensive with more
locales. So the estimate for six locales is close to the estimate for two, and
the argument for doing the once work properly is stronger than it looks.

**The error most readers make here.** Estimating the project as translation
management. The translation vendor's work is real and it is not the engineering
project. The engineering project is the 60 concatenation sites and the stylesheet.

!!! tip "Self-explanation prompt"

    Before reading Block B, say which single row would change most if Arabic were
    dropped from the launch list, and whether you would still do that work.

#### Block B. Purpose: fix the message layer, since it constrains everything downstream

The 60 concatenation sites are the critical path, because a translator cannot
start on a fragment and because Polish and Arabic make fragments unfixable.

The migration, and why in this order:

1. Introduce a message format that supports plural and selection, with one
   message per user visible sentence.
2. Convert the 60 concatenation sites. Each becomes one message with a
   placeholder and, where a count is involved, a plural selection.
3. Audit component APIs for two category assumptions: any prop pair named
   singular and plural, any conditional on `count === 1`. These are the sites that
   look correct in English and are unfixable by translation.
4. Add context notes to ambiguous keys before extraction, because retrofitting
   context after the first delivery means re-reviewing everything.

I size it: 60 sites at roughly 20 minutes each including review is about 20
hours, plus the component API audit which is the uncertain part. I would ask for
a count of components accepting singular and plural props before committing.

**The error most readers make here.** Converting the messages and leaving the
`count === 1` conditionals in components. They are the same defect in a different
file, and they are invisible in English, German and Portuguese while being wrong
in Polish and Arabic.

#### Block C. Purpose: make direction a build in property rather than a variant

Arabic makes direction real. The cheap approach is to convert the stylesheet to
logical properties and set `dir` from the locale, and the expensive approach is a
mirrored stylesheet variant.

I take the first, and I state the work:

- Count declarations naming a physical side. This is a mechanical scan, and it is
  the estimate.
- Convert them. Most convert one for one. The ones that do not are the interesting
  cases: absolutely positioned decorations, transforms and shadows with a
  horizontal offset, and anything with a hard-coded direction in an image.
- Handle the non mirroring list from earlier: numbers, phone numbers, media
  timelines, charts with a time axis, logos and code. Each needs an explicit
  direction so it is not mirrored by the ancestor.
- Add bidi isolation around every user supplied string rendered inside a
  sentence: names, file names, search terms.

The result is that adding a second right to left locale later costs nothing,
which is the argument for doing it this way rather than shipping a mirrored
variant stylesheet.

!!! tip "Self-explanation prompt"

    Say why bidi isolation is needed even in the English interface, using a user
    whose display name is written in Arabic.

**The error most readers make here.** Mirroring everything by applying a
transform to the whole page. It reverses text that must not reverse, breaks
images, and produces a result that is confidently wrong rather than obviously
wrong, which is harder to catch in review.

#### Block D. Purpose: cut the bundle so 6 locales cost what 1 costs

Arithmetic from the stated inputs: 800 messages at 60 bytes is about 48 KB per
locale, 6 locales is about 288 KB if bundled together, and each user needs 48 KB.

- Split one chunk per locale: the user downloads 48 KB rather than 288 KB, saving
  240 KB, about 1.2 s at 200 KB/s.
- The extra round trip costs about 150 ms, so the split is clearly positive here.
  I check it rather than assuming it, because at 3 small locales it would not be.
- Load the locale chunk as early as possible, ideally as a preload declared in the
  document, since first render usually needs messages and a late request creates
  a waterfall.

I decline per route splitting at this size and say why: 48 KB per locale is
already small relative to the application bundle, and splitting further adds
navigation time round trips for a few kilobytes each, below the break even from
Module 11.

**The error most readers make here.** Splitting by locale and then loading the
chunk from application code after hydration, which puts a network round trip
between first paint and readable text. The request must start with the document.

#### Block E. Purpose: verify layout per locale without hiring six reviewers

Three mechanisms in increasing cost, and I use all three because they catch
different things.

| Mechanism | Catches | Cost |
|---|---|---|
| Pseudo-localisation at a 35 percent expansion in the visual regression suite | Truncation, overflow, fixed width containers, wrapping | Runs on every build, no translators needed |
| Visual regression in Arabic on a chosen set of screens | Direction defects, unmirrored elements, bidi problems | One extra suite run per build |
| Human review per locale on the top screens | Tone, terminology, anything culturally wrong | Per locale, per release, so it must be a small set |

I bound the last one explicitly: the 20 screens covering the most traffic, once
per release, not the whole product. An unbounded review commitment is the thing
that quietly stops happening in quarter three.

**The error most readers make here.** Waiting for real translations to test
layout. Pseudo-localisation finds the same truncation bugs weeks earlier, with no
dependency on the vendor's schedule, and it is a build configuration rather than
a process.

### 15e. Fade the scaffold

**Fade 1: a numbers heavy analytics product.** Dashboards full of currency,
percentages, large numbers and dates, launching in the same six locales. Same
five block shape, last block blank.

- Block A, separate per locale from once work: the message layer matters less
  here (dashboards have few sentences) and the formatter layer matters far more.
  The once work is dominated by finding every place a number, currency or date is
  turned into a string.
- Block B, fix the layer that constrains the rest: not messages, formatters. Every
  formatting site must take an explicit locale, an explicit currency where
  relevant, and an explicit time zone. Grouping separators, decimal separators and
  currency placement all differ, and a hand-rolled formatter will be wrong in at
  least one launch locale.
- Block C, direction: less layout work than the worked example (charts and tables
  mostly do not mirror) and more numeric care. Numbers keep their direction inside
  a right to left layout, table columns of numbers should stay aligned by decimal,
  and axis labels need explicit direction.
- Block D, bundle: message bytes are small here, so per locale splitting saves
  little. The bytes that matter are locale data for formatting, and the platform
  already ships that, which is a strong argument against a formatting library.
- Block E, verification per locale: **you write this block.**

??? note "Show answer"

    Pseudo-localisation is nearly worthless here and saying so is the point.
    Expanding string length by 35 percent tests wrapping and truncation, and this
    product's risk is not string length, it is numeric formatting correctness.

    The verification that matches the risk is a formatting fixture suite: a table
    of representative values (a negative currency amount, a value above the
    grouping threshold, a fractional percentage, a date near a zone transition, a
    date in the first days of a month, a large number in a locale that groups
    differently) rendered in every launch locale, asserted against expected output.
    That is roughly 6 locales x 10 values = 60 assertions, cheap to write and
    stable.

    Add one verification the worked example did not need: a check that no
    dashboard renders a number through string interpolation. A lint rule against
    template literals containing a numeric variable near a currency symbol is
    crude and catches the common regression.

    Layout verification narrows to the specific failure this product has: numeric
    columns getting wider in locales with different grouping and currency
    placement, which breaks fixed width table columns. Run the visual suite with a
    fixture of long formatted values rather than with expanded text.

    Human review stays, and its focus changes: not tone, but whether the number
    conventions look right to someone who reads that locale daily. A number that
    is technically correctly formatted and unconventional for the audience is a
    real defect that no assertion catches.

**Fade 2: adding one right to left locale to an otherwise finished product.** The
product already handles 8 left to right locales correctly, with a proper message
layer, formatters and per locale bundles. Only Hebrew is being added. Last two
blocks blank.

- Block A, separate per locale from once work: almost everything per locale is
  already solved. The whole project is the once work triggered by having any right
  to left locale, which was deferred and is now due.
- Block B, fix the constraining layer: the stylesheet. Count declarations naming a
  physical side, and expect this count to be the schedule. In a mature product
  with a design system, most of these live in the design system, which is good
  news: fix them once and every consumer benefits.
- Block C, direction: **you write this block.**
- Block D, bundle: **you write this block.**

??? note "Show answer"

    Block C. The work divides into three passes, and the third is the one teams
    forget.

    Pass one, mechanical: convert physical properties to logical ones, set `dir`
    on the document root from the locale, and let the browser mirror. In a design
    system this is concentrated and reviewable.

    Pass two, the exceptions from the non mirroring table: numbers, phone numbers,
    monetary amounts, media timelines, charts with a time axis, logos, code blocks
    and terminal output. Each needs an explicit direction so it does not inherit
    mirroring from an ancestor. Add these as a checklist to the design system's
    component documentation, because the next right to left locale will be added
    by someone who was not here.

    Pass three, the one that gets forgotten: anything positioned by JavaScript
    rather than by stylesheet. Popover and tooltip placement, drag and drop
    offsets, virtualised list horizontal scroll, canvas rendering and any code
    reading `getBoundingClientRect` and comparing to a hard-coded side. These do
    not respond to `dir` and each needs individual attention. This pass is usually
    larger than pass one in a mature product.

    Plus bidi isolation on user supplied strings, which the product needed already
    for Hebrew names appearing in its English interface.

    Block D. Almost nothing, and recognising that is the answer.

    Per locale bundles already exist, so Hebrew is one more chunk of roughly the
    same size as the others, downloaded by Hebrew users only. There is no new
    splitting decision.

    Two small things do change. First, if a font is being loaded for the interface,
    the existing font may not cover Hebrew, so a script specific font subset is
    needed with a unicode range so left to right users do not download it. Price
    it: a Hebrew subset is small, tens of kilobytes, and without the range
    restriction every user in every locale pays for it.

    Second, the direction specific stylesheet, if any survives pass one, should not
    ship to left to right users. If the logical properties conversion is complete
    there is no such stylesheet, which is the argument for finishing pass one
    rather than shipping a mirrored variant.

    The wider point: in a product that did the once work, adding a locale is a
    content operation. In a product that did not, every locale is a project. That
    difference is the entire case for the earlier modules' insistence on doing the
    once work first.

### 15f. Check yourself

**Q1.** A component in your library accepts `singularLabel` and `pluralLabel`
props and is used in 120 places. Explain what is wrong, and design the migration.

??? note "Show answer"

    The API encodes a two category plural rule, which is correct for English and
    German and wrong for Polish (four categories) and Arabic (six). No translator
    can fix it, because the component only ever asks for two strings. It is an API
    defect, not a translation defect.

    The replacement takes a single message identifier and a count, and resolves
    the category using the active locale's plural rules inside the message layer.
    The caller no longer decides anything about grammar.

    Migration, using the deprecation approach from Module 12. Ship the new prop
    alongside the old ones so nothing breaks. Write a codemod: the transformation
    is mechanical for call sites passing two literals, since both strings and the
    key can be derived, and 120 call sites is above the roughly 40 site threshold
    where a codemod pays. Sites passing computed values need a human.

    Then, and this is the part that makes it stick, add a lint rule that fails on
    the old props so the count only goes down. Without it, a 120 site migration
    finishes at about 90 and stays there.

    One honest caveat to state: until every call site migrates, those screens are
    wrong in Polish and Arabic. That is a launch blocker for those locales rather
    than a background task, and it should be scheduled as one.

**Q2.** Your team proposes shipping all 12 locales in one bundle "to avoid a
round trip on first paint". Evaluate with numbers, then give the option they have
not considered.

??? note "Show answer"

    The concern is legitimate: a locale chunk requested by application code after
    hydration does sit between first paint and readable text, and a round trip at
    an assumed 150 ms is real.

    But price the alternative. At 800 messages and 60 bytes each, one locale is
    about 48 KB, so 12 locales is about 576 KB. The user needs 48 KB, so 528 KB is
    waste: about 2.6 s of transfer at 200 KB/s plus parse cost, paid by every user
    on every cold load, to avoid a 150 ms round trip. That is roughly a 17 to 1
    loss.

    The option they have not considered: keep the split and remove the round trip
    rather than the split. Declare the locale chunk as a preload in the document
    head so the request starts with the document rather than after hydration. The
    round trip then overlaps with the rest of the critical path and costs close to
    nothing.

    A second option if the document is server rendered: inline the messages for
    the first route into the document itself, and load the rest as a chunk. The
    user gets zero extra requests for first paint and still does not download 11
    unused locales. The cost is that the document varies by locale, so the cache
    key must include it, which is acceptable for a small fixed set of locales.

    The generalisable move: when someone proposes undoing a split to avoid a
    waterfall, check whether the waterfall can be removed by starting the request
    earlier. It usually can, and it is much cheaper than the bytes.

### 15g. When not to use this

**A full message format runtime with plural and selection support.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You ship, or have a dated plan to ship, a locale whose plural categories exceed two, OR you have messages with grammatical gender or number agreement |
| Scale threshold below which it is over-engineering | With English only and under a few dozen strings, a flat key to string map with placeholder substitution is enough, and the parser plus locale rule data is bytes for a capability with no consumer |
| Cheaper alternative | A flat map with named placeholders and a lint rule banning string concatenation of translatable text. It keeps the door open at almost no cost |

**Per locale bundle splitting.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Total message bytes across all locales exceed what one locale needs by more than roughly 30 KB compressed, which is the round trip break even from Module 11 |
| Scale threshold | At two or three locales with small message files, splitting adds a request and a build output for a saving smaller than the round trip. Below about two locales' worth of messages, do not |
| Cheaper alternative | Ship all locales inline and revisit at the fourth locale. Set a byte budget on the messages file so the decision is triggered by data rather than by memory |

**Right to left support.**

| Question | Answer |
|---|---|
| Measurement that justifies it | A right to left locale is on a roadmap with a date, OR you render user supplied text that may be right to left, which is true of almost any product with names |
| Scale threshold | Full mirroring, exception handling and JavaScript positioning audits are a real project. With no right to left locale planned, doing the full project now is speculative |
| Cheaper alternative | Two cheap things that are worth doing regardless: use logical properties in new code from today, and apply bidi isolation to user supplied strings. Both cost nothing now and remove most of the future project |

**A third party formatting library for numbers and dates.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You need a capability the platform formatters genuinely lack, such as complex calendar arithmetic or parsing arbitrary user typed dates, and you can name it |
| Scale threshold | For display formatting of numbers, currencies, dates, relative times, lists and sorting, the platform already ships the locale data. Adding a library duplicates it in bytes and lags it in correctness |
| Cheaper alternative | The platform formatters with an explicit locale, an explicit time zone, and a thin wrapper of your own so options are consistent across the application |

---

## Module 16. Testing and rollout

**Time: 35 to 45 minutes reading, 25 to 35 minutes practice.**

Testing questions in a frontend round are rarely "do you write tests". They are
"what would you test, at which level, and what would you refuse to test", and the
refusal is where the signal is. Rollout is the same question one step later: how
does a change reach users in a way you can stop.

### 16a. Predict first

Read this artefact and commit to an answer before continuing.

```
+---------------------------------------------------------------+
| Continuous integration suite, measured over one month:        |
|                                                               |
|   unit           4,000 tests    2 min    flake near zero      |
|   integration      600 tests    6 min    flake 0.1% per test  |
|   end to end       300 tests   34 min    flake 0.5% per test  |
|                                                               |
| Policy: any red run blocks the merge queue, rerun on red.     |
| The team opens 40 pull requests per day.                      |
+---------------------------------------------------------------+
Caption: suite sizes, durations and per test flake rates, with
the merge policy that consumes them.
```

**Question.** Compute the probability that one run is green when the code is
perfect, then compute the machine time the merge queue spends per day, and say
which lever fixes it fastest.

??? note "Show answer"

    End to end: probability all 300 pass is 0.995^300. Taking logs, 300 x
    ln(0.995) = 300 x (-0.005013) = -1.504, so the probability is e^(-1.504) =
    0.222.

    Integration: 0.999^600 = e^(600 x -0.001) = e^(-0.600) = 0.549.

    Combined: 0.222 x 0.549 = 0.122. About 12 percent of runs are green on
    perfect code, so 88 percent of red builds are false.

    Expected attempts to reach one green run: 1 / 0.122 = 8.2. At 42 minutes per
    full run, one merge costs 8.2 x 42 = 344 minutes of machine time. Across 40
    pull requests that is 13,760 minutes, about 229 machine hours per day, and a
    developer wait measured in hours per merge.

    The fastest lever is not fixing flakes, it is deleting end to end tests. At 30
    end to end tests instead of 300, the pass probability becomes 0.995^30 = 0.860,
    and combined with integration it is 0.860 x 0.549 = 0.472. Attempts drop from
    8.2 to 2.1.

    Halving the flake rate instead gives 0.9975^300 = 0.472 for the end to end
    tier, combined 0.259, so 3.9 attempts. Better, and it is far more work than
    deleting 270 tests, and the flake rate will drift back up.

    The transferable point: suite reliability compounds multiplicatively with test
    count, so the count is a first class design decision and not a measure of
    diligence.

### 16b. What belongs at each level in a user interface

Define the levels by what they instantiate, since that is what determines their
cost and their flake rate.

| Level | Instantiates | Belongs here | Does not belong here |
|---|---|---|---|
| Unit | One function or module, no DOM | Reducers, state machines, formatters, URL parsing, cache key construction, validation rules | Anything whose correctness depends on rendered output |
| Integration | Real components in a real DOM, network mocked at the transport boundary | The large majority of interface behaviour: forms, keyboard interaction, loading and error states, focus management | Cross service behaviour, real payment providers |
| End to end | The whole system, real network, real browser | A small number of revenue critical or irreversible journeys | Anything you could have asserted one level down |

**The mock boundary matters more than the level names.** Mock at the transport
boundary (the request and response) rather than at your own module boundary. A
test that stubs your own data access layer passes when that layer is wrong.

**Query through the accessible interface.** Find elements by role and accessible
name rather than by test identifier or class. Two benefits for the price of one:
the test survives refactoring, and it fails when the accessible name is missing,
which turns a large share of your integration suite into an accessibility
assertion at no extra cost.

**A useful ratio to be able to defend.** For a mature interface, the shape that
survives is roughly: many unit tests, a substantial integration suite that is the
real safety net, and end to end tests numbering in the tens. The artefact above
shows what happens when the last number reaches the hundreds.

### 16c. Contract testing, visual regression, accessibility checks

**Contract testing** verifies that the shape you mock in tests still matches what
the server actually returns.

The problem it solves is mock drift: your integration tests mock a response, the
server changes a field, every test still passes, and production breaks. The mock
was a copy of a truth that has moved.

| Approach | How it works | Cost |
|---|---|---|
| Schema derived mocks | Mocks are generated from the server's published schema, so a schema change breaks the build | Requires a machine readable schema kept current |
| Consumer driven contracts | The consumer publishes its expectations, the provider's build verifies them | Requires provider cooperation and a broker, so it is an organisational commitment |
| Recorded fixtures refreshed on a schedule | Real responses captured and replayed, refreshed nightly | Cheapest to start, and it detects drift a day late |

Pick by whether you control the provider. If you do, schema derived mocks are
usually enough. If you do not, recorded fixtures with a scheduled refresh is the
realistic option.

**Visual regression, priced.** Assume 60 components, 3 viewports, 2 themes.

- Images per run: 60 x 3 x 2 = 360
- At 20 builds per day: 7,200 renders per day, a real but affordable compute cost
- The dominant cost is human: at an assumed 2 percent of images differing for
  reasons unrelated to the change, 360 x 0.02 = about 7 images to review per
  build, every build

That 7 is the number that kills visual regression suites. Below a threshold
nobody reviews them and everyone approves. Three controls, in order of value:

| Control | Effect |
|---|---|
| Run only on components touched by the diff | Cuts images per run by an order of magnitude on a typical change |
| Freeze the sources of non determinism: fonts loaded before capture, animations disabled, scrollbars hidden, time frozen | Removes most of the 2 percent |
| A per image tolerance set from measured noise, not from taste | Stops the argument about whether a one pixel change matters |

**Accessibility in continuous integration, honestly.** Automated rule sets check
the machine checkable subset: missing accessible names, insufficient contrast
against a computed background, invalid roles, duplicate identifiers, missing form
labels. They cannot check whether the focus order makes sense, whether an error
message is announced at the right moment, or whether a custom widget's
interaction model matches user expectations.

So run both, and know what each is for:

| Check | Catches | Runs |
|---|---|---|
| Automated rule set on rendered output | The machine checkable subset, cheaply and on every build | Every integration test, as an assertion |
| Keyboard path assertions written by hand | Focus traps, focus restoration, roving focus, escape handling | Per interactive component, as ordinary integration tests |
| Manual review with a screen reader | Everything else, which is most of the meaningful problems | Per release for changed flows, not per build |

The middle row is the highest value and the most neglected. A test that opens a
dialog, asserts focus moved into it, presses escape, and asserts focus returned
to the trigger is ten lines and catches the defect that automated rules never
see.

### Rollout: flags, canaries and kill switches

**Three kinds of flag, with different lifetimes.** Conflating them is why flag
systems become unmaintainable.

| Kind | Purpose | Lifetime | Removal |
|---|---|---|---|
| Release flag | Decouple deploying code from enabling a feature | Weeks | Removed as soon as the feature is fully on. Give it an expiry date at creation |
| Experiment flag | Randomised assignment for measurement | The experiment | Removed with the experiment's conclusion |
| Operational kill switch | Turn something off during an incident | Permanent | Never removed, and tested on a schedule so it works when needed |

**Canary sizing, derived.** For a two sided comparison at 5 percent significance
and 80 percent power, the samples needed per arm are approximately
n = 16 x variance / (effect size squared). For a rate p the variance is
p x (1 - p).

Case one, detecting a 1 percentage point drop in a 20 percent conversion rate:

- variance = 0.2 x 0.8 = 0.16
- effect = 0.01, squared = 0.0001
- n = 16 x 0.16 / 0.0001 = 25,600 per arm

Case two, detecting a doubling of a 0.5 percent session error rate:

- variance = 0.005 x 0.995 = 0.004975
- effect = 0.005, squared = 0.000025
- n = 16 x 0.004975 / 0.000025 = 3,184 per arm

**The conclusion that changes how you design a canary.** Error rate is detectable
in about 3,200 sessions per arm, which on a busy site is minutes. Conversion at
one percentage point needs 25,600 per arm, which is hours, and a 0.2 point change
would need 640,000 per arm.

So a canary is an error and latency detector, not a conversion detector. Gate the
automatic abort on technical guardrails that move fast and loudly. Measure
conversion with a proper experiment that runs long enough. Teams that gate a
canary on conversion either wait too long to roll out or abort on noise.

**Kill switch requirements, which are more specific than they sound.**

| Requirement | Reason |
|---|---|
| Takes effect without a deploy | During an incident your deploy pipeline may be the thing that is broken |
| Reaches running clients within a bounded time, say 60 s | A single page application may not reload for hours |
| Has a safe default when the configuration service is unreachable | Otherwise the kill switch's dependency becomes a new single point of failure |
| Is exercised on a schedule | An untested switch is a belief, not a control |

### 16d. Worked example

**The prompt.** "We rewrote the checkout form. It is the highest value flow in
the product. Design the testing and rollout plan."

Inputs, stated before I touch anything:

| Input | Value | Status |
|---|---|---|
| Sessions reaching checkout per day | 120,000 | GIVEN |
| Conversion rate at this step | 20 percent | GIVEN |
| Session level JavaScript error rate today | 0.5 percent | GIVEN |
| Existing end to end suite | 300 tests, 34 minutes, 0.5 percent flake each | GIVEN |
| Payment provider | Third party, with a sandbox | GIVEN |
| Acceptable revenue loss during rollout | Not stated | To be asked, it sets the canary size and duration |

#### Block A. Purpose: decide what must be true before any of this reaches a user

I start from the failure I am most afraid of, not from a test pyramid diagram.
For checkout that is: a user can enter valid details and cannot complete the
purchase, and we do not find out for hours.

That fear implies three requirements, and everything else follows from them:

| Requirement | Consequence |
|---|---|
| The failure must be detectable in minutes, not hours | Rules out conversion as the primary signal, per the sizing arithmetic |
| The change must be reversible in seconds | Requires a kill switch, not a rollback |
| The exposed population must be small until it is not | Requires percentage based exposure with an abort rule |

Saying this first reframes the answer from a list of test types into a design
with a stated objective, which is the difference between a mid and a senior
answer to this exact prompt.

**The error most readers make here.** Opening with the test pyramid. The pyramid
is a means. Starting from the feared failure produces a plan you can defend and
also produces the test selection for free.

!!! tip "Self-explanation prompt"

    Before reading Block B, say which of the three requirements would relax if
    checkout were an internal tool used by 30 staff, and by how much.

#### Block B. Purpose: put each assertion at the cheapest level that can hold it

I go through the flow and assign each behaviour to a level, with a reason.

| Behaviour | Level | Reason |
|---|---|---|
| Card number formatting, expiry validation, postcode rules per country | Unit | Pure functions with many cases. Hundreds of cases cost seconds |
| Field level error display, error announcement to assistive technology, focus moving to the first invalid field | Integration | Needs a real DOM, does not need a real server |
| Loading, error and retry states for the payment call | Integration, transport mocked | The interesting cases are server responses you cannot reliably produce for real |
| Address autocomplete keyboard model | Integration | Real DOM, mocked suggestions |
| Full purchase against the provider sandbox | End to end, exactly one test | Irreversible and cross system. One is enough to prove the wiring |
| Purchase with a declined card | End to end, one test | The second most valuable path, and it exercises the error branch across systems |

Two end to end tests, not twenty. From the opening arithmetic, 2 tests at 0.5
percent flake pass together with probability 0.995^2 = 0.990, so the suite is a
signal rather than a lottery.

I also state what I deliberately do not test end to end: every validation rule,
every error message, every country's postcode format. Those are asserted one or
two levels down where they are fast and deterministic.

**The error most readers make here.** Adding end to end coverage in proportion to
how important the flow is. Importance argues for more assertions, not for more
assertions at the most expensive and least reliable level.

#### Block C. Purpose: stop the mocks from drifting away from the server

Every integration test in Block B mocks the payment and address services. That
mock is a copy of a contract that will change.

- The payment provider is third party, so consumer driven contracts are not
  available. I use recorded fixtures from their sandbox, refreshed nightly by a
  scheduled job that fails loudly when a response shape changes.
- Our own order service is ours, so mocks are generated from its published
  schema, and a schema change breaks the consumer build immediately.
- One end to end test against the sandbox is the final backstop, which is a
  second reason the two end to end tests earn their place.

I name the failure this prevents, because it is specific: the provider adds a
required field, our mocks keep the old shape, every test passes, and checkout
fails in production for exactly the users whose payment path hits the new
requirement.

**The error most readers make here.** Treating mock drift as a testing detail. It
is the mechanism by which a green suite becomes uninformative, and it fails
silently, which makes it worse than a flaky test.

#### Block D. Purpose: design the exposure ramp and its abort rule

I use the sizing arithmetic to pick the ramp rather than picking round numbers.

- Session error rate baseline 0.5 percent. Detecting a doubling needs about 3,184
  sessions per arm.
- At 120,000 checkout sessions per day, a 5 percent canary gets 6,000 sessions
  per day, so about 250 per hour. Reaching 3,184 takes roughly 13 hours.
- That is too slow for the "detectable in minutes" requirement, so I start the
  ramp higher: 25 percent gives 1,250 sessions per hour and reaches the threshold
  in about 2.5 hours.
- For faster detection I add a much larger effect guardrail: a checkout specific
  error rate that would rise from near zero to a large fraction if the form is
  broken. A change from 0.5 percent to 10 percent needs
  16 x 0.005 x 0.995 / (0.095 squared) = 16 x 0.004975 / 0.009025 = about 9
  sessions per arm, so a catastrophic break is visible almost immediately.

The ramp: 5 percent for 1 hour as a smoke check, 25 percent for 3 hours, 50
percent overnight, 100 percent the next morning. Abort automatically on the
catastrophic guardrail, page a human on the doubling guardrail.

Conversion is measured, and it is not a gate. From the arithmetic, one percentage
point needs 25,600 sessions per arm, about 10 hours at 25 percent exposure, so it
informs the decision to stay at 100 percent rather than gating the ramp.

!!! tip "Self-explanation prompt"

    Say why a guardrail with a large expected effect can gate an automatic abort
    while a guardrail with a small expected effect cannot, in terms of the sample
    size formula rather than in terms of intuition.

**The error most readers make here.** Picking 1 percent, 5 percent, 25 percent,
100 percent because those are the conventional numbers. The exposure percentage
should come from how long you are willing to wait for the guardrail to become
readable, which is arithmetic.

#### Block E. Purpose: make the reversal faster than the diagnosis

A rollback redeploys. A kill switch does not. For the highest value flow in the
product I want the second.

| Property | Choice | Reason |
|---|---|---|
| Mechanism | A configuration value read by the client, defaulting to the old form | The old code stays deployed for the duration of the ramp, which is what makes reversal instant |
| Propagation | Polled or pushed, bounded at 60 s | Single page application sessions can last hours without a reload |
| Default on failure | Old form | If the configuration service is unreachable, fail to the known good path |
| Who can flip it | On call, without a code review | A control that requires an approval is not available during the incident it exists for |
| Verification | Flipped once in a scheduled game day before the rollout starts | An untested switch is a belief |

The cost of this is that both code paths ship during the ramp, which adds bytes.
I price it and accept it: if the new form is 30 KB compressed, carrying both for
a week costs about 0.15 s of transfer at 200 KB/s for that week. That is a cheap
insurance premium on the highest value flow in the product.

**The error most readers make here.** Deleting the old code path in the same
release that introduces the new one, which converts a 60 second reversal into a
deploy cycle, on the day you least want one.

### 16e. Fade the scaffold

**Fade 1: rolling out a design system major to 6 consuming applications.** The
change is visual and API breaking, a codemod exists, and each app has its own
release train. Same five block shape, last block blank.

- Block A, the feared failure: not an outage. It is a visual regression that
  reaches users in one app and is attributed to the design system a week later,
  after five other apps have adopted. The requirement is attribution, not
  reversal.
- Block B, assertions at the cheapest level: unit tests for the changed component
  APIs, integration tests for the interaction models that changed, and visual
  regression as the primary signal, since the change is visual. End to end tests
  are close to useless here and should not be added.
- Block C, stop the drift: the equivalent of contract testing is running the
  consumers' visual regression suites against the release candidate before it
  ships, which turns consumer breakage into a pre-release signal.
- Block D, the exposure ramp: there is no percentage ramp, because adoption is per
  application. The ramp is ordinal: one low risk consumer first, then two, then
  the rest, with a defined wait between them.
- Block E, make reversal fast: **you write this block.**

??? note "Show answer"

    Reversal here is version pinning, and it is only fast if it was designed for.
    Each consumer must be able to return to the previous major with one dependency
    change and no code changes, which is only true if the codemod's output does
    not depend on the new major existing.

    That constraint is worth stating explicitly, because it is easy to violate:
    if the codemod rewrites call sites into a form only the new major accepts,
    reverting the dependency breaks the build and the consumer is trapped forward.
    Design the codemod's output to be valid in both majors where possible, and
    where it is not, say so and provide the reverse codemod.

    Second mechanism: keep the previous major published and supported for the
    whole adoption window, with a documented end date, per the deprecation clock
    in Module 12. A consumer that hits a problem at 4 pm needs to pin, not to wait
    for a patch release from a team in another time zone.

    Third: because the failure is attribution rather than outage, the useful
    control is a version identifier reported with every error and every visual
    regression failure in consumer suites. Then the week later report says which
    design system version the app was on, and the investigation is a minute rather
    than an afternoon.

    The general shape: when the feared failure is slow and diffuse rather than
    fast and sharp, invest in attribution and in the ability to pin, not in a kill
    switch.

**Fade 2: an internal admin tool.** 40 staff users, no revenue path, one team,
weekly releases, and the team currently has no tests at all and is asking where
to start. Last two blocks blank.

- Block A, the feared failure: a member of staff performs a destructive action
  (bulk delete, refund, permission change) on the wrong records, and there is no
  undo. Slowness and visual defects are annoyances here, not risks.
- Block B, assertions at the cheapest level: start with unit tests for the
  destructive operations' preconditions and integration tests for the confirmation
  interactions. Do not start with a test pyramid, start with the three operations
  that cannot be undone.
- Block C, stop the drift: **you write this block.**
- Block D, the exposure ramp: **you write this block.**

??? note "Show answer"

    Block C. With 40 users and one team, formal contract testing is over
    engineering, and saying that plainly is part of the answer. The realistic
    mechanism is recorded fixtures from the real API, refreshed on a schedule, so
    a shape change surfaces within a day.

    The one place to spend more: the destructive endpoints. For those, a nightly
    smoke test against a staging environment that actually performs the operation
    on a seeded record and asserts the result is worth its cost, because the fear
    from Block A is precisely that these paths silently change meaning. That is
    two or three tests, not a suite.

    There is also a non testing control that is cheaper than all of it and belongs
    in the answer: make the destructive operations reversible on the server, with
    a soft delete and a restore path. If undo exists, the entire fear from Block A
    downgrades from incident to inconvenience, and much of the testing investment
    becomes optional. Proposing the change that removes the need for the
    machinery is a stronger answer than proposing the machinery.

    Block D. There is no percentage ramp with 40 users, and pretending otherwise
    would be cargo cult. Percentage exposure needs enough sessions for a guardrail
    to become readable, and by the sizing arithmetic even a doubling of a 0.5
    percent error rate needs thousands of sessions per arm. With 40 users the
    canary is statistically empty.

    What replaces it: release to the two or three staff who are closest to the
    team on the morning of a working day, watch, then release to the rest that
    afternoon. Human observation is the measurement instrument at this scale, and
    it is a legitimate one. State the wait explicitly so it is a plan rather than
    a hope.

    Keep one thing from the worked example: a kill switch on any newly added
    destructive operation, defaulting off, because the cost of the switch is an
    hour and the cost of the failure is unrecoverable data. Reversibility, not
    exposure control, is what scales down to 40 users.

### 16f. Check yourself

**Q1.** Your team wants an end to end test for every user story, arguing that it
is the only level that tests what the user actually experiences. Respond with
arithmetic and then with a concession.

??? note "Show answer"

    Arithmetic first. Suite reliability compounds: at a per test flake rate f and
    n tests, the probability that a run is green on correct code is (1 - f)^n. At
    f = 0.005, 50 tests give 0.78, 200 give 0.37, and 500 give 0.08. Every test
    added makes every other test's signal less useful, which is the property that
    makes "one per story" unworkable rather than merely expensive.

    Then the wait time. If a green run takes 1 / 0.08 = 12.5 attempts at 34
    minutes, the merge queue is the bottleneck of the whole team, and the
    predictable response is that people start rerunning until green without
    reading the failure, which is when the suite stops catching real defects too.

    The concession, and it is genuine: they are right that integration tests with
    a mocked transport can pass while production fails, because the mock can be
    wrong. That is a real gap and it deserves a real answer, which is contract
    testing plus a small number of end to end tests on the paths where a
    cross system failure is unrecoverable.

    So the proposal I would make: keep end to end coverage for irreversible and
    revenue critical journeys, target a count in the tens, and put the saved
    effort into contract testing, which addresses their actual concern at a
    fraction of the reliability cost.

**Q2.** A canary at 1 percent exposure has been running for 6 hours on a site with
40,000 daily sessions. Conversion is down 0.8 percentage points from a 15 percent
baseline. Should you roll back?

??? note "Show answer"

    First compute how much data exists. 40,000 sessions per day at 1 percent is
    400 per day, and 6 hours is a quarter of a day, so about 100 sessions in the
    canary arm.

    Now compute what is detectable. For a 0.8 point change on a 15 percent
    baseline, variance is 0.15 x 0.85 = 0.1275, effect squared is 0.000064, so
    n = 16 x 0.1275 / 0.000064 = about 31,900 per arm. You have 100.

    So the observed 0.8 point difference carries essentially no information. With
    100 sessions at 15 percent you expect about 15 conversions, and the standard
    deviation of that count is sqrt(100 x 0.15 x 0.85) = 3.6, which is 3.6
    percentage points. The observed difference is a fifth of one standard
    deviation.

    The answer is: this observation is not a reason to roll back and it is also
    not a reason to proceed. Look at the guardrails that are readable at this
    sample size, which is error rate for a large effect, and if those are clean,
    increase exposure so the conversion question becomes answerable in a
    reasonable time rather than waiting at 1 percent for 80 days.

    The underlying error is treating a canary as a small experiment. A canary
    detects breakage. Measuring a small conversion effect is an experiment, and it
    needs to be sized as one.

### 16g. When not to use this

**End to end tests beyond a small set.**

| Question | Answer |
|---|---|
| Measurement that justifies each additional test | The journey crosses a system boundary you cannot mock faithfully, AND a failure there is irreversible or revenue critical. Both, not either |
| Scale threshold above which it is over-engineering | Compute (1 - flake)^n for your measured flake rate. Once the green probability drops below roughly 0.8, every added test is reducing the suite's usefulness. At a 0.5 percent flake rate that is about 45 tests |
| Cheaper alternative | Integration tests with the transport mocked, plus contract testing against the provider's schema or recorded fixtures. It addresses the same fear with deterministic, fast tests |

**Visual regression testing.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You ship a shared component library or a design critical surface, AND you can keep the per build review load in single digits after freezing fonts, animations and time |
| Scale threshold | For a small application where visual changes are reviewed by the person who made them, the suite adds a per build human cost with no separate consumer to protect |
| Cheaper alternative | Screenshot the few pages that matter, on demand, in the pull request, for human review. Same information, no baseline management, no flake budget |

**Consumer driven contract testing.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You consume an interface owned by another team that changes without telling you, and you can name an incident caused by that. Provider cooperation is available |
| Scale threshold | If you own both sides and can change them in one commit, a shared schema and generated types already give you the guarantee at compile time |
| Cheaper alternative | Recorded fixtures refreshed nightly against a real environment. Detects drift a day late, costs a scheduled job, and needs nothing from the provider |

**A percentage based canary with an automatic abort.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Your traffic is high enough that a guardrail with a large expected effect becomes readable within your tolerance window. Compute n = 16 x variance / effect squared and divide by sessions per hour |
| Scale threshold | Below a few thousand sessions per day, the canary arm is statistically empty and an automatic abort is triggered by noise. This includes almost every internal tool |
| Cheaper alternative | Release to a small named group of users at a time of day when people are watching, with a kill switch. Human observation is a legitimate instrument at small scale |

---

## Module 17. Observability

**Time: 30 to 40 minutes reading, 20 to 30 minutes practice.**

Observability is where a frontend design answer either becomes operable or stays
theoretical. The organising question: when a user says the page is slow or
broken, what exactly do you look at, and does it contain enough data to answer.

### 17a. Predict first

Read this artefact and commit to an answer before continuing.

```
+---------------------------------------------------------------+
| Real user monitoring, as configured:                          |
|   head sampling      1% of sessions                           |
|   page views per day 2,000,000                                |
|   dashboard cells    30 routes x 4 device classes = 120       |
|   errors             sampled at the same 1%                   |
|                                                               |
| Dashboard reads: LCP p75 = 2.1 s, labelled good.              |
| Support queue: 40 tickets this week reporting slowness.       |
+---------------------------------------------------------------+
Caption: a sampling configuration, the dashboard it produces,
and the contradicting evidence from support.
```

**Question.** Compute the samples per dashboard cell and say what the dashboard
can and cannot support. Then name the single configuration change with the
largest effect.

??? note "Show answer"

    Samples: 1 percent of 2,000,000 is 20,000 per day. Across 120 cells that is
    167 per cell.

    Using the binomial arithmetic from Module 11, the count above the 75th
    percentile in a cell of 167 has standard deviation sqrt(167 x 0.1875) = 5.6,
    so two standard deviations is 11.2 samples, which is 6.7 percentile points.
    Each cell's "p75" could be anywhere from about the p68 to the p82. Cell level
    numbers are not readable.

    The global number is fine: sqrt(20,000 x 0.1875) = 61, two standard deviations
    is 122 out of 20,000, which is 0.6 percentile points. So the dashboard can
    support "did the whole site move", and cannot support "which route regressed",
    which is the question people actually ask it.

    The support tickets are a separate lesson. 40 tickets against roughly 14
    million weekly page views is about 0.0003 percent of traffic. Even at 100
    percent sampling, that population is invisible in a p75 and invisible in a
    p95. A percentile cannot find a small suffering group. You need the tail
    (p99 and above), or per session outlier capture, or a way to look up a
    specific complaining user.

    The single highest value change: stop sampling errors. Errors are rare events
    and sampling them at 1 percent both discards rare crashes entirely and makes
    the count uninterpretable. Collect 100 percent of errors and sample only the
    high volume timing data.

### 17b. What to collect, and why attribution is the whole game

A vitals number on its own tells you something changed. It never tells you what
to do. The payload alongside it does.

| Signal | Collect with it | What that enables |
|---|---|---|
| Largest contentful paint | The element identifier, its URL, and the time split into time to first byte, resource load delay, resource load time and render delay | Distinguishes a slow server from a late discovered image from a render blocked paint |
| Interaction to next paint | The interaction target, the event type, and the split into input delay, processing time and presentation delay | Names which of the three costs grew, which is the entire diagnosis in Module 11 |
| Layout shift | The elements that moved and the largest shift's sources | Turns one number into a specific unreserved container |
| Every sample | Route pattern (not the raw URL), device class, connection class, build identifier, cache state, whether the tab was backgrounded | Segmentation, and attribution to a deploy |

**Route pattern, not raw URL.** Collecting the raw path creates unbounded
cardinality and makes every cell too small to read, which is exactly the failure
in the opening artefact. Collect the pattern, and keep the raw path only on error
records where volume is low.

**Build identifier on every sample.** It is the cheapest field on the list and it
is what turns "performance got worse this week" into "performance got worse in
build 4471". Without it, correlating a regression to a deploy is manual and
usually wrong.

### 17c. Sampling that leaves the data readable

Three strategies, and the third is the one that rarely appears in an answer at all.

| Strategy | How | Good for | Weakness |
|---|---|---|---|
| Head sampling | Decide at session start with a fixed probability | Simple, and the client sends less | Rare routes and rare devices get too few samples, as computed above |
| Tail sampling | Collect everything on the client, send only what is interesting (slow, errored, outlier) | Keeps the tail, which is where the complaints live | Biases any distribution you compute from it, so you must keep a separate unbiased stream |
| Stratified sampling | Different rates per route or device class, with a weight recorded per sample | Makes low traffic cells readable without paying for high traffic ones | Requires you to divide by the weight when aggregating, and someone will forget |

**Sizing stratification, worked.** Target roughly 1,000 samples per cell per day,
which by the arithmetic in Module 11 gives a p75 stable to about 2.7 percentile
points.

- A route with 500,000 daily views needs a 0.2 percent rate to reach 1,000
- A route with 5,000 daily views needs 20 percent
- A route with 800 daily views cannot reach 1,000 at any rate, so aggregate it
  into an "other" bucket and stop pretending it has its own number

That last line is the discipline. A dashboard cell that cannot be readable should
not exist, because an unreadable cell is worse than an absent one: someone will
act on it.

**Always collect at 100 percent.** Errors, and any event rare enough that
sampling changes the answer. The cost is low precisely because they are rare.

### Errors, boundaries and the noise you did not cause

**Error boundaries catch less than people assume.** State the gaps, because
"we will add error boundaries" is a common and incomplete answer.

| Caught by a render boundary | Not caught |
|---|---|
| Errors thrown while rendering a child | Errors inside event handlers |
| Errors in lifecycle or effect setup, depending on the framework | Errors in asynchronous callbacks and promise rejections |
| | Errors during server rendering, which need their own handling |
| | Errors thrown by the boundary itself |

So a complete answer pairs boundaries with a global handler for uncaught errors
and one for unhandled promise rejections, and it places boundaries deliberately:
one per route, one per independently failing widget, and not one per component,
which produces a page of fallbacks and hides the failure.

**The noise sources, which are most of your error volume.**

| Source | Signature | What to do |
|---|---|---|
| Browser extensions | Stack frames from extension URLs, errors in scripts you never shipped | Filter by frame origin, do not attempt to fix |
| Injected consumer software and network middleboxes | Syntax errors in your own files, which is otherwise impossible | Filter, and treat a spike as a signal about a population, not a bug |
| Bots and synthetic traffic | Impossible user agents, no interaction events | Filter at ingestion |
| Layout observation loops | A specific recurring message with no user impact | Fix if yours, otherwise rate limit |
| Chunk load failures after a deploy | Failure to load a hashed asset, clustered right after a release | Covered below, this one is yours |

Without filtering, the noise categories dominate and the tracker becomes
ignorable, which is the actual failure mode: the data is present and nobody
reads it.

**Chunk load failures, and the retention arithmetic.** After a deploy, a client
holding the old document requests an asset that no longer exists.

Price keeping old assets:

- Assume 400 chunks per build at an average 30 KB, so about 12 MB per build
- 20 builds per day for 30 days is 600 builds, about 7.2 GB total

Storage of a few gigabytes is not a meaningful cost, so retention is a policy
choice rather than a budget question. Keep 30 days of build assets.

Then add the client side half, because retention alone does not cover a tab open
for months:

- On a chunk load error, reload the page once, guarded by a marker so it cannot
  loop
- If the marker is already set, show an explicit "a new version is available"
  message rather than reloading again
- Emit a distinct event for this case so it does not pollute the general error
  rate

### Source maps and session replay

**Source maps in production, three options.**

| Option | Who can read your source | Operational cost | When it is right |
|---|---|---|---|
| Published alongside the bundle | Anyone | None | Open source, or code whose disclosure genuinely does not matter |
| Uploaded privately to the error tracker, not served to browsers | The tracker vendor and your team | A build step and a secret | The default for most products |
| Self hosted behind authentication, symbolication performed by your own service | Your team only | A service to run | Strict requirements about where source may go |

Two notes worth stating. First, minified code is not a security control, so
withholding maps protects intellectual property and convenience, not safety.
Second, whichever you choose, the map must be keyed to the build identifier or
symbolication silently produces wrong line numbers after the next deploy.

**Session replay, and its real price.** Replay reconstructs a session by
recording document mutations and input events, then replaying them.

Cost, computed:

- Assume 60 KB per minute of compressed mutation data for a moderately dynamic
  interface, and a 6 minute average session, so about 360 KB per recorded session
- 2,000,000 page views per day at 3 views per session is about 667,000 sessions
- At 1 percent recording that is 6,670 sessions, about 2.4 GB per day, roughly 72
  GB per month before retention policy

Plus a device cost that rarely gets mentioned: continuously observing document
mutations is main thread work on the user's device, so a replay tool can degrade
the interaction metric you installed it to investigate. Measure your own
interaction metric with the tool on and off before deciding.

**The privacy cost, stated as a design rule.** Replay records what is on screen,
which in most products includes personal data.

| Rule | Reason |
|---|---|
| Mask by default, unmask by explicit allowlist | An opt out list is a list of the fields someone remembered. The one they forgot is the one in the incident |
| Never record password, payment or authentication fields, under any configuration | The consequence of one mistake is unbounded |
| Strip credentials and tokens from recorded URLs | Tokens in query strings are common and are captured verbatim |
| Set a short retention, days rather than months | Recordings are personal data, subject to access and deletion requests, and every extra day is extra exposure |
| Record who viewed a replay | Internal access to recordings is the risk people forget |

### 17d. Worked example

**The prompt.** "We are launching a checkout funnel. Design the observability so
that when it breaks at 2 am, the person on call can tell what broke and for
whom."

Inputs, stated before I touch anything:

| Input | Value | Status |
|---|---|---|
| Sessions entering the funnel per day | 120,000 | GIVEN |
| Steps in the funnel | 4 | GIVEN |
| Device classes worth separating | 3 | GIVEN |
| Session level error rate today | 0.5 percent | GIVEN |
| Budget for monitoring data | Not stated | To be asked, it decides replay and sampling |
| Average recorded replay session size | 360 KB | ASSUMED, from the arithmetic above. Measure yours in a one week trial before committing |

#### Block A. Purpose: write the questions the data has to answer, before choosing tools

I list the questions the person on call will actually ask, because each one
implies a field or a signal, and any question I cannot answer is a gap I would
rather find now.

| Question at 2 am | What it requires |
|---|---|
| Is it everyone or a segment | Route pattern, device class, connection class, build identifier on every sample |
| Did it start with a deploy | Build identifier, and a deploy annotation on the time series |
| Which step of the funnel | A step identifier on every event, not just a page URL |
| Is it the client or the server | Client error rate and request failure rate by status, separated |
| Can I see one broken session | A session identifier, and a way to look one up from a support ticket |
| Is it getting worse | A rate, not a count, with an interpretable denominator |

The last row is the one that catches people. An error count rises when traffic
rises. On call needs errors per session, and the denominator must be collected
with the same sampling as the numerator or the ratio is meaningless.

**The error most readers make here.** Choosing tools first. The tool list is
short and mostly interchangeable. The field list is the design, and it is what
makes any tool useful.

!!! tip "Self-explanation prompt"

    Before reading Block B, pick the one row you would drop if you had to drop
    one, and say what question becomes unanswerable.

#### Block B. Purpose: set sampling so every cell is readable or absent

Cells I want: 4 funnel steps x 3 device classes = 12 cells. That is a
deliberately small number, chosen because the funnel is one route family rather
than the whole site.

- Target 1,000 samples per cell per day, from the Module 11 arithmetic, giving a
  p75 stable to about 2.7 percentile points
- 12 cells x 1,000 = 12,000 samples per day needed
- 120,000 sessions produce roughly 4 step views each, so about 480,000 step views
  per day
- Required rate: 12,000 / 480,000 = 2.5 percent

I round to 5 percent for headroom on the smallest device class, which is
typically much smaller than the largest. Then I check the smallest cell rather
than the average: if the least common device class is 10 percent of traffic, its
cells get 480,000 x 0.10 x 0.05 / 4 = 600 samples, which is below target, so I
stratify and raise the rate for that class to 10 percent with a recorded weight.

Errors and funnel step transitions are collected at 100 percent, because they are
rare and because the denominator must be exact.

**The error most readers make here.** Setting one sampling rate for everything.
Timing data is high volume and tolerates sampling. Errors and conversion events
are low volume and do not.

#### Block C. Purpose: place error handling so a failure is contained and named

Three layers, each catching what the others cannot:

| Layer | Placement | Catches |
|---|---|---|
| Route level boundary | One per funnel step | A render failure in that step, with a fallback that offers a retry and a support path |
| Widget level boundary | Around the payment element and the address lookup only | A third party component failing without taking the step down |
| Global handlers | One for uncaught errors, one for unhandled promise rejections | Event handler and asynchronous failures, which no boundary catches |

Plus the filtering rules from earlier, applied at ingestion rather than at query
time, so the dashboards are clean by default: extension frames, injected script
signatures, bot user agents, and chunk load failures routed to their own event
type.

I place a boundary around the payment element specifically because it is third
party code whose failure I cannot fix at 2 am. The fallback for that boundary
matters: it must let the user complete by another route if one exists, or tell
them clearly if not, because a silent empty region is the worst outcome.

**The error most readers make here.** A boundary per component. It converts one
visible failure into a page of small fallbacks, none of which is alarming enough
to be reported, and the aggregate looks like a working page.

!!! tip "Self-explanation prompt"

    Say why chunk load failures deserve their own event type rather than counting
    toward the general error rate, in terms of what the on call person would
    conclude if they were mixed together.

#### Block D. Purpose: decide on replay by pricing it against the alternative

Replay is the expensive item, so I price it and then price the cheaper thing.

- 120,000 sessions per day, recording 1 percent is 1,200 sessions at 360 KB, so
  about 430 MB per day, about 13 GB per month before retention. Affordable
- The device cost is the real question: I would measure the funnel's interaction
  metric with recording on and off before enabling it, because degrading checkout
  interactivity to observe checkout interactivity is a bad trade

The cheaper alternative I compare against: a breadcrumb trail attached to every
error, recording the last 20 user actions, the last 10 network requests with
status and duration, the funnel step history, and the state of any feature flags.
That is a few kilobytes per error, sent only with errors, with no continuous
observation cost and a far smaller privacy surface.

My choice: breadcrumbs for everyone, and replay enabled only for sessions that
have already errored, if the tool supports retroactive capture, or for a small
opt in population. This is tail sampling applied to replay, and it captures most
of the value at a fraction of the cost.

Masking is default deny, payment and authentication fields are never recorded
under any configuration, retention is set in days, and access to recordings is
logged.

**The error most readers make here.** Enabling replay at a high rate on the most
sensitive flow in the product, because that is where the debugging value seems
highest. It is also where the privacy exposure is highest, and the two scale
together.

#### Block E. Purpose: define what wakes someone up, and what does not

Alerting rules, each with the reason for its threshold:

| Alert | Rule | Reason |
|---|---|---|
| Funnel step error rate | Errors per session in any step above 2 percent for 10 minutes | Baseline is 0.5 percent. Four times baseline is well outside daily variation and is detectable in minutes at this traffic |
| Step to step transition rate | A drop of more than 30 percent from the trailing 7 day value at the same hour, sustained 15 minutes | Compares like with like, since the funnel has a daily shape. 30 percent is large enough to exceed noise at these volumes |
| Payment provider failure rate | Above 5 percent for 5 minutes | Third party, and the response is to switch or degrade, so it needs its own signal |
| Chunk load failures | A rate above the post deploy baseline, which is measured rather than assumed | Distinguishes a broken build from ordinary post deploy churn |
| Interaction to next paint p75 | Alert only, never page | Field data lags and no p75 change is worth a 2 am wake up |

The last row is a policy statement and it belongs in the design: performance
degrades, it does not break. Paging on a percentile trains people to ignore
pages.

**The error most readers make here.** Alerting on absolute counts. A count alert
fires every peak hour and stays silent during an overnight outage, which is
precisely inverted from what you want.

### 17e. Fade the scaffold

**Fade 1: an internal admin tool.** 40 staff users, 1,500 page views per day,
one team, and the tool performs destructive operations. Same five block shape,
last block blank.

- Block A, the questions to answer: mostly "what did this specific person do
  before it broke", not "which segment is affected". With 40 users, individual
  identification is both possible and appropriate, which changes the design more
  than any tooling choice.
- Block B, sampling: collect everything at 100 percent. 1,500 page views per day
  is a rounding error in any monitoring budget, and any sampling makes the data
  unusable for the individual level questions in Block A.
- Block C, error handling: one boundary per route is enough. Global handlers
  matter more here than boundaries, because a small tool has more ad hoc event
  handler code and fewer render paths.
- Block D, replay versus breadcrumbs: replay is genuinely attractive at this scale
  because the volume is tiny and the users are colleagues who can be told they are
  recorded. The privacy analysis is different, not absent: staff are still people,
  and the tool shows customer data, so masking rules still apply to the content on
  screen even when the user consents.
- Block E, what wakes someone up: **you write this block.**

??? note "Show answer"

    Almost nothing should wake anyone up, and saying so is the answer rather than
    a failure to design alerts.

    Compute why: 1,500 page views per day is about 60 per working hour. At a 0.5
    percent baseline error rate that is 0.3 errors per hour. Any threshold based
    alert on a rate is noise at this volume, because a single error moves the rate
    by hundreds of percent.

    What replaces rate alerts: alert on individual occurrences of specific events
    rather than on rates. A failure of a destructive operation, an authorisation
    failure, or an unhandled error on the bulk action screen should each notify
    the team channel once, with full context, during working hours. That is 1 to 5
    notifications per week and each is actionable.

    Nothing pages overnight, because nobody uses the tool overnight. This is worth
    stating explicitly in the design, because inherited alerting configurations are
    how teams end up with 3 am pages for systems with no 3 am users.

    One thing does deserve a page: if the destructive operations start succeeding
    against unexpected volumes, for example a bulk delete exceeding a threshold
    number of records. That is not an error rate, it is a business invariant, and
    it is the one signal here where minutes matter.

**Fade 2: a marketing site.** 500,000 page views per day, largely anonymous
first time visitors, three third party tags, no login, no funnel beyond a
newsletter form. Last two blocks blank.

- Block A, the questions to answer: the dominant question is "did a marketing
  change break or slow the page", and the secondary one is "which third party is
  responsible". The site's own code changes rarely and the tags change weekly.
- Block B, sampling: 500,000 views per day at 10 percent gives 50,000 samples.
  With 8 landing pages and 3 device classes, 24 cells get about 2,000 each, which
  is above the 1,000 target. Errors at 100 percent as always.
- Block C, error handling: **you write this block.**
- Block D, replay versus breadcrumbs: **you write this block.**

??? note "Show answer"

    Block C. The distinguishing fact is that most script on this page is not
    yours, which inverts the usual advice.

    Error boundaries have little to catch, because a marketing page has few render
    paths and little client state. Global handlers matter far more, and the single
    most valuable configuration is attribution by script origin: group errors by
    the host that served the failing script, so the weekly question "which tag
    broke" is answered by a dashboard rather than an investigation.

    Add a specific detector for the failure mode that actually costs money here: a
    third party script failing to load or timing out, blocking or degrading the
    page. Record the load status and duration of each third party script as an
    explicit event, not as an error, so you have a rate over time per vendor and
    can hold a conversation with them or remove them with evidence.

    Filtering matters more than usual, because anonymous first time traffic has
    the highest proportion of extensions, bots and injected software. Without
    aggressive ingestion filters, your own errors are a small minority of the
    volume.

    Block D. Replay is a poor fit and the reasoning is worth being explicit about.
    The users are anonymous first time visitors who have no relationship with you,
    the consent situation is the most demanding of any example in this module, and
    the debugging value is low because the page has almost no state to reconstruct.

    Breadcrumbs are also less useful than usual, for the same reason: there is
    little interaction history to record.

    What replaces both: a page level snapshot attached to any error, containing
    which third party scripts loaded and their timings, which experiment or tag
    variants were active, the viewport and device class, and the build identifier
    of the page. That reconstructs the situation without recording the person, and
    it is a few hundred bytes.

    The one place to spend more: the newsletter form, which is the only conversion
    on the site. Instrument it as a two step funnel with 100 percent event
    collection, including validation failures by field. A form that silently
    rejects a valid email address in some locale is the highest cost defect this
    site can have, and it is invisible in every other signal.

### 17f. Check yourself

**Q1.** Your error dashboard shows 40,000 errors per day and the team has stopped
looking at it. Give a plan to make it useful again, in order.

??? note "Show answer"

    First, classify before fixing anything. Group the 40,000 by the origin of the
    top stack frame and by message signature. In most consumer facing products the
    large majority resolves into a handful of categories: browser extensions,
    injected consumer software and network middleboxes, bots, and chunk load
    failures after deploys.

    Second, filter at ingestion rather than at query time. A filter applied in the
    query is a filter each person has to remember. Filtering at ingestion makes
    the default view correct, which is the only view most people will ever use.

    Third, separate the categories that are yours but not defects. Chunk load
    failures get their own event type with their own baseline, because their
    normal level after a deploy is not zero and mixing them into the error rate
    destroys the rate's meaning.

    Fourth, convert counts to rates with an explicit denominator, so that a busy
    Tuesday does not look like an incident and a quiet outage does not hide.

    Fifth, and only now, set alerts. Alerting on a noisy dashboard is what taught
    the team to ignore it, and repeating that before the cleanup would repeat the
    outcome. Expect the post filter volume to be one to two orders of magnitude
    smaller, at which point individual investigation becomes possible again.

**Q2.** A product manager asks for session replay on 100 percent of sessions "so
we never miss anything". Give the cost in numbers and a counter proposal.

??? note "Show answer"

    Take a site with 667,000 sessions per day and the assumed 360 KB per recorded
    session. At 100 percent that is 240 GB per day, about 7.2 TB per month before
    retention policy. Storage, ingestion and the vendor's pricing all scale with
    that number, and it is typically the largest line in a monitoring budget by a
    wide margin.

    The device cost is the second number and it is the one that should decide it.
    Continuous mutation observation is main thread work, so replay can measurably
    raise the interaction metric on lower end devices. Recording every session to
    investigate slowness, while making sessions slower, is a self defeating
    configuration. Measure it before enabling, with the tool on and off.

    The privacy cost is the third: every recording is personal data with access
    and deletion obligations, and the exposure scales linearly with coverage.
    Recording every session multiplies the consequence of one masking mistake by
    the whole user base.

    Counter proposal, in three parts. Breadcrumbs on 100 percent of sessions,
    attached to errors, which answers most "what happened before this broke"
    questions at a few kilobytes per error. Replay retained only for sessions that
    errored or that exceeded a latency threshold, which is tail sampling and keeps
    the interesting sessions. And a short retention measured in days, with masking
    default deny.

    That gets most of the diagnostic value at a small fraction of the cost, and it
    is a proposal rather than a refusal, which is what makes it land.

### 17g. When not to use this

**Session replay.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You can name recent defects that breadcrumbs plus error context failed to diagnose, AND you have measured the interaction metric with recording on and off and found the cost acceptable |
| Scale threshold below which it is over-engineering | If you can reproduce most reported issues from a description, or if traffic is small enough to talk to affected users directly, replay is a large privacy and cost commitment for convenience |
| Cheaper alternative | Breadcrumbs attached to errors: last 20 user actions, last 10 requests with status and duration, route history and active flags. A few kilobytes per error, no continuous recording, far smaller privacy surface |

**A full real user monitoring pipeline with per segment dashboards.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Every cell you intend to read reaches roughly 1,000 samples per day after sampling, and there is a named person who will act on a segment level finding |
| Scale threshold | Below a few thousand sessions per day, per segment percentiles are noise. The dashboard will be built, admired, and then quietly disbelieved |
| Cheaper alternative | 100 percent error collection plus a scheduled lab run on your most important routes, charted over time. Two signals, both readable, neither requiring a sampling design |

**An error boundary at every component.**

| Question | Answer |
|---|---|
| Measurement that justifies a boundary at a given place | That subtree can fail independently and the rest of the page is still useful without it. Third party embeds and independently loaded regions qualify |
| Scale threshold | Below a route level boundary plus two or three widget level ones, more boundaries mainly hide failures. A page of small fallbacks looks like a working page and gets reported by nobody |
| Cheaper alternative | One boundary per route with a clear, actionable fallback, plus global handlers for uncaught errors and unhandled rejections, which catch the cases boundaries never do |

**Publishing source maps publicly.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Your source is already public, or the debugging convenience for external contributors outweighs disclosure, which you can state as a decision rather than an oversight |
| Scale threshold | For a proprietary product, public maps are a disclosure with no operational benefit, since your own tracker can symbolicate from a private upload |
| Cheaper alternative | Upload maps privately to the error tracker as a build step, keyed to the build identifier, and do not serve them to browsers |

---

## Module 18. Level calibration

**Time: 25 to 35 minutes reading, 20 to 30 minutes practice.**

The same prompt is asked at every level, and the difference is not how much you
know. This module makes the difference explicit so you can hear it in your own
answers.

### 18a. Predict first

Read this artefact and commit to an answer before continuing.

```
+---------------------------------------------------------------+
| Prompt: design the front end for a live order tracking page.  |
| Sub question: how does the driver location reach the page?    |
|                                                               |
| ANSWER 1  Use WebSockets. The server pushes coordinates and   |
|           we move the marker. It is real time.                |
|                                                               |
| ANSWER 2  Roughly one small message every few seconds per     |
|           active order, and the client writes almost nothing, |
|           so I would take server sent events plus POST. I     |
|           would size the heartbeat from the shortest idle     |
|           timeout in the path and use full jitter on          |
|           reconnect so a deploy does not make a herd.         |
|                                                               |
| ANSWER 3  First, who owns the location feed and what is its   |
|           update rate and freshness target? If it is an       |
|           existing service, the front end question is mostly  |
|           degradation and cost. I would make the page correct |
|           with a polling fallback first, because that decides |
|           whether push is a dependency or an enhancement, and |
|           it decides what the sessions that never connect     |
|           see. Then I would pick a transport, and name the    |
|           measurement that would make me revisit it.          |
+---------------------------------------------------------------+
Caption: three answers to one sub question, unlabelled.
```

**Question.** Rank them by level, then state the specific thing each higher
answer added, in one clause each.

??? note "Show answer"

    Ranking: 1 is a mid level answer, 2 is senior, 3 is staff.

    Answer 1 names a technology. It is not wrong, and it contains no reasoning
    that would let anyone check it, so an interviewer cannot tell whether the
    candidate has judgement or a memorised default.

    Answer 2 adds a derivation. It counts the traffic in both directions, uses
    that count to make the choice, and continues past the choice into the
    operational consequences (heartbeat sizing, reconnect behaviour) that only
    someone who has run one of these mentions.

    Answer 3 adds two things that are not about the transport at all. It
    identifies a dependency owned by someone else and asks about it, which
    changes what the front end is responsible for. And it designs the degraded
    path first, which makes the whole feature robust regardless of which
    transport wins.

    The pattern across the three: mid names, senior derives, staff reframes. Notice
    that answer 3 never says WebSocket or server sent events. It would, thirty
    seconds later. The reframing came first because the reframing changes what the
    right answer is.

    One trap to avoid: answer 3 is not better because it asks a question. Asking
    questions with no consequence attached is a common imitation of seniority.
    It is better because the question's two possible answers lead to visibly
    different designs.

### 18b. What each level adds, dimension by dimension

| Dimension | Mid | Senior | Staff |
|---|---|---|---|
| Scoping | Accepts the prompt as given, asks a few clarifying questions | Narrows the prompt to what fits the time, and says what is excluded | Questions the premise where it is wrong, and identifies the dependency or constraint that reshapes the problem |
| Requirements | States functional requirements | States budgets as numbers with a source | Derives budgets from a user or business consequence, and says which one is binding |
| Design | Produces a working design | Produces the simplest design that meets the requirements, and names what breaks next | Produces a design plus the evolution path, and states what evidence would trigger each step |
| Trade-offs | Names a trade-off | Prices both sides of it | Prices both sides and says which measurement would change the decision |
| Depth | Explains one area competently when asked | Goes deep unprompted in the area where this problem is actually hard | Chooses the deep dive because it is where the risk is, and says why the other candidates are less risky |
| Failure | Mentions error handling | Names specific failure modes and their detection | Designs the degraded mode as a first class state, and defines what wakes someone up |
| Organisation | Not expected | Mentions rollout and ownership when asked | Treats team boundaries, migration and adoption as part of the design |
| Communication | Answers the question asked | Drives the conversation and manages the clock | Makes the interviewer's job easy: states the plan, flags the branch points, and checks in at real decisions |

**The single sentence version.** Mid demonstrates knowledge, senior demonstrates
judgement under constraints, staff demonstrates judgement about which problem to
solve.

**What does not raise your level, despite feeling like it does.**

| Move | Why it does not help |
|---|---|
| Naming more technologies | Breadth without derivation reads as a list, and invites a question you cannot answer |
| Drawing more boxes | Complexity without a triggering requirement reads as inability to restrain |
| Mentioning scale numbers with no consequence | A number that rules nothing out is decoration |
| Describing a company's published architecture | It answers a different problem, and you inherit its constraints without its context |
| Talking for longer | Interview signal is per minute, not per sentence |

### 18c. The negative signals, by level

Interviewers calibrate down on specific things, and they are different at each
level.

| Level | Calibrates down when the candidate |
|---|---|
| Mid | Cannot explain something they named. Guesses rather than saying they do not know. Writes code or markup in the air without a design |
| Senior | Designs before scoping. Cannot price a trade-off they named. Goes deep on the area they find comfortable rather than the area that is risky. Defends a choice after the interviewer supplies contradicting information |
| Staff | Stays at the level of principles without ever getting concrete. Reframes the problem and then does not solve it. Talks about organisation to avoid technical depth. Cannot say what they would measure first |

The last row is the one that surprises people. Reframing is a staff move only
when it is followed by a concrete design. Reframing alone reads as avoidance, and
it is a common failure mode for candidates who have read about seniority rather
than practised it.

### 18d. Worked example

**The prompt.** "Design the front end for a live order tracking page. Users open
it after ordering and watch until delivery."

I will answer the same prompt three times, one level per block, at the same
length, so the difference is visible as content rather than as volume.

Inputs, shared by all three:

| Input | Value | Status |
|---|---|---|
| Concurrent tracking sessions at peak | 80,000 | GIVEN |
| Location update rate from the delivery service | Every 5 s while moving | GIVEN |
| Median tracking session length | 12 minutes | GIVEN |
| Share of sessions on mobile networks | 85 percent | GIVEN |
| Share of sessions where a socket never establishes | 3 percent | ASSUMED, corporate networks and restrictive mobile carriers. Ask for the real figure, and design as if it is non zero |

#### Block A. Purpose: show the mid level answer in full, so its ceiling is visible

"I would build a page showing a map with the driver's position, the order status
steps, and an estimated arrival time. I would use WebSockets so the position
updates live. The map is a third party component. State lives in a store, and I
would show a loading state while the first data arrives and an error state if the
request fails.

For performance I would code split the map since it is large, and lazy load it.
For accessibility I would make sure the status steps are readable by a screen
reader, since a map is not accessible on its own."

What is right about this: it is a complete, working design with no errors in it.
The map splitting instinct is correct. Noticing that a map is not accessible is
better than most answers give.

What sets its ceiling: every choice is asserted. WebSocket is chosen without a
count, code splitting without a size, and the accessibility point stops at
"readable" without saying what a screen reader user actually needs from a live
tracking page.

**The error most readers make here.** Believing this answer is weak. It is not
weak, it is unfalsifiable. The interviewer has nothing to probe except by asking
"why", and the candidate's answer to "why" is what actually determines the level.

!!! tip "Self-explanation prompt"

    Before reading Block B, pick the single sentence in the mid answer that could
    be upgraded with one number, and write that sentence.

#### Block B. Purpose: show what the senior answer adds, and where it adds it

"Let me size it before choosing. 80,000 concurrent sessions, one update every 5
seconds, so 16,000 messages per second outbound to clients. Clients send almost
nothing: a page view and maybe a support tap.

That read to write ratio, several thousand to one, points at server sent events
rather than WebSocket. WebSocket would cost me reconnect code I have to write
myself and middlebox incompatibility, and buys duplex I do not need. I would move
to WebSocket if the product added driver chat, which would change the write rate
by orders of magnitude.

Reconnection: immediate first retry, then full jitter capped at 30 seconds. With
80,000 clients and a deploy, an uncapped fixed retry would put 80,000 connection
attempts into one second against a fleet that accepts a few thousand.

Budgets: the map component is the biggest byte item, so I would lazy load it
behind the initial paint and render order status from server rendered markup
first, since status is what the user checks most often and it does not need the
map. I would set a byte budget on the route entry and enforce it in the build.

Accessibility: the map is decorative for this purpose. The authoritative content
is the status and the estimate, which I would put in a live region that announces
on change but not on every position update, since announcing a coordinate every 5
seconds is hostile."

What this added: a count that drove the transport choice, the condition that
would reverse it, an operational consequence, a byte decision tied to what users
look at, and an accessibility decision that is specific rather than aspirational.

**The error most readers make here.** Adding numbers that do not eliminate
anything. Every number above rules out an option. A number that does not is
decoration, and interviewers notice the difference.

!!! tip "Self-explanation prompt"

    Say which sentence in the senior answer would be the strongest thing to say if
    you only had thirty seconds, and why it beats the transport choice.

#### Block C. Purpose: show what the staff answer adds beyond the senior one

"Before transport, two questions that change the design. Who owns the location
feed, and what is its actual freshness? If it updates every 5 seconds and is
already smoothed, my job is presentation and degradation. If it is raw and jittery,
I own interpolation and the perceived quality of the whole feature.

Second: what does the user actually want from this page? My assumption is
reassurance, not coordinates. If that is right, then the arrival estimate and the
status are the product, and the moving marker is supporting evidence. That
reorders everything: the estimate must be correct and stable, and the map can be
late, degraded or absent.

So I design the page to be correct with a 30 second polling fallback first. About
3 percent of sessions never establish a socket, and on a page whose purpose is
reassurance, 3 percent of users seeing a frozen map is worse than 100 percent
seeing a slightly stale one. Push becomes an enhancement over a correct baseline,
which also means a push outage is a degradation rather than an incident.

Then transport, and the senior reasoning applies: read heavy, so server sent
events, with the write rate as the condition that would change it.

One organisational point: the estimate is computed by another team's service. Its
accuracy determines the perceived quality of my page, and I will be the one
receiving the complaints. I would agree an accuracy target and a way to display
uncertainty, because showing a confident wrong time is worse than showing a range.

What I would measure first: the share of sessions that never receive an update
within 10 seconds of load, by network class. That single number tells me whether
the fallback is carrying 3 percent or 15 percent of traffic, and it decides where
the next quarter of work goes."

What this added: a reframing that changed the design (fallback first), an
assumption about user intent stated as an assumption, a cross team dependency
owned rather than mentioned, and a first measurement that would direct future
work.

**The error most readers make here.** Delivering the staff framing and stopping
there. Notice that the answer above still lands on a transport, a fallback
interval, a percentage and a metric. Reframing without landing reads as evasion,
and it calibrates down, not up.

### 18e. Fade the scaffold

**Fade 1: an internal admin console for support staff.** The prompt is "design
the front end for the tool support agents use while on a call with a customer".
Same three block shape, last block blank.

- Block A, the mid answer: a search box, a customer detail view with tabs for
  orders, payments and notes, an action panel for refunds and account changes.
  Client side routing, a data fetching layer with caching, loading and error
  states, and a design system for consistency. Complete, correct, unfalsifiable.
- Block B, the senior answer, the budget: agents handle calls back to back, so
  the binding constraint is time to first useful data during a live call, not
  bundle bytes on a fast internal network.
- Block B, the senior answer, the moves that follow: prefetch the customer record
  from the telephony system's incoming call event before the agent opens
  anything, which removes a full round trip from the start of every call.
  Keyboard first design, since agents type while talking. Optimistic updates
  rejected for refunds, because a rolled back refund shown to an agent who has
  already told the customer is worse than a 400 ms wait.
- Block C, the staff answer: **you write this block.**

??? note "Show answer"

    The reframing available here is that the tool's real constraint is the call,
    not the interface. An agent is a person under time pressure with a customer
    listening, so every design decision should be judged by whether it reduces the
    agent's cognitive load during a live conversation.

    Two consequences that follow and would not otherwise be reached. First,
    information hierarchy is the design: what the agent needs in the first 10
    seconds of a call is a small set (who is this, what is their most recent
    problem, what are they likely calling about), and everything else is a
    distraction that costs call time. That argues against a tabbed detail view,
    which is what the mid answer proposed.

    Second, the destructive actions need a different treatment than speed. A
    refund issued to the wrong account during a rushed call is the highest cost
    error this tool can produce, so it deserves friction: confirmation with the
    specific amount and account restated, and reversibility on the server, per
    Module 16's fade. That is a deliberate slowing of the one path everyone would
    otherwise optimise.

    The cross team point: the tool aggregates data from orders, payments and
    identity, all owned elsewhere. A staff answer owns the aggregation contract
    rather than assuming it, because the failure of any one of those services must
    degrade the tool rather than blank it, and each team needs to know that their
    latency lands on a person mid conversation.

    And the first measurement: time from call start to the agent's first
    meaningful action, segmented by call reason. That is the number the whole
    design serves, and almost nobody instruments it.

**Fade 2: a component scale prompt.** "Design a date range picker for a booking
product." Last two blocks blank.

- Block A, the mid answer: two linked calendar months, a start and end selection,
  disabled dates for unavailable ranges, keyboard navigation with arrow keys, and
  a controlled value with an onChange callback. Localised month names. Complete
  and correct as far as it goes.
- Block B, the senior answer: **you write this block.**
- Block C, the staff answer: **you write this block.**

??? note "Show answer"

    Block B, senior. The additions are an explicit state machine and an explicit
    API contract, because at component scale those are where the difficulty lives.

    The state machine has more states than people expect: idle, start selected and
    awaiting end, hovering a candidate end (which previews a range), both selected,
    and both selected with the user re-clicking to start over. Writing those five
    out loud is the move, because the ambiguous transitions (a click after both are
    selected, a hover after selection) are where every implementation of this
    component differs.

    The API contract: controlled and uncontrolled variants, a value that is a pair
    rather than two independent props (so an invalid state where end precedes
    start is unrepresentable), an onChange that fires on complete selections and a
    separate callback for partial ones, and disabled dates supplied as a predicate
    rather than an array so a booking product can express "no arrivals on
    Tuesdays" without enumerating a year.

    Plus the accessibility contract as part of the API: the grid semantics, roving
    focus across days, the announcement when a range is previewed, and a text
    input alternative because a calendar grid is a poor experience for a user who
    knows the dates.

    Block C, staff. Two reframings, and both change the component rather than
    describing it.

    First, a date range picker in a booking product is not a date component, it
    is an availability and pricing browser. Users are choosing when to travel
    subject to what is available and what it costs, so the calendar is where
    price and availability are communicated.

    That changes the API: each day cell needs to render supplied content, not
    just a number, which means the component must expose a day renderer rather
    than accepting a disabled date list. A component designed without this gets
    forked by the first product team that needs prices in the grid.

    Second, the hardest correctness problem is not the interaction, it is the
    dates themselves. A booking date is a local calendar date at the destination,
    not an instant, so it must never be represented as a timestamp that a time zone
    can shift by a day. This is the classic off by one bug in every booking
    product, and it is an API decision: accept and emit plain calendar dates, and
    document that the component never touches time zones.

    Then land it: a state machine with the five states, a pair valued controlled
    API, a day renderer, plain calendar dates, and the accessibility contract. And
    the first measurement: the rate at which users abandon the picker after opening
    it, segmented by input method, because a keyboard user quietly abandoning is
    invisible in every other metric.

### 18f. Check yourself

**Q1.** You are interviewing at senior level. In the last ten minutes the
interviewer asks about rolling the design out across three teams. You have not
finished the deep dive you started. What do you do, and why?

??? note "Show answer"

    Answer the question briefly, then return to the deep dive with an explicit
    statement of the trade you are making. Something like: "Briefly, I would ship
    it behind a flag with one team adopting first, and the coordination cost is the
    shared component version. I would rather spend the remaining time finishing the
    caching dive, because that is where the design risk is, unless the rollout is
    what you want to probe."

    The reasoning is that at senior level the deep dive is the primary signal and
    the organisational question is secondary, so abandoning the dive to give a
    thorough rollout answer trades a strong signal for a weak one. But refusing to
    answer reads as inflexibility, and the interviewer may have asked because they
    are probing exactly this.

    The move that makes both work is the explicit check in. It gives the
    interviewer the chance to redirect you, which is information you cannot get any
    other way, and it demonstrates clock management, which is itself a senior
    signal.

    The wrong answers are: switching topics silently and never returning, which
    loses the dive; ignoring the question, which reads as not listening; or giving
    a long organisational answer, which at senior level is the lower value use of
    ten minutes.

**Q2.** A candidate opens with "before I design anything, I have five questions",
asks all five, receives the answers, and then produces a design that would have
been identical regardless of any of them. What level does this read as, and why?

??? note "Show answer"

    It reads as mid, and often as slightly worse than a candidate who asked
    nothing, because it demonstrates that the clarifying step has been learned as a
    ritual rather than as a tool.

    The diagnostic property of a good clarifying question is that its answers lead
    to visibly different designs. That is exactly the structure of a branch table:
    if the answer is A the design becomes this, if B it becomes that. A question
    whose answers do not branch is consuming interview minutes to perform a
    behaviour.

    There is a second cost that candidates underestimate. Five questions at the
    start, answered and then unused, means the interviewer has watched the
    candidate ignore information they just requested. That is a worse signal about
    listening than never asking.

    The repair is mechanical: for each question you plan to ask, say the branch out
    loud. "Is this mostly read or mostly write, because if writes are within about
    5x of reads I would take a duplex transport and otherwise I would not." Now the
    question is doing work, the interviewer can see the reasoning, and even an
    unhelpful answer has advanced the design.

### 18g. When not to use this

**Answering above your level.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You are interviewing for the higher level, or the interviewer has explicitly asked about organisation, migration or team boundaries |
| Scale threshold below which it is counterproductive | In a mid or senior round, minutes spent on adoption governance are minutes not spent on the technical depth those levels are graded on. The rubric does not award points you did not attempt to earn |
| Cheaper alternative | Answer the level you are in, thoroughly, and let depth do the work. A senior answer delivered completely outscores a staff framing delivered partially |

**Opening with a reframing of the prompt.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You can name the specific decision the reframing changes, and you can still land a concrete design in the remaining time |
| Scale threshold | If the prompt is already well scoped, reframing consumes minutes and signals that you did not accept a reasonable brief. Not every prompt has a hidden premise |
| Cheaper alternative | Accept the prompt, then state one assumption that would change the design if wrong, and proceed. Same benefit, a tenth of the time |

**The level delta as a preparation tool.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You have practised a full answer end to end and want to find what your answers are missing, which is what the dimension table is for |
| Scale threshold | Reading the level table before you can produce a complete answer at any level teaches vocabulary rather than judgement, which produces the ritual described in the second check yourself question |
| Cheaper alternative | Answer a prompt fully, record yourself, and mark your own transcript against the dimension table afterwards. Diagnosis after the attempt, not a script before it |

---

## Case study 1: News feed application

Four assumptions run through every case study on this page. They exist so that every latency, byte and memory claim below is arithmetic you can redo with your own measurements instead of a number you have to trust.

| Assumption | Value | Why it is reasonable |
|---|---|---|
| Frame budget | 16.7 ms per frame at 60 Hz, of which about 8 ms is available to our JavaScript | 1000 divided by 60 is 16.7. Roughly half is consumed by style, layout, paint and compositing, which is the larger share on a mid-tier phone. Measure yours, then re-run every division below |
| Target network profile | 1.6 Mbps effective downlink, which is 200 KB per second | A common slow-4G throttling profile. Pessimistic at a desk, optimistic on a train. It is a stated profile, not a claim about your users |
| Mid-tier device JavaScript cost | about 1 second of main-thread work per 1 MB of uncompressed JavaScript parsed and executed | A placeholder anchored to mid-tier phones running several times slower than a developer laptop. Re-measure on your lowest supported device in week one |
| Forced synchronous layout | about 4 ms per forced reflow over a tree of a few thousand elements | A placeholder. What matters is its ratio to the 8 ms budget, not the absolute value |

Say all four out loud in the room. A number offered as a placeholder you will measure reads as engineering. The same number stated as fact reads as recall.

### The prompt (news feed)

"Design the front end of a news feed: an infinite list of posts that a user scrolls, likes and comments on."

### Questions that fork the design (news feed)

| Question worth asking | If the answer is A the design becomes | If the answer is B the design becomes |
|---|---|---|
| Is the feed identical for everyone, or personalised per user? | Identical: the first page of HTML is one cache entry at the edge, first paint is a cache hit, and the client never renders page one | Personalised: per-user HTML cannot be cached at the edge, so you either stream a shell with personalised holes or accept a client render for page one and design around the empty first frame |
| Is the order append-only by time, or ranked and mutating? | Append-only: a cursor is stable, page N always means the same posts, and duplicates are impossible | Ranked and mutating: the same post can appear on page 2 and again on page 5, so the client owns dedup by identifier and the server owns a snapshot token so pagination is consistent within a session |
| Are post heights fixed or arbitrary? | Fixed: the window is index arithmetic, total height is a multiplication, and the scrollbar is exact | Arbitrary (images, embeds, expandable text): you inherit measurement, estimation error and scroll anchoring, which is deep dive one |
| Must individual posts be linkable and indexable? | No, it is a logged-in app: client rendering is fine and the whole search-visibility branch disappears | Yes: each post needs a standalone server-rendered route, and the feed's client behaviour must not be the only path to the content |
| Is the media static images or autoplaying video? | Images: cost is bytes and decode, both solvable with sizing and lazy loading | Video: you add a playback controller driven by intersection, a policy for cellular, and a bandwidth budget that competes directly with pagination fetches |
| Can the user act while offline or on a dead connection? | No: mutations are plain requests with a pending state, and failure is a toast | Yes: you need a durable outbox with idempotent intents, which is deep dive two |
| Is the like button optimistic or confirmed? | Confirmed: simpler code, but perceived latency equals the round trip and rapid taps produce ambiguous state | Optimistic: instant feedback, and you now own coalescing, rollback and read-cache reconciliation |
| Which devices and assistive technologies are in scope? | Evergreen desktop only: you can lean on platform features and skip most of the node-count work | Mid-tier Android plus screen readers: element count and the conflict between windowing and assistive technology become the two hardest constraints on the page |

### Requirements (news feed)

Functional:

- Render a continuously extending list of posts, fetching the next page before the user reaches the end.
- Like and unlike from within the list, with the change visible immediately.
- Open a post detail view and return to the feed with the same posts loaded and the same item under the eye.
- Operate fully by keyboard, and expose position and set size to a screen reader without lying about either.
- Survive a failed mutation, a slow network and a backgrounded tab without losing the user's action or their place.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Interaction to next paint, 75th percentile | under 200 ms | GIVEN by the product, matching the published "good" threshold for the metric (see the sources section) |
| Largest contentful paint, 75th percentile | under 2.5 s on the target network profile | GIVEN, same source |
| JavaScript budget for the feed route | 170 KB compressed | DERIVED below from the 2 s interactive budget |
| JavaScript per frame during scroll | 8 ms | DERIVED from the 16.7 ms frame at 60 Hz with half reserved for the rest of the pipeline |
| Posts scrolled per session, 75th percentile | 60 | ASSUMED; a casual feed session is a few screens, and the number only sets where V0 stops being enough |
| Posts scrolled per session, 99th percentile | 500 | ASSUMED; heavy users exist and they are the ones whose tab gets discarded |
| Posts per page request | 20 | GIVEN by the API contract |
| DOM elements per post | 40 | ASSUMED: avatar, author, timestamp, body, media, four action controls and their labels and wrappers |
| Average rendered post height | 400 px on a 800 px viewport | ASSUMED; it makes the scroll arithmetic concrete and you can substitute your own |
| Accessibility conformance | WCAG 2.2 level AA | GIVEN |

### Estimation that eliminates (news feed)

| Calculation | Result | What it rules out |
|---|---|---|
| 60 posts times 40 elements | 2,400 elements at the 75th percentile session | Rules out virtualization as a day-one requirement. Most sessions never build a tree that any device struggles with, and a windowing implementation you cannot justify is the most expensive kind of code |
| 500 posts times 40 elements | 20,000 elements at the 99th percentile session | Rules out unbounded retention. Whatever you build must have a ceiling, even if the ceiling is a crude one |
| 800 by 450 px image at 4 bytes per pixel | 1.44 MB decoded per image, 720 MB across 500 posts | Rules out keeping every post mounted with its image attached. This single line, not element count, is what actually kills a long feed session on a phone |
| 20 posts times 1.5 KB of JSON, divided by 200 KB per second | 30 KB, 0.15 s per page | Rules out optimising the JSON payload first. It is under a fifth of a second and it is not your problem |
| 20 posts times 80 KB of image, divided by 200 KB per second | 1.6 MB, 8 s per page | Rules out loading a page's images when the page's JSON arrives. Images are 53 times the payload, so image loading must be driven by the viewport, never by pagination |
| 170 KB compressed at 200 KB per second, plus 510 KB uncompressed at 1 s per MB | 0.85 s download plus 0.51 s parse and execute, 1.36 s total | Rules out a route budget over about 250 KB compressed if you want to stay under 2 s, and it rules out arguing about a 15 KB windowing helper before you have looked at the framework and the third-party tags |
| 3 viewport heights per second times 800 px, divided by 400 px per post | 2,400 px per second, 6 posts entering view per second | Rules out nothing alone, but it is the denominator for the next row |
| 60 frames times 8 ms, divided by 6 posts | 80 ms of main-thread budget per newly mounted post | Rules out per-post work heavier than 80 ms, which sounds generous until the next row |
| 30 mounted posts times 4 ms per forced reflow | 120 ms if you measure each mounted post individually after writing to it | Rules out interleaved reads and writes decisively: one naive measurement pass costs 150 percent of the entire per-post budget. Measurement has to be batched into a single reflow per frame |
| 800 ms 95th percentile like round trip, at roughly one tap per 300 ms while scrolling | two to three further taps land before the first response | Rules out disabling the control until the server confirms. The user will out-type your network |

The 150 ms median and 800 ms 95th percentile round trip are ASSUMED. They are the shape of a mobile write to a nearby region, and the design below only depends on the tail being several hundred milliseconds, not on the exact figure.

### V0, the simplest thing that works (news feed)

**Predict first: a competent engineer proposes shipping this feed with no windowing, no client cache library and no optimistic updates. What is the first thing that breaks, and what number tells you it broke?**

??? note "Show answer"

    Not scroll performance, which is the intuitive answer. The first break is memory on long sessions: 500 posts holding attached images is 720 MB of decoded pixels by the arithmetic above, and the operating system discards the tab.

    The number that tells you is not a frame metric at all. It is the rate of sessions that end in a page reload from a restored tab, segmented by scroll depth. Frame drops are the second failure and they arrive later, because 20,000 elements still lay out in a few frames if you are not touching them.

```
+-----------+     +-------------------+     +----------------+
| Server    | --> | Feed route HTML   | --> | Post list DOM  |
| renders   |     | page 1 inlined    |     | append only    |
| page 1    |     +-------------------+     +----------------+
+-----------+                                       ^
                  +-------------------+             |
                  | Bottom sentinel   | --- fetch --+
                  | observes viewport |   page N
                  +-------------------+
```

**Caption.** V0 renders the first page on the server, appends every subsequent page to a single growing list in the DOM, and triggers the next fetch from an intersection observer on a sentinel element near the bottom of the list; images use the platform's lazy loading attribute and no page is ever removed.

Why this is correct at a real product's scale:

- The 75th percentile session ends at 2,400 elements. There is no measurement, no spacer arithmetic and no estimation error to be wrong about.
- If the feed is a real navigation rather than a client-side route, the browser restores scroll position on back navigation for free, and the back-forward cache can restore the whole page. Scroll restoration is the hardest part of this problem and V0 does not have it.
- Every post is in the accessibility tree, in document order, with real headings and real links. A screen reader user gets a working page with zero accessibility code.
- Lazy loading on the image element already defers the 8 s of image bytes per page to the viewport, which is the largest single win available and costs one attribute.

!!! warning "When not to move off the plain growing list"

    Below roughly 150 mounted posts at the 95th percentile, or when interaction to next paint stays under 200 ms in the deepest scroll decile, windowing is over-engineering. The measurement that justifies moving is interaction to next paint segmented by scroll-depth decile, plus the rate of tab discards or restored-session reloads. The cheaper intermediate step, before you own a windowing implementation, is `content-visibility: auto` with `contain-intrinsic-size` on each post, which lets the browser skip layout and paint for off-screen posts while keeping them in the DOM and in the accessibility tree.

### Breaking point, then V1 (news feed)

**What fails: the long session, and it fails on memory before it fails on frames.** At 500 posts the tab holds 20,000 elements and, if the images stay attached, several hundred megabytes of decoded pixels. The operating system reclaims the tab, the user comes back to a reload, and they land at the top of a feed they had scrolled for ten minutes.

How you would observe it, in order of how early the signal appears:

| Signal | How to collect it | What it means |
|---|---|---|
| Mounted post count at session end, 95th percentile | One counter reported on page hide | Tells you whether you are anywhere near the problem before users complain |
| Reloads that begin at scroll depth zero after a session that reached depth over 100 | Compare the entry scroll position against the previous session's exit depth | The discard signature. Users will describe it as "the app forgot where I was" |
| Interaction to next paint by scroll-depth decile | Real user monitoring, bucketed | A rising line across deciles is the element-count cost showing up |
| Long tasks per scroll second | Long task observation, normalised by scroll time | Separates "the page is big" from "we run expensive code on scroll" |

V1 mounts a window instead of the whole list:

```
+---------------+   +------------------+   +------------------+
| Scroll        |-->| Window control   |-->| Mounted window   |
| container     |   | first, last, pad |   | 15 posts + over  |
+---------------+   +------------------+   +------------------+
        |                    ^                      |
        v                    |                      v
+---------------+   +------------------+   +------------------+
| Top spacer    |   | Height cache     |<--| ResizeObserver   |
| Bottom spacer |   | id -> px         |   | measures mounted |
+---------------+   +------------------+   +------------------+
                             ^
                    +------------------+
                    | Page cache       |
                    | cursor -> posts  |
                    +------------------+
```

**Caption.** V1 keeps the full list of post identifiers in memory but mounts only the posts intersecting the viewport plus an overscan band; two spacer elements above and below reproduce the height of the unmounted posts from a height cache that a resize observer keeps up to date, and fetched pages are cached by cursor so scrolling back up does not refetch.

The window sizing, which is where the design becomes real:

| Quantity | Value | How it follows |
|---|---|---|
| Posts intersecting an 800 px viewport | 2 | 800 divided by 400 px per post |
| Scroll speed on a fast flick | 2,400 px per second | 3 viewport heights per second, stated in the assumptions |
| Overscan needed to survive a 330 ms stall | 2 posts each direction | 0.33 s times 2,400 px per second is 800 px, which is 2 posts |
| Mounted posts | about 15 | Two visible plus generous overscan, rounded up so that a fast flick with a slow frame still paints content rather than a spacer |
| Mounted elements | 600 | 15 posts times 40 elements, down from 20,000 |

The rule to take away: overscan is not a taste setting. It equals the stall you are willing to survive, times the scroll speed you support, divided by the item height. State the stall you chose and why.

!!! warning "When not to build windowing yourself"

    A windowing implementation owns estimation error, spacer arithmetic, scroll anchoring, focus retention and set-size reporting. Below the thresholds above it buys nothing. Even above them, first try the two cheaper moves: `content-visibility: auto` on posts, and detaching media (not the whole post) outside a retention band, which recovers the 720 MB without touching element count at all. Only when element count itself is measurably costing frames does the window earn its complexity.

### Breaking point, then V2 (news feed)

**What fails: the window is right and the scroll position is wrong.** Two symptoms, one cause. The cause is that the height cache is a guess until a post is measured, and posts above the viewport change height after they are measured, when an image loads or a font swaps or text expands.

| Symptom | The number that shows it | Why it happens |
|---|---|---|
| The content under the eye jumps while scrolling upward | Layout shift attributed to the feed container, and a spike in scroll events with a large delta the user did not cause | A spacer above the viewport shrinks or grows when a real measurement replaces an estimate |
| Coming back from a post detail lands in the wrong place | Restoration error in pixels: the difference between the offset you asked for and the offset the user ends up at, at the 90th percentile | You saved a pixel offset, but the pixel offset means nothing once the heights above it have been re-estimated |

The estimation error is quantifiable and it is large. With 500 posts, a 400 px estimate and a 460 px true mean, the total height is wrong by 500 times 60 px, which is 30,000 px, or 37 viewport heights. A scrollbar that lies by 37 screens is a broken scrollbar, and every pixel-based restoration built on it is broken too.

V2 replaces pixel offsets with an anchor, and tells the truth to assistive technology:

```
+----------------+  +-------------------+  +-----------------+
| Anchor state   |->| Window control    |->| Mounted window  |
| id + offset px |  | union with focus  |  | aria-posinset   |
+----------------+  +-------------------+  +-----------------+
        ^                    |                      |
        |                    v                      v
+----------------+  +-------------------+  +-----------------+
| Session snap   |  | Height cache      |  | Media retention |
| cursors+anchor |  | id -> px          |  | band, +-2 pages |
+----------------+  +-------------------+  +-----------------+
```

**Caption.** V2 stores the scroll position as an anchor, meaning a post identifier plus the pixel offset within that post, persists the loaded cursors and the anchor to session storage so a restored tab can rebuild, forces the mounted window to include whichever post contains keyboard focus, keeps decoded media only within a retention band around the anchor, and stamps each mounted article with its position and the set size.

The three changes and what each one buys:

| Change | What it fixes | What it costs |
|---|---|---|
| Anchor instead of pixel offset | Restoration is exact regardless of estimation error, because you scroll until the anchored post's top is at the recorded offset | You must be able to find the anchor post, which means the cursors that contain it have to be refetched before you can restore |
| Session snapshot of cursors plus anchor | A discarded tab rebuilds to the same place instead of the top | One write per settled scroll, plus a size limit and an expiry, since session storage is small and synchronous |
| Media retention band | Removes the 720 MB failure entirely by detaching image sources outside two pages either side of the anchor | Re-entering a detached band costs a decode, which is why the band is two pages and not zero |

!!! warning "When not to persist a session snapshot"

    If the feed is ranked and the server cannot give you a stable snapshot token, restoring old cursors reconstructs a feed that no longer exists, and the user is returned to a position that is subtly wrong in a way they cannot articulate.

    The measurement that justifies persistence is the share of sessions that end in a discard or a reload after depth 50. Below a few percent, restore to the top with a "jump to where you were" control and let the user decide, which is honest, cheap and often preferred.

### Deep dive one: the variable-height window, and the lie it tells assistive technology

Windowing is usually presented as a rendering optimisation. It is really two problems that happen to share an implementation: keeping the geometry honest, and keeping the accessibility tree honest. The second one is the part that gets shipped broken.

**Part one: three ways to know how tall a post is.**

| Strategy | What it needs | What it costs | Where it breaks |
|---|---|---|---|
| Fixed height, enforced by clamping content | Nothing at runtime | A truncated feed, which is a product decision not a technical one | Not at all, technically. It breaks the product if posts carry media of varying aspect ratio |
| Estimate, then correct on measure | One estimate per post type and a height cache | Total height is wrong until every post has been measured, by 37 viewport heights in the arithmetic above | Scrollbar position and any pixel-based restoration, both silently |
| Measure on mount, anchor the scroll | A height cache plus an anchor, and one batched measurement per frame | You give up an exact scrollbar in exchange for an exact reading position | Nothing under the eye, which is the only thing users notice |

The third row is the answer for a feed. Users do not read scrollbars in an infinite list; they notice content moving. Optimise the thing that is observable.

**Part two: measure without paying for a reflow per post.** The estimation above showed that measuring 30 mounted posts one at a time costs 120 ms against an 80 ms budget. Two rules follow:

- Never read geometry inside the same loop that writes styles. Collect all reads, then all writes, so one reflow serves the whole batch.
- Prefer an observer that reports size changes asynchronously over a synchronous geometry read. A resize observer also catches the late changes a one-shot read cannot see: an image finishing decode, a web font swapping, a translated string wrapping to a second line.

**Part three: the browser is already anchoring, and it will fight you.** Browsers implement scroll anchoring, which adjusts the scroll offset when content above the viewport changes size, precisely so that this class of jump does not happen. A windowing implementation mutates spacer heights above the viewport constantly, which is exactly the input that feature reacts to. Two coherent positions exist, and mixing them is the bug:

| Position | What you do | When it is right |
|---|---|---|
| Let the browser anchor | Never change a spacer height above the current anchor within a frame; only correct heights below the viewport | Simple lists where corrections are small and rare |
| Own the anchoring | Set `overflow-anchor: none` on the scroll container and apply your own compensating scroll adjustment in the same frame as the height correction | Any list where you already store an anchor, which is V2 |

There is no third position. If you correct heights above the viewport and leave browser anchoring on, two mechanisms compensate for the same delta and the content moves by twice the correction.

**Part four: what a window does to a screen reader.** A windowed list of 500 posts contains 15 posts. Everything that follows is a consequence of that sentence.

| Problem | What the user experiences | The mechanism that fixes it |
|---|---|---|
| The set size is wrong | The virtual cursor reaches the end of the window and reports the end of the list | `role="feed"` on the container with each post as an `article`, carrying `aria-posinset` for its true index and `aria-setsize` for the loaded total, or -1 when the total is unknown |
| Loading is invisible | The user waits with no indication that more is coming | Set `aria-busy="true"` on the feed while inserting or removing articles, then back to false. This is the announcement channel for a feed, not a live region |
| The virtual cursor runs off the end | Reading stops even though more content exists | Load the next page before the boundary, and document the keyboard interface: Page Down and Page Up move article to article, Control plus Home and Control plus End leave the feed |
| Focus is destroyed | Focus falls to the document body, the reading position is lost, and the next Tab starts at the top of the page | The mounted window is the union of the visual window and the post containing the currently focused element. Never unmount the ancestor of the active element |
| Announcements flood | Every appended page speaks over whatever the user was reading | No live region for content arrival. The busy state carries it |

The focus rule deserves its own sentence, because it is the same failure class as the dialog return path in case study 4: any code that can remove the element containing focus owes a policy for where focus goes next. Here the policy is trivial, keep it mounted, and it is still the most common thing missing from hand-rolled windowed lists.

**Part five: the honest limitation.** A feed does not give assistive technology users a real total, does not give them a way to jump to item 400, and does not let find-in-page reach unmounted posts. Those are real regressions against the plain list in V0, and they are the price of the memory fix. Say so in the room. The mitigation is a text search that queries the server rather than the DOM, plus a keyboard shortcut back to the top.

!!! warning "When not to window a list at all"

    If the maximum realistic length is a few hundred items of modest markup, use the plain list plus `content-visibility: auto`. The measurement is mounted element count at the 95th percentile and interaction to next paint by scroll depth. If element count stays under a few thousand and interaction latency is flat across deciles, windowing costs you find-in-page, native scroll restoration and accurate set size in exchange for nothing.

### Deep dive two: the mutation outbox, coalescing and what optimistic really costs

The naive optimistic like is four lines: flip the local value, send the request, revert on error. It handles the happy path and one failure. There are three others, and each one produces a bug report that reads like a ghost story.

| Failure | What the user sees | Why the four-line version cannot fix it |
|---|---|---|
| Rapid toggling | The heart ends up in the wrong state after tapping twice quickly | Two requests are in flight; whichever response lands last wins, and that is not necessarily the one the user meant |
| Refetch overwrite | The heart fills, then empties a second later with no error | A feed page fetch that started before the like was processed returns the old value, and the read cache overwrites the optimistic value |
| Navigation loss | A comment vanishes | The request was in flight when the route changed and the component that owned the retry was unmounted |

**The shape of the fix: intents, not effects.** Model each user action as a durable intent with an identity, and let a queue own delivery.

| Element | Definition | Why it is this way |
|---|---|---|
| Intent key | (entity id, field), for example (post 91, liked) | Everything that follows depends on being able to say "these two actions are about the same thing" |
| Payload | The desired absolute state, `liked: true`, never a delta such as `increment` | Absolute state is idempotent and coalescible. A delta is neither, so a retried delta double-counts and two deltas cannot be merged |
| Idempotency key | One identifier per intent, regenerated only when the desired value changes | Lets the server discard a duplicate delivery. This is the same contract described in the product architecture course, applied from the client side |
| Coalescing rule | The queue holds at most one pending intent per intent key, replaced in place | Removes the rapid-toggle race by construction rather than by ordering luck |
| Revision | A monotonically increasing local counter per entity, stamped when an intent is enqueued | Gives the read cache a rule for whose value wins |

**Reconciliation with the read cache, which is the part people skip.** Three workable rules, and they are not equivalent:

| Rule | How it works | Cost | When to choose it |
|---|---|---|---|
| Dirty flag | While an entity has pending intents, server values for that entity are ignored | One boolean per entity, trivially correct | Default. Choose it unless you have a specific reason not to |
| Revision compare | Server payloads carry a version; drop any server value older than the last acknowledged local write | Requires the server to return a version the client can compare | When several clients edit the same entity and you want their edits to land promptly |
| Refetch after settle | Accept overwrites, then refetch the entity once the queue drains | Simplest to reason about, one extra request per settled entity | When entity payloads are tiny and correctness matters more than request count |

**What coalescing is worth, as a number.** Assume 8 percent of impressions receive a like and 15 percent of those are toggled again within a second. Coalescing removes those second requests, so write volume falls by 15 percent of the like traffic. That is the small benefit. The large benefit is that it deletes an entire correctness class: with one pending intent per key there is no ordering to get wrong, so no amount of network reordering can produce a wrong final state.

**Rollback is a user interface decision, not a technical one.** Silently reverting is a lie: the count animates back and the user concludes the app is broken. Choose by consequence class:

| Consequence class | Example | Policy |
|---|---|---|
| Reversible, low value, user can repeat in one tap | Like, follow, save | Retry quietly with backoff. If it finally fails, revert and show one non-blocking message. Do not interrupt scrolling |
| User-authored content | Comment, reply | Never discard silently. Keep the text, keep the intent durable across a reload, and surface a retry affordance attached to the item |
| Externally consequential | Purchase, delete, share to another account | Do not be optimistic at all. Show a pending state and wait. An incorrect optimistic paint here is a trust incident |

**Durability, priced.** Persisting the queue to IndexedDB survives a tab close. Assume 5 ms per write. At the like rate above, on a session of 500 impressions, that is 40 likes, 200 ms of storage work spread across minutes: free. But persist only what is expensive to recreate. A lost like costs one tap. A lost comment costs a paragraph the user will not retype. Persist authored content, keep toggles in memory, and say why in the room.

**The accessibility contract of an optimistic control.** Three rules, each of which is commonly broken:

- The like button is a toggle button with `aria-pressed`. Its accessible name stays "Like" in both states. Flipping both the name and the pressed state announces the change twice and reads as a bug.
- Do not put the like count in a live region. Counts change because other people act, and a live region would speak continuously while the user is trying to read.
- While an intent is pending, do not disable the control and do not remove it from the tab order. Disabling a focused element moves focus to the body. Keep it focusable and let the pressed state reflect the optimistic value, which is what the user asked for.

!!! warning "When not to update optimistically"

    Optimistic updates are wrong when the server can legitimately return something other than what you painted: a comment that gets held for review, a follow that requires approval, a value the server normalises. They are also unnecessary when the 95th percentile round trip is under about 100 ms, because a pending state that resolves within one or two frames is honest and costs nothing.

    The measurement that justifies going optimistic is the 95th percentile mutation round trip on your worst supported network, compared against the interaction budget. If it is under 100 ms, ship the pending state and delete the queue.

### Failure modes and evaluation (news feed)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Memory eviction | The tab is discarded on long sessions and the user returns to the top of a feed they had scrolled for ten minutes | The media retention band from V2, plus the session snapshot so a discard is recoverable rather than fatal |
| Retry storm from a sentinel | The connection fails, the sentinel stays intersecting, and the fetch retries every frame, so the client attacks the API during an outage | The sentinel triggers at most one in-flight request, failures back off with jitter, and the sentinel is disarmed until the user scrolls again |
| Duplicate and missing items | The same post appears twice, or a post is skipped, on a ranked mutating feed | Dedup by identifier on merge, and a snapshot token from the server so a session paginates a consistent view. This is the client half of the pagination contract |
| Layout shift storm | Content jumps under the eye during scroll | Reserve space with intrinsic size for media, own the anchoring explicitly per deep dive one, and never correct heights above the anchor with browser anchoring left on |
| Focus loss | A keyboard or screen reader user is thrown to the top of the page mid-read | The window includes the focused post, and any code path that unmounts content states where focus goes |
| Live region flooding | A screen reader user hears "loading" repeatedly and cannot read | Content arrival is signalled with the busy state on the feed, never a polite live region on every append |
| Bandwidth starvation | Pagination stalls because images are saturating the connection | Images are viewport-driven and low priority, the next page fetch is high priority, and the retention band bounds concurrent image requests |
| Cache key leakage | A user sees another user's feed after a shared edge cache serves a personalised response | The feed route is either uncacheable at the edge or split into a cacheable shell plus a private data request. Never vary a shared cache on a header you do not control |

Service level indicators:

- Interaction to next paint at the 75th percentile, segmented by scroll-depth decile. The segmentation is the point: an average hides a design that degrades with depth.
- Frames longer than 16.7 ms per second of active scrolling, reported separately from idle time.
- Mounted post count and mounted element count at the 95th percentile, sampled on page hide.
- Restoration error in pixels at the 90th percentile, defined as the distance between the anchor's requested offset and its achieved offset after restore.
- Mutation settle time at the 95th percentile, from intent enqueued to server acknowledgement, plus queue depth at the 99th percentile.
- Duplicate item rate per session and empty-page rate, which catch pagination faults that no latency metric sees.
- Accessibility assertions in continuous integration: an automated audit of the feed route with the window at the top, the middle and after a restore, plus a keyboard-only path test that likes a post and returns from a detail view.

Alert on: interaction to next paint regressing in the deepest decile while the shallow deciles are flat, which is the signature of a retention regression; sentinel fetches per session exceeding a small multiple of pages actually rendered, which is the retry storm; mutation queue depth at the 99th percentile rising, which means the write path is failing before any error rate does; and restoration error crossing a stated pixel threshold.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Interaction to next paint under 200 ms | Yes at the mounted counts V1 produces, because the per-frame work is bounded by 15 mounted posts rather than by session length. It must be verified per decile, not in aggregate |
| Largest contentful paint under 2.5 s | Yes for the server-rendered first page. A client-rendered first page on the stated network profile spends 1.36 s on JavaScript alone before it can ask for data, which leaves no margin |
| 170 KB route budget | Yes, but only if the windowing implementation, the cache and the framework together stay well under it. This is the requirement most likely to be quietly broken |
| Return to the same item after a detail view | Yes, via the anchor. Exact within one post, not exact to the pixel, and that is the correct target |
| No lying to assistive technology | Partly. Position and set size are honest for loaded posts; the true total is unknown by construction and is reported as unknown rather than guessed |
| Survive a failed mutation | Yes for likes, which retry, and for comments, which persist. Externally consequential actions are deliberately not optimistic |

### Where this answer is a simplification (news feed)

Every link below was checked on 1 August 2026.

- **ARIA Authoring Practices Guide, Feed pattern, `https://www.w3.org/WAI/ARIA/apg/patterns/feed/`.** The primary description of `role="feed"`, `aria-posinset`, `aria-setsize`, the `aria-busy` protocol during insertion, and the Page Down and Page Up keyboard model this case study relies on.
- **CSS Scroll Anchoring Module Level 1, `https://drafts.csswg.org/css-scroll-anchoring/`.** Defines the anchor selection algorithm and `overflow-anchor`, which is the feature a windowed list is silently fighting.
- **MDN, `content-visibility`, `https://developer.mozilla.org/en-US/docs/Web/CSS/content-visibility`.** The cheaper alternative to windowing, including `contain-intrinsic-size` and the fact that skipped content stays in the accessibility tree and reachable by find-in-page.
- **web.dev, Interaction to Next Paint, `https://web.dev/articles/inp`.** Source of the 200 ms and 500 ms thresholds and the fact that the metric is assessed at the 75th percentile.
- **WCAG 2.2, `https://www.w3.org/TR/WCAG22/`.** The conformance target, and the source of the target size and pause-stop-hide criteria used in later case studies.
- **What this answer skips entirely.** Ranking, fan-out and the read path behind the feed are a separate design; see the news feed case study in [modern system design](modern-system-design.md) for the server side and the [ML system design](ml-system-design.md) course for how the order is produced. The client half of this problem on a native mobile client, where the operating system reclaims memory far more aggressively, is treated in [mobile system design](mobile-system-design.md).
- **Real feeds are not one list.** Interleaved advertisements, story trays, suggested-follow modules and inline video each have different heights, different retention needs and different accessibility semantics. A homogeneous list is a teaching simplification, and a good interviewer will insert an advertisement every ten posts to see whether your height cache and your set size survive it.
- **Snapshot pagination is harder than one token.** Holding a consistent ranked view for a session costs the server memory or a stable ordering key. The honest client-side answer is that duplicates will happen and you must dedup, not that the server will prevent them.

### Level delta (news feed)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Draws a virtualized list immediately, because that is what a feed question is known to be about | Starts from a plain growing list and names the number at which it stops working | Asks whether the feed is personalised and whether it mutates before drawing anything, because those two answers decide caching, pagination stability and whether page one can be server-rendered at all |
| Why virtualize | "Too many DOM nodes" | Computes 20,000 elements at the 99th percentile session | Computes 720 MB of decoded images and shows that memory, not element count, is the actual failure, then fixes it with a media retention band before touching element count |
| Overscan | Picks a number that felt right | Explains that overscan trades nodes for blank frames | Derives it: stall tolerance times scroll speed divided by item height, and states the stall tolerance chosen |
| Measurement | Reads geometry on mount | Batches reads and writes | Prices the naive version at 120 ms against an 80 ms per-post budget, then uses an observer so late layout changes are caught at all |
| Scroll restoration | Saves the scroll offset | Saves offset plus loaded pages | Shows that a pixel offset is meaningless when the total height is wrong by 37 viewport heights, switches to an anchor, and names what the anchor cannot restore |
| Optimistic writes | Flips local state, reverts on error | Adds a queue and rollback | Sends absolute state rather than deltas so intents are coalescible, gives the read cache an explicit precedence rule, and chooses rollback behaviour per consequence class |
| Accessibility | Mentions ARIA labels | Names the feed role and set size attributes | Treats the window as a lie to assistive technology, fixes position, set size, busy state and focus retention, and then volunteers the regressions the window causes that cannot be fixed |
| Honesty | Presents the design as complete | Notes a couple of limitations | States which requirement the design is most likely to violate quietly, the route byte budget, and proposes the guard that would catch it |

What each level added:

- **Mid to senior:** the design stops being a remembered pattern and starts being derived from a measurement, and the first thing offered is the smallest thing that works.
- **Senior to staff:** every threshold becomes a formula with its inputs stated, the failure that actually kills the feature is separated from the failure everyone talks about, and the design's own weak point is volunteered before the interviewer finds it.

---
## Case study 2: Autocomplete component

Case study 1 was application scale: routes, caches, sessions. This one is component scale, and the shift changes what "good" means. Nobody will ask you how it scales to a million users. They will ask what happens when the user types faster than the server answers, and what a screen reader hears. The four assumptions from case study 1 still apply.

### The prompt (autocomplete)

"Design an autocomplete input that suggests results as the user types, and make it reusable across our products."

### Questions that fork the design (autocomplete)

| Question worth asking | If the answer is A the design becomes | If the answer is B the design becomes |
|---|---|---|
| Is this a search box that submits, or a picker that commits a value into a form? | Search: free text is a valid final answer, Enter submits whatever is typed, and suggestions are hints the user may ignore | Picker: there is a committed value distinct from the visible text, so blur, clear and invalid-text semantics dominate the design and Enter must be unambiguous |
| Single value or multiple, with chips? | Single: the value is a scalar and the keyboard model is the standard one | Multiple: you add chip removal semantics, Backspace behaviour on an empty input, an announced selected count, and a second focus region inside the same control |
| Is the suggestion source remote, local, or both? | Local: filter synchronously, there is no network design at all, and debouncing would only add latency | Remote: debounce, cancellation, caching and out-of-order handling, which is deep dive one |
| Can the user type faster than the server answers? | No, the source is local or the 95th percentile is well under 100 ms: render every keystroke | Yes: results must be matched to the query that produced them, because correctness now depends on ordering |
| Is it controlled by the consumer, or self-contained? | Self-contained: internal state, small API, no synchronisation bugs | Controlled: you owe value and change handlers, and you must decide what happens when the consumer sets a value the suggestion list does not contain |
| Must it support composition input methods and virtual keyboards? | No: key handling is straightforward | Yes: composition events gate both the search trigger and the Enter key, otherwise the control commits a suggestion while the input method's candidate window is open, which makes it unusable in several languages |
| Does the consumer render its own option markup? | No: you own the markup and can guarantee the accessibility contract | Yes: the component becomes headless and the contract moves into the API surface, which is deep dive two |
| What happens when there are zero results? | Nothing renders: sighted users infer it from the closed popup and screen reader users get silence | An explicit empty state: the popup stays open with a message, and the result count is announced, which is the only way a non-visual user learns that the query failed |

### Requirements (autocomplete)

Functional:

- Suggestions reflect the most recent input, and a slower earlier response never replaces a newer one.
- The control is fully operable by keyboard: open, move, select, dismiss, and escape without committing.
- A screen reader user receives the number of results, the currently active option, and the committed value.
- Consumers can render their own option content without being able to break the accessibility contract.
- Mouse, touch and keyboard produce identical committed values and identical events.
- The user can always submit text that matches no suggestion, or the search variant of the control is broken.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Interaction to next paint, 75th percentile | under 200 ms | GIVEN, the published "good" threshold at the 75th percentile |
| Keystroke handler cost | under 8 ms | DERIVED from the 16.7 ms frame with half reserved for the rest of the rendering pipeline |
| Time from the user pausing to the first painted suggestion, 75th percentile | under 300 ms | ASSUMED. Justified below: it is roughly the pause between two typed words, so a slower list arrives after the user has moved on |
| Typing speed | 200 ms between keystrokes | DERIVED: 60 words per minute times 5 characters is 300 characters per minute, which is 5 per second |
| Median committed query length | 12 characters | ASSUMED; it sets request volume and nothing else |
| Suggestion service latency | 60 ms median, 250 ms at the 95th percentile | ASSUMED, the shape of an in-region prefix index. The design depends only on the tail exceeding the debounce window |
| Compressed bytes for the component | under 12 KB | ASSUMED, and defensible: it is one widget on a route with a 170 KB budget, so about 7 percent |
| Visible options at once | 10 | ASSUMED; beyond this the list stops being scannable and keyboard traversal stops being usable |
| Conformance | WCAG 2.2 level AA, ARIA 1.2 combobox pattern | GIVEN |

### Estimation that eliminates (autocomplete)

| Calculation | Result | What it rules out |
|---|---|---|
| 12 characters at 200 ms each | 2.4 s of typing per query, and 12 requests if you fire on every keystroke | Rules out request-per-keystroke. At 100,000 searches a day it is 1.2 million requests instead of roughly 300,000, for information the user never sees |
| Debounce of 150 ms against a 200 ms inter-keystroke interval | The timer expires before the next key arrives, every time | Rules out any debounce shorter than the inter-keystroke interval. It degenerates to one request per keystroke while adding 150 ms of latency to each one, which is the worst of both |
| 250 ms debounce plus 60 ms median service plus one frame | 326 ms to the first suggestion at the median | Rules out debouncing alone as the answer: it already misses the 300 ms target at the median, before the network has a bad day |
| 250 ms debounce plus 250 ms 95th percentile service plus one frame | 516 ms at the tail | Rules out treating the network as the only source of suggestions. Something must paint before the response, which is what makes the cache load-bearing rather than an optimisation |
| Three requests per query, each overlapping the next only if it outlives the 250 ms gap, at a 5 percent tail | 1 minus 0.95 cubed, which is 14.3 percent of queries containing an overlap | Rules out assuming responses arrive in the order they were sent. One query in seven can race, which is far too common to leave to luck |
| 5,000 local items at 30 bytes each | 150 KB, against a 12 KB component budget and a 170 KB route budget | Rules out shipping the corpus to the client above roughly 1,000 short items. Below that, at about 30 KB, the entire network design disappears and you should say so |
| 5,000 items divided by 26 first letters, then by 676 two-letter prefixes | 192 candidates after one character, 7 after two | Rules out querying on the first character: the result set cannot fit the 10 visible slots and the ranking is meaningless. A minimum length of 2 is derived, not a convention |
| 500 rendered options times 3 elements each, at an assumed 0.01 ms per element created | 15 ms per render | Rules out unbounded result lists: it is nearly double the 8 ms frame budget, and a list that long is unusable by keyboard anyway |
| Speech at 180 words per minute, so 3 words per second, against one 3-word announcement per keystroke at 5 keystrokes per second | The announcement queue grows by 4 seconds for every second of typing | Rules out announcing the result count on every keystroke, decisively. A screen reader user would hear results for a query they finished typing ten seconds ago |

The uniform distribution over first letters is an explicit ASSUMPTION and it is wrong in detail, since real corpora are skewed. It is right in magnitude, which is all the minimum-length decision needs.

### V0, the simplest thing that works (autocomplete)

**Predict first: for a country picker with 195 options, what does a hand-built combobox give the user that the platform's native one does not?**

??? note "Show answer"

    Custom visual design, custom option content, and control over filtering. That is the complete list. What it costs is the keyboard model, the announcement behaviour, the mobile presentation, the input method handling and the escape semantics, all of which the platform already implements correctly and which you will now maintain forever. For 195 static options with no design requirement, the platform control is the correct engineering answer, and being able to say that in an interview is worth more than reciting the ARIA pattern.

```
+------------------+        +--------------------+
| <input list=...> | -----> | <datalist> options |
| user types       |        | shipped in HTML    |
+------------------+        +--------------------+
         ^                            |
         +--- browser filters and ----+
              renders the popup
```

**Caption.** V0 is a plain text input associated with a native suggestion list whose options are delivered in the initial HTML; the browser owns filtering, popup presentation, keyboard interaction and announcement, and the page ships no JavaScript for the control at all.

Why this is correct for a real class of problems:

- Zero bytes. Against a 12 KB component budget, zero is a strong opening position.
- The keyboard model, the touch behaviour and the screen reader announcement are the platform's, which means they are consistent with every other control the user has ever operated.
- Free text still submits, because it is a real input. The most common bug in hand-built search comboboxes, trapping the user into choosing a suggestion, is impossible here.
- It degrades to a plain text field if anything fails, which is the correct failure mode for a search box.

!!! warning "When not to use the native suggestion list"

    It stops being viable the moment any of four things is true: the option set is remote, the option set exceeds roughly a few hundred entries, the design specifies the popup's appearance, or you must guarantee a specific announcement. The measurement is blunt and it is a product question, not a technical one: ask whether the design file specifies what the dropdown looks like. If it does, the native control is already ruled out, and you should say that out loud rather than discovering it in week three.

### Breaking point, then V1 (autocomplete)

**What fails: the source moves to the server.** Once options come from a network call, the native control has no mechanism to say "these options are for the text as it was 400 ms ago", no way to express a loading state, and no hook to cancel anything. The symptom users report is that the list shows suggestions for a prefix they have already typed past.

How you would observe it before users report it:

| Signal | How to collect it | What it means |
|---|---|---|
| Requests per committed query | Count suggestion requests between focus and submit, divided by submissions | Above about 5 means the debounce is not working. It is most often broken by a re-render that recreates the timer |
| Stale render rate | Share of painted result sets that are replaced within 100 ms by a different set | Directly measures how often the user sees a list that is already wrong |
| Suggestion click-through by position | Standard interaction logging | A collapse in click-through with no ranking change usually means users stopped trusting the list |

V1 builds the combobox explicitly:

```
+-----------+   +------------------+   +----------------+
| Input     |-->| Debounce timer   |-->| Fetch          |
| keystroke |   | trailing, 250 ms |   | suggest?q=...  |
+-----------+   +------------------+   +----------------+
      |                  ^                     |
      v                  |                     v
+-----------+   +------------------+   +----------------+
| Reducer   |<--| Sequence check   |<--| Response       |
| open,list |   | drop if stale    |   | results + seq  |
+-----------+   +------------------+   +----------------+
      |
      v
+-----------+   +------------------+
| Listbox   |   | Status region    |
| options   |   | count, polite    |
+-----------+   +------------------+
```

**Caption.** V1 runs each keystroke through a trailing debounce, tags every request with a monotonically increasing sequence number, discards any response whose sequence is lower than the highest already rendered, and feeds the surviving results into a reducer that owns whether the popup is open, which option is active and what the polite status region says.

The keyboard model, which is the actual product:

| Key | Behaviour | Why it is this and not something else |
|---|---|---|
| Down arrow | Opens the popup if closed and makes the first option active; otherwise moves to the next option | Opening on Down is how every native list behaves, so it is discoverable without documentation |
| Up arrow | Moves to the previous option; from the first option, returns to the input with no active option | Returning to the input rather than wrapping lets the user get back to editing without Escape |
| Home and End | Move to the first and last option when the popup has focus semantics, or to the text ends when it does not | Pick one and document it. Ambiguity here is a genuine design decision, not an oversight |
| Enter | Commits the active option if there is one, otherwise submits the typed text | Blocking submission when nothing is active is the single most common way search comboboxes break |
| Escape | First press closes the popup and keeps the text; second press clears the text | Two-stage Escape lets a user dismiss suggestions without losing what they typed |
| Tab | Commits the active option, then moves focus onward | Defensible either way. Committing matches most picker expectations; closing without committing matches most search expectations. State which you chose |
| Alt plus Down arrow | Opens the popup without changing the active option | Gives keyboard users a way to see suggestions again after Escape |

!!! warning "When not to debounce"

    Debouncing a local, synchronous filter is pure harm: it adds its full interval to every keystroke and prevents nothing, since there is no request to prevent. The measurement that justifies a debounce is the cost of the work being deferred. If filtering costs under about 8 ms, run it on every keystroke and render immediately. If it costs more, the answer is usually to cap the result set or move the work off the main thread, not to make the user wait.

### Breaking point, then V2 (autocomplete)

**What fails: three things at once, and they are all consequences of reuse.**

| What fails | The observation | Why V1 cannot fix it |
|---|---|---|
| Races under a slow network | 14 percent of queries contain an overlap, by the arithmetic above, and some fraction of those paint the wrong list before correcting | V1's sequence check handles ordering but still lets the client and the server do work for results nobody will see |
| Composition input | Users typing Japanese or Chinese report that Enter selects a suggestion instead of accepting their candidate text | Key handling that ignores composition state cannot distinguish "Enter confirms my candidate" from "Enter commits your suggestion" |
| Every consumer reimplements the contract | Three teams ship three option markups, two of which have no option role and one of which uses a div with a click handler | V1 owns the markup, so consumers fork it the moment the design differs |

V2 turns the component inside out. The logic is a state machine plus a request controller, and the markup belongs to the consumer, but the accessibility attributes are supplied by the component so they cannot be forgotten:

```
+-------------------+   +----------------------+
| Consumer render   |-->| getOptionProps(i)    |
| rows, groups, css |   | id, role, selected   |
+-------------------+   +----------------------+
          ^                        ^
          |                        |
+-------------------+   +----------------------+
| getInputProps()   |-->| State machine        |
| role, expanded    |   | closed, open, active |
+-------------------+   +----------------------+
                                   ^
                        +----------------------+
                        | Request controller   |
                        | debounce, abort, LRU |
                        +----------------------+
```

**Caption.** V2 exposes property-getter functions rather than markup: the consumer writes its own elements and spreads the returned properties onto them, so roles, identifiers, expanded state and active-descendant wiring come from the component while appearance and content come from the consumer.

!!! warning "When not to go headless"

    A headless component is over-engineering for a single product with one visual design. It roughly doubles the API surface, moves failure modes into consumer code, and makes every bug report start with "show me how you spread the props". The measurement that justifies it is the number of genuinely distinct option layouts requested across your products in six months. Below two, ship a styled component with one `renderOption` slot and keep the markup.

### Deep dive one: the request lifecycle, from keystroke to committed pixels

Four mechanisms are commonly named together as if they were interchangeable. They solve four different problems, and you need all four for different reasons.

| Mechanism | The problem it solves | The problem it does not solve |
|---|---|---|
| Debounce | Request volume, and wasted server work | Ordering. A debounced request can still return after a later one |
| Cache | Latency for a query already seen, including every backspace | The first time a prefix is typed, which is most of them |
| Cancellation | Client work and server load for results nobody will read | Ordering, again. Cancelling request A does not stop an uncancelled request C from landing late |
| Sequence check | Correctness: a stale result never replaces a fresher one | Nothing else. It is one comparison and it is not optional |

The senior point is the second column of the cancellation row. Aborting a request prevents your handler from running for that request. It does not impose an order on requests you did not abort, and it does not stop a response already in the network stack. The only thing that guarantees the rendered list matches the newest query is a comparison against a monotonic counter at the moment you are about to render.

```
keydown   keydown          keydown         resp A     resp B
 "lap"     "lapt"           "lapto"        dropped    rendered
   |         |                |               |          |
   +---------+----------------+---------------+----------+
   0        200    450        520   770      830       1020 ms
                    |               |
                 request A       request B
                 seq = 1         seq = 2, A aborted
```

**Caption.** One query's timeline: the debounce timer is rearmed by each keystroke and fires 250 ms after the last one, request A is issued at 450 ms and aborted at 770 ms when request B is issued, and the reducer renders only the response whose sequence number equals the highest issued.

**The cache, designed rather than added.**

| Decision | Choice | Reason |
|---|---|---|
| Key | The normalised query: trimmed, case-folded and locale-normalised, plus any filter parameters | Two queries that produce the same request must share an entry, or the cache does nothing for the most common case, which is retyping |
| Eviction | Least recently used, about 50 entries | A 12-character query passes through at most 12 prefixes, so 50 covers a session's worth of backtracking at a few kilobytes |
| Freshness | A time to live chosen by corpus volatility: minutes for a product catalogue, zero for anything permission-scoped or user-mutable | A stale suggestion for a deleted record is a correctness bug, not a performance win |
| Scope | Per authenticated identity, cleared on any sign-in or sign-out | A module-level cache shared across sessions leaks one user's permission-filtered results to the next. This is a security defect, not a cache-tuning question |
| Render policy | On a cache hit, render synchronously with no debounce, then revalidate if the entry is older than the time to live | This is what closes the 326 ms gap computed in the estimation section: the first painted list comes from memory, and the network corrects it |

**Negative caching, and the condition under which it is sound.** If the query "zxq" returned zero results, then for a strict prefix search every extension of "zxq" also returns zero, so every one of those requests can be answered locally. This removes an entire class of requests: users who mistype a long token generate a run of guaranteed-empty queries.

The condition matters. The optimisation depends on the result set being monotonically non-increasing as characters are appended. That holds for prefix and substring matching. It fails for fuzzy or typo-tolerant search, where adding a character can make a previously non-matching record match. Ask which one the server implements before shipping this. If you cannot find out, do not ship it.

**Policies that are decisions, not defaults.**

| Policy | The rule | The derivation or reason |
|---|---|---|
| Minimum query length | 2 characters, unless the corpus is under about 1,000 items | 5,000 items over 26 first letters leaves 192 candidates, which cannot fit 10 slots meaningfully; two characters leaves about 7 |
| Leading-edge fire | Only on a cache hit | Firing a network request on the first keystroke spends a request on the least informative query the user will type |
| Retry on failure | Never retry a suggestion request | It is a superseded operation by construction: the user's next keystroke replaces it. Retrying builds a queue behind a user who has moved on, which is how a slow backend becomes an outage |
| Client deadline | 1 second, then abandon quietly | At 200 ms per keystroke, a 1 second old answer is 5 characters out of date. Rendering it would be worse than rendering nothing |
| Result cap | 10 rendered, more available only by refining the query | Derived from the 8 ms render budget and from keyboard usability, which are two independent reasons landing on the same number |

!!! warning "When not to cache suggestions at all"

    Skip the cache when results are permission-scoped and change frequently, when the corpus is small enough to ship to the client, or when the median service latency is under about 50 ms, at which point a cache hit saves less than four frames and you have added an invalidation problem for it. The measurement that justifies a cache is the repeat rate: instrument the share of suggestion requests whose normalised query was already requested in the same session. Below about 20 percent, the cache is not paying for itself.

### Deep dive two: the accessibility contract as a public API surface

For a component used by other teams, accessibility is not an implementation detail. It is an API guarantee, and the moment a consumer can render an option, the consumer can break it. The design problem is therefore not "which attributes are correct" but "how do I make the correct thing the only convenient thing".

**The shape the pattern requires.** The input carries the combobox role, an expanded state that tracks the popup, a reference to the popup element, and a declaration of what kind of autocompletion is happening. The popup is a listbox whose children are options. Focus stays in the input, and the active option is referenced by identifier rather than by moving focus.

**Why focus stays in the input, and where the other model belongs.**

| Model | Where focus lives | Correct for | Wrong for |
|---|---|---|---|
| Active descendant | Focus never leaves the input; a property names the active option's identifier | Comboboxes, because the user is still typing and taking focus away from the text field breaks editing | Widgets where the user is navigating structure rather than typing |
| Roving tabindex | Real focus moves between items; exactly one item is in the tab sequence | Grids, menus, toolbars and tab lists, where movement is the interaction | Comboboxes, because moving focus out of a text input while it is being edited breaks composition, selection and mobile keyboards |

This is one of the two deep dives on this page that touch focus management, and they are deliberately different. Case study 3 uses roving tabindex for a grid, for the reason in the table above. Case study 4 handles focus movement across a boundary rather than within a widget. If they collided, one of them would be padding.

**Announcement design, which is where the derived number bites.** The estimation section showed that announcing on every keystroke queues speech four times faster than it drains. The design that follows:

| Announcement | Channel | Timing |
|---|---|---|
| Result count | A polite status region containing text such as "8 results available" | Once per settled query, meaning after the response renders and only if the count changed. Never on a keystroke |
| The active option | The active-descendant reference, read automatically by the screen reader | On every active-option change, and it must not also be pushed into the status region, which would speak it twice |
| Zero results | The status region, with a message the user can act on | Once, and it is the only feedback a non-visual user gets that the query failed |
| Loading | Usually nothing | If the wait exceeds about a second, a single busy announcement. Announcing every load turns the control into a metronome |
| The committed value | The input's own value change, which the screen reader reads because the user caused it | On commit only |

**Composition input, in three rules.** Ignore these and the control is unusable for a large share of the world's typists.

- Do not trigger a search while a composition session is active. The intermediate text is not a query.
- Do not act on Enter during composition. It confirms the input method's candidate, and stealing it prevents the user from typing at all.
- Trigger a search on composition end, not on the key event that ended it, because the committed text arrives with the composition event.

**Touch and virtual keyboards.** Three constraints that pure keyboard testing never surfaces:

- The virtual keyboard covers the bottom of the viewport, so a popup anchored below the input can be entirely hidden. Position against the visual viewport, and flip above the input when there is not enough room below.
- Each option must be a pointer target of at least 24 by 24 CSS pixels to meet the level AA target size criterion, which a compact 28 px row with padding-collapsed content can quietly fail.
- Touch screen reader users do not generate the arrow key events your keyboard model is built on. They explore by touch and activate by double tap, which works only if each option is a real element with an option role and an accessible name, not a styled row with a click handler on a wrapper.

**The API that makes the contract hard to break.**

| API element | What it returns | What it prevents |
|---|---|---|
| `getInputProps()` | Role, expanded state, popup reference, autocomplete mode, and the active descendant identifier | A consumer wiring the input themselves and omitting the expanded state, which silently makes the popup invisible to assistive technology |
| `getListboxProps()` | Role, identifier, and a label association | A popup rendered as a plain container, which produces a list of unlabelled text |
| `getOptionProps(index)` | Identifier, option role, selected state, and the pointer and focus handlers | Options built from wrappers with click handlers, which are unreachable by keyboard and invisible to touch exploration |
| A development-mode assertion | Warns when a rendered option is missing the identifier the getter supplied | Turns a silent accessibility failure into a console error during development, which is the only feedback loop that actually works |
| A conformance test exported with the package | A test the consumer runs against their own composition | Moves the guarantee from "we implemented it" to "your usage is verified", which is the only honest claim a headless library can make |

!!! warning "When not to expose an escape hatch"

    Consumers will ask to override the roles, to render options inside their own wrapper elements, or to add a second focusable control inside an option. Refuse the first. Allow the second only if the getter's properties land on the interactive element. Refuse the third: a focusable control inside an option has no defined behaviour in this pattern and produces a widget nobody can operate.

    The measurement is the number of consumer bug reports that are actually contract violations. If it climbs, your API is too permissive, not your documentation too thin.

### Failure modes and evaluation (autocomplete)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Stale render | A list for a prefix the user typed past, briefly or permanently | The sequence check at render time, not merely cancellation at request time |
| Debounce defeated by re-render | Request volume silently returns to one per keystroke after an unrelated refactor | Own the timer outside render state, and alert on requests per committed query. This bug is invisible in every other metric |
| Dangling active descendant | The active option identifier points at an element that no longer exists, so the screen reader falls silent and arrow keys appear dead | On every result set change, either map the active option to its new index or clear it. Never leave it pointing at a removed identifier |
| Announcement flooding | Speech queues minutes behind the user | One announcement per settled query, with the count as the only payload |
| Cross-user cache leakage | A user sees suggestions filtered by the previous user's permissions | Scope the cache to the authenticated identity and clear it on any authentication change. Treat this as a security bug with a security bug's urgency |
| Retry queue | A slow suggestion backend gets slower because every client is retrying superseded queries | Never retry. Abandon at the client deadline |
| Composition break | Enter commits a suggestion instead of the candidate text | Gate all key handling on composition state |
| Trapped input | The user cannot submit text that matches no suggestion | Enter submits typed text whenever no option is active, and this needs an explicit test because it only breaks for queries with zero results |
| Popup hidden by the keyboard | On touch, the suggestion list is behind the virtual keyboard | Position against the visual viewport and flip when space is insufficient |

Service level indicators:

- Requests per committed query, which should sit near 3 given a 12-character median and a 250 ms debounce. This is the single most informative number for this component.
- Stale render rate: the share of painted result sets replaced within 100 ms.
- Cache hit rate, and the repeat-query rate that justifies the cache existing.
- Time from last keystroke to first painted suggestion at the 75th percentile, split by cache hit and cache miss, because a blended number hides which half regressed.
- Abandonment rate: queries typed but never committed, which is the closest available proxy for "the suggestions were not useful".
- Automated accessibility assertions on the open popup with an active option, plus a keyboard-only path test and a composition test that types with an input method and presses Enter.

Alert on: requests per committed query rising above about 5, which means the debounce broke; stale render rate rising, which usually means the backend tail got worse rather than that the client changed; and any accessibility assertion failing in continuous integration, which for a shared component should block release rather than file a ticket.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Newest query always wins | Yes, by the sequence check. Cancellation alone would not have been sufficient |
| First suggestion within 300 ms at the 75th percentile | Yes on a cache hit, which is why the cache is load-bearing. On a cold miss the median is 326 ms and the tail is 516 ms, and that gap should be stated rather than hidden |
| Keyboard operable | Yes, with two documented judgement calls: what Tab does and what Home and End do |
| Screen reader receives count, active option and value | Yes, through three separate channels, deliberately not through one live region |
| Consumers cannot break the contract | Mostly. Property getters plus a development assertion plus an exported conformance test make it hard, not impossible. Claiming impossible would be dishonest |
| Free text always submittable | Yes, and it is covered by a specific test because it only fails in the zero-result case |
| Under 12 KB compressed | Plausible for the state machine, the getters and the request controller. It stops being plausible if you also bundle a positioning library, which is the usual way this budget dies |

### Where this answer is a simplification (autocomplete)

Every link below was checked on 1 August 2026.

- **ARIA Authoring Practices Guide, Combobox pattern, `https://www.w3.org/WAI/ARIA/apg/patterns/combobox/`.** The primary source for the combobox role, the expanded and controls relationships, the autocomplete modes, and the active-descendant focus model used here.
- **WAI-ARIA 1.2, `https://www.w3.org/TR/wai-aria-1.2/`.** The normative definitions of the states and properties named above, including active descendant, set size and position in set.
- **MDN, AbortController, `https://developer.mozilla.org/en-US/docs/Web/API/AbortController`.** The cancellation primitive, and worth reading precisely because it makes clear that it aborts an operation rather than ordering several.
- **WCAG 2.2, Understanding Target Size (Minimum), `https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html`.** Success criterion 2.5.8 at level AA, requiring 24 by 24 CSS pixels, which is the constraint on compact option rows.
- **web.dev, Interaction to Next Paint, `https://web.dev/articles/inp`.** The 200 ms threshold at the 75th percentile used as the interaction budget.
- **Ranking is not covered here.** Which suggestions come back, and in what order, is a retrieval problem; the typeahead case study in [modern system design](modern-system-design.md) covers the index side and the [ML system design](ml-system-design.md) course covers the ranking side.
- **Real comboboxes are multi-select more often than the pattern suggests.** Chips introduce a second focus region, a removal keyboard model, and an announced selected count, and they interact badly with the active-descendant model because the user now has two places focus could reasonably be. This case study designs the single-value control and says so.
- **Positioning is its own project.** Flipping, shifting, collision detection against the visual viewport and behaviour inside scrolling and clipping ancestors are what actually make a popup library large. Treating positioning as solved is the largest simplification on this page.

### Level delta (autocomplete)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Starts building a custom combobox | Asks whether the source is remote before choosing an approach | Asks whether it is a search box or a picker first, because that decides Enter, blur and free-text semantics, which are the requirements everything else hangs off |
| Debounce | "Add a debounce, maybe 300 ms" | Explains that debouncing trades latency for request volume | Derives the floor from the 200 ms inter-keystroke interval, shows that a shorter debounce is strictly worse, and then shows that the debounce alone misses the target so the cache must carry the first paint |
| Ordering | Does not come up | Adds cancellation and considers the problem solved | Separates cancellation from ordering, computes that 14 percent of queries contain an overlap, and adds a sequence check because cancellation cannot provide the guarantee |
| Caching | "Cache the results" | Keys by query with an eviction policy | Adds negative prefix caching, then states the monotonicity condition under which it is sound and refuses to ship it against a fuzzy backend |
| Accessibility | Names the ARIA roles | Implements the pattern including active descendant | Treats the contract as an API, derives the announcement policy from speech rate against typing rate, and designs property getters plus a development assertion so consumers cannot omit it |
| Internationalisation | Not raised | Mentions right-to-left | Gates search and Enter on composition state, and explains that without it the control is unusable for a large share of typists |
| Reuse | Ships a styled component | Adds a render slot | Chooses headless only against a stated threshold of distinct layouts, and names what headless costs in support burden |
| Operations | "Add some logging" | Tracks latency and click-through | Names requests per committed query as the metric that catches the debounce regression no other metric can see, and states its expected value from the same arithmetic used to design the debounce |

What each level added:

- **Mid to senior:** the mechanisms stop being a list of recommended techniques and start being matched to specific failures.
- **Senior to staff:** the numbers that set each parameter are derived in the room, one widely recommended technique is rejected on a stated condition, and the accessibility work is treated as a contract with an enforcement mechanism rather than a checklist.

---
## Case study 3: Data table or grid

The feed had one axis and homogeneous items. A grid has two axes, heterogeneous columns, a selection model that outlives the visible rows, and an export button that quietly promises something the client cannot deliver. The four assumptions from case study 1 still apply, and one more is added below.

### The prompt (data grid)

"Design a data grid: a million rows, a hundred columns, sortable and filterable, with row selection and export."

### Questions that fork the design (data grid)

| Question worth asking | If the answer is A the design becomes | If the answer is B the design becomes |
|---|---|---|
| Does the data already fit on the client, or is it paged from a server? | Client: sorting and filtering are local and instant, and the entire design is a rendering problem | Server: every sort and filter is a request, the visible row range maps to a range query, and you need coordination between scrolling and fetching |
| Are rows a fixed height? | Fixed: both axes are index arithmetic, you can jump to row 900,000 exactly, and the scrollbar is truthful | Variable (wrapped text, expandable detail rows): you inherit the measurement and anchoring problem from the feed, which is why this case study assumes fixed and says so |
| Is the column set fixed, or can users reorder, hide, resize and pin? | Fixed: the column layout is a constant and can be compiled into a template | Configurable: column layout becomes persisted per-user state, and resizing must not relayout every mounted cell on every pointer move |
| Must selection survive filtering and paging? | No: selection is a set of identifiers currently in view, and it clears on filter change | Yes: "select all" means "everything matching the current filter", which cannot be an identifier set, which is deep dive two |
| Is cell editing in scope? | Read-only: the design is rendering plus selection | Editable: you add a per-cell state machine, validation, dirty tracking, conflict handling and undo, which is a larger problem than everything else here combined |
| How large can an export get? | Bounded, under roughly 50 MB: generate it on the client from a dedicated fetch | Unbounded: the client cannot hold it, so export becomes an asynchronous server job with a notification, and the button stops meaning "download now" |
| Which assistive technologies are in scope? | None specified, which is how grids end up as unlabelled nested containers | Screen readers: you owe row and column indices against totals the DOM does not contain, and a keyboard cell cursor that drives the window |
| Is there a pinned or frozen column region? | No: one scroll container, and the layout is simple | Yes: multiple regions sharing scroll state, and a synchronisation decision that has a visible failure mode, which is deep dive one |

### Requirements (data grid)

Functional:

- Display rows from a server-side result set that the client never holds in full.
- Sort by any column and filter by any column, with the result reflected in a shareable location.
- Select individual rows, ranges of rows, and every row matching the current filter.
- Run a bulk action against the selection and report exactly what happened.
- Export the current selection, or the full filtered result set, in a form a spreadsheet opens correctly.
- Be navigable by keyboard and by screen reader, including movement to rows that are not in the DOM.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Row count | 1,000,000 | GIVEN by the prompt |
| Column count | 100 | GIVEN by the prompt |
| Grid viewport | 1,400 by 800 CSS pixels | ASSUMED; a maximised window on a laptop, which is what this tool is used on |
| Row height | 32 px, fixed | ASSUMED, and the fixed part is a real design decision that buys exact addressing |
| Default column width | 140 px | ASSUMED |
| Bytes per cell in the wire format | 50 | ASSUMED; short identifiers, dates and amounts with JSON overhead |
| Elements per rendered cell | 3 | ASSUMED: a cell container, a content wrapper and a text node holder, which is a modest cell |
| Mounted element ceiling before layout costs more than a frame | 5,000 | ASSUMED placeholder tied to the 4 ms reflow assumption from case study 1. Measure it on your target machine before defending it |
| Interaction to next paint, 75th percentile | under 200 ms | GIVEN |
| Time from a sort click to the first correct row painted | under 1 s at the 75th percentile | ASSUMED; beyond a second users click the header again, which is its own failure |
| Conformance | WCAG 2.2 level AA, ARIA 1.2 grid pattern | GIVEN |

### Estimation that eliminates (data grid)

| Calculation | Result | What it rules out |
|---|---|---|
| 1,000,000 rows times 100 columns | 100,000,000 cells | Rules out rendering the grid by five orders of magnitude. Nobody argues with this, which is why it is not the interesting number |
| 800 px viewport divided by 32 px rows, and 1,400 px divided by 140 px columns | 25 rows and 10 columns visible, so 250 visible cells | Rules out nothing yet. It is the denominator for everything below |
| Row-only windowing: (25 plus 10 overscan) times 100 columns times 3 elements | 10,500 mounted elements | Rules out windowing rows alone at 100 columns: it is more than double the 5,000 element ceiling, entirely because of columns nobody is looking at |
| Two-axis windowing: 35 rows times 14 columns times 3 elements | 1,470 mounted elements | Rules out the argument that a second axis is premature. It is a 7x reduction, and it is the reason this case study does not simply reuse case study 1's answer |
| 100 columns times 50 bytes, times 100 rows per page, divided by 200 KB per second | 5 KB per row, 500 KB per page, 2.5 s | Rules out fetching full-width rows at scroll speed. The user outruns the fetch by an order of magnitude |
| 15 projected columns times 50 bytes, times 100 rows, divided by 200 KB per second | 750 bytes per row, 75 KB per page, 0.375 s | Rules out treating the horizontal window as purely a rendering concern. It must also shape the request, or the network cost of a hundred columns is paid on every page |
| 1,000,000 rows at n log n comparisons, which is about 20,000,000, at an assumed 20 ns per comparison | 0.4 s of blocked main thread | Rules out sorting a million client-side rows synchronously, at twice the interaction budget. It does not rule out client sorting: it rules out doing it on the main thread |
| 1,000,000 rows times 100 columns times 50 bytes | 5 GB of export payload | Rules out building the export in client memory, and rules out any design where the download button is synchronous |
| 1,000,000 selected identifiers at 8 bytes | 8 MB as a compact array, and larger as a JSON request body | Rules out materialising "select all" as an identifier set, and rules out sending the selection to the server as a list |
| A user deselecting one row per second for five minutes, at 8 bytes each | 300 exceptions, about 2.4 KB | Rules out the objection that a predicate-based selection cannot express manual deselection. Human-scale exceptions are three orders of magnitude smaller than the set |
| 2,000 px per second horizontal flick divided by 60 frames per second | 33 px of misalignment for every frame of synchronisation lag | Rules out driving a second scroll container from the first container's scroll event. A 33 px offset between a header and its column is not subtle |

The 20 ns per comparison is an ASSUMED placeholder for a comparison over two object property reads in a modern engine. The conclusion depends only on it being tens of nanoseconds rather than single nanoseconds.

### V0, the simplest thing that works (data grid)

**Predict first: before any windowing, what is the cheapest change that makes a 100-column server-paginated table usable, and why does it beat virtualization on effort per unit of benefit?**

??? note "Show answer"

    Reducing the number of columns shown by default. A hundred columns is almost always a configuration failure rather than a requirement: users work with eight to fifteen at a time and reach for the rest occasionally. Shipping a default column set plus a picker cuts mounted elements, wire bytes and horizontal scrolling in one change, needs no new rendering machinery, and is reversible.

    It is also the easiest answer to skip, because the prompt says a hundred columns and it is tempting to treat that as a constraint rather than as a symptom.

```
+-----------+   +--------------------+   +----------------+
| URL query |-->| Server sorts,      |-->| HTML <table>   |
| sort,page |   | filters, paginates |   | 50 rows        |
+-----------+   +--------------------+   +----------------+
                          |
                          v
                +--------------------+
                | Export endpoint    |
                | same query, a file |
                +--------------------+
```

**Caption.** V0 puts the sort, filter and page into the address bar, lets the server produce 50 rows of a real HTML table per request, and points the export button at an endpoint that takes the same query parameters, so the client holds no state at all.

Why this is correct for a large class of internal tools:

- The address bar is the state. The view is shareable, bookmarkable, restorable, and the back button works without a single line of history code.
- A real table element gives screen readers row and column association, header association and navigation commands for free. Every one of those is expensive to rebuild.
- The server is already sorting and filtering, because it must: it owns the index. Doing it twice is the duplication, not doing it once.
- Export is a link. The hardest requirement on the page is solved by the simplest possible mechanism.

!!! warning "When not to move off the paginated table"

    Below roughly 2,000 rows and 20 columns, a plain table with client-side sorting beats every grid library on bundle size, accessibility and time to build. And before adding any grid machinery, ask what the user is actually doing: if the dominant task is "find one record", a search box with a result list is a better product than a grid, and it deletes this entire design.

    The measurement that justifies a grid is the share of sessions that scroll past the first page more than a couple of times, and the share that compare rows far apart in the ordering.

### Breaking point, then V1 (data grid)

**What fails: the workflow, not the frame rate.** Users comparing records lose their comparison at every page boundary, and each sort costs a full document round trip. The technical symptom is mild; the product symptom is that people export to a spreadsheet and stop using your tool.

How you would observe it:

| Signal | How to collect it | What it means |
|---|---|---|
| Page-flip rate per session | Count of page parameter changes per session | High flip rates with low action rates means people are hunting, not working |
| Export rate per session | Exports divided by sessions | An export rate near one means the grid is a download button with extra steps |
| Time from sort click to first painted row | Real user timing on the navigation | A full document round trip on every sort is usually a second or more, over the stated budget |

V1 keeps one scroll container and windows the rows only:

```
+---------------+   +------------------+   +-----------------+
| Scroll        |-->| Row window       |-->| Mounted rows    |
| container     |   | first, last      |   | 35 x 100 cells  |
+---------------+   +------------------+   +-----------------+
                            |                      ^
                            v                      |
                    +------------------+   +-----------------+
                    | Range fetcher    |-->| Row cache       |
                    | keyset+snapshot  |   | index -> row    |
                    +------------------+   +-----------------+
```

**Caption.** V1 replaces pagination with a virtual scroll region whose height is the exact product of row count and row height, mounts the 25 visible rows plus overscan, and fetches the ranges it needs through a keyset-paginated endpoint that carries a snapshot token so the ordering does not shift underneath the user.

Two design rules that make the fetching survive a fast scroll:

| Rule | Why |
|---|---|
| Do not fetch on every scroll frame. Fetch when scroll velocity falls below a threshold, and render skeleton rows in the meantime | A flick across 200,000 rows would otherwise issue dozens of range requests for ranges the user passes in one frame each. This is the grid's version of the retry storm |
| Cancel superseded range requests, and drop late responses whose range no longer intersects the window | The same sequencing rule as the autocomplete, applied to ranges rather than to queries. It is the same failure class and it deserves the same defence |

!!! warning "When not to replace pagination with virtual scrolling"

    Pagination is a real user interface with real advantages: it is addressable, it makes "where am I" answerable, and it lets a user say "row 40 on page 3". Virtual scrolling gives a smoother reading experience and takes those away unless you rebuild them. The measurement is the comparison behaviour above. If users mostly read the first page and act, keep pagination and spend the effort on filters instead.

### Breaking point, then V2 (data grid)

**What fails: the hundred columns you are not looking at.** V1 mounts 3,500 cells, which is 10,500 elements against a 5,000 element ceiling, and it pulls 5 KB per row over the wire when the user can see 750 bytes of it. Both costs are paid for columns that are off screen.

How you would observe it: mounted element count sampled during scroll; bytes received per hundred rows scrolled; and frames over budget during horizontal scroll reported separately from vertical, because they have different causes and averaging them hides both.

V2 windows the second axis and lets that window shape the request:

```
+--------------+  +-------------------+  +------------------+
| Row window   |->| Cell window       |->| Mounted cells    |
| 25 + 10 over |  | 10 cols + 4 over  |  | 35 x 14 = 490    |
+--------------+  +-------------------+  +------------------+
        |                  |                      ^
        v                  v                      |
+--------------+  +-------------------+  +------------------+
| Pinned key   |  | Range fetcher     |  | Column layout    |
| sticky col   |  | rows + col subset |  | widths resolved  |
+--------------+  +-------------------+  +------------------+
```

**Caption.** V2 adds a horizontal window over the resolved column layout, pins the identifying column so a horizontally scrolled row is still recognisable, and projects the request to the visible columns plus the key, taking mounted elements from 10,500 to 1,470 and per-row bytes from 5 KB to 750.

The row cache now has to hold partial rows, which is the cost nobody mentions:

| Consequence | Detail |
|---|---|
| Cache entries are per row and per column set | Scrolling right and back must not refetch columns already held, so the cache key is the row index and the value is a sparse map of column to value |
| A sort or filter change invalidates everything | The row index no longer means the same record, so the cache is keyed by (snapshot token, index) and a new token empties it by construction |
| Export cannot read from this cache | It holds a window of a window. Export must fetch its own data, which is the point deep dive two turns into a rule |

!!! warning "When not to window columns"

    Below roughly 30 columns, row-only windowing keeps you under the element ceiling (35 rows times 30 columns times 3 elements is 3,150) and column windowing buys nothing but complexity: a second measurement axis, sparse cache entries and a harder accessibility story. The measurement is mounted element count during scroll and bytes per hundred rows. The cheaper move first is reducing elements per cell from three to one and shipping a smaller default column set.

### Deep dive one: two axes are not one axis twice, and the frozen column problem

It is tempting to describe column windowing as "the same thing, sideways". It is not, and the differences decide the implementation.

| Property | Rows | Columns |
|---|---|---|
| Count | Unbounded, a million here | Small and known, a hundred here |
| Homogeneity | Identical structure, so one estimate serves all | Heterogeneous: widths differ by content type and by user configuration |
| Size knowledge | Known only after render, unless fixed height is enforced | Known before render, from the column definitions and any stored user widths |
| What is hard | Measurement, estimation error and anchoring | Scroll synchronisation across regions, and paint cost during horizontal movement |

That last row is why this deep dive is not a repeat of case study 1's. There the unknown was how tall an item is; here widths are configuration, and the unknown is how to keep three regions aligned at 2,000 pixels per second.

**Three ways to build a frozen column region, and what each costs.**

| Approach | Mechanism | Paint and scroll cost | What happens to semantics |
|---|---|---|---|
| Sticky positioning inside one grid | The pinned cells stay in normal flow and are stuck by the compositor | Best: one scroll container, no synchronisation code, no lag by construction | Preserved. One table, one row, correct header and cell association for free |
| Separate scroll regions kept in step | Two or three DOM subtrees, one driving the others | Worst: the driven region updates a frame late, which the estimation section prices at 33 px of visible misalignment | Destroyed. A logical row is now three DOM fragments, so a screen reader reads three unrelated groups unless you rebuild the relationships, and rebuilt relationships are fragile |
| A single canvas | Draw cells yourself | Excellent, and you also lose text selection, find-in-page and zoom reflow | None at all. You must build and maintain a parallel accessible representation, which is a second implementation of the whole grid |

The decision rule is short: use sticky positioning unless you have measured that it does not work in your browser matrix. Separate synchronised scrollers are the approach most likely to be reached for and the one with both the visible defect and the accessibility defect.

**Why the synchronised-scroller lag is not fixable by trying harder.** A scroll event is dispatched after the compositor has already scrolled. Any position you write in response is applied to the next frame. At 2,000 px per second that is 33 px behind, every frame, for the entire gesture.

No amount of throttling, no listener flag and no frame-callback arrangement recovers a frame you are structurally behind by. The only fixes are to make the browser do the moving, with sticky positioning, or to make exactly one element move, with a transform on a single scroll parent.

**Column resize without relayout per frame.** A live resize that reflows the grid on every pointer move costs a full layout of 490 cells per frame, which the 4 ms reflow assumption puts at half the entire JavaScript frame budget before you have done anything else. The standard fix is a ghost divider:

- On pointer down, capture the pointer and show one absolutely positioned divider element.
- On pointer move, translate the divider. One composited transform, no layout of anything else.
- On pointer up, write the new width once, relayout once, and persist it.

The user sees a divider follow their finger at full frame rate, and the grid snaps to the new width once. Live-reflow resize looks more impressive in a demo with ten rows and falls apart at 490 cells.

**Accessibility of a doubly windowed grid.** The DOM holds 490 of 100,000,000 cells. Everything below follows from that.

| Problem | Mechanism |
|---|---|
| The set sizes are wrong on both axes | Declare the totals on the grid with row count and column count, and stamp every row and cell with its true index. These attributes exist precisely because the DOM is allowed to hold a subset |
| Focus must be able to leave the window | The grid has exactly one tab stop; arrow keys move a cell cursor. Moving the cursor to a row outside the window must scroll the window and fetch the range, which means keyboard navigation and scrolling share one piece of state |
| The focused cell can be unmounted | The mounted window is the union of the visual window and the focused cell's row and column. This is the same rule as the feed's focused post, and it is the same failure class |
| Sort state is invisible | The sortable header is a real button inside the header cell, and the header cell carries the current sort direction as state, so it is announced when the user lands on it rather than needing a live region |
| Jumping is impossible | Control plus Home and Control plus End move to the first and last cell, and Page Up and Page Down move by a viewport. Without these, a keyboard user cannot reach row 900,000 in any reasonable time |

The keyboard cursor sharing state with the scroll window is the part that is usually missing. Implementations that treat scrolling as visual and focus as separate produce a grid where a screen reader user's cursor and the rendered window drift apart, and the user hears empty cells.

!!! warning "When not to pin columns"

    A pinned region is only worth it when a horizontally scrolled row becomes unidentifiable without it. If the leftmost column is not the identifier, or if users rarely scroll horizontally because the default column set fits, pinning adds a stacking context, a shadow that can force repaints, and an edge case in every measurement path. The measurement is the share of sessions with any horizontal scroll, and the share of those that scroll past the identifying column.

### Deep dive two: selection as a predicate, and export as its only honest consumer

"Select all" is where grid designs quietly become incorrect. The estimation section priced the naive answer at 8 MB of identifiers, which is not merely large but unrepresentative of what the user meant: they did not select a million specific rows, they selected a condition.

**Three representations, and they are not interchangeable.**

| Representation | Memory | What "select all" means | What happens when the filter changes | What the server receives |
|---|---|---|---|---|
| Explicit identifier set | Linear in selected rows, 8 MB at a million | "The rows I have individually accumulated" | Selection is stale: it contains rows the user can no longer see | A list, which is unsendable at this size |
| Page-scoped set | Trivial | "The rows on this page" | Cleared, honestly | A short list |
| Predicate plus exceptions | Linear in manual deviations, about 2.4 KB for five minutes of clicking | "Everything matching this filter, minus these" | Either cleared or re-evaluated, and you must choose deliberately | A filter specification, a mode, an exception list and a consistency token |

The third row is the only representation that can express what the user did. The state is small:

```
+---------------------+       +----------------------+
| Selection state     |       | Bulk action request  |
| filter spec         | ====> | filter spec          |
| sort spec           |       | mode: all or none    |
| mode: all or none   |       | exception id list    |
| exception id set    |       | snapshot token       |
+---------------------+       +----------------------+
```

**Caption.** Selection is stored as the filter that produced the visible set, a mode saying whether the base is everything or nothing, and a set of individual exceptions; a bulk action sends that description plus the snapshot token, so the server re-evaluates the predicate rather than receiving a list of identifiers.

**The consequences, which is where this becomes a senior answer.**

| Consequence | Why it follows | What you do |
|---|---|---|
| The predicate is evaluated later than it was built | Rows can be inserted, deleted or edited between selection and action | Send the snapshot token, and have the server report actual counts: "12,431 matched at selection time, 12,428 updated, 3 no longer match". Report it in the interface, not only in a log |
| Showing a count requires a count query | The client does not hold the rows and cannot count them | Make the count a separate, cacheable request. When it is expensive, display "all matching rows" and offer an exact count on demand, which is honest and cheap |
| Range selection crosses unfetched rows | Shift-clicking row 5 then row 90,000 selects rows the client has never seen | Express the range as indices under the current sort, not as identifiers. This only works if the sort is total, which forces the next row |
| The sort must be stable and total | If two rows tie and the tiebreak is arbitrary, index 900 means a different record on the next request | Every sort specification carries the primary key as its final tiebreaker. An unstable sort makes range selection silently wrong, which is a rendering decision reaching into correctness |
| The header checkbox has three states | It must show "none", "all" or "some" | Use a checkbox whose checked state can express a mixed value. Note that the platform's indeterminate flag is a property rather than an attribute, so a server-rendered grid must set it after hydration or it renders as unchecked |

**Keyboard and announcement for selection.**

- Space toggles the focused row. Enter activates the row's primary action. Conflating them means a keyboard user cannot select without navigating away.
- Shift plus arrow extends from an anchor index, and the anchor is set by the last unshifted selection. Extension operates on indices, so it works across unmounted rows.
- Row selection state lives on the row as a selected state. Do not push selection changes into a live region: the user caused the change and the screen reader reports the row they are on.
- The selected count is a status, announced once when it settles, not on every toggle. This is the same drain-rate reasoning as the autocomplete's result count.

**Export, and the three ways to get it wrong.**

| Approach | When it is right | The failure it produces when it is wrong |
|---|---|---|
| Serialise the rows the client has | Never for a windowed grid | Exports whatever happened to be cached, silently, so the user gets 3,000 of 12,431 rows and no warning. This is the most common data grid defect in production |
| Fetch the full selection to the client and build a file | Below roughly 50 MB, which by the 50 bytes per cell assumption is about 10,000 rows at 100 columns | Above that, memory pressure and a tab that stops responding while it concatenates |
| Server job producing a file, plus a link | Above that threshold, and always for "everything matching the filter" | Costs a queue, object storage, an expiry policy and a notification channel. The button now means "we will tell you when it is ready", and the interface must say so |

Two correctness rules that apply to all three:

- **Export from the data layer, never from the view layer.** Exporting rendered values ships locale-formatted numbers with thousands separators, which a spreadsheet imports as text, and ambiguous dates, which it imports as the wrong date. The exported file must come from the same values the API returned, formatted for machines.
- **Neutralise formula injection.** Spreadsheet software evaluates a cell whose text begins with an equals sign, a plus, a minus, an at sign or certain whitespace characters. A user-supplied field can therefore become a formula in someone else's spreadsheet. Prefix or quote affected cells at export time. Treat it as a security control with an owner, not as a formatting preference.

!!! warning "When not to use predicate selection"

    If the largest realistic filtered result is a few thousand rows, an explicit identifier set is exactly representable, trivially serialisable, and free of the consistency token, the count query and the "3 rows no longer match" reporting path. The measurement is the 99th percentile row count matching a user's filter. Below a few thousand, ship the set. The predicate model earns its cost only when the set stops fitting in a request body.

### Failure modes and evaluation (data grid)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Request storm during a fast scroll | Dozens of range requests for ranges already passed | Fetch on scroll settle, cancel superseded ranges, and drop late responses that no longer intersect the window |
| Torn pinned region | The pinned column drifts from its rows during horizontal scroll | Sticky positioning rather than synchronised scroll containers. The defect is structural, so the fix must be too |
| Phantom and duplicate rows | Rows appear twice or vanish as the user scrolls a mutating dataset | Keyset ranges under a snapshot token, and a cache keyed by (token, index) so a new token invalidates everything at once |
| Unstable sort | Rows jump between requests, and range selection selects the wrong records | Force a primary-key tiebreaker into every sort specification |
| Unbounded row cache | Memory grows with scroll distance and never shrinks | Evict by least recently used range, with a ceiling stated in rows, and report the ceiling in the interface only if it ever becomes user-visible |
| Selection drift | A bulk action affects a different set than the user saw | Snapshot token, server-side re-evaluation, and a result report that names the mismatch count |
| Silent partial export | The exported file contains only cached rows | Export never reads the render cache. Enforce it in code review and in a test that exports while the window is scrolled to the middle |
| Formula injection | An exported field executes in a user's spreadsheet | Escape at the export boundary, on both the client and the server path |
| Focus loss on window change | A keyboard user's cell cursor is unmounted by a scroll | The mounted window is the union of the visual window and the focused cell |
| Layout thrash on resize | Frames collapse while dragging a column edge | Ghost divider during the drag, one layout on commit |

Service level indicators:

- Frames over 16.7 ms per second of scrolling, reported separately for vertical and horizontal scroll.
- Mounted element count at the 95th percentile during scroll.
- Range requests per thousand rows scrolled, which is the direct measure of whether the settle logic works.
- Skeleton-visible time at the 75th percentile: how long a user looks at placeholder rows.
- Time from a sort or filter interaction to the first correct row painted, at the 75th percentile.
- Bulk action mismatch count, meaning rows that no longer matched at execution time, as a rate.
- Export job success rate and duration at the 95th percentile, plus the share of exports that exceed the client-side threshold.
- Automated accessibility assertions with the window scrolled on both axes, plus a keyboard-only test that reaches the last row and selects a range.

Alert on: range requests per thousand rows scrolled rising, which means the settle threshold regressed; bulk action mismatch rate rising, which means the snapshot semantics are wrong or tokens are living too long; export failure rate; and any accessibility assertion failure.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Never hold the full result set | Yes. The client holds a window plus a bounded cache, and export deliberately does not read from it |
| Sort and filter reflected in a shareable location | Yes if the sort, filter and anchor row stay in the address bar. This is the requirement most often lost during a rewrite to a virtual scroller |
| Select individually, by range and by predicate | Yes, with range selection depending on the sort being total, which is stated rather than assumed |
| Bulk action reports what happened | Yes, through server-side re-evaluation against a snapshot token and an explicit mismatch count |
| Export opens correctly in a spreadsheet | Yes, given data-layer values and formula neutralisation. Both need tests, because both fail silently |
| Screen reader navigable including off-window rows | Yes through declared totals, per-cell indices and a cell cursor that drives the window. It is worse than a plain table for browsing, and that trade should be stated |
| Interaction to next paint under 200 ms | Yes at 1,470 mounted elements, provided sorting and filtering happen on the server or off the main thread. A synchronous client sort would blow it by a factor of two |

### Where this answer is a simplification (data grid)

Every link below was checked on 1 August 2026.

- **ARIA Authoring Practices Guide, Grid pattern, `https://www.w3.org/WAI/ARIA/apg/patterns/grid/`.** The primary source for the single-tab-stop cell cursor, the arrow, Home, End and Page navigation model, and the guidance that focus placement depends on what a cell contains.
- **WAI-ARIA 1.2, `https://www.w3.org/TR/wai-aria-1.2/`.** Normative definitions of row count, row index, column count and column index, which exist specifically so that a partially rendered grid can tell the truth about its size.
- **OWASP, CSV Injection, `https://owasp.org/www-community/attacks/CSV_Injection`.** The characters that trigger formula evaluation, and why naive escaping is insufficient in some spreadsheet software.
- **MDN, `content-visibility`, `https://developer.mozilla.org/en-US/docs/Web/CSS/content-visibility`.** The cheaper first step before any windowing, applied per row.
- **WCAG 2.2, `https://www.w3.org/TR/WCAG22/`.** The conformance target for the header controls, the selection checkboxes and the resize affordances.
- **The server side is not designed here.** Keyset pagination, snapshot consistency and the cost of deep offsets are contract problems; see the pagination and idempotency material in [product architecture](product-architecture.md), and the metrics-pipeline case study in [modern system design](modern-system-design.md) for how expensive an arbitrary filter over a hundred columns actually is to serve.
- **Fixed row height is doing a lot of work.** Variable heights bring back estimation, anchoring and an inexact scrollbar, exactly as in [case study 1](#case-study-1-news-feed-application), and they make jump-to-row impossible without a server-side offset index. Real grids with expandable detail rows live in that world.
- **Editing is a larger problem than everything above.** A per-cell state machine, validation, optimistic writes with per-cell rollback, conflict handling and undo across a windowed surface is a separate design, and an interviewer who adds "and cells are editable" has roughly doubled the question.

### Level delta (data grid)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Names a grid library and starts drawing virtualization | Starts from a server-paginated table and names what breaks | Asks what the user is actually doing, observes that a hundred columns is usually a configuration failure, and proposes a default column set before any rendering machinery |
| Windowing | "Virtualize the rows" | Computes 3,500 mounted cells and adds column windowing | Shows that the two axes have different unknowns, so they need different mechanisms, and lets the horizontal window shape the request as well as the render |
| Frozen columns | Synchronises two scroll containers | Prefers sticky positioning | Prices the synchronised version at 33 px of visible lag per frame at flick speed, explains why no amount of throttling recovers it, and notes that it also destroys the row's semantics |
| Sorting | "The server sorts it" | Notes that a client sort of a million rows blocks the main thread | Computes 0.4 s, concludes the constraint is the main thread rather than the client, and adds a primary-key tiebreaker because range selection depends on a total order |
| Selection | A set of identifiers | Notes that select-all cannot be a set and proposes a flag | Models selection as a predicate with exceptions, prices both representations, and then designs the consistency token and the mismatch report that the predicate makes necessary |
| Export | Builds a file from the rows in memory | Fetches the full set and builds a file | Sets the client threshold from the byte arithmetic, moves past it to a server job, and adds the two silent-failure rules: export from the data layer, and neutralise formula injection |
| Accessibility | Adds a grid role | Adds row and column indices | Makes the keyboard cell cursor drive the window and the fetch, so a screen reader user can reach row 900,000, then states honestly that browsing is worse than a plain table |
| Operations | Watches error rates | Adds latency metrics | Names range requests per thousand rows scrolled and bulk-action mismatch rate as the two metrics that catch this design's specific regressions |

What each level added:

- **Mid to senior:** the design starts from the smallest thing that satisfies the requirement, and each addition is attached to a measured failure rather than to the word "scale".
- **Senior to staff:** the second axis is derived rather than assumed, one popular implementation is rejected with a number attached, and selection stops being a rendering concern and becomes a data contract with consistency semantics.

---
## Case study 4: Image carousel and modal dialog

The previous three case studies were about how much you render and when you fetch. This one is about control transfer: focus crosses a boundary and has to come back, and a pointer gesture competes with the browser for the same movement. Both are state problems, and both are usually implemented with booleans that cannot represent the states that actually occur. The four assumptions from case study 1 still apply.

### The prompt (carousel and dialog)

"Design an image carousel whose items open in a full-screen viewer, working with mouse, touch and keyboard."

### Questions that fork the design (carousel and dialog)

| Question worth asking | If the answer is A the design becomes | If the answer is B the design becomes |
|---|---|---|
| Is the carousel decorative promotion or a gallery the user must traverse? | Decorative: the strongest accessibility answer is making it skippable and not blocking anything behind it | Functional: every item must be reachable, announced and individually addressable, and skipping is not an option |
| Does it rotate automatically? | Yes: you owe a visible pause control, pausing on hover and on focus, and you have accepted a permanent conformance obligation | No: an entire requirement, an entire control and an entire class of complaints disappear. Ask this before anything else |
| How many items, and are they known at open time? | A fixed small set: render them all, no windowing, no fetching design | Unbounded or paged: you inherit windowing and prefetch, and the viewer needs a loading state that is not a spinner over a black screen |
| Is the viewer a route or ephemeral interface? | A route: it is deep-linkable, the back button closes it, it can be server-rendered, and history is your state machine | Ephemeral: no address, and the back button will do something the user did not intend unless you intercept it |
| Are gestures in scope beyond tap? | No: buttons only, and the whole gesture design disappears | Swipe and drag to dismiss: a pointer state machine with direction locking and interruptible animation, which is deep dive two |
| Can the viewer contain further overlays? | No: one focus boundary, and closing is unambiguous | Yes: a stack, ordered inertness, and Escape must close only the topmost, which changes the close handler from a function into a policy |
| May the page behind scroll while the viewer is open? | Yes, it is a non-modal panel: no trapping, and the surrounding page stays interactive | No: background scroll must be prevented, which is the single most platform-dependent requirement on this page |
| Is there one image size or a responsive set? | One: simple, and probably too large or too small on most devices | A set: you choose per viewport and per device pixel ratio, and you must decode the neighbour before the user asks for it |

### Requirements (carousel and dialog)

Functional:

- Traverse items by pointer, by keyboard and by touch gesture, with identical results.
- Open any item in a full-screen viewer and return to exactly where the user was, with focus on the item they opened.
- Change images inside the viewer without a page load, and announce the change to a screen reader.
- Respect a user's reduced-motion preference for every transition.
- Never leave a partial gesture stuck when the browser or the operating system takes the pointer away.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Viewer viewport | 1,400 by 900 CSS pixels at device pixel ratio 2 | ASSUMED; a laptop in a browser, which is the largest common case and therefore the one that sizes memory |
| Full-size source file | 400 KB | ASSUMED; a compressed photograph at that resolution |
| Decode cost for a 5 megapixel image | about 60 ms | ASSUMED placeholder. Re-measure on your lowest supported device; the conclusion needs only that it exceeds a frame by several times |
| Frame budget during a gesture | 16.7 ms, of which about 8 ms is ours | DERIVED, as in case study 1 |
| Open and close transition | 200 ms, and 0 under reduced motion | ASSUMED; long enough to read as movement, short enough that it never becomes waiting |
| Direction lock threshold | 10 px | ASSUMED, justified below against pointer jitter and against flick speed |
| Commit thresholds | 30 percent of the slide width, or 500 px per second | ASSUMED; derived against each other below so that a fast short flick still commits |
| Pointer target size | at least 24 by 24 CSS pixels | GIVEN by WCAG 2.2 success criterion 2.5.8 at level AA |
| Auto-rotation obligation | any automatic movement lasting over 5 seconds needs a pause mechanism | GIVEN by WCAG 2.2 success criterion 2.2.2 at level A |
| Conformance | WCAG 2.2 level AA | GIVEN |

### Estimation that eliminates (carousel and dialog)

| Calculation | Result | What it rules out |
|---|---|---|
| 1,400 by 900 CSS px at device pixel ratio 2, so 2,800 by 1,800 device pixels, times 4 bytes | 20.2 MB of decoded pixels per full-screen image | Rules out treating image memory as a rounding error. One viewer image costs more than most entire single-page applications |
| Five images held decoded, meaning the current one and two either side | 101 MB | Rules out a prefetch window of plus or minus two. On a phone with a few hundred megabytes of tab budget this is the whole budget for pictures the user has not asked for |
| 400 KB divided by 200 KB per second | 2 s to fetch a neighbour on demand | Rules out fetching only when the user swipes. Two seconds of blank after a gesture reads as a broken control |
| The two rows above, together | Prefetch exactly one image either side | Rules out both extremes at once. This is the rare case where two independent constraints pin a parameter to a single value, and it is worth saying so out loud |
| 60 ms decode against a 16.7 ms frame | 3.6 frames dropped if decoding happens on the main thread during a transition | Rules out attaching a full-size source and hoping. The decode must be requested and awaited before the element is shown |
| 10 px lock threshold at a 2,000 px per second flick | Reached in 5 ms, under one frame | Rules out the objection that a lock threshold delays fast gestures. It only delays slow, ambiguous ones, which is exactly what it is for |
| One frame of movement at 2,000 px per second | 33 px, so a single frame's delta is a 2,000 px per second velocity estimate and a dropped frame reads as 4,000 | Rules out computing release velocity from the last pointer event. It must be averaged over about 100 ms, which is roughly six samples |
| 30 percent of a 400 px slide | 120 px of travel required to commit by distance alone | Rules out a distance-only rule: a decisive 60 px flick would snap back, which users experience as the control ignoring them. Hence the parallel velocity rule |
| One full document load per image at an assumed 800 ms, over a 10 image browsing session | 8 s of waiting | Rules out keeping the viewer as a server-rendered route once browsing rather than deep-linking becomes the main use |
| Scrollbar width of about 15 px removed when the background is locked with hidden overflow | A 15 px horizontal shift of the entire page behind the viewer | Rules out the simplest scroll lock without also reserving the scrollbar space. The shift is visible through a translucent backdrop and it is a layout shift on both open and close |

### V0, the simplest thing that works (carousel and dialog)

**Predict first: which parts of a swipeable carousel does the browser already implement, and what exactly do you lose by writing your own?**

??? note "Show answer"

    A scroll container with horizontal snap points already gives you touch dragging with momentum, rubber banding at the ends, snapping to items, mouse-wheel and trackpad support, keyboard scrolling when the container is focusable, right-to-left mirroring, and correct behaviour under browser zoom.

    Writing your own means reimplementing every one of those on top of pointer events, and the ones people forget are momentum and right-to-left. What you gain is programmatic control over the transition, a reliable current-index value, and freedom in how items animate. That is a real gain, but it should be a trade you name rather than a default.

```
+---------------------+   +----------------------+
| Scroll container    |-->| Item, item, item ... |
| snap type: x        |   | each snap aligned    |
+---------------------+   +----------------------+
           |
           v
+---------------------+
| Each item is a link |
| to a full-size page |
+---------------------+
```

**Caption.** V0 is a horizontally scrolling list with CSS scroll snapping and no JavaScript at all; each item is an ordinary link to a server-rendered page showing the full-size image, so the browser provides the gesture, the momentum and the navigation.

Why this is correct for a large class of products:

- It is a list of links. Every assistive technology, every input device and every keyboard already knows what to do with it.
- Each image has a real address, which makes it shareable, indexable and reachable without the carousel existing at all.
- Zero bytes of JavaScript, so it works during the window before hydration, which is exactly when a user first sees a page.
- It degrades to a scrollable row of images if anything at all goes wrong, which is a working product rather than a broken widget.

!!! warning "When not to move off the scroll-snap list"

    Stay here unless you need at least one of: a reliable current-index value for indicators, a transition you control, or a viewer that preserves the surrounding page state. The measurement is behavioural: instrument how many images a session views. If the median session views one image, the full-page viewer costs nothing and the client-side one buys nothing. The 8 s figure above only bites for users who browse.

### Breaking point, then V1 (carousel and dialog)

**What fails: browsing.** A user who opens six images pays six document loads, which the arithmetic above puts at roughly 8 s of waiting across a session, and each return re-enters the gallery page at whatever scroll position the browser chooses. The second failure is that indicators are impossible: the scroll offset tells you where the container is, a frame late and between items, not which item the user considers current.

How you would observe it:

| Signal | How to collect it | What it means |
|---|---|---|
| Images viewed per session | Count viewer opens per session | A median above two means the per-image navigation cost is being paid repeatedly |
| Time between consecutive views | Timestamp deltas within a session | If it is dominated by document load time rather than by looking, the navigation is the product |
| Return-position accuracy | Compare the gallery scroll position on entry against its position when the user left | Users describe this as "it loses my place", which is the same complaint as the feed and it has the same cause |

V1 makes the viewer a client-side modal dialog and lets the platform own the hard parts:

```
+-----------+   +------------------+   +-----------------+
| Trigger   |-->| Index state      |-->| Modal dialog    |
| item link |   | current, count   |   | top layer, esc  |
+-----------+   +------------------+   +-----------------+
      ^                  |                     |
      |                  v                     v
+-----------+   +------------------+   +-----------------+
| Focus     |   | Prefetch +-1     |   | Status region   |
| return    |   | decode off path  |   | index and text  |
+-----------+   +------------------+   +-----------------+
```

**Caption.** V1 opens a native modal dialog, which supplies the top layer, the backdrop, inertness of the rest of the document and the Escape key without any of it being written by hand; the index state drives a prefetch of exactly one neighbour either side, each decoded before it is shown, and a polite status region carries the position and alternative text when the image changes.

What the platform gives you, and what you still owe:

| Provided by a native modal dialog | Still yours |
|---|---|
| Top layer stacking, so no z-index arithmetic | Background scroll prevention, which is not part of the same guarantee |
| Inertness of everything outside the dialog | Focus placement on open and the return path on close, both of which are policy |
| Escape closing, without a key handler | Waiting for a close transition before the element is removed |
| A backdrop you can style | Deciding whether clicking the backdrop closes, and declaring it rather than reimplementing it |

!!! warning "When not to build a dialog by hand"

    Almost never build the focus trap yourself. The platform's modal dialog provides inertness, top layer and Escape, and a hand-built trap has to enumerate focusable elements, handle shadow trees, handle iframes and handle content that changes while open. The measurement that would justify hand-building is a browser support matrix that includes engines without the native element or the inert mechanism. If it does not, a hand-rolled trap is code you will maintain forever in exchange for nothing.

### Breaking point, then V2 (carousel and dialog)

**What fails: the moment a finger touches the screen.** Touch users expect to swipe between images and to drag the viewer away to dismiss it. Adding pointer handling introduces states that the boolean flags in V1 cannot express, and each unrepresentable state is a bug report.

| Unrepresentable state | What the user does | What the boolean version does |
|---|---|---|
| Dragging while a transition is running | Grabs the image mid-slide | Either ignores the grab until the animation ends, or applies both, so the image moves at double speed |
| The pointer is taken away mid-drag | The system starts a back-swipe or the browser starts scrolling | Leaves the slide stuck halfway, because only a pointer-up path exists |
| Closing while dragging | Presses Escape mid-drag | Closes while a transform is being written each frame, so the next open starts offset |
| A second pointer arrives | Puts a second finger down | Both pointers write the same transform, and the image jumps between them |

V2 replaces the flags with one explicit machine:

```
+---------+  pointerdown  +----------+  move > 10 px  +----------+
| idle    | ------------> | pressed  | -------------> | locked-x |
+---------+               +----------+                +----------+
     ^                         |                           |
     |                         | pointerup, no move        | pointerup
     |                         v                           v
     |                    +----------+                +----------+
     +------------------- | activate |                | settling |
     |      done          +----------+                +----------+
     |                                                     |
     +-----------------------------------------------------+
          animation ends, or pointercancel from anywhere
```

**Caption.** V2 models the gesture as five states with one active pointer: a press that has not moved far enough to mean anything, a press locked to the horizontal axis, a tap that activates, and a settling state that runs the commit or the snap-back animation, with a cancel transition available from every state back to settling.

The vertical branch is deliberately absent from the diagram and present in the code: if the first significant movement is vertical, the machine returns to idle and lets the browser scroll, which is handled declaratively rather than by state, as deep dive two explains.

!!! warning "When not to add gestures at all"

    On a desktop-first internal tool, gestures add pointer capture, cancellation handling, velocity estimation, interruptible animation and a test matrix, to serve a small share of sessions. The measurement is the share of sessions with any touch input. Below roughly 10 percent, ship buttons plus the scroll-snap container from V0, which already gives touch users a familiar swipe with momentum that you did not write.

### Deep dive one: the dialog focus contract, from the trigger to the return path

Opening a dialog is easy. The interesting half is the return, because by the time the dialog closes the page may no longer be the page that opened it.

**Opening, in order, and the order matters.**

| Step | Action | The error most implementations make |
|---|---|---|
| 1 | Capture a reference to the element that triggered the open, plus enough context to find a replacement later, such as the item index within its list | Storing only the element. When it is gone, there is nothing to fall back to |
| 2 | Render the dialog and show it as modal, which places it in the top layer and makes the rest of the document inert | Rendering it inside a container with a transform or a clipping ancestor, which quietly reintroduces stacking problems the top layer was there to solve |
| 3 | Move focus into the dialog | Leaving focus on the trigger, so the first Tab goes somewhere unpredictable, or focusing the close button, which is the one control the user did not ask for |
| 4 | Announce the dialog by giving it an accessible name from its visible title | Putting the title in a live region, which speaks it twice |

**Which element gets focus, decided by what the dialog is for.**

| Dialog content | Focus target | Reason |
|---|---|---|
| Content to be read, such as an image viewer | The dialog container itself, made programmatically focusable and given a name | The screen reader reads the name and then the content in order. Focusing a control would skip past the thing the user opened |
| A form | The first field | The user opened it to type |
| A confirmation with a destructive action | The safe control, never the destructive one | A user who presses Enter reflexively should not delete something |
| A long scrolling document | The container, so the scroll region is reachable by keyboard | A focused control inside a scroll region leaves the region itself unreachable for keyboard scrolling |

**The return path, which is the part that gets shipped broken.** The trigger may have been removed while the dialog was open: the item was deleted, the list re-windowed it away as in case study 1, a refetch replaced the row, or the route changed underneath. Focus must go somewhere, and the browser's default is the document body, which sends the next Tab to the top of the page and leaves a screen reader user with no idea where they are.

The policy is a ladder, evaluated in order:

1. The original trigger, if it is still in the document and still focusable.
2. The element now occupying that position in the same list, found by the index captured at step 1 of opening.
3. The nearest surviving neighbour in that list, preferring the previous item so that repeated deletions walk backwards predictably.
4. The list container itself, made programmatically focusable for this purpose.
5. The main landmark. Never the body, and never nothing.

This is the same failure class as the feed's unmounted focused post and the grid's unmounted focused cell. Any code that can remove the element containing focus owes a policy for where focus goes next. It appears three times on this page because it is the single most common accessibility defect in dynamic interfaces, and the fix is different each time only in which fallback is sensible.

**Background scroll prevention, which the platform does not fully give you.**

| Approach | What it fixes | What it breaks |
|---|---|---|
| Contain overscroll on the dialog's own scroll region | Stops scroll from chaining out to the page when the dialog's content reaches its end | Nothing. This is the cheapest fix and it addresses the most common symptom, so do it first |
| Hidden overflow on the document element | Prevents background scrolling on desktop | Removes the scrollbar, which shifts the page by its width. Reserve the space with a stable scrollbar gutter, or measure and compensate, or you have introduced a layout shift on open and another on close |
| Fixed positioning of the body with a negative offset | Prevents scrolling on engines where hidden overflow does not, notably some mobile browsers | Loses the scroll position unless you store and restore it, and interacts badly with anything else using fixed positioning |

Choose the least invasive one that passes your device matrix, and test the restore path, because the failure is silent: the user closes the viewer and finds themselves at the top of a page they had scrolled.

**Nested overlays.** A share menu or a confirmation inside the viewer turns close into a policy:

- Maintain a stack. Only the topmost entry is non-inert.
- Escape pops one entry. Closing all of them on one press is a common and infuriating bug.
- Each entry carries its own return target, so popping restores focus to whatever opened that entry, not to whatever opened the first one.
- Native modal dialogs stack in the top layer in open order, which is why using them rather than positioned containers removes most of this work.

**Announcing an image change inside the viewer.** When the user presses the right arrow, nothing about focus changes, so a screen reader has nothing to report. This is the one place on this page where a polite status region is the correct mechanism, and it is worth contrasting with the two places where it was wrong:

| Situation | Live region? | Why |
|---|---|---|
| Feed appending a page | No | The user did not act, and it would speak over their reading. The busy state carries it |
| Autocomplete result count on every keystroke | No | Announcements queue faster than speech drains |
| Viewer image changed by an arrow key | Yes | The user acted, focus did not move, and without it the action produces no feedback at all. Content is the position and the alternative text, announced once per settled change |

!!! warning "When not to make the dialog modal"

    If the user needs to interact with the page behind it, a modal dialog is the wrong pattern and declaring one is worse than not having one, because the declaration tells assistive technology that everything else is unavailable while the visual design says otherwise. The measurement is whether any task requires reading or editing the background. If one does, use a non-modal panel with no trap and no inertness, and give it a clearly labelled close control, since Escape is less discoverable when focus may be outside.

### Deep dive two: the gesture state machine and interruptible motion

A drag is a negotiation between your code and the browser over who owns a movement. Most gesture bugs are lost negotiations, not arithmetic errors.

**Declare the negotiation in CSS, not in JavaScript.** The browser decides whether to scroll before your listener runs. Telling it after the fact, by preventing the default action inside a handler, means it has already made a decision and your listener must be non-passive, which delays scrolling for everyone. Declaring on the element which axes the browser may keep, for example allowing vertical panning only, lets the compositor scroll vertically without waiting for JavaScript and hands horizontal movement to you. Two consequences follow:

- Vertical scrolling stays on the compositor and never janks because of your handler.
- Horizontal movement arrives as pointer events you own, with no need to prevent anything.

The value is fixed when the gesture begins, so changing it mid-gesture does nothing. That is a feature: it means the negotiation cannot change under you halfway through.

**Direction locking, sized.** Do not commit to an axis until the movement passes a threshold. At 10 px:

- It rejects the few pixels of jitter present in almost every tap, which would otherwise turn taps into micro-drags.
- At a 2,000 px per second flick it is crossed in 5 ms, less than one frame, so decisive gestures are not delayed.
- Below it, the machine is in the pressed state and has committed to nothing, so a release is a tap and a vertical move returns control to the browser.

**Pointer capture and cancellation.** Capture the pointer when the lock engages, so a drag that leaves the element keeps delivering events instead of stopping when the finger crosses a boundary.

Then treat cancellation as a first-class transition rather than an error path. The browser or the operating system can take the pointer away at any moment, for a system back gesture, a notification, or a scroll it decided to own. A machine without a cancel transition leaves the slide stuck at whatever offset it had reached, which is the single most common gesture bug in shipped carousels.

**The commit decision, and why one threshold is not enough.**

| Rule | Behaviour | Why it alone is wrong |
|---|---|---|
| Distance only, 30 percent of the slide | A 400 px slide needs 120 px of travel | A decisive 60 px flick snaps back. Users read that as the control ignoring them, and they flick harder, which does not help |
| Velocity only, 500 px per second | A fast flick commits regardless of distance | A slow deliberate drag to 90 percent of the slide snaps back, which is worse |
| Either one | Commit if distance exceeds 30 percent or release velocity exceeds 500 px per second in the locked direction | This is the rule. Both intents are expressible, and neither user is punished |

**Measuring release velocity without lying to yourself.** The estimation section showed that one frame at flick speed is 33 px, so a velocity computed from the last event is a single noisy sample, and a dropped frame doubles it.

Average the displacement over roughly the last 100 ms, which is about six samples at 60 Hz, and discard samples older than that so a pause before release reads as a pause rather than as speed. A user who drags, stops, holds still and releases must not commit, and only a windowed estimate gets that right.

**Interruptible animation, compared honestly.**

| Technique | Interruptibility | Runs off the main thread | Cost |
|---|---|---|---|
| A CSS transition on a transform | Poor: to hand off you must read the current interpolated value from computed style, which is a forced read at exactly the wrong moment | Yes, for transform and opacity | Cheapest to write, worst to interrupt |
| The Web Animations interface | Good: the animation object exposes its current time, can be cancelled, and its current value can be committed before handing off | Yes, for the same properties | Slightly more code, and it is the right default for anything a user can grab |
| A per-frame loop writing transforms | Total: you own the value, so there is nothing to hand off | No: the value is written from the main thread every frame | Most code, and it competes for the frame budget you were trying to protect |

For a carousel, the middle row is the answer: users grab slides mid-transition, and an animation you can query and cancel is the only way to continue from where the motion actually is rather than from where it started.

**Reduced motion, done properly.**

- Reduced motion changes the transition, not the machine. The states, the thresholds and the commit rule are identical; only the settling animation becomes instant or a cross-fade.
- Read the preference at animation time, not once at module load. Users change it mid-session, and on some systems it is toggled precisely because something on the page is moving.
- Auto-rotation must not run under reduced motion at all. This is stricter than the conformance minimum and it is the right default.

**If it does auto-rotate at all.** Movement lasting over five seconds that starts automatically requires a mechanism to pause it. Beyond the minimum, rotation stops when keyboard focus enters the carousel and does not restart on its own, stops on hover, and stops when the page is hidden. The pause control is a real button whose accessible name changes to describe what pressing it will do, and it is placed first in the tab order so a keyboard user reaches it before the moving content.

!!! warning "When not to use a state machine for this"

    For a control with exactly two states and no gestures, such as a disclosure, an explicit machine is ceremony. The threshold is concrete: write down every combination of your boolean flags, cross out the ones that cannot legally occur, and count what remains. If the illegal combinations outnumber the legal ones, or if any illegal combination has ever produced a bug report, the machine pays for itself. For a draggable, animated, dismissible viewer that count is not close.

### Failure modes and evaluation (carousel and dialog)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Focus lost to the body | Closing the viewer after the item was deleted leaves a keyboard user at the top of the page | The return ladder from deep dive one, with the list container as the last resort before the main landmark |
| Stuck gesture | The slide sits at 40 percent offset and never settles | A cancel transition from every state, driven by the pointer cancellation event |
| Double motion | The image moves at twice the finger speed | The settling state must be cancelled and its current value committed before a new drag begins |
| Layout shift on open | The page behind jumps sideways by the scrollbar width | Reserve the scrollbar space, or compensate by the measured width, and verify on both open and close |
| Lost background position | Closing the viewer returns the user to the top of the page | If you use fixed positioning to lock scrolling, store and restore the offset, and test it as a path rather than assuming it |
| Memory exhaustion | The tab is discarded after browsing a dozen large images | Prefetch and retain exactly one neighbour either side, and release sources outside that band |
| Decode jank | The transition drops frames on every image change | Request a decode and await it before showing the element, so the cost is paid off the transition path |
| History desync | The back button closes two overlays, or none | One history entry per overlay if you use history at all, and a stack whose depth matches |
| Announcement silence | Arrow keys change the image and a screen reader says nothing | A polite status region with position and alternative text, announced once per settled change |
| Motion against preference | Content animates for a user who asked it not to | Read the preference per animation, and disable auto-rotation entirely |
| Targets too small | Indicator dots are 12 px | Pointer targets of at least 24 by 24 CSS pixels, with hit areas larger than the visible dot |

Service level indicators:

- Dropped frames per gesture, reported per gesture rather than averaged over a session, because a single bad gesture is what the user remembers.
- Gesture completion rate: gestures started versus gestures that reached a terminal state, which is how a stuck-state regression becomes visible before anyone files a report.
- Time to first painted full-size image at the 75th percentile, split by whether the image was prefetched.
- Decoded image memory at the 95th percentile, sampled on viewer close.
- Focus return success rate, instrumented directly: after close, record whether the active element is the trigger, a fallback in the ladder, or the body. The share landing on the body is a defect rate you can watch.
- Escape close success rate, and the rate at which one Escape closes more than one overlay.
- Cumulative layout shift attributed to the open and close transitions.
- Automated accessibility assertions with the viewer open, plus a keyboard-only path test that opens, navigates, closes and confirms the focus position.

Alert on: focus returning to the body at any measurable rate, since the correct value is zero; gesture completion rate falling, which usually follows a browser update changing when it takes over a gesture; and decoded memory at the 95th percentile rising, which means the retention band regressed.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Pointer, keyboard and touch produce the same result | Yes. All three feed the same machine, and commit is a single transition rather than three code paths |
| Return to exactly where the user was, with focus on the item | Yes for the common case, and degraded predictably through the ladder when the trigger is gone. Exactness is not achievable when the item was deleted, and the design says so |
| Change images without a page load, announced | Yes, through the status region, which is the only feedback channel available when focus does not move |
| Respect reduced motion everywhere | Yes, and more strictly than the conformance minimum, since auto-rotation is disabled entirely |
| Never leave a partial gesture stuck | Yes, given that cancellation is a transition from every state. This is the requirement most likely to regress, because it is invisible until a platform changes its gesture behaviour |
| Memory within a phone's budget | Yes at plus or minus one image, which the arithmetic pins as the only viable window. Any product request to preload more should be answered with the 20.2 MB per image figure |

### Where this answer is a simplification (carousel and dialog)

Every link below was checked on 1 August 2026.

- **ARIA Authoring Practices Guide, Modal Dialog pattern, `https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/`.** The primary source for the dialog role with the modal state, naming requirements, focus placement on open, focus return on close, and the warning that declaring modality without enforcing it is itself a failure.
- **ARIA Authoring Practices Guide, Carousel pattern, `https://www.w3.org/WAI/ARIA/apg/patterns/carousel/`.** The source for the rotation control being a real button whose label changes with state, for stopping rotation when focus enters and on hover, and for the tab order placing the rotation control first.
- **WHATWG HTML Standard, interactive elements, `https://html.spec.whatwg.org/multipage/interactive-elements.html`.** The normative behaviour of the dialog element, showing it as modal, the top layer, and the attribute that governs light dismissal.
- **MDN, `touch-action`, `https://developer.mozilla.org/en-US/docs/Web/CSS/touch-action`.** Why declaring the allowed axes beats preventing the default action in a handler, and the fact that the value is fixed at gesture start.
- **MDN, `Element.setPointerCapture()`, `https://developer.mozilla.org/en-US/docs/Web/API/Element/setPointerCapture`.** Pointer capture and the cancellation event that the state machine treats as a first-class transition.
- **MDN, `HTMLImageElement.decode()`, `https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement/decode`.** Decoding before insertion so that the decode cost is not paid inside a transition.
- **MDN, `overscroll-behavior`, `https://developer.mozilla.org/en-US/docs/Web/CSS/overscroll-behavior`.** Preventing scroll chaining out of the dialog, which is the cheapest of the three scroll-lock approaches.
- **WCAG 2.2, `https://www.w3.org/TR/WCAG22/`, and Understanding Target Size (Minimum), `https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html`.** Success criterion 2.2.2 for automatic movement at level A, and 2.5.8 requiring 24 by 24 CSS pixels at level AA.
- **Pinch zoom is not designed here.** A real photo viewer supports pinch to zoom and pan within a zoomed image, which adds a second simultaneous pointer, a transform with a scale component, focal-point maths and bounds constraints. It roughly doubles the state machine, and an interviewer who adds it has asked a genuinely harder question.
- **Video changes the memory and the controls.** Autoplay policies, captions, keyboard media controls and buffering belong to a different design; the video player prompt in this course's case study list and the [mobile system design](mobile-system-design.md) video client cover that ground.
- **Right-to-left is treated as free here, and it is not.** Direction locking, commit direction, indicator order and the meaning of "next" all mirror, and a gesture implementation that hard-codes a positive delta as "forward" is wrong in half the world. The scroll-snap version in V0 gets this right without being asked, which is one more argument for starting there.

### Level delta (carousel and dialog)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Starts building a custom slider with pointer handlers | Starts from scroll snapping and names what it cannot do | Asks whether it auto-rotates and whether the viewer is a route, because those two answers remove or add whole requirements before anything is drawn |
| Focus | "Trap focus in the modal" | Uses the platform modal so the trap, inertness and Escape come for free | Treats the return path as the hard half, gives a five-step fallback ladder, and connects it to the same failure class in the feed and the grid |
| Scroll lock | Sets hidden overflow on the body | Notes the scrollbar shift and compensates | Ranks three approaches by invasiveness, starts with overscroll containment, and names the silent failure in the restore path of the most invasive one |
| Gestures | Booleans and pointer handlers | An explicit state machine | Declares the axis negotiation in CSS so the compositor never waits on JavaScript, and treats pointer cancellation as a transition rather than an error |
| Commit rule | 50 percent of the width | Distance or velocity | Shows why either rule alone punishes a real user, and specifies velocity as a 100 ms windowed estimate because one frame at flick speed is a 33 px sample |
| Media | Loads the image on open | Prefetches neighbours | Computes 20.2 MB decoded per image and 2 s to fetch, which pins the prefetch window to exactly plus or minus one from both directions, then moves decode off the transition path |
| Motion preference | Adds a reduced-motion query around the transition | Also disables auto-rotation | Reads the preference per animation because users change it mid-session, and keeps the machine identical so reduced motion cannot introduce a second code path with its own bugs |
| Operations | Watches error rates | Adds interaction latency | Instruments focus return success and gesture completion rate, two metrics whose correct value is known in advance, which makes any regression unambiguous |

What each level added:

- **Mid to senior:** the platform is used instead of reimplemented, and the interaction is modelled as states rather than as flags.
- **Senior to staff:** two independent constraints are used to pin a parameter rather than to argue for a preference, the negotiation with the browser is made declarative so the compositor is never blocked, and the operational metrics chosen are ones whose correct value is zero or one, so a regression needs no interpretation.

---

## Case study 5: Rich text editor

This is the prompt where candidates reach for a library name in the first thirty seconds. The interesting part is not which library. It is that the browser already ships an editing surface, that surface has no data model, and every feature you will be asked about next (comments, collaboration, export, undo that does the right thing) is downstream of the model you refuse to build in minute two.

### The prompt (rich text editor)

"Design the rich text editor our product uses for notes and comments, the part that runs in the browser."

### Questions that fork the design (rich text editor)

Each row changes a box on the whiteboard. Pick the three that matter for the answer you intend to give, and say why you picked them.

| Question worth asking | If the answer is A the design becomes | If the answer is B it becomes |
|---|---|---|
| Is the stored artefact an HTML string, or a structured document we own? | HTML string: the browser's DOM is your model, every browser quirk is a data bug, and you cannot query the content without parsing it again | Structured tree with a schema: you own the model, the DOM becomes a projection of it, and you owe a serialiser for every output format |
| What is the largest document a user will open? | A comment box under 500 words: you can rebuild the whole view on every keystroke and never notice | Long-form, tens of thousands of words: rendering must be incremental and anything derived from the whole document needs its own strategy |
| Inline marks only, or nested block content? | Bold, italic, link: the model is a flat array of text runs and selection is one integer pair | Tables, lists, code blocks, embeds: a tree with content rules, and selection becomes a path plus an offset |
| Do two people ever edit the same document at the same time? | Single writer: undo is a private stack and you may mutate the model freely | Multiple writers: every change must be a transportable, composable operation, and undo becomes selective. See case study 6 |
| Is the content rendered anywhere outside our own renderer? | Only in our app: sanitisation is a paste-time concern | Email, export, third-party embed: sanitisation is also a publishing boundary and the schema must degrade predictably |
| Are input method editors and mobile autocorrect first-class? | Desktop keyboards only: you can intercept key events and write the DOM yourself | Composition input: you must let the browser compose and reconcile afterwards, which changes the input pipeline, not just a setting |
| Does anything anchor to a position in the text? | Nothing anchors: positions are transient and die with the selection | Comments, mentions, suggestions, presence cursors: you need positions that survive edits made elsewhere, which is deep dive one |
| Who owns undo when the editor sits inside a host application? | The editor: one stack, no coordination | The host: the editor must expose transactions and accept externally driven undo and redo, so the stack cannot live inside the editor |

### Requirements (rich text editor)

Functional:

- Edit text with inline marks (bold, italic, code, link) and block types (paragraph, heading, list, quote, code block).
- Undo and redo, restoring both the content and the selection.
- Paste from other applications without importing their markup or their scripts.
- Persist the document without the user pressing save, and survive a tab crash.
- Expose the document to features that need to read it: word count, mentions, export.
- Remain usable with a keyboard alone and with a screen reader.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Median document size | 400 words | ASSUMED; notes and comments skew short, and the median is what most sessions feel |
| p95 document size | 20,000 words, about 800 blocks | ASSUMED at 25 words per block, which is a normal paragraph |
| Sustained typing rate | 10 keystrokes per second | DERIVED: 120 words per minute at 5 characters plus a space is 600 characters per minute |
| Keystroke to paint | under 50 ms at p99 | ASSUMED; past roughly a tenth of a second the caret feels detached from the finger |
| Main thread work per keystroke | under 8 ms | DERIVED: a 16.7 ms frame at 60 Hz, leaving half the frame for style, layout and paint |
| Undo depth | 200 steps | ASSUMED; deeper stacks are almost never reached and the cost of the choice is memory, computed below |
| Editor route transfer size | under 150 KB compressed | ASSUMED budget; it is a route the user opts into, not the first paint of the site |
| Slow-network throughput | 200 KB per second | ASSUMED; a congested mobile connection at roughly 1.6 Mbps, used as the pessimistic case for every transfer number here |
| p95 paste payload | 250 KB of HTML, 4,000 elements | ASSUMED; a page copied out of a word processor, which is the realistic worst case |
| Autosave frequency | at most one write every 2 seconds per document | ASSUMED; frequent enough that a crash loses a sentence, not a paragraph |
| Anchored comments on the p95 document | 300 | ASSUMED; a heavily reviewed long document, used to size the position work |

### Estimation that eliminates (rich text editor)

Every line ends with the option it removes. A number that removes nothing is deleted from the answer.

| Calculation | Result | What it rules out |
|---|---|---|
| 20,000 words divided by 25 words per block | 800 blocks in the p95 document | Rules out modelling the document as one text node with markup inside it. At 800 blocks you need block identity for anything incremental |
| 800 blocks times 50 microseconds to diff and reconcile one block (ASSUMED: a 25-word block is roughly 30 nodes after marks, and 50 microseconds is a mid-range phone doing property comparisons with no layout) | 40 ms per full rebuild | Rules out rebuilding the whole view per keystroke. It is five times the 8 ms budget, and at 10 keystrokes per second it consumes 40 percent of the main thread on its own |
| One dirty block times 50 microseconds | 0.05 ms | Rules out virtualization as the first optimisation. Reconciling only the changed block is 800 times cheaper, and it does not break find-in-page, print or screen reader reading order the way windowing does |
| 20,000 words times 6 characters | 120,000 characters, about 400 KB serialised at an ASSUMED 3.3x structural inflation for a tree with per-run mark arrays | Rules out sending the whole document on every autosave, computed two rows down |
| 200 undo steps times 400 KB per full snapshot | 80 MB | Rules out snapshot-per-step undo outright on any device you care about |
| 200 undo steps times 150 bytes per inverted step (ASSUMED: a from offset, a to offset and a small replacement slice) | 30 KB | Rules out treating undo memory as a design constraint at all. The hard part of undo is semantics, not size, which is why deep dive two is about grouping |
| 200 steps times roughly 1 KB per structurally shared snapshot (only the changed root-to-block path is copied, about 4 nodes at 100 bytes plus pointer copies) | 200 KB | Rules out the claim that you must choose between snapshots and steps. With a persistent model you can afford both, and you should say so |
| 400 KB document divided by a 2 second autosave interval | 200 KB per second of upload, which is the entire assumed uplink | Rules out full-document autosave on the p95 document. It saturates a slow connection permanently |
| 10 steps per second times 150 bytes | 1.5 KB per second | Rules out any concern about step-based autosave bandwidth. It is 0.75 percent of the assumed uplink |
| 4,000 pasted elements times 5 microseconds to walk one node and test it against an allowlist (ASSUMED, a few string comparisons per node) | 20 ms | Rules out moving sanitisation to a worker for throughput reasons. Twenty milliseconds once is not a streaming problem, and a worker adds a serialisation round trip on the paste path |
| 4,000 pasted elements normalising to roughly 900 blocks, times 50 microseconds | 45 ms of reconcile, so paste is a 65 ms operation before browser layout | Rules out treating a paste as one more keystroke. It needs its own budget, its own single undo step, and above a threshold its own progress affordance |
| 300 anchors recomputed by walking 800 blocks per keystroke | 40 ms per keystroke, growing with document length | Rules out recomputing anchor positions. Mapping 300 integers through one step is roughly 300 comparisons, under 10 microseconds, which is deep dive one |
| 150 KB compressed divided by 200 KB per second | 0.75 seconds of transfer before parse and execute | Rules out shipping the editor in the first-load bundle when only a minority of sessions type. Load it on intent, which is focus or hover of the composer |
| 4 to 8 composition events per committed character in a candidate-based input method (ASSUMED; each keystroke updates the candidate string before commit) | The DOM changes many times per logical character | Rules out a key-event-driven input pipeline. If you rewrite the DOM mid-composition, the candidate window closes and the user cannot type their own language |

!!! tip "Say this out loud in the room"

    "Rebuilding the view is 40 ms and my budget is 8, but rebuilding one block is 0.05 ms, so the first design decision is block identity, not virtualization. I want to be explicit that I checked before optimising."

### V0, the simplest thing that works (rich text editor)

**Predict first: at what point does letting the browser own the document stop working?**

??? note "Show answer"

    At the first feature that has to read the content rather than display it. Word count over a selection, a mention that must resolve to a user id, a comment anchored to a range, an export to anything that is not HTML. Until then the browser's editing surface is genuinely correct and cheaper than anything you would write. The trigger is a query requirement, not a size requirement, and size is the tempting answer because size is visible and a query requirement is not.

```
+-----------+     +------------------+     +----------------+
| Keyboard  | --> | contenteditable  | --> | HTML string    |
| and mouse |     | browser owns     |     | sent on submit |
+-----------+     | the model        |     +----------------+
                  +------------------+             |
                          |                        v
                  +------------------+     +----------------+
                  | Native undo      |     | Server-side    |
                  | stack            |     | sanitiser      |
                  +------------------+     +----------------+
```

**Caption.** V0 has four parts. The browser's editable region holds the content and its own undo history, the application reads the markup only when the user submits, and a server-side allowlist sanitiser is the single security boundary.

Why this is correct for a comment box:

- The document is under 500 words, so a full rebuild would cost 500 divided by 25 blocks times 50 microseconds, which is 1 ms. There is nothing to optimise.
- Native undo already restores the selection correctly, handles composition input correctly, and matches every other text field on the user's machine.
- With no anchored features, no position ever has to survive an edit, so the whole of deep dive one is out of scope.
- One sanitiser on the server is one boundary to audit, and it is the boundary that matters because it is the one that cannot be bypassed by a modified client.

!!! warning "When not to use even this much"

    If the field only ever holds one line of unformatted text, use a plain input and delete the editable region entirely. An editable region invites paste, invites markup, and gives you a sanitisation obligation you did not need. The measurement that justifies upgrading from a plain input is a product requirement for a link or a line break, not a designer's preference for a toolbar.

### Breaking point, then V1 (rich text editor)

**Breaking point one: the stored markup has no canonical form, and it fails the first time you query it.**

What fails, precisely: applying bold to a partial selection produces different markup depending on the browser, the surrounding nodes and the direction the selection was made. Nothing normalises it, so your database accumulates several structurally different representations of the same visible document.

How you observe it: sample 10,000 stored documents and count the distinct element-and-attribute shapes that render as bold. If that count is above three, you do not have a document format, you have a union of browser behaviours. A second observable is the failure rate of any parser you write against that stored HTML.

V1 inverts the ownership. The model is the truth and the DOM is a projection of it.

```
+---------+   +--------------+   +-----------------+
| Input   |-->| Transaction  |-->| Document model  |
| events  |   | list of      |   | typed tree with |
+---------+   | steps        |   | a schema        |
              +--------------+   +-----------------+
                    |                     |
                    v                     v
              +--------------+   +-----------------+
              | Undo history |   | View reconciler |
              | inverted     |   | dirty blocks    |
              | steps        |   | only            |
              +--------------+   +-----------------+
                                          |
                                          v
                                 +-----------------+
                                 | contenteditable |
                                 | projection      |
                                 +-----------------+
```

**Caption.** V1 turns every input event into a list of steps, applies the steps to a typed document tree, records the inverted steps for undo, and reconciles only the blocks the steps touched into the editable region.

The V1 changes, each with the observation that justified it:

| Change | Justified by | Cost you accept |
|---|---|---|
| A typed tree with a schema | Several markup shapes meaning the same thing in stored data | You now own normalisation, and every new block type is a schema change with a migration |
| Every mutation expressed as a step | Nothing can be undone, transported or logged when mutation is direct | Features must be written as steps, which is a real constraint on contributors to your codebase |
| Dirty-block reconciliation | 40 ms full rebuild against an 8 ms budget | The reconciler must know which blocks a step touched, so steps carry positions rather than intentions |
| Input read from composition-aware events, not key events | Composition input breaks under a key-driven pipeline | The browser sometimes changes the DOM before you see the event, so you must reconcile after the fact rather than prevent |
| Autosave sends steps, not the document | 200 KB per second saturating the assumed uplink | The server must be able to apply steps, which is a server change you have to negotiate |

!!! warning "When not to own the document model"

    Below roughly 50 blocks and with no anchored features, no export target and no collaboration, the owned model costs you a schema, a reconciler and a serialiser in exchange for consistency nobody is currently querying. The measurement that justifies it is the count of distinct stored representations above, or the first ticket that says an export lost formatting. The cheaper thing first: normalise on save with a server-side allowlist that rewrites to a canonical shape.

### Breaking point, then V2 (rich text editor)

**Breaking point two: two problems arrive together, and both are invisible in a demo.**

The first is derived state. Once 300 comments anchor into the p95 document, anything that recomputes anchor positions by walking the document costs 40 ms per keystroke, and that cost grows with both document length and comment count. You observe it as keystroke-to-paint latency that rises linearly with the number of comments, which is a shape no capacity increase fixes.

The second is untrusted markup. Paste is a hostile input channel that arrives already formatted as HTML, and it bypasses every guard you put on typed input. You observe it as sanitiser rejection counts by tag, and as the first report of markup surviving a round trip that your schema does not contain.

V2 adds a mapping layer and a sanitisation boundary, and neither of them is a new box in the running system so much as a new rule about how steps behave.

```
+-------------+   +---------------+   +----------------+
| Clipboard   |-->| Paste         |-->| Schema-shaped  |
| HTML, text  |   | normaliser    |   | slice          |
+-------------+   | allowlist     |   +----------------+
                  +---------------+           |
                                              v
+-------------+   +---------------+   +----------------+
| Steps from  |-->| Position map  |-->| Anchors,       |
| any source  |   | per step      |   | cursors,       |
+-------------+   +---------------+   | decorations    |
                                      +----------------+
```

**Caption.** V2 routes clipboard content through an allowlist normaliser that emits only shapes the schema permits, and routes every step through a position map so that anchors, cursors and decorations are moved rather than recomputed.

| Change | Justified by | Cost you accept |
|---|---|---|
| Position mapping per step | Anchor recompute at 40 ms per keystroke against an 8 ms budget | You must choose a bias for every anchor, and the wrong bias is a subtle product bug rather than a crash |
| Paste normaliser driven by the schema | Untrusted HTML entering through a channel typed input never touches | Pasted content loses fidelity, and users notice. You need a plain-text paste shortcut and a visible rule for what survives |
| Paste as exactly one undo step | 65 ms of work that a user thinks of as one action | The step is large, so its inverse is large, which is the one case where inverted steps are not tiny |
| Viewport-scoped decoration | Decorations over 800 blocks when only about 15 are visible | Decorations must be recomputed on scroll, so scroll now has a budget too |

!!! warning "When not to build a position map"

    With no comments, no collaboration and no persistent decorations, mapping is machinery with nothing to map. The measurement that justifies it is the first feature whose position must survive an edit made somewhere else in the document. The cheaper thing first: anchor to block identity rather than to a character offset, which is stable for free as long as nobody splits or merges the block.

### Deep dive one: mapping a position through a step, and why every editor feature is the same feature

I choose this dive because in a rich text editor almost everything a product manager will ask for is secretly the same problem: a number that pointed at a place in the document, and an edit that happened somewhere else.

**The representation.** Flatten the tree into a single integer coordinate space, where every character costs one and every block boundary costs one. A position is one integer. A selection is two. A comment anchor is two. A collaborative cursor is one. A decoration range is two.

**The step.** A step that replaces the range from `a` to `b` with content of length `L` has a delta of `L - (b - a)`. That is the entire mechanism.

**The mapping rule.**

| Where the position sits | New position | Why |
|---|---|---|
| Before `a` | Unchanged | Nothing before the edit moved |
| At or after `b` | Position plus delta | Everything after the edit shifted by the same amount |
| Strictly inside the replaced range | Collapses to `a` or to `a + L`, decided by bias | The text it pointed at no longer exists, so the answer is a product decision, not arithmetic |

**Worked example, in three blocks by purpose.**

Block one, set up the state. A document of 1,000 positions. A comment anchored to the range 35 to 48. A mention anchored at position 12. The user selects 10 through 20 and types three characters.

Block two, compute the step and the delta. The step replaces the range 10 to 20 with content of length 3. Delta is 3 minus 10, which is negative 7.

*Before reading the next block, say what happens to the mention at position 12 and why it is not an arithmetic question.*

Block three, apply the rule to each anchor. The comment starts at 35, which is at or after 20, so it becomes 28. It ends at 48, which becomes 41. The comment survives intact and points at the same words. The mention sits at 12, strictly inside the replaced range, so it collapses to either 10 or 13 depending on bias, and neither answer is more correct than the other in the abstract.

The error most readers make here: treating the inside-the-range case as an edge case to handle later. It is the only interesting case, and it is where every anchored feature acquires its personality.

**Bias, stated as a product rule rather than a flag.**

| Anchor kind | Bias | Consequence you are choosing |
|---|---|---|
| Comment range | Delete the comment when its whole range is replaced | The user who deletes the sentence a comment is on expects the comment to go with it |
| Comment range, partial overlap | Clamp to the surviving edge | The comment stays attached to what is left, which reads as the comment following the text |
| Mention or inline embed | Atomic: an edit that touches any part of it removes the whole thing | Half a mention is not a mention, and leaving fragments produces unresolvable ids |
| Collaborative cursor | Right bias | A cursor should end up after the text a peer just typed, not before it |
| Search decoration | Recompute, do not map | Search results are cheap to derive and mapping them produces stale highlights that look like a bug |

**The features that are all this one feature.**

| Feature | What it is, restated |
|---|---|
| Undo | An inverted step plus a mapped selection |
| Comment anchoring | Two mapped positions plus a bias rule |
| Collaborative cursors | One mapped position per peer, mapped through remote steps |
| Find and replace across an edit | A mapped list of ranges |
| Restoring the caret after a remote change | One mapped position |
| Scroll anchoring on a long document | One mapped position plus a pixel offset |

**Why the projection can still drift, and how you find out.** The DOM is edited by things you do not control: composition input, mobile autocorrect, browser translation, password managers, extensions. The defence is not prevention, it is detection.

- On idle, compute a cheap checksum of the visible projection and compare it with what the model says the projection should be.
- On mismatch, re-project the affected blocks from the model, restore the mapped selection, and log the event with the block type.
- Track the mismatch rate per thousand sessions as a service level indicator. A rising rate after a release is a regression in your input pipeline, and it is otherwise invisible because the user just sees text behaving strangely once.

!!! warning "When not to build this much position machinery"

    If the only features are marks, undo and a word count over the whole document, you do not need a coordinate space. Word count can be recomputed on idle, and undo can carry its own selection. The measurement that justifies the coordinate space is the second feature that needs to survive a remote or distant edit. One such feature can be special-cased honestly.

### Deep dive two: undo is a product decision that happens to have a data structure

This dive differs deliberately from the one above. The first is arithmetic that has a correct answer. This one has no correct answer, only a defensible one, and the interviewer is watching whether you notice the difference.

**Three representations, compared on what actually matters.**

| Representation | Memory for 200 steps | Restores selection | Supports selective undo | Cost per step |
|---|---|---|---|---|
| Full snapshot of the document | 80 MB at the p95 size | Only if stored alongside | No | A full copy, so it grows with the document |
| Inverted step | 30 KB | Yes, if the selection is stored with it | Yes, with mapping | Constant, independent of document size |
| Structurally shared snapshot | 200 KB | Only if stored alongside | No, without extra work | Copies one root-to-block path |

The inverted step wins on the column that matters, which is selective undo, and it is also the cheapest. That is unusual and worth saying out loud: the representation that scales is also the one that enables the feature you will be asked for next.

**Coalescing, which is where the actual bugs live.** A user does not think in steps. They think in actions. The grouping rules:

| Rule | Threshold | Why |
|---|---|---|
| Consecutive typed characters merge into one step | Same block, adjacent positions, gaps under 500 ms | At 10 keystrokes per second an ungrouped stack of 200 steps is 20 seconds of typing, so the whole stack would undo two sentences |
| A paste is always exactly one step | No merging in or out | The user performed one action, and the 900 blocks it produced are an implementation detail |
| A formatting toggle is one step | Never merges with typing | Undo after bolding should remove the bold, not the word before it |
| Deleting by holding a key merges | Same direction, gaps under 500 ms | Same reasoning as typing, and reversing direction always breaks the group |
| A remote step never enters the local stack | Always | Undoing another person's edit is not undo, it is a moderation feature with a different name |

The arithmetic behind the 500 ms threshold: a fast typist emits a keystroke every 100 ms, so 500 ms of silence is five missed keystrokes, which reads as a deliberate pause rather than a stutter. That number is an assumption, and it is one you should tune against real sessions rather than defend on principle.

**Selection restoration is the bug users actually report.** Store, with every step, the selection as it was before the step and the selection as it was after. On undo, apply the inverse and restore the before-selection mapped through any steps that have arrived since. Editors that skip this are the ones where undo works but you lose your place, and users describe that as "undo is broken" without being able to say why.

**Redo invalidation.** A new local step clears the redo stack. A remote step does not, because the user's redo intent has not changed just because a colleague typed elsewhere. That single rule is the cheapest signal in this dive that you have thought about the collaborative case, which is case study 6 and which we deliberately do not solve here.

**How you tell your grouping is wrong, without asking anyone.** Track the ratio of redo presses to undo presses. Redo is mostly a correction: the user undid more than they meant to and put it back. A ratio climbing above roughly 30 percent suggests steps that are too coarse, and a ratio near zero with many consecutive undos suggests steps that are too fine. Both thresholds are assumptions, and their value is that they turn a taste argument into a measurement.

!!! warning "When not to build a custom undo stack"

    If the editor is a single editable region with no owned model, the native undo stack is better than a partial replacement, because a partial replacement fights the browser and loses on composition input. The measurement that justifies taking over is the first step your model applies that the browser did not perform, because from that moment the two histories have diverged and the native stack is undoing the wrong things.

### Failure modes and evaluation (rich text editor)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Split brain, in miniature | The DOM and the model disagree after composition input or an extension mutation, and subsequent steps apply to the wrong positions | Idle checksum, re-project on mismatch, log the block type, and never trust a position derived from the DOM |
| Metastable behaviour | A long document plus many decorations makes each keystroke slower, which makes the user type faster to compensate, which queues more input | Time-slice decoration work, yield to the main thread between blocks, and drop decoration recomputation under input pressure |
| Unbounded growth | Decoration sets and the step log grow for the life of the tab, and a document left open all day slows down | Cap the undo depth at 200, evict decorations outside the viewport, and measure heap growth per hour of an idle open tab |
| Untrusted input | Pasted markup survives the schema and renders elsewhere, or a pasted style rule escapes the editor | Allowlist by schema, strip all event attributes and all URL schemes that are not explicitly permitted, and sanitise again at the publishing boundary |
| Lost work | Autosave fails silently while the user keeps typing, then the tab closes | Persist unacknowledged steps locally, show an explicit unsaved state, and alert on the acknowledgement gap rather than on request errors |
| Input starvation | A single long task during paste or a large replace blocks the caret for hundreds of milliseconds | Budget paste separately, show a progress affordance above a threshold, and measure the longest task on the input path |

Service level indicators to publish:

- Keystroke-to-paint latency at p50 and p99, segmented by document block count.
- Editor time to first input, measured from the intent event that triggered the lazy load.
- Model and projection divergence events per thousand sessions.
- Autosave acknowledgement lag at p95, and the count of sessions ending with unacknowledged steps.
- Sanitiser rejection counts by element and attribute, which is both a security signal and a fidelity complaint predictor.
- Undo to redo ratio, as the grouping health signal.

Alert on sessions ending with unacknowledged steps, and on divergence rate. Both are silent in aggregate performance dashboards and both destroy trust in a single incident. Do not alert on keystroke latency averages, because the p50 of a typing session is dominated by short documents and hides exactly the case you care about.

Re-checking against the non-functional requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Keystroke to paint under 50 ms at p99 | Yes, with dirty-block reconciliation at 0.05 ms and mapped rather than recomputed anchors. No, if decorations are recomputed over the whole document, which is the regression to watch for |
| Main thread work under 8 ms per keystroke | Yes for typing. No for paste, which is explicitly budgeted separately at roughly 65 ms |
| Undo depth 200 | Yes, at 30 KB with inverted steps, so the cap is a semantic choice rather than a memory one |
| Autosave every 2 seconds | Yes with step-based saves at 1.5 KB per second. No with full-document saves, which is why V1 changed it |
| Editor route under 150 KB compressed | Only if the schema, the reconciler and the paste normaliser are the whole editor. Plugins are how this budget dies, so treat plugin size as a governed budget |
| Usable with a keyboard and a screen reader | Not established by anything above. The editable region gives you a starting point, and everything you add on top is a new obligation. See the simplification section |

### Where this answer is a simplification (rich text editor)

- **Tables and nested lists are where schemas break.** A merged table cell containing a list containing a code block is a legal document in most products, and the operations users expect (split a cell, outdent an item at the boundary of two lists) have no obviously correct result. Real editors carry per-structure rules that dwarf the core.
- **Accessibility of a custom editor is a project, not a section.** A screen reader in its own browsing mode navigates your projection, not your model, and formatting changes need announcing without flooding a live region. Start from [WCAG 2.2](https://www.w3.org/TR/WCAG22/) and the authoring patterns in the [ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/), and budget manual testing with at least two screen readers, since automated checks cannot evaluate an editing experience.
- **Composition input deserves more than one row in a table.** The behaviour of composition and the `beforeinput` event is specified in [UI Events](https://www.w3.org/TR/uievents/) and [Input Events](https://www.w3.org/TR/input-events-2/), and the editable region itself is specified in the [WHATWG HTML standard](https://html.spec.whatwg.org/multipage/interaction.html). The specifications are the right primary source here precisely because implementations differ.
- **Clipboard access is not one API.** The formats available, and whether you can read them at all, depend on the gesture and the permission state. See the [W3C Clipboard API and events](https://www.w3.org/TR/clipboard-apis/) specification.
- **Sanitisation belongs in the platform where possible.** Enforcing that no string reaches an injection sink without passing a policy is what [Trusted Types](https://www.w3.org/TR/trusted-types/) exists for, and it converts a code review discipline into an enforced one.
- **Plugins are the real product.** Once the editor is extensible, third-party steps can violate your schema, third-party decorations can blow your budget, and your undo grouping rules become an API contract. That governance problem is larger than the editor.

*These are links to canonical specification locations. Specifications move and are revised, so check the version banner at the top of each before quoting a date from it. Last checked structure of this list: August 2026.*

### Level delta (rich text editor)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Starting point | Names a library and describes its architecture | Starts from the browser's editable region and states the specific requirement that forces a model | Asks which query the product will run against the content, and derives the model from the answer |
| Rendering | Says the view updates when the model changes | Computes 40 ms for a full rebuild against an 8 ms budget and reconciles dirty blocks | Notices that virtualization would break find-in-page and reading order, and rejects it with the reason rather than adopting it as a default |
| Positions | Handles selection as an offset pair | Maps positions through steps and names the bias question | Turns bias into a table of product rules, and points out that six unrelated features are the same feature |
| Undo | Describes a stack of previous states | Uses inverted steps and stores the selection with each | Defines the coalescing rules, names selection restoration as the bug users report, and proposes the undo-to-redo ratio as the health metric |
| Paste | Says it will sanitise | Allowlists against the schema and makes the paste one undo step | Treats paste as a separate latency budget with its own affordance, and moves enforcement to a platform mechanism |
| Failure handling | Mentions autosave | Persists unacknowledged steps and shows an unsaved state | Alerts on sessions ending with unacknowledged steps and on projection divergence, because both are invisible in an averages dashboard |
| Scope discipline | Adds a plugin system because editors have one | Defers plugins and says why | States that the plugin API is the real long-term cost and proposes a size and step budget per plugin before the first one ships |

What each level added, in one line each:

- **Mid to senior:** every structural choice now has a number behind it, and the model exists because a requirement forced it rather than because editors have models.
- **Senior to staff:** the operational and organisational consequences enter the design (divergence detection, the metrics that reveal grouping bugs, plugin governance), and at least one popular technique is rejected on evidence.

---

## Case study 6: Collaborative document

The trap in this prompt is that it sounds like a distributed systems question and is graded like a client question. The convergence algorithm is the part that has published answers. What separates candidates is everything around it: what the client holds in memory, what it does while offline, what happens in the 400 milliseconds after a reconnect, and whether the cursor you draw for a colleague is in the document or beside it.

### The prompt (collaborative document)

"Design the client side of a document that several people can edit at the same time, including when one of them has been offline."

### Questions that fork the design (collaborative document)

| Question worth asking | If the answer is A the design becomes | If the answer is B it becomes |
|---|---|---|
| Do two people edit the same paragraph at the same time, or different sections? | Different sections: per-block last-writer-wins is enough, and the whole convergence discussion disappears | The same paragraph, interleaved by character: you need a character-addressable mechanism, and that decision drives client memory |
| How long may a client be offline before we refuse to merge it? | Minutes: the server can hold a buffer and the client just replays | A day or more: the client holds unbounded local history, and you can never garbage collect anything a straggler might still reference |
| Is there a server we control on the edit path? | Yes: a central sequencer exists, ordering is free, and rebasing is a small amount of code | No, or the server may be unavailable mid-session: convergence must not depend on an ordering authority |
| Plain text, or a rich tree? | Plain text: sequence convergence is textbook and you can lean on published results | Rich tree: concurrent structural edits (splitting a list item that someone else is turning into a table cell) have no obviously correct merge, so you need product rules on top of the algorithm |
| Must history be attributable and reversible? | No: you may compact aggressively and keep only snapshots | Yes, for version history, attribution or legal hold: the operation log is a permanent asset and storage growth becomes a design constraint |
| Do comments or suggestions anchor to text? | No: positions are transient and die with the session | Yes: anchors must survive other people's edits, which reuses the mapping machinery from case study 5 |
| Is presence decorative or authoritative? | Decorative cursors: the channel may be lossy, unordered and dropped freely | Authoritative, meaning it grants or blocks editing: presence needs delivery guarantees, leases and an expiry story, and it stops being cheap |
| How large can a single document get? | Bounded, say under 50,000 characters: you can hold everything and never compact | Unbounded: you need snapshots, a bounded tail, and a policy for what a new joiner downloads |

### Requirements (collaborative document)

Functional:

- Apply local edits immediately, without waiting for any network round trip.
- Converge: any two clients that have received the same set of operations display the same document.
- Show where other editors' carets and selections are, and who they belong to.
- Accept edits while offline and merge them on reconnect without asking the user to resolve anything.
- Anchor comments to ranges so they survive edits made elsewhere in the document.
- Open a long document quickly, without replaying its whole history.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Concurrent editors, median | 3 | ASSUMED; most shared documents have one writer and some readers, and the median is what the common path must be fast for |
| Concurrent editors, p95 | 25 | ASSUMED; a review meeting with a document open on everyone's screen, which is the realistic bad case |
| Typing rate per editor | 10 characters per second | DERIVED as in case study 5, from 120 words per minute at 6 characters per word |
| Local echo | under 16 ms | ASSUMED; a keystroke must land in the same frame it was typed, so it can never wait for a network response |
| Remote edit becomes visible | under 250 ms at p95 | ASSUMED; below roughly a quarter of a second, a colleague's typing reads as live |
| Cursor update becomes visible | under 400 ms at p95 | ASSUMED; presence tolerates more delay than text, and this difference is what lets you spend less on it |
| Supported offline window | 24 hours | ASSUMED; a flight or a bad week of connectivity, and it is the number that decides whether tombstones can ever be collected |
| Document final length | 120,000 characters | GIVEN by the p95 document assumption in case study 5, reused so the two answers stay consistent |
| Lifetime insertions | 600,000 | ASSUMED at 5 times the final length, because a heavily revised document is mostly rewritten |
| Document open time | under 2.5 seconds at p75 | ASSUMED, measured on the 200 KB per second link assumed in case study 5 |
| Divergence | zero tolerated, and detected rather than assumed | GIVEN; a design that cannot detect divergence has not met this requirement, it has only claimed to |

### Estimation that eliminates (collaborative document)

| Calculation | Result | What it rules out |
|---|---|---|
| 25 editors times 10 characters per second | 250 operations per second entering one document | Rules out nothing on its own, which is why the next row exists |
| 250 operations per second fanned out to 25 clients | 6,250 messages per second for one document, 250 inbound per client | Rules out claiming that per-keystroke messaging is obviously fine. The client, not the server, is the constrained party here |
| 250 inbound messages per second times 0.4 ms per message (ASSUMED: parse the frame, map anchors, mark a block dirty, reconcile it) | 100 ms of main thread work per second, so 10 percent of the budget spent watching other people type | Rules out unbatched delivery. Ten percent for zero local benefit is not a rounding error when typing has an 8 ms per-keystroke budget |
| Batching at 50 ms: 20 frames per second per client, averaging 12.5 operations each, at an ASSUMED 0.15 ms per frame plus 0.03 ms per operation | 10.5 ms per second, about 1 percent | Rules out any further transport optimisation. You have a factor of ten of headroom, and 50 ms of the 250 ms visibility budget is a cheap price |
| Raw pointer movement at 60 Hz from 25 peers, fanned to 25 clients | 37,500 messages per second for one document | Rules out sending pointer movement as it happens. This single number is why presence is a separate design and not a field in the edit message |
| Presence coalesced to 4 Hz per sender and merged into one frame per client per 250 ms | 4 frames per second per client, 100 messages per second for the document | Rules out one message per peer per update. The merge is a 375 times reduction and costs 250 ms of staleness against a 400 ms budget |
| 600,000 lifetime insertions times 24 bytes per character identifier (ASSUMED: an 8 byte site id, an 8 byte counter, 8 bytes of ordering and flag fields) | 14.4 MB of identifiers | Rules out nothing yet, which the next row fixes |
| The same identifiers as one runtime object per character, at an ASSUMED 3 times overhead for object headers and pointers | roughly 43 MB for a 120 KB document | Rules out one runtime object per character. This is the number behind the belief that character-level convergence cannot run in a browser |
| Run-length encoding, at an ASSUMED average run of 12 characters (a typing burst before someone else's edit splits it), 50,000 runs at 40 bytes | 2 MB | Rules out that belief. The mechanism is affordable, the naive representation is not, and the difference is an encoding choice |
| Replaying the full log on open: 600,000 operations at 30 bytes, over the 200 KB per second link | 18 MB and 90 seconds | Rules out log-only storage and rules out replay on open, against a 2.5 second budget |
| A snapshot at 400 KB plus an ASSUMED 2,000 operation tail at 30 bytes | 460 KB, so 2.3 seconds | Rules out any design without periodic snapshots, and sets the tail length as the tunable |
| 24 hours offline containing an ASSUMED 20 minutes of actual typing at 10 characters per second, at 30 bytes per operation | 12,000 operations, 360 KB | Rules out worrying about the size of the offline queue. The offline problem is semantic, not spatial |
| Three peers each editing for 20 minutes during that window | 36,000 remote operations to apply on reconnect, at an ASSUMED 30 microseconds each, so 1.1 seconds | Rules out applying catch-up synchronously with no loading state. One second of frozen tab on reconnect is the worst moment to freeze |
| Tombstones retained for the whole offline window: 600,000 entries never shrinking toward 120,000 live characters | a permanent 5 times overhead | Rules out an unbounded offline window. Every hour you promise is an hour you cannot compact, so the 24 hour number is a storage decision disguised as a product one |
| V0 whole-document saves every 30 seconds during a 20 minute shared session | 40 saves each, and every peer save between two of yours invalidates your base version | Rules out optimistic whole-document concurrency the moment two people are genuinely simultaneous. Conflicts are the normal case, not the exception |

!!! tip "Say this out loud in the room"

    "Presence is 375 times more expensive than text if I treat it the same way, and it has a looser latency budget, so I am going to design it as a separate lossy channel and spend my remaining time on the merge."

### V0, the simplest thing that works (collaborative document)

**Predict first: what makes whole-document saving with a version check fail, and is it the algorithm or the schedule?**

??? note "Show answer"

    The schedule. Optimistic concurrency on a whole document is correct: it never silently loses a write, and it detects every conflict. It fails because the conflict rate is a function of how often people save and how long they overlap, and with a 30 second autosave and two people in the document for 20 minutes, nearly every save conflicts. The algorithm is sound and the user experience is unusable, which is a distinction worth naming, because it tells you the fix is finer-grained operations rather than a better locking scheme.

```
+---------+   +----------------+   +-------------------+
| Editor  |-->| Save whole doc |-->| Server compares   |
| client  |   | with a base    |   | base version, and |
|         |   | version number |   | accepts or 409s   |
|         |   +----------------+   +-------------------+
|         |                                  |
|         |   +----------------+             |
|         |<--| Poll every 10s |<------------+
+---------+   | who is editing |
              +----------------+
```

**Caption.** V0 saves the entire document with the version it was based on, the server rejects a save whose base version has moved, and a ten second poll tells the editor client who else has the document open.

Why this is correct when editing is rarely simultaneous:

- With one writer at a time, the base version never moves under you, so the conflict path is dead code that costs nothing to carry.
- There is no operation format, no log, no compaction, no tombstones and no offline semantics, which is four fewer things to get wrong.
- The poll costs 6 requests per minute per client, and at the median of 3 editors that is 18 requests per minute for the document, which no capacity plan notices.
- A conflict, when it happens, is honest: the user is told, and shown both versions. That is worse than merging and much better than losing a paragraph.

!!! warning "When not to move off V0"

    If your measured rate of overlapping edit sessions is low, live collaboration is a large permanent complexity cost paid to avoid a dialog nobody sees. The measurement that justifies moving is the count of 409 responses per hundred saves, plus the count of sessions where two people had the document open within the same minute. Cheaper alternative first: shorten the autosave interval and show the other editor's name prominently, which converts most conflicts into social coordination.

### Breaking point, then V1 (collaborative document)

**Breaking point one: two genuine simultaneous editors, where the conflict rate approaches one per save.**

What fails: the base version moves between your fetch and your save on nearly every cycle, so the user is asked to resolve a merge every 30 seconds. Worse, the resolution is at whole-document granularity, so the user is choosing between two versions of everything in order to reconcile one sentence.

How you observe it: 409 responses per hundred saves, plotted against the number of distinct editors active in the same minute. The two curves will move together, which is the signature of a granularity problem rather than a bug.

V1 replaces the document with a stream of operations ordered by the server.

```
+-----------+   +--------------+   +------------------+
| Local     |-->| Pending      |-->| Server assigns   |
| steps     |   | queue, sent  |   | a sequence and   |
| applied   |   | in 50 ms     |   | broadcasts       |
| instantly |   | batches      |   +------------------+
|           |   +--------------+            |
|           |                               v
|           |   +--------------+   +------------------+
|           |<--| Rebase local |<--| Inbound ordered  |
+-----------+   | pending onto |   | operations       |
                | server order |   +------------------+
                +--------------+
+-----------+   +--------------+
| Presence  |-->| Separate     |
| cursors   |   | lossy 4 Hz   |
+-----------+   | channel      |
                +--------------+
```

**Caption.** V1 applies local steps instantly, queues them for a 50 millisecond batched send, lets the server assign a total order, and rebases any still-unacknowledged local steps onto the operations that arrived first. Presence travels on its own coalesced channel.

| Change | Justified by | Cost you accept |
|---|---|---|
| Operations instead of documents | A conflict rate approaching one per save | You now need an operation format that is stable across client versions, which is a compatibility obligation forever |
| Local echo before acknowledgement | The 16 ms echo requirement, which no round trip can meet | The local document is speculative until acknowledged, so every anchor and every cursor must be able to move underneath the user |
| Server as sequencer | Ordering is the hard part, and one server makes it free | The design now depends on the server being reachable for progress, which is exactly what breaking point two attacks |
| 50 ms outbound batching | 10 percent of the main thread spent on unbatched inbound traffic | 50 ms of the 250 ms visibility budget, spent deliberately |
| Presence on its own channel at 4 Hz | 37,500 messages per second if presence is treated like text | Cursors lag text slightly, which users read as normal and never report |

!!! warning "When not to introduce a server-ordered operation log"

    Below roughly one overlapping session per hundred, and with no offline requirement, the log costs you an operation format, a rebase implementation and a permanent compatibility surface. The measurement that justifies it is the 409 rate above. Cheaper alternative first: section-level locking, where a user claims a heading and its content, which is unfashionable, easy to explain and completely adequate for structured documents like forms and reports.

### Breaking point, then V2 (collaborative document)

**Breaking point two: the 24 hour offline window, which quietly invalidates the sequencer.**

What fails: rebasing works because there is a single authority that decides what came first, and because the set of operations you must rebase against is small. After 24 hours offline, a client returns with 12,000 local operations to rebase against 36,000 remote ones. The rebase is not just slow, it is semantically fragile, because rebasing a structural operation across a day of other people's structural operations produces results nobody can predict or explain.

How you observe it: catch-up duration on reconnect at p95, the count of operations pending at reconnect, and the rate at which reconnect ends in a forced reload. A forced reload is the system admitting it could not merge, and it should be a counted, alerted event rather than a quiet fallback.

V2 makes convergence a property of the data rather than of the ordering service.

```
+-------------+   +-----------------+   +---------------+
| Local edits |-->| Replicated      |-->| Local view    |
| any time    |   | sequence, run   |   | projection    |
+-------------+   | length encoded  |   +---------------+
                  +-----------------+
                     |           |
                     v           v
        +---------------+   +--------------------+
        | Durable local |   | Sync engine, sends |
        | store,        |   | and receives only  |
        | snapshot plus |   | missing operations |
        | bounded tail  |   +--------------------+
        +---------------+             |
                                      v
                            +--------------------+
                            | Peers, via server  |
                            | relay              |
                            +--------------------+
```

**Caption.** V2 stores the document as a replicated sequence whose identifiers make merge order deterministic without a sequencer, persists it locally with a snapshot and a bounded tail so a closed tab loses nothing and a new joiner never replays the whole history, and runs a sync engine that exchanges only the operations each side is missing. Operations received from peers re-enter the replicated sequence, closing the loop.

| Change | Justified by | Cost you accept |
|---|---|---|
| Convergence without a sequencer | A 24 hour offline window and 36,000 operations to rebase | Identifier overhead, which is 2 MB after run-length encoding and 43 MB without it |
| Durable local store | 12,000 offline operations that must survive a tab close | A storage schema you must migrate, and a quota you can exceed |
| Snapshot plus bounded tail | 90 seconds of replay against a 2.5 second open budget | Snapshots must be produced by someone, and a snapshot written from a diverged client poisons everyone who loads it |
| Compaction bounded by the offline window | A permanent 5 times tombstone overhead | You must refuse to merge a client older than the window, and that refusal needs a user-facing story |
| Divergence detection | The requirement says zero divergence, which is only meaningful if you can see it | An extra periodic checksum exchange, and an incident class you did not previously know you had |

!!! warning "When not to move to sequencer-free convergence"

    If every client is online with the server whenever it edits, a server-ordered log is simpler, smaller, easier to debug and easier to reason about during an incident, because there is one authoritative history to read. The measurement that justifies the move is the p95 offline duration of real sessions and the reconnect forced-reload rate. Cheaper alternative first: shorten the promised offline window to minutes and buffer on the server, which keeps the sequencer and deletes most of this section.

### Deep dive one: choosing the merge mechanism from the offline window, not from the literature

I choose this dive because the usual version of this answer names two algorithm families and argues about them in the abstract. The requirement that actually decides it is the offline window, and you can show that in one table.

**The three candidates, compared on what the requirements care about.**

| Mechanism | Needs an ordering authority | Tolerated offline window | Client memory | Behaviour on concurrent structural edits |
|---|---|---|---|---|
| Per-block last-writer-wins | No | Any | Nothing extra | One editor's whole block is discarded, silently |
| Server-sequenced rebase | Yes, for progress | Minutes | Only the pending queue | Predictable for a short window, unexplainable across a long one |
| Character-addressable replicated sequence | No | Any, bounded only by compaction | 2 MB encoded, 43 MB naive | Converges, but the converged result can still be nonsense that needs product rules |

Note the last column. No mechanism makes a concurrent structural merge *good*. Convergence guarantees that everyone sees the same thing, not that the thing is what either author wanted. Saying that sentence is the single highest-value thing in this dive, because it separates a candidate who has read about the algorithms from one who has shipped with them.

**Worked example, in three blocks by purpose.**

Block one, establish the state that makes the problem visible. Both clients hold the text `abc`. Editor A inserts `X` at offset 1. Editor B, at the same moment, inserts `Y` at offset 2. Neither has seen the other.

Block two, apply the operations naively and watch them diverge. A's document is `aXbc`; applying B's operation at its stated offset 2 gives `aXYbc`. B's document is `abYc`; applying A's operation at its stated offset 1 gives `aXbYc`. Two clients, the same two operations, two different documents. Convergence has failed, and no amount of retrying fixes it, because both clients believe they are correct.

*Before reading the next block, say what information the operation is missing. It is not the offset.*

Block three, name the two repairs and what each costs. The rebase repair transforms the incoming offset against the operations already applied locally: A knows it inserted one character at offset 1, so B's offset 2 becomes 3, and A produces `aXbYc`, matching B. The replicated-sequence repair never uses offsets at all: each character carries an identifier, B's `Y` is inserted between the identifiers of `b` and `c`, and both clients place it there regardless of arrival order.

The error most readers make: assuming the tie is broken by time. Two operations at the identical position need a deterministic tie-break that every client computes the same way, and the only inputs both clients definitely share are the operation identifiers themselves. Ordering by wall clock reintroduces clock skew as a correctness bug, and it is a correctness bug that only appears when two people type in the same place at the same instant, which is exactly the case the feature exists for.

**What the encoding decision does to the memory number.**

| Representation | Entries for 600,000 lifetime insertions | Bytes | Verdict |
|---|---|---|---|
| One object per character | 600,000 | roughly 43 MB with runtime overhead | Rejected. It is 350 times the size of the text |
| Packed identifier array per character | 600,000 | 14.4 MB | Workable on a laptop, painful on a low-end phone |
| Run-length encoded, average run 12 | 50,000 | 2 MB | Accepted, and the run length is measurable rather than assumed once you ship |

**Compaction, and why the offline window is a storage decision.** A deleted character cannot be forgotten while any replica might still send an operation referring to it. So the retention rule is: you may collect a tombstone once every replica has acknowledged the operation that created it, or once it is older than the maximum offline window and you are willing to refuse older clients. Promising 24 hours means holding a day of tombstones. Promising unlimited offline means never compacting, and that promise is how documents become permanently slow.

**Divergence detection, because the guarantee is worthless unsupervised.** Every client periodically hashes its document and sends the hash with the highest operation identifier it has applied. The server groups by that identifier and compares hashes. A mismatch is an incident with a specific signature, and without this check the failure presents as a user saying their colleague's screen looks different, months later, with no reproduction.

!!! warning "When not to use character-addressable convergence"

    If your document is a form, a structured record or a set of independent blocks, per-field last-writer-wins with a visible author label is adequate, and it is a hundred times less code. The measurement that justifies the upgrade is the observed rate of two editors inside the same block within the same second. If that rate is near zero, you are paying 2 MB and a compaction policy for a scenario that does not occur.

### Deep dive two: presence is a separate system, and it is allowed to be wrong

This dive is deliberately not more merge arithmetic. The cursor position itself maps through steps using exactly the machinery derived in case study 5, and re-deriving it here would be the template repeating itself. What is genuinely different is that presence is the one part of a collaborative document that is permitted to be lossy, stale and incomplete, and designing it as though it were not is the most common way this feature becomes expensive.

**The three properties that make presence cheap, stated as permissions.**

| Property | Permission it grants | What it saves |
|---|---|---|
| Lossy | Drop any update; the next one supersedes it | No retransmission, no ordering, no acknowledgement |
| Expiring | Every presence record has a time to live | No reliable disconnect notification is required |
| Merged | One frame per client carries every peer's state | Fan-out drops from per-peer to per-client, the 375 times reduction computed above |

**The rate arithmetic, restated as a budget.** With a 400 ms visibility budget, you may spend 250 ms of it on coalescing. At 4 frames per second per client and 25 peers, one document costs 100 messages per second, and each frame carries at most 25 small records, which is roughly 25 times 40 bytes, or 1 KB per frame and 4 KB per second per client.

Compare that with the 37,500 messages per second of the naive design. The entire saving came from relaxing a latency number that nobody in the room would have defended anyway.

**Expiry beats disconnection.** A closed laptop lid, a killed tab and a dead network are indistinguishable from the server's point of view, and every one of them produces a ghost cursor.

| Mechanism | Time to remove a ghost | Cost |
|---|---|---|
| Explicit leave message | Immediate when it arrives, never when it does not | Free, but only covers the polite case |
| Socket close event | Usually seconds, sometimes never on a mobile network | Free, and misleading, because it makes ghosts rare enough to be unfixed |
| Time to live on each record, refreshed by the 4 Hz frame | Bounded, and you choose the bound | One expiry timestamp per record |

Set the time to live at three missed frames, which at 4 Hz is 750 ms. That is short enough that a ghost never outlives a coffee break and long enough that a single dropped frame does not make a colleague's cursor flicker. Both halves of that sentence are the reason for the number, and a candidate who gives the number without both halves has guessed.

**What presence must never do.**

- It must never be written into the persisted document. A cursor in the saved artefact is a privacy leak and a merge hazard, and it will eventually appear in an export.
- It must never gate editing. The moment presence grants or denies a write, it needs delivery guarantees, and every property in the first table is revoked.
- It must never carry content. A selection range is a position pair. Sending the selected text so a peer can preview it turns a lossy channel into a data path with confidentiality obligations.

**The privacy question a staff candidate raises unprompted.** Presence reveals who is reading, not only who is writing. On a document shared with a large group, broadcasting viewers by name is a product decision with a real cost, and the design should let it be configured: viewers anonymous, viewers named, or viewers hidden entirely. Making that a parameter of the presence record rather than a later feature costs nothing now.

!!! warning "When not to build a presence channel at all"

    With a median of three editors and a strong section-based working style, a simple list of who has the document open, refreshed every ten seconds, delivers most of the social value of presence for none of the cost. The measurement that justifies live cursors is the rate at which two editors are within the same block at the same time. If they never are, live cursors are decoration you now have to keep working during every incident.

### Failure modes and evaluation (collaborative document)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Split brain | Two clients apply the same operations and hold different documents, and nobody notices for weeks | Periodic hash exchange keyed by the highest applied operation, with a mismatch treated as an incident, not a retry |
| Thundering herd | A deploy drops every socket, and every client reconnects and requests catch-up in the same second | Full-jitter reconnect backoff, and a catch-up endpoint that can serve a snapshot rather than a log |
| Metastable failure | Catch-up is slow, so clients time out and retry catch-up, which makes catch-up slower | Cap concurrent catch-up work, serve a snapshot under load, and shed with a clear "reloading" state rather than an invisible retry |
| Unbounded growth | Tombstones accumulate for every promised offline hour and never shrink | Compaction bounded by the offline window, plus an explicit refusal path for clients older than it |
| Clock skew | Ordering or tie-breaking by wall clock produces a different result on two machines | Never order by wall clock. Tie-break on identifiers, and use timestamps only for display |
| Poisoned snapshot | A diverged client writes the snapshot that every new joiner then loads | Only allow snapshot production from a client whose hash matched at that operation identifier, and keep the previous snapshot until the new one is corroborated |
| Silent data loss | A merge converges to a document neither author wanted, which no error counter can see | Version history with attribution, so the loss is recoverable, and a user-facing way to see what changed while they were away |

Service level indicators to publish:

- Local echo latency at p99, which must be independent of network conditions by construction. If it correlates with network latency, something in the send path is on the input path.
- Remote apply latency, measured from server sequence assignment to visible paint, at p95.
- Catch-up duration and pending-operation count at reconnect, at p95.
- Forced reload rate on reconnect, which is the count of merges the system gave up on.
- Divergence detections per thousand document-hours.
- Presence staleness, measured as the age of the oldest displayed cursor.

Alert on divergence detections and on forced reload rate. Do not alert on presence staleness, because presence is permitted to be wrong and paging someone about a stale cursor teaches the team to ignore the channel.

Re-checking against the non-functional requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Local echo under 16 ms | Yes by construction, because local steps apply before anything is sent. This is the one requirement the architecture guarantees rather than achieves |
| Remote edit visible under 250 ms at p95 | Yes: 50 ms batching plus network plus roughly 10 ms of client work. The batching interval is the tunable if the budget tightens |
| Cursor visible under 400 ms at p95 | Yes at 4 Hz coalescing, with 250 ms of the budget deliberately spent |
| 24 hour offline window | Yes, at the cost of a day of retained tombstones and a refusal path for older clients |
| Open under 2.5 seconds at p75 | Yes with a snapshot plus a 2,000 operation tail at 460 KB. No without snapshots, which is 90 seconds |
| Zero divergence | Not guaranteed by any mechanism. Detected, bounded and recoverable, which is the honest form of this claim |

### Where this answer is a simplification (collaborative document)

- **Convergence is not correctness.** Every mechanism here guarantees that clients agree. None guarantees that the agreed result preserves what either author meant. Concurrent structural edits (someone splits a list while someone else converts it to a table) need product rules that are specific to your schema and that no paper will give you.
- **Rich text multiplies the problem.** Applying a mark to a range while a peer deletes part of that range, or two peers applying conflicting block types, are cases with several defensible answers. The published sequence results cover the sequence, not the tree over it.
- **Undo in a collaborative document is a different feature.** Undo must revert your change and not your colleague's, which means selective undo over an operation history, mapped through everything that arrived in between. Case study 5 builds the local stack and deliberately stops there.
- **The literature is worth reading in the original.** The operational transformation line begins with Ellis and Gibbs, "Concurrency Control in Groupware Systems" (1989), and the conflict-free replicated data type line with Shapiro, Preguica, Baquero and Zawirski, "Conflict-free Replicated Data Types" (2011). These are named rather than linked because this session could not verify a stable URL for each; search by exact title and author, and prefer the original over a summary.
- **Transport is specified, and worth reading once.** The socket transport is [RFC 6455](https://www.rfc-editor.org/rfc/rfc6455), and the durable local store is [IndexedDB](https://www.w3.org/TR/IndexedDB/). Both have behaviour under storage pressure and background throttling that the specification states and that most designs assume away.
- **Server-side storage is out of scope here and is not small.** Operation log retention, snapshot cadence, per-document sharding and the cost of version history are the other half of this system, and they belong to the distributed systems answer in [modern-system-design.md](modern-system-design.md).

*Specification links point at canonical locations. Specifications are revised, so check the version banner before quoting a date. Last checked structure of this list: August 2026.*

### Level delta (collaborative document)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Names an algorithm family in the first minute | Asks about the offline window first, because it is what decides the algorithm | Asks whether concurrent editing in the same paragraph actually happens, and is prepared to argue for section locking |
| Merge | Describes the mechanism correctly | Works the divergence example and names the deterministic tie-break | States plainly that convergence is not correctness, and defines the product rules for structural conflicts |
| Client cost | Does not size it | Computes 43 MB naive against 2 MB encoded | Notes that the run length is measurable after launch, and treats the assumption as a thing to verify rather than defend |
| Presence | Sends cursors with the edits | Separates the channel and coalesces to 4 Hz with the arithmetic | Derives the time to live from the frame rate, forbids presence in persistence, and raises viewer privacy unprompted |
| Offline | Says operations are queued | Sizes the queue and the catch-up cost, and adds a loading state | Connects the promised offline window to tombstone retention and shows it is a storage decision |
| Failure handling | Mentions reconnect | Adds jittered backoff and a snapshot-serving catch-up path | Adds divergence detection, poisoned snapshot prevention, and refuses to claim zero divergence without a detector |
| Scope discipline | Builds the general case | Justifies each mechanism against a requirement | Offers to delete the whole merge layer if the overlap measurement does not support it |

What each level added, in one line each:

- **Mid to senior:** the algorithm is now chosen by a requirement, and every channel has a budget derived from a latency number.
- **Senior to staff:** the design admits what it cannot guarantee, adds the detection that makes the guarantee meaningful, and is willing to remove the hardest component if the data does not justify it.

---

## Case study 7: Video player

Every browser already ships a video player that is fast, keyboard accessible, screen reader accessible and free. The first thing a strong answer does is say that out loud, then name the specific requirement that forces you to give it up. Candidates who skip that step spend the rest of the round rebuilding features they deleted in minute one without noticing.

### The prompt (video player)

"Design the video player for our web product, the client side, including how it chooses quality."

### Questions that fork the design (video player)

| Question worth asking | If the answer is A the design becomes | If the answer is B it becomes |
|---|---|---|
| On demand or live? | On demand: the whole manifest is known, seeking is unrestricted, and captions are one file | Live: a sliding manifest that must be refreshed, a latency target that fights segment duration, and a rewind window that is a product decision |
| Is the content protected? | Unprotected: plain segments, simple caching, and nothing on the startup path but bytes | Protected: licence acquisition sits on the startup path, device capability varies, and playback can fail for reasons that are not network reasons |
| Do we keep the browser's own controls? | Native controls: keyboard, screen reader and platform behaviour arrive free, and design control is near zero | Custom controls: you inherit an accessibility contract the platform was fulfilling for you, which is deep dive two |
| Which is the harder requirement, fast start or high quality? | Start fast: begin low and ramp, accept a visibly soft first few seconds | Quality floor, for sport or cinema: prebuffer more, start higher, and accept a slower start |
| Is cellular data a first-class concern? | No: one policy everywhere | Yes: a data ceiling, a saver mode, and a persisted user preference that must survive across devices or you will be asked why it did not |
| Is offline download in scope? | No: everything is transient | Yes: persistent storage, persistent licences, a separate quality policy for downloads, and a storage quota you can exceed |
| Are captions optional or required? | Optional: a feature that can degrade | Required by policy or law: caption availability becomes a service level indicator and a missing caption file is an incident |
| One audio track, or several? | One: a single buffer and no switching logic | Several, including audio description: track switching without a gap, a second buffer to keep aligned, and a language preference to persist |

### Requirements (video player)

Functional:

- Play, pause, seek, change volume and change playback rate.
- Choose quality automatically, and let the user override the choice.
- Show captions, let the user turn them on and choose a language.
- Be fully operable from the keyboard, and expose state to assistive technology.
- Resume where the viewer left off.
- Report playback quality events so the delivery pipeline can be measured.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Rendition ladder | 400, 900, 1,800, 3,500 and 6,000 kbps | ASSUMED; a conventional ladder roughly doubling each rung, which is what makes the switch arithmetic below tractable |
| Segment duration | 4 seconds | ASSUMED as the starting point; the live section below shows what forces it down |
| Typical connection | 5,000 kbps, which is 625 KB per second | ASSUMED; a normal fixed or good mobile connection, used as the reference case |
| Poor connection | 1,600 kbps, which is 200 KB per second | ASSUMED, and deliberately the same figure used in case studies 5 and 6 so the numbers stay comparable |
| Startup, click to first frame | under 1.5 seconds at p75 | ASSUMED; past roughly a second and a half viewers begin abandoning, and the number should be replaced with your own abandonment curve |
| Rebuffer ratio | under 0.3 percent of watch time | ASSUMED; it is the industry-shaped target and, more usefully, it converts into a stall budget below |
| Seek to first frame | under 500 ms at p95 | ASSUMED; a seek is a direct manipulation and needs to feel like scrubbing, not like loading |
| Forward buffer target, steady state | 30 seconds | ASSUMED; enough to survive a short tunnel, and the section below shows why it must not be the startup value |
| Source buffer ceiling | 40 MB | ASSUMED; a conservative ceiling for a memory-constrained device, and the ceiling is in bytes because that is what the device cares about |
| Session length used for budgets | 5,400 seconds, a 90 minute film | ASSUMED; the long case, because a short case never exercises the buffer policy |
| Early abandonment | 20 percent of sessions end within 60 seconds | ASSUMED; browsing behaviour on a catalogue, and it is the number that makes aggressive prebuffering expensive |

### Estimation that eliminates (video player)

| Calculation | Result | What it rules out |
|---|---|---|
| 6,000 kbps times 4 seconds divided by 8 | 3.0 MB for one top-rendition segment | Rules out nothing alone; the next row is what it enables |
| 3.0 MB divided by 625 KB per second | 4.8 seconds to fetch the first segment at the top rendition | Rules out starting at the top rendition against a 1.5 second startup budget, on a connection that can sustain it in steady state |
| 400 kbps times 4 seconds divided by 8, divided by 625 KB per second | 200 KB, so 0.32 seconds | Rules out any argument that a low starting rendition is unnecessary. It is 15 times faster to first frame |
| Manifest at 20 KB plus initialisation segment at 100 KB, over 625 KB per second (ASSUMED sizes for a five-rung ladder and a single video track) | 0.19 seconds | Rules out treating manifest fetch as free on a slow link: at 200 KB per second the same 120 KB is 0.6 seconds, which is 40 percent of the whole startup budget |
| 0.19 plus 0.32 seconds | 0.51 seconds of network before first frame at the lowest rendition | Rules out blaming the network for a startup number above 1.5 seconds without first measuring player and decode time, which is the remaining second |
| Licence acquisition at an ASSUMED 300 ms round trip, placed after the first segment rather than beside it | 0.81 seconds instead of 0.51 | Rules out serialising licence acquisition. It is the cheapest 300 ms in the whole design, and it is available only if you start it with the manifest request |
| 5,400 second session times 0.3 percent | 16.2 seconds of stall budget for a whole film | Rules out any policy evaluated on averages. Sixteen seconds is the entire allowance, so the design target is the count of stalls, not their mean duration |
| 16.2 seconds divided by an ASSUMED 2 seconds to refill and resume after a stall | 8 stalls per film | Rules out a controller that reacts to every measurement. Eight events across ninety minutes is a hysteresis requirement stated as a number |
| 30 second buffer at 6,000 kbps | 22.5 MB, against a 40 MB ceiling | Rules out expressing the buffer only in seconds when memory binds: the same 30 seconds is 1.5 MB at the bottom rung, a 15 times swing |
| 40 MB ceiling at 400 kbps | 800 seconds of buffer | Rules out expressing the buffer only in bytes when data cost binds. The policy needs both a seconds target and a byte ceiling, and neither alone is right |
| 20 percent of sessions abandoning within 60 seconds, each having prebuffered 30 seconds at 1,800 kbps | 6.75 MB fetched and discarded per abandoned session | Rules out using the steady-state buffer target at startup. Start at 10 seconds and grow, which cuts the waste by two thirds |
| Switching down while 30 seconds are buffered at 3,500 kbps | up to 13.1 MB of already-paid bytes discarded if you flush | Rules out flushing the buffer on every downswitch. Switch at the append point by default, and flush only when a stall is otherwise unavoidable |
| A throughput estimate averaged over the last 5 segments | a 20 second window, so a step change takes 4 segments to be reflected | Rules out a throughput-only controller. A connection that collapses entering a tunnel stalls you within 2 segments, which is before the estimator has noticed |
| 3,500 kbps rendition on a 5,000 kbps link: 1,750 KB divided by 625 KB per second | 2.8 seconds to fetch 4 seconds of media, so the buffer grows 1.2 seconds per segment | Rules out expecting to reach a 30 second buffer quickly at high quality: 25 segments, which is 100 seconds of wall clock. The startup ramp is not optional, it is arithmetic |
| A 90 minute film with an ASSUMED one caption cue per 6 seconds, 900 cues at 120 bytes | 108 KB, so 0.17 seconds | Rules out segmenting captions for on-demand playback. Fetch the whole file once and stop thinking about it |
| Live with a 4 second segment and 2 segments of buffer before playback | at least 8 seconds of glass-to-glass latency before any network time | Rules out a sub-5 second live target with 4 second segments. Either the target moves or the delivery unit does, and that is a conversation to have out loud |

!!! tip "Say this out loud in the room"

    "My stall budget for a whole film is sixteen seconds, which is about eight events. That means the controller has to be biased toward not switching, and it changes what I optimise: I am designing for the count of decisions, not for the average bitrate."

### V0, the simplest thing that works (video player)

**Predict first: with a single file and the browser's own controls, what fails first, and is it the network or the player?**

??? note "Show answer"

    The network, and specifically the mismatch between one encoded bitrate and a population of connections. A 3,000 kbps file on a 1,600 kbps connection plays at roughly half real time, so it stalls forever rather than occasionally: the viewer never catches up. Nothing about the player is wrong. This matters because it tells you the first upgrade is delivery, not controls, and candidates who reach for a custom interface first have solved a problem they do not yet have.

```
+---------+   +------------------+   +------------------+
| Browser |-->| One MP4 file,    |-->| CDN, byte range  |
| video   |   | native controls, |   | requests         |
| element |   | native captions  |   +------------------+
+---------+   +------------------+
```

**Caption.** V0 points the browser's own video element at a single encoded file served over a content delivery network, and uses the platform's controls and its native caption rendering.

Why this is correct for a small catalogue of short clips:

- Keyboard operation, focus behaviour, screen reader labels, picture in picture, system media keys and full screen all work, and none of it is code you wrote or maintain.
- Byte range requests give you seeking for free, and the delivery network caches whole files trivially because there is one URL per video.
- A caption track attached to the element renders with the platform's own styling, which respects the viewer's system caption preferences, something a custom renderer has to reimplement deliberately.
- There is no manifest, no controller, no buffer policy and no state machine, which is four categories of bug you do not have.

!!! warning "When not to use even this much"

    For a silent decorative loop under ten seconds, do not attach controls, do not preload, and consider whether an animated image is a better fit than a decoding pipeline. The measurement that justifies a real player is a rebuffer ratio or a startup number segmented by connection type. If you do not have those numbers yet, adding a player library is buying a solution before observing the problem.

### Breaking point, then V1 (video player)

**Breaking point one: one bitrate cannot serve a population whose connections differ by a factor of fifteen.**

What fails: at 200 KB per second and a 3,000 kbps encode, the player receives 0.53 seconds of media per second of wall clock. The buffer never fills, so the session is a permanent alternation of two seconds of video and four seconds of spinner. At the other end, a 25,000 kbps viewer is watching a soft picture with bandwidth to spare.

How you observe it: rebuffer ratio and startup time segmented by connection class, not aggregated. Aggregated, this looks like a mediocre average. Segmented, it is two completely different products.

V1 replaces the file with a ladder and adds the controls that a ladder makes necessary.

```
+------------------+   +------------------+
| Throughput and   |-->| Bitrate          |
| buffer level     |   | controller       |
+------------------+   +------------------+
                                |
                                v
+------------------+   +------------------+
| Manifest of      |-->| Segment fetcher, |
| 5 rungs          |   | chosen rung      |
+------------------+   +------------------+
                                |
                                v
+------------------+   +------------------+
| Custom controls, |<--| Media buffer,    |
| quality, caption |   | append and play  |
+------------------+   +------------------+
```

**Caption.** V1 fetches a manifest describing five renditions, downloads each segment at a rung chosen by a controller that reads recent throughput and the current buffer level, appends segments to a media buffer, and exposes a quality and caption menu that the native controls cannot provide. The media buffer's state is what feeds the first box, closing the loop.

| Change | Justified by | Cost you accept |
|---|---|---|
| Segmented delivery with a ladder | A fifteen times spread in connection speed | A manifest format, an encoding pipeline, and many more cache objects at the delivery network |
| A bitrate controller | Nothing else chooses a rung | Every stall and every quality complaint is now attributable to code you wrote |
| Start at the lowest rung, ramp up | 4.8 seconds against a 1.5 second startup budget | The first few seconds look soft, which is a visible, defensible trade |
| A 10 second startup buffer target that grows to 30 | 6.75 MB discarded per abandoned session | Slightly less protection in the first minute, which is when connections are also least likely to have been measured |
| Custom controls | A quality menu and a caption menu have no native equivalent | You have taken over an accessibility contract, which breaking point two is about |

!!! warning "When not to move to adaptive delivery"

    If your content is short clips inside a feed, a single mid-rung encode plus a poster image is often better than adaptivity, because a ten second clip finishes before a controller has learned anything. The measurement that justifies the ladder is rebuffer ratio at the tenth percentile of connection speed. Cheaper alternative first: two encodes chosen once at session start from a connection hint, which captures most of the benefit with none of the controller.

### Breaking point, then V2 (video player)

**Breaking point two: the controller reacts too late, and the controls you built are quietly less usable than the ones you removed.**

The first failure is temporal. A throughput estimate over the last five segments is a 20 second window. When a connection collapses on entering a tunnel, the buffer drains in real time while the estimator is still averaging the good measurements. With a 30 second buffer and a collapse to a tenth of the previous rate, you have roughly 30 seconds before a stall and an estimator that will need 4 segments, or about 40 seconds at the degraded rate, to reflect reality. You stall.

How you observe it: the distribution of buffer level at the moment of a downswitch. A healthy controller downswitches while the buffer is still comfortable. A throughput-only controller downswitches at a buffer level near zero, because that is when the average finally moved, and the histogram makes it obvious.

The second failure is accessibility, and it has no automatic signal at all. The moment you replace native controls, you own focus order, keyboard commands, state announcement, caption rendering and caption styling. Nothing errors. Nobody pages you. The player simply becomes unusable for some people.

```
+--------------+   +----------------+  +-----------------+
| Throughput   |-->| Hybrid         |->| Rung choice     |
| estimate     |   | controller     |  | with hysteresis |
+--------------+   | buffer level   |  +-----------------+
+--------------+   | dominates when |
| Buffer level |-->| it is low      |
+--------------+   +----------------+
+--------------+   +----------------+  +-----------------+
| Media events |-->| Player state   |->| Controls, ARIA  |
| from element |   | machine, one   |  | live region,    |
+--------------+   | source of truth|  | caption layer   |
                   +----------------+  +-----------------+
```

**Caption.** V2 feeds both a throughput estimate and the current buffer level into one controller whose decisions are damped by hysteresis, and derives every visible control state from a single explicit player state machine rather than from raw media element events.

| Change | Justified by | Cost you accept |
|---|---|---|
| Buffer level as a first-class control signal | Downswitches happening at near-zero buffer | The controller has two inputs that can disagree, so the arbitration rule must be written down, not implied |
| Hysteresis on every switch | An 8 stall budget for a whole film | The player is slower to take advantage of a genuinely improved connection, which is the correct side to err on |
| An explicit state machine | "Is it buffering" is not a boolean, and event ordering differs across implementations | One more layer between the platform and the interface, which must be kept honest with a conformance test |
| Caption rendering owned by the player | Custom controls overlap the platform's caption region | You must reimplement caption positioning and respect user caption preferences, which is real work with a legal dimension |
| Keyboard model and announcements | Native keyboard behaviour was deleted when native controls were | An accessibility test suite that runs in continuous integration, plus manual passes that cannot be automated |

!!! warning "When not to build a hybrid controller"

    For short-form content that finishes inside one buffer fill, the controller never gets to make a second decision, so the hybrid adds tuning surface and no outcome. The measurement that justifies it is the buffer-level-at-downswitch histogram above. Cheaper alternative first: keep the throughput estimator and add a single panic rule, switching to the lowest rung whenever the buffer falls below 5 seconds. That one rule captures most of the benefit.

### Deep dive one: what a throughput estimate cannot tell you, and what the buffer can

I choose this dive because the bitrate controller is the only part of a player that is genuinely a control system, and because the interesting content is not the algorithm but the conditions under which each signal is trustworthy.

**Signal one: recent throughput.** Measure bytes divided by wall clock time for each completed segment.

| Property | Reality |
|---|---|
| What it measures well | The steady-state capacity of a stable connection |
| Lag | One segment minimum, and the length of your averaging window in practice |
| Systematic bias | It measures the connection while you were using it, so idle periods between segments make it optimistic |
| Failure case | A step change, which is the exact case that causes stalls |

**Signal two: buffer occupancy.** How many seconds of media are decoded and ready.

| Property | Reality |
|---|---|
| What it measures well | Whether you are currently winning or losing, integrated over all causes |
| Lag | None. It is a state, not an estimate |
| Systematic bias | It says nothing about capacity, so on its own it cannot pick a rung, only a direction |
| Failure case | Startup, when the buffer is empty for reasons that have nothing to do with the connection |

The two signals are complementary in precisely the way that makes a hybrid obvious: throughput is a good estimator with bad latency, and buffer level is a bad estimator with no latency.

**The controller, in three blocks by purpose.**

Block one, decide what the rule must do at each buffer level rather than writing a formula first. Below a low threshold, survival is the only goal and quality is irrelevant. Above a high threshold, you are safe and may pursue quality. Between them, follow the throughput estimate with a margin.

Block two, put numbers on the thresholds using the budgets already derived. The low threshold is 5 seconds, because at the top rung a segment takes 4.8 seconds to fetch on the reference connection, so anything less than one segment of headroom means the next fetch decides whether you stall. The high threshold is 20 seconds, chosen as two thirds of the 30 second steady-state target so that upswitching happens while there is still room to be wrong.

*Before reading the next block, say why the safety factor below is applied to the throughput estimate and not to the buffer thresholds.*

Block three, write the rule and its damping.

- Choose the highest rung whose bitrate is at or below 0.8 times the throughput estimate. On a 5,000 kbps estimate that permits 4,000 kbps, so the 3,500 rung, not the 6,000 rung.
- If the buffer is below 5 seconds, drop to the lowest rung immediately and ignore the estimate. It is the only signal that has not lagged.
- If the buffer is above 20 seconds and the estimate has permitted a higher rung continuously for 3 segments, move up exactly one rung.
- Never move up more than one rung at a time. Never move up within 10 seconds of a downswitch.

The safety factor exists because the estimate is optimistic by construction, and because a rung that exactly matches capacity has zero margin to build buffer. The thresholds do not need a safety factor because the buffer level is measured, not estimated.

The error most readers make: tuning the up rule and leaving the down rule symmetric. Downswitching is cheap and reversible. Upswitching too eagerly costs a stall, and a stall costs an eighth of your entire budget for a film.

**Where the switch happens, which is a separate decision from whether to switch.**

| Strategy | Latency of the change | Bytes discarded | When to use |
|---|---|---|---|
| Switch at the append point, next segment onward | Up to the current buffer depth, so up to 30 seconds | None | The default for both directions |
| Replace buffered segments ahead of playback | Seconds | Up to 13.1 MB at 3,500 kbps with a full buffer | Only when a stall is otherwise unavoidable |
| Flush everything and refill | Immediate visual change, and a probable stall | Everything buffered | Only on a user-initiated quality change, where the user has asked for the cost |

**The three special cases that are not steady state.**

| Case | Why the steady-state rule is wrong | What to do instead |
|---|---|---|
| Startup | The buffer is empty and the estimate does not exist | Start at the lowest rung, target a 10 second buffer, then hand over to the steady-state rule |
| Seek | The buffer is discarded, and the 500 ms budget allows 0.32 seconds of fetch at the lowest rung and nothing at the top | Seek at a low rung and ramp, exactly as at startup |
| User override | The user has expressed a preference that outranks your estimate | Honour it, but keep the panic rule, and tell the user when you cannot sustain their choice rather than silently ignoring it |

!!! warning "When not to write your own controller"

    If you are already using a delivery library that ships a tuned controller, replacing it is a large risk for a benefit you cannot measure without a large audience. The measurement that justifies your own is a specific, reproducible failure of the existing one, such as the buffer-level-at-downswitch histogram being wrong for your traffic mix. Tuning the existing controller's thresholds is almost always the cheaper first move.

### Deep dive two: rebuilding what the native element gave away

This dive is deliberately not more control theory. It is the bill for a decision made in V1, and it is the part of a player answer that is usually left unreached, because the transport discussion eats the clock. Note that it is also not a focus-trapping exercise: dialog focus management belongs with the modal case study, and what is specific here is that a media player has continuous state that changes without user action.

**The state machine, because "buffering" is not a boolean.** The platform emits a stream of media events whose ordering varies, and reading them directly produces controls that flicker.

| State | Entered when | What the interface shows |
|---|---|---|
| Idle | Nothing loaded | A poster and a play affordance |
| Loading | Manifest or first segment in flight | A determinate or indeterminate progress affordance, not a paused button |
| Playing | Time is advancing | Pause affordance, live time updates |
| Paused | User paused | Play affordance, time frozen |
| Stalled | Time is not advancing and the buffer is empty | A spinner, and only after a delay, discussed below |
| Seeking | A seek was requested and the target is not yet ready | Scrub position at the target, not at the old position |
| Ended | Playback reached the end | Replay affordance, and no spinner ever |
| Failed | Unrecoverable error | A message naming what to try, not an error code |

Two rules that come from experience rather than the table:

- Do not show a spinner until the stall has lasted an ASSUMED 250 ms. Shorter interruptions are invisible in the picture and very visible if you flash a spinner over them, and 250 ms is roughly the threshold at which a viewer registers an interruption as one.
- Seeking and stalled are different states even though both are "waiting". A viewer who caused the wait tolerates it. A viewer who did not is watching your product fail.

**The keyboard model.** The native element had one. You deleted it. This is the replacement, and its rows are not arbitrary: they are the set that viewers have learned from every other player.

| Key | Action | Note |
|---|---|---|
| Space or K | Toggle play and pause | Space must not also scroll the page when the player has focus |
| Left and Right arrow | Seek back and forward 5 seconds | Only when the player region or the scrubber has focus |
| J and L | Seek back and forward 10 seconds | The learned convention for coarse scrubbing |
| Up and Down arrow | Volume | Only when the volume control has focus, otherwise arrow keys collide with seeking |
| M | Mute toggle | Must announce the resulting state, not the action |
| F | Full screen toggle | Must return focus to the player on exit, not to the top of the document |
| C | Captions toggle | Must announce which track became active |
| 0 to 9 | Seek to that tenth of the duration | Meaningless for live, so it must be disabled rather than silently wrong |
| Home and End | Seek to start and end | End on a live stream means the live edge, which is a different thing |

The trap in this table is scope. If these keys are bound globally, they fight the page, and typing a comment beneath the video seeks the film. Bind them to a focusable player region and make that region's focus visible, which is a requirement anyway.

**The scrubber is a real control, not a styled bar.** It needs a role that means slider, a minimum, a maximum, a current value and a human-readable version of that value, because "3,742" is not a time. It needs to be operable by keyboard with the same increments as the arrow keys above. On a live stream its maximum moves, and a control whose maximum changes every second while a screen reader is reading it is a problem you must solve by not announcing every change.

**Announcing state without flooding.** A live region that announces every buffering event, every quality change and every time update is worse than silence.

| Event | Announce | Why |
|---|---|---|
| Play and pause | Yes, politely | The user did it and expects confirmation |
| Captions on or off, and language | Yes, politely | It is a mode change the user requested |
| Stall longer than 3 seconds | Yes, once | The user needs to know the product is waiting, not broken |
| Automatic quality change | No | The user did not ask, cannot act, and will hear it dozens of times |
| Time updates | No | Available on demand from the scrubber's value |

**Captions, which is where the legal and the technical meet.**

- Fetch the whole caption file for on-demand content. It is 108 KB, so there is no engineering reason to be clever.
- Position cues so they never sit under the controls. The most common defect in a custom player is a caption hidden behind a control bar that appears on mouse move, which means the caption disappears exactly when the viewer touches the mouse.
- Respect the viewer's system caption preferences for size, colour and background where the platform exposes them. A custom renderer that ignores them is a regression against the native element, which honours them.
- For live content, captions arrive with the stream and lag it. Decide and state the acceptable lag, and make caption absence visible rather than silent, because a viewer who depends on captions cannot tell the difference between "no captions yet" and "captions are broken".

!!! warning "When not to build custom controls"

    If your product needs nothing beyond play, pause, seek and captions, the native controls are better than what you will build, and they are free. The measurement that justifies replacing them is a specific control the platform does not expose, most often a quality selector or a branded scrubber that a designer can defend in numbers. The cheaper thing first: keep native controls and add a single external quality control beside the player.

### Failure modes and evaluation (video player)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Metastable failure | A regional network dip makes every client downswitch and retry, which increases request volume, which slows delivery further | Jittered retry, a cap on retries per segment, and a controller that gives up on a rung rather than retrying it |
| Thundering herd | On a live stream, every client requests the same newly published segment in the same second | Jitter the manifest refresh, and rely on an origin shield so one origin fetch serves the herd |
| Cache stampede | The newest live segment is not yet at the edge and thousands of misses reach the origin together | Request coalescing at the edge, and a client that tolerates a not-yet-available response by waiting rather than retrying immediately |
| Hot key | The live edge segment is the only object anyone wants, so one cache key takes all the traffic | This is the delivery network's problem to solve and yours to know about; see [modern-system-design.md](modern-system-design.md) |
| Clock skew | The client computes which live segment should exist from its own clock and requests a segment from the future | Derive the live edge from the manifest, never from the local clock, and treat repeated not-found responses as a skew signal |
| Retry storm | A failing segment is retried aggressively by every client that reaches it | Bounded retries with jitter, then a rung change, then a visible failure. Three strategies in order, not one repeated |
| Memory exhaustion | The buffer is expressed in seconds and grows to hundreds of megabytes at a high rung on a long film | A byte ceiling as well as a seconds target, and eviction of buffered media well behind the playhead |
| Silent accessibility failure | Custom controls ship without keyboard bindings or announcements, and no counter moves | Automated checks in continuous integration for roles, names and focus order, plus a manual pass per release that is a release gate |

Service level indicators to publish:

- Startup time, click to first frame, at p75 and p95, segmented by connection class.
- Rebuffer ratio, and separately the count of stalls per hour of watch time.
- Mean sustained bitrate, and the distribution of rung changes per hour.
- Buffer level at the moment of a downswitch, as a histogram. This is the controller's health metric and it has no substitute.
- Seek to first frame at p95.
- Playback failure rate, split by network, decode and licence causes, because they have different owners.
- Caption availability, meaning the fraction of playbacks where a required caption track loaded successfully.

Alert on playback failure rate and on caption availability. Alert on rebuffer ratio by error budget burn rate rather than by instantaneous value, because a regional network event will always spike it and paging on a spike teaches people to ignore the page.

Re-checking against the non-functional requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Startup under 1.5 seconds at p75 | Yes at 0.51 seconds of network on the reference connection, with roughly a second of headroom for player, decode and licence. No on the 200 KB per second link, where the manifest alone is 0.6 seconds, and that case needs a smaller manifest or a preloaded one |
| Rebuffer under 0.3 percent | Plausible with hysteresis and the panic rule, and directly measurable as 8 stalls per film. It is a target the controller is tuned against, not a property the design guarantees |
| Seek under 500 ms at p95 | Yes when seeking at the lowest rung, which is why seek reuses the startup path rather than the steady-state one |
| 30 second buffer, 40 MB ceiling | Yes, with both expressed and the smaller of the two winning at any moment |
| Keyboard operable and exposed to assistive technology | Only if the deep dive two work actually ships and stays shipped. This is the requirement most likely to regress silently, which is why it has a release gate rather than a dashboard |
| Captions available and legible | Yes for on demand. For live, availability is best effort and the design must show its absence rather than hide it |

### Where this answer is a simplification (video player)

- **The encoding ladder is a design artefact, not a given.** Real ladders vary per title, because a talking-head interview and a fast-panning sport clip need different bitrates for the same perceived quality. Per-title and per-scene encoding changes every number in the estimation table.
- **Codec selection is a compatibility matrix, not a preference.** Which codecs a device can decode in hardware, at which resolutions, is queryable at runtime and varies enormously. Choosing wrong drains a battery or fails silently to software decode.
- **Protected playback is its own subsystem.** Licence acquisition, key rotation, output protection and per-device robustness levels add failure modes that look like network failures and are not. The interface is specified in [Encrypted Media Extensions](https://www.w3.org/TR/encrypted-media/).
- **Delivery formats are worth reading in the original.** HTTP Live Streaming is specified in [RFC 8216](https://www.rfc-editor.org/rfc/rfc8216), and the segmented-delivery alternative is ISO/IEC 23009-1, which is a paid standard and is therefore named rather than linked. The buffering interface is [Media Source Extensions](https://www.w3.org/TR/media-source-2/), and the element's own behaviour is in the [WHATWG HTML media elements section](https://html.spec.whatwg.org/multipage/media.html).
- **Captions have a format and a legal floor.** [WebVTT](https://www.w3.org/TR/webvtt1/) is the format, and the relevant conformance criteria live in [WCAG 2.2](https://www.w3.org/TR/WCAG22/), which is the document to cite when someone proposes shipping without captions to hit a date.
- **Low latency live is a different design.** Reaching a few seconds of glass-to-glass latency requires delivering partial segments as they are produced, which changes the manifest, the fetch loop and the controller, and it interacts badly with the hysteresis this answer relies on.
- **Advertising insertion breaks every assumption here.** Ad breaks change the timeline, the buffer, the state machine and the meaning of the scrubber, and they are usually the reason a player is custom in the first place.

*Specification links point at canonical locations. Specifications are revised, so check the version banner before quoting a date. Last checked structure of this list: August 2026.*

### Level delta (video player)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Describes a player architecture | States that the platform already ships a player and names the requirement that forces a replacement | Asks whether the requirement is worth the accessibility contract it costs, and is willing to keep native controls |
| Delivery | Says adaptive bitrate | Computes 4.8 seconds for a top-rung first segment and starts low | Notices that the startup buffer target and the steady-state target must differ, and sizes the waste at 6.75 MB per abandoned session |
| Controller | Picks by measured bandwidth | Adds buffer level and hysteresis, with thresholds derived from segment duration | Names the buffer-level-at-downswitch histogram as the controller's health metric, and asymmetrises the up and down rules |
| Buffer | Expresses it in seconds | Adds a byte ceiling for memory-constrained devices | Shows the two expressions disagree by fifteen times across the ladder and takes the smaller at every moment |
| Controls | Builds a custom bar | Adds keyboard bindings and a slider role | Scopes the key bindings to a focus region, defines what is announced and what is suppressed, and makes the accessibility pass a release gate |
| Failure handling | Adds retries | Names stall storms and jitters the retries | Names the metastable pattern, orders retry then rung change then visible failure, and refuses to derive the live edge from a client clock |
| Scope discipline | Builds low latency live because it is impressive | Defers it and says why | Shows that a 4 second segment implies at least 8 seconds of latency, so the requirement and the delivery unit must be negotiated together |

What each level added, in one line each:

- **Mid to senior:** every threshold in the controller now comes from a budget rather than from a default, and the startup path is treated as a separate regime.
- **Senior to staff:** the invisible costs are made visible (accessibility regression, discarded bytes, the metric that reveals a bad controller), and at least one impressive feature is renegotiated rather than accepted.

---

## Case study 8: Chat application

Chat is the prompt where the interviewer is most likely to be running a backend round under a frontend title. If the conversation drifts to fan-out and shard keys, that is a fine conversation to have, and it is in [modern-system-design.md](modern-system-design.md). What is genuinely a client problem is the part nobody can solve on the server: what the interface shows in the second between pressing send and the server agreeing, and what it shows after a week offline.

### The prompt (chat application)

"Design the client side of a chat product: how messages get in and out, how they are ordered, and what happens offline."

### Questions that fork the design (chat application)

| Question worth asking | If the answer is A the design becomes | If the answer is B it becomes |
|---|---|---|
| One device per user, or several? | One: local order is the order, and unread is a local number | Several: read state and message state must converge across devices, so unread becomes a synced cursor rather than a counter |
| Is the message list append-only? | Append-only: a monotonic sequence is the whole model | Edits, deletions and reactions: every message is a mutable entity with a version, and the client cache must be normalised rather than a list |
| Who assigns order? | The server, per conversation: gaps are detectable with an integer comparison | Client timestamps: you cannot detect a gap at all, and you inherit clock skew as a correctness bug |
| Is the content end-to-end encrypted? | No: the server can order, search, preview and count unread messages | Yes: the client owns search, previews and unread counts, and the outbound queue holds ciphertext plus key state that must not be lost |
| How long may the client be offline? | Minutes: replay from a server-side buffer | Days: a delta endpoint keyed by a cursor, plus an explicit "too far behind, resync" path that the interface must be able to show |
| How large is the largest group? | Small conversations: fan-out of anything is free | Hundreds of members: typing signals and read receipts become the dominant traffic, not messages, which is a counterintuitive result worth deriving |
| Are attachments in scope? | No: a message is a row | Yes: upload lifecycle is separate from message lifecycle, so a message can exist in a sent state whose attachment is still uploading |
| Is the unread count authoritative on the server? | Locally computed: cheap, and wrong across devices | Server-authoritative: one more thing to sync, and the one users notice immediately when it is wrong |

### Requirements (chat application)

Functional:

- Send a message and see it immediately, before any server acknowledgement.
- Receive messages in the order the server assigned, and notice when one is missing.
- Compose and send while offline, and deliver on reconnect without duplicates.
- Show conversation list state (last message, unread) without loading every conversation.
- Show a coarse typing signal, and read state per conversation.
- Survive a tab close, a crash and a week of disconnection without losing a composed message.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Conversations per user, median | 40 | ASSUMED; the median is what the common path must be fast for |
| Conversations per user, p95 | 400 | ASSUMED; a heavy user after several years, which is the case that breaks storage |
| Messages retained locally per conversation | 100 | ASSUMED; roughly a screen and a half of scrollback, which is what people actually revisit |
| Stored size of a message | 500 bytes | ASSUMED; short text plus identifiers, sender, two timestamps and receipt state |
| Local echo of your own message | under 50 ms | ASSUMED; sending must feel like typing, not like submitting a form |
| Send to delivered acknowledgement | under 1 second at p95 | ASSUMED; beyond a second the pending state becomes the thing the user is watching |
| Supported offline window | 7 days | ASSUMED; a holiday, and the number that decides whether a delta can be unbounded |
| Group size, p95 | 500 members | ASSUMED; a large team channel, and the case where signal traffic overtakes message traffic |
| Busy conversation message rate | 5 messages per second | ASSUMED; a live discussion, used to size render and receipt work |
| Client clock skew | up to 60 seconds | ASSUMED; unsynchronised device clocks drift by tens of seconds routinely, and this is the number that kills timestamp ordering |
| Synchronous key-value storage ceiling | 5 MB | ASSUMED; the commonly available practical limit for the simple browser store, and it is also synchronous, which matters more than the size |
| Slow link | 200 KB per second | ASSUMED, matching case studies 5 to 7 so the transfer numbers stay comparable |
| Concurrent clients reconnecting after a deploy | 100,000 | ASSUMED; used only to size backoff, and it is the number to ask the interviewer for |

### Estimation that eliminates (chat application)

| Calculation | Result | What it rules out |
|---|---|---|
| 400 conversations times 100 messages times 500 bytes | 20 MB of local message data at p95 | Rules out the synchronous 5 MB key-value store, and rules out holding all conversations in memory on a low-end device |
| 40 conversations times 100 messages times 500 bytes | 2 MB at the median | Rules out designing eviction for the median user. The median fits in memory comfortably, so eviction exists for the tail and should be measured against the tail |
| 400 conversations fetched one request each, at an ASSUMED 6 concurrent connections and 80 ms per request | 5.3 seconds of startup sync | Rules out a per-conversation sync endpoint. You need one delta endpoint keyed by a single cursor |
| 400 conversation heads at an ASSUMED 200 bytes each (last message preview, unread cursor, participants hash) | 80 KB, so 0.4 seconds on the slow link | Rules out downloading message bodies at startup. The conversation list is 250 times smaller than the messages behind it |
| 7 days offline at an ASSUMED 30 relevant messages per hour, times 500 bytes | 5,040 messages, 2.5 MB, so 12.5 seconds on the slow link | Rules out an unbounded delta. Past a threshold, the correct response is to reset to conversation heads and lazily reload bodies |
| 500-member group where 5 people type, each emitting a signal per keystroke at 10 per second, fanned to 500 members | 25,000 messages per second for one conversation | Rules out per-keystroke typing signals outright. This is the single number that changes how people think about chat traffic |
| The same 5 typists coalesced to one signal every 3 seconds with a 5 second expiry, aggregated server-side into one frame per member every 2 seconds | 250 messages per second for the conversation | Rules out client-driven fan-out of any signal in a large group. The client's job is to ask for aggregation, not to send less often |
| Per-message per-member read receipts at 5 messages per second in a 500-member group | 2,500 receipt events per second | Rules out per-message receipts in large groups |
| A read cursor per member instead, updated at most once every 2 seconds | 250 updates per second, and the same information | Rules out the belief that receipts are inherently expensive. A cursor is one integer that subsumes every message below it |
| 60 seconds of clock skew at 5 messages per second | up to 300 messages that a timestamp sort can place wrongly | Rules out ordering by client timestamp under any circumstances, including as a tie-break |
| A server-assigned per-conversation sequence, compared as integers | gap detection is one comparison per received message | Rules out any need for global ordering across conversations. Order matters within a conversation and nowhere else, and noticing that saves you a distributed problem |
| 100,000 clients reconnecting with a fixed 1 second backoff after a deploy | 100,000 connection attempts in the same second | Rules out fixed backoff. With full jitter over 30 seconds the same herd is 3,300 per second, a thirty times reduction for one line of code |
| An ASSUMED 20 messages composed during a realistic offline session, at 500 bytes | 10 KB | Rules out worrying about the size of the outbound queue. Its problems are ordering, duplication and poison, not bytes |
| An ASSUMED 1 percent of sends retried after an ambiguous timeout, at an ASSUMED 100 sends per active user per day | 1 duplicate per active user per day without client-side deduplication | Rules out trusting the network to be exactly once. The client must mint the identity of the message before it ever leaves |
| 40,000 locally retained messages scanned linearly at an ASSUMED 0.5 microseconds per message | 20 ms for a full local search | Rules out building a local search index for the retained window. Full-history search is a server feature, and mixing the two in one interface is a product decision, not an indexing one |

!!! tip "Say this out loud in the room"

    "In a five-hundred-person group, typing signals at one per keystroke are twenty-five thousand messages a second and the actual chat is five. So the interesting client design here is the signals, not the messages, and I want to spend my time there."

### V0, the simplest thing that works (chat application)

**Predict first: with a three second poll, what fails first, the latency or the request volume?**

??? note "Show answer"

    Latency, and it fails at a number you can state. The mean delay before a message appears is half the poll interval, which is 1.5 seconds against a 1 second acknowledgement budget, so the product is already outside its requirement with a perfectly healthy server. Request volume fails later and more visibly: 1,200 requests per user per hour, nearly all returning nothing. Volume is the tempting answer because it sounds like the systems answer, and naming latency first shows you checked the requirement rather than the instinct.

```
+---------+   +----------------+   +------------------+
| Compose |-->| POST message   |-->| Server stores    |
| box     |   | wait for 200   |   | and returns id   |
+---------+   +----------------+   +------------------+
+---------+   +----------------+
| Message |<--| GET since=id   |
| list    |   | every 3 s      |
+---------+   +----------------+
```

**Caption.** V0 posts each message and waits for the response before showing it, and polls a single endpoint every three seconds for anything newer than the last identifier it holds.

Why this is correct for a small internal tool:

- There is no local store, no queue, no reconnection logic and no ordering code. The server's response is the truth and the interface is a projection of the most recent response.
- The `since` cursor means the poll is already a delta, so the design does not have to change shape when volume grows, only its transport.
- Duplicates cannot occur, because nothing is retried automatically and the user sees the failure.
- At a handful of concurrent users the entire request volume is a rounding error on any server.

!!! warning "When not to use even this much"

    For a support widget with one agent and a conversation that lasts minutes, a ten second poll is better than three seconds and much better than a socket, because a socket must be reconnected, heartbeated and cleaned up while the widget is usually closed. The measurement that justifies moving is the p95 gap between a message being stored and being displayed, compared against your own requirement, not against how modern the transport feels.

### Breaking point, then V1 (chat application)

**Breaking point one: the poll interval is the latency, and the send path makes the interface feel slow even when it is not.**

Two failures share one cause, which is that the network is on the critical path of things it does not need to be on.

- Receiving: a 3 second poll gives a 1.5 second mean and a 3 second worst case, against a 1 second budget. Shortening the interval to 500 ms would meet the budget and multiply request volume by six, nearly all of it empty.
- Sending: waiting for the response before rendering makes the composer's responsiveness equal to the round trip. On a slow connection the user presses send and watches nothing happen, which reads as a broken button and produces double sends.

How you observe it: for receiving, the distribution of the delay between server store time and client display time, which under polling has a distinctive flat shape between zero and the interval. For sending, the rate of duplicate messages with near-identical content from the same user within a few seconds, which is the measurable footprint of an unresponsive composer.

```
+---------+   +----------------+   +------------------+
| Compose |-->| Render locally |-->| Send over socket |
| box     |   | as pending,    |   | with a client id |
+---------+   | client id      |   +------------------+
              +----------------+            |
                                            v
              +----------------+   +------------------+
              | Reconcile on   |<--| Server assigns a |
              | acknowledgement|   | sequence, echoes |
              +----------------+   +------------------+
                     |
                     v
              +----------------+   +------------------+
              | Message list,  |<--| Inbound messages |
              | sorted by seq  |   | from the socket  |
              +----------------+   +------------------+
```

**Caption.** V1 renders the outgoing message immediately in a pending state carrying a client-generated identifier, sends it over a persistent socket, and reconciles the pending message when the server echoes it back with an assigned sequence number.

| Change | Justified by | Cost you accept |
|---|---|---|
| Persistent socket | A 1.5 second mean delay against a 1 second budget | A connection lifecycle: heartbeats, reconnection, backoff, and a visible connection state |
| Optimistic local render | Composer responsiveness equal to the round trip | The list now contains two kinds of message, pending and confirmed, and every piece of interface logic must handle both |
| Client-generated identifier per message | Duplicate sends observed as a real rate | The identifier must be stable across retries, so it cannot be generated at send time inside a retry loop |
| Server-assigned per-conversation sequence | Ordering cannot come from clocks with 60 seconds of skew | The server must supply it, which is a contract negotiation, and a client that cannot get it has no ordering story at all |
| Read cursor rather than per-message receipts | 2,500 receipt events per second in a large group | Receipt state is coarser: you know someone read up to a point, not which messages they looked at |

!!! warning "When not to open a socket"

    If your product is a notification-shaped experience where messages arrive rarely and the tab is usually in the background, a socket is a connection you pay to keep alive for nothing, and background throttling will suspend it anyway. The measurement that justifies it is the display-delay distribution against your budget. Cheaper alternative first: a one-way server-sent event stream, which reconnects itself, carries its own resumption cursor, and needs no upstream channel because sends can stay on ordinary requests.

### Breaking point, then V2 (chat application)

**Breaking point two: everything that happens when the socket is not connected.**

The socket is the happy path, and the happy path is not where chat clients fail. Three failures arrive together.

- Composed messages die with the tab. A user types on a train, loses signal, closes the tab, and the message is gone with no error and no trace.
- Reconnection is silent about gaps. The socket resumes and new messages flow, but the messages sent while it was down are simply absent, and the list looks complete.
- Retries duplicate. A send that timed out ambiguously is retried, and at an assumed 1 percent ambiguity rate that is one duplicate per active user per day.

How you observe each: the count of sessions ending with an unsent composed message; the rate of detected sequence gaps per hour and, more damningly, the rate of gaps discovered only when a user scrolled; and the rate of messages arriving with an already-seen client identifier.

```
+----------+   +-----------------+   +-----------------+
| Compose  |-->| Durable outbound|-->| Single flight   |
| box      |   | queue, on disk  |   | per conversation|
+----------+   +-----------------+   +-----------------+
                                              |
                                              v
+----------+   +-----------------+   +-----------------+
| Delta    |-->| Socket, with    |-->| Gap detector,   |
| sync by  |   | jittered        |   | backfill range  |
| cursor   |   | reconnect       |   +-----------------+
+----------+   +-----------------+            |
                                              v
                                     +-----------------+
                                     | Message list,   |
                                     | sorted by       |
                                     | sequence        |
                                     +-----------------+
```

**Caption.** V2 persists composed messages to disk before any send is attempted, sends one message per conversation at a time so order is preserved, resynchronises from a stored cursor whenever the socket reconnects, and checks every inbound message for a sequence gap before it reaches the list, backfilling the missing range when it finds one.

| Change | Justified by | Cost you accept |
|---|---|---|
| Durable outbound queue | Sessions ending with an unsent message | A storage schema to migrate, a quota that can be exceeded, and messages that can outlive the interface that created them |
| Single flight per conversation | Message 2 succeeding while message 1 is retrying reorders the conversation | Slower drain of a large queue, which is the correct trade because chat is a sequence and not a set |
| Gap detection and backfill | Gaps discovered by users rather than by code | A backfill request path, and an interface that can insert history above the current scroll position without moving it |
| Delta sync by cursor on reconnect | 5.3 seconds of per-conversation sync, and 12.5 seconds for a 7 day delta | A cursor to store and version, and a reset path when the delta is too large |
| Jittered reconnect backoff | 100,000 simultaneous reconnects after a deploy | Slightly slower reconnection for the individual user, in exchange for the service being available when they reconnect |

!!! warning "When not to build the durable queue"

    If sends only ever happen with a live connection and a failed send can honestly show an error the user will act on, an in-memory retry with a visible failed state is simpler and has no storage failure modes. The measurement that justifies durability is the fraction of send attempts that begin while the connection is down or the page is being hidden. Cheaper alternative first: block the compose box when disconnected, which is honest, unpopular, and eliminates the entire category.

### Deep dive one: ordering a list that contains messages the server has not seen yet

I choose this dive because the moment you render optimistically, your list contains two kinds of object with two different identities and no common ordering key, and every visible chat bug (jumping messages, duplicates, a message that lands in yesterday) is a symptom of that one problem.

**The two identities, stated plainly.**

| Identity | Minted by | Exists from | Used for |
|---|---|---|---|
| Client identifier | The client, at compose time | Before any network attempt | Deduplication, reconciliation, and finding the pending row to replace |
| Sequence number | The server, per conversation | On acceptance | Ordering, gap detection, and the sync cursor |

Minting the client identifier at compose time rather than at send time is the load-bearing detail. If it is generated inside the send function, every retry produces a new identity and the server has no way to recognise the duplicate.

**Sorting a list where some rows have no sequence number.** Give every pending message a sort key of the highest sequence number known at the moment it was composed, plus a local monotonic counter as the fractional part. Pending messages then sit after everything currently known and in the order the user typed them, which is the only order the user believes in.

**Worked example, in three blocks by purpose.**

Block one, set up the state that produces the bug. The conversation has messages up to sequence 100. The user composes two messages while the socket is briefly down. Both get sort keys 100.1 and 100.2. Meanwhile a colleague sends two messages that the server assigns 101 and 102.

Block two, reconnect and watch the naive implementation misbehave. The socket resumes and delivers 101 and 102. If the interface sorts by sequence and treats the pending messages as having no sequence, they either sink to the top or float to the bottom, and the user watches their own two messages jump position for no reason they can perceive.

*Before reading the next block, say why the fix cannot be "sort pending messages by their local timestamp". The answer is in the requirements table.*

Block three, apply the rule and reconcile. The pending keys 100.1 and 100.2 sort below 101 and 102 by construction, so the user's messages stay where they were and the colleague's arrive beneath them. When the server accepts the first pending message and assigns it 103, the client finds the pending row by client identifier, replaces it in place, and its sort key becomes 103, moving it below 102. That one movement is honest and expected: the message went out after the colleague's did.

Local timestamps cannot be the fix because a device clock can be 60 seconds off, so a message composed now can sort into yesterday. The error most readers make here is assuming the skew problem only affects other people's messages. It affects your own, on the device the user is looking at.

**Gap detection, which is a comparison and not a subsystem.**

| Situation | Detection | Response |
|---|---|---|
| Received sequence is exactly last plus one | Normal | Append |
| Received sequence is greater than last plus one | A gap of known size | Request the missing range, hold the newer messages until it fills or a timeout expires |
| Received sequence is less than or equal to last | A duplicate or a late delivery | Discard, and count it |
| Reconnect with a stored cursor | Not a gap, a resume | Delta sync from the cursor, and reset entirely if the delta exceeds the threshold |

Two rules that turn this from a comparison into a design:

- Hold newer messages briefly rather than rendering them above the gap. Rendering them and then inserting the missing ones underneath makes the list rearrange itself while the user reads, which is worse than a short delay.
- Give the hold a timeout, an ASSUMED 2 seconds, and after it render the newer messages with an explicit marker for the missing range. A visible marker is much better than either a silent hole or an indefinite wait, and the marker can be retried by tapping it.

**Inserting history without moving the reader.** Backfill and scrollback both prepend content above the current viewport, which shifts everything down by the height of what you added. Anchor on a specific element: record the top visible message and its pixel offset before the insertion, and restore that offset afterwards. This is the same class of problem as the position mapping in case study 5, applied to pixels instead of characters, and it is the difference between a list that grows and a list that lurches.

!!! warning "When not to implement gap detection"

    If the transport already guarantees ordered, gapless delivery for the lifetime of a connection and you resynchronise fully on every reconnect, gap detection is code that will never fire and will therefore never be correct when it does. The measurement that justifies it is the observed rate of sequence discontinuities in production telemetry. Cheaper alternative first: log the discontinuity, resynchronise the whole conversation, and see how often it happens before building the incremental path.

### Deep dive two: an outbound queue that survives the tab

This dive is deliberately not another convergence discussion. Case study 6 owns merge semantics and presence, and chat does not need either: messages are immutable and append-only, so there is nothing to converge. What chat has that a document does not is a send that must survive the process, and that is a durability and idempotency problem, not an ordering one.

**What the queue must guarantee, and what it must not promise.**

| Guarantee | Mechanism | Why it is achievable |
|---|---|---|
| A composed message is never lost | Write to durable storage before the first send attempt | The write is 500 bytes and completes in milliseconds, so it can precede the network without being felt |
| A message is delivered at most once | Client identifier minted at compose, server deduplicates on it | The client is the only party present for the whole retry sequence |
| Messages in one conversation arrive in the order composed | Single flight per conversation | Concurrency across conversations is free; concurrency within one is what reorders |
| A permanently failing message cannot block the others | Attempt limit, then quarantine | Without this, one rejected message stops a conversation forever |
| Exactly-once delivery | Nothing | It is not achievable, and claiming it in an interview invites a follow-up you will lose |

**The message lifecycle, as states the interface must be able to draw.**

| State | Meaning | What the user sees |
|---|---|---|
| Queued | Written to disk, not yet attempted | The message, with a subtle pending indication |
| In flight | An attempt is outstanding | Identical to queued, deliberately, because the distinction is not actionable |
| Sent | Server acknowledged and assigned a sequence | Normal appearance |
| Retrying | Attempt failed with a retryable classification | Pending indication, plus a connection state elsewhere in the interface rather than per message |
| Failed | Attempt limit reached, or a non-retryable rejection | An explicit failed state with a retry affordance and, if known, a reason |
| Quarantined | Failed and moved aside so the queue can drain | The message stays visible in place, marked, and does not block later messages |

Do not show "in flight" as a distinct visual state. The user cannot act on it, it changes several times per second on a bad connection, and the flicker is read as instability.

**Retry classification, which is where most queues go wrong.**

| Failure | Class | Action |
|---|---|---|
| Network unreachable, socket closed | Retryable, transient | Retry with backoff, no attempt-limit penalty while offline |
| Server 5xx or timeout | Retryable, ambiguous | Retry with the same client identifier, and count the attempt |
| Rejected: too large, banned content, no permission | Not retryable | Fail immediately with a reason. Retrying a rejection is how a queue becomes a loop |
| Rejected: conversation no longer exists | Not retryable | Fail and offer to copy the text, because the text is the user's work and it is about to disappear |
| Unknown or unparseable response | Retryable, capped low | Two attempts, then quarantine. An unknown failure repeated is a poison message |

**Backoff arithmetic, with the herd number attached.** Use exponential backoff with full jitter: wait a random duration between zero and the current cap, doubling the cap from 1 second up to 30 seconds. With 100,000 clients returning after a deploy, a fixed 1 second backoff produces 100,000 attempts in one second, while full jitter across the 30 second cap produces roughly 3,300 per second. The randomness, not the exponent, is what does the work, and that is the sentence worth saying.

**Ordering across the boundary, which is the subtle bug.** Suppose messages A and B are queued for the same conversation. A fails and enters retry, B is ready. If B is sent, the conversation now reads B then A, permanently, because the server assigns sequences in the order it accepts them. Single flight per conversation prevents it. The cost is that a slow message delays the ones behind it, and the correct response to that is quarantine after the attempt limit, not concurrency.

**When the queue outlives the user's intent.** A message queued seven days ago and finally sent is, at best, confusing. Give queued messages an expiry, an ASSUMED 24 hours, after which they move to failed rather than being sent, with the text preserved so the user can decide. A message is a communication with a time context, and delivering it late is not the same as delivering it.

**Storage failure is a real branch.** The durable store can refuse a write when the device is out of space or the origin's quota is exhausted. If your queue silently swallows that error, you have built something less reliable than not having a queue, because the user now trusts it. Surface the failure at compose time, keep the message in memory, and count quota errors as a first-class metric.

!!! warning "When not to persist the queue to disk"

    On a desktop application that is always connected and whose window closing is a deliberate user act, an in-memory queue with a confirmation prompt on close is simpler and has no quota, migration or expiry problems. The measurement that justifies durability is the fraction of send attempts starting while the page is hidden or the connection is down. If that fraction is under a percent, an honest error is a better product than a queue nobody can see.

### Failure modes and evaluation (chat application)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Thundering herd | A deploy drops every socket and all clients reconnect and request delta sync together | Full-jitter backoff, plus a delta endpoint that can return a reset instruction under load rather than computing a large delta |
| Retry storm | An ambiguous failure is retried by every client at the same interval | Exponential backoff with jitter, an attempt limit, and non-retryable classification for rejections |
| Poison message | One rejected message retries forever and blocks its conversation's queue | Attempt limit then quarantine, with the message left visible and marked |
| Duplicate delivery | The same message rendered twice after a retry that actually succeeded | Client identifier minted at compose, deduplicated on receipt as well as on the server |
| Silent gap | Messages sent during a disconnect never arrive and nothing notices | Sequence comparison on every inbound message, a bounded hold, and a visible marker for an unfilled range |
| Clock skew | A message sorts into yesterday because the device clock is wrong | Never sort by client time. Display it, and label it as the sender's time if you show it at all |
| Metastable failure | Reconnect storms make sync slow, which causes timeouts, which cause more reconnects | Cap concurrent sync work, prefer a reset over a large delta under load, and make the client's timeout longer than the server's slow path |
| Storage exhaustion | The durable queue or the message cache exceeds the origin quota and writes start failing | Evict oldest conversation bodies first, never evict the outbound queue, and surface quota failure at compose time |
| Unbounded growth | Local message storage grows with every conversation ever opened | Retain 100 messages per conversation and evict whole conversations by last-access, measured against the 20 MB p95 figure |

Service level indicators to publish:

- Local echo latency at p99, which should be independent of network state by construction.
- Send to acknowledgement at p95, split by whether the send started online or from the queue, because they are different products.
- Outbound queue age at p95, and the count of quarantined messages.
- Sequence gap rate per hour, and separately the fraction of gaps that were filled before the user could have noticed.
- Duplicate render rate, per thousand messages.
- Reconnect to first delivered message, at p95.
- Storage quota failures per thousand sessions.
- Sessions ending with a queued message, which is the direct measure of the problem the queue exists to solve.

Alert on sessions ending with a queued message, on quarantine rate, and on quota failures. Do not alert on reconnect counts, which are dominated by normal mobile behaviour and will train the team to ignore the channel.

Re-checking against the non-functional requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Local echo under 50 ms | Yes by construction, since the render precedes both the disk write acknowledgement and the send. If the disk write is awaited before rendering, this requirement fails, which is the implementation detail to watch |
| Send to acknowledgement under 1 second at p95 | Yes on a live socket. Not applicable from the queue, which is why the metric is split rather than averaged |
| 7 day offline window | Yes, with the reset path. A pure delta would be 2.5 MB and 12.5 seconds, so the threshold that triggers a reset is the design's real answer to this requirement |
| Duplicates | Bounded, not eliminated. Client identifiers deduplicate the cases we can see, and the honest claim is at-most-once rendering rather than exactly-once delivery |
| Ordering within a conversation | Yes, from the server sequence plus single flight. Across conversations there is no ordering guarantee and none is needed |
| 20 MB local storage at p95 | Yes with 100-message retention and conversation-level eviction. Both numbers should be verified against real storage telemetry rather than trusted |
| Typing and read state in a 500-member group | Only with server-side aggregation and cursor-based receipts. The client cannot fix this alone, and saying so is part of the answer |

### Where this answer is a simplification (chat application)

- **End-to-end encryption changes the client's job entirely.** Search, previews, unread counts, notification content and the outbound queue's contents all move into the client, and key state becomes something the queue must not lose. That is a different case study.
- **Multi-device convergence is harder than it looks.** Read cursors that move backwards, messages sent from a phone appearing on a laptop that was offline, and per-device notification suppression are a synchronisation problem in their own right.
- **Attachments have their own lifecycle.** A message can be accepted while its attachment is still uploading, which means the message states in this answer are a simplification of a product with two independent state machines per row.
- **Notifications are a separate delivery system.** Background delivery, permission state and the relationship between a notification and an unread cursor are governed by the [Push API](https://www.w3.org/TR/push-api/) and [Service Workers](https://www.w3.org/TR/service-workers/), and a socket-based design says nothing about the case where the tab is closed.
- **The transports are specified, and the choice matters more than it appears.** The bidirectional option is [RFC 6455](https://www.rfc-editor.org/rfc/rfc6455); the one-way option with built-in reconnection and a resumption cursor is [server-sent events in the WHATWG HTML standard](https://html.spec.whatwg.org/multipage/server-sent-events.html). The durable store is [IndexedDB](https://www.w3.org/TR/IndexedDB/), whose behaviour under storage pressure is part of the specification and part of your design.
- **Background throttling is real and undocumented in most designs.** A hidden tab's timers, sockets and storage operations are throttled or suspended by the browser and by the operating system, so any queue that assumes it will drain while backgrounded is assuming something the platform does not promise.
- **The server side of this is a large system.** Fan-out, per-conversation partitioning, retention and search are the other half, and they are answered in [modern-system-design.md](modern-system-design.md) rather than here.

*Specification links point at canonical locations. Specifications are revised, so check the version banner before quoting a date. Last checked structure of this list: August 2026.*

### Level delta (chat application)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Describes a socket and a message list | Notices the prompt is a client prompt and states which parts belong to the backend round | Asks which of multi-device, encryption and large groups is in scope, because each one changes a different half of the design |
| Ordering | Sorts by timestamp | Uses a server-assigned per-conversation sequence and explains the 60 second skew problem | Designs the sort key for pending messages so optimistic rows never jump, and names the bug that key prevents |
| Sending | Posts and waits, or posts optimistically without an identifier | Mints a client identifier at compose time and reconciles on acknowledgement | Enforces single flight per conversation, explains why concurrency reorders, and refuses to claim exactly-once |
| Offline | Says messages are queued | Persists the queue and classifies retryable failures | Adds quarantine, an expiry on stale messages, and treats a storage quota failure as a visible product state |
| Large groups | Treats signals like messages | Derives 25,000 messages per second and coalesces | Replaces per-message receipts with cursors, and states plainly that the fix requires server aggregation the client can only request |
| Reconnect | Reconnects immediately | Adds exponential backoff | Adds full jitter with the herd arithmetic, and adds a reset path for a delta that is too large to compute under load |
| Failure handling | Mentions retries | Detects gaps and backfills | Bounds the hold before rendering past a gap, makes an unfilled gap visible, and anchors scroll position when history is inserted |
| Scope discipline | Builds search, reactions and threads | Defers them and says why | Points out that the 40,000 locally retained messages scan in 20 ms, so local search is free and full-history search is a server feature, and refuses to build an index |

What each level added, in one line each:

- **Mid to senior:** the client stops trusting the network for identity, ordering and timing, and each of those three now has a named mechanism with a number behind it.
- **Senior to staff:** the queue is treated as a system with its own failure modes and its own lifecycle, the large-group traffic result is derived rather than assumed, and at least one feature is shown to need no engineering at all.

---

## 10. Practice problems

Two sets doing two different jobs. The blocked set rehearses the arithmetic you just watched, one move at a time, with the topic already named. The interleaved set hides the topic, so your first task is deciding which tool the prompt is asking for.

That first task is the one a frontend design round actually scores. An interviewer cannot see whether you have read about virtualization. They can see whether you reached for it at the right moment and refused it at the wrong one.

Work everything on paper or out loud before opening an answer. Rubrics, model answers and diagnosed wrong answers for all eleven problems are in Section 11.

### 10a. Blocked set: five problems, same shape as the worked examples

**B1. Split an interaction budget between the client path and the server path.**

A filter panel on a product listing must respond to a checkbox toggle within 200 ms, measured from the input event to the next painted frame, at p95 on a mid-tier phone.

Measured on that device, with a profiler, on this page:

| Step | Cost |
|---|---|
| Event handler plus state update | 12 ms |
| Re-render of the checkbox control alone | 3 ms |
| Re-render of the 500 row result list | 25 ms |
| Style and layout for the list | 18 ms |
| Paint and composite | 9 ms |
| Server p95 for the filtered query | 140 ms |

Does the obvious design meet the budget, and which two options does the arithmetic eliminate?

??? note "Show answer"

    Path A, the obvious one, where the checkbox reflects the server's answer: 12 plus 140 plus 25 plus 18 plus 9 equals 204 ms. That is over a 200 ms budget before any queueing, and these are independent p95 figures that do not compose into a p95, so the real number is worse.

    Path B, where the control paints first: 12 plus 3 plus 9 equals 24 ms to the first frame. The list updates later, at the same 204 ms, but that is a content update and not the response to the input.

    Eliminated, one: a design in which the checkbox's own visual state waits on the network. Nothing about a checkbox requires server confirmation to render as checked, and 140 ms of the 204 is spent proving something the client already knows.

    Eliminated, two: rendering the full 500 row list on the same frame as the interaction. That work is 25 plus 18 equals 43 ms, which is one task long enough to drop two frames at 60 hertz and to delay any input arriving during it by up to its own length.

    The third answer, and the one worth volunteering: check whether the round trip is needed at all. 500 rows at an assumed 400 bytes each is 200 KB, which you have very likely already fetched. Filtering in memory removes the 140 ms entirely: 12 plus 25 plus 18 plus 9 equals 64 ms, comfortably inside budget.

    State the threshold out loud, because that is what makes it a design decision rather than a preference: client-side filtering wins while the full result set fits one fetch you were going to make anyway. Above roughly a few thousand rows, or when the filter must respect permissions the client cannot evaluate, the round trip comes back and the split in Path B is how you stay inside the budget.

**B2. Size a virtualized table and pick the window.**

A grid renders 40,000 rows of 8 columns. The viewport is 800 px tall and rows are 48 px.

Assume 6 DOM nodes per cell once you count the cell element, its content wrapper and the interactive affordances inside it, so 48 nodes per row plus 1 for the row element, which is 49.

Assume 1.2 KB of retained memory per DOM node on a mid-tier device, covering the element, its computed style and its layout box. Assume style and layout cost 0.02 ms per node on the same device. Both are order-of-magnitude figures you should replace with a measurement from your own app the moment you have one.

Size the naive version, size the windowed version, choose the overscan, and say when column virtualization becomes necessary.

??? note "Show answer"

    Naive, all rows in the DOM: 40,000 times 49 equals 1,960,000 nodes.

    Memory: 1.96e6 times 1.2 KB equals about 2.35 GB. A mobile browser tab is killed long before that, so this is not a slow design, it is a design that does not run.

    Style and layout: 1.96e6 times 0.02 ms equals 39,200 ms, so about 39 seconds on the first render. Eliminated.

    Windowed: visible rows are 800 divided by 48 equals 16.7, so 17. With 8 rows of overscan above and below, 33 rows are mounted. 33 times 49 equals 1,617 nodes, which is 1,617 times 0.02 equals 32 ms of style and layout for a full remount, and roughly 1.9 MB of retained DOM.

    Where the overscan number comes from: at an assumed fling velocity of 2,000 px per second, 8 rows is 8 times 48 equals 384 px, which is 192 ms of runway. That covers one 32 ms render plus several dropped frames. Overscan is a latency buffer measured in milliseconds of scroll, not a taste.

    On a scroll tick you mount and unmount only the rows that entered and left the window, not all 33, so steady-state scroll cost is a few rows times 49 nodes, not 1,617.

    Column virtualization: at 8 columns each row is 48 nodes and the window is 1,617 nodes, which is fine. At 60 columns each row is 360 nodes, the window is 33 times 361 equals 11,913 nodes, and style and layout is 11,913 times 0.02 equals 238 ms. That is the number that forces horizontal windowing.

    When not to virtualize at all: below roughly 200 rows the naive DOM is 200 times 49 equals 9,800 nodes and about 196 ms of first layout, which is one visible pause on load and nothing thereafter. Virtualization costs you correct scrollbar geometry, find-in-page, keyboard navigation across unmounted rows, screen reader row counts and print. Do not pay that for a list you can measure at under a frame or two.

**B3. Work backwards from a first render budget to a byte budget.**

The primary content of a landing page must be visible within 2,500 ms at p75. Your p75 user is on a mid-tier Android phone with an effective 1.6 Mbps downlink and a 300 ms round trip.

Assume 1 ms of main-thread time per KB of uncompressed JavaScript for parse, compile and execute on that device. Assume a compression ratio of 3.3 to 1 for JavaScript, which is the order of magnitude modern text compression achieves on minified code. Assume 100 ms of server think time for the document.

The app currently ships 900 KB of compressed JavaScript and renders entirely on the client. Compute where it lands, then derive the byte budget that fits.

??? note "Show answer"

    Bandwidth in useful units: 1.6 Mbps is 1,600,000 bits per second, divided by 8 equals 200,000 bytes per second, so 200 KB per second. One KB costs 5 ms of transfer.

    Where it lands today:

    | Step | Arithmetic | Cost |
    |---|---|---|
    | Document round trip | 300 ms RTT plus 100 ms server | 400 ms |
    | JavaScript transfer | 900 KB divided by 200 KB per second | 4,500 ms |
    | Parse, compile, execute | 900 times 3.3 equals 2,970 KB, at 1 ms per KB | 2,970 ms |
    | **Total to first content** | | **7,870 ms** |

    Against a 2,500 ms budget that is 5,370 ms over, and no amount of caching fixes a first visit.

    Now derive the budget. Let x be compressed KB on the critical path. Transfer is 5x ms. Execution is 3.3x KB at 1 ms per KB, so 3.3x ms. Together 8.3x ms.

    2,500 minus 400 equals 2,100 ms available. 2,100 divided by 8.3 equals about 253, so roughly 250 KB compressed is the critical-path budget. Every KB you add costs 8.3 ms on this device, which is the sentence to write in the budget file.

    Eliminated: shipping the current 900 KB on any critical path, and any plan that treats "we will optimise later" as compatible with the stated budget.

    What server rendering does and does not buy, stated honestly. Assume the server-rendered HTML for the first screen is 60 KB compressed. Time to visible content becomes 400 plus 60 times 5 equals 700 ms. Time to interactive is still 400 plus 4,500 plus 2,970 equals 7,870 ms, because the same JavaScript still has to arrive and run.

    So server rendering converts a blank screen into a visible screen that ignores clicks for seven seconds. That is a real improvement and a real new failure mode, and saying both is the difference between a senior answer and a recited one. The fix for interactivity is still bytes: split by route so only the landing route's roughly 250 KB is critical, and defer or delete the rest.

**B4. Choose a debounce interval from typing speed, not from habit.**

A search box issues suggestion requests as the user types. The product does 1.2 million searches a day. Assume a median inter-keystroke interval of 180 ms for a competent typist, and a median query of 14 characters. Backend p95 for a suggestion query is 60 ms.

Pick the debounce interval, give the request rate for each option, and name what the chosen interval costs the user.

??? note "Show answer"

    No debounce: 14 requests per query. 1.2e6 times 14 equals 16.8 million requests a day. 16.8e6 divided by 86,400 equals about 194 per second on average, and at an assumed peak of three times average, about 583 per second. Every one of those results is rendered under a caret that has already moved.

    A 120 ms debounce: 120 is shorter than the 180 ms gap between keystrokes, so the timer expires between every pair of keys and fires anyway. It coalesces nothing and adds 120 ms to every suggestion. This is the single most useful number on the problem: a debounce below the inter-keystroke interval is pure added latency with no saving, and it is the most common thing people ship.

    A 250 ms debounce: it fires only when the user genuinely pauses. Assume two pauses per 14 character query, one mid-query and one at the end. That is 2 requests per query, 2.4 million a day, about 28 per second average. A 7x reduction against the 194.

    What the 250 ms costs: a user who stops typing waits 250 plus 60 equals 310 ms before suggestions appear. That is well past the roughly 100 ms at which a response reads as instantaneous, so the interface owes them a visible pending state, and the pending state must not reflow the list.

    The design that beats both, and the one to volunteer: debounce the network at 250 ms, but serve a local prefix cache on every keystroke so the visible list updates inside a frame and the network result replaces it when it lands. Assume 600 bytes per cached suggestion set and 200 distinct prefixes in a session, which is 120 KB, so the cache is free.

    Then say the correctness bug that the arithmetic hides: responses can arrive out of order. A slow response for "lapt" can land after a fast one for "laptop" and overwrite it. Debouncing reduces how often this happens and never prevents it. The fix is cancellation of the in-flight request plus a check that the response's query still matches the current input before it is applied.

**B5. Size the receive path of a collaborative client.**

A shared canvas has 25 participants. Each participant's pointer generates up to 120 position samples per second on a modern device.

Assume 0.15 ms on the receiving client to parse one message and apply it to state. Assume a coalesced cursor render costs 0.4 ms fixed plus 0.05 ms per moved cursor.

Compute the main-thread load for the naive design, then for the fixed one, and state the participant count below which none of this machinery is worth building.

??? note "Show answer"

    Naive receive: 24 other participants times 120 samples equals 2,880 messages per second.

    Parse and apply: 2,880 times 0.15 equals 432 ms per second, so 43 percent of the main thread doing nothing but ingesting cursors.

    Render per message: 2,880 times 0.4 equals 1,152 ms per second. That exceeds one second of work per second of wall clock, which means the client can never catch up. It is not slow, it diverges. Eliminated.

    Fix one, throttle the sender. Cursors are interpolated on the receiver, so an assumed 20 samples per second per participant is enough to look continuous. Inbound becomes 24 times 20 equals 480 messages per second, and parse cost becomes 480 times 0.15 equals 72 ms per second, about 7 percent.

    Fix two, coalesce the render to one frame. At 60 frames per second with at most 24 cursors changed per frame: 60 times (0.4 plus 24 times 0.05) equals 60 times 1.6 equals 96 ms per second, about 10 percent.

    Total after both: 72 plus 96 equals 168 ms per second, roughly 17 percent of the main thread. That leaves the budget the actual drawing work needs.

    The structural point, which scores higher than the arithmetic: cursors must not live in the same state container as the document. If they do, every one of those 480 messages a second invalidates every subscriber of the document state, and you have rebuilt the render-per-message cost through the back door.

    When not to build this: with 3 participants the naive path is 2 times 120 equals 240 messages per second, so 36 ms of parse and 96 ms of render, about 13 percent, with no throttling and no coalescing. Interpolation, coalescing and a separate presence channel are machinery you would not write.

    The number that flips it is the participant count multiplied by the send rate. Once inbound messages per second exceeds roughly the frame rate, coalescing starts paying, and once parse cost passes about 10 percent of a second, throttling does.

### 10b. Interleaved and cumulative set: six problems, topic not stated

These are unlabelled on purpose. Decide which tool applies before you design anything. Several reach outside this page: the [trade-off atlas](index.md#the-trade-off-atlas) and the [failure library](index.md#the-failure-library) on the hub, [Modern System Design](modern-system-design.md) for the server the client is talking to, [ML System Design](ml-system-design.md) for what a ranked surface owes its ranker, and the [system design question bank](/Interview-Questions/System-design/).

Twenty to thirty minutes each, out loud, with a timer running.

**I1.** Four teams work on one web application. A staff engineer has proposed splitting it into four independently deployed micro-frontends so that teams stop blocking each other's releases. Design it.

**I2.** Here is what runs today: a client-rendered checkout page, one bundle, 410 KB compressed. Last Tuesday a date formatting dependency was upgraded. The bundle is now 460 KB, and field interaction latency at p75 went from 180 ms to 520 ms. Continuous integration was green and the release shipped. Diagnose and plan the fix.

**I3.** Our continuous integration performance job scores the product page at the top of its scale on every build. Support tickets say the page is slow, and the sales team has started demoing on office wifi to avoid it. Find the disagreement.

**I4.** Design the browser client for a market data screen: 200,000 concurrent users, 500 instruments, each instrument updating about 10 times a second.

**I5.** Our feed is ordered by a ranking service owned by another team. They have asked us for "better data from the client". Design what the client sends and what it guarantees.

**I6.** Users on our infinite feed report seeing the same post twice, and report losing their place when they navigate to a post and come back. Both are reproducible. Fix it.

!!! warning "Honesty note about the interleaved set"

    You will score lower here than on the blocked set, and it will feel like the reading did not take. The gap is the design. The blocked set measures whether you can run a move once someone has named it; the interleaved set measures whether you can pick one, which is the only thing an interviewer is able to observe. A drop of one or two rubric levels between the sets is the expected result, not a signal to reread the page.

---

## 11. Self-assessment rubrics

### The master rubric

Every problem in Section 10 is scored on the same four criteria, with named levels rather than numbers. A visible score crowds out the comment, and the comment is the part that changes your next answer.

| Criterion | Developing | Adequate | Strong | Exceptional |
|---|---|---|---|---|
| Problem framing and scoping | Starts drawing components before asking what the interface must do or on what device | Asks questions, but no answer would move a budget, a boundary or a box | Every question has two branches and the taken branch is stated. Names the interaction budget, the byte budget and the device class as separate numbers | Names the constraint nobody raised (accessibility conformance, locale count, an unupgradable browser floor) and the number at which it blocks the design |
| Design and trade-off reasoning | One architecture, presented as the answer. No alternative named | Alternatives named, chosen by familiarity or by what is current | Each choice is settled by a number computed in front of the interviewer, and the rejected option is named with the condition under which it wins | Identifies which decision is expensive to reverse (the rendering boundary, the component's public API, the state ownership split) and sequences the work around it |
| Depth in at least one area | Box level everywhere. Libraries named instead of mechanisms | Goes one layer deeper when pushed | Volunteers one dive that reaches implementation detail: the focus order of a widget, the cache key and its invalidation, the exact frame the optimistic update paints on | The dive includes how it fails silently, what would detect it in the field, and what the rollback costs |
| Communication and drive | Waits to be told what to cover next | Answers well, does not move the conversation | States the plan, keeps the clock, offers forks instead of asking permission | Absorbs a hostile follow-up by changing the design and naming which earlier claim is now void |

Score all four on every problem. Two Strongs and two Adequates is a passing senior answer. Four Adequates with nothing Strong is the profile most often written up as "hire, one level down", because nothing in it was memorable.

### Rubric notes, blocked set

??? note "B1 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks whether the 200 ms is measured to the painted frame or to the handler returning, because that moves 27 ms | Takes 200 ms as a network budget |
    | Trade-offs | Splits the interaction into the part the client already knows and the part that needs the server | Optimises the 140 ms and leaves the structure alone |
    | Depth | Notices that 43 ms of list work is one long task and says what it blocks | Sums the numbers and stops |
    | Communication | States the threshold at which client-side filtering stops being correct | Says "filter on the client" with no bound |

    Model answer: I need to know where the 200 ms is measured, because the frame is 27 ms after the handler. Assuming it is to the painted frame: the naive path is 12 plus 140 plus 25 plus 18 plus 9, so 204 ms, already over, and those are independent p95 figures that will compose worse than that.

    So I split the interaction. The checkbox's own state is client-authoritative and paints in 12 plus 3 plus 9, so 24 ms. The list is a separate update that lands when the data does. Second, the list work is 43 ms in one task, which drops two frames and swallows any input that arrives inside it, so the list render is windowed or yielded regardless of where the data comes from.

    Third, I would check whether the round trip is needed. 500 rows at an assumed 400 bytes is 200 KB, probably already fetched, and filtering in memory puts the whole interaction at 64 ms. I would keep the server path for result sets above a few thousand rows or where the filter depends on permissions the client cannot evaluate.

    Wrong answer: "Add a loading spinner." Reveals that a budget has been read as a perception problem. A spinner is the right affordance for the 140 ms list update and does nothing about a checkbox that takes 204 ms to look checked, which is the part the user blames.

    Wrong answer: "Cache the filtered results so it is fast the second time." Reveals a confusion between a hit rate and a budget. The first toggle of every distinct filter combination is a miss, and combinations are the product of the facets, so the miss rate is where the user lives.

    Wrong answer: "Move the filtering to a web worker." Reveals reaching for a mechanism before locating the cost. Of the 64 ms client path, 27 ms is style, layout and paint, which a worker cannot do. Workers help when the cost is computation, and here most of it is not.

    Compare your process: did you separate the two things the click has to do before you tried to make either one faster?

??? note "B2 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks the row height and viewport height before proposing anything, because they set the window | Proposes virtualization from the row count alone |
    | Trade-offs | Prices what virtualization takes away: find-in-page, scrollbar geometry, print, screen reader counts | Treats virtualization as free |
    | Depth | Derives overscan from scroll velocity in milliseconds of runway | Picks an overscan of 5 because it is a common default |
    | Communication | Gives the row count below which not to do this, and the column count above which to do more | Answers only the question asked |

    Model answer: all rows is 40,000 times 49 nodes, so 1.96 million. At an assumed 1.2 KB per node that is 2.35 GB, which the tab does not survive, and at an assumed 0.02 ms per node the first layout is 39 seconds. So the naive version is not slow, it does not run.

    Windowed: 800 divided by 48 is 17 visible rows. I take 8 rows of overscan each way because at an assumed 2,000 px per second fling that is 384 px, or 192 ms of runway, which covers a 32 ms window render and several dropped frames. 33 rows times 49 is 1,617 nodes, about 32 ms of full layout, and steady-state scrolling only mounts the rows crossing the boundary.

    Columns: at 8 columns this is fine. At 60 columns the window is 11,913 nodes and 238 ms, which is where I would add horizontal windowing. And I would not virtualize at all below roughly 200 rows, where the naive DOM is under 10,000 nodes, because virtualization costs find-in-page, native scrollbar geometry, print output, and a correct row count for assistive technology, all of which then have to be rebuilt by hand.

    Wrong answer: "Use a virtualization library." Reveals that the sizing was skipped. The library does not choose your row height, your overscan or your accessibility strategy, and those are the three things that decide whether the grid is usable.

    Wrong answer: "Paginate at 100 rows per page instead." Reveals a product change dressed as an engineering one. It may well be correct, but it is a different interface, and proposing it without saying so is how a design conversation goes sideways.

    Wrong answer: "Set overscan to 20 to be safe." Reveals a buffer chosen by anxiety. 20 rows each way is 57 mounted rows, 2,793 nodes and 56 ms per full render, which is a longer main-thread task in exchange for runway you derived no need for.

    Compare your process: did you say what virtualization takes away, or only what it gives?

??? note "B3 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks which device and which connection the p75 user is on before quoting any byte number | Optimises for the machine the developer owns |
    | Trade-offs | Converts the budget into a per-KB cost, so every future change is priced automatically | Gives a byte target with no derivation |
    | Depth | Separates time to visible content from time to interactive and says server rendering only moves the first | Presents server rendering as the fix |
    | Communication | Names the new failure mode server rendering introduces | Presents a strategy with only upside |

    Model answer: 1.6 Mbps is 200 KB per second, so a KB costs 5 ms of transfer and, at 3.3 to 1 compression and 1 ms per uncompressed KB, another 3.3 ms of main thread. Call it 8.3 ms per compressed KB.

    Today: 400 ms of document round trip, plus 900 divided by 200 equals 4,500 ms of transfer, plus 2,970 ms of execution, so 7,870 ms against a 2,500 ms budget.

    The budget: 2,500 minus 400 leaves 2,100 ms, divided by 8.3 gives about 250 KB compressed on the critical path. I would write that number into a continuous integration check so the next 50 KB import fails the build rather than the quarter.

    Server rendering gets first content to 400 plus 300, so 700 ms, assuming 60 KB of HTML. It does not move time to interactive, which is still 7,870 ms, so it converts a blank screen into a screen that ignores clicks. That is worth doing and it is not the fix. The fix is route-level splitting so the landing route stays near 250 KB.

    Wrong answer: "Use a content delivery network." Reveals a latency instinct applied to a bandwidth problem. A CDN removes some of the 300 ms round trip and none of the 4,500 ms of transfer or the 2,970 ms of execution, because the bytes still have to arrive and still have to run.

    Wrong answer: "Lazy load the components below the fold." Reveals that the measurement was skipped. Deferring components does nothing if they are in the same chunk, and the question is which of the 900 KB is on the landing route, which nobody has looked at yet.

    Wrong answer: "Switch to a smaller framework." Reveals a solution chosen at the level of the runtime rather than the application. A framework runtime is typically a small fraction of 900 KB; the rest is application code and dependencies, and a rewrite is the most expensive way to discover that.

    Compare your process: did you produce a per-KB price, or only a total?

??? note "B4 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks for typing speed, because the debounce interval is meaningless without it | Picks 300 ms because it is a common default |
    | Trade-offs | Shows that a debounce under the inter-keystroke interval saves nothing and costs latency | Assumes any debounce reduces requests |
    | Depth | Names out-of-order responses as a correctness bug that debouncing does not solve | Treats debouncing as the whole answer |
    | Communication | States what the chosen delay costs the user and what the interface owes them in exchange | Reports only the request saving |

    Model answer: the deciding quantity is the gap between keystrokes, an assumed 180 ms here. A 120 ms debounce expires inside that gap, so it fires on every key and adds 120 ms to every result: all cost, no saving. That is the number I would put on the whiteboard.

    250 ms sits above the gap, so it fires on a genuine pause. At an assumed two pauses per 14 character query that is 2 requests instead of 14, taking 194 per second down to about 28. The price is that suggestions appear 310 ms after the user stops, which is past instantaneous, so I show a pending state that does not reflow the list.

    Then I would keep the perceived response inside one frame with a local prefix cache, at an assumed 600 bytes per suggestion set and about 200 prefixes a session, so 120 KB. And separately from all of this, I cancel in-flight requests and check that a response's query still matches the input before applying it, because a slow reply for a shorter prefix can otherwise overwrite a fast reply for a longer one.

    Wrong answer: "Debounce at 100 ms so it still feels instant." Reveals that the coalescing mechanism was never modelled. At a 180 ms typing gap this is a per-keystroke request with 100 ms of extra latency attached, which is strictly worse than no debounce at all.

    Wrong answer: "Throttle instead of debounce." Reveals mechanism substitution without a goal. Throttling at 250 ms produces a steady stream of requests for prefixes the user has already passed, which is the behaviour debouncing exists to remove. Throttle is right for a continuous signal such as scroll, not for a query that is only meaningful when it stops changing.

    Wrong answer: "Cache the results so repeated queries are free." Reveals an optimisation aimed at the wrong axis. A cache helps the second time a prefix is typed; the 194 requests per second come from first-time prefixes inside a single query.

    Compare your process: did you ask how fast the user types before you chose a number?

??? note "B5 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks the participant count and the send rate, because the product of the two is the whole problem | Designs for an unstated number of users |
    | Trade-offs | Fixes the sender and the receiver separately and prices each | Applies one fix and declares victory |
    | Depth | Separates presence state from document state and says why they cannot share a store | Puts cursors in the same state as the document |
    | Communication | Gives the participant count below which none of this is built | Presents the full machinery as always correct |

    Model answer: inbound is 24 times 120 equals 2,880 messages a second. At an assumed 0.15 ms to parse and apply, that is 432 ms a second of main thread, 43 percent, before rendering. Rendering per message at an assumed 0.4 ms is 1,152 ms a second, which is more than a second of work per second, so the naive design does not fall behind, it diverges.

    Two independent fixes. On the sender, cursors are interpolated on the receiver, so 20 samples a second is enough and inbound drops to 480 messages, or 72 ms a second. On the receiver, coalesce to one render per frame: 60 frames times 0.4 plus 24 times 0.05 gives 96 ms a second. Together about 17 percent, which leaves room for the drawing the user actually came for.

    The structural part: presence lives in its own channel and its own store. If cursor updates write into the document state, every one of those messages invalidates every document subscriber and the coalescing is undone.

    And I would not build any of this for 3 participants: 240 messages a second is 36 ms of parse and 96 ms of render, about 13 percent, with no machinery. It starts paying when inbound message rate passes the frame rate.

    Wrong answer: "Use a web worker for the socket." Reveals a fix aimed at the wrong half. Moving parsing off the main thread does help with the 432 ms, but the 1,152 ms of render is main-thread work by definition and is the part that diverges.

    Wrong answer: "Use a CRDT so cursors merge correctly." Reveals a correctness mechanism proposed for a throughput problem. Cursor position is last-write-wins by nature; no convergence question arises, and a CRDT here adds metadata to a message you are already sending too often.

    Wrong answer: "Increase the send rate so it feels smoother." Reveals that smoothness has been located in the network rather than in the render. Interpolation between 20 samples a second produces motion at the frame rate; more messages produce more parsing.

    Compare your process: did you check whether the naive design was slow, or whether it was impossible?

### Rubric notes, interleaved set

??? note "I1 rubric, model answer and wrong answers (the answer is not to add it)"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks how often a release is actually blocked today and for how long, before designing anything | Starts comparing integration strategies |
    | Trade-offs | Prices duplicated runtime in milliseconds on every page load against hours of release coordination per month | Prices only the autonomy |
    | Depth | Names version skew, cross-app auth, error isolation and the observability seam as things you now own | Names bundle size as the only cost |
    | Communication | Declines with arithmetic and states the team count at which the answer flips | Either builds it or dismisses it as hype |

    Model answer: I would not split it yet, and here is the arithmetic. Assume each independently deployed app carries its own copy of the framework runtime and the shared design system at 45 KB compressed. Four copies is 180 KB against 45 KB shared, so 135 KB extra. At the 200 KB per second and 8.3 ms per compressed KB from B3, that is about 1,120 ms added to every first load, forever, for every user.

    What it buys is release independence. So I want the number: how many deploys were blocked last quarter, by how long, and was the blocker the build or the review process? If four teams deploy several times a day and the coordination cost is a few hours a month, we are buying back a few hours a month with a second of latency on every session.

    The threshold at which I change my mind, stated as quantities rather than opinions: when the number of teams that must agree on one release is large enough that a team routinely waits days rather than hours, when parts of the product have genuinely different lifecycles (an acquired product, a third-party embedded surface, a legacy admin console that must not be rewritten), or when one surface has a hard isolation requirement such as a different compliance boundary.

    What I would do instead, this quarter: route-level code splitting with one owner per route, code ownership rules that make review routing automatic, independent feature flags so a team can ship dark and enable on its own schedule, and a build that fails a route whose owner's tests fail without blocking the others.

    And the costs I would name before anyone commits: dependency version skew across apps, an auth session that must be shared without being duplicated, error boundaries and source maps that now span deploy units, and a performance budget that no single team owns.

    Wrong answer: designing the micro-frontend architecture competently, as asked. Reveals a candidate who optimises whatever they are pointed at. This prompt exists to see whether you will produce a number that contradicts the request.

    Wrong answer: "No, micro-frontends are an anti-pattern." Reveals the right instinct with no evidence. A refusal without arithmetic scores the same as compliance without arithmetic, because neither shows a decision process.

    Wrong answer: "Use module federation, it solves the duplication." Reveals a tool named in place of an analysis. Sharing a runtime across independently deployed units re-couples their upgrade schedules, which is most of the independence the proposal was for, and that trade is the actual design conversation.

    Compare your process: did you ask what the current release pain measures, or did you accept it as given?

??? note "I2 rubric, model answer and wrong answers (brownfield)"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Separates the startup cost of 50 KB from the interaction cost, and notices only the second matches the symptom | Attributes the regression to bundle size |
    | Trade-offs | Chooses a targeted fix over a revert, and says what the revert would have cost | Reverts and stops |
    | Depth | Names the specific per-call construction cost and computes it against the row count | Says "the library is slow" |
    | Communication | Fixes the process that let it ship, not only the code | Fixes the code |

    Model answer: first, the two numbers point at different things. 50 KB compressed is about 165 KB uncompressed, which at 1 ms per KB is 165 ms of extra parse and compile. That is a startup cost. Interaction latency moving from 180 to 520 ms means work is happening during the interaction, so the bundle growth is a symptom sitting next to the cause rather than the cause.

    Hypothesis I would test first: the new version constructs a formatter object per call instead of reusing one. Locale-aware formatter construction is expensive. Assume 0.8 ms per construction, measured on the target device rather than guessed. The order table renders 200 rows with 3 dates each, so 600 constructions equals 480 ms, which matches the 340 ms regression closely enough to be worth confirming.

    How I confirm rather than assume: record a profile of the real interaction on a throttled device, look at the long task attribution and the self time by function, and check the call count. One profile settles it in minutes and stops three people arguing for a day.

    Fix: hoist the formatter to one instance per locale and format combination, memoised at module scope. That is a small change, and it keeps the upgrade, which carried the correctness fixes we upgraded for.

    Then the process fix, which is the part that matters more than the patch. Continuous integration was green, so it was not measuring interaction. I would add a bundle byte budget that fails the build, an interaction benchmark on the checkout flow running on a throttled profile, and a field alert on interaction latency at p75 with the release annotated, so the next one is caught in hours rather than a week.

    What I would deliberately not do: revert. The upgrade was taken for a reason, and reverting hides the missing measurement that let a 340 ms regression ship silently. If the fix is not found within a stated time box, revert as a mitigation and keep the investigation open.

    Wrong answer: "Revert the dependency." Reveals mitigation mistaken for diagnosis. It restores the number and leaves the gap that let it through, so the next upgrade does the same thing.

    Wrong answer: "The bundle grew, so tree-shake it." Reveals a correlation followed instead of a mechanism. 165 ms of extra parse cannot produce a 340 ms interaction regression, because parse happens once at startup and the interaction happens later.

    Wrong answer: "Move the formatting to the server." Reveals a redesign chosen before the cost was located. It would work, it changes the caching story for the whole page, and it is a large answer to a problem that a one-line memoisation may close.

    Compare your process: did you check whether the two numbers described the same phase of the page's life?

??? note "I3 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks what the CI job actually loads, and on what hardware, before doubting the users | Trusts the score |
    | Trade-offs | Separates lab and field as different instruments with different jobs, and keeps both | Proposes replacing one with the other |
    | Depth | Prices the third-party payload in transfer and execution on the p75 device | Says "third-party scripts are slow" |
    | Communication | Proposes one measurement before any change | Proposes a list of optimisations |

    Model answer: a lab score and a field percentile disagree for structural reasons, so I would list them before touching the page. The lab job runs on build hardware with a synthetic network profile, a cold single load, no consent banner, no logged-in state, and usually no third-party tags, because those are often blocked or absent in the test environment.

    The specific gap I would check first: assume tag manager, analytics, session replay and a chat widget total 260 KB compressed. At the 200 KB per second and 8.3 ms per KB figures from B3, that is about 2,160 ms of transfer plus execution that continuous integration never sees. If the CI environment does not load them, the score is measuring a page that no user receives.

    Second gap: device mix. If 40 percent of sessions are on devices roughly four times slower than the CI machine, everything CPU-bound is four times its measured cost for those users, and a single score cannot show it.

    Third gap: statistics. One lab run is a point; a field metric is a distribution. A mean of 1.2 seconds with a p95 of 6 seconds is a product with a slow tail, and the mean is the number that hides it. I would report p75 and p95 and never the average.

    What I would do, in order: turn on real user monitoring with attribution so the slow sessions can be grouped by device, country and route; make the CI job load the third-party tags and run on a throttled profile matched to the p75 device; then give the third-party payload a byte budget with a named owner, because it is the only part of the page nobody currently owns.

    When not to add real user monitoring: below a few thousand sessions a day the percentiles are noise, and a throttled synthetic run plus a handful of real devices is cheaper and more actionable. Say that, because it is the honest boundary of the recommendation.

    Wrong answer: "The users are wrong, the score is objective." Reveals an instrument trusted over the thing it is a proxy for. The score exists to predict field experience; when they disagree, the field is the ground truth and the score is the hypothesis.

    Wrong answer: "Drop the third-party scripts." Reveals an engineering answer to a business decision. Some of those tags are revenue attribution. The engineering move is a budget, an owner and a loading strategy, not unilateral removal.

    Wrong answer: "Add real user monitoring and see what it says." Reveals monitoring proposed without a hypothesis. It is the right tool, and without naming what you expect to find you will collect a dashboard nobody reads.

    Compare your process: did you list the structural differences between the two instruments before choosing which to believe?

??? note "I4 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks whether the client ever sends, because that alone decides the transport | Chooses WebSockets by reflex |
    | Trade-offs | Sizes per-connection state and per-client message rate separately | Sizes neither |
    | Depth | Names the reconnect storm at deploy time and the sequence number that makes gaps detectable | Designs only the steady state |
    | Communication | Ties the transport choice to a row of the shared trade-off atlas rather than to preference | Asserts a transport |

    Model answer: the first question is whether the client sends anything on this channel. If prices flow one way, server-sent events carry it with automatic reconnection and plain HTTP semantics, and I would take that. The [trade-off atlas](index.md#the-trade-off-atlas) row on long polling against WebSockets against server-sent events puts the deciding quantities as messages per connection per minute and concurrent connections against per-connection memory, and only the second matters here.

    Connection state: 200,000 concurrent at an assumed 30 KB per connection of socket buffers and per-subscription state is about 6 GB across the fleet. At an assumed 50,000 connections per node that is 4 nodes, so I would run 6 for headroom and for a rolling deploy.

    Per-client message rate is the number that decides the subscription model. 500 instruments at 10 updates a second is 5,000 messages a second at the source. A client watching 20 instruments receives 200 a second, which at the 0.15 ms per message figure from B5 is 30 ms a second, about 3 percent, and is fine. A client subscribed to all 500 receives 5,000 a second, which is 750 ms a second and is not. So subscriptions are server-side filtered per connection, and that is a requirement, not an optimisation.

    Render coalescing: a price grid is read by a human, so I paint at most once per 100 ms and apply the latest value per cell, which is roughly a 10x reduction in layout work against painting every message.

    The failure I would design for explicitly: every deploy or load balancer restart disconnects all 200,000 clients at once, and without jitter they all reconnect inside the same second. That is a thundering herd from the [failure library](index.md#the-failure-library). The fixes are full-jitter backoff on the client, a staggered server drain, and a connection rate limit at the edge that sheds rather than queues.

    Resume semantics: prices are last-value-wins, so on reconnect the server sends a snapshot then a delta stream. Every message carries a monotonic sequence number per subscription so a client can detect a gap and ask for a fresh snapshot instead of silently displaying a stale price, which in this product is the expensive failure.

    Wrong answer: "WebSockets, because it is real time." Reveals a transport chosen by association. Bidirectionality is the thing WebSockets buys, and if the client never sends, you have paid for connection state you must now shed and rebalance for no benefit.

    Wrong answer: "Poll every second." Reveals a rate treated as the only variable. 200,000 clients polling once a second is 200,000 requests a second through the full request path, against 200,000 idle connections, and the latency is worse.

    Wrong answer: "Send all 500 instruments and let the client filter." Reveals a cost moved rather than removed. It is 750 ms a second of main thread per client, on every client, to save a filter on the server that runs once per subscription.

    Compare your process: did you ask which direction the data flows before naming a protocol?

??? note "I5 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks what the ranking team will train on, because that decides what the client must guarantee | Offers to send more events |
    | Trade-offs | Chooses a batching and delivery scheme priced against event volume | Sends one request per event |
    | Depth | Writes down the definition of an impression and says why the definition is the deliverable | Logs "views" with no definition |
    | Communication | Names position logging as the thing that makes their bias correction possible | Lists event names |

    Model answer: the useful question is what they cannot compute without us. A ranker trained on clicks alone learns that the top slot is good, because the top slot gets clicked. Correcting that needs the position each item was shown in, and the client is the only place that knows what was actually on screen. So the first deliverable is position, logged from the server-provided order rather than from the rendered index, since the rendered index can be changed by client-side filtering.

    The second deliverable is a written definition of an impression, because the offline training label and the online metric have to mean the same thing. I would propose: at least 50 percent of the item's area visible for at least 1 continuous second, measured with an intersection observer.

    The definition is then versioned, so that a change to it shows up in their data as a version change rather than as an unexplained model effect. This is the [ML System Design](ml-system-design.md) course's point about position bias and label definition, arriving on our side of the boundary.

    Volume, which decides the transport. Assume 30 impressions per session and 20 million sessions a day, so 600 million events, at an assumed 220 bytes each. That is 600e6 divided by 86,400 equals about 6,940 events a second and roughly 132 GB a day. One request per impression is eliminated immediately. Batching 50 events per request gives about 139 requests a second, which is a normal service.

    What the client must guarantee, and these are the parts people forget:

    - An outbound queue that survives a tab close, flushed with a keepalive send on page hide rather than a normal fetch the browser will cancel.
    - An event identifier, so a retried batch is deduplicated rather than double-counted. A duplicated impression looks to the ranker like a real engagement signal.
    - Sampling that is deterministic per session, so a sampled dataset is still a complete picture of the sessions it does contain.
    - A client clock that is never trusted for ordering. Events carry a monotonic sequence and the server stamps arrival time.

    What I would not send: raw dwell traces, keystrokes or anything with content in it. Log volume is cheap to add and expensive to remove, and every field becomes a privacy review.

    Wrong answer: "We will log clicks and views." Reveals no definition. Two teams will implement two definitions of a view within a month, and the disagreement will surface as an unexplained metric shift after a release.

    Wrong answer: "Send every event immediately so nothing is lost." Reveals volume not modelled. It is 6,940 requests a second for data nobody reads in real time, and it still loses the last batch when the tab closes, which is exactly the batch that contains the outcome.

    Wrong answer: "Let the ranking team read our analytics warehouse." Reveals a boundary avoided rather than designed. The warehouse has whatever definition analytics chose, which is usually not the one a training label needs, and the coupling means our analytics migration becomes their model incident.

    Compare your process: did you ask what they will train on, or did you start listing events?

??? note "I6 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Recognises two distinct bugs sharing one symptom, and separates them before fixing either | Treats "duplicates and lost place" as one issue |
    | Trade-offs | Names the case where offset pagination is still the right answer | Declares cursors always correct |
    | Depth | Explains why deduplicating on the client hides the skip rather than fixing it | Proposes client-side deduplication as the fix |
    | Communication | Says which fix is server-side and which is client-side, and who owns each | Presents one undifferentiated plan |

    Model answer: these are two bugs. The duplicates are a pagination bug and the lost place is a navigation state bug, and fixing one does not touch the other.

    Duplicates: offset pagination over a collection that is being written to. If 3 posts are inserted above the window while the user reads page 1, the request for offset 20 now starts 3 items earlier in the ordering, so 3 items repeat. Deletions produce the mirror image, where items are skipped and nobody notices, because a skipped item is invisible by construction.

    The fix is keyset pagination on a stable composite sort key, for example created time plus identifier, where the cursor names a position in an ordering rather than a count from the start. That is a server contract change, and the [Modern System Design](modern-system-design.md) storage material and the [system design question bank](/Interview-Questions/System-design/) both carry the ordering argument in more depth.

    Client-side deduplication by identifier is worth adding as a belt, and it is not the fix, because it removes the visible half of the symptom and leaves the invisible half. A design that makes a bug undetectable is worse than one that makes it obvious.

    Lost place: the feed's scroll position and its loaded pages live in component memory, so navigating away destroys them. The fix is to keep the restoration state where the navigation lives: the last cursor and the scroll offset go into history state keyed by the navigation entry, so a back navigation restores both. Restoring scroll before the content is measured produces a jump, so restoration waits for the window's rows to be measured, which is another reason the row height model in B2 matters.

    The related bug I would raise before being asked: new items arriving above the viewport must not be inserted while the user is reading, because that moves content under their thumb. They are buffered behind a control that says how many are waiting, and inserting them is an explicit user action.

    When cursors are the wrong answer: an admin table with numbered page controls, where a user must jump to page 47. Cursors cannot express that. There you keep offset pagination and either accept the instability or freeze the result set with a snapshot identifier for the duration of the session.

    Wrong answer: "Deduplicate by identifier on the client." Reveals a symptom treated as the defect. It hides duplicates and leaves skips, which are the same bug with the visible half removed.

    Wrong answer: "Cache the list so navigating back is instant." Reveals the second bug misread as a performance problem. A cache without a restored scroll offset returns the user to the top of a fast list, which is the complaint they made.

    Wrong answer: "Use cursor pagination." Reveals the right mechanism named without the ordering argument. Cursors over an unstable sort key, for example a rank score that changes between requests, reproduce exactly the duplicates you set out to remove.

    Compare your process: did you separate the two symptoms before proposing anything?

---

## 12. Mastery check and remediation map

Seven short-answer items, one for each learning objective in Section 4, in the same order. Objective 1 is tested by M1 and by nothing else, and M1 tests objective 1 and nothing else.

Write or say your answer before opening the block. Producing an answer is what builds the retrieval path. Recognising a correct one does not.

**M1** (Objective 1, classify the round in sixty seconds). Classify each of these three prompts as application-scale or component-scale, naming the two signals in the wording that decided it. For the one that is genuinely ambiguous, give the single clarifying question that settles it.

- (a) "Design a typeahead for our site search."
- (b) "Design the checkout flow for our storefront."
- (c) "Design the search experience for our storefront."

??? note "Show answer"

    (a) Component-scale. Signal one: the noun is a control, not a surface. Signal two: there is no user job in the sentence, only a widget, which means the interviewer is expecting an interface rather than a screen.

    (b) Application-scale. Signal one: "flow" names a journey across more than one screen, which brings routing, cross-screen state and navigation with it. Signal two: it names a business outcome, and outcomes belong to surfaces, not to controls.

    (c) Genuinely ambiguous. "Experience" can mean the results page or the search box, and the two answers share almost nothing. The question that settles it, asked once and in one sentence: "Do you want the whole surface, or do you want me to design this as a reusable component with a public API?"

    The deeper signal, worth stating if you have it: ask who the callers are. If the answer names other teams as consumers, it is component-scale no matter how the prompt was phrased, because a public interface and a versioning story are now part of the answer.

    The failure this item exists to catch is silent classification. Deciding privately and being wrong costs the whole round; deciding out loud and being corrected costs eight seconds.

**M2** (Objective 2, rendering strategy). A product detail page. Content must be visible within 2,000 ms at p75 on a mid-tier phone over an effective 1.6 Mbps connection. The page must be indexable by search crawlers. Price and stock vary by one of 12 delivery regions. The catalogue is 400,000 products and prices change a few times a day. Choose a strategy, name the number that decides it, and state what the rejected options would have cost.

??? note "Show answer"

    The two deciding quantities are the first-render budget against critical-path bytes, and the cardinality of the personalisation. Everything else is preference.

    Pure client rendering is eliminated by the budget. Using the 200 KB per second and 8.3 ms per compressed KB figures derived in Section 10, a 900 KB bundle is roughly 7,900 ms to first content, against a 2,000 ms budget, and it hands a crawler an empty document.

    Full static generation of every product page is eliminated by build time. At an assumed 40 ms to render and write one page, 400,000 pages is 16,000 seconds, so about 4.4 hours per full build. A price that changes a few times a day cannot wait 4.4 hours, so this option makes the freshness requirement unmeetable.

    What survives: a server-rendered, streamed shell that is cacheable at the edge, with the region-varying price and stock as a separately cached fragment. The cardinality is the reason it works: 12 regions times one product is 12 cache variants, which is a cache that still hits. Per-user pricing would have a cardinality of the user base, which is a cache that never hits, and then the price must arrive after the shell rather than inside it.

    The number to say out loud: personalisation cardinality per URL. Under roughly a dozen variants you cache the whole response. Above that you cache the shell and stream or fetch the varying part, and you accept that the price arrives in a second paint.

    What the chosen option costs, said before being asked: per-request server CPU that you now have to size and pay for, a cache key you have to get exactly right or you will serve one region's prices to another, and a streaming response that is harder to debug than a static file.

**M3** (Objective 3, derive the delivery budget). Your p75 user is on a mid-tier phone over an effective 2.5 Mbps connection with a 250 ms round trip. Give the JavaScript bytes allowed on the critical path, a first-render target and an interaction-response target, showing the arithmetic that converts bytes into milliseconds.

??? note "Show answer"

    Bandwidth into usable units: 2.5 Mbps is 2,500,000 bits per second, divided by 8 equals 312,500 bytes per second, so about 312 KB per second. One compressed KB therefore costs 1,000 divided by 312, which is 3.2 ms of transfer.

    Execution: assume a compression ratio of 3.3 to 1 for minified JavaScript and 1 ms of main-thread time per uncompressed KB on a mid-tier phone. One compressed KB is 3.3 uncompressed KB, so 3.3 ms of main thread.

    Combined price of a compressed KB on this profile: 3.2 plus 3.3 equals 6.5 ms. That single figure is the useful output, because it prices every future dependency automatically.

    First-render target: 2,000 ms. Document round trip is 250 ms plus an assumed 100 ms of server think, so 350 ms. That leaves 1,650 ms, and 1,650 divided by 6.5 equals about 254 KB compressed on the critical path.

    Interaction-response target: 200 ms from input to the next painted frame. The derivation, so it is not a number from a poster: below roughly 100 ms a response reads as instantaneous and needs no affordance at all; past 200 ms the interface must show a visible pending state or the user will click again. So 200 ms is the last point at which the simple design is still correct, which makes it the right thing to budget against.

    Two sentences to add, because they are what make the budget real. First, the budget is enforced in continuous integration on compressed bytes per route, not reviewed by hand. Second, 254 KB is the critical path only; lazily loaded routes are outside it, which is precisely why the split boundary has to be drawn deliberately rather than discovered.

**M4** (Objective 4, the public interface of a component). Specify the public interface of a multi-select combobox, to the point where a second engineer could build it without asking you a question about behaviour. Cover the controlled decision, the state machine and the keyboard and focus contract.

??? note "Show answer"

    Ownership, stated first because it settles most of the rest. The caller owns the option data, any asynchronous loading, and the sort order. The component owns the open state, the active option index, the filter applied to the supplied options, and every keyboard behaviour.

    Controlled decision: expose both modes for both pieces of state, independently.

    | State | Uncontrolled | Controlled | Change event |
    |---|---|---|---|
    | Selected values | `defaultValue` | `value` | `onValueChange(next)` |
    | Open state | `defaultOpen` | `open` | `onOpenChange(next)` |

    The rule behind the table: a component that is only controlled forces every caller to write reducer boilerplate for a dropdown. A component that is only uncontrolled cannot be driven from a URL or restored from history state. Ship both, and never make one of the pair conditional on the other.

    Remaining interface: `options`, `getOptionLabel`, `filter`, `maxSelected`, `disabled`, `invalid`, `id`, `labelledBy`, `describedBy`, a `renderOption` slot, and an imperative handle exposing `focus()`, `open()` and `close()`. The accessible name is a required input rather than an optional one, because a required input is testable and a guideline is not.

    State machine, six states: `closed`, `open-idle`, `open-filtering`, `open-loading`, `open-empty`, `open-error`. Transitions: typing in the input moves `closed` or `open-idle` to `open-filtering`; a resolved fetch moves `open-loading` to `open-idle` or `open-empty`; Escape and outside pointer-down move any open state to `closed`; selection stays open because the control is multi-select.

    Keyboard contract, complete enough to implement from:

    | Key | Behaviour |
    |---|---|
    | Down | Opens if closed and makes the first option active; otherwise moves active down, stopping at the last |
    | Up | Opens if closed and makes the last option active; otherwise moves active up |
    | Home and End | First and last option, without wrapping |
    | Printable characters | Typed into the input and treated as a filter, never as type-ahead jumping |
    | Enter | Toggles the active option and leaves the list open |
    | Escape | Closes and returns focus to the input, leaving the current selection intact |
    | Tab | Closes, commits, and moves focus onward normally |
    | Backspace on an empty input | Removes the last selected chip |
    | Delete on a focused chip | Removes it and moves focus to the next chip, or to the input if it was the last |

    Focus contract: DOM focus never leaves the text input. The active option is expressed as an active-descendant relationship rather than by moving focus, which is what lets typing keep working while the list is open. Focus is never allowed to land on the document body, which is the single most common bug in hand-built versions of this control.

    Announcement contract: the selected chips live in a polite live region announcing the count, and the filtered result count is announced on a debounce so that it does not interrupt every keystroke.

    Evolution rule, offered unprompted because it is rarely asked for and it is what separates a component from a library: adding an input is a minor version, changing a default or a keyboard behaviour is a major one, and every deprecated input keeps working for two majors with a console warning.

**M5** (Objective 5, split the state). An order detail screen shows an order header, its line items, a live shipment position, a note field the user is editing, and a theme toggle. Sort the state into client, server-cache and shared, and for every server-cache entry give a cache key, a staleness tolerance in seconds and the event that invalidates it.

??? note "Show answer"

    Client and UI state, which dies with the screen: whether the cancel dialog is open, the active tab, and scroll position. One exception: the note field's draft text is client state that must survive an accidental reload, so it is persisted to session storage under the order id and cleared on successful save.

    Server-cache state:

    | Entry | Cache key | Staleness tolerance | Invalidated by |
    |---|---|---|---|
    | Order header | `order:{orderId}:{viewerId}` | 60 s | A successful cancel or address-change mutation, and a manual refresh |
    | Line items | `order:{orderId}:items` | 300 s | A refund or item-cancel mutation only. Line items do not change after purchase in any other way |
    | Shipment position | `shipment:{shipmentId}:position` | 15 s while the tab is visible, unbounded while hidden | A position message on the live channel, and a refetch on the tab becoming visible again |

    The viewer identity is in the order header key on purpose: the response is permission-shaped, and a cache key that omits the viewer is how one customer sees another customer's order.

    Shared or global state: the authenticated session and its permission set, the locale and time zone, the theme, and the feature flag set. These are read by many screens, change rarely, and have no staleness contract because they are pushed rather than polled.

    The URL owns the order id and the active tab, because a user would reasonably expect to share or reload into either.

    The trap to name out loud: putting server data into the global store. It looks tidy for a week, and it discards per-entry staleness, per-entry invalidation and request deduplication, so the team ends up hand-writing a cache with none of the three. If a value came from the network and can go stale, it belongs in the server-cache category no matter how many screens read it.

**M6** (Objective 6, diagnose an interaction-latency regression). Clicking a filter chip on the listing page went from 120 ms to 700 ms at p75 after a "recommended items" panel was enabled. The panel is rendered by a third-party script that was already loaded on the page before the change. Compressed bundle size is unchanged. Name the cause and the single measurement that confirms it, then give the confirming measurement for each of the other four causes.

??? note "Show answer"

    Cause: a render cascade. The bundle is unchanged, so nothing new is being downloaded or parsed. The third-party script was already present, so it is not newly blocking. What changed is that a new subscriber was added to the state that the filter chip writes, so the click now re-renders a subtree that has nothing to do with filtering.

    The single confirming measurement: record a profile of one click and count which components committed. If components whose inputs did not change are in the commit list, it is a cascade, and the fix is the subscription boundary rather than the panel.

    The five causes and the one measurement that separates each:

    | Cause | The one measurement that confirms it |
    |---|---|
    | Long task | A performance timeline of the interaction showing a single task longer than 50 ms, attributed to a specific script and function |
    | Render cascade | A profile of one interaction listing committed components whose inputs did not change |
    | Unshipped code split | The network panel showing a chunk requested during the interaction, or a coverage report showing a large script executing for the first time inside it |
    | Blocking third party | The same interaction repeated with third-party origins blocked at the network layer, compared like for like |
    | Layout thrash | A recording showing style and layout entries interleaved repeatedly inside one task, which is a read-write loop forcing synchronous layout |

    Why 50 ms is the threshold for a long task rather than an arbitrary choice: at 60 hertz a frame is 16.7 ms, so 50 ms is three frames. Any task that long guarantees dropped frames, and it delays any input arriving during it by up to its own length, which is exactly the quantity an interaction metric measures.

    The delivery point, which is worth as much as the diagnosis: name one hypothesis and the measurement that would kill it, then take the measurement. Proposing three fixes at once tells the interviewer you cannot distinguish between them.

**M7** (Objective 7, mid to staff). Name the four additions that convert a mid-level frontend design answer into a staff-level one on this page, and say what each one changes about a decision that had already been made.

??? note "Show answer"

    - **A performance budget enforced in continuous integration.** Changes the dependency decision. Once the budget is 254 KB and a compressed KB costs 6.5 ms on the p75 profile, adding a charting library is a trade someone has to argue for in a pull request rather than a line in a lock file. Without enforcement, the budget is a wish and the bytes arrive one merge at a time.
    - **A conformance target with its testing method.** Changes the component API. Naming a level, the automated checks that run per commit and the manual keyboard and screen reader pass per release turns the accessible name from an optional input into a required one, because a required input is testable and a guideline is not.
    - **A rollout and kill-switch plan.** Changes the rendering-strategy decision. A strategy you cannot put behind a flag and ramp by cohort has to ship all at once, which raises the evidence bar it must clear before you choose it. The plan needs the flag, the ramp, the field metric, the threshold and the window that trips an automatic revert.
    - **A degradation path.** Changes the dependency graph. Stating what the screen shows when the live channel drops, the CDN serves a stale asset or a third-party script never loads converts hard dependencies into soft ones. It is usually the cheapest availability win available to a client, because it costs a fallback branch rather than infrastructure.

    Each of the four converts a claim into something a person can be held to. That, rather than extra components on the diagram, is what the level difference is made of.

**Threshold.** Five of seven, with M3 and M6 among them. Those two are weighted because a candidate who cannot derive a byte budget or attribute a regression to a single cause has described a frontend rather than designed one, and both gaps are visible to an interviewer within two follow-ups.

| Missed item | Reread | Redo |
|---|---|---|
| M1 | Module 1 on scope and honesty, plus the "Which clock am I on?" block in Section 7 | Take five prompts from the case study list, classify each in under sixty seconds, and write the two signals for each |
| M2 | Module 4 on rendering strategy, and the requirements block of the news feed application case study | Problem B3, then redo M2 with per-user pricing instead of per-region and see which decisions flip |
| M3 | Module 3 on interface requirements and Module 11 on performance | Problems B3 and I3, out loud, naming the eliminated option after every number |
| M4 | Module 6 on component design and Module 7 on accessibility, plus the autocomplete component case study | Specify the interface for a date range picker from scratch, then diff it against the combobox answer above |
| M5 | Module 8 on state split three ways, Module 9 on data fetching and caching, and Module 10 on network and real-time | Take the chat application case study and write its full state table, keys and invalidation triggers included |
| M6 | Module 11 on performance and Module 17 on observability | Problem I2, then problem I3, then write the five-cause table from memory |
| M7 | Module 16 on testing and rollout, and Module 18 on level calibration | Answer the news feed case study at mid level, then again at staff level, and diff the two line by line |

!!! warning "Scoring well right now is a weak signal"

    Passing this immediately after reading mostly measures that the page is still in working memory. The signal that predicts interview performance is the delayed check in Section 15, taken a week later with the page closed. If you score seven of seven today and four of seven next Tuesday, the honest reading of your ability is four.

---

## 13. Common mistakes

Technical mistakes cost you a follow-up. Delivery mistakes cost you the round. The last five rows are delivery, they are the ones an interviewer can observe directly rather than infer, and they are the cheapest of all of these to fix.

| The mistake | Why candidates make it | What the interviewer concludes | What to say instead |
|---|---|---|---|
| Naming a framework or library before naming a budget | It is the part that was studied, and it feels like a decision | Chooses tools by familiarity and will discover the constraints after the code is written | "Before I pick anything: the p75 device gives me about 250 KB on the critical path at 6.5 ms per compressed KB, and 200 ms to the next paint on an interaction. Now the options are narrower" |
| Answering an application prompt when the round is a component prompt | Application-scale answers are what most preparation material trains | Did not listen, and will build the wrong thing thoroughly | "Do you want the whole surface, or this as a reusable component with a public API?" asked in the first two minutes |
| Treating accessibility as a checklist item at minute forty | It is presented as compliance rather than as behaviour | Has never owned a widget's keyboard model, and will ship one that keyboard users cannot operate | "The keyboard model is part of the interface, so I will draw it with the state machine: Escape closes and returns focus to the trigger, Enter toggles and stays open" |
| Adding virtualization to every list | It is the component associated with lists | Adds machinery by association rather than by measurement | "200 rows is about 9,800 nodes and roughly 200 ms of first layout. I would not virtualize until the node count starts costing me a frame on scroll, because virtualization costs find-in-page, print and scrollbar geometry" |
| Choosing WebSockets whenever the word "live" appears | Bidirectional is treated as a superset of good | Selects transports by association and has not operated connection state | "The data flows one way here, so server-sent events give me reconnection for free. WebSockets would buy bidirectionality I do not need and connection state I would have to shed and rebalance" |
| Proposing micro-frontends for four teams | Organisational autonomy is a real pain and this is the named cure | Solves a process problem with an architecture, and will pay for it in bytes forever | "Four copies of a 45 KB runtime is 135 KB extra, about 1.1 seconds on every load. What did release coordination actually cost us last quarter?" |
| Debouncing at 100 ms | It is a habitual number | Has not modelled the mechanism, only copied the shape | "Typing gaps are around 180 ms, so a 100 ms debounce fires on every key and only adds latency. 250 ms coalesces; the price is a pending state I have to show" |
| Server rendering offered as a performance fix, with no caveat | It is the well-known answer to a slow first paint | Sees only the upside of a strategy, which is how a team ends up with a visible page that ignores clicks | "It moves first content from 7.9 seconds to 700 ms and does not move time to interactive at all. So it buys a visible screen that is not yet usable, and the byte problem is still there" |
| Sending an analytics event per interaction | Each event looks small on its own | Has not sized the client's outbound path and will find out in production | "30 impressions a session times 20 million sessions is about 6,900 events a second. I batch at 50, send on idle, and flush with a keepalive request on page hide" |
| Designing before clarifying (delivery) | Silence feels like failure and drawing feels like progress | Will build the wrong thing quickly and confidently | "Two questions first, because they change the shape: which format is this, and what device and connection is the p75 user on?" |
| Monologuing through the component tree (delivery) | Reciting a known structure feels safe and fills the silence | Cannot collaborate, and has left the interviewer no opening to steer | "That is the render path. Do you want the state split next, or the offline behaviour?" |
| Refusing to say "I have not done that" (delivery) | Believing that a visible gap is a scoring event | Bluffs, which is expensive on a real team and easy to detect | "I have not shipped a service worker myself. What I know is the cache strategy trade and the update trap, and here is how I would verify it before trusting it" |
| Defending a choice instead of updating it (delivery) | Reading a follow-up as an attack rather than as information | Will not update a design in a review either, which is the expensive version | "You are right that this breaks with per-user pricing. That changes the caching decision, not just the fetch, so let me redo the boundary" |
| Running out of time inside the render discussion (delivery) | Rendering is comfortable, and accessibility and rollout feel like a formality | No sense of the clock, and no evidence they can tell whether the thing works | At minute 33: "I want to leave seven minutes for the accessibility contract and what I would measure first, so I am stopping the rendering discussion here" |

---

## 14. Interview-day checklist

One screen. Print it, or keep it open in a second window. Nothing here is new.

**Minute budget, whichever clock you are on**

| Phase | Application, 45 min | Component, 45 min |
|---|---|---|
| Restate and name the format | 0 to 2 | 0 to 2 |
| Clarify plus budgets | 2 to 8 | 2 to 7 |
| Sizing that eliminates, or the public interface | 8 to 11 | 7 to 14 |
| v0 drawn, or the state machine and keyboard model | 11 to 19 | 14 to 21 |
| Breaking point and v1, or the data path | 19 to 28 | 21 to 29 |
| One deep dive | 28 to 38 | 29 to 37 |
| Accessibility, degradation, first measurement, skips | 38 to 45 | 37 to 45 |

**The first four sentences**

1. "Let me restate it: users do X on this surface, and the part I think is hard is Y."
2. "Before I draw: do you want the whole surface, or this as a reusable component with a public API?"
3. "I will spend about six minutes on requirements and budgets, then draw it, then break it on purpose."
4. "Stop me if you want depth somewhere specific rather than coverage."

**Five clarifying questions that generalise across frontend prompts**

1. Which format is this: the surface, or one reusable component with callers I do not control?
2. What device and connection is the p75 user on, and is there a browser floor I must support?
3. What is the first-render budget and the interaction-response budget, and who set them?
4. What is personalised, and at what cardinality: per region, per cohort or per user?
5. What accessibility conformance level are we held to, and is it tested per commit or per release?

**Whiteboard drawing order**

```
DRAW IN THIS ORDER. WITHHOLD THE REST UNTIL A NUMBER FORCES IT.

   1              2              3              4
 +--------+    +---------+    +---------+    +---------+
 | ENTRY  | -> | ROUTE   | -> | VIEW    | -> | ONE     |
 | doc +  |    | shell   |    | tree    |    | API     |
 | assets |    |         |    |         |    | CALL    |
 +--------+    +---------+    +---------+    +---------+
                    |                             ^
                    | 5, after a staleness number |
                    v                             |
               +---------+                   +---------+
               | STATE   | ----------------> | CACHE   |
               | SPLIT   |                   | + KEY   |
               +---------+                   +---------+

 NOT YET: service worker, web worker, real-time channel,
 micro-frontend split, design-system layer, offline queue,
 second data source, CDN topology.
 Caption: four boxes left to right first, then the state split
 and its cache key, then nothing else until a stated number
 makes it necessary.
```

Draw the delivery path left to right first, because it is what the prompt asked about. Add the state split underneath only when you reach data, and draw the cache as a key with a staleness figure rather than as an unlabelled box. Erasing a box you drew in minute three reads as guessing; adding one in minute twenty after a number reads as design.

**Three recovery scripts**

| Situation | Say this |
|---|---|
| You are stuck | "Let me say what I am optimising for and give the two options I can see. A trades first paint for interactivity, B the reverse. I will take A because the budget is 2,000 ms to visible content, and I will flag it as reversible." |
| The interviewer disagrees | "That is a case I did not price. If pricing is per user, my cache key has a cardinality of the user base and the fragment stops hitting, so I would move the price out of the cached response entirely. Does that address it?" |
| You are running out of time | "I have about six minutes. I would rather spend them on the accessibility contract and what I would measure in week one than on finishing this dive, because I think that is the higher-value part. Say if you would prefer otherwise." |

**Before you finish**

- State the two trade-offs you actually made, and what each one cost.
- State what you deliberately skipped, one sentence each.
- Name the single field metric you would watch in week one, and the value that would trip a revert.
- Name the degradation path: what the user sees when the live channel, the third-party script or the fresh asset does not arrive.
- Name the decision that is most expensive to reverse. It is usually the rendering boundary or the component's public interface, not the library.

---

## 15. Study schedule and spaced review

### 15a. Three plans, each defined by what it drops

Day 1 is whatever day you start. The skip column is the hard part to write and the useful part to read.

**One week (about 9 to 12 working hours)**

| Day | Do | Skip |
|---|---|---|
| 1 | Modules 1 to 3, then problems B1 and B3 | Nothing yet |
| 2 | Modules 4 and 5, then the news feed application case study end to end | Its second deep dive, unless you finish early |
| 3 | Modules 6 and 7, then the autocomplete component case study, then problem B4 | Module 12 on design systems. It is the least likely to be examined and the easiest to read later |
| 4 | Modules 8 and 9, then the data table or grid case study, then problem B2 | Module 15 on internationalisation, unless your target product ships right-to-left locales |
| 5 | Modules 10 and 11, then problems B5 and I4 | Module 13 on micro-frontends, apart from its refusal argument, which is problem I1 |
| 6 | Modules 14, 16 and 17, then the chat application case study, then problem I2 with a timer | The image carousel, rich text editor, video player and collaborative document case studies |
| 7 | Module 18, then Section 12, then Section 14 | Everything unreached. Do not start new material the day before a round |

**One weekend (about 5 to 7 working hours)**

- Saturday morning: Modules 1, 3 and 4. Format classification, interface budgets and rendering strategy touch every prompt on this page and every prompt you will be given.
- Saturday afternoon: one case study end to end, out loud, with a timer. Take the autocomplete component if your target companies run the component format, the news feed application otherwise.
- Sunday morning: Modules 6, 7 and 8, then problems B4 and I6.
- Sunday afternoon: Sections 12, 13 and 14. Rehearse the first four sentences until they run without attention.
- Skip entirely: Modules 5, 9, 12, 13, 15 and 17, every second deep dive, and seven of the eight case studies. One case study done properly beats five skimmed, and the difference is visible within two minutes of a real round.

**One evening (about 2 to 3 hours)**

- 0:00 to 0:25, Section 14 and Module 1. Get the clock and the format question.
- 0:25 to 1:10, Module 3 and Module 4. Budgets and rendering strategy are the fastest visible upgrade to a frontend answer, because they put numbers into a conversation that usually has none.
- 1:10 to 2:00, one case study, requirements and v0 only, no deep dives.
- 2:00 to 2:30, Section 13 once, then Section 14 again.
- Skip: every other module, every deep dive, the whole mastery check, and all interleaved problems except I1. If the round is tomorrow, extra breadth will not be retrievable under pressure and rehearsed delivery will be.

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

One fact or one decision per card. Copy this block into whatever tool you use. The format is prompt, two colons, answer.

```
Q :: Convert a compressed KB into milliseconds on a stated profile.
A :: Transfer = 1000 / (bandwidth in KB per second). Execution =
     3.3 ms at 3.3:1 compression and 1 ms per uncompressed KB.

Q :: What is the first question of any frontend design round?
A :: Whole surface, or one reusable component with a public API.
     Eight seconds, and it redirects the next forty minutes.

Q :: Why does server rendering not fix time to interactive?
A :: The same JavaScript still transfers and still runs. It moves
     first content only, so you get a visible screen that is dead.

Q :: What decides whether a personalised response is cacheable?
A :: Cardinality per URL. About a dozen variants still hits; per
     user never hits, so the varying part leaves the cached body.

Q :: When is a debounce interval pure loss?
A :: When it is shorter than the inter-keystroke gap, around
     180 ms. It fires on every key and only adds latency.

Q :: Why is 50 ms the long task threshold?
A :: Three frames at 60 hertz. It guarantees dropped frames and
     delays any input arriving inside it by up to its own length.

Q :: Where does overscan come from in a virtualized list?
A :: Scroll velocity. Overscan rows times row height divided by
     px per second is the runway in ms; cover one window render.

Q :: What does virtualization take away?
A :: Find-in-page, native scrollbar geometry, print output, and a
     correct row count for assistive technology.

Q :: Two independent fixes for a flooded real-time client?
A :: Throttle the sender to a perceptual rate, and coalesce the
     receiver to one render per frame. They multiply.

Q :: Why must presence state not share a store with document state?
A :: Every cursor message would invalidate every document
     subscriber, which rebuilds render-per-message cost.

Q :: What must a server-cache entry always carry?
A :: A key including anything that varies the response, a
     staleness tolerance in seconds, and an invalidating event.

Q :: Why does offset pagination duplicate items on a live feed?
A :: An insert above the window shifts the ordering, so the next
     offset starts earlier. Deletes cause invisible skips.

Q :: What does a ranked feed's client owe the ranking team?
A :: Server-provided position per impression, and one written,
     versioned definition of what counts as an impression.

Q :: Four additions that read as staff level here?
A :: A CI-enforced performance budget, a conformance target with
     its test method, a rollout with a kill switch, a degradation
     path.

Q :: The one measurement that confirms a render cascade?
A :: A profile of one interaction listing committed components
     whose inputs did not change.
```

### 15d. Delayed self-check

Answer these from memory a week after you finish, with this page closed. Write the answers down before you check anything.

1. Take a frontend prompt you have not studied here. Classify it as application-scale or component-scale, name the two signals that decided it, and state the budgets you would ask for.
2. Derive its delivery budget from a device and network profile you state, giving the price of one compressed KB in milliseconds, the critical-path byte allowance and the interaction-response target.
3. Describe an interaction-latency regression you would expect that system to suffer, name the cause from the five, and name the single measurement that would confirm it.

Twenty minutes, no notes. If you can do all three, the material is retrievable under pressure. If you cannot, the step where you stalled is the one to redo, not the whole page.

---

## 16. Reference block

!!! info "This section teaches nothing new"

    Everything below appeared earlier with its derivation. This is the version to reread the morning of the round. If a row here is the first place you have met an idea, go and read the module it came from rather than memorising the row.

### 16a. Decision tables

**Rendering strategy**

| If | Choose | Because | Do not choose it when |
|---|---|---|---|
| The surface is behind a login, sessions are long, the audience is on capable devices, and no crawler needs it | Client rendering | Nothing is rendered twice, the server stays a data API, and the interaction model is simple to reason about | The first-render budget is under a few seconds on a slow connection, because the critical-path bytes have to arrive and run before anything is visible |
| Content must be visible fast and indexable, and it varies per request at low cardinality | Server rendering, streamed | First bytes of markup can leave before the slowest data resolves, so visible content is decoupled from the slowest query | Per-request CPU is the constraint you cannot fund, or the page is fully static and you are paying to render the same thing repeatedly |
| The content changes rarely and is identical for everyone | Static generation | The cheapest possible serve, cacheable everywhere, no origin on the request path | The page count times per-page build time exceeds your acceptable freshness window. At an assumed 40 ms per page, 400,000 pages is 4.4 hours |
| Mostly static, with a long tail and occasional updates | Incremental generation with on-demand revalidation | You pay build cost only for pages people ask for, and freshness becomes a revalidation trigger rather than a full build | You cannot name the event that invalidates a page, in which case you have chosen a cache with no invalidation |
| The page is mostly text with a few interactive regions | Islands or partial hydration | Only the interactive regions ship and run JavaScript, so the byte budget follows the interactivity rather than the page | The page is interactive nearly everywhere, where island boundaries add complexity for a saving you cannot measure |
| First paint matters and the slowest data is not needed for it | Streaming with suspense boundaries | The shell paints while the slow region resolves behind a placeholder | The placeholder causes a layout shift, or the slow region is the thing the user came for, in which case you are paying complexity to show them a spinner sooner |

**Which state category does this value belong to**

| If the value | Category | What it must carry | Do not put it here when |
|---|---|---|---|
| Dies with the screen and nobody else reads it | Client and UI state | Nothing, except a persistence rule if losing it on reload would anger a user | It came from the network and can go stale |
| Came from the network and can be refetched | Server-cache state | A key covering everything that varies the response, a staleness tolerance in seconds, and the event that invalidates it | It is the source of truth for something the user is editing, which is client state until it is saved |
| Is read by many screens and changes rarely | Shared or global state | An owner, and a rule for how it is refreshed on session change | It can go stale, because the global store gives you no staleness or invalidation and you will hand-roll both |
| A user would reasonably share or reload into | The URL | A parse and serialise pair, and a default for every absent parameter | It is large, private, or changes many times a second |

**Transport for live updates**

| If | Choose | The number that decides it | Do not choose it when |
|---|---|---|---|
| Updates flow server to client only and are text | Server-sent events | Direction of flow. Reconnection and event identifiers come free, over ordinary HTTP | You need to send frequently from the client, where you would be running two transports |
| Both directions, frequent, low latency, and you can operate sticky connections | WebSockets | Messages per connection per minute, and concurrent connections times per-connection memory. At an assumed 30 KB per connection, 200,000 connections is 6 GB of fleet memory | The client never sends, in which case you have bought connection state you must shed and rebalance for nothing |
| Updates are rare and the path must stay plain HTTP through every proxy | Long polling | Messages per connection per minute. Below roughly one per minute the request overhead is irrelevant | Message rate is high, where you burn one full request path per update |
| The user must be told about something while the tab is closed | Push, outside the page entirely | Whether the value of the message survives the session | The message is only meaningful while the user is looking at the screen |

**Browser storage tiers**

| Tier | Typical use | What it costs | When not to use it |
|---|---|---|---|
| In-memory | Anything within one page lifetime | Nothing, and it is gone on reload | The user would be annoyed to lose it, for example a half-written form |
| Session storage | Per-tab drafts and scroll restoration | Small quota, synchronous access on the main thread | Anything larger than a few tens of KB, because the access blocks |
| Local storage | Small durable preferences | Synchronous and main-thread blocking, string only, no eviction policy you control | Anything on the critical path, because a large read stalls the first frame |
| Indexed database | Offline outboxes, cached documents, large structured data | Asynchronous, more code, and a schema you now version | The data fits comfortably in a few KB, where the machinery is not worth writing |
| Service worker cache | Assets and full responses for offline and repeat visits | An update lifecycle that can strand users on old code | You cannot name your update and activation strategy, because a stale service worker is the hardest client bug to recall |

**Caching directives by asset class**

| Asset | Directive | Why | The rule that makes it safe |
|---|---|---|---|
| Content-hashed script or style, for example `app.a1b2c3.js` | `max-age=31536000, immutable` | 31,536,000 seconds is 365 times 86,400, so a year of free repeat visits | The filename changes whenever the bytes change, and nothing ever serves different bytes under the same name |
| HTML document | `no-cache` | The browser revalidates, so a deploy reaches users on their next navigation | The revalidation round trip is on the critical path, so it must be cheap and it must be at the edge |
| User-specific API response | `private, max-age` in seconds, or no store at all | Keeps a shared cache from serving one user's data to another | Any cache key that omits the varying dimension is a data leak, not a performance bug |
| Region-varying page fragment | Edge cache keyed on the region | 12 regions is 12 variants per URL, which still hits | Cardinality per URL stays small. Per-user cardinality never hits and the varying part must leave the cached body |
| Image or font | Long max-age with a hashed or versioned URL | These are large and change rarely, so repeat visits should never pay for them | Fonts additionally need a display strategy, or text is invisible while they load |

**Rendering a large collection**

| If | Choose | The number that decides it | Do not choose it when |
|---|---|---|---|
| Total nodes are a few thousand and the list is not scrolled continuously | Render everything | Rows times nodes per row times an assumed 0.02 ms of style and layout per node. 200 rows at 49 nodes is about 196 ms once | Never. This is the default and it is correct far more often than it is used |
| Rows times nodes per row costs more than a frame on every scroll | Row windowing | Mounted rows equals visible rows plus overscan; overscan is derived from scroll velocity in milliseconds of runway | You cannot rebuild find-in-page, print, scrollbar geometry and an announced row count, all of which windowing removes |
| Row windowing is in place and a row still holds hundreds of nodes | Column windowing as well | Nodes per row times mounted rows. At 60 columns a 33-row window is 11,913 nodes and about 238 ms | Column count is small, where horizontal windowing adds complexity for nothing |
| The collection is a chart with tens of thousands of marks | Draw to a canvas, and downsample server-side to the pixel width | One node against one node per mark. 50,000 marks as elements is 50,000 nodes | You need per-mark accessibility or per-mark DOM interaction, which a canvas does not give you for free |

### 16b. The numbers this course uses, and where each comes from

Inputs marked as assumed are order-of-magnitude figures chosen for the scenario as stated. Every derived line stays correct as a method when you substitute the interviewer's number or your own measurement.

| Number | Derivation | What it eliminates |
|---|---|---|
| 200 KB per second | 1.6 Mbps is 1,600,000 bits, divided by 8 | Nothing alone. It is the input that prices every byte below |
| 312 KB per second | 2.5 Mbps is 2,500,000 bits, divided by 8 | Nothing alone, same reason |
| 3.3 to 1 compression for JavaScript | Assumption, the order of magnitude modern text compression reaches on minified code | Reasoning about transfer bytes as if they were execution bytes |
| 1 ms per uncompressed KB of execution | Assumption for a mid-tier phone covering parse, compile and execute. Replace with a measurement from your own bundle | Treating a bundle as a download cost only |
| 8.3 ms per compressed KB at 200 KB per second | 5 ms of transfer (1,000 divided by 200) plus 3.3 ms of execution (3.3 uncompressed KB at 1 ms each) | Adding a dependency without a price |
| 6.5 ms per compressed KB at 312 KB per second | 3.2 ms of transfer (1,000 divided by 312) plus 3.3 ms of execution | The same, on a faster profile |
| 7,870 ms to first content today | 400 ms document round trip, plus 900 divided by 200 equals 4,500 ms transfer, plus 2,970 uncompressed KB at 1 ms | A client-rendered 900 KB bundle against any budget under about 8 seconds |
| about 250 KB on the critical path | (2,500 minus 400) divided by 8.3 | Every plan that treats "optimise later" as compatible with a 2,500 ms budget |
| about 254 KB on the critical path | (2,000 minus 350) divided by 6.5 | The same budget on the faster profile, so the two are not far apart and neither allows 900 KB |
| 16.7 ms per frame at 60 hertz | 1,000 divided by 60 | Reasoning about smoothness without a frame budget |
| 50 ms long task threshold | Three frames at 60 hertz. A task this long guarantees dropped frames and delays input arriving inside it by up to its own length | Calling a 40 ms handler the cause of a 700 ms interaction |
| 200 ms interaction budget | The last point at which a response still needs no pending affordance: under about 100 ms it reads as instant, past 200 ms the interface owes the user a visible pending state | Designing an interaction with no committed target |
| 1,960,000 nodes | 40,000 rows times 49 nodes, where 49 is 8 columns at an assumed 6 nodes per cell plus one row element | Rendering a 40,000 row grid into the DOM |
| 2.35 GB of retained DOM | 1.96e6 nodes at an assumed 1.2 KB per node | The claim that a large grid is merely slow rather than impossible |
| 39,200 ms of first layout | 1.96e6 nodes at an assumed 0.02 ms per node | Any answer that does not window the rows |
| 1,617 nodes in the window | 33 mounted rows times 49 nodes, where 33 is 17 visible plus 8 overscan each way | Mounting more than you can lay out inside a frame or two |
| 8 rows of overscan | 8 times 48 px is 384 px, which at an assumed 2,000 px per second fling is 192 ms of runway, enough for one 32 ms window render and several dropped frames | Overscan chosen by anxiety rather than by scroll velocity |
| 11,913 nodes at 60 columns | 33 rows times 361 nodes, so about 238 ms of layout | Row windowing alone once a row carries hundreds of nodes |
| 180 ms between keystrokes | Assumption for a competent typist. Measure it in your own product from input event timestamps | A debounce interval chosen without reference to typing speed |
| 194 requests per second | 1.2e6 searches times 14 characters, divided by 86,400 | One request per keystroke |
| 28 requests per second | 1.2e6 searches times an assumed 2 pauses, divided by 86,400 | Nothing on its own. It is the payoff line for the 250 ms debounce |
| 2,880 messages per second | 24 other participants times 120 pointer samples | Nothing yet. It is the input to the two lines below |
| 432 ms per second of parsing | 2,880 messages at an assumed 0.15 ms each | An untuned receive path, which spends 43 percent of the main thread ingesting cursors |
| 1,152 ms per second of rendering | 2,880 messages at an assumed 0.4 ms each | Rendering per message, which cannot keep up with wall clock |
| 96 ms per second when coalesced | 60 frames times (0.4 ms fixed plus 24 cursors at 0.05 ms) | The belief that a faster transport fixes a render problem |
| 135 KB of duplicated runtime | 4 independently deployed apps at an assumed 45 KB compressed each, minus one shared copy | Micro-frontends for four teams, since at 8.3 ms per KB that is about 1,120 ms on every load |
| 6 GB of connection state | 200,000 concurrent connections at an assumed 30 KB each | Running 200,000 connections without sizing the fleet |
| about 6,940 impression events per second | 20 million sessions times 30 impressions, divided by 86,400 | One request per impression event |
| 4.4 hours for a full static build | 400,000 pages at an assumed 40 ms to render and write | Full static generation for a catalogue whose prices change daily |
| 400 upload chunks | 2 GB divided by 5 MB | A single multipart upload, since at an assumed 0.5 percent chunk failure rate that is about 2 expected failures per file |
| 480 ms of formatter construction | 200 rows times 3 dates times an assumed 0.8 ms per locale-aware formatter construction | Attributing a 340 ms interaction regression to a 50 KB bundle increase |
| 165 ms of extra parse | 50 KB compressed times 3.3, at 1 ms per uncompressed KB | The same, from the other direction: parse happens at startup and cannot move an interaction metric |
| about 2,160 ms of third-party cost | 260 KB compressed at 8.3 ms per KB | A continuous integration score taken as evidence about real users, when the job does not load the tags |

### 16c. ASCII glyph legend

```
ASCII CONVENTIONS USED IN EVERY DIAGRAM ON THIS PAGE

 +--------+   solid box: something that runs inside the tab on
 | NAME   |   the main thread, and that a user can be blocked by
 +--------+

 +========+   double edge: something that survives a reload
 | NAME   |   (HTTP cache, service worker cache, storage tier)
 +========+

   --->       a request or a render pass a user is waiting on
   ==>        deferred or background work, nobody is waiting
   <-->       two copies of one definition that must agree
   |   v      vertical continuation of the same path

 Caption: box style says whether the thing survives a reload;
 arrow style says whether a user is waiting on it.
```

### 16d. One-page summary cards

Twelve cards. Cards 1 to 8 are the eight case studies above, in the order they appear. Cards 9 to 12 are four further prompts that do not carry a full arc on this page, kept here in compressed form because they are common enough to be worth rehearsing.

**Card 1. News feed application**

| Field | Content |
|---|---|
| Prompt | Design the web client for an infinitely scrolling social feed |
| Format | Application-scale |
| Budgets | 2,000 ms to first content at p75, 200 ms interaction response, about 250 KB on the critical path |
| Number that decides | Retained nodes, not rows on screen. Scroll cost rises with pages loaded rather than with the window, which is the tell |
| V0 | Server-rendered first page, cursor pagination, a plain DOM list, no windowing |
| Breaking point | After several pages the retained DOM dominates; observe as scroll frame time plotted against pages loaded |
| V1 | Row windowing with recycled rows, plus a bounded page cache keyed by cursor and an insert buffer for new items |
| Deep dive one | Scroll restoration and cursor stability under concurrent inserts |
| Deep dive two | Optimistic interactions with rollback, and the impression logging contract owed to the ranking team |
| Failure class | Unbounded retained DOM, the client analogue of unbounded queue growth in the [failure library](index.md#the-failure-library) |
| Staff delta | Byte budget enforced in CI, a versioned impression definition agreed with the ranking team, and a degradation path to the last cached page when the feed API is unavailable |

**Card 2. Autocomplete component**

| Field | Content |
|---|---|
| Prompt | Design a typeahead for site search, as a reusable component |
| Format | Component-scale |
| Budgets | Visible response within one frame of a keystroke, network suggestions within 400 ms of a pause |
| Number that decides | The inter-keystroke interval, an assumed 180 ms, against the debounce interval. Below it the debounce coalesces nothing |
| V0 | Uncontrolled input, one request per keystroke, results rendered on arrival |
| Breaking point | 14 requests per query and out-of-order responses overwriting newer results; observe as list flicker and as requests per session |
| V1 | 250 ms debounce, in-flight cancellation, a local prefix cache, and an active-descendant keyboard model |
| Deep dive one | The full keyboard and focus contract, including where focus goes on Escape |
| Deep dive two | Request race resolution: cancellation plus a check that the response's query still matches the input |
| Failure class | Last-write-wins over out-of-order responses, which debouncing reduces and never prevents |
| Staff delta | A stated versioning rule for the public interface, the accessible name as a required input, and a keyboard regression test that fails the build |

**Card 3. Data table or grid**

| Field | Content |
|---|---|
| Prompt | Design a data grid with sorting, filtering, selection and export |
| Format | Component-scale, with an application-scale variant when the data path is in scope |
| Budgets | One frame per scroll tick, 200 ms per sort or filter interaction |
| Number that decides | Nodes per row times mounted rows, at an assumed 0.02 ms of style and layout per node |
| V0 | A plain table element, all rows rendered, sorting and filtering in memory |
| Breaking point | 40,000 rows is 1.96 million nodes, about 2.35 GB and 39 seconds of first layout, so the naive version does not run |
| V1 | Row windowing with derived overscan, sorting and filtering moved to the server, and selection stored as a set of identifiers |
| Deep dive one | Selection semantics across pages and under filter changes, including what "select all" means |
| Deep dive two | The column windowing threshold, and keyboard navigation in a virtualized grid |
| Failure class | Quadratic re-render caused by storing a selected flag on every row instead of a set of identifiers |
| Staff delta | An export path that streams rather than loading everything into memory, a written accessibility strategy for a virtualized grid, and a scroll frame-time budget in CI |

**Card 4. Image carousel and modal dialog**

| Field | Content |
|---|---|
| Prompt | Design a carousel and the modal it opens, as design-system components |
| Format | Component-scale |
| Budgets | No capacity budget applies. The complexity measure is the transition count of the state machine and the number of focus destinations |
| Number that decides | Focus destinations, not bytes. Every state must name where focus is, and there are no exceptions |
| V0 | Absolutely positioned slides with next and previous buttons, and a dialog rendered in place |
| Breaking point | Keyboard focus escapes behind the open dialog and the background scrolls; observe it with a keyboard-only pass, which no automated check will do for you |
| V1 | A focus trap with restoration to the trigger, the background made inert, and a slide machine that pauses on hover, on focus and under reduced motion |
| Deep dive one | Focus trap, restoration, and why making the background inert differs from hiding it from assistive technology |
| Deep dive two | Pointer and gesture handling that does not fight the page's own scrolling |
| Failure class | Focus lost to the document body, plus a state machine with a state that has no exit |
| Staff delta | Reduced-motion behaviour specified rather than added later, a documented trigger contract for callers, and an automated focus-order assertion in CI |

**Card 5. Rich text editor**

| Field | Content |
|---|---|
| Prompt | Design a rich text editor for long-form content |
| Format | Component-scale, at the largest end of it |
| Budgets | One frame per keystroke at a stated document size, and a stated paste-handling latency |
| Number that decides | Document node count against the cost of re-rendering the document per keystroke, which is what forces a model separate from the DOM |
| V0 | A contenteditable region, with the rendered markup read back as the source of truth |
| Breaking point | A paste from a word processor injects arbitrary markup that the DOM now owns; observe as a growing list of unreproducible formatting bugs |
| V1 | An explicit document model, a command layer that mutates only the model, and rendering derived from the model |
| Deep dive one | Selection and cursor mapping between the model and the DOM, in both directions |
| Deep dive two | Undo granularity, and paste sanitisation treated as a security boundary rather than a cleanup step |
| Failure class | Cross-site scripting through pasted content, and silent divergence between the model and the DOM |
| Staff delta | A sanitiser allowlist with a named owner, a stated decision to defer collaborative editing with the reason, and fuzz tests on the paste path |

**Card 6. Collaborative document**

| Field | Content |
|---|---|
| Prompt | Design the client for a document several people edit at once |
| Format | Application-scale |
| Budgets | Local edits visible within one frame, remote edits within a stated few hundred milliseconds, and a stated main-thread ceiling for presence |
| Number that decides | Participants times send rate against the frame rate. 24 peers at 120 samples is 2,880 messages a second, which is more work than a second contains |
| V0 | Whole-document save with last-write-wins, plus an advisory lock |
| Breaking point | Two people typing lose each other's edits; observe as save conflicts per session and as complaints about disappearing text |
| V1 | Operation-based edits over a persistent channel, presence on a separate channel with its own store, and an outbox for edits made offline |
| Deep dive one | Presence coalescing and interpolation, and why presence cannot share a store with the document |
| Deep dive two | The offline outbox: ordering, deduplication, and reconciliation on reconnect |
| Failure class | Thundering herd on reconnect after a deploy, straight from the [failure library](index.md#the-failure-library) |
| Staff delta | A stated convergence guarantee rather than a library name, a kill switch back to single-writer mode, and an explicit and agreed data-loss budget |

**Card 7. Video player**

| Field | Content |
|---|---|
| Prompt | Design a video player with adaptive quality and captions |
| Format | Component-scale, with a delivery path in scope |
| Budgets | Time to first frame, rebuffer ratio, and a stated startup quality on a slow connection |
| Number that decides | Buffer seconds against measured throughput. A 4 Mbps stream on a 1.6 Mbps connection drains faster than it fills, which is the whole argument for a ladder |
| V0 | A single-bitrate file in a native media element |
| Breaking point | Users on slow connections stall repeatedly; observe as rebuffer ratio sliced by connection type, never as an overall average |
| V1 | A segment ladder with a switching policy, a buffer target, and a startup rung chosen from measured throughput rather than from a default |
| Deep dive one | The switching policy and how it is scored: rebuffer ratio traded against average delivered bitrate |
| Deep dive two | Captions, keyboard controls and the accessible media contract, including what a caption track owes a deaf user beyond dialogue |
| Failure class | Oscillation between rungs, a feedback loop where the policy reacts to a measurement its own switching produced |
| Staff delta | A quality-of-experience metric with a named owner, a degradation path to audio only, and a kill switch on the switching policy |

**Card 8. Chat application**

| Field | Content |
|---|---|
| Prompt | Design the browser client for a chat product |
| Format | Application-scale |
| Budgets | Sent message visible within one frame, remote message within a stated latency, and a stated behaviour with no connection |
| Number that decides | Messages per second per open conversation times the per-message client cost, which decides whether you need coalescing at all |
| V0 | Fetch history on open, poll every 5 seconds |
| Breaking point | Messages arrive up to 5 seconds late and every open tab polls; observe as perceived latency and as requests per second per user |
| V1 | One multiplexed connection with per-conversation subscriptions, an outbox for sends, and idempotency keys on every send |
| Deep dive one | Ordering and deduplication on the client, including why a client clock is never trusted for ordering |
| Deep dive two | The offline send queue: retry policy, jitter, and what the interface shows for a message that has not been acknowledged |
| Failure class | Duplicate sends on retry when the send carries no idempotency key |
| Staff delta | The delivery guarantee made visible in the interface rather than assumed, a jittered reconnect policy, and a documented degradation to polling |

**Card 9. E-commerce product and checkout flow**

| Field | Content |
|---|---|
| Prompt | Design the product page and the checkout flow for a storefront |
| Format | Application-scale |
| Budgets | 2,000 ms to visible content at p75, crawlable markup, and a stated conformance level for the forms |
| Number that decides | Personalisation cardinality per URL. Twelve regions is twelve cache variants and still hits; per-user pricing never hits |
| V0 | Server-rendered pages, no client routing, one form post per checkout step |
| Breaking point | Price and stock vary by region while the page is cached without that dimension in the key; observe as a mismatch rate between displayed and charged price |
| V1 | Cached shell with a region-keyed fragment, client-side validation with the server as authority, and an idempotent payment submission |
| Deep dive one | Cache keys and varying dimensions, and the cardinality that decides whether the varying part stays in the body |
| Deep dive two | Form correctness: error announcement, autofill, and a payment step that survives a double submit |
| Failure class | Serving one region's cached price to another, and a double charge from a retried submission |
| Staff delta | An alert on price mismatch rate, a checkout kill switch back to a plain server-rendered form, and a conformance target checked per commit |

**Card 10. Analytics dashboard**

| Field | Content |
|---|---|
| Prompt | Design a dashboard with heavy charts over a large time range |
| Format | Application-scale |
| Budgets | 200 ms per cross-filter interaction, and a stated first-render target for the default range |
| Number that decides | Marks rendered times nodes per mark. 50,000 points as elements is 50,000 nodes; on a canvas it is one |
| V0 | Element-based charts, one node per point, every series fetched on load |
| Breaking point | Interaction latency rises with the selected range rather than with the number of charts; observe by plotting interaction latency against range length |
| V1 | Canvas rendering, server-side downsampling to the pixel width of the chart, and aggregation moved off the main thread |
| Deep dive one | Downsampling honestly: what a min-max decimation preserves, what it hides, and why the rule must be documented for readers of the chart |
| Deep dive two | Cross-filter latency, and deciding per interaction whether the work belongs in a worker or on the server |
| Failure class | Main-thread starvation from synchronous aggregation, which looks like a hung tab rather than a slow one |
| Staff delta | A per-widget interaction budget, a visible stale-data indicator, and a published downsampling rule so nobody misreads a chart |

**Card 11. File uploader**

| Field | Content |
|---|---|
| Prompt | Design an uploader for large files with progress and recovery |
| Format | Component-scale, with a server contract in scope |
| Budgets | A stated maximum re-upload cost after a failure, and honest progress |
| Number that decides | File size divided by chunk size times per-chunk failure probability. 400 chunks at an assumed 0.5 percent is about 2 expected failures per file |
| V0 | A single multipart post of the whole file |
| Breaking point | A 2 GB file at 1 MB per second takes about 33 minutes, and one failure restarts all of it; observe as upload success rate by file size |
| V1 | 5 MB chunks, per-chunk retry with jitter, a resumable session identifier, and progress derived from acknowledged chunks |
| Deep dive one | Resumability: the acknowledgement contract, and what the client must persist to resume after a reload |
| Deep dive two | Progress that does not lie, reporting bytes acknowledged by the server rather than bytes written to a socket |
| Failure class | Retry storm when a cohort of clients retries a failed upload at the same moment, from the [failure library](index.md#the-failure-library) |
| Staff delta | A per-user concurrency cap, a resumable session lifetime agreed with the server team rather than assumed, and a degradation to single-part for small files |

**Card 12. Brownfield: an interaction regression after a dependency upgrade**

| Field | Content |
|---|---|
| Prompt | A shipped page got slower to interact with after a dependency upgrade. Diagnose and plan the fix |
| Format | Application-scale, with no v0 to draw |
| Budgets | Restore the previous interaction latency at p75, and prevent a silent recurrence |
| Number that decides | Which phase the cost lands in. 165 ms of extra parse is a startup cost and cannot move an interaction metric, so the two numbers describe different problems |
| V0 (what exists) | Client-rendered checkout, one bundle, continuous integration green, no field alerting on interaction latency |
| Breaking point | Interaction latency at p75 moved from 180 ms to 520 ms with no bundle-size explanation; the profile shows 480 ms of repeated formatter construction |
| V1 | Hoist and memoise the formatter, then add a byte budget and an interaction benchmark to CI, plus a field alert annotated by release |
| Deep dive one | Attribution, from a field percentile to a long task to a specific function, without changing anything first |
| Deep dive two | The CI gap: what a budget must assert to have caught this, and why a green score was compatible with a 340 ms regression |
| Failure class | A silent regression shipped by a pipeline that measured the wrong phase of the page's life |
| Staff delta | A time-boxed revert kept available as a mitigation while the investigation stays open, and a named owner for the third-party byte budget |

---

## 17. What this course does not cover

Naming our limits is cheaper than pretending we have none, and it stops you studying the wrong thing.

| Not covered | Why | Where to go instead |
|---|---|---|
| Frontend coding rounds: implement a debounce, build this from a screenshot, write this component live | A different round with a different rubric. Doing it here would double the page and improve no design answer | Eloquent JavaScript, Marijn Haverbeke, 3rd edition, 2018, readable free at eloquentjavascript.net, plus the [data structures and algorithms question bank](/Interview-Questions/data-structures-algorithms/) on this site |
| CSS layout, visual design, typography and motion craft | Real skills, and not what a design round scores. We treat layout only where it costs main-thread time | The [MDN CSS layout guide](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/CSS_layout), which is free, current and maintained by contributors rather than a vendor |
| Framework-specific APIs, and which framework to pick | Framework APIs move faster than a page can be reviewed, and a design answer should not turn on one | Whichever framework's own current documentation, checked on the day. If an interviewer wants framework depth, that is a different round and you should ask |
| Build tooling configuration: bundler options, module graphs, plugin ecosystems | The design decision is where the split boundary goes, which we teach. The configuration is a manual | The documentation for the tool your team actually uses, checked on the day |
| Native mobile clients: battery, background execution, over-the-air release, version skew | Genuinely different constraints, and enough material for its own arc | Our [Mobile System Design](mobile-system-design.md) course in this section |
| The API contract itself: domain modelling, error taxonomy, idempotency, versioning, webhooks | We consume contracts here and state what we need from them. Designing them is its own round | Our [Product Architecture](product-architecture.md) course in this section |
| Everything behind the network call: services, stores, queues, sharding, consensus, multi-region | Assumed as background rather than taught. A client sits on top of it | Our [Modern System Design](modern-system-design.md) course, and Designing Data-Intensive Applications, Martin Kleppmann, 1st edition, O'Reilly, 2017 |
| The quality of what the screen shows: ranking, retrieval, recommendation, abuse scoring | We teach what the client owes a ranking system and stop there | Our [ML System Design](ml-system-design.md) course in this section |
| Accessibility law, policy and procurement obligations by jurisdiction | Genuinely important and genuinely jurisdiction-specific. Generic advice here would be worse than none | Your own legal function, plus WCAG 2.2, a W3C Recommendation published in 2023, and the [ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/patterns/) for the per-pattern behaviour |
| Assistive technology behaviour in depth: how specific screen readers announce specific markup | It varies by product and version, changes without notice, and unverifiable claims about it are exactly what we refuse to publish | Test with the assistive technology your users actually use, and treat the authoring pattern guide as the specification rather than as a description |
| Network and protocol internals: connection setup, multiplexing, congestion control, prioritisation | We use bandwidth and round trip as inputs. The mechanics underneath are a book | High Performance Browser Networking, Ilya Grigorik, O'Reilly, 2013, readable free at hpbn.co. Note the date: the transport chapters have aged, the measurement discipline has not |
| Graphics-heavy surfaces: canvas engines, WebGL and WebGPU, game loops | Adjacent and large. We touch canvas only where it replaces tens of thousands of DOM nodes | The MDN reference for the specific API, checked for its date, because these are moving quickly |
| WebAssembly, and moving heavy computation off JavaScript | Rarely what a frontend design round asks, and the decision is usually made by a profile rather than by a design | The MDN WebAssembly reference plus your own profile, in that order |
| Browser extensions, Electron and other desktop shells, and HTML email | Different runtimes with different constraints, and none of them is what "frontend system design round" means | The platform's own documentation, and be sceptical of advice written for the browser |
| Specific testing frameworks and their assertion styles | We teach what to test at which layer. Which runner you use does not change that | The documentation for your runner, and Module 16 for the layering argument |
| Behavioural rounds and company-specific loop formats | Loop structures change per team and per quarter, and unverifiable claims about them are exactly what we refuse to publish | Ask your recruiter for the current format, in writing |

!!! note "On anything we cite"

    We name the edition and the year so you can tell what has aged. A 2013 networking book is still the best free explanation of measurement discipline and is out of date on protocol specifics, and saying so is more useful than either recommending it silently or dropping it.

    If a link here is dead, or a newer edition exists, open an issue and it is fixed in the next review pass rather than sitting behind an unverifiable "recently updated" badge.

---

## 18. Provenance

**Last reviewed:** 1 August 2026. A page whose last-reviewed date is more than nine months old is flagged for review rather than quietly left in place.

**Changelog, most recent first**

| Date | Change |
|---|---|
| August 2026 | First publication: eighteen modules, eight case studies with the full arc and level deltas, eleven practice problems with public rubrics, and seven mastery items mapped one to one against the seven learning objectives |
| August 2026 | Component-scale format separated from application-scale throughout after review, including a second delivery clock, because merging the two produced a page that served neither round well |
| August 2026 | Every figure in Sections 10, 12 and 16 re-derived from inputs stated in the same section. Figures that could not be derived were relabelled as assumptions with the sentence that says why each is reasonable, and two that eliminated no design option were deleted |

**Found a problem?** Open an issue at [github.com/datascienceinterviews/datascienceinterviews.github.io/issues](https://github.com/datascienceinterviews/datascienceinterviews.github.io/issues). Corrections, disputed numbers, dead links and better derivations are all in scope. Errors get fixed in days, and the changelog above records what changed.

All content on this page is original. Research informed which topics belong here; no prose, framework, acronym, worked example, table or diagram is derived from any paid course. Primary sources are cited inline with their edition and year, and any claim we could not source is either labelled as an assumption or is not on the page.
