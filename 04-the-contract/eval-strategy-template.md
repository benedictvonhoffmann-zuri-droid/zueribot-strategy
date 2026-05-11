# Eval Strategy

> **Purpose.** Probabilistic systems need a contract with the user about reliability. The eval strategy is how you build that contract: *what we measure, how we measure it, what triggers when we miss, how the user verifies our claims.* Without an eval strategy you don't have a reliability claim — you have a hope.
>
> **The structural insight this doc serves.** Most product teams treat eval as "we have a golden dataset." That conflates the *measurement instrument* (the dataset) with the *measurement methodology* (the strategy). This doc owns the methodology; [`golden-dataset.md`](golden-dataset.md) owns the dataset. Both are required; neither substitutes for the other.
>
> **How to use it.** Pick a grading mechanism honestly. Set reliability targets you can defend in front of users. Wire the feedback loop *before* shipping — the day after launch is too late to start instrumenting it.

---

## Connection to other modules

- **`golden-dataset.md`** — the dataset is the *measurement instrument*. This doc is the methodology that uses it. Co-design both.
- **AW Spec / MAC Spec** — every Critic role in the agent architecture is itself an eval. The agent's runtime self-verification and the eval strategy should be aligned: same grading mechanism, same pass criteria.
- **Kill Switch (Module 2)** — model swap-ability is only useful if you can verify the new model meets the bar. The eval suite is what *enables* the kill switch in practice.
- **Cost Curve (Module 3)** — evals cost money to run (especially LLM-as-judge). Eval cost is a line item in the cost curve, not a free externality.
- **Privacy posture** — eval data may itself be sensitive. If your product is stateless-by-design, your eval set has to be synthetic or PII-redacted.

---

## Eval taxonomy

Ten categories across two axes — **static vs dynamic**.

**Static eval** uses a fixed, curated dataset (the golden set). Same inputs every run, same expected outputs. Output: a number you can compare against last week's number. Measures regression.

**Dynamic eval** generates or samples inputs at runtime. Different inputs every run. Output: discovery of failure modes you didn't think to test for.

**A serious eval strategy uses both.** Static is the controlled baseline that lets you measure whether you broke what you promised would work. Dynamic is the exploration frontier that surfaces what you should *add* to the static set so the floor rises over time. Static without dynamic gives you false confidence ("nothing regressed!" — but you only test 50 things). Dynamic without static gives you no baseline ("everything's changing all the time, who knows what's broken").

| # | Category | Static / Dynamic | What it tests | Typical cadence | Cost / call |
|---|---|---|---|---|---|
| 1 | **Unit eval** | Static | A single LLM call or agent role hits pass criteria on a curated subset | On-commit (CI) | Low |
| 2 | **End-to-end eval** | Static | Full pipeline produces the right output for realistic scenarios | Pre-release | Medium |
| 3 | **Adversarial / red-team (curated set)** | Static | Known attack categories (injection, jailbreak, OOD, social engineering, etc.) | Monthly + on system-prompt changes | Medium |
| 4 | **Regression** | Static | Catch quality drops between releases against historical baseline | Pre-release + every model swap | Same as end-to-end |
| 5 | **Production telemetry** | Dynamic | Live behaviour (latency, refusal rate, escalation rate, judge-pass rate) | Continuous | Free |
| 6 | **User feedback** | Dynamic | Explicit corrections + implicit signals (abandonment, re-asking) | Continuous + weekly aggregation | Free at capture; cost to process |
| 7 | **Production replay** | Dynamic | Real user queries (PII-scrubbed) re-run on new model/prompt, output compared | Pre-release + on model swap | Medium |
| 8 | **Continuous random sampling** | Dynamic | LLM-as-judge runs on a random 5-10% of live answers, in near-real-time | Continuous | Low per sample; cumulative |
| 9 | **Synthetic generation eval** | Dynamic | A generator LLM produces *novel* adversarial inputs on a schedule; results feed back into the curated adversarial set | Monthly | Medium (generation + grading) |
| 10 | **Property-based eval** | Dynamic | Properties any valid response must have (e.g. "every quoted clause exists verbatim in source"); tested against many novel inputs | Continuous + per-release | Low (rule-checkable) |

You don't need all ten. Most products do well with **4 static (1, 2, 3, 4)** + **3 dynamic (5, 6, 10)**, adding production replay and synthetic generation as the system matures. Property-based eval is criminally underused — when it applies to your domain, it's the cheapest highest-signal eval available.

---

## Grading mechanism rubric

How do you decide an output is right or wrong? Four mechanisms. Each has a cost / reliability / scale profile.

| Mechanism | What it is | Cost per call | Reliability | When it works | When it breaks |
|---|---|---|---|---|---|
| **Rule-based** | Regex, schema validation, exact match, deterministic checks | Free | Perfect within the rule | Outputs with structural rules (JSON schemas, specific anchor strings, format constraints) | Free-form text outputs; nuanced correctness; subjective quality |
| **LLM-as-judge** | A model grades each output against criteria + reference answer | ~$0.01–0.10 per call | Good but biased *(length bias, position bias, judge can be wrong; periodic recalibration required)* | Scalable grading of nuanced text outputs; large datasets; rapid iteration | Catastrophic mistakes if judge model has the same blind spots as the runtime model; cost at scale |
| **Human expert** | A domain expert grades each output | $5–50 per call *(depending on expertise)* | Gold standard | Ground-truth construction; high-stakes domains; building trust in any other mechanism | Doesn't scale to hundreds of evals per release; expensive; bottleneck |
| **Hybrid** | LLM-as-judge with human spot-check on a sample (~5–10%) | LLM cost + (sample × human cost) | Strong | Production-grade systems that need both scale and calibration | Setup overhead; requires you to actually *do* the spot-check, not just promise it |

**Decision principle:** Use the cheapest mechanism that meets your reliability bar. For most AI products that means LLM-as-judge with human spot-check (hybrid). For high-stakes legal / medical / financial outputs, human-in-the-loop is mandatory. For structured outputs (JSON, schemas), rule-based is the right starting point.

**The LLM-as-judge trap:** If your judge is from the same model family as your runtime, you're effectively self-grading. Different family + different prompt + periodic human spot-check is what makes LLM-as-judge defensible.

---

## Eval cadence

When does each eval run?

| Trigger | Evals to run | Why |
|---|---|---|
| **On commit** *(CI)* | Unit evals (small subset of golden set, fast) | Catches breaking changes before they merge |
| **Pre-release** | Full golden set (end-to-end + regression) | Decisional gate — release does not ship if eval fails |
| **On any model swap** | Full golden set + regression vs. previous baseline | Kill-switch enabler; no swap without verified quality |
| **Scheduled (e.g. monthly)** | Adversarial red-team set | Catches drift in attack landscape; new injection patterns |
| **Continuous** | Production telemetry + user feedback aggregation | Catches drift in real user behavior |
| **On any prompt-template change** | Full golden set | Prompts are code; treat changes accordingly |

---

## Reliability contract — quantified targets

This is the *user-facing claim* about how reliable the system is. The contract has to be:
1. **Quantified** (not "highly accurate")
2. **Measured against a defined eval set** (not "in our experience")
3. **Tied to a counter-metric** (the thing that mustn't go up while the primary goes up)
4. **Anchored to a consequence** (what happens when we miss)

Typical targets:

| Metric | Definition | Target *(adjust per product)* | Counter-metric |
|---|---|---|---|
| **Correct-answer rate** | % of in-scope queries the eval grades as correct | ≥95% | False-confidence rate ≤1% |
| **Refusal precision** | % of out-of-scope queries correctly refused | ≥99% | False-refusal rate ≤5% *(don't over-refuse)* |
| **Hallucination rate** | % of outputs that contain ungrounded content | ≤1% | Coverage gap — silent omissions |
| **Latency p95** | Worst 5% of user wait times | <X seconds | p99 — extreme tail |
| **Drift velocity** | How fast metrics move between releases | <Y%/month change | New failure modes per quarter |

**Public vs internal targets.** You generally publish the *softer* of the two — what you can comfortably commit to externally — and hold internal teams to a tighter target. Don't publish a number you can't defend with eval data.

---

## Perceived confidence — making the contract feelable in the UI

The reliability contract above is a number. **Perceived confidence is whether the user can *feel* that number in the response itself** — in the wording, the layout, the visual treatment, the actions the UI invites. The eval strategy made tangible.

**The principle.** Don't *tell* the user how confident the system is; *show* it through every design dimension of the response. A user who reads a 70%-confidence answer rendered as a definitive paragraph in normal typography will trust it like a 99%-confidence answer — because nothing in the experience tells them not to. The label said "medium confidence"; the design said "fully confident." The design wins.

The goal is **calibrated trust**: users trust high-confidence answers, scrutinise low-confidence ones, and follow escalations without arguing. The only way to get there is to make confidence *felt*, not labelled.

### Six dimensions of perceived confidence

A response can encode confidence through any (ideally several) of these design dimensions. Don't pick one — layer them.

| Dimension | High-confidence response | Low-confidence response |
|---|---|---|
| **Linguistic** | Direct, declarative. *"Yes. The agreement runs 24 months."* | Hedged language baked in. *"From what I can find, this looks like a 24-month term — please double-check section 3."* |
| **Visual hierarchy** | Clean answer first; citation in tasteful inline form | Answer is *visually demoted* relative to the source; the source is the primary visual element, the answer is below it |
| **Typography / motion** | Decisive — appears at full speed, full weight, no hedging glyphs | Visibly slower presentation; explicit "thinking" indicator; weight or colour subtly softer |
| **Source prominence** | Quote presented inline as inline-citation, [Page 4] anchor inline | Source *foregrounded* — the quote appears in a prominent callout box, with the agent's interpretation below as supporting commentary, inviting the user to read the source first |
| **Iconography / colour accents** | Small green / "verified"-style indicator | Small amber / "review"-style indicator; ideally tied to a tooltip explaining *why* low confidence |
| **Action affordance** | "Anything else?" — invites a follow-up | "Want me to check with [authority]?" or "Here's the page on zurich.ch for the official answer" — invites verification or escalation |

### The five patterns these dimensions implement

| Pattern | What it does | When to use |
|---|---|---|
| **Tiered confidence indicator** | Small persistent visual cue per answer (badge/colour/icon) | Default; users build a mental model over time of what each tier means |
| **Refusal at low confidence** | System refuses to answer rather than guess; routes to authority | Mandatory for safety-critical domains (legal, medical, financial); recommended for high-trust contexts |
| **Answer-with-caveat (linguistic hedging)** | Answer is given, but the *wording* of the answer conveys uncertainty | Medium-confidence; partial info is useful |
| **Source-foregrounded answer** | The source quote is the primary visual element; the agent's synthesis is supporting | Use whenever confidence is anything less than high — invites verification |
| **HITL-by-UX (visual citation)** | Citations are clickable anchors back into the source document, highlighted | Strongest pattern; the user can't trust without seeing the source, the UI makes the source prominent |

**The trap to avoid.** Showing a "70% confidence" label below a confidently-phrased paragraph in normal typography is *worse than no label*. Users learn to ignore the label because the design contradicts it. Either commit to the perceived-confidence design across every dimension, or don't display the label at all.

**The contract.** Perceived confidence *is* the reliability contract made user-facing. If the system's eval results say it's right 90% of the time on this query type, and the user feels equally confident in *all* of its answers, the UX has broken the contract — even if the numbers haven't. Calibrating perception is the work.

---

## HITL architecture

When does a human enter the loop?

| HITL trigger | What it covers | Example |
|---|---|---|
| **Self-escalation** | Agent voluntarily routes to human when uncertain | "I don't have a confident answer — recommending you contact Stadt Zürich directly" |
| **Threshold trigger** | Automated routing when confidence < threshold | Refuse + escalate any answer below 70% confidence |
| **Sample-rate review** | Random sample of all outputs reviewed by human | 5% of production answers reviewed daily for drift detection |
| **Failure-mode review** | Every flagged failure triggers human review | User flags "wrong answer" → goes to review queue |
| **Authority gates** | High-stakes actions require human approval before execution | Agent drafts an email; human approves before send |

**Closing the loop.** Human reviews are valuable only if they feed back into the eval set. Every confirmed failure becomes a new eval row; every disagreement with the agent becomes calibration data for the judge model.

---

## Drift detection

Things that change without you doing anything:

1. **Model provider updates** — Claude 4.5 → 4.6 changes behavior. Catch with regression evals on every detected version.
2. **User distribution shifts** — your eval set was designed for 2024 users; 2025 users ask different questions. Catch with production telemetry segmentation.
3. **Adversarial landscape evolution** — new prompt-injection patterns emerge. Catch with continuous red-team contribution from security community + scheduled refresh.
4. **Vendor pricing changes** — same model becomes 3× cost. Catch with cost telemetry tied to eval volume.
5. **Compliance landscape shifts** — EU AI Act, revDSG, sector-specific regulation. Catch with quarterly compliance review.

**Drift response cadence:** small drift → tune at next release. Material drift → reset reliability claims, reset eval set, communicate to users.

---

## Open questions

- 
- 

---

## Review log

| Date | Reviewer | What changed |
|---|---|---|
| | | |
