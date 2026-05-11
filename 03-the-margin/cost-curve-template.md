# Cost Curve

> **Purpose.** Understand the unit economics of an AI product. What does it cost to deliver *one unit of value*? How do those costs scale? Where is margin most at risk?
>
> **The structural insight this doc serves.** Traditional SaaS has *fixed* COGS — gross margin lands at 75–90% and stays there as you grow. AI inverts that: **a meaningful chunk of COGS is variable per use** (LLM tokens, vector retrieval, HITL review). Margin is no longer a stable plateau; it's a function of usage patterns. Designing the cost curve is no longer "set it and forget it" — it's an active product responsibility.
>
> **How to use it.** Fill it in once with current realities, then re-score quarterly. Don't fake numbers — empty cells are honest, fake cells are dangerous. Treat the Before / After synthesis at the bottom as the deliverable; everything above is what justifies it.

---

## Timeframe

**Operational — 12 to 24 months.** This doc is about live cost dynamics, not 5-year financial projections. Fixed-cost depreciation, vendor pricing changes, model-version deprecations, scale-up impact — all land within this window.

---

## Connection to other modules

- **Module 1 — Diagnostic Axis 2 (Data).** If the data flywheel compounds, willingness-to-pay per unit can rise faster than cost per unit. The cost curve is one half of margin; the other half is the value the data advantage lets you capture. See `pricing-strategy.md`.
- **Module 2 — Kill Switch.** Vendor-cost stress tests live in the kill switch; vendor-cost *trajectory* lives here. Cost curve assumes the kill switch holds — if it doesn't, *all* cost projections are at risk of step-changes you can't control.
- **Module 2 — Data Flywheel.** Scale dynamics: variable cost per unit might *decrease* with usage (cascade ratios improve as cheaper models handle more) or *increase* (power-user concentration burns through the cheap tier). The flywheel determines which.

---

## AI cost stack

The line items every AI product should account for. Each row needs **type** (fixed vs variable) + **unit cost**. Empty rows where they don't apply.

| Cost line | Type | Unit cost | Notes |
|---|---|---|---|
| Inference — primary model (workhorse generation) | Variable | $ / token or $ / call | The bulk of variable cost for most agents |
| Inference — triage / cheap model | Variable | $ / token or $ / call | Cascading strategy keystone |
| Inference — premium model (for high-stakes calls only) | Variable | $ / token or $ / call | Used sparingly; if the share grows, margin shrinks |
| Embeddings (initial + incremental) | Variable | $ / 1M tokens | One-shot for static KBs; recurring for live KBs |
| Vector DB / retrieval | Fixed or variable | $ / month or $ / query | Self-hosted = mostly fixed; managed = often variable |
| HITL review (sample-rate or full-coverage) | Variable | $ / case reviewed | Labour cost; usually under-counted at design time |
| Guardrail / safety services | Variable or fixed | depends on provider | Often free at low scale, billed at high |
| Observability (LangSmith, Helicone, OTel) | Fixed or variable | $ / month or $ / event | |
| Compute infrastructure (VMs, GPUs, storage) | Fixed | $ / month | The "always on" baseline |
| Bandwidth + egress | Variable | $ / GB | Trivial for chat; meaningful for media |
| Compliance / audit / certifications | Fixed | annualised | Becomes material at regulated-customer scale |

**Fixed-cost subtotal:** `$X / month`
**Variable-cost coefficient:** `$Y per [unit]` *(define the unit explicitly — per query, per token, per resolved task, etc.)*

---

## Cascading strategy

The single biggest lever for managing AI COGS. The principle: **don't pay for premium model capability on queries that a cheaper model can handle correctly.** Designed properly, this cuts blended cost-per-task by 60–90% without quality loss.

| Tier | Model character | Typical cost ratio | What it handles |
|---|---|---|---|
| **Triage** | Cheap, fast small model | 1× | Scope checks, classification, structured extraction, simple Q&A |
| **Workhorse** | Mid-tier general model | 5–20× | Most user-facing generation; the volume layer |
| **Premium** | Frontier / specialist model | 30–100× | High-stakes generation, complex reasoning, fallback when workhorse fails |

**Routing rules.** *(Specific to your task. Example: "Scope check → Triage. If in_scope, generate with Workhorse. If Workhorse confidence < 0.7 OR Critic verdict = fail, escalate to Premium for one retry.")*

**Expected cascade ratio.** *(Honest estimate: e.g. 70% triage-only / 25% triage→workhorse / 5% workhorse→premium. Verify against actual logs once shipped — this number is usually the most wrong assumption in the model.)*

**Blended cost per task.**
```
Blended = (ratio_triage × cost_triage) + (ratio_workhorse × cost_workhorse) + (ratio_premium × cost_premium)
```
*Compute this explicitly. The blended cost — not any single model's cost — is what determines unit economics.*

---

## Stress tests

Six scenarios that cover both downside and upside risk. For each: impact on margin (in %), response plan, time-to-recover.

| # | Scenario | Margin impact | Response plan | Time-to-recover |
|---|---|---|---|---|
| 1 | **Vendor pricing 2–5× overnight** | | Switch to alternate provider (per kill switch) | hours / days |
| 2 | **Heaviest 10% of users doubles usage** | | Tighten cascade rules; rate-limit; introduce usage-tier pricing | weeks |
| 3 | **Cascade ratio inverts** *(more queries escalate than expected)* | | Investigate why workhorse is failing; tune Critic threshold | days |
| 4 | **Provider deprecates a model** | | Eval suite + migration playbook (kill switch); cost can change in either direction | weeks |
| 5 | **Sovereignty / regulatory constraint** *(forces swap to more expensive sovereign provider)* | | Cascade harder; price increase; or accept margin compression as cost of doing business in that market | months |
| 6 | **Adoption 10× in 12 months** | | Fixed costs amortize; variable costs scale linearly; need to renegotiate vendor pricing at volume | months |

---

## Before / After Lab Exercise

The course deliverable: explicit comparison of unit economics in the *traditional SaaS* mode versus the *AI-powered* mode. For greenfield AI-native products with no real "Before," use a **counterfactual Before** — *"if this had been built as traditional SaaS, the economics would look like..."* — useful for board narrative and for naming which parts of AI economics are advantages vs liabilities.

### Before — Traditional SaaS *(or counterfactual)*

- **Revenue formula:** `$X / seat × N seats = $monthly_revenue`
- **Fixed COGS:** `$X (infra, support, hosting)`
- **Variable COGS per seat:** ~`$0` *(each marginal seat is nearly pure margin)*
- **Gross margin:** typically **75–90%**
- **Implied dynamics:** each new seat ≈ pure profit once acquired

### After — AI-Powered

- **Revenue formula:** `$A base + $B × units` *(define the unit)*
- **Fixed COGS:** `$X (infra, vendor minimums, observability, compliance)`
- **Variable COGS per unit:** `$Y (blended cost per task — from cascading section)`
- **Gross margin:** typically **40–75%** *(highly dependent on cascade discipline and pricing model)*
- **Implied dynamics:** each marginal user carries marginal cost; power-user concentration is a margin risk

### Net margin shift

- **Δ margin %:** *(After − Before)*
- **Δ gross $:** *(at projected volume)*
- **Narrative (3 sentences max):** *"Why margin % moves [direction]. Where gross $ wins despite margin %. The hedge we're building [usage tiering / cost cap / NRR offset / etc.]."*

---

## Honest read — when there is no real "Before"

For AI-native products that never had a traditional SaaS version, the Before column is *synthetic* — *"if a comparable product had been built as traditional SaaS, it would look like X."* Useful for board narrative *("here's why AI changes the unit economics for this category")*, but be honest in the narrative: **the comparison is counterfactual, not historical.** Don't fake a traditional-SaaS baseline that never existed.

Some products (civic, cooperative, public-good, mission-driven) also don't have a *commercial* SaaS analog because they were never profit-maximizing. For those, the Before/After comparison can still be useful as an economic-design exercise — but the strategic question shifts from *"how do we preserve margin?"* to *"how do we sustainably cover cost?"* See `pricing-strategy.md` — the **Cost-Recovery / Sustainable** posture is built for exactly this case.

---

## Open questions

- 
- 

---

## Review log

| Date | Reviewer | What changed |
|---|---|---|
| | | |
