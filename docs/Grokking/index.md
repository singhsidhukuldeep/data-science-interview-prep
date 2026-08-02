---
title: "Grokking Interviews: Free In-Depth Interview Courses"
description: Seven free design interview courses with public rubrics, graded practice, level deltas and full case studies. No account, no email wall, nothing gated.
last_reviewed: 2026-08-02
---

# Grokking Interviews

Seven free paths, one per design round companies actually run. No account, no email capture, no gated lesson.

- [Modern System Design](modern-system-design.md): services, stores, pipelines.
- [ML System Design](ml-system-design.md): ranking, retrieval, prediction, abuse.
- [Generative AI System Design](generative-ai-system-design.md): products on foundation models.
- [Mobile System Design](mobile-system-design.md): the client as a distributed node.
- [Frontend System Design](frontend-system-design.md): browser apps and single components.
- [Product Architecture](product-architecture.md): domain models, contracts, error semantics.
- [Fast-Track](system-design-fast-track.md): an hour-by-hour plan when the round is days away.

One honest limit: this does not replace a human mock interviewer, and we will not pretend otherwise. Instead we ship the apparatus a coach carries: a four-criterion rubric on every open problem, [interviewer script packs](#interviewer-script-packs) a friend with no domain knowledge can read aloud, timed drills against a stated clock, and a level delta per case study showing what mid, senior and staff answers each added.

## Pick your path

| Path | Train for this if | Do not start here if | Time |
|---|---|---|---|
| [Modern System Design](modern-system-design.md) | The prompt is a service, a store or a data pipeline, and you will be pushed on capacity, consistency and failure modes | Your round is scoped to a model, a phone, a browser or an API contract. Take the matching path below | 26 to 34 h |
| [ML System Design](ml-system-design.md) | The prompt names a ranking, retrieval, prediction or abuse problem, and the follow-ups are about labels, metrics and serving | The round is algorithms and math on a whiteboard. This path designs systems around models, not models | 22 to 30 h |
| [Generative AI System Design](generative-ai-system-design.md) | The product sits on a foundation model: retrieval, agents, evaluation, guardrails, tokens and cost | You will be asked to pretrain a foundation model. That is a research round, and module 1 says which parts transfer | 20 to 28 h |
| [Mobile System Design](mobile-system-design.md) | You are hired to write client code, and the round covers offline sync, background work, battery and version skew | The invite says mobile but the interviewer keeps drawing server boxes. Take Modern, and read module 1 here for the steering script | 20 to 26 h |
| [Frontend System Design](frontend-system-design.md) | The round is a browser application or one component, with rendering, accessibility and bundle budgets on the table | You want general distributed systems breadth for a full stack loop. Take Modern first, then this | 20 to 26 h |
| [Product Architecture](product-architecture.md) | The answer is a domain model plus a contract: resources, errors, idempotency, pagination, versioning, webhooks | You expect a product sense round about which feature to build. This trains interface and domain design, and we say so up front | 18 to 24 h |
| [Fast-Track in 48 hours](system-design-fast-track.md) | Your round is this week and you need a plan that names what to drop | You have more than two weeks. Cramming loses to a full path, and module 1 tells you to leave | 26 to 30 working h inside 48 wall-clock h, with 5 to 6 h and 14 to 16 h branches |

!!! note "How these hour ranges were produced"

    Each course sums its own per-module estimates (reading at a slow, engaged rate for dense technical material, plus practice and spaced review), gives the result as a range, and pads it because self-estimates run optimistic. The per-section arithmetic is published in each course's time budget table, so you can recompute it and disagree.

### If nobody told you which round you face

```
+------------------------------------------------+
| Start: what will you be asked to draw?         |
+------------------------------------------------+
   |
   +-> +------------------------------------------------+
   |   | A service, a store or a pipeline = Modern      |
   |   +------------------------------------------------+
   |
   +-> +------------------------------------------------+
   |   | A model that ranks, scores or detects = ML     |
   |   +------------------------------------------------+
   |
   +-> +------------------------------------------------+
   |   | A feature built on an LLM = Generative AI      |
   |   +------------------------------------------------+
   |
   +-> +------------------------------------------------+
   |   | An app running on a phone = Mobile             |
   |   +------------------------------------------------+
   |
   +-> +------------------------------------------------+
   |   | A screen or a widget in a browser = Frontend   |
   |   +------------------------------------------------+
   |
   +-> +------------------------------------------------+
   |   | An API, a schema or a webhook = Product Arch   |
   |   +------------------------------------------------+
   |
   +-> +------------------------------------------------+
       | Still unclear and the round is soon = Fast     |
       | Track, then the path it routes you to          |
       +------------------------------------------------+
```

Caption: a one-question router. The artifact you are asked to draw picks the path; if the artifact is still unclear and the round is imminent, start with Fast-Track, which ends by routing you into a full path.

Two follow-ups worth sending the recruiter before you pick, because both change the answer: which team runs the round, and whether the session is 45 or 60 minutes. Both are ordinary scheduling questions and cost you nothing to ask.

## The naming confusion, explained

"Grokking" is not one product line. The verb comes from Robert A. Heinlein's 1961 novel *Stranger in a Strange Land*, and at least three publishers now use it as a title prefix. Searching the phrase returns near-identical course titles from different companies, and that overlap is what makes the category hard to buy in. Here is the factual picture, with each claim sourced to the publisher's own page, checked 2 August 2026.

| Publisher | What it publishes under the name | Source, checked 2 August 2026 |
|---|---|---|
| Manning Publications | A long-running book series, for example *Grokking Algorithms, Second Edition* by Aditya Y. Bhargava (2024) | [manning.com](https://www.manning.com/books/grokking-algorithms-second-edition) |
| Design Gurus | A broad catalogue of courses prefixed "Grokking", spanning system design, coding patterns, behavioural rounds, databases and AI. Its own catalogue page groups them into eight categories, so the headline count depends on whether you count roadmaps and non-Grokking titles | [designgurus.io/courses](https://www.designgurus.io/courses) |
| Design Gurus, *Grokking the System Design Interview* | The course page states: "DesignGurus.io is the birthplace of the 'Grokking' interview methodology." It also claims authorship of the frameworks it says defined modern system design preparation | [designgurus.io course page](https://www.designgurus.io/course/grokking-the-system-design-interview) |
| Educative | A family of in-house interview courses, including *Grokking Modern System Design Interview* (204 lessons per its own page) and *Grokking the Product Architecture Interview* | [educative.io](https://www.educative.io/courses/grokking-the-system-design-interview) |

The detail that trips people up: the Educative URL ending `grokking-the-system-design-interview` today serves a course whose on-page title is *Grokking Modern System Design Interview*, credited to Educative co-founder Fahim ul Haq. A link saved from a blog post or a Reddit thread years ago therefore does not necessarily open the course that post was describing.

Two counts that circulate in reviews of this category did not survive checking on 2 August 2026, so we do not repeat them. A "more than twenty courses" figure for the Design Gurus catalogue is not something the catalogue page states, and the lesson count often quoted for a Grokking mobile system design course points at an Educative URL that now returns 404. Where we cannot open the page ourselves, we describe the catalogue instead of quoting a number.

None of this is a scandal, and we are not calling anyone dishonest. Both companies are real publishers shipping real courses, and a shared title prefix is a branding outcome, not a defect. It does mean the burden of checking falls on you.

!!! tip "Six things to check before you buy anything, from anyone"

    | Check | Why it decides the purchase |
    |---|---|
    | Which company owns the checkout domain | Two products with near-identical titles are sold by different companies. Read the domain, not the title |
    | Whether the full lesson list is public before payment | If you cannot see every lesson title, you cannot tell whether the topic you need is one lesson or one chapter |
    | What happens on the day your subscription lapses | Subscription access is a rental. Decide whether you need the material after your offer, not before |
    | Whether mocks, rubrics and feedback are in the tier you are buying | Graded mocks are frequently a higher tier or a separate product than the course that advertises them |
    | Whether "recently updated" is backed by a visible changelog | An update badge with no diff is unauditable. Ask what changed and when |
    | The refund window in the terms, and how renewal is triggered | Read the actual terms page rather than the marketing page, and note the renewal date somewhere you will see it |

    We publish this section because it is a high-intent question that no vendor can answer neutrally about itself. It applies to us too: our answer to every row above sits in the provenance footer at the bottom of each course page.

## How these courses are built

Every course page in this section follows the same eighteen-section anatomy. The order is deliberate: qualify the reader, test readiness, state objectives, then teach, then make the reader produce work, then tell them honestly whether they are done.

| Section | Why it exists |
|---|---|
| 1. Header contract | Lets you decide in ten seconds. Includes who the page is *not* for, with a link to the page that does serve you |
| 2. Prerequisites | A testable readiness check, replacing the useless beginner and intermediate labels. Each row has a one-minute quick check |
| 3. Time budget | Reading, practice and review as ranges, computed per module and summed, with the method stated |
| 4. Learning objectives | Observable verb, object, condition, criterion. Written before the explanation, so the page is built backwards from what it claims to teach |
| 5. Warm-up retrieval | Cumulative recall from earlier pages, including one question that reaches back several pages |
| 6. Module map | The shape of the whole before the parts, with a "Skip if" column so experienced readers can route around what they hold already |
| 7. Delivery clock | Minute budgets for 45 and 60 minute rounds, how the split shifts by level, and when following the clock is the wrong move |
| 8. Modules | Each one: a prediction question, small steps, a diagram, a fully worked example, a faded example, a check, and a when-not-to-use |
| 9. Case studies | An identical nine-part arc per prompt, so no prompt is thin because it was less fun to write |
| 10. Practice, two sets | A blocked set matched to the worked examples, then an interleaved set that mixes earlier pages and hides which tool applies |
| 11. Rubrics | Four criteria with named levels, a model answer, and the common wrong answers each diagnosed by the misconception it reveals |
| 12. Mastery check | Short answer, one item per objective, with a remediation table telling you exactly what to reread and redo |
| 13. Mistakes catalogue | What the mistake is, why people make it, what the interviewer concludes, and what to say instead. At least three are about delivery |
| 14. Interview-day checklist | One printable screen for the reader who opens the page an hour before the round |
| 15. Study schedule | Branch plans for one week, one weekend and one evening, each naming what to skip |
| 16. Reference block | Kept structurally separate, because mixing lookup material into teaching prose degrades both |
| 17. What this does not cover | The adjacent topics we omit, why, and where to go instead, including resources we do not control |
| 18. Provenance footer | Last reviewed date, a three-item changelog, and a link to open an issue when we are wrong |

### Five choices you will notice

| Choice | What it looks like on the page | Why |
|---|---|---|
| V0 before V1 | The first diagram in a case study is the fewest boxes the stated requirements permit. You are told what breaks, at what number, and how you would observe it, before a box is added | A finished architecture revealed in one move teaches the answer, not the derivation. Removing a box under a stated constraint is a skill that only a page starting from the smallest design can rehearse |
| When-not-to-use on every component | Every pattern carries the measurement that would justify it and the scale threshold below which it is over-engineering | The common failure is a candidate adding a cache and a queue with no bottleneck in evidence. Restraint has to be taught explicitly, because complexity is easier to write |
| Level deltas | Every case study closes by answering the same prompt at mid, senior and staff level, side by side, annotating exactly what each level added | Most material serves one band and never says which. If you are targeting a promotion band, the delta is the part you actually need |
| Every answer collapsed | Checks, pre-questions, practice items and mastery items hide their answers in a disclosure block | A visible answer converts retrieval practice into rereading, which feels productive and is not |
| Nothing gated | No account, no email wall, no partially hidden section, no premium label, permanent URLs, printable pages | Rubrics and graded practice are the parts most often locked elsewhere. They are the parts that do the work, so they are the parts we refuse to lock |

One more structural choice, less visible: every page carries a last-reviewed date and a changelog, and contested claims are marked as contested. A claim about a named company appears only with a link to that company's own writing and a date on the link. You should be able to audit this section rather than trust it.

## The shared design framework

Every course in this section trains a different surface, but the rounds share one shape: you are given an underspecified prompt and about forty working minutes to show judgement under time pressure. This part of the hub holds what is true across all seven tracks, so the course pages never repeat it.

Three things are shared. The delivery clock below. The estimation discipline in the next section. The trade-off and failure references after that.

!!! warning "Read this before you memorise anything"

    The paid Grokking courses each ship an acronym spine (RESHADED, SCADET, SCALED, REDCAAP are theirs, one per course) and apply it identically to every case study. That is their choice, and it is the one thing we deliberately do not copy.

    A candidate who narrates phase names spends round time on labels rather than on decisions, and the transcript that results is hard to distinguish from one produced without thinking. We give you a clock and then teach you when to break it.

    To be exact about what we do and do not offer: these pages do follow a consistent structure by design, and the eighteen-section anatomy above is that structure. What we do not ship is a mnemonic to recite in the room. The structure is how the material is organised for study; it is not a script to narrate at an interviewer.

### The clock

The budgets below assume a 45 minute slot loses about 5 minutes to introductions, tooling and closing questions, leaving 40 working minutes, and that a 60 minute slot loses about 8 minutes for the same reasons, leaving 52. Those two deductions are assumptions, and they are reasonable because almost every remote round spends time on hellos, screen sharing and the candidate's own questions at the end. Adjust once you see how your interviewer runs the first two minutes.

| Phase | What ends the phase | 45 min round | 60 min round |
|---|---|---|---|
| Scope and clarify | You and the interviewer agree on what is being built and for whom | 5 min | 6 min |
| Requirements written down | A visible list of functional items plus two or three numbers | 4 min | 5 min |
| Estimation gate | Either a number that rules something out, or a sentence saying you are skipping it | 2 min | 3 min |
| V0 design | The smallest design that satisfies the written requirements, drawn | 8 min | 9 min |
| Breaking point and V1 | You name what fails, at what number, and change one thing | 8 min | 10 min |
| Deep dive | One or two areas taken to implementation detail | 9 min | 14 min |
| Failure modes and wrap | Named failure classes, what you would measure, what you skipped | 4 min | 5 min |
| **Total** | | **40 min** | **52 min** |

```
+--------+--------+-------+--------+--------+---------+-------+
| Scope  | Reqs   | Est   | V0     | Break  | Deep    | Wrap  |
| 5 min  | 4 min  | 2 min | 8 min  | 8 min  | 9 min   | 4 min |
+--------+--------+-------+--------+--------+---------+-------+
0        5        9       11       19       27        36      40
Elapsed working minutes across a 45 minute round. The two
widest blocks are the breaking point and the deep dive, not
the first drawing.
```

The shape matters more than the numbers. More than half the clock sits after the first diagram. Candidates who fail on delivery almost always spend that half drawing more boxes instead of stressing the ones they have.

### How the split moves by level

Same 52 working minutes, allocated differently. The splits below are our assumption about where each band should spend its budget, derived from what the level deltas in the case studies say each band adds. They are expressed as a reallocation rather than extra time, because nobody gets extra time.

| Phase | Mid | Senior | Staff |
|---|---|---|---|
| Scope and clarify | 5 | 6 | 9 |
| Requirements | 5 | 5 | 6 |
| Estimation | 4 | 3 | 2 |
| V0 | 12 | 8 | 6 |
| Breaking point and V1 | 10 | 12 | 10 |
| Deep dive | 11 | 14 | 12 |
| Failure modes and wrap | 5 | 4 | 7 |

Read the columns, not the cells. A mid-level answer is judged on whether a correct system appears. A senior answer is judged on the breaking point and the depth. A staff answer spends its budget at the two ends: negotiating what is worth building, and stating how it will be operated and what it will cost.

### The first two minutes

Four sentences, in this order. Say them before you draw anything.

1. Restate the prompt in your own words, with one number attached, so a mismatch in what you each think is being built surfaces now rather than at minute thirty.
2. Name the two or three things you believe are actually hard about it.
3. State the scope you propose, and name what you are dropping.
4. Ask the single clarifying question whose answer changes the design most.

A worked version, for a prompt of "design a link shortener":

> "So: a service that takes a long URL, returns a short one, and redirects on lookup, at something like a hundred million redirects a day. I think the hard parts are read latency on the redirect, key generation without collisions, and what happens when one link goes viral. I am going to treat analytics and custom vanity domains as out of scope unless you want them. Before I draw anything: are links permanent, or do they expire? That changes whether I need a deletion path at all."

That is under sixty seconds spoken. Note what it does not contain: no phase name, no acronym, no promise about what section comes next.

### Stating what you are deliberately not covering

Scoping out is a senior signal only when it is explicit and reversible. Silence reads as an oversight. Use one of these three forms.

| Form | Use it when | Example |
|---|---|---|
| Drop with a reason | The topic is real but not on the critical path | "I am leaving abuse and spam detection out. It is a real system, but it does not change the storage or the read path, which is where I think the interesting failure is." |
| Drop with a return ticket | You suspect the interviewer cares | "I am parking multi-region for now and building single region first. Flag me if you want it, and I will come back with what changes." |
| Drop by admission | You genuinely do not have the depth | "I have not operated a consensus system in production, so I will use a managed one as a black box and say what I depend on it for." |

The third form costs nothing and buys a great deal. Candidates lose more points defending a shallow claim than they would have lost by naming the gap.

### Checking in without sounding unsure

The difference is whether you are asking for reassurance or offering a routing decision. Offer a decision.

| Weak phrasing | What the interviewer hears | Stronger phrasing |
|---|---|---|
| "Does that make sense?" | You want reassurance and cannot self-assess | "The write path is settled. I want the next ten minutes on the read path, unless you would rather see storage." |
| "Should I do estimation now?" | You are following a script and asking permission | "I do not think volume decides this design. Say the word and I will run the numbers anyway." |
| "Is this right?" | You have no way to check your own work | "The part I am least sure of is the ordering guarantee here. I will state the assumption and move on, and come back if it turns out to matter." |
| "Sorry, I am going too fast" | You are apologising for pace instead of setting it | "I am moving fast on purpose so there is time for failure modes. Stop me anywhere you want more detail." |

Two check-ins in a 45 minute round is about right: one after V0, one before the deep dive. More than four and you have handed the interviewer the driving seat, which is the thing the round is testing.

!!! danger "WHEN TO BREAK THIS"

    The clock is a default for a prompt that arrives cold and open. Following it in any of these situations is the wrong move.

    1. **The interviewer opens with the deep dive.** If the prompt is "assume the feed service exists, tell me how you would shard it", there is no V0 phase. Spend the whole clock on the one question asked. Candidates who insist on running requirements first are visibly executing a script.
    2. **The interviewer interrupts.** An interruption is the highest-value signal in the round: it tells you exactly what is being graded. Follow it immediately. Never say "I will get to that later". If it destroys your plan, say so out loud and rebuild the plan in one sentence.
    3. **Estimation is noise for this prompt.** API and product architecture rounds, most frontend component rounds, and schema-centric prompts are decided by semantics, not volume. Two minutes of invented arithmetic in those rounds is two minutes of visible ritual. Use the one-line offer in the next section instead.
    4. **The prompt is brownfield.** If you are handed a running system and a new constraint, V0 already exists. The clock reallocates to blast radius, migration order and rollback, and the deep dive is usually the cutover.
    5. **The client is the system.** In mobile and frontend rounds, server QPS math is off-target. The equivalent gate is device and network budget: bytes shipped, memory ceiling, offline behaviour, battery.
    6. **You are visibly behind at the halfway mark.** Do not compress every remaining phase equally. Drop the second deep dive entirely, say you are dropping it, and protect the failure-modes block, because that is where senior signal concentrates.

### The tells, and what to do instead

| The tell | What it looks like | What to do instead |
|---|---|---|
| Narrating phases | "Now I will move to requirement engineering" | Announce the decision, never the phase. The structure should be visible from your output, not your labels. |
| Estimation with no consequence | Numbers computed, then never referenced again | Delete the number, or state the option it rules out in the same breath |
| Uniform depth | Every prompt deep dives on sharding | Choose the dive this problem makes interesting, and say why you chose it |
| The fixed box set | Load balancer, cache, queue, database, drawn before the requirements are written | Start from the fewest boxes the written requirements permit, then add under pressure |
| Coverage over depth | Ten components each described for thirty seconds | One component described for eight minutes beats ten described for one |
| Deferring the interviewer | "I will get to that in the deep dive section" | There are no sections. Follow the question now. |

---

## Interviewer script packs

Hand one of these to a friend, a partner or a flatmate. They need no engineering background: everything they say is written out, every probe carries the trigger that tells them when to use it, and the scoring sheet is ticked on observable behaviour rather than on whether the answer was correct.

How to run one:

- Give the reader the pack, a clock and nothing else. They do not prepare.
- The reader reads the opening prompt verbatim, then stays quiet.
- The reader uses a probe only when its trigger fires, and reads it as written.
- The reader introduces each curveball at the stated minute, whatever is happening.
- At the end the reader ticks four boxes and reads the tallies back. That is the whole feedback loop.

Each pack fits one screen so you can print it. Minutes are elapsed working minutes, counted from the moment the prompt is read.

### Pack 1: general system design

**Read this out loud, then stop talking.**

> "Design a service that turns a long web link into a short one, and sends
> anyone who opens the short one to the original. Assume a hundred million
> of those redirects a day. Take about forty minutes. Start wherever you want."

| Read this probe | Only when this trigger fires |
|---|---|
| "Before the boxes: what are you assuming about how many of these there are a day, and how big each one is?" | They start drawing within the first two minutes |
| "You said a number a moment ago. What did that number rule out?" | They state any figure and then never refer to it again |
| "If I told you the traffic was a hundred times smaller, which of those boxes would you delete?" | Their drawing reaches four or more boxes |
| "Pick the piece most likely to break first. What does a user see when it does?" | Nothing about failure has been said by minute 25 |
| "What are you deliberately not covering, and why that one?" | They go quiet, repeat themselves, or drift |

| Minute | Curveball, read as written |
|---|---|
| 12 | "One single link is now getting almost all of the traffic. Does anything in your design change?" |
| 22 | "Overnight you lose a whole datacentre. What stops working, and what keeps working?" |
| 32 | "Five minutes left. Tell me the part you skipped, and why you skipped it." |

| Tick the box if you saw this | Yes or no |
|---|---|
| They asked you at least one question before they started designing | |
| They said a number out loud and then used it to decide something | |
| They named something that could break, not just something that works | |
| They said what they were leaving out and gave a reason | |

Time-keeping: call out "ten minutes gone", "twenty", "thirty", then stop them at forty and do not grant extra time.

### Pack 2: machine learning system design

**Read this out loud, then stop talking.**

> "Design the system that picks which items to show a shopping app user on
> their home screen, using a machine learning model. About fifty million
> people open the app each day. Take about forty minutes."

| Read this probe | Only when this trigger fires |
|---|---|
| "What exactly is the model predicting, and where does one example of the right answer come from?" | They name a model, an algorithm or a framework before saying what it predicts |
| "If that measurement went up but sales went down, which one would you believe?" | They mention any accuracy, score or metric |
| "Walk me through what happens in the seconds between the user opening the app and seeing the list." | They describe training but have not described serving |
| "How many of these predictions per second, and how did you get that number?" | No figure of any kind has been stated by minute 15 |
| "What tells you it is time to update the model, and who notices if that stops happening?" | They say the model will be retrained or refreshed |

| Minute | Curveball, read as written |
|---|---|
| 14 | "Most of the examples you learn from come from what the old system already chose to show people. Is that a problem?" |
| 24 | "The model takes four hundred milliseconds and the screen has to be up in two hundred. What changes?" |
| 32 | "After a launch, one group of users got a much worse experience. How would you have caught that before launching?" |

| Tick the box if you saw this | Yes or no |
|---|---|
| They asked you at least one question before they started designing | |
| They said a number out loud and then used it to decide something | |
| They named something that could break, not just something that works | |
| They said what they were leaving out and gave a reason | |

Time-keeping: call out "ten minutes gone", "twenty", "thirty", then stop them at forty and do not grant extra time.

### Pack 3: generative AI system design

**Read this out loud, then stop talking.**

> "Design an assistant that answers employee questions by reading the
> company's own internal documents. Ten thousand employees, and the
> documents change every day. Take about forty minutes."

| Read this probe | Only when this trigger fires |
|---|---|
| "At the moment the model runs, what text is in front of it, and where did each piece of that text come from?" | They say they will use a language model without saying what it is given |
| "What happens when the document with the real answer is not one of the ones you fetched?" | They mention search, retrieval or looking documents up |
| "Who or what decides an answer was good, and how many answers get checked?" | They say the system will be evaluated, tested or measured |
| "What does one answer cost, and what did you multiply together to get that?" | No cost or latency figure has been stated by minute 18 |
| "Give me one question this thing must refuse, and tell me where the refusal happens." | They mention safety, guardrails, filtering or abuse |

| Minute | Curveball, read as written |
|---|---|
| 12 | "Some of those documents may only be read by some employees. Does anything in your design move?" |
| 22 | "The model provider gets slower: every answer now takes three seconds. Redesign the part the user sees." |
| 30 | "Someone proves it confidently invented a company policy that does not exist. What in your design catches that next time?" |

| Tick the box if you saw this | Yes or no |
|---|---|
| They asked you at least one question before they started designing | |
| They said a number out loud and then used it to decide something | |
| They named something that could break, not just something that works | |
| They said what they were leaving out and gave a reason | |

Time-keeping: call out "ten minutes gone", "twenty", "thirty", then stop them at forty and do not grant extra time.

??? note "Show answer: what a score of four out of four does and does not tell you"

    Four ticks means you did the four things visible to someone who cannot judge the content: you scoped before designing, you used a figure rather than merely producing one, you named a failure, and you made your omissions explicit. That is the delivery half of the round.

    It says nothing about whether the design was any good. For that, take the same prompt to the matching case study and grade yourself against its four-criterion rubric. Then run the pack again a week later with a different friend, so the prompt is still cold.

---

## Capacity estimation that earns its place

Most capacity math taught in interview courses is arithmetic on invented inputs, and the arithmetic is usually correct while the answer changes nothing. The fix is not better arithmetic. It is a rule about when a number is allowed to exist.

!!! tip "The hard rule"

    A number may appear in your answer only if the next sentence uses it to rule out a design option. If no option is ruled out, delete the number. This applies to every course page in this section and is enforced at review.

There is one legitimate exception: a number that is explicitly labelled an assumption because a later step depends on it. Those are allowed, and they must carry one sentence saying why the assumption is reasonable.

### The one-line offer

When the prompt does not turn on volume, say this and move on:

> "I can size this if it will drive a decision. Right now I think the thing deciding the design is the consistency requirement, not the request rate. Want me to run the numbers anyway?"

The sentence costs about eight seconds and does two jobs: it names what you think decides the design, and it hands the choice back, so you are covered if the interviewer did want the math.

### Anchors worth memorising

A short list, each with its derivation, so you can rebuild it under pressure instead of trusting a memorised table.

| Anchor | Value | Where it comes from |
|---|---|---|
| Seconds in a day | 86,400 | 24 x 60 x 60 |
| Seconds in a year | about 31.5 million | 365 x 86,400 = 31,536,000 |
| Requests per second from a daily count | daily count divided by 86,400 | 1 million per day is about 12 per second. 1 billion per day is about 11,600 per second |
| Peak multiplier over average | assume 2x to 5x | Traffic concentrates in waking hours for a single-region consumer product, so peak hour carries several times the mean. Reasonable because it is a diurnal pattern, not a spike |
| Short text post | about 300 bytes | 280 characters mostly 1 byte each in UTF-8, plus an 8 byte author id, an 8 byte post id and an 8 byte timestamp |
| URL mapping row | about 150 bytes | 7 byte code, roughly 100 byte URL, 8 byte owner id, 8 byte created-at, plus index and row overhead |
| Float32 embedding, 768 dimensions | 3,072 bytes | 768 x 4 bytes |
| Bytes per day from a write rate | writes per second x bytes x 86,400 | 1,000 writes per second at 300 bytes is 300 KB/s, which is about 26 GB per day |

### The latency ladder

Memorise the ratios, not the absolute values. The ratios are what eliminate options.

| Operation | Order of magnitude | Where it comes from |
|---|---|---|
| Main memory read | about 100 ns | Order-of-magnitude assumption for DRAM access on commodity servers. Reasonable because it has stayed within the same decade of nanoseconds for many hardware generations |
| SSD random read | about 100 microseconds | Roughly 1,000x a memory read |
| Spinning disk random read | about 9 ms | Half a rotation at 7,200 rpm is 60 / 7,200 / 2 = 4.2 ms, plus a seek of roughly 5 ms |
| Same datacentre round trip | under 1 ms | A few hundred metres of cable plus a small number of switch hops |
| Intercontinental round trip, 4,000 km each way | about 80 ms | Light in fibre travels roughly 200,000 km/s, so 8,000 km of glass is 40 ms, and real routes plus queueing roughly double it |

The three ratios that decide designs: memory to SSD is about 1,000x, SSD to spinning disk is about 90x, spinning disk to intercontinental network is about 9x. So a cross-region round trip costs about as much as nine disk seeks, and about a million memory reads.

### Worked estimate one: a number that eliminates

Prompt: a link shortener. GIVEN by the interviewer: 500 million links stored, 100 million redirects per day.

**Step 1, read rate.** 100,000,000 / 86,400 = about 1,160 redirects per second average. Assume a 4x peak, which sits in the 2x to 5x band above, giving about 4,600 per second. That rules out sharding the primary datastore for read throughput: a single node serving primary-key lookups handles thousands of reads per second, so throughput alone does not force a shard.

**Step 2, data size.** 500,000,000 rows x 150 bytes = 75 GB. That rules out sharding for size as well. 75 GB fits on one ordinary SSD with a large margin, so a single primary with replicas is still on the table, and any shard we add now would be for blast radius, not capacity.

**Step 3, cacheable working set.** Assume 10 percent of links take 90 percent of traffic, which is reasonable because link popularity follows the same heavy skew as the content it points at. 50,000,000 x 150 bytes = 7.5 GB. That rules out a sharded cache tier: 7.5 GB fits in the memory of one cache node, so the design is one cache plus a replica for availability, not a cluster.

Three numbers, three options eliminated, and the design is now smaller than the one most candidates draw.

### Worked estimate two: the honest skip

Prompt: make payment creation idempotent in a public API.

The tempting move is to size the idempotency key store: keys per day equals requests per day, multiply by a key record size, quote a number of gigabytes. Do not. Here is why, and here is what to say.

There is at most one key record per payment, and you already committed to storing every payment. So the key store is bounded above by data whose volume you have already accepted. No storage number can rule out any option, because every option holds the same rows.

What actually decides the design is whether the key record and the payment are written in one atomic step. If they are not, a retry that arrives during the gap creates a duplicate charge. That is a transaction-boundary question, and no amount of arithmetic touches it.

> "Sizing will not drive this one. The key set is bounded by the payment set, which we are storing anyway, so the decision is about the transaction boundary, not volume. I would rather spend the time on what happens when a retry lands in the middle of the write. Tell me if you want the numbers anyway."

??? note "Show answer: which of these two prompts still deserves a number, and which number"

    The payments prompt does deserve exactly one number, but not a storage number: the retention window for keys. If keys are kept forever, the table grows without bound and the uniqueness index degrades; if they are kept for 24 hours, a client retrying after a long outage gets a second charge. So the number to state is the client's maximum retry horizon, and it eliminates the "keep keys forever" option once you can name it. That is the rule working correctly: the number earns its place because an option dies.

!!! info "WHEN NOT TO ESTIMATE"

    The measurement that justifies estimation is simple: can you name, before you start, the design option a number would kill? If you cannot, skip it.

    Concretely, skip estimation when the interviewer already gave you the numbers and no derived quantity crosses a threshold, when the prompt is schema-centric or semantics-centric (API design, error models, pagination, a UI component), when the whole system fits on one machine by inspection, or when you have under 40 working minutes and an unresolved correctness question is still open.

    Below roughly a single-node-sized system, capacity math is theatre: the answer is always "one box", and spending two minutes proving it is two minutes not spent on the failure modes.

---

## The trade-off atlas

This is the reference the paid replacement dropped: the Design Gurus original carried a large chapter devoted purely to trade-offs, and the Educative rewrite has no equivalent. It lives on the hub, once, and each course adds its own domain rows rather than repeating these.

!!! warning "About the thresholds in column four"

    Where a specific threshold appears below, treat it as a labelled assumption, not a constant. Each one is an order-of-magnitude starting point for a system operated by a single ordinary team, chosen because that is the scale most interview prompts imply. The column's real job is to name the quantity you should measure. Replace every stated value with your own measurement the moment you have one, and in an interview, say the quantity out loud even if you do not have the value.

| Decision | Choose A when | Choose B when | The number that decides it | How it fails if you get it wrong |
|---|---|---|---|---|
| **A: relational. B: document or wide-column** | Query shapes are still changing, you need joins and multi-row transactions, and the working set fits one primary plus replicas | Two or three fixed access patterns, unbounded row growth, per-key atomicity is sufficient | Count of distinct query shapes, and largest table size against one node's disk. Above roughly ten query shapes, relational wins on flexibility | Choosing B early rebuilds joins in application code and loses them under concurrency. Choosing A too long forces a migration while on call |
| **A: normalise. B: denormalise** | Writes are frequent relative to reads and a shared entity must stay correct | Reads dominate, the read shape is stable, and you can build a repair path | Reads per write for that entity. Assume denormalisation starts paying above roughly 100 reads per write | Denormalising early makes every schema change a backfill. Too late, a join fan-out blows the read latency budget |
| **A: strong consistency. B: eventual** | Two users act on the same row and a stale read costs money or safety | The reader tolerates a bounded stale window and can be shown that it is stale | Perceivable staleness window in seconds, against the cross-region round trip you would pay per write | Strong across regions adds a network round trip to every write. Eventual under a uniqueness constraint creates duplicate accounts |
| **A: synchronous write path. B: asynchronous** | The caller cannot proceed correctly without the result, or durability must precede the response | The caller needs only an acknowledgement and the work completes within a stated delay | Endpoint tail latency budget, against the p99 of the work you would inline | Sync inlining a slow dependency exports its outage to your API. Async hides failure until a user asks where their thing went |
| **A: cache-aside. B: write-through** | Reads are skewed, the cache is often cold, and a miss is cheap to serve | The same rows are read immediately after being written and a miss is expensive | Fraction of writes read back within the TTL | Cache-aside without single-flight gives a stampede. Write-through fills the cache with rows nobody reads and evicts the ones people do |
| **A: push fan-out at write. B: pull at read** | Producers have small audiences and readers are many | A small number of producers have very large audiences | Freshness budget in seconds, times the per-author fan-out insert rate. At a 30 second budget and 20,000 inserts per second per author, push stops paying above 600,000 followers, [derived in the feed case study](modern-system-design.md#numbers-that-eliminate-options-feed). Recompute it with your own two inputs rather than carrying the 600,000 | Pure push turns one celebrity write into a million-row job. Pure pull turns every feed read into a scatter-gather over hundreds of producers |
| **A: monolith. B: services** | One team, one deploy cadence, and feature velocity is the bottleneck | Independent scaling or independent failure domains are required, and teams block each other on releases | Number of teams that must coordinate one release, and the peak-load ratio between the noisiest and quietest component | Splitting early converts function calls into network calls with no transaction. Splitting late means one bad deploy stops everything |
| **A: REST. B: RPC. C: events** | Public or partner-facing, cacheable resource reads, many clients you do not control | RPC for internal, latency-sensitive, high-volume calls with a schema both ends own. Events when the producer must not know its consumers and consumers may lag | Consumer count you control, plus per-call payload size and rate | Events for a request that needs an answer creates a correlation-id maze. REST between two internal services pays serialisation cost for nothing |
| **A: single region. B: multi region** | Users are geographically concentrated and an hours-long regional outage is survivable | A regulator requires data residency, or the recovery time objective is minutes | Recovery time objective in minutes, and cross-region round trip in ms against the write latency budget | Multi-region without a decided write model produces silent divergence. Single region turns one provider incident into your entire outage |
| **A: leader-follower. B: leaderless** | You need one ordering point and read-your-writes must be easy to reason about | Writes must survive the loss of any node and you can define a merge rule | Acceptable failover window in seconds, and whether the workload has a defensible conflict resolution rule | Leader-follower gives a write outage for the failover duration. Leaderless with last-write-wins silently drops concurrent writes |
| **A: batch. B: stream** | Consumers tolerate hours and the logic is complex enough to need reprocessing | A decision is made within seconds of the event arriving | Freshness requirement in minutes, stated by whoever consumes the output | Streaming a nightly report doubles operational surface for no user gain. Batching a fraud signal makes it arrive after the money left |
| **A: client rendering. B: server rendering** | Highly interactive after load, session-heavy, capable devices | First paint and crawlability matter, or the audience is on low-end devices and slow networks | First contentful paint budget in ms, and JavaScript bytes shipped on the critical path | Client rendering ships a blank screen to a slow phone. Server rendering moves per-request CPU cost onto your fleet |
| **A: long polling. B: WebSockets. C: SSE** | Long polling when updates are rare and the path must stay plain HTTP | WebSockets for frequent bidirectional low-latency traffic you can operate as sticky connections. SSE for server-to-client text with free reconnect | Messages per connection per minute, and concurrent connections against per-connection memory | WebSockets everywhere creates connection state you must now shed and rebalance. Long polling at high message rates burns one request per update |
| **A: shared tenancy. B: isolated** | Many small tenants, cost per tenant matters, no per-tenant compliance boundary | One tenant can be large enough to hurt others, or a contract requires separation | Ratio of the largest tenant to the median tenant. Assume isolation starts paying above roughly 50x | Shared lets one tenant's backfill become everyone's incident. Isolated multiplies deploy and migration cost by tenant count |
| **A: hash partitioning. B: range** | Access is by exact key and you want even spread | Queries are ordered scans over a key such as time or sequential id | Fraction of queries that are range scans | Hash makes a time-range query a full fan-out. Range on a monotonic key makes the newest partition the only hot one |
| **A: at-least-once plus idempotent consumer. B: transactional effectively-once** | The effect can be made idempotent with a key you control | The effect is external and cannot be deduplicated, such as moving money once | Cost of one duplicate, in currency or trust, against the throughput cost of transactional commit | Assuming exactly-once across a boundary that cannot provide it produces duplicates nobody planned for. Building a transactional pipeline where a dedup key sufficed costs throughput forever |
| **A: index. B: scan** | The predicate selects a small fraction of rows and the query is frequent | The predicate selects a large fraction, or the table is small enough that the scan beats the lookup | Selectivity. Assume an index stops paying above roughly 10 percent of rows returned | Over-indexing slows every write and inflates storage. Under-indexing turns a p50 of milliseconds into a p99 of seconds |
| **A: vertical scaling. B: horizontal** | You are well below the largest instance available, the workload is stateful, and the team is small | You are within a small factor of the largest instance, or you need failure isolation | Current peak utilisation as a fraction of the biggest single node you can buy | Scaling up hits a ceiling with no plan and forces a rushed shard under load. Scaling out early buys distributed-systems problems you did not need |
| **A: object store. B: database, for large values** | The value exceeds a few hundred kilobytes and is served by URL | The value is small, always read alongside its row, and must be transactionally consistent with it | Median object size, and whether the object is ever queried by its content | Blobs in the database bloat backups, caches and replication. Blobs in object storage without a consistency plan leave rows pointing at missing files |
| **A: cache. B: read replica** | A small hot subset serves most reads and seconds of staleness are fine | Reads spread across the whole dataset and must follow the schema | Achievable hit rate on the hot subset. Assume a cache stops earning its place below roughly 80 percent | A low-hit-rate cache adds a hop and a staleness bug for nothing. Replicas without lag monitoring serve stale reads that look like data loss |
| **A: optimistic concurrency. B: pessimistic** | Conflicts are rare and a retry is cheap | Conflicts are common, or a retry is expensive or user-visible | Measured conflict rate on the contended row. Assume optimistic stops working above roughly a 10 percent retry rate | Optimistic under contention becomes a livelock of retries. Pessimistic locks held across a network call create a lock convoy |
| **A: rate limit at the edge. B: at the service** | Abuse is volumetric and classifiable by IP, API key or route | The limit depends on business state such as plan, balance or per-tenant quota | Fraction of rejected requests that need business state to classify | Edge-only limiting lets through a cheap request that is expensive downstream. Service-only limiting means the attack still costs you the full request path |

??? note "Show answer: how to use a row of this table in a live round"

    Do not read the row. Say the quantity and the consequence, in that order, in one sentence: "Whether I denormalise here depends on reads per write for the feed entity. If it is above about a hundred to one I precompute, and I accept that a schema change becomes a backfill."

    That is roughly twelve seconds, and it does the two things the column exists to do: it names what you would measure, and it names what being wrong costs. Reciting a trade-off without either is the recited-template tell.

---

## The failure library

Almost every production incident worth naming in an interview belongs to one of a small number of classes. Recognising the class is the signal; inventing a bespoke story is not. Each case study in every track tags its failures against this list.

Most of these share one shape, which is worth drawing once:

```
+-------------+      +--------------+      +------------+
| Client      | ---> | Service      | ---> | Datastore  |
| retries x3  |      | queue grows  |      | slower     |
+-------------+      +--------------+      +------------+
       <                    <                     |
       +--------------------+---------------------+
A feedback loop. The extra load is produced by the slowdown
it is reacting to, so removing the original trigger does not
clear the failure.
```

| Name | What happens | What triggers it | The design change that prevents it |
|---|---|---|---|
| Retry storm | Retries multiply load on an already slow dependency, so offered load rises as capacity falls | A latency spike, plus fixed retry counts configured at more than one tier | One retry layer only, a retry budget capped as a fraction of base traffic, full-jitter backoff, and a circuit breaker |
| Cache stampede | A popular key expires and every concurrent request recomputes it at once | A shared TTL on a hot key, a cache flush, or a deploy that empties the cache | Single-flight or a per-key lock, probabilistic early recompute, and TTLs spread by a random offset |
| Thundering herd | Many waiters wake on one event and contend for the same resource | Broadcast wakeups, a shared cron minute, mass reconnect after a disconnect | Wake one waiter, spread schedules with a random offset per client, and reconnect with jittered backoff |
| Hot key | One key or partition absorbs a disproportionate share of traffic and saturates while peers idle | A viral entity, a monotonically increasing key, or a default tenant that owns most rows | Split the key with a random suffix, put a request-level cache in front, and let the read path serve that key from a replica set |
| Split brain | Two nodes each believe they are leader and both accept writes, so data diverges | A network partition, or a paused leader whose lease was assumed expired | Leases carrying a fencing token that the storage layer checks and rejects on, plus a quorum that both sides cannot form |
| Metastable failure | The system stays degraded after the original trigger is gone, because the degraded state generates its own load | A brief overload combined with a retry or queue feedback loop | Load shedding at the entry point, bounded queues, and a recovery path that drops work instead of replaying it |
| Cascading timeout | A slow leaf holds threads or connections at every caller above it until the whole path is exhausted | A caller timeout longer than its callee's, or no timeout configured at all | Timeouts that shrink with depth, deadline propagation across hops, and bulkheads so one dependency cannot consume the whole pool |
| Head-of-line blocking | One slow item at the front stalls everything behind it, including work that would have been fast | A single queue for mixed work sizes, strict ordering per connection, one shared thread pool | Separate queues by cost class, per-partition rather than global ordering, and a concurrency limit per class |
| Backpressure collapse | Producers keep accepting work the system cannot finish, so memory and latency grow until the process dies | Unbounded buffers, or an ingest endpoint that always returns success immediately | Bounded queues that reject when full, admission control at the edge, and a 429 or 503 that clients are built to honour |
| Clock skew | Two machines disagree about time, so last-write-wins picks the wrong write or a token is honoured after expiry | Wall-clock timestamps used for ordering or lease expiry, plus ordinary NTP drift | Logical clocks or monotonic sequence numbers for ordering, and leases measured as elapsed local time rather than absolute deadlines |
| Poison message | One unprocessable message is retried forever, blocking or burning the consumer | A schema change, a null field, or an entity deleted between publish and consume | A delivery-attempt counter, a dead letter queue with an alert on it, and a consumer that can skip and record rather than halt |
| Unbounded queue growth | The queue absorbs a permanent throughput deficit and converts it into ever-growing delay nobody notices until it is hours | Producers faster than consumers for a sustained period, with alerting on depth rather than age | Alert on oldest-message age, autoscale consumers on age, and set a maximum age after which messages are dropped or diverted |
| Cold-start collapse | After a restart or deploy an empty cache sends full traffic to the origin, which cannot serve it | A capacity plan that silently assumes a warm cache, or simultaneous restarts across the fleet | Rolling restarts, cache warming before taking traffic, and sizing the origin for a stated fraction of cold traffic |
| Lock convoy | Threads queue on one lock and each handoff costs more than the work, so throughput falls as concurrency rises | A coarse lock on a hot path, or a single row updated by every request | Shard the lock or counter, batch updates, and prefer atomic operations over read-modify-write |
| Gray failure | A node is up and passing health checks but serving slowly or incorrectly, so it stays in rotation and poisons a fraction of traffic | Health checks that test the process rather than a real request path | Health checks that exercise a genuine dependency path, outlier ejection on latency, and client-side load balancing that reacts to p99 |

!!! info "WHEN NOT TO ADD THESE PREVENTIONS"

    Every prevention above is a component, and every component has a scale below which it is over-engineering. The measurement that justifies each one is in the second column.

    | Prevention | Do not add it until |
    |---|---|
    | Circuit breaker | You have more than one remote dependency on the request path and have observed one stay slow for over a minute. Below that, a timeout plus a single retry is the whole answer |
    | Dead letter queue | You have an asynchronous consumer at all. Under a few messages per second with a human watching, a log line and manual replay is cheaper to build and to operate |
    | Fencing tokens | Two writers can actually exist, meaning you have a real leader election. A single-writer system does not need them and paying for them signals pattern-matching |
    | Load shedding | Peak exceeds provisioned capacity often enough that the event has a name. Below that, autoscaling with headroom is simpler and fails less strangely |
    | Key splitting | You have measured one partition taking several times the median. Splitting costs a scatter-gather on every read of that key, forever |
    | Cache warming | The origin genuinely cannot serve cold traffic. If it can serve 100 percent of traffic without the cache, the cache is a latency optimisation and warming it is wasted machinery |

---

## Glossary

Terms used across more than one track, defined once here. Course pages link to this list rather than redefining them.

| Term | Definition |
|---|---|
| At-least-once delivery | A guarantee that a message arrives one or more times, so consumers must be idempotent |
| Availability zone | An isolated failure domain within a region, with independent power and network |
| Backfill | Recomputing or repopulating historical data after a schema, model or logic change |
| Backpressure | A signal from a slow consumer that makes producers slow down or stop, rather than buffering without limit |
| Blast radius | The set of users, tenants or components a single failure or change can affect |
| Bloom filter | A compact probabilistic set membership structure with false positives but no false negatives, used to avoid pointless lookups |
| Brownfield prompt | An interview prompt that hands you a running system plus a new constraint, rather than a blank page |
| Bulkhead | A resource partition, such as a per-dependency connection pool, so one saturated dependency cannot exhaust everything |
| CAP | The observation that a partitioned system must choose between consistency and availability for the duration of the partition |
| CDN | A network of geographically distributed caches that serve content near the user |
| Change data capture | Streaming a database's committed changes out as an ordered event log |
| Circuit breaker | A client-side switch that stops calling a failing dependency for a cooldown period instead of retrying into it |
| Cold start | The period after a restart, deploy or scale-out during which caches, connections or models are not yet warm |
| Consistent hashing | A key-to-node mapping where adding or removing a node moves only a small fraction of keys |
| Coordinated omission | A measurement error where a load generator stops issuing requests while the system is slow, hiding the worst latencies |
| CRDT | A data type whose concurrent updates merge deterministically without coordination |
| Cursor pagination | Paging by an opaque position token rather than an offset, so results stay stable while the underlying data changes |
| Dead letter queue | A separate destination for messages that failed processing repeatedly, so they stop blocking the main queue |
| Denormalisation | Storing derived or duplicated data so a read does not have to join or fan out |
| Dual write | Writing the same fact to two stores from application code, which has no atomicity and is the usual source of divergence during migrations |
| Durability | The guarantee that an acknowledged write survives the failures you have declared you will tolerate |
| Exactly-once processing | The property that each input affects the output state once, normally achieved by at-least-once delivery plus deduplication or transactional commit |
| Exponential backoff | Retrying after a delay that grows multiplicatively, ideally with full jitter so clients do not resynchronise |
| Fan-out | The number of downstream writes or reads produced by one upstream operation |
| Fencing token | A monotonically increasing number attached to a lease so a storage layer can reject writes from a stale holder |
| Gray failure | A component that is up and passing health checks while serving slowly or incorrectly |
| Head-of-line blocking | One slow item at the front of an ordered channel delaying everything behind it |
| Hedged request | Sending a duplicate request to a second replica after a short delay and taking the first response, to cut tail latency |
| Hot key | A single key or partition receiving a disproportionate share of traffic |
| Idempotency key | A client-supplied identifier that lets a server recognise a retry of the same logical operation and return the original result |
| Jitter | Deliberate randomness added to retry delays, TTLs or schedules so independent clients do not synchronise |
| Leaderless replication | A model where any replica accepts writes and conflicts are resolved on read or by a merge rule |
| Linearizability | The property that every operation appears to take effect at a single instant between its call and its return |
| Load shedding | Rejecting a fraction of requests at the entry point to keep the rest within their latency budget |
| Long polling | A client HTTP request the server holds open until data is available or a timeout fires |
| Materialised view | A stored, precomputed query result that is refreshed rather than recomputed per read |
| Metastable failure | A degraded state that sustains itself through a feedback loop after the original trigger has gone |
| Multi-tenancy | Serving multiple customers from shared infrastructure, with isolation enforced logically rather than physically |
| Outbox pattern | Writing an event into the same transaction as the state change, then relaying it, so the two cannot diverge |
| p99 | The latency below which 99 percent of requests complete. Usually the number a user-facing budget is written against |
| PACELC | An extension of CAP noting that even without a partition you still trade latency against consistency |
| Partition key | The field whose value determines which shard or partition a record lives in |
| Point of presence | An edge location where a provider terminates user connections close to the user |
| Quorum | A subset of replicas large enough that any two such subsets overlap, which is what makes reads see committed writes |
| Rate limiter | A control that caps request rate per client, key or route, commonly implemented as a token bucket |
| Read repair | Fixing a stale replica opportunistically when a read observes the inconsistency |
| Read-your-writes | The guarantee that a client that just wrote a value will not subsequently read an older one |
| Replication lag | The delay between a write committing on the leader and becoming visible on a follower |
| Retry budget | A cap on retries expressed as a fraction of base traffic, so retries cannot become the majority of load |
| Saga | A long-running business transaction expressed as a sequence of local transactions plus compensating actions |
| Serializability | The property that concurrent transactions produce a result equal to some serial order of them |
| Server-sent events | A one-way stream of text events from server to browser over HTTP, with automatic reconnection |
| Shadow traffic | Copying live requests to a new system without using its responses, to compare behaviour before cutting over |
| Shard | One horizontal slice of a dataset, holding a subset of keys and served independently |
| Split brain | Two nodes both acting as leader during a partition, so both accept writes and the data diverges |
| Sticky session | Routing a client's requests to the same server instance, which trades load balance for local state |
| Tail latency | The slow end of the latency distribution, normally p99 and above, which dominates perceived quality |
| Thundering herd | Many clients waking or reconnecting simultaneously and contending for the same resource |
| Token bucket | A rate-limiting algorithm holding refillable tokens, allowing bursts up to the bucket size |
| Two-phase commit | A blocking atomic commit protocol across participants, which trades availability for atomicity |
| Vector clock | A per-node counter set that lets a system detect whether two versions are concurrent or causally ordered |
| Watermark | A stream-processing marker asserting that no events older than a given time are expected, used to close windows |
| WebSocket | A persistent bidirectional connection over a single upgraded HTTP connection |
| Working set | The subset of data actually touched over a given window, which is the number that decides cache sizing |
| Write amplification | The ratio of bytes physically written by the storage engine to bytes logically written by the application |
| Write-ahead log | An append-only durability record written before applying a change, which is what makes crash recovery possible |

---

## Provenance

**Last reviewed:** 2 August 2026. Any page in this section whose last-reviewed date passes nine months is flagged for review rather than quietly left in place.

**Changelog, most recent first**

| Date | Change |
|---|---|
| 2 August 2026 | Interviewer script packs published, so the promise made in the header is now an artifact on this page. Competitor claims rechecked against the publishers' own pages: the "birthplace" quotation moved to the course page that carries it, a disputed course count replaced with a qualitative description, and a lesson count whose source now returns 404 removed. The fan-out threshold in the trade-off atlas restated as the derivation used in the feed case study |
| August 2026 | First publication of the hub: the seven-path router, the shared delivery clock, the estimation anchors, the trade-off atlas, the failure library and the glossary |
| August 2026 | Cross-cutting material moved off the individual course pages and onto this hub, so the clock and the anchors are stated once and linked rather than repeated seven times |
| August 2026 | Every figure on this page re-derived from inputs stated in the same section, and anything not derivable relabelled as an assumption with the reason it is reasonable |

**Found a problem?** Open an issue or a pull request at [github.com/datascienceinterviews/datascienceinterviews.github.io/issues](https://github.com/datascienceinterviews/datascienceinterviews.github.io/issues). Disputed numbers, dead links, better derivations and missing counter-arguments are all in scope.

All content in this section is original. Research informed which topics belong here; no prose, framework, acronym, example, table or diagram is derived from any paid course. Primary sources are cited inline with their edition and year, and any claim we could not source is either labelled as an assumption or is not on the page.
