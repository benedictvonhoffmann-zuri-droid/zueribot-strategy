# Board Pitch — Bünzli

> Filled instance of [`board-pitch-template.md`](board-pitch-template.md). Snapshot dated 2026-05-18.
>
> **Audience.** Written for a future first conversation with a civic-tech sponsor at Stadt Zürich, a Swiss-sovereign-AI funder, or a strategic advisor. Today's "board" is Benedict alone; the doc is framed as if the audience were external because that is the constraint that forces the thinking to be honest.

---

## Connection to other modules

- M1 diagnostic (8/15 niche bet), M2 (Data + Workflow moats), M3 (Cost-Recovery posture, CHF 148/mo fixed cost), M4 (defence-in-depth eval contract), M5 (compounding gap honestly named), M6 roadmap (how the gap closes).

---

## Thesis

> **Bünzli is the sovereign, hyper-local civic AI assistant for Zürich — Swiss-hosted on Apertus + Ministral, eval-contracted against a Züri-specific golden set, governed under EU AI Act limited-risk + revDSG, and architected so that the moat is the corrections users leave behind, not the model under the hood.**

---

## The case

### 1. Why now

Three shifts converged in the last 12 months. **Apertus** (Swiss-hosted multilingual model) made sovereign inference economically viable for the first time — no US dependency, Swiss data residency, CHF-denominated cost curve. **Hyperscaler attention is elsewhere** — OpenAI and Google are not building Züritüütsch-tuned civic assistants for a 1.5M-person canton; we have a 24-month window before that calculus changes (M1 diagnostic, Axis 3 = 4/5 platform exposure). **EU AI Act enters enforcement** — limited-risk classification is now an explicit, navigable posture rather than a question mark, and Swiss revDSG gives Bünzli a sovereignty story that Brussels-hosted competitors cannot tell.

### 2. What's defensible

The moat is **not** the model — we are explicit that Apertus and Ministral are commodities we route to. The moat is **the Quartier-Wissen + correction-loop flywheel** (M2, M5): every user who flags "war das richtig?" feeds an eval-set row, every eval-set row sharpens future answers, and every sharpened answer is in *Züri-specific civic vocabulary that nobody else has invested in*. At month 1 the flywheel is aspirational (M5 freeze-test: "positional, not yet compounding"); H1 of the roadmap ships the primitive that turns position into compounding. **The honest read: the moat does not exist yet — H1 builds it. That is the bet.**

### 3. The economics

CHF 148/mo fixed infra (Infomaniak Swiss VPS, Apertus inference, Qdrant) — verified, not modelled (M3 cost-curve). Variable cost dominated by inference; Apertus voice + Ministral tools route is ~70% cheaper than Sonnet-everywhere. Pricing posture: **Cost-Recovery**, anchored to the Zürich Tagesticket (CHF 5.60/mo). Three monetization paths kept open — Tagesticket subscription, pay-per-action on agentic tools (Quandoo, Züri wie neu), B2G licensing to Stadt Zürich. **No revenue at v1 by design.** Open-beta monetization decision lives in H3.

---

## The risks

1. **The correction loop fails to compound.** If after 12 weeks of "war das richtig?" being live the eval-set additions don't produce measurable week-over-week improvement, the M5 finding becomes structural — Bünzli is a defensible position, not a compounding product. *Mitigation:* H2 kill criterion is explicit (≥5% WoW after 200 valid flags), and the diagnostic Axes 1+2 are re-scored downward if it doesn't fire. We will not paper over it.

2. **Sovereignty gap on the LLM-as-judge.** The eval contract uses Sonnet 4.7 today (M4) — the only Anthropic dependency left in the stack. *Mitigation:* H2 swaps to self-hosted Prometheus 2 on rent-a-pod with an explicit agreement-with-human threshold (≥85%). If the swap fails (<75% agreement), we name it as residual sovereignty gap in the Open Strategy page rather than hide it.

3. **B2G timeline mismatch.** Stadt Zürich procurement cycles run in years; Bünzli's runway runs in months. *Mitigation:* B2G lives in H3 as a *conversation*, not a deliverable — Bünzli does not depend on city revenue to exist. The civic assistant ships either way; B2G is upside, not lifeline.

---

## The ask

Two specific things, neither monetary.

1. **One warm introduction** to a civic-tech contact at Stadt Zürich (Statistik, OGD, or Smart City Lab) — to start the H3 procurement conversation without going through a cold front door.
2. **One sponsor for the open-beta launch announcement** — a Zürich-local voice (journalist, civic-tech figure, or politician) willing to be quoted at launch about why Swiss-sovereign civic AI matters. Reach, not money.

If a third item makes sense: **30 minutes of feedback on the Open Strategy page** before it goes live publicly — the page is the audit trail of the bet, and we'd rather know the holes before the internet finds them.

---

## M1 baseline vs. now

**M1 baseline (2026-04-27):**

> Bünzli is a Zürich-focused AI assistant built on a sovereign Swiss stack. The bet is that hyperscalers will not bother with Swiss-German Zürich for 24 months, and we will use that window to build a moat from hyper-local civic knowledge. Scored 8/15 on the niche-bet diagnostic — positional advantage real, compounding advantage to be earned.

**Now (2026-05-18, after M2–M6):**

> Bünzli is sovereign civic AI for Zürich — Apertus + Ministral inference, defence-in-depth eval contract (Researcher → Critic → Copywriter, with Sonnet-judge today and Prometheus-2 in H2), Cost-Recovery pricing anchored to the Tagesticket, EU AI Act limited-risk + revDSG governance, monetization deferred to open beta. The compounding loop is not yet running — the v1 advantage is positional, and H1 of the roadmap (correction primitive + property-based eval + confidence-UX) is what converts position into compounding. The Open Strategy page publishes this honestly, including the gaps.

**What sharpened:**

- **From "sovereign stack" to "named routing decision"** — Apertus voice + Ministral tools, with Sonnet as the only remaining external dependency (named, scheduled for replacement in H2).
- **From "moat = hyper-local knowledge" to "moat = correction loop + Quartier-Wissen flywheel, currently aspirational"** — M5 freeze-test forced the honest read that v1 is positional, not yet compounding.
- **From "pricing TBD" to "Cost-Recovery posture, Tagesticket anchor, three monetization paths kept open to open beta"** — M3 made the posture explicit and refused premature commitment.
- **From "eval at some point" to "defence-in-depth eval contract with closed user-flag loop and confidence-UX in Züritüütsch"** — M4 went from concept to running framework, with perceived-confidence treated as a product surface, not a backend metric.
- **From "responsible AI" hand-wave to "EU AI Act limited-risk + revDSG-primary + autonomy boundaries codified"** — M5 governance went from aspiration to operational policy with named audit cadence.

---

## Review log

| Date | Reviewer | What changed |
|---|---|---|
| 2026-05-18 | Benedict + Claude (Opus 4.7) | Initial fill. Thesis crystallized as sovereign civic AI for Zürich. Three risks named (correction loop fails to compound; sovereignty gap on judge; B2G timeline mismatch). Ask = warm intro + launch sponsor + Open Strategy feedback. M1 baseline (8/15 niche bet) vs Now (architecture concrete, moat honestly named as aspirational, posture explicit) — five named sharpenings. |
