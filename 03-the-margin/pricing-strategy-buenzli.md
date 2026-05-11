# Pricing Strategy — Bünzli

> Filled instance of [`pricing-strategy-template.md`](pricing-strategy-template.md). Snapshot dated 2026-05-02.
>
> **Headline.** Bünzli's pricing posture is **Cost-Recovery / Sustainable** — price ≈ cost + minimal infrastructure margin. The charging model at v1 is **deferred — free at launch, with the v2 default being usage-based per query**. If usage-based proves friction-heavy, the fallback is a **flat subscription at CHF 5.60 / month — the price of a Zürich Tagesticket** (chosen as a hyperlocal pricing anchor that is instantly legible to every Zürich resident). Three monetization paths sit in parallel: B2G procurement (Stadt Zürich), Freemium consumer (the default), Partnership revenue (Quandoo-style commissions). At v1 we don't pick one; we ship free, watch usage, decide at open beta.

---

## Connection to other modules

- **`cost-curve-buenzli.md`** — the floor. Blended variable cost per query is **~CHF 0.0045**; fixed cost is **CHF 148/month**. At 10,000 queries/month, all-in cost-per-query is ~CHF 0.019 and cost-per-active-user (assuming ~10 queries / user / month) is ~CHF 0.19. Any price has to live above those numbers at the chosen scale.
- **Diagnostic Axis 2 (Data).** Bünzli's flywheel is community correction + civic outcomes, not paid-tier consumption. The data advantage compounds *regardless* of pricing — which is why Cost-Recovery posture is viable: we don't need monetization-driven growth to build the moat.
- **Diagnostic Axis 1 (Workflow) + Attacker D (Stadt Zürich).** B2G procurement is the strongest path to commercial sustainability. If Stadt Zürich procures Bünzli as civic infrastructure, the pricing question shifts from B2C willingness-to-pay to municipal procurement, which is an entirely different conversation. The partner-don't-compete strategic move (Module 1) becomes the pricing strategy.

---

## Pricing posture

**Selected: Cost-Recovery / Sustainable.**

Bünzli is a civic infrastructure product, not a profit-maximizing B2B SaaS. The sovereignty + privacy posture (no PII storage, Swiss-hosted, community-correction-driven) is incompatible with rent-seeking pricing — you cannot credibly tell users *"your data stays in Switzerland and we don't track you"* while simultaneously extracting maximum willingness-to-pay. The two positionings contradict each other.

Cost-Recovery is the correct posture for what Bünzli actually is. It is also the posture that keeps doors open to B2G procurement (Stadt Zürich won't procure from a rent-seeking commercial provider) and partnership revenue (Swiss services partner with peers, not exploiters).

**What we explicitly reject:**
- **Skim** — Bünzli is not a premium-positioned product; it's civic infrastructure. Skim pricing contradicts the mission.
- **Penetrate** — There is no winner-takes-most race to win in Zürich civic-AI. Penetrate pricing requires capital and a competitive landscape that doesn't apply.
- **Maximize** — Defaulting to Maximize would mean *"we haven't thought about it,"* which would be dishonest. We have thought about it; Cost-Recovery is the answer.

---

## Charging model

**Selected at v1: None (free).** Monetization is deferred to open beta.

**Selected default at v2 (consumer freemium path): Usage-based per query, with a free tier.**

| Tier | Price | What you get |
|---|---|---|
| **Free** | CHF 0 | Up to N queries / month *(N to be set; likely 30–50, enough for genuine civic use without enabling paid-tier displacement)* |
| **Pay-per-query** | CHF 0.05 / query | Beyond the free cap |

**Fallback charging model if usage-based proves friction-heavy: Flat subscription at CHF 5.60 / month — the Tagesticket anchor.**

| Tier | Price | What you get |
|---|---|---|
| **Free** | CHF 0 | Up to N queries / month |
| **Unlimited (Tagesticket)** | CHF 5.60 / month | Unlimited queries within fair-use |

The Tagesticket anchor is deliberate: every Zürich resident knows what a single-zone day pass costs, and pricing Bünzli at that level signals *"this is a small civic utility you spend day-pass money on, not a SaaS subscription you have to justify."* It's also the right number for sustainability — at 10,000 monthly active users, CHF 5.60 × ~10% conversion to paid = CHF 5,600/month, which covers fixed cost (CHF 148) + variable cost (CHF 450 at 100k queries) with healthy buffer for growth.

### Unit definition (good, per the template's principle)

- ❌ Bad: *"Per LLM call"* / *"Per Apertus inference"*
- ✓ Good: *"Per Zürich question answered"* — the customer outcome, named in customer language

The unit Bünzli charges (in the usage-based path) is *one Zürich question answered* — the user value, not the model action. Internally that may translate to 2–5 LLM calls; that's our problem, not the user's.

---

## Pricing calculation — usage-based default

| | Value |
|---|---|
| Base fee | CHF 0 / month |
| Price per unit | CHF 0.05 / query (above free tier) |
| Unit definition | Per Zürich question answered |
| Estimated units per paid user / month | ~30 queries *(power-user assumption)* |
| Implied revenue per paid user / month | CHF 1.50 |
| Cost per user / month *(at moderate scale)* | ~CHF 0.20 *(from cost curve)* |
| Gross margin per paid user / month | ~87% |
| Break-even active users at CHF 148 fixed | ~100 paid users at 30q/month |

## Pricing calculation — Tagesticket fallback

| | Value |
|---|---|
| Base fee | CHF 5.60 / month *(unlimited within fair use)* |
| Price per unit | n/a (flat) |
| Unit definition | Unlimited Zürich questions / month |
| Estimated units per paid user / month | ~50 queries |
| Implied revenue per paid user / month | CHF 5.60 |
| Cost per paid user / month *(at moderate scale, including fair-use heads)* | ~CHF 0.30 |
| Gross margin per paid user / month | ~95% |
| Break-even active paid users at CHF 148 fixed | ~30 paid users |

Both paths are economically viable at modest scale. The choice between them at v2 is a **UX call, not an economics call** — which feels less like friction for casual Zürich residents who just want an answer to *"wenn isch d'Ghüderabfuhr?"*

---

## Three monetization paths — all open at v1

### (a) B2G procurement *(Stadt or Kanton Zürich)*

The strategic preferred path per Module 1's *partner-don't-compete* response to Attacker D.

- Stadt Zürich procures Bünzli as civic infrastructure (think *"we are the city's official AI assistant"*).
- Pricing model: not retail — comes from RFP response. Plausibly a multi-year contract at CHF 50–200k / year covering operating cost + sustainability margin.
- Pros: full sustainability, official legitimacy, defensive moat against Attacker D *(the city won't fund a competing in-house assistant if it's already using ours)*.
- Cons: long procurement cycle (12–24 months); requires civic relationship-building; tied to political cycles.

### (b) Freemium consumer *(usage-based or Tagesticket subscription)*

The default path described above. Most accessible; lowest dependency on external relationships.

- Pros: under user control, accessible, no procurement gatekeeper.
- Cons: requires consumer marketing; conversion rates on freemium products are notoriously hard to predict; CHF 5.60 / month is not a category-defining price point that drives word-of-mouth.

### (c) Partnership revenue *(Quandoo-style commissions on agentic actions)*

Where Bünzli triggers a transaction on a partner platform (a Quandoo booking, a Migros loyalty action, a ZVV ticket), the partner shares revenue.

- Pros: pricing is invisible to the user (they pay nothing extra); aligns Bünzli with partner success.
- Cons: dependent on partner willingness to share; introduces commercial conflicts of interest (do we surface the partner with the best UX, or the highest commission?); doesn't cover queries that don't trigger transactions.

### v1 strategy: all three open, none committed

At v1, Bünzli ships **free**, accumulates **6–12 months of usage data**, and revisits the monetization decision **at open beta**. The choice between paths (a), (b), (c) is determined by which path is showing the most traction by then. *Defer with discipline, decide with data.*

---

## Segment analysis

Bünzli's user base will likely split into three patterns:

| Segment | Usage | Willingness-to-pay | Implication |
|---|---|---|---|
| **Casual visitors** *(occasional, "what's open this weekend"-type queries)* | Low (1–5 queries / month) | Low | Free tier serves them; not where monetization lives |
| **Engaged residents** *(weekly use across civic + commercial domains)* | Medium (10–30 queries / month) | Moderate (Tagesticket range) | The Tagesticket sweet spot; the bulk of paid conversion if subscription is chosen |
| **Power users** *(daily use, multiple agentic actions / month)* | High (50+ queries / month) | Higher *(but still consumer-anchored)* | Usage-based may overcharge them at CHF 0.05/query; subscription is better for this segment |

The implication for pricing: **subscription (Tagesticket) is more user-friendly across all three segments**; usage-based has cleaner economics but may friction-out casual users who'd otherwise convert. Default to subscription unless usage data shows the median user pays significantly less than CHF 5.60 worth of queries — in which case usage-based is fairer.

---

## Honest read

Pricing for Bünzli is genuinely deferred — not in a "we'll figure it out later" way, but in a structured *"we don't have the user data yet to make this decision, and faking the decision now would be worse than naming the deferral honestly"* way. What we *do* know:

1. **Cost-Recovery is the only posture compatible with the sovereignty + privacy + civic positioning.** Anything else contradicts the rest of the strategy.
2. **B2G procurement is the strongest path if it lands**, because it converts the partner-don't-compete strategic move into the pricing strategy. It's also the slowest and most relationship-dependent. We can't make it happen on a v1 timeline.
3. **Freemium consumer with Tagesticket anchor is the v2 default** because (a) it's under our control, (b) the price point is hyperlocal-credible, (c) the math works at moderate scale.
4. **Partnership revenue is a real complement, not a primary path.** Quandoo commissions (if they exist) bolt on cleanly to (b); they don't replace it.

The strategic discipline is to ship v1 free, instrument usage carefully, and revisit at open beta with a year of data. That is the strategy, not the absence of one.

---

## Open questions

1. **Free-tier cap** — 30 / 50 / 100 queries per month? Needs an honest read of typical civic-query volume (TBD until production logs exist).
2. **Quandoo commission structure** — open question from `cost-curve-buenzli.md`. If commissions exist, partnership revenue path becomes more concrete.
3. **B2G procurement timeline** — what's the realistic earliest a Stadt Zürich pilot could close? Likely 12–18 months given Swiss public-sector procurement cycles.
4. **EU AI Act implications for free-tier provision** — does providing AI service for free in EU territory trigger any documentation obligations that wouldn't apply to paid service?
5. **Fair-use definition for Tagesticket subscription** — at what query volume per month does a "Tagesticket" subscriber become economically unsustainable?

---

## Review log

| Date | Reviewer | What changed |
|---|---|---|
| 2026-05-02 | Benedict + Claude (Opus 4.7) | Initial fill. Cost-Recovery / Sustainable posture selected. v1 = free, v2 default = usage-based per query (CHF 0.05) with Tagesticket subscription fallback at CHF 5.60/month as a hyperlocal pricing anchor. Three monetization paths (B2G procurement, freemium consumer, partnership revenue) all named, all open at v1, decision deferred to open beta. |
