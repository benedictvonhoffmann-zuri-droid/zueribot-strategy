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

Six categories. Most products need at least four. Pick consciously; don't accidentally have only one.

| Category | What it tests | Typical cadence | Cost / call |
|---|---|---|---|
| **Unit eval** | A single LLM call or single agent role hits its pass criteria on a curated set | On-commit (CI) | Low (subset of golden set) |
| **End-to-end eval** | The whole pipeline produces the right output for realistic scenarios | Pre-release | Medium (full golden set) |
| **Adversarial / red-team** | Deliberate attempts to break the system (injection, jailbreak, OOD, social engineering) | Scheduled (monthly) + on any system-prompt change | Medium-high (dedicated adversarial set) |
| **Regression** | Catch quality drops between releases — historical reference run | Pre-release + every model swap | Same as end-to-end |
| **Production telemetry** | Live behavior (latency, error rates, refusal rates, escalation rates) | Continuous | Free *(in-band metrics)* |
| **User feedback** | Explicit corrections + implicit signals (abandonment, escalation, re-asking) | Continuous + weekly aggregation | Free at point of capture; cost to process |

Each category answers a different question. You can ship without one, but you should know *which one* you're skipping and *why*.

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

## Confidence UX patterns

How does the system *communicate uncertainty to the user*? This is the eval strategy made user-facing.

| Pattern | What it looks like | When to use |
|---|---|---|
| **Tiered confidence** | Visual badge: high / medium / low confidence | Default for any answer system; users build calibrated trust over time |
| **Refusal at low confidence** | System refuses to answer rather than guess | Mandatory for safety-critical domains (legal, medical, financial) |
| **Answer-with-caveat** | Answer + explicit "I'm not sure about X" | Right for medium-confidence answers where partial info is useful |
| **Source citation** | Answer + verifiable links to where it came from | Builds trust via verifiability; the user becomes the final check |
| **Visual citation (HITL-by-UX)** | Citations are clickable anchors back into the source | Strongest pattern; the user can't trust without seeing the source, the UI makes the source prominent |

The confidence UX *is* the contract. If your system surfaces "I'm 95% confident" but is wrong 30% of the time, the UX is broken even if the numbers are technically calibrated.

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
