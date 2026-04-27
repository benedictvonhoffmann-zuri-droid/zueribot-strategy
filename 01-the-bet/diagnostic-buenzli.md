# Three-Axis Vulnerability Diagnostic — Bünzli

> Filled instance of [`diagnostic-template.md`](diagnostic-template.md). Scores the **v1 product**, not today's pre-launch state. The strategic question is whether the v1 bet is structurally defensible — not whether the prototype already is.
>
> **Each axis owns one primary claim.** Workflow lock-in lives in Axis 1. Data signal lives in Axis 2. Hyperscaler economics — including sovereignty, market size, and bespoke civic API depth — lives in Axis 3. We do not repeat arguments across axes. If a claim defends three things, it sharply defends none.

---

## Product

**Product:** Bünzli — a Zürich-focused, Swiss-hosted AI assistant. Conversational interface backed by a curated Zürich knowledge base, Swiss-German voice, and agentic actions against local civic and commercial services (Züri wie neu, ERZ, Quandoo).
**Your role:** PM / founder.
**Stage:** Prototype, v1 launch in planning. Diagnostic scores the v1 bet.
**Date of diagnostic:** 2026-04-27.

---

## Timeframe

Structural — 3 to 5 years. Not "what could OpenAI ship next quarter," but "once the AI platform shake-out has settled, does Bünzli still have a reason to exist?"

The bet behind the bet: hyperscalers go where volume and margin justify the focus. Switzerland — small, multilingual, civic-bespoke, sovereignty-conscious — is the kind of market the math doesn't work for them but does for us.

---

## Axis 1 — Contextual Moat

**Score: 2 / 5 — Sticky-ish.**

**The single claim of this axis: at v1, switching costs are real but small.**

**Rationale.**
A v1 user accumulates a personal-context profile (address, household, schedule, a handful of preferences) and a history of agentic actions taken in their name. That is real — but it is operationally a small JSON, and ChatGPT-style memory features narrow the personalization gap further. Agentic actions produce *outcomes the user keeps* (the Züri wie neu report was filed, the Quandoo booking exists), but those outcomes do not bind the user to Bünzli for the next action.

There is also a **values-based stickiness layer**: a meaningful slice of v1 users will choose Bünzli specifically because it is privacy-first and Swiss-hosted. For those users, switching back to a US-hosted alternative is not just inconvenient — it compromises the reason they came in the first place. This is the user-side mirror of the market-side claim in Axis 3 (the platform owners cannot credibly serve this segment); here it shows up as switching reluctance once the user is on Bünzli. We weight this lightly at v1 — values stickiness is real but erodes fast if the product is not also functionally good — and it is part of why the score is 2 rather than 1.

Per the rubric: switching is annoying, not costly. 3 ("Embedded") is the v2 ambition, reached by becoming the entry point for *multiple* recurring civic and household workflows that other tools route through — not by accumulating profile fields.

**Evidence.**
- Personal-context volume per active user (small at v1 by design — a profile, not an archive).
- Agentic actions completed per active user per month — recurring use is what would lift this score.
- 4-week and 12-week retention curves.
- Qualitative: "what would you do if Bünzli shut down tomorrow?" — answered with concrete workflow disruption or with a shrug.

**Falsifier.**
- **Rises to 3** if (a) the median active user completes ≥X agentic actions per month *and* (b) at least one third-party Swiss service treats Bünzli as an identity / chat layer.
- **Drops to 1** if 30-day retention is below the threshold for a useful consumer product.

**Named attacker (from partner challenge):** *to be filled by partner exercise.*

---

## Axis 2 — Data Advantage

**Score: 2 / 5 — Surface signal.**

**The single claim of this axis: a curated dataset is not yet a flywheel.**

**Rationale.**
The KB is real (41,714 chunks, 10-category Zürich taxonomy) and the Swiss-German voice grounded in Apertus is a register a vanilla model does not match by default. Both raise the floor of v1 quality. But the rubric is strict on the difference between *data* and *flywheel*: at v1 the KB is essentially static, the user-correction loop is planned but not instrumented, and there is no measured eval lead over a strong baseline. "Data exists; flywheel doesn't" is the honest read. The path to 3 is operationally clear — ship the correction loop — and it is the most actionable lever on this entire diagnostic.

**Evidence.**
- KB chunk count and 10-category taxonomy coverage (current: 41,714 chunks, Phase 1 sealed 2026-04-25).
- Swiss-German voice grounded in Apertus + glossary in system prompt.
- Correction loop: planned for v1, not yet instrumented.
- Eval delta vs. vanilla baseline on Zürich-specific tasks: not yet run.

**Falsifier.**
- **Rises to 3** once a correction loop is shipped and accumulating at a meaningful rate (≥X corrections / week / active user — to be defined).
- **Rises to 4** if Bünzli's Swiss-context eval beats a strong vanilla baseline by a measurable margin within 6 months of v1 launch.
- **Drops to 1** if curation produces no answer-quality lead even with the loop in place.

**Named attacker (from partner challenge):** *to be filled by partner exercise.*

---

## Axis 3 — Platform Exposure

**Score: 4 / 5 — Not worth their focus.**

**The single claim of this axis: hyperscaler economics, sovereignty, and bespoke civic API depth combine to make this market structurally unattractive to platform owners.**

**Rationale.**
Three reinforcing reasons hyperscalers and OS platforms will not credibly serve this market within the diagnostic timeframe:

1. **Market math.** ~5M German-speakers, Swiss German is a tail dialect, and hyperscaler localization to date is "top-N languages, biggest metros first." Swiss-German Zürich is several rungs down that ladder.
2. **Sovereignty.** A meaningful share of Swiss users actively prefers Swiss-hosted, sovereign infrastructure for civic and personal use. Even a credible hyperscaler feature does not capture that segment.
3. **Bespoke civic depth.** Züri wie neu, ERZ, cantonal services are per-municipality integrations with no standard interface. Doing them properly is integration work hyperscalers do not do at city scale.

We score 4 rather than 5 because nothing structurally prevents a hyperscaler — it is an economics argument, not a regulatory wall. 5 would require a regulatory or distribution reality that genuinely walls them out.

**Evidence.**
- Switzerland market size: ~8.7M total, ~5M German-speakers; absolute volume small relative to a hyperscaler's roadmap calculus.
- Apple Intelligence regional rollout pattern: English first, then major EU languages — Swiss German not on any public timeline.
- Apertus + Swiss AI ecosystem: a national-level effort to retain sovereign capability, signalling the Swiss preference for non-US infrastructure is a real market signal, not a fringe view.
- No public hyperscaler roadmap mentions Swiss civic integrations.

**Falsifier.**
- **Drops to 2** if Apple Intelligence ships Swiss German + a Swiss local-services framework within 24 months.
- **Drops to 3** if a Swiss incumbent (Swisscom, local.ch, SRG, 20min) bundles a comparable assistant into existing distribution.
- **Stays at 4** if hyperscaler localization continues "top languages, biggest metros only" through 2027.

**Named attacker (from partner challenge):** *to be filled by partner exercise.*

---

## Killer memo — three attackers

### Attacker A — The hyperscaler / frontier lab

1. **Attack:** Add Swiss-German fine-tuning + a regional plug-in framework municipalities can register against. Bundle into ChatGPT Plus at no extra cost.
2. **Wedge:** Existing massive Swiss user base already paying for ChatGPT. Zero install friction.
3. **Why users switch:** For the casual half of Bünzli's user base — quick questions, no civic actions, no values preference — ChatGPT is "good enough" and already on the home screen.

### Attacker B — The vertical incumbent

1. **Attack:** local.ch adds a chat assistant on top of its directory + reviews. Or Swisscom bundles a Swiss-aware assistant into Blue/My Swisscom for free.
2. **Wedge:** Decades of Swiss brand trust, distribution to millions of Swiss users via existing apps and TV ad spend.
3. **Why users switch:** Brand familiarity beats novelty. "I already trust Swisscom with my data" beats "Bünzli is the new Zürich assistant."

### Attacker C — The OS / platform owner

1. **Attack:** Apple Intelligence ships Swiss German voice and a SiriKit-style local-services framework that municipalities (or Swiss aggregators) register against.
2. **Wedge:** Free, on-device, pre-installed on every iPhone in Switzerland. Integrates with Maps, Calendar, Mail, Shortcuts.
3. **Why users switch:** Zero install. For workflows that don't require Bünzli's depth, the OS-native version wins by default.

---

## Synthesis

**Combined score:** **8 / 15** (Moat 2 + Data 2 + Platform 4).

**Pattern.** Thin moat + thin data + high platform-resistance = **niche bet, carried by Axis 3.** The defensibility today is *positional, not earned*. Strip Axis 3 and what remains is a thin chat surface around curated content — defended by geography, not by product depth.

**The bet, in one sentence.**
*Hyperscalers will not bother with Swiss-German Zürich for the next 24 months. We will use that window to convert a curated KB into a correction flywheel (Axis 2 → 3) and a personal-context profile into recurring multi-workflow embedment (Axis 1 → 3). After 24 months, Bünzli is defensible by what it has earned, not just by where it lives.*

**What kills it.**
- Axis 3 collapses early (Apple ships native Swiss German + civic integrations within 18 months) **before** Axis 1 and Axis 2 have shipped any earned advantage.
- Or: Axis 3 holds, but Bünzli launches and never converts the window into earned defensibility — the diagnostic at month 24 still reads 8/15, and the niche slowly hollows out as native alternatives improve.

**What this means strategically.**
The diagnostic is the question, not the answer. The honest score today is 8/15. Every quarter from launch should be measurable against one question: *did we lift Axis 1 or Axis 2 by the end of it?* If a quarter passes without movement on either, the bet is decaying — even if the score has not yet fallen.

---

## 24-month focus

The diagnostic implies a tight focus list. Everything else is a distraction. If a piece of work does not lift one of these five, it does not belong on the v1–v2 roadmap.

1. **Ship the correction loop in v1, instrumented from day one.** Without this, Axis 2 cannot move. The correction loop is what turns a curated KB into a flywheel — every quarter without it is a quarter the data axis stays at 2.
2. **Design Bünzli for *recurrence*, not for *one-shot Q&A*.** The Axis 1 lift comes from users returning week after week through household and civic rhythms (bin pickup, school updates, recurring bookings, neighbourhood alerts). One-shot questions do not build switching cost; recurring use does.
3. **Land at least one third-party Swiss service that treats Bünzli as an identity or chat layer.** This is the structural move from Axis 1 = 2 to Axis 1 = 3. A single embedding (a cantonal service, a Swiss app, a community platform) proves the moat is becoming load-bearing for things outside the product itself.
4. **Run a Swiss-context eval against vanilla GPT and a strong open baseline. Publish internally each quarter.** Even a losing first run is more valuable than no measurement, because Axis 2 movement is invisible without it. "We think we're better at Zürich" is not an asset; "we are X points better on the Zürich eval" is.
5. **Test the sovereignty hypothesis with real users.** Do users actually switch *for* the Swiss-hosted angle, or do they say they care and not act? Both Axis 1's values-stickiness claim and Axis 3's sovereignty claim depend on this being true behaviourally, not just stated. If it is not, both axes weaken.

**Single guiding question for the next 24 months.** *Are we converting positional advantage (Axis 3) into earned advantage (Axis 1 + Axis 2) faster than the platform owners are closing the gap?* Every quarterly review answers exactly this question.

---

## Top vulnerability

Axis 3 collapses (a hyperscaler or OS platform ships native Swiss-German + civic integration) **before** Bünzli has converted positional advantage into earned advantage on Axis 1 or Axis 2 — leaving the product with neither.

---

## Confidence

**Level:** **M.**

> **Legend.**
> **H — High.** Scores grounded in shipped-product evidence: real users, measured retention, run evals, observed competitor behavior. The diagnostic could be defended in front of a skeptical board.
> **M — Medium.** Scores grounded in a concrete v1 plan and credible market read. Key unknowns are execution risk (will we ship what we said?) and unobservable competitor moves (what will Apple do?). Defensible, but a year of evidence would sharpen it.
> **L — Low.** Scores are mostly intent. No users, no evals, no committed plan. Useful as a thinking exercise, not yet as a strategic instrument.

**Why M, specifically.**
- Axis 1 (2) is honest about v1 reality — switching cost is small, by design, until recurring multi-workflow use accumulates.
- Axis 2 (2) is honest about today's data play — a curated KB without a flywheel.
- Axis 3 (4) assumes hyperscalers continue their established localization pattern through the next 24 months. Observable, but not guaranteed.

**What would raise confidence to H.**
- First 100 active v1 users with 4-week retention above a defined threshold.
- A measured Bünzli vs. vanilla-GPT eval on Zürich-specific tasks showing a meaningful delta.
- 12 months of Apple/Google not shipping Swiss-German + Swiss local-services.
- Evidence that the sovereignty preference drives actual switching behaviour, not just stated preference.

---

## Review log

| Date | Reviewer | What changed |
|---|---|---|
| 2026-04-27 | Benedict + Claude (Opus 4.7) | Initial fill, v1 bet |
| 2026-04-27 | Benedict + Claude (Opus 4.7) | Axis 2 demoted 3→2; Axis 1 reframed around hyper-local + community + civic graph (privacy/sovereignty moved to Axis 3); H/M/L legend added. Combined: 10→9. |
| 2026-04-27 | Benedict + Claude (Opus 4.7) | Axis 1 demoted 3→2 after VC stress-test — switching cost at v1 is annoying not costly. Each axis rewritten to own one primary claim, repetition cut. Synthesis sharpened: bet stated in one sentence, "what kills it" made explicit, "lift one axis per quarter" added as the operating instruction. Combined: 9→8. |
| 2026-04-27 | Benedict + Claude (Opus 4.7) | Privacy-first added to Axis 1 as user-side values-stickiness claim (mirror of Axis 3's market-side claim). 24-month focus section added: 5 priorities + a single guiding question. |
