# Roadmap — Three Horizons (AI-compressed)

> **Purpose.** A roadmap an AI product can actually run against. Three Horizons sized for AI cadence (weeks/months, not quarters/years), with per-horizon confidence bands and a small set of AI-specific board metrics that make "is the product getting better" legible to non-technical stakeholders.
>
> **The structural insight this doc serves.** Pre-AI roadmaps optimized for predictability over a long horizon. AI roadmaps optimize for *learning velocity* over a short one. The model under the hood changes every quarter; the eval set grows weekly; user behaviour shifts as soon as the product gets good enough to use seriously. A 12-month plan written in month 1 is fiction by month 3. The fix is not "stop planning" — it's "plan in three horizons sized to the speed the product actually moves at."
>
> **How to use it.** Fill in honestly. **One horizon only = incomplete roadmap** (no Now = no shipping; no Next = no validation; no Explore = no future). Re-review monthly, not quarterly — that is the AI-native cadence.

---

## Connection to other modules

- **`01-the-bet/diagnostic.md`** — Horizon 3 (Explore) should attack the worst axis from the diagnostic. If your weakest axis is "Moat = 2" and H3 doesn't include a compounding experiment, the roadmap isn't matching the bet.
- **`02-the-moat/data-flywheel.md`** + **`05-the-guardrails/compounding-system.md`** — H1 ships the flywheel; H2 validates that it's actually compounding. Without H1, no data. Without H2, no proof.
- **`03-the-margin/cost-curve.md` + `pricing-strategy.md`** — Inference ROI metric below ties to cost-curve assumptions. If unit economics shift, H2 monetization initiatives shift.
- **`04-the-contract/eval-strategy.md`** — Hallucination Rate, Drift Velocity, Eval Regression all defined here. Board metrics are the eval contract surfaced one layer up.
- **`05-the-guardrails/governance-policy.md`** — HITL Rate target here = autonomy boundary commitment there. If they disagree, one is wrong.

---

## The three horizons — anchored rubric

| Horizon | Timeframe | Confidence | What lives here |
|---|---|---|---|
| **H1 — Ship** | 0–4 weeks | High | Execute. Already-decided work, known-good capabilities, in-flight commitments. If H1 is uncertain, you're not shipping — you're still validating. |
| **H2 — Validate** | 1–3 months | Medium | Bets with explicit kill criteria. "If metric X doesn't move by date Y, we kill this." No kill criteria = not H2, it's a wish. |
| **H3 — Explore** | 3–6 months | Low | Experiments. Cheap, parallel, designed to *learn* not deliver. Output is a decision, not a feature. |

**The principle.** AI compresses every horizon. What used to be "Now / Next / Later = quarter / year / multi-year" is now "weeks / quarter / two quarters." The structure is the same; the clock is faster.

---

## H1 — Ship (0–4 weeks)

*Quick wins. Already-decided. Execution risk only.*

| Initiative | Metric (what proves it shipped) | Owner | Confidence |
|---|---|---|---|
| | | | H |
| | | | H |

---

## H2 — Validate (1–3 months)

*Bets with kill criteria. Each row must name the metric that decides keep/kill.*

| Initiative | Hypothesis being tested | Kill criterion | Owner | Confidence |
|---|---|---|---|---|
| | | | | M |
| | | | | M |

---

## H3 — Explore (3–6 months)

*Cheap experiments. Designed to learn, not to deliver. Output = decision.*

| Initiative | Question being answered | Cost to run | Owner | Confidence |
|---|---|---|---|---|
| | | | | L |

---

## AI-specific board metrics

The six metrics non-technical stakeholders need to read the product's health. Each one ties to a doc that defines it; the board sees the headline, the operator reads the source.

| Metric | What it measures | Why it matters at board level | Source |
|---|---|---|---|
| **Hallucination Rate** | % of answers with at least one unsupported claim, measured on the eval set | The single most damaging failure mode for trust. Trends here precede churn. | `04-the-contract/eval-strategy.md` |
| **Drift Velocity** | Rate at which production behaviour diverges from the eval baseline (e.g. week-over-week regression on golden set) | Tells you whether the model is silently degrading. A flat eval but rising drift = trouble. | `04-the-contract/eval-strategy.md` |
| **Confidence Distribution** | Distribution of agent self-reported confidence buckets (high / medium / refuse) on real traffic | If "high confidence" share is growing while Hallucination Rate is flat or rising, calibration is broken. | `04-the-contract/eval-strategy.md` |
| **HITL Rate** | % of agentic actions that required human-in-the-loop confirmation (and % that the human overrode) | Trade-off knob between safety and autonomy. Trends here are the governance contract made visible. | `05-the-guardrails/governance-policy.md` |
| **Inference ROI** | Revenue (or value proxy) per CHF of inference + infra spend | Whether the unit economics work *at current model choice and routing*. Tracks pricing-strategy assumptions. | `03-the-margin/cost-curve.md` |
| **Eval Regression** | # of eval-set rows that regressed since last release | Did the last shipped change make something worse? Catches silent breakage that aggregate metrics hide. | `04-the-contract/eval-strategy.md` |

**Reporting cadence.** Monthly to board / leadership. Weekly internally. Anything in a red zone triggers an out-of-cycle review.

---

## Monthly review cadence

| Review | What gets re-examined | Owner |
|---|---|---|
| **Horizon hygiene** — is each row still in the right horizon? | Pull H2 → H1 anything de-risked; demote H1 → H2 anything that lost confidence | PM |
| **Kill-criterion check** — any H2 row past its kill date? | Either kill it, or write down why the criterion is being extended (once) | PM |
| **Metric trends** — six board metrics vs last month | Anything red → root-cause; anything yellow → watchlist | PM + Eng |
| **Eval set additions** — what got added in the last 4 weeks? | Confirm closed-loop is working; if eval set isn't growing, the correction primitive is broken | PM + Eng |

---

## Open questions

- 
- 

---

## Review log

| Date | Reviewer | What changed |
|---|---|---|
| | | |
