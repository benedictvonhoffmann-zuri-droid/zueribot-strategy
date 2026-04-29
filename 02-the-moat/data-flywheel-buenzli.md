# Data Flywheel Map — Bünzli

> Filled instance of [`data-flywheel-template.md`](data-flywheel-template.md). Scores Bünzli's data and workflow moats against the four compounding mechanisms (Correction, Preference, Domain context, Network). Filled 2026-04-29.
>
> **The two primary moats this document engineers** are **Data** and **Workflow** — the same two levers Module 1's diagnostic identified as Axis 2 and Axis 1. The other Hamilton-Helmer-style moats (regulatory, distribution, ecosystem, network, physical, scale) are either tailwinds, future plays, or structurally not available to Bünzli — see Module 1's synthesis for that conversation.

---

## What is a data flywheel? (recap)

A self-reinforcing loop where using the product produces signal, the signal makes the system better, the better system produces better outputs, and the better outputs drive more usage — which produces more signal. Each turn adds energy. The longer the wheel spins, the harder it is for a competitor starting from zero to catch up. A static dataset is not a flywheel.

For Bünzli the loop must spin **without violating the privacy commitment** — meaning signal is *volunteered, aggregate, or community-contributed*, never extracted by passive surveillance. That single constraint shapes every score below.

---

## Connection to the diagnostic

This document engineers Axis 2 of the diagnostic.

> **Diagnostic reference.** Axis 2 (Data) — current **2 / 5** ("Surface signal: data exists, flywheel doesn't"). Target M24: **3 / 5** ("Latent advantage" with proven correction loop) or **4 / 5** if we hit a measured eval lead. The flywheel total below should grow in step.

It also informs Axis 1 (Contextual moat) — the workflow moat is how Domain Context compounds operationally. The two modules' primary moats are deliberately the two highest-ceiling loops in this document.

---

## Loop 1 — Correction

**Score: 1 / 5 — No capture.**

**Rationale.**
At v1 there is no shipped UI primitive for users to flag wrong answers, edit facts, or contribute corrections. The KB is curated by the team, not by users. This is the single highest-leverage move on the entire flywheel — and it is the bet of the next 6 months. Until the correction loop ships, every other score is structurally limited because the system has no input pathway to learn from. Per the rubric, "no mechanism for users to flag or fix outputs" = 1.

**Evidence.**
- No "war das richtig?" widget in v1 prototype.
- No Quartier-Wissen contribution channel.
- KB updates require team-side editing.

**Falsifier.**
- **Rises to 3** once an in-product correction widget is live and flowing into a weekly review-and-merge cadence.
- **Rises to 4** once corrections feed an automated retrieval refresh + quarterly eval cycle, with measurable system delta per batch.
- **Stays at 1** if we ship v1 without a correction primitive — the riskiest possible outcome.

**Trajectory.** *Now: 1/5 · M12: 3/5 · M24: 4/5.*
- M12: in-product correction loop shipped + Quartier-Wissen wiki-style contributions live + weekly merge cadence.
- M24: instrumented pipeline — corrections feed automated retrieval refresh, quarterly eval cycle showing measurable delta vs. baseline.

---

## Loop 2 — Preference

**Score: 1 / 5 — Stateless.**

**Rationale.**
At v1 the product has no per-user state across sessions. Every user gets the same Bünzli. This is partly cold-start (we haven't shipped accounts yet) and partly intentional — the privacy commitment caps how aggressively we can profile users. We will personalise on *stated* preferences and *user-controlled* personal-context fields (address, household, schedule, preferred restaurants), but we will not infer behaviourally in ways the user hasn't opted into.

That makes Preference **structurally capped at 3 ("Stated preference") by our values**. Reaching 4 ("Behavioural inference") would require collecting and acting on signal the user did not consciously volunteer — which we won't do. This is a deliberate trade-off, not a weakness, and it should be named in the synthesis. The strong moats live elsewhere (Correction, Domain Context).

**Evidence.**
- No personal-context profile in v1 prototype.
- No saved-history personalization.
- Privacy commitment publicly states no model training on user data.

**Falsifier.**
- **Rises to 2** once basic personal-context fields (address, household, schedule) are stored.
- **Rises to 3** once stated preferences (vegetarian, short answers, Quartier preferences) are remembered and applied across sessions.
- **Will not rise to 4 or 5** without changing the privacy posture — and we won't.

**Trajectory.** *Now: 1/5 · M12: 2/5 · M24: 3/5.*
- M12: personal-context profile shipped, surface personalization (Surface profile).
- M24: stated preferences applied across multiple workflows (Stated preference). Capped here by values.

---

## Loop 3 — Domain context

**Score: 1 / 5 — Siloed.**

**Rationale.**
At v1 the KB exists in one bucket and the agentic actions are independent flows — there is no signal cross-pollination between civic (Züri wie neu, ERZ), commercial (Quandoo), and household domains. This is the **other high-ceiling loop** alongside Correction, and it is uniquely well-suited to Bünzli: a product that combines civic + commercial + household at the Zürich Quartier level can produce cross-domain signal that no single-domain competitor can match.

Concrete examples of cross-domain compounding to build:
- **Quandoo bookings inform restaurant-info quality** (which Quartier preferences cluster by demographic, what's actually busy on Sa 20:00).
- **Züri wie neu resolution outcomes inform civic-question accuracy** ("when is the right time to file a pothole report? Bünzli knows because it has aggregate resolution-time data").
- **Aggregate Quartier-level signal informs all domain answers about that Quartier** (the more questions asked from Wipkingen, the better Bünzli's answers about Wipkingen across every domain).

This is the moat Stadt Zürich's official "Züri-Bot" (the Attacker-D scenario) cannot replicate — it would only have civic data, not cross-domain.

**Evidence.**
- No cross-domain signal pipeline in v1.
- Each agentic action runs in isolation.
- Quartier-level aggregation not yet instrumented.

**Falsifier.**
- **Rises to 2** once domains share infrastructure (common KB, shared retrieval).
- **Rises to 3** once one cross-domain transfer is operational (e.g. Züri wie neu outcome data improves civic answer quality).
- **Rises to 4** once signal from any domain measurably improves quality in ≥2 others.
- **Drops to 1 (stays there)** if we build domain integrations as parallel verticals without a shared signal layer — the standard mistake.

**Trajectory.** *Now: 1/5 · M12: 2/5 · M24: 4/5.*
- M12: shared stack live, first cross-domain transfer instrumented.
- M24: multi-domain compounding — signal from any of {civic, commercial, household} measurably improves ≥2 others. Quartier-level aggregation as a feature.

---

## Loop 4 — Network

**Score: 1 / 5 — Isolated.**

**Rationale.**
Bünzli is not a social product, by structure. There is no inter-user messaging, no shared lists, no community feed. What looks like a network effect — community contributions making the KB better for everyone — is actually **Correction at scale, relabelled**. We are honest about this: Network is **structurally capped at 3 ("Indirect benefit")** for a product of Bünzli's shape. We will never build value that flows directly user-to-user, because that would mean either becoming a social product (mission drift) or weakening the privacy posture (values violation).

What we *will* build:
- **Aggregate visibility:** "popular this week in your Quartier" cards (level 2).
- **Indirect benefit via community:** Quartier-Wissen contributions improve answers for everyone (level 3, but properly counted as Correction at scale).

Naming this honestly in the synthesis matters more than fudging the score. The flywheel is strong on the loops where it should be — *not* on Network, and that is fine.

**Evidence.**
- No user-to-user value primitives in v1 or v2 plan.
- Privacy posture explicitly forbids cross-user data exposure.
- Product mission is a personal civic assistant, not a community platform.

**Falsifier.**
- **Rises to 2** once aggregate-Quartier visibility is shipped.
- **Rises to 3** once community contributions (Quartier-Wissen) are at meaningful volume.
- **Will not rise to 4 or 5** without a fundamental product-shape change.

**Trajectory.** *Now: 1/5 · M12: 2/5 · M24: 3/5.* Capped here by structure.

---

## Synthesis

**Total flywheel score:** **4 / 20** *(now)* · **9 / 20** *(M12 target)* · **14 / 20** *(M24 target)*

**Weakest loop today.** All four loops are tied at 1 (cold start). Looking forward, the **structurally weakest loop is Network**, capped at 3 by product shape. Among the loops we can actually move, the lowest-ceiling is **Preference**, capped at 3 by values. The two loops with real ceilings (4–5) are **Correction** and **Domain Context** — and that is exactly where the operational focus belongs.

**Bottleneck logic.**
At v1 the bottleneck is cold start: every loop is 1 because nothing has shipped. After v1, the bottleneck shifts — and shifts in a deliberate, designed way. **Correction is the gating dependency for everything else**: until the correction loop ships, the system has no input pathway to compound on, so Domain Context cannot mature (no signal to cross-pollinate), Preference stays at "stated only," and Network's indirect benefit (level 3) is not reachable. Lifting Correction from 1 → 3 in months 0–6 is the single highest-leverage move on the whole map. After that, Domain Context becomes the next bottleneck and the focus of months 6–24.

**Fix for the weakest loop.**
Ship the in-product correction primitive by **month 6** of v1 launch:
- "War das richtig?" thumbs up/down + optional typed correction at the end of every Bünzli response.
- Quartier-Wissen wiki-style contribution channel separate from chat data, public-by-design.
- Weekly review-and-merge cadence with a defined owner.
- Instrumented metric: corrections-per-active-user per week, with a target threshold to be defined.

This single move lifts Correction from 1 → 3, which alone moves the flywheel total from 4 → 6 — and unblocks every downstream loop.

**The honest read.**
Bünzli's flywheel is **Correction-heavy and Domain-Context-heavy by design**, **Preference-capped at 3 by values**, and **Network-capped at 3 by structure**. The 14/20 M24 target reflects that — it is not a flywheel that maxes out at 20, and it should not pretend to be. The two high-ceiling loops align exactly with the two primary moats from the moat-types conversation: Data (Correction) and Workflow (Domain Context as the engineered substrate). The other two loops are honest constraints, not failures. A real flywheel must show movement on every quarterly re-score; if Correction or Domain Context flatlines for two consecutive quarters, the moat is decaying — even if Preference and Network still look fine.

---

## Competitive positioning

**Selected axes: Sovereignty × Agency** *(sovereign–global × informational–agentic)*

**Why these two.** They are the cleanest single map of both moats: data sovereignty pulls in privacy-first, Swiss-hosting, and Apertus alignment; agency pulls in workflow depth, the agentic-action play, and the multi-domain integration thesis. Together they capture the bet that Bünzli holds the high-sovereignty corner *and* climbs to high-agency multi-domain — a quadrant no global player and no civic-only player will fill.

**Map — current position and target moat.**

| Competitor | Sovereignty | Agency | Notes |
|---|---|---|---|
| **Bünzli — today** | High | Low–Medium | Swiss-hosted from day one (sovereignty already won); basic Züri wie neu / ERZ / Quandoo agentic flows. |
| **Bünzli — target moat (M24)** | High | High *(multi-domain)* | Same sovereignty posture, but multi-domain agentic chains crossing civic + commercial + identity. The empty quadrant. |
| ChatGPT / Gemini | Low | Medium *(Operator, Tasks)* | Global, generalist, US-hosted. Best at casual Q&A; structurally cannot match sovereignty. |
| Apple Intelligence | Medium *(on-device)* | Low–Medium | Privacy via on-device, but no civic / Swiss depth. |
| local.ch + AI / 20min + AI | Medium *(Swiss)* | Low *(informational)* | Existing Swiss distribution, but no agentic muscle. |
| Stadt Zürich official "Züri-Bot" *(Attacker D)* | High | Low *(civic only)* | High sovereignty, but bound to civic services only — can't cross into commercial or household. |

**The move.**
*We hold the sovereignty axis as a structural commitment and travel up the agency axis from low-medium to high — by deepening Tier-3 partnerships (SwissID, Stadt Zürich, Tagesschule) and instrumenting cross-domain Quartier-level signal — converting positional advantage into multi-domain action depth no global player and no civic-only player can match.*

**The honest read.**
At the M24 position, Bünzli beats global generalists (ChatGPT, Gemini) for Swiss-resident multi-domain civic + commercial + household use because of the sovereignty + agency combination. Bünzli beats the Stadt Zürich official "Züri-Bot" on breadth (commercial + household, not just civic). Bünzli does **not** beat ChatGPT for casual general-purpose Q&A — and is not trying to. The move from today's dot to M24 is ambitious-but-credible *only if* the operational work lands: correction loop by M6, first cross-domain transfer by M12, SwissID and Stadt Zürich relationships by M18. Without that work, M24's dot is wishful and the strategy is decaying.

---

## 90-day encroachment plan — three attack vectors

A single named attacker creates blind spots. Bünzli is plausibly attackable from three different angles with three different shapes — a hyperscaler with model quality and pre-installation, a focused vertical startup with deeper data than ours, and a Swiss incumbent with massive distribution who adds AI as "one more thing." Each requires its own response, and the structural defence (partnering with Stadt Zürich) addresses several of them simultaneously.

---

### Vector 1 — Platform

**Named attacker.** **Apple Intelligence ships Swiss German + a SiriKit-style local-services framework** that municipalities (or a Swiss aggregator like Apertus or local.ch) can register against. Within 24 months is plausible; within 12 months is possible but unlikely.

**Their move.** Native on-device assistant in Swiss German, integrated with Maps, Calendar, Mail, Shortcuts, and a local-services plug-in framework. Default-status for everything Bünzli does today, available to every iPhone user in CH at zero install friction.

**Their leverage.** Pre-installation on ~50%+ of Swiss smartphones. Privacy story (on-device) that *competes directly with* Bünzli's sovereignty axis — the only hyperscaler that can credibly claim it. Apple doesn't need to monetise the feature; it sells hardware.

**Tripwire.**
- WWDC announcement of Swiss German support or Switzerland-specific Apple Intelligence rollout.
- SiriKit framework for civic services or local-government plug-ins.
- Rumour of an Apple ↔ Stadt Zürich or Apple ↔ SBB partnership.
- Apple Maps adding Zürich-specific civic data layers.

*Also-watch: OpenAI ships a Swiss-German fine-tune with civic plug-ins. Less existentially dangerous (no on-device privacy story) but more likely within 12 months.*

---

### Vector 2 — Vertical competitor

**Named attacker.** **Tsüri.ch ships "Tsüri-Bot"** — an AI assistant on top of seven years of curated, hyper-local Zürich journalism, community engagement, and Quartier-level reporting. Could happen as a product evolution, an acquisition by a media group, or via a small dedicated team.

**Their move.** A chat assistant grounded in Tsüri.ch's content archive, plus their existing community-contribution muscle. Could plausibly include event listings, neighbourhood-specific stories, recommendations, and editorial answers about Zürich life.

**Data they have that we don't.**
- **7+ years of editorial Quartier content**, written by Zürich journalists for Zürich readers — exactly the curation-quality bar we are aiming for.
- **A real community engagement track record** — comments, story corrections, reader tips, story requests. The community-correction loop we are building, they have already been operating (in journalistic form) for years.
- **Trust with the young / progressive demographic in Zürich** — the early-adopter slice we most want.
- **A network of local contributors**: journalists, photographers, neighbourhood sources.

This is the honest answer to "what data do they have that you don't" — content depth, community trust, and editorial taste, all accumulated over years that we cannot compress.

**Tripwire.**
- Tsüri.ch hires an AI engineer or ML lead.
- A funding round for Tsüri.ch with AI in the wording.
- Product announcement of a chat / search interface, even an MVP.
- Partnership announcements with Apertus or a Swiss AI lab.

*Also-watch: a new Zürich-focused civic-tech startup we don't know about yet. Watch the Swiss startup-scene news, Pioneer's portfolio, F10 cohorts.*

---

### Vector 3 — Adjacent expansion

**Named attacker.** **Swisscom bundles a Swiss-aware AI assistant into Blue or My Swisscom** — possibly co-branded with Apertus or built on a Swisscom-developed stack.

**Their move.** AI assistant pre-installed inside Blue (their TV / streaming app) and surfaced in My Swisscom (the customer portal) and the Swisscom mobile app. Likely focused on telco support first, then expanded to cover Swiss services more broadly. Probably free for Swisscom customers, possibly bundled as a premium feature.

**Their distribution advantage.**
- **~5M Swiss customers** with active Swisscom relationships — meaningful overlap with Bünzli's addressable market.
- **Pre-installed mobile presence** through Blue, the Swisscom app, and the SwissID-equivalent identity layer they already operate.
- **Trust as Swiss infrastructure** — Swisscom is one of the few brands with sovereignty credibility comparable to ours.
- **Capital and BD muscle** to do the Tier-3 partnerships we are pursuing — they could close a Stadt Zürich civic AI deal faster than we could.

This is the most-likely-to-happen-within-24-months vector: Swisscom has explicitly been building AI offerings, and adding a consumer assistant on top of existing distribution is a natural product move.

**Tripwire.**
- Swisscom AI division announcement of a consumer-facing assistant.
- Blue update notes mentioning a chat or AI feature.
- Swisscom + Stadt Zürich partnership news (this would address several attackers' moves at once).
- Swisscom hiring "Conversational AI" PM/eng roles publicly.

*Also-watch: 20min adds AI to its app (massive daily reach via news distribution); local.ch + Salt do similar in the directory + telco intersection.*

---

### Response by horizon

| Horizon | vs Platform *(Apple)* | vs Vertical *(Tsüri.ch)* | vs Adjacent *(Swisscom)* |
|---|---|---|---|
| **Week 1 — reactive** | Public messaging: *Apple isch für d'Schnell-Frag, Bünzli macht d'Arbet für di.* Apple covers casual queries; Bünzli covers multi-domain *action*. Sharpen the action-vs-info distinction publicly within 7 days. | Public messaging: *Tsüri.ch verzellt, Bünzli macht.* Their strength is content and editorial; ours is action-taking and personal civic graph. Position as complementary, not competitive. | Public messaging: *Sicher, Schwiizerisch, und unabhängig.* Bünzli is not bundled into a telco — works for any Swisscom, Salt, or Sunrise customer; users keep their data and their assistant decoupled. |
| **Month 1 — product** | Ship one multi-domain agentic chain Apple structurally cannot easily do natively: e.g. file Züri wie neu → calendar reminder for resolution → ZVV reroute suggestion. Real cross-domain orchestration in one user flow. | Open a content-partnership track: integrate Tsüri.ch RSS / public archive into Bünzli's KB with attribution and a click-through. Convert their content lead into our distribution by becoming a friendly upstream surface. If they refuse, ship a clear editorial-quality demo that proves we close the gap. | Ship a sovereignty-and-portability proof: explicit "your data is here, export it now, delete it now" UI. Make the *independence* angle a feature Swisscom structurally can't replicate inside their own walled experience. |
| **Quarter 1 — structural** | **Lock in Tier-3 partnerships Apple cannot easily replicate** — SwissID integration, Stadt Zürich civic-data depth, Tagesschule pilot. The structural move is *exclusivity-by-locality*: become the assistant whose civic depth lives in places Apple cannot economically build for. | Acquire, partner, or out-execute. Best move with a sympathetic vertical competitor is often *partnership* (us as the action layer, them as the editorial layer). If they go independent, out-execute on action depth — content alone isn't a moat once we have a correction flywheel running. | **Pre-empt by becoming the procured supplier.** Open a formal partnership conversation with Stadt or Kanton Zürich *before* Swisscom or any other adjacent gets there. Same partner-don't-compete play we discussed against the public-sector Attacker D — and it addresses Vectors 1 and 3 simultaneously. |

The Quarter-1 row's third cell is the highest-leverage move on this entire plan: **a Stadt/Kanton Zürich partnership addresses Platform (lock in distribution Apple can't replicate), Vertical (city deal beats Tsüri.ch's content), AND Adjacent (city deal beats Swisscom's distribution).** It is the single structural play that addresses all three vectors at once. Everything else is reactive.

---

### The honest read

**Most urgent.** Vertical (Tsüri.ch). Most under-estimated, easiest to launch quietly, and structurally has the data we are still trying to build. The other two vectors are louder but slower.

**Where do we survive intact?** Only if the Quarter-1 Stadt/Kanton Zürich partnership lands. With it, all three vectors have a credible counter — Bünzli is the city's procured assistant, has Tier-3 partnerships locked in, and a Swisscom or Apple offering becomes a complement (or a downstream client) rather than a replacement.

**Where do we survive smaller?** If the partnership doesn't land but the Month-1 product moves do, Bünzli holds at reduced scope as the *broader-than-civic, broader-than-content, independent-of-telco* Zürich assistant. The committed-user segment stays; the casual segment is lost to whichever attacker reaches them first. Scope shrinks; moat survives.

**Where do we not survive at all?** If Apple ships Swiss German + on-device civic services *and* Swisscom bundles a comparable assistant into Blue *and* Tsüri.ch ships an editorial AI — all within 12 months — and we have not landed the Stadt Zürich partnership, the niche collapses. This is the catastrophic scenario the diagnostic flagged as the Top Vulnerability, now made explicit at vector level. Unlikely but not negligible.

**The single structural move.** The Quarter-1 Stadt/Kanton Zürich partnership is not just an answer to one attacker — it is the keystone defence against all three. Treat it as the most important non-product piece of work in the next 12 months. It is mostly BD and partnerships, not engineering — and that is the work we have not yet started.

---

## Review log

| Date | Reviewer | What changed |
|---|---|---|
| 2026-04-29 | Benedict + Claude (Opus 4.7) | Initial fill against the four course loops (Correction, Preference, Domain context, Network). Honest naming of structural caps on Preference (values) and Network (product shape). Sovereignty × Agency selected as the positioning axes. Stadt Zürich named as the most concrete near-term attacker; partner-don't-compete framed as the structural response. |
| 2026-04-29 | Benedict + Claude (Opus 4.7) | Encroachment plan restructured into three attack vectors: Platform (Apple Intelligence), Vertical (Tsüri.ch), Adjacent (Swisscom). 3×3 response grid added (week 1 / month 1 / quarter 1 × three vectors). Stadt/Kanton Zürich partnership identified as the single structural move addressing all three vectors at once. Generic template updated to match. |
