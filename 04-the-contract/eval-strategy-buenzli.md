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

Bünzli runs both static and dynamic eval — both are required (static for measuring regression against the controlled baseline; dynamic for discovering failure modes the static set doesn't cover yet).

### Static evals (golden-set-based)

| Category | Bünzli v1 implementation | Status |
|---|---|---|
| Unit eval | Per-agent-role pass criteria on a 30-row subset of the golden set, runs in CI | Planned |
| End-to-end eval | Full 50-row golden set, runs pre-release | Active *(synthetic dataset)* |
| Adversarial / red-team (curated set) | 15 dedicated adversarial rows across 7 attack categories; runs monthly + on system-prompt changes | Active |
| Regression | Pre-release + on any model swap; compares against last-known-good baseline | Active |

### Dynamic evals (runtime / production-driven)

| Category | Bünzli v1 implementation | Status |
|---|---|---|
| Production telemetry | Latency, refusal rate, escalation rate, judge-pass rate; continuous | Wired (Caddy + LangSmith) |
| User feedback | "War das richtig?" flag → LLM-as-judge → valid flags add to eval set | **Closed-loop — Bünzli's signature pattern** |
| Continuous random sampling | 5% of live answers re-graded by judge in near-real-time | Planned for v1 launch + 1 month |
| Production replay | On any model swap, last 100 production queries (PII-scrubbed) re-run on new model; outputs compared | Planned for first model swap |
| Synthetic generation eval | Sonnet generates 5-10 *novel* adversarial inputs in Swiss German each month; results scored, valid attacks added to curated adversarial set | Planned for v1 + 3 months |
| Property-based eval | Every claim cited in a Bünzli response must appear verbatim in the retrieved context; rule-checkable continuously | **Active — automated check on every response, not just eval rows** |

**The discipline:** static evals are the *floor* (have we regressed?). Dynamic evals are the *frontier* (what are we failing on that we never thought to test?). Failures discovered dynamically become rows in the static set — the dataset compounds, the floor rises.

---

## Grading mechanism

**v1: LLM-as-judge (Sonnet 4.7 via API).** **v1 + 3 months: planned migration to Prometheus 2 (8x7B) on rent-a-pod.**

### v1 — Sonnet 4.7 via API

| | Choice |
|---|---|
| Judge model | **Claude Sonnet 4.7** |
| Why this model at v1 | Different family from runtime stack (Apertus = Swiss/Mistral lineage; Mistral tools; Sonnet = Anthropic). Strong at multilingual reasoning. Operationally simplest — API call, no infra to manage. Right tradeoff for shipping speed. |
| Judge cost per call | ~$0.03–0.08 |
| Bias mitigations | Same judge prompt template across the eval set. Randomize order on side-by-side comparisons. Explicit pass criteria, not "is this good?" |
| Calibration plan | At open beta: 5% of judge decisions reviewed by a Swiss-German-fluent human reviewer. Disagreement rate informs judge-prompt iteration. |

### v1 + 3 months — Prometheus 2 on rent-a-pod (planned migration)

The decision: at v1 + 3 months, run **Prometheus 2 8x7B on rent-a-pod** (e.g. RunPod A100 80GB, EU/Swiss region preferred) as the judge.

| | Choice |
|---|---|
| Judge model | **Prometheus 2 8x7B** *([arxiv 2405.01535](https://arxiv.org/abs/2405.01535))* |
| Why migrate | (1) **Sovereignty:** closes the only Anthropic dependency in Bünzli's stack — runtime is Mistral-family (Apertus + Ministral); eval becomes Mistral-family too. Full sovereign-AI architecture. (2) **Open-source auditability:** the model judging Bünzli's outputs is itself open and reproducible by external parties — a public reliability claim becomes externally verifiable in a way "we use Anthropic Sonnet" never can. (3) **Cost:** rent-a-pod at ~$1.19/hr × ~15 eval runs/month × 10-15 min per run ≈ ~$3-5/month, vs. ~$25-40/month for Sonnet at the same volume. |
| Judging Swiss German artifacts | **Eval rubric phrased in English; artifacts being judged are Swiss German.** Judge needs semantic-equivalence recognition, not Swiss German fluency. This was Prometheus's biggest perceived weakness; the English-rubric framing largely neutralises it. |
| Operational shape | Pod orchestration script: spin up → load weights (cached on persistent volume) → run eval → shut down. ~10-15 min end-to-end per full eval. |
| Migration validation | Run Prometheus 2 in side-by-side with Sonnet on the full 50-row golden set for one release. Migrate if Prometheus agrees with Sonnet at ≥95%, or if disagreements (on human review) favour Prometheus. |
| Cost target | ~$5/month at planned eval volume *(vs. ~$25-40 on Sonnet)* — meaningful savings that compound as eval volume grows. |

### What we explicitly reject

- **Rule-based as primary** — Bünzli's outputs are free-form Swiss German prose. Rule-based works only for the structured tool-call sections (Züri wie neu JSON schema, ERZ date parsing) and as a complement to the LLM judge, not as the primary mechanism. *(Note: property-based eval — "every quoted claim appears verbatim in retrieved context" — is rule-based and runs continuously alongside the LLM judge.)*
- **Human-as-primary** — doesn't scale; no reviewers engaged at v1. Spot-check layer at open beta only.
- **Reward model as judge** (e.g. Nemotron-340B-Reward) — wrong shape: reward models predict *human preference* (which can favour confident wrong answers), not *correctness*. Plus the size is incompatible with Bünzli's cost envelope.

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

## Perceived confidence — Bünzli's design

The principle from the template: *don't tell the user the confidence; make them feel it through every dimension of the response design.* For Bünzli this is especially important because the user base (Zürich residents asking civic questions) needs to build calibrated trust in *which* of Bünzli's answers can be relied on and *which* should be double-checked with the city. The Swiss civic-trust context is high; the consequence of betraying it is brand-existential.

### How each confidence tier is *felt*, not just labelled

| Tier | Linguistic *(Züritüütsch)* | Visual hierarchy | Iconography | Source treatment | Action affordance |
|---|---|---|---|---|---|
| **High (>90%)** | *"Saul sagt: D'Bioabfuhr in dim Quartier isch jede Mittwuch."* — direct, declarative, no hedging | Answer is the dominant visual element; source inline with subtle [Quelle] anchor | Small green "✓ überprüäft" badge (verified-style) | Inline link — *"Quelle: ERZ"* — confidence in source is implicit | *"Es no öppis?"* — invites the next question |
| **Medium (70–90%)** | *"Saul gloubt: D'Bioabfuhr söt am Mittwuch sii — bestätig das aber au no bi de Stadt direkt."* — explicit hedge baked into the *wording* | Source quote *foregrounded* in a callout box ABOVE the answer; the answer is supporting commentary | Small amber "ℹ️ luegisch's nomal aa" badge | Source quote prominently displayed; the user sees the evidence first, then Bünzli's interpretation | *"Wottsch mer das vo ERZ bestätige?"* — explicit link to ERZ canonical source |
| **Low (<70%)** | *"Saul isch sich nöd sicher gnueg. Bessere ruefisch direkt am ERZ aa under 044 645 75 75."* | Bünzli's draft answer is *de-emphasized* or hidden by default behind a "show what Saul thought" expand — the official route is what the user sees first | Red "→ frög lieber d'Stadt direkt" badge | The route to authority is the primary visual element; Bünzli's tentative answer is secondary | Phone number + link to authority page surfaced as primary action |
| **Out-of-scope** *(handled at scope classifier — no Bünzli answer at all)* | *"Das isch e Rächtsfrog — ich chan dir do nöd helfen. Hesch en Aawalt im Sinn? D'Mieterverband Züri isch unter mv-zh.ch erreichbar."* | No answer; refusal explanation is the only content | Red "→ Aawalt nötig" badge | n/a | Specific authority + route provided ("don't ask me, ask this real person") |

### The Zürich-specific design choices

- **"Saul sagt / Saul gloubt / Saul isch sich nöd sicher"** is the canonical opening triplet. It teaches users in three exposures what to expect: when Saul *sagt*, trust it; when Saul *gloubt*, verify; when Saul *isch sich nöd sicher*, don't bother with Saul, ask the authority.
- **Source-foregrounded for anything below high confidence** — for civic content, the official source (ERZ, Stadt Zürich, ZVV) carries far more authority than Saul. When confidence drops, get out of the way and put the source first.
- **Authority phone number as the action affordance for low confidence** — Zürich civic infrastructure is reachable. 044 645 75 75 (ERZ) isn't an abstraction; it's a real number a Zürich resident can call. Surfacing it as the action is the right design move.
- **No fake confidence theatre** — Bünzli should *never* present a 70% answer with the same visual confidence as a 95% answer. If the design and the label say different things, the design wins, and we've broken the contract.

### The HITL-by-UX foundation

Underneath all of this: page-anchored citations clickable back into the source document. **The user can never fully trust Saul without seeing the source — and the UI makes the source one click away in every response above out-of-scope.** That structural feature is what makes the perceived-confidence design honest. Without it, hedging is just decoration.

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
| 2026-05-02 | Benedict + Claude (Opus 4.7) | **Three substantive additions:** (1) **Dynamic eval added as a first-class concern** — taxonomy split into static (4 categories) + dynamic (6 categories) with explicit "static = floor; dynamic = frontier" framing. Property-based eval flagged as Bünzli's continuous grounding check on every response. (2) **Prometheus 2 on rent-a-pod planned for v1 + 3 months migration** — closes the only Anthropic dependency in the stack; ~$5/mo vs ~$25-40/mo for Sonnet; English rubric / Swiss German artifact framing neutralises the multilingual concern. Reward models (Nemotron) explicitly rejected as judge — wrong shape. (3) **Perceived-confidence design** rewritten as the core principle: don't tell, *show*. Six design dimensions × three confidence tiers in Züritüütsch with concrete copy. "Saul sagt / gloubt / isch sich nöd sicher" as the canonical trust-calibration vocabulary. |
