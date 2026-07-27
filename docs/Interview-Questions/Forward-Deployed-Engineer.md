---
title: Forward Deployed Engineer (FDE) Interview Questions
description: 100+ Forward Deployed Engineer interview questions covering customer deployments, data integration, LLM systems, and stakeholder management
---

# Forward Deployed Engineer (FDE) Interview Questions

<!-- [TOC] -->

A Forward Deployed Engineer is an engineer embedded with a customer and held accountable for a working outcome inside that customer's environment, rather than for a feature shipped into a shared product. The role originated at Palantir, which also supplied the vocabulary the rest of the industry borrowed: product engineers are Devs, forward deployed engineers are Deltas, and the two are separated by scope rather than seniority.

The title exploded in 2025 and 2026. Indeed data shared with Business Insider put FDE postings at 643 in April 2025 and 5,330 in April 2026. The funded org-chart moves arrived within nine weeks of each other: OpenAI's Deployment Company (11 May 2026), Anthropic's joint venture Ode (4 May 2026), an AWS forward deployed organization (30 June 2026), and Microsoft's Frontier Company (2 July 2026). Loop shapes are described in reasonable detail, by a mix of published job descriptions and candidate reports, for Palantir, OpenAI, Google Cloud, Databricks, Sierra, Cognition and LangChain. Many more companies now hire under the title, or under a close equivalent, without a well-described process: Anthropic (whose customer-facing engineering roles were posted under Applied AI as of mid-2026), Glean, ElevenLabs, Cursor, Cohere, Mistral, Scale AI, Decagon, Baseten, Ramp, Snowflake, Anduril and Applied Intuition.

Prepare for the fact that two genuinely different jobs now wear the same title. The classic Palantir-style FDE is a data integration and deployment engineer: terabyte-scale pipelines, ontology and semantic modelling over messy customer sources, access control, on-prem and air-gapped installs, on-call. The AI-lab FDE is a full-stack LLM product engineer with a customer attached: discovery, eval design, RAG and agent architecture, guardrails, production rollout, adoption. This page is organized to cover both. It runs through role positioning, data and integration, deployment and operations, AI and LLM systems, customer craft and scoping, and behavioral and situational questions, then closes with a 100-question quick reference table and a two week preparation plan.

---

## What Interviewers Are Actually Testing

FDE loops are not standard software engineering loops with a customer flavour. The rounds candidates report failing on are the ones with no LeetCode analogue: Palantir's Decomposition and Learning rounds, and the AI-lab eval question. Coding rounds are reported at LeetCode easy to medium with unusual emphasis on writing tests and on completeness over optimality.

| Competency area | What the round actually probes | Where candidates report seeing it |
|---|---|---|
| Decomposition | Turning a vague business goal into a P0 scope, data model, KPIs and an executive summary. Not a FAANG system design round | Palantir Decomposition, Databricks Decomposition |
| Learning | Absorbing unfamiliar code or documentation live and extending it inside the hour, thinking out loud | Palantir Learning, LangChain codebase task |
| Re-engineering and debugging | Hypothesis-driven debugging of a subtle logic bug in code you did not write, without rewriting it | Palantir Re-engineering, Sierra React debugging |
| Coding | Correct, tested, complete solutions rather than optimal ones. SQL and paginated REST work alongside algorithms | Palantir online assessment and the CodePair or Karat screens candidates describe, Databricks notebook |
| Data and integration | Connectivity, CDC prerequisites, entity resolution, idempotency, data health checks | Palantir, Databricks, Google Cloud |
| Deployment and operations | Customer VPC, air-gapped constraints, version skew, tenancy, blast radius, security review | Palantir, Sierra, Google Cloud |
| AI and LLM systems | RAG design, agent versus workflow, tool design, guardrails, latency and cost levers | OpenAI, Anthropic, Cohere, Cursor |
| Evaluation | The single differentiating AI-lab question: how do you know the system is actually working | OpenAI, Anthropic, Glean, Sierra |
| Customer craft | Discovery before design, scoping by subtraction, holding a line without breaking a relationship | Discovery role plays across AI-lab loops, OpenAI solution design, Palantir Deployment Strategist |
| Presentation | Explaining technical work to a non-technical or executive audience, live | LangChain, Sierra, Cognition, OpenAI video walkthrough |

**The two loop shapes to prepare for.** Loop structures below are drawn from publicly shared candidate accounts and from live job postings. Processes change, and companies tailor rounds to the role, so treat these as a preparation guide rather than a guarantee and confirm the current format with your recruiter.

```
PALANTIR FDSE (round-name driven, code heavy)
  The shape candidates consistently describe:
    Recruiter screen (~30 min)
    -> online assessment, reported as HackerRank at 75-90 min:
       coding + SQL + a paginated REST task, and/or a live
       CodePair or Karat screen
    -> ~3 x 60-min rounds reportedly drawn from a pool:
         Decomposition | Learning | Re-engineering | Coding | System Design
    -> hiring manager final that candidates say re-tests whatever looked weak
  Behavioral questions are reported to sit INSIDE the technical rounds,
  ~15-20 min each.
  Candidates report the round mix varies by skillset and by role.

AI LAB (artifact driven, as described in candidate write-ups)
  OpenAI:     take-home on OpenAI's own APIs, reported at around 5 hours,
              plus a recorded video walkthrough you defend live, then HM
              and a customer-ability round
  Sierra:     buggy React debugging round + agent presentation + behavioral
  Cognition:  take-home inside Devin, executive pitch to a panel role-playing
              executives, timed simulated customer call; no classic DSA
              round reported
  LangChain:  20-minute non-technical product explainer, then build an agent
  Google:     DSA -> a build-with-AI round -> agentic and ML system design
  Databricks: technical screen -> notebook coding -> Decomposition -> values
```

Reports differ on ordering and detail, and the same company runs different round lists for different requisitions, so ask for your round list in writing rather than treating the sketch above as a syllabus.

Selectivity is real and self-reported aggregator data is only indicative: Taro's public Palantir FDSE page computes its metrics from 125 submitted interview experiences and characterises the loop as failing the large majority of engineers who attempt it. Treat that as directional rather than as a measured pass rate. Timelines are contested, with candidate reports ranging from an offer within about a week of the final round to processes stretching across months with long silences in between. Do not plan around either extreme.

---

## Premium Interview Questions

## Role and Positioning

### What Is a Forward Deployed Engineer and Why Does the Role Exist? - Palantir, OpenAI Interview Question

**Difficulty:** 🟢 Easy | **Tags:** `FDE Fundamentals`, `Role Positioning`, `Industry Context` | **Asked by:** Palantir, OpenAI, Databricks

??? success "View Answer"

    **The one-sentence definition:**

    A Forward Deployed Engineer is an engineer embedded with a customer and held accountable for a **working outcome inside that customer's environment**, rather than for a feature shipped into a shared product.

    **Where the role came from:**

    - The role originated at **Palantir**, where engineers were sent to work inside customer sites rather than shipping features from a central product org, and the pattern spread from there.
    - The reason the model exists is that the engineer who sits with the customer sees what the product is missing. Treat the role as an **R&D function, not a services function**: the observations you bring back are supposed to become platform capability rather than another bespoke application.

    **The framing worth memorizing, because it is the one interviewers reach for:**

    Product engineering and forward deployed engineering split by scope, not by seniority. A product engineer optimises for one capability across many customers. A forward deployed engineer optimises for one customer across many capabilities. Everything else about the role follows from that one sentence: the travel, the ambiguity, the willingness to write code you know is not the general solution, and the obligation to feed what you learn back into the product.

    **Naming precision that scores points:** Palantir's primary title is Forward Deployed Software Engineer (FDSE), internally "Delta," and it sits in **Business Development, not Product Development**. The sibling role, Deployment Strategist, is internally "Echo." Newer Palantir new-grad postings have rebranded to "Forward Deployed Engineer (FDE)" with harder copy, including the line "You will not be handed a ticket queue." The posture that copy is signalling is that you are expected to sit with the customer, the mess and the consequences rather than behind a triage layer.

    **Why every AI lab suddenly needs the role:**

    | Pressure | What the FDE role is a response to |
    |---|---|
    | Enterprise AI pilots stall before P&L impact | Someone accountable past the demo |
    | Probabilistic systems degrade on production data | Ownership after launch, not at launch |
    | The gap between "it works" and "people use it" | An engineer who owns adoption, not just delivery |
    | Customer data reality never matches the sales pitch | Discovery done by the person who has to build it |

    Almost every 2026 FDE announcement cites the MIT NANDA finding that roughly 95% of enterprise generative-AI pilots produce no measurable P&L impact, and reads the diagnosis as a deployment problem rather than a model problem. **Attribute and hedge it if you use it**, because the figure is contested and argued to be widely misrepresented. The safer framing: traditional software delivery ends at launch, but AI systems are probabilistic, so FDEs are measured on whether the system keeps running well and keeps adding value *after* go-live.

    **Scale of the 2026 shift (useful for a "why now" answer):**

    - Indeed data shared with Business Insider: FDE postings went from 643 in April 2025 to 5,330 in April 2026, up roughly 729% year over year.
    - OpenAI Deployment Company (announced 11 May 2026, $4B+ raised), Anthropic's joint venture Ode (4 May 2026, $1.5B), AWS's $1B FDE org (30 June 2026, ~45-day sprints), Microsoft Frontier Company (2 July 2026, $2.5B, 6,000 people). Reported valuations for these vehicles vary widely across coverage; quote the announced capital, not a valuation.

    **What it is not: consulting.** A consulting engagement typically produces an analysis, a recommendation or a one-time solution and then closes. An FDE engagement produces a running system the customer keeps, plus a set of observations that change the vendor's product. The tell is what happens after go-live: a consultancy bills the next statement of work, while an FDE org is judged on whether the deployment is still being used and whether anything from it shipped into the platform. Nor is it sales: FDE postings are overwhelmingly written into engineering job families rather than sales ones. That tells you what the work is; it does not tell you how the job is paid, so confirm the compensation structure with the recruiter rather than assuming it from the title.

    **What the title has fragmented into.** As of a mid-July 2026 snapshot of Palantir's live job feed, 287 postings were open, of which roughly 69 were in the "Forward Deployed" family and 35 were Deployment Strategist. Posting counts move week to week, so treat these as a shape rather than a constant. The FDE title alone spans at least seven specialisations that do not share one skill surface. The split below is an approximate reading of that single snapshot, grouped by hand from posting titles, and the right-hand column paraphrases posting copy rather than quoting an official taxonomy:

    ```
    Approximate grouping of one snapshot, not a published breakdown

    Forward Deployed Software Engineer  ~51   pipelines, ontology, apps
    Forward Deployed Infrastructure      ~7   monitoring, upgrades, on-call
    Forward Deployed Enablement / CS     ~3   training, adoption, workflows
    Forward Deployed AI                  ~2   LLM and agent deployments
    Forward Deployed Reliability         ~2   Python/Java/SQL, Spark tuning
    Forward Deployed Security            ~1   NIST 800-53, ATO ownership
    Forward Deployed Mixed Reality       ~1   specialised client surface
    ```

    Elsewhere the spread is wider still. Anduril's Forward Deployed Engineer, Air Defense posting is a field hardware job: it asks for the "ability to climb towers to install, maintain, and repair sensor equipment and components including cameras, radars, and Pan Tilt Units," "strong mechanical aptitude to troubleshoot and repair critical site infrastructure including generators, solar power systems, and hydraulic systems," eligibility for a US Top Secret/Secret clearance, and the "ability to travel to remote regions of the world for up to two months at a time (up to 80% of the year)." Applied Intuition's Forward Deployed Engineer posting (note the title, which is not Forward Deployed Software Engineer) asks for proficiency in "C++, Python," US citizenship, and a US security clearance held or obtainable. Always ask which variant you are interviewing for.

    **Know the dissent too, because interviewers respect it.** Ex-Palantir FDE Piotr Kraus, speaking to [LeadDev](https://leaddev.com/hiring/the-rise-of-the-forward-deployed-engineer), put it as "the role is definitely real, but the title might have become a little bit frothy." Those are Kraus's words, not the publication's position. The broader criticism is that many orgs now apply the label to anyone who is customer-facing and somewhat technical. The common thread in the dissent is that the feedback loop only exists when the same people own implementation end to end, so an org that keeps the customer-facing engineer away from the build gets the title without the mechanism. Being able to state that test, and then ask an interviewer whether their team passes it, is a stronger answer than either cheerleading or cynicism.

    !!! tip "Interviewer's Insight"
        **What they're testing:** Whether you understand the role you applied for, or whether you read a listicle.

        **Strong answer covers:**
        - The outcome-ownership definition, not a list of tasks
        - The Dev vs Delta framing, and that FDE work feeds product R&D
        - Why the model resurfaced in 2026 (deployment gap, probabilistic systems)
        - That it is a product-feedback function, not billable services

        **Red flags:**
        - "It's like consulting but at a tech company"
        - Describing it purely as customer support or pre-sales
        - Quoting the 95% pilot-failure stat as settled fact with no attribution

---

### How Does an FDE Differ from a Solutions Architect, Sales Engineer or Professional Services? - Anthropic, OpenAI Interview Question

**Difficulty:** 🟢 Easy | **Tags:** `Role Positioning`, `FDE Fundamentals`, `Customer Facing` | **Asked by:** Anthropic, OpenAI, Sierra

??? success "View Answer"

    **Why this gets asked:** Interviewers notice immediately when a candidate does not know which job they are applying for. The distinction is defensible from primary sources, so vague answers read as lack of preparation.

    **The cleanest evidence is usually a single company's own two postings.** Read the pre-sales architect posting and the forward deployed posting from the same employer side by side and the boundary writes itself. The architect posting will talk about becoming a trusted technical advisor, partnering with account executives, producing reference content, and travelling occasionally for workshops. The forward deployed posting will talk about building production applications inside customer systems, owning rollout, and travelling heavily.

    That is the whole distinction in one comparison: **pre-sales advisory with occasional travel versus in-system production building with heavy travel.**

    | Role | Primary deliverable | Engaged when | Writes code in customer systems | Typical travel |
    |---|---|---|---|---|
    | **FDE** | A production system running in the customer's environment | Post-sale, through rollout and adoption | Yes, on customer infrastructure with customer tooling | 25-50%, sometimes higher |
    | Solutions Architect / Applied AI Architect | Reference architecture, technical trust, unblocked deal | Pre-sale | Rarely, and usually only reference material | Occasional |
    | Sales Engineer | Demo, POC, technical objection handling | Pre-sale, quota-attached org | Demo and POC code only | Varies |
    | Implementation / Customer Engineer | Configured, live instance of a shipped product | Post-sale onboarding | Configuration more than construction | Low to moderate |
    | Professional Services | Scoped statement of work, billed against hours | Post-sale, contracted | Yes, but scoped to the SOW | Moderate |
    | Technical Account Manager | Ongoing account health, escalation ownership | Continuous, post-sale | No | Low |

    **More boundaries you can defend:**

    - A Solutions Engineer typically sits on a technical success or pre-sales team running demos, use case scoping and proofs of concept, and fielding the first round of security and compliance questions, while the FDE owns production rollout. The operational test that travels across companies is where the code lives: an FDE writes code that runs on the customer's infrastructure, while a Solutions Architect builds MVPs and proofs of concept in a sandbox the customer does not depend on.
    - Rajkumar Irudayaraj, quoted in [Insight Partners' write-up on the role](https://www.insightpartners.com/ideas/demystifying-forward-deployed-engineers/), draws the demo boundary sharply: "I'm not going to build a demo with an FDE. There are others who can do that for you."
    - Jason Martin, in the same write-up, states the outcome-and-ownership boundary: "Give us your hardest problem, we'll go solve that... you own the IP, we'll hand it over to you." That is the opposite of a billable-hours engagement.
    - Job family is a useful tiebreaker: FDE postings are written into engineering families, while Sales Engineer postings sit in quota-attached sales orgs. Compensation structure varies by company and is not always inferable from the title, so ask the recruiter which ladder the role sits on and how it is paid.

    **The nuance that separates a good answer from a great one:** some companies treat all of these as one talent family, and write senior postings that list Solutions Architect, Forward Deployed Engineer, Customer Engineer and Sales Engineer as interchangeable prior experience. So the honest answer is that the *function* is well defined even where the *titles* are not. Say that out loud, then describe the function you want rather than arguing about labels.

    **The internal division of labour is worth knowing too**, because it is the same question asked from inside a company. Palantir pairs Deployment Strategists (Echo) with FDSEs (Delta), and the pairing is deliberately blurry: the strategist side leans toward problem definition, workflow and datasets, the engineering side toward pipelines and applications, but both sit somewhere on a spectrum between product manager, engineer and strategist, and individual strategists range from writing code daily to writing none. Do not walk into an interview asserting a clean split. Glean's version of the pairing is a pod: its Founding Forward Deployed Engineer posting describes "working in a pod with Forward Deployed PMs directly with the C-suite of the world's most influential companies," and asks the engineer to "operate with the autonomy and accountability of a founder."

    **A tight answer you can deliver in 30 seconds:**

    ```
    "A Sales Engineer proves it can work. A Solutions Architect designs how
     it should work and advises. Professional Services delivers a scoped
     statement of work against billed hours. A TAM keeps the account healthy.
     An FDE is accountable for a working, adopted system inside the
     customer's environment, writes code on their infrastructure, and feeds
     what they learn back into the product roadmap."
    ```

    **Two traps specific to 2026 hiring:**

    - As of mid-2026, Anthropic's careers board had nothing posted under the Forward Deployed Engineer title and the customer-facing engineering roles sat under **Applied AI**. That is an observation of the board at one moment rather than a standing company policy. Say "Anthropic has been posting its FDE-equivalent function under Applied AI" rather than asserting a Forward Deployed title, and re-check the board yourself before an interview, because titling in this space changes quickly.
    - Several vendors run more than one customer-facing engineering ladder side by side, with titles that differ by a word or two and loops that differ substantially. Do not infer the ladder from the title. Ask the recruiter, in writing, which requisition you are being assessed against and which round list goes with it.

    !!! tip "Interviewer's Insight"
        **What they're testing:** Role literacy, and whether you will be surprised by the job in month two.

        **Strong answer covers:**
        - Pre-sales advisory versus post-sale in-system production build
        - Where the code physically runs (customer infra versus sandbox)
        - Outcome accountability versus billed hours versus quota
        - Acknowledgement that titles vary while the function does not

        **Red flags:**
        - Treating FDE, SA and SE as interchangeable synonyms
        - Describing the FDE role as "the technical person on the sales call"
        - Not knowing the company's own internal naming for the role

---

### Contrast the Palantir-Origin FDE With the Modern AI-Lab FDE - Palantir, Databricks Interview Question

**Difficulty:** 🟡 Medium | **Tags:** `Role Positioning`, `Interview Strategy`, `FDE Fundamentals` | **Asked by:** Palantir, Databricks, Google Cloud

??? success "View Answer"

    **The claim to lead with:** there are now two genuinely different jobs wearing the same title, and you prepare differently for each. Blending them in one answer is the most common way candidates sound unprepared.

    | Dimension | Classic (Palantir-style) FDE | AI-lab FDE |
    |---|---|---|
    | Core surface | Data integration, deployment, operations | LLM and agent systems in production |
    | What you build | Terabyte-scale pipelines, ontology and semantic modelling, access controls, workflows and apps | RAG and agent architectures, MCP servers, sub-agents, agent skills, guardrails, evals |
    | Environment | On-prem, air-gapped, customer VPC, on-call | Customer cloud plus vendor APIs, sometimes VPC |
    | Success signal | The deployment works, users adopt, the platform absorbs the learning | Eval scores, production adoption, measurable workflow impact |
    | Experience bar | Palantir FDSE asks for "1+ years of relevant, post-college work experience" and runs internships | Senior: OpenAI's Forward Deployed Engineer, Gov posting asks "5+ years of engineering or technical deployment experience," OpenAI's Forward Deployed Software Engineer asks "7+ years of professional full stack engineering experience," Cursor asks "5+ years" of software development plus "2+ years in a customer-facing role" |
    | Time horizon | Months to quarters embedded | Days to weeks per iteration, in phases |

    **What each side is actually accountable for:**

    - Classic side: wrangling large-scale data, modelling it into something the business recognises, building the applications on top, configuring access controls that satisfy a regulator, and being the person who investigates the production outage at 06:00. The deliverable is a running deployment plus the operational apparatus around it.
    - AI-lab side: owning discovery, technical scoping, system design, build and production rollout for an LLM system, with success measured on production adoption, a measurable workflow change, and eval results that are credible enough to change what the vendor builds next. The deliverables tend to be integration surfaces the customer keeps and extends: tool servers, agent definitions, eval suites and the runbook that goes with them.
    - Cursor's job description is the bluntest published statement of the bar: "This is not a demo role. You are responsible for systems that work in the real world."

    **The interview loops differ structurally, not just in difficulty.** Based on candidate write-ups, Palantir's is round-name driven and code heavy: an online assessment mixing algorithms, SQL and a paginated REST task, then roughly three 60-minute rounds pulled from a named pool (Decomposition, Learning, Re-engineering, Coding, System Design), then a hiring manager final. The AI labs are described as artifact driven: you produce something on your own time and then defend it live. Candidates report that OpenAI issues a multi-hour take-home built on its own APIs plus a recorded video walkthrough, that Cognition asks for a take-home inside Devin and a timed simulated customer call, and that LangChain opens with a non-technical product explainer before any code. Preparing an algorithms drill for an artifact loop, or a portfolio for a round-name loop, is the mismatch that costs candidates the offer. (The full loop-by-loop comparison, and the caveat that goes with it, sits in the "What Interviewers Are Actually Testing" section at the top of this page.)

    **What each side actually optimises for:** the Decomposition round is the one with no FAANG equivalent and the one candidates most often fail. It runs at a higher altitude than a system design round: the failure mode is staying abstract and never landing on a concrete build, and the round ends on business outcome, KPIs and what you would present to executives rather than on throughput and caching. The AI-lab equivalent differentiator is a single question: **"How do you know your AI system is actually working?"**

    **Travel is the constant across both:** Palantir FDSE "25-50% preferred," Deployment Strategist "25-75% required," and OpenAI's Forward Deployed Software Engineer posting states that travel up to 50% is required. Read the band on the specific posting rather than assuming an industry norm.

    **Do not treat Palantir as one loop.** Candidates who have been through the Deployment Strategist (Echo) process describe a different shape, with a case-style data-table analytical round and a product-sense round. Palantir's Forward Deployed AI, Infrastructure, Reliability, Enablement, Security and Mixed Reality variants do not share one skill surface either.

    !!! tip "Interviewer's Insight"
        **What they're testing:** Whether you researched this specific company's version of the role.

        **Strong answer covers:**
        - Two skill surfaces, named and kept separate
        - Concrete deliverables per side (pipelines and ontology versus evals and agents)
        - Awareness that the loops differ structurally, not just in difficulty
        - Which one you are stronger at, said honestly

        **Red flags:**
        - "FDE is FDE, it's all customer-facing engineering"
        - Preparing only LeetCode for a loop where Decomposition and Learning are the filters
        - Claiming deep AI-agent experience with no eval story behind it

---

### Walk Me Through What an FDE Deployment Lifecycle Actually Looks Like - OpenAI, Cognition Interview Question

**Difficulty:** 🟡 Medium | **Tags:** `Delivery Lifecycle`, `Scoping`, `Customer Facing` | **Asked by:** OpenAI, Cognition, LangChain

??? success "View Answer"

    **What they are probing:** whether you have actually delivered into someone else's environment, or whether your mental model stops at "build the thing." The tell is whether you talk about the end of the engagement.

    **A defensible lifecycle, with real published shapes behind each phase:**

    ```
    PHASE 0  QUALIFY            entry criteria, exit criteria, kill triggers
             |                  agreed BEFORE any build starts
    PHASE 1  DISCOVERY          a short, intense onsite block, not a month
             |                  find the undocumented workflow, the data source
             |                  people actually trust, and the person who knows
             |                  why the process works that way
    PHASE 2  VALIDATION         build evals and testing criteria WITH customer
             |                  domain experts, before significant development
    PHASE 3  DELIVERY           recurring onsite days through the build
             |                  ship into their infra, their tooling, their review
    PHASE 4  ADOPTION           the phase everyone underestimates
             |                  training, champions, workflow change, trust
    PHASE 5  TRANSFER           runbooks, docs, trained internal owners
                                the engagement is designed to end
    ```

    **Numbers you can cite for pacing:**

    - **Budget the phases unevenly.** Published enterprise AI deployments consistently show the pipeline landing quickly and the pilot-and-trust phase running several times longer. If your plan gives PHASE 3 six weeks and PHASE 4 one, you have drawn the wrong calendar, and the sponsor should see the real shape at kickoff rather than at the first slipped date.
    - **AWS FDE org:** small teams embedded with one customer on ~45-day sprints, with the deliverables designed to be things the customer keeps: a running system in their own account, the semantic layer beneath it, runbooks, architectural documentation and trained internal champions.
    - **FIS and Anthropic (announced May 2026):** the two companies announced a collaboration bringing agentic AI to banking, starting with financial crimes, with the stated aim of compressing anti-money-laundering investigations from hours to minutes by automatically assembling evidence across a bank's core systems. Embed, build, transfer is the shape to describe.
    - **Crisis engagements collapse the whole calendar.** Emergency response work can require something operational within days, which is the counterexample to any answer that assumes a fixed cadence. If you claim a standard phase length, be ready to say what you would cut to hit a one-week deadline instead.

    **What a week looks like in practice:** a few days onsite or on calls with the users and the data owners, a few days building, and a running thread of unblocking. Nabeel Qureshi's essay [Reflections on Palantir](https://nabeelqu.substack.com/p/reflections-on-palantir) puts a number on it: he describes FDEs as typically expected to go onsite to the customer's offices 3 to 4 days per week, which meant a great deal of travel. The split to expect is blunt: an FDE spends far more of the week in front of customers than a product engineer does, and the building happens around that.

    **Governance is part of the answer, not an afterthought.** Insight Partners' practitioner write-up on the role frames it in two halves. On the front door, Rajkumar Irudayaraj: "Interest is not the same as readiness. Define entry criteria, exit criteria, and kill triggers before the build starts." On the back door, the article's own section heading is "Design every engagement to end. Solve the hard problem, then leave." Irudayaraj again on what the team is for: "The FDE should be a permanent learning loop for the company, not a permanent crutch."

    **Discovery is where the value is created, and where candidates are thin.** Expect the customer's description of their systems to differ from what you actually find, and budget discovery time for the gap rather than treating it as a surprise. Two habits worth internalising:

    - [Ramp's engineering blog](https://engineering.ramp.com/post/forward-deployed-engineering) puts the discipline in two words: "always be scoping". Treat scope as something you re-cut every week against what discovery has actually turned up, not something you agree once at kickoff and then defend.
    - Prototype rather than interview when a stakeholder cannot articulate what they want. People are far better at reacting to something concrete than at specifying it in the abstract, so put a rough version in front of them within days and ask whether this is what they meant. A wrong prototype extracts more requirements in ten minutes than a workshop does in a week.

    **Anchor discovery to the executive, then work downward.** The problem the sponsor can state in one sentence is the one that survives the next budget cycle; the problems the operators describe are the ones that make the system usable. You need both, in that order, and you only get the first by asking for the meeting. PHASE 1 is not finished until you can write the sponsor's problem, the measured baseline, the system of record and the security path to production on a single page and have the customer agree with all four.

    !!! tip "Interviewer's Insight"
        **What they're testing:** Whether you scope before you build, and whether you plan the handoff.

        **Strong answer covers:**
        - Explicit entry, exit and kill criteria agreed up front
        - Evals or acceptance criteria defined with domain experts before development
        - The adoption phase named separately from the build phase
        - A concrete transfer artifact list (runbooks, docs, trained owners)

        **Red flags:**
        - Jumping to architecture without asking what problem is being solved
        - No plan for what happens when you leave
        - Treating adoption as the customer's problem rather than yours

---

### How Is Success Measured for an FDE? - Sierra, OpenAI Interview Question

**Difficulty:** 🟡 Medium | **Tags:** `Success Metrics`, `Evals`, `Adoption` | **Asked by:** Sierra, OpenAI, Anthropic

??? success "View Answer"

    **The framing that wins this question:** an FDE is not measured on shipping. Shipping is the midpoint. You are measured on three things in order: **the system works, the users adopt it, and the company learns something reusable.**

    **Layer 1: does it work, provably.** In AI-lab loops this collapses to one question, and it is the single most differentiating one in the whole loop: *"How do you know your AI system is actually working?"* Weak answers gesture at monitoring it. Strong answers name artifacts:

    ```python
    # Acceptance gates an FDE can defend in a customer review
    GATES = {
        "quality":   {"metric": "exact_match_on_golden_set", "threshold": 0.92,
                      "dataset": "500 labelled items, curated with customer SMEs"},
        "regression":{"metric": "no_drop_vs_last_release",   "threshold": 0.00},
        "safety":    {"metric": "jailbreak_suite_pass_rate", "threshold": 0.99},
        "latency":   {"metric": "p95_seconds",               "threshold": 4.0},
        "cost":      {"metric": "usd_per_resolved_task",     "threshold": 0.11},
    }

    # Per-intent eval sets matter more than one aggregate number:
    # a 92% average can hide a 40% pass rate on the intent that
    # drives most of the customer's volume.
    INTENT_SETS = ["refund", "address_change", "policy_lookup", "escalation"]
    ```

    Make it concrete by building the evaluation set with the customer's domain experts before significant development starts, not after. In a deep technical-debugging workflow that means writing down the ordered sequence of actions a skilled human performs to work a case from symptom to resolution, which can easily run to dozens of steps. That sequence is the label. Development is not complete until the evals verify efficacy. Starting with evaluations rather than bolting them on at the end is the recommendation this page would give for any customer-facing AI build, because the eval set is also the artifact that lets you and the customer agree what "good" means before either of you has an incentive to argue about it.

    **Layer 2: adoption, which is where deployments actually die.** A system with perfect offline scores and 5% adoption is a failed deployment, and it will be scored as one. Instrument adoption the way you instrument quality: weekly active users **inside the workflow** rather than logins, task completion rate, the share of tasks where a user overrode the system, and the cohort curve showing whether week-4 users are still there in week 12. Set the adoption target with the sponsor before launch, in the same document as the quality gates, so nobody can relitigate what "rolled out" meant.

    **Layer 3: measured business impact, with a baseline.** Senior forward deployed postings write this in as a responsibility: own the value case, set the impact hypothesis, measure a baseline before anything ships, and run the pre-deployment and post-deployment comparison yourself. The honest version of this answer names the target you agreed with the customer, the baseline you measured before anything shipped, and the delta you actually landed, including when the delta came in under the target. A candidate who reports a real shortfall against a stated goal is a stronger signal than one who reports a round number with no baseline behind it.

    **Layer 4: what the company learned.** The durable version of this loop is that product engineers watch what forward deployed engineers keep doing by hand and build the thing that deletes the manual step. Your side of it is to identify and codify the repeatable pattern rather than leaving it in one account's repository. How much of the bespoke work gets absorbed back into the product is a fair question to ask your interviewer, because it tells you whether the team is a learning loop or a delivery shop.

    | Metric family | Example measure | Who cares |
    |---|---|---|
    | Quality | Golden-set pass rate, per-intent eval scores | You, and the customer's technical lead |
    | Reliability | p95 latency, error budget burn, drift alerts | Ops and on-call |
    | Adoption | Weekly active users in the workflow, task completion | The executive sponsor |
    | Business | Baseline versus post-deployment KPI delta | Finance and the renewal |
    | Product | Reusable components, codified patterns shipped | Your own engineering org |

    **Measure the constraints that must never break, separately from quality.** Any rule the business genuinely cannot violate should be checked by deterministic code and reported as a violation count that has to be zero, not folded into an aggregate accuracy score where a 99% pass rate looks acceptable. Quality metrics are averages; constraint metrics are absolutes, and mixing them is how a deployment passes its review and still breaks a regulatory rule in week two.

    !!! tip "Interviewer's Insight"
        **What they're testing:** Whether "done" means shipped or means working, adopted and measured.

        **Strong answer covers:**
        - A golden dataset, a regression suite, per-intent eval sets, drift detection
        - Named sign-off owner and acceptance thresholds agreed with the customer
        - Adoption as a first-class metric with a baseline
        - Deterministic guardrails for constraints that must never be violated

        **Red flags:**
        - "We'd monitor it and iterate"
        - One aggregate accuracy number with no per-segment breakdown
        - No mention of who signs off or what the pre-deployment baseline was

---

### How Do You Decide What Stays Bespoke and What Becomes Product? - Palantir, Baseten Interview Question

**Difficulty:** 🔴 Hard | **Tags:** `Productization`, `Trade-offs`, `Platform Strategy` | **Asked by:** Palantir, Baseten, OpenAI

??? success "View Answer"

    **This is the discriminating topic for senior FDE candidates.** Every FDE org lives on the same tension: you are paid to solve one customer's problem completely, and your company only survives if that work compounds into something sellable to the next customer.

    **The two statements of the tension worth holding at once:**

    - The classic answer is a division of labour rather than a rule: the forward deployed side goes deep on one customer without worrying about overfitting, and the product side is responsible for taking what was built and generalising it into something sellable elsewhere. The honest cost of that contract is that forward deployed code is written fast, which usually means real technical debt and workarounds you would not defend in a design review.
    - The mirror-image failure is generalising too early. Building the abstraction before you have seen the same need twice produces a framework that fits nobody, and the engagements that produce the most reusable insight are usually the ones that went deepest on one customer's specific problem without setting out to generalise at all.

    Those are not contradictory. Both say the same thing: **solve deeply first, generalise on evidence, never on speculation.**

    **A decision framework you can run live in an interview:**

    ```
    For each component you built, score 0-2 on each axis:

      REPETITION    Have you seen this need at 2+ customers?
                    0 = one customer   1 = two   2 = three or more
      STABILITY     Will the requirement survive the customer's next reorg?
      COST          Is maintaining this bespoke copy expensive per customer?
      RISK          Does a bug here have compliance or safety blast radius?
      COUPLING      Can it be extracted without customer-specific data leaking?

    Score 8-10  -> productize now, hand to core engineering with a spec
    Score 5-7   -> harden into a reusable internal library or template,
                   keep it in the FDE org, revisit after the next engagement
    Score 0-4   -> leave it bespoke, document the hack, set a review date
    ```

    **The second engagement is where you learn, not the first.** Reuse compounds: the first build for a new problem is almost entirely bespoke, and it is only after the second and third that you can see which pieces were the customer and which were the problem shape. Team structure follows the same curve. A forward deployed org that only ever optimises for the account in front of it never gets cheaper; the larger prize is building the internal systems that make each subsequent deployment cost less than the last, and that work has to be funded deliberately because no single customer will ever ask for it.

    **The mechanism matters as much as the decision.** The products that come out of forward deployed work are almost never designed as products. They start as the tool somebody built to delete a manual step they had watched three engineers perform at three different customer sites: an ingestion helper, a visualisation surface, an internal app builder. Note the direction of travel, because it is the part candidates invert. Nobody set out to build a platform, they set out to stop doing the same cruft work by hand. The handoff artifact that makes this work is a written spec describing the observed need, how often it appeared, and what the bespoke version currently does, not a repository thrown over the wall for someone else to interpret.

    | Approach to customer variation | When it is right | Failure mode |
    |---|---|---|
    | Config and input presets | Variation is data or thresholds | Config sprawl nobody can audit |
    | Feature flags | Variation is behavioural and temporary | Ungoverned toggle inventory |
    | Extension points and plugins | Variation is a genuine domain difference | Plugin API becomes a public contract |
    | Fork per customer | Almost never | N codebases, N security patches |

    **Name the economics, because senior interviewers do.** Senior forward deployed postings encode this as a responsibility: align early on what should generalise, what stays customer-specific, and what "ready for handoff" means in concrete terms, then turn ambiguous feedback, failures and escalations into durable product requirements rather than a growing pile of one-off fixes. The counterweight worth naming is contract size. Embedding engineers inside a customer is expensive per account, so the model only pays for itself above some deal value; below it the economics collapse back into consulting with a product logo on the invoice. You will not find one published threshold that everybody agrees on, which is exactly why "what contract sizes does this team support?" is a strong question to ask your interviewer. Nabeel Qureshi's evidence for the other side, in [Reflections on Palantir](https://nabeelqu.substack.com/p/reflections-on-palantir), is margin structure: he contrasts Palantir's 80% gross margins in 2023, which are software margins, with Accenture's 32%.

    **The cleanest one-liner for the division of labour is a road metaphor:** the forward deployed side lays the rough track that proves people want to travel that way, and core engineering decides which tracks are worth paving. Say the counterweight out loud as well: aggressive scoping genuinely does conflict with the engineering instinct to generalise, because good interfaces and platforms are what make software scale, so treat this as a judgement call you re-make per component rather than a rule you apply once.

    !!! tip "Interviewer's Insight"
        **What they're testing:** Whether you can hold two objectives at once without collapsing into either pure hacking or pure platform-building.

        **Strong answer covers:**
        - Solve deeply first, generalise on repetition evidence
        - A concrete signal threshold (seen at N customers, cost per copy)
        - The handoff artifact: a spec, not a thrown-over-the-wall repo
        - Honest acknowledgement of the technical debt you are choosing

        **Red flags:**
        - Building a plugin framework on engagement one
        - Refusing to write anything customer-specific
        - No mechanism for feeding learning back into product

---

### Why FDE Rather Than a Standard SWE Role, and Who Actually Thrives in It? - Cognition, ElevenLabs Interview Question

**Difficulty:** 🔴 Hard | **Tags:** `Behavioral`, `Career Paths`, `Role Fit` | **Asked by:** Cognition, ElevenLabs, Palantir

??? success "View Answer"

    **Treat this as a screening question with real teeth.** Recruiter screens across these companies ask some version of why you want a forward deployed role specifically rather than a standard engineering one, and generic answers about liking hard problems get filtered immediately. Prepare an answer that would not survive being pasted into an application for a normal SWE job.

    **The traits worth demonstrating, drawn from how these teams describe their strongest hires:**

    - **Independent thinking.** The job does not come with a playbook, and pattern matching against your last company is usually wrong because the constraint set is different in every customer environment.
    - **Grit.** Deployments are frequently unpleasant: hostile networks, stale credentials, a sceptical operator, and a date that does not move. The people who last believe the situation is solvable before they can see how.
    - **Compulsive building.** The strongest forward deployed engineers reach for a prototype where others reach for a document.
    - **Business curiosity.** You have to want to know how the customer makes money, or you will build the technically elegant thing nobody needed.
    - The blunt version of the hiring bar is whether the interviewer would want you next to them in a difficult customer room at the end of a bad week.

    Value orientation is the trait these teams say they cannot teach. Coding is teachable; caring about whether the customer's number actually moved is not. Behavioral rounds in this family are consequently biased toward failure rather than triumph, so bring the mistake, the struggle and what you changed, not a polished success story. The other counterweight for candidates preparing this as a pure engineering role is social: the job demands unusual sensitivity to context and the ability to earn the trust of senior corporate or government counterparts who did not ask for you to be there.

    **Palantir's new-grad FDE posting names five values, the clearest modern trait bar:** going where you are needed most; **agency**, meaning you learn continuously, make decisions with incomplete information and do not wait to be told what to do next; embracing the ambiguity; intrinsic motivation; and **ruthless goal orientation**, meaning you do not treat a product's existing boundaries as the limit of what you can do about the customer's problem.

    **Who does not thrive, stated plainly:**

    | Anti-signal | Why it fails in this role |
    |---|---|
    | Needs a well-specified ticket | The spec does not exist; you write it from a conversation |
    | Optimises for elegance over shipped outcome | Cursor's JD: "This is not a demo role" |
    | Long tenure inside one large, highly-scaffolded org | There is no scaffolding here: no spec, no platform team, no established playbook to fall back on |
    | Dislikes travel or unpredictable weeks | Travel is built into the job: Palantir FDSE "25-50% preferred," Deployment Strategist "25-75% required," Anduril up to 80% with two-month deployments |
    | Says "we" when describing owned work | A general FDE anti-signal, because the evaluator is trying to establish what you personally decided and built |

    **Name the burnout risk honestly.** Work-life balance is the concern practitioners raise most often about this role, and sustained travel at the top of the published bands is the reason. Saying this out loud, along with how you would manage it, reads as maturity rather than hesitancy.

    **A structure for the answer itself:**

    ```
    1. The pull, not the push
       "I want to own the outcome, not the ticket" + one concrete story
       where you owned a result end to end, including the unglamorous part.
    2. Evidence you already do the job informally
       You ran the discovery call. You wrote the eval set. You trained the
       users. You held a line with a stakeholder. Use "I", not "we".
    3. Why THIS company
       Name a specific deployment, product or engineering post. "Interesting
       hard problems" is the answer that gets filtered.
    4. The honest trade-off you accept
       Travel, ambiguity, technical debt you will knowingly write.
    ```

    **Career paths in and out, worth knowing before you commit:**

    - **In:** Palantir hires very early career (FDSE asks for "1+ years of relevant, post-college work experience" and runs internships). AI labs hire senior: OpenAI's Forward Deployed Engineer, Gov posting asks "5+ years of engineering or technical deployment experience," its Forward Deployed Software Engineer asks "7+ years of professional full stack engineering experience," and Cursor asks "5+ years of experience with software development" plus "2+ years in a customer-facing role, leading discovery conversations." Senior forward-deployed postings generally want several years of production engineering behind you, and increasingly want evidence you have sat in front of a customer.
    - **Out:** Nabeel Qureshi's first-hand account of the original model, [Reflections on Palantir](https://nabeelqu.substack.com/p/reflections-on-palantir), claims a typical YC batch contains more ex-Palantir founders than ex-Googlers, despite Google employing roughly 50 times more people. Treat that as one practitioner's observation rather than a measured statistic, but the underlying logic is sound: the role gives you repeated exposure to unsolved business problems, real customers and end-to-end ownership, which is close to founder training. The other common exits are into core product engineering at the same company, into engineering leadership, and into the same function at a competitor at a higher level. Ask your interviewer where the last three people who left this team went.
    - **Compensation context (2026):** posted bands vary a lot by company, level and location, and aggregator medians move quickly. Read the band on the posting you are actually interviewing for, and check levels.fyi yourself for current totals rather than repeating a figure from a blog post.

    **Know the dissent on longevity.** One industry view is that the title is a phase and will fade the way "prompt engineer" did, once AI tooling settles and organizational data silos break down. The competing view is convergence rather than disappearance: as product engineers are pushed closer to customers and forward deployed engineers are pushed to build reusable platform pieces, the two roles meet in the middle and the distinction becomes one of emphasis. A candidate who can discuss this credibly signals they chose the role rather than chased a title.

    !!! tip "Interviewer's Insight"
        **What they're testing:** Genuine role fit, individual ownership, and whether you will still be here in 18 months.

        **Strong answer covers:**
        - A specific story where you owned an outcome, not a component
        - "I" framing with concrete decisions and trade-offs you made
        - A company-specific reason grounded in a real deployment or product
        - Clear-eyed acceptance of travel, ambiguity and pragmatic debt

        **Red flags:**
        - "I want to work on interesting problems" with nothing specific
        - Describing the customer as an obstacle to the engineering
        - Hedging on travel or on-call rather than answering directly
        - Using "we" throughout a story you are claiming credit for

---

## Data and Integration

### How Would You Plan the First Integration Sprint Against a Customer's Source Systems? - Palantir, Databricks Interview Question

**Difficulty:** 🟢 Easy | **Tags:** `Data Integration`, `Scoping`, `Deployment` | **Asked by:** Palantir, Databricks, Google Cloud

??? success "View Answer"

    **What the interviewer is probing:** whether you treat "integrate their data" as a networking and permissions problem, not a code problem. Most FDE schedule slips in week one are firewall rules, credentials and a DBA who has not been told you exist.

    **Step 1: inventory before you connect anything.** For every candidate source, capture six facts:

    | Fact | Why it decides your design |
    |------|---------------------------|
    | Transport | JDBC, object storage, SFTP, REST/GraphQL, Kafka, SaaS API |
    | Volume and growth | 2 GB/day vs 2 TB/day changes everything downstream |
    | Change semantics | Append-only, edited in place, or full snapshot only |
    | Ownership | Who grants credentials, who is paged when it breaks |
    | Sensitivity | PII, PHI, export-controlled, contractual restrictions |
    | Trust | Is this the source users actually believe, or a copy of it |

    That last row is the one candidates skip. The hard part of this job is rarely the modelling. It is finding the workflow nobody documented, the data source people actually trust, and the person who knows why the process works the way it does.

    **Step 2: know the connectivity model.** On a mature platform the connection is agent-based, not direct: an agent process runs *inside* the customer network and reaches out, so you reason about egress rules, proxies and private link rather than asking security to open an inbound port. The usual shape is a worker component that does the reading and a proxy component that brokers the connection, with direct database connections treated as the legacy path. Private link on the major clouds keeps the data path off the public internet, which is normally a hard security-review requirement rather than a preference. Say this out loud in the interview; it signals you have deployed somewhere real.

    **Step 3: map transport to first-day risk.**

    | Source type | Typical connector | The thing that actually bites you |
    |-------------|-------------------|-----------------------------------|
    | Oracle / SQL Server / Postgres | JDBC | Read replica lag, no CDC prerequisites enabled |
    | S3 / GCS / Azure Blob / OneLake | Cloud storage | Bucket policy and KMS key, not the SDK |
    | SFTP / FTPS / SMB | File transfer | Filename-encoded dates, partial file writes |
    | REST / GraphQL / OData | API | Pagination, rate limits, no bulk export |
    | Kafka / Kinesis / Pub/Sub | Streaming | No schema registry, retention shorter than backfill |
    | Salesforce / SAP / Workday / NetSuite | Enterprise SaaS | API quotas and a licence you must buy |

    **Step 4: write the inventory down as config, not as a doc.** Make it the artefact the pipeline reads, so it cannot drift from reality:

    ```yaml
    sources:
      - name: orders_oracle
        transport: jdbc
        driver: oracle
        network: agent            # agent runs inside customer VPC, egress only
        auth: vault://cust/oracle_ro
        change_semantics: edited_in_place
        load_strategy: watermark  # cdc pending DBA change window (ticket INF-4412)
        watermark_column: last_modified_ts
        owner: dba-team@customer
        sensitivity: [pii]
        expected_daily_rows: [180000, 420000]

      - name: telemetry_sftp
        transport: sftp
        path: /outbound/telemetry/%Y/%m/%d/*.csv.gz
        network: agent
        change_semantics: append_only
        load_strategy: file_watch
        owner: plant-it@customer
        sensitivity: []
        expected_daily_rows: [9000000, 14000000]
    ```

    **Step 5: prove the path before you promise a date.** A three-minute smoke test per source has saved more schedules than any architecture diagram:

    ```python
    def smoke(source) -> dict:
        """Resolve, connect, authenticate, read one row, measure. In that order."""
        checks = {}
        checks["dns"]      = resolve(source.host)          # split-horizon DNS is common
        checks["tcp"]      = tcp_connect(source.host, source.port, timeout=5)
        checks["auth"]     = authenticate(source)          # expired service account?
        checks["read_one"] = read_first_row(source)        # SELECT-level grants?
        checks["throughput_mb_s"] = timed_read(source, mb=50)
        return checks
    ```

    Run it on day one and report the failures to the customer as a single list with owners. Discovering on day nine that the service account has read access to a schema but not to the three views you actually need is the classic avoidable slip.

    **Step 6: sequence the sprint.** A defensible two-week plan:

    - **Days 1 to 2:** one source, end to end, all the way to something a user can see. Prove the network path, credential flow and scheduling before you touch source two.
    - **Days 3 to 6:** the remaining P0 sources, landing raw and immutable first.
    - **Days 7 to 8:** joins and the semantic model, where you discover the systems disagree.
    - **Days 9 to 10:** health checks, an owner for each alert, and a written runbook.

    **Strong answer covers:**

    - Asks what the *first workflow* is before listing connectors, so scope is anchored to one outcome
    - Names credentials, egress and a customer DBA as the critical path
    - Lands raw data immutably so re-parsing never requires re-pulling from the source
    - Puts a named human, not a Slack channel, on every failure mode

    **Red flags:** proposing a full enterprise data lake in week one; assuming you get production credentials on day one (you usually get read access to a staging copy that is three weeks stale); saying "we will just use CDC" without asking whether the DBA will enable supplemental logging.

---

### Write the Month-over-Month Change in Average Product Rating - Palantir, Databricks Interview Question

**Difficulty:** 🟢 Easy | **Tags:** `SQL`, `Window Functions`, `Data Quality` | **Asked by:** Palantir, Databricks

??? success "View Answer"

    **The prompt:** given a reviews table with product IDs, ratings and timestamps, return the month-over-month change in average rating per product. Expect a SQL problem of roughly this shape to sit alongside a coding problem in the same timed sitting rather than in a round of its own, so budget accordingly.

    **Schema:**

    ```sql
    -- reviews(review_id BIGINT, product_id BIGINT, rating INT, created_at TIMESTAMP)
    ```

    **The clean answer:**

    ```sql
    WITH monthly AS (
        SELECT
            product_id,
            DATE_TRUNC('month', created_at) AS month,
            AVG(rating::NUMERIC)            AS avg_rating,
            COUNT(*)                        AS n_reviews
        FROM reviews
        WHERE rating IS NOT NULL
        GROUP BY 1, 2
    )
    SELECT
        product_id,
        month,
        ROUND(avg_rating, 3)                                          AS avg_rating,
        n_reviews,
        ROUND(LAG(avg_rating) OVER w, 3)                              AS prev_avg_rating,
        ROUND(avg_rating - LAG(avg_rating) OVER w, 3)                 AS mom_change
    FROM monthly
    WINDOW w AS (PARTITION BY product_id ORDER BY month)
    ORDER BY product_id, month;
    ```

    **Now earn the round.** Anyone can write the `LAG`. What separates candidates is naming the assumptions the query silently makes:

    - **Gaps.** `LAG` returns the previous *row*, not the previous *month*. A product with reviews in January and April reports April against January and calls it month over month. If the customer wants true adjacency, generate a calendar and left join, or guard with `LAG(month) OVER w = month - INTERVAL '1 month'`.
    - **Thin months.** A month with two reviews swings wildly. Ask for a minimum volume threshold, or return `n_reviews` so the consumer can filter. Never hide it.
    - **Time zone.** `DATE_TRUNC` on a UTC timestamp buckets a Tokyo customer's evening reviews into the next day. Ask which time zone the business reports in before you write the query, not after.
    - **Ratings that are not ratings.** Sentinel values like 0 or -1 for "no rating given" are common in migrated systems and will drag every average down.
    - **Late arrivals and edits.** If reviews can be updated, an average for a closed month is not stable. Either report on an as-of snapshot or accept restatement and say so.

    **The gap-safe variant:**

    ```sql
    WITH monthly AS (          -- same CTE as above; repeated so this runs standalone
        SELECT
            product_id,
            DATE_TRUNC('month', created_at) AS month,
            AVG(rating::NUMERIC)            AS avg_rating,
            COUNT(*)                        AS n_reviews
        FROM reviews
        WHERE rating IS NOT NULL
        GROUP BY 1, 2
    )
    SELECT
        m.product_id,
        m.month,
        m.avg_rating,
        CASE WHEN p.month = m.month - INTERVAL '1 month'
             THEN m.avg_rating - p.avg_rating
        END AS strict_mom_change
    FROM monthly m
    LEFT JOIN monthly p
      ON p.product_id = m.product_id
     AND p.month      = m.month - INTERVAL '1 month';
    ```

    !!! tip "Interviewer's Insight"
        **What they're testing:** window function fluency plus the instinct to interrogate data before trusting an aggregate. Candidates describe the Palantir OA as a coding problem, a SQL problem and a paginated REST task inside roughly 90 minutes, so if that is the format you are given, budget about 20 minutes here and do not gold-plate.

        **Strong answer signals:** writes it once, correctly, then volunteers the gap and thin-month caveats unprompted; returns row counts alongside averages; asks about time zone. **Red flag:** a correlated subquery per row, or silently dropping products with a single month of data.

---

### Ingest a Paginated REST API That Rate-Limits and Fails Halfway - Palantir, ElevenLabs Interview Question

**Difficulty:** 🟡 Medium | **Tags:** `API Integration`, `Python`, `Reliability` | **Asked by:** Palantir, ElevenLabs, OpenAI

??? success "View Answer"

    **Why this is asked:** publicly shared accounts of Palantir's online assessment describe a REST task with pagination bundled alongside coding and SQL, and every AI-lab FDE ends up pulling a customer SaaS API that was never designed for bulk export. The interviewer wants to see whether you write a loop or a *loader*.

    **Establish the contract first.** Ask, or read the docs for, five things:

    1. **Pagination style:** offset/page, cursor, or link header. Offset pagination over a table being written to will skip and duplicate rows; cursor pagination will not.
    2. **Rate limit:** requests per second, per minute, per day, and whether there is a `Retry-After` header.
    3. **Ordering and filtering:** can you sort by `updated_at` and filter `since`? If yes you get incremental loads for free. If no, you are doing full pulls.
    4. **Total size:** 50k records is a script, 50M records is a paged backfill plus a daily delta.
    5. **Idempotency:** is there a stable record ID you can dedupe on?

    **A loader that survives contact with production:**

    ```python
    import random, time, logging
    from datetime import datetime, timezone
    from email.utils import parsedate_to_datetime
    from typing import Iterator, List, Optional, Tuple
    import requests

    log = logging.getLogger("api_loader")
    RETRYABLE = {429, 500, 502, 503, 504}

    def parse_retry_after(raw: Optional[str]) -> Optional[float]:
        """RFC 9110 permits delta-seconds OR an HTTP-date. Handle both.

        Returns seconds to wait, or None if the header is absent or unparseable
        so the caller can fall back to its own backoff. Never raises: a bad
        header must not turn a retryable 429 or 503 into a hard failure.
        """
        if not raw:
            return None
        try:
            return max(0.0, float(raw))          # delta-seconds form
        except ValueError:
            pass
        try:
            when = parsedate_to_datetime(raw)    # HTTP-date form
        except (TypeError, ValueError):
            return None
        if when is None:
            return None
        if when.tzinfo is None:                  # naive means UTC per RFC 9110
            when = when.replace(tzinfo=timezone.utc)
        return max(0.0, (when - datetime.now(timezone.utc)).total_seconds())

    def request_with_backoff(session, url, params, max_attempts=6, base=0.5, cap=30.0):
        """Full jitter backoff. Honours Retry-After when the server sends it."""
        for attempt in range(max_attempts):
            try:
                resp = session.get(url, params=params, timeout=(5, 60))
            except requests.RequestException as exc:
                if attempt == max_attempts - 1:
                    raise
                log.warning("transport error %s attempt=%d", exc, attempt)
                time.sleep(random.uniform(0, min(cap, base * 2 ** attempt)))
                continue

            if resp.status_code not in RETRYABLE:
                resp.raise_for_status()
                return resp

            backoff = random.uniform(0, min(cap, base * 2 ** attempt))
            server_wait = parse_retry_after(resp.headers.get("Retry-After"))
            wait = backoff if server_wait is None else server_wait
            log.warning("status=%s sleeping=%.1fs attempt=%d",
                        resp.status_code, wait, attempt)
            time.sleep(wait)
        raise RuntimeError(f"exhausted {max_attempts} attempts for {url}")

    def paginate(session, url, since: str, per_page: int = 200,
                 cursor: Optional[str] = None
                 ) -> Iterator[Tuple[List[dict], Optional[str]]]:
        """Cursor pagination. Yields (records, next_cursor) one page at a time.

        next_cursor is the checkpoint: persist it in the same transaction that
        writes the page, then pass it back as `cursor` to resume a failed run.
        It is None on the final page, which is how the caller knows the pull
        finished rather than died.
        """
        pages = 0
        while True:
            params = {"per_page": per_page, "updated_since": since}
            if cursor:
                params["cursor"] = cursor
            body = request_with_backoff(session, url, params).json()

            cursor = body.get("next_cursor")
            yield body["data"], cursor

            pages += 1
            if not cursor:
                log.info("complete pages=%d", pages)
                return
            if pages > 50_000:                      # runaway guard
                raise RuntimeError("pagination did not terminate")

    # Caller: the checkpoint and the rows commit together, so a re-run resumes.
    for records, checkpoint in paginate(session, url, since, cursor=resume_from):
        write_staging(records)
        store.set("orders_cursor", checkpoint)
    ```

    **The four failure modes to name out loud:**

    - **Partial failure.** Page 400 of 900 dies. Do not restart from zero. `paginate` hands you the cursor with every page: persist it and the high-water `updated_at` in the same transaction that writes the page, then pass the stored cursor back in as `cursor` so a re-run resumes where it stopped. Write to a staging location and promote atomically, so a half-finished run never becomes visible downstream.
    - **Duplicates.** Retries and cursor resumes both produce them. Dedupe on the source primary key keeping the newest `updated_at`; never rely on the API to be exactly-once.
    - **Silent truncation.** If `total_pages` or `total_count` is present, assert your row count against it and fail the build on mismatch. Silent short reads are the worst class of bug because everything downstream looks healthy.
    - **Rate limit budget.** If the customer's quota is shared with their production application, your backfill can take down their business. Cap concurrency, run backfills off-hours, and get that agreed in writing.

    **Red flags:** a bare `while True` with no terminating guard; retrying 400s and 401s; a fixed `sleep(1)` instead of backoff; no jitter, which synchronises every worker into the same retry wave; parsing `Retry-After` as a bare number, when RFC 9110 also permits an HTTP-date and the `float()` blows up a retryable 429 into a hard failure; and treating a 200 with an error body as success.

---

### Make a Customer Pipeline Safe to Re-Run: Incremental Loads, CDC and Idempotency - Palantir, Databricks Interview Question

**Difficulty:** 🟡 Medium | **Tags:** `CDC`, `Idempotency`, `Pipelines` | **Asked by:** Palantir, Databricks

??? success "View Answer"

    **The scenario:** an order table of 400M rows in the customer's Oracle instance. Full snapshots take four hours and the business wants 15-minute freshness. What do you build?

    **Three options, in increasing order of what you have to ask the customer for:**

    | Approach | Needs from customer | Catches deletes? | Catches in-place edits? |
    |----------|--------------------|------------------|------------------------|
    | Full snapshot | Read access | Yes | Yes |
    | Watermark on `updated_at` | Read access, trustworthy column | No | Only if column is maintained |
    | CDC (log-based) | DBA configuration changes | Yes | Yes |

    **Watermark loads** are the pragmatic default and the one to propose first. The failure mode to name: if `updated_at` is set by application code rather than a database trigger, some write path will forget it, and those rows become invisible to you forever. Mitigate with a periodic full reconciliation (weekly count and checksum by partition) rather than trusting the column.

    **CDC** on a mature platform is built on Debezium, and the interviewer wants to hear that it is a *negotiation*, not a config toggle. Oracle requires one-time system configuration changes, privilege grants and supplemental logging enabled at table level. PostgreSQL requires logical replication. Those are asks against a DBA who does not report to you and whose change window is monthly. CDC syncs then behave like streaming syncs with changelog metadata propagated, which is why they suit data that is edited rather than append-only.

    **Idempotency is the actual answer to "safe to re-run."** Design so that running the same job twice produces the same table:

    ```sql
    -- Deduplicate the changelog to one row per key, newest wins, then MERGE.
    WITH ranked AS (
        SELECT *,
               ROW_NUMBER() OVER (
                   PARTITION BY order_id
                   ORDER BY change_lsn DESC, change_ts DESC
               ) AS rn
        FROM stg_orders_cdc
        WHERE ingest_batch_id = :batch_id
    )
    MERGE INTO orders AS t
    USING (SELECT * FROM ranked WHERE rn = 1) AS s
       ON t.order_id = s.order_id
    WHEN MATCHED AND s.op = 'D' AND s.change_lsn > t.change_lsn
        THEN UPDATE SET is_deleted = TRUE, deleted_at = s.change_ts,
                        change_lsn = s.change_lsn, updated_at = s.change_ts
    WHEN MATCHED AND s.op <> 'D' AND s.change_lsn > t.change_lsn
        THEN UPDATE SET status = s.status, total = s.total,
                        is_deleted = FALSE, deleted_at = NULL,
                        change_lsn = s.change_lsn, updated_at = s.change_ts
    WHEN NOT MATCHED AND s.op <> 'D'
        THEN INSERT (order_id, status, total, is_deleted, change_lsn, updated_at)
             VALUES (s.order_id, s.status, s.total, FALSE, s.change_lsn, s.change_ts);
    ```

    Three properties make that re-runnable: the changelog is deduped to one row per key, **every** matched branch (delete included) is guarded by the monotonic sequence and advances `t.change_lsn`, so a replayed or late change cannot regress a row, and deletes are soft so downstream joins do not silently lose history. Note the two details candidates usually miss: putting the `change_lsn` guard only on the update branch lets a re-delivered delete re-apply on top of a newer state, and omitting `is_deleted` from the insert leaves the flag undefined for fresh rows even though the soft-delete logic depends on it.

    **Watermark bookkeeping, with the overlap that saves you:**

    ```python
    OVERLAP = timedelta(minutes=5)   # covers clock skew and long transactions

    def next_window(state_store, table: str, now):
        last = state_store.get(table) or datetime(1970, 1, 1, tzinfo=timezone.utc)
        return last - OVERLAP, now

    # commit the watermark ONLY after the MERGE commits, never before
    def run(table, now):
        lo, hi = next_window(store, table, now)
        rows = extract(table, lo, hi)
        merge(rows)                       # idempotent by construction
        store.set(table, hi)
    ```

    Deliberately re-reading a five-minute overlap is correct precisely *because* the merge is idempotent. A transaction that opens before your cutoff and commits after it is otherwise lost, and that class of bug surfaces months later as a handful of missing orders nobody can explain.

    **Fail fast rather than propagate.** Declare expectations on the dataset (primary key uniqueness, non-null, row-count bounds) so a failing build aborts before downstream tables are corrupted. Mature pipeline platforms implement this as declarative expectations attached to a dataset's inputs and outputs: when one fails during a build, the build aborts automatically, which saves compute and keeps bad data out of everything downstream. Repairing one broken table is an afternoon; unwinding it from twelve downstream dashboards a week later is a lost sprint.

    **Red flags:** `DELETE` then `INSERT` inside the same job with no transaction; storing the watermark before the write succeeds; assuming `updated_at` is trustworthy; and promising CDC in the scoping call before you have talked to the DBA.

---

### Resolve Customer Entities Across Three Systems That Disagree - Palantir, Google Cloud Interview Question

**Difficulty:** 🟡 Medium | **Tags:** `Entity Resolution`, `Schema Mapping`, `SQL` | **Asked by:** Palantir, Google Cloud

??? success "View Answer"

    **The setup:** the CRM, the billing system and the support desk each have a "customer." The CRM keys on an internal account ID, billing keys on a tax number, support keys on an email address. Nothing joins. This is the single most common reason an FDE deployment slips, and there is no out-of-the-box answer even on a mature platform.

    **Say the uncomfortable thing first:** entity resolution is a *business* decision wearing a technical costume. Whether "Acme Corp (Bristol)" and "ACME Corporation Ltd" are one customer depends on how the business bills them, not on string distance. Find the person who owns that decision before you write matching logic.

    **The pipeline, in the order you should describe it:**

    1. **Normalise.** Lowercase, strip punctuation and legal suffixes, standardise country codes, parse addresses, validate emails and tax IDs against their formats. Half the "hard" matching problem disappears here.
    2. **Deterministic pass.** Join on any genuinely unique identifier: tax number, DUNS, verified email. Every pair matched here is one you never have to pay for probabilistically.
    3. **Block.** Never compare all pairs. 1M records is 500 billion comparisons. Block on postcode prefix, normalised name first token, or the domain of the email so you only compare within small buckets.
    4. **Score within blocks.** Combine similarities into one score with weights the business can read.
    5. **Three-way decide.** Auto-accept above a high threshold, auto-reject below a low one, and route the middle band to a human review queue. That queue is a feature, not an admission of failure.
    6. **Persist a stable surrogate ID.** Downstream applications must key on your resolved ID, never on the source key, or every re-run reshuffles the world.

    **Blocked fuzzy match in SQL:**

    ```sql
    WITH crm AS (
        SELECT account_id,
               REGEXP_REPLACE(LOWER(name), '\s+(inc|ltd|llc|gmbh|plc)\.?$', '') AS nm,
               LEFT(postcode, 4) AS blk
        FROM crm_accounts
    ),
    bil AS (
        SELECT billing_id,
               REGEXP_REPLACE(LOWER(legal_name), '\s+(inc|ltd|llc|gmbh|plc)\.?$', '') AS nm,
               LEFT(postcode, 4) AS blk
        FROM billing_customers
    )
    SELECT c.account_id, b.billing_id, c.nm, b.nm,
           1.0 - (LEVENSHTEIN(c.nm, b.nm)::NUMERIC
                  / GREATEST(LENGTH(c.nm), LENGTH(b.nm))) AS name_sim
    FROM crm c
    JOIN bil b USING (blk)                        -- blocking key does the pruning
    WHERE LEVENSHTEIN(c.nm, b.nm) <= 4
    ORDER BY name_sim DESC;
    ```

    That shape is the one to reach for on any platform: an edit-distance expression inside a blocked join, so the expensive comparison only ever runs within a bucket. Resist the urge to make an LLM the matcher. A model is a reasonable assistant for the human review queue, where it can explain why two records look alike, but it is the wrong tool for the deterministic, auditable, re-runnable core of entity resolution, and you will not be able to explain a merge decision to the customer six months later.

    **Weighted scoring in Python:**

    ```python
    WEIGHTS = {"name": 0.45, "postcode": 0.25, "domain": 0.20, "phone": 0.10}

    def score(a, b) -> float:
        s = {
            "name":     jaro_winkler(a.norm_name, b.norm_name),
            "postcode": 1.0 if a.postcode == b.postcode else 0.0,
            "domain":   1.0 if a.email_domain == b.email_domain else 0.0,
            "phone":    1.0 if a.phone_e164 == b.phone_e164 else 0.0,
        }
        return sum(WEIGHTS[k] * v for k, v in s.items())

    def decide(s: float) -> str:
        if s >= 0.90: return "auto_match"
        if s <= 0.55: return "no_match"
        return "review"          # sized so a human can clear it daily
    ```

    **Survivorship rules.** Once two records match you still have to pick which values win. Make this explicit and per-field, for example: legal name from billing, mailing address from CRM, contact email from the most recently updated record. Keep every source value with its provenance so a user who disagrees can be shown where each field came from.

    **How you know it works:** hand-label 300 pairs with the business owner, then track precision and recall. Precision matters more than recall here because a false merge shows two customers each other's invoices, which is a security incident, while a false split is a nuisance. Re-measure after every threshold change.

    **Red flags:** proposing an all-pairs cross join; a single global threshold with no review band; matching on names alone; and treating the resolved ID as disposable so it changes on every run.

---

### The Customer Insists Their Data Is Clean. How Do You Triage It? - OpenAI, Palantir Interview Question

**Difficulty:** 🔴 Hard | **Tags:** `Data Quality`, `Stakeholder Management`, `SQL` | **Asked by:** OpenAI, Palantir, Databricks

??? success "View Answer"

    **Why this is a hard question:** it is technical and political at once. The constant in this job is that the customer's description of their data will not match what you find in it, and the person who described it is usually not lying, just describing the system as designed rather than as operated. Your job is to find the gap and report it without making the person who built the system your enemy.

    **Rule one: bring evidence, never adjectives.** "Your data is messy" starts a fight. "17.3% of the 2.4M rows in `orders` have a `customer_id` that does not exist in `customers`, concentrated in the 2019 to 2021 range, which is when you migrated from the legacy CRM" starts a working session. Profile before you meet.

    **A profiling pass you can run in an hour:**

    ```sql
    SELECT
        COUNT(*)                                                   AS n_rows,
        COUNT(DISTINCT order_id)                                   AS n_keys,
        COUNT(*) - COUNT(DISTINCT order_id)                        AS dup_keys,
        ROUND(100.0 * COUNT(*) FILTER (WHERE customer_id IS NULL)
              / COUNT(*), 2)                                       AS pct_null_cust,
        ROUND(100.0 * COUNT(*) FILTER (WHERE total < 0)
              / COUNT(*), 2)                                       AS pct_negative,
        MIN(created_at), MAX(created_at),
        COUNT(DISTINCT currency)                                   AS n_currencies,
        COUNT(DISTINCT status)                                     AS n_statuses
    FROM orders;

    -- Referential integrity, the check that finds migration scars
    SELECT COUNT(*) AS orphan_orders
    FROM orders o
    LEFT JOIN customers c ON c.customer_id = o.customer_id
    WHERE o.customer_id IS NOT NULL AND c.customer_id IS NULL;

    -- Volume by day: gaps and spikes are process changes, not randomness
    SELECT DATE_TRUNC('day', created_at) AS d, COUNT(*) AS n
    FROM orders GROUP BY 1 ORDER BY 1;
    ```

    **Use a taxonomy so nothing is missed.** The five categories below are the standard vocabulary for data health checks on mature platforms, and running through them out loud is a fast way to show an interviewer you have operated a pipeline rather than only built one:

    | Category | Checks | What it catches |
    |----------|--------|-----------------|
    | Status | Schedule, build, job, sync status | The pipeline stopped and nobody noticed |
    | Time | Freshness, build duration, sync duration | Data is stale but present, the worst failure |
    | Size | Row count, file count, partition, transaction size | Silent truncation, runaway growth |
    | Content | Primary key uniqueness, null percentage, mean/median, regex, allowed values | Semantic drift, sentinel values |
    | Schema | Column existence and type, column count, full comparison | An upstream team renamed a column on Friday |

    Note the primary key check is defined precisely: it verifies that the values in a column are 100% unique and non-null. Bulk-apply these across a pipeline via lineage rather than hand-writing them per dataset.

    **The five findings you will actually get, in rough order of frequency:**

    1. **Definitional disagreement.** Finance and operations each have a "shipped date" that differs by two days. This is not dirt, it is two correct answers to different questions, and it needs a business owner to arbitrate.
    2. **Migration scars.** Orphan keys and duplicate records clustered in one date range from a system cutover.
    3. **Sentinel values.** `9999-12-31`, `-1`, `UNKNOWN`, `0` standing in for null and quietly poisoning averages.
    4. **Free-text where there should be an enum.** Fourteen spellings of one status field.
    5. **Freshness that is worse than believed.** The nightly job has failed silently for six weeks and the dashboard shows stale numbers with no staleness indicator.

    **How to deliver it.** Frame findings against the workflow, not against the data team: "to hit the 15-minute freshness this workflow needs, we have to solve the nightly job's silent failure. Here are three options and what each costs." Bring the fix, propose a named owner per issue, and offer to build the health checks yourself so it never regresses. That converts an audit into a shared to-do list.

    **Then design so quality is enforced, not hoped for.** Code-defined expectations on dataset inputs and outputs abort a build when violated, which is strictly better than shipping bad numbers to an executive dashboard. Add a staleness indicator to the user-facing view so users can see for themselves when data last landed; nothing destroys trust faster than a confident number computed on last month's data.

    !!! tip "Interviewer's Insight"
        **Strong answer signals:** profiles before opining; distinguishes definitional disagreement from actual corruption; names an owner per issue; proposes automated checks rather than a one-off cleanup; keeps the customer's dignity intact.

        **Red flags:** "I'd tell them their data is bad and wait"; proposing a six-month data-quality programme before the first workflow ships; cleaning data silently so the customer never learns their source system is broken.

---

### Design a Terabyte-Scale Mixed-Format Ingestion Pipeline With a Freshness SLA - Palantir, Google Cloud Interview Question

**Difficulty:** 🔴 Hard | **Tags:** `System Design`, `Pipelines`, `SLA` | **Asked by:** Palantir, Google Cloud, Databricks

??? success "View Answer"

    **The prompt:** ingest terabytes of sensor data arriving as JSON, CSV and XML, then surface equipment failure predictions to a non-technical operator. This is a reported Palantir FDE system design question, and the trap is that the modelling is the easy part.

    **Scope before you draw.** Ask these four, and expect the answers to change your design:

    - **Who acts on the output, and what do they do differently?** If a maintenance planner schedules an engineer, the SLA is hours and you need a work-order integration. If a line operator halts a machine, the SLA is seconds and you need streaming.
    - **What is the freshness SLA, and what is the cost of missing it?** "Real time" almost never means real time; it usually means "before the 07:00 shift handover."
    - **Cost of a false positive vs a false negative?** An unnecessary teardown costs a shift of production. A missed failure costs a week. That ratio sets your threshold, not the ROC curve.
    - **What is the current process?** There is always an incumbent, usually a spreadsheet, and it is your baseline.

    **Architecture:**

    ```
    SOURCES                    LANDING              STANDARDISE           SERVE
    ┌──────────────┐      ┌───────────────┐    ┌────────────────┐   ┌──────────────┐
    │ Kafka (JSON) │─────▶│  RAW zone     │───▶│ parse + schema │──▶│ feature +    │
    │ SFTP (CSV)   │      │  immutable    │    │ unit normalise │   │ scoring job  │
    │ S3 (XML)     │      │  partitioned  │    │ dedupe by key  │   └──────┬───────┘
    │ Historian API│      │  by ingest_ts │    │ expectations   │          │
    └──────────────┘      └───────────────┘    └────────────────┘          ▼
                                 │                      │           ┌──────────────┐
                                 │  replay              │  abort    │ Operator UI  │
                                 ▼                      ▼           │ ranked list  │
                          ┌───────────────┐      ┌────────────┐     │ + freshness  │
                          │ backfill path │      │ quarantine │     │ + why banner │
                          └───────────────┘      └────────────┘     └──────────────┘
    ```

    **Design decisions to defend:**

    - **Land raw and immutable, partitioned by ingest time.** Parsers will be wrong. If the raw bytes are kept, a parser fix is a replay; if not, it is a data loss incident and an apology to the customer.
    - **One canonical sensor reading schema.** Normalise all three formats into `(asset_id, sensor_id, ts_utc, value, unit, quality_flag, source_file)` at the standardise step. Unit normalisation is not a footnote: mixed Celsius and Fahrenheit from two plants is a classic and it silently destroys a model.
    - **Quarantine, do not drop.** Malformed records go to a quarantine table with the reason and the source offset. A quarantine rate that jumps from 0.2% to 4% is an early warning that an upstream system changed.
    - **Dedupe on `(asset_id, sensor_id, ts_utc)` keeping the latest ingest.** Sensor gateways replay on reconnect, so duplicates are guaranteed.
    - **Late data.** Edge devices buffer offline and dump hours later. Choose an allowed lateness window explicitly and reprocess affected windows rather than pretending it does not happen.

    **Turn the SLA into checks with numbers.** An SLA nobody measures is a sentence in a slide deck:

    | Check | Threshold | Action |
    |-------|-----------|--------|
    | Data freshness | max `ts_utc` within 20 min | Page on-call |
    | Sync duration | < 8 min per cycle | Warn, investigate before it breaches |
    | Row count vs 7-day median | within ±30% | Warn, likely upstream outage |
    | Quarantine rate | < 1% | Warn, schema drift suspected |
    | Primary key uniqueness | 100% unique and non-null | Abort build |
    | Schema comparison | exact match | Abort build |

    Build the abort behaviour in rather than alerting after the fact: when an expectation fails during a build, aborting saves compute and prevents bad data reaching downstream consumers. Also set the SLA with headroom. If the business needs 30-minute freshness, alert at 20 so you have room to fix it before anyone is affected.

    **Surfacing to a non-technical user is half the job.** The operator screen should show a ranked list of assets by risk, the two or three signals driving each score, the recommended action, and a visible "data as of HH:MM" indicator. When the pipeline is degraded, degrade the UI honestly with a banner rather than showing a stale score as if it were live. And close the loop: let the operator mark a prediction right or wrong, because that feedback is both the adoption mechanism and your only source of production labels.

    **Do not skip the operational tail.** A runbook naming the on-call owner, expected recovery steps, and how to trigger a replay is a deliverable, not a nice-to-have. The classic forward deployed week is exactly this shape: building and maintaining large pipelines, configuring access controls that satisfy a regulator, designing workflows so non-technical users can act on high-noise data, and investigating the outage nobody scheduled.

    !!! tip "Interviewer's Insight"
        **What they're testing:** whether you can hold data engineering, ML and a human workflow in your head at once, then land on something shippable. The standard to hit is a solution that works before there is time to make it perfect: name the alternatives and the trade-offs, then commit to one functioning approach and expand it afterwards.

        **Strong answer signals:** asks who acts on the output before designing; names an explicit freshness SLA with thresholds and owners; keeps raw data replayable; quarantines rather than drops; ends on adoption and the operator's trust, not on the model.

        **Red flags:** starting with model architecture; "real time" with no number attached; no story for late or duplicated sensor data; and no plan for what the user sees when the pipeline is behind.

---

## Deployment, Debugging and Operations

### How Do You Choose Between Single-Tenant, Multi-Tenant and Customer-Hosted Deployment? - Palantir, Databricks Interview Question

**Difficulty:** 🟢 Easy | **Tags:** `Tenancy`, `Deployment Architecture`, `Trade-offs` | **Asked by:** Palantir, Databricks, Anthropic

??? success "View Answer"

    **What the interviewer is probing for:** whether you treat isolation as a spectrum with costs attached, or as a binary you assert. Candidates who answer "we'd just give them their own instance" without pricing it lose the round.

    **The four models (Azure Architecture Center taxonomy, the right vocabulary to use):**

    | Model | What it is | Buy it when | The bill |
    |---|---|---|---|
    | Automated single-tenant | One full stack per customer, deployed by automation | Data isolation is contractual, noisy neighbours are unacceptable | Cost scales close to linearly: 100 tenants means roughly 100 copies of the infrastructure bill |
    | Fully multitenant | One shared stack, tenant ID on every row and request | Many small customers, fast iteration, one thing to upgrade | Any change can affect the entire customer base at once, and you eventually hit platform resource quotas |
    | Vertically partitioned | Some tiers shared, some dedicated per tenant | Only the data plane needs isolation, control plane can be shared | The codebase has to support both shared and dedicated deployment, which taxes every feature you ship |
    | Horizontally partitioned | Shared design, sharded into stamps or supertenants | You need blast-radius control without per-customer cost | Requires a tenant to deployment mapping table and a rebalancing story |

    **Say this out loud:** isolation is a spectrum, and different tiers of one architecture can sit at different points on it. The database can be single-tenant while the ingest workers are shared.

    **The operational argument for single-tenant that candidates forget:** updates can be rolled out progressively across tenants, which reduces the likelihood of a system-wide outage. Isolation is a release-safety feature, not only a data-privacy feature.

    **Vendor-hosted versus customer-hosted is a separate axis, and it has feature skew.** Never claim parity. Running a vendor's models through a cloud marketplace platform such as Amazon Bedrock reliably lags that vendor's own first-party API on newer surfaces (file handling, server-side tools, connector protocols, batch APIs and packaged capabilities), and it imposes its own limits, for example a 20 MB payload cap. Do not enumerate the current gaps from memory: pull the platform's feature matrix for the exact model and region in week one and design against that document, because the list moves every few months. Model lifecycle dates on a partner-operated platform are also set by that partner and can differ from the vendor's own schedule. That last one is the trap, because a deprecation you did not schedule can force an upgrade into an environment you do not control.

    So the customer-hosted conversation is really four questions:

    - **Which features do you lose**, and does the use case depend on any of them?
    - **Who patches it**, and what is your access when it breaks?
    - **Whose deprecation calendar governs**, yours or the platform partner's?
    - **What is the egress and residency path** for both data and telemetry?

    **Make the mapping explicit in config, not in tribal knowledge:**

    ```yaml
    # tenants.yaml, source-controlled, one row per customer
    tenants:
      - id: acme-health
        model: single_tenant          # regulated workload, dedicated stack required
        stamp: use1-stamp-03
        residency: us-east-1
        data_plane: dedicated
        control_plane: shared
      - id: northwind-retail
        model: horizontally_partitioned
        stamp: use1-stamp-01
        residency: us-east-1
        data_plane: shared
        control_plane: shared
    ```

    **Strong answers also cover:**

    - A **migration path from shared to dedicated**, because your first enterprise customer will demand it after signing on the shared tier.
    - **Isolation testing by fault injection**, not by assertion. Prove a tenant cannot read another tenant's rows by trying it in a test, on every release.
    - Noisy-neighbour controls in the shared model: per-tenant rate limits, per-tenant quotas, and a per-tenant cost line so you know who is expensive.

    **Red flags:** treating "multi-tenant" and "insecure" as synonyms; promising the customer a dedicated deployment without knowing whether the codebase supports one; quoting parity between a vendor-hosted and partner-hosted deployment without checking the feature matrix.

---

### Deploying Into a Customer VPC, On-Premises or Air-Gapped Network: What Actually Breaks? - Sierra, Palantir Interview Question

**Difficulty:** 🟡 Medium | **Tags:** `Customer VPC`, `Air-Gapped`, `Networking`, `Deployment` | **Asked by:** Sierra, Palantir, Google Cloud

??? success "View Answer"

    **Framing:** the demo worked on your laptop because your laptop had DNS, egress, a package registry and a valid TLS chain. A customer environment is defined by which of those it takes away. Forward deployed infrastructure postings describe the required surface consistently: cloud services and Terraform, container orchestration, and cloud networking (VPCs, IAM, DNS, load balancing), plus a track record of shipping into customer-owned environments and surviving their security review, compliance requirements and change management process.

    **The failure list, roughly in the order it bites you:**

    1. **Egress.** There is no route to the public internet, or there is one through an inspecting proxy that terminates TLS with a private CA. Your SDK does not trust that CA and every call fails with a certificate error that looks like an auth bug.
    2. **Image pull.** The cluster cannot reach your registry. You need registry mirroring and injected pull secrets, and the component that performs that rewrite has to be installable with zero dependencies on anything else in the environment. Design it as the first thing installed, because the thing that fixes image pull cannot itself require a working image pull.
    3. **Inbound connectivity.** You usually do not get any. Design agents that **poll** rather than accept pushes, assume an environment may be disconnected for long periods, and make reconciliation on reconnect the normal path rather than the recovery path.
    4. **Source connectivity.** Data connections should be **agent-based, not direct**: an agent running inside the customer network reaches out, rather than you asking for an inbound hole to a database. Private link and VPC connectivity on the major clouds keep the data path off the public internet, which is usually a hard security-review requirement, not a preference.
    5. **Database prerequisites you cannot self-serve.** Log-based change data capture needs the customer's DBA to act: Oracle requires one-time system configuration changes, privilege grants and supplemental logging enabled at table level, and PostgreSQL requires logical replication. Discover this in week one, not week six, because it is a change-management ticket, not a code change.
    6. **Credentials.** The security team will not hand you production credentials. Plan for a customer-operated runbook and a break-glass path from day one.
    7. **Observability.** In a fully air-gapped deployment you should assume the team that built the software has no access at all to its metrics, logs or runtime. Design for that before you deploy, not after: ship the diagnostics with the release, make them collectable by someone else, and never let your debugging story depend on a dashboard you can open.

    **A pre-flight checklist worth naming in the interview:**

    ```bash
    # Run inside the target environment, before anything else is installed.
    # Each line answers a question the customer's network team cannot answer from memory.
    getent hosts your-service.internal              # split-horizon DNS resolving?
    curl -sv https://registry.internal/v2/ 2>&1 \
      | grep -E 'issuer|subject'                    # which CA terminates TLS?
    echo | openssl s_client -connect api.vendor:443 -showcerts   # proxy interception?
    nc -zv db.customer.internal 5432                # security group / firewall reality
    kubectl get nodes -o wide                       # arch (arm64?), kernel, node count
    df -h /var/lib/containerd                       # disk for images you cannot re-pull
    date -u                                         # clock skew breaks SAML and TLS
    ```

    **Terraform posture the security review will ask about:**

    ```hcl
    resource "aws_vpc_endpoint" "vendor_api" {
      vpc_id              = var.customer_vpc_id
      service_name        = var.vendor_privatelink_service
      vpc_endpoint_type   = "Interface"
      private_dns_enabled = true                # keeps the data path off the internet
      security_group_ids  = [aws_security_group.egress_only.id]
    }

    resource "aws_iam_role" "deploy" {
      name                 = "fde-deploy"
      permissions_boundary = var.customer_boundary_arn   # customer keeps the ceiling
      assume_role_policy   = data.aws_iam_policy_document.trust.json
    }
    ```

    **Strong answers cover:** a documented minimal footprint (what you install and what it depends on), a written egress list of hostnames and ports handed to the network team as an artifact, an offline install path with pre-staged images, clock and CA assumptions stated explicitly, and a named owner on the customer side for each prerequisite.

    **Red flags:** assuming outbound internet; needing inbound ports; discovering the private CA during the go-live window; treating the security review as an obstacle rather than a deliverable with a schedule.

---

### You Cannot Access Production, You Cannot Export Logs and the Customer Is Watching. How Do You Debug? - Palantir, OpenAI Interview Question

**Difficulty:** 🔴 Hard | **Tags:** `Debugging`, `Incident Response`, `Air-Gapped`, `Customer Communication` | **Asked by:** Palantir, OpenAI, Cognition

??? success "View Answer"

    **This is the round that separates FDEs from strong SWEs.** You are being tested on two things at once: a disciplined debugging method under an information blackout, and your composure while a customer stakeholder watches you work. Forward deployed job descriptions name the second one as a hiring criterion in as many words: staying calm and exercising judgement when the stakes are high is part of the job specification, not a personality bonus.

    **Split the clock immediately.** Run two tracks in parallel and say so out loud:

    - **Track A, mitigation:** restore service. Roll back, fail over, disable the feature flag, drop to the degraded path.
    - **Track B, diagnosis:** find the cause. This track never blocks Track A.

    **The information problem.** In a genuinely disconnected deployment, diagnostics move at human speed: someone on the customer's side has to collect them, review them, and carry them out of the enclave, and every round trip can cost hours. Design for that up front rather than improvising it during an incident:

    - **Pre-defined operator runbooks** so the customer's operator can collect what you need without you dictating commands live over a phone line.
    - **Scrubbing built into the collection script**, so the operator is never the person deciding whether a log line is safe to send.
    - **Auto-repair** for the boring failures: restart, scale, re-queue. Anything that can be fixed without a human should never consume an information round trip.
    - **Short numeric error codes** that map to a catalogue you hold, so a phone call from an operator who cannot send you a file still carries real information.
    - **Local summarisation of logs** inside the environment, so aggregate signal (counts, anomaly flags, first and last occurrence) can leave even when raw text cannot.

    **Ask for a bounded artifact, not "the logs."** A single scripted bundle the operator can run and scrub beats ten rounds of "can you also send...".

    ```bash
    #!/usr/bin/env bash
    # collect-bundle.sh, shipped with the release, run by the CUSTOMER operator.
    set -euo pipefail
    OUT="bundle-$(date -u +%Y%m%dT%H%M%SZ)"; mkdir -p "$OUT"

    kubectl get pods -o wide                     > "$OUT/pods.txt"
    kubectl top pods --containers                > "$OUT/usage.txt"
    kubectl describe pod -l app=ingest           > "$OUT/describe.txt"
    kubectl logs -l app=ingest --since=2h --tail=5000 \
      | sed -E 's/[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+/<EMAIL>/g;
                s/[0-9]{1,3}(\.[0-9]{1,3}){3}/<IP>/g;
                s/"token":"[^"]*"/"token":"<REDACTED>"/g'  > "$OUT/app.log"

    tar czf "$OUT.tgz" "$OUT"   # operator reviews, then transfers out of band
    ```

    **The method, and it is the same method a re-engineering round tests:** form hypotheses, test them, refine them. Reject the instinct that wrecks candidates, which is to look at unfamiliar code, decide it is bad and propose a rewrite. In a customer environment the rewrite is never available to you, and in an interview it reads as an inability to work inside someone else's constraints.

    - **Anchor on the delta.** What changed: a release, a config value, a data volume, a certificate expiry, a customer-side upgrade. Most customer-environment incidents are environment changes, not code changes.
    - **Reproduce outside production.** Rebuild the failing input shape in a local harness from the pseudonymised bundle. If you cannot reproduce, your hypothesis is not specific enough yet.
    - **Bisect by boundary,** not by line: is it ingest, transform, serve, or the customer's upstream source? Health checks give you this cheaply if you built them in.
    - **Ask for one decisive measurement at a time.** Each operator round trip may cost an hour, so spend it on the observation that splits your hypothesis space in half.

    **Communication while they watch.** This is graded as heavily as the fix.

    - Acknowledge with scope and impact in plain language, then give the **time of the next update**, and hit it even when you have nothing new.
    - Say "I" not "we"; individual ownership is the trait being assessed.
    - Separate **what is known** from **what is suspected**. Never name a root cause you have not tested; a retracted root cause costs more trust than an hour of silence.
    - Do not blame the customer's infrastructure before you have investigated. This is reported as the single weakest answer to the outage scenario, even when the infrastructure turns out to be the cause.

    **Close the loop like an FDE, not a firefighter:** a written timeline, the specific detection gap ("we had no alert on sync freshness"), the health check or data expectation you are adding so the next occurrence is caught by the system rather than by the customer, and whether the fix belongs in this deployment or in the product.

---

### Design an Upgrade Strategy for Software Running in Environments You Do Not Control - Palantir, Databricks Interview Question

**Difficulty:** 🔴 Hard | **Tags:** `Version Skew`, `Release Engineering`, `Migrations`, `Fleet Management` | **Asked by:** Palantir, Databricks, Google Cloud

??? success "View Answer"

    **Why standard CI/CD fails here, stated precisely:** a pipeline couples a service to an environment. Give every customer environment its own pipeline and each one evolves independently, so the pipelines themselves drift apart and nobody can say what version any given customer is actually running. The drift is not in the software, it is in the delivery mechanism.

    **The shape that works is a constraint-based control loop, not a pipeline.** Instead of pushing a build at an environment, you publish a target state and let each environment converge on it when its own constraints allow. The planner only ever recommends steps toward the target that satisfy every declared constraint, which is what makes one intent safe to apply across environments that are nothing like each other: a cloud tenant, an on-premises rack and a disconnected field installation all read the same target and each moves as far as it legally can.

    **Four mechanisms to name:**

    **1. Declare dependencies bidirectionally.** Each release ships a manifest declaring minimum and maximum versions of the services it depends on, and dependency constraints are applied **both ways**, blocking forward-incompatible and backward-incompatible deployments alike. Releases can also declare outright incompatibilities, for example against a range of database versions known to break them, and those apply bidirectionally too.

    ```yaml
    # release manifest, shipped WITH the artifact so expectations travel with the code
    product: ingest-service
    version: 4.7.2
    dependencies:
      - product: ontology-service
        minimum-version: 4.5.0
        maximum-version: 5.0.0
    incompatibilities:
      - product: postgresql
        minimum-version: 9.3.6
        maximum-version: 9.6.99
    schema:
      supported-versions: { minimum: 31, maximum: 33 }   # phased migration window
    ```

    **2. Treat schema migrations as a deployment constraint.** Validate that the target release's supported schema version range contains the environment's current schema version before the upgrade is even planned. That is what makes a multi-release migration across a fleet you do not control possible: release A supports schema 31 to 32 and writes both, release B supports 32 to 33 and stops reading 31. Expand, migrate, contract, spread over three releases and however many months the slowest customer needs.

    **3. Pull, do not push.** Environments **subscribe** to release channels (development, release candidate, release) and upgrade during their own maintenance windows to the latest release satisfying all constraints. Promotion between channels is gated on soak duration, canary readiness and overall timeouts, with manual promotion available during incidents.

    **4. Engineer the blast radius explicitly.**

    | Control | What it does |
    |---|---|
    | Maintenance windows | Distinguish downtime operations from no-downtime operations per environment |
    | Suppression windows | Created manually, automatically after a plan failure, or during promotion |
    | Artifacts-missing constraint | Blocks a plan outright when release artifacts cannot be pulled |
    | Recall | Withdraws a bad release fleet-wide, takes priority over normal planning |
    | Break-glass | Operator override that also outranks normal planning |

    **The FDE-specific judgement questions to raise:**

    - **Who owns the upgrade decision?** In a customer-hosted deployment the customer owns the maintenance window, so your rollout plan is a negotiation, not a schedule.
    - **How long is your support tail?** If a disconnected environment reconciles after three months, your N-3 compatibility promise is not theoretical.
    - **How do you find out an upgrade failed** when the environment cannot report to you? Expected-state reconciliation with health reporting on reconnect, plus operator-runnable verification, is the answer.
    - **What is the rollback story for data?** Code rolls back cleanly; a schema migration that dropped a column does not.

    **Red flags:** proposing "just force everyone onto the latest version"; a migration plan that assumes all customers upgrade in the same week; version pinning in one direction only; no answer for the environment that has been offline for a quarter.

---

### How Do You Customize for One Customer Without Forking the Product? - OpenAI, LangChain Interview Question

**Difficulty:** 🟡 Medium | **Tags:** `Feature Flags`, `Configuration`, `Productization`, `Technical Debt` | **Asked by:** OpenAI, LangChain, Palantir

??? success "View Answer"

    **Frame it as a cost-of-ownership question, not a purity question.** Every customization is a bet on how long you will be maintaining it and how many people will touch it. The right answer is almost never "no" and almost never "fork"; it is picking the cheapest mechanism that actually holds the customer's requirement, and being able to say out loud what that mechanism will cost to own in two years. Senior forward deployed postings write this in as an explicit responsibility: align early, in writing, on what should generalise, what stays customer-specific, and what "ready for handoff" concretely means for this piece of work.

    **The escalation ladder, cheapest to most expensive.** Say where you would stop and why.

    | Mechanism | Fits | Cost of ownership |
    |---|---|---|
    | Configuration (source-controlled) | Values that differ per customer: limits, endpoints, field mappings | Lowest; reviewable, diffable, rollback-able |
    | Semantic layer / mapping | Customer schema differences | Moderate; needs a modelling discipline |
    | Feature flags | Behaviour that must differ or be switchable at runtime | Medium, and it compounds if ungoverned |
    | Plugin or extension point | Genuinely customer-specific logic | High; you now own an interface contract forever |
    | Fork | Almost never | Highest; two codebases, two upgrade paths, permanent drift |

    **Feature flags, using Fowler's taxonomy, because "feature flag" alone is not an answer:**

    - **Release toggles:** transitional, days to weeks, static config is fine.
    - **Experiment toggles:** dynamic per request, static config.
    - **Ops toggles / kill switches:** need fast reconfiguration **without a redeploy**, which is exactly what you want when a customer's environment is degrading at 2am.
    - **Permissioning toggles:** can live for years, dynamic per request on user identity.

    Two axes decide the implementation: how dynamic the routing decision must be, and how long the toggle will live.

    ```yaml
    # flags.yaml in source control; ops toggles additionally readable from a runtime store
    toggles:
      new_ranking_pipeline:
        type: release           # transitional, delete after rollout
        default: false          # OFF = existing/legacy behaviour, ON = new behaviour
        owner: fde-acme
        expires: 2026-09-30     # inventory to be paid down, not a permanent branch
      strict_pii_redaction:
        type: permissioning     # long-lived, evaluated per request
        default: true
      disable_llm_enrichment:
        type: ops               # kill switch, runtime-reconfigurable, no redeploy
        default: false
    ```

    **Operating rules worth stating:**

    - Manage toggle configuration through source control and re-deployment wherever the nature of the flag allows it. Reach for a runtime store only when the toggle genuinely has to change without a deploy.
    - Toggles are **inventory to be paid down**, with an owner and a removal date.
    - Do not test the combinatorial explosion. Test the **expected production configuration plus all-toggles-off**.
    - Keep the polarity consistent: off means the existing or legacy behaviour, on means the new behaviour. Inverting this on one flag is how a rollback makes an incident worse.
    - The cost of ungoverned toggles is not hypothetical. [Martin Fowler's feature toggle article](https://martinfowler.com/articles/feature-toggles.html) points at Knight Capital, which it describes as "a $460 million dollar mistake."

    **Decouple the customer's data model from your application.** A semantic layer is what stops a customer-specific rename from becoming a code change. The shape to build is a set of business objects with properties and links, defined once and mapped onto whatever the underlying tables happen to be called, plus a small set of named actions that write back through the same layer. Applications and user-facing workflows bind to the objects and actions, never to a source table, so the customer can restructure a source system without breaking the thing their staff use every day.

    **Package the customization so it is portable.** Aim for the customer-specific part to be a versioned artifact rather than a diff against your source tree: it declares its own dependencies, carries input presets for the values that differ per customer, installs into a fresh environment without modification, and can be rolled back on its own. When two customers' packages share content, extract the overlap into an upstream package that both depend on by version. That is the productization move: extract the overlap upstream, keep the presets per customer.

    **On when to generalize, hold this line:** generalising too early is the more expensive mistake. The reusable share of what you build rises across successive engagements rather than appearing on the first, and the engagements that produce the most transferable insight are usually the ones that went deepest on one customer's specific problem. Solve it specifically first, then extract the pattern once you have seen it twice.

    **Red flags:** proposing a fork; a flag with no owner and no expiry; per-customer `if customer_id == "acme"` branches in application code; refusing all customization on purity grounds when the deployment is what you are accountable for.

---

### A Customer Says the System Is Too Slow and Too Expensive on Their Hardware. Triage It. - OpenAI, ElevenLabs Interview Question

**Difficulty:** 🟡 Medium | **Tags:** `Performance`, `Latency`, `Cost Optimization`, `Triage` | **Asked by:** OpenAI, ElevenLabs, Databricks

??? success "View Answer"

    **First move: refuse the premise until it is measured.** "Slow" is not a defect report. Get to a number and a percentile before you touch anything:

    - **Where** in the request: which span, which hop, which stage.
    - **Which percentile**: a p99 complaint and a p50 complaint have different causes and different fixes.
    - **Since when**: correlate against a release, a config change, a data-volume change or a customer-side upgrade.
    - **Against what target**: a latency budget the customer agreed to, or you will optimise forever.

    **The counter-intuitive lever, and interviewers do test it.** For LLM systems, output tokens dominate latency and input tokens barely register: halving the output can take roughly half the wall-clock time out, while halving the prompt typically buys a low single-digit percentage. Candidates instinctively attack the prompt because it is the biggest object on screen. That is usually the wrong end.

    **Seven levers, applied in order of effort against payoff:**

    | Lever | Concrete move |
    |---|---|
    | Process tokens faster | Smaller model for the sub-task; predicted outputs where the shape is known |
    | Generate fewer tokens | Cap output; ask for structured fields rather than prose; stop the model narrating |
    | Use fewer input tokens | Trim retrieved context; deduplicate; retrieve top-150 then rerank to top-20 rather than stuffing |
    | Make fewer requests | Combine sequential calls; consolidate tools so one call does what three did |
    | Parallelize | Fan out independent retrievals and guardrail checks concurrently |
    | Make users wait less | Stream; chunk; show progress. Perceived latency is the one the customer reports |
    | Do not default to an LLM | Hardcoding, pre-computation or a traditional algorithm where it fits |

    **Prompt caching is the highest-leverage cost lever, and it fails silently.** Cache reads cost 0.1x base input price; 5-minute cache writes cost 1.25x and 1-hour writes 2x; the default TTL is 5 minutes and is refreshed at no additional cost each time the cached content is used; there is a maximum of 4 cache breakpoints per request with a 20-block lookback; and the minimum cacheable prompt length varies by model from 512 to 4,096 tokens, with shorter prompts silently failing to cache and **no error returned**. Invalidation cascades tools, then system, then messages, so the cache breakpoint must sit on the last block whose prefix is identical across requests.

    ```python
    # Cheap instrumentation before any optimisation: attribute time and tokens per stage.
    import time, contextlib

    @contextlib.contextmanager
    def stage(name, sink):
        t0 = time.perf_counter()
        try:
            yield
        finally:
            sink[name] = round((time.perf_counter() - t0) * 1000, 1)  # ms

    spans = {}
    with stage("retrieve", spans):   chunks = retriever.search(q, k=150)
    with stage("rerank", spans):     top = reranker.rank(q, chunks)[:20]
    with stage("generate", spans):   out = model.complete(prompt(q, top), max_tokens=400)

    # Log alongside: input_tokens, output_tokens, cache_read_tokens, cache_write_tokens.
    # Cost and latency regressions are usually visible here before a customer notices.
    ```

    **On classic (non-LLM) customer hardware, the same discipline applies with different suspects:** partition skew on a Spark job where one task holds 90% of the data, a missing index on the customer's source database, a sync running full instead of incremental, disk pressure on nodes that cannot re-pull images, and CPU architecture differences (arm64 nodes, no GPU, older kernel) that never appeared in your test cluster.

    **The 10x traffic spike variant.** Answer in three tiers: **shed** (per-tenant rate limits, queueing with backpressure, degrade to a cheaper model or a cached answer), **scale** (autoscaling limits, quota headroom you must request in advance, batch the batchable), and **communicate** (tell the customer which tier of service is degraded and what it costs to hold the original target). Naming the cost of the ask is what separates an engineer from an order-taker here.

    **Red flags:** optimising without a baseline; reaching for a bigger instance first; claiming a fix without a before-and-after number; ignoring cost entirely, when the customer's complaint was half about the bill.

---

### Which Security and Compliance Topics Must an FDE Speak To Fluently? - Anthropic, Google Cloud Interview Question

**Difficulty:** 🟢 Easy | **Tags:** `Security`, `Compliance`, `RBAC`, `Data Residency` | **Asked by:** Anthropic, Google Cloud, Palantir

??? success "View Answer"

    **Why this is a core skill, not an edge case:** forward deployed postings list governance, risk and compliance alongside product, research and go-to-market as routine collaborators rather than as an occasional gate. You will be in a security review in your first month, and the deployment date moves based on how well you handle it.

    **The enterprise surface you should be able to discuss without notes:**

    - **SSO/SAML** with domain capture, and **SCIM** provisioning so joiners and leavers are automatic rather than a ticket.
    - **RBAC**, plus attribute-based controls where roles alone are too coarse.
    - **Audit logs** that are complete, queryable and tamper-evident.
    - **Standards-based telemetry** (OpenTelemetry traces and metrics) plus a programmatic path for the customer's compliance team to collect evidence without filing a ticket with you.
    - **Data retention controls**, including zero-retention configurations.
    - **SOC 2, ISO 27001, GDPR, CCPA** as table stakes.

    **Access control, the layered model to describe:**

    | Layer | Mechanism | What it buys |
    |---|---|---|
    | Mandatory, organizational | Organization boundaries enforcing strict silos between groups of users and resources | Hard tenancy boundary |
    | Mandatory, data-level | Markings on sensitive data (PII, financial) that propagate through provenance and lineage | A derived dataset inherits its parent's sensitivity automatically |
    | Discretionary | Roles (view, edit) on individual resources | Day-to-day sharing |
    | Attribute-based | Row-level and column-level controls evaluated against the calling user's attributes | One dataset, many audiences |

    The point to make out loud: **markings propagating through lineage** is the property that stops the classic leak, where an analyst joins a restricted table into an unrestricted one and the restriction quietly disappears.

    **Compliance is a configuration, not a checkbox.** Get these distinctions right and you will be trusted in the room:

    - A **HIPAA BAA** covers a configured deployment, not a product, so "HIPAA-ready" is never a property of a model. The specific carve-out worth knowing, because candidates miss it, is that a BAA does not extend to web search functionality: if your architecture reaches the open web on behalf of a covered workflow, that path sits outside the agreement and has to be removed or replaced.
    - **Regulated and government deployments run on their own authorization paths**, and those paths differ by vendor, by hosting platform and by region. Do not recite an authorization matrix from memory in a customer meeting. Name the constraint the customer actually has (a specific accreditation, a residency requirement, an export-control restriction) and then confirm the current authorization status and eligibility rules with the vendor in writing, because these change on their own schedule and being confidently wrong here is worse than saying you will check.
    - **Residency has a price.** Regional endpoints, which you need for residency, carry a 10% premium over global endpoints on Bedrock. Say the number; do not present residency as free.
    - Assume the accreditation question drives your architecture rather than decorating it. Whether the workload can run on a shared endpoint at all determines your deployment target, which determines your available feature set, which determines the design. Ask it in week one.

    **Telemetry is a compliance surface too.** Anything you turn on for debugging can become a data-handling question, and in a regulated deployment the safest default is that no prompt content, model output, tool argument or raw request body ever leaves the environment unless somebody has deliberately approved it. Three properties are worth designing for and worth naming in an interview:

    - **Content redaction on by default.** Emit metrics, timings, token counts and error classes freely. Treat anything containing customer text as a separate, explicitly enabled tier.
    - **Administrator-pinned settings.** The customer's platform team should be able to enforce telemetry configuration centrally, in a location an individual user cannot override, so a single engineer cannot turn on verbose logging in a regulated environment.
    - **A named export destination inside the customer's control.** Point collectors at the customer's own endpoint rather than at anything of yours, and be able to state on a whiteboard exactly which fields leave the boundary and where they land.

    **Audit logging design, a question that shows up directly in these loops** (design an audit log for a platform used by government agencies where every action must be traceable, tamper-evident and queryable by regulators): append-only storage, hash chaining or write-once object storage for tamper evidence, actor plus subject plus resource plus decision plus reason on every entry, a defined retention period tied to the regulation rather than to your storage bill, separate access control for the audit log itself, and a query path a regulator can use without engineering help.

    **Red flags:** saying "we're SOC 2 compliant" as if it answered a data-residency question; asserting an accreditation status from memory instead of confirming it; enabling verbose telemetry in a regulated environment without asking; treating the customer's security team as a gate to route around rather than a stakeholder to schedule.

---

## Deploying AI and LLM Systems

### Taking an LLM Proof of Concept to Production Inside a Customer's Environment - OpenAI, Anthropic Interview Question

**Difficulty:** 🟡 Medium | **Tags:** `LLM Deployment`, `Production Readiness`, `Data Residency` | **Asked by:** OpenAI, Anthropic, Databricks

??? success "View Answer"

    **What the interviewer is probing:** whether you know that a working demo and a deployed system are separated by months of unglamorous work, and whether you can name that work without being prompted. Cursor's job description states the bar bluntly: "This is not a demo role. You are responsible for systems that work in the real world."

    **The gap that matters most: works vs adopted.**

    The best documented example is Morgan Stanley's OpenAI deployment. Getting the technical pipeline working (retrieval, guardrails, functionality) was the fast part. Piloting it and building enough trust for the firm's advisors to rely on it took considerably longer, and only then did the headline result land: **98% advisor-team adoption**. If your answer stops at "the pipeline works," you have described the first part of the job.

    **The production checklist an FDE actually runs:**

    | Dimension | Demo state | Production state |
    |---|---|---|
    | Failure handling | Exception bubbles up | Timeouts, retries with jitter, graceful degradation to a non-AI path |
    | Observability | Print statements | Traces per request, token and cost counters, latency percentiles |
    | Quality | Vibes on 5 examples | Golden set plus regression suite that gates deploys |
    | Identity | Shared API key | Customer SSO, RBAC, per-user audit trail |
    | Data path | Laptop to public API | Agreed egress path, residency and retention terms signed off |
    | Rollout | Everyone at once | Cohort by cohort with a kill switch |
    | Ownership | You | A named customer team with a runbook |

    **Hardening the call itself.** This is also the part of an artifact-driven take-home that separates submissions. A working happy path is table stakes; what gets remarked on is error handling, structured logging and a degradation path that keeps the user's workflow alive when the model tier is unavailable. Write those three in even when the brief does not ask for them, and say why in your walkthrough.

    ```python
    import time, random, logging
    from dataclasses import dataclass

    @dataclass
    class Result:
        text: str
        degraded: bool
        latency_ms: int
        input_tokens: int
        output_tokens: int

    def call_with_guarantees(client, messages, *, model, fallback_fn,
                             timeout_s=20, max_attempts=3) -> Result:
        started = time.time()
        for attempt in range(max_attempts):
            try:
                resp = client.messages.create(
                    model=model, messages=messages,
                    max_tokens=800, timeout=timeout_s,
                )
                return Result(
                    text=resp.content[0].text, degraded=False,
                    latency_ms=int((time.time() - started) * 1000),
                    input_tokens=resp.usage.input_tokens,
                    output_tokens=resp.usage.output_tokens,
                )
            except Exception as exc:
                logging.warning("llm_attempt_failed attempt=%s err=%s", attempt, exc)
                time.sleep(min(2 ** attempt, 8) * (0.5 + random.random() / 2))
        # Never fail the user workflow because the model tier is unavailable
        return Result(text=fallback_fn(messages), degraded=True,
                      latency_ms=int((time.time() - started) * 1000),
                      input_tokens=0, output_tokens=0)
    ```

    **Where the data goes is a design constraint, not paperwork, and it has to be settled before you design.** The deployment target the customer's security team will accept determines which model features you are allowed to build on, and partner-operated platforms routinely lag the vendor's own API on batch APIs, file handling, connectors and payload size limits. Get the feature list for the specific target in writing in week one and design against that list, because discovering in week eight that the connector your architecture assumes is unavailable on the customer's chosen platform is a rewrite, not a config change. The same applies to retention: a zero-retention configuration can be a contractual prerequisite for a regulated customer, and it changes what you are able to log, which changes how you debug. Decide the logging design once you know the retention terms, never the other way round.

    **Progressive autonomy beats a big-bang cutover.** The pattern to describe for a debug-and-triage agent is a staged rollout: advisory output the human acts on first, then drafted changes the human reviews and merges, then supervised automation with a human on the exception path. Each stage buys the evidence that justifies the next, and each gives the customer a defined place to stop without the engagement being a failure.

    !!! tip "Interviewer's Insight"
        **Strong answer covers:** the works-vs-adopted gap with a concrete timeline; degradation paths that keep the human workflow alive; residency and retention decided before build, not after; staged autonomy; and a named handover owner on the customer side.

        **Red flags:** treating security review and data agreements as someone else's problem; a rollout plan with no kill switch; declaring done at "the pipeline runs"; assuming every model feature exists on every deployment target.

---

### Building an Evaluation Harness When the Customer Has No Labelled Data - OpenAI, Sierra Interview Question

**Difficulty:** 🔴 Hard | **Tags:** `Evals`, `LLM Quality`, `Discovery` | **Asked by:** OpenAI, Sierra, Google Cloud

??? success "View Answer"

    **This is the single differentiating question in AI-lab FDE loops.** It usually arrives as "How do you know your AI system is actually working?" Forward deployed postings across these companies name evaluation frameworks as a hard requirement and describe success in terms of eval-driven feedback rather than delivery milestones. Weak candidates answer that they would monitor it. That answer ends the round.

    **Reframe first: the customer always has labels, they are just not in a table.** Your job in discovery is to find them.

    - **Expert action sequences.** Build the eval set with the customer's domain experts before significant development, not after. In a deep debugging workflow that means sitting with the best engineer on their team and writing down the ordered sequence of actions they perform to take a failure from symptom to resolution, which routinely runs to dozens of steps. **That sequence is the label**, and it is also the specification.
    - **Historical outcomes.** Closed tickets, approved claims, resolved cases, and the final human decision on each are retrospective ground truth.
    - **Rejections.** Anything a human overrode, escalated or sent back is a high-value negative example.
    - **Policy documents.** Rules the business already wrote down convert directly into assertions.

    **A workable first harness is 50 to 150 items, not 10,000.** Build it in a week with two domain experts in the room, and treat disagreement between them as a finding, not noise: if two experts label the same case differently, the task specification is ambiguous and the model was never going to win.

    **Structure it as data plus graders.** An eval is a JSONL dataset of items carrying inputs and ground truth, plus testing criteria templated over the sample output and the item label:

    ```jsonl
    {"item": {"question": "Is invoice 4471 eligible for early-pay discount?", "correct_label": "no", "reason_contains": "past 30 day window"}}
    {"item": {"question": "What is the lead time on part XR-9?", "correct_label": "14 days", "reason_contains": "supplier contract"}}
    ```

    Grader types you should be able to name and choose between: **exact string checks** (equality, containment, a normalised match), **text similarity** for answers that are allowed to vary in wording, **code graders** for anything you can assert programmatically, and **model graders** (LLM as judge) for open-ended output where nothing cheaper works. Use the cheapest grader that can detect the failure you care about, and validate any model grader against human labels before you trust its numbers.

    ```python
    from dataclasses import dataclass
    from typing import Callable, List, Dict

    @dataclass
    class Case:
        inputs: Dict
        label: str
        tags: List[str]          # intent, business unit, difficulty

    def exact(pred: str, case: Case) -> bool:
        return pred.strip().lower() == case.label.strip().lower()

    def judge(pred: str, case: Case, client, model) -> bool:
        rubric = (
            "Answer YES only if the candidate answer is factually consistent "
            f"with the reference and cites a source.\nReference: {case.label}\n"
            f"Candidate: {pred}\nAnswer YES or NO."
        )
        out = client.messages.create(model=model, max_tokens=5,
                                     messages=[{"role": "user", "content": rubric}])
        return out.content[0].text.strip().upper().startswith("YES")

    def run(cases: List[Case], predict: Callable, graders: Dict[str, Callable]):
        rows = []
        for c in cases:
            pred = predict(c.inputs)
            rows.append({
                "tags": c.tags,
                **{name: g(pred, c) for name, g in graders.items()},
            })
        return rows   # aggregate per tag, never just a single global number
    ```

    **Slice by intent, never report one aggregate.** A customer-service deployment that grows from a handful of policies to hundreds only stays measurable if instructions are parameterised and every intent carries **its own eval set**. A global accuracy of 91% hides the one intent sitting at 40%, and that intent is the one the customer's executives will see first.

    **Split deterministic from probabilistic and say so out loud.** In a supply-chain style deployment the division is stark: hard business constraints such as minimum supplier counts, lead times and material coverage are verified **in code**, on the model's output, rather than requested of the model in a prompt, while the model handles the open-ended business-intelligence questions and calls simulation or solver tools instead of attempting the optimisation itself. Never ask a probabilistic system to maintain a constraint that must hold.

    **Close the loop.** Evals go in CI as a regression gate before any prompt, model or retrieval change ships. Every production incident becomes a new case. Name the customer-side person who signs off on the pass bar, because in an FDE engagement they, not you, own the threshold.

    !!! tip "Interviewer's Insight"
        **Strong answer covers:** where labels come from when none exist; expert-authored trajectories; grader selection by cost; per-slice reporting; deterministic verification of hard constraints; evals wired into CI; a named human owner of the bar.

        **Red flags:** "we would use an LLM to grade it" with no rubric or spot-check of the judge; one aggregate accuracy number; building 5,000 synthetic cases before talking to a domain expert; treating evals as a pre-launch gate rather than a permanent asset the customer keeps.

---

### Your RAG System Returns Wrong Answers on the Customer's Corpus: How Do You Fix Retrieval? - OpenAI, Anthropic Interview Question

**Difficulty:** 🟡 Medium | **Tags:** `RAG`, `Retrieval`, `Debugging` | **Asked by:** OpenAI, Anthropic, Databricks

??? success "View Answer"

    **Step one is not tuning anything.** Split the failure. For each wrong answer, check whether the passage containing the correct answer was in the retrieved context.

    - Correct passage **absent** from context, answer wrong: a **retrieval** failure. Fix the index.
    - Correct passage **present**, answer still wrong: a **generation** failure. Fix the prompt, the ordering, or the model.
    - Correct passage present and the answer is right but the customer says it is wrong: a **specification** failure. The expected answer lives in a system, a person or a convention nobody documented.

    That third bucket is larger than candidates expect. The model is usually the cleanest part of the job; the hard part is finding the workflow nobody documented and the data source people actually trust.

    **Measure before you tune.** Instrument top-k retrieval recall against the golden set. Anthropic's published contextual retrieval numbers give you a target shape for the improvement ladder, measured as top-20 retrieval failure rate:

    | Configuration | Top-20 failure rate | Relative improvement |
    |---|---|---|
    | Baseline embeddings | 5.7% | reference |
    | Contextual embeddings | 3.7% | 35% fewer failures |
    | Contextual embeddings + contextual BM25 | 2.9% | 49% fewer failures |
    | Above + reranking | 1.9% | 67% fewer failures |

    **The fixes, cheapest first:**

    1. **Chunking.** Keep chunks small enough to stay on one topic, around 800 tokens as a reasonable working size for both retrieval and cost modelling. Chunk on document structure, not a fixed character count, so a table or a clause is not cut in half.
    2. **Contextual retrieval.** Prepend a 50 to 100 token model-generated description of where the chunk sits in its document, **before embedding and before BM25 indexing**. This is what kills the "this paragraph says 'the rate increased by 3%' with no idea which rate" failure. With prompt caching the one-time cost is about **$1.02 per million document tokens**.
    3. **Hybrid retrieval.** BM25 plus embeddings with rank fusion. Lexical search rescues exact identifiers, part numbers and error codes that embeddings smear together.
    4. **Rerank and widen.** Retrieve **top-150**, rerank down to **top-20**. Passing the top-20 chunks is measurably more effective than top-10 or top-5.
    5. **Embedding model.** Benchmark two or three candidates on the customer's own corpus rather than on a public leaderboard, and swap only after the four fixes above, because a swap forces a full reindex and invalidates any retrieval numbers you have already reported.

    ```python
    CONTEXT_PROMPT = """<document>{doc}</document>
    Here is the chunk we want to situate within the document:
    <chunk>{chunk}</chunk>
    Give a short succinct context (50-100 tokens) to situate this chunk
    within the overall document for improving search retrieval.
    Answer only with the context."""

    def contextualize(client, model, doc: str, chunk: str) -> str:
        msg = client.messages.create(
            model=model, max_tokens=150,
            messages=[{"role": "user",
                       "content": CONTEXT_PROMPT.format(doc=doc, chunk=chunk)}],
        )
        return msg.content[0].text.strip()

    def index_chunk(doc, chunk, embed, bm25_index, vector_index, meta):
        ctx = contextualize(client, model, doc, chunk)
        enriched = f"{ctx}\n\n{chunk}"          # context goes into BOTH indexes
        vector_index.add(embed(enriched), payload={"text": chunk, **meta})
        bm25_index.add(enriched, payload={"text": chunk, **meta})

    def reciprocal_rank_fusion(*ranked_lists, k=60):
        scores = {}
        for lst in ranked_lists:
            for rank, doc_id in enumerate(lst):
                scores[doc_id] = scores.get(doc_id, 0) + 1 / (k + rank + 1)
        return sorted(scores, key=scores.get, reverse=True)
    ```

    **Two constraints specific to customer corpora that pure RAG tutorials ignore:**

    - **Permissions must be enforced at query time, not after generation.** Carry the source document's access-control attributes in the chunk payload and filter the candidate set by the calling user's attributes before the model ever sees text. Generating an answer from a document the user cannot open is a data breach even if you redact the citation.
    - **Freshness and duplicates.** Enterprise corpora are full of superseded policy versions. Retrieval that returns the 2019 and 2026 versions of the same policy with equal confidence produces answers the customer will correctly call wrong. Version metadata plus a recency filter is often a bigger win than any embedding change.

    !!! tip "Interviewer's Insight"
        **Strong answer covers:** diagnosing retrieval vs generation vs specification before tuning; contextual chunks written into both the vector and lexical index; retrieve wide then rerank narrow; concrete numbers; access control applied at retrieval; document versioning.

        **Red flags:** jumping straight to "use a bigger context window" or "fine-tune it"; no measurement of retrieval recall; treating chunk size as the only knob; ignoring who is allowed to see the retrieved document.

---

### Choosing Between Prompting, Retrieval, Fine Tuning and a Different Model - Anthropic, Databricks Interview Question

**Difficulty:** 🟢 Easy | **Tags:** `Model Selection`, `RAG`, `Fine Tuning` | **Asked by:** Anthropic, Databricks, Sierra

??? success "View Answer"

    **This question is usually asked because a customer executive has already asked for fine-tuning by name.** The interviewer wants to see whether you diagnose before you prescribe, and whether you can decline a request without sounding obstructive.

    **Use two axes, not a ranked list.** Sort every complaint into one of two buckets before you propose anything:

    - **Missing knowledge.** The model does not know something: it was never in the training data, it has gone out of date, or it is proprietary to this customer. Fixing this raises **answer accuracy**, and retrieval is the primary tool.
    - **Wrong behaviour.** The model knows enough but does the wrong thing with it: inconsistent output, broken formatting, the wrong tone, or a procedure it keeps skipping. Fixing this raises **consistency**, and prompting, decomposition into a workflow and (rarely) fine-tuning are the tools.

    They **stack**. Start with prompt engineering, then layer retrieval and/or fine-tuning. Most production systems end up with prompting plus retrieval and no fine-tuning at all.

    | Symptom the customer reports | Axis | First move |
    |---|---|---|
    | "It doesn't know our products" | Context | Retrieval over the product corpus |
    | "It used last year's pricing" | Context | Retrieval plus freshness metadata |
    | "The JSON is malformed a third of the time" | Behavior | Structured output, schema validation, few-shot |
    | "It doesn't sound like us" | Behavior | Prompt with style exemplars; fine-tune only if this persists at scale |
    | "It skips step 3 of our procedure" | Behavior | Decompose into a workflow with explicit steps |
    | "It's too slow and too expensive" | Neither | Smaller model, fewer output tokens, caching |

    **Do not reach for an agent either.** For many applications, optimizing single model calls with retrieval and in-context examples is enough. Workflows orchestrate models and tools through predefined code paths and give predictability and consistency for well-defined tasks; agents direct their own processes and suit open-ended problems where the number of steps cannot be predicted. Agentic systems often trade latency and cost for better task performance.

    ```python
    def recommend_approach(symptom: dict) -> list[str]:
        """Ordered interventions. Cheapest, most reversible first."""
        plan = ["prompt: clarify task, add 3-5 in-context examples, constrain output schema"]
        if symptom["missing_domain_knowledge"] or symptom["knowledge_is_volatile"]:
            plan.append("retrieval: hybrid index over the source of truth, cite sources")
        if symptom["format_or_procedure_drift_after_prompting"]:
            plan.append("workflow: decompose into steps with code-enforced checks")
        if symptom["style_still_wrong"] and symptom["examples_available"] >= 1000:
            plan.append("fine-tune: only with a held-out eval and a retraining owner")
        if symptom["latency_or_cost_over_budget"]:
            plan.append("model: try the smaller tier against the same eval set")
        return plan
    ```

    **The honest case against fine-tuning in a customer deployment** (say this out loud, it scores):

    - It needs labelled data the customer usually does not have, which is the same blocker as building evals.
    - It creates an artifact somebody must **retrain** when the base model version changes or the business rules move. Name that owner or do not ship it.
    - It raises data rights questions: whose data trained it, where the weights live, what happens at contract end.
    - It is the hardest change to reverse. Prompts and indexes roll back in minutes.

    **How to say no to the executive:** "Fine-tuning fixes how the model behaves; what you are describing is what the model knows. We can test both cheaply, so let me run your top 50 cases through a retrieval build this week and show you the numbers before we commit to a training pipeline."

    **The fifth option candidates forget: change nothing about the model and change the task.** Frequently the right move is to narrow scope. A system that answers three question types extremely well and routes the rest to a human beats one that attempts everything at 70%. Narrowing is also the fastest path to the adoption number the customer will actually be judged on.

    !!! tip "Interviewer's Insight"
        **Strong answer covers:** diagnosis by symptom; the two axes; the stacking order; the operational cost of owning a fine-tuned model in someone else's environment; a customer-facing sentence, not just an engineering answer.

        **Red flags:** treating the four options as a ladder of sophistication; recommending fine-tuning before an eval set exists; never mentioning the smaller and cheaper model as a legitimate answer.

---

### Controlling Latency and Cost Against a Customer's Budget - OpenAI, Google Cloud Interview Question

**Difficulty:** 🟢 Easy | **Tags:** `Latency`, `Cost Optimization`, `LLM Serving` | **Asked by:** OpenAI, Google Cloud, ElevenLabs

??? success "View Answer"

    **The counter-intuitive fact that separates practitioners from readers of blog posts:** output tokens dominate latency, input tokens barely matter. Cutting 50% of your output tokens may cut roughly **50% of your latency**, whereas cutting 50% of your prompt may only yield a **1 to 5%** improvement. Candidates who open with "we'll shorten the system prompt" are optimising the wrong end.

    **The seven levers, in the order worth trying:**

    1. **Process tokens faster:** smaller model tier, predicted outputs.
    2. **Generate fewer tokens:** cap max_tokens, ask for structured output rather than prose, forbid restating the question.
    3. **Use fewer input tokens:** retrieval instead of stuffing, and caching for the part that repeats.
    4. **Make fewer requests:** combine steps that do not need separate reasoning.
    5. **Parallelize:** independent subtasks, guardrails alongside the main call.
    6. **Make users wait less:** stream, chunk, show progress. Perceived latency is the one the customer complains about.
    7. **Do not default to a model at all:** hardcoding, pre-computation and traditional algorithms are frequently the right answer for a sub-step.

    **Prompt caching economics, precisely.** Cache reads cost **0.1x** the base input price; 5-minute cache writes cost **1.25x** and 1-hour writes **2x**. Default TTL is 5 minutes, refreshed at no additional cost each time the cached content is used. Maximum **4 cache breakpoints** per request with a 20-block lookback. Minimum cacheable prompt length varies by model from **512 to 4,096 tokens**, and shorter prompts silently fail to cache with **no error returned**, which is the single most common reason a team believes caching is on when it is not.

    Invalidation cascades **tools, then system, then messages**, so `cache_control` must sit on the last block whose prefix is identical across requests. Put the volatile user turn after the breakpoint, never before it.

    ```python
    def build_request(system_policy: str, tools: list, user_turn: str):
        return dict(
            model=MODEL,
            tools=tools,                       # stable: goes first
            system=[{
                "type": "text",
                "text": system_policy,         # stable: 6k tokens of customer policy
                "cache_control": {"type": "ephemeral"},   # breakpoint here
            }],
            messages=[{"role": "user", "content": user_turn}],  # volatile: after
            max_tokens=400,                    # the real latency lever
        )

    # A miss pays the 1.25x cache-WRITE multiplier on the cacheable prefix,
    # not 1.0x. Pricing the cold path at 1.0x understates the real bill.
    def cost_per_1k_calls(in_tok, cached_tok, out_tok, p_in, p_out, hit_rate=0.9):
        miss = in_tok + 1.25 * cached_tok        # cache write
        hit  = in_tok + 0.10 * cached_tok        # cache read
        billed_in = (1 - hit_rate) * miss + hit_rate * hit
        return 1000 * (billed_in / 1e6 * p_in + out_tok / 1e6 * p_out)
    ```

    **Worked example.** 500 volatile input tokens, a 6,000 token cached policy prefix, 400 output tokens, at $3 per million in and $15 per million out with a 90% hit rate. A miss bills 500 + 1.25 x 6,000 = 8,000 input tokens; a hit bills 500 + 0.1 x 6,000 = 1,100. Blended that is 1,790 input tokens per call, so **$11.37 per 1,000 calls**, or **$0.0114 per call**. Billing the miss at 1.0x instead would have quoted $10.92, understating the bill by about 4%, and the gap widens fast as the hit rate drops.

    **Work the customer's budget backwards.** If finance says $40k per year for 2 million calls, that is **$0.02 per call**. Compute the per-call token budget from that number and design to it, rather than building first and discovering the bill. Then price the floor you cannot remove: guardrail classifiers, reranking calls, retries and the occasional escalation to a larger model all bill against the same $0.02, and a 3% retry rate on a three-call chain is a real line item rather than a rounding error. Quote the customer a cost per resolved task, never a cost per token, because the per-token number always looks affordable and the per-task number is the one finance renews on.

    **Set a latency budget per stage** the same way you would for any serving system, then hold each stage to it:

    | Stage | Budget (p95) | Lever if it blows the budget |
    |---|---|---|
    | Guardrail classifiers | 120 ms | Run in parallel with retrieval; smaller classifier |
    | Retrieval + rerank | 250 ms | Cap candidate count; cache hot queries |
    | Time to first token | 800 ms | Smaller model tier; cached prefix |
    | Full generation | 3.5 s | Lower max_tokens; structured output instead of prose |
    | Tool round trips | 2 per turn | Consolidate tools; pre-fetch predictable lookups |

    Publish p50 and p95 to the customer weekly. A system that is fast on average and terrible at p95 is the one users quietly abandon, and abandonment is what shows up in the quarterly review, not your token bill.

    !!! tip "Interviewer's Insight"
        **Strong answer covers:** output tokens as the dominant latency term; caching mechanics including the silent minimum-length failure; budget-backwards design; streaming for perceived latency; the willingness to remove the model from a step entirely.

        **Red flags:** "we'll just use a bigger context window"; optimising prompt length for latency; quoting a cost per token instead of a cost per business transaction; ignoring p95.

---

### Hallucinations, Guardrails and Explaining Accuracy Limits to a Non Technical Stakeholder - Sierra, LangChain Interview Question

**Difficulty:** 🟡 Medium | **Tags:** `Guardrails`, `Reliability`, `Stakeholder Communication` | **Asked by:** Sierra, LangChain, Cognition

??? success "View Answer"

    **This question comes in two halves and most candidates only answer the first.** The technical half is the guardrail stack. The customer-facing half, often phrased as "explain to a non-technical VP why your system cannot guarantee 100% accuracy," is where the round is won or lost. Candidate accounts describe Cognition role-playing it with a panel acting as executives, and both LangChain and Sierra running a non-technical presentation as a formal round.

    **The guardrail stack is layered by design.** A single mechanism is unlikely to provide sufficient protection, so run several cheap ones in combination:

    | Layer | Catches | Notes |
    |---|---|---|
    | Relevance classifier | Off-topic and scope creep | Cheap model, runs before the expensive one |
    | Safety classifier | Jailbreaks, prompt injection | Especially important once retrieval pulls in third-party text |
    | PII filter | Sensitive data leaving or entering | Often regex plus a classifier |
    | Moderation | Harmful content | Vendor-provided |
    | Tool safeguards | Dangerous actions | Risk-rated per tool, see below |
    | Rules-based | Blocklists, input length limits, regex | Deterministic, zero latency variance |
    | Output validation | Schema, citations, forbidden claims | Runs after generation |

    The motivation for screening with a cheap fast model before the expensive one runs is both cost and latency. Agent frameworks generally expose this as a tripwire: a guardrail that fires cancels the run and raises rather than returning a value. Know the trade-off in how you schedule them. Running input guardrails in parallel with the main call is better for latency but means the agent may already have burned tokens and executed a tool before the tripwire fires, while running them as a blocking pre-check costs a round trip and saves the spend. Output guardrails necessarily run after generation, so budget for the case where you generate an answer and then refuse to send it.

    ```python
    class Refusal(Exception): ...

    def answer(query, user, client):
        # Cheap deterministic checks first, zero model spend
        if len(query) > 4000:
            raise Refusal("input_too_long")
        if blocklist.hits(query):
            raise Refusal("blocked_term")

        # Parallel cheap classifiers, then the expensive call
        rel, safe = run_parallel(relevance_clf(query), jailbreak_clf(query))
        if not rel:  raise Refusal("out_of_scope")
        if not safe: raise Refusal("suspected_injection")

        docs = retrieve(query, acl=user.attributes)
        if not docs:
            return "I don't have a source for that. Routing you to a specialist."

        out = client.messages.create(model=MODEL, max_tokens=500,
                                     messages=grounded_prompt(query, docs))
        text = out.content[0].text

        # Output validation: every factual claim must carry a retrieved citation
        if not every_claim_cited(text, docs):
            log_metric("ungrounded_answer", 1)
            return "I can't verify that from your documents. Here are the sources I found."
        return text
    ```

    **Three structural moves that reduce hallucination more than any prompt wording:**

    - **Ground and cite.** Force answers to quote retrieved passages and refuse when retrieval returns nothing. "I don't know" must be a first-class, rewarded output in your evals, not an embarrassment.
    - **Move hard constraints into code.** Any rule the business genuinely cannot violate is checked by deterministic code *after* the model produces its answer, not requested of the model in the prompt. Refund ceilings, credit limits, eligibility rules and regulatory thresholds are assertions, and an assertion that fails should block the response rather than log a warning.
    - **Escalate rather than guess.** Two triggers deserve a human: **exceeding failure thresholds** and **high-risk actions** such as cancelling orders, issuing large refunds or moving money.

    **Now the executive conversation.** Do not open with temperature, sampling or probability distributions. Open by changing the question from "is it accurate?" to "what happens when it is wrong?"

    > "Your best analyst is not right 100% of the time either, which is why you have a review step. This system will be right on the large majority of cases in the categories we tested, and we can show you those numbers per category. What I can guarantee is the behaviour when it is unsure: it cites its source, it says 'I don't know' rather than inventing an answer, and anything above your refund threshold goes to a person before it executes. Let's agree an error budget per task type, and I'll report against it every week."

    That answer holds the line, avoids over-promising, and gives the stakeholder a decision to make. Over-promising in a customer simulation is a documented failure mode; so is retreating into jargon.

    !!! tip "Interviewer's Insight"
        **Strong answer covers:** layered guardrails with cheap checks first; parallel vs blocking trade-off; grounding, citation and a rewarded refusal path; deterministic enforcement of hard constraints; explicit escalation triggers; an executive framing built on error budgets and failure behaviour.

        **Red flags:** promising 100% accuracy or "we'll prompt it not to hallucinate"; a single moderation call presented as the whole defence; treating prompt injection as theoretical once you are retrieving third-party documents; talking to a VP about token probabilities.

---

### Designing Agent Tools and Handling Agent Failure Modes in a Customer System - Anthropic, Cognition Interview Question

**Difficulty:** 🔴 Hard | **Tags:** `Agents`, `Tool Design`, `Reliability` | **Asked by:** Anthropic, Cognition, Sierra

??? success "View Answer"

    **First, justify the agent.** Workflows orchestrate models and tools through predefined code paths and offer predictability and consistency for well-defined tasks. Agents dynamically direct their own processes and are the better option when flexibility and model-driven decision making are needed at scale, particularly for open-ended problems where you cannot predict the required number of steps. Agentic systems often trade latency and cost for better task performance. Named patterns worth reaching for before full autonomy: **prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer**. If the customer's process has a fixed sequence, ship the workflow and keep the budget.

    **The agent-computer interface is where deployments actually fail.** Concrete, testable rules:

    - **Consolidate tools around tasks, not endpoints.** Instead of `list_users`, `list_events` and `create_event`, implement a single `schedule_event` tool. Every extra hop is a chance to lose the thread.
    - **Cap tool output.** Anthropic's [Writing Tools for AI Agents](https://www.anthropic.com/engineering/writing-tools-for-agents) notes that Claude Code restricts tool responses to **25,000 tokens by default**. An uncapped database tool returning 40,000 rows destroys the context and the run.
    - **Expose a `response_format` enum** with "concise" and "detailed" so the model can ask for less.
    - **Namespace by service and resource:** `asana_search` vs `jira_search`, `asana_projects_search` vs `asana_users_search`. Ambiguity in names shows up as wrong tool selection.
    - **Prompt-engineer your error responses.** Errors should clearly communicate specific and actionable improvements, not opaque codes or tracebacks. `Error: unknown field 'cust_id'. Valid fields: customer_id, customer_name.` recovers; `500 Internal Server Error` loops.
    - **Leave thinking room.** Give the model enough tokens to reason before it writes itself into a corner.

    One more rule this page would add on top of that list, from deployment experience rather than from any vendor's guidance: **design the arguments so the wrong call cannot be expressed.** Prefer absolute identifiers over relative ones, closed enums over free strings, and a single required idempotency key over an optional one. Every ambiguity you leave in a schema becomes a class of production incident that no amount of prompt wording reliably prevents.

    ```python
    SCHEDULE_EVENT = {
        "name": "calendar_schedule_event",
        "description": (
            "Find a free slot and create a calendar event in one call. "
            "Use this instead of listing users or availability separately. "
            "Returns the created event id and the chosen slot."
        ),
        "input_schema": {
            "type": "object",
            "properties": {
                "attendee_emails": {"type": "array", "items": {"type": "string"}},
                "duration_minutes": {"type": "integer", "minimum": 15, "maximum": 240},
                "earliest_start_iso": {"type": "string",
                    "description": "Absolute ISO-8601 UTC timestamp, e.g. 2026-08-03T09:00:00Z"},
                "response_format": {"type": "string", "enum": ["concise", "detailed"],
                                    "default": "concise"},
                "idempotency_key": {"type": "string",
                    "description": "Caller-generated; retries with the same key never double-book."},
            },
            "required": ["attendee_emails", "duration_minutes",
                         "earliest_start_iso", "idempotency_key"],
        },
    }

    RISK = {"calendar_schedule_event": "medium",   # write, reversible, no money
            "crm_read_account":        "low",      # read-only
            "billing_issue_refund":    "high"}     # write, irreversible, financial

    def dispatch(name, args, ctx):
        if RISK[name] == "high" or ctx.consecutive_failures >= 3:
            return escalate_to_human(name, args, ctx)
        try:
            return truncate(TOOLS[name](**args), max_tokens=25_000)
        except ValidationError as e:
            return {"error": f"{e.field} invalid. Expected {e.expected}. "
                             f"Valid values: {e.allowed[:10]}"}
    ```

    **Rate every tool before you expose it.** Score low, medium or high on read-only vs write access, reversibility, required account permissions and financial impact, then use the rating to decide whether the call pauses for a guardrail check or escalates to a human. This is the single most useful artifact to hand a customer's security reviewer, and it converts an abstract "is the agent safe?" argument into a table they can approve line by line.

    **Failure modes to name, with the mitigation attached:**

    | Failure | Signature in traces | Mitigation |
    |---|---|---|
    | Tool loop | Same tool, same args, 5+ times | Step budget, loop detector, escalate |
    | Context exhaustion | Quality collapses mid-run | Output caps, summarisation checkpoints, sub-agents |
    | Partial write | Step 3 of 5 succeeded, run died | Idempotency keys, compensating actions, resumable state |
    | Hallucinated argument | Field not in schema | Strict schema validation, actionable error text |
    | Silent wrong result | Passes, output is incorrect | Trajectory evals, not just final-answer evals |
    | Injected instruction from data | Agent follows text in a retrieved doc | Treat all tool output as untrusted, never as instructions |

    **Evaluate on trajectories, not answers.** Build eval tasks that require multiple tool calls, potentially dozens, and score the sequence of actions against the one your customer's domain experts wrote down. An agent that reaches the right answer through an unauthorised path has failed.

    **Then hand it over.** The deliverables in an agent engagement are integration surfaces the customer keeps: tool servers, sub-agent definitions, reusable skills, the eval suite and the per-tool risk table. Because the customer versions and extends them after you leave, they need README-quality tool descriptions, tests and a runbook rather than just working code. Insist on sandboxed testing behind the full guardrail stack before any of it touches production, and write down which tools were exercised in that sandbox and which were not.

    !!! tip "Interviewer's Insight"
        **Strong answer covers:** justifying agent over workflow; task-shaped consolidated tools; output caps and namespacing; errors written for a model to act on; per-tool risk ratings driving escalation; idempotency for partial failure; trajectory-level evals; treating tool output as untrusted input.

        **Red flags:** wrapping every REST endpoint as a tool; no step budget or spend cap; no story for a half-completed multi-step action; evaluating only the final answer; assuming the customer's security team will accept "the model decides" as an access-control model.

---

## Customer Craft and Scoping

### How Would You Run a Discovery Session With a New Enterprise Customer? - Anthropic, OpenAI Interview Question

**Difficulty:** 🟡 Medium | **Tags:** `Discovery`, `Customer Facing`, `Scoping` | **Asked by:** Anthropic, OpenAI, Palantir, Databricks

??? success "View Answer"

    **What the interviewer is actually probing:** whether you can hold off on designing. These loops commonly run discovery as a role play, with the interviewer playing a senior customer stakeholder who has a budget and a vague idea. Picture the failure mode: a candidate starts sketching architecture in the first minute, and the interviewer redirects by asking what they would ask the customer first. That redirect **is** the round, so treat every architecture instinct in the opening ten minutes as a trap.

    **Why the discovery gap exists.** Expect the customer's description of their systems and data to differ from what you actually find, and budget time for the difference. The model is usually the cleanest part of the job. The hard part is finding the workflow nobody documented, the data source people actually trust, and the person who knows why the process works the way it does.

    **Run three tracks in the same session, not sequentially:**

    | Track | What you are extracting | You are done when you can state |
    | --- | --- | --- |
    | Workflow | Who does the task today, step by step, and where it hurts | The current process in 5 to 7 steps with a time or error cost on each |
    | Data and systems | Which sources exist, which are trusted, who grants access | The one system of record and the name of the person who owns it |
    | Decision rights | Who sponsors it, who can kill it, who signs off on "good" | The executive sponsor and the person who defines success |

    **The question set that earns its place in the hour:**

    - "Walk me through the last time you did this task. Not the ideal version, the actual one."
    - "What happens today when it goes wrong? Who finds out, and how?"
    - "Of the systems you just named, which one do people actually trust when the numbers disagree?"
    - "If this worked perfectly, what number on your dashboard moves, and by how much?"
    - "Who has tried to fix this before and what stopped them?" (this surfaces political landmines faster than any architecture question)
    - "Who needs to approve production access, and what has that taken historically?" (security review is the most commonly underestimated path to production)
    - "What is the cost of a wrong answer here, and who eats it?" (this determines your guardrail budget later)

    **Anchor to the executive, then go down.** Nabeel Qureshi's account of his Airbus engagement is the canonical example: the problem the company's leadership named was scaling A350 manufacturing, and the software that got built went directly at that problem rather than at an adjacent one that was easier to specify. The lesson he draws is the one to internalise: how fast you become effective correlates with how fast you learn to speak the customer's language and understand how their business actually works.

    **Leave the room with an artifact, not a feeling.** Write the brief live on the call and read it back:

    ```yaml
    engagement_brief:
      sponsor: "VP Payments Ops"          # can fund and unblock
      user: "12 fraud analysts, NY + Dublin"
      workflow: "manual alert triage, ~40 alerts/analyst/day"
      pain: "18 min median per alert, 60% closed as false positive"
      baseline: "measured this week from the case system, not estimated"
      target: "median triage < 8 min with no drop in true-positive catch rate"
      data:
        system_of_record: "case management DB (Postgres, on-prem)"
        trusted_by_users: true
        access_owner: "Data Governance, security review ~3 weeks"
      hard_constraints: ["PII cannot leave the VPC", "audit log for every action"]
      out_of_scope_v1: ["model retraining", "mobile UI", "the other 2 regions"]
      kill_trigger: "no baseline measurable by week 2"
    ```

    **Strong answer covers:**

    - Asking for the **baseline before the target**. If nobody can tell you the current number, you cannot prove impact later and the pilot will die politically even if it works.
    - Naming the **out of scope** list out loud in the room. Scoping is subtraction.
    - Talking to an actual end user, not just the sponsor. Sponsors describe the process they believe exists.
    - Prototyping fast to force clarity. People are far better at rejecting something concrete than at specifying something abstract, so get a rough version in front of them within days and ask whether this is what they meant.

    **Red flags:** proposing RAG, agents or a vector database before you know what the workflow is; accepting "we want AI in our support flow" as a requirement; never asking who owns access; treating discovery as a one time meeting rather than something you re-run every time the data contradicts the story.

---

### A VP Says "We Are Delaying Too Many Flights, Fix It." Turn That Into an Engineering Plan - Palantir, Databricks Interview Question

**Difficulty:** 🟡 Medium | **Tags:** `Scoping`, `Product Sense`, `Decomposition` | **Asked by:** Palantir, Databricks, OpenAI

??? success "View Answer"

    **Where this comes from:** "Why are we delaying so many flights?" is printed in Palantir's own FDSE job description alongside "How can we better identify instances of money laundering?" and "How do we predict and mitigate wildfire risks to optimize power grids?" Candidates describe a Databricks Decomposition round asking the same shape: turn a vague operational goal into a measurable outcome for an end user.

    **This is not a FAANG system design round.** It runs at a much higher altitude, and the common failure is a candidate who stays abstract, never gets into any technical specifics, and never commits to a build. The reported arc ends on **business outcome**: P0 features, technical decisions, data models, then how you would analyse whether the feature is working and what you would present to executives. Candidates who reach for load balancers and caching in minute three have already missed it.

    **The translation ladder. Say each rung out loud:**

    ```
    1. Executive ask       "We delay too many flights."
    2. Decision            Which flight, right now, needs an intervention,
                           and which intervention?
    3. User + moment       The ops controller at the hub desk, 90 minutes
                           before scheduled departure.
    4. Measurable outcome  Reduce departure delays > 15 min at the top 3
                           hubs, measured against last quarter's baseline.
    5. Minimum system      One ranked worklist of at-risk flights with the
                           driving cause attached.
    6. First slice         One hub, one delay cause, read-only, 3 weeks.
    ```

    **Scope the P0 by asking what the user does differently.** If the answer to "what would you do with this screen?" is "look at it," you have built a dashboard, not a workflow. The framing of the day job that these loops reward is exactly this: designing workflows so non-technical users can act on high-noise data, not producing charts.

    **Decompose the causes before the architecture.** Delays are not one problem: inbound aircraft late, crew out of legal hours, gate conflict, ground handling, weather, maintenance hold. Each has a different data owner and a different intervention. Pick the one with the highest volume **and** an intervention the user actually controls. A cause you can predict but cannot act on is worth zero.

    **Then, and only then, the technical decisions:**

    | Decision | Options | What decides it |
    | --- | --- | --- |
    | Ingest | Batch nightly vs streaming | Does the controller act 90 min out or 10 min out? |
    | Joins | Flight, crew, gate, maintenance | Entity resolution across systems with no shared key |
    | Model | Rules first, then learned ranking | Rules ship in week 1 and give you the eval baseline |
    | Delivery | Worklist app vs alert into the existing tool | Never make users open a new tab if you can avoid it |

    **Close with the executive slide, because the round ends there:**

    - **Leading metric:** percentage of at-risk flights flagged 60+ minutes before the delay materialises.
    - **Outcome metric:** delayed departures per 1,000 at the pilot hubs versus the matched baseline period.
    - **Adoption metric:** percentage of flagged flights where a controller took the recommended action. Adoption is the metric that kills projects, not accuracy.
    - **Counter metric:** alert volume per controller per shift. If you triple their alerts to shave a minute, you have made things worse.

    **The grading rubric in disguise:** a solution that works beats a solution that is perfect but hypothetical. Articulate the alternatives and the trade-offs, then be pragmatic enough to land on one concrete approach, deliver a functioning version of it, and expand afterwards. An answer that never converges scores below an answer that converges on something imperfect and says why.

    **Red flags:** never converging on one concrete approach; designing for all six delay causes and all hubs at once; presenting model accuracy to an executive who asked about flights; forgetting to ask what the P0 features are before proposing them.

---

### Why Do AI Proofs of Concept Die, and What Is Your Checklist to Get One Into Production? - OpenAI, Databricks Interview Question

**Difficulty:** 🔴 Hard | **Tags:** `POC to Production`, `Adoption`, `Delivery`, `Evals` | **Asked by:** OpenAI, Databricks, Anthropic

??? success "View Answer"

    **Why this is asked at all.** The forward deployed function is widely described as a market response to a growing AI value gap and widespread proof of concept failure. The frequently cited MIT NANDA figure that roughly 95% of enterprise generative AI pilots produce no measurable P&L impact should be quoted **with attribution and a hedge**, because it is contested and has been argued to be misrepresented. The safer and more defensible framing: traditional software delivery ends at launch, but AI systems are probabilistic and can degrade on production data, so forward deployed engineers are measured by whether the system keeps working and keeps adding value after go live.

    **Most proofs of concept die because nobody ever wrote down what "in production" means.** A demo has an implicit definition of done that everyone in the room agrees with for as long as the demo is running. Production has an explicit one: which users, on which workflow, at what quality bar, monitored by whom, with whose budget paying for it. Agree those five in writing before the build starts and the PoC has a destination it can actually reach. Leave them implicit and the project ends the way most of them do, with a well-received demo, an enthusiastic email thread, and a quiet reorganisation two quarters later.

    **The five reasons POCs die, and the countermeasure for each:**

    | Cause of death | What it looks like | Countermeasure |
    | --- | --- | --- |
    | No baseline | "It feels faster" | Measure the current process before you build anything |
    | No owner after handoff | Champion moves teams, system rots | Named runbook owner and an on-call rota in the SOW |
    | Demo data, not real data | Works on the curated 200 docs | Build on production-shaped data from week 1 |
    | Security review discovered late | Blocked in month 3 | Start the review in week 1, in parallel with the build |
    | No evals | Nobody can say if it regressed | Golden set plus per-intent evals before development is "done" |

    **Eval driven delivery is the defining practice.** Build the evaluation set **with the customer's domain experts before significant development begins**. In a deep debugging workflow that means writing down the ordered sequence of actions a human engineer performs to take a failure from symptom to resolution, and treating that sequence as the label. Development is not complete until the evals verify efficacy. Start with evaluations rather than adding them once the build feels finished. The second-order benefit is political rather than technical: a customer who helped author the eval set has already agreed what good looks like, so the go-live conversation is a reading of a number rather than a negotiation about confidence.

    **The production checklist, in the order you should actually work it:**

    ```
    WEEK 0   Baseline measured from the customer's own system, signed off by the sponsor
             Kill triggers and exit criteria written down before any code
             Security review and data access request submitted (long pole, start now)

    WEEK 1-2 Golden eval set built with domain experts (50-200 labelled items minimum)
             Deterministic constraints separated from probabilistic reasoning
             Thin end-to-end slice on real data, ugly but complete

    WEEK 3-6 Guardrail stack layered: relevance, safety, PII, tool risk rating,
             output validation. No single guardrail is sufficient on its own
             Human-in-the-loop escalation on failure thresholds and high-risk actions
             Observability: traces, per-step latency, cost per task, failure taxonomy

    PRE-GA   Regression suite wired into CI, run on every prompt/model/tool change
             Rollback path and a kill switch the customer can pull without you
             Runbook, on-call owner, and named internal champions trained
             Pre/post measurement plan agreed with the sponsor
    ```

    **Ship one workflow end to end rather than five workflows halfway.** The checklist above only works against a scope small enough to finish. A PoC covering one intent for one team, in production, with a measured before-and-after, buys the mandate for the next five. A PoC covering five intents at demo quality buys nothing, because there is no single number the sponsor can take to their own leadership. When a customer asks for breadth in the pilot, that is the moment to trade scope for a date rather than to accept both.

    **Design the engagement to end.** The practitioner guidance collected in Insight Partners' write-up on the role points the same way: guard the front door with entry criteria, exit criteria and kill triggers agreed before the build starts, and design every engagement to end, because once the outcome is proven, ownership transfers to the teams who productize and run it at scale. Rajkumar Irudayaraj, quoted in that write-up, states the purpose of the team: "The FDE should be a permanent learning loop for the company, not a permanent crutch." The FIS and Anthropic collaboration announced in May 2026, which brings agentic AI to financial-crime investigations, is publicly framed the same way: build the capability with the bank, then leave it running inside the bank's own systems.

    **Red flags:** treating go live as the finish line; no eval story ("we would monitor it" is the reported weak answer to the differentiating AI-lab question, "How do you know your AI system is actually working?"); no handoff plan; no kill trigger, which means the engagement can only end in success theatre or silence.

---

### The Customer Is Waiting and You Do Not Have the Answer. What Do You Say? - Palantir, Sierra Interview Question

**Difficulty:** 🟢 Easy | **Tags:** `Communication`, `Stakeholder Management`, `Behavioral` | **Asked by:** Palantir, Sierra, Databricks

??? success "View Answer"

    **This is asked almost verbatim.** Deployment-facing loops ask how you handle a situation where you do not have the answer and the client is waiting. Values rounds ask you to describe pushing back on a customer to protect the outcome. Simulation prompts run the same test with a clock attached: the deployment slipped three weeks and the customer's CTO is on the line, tell them.

    **What is being probed:** can you hold the line without burning the relationship, and do you default to honesty under time pressure. The trait these teams describe wanting is low ego combined with an unwillingness to accept a surface-level answer: you keep digging until you actually understand the failure, and you care more about the outcome than about who is seen to have produced it.

    **The four move sequence for "I don't know":**

    1. **Say it in one sentence, without hedging.** "I don't know yet." Filling the silence with a guess is the failure mode. A guess that turns out wrong costs you the next six months of credibility.
    2. **Say what you do know and how confident you are.** "What I can tell you is the ingest job failed on the 04:00 run, it affected the EMEA partition only, and no downstream numbers were published from it."
    3. **Commit to a time, not an answer.** "I will have the root cause or a clear status by 3pm your time." Then hit that time even if the update is "still working, here is what I have ruled out."
    4. **Give them something they can use now.** A workaround, a manual path, a scope of who is affected. Ramp's engineering blog illustrates the instinct well: a customer in onboarding was blocked by a feature gap the team estimated at about three days of engineering work, and rather than queue the build and leave the customer stalled, they got on a call and found a path around it that day. The habit to copy is that an unblocking answer beats a complete answer whenever the customer's clock is the binding constraint.

    **Pushing back is a different skill from apologising. The pattern is: agree with the goal, disagree with the method.**

    | Customer request | Weak response | Strong response |
    | --- | --- | --- |
    | "Just give the agent write access to production" | "Sure, we can do that" | "We both want it to resolve tickets end to end. Write access to prod is a high-risk tool by any rating, so let us gate it: propose the change, human approves, then execute, and we lift the gate once the eval suite holds for two weeks." |
    | "Can we skip the eval set and ship?" | "It's not best practice" | "We can ship, and here is the specific risk we take on: no way to detect a regression when the prompt or model changes. Give me three days to label 100 cases with your two senior analysts and you get a release gate you own." |
    | "Add this feature or we churn" | Escalate silently to sales | "Let me make sure I understand the problem behind the request." Then take the commercial conversation to the account team, in writing, same day. |

    **Know where your boundary sits with the account team.** Forward deployed job descriptions name the account team explicitly: you collaborate with sales, solutions engineering, solutions architects and customer success on the same account, you may be pulled in before the deal closes to qualify an engagement and scope the work, and you are expected to represent the customer's reality in internal planning. None of that makes you the commercial owner.

    | Question on the call | Who owns the answer | Your move in the moment |
    | --- | --- | --- |
    | "Will this work on our data?" | You | Answer it, with the caveat and the test you would run |
    | "When will it be live?" | You | Give a date you can defend, or give a date for the date |
    | "What will this cost us?" | Account team | "Let me get you a proper answer today" |
    | "Can you commit to that in the contract?" | Account team | Never commit live. Route it, in writing |
    | "Is this feature on the roadmap?" | Product, via you | Report what you know is public; do not speculate |

    Practically: **you own the technical truth and the delivery commitment; you do not own pricing, contract scope or renewal.** The job family points the same way: FDE postings sit in engineering families rather than in quota-attached sales orgs, though how your own package is structured is a question for your recruiter rather than an industry constant. Either way, never negotiate commercials on a technical call, and never let a technical concession become a contractual one by accident.

    **The written follow-up is the part candidates forget.** Anything you said under pressure should exist in a two paragraph note within a few hours, addressed to the customer with the account team copied:

    ```
    Subject: EMEA ingest failure, 27 Jul, status

    What happened:  The 04:00 sync failed on the EMEA partition. No
                    downstream tables published from that run, so no
                    incorrect numbers reached the analyst worklist.
    Impact:         12 analysts saw stale data between 06:00 and 10:40.
    Where we are:   Root cause is a schema change on the source table
                    (two new columns, one renamed). Confirmed, not suspected.
    Fix:            Backfill running now, complete by 18:00 UTC. Schema
                    check added to the pipeline so this fails fast next time.
    What I need:    30 minutes with your DBA on change notification, so we
                    hear about source schema changes before the job does.
    Impact on date: None. Friday's go-live is unchanged.
    ```

    Three properties make that note work: it separates confirmed from suspected, it states whether bad data reached users, and it answers the question the sponsor actually has (does the date move) without being asked.

    **Say "I" not "we."** Describing owned work as "we" is a general anti-signal across FDE loops, because the evaluator cannot tell what you personally decided. The role is judged on individual ownership.

    **Red flags:** over-promising in the moment to end the discomfort; blaming the customer's infrastructure before investigating, which is the reported weak answer to the production outage scenario; going quiet for two days after bad news; escalating to sales instead of having the hard conversation yourself.

---

### You Have One Week and Four Customers Demanding Different Things. How Do You Prioritize? - Sierra, OpenAI Interview Question

**Difficulty:** 🟡 Medium | **Tags:** `Prioritization`, `Delivery`, `Customer Facing` | **Asked by:** Sierra, OpenAI, Palantir

??? success "View Answer"

    **Asked directly.** Expect a version of this with the constraints made explicit: one engineer, one week, several enterprise customers wanting different things, plus a production issue and sales pressure arriving in the same inbox. Forward deployed job descriptions name the underlying behaviour as a hiring criterion: scope the work, sequence the delivery, remove blockers early, and make explicit trade-offs between scope, speed and quality to protect the delivery date.

    **First move: separate the four buckets, because they are not comparable.**

    | Bucket | Example | Rule |
    | --- | --- | --- |
    | Production down | Customer A's nightly pipeline failing | Always wins. Stabilise first, root cause second |
    | Committed milestone | Customer B's go-live Friday, sponsor is presenting it | Protect the date by cutting scope, not by cutting quality |
    | Unblock | Customer C blocked on a 30 minute config answer | Do it now. Cheapest possible unit of goodwill |
    | Net new build | Customer D wants a new feature | Queue it, scope it, and be explicit about when |

    A frequent mistake is treating the loudest as the most urgent. Loudness correlates with the customer's internal politics, not with impact.

    **Then score what is left. Say the rubric out loud rather than asserting a ranking:**

    ```python
    def priority(item) -> float:
        """Higher wins. Every input is something you can defend to a customer."""
        impact      = item.users_affected * item.value_per_user   # measured, not guessed
        urgency     = 1.0 if item.has_hard_external_date else 0.4
        reuse       = 1.6 if item.generalizes_to_other_accounts else 1.0
        risk        = 1.5 if item.blocks_a_go_live else 1.0
        cost        = max(item.eng_days, 0.5)
        return (impact * urgency * reuse * risk) / cost

    # Ties are broken by: does this remove a dependency on me?
    # Work that makes the customer self-sufficient outranks work that doesn't.
    ```

    **The reuse multiplier is the FDE-specific term.** A three day build that lands for one account is worth less than a three day build that lands for four. Reuse compounds across engagements: the first build for a new problem is mostly bespoke, and the share you can carry forward grows once you have seen the same shape two or three times. If two items score close, ship the one that generalises.

    **Communicate the trade-off before you make it, not after.** The move that keeps all four relationships intact:

    - Tell each customer where they sit **and why**, in one paragraph, in writing. Customers tolerate being second. They do not tolerate finding out they were second.
    - Offer the degraded option instead of the delay: "I cannot deliver the full reconciliation view Friday. I can deliver read-only for the top two regions Friday, and the rest the following week."
    - Loop the account team in the same message. They are the ones fielding the escalation you just created, and finding out about it from the customer rather than from you is how that relationship degrades.
    - Re-scope rather than slip where you can. Ramp's engineering blog reduces the discipline to two words worth quoting: "always be scoping". A week where you cut one deliverable in half and told the sponsor why beats a week where you slipped four things silently.

    **Guard your own capacity honestly.** Travel bands of 25% to 50% are written into Palantir and OpenAI forward deployed postings, and Palantir's Deployment Strategist postings go to "25-75% required." Sustained travel at the top of those bands is the concern practitioners raise most often about the role. An answer that quietly assumes you will absorb everything by working nights is a weaker answer than one that says "here is what I dropped and who I told."

    **Strong answer covers:** an explicit rubric rather than vibes; production incidents pre-empting everything; naming what you will **not** do this week; writing the trade-off down; and looking for the item that removes future work rather than the item that shouts loudest.

---

### A Customer Wants a Feature That Would Fork the Product. What Do You Do? - Palantir, OpenAI Interview Question

**Difficulty:** 🔴 Hard | **Tags:** `Product Sense`, `Bespoke vs Product`, `Architecture` | **Asked by:** Palantir, OpenAI, Anthropic

??? success "View Answer"

    **The word "fork" is the interviewer's trap, and the answer is that a fork is a decision about maintenance, not about the feature.** A fork is cheap on the day you cut it and expensive every day after: two upgrade paths, two security patch queues, and a guarantee that every fix you make upstream never reaches this customer. So the answer is not "yes" or "no" to the feature, it is a walk down the ladder of mechanisms until you find the cheapest one that genuinely holds the requirement, with the fork itself sitting at the bottom as the option you are arguing against. Senior forward deployed postings write the underlying expectation in plainly: turn ambiguous feedback, failures and escalations into durable product requirements and reusable platform capabilities, rather than into an accumulating pile of one-off fixes.

    **Step 1: separate the request from the problem.** "We need a custom scoring rule per business unit" is a request. The problem may be that one business unit's data is labelled differently. Do not architect against a request you have not traced to a workflow. Three questions get you there:

    - "What decision does this change for the person using it?"
    - "What do you do today when the current behaviour is wrong?"
    - "If we solved that underlying problem a different way, would you still want this?"

    A surprising share of fork requests dissolve on the third question, because the customer proposed the only solution they could see from where they sit.

    **Step 2: place it on the ladder. Fork is the last rung, not the first.**

    | Rung | Mechanism | Persistence | When it is right |
    | --- | --- | --- | --- |
    | 1 | Configuration or input preset | Permanent | Behaviour differs but the code path does not |
    | 2 | Permissioning toggle | Years, per request, on user identity | Different customers see different capability |
    | 3 | Ops toggle / kill switch | Long lived, must change without redeploy | Risky capability you may need to disable fast |
    | 4 | Release toggle | Days to weeks, static config | Shipping incrementally toward a common feature |
    | 5 | Extension point or plugin | Permanent | The variation is genuinely per customer logic |
    | 6 | Vertically partitioned deployment | Permanent | Isolation or regulation demands a separate stamp |
    | 7 | Fork | Forever, and it is forever | Almost never. This is the answer to reject |

    Feature flag hygiene matters and interviewers notice it: manage toggle configuration through source control and re-deployment where the nature of the flag allows it, treat toggles as inventory to be paid down with an owner and a removal date, and do not try to test the combinatorial explosion. Test the expected production configuration plus all toggles off. The convention that keeps you safe is that off means existing or legacy behaviour and on means the new behaviour, consistently, on every flag. Ungoverned toggles have a famous price tag: Martin Fowler's feature toggle article points at Knight Capital, which it calls "a $460 million dollar mistake."

    **What rungs 1 and 5 look like in practice.** The shape you are aiming for is one code path with a customer-supplied policy object, not two code paths:

    ```yaml
    # customers/acme.yaml  -- shipped as config, versioned, reviewable
    scoring:
      strategy: weighted_rules        # shared implementation
      weights: {amount: 0.5, velocity: 0.3, geo: 0.2}
      thresholds: {review: 0.6, block: 0.85}
    entity_resolution:
      match_on: [tax_id, normalized_name]
      fuzzy: {enabled: true, max_edit_distance: 2}
    escalation:
      high_risk_tools: [issue_refund, close_account]   # always human-approved
    ```

    ```python
    # The extension point (rung 5) when config is genuinely not enough.
    class ScoringPolicy(Protocol):
        def score(self, case: Case) -> float: ...

    REGISTRY: dict[str, ScoringPolicy] = {
        "weighted_rules": WeightedRules(),   # 90% of customers
        "acme_legacy":    AcmeLegacyScore(), # one class, in the main repo,
                                             # tested by the shared suite
    }
    ```

    The rule of thumb: a customer-specific **class** in the shared repository, covered by the shared test suite, is maintainable. A customer-specific **branch** is not, because nothing you fix upstream ever reaches it.

    **Step 3: if it must be customer specific, make it portable rather than forked.** The property to insist on is that the customer-specific part is a versioned, deployable *artifact* rather than a diff against your source tree: it declares its own dependencies, it carries its own presets, it can be rolled back to the previous version on its own, and installing it does not require modifying the shared code it runs on. Ask yourself whether a colleague could deploy this customer's configuration into a fresh environment tomorrow without reading your commit history. If the answer is no, you have a fork with better branding.

    **Step 4: if the driver is isolation rather than logic, use the tenancy vocabulary.** Isolation is a spectrum, not a binary, and different tiers of one architecture can sit at different points. Single tenant deployments give data isolation, avoid noisy neighbours, and let updates roll out progressively across tenants, which reduces the likelihood of a system-wide outage. The cost line to say out loud: per-tenant infrastructure cost scales close to linearly, so 100 tenants means roughly 100 copies of the bill. The sentence that scores points: **your codebase has to be designed to support both multitenant and single-tenant deployments**, plus a migration path from shared to dedicated and deliberate isolation testing via fault injection rather than assertion.

    **Step 5: solve the customer's problem now, and let generalisation follow.** Do not moralise about technical debt in this round. Refusing a customer-specific need in order to protect an abstraction you have not yet earned is the failure mode on the other side, and interviewers watch for it as closely as they watch for the fork. Generalising too early is the more expensive of the two mistakes, because a framework built on one example fits nobody and still has to be maintained. Solve it specifically, keep the specific thing contained and reviewable, and extract the shared piece once the same need has shown up at a second and third customer.

    **Step 6: close the loop back into the product.** This is what makes the job R&D rather than consulting. The forward deployed engineer is the first line of sight on what the platform is missing, and the discipline is to route that observation into the product rather than hard-wiring it into another bespoke application. Concretely: write down the observed need, how often it has appeared, what your bespoke version does, and what the smallest generalizable primitive would be, then own that document until someone accepts or rejects it. Ankit Sobti, quoted in Insight Partners' write-up on the role, goes further on how central the motion is: "if I were starting a B2B business today, I would think about an FDE motion as my primary engineering motion."

    **Red flags:** agreeing to a fork to save a renewal; refusing all customisation on purity grounds while the customer's problem goes unsolved; not knowing the difference between configuration, a flag, a deployment stamp and a fork; and having no story for how the learning gets back into the product.

---

### The Live Demo Breaks in Front of the Customer's Executives. What Do You Do? - Cognition, LangChain Interview Question

**Difficulty:** 🟢 Easy | **Tags:** `Demos`, `Executive Communication`, `Customer Facing` | **Asked by:** Cognition, LangChain, ElevenLabs, Sierra

??? success "View Answer"

    **Do not treat presentation rounds as fluff.** Publicly shared interview accounts describe Cognition running an executive pitch to a panel role playing company executives plus a timed simulated customer call; LangChain opening with a 20 minute presentation explaining a product such as LangSmith or LangGraph to a **non-technical** audience; Sierra asking candidates to present a technical topic of their choice to a non-technical audience; and OpenAI asking for a recorded video walkthrough of the take-home. The common thread is that FDEs present to customers every day. The bar these rounds are measuring is the ability to move fluidly between a strategic conversation with a senior stakeholder and a hands-on debugging session with their engineers, sometimes within the same hour.

    **Before the demo, buy yourself options:**

    - **Rehearse the exact path**, on the exact machine, on the network you will actually be on. Air-gapped and customer-network demos fail for reasons that never appear in your office.
    - **Record a clean run** of the happy path the night before. A 90 second video is the difference between a recovery and a cancellation.
    - **Seed deterministic data.** Never demo a probabilistic system on data you have not run through at least once. Pin the model version and the prompt.
    - **Have a fallback ladder written down:** live system, then local instance, then recorded run, then screenshots, then whiteboard the flow.
    - **Know the one thing this audience needs to believe.** Executives are not buying the feature. They are buying that the number in the business case can move.

    **When it breaks, the sequence is 30 seconds long:**

    ```
    1. NAME IT           "That call is timing out. Give me twenty seconds."
                         Do not narrate the stack trace. Do not apologise twice.
    2. SWITCH FAST       Move to the recorded run or the local instance.
                         Decide inside 30 seconds; dead air is the real damage.
    3. KEEP THE STORY    "What you were about to see is the analyst going from
                         18 minutes to under 4 on this alert." Continue the
                         narrative on the fallback.
    4. OWN THE CAUSE     Later, in writing: what broke, why, what you changed,
                         and whether it affects the delivery date. Same day.
    ```

    **What executives actually read from a broken demo.** Not "the product is fragile," usually. They read your composure, which forward deployed job descriptions list as a hiring criterion in plain words: stay calm and exercise judgement when the stakes are high. A candidate who debugs live for six silent minutes fails; a candidate who switches to the recording, keeps the room, and sends a root cause note that afternoon often ends the meeting in a stronger position than a clean run would have.

    **Do not fake it.** If the failure is real and material (the retrieval is returning wrong documents rather than a laptop dropping Wi-Fi), say so and reframe the meeting around what you learned. Faking a result that a customer engineer later reproduces is unrecoverable, and it is exactly the behaviour that lets a room dismiss the whole engagement.

    **Demoing to a mixed room, in one pass:**

    | Audience in the room | What they need in the first two minutes |
    | --- | --- |
    | Executive sponsor | The baseline number, the target number, and the date |
    | Line manager | What changes for their team on Monday |
    | Engineer | Where the data comes from and what the system will not do |
    | Security or GRC | Where data lives, who can see it, what is logged |

    Naming the constraint before security asks is one of the highest-leverage moves available, and it is why these postings list governance and security as routine collaborators rather than as an occasional obstacle.

    **Strong answer covers:** the pre-built fallback ladder; deciding inside 30 seconds; keeping the business narrative alive independent of the tooling; honesty about what actually failed; and a same-day written follow-up that separates "laptop problem" from "system problem."

    **Red flags:** silent live debugging; blaming the customer's network before checking; promising a fix date on the spot without knowing the cause; and treating the demo as a technology showcase rather than an argument that a specific number will move.

---

## Behavioral and Situational

### How Should You Structure a Behavioral Answer for a Forward Deployed Engineer Loop? - Palantir, OpenAI Interview Question

**Difficulty:** 🟢 Easy | **Tags:** `Behavioral`, `STAR`, `Communication` | **Asked by:** Palantir, OpenAI, Glean, Cognition

??? success "View Answer"

    **What the interviewer is actually probing:**

    Whether you can be trusted alone in a room with a customer. FDE behavioral questions are not a culture-fit formality; candidates consistently report that at Palantir they are embedded inside the technical rounds, roughly 15 to 20 minutes of each 60-minute round, rather than run as a separate stage. That means your story has to land in two to three minutes, in the middle of a coding or decomposition conversation, without a warm-up.

    **Standard STAR is not enough. Use STAR plus two FDE beats:**

    | Beat | Standard STAR | The FDE version |
    |---|---|---|
    | **S**ituation | Project context | Name the *customer* and their business pressure, not your sprint |
    | **T**ask | What you were assigned | What outcome you were accountable for, and what was undefined |
    | **A**ction | What you did | Decisions **you** made, with the trade-off you rejected |
    | **R**esult | Metric moved | Business metric plus **adoption**: who actually uses it now |
    | **C**arryover | (missing) | What generalized: a pattern, a tool, a product requirement |
    | **D**ebt | (missing) | What you knowingly left broken, and why that was correct |

    The last two beats are where FDE answers separate from generic SWE answers. If your story ends at "we shipped it," you have described a feature rather than a deployment, and the interviewer learns nothing about whether you can be left alone with a customer. The Debt beat is the one candidates omit out of nervousness, and omitting it costs more than admitting it: forward deployed work legitimately produces fast, ugly code under customer pressure, so naming what you knowingly left broken and why that was the right call that week reads as judgement. Pretending the engagement produced only clean code reads as inexperience.

    **A timing budget that survives a compressed round:**

    ```
    0:00-0:20  Situation + Task   (who the customer was, what was at stake, what was undefined)
    0:20-1:30  Action             (2-3 concrete decisions, each with the option you rejected)
    1:30-2:10  Result             (one hard number + one adoption number)
    2:10-2:40  Carryover + Debt   (what generalized, what you left ugly on purpose)
    2:40       Stop. Let them pull the thread.
    ```

    **Say "I", not "we".** Narrating owned work in "we" is a general FDE anti-signal, because the role is judged on individual ownership and the evaluator cannot separate your contribution from your team's. Use "we" only when describing a team decision you then say you disagreed with or executed.

    **Prepare a story matrix, not a story list.** Build six to eight stories and index them against the themes below so you can re-cut one story for several prompts rather than scrambling for a new one:

    - A failure you owned outright
    - A customer relationship you repaired
    - A decision made with missing information
    - A disagreement you lost, and one you won
    - Something you shipped that nobody adopted
    - Something ugly you built fast that turned out to be right
    - A pattern you spotted across two customers and turned into a reusable thing

    **Strong answers cover:**

    - The customer's language, not yours. Describe the problem the way the customer's own people describe it, using their nouns for their systems and their units for their pain. Practitioner accounts of the role consistently make this the strongest predictor of how fast an FDE becomes effective on a new account.
    - Numbers you actually measured, including the negative ones
    - The constraint you refused to trade away (security, compliance, an eval bar)
    - A concrete mechanism change, not a feeling change

    **Red flags interviewers report:**

    - Stories with no customer in them. The blunt version of the hiring bar is whether the interviewer would want you beside them in a difficult customer room, and a story about an internal refactor does not answer that.
    - Success-only narratives. Behavioral rounds in this family deliberately probe failures, mistakes and struggles, and a candidate with no real failure reads as someone who has never owned an outcome.
    - Rehearsed delivery that cannot be interrupted. Interviewers interrupt on purpose.
    - Five-minute Situation sections. If you have not reached an Action by the one-minute mark, you have lost the round.

---

### Tell Me About a Time You Dealt With a Very Unhappy Customer - Sierra, ElevenLabs Interview Question

**Difficulty:** 🟡 Medium | **Tags:** `Behavioral`, `Customer Management`, `Incident Response` | **Asked by:** Sierra, ElevenLabs, OpenAI, Palantir

??? success "View Answer"

    **What the interviewer is actually probing:**

    Two separate skills that candidates blur together: can you stabilise a technical incident, and can you stabilise a human being. Forward deployed postings name the second one directly as a hiring criterion, in the form of staying calm and exercising judgement when the stakes are high. A customer is rarely angry about the bug. They are angry because they defended you internally and now look foolish.

    **Handle the two tracks in parallel, and say so out loud:**

    | Track | Owner | Clock | First move |
    |---|---|---|---|
    | Technical | You and the on-call | Minutes | Restore service or ship a workaround |
    | Relational | You and their sponsor | Hours to days | Give them something to forward to their boss |

    **A workable first-30-minutes ordering:**

    1. **Acknowledge without diagnosing.** "I own this, I am on it, you will hear from me at the top of every hour." Do not theorise about cause on the first call.
    2. **Restore, then explain.** A degraded but working path beats a correct root cause. Kill switches and feature flags exist for exactly this: ops toggles are the category that has to be reconfigurable fast, without a redeploy, at an hour when nobody wants to run a release.
    3. **Investigate your own surface first.** The reported weak answer to the production-outage scenario is blaming the customer's infrastructure before investigating. Even when it turns out to be their proxy, their firewall or their expired credential, you do not get to say so until you have cleared your own side.
    4. **Write the update they can forward.** This is the artifact that repairs the relationship.

    ```
    STATUS UPDATE  -  14:00 local  -  Ingestion pipeline, EU tenant

    IMPACT      Nightly sync incomplete since 02:10. 3 of 11 dashboards stale.
                No data loss. No exposure of restricted markings.
    NOW         Workaround deployed 13:40; the 3 dashboards backfill by 18:00.
    CAUSE       Under investigation. Two hypotheses, both on our side.
    NEXT UPDATE 15:00, from me, even if nothing has changed.
    ASK         We need a DBA on the 15:30 call to confirm supplemental
                logging is still enabled on the source tables.
    ```

    **Then close the loop properly.** A written post-incident note within 48 hours, a mechanism change (a data health check on freshness, an expectation that aborts the build, an alert threshold), and a follow-up two weeks later showing the check firing correctly. The standard data health taxonomy gives you concrete vocabulary here: Status, Time (freshness, build duration, sync duration), Size (row count, partition), Content (primary key uniqueness, null percentage, allowed values) and Schema checks.

    **Strong answers cover:**

    - A specific moment where you absorbed anger without becoming defensive or matching their tone
    - The workaround before the root cause, and why that ordering was right for their business
    - What you told them you did **not** yet know, rather than guessing to fill silence
    - The mechanism, not the apology: what now makes this class of failure visible before the customer sees it
    - The relationship afterwards. These roles are expected to build a long-term relationship and to keep finding the next thing worth deploying across the life of an engagement, so an answer that ends at incident close misses half the job.

    **Red flags interviewers report:**

    - Making the customer the villain of your own story
    - Blaming their infrastructure, their data quality or their security team before you have evidence
    - Over-promising a fix date on the call to make the anger stop, then missing it, which converts a technical problem into a credibility problem
    - "We escalated it to support" as the entire action. FDEs are the escalation path.
    - Narrating in "we" throughout, so the interviewer cannot tell what you personally did

---

### Walk Me Through a Deployment That Failed and What You Learned - Palantir, Anthropic Interview Question

**Difficulty:** 🟡 Medium | **Tags:** `Behavioral`, `Failure`, `Post-Mortem` | **Asked by:** Palantir, Anthropic, OpenAI, Databricks

??? success "View Answer"

    **What the interviewer is actually probing:**

    Calibration. Hiring managers in this family ask directly what you consider your biggest career failure and what you took from it, and the round is explicitly not fishing for a success story dressed as a lesson. A candidate with no real failure has either not owned an outcome or is not being honest.

    **Pick a failure from the right category.** FDE failures cluster into four types, and the more customer-shaped the failure, the better it scores:

    | Failure type | What actually went wrong | Why it scores well |
    |---|---|---|
    | **Scoping** | You built what they asked for, not what they needed | Tests discovery instinct |
    | **Adoption** | The system worked; nobody used it | The single most FDE-specific failure |
    | **Ground truth** | The data source everyone trusted was wrong | Tests investigation over assumption |
    | **Generalization** | Bespoke work that produced nothing reusable | Tests the product feedback loop |

    Avoid: a failure caused by someone else, a failure that resolves into a compliment, and anything where the lesson is "communicate more."

    **The adoption failure is the highest-value story in this role, and it is the hardest one to tell well.** The temptation is to narrate it as a user-education problem, which reads as blaming the customer. Tell it structurally instead: name the assumption you made about how people worked (usually that they would change a habit because your version was better), name the evidence that would have falsified it (a shadowing session, a usage cohort at week four, a single question asked of a sceptic rather than of the sponsor), and name why you did not go looking for that evidence until it was late. An interviewer who has run deployments will recognise the shape immediately, because they have lived it.

    **Structure the answer as a post-mortem, not a confession:**

    ```
    1. THE COMMITMENT   What outcome did you personally sign up for, and by when?
    2. THE SIGNAL       What was the first indication it was going wrong?
                        Be honest about how many weeks passed before you acted on it.
    3. THE DECISION     What you did when you knew. Escalate? De-scope? Restart?
    4. THE COST         Real numbers: weeks lost, users who churned off the workflow,
                        the renewal conversation it made harder.
    5. THE MECHANISM    What is structurally different now, in a way a stranger
                        could verify: an entry criterion, an eval set, a kill trigger,
                        a weekly usage review with the sponsor.
    ```

    **The mechanism beat is what separates senior answers.** Insight Partners' practitioner write-up gives you the vocabulary, in Rajkumar Irudayaraj's words: "Interest is not the same as readiness. Define entry criteria, exit criteria, and kill triggers before the build starts." The article's companion instruction is to design every engagement to end. If your lesson is a process someone else can now run without you, it counts. If it is a personal resolution, it does not.

    **AI-lab loops want the eval version of this story.** Forward deployed postings at these companies require evaluation frameworks by name and describe success in terms of eval-driven feedback rather than delivery milestones. A failure story where you discovered too late that you had no way to tell whether the system was getting better or worse is directly on target, and the mechanism beat writes itself: you now build the eval set with the customer's experts before development rather than after.

    **Also acceptable, and unusually well-regarded: the generalization failure.** Building the abstraction first is a real and common error, and the engagements that produce genuinely reusable insight are usually the ones that went deepest on one customer's specific problem without setting out to generalise. Telling a story where you built the framework first and it fit nobody is a credible senior failure.

    **Red flags interviewers report:**

    - The humblebrag failure ("I cared too much about quality and shipped late")
    - Root cause located entirely outside your control
    - No numbers, so the interviewer cannot judge severity
    - A lesson with no mechanism attached
    - Discovering the failure at the retrospective rather than during the engagement

---

### How Do You Operate With No Requirements and No Authority? - Anthropic, Databricks Interview Question

**Difficulty:** 🔴 Hard | **Tags:** `Behavioral`, `Ambiguity`, `Decision Making`, `Ownership` | **Asked by:** Anthropic, Databricks, Palantir, Google Cloud

??? success "View Answer"

    **What the interviewer is actually probing:**

    The two failure modes that end FDE engagements: the engineer who freezes waiting for a spec, and the engineer who charges ahead and breaks something that mattered. Expect prompts about a time the business problem was not clearly defined, and about a bold decision you made and had to defend. Palantir's new-grad FDE posting names **Agency** as one of its five values, meaning you learn continuously, make decisions with incomplete information and do not wait to be told what to do next, alongside embracing the ambiguity. Candidate accounts describe Databricks running a whole Decomposition round built from a vague business goal.

    This is also the everyday reality of the job. You will work in a great deal of ambiguity, and what the customer described during scoping will routinely fail to match the data and systems you find once you have access.

    **A four-move framework you can narrate under pressure:**

    **1. Reduce ambiguity cheaply before you decide.** Not a two-week discovery phase. A day of the right conversations. The model is usually the cleanest part of the work; the hard part is finding the workflow nobody documented, the data source people actually trust, and the person who knows why the process works the way it does. Find that person first.

    **2. Classify the decision by reversibility before you classify it by difficulty.**

    | Decision class | Example | How to handle it |
    |---|---|---|
    | Reversible, low blast radius | Chunk size, retry policy, dashboard layout | Decide alone, in minutes, note it |
    | Reversible, visible | Workflow shape, which team pilots first | Decide, then tell the sponsor within a day |
    | Irreversible, technical | Ontology and semantic model, tenancy model | Decide with a written options memo |
    | Never yours | Data governance, access markings, compliance posture | Escalate, always, even at cost of the date |

    That last row is what makes "bold" safe. The strong version of this answer names the guardrail you deliberately did not cross. The layered access-control vocabulary is useful here: organizational boundaries that silo groups of users and resources, mandatory data markings that propagate through provenance and lineage, and attribute-based row-level and column-level controls. Nobody is impressed by an FDE who exercised agency over a data marking.

    **3. Ship a concrete first cut, fast.** The line to internalise for open-ended problems is that you need something that works before you have the time to make it perfect. Articulate the alternatives and the trade-offs, then be pragmatic enough to arrive at a concrete approach, deliver a functioning version, and expand it afterwards. Prototypes beat specifications with a customer who cannot articulate requirements, because people reject a concrete thing far more precisely than they describe an abstract one.

    **4. Write the decision down so it is auditable later.** A short decision record turns "he went rogue" into "he made a documented call under time pressure." Keep it to something you can produce in five minutes:

    ```yaml
    decision: Ship read-only advisory agent to 12 pilot users on Friday
    date: 2026-03-04
    owner: me
    decided_because:
      - Sponsor is presenting to their exec committee on the 11th
      - Write access needs a security review we cannot complete in a week
    assumptions:
      - Pilot users tolerate manual copy-paste for the first two weeks
      - The 200-item golden set is representative of their real queries
    not_crossed:
      - No write path to the ERP; no PII leaves the customer VPC
    reversible: yes, feature-flagged, off by default for everyone else
    revisit: 2026-03-25, or earlier if eval pass rate drops below 85%
    ```

    **Authority you do not have, you borrow.** When a decision needs a signature you cannot give, the move is to make the decision easy for whoever can: present two options, the cost of each, your recommendation, and a default that takes effect if you do not hear back by a stated time. That converts an open question into a closable one.

    **Strong answers cover:**

    - A named person you found who held the undocumented knowledge
    - The timebox you put on ambiguity before forcing a decision
    - The option you rejected and why
    - The guardrail you refused to cross under schedule pressure
    - How you socialised the decision so it could be reversed cleanly

    **Red flags interviewers report:**

    - Waiting to be told. Palantir's new-grad FDE copy is deliberately harsh here: "You will not be handed a ticket queue."
    - Escalating every decision upward, which reads as low agency
    - Boldness with no guardrail, which reads as a liability in a regulated customer environment
    - Never landing on a concrete approach. This is the reported failure mode of Palantir's Decomposition round, where candidates enumerate trade-offs forever and never build anything.
    - Deciding silently and leaving no trace, so the customer discovers the decision in production

---

### Tell Me About Pushing Back on a Customer, and on Your Own Product Team - Databricks, OpenAI Interview Question

**Difficulty:** 🔴 Hard | **Tags:** `Behavioral`, `Influence`, `Stakeholder Management` | **Asked by:** Databricks, OpenAI, Sierra, Anthropic

??? success "View Answer"

    **What the interviewer is actually probing:**

    Candidates report being asked this directly in a Databricks values round, in the form of a prompt about pushing back on a customer to protect the outcome. Customer-simulation prompts push the same nerve, for example a customer demanding a feature that compromises data governance. What the interviewer wants to see is that you have a spine and a relationship at the same time, and that you know which disagreements are worth spending credibility on.

    Candidates almost always prepare only the external half. The internal half, disagreeing with your own product or research team, is where senior FDEs are actually tested, because the whole role sits on that seam.

    **These are two different playbooks. Do not use one for the other.**

    | | Pushing back on the customer | Pushing back on your product team |
    |---|---|---|
    | Your leverage | Being right about their business | Evidence across accounts |
    | Opening move | Reframe from solution to outcome | Bring frequency data, not an anecdote |
    | What closes it | A decision made by their sponsor, in writing | The smallest generalizable primitive |
    | Acceptable loss | They overrule you; you document the risk | "Keep it bespoke"; you write down why |
    | Never acceptable | Silently building it anyway | Calling the roadmap team blockers |

    **External: reframe, price, then hold one line.** When a customer demands the wrong thing, argue about the outcome, not the feature. Ask what they will do differently once it exists. Then price two paths honestly, including the one you disagree with, and name the single constraint you will not trade: data governance, an access marking, an eval pass rate, a security review. Everything else is negotiable. Ramp's engineering blog illustrates the productive version of this: a customer in onboarding was blocked by a feature gap the team sized at roughly three days of engineering work, and rather than queue the build they got on a call and worked out a path around it. Pushback and unblocking are the same conversation when done well.

    If they overrule you, you build it and you write down the risk you flagged and the date you flagged it. That record is what makes you credible the next time, and it is the difference between an engineer with judgement and one who sulks.

    **Internal: pushing a customer-specific need into someone else's roadmap is the harder half.** Your leverage here is not that this customer is important, because every account team says that and the roadmap team has heard it from all of them. Your leverage is frequency evidence across accounts, plus a working bespoke version that proves the need is real and shows exactly how small the generalizable piece is. The ask that lands is the smallest primitive, not the feature you already built.

    So when you push a customer-specific need toward the product team, bring the loop, not a request:

    ```
    THE ASK      Native CDC support for Oracle in the connector framework.
    FREQUENCY    3 of my last 5 accounts; 2 blocked at security review because
                 the current workaround needs a broader privilege grant.
    WHAT I BUILT A per-account shim. ~400 lines. Two customers now run it.
    WHAT I LEARNED  The real blocker is not the connector, it is negotiating
                 supplemental logging with the customer DBA. Ship the runbook
                 and the pre-flight check, and the connector matters less.
    SMALLEST FIX A pre-flight validator for source-side prerequisites.
    IF YOU SAY NO   Fine. I will keep the shim in the account repo and re-raise
                 at 5 accounts. Here is the doc so nobody rebuilds it.
    ```

    Senior forward deployed postings encode this expectation as a responsibility: align early on what should generalise, what stays customer-specific and what "ready for handoff" means, then turn ambiguous feedback, failures and escalations into durable product requirements rather than one-off fixes. The shape to describe is that reuse compounds: almost nothing carries forward from a first engagement, and a meaningful share does once the same problem has appeared two or three times. Saying that out loud tells the interviewer you understand what you are feeding.

    **Know the counter-argument too.** Aggressive scoping genuinely does conflict with the instinct to generalise, because good interfaces and platforms are what make software scale in the first place. Saying out loud that this is a judgement call rather than a rule is a maturity signal.

    **Red flags interviewers report:**

    - Every story ends with you winning. Bring one you lost and executed well.
    - Every story ends with you folding. That reads as an order-taker.
    - "I escalated to my manager" as the entire action
    - Framing product or research colleagues as obstacles
    - Treating every customer-specific request as technical debt to be refused, which misreads the role: bespoke depth is the input to the product loop, not a failure of it

---

### Delivering Bad News: A Slipped Date or an Impossible Request - Cognition, Palantir Interview Question

**Difficulty:** 🟡 Medium | **Tags:** `Behavioral`, `Communication`, `Executive Presence` | **Asked by:** Cognition, Palantir, OpenAI, ElevenLabs

??? success "View Answer"

    **What the interviewer is actually probing:**

    Whether you can be put in front of an executive unsupervised. This gets tested with unusual directness in FDE loops. Candidate write-ups on Cognition's Deployed Engineer process describe an executive pitch to a panel role-playing company executives and a timed case-study simulated customer call. Deployment-facing loops ask flat out how you handle a situation where you do not have the answer and the client is waiting. The target these rounds are calibrated against is executive presence: moving fluidly between a strategic conversation with senior stakeholders and a hands-on debugging session with their engineers.

    **Rule one: they hear it from you, early, or your credibility is the thing that slipped.** A date you flag three weeks out is a project management problem. The same date discovered by the customer two days before launch is a trust problem, and trust is the FDE's only real asset.

    **Use bottom line up front. Four beats, in this order:**

    ```
    1. HEADLINE   "The integration will not be ready for the 14th.
                   My new date is the 28th."
                  (No context first. No throat-clearing. One sentence.)

    2. IMPACT     In their terms, not yours. "That means your ops team
                   runs the manual process for two more weeks, and the
                   board demo shows the read-only view, not the live one."

    3. PLAN       What is already true, not what you hope. "The pipeline
                   and access controls are done and tested. What is left
                   is the CDC prerequisite on the Oracle source, which
                   needs a privilege grant from your DBA team."

    4. ASK        Give them something to do. "I need 30 minutes with
                   your DBA lead this week, or I need your call on
                   shipping the batch-load version on the 14th instead."
    ```

    **Always bring the de-scoped option.** The strongest move is to protect the original date with reduced scope, and let the sponsor choose. "You can have the full thing on the 28th, or the read-only version on the 14th" is a decision. "It will be late" is a complaint. Forward deployed job descriptions name this exact skill as a hiring criterion: making explicit trade-offs between scope, speed and quality, and adjusting the plan to protect delivery.

    **Re-forecast once, with padding you can defend.** Slipping twice costs more than slipping once by double. If you cannot commit to a date, commit to a date for the date: "I will know by Thursday whether the 28th holds, and I will tell you Thursday either way."

    **The impossible request is a different conversation.** Sort it before you answer:

    | Class | What it actually means | Your response |
    |---|---|---|
    | Impossible now | Real, just not by that date | Offer scope trade or a workaround |
    | Impossible here | Blocked by their environment, policy or data | Name the blocker and who owns it on their side |
    | Impossible ever | Physics, compliance, or model capability | Say no clearly, then redirect to the outcome |

    The third case is where candidates flinch. A common simulation prompt is explaining to a non-technical VP why a retrieval system cannot guarantee 100% accuracy. The good answer does not apologise for probabilistic systems; it converts the demand into a measurable one. Anthropic's published retrieval results give you concrete language: a baseline top-20 retrieval failure rate of 5.7% drops to 3.7% with contextual embeddings, 2.9% adding contextual BM25, and 1.9% with reranking, a 67% reduction overall. That reframes "is it accurate" into "what failure rate is acceptable for this workflow, how do we measure it, and what do we do on the residual." Pair it with an escalation design: rate each tool by risk on read-only versus write access, reversibility, the account permissions it needs and its financial impact, then route the high-risk actions to a human.

    **Strong answers cover:**

    - How many days before the deadline you told them, and how you knew
    - The de-scoped alternative you brought to the same conversation
    - Who else you told internally, and when
    - A concrete sentence you actually said, ideally an uncomfortable one
    - The follow-up written summary, since the person in the room is rarely the only person who needs it

    **Red flags interviewers report:**

    - Burying the headline under five minutes of context
    - Naming and blaming an internal team or a colleague to the customer
    - Committing to a new date live on the call, without checking, to relieve the discomfort
    - Going quiet as the date approaches, which is the most common real-world version of this failure
    - Saying yes to something impossible and planning to renegotiate later
    - Treating the presentation and simulation rounds as fluff. Candidates report a 20-minute explanation of a product to a non-technical audience at LangChain, a presentation of a self-chosen technical topic to a non-technical audience at Sierra, and a recorded video walkthrough of the take-home at OpenAI, all of it because FDEs present to customers constantly.

---

### Travel, Burnout, and the Questions You Should Ask Your Interviewer - Google Cloud, LangChain Interview Question

**Difficulty:** 🟢 Easy | **Tags:** `Behavioral`, `Role Fit`, `Career` | **Asked by:** Google Cloud, LangChain, Anthropic, Palantir

??? success "View Answer"

    **What the interviewer is actually probing:**

    Attrition risk. Travel is the one constant across every version of this role, and hiring teams have watched people burn out. Ex-Palantir FDE Piotr Kraus, speaking to [LeadDev](https://leaddev.com/hiring/the-rise-of-the-forward-deployed-engineer) about postings that carry up to 50% travel, says the constant travel makes the role prone to burnout earlier than most engineering jobs, and in the same interview he says the title "might have become a little bit frothy." That is one practitioner's read rather than a published attrition rate, but it is the concern hiring teams are screening for. Recruiters ask a soft version of it, for example what you pursue outside work, or why you left your last role.

    **Know the real numbers before you answer:**

    | Role | Stated travel, as written in the posting |
    |---|---|
    | Palantir FDSE | "25-50%" preferred, varies by team and location |
    | Palantir Deployment Strategist | "25-75%" required |
    | OpenAI Forward Deployed Software Engineer | travel up to 50% required |
    | Anduril, Air Defense FDE | "up to two months at a time (up to 80% of the year)" |

    Two things follow. First, hardware and defense-facing forward deployed roles carry substantially heavier travel than software-facing ones, so read the band rather than assuming the 25-50% norm. Second, bands move between postings and between teams inside the same company, so the number written into the specific job description you applied to is the only one that matters; do not argue from a table someone else compiled, including this one.

    Relocation for the length of an engagement is a documented pattern rather than an edge case. Nabeel Qureshi's account of the original Palantir model describes his first engagement as a year spent living in Toulouse and working inside the Airbus A350 factory alongside the manufacturing staff. Read that as the upper bound of what "travel" can mean in this role, and ask early whether any engagement on the team you are joining is structured that way, because a year-long relocation is a different life decision from a two-day-a-week commute.

    **How to answer the travel question without lying or hedging:**

    1. **Give a real number.** "I can sustain three weeks a month onsite for a quarter, then I need a lighter month" is a better answer than "whatever it takes." Interviewers have heard the enthusiastic version fail.
    2. **Show your operating model.** What do you do onsite that you cannot do remotely? Strong candidates separate the two deliberately: discovery, workshops, shadowing operators and trust-building onsite; building, reviews and eval runs remote. The published engagement shapes all follow the same rhythm, a short intense scoping block onsite, then validation, then a build phase with recurring onsite days rather than continuous presence.
    3. **Cite evidence, not intent.** A period where you actually did it, and what you changed the second time.
    4. **Be honest about your hard constraints.** Stating a real limit up front is respected. Discovering it in month two is not.

    **Then turn the round around. The questions you ask are scored.** Ask things that only an insider can answer:

    | Ask | What it de-risks |
    |---|---|
    | "Which exact title am I interviewing for, and how does it differ from the neighbouring one?" | Candidates describe Palantir running FDSE (Delta) and Deployment Strategist (Echo) as different loops; OpenAI posts more than one forward deployed family side by side, including a Gov variant; Anthropic's board listed customer-facing engineering under Applied AI rather than FDE when it was last checked |
    | "What size are the contracts this team supports?" | Embedding engineers per account only pays for itself above some deal value. Below it the team is doing consulting under a product banner, whatever the title says |
    | "What percentage of what this team builds ends up in the product?" | Separates a learning loop from a delivery shop, and tells you whether the productisation path is real or aspirational |
    | "What are the entry and exit criteria for an engagement, and who can kill one?" | Insight Partners' write-up on the role runs a section headed "Design every engagement to end"; an org with no kill trigger will keep you on a dying account |
    | "Am I on the engineering ladder or a go-to-market ladder?" | Ladder placement drives the comp structure, the promotion path and who writes your review, and it is often not inferable from the title. Ask it directly rather than assuming |
    | "What is the on-call and travel cadence in practice last quarter, not on paper?" | The gap between stated and actual load is where burnout lives |
    | "Do FDEs rotate into core engineering?" | Ask for the actual count over the last two years, not whether it is possible in principle. A team with a real rotation path is a team that expects you to still be there in three years; a team that cannot name anyone who moved is telling you the forward deployed org is a terminal assignment |

    **Strong answers cover:**

    - A sustainable number you can defend, plus what you do to sustain it
    - Explicit awareness that onsite time is the point of the role, not overhead
    - Questions that show you know the role has variants, and which one you want
    - Curiosity about the product feedback loop, since that is what separates this from consulting

    **Red flags interviewers report:**

    - "I'll do whatever it takes" with no evidence of ever having done it
    - Treating travel as a negotiation item in the first conversation
    - No questions at the end, or only compensation questions in an early round
    - Asking things published on the careers page
    - Generic enthusiasm about the company. "I want to work on hard problems" reportedly gets filtered at the recruiter screen; be able to name a specific deployment, product or engagement and say why it interests you, and have a real answer to "why FDE rather than a standard software engineering role"

---

## Quick Reference: 100 FDE Interview Questions

| # | Question | Focus Area | Companies Asking | Difficulty |
|---|----------|------------|------------------|------------|
| 1 | What is a Forward Deployed Engineer, and how is the role different from a normal product engineer? | Role and Positioning | Palantir, OpenAI | Easy |
| 2 | A product engineer owns one capability across many customers; an FDE owns one customer across many capabilities. Which side are you applying for, and why? | Role and Positioning | Palantir, Databricks | Easy |
| 3 | Why forward deployed engineering rather than a standard software engineering role? | Role and Positioning | Palantir, OpenAI, Cognition | Easy |
| 4 | How would you explain the difference between an FDE and a Solutions Architect or Sales Engineer to a candidate you were recruiting? | Role and Positioning | Anthropic, OpenAI | Medium |
| 5 | An FDE team can become a permanent crutch for customers instead of a learning loop for the company. How do you keep it from happening? | Role and Positioning | Databricks, Snowflake | Hard |
| 6 | When does the forward deployed model stop making economic sense for a given contract size? | Role and Positioning | Databricks, OpenAI | Hard |
| 7 | Why this company specifically, and which of our deployments or products made you apply? | Role and Positioning | Palantir, Anthropic, ElevenLabs | Easy |
| 8 | Where do you want this role to take you in three years, and what would make you leave it? | Role and Positioning | Cognition, Sierra | Easy |
| 9 | A customer has data in Oracle, S3, an SFTP drop and a Salesforce instance. How do you plan the first integration? | Data Integration | Palantir, Databricks | Medium |
| 10 | Why do enterprise platforms use agent based connections inside the customer network rather than connecting directly to their databases? | Data Integration | Palantir, Snowflake | Medium |
| 11 | The customer's DBA says change data capture on Oracle requires supplemental logging and new privileges. How do you run that conversation? | Data Integration | Palantir, Databricks | Hard |
| 12 | When would you choose change data capture over a scheduled batch sync, and when is CDC the wrong tool? | Data Integration | Databricks, Snowflake | Medium |
| 13 | Two customer systems describe the same customer entity with no shared key. How do you resolve them? | Data Integration | Palantir, Google Cloud | Hard |
| 14 | Design a semantic or ontology layer so that renaming a source table does not break the application users depend on. | Data Integration | Palantir, Databricks | Hard |
| 15 | Parse a messy CSV with inconsistent quoting, mixed encodings and ragged rows, and produce a clean dataset. | Data Integration | Palantir, OpenAI | Medium |
| 16 | Design a pipeline that ingests terabytes of sensor data in mixed JSON, CSV and XML formats and surfaces failure predictions to a non technical user. | Data Integration | Palantir, Databricks | Hard |
| 17 | Write a SQL query that returns the month over month change in average rating for each product. | Data Integration | Palantir, Databricks | Medium |
| 18 | Given a table of client sessions across cities, write a SQL query totalling session duration per city. | Pipelines and Backend | Palantir, Databricks | Easy |
| 19 | Retrieve records from a paginated REST endpoint and filter them by production period. How do you handle partial failures mid pagination? | Pipelines and Backend | Palantir, OpenAI | Medium |
| 20 | Implement exponential backoff with jitter for a flaky third party API the customer controls. | Pipelines and Backend | Palantir, Sierra | Medium |
| 21 | What data health checks would you attach to a pipeline feeding a mission critical workflow? | Pipelines and Backend | Palantir, Databricks | Medium |
| 22 | How do you make a pipeline fail fast on bad input rather than quietly corrupting everything downstream? | Pipelines and Backend | Palantir, Snowflake | Medium |
| 23 | Write a rate limiter supporting per user and global limits, then extend it to a distributed deployment. | Pipelines and Backend | OpenAI, Sierra | Medium |
| 24 | Given a log file of pipeline runs with job IDs, start times and completion statuses, find every job that exceeded its expected runtime by a configurable threshold. | Pipelines and Backend | Palantir, Databricks | Medium |
| 25 | Design a system that lets several organisations query a shared dataset without exposing each other's raw records. | Pipelines and Backend | Palantir, Snowflake | Hard |
| 26 | A Fortune 500 customer wants the platform deployed inside their own AWS VPC with Okta SSO and Snowflake as the data source. Walk me through it. | Deployment and Infrastructure | Sierra, Snowflake | Hard |
| 27 | How do you ship software into an environment where you have no access to the logs, metrics or runtime? | Deployment and Infrastructure | Palantir, Anduril | Hard |
| 28 | How do you manage version skew when 200 customer environments are all running different releases? | Deployment and Infrastructure | Palantir, Databricks | Hard |
| 29 | Why does normal CI/CD break down when the environments are owned by customers, and what replaces it? | Deployment and Infrastructure | Palantir, Google Cloud | Hard |
| 30 | Walk me through a schema migration that has to roll out across a fleet you do not control. | Deployment and Infrastructure | Palantir, Snowflake | Hard |
| 31 | Choose between an automated single tenant deployment, a fully multitenant one, and a vertically partitioned one for this customer. Justify it. | Deployment and Infrastructure | Anthropic, Google Cloud | Hard |
| 32 | How would you limit blast radius during an upgrade rollout across customer environments? | Deployment and Infrastructure | Palantir, Databricks | Medium |
| 33 | What is the smallest footprint you would install in a customer environment on day one, and why? | Deployment and Infrastructure | Palantir, Sierra | Medium |
| 34 | Two customers want conflicting behaviour from the same feature. Do you fork, flag, or configure? | Deployment and Infrastructure | Palantir, OpenAI | Hard |
| 35 | Here is a Python script of several functions. A couple compute the wrong result. Find and fix them. | Debugging and Operations | Palantir, OpenAI | Medium |
| 36 | Debug a program that models infection spread across a social graph and returns a count that is too high. | Debugging and Operations | Palantir, OpenAI | Hard |
| 37 | Find and fix the bugs in this React application, and explain how each one affects the customer's experience. | Debugging and Operations | Sierra, ElevenLabs | Medium |
| 38 | You inherit 250 lines of unfamiliar code with a subtle logic bug. Talk me through your first ten minutes. | Debugging and Operations | Palantir, Cognition | Medium |
| 39 | Your largest customer's production system is down and the logs show intermittent connection timeouts to your service. What do you do first? | Debugging and Operations | Palantir, OpenAI | Hard |
| 40 | A payment module produces incorrect totals under concurrent writes. How do you find the root cause and what do you propose? | Debugging and Operations | Palantir, Ramp | Hard |
| 41 | An aggregation is off by a consistent factor on certain partitions. Is the bug in the aggregation, the partitioning, or the data? | Debugging and Operations | Palantir, Databricks | Hard |
| 42 | Walk me through how you would diagnose high latency in an LLM inference pipeline running on customer infrastructure. | Debugging and Operations | OpenAI, Anthropic | Hard |
| 43 | Design row and column level access controls so two teams share one platform but not one another's data. | Security and Compliance | Palantir, Snowflake | Hard |
| 44 | What is the difference between mandatory markings, discretionary permissions and attribute based controls, and when do you need each? | Security and Compliance | Palantir, Google Cloud | Hard |
| 45 | Design an audit logging system for a government platform where every action must be traceable, tamper evident and queryable by a regulator. | Security and Compliance | Palantir, Anthropic | Hard |
| 46 | A healthcare customer needs a private RAG deployment over 50 million documents under HIPAA constraints. What changes in your architecture? | Security and Compliance | Anthropic, Google Cloud | Hard |
| 47 | The customer's security team will not give you production credentials. How do you unblock yourself? | Security and Compliance | OpenAI, Sierra | Medium |
| 48 | The customer asks for a feature that would weaken their own data governance. How do you push back without damaging the relationship? | Security and Compliance | Palantir, Anthropic | Medium |
| 49 | A regulated customer requires data residency and refuses any egress to the public internet. How do you deliver? | Security and Compliance | Anthropic, Snowflake | Hard |
| 50 | How do you handle a customer security review that arrives three weeks into an eight week engagement? | Security and Compliance | Sierra, Databricks | Medium |
| 51 | Build a retrieval layer over a customer's private knowledge base and justify your chunking and indexing choices. | AI and LLM Deployment | OpenAI, Glean | Hard |
| 52 | When would you use a fixed workflow instead of an autonomous agent, and how do you explain that trade off to a customer who wants agents? | AI and LLM Deployment | Anthropic, Cohere | Medium |
| 53 | Design the tool surface for an agent operating inside a customer's ticketing system. How many tools, and how do you name them? | AI and LLM Deployment | Anthropic, Cognition | Hard |
| 54 | Which parts of this workflow should be enforced deterministically in code rather than trusted to the model? | AI and LLM Deployment | OpenAI, Sierra | Hard |
| 55 | Design guardrails for an AI system that can take actions on the customer's behalf. | AI and LLM Deployment | OpenAI, Anthropic | Hard |
| 56 | When should a request escalate to a human, and who decides the threshold, you or the customer? | AI and LLM Deployment | Sierra, Decagon | Medium |
| 57 | The customer wants better answers. Do you reach for prompt engineering, retrieval, or fine tuning, and how do you decide? | AI and LLM Deployment | OpenAI, Mistral | Medium |
| 58 | A customer's cost per conversation is triple the budget. Give me five levers, ranked. | AI and LLM Deployment | OpenAI, Anthropic | Hard |
| 59 | Explain how prompt caching changes your architecture, and what silently stops it from working. | AI and LLM Deployment | Anthropic, Cursor | Hard |
| 60 | How do you know your AI system is actually working? | Evaluation and Quality | OpenAI, Anthropic | Hard |
| 61 | Build the evaluation set for this use case. Who writes the ground truth, and how many examples do you need before you start building? | Evaluation and Quality | OpenAI, Cohere | Hard |
| 62 | What would you build to catch a regression before the customer sees it? | Evaluation and Quality | OpenAI, Sierra | Medium |
| 63 | When is an LLM judge appropriate, and how do you validate the judge itself? | Evaluation and Quality | Anthropic, OpenAI | Hard |
| 64 | The system passed every eval and users still do not trust it. What now? | Evaluation and Quality | OpenAI, Glean | Hard |
| 65 | How do you detect that a deployed model or workflow has degraded on live production data? | Evaluation and Quality | OpenAI, Databricks | Hard |
| 66 | What metrics and observability would you instrument if this agent shipped to production tomorrow? | Evaluation and Quality | Sierra, Cursor | Medium |
| 67 | The customer wants a 100 percent accuracy guarantee. What do you tell them, and what do you offer instead? | Evaluation and Quality | Anthropic, Sierra | Medium |
| 68 | You have 8,000 rows of London taxi trips with fare, locations, times and distance. Propose something buildable and deployable in one week. | System Design | Palantir, Databricks | Hard |
| 69 | A delivery driver can choose which orders to accept and has full historical and live order data. Design a system that helps him choose. Then scale it to the platform assigning orders. | System Design | Palantir, Ramp | Hard |
| 70 | Design an application to catalogue and log species while exploring an unfamiliar planet. | System Design | Palantir, Cognition | Medium |
| 71 | Design a system for internal teams to organise events and meetups, then write the code for one core feature. | System Design | Palantir, Google Cloud | Medium |
| 72 | Design a system using city traffic data to reduce congestion, and define how you would measure whether it worked. | System Design | Databricks, Google Cloud | Hard |
| 73 | Design a disaster response coordination system. Who are the users, what data arrives, what permissions apply, and what is version one? | System Design | Palantir, Anduril | Hard |
| 74 | Design an end to end batching system for LLM queries and reason about latency and cost. | System Design | OpenAI, Anthropic | Hard |
| 75 | A customer's traffic spikes tenfold overnight. Keep the system inside its latency and cost targets. | System Design | OpenAI, Cursor | Hard |
| 76 | Design an agentic system covering data flow, model integration, orchestration, retrieval and vector storage. | System Design | Google Cloud, Cohere | Hard |
| 77 | You are on a call with a customer's VP of Operations who says "we want AI". What are your first five questions? | Customer Discovery and Scoping | OpenAI, Anthropic | Medium |
| 78 | The workflow the customer described does not match what their data and systems actually do. How do you handle the gap? | Customer Discovery and Scoping | OpenAI, Palantir | Hard |
| 79 | How do you find the person who actually knows why a process works the way it does? | Customer Discovery and Scoping | Palantir, Glean | Medium |
| 80 | A regional bank wants unified fraud detection across three legacy systems from acquisitions, with inconsistent labels. Scope the first 90 days. | Customer Discovery and Scoping | Databricks, Palantir | Hard |
| 81 | Given four candidate use cases from one customer, which do you build first and what is your selection rule? | Customer Discovery and Scoping | Sierra, Glean | Medium |
| 82 | What are the entry criteria, exit criteria and kill triggers for this engagement? | Customer Discovery and Scoping | Databricks, OpenAI | Hard |
| 83 | What does "done" look like here, and how would you know you achieved it? | Customer Discovery and Scoping | Palantir, Cognition | Medium |
| 84 | You have one engineer and one week. Three enterprise customers all want something different. How do you prioritise? | Customer Discovery and Scoping | Sierra, LangChain | Medium |
| 85 | Explain retrieval augmented generation to a VP of Operations who has never written code. | Stakeholder Management | Sierra, LangChain | Medium |
| 86 | Prepare a 20 minute explanation of one of our products for a completely non technical audience. | Stakeholder Management | LangChain, Sierra | Medium |
| 87 | The deployment slipped three weeks and the customer's CTO is on the line. Tell them. | Stakeholder Management | Cognition, OpenAI | Hard |
| 88 | Pitch the outcome of this engagement to a panel of the customer's executives. | Stakeholder Management | Cognition, Ramp | Hard |
| 89 | The customer's principal engineer knows the domain better than you do and disagrees with your design. What do you do? | Stakeholder Management | Cognition, Palantir | Medium |
| 90 | How do you drive adoption after the system technically works but nobody is using it? | Stakeholder Management | OpenAI, Glean | Hard |
| 91 | What would you present to executives to show that the feature you shipped succeeded, and which KPIs would you put on the slide? | Stakeholder Management | Palantir, Databricks | Hard |
| 92 | How do you hand an engagement over so the customer's own team can extend it without you? | Stakeholder Management | Anthropic, Databricks | Hard |
| 93 | Tell me about your biggest career failure and what you took from it. | Behavioral and Situational | Palantir, Anthropic | Easy |
| 94 | Tell me about a deployment that went badly and what you did about it. | Behavioral and Situational | Palantir, OpenAI | Medium |
| 95 | Tell me about a time you disagreed with a customer and held the line. | Behavioral and Situational | Databricks, Sierra | Medium |
| 96 | Tell me about a time you had to operate in a domain you did not understand. | Behavioral and Situational | Palantir, Anduril | Medium |
| 97 | Tell me about a time you spotted a pattern across customers and changed how your team worked. | Behavioral and Situational | Anthropic, Baseten | Hard |
| 98 | Describe your first 30, 60 and 90 days in this role. | Behavioral and Situational | OpenAI, Cognition | Medium |
| 99 | Tell me about the most complex project you have led, and the accomplishment you are proudest of. | Behavioral and Situational | ElevenLabs, Cognition | Easy |
| 100 | This role involves travel of 25 to 50 percent and long stretches inside someone else's building. Why does that work for you? | Behavioral and Situational | Palantir, Anthropic, OpenAI | Easy |

---

## Preparation Plan

Two weeks is enough if you spend it on the rounds that actually fail people rather than on the ones that feel productive. Grinding algorithms is necessary but far from sufficient here: the reported coding bar is LeetCode easy to medium with an emphasis on tests and completeness, while Decomposition, Learning and the eval question are where candidates are eliminated.

**Before day one, do the two-minute triage that saves the whole plan.** Confirm with your recruiter which exact title you are interviewing for. Candidates report that Palantir runs Forward Deployed Software Engineer (Delta) and Deployment Strategist (Echo) as different loops with different rounds. OpenAI posts more than one forward deployed family side by side, with experience bars ranging from 5+ years on its Forward Deployed Engineer, Gov posting to 7+ years on its Forward Deployed Software Engineer posting, and it also runs neighbouring customer-facing engineering titles that candidates routinely confuse with FDE. Anthropic's careers board, as of mid-2026, listed its customer-facing engineering under Applied AI rather than Forward Deployed Engineer. Anduril's and Applied Intuition's forward deployed roles are defense field and hardware jobs with clearance requirements. Preparing for the wrong variant is the most expensive mistake available.

**Week 1: the technical surface**

| Day | Focus | What "done" looks like |
|---|---|---|
| 1 | Role literacy and company research | You can state the outcome-ownership definition, the Dev versus Delta split, and name a specific deployment at your target company |
| 2 | Decomposition reps | Three timed 45-minute runs on prompts like the London taxi dataset, the Uber Eats driver system, and "why are we delaying so many flights", each ending on KPIs and an executive summary |
| 3 | SQL and data wrangling | Window functions, month-over-month deltas, joins across mismatched keys, and profiling queries you can write without a reference |
| 4 | Pipelines and integration | Watermark versus CDC, idempotent MERGE, paginated REST ingestion with backoff and checkpointing, data health check taxonomy |
| 5 | Re-engineering practice | Take an unfamiliar 200 to 300 line repo, plant nothing, and find a real bug by hypothesis rather than by reading every line |
| 6 | Learning round simulation | Ask someone to hand you an unfamiliar library's docs and a task, then narrate continuously for 45 minutes. Long silences read as being stuck |
| 7 | System design for FDE shapes | Terabyte-scale mixed-format ingestion, shared dataset with per-organisation isolation, audit logging for a regulated platform |

**Week 2: the AI, customer and behavioral surface**

| Day | Focus | What "done" looks like |
|---|---|---|
| 8 | Evals | You can answer "how do you know your AI system is actually working" with a golden set, per-intent slices, grader selection, regression gating in CI, and a named customer-side sign-off owner |
| 9 | RAG and retrieval | Chunking, contextual retrieval, hybrid BM25 plus embeddings, retrieve top-150 and rerank to top-20, plus access control applied at query time |
| 10 | Agents and tools | Agent versus workflow justification, consolidated task-shaped tools, output caps, per-tool risk ratings, escalation triggers, trajectory evals |
| 11 | Deployment and compliance | Customer VPC and air-gapped failure modes, version skew and fleet upgrades, tenancy models, data residency and what it costs, PII handling, layered access control, audit logging, and how to confirm a vendor's current authorization status in writing rather than reciting one from memory |
| 12 | Customer craft | Run a mock discovery call. Practise the redirect: what questions would you ask the customer first, before any architecture |
| 13 | Behavioral story matrix | Six to eight stories indexed to failure, adoption, ambiguity, disagreement, pattern-spotting. Two to three minutes each, in "I" not "we", ending on mechanism and carryover |
| 14 | Presentation and dry run | Deliver a 20-minute explanation of a technical product to a non-technical listener, record it, and watch it back |

**Daily throughout:** 45 to 60 minutes of coding practice at easy to medium difficulty, always writing tests, always finishing. Use a mainstream imperative language you are genuinely fluent in (Python, Java, C/C++, C#, JavaScript, TypeScript, Go or Rust), because these loops are not the place to demonstrate an exotic one, and be ready to code on a whiteboard onsite.

**Read the primary sources, not the listicles.** Palantir publishes its own hiring-process guidance on its careers site, and the topics it covers map closely onto its round names. Read the current set yourself rather than trusting anyone's summary of it, including this one. Read the job description of the exact role you applied to and mine it for case prompts; Palantir's FDSE posting literally lists "Why are we delaying so many flights?" and "How can we better identify instances of money laundering?"

**The pitfalls worth rehearsing away:** jumping to a solution before scoping; treating Decomposition like a FAANG system design round; enumerating trade-offs forever without landing on a concrete approach; going silent while thinking; the rewrite instinct in a debugging round; hand-waving evaluation; generic "why this company" answers; and saying "we" when the interviewer is trying to work out what you personally did.

---

## Additional Resources

- [Palantir Careers: Getting Hired](https://www.palantir.com/careers/getting-hired/) - Palantir's own hiring-process guidance, which maps onto its round names
- [Nabeel Qureshi: Reflections on Palantir](https://nabeelqu.substack.com/p/reflections-on-palantir) - the canonical first-hand account of the original FDE model, and the source of the Airbus, onsite-cadence, YC-founder and margin claims used above, all of them his observations rather than measured statistics
- [Insight Partners: Demystifying Forward Deployed Engineers](https://www.insightpartners.com/ideas/demystifying-forward-deployed-engineers/) - the source of the Rajkumar Irudayaraj, Jason Martin and Ankit Sobti quotes used above, and of the "Design every engagement to end" section heading, which is the article's own wording rather than any one practitioner's
- [Ramp Engineering: Forward Deployed Engineering](https://engineering.ramp.com/post/forward-deployed-engineering) - the "always be scoping" mantra and the scoping versus generalizing tension
- [LeadDev: The rise of the forward deployed engineer](https://leaddev.com/hiring/the-rise-of-the-forward-deployed-engineer) - the source of Piotr Kraus's remarks on travel, burnout and title inflation quoted above
- [Anthropic Engineering: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) - workflows versus agents, and the named orchestration patterns
- [Anthropic Engineering: Writing Tools for AI Agents](https://www.anthropic.com/engineering/writing-tools-for-agents) - tool consolidation, output caps, namespacing and actionable error design
- [Anthropic: Introducing Contextual Retrieval](https://www.anthropic.com/news/contextual-retrieval) - the published retrieval failure-rate numbers used throughout this page
- [OpenAI: A Practical Guide to Building Agents](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf) - layered guardrails, tool risk ratings and human escalation triggers
- [Martin Fowler: Feature Toggles](https://martinfowler.com/articles/feature-toggles.html) - the toggle taxonomy to use when asked about customization without forking
- [Azure Architecture Center: Multitenant Architectural Approaches](https://learn.microsoft.com/en-us/azure/architecture/guide/multitenant/overview) - the tenancy vocabulary for single-tenant, multitenant and partitioned deployments
