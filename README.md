# Bünzli — AI Product Strategy

> A living strategy built across 6 modules of the Productschool *AI Product Strategy* course (Apr–May 2026), plus a 7th-module bonus track for the *Advanced AI Agents* course final project.
>
> Each module ships a **template** (the reusable artifact, anchored rubrics, structural insight) and a **Bünzli fill** (the honest instance, named gaps and all). The repo *is* the strategy — version-controlled, board-ready, portable — and is published verbatim at [`buenzli.space/strategie`](https://buenzli.space) as the **Open Strategy** commitment.
>
> **What Bünzli is.** The sovereign, hyper-local civic AI assistant for Zürich. Swiss-hosted on Apertus + Ministral, eval-contracted against a Züri-specific golden set, governed under EU AI Act limited-risk + revDSG. The moat is the corrections users leave behind, not the model under the hood.

---

## Strategy at a glance

| Module | Component | Status | Headline finding |
|---|---|---|---|
| **M1** | [The Bet](01-the-bet/) | ✅ | 8/15 niche bet — positional advantage real (Axis 3 = 4/5), compounding to be earned |
| **M2** | [The Moat](02-the-moat/) | ✅ | Data + Workflow moats; flywheel scored 4/20 at v1 → projected 14/20 at M24 |
| **M3** | [The Margin](03-the-margin/) | ✅ | CHF 148/mo fixed; Cost-Recovery posture; Tagesticket (CHF 5.60) anchor |
| **M4** | [The Contract](04-the-contract/) | ✅ | Defence-in-depth eval; closed user-flag loop; Sonnet→Prometheus 2 swap planned |
| **M5** | [The Guardrails](05-the-guardrails/) | ✅ | EU AI Act limited-risk + revDSG; freeze-test: positional, not yet compounding |
| **M6** | [The Pitch](06-the-pitch/) | ✅ | Three Horizons (AI-compressed); six board metrics; ask = warm intro + launch sponsor |
| **M7** | [The Agent (bonus)](07-the-agent/) | ✅ | NDA-helper AW Spec v0.4 + multi-agent collab spec, from Advanced AI Agents final |

---

## The Bet (M1)

**What we're building, for whom, why now.**

- **Product:** Bünzli — Zürich-focused civic AI assistant (web today, mobile later)
- **AI Value Archetype:** Sovereign hyper-local assistant (sovereignty × agency)
- **Vulnerability Scores:** Moat **2**/5 · Data **2**/5 · Platform **4**/5 → **8/15 niche bet**
- **Top Risk:** Hyperscaler attention turns to Swiss-German civic in <24 months
- **Confidence:** Medium
- **Kill criteria:** Apertus pricing degrades, or hyperscaler ships Züritüütsch-tuned civic answers
- **Open Strategy commitment:** Re-run the full exercise close to v1 launch and publish curated extracts publicly

→ Details: [`01-the-bet/`](01-the-bet/)

---

## The Moat (M2)

**Why this won't get copied in 6 months.**

- **Data flywheel score:** **4/20 at v1** → projected **14/20 at M24**
- **Loops:** Correction · Preference · Domain context · Network
- **Weakest loop:** Network Intelligence (structurally capped at 3/5 by privacy posture — accepted, not deficit)
- **Competitive position:** Sovereignty × Agency — partner-don't-compete vs Stadt Zürich
- **Kill switch:** Anthropic-default runtime ↔ sovereign-strategy gap (release-gate finding 5/15 at v1)
- **Vendor portability:** Partial — Apertus + Ministral routed, Sonnet-as-judge remains the residual dependency

→ Details: [`02-the-moat/`](02-the-moat/)

---

## The Margin (M3)

**Will this make money or bleed it?**

- **Fixed infra:** **CHF 148/mo** (Infomaniak VPS + Qdrant + domain + edge) — verified, not modelled
- **Variable cost:** dominated by inference; Apertus voice + Ministral tools ~70% cheaper than Sonnet-everywhere
- **Pricing posture:** **Cost-Recovery** (fourth posture beyond Skim / Penetrate / Maximize)
- **Anchor:** Zürich Tagesticket = CHF 5.60/mo — civic legitimacy, not market-clearing price
- **Three monetization paths kept open to open beta:** Tagesticket subscription · pay-per-action · B2G licensing
- **Break-even:** designed not to need it at v1 — no revenue by design, decision deferred to H3

→ Details: [`03-the-margin/`](03-the-margin/)

---

## The Contract (M4)

**Why users will trust a probabilistic system.**

- **Reliability target:** Hallucination ≤8% at M1 → ≤3% at M6 on golden set
- **Golden dataset:** **50 rows · 60/24/16 split · 7 adversarial categories**
- **Eval taxonomy:** 4 static + 6 dynamic categories (property-based, adversarial, drift, calibration, etc.)
- **LLM-as-judge:** Sonnet 4.7 v1 → **Prometheus 2 on rent-a-pod v1+3mo** (named sovereignty gap, scheduled close)
- **Confidence UX:** *"Bünzli sagt / gloubt / isch sich nöd sicher"* — perceived confidence as product surface, Züritüütsch
- **Closed loop:** "war das richtig?" flag → LLM-as-judge → eval-set row (the M5 Recursive Learning input)
- **HITL architecture:** defence-in-depth — Researcher reads, Critic verifies, Copywriter writes from verified findings only

→ Details: [`04-the-contract/`](04-the-contract/)

---

## The Guardrails (M5)

**What breaks when this scales — and what compounds.**

- **Compounding loops at v1:** all three aspirational — Recursive Learning **missing** (ships H1), Cross-Domain Transfer **missing** (unblocks H2 with Quandoo), Network Intelligence **capped at 3/5** (by-design, not deficit)
- **Freeze test:** *"Bünzli would still be winning frozen for 12 months — but for the wrong reason. Position, not compounding."* Strategic finding named, not papered over.
- **Governance posture:** EU AI Act **limited-risk** · **revDSG-primary** data protection · autonomy boundaries codified per action class
- **Audit cadence:** weekly telemetry · monthly red-team + sample review · quarterly privacy + regulatory · annual full review
- **Agent boundaries:** read & respond solo; write & decide require HITL; no cross-user data access; no runtime self-modification

→ Details: [`05-the-guardrails/`](05-the-guardrails/)

---

## The Pitch (M6)

**How you get this funded, shipped, and adopted.**

- **Horizon 1 (0–4 weeks, ship):** correction primitive · property-based eval on CI · confidence-UX live · Open Strategy publish
- **Horizon 2 (1–3 months, validate):** Recursive Learning compounds · Prometheus 2 swap · first 100 retained · Quandoo dry-run
- **Horizon 3 (3–6 months, explore):** B2G with Stadt Zürich · Cross-Domain Transfer measurable · monetization decision at open beta · legal-KB as own product
- **Board metrics (6):** Hallucination Rate · Drift Velocity · Confidence Distribution · HITL Rate · Inference ROI · Eval Regression — each with M1/M3/M6 targets
- **Board thesis:** *Bünzli is the sovereign, hyper-local civic AI assistant for Zürich — Swiss-hosted, eval-contracted, EU AI Act limited-risk, with the moat built from corrections users leave behind, not the model under the hood.*
- **Ask:** one warm intro to Stadt Zürich · one open-beta launch sponsor · 30 minutes of Open Strategy feedback

→ Details: [`06-the-pitch/`](06-the-pitch/)

---

## The Agent — bonus track (M7)

**Multi-agent architecture, from the Advanced AI Agents course final project.**

- **Project:** *Better Ask Saul* — NDA reviewer for fictional Pactly, deployed at [`asksaul.ch`](https://asksaul.ch)
- **Architecture:** LangGraph orchestration · Senior Counsel / Analyst / Copywriter role-character mapping · reflexion loop
- **Spec v0.4 findings:** research-write split · token economy as design constraint · instruction location (user-message > system) · abstraction defence · defence-in-depth verification
- **Crossover into Bünzli:** the defence-in-depth pattern from this build is what M4's eval contract codified for Bünzli

→ Details: [`07-the-agent/`](07-the-agent/)

---

## How to read this repo

Each module folder contains **template + Bünzli fill** pairs:

- **`*-template.md`** — the reusable artifact: anchored rubric, structural insight, connection to other modules, open questions, review log. Reusable across any AI product.
- **`*-buenzli.md`** — the honest instance: filled with Bünzli's actual numbers, gaps named, decisions dated.

**Connections matter.** Each filled doc opens with a "Connection to other modules" section that names which prior findings feed it and which downstream findings depend on it. The strategy is a graph, not a stack.

**Honest reads, not pitches.** Where the v1 product is weak, the doc says so (M5 freeze test, M2 flywheel 4/20 at v1, residual Sonnet dependency in M4). The Open Strategy commitment means these gaps get published, not hidden.

---

## Decision log — top 10

1. **Sovereign stack** — Apertus voice + Ministral tools + Infomaniak Swiss VPS · *M1 + M3*
2. **Sonnet-as-judge today, Prometheus 2 in H2** — named sovereignty gap with scheduled close · *M4*
3. **Cost-Recovery pricing posture** — Tagesticket anchor, three monetization paths open to open beta · *M3*
4. **EU AI Act limited-risk classification** — Article 50 transparency, revDSG-primary · *M5*
5. **No per-user behavioural inference** — Network Intelligence loop capped at 3/5 by design · *M2 + M5*
6. **Closed-loop "war das richtig?"** — user flag → LLM-as-judge → eval-set row · *M4*
7. **Defence-in-depth eval** — Researcher reads, Critic verifies, Copywriter writes from verified findings only · *M4 + M7*
8. **AI-compressed roadmap timeframes** — weeks/months, not quarters/years; monthly review cadence · *M6*
9. **Partner-don't-compete vs Stadt Zürich** — B2G as upside, not lifeline · *M2 + M6*
10. **Open Strategy publication** — re-run + publish at v1 launch; this repo is the audit trail · *M1 commitment*

---

## Repo conventions

- **Dates are absolute** (e.g. 2026-05-18), never relative ("last week").
- **Review logs** at the bottom of every filled doc; updated on substantive change.
- **Open questions** kept explicit rather than resolved prematurely.
- **No emoji** in the strategy docs themselves (only in README at-a-glance for scan-ability).
- **All docs are public.** This is the Open Strategy commitment from M1.

---

## License & attribution

Authored by Benedict von Hoffmann with Claude (Opus 4.7) as drafting partner, during the Productschool *AI Product Strategy* and *Advanced AI Agents* courses, April–May 2026. Templates are reusable under permissive terms; Bünzli fills are the product strategy of [buenzli.space](https://buenzli.space).
