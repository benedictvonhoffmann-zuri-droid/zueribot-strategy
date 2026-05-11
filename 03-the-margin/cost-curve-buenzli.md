# Cost Curve — Bünzli

> Filled instance of [`cost-curve-template.md`](cost-curve-template.md). Snapshot dated 2026-05-02.
>
> **Headline.** Bünzli's fixed cost is **CHF 148/month** (GPU + VM). Blended variable cost is dominated by **Apertus output tokens at CHF 2.50 per 1M** — about 75% of per-query cost. The sovereignty premium is *not* "the whole stack is more expensive than US cloud" — Mistral's Ministral-3-14B is priced at-market. The premium lives specifically in Apertus output. That's the cost line that defines Bünzli's unit economics, and that's where cascade discipline matters most.

---

## Timeframe

Operational 12–24 months. Pre-launch as of writing — numbers below are projections, to be reverified against production logs once shipped.

---

## Connection to other modules

- **Diagnostic Axis 2 (Data).** Our flywheel doesn't compound through paid tier monetization — Bünzli's data flywheel runs on community correction + civic outcomes, not on premium subscriptions. That decouples the cost curve from the value curve in ways most B2B SaaS products don't experience.
- **Kill Switch.** The biggest cost-curve risk is **Apertus terms changing** (price increase, license restriction, model deprecation). The kill switch needs to support a graceful degradation path — *"if Apertus becomes uneconomic, voice degrades to Mistral-only with a quality regression we're honest about."* Not graceful but not catastrophic.
- **Flywheel.** Cascade ratio improves as we learn which questions need Apertus voice vs. which can be served by Mistral structured output. Today's cascade ratio is a guess; in 6 months it's a measurement.

---

## AI cost stack

| Cost line | Type | Unit cost | Notes |
|---|---|---|---|
| **GPU** *(embeddings + reranker + Qdrant)* | **Fixed** | CHF 123 / month | Self-hosted; covers KB infrastructure |
| **VM** *(FastAPI backend, orchestration)* | **Fixed** | CHF 25 / month *(conservative; actual ~CHF 11)* | Conservative buffer for upgrades |
| Mistral Ministral-3-14B — input | Variable | CHF 0.30 / 1M tokens | Tool calls, structured operations |
| Mistral Ministral-3-14B — output | Variable | CHF 0.40 / 1M tokens | |
| **Apertus-70B — input** | Variable | **CHF 0.70 / 1M tokens** | Voice generation, sovereignty premium |
| **Apertus-70B — output** | Variable | **CHF 2.50 / 1M tokens** | **The single biggest per-query line item.** 6.25× Mistral output cost. |
| Embeddings *(initial KB + incremental)* | Variable → effectively fixed | Free on owned GPU | Self-hosted; no marginal cost |
| HITL review | Variable | Not budgeted at v1 | Sample-rate Legal review is a v2 concern |
| Observability *(LangSmith / OTel)* | Fixed | Free tier at v1 | LangSmith free tier covers projected v1 traffic |
| Bandwidth / egress | Variable | Trivial | Text-only payloads |
| Quandoo commission | Variable | **TBD** | Open question — may be free, may be revenue-share |

**Fixed-cost subtotal:** **CHF 148 / month**

**Variable-cost coefficient (estimated, per query):**
- Mistral portion (in + out, across ~2-3 tool calls): ~CHF 0.001
- Apertus portion (in + out, voice generation): ~CHF 0.0034
- **Blended per query: ~CHF 0.0045**

Apertus output alone is ~CHF 0.00125 per query — **~28% of total variable cost from one token line.**

---

## Cascading strategy

The cost-curve consequence: **every interaction that requires Apertus is ~7× the cost of an equivalent Mistral-only interaction.** Cascade discipline matters more for Bünzli than for typical AI products.

| Tier | Model | Cost ratio | What it handles |
|---|---|---|---|
| **Triage** | Mistral Ministral-3-14B | 1× | Scope check, intent classification, tool routing, structured extraction, retrieval re-ranking |
| **Workhorse** | Mistral Ministral-3-14B | 1× | Tool-call orchestration, content structuring (anything not directly user-facing voice) |
| **Voice (Premium)** | Apertus-70B | ~7× | Swiss German voice generation, final answer composition — *only when the user is going to read it* |

**Routing rules:**
- Scope check, decomposition, retrieval-orchestration → **Mistral**
- Civic-action tool flows (Züri wie neu submission JSON, ERZ schedule lookup, Quandoo booking schema) → **Mistral**
- Final voice answer in Züritüütsch → **Apertus**
- Fallback: if Apertus is unavailable, voice degrades to Mistral with a quality regression flagged in the UI

**Expected cascade ratio (estimate, pre-launch):**
- 100% of queries → Mistral (always at least one tool call)
- ~80% of queries → Apertus (most queries terminate in a user-facing answer)
- ~20% of queries → Mistral-only (tool-action queries where the user gets a confirmation, not a paragraph)

**Blended cost per task (current best estimate):** **CHF 0.0045**
**Blended cost if cascade discipline tightens** *(Apertus share drops to 50% via better tool-only flows for short answers)*: **CHF 0.003** (33% reduction)

---

## Stress tests

| # | Scenario | Margin impact | Response | Time |
|---|---|---|---|---|
| 1 | **Apertus pricing 2× overnight** | Variable cost +50%; blended per query rises to ~CHF 0.007 | Tighten cascade harder (Apertus only for explicit voice queries); raise per-query price proportionally | weeks |
| 2 | **Heaviest 10% of users doubles usage** | Variable scales linearly; fixed unchanged → margin per user falls but total margin holds | Introduce usage tier / fair-use cap; freemium → paid tier nudge | weeks |
| 3 | **Cascade ratio inverts** *(more queries hit Apertus than expected, e.g. 95% instead of 80%)* | Blended cost rises ~15%; per query → ~CHF 0.0052 | Audit Apertus routing rules; deflect short answers to Mistral | days |
| 4 | **Apertus deprecated or licensed for non-commercial only** | **Existential.** Voice posture collapses or moves to a Mistral fine-tune (no sovereignty) | Fall back to Mistral voice; rebuild Apertus parity with self-hosted Llama + Swiss German LoRA *(see kill-switch)* | months |
| 5 | **Mistral pricing 2×** | Less severe than Apertus 2× — Mistral is ~25% of per-query cost | Tighten cascade; consider Apertus-only for some tool calls (counter-intuitively cheaper if Mistral surges) | weeks |
| 6 | **10× usage in 12 months** | Fixed amortizes to negligible per-query; variable scales linearly. Per-query cost approaches CHF 0.0045 (the floor); per-user cost falls if users are similar | Renegotiate Apertus pricing at volume; introduce premium tier; B2G procurement track *(Stadt Zürich)* becomes economically more interesting | months |

---

## Before / After Lab Exercise

### Before — Counterfactual Traditional SaaS

There is no real Before for Bünzli — it's an AI-native civic product. The Before column is *synthetic*: **"if a Swiss-hosted civic information service had been built as traditional B2C SaaS"** (think a paid version of an enriched local.ch with curated content).

- **Revenue formula:** `CHF 9 / user / month × N subscribers`
- **Fixed COGS:** ~CHF 100 / month (hosting + small CMS)
- **Variable COGS per user:** ~CHF 0.10 / month (storage, bandwidth)
- **Gross margin:** ~98%
- **Implied dynamics:** every new paid subscriber ≈ pure margin

### After — AI-Powered (Bünzli's actual model)

- **Revenue formula:** `CHF 0 base + CHF 0 × queries` *(at v1; monetization deferred to open beta)*
- **Fixed COGS:** **CHF 148 / month** (GPU + VM)
- **Variable COGS per query:** **~CHF 0.0045** (Mistral tools + Apertus voice, blended)
- **Gross margin at v1:** **−∞** *(no revenue, only cost — by design until open beta)*
- **Implied dynamics:** the cost-curve question for Bünzli isn't "how much margin do we capture per user" but "**how big does our user base have to be before sustainable monetization kicks in?**"

### Net margin shift narrative

*Bünzli isn't trading SaaS margin for AI margin — it's trading "high-margin info product" for "sovereign civic infrastructure that costs CHF ~5 per active user per month at moderate scale." Gross margin % is the wrong metric; cost-per-active-user at sustainable adoption is the right one. At 1,000 monthly active users, cost-per-user is CHF 0.15/month. At 10,000, it drops to CHF 0.02. The architectural question for sustainability is which scale tier Bünzli reaches before monetization decisions become forced.*

---

## Honest read — Bünzli's cost curve is real, not optimised

A few things to be honest about:

1. **Apertus output dominates variable cost.** Any "we'll just scale" plan has to either (a) tighten Apertus usage (which compromises the sovereign-voice premium), (b) negotiate Apertus pricing at volume, or (c) accept that Bünzli's per-query economics will always be ~7× a US-hyperscaler equivalent. None of those are bad — they're conscious choices. The cost curve is what those choices look like in numbers.
2. **Fixed cost is dominated by the GPU.** CHF 123/month for embeddings + reranker + Qdrant is the price of self-hosted sovereignty on Infomaniak. Could go to a managed Qdrant for less, but loses Swiss-residency on the vector store. Not recommended.
3. **HITL is unbudgeted.** v1 has no Legal review loop because Bünzli isn't a legal product. But if civic-AI moves into higher-stakes territory (legal advice, medical referrals — both *out of scope* per the spec), HITL costs would change the math materially.
4. **Quandoo commission is the open question that could shift this.** If Quandoo charges per-booking commission, that's a *revenue* line on the back of an *agentic action*. The cost-curve becomes commission-net rather than purely cost-side.

---

## Open questions

1. **Quandoo commission structure** — does it exist, what's the rate, who collects it?
2. **Apertus volume pricing** — is there a public volume discount, or does it require direct negotiation with the Swiss AI lab?
3. **Actual cascade ratio in production** — current estimate (80% Apertus) is a guess; first 1,000 production queries will tell us the real number.
4. **Bandwidth / egress** — text-only at v1, but if Bünzli adds image upload (e.g. photo with Züri wie neu reports), egress becomes material.
5. **Compliance cost at scale** — if Bünzli reaches the threshold for revDSG or EU AI Act documentation requirements, fixed costs add a meaningful new line.

---

## Review log

| Date | Reviewer | What changed |
|---|---|---|
| 2026-05-02 | Benedict + Claude (Opus 4.7) | Initial fill with real Bünzli costs. Apertus output named as the dominant variable-cost line. Cascade strategy + cost-recovery framing established. Synthesis narrative: gross margin % is the wrong metric for Bünzli; cost-per-active-user at sustainable adoption is the right one. |
