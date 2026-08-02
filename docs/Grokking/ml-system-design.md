---
title: Machine Learning System Design Interview Course
description: A free course on designing ranking, retrieval and risk systems end to end, from objective and labels to serving, evaluation, drift and cost.
last_reviewed: 2026-08-01
---

# Machine Learning System Design Interview

!!! abstract "The contract"

    **After this course you can** take a one-line ML prompt, turn it into a trainable objective with a guardrail, derive a serving path that fits a stated latency budget, name the number at which it stops working, and say how you would know in production that it had.

    **Who it is for**

    - An ML engineer or applied scientist with roughly 2 to 8 years of shipping models, facing a 45 or 60 minute design round.
    - A backend engineer joining an ML team who can build the pipeline but stalls when asked what the system optimises and what it trades away to get it.
    - A senior candidate whose feedback keeps saying "described a pipeline" and who cannot tell which minutes went to the wrong thing.

    **Who it is not for**

    | If your round is about | Go here instead |
    |---|---|
    | A product built on a foundation model: retrieval-augmented generation, agents, prompts, token cost | [Generative AI System Design](generative-ai-system-design.md) |
    | A service, a store or a data pipeline with no model in the prompt | [Modern System Design](modern-system-design.md) |
    | An API contract, error model, domain model, versioning, webhooks | [Product Architecture](product-architecture.md) |
    | On-device inference, model updates over the air, battery and offline behaviour | [Mobile System Design](mobile-system-design.md) |
    | Deriving a loss function, proving convergence, or coding a model from scratch | [Machine learning question bank](/Interview-Questions/Machine-Learning/) |
    | A round inside the next week | [Fast-Track in 48 Hours](system-design-fast-track.md) |

    **Trains for:** the 45 and 60 minute ML system design round in machine learning engineer, applied scientist and ML platform loops, plus the design portion of senior and staff ML loops.

    **Does not train for:** ML coding rounds, probability and statistics screens, research-style modelling rounds, product-sense rounds, or behavioural rounds.

    **Fast path:** already comfortable? Go straight to the [self-assessment rubrics](#11-self-assessment-rubrics). Just want the tables? Go to the [reference block](#16-reference-block).

---

## Prerequisites

Seniority is a weak readiness signal and a job title is worse. Prerequisite mastery is the signal that works. Answer every Quick check in under a minute. Two or more misses means start with the linked material rather than with Module 1.

| You should already be able to | Where to learn it | Quick check |
|---|---|---|
| Say what moving a decision threshold costs, in product terms and not in metric names | [Confusion matrix](/Machine-Learning/Confusion%20Matrix/) | A spam filter moves its threshold from 0.9 to 0.6. Which of precision and recall rises, and which group of users notices first? |
| Convert a daily volume into a per-second rate without a calculator | [Estimation anchors on the hub](index.md#anchors-worth-memorising) | 40 million predictions a day is roughly how many per second? |
| Say how class imbalance changes which evaluation curve is informative | [Unbalanced and skewed data](/Machine-Learning/Unbalanced,%20Skewed%20data/) | The positive rate is 0.2 percent. You double the positives you catch. Which curve moves visibly, the receiver operating characteristic or the precision-recall curve? |
| Name the leak in a validation split | [scikit-learn cross-validation guide](https://scikit-learn.org/stable/modules/cross_validation.html) | Your data spans a year and you split it randomly. State the leak in one sentence, and what it does to your offline number |
| Size a vector table in bytes | [Estimation anchors on the hub](index.md#anchors-worth-memorising) | 1 million items, 256 dimensions, float32. How many bytes before any index overhead? |
| Reason about a latency budget across chained stages | [Google SRE Book, chapter 6, free online](https://sre.google/sre-book/monitoring-distributed-systems/) | Retrieval p99 is 60 ms and ranking p99 is 30 ms, run in sequence. Is the end-to-end p99 exactly 90 ms? |
| Compute a rate metric per slice in SQL | [SQL question bank](/Interview-Questions/SQL-Interview-Questions/) | Which clause gives you click-through rate per country, and what goes wrong for a country with three impressions? |
| Read an inconclusive experiment without inventing a result | [A/B testing question bank](/Interview-Questions/AB-testing/) | Your test is flat after three days. Name two legitimate fixes and the one move that is cheating |

??? note "Show answers to the quick checks"

    1. Recall rises, precision falls. The people who notice first are the ones whose legitimate mail is now filtered, because a false positive is visible to its victim and a false negative is invisible to everyone.
    2. 40,000,000 divided by 86,400 is about 463 per second. Memorise 86,400 and stop multiplying out the hours.
    3. The precision-recall curve. The negative class is so large that catching more positives barely moves the false positive rate, so the receiver operating characteristic curve looks almost unchanged while precision moves a lot.
    4. A row from a later week trains the model that predicts an earlier week, so the model has seen the future. The offline number goes up, production does not, and nothing in your validation report tells you why.
    5. 1,000,000 times 256 times 4 bytes equals 1,024,000,000 bytes, so about 1.02 GB. Any approximate nearest neighbour structure adds to this rather than replacing it.
    6. No. Both stages rarely have their slow request on the same call, so the end-to-end p99 sits below 90 ms. Ninety is a safe upper bound to budget with, and the real value is something you measure rather than derive.
    7. GROUP BY country over impressions joined to clicks. What goes wrong is that three impressions and one click reports 33 percent, so you need a minimum denominator, a confidence interval, or a smoothed rate before anyone reads the number.
    8. Legitimate: run longer to collect more samples, or reduce variance (better randomisation unit, a less noisy metric, covariate adjustment). Cheating: watching the number daily and stopping the moment it crosses, which inflates the false positive rate well past the level you think you are running at.

---

## Time budget

| Part of the course | Reading | Practice | Spaced review |
|---|---|---|---|
| Modules 1 to 3, scope, objective design, sizing | 100 to 125 min | 1.0 to 1.5 h | included below |
| Modules 4 to 5, data, labels, features | 90 to 120 min | 1.5 to 2.0 h | included below |
| Modules 6 to 9, retrieval, ranking, calibration | 175 to 225 min | 1.5 to 2.5 h | included below |
| Modules 10 to 12, evaluation and serving | 125 to 165 min | 1.5 to 2.0 h | included below |
| Modules 13 to 16, training cost, drift, patterns, levels | 110 to 145 min | 0.5 to 1.0 h | included below |
| Eight case studies, attempted before reading each arc | included above | 3.0 to 4.0 h | included below |
| Five review sessions on days 1, 3, 7, 21 and 60 | 0 | 0 | 3.0 to 4.0 h |
| **Total** | **600 to 780 min, so 10 to 13 h** | **9 to 13 h** | **3 to 4 h** |

!!! note "How these were computed, so you can argue with them"

    Reading is summed per module from the Module Map table below, not guessed for the page as a whole. Those sixteen module ranges total 600 minutes at the low end and 780 at the high end, which is the 10 to 13 hour figure above. Per-module minutes assume 120 to 150 words a minute, a deliberately slow engaged rate, because dense technical prose with diagrams and arithmetic is not read at prose speed.

    Practice is the sum of five components, all of them on this page, so you can count the items yourself and argue with the per-item minutes rather than with a total.

    | Practice component | Items | Minutes each | Minutes |
    |---|---|---|---|
    | Attempting a case study before reading its arc | 8 | 23 to 30 | 184 to 240 |
    | Fade-the-scaffold exercises, one per module | 16 | 6 to 10 | 96 to 160 |
    | Blocked practice set, Section 10a | 5 | 15 to 25 | 75 to 125 |
    | Interleaved practice set, Section 10b | 6 | 20 to 30 | 120 to 180 |
    | Timed self-mock: a 45 to 55 minute run at a case study you have already read, plus 20 minutes of written self-debrief | 1 | 65 to 75 | 65 to 75 |
    | **Total practice** | **36** | | **540 to 780 min, so 9 to 13 h** |

    The practice cells in the table above are rounded to the nearest half hour and spread those same five components across the part of the course each one belongs to. The itemised minutes here are the exact figures. Review assumes five sessions of 36 to 48 minutes, which is the 3 to 4 hour column. Every figure is padded by roughly 15 percent, because self-estimates of study time run optimistic.

    Minutes are the authoritative unit here. A single headline number would be more marketable and less true. At an hour a day this is three weeks; full time it is four to five days.

---

## Learning objectives

Each objective is tested by exactly one numbered item in the mastery check, and the numbering matches: objective 1 is tested by mastery item 1.

1. **Translate** a stated business objective into a trainable objective plus at least one guardrail metric in under five minutes, and **name the specific behaviour the model would produce** if that guardrail were removed.
2. **Compute**, for a given prompt, the candidate pool size, a per-stage latency split that sums to the stated serving budget, and the embedding index footprint in gigabytes, showing each step, and **delete every number that eliminates no design option**.
3. **Choose** between batch precompute, online inference and streaming features for a stated freshness requirement and request rate, **citing the freshness figure in minutes that decides it** and the cost the rejected option would have carried.
4. **Diagnose** a described offline-to-online metric disagreement into exactly one named cause (training-serving skew, a join that is not point-in-time correct, position bias, delayed labels, or an evaluation slice that does not match live traffic) and **name the single check that confirms it**, in under three minutes.
5. **Design** a retrieval-then-ranking path for a stated pool size and latency budget, **stating for each stage the candidate count it reduces to** and the recall that stage is permitted to lose.
6. **Specify** an online experiment for a stated baseline rate and target lift, giving samples per arm and duration in days, and **state the condition under which you would ship despite a flat primary metric**.
7. **Rewrite** a mid-level ML answer into a staff-level one by adding label delay handling, a retrain cadence with its monthly cost, a rollback criterion and one degradation path, annotating what each addition changed about the decision.

---

## Warm-up retrieval

Three to five minutes. Answer out loud or in writing before opening anything. Retrieval beats rereading, and a visible answer converts one into the other.

**Q1.** From the hub's round router: name two signals inside the first minute that tell you this is an ML design round rather than a generalist distributed-systems round.

??? note "Show answer"

    First, the prompt names a judgement the system must make (rank, recommend, score, detect, predict) rather than an artefact it must store or move. Second, the first follow-up is about labels, metrics or freshness rather than about capacity or consistency. A third, weaker signal: the interviewer asks who consumes the output and what decision it drives, which is a question about an objective, not about a request path.

**Q2.** From the trade-off atlas on the hub, several pages back: state the decision rule for batch versus stream, and name the number that decides it.

??? note "Show answer"

    Batch when consumers tolerate hours and the logic is complex enough that you will want to reprocess it. Stream when a decision has to be made within seconds of the event arriving. The deciding number is the freshness requirement in minutes, and it has to come from whoever consumes the output rather than from you. Streaming a nightly report doubles the operational surface for no user gain; batching a fraud signal makes it arrive after the money left.

**Q3.** The hub states one hard rule about when a number is allowed to appear in your answer. State the rule, and state the one-line offer you make when a prompt does not turn on volume.

??? note "Show answer"

    The rule: a number may appear only if the next sentence uses it to rule out a design option. If nothing is eliminated, delete the number. The single exception is a number explicitly labelled an assumption because a later step depends on it, and it carries one sentence saying why it is reasonable. The offer: "I can size this if it will drive a decision. Right now I think the thing deciding the design is the label delay, not the request rate. Want me to run the numbers anyway?"

---

## Module map

```
+--------------------------------+
| FRAME              M1 to M3    |
| scope, objective and guardrail |
| design, estimation that cuts   |
+---------------+----------------+
                |
                v
+--------------------------------+
| DATA               M4 to M5    |
| labels, label delay, features, |
| point-in-time joins, skew      |
+---------------+----------------+
                |
                v
+--------------------------------+   +----------------------+
| SERVING PATTERN    M6 to M8    |   | TRUTH                |
| retrieve, filter, rank, index, |==>| M9  calibration      |
| model choice under latency     |   | M10 offline eval     |
+---------------+----------------+   | M11 online eval      |
                |                    +-----------+----------+
                |                                |
                v                                v
+-------------------------------------------------------+
| OPERATE AND JUDGE                      M12 to M16     |
| serving and rollout, training cost, drift, patterns,  |
| level calibration                                     |
+-------------------------------------------------------+

Caption: the five clusters of this course and their reading order.
Frame feeds Data, because you cannot choose features before you
know the objective. Data feeds the Serving Pattern. The Serving
Pattern feeds Truth, which is calibration and both evaluation
layers. Both feed Operate and Judge, where mid-level and staff
answers separate. Arrows point from prerequisite to dependent.
```

| Module | What you will be able to do | Time | Skip if |
|---|---|---|---|
| M1 Scope and honesty | Tell within the first minute whether you are in an ML design round, a modelling round or a data round, and steer back when the title and the questions disagree | 15 to 20 min | You have been told your exact round format in writing and it matches |
| M2 Problem framing | Turn a business objective into a trainable objective with a guardrail, and name what the model will do to game a metric you left unguarded | 40 to 50 min | Never skip. Every case study on this page starts here, and this is where most rounds are lost in the first ten minutes |
| M3 Estimation for ML systems | Size candidate pools, index footprint, training rows per day and cost per thousand inferences, and delete any number that eliminates nothing | 45 to 55 min | You can already state what each of your numbers ruled out, every time |
| M4 Data and labels | Design a labelling plan under cold start, weak supervision and delay, and say what the delay does to your retrain loop | 40 to 55 min | You have shipped a system whose labels arrived days late and can describe how you evaluated it in the meantime |
| M5 Features and the feature store | Split feature computation across batch, request time and streaming, and write a point-in-time correct join without leaking | 50 to 65 min | You have debugged a training-serving skew incident end to end |
| M6 Retrieval then ranking | Split a serving path into candidate generation, filtering, ranking and re-ranking, with a latency budget that sums to the SLO | 40 to 50 min | You operate a multi-stage ranker and can state each stage's candidate count from memory |
| M7 Retrieval infrastructure | Choose an approximate nearest neighbour index family on build, query and memory cost, and plan a refresh and hot swap | 50 to 65 min | You have tuned and measured a vector index's recall against latency yourself |
| M8 Model choice as a design decision | Defend gradient boosted trees over a network on a stated feature and latency profile, and say when a two-tower model earns its complexity | 40 to 50 min | You can already name the tabular feature count and latency budget at which you would switch |
| M9 Calibration and bias | Say why an auction needs calibrated probabilities, correct position bias, and name the loop that makes popular items more popular | 45 to 60 min | You have shipped a calibration layer and can explain what it did to your ranking order |
| M10 Offline evaluation | Pick metrics by task type, build slices that expose the failure you care about, and state what an offline win does not tell you | 35 to 45 min | You run slice-based error analysis as routine practice |
| M11 Online evaluation | Size an experiment from a baseline rate and a target lift, and read a result that disagrees with your offline metric | 45 to 60 min | You size and read your own experiments and have called at least one launch off on a guardrail |
| M12 Serving and deployment | Choose batch, online or streaming serving from a freshness number, and specify canary, shadow and rollback criteria | 45 to 60 min | You have run a shadow deployment and can state the criterion that ended it |
| M13 Training infrastructure and cost | Attach a monthly cost to a retrain cadence and show how that cadence constrains the architecture | 30 to 40 min | You already do capacity and cost planning for training jobs |
| M14 Monitoring and drift | Distinguish covariate, label and concept shift, detect them when labels are late, and design an alert that does not cry wolf | 40 to 50 min | You have an ML dashboard you designed and an alert that has fired correctly |
| M15 Patterns and anti-patterns | Name the shape of a failing ML system from a symptom, and name the repair | 20 to 30 min | Never skip. This is the highest-yield module for senior candidates and takes half an hour |
| M16 Level calibration | Say what your answer is missing to read one level higher | 20 to 25 min | Never skip if you are targeting senior or above |

Low ends total 600 minutes, high ends total 780. That is the 10 to 13 reading hours in the time budget above.

---

## The delivery clock for this round

Most candidates who fail an ML design round did not lack a fact. They described a pipeline that could exist for almost any prompt, and never committed to an objective, a label, a latency budget or a way of being wrong. The clock below is a budget, not a script, and this round's budget is shaped differently from a generalist one: more minutes on the objective and the data, fewer on boxes.

### The 45 minute round

| Minutes | Spend it on | You leave this phase with |
|---|---|---|
| 0 to 2 | Restate the prompt as a prediction task, and name who consumes the output | The interviewer agreeing, or correcting you cheaply |
| 2 to 9 | Clarify, then write down the objective, the guardrail, the label and the freshness need | Two or three numbers tagged GIVEN or ASSUMED, and a label definition |
| 9 to 13 | Estimation, only where a number will eliminate an option | One or two decisions already made, not a page of arithmetic |
| 13 to 21 | v0 drawn end to end: data in, training, serving, feedback | A correct system at the stated scale, on the board |
| 21 to 30 | The breaking point, then v1 | A named metric, a threshold, and the one stage you added |
| 30 to 40 | One deep dive, ideally the one the interviewer picks | Depth in a single area, with a trade-off stated both ways |
| 40 to 45 | Evaluation plan, failure modes, what you would measure first, what you skipped | An explicit list of what you did not cover and why |

The phases sum to 45 by construction: 2 plus 7 plus 4 plus 8 plus 9 plus 10 plus 5.

### The 60 minute round

| Minutes | Spend it on |
|---|---|
| 0 to 2 | Open and restate as a prediction task |
| 2 to 11 | Clarify, objective, guardrail, label definition, freshness |
| 11 to 16 | Estimation that eliminates |
| 16 to 25 | v0 end to end |
| 25 to 36 | Breaking point and v1 |
| 36 to 52 | Two deep dives, different in kind from each other |
| 52 to 58 | Offline and online evaluation, monitoring, drift, cost |
| 58 to 60 | Close: trade-offs, what you skipped, what you would measure first |

These sum to 60: 2 plus 9 plus 5 plus 9 plus 11 plus 16 plus 6 plus 2. The extra 15 minutes buys one more deep dive and a real evaluation and monitoring section. It does not buy more boxes.

### The first two minutes

Say four sentences, in this order. Rehearse them once so they are automatic and you can spend your attention on listening.

1. "Let me restate it as a prediction: for each X we predict Y, and the consumer of that prediction is Z." (If you cannot fill in all three slots, that is your first clarifying question.)
2. "I am going to spend about seven minutes on the objective and the labels, then draw the system end to end, then break it on purpose." (You just told them your clock, which reads as control.)
3. "I will optimise for A, and I will hold B as a guardrail so we do not win A by damaging it." (Naming the guardrail is the cheapest senior signal available in this round.)
4. "Stop me if you want depth somewhere specific rather than coverage." (Interviewers take this invitation more often than candidates expect.)

### Declaring what you are skipping

Skipping silently reads as not knowing. Skipping out loud reads as prioritising. The pattern is: name it, say why it cannot change a decision in the next twenty minutes, say what you would do if it were in scope, in one sentence.

- "I am not designing the training cluster. The model fits on one machine at this data volume, so the distributed training choice cannot change anything on the serving path, which is where the latency budget is tight."
- "I am skipping the cold-start ranker for brand new users. It matters commercially, and it is a separate model with its own evaluation. If you want it, it is a popularity prior blended down as personal signal accumulates."
- "I will not size the feature store. Nothing here changes based on whether it holds 2 or 20 terabytes, because the deciding constraint is the point-in-time join, not the volume."

### Checking in without sounding unsure

A weak check-in asks for approval. A strong one offers a fork.

| Weak, reads as unsure | Strong, reads as driving |
|---|---|
| "Does that make sense?" | "That is the retrieval stage. Do you want ranking next, or the label pipeline that trains it?" |
| "Is this the model you wanted?" | "I have assumed labels arrive within an hour. Correct me now if it is days, because that decides the retrain loop and the evaluation." |
| "Should I go deeper?" | "I can go one level into the index refresh, or stay wide and cover monitoring. Which is more useful to you?" |
| "Sorry, I am rambling." | "I am three minutes over on labels. Moving to the serving path." |

### How the split shifts by level

| Phase | Mid | Senior | Staff |
|---|---|---|---|
| Objective and clarify | Same | Same | Longer: challenges whether a model is the right instrument at all |
| Estimation | Full, shown step by step | Trimmed to the numbers that decide the serving mode | Often one number, then a monthly cost or headcount consequence |
| Data and labels | Named as a source | A plan covering delay, cold start and bootstrap | The constraint the rest of the design is built around |
| v0 and v1 | Most of the time here | Balanced | Compressed, because it is assumed correct |
| Deep dives | One, chosen by the interviewer | Two, one chosen by the candidate and defended | Two, plus the migration path from whatever exists today |
| Evaluation and monitoring | A metric named at the end | Offline plus online, with a stated ship criterion | Woven throughout, plus drift, rollback and who gets paged |

The rough shape: mid level spends its minutes proving a working system could be built, senior spends them proving the metric and the trade-off were chosen deliberately, staff spends them proving the system is reachable from today, operable when the labels are late, and affordable next quarter.

!!! tip "One note on frameworks, made once"

    A lot of paid interview material sells an acronym spine: one memorable word whose letters name the phases, applied in the same order to every case study. This page does not have one, and that is deliberate.

    The problem is not the letters. It is that an identical spine applied to nine prompts produces an answer that sounds recited, and interviewers who run these rounds weekly now screen for that tell. The clock above is a time budget. It never appears as a heading in any case study on this page, and Module 1 teaches when to abandon it.

!!! danger "When to break this clock"

    The measurement that justifies following the clock: you have run one timed self-mock and finished with a system on the board, an evaluation plan stated, and time left. Below that, the clock is scaffolding you still need. Above it, treat it as advisory. Three situations specific to this round where following it is the wrong move:

    **1. The interviewer wants the model, not the system.** If minute four is "what features would you use, and why those?", you are in a modelling conversation wearing a design title. Compress the whole serving path into three boxes and ninety seconds, then spend the round on features, label definition and evaluation. Drawing a message queue while they are asking about feature interactions is the fastest way to look like you are running a script.

    **2. There is no label yet.** Some prompts have no naturally occurring outcome to learn from: a brand new abuse category, a product with no traffic, a task whose truth is known only months later. When that is the case, the labelling and bootstrap plan is the interview.

    Replace the estimation and v0 phases with four questions: what is the label exactly, who or what produces it, how long until you have enough, and what ships in the meantime. A serving diagram for a model that cannot be trained yet is a diagram of nothing.

    **3. The system already exists, or the prompt is a platform.** "Make this daily-batch recommender fresher without doubling cost", or "design a feature store", both start from a running world, so there is no greenfield v0 to draw. The minutes go to the current design's measured pain, the smallest change that removes it, migration order, dual computation of features, shadow scoring and rollback. Drawing a greenfield architecture for a system already in production is the most common way to fail a staff ML round.

    A fourth, quieter one: read who is in the room. An applied scientist will follow you into calibration and evaluation and lose interest in the queue; an infrastructure engineer will do the opposite. Both are asking a fair question. Rebalance the two deep dives towards whichever one they lean into, and say out loud that you are doing it.

## Module 1. Scope and honesty: which ML rounds this trains for

**Time: 20 to 30 minutes reading, 10 to 15 minutes practice.**

### 1a. Predict first

Here is a real-shaped onsite schedule, copied from the kind of confirmation email a recruiter sends. Read the five round descriptions before you read anything else on this page.

| Round | What the confirmation email says |
|---|---|
| R1 | "Coding: implement k-means from scratch, then discuss how you would choose k." |
| R2 | "ML Design: design the system that decides which of 200 million videos a user sees next." |
| R3 | "Applied ML: here is a churn dataset in a notebook. Improve the metric and narrate your reasoning." |
| R4 | "ML Depth: our ranking model gained 2 points of offline AUC and the online test was flat. Explain." |
| R5 | "Platform: design a feature store used by 40 teams." |

??? note "Show answer"

    Prediction asked for: which of these five does this course train, which does it partly train, and which belong to a different rubric entirely?

    | Round | Rubric it is scored against | This course |
    |---|---|---|
    | R1 | Algorithm implementation and numerical reasoning | No |
    | R2 | End-to-end ML system design | Yes, this is the centre |
    | R3 | Applied modelling and error analysis on a fixed dataset | No |
    | R4 | Evaluation reasoning: offline to online gap | Yes, in the evaluation modules |
    | R5 | ML infrastructure design | Yes, and almost no paid course covers it |

    Two of five. That ratio is the honest reason this page opens with scope. A candidate who prepares only ML system design and walks into R1 and R3 has prepared 40 percent of the loop.

    The tell for R3 is that the data is already given to you. When the dataset exists, nobody is asking you to design a data pipeline; they are asking you to model. The tell for R2 and R5 is that no data exists yet and the deliverable is a diagram plus a set of numbers.

### The six rounds that share the phrase "machine learning interview"

Six distinct rounds sit behind that phrase. They score different artefacts, and preparing for the wrong one is the most expensive mistake on offer because it costs weeks.

| Round | The artefact the interviewer wants at the end | What earns a strong rating | This course |
|---|---|---|---|
| ML system design | A diagram of a data, training and serving loop, plus the numbers that forced it | Deriving the design from stated constraints, naming what breaks next | Fully |
| ML platform or infrastructure design | A component with an interface, a consistency story and an operations story | Point-in-time correctness, versioning, multi-tenant blast radius | Fully |
| ML depth or "ML breadth" | Spoken mechanism: how a technique works and when it fails | Failure conditions, not definitions | Partly |
| Applied modelling on a dataset | A better metric on a given dataset, narrated | Error analysis, leakage detection, honest validation | No |
| ML coding | Working code for an algorithm or a data transform | Correctness, complexity, edge cases | No |
| Research or paper deep dive | A defence of work you did | Novelty, ablation reasoning, honest limitations | No |

**How to tell inside 90 seconds.** Listen for what already exists in the prompt.

- Nothing exists yet, and the noun is a product surface ("feed", "search", "recommendations"): ML system design.
- Nothing exists yet, and the noun is a component ("feature store", "experimentation platform", "model registry"): ML platform design.
- A dataset exists: applied modelling.
- A result exists and it is confusing: ML depth, usually on evaluation.
- A function signature exists: ML coding.

### Why a modelling interview is not this round

The single most common preparation error is treating an ML design round as a longer modelling round. The two rounds have different deliverables, so they have different failure modes.

| Dimension | Modelling round | ML system design round |
|---|---|---|
| What is given | A dataset | A business objective and nothing else |
| The hard step | Choosing and tuning a model | Choosing what to predict, and what a label even is |
| Where minutes go | Features, validation split, model family | Framing, sizing, serving path, feedback loop |
| What a wrong answer looks like | Overfitting, leakage, wrong validation | A pipeline with no latency budget and no way to know it works |
| Model architecture is worth | Most of the score | Roughly 5 minutes of 45 |
| The number that dominates | Cross-validated metric | Requests per second, candidate count, label delay |

!!! warning "The single most common way this round is failed"

    Spending 15 of 45 minutes on model architecture. In an ML design round the model is one box. The interviewer is scoring the eight boxes around it: where labels come from, how features are computed twice without disagreeing, how 200 million candidates become 10, and how you would learn that the system got worse.

    If you catch yourself describing attention heads, you are in the wrong round. Say so out loud and move: "I will pick a model family in a minute, but the harder question here is where the labels come from, so let me do that first."

### The routing diagram

```
        first line of the prompt
                  |
     +------------+------------+
     |            |            |
 nothing      a dataset    a result
 exists yet   is given     is given
     |            |            |
     v            v            v
+----------+ +----------+ +-----------+
| design   | | applied  | | ML depth  |
| round    | | modelling| | on eval   |
+----------+ +----------+ +-----------+
     |
 product noun or component noun
     |
  +--+-------------+
  |                |
  v                v
+----------+  +-------------+
| product  |  | ML platform |
| ML design|  | design      |
+----------+  +-------------+

Figure routing the first line of an ML round to one of five rubrics
```

### 1d. Worked example

The email says: "Onsite for Machine Learning Engineer, Trust and Safety. Four rounds: ML Design (60 min), ML Depth (45 min), Coding (45 min), Behavioural (45 min). The ML Design interviewer is a tech lead on the enforcement platform."

**Block 1. PURPOSE: convert the round list into a list of deliverables, because I prepare deliverables, not topics.**

Four rounds, three technical. The ML Design round wants a diagram and numbers. The ML Depth round wants spoken mechanism. Coding wants working code. I write those three words at the top of my plan: diagram, mechanism, code. Everything I study has to serve one of them.

The team name is the strongest signal in the email. Trust and Safety means the domain is classification under adversarial pressure with expensive human review, which tells me the ML Design prompt is very likely harmful content, spam, or account takeover.

**The error most readers make here** is reading "Machine Learning Engineer" and preparing recommendation systems because that is what most study material covers. The team name overrides the job title every time.

!!! note "Say it before you read on"

    Name one thing about a Trust and Safety ranking problem that is structurally different from a recommendation ranking problem. Say it out loud before continuing.

**Block 2. PURPOSE: decide what I will deliberately not prepare, because preparation is subtraction.**

Given enforcement, I drop three things entirely: two-tower retrieval at billion scale, exploration and bandits, and personalisation. Enforcement is a precision and recall economics problem over a stream, not a candidate generation problem.

That frees roughly all of my ML Design time for four things: label sources including human review capacity, precision versus recall costed in currency, appeal and feedback loops, and adversarial drift.

**The error most readers make here** is additive preparation. They add Trust and Safety material on top of a general syllabus and arrive having half-learned everything.

**Block 3. PURPOSE: buy information cheaply before the round, because two sentences of reply beat two hours of inference.**

I send the recruiter one message: "For the ML Design round, is the prompt usually a greenfield product system, or an existing pipeline being extended? And does the interviewer expect the modelling layer, the platform layer, or both?"

Recruiters answer this. If the answer is "platform layer", my preparation moves almost entirely to module 5 and the labelling pipeline material, and the case study I rehearse changes.

!!! note "Say it before you read on"

    Which single sentence in a recruiter's reply would most change how you spend the remaining preparation hours? Answer before reading block 4.

**Block 4. PURPOSE: rehearse the opening, because it is the only fully predictable 40 seconds of the round.**

I write and say out loud: "I want to spend the first six or seven minutes on what we are optimising and where labels come from, because in enforcement those two decide the architecture. Then I will size the stream, draw the smallest thing that works, and break it. If you would rather I go deep on one piece instead, say so now and I will restructure."

**The error most readers make here** is improvising the opening, then spending minute three drawing boxes nobody asked for.

**Result.** The reply was: "Greenfield product prompt, but she always asks how you would operate it." That moved the last two hours of preparation from architecture drills to monitoring, alert thresholds and the reviewer queue, which is exactly where the round went.

### 1e. Fade the scaffold

**Faded example 1.** The email says: "Onsite for Machine Learning Engineer, Ads Ranking. Rounds: ML Design (60), ML Depth (60), Coding (45). The ML Depth interviewer works on auction infrastructure." Blocks 1 to 3 are given. Produce block 4 yourself.

- Block 1: deliverables are a diagram, spoken mechanism, and code. The team name says ads ranking, so the design prompt is click or conversion prediction.
- Block 2: I drop content moderation, recommendation diversity and cold-start-heavy material. I keep calibration, delayed conversions and cost per thousand predictions.
- Block 3: I ask whether the depth round is model-side or serving-side, because "auction infrastructure" could mean either.

??? note "Show answer"

    A defensible block 4, shaped by the fact that an auction consumes a probability rather than a ranking:

    "Before I draw anything I want to pin down what the model's output is used for, because in an ads system the score feeds a bid, so it has to be a calibrated probability rather than a rank. That changes the loss, the evaluation metric and how I would monitor it. I will spend six minutes on framing and labels, then size the request path, then draw the funnel. Tell me if you want the auction side rather than the model side."

    The specific thing that earns credit is naming calibration in the first minute. It is the fastest available signal that you have seen an ads system rather than read about ranking in general.

**Faded example 2.** The email says: "Virtual onsite, Machine Learning Engineer, Search Infrastructure. Rounds: ML Design (60), Systems Design (60), Coding (45)." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: three deliverables, and note that one of the three rounds is plain distributed systems design, not ML.
- Block 2: I drop nothing from the ML side yet, because "Search Infrastructure" covers both retrieval quality and index serving. I do subtract the whole recommendation-personalisation stack.

??? note "Show answer"

    Block 3: the cheap purchase here is about the split, not the topic. I ask: "Is the Systems Design round scored on the same rubric as a backend round, or is it an ML platform round wearing that name?" If it is a plain backend round, I spend an evening on the sibling course, [Modern System Design Interview](modern-system-design.md), rather than on more ML material. That single question can redirect a quarter of the remaining hours.

    Block 4: the opening has to work for a prompt that could be quality-side or infrastructure-side. "Search covers two very different design problems: relevance quality, and index serving at scale. I will start by stating which one I think you want and why, then you can redirect me." Performing the selection out loud is the point, because in a search round the selection is half the difficulty.

### 1f. Check yourself

**Q1.** An interviewer opens with: "We have a model in production that flags risky transactions. Recall is 60 percent and the review team is saturated. What would you do?" Which round are you in, and what is the first thing you change about your plan?

??? note "Show answer"

    A brownfield ML design round, not a modelling round. Two things exist already: a model and a capacity-constrained human process. The first change is that every proposal now needs a review-capacity story. Raising recall without changing the queue makes the situation worse, because the reviewers are the bottleneck, not the model.

    The strongest opening move is to ask for two numbers: reviews per day the team can actually complete, and the cost of a missed fraud case versus the cost of a wrong flag. Both change the answer, and neither is about model architecture.

**Q2.** You are 20 minutes into what you believed was an ML design round and the interviewer says "assume the model is a black box that returns a score between 0 and 1". What just happened, and what do you do?

??? note "Show answer"

    The interviewer removed modelling from scope and made it a systems and product round: thresholds, serving, feedback, evaluation, operations. This is a gift, not a restriction, and it is one of the clearest signals available.

    You do two things. First, say the reframe out loud so it is on record: "Then the interesting decisions are threshold selection, what happens on each side of the threshold, and how we would detect that the score distribution moved." Second, delete your prepared model comparison. Continuing to volunteer architecture after that sentence reads as not listening.

### 1g. When not to use this

Round triage is a preparation tool. It is over-applied far more often than under-applied.

| Question | Answer |
|---|---|
| The measurement that justifies doing triage | You have two or more loops booked, or you cannot name the rubric of any round from its description |
| Threshold below which it is over-engineering | One loop, with a written round list that already names each round's format and length. Triage adds nothing to that |
| The cheaper alternative | One message to the recruiter naming the two things you cannot infer. Reply time is usually a day and the information is exact |
| Failure mode of over-applying it | Spending study hours on meta-strategy rather than on modules 2 to 8. The whole triage exercise is worth about 30 minutes |

---
## Module 2. Problem framing: from a business objective to a trainable one

**Time: 35 to 45 minutes reading, 25 to 35 minutes practice.**

### 2a. Predict first

A product manager says: "We want people to spend more time watching." An engineer proposes ranking videos by predicted watch time in seconds. Here are three videos with their measured completion rates.

| Video | Duration | Completion rate | Expected watch time |
|---|---|---|---|
| A | 15 s | 0.80 | ? |
| B | 60 s | 0.45 | ? |
| C | 300 s | 0.12 | ? |

??? note "Show answer"

    Prediction asked for: fill in the third column, then say which video this objective promotes and what the creator ecosystem does in response.

    Expected watch time is duration times completion rate: A is 12 s, B is 27 s, C is 36 s. Ranking by expected watch time puts C first, then B, then A, even though C is the video 88 percent of viewers abandon.

    The mechanism: completion rate falls with duration, but it falls more slowly than duration rises. So the product of the two grows with duration across this range. The objective therefore contains a structural preference for long content that nobody chose and nobody wrote down.

    What creators do in response is the second half of the answer. Within one or two quarters, uploads get longer, intros get padded, and the median item in the catalogue shifts. The objective did not just rank the catalogue. It changed it.

### The chain from money to loss function

Every ML design round contains one translation, done four times, and each step loses information. Naming the chain out loud in the first five minutes is one of the cheapest strong signals available.

| Layer | Example | Who owns it | Measured how |
|---|---|---|---|
| Business objective | Revenue, retention, marketplace liquidity | The company | Quarterly, noisy, unattributable |
| Product decision metric | Sessions per user per day | The product team | Weekly cohorts |
| Experiment metric | Day-7 retention in an A/B test | The experiment platform | Per test, powered, with guardrails |
| Trainable target | Probability the user watches past 10 s | You | Per example, from logs |
| Loss | Binary cross entropy, weighted | You | Per batch |

**Two rules that come out of the chain.**

- Each step must be defensible in one sentence. If you cannot say why the trainable target moves the experiment metric, the round is already lost and no amount of architecture recovers it.
- The gap between layers is where gaming lives. The model optimises layer four exactly. It optimises layer one only to the extent that your translation was honest.

### Choosing a trainable target, and how each one gets gamed

For any product objective there are usually four or five candidate targets. Enumerating them out loud, with their failure direction, is worth more interview credit than picking the right one silently.

| Candidate target | What it buys | How it degrades | Cheapest guardrail |
|---|---|---|---|
| Click or tap | Dense labels, fast feedback, easy to log | Rewards misleading titles and thumbnails | Post-click dwell, and click-then-immediate-back rate |
| Expected watch time | Correlates with the stated objective | Structural bias toward long items, shown above | Median item duration in the served slate |
| Completion rate | Removes the duration bias | Structural bias toward very short items | Median item duration, again, in the other direction |
| Explicit rating or like | Reflects stated preference | Sparse, roughly 1 to 2 percent of impressions carry one, and biased to extremes | Coverage: fraction of impressions with any explicit signal |
| Next-day return | Closest to the business objective | Delayed by a day, extremely noisy per item, hard to attribute | Not a per-item target. Use it as an experiment metric instead |

!!! tip "The sentence that gets this right in an interview"

    "I would not train on the business metric directly. Day-7 retention is not attributable to a single impression, so as a per-item label it is almost pure noise. I will train on a short-horizon proxy, and put retention in the guardrail set so that if the proxy and the objective diverge, the experiment tells us."

### Guardrail metrics, and how much leverage they carry

A guardrail is a metric you are not trying to improve, which can veto a launch. Guardrails are usually treated as a checklist item. They deserve arithmetic, because a small guardrail move can outweigh a large primary move.

Take a steady-state retention model. If a product acquires **n** new users per day and retains a fraction **r** of yesterday's users each day, the steady-state daily active user count is:

```
DAU_steady = n / (1 - r)

r = 0.950  ->  DAU_steady = n * 20.0
r = 0.945  ->  DAU_steady = n * 18.2

Figure the DAU multiplier is very sensitive near high retention
```

A drop of half a percentage point in daily retention, from 0.950 to 0.945, cuts steady-state DAU by 1 minus 18.2 over 20.0, which is 9.1 percent. That is the whole point of a guardrail: a change that improves revenue per session by 45 percent while costing half a point of retention nets out to roughly 1.455 times 0.909, or a 32 percent gain, not a 45 percent one, and the retention loss keeps compounding after the experiment ends while the revenue gain does not.

**What this means for the design.** Guardrails belong on the metric that has the largest multiplier, not on the metric that is easiest to compute. In a consumer product that is almost always a retention or return-frequency metric. Put session-level metrics in the primary set and cohort-level metrics in the guardrail set.

### Metric cannibalisation between competing goals

Cannibalisation happens when two objectives share a fixed resource. In ranking the shared resource is nearly always **slots**: positions in a finite list. Every slot given to one objective is taken from another.

| Shared resource | Objective A | Objective B | The trade you are actually making |
|---|---|---|---|
| Feed slots | Engagement | Ads revenue | Each ad slot removes one organic slot |
| Feed slots | Relevance | Diversity or exploration | Each exploratory slot costs its expected value |
| Feed slots | Established creators | New creators | Cold-start supply versus short-term engagement |
| Review capacity | Precision of enforcement | Recall of enforcement | Each review spent on a borderline case is not spent on a clear one |
| Latency budget | Model quality | Candidate count | Milliseconds spent per item versus items considered |

The design consequence is that a multi-objective system needs an explicit combination rule, and the weights in that rule are a **product decision measured online**, not a hyperparameter tuned offline. There is no offline dataset that tells you how much retention a dollar of ad revenue is worth.

### The framing diagram

```
+------------------+     +--------------------+
| business object  | --> | experiment metric  |
| revenue, retain  |     | day-7 retention    |
+------------------+     +--------------------+
                              |         ^
                     must be  |         | guardrail veto
                    justified |         |
                              v         |
                     +--------------------+
                     | trainable target   |
                     | p(watch past 10s)  |
                     +--------------------+
                              |
                              v
                     +--------------------+
                     | loss and weights   |
                     +--------------------+

Figure the four-layer chain, with the guardrail as a veto edge
```

### 2d. Worked example

**Prompt.** "Our short-video feed shows 30 items per session. Leadership wants daily time spent up 10 percent. Frame the ML problem."

**Block 1. PURPOSE: pin the objective to a decision, because an objective that decides nothing cannot be optimised.**

I ask one question first: is time spent the goal, or is it a proxy for something else? The answer I get is that time spent is the proxy and the real target is retention, because ad revenue scales with DAU.

That reply is the most valuable thing I will hear in the round. It tells me time spent is a **product decision metric**, not the business objective, so it is legitimate to trade a little of it for retention. Without that sentence I would have optimised a proxy to the point where it hurt the thing it proxies for.

**The error most readers make here** is accepting the stated metric as the objective. Leadership states proxies constantly. Asking what the proxy is for takes 20 seconds and reframes the entire round.

!!! note "Say it before you read on"

    Say why "increase time spent by 10 percent" is not yet enough to write a loss function. Name the two missing pieces before continuing.

**Block 2. PURPOSE: enumerate trainable targets and eliminate on structural bias, not on taste.**

I write four candidates on the board and evaluate each against the duration arithmetic from the pre-question.

| Target | Structural bias | Verdict |
|---|---|---|
| p(click) | Toward misleading thumbnails | Keep as one head, not as the objective |
| E[watch seconds] | Toward long items, by the 12 / 27 / 36 arithmetic | Reject as sole objective |
| p(completion) | Toward very short items | Reject as sole objective |
| p(watch past 10 s) | Mild, and 10 s is short relative to the shortest content | Keep as the primary head |

The 10 second threshold is a **choice with a cost**, and I say so: it makes a 12 second video and a 12 minute video equivalent once both pass 10 seconds. That is why it cannot be the only head.

So the answer is not one target. It is a small set of heads with a combination rule: **p(watch past 10 s)**, **p(completion)** and **p(explicit negative)**, where the last is the signal that the user hid, skipped fast, or reported the item.

**The error most readers make here** is picking a single target and defending it. The interviewer is not testing whether you pick the same one they would. They are testing whether you can name the failure direction of each option.

**Block 3. PURPOSE: derive guardrails from the failure directions I just named, so the guardrail set is not a generic checklist.**

Each rejected or partially accepted target has a known failure direction, and each failure direction gets a guardrail that would catch it.

| Failure I am worried about | Guardrail metric | Why it catches it |
|---|---|---|
| Slate drifts long | Median duration of served items | Moves immediately, before retention does |
| Slate drifts short and shallow | Sessions per user per day, and items per session | A shallower slate raises items per session while time falls |
| Clickbait rises | Rate of watch under 3 s after a tap | Directly measures the misleading-thumbnail failure |
| Long-run harm | Day-7 and day-28 retention | Has the 9 percent multiplier derived above |
| Creator supply collapses | Share of impressions to items under 30 days old | Detects a feedback loop toward established items |

!!! note "Say it before you read on"

    Two of those five guardrails move within hours and three take a week or more. Say which are which, and why that matters for how you would run the experiment.

**The error most readers make here** is listing guardrails that sound responsible but are not caused by anything in the design. A guardrail exists to catch a specific failure you have already named. If you cannot name the failure, delete the guardrail.

**Block 4. PURPOSE: write the combination rule and state how its weights get set, because unstated weights are where multi-objective designs quietly fail.**

I propose a weighted product on the ranking score, because a product makes a near-zero on any head able to veto the item, which a weighted sum cannot do:

```
score = p10 ^ a  *  pcomp ^ b  *  (1 - pneg) ^ c

p10    = probability of watching past 10 seconds
pcomp  = probability of completing
pneg   = probability of hide, report or fast skip
a,b,c  = exponents set by online experiment, not offline tuning

Figure the combination rule, with one exponent per objective head
```

Then I say the part that earns the credit: a, b and c cannot be chosen offline. There is no offline dataset containing the exchange rate between completion and reported content. They are set by running a small grid of exponent settings as an online experiment and reading the guardrails, and they are revisited whenever the product changes.

**The error most readers make here** is treating the combination weights as hyperparameters and saying they would tune them on a validation set. That answer reveals the modelling-round habit: it optimises the proxy, which is exactly the thing the weights exist to trade off.

**Result.** The framing I deliver in six minutes is: three heads, a product combination rule, five guardrails each tied to a named failure, and one sentence saying the exponents are a product decision measured online. No architecture has been mentioned yet, and that is correct.

### 2e. Fade the scaffold

**Faded example 1.** Prompt: "We run a marketplace. Sellers list items, buyers search. Leadership wants gross merchandise value up." Blocks 1 to 3 are given. Produce block 4 yourself.

- Block 1: I ask what GMV is a proxy for and learn it is a proxy for take rate revenue, and that repeat purchase rate is the real long-run target.
- Block 2: candidate targets are p(click), p(add to cart), p(purchase), and E[order value]. E[order value] has a structural bias toward expensive items exactly as expected watch time biased toward long ones.
- Block 3: guardrails are median listed price of shown items, purchase rate for new sellers, return and refund rate, and 30-day repeat purchase rate.

??? note "Show answer"

    A defensible block 4: the ranking score is a product of a purchase-probability head and a value term, with the value term deliberately compressed so it cannot dominate.

    ```
    score = p(purchase) ^ a  *  value_norm ^ b  *  (1 - p(return)) ^ c

    value_norm = clip(order_value / median_price_in_category, 0.5, 2.0)

    Figure a marketplace ranking rule with the value term clipped
    ```

    The clip is the load-bearing decision and is what a strong answer says out loud. Without it, one item priced 100 times the category median outranks everything, and the failure mode is a search page of items nobody can afford. Clipping to a two-times band means the value head can reorder within a plausible range but cannot override purchase probability.

    Weights a, b and c are set online. The refund head enters as a veto term because a returned purchase has negative value once shipping and handling are counted, which a plain revenue objective would never see.

**Faded example 2.** Prompt: "Design the objective for a notifications system that decides which push notifications to send and when." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: the stated objective is sessions per day. On asking, it turns out to be a proxy for retention, and there is a known history of notification fatigue causing uninstalls.
- Block 2: candidate targets are p(open), p(open and then engage for 30 s), and p(disable notifications or uninstall within 7 days). The last is a rare negative event: assume roughly 1 in 2,000 notifications, which makes it 500 times rarer than an open.

??? note "Show answer"

    Block 3, guardrails: notification disable rate, uninstall rate within 7 days of a send, opens per notification sent (not per user, so volume increases cannot hide a falling rate), and share of users receiving more than three notifications per day.

    Block 4, the rule. Notifications are unusual because the decision is not only which item but **whether to send at all**, so the design needs a threshold, not just a ranking:

    ```
    send if:  p(engage30) * value  >  p(disable) * cost_disable

    cost_disable = lifetime value of a user, in the same units
    Figure the send decision as an expected-value threshold
    ```

    Two things a strong answer adds. First, cost_disable is enormous relative to the value of one session, so the threshold is high and most candidate notifications are never sent. Second, p(disable) is rare and delayed, so it cannot be trained on the same data or the same cadence as p(engage30): it needs a longer window, heavy class weighting, and calibration checked against the observed base rate. Module 4 covers the label delay and imbalance mechanics this needs.

### 2f. Check yourself

**Q1.** A colleague proposes training a feed ranker directly on 7-day retention as the per-impression label. Give the strongest technical objection in two sentences.

??? note "Show answer"

    Retention is a user-level, week-scale outcome, so attributing it to one of the roughly 200 impressions that user saw that week assigns almost all of the label's variance to noise. The model will learn user-level propensity, which it already gets from user features, and learn nearly nothing about which item to show, so it will rank items by whether the viewer is a retained kind of person rather than by whether the item is good.

    The constructive follow-up: keep retention as an experiment metric and guardrail, and train on a short-horizon proxy whose correlation with retention you verify across past experiments.

**Q2.** Your primary metric is up 3 percent and one guardrail, day-7 retention, is down 0.4 percentage points but is not statistically significant. What do you say?

??? note "Show answer"

    That the guardrail is under-powered, not clean. Retention moves are small and slow, so an experiment powered to detect a 3 percent primary move is usually nowhere near powered to detect a 0.4 point retention move. Reporting "not significant" as "no harm" mistakes absence of evidence for evidence of absence.

    Using the multiplier derived above, 0.4 points off a 0.95 base is roughly a 7 percent steady-state DAU effect, which would dwarf the 3 percent primary gain. The correct action is to extend the experiment or run a longer holdback for the retention read specifically, and to say up front what effect size the test could actually detect.

### 2g. When not to use this

Multi-head objectives with combination rules are a real cost: more heads, more labels, more calibration, more experiment surface, and a weight vector that has to be maintained forever.

| Question | Answer |
|---|---|
| The measurement that justifies multiple heads | A measured conflict: an offline or online result where improving the single objective moved a guardrail in the wrong direction by more than its noise band |
| Threshold below which it is over-engineering | One surface, one stakeholder, and no shared scarce resource. If nothing is being traded, there is nothing to weight. A single well-chosen head plus guardrails is correct |
| The cheaper alternative | One head, plus hard business rules as filters. A rule such as "never show more than two items from the same creator in ten slots" achieves diversity at zero modelling cost and is trivially auditable |
| The failure mode of over-applying it | Five heads whose exponents nobody has re-tuned in a year, where two heads are 0.98 correlated and the extra ones only add serving latency and calibration drift |

Also skip the whole framing exercise when the target is imposed from outside. Regulatory or contractual detection tasks, such as flagging transactions a regulator defines, come with the label definition already written. There, framing is a 60 second confirmation and the interesting work is entirely in modules 4 and 7.

---
## Module 3. Estimation for ML systems

**Time: 45 to 60 minutes reading, 35 to 50 minutes practice.**

Capacity estimation in a general system design round sizes storage and bandwidth. In an ML round it does something more useful: it eliminates model families. This module derives six numbers, and every one of them rules an option out. If a number rules nothing out, delete it from your answer.

### 3a. Predict first

Here is the assumption block for a short-video feed. Nothing else is given.

| Symbol | Assumption | Value |
|---|---|---|
| A1 | Daily active users | 50,000,000 |
| A2 | Sessions per user per day | 4 |
| A3 | Feed requests per session | 6 |
| A5 | Candidates scored by the ranker per request | 500 |
| A6 | Peak to mean traffic ratio | 2.5 |
| A11 | Usable throughput of one rented accelerator | 100 TFLOP/s |

A colleague proposes running a 6-layer transformer cross-encoder, costing about 11 GFLOPs per item, over all 500 candidates.

??? note "Show answer"

    Prediction asked for: without doing the full derivation, which single quantity decides whether this is possible, and roughly what will it say?

    The deciding quantity is FLOPs per request against accelerator throughput, and it kills the proposal on latency before cost is even discussed.

    One request needs 500 times 11 GFLOPs, which is 5.5 TFLOPs. At 100 TFLOP/s that is 55 milliseconds of accelerator time for a single request, with the whole accelerator dedicated to it. A feed ranking stage typically gets 40 to 50 milliseconds of the end-to-end budget, so the design fails even if you buy unlimited hardware.

    That is the shape of ML estimation. The number did not size a disk. It removed a model family from the whiteboard in one line of arithmetic, which is exactly what the interviewer is watching for.

### The six numbers, and what each one eliminates

| # | Number | Typically eliminates |
|---|---|---|
| 1 | Peak requests per second | Synchronous designs, single-region designs |
| 2 | Item scores per second | Model families, by FLOPs per item |
| 3 | Index footprint in bytes | Single-host retrieval, or full-precision vectors |
| 4 | Training rows and bytes per day | Retention windows, and full-log training |
| 5 | Retrain cadence in days | Batch-only feature pipelines |
| 6 | Cost per thousand predictions | Whichever of the above survived on technical grounds |

Work them in that order. Each one consumes the previous.

### Number 1: peak request rate

```
requests/day = A1 * A2 * A3 = 50e6 * 4 * 6 = 1.2e9
mean rps     = 1.2e9 / 86,400                = 13,900
peak rps     = 13,900 * A6                   = 34,750, call it 35,000

Figure the request-rate derivation, three lines, memorisable
```

**What it eliminates.** 35,000 requests per second is comfortably beyond one host but nowhere near the scale that forces exotic infrastructure. It rules out a design where the model is called once per user per day and cached, because the request rate says users come back within a session and expect the feed to change. It does not yet rule out any model.

### Number 2: item scores per second, and the model-family cut

This is the number that does the most work in an ML round, and almost nobody computes it.

```
item-scores/s at peak = peak rps * A5 = 35,000 * 500 = 17,500,000

Figure the ranking stage scores 17.5 million items every second
```

Now cost each candidate model family in units per item. Two derivations, both from stated assumptions.

**Gradient boosted trees.** Assume 500 trees of depth 8, so 4,000 node visits per item. Assume 2.5 nanoseconds per node visit, on the grounds that the whole ensemble is tens of megabytes and stays resident in last-level cache, so each visit is a cache hit plus a compare and a branch.

```
per item      = 4,000 * 2.5 ns          = 10 microseconds
per core      = 1 / 10 us               = 100,000 items/s
cores at peak = 17.5e6 / 100,000        = 175
at 50% target utilisation               = 350 cores
hosts of 16 vCPU                        = 22

Figure the tree ensemble needs 22 ordinary hosts at peak
```

**Transformer cross-encoder.** A 6-layer model with hidden size 768 has about 12 times 768 squared parameters per layer, which is 7.1 million, so 42.5 million total. Forward-pass FLOPs are about 2 times parameters times tokens. At 128 tokens per item that is 10.9 GFLOPs, call it 11.

```
full slate   = 17.5e6 * 1.09e10 = 1.9e17 FLOP/s = 190 PFLOP/s
accelerators = 1.9e17 / 1e14    = 1,900
top-25 only  = 35,000 * 25 * 1.09e10 = 9.5e15 = 95 accelerators

Figure the same model is 20x cheaper when restricted to the top 25
```

**What it eliminates.** Three things at once, and this is the payoff for doing the arithmetic.

- A cross-encoder over the full 500 candidates is dead on latency (55 ms per request, shown above) and dead again on cost.
- A cross-encoder over the top 25 survives: 2.7 milliseconds of accelerator time per request and 95 accelerators. That is the arithmetic case for a re-ranking stage, derived rather than asserted.
- Trees over 500 candidates cost 22 hosts, which is small enough that ranking cost stops being an architectural constraint and the design conversation can move on.

### Number 3: index footprint

Assume a retrievable catalogue of 100 million items and 256-dimensional embeddings. Compute bytes per vector for each storage choice, then multiply.

| Storage choice | Bytes per vector | Total for 100M | Fits on one 128 GB host |
|---|---|---|---|
| float32, no graph | 256 * 4 = 1,024 | 102 GB | Barely, with nothing left over |
| float32 plus HNSW links, M = 32 | 1,024 + 64 * 4 = 1,280 | 128 GB | No |
| float16 plus HNSW links | 512 + 256 = 768 | 77 GB | Yes, with headroom |
| Product quantized, 64 bytes, plus links | 64 + 256 = 320 | 32 GB | Yes, comfortably |
| Product quantized, 64 bytes, IVF only | 64 + list overhead | 7 GB | Yes, trivially |

The HNSW link cost is derived, not quoted: the bottom layer of the graph stores up to 2M neighbour identifiers per node, so at M = 32 that is 64 identifiers of 4 bytes each, which is 256 bytes per vector regardless of dimension.

**What it eliminates.** Full-precision vectors plus a graph do not fit on a single ordinary host, so the design must either shard the index, compress the vectors, or both. Naming that fork from one multiplication is worth more than any statement about which index library you prefer. Module 7 chooses between the branches.

### Number 4: training rows and bytes per day

```
impressions/day = requests/day * items shown = 1.2e9 * 10 = 1.2e10
positives at 4% = 4.8e8
negatives       = 1.152e10
negatives kept at 2% sampling = 2.3e8
rows/day        = 4.8e8 + 2.3e8 = 7.1e8

Figure negative downsampling turns 12 billion rows into 710 million
```

Bytes follow. Assume 1 kilobyte per serialized row: roughly 200 features plus identifiers, context and timestamps.

```
raw          = 7.1e8 * 1 KB           = 710 GB/day
columnar, 3x = 237 GB/day
90 day window                          = 21 TB

Figure the training corpus at a 90-day retention window
```

**What it eliminates.** Training on every logged impression. 12 billion rows a day is 4.4 trillion a year, and the model gains nothing from the 96 percent of negatives that are trivially negative. Downsampling is therefore not an optimisation, it is a design requirement, and it carries an obligation: **the model's output probabilities are now wrong and must be corrected**. Module 4 derives the correction.

It also eliminates arguments about training compute, at least for trees. Histogram-based boosting costs roughly rows times features per tree:

```
ops = 7.1e8 rows * 200 features * 500 trees = 7.1e13
on 32 cores at 5e9 ops/s each                = 444 seconds floor
budget 3x to 10x above the floor             = 25 to 75 minutes
cost at $0.04 per vCPU-hour                  = about $1.30 per run

Figure training cost for this model family is operationally free
```

At $1.30 a run, training cost cannot decide anything, so spending interview minutes on it is a mistake. For a deep model the binding constraint is usually not FLOPs either: reading 237 GB three times at 2 GB/s sustained is about 6 minutes of pure input-pipeline time, which is the number that decides whether you need a cached or sharded reader.

### Number 5: retrain cadence

Cadence is usually asserted ("we retrain daily") and almost never derived. There are two honest derivations, and which one applies depends on whether the model consumes item identities or item counters.

**Derivation A, from content churn.** Assume the catalogue grows 1 percent per day, so 1 million new items daily, and assume items younger than 7 days take 25 percent of impressions because the product promotes new content.

```
cold impression share after k days = 25% * (k / 7)
tolerance: keep it under 5%
k = 7 * (5 / 25) = 1.4 days  ->  retrain daily

Figure cadence forced by how fast the catalogue turns over
```

**Derivation B, from measured decay.** Score today's holdout with the model trained d days ago and record the metric. Assume the backtest gives an AUC decay of 0.0008 per day, and assume the smallest gain your experiment platform can detect and launch is 0.002 AUC.

```
retrain when accumulated decay = smallest detectable gain
d = 0.002 / 0.0008 = 2.5 days  ->  round to daily

Figure cadence forced by measured staleness, not by habit
```

**What it eliminates.** A daily cadence with a batch-only feature pipeline cannot serve an item created three hours ago, because that item has no features until tomorrow's job runs. So either the cadence goes sub-hourly, which is expensive, or item-level engagement features move to a streaming pipeline while the model itself stays daily. That fork is the entire reason module 5 exists, and it was produced by one division.

### Number 6: cost per thousand predictions

Define a prediction as one ranked request returning 10 items, so the unit matches the product.

| Component | Derivation | Cost per day |
|---|---|---|
| Ranking, trees | 350 vCPU * $0.04 * 24 | $336 |
| Feature fetch and retrieval | assume 2x the ranking cost, since both are network and memory bound | $672 |
| Re-ranking, 95 accelerators | 95 * $2 * 24 | $4,560 |
| Training | one run per day | $2 |
| **Total online** | | **$5,570** |

```
cost per 1,000 requests without re-ranking
  = ($336 + $672) / (1.2e9 / 1,000) = $0.00084

cost per 1,000 requests with re-ranking
  = $5,570 / 1.2e6 = $0.0046

Figure the re-ranking stage multiplies serving cost by about 5.5x
```

**What it eliminates.** Nothing yet, and saying so is the point. A 5.5x cost multiplier is only decisive next to revenue per request. State the comparison and hand the decision back: "Re-ranking the top 25 costs about 4.6 tenths of a cent per thousand feed requests. Whether that is worth it depends on revenue per thousand requests, which you would know better than me. If it is above a few cents, this is obviously worth testing; if it is under half a cent, it is not."

That sentence is what separates cost estimation from cost theatre.

### The one-page estimation sheet

```
+---------------------+   +----------------------+
| DAU, sessions,      |-->| peak requests/sec    |
| requests/session    |   | 35,000               |
+---------------------+   +----------------------+
                                     |
        +----------------------------+-------------+
        v                                          v
+----------------------+                +--------------------+
| item-scores/sec      |                | index footprint    |
| 17.5M -> model cut   |                | 128 GB -> shard    |
+----------------------+                +--------------------+
        |                                          |
        v                                          v
+----------------------+                +--------------------+
| rows/day 710M        |                | retrain cadence    |
| bytes/day 237 GB     |--------------->| 1.4 days -> daily  |
+----------------------+                +--------------------+
                    |
                    v
        +--------------------------+
        | cost per 1,000 preds     |
        | $0.00084 to $0.0046      |
        +--------------------------+

Figure the order the six numbers are derived, each feeding the next
```

### 3d. Worked example

**Prompt.** "We run product search over 400 million listings. We want semantic retrieval using 384-dimensional embeddings. Size it and tell me what infrastructure that implies."

**Block 1. PURPOSE: put the assumptions on the board first, so the interviewer can correct them before I build on them.**

I write four lines: 400 million vectors, 384 dimensions, a target of 25 milliseconds for the retrieval stage, and a peak of 20,000 search queries per second (assumption: e-commerce search runs at lower request volume than feed scrolling because each query is deliberate).

Then I say the sentence that buys goodwill: "If any of those four are wrong by more than a factor of two, tell me now, because the conclusion changes."

**The error most readers make here** is starting the arithmetic before stating assumptions. When the interviewer corrects an assumption at minute 12, all the arithmetic after it is wasted, and the correction lands as a failure rather than as collaboration.

**Block 2. PURPOSE: compute the footprint and kill the options it cannot support.**

```
float32 vector        = 384 * 4        = 1,536 bytes
HNSW links at M = 16  = 32 * 4         =   128 bytes
per vector                             = 1,664 bytes
400M vectors                           = 666 GB

float16 variant       = 768 + 128      =   896 bytes -> 358 GB
PQ at 96 bytes        = 96 + 128       =   224 bytes ->  90 GB

Figure three storage choices for the same 400 million vectors
```

Three conclusions, each an elimination. 666 GB does not fit any ordinary host, so full precision plus a graph is out for a single node. 358 GB in float16 still does not fit a 256 GB host, so halving precision alone does not save the single-node design. 90 GB with product quantization does fit, with room for the process and the operating system.

!!! note "Say it before you read on"

    Before reading block 3, say what product quantization costs you, in one sentence, in terms an interviewer would accept.

**The error most readers make here** is stopping at "we would use a vector database". The footprint is what decides how many machines that database needs and what it can promise, and a named product does not answer the question.

**Block 3. PURPOSE: choose between compression and sharding by naming the price of each, not by preference.**

Two viable designs remain, and they fail differently.

| Design | Machines | What it costs | Where it bites |
|---|---|---|---|
| float16, sharded 4 ways, 2 replicas | 8 hosts of 128 GB | Every query fans out to 4 shards and merges | Tail latency is the max of 4 shards, not the mean |
| PQ to 96 bytes, single shard, 3 replicas | 3 hosts of 128 GB | Approximate distances, so recall drops | Needs an exact re-scoring pass to recover recall |

I choose PQ plus exact re-scoring, and I say why in one sentence: fan-out multiplies tail latency by the slowest shard, and at 20,000 queries per second I would rather pay a second local lookup than pay a four-way network fan-out on every query.

The recovery pass is concrete: retrieve 1,000 candidates by quantized distance, then re-score those 1,000 against their float16 vectors fetched from a local key-value store, and keep the top 200.

**The error most readers make here** is treating quantization as free. It trades recall for memory, and the trade is only acceptable when you have named the recovery mechanism and budgeted its time.

!!! note "Say it before you read on"

    Say roughly how long the exact re-scoring of 1,000 vectors should take, using only numbers already on this page. Then read block 4.

**Block 4. PURPOSE: verify the choice against the latency budget, because a design that fits in memory can still miss the clock.**

```
re-score 1,000 vectors, 384 dims, float16
  = 1,000 * 384 fused multiply-adds = 384,000 FLOPs
at 20 GFLOP/s per core with vector instructions
  = 19 microseconds of compute

fetch 1,000 vectors of 768 bytes from local memory
  = 768 KB, at 10 GB/s = 77 microseconds

total re-scoring cost: well under 0.2 ms of the 25 ms budget

Figure the recovery pass is negligible against the retrieval budget
```

The re-scoring pass is essentially free, which is exactly why it is the standard fix. What is not free is the quantized search itself and the network round trip, and those consume the remaining budget.

**Result.** The answer delivered in about five minutes: 400 million vectors at 384 dimensions is 666 GB uncompressed, which forces either sharding or compression. I compress to 96 bytes per vector, hold 90 GB on one node with three replicas for throughput and availability, retrieve 1,000 approximate candidates and re-score them exactly for under 0.2 milliseconds. Every number came from four stated assumptions.

### 3e. Fade the scaffold

**Faded example 1.** Prompt: "Size the ranking stage for a jobs marketplace. 8 million monthly active users, 30 percent visit on a given day, 2 sessions each, 3 result pages per session, 200 candidates ranked per page, peak-to-mean 3." Blocks 1 to 3 are given. Produce block 4 yourself.

- Block 1: assumptions are as stated, plus a ranking latency budget of 60 ms and a model of 300 trees at depth 6.
- Block 2: DAU is 8e6 times 0.30, which is 2.4 million. Requests per day are 2.4e6 times 2 times 3, which is 14.4 million. Mean rps is 14.4e6 over 86,400, which is 167. Peak is 500.
- Block 3: item scores per second at peak are 500 times 200, which is 100,000.

??? note "Show answer"

    Block 4: cost the model and state what it eliminates.

    Per item, 300 trees at depth 6 is 1,800 node visits, which at 2.5 ns is 4.5 microseconds, so one core scores about 222,000 items per second. The peak requirement of 100,000 item-scores per second therefore needs **less than one core**, and at 50 percent target utilisation, one core.

    What that eliminates is the entire distributed serving conversation. At this scale, a ranking service is a process on the same host that serves the request. Proposing a separate autoscaled model-serving fleet, a batching layer or an accelerator here is over-engineering by a factor of hundreds, and saying so out loud is the highest-scoring move available.

    The second elimination follows: at 100,000 item-scores per second, even the 11 GFLOP cross-encoder needs 1.09e15 FLOP/s over the full slate, which is 11 accelerators. Still too expensive for a business this size, but the top-20 version needs 500 times 20 times 1.09e10, which is 1.09e14, so roughly one accelerator. At this scale, a heavy re-ranker is genuinely affordable, which is the opposite conclusion to the 50 million DAU feed. Scale changes which model you can afford, in both directions.

**Faded example 2.** Prompt: "A fraud system scores every card transaction. 40 million transactions per day, peak-to-mean 4, 500 features, and a hard 50 ms budget because the score sits inside the authorisation path." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: assumptions as stated, plus a fraud base rate of 0.1 percent and a 180-day training window.
- Block 2: mean rate is 40e6 over 86,400, which is 463 per second. Peak is 1,852. Each transaction is scored once, so item-scores per second equals request rate: about 1,900.

??? note "Show answer"

    Block 3, training data. Positives per day are 40e6 times 0.001, which is 40,000. Over a 180-day window that is 7.2 million positives, against 7.2 billion negatives. Downsampling negatives at 1 percent gives 72 million negatives, so a 79 million row training set at roughly a 1 to 10 positive ratio. At 500 features and 2 KB per row that is 158 GB, which fits on one machine, so distributed training is unnecessary and proposing it is over-engineering.

    Block 4, the latency check and what it eliminates. 1,900 scores per second against a per-item cost of 10 microseconds is 0.019 cores, so throughput is a non-issue and the 50 ms budget is not threatened by the model at all.

    The budget is threatened by **feature fetch**: 500 features, if any of them require a network call to an aggregate store, will dominate. So the entire design conversation moves from model serving to feature retrieval, and the correct answer names a strict fan-out limit, for example one batched call to one store, with a hard timeout and a defined default when it expires.

    The second thing the numbers eliminate: at 463 transactions per second average, real-time streaming aggregation is cheap, so there is no reason to accept day-old velocity features. Any answer that computes "transactions in the last hour" from a nightly batch job has been ruled out by the arithmetic.

### 3f. Check yourself

**Q1.** An interviewer says "assume 1 billion items in the catalogue and 512-dimensional float32 embeddings". Give the memory footprint and the first design decision it forces, in under 20 seconds.

??? note "Show answer"

    1e9 times 512 times 4 bytes is 2.05 terabytes for the raw vectors alone, before any index structure. Adding an HNSW graph at M = 32 adds 256 bytes per vector, which is another 256 GB, for about 2.3 TB.

    The decision it forces: the index cannot live in the memory of any single machine, so retrieval is a sharded service from the first diagram, and either the vectors are quantized or the shard count is in the tens. Saying "2 terabytes, so this is sharded from day one" in one breath is the whole answer.

**Q2.** Your ranking stage scores 4 million items per second and you are asked whether you can add a 40-parameter-million neural ranker at 128 tokens per item. Decide, showing the arithmetic.

??? note "Show answer"

    FLOPs per item are about 2 times 40e6 times 128, which is 1.02e10, so 10 GFLOPs. Times 4 million items per second is 4.1e16 FLOP/s, or 41 PFLOP/s. At 100 TFLOP/s of usable throughput per accelerator that is 410 accelerators.

    The answer is no for the full slate, and the follow-up is the useful part: at what candidate count does it become affordable? Dividing, roughly 10 accelerators buys you 1e15 FLOP/s, which is 100,000 items per second, which at the same request rate is 1 out of every 40 candidates. So the neural model belongs on the top 2 to 3 percent of the slate, which is the definition of a re-ranking stage.

### 3g. When not to use this

Estimation is over-performed in ML rounds more often than it is skipped, because it is the part candidates have rehearsed.

| Question | Answer |
|---|---|
| The measurement that justifies a full six-number sizing | Any prompt where a candidate pool exceeds roughly 100,000 items, or where a model runs inside a latency-bound request path. Those are the two conditions under which arithmetic changes the diagram |
| Threshold below which it is over-engineering | Under about 1,000 requests per second and under 100,000 candidates, every model family fits on a few ordinary machines. Sizing tells you nothing you did not already know |
| The cheaper alternative | One line: "At this volume everything fits on a handful of machines, so I will not size it unless you want me to. The binding constraint here is label quality, not capacity." Then spend the four minutes on the constraint that is actually binding |
| The failure mode of over-applying it | Eight minutes of arithmetic that eliminates nothing, followed by a rushed serving path. The hub's [capacity estimation section](index.md#capacity-estimation-that-earns-its-place) states the general form of this rule for all tracks |

One more honest limit. Every number in this module rests on assumptions you invented, and inventing plausible assumptions is a skill an interviewer can only partly assess. If they hand you real numbers, drop yours immediately and say you are doing so. Arguing for your assumption against their measurement is the fastest way to lose a round that was going well.

---
## Module 4. Data and labels

**Time: 35 to 50 minutes reading, 25 to 35 minutes practice.**

Model architecture is the most rehearsed part of an ML design answer and the least decisive. Label design is the least rehearsed and the most decisive. This module is about where the target column comes from, what it costs, how late it arrives, and how wrong it is.

### 4a. Predict first

A card issuer wants to detect fraudulent transactions. Here are four candidate definitions of the label "this transaction was fraud".

| # | Label definition | Available after |
|---|---|---|
| L1 | The customer filed a dispute and the issuer upheld it | 45 to 90 days |
| L2 | A human analyst reviewed and marked it fraudulent | Hours to days |
| L3 | The transaction was declined by the current rules engine | Immediately |
| L4 | The card was reported lost or stolen within 7 days | 7 days |

??? note "Show answer"

    Prediction asked for: which of these can be used as the training label, and which one will silently destroy the model if used?

    L3 destroys the model, and it is the one candidates reach for because it is free and instant. It is the output of the system you are replacing, so training on it teaches the model to imitate the existing rules, including their blind spots. Worse, declined transactions never complete, so you never observe whether they would have been fraud. The label is not just biased, it is defined by the treatment.

    L1 is the ground truth and it arrives too late to train on alone. L2 is fast and reasonably accurate but only exists for transactions someone chose to review, so it is available on a biased sample. L4 is a cheap, fast, noisy proxy with a real signal in it.

    The design answer is not to pick one. It is to use L2 and L4 as short-horizon training labels, use L1 as the delayed ground truth that corrects and calibrates them, and use L3 only as a feature describing what the old system did, never as a target.

### Where labels come from, and what each one costs

Every label in an ML system comes from one of five sources. Pricing them is a two-minute exercise that reframes most design rounds.

| Source | Cost per label | Latency | Characteristic bias |
|---|---|---|---|
| Implicit user behaviour (click, watch, purchase) | Effectively zero | Seconds to hours | Position bias, presentation bias, only observed for items you showed |
| Explicit user signal (rating, report, hide) | Zero | Seconds | Extremely sparse, and skewed to the ends of the opinion distribution |
| Human review | $0.10 to $5 depending on task difficulty | Minutes to days | Only exists on the sample you chose to send, so it inherits your sampling policy |
| Downstream business event (dispute, refund, churn) | Zero | Days to months | Delayed, censored, and often the definition of the outcome you care about |
| Programmatic or weak rules | Near zero after authoring | Immediate | Systematically wrong in correlated ways, which is worse than random noise |

!!! warning "The one that ends rounds"

    Training on the output of the system you are replacing. It appears as "we have labels from the current rules engine" or "we can label with the current model's high-confidence predictions". Both create a closed loop where the new model can at best reproduce the old one, and the failure is invisible offline because the evaluation set has the same bias as the training set.

    The correct use of an existing system's decisions is as a **feature** and as a **comparison baseline**, never as a target.

### Class imbalance, and the correction downsampling forces

Module 3 downsampled negatives to 2 percent to make the corpus tractable. That is standard, and it breaks the model's output probabilities in a way that is exactly computable.

Downsampling negatives by a factor **r** leaves positives untouched, so it multiplies the odds of the positive class by **1/r**. Reversing it:

```
odds' = p' / (1 - p')          the model's odds on sampled data
odds  = r * odds'              true odds
p     = r * p' / (1 - p' + r * p')

Figure the prior correction for negative downsampling
```

Worked, with the fraud base rate from module 3. True positive rate 0.001, negatives kept at r = 0.01.

```
true odds       = 0.001 / 0.999      = 0.001001
sampled odds    = 0.001001 / 0.01    = 0.1001
model outputs   = 0.1001 / 1.1001    = 0.091

Figure a 0.1% event becomes a 9.1% model output, a 91x inflation
```

**Why this matters beyond tidiness.** If the score is only used to rank a review queue, the inflation is harmless because it is monotonic. If the score is compared against a cost threshold, fed into an expected-value calculation, or displayed to an analyst as a probability, the inflation is a production bug. Deciding which case you are in is a design decision, and saying which one out loud is the credit-earning move.

Two things are commonly confused here, so separate them explicitly:

| Technique | What it changes | When it is right |
|---|---|---|
| Negative downsampling | Corpus size and training cost | Almost always at high volume, with the correction applied |
| Class weighting in the loss | The gradient the model sees | When you want the model to care more about the rare class, not when you want a calibrated probability |
| Threshold selection | The decision, not the model | Always. This is where cost asymmetry belongs, not in the loss |

The default answer that survives follow-up: downsample negatives for cost, correct the prior, keep the loss unweighted, and put the business cost asymmetry entirely in the threshold. Weighting the loss and then also moving the threshold applies the same asymmetry twice.

### Label delay, treated as a first-class constraint

Assume the dispute maturation curve for the fraud problem: 50 percent of true fraud is reported by day 20, 90 percent by day 45, 99 percent by day 90 (assumption: disputes follow statement cycles, so the mass sits after the monthly statement arrives).

The trap is that unlabelled does not mean negative. If you build a training set from transactions 10 days old and treat every un-disputed transaction as clean, you are mislabelling the majority of positives.

```
share of true fraud reported by day 10   = about 30%
observed positive rate on 10-day-old data = 0.001 * 0.30 = 0.0003
true positive rate                        = 0.0010

Figure waiting 10 days measures a third of the fraud that occurred
```

The model trained this way learns a base rate 3.3 times too low and treats 70 percent of the positives as negatives, which is systematic label noise concentrated on exactly the hard cases.

| Response | What it costs | When to choose it |
|---|---|---|
| Wait for maturity, train on 45-day-old data | Blind to any pattern newer than 45 days, which is fatal against adversaries | Only when the phenomenon is stable |
| Short-horizon proxy label (analyst review, card reported stolen) | Proxy is biased, and needs periodic recalibration against matured labels | The usual production answer |
| Delayed feedback model: treat recent negatives as censored, model time-to-report | More modelling complexity, needs the maturation curve estimated | When calibration matters, as in ads conversions |
| Two models: fast proxy model plus slow correction model | Two pipelines to operate | When the fast and slow signals disagree in useful ways |

The censored formulation is worth being able to state: a recent unlabelled example is not a negative, it is a positive-with-probability equal to the share of reports still outstanding at its age. Modelling delayed conversions this way is a known line of work in ads, going back to Chapelle's delayed-feedback modelling paper at KDD 2014, and it is the single fastest way to sound like you have shipped one of these.

### Cold start, split three ways

"Cold start" is three different problems that people merge. Separating them is worth a minute in any recommendation round.

| Type | The missing thing | Mechanism that works | Its cost |
|---|---|---|---|
| Item cold start | No interaction history for a new item | Content features only: text, image, category, creator prior | Content features are weaker, so new items rank lower than they deserve |
| User cold start | No history for a new user | Context features: locale, device, referrer, first session behaviour, plus popularity priors | Popularity priors homogenise the early experience |
| System cold start | No logs at all, day one of the product | Rules, editorial ranking, or a model trained on a related product | You are shipping a non-ML system and calling it a baseline. That is correct |

The design consequence people miss: item cold start is a **supply** problem, not only an accuracy problem. If new items cannot earn impressions, they never generate the interactions that would let them rank, and the catalogue ossifies. The fix is an explicit exploration budget, stated as a number of slots, which module 2's guardrail on new-item impression share is there to monitor.

### Weak supervision: estimating accuracy with no ground truth

When human labels cost dollars, you write labelling functions instead: cheap noisy rules that vote on the label and abstain otherwise. The interesting question is how to combine them when you have no gold labels to score them against.

For binary labels in the range minus one to plus one, and three labelling functions whose errors are conditionally independent given the true label, the expected pairwise agreement between two functions depends only on their accuracies:

```
E[Li * Lj] = (2ai - 1) * (2aj - 1)

so, using all three pairs:
(2a1 - 1)^2 = E[L1 L2] * E[L1 L3] / E[L2 L3]

Figure recovering each rule's accuracy from agreements alone
```

Worked, on observed agreement rates from unlabelled data.

```
E[L1 L2] = 0.60, E[L1 L3] = 0.50, E[L2 L3] = 0.40

(2a1-1)^2 = 0.60 * 0.50 / 0.40 = 0.750 -> a1 = 0.933
(2a2-1)^2 = 0.60 * 0.40 / 0.50 = 0.480 -> a2 = 0.846
(2a3-1)^2 = 0.50 * 0.40 / 0.60 = 0.333 -> a3 = 0.789

Figure three rules ranked by accuracy without a single gold label
```

The load-bearing assumption is conditional independence, and it is usually false: two keyword rules written by the same person fail on the same examples. The practical answer is to test it, by checking whether observed three-way agreement matches what the pairwise estimates predict, and to hold back a small gold set purely to validate the estimates. This family of methods is what the Snorkel line of work formalised (VLDB 2017), and citing it as a named approach rather than inventing terminology is the honest move.

### Active learning under a fixed review budget

Human review capacity is a hard constraint that does not scale with traffic, which makes it one of the few genuinely interesting optimisation problems in an ML design round.

```
reviewers                = 200
items per reviewer per day = 300
review capacity          = 60,000 labels/day
content volume           = 20,000,000 posts/day
coverage                 = 60,000 / 20e6 = 0.3%

reviewer cost at $15/hour, 300 items in 8 hours
cost per label           = $15 / 37.5 = $0.40
daily labelling cost     = 60,000 * $0.40 = $24,000

Figure the review budget, in items and in dollars
```

Now the allocation question, which is where the design lives. Assume a violation base rate of 0.2 percent.

| Allocation | Share | Labels/day | Purpose | What you lose without it |
|---|---|---|---|---|
| Highest-scoring items | 70% | 42,000 | Enforcement: actually remove violations | The product does not work |
| Uncertainty band, score 0.3 to 0.7 | 20% | 12,000 | Training signal where the model is least sure | Model stops improving on the boundary |
| Uniform random | 10% | 6,000 | Unbiased measurement of prevalence and recall | You can never estimate recall, at all |

The uniform random slice is the part candidates omit and interviewers probe. Its size is derivable, not arbitrary:

```
random labels/day    = 6,000
violations found     = 6,000 * 0.002 = 12 per day
positives needed for a stable recall estimate = about 100
days to accumulate   = 100 / 12 = 8.3 days

Figure the random slice sets how often you can trust a recall number
```

That derivation produces an operational fact: **your recall metric has a weekly cadence, not a daily one**. Anyone proposing a daily recall alert on this system has not done the arithmetic. If you need a faster read, you must enlarge the random slice, which directly reduces enforcement capacity. That trade is the answer.

### The label pipeline diagram

```
+--------------+     +----------------+     +--------------+
| raw events   | --> | sampler        | --> | review queue |
| 20M/day      |     | 70/20/10 split |     | 60k/day      |
+--------------+     +----------------+     +--------------+
                             |                      |
                             v                      v
                     +----------------+     +--------------+
                     | weak labellers |     | gold labels  |
                     | rules, models  |     | + appeals    |
                     +----------------+     +--------------+
                             |                      |
                             +----------+-----------+
                                        v
                              +---------------------+
                              | training corpus     |
                              | with label source   |
                              | and weight per row  |
                              +---------------------+

Figure labels from three sources, tagged by origin, into one corpus
```

The detail that matters and is nearly always missing: **every row carries its label source and a confidence weight**. Without that column you cannot ever separate "the model is wrong" from "the label was wrong", and every future error analysis is guesswork.

### 4d. Worked example

**Prompt.** "Design the labelling system for harmful content classification. 20 million posts a day, 200 reviewers."

**Block 1. PURPOSE: price every available label source before choosing any, because the cheapest adequate label wins.**

I list five sources and put a number on each: implicit signals are free but do not measure harm; user reports are free, high volume and low precision, assume around 3 percent of reports are upheld; reviewer labels cost $0.40 each; appeal overturns are high quality but only exist on items already enforced; and prior enforcement decisions are free but are the closed loop I must not train on.

**The error most readers make here** is starting with the reviewer queue because it is the highest-quality source. Quality per label is the wrong ordering. Labels per dollar, times the value of that label at the decision boundary, is the right ordering.

!!! note "Say it before you read on"

    User reports have roughly 3 percent precision. Say, in one sentence, what they are still useful for despite that, before continuing.

**Block 2. PURPOSE: allocate the fixed review budget, because it is the binding constraint and the split is the design.**

I use the 70 / 20 / 10 split derived above and defend the 10 percent slice out loud, because it is the piece that will be challenged: "Without a uniformly random sample I can measure precision but never recall, because I only ever see the items my own model chose to show a reviewer."

Then I state the operational consequence I derived: 6,000 random reviews a day yield about 12 violations a day, so a trustworthy recall figure takes roughly eight days to accumulate. Recall is a weekly metric here. Precision, which is measured on the enforcement slice, is a daily metric.

**The error most readers make here** is proposing active learning as the whole answer. Pure uncertainty sampling maximises training value and destroys measurement, because the labelled set becomes a function of the current model and cannot be used to estimate anything about the population.

**Block 3. PURPOSE: multiply the effective label supply with programmatic labels, and prove they are good enough without buying gold labels.**

60,000 reviews a day against 20 million posts is 0.3 percent coverage, which is not enough to train a model that must generalise across dozens of policy categories. So I add labelling functions: keyword and phrase rules, a reused classifier from an adjacent policy area, a media hash match against known violating assets, and account-level reputation.

I estimate each function's accuracy from pairwise agreement using the triplet identity above, then validate the independence assumption against the small gold set, then combine into probabilistic labels with the estimated accuracies as weights.

!!! note "Say it before you read on"

    Two of my four labelling functions are keyword rules written by the same policy specialist. Say what that does to the accuracy estimates, and what you would do about it.

**The error most readers make here** is treating weak labels as free extra training data. They are correlated noise. Used without accuracy estimation and without a gold-label validation set, they teach the model the rules' blind spots and the offline metric goes up while enforcement quality goes down.

**Block 4. PURPOSE: restore the probability, because both the sampling and the weak labels broke it.**

Two distortions stack. First, negatives were downsampled for corpus size, so the prior correction from earlier in this module applies. Second, and more subtly, the reviewed examples came from a biased sampler, so the model's training distribution is not the serving distribution at all.

My fix is explicit and separable:

- Correct the downsampling analytically, with the odds formula. It is exact and needs no data.
- Correct the sampling bias by fitting the final calibration layer, isotonic or Platt, **only on the uniformly random slice**, because that slice alone is drawn from the serving distribution.
- Recheck calibration weekly on that same slice, and alert on the gap between predicted and observed prevalence rather than on raw score distribution.

**The error most readers make here** is calibrating on the full labelled set. It is the largest and most tempting dataset available and it is drawn from the wrong distribution, so it produces a confidently wrong probability.

**Result.** The deliverable is: five label sources priced, a 70 / 20 / 10 review split with the random slice defended by a recall-precision-of-measurement argument, weak supervision with accuracies estimated from agreement, a corpus where every row is tagged with its source and weight, and calibration fitted only on the unbiased slice. Not one sentence about model architecture, and that is the correct allocation.

### 4e. Fade the scaffold

**Faded example 1.** Prompt: "Design the labels for an ads conversion model. A click is observed instantly; a purchase can happen up to 30 days later; advertisers are billed per conversion." Blocks 1 to 3 are given. Produce block 4 yourself.

- Block 1: sources are click logs (instant, free), advertiser-reported conversions via a pixel or server callback (delayed, mostly reliable), and advertiser-reported offline conversions (very delayed, uploaded in batches).
- Block 2: the maturation curve, assumed, is 60 percent of conversions within 1 day, 85 percent within 7 days, 99 percent within 30 days.
- Block 3: a naive model trained on data from 2 days ago sees 60 percent of the conversions, so it under-predicts conversion rate by roughly a factor of 1 over 0.6, which is 1.67.

??? note "Show answer"

    Block 4: fix the probability, and note why here it is not optional.

    In the moderation case, calibration mattered for measurement. Here it decides how much money changes hands, because the bid is a function of predicted conversion rate. A model that under-predicts by 1.67 systematically under-bids and loses auctions it should win, and the error is not uniform across advertisers, because conversion delay varies by vertical. A furniture retailer's conversions mature far more slowly than a food delivery app's.

    The correct construction has two parts. First, model the delay explicitly: predict conversion probability and time-to-conversion jointly, so a recent unlabelled click contributes as a censored observation weighted by the share of its conversion mass still outstanding at its current age, rather than as a negative. Second, fit the delay distribution **per advertiser vertical**, because a single global curve bakes the fast verticals' shape into the slow ones and shifts spend between them.

    The check that proves it works is not AUC. It is predicted total conversions versus eventually observed total conversions, per vertical, per week, once the labels have matured.

**Faded example 2.** Prompt: "A new video platform launches next month. Zero users, zero interactions, 500,000 seeded videos. Design the label and data strategy for the first six months." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: at launch there are no behavioural labels at all, so this is system cold start. Content metadata exists: title, description, category, creator, thumbnail, duration.
- Block 2: the day-one ranker is not a model. It is editorial curation plus popularity, and saying so is the right answer rather than a concession.

??? note "Show answer"

    Block 3: design the logging before the model, because the first six months are a data acquisition problem.

    Concretely: log the full slate that was shown with positions, not only the items that were clicked, because without the negatives and the positions you cannot correct for position bias later and cannot train a ranker at all. Log the ranker version and the random seed for any exploration. Log dwell and skip, not only clicks, because a launch product has no purchase signal to fall back on.

    This logging decision is irreversible in the sense that data you did not record in month one is gone forever, which is why it comes before modelling.

    Block 4: buy exploration deliberately and derive its price. If you allocate 10 percent of slots to uniform-random items, then with 500,000 videos and, say, 50,000 slates a day of 10 items, random exploration delivers 50,000 random impressions a day, which is 0.1 impressions per video per day. Every video reaches 30 impressions, enough for a very rough engagement estimate, after roughly 300 days.

    That number says uniform exploration is far too slow, and it forces the real answer: explore proportionally to content-feature-predicted value rather than uniformly, so the budget concentrates on plausible items, and accept that the resulting data is biased and needs propensity logging to correct.

### 4f. Check yourself

**Q1.** A teammate says "we have 400 million labelled examples from the current rules engine, so data is not our problem". Give the two-sentence objection.

??? note "Show answer"

    Those are not labels, they are the previous system's decisions, so a model trained on them can at best reproduce the rules engine including everything it misses, and the offline evaluation will not reveal this because the test set carries the same bias. The rules engine's output belongs in the design as a feature and as the baseline to beat, while the actual labels have to come from a source that is independent of the decision, such as human review on a randomly sampled slice or a downstream business outcome.

**Q2.** You downsample negatives at 5 percent and the model outputs 0.4 for an example. What is the corrected probability, and when do you not bother?

??? note "Show answer"

    Using p equals r times p' over (1 minus p' plus r times p'): 0.05 times 0.4 is 0.02, divided by (1 minus 0.4 plus 0.02) which is 0.62, giving 0.032. So a 0.4 output corresponds to a true probability of about 3.2 percent.

    You do not bother when the score is used only to sort items within a single request, because the correction is monotonic and cannot change any ordering. You must bother the moment the score crosses a fixed threshold, enters an expected-value or bidding calculation, is compared across requests or models, or is shown to a human as a percentage.

### 4g. When not to use this

The machinery in this module (weak supervision, active learning, delayed-feedback modelling, sampling splits) is expensive to build and expensive to operate.

| Technique | Measurement that justifies it | Threshold below which it is over-engineering | Cheaper alternative |
|---|---|---|---|
| Weak supervision | Human labels cost more per week than an engineer-week of rule writing, and coverage is under roughly 1 percent of the population | Under about 50,000 needed labels total. Just buy them; a one-off annotation project is simpler and higher quality | A single well-specified annotation guideline and a fixed vendor batch |
| Active learning | The model's improvement has visibly stalled while the labelling budget is fully spent | Fewer than about 10,000 labels per week, where the sampler's gain is smaller than the variance between annotators | Random sampling plus manual review of the confusion matrix once a week |
| Delayed-feedback modelling | Measured gap between short-horizon and matured conversion counts above roughly 20 percent, and the score feeds a monetary decision | The outcome matures within a day, or the score is only used to rank | Train on matured labels and accept the staleness |
| A dedicated labelling platform | More than about five models and more than two annotation teams | One model, one team | A spreadsheet, a queue and a review script. Genuinely |

The general rule: label infrastructure earns its place when human attention is the scarce resource. If your labels are free behavioural signals and arrive in seconds, most of this module is not your problem and you should say so and move to module 5, where your problem probably is.

---
## Module 5. Features and the feature store

**Time: 35 to 50 minutes reading, 25 to 40 minutes practice.**

Training and serving skew is the most common real bug in production ML and the least discussed in interview material. It is not a modelling problem. It is a problem of two pieces of code computing what is supposed to be the same number, at different times, from different data.

### 5a. Predict first

Here is a feature called `purchases_7d`, defined twice. The first is the training query. The second is the serving code.

```
-- TRAINING
SELECT l.user_id, l.label, COUNT(*) AS purchases_7d
FROM   labels l
JOIN   purchases p ON p.user_id = l.user_id
WHERE  p.day BETWEEN l.day - 7 AND l.day
GROUP BY l.user_id, l.label

// SERVING
count = store.count(user_id, "purchase",
                    from = request_ts - 7 days,
                    to   = request_ts)
```

??? note "Show answer"

    Prediction asked for: name the two defects, and say which one is invisible in every offline metric.

    **Defect one, time travel.** The training query filters on `p.day`, a date, and includes `l.day` itself. If the label event happened at 09:00 and the user purchased at 17:00 the same day, that purchase is counted. The model learns from the future. Offline AUC rises, online performance does not move, and the gap is unexplained until somebody reads this query.

    **Defect two, the silent zero.** The `JOIN` is an inner join, so users with no purchases in the window produce no row at all. The training set therefore contains no examples with `purchases_7d = 0`, while at serving time that is the single most common value. The model has never seen the modal input.

    Defect one is invisible in every offline metric, because the offline metric is computed on the same leaky data. Defect two is invisible too, unless somebody compares the training-time distribution of the feature against the serving-time distribution, which is exactly why that comparison is a required piece of the design.

### The three places a feature can be computed

Every feature is computed in one of three modes, and the choice is forced by freshness, not by importance.

| Mode | Freshness | Cost driver | Characteristic failure | Example |
|---|---|---|---|---|
| Batch | Hours to a day | Full recompute over all entities | Silent staleness. The job fails and yesterday's values are served indefinitely | User's 90-day category affinity |
| Streaming | Seconds to minutes | Continuous, proportional to event rate | Out-of-order and late events; a replay produces different values than the live run | Item's click count in the last 15 minutes |
| Request time | Zero | Latency inside the request budget | Adds directly to p99. Cannot be backfilled for training unless it is logged | Distance between the user's current location and the item |

**The rule that decides.** Ask what the feature's value would be if it were one hour stale, then ask whether the decision changes. If it does not, use batch, because batch is one or two orders of magnitude cheaper to operate and can be recomputed from history.

Sizing batch is one multiplication. For 50 million users with 300 numeric features at 4 bytes:

```
50e6 * 300 * 4 bytes = 60 GB per full snapshot
recomputed nightly, 30-day history retained = 1.8 TB

Figure the entire user feature table is 60 GB, which is small
```

That number eliminates an argument. At 60 GB, a nightly full recompute is cheap and simple, so incremental update machinery for user features is unjustified. The same multiplication over 100 million items with a 15 minute refresh gives a completely different answer, which is why item features go to streaming and user features usually do not.

### Point-in-time correctness, worked

A point-in-time correct join answers: what did we know about this entity at the exact instant the prediction was made? Anything else leaks.

Here is a single user's event log and two candidate training rows.

| Time | Event |
|---|---|
| Mar 01 09:00 | purchase |
| Mar 03 14:00 | **prediction made, label recorded** |
| Mar 03 17:00 | purchase |
| Mar 05 11:00 | purchase |

| Join method | `purchases_7d` produced | Correct |
|---|---|---|
| Join on date, window `l.day - 7` to `l.day` | 2 | No. Includes Mar 03 17:00, three hours after the prediction |
| Join on timestamp, window `ts - 7d` to `ts`, strictly less than | 1 | Yes |
| Latest snapshot of the feature table, taken Mar 06 | 3 | No. Includes two future events |

The third row is the one that catches experienced people out, because it looks like ordinary engineering: you have a feature table, you join it to your labels. Feature tables hold current values. Joining current values to historical labels leaks the entire future.

**What the design must therefore contain**, stated as three requirements:

- Every feature value is stored with the timestamp from which it was valid, not only its value.
- The training join is an as-of join on that timestamp, with a strict inequality against the prediction time.
- The join carries a deliberate lag equal to the feature's real serving delay. If a streaming feature is typically 90 seconds behind live at serving time, the training join must also cut 90 seconds early, otherwise training sees a fresher feature than serving ever will.

That third requirement is the one almost nobody states, and it converts a generic answer into a specific one.

### Offline and online consistency: three strategies, priced

| Strategy | How skew arises | Skew risk | What it costs |
|---|---|---|---|
| Two implementations from one written spec | Any difference in rounding, null handling, time zone, or window boundary | Highest | Cheapest to build, most expensive to debug |
| One transformation library, executed in both paths | Only from differing input data | Medium | Language and runtime constraints on both sides |
| Log the feature vector actually served, train on those logs | Almost none by construction | Lowest | Storage, and you cannot train on a feature you did not already serve |

The third is the strongest and its cost is computable. Using module 3's numbers:

```
per-request context and user features
  1.2e9 requests * 200 features * 4 bytes  = 960 GB/day

per-impression item features
  1.2e10 impressions * 100 features * 4 B  = 4.8 TB/day
  compressed 3x                            = 1.6 TB/day
  7-day hot retention                      = 11 TB

Figure logging what was served costs about 1.6 TB per day compressed
```

The design that follows: log everything served into a short-retention store, join labels as they arrive over the following days, then downsample negatives at that point and promote the survivors into the long-retention training corpus. That ordering matters, because you cannot downsample before the labels exist without discarding the positives you have not yet learned about.

**The honest limitation** of log-at-serve, and the reason nobody uses it exclusively: a feature you invent today has no history. You cannot evaluate it until you have shipped it into the serving path and waited. So real systems run a hybrid: logged features for anything already in production, and a backfill path with point-in-time joins for anything new. Saying that out loud is what a practitioner answer sounds like.

### The skew taxonomy, and how each type is detected

| Type | What differs | How you detect it |
|---|---|---|
| Definition skew | The two implementations compute different things | Compare training values against logged serving values for the same entity and time. Any nonzero difference is a bug |
| Time-travel skew | Training used data unavailable at prediction time | Offline gain that fails to appear online, plus a feature with suspiciously high importance |
| Source skew | Same logic, different upstream table or topic | Row counts and null rates that differ between the two sources |
| Freshness skew | Training used a fresher value than serving can deliver | Compare the age distribution of feature values in training against production |
| Distribution skew | Serving traffic no longer resembles training data | Population stability of each feature, monitored per day, alerting on the tail |

The first four are bugs and are fixable. The fifth is drift and is expected, and confusing the two is a common interview stumble: a drifting feature needs retraining, a skewed feature needs a code fix, and retraining a skewed feature just relearns the bug.

### The feature layer diagram

```
+--------------+   +----------------+   +----------------+
| event stream | ->| stream compute | ->| online store   |
| clicks, buys |   | 15 min windows |   | key-value, ms  |
+--------------+   +----------------+   +----------------+
       |                                        ^
       v                                        |
+--------------+   +----------------+           |
| event lake   | ->| batch compute  | ----------+
| raw, dated   |   | nightly, 60 GB |
+--------------+   +----------------+
       |                    |
       |                    v
       |           +-----------------+
       +---------> | offline store   |
                   | as-of joins     |
                   +-----------------+
                            |
                            v
                   +-----------------+
                   | training corpus |
                   +-----------------+

Figure one definition, two paths, with the online store fed by both
```

The single property that makes this a feature store rather than two pipelines: the same registered definition drives both the streaming path and the batch path, and the offline store keeps every value with its valid-from timestamp so an as-of join is possible.

### 5d. Worked example

**Prompt.** "Design the feature layer for the short-video feed ranker from module 3."

**Block 1. PURPOSE: classify features by freshness requirement, because freshness picks the pipeline and nothing else does.**

I split the feature set into three groups and give each a required staleness bound, then let the bound choose the mode.

| Feature group | Count | Staleness that changes the decision | Mode |
|---|---|---|---|
| User long-term affinity, demographics, device | 120 | A day. Yesterday's affinity ranks the same items | Batch |
| Item lifetime engagement, creator reputation | 60 | A day, except for items under 48 hours old | Batch, plus streaming override for new items |
| Item recent engagement, last 15 and 60 minutes | 25 | Minutes. A video going viral must be visible now | Streaming |
| Session context: items seen, dwell so far, time of day | 15 | Zero. It is about this session | Request time |

**The error most readers make here** is sorting features by predictive importance and then arguing about pipelines. Importance does not choose a pipeline. Staleness tolerance does, and a highly important feature that is stable over a day belongs in the cheapest pipeline available.

!!! note "Say it before you read on"

    One row above has two modes: item engagement is batch, with a streaming override for items under 48 hours old. Say why that split exists rather than putting everything in streaming.

**Block 2. PURPOSE: choose the consistency mechanism, and pay its price out loud.**

I choose log-at-serve for everything already in production, with the storage cost stated: about 1.6 TB per day compressed, 11 TB of hot storage for a 7-day label window. I say the number because it is the objection an interviewer will raise, and pre-empting it costs one sentence.

For new features that have never been served, I use point-in-time backfill with the deliberate lag rule: each feature's backfill cuts off at the prediction time minus that feature's measured serving delay. The streaming features carry a 90 second lag; the batch features carry a lag equal to the age of the most recent completed job, which at serving time averages 12 hours.

**The error most readers make here** is choosing one mechanism for everything. Log-at-serve cannot evaluate a new feature; backfill cannot guarantee consistency. Real designs run both and say which applies when.

**Block 3. PURPOSE: make the offline join correct, in a way I can state as a rule rather than as an intention.**

I write the join rule as three clauses: as-of on `valid_from`, strictly less than `prediction_ts`, minus the per-feature serving lag. Then I add the test that proves it, which is the part that separates this answer from a slogan:

For a random sample of 10,000 logged predictions, recompute every feature through the offline path and compare it to the value that was actually logged at serving time. Report the fraction of features that differ by more than a rounding tolerance. Gate every model release on that fraction being under a stated threshold, for instance 0.1 percent of feature-value pairs.

!!! note "Say it before you read on"

    That test compares offline recomputation to logged serving values. Say what class of bug it cannot catch, before reading on.

**The error most readers make here** is describing point-in-time correctness as a principle rather than as an executable check. Interviewers hear the principle from everybody. The check is what they remember.

**Block 4. PURPOSE: budget the online read path, because a correct feature that arrives late is a wrong feature.**

The user features are one batched read per request: 120 values, one key, one round trip. Trivial.

The item features are not trivial, and the arithmetic shows why:

```
per request: 500 candidates * 100 item features * 4 bytes = 200 KB
at 35,000 requests/second = 7 GB/s of feature traffic

Figure fetching item features remotely needs 56 gigabits per second
```

7 GB per second is enough to make the feature store, not the model, the most expensive component in the system. So I use the shape of the traffic against it. Item popularity in a feed is heavily skewed, so I hold a local in-process cache of the hot items on each ranking host:

```
hot set: top 1,000,000 items
cache size: 1e6 * 100 features * 4 bytes = 400 MB per host
assume 90% of candidate lookups hit that set
remote traffic drops to 0.7 GB/s across 22 hosts
per host: about 0.25 gigabits per second

Figure a 400 MB local cache removes 90 percent of feature traffic
```

The cache carries an obligation: it is a second copy of the feature values, so it has its own staleness. I bound it by refreshing hot-item entries every 60 seconds from the streaming store and by tagging every logged feature vector with the version actually used, so the training data records what the model really saw rather than what the store currently holds.

**The error most readers make here** is treating feature retrieval as a detail below the level of the diagram. At 500 candidates per request it is frequently the dominant cost in the whole serving path, and the candidate who notices that is visibly ahead of the one who spends the same minute on the loss function.

**Result.** Four groups of features assigned to pipelines by staleness tolerance, log-at-serve plus lagged backfill as the consistency mechanism, an executable skew test gating release, and a 400 MB per-host cache derived from a 7 GB per second problem.

### 5e. Fade the scaffold

**Faded example 1.** Prompt: "Design the feature layer for the fraud scorer from module 3: 40 million transactions a day, 500 features, a hard 50 ms budget inside the authorisation path." Blocks 1 to 3 are given. Produce block 4 yourself.

- Block 1: three groups. Card and account features (batch, daily). Velocity features such as transactions in the last 1, 10 and 60 minutes (streaming, seconds). Transaction-intrinsic features such as amount, merchant category and country (request time, free).
- Block 2: log-at-serve is mandatory here, not optional, because the scorer's decisions are auditable and a regulator may ask what the model saw.
- Block 3: the as-of join must cut at the authorisation timestamp exactly, and velocity features must exclude the current transaction, which is the single most common leak in fraud systems.

??? note "Show answer"

    Block 4, the online read path against a 50 ms budget.

    The model costs about 10 microseconds per score, so essentially all of the budget belongs to feature retrieval. That inverts the usual design conversation, and saying so is the first move.

    The budget: assume 50 ms total, of which 15 ms is network and authorisation-path overhead outside your control, leaving 35 ms. A single batched read to one online store at p99 of 8 ms fits. Two sequential reads at 8 ms each also fit. Four sequential reads do not, once you add the model, serialisation and a safety margin.

    So the design constraint is stated as a rule rather than a wish: **at most two sequential round trips, each with a hard 12 ms timeout, and a defined default for every feature.**

    The default matters more than the timeout. A velocity feature that times out must not default to zero, because zero is the value that looks safest and an attacker who can induce timeouts then gets a free pass. It should default to a high-risk value, or the request should route to a conservative fallback rule. Naming the direction of the default is the senior-level detail here.

**Faded example 2.** Prompt: "Two teams, search ranking and ads ranking, both compute a feature called `user_category_affinity_30d`. Their values disagree by 12 percent on the same users. Design the fix." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: this is a brownfield organisational problem wearing a technical costume. Both teams are correct according to their own specification, so the first task is to find out whether the two specifications differ or the two implementations differ.
- Block 2: the diagnostic is to run both implementations over one identical input snapshot. If they agree on identical inputs, the problem is source data. If they disagree, the problem is logic.

??? note "Show answer"

    Block 3: the likely findings and what each implies. Different window boundaries (30 calendar days versus 30 times 24 hours) explain a few percent. Different event filters (search team excludes bot traffic, ads team does not) explain more. Different null handling for users with no events (zero versus omitted, exactly the silent-zero defect from the pre-question) explains the tail. Time zone handling for the day boundary explains a persistent daily oscillation. Every one of these is boring, and boring is why the bug survived.

    Block 4: the fix, in the order that actually ships.

    First, publish one registered definition with an owner, and version it, so the question "which definition is right" has an answer rather than a meeting. Second, do not force an immediate migration: register both as `affinity_30d_v1` and `affinity_30d_v2`, let each team keep serving what it serves, and require any new consumer to take v2.

    Third, run a shadow comparison for two weeks and publish the difference distribution. Fourth, migrate one team, measure model impact as an experiment rather than assuming it is neutral, then migrate the other.

    The step that gets skipped and should not: **measure the model impact of the migration.** A model trained on the old definition and served the new one is a textbook definition skew, and switching a feature under a live model without retraining is how a correctness fix becomes an incident.

### 5f. Check yourself

**Q1.** A model gains 3 points of offline AUC from one new feature and shows no online effect. Name the three checks you run, in order.

??? note "Show answer"

    First, check for time travel: does the feature's offline computation use any data timestamped after the prediction, including same-day aggregates and the label event itself. A 3 point jump from a single feature is more often a leak than a discovery.

    Second, compare the offline feature distribution against the served one for the same entities. Look specifically for values that exist offline and never appear online, such as the zero bucket dropped by an inner join, and for freshness differences.

    Third, check that the online path actually uses the feature. A surprising share of these cases turn out to be a feature computed and logged but never wired into the serving request, or read under a different key and silently defaulted.

    Only after all three would I consider the interesting explanation, which is that the offline metric and the online metric measure different things. Module coverage of that gap belongs to the online evaluation material, not here.

**Q2.** Give one feature that must be request-time, one that must be streaming, and one that must be batch, for a food delivery ETA model. Justify each by staleness tolerance, not by importance.

??? note "Show answer"

    Request time: straight-line distance between courier and restaurant right now. Its correct value changes every few seconds, so any stored value is wrong on arrival, and it is cheap to compute from two coordinates already in the request.

    Streaming: the restaurant's average preparation time over the last 30 minutes. A restaurant that just got slammed by a rush behaves nothing like its daily average, and a 30 minute window captures that while a daily job cannot. Staleness tolerance is minutes.

    Batch: the restaurant's 90-day median preparation time by hour of day and day of week. It is stable, it is expensive to compute over a long history, and yesterday's value is indistinguishable from today's. Staleness tolerance is a day or more, so it belongs in the cheapest pipeline.

### 5g. When not to use this

A feature store is infrastructure, and infrastructure built for one model is a liability rather than an asset.

| Question | Answer |
|---|---|
| The measurement that justifies a feature store | More than roughly five models in production sharing more than roughly 50 feature definitions across at least two teams, or a measured skew incident that cost real time. Feature stores solve a sharing problem, and sharing only exists above those counts |
| Threshold below which it is over-engineering | One or two models owned by one team. A shared transformation library plus a nightly table and a documented as-of join gives you 80 percent of the benefit for a few percent of the cost |
| The cheaper alternative | A single versioned SQL or dataframe transformation, imported by both the training job and the serving process, with a scheduled test that recomputes features for 1,000 logged requests and asserts equality |
| The failure mode of over-applying it | A feature platform with two teams maintaining it, a registry with 4,000 definitions of which 300 are used, and materialisation cost that exceeds the serving cost of every model it supports |

**When streaming features are the over-engineering.** If no feature's staleness tolerance is under an hour, a streaming pipeline buys nothing and costs you a second execution engine, a second failure mode, and replay semantics that will not match the live run. Batch every hour is dramatically simpler than a stream, and the correct answer surprisingly often.

**When request-time computation is the over-engineering.** If the value is stable over a minute and the cost per request is more than a millisecond, it belongs in a store. The test is whether the value would differ between two requests one minute apart, not whether it feels dynamic.

---
## Module 6. Retrieval then ranking, as the reusable serving pattern

**Time: 30 to 40 minutes reading, 20 to 30 minutes practice.**

Almost every large ML serving system has the same shape: cheaply narrow a huge pool, then spend increasing amounts of compute on an increasingly small set. This module derives that shape from a latency budget rather than presenting it as a convention, and then breaks the budget on purpose.

### 6a. Predict first

A candidate proposes this latency budget for a feed request with a 300 millisecond end-to-end target. Every figure is the stage's p99.

| Stage | p99, ms |
|---|---|
| Client to edge network | 80 |
| Gateway and auth | 10 |
| User feature fetch | 25 |
| Retrieval | 30 |
| Filtering and dedup | 10 |
| Item feature fetch | 35 |
| Ranking, 500 items | 45 |
| Re-rank, top 25 | 30 |
| Blending and business rules | 10 |
| Serialisation and response | 25 |
| **Total** | **300** |

??? note "Show answer"

    Prediction asked for: the budget sums to exactly the target. What is wrong with it?

    Percentiles do not add. Budgeting each of nine server-side stages at its own p99 and summing to the target guarantees the target is missed, and the arithmetic says by how much.

    If the stages were independent, the probability that a request stays under every stage's individual p99 is 0.99 to the power of 9, which is 0.9135. So about 8.6 percent of requests have at least one stage above its own p99. A budget with zero slack therefore fails for roughly one request in twelve, not one in a hundred.

    In reality the stages are not independent: they share hosts, garbage collectors, network paths and downstream stores, so slow stages co-occur and the picture is worse, not better.

    The correct construction is to budget stages at their **median** or p75, sum those, and reserve the difference between that sum and the target as explicit slack for whichever stage tails on any given request. A useful rule of thumb: allocate the sum of medians at about 50 to 60 percent of the target, leaving 40 percent as tail absorption.

### Why the funnel exists, in one multiplication

Take the module 3 numbers: 100 million retrievable items, a tree ensemble costing 10 microseconds per item, and a ranking budget of 45 milliseconds.

```
scoring the whole catalogue once
  = 1e8 items * 10 us = 1,000 seconds per request

Figure ranking everything is not slow, it is impossible by 4 orders
```

The funnel is not an optimisation. It is the only thing that fits. The design question is therefore not whether to have stages but **how many items each stage sees and how much it may spend on each**.

### The balanced funnel principle

Useful heuristic: give each stage roughly the same total time, which means the per-item budget must rise in inverse proportion to the item count.

| Stage | Items in | Items out | Per-item budget | Stage total | Mechanism it permits |
|---|---|---|---|---|---|
| Retrieval | 100,000,000 | 1,000 | about 0.3 ns amortised | 30 ms | Approximate nearest neighbour, inverted index, cached lists |
| Filtering | 1,000 | 500 | 10 us | 10 ms | Set membership, blocklists, eligibility rules |
| Ranking | 500 | 50 | 90 us available, 10 us used | 45 ms | Tree ensemble or a small network |
| Re-ranking | 50 | 10 | 600 us available | 30 ms | Cross-encoder, sequence model, slate optimisation |

Two things fall out of that table and both are worth saying in a round.

- Retrieval cannot look at each item. At 0.3 nanoseconds per catalogue item, no per-item computation is possible at all, so retrieval must be a **structure**, not a scan. That is the entire argument for an index, derived rather than asserted.
- The ranking stage has 90 microseconds of budget per item and uses 10. Ranking is therefore not latency constrained, it is **cost** constrained, because the fleet is sized by throughput. That distinction changes which optimisation is worth doing: batching and caching help, and shaving microseconds off tree traversal does not.

### The filtering stage, and the trap in it

Filtering removes items the user must not see: already watched, blocked creators, region restrictions, policy holds, and inventory that is out of stock. It is boring and it is where recall silently dies.

The trap is filtering **after** retrieval. If a filter passes a fraction **f** of items and you need **k** survivors:

```
candidates to retrieve = k / f

f = 0.05, k = 500  ->  retrieve 10,000, not 500
f = 0.5,  k = 500  ->  retrieve 1,000

Figure post-filtering forces over-retrieval by one over the pass rate
```

At f = 0.05, retrieving the usual 1,000 candidates leaves 50 survivors, so the ranking stage receives one tenth of what it was designed for and quality collapses in a way that never appears in an offline metric computed on unfiltered data.

Three responses, and which is right depends on what the filter is:

| Filter type | Right response | Why |
|---|---|---|
| High selectivity, known in advance (region, language) | Partition the index and search only the eligible partition | Turns filtering into routing, which is free |
| Medium selectivity, per user (already watched) | Over-retrieve by 1/f and filter after | Cheap, and f is measurable per user |
| Low selectivity, expensive to evaluate (policy holds) | Filter after ranking, on the small set | Evaluating an expensive rule on 10,000 items wastes the budget |

### The re-ranking stage, and what it is actually for

Ranking is pointwise: each item is scored without knowing what else is on the slate. That is a modelling convenience, and it is wrong in a specific way. The value of the fourth item depends on the first three.

Everything that depends on the slate belongs in re-ranking:

- Diversity and dedup: no more than two items from one creator, no near-duplicate videos.
- Freshness and exploration quotas: at least one item under 48 hours old, stated as a slot count so it is auditable.
- Business rules: ad load, sponsored placement, editorial pins.
- Slate-level models: a sequence model that scores the ordered list rather than the items.

The design point interviewers probe: a diversity rule implemented as a penalty inside the ranking score is worse than the same rule implemented as a slot constraint at re-rank, because the penalty is opaque, needs retuning after every model change, and cannot be audited. Slot constraints are checkable by looking at the output.

### The funnel diagram

```
+---------------------+
| catalogue 100M      |
+---------------------+
          |
          v  several sources in parallel
+---------------------+  +--------------------+
| ANN embedding       |  | recent + follows   |
| 400 items, 25 ms    |  | 600 items, 15 ms   |
+---------------------+  +--------------------+
          |                        |
          +-----------+------------+
                      v
          +---------------------+
          | filter + dedup      |
          | 1,000 -> 500        |
          +---------------------+
                      v
          +---------------------+
          | rank 500 -> 50      |
          +---------------------+
                      v
          +---------------------+
          | re-rank 50 -> 10    |
          | slate rules apply   |
          +---------------------+

Figure the funnel, with retrieval sources running in parallel
```

The parallel retrieval sources matter to the budget: two sources running concurrently cost the maximum of their latencies, not the sum, so the retrieval stage costs 25 ms rather than 40.

### 6d. Worked example

**Prompt.** "Give me the latency budget for the feed request, and tell me what happens when you miss it."

**Block 1. PURPOSE: find the real budget by subtracting what I do not control, before allocating anything.**

The product target is 300 ms end to end at p99. Client network round trip on mobile is assumed at 80 ms p99, and I do not control it. Serialisation and transfer of the response is 25 ms. That leaves 195 ms of server time to allocate.

I say the assumption out loud because it is the one most likely to be wrong: "I am assuming 80 ms of network at p99. If your clients are mostly on fixed broadband that is closer to 30 and I have another 50 ms to spend, so tell me if that is wrong."

**The error most readers make here** is allocating the full 300 ms across server stages, which quietly assumes an instantaneous network and produces a design that misses the product target on every mobile request.

**Block 2. PURPOSE: allocate by parallelism first, because concurrent stages cost the maximum rather than the sum.**

I draw the dependency order before assigning any numbers. User feature fetch and retrieval-source lookups can start together as soon as the user identifier is known. Item feature fetch cannot start until candidates exist. Ranking cannot start until item features arrive.

```
t=0    gateway, auth                       10 ms
t=10   [user features 25 | retrieval 25]   25 ms in parallel
t=35   filter and dedup                    10 ms
t=45   item feature fetch                  35 ms
t=80   rank 500                            45 ms
t=125  re-rank 25                          30 ms
t=155  blending and slate rules            10 ms
t=165  serialise                           25 ms
t=190  done, against a 195 ms server budget

Figure the sequential path, with one parallel section at t=10
```

The parallel section buys 25 ms, and that 25 ms is the entire difference between a design that fits and one that does not.

!!! note "Say it before you read on"

    Those figures still sum to nearly the whole budget. Using the reasoning from the pre-question, say what is wrong with that, before reading block 3.

**The error most readers make here** is drawing the pipeline as a straight line because that is how diagrams are drawn, then never noticing that two of the stages have no dependency on each other.

**Block 3. PURPOSE: convert the stage figures from p99 targets into medians with reserved slack, so the budget can actually be met.**

The 190 ms above is the sum of stage p99s, which the pre-question showed is the wrong quantity to sum. So I restate every figure as a median and reserve the remainder.

| Stage | Median, ms | p99, ms |
|---|---|---|
| Gateway and auth | 4 | 10 |
| Parallel features and retrieval | 12 | 25 |
| Filter and dedup | 4 | 10 |
| Item feature fetch | 15 | 35 |
| Rank 500 | 20 | 45 |
| Re-rank 25 | 12 | 30 |
| Blending and slate rules | 4 | 10 |
| Serialise | 10 | 25 |
| **Sum of medians** | **81** | |

81 ms of medians against a 195 ms server budget leaves 114 ms of slack, which is 58 percent. That is the number that makes the p99 target achievable: any single stage can go to its p99 and several can go well past it while the request still lands inside the budget.

**The error most readers make here** is treating a budget as an allocation of the target. It is an allocation of the median, with the remainder held as tail absorption. Interviewers who run latency-critical services notice the difference immediately.

!!! note "Say it before you read on"

    Name the stage in that table you would attack first if the p99 were missed, and give your reason from the numbers rather than from intuition.

**Block 4. PURPOSE: design the degradation ladder, because the budget will be exceeded and the only question is what happens then.**

A budget without a degradation plan is a wish. I put a deadline on the request at entry and check it at each stage boundary.

| Elapsed at checkpoint | Action | Quality cost |
|---|---|---|
| Over 120 ms before re-rank | Skip re-ranking, return the ranker's top 10 | Loses slate-level diversity and the heavy model's gain |
| Over 150 ms before ranking completes | Rank only the first 200 candidates that arrived | Loses the tail of the candidate set, which is the least valuable part |
| Over 170 ms at any point | Return the previous slate for this user from cache | Feed appears not to refresh, which is visible but not broken |
| Retrieval source times out | Proceed with the sources that answered | Loses one source's contribution, usually a few percent of quality |

The last row is the important one and is where the parallel design pays off a second time: independent retrieval sources mean a slow source degrades the result rather than failing the request. A single retrieval source has no such property, and that is a real argument for having more than one that has nothing to do with recall.

**Result.** 195 ms of server budget, one parallel section worth 25 ms, stages budgeted at medians summing to 81 ms with 58 percent slack, and a four-rung degradation ladder tied to elapsed-time checkpoints rather than to per-stage timeouts.

### 6e. Fade the scaffold

**Faded example 1.** Prompt: "Design the funnel for product search over 400 million listings. Target 200 ms end to end, desktop and mobile mixed, so assume 50 ms of network." Blocks 1 to 3 are given. Produce block 4 yourself.

- Block 1: 200 minus 50 network minus 20 serialisation leaves 130 ms of server budget.
- Block 2: query understanding and user feature fetch run in parallel with nothing else, because retrieval depends on the parsed query. Lexical and vector retrieval then run in parallel with each other.
- Block 3: medians are query parse 5, retrieval 15, fusion and filter 5, item features 10, rank 400 items 15, re-rank 40 with a cross-encoder 20, serialise 8. Sum of medians is 78 against 130, which is 40 percent slack, tighter than the feed but workable.

??? note "Show answer"

    Block 4, the degradation ladder for search, which differs from the feed in one structural way: **search cannot return a cached previous slate**, because the query is new. The bottom rung has to be something else.

    | Elapsed | Action | Quality cost |
    |---|---|---|
    | Over 70 ms before re-rank | Skip the cross-encoder re-rank | Loses the largest single quality contributor, but results stay relevant |
    | Over 90 ms | Drop the vector retrieval arm, return lexical results only | Loses semantic matches; exact-match queries are unaffected, which is most of the traffic |
    | Over 110 ms | Return the top results by lexical score with no ML ranking at all | Degrades to a search engine from a decade ago, which is still usable |
    | Vector arm times out | Proceed with lexical, and log the event | Small quality loss, no user-visible failure |

    The design principle worth stating: for search, the degradation ladder ends at a **non-ML baseline that is always correct**, because the lexical index can answer any query without any model. Having that floor is worth designing for deliberately. It is also the answer to "what happens if the model service is down", which is a question you should expect.

**Faded example 2.** Prompt: "A notifications system decides, once per hour, which of 40 million users to send a push to, from a pool of 10,000 candidate messages. There is no request latency budget at all, because it is a batch job." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: there is no user-facing latency, so the binding constraint is the batch window and total compute cost, not milliseconds.
- Block 2: naive full scoring is 40e6 users times 10,000 messages, which is 4e11 pairs. At 10 microseconds per pair that is 4e6 core-seconds, or 1,111 core-hours per run. At hourly cadence that is 1,111 cores running continuously, at $0.04 per vCPU-hour, or about $32,000 per month.

??? note "Show answer"

    Block 3: the funnel still applies, and the arithmetic proves it. Most of the 10,000 messages are ineligible for most users by simple rules: campaign targeting, language, platform, and frequency caps. Assume eligibility rules cut the candidate set to 50 messages per user on average. Then pairs drop from 4e11 to 2e9, which is 20,000 core-seconds, or 5.6 core-hours per run. That is a factor of 200, achieved with set operations rather than with any model, and it costs about $160 per month instead of $32,000.

    The general lesson: a funnel in a batch system is a **cost** structure rather than a latency structure, and it is derived the same way. The first stage must be a structure or a rule, never a score.

    Block 4: the degradation story is different again, because there is no request to degrade. Here the failure mode is the batch window: if the hourly job takes more than an hour, runs overlap and users get duplicate sends. So the ladder is about the window, not the request.

    Rung one, if the job exceeds 40 minutes: reduce candidates per user by tightening the eligibility rules. Rung two, if it exceeds 50 minutes: process only users whose last send was more than 24 hours ago, which is the highest-value segment. Rung three, at 60 minutes: abandon the run entirely and skip the hour, which is safe because a missed notification is far cheaper than a duplicate one. Naming "skip the run" as an acceptable outcome is the answer most candidates never reach, and it is usually correct for notifications.

### 6f. Check yourself

**Q1.** Your filter passes 20 percent of retrieved items and the ranker needs 500 candidates. How many do you retrieve, and what is the second-order effect of that decision?

??? note "Show answer"

    Retrieve 500 divided by 0.2, which is 2,500 candidates.

    The second-order effect is that retrieval quality now matters much further down its ranked list. The 2,500th nearest neighbour is a substantially worse match than the 500th, so the approximate index must maintain acceptable recall at a much larger k, which usually means raising the search parameter, which costs latency. Over-retrieval is not free even though the extra items are discarded, and the cost lands in the retrieval stage rather than in the filter.

    The follow-up worth volunteering: if the filter is predictable per user, such as already-watched items, its pass rate can be estimated per user and the over-retrieval factor set per request rather than globally, which saves the median request from paying for the worst case.

**Q2.** Someone proposes moving the diversity rule from re-ranking into the ranking model as a feature. Give one argument for and one against.

??? note "Show answer"

    For: the model can learn a smooth trade-off between relevance and diversity rather than obeying a hard slot rule, and it can condition on context, for example allowing less diversity for a user who is deliberately browsing one creator.

    Against: a pointwise ranker scores each item without seeing the rest of the slate, so it cannot express "this item is redundant given the first three". Any diversity feature it uses is a proxy computed before the slate exists. The rule also becomes unauditable: you can no longer answer "why did three items from the same creator appear" with anything better than a feature attribution, and you cannot fix the failure without retraining.

    The practical resolution: keep the hard slot constraint at re-rank as a floor, and let the model learn preferences above that floor. The constraint is what you can promise; the model is what makes the promise pleasant.

### 6g. When not to use this

The multi-stage funnel is the default in large systems and is over-applied in small ones, where it triples the number of components for no benefit.

| Question | Answer |
|---|---|
| The measurement that justifies a retrieval stage | Catalogue size times per-item scoring cost exceeds the ranking budget. With a 45 ms budget and 10 us per item, that is about 4,500 items. Above that number you need a narrowing stage; below it you do not |
| Threshold below which it is over-engineering | Under roughly 5,000 eligible items per request. Score them all with one model, sort, and return. This covers most internal tools, most B2B products and a surprising number of consumer ones |
| The cheaper alternative | One stage, one model, plus a database query with a `WHERE` clause and an `ORDER BY`. If eligibility rules already narrow the pool to hundreds, the rules are your retrieval stage and you should say so |
| The failure mode of over-applying it | Four services, four deploy pipelines, four failure modes and an end-to-end debugging problem, to rank 800 items. Every stage boundary is also a place where the candidate set can silently become empty |

**When a re-ranking stage specifically is not justified.** If your ranking model already fits in the per-item budget for the whole candidate set, a second stage adds latency and cost for nothing. The re-ranker exists only because a better model is too expensive to run on all candidates. Measure that first: if the heavy model costs less than the per-item budget from the balanced funnel table, use it everywhere and delete a stage.

**When even the ranking model is not justified.** If the catalogue is under a few thousand items and engagement is dominated by recency or popularity, a sorted query is competitive with a model and needs no training data, no feature pipeline and no monitoring. The correct interview answer is sometimes to build the funnel's structure now and populate it with rules until there is enough data to train on. Saying that costs nothing and demonstrates the restraint that the case-study modules keep scoring.

---
## Module 7. Retrieval infrastructure

**Time: 35 to 50 minutes reading, 25 to 35 minutes practice.**

Module 6 derived that retrieval must be a structure rather than a scan. This module chooses the structure, on build cost, query cost and memory, and then handles the part that decides whether the system survives contact with production: keeping the index fresh.

### 7a. Predict first

Three index configurations for the same corpus of 100 million 256-dimensional vectors. The corpus grows by 1 million vectors per day.

| # | Configuration | Memory |
|---|---|---|
| I1 | Flat float32, exhaustive scan | 102 GB |
| I2 | IVF, 4,096 lists, product quantized to 64 bytes | 7 GB |
| I3 | HNSW, M = 32, float16 vectors | 77 GB |

??? note "Show answer"

    Prediction asked for: which of these can absorb 1 million new vectors per day without a rebuild, and which one degrades in a way you would not notice?

    I1 absorbs inserts trivially: append to an array. It is also the slowest to query by a wide margin, which is why nobody uses it at this size.

    I3 absorbs inserts at roughly the cost of one query each, because inserting into a navigable graph means searching for the new node's neighbours and linking. At 1 million per day, that is 12 inserts per second, which is nothing. Deletion is the weakness: graph indexes generally cannot remove a node without breaking connectivity, so deletes become tombstones and accumulate.

    I2 is the trap, and it is the answer the question is really asking for. Inserting into an IVF index is cheap: assign the vector to its nearest centroid and append. But the centroids and the quantization codebooks were **learned from a sample of the original corpus**. As new data arrives with a different distribution, list occupancy skews and quantization error rises. Query latency drifts up and recall drifts down, gradually, with no error and no alert, and the index looks healthy the entire time.

    The design consequence: any index with learned components needs a scheduled retraining of those components, and a measurement that would detect the drift before a user does.

### The families, compared on the three costs that matter

| Family | Memory per vector, 256 dims | Query cost | Build cost | Insert | Delete | Learned parts |
|---|---|---|---|---|---|---|
| Flat | 1,024 B | N distance computations | Zero | Append | Swap and shrink | None |
| IVF | 1,024 B plus centroids | nprobe times N/nlist distances | One clustering pass | Assign and append | Mark in list | Centroids |
| IVF plus PQ | 64 to 128 B | Same count, but table-lookup distances | Clustering plus codebook fit | Assign and append | Mark in list | Centroids and codebooks |
| HNSW | 1,024 B plus 256 B links | About efSearch times M distances | N log N graph construction | One search plus link update | Tombstone only | None |
| HNSW plus PQ | 64 B plus 256 B links | Cheap distances, more of them | Both of the above | As above | As above | Codebooks |
| Disk-resident graph | About 32 B in memory, rest on disk | Distances plus disk reads | Expensive, partitioned build | Batched | Tombstone | Codebooks |

Locality-sensitive hashing appears in a lot of older material. It is worth naming and dismissing in one sentence: for the same recall it generally needs far more memory than a graph index, and it lost the practical argument once graph and quantization methods matured, so proposing it today invites a follow-up you do not want.

### Query cost, derived rather than quoted

Both major families have a query cost you can compute from their parameters, which means you can defend a configuration instead of naming a product.

**IVF plus PQ.** With N = 100 million, nlist = 4,096 and nprobe = 32:

```
vectors per list      = 1e8 / 4096          = 24,414
vectors scanned       = 24,414 * 32         = 781,250
per distance, PQ, 64 subquantizers          = about 128 ops
ops per query         = 781,250 * 128       = 1.0e8
at 2e10 ops/s per core                      = 5 ms

Figure IVF plus PQ costs about 5 ms of one core per query
```

**HNSW.** With efSearch = 128 and M = 32, a search visits on the order of efSearch times M candidate nodes:

```
distance computations = 128 * 32            = 4,096
per distance, 256 dims float16              = 512 flops
flops per query       = 4,096 * 512         = 2.1e6
at 2e10 flop/s per core                     = 0.1 ms

Figure the graph index costs about 0.1 ms of one core per query
```

So the graph is roughly 50 times faster per query and costs roughly 11 times the memory. Neither wins in the abstract. What decides is query rate, and that break-even is computable.

### The break-even between memory and CPU

Assume $0.04 per vCPU-hour and $0.005 per gigabyte of RAM per hour (assumption: these are the ratios implied by ordinary general-purpose cloud instance pricing, where memory and cores are sold together in fixed ratios; substitute your own and redo the two lines).

```
HNSW    per replica: 77 GB and Q * 0.0001 core-seconds per second
IVF-PQ  per replica:  7 GB and Q * 0.005  core-seconds per second

daily cost, HNSW   = 9.6e-5 * Q  + 9.24
daily cost, IVF-PQ = 4.8e-3 * Q  + 0.77

equal when  4.704e-3 * Q = 8.47  ->  Q = 1,800 queries/second

Figure below 1,800 queries per second, quantize; above it, use a graph
```

That single number is the answer to "which index would you use", and it is derived from four inputs the reader can change. Below roughly 1,800 queries per second per replica, memory dominates the bill and quantization is right. Above it, CPU dominates and the graph is right. At the 20,000 queries per second of the product search example, the graph wins by roughly a factor of eight on total cost.

**The caveat that keeps this honest**: the two configurations do not deliver identical recall. The comparison is only valid once both have been tuned to the same measured recall at the same k, which is what the next section is for.

### Build cost: measure small, extrapolate, never quote

Graph construction is roughly N log N in distance computations, and the constant depends on your hardware, dimensionality and parameters far too much to quote. So measure it.

```
build 1% of the corpus, 1,000,000 vectors, and time it
scale factor to 100,000,000 vectors:

  (100e6 / 1e6) * (log2(100e6) / log2(1e6))
  = 100 * (26.6 / 19.9)
  = 133

Figure extrapolating a 1 percent build to the full corpus
```

If the 1 million vector build takes 4 minutes on your machine, budget roughly 9 hours for the full one. That estimate is worth an order of magnitude more than any published benchmark, because it is measured on your data with your parameters.

The number matters because it decides your refresh strategy. A 9 hour build cannot run hourly. It can run nightly, which means everything fresher than one day has to come from somewhere else.

### Measuring recall without a labelled dataset

The recall of an approximate index is measurable exactly, and cheaply, and almost nobody in an interview says how. Take a sample of queries, compute their true nearest neighbours by brute force once, and store that as ground truth.

```
1,000 sampled queries against 100,000,000 vectors
distance computations = 1e3 * 1e8 = 1e11
flops                 = 1e11 * 512 = 5.1e13
at 2e10 flop/s per core = 2,560 core-seconds
on 32 cores             = 80 seconds

Figure exact ground truth for a recall curve costs 80 seconds
```

With that ground truth fixed, sweep efSearch or nprobe and plot recall at k against p99 latency. Pick the point where the recall curve flattens. Re-run it after every index rebuild and after every embedding model change, and alert if the operating point moves.

**Why this earns credit**: "we would tune for recall" is what everybody says. "Ground truth for a thousand queries costs 80 core-seconds, so I would regenerate it on every rebuild and gate the deploy on the recall-latency curve" is a design, not an intention.

### Freshness: the two-tier index

The build cost derivation says the main index refreshes nightly. The product usually needs new items searchable in minutes. Both are satisfied by splitting the index in two.

```
+------------------------+     +---------------------+
| base index, frozen     |     | hot index           |
| 100M vectors, 77 GB    |     | 1M vectors, 1.3 GB  |
| rebuilt nightly, 9 hrs |     | rebuilt every 10 min|
+------------------------+     +---------------------+
            |                            |
            +-------------+--------------+
                          v
              +------------------------+
              | merge top-k from both  |
              | apply tombstone filter |
              +------------------------+
                          v
              +------------------------+
              | candidates to filter   |
              +------------------------+

Figure a frozen base index plus a small hot index, merged at query
```

The hot index sizing is derived: 1 million new items per day at 1,280 bytes each is 1.3 GB, which builds in seconds and can be rebuilt from scratch every ten minutes rather than updated in place. Rebuilding from scratch is dramatically simpler than incremental maintenance, and at 1.3 GB it is affordable, so simplicity wins.

Deletion is handled by a tombstone set applied after the merge, not inside either index. The set is small: at an assumed 2 percent deletion per week, it holds 2 million identifiers, which as a hash set of 8 byte keys is 16 MB.

**When the base index must be rebuilt** is also derivable rather than a matter of taste:

| Trigger | Threshold | Why |
|---|---|---|
| Tombstone fraction | Above 20 percent | Query work rises by roughly 1 over (1 minus t) and recall at k degrades because deleted nodes occupy result slots |
| Vectors added since the last learned-component fit | Above 20 percent of corpus | Centroids and codebooks were fitted to a distribution that no longer holds |
| Measured recall at k on the fixed probe set | Drop of more than 2 points | The direct measurement, which supersedes both proxies above |
| Embedding model version change | Always | Vectors from two models are not comparable, so a mixed index silently returns nonsense |

The last row is the one that causes real incidents. Changing the embedding model requires re-embedding the entire corpus and swapping indexes atomically. Serving a mixed index during a partial migration produces results that are not wrong in an obvious way, which is the worst kind of wrong.

### Hot swap and the memory cliff

Swapping a 77 GB index means holding the old and new copies simultaneously, which needs 154 GB plus working memory. Two options:

| Approach | Memory needed | Cost |
|---|---|---|
| Build in place, atomic pointer swap | 2x, so 192 GB hosts | Larger, more expensive instances, permanently, for an event that happens nightly |
| Roll shard by shard across replicas | 1x plus one replica's worth | A window where replicas serve different index versions, so results are inconsistent across retries |

The second is usually correct and its cost must be stated: for the duration of the roll, two identical queries can return different results. If the product requires stable pagination or reproducible results within a session, pin a session to an index version, which is one more thing to build.

### Filtered search, and where graph indexes break

Module 6 covered over-retrieval for post-filtering. Graph indexes have an additional failure that quantized list-based indexes do not.

A graph search walks from an entry point toward the query. If eligibility is checked during traversal and only 1 percent of nodes are eligible, the walk spends its budget on ineligible nodes and can terminate in a region with no eligible neighbours at all. Recall does not degrade gracefully; it collapses.

| Filter selectivity | Workable approach |
|---|---|
| Above roughly 10 percent pass rate | Post-filter with over-retrieval by 1 over the pass rate |
| Roughly 1 to 10 percent | Filter during traversal, with a raised efSearch and a hard visited-node cap so latency stays bounded |
| Below roughly 1 percent, and the attribute is low cardinality | Partition: build one index per attribute value and route the query. Filtering becomes routing, which is free |
| Below roughly 1 percent, high cardinality | Do not use vector search for this query. A conventional inverted index on the attribute, followed by exact scoring of the small result set, will be both faster and correct |

That last row is the one worth remembering. When a filter is very selective, the filter itself is the better retrieval structure, and insisting on vector search is the mistake.

### 7d. Worked example

**Prompt.** "Pick and configure the retrieval index for the 400 million listing product search from module 3, and tell me how it stays fresh."

**Block 1. PURPOSE: reduce the requirements to the three numbers that select a family, before naming any technology.**

I write three numbers on the board: 400 million vectors at 384 dimensions, 20,000 queries per second at peak, and a 25 millisecond retrieval budget. Then I add a fourth that nobody gave me and that decides the refresh design: how often does the catalogue change? I ask, and assume 2 percent of listings change price or availability per hour, and 1 percent of listings are new per day.

**The error most readers make here** is naming a vector database in the first sentence. The product does not choose itself; the three numbers do, and a named product without the numbers cannot be defended when the interviewer changes one of them.

!!! note "Say it before you read on"

    Before block 2, say which of those four numbers you expect to be decisive, and why.

**Block 2. PURPOSE: apply the memory-versus-CPU break-even, because at this query rate it decides the family.**

At 20,000 queries per second, spread over replicas, the per-replica rate is far above the 1,800 queries per second break-even derived above, so CPU dominates the bill and the graph index wins. But module 3 showed the graph in float16 is 358 GB, which does not fit one host.

So I combine, rather than choose: **HNSW graph over product-quantized vectors**, with exact re-scoring of the top 1,000 from a local float16 store.

```
per vector: 96 B quantized + 128 B links (M = 16)  = 224 B
400M vectors                                        =  90 GB
plus float16 vectors for re-scoring, on local SSD   = 307 GB on disk
three replicas of 128 GB RAM

Figure the graph over quantized vectors fits one host per replica
```

**The error most readers make here** is treating the choice as graph versus quantization. They are orthogonal: the graph decides how you navigate, the quantization decides how much each vector costs. Combining them is standard and saying so distinguishes a designed answer from a recalled one.

**Block 3. PURPOSE: design the refresh path, because catalogue churn, not query volume, is what will actually break this.**

Three distinct kinds of change, three distinct mechanisms, and conflating them is the common failure:

| Change | Rate | Mechanism |
|---|---|---|
| New listing | 4 million/day | Hot index, rebuilt from scratch every 10 minutes, merged at query time |
| Listing removed or sold out | Assume 3 million/day | Tombstone set applied after the merge, 24 MB in memory |
| Price or stock change | 8 million/hour | **Not an index update at all.** Price is not in the embedding; it is a filter and a ranking feature fetched at query time |

The third row is the one that matters most, and it is a modelling decision disguised as an infrastructure one: putting a fast-changing attribute into the embedding forces an index update on every change. Keeping it out of the embedding turns 8 million hourly index updates into zero.

!!! note "Say it before you read on"

    Name one other listing attribute you would deliberately keep out of the embedding for the same reason, and one you would definitely put in.

**The error most readers make here** is designing an index that must be updated whenever anything about an item changes. The embedding should carry only slow-moving semantics.

**Block 4. PURPOSE: define the measurement that tells me it is working, because an approximate index degrades silently.**

Three measurements, each with a cadence and a trigger:

- **Recall at 100 against exact ground truth**, on 1,000 fixed probe queries. Ground truth costs about 80 core-seconds to recompute. Run after every nightly rebuild, alert on a drop of 2 points.
- **p99 retrieval latency**, per replica, per index version. Alert on a 20 percent increase, since a rising latency at fixed efSearch usually means tombstone accumulation.
- **Hot index coverage**: the fraction of results in the final slate that came from the hot index. If that goes to zero, the hot path has silently broken, and no other metric would show it.

The third is the one people forget. A merge path that silently stops contributing looks perfectly healthy on latency and error rate, and the only symptom is that new listings never appear, which the business notices weeks later through a complaint.

**Result.** A quantized graph index at 224 bytes per vector, 90 GB per replica, three replicas, a 10 minute hot index, a tombstone set, price and stock deliberately excluded from the embedding, and three measurements with cadences and triggers.

### 7e. Fade the scaffold

**Faded example 1.** Prompt: "Semantic search over 3 million internal documents. 5 queries per second. Documents change constantly, and a change must be searchable within a minute." Blocks 1 to 3 are given. Produce block 4 yourself.

- Block 1: the numbers are 3 million vectors, 5 queries per second, and a 60 second freshness requirement. The query rate is four orders of magnitude below the break-even, so CPU cost is irrelevant and simplicity should win.
- Block 2: at 768 dimensions and float32, 3 million vectors is 9.2 GB, which fits comfortably in the memory of one ordinary host, so no quantization and no sharding are needed. A flat index would cost 3e6 times 768 times 2 flops, which is 4.6 GFLOPs per query, or roughly 230 ms per core, which is too slow. So a graph index it is, at 9.2 GB plus 128 MB of links.
- Block 3: freshness within 60 seconds with a 3 million vector corpus means the whole index rebuilds in well under a minute, so a hot index is unnecessary complexity.

??? note "Show answer"

    Block 4: given all of the above, the right answer is the one that removes components rather than adding them.

    Rebuild the entire index from scratch on a schedule, every 60 seconds if the build allows it, or maintain a single in-memory graph with live inserts and periodic full rebuilds to clear tombstones. At 3 million vectors, extrapolating from a 1 million vector build, the full build is a small number of minutes at most and quite possibly seconds with modest parameters, so measure it and let that decide.

    The measurements are the same three as before but with different cadences: recall against ground truth after each rebuild, p99 latency, and index freshness measured as the age of the newest document present in the index. That third metric replaces hot index coverage because there is no hot index.

    What a strong answer explicitly refuses to build here: sharding, quantization, a two-tier index, and a dedicated vector database service. At 5 queries per second and 9 GB, this is a library inside the existing application process, and proposing infrastructure is the failure mode. If the interviewer pushes, the honest line is: "I would revisit all of this above roughly 50 million documents or 1,000 queries per second, and here are the two numbers I would watch."

**Faded example 2.** Prompt: "You are migrating from a 768-dimensional embedding model to a 1024-dimensional one, over 200 million vectors, with no allowed downtime and no allowed quality regression." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: the two vector spaces are not comparable, so a mixed index is invalid. This is a full re-embedding plus a full rebuild, not an update.
- Block 2: the cost is dominated by re-embedding, not by index building. 200 million documents through an encoder is an offline batch job whose duration should be computed from the encoder's throughput before anything else is decided.

??? note "Show answer"

    Block 3: run the two indexes side by side. Build the new index offline into a separate service while the old one continues to serve every query. Memory cost during the migration is the sum of both, which for 200 million vectors is substantial and must be budgeted as a temporary doubling. This is the shadow index pattern, and it is the only approach that satisfies "no downtime".

    Then compare, before switching anything: run a fraction of live queries against both, log both result sets, and measure overlap at k plus the downstream ranking metric on each. Expect overlap to be well below 100 percent, because a better model should disagree with a worse one. Overlap is a diagnostic, not a target.

    Block 4: switch by traffic fraction, not by flag flip. Route 1 percent, then 5, then 25, then 100, holding at each step long enough for the online metric to reach significance. Keep the old index warm and serving the remainder throughout, so rollback is a routing change rather than a rebuild.

    The two things a strong answer adds. First, the ranking model consumes retrieval output, so a retrieval change shifts the ranker's input distribution, which means the ranker should be retrained on data from the new retrieval regime before the final step rather than after.

    Second, name the end state explicitly: the old index and the old embedding pipeline get deleted on a dated plan, because a migration with no deprecation date becomes two systems forever. The general form of this argument lives in the brownfield material of [Modern System Design Interview](modern-system-design.md).

### 7f. Check yourself

**Q1.** Your graph index has accumulated tombstones for 30 percent of its vectors. Describe the two symptoms you would see and what you would do.

??? note "Show answer"

    Symptom one, latency rises at fixed parameters, because the search visits deleted nodes that consume traversal budget without contributing results, so the effective work per query scales roughly as 1 over (1 minus 0.3), which is about a 43 percent increase.

    Symptom two, recall at k falls, because deleted nodes occupy positions in the candidate heap during traversal and the search terminates having examined fewer live candidates than the parameters imply.

    The action is a full rebuild, because graph indexes generally cannot reclaim deleted nodes without reconstructing connectivity. The preventive action is a rebuild trigger at a stated tombstone fraction, around 20 percent, monitored as a metric rather than noticed as an incident.

**Q2.** An interviewer asks why you would not just use exact search. Give the threshold and the arithmetic.

??? note "Show answer"

    Exact search costs N distance computations per query; a graph index costs roughly efSearch times M, which for typical parameters is a few thousand, but each of those is a random memory access rather than a sequential scan, so it is perhaps 10 times more expensive per distance.

    Setting them equal: exact wins while N is below about efSearch times M times 10, which for efSearch 128 and M 32 is roughly 40,000 vectors. Adding index build cost, memory overhead and the operational burden of refresh, the practical crossover sits closer to 100,000 to 1 million vectors depending on query rate.

    So the honest answer is: under about 100,000 vectors, use exact search, it is faster to build, exactly correct, trivially updatable and has no recall metric to monitor. Above about 1 million, approximate search is required. In between, measure.

### 7g. When not to use this

| Question | Answer |
|---|---|
| The measurement that justifies an approximate index | Corpus size times per-distance cost exceeds the retrieval latency budget at your query rate. Concretely, above roughly 1 million vectors, or above roughly 100,000 with high query volume |
| Threshold below which it is over-engineering | Under about 100,000 vectors. An exhaustive scan with vectorised distance computation is sub-millisecond, exact, has no parameters, no build step, no recall metric and no refresh problem |
| The cheaper alternative | A float32 array and a matrix multiply. For under 100,000 vectors this genuinely outperforms an approximate index on total cost, because the index's build and refresh cost never amortises |
| The failure mode of over-applying it | A separate vector database service, with its own deployment, replication, backup and on-call rotation, holding 40,000 vectors that a 300 megabyte array in the application process would serve faster |

**When a dedicated vector service is the over-engineering, even at scale.** If your corpus already lives in a relational or search engine that supports vector indexing, adding a second datastore introduces a consistency problem: two systems that must agree about which documents exist. That consistency problem is frequently more expensive than the performance you gained. The measurement that justifies separating them is when the vector workload's memory or query pattern actively harms the primary store's performance, which is observable, not hypothetical.

**When approximate retrieval is the wrong tool entirely.** If the query is a filter with high selectivity, if results must be exactly reproducible, or if the corpus is small and highly dynamic, an inverted index or a plain database query with exact scoring beats vector search on every axis. Module 6's filtering table gives the selectivity threshold: below roughly 1 percent pass rate on a high-cardinality attribute, stop reaching for the embedding.

---
## Module 8. Model choice as a design decision

**Time: 35 to 45 minutes reading, 25 to 35 minutes practice.**

Model architecture deserves about five minutes of a 45 minute round, and this module is what those five minutes should contain. The claim being defended: at production scale the model family is chosen by the serving pattern, the data shape and the cost per item, and almost never by an accuracy argument.

### 8a. Predict first

Three ranking or retrieval problems, described only by their data and their constraints.

| # | Rows available | Feature shape | Constraint |
|---|---|---|---|
| P1 | 800,000 labelled transactions | 60 numeric and low-cardinality categorical features | 50 ms, inside an authorisation path |
| P2 | 20 billion impressions | 300 features including an item identifier with 200 million distinct values and a 50-item interaction history | 90 us per item at ranking |
| P3 | 100 million retrievable items | User identifier, item identifier, plus content features | 25 ms to select 1,000 from 100 million |

??? note "Show answer"

    Prediction asked for: name the model family for each, and say which constraint forced it.

    **P1: gradient boosted trees.** 800,000 rows of tabular features is squarely in the range where boosting is competitive with or better than a network, needs no embedding infrastructure, trains in minutes, and gives usable feature attributions for the analysts who will have to explain declines. The constraint that forces it is the row count, not the latency.

    **P2: a neural network with embeddings.** Not because networks are better, but because a 200 million value identifier and a 50-item sequence cannot be represented by axis-aligned splits. A tree can only split a categorical feature by enumerating values or by target encoding, and neither survives 200 million values. The constraint that forces it is the feature shape.

    **P3: a two-tower model.** Not for accuracy. Because module 7's index requires item vectors computable **without knowing the user**, so any architecture with user-item cross features is structurally impossible here. The constraint that forces it is the serving pattern.

    Note that none of the three was decided by which family scores higher on a benchmark. That is the point of the module.

### Gradient boosted trees against neural networks

| Dimension | Favours trees | Favours a network | The number that decides |
|---|---|---|---|
| Rows available | Under about 1 million | Above about 100 million | Rows per parameter you would need |
| Feature cardinality | Under about 1,000 distinct values per categorical | Millions of distinct identifiers | Distinct values in the largest categorical |
| Feature type | Tabular numeric and categorical | Sequences, text, images, audio | Whether any feature is not a scalar |
| Cost per item at serving | 10 us for 500 trees at depth 8 | 10 GFLOPs and up for a transformer | The per-item budget from module 6 |
| Feature engineering effort | High. Trees need crafted interactions | Lower. The network learns interactions | Engineer-weeks per point of metric |
| Cold start on new categories | Handles unseen values by falling to a default branch | Needs a hashing scheme or an out-of-vocabulary embedding | Fraction of serving traffic with unseen identifiers |
| Monotonic constraints | Supported directly and enforceable | Awkward and usually approximate | Whether a regulator requires monotonicity |
| Reuse of learned representations | None. Each model is standalone | Embeddings transfer across tasks and models | Number of downstream models that want the representation |

**The honest default**: for tabular data under roughly 10 million rows, boosted trees are the baseline that must be beaten, and they frequently are not beaten. Saying that in an interview is a strength, not a hedge, provided you also name the conditions under which you would switch.

**Where the switch actually comes from**, with arithmetic. Representing a 200 million value item identifier:

```
one-hot in a tree: 2e8 candidate splits evaluated per node.
                   Not a tuning problem, an impossibility.

embedding, dim 32: 2e8 * 32 * 4 bytes = 25.6 GB of table

frequency cutoff, keep the top 5 million identifiers,
assume they cover 95 percent of impressions:
                   5e6 * 32 * 4 bytes = 640 MB

Figure the identifier problem, and the cutoff that makes it tractable
```

That 640 MB number is the interesting one. It says the network's advantage is affordable, and it says exactly what you gave up: the 195 million rare items now share a hashed or out-of-vocabulary embedding, so the model is deliberately worse on the long tail. Whether that is acceptable is a product question, and the guardrail from module 2 on new-item impression share is the thing that would detect it going wrong.

### Two-tower retrieval, and why its shape is forced

The item tower's output must be computable offline, because it goes into the index built in module 7. That single requirement forces the architecture:

```
+--------------+                +---------------+
| user tower   |                | item tower    |
| online, 5 ms |                | offline batch |
+--------------+                +---------------+
        |                               |
        v                               v
   user vector                     item vectors
    256 dims                    100M x 256 dims
        |                               |
        +-------------> dot <-----------+
                   product / ANN

Figure the towers never meet before the dot product, by construction
```

**What this costs you, stated plainly**: no feature can depend on both the user and the item until after retrieval. "How many of this creator's videos has this user watched" is exactly the feature you want and exactly the feature this architecture forbids. That is why retrieval is followed by ranking: the ranker is where cross features live, and the funnel exists partly to make cross features affordable.

**Three specifics that separate a designed answer from a recited one.**

| Detail | Why it matters | What goes wrong without it |
|---|---|---|
| Sampled softmax with a logQ correction | In-batch negatives appear in proportion to their popularity, so popular items are over-penalised | Popular items get systematically pushed down, and the retrieval slate skews obscure |
| Mixed negatives: in-batch plus uniformly sampled from the catalogue | In-batch negatives are all items someone engaged with, which is not the population the index searches | The model never learns to reject the 99.99 percent of the catalogue that is irrelevant |
| Temperature on the dot product | Controls how peaked the softmax is, and interacts with the negative count | Untuned temperature makes the loss either flat or degenerate |

The popularity correction is the one interviewers probe. The mechanism is simple enough to state: if an item appears as a negative with probability proportional to its frequency Q(j), subtracting log Q(j) from its logit removes that bias. The sampled-softmax formulation with this correction is described in Yi and colleagues at RecSys 2019, and naming the source is better than presenting it as folklore.

**The cost of keeping item vectors current** connects straight back to module 7:

```
item tower forward pass, assume 5 MFLOPs per item
100e6 items * 5e6 FLOPs = 5e14 FLOPs
at 1e14 usable FLOP/s   = 5,000 seconds = 1.4 accelerator-hours

Figure re-embedding the entire catalogue costs under two GPU hours
```

That number matters because it says a full daily re-embedding is affordable, which in turn means the nightly index rebuild from module 7 can use fresh vectors rather than months-old ones. If the number had come out at 200 hours, the design would need incremental embedding and a version-mixing problem, and the whole retrieval section would change.

### Multi-task and multi-objective ranking

Module 2 produced three heads and a combination rule. There are two ways to build that, and the choice is usually made on cost rather than accuracy.

| Design | Serving cost for 3 objectives | Training | When it wins |
|---|---|---|---|
| Three separate models | 3 full forward passes, so 33 GFLOPs per item at the transformer size from module 3 | Independent, simple, each tunable | Tasks are unrelated, or teams need independent release cadences |
| One shared trunk, three heads | 1 trunk pass plus 3 small heads, about 11.3 GFLOPs | Coupled. One task's regression can be caused by another's data | Tasks share signal, and serving cost matters |
| Shared trunk with gated experts | Between the two, plus gating overhead | Most complex | Tasks are related but conflicting, so a single shared trunk underfits some of them |

The serving arithmetic is decisive at scale: 33 GFLOPs against 11.3 is a factor of 2.9, which at module 3's re-ranking cost of $4,560 per day is a difference of about $8,900 per day. Multi-task learning at that scale is frequently a **cost decision** that gets narrated afterwards as an accuracy decision.

The second real reason is label sparsity. If clicks occur at 4 percent and purchases at 0.1 percent, the purchase head has 40 times less signal. A shared trunk trained mostly by the click task gives the purchase head a representation it could never have learned alone.

The counter-risk is negative transfer, where an unrelated task's gradients degrade the shared representation, and the mitigation is expert gating so each task can select which parts of the trunk it uses. The multi-gate mixture-of-experts formulation from KDD 2018 is the standard reference to name here.

**What the heads do not decide**: the combination weights. Those still come from module 2 and are set online. A multi-task model produces three calibrated probabilities; it does not tell you what a completion is worth relative to a report.

### Distillation, as a latency and cost decision

Distillation trains a small student to reproduce a large teacher's outputs, including the teacher's probabilities on examples that have no label.

```
teacher: 11 GFLOPs/item, 95 accelerators, $4,560/day (module 3)
student:  0.35 GFLOPs/item, 31x cheaper
          95 / 31 = 3 accelerators, about $150/day

Figure distillation converts a 31x compute reduction into $4,400/day
```

The trade is quality, and the honest way to present it is as a rate: if the student gives up 0.3 points of AUC and the cost falls by 97 percent, whether that is right depends on how much 0.3 points of AUC is worth in your product, which is measurable from past experiments. That framing, cost per unit of metric, is what a senior answer sounds like.

**Two things distillation buys that are easy to miss.** The student can be trained on unlabelled traffic, because the teacher supplies targets, which is useful when labels are the scarce resource. And the student inherits the teacher's calibration behaviour if trained on soft targets, which a separately trained small model would not.

**Where it fails**: when the teacher is only better because of features the student cannot afford to fetch. Distillation compresses computation, not information. If the teacher's advantage came from a 200 millisecond feature lookup, no student will reproduce it.

### 8d. Worked example

**Prompt.** "Pick the models for the whole short-video feed stack and justify each one."

**Block 1. PURPOSE: choose the retrieval model from the index's constraint, because that constraint is not negotiable.**

Module 7's index stores one vector per item and searches by dot product, so the item representation cannot depend on the user. That eliminates every architecture with early user-item interaction, regardless of its accuracy, and it does so before any experiment is run.

So: a two-tower model, 256 dimensions to match the index sizing, trained with sampled softmax over in-batch negatives mixed with uniform catalogue negatives, with the logQ popularity correction. The user tower runs online inside the 25 millisecond retrieval budget; the item tower runs nightly at a cost of about 1.4 accelerator-hours for the full catalogue.

I add one thing that is not a model decision but decides the design: retrieval is **not a single tower**. I run the embedding source alongside a follows-and-recency source and a trending source, because module 6's degradation ladder needs independent sources to degrade gracefully, and because a single learned retriever has no diversity floor.

**The error most readers make here** is designing retrieval for accuracy and only afterwards asking whether it can be indexed. The index requirement comes first and removes most of the option space for free.

!!! note "Say it before you read on"

    Say what the two-tower model structurally cannot learn, and name where in the funnel that capability is recovered.

**Block 2. PURPOSE: choose the ranking model from data shape and per-item cost, since those two are what actually differ between families here.**

The per-item budget from module 6 is 90 microseconds and the corpus is 20 billion impressions with a 200 million value item identifier and a 50-item history sequence. Data shape says a network; the budget says the network must be small.

I choose a moderate network: embeddings for the identifiers with a frequency cutoff at the top 5 million items (640 MB of table, covering an assumed 95 percent of impressions), a short sequence encoder over the last 50 interactions, and a dense trunk of a few hidden layers. I size it to roughly 300 MFLOPs per item, which is 3 microseconds at 100 TFLOP/s and comfortably inside the budget.

Then I do the comparison that earns credit rather than asserting the choice: I would ship boosted trees first. They train in under an hour, they need no embedding infrastructure, and they establish the baseline. The network has to beat that baseline on an online experiment before any of its infrastructure is justified.

**The error most readers make here** is proposing the sophisticated model as the starting point. The sophisticated model is the second model. Interviewers who have shipped ranking systems are listening for whether you know that.

**Block 3. PURPOSE: decide multi-task against separate models on cost and label sparsity, not on elegance.**

Module 2 requires three heads: p(watch past 10 s) at roughly 30 percent positive rate, p(completion) at roughly 12 percent, and p(negative reaction) at roughly 0.3 percent.

The negative-reaction head is 100 times sparser than the first, which is the label-sparsity argument for sharing a trunk. The serving argument points the same way: three separate networks would triple the ranking cost, taking the ranking stage from a rounding error to a real line item.

So: one shared trunk, three heads, with gated experts if and only if a measured regression appears. I state the fallback condition explicitly, because "we will add expert gating" without a trigger is decoration: if adding the negative-reaction head degrades the watch-past-10s head by more than its experiment noise band, that is negative transfer, and gating is the response.

!!! note "Say it before you read on"

    The three heads have very different positive rates. Say what that does to the loss if you sum the three cross-entropies with equal weight, before reading block 4.

**The error most readers make here** is treating multi-task learning as free accuracy. It couples three release cadences, three regression surfaces and three calibration curves into one artefact, and that coupling is a real operational cost that has to be paid for by a real benefit.

**Block 4. PURPOSE: decide whether the heavy re-ranker earns its place, and state what happens if it does not.**

Module 3 costed the top-25 cross-encoder at 95 accelerators and $4,560 per day, about 5.5 times the rest of the online path. That is the number, and the number does not by itself decide anything.

So I frame it as an experiment with a stated decision rule, which is the correct answer to a cost question you cannot resolve from the whiteboard: run the cross-encoder on 1 percent of traffic, measure the primary metric, and convert the observed lift into revenue using the product's existing revenue per session. Launch if the lift pays for $4,560 per day with margin; otherwise distil the cross-encoder into a 0.35 GFLOP student at about $150 per day and re-measure.

The distillation branch is the interesting one, because it usually keeps most of the gain at 3 percent of the cost, and because it changes the design: a cheap student can score all 500 candidates rather than 25, which may recover more quality than the teacher lost.

**The error most readers make here** is adding the heavy re-ranker to the diagram because sophisticated systems have one. Every stage must be paid for by a measured lift, and being willing to delete a stage is scored.

**Result.** Two-tower retrieval forced by the index, boosted trees as the shipped baseline for ranking, a moderate embedding network as the challenger with a 640 MB table from a stated frequency cutoff, one shared trunk with three heads justified by a 100-fold sparsity difference and a 2.9-fold serving saving, and a re-ranking stage that only exists if an experiment pays for it.

### 8e. Fade the scaffold

**Faded example 1.** Prompt: "Choose the model for a delivery time estimate. 2 million completed deliveries per month, 80 features, all tabular, served in 30 ms, and the estimate is shown to customers so it must not be wildly wrong on any segment." Blocks 1 to 3 are given. Produce block 4 yourself.

- Block 1: no retrieval stage exists. There is one prediction per order, so the funnel does not apply and the whole model choice is a single decision.
- Block 2: 2 million rows per month, 80 tabular features. That is squarely in tree territory: a network would need feature engineering effort to match and gains nothing from an embedding.
- Block 3: the target is a duration, so this is regression. Because the estimate is shown to customers, the loss should not be plain squared error: being 10 minutes late is far worse than being 10 minutes early, which is an asymmetric cost.

??? note "Show answer"

    Block 4: choose the loss and the output form from how the number will be used, which is the whole point of this problem.

    Predict a **quantile**, not a mean. Train with pinball loss at, say, the 0.8 quantile, so the displayed estimate is one the delivery beats 80 percent of the time. That converts a vague statement about asymmetric cost into a single tunable parameter that a product manager can move and that maps directly to the late-delivery rate.

    Two additions that a strong answer makes. First, monotonic constraints: the estimate must not decrease as distance increases, and boosted tree implementations support this directly. That protects against the visibly absurd prediction that destroys user trust, and it is a genuine argument for trees over a network in a customer-facing estimate.

    Second, the segment requirement in the prompt is a **slice evaluation** requirement, not a model choice. Report pinball loss and late rate per city, per hour of day and per courier tier, and gate the release on the worst slice rather than on the average. A model that improves the mean while regressing one city is not a launch.

**Faded example 2.** Prompt: "You have a ranking model at 11 GFLOPs per item that you cannot afford. Design the plan to cut the cost by 10 times without losing more than 0.5 points of AUC." Blocks 1 and 2 are given. Produce blocks 3 and 4.

- Block 1: enumerate the levers rather than picking one. Fewer candidates scored, a smaller model, cheaper arithmetic, caching, and moving work offline are five independent axes.
- Block 2: order them by cost per engineer-week. Reducing candidates scored is a configuration change; quantizing to lower precision is days; distillation is weeks; a new architecture is months.

??? note "Show answer"

    Block 3: apply the cheap levers first and measure after each, because they compose multiplicatively and any one might be sufficient.

    Cut the re-ranked candidate set from 50 to 25 for a 2x saving, and measure the metric cost, which is usually small because items ranked 26 to 50 rarely reach the final slate. Then quantize the model's arithmetic to a lower precision format for a further saving that is typically close to free in quality. Two configuration changes may already deliver 3 to 4 times.

    Then cache. In a feed, the same item is scored for many users within a short window, but the score depends on the user, so the item's score is not cacheable. What is cacheable is the **item-side computation**, if the architecture separates item-only layers from cross layers. Restructuring so that item-side layers can be precomputed offline can remove most of the per-request cost, and it is the same idea as the two-tower split applied inside the ranker.

    Block 4: distil only if the cheap levers fall short, and state the acceptance test before training anything.

    The test: the student must be within 0.5 points of teacher AUC overall **and** within 1 point on every slice you monitor, because distillation typically loses the most on the rare cases the teacher handled well, and an average-only test hides that. Train the student on soft teacher targets over logged traffic, including impressions with no label, since that is where the teacher's extra information lives.

    The honest closing statement, which is the part most answers omit: if all five levers together do not reach 10 times, the correct outcome is to report that the target is not achievable at this quality and to present the cost-versus-quality curve so that somebody can choose a point on it. Presenting a curve rather than a single failed attempt is what makes this a design answer.

### 8f. Check yourself

**Q1.** An interviewer asks why you would not use a transformer for a 300,000 row tabular fraud problem. Answer in three sentences.

??? note "Show answer"

    At 300,000 rows and purely tabular features, boosted trees typically match or beat a network while training in minutes and requiring no embedding, no sequence handling and no accelerator, so the network's only contribution would be cost and complexity. The network's structural advantages, which are high-cardinality identifiers, sequences and transferable representations, are all absent from this feature set, so there is nothing for it to exploit.

    The condition under which I would revisit that: if the feature set gained a transaction sequence per card, or a free-text merchant description, or if the row count reached tens of millions, because each of those is a shape trees handle poorly.

**Q2.** Your two-tower retrieval model returns almost exclusively popular items. Give the most likely cause and the fix.

??? note "Show answer"

    The most likely cause is in-batch negative sampling without a popularity correction. In-batch negatives are drawn from the traffic distribution, so popular items appear as negatives far more often than rare ones, and the model over-corrects by pushing them down, or, with the opposite sign convention, the softmax over a popularity-skewed sample inflates their scores. Either way the sampling distribution rather than the data is driving the result.

    The fix is the logQ correction: subtract the log of each item's sampling probability from its logit, so the loss reflects the true distribution rather than the sampled one. The second fix, applied together, is to mix in uniformly sampled catalogue negatives, so the model sees items nobody engaged with, which is most of the index it will actually search.

    The check that confirms it: compare the popularity distribution of retrieved items against the popularity distribution of the catalogue, before and after. If the retrieved distribution is far more skewed than engagement alone explains, the sampler is the cause.

### 8g. When not to use this

| Choice | Measurement that justifies it | Threshold below which it is over-engineering | Cheaper alternative |
|---|---|---|---|
| A neural ranker instead of trees | A measured online win over a tuned boosted tree baseline, on the same features | Under about 10 million rows with no high-cardinality identifiers, sequences or unstructured features | Boosted trees plus a week of feature engineering, which frequently wins outright |
| Two-tower retrieval | Candidate pool above roughly 100,000 items, so exhaustive scoring is impossible | Under roughly 5,000 eligible items. Score them all with the ranker and delete the retrieval stage | Business-rule candidate generation: recency, follows, category, popularity |
| Multi-task learning | Either a serving cost saving above roughly 2x, or a task whose positive rate is more than roughly 10 times sparser than another's | Two tasks with similar label density and no serving pressure | Two independent models, which are far easier to debug, release and roll back |
| Distillation | Serving cost that is a material fraction of the product's margin, and a teacher whose advantage is compute rather than features | A model already inside its per-item budget | Do nothing. A model that fits its budget does not need compressing |
| A frequency cutoff on identifier embeddings | Embedding table that exceeds host memory, computed as cardinality times dimension times 4 bytes | Under about 1 million distinct identifiers, where the full table is tens of megabytes | Keep every identifier. The cutoff is a quality loss you take only when forced |

**The general principle, stated once.** Every model choice above is downstream of a constraint that already exists in the system: the index shape, the per-item budget, the row count, the feature shape, the label density. If you can name the constraint, the model choice is nearly automatic and takes a minute. If you cannot name the constraint, no amount of architecture discussion will produce a defensible answer, and the minutes are better spent on modules 2, 4 and 5.

---

---

## Module 9. Calibration and bias

**Time: 40 to 50 minutes reading, 30 to 40 minutes practice.**

This module is about the gap between a score that sorts well and a score that
means something. Almost every ML design round eventually walks into that gap,
and almost every recited pipeline answer walks straight past it.

### 9a. Predict first

Read this artefact and commit to an answer before continuing.

```
+---------------------------------------------------------------+
| Ad ranker. Slot is sold if eCPM passes a floor.               |
|   eCPM = bid_per_click x pCTR x 1000                          |
|   floor = 10.00 per 1000 impressions                          |
| Offline, held out on the next day of traffic:                 |
|   AUC 0.812     log loss 0.089                                |
| Predicted vs observed click rate by predicted decile:         |
|   decile 1    pred 0.004    obs 0.002                         |
|   decile 5    pred 0.031    obs 0.016                         |
|   decile 10   pred 0.142    obs 0.070                         |
| The ad in question bids 0.50 per click and lands in decile 5. |
+---------------------------------------------------------------+
Caption: an ad ranker's discrimination and its reliability table,
plus the auction rule that consumes the score.
```

**Question.** The model sorts ads well. Name the decision it still gets wrong,
and compute what that decision costs per million impressions.

??? note "Show answer"

    The sorting is fine. The floor test is wrong.

    Predicted eCPM is 0.50 x 0.031 x 1000 = 15.50, which clears the 10.00 floor,
    so the slot is sold. The advertiser pays per click, so realised revenue is
    0.50 x 0.016 x 1000 = 8.00 per mille. Every one of those slots earns 2.00
    per mille less than the floor you set to protect it.

    Per million impressions that is 1,000 x 2.00 = 2,000 currency units of
    inventory sold below its own reserve.

    AUC will never show you this. Every decile is over-predicted by roughly the
    same factor, and AUC is invariant to any strictly increasing transform of the
    score. A model can be perfectly ranked and arbitrarily wrong in level.

**Two properties, one number.** A score has a discrimination property (does a
higher score mean a more likely positive) and a calibration property (does the
number equal the rate). They are independent, and different consumers of the
score need different ones.

| What consumes the score | Needs ranking | Needs calibration | Why |
|---|---|---|---|
| Sort a feed with one model | Yes | No | Only order reaches the user |
| eCPM = bid x pCTR | Yes | Yes | The score multiplies a number the model never saw |
| Block if score above 0.9 | Yes | Yes | A fixed threshold moves if the level drifts |
| Blend 0.7 x pClick + 0.3 x pShare | Yes | Yes | Two heads must share units to be summed |
| Expected loss = pFraud x amount | Yes | Yes | The product is compared against a currency budget |
| Retrieve top 500 of 10 M | Yes | No | Only the cut point matters |

**The one-line rule.** Calibration matters exactly when the score is combined
with a number the model did not produce. If you can state that sentence in an
interview and then apply it, you are already above the recited-pipeline answer.

**Defining calibration so it can be measured.** A model is calibrated on a set
if, for every band of predictions near p, the observed positive rate is p.

| Measure | Formula | What it is good for | Where it hides things |
|---|---|---|---|
| Reliability table | Observed rate per predicted decile | Reading the shape of the error | Deciles are coarse at the tail |
| Expected calibration error | Sum over bins of (n_b / N) x abs(obs_b - pred_b) | One number for a dashboard | Averages away a broken slice |
| Calibration ratio | Sum of predictions divided by sum of labels | Operable, one number per slice, easy to alert on | Says nothing about shape |

Use the ratio for monitoring, the reliability table for debugging, and never
report a single global expected calibration error as evidence that an auction
is safe.

**A defensible tolerance band.** Treat a calibration ratio inside 0.95 to 1.05
as healthy. That is an operating convention, not a law, and here is the reason
it is reasonable: bid increments in most auction interfaces are coarser than 5
percent, so an error smaller than that rarely flips which ad wins. Set your own
band from the smallest decision your system can actually take.

**The four things that break calibration, and their inverses.**

| Cause | Mechanism | Inverse |
|---|---|---|
| Negative downsampling | Odds in the training set are scaled by 1 / w | Closed form, shown below. Free and exact |
| Class weighting or rebalancing | Same effect through the loss | Same closed form with the weight ratio |
| Regularisation and early stopping | Extreme logits are shrunk toward the mean | Post-hoc fit: Platt, isotonic or temperature |
| Prior shift between training and serving | The base rate itself moved | Refit the calibration layer on recent data |

**The downsampling correction, derived.** Keep every positive and keep each
negative with probability w. Sampling does not change the positives, so the odds
in the sampled set are the true odds divided by w.

- True odds: o = p / (1 - p)
- Sampled odds: o' = o / w, so the model learns p' = o / (w + o)
- To invert: o' = p' / (1 - p'), then o = w x o', then p = o / (1 + o)

Worked once, with w = 0.05 and a model output of 0.28: o' = 0.28 / 0.72 =
0.3889, o = 0.05 x 0.3889 = 0.019444, p = 0.019444 / 1.019444 = 0.01907. The
model said 28 percent. The truth is 1.9 percent. Facebook engineers published
this correction with their ad ranking system in 2014
([Practical Lessons from Predicting Clicks on Ads at Facebook](https://research.facebook.com/publications/practical-lessons-from-predicting-clicks-on-ads-at-facebook/),
ADKDD 2014, link checked 1 August 2026).

**Post-hoc calibration methods, compared honestly.**

| Method | Parameters | Data needed | Preserves ranking | Fails when |
|---|---|---|---|---|
| Platt scaling | 2 | A few thousand held-out rows | Yes, it is monotone | The true miscalibration is not sigmoidal |
| Isotonic regression | Non-parametric | Tens of thousands, more for the tail | Yes, but it creates ties | The tail bins are thin, so it overfits the extremes |
| Temperature scaling | 1 | A few thousand | Yes, exactly | The error is not a uniform logit scale |
| Per-slice binning | One table per slice | Thousands per slice | Within a slice only | Slices are thin or numerous |

Every one of these is a monotone map, so none of them can change AUC by a single
point. That is the sentence that settles the argument with whoever asks why you
bothered. Guo and colleagues documented the shrink-the-logits failure in modern
networks and the one-parameter fix in 2017
([On Calibration of Modern Neural Networks](https://arxiv.org/abs/1706.04599),
accessed August 2026).

**Position bias, stated as a model.** Under the examination hypothesis, a click
happens when the user examines the slot and the item is relevant:

P(click at slot k) = P(examine slot k) x P(relevant)

The first factor belongs to your old ranker's layout. The second is what you
want to learn. Training on raw clicks teaches the new model to reproduce the old
model's slot assignment.

**How badly, with numbers.** Take these examination propensities, measured on a
randomised slice (they are an assumption here, and the section below says how to
measure your own):

| Slot | Propensity | Item true relevance | Observed click rate |
|---|---|---|---|
| 1 | 1.00 | 0.05 | 0.050 |
| 3 | 0.45 | 0.12 | 0.054 |
| 8 | 0.13 | 0.20 | 0.026 |

The genuinely best item, at relevance 0.20, produces the lowest click rate in
the log. Fitted naively, your model learns that item is the worst of the three.
That is not a subtle statistical concern, it is an inverted label.

**Inverse propensity weighting, and what it costs.** Weight each logged event by
1 / propensity. The estimator becomes unbiased for the relevance term, and the
price is variance. Measure the price with effective sample size:

ESS = (sum of weights)^2 / (sum of squared weights)

Worked: one million clicks, half at slot 1 (weight 1) and half at slot 10
(weight 10, propensity 0.10).

- Sum of weights = 500,000 x 1 + 500,000 x 10 = 5,500,000
- Sum of squares = 500,000 x 1 + 500,000 x 100 = 50,500,000
- ESS = 5.5e6 squared / 5.05e7 = 3.025e13 / 5.05e7 = 599,000

You paid 40 percent of your data for unbiasedness. Now clip weights at 5:

- Sum of weights = 500,000 + 2,500,000 = 3,000,000
- Sum of squares = 500,000 + 12,500,000 = 13,000,000
- ESS = 9e12 / 1.3e7 = 692,000

Clipping bought back 93,000 effective rows and reintroduced a bounded bias
against deep slots. That is the whole trade, and being able to compute both
sides of it in an interview is a senior signal. Joachims and colleagues gave the
unbiased learning-to-rank formulation in 2017
([Unbiased Learning-to-Rank with Biased Feedback](https://arxiv.org/abs/1608.04468),
accessed August 2026).

**Three ways to get propensities, priced.**

| Method | What it costs | When to pick it |
|---|---|---|
| Randomised slot swaps on a traffic slice | Degrades that slice; 1 percent of traffic with a mild swap is usually tolerable | You need a clean baseline once, and you can afford one bad week for 1 percent of users |
| Intervention harvesting from natural ranker variation | Free, but needs history where the same item appeared at different slots | You already run many rankers or frequent reranks |
| Model-based estimation, fitted jointly with relevance | Free, but identifiability depends on assumptions you cannot check | Randomisation is politically impossible |

Agarwal and colleagues published the intervention-harvesting approach in 2019
([Estimating Position Bias without Intrusive Interventions](https://arxiv.org/abs/1812.05161),
accessed August 2026).

**The operational instruction that matters more than any of this.** Log the
propensity at serve time, in the same record as the impression. It is the
probability your serving policy assigned to the action it took. You cannot
reconstruct it later, because the model has been retrained, the features have
moved, and the code has changed. Teams that skip this line lose the ability to
do off-policy evaluation forever, and they usually discover it during the
quarter they most need it.

**Other named biases in the same log.**

| Bias | Mechanism | Cheapest defence |
|---|---|---|
| Trust bias | Users click top results partly because they are top | Model it as a slot-dependent relevance offset, not just an examination factor |
| Presentation bias | A thumbnail or badge changes clicks independent of quality | Include presentation features so the model does not attribute them to relevance |
| Selection bias | Items never shown have no data at all | A small random exploration budget, plus a permanent random-serving holdout |
| Popularity feedback | Shown more, so trained on more, so shown more | Exploration budget and a popularity-debiased eval set |

**Delayed feedback, stated as arithmetic.** Let F(W) be the fraction of
conversions that arrive within window W of the click. If you label with a window
W, the observed rate is the true rate times F(W).

- Measured on matured cohorts: F(1 day) = 0.62, F(7 days) = 0.88, F(30) = 1.00
- A model trained on 1-day labels predicts 0.62 of the true 30-day rate
- Multiply by 1 / 0.62 = 1.613 to recover the level

The trap is that F is not one curve. App installs mature in hours, insurance
quotes in weeks. A single global uplift over-corrects the fast segment and
under-corrects the slow one, which is worse than no correction for the segments
that matter.

**Four ways to handle it, with the architectural consequence.**

| Approach | Bias | Freshness cost | Consequence for the design |
|---|---|---|---|
| Wait for the full window | None | Model never sees the last W days | Cannot react to a new advertiser or creative for W days |
| Short window plus per-segment uplift 1 / F_s(W) | Small if segments are homogeneous | None | Needs a maintained delay curve per segment, refreshed monthly |
| Joint conversion and delay model | Small, model-dependent | None | One more model to own, and it needs its own monitoring |
| Streaming with elapsed time as a feature and label correction | Small | None | Forces a streaming trainer, which is the biggest architectural commitment on this list |

Chapelle published the joint conversion-and-delay formulation in 2014
([Modeling Delayed Feedback in Display Advertising](https://dl.acm.org/doi/10.1145/2623330.2623634),
KDD 2014, accessed August 2026).

```
+-----------------+     +------------------+     +-----------------+
| RANKER v1       | ==> | IMPRESSION LOG   | ==> | LABEL STREAM    |
| chooses slot k  |     | slot k, p_serve  |     | click now,      |
| writes p_serve  |     | features frozen  |     | convert later   |
+-----------------+     +------------------+     +--------+--------+
                                                          |
   +------------------------------------------------------+
   |
   v
+---------------------------------------------------------------+
| TRAINING SET                                                   |
|   weight = 1 / propensity(slot)     undoes position bias       |
|   uplift = 1 / F(W) per segment     undoes delay bias          |
|   odds   = w x odds_model           undoes downsampling        |
+-------------------------------+-------------------------------+
                                |
                                v
+---------------------------------------------------------------+
| RANKER v2 > CALIBRATION LAYER > eCPM = bid x pCTR x 1000       |
+---------------------------------------------------------------+
Caption: three distinct distortions enter between the serving
ranker and the training set, each with its own inverse. Fixing one
and skipping the others still leaves the auction mispriced.
```

### 9d. Worked example

**The prompt.** "We predict click probability for an ad auction. Training on the
full log is too slow, the old ranker chose the slots, and conversions arrive
days later. Make the number trustworthy."

Inputs, stated before I touch anything:

| Input | Value | Status |
|---|---|---|
| Impressions per day | 2,000,000,000 | GIVEN |
| Click rate | 2 percent | GIVEN |
| Ad slots per page | 3 | GIVEN |
| Slot propensities (1, 2, 3) | 1.00, 0.55, 0.38 | ASSUMED, from a 1 percent randomised-slot slice. Measure yours; the shape varies with layout |
| Conversion maturity F(1 day) | 0.62 | ASSUMED, from matured cohorts 45 days old. Recompute monthly |
| Auction floor | 10.00 per 1000 impressions | GIVEN |

#### Block A. Purpose: decide which properties the score must have

I start by asking what consumes the score. It is multiplied by a bid and
compared against a floor. That single fact tells me calibration is a hard
requirement, not a nicety, and it tells me AUC cannot be my ship criterion.

I say this out loud in the first two minutes, because it reframes everything
that follows: I am not building a better sorter, I am building an estimator of a
rate that other people will do arithmetic with.

**The error most readers make here.** Opening with model architecture. The
consumer of the score decides the requirements, and the requirements decide
whether a gradient boosted tree with a calibration layer beats a larger network
without one. Most of the time it does.

!!! tip "Self-explanation prompt"

    Before reading Block B, say in one sentence which of the three biases in this
    prompt would still matter if the product simply sorted three ads into three
    slots with no floor and no bid, and why.

#### Block B. Purpose: make training affordable without lying about the level

Full log is 2e9 rows per day. At 2 percent click rate that is 40 M positives and
1.96e9 negatives per day. I keep all positives and keep negatives with
probability w = 0.05, giving 98 M negatives and a training set of 138 M rows.
That is 14 times smaller, which is the difference between a 90 minute run and a
21 hour run on the same hardware.

The price is a scale distortion in the level, and the price is exactly zero
because the inverse is closed form:

```
+-------------------------------------------------------------+
| p_model  = model output on the sampled distribution         |
| o_sample = p_model / (1 - p_model)                          |
| o_true   = w x o_sample                     w = 0.05        |
| p_true   = o_true / (1 + o_true)                            |
+-------------------------------------------------------------+
Caption: the four-line inverse of negative downsampling, applied
at serve time before any auction arithmetic.
```

I apply this at serving time, not in training, so that the model file stays
independent of the sampling rate I happened to use. When someone changes w next
quarter they change one constant in the serving config.

**The error most readers make here.** Sampling negatives and then reporting log
loss on the sampled set as if it measured production quality. It does not. Log
loss on a set with an artificial base rate is not comparable to anything.

#### Block C. Purpose: remove the bias the old ranker wrote into the logs

Every training row carries the slot it was shown in, so I weight by the inverse
propensity: slot 1 weight 1.00, slot 2 weight 1.82, slot 3 weight 2.63.

I compute what this costs before deciding to do it. Suppose slots are used
evenly, so one third of rows at each weight.

- Sum of weights per 3 rows = 1.00 + 1.82 + 2.63 = 5.45
- Sum of squares per 3 rows = 1.00 + 3.31 + 6.92 = 11.23
- ESS per 3 rows = 5.45 squared / 11.23 = 29.70 / 11.23 = 2.64

So 3 rows are worth 2.64, a 12 percent loss of effective data. With 138 M rows
that is still 121 M effective rows, which is plenty. I do not clip, because the
maximum weight is 2.63 and clipping only matters when weights reach the tens.

That last sentence is the point of the whole block. With three slots, position
bias correction is nearly free. With twenty slots and a propensity of 0.05 at
the bottom, the same correction would cost most of my data and I would have to
clip, or restrict training to the top few slots, or run a small randomised slice
to get cleaner estimates.

!!! tip "Self-explanation prompt"

    Say why the ESS calculation had to come before the decision to weight, and
    what you would have done differently if it had returned 40 M effective rows
    instead of 121 M.

**The error most readers make here.** Applying inverse propensity weighting with
propensities estimated from the very logs the ranker produced, without any
randomisation or natural variation. The estimate then encodes the ranker's own
behaviour and the correction quietly does nothing.

#### Block D. Purpose: pay for conversions that have not arrived yet

Clicks are immediate, so pCTR needs no delay treatment. Advertisers billed per
acquisition need pCVR, and that label matures over days.

I choose a 1-day label plus a per-segment uplift, and I say why I rejected the
alternatives:

| Option | Why not |
|---|---|
| Wait 30 days | A new advertiser gets no model support for a month, which is a product failure, not a modelling one |
| Global uplift 1 / 0.62 | Over-corrects fast verticals and under-corrects slow ones. Wrong in both directions at once |
| Joint delay model | Correct, and I would build it if the segment table stops holding. It is a second model with its own on-call surface |

The segment table holds F_s(1 day) per (vertical, device), refreshed monthly from
cohorts at least 45 days old. Any segment with under 1,000 matured conversions
falls back to the parent vertical, because an uplift estimated on 50 conversions
has a relative standard error of 1 / sqrt(50) = 14 percent and would inject more
error than it removes.

**The error most readers make here.** Refreshing the delay curve from recent
cohorts. A cohort from last week is itself immature, so it reports a smaller
F(W) and inflates the uplift, which then over-predicts, which then raises bids.
The curve must come from cohorts older than the longest delay you believe in.

#### Block E. Purpose: pick the monitor that catches this the day it starts

One number, per slice, daily: calibration ratio = sum of predictions divided by
sum of realised labels.

Slices: vertical, slot, device, and new-advertiser-under-14-days. That is four
dimensions, not the full cross product, because the cross product produces
thousands of thin slices and a permanent alert storm.

I set the alert band from the noise, not from taste. For a slice with n
impressions and click rate p, the count of positives is about n x p and the
relative standard error of a count is 1 / sqrt(n x p).

- n = 10,000 and p = 0.02 gives 200 positives, relative error 1 / sqrt(200) =
  7.1 percent
- So a plus or minus 7 percent daily wobble on a 10,000 impression slice is pure
  noise
- The alert therefore fires on two consecutive days outside 0.93 to 1.07, and
  only for slices with more than 10,000 impressions

Two consecutive independent breaches at roughly a 5 percent one-day rate is
about 0.25 percent per slice, which across 40 monitored slices gives roughly one
false page every ten days. That is the number I would defend in a design review.

**The error most readers make here.** Alerting on the global calibration ratio.
It is the average of everything and it is the last number to move. A vertical
can be 40 percent miscalibrated for a month while the global number sits at
1.01.

### 9e. Fade the scaffold

**Fade 1: a fraud model whose score multiplies transaction amount.** Expected
loss = pFraud x amount, and a transaction is held for review when expected loss
exceeds 25 currency units. Same five-block shape, last block blank.

- Block A, which properties the score needs: the score multiplies an amount and
  is compared to a currency threshold, so calibration is required. Ranking alone
  would let the level drift and silently move the review rate.
- Block B, affordable training: fraud rate is 0.3 percent, so negatives are
  downsampled at w = 0.02 and the closed-form inverse is applied at serve time.
- Block C, bias from the old policy: transactions the old model blocked have no
  outcome label at all. This is selection bias, not position bias, and inverse
  propensity weighting cannot fix a missing outcome. A small random-release
  slice of high-score transactions is the only source of labels in that region.
- Block D, delayed labels: chargebacks arrive up to 120 days later, so the
  30-day label captures a fraction F(30) that must be measured on matured
  cohorts and inverted per segment.
- Block E, the monitor: **you write this block.**

??? note "Show answer"

    The monitored number is the calibration ratio of predicted expected loss to
    realised loss, in currency, per slice, because the decision is a currency
    comparison. A ratio on probabilities alone would miss the case where the
    model is calibrated on the many small transactions and wrong on the few large
    ones.

    Weight the ratio by amount. Slice by merchant category, amount decile, card
    country and account age under 30 days. Alert on two consecutive days outside
    band, with the band set by the noise in that slice: a slice with 200 fraud
    events has a 7 percent relative standard error, so 0.93 to 1.07.

    Add one non-calibration monitor that no ratio catches: the fraction of
    transactions in the random-release slice, and the label arrival rate on it.
    If that slice quietly gets switched off during an incident, every future
    model in this system loses its only unbiased data and nobody notices for a
    quarter.

**Fade 2: a notification ranker.** The score decides whether to send, subject to
a cap of 3 sends per user per day, and the label is "opened within 24 hours".
Last two blocks blank.

- Block A, which properties the score needs: the threshold is fixed and the cap
  makes this a per-user top-3 selection, so both ranking within a user and
  calibration across users are needed. Calibration decides how many users get
  zero sends.
- Block B, affordable training: sends are already sparse, so no downsampling is
  needed. The scale distortion here comes from class weighting, which someone
  added to fix a recall complaint. Remove it and let the threshold do that job.
- Block C, bias from the old policy: **you write this block.**
- Block D, delayed labels: **you write this block.**

??? note "Show answer"

    Block C. The old policy sent notifications only to users it already liked,
    so the log contains almost no observations for quiet users. That is
    selection bias on the user axis rather than position bias on the item axis.
    The fix is a small forced-send exploration budget, for example 1 percent of
    eligible users receive a send drawn from the whole eligible set rather than
    the top three, with the selection probability logged.

    Inverse propensity weighting on that logged probability then gives usable
    estimates in the region the old policy never visited. Without it, every new
    model inherits the old policy's blind spot and each generation narrows the
    audience further.

    Block D. A 24 hour label means the freshest 24 hours of data are unusable
    for training, which caps retrain cadence at daily rather than hourly and
    means a breaking-news topic cannot be learned inside its own news cycle.

    Measure the open-delay curve: if F(1 hour) = 0.55 and F(24 hours) = 1.00, an
    hourly trainer can use a 1 hour label with an uplift of 1.82, at the cost of
    noisier labels. Decide by the product: if content half-life is under a day,
    take the noisy fast label. If content is evergreen, take the clean slow one
    and stop pretending the pipeline needs to be hourly.

### 9f. Check yourself

**Q1.** You add isotonic regression to a ranker and offline AUC does not move by
a single point. Your product manager asks why you spent a sprint on it. Answer
in three sentences, with one number.

??? note "Show answer"

    Isotonic regression is a monotone map, so it cannot change AUC by
    construction, and an unchanged AUC is evidence the change was safe rather
    than evidence it was pointless. What changed is the level: before the fix,
    predicted rates in the top decile were roughly double the observed rate, so
    every downstream decision that multiplied the score by a bid or compared it
    to a fixed threshold was systematically wrong.

    Concretely, at a 10.00 per mille floor and a 0.50 bid, slots predicted at
    15.50 eCPM were earning 8.00, which is 2,000 currency units of under-floor
    inventory per million impressions.

**Q2.** A colleague fits position propensities using a click model trained on the
same logs your production ranker generated, with no randomisation anywhere in
the system. Name the circularity and say what visibly breaks first.

??? note "Show answer"

    The click model can only explain variation that exists in the logs, and the
    production ranker put each item at essentially one slot, so slot and item
    identity are confounded. The fitted propensity absorbs item quality, and the
    weights it produces are close to a constant rescale, which changes nothing.

    What breaks first is silent: the reweighted model reproduces the incumbent's
    ordering, offline metrics computed on the same logs improve slightly because
    both now agree with the log, and the online test comes back flat. The
    diagnostic is to check the joint distribution of item and slot: if the mutual
    information between them is high, you have no identifying variation and you
    need randomisation, natural rank churn, or an intervention-harvesting scheme
    before any of this arithmetic means anything.

### 9g. When not to use this

**A post-hoc calibration layer.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The calibration ratio on your most decision-relevant slice sits outside 0.95 to 1.05, AND the score is multiplied by or compared against a number the model did not produce |
| Scale threshold below which it is over-engineering | If the score is only ever used to sort one candidate list with one model, calibration changes nothing that reaches a user. Skip it and monitor rank metrics instead |
| Cheaper alternative | Remove the cause. Fix downsampling with the closed form, drop the class weights someone added to satisfy a recall complaint, and train on the serving distribution |

**Inverse propensity weighting for position bias.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Propensity at the bottom slot is under about 0.3, AND effective sample size after weighting stays above roughly half your row count, AND you have a source of identifying variation that is not the incumbent ranker |
| Scale threshold | With 3 or fewer slots and propensities above 0.35, the correction changes ranking decisions marginally while costing a real fraction of effective data. Below roughly 10 M labelled events, the variance usually eats the bias correction |
| Cheaper alternative | Train only on the top slots where propensity is near 1, or add slot as a feature and set it to slot 1 at inference. Both are crude, both are honest, and both are one line of code |

**A joint delayed-feedback model.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Your delay curves differ across segments by more than about 20 percentage points at the label window, so no single uplift is defensible, AND conversions matter to billing |
| Scale threshold | If the fastest and slowest segments both reach F(W) above 0.9 at your window, a per-segment constant is within a few percent and a second model is not worth its on-call rota |
| Cheaper alternative | A refreshed table of per-segment uplift factors, computed monthly from matured cohorts, with a documented fallback to the parent segment when a cell is thin |

**A permanent randomised exploration slice.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You can name a decision you cannot currently evaluate because the incumbent policy never takes it, and you can price the slice: 1 percent of traffic at a measured quality loss per exposed user |
| Scale threshold | Below roughly 1 M daily decisions, a 1 percent slice yields too few events per slice to estimate anything, and you are paying quality for noise. Use scheduled ranker swaps instead |
| Cheaper alternative | Log the serving propensity for the policy you already run, and harvest the natural variation from reranks, model rollouts and tie-breaks. It is free and it is usually enough to start |
---

## Module 10. Offline evaluation

**Time: 35 to 45 minutes reading, 25 to 35 minutes practice.**

Offline evaluation is where most candidates are weakest and most confident. The
recited answer names three metrics. The strong answer picks one metric because
of what the system does with the score, then spends its time on the split, the
slices and the errors.

### 10a. Predict first

```
+---------------------------------------------------------------+
| System: retrieve 500 of 8 M items, then rank, then show 10.   |
| Change under test: the RANKER only. Retrieval is untouched.   |
| Offline results, new model versus incumbent:                  |
|   AUC          0.781  ->  0.802     (+2.7 percent)            |
|   PR-AUC       0.061  ->  0.058     (-4.9 percent)            |
|   NDCG at 10   0.312  ->  0.319     (+2.2 percent)            |
|   recall at 100  0.71 ->  0.70      (-1.4 percent)            |
|   log loss     0.214  ->  0.219     (worse)                   |
| Evaluation set: random 10 percent of the last 90 days.        |
+---------------------------------------------------------------+
Caption: five offline numbers for a ranker change in a two-stage
funnel, with the split that produced them.
```

**Question.** Which single number should decide whether this ships, and which
line of the artefact makes all five numbers untrustworthy?

??? note "Show answer"

    The deciding number is NDCG at 10, because the surface shows 10 items and the
    change is the ranker. Recall at 100 measures the retrieval stage, which was
    not changed, so its movement is noise or leakage. AUC integrates over
    thresholds this system never uses. Log loss matters only if something
    downstream multiplies the score, which the prompt does not say. PR-AUC
    falling while AUC rises is a useful warning that the gain sits in the easy
    negative region, but it is not the ship criterion here.

    The line that poisons all five: "random 10 percent of the last 90 days". A
    random split puts tomorrow's events in the training set and yesterday's in
    the test set. Any feature aggregated over the window, such as a user's 30-day
    click rate, is then partly computed from the row being scored. Split by time,
    then re-run everything, then argue about which metric wins.

**Metric selection starts from the decision, not the task type.** Write down the
sentence "the system uses this score to ..." and finish it. The metric follows.

| What the system does with the score | Metric that matches | Metric that lies here |
|---|---|---|
| Shows the top 10 of 500 | NDCG at 10, precision at 10 | AUC over all pairs |
| Retrieves 500 of 8 M | Recall at 500 | NDCG, which ignores what was never retrieved |
| Blocks above a fixed threshold | Precision and recall at that threshold, and at plus or minus 20 percent of it | AUC |
| Multiplies by a bid | Log loss and calibration ratio | Any pure ranking metric |
| Sends at most 3 per user per day | Precision at 3, per user, then averaged | Global precision, which lets heavy users dominate |
| Promises a delivery time | Pinball loss at the promised quantile, plus MAE | RMSE, when being 10 minutes late costs far more than 10 minutes early |

**Why AUC lies under imbalance, derived.** Take 1,000,000 events with 1,000
positives, a prevalence of 0.1 percent. Suppose the model achieves AUC 0.9, and
at the operating point you would actually pick, true positive rate is 0.8 and
false positive rate is 0.1.

- True positives = 0.8 x 1,000 = 800
- False positives = 0.1 x 999,000 = 99,900
- Precision = 800 / 100,700 = 0.79 percent

A reviewer would work 100,700 cases to catch 800. To reach 10 percent precision
you need false positives under 7,200, so a false positive rate under 0.0072.
Whether the model can do that is a question about one point on the curve, and
AUC never tells you. Ask for precision at the recall you need, or the partial
area over the region you would operate in.

**Ranking metrics, defined before they are used.**

| Metric | Definition in one line | The trap |
|---|---|---|
| Precision at k | Fraction of the top k that are relevant | Ignores order inside the top k |
| Recall at k | Fraction of all relevant items that appear in the top k | Meaningless if the relevant set is unbounded |
| NDCG at k | Sum of gain divided by log of rank, normalised by the ideal ordering | With one relevant item and binary labels it is nearly MRR wearing a suit |
| MRR | Reciprocal of the rank of the first relevant item | Blind to everything after the first hit |
| MAP | Mean over queries of average precision | Sensitive to the tail, so it moves when nobody notices |

**The structural limit nobody states.** All of these are computed on logged
impressions, so they measure re-ordering of what the incumbent already
retrieved. A new model whose whole value is surfacing items the incumbent never
showed will score zero improvement, or worse, on every one of them. That is not
a metric bug, it is the reason online evaluation exists.

**Splits, and the leakage arithmetic.** Split by time for anything with temporal
features, by entity for anything with grouped rows, and by both when both apply.

Concrete leakage: a feature "user click rate over the trailing 30 days"
computed once over the whole window, on a user with 20 events. The scored event
contributes 1 of 20 observations to its own feature, so 5 percent of the feature
is the label. For a user with 2 events it is 50 percent. Sparse users are
exactly the cohort you care about, and they are exactly where the leak is worst,
which is why leaked evaluations flatter new-user performance most.

**The evaluation split ladder.**

| Split | Use it when | What it still misses |
|---|---|---|
| Random row | Almost never in a production system | Time, entity, and policy shift, all three |
| Time-based, train before a cut, test after | Default for any online system | Seasonality, if the test window is short |
| Time plus entity, no user in both sides | Personalisation and cold start questions | The new policy's own effect on the data |
| Forward-chaining across several cuts | You need a variance estimate on the offline delta | Still offline |

**Slice-based evaluation, and the arithmetic that makes it mandatory.** Define
your slices before the run, five to eight of them, each tied to a named risk.

Suppose overall click rate improves 1.02 percent while new users regress:

- New users are 15 percent of traffic, effect minus 3.4 percent
- Returning users are 85 percent, effect plus 1.8 percent
- Weighted: 0.15 x (-3.4) + 0.85 x (1.8) = -0.51 + 1.53 = +1.02

The aggregate is positive and the cohort that drives growth is down 3.4 percent.
This is not an exotic case. It is what happens whenever a model gets better at
exploiting history, because the cohort with no history loses.

**A default slice set for a ranking system.**

| Slice | Risk it covers |
|---|---|
| Account age under 7 days | Cold start, the cohort with no features |
| Head, torso and tail by item popularity | Popularity collapse and catalogue coverage |
| Query or session length | Short ambiguous inputs versus long specific ones |
| Locale or language | Silent failure outside the majority market |
| Device class | Latency-coupled quality differences |
| The 1 percent random-serving holdout | The only unbiased view you own |

**Error analysis as a design skill.** Sample 100 errors stratified across your
slices, read them, and tag each by cause. The output is not a model idea, it is
a budget allocation.

| Cause found | Count in a typical first pass | Cheapest intervention | Time to fix |
|---|---|---|---|
| Label is wrong or the label definition is wrong | 22 | Fix the definition, relabel a sample | Days |
| Feature missing or stale at serve time | 18 | Repair the serving path, add a null monitor | Weeks |
| Genuinely ambiguous, two right answers | 25 | Change the product surface, not the model | A quarter |
| Too little data in this slice | 20 | Targeted collection or oversampling | Weeks |
| Model capacity or features insufficient | 15 | New model or new features | A quarter |

Those counts are illustrative, not measured, and you should produce your own.
The structural point survives whatever numbers you get: in most first passes,
the majority of errors are not model problems. A candidate who says "I would
pull 100 errors and count causes" ranks above one who says "I would try a
larger model", because the first is a decision procedure and the second is a
guess.

**How precise is a count of 100?** A cause seen 22 times has a standard error of
about sqrt(22) = 4.7, so roughly 22 plus or minus 9 at 95 percent confidence.
That means 100 samples can separate a 22 percent cause from a 5 percent cause
and cannot separate it from a 15 percent cause. Sample 400 if you need that
resolution, and do not build a roadmap on a difference of five errors.

**What an offline win does not tell you.** Say this list out loud in the
interview and you will sound like someone who has shipped.

| Blind spot | Why offline cannot see it |
|---|---|
| The new policy changes the data it collects | Your eval set was generated by the old policy |
| Items the incumbent never showed | They have no labels, at any position |
| Latency and cost | The offline harness has no clock and no bill |
| Novelty and habituation | Users respond to change itself, then stop |
| Cannibalisation across surfaces | The metric is scoped to one surface |
| Marketplace interference | Supply is finite and one user's win is another's loss |
| Whether this metric predicts the business at all | That is an empirical property of your metric, and it is measurable |

**Measure your own offline-to-online agreement.** Take your last ten
experiments, record the sign of the offline delta and the sign of the online
primary metric, and count agreements. If they agree 6 times out of 10, that is
not evidence of anything: under a coin flip, 6 or more agreements happens about
38 percent of the time. Agreement of 9 or 10 out of 10 is a signal. Teams that
have never done this count are guessing about the value of their whole offline
stack.

**Off-policy estimation, at interview depth.** If you logged the serving
propensity (Module 9), you can estimate a new policy's value from old data:

V = (1 / n) x sum over logged events of (pi_new(a) / mu_logged(a)) x reward

Two rules make it honest. First, report effective sample size alongside the
estimate; if ESS is under about 10 percent of n, report a direction and refuse
to report a number. Second, the estimate is only valid where the logging policy
had non-zero probability, so a new policy that promotes items the old one never
showed cannot be evaluated this way at all.

```
+----------------------------------------------------------+
| STEP 1  LEAK CHECK      time split, entity split,         |
|                         point in time features            |
+----------------------------------------------------------+
                          |
                          v
+----------------------------------------------------------+
| STEP 2  MATCHED METRIC  one metric chosen from what the   |
|                         system does with the score        |
+----------------------------------------------------------+
                          |
                          v
+----------------------------------------------------------+
| STEP 3  SLICES          5 to 8 named slices, each tied    |
|                         to a risk, all reported           |
+----------------------------------------------------------+
                          |
                          v
+----------------------------------------------------------+
| STEP 4  ERROR SAMPLE    100 errors, tagged by cause,      |
|                         counts drive the next sprint      |
+----------------------------------------------------------+
                          |
                          v
+----------------------------------------------------------+
| STEP 5  DECISION        ship to online test, or fix, or   |
|                         stop. Offline never ships alone   |
+----------------------------------------------------------+
Caption: the offline evaluation ladder. A step skipped early
invalidates every step after it, so the order is the method.
```

### 10d. Worked example

**The prompt.** "Your team has a new ranker with plus 2.2 percent NDCG at 10.
Decide whether it goes to an online test, and defend the decision."

Inputs:

| Input | Value | Status |
|---|---|---|
| Surface | Top 10 of 500 candidates | GIVEN |
| Traffic | 40 M ranked requests per day | GIVEN |
| Incumbent NDCG at 10 | 0.312 | GIVEN |
| Candidate NDCG at 10 | 0.319 | GIVEN |
| Eval window | 90 days | GIVEN |
| Cost of an online test | Two weeks of one experiment slot | ASSUMED. Slots are the scarce resource in most teams, which is why the offline gate exists at all |

#### Block A. Purpose: destroy the result before trusting it

I do not start by asking whether 2.2 percent is a lot. I start by trying to
break the measurement, because a wrong number is worse than no number.

1. Was the split temporal? If it was random, stop, re-run, and expect the gain
   to shrink. I have never seen it grow.
2. Were features computed as of the request time, or with any aggregate that
   spans the label? This is the point-in-time join question from the feature
   store module, and it is the single most common real bug in this space.
3. Was the eval set built from the incumbent's own impressions? Then the metric
   measures re-ordering only, and a model that finds new items is penalised.
4. Are the two models scored on identical candidate sets? If the new ranker also
   changed filtering, NDCG is comparing two different problems.

**The error most readers make here.** Treating the offline number as the finding
and the split as a detail. The split determines whether the number exists.

!!! tip "Self-explanation prompt"

    Before Block B, say which of those four checks would still matter if the
    system were a fraud classifier with a fixed threshold rather than a ranker,
    and which would become irrelevant.

#### Block B. Purpose: size the gain against its own noise

A delta with no interval is a rumour. I compute a paired estimate: for each
query in the eval set, NDCG under the new model minus NDCG under the incumbent,
then the mean and standard error of that per-query difference.

With 200,000 eval queries and a per-query difference standard deviation of 0.18
(this is measured, not assumed, and 0.18 is typical only in the sense that
per-query NDCG is highly variable):

- Standard error = 0.18 / sqrt(200,000) = 0.18 / 447 = 0.0004
- Observed delta = 0.319 minus 0.312 = 0.007
- Ratio = 0.007 / 0.0004 = 17.4 standard errors

So the offline gain is real as a measurement. That says nothing about whether it
is real as a product effect, and I say both halves of that sentence out loud,
because candidates who say only the first sound naive and candidates who say
only the second sound obstructive.

**The error most readers make here.** Reporting the aggregate delta with no
variance at all, then being unable to answer "is that within noise?".

#### Block C. Purpose: find out who paid for the average

I break the delta by the six slices above. Suppose the table comes back:

| Slice | Share of traffic | NDCG at 10 delta |
|---|---|---|
| Account age under 7 days | 15 percent | minus 3.4 percent |
| Returning users | 85 percent | plus 2.2 percent |
| Tail items, bottom 80 percent by popularity | n/a | minus 5.1 percent coverage in top 10 |
| Locale other than the top market | 22 percent | plus 0.4 percent |

Now the decision has changed shape. The aggregate is a growth-negative trade:
better for users with history, worse for users without, and it concentrates
impressions on popular items. That last row is a feedback-loop risk, not a
metric quibble, because next month's training data will contain even fewer tail
impressions.

**The error most readers make here.** Reporting slices and then shipping anyway
because the aggregate is positive. The slice table is only useful if it can
change the decision.

!!! tip "Self-explanation prompt"

    Say what you would need to see in the new-user slice to ship this anyway, and
    name the number that would have to move.

#### Block D. Purpose: read 100 errors before proposing a fix

I pull 100 new-user queries where the incumbent put a relevant item in the top
10 and the candidate did not. Tagging them:

- 41 have almost no user features, and the candidate leans harder on
  personalisation features that are null for these users
- 26 are ambiguous queries where both orderings look defensible
- 18 have a stale item-popularity feature computed on a daily batch
- 15 are genuine quality losses

That distribution names the fix precisely: the candidate degrades badly under
null features. The cheap repair is not a new architecture, it is a null-handling
change plus a cold-start path, and I can estimate its value at up to 41 percent
of the regression before writing any code.

**The error most readers make here.** Jumping from "new users regressed" to "add
a new-user model". Two thirds of this regression is null handling and stale
features, both of which are cheaper and land sooner.

#### Block E. Purpose: state the decision and the condition to revisit it

My decision: do not take the experiment slot yet. Fix null handling, re-run the
slice table, and require the new-user slice to be no worse than minus 0.5
percent before the online test.

I write the ship rule now, before the next result exists, because a threshold
chosen after seeing the number is not a threshold. I also write what the online
test will measure that offline cannot: tail item coverage over time, and the
new-user retention effect, neither of which appears in NDCG.

**The error most readers make here.** Saying "needs more work" without a numeric
re-entry condition. A decision with no threshold attached becomes a negotiation
next week, and the person with the most patience wins it rather than the person
with the evidence.

### 10e. Fade the scaffold

**Fade 1: a moderation classifier.** New model shows PR-AUC 0.61 to 0.66 on a
held-out month. It runs at a fixed threshold that currently sends 2 percent of
content to human review. Last block blank.

- Block A, destroy the result first: check that the eval month is after the
  training window, that appeals-driven relabels have not been folded back into
  the test set, and that the test set is not the rebalanced 50-50 set used for
  training.
- Block B, size the gain: PR-AUC moved, but the decision runs at one threshold,
  so recompute precision and recall at the threshold that holds review volume at
  2 percent, with a bootstrap interval over the positive class.
- Block C, who paid: slice by language, content type and reporter type. A gain
  concentrated in the majority language while a minority language regresses is a
  shipping blocker even at a positive aggregate.
- Block D, read the errors: sample 100 false negatives at the operating
  threshold and tag by policy category.
- Block E, decision and revisit condition: **you write this block.**

??? note "Show answer"

    The decision is a threshold decision, not a model decision. State it as:
    ship only if, at fixed review capacity of 2 percent of content, recall on the
    highest-harm policy category is not lower than the incumbent and the worst
    language slice is within 1 percentage point of recall parity.

    Write the revisit condition in terms of the operating point, since that is
    what production has: if review capacity changes, the whole comparison is void
    and must be recomputed at the new threshold, because two models can swap
    order between operating points even with the same PR-AUC.

    Name the thing offline cannot tell you here: the adversary adapts. A
    moderation model's offline win decays because the population producing the
    content responds to enforcement, so the online plan must include a decay
    measurement, not just a launch readout.

**Fade 2: an ETA model.** New model shows RMSE 4.8 minutes to 4.4 minutes on 30
days of held-out trips. The number shown to users is a promise, and being late
generates support contacts. Last two blocks blank.

- Block A, destroy the result first: split by time and by city, confirm no trip
  appears in both sides through a shared driver-level feature, and confirm the
  features are as of dispatch time rather than as of trip completion.
- Block B, size the gain: **you write this block.**
- Block C, who paid: **you write this block.**

??? note "Show answer"

    Block B. RMSE is the wrong loss for an asymmetric cost, so the size of the
    gain must be recomputed in the units of the decision. If the product
    promises a time that should be met 80 percent of the time, evaluate pinball
    loss at the 0.8 quantile and report the realised coverage: what fraction of
    trips arrived by the promised time.

    A model can improve RMSE by tightening the middle of the distribution while
    making the late tail worse, which raises support contacts even as the
    headline metric improves. Report late-tail mass explicitly: the fraction of
    trips more than 10 minutes past promise.

    Block C. Slice by city, by time of day, by trip length and by whether the
    trip crosses a known bottleneck. Long trips and peak hours carry the
    asymmetric cost, and a model tuned on the volume-dominant short urban trips
    will look excellent overall while degrading exactly the trips where a
    late arrival costs a support contact. Also slice by new drivers, whose
    history features are null, because that is the ETA equivalent of the
    cold-start regression in the worked example.

### 10f. Check yourself

**Q1.** A teammate reports AUC 0.94 on a fraud model with a 0.2 percent base
rate and proposes shipping. What two numbers do you ask for, and what will you do
with them?

??? note "Show answer"

    Ask for precision and recall at the operating threshold the review team can
    actually staff, and the daily volume that threshold implies. With 1 M daily
    transactions and 0.2 percent fraud, there are 2,000 fraud events. If the
    chosen point catches 70 percent of them at a 1 percent false positive rate,
    that is 1,400 caught and 9,980 false alerts, so precision is 12.3 percent
    and a reviewer team sees 11,380 cases a day. Whether that ships is a
    staffing question, and AUC never touches it.

    Then ask for the same two numbers on the highest-amount decile, because the
    currency loss is concentrated there and the count-based metrics are not.

**Q2.** Your offline metric and your online metric have agreed in sign on 6 of
your last 10 experiments. What does that tell you, and what would you do next?

??? note "Show answer"

    It tells you almost nothing. Under pure chance, 6 or more agreements out of
    10 occurs about 38 percent of the time, so this result is indistinguishable
    from a coin. Do not conclude the offline metric is useless, and do not
    conclude it works.

    Next steps, in cost order: increase the sample by including experiments you
    stopped early, since those are informative too; check whether disagreement
    concentrates in one experiment type, for example whether retrieval changes
    agree and ranking changes do not; and inspect whether the offline metric is
    computed on impressions from the incumbent, which caps its ability to
    predict any change that alters what is shown.

    If after 25 experiments agreement is still near half, replace the offline
    gate with a cheaper online mechanism such as interleaving, and stop paying
    for the harness.

### 10g. When not to use this

**A full slice matrix on every evaluation.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You can name a past incident or a real risk for each slice, and each slice has enough events for a stable estimate: at least a few hundred positives |
| Scale threshold below which it is over-engineering | Below roughly 100,000 evaluation events, most slices are noise and you will chase phantoms. Report two or three slices with real mass |
| Cheaper alternative | One cold-start slice and one tail-item slice. Those two catch the majority of the regressions that aggregates hide |

**Off-policy evaluation with importance weights.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Serving propensities are logged, the new policy's support is contained in the old one, and effective sample size stays above roughly 10 percent of your row count |
| Scale threshold | If the new policy differs sharply from the logger, weights explode and the estimate has enormous variance. Below about 1 M logged decisions with propensities, skip it |
| Cheaper alternative | A short online interleaving test, which measures a preference directly and needs far less traffic than a full A/B test |

**Formal error analysis on a sample of 100.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You are about to spend more than a sprint on a fix, and you cannot state which of the five error causes dominates |
| Scale threshold | If the model has fewer than a few hundred errors in total, read all of them instead of sampling. Sampling is for when reading everything is impossible |
| Cheaper alternative | Read 20 errors informally before the planning meeting. It is unreliable as a percentage but it reliably kills at least one bad roadmap item |

**A curated golden evaluation set maintained by hand.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Your logged evaluation cannot see the change under test, for example a retrieval change that surfaces never-shown items, and you have annotator capacity for periodic refresh |
| Scale threshold | Under a few thousand judged items, the set has no statistical power for small deltas and mainly serves as a regression tripwire. That is a legitimate use, but call it that |
| Cheaper alternative | A frozen sample of production traffic replayed for score-distribution comparison, which catches gross regressions with no annotation cost at all |
---

## Module 11. Online evaluation

**Time: 40 to 50 minutes reading, 30 to 40 minutes practice.**

Offline evaluation asks whether the model is better. Online evaluation asks
whether the system with the model in it is better, for real users, at a cost you
can measure. They are different questions and they disagree often enough that
handling the disagreement well is a senior-level signal by itself.

### 11a. Predict first

```
+---------------------------------------------------------------+
| Experiment readout, 14 days, 1 percent of users per arm.      |
| Randomisation unit: user. Analysis unit: impression.          |
| Primary metric declared before launch: none                   |
| Metrics reported: 14                                          |
|   click rate         +0.9 percent   p = 0.04                  |
|   sessions per user  +0.2 percent   p = 0.61                  |
|   revenue per user   -0.4 percent   p = 0.33                  |
|   ... 11 more, all p above 0.10                               |
| Assignment counts: control 501,900   treatment 499,100        |
+---------------------------------------------------------------+
Caption: a typical experiment readout with no declared primary
metric, a unit mismatch, and its assignment counts.
```

**Question.** Compute the probability that at least one of 14 metrics with no
true effect shows p below 0.05, and name the second thing in this readout that
would invalidate it even if only one metric had been reported.

??? note "Show answer"

    With 14 independent null metrics at a 5 percent level, the chance of no false
    positive is 0.95 to the power 14 = 0.4877, so the chance of at least one is
    51.2 percent. A single metric at p = 0.04 out of 14 is what a coin looks
    like. Declaring the primary metric before launch is what converts that into
    evidence.

    The second problem is the unit mismatch. Users were randomised, impressions
    were analysed. Impressions from one user are correlated, so the standard
    error computed at impression level is too small. With 20 impressions per user
    and an intra-user correlation of 0.1, the design effect is 1 + (20 - 1) x 0.1
    = 2.9, so true standard errors are sqrt(2.9) = 1.7 times larger. A nominal 5
    percent test is really rejecting about 25 percent of the time under the null.

    A third issue is worth flagging: 501,900 versus 499,100 is a 0.28 percent
    imbalance on a million users, which is itself suspicious. See the sample
    ratio check below.

**Pick the randomisation unit from where interference lives.** Interference
means one unit's treatment affects another unit's outcome. Randomise at or above
the level where it happens.

| System | Interference source | Unit |
|---|---|---|
| Feed or search ranking | Usually none between users | User |
| Marketplace pricing or dispatch | Supply is shared and finite | Region and time window |
| Social features, invitations, sharing | The network itself | Cluster of connected users, or none |
| Ranker A versus ranker B on quality only | None | Query, using interleaving |
| Caching or infrastructure change | Shared capacity | Server or shard, with care |

Randomising a marketplace change by user is the classic error: treatment users
take the supply that control users then cannot have, so the measured effect
contains a transfer as well as a creation.

**Power, derived once so you can redo it at the whiteboard.** For a two-arm test
with equal allocation, 80 percent power and a two-sided 5 percent level, the
required sample per arm is:

n = 2 x (1.96 + 0.84)^2 x variance / effect^2, which is about 16 x variance /
effect^2

For a binary metric at a 2 percent base rate, variance is p(1 - p) = 0.0196.

| Relative lift you want to detect | Absolute effect | n per arm | Days at 250 k users per arm per day |
|---|---|---|---|
| 1 percent | 0.0002 | 7.84 M | 31 |
| 2 percent | 0.0004 | 1.96 M | 8 |
| 5 percent | 0.0010 | 314 k | 1.3 |
| 10 percent | 0.0020 | 78 k | 0.3 |

Read the table backwards, which is how it is actually used. If the team can only
run for a week at this traffic, the smallest honest question is "is this better
by at least 2 percent?". Running a 1 percent question for a week and reporting
"no significant difference" is a statement about your sample size, not about the
feature.

**The correction most readouts skip.** That table assumes the metric's variance
is p(1 - p), which holds for a single Bernoulli draw. Your metric is usually a
per-user rate, whose variance across users is larger because users differ. In
practice, multiply the required sample by the design effect measured from an A/A
test on your own traffic. Do not assume the factor; measure it once and reuse
it, and re-measure when the product changes.

**Variance reduction buys time, and the arithmetic is exact.** CUPED adjusts
each user's metric using their pre-period value:

Y_adjusted = Y - theta x (X_pre - mean of X_pre), with theta =
covariance(Y, X_pre) / variance(X_pre)

The adjusted variance is (1 - correlation squared) times the original. Sample
size scales with variance, so:

| Correlation between pre-period and in-period metric | Variance remaining | Sample or days saved |
|---|---|---|
| 0.3 | 0.91 | 9 percent |
| 0.5 | 0.75 | 25 percent |
| 0.7 | 0.51 | 49 percent |
| 0.8 | 0.36 | 64 percent |

Deng and colleagues published this method in 2013
([Improving the Sensitivity of Online Controlled Experiments by Utilizing
Pre-Experiment Data](https://dl.acm.org/doi/10.1145/2433396.2433413), WSDM 2013,
accessed August 2026). The catch is structural: new users have no pre-period, so
the technique helps least in the cohort where you most often need power.

**One primary, a few guardrails, everything else is a diagnostic.**

| Role | Count | Decision authority | Test shape |
|---|---|---|---|
| Primary | 1 | Ships or does not ship | Two-sided, declared before launch |
| Guardrail | 2 to 4 | Blocks a ship if breached | One-sided non-inferiority with a stated margin |
| Diagnostic | As many as you like | None | Reported without p-values, used for explanation |

A guardrail needs a margin, not just a direction. "Latency p95 must not increase
by more than 5 ms" is testable. "Latency must not regress" is not, because
noise guarantees some regression somewhere.

**Sample ratio mismatch is the cheapest invalidity check you own.** Compare
observed assignment counts to intended, with a chi-square test.

Worked, from the artefact: total 1,001,000 with 501,900 and 499,100 gives
expected 500,500 each. Deviation 1,400 each, so chi-square = 2 x (1,400^2 /
500,500) = 2 x 3.915 = 7.83, and for one degree of freedom that is about p =
0.005. That is a red flag, not a rounding artefact.

Common causes are assignment-time filtering that differs by arm, a treatment
that crashes for some users before logging, and bot traffic landing unevenly.
Fabijan and colleagues catalogued the causes in 2019 ([Diagnosing Sample Ratio
Mismatch in Online Controlled
Experiments](https://dl.acm.org/doi/10.1145/3292500.3330722), KDD 2019, accessed
August 2026). A failed ratio check voids the readout. Do not analyse further,
find the bug.

**Interleaving, and its four hard limits.** Interleaving merges two rankers'
results into one list and attributes each click to whichever ranker contributed
the item. Because each user sees both rankers, between-user variance cancels and
the comparison needs far less traffic than a split test.

| Limit | Consequence |
|---|---|
| It compares two rankers only | No absolute metric, no effect size in business units |
| It cannot measure revenue, retention or long-term effects | Those need a real assignment |
| It breaks when the two systems have different candidate sets or layouts | The merge is no longer fair |
| It measures preference, not value | A ranker can win interleaving and lose the business metric |

Chapelle and colleagues published a large-scale validation of interleaved
evaluation in 2012
([Large-Scale Validation and Analysis of Interleaved Search
Evaluation](https://dl.acm.org/doi/10.1145/2094072.2094078), TOIS 2012, accessed
August 2026). Their sensitivity results are theirs, measured on their traffic;
run your own comparison before quoting a factor.

**Switchbacks, for systems where users are not independent.** Randomise
time-region cells to treatment or control, then alternate.

Design rules, each with its reason:

| Rule | Reason |
|---|---|
| Period length longer than the carryover you can measure | Effects that leak across the boundary bias both arms toward each other |
| Discard a burn-in at each switch | The first minutes after a switch contain the old regime's state |
| The unit of analysis is the period, not the trip | Trips inside a period are not independent |
| Balance periods across time of day and day of week | Demand cycles will otherwise be confounded with the arm |

The arithmetic that surprises people: 5 regions, 14 days, 4-hour periods gives 5
x 14 x 6 = 420 periods, so 210 per arm. If period-level metric variation has a
coefficient of variation of 8 percent, the minimum detectable relative effect is
about 2.8 x 0.08 x sqrt(2 / 210) = 2.8 x 0.08 x 0.0976 = 2.2 percent. Two
million trips, and you can only see a 2.2 percent effect, because your sample is
420 periods.

**Bandits, and the honest scope.**

| Use a bandit when | Do not, when |
|---|---|
| Arms are numerous and short-lived, such as creatives or thumbnails | You need an unbiased effect size for a decision review |
| Reward arrives in seconds or minutes | Reward is a 30-day conversion |
| The cost of showing a bad arm is high per impression | The main cost is a wrong permanent decision, which a bandit does not reduce |
| Arms come and go continuously | The system has interference a bandit cannot randomise around |

The regret argument is weaker than it sounds for two arms. A two-week split test
at 50 percent allocation with a true 2 percent effect wastes 2 percent of half
the traffic for two weeks. If the decision it informs runs for a year, that is
about 0.2 percent of the annual metric. With 200 arms, the same argument
reverses completely, because a split test never reaches power and the bandit is
the only mechanism that converges.

**When online disagrees with offline, work this table top to bottom.**

| Cause | Diagnostic that separates it |
|---|---|
| Sample ratio mismatch | Chi-square on assignment counts. Do this first, always |
| Metric mismatch, offline metric does not match the surface | Recompute the offline metric at the exact k the product shows |
| Training-serving skew | Replay logged serving features offline and compare model outputs row by row |
| Position bias in the offline set | Re-evaluate with propensity weights, or on the random-serving holdout |
| Latency regression eating the gain | Compare treatment and control p95 latency, then check your measured latency-to-metric exchange rate |
| Novelty or primacy | Plot the treatment effect by day. A monotone decay over the first week is the signature |
| Underpowered test | Compute the minimum detectable effect the test actually had before calling it flat |
| Cannibalisation | Check the neighbouring surface's metrics, not just yours |
| Interference | Check whether the effect size changes with treatment allocation, for example 5 percent versus 50 percent |

**Novelty, quantified.** Suppose the daily treatment effect runs plus 4.1, plus
2.6, plus 1.8, plus 1.4, plus 1.2, plus 1.1, plus 0.9 percent over seven days.
The asymptote near 1 percent is the number to plan with, not the 2.2 percent
two-week average. If your readout averages a decaying series and reports the
mean, you will systematically over-forecast every launch.

```
+---------------------------------------------------------------+
| Does one user's treatment change another user's outcome?      |
+---------------------------------------------------------------+
       | no                                    | yes
       v                                       v
+---------------------------+   +-------------------------------+
| Do you need a business    |   | Is the shared resource        |
| effect size in currency   |   | bounded by geography or time? |
| or retention?             |   +-------------------------------+
+---------------------------+        | yes            | no
   | yes         | no                v                v
   v             v          +----------------+  +----------------+
+---------+  +-----------+  | SWITCHBACK     |  | CLUSTER TEST   |
| A/B     |  | INTERLEAVE|  | unit is a      |  | unit is a      |
| test    |  | ranker vs |  | region-time    |  | network        |
| by user |  | ranker    |  | cell           |  | component      |
+---------+  +-----------+  +----------------+  +----------------+
Caption: choosing the online method from the interference question
first and the metric question second. Arm count and reward delay
then decide whether a bandit replaces the chosen design.
```

### 11d. Worked example

**The prompt.** "The ranker from the offline module passed its gate. Design the
online test, then tell me what you do if click rate is up and revenue is flat."

Inputs:

| Input | Value | Status |
|---|---|---|
| Daily active users | 5 M | GIVEN |
| Ranked requests per day | 40 M | GIVEN |
| Baseline click rate | 2.0 percent | GIVEN |
| Expected effect, from the offline delta | 2 percent relative, and I distrust it | DERIVED, offline deltas usually shrink |
| Experiment slots available | 1, for 2 weeks | GIVEN |
| Pre-period to in-period metric correlation | 0.65 | ASSUMED, measured once from an A/A test. Re-measure per metric |

#### Block A. Purpose: choose the unit and the primary metric before anything else

Interference: this is feed ranking with no shared inventory, so one user's
ranking does not consume another's. Unit is the user.

Primary metric: I do not pick click rate. Clicks are the model's objective, so a
model trained to maximise clicks will raise clicks almost by construction, and
the interesting question is whether anything else moves. I declare revenue per
user as primary and click rate as a diagnostic, with two guardrails: p95 latency
must not rise by more than 15 ms, and 7-day return rate must not fall by more
than 0.3 percentage points.

I write this in the experiment document before launch and I say so, because a
primary metric chosen after the readout is not a primary metric.

**The error most readers make here.** Choosing the metric the model optimises as
the primary metric of the experiment. That converts the test into a check that
the training loop works.

!!! tip "Self-explanation prompt"

    Before Block B, say what would have to be true about this product for click
    rate to be a defensible primary metric, and name the type of product where it
    genuinely is.

#### Block B. Purpose: size it, then negotiate the question to fit the traffic

Revenue per user has higher variance than a click indicator, so I use a measured
coefficient of variation from history rather than p(1 - p). Assume it is 2.2,
which I would take from the last quarter of data.

For a relative effect d on a metric with coefficient of variation c:

n per arm = 16 x c^2 / d^2

- Detect 2 percent: n = 16 x 4.84 / 0.0004 = 193,600 users per arm
- Detect 1 percent: n = 16 x 4.84 / 0.0001 = 774,400 users per arm

With 5 M daily users, a 50-50 split reaches 774,400 users per arm in under a
day of exposure, but users must accumulate their full in-period metric, so the
binding constraint is the 14-day window, not the user count. That is the usual
situation for revenue-style metrics and it is worth saying plainly: for
per-user metrics, duration is driven by the metric's accumulation window, and
extra traffic beyond a point buys nothing.

Applying CUPED at correlation 0.65 removes 1 - 0.65^2 = 58 percent of the
variance, so the same question needs 42 percent of the sample. I use it to add
guardrail precision rather than to shorten the test, because the 14 days are set
by the metric window.

**The error most readers make here.** Computing power on the metric they can
compute and then reporting on a different metric. Power belongs to the primary
metric only.

#### Block C. Purpose: build the invalidity checks into the launch

Before I look at any result, three checks run:

1. Sample ratio: chi-square on assignment counts, with the readout blocked if p
   is below 0.001.
2. A/A period: the first day runs both arms with the same model, and any metric
   showing a significant difference is flagged as having an instrumentation
   problem.
3. Guardrail telemetry: exposure logging fires at assignment time, not after the
   model returns, because logging after a treatment-only code path is one of the
   commonest causes of ratio mismatch.

!!! tip "Self-explanation prompt"

    Say why exposure logging placed after the model call creates a ratio
    mismatch, and which arm ends up under-counted.

**The error most readers make here.** Running the ratio check after the analysis
and treating it as a footnote. It is a gate, not a caveat.

#### Block D. Purpose: read the disagreement without guessing

Result: click rate plus 3.1 percent with a tight interval, revenue per user
minus 0.2 percent with an interval spanning zero. I work the table:

| Check | Finding | Consequence |
|---|---|---|
| Sample ratio | Passes | Continue |
| Power on revenue | Minimum detectable effect was 1.0 percent | The test cannot distinguish flat from minus 0.9 percent, so "flat" is not a finding |
| Effect by day | Click effect declines from 5.2 to 2.4 over the fortnight | Partly novelty. Plan with the asymptote, not the mean |
| Click composition | Clicks shifted toward low-price items | The model found cheap clicks, which is a metric cannibalisation, not a win |
| Latency | Plus 4 ms p95 | Within guardrail |

The finding is now precise: the model increased engagement with cheaper
inventory and moved no revenue, and the test could not have detected a 0.9
percent revenue loss. That is a very different sentence from "click rate is up,
ship it".

**The error most readers make here.** Reading a non-significant guardrail as
"no harm". A non-significant result on an underpowered metric means you did not
look hard enough.

#### Block E. Purpose: decide, and say what would change the decision

Decision: do not ship as is. Two paths, priced:

| Path | Cost | What it tests |
|---|---|---|
| Add a value term to the training objective and retest | One model iteration plus another slot | Whether the click gain can be steered onto revenue-bearing items |
| Ship behind a price-aware reranker and retest | Two weeks of engineering | Whether the fix belongs in the ranker or the layer above it |

I also name the number that would flip me: if a re-run with 4 weeks and CUPED
shows revenue per user non-inferior within a 0.3 percent margin while clicks
hold plus 2 percent, I ship, because the engagement gain then comes without a
revenue transfer.

**The error most readers make here.** Shipping anyway because the primary metric
was not significantly negative. Non-inferiority is a claim that needs a stated
margin and the power to support it, and "not significantly worse" on an
underpowered metric is not that claim.

### 11e. Fade the scaffold

**Fade 1: a dispatch change in a delivery marketplace.** The change assigns
couriers in batches instead of one at a time. Last block blank.

- Block A, unit and primary metric: couriers are a shared bounded resource, so
  user-level randomisation would let treatment orders take couriers from control
  orders. Use a switchback on region and time. Primary metric is orders
  delivered per courier hour. Guardrails are late-delivery rate and courier
  earnings per hour.
- Block B, sizing: 6 regions, 21 days, 3-hour periods gives 6 x 21 x 8 = 1,008
  periods, 504 per arm. At a period-level coefficient of variation of 10
  percent, the minimum detectable effect is 2.8 x 0.10 x sqrt(2 / 504) = 1.8
  percent.
- Block C, invalidity checks: verify that period assignment is balanced across
  hour of day, that the burn-in exclusion is applied symmetrically, and that
  region boundaries do not leak couriers.
- Block D, reading the result: **you write this block.**

??? note "Show answer"

    Result to read: deliveries per courier hour plus 2.4 percent, late-delivery
    rate plus 0.8 percentage points, courier earnings flat.

    First, check the burn-in. Batching creates a queue, and the first minutes of
    a treatment period inherit an unbatched queue, so an asymmetric burn-in would
    manufacture part of the gain. Second, check whether the efficiency gain and
    the lateness cost are the same trade or two effects: plot lateness against
    batch size within the treatment arm. Third, convert both into one currency,
    because a 2.4 percent efficiency gain and a 0.8 point lateness rise are only
    comparable through a stated cost of a late delivery.

    The decision most likely available: ship with a batching cap that bounds
    wait time, then re-run. Name the number that decides it, for example the
    batch size at which lateness rises faster than efficiency, which is visible
    inside the treatment arm without another experiment.

**Fade 2: a two-model comparison for search relevance with no revenue effect
expected.** Last two blocks blank.

- Block A, unit and primary metric: no interference between users, and the
  question is purely which ranker users prefer. Interleaving on the query is
  available and needs far less traffic than a user-split test.
- Block B, sizing: **you write this block.**
- Block C, invalidity checks: **you write this block.**

??? note "Show answer"

    Block B. Interleaving is a paired comparison per query, so size it on the
    per-query preference rate rather than a per-user metric. If the expected
    preference split is 52 to 48, the effect is 0.04 on a proportion with
    variance about 0.25, so n = 16 x 0.25 / 0.0016 = 2,500 queries with a
    recorded preference.

    Only queries where the two rankers differ and a click occurs produce a
    preference, so if that is 8 percent of queries you need about 31,250
    queries. State both numbers, because the second is the one that schedules
    the test.

    Block C. Fairness of the merge is the invalidity risk unique to interleaving.
    Check that each ranker contributes the top slot equally often, that
    deduplication does not systematically drop one ranker's items, and that
    ties are broken randomly rather than by a stable identifier. Then run an A/A
    interleaving of the ranker against itself: the preference should sit at 50
    percent within noise, and any deviation is a bug in the merge, not a finding
    about relevance.

### 11f. Check yourself

**Q1.** A team reports "no significant difference" on a guardrail after a
one-week test and ships. What do you ask for, and how do you phrase the risk in
one sentence a product manager will act on?

??? note "Show answer"

    Ask for the minimum detectable effect the test actually had on that
    guardrail, computed from its own variance and sample. Then phrase it as
    exposure rather than statistics: "this test could not have detected a
    retention loss smaller than 1.2 percentage points, and a 1 percentage point
    retention loss across our base is worth more than the entire gain we are
    shipping for". That sentence gets a longer test funded. "It was
    underpowered" does not.

**Q2.** Your experiment shows a 3 percent lift that decays from 5.2 percent on
day one to 2.4 percent on day fourteen. Two colleagues disagree: one says
novelty, the other says the treatment is degrading. How do you tell them apart
with data you already have?

??? note "Show answer"

    Split the effect by user tenure inside the experiment rather than by calendar
    day. Novelty is a property of exposure age: users on their first day of
    exposure show a large effect whichever calendar day that is, so plotting
    effect against days-since-first-exposure gives a decaying curve while the
    calendar-day plot for a fixed exposure age stays flat.

    Degradation is a property of calendar time: it hits new and tenured users
    together on the same day, usually because a cache filled, an index went
    stale, or a fallback started firing. So the discriminating plot is effect by
    exposure age versus effect by calendar date, and the second diagnostic is the
    fallback rate and index freshness over the same fortnight. If neither pattern
    is clean, extend the test rather than arguing.

### 11g. When not to use this

**A full A/B test with a declared primary metric.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The decision is durable, the effect must be stated in business units, and your traffic gives a minimum detectable effect smaller than the effect you care about |
| Scale threshold below which it is over-engineering | Below roughly 50,000 users per arm you cannot detect anything under about 5 percent relative on a 2 percent base rate. Under that, use a staged rollout with monitoring and be honest that you are not measuring, you are watching |
| Cheaper alternative | Staged rollout with automatic rollback on fast signals, plus a qualitative review of a sample of outputs |

**Interleaving.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The question is strictly ranker versus ranker, the two produce comparable candidate sets, and a user-split test would need more traffic than you have |
| Scale threshold | If your split test already reaches power in a few days, the added machinery and its fairness bugs cost more than they save |
| Cheaper alternative | A shorter split test on a highly sensitive engagement metric, accepting that you learn less about long-term effects either way |

**Switchback experiments.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You can demonstrate interference, for example by showing that the measured effect changes when allocation moves from 5 percent to 50 percent |
| Scale threshold | Fewer than about 200 periods per arm leaves you unable to detect effects under a few percent, no matter how many trips you have. Below that, run a geographic split and accept the confounding |
| Cheaper alternative | A city-level or region-level split test with matched pairs, which is easier to run and easier to explain, at the cost of far fewer independent units |

**Bandits in place of an experiment.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Arm count above roughly 20, reward observed within minutes, and no requirement for an unbiased effect estimate |
| Scale threshold | With 2 to 5 arms and a durable decision, the regret saved is a rounding error against the value of a clean effect size |
| Cheaper alternative | A fixed split test with an early-stopping rule agreed in advance, which keeps the statistics interpretable and still cuts exposure to a clear loser |
---

## Module 12. Serving and deployment

**Time: 40 to 50 minutes reading, 30 to 40 minutes practice.**

This is the module where an ML design becomes a system. Everything before it
decides what to compute. This decides where it runs, what it costs, how fast it
answers, and how you take it back out when it is wrong.

### 12a. Predict first

```
+---------------------------------------------------------------+
| SLO: end to end p99 of 250 ms. Measured p99 by stage:         |
|   feature fetch, parallel to 8 stores          90 ms          |
|   retrieval, parallel to 12 index shards       60 ms          |
|   ranking, one model over 600 candidates       45 ms          |
|   reranking and policy filter                  20 ms          |
|   gateway, serialisation, network              25 ms          |
|   sum of the stage p99 values                 240 ms          |
| Measured end to end p99                       310 ms          |
+---------------------------------------------------------------+
Caption: per-stage and end-to-end tail latency for a ranking
service, with the stage values summing to less than the whole.
```

**Question.** The stages sum to 240 ms and the whole is 310 ms. Explain the gap,
and say which stage you attack first.

??? note "Show answer"

    If stage latencies were independent, the p99 of the sum would be
    substantially below the sum of the p99 values, because it is unlikely that
    several stages are simultaneously at their own worst percentile. Measuring
    310 ms against a 240 ms sum is therefore evidence of positive correlation
    between stages, not evidence that one stage is slow.

    Correlation of that kind comes from something shared: a saturated host, a
    garbage collection pause that hits every stage on the same process, a shared
    thread pool or connection pool with a queue, a noisy neighbour on the same
    machine, or one slow request being slow because the user has more history and
    therefore more work at every stage.

    The stage to attack is none of them. The first move is to trace individual
    requests and compute the correlation between stage latencies within a
    request. If the slow requests are the same requests at every stage, fix the
    shared resource. If they are the same users at every stage, fix the work
    that scales with user history. Tuning the 90 ms feature fetch in isolation
    will move the sum and not the whole.

**Three serving modes, and the number that chooses between them.**

| Mode | Compute triggered by | Freshness | Cost scales with |
|---|---|---|---|
| Batch precompute | A schedule | As old as the schedule | Number of entities |
| Online | A request | Fully fresh | Number of requests |
| Near-line or streaming | An event | Seconds to minutes | Number of relevant events |

Compute all three volumes before choosing. Take 100 M users, 8 sessions per user
per day, one ranked request per session:

- Batch, top 500 per user, once a day: 100 M inferences per day, 1,157 per
  second averaged, no tail requirement
- Online: 800 M inferences per day, 9,259 per second average, roughly 27,800 per
  second at a 3x peak, with a hard tail requirement
- Near-line, recomputing only when a user takes a meaningful action, assume 5
  percent of events: 40 M per day

Online costs 8 times the inference volume of daily precompute and adds a tail
SLO. It buys freshness. If the answer would be identical for 24 hours, you are
paying 8 times for nothing.

**The hybrid is usually the right answer and is the one candidates skip.**
Precompute the candidate list nightly, then rerank the top 200 online with
features that changed since. Cost is the batch cost plus a 200-candidate online
pass, and freshness is request-time for anything that matters. When an
interviewer asks for fresher results without doubling cost, this is the shape of
the answer.

**Latency budget, built from the SLO downward.** Write the budget before the
architecture, then let the budget reject boxes.

| Stage | Budget at 250 ms p99 | What it buys |
|---|---|---|
| Gateway, auth, serialisation | 20 ms | Fixed overhead you rarely reduce |
| Feature fetch | 60 ms | Everything the model reads |
| Retrieval | 40 ms | Candidate generation |
| Ranking | 60 ms | The model itself |
| Reranking, policy, diversity | 20 ms | The layer that makes it shippable |
| Reserve | 50 ms | Retries, hedges, and the day traffic is 20 percent above forecast |

The reserve is not padding. A budget with no reserve is a budget that is
exceeded on the first bad Tuesday.

**Fan-out multiplies the tail, and the arithmetic is exact.** If a request waits
on k independent shards and each shard exceeds x with probability q, the request
exceeds x with probability 1 minus (1 - q)^k.

To hold a request-level p99 across 12 shards you need (1 - q)^12 = 0.99, so q =
1 minus 0.99^(1/12) = 0.00084. Each shard must meet the target at its 99.92nd
percentile. That is the single most useful number in this module, because it
explains why fan-out systems feel unfixable: you are tuning a percentile two
orders of magnitude deeper than the one you report.

Three responses, in cost order:

| Response | Cost | When |
|---|---|---|
| Reduce k by partitioning on a key that lets you hit one shard | Free at design time, expensive later | The query has a natural partition, such as locale or vertical |
| Hedge: send a duplicate after the p95 and take the first answer | About 5 percent extra load | The work is idempotent and read-only |
| Make the tail shorter at the shard | The real fix, and the slow one | Always, eventually |

Dean and Barroso set out the mechanisms and the hedging technique in 2013
([The Tail at Scale](https://dl.acm.org/doi/10.1145/2408776.2408794), CACM 2013,
accessed August 2026).

**Where the cost actually is, derived.** Assume a ranker costing 2 MFLOP per
candidate, 600 candidates, so 1.2 GFLOP per request. Assume an accelerator
delivering 100 TFLOPS peak and, at online batch sizes with a latency
constraint, 10 percent of that in practice. That utilisation assumption is the
one to challenge first, and it is low because online serving cannot batch the
way training can.

- Effective throughput: 1e13 / 1.2e9 = 8,333 requests per second per device
- At a 27,800 per second peak: 4 devices, call it 8 with headroom
- At an assumed 3.00 per device-hour: 24.00 per hour, 576 per day
- Per thousand requests: 576 / 800,000 = 0.00072

Now the same request's feature traffic. Assume 600 candidates each carrying 200
float features:

- 600 x 200 x 4 bytes = 480 KB per request
- At 27,800 requests per second: 13.3 GB per second, about 107 Gbps

The model is fractions of a cent per thousand requests. The feature movement is
a hundred gigabits per second. This is the reason experienced answers put item
features and embeddings in the ranker's own memory, precompute item vectors, and
prune features aggressively, while inexperienced answers spend the whole deep
dive on model architecture.

**The cascade, priced.** Run a cheap model over all candidates, then the
expensive one over the survivors.

- One stage: 600 x 2 MFLOP = 1,200 MFLOP
- Two stages: 600 x 0.05 MFLOP + 60 x 2 MFLOP = 30 + 120 = 150 MFLOP
- Eight times cheaper

The quality question has one honest measurement: what fraction of the items that
the full model would place in the final top 10 survive the cheap stage's top 60.
Measure it offline on logged requests. If it is 97 percent, ship the cascade. If
it is 80 percent, the cheap stage is not cheap, it is a different product.

**Caching, and what is actually cacheable.**

| Object | Cacheable | Sensible time to live | Why |
|---|---|---|---|
| Item embeddings | Yes | Hours to days | They change only when the item model is retrained |
| Item metadata and static features | Yes | Minutes to hours | Changes are rare and tolerable when stale |
| User embeddings | Yes | Minutes to hours | Depends on how fast behaviour shifts |
| Query to candidate list | Yes, for repeated queries | Minutes | Freshness cost is that new items wait for the time to live |
| Final personalised score | Rarely | Seconds | The key includes the user, so hit rates collapse |
| Model weights in process | Yes | Until deploy | This is the one everyone forgets to warm |

**Query caching, sized with a distribution you can defend.** Assume query
frequency follows a Zipf distribution with exponent 1, which you should verify
by plotting log frequency against log rank on one day of logs. Then the share of
traffic covered by the top k of N distinct queries is the ratio of harmonic
numbers, approximately (ln k + 0.577) / (ln N + 0.577).

- N = 10 M distinct queries, k = 100,000
- (11.51 + 0.58) / (16.12 + 0.58) = 12.09 / 16.70 = 0.72

So caching 100,000 query results covers about 72 percent of traffic. Memory: 100
k entries x 500 candidate identifiers x 8 bytes = 400 MB, which fits in one
process. The cost is freshness: with a 10 minute time to live, a new item waits
up to 10 minutes to appear on a cached query, and you should say that number out
loud rather than letting the interviewer find it.

**Sharding: replicate for throughput, shard for size.**

- An index of 500 M vectors at 768 dimensions in 4-byte floats is 500e6 x 768 x
  4 = 1.54 TB
- At 128 GB usable per host: 12 shards
- 12 shards means the fan-out tail arithmetic above, so per-shard p99.92
- Quantising vectors to 1 byte per dimension cuts this to 384 GB and 3 shards,
  at a recall cost you must measure

Notice that the memory decision and the tail latency decision are the same
decision. Compressing the index is a latency intervention, not just a cost one.

**Model optimisation, in the order that pays.**

| Lever | Typical win | Quality cost | Measure before doing it |
|---|---|---|---|
| Prune features by gain | Often the largest, because it cuts feature fetch bytes | Small if you cut the bottom of the gain curve | Feature fetch as a share of request latency |
| Reduce candidate count | Linear in cost | Recall at the new cut point | Overlap of final top 10 versus the larger set |
| Quantise to 8-bit integers | Meaningful on memory-bound serving | Small, and measurable per model | Whether you are memory bound or compute bound |
| Distil to a smaller student | Large, and it compounds with the above | Depends entirely on the task | Whether the teacher's advantage survives distillation on your data |
| Batch requests | Throughput, not latency | None to quality, real to tail latency | Whether you have headroom in the budget for queueing |

The ordering matters more than the list. Teams reach for quantisation and
distillation because they are interesting, and then discover the model was 12
percent of the request.

```
+-------------+   +--------------+   +----------------+
| GATEWAY     |=>>| RETRIEVAL    |=>>| RANKER         |
| 20 ms       |   | 12 shards    |   | 600 cands      |
| auth, parse |   | 40 ms        |   | 60 ms          |
+------+------+   +------+-------+   +--------+-------+
       |                 |                    |
       |                 v                    v
       |          +--------------+   +----------------+
       |          | FEATURE CACHE|   | POLICY LAYER   |
       |          | item vectors |   | filters, caps  |
       |          | in process   |   | 20 ms          |
       |          +--------------+   +--------+-------+
       |                                      |
       v                                      v
+---------------+                    +----------------+
| SHADOW COPY   |                    | RESPONSE       |
| mirrored, no  |                    | 50 ms reserve  |
| user impact   |                    | unspent        |
+---------------+                    +----------------+
Caption: the serving path with the latency budget written inside
each box, plus the shadow copy that receives mirrored traffic and
returns nothing to the user.
```

### 12d. Worked example

**The prompt.** "Our recommender recomputes everything nightly. Users complain
results are stale. Make it fresher without doubling the bill, and tell me how
you would roll it out."

Inputs:

| Input | Value | Status |
|---|---|---|
| Users | 100 M, 20 M active daily | GIVEN |
| Items | 5 M, with 50 k new per day | GIVEN |
| Requests | 800 M per day, peak 27,800 per second | GIVEN |
| Current pipeline | Nightly batch, top 500 per user, 6 hours on 200 machines | GIVEN |
| Current bill | 100 percent, and the ceiling is 130 percent | GIVEN |
| SLO | p99 250 ms | GIVEN |

#### Block A. Purpose: locate the staleness before repricing anything

Staleness is not one thing. I split it into three and ask which one users
actually feel:

| Source of staleness | Age | Who notices |
|---|---|---|
| A new item is not in anyone's list | Up to 24 hours | Creators, and users following them |
| The user's own recent actions are not reflected | Up to 24 hours | Every active user, in the session |
| The model is a day old | 24 hours | Almost nobody, at this cadence |

The second row is the complaint. That is decisive, because reflecting
within-session behaviour does not require recomputing the whole catalogue, and
recomputing the whole catalogue does not fix it.

**The error most readers make here.** Answering "make the batch run hourly".
That multiplies the batch bill by up to 24 and still leaves a session-level lag
that users experience as the system ignoring them.

!!! tip "Self-explanation prompt"

    Before Block B, say which of the three staleness rows a nightly-to-hourly
    change actually fixes, and what it costs per row fixed.

#### Block B. Purpose: choose the hybrid split and price it

The design: keep the nightly batch producing 500 candidates per user, add an
online reranking pass over the top 200 using request-time features, and add a
near-line path that injects new and trending items into the candidate pool
within minutes.

Pricing the online rerank against the numbers above:

- 200 candidates x 2 MFLOP = 400 MFLOP per request
- At 1e13 effective FLOPS per device: 25,000 requests per second per device
- Peak 27,800 needs 2 devices; provision 6 for headroom and failure domains
- 6 devices x 3.00 per hour x 24 = 432 per day

Against a nightly batch of 200 machines for 6 hours, the online layer is a small
addition, and the number that proves it is the ratio of the two compute bills,
which I would compute from the actual machine prices rather than asserting.

The near-line path is cheaper still: 50,000 new items per day need embedding
once each, which is 50,000 inferences, a rounding error against 800 M.

**The error most readers make here.** Pricing only the model and forgetting the
feature movement. 200 candidates x 200 features x 4 bytes = 160 KB per request,
and at 27,800 per second that is 4.4 GB per second. That number, not the FLOPs,
decides whether item features live in the ranker's memory or behind a network
call.

#### Block C. Purpose: protect the tail while adding a stage

I have added a stage to a path that already meets 250 ms, so I must take the
time from somewhere. The budget:

| Stage | Before | After | How |
|---|---|---|---|
| Retrieval | 60 ms | 20 ms | Precomputed list is a single key lookup, not a fan-out |
| Feature fetch | 90 ms | 45 ms | Item features for 200 candidates are in process, only user features go over the network |
| Rerank | 0 | 45 ms | New stage |
| Policy and response | 45 ms | 45 ms | Unchanged |
| Reserve | 55 ms | 95 ms | Better than before |

The hybrid is faster than the pure online design would be, because the nightly
batch already eliminated the 12-way index fan-out. That is the trade to say out
loud: precompute buys tail latency as well as cost, and it pays for both with
freshness, which the near-line path then buys back for the subset of items where
freshness matters.

!!! tip "Self-explanation prompt"

    Say why moving item features in process is safe here but would not be safe if
    the reranker scored 600 candidates drawn from the full 5 M catalogue.

**The error most readers make here.** Adding the online stage on top of the
existing budget and declaring the SLO unchanged. Every new box spends time that
belonged to something else.

#### Block D. Purpose: roll it out so that being wrong is cheap

Three stages, each catching a different class of failure:

| Stage | Traffic | Catches | Time to a verdict |
|---|---|---|---|
| Shadow | 100 percent mirrored, no user impact | Crashes, latency, score distribution shift, feature nulls | Hours |
| Canary | 1 percent | Quality regressions with enough exposure to measure | See the arithmetic below |
| Staged | 5, 25, 50, 100 percent | Capacity and rare-path failures | Days |

Canary detection time, derived: at 1 percent of 800 M requests per day, the
canary sees 8 M requests per day, which is 333,000 per hour. To detect a 5
percent relative drop in a 2 percent click rate, n = 16 x 0.0196 / (0.001)^2 =
313,600 in the small arm. So roughly one hour of canary gives power to see a 5
percent quality regression, while an error-rate spike is visible in seconds.

That asymmetry sets the schedule: hold each canary step for the slowest signal
you intend to gate on, not the fastest one you can see.

**The error most readers make here.** Treating shadow traffic as a quality test.
Shadow has no labels, because nobody saw the output. It tests the machinery, not
the recommendation.

#### Block E. Purpose: write the rollback rules before launch

| Signal | Threshold | Action | Automatic |
|---|---|---|---|
| Error rate | Above 0.5 percent for 2 minutes | Roll back | Yes |
| p99 latency | Above 250 ms for 5 minutes | Roll back | Yes |
| Empty result rate | Above 0.2 percent | Roll back | Yes |
| Score distribution shift | Mean predicted score outside 3 standard deviations of the shadow baseline | Page, then decide | No |
| Click rate | Below minus 5 percent relative for 2 hours at 1 percent | Roll back | Yes |
| Revenue per user | Any breach of the pre-set non-inferiority margin | Human decision | No |

Rollback is a configuration flip to a model version that is already loaded and
warm, not a redeploy. If your rollback path requires a build, your mean time to
recovery is your build time, and you should say that number in the interview
because it is usually the honest weak point of the design.

**The error most readers make here.** Writing rollback criteria that all require
a human judgement. If every threshold ends in "then investigate", your rollback
time is the time it takes to wake someone up, and the fast signals should have
been automatic.

### 12e. Fade the scaffold

**Fade 1: a fraud scorer at checkout with a 40 ms budget.** Last block blank.

- Block A, locate the real constraint: the budget is 40 ms end to end inside a
  payment flow, so the constraint is the tail, not the cost. Precompute is
  impossible because the request itself is the entity being scored.
- Block B, choose the shape and price it: a small model in process, features
  restricted to those available within the request plus a cached account
  aggregate. Any feature requiring a network call to a second service is either
  cached with a stale-but-bounded value or dropped.
- Block C, protect the tail: a hard deadline with a defined degraded answer. If
  the account aggregate lookup exceeds 8 ms, score without it and set a flag,
  because a slow answer here is a failed checkout.
- Block D, rollout: **you write this block.**

??? note "Show answer"

    Shadow first, on 100 percent of mirrored checkouts, because it establishes
    the score distribution and the latency profile with zero risk to payment
    flow. Compare the new model's score histogram to the incumbent's, and compare
    the rate at which the degraded path fires.

    Canary differently from a recommender: do not canary by percentage of
    traffic, canary by decision. Run the new model in scoring-only mode where it
    computes and logs but the incumbent still decides. That gives full-volume
    label collection with zero user exposure, which is possible here precisely
    because the label arrives later anyway.

    Only then take a small share of decisions, and gate on review-queue volume
    rather than on model metrics, because the human capacity constraint breaks
    before the quality does. Rollback is a flag flip to the incumbent decision,
    and the new model keeps scoring in the background so the comparison
    continues.

**Fade 2: a semantic search service, 200 M documents, p99 budget 150 ms.** Last
two blocks blank.

- Block A, locate the constraint: index size. 200 M vectors at 768 dimensions
  and 4 bytes is 614 GB, which does not fit one host, so fan-out and its tail
  arithmetic are forced unless the vectors are compressed or the corpus is
  partitioned by a filter that queries always carry.
- Block B, choose the shape and price it: **you write this block.**
- Block C, protect the tail: **you write this block.**

??? note "Show answer"

    Block B. Two levers before sharding. Quantise vectors to one byte per
    dimension, taking 614 GB to 154 GB, and measure recall at 100 against the
    uncompressed index on a fixed query set; if recall loss is under a point,
    take it. Partition by any filter present in essentially every query, such as
    language or tenant, so a query touches one partition rather than all of them.
    If both apply, the index fits 2 hosts instead of 6, and replicas are then
    added for throughput rather than for size.

    Block C. With 2 shards, holding a request p99 needs each shard at (1 - q)^2
    = 0.99, so q = 0.005, or per-shard p99.5. That is achievable, where the
    p99.92 demanded by 12 shards is not.

    Add a hedge after the p95 for the read-only query path at about 5 percent
    extra load, cap the candidate count the index is allowed to explore so that
    a pathological query cannot run long, and return a partial result with a
    flag rather than exceeding the deadline. State the recall cost of the
    exploration cap, because it is a quality decision being made in a latency
    budget.

### 12f. Check yourself

**Q1.** A colleague proposes moving from a nightly batch recommender to fully
online scoring, arguing that it is simpler because there is only one code path.
Give the strongest version of their argument, then the number that decides it.

??? note "Show answer"

    The strongest version is real and worth stating: two paths means two feature
    implementations, which is the most reliable source of training-serving skew
    in this whole course, plus two sets of bugs, two on-call surfaces and a
    permanent reconciliation task. One path is genuinely cheaper to own.

    The number that decides it is the ratio of requests to entities. Here it is
    800 M requests against 100 M users, a factor of 8, so online scoring does
    roughly 8 times the model work and must do it inside a tail budget. If the
    ratio were 1.2, the argument for one path would win easily. The
    counter-argument to skew is not to abolish precompute, it is to compute
    features once in one library and use it from both paths, which is the point
    of a feature store.

**Q2.** Your ranking service fans out to 12 shards and reports a p99 of 60 ms
while each shard reports a p99 of 45 ms. Someone concludes the shards are fine.
What do you tell them?

??? note "Show answer"

    Shard p99 is not the relevant percentile. With 12 shards, a request waits for
    the slowest, so the request-level p99 requires each shard's 99.92nd
    percentile to be inside the budget: 1 minus 0.99 to the power one twelfth is
    0.00084. A shard whose p99 is 45 ms can easily have a p99.9 of 300 ms if it
    has a garbage collection pause, a cold cache path or a hot key, and that tail
    is what your users are experiencing.

    Ask for shard p99.9 and p99.99, not p99. Then reduce k if the query has a
    natural partition, hedge after the p95 for read-only work, and treat the
    shard tail as the actual engineering task. Reporting shard p99 as evidence of
    health is the most common measurement error in fan-out systems.

### 12g. When not to use this

**Online scoring rather than batch precompute.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The answer measurably changes within the precompute period: for example, session-level behaviour moves the top 10 for more than about 20 percent of requests. Measure this by scoring a sample both ways |
| Scale threshold below which it is over-engineering | If requests per entity per period is under about 2, online scoring costs more and gains nothing. Under a few hundred requests per second, a nightly batch into a key-value store is simpler, cheaper and has a better tail |
| Cheaper alternative | Batch precompute plus an online rerank of the top 200, which captures most of the freshness at a fraction of the cost |

**A cache in front of the model.**

| Question | Answer |
|---|---|
| Measurement that justifies it | A measured repeat rate: the share of traffic covered by the top k keys, computed from one day of logs, not assumed from a distributional argument |
| Scale threshold | If the key includes the user identifier, hit rates are usually under 10 percent and the cache adds a failure mode, a stampede risk and a staleness bug for very little. Below about 30 percent hit rate, do not bother |
| Cheaper alternative | Cache the components rather than the answer: item embeddings and static features have far higher reuse than final scores |

**Sharding the index.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The index does not fit the memory of one host after quantisation and after removing fields you do not query |
| Scale threshold | Under roughly 50 M vectors at moderate dimension, one host with a replica for availability beats a sharded design, and it removes the p99.9 tail requirement entirely |
| Cheaper alternative | Quantise, prune the corpus, or partition by a filter every query already carries so that a request touches one partition |

**Shadow deployment.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The change alters the serving path in a way a unit test cannot cover, and you can afford to double the compute for the shadowed traffic |
| Scale threshold | For a pure weight update to an existing model with an unchanged feature set, shadowing mostly re-measures what a canary would tell you an hour later. Skip it and canary |
| Cheaper alternative | Replay a fixed sample of logged requests through the new model offline and compare score distributions. It catches most of what shadow catches, at no serving cost |
---

## Module 13. Training infrastructure and cost

**Time: 35 to 45 minutes reading, 25 to 35 minutes practice.**

Training infrastructure is missing from most interview preparation, which is
strange, because retrain cadence constrains architecture more tightly than model
choice does. If a full run takes 22 hours, a daily cadence is a promise you
cannot keep, and every design decision downstream inherits that.

### 13a. Predict first

```
+---------------------------------------------------------------+
| Ranking model retrain, as it runs today                       |
|   training examples per day          10,000,000,000           |
|   example size on disk               2 KB                     |
|   dense parameters                   100 M                    |
|   embedding table                    1e9 rows x 64 dims       |
|   feature backfill before training   5 hours                  |
|   full training run, 16 devices      14 hours wall clock      |
|   validation and canary gate         3 hours                  |
|   target cadence                     daily                    |
+---------------------------------------------------------------+
Caption: the wall-clock stages of one retrain, with the model
sizes that determine how it must be distributed.
```

**Question.** Does the daily cadence hold? Compute two numbers before answering,
and name the first change you would make.

??? note "Show answer"

    Number one: 5 + 14 + 3 = 22 hours of a 24 hour day. There is no room for a
    single failure. A useful rule is that the pipeline must fit in half the
    cadence, because you need room for exactly one failed run and its retry, so
    the daily target requires 12 hours and you are at 22.

    Number two: the embedding table is 1e9 x 64 x 4 bytes = 256 GB. It does not
    fit one accelerator, so it must be sharded across hosts or held on host
    memory with a lookup path. That is not optional, and it is the fact that
    decides the distribution strategy before anyone argues about it.

    The first change is not more hardware. It is to stop retraining from scratch:
    warm start from yesterday's checkpoint and train on the new day plus a
    sample of history. That attacks the 14 hours directly. The second change is
    to overlap backfill with training by streaming examples as they are
    materialised, which attacks the 5 hours. Adding devices attacks 14 hours at
    sublinear returns and does nothing to the other 8.

**Distribution modes, and what forces each one.**

| Mode | What it splits | Forced when | Main cost |
|---|---|---|---|
| Single device | Nothing | Default. Stay here as long as you can | None |
| Data parallel | The batch | One epoch is too slow, model fits in memory | Gradient communication every step |
| Sharded embeddings, replicated dense | The embedding table | Embedding table exceeds device memory | A lookup and a scatter every step |
| Model or tensor parallel | The layers themselves | A single layer's activations exceed device memory | Communication inside the forward pass, which is far more sensitive |
| Pipeline parallel | Layer groups across devices | Very deep models | Bubbles, and a scheduling problem |

For classic ranking and recommendation systems, the honest answer is that you
almost always need the third row and almost never need the last two. Say that,
and then say what would change your mind.

**Data-parallel communication, derived.** Ring all-reduce moves about 2 x P
bytes of gradient per worker per step, where P is the parameter payload in
bytes, and this is nearly independent of worker count.

- 100 M dense parameters in 2-byte precision: P = 200 MB
- Traffic per worker per step: about 400 MB
- Interconnect at 100 Gbps, which is 12.5 GB per second: 32 ms per step

If a step's compute is 100 ms:

- Scaling efficiency = 100 / (100 + 32) = 0.76
- 16 workers give an effective speedup of 16 x 0.76 = 12.1

That number belongs in your answer. It converts "we will distribute it" into
"16 devices buy a 12x speedup at 16x the hourly cost, so we pay 32 percent more
money for the wall-clock win". That is a decision, not a slogan.

**Larger batches move the ratio, and the trade is explicit.** Doubling the batch
roughly doubles compute per step while leaving communication per step unchanged,
so efficiency rises. The cost is that very large batches often need a learning
rate schedule change and can reduce final quality, so treat batch size as a
tuned parameter with a quality measurement attached, not as a free lever.

**Sparse embedding traffic, derived separately.** Embedding gradients are
sparse, so they do not go through the dense all-reduce.

- Batch 8,192, 50 active identifiers per example, 64 dimensions, 4 bytes
- 8,192 x 50 x 64 x 4 = 105 MB gathered per step
- At 12.5 GB per second: 8.4 ms

So embedding lookup traffic is a quarter of the dense gradient traffic here.
Compute both numbers before deciding which one to optimise, because teams
routinely optimise the one they find more interesting.

**Checkpointing, with the optimal interval derived rather than guessed.** Let C
be the cost of writing one checkpoint and M the mean time between failures for
the job. Daly's approximation gives an optimal interval of

T = sqrt(2 x C x M)

and a total overhead fraction of about sqrt(2 x C / M). Daly published the
derivation in 2006
([A higher order estimate of the optimum checkpoint
interval](https://doi.org/10.1016/j.future.2004.11.016), Future Generation
Computer Systems, accessed August 2026).

Worked, on interruptible capacity: 64 instances, each with an assumed 5 percent
chance of preemption per hour, so the job's failure rate is 64 x 0.05 = 3.2 per
hour and M = 1,125 seconds.

| Checkpoint write cost C | Optimal interval T | Overhead |
|---|---|---|
| 60 s | sqrt(2 x 60 x 1125) = 367 s | sqrt(120 / 1125) = 32.7 percent |
| 300 s | sqrt(2 x 300 x 1125) = 822 s | sqrt(600 / 1125) = 73.0 percent |

Now price the interruptible capacity. Assume a 65 percent discount, so you pay
0.35 of the on-demand rate:

- At C = 60 s: 0.35 x 1.327 = 0.464, so 54 percent cheaper. Take it
- At C = 300 s: 0.35 x 1.730 = 0.606, so 39 percent cheaper, with far more
  operational noise. Marginal

The conclusion is not "use interruptible capacity". It is that checkpoint write
speed decides whether interruptible capacity is worth it, so asynchronous or
incremental checkpointing is the enabling investment, not a nicety.

**Checkpoint size, sized.** Adam-style optimisers carry two moments plus the
parameters. At 100 M parameters in 4-byte precision that is 100 M x 4 x 3 = 1.2
GB for the dense part, before the 256 GB embedding table. That contrast is the
whole story: for recommendation models the checkpoint is dominated by
embeddings, so incremental checkpointing of only the touched rows is what turns
C from minutes into seconds.

**GPU economics, derived end to end.** Take these inputs and recompute them with
your own numbers, because hardware and prices change faster than this page.

| Input | Value | Status |
|---|---|---|
| Examples per epoch | 10 B | GIVEN |
| Forward cost per example | 50 MFLOP | ASSUMED. Derive yours from layer sizes rather than guessing |
| Backward multiplier | 2x forward, so 3x total | Standard for backpropagation |
| Device peak | 300 TFLOPS | ASSUMED, a current-generation accelerator |
| Model FLOPs utilisation in training | 40 percent | ASSUMED. Measure yours; embedding-heavy models run lower |
| Device price | 3.00 per hour | ASSUMED |

- Total: 10e9 x 50e6 x 3 = 1.5e18 FLOP per epoch
- Effective throughput: 300e12 x 0.40 = 1.2e14 FLOPS
- Single device: 1.5e18 / 1.2e14 = 12,500 s = 3.47 hours, costing 10.40
- On 16 devices at 0.76 efficiency: 3.47 / (16 x 0.76) = 0.285 hours wall clock,
  costing 16 x 0.285 x 3.00 = 13.70

**Now compute the other two bills, because one of them usually wins.**

| Bucket | Derivation | Result |
|---|---|---|
| Training | Above | 13.70 per epoch |
| Data movement | 10 B examples x 2 KB = 20 TB read per epoch. At 200 MB per second per reader, 100 readers finish in 1,000 s. At 0.50 per reader-hour: 100 x 0.28 x 0.50 | 14.00 per epoch |
| Online inference | From the serving module: 8 devices at 3.00 for 24 hours | 576.00 per day |

Inference costs 40 times what a training epoch costs here, and the data path
costs slightly more than the accelerators. Both facts are common and both are
routinely missed, because candidates compute the one number they were taught to
compute. Compute all three and say which one you would attack.

**Retrain cadence constrains architecture. This table is the module in one
place.**

| Cadence | Property the pipeline must have | What it rules out |
|---|---|---|
| Monthly | Nothing special. Manual steps are survivable | Nothing |
| Weekly | Automated validation, reproducible data snapshot | Hand-run notebooks |
| Daily | Full pipeline under half the cadence, warm start, automatic gate and rollback | Full retrain from scratch on a growing corpus |
| Hourly | Incremental training, no global shuffle, streaming feature materialisation | Any feature that only exists in a nightly batch |
| Continuous | Streaming trainer, per-window validation, instant model rollback, online monitoring as the gate | Batch-only feature pipelines, and any label with a long window |

Read the last column as a constraint on the rest of your design. A team that
promises hourly retraining has promised that no feature will ever require a
nightly join, and that promise is usually broken within two quarters.

**Cost levers, ordered by what they typically return.**

| Lever | Typical target | The measurement that tells you it applies |
|---|---|---|
| Stop retraining from scratch | Training wall clock | The quality delta between warm start and cold start, measured once |
| Sample the negatives | Data volume and training time | Calibration is recoverable in closed form, so the only cost is variance |
| Shrink the feature set | Data movement and serving bytes | Feature gain curve, plus feature fetch as a share of latency |
| Cache and reuse materialised features across models | Data pipeline | How many models read the same features |
| Interruptible capacity | Hourly rate | Checkpoint write cost, per the table above |
| More devices | Wall clock only, never total cost | Scaling efficiency at your interconnect |

```
+---------------------------------------------------------------+
| DAY BUDGET 24 h.  Rule: pipeline must fit in half the cadence |
+---------------------------------------------------------------+
   +----------+   +----------+   +----------+   +-------------+
   | BACKFILL |=>>| TRAIN    |=>>| VALIDATE |=>>| PROMOTE     |
   | 5 h      |   | 14 h     |   | 3 h      |   | canary 1 h  |
   +----------+   +----------+   +----------+   +-------------+
        |              |              |
        | stream       | warm start   | gate on a fixed
        | instead      | from         | golden set, then
        | of stage     | yesterday    | on canary metrics
        v              v              v
   +----------+   +----------+   +----------+
   | 0 h      |   | 4 h      |   | 1 h      |
   +----------+   +----------+   +----------+
   Total after the three changes: 6 h, inside the 12 h rule
Caption: the retrain pipeline as a wall-clock budget, with the
three changes that bring a 22 hour pipeline inside a daily cadence.
```

### 13d. Worked example

**The prompt.** "Design the training infrastructure for the ranking model in the
previous module. Daily retrain, and finance has asked why the bill grew 40
percent last quarter."

Inputs are those from the pre-question artefact, plus a stated budget ceiling of
the current spend plus 10 percent.

#### Block A. Purpose: find out what the money is actually buying

Before optimising anything I split the bill into the three buckets and get an
order of magnitude for each. Using the derivations above: training around 14 per
epoch, data movement around 14 per epoch, inference around 576 per day.

At one epoch per day, training plus data is under 30 per day and inference is
576. So the 40 percent growth is almost certainly not in training. I would check
inference volume growth and feature count growth before touching the trainer.

I say this out loud, because the prompt invites me to optimise training and the
correct first move is to refuse until the bill is decomposed.

**The error most readers make here.** Optimising the thing the question pointed
at. The question pointed at training infrastructure, and the money is in
serving.

!!! tip "Self-explanation prompt"

    Before Block B, say what evidence would reverse this conclusion, and name the
    kind of model for which training genuinely dominates the bill.

#### Block B. Purpose: make the cadence achievable at all

Cadence is a correctness problem before it is a cost problem, because a pipeline
that takes 22 of 24 hours fails permanently the first time anything goes wrong.

Three changes, each with its number:

| Change | Before | After | Mechanism |
|---|---|---|---|
| Warm start from yesterday's checkpoint, train on the new day plus a 10 percent sample of the trailing 30 days | 14 h | 4 h | Fewer examples per run, because the model already knows the history |
| Stream feature materialisation into the trainer rather than staging it first | 5 h | overlapped, so 0 h added | The trainer reads as the backfill writes |
| Gate on a fixed golden set first, canary second | 3 h | 1 h | The expensive part of validation moves after promotion, behind a canary |

Total: 6 hours, comfortably inside the 12 hour rule. Now a failed run has room
to retry inside the same day, which is the property that makes the cadence real.

The risk I take on with warm start is drift accumulation: the model can carry
forward a pathology that a cold start would have cleared. My mitigation is a
scheduled cold retrain, weekly, compared against the warm chain on the same
golden set. If the warm chain drifts, the comparison catches it in a week rather
than a quarter.

**The error most readers make here.** Warm starting forever with no cold-start
comparison. The failure is silent, slow, and shows up as an unexplained decline
that nobody can attribute.

#### Block C. Purpose: choose the distribution strategy from the sizes

The embedding table is 256 GB, which decides the shape: shard embeddings across
hosts, replicate the dense part, all-reduce the dense gradients, and gather the
sparse rows.

I check both communication numbers before sizing the cluster:

- Dense all-reduce: 400 MB per worker per step at 12.5 GB/s is 32 ms
- Sparse gather: 105 MB per step is 8.4 ms
- Compute per step: 100 ms
- Efficiency: 100 / (100 + 32 + 8.4) = 0.71

At 16 workers the effective speedup is 11.4. Going to 32 workers, the
communication terms stay roughly constant per worker while compute per worker
halves, so efficiency falls to 50 / (50 + 40.4) = 0.55 and effective speedup is
17.7 for twice the money. The marginal 16 devices buy 6.3x of speedup at 16x of
cost.

That table is the argument against the reflexive answer of adding hardware, and
it took four multiplications.

!!! tip "Self-explanation prompt"

    Say why doubling the batch size changes the efficiency figures above, and
    what quality measurement must accompany that change.

**The error most readers make here.** Quoting a scaling efficiency without
naming the interconnect. The same model on a slower fabric has a completely
different answer, and the interviewer usually knows that.

#### Block D. Purpose: survive interruption and price the capacity choice

With a 4 hour run on 16 devices, and an assumed 5 percent preemption per device
per hour, failures arrive at 16 x 0.05 = 0.8 per hour, so M = 4,500 s.

- With incremental embedding checkpointing at C = 45 s: T = sqrt(2 x 45 x 4500)
  = 636 s, and overhead is sqrt(90 / 4500) = 14.1 percent
- Effective cost on interruptible capacity: 0.35 x 1.141 = 0.40, so 60 percent
  cheaper

I take the interruptible capacity, and I state the dependency plainly: this only
works because the checkpoint writes in 45 seconds, which is only true because we
checkpoint the touched embedding rows rather than the full 256 GB. If someone
later replaces incremental checkpointing with a full write, C goes to several
minutes and the economics invert.

**The error most readers make here.** Treating interruptible capacity as a
procurement decision. It is an engineering decision whose viability is set by
checkpoint write time.

#### Block E. Purpose: state what the cadence forbids

Daily retraining is now real, and I close by naming what it forbids, because
that is the constraint the rest of the design must respect:

- No feature may require a join that is only available in a weekly batch
- No validation step may require a human in the loop on the critical path
- Any label with a window longer than a day must be handled by the delayed
  feedback machinery from Module 9, not by waiting
- The rollback path must be a version pin in a registry, warm and loaded, since
  a bad daily model reaches production within hours

**The error most readers make here.** Treating the cadence as a scheduling
detail rather than a contract. Every one of those four lines is a promise the
rest of the system must keep, and the team that adds a weekly feature join in
month three has broken the daily cadence without anyone filing a bug.

### 13e. Fade the scaffold

**Fade 1: a fraud model that must retrain hourly because attackers adapt within
a day.** Last block blank.

- Block A, find the money: hourly retraining multiplies data pipeline reads by
  24 unless the trainer is incremental, so the data path, not the accelerators,
  is where the bill moves. Compute the read volume per hour before anything
  else.
- Block B, make the cadence achievable: incremental training on the last hour
  plus a reservoir sample of history, with the full cold retrain nightly as the
  reference.
- Block C, distribution: the model is small and the feature space is modest, so
  a single device with a large batch is likely sufficient. Check it before
  distributing anything.
- Block D, interruption: an hourly job that takes 12 minutes has an expected
  preemption cost far below its checkpoint cost, so run it on on-demand capacity
  and skip checkpointing entirely inside a run.
- Block E, what the cadence forbids: **you write this block.**

??? note "Show answer"

    Hourly cadence forbids any label that takes longer than an hour to observe,
    which for fraud is nearly all of them: chargebacks arrive in weeks. So the
    hourly model cannot be trained on the outcome label at all. It must be
    trained on a fast proxy, for example rule hits, manual review decisions, or
    an early signal such as a failed authentication, with the slow chargeback
    label reserved for the nightly reference model.

    It also forbids human validation on the critical path, so the gate must be
    automatic: a fixed golden set of known fraud patterns, a score distribution
    comparison against the previous hour, and an automatic hold if the flagged
    rate moves outside a band. And it forbids any feature that depends on a
    nightly aggregate, which usually means rewriting the account-level features
    as streaming aggregations with a stated staleness bound.

**Fade 2: a demand forecasting model, retrained weekly, that has grown from 4
hours to 30 hours on the same hardware over a year.** Last two blocks blank.

- Block A, find the money: growth in wall clock with fixed hardware means either
  data volume growth, feature count growth, or a change in model size. Get the
  three time series before forming a hypothesis.
- Block B, make the cadence achievable: **you write this block.**
- Block C, distribution: **you write this block.**

??? note "Show answer"

    Block B. Weekly cadence gives a 168 hour budget, so 30 hours is not an
    emergency, and the honest first answer is that nothing needs to change yet.
    The rule of fitting in half the cadence is satisfied with a wide margin.

    What does need to change is the trend: at the observed growth rate the run
    will cross 84 hours within a year, so put a wall-clock budget alert in
    continuous integration that fails the build when a run exceeds a stated
    threshold, and make feature additions carry their measured training cost.
    Restraint is the correct answer here and saying so is a scored behaviour.

    Block C. Do not distribute. A 30 hour single-device run inside a 168 hour
    budget does not justify the communication overhead, the scaling
    inefficiency, the debugging surface or the new failure modes. If the trend
    forces action later, the ordered options are: sample the training window,
    since old demand data has declining value; prune features by measured gain;
    and only then distribute. Naming the order, and naming the threshold at which
    you would move down it, is the whole answer.

### 13f. Check yourself

**Q1.** A team says distributed training gave them a 16x speedup on 16 devices.
What single question exposes whether that number is real, and what is the most
likely true figure?

??? note "Show answer"

    Ask what the per-step communication time is relative to per-step compute
    time. Linear scaling requires communication to be effectively zero, which
    happens only when the model is tiny relative to the batch, or when the
    interconnect is far faster than the compute.

    For a 100 M parameter model at 2 bytes per parameter, ring all-reduce moves
    roughly 400 MB per worker per step, which is about 32 ms at 100 Gbps. Against
    a 100 ms step that is 76 percent efficiency, so the honest figure is around
    12x, not 16x. If the team measured 16x, they most likely changed the batch
    size at the same time, which changes the work per step and makes the
    comparison invalid.

**Q2.** Your training job runs on interruptible capacity at a 65 percent
discount, and someone proposes replacing incremental checkpointing with a simple
full checkpoint because the code is easier to maintain. Price the proposal.

??? note "Show answer"

    The relevant quantity is the checkpoint write cost C, which sets both the
    optimal interval sqrt(2 C M) and the overhead sqrt(2 C / M). With M = 4,500 s
    and C rising from 45 s to, say, 300 s, overhead rises from sqrt(90 / 4500) =
    14.1 percent to sqrt(600 / 4500) = 36.5 percent.

    Effective cost goes from 0.35 x 1.141 = 0.40 to 0.35 x 1.365 = 0.48, so the
    saving against on-demand falls from 60 percent to 52 percent. Whether that is
    worth simpler code depends on the absolute bill, and the point of the
    calculation is that the trade is now a number rather than an argument. If the
    training bill is 14 per epoch, take the simpler code without discussion. If
    it is 14,000, do not.

### 13g. When not to use this

**Distributed data-parallel training.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Single-device epoch time exceeds half your retrain cadence, and measured scaling efficiency at your interconnect is above about 0.7 |
| Scale threshold below which it is over-engineering | If one epoch fits comfortably in the cadence, distribution buys wall clock you do not need at a real increase in total cost and a large increase in failure modes |
| Cheaper alternative | Sample the training data, warm start, or use a larger batch on one device. All three are reversible in an afternoon |

**Sharded embedding tables and a parameter server style topology.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The embedding table exceeds device memory after you have removed identifiers below a frequency threshold and considered hashing collisions |
| Scale threshold | Under roughly 50 M rows at modest dimension, the table fits on a single host and sharding adds a network hop to every step for nothing |
| Cheaper alternative | Prune rare identifiers, use the hashing trick with a measured collision cost, or reduce the embedding dimension, which is usually the cheapest quality-for-memory trade available |

**Interruptible or spot capacity.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Checkpoint write cost, measured, small enough that sqrt(2 C / M) overhead leaves you meaningfully cheaper after the discount |
| Scale threshold | For jobs under about an hour, the expected number of preemptions is small but so is the saving in absolute money. Do not add complexity to save a few units of currency |
| Cheaper alternative | Reserved or committed capacity for the baseline and on-demand for peaks, which has no engineering cost at all |

**Hourly or continuous retraining.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Measured decay: the model's online metric declines by an amount you care about within the cadence you propose. Prove it by holding a model fixed for a week and plotting its metric |
| Scale threshold | If a model held fixed for a week loses under about 1 percent of its metric, hourly retraining is a large infrastructure commitment buying nothing. Most ranking models are in this regime and most fraud models are not |
| Cheaper alternative | Daily retraining plus fast-moving features. Freshness in features often captures most of the benefit that people attribute to freshness in weights |
---

## Module 14. Monitoring and drift

**Time: 30 to 40 minutes reading, 20 to 30 minutes practice.**

A model that is not monitored is a model whose quality is unknown, and unknown
quality decays. The interview version of this topic is usually three drift
definitions recited without a detection plan. The useful version is a ladder of
signals ordered by how fast they arrive, and an alert budget computed from
arithmetic rather than taste.

### 14a. Predict first

```
+---------------------------------------------------------------+
| Dashboard, yesterday against the trailing 28 days             |
|   feature 17 two-sample KS p-value          0.0001            |
|   feature 17 population stability index     0.02              |
|   mean predicted score            0.0208 => 0.0209            |
|   null rate, feature 42              0.3% => 4.1%             |
|   realised click rate               2.01% => 2.00%            |
|   labels arriving within 24 hours     98% => 71%              |
| Volume: 40,000,000 scored requests per day                    |
+---------------------------------------------------------------+
Caption: six monitored signals for one model on one day, with the
daily volume that determines what a p-value means here.
```

**Question.** Rank these by how urgently you would act, and name the one that is
not evidence of anything at all.

??? note "Show answer"

    First: labels arriving within 24 hours fell from 98 to 71 percent. This is
    the one that blinds you. Every quality metric you compute tomorrow will be
    computed on a biased subset of labels, and the model will appear to degrade
    even if it is fine. Fix the measurement system before you interpret anything
    it produces.

    Second: the null rate on feature 42 went from 0.3 to 4.1 percent, a
    fourteen-fold jump. That is a pipeline break, and the model is now scoring
    4 percent of traffic with a missing input. The size of the harm depends on
    that feature's importance and on how nulls are imputed, both of which you
    should already know.

    Not evidence of anything: the KS p-value. At 40 M samples per side, the
    critical KS statistic at a 5 percent level is about 1.36 x sqrt(2 / 4e7) =
    0.0003, so a maximum cumulative difference of three hundredths of a percent
    is "significant". The population stability index of 0.02 is the same fact
    stated as an effect size, and it says the shift is negligible. At scale, use
    effect sizes and drop the p-values.

    The flat predicted score and flat realised click rate are the good news, and
    they are also the reason the drift alert should never have fired.

**Three shifts, defined before use, and which are detectable without labels.**

| Shift | What changes | Detectable without labels | Example |
|---|---|---|---|
| Covariate shift | P(x), the input distribution | Yes | A marketing campaign brings a new user mix |
| Label shift | P(y), the outcome base rate | Only through predictions and delayed labels | A seasonal rise in purchase rate |
| Concept drift | P(y given x), the relationship itself | No. This is the hard one | Fraudsters change tactics while inputs look identical |

The uncomfortable consequence: the shift that hurts most is the one no
input-distribution monitor can see. Anyone who proposes feature drift monitoring
as the answer to model decay has answered the easy third of the question.

**Effect sizes that survive large samples.**

| Measure | Formula | Reading it |
|---|---|---|
| Population stability index | Sum over bins of (a_i - b_i) x ln(a_i / b_i), with a and b as bin shares | Under 0.1 is usually noise, 0.1 to 0.25 is worth looking at, above 0.25 is a real move. These bands are an industry convention, not a theorem, and you should calibrate them on your own history |
| Jensen-Shannon divergence | Symmetric, bounded between 0 and 1 | Bounded, so it is comparable across features |
| Difference in means over pooled standard deviation | Standardised mean difference | Interpretable, but blind to shape changes |

Calibrate any threshold you use by replaying the last year: compute the metric
day over day on days when nothing happened, take the 99th percentile of that
distribution, and set the threshold above it. A threshold derived from your own
quiet days is defensible. A threshold copied from a blog post is not.

**The signal ladder, ordered by arrival time.** This is the structure to draw
when asked how you would monitor a model.

| Tier | Signal | Latency | What it catches |
|---|---|---|---|
| 0 | Request volume, error rate, latency, timeout rate | Seconds | Deployment and infrastructure failures |
| 1 | Feature null rate, feature range violations, schema mismatch | Minutes | Upstream pipeline breaks, the most common real incident |
| 2 | Prediction distribution: mean, deciles, share above threshold | Minutes | Model swapped, features silently wrong, input mix moved |
| 3 | Fast proxy labels, for example clicks or immediate dismissals | Hours | Real quality regressions, with enough volume |
| 4 | Realised metric on matured labels, plus calibration ratio by slice | Days to weeks | Everything, but too late to be your only defence |
| 5 | Cumulative effect against a permanent holdout | Months | Slow decay and metric ratchets that per-launch tests never see |

Most teams build tier 4 first because it is the one that sounds like machine
learning, and then discover that tier 1 catches most of their incidents.

**Detection under delayed labels, made concrete.** If your label window is 30
days, tier 4 is 30 days late. Build the bridge with the same arithmetic from
Module 9.

- Measured maturity: F(1 day) = 0.62, F(7) = 0.88, F(30) = 1.00
- A partial 1-day conversion count divided by 0.62 gives an unbiased estimate of
  the eventual 30-day count for that cohort
- Its noise is the noise of the partial count, so a cohort with 200 observed
  conversions has a relative standard error of 1 / sqrt(200) = 7.1 percent
- So you can detect a 15 percent regression within a day, and a 5 percent
  regression takes about nine times the volume or nine times the days

That converts "labels are delayed so we cannot monitor" into "we can detect a 15
percent break tomorrow and a 5 percent break in a week", which is a design, not
an excuse.

**Alert design as an arithmetic problem.** Decide the alert budget first, then
derive the thresholds.

- Suppose you monitor 40 signals, each checked daily
- You will tolerate one false page per week
- Allowed false positive rate per check per day: 1 / (7 x 40) = 0.0036

A single-day threshold at 0.0036 corresponds to roughly 2.9 standard deviations
for a symmetric signal. Alternatively, require two consecutive days outside a 2
standard deviation band: the per-day rate is about 0.046, and two independent
consecutive breaches give 0.046 x 0.046 = 0.0021, which meets the budget with a
tighter band and one extra day of delay.

That is the entire trade, and it fits in two lines:

| Design | Sensitivity | Delay | False pages per week |
|---|---|---|---|
| One day at 2.9 standard deviations | Lower | 1 day | About 1 |
| Two consecutive days at 2.0 | Higher | 2 days | About 0.6 |
| One day at 2.0 | Highest | 1 day | About 13 |

**False-alarm economics, stated plainly.** Alert precision is real incidents
divided by all pages. If you have 2 real incidents a month and 13 false pages a
week, precision is 2 / 58, or 3 percent. At that precision the on-call engineer
stops reading the pages, and your monitoring is now worse than nothing, because
it produces false confidence and real fatigue.

Set a target precision, for example above 30 percent, and treat every false page
as a bug with an owner. The fix is usually not a higher threshold. It is a
better signal: a page on realised click rate is noisy, a page on the null rate of
a top-10 feature almost never fires falsely.

**Training-serving skew is the highest-yield monitor nobody builds.** Log the
exact feature vector used at serving time, then recompute those same features
offline from the same sources and compare row by row.

- Alert on the fraction of rows where any feature differs beyond a tolerance
- A healthy system sits near zero; anything above about 0.1 percent of rows is a
  real bug
- This single check catches the class of failure that offline evaluation is
  structurally incapable of detecting

**What the dashboard must contain, in one screen.**

| Panel | Content | Why it earns the space |
|---|---|---|
| Traffic and health | Requests, errors, p99, timeout and fallback rate | The fallback rate is the one people forget, and a silent fallback is a common outage |
| Input integrity | Null rate and range violations for the top 20 features by gain | Catches most real incidents |
| Prediction distribution | Mean and deciles, versus the same day last week | Detects a swapped or broken model in minutes |
| Label pipeline | Label arrival rate and maturity curve | If this breaks, every metric below it is wrong |
| Quality | Realised metric and calibration ratio, by the same slices used in evaluation | Ties monitoring to the evaluation you already agreed |
| Holdout | Cumulative effect versus a permanent unpersonalised holdout | The only view of slow decay |

```
+---------------------------------------------------------------+
| TIER 0  traffic, errors, latency            seconds           |
+---------------------------------------------------------------+
| TIER 1  feature nulls, ranges, schema       minutes           |
+---------------------------------------------------------------+
| TIER 2  prediction distribution             minutes           |
+---------------------------------------------------------------+
| TIER 3  fast proxy labels                   hours             |
+---------------------------------------------------------------+
| TIER 4  matured labels, calibration ratio   days to weeks     |
+---------------------------------------------------------------+
| TIER 5  permanent holdout, cumulative gap   months            |
+---------------------------------------------------------------+
Caption: the monitoring ladder ordered by how fast each signal
arrives. Build upward from tier 0; a team that starts at tier 4
is blind for the first three days of every incident.
```

### 14d. Worked example

**The prompt.** "Design monitoring for the ad ranking system. Conversions take
up to 30 days. You get one on-call rotation, so do not page them into
uselessness."

Inputs:

| Input | Value | Status |
|---|---|---|
| Scored requests per day | 2 B | GIVEN |
| Click rate | 2 percent | GIVEN |
| Conversion window | 30 days, F(1 day) = 0.62 | GIVEN, measured |
| On-call | One rotation, tolerance one page per week | GIVEN |
| Monitored slices | Vertical, slot, device, new advertiser | From Module 9 |

#### Block A. Purpose: decide what a page is for before choosing any signal

I split the space into three response classes, because a signal with no
distinct response should not be a page.

| Class | Response | Signal tier |
|---|---|---|
| Stop the bleeding now | Automatic rollback, no human | 0 to 2 |
| Wake someone | Page, with a runbook | 1 to 3 |
| Look at it on Monday | Ticket, no page | 3 to 5 |

That mapping settles most arguments about thresholds. A drift metric that
nobody would act on at 3 a.m. is a ticket, not a page, and it should be on a
weekly review rather than in the alerting system.

**The error most readers make here.** Building an alert per metric. The right
unit is an alert per response, and multiple metrics can feed one response.

!!! tip "Self-explanation prompt"

    Before Block B, say which of the six signals from the pre-question artefact
    belongs in each of these three classes, and why the KS p-value belongs in
    none of them.

#### Block B. Purpose: build the fast tiers, which catch most incidents

Tier 0 and tier 1 get automatic responses with no human:

- Error rate above 0.5 percent for 2 minutes: roll back
- p99 above the SLO for 5 minutes: roll back
- Fallback path serving above 1 percent of requests for 10 minutes: page, since
  a silent fallback is invisible in every quality metric
- Null rate on any top-20 feature above 3x its trailing 28-day median: page

I size the null-rate threshold from noise rather than taste. A feature with a
0.3 percent null rate over 2 B requests has 6 M nulls, and the relative standard
error of that count is 1 / sqrt(6e6) = 0.04 percent. Day-to-day variation is
therefore tiny, and a 3x jump is unambiguous. This is the rare signal where a
huge sample makes the monitor better rather than useless.

**The error most readers make here.** Applying the same statistical machinery to
every signal. Count-based integrity signals at high volume are nearly
noise-free, while rate-based quality signals on thin slices are nearly all
noise. Different signals, different thresholds, derived separately.

#### Block C. Purpose: bridge the 30-day label gap

Three quality signals at three latencies:

| Signal | Latency | Detects |
|---|---|---|
| Click rate by slice | Hours | Ranking breakage |
| 1-day conversions divided by F(1 day) = 0.62 | 1 day | Conversion-model breakage, at 15 percent sensitivity per the derivation above |
| 30-day realised calibration ratio by slice | 30 days | Everything, as a confirmation rather than a detector |

The middle row is the one that makes this design work, and it exists only
because we measured the maturity curve in Module 9. I say that dependency out
loud: the delayed feedback work and the monitoring work are the same work.

!!! tip "Self-explanation prompt"

    Say what happens to the middle row if the maturity curve itself shifts, for
    example because a new vertical with slow conversions grows, and name the
    monitor that would catch that.

**The error most readers make here.** Treating the maturity curve as a constant.
It is a measured quantity with its own drift, so it needs its own monthly
refresh and its own comparison against the previous value.

#### Block D. Purpose: fit the page budget

I count the checks: 4 infrastructure, 20 feature integrity, 3 prediction
distribution, and 12 quality checks across slices, so 39 checks.

Budget of one page per week over 39 daily checks gives an allowed per-check
daily false positive rate of 1 / (7 x 39) = 0.0037.

Applying it:

- Integrity checks are near noise-free, so they use a fixed multiple rather than
  a statistical band and contribute almost no false pages
- Quality checks use two consecutive days outside a 2 standard deviation band,
  giving roughly 0.002 per check per day and about 0.6 pages per week across all
  12
- Prediction distribution uses a single-day 3 standard deviation band because
  its response is a rollback and delay is expensive there

Total expected false pages: under one per week, with the fast signals kept fast.
I write this arithmetic into the runbook, because the next engineer who adds 20
checks needs to see that the budget is a shared resource.

**The error most readers make here.** Adding monitors without a budget. Alerting
systems degrade by accretion, and the failure is gradual until the rotation
stops reading pages entirely.

#### Block E. Purpose: catch the decay that no page will ever fire for

A permanent 0.5 percent holdout served by a fixed non-personalised policy,
never changed, never optimised.

Its purpose is not to detect an incident. It is to answer a question no
experiment can: what has the sum of two years of launches actually delivered
against doing nothing. Individual tests each measured a small win against the
then-current baseline, and those wins do not add up cleanly, because each one
shifted the population the next was measured on.

The cost is honest and I state it: 0.5 percent of users get a worse product
permanently. At 2 B requests a day that is 10 M requests, and the design should
rotate which users are in the holdout only if the metric being measured does not
require a persistent cohort, which for cumulative effects it does. That is a
real trade with a real cost, and it is the trade that lets you answer the
question at all.

**The error most readers make here.** Proposing the holdout and then letting it
be rotated, resized or quietly switched off during a capacity incident. A
holdout that changes composition measures nothing, so its protection has to be
written into the deploy process, not into a team's good intentions.

### 14e. Fade the scaffold

**Fade 1: a content moderation model.** Labels come from human reviewers with a
2 to 5 day queue, and enforcement changes attacker behaviour. Last block blank.

- Block A, what a page is for: automatic rollback on serving failures, page on
  review-queue volume breaching capacity, ticket for drift metrics.
- Block B, fast tiers: flagged rate per hour by content type, model score
  distribution, and the fraction of items hitting the default action because a
  feature was missing.
- Block C, bridging the label gap: reviewer decisions arrive in days, so use
  reviewer agreement rate on a small sample of already-actioned items as the
  fast proxy, and reserve full precision and recall for the weekly readout.
- Block D, page budget: **you write this block.**

??? note "Show answer"

    Count the checks and allocate. Roughly 4 infrastructure, 10 integrity, 3
    distribution and 6 quality checks by content type gives 23 daily checks, so a
    one-page-per-week budget allows 1 / (7 x 23) = 0.0062 per check per day.

    Then depart from the arithmetic where the cost is asymmetric. The
    review-queue capacity breach is not a statistical alert at all: it is a
    threshold on a known operational limit, it fires rarely, and its false
    positive rate is essentially zero, so it does not consume budget. The
    flagged-rate alerts do consume budget and should use two-consecutive-period
    logic.

    Add one thing the arithmetic does not produce: a manual weekly review of 50
    sampled enforcement decisions. Adversarial systems fail in ways that look
    normal in aggregate, and no threshold on a rate will show you a new evasion
    technique that keeps the flagged rate constant.

**Fade 2: an ETA model in a delivery product.** Labels arrive within an hour of
each trip, so there is no label delay problem at all. Last two blocks blank.

- Block A, what a page is for: rollback on serving failures, page when realised
  coverage of the promised time drops, ticket for feature drift.
- Block B, fast tiers: prediction distribution by city, null rate on traffic and
  courier features, and the rate at which the fallback estimate is served.
- Block C, quality signals: **you write this block.**
- Block D, page budget: **you write this block.**

??? note "Show answer"

    Block C. With hour-old labels, the realised metric is the monitor, so skip
    proxies entirely and monitor what the product promises: the fraction of trips
    arriving by the promised time, per city, per hour. Add the late-tail mass,
    meaning the share of trips more than 10 minutes past promise, because a
    model can hold coverage while making the tail worse and the support-contact
    cost lives in the tail. Slice by city and by peak versus off-peak, since a
    national average hides a city-level failure for days.

    Block D. Cities differ enormously in volume, so one threshold cannot serve
    all of them. Set the band per city from that city's own trailing variance,
    and require a minimum trip count before a city can page at all: at 200 trips
    per hour, a coverage estimate has a relative standard error near 7 percent,
    so smaller cities aggregate to daily rather than hourly checks. Then the
    budget is met by construction, because the number of eligible hourly checks
    is small and each is genuinely low-noise.

### 14f. Check yourself

**Q1.** A colleague reports that a Kolmogorov-Smirnov test on 12 features shows
significant drift on 9 of them, at a daily volume of 50 M rows, and proposes an
emergency retrain. What do you say?

??? note "Show answer"

    At 50 M rows per side, the KS critical value at a 5 percent level is about
    1.36 x sqrt(2 / 5e7) = 0.00027, so any difference larger than about three
    hundredths of a percent in the cumulative distribution registers as
    significant. Nine of twelve features clearing that bar is what a healthy
    system looks like, not an incident.

    Ask for effect sizes: population stability index per feature, plus the
    prediction distribution and the realised metric. If the stability index is
    under 0.1 and the realised metric is flat, there is nothing to retrain for.
    Then fix the monitor: replace the p-value with an effect size, and set the
    threshold from the 99th percentile of quiet-day variation in your own
    history.

**Q2.** Your model's realised metric has been flat for six months and every one
of the last eight experiments showed a positive lift. Reconcile these two facts,
and name the single instrument that would have told you which explanation is
right.

??? note "Show answer"

    Several explanations fit. The lifts may have been real but measured against a
    baseline that was itself degrading, so the sum of the launches held the metric
    level rather than raising it. The lifts may have been novelty effects that
    decayed after each readout ended. They may have cannibalised each other,
    since each was measured while the others were live. Or the metric may be
    saturated for reasons outside the model, such as a demand-side ceiling.

    The instrument that distinguishes them is a permanent untouched holdout
    population. It measures the cumulative effect of everything shipped against
    doing nothing, which is precisely the question that a sequence of pairwise
    experiments cannot answer. Without it you can argue about these explanations
    indefinitely. With it, the answer is one chart.

### 14g. When not to use this

**Feature drift monitoring across the whole feature set.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You have a past incident where a feature distribution moved before quality did, and you can name the response you would take when it fires |
| Scale threshold below which it is over-engineering | Below a few hundred thousand rows a day, drift metrics on many features are mostly sampling noise. Below that, monitor the null rate and the prediction distribution and stop |
| Cheaper alternative | Monitor the top 10 features by gain plus the prediction distribution. The prediction distribution is a weighted summary of every input the model actually uses |

**A permanent non-personalised holdout.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You ship enough changes that the cumulative question is meaningful, say more than 10 launches a year on the same surface, and you can state the cost per holdout user |
| Scale threshold | Under about 1 M daily users, a 0.5 percent holdout is too small to measure anything and you have simply made 5,000 people's product worse for nothing |
| Cheaper alternative | Periodic reverse experiments: turn personalisation off for a random 1 percent for two weeks, once or twice a year, and accept a coarser answer |

**Sequential or change-point detection methods.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Detection delay has a measurable cost, for example every hour of an undetected fraud model failure costs a known amount, and daily checks are demonstrably too slow |
| Scale threshold | If your response to an alert takes a business day anyway, faster detection buys nothing. Match detection latency to response latency |
| Cheaper alternative | A two-consecutive-period rule on a simple threshold, which is easy to explain, easy to tune, and easy for the on-call engineer to reason about at 3 a.m. |

**Automatic retraining triggered by a drift alert.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You have shown, on historical incidents, that retraining actually recovered the metric. Most drift alerts are pipeline bugs, and retraining on broken data makes it worse |
| Scale threshold | Any system without an automatic rollback path should not have automatic retraining, at any scale. The two are the same investment and rollback comes first |
| Cheaper alternative | A drift alert that opens a ticket with the diagnostic bundle attached: the affected slice, the feature, and a link to the last successful pipeline run |
---

## Module 15. Pattern and anti-pattern catalogue

**Time: 30 to 40 minutes reading, 25 to 35 minutes practice.**

Distributed systems interviews have a shared vocabulary of failure: retry storm,
cache stampede, hot key, split brain. Machine learning systems have the same
recurring failures and almost no shared names for them, so candidates describe
them from scratch every time and interviewers cannot tell recognition from
improvisation. This module gives them names, symptoms and repairs.

### 15a. Predict first

```
+---------------------------------------------------------------+
| A three year old recommender. Six observations, one day:      |
| 1. Offline AUC has risen every quarter for two years          |
| 2. The realised engagement metric is flat over the same time  |
| 3. 11 percent of requests are served by the heuristic path    |
| 4. New items take 6 days to reach 1,000 impressions           |
| 5. The moderation threshold has not been changed since launch |
| 6. Nobody can name the owner of the second-stage ranker       |
+---------------------------------------------------------------+
Caption: six independent symptoms observed on one day in a mature
recommendation system, with no incident open.
```

**Question.** Each line is a named failure. Name as many as you can, then say
which one you would repair first and why.

??? note "Show answer"

    Line 1 and 2 together are the metric ratchet: every launch beat the
    then-current baseline, and the sum of those wins is not visible because no
    permanent holdout exists to measure it.

    Line 3 is the silent fallback. Eleven percent of traffic bypasses the model
    and every quality metric averages the two populations, so the model looks
    worse than it is and the fallback's failure is invisible.

    Line 4 is popularity collapse driven by the feedback loop: the ranker trains
    on its own impressions, so items it never shows never earn the data that
    would make it show them.

    Line 5 is the frozen threshold. Score distributions drift with every
    retrain, so a fixed cut point silently moves the operating point.

    Line 6 is a zombie model: serving, unowned, and therefore unrepairable
    within any reasonable time.

    Repair order: line 3 first. It is the cheapest to fix, it corrupts every
    other measurement on the list, and until it is fixed you cannot trust the
    numbers you would use to prioritise the rest.

**How to use a catalogue in an interview.** Not as a recital. The value is
recognition speed: when the interviewer describes a symptom, you name the class,
state the mechanism in one sentence, and propose the detection before the
repair. That sequence is what separates someone who has operated a system from
someone who has read about one.

**Serving patterns.**

| Pattern | One-line definition | The measurement that justifies it |
|---|---|---|
| Candidate funnel | Retrieve, filter, rank, rerank as separate stages with separate budgets | Candidate count times per-candidate cost exceeds the latency budget for a single-stage design |
| Precompute plus online rerank | Batch produces a shortlist, an online pass reorders the top slice with fresh features | Share of requests whose top-k changes within the batch period |
| Embedding split | Item vectors precomputed, user vector computed at request time, matched by nearest neighbour | Item count times item encoder cost is too large to do per request |
| Model cascade | A cheap model narrows the field before an expensive one | Overlap between the cheap stage's survivors and the full model's final top-k |
| Log at serve | The exact feature vector used at inference is written to the log | Any difference at all between offline and online feature values |
| Policy layer outside the model | Business rules, caps and diversity applied after scoring, never inside the loss | Number of rules that change on a weekly cadence |
| Fallback ladder | Model, then heuristic, then static, each with its own monitor | Fallback rate, which must be a first-class monitored metric |
| Bounded staleness cache | Cached with an explicit maximum age stated in the design | Measured repeat rate on the cache key |

**Training patterns.**

| Pattern | One-line definition | The measurement that justifies it |
|---|---|---|
| Point-in-time join | Every feature value is the value as of the prediction time, never later | Difference between naive-join and point-in-time offline metrics |
| Warm chain with a cold anchor | Warm start daily, cold retrain weekly, compare on a fixed set | Divergence between the warm chain and the cold anchor |
| Sample and correct | Downsample negatives for speed, invert the odds in closed form at serving | Training wall clock against cadence, and calibration ratio after the inverse |
| Distil to the budget | Train a large teacher offline, serve a small student | Quality gap between teacher and student on your own data |
| Shared trunk, multiple heads | One representation, several task heads with separate losses | Correlation between tasks, and per-task quality against separate models |
| Delayed label reconciler | A job that revisits old predictions as labels mature and repairs the training set | Label maturity curve, per segment |

**Operations patterns.**

| Pattern | One-line definition | The measurement that justifies it |
|---|---|---|
| Registry pin and warm rollback | Previous model stays loaded; rollback is a pointer flip | Mean time to rollback, measured in a drill, not assumed |
| Golden set gate | A fixed labelled set must not regress before promotion | Historical incidents that a fixed set would have caught |
| Permanent holdout | A never-optimised population measures cumulative effect | Number of launches per year on the same surface |
| Alert budget | A fixed page budget allocated across checks | Pages per week, and alert precision |
| Exploration budget with logged propensity | A share of traffic acts randomly and records the probability | Fraction of the action space the current policy never visits |

Sculley and colleagues named the general problem of accumulating machine
learning system debt in 2015
([Hidden Technical Debt in Machine Learning
Systems](https://papers.nips.cc/paper_files/paper/2015/hash/86df7dcfd896fcaf2674f757a2463eba-Abstract.html),
NeurIPS 2015, accessed August 2026). Their paper names categories. What follows
adds what an operator needs and what an interview rewards: the symptom you would
observe, and the detection that separates one cause from another.

**The anti-pattern catalogue.**

| Name | Symptom you observe | Mechanism | Detection | Repair |
|---|---|---|---|---|
| Feature time travel | Offline metric far better than online, repeatedly | A feature was computed with data from after the prediction time | Recompute one feature strictly as of prediction time and compare the offline metric | Point-in-time joins, and a test that fails the build when a join is not time-bounded |
| Two pipelines, one truth | Unexplained gap between offline scores and logged serving scores | Features implemented twice, in two languages, by two teams | Replay logged serving features offline and diff row by row | One feature definition, used by both paths, with the serving path as the source of truth |
| The silent fallback | Quality metrics slowly worsen with no code change | A fallback path serves a growing share of traffic and is not in any dashboard | Fallback rate as a tier 0 metric | Alert on fallback rate, and treat fallback share above a threshold as an incident |
| The frozen threshold | Review volume or block rate drifts without a decision | Score distribution moves with each retrain while the cut point stays fixed | Plot the share of traffic above the threshold over months | Set the threshold by target rate, recalibrated per retrain, not by score value |
| Retrain on your own shadow | Catalogue coverage narrows, new items never take off | The model trains on impressions it chose, so unshown items never earn data | Impression share of the top 1 percent of items over time | Exploration budget, propensity logging, popularity-debiased evaluation |
| Precision theatre | Excellent offline metrics, terrible production precision | Evaluation ran on a rebalanced set with an artificial base rate | Compare the base rate of the eval set to production | Evaluate on the production distribution, always, and report the operating point |
| The champion's eval set | New retrieval approaches always look worse offline | The evaluation set contains only what the incumbent already showed | Check what fraction of eval items the incumbent ranked in its top k | Add a randomised-serving slice, and use online tests for retrieval changes |
| Metric ratchet | Every launch wins, the aggregate never moves | Each experiment is measured against a baseline that the previous launch already shifted | Permanent holdout comparison | Keep the holdout, and report cumulative effect quarterly |
| Label lag denial | Model appears to underpredict, always, and nobody can say why | A delayed label is treated as final at the observation window | Compare cohort maturity curves against the training window | Uplift by maturity, or a delay model, per Module 9 |
| Dashboard with no denominator | Counts rise, alarm follows, nothing is wrong | Absolute counts are plotted without exposure, so traffic growth reads as regression | Divide by requests and re-plot | Rates on every panel, counts only for capacity |
| Zombie model | Nobody can say who owns a serving component | Ownership decayed faster than the system | Registry entry with an owner and a last-reviewed date | Ownership metadata as a deploy requirement, and a scheduled review |
| Alert accretion | On-call ignores pages | Checks were added without a budget, so precision fell below usefulness | Alert precision: real incidents over total pages | A fixed page budget, and every false page treated as a bug |
| The unbounded feature | Memory grows, then a crash, then a hasty hash | A categorical feature's cardinality grows without a cap | Cardinality per feature over time | A frequency floor and an explicit hashing policy with a measured collision cost |
| Model-shaped product bug | Users complain about quality, offline and online metrics are fine | The model is correct and the surface misrepresents it, for example stale text next to a fresh ranking | Look at 20 real screens, not 20 rows | Fix the surface. Resist retraining |

**Three that cost the most, with their arithmetic.**

**Feature time travel.** The commonest single reason an offline win vanishes. The
detection is cheap and specific: pick the feature with the highest gain,
recompute it strictly as of the prediction timestamp, and re-run the evaluation.
If the metric drops materially, you have found it. Teams skip this because it
takes a day, and then spend a quarter on an experiment that was never going to
work.

**The silent fallback.** Suppose 11 percent of traffic is served by a heuristic
whose click rate is 1.2 percent while the model's is 2.1 percent. The reported
blended rate is 0.89 x 2.1 + 0.11 x 1.2 = 1.87 + 0.13 = 2.00 percent.

The model looks 5 percent worse than it is, and every experiment run on blended
traffic is diluted by the same fraction, so an 11 percent fallback rate silently
reduces every measured effect by about 11 percent. That is often the difference
between a significant result and a flat one.

**Metric ratchet.** Eight launches, each measured at plus 1.5 percent against
its own baseline. Naive compounding predicts 1.015 to the power 8, which is plus
12.6 percent. The holdout says plus 2 percent. The gap is not fraud, it is
arithmetic: effects measured against a moving baseline do not compose, novelty
decays, and launches cannibalise each other. Without a holdout you cannot tell a
team that ships well from a team that ships noise.

```
+-------------------+   +--------------------+   +----------------+
| DATA              |   | TRAINING           |   | SERVING        |
| feature time      |   | precision theatre  |   | silent         |
| travel            |   | champion eval set  |   | fallback       |
| two pipelines     |   | label lag denial   |   | frozen         |
| unbounded feature |   | warm chain drift   |   | threshold      |
+---------+---------+   +----------+---------+   +--------+-------+
          |                        |                      |
          +------------------------+----------------------+
                                   |
                                   v
                    +---------------------------+
                    | OPERATIONS                |
                    | metric ratchet            |
                    | alert accretion           |
                    | zombie model              |
                    | retrain on your own shadow|
                    +---------------------------+
Caption: the four zones where ML systems rot, with the named
failures that live in each. Failures in the operations zone are
the ones no single launch review will ever catch.
```

### 15d. Worked example

**The prompt.** "Here is a running recommender with the six symptoms from the
artefact above. You have one quarter and two engineers. What do you do, in what
order, and what do you deliberately not do?"

#### Block A. Purpose: separate what corrupts measurement from what costs money

I sort the six symptoms by whether they break my ability to measure anything
else, because repairs made on corrupted measurements are guesses.

| Symptom | Corrupts measurement | Costs money or quality directly |
|---|---|---|
| Silent fallback at 11 percent | Yes, dilutes every metric and every experiment | Yes, 11 percent of users get a worse product |
| Metric ratchet, no holdout | Yes, makes the whole roadmap unfalsifiable | No, not directly |
| Frozen threshold | Partly, the operating point is unknown | Yes, if the drift went the expensive direction |
| Popularity collapse | Yes, poisons the training data | Yes, over quarters |
| Zombie second-stage ranker | No | Yes, as unbounded incident risk |
| Rising AUC, flat engagement | This is the observation, not a cause | n/a |

Ordering falls out: fix the measurement corruptions first, cheapest first.

**The error most readers make here.** Starting with the most intellectually
interesting item, which is always the feedback loop. It is the slowest to repair
and the hardest to measure, and it should be third.

!!! tip "Self-explanation prompt"

    Before Block B, say why the zombie model appears low on this list despite
    being an obvious risk, and what single piece of evidence would move it to the
    top.

#### Block B. Purpose: repair the cheap corruptions in the first three weeks

Week 1: fallback rate becomes a tier 0 metric with an alert. Then find out why
11 percent. In my experience the answer is usually one of three: a timeout that
is too tight for a slow dependency, a null feature triggering the guard path, or
a class of requests the model was never given inputs for. Each is a days-long
fix, and each removes a chunk of the 11 percent.

Week 2: the frozen threshold becomes a target-rate threshold. Instead of "block
above 0.83", the rule becomes "block the top 0.4 percent of scored items", with
the score cut recomputed after every promotion. The target rate is set by review
capacity, which is a real constraint, rather than by a number someone picked in
2023.

Week 3: launch the permanent holdout at 0.5 percent, and state honestly that it
gives no answer for a quarter. That is the reason to do it in week 3 rather than
week 12.

**The error most readers make here.** Rewriting the fallback path rather than
first measuring which of the three causes it is. The fix is cheap once the cause
is known and expensive when guessed.

#### Block C. Purpose: attack the feedback loop with a measurement, not a redesign

I do not propose a new architecture for popularity collapse. I propose an
instrument and a small budget.

- Measure: impression share of the top 1 percent of items, monthly, over the
  last two years. This turns an argument into a trend.
- Instrument: log the serving propensity, which costs one field and unlocks
  every off-policy method later.
- Budget: 1 percent of requests get a slot filled by an item sampled from
  outside the model's top 100, with the sampling probability logged.

Then measure the cost of that 1 percent directly, so the exploration budget is
defended by a number rather than by principle. If the exposed users lose 3
percent of engagement on 1 percent of traffic, the cost is 0.03 percent of the
metric, which is a rounding error against the value of being able to evaluate
anything counterfactually.

!!! tip "Self-explanation prompt"

    Say why logging the propensity is worth doing even in a quarter where you run
    no off-policy evaluation at all.

**The error most readers make here.** Proposing a diversity term in the loss as
the first move. It changes the objective before anyone has measured the
problem, and it makes the next quarter's comparisons harder.

#### Block D. Purpose: state what I will not do, and why

| Not doing | Why |
|---|---|
| Retraining anything in the first month | Every measurement is currently corrupted, so a retrain would be evaluated against numbers I do not trust |
| Replacing the second-stage ranker | Zombie status is an ownership problem, not a code problem. Assign an owner, add a registry entry, and leave the code alone until it fails a measurement |
| A new model architecture | Nothing on the symptom list is caused by model capacity, and the offline metric has been rising for two years without helping |
| Reworking the offline evaluation harness | It is producing rising numbers that do not transfer, which is worth fixing, but it is a quarter of work and it is fourth in line |

Closing sentence I would actually say: "Two engineers, one quarter, and the
plan is measurement repair, an operating-point fix, a permanent holdout and a 1
percent exploration budget. No new model. I expect the first honest read on
whether this system is improving in about four months, and that is the main
deliverable."

**The error most readers make here.** Producing the not-doing list and then
apologising for it. The list is the strongest part of the answer, because a
plan that fits two engineers into a quarter is only credible once you have said
what it excludes.

### 15e. Fade the scaffold

**Fade 1: a fraud system.** Symptoms: precision reported at 94 percent offline
and 31 percent in production; the review queue has doubled with no rule change;
the model was last retrained 8 months ago; blocked transactions have no
outcome labels at all. Last block blank.

- Block A, separate corruption from cost: the offline-production precision gap
  is a measurement corruption, and the doubled queue is a direct cost. The
  missing labels on blocked transactions corrupt everything downstream.
- Block B, repair the cheap corruptions: the 94 versus 31 gap is precision
  theatre, so re-evaluate on the production base rate at the production
  threshold. Then convert the fixed threshold to a target review rate matched to
  team capacity, which fixes the doubled queue immediately.
- Block C, attack the structural problem: blocked transactions have no labels,
  so add a small random-release slice with logged probability. Without it, the
  model can never acquire data on the region it currently blocks.
- Block D, what I will not do: **you write this block.**

??? note "Show answer"

    Not retraining as the first move, even at eight months stale. Retraining on
    data whose positive region has never been observed will reproduce the current
    blind spot with more confidence, and the measurement to prove improvement
    does not yet exist.

    Not adding features. The gap between 94 and 31 percent is explained by the
    evaluation setup, not by model capacity, so features would be tuned against a
    number that is wrong.

    Not raising the threshold to shrink the queue, because that hides a capacity
    problem inside a quality decision and the effect on caught fraud is unknown
    until the operating point is measured properly.

    Say the deliverable plainly: an honest precision and recall curve at the
    production base rate, a threshold tied to capacity, and a labelled random
    slice. Then retrain, in month two, against numbers that mean something.

**Fade 2: a search ranking system.** Symptoms: every retrieval change loses
offline and wins online; the offline harness is trusted by leadership; a
cross-encoder reranker added last year serves 100 percent of traffic and nobody
has measured its contribution. Last two blocks blank.

- Block A, separate corruption from cost: retrieval changes losing offline while
  winning online is the champion's eval set, a pure measurement corruption. The
  unmeasured reranker is a cost of unknown size.
- Block B, repair the cheap corruptions: **you write this block.**
- Block C, attack the structural problem: **you write this block.**

??? note "Show answer"

    Block B. The evaluation set contains only documents the incumbent retrieved,
    so a new retriever that surfaces unseen documents is scored against labels
    that do not exist and is penalised for finding them.

    Measure the size of the problem first: what fraction of the eval set's
    judged documents were in the incumbent's top 100. If it is above about 90
    percent, the harness cannot evaluate retrieval at all, and leadership needs
    to hear that in those words. The cheap repair is to route retrieval changes
    to online interleaving and to stop gating them on the offline number.

    Block C. Run a holdback on the reranker: 1 percent of traffic served without
    it, for two weeks. This is the cheapest possible answer to "what is this
    component worth", it needs no new code beyond a flag, and it is the honest
    response to a component that has never been measured. State the possible
    outcome that most teams avoid saying out loud: if the reranker contributes
    under its latency cost, the correct action is to remove a box, and removing a
    box is a shippable result.

### 15f. Check yourself

**Q1.** An interviewer says: "Our offline metric has improved for six straight
quarters and our engagement metric has not moved. What is going on?" Give three
candidate explanations and the single instrument that separates them.

??? note "Show answer"

    Three candidates. First, the metric ratchet: each launch was measured against
    a baseline the previous launch had already moved, so the wins do not compose.
    Second, an offline-online mismatch: the offline metric measures re-ordering
    of what the incumbent already retrieved, which is not what changes user
    behaviour. Third, dilution by a silent fallback or a growing segment the
    model does not serve, which shrinks every measured effect.

    The instrument that separates them: a permanent holdout answers the first
    directly; a comparison of the offline metric's sign against the online
    result across the last ten experiments answers the second; and a fallback
    rate panel answers the third. Two of those three cost almost nothing and can
    be in place next week, which is the part of the answer that shows you have
    done this rather than read about it.

**Q2.** You inherit a model whose block threshold has not changed in two years.
Nothing appears wrong. Name the specific measurement you would take, and the two
possible results and what each implies.

??? note "Show answer"

    Measure the share of scored traffic above the threshold, plotted monthly for
    two years, alongside the model's promotion dates.

    Result one: the share is stable. Then the score distribution has been stable
    across retrains, the frozen threshold has cost nothing so far, and the repair
    is preventive rather than urgent: convert to a target rate so that the next
    retrain cannot silently move it.

    Result two: the share has drifted, say from 0.4 to 0.9 percent. Then the
    operating point has moved by more than a factor of two with no decision, and
    the consequences are already in the business: doubled review load, or doubled
    false blocks, depending on direction. The repair is immediate, and the
    interesting question becomes which retrain moved it and why nobody noticed,
    which is a monitoring gap as much as a threshold gap.

### 15g. When not to use this

**A named pattern catalogue as an answer structure.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The interviewer has described a symptom in a running system, and naming the class shortens the diagnosis |
| Scale threshold below which it is over-engineering | On a greenfield prompt with no running system, reciting failure names is padding. Design the thing first, then name the two failures your design is most exposed to |
| Cheaper alternative | One sentence: "the failure I would expect first here is X, and I would detect it with Y" |

**A permanent exploration budget.**

| Question | Answer |
|---|---|
| Measurement that justifies it | Measured coverage collapse, for example impression share of the top 1 percent of items rising year over year, plus a measured cost per exposed user |
| Scale threshold | With a catalogue small enough that everything gets impressions anyway, say under a few thousand items, there is no collapse to prevent |
| Cheaper alternative | Freshness boosts for new items with a decay, which is cruder, cheaper and reversible in an afternoon |

**A permanent holdout for cumulative effect.**

| Question | Answer |
|---|---|
| Measurement that justifies it | More than roughly ten launches a year on one surface, so the composition question is real, and enough traffic that 0.5 percent is statistically usable |
| Scale threshold | For a system shipping two changes a year, the per-launch experiments already answer the question and the holdout is a permanent cost for nothing |
| Cheaper alternative | An annual reverse experiment: switch the model off for a random 1 percent for two weeks and measure the gap |

**Rewriting a legacy component you have identified as a zombie.**

| Question | Answer |
|---|---|
| Measurement that justifies it | It has failed a measurement, not merely an ownership check: it costs latency you need, or it has caused an incident, or a holdback shows its contribution is below its cost |
| Scale threshold | If it serves correctly and cheaply, unowned code is an ownership problem with a one-hour fix, not an engineering project |
| Cheaper alternative | Assign an owner, add a registry entry with a last-reviewed date, write the runbook, and schedule a holdback experiment to price it before touching the code |
---

## Module 16. Level calibration

**Time: 25 to 35 minutes reading, 25 to 35 minutes practice.**

Every course tells you what to say. Almost none tells you what a mid, senior or
staff version of the same answer sounds like, which is the thing candidates
actually ask for. This module answers one prompt three times and annotates every
difference.

### 16a. Predict first

```
+---------------------------------------------------------------+
| Three answers to: "how would you know this model is working?" |
| A: "I would track AUC offline and click-through rate online,  |
|     and set up dashboards for both."                          |
| B: "Click rate is the model's own objective, so I would make  |
|     revenue per user primary and click rate a diagnostic,     |
|     with unsubscribe rate as a guardrail at a stated margin." |
| C: "Before picking metrics I would ask what decision this     |
|     model's output changes, because our current metric moved  |
|     3 percent last year while the holdout moved 0.4, and I    |
|     want the measurement fixed before we optimise against it."|
+---------------------------------------------------------------+
Caption: three answers to one follow-up question, differing not in
knowledge but in what each treats as already settled.
```

**Question.** Assign each answer to mid, senior or staff, and state the single
structural difference that made the assignment, without using the word
experience.

??? note "Show answer"

    A is mid, B is senior, C is staff.

    The structural difference is scope of what each answer treats as given. A
    accepts the metric as given and answers about instrumentation. B accepts the
    system as given and answers about metric design, including which metric has
    decision authority. C accepts nothing as given and questions whether the
    measurement apparatus can support the decision at all, using a number to do
    it.

    Notice what is not the difference: none of these answers contains a technique
    the others could not name. Level shows in what you consider your problem, not
    in what you know.

**The mistake this module exists to correct.** Candidates try to level up by
adding vocabulary. Interviewers read vocabulary as noise and read scope,
prioritisation and self-correction as level. The additions that actually raise a
rating are structural.

| Level | What the answer treats as its problem | Typical failure at this level |
|---|---|---|
| Mid | Building the described system correctly | Builds the wrong thing correctly, because the objective was accepted as stated |
| Senior | Choosing among correct designs under stated constraints, and naming what breaks | Optimises inside a frame nobody validated, and rarely says what to delete |
| Staff | Choosing which problem is worth solving, and making the choice measurable and reversible | Abstracts so far that no system gets designed, which reads as evasion |

**The eight behaviours that move a rating, in rough order of value.**

| Behaviour | What it looks like in a transcript |
|---|---|
| Restating the objective in a form you can train and measure | "Engagement is the goal. The trainable proxy is X and it diverges from the goal in these two cases" |
| Naming what you will not build, with a reason | "No streaming trainer. Daily retrain fits, and I will show you the arithmetic" |
| Attaching a number to a trade-off, derived on the spot | "That fan-out means each shard needs its p99.9, so I would rather partition" |
| Naming the measurement that would change your mind | "If the top-1-percent impression share is flat, my feedback loop concern is wrong" |
| Updating when the interviewer pushes, and saying what changed | "That is a better point than mine. It moves the retrain cadence, so the streaming path is back on the table" |
| Sequencing under a real constraint | "Two engineers, one quarter, so measurement repair first and no new model" |
| Naming failure classes before being asked | "The first thing that breaks here is the silent fallback, and I would monitor its rate at tier 0" |
| Saying "I do not know" and then saying how you would find out | "I do not know our label maturity curve. I would measure it on cohorts older than 45 days" |

**What reads as over-reaching.** Staff behaviours performed without the
underlying design are worse than an honest mid answer.

| Over-reach | Why it lands badly |
|---|---|
| Questioning the prompt without ever designing anything | Reads as avoidance. Design first, question second, or question in one sentence and then design |
| Organisational framing with no system underneath | "This is really a team alignment problem" is a staff sentence only after the system is on the board |
| Numbers with no derivation | A confident unsourced figure is worse than no figure, because it invites one question you cannot answer |
| Naming many patterns quickly | Reads as a catalogue recital. One named failure with its detection beats six names |

```
+---------------------------------------------------------------+
| MID     builds the system as described, correctly              |
|         objective given, metrics given, scope given            |
+---------------------------------------------------------------+
                            | adds: chooses between designs,
                            v        names what breaks, deletes
+---------------------------------------------------------------+
| SENIOR  derives the design from constraints, with numbers      |
|         and states what it does NOT build                      |
+---------------------------------------------------------------+
                            | adds: questions the objective,
                            v        makes it measurable, sequences
+---------------------------------------------------------------+
| STAFF   chooses the problem, fixes the measurement, and        |
|         leaves a reversible path for whoever inherits it       |
+---------------------------------------------------------------+
Caption: each level contains the one below it. A staff answer that
skips the senior layer reads as evasion, not as seniority.
```

### 16d. Worked example

**The prompt.** "Design the system that decides which push notifications each
user receives, and how many."

Shared inputs for all three answers, so the comparison is fair:

| Input | Value | Status |
|---|---|---|
| Users | 200 M, 60 M daily active | GIVEN |
| Candidate notifications generated per user per day | 40 | GIVEN |
| Current policy | Send anything above a fixed score, capped at 5 per day | GIVEN |
| Current metric | Notification open rate | GIVEN |
| Assumed 30-day user value | 20 currency units | ASSUMED, and I would ask for the real figure. Every threshold below scales with it |

#### Block A. Purpose: the mid answer, in full, so the deltas are visible

"I would build a ranking system. Candidate notifications are generated by
upstream producers, then a model scores each one for the probability the user
opens it. I would use gradient boosted trees on user features, notification
type, time of day and recent activity. Train on the last 30 days of send and
open logs.

Serving is online, since candidates are event-driven. I would rank the
candidates and send the ones above a threshold, up to the cap of 5.

For evaluation I would use AUC offline and open rate online, with an A/B test.
For monitoring I would track open rate and model latency, and retrain weekly."

**What is right about this.** The pipeline is correct, the model choice is
defensible, the serving mode matches the trigger, and it would work.

**What caps it at mid.** Every given was accepted. The objective is open rate,
so the system will learn to send whatever gets opened, which is
notification-shaped clickbait. The cap of 5 was inherited, not derived. There is
no notion of what a notification costs the user, so the system has no way to
choose zero.

**The error most readers make here.** Believing the mid answer is wrong. It is
not wrong, it is unbounded: nothing in it can ever conclude "send nothing".

!!! tip "Self-explanation prompt"

    Before Block B, write the one sentence you would add to the mid answer that
    would most raise its level, and check it against the eight behaviours table.

#### Block B. Purpose: what the senior answer adds, item by item

The senior version keeps the pipeline and adds five things:

**1. It separates the goal from the trainable proxy.** "Open rate is a proxy,
and it is the wrong one on its own, because a notification that is opened and
regretted is a loss. I would train on open, then evaluate against downstream
session length and next-day return, and I would add unsubscribe and uninstall as
guardrails with stated margins."

**2. It makes the cap a decision, not an inheritance.** "The cap of 5 is a
constant nobody derived. I would measure the marginal effect of the nth
notification of the day by capping at n for a random slice, and set the cap
where the marginal notification stops paying."

**3. It notices that a threshold requires calibration.** "The send decision is a
fixed threshold, so the score must be calibrated, not just ranked. If the score
distribution shifts after a retrain, the send volume moves with no decision,
which is the frozen threshold failure. I would set the cut by target send rate
and recalibrate on every promotion."

**4. It handles the delayed label.** "The open label matures over about a day.
That caps retrain cadence at daily unless I use a shorter window with an uplift
factor, and I would only do that if the content half-life is under a day."

**5. It states what it will not build.** "No streaming trainer, no per-user
reinforcement learning loop, no separate model per notification type. Daily
retraining with a type feature fits inside the cadence, and I would revisit if
the per-type quality gap exceeds a measured threshold."

**The error most readers make here.** Adding the guardrails but never stating a
margin. "I would watch unsubscribes" is a mid sentence. "Unsubscribe rate must
not rise by more than 0.05 percentage points, one-sided" is a senior sentence.

#### Block C. Purpose: what the staff answer adds, item by item

The staff version keeps everything above and adds four things, each of which is
a scope change rather than a technique.

**1. It prices the send in a common currency, and derives the reserve.** "Every
producer wants to send, so the real system is an allocation problem under a
shared user attention budget. I would give every candidate a value in one
currency: expected incremental sessions minus the expected cost in churn risk.

Take the measured numbers we would need: if the marginal notification produces
0.006 additional sessions at an assumed session value of 0.05, the gain is
0.0003. If it adds 0.0004 to the probability of uninstall against an assumed
30-day user value of 20, the cost is 0.008. Cost exceeds gain by more than an
order of magnitude, so at that margin the correct action is to send nothing, and
the reserve price is whatever value makes those two terms equal."

That derivation is the whole staff move: it converts "how many notifications"
from a product argument into a break-even calculation with named, measurable
inputs.

**2. It names the organisational failure that will defeat the design.** "Every
producing team is measured on the opens their own notifications get, so each
will lobby for exemptions from the budget. The system will not survive unless
the budget is enforced centrally and producers are measured on their share of
the user-level outcome rather than on their own open rate. I would propose that
change alongside the design, because the design fails without it."

**3. It designs the measurement to survive two years.** "Per-launch A/B tests
will each show a small win, and the sum will not appear. I would launch a
permanent 0.5 percent no-notification holdout on day one, accept that it gives
no answer for a quarter, and report cumulative effect against it quarterly. That
is the only number that can settle whether this product is net positive."

**4. It leaves a reversible path.** "Migration order: shadow the value model
against the current policy for two weeks, then run it as an advisory reserve on
1 percent, then enforce. Kill switch reverts to the current fixed threshold and
the cap of 5, and it stays in place for two quarters. Blast radius of a bad
value model is bounded by the daily cap, which is the reason I would keep a hard
cap even after the budget is live."

!!! tip "Self-explanation prompt"

    Say which of these four additions you could make in a 45 minute round without
    running out of time, and which one you would name in a single sentence and
    move on from.

**The error most readers make here.** Delivering item 2 first. Organisational
framing before a design is on the board reads as avoidance, and interviewers
have heard it from candidates who could not do item 1.

#### Block D. Purpose: the same follow-up, answered at three levels

Follow-up: "You ship it and open rate is up 6 percent, but daily active users
are flat. What now?"

| Level | Answer | What the interviewer hears |
|---|---|---|
| Mid | "I would check whether the test had enough power on daily actives, and look at the metric by segment" | Correct diagnostics, no hypothesis, no ranking of causes |
| Senior | "Open rate is the model's own objective, so a rise is nearly guaranteed. Flat daily actives is the real result. I would check power on daily actives first, then whether opens shifted toward low-value notification types, then whether the effect decays across the fortnight, which would indicate novelty" | A ranked set of hypotheses, each with a distinguishing test |
| Staff | "The result is that we moved a proxy and not the goal, which is exactly what the holdout was built to detect. I would report it that way. Then the decision is whether to keep optimising the proxy: I would take the marginal-value derivation to the review, show that the top notification types are near break-even, and propose reducing volume rather than increasing it. If the answer is that we cannot reduce volume for organisational reasons, that is the finding, and it belongs in the readout" | Reframes the result, makes a recommendation against the team's incentive, and names the constraint honestly |

**The error most readers make here.** At every level, treating "flat" as "no
effect". Compute the minimum detectable effect first, as in Module 11.

#### Block E. Purpose: signal one level up without over-reaching

Three moves that are cheap, safe and read one level higher than they cost:

| Move | Sentence |
|---|---|
| Convert an inherited constant into a measurement | "The cap of 5 is a constant. I would derive it from the marginal effect of the nth send" |
| Name what you are not building and why | "No streaming trainer. Daily fits the cadence, and here is the arithmetic" |
| State the measurement that would change your mind | "If the marginal fifth notification is still net positive, my whole volume argument is wrong and I would keep the cap" |

And one move to avoid: do not question the prompt in your first two minutes
unless you can then design under your own reframing. An interviewer who hears
the reframe and no design has learned only that you can reframe.

**The error most readers make here.** Deploying all three moves in the first
five minutes. Delivered together and early they read as a rehearsed opening.
Delivered one each at the point where the design actually forces the choice,
they read as judgement.

### 16e. Fade the scaffold

**Fade 1: "design fraud detection for a payments product."** Mid and senior
blocks given, staff block blank.

- Mid: a supervised classifier on transaction and account features, threshold
  chosen for precision, online serving inside the payment path, weekly retrain,
  precision and recall monitored.
- Senior: adds that labels arrive as chargebacks weeks later, so the training
  set is systematically incomplete; that blocked transactions produce no labels
  at all, so a random-release slice is required; that the threshold must be set
  by review capacity rather than by score value; and that the latency budget
  inside checkout forces a small in-process model with a degraded path when a
  feature lookup is slow.
- Staff: **you write this block.**

??? note "Show answer"

    Three scope changes, and one currency.

    First, put the decision in money. The system is not a classifier, it is a
    decision that trades fraud loss against declined-good-customer loss. State
    both in the same units: expected loss equals probability of fraud times
    amount, against the expected value of a declined legitimate customer, which
    needs an assumed lifetime figure. The threshold then falls out of the
    arithmetic and the review capacity constraint becomes an explicit budget
    rather than an implicit one.

    Second, name the adversary as a system property. Any static evaluation
    decays because attackers respond to enforcement, so measurement must include
    a decay rate, and the roadmap must include a mechanism for fast rule
    deployment that does not require a model retrain.

    Third, design for the inheriting team: a registry with owners, a kill switch
    that reverts to rules, a random-release slice that cannot be quietly turned
    off during an incident, and a documented statement that the slice is the only
    source of unbiased labels in the whole system.

**Fade 2: "design ranking for a marketplace search page."** Mid block given,
senior and staff blank.

- Mid: two-stage retrieval and ranking, a gradient boosted ranker on query,
  item and user features, NDCG offline, click rate online, daily retrain, cached
  item features.
- Senior: **you write this block.**
- Staff: **you write this block.**

??? note "Show answer"

    Senior. Names the multi-objective problem: buyers want relevance, sellers
    want exposure, the marketplace wants completed transactions, and clicks
    proxy none of these well. Proposes a combined objective with explicit
    weights and says that the weights are a product decision that must be
    written down. Derives the latency budget and shows the retrieval fan-out
    tail arithmetic.

    Names position bias, since the training data is generated by the incumbent
    ranker, and prices the inverse propensity correction with an effective
    sample size calculation. States what it will not build: no per-seller model,
    no real-time index, and gives the thresholds at which it would revisit.

    Staff. Notices that user-level experiments are invalid because supply is
    shared: promoting one seller's items demotes another's, so treatment and
    control interfere and a user-split test measures a transfer as well as a
    creation. Proposes a switchback or a market-level design and states the
    sample size consequence, which is that millions of sessions collapse to a
    few hundred independent units.

    Then names the feedback loop: exposure creates sales, sales create ranking
    signal, so the ranker manufactures the concentration it then learns from,
    and proposes measuring impression share of the top 1 percent of sellers over
    time as the instrument. Finally, sequences it under a real constraint and
    states which part is deliberately deferred.

### 16f. Check yourself

**Q1.** You are interviewing for a senior role and you have 40 minutes. You
notice the prompt's stated objective is the wrong thing to optimise. What
exactly do you do, and in what order?

??? note "Show answer"

    Say it once, early, in one sentence, and then design under a stated
    assumption. For example: "Open rate will reward clickbait, so I will design
    against a proxy for open rate and treat downstream session value as the real
    objective. If you would rather I optimise open rate directly, say so and I
    will adjust." That takes fifteen seconds and buys the whole frame.

    Then design. Do not spend five minutes negotiating the objective, because at
    senior level the rating comes from deriving a design under constraints, and a
    candidate who spends a quarter of the round on framing has shown one
    behaviour and skipped six. Return to the objective once more at the end, when
    you name what you would measure first, which is where the reframing pays off
    without costing time.

**Q2.** An interviewer says "that is over-engineered" about your design. Give the
mid, senior and staff response, and say which one is a trap.

??? note "Show answer"

    Mid response: defend the design by restating its benefits. This is the trap.
    Defending rather than updating is the single most reliably down-levelling
    behaviour available, because it shows the design was a position rather than a
    derivation.

    Senior response: name the measurement that would decide it. "You may be
    right. The component is justified only if the repeat rate on that cache key
    is above about 30 percent. I assumed it was; if it is 5 percent I would
    remove the cache." That converts a disagreement into a testable claim and
    concedes gracefully without collapsing.

    Staff response: remove the box, out loud, and say what you gain. "Agreed, I
    will take it out. That removes a stampede risk and a staleness bug, and it
    costs us latency only if the repeat rate is high, which I would measure
    before adding it back." Removing a box on the interviewer's cue, and stating
    what the removal buys, is one of the few moments in these rounds where the
    rating goes up rather than merely staying flat.

### 16g. When not to use this

**Answering at a level above the one you are interviewing for.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You can complete the level below in the time available. Staff framing on top of an incomplete senior design consistently scores lower than a complete senior design |
| Scale threshold below which it is over-engineering | In a 45 minute round with a broad prompt, there is rarely time for organisational framing and a full design. Pick the design |
| Cheaper alternative | One sentence of higher-level framing, delivered after the design is on the board, in the last five minutes |

**Level deltas as a study device.**

| Question | Answer |
|---|---|
| Measurement that justifies it | You have already produced complete answers at your own level and want to know what the next one adds |
| Scale threshold | If you cannot yet produce the mid answer inside the time limit, reading staff answers is entertainment. Practise the mid answer against a clock instead |
| Cheaper alternative | Record yourself answering one prompt in 20 minutes, then check it against the eight behaviours table and fix the two weakest |

**The marginal-value derivation shown in Block C.**

| Question | Answer |
|---|---|
| Measurement that justifies it | The system takes a repeated action with a per-action cost to the user, and you can name both the gain and the cost in one currency |
| Scale threshold | For a system with a single action per session and no fatigue effect, this machinery is elaborate framing for a threshold. Set the threshold from capacity and move on |
| Cheaper alternative | A measured cap derived from the marginal effect of the nth action, which is one experiment and no economics |

## Case study 1: Video recommendation

Two assumptions run through every case study on this page, because they turn latency and cost claims into arithmetic you can redo with your own numbers.

| Assumption | Value | Why it is reasonable |
|---|---|---|
| Heavy ranker throughput | 200,000 item scorings per second per inference accelerator | A few-hundred-feature network with a batched forward pass over hundreds of candidates. Treat it as a placeholder: measure yours on day one, then re-run every division below |
| Accelerator price | 2 dollars per hour | An order-of-magnitude anchor for a rented inference accelerator. Costs move; the ratios in these case studies do not |

Say both out loud in the room. A number labelled as a placeholder that you offer to re-measure reads as engineering. The same number stated as fact reads as recall.

### The prompt (video recommendation)

"Design the system that decides which videos to show on the home screen of a video app."

### Questions that fork the design (video recommendation)

| Question worth asking | If the answer is A the design becomes | If the answer is B the design becomes |
|---|---|---|
| Home screen, or the next-video panel beside a playing video? | Home: there is no strong context signal, so personalisation carries the whole load and diversity has to be engineered in | Next-video: the currently playing item is a powerful context feature, item to item similarity dominates retrieval, and the interesting risk is a topic rabbit hole |
| What is the objective the business will grade you on? | Watch time: one regression head, dense labels, and a fast feedback loop | Stated satisfaction from surveys: sparse labels on a small sample, so you need a satisfaction head trained on thousands of rows and a combiner that mixes it with dense engagement heads |
| How big is the eligible corpus, and how much of it is new? | Ten million items, stable: exploration is barely a problem because every item accumulates impressions | Two hundred million items with a million new per day: the cold start budget becomes the design's hardest constraint, which is deep dive one |
| Is there a subscription or follow graph? | Yes: a cheap high precision retrieval source that costs one range read and needs no model | No: every candidate comes from a learned embedding, and a bad embedding has no fallback |
| Are creators paid based on views? | No: exploration is a learning device you can tune freely | Yes: exploration is also a supply side obligation, and starving new creators is a business failure before it is a model failure |
| Can the page be precomputed, or must it be built per request? | Precomputed nightly: retrieval and ranking can be arbitrarily heavy, at the cost of stale results | Per request: you get a hard latency budget that has to be split across stages, which is what V1 does |
| Do you serve logged out or brand new users? | No: every request has a user embedding | Yes: you need a context only path (locale, device, entry point) and the cold start path becomes a first class product surface, not an edge case |
| Who owns eligibility, meaning age gates, regional blocks and takedowns? | Ranking owns it: eligibility becomes a model feature and mistakes are silent | A separate policy service owns it: it becomes a hard filter, ranking never sees ineligible items, and the filter placement decides your recall, which is deep dive two |

### Requirements (video recommendation)

Functional:

- Return a page of 20 ranked videos for the home screen of a signed in user.
- Exclude items the user has already watched, and items the policy service marks ineligible for that user.
- Give newly published videos a path to their first impressions.
- Log every impression with its position, its retrieval source and the probability it was selected with.
- Refresh the page on a pull, without repeating what the user saw on the previous pull of the same session.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Daily active users | 100 million | ASSUMED; the interviewer normally gives this, so ask before assuming |
| Home screen opens per user per day | 4 | ASSUMED; a conservative figure for an app opened casually |
| Items rendered per open | 20 | GIVEN by the product surface |
| Eligible corpus | 200 million videos | ASSUMED |
| New videos published per day | 1 million | ASSUMED, and this is the number that sizes the exploration budget |
| Server side latency for a home request | under 300 ms at p99 | ASSUMED; the home screen is the first paint of the app |
| Ranking model retrain cadence | daily | ASSUMED; justified in the failure section by how fast engagement distributions move |
| Peak home requests per second | 13,900 | DERIVED below |
| Impressions available per day | 8 billion | DERIVED below |

### Estimation that eliminates (video recommendation)

| Calculation | Result | What it rules out |
|---|---|---|
| 100e6 users times 4 opens divided by 86,400 | 4,630 requests per second, 13,900 at 3x peak | Rules out nothing yet, but it is the multiplier for every line below |
| 13,900 requests per second times 200e6 corpus items | 2.78e12 item scorings per second | Rules out scoring the corpus per request, by nine orders of magnitude. This single line is why retrieval and ranking are separate stages, and it is worth writing on the board |
| 4e8 requests per day times 1,000 candidates, divided by 1.728e10 scorings per accelerator day | 23 accelerators steady, 70 provisioned for 3x peak | Rules out ranking 10,000 candidates per request, which would need 700. The candidate count is a cost dial, not a taste decision |
| 70 accelerators times 2 dollars times 24 hours, divided by 400,000 thousand-requests | 0.0084 dollars per thousand home requests | Rules out inference cost as the thing to optimise first. At under a cent per thousand requests, engineering time belongs on quality, not on shaving the ranker |
| 200e6 vectors times 128 dimensions times 4 bytes | 102 GB in fp32 | Rules out holding the retrieval index in memory on one 64 GB node at full precision |
| The same vectors quantised to one byte per dimension | 25.6 GB | Rules out sharding the index for capacity reasons alone. One node holds it, so any sharding you propose must be justified by query throughput instead |
| 100e6 users times 4 opens times 20 slots | 8e9 impressions per day | Rules out nothing alone, but it is the denominator for the exploration budget in deep dive one |
| 1e6 new videos times 5,400 impressions each to resolve a 1 point click rate difference | 5.4e9 impressions, which is 67 percent of all daily impressions | Rules out uniform exploration over new items, decisively. The whole of deep dive one follows from this number |

The sample size of 5,400 is derived in deep dive one rather than asserted here, because it needs two lines of arithmetic and one stated confidence level.

### V0, the simplest thing that works (video recommendation)

**Predict first: at what daily active user count does a nightly precomputed home screen stop being the right answer?**

??? note "Show answer"

    It stops when either freshness or coverage breaks, not when scale breaks. Precompute cost is linear in users, and a nightly batch that scores 1,000 candidates for 100 million users is 1e11 scorings, about 6 accelerator days of work, which is affordable. What kills it is that a video published at 9am cannot appear until tomorrow, and that a user who watches something at 9am gets the same list all day. If your corpus turns over slowly and sessions are short, precompute is genuinely correct at any user count.

```
+--------+     +------------------+     +------------------+
| Client | --> | Home service     | --> | Precomputed rows |
+--------+     | read one row     |     | user id -> ids   |
               +------------------+     +------------------+
                                                 ^
                                        +-----------------+
                                        | Nightly batch   |
                                        | score and write |
                                        +-----------------+
```

**Caption.** V0 has a nightly batch job that scores a candidate set per user and writes one row of video identifiers per user; a home request is a single key lookup plus metadata hydration, with no model in the request path at all.

Why this is correct at a real company's scale:

- No online inference means no latency budget to split and no accelerator fleet to operate.
- The batch job can use a model that is far too slow to serve, which often beats a fast online model on quality.
- Debugging is trivial: the recommendation for any user is a row you can read.
- Retrieval and ranking still exist, they just both run offline, so the concepts transfer when you move online.

!!! warning "When not to move off nightly precompute"

    Below roughly one million daily active users, or when the corpus turns over slower than about one percent per day, precompute is the better answer and moving online is over-engineering. The measurement that justifies moving is the gap between offline scored quality and the quality of what users actually saw, computed by replaying the precomputed row against the state of the world at click time. If that gap is under a few percent, the online path buys nothing.

### Breaking point, then V1 (video recommendation)

**What fails: intra-session staleness.** A user watches three cooking videos at 9am and the home screen still shows the list computed at 3am. How you observe it: bucket sessions by position within the day, then plot engagement rate against hours since the batch ran. A downward slope that is steep in the first hour and flat afterwards is a context signal problem. A slope that is flat all day is not, and means you should leave V0 alone.

The second failure is coverage. A video published at 10am gets zero impressions until the next batch, so the median time to first impression is about 12 hours. With a million new videos per day, that is a permanent 12 hour handicap on every new item.

V1 puts a staged pipeline in the request path:

```
+--------+    +--------------+    +-----------------+
| Client |--> | Home service |--> | Retrieval, 4    |
+--------+    +--------------+    | sources, 250    |
                                  | candidates each |
                                  +-----------------+
                                           |
                                           v
+--------------+  +---------------+  +-----------------+
| Re-rank and  |<-| Ranker over   |<-| Filter, dedup,  |
| explore mix  |  | 1000, batched |  | policy, seen    |
+--------------+  +---------------+  +-----------------+
       |                  ^
       v                  |
+--------------+  +---------------+
| 20 item page |  | Feature store |
+--------------+  +---------------+
```

**Caption.** V1 fans out to four retrieval sources in parallel for about 250 candidates each, removes ineligible, seen and duplicate items, scores the surviving 1,000 in one batched forward pass using features read from a feature store, then re-ranks for diversity and mixes in an exploration lane before returning 20 items.

The latency budget, which is where the design becomes real:

| Stage | Budget | What it does |
|---|---|---|
| Parse, auth, user feature read | 20 ms | One key read for the user's recent history and embedding |
| Retrieval, four sources in parallel | 30 ms | Bounded by the slowest source, not the sum |
| Filter, dedup, policy check | 10 ms | Set operations plus one batched policy call |
| Feature hydration for 1,000 candidates | 40 ms | One batched read of item side features |
| Ranker forward pass | 80 ms | One batch of 1,000 |
| Re-rank, diversity, exploration mix | 15 ms | Pure CPU over 1,000 scored items |
| Response build and metadata hydration | 25 ms | 20 items only |
| **Sum** | **220 ms** | Leaves 80 ms of headroom against the 300 ms p99 objective |

The headroom is the point. A budget that sums exactly to the objective has no room for a slow feature store call, and p99 is where slow calls live.

!!! warning "When not to add the online ranker"

    If the slope of engagement against hours since batch is under about five percent across the day, the online ranker is buying a number you cannot measure. It costs 70 accelerators, a feature store with an offline and online consistency problem, and a permanent latency budget. The cheaper thing is a partial refresh: recompute rows only for users who were active in the last hour, which is a fraction of the fleet and fixes most of the staleness with none of the serving complexity.

### Breaking point, then V2 (video recommendation)

**What fails: new items never accumulate the impressions they need to be rankable.** The ranker's strongest features are the item's own historical engagement rates. A video with zero impressions has no such features, so it scores near the prior, so it stays below the 20 item cut, so it keeps zero impressions. That loop is self sealing and it does not resolve on its own.

How you observe it: plot the share of daily impressions going to items first published in the last seven days against the share of the eligible corpus those items represent. With 1 million new items per day and 200 million eligible, seven days of new items are 3.5 percent of the corpus. If they receive far under 3.5 percent of impressions, you have a starvation problem rather than a quality signal.

V2 adds a dedicated exploration lane and a cold start scorer, and it changes the logging contract:

| Element | What it adds | Cost |
|---|---|---|
| Cold start scorer | Predicts engagement for a zero impression item from creator history, topic, title and thumbnail embeddings | One extra small model, run on new items in batch, not per request |
| Exploration lane | A reserved share of the 20 slots filled by sampling from cold start candidates | Priced in deep dive one at about 2 percent of watch time |
| Propensity logging | Every impression records the probability with which that item was selected | One float per impression, 8e9 per day, and it is unrecoverable if you do not log it at serving time |

!!! warning "When not to build an exploration lane"

    If your corpus turns over under about one percent per week, and new item engagement rates are within noise of established items, skip it. Organic traffic will resolve every item eventually. The measurement that justifies the lane is the impression share ratio above, combined with a measured lift for items that were exposed by accident (for example items that briefly rode a trending source), which tells you what you are leaving on the table.

### Deep dive one: the exploration budget, priced in watch time

Exploration is usually discussed as a philosophy. It is a budget, and the budget is derivable.

**Step one: how many impressions does one new item need?** To distinguish a click rate of 3.5 percent from 4.5 percent at 95 percent confidence and 80 percent power, the standard two proportion sample size is approximately:

```
n = 16 * p * (1 - p) / d^2
```

The 16 comes from 2 times (1.96 + 0.84) squared, which is 15.7 rounded. With p = 0.035 (ASSUMED base click rate) and d = 0.01:

- n = 16 times 0.035 times 0.965 divided by 0.0001
- n = 0.5404 divided by 0.0001
- n = 5,404, call it 5,400 impressions per item

**Step two: what does exploring everything cost?** 1 million new items per day times 5,400 impressions is 5.4e9 impressions. You have 8e9 per day. Uniform exploration over new items consumes 67 percent of your entire inventory. That is not a budget, it is the product.

**Step three: gate the candidates, not the budget.** The cold start scorer ranks the day's new uploads before any of them is shown. Admit the top 5 percent:

| Quantity | Value | How it follows |
|---|---|---|
| New items admitted per day | 50,000 | 5 percent of 1 million |
| Impressions needed | 2.7e8 | 50,000 times 5,400 |
| Share of daily inventory | 3.4 percent | 2.7e8 divided by 8e9 |
| Time for an admitted item to resolve | under 24 hours | The budget is sized to exactly cover one day's admitted items |

**Step four: price it.** Assume exploration slots earn 40 percent of the engagement of exploited slots. That number is not a guess you carry in; it is directly measurable as the ratio of engagement on the exploration lane to the exploitation lane, and you will have it within a day of launching. The cost is 3.4 percent of impressions times a 60 percent shortfall, which is about 2.0 percent of total watch time, permanently.

Saying "we spend two percent of watch time on exploration, here is what it buys" is a different conversation from "we should explore more".

**Step five: choose the mechanism, and notice this is not a classic bandit.**

| Mechanism | What it needs | Where it fits here | Where it does not |
|---|---|---|---|
| Epsilon greedy | One constant | Fine as a first implementation, and it makes the budget explicit by construction | Spends the same on an item it already resolved as on a fresh one |
| Upper confidence bound | A count and a mean per item | Cheap, deterministic, easy to reason about in an incident | Needs a tuned exploration constant per surface, and behaves badly when reward scales differ across item types |
| Thompson sampling | A posterior per item | Naturally spends less as evidence accumulates, and composes with the cold start prior as an informative prior | You now maintain a posterior for 200 million items, and the update path is a second write path to operate |
| Contextual bandit over a slate | A policy plus logged propensities | Matches the actual problem, because a slate returns feedback on all 20 positions rather than one arm | Position bias makes the 20 observations non exchangeable, so you need a position correction before the feedback is usable |

The last row is the senior point. A home screen is not a one armed decision. You observe outcomes for every slot you filled, which is far more information than a bandit formulation assumes, and the price of that information is that slot 1 and slot 20 are not comparable until you correct for position.

**Step six: log propensities or lose the ability to evaluate.** Off policy estimation, meaning "what would this new policy have earned on last week's traffic", needs the probability that the logged policy chose the item it chose. That float exists only at serving time. If you skip it, the only way to compare policies is a live experiment, which costs weeks per comparison. Eight billion floats per day at 4 bytes is 32 GB per day, which is a rounding error against your impression logs.

!!! warning "When not to run a bandit"

    Below roughly 100,000 items with under one percent weekly turnover, every item accumulates 5,400 impressions on its own within days, and a bandit adds a posterior store, a propensity logging contract and an off policy evaluation pipeline to buy nothing. The measurement that justifies it is the seven day impression share ratio described in the V2 breaking point. Until that ratio is meaningfully below one, run epsilon greedy with a fixed small epsilon and spend the effort elsewhere.

### Deep dive two: recall accounting across the retrieval and ranking split

The split saved you nine orders of magnitude. The bill arrives as a question nobody asks until quality stalls: how much of the ranker's potential is the retrieval stage throwing away before the ranker ever sees it?

**Define the metric precisely.** Retrieval recall at k is the fraction of the items the ranker would have placed in its top k, had it scored the whole corpus, that actually appeared in the retrieved candidate set. It is a property of the pair (retrieval stage, current ranker), so it changes every time you retrain the ranker, which is the part teams miss.

**Measure it without scoring 200 million items per request.** Sample the traffic and sample the corpus:

| Step | Quantity | Cost |
|---|---|---|
| Sample requests per day | 1,000 | Free |
| Score a uniform random corpus sample per sampled request | 1e6 items | 1e9 scorings per day |
| Accelerator time at 200,000 scorings per second | 5,000 seconds | 1.4 accelerator hours per day |
| Overhead against a 70 accelerator serving fleet | 1,680 accelerator hours per day | 0.08 percent |

Eight hundredths of one percent. There is no version of this where you cannot afford to measure it. Note the honest caveat: recall measured against a uniform corpus sample is optimistic, because the hardest misses are correlated with the items your retrieval already favours. Report it as a trend line, not as an absolute.

**The four sources, and what each one uniquely covers.**

| Source | How it is built | Quota | What it uniquely covers | What breaks if you delete it |
|---|---|---|---|---|
| Two tower approximate nearest neighbour | User tower embedding queried against the item index | 250 | Long tail items that match latent taste with no co-watch history | Recall collapses for users with narrow histories |
| Subscription and follow | One range read over the user's follow edges, most recent first | 250 | Items the user explicitly asked for, at high precision and near zero cost | The product feels broken in a way no metric names, because users notice missing subscriptions personally |
| Co-watch item to item | Precomputed neighbours of the user's recent watches | 250 | Session continuity, the "more like the thing I just watched" intent | Sessions get shorter, and the effect is concentrated in the first minute |
| Fresh and trending | Time windowed index over items published or accelerating in the last 24 hours | 250 | Anything published today, which no history based source can reach | New item impression share falls to near zero and the V2 loop reasserts itself |

The quotas are the design. Fixed quotas are predictable, debuggable and slightly wasteful. Learned quotas earn a few percent and cost you the ability to answer "why did this item not appear" in an incident. Start fixed; move only one quota at a time, and only when the recall metric above says that source is starved.

**Where the filter goes, and why it is not a detail.** Three placements, three different failure shapes:

| Placement | Mechanism | Cost | Failure |
|---|---|---|---|
| After retrieval | Retrieve 1,000, drop ineligible and seen, rank what remains | Simplest, one code path | A heavy user with 5,000 watched items loses a large slice of every source, and the loss is invisible because the response still has 20 items |
| Over-fetch then filter | Retrieve 1,000 divided by (1 minus expected drop rate) | If 30 percent of candidates are dropped, retrieve 1,430 instead of 1,000, so 43 percent more retrieval work for the same output | Correct on average, wrong for the users at the tail of the drop rate distribution, which is exactly your heaviest users |
| Filter during traversal | The index evaluates the eligibility predicate while walking candidates | No wasted retrieval, correct for every user | Couples the index to the policy service, and a slow predicate turns a 30 ms stage into a 300 ms one |

The measurement that decides it is the distribution of drop rate by user, not its mean. Plot drop rate at p50 and p99. If p99 drop rate is under about 20 percent, over-fetch and move on. If it is over 50 percent, your heaviest users are being served from a quarter of the candidates everyone else gets, and no amount of ranker improvement will show up for them.

**Seen suppression is a different problem from eligibility.** Eligibility is a correctness constraint and must be exact, because showing a blocked item is a policy incident. Already watched is a quality preference where a rare mistake costs one repeated video. That asymmetry is the entire design: eligibility gets an exact check against the policy service, seen suppression gets an approximate structure sized for memory rather than exactness.

!!! warning "When not to build multi-source retrieval"

    With one retrieval source you have no blending logic, no quota tuning and one recall number. Add a second source only when the recall measurement shows the first one systematically missing a nameable category (new items, subscriptions, session continuity). Adding sources because more candidates sounds better raises retrieval latency, raises the drop rate at the filter, and gives the ranker more near duplicates to break ties between.

### Failure modes and evaluation (video recommendation)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Feedback loop and popularity collapse | The ranker trains on impressions it generated, so items it favoured yesterday have richer features today, and diversity narrows week over week without any single bad deploy | The exploration lane is the structural fix. The detector is a Gini coefficient over daily impressions per item, tracked as a first class metric, not a report |
| Training and serving skew | An offline feature computed over a full day's log differs from the online feature computed from a partial day, so offline metrics improve and online metrics do not move | Compute both from the same feature store code path, and run a daily job that samples live requests, recomputes features offline, and alerts on any feature whose distribution differs beyond a threshold |
| Stale index hot swap | The nearest neighbour index is rebuilt and swapped while a request is in flight, returning identifiers that no longer resolve | Version the index, hydrate against the version that answered, and fail the individual candidate rather than the request |
| Cold start starvation | Covered above, but it also arrives as a creator complaint before any dashboard shows it | The seven day impression share ratio, alerted on, not merely charted |
| Thundering herd on a viral item | One item is hydrated 13,900 times per second from one metadata shard | A short lived local cache tier on item metadata, which is a request path problem rather than a model problem |
| Silent model rollback gap | A rollback restores the old model but not the old feature definitions, so the restored model runs on features it never saw in training | Version the model and its feature schema as one artefact, and refuse to load a mismatched pair |
| Position bias in the training labels | The model learns that position 1 items are good, because position 1 items get clicked | Log position, include it as a feature at training time and set it to a constant at serving time, or model it in a separate shallow tower |

Service level indicators:

- Home request latency at p50 and p99, segmented by whether the exploration lane fired, because the exploration path has a different candidate mix and can hide a regression.
- Retrieval recall at 100 against the current ranker, computed daily from the 1,000 request sample.
- Impression share of items published in the last seven days, against their corpus share.
- Gini coefficient over per item daily impressions.
- Feature freshness lag at p99, measured as the age of the newest event reflected in a served feature.
- Empty or short page rate, which catches retrieval failures that latency metrics never see.

Alert on retrieval recall dropping more than a stated amount day over day (it moves whenever the ranker retrains, so alert on the delta, not the level), on feature freshness lag crossing the window the model was trained with, and on empty page rate deviating from baseline.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| p99 under 300 ms | Yes: the stage budget sums to 220 ms with 80 ms of headroom |
| 13,900 requests per second at peak | Yes, at 70 accelerators, and it scales linearly with candidate count |
| New videos reach their first impressions | Yes for the 5 percent the cold start scorer admits. The other 95 percent are explicitly deprioritised, which is a product decision you should name out loud rather than hide |
| Exclude seen and ineligible | Eligibility yes, exactly. Seen suppression approximately, and the asymmetry is deliberate |
| Log propensity per impression | Yes, at 32 GB per day |
| Daily retrain | Yes, and the retrieval index rebuild is weekly, which is a separate cadence with its own staleness cost |

### Where this answer is a simplification (video recommendation)

All links in this list were checked on 1 August 2026.

- **Covington, Adams and Sargin, ["Deep Neural Networks for YouTube Recommendations"](https://research.google/pubs/deep-neural-networks-for-youtube-recommendations/) (RecSys, 2016)** is the primary published description of the candidate generation plus ranking split used here, including the argument for training on all watches rather than only recommended watches.
- **Chen and colleagues, "Top-K Off-Policy Correction for a REINFORCE Recommender System" (WSDM, 2019)** is the clearest primary treatment of logged propensities and off policy correction for slate recommenders, which is the machinery the propensity logging above exists to enable.
- **Steck, "Calibrated Recommendations" (RecSys, 2018)** argues that a recommender should preserve the proportions of a user's interests rather than collapse onto their strongest one, which is the re-rank stage this case study treats too briefly.
- **Chandrashekar, Amat, Basilico and Jebara, ["Artwork Personalization at Netflix"](https://netflixtechblog.com/artwork-personalization-c589f074ad76) (Netflix Technology Blog, 7 December 2017)** is a readable primary account of running contextual bandits in production, including the offline replay evaluation problem.
- **McInerney and colleagues, "Explore, Exploit, and Explain" (RecSys, 2018)** connects bandit exploration to explanation, which matters because unexplained exploration reads to users as a broken product.
- **The real objective is not watch time.** Every serious version of this system predicts several outcomes and combines them, which is case study 2 on this page. Treating watch time as the target is a teaching simplification that a good interviewer will challenge within five minutes.
- **Multi-surface reality.** Home, next-video, search, subscriptions and notifications all want candidates. Building five retrieval stacks is the actual architectural problem inside a large company, and the answer is usually a shared candidate service with per surface quotas.

### Level delta (video recommendation)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Draws retrieval, ranking and re-rank immediately | Starts at nightly precompute and names what breaks | Asks whether this is the home screen or the next-video panel before drawing anything, because the answer decides whether personalisation or context carries the load |
| The split | "You cannot score everything, so you retrieve first" | Computes 2.78e12 scorings per second to show why | Turns candidate count into a cost dial (1,000 candidates is 70 accelerators, 10,000 is 700) and offers to trade it against measured recall |
| Exploration | "Add some randomness for new items" | Reserves a share of slots and explains cold start | Derives 5,400 impressions per item, shows uniform exploration would cost 67 percent of inventory, gates to the top 5 percent, and prices the whole thing at 2 percent of watch time |
| Evaluation of retrieval | Not raised | Mentions recall at k | Defines recall against the current ranker, notes it moves on every retrain so the alert is on the delta, and prices the measurement at 0.08 percent of the fleet |
| Filtering | Filters after retrieval | Over-fetches to compensate | Asks for the drop rate distribution rather than its mean, and separates exact eligibility from approximate seen suppression on the grounds that the cost of a mistake differs |
| Logging | Logs clicks | Logs impressions with position | Logs the selection propensity, and states that it is one float that is unrecoverable after the fact and worth 32 GB per day |
| Honesty | Claims the design covers cold start | Notes some new items wait | Says out loud that 95 percent of new uploads are deliberately deprioritised, and frames it as a product decision needing a product owner |

What each level added:

- **Mid to senior:** the architecture stops being a remembered pipeline and starts being derived from two divisions, and the first design offered is the small one.
- **Senior to staff:** every knob becomes a priced lever with a measurement attached, the measurement itself gets costed to prove it is affordable, and the weakest part of the design is volunteered before the interviewer finds it.

---

## Case study 2: Social feed ranking

Case study 1 optimised one outcome. This prompt has six, they disagree, and one of them is negative. The interesting work is not the model, it is deciding what the number the model produces is worth.

### The prompt (feed ranking)

"Design the ranking system for a social feed, where a user opening the app sees a mix of posts from accounts they follow, groups they belong to, and recommended content."

### Questions that fork the design (feed ranking)

| Question worth asking | If the answer is A the design becomes | If the answer is B the design becomes |
|---|---|---|
| Who owns the objective: is there a single stated goal, or a set? | Single goal such as time spent: one head, and the whole design is case study 1 again | A set including satisfaction, integrity and creator health: you need a value model, an exchange rate between heads, and a governance process for the weights, which is deep dive one |
| Where does the candidate inventory come from? | Follow graph only: inventory is bounded by who you follow, and a thin day means a thin feed | Follow graph plus recommendations: inventory is effectively unbounded, ranking quality dominates, and the follow graph becomes one retrieval source among several |
| Is integrity a ranking signal or a separate system? | A signal: policy scores go in as features, ranking learns what to do with them | A separate system: integrity produces actions (demote, remove, no action) that ranking must obey, which is auditable, changeable without retraining, and deep dive two |
| Must the ordering be stable across pages within a session? | No: each page is ranked fresh, which is cheap and occasionally shows a duplicate | Yes: you materialise a session ordering, which is a delivery problem covered in the news feed case study of the [Modern System Design course](modern-system-design.md), not a ranking problem |
| How fast must a policy change take effect? | Next retrain, so a day or more: policy can live in the model | Minutes: policy has to be a served rule outside the model, because you cannot retrain to fix a live harm |
| Does a demotion need to be explainable to the affected author? | No: any monotone combination will do | Yes: you need per post attribution of which rule fired, which forces integrity out of the value model and into a rule layer |
| How many prediction targets do you have labels for? | Two or three dense engagement events | Six, including rare ones such as report and hide: the rare heads have thousands of positives per day rather than millions, and they need loss weighting or they contribute nothing |
| Is there a regulatory transparency obligation? | No | Yes: the ranking inputs must be describable, which rules out designs where the only answer to "why did I see this" is a model output |

### Requirements (feed ranking)

Functional:

- Return a page of 20 posts drawn from follow, group and recommendation inventory.
- Predict several outcomes per candidate and combine them into one ordering.
- Apply integrity actions produced by a separate policy system, before the ordering is returned.
- Guarantee that a post the policy system has removed cannot be served, at any latency.
- Log every impression with position, predicted values per head, and the integrity actions that applied.
- Support changing the combination weights without retraining the model.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Daily active users | 180 million | ASSUMED |
| Sessions per user per day | 7 | ASSUMED; feed products are opened more often and for shorter periods than video products |
| Posts rendered per session | 20 | GIVEN by the product surface |
| Candidates surviving retrieval | 3,000 | ASSUMED; unseen posts from follows, groups and recommendations within a 7 day window |
| Server side latency for a feed request | under 250 ms at p99 | ASSUMED |
| Removal action propagation | under 60 seconds from decision to unservable | ASSUMED, and it is the requirement that decides where the integrity check sits |
| Peak feed requests per second | 43,800 | DERIVED below |
| Impressions per day | 25.2 billion | DERIVED below |

### Estimation that eliminates (feed ranking)

| Calculation | Result | What it rules out |
|---|---|---|
| 180e6 users times 7 sessions divided by 86,400 | 14,600 requests per second, 43,800 at 3x peak | The multiplier for everything below |
| 43,800 requests per second times 3,000 candidates | 1.314e8 item scorings per second | Rules out one heavy model over the full candidate set: at 200,000 scorings per accelerator second that is 657 accelerators |
| A model 10x cheaper (no sequence tower, no cross features) at 2e6 scorings per second, over 3,000, plus the heavy model over 600 | 66 plus 131, so 197 accelerators | Rules out the single stage ranker. Two stages cost 3.3x less for the same top 600, and this is the number that justifies a pre-ranker rather than a preference for one |
| Six single task models over 600 candidates | 786 accelerators | Rules out one model per objective. A shared trunk with six heads costs 131, because the trunk is the expensive part and the heads are a matrix multiply each |
| 197 accelerators times 2 dollars times 24 hours, divided by 1.26e6 thousand-sessions | 0.0075 dollars per thousand sessions | Rules out cost as the reason to cut a head. Adding a seventh prediction head is nearly free; the expensive part of a head is the label pipeline and the weight negotiation |
| 16 divided by 0.003 squared, for a per user daily metric with a coefficient of variation of 1.0 | 1.78e6 users per arm to detect a 0.3 percent relative change | Rules out reading small metric movements off small experiments, and it sets the minimum exposure for any weight change |
| Six weights at three levels each, as a grid | 729 arms times 1.78e6, which is 1.3e9 users, or 7.2 times your entire user base | Rules out tuning the value model by online grid search. Deep dive one is what you do instead |
| 180e6 users times 5 percent who post times 1.6 posts | 14.4 million posts per day, so about 100 million in a 7 day candidate window | Rules out per post human review: at 720 reviewed items per reviewer day that is 20,000 reviewers |
| 0.4 percent of posts routed to human review | 57,600 per day, so 80 reviewers | Rules out treating the review threshold as a modelling detail. It is a staffing number, and it is set by budget as often as by precision |

### V0, the simplest thing that works (feed ranking)

**Predict first: what does a feed ranked purely by predicted click probability start doing within a few weeks, and which metric shows it first?**

??? note "Show answer"

    It drifts toward content that is cheap to click and expensive to regret: strong headlines, outrage, and anything that rewards a tap without rewarding the time after the tap.

    The first metric to move is not clicks, which go up, and not time spent, which often also goes up. It is the ratio of a negative action to a positive one, such as hides per like, or the share of sessions ending immediately after a click. Any single positive objective can be satisfied by content that is bad along an axis that objective cannot see, which is the whole argument for multiple heads.

```
+--------+   +--------------+   +----------------+
| Client |-->| Feed service |-->| Candidates     |
+--------+   +--------------+   | recent, unseen |
                    |           +----------------+
                    v
             +--------------+   +----------------+
             | One model    |-->| Sort by score  |
             | p(click)     |   | take top 20    |
             +--------------+   +----------------+
```

**Caption.** V0 fetches recent unseen candidates, scores each with one model predicting click probability, sorts, and returns the top 20; there is no pre-ranker, no multi-head combination and no integrity layer.

Why this is correct at a real company's scale:

- One model, one label, one metric. Every regression has one place to look.
- The value model, the weight governance process and the experiment queue that comes with them do not exist yet, and they are organisational costs as much as technical ones.
- With a small community, integrity is handled by people reading reports, which is both cheaper and better than a classifier trained on 40 positive examples.
- A single head that is well calibrated is worth more than six heads that are not, and calibration is easier to get right once.

!!! warning "When not to move off a single objective"

    Below roughly 100,000 daily active users, you cannot power an experiment that detects a 0.3 percent metric change, so you cannot tune weights, so a value model is a set of numbers somebody picked. The measurement that justifies the move is a demonstrated divergence: one head goes up while a guardrail you care about goes down, measured in a real experiment. Until you have seen that divergence with your own data, adding heads adds label pipelines and nothing else.

### Breaking point, then V1 (feed ranking)

**What fails: 657 accelerators, and the fact that the single objective is provably wrong.**

The cost failure is arithmetic: one heavy model over 3,000 candidates at 43,800 requests per second is 1.314e8 scorings per second. How you observe it is trivially, on the invoice, but the design signal arrives earlier as p99 latency climbing linearly with candidate count, which is the signature of an unstaged ranker.

The quality failure is the one from the pre-question. How you observe it: pick one negative outcome you can label cheaply, such as hide, and plot it against your positive metric over successive model launches. If clicks rise 4 percent and hides rise 9 percent across three launches, the objective is selecting for something you did not intend.

V1 splits the ranking stage and the model head:

```
+--------+  +--------------+  +----------------+
| Client |->| Feed service |->| Retrieval      |
+--------+  +--------------+  | 3000 candidates|
                   |          +----------------+
                   v                   |
            +--------------+           v
            | Pre-ranker   |<--+----------------+
            | cheap, 3000  |   | Feature store  |
            +--------------+   +----------------+
                   |
                   v
            +--------------+  +----------------+
            | Heavy ranker |->| Value model    |
            | 600, 6 heads |  | weights served |
            +--------------+  +----------------+
                                       |
                                       v
                              +----------------+
                              | Top 20         |
                              +----------------+
```

**Caption.** V1 narrows 3,000 candidates to 600 with a cheap pre-ranker, scores the 600 with one shared trunk producing six calibrated probabilities, then combines them using weights served from configuration rather than baked into the model, and returns the top 20.

The two changes are independent and worth stating separately:

| Change | What it buys | What it costs |
|---|---|---|
| Pre-ranker stage | 657 accelerators becomes 197 | A second model to train, and a new recall question: how much of the heavy ranker's top 600 does the pre-ranker actually deliver? |
| Six heads on a shared trunk | The ordering can express "this is likely to be clicked and likely to be reported" | Six label pipelines, six calibration curves, and a weight vector that somebody has to own |
| Weights served from config | A weight change ships in minutes and can be experimented on without retraining | A configuration surface that can take the feed down faster than any deploy, so it needs the same review as code |

!!! warning "When not to add a pre-ranker"

    If candidates times request rate is under roughly 1e7 scorings per second, the heavy ranker handles the full candidate set and the pre-ranker is a second model to keep aligned with the first. The measurement that justifies it is ranker cost per thousand sessions crossing a threshold you have agreed with whoever pays. The cheaper alternative, tried first, is cutting the candidate count: if recall against the heavy ranker's top 600 barely moves when you retrieve 1,500 instead of 3,000, you just saved half the cost with no new model.

### Breaking point, then V2 (feed ranking)

**What fails: a policy decision cannot wait for a retrain.**

A category of content becomes harmful on a Tuesday afternoon. Your options in V1 are to retrain the ranker, which is a day, or to hand edit a weight, which is blunt and affects everything. Neither meets the 60 second removal requirement, and neither produces a record of what was done to which post and why.

How you observe it: measure the time from a policy decision being made to the last impression of the affected content. In V1 that number is bounded by your retrain cadence and it will be measured in hours or days.

V2 adds an integrity action layer between the value model and the response:

| Layer | Input | Output | Latency to take effect |
|---|---|---|---|
| Policy service | Post identifier, classifier scores, human review outcomes | An action per post: remove, demote by a factor, no action | Seconds, because it is a served lookup rather than a model weight |
| Hard filter | The remove set | Candidates dropped before ranking | Under 60 seconds via a push updated deny set |
| Score modifier | The demote set | Value multiplied by a per rule factor | Seconds, and the applied rule is logged per impression |

The rule layer is not a fallback for a weak model. It exists because policy changes on a different clock from model training, and because a demotion you cannot attribute to a rule is a demotion you cannot explain, appeal or audit.

!!! warning "When not to build an integrity action layer"

    If your policy set is stable, your volume is low enough that people read reports directly, and you have no obligation to explain a demotion, this layer is a rules engine with three rules in it. The measurement that justifies it is the count of distinct policy changes per quarter and the time each one took to take effect. If policy changes twice a year, put the signal in the model and revisit when that number moves.

### Deep dive one: combining six predictions into one ordering

The model outputs six calibrated probabilities. Ranking needs one number. Everything hard about this system lives in that reduction.

**Start with the shape.** The default is a weighted sum of calibrated probabilities:

```
value = w1*p(click) + w2*p(long dwell) + w3*p(like)
      + w4*p(comment) + w5*p(share) - w6*p(hide or report)
```

Two properties make this the right default and one makes it dangerous:

| Property | Consequence |
|---|---|
| Linear in each probability | You can reason about one weight at a time, and a weight has a unit you can name |
| Negative weight allowed | A post with high click probability and high report probability can rank below a quiet post, which is the entire point |
| Compensatory | A large enough positive term can outweigh any negative term, so a post that will be reported by three percent of viewers can still win on virality. If that is unacceptable, the term belongs in the rule layer, not the value model |

That last row is the cleanest test for whether something is a weight or a rule. If there is any value of the other predictions that should not be allowed to override it, it is a rule.

**Now the hard part: what is a weight?** A weight is an exchange rate. Setting w3 to 8 says one like is worth eight times one unit of whatever w1 is denominated in. Nobody can defend that number in the abstract, so do not try. Two workable methods:

| Method | How it works | When it wins | What it costs |
|---|---|---|---|
| Experimental exchange rate | Run a one weight experiment, measure the movement in every metric you track, then express each head's weight in units of the metric leadership actually cares about | The weights become defensible statements such as "one share is worth 0.6 sessions per week" | One experiment per head per revision, at 1.78e6 users per arm |
| Constrained optimisation on logged data | Fit the weights offline to maximise a north star subject to guardrail constraints, using off policy estimation on logged impressions | Explores far more of the weight space than experiments can | Off policy estimates are biased where the logging policy rarely went, which is exactly where a proposed new policy wants to go |

The honest answer combines them: search offline, confirm the top two or three candidates online. This is forced by the estimate above, where a naive grid needs 7.2 times your user base.

**The cannibalisation table is the artefact to draw.** For each weight you consider raising, name what falls:

| Weight raised | What rises | What falls | Why |
|---|---|---|---|
| Comment | Sessions with a reply, community formation | Sessions per user, briefly | Comments cost time, so time spent per unit of content rises and content consumed per session falls |
| Share | Reach of new content, creator retention | Precision for the viewer | Shares are a signal about a third party's interest, not the viewer's |
| Long dwell | Depth of engagement | Short form content of genuine value | Dwell has a length confound: a longer post dwells longer without being better |
| Negative weight on hide | Reported content, complaints | Total engagement, measurably | You are choosing to show less of something people click |

A candidate who lists the metric that falls for each weight they raise is doing the actual job. A candidate who says "we optimise for meaningful interactions" has restated the problem.

**Calibration is a prerequisite, not a refinement.** A weighted sum of six numbers is meaningless if the numbers are not on a common scale. If p(share) is systematically 3x too high in a slice, w5 is effectively 3x larger for that slice, and you have a per slice value model you never designed. That is why calibration gets a full deep dive in case study 3 rather than a mention here: in ads it decides money, and here it silently decides your exchange rates.

**Rare heads need care.** Report and hide have perhaps three orders of magnitude fewer positives than click. Three consequences:

- Loss weighting or the head contributes nothing to the shared trunk's gradients.
- Calibration on a rare head is noisy, so bin by decile and accept wider error bars.
- The confidence interval on the rare head's contribution to value should be checked before the weight is raised, or you are ranking on noise.

!!! warning "When not to use a learned value model"

    If you have fewer than about two heads worth trusting, or you cannot power an experiment to move a weight, a hand set weighted sum reviewed quarterly is the right answer and it is honest. The measurement that justifies going further is a demonstrated Pareto improvement found offline that survived a confirmatory experiment. If your last three weight changes were all within noise, the value model is not your bottleneck.

### Deep dive two: integrity as a constraint layer, not a feature

The moderation system that produces integrity scores is its own design problem, with its own labelling pipeline, appeal loop and precision economics, and it is a separate case study on this page. This dive is narrower and more often botched: given those scores, how do they enter ranking?

**Three actions, three mechanisms, three different correctness requirements.**

| Action | Mechanism | Correctness requirement | Failure if you get it wrong |
|---|---|---|---|
| Remove | Hard filter on a deny set, evaluated before ranking and again before response | Exact, and fail closed. If the deny set is unavailable, the safe behaviour is to serve from cache and alert, not to serve unfiltered | Serving removed content is an incident with legal and press consequences, and no engagement gain offsets it |
| Demote | Multiply the value by a per rule factor between 0 and 1 | Approximate is fine, and the factor is a product decision | Over-demotion suppresses legitimate authors invisibly, which surfaces as creator complaints long before any dashboard |
| No action | Nothing | None | None |

**Why not just feed the integrity score into the ranker as a feature?** It is a fair question and the interviewer will ask it. Five reasons, in the order they will actually bite you:

| Reason | Detail |
|---|---|
| Clock speed | A rule change takes effect in seconds; a retrain takes a day. Harm does not wait for your training pipeline |
| Attribution | With a rule you can tell an author which policy applied. With a feature you can only say the model scored them lower |
| Appeal | An appeal has to be able to reverse a specific decision. You cannot reverse a feature's contribution to a score |
| Compensation | A feature can be outvoted by a large enough engagement term, which is precisely the outcome the policy exists to prevent |
| Auditability | A regulator, a court or your own trust team can read a rule. Nobody can read a weight |

Note what this does not say. Integrity signals can also be features, and usually are, for the many cases where a soft nudge is right. The argument is that a policy with consequences must not be only a feature.

**Sizing the human loop, because thresholds are staffing decisions.**

| Quantity | Value | Derivation |
|---|---|---|
| Posts per day | 14.4 million | 180e6 users times 5 percent posting times 1.6 posts |
| Reviewer throughput | 720 items per day | ASSUMED: 120 items per hour times 6 productive hours, and it varies enormously by content type |
| Reviewing everything | 20,000 reviewers | 14.4e6 divided by 720. This is the number that rules out review-everything, and it should be said in the room |
| Auto demote threshold, 2 percent of posts | 288,000 posts per day, no human involved | The volume is only tolerable because demotion is reversible and cheap when wrong |
| Human review threshold, 0.4 percent | 57,600 per day, so 80 reviewers | The threshold is set by the review budget as much as by the precision curve, and pretending otherwise is the mistake |
| Appeals at 3 percent of demotions | 8,640 per day, so 12 more reviewers | An appeal path with no staffing is a product claim, not a system |

The threshold that routes to humans is where precision and recall become dollars. Lower it to 0.8 percent and you need 160 reviewers. Raise it to 0.2 percent and you halve the staff and double the harm that reaches the feed unreviewed. That trade is a budget conversation with a precision-recall curve attached, and framing it that way is a senior move.

**Propagating a removal in under 60 seconds.** The requirement is not met by the classifier, it is met by the distribution path:

| Element | Design | Why |
|---|---|---|
| Deny set distribution | Push a compact set of removed identifiers to every ranking node, not a per candidate lookup | 43,800 requests per second times 600 candidates is 2.6e7 lookups per second against a remote service, which is a service you do not want to own |
| Set size | 30 days of removals at 288,000 demotions and a much smaller removal rate. At an assumed 20,000 removals per day, 600,000 identifiers times 8 bytes is under 5 MB | Small enough to hold on every node and to refresh continuously |
| Failure behaviour | If the deny set is stale beyond a threshold, keep serving with the last known set and alert. Never fail open | The cost asymmetry is total: serving stale-but-safe is fine, serving unfiltered is not |
| Second check | Re-check the final 20 against the deny set immediately before response | Costs 20 lookups and catches anything removed during the request |

!!! warning "When not to push a deny set to every node"

    If removals are under a few hundred per day and your ranking fleet is small, a cached lookup with a short time to live is simpler and meets a 60 second requirement fine. The measurement that justifies pushing is candidate lookups per second against the policy service crossing what that service can serve at your latency budget. Pushing a set to every node is a distribution system with its own staleness monitoring, and you should not own one for 300 identifiers.

### Failure modes and evaluation (feed ranking)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Metric cannibalisation | A launch raises the north star and quietly moves three guardrails, each within its own noise band, and the aggregate harm is only visible a quarter later | Fix a guardrail set before the experiment, evaluate all of them at the same power, and require a written call on any guardrail that moved more than a stated fraction of its noise band |
| Config change blast radius | A weight edited by hand takes the feed to near random ordering in the time it takes to propagate | Treat served weights as code: review, staged rollout by traffic share, automatic rollback on a guardrail breach, and a one keystroke revert |
| Pre-ranker and ranker divergence | The heavy ranker improves, the pre-ranker is not retrained, and the heavy ranker never sees the candidates it would now prefer | Track pre-ranker recall against the heavy ranker's top 600 as a first class metric, and retrain the pre-ranker on the heavy ranker's outputs |
| Feedback loop on rare heads | Demoted content gets fewer impressions, so it collects fewer labels, so the classifier that demoted it decays on exactly the class it was built for | Reserve a small unbiased sample that bypasses demotion for labelling, accept the cost explicitly, and treat it as the price of a working classifier |
| Fail open on the deny set | A policy service outage causes ranking to skip the filter and serve removed content | Fail closed by construction; the pushed set has no runtime dependency to fail |
| Position bias in six labels at once | Every head learns that position 1 is good, and the biases differ per head because the engagement rates differ | One shared position handling mechanism, applied at training and neutralised at serving, with the neutralisation verified rather than assumed |
| Calibration drift after retrain | Weights are unchanged but the effective exchange rates move because one head's calibration shifted | Gate every model launch on per head calibration error by slice, and block the launch if it regresses, independently of the ranking metric |

Service level indicators:

- Feed request latency at p50 and p99, split by pre-ranker and heavy ranker stages, so a regression has an owner.
- Pre-ranker recall at 600 against the heavy ranker.
- Per head expected calibration error, by slice, refreshed daily.
- Time from policy decision to last impression, at p99. This is the removal requirement, measured directly.
- Deny set staleness at p99 across ranking nodes.
- Guardrail panel: hides per impression, reports per impression, and the share of sessions ending immediately after one click.
- Weight configuration change log, joined to metric movements, so the exchange rates have a history.

Alert on deny set staleness, on p99 removal propagation crossing 60 seconds, on any per head calibration error breaching its threshold, and on the guardrail panel rather than on the north star, because the north star is what a bad launch improves.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| p99 under 250 ms | Yes with a pre-ranker: the heavy stage now scores 600 rather than 3,000, and the stages parallelise across candidates |
| 43,800 requests per second at peak | Yes at 197 accelerators, which is 3.3x cheaper than the single stage design |
| Removal within 60 seconds | Yes, via the pushed deny set plus the pre-response re-check, and it is measured directly rather than assumed |
| Weight change without retrain | Yes, weights are served configuration, at the cost of a configuration surface with deploy grade risk |
| Six heads with labels | Yes, with loss weighting on the rare heads, and with wider confidence intervals stated rather than hidden |
| Explainable demotion | Yes for rule based demotions, no for the model's own scoring. That boundary is the honest answer and it is worth saying before you are asked |

### Where this answer is a simplification (feed ranking)

All links in this list were checked on 1 August 2026.

- **Zhao and colleagues, "Recommending What Video to Watch Next: A Multitask Ranking System" (RecSys, 2019)** is the primary published description of a multi task ranker with a separate shallow tower for position bias, which is the mechanism referenced above.
- **Meta's engineering post ["How machine learning powers Facebook's News Feed ranking algorithm"](https://engineering.fb.com/2021/01/26/ml-applications/news-feed-ranking/) (Meta Engineering, 26 January 2021)** describes a three pass design with a lightweight pass 0 that selects roughly 500 posts before the heavier scoring pass, and a weighted combination of several predicted actions, which is the shape of V1 here.
- **Meta's Transparency Center page ["Types of content we demote"](https://transparency.meta.com/features/approach-to-ranking/types-of-content-we-demote/)** states that some content is reduced in distribution rather than removed, and groups the demoted categories by rationale, including clickbait, engagement bait and fact checked misinformation. It is a useful existence proof that the removal against demotion split can be written down and published, which is the property the audit trail above depends on.
- **X, formerly Twitter, published the source of its recommendation pipeline in March 2023 at [github.com/twitter/the-algorithm](https://github.com/twitter/the-algorithm)**, which is an unusually direct primary source for the stage structure of a production feed: candidate sources, a heavy ranker and a set of post ranking filters, readable as code rather than described in prose. The trained weights are not in the repository, so read it for the shape and not for the numbers.
- **Mosseri, ["Instagram Ranking Explained"](https://about.instagram.com/blog/announcements/instagram-ranking-explained) (Instagram Blog, 31 May 2023)** is a primary account of per surface ranking signals from the product owner, with different signal orderings named for Feed, Stories, Explore and Reels.
- **The value model is not the whole ordering.** Diversity constraints, author fatigue rules, advertisement insertion and inventory pacing all sit between the value model and the rendered page, and each of them can dominate the ordering in practice.
- **Off policy evaluation is harder than described.** Estimates degrade where the logging policy rarely went, and the proposals worth testing are exactly the ones that go there. Treat the offline search as a candidate generator for experiments, never as a substitute for them.

### Level delta (feed ranking)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Objective | Predicts engagement and ranks by it | Names several objectives and combines them with weights | Asks who owns the weights and on what clock they change, then designs the combination so that ownership is possible |
| Staging | One ranker over all candidates | Adds a pre-ranker because the candidate set is large | Derives 657 against 197 accelerators, and first tries the cheaper move of cutting candidates and measuring recall |
| Heads | One model per objective | One shared trunk with several heads | Prices the difference at 786 against 131 accelerators, then points out that the real cost of a head is its label pipeline, not its inference |
| Weight setting | Picks numbers that seem reasonable | Runs experiments per weight | Shows a grid needs 7.2 times the user base, so proposes offline search with online confirmation, and states the bias where the logging policy never went |
| Cannibalisation | Not raised | Mentions guardrail metrics | Brings a table of what falls for each weight raised, and requires a written call on any guardrail movement |
| Integrity | Adds a policy score as a feature | Separates removal from demotion | Gives five specific reasons policy cannot be only a feature (clock speed, attribution, appeal, compensation, audit) and designs the deny set distribution to fail closed |
| Human review | Not raised | Mentions a review queue | Turns the threshold into a staffing number (0.4 percent is 80 reviewers, 20,000 to review everything) and treats the threshold as a budget conversation |
| Honesty | Says the design is explainable | Says the rules are explainable | States exactly where explainability stops, which is the model's own scoring, before being asked |

What each level added:

- **Mid to senior:** the ordering stops being one number and becomes a combination with a stated shape, and the ranking stack is staged for a reason with a number behind it.
- **Senior to staff:** thresholds become budget and staffing decisions with named owners, the weight tuning method is derived from a power calculation rather than asserted, and the boundary between what the system can and cannot explain is volunteered.

---

## Case study 3: Ad click prediction

In case studies 1 and 2 the model's output was consumed by a sort, and a sort is invariant to any monotone transformation of the scores. Here the output is consumed by an auction that charges money, and an auction is not. That one sentence is the whole reason this problem is different.

### The prompt (ad click prediction)

"Design the system that predicts whether a user will click an ad, so the ad server can decide which ad to show and what to charge."

### Questions that fork the design (ad click prediction)

| Question worth asking | If the answer is A the design becomes | If the answer is B the design becomes |
|---|---|---|
| Is the auction first price or second price? | Second price: the winner pays the runner up's bid, so small ranking errors cost little and bidders have no incentive to shade | First price: the winner pays their own bid, so every bidder needs a win rate model and bid shading, and prediction error converts directly into overpayment |
| What does the advertiser pay for? | Per click: you need pCTR only, and the label arrives in seconds | Per conversion or a target cost per acquisition: you need pCTR times pCVR, and the conversion label arrives days later, which is deep dive two |
| Are you the exchange or a bidder into someone else's exchange? | Exchange: you see all demand, you set the rules, and your latency budget is your own | Bidder: an external timeout is imposed on you, often around 100 ms end to end, and a late response is a lost auction with no error message |
| What is the attribution window? | 1 day post click: labels complete fast, models stay fresh | 30 day post click and 1 day post view: a third of your conversions arrive after most models would have retrained twice |
| How many creatives are eligible per request? | Hundreds after targeting: score them all with a good model | Tens of millions: you need a targeting index that reduces the set before any model runs, and the index is the actual system |
| Does the platform guarantee budget delivery? | No: you rank and charge, nothing else | Yes: pacing becomes a control loop that modifies effective bids through the day, and a miscalibrated pCTR destabilises it |
| Is there a floor price or a reserve? | No | Yes: calibration errors near the floor change whether an auction clears at all, so absolute probability accuracy matters more than relative ordering |
| How new is the average creative? | Stable, long running campaigns | Constant churn with new creatives daily: the cold start problem from case study 1 returns, but here a wrong guess spends real money |

### Requirements (ad click prediction)

Functional:

- For each ad request, return the winning ad and the price to charge, within the exchange timeout.
- Predict click probability per candidate creative, calibrated to absolute probability, not just ordered correctly.
- Predict conversion probability for campaigns bidding on conversions.
- Attribute conversions to impressions within the advertiser's stated attribution window.
- Log every auction with the predicted probabilities, the bid, the clearing price and the model version.
- Support daily model retraining without a calibration regression.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Ad requests per day | 5 billion | ASSUMED for a mid sized network |
| Exchange timeout | 100 ms end to end | GIVEN by the exchange contract |
| Internal budget for the ad server | under 80 ms at p99 | DERIVED: the timeout minus network and serialisation margin |
| Eligible creatives after targeting | 2,000 per request | ASSUMED |
| Total active creatives | 50 million | ASSUMED |
| Base click rate | 0.5 percent | ASSUMED; the exact value changes the arithmetic below but not its shape |
| Conversion rate given a click | 3 percent | ASSUMED |
| Effective revenue per thousand impressions | 2 dollars | ASSUMED, and it is the denominator that makes every cost number below interpretable |
| Peak ad requests per second | 145,000 | DERIVED below |

### Estimation that eliminates (ad click prediction)

| Calculation | Result | What it rules out |
|---|---|---|
| 5e9 requests divided by 86,400, times 2.5 for peak | 57,900 per second average, 145,000 at peak | Ad traffic peaks less sharply than consumer app traffic because it follows the whole web, so 2.5x rather than 3x is the assumption, and it is worth stating |
| 145,000 requests per second times 50e6 active creatives | 7.25e12 scorings per second | Rules out scoring all creatives. The targeting index that reduces 50 million to 2,000 is the first system you must name, before any model |
| 145,000 times 2,000 candidates | 2.9e8 item scorings per second | The number every cost line below divides |
| A sparse model with 100 active features, at 100 ns per random lookup, so 1e5 scorings per core second | 2,900 cores | Rules out nothing yet, but note the model is memory latency bound, not compute bound, which is why the feature count matters more than the arithmetic |
| 2,900 cores at 0.04 dollars per core hour, times 24 | 2,784 dollars per day | Compare with the revenue line below before deciding anything |
| A deep model at 200,000 scorings per accelerator second | 1,450 accelerators, so 69,600 dollars per day | Rules out treating the model choice as a pure quality question |
| 5e9 impressions per day at 2 dollars per thousand | 10 million dollars of revenue per day | The denominator. The sparse model costs 0.028 percent of revenue; the deep model costs 0.70 percent |
| Incremental cost of the deep model divided by revenue | 66,816 dollars per day, which is 0.67 percent | This is the break-even. The deep model must raise revenue by more than 0.67 percent to pay for itself, which is a target you can actually test in an experiment. It rules out both "always use the biggest model" and "never afford the big model" |
| 5e9 rows per day at 200 bytes | 1 TB per day of raw training data | Rules out daily full retrains over months of raw logs on a modest cluster |
| Negative downsampling at 1 in 100: 2.5e7 positives plus 4.98e7 sampled negatives, at 200 bytes | 7.5e7 rows, so 15 GB per day, and 1.35 TB for 90 days | Rules out the storage objection to keeping a long history, and it forces the recalibration step that is deep dive one |
| Conversions per day: 2.5e7 clicks times 3 percent | 750,000 | Rules out training the conversion model on the same cadence and infrastructure as the click model. Three orders of magnitude fewer positives is a different statistical problem |

### V0, the simplest thing that works (ad click prediction)

**Predict first: you double every predicted click probability. What breaks, and what does not?**

??? note "Show answer"

    The ranking does not break at all, because doubling is monotone and the order of expected revenue per impression is unchanged when every bidder's prediction doubles by the same factor.

    What breaks is everything denominated in absolute probability: the price charged in any scheme where price depends on predicted quality, the reserve comparison that decides whether the auction clears, budget pacing which spends against a forecast of clicks, and the advertiser's reported expected cost per click. A well ordered model can be a broken model, which is why calibration gets its own deep dive.

```
+---------+   +---------------+   +----------------+
| Request |-->| Targeting     |-->| Sparse model   |
+---------+   | index, 2000   |   | p(click)       |
              +---------------+   +----------------+
                                          |
                                          v
                                  +----------------+
                                  | Auction        |
                                  | rank by bid    |
                                  | times p(click) |
                                  +----------------+
```

**Caption.** V0 reduces 50 million creatives to 2,000 with a targeting index, scores each with one sparse logistic model, ranks by bid multiplied by predicted click probability, and runs the auction; there is no calibration layer, no conversion model and no delayed feedback handling.

Why this is correct at a real network's scale:

- A sparse linear or factorisation model over hashed features trains in minutes and serves in microseconds, and for click prediction it is a genuinely strong baseline rather than a placeholder.
- The whole system is one model, one label that arrives in seconds, and one metric.
- With a second price auction and no reserve, ranking is all you need, so calibration can wait.
- It costs 0.028 percent of revenue, which means you can spend all your attention on the targeting index and the logging, both of which you will need forever.

!!! warning "When not to move off the sparse model"

    Below roughly 1e7 scorings per second, or when your auction is second price with no reserve and no pacing, the sparse model plus a good feature pipeline is the correct answer. The measurement that justifies a heavier model is an offline lift in normalised entropy or log loss that you have converted into an expected revenue lift and compared against 0.67 percent. If the expected lift is 0.2 percent, the deep model loses money and choosing it is an engineering error, not a bold bet.

### Breaking point, then V1 (ad click prediction)

**What fails: the model outputs 33 percent where the truth is 0.5 percent.**

You downsampled negatives at 1 in 100 to make daily training affordable, which was correct. The consequence is that the sampled dataset has a click rate of:

```
p_s = p / (p + r * (1 - p))
p_s = 0.005 / (0.005 + 0.01 * 0.995) = 0.005 / 0.01495 = 0.334
```

The model faithfully learns 0.334. Ranking is untouched. Pacing, reserve comparison and reporting are all wrong by a factor of 67.

How you observe it: plot predicted probability against observed click rate in deciles of prediction. A model with correct ordering and broken calibration produces a curve that is monotone and nowhere near the diagonal. The single number version is expected calibration error, the average absolute gap between predicted and observed rate across bins, weighted by bin population.

V1 adds an explicit calibration stage and a per slice calibration monitor:

```
+---------+   +---------------+   +----------------+
| Request |-->| Targeting     |-->| Sparse model   |
+---------+   | index, 2000   |   | p_s sampled    |
              +---------------+   +----------------+
                                          |
                                          v
              +---------------+   +----------------+
              | Auction and   |<--| Recalibrate    |
              | pacing        |   | invert sample  |
              +---------------+   | then isotonic  |
                     ^            +----------------+
                     |                    ^
              +---------------+   +----------------+
              | Reserve and   |   | Calibration    |
              | price rules   |   | monitor, slices|
              +---------------+   +----------------+
```

**Caption.** V1 inserts a two step recalibration between the model and the auction: an exact algebraic inversion of the negative downsampling, then a fitted monotone correction, with a monitor that reports calibration error per slice rather than only in aggregate.

!!! warning "When not to add a fitted calibration stage"

    If you did not downsample and your auction is second price with no reserve and no pacing, the algebraic step is unnecessary and the fitted step is a second model to retrain, version and monitor. The measurement that justifies it is expected calibration error above the threshold at which your downstream consumers care, which you get by asking each consumer what error they can absorb. Pacing usually answers with a small number; reporting answers with a smaller one.

### Breaking point, then V2 (ad click prediction)

**What fails: a third of your conversions have not happened yet when you train.**

For campaigns bidding on conversions, the label is not "did they click" but "did they convert", and conversions arrive over days. Train nightly on yesterday's clicks and label everything unconverted as a negative, and you are training on a systematically incomplete label.

How you observe it: take a cohort of clicks from a fixed hour, then plot the cumulative conversion count for that cohort against days elapsed. If the curve is still rising at day seven, every model trained with a shorter window is biased low, and the bias is a number you can read straight off the curve.

V2 adds a conversion path with explicit delay handling:

| Element | What it does | Why it is separate from the click path |
|---|---|---|
| Conversion model | Predicts probability of conversion given a click | 750,000 positives per day against 25 million clicks, so a different model class, cadence and regularisation |
| Delay model | Estimates the fraction of eventual conversions already observed at elapsed time t | Turns an incomplete label into an unbiased one, which is deep dive two |
| Attribution joiner | Matches conversions back to impressions inside the advertiser's window | A stateful stream join with a window measured in weeks, and it is a storage problem as much as a modelling one |
| Replay of late labels | Re-emits training rows when a late conversion arrives | Without it, every row is frozen at its first, wrong label |

!!! warning "When not to model conversion delay"

    If your attribution window is a day and 95 percent of conversions land within an hour, wait the hour and use complete labels. The delay model is a second estimator whose errors compound with the conversion model's. The measurement that justifies it is the cumulative conversion curve above: if the fraction observed at your training cut is above roughly 0.9, correcting by 1.1x is within the noise of everything else and the model is not earning its complexity.

### Deep dive one: calibration, because the auction spends money on the third decimal

**Why ordering is not enough, stated once and precisely.** A sort is invariant under any strictly increasing transformation of scores. An auction is not, because at least four downstream consumers read the number itself:

| Consumer | What it does with the absolute value | Effect of a 2x over-prediction |
|---|---|---|
| Effective bid, on a per click bid | Multiplies bid by predicted click rate to get expected revenue per impression | Uniform error cancels in the comparison, but non-uniform error silently reallocates spend between advertisers |
| Reserve price comparison | Decides whether the best bid clears the floor | Auctions clear that should not, so you sell inventory below its floor |
| Budget pacing | Spends a daily budget against a forecast of clicks | Predicts twice the clicks, paces twice as fast, and the campaign exhausts its budget by noon |
| Advertiser reporting | Reports expected cost per click before delivery | The advertiser is told a number that is wrong by 2x, which is a commercial problem, not a modelling one |

The row that matters most is the first, and it contains the subtlety candidates miss. Uniform miscalibration does not change the winner. Non-uniform miscalibration, meaning error that differs by slice, does, because it moves spend between advertisers who sit in different slices. So the metric that matters is not overall calibration error, it is calibration error per slice.

**Step one: invert the sampling exactly.** With positives kept at rate 1 and negatives kept at rate r, the sampled and true probabilities are related by:

```
p_s = p / (p + r*(1 - p))
p   = r * p_s / (1 - p_s * (1 - r))
```

Check it with the numbers above: r = 0.01, p_s = 0.3344.

- Numerator: 0.01 times 0.3344 equals 0.003344
- Denominator: 1 minus 0.3344 times 0.99, which is 1 minus 0.3311, so 0.6689
- p equals 0.003344 divided by 0.6689, which is 0.005

This step is exact, it costs two multiplications, and forgetting it is the single most common calibration bug in ad systems. It has no free parameters, so it cannot overfit and it never needs retraining.

**Step two: fit the residual miscalibration.** Even after the exact inversion, a model trained with regularisation, class weighting or an imperfect loss is not perfectly calibrated. Two standard fixes:

| Method | Shape | Parameters | When it wins | When it hurts |
|---|---|---|---|---|
| Platt scaling | A logistic on the model's logit | Two | Small validation sets, and it cannot produce wild corrections | Cannot fix a mis-shaped curve, only shift and stretch it |
| Isotonic regression | Any non decreasing step function | Many, bounded by bin count | Large validation sets, and it fixes arbitrary monotone distortion | Overfits with few points, and it produces flat regions that destroy ordering within a bin, which matters when the auction is tight |

Fit on a held out sample that is fresh, not on the training set, and fit it per slice.

**Step three: decide which slices.** Slice by anything whose population shifts between training and serving:

| Slice | Why it drifts | Consequence of not slicing |
|---|---|---|
| Position or placement | Different surfaces have different base rates by an order of magnitude | The head of the distribution calibrates well and everything else does not |
| Creative age | New creatives have thin history, so their features are near the prior | New campaigns systematically over or under bid on their first day, which is when advertisers are watching |
| Device and platform | Base click rates differ, and the mix shifts by time of day | Calibration error follows a daily cycle and looks like a mystery |
| Advertiser vertical | Base rates vary widely by industry | Spend moves between verticals for no reason anyone can name |

**Step four: monitor it as an SLI, not a launch check.** Calibration degrades continuously between retrains as the traffic mix moves. The dashboard needs predicted against observed rate by decile, refreshed hourly, per slice, with the gap alerted on. A model launch gate that checks calibration once and never again catches the first failure mode and none of the rest.

**Step five: position bias and calibration interact, and the interaction is a trap.** If you correct for position by training with position as a feature and serving with it fixed to a constant, your served predictions are calibrated for that constant position, not for the position the ad actually occupies. Both are defensible; what is not defensible is doing one and reporting the other. Say which one your predicted number means.

!!! warning "When not to build per slice calibration"

    If you have one surface, one device class and stable creatives, a single global calibration curve is right, and per slice fitting splits your validation data until each slice is noise. The measurement that justifies slicing is calibration error computed per candidate slice on a single global curve: if no slice deviates by more than the noise band of your smallest slice, you have nothing to fix and you should keep the simpler artefact.

### Deep dive two: training on labels that have not happened yet

The click label arrives in seconds. The conversion label arrives over days, and the design has to decide what to do in the meantime.

**First, characterise the delay, because everything else is a function of it.** Take a cohort of clicks from one hour and count conversions by elapsed time. Call the resulting cumulative fraction F(t). An assumed shape, stated so the arithmetic is followable, and directly measurable from your own logs within a week:

| Elapsed time | F(t), the fraction of eventual conversions already observed |
|---|---|
| 1 hour | 0.50 |
| 24 hours | 0.70 |
| 7 days | 0.85 |
| 30 days | 0.95 |

**Second, see the bias exactly.** Train nightly with a 24 hour cut and label an unconverted click as a negative. Then:

- The observed conversion rate is F(24h) times the true rate, so 0.70 times 3 percent, which is 2.1 percent.
- A model that learns 2.1 percent under-predicts by 30 percent.
- For a campaign bidding a target value per conversion, the effective bid is value times predicted conversion rate, so it bids 30 percent low.
- If win rate is locally proportional to bid over the relevant range (an assumption you should state and can test with your own auction logs), the campaign wins roughly 30 percent fewer auctions than intended and under-delivers its budget.

The advertiser experiences this as "your platform will not spend my money", which is a support ticket, not a metric regression. That is what makes it dangerous: it does not look like a model bug.

**Third, choose a correction, and know what each one assumes.**

| Approach | Mechanism | Assumes | Cost |
|---|---|---|---|
| Wait for the window | Train only on clicks older than the full attribution window | Nothing about the delay shape | The model is 7 to 30 days stale, so it cannot react to new creatives, seasonality or a competitor's campaign launch |
| Short window, constant correction | Train on a 24 hour cut and divide predictions by F(24h) | The delay shape is the same for every click | Wrong wherever delay correlates with conversion type, which it usually does: an app install converts in minutes, a car purchase does not |
| Elapsed time reweighting | Treat unconverted rows as a mixture, weighting each by the probability it is still pending given its age | You can estimate F(t) reliably, per segment | Two estimators instead of one, and their errors compound |
| Joint delay and conversion model | Fit conversion probability and the delay distribution together, so an unconverted click of age t contributes according to both | The delay distribution has a modellable form per segment | The most machinery, and the standard reference implementation for it is the 2014 paper cited below |
| Label replay | Emit a training row at click time, then re-emit it when the conversion lands | Your training pipeline can handle a mutable label | Storage and a stream join with a window as long as the attribution window |

**Fourth, the freshness against completeness trade is the actual decision.** State it as a table and let the interviewer pick:

| Training cut | Label completeness | Model staleness | Correction factor needed |
|---|---|---|---|
| 1 hour | 0.50 | Effectively live | 2.00x, which is large enough that errors in F(t) dominate |
| 24 hours | 0.70 | One day | 1.43x |
| 7 days | 0.85 | One week | 1.18x |
| 30 days | 0.95 | One month | 1.05x, and by now the creatives have changed |

There is no free choice here. Fresh models are biased and stale models are unbiased, and the correct answer depends on how fast your creative inventory turns over, which is a question you should ask rather than assume.

**Fifth, the attribution window is a business input, not a modelling choice.** A 30 day post click and 1 day post view window is a contract with the advertiser. It sets the storage horizon for the join, the length of the replay window, and the point after which a conversion is simply not yours. Modelling teams that pick this number themselves discover later that finance picked a different one.

**Sixth, note what this shares with fraud and what it does not.** Fraud detection also has delayed labels, and the fraud case study on this page treats them. The difference is that ad conversion delay is roughly stationary and benign, while fraud label delay is adversarially controlled: the delay itself is something an attacker manipulates. Same symptom, different cause, so the fixes do not transfer.

!!! warning "When not to build a delay model"

    Two cases. If F at your training cut is above about 0.9, apply the constant correction and move on. If your conversion volume is under roughly 10,000 per day, a delay model fitted per segment has segments with dozens of points, and you will be fitting noise. The cheaper alternative in both cases is a single global multiplier, recomputed weekly from the cohort curve, with the curve itself on a dashboard so you can see when the assumption stops holding.

### Failure modes and evaluation (ad click prediction)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Silent calibration regression | A retrain improves log loss and moves calibration in one slice, and the only visible symptom is that a group of campaigns paces differently | Per slice calibration as a launch gate and as an hourly SLI, alerting on the gap rather than on the model metric |
| Feedback loop through the auction | The model predicts a low rate for a creative, it loses auctions, it gets no impressions, its features never update, and the prediction is never corrected | A small always-on exploration budget for cold creatives, priced the same way as case study 1's, and logged with its propensity |
| Pacing instability | Miscalibrated predictions make the pacing controller over-correct, campaigns spend in bursts, and the burst pattern feeds back into the training data | Damp the controller, and treat pacing as a consumer with a stated calibration tolerance rather than an afterthought |
| Attribution join drift | Late conversions arrive after the join window closes and are dropped, so measured conversion rate falls without any model change | Monitor the count of conversions arriving after the window as a first class metric; a rise means either the window is too short or the delay distribution moved |
| Training and serving skew on hashed features | A feature hashing change or a vocabulary refresh shifts collisions, so training and serving disagree on what a feature means | Version the hashing scheme with the model, and refuse to load a model whose feature schema hash does not match the server's |
| Cold creative overspend | A new creative's predicted rate is high, it wins many auctions, and it turns out to be poor | Cap spend per creative until it accumulates enough impressions for a usable estimate, using the same sample size arithmetic as case study 1 |
| Clock skew across the auction path | Impression, click and conversion timestamps come from different systems, so elapsed time in the delay model is wrong | Timestamp at one authoritative point and carry it through, rather than re-stamping at each hop |

Service level indicators:

- Ad server latency at p99 against the internal 80 ms budget, and the count of auctions lost to timeout, which is the metric the exchange sees.
- Expected calibration error, per slice, hourly.
- Predicted against actual click count per campaign per day, which is the pacing consumer's view of calibration.
- Cumulative conversion curve by cohort, refreshed daily, with the correction factor derived from it published as a number.
- Conversions arriving after the attribution window closes.
- Share of impressions going to creatives with under the sample size needed for a usable estimate.
- Revenue per thousand impressions, segmented by model version, so a launch's effect is attributable.

Alert on calibration error per slice, on timeout losses, and on the late conversion count. Do not alert on log loss: it moves for reasons that are not incidents and it does not move for some that are.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| Under 80 ms at p99 | Yes with the sparse model at 2,900 cores. With a deep model at 1,450 accelerators it is achievable in one batch of 2,000 but the budget gets tight, which is a second reason to want the 0.67 percent break-even before switching |
| Calibrated absolute probability | Yes, via exact sampling inversion plus a fitted per slice correction, monitored hourly rather than at launch |
| Conversion prediction | Yes, with delay handling, and the freshness against completeness trade stated rather than hidden |
| Attribution inside the window | Yes, with a stateful join whose storage horizon is set by the business window |
| Daily retrain without calibration regression | Yes, gated on per slice calibration error, and this gate will block launches that improve log loss |
| 5 billion requests per day | Yes, at 0.028 percent of revenue for the sparse path |

### Where this answer is a simplification (ad click prediction)

All links in this list were checked on 1 August 2026.

- **He and colleagues, ["Practical Lessons from Predicting Clicks on Ads at Facebook"](https://research.facebook.com/publications/practical-lessons-from-predicting-clicks-on-ads-at-facebook/) (ADKDD, 2014)** is the primary published source for negative downsampling in this setting, for the exact recalibration relation used above, and for normalised entropy as a calibration aware evaluation metric.
- **McMahan and colleagues, ["Ad Click Prediction: a View from the Trenches"](https://research.google/pubs/ad-click-prediction-a-view-from-the-trenches/) (KDD, 2013)** is Google's primary account of the engineering around a production click model, including online learning, memory saving feature handling and calibration monitoring.
- **Chapelle, "Modeling Delayed Feedback in Display Advertising" (KDD, 2014)** is the reference formulation of the joint conversion and delay model described in deep dive two.
- **Google's announcement, ["Simplifying programmatic: first price auctions for Google Ad Manager"](https://blog.google/products/admanager/simplifying-programmatic-first-price-auctions-google-ad-manager/) (Google blog, 6 March 2019)** is the primary source for the auction shift that makes bid shading and calibration commercially load bearing rather than academically interesting.
- **Zhou and colleagues, "Bid Shading in The Brave New World of First-Price Auctions" (CIKM, 2020)** describes the win rate modelling that a bidder needs once second price no longer protects it from its own errors.
- **Criteo's [public attribution modelling and bidding dataset](https://ailab.criteo.com/criteo-attribution-modeling-bidding-dataset/) (released 10 October 2017)** is the most accessible public data with real conversion timing attached: 30 days of live traffic, 16.5 million impressions, and a conversion timestamp per row, which makes the delay curve above something you can reproduce rather than take on trust.
- **The auction is more complicated than a sort.** Reserve prices, budget pacing, frequency capping, brand safety filters and advertiser level fairness constraints all sit between the predicted probability and the served ad, and several of them can override the ranking entirely.
- **Privacy changes the feature set faster than modelling changes the model.** Identifier availability, on device measurement and aggregated reporting have all moved the inputs to these systems, and a design that assumes a stable per user identifier is describing an earlier era.

### Level delta (ad click prediction)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Framing | Designs a click prediction model | Notes that the auction consumes the prediction | Opens with the invariance argument: a sort ignores monotone transformations and an auction does not, so calibration is a requirement rather than a refinement |
| Model choice | Reaches for a deep model | Compares sparse and deep on quality | Derives a 0.67 percent revenue break-even from 69,600 dollars against 10 million dollars, and makes the model choice an experiment with a stated success threshold |
| Downsampling | Downsamples to make training fit | Downsamples and remembers to recalibrate | Writes the inversion formula, checks it numerically, and notes it is exact, parameter free and the most common bug in the category |
| Calibration scope | One global curve | Fits Platt or isotonic on a holdout | Slices by placement, creative age, device and vertical, and explains that uniform error cancels in the auction while non-uniform error reallocates spend |
| Delayed conversions | Trains on whatever labels exist | Waits for the attribution window | Puts freshness against completeness in a table with the correction factor per cut, and asks how fast creatives turn over before choosing |
| Delay correction | Not raised | Applies a constant multiplier | Names where a constant fails (delay correlates with conversion type), and gives the volume threshold below which a fitted delay model is fitting noise |
| Monitoring | Watches log loss | Adds calibration to the launch gate | Makes calibration an hourly per slice SLI, and states that log loss should not be alerted on because it moves without incidents and stays still during some |
| Failure framing | Lists model failures | Lists system failures | Names the failure the advertiser actually reports (my budget will not spend) and traces it back through bid shading to a 30 percent label bias |

What each level added:

- **Mid to senior:** the model stops being the system, and the consumers of its output start constraining its design.
- **Senior to staff:** a commercial break-even replaces a preference for a model class, the most common bug in the category is written out as algebra and checked, and the failure modes are stated in the language of the person who will report them.

---

## Case study 4: Semantic and visual search

The previous three case studies treated the retrieval index as a component with a size. Here it is the system. The two dives are the index itself, and the training decision that determines what the index is even made of.

### The prompt (semantic and visual search)

"Design search over a large catalogue, where a user types a phrase or uploads a photo and expects results that match what they meant rather than the words they typed."

### Questions that fork the design (semantic and visual search)

| Question worth asking | If the answer is A the design becomes | If the answer is B the design becomes |
|---|---|---|
| Is lexical search already in place and failing, or is there no search at all? | No search at all: build lexical search first, because it is the correct V0 and it becomes your hard negative miner later | Lexical exists and fails on paraphrase and intent: you are adding a dense path alongside it, and fusion is the design question |
| Is the image a query, a document, or both? | Query only, meaning search by photo over a text catalogue: you need one shared space and the alignment problem is the whole difficulty | Both: you index image vectors too, which triples your vector count and forces the memory arithmetic below |
| What does relevant mean here, and who decides? | Human judged relevance on a labelled set: you can measure offline and iterate fast | Downstream conversion or purchase: your offline metric is a proxy, and every offline win needs an online confirmation |
| How fast must a new or changed item become findable? | Hours: rebuild the index nightly and life is simple | Minutes: you need incremental upsert, which constrains the index family before you have chosen it |
| How often does the embedding model change? | Rarely: the index is a stable artefact | Every quarter: every change re-embeds the entire corpus, and you need a migration story with no search outage |
| Are there hard filters, such as availability, region or price range? | Rare and coarse: post-filter and over-fetch | Common and selective: filtered nearest neighbour search is a different and much harder problem, and naive post-filtering returns empty pages |
| Is the head of the query distribution large? | Yes, a few thousand queries cover most traffic: cache aggressively and spend your effort on the tail | No, mostly unique queries: caching buys little and the index has to be fast for everything |
| What is the acceptable recall? | 0.90 at 100 is fine: you can push the index hard and cheap | Near exact: your cost rises steeply and you should ask what business outcome depends on the last few points |

### Requirements (semantic and visual search)

Functional:

- Accept a text query or an uploaded image and return a ranked page of catalogue items.
- Apply hard filters (availability, region, category) so no ineligible item is ever returned.
- Reflect item additions, edits and deletions within a stated freshness window.
- Support replacing the embedding model without a search outage.
- Report retrieval recall against exact search on a fixed query set, on a schedule.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Daily active users | 50 million | ASSUMED |
| Searches per user per day | 3.5 | ASSUMED |
| Catalogue size | 500 million items | ASSUMED |
| Images per item | 3 | ASSUMED, so 1.5 billion image vectors |
| Text embedding dimension | 768 | ASSUMED; the arithmetic below is the reason to negotiate this number down |
| Image embedding dimension | 512 | ASSUMED |
| End to end search latency | under 250 ms at p99 | ASSUMED |
| Budget for the nearest neighbour stage | under 40 ms at p99 | DERIVED from the stage budget below |
| Item changes per day | 2 million | ASSUMED, which is 0.4 percent of the catalogue |
| Recall at 100 against exact search | at least 0.90 | ASSUMED, and it is the number that sets every index parameter |
| Peak queries per second | 6,100 | DERIVED below |

### Estimation that eliminates (semantic and visual search)

| Calculation | Result | What it rules out |
|---|---|---|
| 50e6 users times 3.5 searches divided by 86,400 | 2,025 queries per second, 6,100 at 3x peak | Rules out nothing yet, and that is the point: 6,100 queries per second is a small number, so the difficulty here is memory and recall, not request rate |
| 500e6 vectors times 768 dimensions times 4 bytes | 1.54 TB | Rules out full precision vectors in memory anywhere reasonable |
| The same at one byte per dimension | 384 GB | Rules out a single 64 GB node even with aggressive quantisation. You would need six shards for the text vectors alone |
| Product quantisation at 96 bytes per vector, plus 8 byte identifiers | 52 GB | Rules out sharding for capacity, for the text index. One node holds it, so recall becomes the reason to shard, not size |
| A graph index with 32 neighbours per node, at 32 times 2 times 4 bytes | 256 bytes per vector, so 128 GB of graph structure alone | Rules out a pure graph index over 500 million vectors on one node. This is the single line that decides the index family, and it decides it before recall is even discussed |
| 1.5e9 image vectors at 64 bytes quantised, plus identifiers | 108 GB | Combined with the text index, 160 GB, so four shards at 40 GB each with headroom on a 64 GB node |
| Inverted file with 11,180 lists per shard (the square root of 1.25e8), probing 32 of them | 358,000 vectors scanned per shard per query | Rules out probing hundreds of lists at this corpus size: at 96 table lookups per vector that is 3.4e7 operations, and probing 128 lists would be four times the latency budget |
| 3.4e7 operations at an assumed 1e9 such operations per core second | 34 ms per shard per query, inside the 40 ms stage budget | Rules out a single shard: one core would take 137 ms over the full corpus |
| 6,100 queries per second times 4 shards times 34 ms | 840 cores, at 0.04 dollars per core hour, so 806 dollars per day | Rules out cost as an objection to sharding. Per thousand queries this is 0.0046 dollars |
| Exact ground truth for 1,000 queries: 1,000 times 5e8 pairs times 1,536 flops | 7.7e14 flops, roughly 3 minutes on one accelerator, and bounded in practice by streaming the corpus once | Rules out the claim that you cannot measure true recall. You can, weekly, for the price of one coffee break |
| 2e6 item changes per day against 500e6 items | 0.4 percent daily churn | Rules out rebuilding the whole index per change, and rules out never rebuilding: 0.4 percent per day is 12 percent per month, which is enough centroid drift to matter |

### V0, the simplest thing that works (semantic and visual search)

**Predict first: for a catalogue search product, what fraction of queries would a well tuned lexical index already answer correctly, and which queries would it miss?**

??? note "Show answer"

    Lexical search answers exact and near exact matches, which for a catalogue means product names, brand names, model numbers, and anything the user copied from somewhere. Those are usually the majority of traffic and they are also the queries with the highest intent.

    What lexical misses is paraphrase (a description of the thing rather than its name), cross lingual queries, and anything where the user does not know the vocabulary. The honest framing is that dense retrieval is a tail improvement on a head dominated distribution, which changes how you size the investment and what you measure.

```
+--------+   +----------------+   +------------------+
| Query  |-->| Lexical index  |-->| Filter, then top |
+--------+   | inverted index |   | k by text score  |
             +----------------+   +------------------+
                     ^
             +----------------+
             | Indexer, items |
             | to terms       |
             +----------------+
```

**Caption.** V0 is an inverted index over item text with a standard term weighting score, hard filters applied as index constraints rather than after retrieval, and a nightly or streaming indexer; there is no embedding model, no vector index and no image path.

Why this is correct at a real catalogue's scale:

- Filters are native. An inverted index intersects a filter posting list with a term posting list, which is the operation it is built for, and this is the single largest practical advantage over vector search.
- Updates are incremental and cheap, so freshness is a solved problem rather than a design constraint.
- It is explainable: you can say exactly why a result matched, which matters for merchandising and for debugging.
- It gives you the query logs, the click logs and the labelled pairs that any dense model will need. You cannot train the replacement without first running the thing you plan to replace.

!!! warning "When not to add dense retrieval"

    If the share of queries returning zero or poor results is under a few percent, dense retrieval is an expensive answer to a small problem. The measurement that justifies it is the null and low engagement query rate, segmented by query length: long natural language queries failing while short keyword queries succeed is the signature that dense retrieval fixes. If failures are uniform across query length, your problem is your indexing or your filters, and embeddings will not touch it.

### Breaking point, then V1 (semantic and visual search)

**What fails: the tail of the query distribution, measured rather than asserted.**

How you observe it: bucket queries by token count and plot the null result rate and click through rate per bucket. Lexical search degrades on longer, more descriptive queries because more terms means a smaller intersection. A rising null rate with query length, while short queries perform fine, is the specific evidence that motivates a dense path. Without that plot you are adding a vector index because it is fashionable.

V1 adds a dense path and fuses it with the lexical one:

```
+--------+   +----------------+   +------------------+
| Query  |-->| Query encoder  |-->| Vector index     |
+--------+   +----------------+   | top 200 by cosine|
     |                            +------------------+
     v                                     |
+----------------+                         v
| Lexical index  |------------->+-------------------+
| top 200        |              | Fuse, then filter |
+----------------+              +-------------------+
                                          |
                                          v
                                +-------------------+
                                | Cross encoder     |
                                | re-rank top 100   |
                                +-------------------+
```

**Caption.** V1 runs the lexical and dense retrievers in parallel for 200 candidates each, fuses the two ranked lists, applies hard filters, and re-ranks the surviving top 100 with a cross encoder that sees the query and the item together.

The stage budget:

| Stage | Budget | Note |
|---|---|---|
| Query encoding | 15 ms | One small transformer forward pass on a short input, batched across concurrent queries |
| Lexical retrieval | 25 ms | Runs in parallel with the dense path |
| Vector index search | 40 ms | The number that sets the index parameters in deep dive one |
| Fusion and filtering | 10 ms | Set operations over 400 candidates |
| Cross encoder re-rank of 100 | 90 ms | The single most expensive stage, and the one to cut first if the budget is tight |
| Hydration and response | 30 ms | Metadata for the 20 items actually shown |
| **Sum** | **170 ms** | The dense and lexical stages overlap, so the critical path is encoding plus vector search plus the rest, leaving room inside the 250 ms objective |

**Fusion, honestly.** Two lists with incomparable score scales have to be merged. The options:

| Method | Mechanism | Parameters | Where it wins | Where it loses |
|---|---|---|---|---|
| Reciprocal rank fusion | Sum of 1 divided by (a constant plus rank) across lists | One | No score calibration needed, robust, and it is the correct first thing to try | Throws away score magnitude, so a result that both retrievers rank first looks the same as one that barely made each list |
| Score normalisation then weighted sum | Normalise each list's scores, then combine | One weight, plus the normalisation choice | Keeps magnitude information | Score distributions shift per query, so a normalisation tuned on average queries is wrong on unusual ones |
| Learned fusion | A small model over rank, score and query features | Many | Best when you have enough labelled pairs | Needs the labelled set, and it is a third model to retrain when either retriever changes |

The weights cannot be chosen from first principles. They are chosen on a labelled evaluation set, and if you do not have one, building it is the first task, not the last. This is the honest version of a step often waved through.

!!! warning "When not to add a cross encoder"

    The cross encoder is 90 ms of a 250 ms budget and it is by far the most expensive component here. Add it only when you can show that the fused top 100 contains the right answer but ranks it below the fold, which you measure as the gap between recall at 100 and precision at 10. If recall at 100 is itself poor, re-ranking cannot help you and the money belongs in retrieval instead.

### Breaking point, then V2 (semantic and visual search)

**What fails: 128 GB of graph structure, and an embedding model that changes.**

The capacity failure was derived above: a graph index over 500 million vectors needs about 128 GB for links alone, before any vectors. How you observe it is at build time rather than at query time, which makes it worse: the index builds fine on a sample and fails when you try the full corpus.

The second failure is operational. You improve the embedding model, which means every one of the 500 million item vectors is now in a different space. Query vectors from the new model and item vectors from the old model are not comparable, so a partial migration returns nonsense rather than degraded results. There is no gradual rollout of an embedding change without a second index.

V2 shards, changes index family, and adds a migration path:

| Element | Design | Why |
|---|---|---|
| Index family | Inverted file with product quantised residuals, four shards | 52 GB of text codes plus 108 GB of image codes fits four 64 GB nodes with headroom; a graph index does not fit at all |
| Shard routing | Broadcast the query to all four shards, merge the top k | At 6,100 queries per second the fan-out is affordable, and any attempt to route semantically defeats the point of the index |
| Rebuild pipeline | Full rebuild on a cadence set by measured centroid drift, with incremental upsert between rebuilds | 0.4 percent daily churn is 12 percent per month, which is enough to move the centroids the quantiser was trained on |
| Shadow index | The new embedding model builds a complete parallel index before any traffic moves | The only way to change embedding spaces without an outage, and it costs a second copy of the memory for the duration |
| Image path | A separate index over 1.5e9 image vectors, queried when the input is an image | Different dimension, different quantiser, different recall curve |

!!! warning "When not to shard the index"

    If your quantised index fits one node with headroom, do not shard. Sharding costs a fan-out, a merge, a per shard tail latency problem (your p99 becomes the maximum of four p99s, not the average), and four things to rebuild instead of one. The measurement that justifies it is memory footprint against node size, or scan time per query against the stage budget, whichever binds first. Both are numbers you compute before writing code.

### Deep dive one: index engineering, where recall becomes a parameter you set

**The three quantities, and the fact that you only get to pick two.** Every index decision trades memory, latency and recall. Naming that up front turns a shapeless topic into a constrained optimisation.

**Step one: memory decides the family, before recall is discussed.**

| Representation | Bytes per vector | 500 million vectors | Verdict at this scale |
|---|---|---|---|
| fp32, 768 dimensions | 3,072 | 1.54 TB | Out |
| fp16 | 1,536 | 768 GB | Out |
| int8 per dimension | 768 | 384 GB | Six or more shards for text alone |
| Product quantisation, 96 sub-vectors at 1 byte | 96 | 48 GB, plus 4 GB of identifiers | Viable on one node |
| Graph links at 32 neighbours (additional, not instead) | 256 | 128 GB on top of whatever vectors cost | Rules out the graph family here |

The graph family is excellent and it is the right answer at ten million vectors. At 500 million it loses on a memory line you can compute in ten seconds, and computing it in the room is worth more than an opinion about which index is better.

**Step two: the latency budget sets how many vectors you may scan.**

Working backwards from the 40 ms stage budget:

- Product quantisation scores a vector with roughly one table lookup and add per sub-vector, so 96 operations per vector.
- At an assumed 1e9 such operations per core second (the lookup tables are small enough to sit in cache and the codes are scanned sequentially, so this is bandwidth friendly), 40 ms buys 4e7 operations, which is about 416,000 vectors.
- Four shards means 358,000 vectors scanned per shard, which fits with margin.

State the assumption as a placeholder and offer to measure it. The structure of the argument survives whatever the real number is.

**Step three: the number of vectors scanned sets your probe count, and the probe count sets recall.**

| Lists probed | Vectors scanned per shard | Approximate latency per shard | Recall at 100 |
|---|---|---|---|
| 8 | 89,000 | 9 ms | Measured, not assumed |
| 16 | 179,000 | 17 ms | Measured, not assumed |
| 32 | 358,000 | 34 ms | Measured, not assumed |
| 64 | 716,000 | 69 ms, over budget | Measured, not assumed |

Leaving the recall column as "measured" is deliberate and it is the point of the section. Recall against probe count depends on your data's clustering structure and cannot be quoted from anyone else's benchmark. What you can quote is the shape: recall rises with diminishing returns and latency rises linearly, so there is a knee, and finding the knee is a one afternoon experiment.

**Step four: measure recall properly, and notice you can afford to.**

| Step | Detail |
|---|---|
| Fix a query set | 1,000 queries sampled to match the production distribution, including the long tail, held constant across measurements so the number is comparable over time |
| Compute exact ground truth | Brute force over the full 500 million vectors: 1,000 times 5e8 pairs at 1,536 flops is 7.7e14 flops, roughly 3 minutes of accelerator time and bounded in practice by streaming 768 GB of vectors once |
| Compute recall at 100 | The overlap between the index's top 100 and the exact top 100, averaged over queries |
| Repeat | Weekly, and on every parameter change, index rebuild and embedding model change |

The common mistake is measuring recall against a sampled sub-corpus, which is optimistic because the hard misses come from the density of the full space. If you must sample, say that the number is an upper bound.

**Step five: filtered search is where naive designs fail in production.** The interaction between a hard filter and an approximate index is genuinely hard:

| Strategy | Mechanism | Fails when |
|---|---|---|
| Post-filter | Retrieve 200, drop ineligible | The filter is selective. If 2 percent of items are in stock in the user's region, 200 candidates yield about 4 results, and the page looks empty |
| Over-fetch then filter | Retrieve 200 divided by the pass rate | The pass rate varies per query by orders of magnitude, so a fixed over-fetch is wrong for most queries and expensive for the rest |
| Partitioned indexes | Build a separate index per common filter value | Works for a handful of coarse values such as region; combinatorially impossible for conjunctions |
| Filter during traversal | The index evaluates the predicate while scanning | The right answer, and the reason to prefer an index that supports it; the cost is that a highly selective filter makes the traversal walk much further to fill k |
| Hand back to lexical | For highly selective filters, run the filter first and rank the survivors with the dense scorer directly | Correct and often overlooked, because when a filter leaves 10,000 items, exact scoring of 10,000 vectors is 15e6 operations, which is under 20 ms |

That last row is the restraint move. When the filter is selective enough that the eligible set is small, approximate search is unnecessary; exact search over the survivors is both faster and perfectly accurate. Knowing the crossover point (roughly, when the eligible set falls under the number of vectors you would have scanned anyway) is worth more than knowing three index families.

**Step six: freshness, drift and the rebuild cadence.**

| Operation | Mechanism | Cost and constraint |
|---|---|---|
| Insert | Assign to the nearest existing centroid, append the quantised code | Cheap, but the quantiser was trained on the old distribution, so new items are encoded slightly worse than old ones |
| Delete | Tombstone the identifier, filter it at merge time | At 3 percent tombstoned, over-fetch by 1.031x to compensate, which is negligible; above about 20 percent, rebuild |
| Update | Delete plus insert | Same as above |
| Rebuild trigger | Track the mean distance from each vector to its assigned centroid, and rebuild when it rises past a threshold | This is the measurement that replaces a guessed cadence. At 0.4 percent daily churn a monthly rebuild is a reasonable starting hypothesis, and the drift metric tells you whether it is right |
| Hot swap | Build into a new index version, warm it, flip the pointer, keep the old one until traffic confirms | Requires headroom for two indexes, which is the real cost of painless rebuilds |

!!! warning "When not to build a quantised sharded index"

    Under about 10 million vectors, a graph index in memory on one node gives near exact recall at single digit milliseconds and needs no quantiser training, no probe tuning and no rebuild cadence. Under about 1 million vectors, exact brute force search is roughly 1e6 times 768 multiply-adds, which is well under a second on one core and trivially parallel, and it has no recall question at all. The measurement that justifies each step up is footprint against node memory and scan time against your stage budget.

### Deep dive two: the negatives you sample decide the index you get

An index is a data structure over vectors. The vectors come from a model. The model's quality is dominated by one choice that gets one line in most treatments: what you compare each positive against during training.

**Why negatives are the lever.** A retrieval model is trained to score a matching pair above non-matching pairs. What it learns is entirely determined by which non-matching pairs it was asked to beat. Train against random items and it learns to separate shoes from garden furniture, which is a distinction the index never has to make at query time, because random items are not the ones crowding the top 100.

**Option one: in-batch negatives, and the popularity bias hiding in them.**

| Aspect | Detail |
|---|---|
| Mechanism | Within a training batch of B pairs, treat the other B minus 1 items as negatives, so one forward pass yields B times (B minus 1) comparisons |
| Efficiency | With B equal to 8,192, one batch produces about 67 million comparisons at the cost of 8,192 encodings |
| The bias | The batch is drawn from the training log, so popular items appear far more often. The model learns to down-score popular items to avoid them as negatives, which is exactly backwards for a catalogue where popular items are often the right answer |
| The correction | Subtract the log of each item's sampling probability from its logit, which is the standard sampled softmax correction. It costs one extra term and it is the difference between a usable and an unusable two tower model |

**Option two: hard negatives, and the false negative problem that comes free with them.**

Hard negatives are items the current index ranks highly that are not labelled positive. They are far more informative than random ones. They also carry a large, computable risk.

Assume a typical query has 50 genuinely relevant items in a 500 million item catalogue, of which 5 are labelled. Assume a decent retriever puts 60 percent of the relevant items in the top 100. Then:

- The top 100 contains about 30 relevant items.
- Removing the 5 labelled leaves 25 unlabelled but relevant items among the 95 remaining candidates.
- So about 26 percent of "hard negatives" mined from the top 100 are actually positives.

Training on those actively teaches the model that correct answers are wrong. This is not a subtle effect at 26 percent; it is the dominant term.

**Mitigations, in increasing order of cost.**

| Mitigation | Mechanism | Effect on the 26 percent | Cost |
|---|---|---|---|
| Mine from a lower rank band | Sample negatives from ranks 100 to 1,000 rather than the top 100 | Of the 20 relevant items outside the top 100, perhaps half fall in this band, so 10 among 900, which is about 1.1 percent | Free, and it is the first thing to try |
| Exclude by metadata | Drop candidates sharing a category, brand or near duplicate signature with the positive | Removes the most likely false negatives cheaply | Slightly reduces the difficulty of what remains |
| Denoise with a teacher | Score mined negatives with a cross encoder and drop any it scores above a threshold | Removes most false negatives at the cost of trusting the teacher | One cross encoder pass over every mined negative, which is a real training cost |
| Mix ratios | Combine random and hard negatives at a fixed ratio | Bounds the damage without removing it | One more thing to tune, and the right ratio is dataset specific |

**Option three: mine against the live index, and accept the coupling.** The strongest negatives come from the index the current model produced. That means training and indexing become a loop: train, build index, mine negatives from it, train again. Two consequences worth naming:

- Each round needs a full corpus re-embedding and index build, so the loop's period is measured in days and the number of rounds you can afford is small.
- The negatives are correlated with the current model's blind spots, which is the point, and also means each round's improvement is smaller than the last.

**What this means for the index.** The two dives are not independent. Harder negatives push positives and negatives further apart, which makes the embedding space more separable, which raises the recall you get from a fixed probe count. A better trained model does not just rank better, it lets you run the index cheaper. That coupling is the thing to say out loud, because it is the reason retrieval quality and retrieval cost cannot be owned by two teams that do not talk.

**Evaluating this at all requires a labelled set you probably do not have.** Building it:

| Source | What it gives | What it costs and biases |
|---|---|---|
| Click logs | Volume, cheap, aligned with real behaviour | Biased toward what the current system already showed, so it cannot reveal what you are missing |
| Human judgement on sampled queries | Unbiased relative to your system, and it can score items you never showed | Expensive per label, so the set is small and needs careful query sampling |
| Query reformulation pairs | Free signal: a user who searched twice then clicked tells you the first result set failed | Sparse and noisy, but it points at exactly the failures you care about |

The honest position is that click logs train the model and human judgements evaluate it, because a metric computed on the system's own output cannot detect what the system never retrieved.

!!! warning "When not to mine hard negatives"

    If your labelled set is under roughly ten thousand pairs, hard negative mining will overfit to the noise in that set, and the 26 percent false negative rate above becomes the dominant signal. Start with in-batch negatives plus the sampling correction, which is nearly free and gets most of the way.

    The measurement that justifies going further is a plateau: offline recall stops responding to more data or more training, while manual inspection of the top 100 shows near misses rather than absurdities. Near misses are what hard negatives fix; absurdities mean something else is wrong.

### Failure modes and evaluation (semantic and visual search)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Embedding mismatch | Query vectors from a new model meet item vectors from an old one, and results are not degraded but nonsense | Version the space, stamp both the query encoder and the index with it, and refuse to serve a mismatched pair rather than serving garbage |
| Silent recall decay | An index rebuild uses a quantiser trained on a stale sample, recall falls from 0.92 to 0.78, and no latency or error metric moves | Weekly recall measurement against exact ground truth, alerted on, because nothing else in your monitoring can see this |
| Empty page under a selective filter | Post-filtering removes almost all candidates for a narrow query and the user sees nothing | Fall back to filter-first exact scoring when the eligible set is small, and alert on the empty page rate by filter selectivity |
| Tail latency amplification from fan-out | p99 across four shards is the maximum of four p99s, not the average, so sharding raises p99 even when it lowers mean latency | Hedge the slowest shard after a deadline, and track per shard latency separately so a single bad node is visible |
| Hot key on a head query | A few thousand queries cover much of the traffic and hit the same lists | Cache the encoded query vector and the result identifiers, keyed on the normalised query plus the filter set, with a short lifetime |
| Rebuild memory exhaustion | Building the new index alongside the old one exceeds node memory and takes down search | Build on separate capacity and swap by pointer, never in place, and treat the second copy as the standing cost of safe rebuilds |
| Popularity collapse from in-batch negatives | The model systematically down-ranks popular items and nobody can explain why relevance fell for the most common queries | The sampling probability correction, verified by checking score against item frequency on a held out set |
| Duplicate and near duplicate flooding | A catalogue with many near identical listings fills the top 20 with the same item | Near duplicate clustering at index time, with one representative retrieved and the rest available on expansion |

Service level indicators:

- Search latency at p50 and p99, per stage and per shard.
- Recall at 100 against exact search, weekly, on the fixed query set, reported alongside the index parameters that produced it.
- Null and low result rate, segmented by query token count and by filter selectivity.
- Index freshness, measured as the age of the oldest un-indexed item change.
- Centroid drift, as the mean distance from vectors to their assigned centroid, which is the rebuild trigger.
- Embedding space version agreement between query encoder and index.
- Head query cache hit rate, which tells you how much of the tail you are actually paying for.

Alert on recall dropping between measurements, on index freshness exceeding the stated window, on space version disagreement, and on empty page rate by filter selectivity. Do not alert on mean latency; the fan-out design makes p99 the number that describes the user's experience.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| p99 under 250 ms | Yes: the critical path is 15 ms encoding plus 34 ms search plus fusion, filtering, re-rank and hydration, with the cross encoder as the first thing to cut if the budget tightens |
| Recall at 100 above 0.90 | Only if measured. The probe count is set from the latency budget, and recall is then whatever your data gives, so this requirement is a measurement obligation rather than a design guarantee. Say that plainly |
| Hard filters always respected | Yes, with filter-during-traversal where supported and filter-first exact scoring when the eligible set is small |
| Freshness | Yes for inserts and deletes via upsert and tombstones; the quantiser staleness between rebuilds is a slow quality cost, tracked as drift |
| Embedding model replacement without outage | Yes, via a shadow index and a pointer flip, at the cost of holding two copies during migration |
| 6,100 queries per second | Yes, at 840 cores and 0.0046 dollars per thousand queries, which makes cost the least interesting constraint in this design |

### Where this answer is a simplification (semantic and visual search)

All links in this list were checked on 1 August 2026.

- **Jégou, Douze and Schmid, "Product Quantization for Nearest Neighbor Search" (IEEE TPAMI, 2011)** is the primary source for the compression scheme whose 96 bytes per vector makes the memory arithmetic above work.
- **Malkov and Yashunin, ["Efficient and Robust Approximate Nearest Neighbor Search Using Hierarchical Navigable Small World Graphs"](https://arxiv.org/abs/1603.09320) (arXiv 2016, IEEE TPAMI 2020)** is the primary description of the graph family that the memory line rules out at this corpus size and that is the right answer at a smaller one.
- **Johnson, Douze and Jégou, ["Billion-Scale Similarity Search with GPUs"](https://arxiv.org/abs/1702.08734) (arXiv 2017, IEEE Transactions on Big Data 2021)** is the reference for the accelerated exact and approximate search used in the ground truth computation above.
- **Karpukhin and colleagues, "Dense Passage Retrieval for Open-Domain Question Answering" (EMNLP, 2020)** is the primary source for in-batch negatives in dense retrieval training.
- **Xiong and colleagues, "Approximate Nearest Neighbor Negative Contrastive Learning" (ICLR, 2021)** formalises mining negatives from a periodically refreshed index, which is the loop described in deep dive two.
- **Qu and colleagues, "RocketQA" (NAACL, 2021)** addresses the false negative problem directly, including denoising mined negatives with a stronger scorer.
- **Yi and colleagues, "Sampling-Bias-Corrected Neural Modeling for Large Corpus Item Recommendations" (RecSys, 2019)** is the primary treatment of the sampling probability correction that the in-batch negative section depends on.
- **Thakur and colleagues, "BEIR" (NeurIPS Datasets and Benchmarks, 2021)** measured dense retrievers out of their training domain and found lexical baselines competitive on several tasks, which is the strongest published support for the V0 in this case study.
- **Radford and colleagues, "Learning Transferable Visual Models From Natural Language Supervision" (ICML, 2021)** is the reference for training a shared image and text space, which the visual query path assumes and does not derive.
- **Published visual search systems** describe this problem end to end at production scale: ["Visual Search at Pinterest"](https://arxiv.org/abs/1505.07647) (KDD, 2015), ["Visual Search at eBay"](https://arxiv.org/abs/1706.03154) (KDD, 2017) and ["Visual Search at Alibaba"](https://arxiv.org/abs/2102.04674) (KDD, 2018) are three primary accounts with different constraints and different conclusions, which makes reading all three more useful than reading any one.
- **Query understanding is missing here.** Spelling correction, segmentation, entity recognition and intent classification sit in front of retrieval and often move metrics more than the retriever does.

### Level delta (semantic and visual search)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Proposes embeddings and a vector database immediately | Starts with lexical search and adds a dense path | Asks for the null rate by query length first, and treats dense retrieval as a tail fix on a head dominated distribution unless the data says otherwise |
| Index choice | Names a popular index by reputation | Compares graph and inverted file families | Computes 128 GB of graph links against 52 GB of quantised codes in the room, and lets that line decide the family before recall is discussed |
| Recall | Quotes a recall figure | Says recall must be measured | Prices exact ground truth at roughly 3 minutes of accelerator time for 1,000 queries, and states that a sampled sub-corpus gives an upper bound rather than a number |
| Parameters | Uses defaults | Tunes probe count for latency | Derives the scan budget from the stage budget, sets the probe count from the scan budget, and leaves the recall column explicitly as "measured" |
| Filtering | Post-filters | Over-fetches to compensate | Names the crossover where a selective filter makes exact scoring of survivors both faster and perfectly accurate, and removes the approximate index for those queries |
| Training | Treats the embedding model as given | Discusses hard negatives | Derives a 26 percent false negative rate from stated assumptions, then shows that mining from ranks 100 to 1,000 drops it to about 1 percent |
| Coupling | Treats model and index as separate | Notes that the index depends on the model | Points out that better separation buys cheaper serving at fixed recall, so retrieval quality and retrieval cost cannot be owned by teams that do not talk |
| Migration | Not raised | Mentions re-embedding | Designs the shadow index and pointer flip, and states that holding two copies is the standing cost of being able to change models at all |
| Honesty | Claims 0.90 recall | Says recall depends on the data | States that the recall requirement is a measurement obligation rather than a design guarantee, before the interviewer notices |

What each level added:

- **Mid to senior:** the vector index stops being a product name and becomes a set of parameters with a budget attached, and the simplest correct system is offered before the interesting one.
- **Senior to staff:** one memory calculation eliminates an entire index family before the discussion starts, a training decision is quantified into a failure rate with a cheap fix, and the coupling between model quality and serving cost is named as an organisational problem rather than a technical one.

---

## Case study 5: Harmful content moderation

### The prompt (content moderation)

"Design the system that decides whether a piece of user-generated content violates policy, and what happens to it after that decision."

### Questions that fork the design (content moderation)

| Question worth asking | If the answer is A the design becomes | If the answer is B it becomes |
|---|---|---|
| Is the decision made before or after the content is visible? | Before publish: the classifier sits in the write path with a hard latency budget, and every millisecond is a product cost | After publish: the classifier is a throughput problem, but harm accrues during the exposure window, so time-to-enforcement becomes an SLI |
| One policy, or many categories? | One binary policy: a single score and a single threshold | Many categories: a threshold per category, separate review queues, and a capacity allocation problem between them |
| Is any category legally mandatory to remove? | No: every threshold is a business trade-off you can tune | Yes (child safety, terrorism): an exact-match hash path with no model, no discretion and a different retention regime, sitting in front of everything else |
| Do enforced users get an appeal? | No appeal: you never observe your own false positives, and the model rots without anyone noticing | Appeal: you gain a false-positive signal, but a badly biased one, which is deep dive two |
| Is the human review budget fixed or elastic? | Elastic: automation can be recall-heavy and hand off liberally | Fixed headcount: the review budget, not the loss function, sets your thresholds. This is the usual real answer |
| Is the abuse organic or coordinated? | Organic: per-item scoring is sufficient | Coordinated campaigns: individually benign items, so per-item scoring cannot see it and you need an entity and cluster layer |
| Which modalities, and is video included? | Text and images: cheap classifiers, and cost is not the binding constraint | Video and live: frame sampling policy dominates the compute bill, and live adds a glass-to-glass budget |
| One language or many? | One: a single calibrated threshold | Many: quality varies by locale, thresholds must be per-locale, and a global threshold silently over-enforces on your worst-supported languages |

### Requirements (content moderation)

Functional:

- Score every published item against every active policy category.
- Remove, restrict or allow each item, and record which rule or model made the call.
- Route the undecidable middle to a human queue, ordered so the highest-harm items are seen first.
- Accept an appeal on any enforcement and produce a final human decision.
- Produce an auditable record for each decision: the score, the model version, the threshold and the policy version.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Items published per day | 500 million | ASSUMED; the interviewer usually gives a volume, so ask before assuming |
| Modality split | 70 percent text, 25 percent image, 5 percent video | ASSUMED; text is cheapest to produce, so it dominates on any user-generated surface |
| Mean video length | 3 minutes | ASSUMED; a short-form surface, and the number only feeds the frame-sampling arithmetic |
| Violation base rate | 0.5 percent of items, across all categories | ASSUMED; the number is contested and varies by category, so treat it as a lever and say so |
| Human reviewers | 3,000, each completing 250 items per day | ASSUMED; 250 items in 7 productive hours is 100 seconds per item, which is plausible for a mixed queue |
| Human review capacity | 750,000 decisions per day | DERIVED: 3,000 times 250 |
| Pre-publish latency, text path | under 300 ms at p99 | ASSUMED; it sits inside the post button and users notice past roughly a third of a second |
| Time from publish to enforcement, async path | under 60 seconds at p50, under 30 minutes at p99 | ASSUMED; this is the exposure window and is the number the policy team will negotiate |
| Appeal rate on enforcement | 10 percent of enforced items | ASSUMED; most enforced users do not contest, and this rate is measurable on day one |
| Peak publish rate | 17,000 items per second | DERIVED below |

### Estimation that eliminates (content moderation)

| Calculation | Result | What it rules out |
|---|---|---|
| 500e6 divided by 86,400 | 5,787 items per second, 17,000 at 3x peak | Rules out treating throughput as the hard part. Seventeen thousand scoring calls per second is an ordinary service, so do not spend interview minutes on horizontal scaling |
| 0.5 percent of 500e6 | 2.5 million violating items per day | Rules out human review of violations. Capacity is 750,000, which is 30 percent of the violating volume |
| 750,000 divided by 500e6 | 0.15 percent of traffic can reach a human | Rules out any threshold that queues more than 1.5 items per thousand. The capacity budget, not the loss curve, is what sets the band |
| 750,000 capacity minus 120,000 appeals (10 percent of an assumed 1.2 million enforcements) | 630,000 primary decisions, so 0.126 percent of traffic | Rules out treating appeals as free. Appeals consume 16 percent of the entire review budget before a single new item is looked at |
| Heavy multi-modal model at 400 ms per item, 4 concurrent streams per accelerator | 10 items per second per accelerator, so 1,700 accelerators to run it on all traffic at peak | Rules out one big model on every item. This single line forces the cascade |
| Same model on an uncertain band of 2 percent of traffic | 116 items per second mean, 350 at peak, so 35 accelerators | Rules out arguing that the heavy model is unaffordable. It is affordable on a band, and unaffordable on everything |
| 5 percent of 500e6 videos times 180 frames (1 frame per second at 3 minutes) | 4.5 billion frames per day, 52,000 frames per second | Rules out dense frame scanning. At 200 images per second per accelerator that is 260 accelerators for video alone |
| Same videos sampled at 1 frame per 10 seconds | 18 frames per video, 5,200 frames per second, 26 accelerators | Rules out the claim that video is inherently unaffordable. Sampling policy is a 10x lever, so it belongs in the design, not in an appendix |
| 10 million known-illegal hashes at 32 bytes | 320 MB | Rules out a network hop for the exact-match path. The set fits in memory on every scoring node |
| Auto-remove band enforcing 1.2 million items per day at 98 percent precision | 24,000 wrongful removals per day | Rules out shipping the auto-remove band without an independent audit, because at a 10 percent appeal rate only 2,400 of those 24,000 are ever contested |
| 2,400 genuine false positives inside 120,000 appeals | 2 percent of the appeal queue is a real error | Rules out reviewing appeals first-in-first-out. Ninety-eight percent of that queue is correct decisions being re-read by hand |

### V0, the simplest thing that works (content moderation)

**Predict first: at what daily item volume does one model plus one human queue stop working?**

??? note "Show answer"

    Around 1 million items per day. At a 0.5 percent violation rate that is 5,000 violating items, and a queue tuned to catch most of them will surface roughly 15,000 items, which is 60 reviewers. Below that, a single model, a single threshold and a queue is not a compromise; it is the right design, because every category-specific threshold you add is a number nobody has the data to set.

```
+--------+    +--------------+    +-------------------+
| Client | -> | Publish API  | -> | Content store     |
+--------+    +--------------+    +-------------------+
                     |
                     v
              +--------------+    +-------------------+
              | One          | -> | Review queue      |
              | classifier   |    | humans decide     |
              +--------------+    +-------------------+
```

**Caption.** V0 publishes the item immediately, scores it asynchronously with one classifier, and sends everything above a single threshold to one human queue where a person makes the enforcement decision.

Why this is correct at the stated smaller scale:

- Every enforcement has a human behind it, so precision is whatever your reviewers achieve and you owe no calibration argument.
- One threshold is one number to move when the policy team complains, and one number to explain in a transparency report.
- There is no cascade, so there is no band to size, no second model to keep in sync, and no version skew between tiers.
- The audit trail is trivial: score, reviewer, decision.

!!! warning "When not to move off V0"

    Below roughly 1 million items per day, or whenever your queue is not saturated, adding an auto-enforce band buys nothing and costs you every wrongful removal it makes without a human noticing. The measurement that justifies the move is queue oldest-item age exceeding your exposure objective on a normal day, not on a spike. Cheaper alternative first: raise the single threshold and accept lower recall until the age curve flattens.

### Breaking point, then V1 (content moderation)

**What fails: at 500 million items per day, any threshold that reaches a human on more than 0.15 percent of traffic overflows the queue permanently.**

How you observe it: plot the age of the oldest item in the queue, not the queue depth. Depth is noisy and recovers overnight; age grows linearly once arrival rate exceeds service rate and never comes back down. A straight upward line on oldest-item age is the unambiguous signature of a capacity shortfall, and it distinguishes this from a model that suddenly got noisier.

The second observable is the exposure integral: items published, multiplied by the seconds they were visible before enforcement. That number is the actual harm, and it grows quadratically with queue age while depth grows linearly.

V1 is a cascade with two machine bands and one human band:

```
+---------+   +--------------+
| Publish | ->| Hash match   | -> remove, known illegal
+---------+   +--------------+
                     | no hit
                     v
              +--------------+
              | Cheap tier   | -> allow, score below low
              | text + image |
              +--------------+
                     | in the uncertain band
                     v
              +--------------+
              | Heavy tier   | -> remove, score above high
              | multi-modal  |
              +--------------+
                     | still in the band
                     v
              +--------------+   +------------------+
              | Review queue | ->| Enforcement and  |
              | by category  |   | appeal record    |
              +--------------+   +------------------+
```

**Caption.** V1 checks an in-memory hash set first, then a cheap per-modality classifier that resolves most traffic, then a heavy multi-modal model on the 2 percent it cannot resolve, and finally a human queue for the 0.126 percent that neither model can decide.

The structural point is that there are two different bands, sized by two different constraints:

| Band | Sized by | Width | Why that constraint |
|---|---|---|---|
| Cheap tier hands to heavy tier | Accelerator budget | 2 percent of traffic | 35 accelerators is affordable, 1,700 is not |
| Heavy tier hands to humans | Reviewer headcount | 0.126 percent of traffic | 630,000 decisions per day after appeals take their share |

Most candidates draw one band. Drawing two, and naming the different constraint behind each, is the move that reads as having operated one of these.

!!! warning "When not to add the heavy tier"

    If the cheap tier's uncertain band is already inside your human capacity, the heavy tier removes nothing and adds a second model to calibrate, version and monitor. The measurement that justifies it is the fraction of traffic in the cheap model's uncertain band exceeding your human budget, sustained over a week. Cheaper alternative: retrain the cheap model on the queue's decisions, which narrows its own band for free.

### Breaking point, then V2 (content moderation)

**What fails: per-item scoring cannot see a coordinated campaign, and nobody is measuring what the auto-remove band gets wrong.**

Two independent failures arrive together at this scale.

The first is coordination. Ten thousand accounts each posting one individually ambiguous item is a campaign, but every item scores at 0.4 and none crosses a threshold. You observe it as a category whose reported-item volume rises sharply while its enforcement volume does not move. That divergence between user reports and machine enforcement is the cleanest detector you have.

The second is unmeasured error. The auto-remove band produces 24,000 wrongful removals per day and surfaces 2,400 of them. Nothing in V1 estimates the other 21,600, and nothing at all estimates the violating content the cheap tier confidently allowed.

V2 adds an entity layer and a measurement layer:

```
+----------------+     +------------------+
| Item scores    | --> | Entity roll-up   |
| from V1        |     | actor, cluster   |
+----------------+     +------------------+
                              |
                              v
+----------------+     +------------------+
| Random audit   | --> | Prevalence and   |
| sample of both |     | precision report |
| allowed and    |     +------------------+
| removed        |             |
+----------------+             v
                       +------------------+
                       | Threshold        |
                       | controller       |
                       +------------------+
```

**Caption.** V2 rolls item scores up to accounts and to clusters of accounts sharing behaviour, and separately draws a random audit sample from both allowed and removed content to estimate prevalence and precision, which then drive the thresholds.

The audit sample is not a nice-to-have. It is the only unbiased estimate in the system, and its sizing is deep dive two.

!!! warning "When not to build the entity layer"

    If your reported-item volume tracks your enforcement volume within its normal noise band, you do not have a coordination problem and the entity layer is a second scoring system to operate for nothing. The measurement that justifies it is a sustained gap between report rate and enforcement rate in a category. Cheaper alternative: a rate rule on new accounts, which catches the crudest campaigns for the cost of one counter.

### Deep dive one: setting the operating point when review capacity binds before the loss function does

Start from the decision, not the metric. For an item with violation probability `s`:

- Removing it costs `(1 - s)` times `C_fp`, the cost of a wrongful removal.
- Leaving it up costs `s` times `C_fn`, the cost of leaving a violation visible.

Auto-removal is correct when `s` times `C_fn` exceeds `(1 - s)` times `C_fp`, which rearranges to a threshold that is pure arithmetic:

`T_high = C_fp / (C_fp + C_fn)`

That formula is the entire trade-off, and it lets you argue about cost ratios instead of arguing about thresholds.

| Category | Assumed `C_fn / C_fp` | `T_high` from the formula | What the number tells you |
|---|---|---|---|
| Child safety | 1,000 | 0.001 | The formula says remove on almost any signal, which is why this category is a hash-match path with mandatory reporting rather than a threshold at all |
| Terrorism and violent extremism | 200 | 0.005 | Same conclusion, and the same reason it gets its own pipeline |
| Hate speech | 20 | 0.048 | The naive answer is aggressive enough to be wrong, which is the point below |
| Graphic violence | 5 | 0.17 | A genuine threshold decision, defensible either way |
| Spam and scams | 0.3 | 0.77 | Wrongful removal costs more than the spam does, so require high confidence |

**Where the naive formula breaks, and this is the senior point.** `C_fp` is not constant. A user's first wrongful removal costs an apology. Their third costs the account. So the marginal cost of a false positive rises with how many that user has already received, which makes `C_fp` convex in enforcement volume and pushes `T_high` up, sharply, for high-volume categories. Model `C_fp` per user, not per item.

**Where capacity overrides all of it, and this is the staff point.** The band between `T_low` and `T_high` must fit 0.126 percent of traffic. When the sum of the per-category bands implied by the cost formula exceeds that budget, you are no longer choosing thresholds. You are allocating a fixed number of human decisions across categories.

Worked allocation, using the 630,000 daily primary decisions:

| Category | Share of the band | Decisions per day | Justification |
|---|---|---|---|
| Hate speech | 30 percent | 189,000 | Highest `C_fn`, and the category where model quality is worst in low-resource languages |
| Harassment | 25 percent | 157,500 | Context-dependent, so models resolve it poorly and humans add the most |
| Graphic violence | 20 percent | 126,000 | Newsworthiness exceptions cannot be modelled |
| Regulated goods | 15 percent | 94,500 | Jurisdiction-specific, so the correct answer varies by viewer |
| Spam | 10 percent | 63,000 | High `T_high`, so the band is naturally narrow and models do well |

Say out loud that this table is a policy decision wearing an engineering costume, and that your job is to make the trade explicit rather than to hide it inside a threshold config file.

!!! warning "When not to use per-category thresholds"

    Below roughly 50,000 labelled items per category, a per-category threshold is fitted to noise. The measurement that justifies splitting is a per-category precision-recall curve whose confidence band is narrower than the difference between the thresholds you are proposing. Cheaper alternative: one threshold, plus a hard allow-list for the two or three categories with legal mandates.

### Deep dive two: the appeal loop is a biased label source, and the audit sample is the fix

The appeal loop looks like free labelled data. It is not, and naming exactly how it is biased is the most transferable idea in this case study.

**What appeals can and cannot see:**

| Error type | Does an appeal surface it? | Consequence |
|---|---|---|
| False positive, content wrongly removed | Sometimes, if the user bothers | You get a censored sample, not a random one |
| False negative, violation left visible | Never | Appeals give you zero information about recall |
| Correct removal | Yes, 98 percent of the appeal queue | Your appeal queue is mostly re-reading correct work |

**Why the censoring is not random.** Appeal propensity correlates with account tenure, with language support in the appeal interface, and with whether the user believes appealing works. So the overturn rate on appeals is not the false-positive rate, and it moves when you change the appeal interface, which makes it useless as a quality metric.

**The fix is a random audit sample, and it is cheap.** Draw items at random from what you removed and from what you allowed, and have humans label them.

Sizing the precision audit. To estimate a 2 percent false-positive rate to within plus or minus 0.5 percentage points at 95 percent confidence:

`n = 1.96^2 times 0.02 times 0.98 / 0.005^2 = 3.84 times 0.0196 / 0.000025 = 3,011`

So about 3,000 audited removals per category per period. Across five categories that is 15,000 decisions, which is 2.4 percent of your 630,000 daily budget. Cheap for the only trustworthy precision number you own.

Sizing the prevalence audit, which is harder because the base rate is low. To estimate a 0.5 percent violation rate among allowed content to within plus or minus 0.1 percentage points:

`n = 3.84 times 0.005 times 0.995 / 0.000001 = 19,104`

That is 19,100 audited allowed items, 3 percent of the daily budget. Together the two audits cost about 5.4 percent of review capacity, and they buy the two numbers that every other metric in the system is a proxy for.

**Prevalence, not removals, is the metric that means something.** Removal counts go up when you get better and also when abuse gets worse, so the number is uninterpretable on its own. Prevalence, the estimated share of views that are of violating content, moves in one direction only when you improve.

Meta's [Community Standards Enforcement Report](https://transparency.meta.com/reports/community-standards-enforcement/) has published prevalence alongside removal counts since its [first edition in May 2018](https://about.fb.com/news/2018/05/enforcement-numbers/), and Meta's [prevalence metric page](https://transparency.meta.com/policies/improving/prevalence-metric/) defines it as estimated views of violating content divided by estimated total views, which is exactly the sampled quantity above. All three links checked 1 August 2026.

**Weight the sample by views, not by items.** An item seen once and an item seen a million times are not equally harmful. Sampling proportional to views gives you view-weighted prevalence, which is what the harm actually is, and it also concentrates your expensive human labels on content that matters. The cost is that rare-but-severe content is under-sampled, so run a second stratum for high-severity categories at a fixed rate.

**Feeding audits back into training is where teams go wrong.** Audit labels are your evaluation set. If you train on them, you no longer have an unbiased evaluation set, and you will not notice for months. Keep two disjoint audit streams: one that trains, one that only ever evaluates, and never let a single item cross.

!!! warning "When not to run a prevalence audit"

    If your violation base rate is above roughly 5 percent, the audit is unnecessary because ordinary review volume already gives you a good estimate. The measurement that justifies a dedicated random sample is a base rate low enough that your queue sees a biased slice, which is any surface where automation decides more than about 90 percent of enforcement. Cheaper alternative: audit only the categories with a legal or regulatory reporting duty.

### Failure modes and evaluation (content moderation)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Feedback loop poisoning | The model trains on labels produced by decisions the model itself made, so its own errors become ground truth and its confidence rises while accuracy falls | Reserve a randomised slice where enforcement follows a frozen policy, and train only on human labels drawn from a sampling frame the model does not control |
| Metastable queue | A news event doubles arrivals, the queue backs up, reviewers rush, quality falls, more items get appealed, appeals consume capacity and the queue never drains after the event ends | Shed the lowest-harm categories from the queue first, with a pre-agreed order, and cap the appeal queue's share of capacity |
| Threshold drift after a model swap | A new model version has a different score distribution, so the old threshold now selects a different fraction of traffic | Recalibrate to a fixed traffic fraction rather than a fixed score, and gate every release on the band width, not on offline area under the curve |
| Hot category surge | A single event makes one category 40x its usual volume while the others sit idle | Per-category capacity floors and a shared elastic pool, so a surge cannot starve child safety |
| Thundering herd on the heavy tier | The cheap model degrades on a locale, its uncertain band widens, and the heavy tier receives 10x its provisioned load | Admission control on the heavy tier by band-rate, not by queue depth, so it sheds to the human queue rather than timing out |
| Reviewer anchoring | Reviewers see the model score before deciding, and agree with it more often than they should | Blind the score on the audit stream, always, even if you show it on the production queue for speed |
| Locale silence | A language with a weak model produces almost no enforcement, which looks like a clean locale rather than a blind spot | Report enforcement per thousand items per locale, and alert on a locale sitting far below the global rate |

Service level indicators:

- Time from publish to enforcement, at p50 and p99, segmented by category. The segmentation is the point; a global p99 is dominated by the highest-volume category.
- View-weighted prevalence per category, from the audit sample, reported with its confidence interval.
- Auto-remove precision per category, from the audit sample, never from appeals.
- Age of the oldest item in the review queue, per category.
- Uncertain-band rate as a fraction of traffic, which is the leading indicator that predicts queue overflow days before depth moves.
- Appeal overturn rate, reported as an operational load metric, explicitly not as a quality metric.

Alert on the uncertain-band rate crossing the capacity budget, on oldest-item age crossing the exposure objective, and on audited auto-remove precision falling below its floor. Do not alert on appeal overturn rate, because it moves for interface reasons.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| 17,000 items per second at peak | Yes: the hash set and cheap tier absorb 98 percent, and only 350 per second reach an accelerator |
| Publish to enforcement under 60 s at p50 | Yes for the machine bands. The human band is minutes to hours, which is why the band must stay at 0.126 percent |
| Under 300 ms at p99 on the pre-publish text path | Yes, provided only the hash set and the cheap text model sit in that path. The heavy tier must be asynchronous |
| Human capacity of 750,000 per day | Yes, and it is the constraint that sets the band, with 5.4 percent reserved for audits and 16 percent for appeals |
| Auditable record per decision | Yes, and the model version is in the record, which is what makes threshold drift diagnosable |

### Where this answer is a simplification (content moderation)

All links in this list were checked on 1 August 2026.

- **Policy is the hard part, not the classifier.** Most disagreement in a real moderation system is between two humans reading the same policy, not between a model and the truth. Inter-rater agreement is the ceiling on model quality, and if it sits at 80 percent, an offline metric above 80 percent is measuring your labelling process rather than the world.
- **Roberts, "Behind the Screen: Content Moderation in the Shadows of Social Media"** (Yale University Press, 2019) is the primary ethnographic account of the human review layer this design treats as a throughput number, and it is the correct corrective to that framing.
- **[The Santa Clara Principles on Transparency and Accountability in Content Moderation](https://santaclaraprinciples.org/)** (first published 2018, expanded in a second version after consultation) set out the notice, appeal and numbers-reporting expectations that this design's audit trail exists to satisfy.
- **Meta's [Community Standards Enforcement Report](https://transparency.meta.com/reports/community-standards-enforcement/)** is the clearest public worked example of prevalence as a sampled estimate rather than a removal count. Meta states on that page that the report is published semi-annually with the data broken down by quarter, so check the cadence on the page rather than assuming it, because it has changed.
- **Xu and colleagues, "Deep Entity Classification: Abusive Account Detection for Online Social Networks"** (USENIX Security, 2020) is a primary published account of the entity-level layer that V2 adds, and it argues the case for account-level rather than item-level features far better than an interview answer can.
- **Hash matching is a policy artefact, not a technique.** Perceptual hashing systems such as [PhotoDNA](https://www.microsoft.com/en-us/photodna), which Microsoft says it developed with Dartmouth College in 2009 and later donated to the National Center for Missing and Exploited Children, work because a curated shared database exists, and building that database is a legal and institutional problem, not an engineering one.
- **The appeal path is increasingly regulated.** The EU Digital Services Act (Regulation 2022/2065, applicable to very large platforms since 2023) imposes statement-of-reasons and internal complaint-handling duties that change the appeal design from a product choice into a compliance requirement.

### Level delta (content moderation)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Draws a classifier and a review queue | Asks whether the decision is pre or post publish, because it decides where the latency budget lives | Asks what the review headcount is first, because that number sets every threshold on the page |
| Cascade | One model | Two tiers, cheap then heavy | Two tiers with two differently sized bands, and names the different constraint behind each |
| Thresholds | "Tune for precision and recall" | Picks a threshold per category | Derives `T_high` from the cost ratio, then shows the cost ratio breaks because `C_fp` is convex per user, then shows capacity overrides both |
| Video | Mentions frame extraction | Samples frames | Prices 1 frame per second at 260 accelerators against 1 per 10 seconds at 26, making sampling policy an architectural decision |
| Measurement | Reports items removed | Adds precision and recall from the review queue | Sizes a random audit at 3,011 for precision and 19,104 for prevalence, and refuses to use appeals as a quality metric |
| Appeals | Adds an appeal button | Notes appeals give false-positive signal | Shows the appeal queue is 98 percent correct decisions, and that appeal propensity varies with interface and locale, so the overturn rate is uninterpretable |
| Coordination | Not raised | Adds account-level features | Names the report-rate versus enforcement-rate divergence as the detector, before proposing any model |
| Train and eval hygiene | Not raised | Keeps a holdout | Keeps two disjoint audit streams, one that trains and one that only evaluates, and explains what breaks if they mix |

What each level added:

- **Mid to senior:** the design stops being one model and becomes a routing problem with an explicit budget at each stage.
- **Senior to staff:** the numbers that decide the design come from operational constraints and sampled measurement rather than from the loss function, and the candidate names which of their own metrics are untrustworthy before the interviewer does.

---

## Case study 6: Fraud detection

### The prompt (fraud detection)

"Design the system that decides, while a payment is being authorised, whether to approve it, decline it or challenge the customer."

### Questions that fork the design (fraud detection)

| Question worth asking | If the answer is A the design becomes | If the answer is B it becomes |
|---|---|---|
| Who bears the loss? | We do: the objective is total loss, and a wrongful decline costs us margin, so both error types are ours to price | The merchant or issuer does: the objective is contractual, thresholds become per-merchant, and you now have a multi-tenant calibration problem |
| Is the decision inside the authorisation path? | Yes: a hard latency budget, and only precomputed or streaming features are reachable | No, review after capture: you can call slow enrichment and reverse the transaction later, and the whole latency section disappears |
| Can we challenge instead of decline? | Binary approve or decline: one threshold, and every mistake is expensive | Three-way with a step-up challenge: two thresholds, and the challenge is a label-generating action, which changes the data strategy more than it changes the model |
| When does ground truth arrive? | Hours, from user reports: online learning becomes viable and this is a fast-feedback problem | Weeks to months, from chargebacks: label delay dominates the design, which is deep dive one |
| Is the fraud organic or ring-based? | Organic and independent: per-transaction features carry the model | Rings: shared devices, addresses and cards, so entity and graph features find what per-transaction scoring cannot |
| Can the attacker probe our decisions? | No, single-shot fraud: static thresholds are fine | Yes, card testing at volume: every decline is free information for them, and you need randomisation and per-entity rate limits |
| What share of traffic is new accounts? | Small: history-based features work | Large: cold start dominates and device, network and behavioural signals carry the model instead |
| Is there a regulatory strong-authentication mandate? | No: the challenge rate is purely an economic choice | Yes: some transactions must be challenged regardless of score, and your challenge budget is partly spent for you |

### Requirements (fraud detection)

Functional:

- Return approve, challenge or decline for every authorisation request.
- Attach a reason code and a model version to every decision, for dispute handling and for debugging.
- Let an analyst review, override and annotate any decision, and feed that annotation back as a label.
- Support a rule that can be written and deployed within minutes, without a model release.
- Reverse or refund after the fact when a decision turns out wrong.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Transactions per day | 40 million | ASSUMED; ask for it, because it changes almost nothing here and knowing that is the point |
| Peak factor | 4x mean, during a shopping event | ASSUMED; retail traffic concentrates far harder than social traffic |
| Decision latency | under 150 ms at p99, server side | ASSUMED; it sits inside an authorisation round trip with several other hops |
| Fraud base rate | 0.1 percent of transactions, 10 basis points | ASSUMED; it varies enormously by vertical, so treat it as a lever |
| Mean transaction value | 60 dollars | ASSUMED; only used to convert error counts into money |
| Cost of a wrongful decline | 5 dollars of lost margin and goodwill | ASSUMED; the true number needs a churn study, and saying so is worth a sentence |
| Chargeback maturity | 50 percent of eventual chargebacks by day 30, 85 percent by day 60, 99 percent by day 120 | ASSUMED; shaped by scheme dispute windows and by how often people read statements |
| Availability of the decision service | 99.99 percent, with a defined fallback decision | ASSUMED; a fraud service that fails closed stops all revenue, so the fallback is a design decision, not an accident |
| Peak decision rate | 1,852 per second | DERIVED below |

### Estimation that eliminates (fraud detection)

| Calculation | Result | What it rules out |
|---|---|---|
| 40e6 divided by 86,400 | 463 decisions per second, 1,852 at 4x peak | Rules out scale-out as an interview topic. Two thousand requests per second is one modest service tier, so if you spend ten minutes on sharding you have chosen the wrong ten minutes |
| 0.1 percent of 40e6 times 60 dollars | 40,000 fraudulent transactions per day, 2.4 million dollars per day at risk | Rules out doing nothing, and gives you the budget ceiling for everything that follows |
| Operating at 99 percent recall with 20 percent precision | 198,000 declines per day, of which 158,400 are good customers, costing 792,000 dollars against 2.376 million dollars of fraud stopped, netting 1.584 million | Rules out "maximise recall". This is the single most common wrong instinct in this interview |
| Operating at 90 percent recall with 60 percent precision | 60,000 declines, 24,000 wrongful, costing 120,000 dollars against 2.16 million stopped, netting 2.04 million | Rules out the high-recall point above by 456,000 dollars per day |
| Operating at 70 percent recall with 85 percent precision | 32,941 declines, 4,941 wrongful, costing 24,706 dollars against 1.68 million stopped, netting 1.655 million | Rules out the conservative point too. The optimum is interior, and you found it with three lines of arithmetic |
| 50 percent chargeback maturity at day 30 | Half of every recent month's labels are missing at any moment | Rules out a naive weekly retrain on "all data with labels", because the recent negatives are half fiction |
| Assumed 21-day half-life of precision at a fixed threshold for a frozen model | After 63 days a frozen model retains one eighth of its launch precision | Rules out a quarterly retrain cadence, and it collides head-on with the 30-day label maturity, which is the whole problem |
| 1,852 peak decisions times 8 velocity features | 14,816 feature reads per second | Rules out computing velocity by querying the transaction table. It needs a streaming aggregate store, and 15,000 reads per second is small enough that one is enough |
| A card tester at 1 attempt per second against 463 mean | 0.2 percent of traffic | Rules out detecting card testing in aggregate metrics. It is inside the noise band, so it needs per-entity rate detection |
| 40e6 transactions times 3 shared attributes over a 30-day window at 32 bytes | 3.6 billion edges, 115 GB | Rules out a single commodity node for a 30-day entity graph |
| Same graph over a 7-day window | 840 million edges, 27 GB | Rules out the distributed graph engine. Ring lifetimes are short, so a week fits in memory on one machine and that is the right answer |
| Exploration at 0.5 percent of 60,000 declines | 300 approvals per day, roughly 180 of them fraudulent, costing 10,800 dollars per day | Rules out arguing that exploration is unaffordable. It is 0.5 percent of the daily net benefit and it buys the only unbiased view of the region you decline |

### V0, the simplest thing that works (fraud detection)

**Predict first: below what transaction volume does a hand-written rule set beat a trained model?**

??? note "Show answer"

    Around 10,000 transactions per day. At a 0.1 percent base rate that is 10 fraud cases per day, 3,650 per year, and a model trained on that many positives has a confidence interval wider than the difference between the operating points you are choosing between. Rules also change in minutes, which matters more than accuracy when the attacker moves in hours. The crossover is about positives per week, not about revenue.

```
+-----------+   +---------------+   +----------------+
| Auth      |-->| Rule engine   |-->| approve or     |
| request   |   | ~20 rules     |   | decline        |
+-----------+   +---------------+   +----------------+
                        |
                        v
                +---------------+
                | Case queue    |
                | analysts      |
                +---------------+
```

**Caption.** V0 evaluates about twenty hand-written rules on the authorisation request, decides immediately, and sends the ambiguous cases to an analyst queue for a human call.

Why this is correct at the stated smaller scale:

- Every rule is readable, so a dispute has an answer and a regulator has an audit trail.
- A new attack pattern becomes a new rule the same afternoon, which no retraining pipeline can match.
- Analysts generate labels as a side effect of working the queue, which is how you accumulate the training data you will later need.
- There is no feature store, no training pipeline and no model registry to operate.

!!! warning "When not to move off rules"

    Below roughly 50 fraud cases per week, or when your rule set is under about 30 rules and its precision is still stable month over month, a model is over-engineering. The measurement that justifies the move is per-rule precision declining while the rule count grows, which means analysts are patching faster than the rules generalise. Cheaper alternative: keep the rules and add one anomaly score as a twenty-first rule.

### Breaking point, then V1 (fraud detection)

**What fails: hand-written rules decay under probing, and at 40 million transactions per day the decay is faster than the humans writing them.**

How you observe it: track precision per rule and firing volume per rule, as time series. A healthy rule has flat precision. A probed rule shows falling precision with rising volume, because the attacker has found the shape of the boundary and is now producing traffic that sits just inside it. The second observable is the chargeback attribution report: the share of confirmed fraud that fired no rule at all. When that share crosses roughly a third, the rule set has stopped covering the space.

V1 adds features, a model and a third decision:

```
+-----------+   +---------------+   +---------------+
| Auth      |-->| Feature       |-->| Rules + model |
| request   |   | service       |   | score         |
+-----------+   +---------------+   +---------------+
                  ^         ^               |
                  |         |               v
        +---------------+ +---------------+ approve
        | Stream aggs   | | Batch profile | challenge
        | 60s, 1h, 30d  | | store         | decline
        +---------------+ +---------------+
                  ^
        +---------------+
        | Event log     |
        +---------------+
```

**Caption.** V1 assembles velocity features from a streaming aggregate store and slow profile features from a batch store, scores them with a model alongside the surviving rules, and emits one of three outcomes rather than two.

Three things changed, and each earns its place:

| Addition | Why it is needed here | What it costs |
|---|---|---|
| Streaming aggregates at 60 s, 1 h and 30 d | Card testing and burst fraud are only visible as counts over short windows, which the request itself does not contain | A streaming job whose lag becomes a new failure mode, and a training-serving skew risk if the offline job computes the same windows differently |
| The challenge outcome | It converts an expensive binary mistake into a cheap question, and a passed or failed challenge is a label that arrives in seconds instead of weeks | Friction, measured as challenge abandonment, which is a real conversion cost you must budget |
| Rules retained beside the model | Rules cover the attacks discovered this morning, which the model cannot learn until it retrains | Two systems that can disagree, so you need a documented precedence order |

!!! warning "When not to add the streaming aggregate store"

    If your fraud is single-shot account takeover with no velocity signature, short-window counts add nothing and you have taken on a streaming job with lag alerts and a skew risk. The measurement that justifies it is a lift analysis showing 60-second counts moving the precision-recall curve at your operating point. Cheaper alternative: a per-card counter in the existing key-value store with a time-to-live, which covers the crudest testing for one line of code.

### Breaking point, then V2 (fraud detection)

**What fails: the model is trained on labels that are half missing and one attack generation old, against an adversary with a 21-day feature half-life.**

The collision is arithmetic. Labels reach 50 percent maturity at day 30. Precision at a fixed threshold halves every 21 days. So by the time a training set is trustworthy, the attack it describes has been superseded, and a model trained only on mature labels launches at roughly half of the precision it would have had on fresh data.

How you observe it: plot the fraud rate among transactions the model scored in its lowest decile, by transaction date, as chargebacks arrive. A rising low-score fraud rate for recent cohorts, while the model's own score distribution is unchanged, is the signature of the adversary having moved. A shifting score distribution with a flat low-score fraud rate is the opposite problem, an upstream data change.

V2 rebuilds the label pipeline and adds an entity layer:

```
+---------------+   +------------------+
| Decisions log | ->| Label joiner     |
+---------------+   | maturity weights |
                    +------------------+
+---------------+          ^  |
| Chargebacks   | ---------+  v
| day 1 to 120  |     +------------------+
+---------------+     | Training set     |
                      | with weights     |
+---------------+     +------------------+
| Analyst calls |          ^   |
| hours         | ---------+   v
+---------------+     +------------------+
                      | Champion vs      |
                      | challenger       |
                      +------------------+
```

**Caption.** V2 joins the decision log to three label sources arriving at three different delays, weights each row by how mature its label is, and runs every new model as a challenger in shadow before it takes traffic.

!!! warning "When not to build the maturity-weighted label pipeline"

    If your ground truth arrives within a day, as it does for user-reported account takeover, weighting adds machinery that solves a problem you do not have. The measurement that justifies it is the fraction of labels still unresolved at your retrain horizon exceeding roughly 20 percent. Cheaper alternative: retrain on data older than your label horizon, and accept a fixed staleness you can state.

### Deep dive one: training a model whose ground truth arrives 30 to 120 days late

This is the dive because it is what makes fraud different from every other classification problem on this page. In ranking, the label arrives in seconds. Here it arrives after the attack has changed.

**The maturity curve, stated as an assumption you would measure.** Let `m(d)` be the fraction of a cohort's eventual chargebacks observed by day `d`. Assume `m(30) = 0.50`, `m(60) = 0.85`, `m(120) = 0.99`. You measure this once from historical cohorts and then it is a property of your business, not a guess.

**What goes wrong if you ignore it.** A transaction from 20 days ago with no chargeback is labelled negative. Roughly 40 percent of the fraud in that window has not surfaced yet, so a meaningful share of your recent negatives are actually positives. The model learns that recent-looking traffic is safer than it is, and it under-predicts fraud on exactly the traffic you most need it to catch.

**Three fixes, with what each costs:**

| Approach | Mechanism | Cost | When it is right |
|---|---|---|---|
| Truncate | Train only on transactions older than 120 days | The model is four months stale at launch, which is nearly six feature half-lives | Only when the adversary is slow, for example first-party friendly fraud |
| Maturity weighting | An unlabelled transaction from day `d` contributes as a negative with weight `m(d)`, and the residual mass is treated as unknown | One extra column, and one assumption: that `m(d)` does not depend on features | The honest default, and it is what you should propose first |
| Joint delay model | Predict `P(fraud)` and `P(observed by day d given fraud)` together, so recent rows contribute correct partial credit | A second head, a second thing to calibrate, and a harder failure mode when the delay distribution shifts | When high-value fraud is disputed much faster than low-value fraud, which breaks the weighting assumption |

The weighting assumption is worth attacking out loud, because it is usually false. A 2,000 dollar fraudulent charge is noticed and disputed within days. A 12 dollar one may never be. So `m(d)` depends on transaction value, which means the weighting under-counts small fraud and your model learns to care less about it than it should. Fitting `m(d)` per value bucket costs almost nothing and fixes most of it.

**Fast proxy labels, and their bias.** Three signals arrive before chargebacks:

| Signal | Delay | Bias |
|---|---|---|
| Failed step-up challenge | Seconds | Only exists for transactions you chose to challenge, so it is censored by your own policy |
| Analyst decision on a queued case | Hours | Only exists for cases the rules or model queued, so it says nothing about what you confidently approved |
| Customer report of an unrecognised charge | Days | Skewed toward attentive customers and larger amounts |

Use these as auxiliary training targets, never as the primary one. A model trained to predict "will an analyst call this fraud" learns your queueing policy, not fraud.

**Evaluating under delay, which is the part candidates never raise.** You cannot measure today's model's recall today. What you can do is state, in advance, what each horizon lets you decide:

| Horizon | What is knowable | What decision it supports |
|---|---|---|
| Day 0 | Score distribution, decline rate, challenge rate, latency | Roll back on an operational regression, immediately |
| Day 1 | Analyst-slice precision, challenge failure rate | Roll back on an obvious quality regression |
| Day 7 | Early chargebacks, roughly 15 percent mature | Detect a large recall loss, not a small one |
| Day 30 | 50 percent mature | Compare champion and challenger with wide but usable bands |
| Day 120 | 99 percent mature | The only number you would publish |

With 40,000 fraud cases per day, a single day's cohort at full maturity gives ample statistical power. Power is not the problem. Calendar time is. Design the rollout so the day-0 and day-1 checks are strong enough to catch the failures you cannot afford to run for a month.

!!! warning "When not to run champion and challenger in shadow"

    Shadow scoring doubles your inference cost and gives you no counterfactual for declines, because the challenger's declines were never actually declined. The measurement that justifies it is a challenger whose offline score distribution differs enough from the champion's that you cannot predict its decline rate. Cheaper alternative: a 1 percent live holdout on the champion, which gives you a real counterfactual on a small population.

### Deep dive two: adversarial adaptation, feature half-life, and the cost of being probed

The dives differ on purpose. The first treats time as a delay. This one treats time as an opponent.

**Feature half-life is measurable, and almost nobody measures it.** Freeze a model version. Keep scoring with it in shadow at a fixed threshold. Plot its precision against calendar time. Define half-life as the days until precision at that fixed threshold halves.

| Feature family | Assumed measured half-life | Design consequence |
|---|---|---|
| Device fingerprint and network identifiers | 21 days | Cheap to replace, so put them in a fast layer that retrains weekly and expect to churn them |
| Behavioural velocity counts | 60 days | Retrain monthly, and keep the window definitions stable so the aggregate store does not need rebuilding |
| Transaction amount, currency, hour of day | 400 days | Slow and stable, so let these carry the base model and rarely touch them |

These numbers are assumptions, stated as illustrations. The transferable part is the method and the conclusion it forces: split the model into a slow base that changes quarterly and a fast layer that changes weekly, because a single monolithic model must retrain at the pace of its fastest-decaying feature, and that is expensive.

**Every decline is a bit of information you hand the attacker.** A card tester probing a threshold learns its position in roughly `log2` of the range in attempts. Three countermeasures, each with a price:

| Countermeasure | Mechanism | Price |
|---|---|---|
| Randomise at the margin | In a narrow score band, decide stochastically | You knowingly approve some fraud |
| Rate limit per entity | Cap attempts per card, device, IP prefix and email domain, regardless of score | False positives on shared infrastructure, for example a corporate network |
| Obscure the reason | Return one generic decline reason to the merchant | Genuine customers and honest merchants lose useful diagnostics |

Price the randomisation, because interviewers ask. Assume the randomised band carries 2 percent of fraud and you randomise 0.5 percent of decisions in it. That is roughly 800 extra fraudulent approvals per day at 60 dollars, so 48,000 dollars per day. That is 2.4 percent of the 2.04 million daily net benefit. State the number, then say it is worth it only if you can show probing exists, and that the detector for probing is a per-entity attempt distribution with a long flat tail.

**Rings are invisible to per-transaction scoring.** Build a graph over shared attributes: card, device, IP prefix, shipping address, email domain, billing name.

| Design choice | Number | Consequence |
|---|---|---|
| Window length | 7 days | 840 million edges at 27 GB, so one memory-resident graph, no distributed engine |
| Attribute degree cap | Skip attributes appearing on more than 10,000 transactions | Prevents a shared corporate IP or a popular email domain from linking the entire graph into one component |
| Signal | Connected component size, and its growth rate | A component that triples in a day is the alert, not its absolute size |
| Feature fed to the model | Component size, component fraud rate over the last 7 days, and the member's distance from the nearest confirmed-fraud node | These are entity features, so they must be computed in the streaming path and served in under 10 ms |

The degree cap deserves the same treatment as the threshold formula in case study 5: without it, one shared attribute merges everything into a single component and the feature becomes constant. That is the classic failure of graph features in fraud, and it is a one-line fix that nobody mentions.

**The counterfactual hole.** You never observe the outcome of a transaction you declined, so your training data describes only the region you approve. Every model you train inherits the previous model's blind spot. The fix is an exploration budget, sized above at 300 approvals per day, costing 10,800 dollars. Draw the sample from the declined region with probability proportional to score uncertainty, not uniformly, so you spend it near the boundary where it teaches you the most.

!!! warning "When not to build the entity graph"

    If confirmed fraud cases rarely share attributes, the graph is a second data pipeline producing a near-constant feature. The measurement that justifies it is the share of confirmed fraud whose attributes touch another confirmed fraud case within the window, which you can compute retrospectively in an afternoon. Below roughly 10 percent, do not build it. Cheaper alternative: a denylist of confirmed-fraud attributes with a time-to-live.

### Failure modes and evaluation (fraud detection)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Selection bias loop | The model declines a region, never sees its outcomes, and the next model trained on that data declines it harder for no evidence | The exploration budget, sampled by uncertainty, is the only structural fix |
| Retry storm | A merchant retries a declined authorisation three times, so one fraud attempt becomes four decisions and four data points | Deduplicate by request fingerprint before scoring, and count retries as one event in every velocity aggregate |
| Metastable analyst queue | A peak event fills the case queue, decisions slow, more transactions time out into the fallback, the fallback approves, fraud rises, more cases queue | Cap the queue by shedding the lowest-value cases first, with the value threshold agreed in advance, not chosen during the incident |
| Fail-open versus fail-closed | The feature service times out and the design must decide without features | Choose per amount band: fail open below a stated value, fail closed above it, and rehearse it. A single global choice is wrong at one end or the other |
| Training and serving skew on windows | The offline job computes a 60-second count on event time, the online store computes it on arrival time, and the two differ during lag | Compute both from the same aggregate definition, and add a daily job that scores yesterday's traffic offline and compares to the logged online score |
| Hot key in the feature store | A popular card issuer prefix or a shared device identifier is read on a large share of requests | Cache the top keys locally with a short time-to-live, and cap per-key load |
| Rule shadowing the model | A broad rule declines everything in a region, so the model never receives training data there and appears to have no signal | Log the model score even when a rule decides, and evaluate the model on rule-decided traffic separately |
| Clock and timezone drift on windows | A daylight-saving transition makes the 30-day aggregate cover 29 or 31 days for one cohort | Compute all windows in a single monotonic time base, never in local time |

Service level indicators:

- Decision latency at p99, and the fallback rate, which is the metric that tells you how often the system is not actually deciding.
- Decline rate and challenge rate, per merchant category, as leading indicators that move before any chargeback arrives.
- Score distribution stability, measured as population stability between today and a fixed reference week.
- Fraud rate in the lowest score decile, by transaction cohort date, as chargebacks mature. This is the leading indicator of adversarial movement.
- Challenge abandonment rate, which is the conversion cost of the challenge tier.
- Feature freshness age at p99 for each streaming aggregate.
- Exploration coverage: the number of approved-for-exploration transactions per day and their realised fraud rate.

Alert on the fallback rate, on population stability crossing its band, and on any velocity aggregate's freshness exceeding its window length, which means the feature is silently wrong rather than missing. Do not alert on daily chargeback counts, because at 30-day maturity today's number describes a model you replaced twice.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| p99 under 150 ms | Yes: one batched feature read, one gradient-boosted scoring at microseconds, and rules evaluated in memory |
| 1,852 decisions per second at peak | Yes, and comfortably. Throughput was never the constraint here |
| 99.99 percent availability | Yes, given an explicit per-amount-band fallback that is exercised in a game day, not merely documented |
| Total loss minimised | Yes at the 90 percent recall, 60 percent precision point, which the estimation table showed beats both neighbours by more than 400,000 dollars per day |
| Rule deployable in minutes | Yes: rules live outside the model release path, with the precedence order documented |
| Auditable reason per decision | Yes, including the model score even when a rule made the call, which is what makes rule shadowing detectable |

### Where this answer is a simplification (fraud detection)

All links in this list were checked on 1 August 2026.

- **The model is the easy part.** Most of the working time in a real fraud team goes to label plumbing, case management tooling and the argument about who owns the decline rate. The interview rewards saying that.
- **Dal Pozzolo and colleagues, "Credit Card Fraud Detection: A Realistic Modeling and a Novel Learning Strategy"** (IEEE Transactions on Neural Networks and Learning Systems, 2018) is the primary academic treatment of verification latency and the delayed-label setting used in deep dive one.
- **Chapelle, "Modeling Delayed Feedback in Display Advertising"** (KDD, 2014) formalises the joint delay model, and although it was written for conversions rather than chargebacks, the mathematics transfers directly.
- **Ktena and colleagues, "Addressing Delayed Feedback for Continuous Training with Neural Networks in CTR Prediction"** (RecSys, 2019) compares the practical loss functions for continuous training under delay, which is the version of this problem you actually implement.
- **Bottou and colleagues, "Counterfactual Reasoning and Learning Systems"** (Journal of Machine Learning Research, 2013) is the reference for why the exploration budget is not optional when your policy censors its own training data.
- **[EMV 3-D Secure](https://www.emvco.com/emv-technologies/3d-secure/)** (EMVCo, specifications available royalty-free) is the public standard behind the challenge tier, and it is worth reading because the liability shift it creates changes the economics of challenging in ways no model can capture.
- **Regulatory strong-customer-authentication regimes**, such as the EU's PSD2 requirements applicable since 2019, mandate challenges on some transactions and exempt others, which means part of your challenge budget is spent for you.

### Level delta (fraud detection)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Proposes a classifier and discusses class imbalance | Asks who bears the loss, because it decides the objective | Asks when the label arrives, because the answer reorganises the entire design |
| Operating point | "Optimise for recall, fraud is expensive" | Picks a threshold from a precision-recall curve | Prices three operating points in dollars and shows the optimum is interior, in four lines of arithmetic |
| Throughput | Discusses scaling the scoring fleet | Notes that 1,852 per second is not hard | Says explicitly that throughput is not the interesting problem here, and spends the reclaimed time on labels |
| Labels | Trains on chargebacks | Notes chargebacks are delayed | States a maturity curve, weights rows by it, attacks the weighting assumption on transaction value, and fixes it per value bucket |
| Adversary | "Fraudsters adapt" | Retrains more often | Defines feature half-life as a measurable quantity, splits the model into slow and fast layers, and prices the randomised band at 48,000 dollars per day |
| Rings | Not raised | Adds shared-attribute features | Sizes the graph at 27 GB for a 7-day window to rule out a distributed engine, and adds the degree cap that stops the graph collapsing into one component |
| Counterfactuals | Not raised | Mentions that declines are unlabelled | Budgets 300 exploration approvals per day at 10,800 dollars, sampled by uncertainty rather than uniformly |
| Evaluation | Reports offline area under the curve | Adds an online holdout | Publishes a horizon table stating what is decidable at day 0, 1, 7, 30 and 120, and designs the rollout around the early rows |
| Failure handling | Retries on error | Adds a timeout | Chooses fail-open below an amount band and fail-closed above it, and says the choice must be rehearsed |

What each level added:

- **Mid to senior:** the objective becomes an economic one, and the design acknowledges that both error types cost money.
- **Senior to staff:** time becomes the organising constraint, delay and adaptation are quantified rather than named, and the candidate designs the measurement plan around what is knowable at each horizon rather than around what is convenient.

---

## Case study 7: Delivery time prediction

### The prompt (delivery time prediction)

"Design the system that tells a customer when their food delivery will arrive, and keeps that number updated until it does."

### Questions that fork the design (delivery time prediction)

| Question worth asking | If the answer is A the design becomes | If the answer is B it becomes |
|---|---|---|
| Is the number a prediction or a promise? | A prediction: minimise error, and a symmetric loss is defensible | A promise backed by refunds: you are choosing a quantile, not estimating a mean, and the number changes behaviour on both sides of the marketplace |
| Is it shown before or after the order is placed? | After: it is a status display, and volume is one number per order per update | Before, on a browse page: it enters ranking and conversion, and the volume is thirty numbers per page view, which is a different service |
| One number or a range? | A point estimate: sharper accountability, worse perceived reliability | A window: you are picking two quantiles, and the width is a product decision about how much uncertainty to admit |
| Do we control assignment and routing? | Yes: the ETA and the dispatch decision are one joint optimisation, and improving dispatch improves the ETA for free | No, a third-party courier: the ETA is a forecast of somebody else's process and your feature set collapses |
| Is preparation time observable? | Yes, the merchant marks the order ready: two models with a clean handoff, each separately measurable | No: preparation is a latent variable, it dominates the error, and most of your modelling effort goes there |
| How often is the number updated? | Once, at order time: one inference per order | Continuously: twenty inferences per order, plus a hard requirement that the number never jumps backwards |
| What is the cost asymmetry of early versus late? | Symmetric: absolute error is the right training objective | Late costs several times early: the optimal display is a high quantile, derived rather than chosen |
| Does the shown number affect the outcome? | No: a clean supervised problem | Yes, couriers pace to it and customers cancel on it: your training labels are contaminated by your own predictions, which is deep dive one |

### Requirements (delivery time prediction)

Functional:

- Return an estimated delivery time for a merchant before the customer orders.
- Return and refresh an estimated delivery time after the order is placed, through preparation, pickup and transit.
- Never present a later-then-earlier sequence that reads as the promise moving backwards.
- Fall back to a defensible number when a feature is missing, and record that the fallback was used.
- Record, for every order, the number shown at each moment and the number that came true.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Orders per day | 5 million | ASSUMED; ask for it, because it determines almost nothing here and the browse volume determines almost everything |
| Share of orders in the two-hour dinner peak | 25 percent | ASSUMED; food ordering is far more concentrated in time than most consumer traffic |
| Browse sessions per day | 20 million, each rendering 30 merchant ETAs | ASSUMED; browsing exceeds ordering by several times on any marketplace |
| Post-order ETA updates per order | 20 | ASSUMED; one every 30 seconds across a ten-minute active window, plus event-triggered refreshes |
| Pre-order ETA latency | under 100 ms at p99 for a full page of 30 | ASSUMED; it renders inside a list, so it must not be the slowest element on the page |
| Accuracy target | 80 percent of orders within 5 minutes of the last shown ETA | ASSUMED; this is a product target and is the right thing to negotiate |
| Late rate | no more than 10 percent of orders more than 10 minutes past the shown ETA | ASSUMED; the tail is what generates support contacts, not the mean |
| Cost asymmetry | one minute late costs four times one minute early | ASSUMED; it comes from the product team, and it is the single input that decides the quantile |
| Peak order rate | 174 per second | DERIVED below |
| Peak pre-order ETA rate | 20,833 per second | DERIVED below |

### Estimation that eliminates (delivery time prediction)

| Calculation | Result | What it rules out |
|---|---|---|
| 25 percent of 5e6 divided by 7,200 seconds | 174 orders per second at the dinner peak | Rules out treating the order path as a scaling problem. It is small |
| 174 orders per second times 20 updates | 3,472 post-order inferences per second | Rules out fearing the update frequency. At an assumed 2 ms of processor time per inference that is 7 cores, so update as often as the product wants |
| 25 percent of 20e6 sessions times 30 merchants, divided by 7,200 | 20,833 pre-order ETAs per second at peak | Rules out one ETA service. The pre-order path is six times the post-order path and must be designed separately |
| 20,833 per second times an assumed 20 ms of routing-engine processor time | 417 cores of routing on the browse path alone | Rules out calling a road-graph router per merchant on browse. Browse gets a coarse zone estimate; the precise route is computed once, after the order exists |
| 5e6 orders times 20 snapshots at 200 bytes | 100 million training rows and 20 GB per day, 7.3 TB per year | Rules out retaining every snapshot at full fidelity forever. Keep all rows for 90 days, then keep one row per order |
| Feedback coefficient of 0.3, so total inflation is 1 divided by (1 minus 0.3) | 1.43x amplification, so a one-off two-minute pad becomes a permanent 2.86-minute inflation | Rules out retraining naively on observed delivery times. This is the number that forces deep dive one |
| Cost asymmetry of 4, so the optimal quantile is 4 divided by (4 plus 1) | Show the 80th percentile, not the mean | Rules out training on mean absolute error and displaying the point estimate |
| Assumed predicted distribution with p50 at 32 minutes and p80 at 41 minutes | A 9-minute gap between the typical experience and the promise | Rules out using one quantile everywhere. Nine minutes of padding on a browse page costs conversion, so pre-order and post-order need different quantiles |
| A merchant with 5 historical orders versus one with 5,000 | Roughly a 30x difference in the standard error of the mean, since it scales with the square root of the count | Rules out a single global spread. The model must predict its own uncertainty per order |

### V0, the simplest thing that works (delivery time prediction)

**Predict first: when does a lookup table of historical medians beat a trained model?**

??? note "Show answer"

    When the cells are thin. A model buys you the ability to condition on order size, weather and courier supply, but it pays for that with variance. Below roughly 50 completed orders per cell, the model's variance exceeds the bias it removes, and the median lookup wins.

    With 2,000 merchants, 168 hour-of-week buckets and 3 distance buckets you have about 1 million cells and 5 million orders per day, so most cells are fine and the thin ones are new merchants. That tells you the first thing to fix is cold start, not the model class.

```
+---------+   +---------------------+   +------------+
| Order   |-->| Lookup table:       |-->| Shown ETA  |
| request |   | median minutes by   |   +------------+
+---------+   | merchant, daypart,  |
              | distance bucket     |
              +---------------------+
                        ^
              +---------------------+
              | Nightly batch over  |
              | last 30 days        |
              +---------------------+
```

**Caption.** V0 looks up the median observed delivery time for this merchant, this part of the week and this distance band, from a table rebuilt nightly over the last 30 days of completed orders.

Why this is correct at the stated smaller scale:

- The median is robust to the long right tail that wrecks a mean, and the tail here is entirely real.
- Every prediction is explainable in one sentence to a support agent, which matters more than three minutes of accuracy.
- There is no feature service, no training pipeline, no serving skew and no online-offline consistency problem.
- It is trivially cacheable, which is what makes 20,833 pre-order lookups per second free rather than 417 cores.

!!! warning "When not to move off the lookup table"

    If your residuals do not correlate with any feature you already have, a model cannot help you and you are about to build a pipeline for nothing. The measurement that justifies the move is a plot of residual against a candidate feature, for example order item count, showing a visible slope. Cheaper alternative: add one more dimension to the lookup table, for example order size band, and check whether the residual slope flattens.

### Breaking point, then V1 (delivery time prediction)

**What fails: the lookup table has no per-order information, so its error is wide exactly when the order is unusual.**

How you observe it: bucket completed orders by a feature the table ignores, such as item count, and plot mean residual per bucket. A flat line means the table is capturing what matters. A sloped line, for example a 12-item order arriving 9 minutes later than predicted on average, is money left on the table. Assume the table achieves a median absolute error of 8 minutes against a target of 5 minutes at the 80th percentile.

The second observable is the composition of the error. Split each completed order into placed-to-ready and ready-to-door, and compute the variance of each. If preparation carries most of the variance, a better routing model will not move the product metric at all, and knowing that before you build is the whole value of the split.

V1 splits the problem in two and models each half:

```
+------------+   +------------------+
| Order      |-->| Prep model       |---+
| context    |   | placed to ready  |   |
+------------+   +------------------+   |
                                        v
+------------+   +------------------+  +------------------+
| Courier    |-->| Transit model    |->| Combine, pick    |
| road state |   | ready to door    |  | quantile, smooth |
+------------+   +------------------+  +------------------+
       ^                                        |
+------------------+                            v
| Feature service  |                       shown ETA
| live + batch     |
+------------------+
```

**Caption.** V1 predicts preparation time and transit time separately, from different feature sets, then combines the two distributions and applies a display policy that picks a quantile and smooths the sequence.

The split is not decoration. It earns its place three ways:

| Reason | Consequence |
|---|---|
| Different features | Preparation depends on the merchant, the basket and the merchant's current queue. Transit depends on distance, road speed and courier supply. Sharing a feature vector wastes both |
| Different observability | The merchant marks ready, so preparation has a clean label. Transit has a clean label too. One end-to-end model has one label and cannot tell you which half is wrong |
| Different failure attribution | When the ETA is bad, you need to know which half moved. A single model gives you a residual and no diagnosis |

!!! warning "When not to split into two models"

    If the merchant does not report a ready time, the split is unmeasurable and you have two models with one label between them. The measurement that justifies the split is coverage of the ready-time event above roughly 80 percent of orders. Cheaper alternative: one model with a merchant-queue feature, plus a manual audit of a few hundred orders to estimate the variance split before you invest.

### Breaking point, then V2 (delivery time prediction)

**What fails: the prediction changes the outcome, so retraining on observed outcomes inflates the ETA without bound, and the displayed number jumps backwards.**

Two failures, both invisible to standard offline metrics.

The first is the feedback loop. Couriers pace to the shown ETA, customers tolerate what they were told, and dispatch may prioritise orders by how close they are to their promise. So the observed delivery time is partly a function of the number you displayed. Retrain on it and you learn your own padding.

How you observe it: keep a slice of orders whose displayed ETA comes from a frozen policy, and track the gap between the production model's prediction and the frozen slice's realised times across model generations. A widening gap with no change in road speed or courier supply is inflation, not seasonality.

The second is monotonicity. A number that reads 31 minutes, then 38, then 34, is experienced as a broken promise even when the final delivery lands at 34. Offline error metrics score that sequence perfectly.

How you observe it: count monotonicity violations per order, defined as any upward revision beyond a tolerance, and correlate them with support contacts.

V2 separates the model from the display and protects the training data:

```
+------------------+   +------------------+
| Model: predicts  |-->| Display policy   |--> shown ETA
| a distribution   |   | quantile + floor |
+------------------+   +------------------+
        ^                       |
        |                       v
+------------------+   +------------------+
| Training set     |<--| Frozen-policy    |
| physical targets |   | control slice    |
| only             |   +------------------+
+------------------+
```

**Caption.** V2 makes the model predict a distribution over physical outcomes, moves every product adjustment into a display policy that never writes back into training, and keeps a control slice whose displayed number is set by a frozen policy so the loop can be measured.

!!! warning "When not to build the frozen-policy control slice"

    If the shown number cannot influence the outcome, for example an ETA for a flight you do not operate, the control slice costs you accuracy on a real slice of users for no information. The measurement that justifies it is a switchback experiment showing a non-zero effect of shown ETA on realised time. Cheaper alternative: run that switchback once a quarter and skip the permanent slice if the effect is indistinguishable from zero.

### Deep dive one: the prediction changes the outcome, and how to bound the loop

This dive exists because it is the thing that makes ETA different from every other regression problem. In most supervised settings the label is fixed before you predict. Here it is not.

**The arithmetic of runaway inflation.** Assume realised time responds to the shown time with coefficient `k`:

`actual = true + k times (shown - true)`

Suppose a well-meaning engineer adds a pad of `p` minutes once, to reduce the late rate. Then:

| Generation | Shown | Realised |
|---|---|---|
| 1 | `true + p` | `true + k p` |
| 2 | `true + p + k p` | `true + k p + k^2 p` |
| n | `true + p times (1 + k + ... + k^(n-1))` | approaches `true + k p / (1 - k)` |

The shown ETA converges to `true + p / (1 - k)`. At `k = 0.3` that is 1.43 times the original pad, so a one-off two-minute pad becomes a permanent 2.86-minute inflation, and nobody who added the pad is still on the team when it is noticed.

**Measuring `k` honestly requires an experiment, not a regression.** Regressing realised time on shown time gives you confounding: busy evenings produce both long ETAs and long deliveries. The clean design is a switchback that randomises a constant offset at the city-hour level.

Power sizing, so you can state the cost:

- Effect to detect: 0.6 minutes of realised change from a 2-minute offset, which is `k = 0.3`.
- Randomising at the city-hour level means the unit of analysis is the city-hour, not the order, so what matters is the between-unit standard deviation. Assume 1.5 minutes.
- `n = 2 times (1.96 + 0.84)^2 times 1.5^2 / 0.6^2 = 2 times 7.84 times 2.25 / 0.36 = 98` units per arm.
- About 200 city-hours in total. Across 40 cities running one dinner block each evening, that is five evenings.

Five evenings is cheap, and being able to say that out loud converts "we should measure the feedback loop" from a platitude into a plan.

**Three containment mechanisms, in order of preference:**

| Mechanism | What it does | Cost |
|---|---|---|
| Train on physical targets | The transit model learns road speed from courier location traces, which the displayed ETA cannot influence, rather than learning end-to-end delivery time | You lose the parts of the signal that only appear end to end, for example handoff delays at large apartment buildings |
| Separate model from display | The model outputs a distribution. The display policy picks a quantile, applies a floor and smooths. Only the model output enters the training set | An extra layer, and a discipline that must be enforced in code review rather than by convention |
| Frozen-policy control slice | A small share of orders get their number from a policy that never changes, giving an uncontaminated reference | Those users get a worse number, so keep the slice small and rotate which cities carry it |

The second mechanism is the architectural point, and it is stated as a rule: **padding lives in the display layer, and the display layer never writes back into training.** Once a product adjustment enters the training set, it is indistinguishable from the world, and no amount of retraining will remove it.

**Smoothing, without lying.** The display policy needs to avoid backwards jumps without freezing a number it knows is wrong. A workable rule:

- Never revise later by more than 2 minutes in a single update.
- Allow unlimited revision earlier, because arriving sooner than promised is the cheap error.
- If the model's new estimate is more than 10 minutes later than the shown number, stop smoothing, show the truth, and trigger a proactive notification, because at that magnitude the customer needs the information more than they need consistency.

Each of those three lines is a product decision with a measurable cost, and naming them as such is what separates a display policy from a hack.

!!! warning "When not to smooth at all"

    If the ETA is shown once and never updated, smoothing has nothing to smooth. The measurement that justifies it is the monotonicity violation rate correlating with support contacts or cancellations. Cheaper alternative: widen the displayed window so ordinary revisions stay inside it, which is honest and costs no machinery.

### Deep dive two: choosing the quantile, and why calibration beats accuracy here

The dives differ because the first treats the prediction as an intervention and this one treats it as a distribution. Both are about the gap between what the model computes and what the customer sees.

**The optimal quantile falls straight out of the cost ratio.** Under the pinball loss with cost `c_late` per minute late and `c_early` per minute early, the minimising prediction is the quantile at:

`q = c_late / (c_late + c_early)`

With the assumed asymmetry of 4, `q = 0.80`. So the correct display is the 80th percentile of the predicted distribution, and training on mean absolute error, which targets the median, is systematically wrong by the p50-to-p80 gap.

**But the right quantile differs by surface, and this is the interesting part.**

| Surface | What a longer number costs | What a shorter number costs | Quantile |
|---|---|---|---|
| Browse page, before ordering | Conversion. A merchant shown at 45 minutes loses orders to one shown at 32 | Nothing yet, because no promise has been made | Lower, for example 0.5, because the number is comparative rather than contractual |
| Order confirmation | Cancellation at the moment of commitment | The promise is now made and lateness is charged against it | The stated 0.80 |
| Live tracking, after pickup | Very little, because the customer is already waiting | A visible broken promise | Higher, for example 0.9, because the distribution has narrowed so the pad in minutes is small anyway |

That table is the answer to "one ETA service or two". It is two, because they optimise different things from the same underlying distribution.

**Calibration is a different property from accuracy, and it is the one you must alert on.** If you display the 80th percentile, then exactly 20 percent of orders should be late. Measure empirical coverage:

| Observed late rate at the displayed p80 | Diagnosis | Action |
|---|---|---|
| 20 percent | Calibrated | Improve sharpness, meaning narrow the distribution, which is a modelling problem |
| 26 percent | Under-dispersed in the tail | The model's uncertainty estimate is too narrow. More features will not fix this |
| 12 percent | Over-dispersed | You are padding, and paying conversion for it |

A model can have excellent mean absolute error and terrible coverage, and the product metric you promised, the late rate, depends only on coverage. Alerting on error and not on coverage is the most common instrumentation mistake in this problem.

**Predict the spread per order, not globally.** The uncertainty in a delivery is not constant:

| Situation | Why the spread is different | If you use one global spread |
|---|---|---|
| Merchant with 5 historical orders | The standard error of the mean scales with the inverse square root of the count, so a 5-order merchant has roughly 30 times the uncertainty of a 5,000-order merchant | You under-promise on new merchants and they get systematically late |
| Heavy rain, courier supply below demand | The right tail of transit lengthens sharply | You miss exactly the conditions that generate support contacts |
| A 2-item order at a fast merchant at 3pm | Genuinely narrow | You over-pad, and lose conversion for nothing |

The implementation is a heteroscedastic head, or quantile regression at several levels, or an ensemble whose spread you use directly. All three are defensible. What is not defensible is a global constant, because it fails in opposite directions at the two ends of the merchant distribution.

**Cold start, which the estimation section already pointed at.** A new merchant has no history, so its distribution must be borrowed. Borrow from a hierarchy: this merchant, then this cuisine and this city, then this city. Shrink toward the parent by the count, which is a one-line empirical Bayes update and needs no new infrastructure.

!!! warning "When not to model the spread per order"

    If your product shows a single point estimate with no promise attached and no refund policy, the spread is unused and predicting it is wasted capacity. The measurement that justifies it is a coverage plot that is well calibrated on average but badly miscalibrated within merchant-count deciles. Cheaper alternative: two global spreads, one for merchants above a history threshold and one below.

### Failure modes and evaluation (delivery time prediction)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Feedback contamination | Predictions inflate generation over generation with no change in the physical world | Physical training targets, display-layer padding, and a frozen-policy control slice |
| Correlated feature loss | Courier location updates drop out during exactly the network congestion that also slows traffic, so the fallback path is exercised precisely when it is least accurate | Measure fallback accuracy separately from model accuracy, and alert on fallback rate rather than assuming the fallback is fine |
| Training and serving skew on time features | The batch job computes hour-of-day in one time base and the serving path uses another, so every prediction is shifted for part of the world | Derive all time features from one shared function, and add a daily replay that scores yesterday's requests offline and compares to the logged online prediction |
| Thundering herd on recomputation | A city-wide traffic feature updates and every active order is recomputed within the same second | Stagger recomputation with a jittered per-order schedule, and treat feature updates as a hint rather than a trigger |
| Cold start | A new merchant gets the city average and is systematically late, generating bad early reviews that never recover | Hierarchical shrinkage plus a wider displayed window while the count is low |
| Stale road state during an incident | A closure makes the routing feature confidently wrong, and the model trusts it | Monitor residual mean by city as a time series, and alert on it crossing a drift band, which separates "the model is wrong" from "the world changed" |
| Monotonicity violation | The number moves backwards and the customer reads it as a broken promise | The display policy's revision limits, with the violation rate as a first-class SLI |
| Label leakage through the ready event | A merchant marks orders ready in bulk at the end of a shift, so the preparation label is fiction for that merchant | Detect bulk-marking by the interval distribution, and exclude those merchants' preparation labels rather than training on them |

Service level indicators:

- Coverage at the displayed quantile, meaning the realised late rate against the promised 20 percent, segmented by city and by merchant history decile.
- Absolute error at p50 and p90, reported separately for preparation and transit so failures are attributable.
- Late rate beyond 10 minutes, which is the requirement, not a proxy for it.
- Monotonicity violations per order.
- Fallback rate and fallback accuracy.
- Feature freshness at p99 for road state and courier supply.
- The gap between production predictions and the frozen-policy control slice, tracked per model generation.

Alert on coverage drifting outside its band, on fallback rate, and on residual mean by city crossing a drift threshold. Do not page on mean absolute error, because it moves with the weather and will train the on-call to ignore it.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| 20,833 pre-order ETAs per second at p99 under 100 ms | Yes, because browse reads a cached zone-level estimate rather than calling a router, which was the 417-core line in the estimation table |
| 3,472 post-order inferences per second | Yes, at roughly 7 cores, so update frequency is a product choice rather than a cost constraint |
| 80 percent within 5 minutes | Achievable only if the sharpness improves. The design guarantees calibration, not sharpness, and saying that distinction out loud is the honest answer |
| Late rate under 10 percent | Yes by construction if coverage holds, since the displayed quantile is chosen to be 0.80 and coverage is the alerted metric |
| Never moves backwards visibly | Yes, via the display policy's revision limits, with the escape hatch above 10 minutes |
| Every shown number recorded | Yes, and this is what makes the feedback loop measurable at all |

### Where this answer is a simplification (delivery time prediction)

All links in this list were checked on 1 August 2026.

- **Dispatch and the ETA are one system.** Everything above forecasts a process that a different team's assignment algorithm controls. The largest available accuracy gain is usually a better assignment, not a better forecast, and a staff answer says so.
- **Derrow-Pinion and colleagues, ["ETA Prediction with Graph Neural Networks in Google Maps"](https://arxiv.org/abs/2108.11482)** (CIKM, 2021) is the primary published account of large-scale road-network travel-time prediction, and it is the right reference for the transit half of this design.
- **Uber Engineering, ["DeepETA: How Uber Predicts Arrival Times Using Deep Learning"](https://www.uber.com/blog/deepeta-how-uber-predicts-arrival-times/)** (uber.com/blog, 10 February 2022) documents a production architecture that predicts the residual between a routing-engine estimate and the observed outcome, which is a structure worth contrasting with the two-model split here.
- **Koenker and Bassett, "Regression Quantiles"** (Econometrica, 1978) is the origin of the pinball loss used to derive the 0.80 quantile, and it is short.
- **Gneiting and Raftery, "Strictly Proper Scoring Rules, Prediction, and Estimation"** (Journal of the American Statistical Association, 2007) formalises the calibration-versus-sharpness distinction that the evaluation section leans on.
- **Switchback experimentation** is the standard tool for marketplace interference: randomise time-by-region blocks rather than users, because a faster ETA changes courier supply for everyone in the region. The consequence to hold on to is that your effective sample size is the number of blocks, not the number of orders, so a switchback needs far longer than the naive calculation in Module 11 suggests.
- **Bottou and colleagues, "Counterfactual Reasoning and Learning Systems"** (Journal of Machine Learning Research, 2013), cited in case study 6 for a different reason, is also the cleanest statement of why a system that acts on its own predictions cannot be evaluated from its own logs.

### Level delta (delivery time prediction)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Proposes a regression on distance and historical times | Asks whether the number is a prediction or a promise | Asks whether the shown number affects the outcome, which reorganises training, evaluation and the display layer at once |
| Service shape | One ETA service | Notes that browse and post-order have different volumes | Derives 20,833 per second and 417 routing cores to show they must be two services with two quantiles |
| Model structure | One end-to-end model | Splits preparation from transit | Justifies the split by feature sets, label availability and failure attribution, and states the coverage precondition that makes it measurable |
| Objective | Mean absolute error | Notes lateness costs more than earliness | Derives the 0.80 quantile from the stated cost ratio, then varies it by surface with a reason for each |
| Feedback loop | Not raised | Mentions that ETAs might influence behaviour | Writes the amplification as `1 / (1 - k)`, shows a 2-minute pad becomes 2.86 permanently, and sizes the switchback at 98 city-hours per arm |
| Uncertainty | A single confidence figure | Adds a per-merchant spread | Predicts spread per order, shows the 30x difference between a 5-order and a 5,000-order merchant, and shrinks hierarchically for cold start |
| Evaluation | Reports error | Adds online monitoring | Alerts on coverage rather than error, and explains that the promised late rate depends on calibration and not on accuracy |
| Display | Shows the model output | Adds smoothing | Makes the display layer a named component with a rule that it never writes back into training |

What each level added:

- **Mid to senior:** the problem stops being a regression and becomes two forecasting problems with an asymmetric loss and a real serving constraint.
- **Senior to staff:** the prediction is treated as an intervention rather than an observation, the feedback loop is quantified and bounded architecturally, and the monitoring is designed around the property the product actually promised.

---

## Case study 8: People you may know

### The prompt (people you may know)

"Design the People You May Know surface for a professional network: given a member, produce a ranked list of other members they should connect with."

### Questions that fork the design (people you may know)

| Question worth asking | If the answer is A the design becomes | If the answer is B it becomes |
|---|---|---|
| Are we optimising for connections made, or connections that turn out valuable? | Accepts: the label arrives in hours, training is easy, and you will recommend the highest-degree people to everyone | Downstream value, for example an interaction within 30 days: the label is delayed by weeks and the ranking objective becomes multi-task |
| Is contact import allowed? | Yes: the strongest cold-start signal you will ever have, and the largest privacy hazard on the page | No: pure graph signals, and members with zero connections are unservable, which changes what V0 can even do |
| Must every recommendation be explainable? | No: any signal can drive the ranking | Yes, "you both know X": you can only recommend what you can say out loud, which constrains candidate generation before it constrains ranking |
| Is it acceptable to reveal a relationship the member did not disclose? | Not a concern for this product | No: some true and useful edges must be suppressed, and the suppression must not itself be observable, which is deep dive two |
| How fresh must the graph be? | Daily: a nightly batch expansion is fine and the design is one job | Minutes: incremental triangle closing on an edge stream, and the batch job becomes a repair path rather than the main path |
| Directed or undirected edges? | Directed follows: candidate sets are asymmetric and you cannot halve the work | Undirected connections: symmetry lets you compute each pair once and cache it for both sides |
| Are there blocks, mutes and hidden connection lists? | No | Yes: hard filters that must apply at serve time on the final list, and that must not change the output in a way an observer can detect |
| How large is the largest degree? | Bounded, say under 1,000 | Unbounded, for example a recruiter with 30,000: the cost of two-hop expansion is dominated entirely by the tail, and you need a hub cap |

### Requirements (people you may know)

Functional:

- Produce a ranked list of candidate members for any member, on request.
- Exclude existing connections, previously dismissed candidates and blocked members.
- Attach a reason to each recommendation that can be shown to the member.
- Reflect a newly created connection in the recommendations of the affected neighbourhoods within a stated window.
- Serve members with zero existing connections.

Non-functional:

| Requirement | Value | Tag |
|---|---|---|
| Registered members | 900 million | ASSUMED; the interviewer usually gives this, and it drives the storage arithmetic |
| Mean connections per member | 300, heavy tailed, capped at 30,000 | ASSUMED; a cap exists on most professional networks and the tail is what costs money |
| Monthly active members | 200 million | ASSUMED; recommendations only need computing for people who will see them |
| Impressions per active member per day | 20 | ASSUMED; the surface appears on several pages |
| New connections per active member per day | 0.02 | ASSUMED; roughly seven per year, which is consistent with a mean degree of 300 over a long tenure |
| Serving latency | under 200 ms at p99 for a card of 8 | ASSUMED; it renders inside a feed |
| Freshness | a new connection changes recommendations within 24 hours | ASSUMED, and worth negotiating, because tightening it to minutes is what forces the incremental path |
| Candidates stored per member | 500 | DERIVED below from storage cost |
| Peak request rate | 17,400 per second | DERIVED below |

### Estimation that eliminates (people you may know)

| Calculation | Result | What it rules out |
|---|---|---|
| 900e6 members times 300 connections, stored as directed adjacency at 8 bytes per entry | 270 billion entries, 2.16 TB | Rules out a single machine holding the graph, and equally rules out a thousand-node graph engine. At 128 GB per node this is about 17 shards, which is a modest sharded store |
| 300 times 300 | About 90,000 raw two-hop candidates per member | Rules out ranking the full candidate set online. Ninety thousand candidates per request at 17,400 requests per second is not a serving problem, it is a fantasy |
| Sum over members of degree squared, at least 900e6 times 90,000 by Jensen's inequality | At least 81 trillion pair emissions for one full expansion | Rules out nothing yet, but see the next two rows |
| 81e12 emissions at an assumed 2 million per second per node, on 1,000 nodes | 40,500 seconds, so 11.3 hours on a thousand-node cluster | Rules out claiming the batch is infeasible. It is feasible and it costs a thousand machines every night |
| 200e6 actives times 0.02 new connections, each costing degree of u plus degree of v emissions | 4 million new edges per day times 600, so 2.4 billion emissions per day, 27,800 per second | Rules out the nightly full batch on cost. Incremental is 1,200 node-seconds per day against 40.5 million, which is four orders of magnitude |
| 200e6 actives times 90,000 candidates at 16 bytes | 288 TB | Rules out storing the full candidate set. Truncation happens at generation, not at serving |
| Same, truncated to 500 candidates | 1.6 TB | Rules out worrying about candidate storage once truncated. This is why 500 is the stored depth |
| A hub with 30,000 connections, expanded as an intermediary | 900 million emissions from one node | Rules out uncapped expansion. One member can cost more than a million ordinary members combined |
| Evidence weight of a shared connection, taken as 1 over the natural log of degree: 1/ln(30,000) against 1/ln(20) | 0.097 against 0.33, a 3.4x difference in evidence for a 1,500x difference in cost | Rules out expanding through hubs at all. This ratio is the justification for the degree cap, and it is derivable rather than asserted |
| 200e6 actives times 20 impressions, in cards of 8 | 500 million requests per day, 5,787 per second, 17,400 at 3x peak | Rules out per-request batch computation, and sets the serving budget |
| 17,400 requests times 500 candidates each | 8.7 million scorings per second | Rules out a cross-attention model at an assumed 1 ms per candidate, which would need 8,700 cores. A gradient-boosted ensemble at roughly 2 microseconds needs 17 cores |
| An assumed mean of 3 blocked members per member, at 8 bytes | 24 bytes per member | Rules out an approximate structure for the block filter. Store it exactly, because a false positive here suppresses a valid recommendation and a false negative is a policy violation |

### V0, the simplest thing that works (people you may know)

**Predict first: at what member count does a nightly full two-hop expansion stop being the right answer?**

??? note "Show answer"

    Around 10 to 20 million members. At 10 million members with a mean degree of 100, the emission count is roughly 10e6 times 10,000, which is 100 billion, or about 50,000 node-seconds. That is one machine-night on a small cluster, with no streaming state, no repair job and no incremental correctness argument. The crossover is not about member count directly; it is about when the batch's cluster cost exceeds the cost of running and repairing a streaming path.

```
+-------------+   +--------------------+   +-------------+
| Nightly     |-->| Count shared       |-->| Top 100 per |
| edge dump   |   | connections for    |   | member      |
+-------------+   | every two-hop pair |   +-------------+
                  +--------------------+          |
                                                  v
+-------------+                          +----------------+
| Member      |------------------------->| Serve, minus   |
| opens app   |                          | existing edges |
+-------------+                          +----------------+
```

**Caption.** V0 dumps the edge list nightly, counts shared connections for every two-hop pair, keeps the top 100 per member, and serves that list with existing connections filtered out at read time.

Why this is correct at the stated smaller scale:

- Shared-connection count is a genuinely strong baseline, and a ranking model typically adds less than a first-time designer expects.
- The reason is free and true: "you both know these three people" comes directly from the computation.
- One batch job means one thing to debug, and a rerun fixes any corruption.
- Filtering existing connections at read time rather than at generation time keeps the batch stateless.

!!! warning "When not to move off the nightly batch"

    Below roughly 20 million members, or whenever a 24-hour freshness window is acceptable to the product, the incremental path is over-engineering. It brings streaming state, out-of-order edge handling and a repair job you will still need to run. The measurement that justifies the move is nightly cluster cost, in machine-hours, exceeding the projected cost of the streaming path plus its repair batch. Cheaper alternative: run the batch twice a day.

### Breaking point, then V1 (people you may know)

**What fails: at 900 million members the nightly job is a thousand machines for 11.3 hours, and a connection made at 9am is invisible until tomorrow.**

How you observe it: two signals, and they are different problems wearing the same clothes.

The first is job runtime against cluster size, which will show the expansion stage dominating and scaling with the sum of squared degrees rather than with the edge count. That shape tells you the hubs are the cost, not the graph size.

The second is a staleness metric: the fraction of served recommendations that the member has already connected with, dismissed, or that no longer exist. A rising already-connected rate is the clearest evidence that the freshness window is too wide, and it is measurable without any experiment.

V1 replaces the batch with incremental triangle closing and adds a ranker:

```
+-------------+   +-------------------+   +---------------+
| Edge stream |-->| Triangle closer   |-->| Candidate     |
| new connects|   | skip hubs > 500   |   | table, top 500|
+-------------+   +-------------------+   +---------------+
                                                  |
+-------------+   +-------------------+           v
| Request     |-->| Ranker: GBDT on   |<----------+
| for cards   |   | affinity features |
+-------------+   +-------------------+
                          |
                          v
                  +-------------------+
                  | Cards with reason |
                  +-------------------+
```

**Caption.** V1 consumes new connection events, emits candidate pairs for the neighbourhoods of both endpoints while skipping high-degree intermediaries, maintains a per-member table of the top 500 candidates, and ranks that table with a gradient-boosted model at request time.

Two additions, each with a reason:

| Addition | Why here | What it costs |
|---|---|---|
| Incremental triangle closing | 2.4 billion emissions per day against 81 trillion, which is the four-orders-of-magnitude line from the estimation table | Streaming state, out-of-order events, and a weekly repair batch to correct drift |
| A ranking model over the candidate table | Shared-connection count alone ranks by candidate degree, which is the wrong objective | A training pipeline, a label definition and a serving path with a latency budget |

!!! warning "When not to add the ranker"

    If your accept rate is already near the ceiling that shared-connection count can reach, and you have fewer than roughly a million labelled impressions, the model will fit noise and you will not be able to tell. The measurement that justifies it is an offline replay showing lift over count-ranking on held-out impressions, at a confidence interval narrower than the lift. Cheaper alternative: divide the shared-connection count by the log of the candidate's degree, which is one line and removes most of the popularity bias.

### Breaking point, then V2 (people you may know)

**What fails: the surface recommends the same high-degree people to everyone, and it reveals things the member never disclosed.**

Two failures again, from opposite directions.

The quality failure: shared-connection count is monotone in candidate degree, so a recruiter with 20,000 connections shares connections with nearly everyone. How you observe it: plot accept rate against candidate degree decile. A falling curve with rising impression share in the top decile means the surface is spending its inventory on people nobody wants to connect with.

The trust failure: the graph knows things members did not say. A member who imported their address book has handed you private phone numbers. A member who hid their connection list still leaks it through other people's explanations. How you observe it: the report and hide rate on the surface, segmented by the signal that generated the candidate, which requires logging the source with each candidate.

V2 adds a serving-time policy stack:

```
+-------------------+   +--------------------+
| Ranked candidates |-->| Privacy filter     |
+-------------------+   | blocks, visibility |
                        | consent on source  |
                        +--------------------+
                                  |
                                  v
                        +--------------------+
                        | Exposure governor  |
                        | cap per candidate  |
                        +--------------------+
                                  |
                                  v
                        +--------------------+
                        | Explanation gate   |
                        | drop if unsayable  |
                        +--------------------+
```

**Caption.** V2 passes the ranked list through three serving-time policies in a fixed order: a privacy filter, a global exposure cap per candidate, and a gate that drops any recommendation whose reason cannot be shown to the member.

The order matters and is worth defending. Privacy first, because it is a correctness constraint rather than a preference. Exposure second, because the cap should be applied to what is otherwise servable. Explanation last, because it is the cheapest to evaluate and dropping there wastes the least work.

!!! warning "When not to add the exposure governor"

    If your impression distribution across candidates is not heavily skewed, the cap binds on nobody and you have added a global counter to the serving path. The measurement that justifies it is the share of total impressions going to the top 0.1 percent of candidates. Cheaper alternative: a per-candidate impression feature in the ranker with a negative coefficient, which achieves most of the effect without a cross-request counter.

### Deep dive one: candidate generation on a 135 billion edge graph

This is the dive because candidate generation, not ranking, is where the money and the difficulty are. The ranker is an ordinary gradient-boosted model; the generator is the system.

**Why full expansion is priced out, restated as one inequality.** The cost of a full two-hop expansion is the sum over every member of the square of their degree. With a mean of 300, that is at least 81 trillion emissions. With a heavy tail, it is worse, and the tail is where it goes.

| Member type | Degree | Emissions from expanding through them | Share of one member's cost |
|---|---|---|---|
| Typical | 300 | 90,000 | Baseline |
| Active professional | 2,000 | 4 million | 44x a typical member |
| Recruiter at the cap | 30,000 | 900 million | 10,000x a typical member |

One capped-out recruiter costs more to expand than ten thousand ordinary members. So the first design decision is not which algorithm, it is which intermediaries you refuse to expand through.

**The hub cap, justified rather than asserted.** Take the evidence value of a shared connection as inversely proportional to the log of the intermediary's degree, which is the Adamic-Adar weighting from 2003 and which encodes the obvious intuition that knowing someone through a person with twenty contacts means more than knowing them through a person with thirty thousand.

| Intermediary degree | Weight, 1 over natural log | Emissions cost | Evidence per emission |
|---|---|---|---|
| 20 | 0.333 | 20 | 0.0167 |
| 300 | 0.175 | 300 | 0.00058 |
| 5,000 | 0.117 | 5,000 | 0.0000234 |
| 30,000 | 0.097 | 30,000 | 0.0000032 |

Evidence per unit of cost falls by more than three orders of magnitude across that table. A cap at degree 500 keeps almost all of the evidence and removes almost all of the cost. State the cap as derived from this ratio, not as a tuning constant.

**The incremental algorithm, stated precisely.** When edge `(u, v)` is created:

- If `degree(u)` is at or below the cap, emit candidate `(w, v)` for every `w` in the neighbours of `u`.
- If `degree(v)` is at or below the cap, emit candidate `(x, u)` for every `x` in the neighbours of `v`.
- Each emission increments an accumulated score in the candidate table of the receiving member.

Cost is `degree(u) + degree(v)` per edge, so 600 emissions on average, 2.4 billion per day, 27,800 per second. That runs on one node with room to spare, which is the point.

**What the incremental path does not do, and the repair job that fixes it.**

| Drift source | Effect | Repair |
|---|---|---|
| Deleted connections | A candidate keeps a score from an edge that no longer exists | The weekly batch recomputes scores for members whose neighbourhood changed by more than a threshold |
| Score decay | A candidate strong six months ago and never accepted stays near the top | Apply a multiplicative decay on read, using the timestamp of last update, so no rewrite is needed |
| Truncation loss | A member who crosses 500 candidates permanently loses the ones that fell off | Accept it. The 501st candidate is not the one you were going to show |
| Hub cap blind spot | Two people whose only connection is through a recruiter never become candidates | Accept it too, and cover it with a second source below |

**Two-hop is not the only source, and it fails completely for new members.** A member with zero connections has an empty two-hop set. Enumerate the alternatives with their cost and their hazard, because the hazard is what deep dive two is about:

| Source | Coverage it adds | Cost | Hazard |
|---|---|---|---|
| Same employer and overlapping tenure | Strong for new members with a filled profile | A grouped scan, trivial | Reveals nothing the member did not publish |
| Same school and overlapping years | Strong for early-career members | Same | Same |
| Imported contacts | The strongest single signal for a brand new member | A hashed identifier join | Severe, and covered in the next dive |
| Co-viewed profiles | Behavioural, and it finds edges the graph cannot see | A co-occurrence count over view logs, which is another expansion problem with the same hub issue | Leaks browsing behaviour if the reason is shown |
| Geographic and event co-attendance | Fills a gap for members with sparse profiles | Depends on location precision | Severe at fine granularity |

The design decision is to blend sources at the candidate table with a per-source weight, and to carry the source identifier all the way to serving. Carrying the source is not a nicety: it is what makes the explanation gate and the privacy filter implementable at all.

!!! warning "When not to build the incremental path"

    If your freshness requirement is 24 hours and your batch fits in a normal nightly window, the incremental path adds streaming state, an out-of-order problem and a repair job for no product gain. The measurement that justifies it is the already-connected rate in served recommendations, or nightly cluster cost. Cheaper alternative: keep the batch and add a small online overlay that suppresses candidates the member connected with today, which fixes the visible symptom for almost nothing.

### Deep dive two: privacy as a hard constraint, and why the filter must sit at serving

The dives differ because the first is about cost and the second is about correctness in a sense no metric captures. The failure here is not a breach. It is a correct inference that the member experiences as a violation.

**The leak channels, each with a mechanism:**

| Channel | How the leak happens | Who is exposed |
|---|---|---|
| Contact import | A uploads an address book containing B's private number. Recommending B to A confirms that the person at that number is on the platform | B, who never consented to being findable by that identifier |
| Explanation | "You both know X" tells the member that X is connected to the candidate | X, who may have hidden their connection list |
| Co-viewed profiles | A recommendation derived from profile views can identify someone who browsed deliberately without leaving a trace | The viewer |
| Location co-presence | Two members at the same address at the same hour | Both, and the inference may be about health, legal or personal matters |
| Hidden connections | A member hides their own list, but every one of their connections can still surface them through explanations | The member who chose to hide |
| Blocks | If a block changes what a member sees, a determined observer can detect that a block exists | The person who blocked |

**The design responses, each with what it costs you:**

| Channel | Control | Cost |
|---|---|---|
| Contact import | Use an imported identifier as a signal only when it matches an identifier the target published, or when both parties imported each other | Loses a large share of the best cold-start signal, and you should measure how much rather than guess |
| Explanation | Only name an intermediary whose connection list is visible to the member on both sides | A share of candidates become unexplainable, so you choose between showing them without a reason and dropping them |
| Co-viewed profiles | Use as a ranking feature, never as a stated reason | You keep the signal and lose the explainability, which is the right trade here |
| Location | Coarsen to a granularity where co-presence is not identifying, for example city and day rather than building and hour | Removes most of the signal, which is the correct outcome |
| Hidden connections | Treat visibility as an attribute of the edge, and let it gate the explanation while still allowing the score | Members can still infer from the ranking, so state the residual exposure rather than claiming it is solved |
| Blocks | Symmetric hard filter, applied at serving on the final list, never at generation | Slightly more work per request, and it is the only correct place |

**Why the block filter must be at serving, which is the subtle part.** If you filtered at candidate generation, then flipping a block would change the candidate table, which changes downstream behaviour in ways that are observable through timing, through slot positions and through the surface's own recommendations elsewhere. The filter itself becomes a side channel.

The rule generalises: **a privacy control must be applied where its effect on the output is indistinguishable from the candidate simply not ranking well.** That means at the end, on a fixed-length list, with backfill from the next-ranked candidate so the list length does not vary.

**Differential exposure, the failure nobody mentions.** Impressions are distributed by a power law, so the arithmetic is stark:

- 4 billion impressions per day across 900 million members is a mean of 4.4 impressions per member per day.
- A power-law ranking gives the top 0.1 percent of candidates thousands each, and gives a long tail zero.

Both ends are problems. Zero impressions means a member is effectively invisible on the surface. Thousands means unwanted contact at a volume that reads as harassment. A cap of, say, 100 impressions per candidate per day binds on roughly the top 0.1 percent, and the accept-rate cost of the cap is measurable rather than assumed: run it as an experiment and read the total accepts, not the accepts among capped candidates.

**Dismissals are a consent signal, not just a negative label.** A member who dismisses a candidate has said something specific. Treat it as a hard exclusion for a long window rather than as a training negative with weight, because using it only as a training signal means the same face returns tomorrow with a slightly lower score.

!!! warning "When not to build the full policy stack"

    If your product has no hidden connections, no contact import and no blocks, the stack is three components filtering nothing. The measurement that justifies each layer is separate: the report-and-hide rate by candidate source for the privacy filter, the top-0.1-percent impression share for the governor, and the unexplainable-candidate rate for the gate. Build the one whose number is bad. Cheaper alternative: a single denylist applied at serving, which covers blocks and dismissals and nothing else.

### Failure modes and evaluation (people you may know)

| Failure class | How it shows up here | What you do about it |
|---|---|---|
| Popularity feedback loop | High-degree candidates get impressions, gain connections, get more degree, get more impressions | The exposure governor, plus degree-normalised features, plus an explicit measure of impression concentration |
| Hot key on hubs | A single high-degree member is read on a large fraction of requests, both as a candidate and as an intermediary | The degree cap removes them from expansion, and a local cache with a short time-to-live handles the serving read |
| Skew in the incremental job | New connections concentrate on a few members during a viral moment, so one partition of the triangle closer falls behind | Partition by the receiving member rather than by the edge, and shed hub expansions first when lag crosses a threshold |
| Stale exclusions | A member connects with a candidate on another surface, and this surface keeps recommending them | An online overlay of today's connections and dismissals, applied at serving, independent of the candidate table's freshness |
| Repair job divergence | The weekly batch and the streaming path disagree, and nobody notices which is right | Score a fixed audit set of members both ways every week and alert on rank correlation falling below a floor |
| Cold start silence | A member with no connections and a sparse profile receives an empty list, and empty is worse than mediocre | A guaranteed fallback source, for example same-city and same-industry, with the list length as an SLI |
| Privacy filter as side channel | A block or a visibility flag changes list length or ordering in a detectable way | Fixed-length lists with backfill, and filtering only at serving |
| Label shortcut | Training on accepts teaches the model to predict "already knows each other well", which the member did not need help with | Add a downstream label, for example an interaction within 30 days, and treat accepts as an auxiliary task |

Service level indicators:

- Accept rate per impression, segmented by candidate degree decile, because the unsegmented number hides the popularity loop entirely.
- Downstream value rate: the share of accepted connections with an interaction within 30 days.
- Report-and-hide rate, segmented by candidate source, which is the only instrumentation that makes the privacy dive actionable.
- Already-connected and already-dismissed rate in served lists, which measures staleness directly.
- Impression concentration, as the share of impressions going to the top 0.1 percent of candidates.
- Empty and short list rate, segmented by member connection-count decile.
- Triangle closer lag at p99, and the rank correlation between the streaming and repaired candidate tables.

Alert on report-and-hide rate by source, on empty list rate, and on triangle closer lag. Do not alert on accept rate alone, because it rises when the surface becomes more obvious and falls when the graph saturates, and neither movement is an incident.

Re-check against the requirements:

| Requirement | Does V2 meet it? |
|---|---|
| 17,400 requests per second at p99 under 200 ms | Yes: one read of a 500-row candidate table, a gradient-boosted scoring at roughly 17 cores total, and three cheap serving-time filters |
| New connection reflected within 24 hours | Yes, and in seconds through the incremental path, with the batch demoted to repair |
| 2.16 TB of adjacency and 1.6 TB of candidates | Yes, on a modest sharded store, which the estimation table showed rules out both a single node and a large graph engine |
| Excludes connections, dismissals and blocks | Yes, via the serving-time overlay and the privacy filter, which is also the only correct place for blocks |
| A reason for every recommendation | Yes, but only for the sources whose reason is sayable. The explanation gate makes the shortfall explicit rather than hiding it |
| Serves members with zero connections | Yes, via the profile-based and cohort sources, at a lower expected accept rate that you should report separately |

### Where this answer is a simplification (people you may know)

All links in this list were checked on 1 August 2026.

- **Adamic and Adar, "Friends and Neighbors on the Web"** (Social Networks, 2003) is the origin of the inverse-log-degree weighting that justifies the hub cap, and it is a short and readable paper.
- **Liben-Nowell and Kleinberg, "The Link Prediction Problem for Social Networks"** (published at CIKM 2003 and in JASIST 2007) is the standard comparison of graph proximity measures, and it is the right thing to read before claiming any one of them is best.
- **Backstrom and Leskovec, "Supervised Random Walks: Predicting and Recommending Links in Social Networks"** (WSDM, 2011) is the primary treatment of learning the edge weights rather than fixing them, which is the natural V3 for the candidate generator.
- **Gupta and colleagues, ["WTF: The Who to Follow Service at Twitter"](https://web.stanford.edu/~rezab/papers/wtf_overview.pdf)** (WWW, 2013) is a company-published account of a production recommendation service on a large graph, including the decision to keep the graph in memory on a single machine, which is a useful counterpoint to the sharded store proposed here.
- **Backstrom and Kleinberg, "Romantic Partnerships and the Dispersion of Social Ties"** (CSCW, 2014) shows how much can be inferred about a relationship from graph structure alone, which is the clearest available argument for why the privacy dive is not hypothetical.
- **Narayanan and Shmatikov, "De-anonymizing Social Networks"** (IEEE Symposium on Security and Privacy, 2009) and **Backstrom, Dwork and Kleinberg, "Wherefore Art Thou R3579X?"** (WWW, 2007) establish that graph structure is itself identifying, which is the technical basis for treating structure as sensitive data rather than as metadata.
- **The ranking objective is a policy question.** Optimising accepts, optimising downstream interaction and optimising network health give visibly different products, and choosing between them is not an engineering decision. A staff answer names the tension and asks who owns it.

### Level delta (people you may know)

| Dimension | Mid level | Senior | Staff |
|---|---|---|---|
| Opening move | Proposes a link prediction model | Asks whether the objective is accepts or lasting value | Asks whether every recommendation must be explainable, because that answer constrains candidate generation before ranking exists |
| Candidate generation | "Two-hop neighbours" | Notes hubs are expensive and caps degree | Derives the cap from evidence per emission falling three orders of magnitude between degree 20 and degree 30,000 |
| Batch versus incremental | Proposes a nightly job | Proposes streaming for freshness | Prices both at 40.5 million node-seconds against 1,200, then keeps the batch as a repair path rather than deleting it |
| Storage | Not sized | Sizes the graph | Sizes the graph at 2.16 TB and candidates at 288 TB untruncated, using the second number to derive the stored depth of 500 |
| Ranking | Proposes a deep model | Chooses a gradient-boosted model for latency | Shows a cross-attention model needs 8,700 cores against 17, so the choice is arithmetic rather than preference |
| Popularity | Not raised | Normalises by candidate degree | Adds an exposure governor, and specifies that its cost is measured as total accepts rather than accepts among capped candidates |
| Privacy | "We would respect user settings" | Adds a block filter | Enumerates six leak channels, and shows why the filter must sit at serving on a fixed-length list or it becomes a side channel itself |
| Cold start | Not raised | Adds profile-based sources | Ranks the sources by hazard as well as by coverage, and carries the source identifier to serving so the policy stack can act on it |
| Evaluation | Reports accept rate | Segments by degree | Alerts on report-and-hide rate by source and on empty list rate, and explicitly declines to alert on accept rate with a reason |

What each level added:

- **Mid to senior:** the problem becomes a candidate generation problem with a cost model, rather than a modelling problem with a graph attached.
- **Senior to staff:** every threshold is derived from a ratio the reader can recompute, privacy becomes a placement decision in the serving path rather than a promise, and the candidate names which of their own metrics will mislead the team.

---

## 10. Practice problems

Two sets, and they do different jobs. The blocked set rehearses the arithmetic and the moves you just watched, one at a time, with the topic already named. The interleaved set hides the topic, so your first task is to work out which tool the prompt is asking for. That first task is the one an ML design round actually scores.

Work everything on paper or out loud before opening an answer. Rubrics, model answers and diagnosed wrong answers for all eleven problems are in Section 11.

### 10a. Blocked set: five problems, same shape as the worked examples

**B1. Split a latency budget across a retrieval and ranking stack.**

Your recommendation API has a stated p99 of 300 ms measured at the gateway. Assume 40 ms of that is client network and TLS, because a mobile round trip on a good cellular connection is tens of milliseconds and you do not control it.

Measured on your hardware: an approximate nearest neighbour query returns in 12 ms p99 with shards queried in parallel, business-rule filtering takes 5 ms, a single batched feature fetch takes 15 ms p99, diversity re-ranking takes 8 ms, and serialisation plus framework overhead takes 10 ms. Your ranker scores 500 candidates in 25 ms on one replica.

How many candidates can the ranker score, and which two design options does that number eliminate?

??? note "Show answer"

    Server budget: 300 minus 40 equals 260 ms.

    Fixed stages: 12 plus 5 plus 15 plus 8 plus 10 equals 50 ms. Remaining is 210 ms.

    Reserve headroom. Take 60 ms, which is roughly a quarter of the server budget, for garbage collection pauses, one in-flight retry to the feature store, and the fact that your stage measurements are p99 individually and do not compose into a p99 overall. Ranking budget is 150 ms.

    Per-candidate cost: 25 ms divided by 500 candidates is 0.05 ms per candidate. At 150 ms the ranker scores 150 divided by 0.05, which is 3,000 candidates.

    Option eliminated, one: ranking the full catalogue. A 10 million item catalogue at 0.05 ms per item is 500 seconds. This is the single number that forces a retrieval stage to exist at all, and it is the sentence to say out loud before you draw the two-stage diagram.

    Option eliminated, two: a cross-encoder as the main ranker. If a cross-encoder costs 2 ms per item (state it as your measurement or as an assumption), 150 ms buys 75 items. A cross-encoder is therefore only affordable as a final re-rank over roughly the top 50 to 75, not as the ranker over 3,000.

    The move that reads as senior: say the headroom reservation out loud and say why. Candidates who allocate the full 210 ms to ranking have designed a system with a p99 that is correct exactly once.

**B2. Size an embedding index and decide the shard count.**

You have 200 million items with 256-dimensional float32 embeddings. Your serving nodes have 128 GB of usable memory after the operating system and the process baseline. You plan to use a graph index with 32 links per node, which stores up to 64 neighbour identifiers at the base layer at 4 bytes each.

What does the index cost, how many shards do you need, and what does int8 quantization change?

??? note "Show answer"

    Vectors: 200e6 times 256 dimensions times 4 bytes equals 204.8e9 bytes, so about 205 GB.

    Graph: 64 neighbour identifiers times 4 bytes equals 256 bytes per vector at the base layer. Upper layers add a small fraction because each layer holds roughly one node in every few from the layer below. Round the total to 300 bytes per vector. 200e6 times 300 bytes equals 60e9, so 60 GB.

    Total: about 265 GB.

    Shards: 265 divided by 128 is 2.07, so three shards minimum. Take four, because at three you cannot lose a node and you cannot hold two index versions during a swap. Four shards at about 66 GB each leaves room to load a new index beside the live one.

    int8 scalar quantization: 200e6 times 256 times 1 byte equals 51.2 GB, plus the same 60 GB graph, so about 111 GB. That fits one node, but with no room for a second copy during a rebuild, so two nodes is the honest answer.

    What is eliminated: keeping float32 vectors resident on a single machine, and any design that rebuilds in place without a second copy.

    What quantization costs: recall at a fixed candidate count drops, by an amount you have to measure on your own data rather than assume. The standard repair is to retrieve more candidates from the quantized index and rescore the top few hundred with the full-precision vectors, which trades a bounded amount of memory and latency for recall you can measure.

**B3. Choose a retraining cadence from decay and cost.**

A full retrain uses 8 accelerators for 6 hours. Assume 3 dollars per accelerator-hour on demand, which is the order of magnitude of published on-demand rates for a mid-range training accelerator in 2026; substitute your provider's current number, because the method does not change. Your measurement: on a rolling fresh holdout, offline AUC falls by about 0.002 per day after the training data is frozen. Your launch bar, from your own experiment history, is that an offline AUC gain below 0.004 has never produced a positive online result.

Pick a cadence and say what each rejected cadence costs.

??? note "Show answer"

    Cost of one full retrain: 8 times 6 equals 48 accelerator-hours, times 3 dollars, equals 144 dollars.

    Decay against the bar: at 0.002 AUC per day, the model has given back your entire 0.004 launch-worthy margin in 2 days. That is the number that sets the cadence, not a calendar preference.

    Weekly is eliminated: by day 7 the model has lost 0.014 AUC, more than three times the smallest delta you have ever been able to detect online. You would be shipping a decay you can measure and choosing not to.

    Hourly is eliminated on cost against benefit: 24 times 30 times 144 dollars is 103,680 dollars a month, to buy at most the 0.002 of AUC that sits between hourly and daily. Compare that against the value of 0.002 AUC in your own currency before proposing it.

    Daily full retrain: 30 times 144 equals 4,320 dollars a month. This clears the bar with one day of margin.

    The answer that beats both: daily warm-start incremental training on the new day of data, plus a weekly full retrain from scratch to clear accumulated optimiser state and to pick up schema changes. If the incremental job is 45 minutes on one accelerator, that is 0.75 accelerator-hours, about 2.25 dollars, so 30 incremental runs plus 4 full runs is about 644 dollars a month.

    State the risk you took on. Warm starting can carry forward a bad state silently, so the weekly full run is not optional, and the two models must be compared on the same holdout before either is promoted.

**B4. Size the experiment before you promise a launch date.**

Baseline click-through rate is 4 percent. You want to detect a 2 percent relative lift with 80 percent power at a two-sided 5 percent significance level. You have 400,000 daily unique users eligible for the surface, split 50 to 50.

How long does the test run, and what does the answer rule out?

??? note "Show answer"

    Use the two-proportion rule of thumb: n per arm is approximately 16 times p times (1 minus p) divided by delta squared. The 16 comes from (1.96 plus 0.84) squared, which is 7.84, doubled because two arms each contribute variance, then rounded up.

    delta: a 2 percent relative lift on a 4 percent base is 0.04 times 0.02 equals 0.0008 absolute.

    n per arm: 16 times 0.04 times 0.96 divided by 0.0008 squared. The numerator is 0.6144. The denominator is 6.4e-7. That gives 960,000 users per arm.

    Duration: 400,000 daily uniques split evenly is 200,000 per arm per day, so 960,000 divided by 200,000 equals 4.8 days. Run 7 days, because weekday and weekend behaviour differ and a 4.8 day test is a partial week.

    Ruled out, one: promising a read in 2 days. At 400,000 per arm accumulated you have detection power for roughly a 3 percent relative lift, not 2 percent, and reading the test early is how a team ships noise.

    Ruled out, two: a 0.5 percent relative lift as a launch criterion. Quartering delta multiplies n by 16, giving 15.36 million per arm, which is about 77 days. If the product wants that resolution, the answer is a different measurement design (interleaving, or a switchback on a shared resource), not a longer A/B test.

    Say the units out loud. If randomisation is by user but the metric is per impression, the effective sample size is smaller than the impression count because impressions within a user are correlated, and the naive calculation above overstates your power.

**B5. Size the online feature path.**

Peak traffic is 1,500 requests per second. Each request scores 2,000 candidates. You have 120 user features and 40 item features, all float32. Your item catalogue is 5 million.

Compute the online feature load and decide where item features live.

??? note "Show answer"

    User features: fetched once per request. 1,500 times 120 times 4 bytes equals 720 KB per second. This is not a design problem and you should say so and move on.

    Item features: 1,500 times 2,000 equals 3 million item feature vectors per second. Each is 40 times 4 equals 160 bytes, so 3e6 times 160 equals 480 MB per second.

    Eliminated: one remote lookup per candidate. Three million individual key-value reads per second against a store that serves tens of thousands of operations per second per node needs a node count in the hundreds, to serve a table whose entire contents are small.

    Option A, batched multi-get: one request per candidate set. 1,500 batched calls per second, each pulling 2,000 times 160 bytes equals 320 KB, so 480 MB per second on the wire. Feasible, and it adds a network hop plus tail latency inside your ranking budget from B1.

    Option B, co-locate: the whole item feature table is 5e6 times 160 bytes equals 800 MB. That fits in the ranker's process memory with room to spare. Zero network calls for item features, and the 15 ms feature fetch from B1 shrinks to a user-feature lookup only.

    Take option B, and then say the cost, because the cost is the interesting part. An in-process copy is a cache with no invalidation protocol. It goes stale between refreshes, every replica can hold a different version, and if the offline training table is refreshed on a different schedule than the in-process copy, you have manufactured training-serving skew that no offline metric will show you. The design obligation is a version stamp on the table, the same stamp logged with every prediction, and an alert when replicas disagree.

### 10b. Interleaved and cumulative set: six problems, topic not stated

These are unlabelled on purpose. Decide which tool applies before you start designing. Several of them reach back to material that is not on this page: the [trade-off atlas](../#the-trade-off-atlas) and the [failure library](../#the-failure-library) on the hub, the [Modern System Design course](../modern-system-design/) for the serving and storage layer underneath a model, the [system design question bank](/Interview-Questions/System-design/), and the [machine learning question bank](/Interview-Questions/Machine-Learning/) for the metric definitions.

Twenty to thirty minutes each, out loud, with a timer running.

**I1.** Our catalogue is 40,000 products. A team member has proposed adding a dedicated approximate nearest neighbour service for retrieval, because the current implementation scans everything. Design it.

**I2.** Here is what runs today: a nightly batch job computes features, trains a ranking model, and precomputes a top 500 list for every one of our 30 million registered users into a key-value store. The app reads that list. Product now wants recommendations to reflect what a user did in the last 15 minutes, and finance has said the infrastructure bill cannot double. Plan it.

**I3.** Our card fraud model's precision has fallen from about 0.70 to about 0.50 over six weeks. Confirmed labels come from chargebacks, which arrive up to 45 days after the transaction. Diagnose this and say what you would change.

**I4.** We launched a new click prediction model. Offline AUC went up by 0.01 on a held-out day, which is our largest gain this year. Two weeks after ramp, advertiser-facing revenue is down about 6 percent and advertisers are complaining about delivery. Find it.

**I5.** A feed ranking change increased daily engagement by 4 percent. In the same period, the rate of users reporting content rose by about 30 percent. The launch review is tomorrow. What is your recommendation and how do you support it?

**I6.** Our recommendation service holds a p99 of 180 ms all day. Every time we push a new model and a new embedding index, p99 goes to about 3 seconds for roughly ten minutes and then returns to normal with no intervention. Name the class and design the fix.

!!! warning "Honesty note about the interleaved set"

    You will score lower here than on the blocked set and it will feel like the reading did not work. The gap is the design. The blocked set measures whether you can execute a move once someone has told you which move it is; the interleaved set measures whether you can pick one, which is the only thing an interviewer can observe. A drop of one or two rubric levels between the two sets is the expected outcome, not a signal to reread.

---

## 11. Self-assessment rubrics

### The master rubric

Every problem in Section 10 is scored on the same four criteria, with named levels rather than numbers. A visible score crowds out the comment, and the comment is the part that changes your next answer.

| Criterion | Developing | Adequate | Strong | Exceptional |
|---|---|---|---|---|
| Problem framing and scoping | Starts drawing a pipeline before asking what the system is for. Restates the prompt as the requirements | Asks questions, but no answer would change a box, a metric or a budget | Every question has two branches and the chosen branch is stated. Names the business metric and the trainable proxy separately | Names the guardrail metric nobody asked about, and the number at which the launch should be blocked |
| Design and trade-off reasoning | One pipeline, presented as the answer. No alternative named | Alternatives named, chosen by familiarity or by what is fashionable | Each choice is settled by a number computed in front of the interviewer, and the rejected option is named with the condition under which it wins | Identifies which decision is expensive to reverse (label definition, embedding version, randomisation unit) and sequences around it |
| Depth in at least one area | Box level everywhere. Model choice is a brand name | Goes one layer deeper when pushed | Volunteers one dive that reaches implementation detail: index parameters, the join key of the training table, the calibration procedure | The dive includes how it fails silently, what would detect it, and what the recovery costs in time and money |
| Communication and drive | Waits to be asked what comes next | Answers well, does not move the conversation | States the plan, keeps the clock, offers forks instead of asking for approval | Absorbs a hostile follow-up by changing the design and naming which earlier claim is now void |

Score all four on every problem. Two Strongs and two Adequates is a passing senior answer. Four Adequates with nothing Strong is the profile most often written up as "hire, one level down", because nothing was memorable.

### Rubric notes, blocked set

??? note "B1 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks whether 300 ms is measured at the gateway or at the client, because the answer moves 40 ms | Takes 300 ms as the server budget |
    | Trade-offs | Reserves headroom explicitly and says why p99 stages do not compose | Allocates every remaining millisecond to ranking |
    | Depth | Converts the ranker measurement into cost per candidate and uses it twice | Quotes 25 ms without dividing by 500 |
    | Communication | States the eliminated option immediately after each number | Presents the arithmetic and stops |

    Model answer: 300 minus 40 is 260 ms of server budget. Fixed stages sum to 50 ms, leaving 210 ms. I reserve 60 ms for headroom, because each stage figure is an independent p99 and combining them gives something worse than any one of them, so 150 ms is the ranking budget.

    The ranker costs 25 divided by 500, or 0.05 ms per candidate, so I can score 3,000. That eliminates ranking the catalogue, which at 10 million items would be 500 seconds. It also eliminates a 2 ms per item cross-encoder as the main ranker, since 150 ms buys 75 items, so the cross-encoder becomes a final re-rank over the top 50.

    Wrong answer: "We will use a fast model so latency is fine." Reveals that latency is being treated as a property of the model rather than a budget shared by stages. The interviewer cannot grade a design that has no number in it.

    Wrong answer: "Score 3,000 candidates, and if it is slow we will add GPUs." Reveals no model of where the time goes. Adding accelerators does not shrink the feature fetch, the filter or the serialisation, which together are 30 ms of the 50 ms fixed cost.

    Wrong answer: "Cache the results so p99 does not matter." Reveals a confusion between a cache hit rate and a tail. A cache that is hit 90 percent of the time leaves the p99 entirely to the 10 percent that miss.

    Compare your process: did you subtract the network before you started allocating, or did you discover it later?

??? note "B2 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks what recall the product needs before choosing a compression scheme | Chooses quantization because it saves memory |
    | Trade-offs | Sizes both float32 and int8 and lets the node count decide | Names quantization without computing what it saves |
    | Depth | Accounts for graph overhead separately from vectors, and for the second copy during a rebuild | Counts only vector bytes |
    | Communication | Says that the recall cost must be measured, not assumed | Asserts a recall number with no source |

    Model answer: vectors are 200e6 times 256 times 4 bytes, so 205 GB. The graph at 64 base-layer neighbours times 4 bytes is 256 bytes per vector, round to 300 with upper layers, so 60 GB. Total 265 GB against 128 GB usable per node is 2.07, so three shards is the floor and four is the answer, because at three I cannot hold a second index copy during a swap or lose a node.

    int8 brings vectors to 51 GB and the total to 111 GB, which fits one node in theory and two in practice for the same reason. If I quantize, I retrieve deeper and rescore the top few hundred with full-precision vectors, and I measure the recall loss on my own query set rather than quoting one.

    Wrong answer: "Use a managed vector database and let it handle sharding." Reveals that the sizing was skipped. The shard count is what determines the fan-out on every query, the cost, and the blast radius of one node, and none of that changes because a vendor operates it.

    Wrong answer: "Compress with product quantization to 64 bytes per vector, so it all fits easily." Reveals a memory-only view. That is 12.8 GB of vectors, which is correct arithmetic, but without a rescoring stage you have replaced a measured recall with an unmeasured one, and recall is what the retrieval stage exists to provide.

    Wrong answer: "Three shards." Reveals no operational model. Three is the arithmetic floor, and it leaves you unable to rebuild or to survive a node loss.

    Compare your process: did you leave room for two copies of the index, or only for one?

??? note "B3 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Ties cadence to a measured decay rate and a launch bar, not to a calendar | Says "retrain nightly, that is standard" |
    | Trade-offs | Prices hourly retraining and rejects it on cost against the AUC it buys | Names cadences without cost |
    | Depth | Proposes incremental plus periodic full, and names the failure that makes the full run mandatory | Proposes incremental with no safety net |
    | Communication | Labels the dollar rate as an assumption and says the method survives a different rate | Presents the rate as a fact |

    Model answer: a retrain is 48 accelerator-hours, about 144 dollars at an assumed 3 dollars per accelerator-hour. Decay is 0.002 AUC per day and my launch bar is 0.004, so the model is spent in 2 days. Weekly is out: 0.014 of decay. Hourly is out: about 103,680 dollars a month to buy at most 0.002 over daily. Daily full retrain is 4,320 dollars a month and clears the bar.

    What I would actually ship is a daily warm-start increment at roughly 2.25 dollars a run plus a weekly full retrain, about 644 dollars a month. Warm starting can silently carry a bad optimiser state or miss a schema change, and the weekly full run is the thing that catches it. Both are compared on the same holdout before either is promoted.

    Wrong answer: "Retrain continuously, fresher is always better." Reveals no cost model and no decay measurement. It also creates an operational problem the candidate did not mention: with continuous retraining, a bad data day reaches production before anyone reviews it.

    Wrong answer: "Retrain when drift alerts fire." Reveals a plausible-sounding trigger that cannot be operated as stated. Drift on which feature, at what threshold, and what happens when the alert fires at 3 a.m.? Event-triggered retraining is a real pattern, but it needs the threshold and the automated gate specified, otherwise it is a nightly job with extra steps.

    Wrong answer: "Cadence is a data science decision, not a system design one." Reveals a scope misread. Cadence sets the training cluster size, the feature backfill schedule and the model registry churn, all of which are architecture.

    Compare your process: did the decay rate choose the cadence, or did you choose the cadence and then look for support?

??? note "B4 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks what the randomisation unit is and whether the metric is per user or per impression | Computes with impressions and users interchangeably |
    | Trade-offs | Shows the quadratic penalty on smaller effects and proposes a different design rather than a longer test | Suggests running longer until it is significant |
    | Depth | Rounds to a whole week and says why (weekly seasonality) | Reports 4.8 days |
    | Communication | Says what an early read would and would not be able to detect | Promises a number by a date with no power statement |

    Model answer: delta is 0.04 times 0.02, so 0.0008. n per arm is 16 times 0.04 times 0.96 divided by 0.0008 squared, which is 0.6144 divided by 6.4e-7, so 960,000 per arm. At 200,000 per arm per day that is 4.8 days, and I would run 7 to cover a full weekly cycle. A 2 day read only has power for about a 3 percent relative lift, so I would not commit to a decision then.

    If someone wants 0.5 percent resolution, that is 16 times the sample, about 77 days. The right response is to change the measurement (interleaving for a ranking change, or a switchback if the treatment leaks between users) rather than to run a two and a half month A/B test.

    Wrong answer: "Run it until it is significant." Reveals no model of the error rate. Repeated peeking at a fixed threshold inflates the false positive rate well beyond 5 percent, and the fix is either a fixed horizon decided in advance or a sequential test designed for continuous monitoring.

    Wrong answer: "Use more traffic, ramp to 100 percent." Reveals a misunderstanding of what the second arm is for. At 100 percent treatment there is no control, so there is no measurement.

    Wrong answer: "Bandits solve this, so no sizing is needed." Reveals a substitution of mechanism for question. A bandit allocates traffic toward the better arm; it does not give you an unbiased estimate of the effect size that finance is going to be quoted.

    Compare your process: did you state the randomisation unit before computing, or after?

??? note "B5 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Separates per-request features from per-candidate features immediately | Treats all features as one lookup |
    | Trade-offs | Computes 3 million reads per second and uses it to kill the naive option | Says "we will use a feature store" and moves on |
    | Depth | Sizes the item table at 800 MB and proposes co-location, then names the staleness contract | Proposes co-location with no version stamp |
    | Communication | Says out loud that this is where training-serving skew is manufactured | Mentions skew as a generic caveat |

    Model answer: user features are 1,500 times 120 times 4 bytes, about 720 KB per second, which is not a problem. Item features are 1,500 times 2,000 equals 3 million vectors per second at 160 bytes each, so 480 MB per second, and that kills per-candidate remote lookup outright. Batched multi-get works and costs a hop plus tail inside my ranking budget.

    Better: the entire item feature table is 5 million times 160 bytes, which is 800 MB, so it lives in the ranker's process memory and the item feature fetch disappears. The price is that this copy has no invalidation protocol, so I attach a version stamp to the table, log that stamp with every prediction, and alert when replicas disagree or when the serving stamp differs from the one the training table was built with.

    Wrong answer: "Put a cache in front of the feature store." Reveals a read-path reflex without sizing. The working set is the whole catalogue, so the cache is the table, and you have arrived at co-location by a longer route with worse guarantees.

    Wrong answer: "Precompute the full feature vector per candidate offline." Reveals that request-time features were forgotten. Anything that depends on the current user or the current session cannot be precomputed per item, which is the reason the split exists.

    Wrong answer: "The feature store guarantees offline and online consistency." Reveals trust in a product claim as a substitute for a design. Consistency is a property of your pipeline definitions and your refresh schedules, and a store can only help you enforce it.

    Compare your process: did you compute 3 million per second before or after you named a technology?

### Rubric notes, interleaved set

??? note "I1 rubric, model answer and wrong answers (the answer is not to add it)"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks what problem the index is meant to solve and finds that the scan is already fast enough | Starts choosing index parameters |
    | Trade-offs | Prices the new failure surface against a saving of a couple of milliseconds | Prices only the speed win |
    | Depth | Names index staleness, deletion handling and recall as new tunables you would now own | Names memory as the only cost |
    | Communication | Declines with arithmetic and gives the threshold at which the answer flips | Either builds it or dismisses the proposal |

    Model answer: I would not add it, and here is the arithmetic. Forty thousand items at 256 float32 dimensions is 40,000 times 256 times 4 bytes, about 41 MB. A brute-force scan reads that once per query. Assume a usable single-socket memory bandwidth of about 20 GB per second, which is a conservative order of magnitude for a server reading a contiguous float array; the scan is therefore around 2 ms, and it is exact.

    Against that, an approximate index buys perhaps 1.5 ms and costs: a recall parameter I now have to measure and defend, a rebuild schedule, a staleness window between a product being added and being retrievable, tombstones for deletions, a second artefact to version alongside the model, and one more service that is down at 3 a.m.

    I would revisit at roughly 20 times this catalogue, where the scan reaches 40 ms and starts eating the retrieval budget, or sooner if the embedding dimension grows or if we batch many queries per request.

    What I would do now instead: check that the scan is actually vectorised as a single matrix product rather than a Python loop, and batch queries so one memory pass serves several users.

    Wrong answer: designing the index competently, as asked. Reveals that the candidate optimises whatever they are pointed at. This prompt exists to see whether you will produce a number that contradicts the request.

    Wrong answer: "No, that is premature optimisation." Reveals the right instinct with no evidence. A refusal without arithmetic scores the same as compliance without arithmetic, because neither shows the interviewer a decision process.

    Wrong answer: "Add it anyway, we will need it when the catalogue grows." Reveals a habit of designing for an unstated future. Say the growth rate that would justify it and the lead time to build it, and the argument becomes legitimate; without those two numbers it is a preference.

    Compare your process: did you say the threshold at which you would change your mind?

??? note "I2 rubric, model answer and wrong answers (brownfield)"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Decomposes "fresher" into feature, candidate and model staleness and asks which one the complaint is about | Accepts "15 minutes" as one requirement |
    | Trade-offs | Finds cost to pay for freshness inside the existing job rather than adding budget | Adds a streaming stack and hopes the bill holds |
    | Depth | Specifies the rollout: shadow, per-cohort ramp, rollback trigger, and what is compared | Says "we will move to real time" |
    | Communication | States explicitly what is not being changed and why | Presents a target architecture with no order |

    Model answer, in the order I would ship it.

    First, decompose. "Fresher" can mean three different lags: features are up to 24 hours old, the candidate list is up to 24 hours old, and the model is up to 24 hours old. They have different costs and only one of them is usually the complaint. If a user watches three cooking videos and the next screen is unchanged, that is candidate and feature staleness within a session, not model staleness.

    Second, find the money before spending it. Precomputing a top 500 for all 30 million registered users assumes all of them return. If, say, 20 percent are active on a given day (measure this, do not assume it), then 80 percent of that batch job is producing lists nobody reads. Moving to on-demand computation with a short-lived cache, or restricting precompute to users active in the last 30 days, frees a large fraction of the existing spend. That is the budget for the freshness work.

    Third, add the smallest fresh layer. Keep the nightly candidate list as the base. Add a session re-ranking stage at request time that reorders the precomputed 500 using session features from a streaming store: last N interactions, current context, time of day. This buys within-session responsiveness for the cost of one small model and a low-latency store, and it leaves training, retrieval and the batch job untouched.

    Fourth, only if the complaint survives. Union in a small streaming candidate source of items that became popular or were newly published in the last 15 minutes, capped at a fixed fraction of the slate so it cannot take over the page.

    What I am deliberately not doing: not retraining every 15 minutes, because model staleness was not the complaint and B3-style arithmetic shows what that costs; and not building online retrieval over the full catalogue, because the batch candidate list is not the thing users are noticing.

    Rollout: shadow the re-ranker for a week comparing its output against the live slate, ramp by cohort, and set the rollback trigger on serving latency and on the guardrail metric, not on the engagement metric.

    Wrong answer: "Move the whole pipeline to streaming." Reveals a template answer. It changes every component at once, has no intermediate state to measure, and its cost is exactly what the constraint forbids.

    Wrong answer: "Retrain the model every 15 minutes." Reveals a conflation of model freshness with content freshness. The model's parameters are not what is stale to the user.

    Wrong answer: "Add a Kafka topic and a Flink job." Reveals technology selection standing in for a problem statement. Neither of those names says which of the three lags is being fixed.

    Compare your process: did you look for money inside the existing system before asking for more?

??? note "I3 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Notices immediately that a 45 day label delay means the measurement describes the past, not the present | Treats the precision number as current |
    | Trade-offs | Separates adversarial adaptation, label definition change and population shift, with a distinguishing signal for each | Concludes "drift" and proposes a retrain |
    | Depth | Names the censored-label feedback loop created by blocking, and proposes a bounded random holdout | Ignores that blocked transactions never get labels |
    | Communication | Proposes label-free monitoring that would have caught this weeks earlier | Proposes only a retrain |

    Model answer: the first thing to say is that with a 45 day chargeback window, the precision I am looking at is computed on transactions that are at least 45 days old. The decline started before I could see it and the current model may already be worse. Every remedy has to account for that lag.

    Three hypotheses, each with a signal that separates it:

    - Adversarial adaptation. Attackers found a gap. Signal: the drop concentrates in a slice (a card range, a device fingerprint, a merchant category, a geography) rather than spreading evenly, and the score distribution inside that slice moved while the rest held.
    - Label definition change. A policy change in what gets charged back, or a change in how the manual review team dispositions cases, alters what the label means. Signal: the base rate of positives moved without any feature distribution moving.
    - Population shift. A new market, a new payment method or a marketing campaign changed the traffic mix. Signal: feature distributions moved on inputs that have nothing to do with fraud, and the model's score distribution shifted with them.

    Monitoring that does not wait 45 days: feature and score distribution drift (no labels needed at all), proxy labels with shorter latency such as manual review dispositions within hours and customer disputes within days, and a small human-reviewed sample of high-score and borderline transactions labelled deliberately.

    The design point most candidates miss: blocked transactions never generate chargebacks, so the training set is censored by the model's own decisions, and the model progressively loses the ability to see what it already blocks. The repair is a small, capped random holdout that is allowed through with a stated maximum expected loss per day, which buys unbiased labels in the region the model is most confident about. That budget is a product decision and it needs an owner.

    Wrong answer: "Retrain on recent data." Reveals no diagnosis. If the label definition changed, retraining bakes the new definition in without anyone noticing. If it is adversarial, retraining on 45 day old labels teaches the model an attack that has already moved.

    Wrong answer: "Lower the threshold to recover precision." Reveals confusion between precision and the threshold. Raising the threshold raises precision and drops recall; the question is whether the model still separates the classes at all, which is a ranking question, not a threshold one.

    Wrong answer: "Add more features." Reveals an instinct to improve rather than to diagnose. Until you know which of the three hypotheses holds, a new feature is a guess with a two month feedback loop.

    Compare your process: did you say the words "this measurement is 45 days old" in the first minute, or did you start proposing fixes?

??? note "I4 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Asks how the score is used downstream before discussing the model | Discusses only model quality |
    | Trade-offs | Distinguishes ranking quality from probability quality and says why the auction needs the second | Treats AUC as a general goodness measure |
    | Depth | Proposes a calibration curve and the predicted-over-observed ratio per slice, and names the fix | Says "recalibrate" with no procedure |
    | Communication | Names the launch gate that should have caught it | Blames the model |

    Model answer: AUC is invariant under any strictly increasing transform of the scores, so a model can rank strictly better and still output probabilities that are systematically wrong. In an ad auction the score is not a ranking, it is a probability: expected value per impression is bid times predicted click probability. If predictions are inflated by a factor, ads win auctions they should lose, delivery goes to the wrong campaigns, and pacing and billing both drift.

    Diagnosis, in order. Compute the ratio of the sum of predicted probabilities to the sum of observed clicks, overall and then by slice (campaign, placement, device, new advertisers). A ratio near 1.0 overall with large per-slice deviations tells you calibration is broken where it matters even though the average looks fine. Plot predicted against observed in deciles.

    Likely causes to check, each testable. One: the training label window changed, so conversions or clicks that arrive late were counted as negatives and the model is under-calibrated on slow-converting slices. Two: negative downsampling was applied for training and the inverse correction was not applied at serving, which inflates every prediction by a constant factor. Three: the position bias correction changed, so the label the model is fitting is no longer the same quantity the auction assumes.

    Fix: fit a monotone recalibration (isotonic, or a Platt-style logistic on the logit) on a recent holdout, per slice where the base rates genuinely differ, and re-evaluate. Then change the launch gate: AUC alone cannot block this launch, so add log loss and an expected calibration error threshold, plus the predicted-over-observed ratio on the top slices, to the offline criteria.

    Wrong answer: "Roll back and retrain with more data." Reveals no diagnosis. Rolling back is correct as an immediate action, but without the calibration check the same launch happens again next quarter.

    Wrong answer: "AUC went up, so the model is better and the revenue drop is a coincidence." Reveals a metric being treated as the objective. The business metric is the objective; AUC is a proxy that was chosen because it usually correlates, and this is the case where it did not.

    Wrong answer: "Add a constant multiplier to the scores until revenue recovers." Reveals a fix aimed at the symptom. If miscalibration varies by slice, a global constant moves the average and leaves the per-slice damage in place, and nobody will remember why the constant is there in a year.

    Compare your process: did you ask what consumes the score before you talked about the model?

??? note "I5 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Normalises the report rate per impression before treating it as a regression | Compares raw report counts |
    | Trade-offs | States the trade rate explicitly and names whose decision it is | Decides unilaterally to ship or to block |
    | Depth | Proposes an integrity head in the blend plus a pre-ranking policy filter, and says what each catches | Proposes "add a filter" |
    | Communication | Says the guardrail should have been declared before the launch, and declares it now | Argues about whether 30 percent is a lot |

    Model answer: first, normalise. Reports per impression is the comparable quantity. If impressions rose 30 percent because sessions got longer, and reports per impression is flat, then the integrity rate did not change and the conversation is about volume of review work, not about ranking quality. Check the report rate on the treatment slate specifically, not on the whole app.

    Assume it is real. A single engagement objective selects for content that provokes a reaction, and reports are one of the reactions. That is metric cannibalisation, and the structural fix is in the objective, not in a post-hoc filter alone.

    What I would propose, in two parts. In the model: a multi-task ranker with an integrity head predicting the probability that this item gets reported or violates policy, entering the final blend with a negative weight, so the ranker itself prices the harm. Before the model: a hard policy filter applied at the candidate stage for anything above a policy threshold, because a weighted blend can always be outvoted by a large enough engagement score and some content must never be shown at any engagement level.

    For tomorrow's review: I would not ship at the current trade rate, and I would bring the number rather than the opinion. Present the trade explicitly: 4 percent engagement for X additional reports per million impressions, and the estimated review cost and user harm of X. Name that this is a product decision with a named owner, and that the guardrail threshold should have been agreed before the experiment ran, which is a process fix I would make regardless of what happens with this launch.

    Wrong answer: "Ship it, engagement is the north star." Reveals no guardrail model. It also fails the interview's real question, which is whether you will surface a cost that is inconvenient.

    Wrong answer: "Block it, integrity always wins." Reveals a rule applied without a trade rate. Some integrity regressions are worth some engagement gains and the organisation needs the number to decide, which is what you are there to produce.

    Wrong answer: "Add a filter for reported content." Reveals a lagging signal used as a control. Content is reported after it has been shown, so a filter on reports cannot prevent the first impressions, which is where most of the harm is.

    Compare your process: did you normalise before you interpreted?

??? note "I6 rubric, model answer and wrong answers"

    | Criterion | Strong on this problem | The tell you are Developing |
    |---|---|---|
    | Framing | Ties the spike to the deploy rather than to traffic, and uses the self-recovery as evidence | Investigates traffic patterns |
    | Trade-offs | Enumerates three warmup mechanisms and gives a distinguishing measurement for each | Names one cause and stops |
    | Depth | Adds the ML-specific step: validate the new index against a golden query set before it takes traffic | Fixes it as a generic cold-start problem |
    | Communication | Proposes one measurement before three changes | Proposes three changes at once |

    Model answer: the class is cold-start collapse from the [failure library](../#the-failure-library), and the fact that it self-heals in ten minutes without intervention is the strongest clue: nothing is permanently broken, something is filling.

    Three candidate mechanisms, each with a signal that separates it:

    - Index page cache is cold. A newly loaded index is on disk and the first queries fault it into memory. Signal: disk read rate and page fault count on the index volume spike and decay over the same ten minutes.
    - In-process feature or embedding cache is empty. Signal: per-replica cache hit rate starts near zero and climbs; the older replicas are fine throughout.
    - Model warmup. The first inferences pay lazy kernel selection, graph compilation or allocator growth. Signal: accelerator or CPU time per request is elevated only on new replicas and falls without any cache filling.

    Measure per-replica cache hit rate, per-replica compute time per request, and index volume read rate during the next deploy, then fix the one that shows up rather than all three.

    The fix set: load the index and touch it before the replica joins the load balancer, send a fixed number of synthetic warmup requests through the model at startup, and ramp traffic to new replicas instead of switching at once so only a fraction of capacity is ever cold.

    Two ML-specific additions: keep the previous index serving until the new one has passed a recall check against a golden query set, and swap per replica rather than fleet-wide, so a bad index affects a fraction of traffic and can be rolled back.

    Wrong answer: "Add more replicas." Reveals no mechanism. More cold replicas at once makes a warmup spike larger, not smaller.

    Wrong answer: "It recovers by itself, so it is acceptable." Reveals a missing user model. Ten minutes of 3 second p99 on every release is an availability cost paid every time you ship, and it silently discourages shipping.

    Wrong answer: "Deploy at night." Reveals a workaround rather than a design. It hides the symptom, leaves the fleet unable to survive an unplanned restart, and guarantees the next incident happens with nobody watching.

    Compare your process: did the self-recovery change your hypothesis, or did you ignore it?

---
## 12. Mastery check and remediation map

Seven short-answer items, one for each learning objective in Section 4, in the same order. Write or say your answer before opening the block. Producing the answer is what builds the retrieval path; recognising a correct answer does not.

**M1** (Objective 1, business objective to trainable objective plus a guardrail). You are asked to build a system that increases the time users spend watching video. Give the trainable objective, one guardrail metric, and the specific behaviour the model produces if that guardrail is removed.

??? note "Show answer"

    The business objective is retention: people who keep coming back. Watch time is a proxy chosen because it is measurable per impression and available immediately, and it is worth saying that out loud before you use it.

    Trainable objective: a per-candidate blend of predicted watch time (or predicted completion fraction) and predicted explicit satisfaction, trained as a multi-task model so that each head can be inspected and reweighted separately.

    Guardrail: reported and hidden content per thousand impressions, plus next-day return rate for users exposed to the change. Both need a threshold agreed before the experiment starts.

    Behaviour with the guardrail removed: the model learns that attention is maximised by long, autoplaying, high-arousal items with misleading thumbnails, because a viewer who feels tricked still generates watch time. It will also shorten the tail of creators, since predicted watch time is easiest to be confident about for items that already have watch history. You end up buying today's minutes with tomorrow's session, and the primary metric will look excellent while the guardrail is the only thing that shows the cost.

**M2** (Objective 2, sizing that eliminates). Catalogue of 10 million items, 128-dimensional float32 embeddings, and a 250 ms server-side budget. Give the index footprint, the per-stage latency split, and the candidate count entering the ranker. Then name one number you would delete from your answer.

??? note "Show answer"

    Index: 10e6 times 128 times 4 bytes equals 5.12 GB of vectors. Add graph overhead at roughly 300 bytes per vector, so 3 GB, for about 8 GB total. That eliminates sharding: this index fits on one node with room for a second copy during a rebuild, so a shard map is a box you should not draw.

    Latency split, using measured stage costs of the kind in Section 10: retrieval 15, filtering 5, feature fetch 15, re-ranking 8, framework overhead 10, which is 53 ms. Reserve 50 ms of headroom, because independent per-stage p99 figures do not compose into an overall p99. Ranking budget is 250 minus 53 minus 50, which is 147 ms.

    Candidates: at 0.05 ms per candidate on one replica, 147 ms buys about 2,900. I would take 2,000 and keep the margin, and I would say why.

    The number I would delete: the count of registered users. It changes nothing on this page. It does not move the index size (that is driven by items), it does not move the latency split, and it does not move the candidate count. If it later drives the size of the online user-feature store, it earns its place there and not before.

**M3** (Objective 3, batch, online or streaming). Give the rule that decides between precomputing results on a schedule, computing them at request time, and maintaining them with a streaming pipeline. Name the deciding quantity and the cost the rejected options carry.

??? note "Show answer"

    The deciding quantity is the freshness requirement in minutes, stated by whoever consumes the output, compared against how often the inputs actually change. Ask for it as a number and refuse to invent it.

    - Required freshness longer than the job interval, and most of the audience returns within that interval: precompute on a schedule. Cost: you pay for every user you compute for, including the ones who never come back, and your freshness floor is the job interval and cannot be improved without rerunning everything.
    - Freshness must be within a session (seconds to a few minutes) and peak request rate times per-request compute is affordable: compute at request time. Cost: the full serving path is now in the user's latency budget, and every model or index problem is a live incident rather than a bad table.
    - Freshness is minutes, the input is already an event stream, and only a small part of the state needs updating: streaming. Cost: the highest operational surface of the three, plus two code paths (batch backfill and stream) that must agree, which is where the skew bugs live.

    The most common real answer is a hybrid: a scheduled base plus a request-time re-rank over the precomputed candidates. It buys within-session freshness for one small model rather than for a whole pipeline.

**M4** (Objective 4, offline and online disagree). Your new ranker gains 0.01 offline AUC on a held-out day and moves online click-through by nothing at all. Its top feature by importance is "items this user clicked in the last hour", built at training time from a table that is refreshed once a day. Name the cause and the single check that confirms it.

??? note "Show answer"

    Cause: training-serving skew, specifically a join that is not point-in-time correct. At training time the feature was computed from a table containing the full day, so for a training row at 10 a.m. the "last hour" window could include clicks that happened at 2 p.m. The model learned from information that does not exist at serving time. The offline gain is leakage, and leakage does not survive contact with production.

    The single confirming check: log the exact feature vector the serving path used, then score those logged vectors offline with the same model. If the offline replay on serving-logged features reproduces the flat online result rather than the 0.01 gain, the features are the difference, not the model.

    The five causes and their confirming checks, for the general case:

    | Cause | The one check that confirms it |
    |---|---|
    | Training-serving skew | Replay serving-logged feature vectors offline and compare metrics |
    | Join that is not point-in-time correct | Recompute one training row's features using only data with a timestamp before that row's event time, and diff |
    | Position bias | Compare the model's gain on the top slot only against its gain over all slots; a gain that vanishes when position is held fixed was position |
    | Delayed labels | Recount positives in the offline set with the full label window; if the base rate rises materially, the offline set was under-labelled |
    | Evaluation slice does not match live traffic | Reweight the offline set to the live traffic mix by slice and re-score; a gain that disappears was a slice artefact |

**M5** (Objective 5, the retrieval and ranking path). For a 50 million item pool and a 200 ms server budget, give the candidate count at each stage and state the recall each stage is permitted to lose.

??? note "Show answer"

    Stages, with counts:

    | Stage | In | Out | Recall it may lose |
    |---|---|---|---|
    | Candidate generation, two sources unioned | 50,000,000 | about 3,000 after dedup | Whatever would not have reached the final slate anyway. Measure recall at 3,000 against the ranker's own top 20 on an exhaustively scored sample. Set the bar (95 percent is a defensible starting point) and report the measured number |
    | Filtering: policy, availability, already seen | 3,000 | about 2,500 | Zero on eligible items. This is a correctness stage, not a quality stage, and a filter that drops eligible items is a bug |
    | Ranking | 2,500 | top 200 | None by definition, because this stage defines the order that everything downstream is measured against |
    | Re-ranking: diversity, freshness, business rules | 200 | 20 shown | A deliberate, bounded loss. State the bound, for example no more than 2 of the ranker's top 10 may be displaced, and measure what it costs on the primary metric |

    The sentence that matters: retrieval recall is the ceiling on everything downstream. A ranker cannot promote an item that candidate generation never returned, so the retrieval recall number is the first thing to measure and the last thing to trade away quietly.

**M6** (Objective 6, sizing an experiment, and shipping on a flat primary). Baseline conversion is 6 percent, you want to detect a 3 percent relative lift, and 250,000 eligible users a day are split evenly. Give samples per arm and duration. Then name a condition under which you would ship despite a flat primary metric.

??? note "Show answer"

    delta: 0.06 times 0.03 equals 0.0018 absolute.

    n per arm: 16 times 0.06 times 0.94 divided by 0.0018 squared. Numerator 0.9024, denominator 3.24e-6, giving about 278,500 per arm.

    Duration: 125,000 per arm per day, so 2.2 days of accumulation. Run 7 days anyway, to cover a full weekly cycle, because conversion mixes differ between weekdays and weekends and a two day read is a read on two similar days.

    Shipping on a flat primary: only when the launch was pre-registered as a non-inferiority test with a stated margin, and the confidence interval on the primary excludes a loss larger than that margin. The two cases where this is the right call:

    - A cost or latency change (a distilled or quantized model that halves serving cost) whose hypothesis was always neutrality on quality. The win is on the cost line and the primary metric only has to be shown not to have moved down.
    - A guardrail improvement (fewer policy violations, better calibration) where the primary is non-inferior within the margin.

    What makes this legitimate is the pre-registration. "Flat" declared after the fact is unreadable, because a flat point estimate with a wide interval is compatible with a meaningful loss. If the margin was not agreed before the test ran, the honest answer is that the test cannot support a launch decision.

**M7** (Objective 7, mid to staff). Name the four additions that convert a mid-level ML design answer into a staff-level one on this page, and say what each one changes about a decision.

??? note "Show answer"

    - **Label delay handling.** Changes the evaluation plan. Once you say labels arrive in 45 days, you can no longer propose a weekly retrain gated on precision, so you are forced into proxy labels, label-free drift monitoring, and a capped random holdout to keep labels unbiased.
    - **Retrain cadence with its monthly cost.** Turns "we retrain regularly" into a budget line. It sizes the training cluster, sets the feature backfill schedule, and often changes the model family, because a model that takes six hours to train cannot be retrained twice a day at a price anyone will approve.
    - **A rollback criterion.** Names the metric, the threshold and the time window at which the launch is reverted without a meeting. Committing to it forces you to instrument those metrics before shipping rather than after the first incident.
    - **A degradation path.** States what the system serves when the model, the feature store or the index is unavailable: the last known good index, a popularity or recency baseline, or a cached slate. It converts a hard dependency into a soft one, and it is usually the cheapest availability win in the whole design.

    Each of the four turns a claim into a decision that someone can be held to. That, rather than extra components, is what the level difference is made of.

**Threshold.** Five of seven, with M4 and M5 among them. Those two are weighted because a candidate who cannot diagnose an offline-to-online disagreement or cannot state per-stage candidate counts has a description of an ML system rather than a design of one.

| Missed item | Reread | Redo |
|---|---|---|
| M1 | Module 2, problem framing, plus the requirements block of the social feed ranking and content moderation case studies | Take any prompt on this page and write the trainable objective, two guardrails, and the gaming behaviour for each, in ten minutes |
| M2 | Module 3, estimation for ML systems, and Module 7 on retrieval infrastructure | Problems B1 and B2, out loud, naming the eliminated option after every number |
| M3 | Module 12, serving and deployment, plus the batch, stream and online rows of the [trade-off atlas](../#the-trade-off-atlas) | Problem I2, then redo it with the freshness requirement changed to 24 hours and see which decisions flip |
| M4 | Module 5, features and the feature store, and Module 10 on offline evaluation | Problems I4 and I6, then write the five-cause table from memory |
| M5 | Module 6, retrieval then ranking | Problem B1, then draw the stage diagram for two other case studies with the counts filled in |
| M6 | Module 11, online evaluation | Problem B4, then size one experiment for a metric your own team uses |
| M7 | Module 4 on labels, Module 13 on training cost, and Module 16 on level calibration | Problem I3, then answer it again at staff level and diff the two answers line by line |

!!! warning "Scoring well right now is a weak signal"

    Passing this immediately after reading mostly measures that the page is still in working memory. The signal that predicts interview performance is the delayed check in Section 15, taken a week later with the page closed. If you score seven of seven today and four of seven next Tuesday, the honest reading of your ability is four.

---

## 13. Common mistakes

Technical mistakes cost you a follow-up. Delivery mistakes cost you the round. The last four rows are the ones that appear in debrief notes, and they are the cheapest to fix.

| The mistake | Why candidates make it | What the interviewer concludes | What to say instead |
|---|---|---|---|
| Naming a model architecture before naming the objective | Model choice is the part that was studied, and it feels like the answer | Optimises an unknown quantity, and will build something that hits a metric nobody wanted | "Before I pick a model: the business objective is retention, the trainable proxy I would use is predicted watch time blended with a satisfaction head, and the guardrail is reports per thousand impressions" |
| Ranking the whole catalogue | The two-stage split is known in the abstract but never costed | Has read about retrieval and ranking without ever holding a latency budget | "At 0.05 ms per candidate, 150 ms of ranking budget buys 3,000 items. Ten million would be 500 seconds, so retrieval is not optional here" |
| Treating AUC as the goal | It is the metric most often quoted in write-ups | Cannot connect a model metric to the decision the score feeds | "AUC cannot see calibration, and this score feeds an auction where expected value is bid times probability. I want log loss and a calibration check in the launch gate" |
| Proposing a feature store as an answer | It is a recognisable component with a name | Named a product instead of solving the consistency problem | "The problem is that the training row and the serving request must see the same value as of the same instant. That is a point-in-time join and a shared definition; a store helps me enforce it, it does not give it to me" |
| Ignoring when labels arrive | Training data is imagined as a finished table | Has not operated a model whose feedback was delayed | "Labels here are chargebacks at up to 45 days, so any metric I quote describes the model as it was 45 days ago. That is why I want drift and proxy signals as the operational monitors" |
| Adding an approximate index to a small catalogue | It is the component associated with retrieval | Adds machinery by association rather than by measurement | "Forty thousand items at 256 dimensions is a 41 MB scan, about 2 ms and exact. I would revisit at roughly 20 times this catalogue" |
| Designing training and serving as one pipeline | Diagrams in tutorials show one loop | Will ship a system whose offline and online paths silently diverge | "These are two systems with one contract between them. The contract is the feature definition and its point-in-time semantics, and I will draw it as a shared artefact rather than a shared box" |
| Promising freshness with no number | "Real time" sounds like an unambiguous improvement | Cannot price the request they just agreed to | "What is the freshness requirement in minutes, and who is the consumer that sets it? Fifteen minutes and twenty-four hours are different architectures" |
| Designing before clarifying (delivery) | Silence feels like failure, and drawing feels like progress | Will build the wrong thing quickly and confidently | "Two questions first, because they change the shape: what decision does this score drive, and how long until we know whether it was right?" |
| Monologuing through the pipeline (delivery) | Reciting a known sequence feels safe and fills time | Cannot collaborate, and the interviewer has no opening to steer | "That is the serving path. Do you want me deeper on retrieval, or on how we would know it is working?" |
| Refusing to say "I do not know" (delivery) | Believing that a visible gap is a scoring event | Bluffs, which is expensive on a real team and easy to detect | "I have not tuned a graph index myself. What I know is the memory and recall trade it makes, and here is how I would measure whether it beats a flat scan for us" |
| Defending a metric choice instead of updating it (delivery) | Reading a follow-up as an attack rather than as information | Will not update a design in a review either | "You are right that AUC hides that. That changes my launch gate, not just my monitoring. Let me redo the offline criteria" |
| Running out of time inside the model section (delivery) | Model detail is comfortable, and evaluation feels like a formality | No sense of the clock, and no evidence they can tell whether the system works | At minute 32: "I want to leave eight minutes for evaluation and failure modes, so I will stop the model discussion here and come back if there is time" |

---

## 14. Interview-day checklist

One screen. Print it or keep it open in a second window. Nothing here is new.

**Minute budget**

| Phase | 45 minute round | 60 minute round |
|---|---|---|
| Clarify, and name the business objective | 0 to 5 | 0 to 6 |
| Trainable objective, guardrails, requirements | 5 to 10 | 6 to 13 |
| Sizing, only where a number eliminates something | 10 to 14 | 13 to 19 |
| Data and labels, including when labels arrive | 14 to 20 | 19 to 27 |
| Serving path drawn: retrieval, filter, rank, re-rank | 20 to 28 | 27 to 37 |
| One deep dive (two if 60 minutes) | 28 to 37 | 37 to 50 |
| Evaluation: offline gate, online test, guardrails | 37 to 42 | 50 to 56 |
| Monitoring, failure and rollback, what you skipped | 42 to 45 | 56 to 60 |

**The first four sentences**

1. "Let me restate: you want a system that decides X for roughly Y requests, and the business outcome it serves is Z."
2. "I will spend a few minutes on the objective and the labels first, because those decide more of this design than the model does."
3. "Then the serving path with a latency budget, then how we would know it works, then how it fails."
4. "Stop me if you would rather go deep somewhere specific than have me cover all of it."

**Five clarifying questions that generalise across ML prompts**

1. What decision does this score drive, and who or what consumes it?
2. Where do labels come from, and how long after the event do they arrive?
3. What is the freshness requirement in minutes, and who set it?
4. What is the serving latency budget, and is it measured at the client or at the gateway?
5. What must never happen, even at the cost of the primary metric?

**Whiteboard drawing order**

```
DRAW IN THIS ORDER. WITHHOLD THE REST UNTIL A NUMBER FORCES IT.

   1              2              3              4
 +--------+    +---------+    +---------+    +---------+
 | CLIENT | -> | CANDID. | -> | RANKER  | -> | SLATE   |
 |        |    | SOURCE  |    | one box |    | SHOWN   |
 +--------+    +---------+    +---------+    +---------+
                                   ^
                                   | 5, after a latency number
                              +---------+
                              | FEATURE |
                              | LOOKUP  |
                              +---------+

 NOT YET: ANN index, feature store, streaming pipeline,
 training loop, model registry, re-ranker, cache, second model.
 Caption: four boxes first, then feature lookup, then nothing
 else until a stated number makes it necessary.
```

Draw the serving path left to right first, because it is what the prompt asked about. Add the training and label loop underneath only when you reach labels, and draw it as a separate flow that meets serving at one shared artefact (the feature definition), not as a circle. Erasing a box you drew in minute three reads as guessing; adding one in minute twenty after a number reads as design.

**Three recovery scripts**

| Situation | Say this |
|---|---|
| You are stuck | "Let me say what I am optimising for and give the two options I can see: A trades freshness for cost, B the reverse. I will take A because the freshness requirement is fifteen minutes, and I will flag it as reversible." |
| The interviewer disagrees | "That is a case I did not price. If that happens, this stage breaks because the labels are censored, so I would change the evaluation plan rather than the model. Does that address it?" |
| You are running out of time | "I have about six minutes. I would rather spend them on how we would know this works and how it degrades than on finishing this dive, because I think that is the higher-value part. Say if you would prefer otherwise." |

**Before you finish**

- State the two trade-offs you actually made and what each one cost.
- State what you deliberately skipped, in one sentence each.
- Name the single metric you would watch in week one and the value that would make you roll back.
- Name the degradation path: what the user sees when the model or the index is unavailable.
- Name the decision that is most expensive to reverse (usually the label definition or the randomisation unit).

---

## 15. Study schedule and spaced review

### 15a. Three plans, each defined by what it drops

Day 1 is whatever day you start. The skip column is the part of this section that is actually hard to write, and it is the part worth reading.

**One week (about 9 to 12 working hours)**

| Day | Do | Skip |
|---|---|---|
| 1 | Modules 1 to 3, then problems B1 and B2 | Nothing yet |
| 2 | Modules 4 and 5, then the video recommendation case study end to end | Its second deep dive; come back only if time allows |
| 3 | Modules 6 and 7, then problem B5 | Index family comparison beyond graph against inverted-file; you need the trade, not the taxonomy |
| 4 | Modules 9 and 10, then the ad click prediction case study | Module 8 on model choice, unless your target role is modelling-heavy |
| 5 | Modules 11 and 12, then problems B4 and I4 | Module 13 on training cost |
| 6 | Module 14, then the fraud detection case study, then problem I3 with a timer | The people you may know and semantic search case studies |
| 7 | Modules 15 and 16, then Section 12, then Section 14 | Everything unreached. Do not start new material the day before a round |

**One weekend (about 5 to 7 working hours)**

- Saturday morning: Modules 2, 3 and 6. Objective framing, sizing and the retrieval-plus-ranking split affect every prompt on this page and every prompt you will be asked.
- Saturday afternoon: one case study end to end, out loud, with a timer. Take video recommendation if you are early career, ad click prediction otherwise, because calibration and delayed feedback are where senior rounds go.
- Sunday morning: Modules 5, 10 and 11 (skew, offline evaluation, online evaluation), then problems I4 and I6.
- Sunday afternoon: Sections 12, 13 and 14. Rehearse the first four sentences until they are automatic.
- Skip entirely: Modules 7, 8 and 13, every second deep dive, and seven of the eight case studies. One case study done properly beats five skimmed, and the difference shows in an interview within two minutes.

**One evening (about 2 to 3 hours)**

- 0:00 to 0:25, Section 14 and Module 1. Get the clock and the opening.
- 0:25 to 1:05, Module 2 and Module 6. Objective, guardrail, and the two-stage split with a budget. These are the fastest visible upgrades to an ML answer.
- 1:05 to 2:00, one case study, requirements and serving path only, no deep dives.
- 2:00 to 2:30, Section 13, read once, then Section 14 again.
- Skip: every other module, every deep dive, the mastery check, and all interleaved problems except I1. If your round is tomorrow, extra breadth will not be retrievable under pressure and rehearsed delivery will be.

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
Q :: What is the first number that forces a retrieval stage to exist?
A :: Ranker cost per candidate times catalogue size against the
     ranking budget. At 0.05 ms an item, 10 million is 500 seconds.

Q :: Retrieval recall is the ceiling on what?
A :: Everything downstream. A ranker cannot promote an item that
     candidate generation never returned.

Q :: Bytes for N vectors of D float32 dimensions, plus a graph index?
A :: N x D x 4 for vectors, plus roughly 300 bytes per vector for
     the graph at 32 links per node.

Q :: Why does int8 quantization need a rescoring stage?
A :: It trades recall you must measure. Retrieve deeper, then
     rescore the top few hundred with full precision vectors.

Q :: Samples per arm for a two-proportion test at 80 percent power?
A :: About 16 x p x (1 - p) / delta squared, from (1.96 + 0.84)
     squared doubled for two arms.

Q :: Quartering the effect size you want to detect does what to n?
A :: Multiplies it by 16, because n scales as one over delta squared.

Q :: Why can AUC rise while auction revenue falls?
A :: AUC is invariant to monotone transforms, so it cannot see
     calibration, and the auction uses the score as a probability.

Q :: One check that confirms training-serving skew?
A :: Replay the serving-logged feature vectors offline and compare
     the metric against the offline claim.

Q :: What makes a join point-in-time correct?
A :: Every feature value is computed only from data whose timestamp
     precedes the label event's timestamp.

Q :: What does a 45 day label delay do to your reported precision?
A :: It describes the model as it was 45 days ago, so monitoring has
     to run on drift and proxy labels instead.

Q :: Why does blocking transactions censor your training labels?
A :: Blocked events never produce outcomes, so the model stops
     seeing the region it is most confident about.

Q :: What decides batch against online against streaming serving?
A :: The freshness requirement in minutes, stated by the consumer,
     against how often the inputs change.

Q :: What does precomputing for all users cost that nobody counts?
A :: Compute for the inactive fraction. If 20 percent return daily,
     80 percent of the job produces lists nobody reads.

Q :: When is an approximate index over-engineering?
A :: When a flat scan fits the budget. 40k items at 256 dims is a
     41 MB scan, about 2 ms, and exact.

Q :: Four things a staff ML answer adds over a mid one?
A :: Label delay handling, retrain cadence with monthly cost, a
     rollback criterion, and a degradation path.
```

### 15d. Delayed self-check

Answer these from memory a week after you finish, with this page closed. Write the answers down before you check anything.

1. Take an ML prompt you have not studied here. State the business objective, the trainable objective, one guardrail, and the behaviour the model produces if the guardrail is dropped.
2. Draw its serving path with a candidate count at every stage that sums to a latency budget you state, then name which stage's recall you would measure first.
3. Describe an offline-to-online disagreement you would expect for that system, name the cause, and name the single check that confirms it.

Twenty minutes, no notes. If you can do all three, the material is retrievable under pressure. If you cannot, the step where you stalled is the one to redo, not the whole page.

---
## 16. Reference block

!!! info "This section teaches nothing new"

    Everything below appeared earlier with its derivation. This is the version to reread the morning of the round. If a row here is the first place you have met an idea, go and read the module it came from rather than memorising the row.

### 16a. Decision tables

**Serving mode**

| If | Choose | Because | Do not choose it when |
|---|---|---|---|
| Required freshness is longer than the job interval and most of the audience returns within it | Scheduled precompute into a key-value store | Cheapest per served result, and a bad run is a bad table rather than a live incident | A large fraction of the audience is inactive, so you are paying to compute results nobody reads |
| Freshness must be within a session, and peak rate times per-request compute is affordable | Online inference on the request path | The result reflects the request that just arrived | Peak request rate times model cost exceeds your serving budget, or the model cannot meet the latency split |
| Freshness is minutes, the input is already an event stream, and only part of the state changes | Streaming update of a precomputed state | Bounded work per event rather than per user | You would be running two implementations of the same feature logic and cannot fund keeping them in agreement |
| Freshness matters only for ordering, not for the candidate set | Scheduled base plus a request-time re-rank | Buys within-session responsiveness for one small model | The stale candidate set itself is the complaint |

**Retrieval index**

| If | Choose | Cost you must say out loud | Below this it is over-engineering |
|---|---|---|---|
| Catalogue times dimension times 4 bytes fits comfortably inside the retrieval budget as a scan | Flat exact scan | None, and it is exact | Always prefer it while the scan is under roughly a fifth of the retrieval budget |
| Millions to hundreds of millions of vectors, high recall required, memory available | Graph index | Roughly 300 bytes per vector of graph on top of the vectors, and a rebuild plan | Under about a million vectors, where a scan or a simple partitioned index is cheaper to operate |
| Memory bound, willing to rescore | Quantized vectors plus full-precision rescoring of the top few hundred | Recall you must measure on your own queries, not quote | When the uncompressed index already fits with room for a second copy |
| Queries carry hard filters that remove most of the corpus | Partitioned or filtered index, filter before search | Partition skew, and a filter applied after search silently destroys recall | When filters remove a small minority of items and post-filtering still returns enough |
| Query and document vocabularies differ, but exact terms still matter | Hybrid sparse plus dense with a fusion rule | Two indexes to keep in sync and one fusion parameter to tune | When lexical alone already has an acceptable zero-result rate |

**Model family**

| If | Choose | The number that decides it | Do not choose it when |
|---|---|---|---|
| Tabular features, hundreds of columns, tens of millions of rows, tight latency | Gradient boosted trees | Feature count and per-candidate inference budget. Trees hit sub-millisecond per candidate at modest depth | The signal is in raw text, image or audio, where trees need someone to hand-build the representation |
| Retrieval over a large corpus with a latency budget that forbids scoring pairs | Two-tower model with precomputed item vectors | Catalogue size against the ranking budget from B1 | The catalogue is small enough to score exhaustively, where a single richer model is simpler and better |
| Several objectives that trade against each other | Multi-task model with one head per objective and an explicit blend | The trade rate between objectives, which is a product number | Nobody will own the blend weights, in which case you have hidden a product decision inside a model |
| A large model wins offline but breaks the latency budget | Distillation into a smaller student, or a cascade with a cheap first pass | Per-candidate cost against the ranking budget | The quality gap survives distillation, in which case reduce the candidate count instead |

**Offline metric by task**

| Task | Primary | Also required | Why the second one |
|---|---|---|---|
| Binary prediction feeding a threshold | Precision and recall at the operating threshold | Precision-recall curve, and the slice table | A single threshold hides what happens when prevalence moves |
| Probability feeding an expected-value calculation | Log loss | Calibration curve and predicted-over-observed ratio, overall and per slice | Ranking metrics are invariant to monotone transforms and cannot see calibration |
| Ranking a slate | NDCG or mean average precision at the slate size | Retrieval recall at the candidate count | The ranker's ceiling is what retrieval returned |
| Regression with asymmetric cost | Quantile loss at the promised quantile | Error by condition slice | Average error hides the tail that generates complaints |
| Anything with delayed labels | The metric on a fully matured cohort | The same metric on partial cohorts, plotted against maturity | Otherwise you compare a mature model against an immature one |

**Monitoring, by what labels you have**

| Label availability | Monitor | Alert on |
|---|---|---|
| Labels within minutes | The business metric directly | The metric itself, with a window long enough to clear noise |
| Labels within days | Proxy labels plus the business metric on a lag | The proxy, with the lagged metric as confirmation |
| Labels in weeks or months | Feature and score distribution drift, which need no labels; plus a deliberately labelled sample | Score distribution shift and input drift, because the metric cannot tell you anything current |
| Labels censored by the model's own actions | All of the above, plus a capped random holdout | The holdout's outcome rate, and the size of the region the model no longer observes |

### 16b. The numbers this course uses, and where each comes from

Inputs marked as assumed are chosen because they are typical order-of-magnitude figures for the scenario as stated. Every derived line stays correct as a method when you substitute the interviewer's number.

| Number | Derivation | What it eliminates |
|---|---|---|
| 0.05 ms per ranked candidate | 25 ms to score 500 candidates on one replica (stated as a measurement), divided by 500 | Ranking a catalogue. Ten million items becomes 500 seconds |
| 3,000 candidates into the ranker | 150 ms of ranking budget divided by 0.05 ms | Any candidate generation stage that returns tens of thousands |
| 150 ms of ranking budget | 300 ms client-side p99, minus 40 ms assumed network and TLS, minus 50 ms of measured fixed stages, minus 60 ms of reserved headroom | Cross-encoders as the main ranker, since at 2 ms per item they buy only 75 candidates |
| 205 GB of vectors | 200e6 items times 256 dimensions times 4 bytes | Holding float32 vectors for this catalogue on one machine |
| about 300 bytes per vector of graph | 64 base-layer neighbour identifiers at 4 bytes, rounded up for upper layers | Sizing an index from vector bytes alone |
| Four index shards | 265 GB total divided by 128 GB usable per node is 2.07, so three is the arithmetic floor and four allows one spare plus a second copy during a swap | Three shards, which cannot rebuild or survive a node loss |
| 111 GB after int8 quantization | 200e6 times 256 times 1 byte, plus the same 60 GB of graph | The claim that quantization removes the need to think about node count |
| 144 dollars per full retrain | 8 accelerators times 6 hours is 48 accelerator-hours, at an assumed 3 dollars per accelerator-hour on demand | Nothing on its own. It earns its place only next to the cadence comparison below |
| 4,320 dollars a month for daily retraining | 30 times 144 dollars | Hourly retraining at 103,680 dollars a month for at most 0.002 of AUC |
| 2 days of model life | A 0.004 AUC launch bar divided by 0.002 AUC of decay per day, both stated as measurements | Weekly retraining, which gives back 0.014 AUC |
| 16 in the sample-size formula | (1.96 plus 0.84) squared is 7.84, doubled because two arms each carry variance, then rounded up | Guessing experiment duration |
| 960,000 users per arm | 16 times 0.04 times 0.96 divided by 0.0008 squared | A two day read on a 2 percent relative lift |
| 77 days for a 0.5 percent lift | Quartering delta multiplies n by 16, so 15.36 million per arm at 200,000 per arm per day | A/B testing as the measurement design for very small effects |
| 3 million item-feature reads per second | 1,500 requests per second times 2,000 candidates | One remote lookup per candidate |
| 800 MB item feature table | 5e6 items times 40 features times 4 bytes | A remote feature store on the per-candidate path, since the whole table fits in the ranker |
| 41 MB scanned per query | 40,000 items times 256 dimensions times 4 bytes | An approximate index for a small catalogue, since at an assumed 20 GB per second of usable memory bandwidth this is about 2 ms and exact |
| 90,000 two-hop candidates | An assumed average degree of 300, squared, before deduplication | Scoring all users pairwise, which for 1e8 users would be 1e16 pairs |
| 80 percent of a precompute job wasted | 1 minus an assumed 20 percent daily active fraction, applied to a job that computes for all registered users | Precomputing for the full registered base as a default |

### 16c. ASCII glyph legend

```
ASCII CONVENTIONS USED IN EVERY DIAGRAM ON THIS PAGE

 +--------+   solid box: an online component on the request path,
 | NAME   |   something you run and can page an engineer about
 +--------+

 +========+   double edge: a stored artefact that survives restart
 | NAME   |   (embedding index, feature table, model file, labels)
 +========+

   --->       online path, a user is waiting for the reply
   ==>        offline or scheduled path, nobody is waiting
   <-->       two copies of one definition that must agree
   |   v      vertical continuation of the same path

 Caption: box style says whether a component holds a stored
 artefact; arrow style says whether a user is waiting on it.
```

### 16d. One-page summary cards

Ten cards, one per case study, in the order the case studies appear.

**Card 1. Video recommendation**

| Field | Content |
|---|---|
| Prompt | Design the recommendation system for a video home screen |
| Objective and guardrail | Predicted watch time blended with a satisfaction head; guardrails are reports per thousand impressions and next-day return rate |
| Number that decides | 0.05 ms per candidate against a 10 million catalogue is 500 seconds, which forces retrieval and ranking to be separate stages |
| V0 | One popularity plus recency list per locale, cached, no personalisation |
| Breaking point | Engagement flat for users with long histories; observe as the metric split by history-length decile, not overall |
| V1 | Two-tower retrieval into a multi-task ranker, with a fixed exploration slot budget per slate |
| Deep dive one | The exploration budget: what fraction of the slate is reserved for uncertain items, and how that cost is measured rather than assumed |
| Deep dive two | The popularity feedback loop and position bias, corrected with inverse propensity weighting on logged impressions |
| Failure class | Popularity feedback loop, and cold-start starvation of new items and creators |
| Staff delta | Retrain cadence with its monthly cost, degradation to the popularity list when the index is unavailable, and an exploration loss budget with a named owner |

**Card 2. Social feed ranking**

| Field | Content |
|---|---|
| Prompt | Design the ranking system for a social home feed |
| Objective and guardrail | A weighted blend over several predicted engagement events; guardrails are reports per impression and the share of impressions from a small set of producers |
| Number that decides | The trade rate between the primary engagement metric and the integrity metric, which is a product number and must be obtained, not invented |
| V0 | Reverse chronological over the accounts a user follows |
| Breaking point | Users following many accounts see only the highest-volume producers; observe as session length falling with followee count |
| V1 | Multi-task ranker with an integrity head entering the blend negatively, plus a hard policy filter applied before ranking |
| Deep dive one | Multi-objective blending: why the weights are a stated trade rate with an owner rather than a tuned constant |
| Deep dive two | Integrity constraints: why a hard pre-ranking filter is needed in addition to a negative weight, since a large enough engagement score outvotes any blend |
| Failure class | Metric cannibalisation, and feature computation hot spots on very large producers |
| Staff delta | Guardrail thresholds pre-registered before the experiment, degradation to chronological, and the review-cost consequence of the chosen threshold |

**Card 3. Ad click prediction**

| Field | Content |
|---|---|
| Prompt | Design click prediction for an ad auction |
| Objective and guardrail | Calibrated click probability; guardrails are the predicted-over-observed ratio per slice and advertiser delivery against target |
| Number that decides | Expected value per impression is bid times predicted probability, so a calibration error of a given factor misallocates delivery by that same factor |
| V0 | Logistic regression on hashed categorical features, retrained daily, no calibration layer |
| Breaking point | Conversions arrive over a multi-day window, so a same-day label undercounts positives; observe as the predicted-over-observed ratio drifting below one |
| V1 | Delayed-feedback modelling with an explicit attribution window, plus a per-slice calibration layer fitted on a recent holdout |
| Deep dive one | Auction calibration: why AUC cannot see it, and what a calibration curve and a per-slice ratio actually show |
| Deep dive two | Delayed conversions: how a truncated window biases the labels, and the correction |
| Failure class | Systematic under-calibration from window truncation, and an omitted correction after negative downsampling |
| Staff delta | Log loss and calibration error added to the launch gate, calibration monitored per slice, and a rollback trigger on the ratio rather than on revenue |

**Card 4. Semantic and visual search**

| Field | Content |
|---|---|
| Prompt | Design search over a large corpus where the query wording differs from the document wording |
| Objective and guardrail | Relevance at the top of the slate; guardrails are the zero-result rate and latency at p99 |
| Number that decides | Index footprint as N times D times 4 bytes plus roughly 300 bytes per vector of graph, against usable memory per node |
| V0 | Lexical inverted index only, ranked by a standard term-weighting score |
| Breaking point | Queries phrased unlike the corpus return nothing; observe as the zero-result rate rising with query length |
| V1 | Hybrid sparse plus dense retrieval with a fusion rule, then a cross-encoder re-rank over the top 50 |
| Deep dive one | Index engineering: parameters, the measured recall against latency curve, refresh cadence and hot swap |
| Deep dive two | Re-embedding migration: building a shadow index, validating on a golden query set, and swapping per replica |
| Failure class | Stale index after a model change, and recall collapse when hard filters are applied after search instead of before |
| Staff delta | Embedding version stamped on every vector and logged with every query, a golden set gating the swap, and cost per thousand queries |

**Card 5. Harmful content moderation**

| Field | Content |
|---|---|
| Prompt | Design a system that detects and acts on policy-violating content |
| Objective and guardrail | Per-policy violation probability; guardrails are the false removal rate and the appeal overturn rate |
| Number that decides | Review capacity. Enqueued volume equals traffic times the fraction above the review threshold, and that must fit the reviewer headcount |
| V0 | Exact and near-duplicate matching against known-bad content, plus user reports |
| Breaking point | Novel violating content evades matching; observe as the share of enforcement that originates from reports rather than proactive detection |
| V1 | Multimodal scoring with two thresholds: an automatic action threshold and a lower enqueue-for-review threshold |
| Deep dive one | Threshold economics: precision against recall priced in review hours and in the cost of a wrongful removal |
| Deep dive two | The appeal loop as a label source, and the selection bias it carries because only some users appeal |
| Failure class | Adversarial adaptation, and censored feedback because removed content generates no further signal |
| Staff delta | Reviewer capacity as a hard constraint on the threshold rather than an afterthought, per-policy thresholds, and an appeal service level with an owner |

**Card 6. Fraud detection**

| Field | Content |
|---|---|
| Prompt | Design fraud scoring in the card authorisation path |
| Objective and guardrail | Probability of fraud at authorisation; guardrails are the false decline rate and authorisation latency |
| Number that decides | Label delay. Chargebacks arrive up to 45 days later, so every labelled metric describes the model as it was 45 days ago |
| V0 | A rules engine with hand-written thresholds, evaluated inline |
| Breaking point | Rules are enumerable by an attacker; observe as a rising share of losses from transactions scoring just below a rule threshold |
| V1 | A model in the authorisation path within a strict latency budget, with rules retained as a hard-block layer above it |
| Deep dive one | Label delay and censored labels: proxy signals, drift monitoring without labels, and a capped random holdout to keep the label set unbiased |
| Deep dive two | Adversarial adaptation: which features an attacker can move cheaply, and why the cheap ones decay fastest |
| Failure class | Censored feedback, and review queue growth that sustains itself during an attack |
| Staff delta | An expected loss budget for the holdout with a named owner, degradation to rules-only when the model is unavailable, and a stated authorisation latency budget |

**Card 7. Delivery time prediction**

| Field | Content |
|---|---|
| Prompt | Design the estimated arrival time shown to a customer at order time |
| Objective and guardrail | A promised quantile of the duration distribution; guardrails are the observed late rate and the over-quote rate |
| Number that decides | The asymmetry of cost. Late by ten minutes and early by ten minutes are not equally expensive, which is why the loss is a quantile loss rather than a squared error |
| V0 | Historical median duration by route and hour of day |
| Breaking point | Tail conditions such as weather or a local event blow the promised quantile; observe as error by condition slice, not overall |
| V1 | A model over route, time, live traffic and supply features, trained on the promised quantile, with the uncertainty shown to the user |
| Deep dive one | Choosing which quantile to promise, and the direct relationship between that choice and the observed late rate |
| Deep dive two | The physical feedback loop: the quote changes courier and customer behaviour, so the model's own output is inside its future training data |
| Failure class | Self-reinforcing feedback through the quote, and distribution shift in new markets with no history |
| Staff delta | The promised quantile framed as a product decision with an owner, degradation to the historical median table, and slice monitoring by market and condition |

**Card 8. People you may know**

| Field | Content |
|---|---|
| Prompt | Design connection suggestions in a social graph |
| Objective and guardrail | Probability that an invitation is sent and accepted; guardrails are the invitation decline rate and the fraction of suggestions a user reports as inappropriate |
| Number that decides | Two-hop candidate volume is roughly the square of average degree, so an assumed degree of 300 gives about 90,000 candidates per user, which eliminates any pairwise approach over the full user base |
| V0 | Rank the two-hop neighbourhood by count of shared connections |
| Breaking point | Very high-degree nodes generate enormous two-hop sets and appear in everyone's candidates; observe as candidate set size by degree percentile |
| V1 | Cap the contribution of any single intermediate node, add graph embeddings for retrieval, and rank with interaction features |
| Deep dive one | Graph features and the hub-node cap: why an uncapped shared-connection count is dominated by popular accounts |
| Deep dive two | Privacy constraints: which signals may be used at all, and why some highly predictive signals must be excluded regardless of their lift |
| Failure class | Hot-node fan-out in candidate generation, and a privacy incident caused by an inferable edge |
| Staff delta | A written data-use decision per feature, degradation to shared-connection ranking, and a suppression list with an audit trail |

**Card 9. Feature store as a platform prompt**

| Field | Content |
|---|---|
| Prompt | Design a feature store for several modelling teams |
| Objective and guardrail | One definition of a feature serving both training and inference; guardrails are the measured skew rate and the freshness of each online feature |
| Number that decides | Online read load as requests per second times candidates per request, which is what decides whether item features are fetched remotely or co-located |
| V0 | A table in the application database plus a nightly job, and no store at all |
| Breaking point | Two teams write two implementations of the same feature and the training and serving values diverge; observe as a skew check failing on a feature nobody changed |
| V1 | A registry of feature definitions with one implementation compiled to both a batch and an online path, point-in-time correct training set generation, and a version stamp logged with every prediction |
| Deep dive one | Point-in-time correctness: how a training set is assembled so that no feature value postdates its label event |
| Deep dive two | The online read path: batched multi-get against co-location, and what co-location costs in staleness |
| Failure class | Silent training-serving skew, and a backfill that rewrites history and invalidates every model trained before it |
| Staff delta | Definition ownership and a deprecation process, the skew check promoted to a first-class service level indicator, and a migration path from the tables that exist today |

**Card 10. Brownfield: fresher recommendations without doubling cost**

| Field | Content |
|---|---|
| Prompt | An existing daily-batch recommender must reflect the last 15 minutes of activity, and the bill cannot double |
| Objective and guardrail | Unchanged primary metric; guardrails are serving latency and the infrastructure cost line itself |
| Number that decides | The daily active fraction. If an assumed 20 percent of registered users return on a given day, 80 percent of the precompute job produces lists nobody reads |
| V0 (what exists) | Nightly features, nightly training, a precomputed top 500 per user in a key-value store, read directly by the app |
| Breaking point | Within-session staleness; observe as the click rate on the slate immediately after a strong in-session signal, compared against the rate at session start |
| V1 | Keep the batch base, add a request-time session re-ranker over the precomputed 500, and fund it by restricting precompute to recently active users |
| Deep dive one | Decomposing freshness into feature staleness, candidate staleness and model staleness, and showing that only one of them is the complaint |
| Deep dive two | Migration mechanics: shadow the re-ranker, ramp by cohort, and set the rollback trigger on latency and guardrails rather than on the engagement metric |
| Failure class | Dual feature code paths drifting apart, and cold caches on the new re-ranker at every deploy |
| Staff delta | The cost model shown as a before-and-after line, an explicit list of what is deliberately not changing, and a rollback criterion agreed before the ramp starts |

---

## 17. What this course does not cover

Naming our limits is cheaper than pretending we have none, and it stops you studying the wrong thing.

| Not covered | Why | Where to go instead |
|---|---|---|
| The modelling round: feature engineering by hand, hyperparameter search, loss function derivations | A different round with a different rubric. Doing it here would double the page and improve no design answer | The [machine learning question bank](/Interview-Questions/Machine-Learning/) on this site, and the scikit-learn user guide for the mechanics |
| Deep learning theory and architecture derivations | We choose model families by their cost and latency profile, which is the interview depth. The mathematics is a book | Deep Learning, Goodfellow, Bengio and Courville, MIT Press, 2016, readable free at deeplearningbook.org |
| Systems built around large language models: retrieval augmentation, agents, inference serving internals | Enough material for its own arc, and the constraints are different (tokens, context budgets, prefill against decode) | Our [Generative AI System Design](../generative-ai-system-design/) course in this section |
| Distributed systems fundamentals: consensus, replication, partitioning, multi-region | Assumed as background here rather than taught. An ML system sits on top of them | Our [Modern System Design](../modern-system-design/) course, and Designing Data-Intensive Applications, Martin Kleppmann, 1st edition, O'Reilly, 2017 |
| Experimentation beyond sizing and reading a single test: variance reduction, sequential testing, interference | We teach enough to size a test and to refuse to launch on a bad one. The field is much larger | Trustworthy Online Controlled Experiments, Kohavi, Tang and Xu, Cambridge University Press, 2020 |
| Causal inference and observational estimation | Adjacent, large, and rarely what an ML design round asks. Where we touch it (position bias, propensity weighting) we stay at design depth | Causal Inference: The Mixtape, Scott Cunningham, Yale University Press, 2021, free online at mixtape.scunning.com |
| Distributed training internals: parallelism strategies, collective communication, checkpoint formats | Cadence and cost decide architecture; the internals of the training job usually do not | The current documentation for whichever framework you use, checked on the day, because these APIs move faster than a page can be reviewed |
| Data engineering and warehouse modelling | The pipeline that lands the data is upstream of everything here, and it is a discipline | The Data Warehouse Toolkit, Kimball and Ross, 3rd edition, Wiley, 2013 |
| Fairness auditing, regulated model governance and model risk management | Genuinely important and genuinely jurisdiction-specific. Publishing generic advice here would be worse than publishing none | Your own legal and policy function, plus the regulator's own published guidance for your sector |
| Privacy techniques: differential privacy, federated learning, secure aggregation | We name privacy as a constraint in the graph and moderation case studies but do not teach the mechanisms | The published papers and reference implementations for the specific technique, checked for their date |
| Tool selection: which feature store, which vector database, which orchestrator | Product names and pricing move faster than we can review a page, and a design answer should not turn on a brand | Vendor documentation on the day you need it, and ann-benchmarks.com for vector index comparisons, which publishes reproducible runs with dates |
| MLOps practice as a discipline: CI for models, artefact promotion, on-call for ML | We cover the parts a designer must state. Operating is a job, not a section | Designing Machine Learning Systems, Chip Huyen, 1st edition, O'Reilly, 2022, and [Rules of Machine Learning](https://developers.google.com/machine-learning/guides/rules-of-ml) by Martin Zinkevich, published free by Google in its machine learning guides, link checked 1 August 2026 |
| Named engineering patterns catalogued at implementation depth | We teach the decision; the catalogue form is useful separately | Machine Learning Design Patterns, Lakshmanan, Robinson and Munn, 1st edition, O'Reilly, 2020 |
| Behavioural rounds and company-specific loop formats | Loop structures change per team and per quarter, and unverifiable claims about them are exactly what we refuse to publish | Ask your recruiter for the current format, in writing |

!!! note "On anything we cite"

    We name the edition and the year so you can tell what has aged. If a link here is dead or a newer edition exists, open an issue and it is fixed in the next review pass rather than sitting behind an unverifiable "updated recently" badge.

---

## 18. Provenance

**Last reviewed:** 1 August 2026. A page whose last-reviewed date is more than nine months old is flagged for review rather than quietly left in place.

**Changelog, most recent first**

| Date | Change |
|---|---|
| August 2026 | First publication: sixteen modules, eight case studies with the full arc and level deltas, eleven practice problems with public rubrics, and the mastery check mapped one to one against the learning objectives |
| August 2026 | Reference block separated from the teaching sections after review, so a reader returning for one decision table does not have to scroll through prose to reach it |
| August 2026 | Every figure in Sections 10, 12 and 16 re-derived from inputs stated in the same section; figures that could not be derived were relabelled as assumptions with the reason each is reasonable, and two that eliminated no design option were deleted |

**Found a problem?** Open an issue at [github.com/datascienceinterviews/datascienceinterviews.github.io/issues](https://github.com/datascienceinterviews/datascienceinterviews.github.io/issues). Corrections, disputed numbers, dead links and better derivations are all in scope. Errors get fixed in days, and the changelog above records what changed.

All content on this page is original. Research informed which topics belong here; no prose, framework, acronym, example, table or diagram is derived from any paid course. Primary sources are cited inline with their edition and year, and any claim we could not source is either labelled as an assumption or is not on the page.
