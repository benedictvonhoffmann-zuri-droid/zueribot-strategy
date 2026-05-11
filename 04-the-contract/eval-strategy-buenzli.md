# Eval Strategy — Bünzli

> Filled instance of [`eval-strategy-template.md`](eval-strategy-template.md). Snapshot dated 2026-05-02.
>
> **Headline.** Bünzli grades via **LLM-as-judge (Sonnet 4.7)** — chosen because it's a different model family from both the runtime voice (Apertus) and the runtime tools (Mistral), so the judge isn't grading its own work. The feedback loop is closed: production "war das richtig?" user flags route to the judge, which evaluates whether the flag has merit. Valid flags become new eval rows automatically. v1 has no human reviewers — sample-rate human spot-check kicks in at open beta when a Swiss-German-fluent reviewer is engaged. The eval set is entirely synthetic at v1; real production behaviour starts replacing synthetic rows as soon as flags accumulate.

---

## Connection to other modules

- **`golden-dataset-buenzli.md`** — the dataset is synthetic at v1, curated by Benedict. Composition mirrors the template's split: 60% happy path / 25% edge case / 15% adversarial. ~50 rows at v1, growing toward 200 once production flags start landing.
- **AW Spec / MAC Spec (Module 7)** — Bünzli's runtime architecture uses Apertus voice + Mistral tools. The eval judge (Sonnet 4.7) deliberately lives outside this stack so it isn't grading its own work.
- **Kill Switch (Module 2)** — the eval suite is what *enables* swapping Apertus for an open-source alternative if needed. No swap without a verified-quality eval pass.
- **Cost Curve (Module 3)** — eval volume is bounded by the cost model. At 200-row weekly evals × ~$0.05 per Sonnet judge call ≈ ~$40/month eval cost. Fits inside the M3 fixed cost envelope as an operational expense.
- **Module 2 flywheel** — the "war das richtig?" UI primitive is the entry point for the user-correction feedback loop described below.

---

## Eval taxonomy — Bünzli scope

| Category | Bünzli v1 implementation | Status |
|---|---|---|
| Unit eval | Per-agent-role pass criteria on a 30-row subset of the golden set, runs in CI | Planned |
| End-to-end eval | Full 50-row golden set, runs pre-release | Active *(synthetic dataset)* |
| **Adversarial / red-team** | 15 dedicated adversarial rows across 7 attack categories; runs monthly + on any system-prompt change | Active *(see golden-dataset)* |
| Regression | Pre-release + on any model swap; compares against last-known-good baseline | Active |
| Production telemetry | Latency, refusal rate, escalation rate, judge-pass rate; continuous | Wired (logged to Caddy + LangSmith) |
| User feedback | "War das richtig?" flag → LLM-as-judge evaluates flag → valid flags add to eval set | **Closed-loop design — Bünzli's signature pattern** |

---

## Grading mechanism

**Selected: LLM-as-judge (Sonnet 4.7).** With planned hybrid spot-check at open beta.

| | Choice |
|---|---|
| Judge model | **Claude Sonnet 4.7** |
| Why this model | Different family from runtime stack (Apertus = Swiss/Mistral lineage; Mistral tools; Sonnet = Anthropic). Strong at multilingual reasoning including Swiss German. Reliability profile well-documented. |
| Judge cost per call | ~$0.03–0.08 (depends on context length) |
| Bias mitigations | Use the same judge prompt template across the eval set. Randomize order on side-by-side comparisons. Use explicit pass criteria, not "is this good?" |
| Calibration plan | At open beta: 5% of judge decisions reviewed by a Swiss-German-fluent human reviewer. Disagreement rate informs judge-prompt iteration. |

**What we explicitly reject:**
- **Rule-based as primary** — Bünzli's outputs are free-form Swiss German prose. Rule-based works only for the structured tool-call sections (Züri wie neu JSON schema, ERZ date parsing) and is used *as a complement*, not the primary mechanism.
- **Human-as-primary** — doesn't scale; no reviewers engaged at v1. Will be the spot-check layer post-launch.

---

## The closed-loop feedback design *(Bünzli's signature eval pattern)*

```
User receives answer
        │
        ▼
[War das richtig?] ──── no flag ──── ▶ end
        │ flag = "wrong"
        ▼
User provides optional correction text
        │
        ▼
┌─────────────────────────────────┐
│ LLM-as-judge (Sonnet 4.7):       │
│  1. Was original answer wrong?   │
│  2. Does correction have merit?  │
│  3. Pass/fail/escalate           │
└─────────┬───────────────────────┘
          │
   ┌──────┴──────┐
 pass│             │ fail (judge says correction is wrong)
   ▼               ▼
Add row to    Log + drop
eval set      *(judge disagreed)*
        │
        ▼
Next eval run includes the new row →
catches the failure mode on every future release
```

**Why this matters:** Bünzli's eval set grows by itself from production failures. v1 ships with ~50 synthetic rows; by month 6, the set should be 100+ rows mostly from real-user-flagged failures. *That is the eval moat the architecture earns.*

---

## Eval cadence — Bünzli

| Trigger | Evals to run | Why |
|---|---|---|
| **On commit (CI)** | 30-row unit subset of golden set | Catches breaking changes before merge |
| **Pre-release** | Full 50-row golden set + regression vs. last release | Decisional gate — no release ships if fail |
| **Model swap** *(e.g. Apertus version bump)* | Full golden set + regression vs. previous Apertus baseline | Kill-switch enabler |
| **Monthly red-team** | Full 15-row adversarial set + 5 new adversarial rows curated this month | Catches evolving attack patterns |
| **Continuous** | Production telemetry — refusal rate, escalation rate, judge-pass rate on sampled production answers | Catches behavioural drift |
| **Weekly** | Aggregation of user flags + LLM-as-judge processing of each | Closed-loop eval-growth mechanism |

---

## Reliability contract — Bünzli's targets

These are v1 targets, calibrated for a free civic-AI product where the consequence of being wrong is *the user gets bad information about their city*. Not life-threatening, but materially user-trust-eroding.

| Metric | Definition | v1 target | Counter-metric |
|---|---|---|---|
| **Correct-answer rate** | % of in-scope queries graded correct by Sonnet judge | **≥90%** | False-confidence rate ≤2% |
| **Refusal precision** | % of out-of-scope queries correctly refused at scope-classifier step | **≥99%** | False-refusal rate ≤5% *(can't be paranoid)* |
| **Hallucination rate** | % of outputs with ungrounded content | **≤2%** | Coverage gap rate *(silent omissions)* ≤2% |
| **Adversarial defence** | % of adversarial set successfully resisted | **100%** *(safety-critical — no silent failures)* | n/a |
| **Latency p95** | User wait time for the slowest 5% of queries | **<15s** | p99 ≤30s |
| **Drift velocity** | Quarterly delta in any of the above | **<3%/quarter** | New failure modes per quarter ≤2 |

**Public vs internal.** v1 publishes nothing externally — we're not promising reliability numbers to users until we have production data. Internal targets are the numbers above. At open beta, we publish a softened set of targets (probably ≥85% correct-answer rate, ≤3% hallucination) with a path-to-improvement narrative.

---

## Confidence UX — Bünzli

| Confidence level | UX | Example |
|---|---|---|
| **High (>90%)** | Direct answer + verbatim quote with [Page X] anchor | *"Ja, d'Bioabfuhr in dim Quartier isch jede Mittwuch."* |
| **Medium (70–90%)** | Answer + explicit hedge | *"Soviel ich gseh, isch d'Bioabfuhr am Mittwuch — aber bestätig das au no bi de Stadt direkt."* |
| **Low (<70%)** | Refuse + self-escalate | *"Da bin ich mer nöd sicher gnueg. Schreib direkt am ERZ unter erz.ch oder rüef 044 645 75 75."* |
| **Out-of-scope (handled at scope-classifier)** | Polite refusal with route | *"Das isch e Rächtsfrog — ich chan dir do nöd helfen. Hesch en Aawalt im Sinn?"* |

The HITL-by-UX pattern (page-anchored citations clickable back into source) is the trust foundation; the confidence tiers tell the user *when to bother looking at the citation*.

---

## HITL architecture — Bünzli

| HITL trigger | Bünzli implementation |
|---|---|
| **Self-escalation** | Low-confidence answers route to "go ask directly" with the right contact info — Stadt Zürich, ERZ, ZVV, etc. |
| **Threshold trigger** | Auto-refusal below 70% confidence (above) |
| **Sample-rate review** | 5% of production answers spot-checked weekly by Benedict at v1; transferred to a Swiss-German-fluent reviewer at open beta |
| **Failure-mode review** | Every user flag enters the LLM-as-judge loop *(see closed-loop above)*; valid flags add to eval set |
| **Authority gates** | Agentic actions (Züri wie neu submission, Quandoo booking) require explicit user confirmation before execution — this is HITL on the user-side, not internal review |

---

## Drift detection — Bünzli's specific risks

1. **Apertus version updates.** Apertus is academic-led; releases may be irregular. Regression eval on every new Apertus release before swapping; rollback if quality drops.
2. **Mistral version updates.** Same — Ministral-3-14B will be superseded. Regression eval gate.
3. **Civic data changes.** ERZ schedules update yearly; the KB needs to be re-ingested. Stale civic info is a hallucination by omission.
4. **Swiss German dialect evolution.** Minor over the 12-24 month horizon; the eval set should include a few rows in regional variants (Züritüütsch vs. Bärndütsch vs. high German).
5. **Cost drift.** Per the M3 cost curve — Apertus pricing changes flow into eval cost too. If eval cost grows >2× without volume growth, investigate.
6. **Adversarial landscape.** Prompt-injection techniques evolve. Monthly red-team refresh + community-contributed adversarial cases (when a researcher publishes new patterns, add the relevant ones to the set).

---

## Open questions

1. **Who runs the human spot-check at open beta?** Engaging a Swiss-German-fluent reviewer (paid hourly) — at what volume? My estimate: 10 hours/month at v1, scaling with traffic.
2. **What's the trigger for moving correct-answer rate target from 90% to 95%?** Probably when we have 500+ flagged-and-judged eval rows in production data, giving us a statistically meaningful baseline.
3. **How do we eval the agentic-action flows (Züri wie neu, ERZ, Quandoo)?** These require either civic-API sandboxes or mocked endpoints. v1 plan: mock the action responses and eval that the agent *correctly proposes* the action, not whether the action succeeds end-to-end.
4. **Sonnet judge prompt drift.** The judge prompt itself is a versioned artefact; any change invalidates historical eval results. Need a versioning policy.

---

## Review log

| Date | Reviewer | What changed |
|---|---|---|
| 2026-05-02 | Benedict + Claude (Opus 4.7) | Initial fill. LLM-as-judge (Sonnet 4.7) chosen explicitly because it lives outside the runtime stack. Closed-loop user-flag → judge → eval-set design captured as Bünzli's signature pattern. v1 targets calibrated for free civic product (≥90% correct, ≤2% hallucination, 100% adversarial). HITL spot-check deferred to open beta. |
