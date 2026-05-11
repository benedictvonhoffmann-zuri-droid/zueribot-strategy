# Pricing Strategy

> **Purpose.** Decide how the product captures value. Pricing is the bridge between the *cost curve* (the floor of what we can charge) and *user willingness-to-pay* (the ceiling). The gap between those two — minus the discount needed to win adoption — is your margin.
>
> **Pricing is a strategic choice, not a marketing afterthought.** The pricing posture you adopt (how aggressively, in which direction) and the charging model you pick (what unit you sell) shape the product roadmap, the customer segment you attract, and the competitive position you can hold. Most AI products under-invest in this decision and end up with seat-based pricing that under-monetises power users and overprices casual ones.
>
> **How to use it.** Pick a pricing posture first (strategic intent), then a charging model that operationalises it, then define the unit you actually sell. Re-evaluate when the cost curve shifts materially or when usage patterns surprise you.

---

## Connection to other modules

- **`cost-curve.md`** — the floor. You cannot price below cost long-term without an explicit subsidy (investor capital, grant funding, B2G procurement). The cost curve tells you what that floor is at each scale tier.
- **Diagnostic Axis 2 (Data).** A real data flywheel means usage-based or outcome-based pricing can compound — users pay more as they use more, and the product gets better as they use more. Without that loop, usage-based pricing is just metered access.
- **Diagnostic Axis 1 (Workflow).** Workflow embedment is what makes outcome-based pricing defensible. If you're billing per resolved task, you need the workflow integration that lets you *prove* the resolution.

---

## Pricing posture — the strategic intent

Four postures. Pick one. Each implies a different product roadmap, different customer segment, different competitive position.

| Posture | Intent | When to use | When *not* to use | Example |
|---|---|---|---|---|
| **Skim** | Launch premium, lower over time. Signal quality. Capture early-adopter willingness-to-pay before commoditisation. | Differentiated product with high perceived value; customer segment that pays for quality; you can credibly defend the premium with the product itself. | Commodity market; price-sensitive segment; can't defend premium positioning. | Apple, Porsche, Harvey AI |
| **Penetrate** | Price low, win volume, optimise cost relentlessly. Capture market share before competitors get scale. | Marginal cost falls fast with volume (scale economies); winner-takes-most dynamics; you have capital to fund the loss-leader phase. | Differentiated product where price is part of the perceived value; markets where margin can't recover later. | Amazon, AWS, OpenAI ChatGPT free tier |
| **Maximize** | Neither extreme. Optimise revenue at the current competitive position. Tune as you learn. | Mature category; established competitive position; no urgent need to skim or penetrate. | Early product / unclear positioning — defaulting to Maximize often means "we haven't thought about it." | Microsoft, HubSpot, most enterprise SaaS |
| **Cost-Recovery / Sustainable** | Price ≈ cost + minimal infrastructure margin. Profit is not the objective; sustainability is. | Mission-driven products (civic infrastructure, cooperatives, NGOs, public goods); markets where rent-seeking would compromise trust or political legitimacy. | Any context where investors expect margin expansion; markets that won't sustain a non-profit-maximizing operator long-term. | Wikipedia, Signal, Mozilla, certain civic-tech products |

> **Note on Cost-Recovery / Sustainable.** This posture is not in most strategy textbooks because mainstream pricing literature assumes profit maximization. It is genuinely how a meaningful slice of internet infrastructure prices itself, and it is the right posture for some products. Trade-off: easier to defend ethically and politically (no rent-seeking); harder to raise growth capital from investors who want margin expansion.

---

## Charging model — the structural choice

Three models. Pick one (or hybrid). The charging model determines how cost and revenue scale relative to each other, and what kind of buyer you attract.

| Charging model | Example | Margin profile | When it works | When it breaks |
|---|---|---|---|---|
| **Seat / access** *(per user, flat monthly)* | Slack, Notion, Figma | Compresses with usage growth (power users cost more, pay the same) | Predictable workflow tool; low per-user usage variance; the user *is* the unit of value. | High-variance usage; power users will lose money; LLM-heavy products almost always wrong fit. |
| **Hybrid (base + usage)** | HubSpot AI, Canva Pro, Grammarly | Base fee covers fixed costs; usage fee covers variable costs; scales cleanly with usage. | Platform-like products with mixed usage profile; buyer comfortable with predictable + variable bill. | Hard to communicate ("what will my bill look like?"); requires usage cap or alerting to avoid bill shock. |
| **Outcome / resolution** *(pay when the AI delivers the result)* | Intercom Fin (per resolved ticket), Harvey (per matter) | Best margin when attribution is clean — you charge for value, not for compute. | Quantifiable outcome the user values; clean attribution (you can prove the AI did it); customer trust to honor the bill. | Fuzzy outcomes; attribution disputes; legal/regulated services where pricing-by-result is unethical or non-compliant. |

### Unit definition — the most under-thought decision in AI pricing

**Name the customer outcome, not the internal model action.** A unit you bill on should describe what the customer values, not what the system does.

| ❌ Bad unit definition | ✓ Good unit definition |
|---|---|
| Per LLM call | Per resolved support ticket |
| Per token | Per drafted document |
| Per inference | Per civic report filed |
| Per workflow run | Per booking confirmed |

The bad versions are easy to bill on but communicate nothing to the buyer. The good versions are harder to instrument but they're what the user actually buys. **If you can't name your unit in terms the customer would use in a sentence about what they got, the unit is wrong.**

---

## Pricing calculation template

For whatever posture + model you picked, work out the numbers:

| | Value |
|---|---|
| **Base fee** | $ / month |
| **Price per unit** | $ |
| **Unit definition** | *(in customer-outcome language — see above)* |
| **Estimated units per user / month** | |
| **Implied revenue per user / month** | (base + price × units) |
| **Cost per user / month** *(from cost-curve.md)* | |
| **Gross margin per user / month** | (revenue − cost) / revenue |
| **Break-even users / month** | (fixed cost) / (revenue per user − variable cost per user) |

---

## Segment analysis

One price for all users is usually wrong. Most products end up serving at least two segments with materially different willingness-to-pay and usage profile.

**Common pattern:** *casual* users (low usage, low willingness-to-pay) and *power* users (high usage, high willingness-to-pay). Per-seat pricing under-monetises power users (they would pay more if asked) and overprices casual users (they don't use enough to feel value).

| Segment | Typical usage | Willingness-to-pay | Pricing implication |
|---|---|---|---|
| *Casual / discovery* | Low | Low | Freemium tier or free with cap |
| *Regular* | Medium | Medium | Subscription at sweet-spot price |
| *Power / professional* | High | High | Usage-based tier or pro plan with higher cap |
| *Enterprise / B2B* | Very high | Highest | Custom contracts; outcome-based or seat-based depending on workflow |

For each segment, name: *who they are*, *what they value*, *what they'd pay*, *what charging model fits them*.

---

## Honest read — when monetization is deferred or unclear

For products that are not yet monetized (pre-launch, free until product-market fit, mission-driven without commercial model), the temptation is to leave this doc empty. Don't. Even *"we are deliberately not charging at v1 because…"* is a strategic choice that deserves articulation.

A few cases this section helps:

- **Pre-PMF / consumer:** *"v1 is free to build trust and adoption; monetization decided at [milestone]. Plausible paths: A, B, C."* Forces you to name the options before you're under launch pressure to pick one.
- **Mission-driven / cost-recovery:** *"We price to cover cost + minimal margin for sustainability. The right number is whatever covers cost at projected sustainable scale."* Use the Cost-Recovery posture above.
- **B2G / public-sector procurement:** *"Pricing is not retail; it's procurement. The number doesn't come from this doc — it comes from the RFP response."* Don't pretend a list price exists when it doesn't.
- **Platform / partnership revenue:** *"We don't charge the user; we charge the partner for access to the user (revenue-share, affiliate fees, etc.)."* The unit moves from per-user to per-transaction or per-partnership.

In all cases: name *why* monetization is deferred, name *when* it will be revisited, name *what options are on the table*. The honesty itself is the strategic discipline.

---

## Open questions

- 
- 

---

## Review log

| Date | Reviewer | What changed |
|---|---|---|
| | | |
