# Compounding System

> **Purpose.** Check whether the loops you designed in [`02-the-moat/data-flywheel.md`](../02-the-moat/data-flywheel-template.md) are *actually* compounding in practice, or whether they're still aspirational. The flywheel doc is design intent; this doc is operational reality. The gap between the two is where vulnerability lives.
>
> **The structural insight this doc serves.** A loop that doesn't compound is just throughput. Bigger pipes with the same shape don't make the product better — they just make it more popular. *Compounding* means the system gets meaningfully smarter as it runs. Most "moat" claims are throughput claims dressed up.
>
> **The freeze test.** *Freeze the product for 12 months — no shipping, no model updates, no curation, no community contributions. Are you still winning?* If yes, you're not compounding — your advantage is positional, not earned. That can be fine (it was M1's diagnostic on Bünzli) but it has to be honest.
>
> **How to use it.** For each loop, score it honestly: is it *active* (running and producing measurable improvement), *broken* (designed but not delivering), or *missing* (planned but not built). Don't fudge. Broken loops are gifts — they tell you exactly what to fix. Missing loops are roadmap items, not strategy claims.

---

## Connection to other modules

- **`02-the-moat/data-flywheel.md`** — the *design* of the loops lives there. This doc is the *governance check* on whether they're actually running. The flywheel doc says "this is the moat we're building"; this doc says "here's whether we've built it yet."
- **`04-the-contract/eval-strategy.md`** — the user-correction feedback loop (M4) is the operational engine that powers the Recursive Learning loop here. Without M4's closed-loop, Recursive Learning stays missing.
- **`01-the-bet/diagnostic.md` — Axis 1 (Workflow) + Axis 2 (Data)** — both axes presume the loops are compounding. If they're not, both axis scores need a downward correction at the next quarterly review.

---

## The three compounding loops

| Loop | What it does | What "compounding" means here |
|---|---|---|
| **Recursive Learning** | Users interact → system captures signal → system improves → users see better outputs | Each correction makes the next answer better. Measurable: per-query quality on the eval set improves week-over-week even without code changes. |
| **Cross-Domain Transfer** | Signal from one domain (e.g. civic queries) improves outputs in adjacent domains (e.g. commercial bookings) | A correction filed in domain A demonstrably improves answer quality in domain B. Measurable: eval delta on B when only A receives new training/correction signal. |
| **Network Intelligence** | Each new user makes the product better for every existing user | The 1,001st user makes the product measurably better for users 1-1,000. Measurable: per-user quality / engagement improves with N, holding other factors constant. |

Each maps roughly to one of M2's four flywheel loops; see the fill for the mapping.

---

## Loop status — the honest read

For each of the three loops, mark **active / broken / missing** with definitions you cannot fudge:

| Status | Definition | What it looks like in practice |
|---|---|---|
| **Active** | The loop is shipped, running, and producing measurable compounding *that you can show on a chart over time* | Per-query quality, user retention, or cross-domain transfer metric is trending up week-over-week without code changes |
| **Broken** | The loop was designed and built, but isn't producing the compounding effect | Loop runs (data flows) but the metric it's supposed to improve isn't moving; usually an instrumentation, prompt, or product-fit issue |
| **Missing** | The loop is planned but not built | The component or feature that powers the loop hasn't shipped yet |

**Don't conflate "we have a roadmap item" with "the loop is active."** A loop being on the roadmap means it's missing. A loop existing in code but with no measured improvement means it's broken.

---

## Loop scoring table

| Loop | Input | Output | Compounds? | Status | Evidence |
|---|---|---|---|---|---|
| Recursive Learning | *(what triggers the loop)* | *(what improves)* | Y / N | active / broken / missing | *(metric + measured delta, or "no measurement yet")* |
| Cross-Domain Transfer | | | Y / N | active / broken / missing | |
| Network Intelligence | | | Y / N | active / broken / missing | |

---

## Broken loop diagnosis

For any loop marked **broken** (data flowing but no improvement showing):

- **Hypothesis on why:** *(instrumentation gap / wrong metric / not enough volume / product-fit issue / model-capability ceiling)*
- **Fix plan:** *(specific work to unbreak it, with timeline)*
- **Re-test criterion:** *(what would prove the loop is now compounding)*

For any loop marked **missing**:

- **Why it isn't built yet:** *(prioritisation / dependency / capability gap)*
- **Planned ship date:** *(specific quarter, not "soon")*
- **What changes when it ships:** *(which axis on the diagnostic moves)*

---

## Context connectivity

*Where does knowledge silo, and what's the cost?* Compounding loops don't compound if the signal is trapped in silos that don't talk to each other.

| Silo | Where it lives | Why it matters | Plan to bridge |
|---|---|---|---|
| | | | |

Examples of common AI-product knowledge silos:
- User-correction signal trapped in support tools, never reaching the eval set or the KB
- Domain-expert content trapped in spreadsheets / Notion / SharePoint, never indexed
- Per-team prompt templates that diverge, with no central source of truth
- Telemetry that gets aggregated for ops but not fed back to model evaluation

---

## The freeze test

*If you froze the product for 12 months — no shipping, no model updates, no curation, no community contributions — are you still winning?*

| Answer | What it means |
|---|---|
| **Yes, we'd still be winning** | Your advantage is *positional*, not *compounding*. That can be fine, but call it honestly. Don't claim a flywheel moat. |
| **No, we'd be losing ground** | Your advantage *is* compounding. The work shipping in the next 12 months matters. |
| **Yes but at degraded quality** | You have compounding loops but they're slow. The 12-month freeze underestimates how much your competitors are compounding. |

Bünzli's honest answer is in the fill. It's worth knowing yours.

---

## Open questions

- 
- 

---

## Review log

| Date | Reviewer | What changed |
|---|---|---|
| | | |
