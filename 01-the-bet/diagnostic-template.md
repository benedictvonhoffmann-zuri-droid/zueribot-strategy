# Three-Axis Vulnerability Diagnostic

> **Purpose.** Stress-test an AI product idea against the three forces most likely to kill it: a competitor stealing the workflow, a model provider absorbing the value, and a platform owner shipping it native. The output is a structural read on whether this bet is defensible — not a forecast of next quarter, but a view on whether the product still has a reason to exist in three to five years.
>
> **How to use it.** Fill it in honestly, with evidence, in one sitting. Then have a partner attack each score. A diagnostic you fill in alone tends to score itself a 4. A diagnostic a skeptic reviews tends to score a 2 — and that's the one worth trusting.

---

## Product

<!-- The real product, not a hypothetical. If you can't name it concretely, the diagnostic won't bite. -->

**Product:**
**Your role:**
**Stage:** *(idea / prototype / in-market / scaling)*
**Date of diagnostic:**

---

## Timeframe

This diagnostic scores **structural** vulnerability — the 3-to-5-year view, after the current AI platform shake-out has settled.

We do not score "what could a hyperscaler ship next quarter," because in the short term anyone can ship anything. We score whether, once the dust settles, this product still has a reason to exist that a generic large model or a platform owner cannot economically erase.

The implicit bet: **big AI companies will not rule every vertical.** They go where the volume and margin justify the focus. A defensible AI product lives in the gaps where that math doesn't work for them — but where it works for you.

---

## Scoring rubric

Each axis is scored 1–5 against an anchored rubric. **Pick the descriptor that matches, then write the score.** No half-points. If you're between two, pick the lower — optimism is the default failure mode of this exercise.

Every score requires:
- **Rationale** — why this descriptor, in plain language.
- **Evidence** — a concrete artifact, number, or observation. Not a feeling. If you can't name evidence, the score is a 1 by default.
- **Falsifier** — the specific signal that would move this score up or down. Names the experiment or metric, not a hope.
- **Named attacker** — from a partner's challenge, the most credible threat to this axis.

---

### Axis 1 — Contextual Moat

**The question.** How deeply is this product woven into the user's actual workflow, and how painful is it to leave? If a competitor with a similar model showed up on Monday, would users still be here on Friday?

**Why it matters.** Models are commoditizing. The product surface — the workflow, the integrations, the muscle memory, the data the user has poured in — is what makes leaving expensive. No moat here means you are renting the customer from whoever has the better demo this quarter.

**Rubric.**

| Score | Descriptor | What it looks like |
|---|---|---|
| 1 | **Disposable** | Standalone chat or tool. Zero integration. User could switch in an afternoon and lose nothing. |
| 2 | **Sticky-ish** | Some saved history or light personalization. Switching is annoying but not costly. |
| 3 | **Embedded** | Integrated into one or two real workflows. User has accumulated context (settings, saved artifacts, learned behavior). Switching costs a weekend. |
| 4 | **Load-bearing** | Multiple workflows depend on it. Other tools or teammates route through it. Switching breaks things and takes weeks to rebuild. |
| 5 | **Infrastructural** | The product is the system of record for something the user can't easily reconstruct elsewhere. Leaving means losing institutional memory or breaking downstream dependencies. |

**Score:** __/5
**Rationale:**
**Evidence:** *(retention curve, average integrations per active user, weekly active workflows, qualitative "what would you do without us")*
**Falsifier:** *(e.g. "score moves to 4 if median user has ≥3 active integrations by Q3"; "score drops to 1 if 30-day retention stays under X%")*
**Named attacker:**

---

### Axis 2 — Data Advantage

**The question.** Is there a proprietary signal flowing through this product that compounds with usage — something a generic foundation model literally cannot see? Does more use make the product measurably better in a way a competitor starting today cannot replicate by buying API credits?

**Why it matters.** "We use AI" is not a moat — everyone uses the same models. The durable advantage is data the model provider doesn't have: private workflows, vertical-specific corrections, proprietary distributions, behavioral signal from a real user base. If your product gets better the more it's used, and that improvement is not available off-the-shelf, you have a flywheel. If not, you are a thin wrapper.

**Rubric.**

| Score | Descriptor | What it looks like |
|---|---|---|
| 1 | **Pure wrapper** | All intelligence is in the foundation model. You see nothing the API provider doesn't already see. |
| 2 | **Surface signal** | You collect usage logs, but nothing structured enough to improve the product. Data exists; flywheel doesn't. |
| 3 | **Latent advantage** | You have proprietary data (user corrections, domain artifacts, private context) but you are not yet using it to compound product quality. |
| 4 | **Compounding loop** | Usage produces structured signal that measurably improves outputs over time — evals show your product beats a vanilla model on your task. |
| 5 | **Unreachable signal** | You sit on data a foundation model cannot legally, technically, or economically acquire — and you have demonstrated it materially changes outputs. |

**Score:** __/5
**Rationale:**
**Evidence:** *(eval delta vs. baseline model, volume and structure of feedback signal, dataset uniqueness, contractual access)*
**Falsifier:** *(e.g. "score moves to 4 if our fine-tuned eval beats GPT-baseline by ≥X points on internal benchmark by date Y")*
**Named attacker:**

---

### Axis 3 — Platform Exposure

**The question.** If a platform owner — Apple, Google, Microsoft, or a frontier model lab — ships your hero feature natively, do you still have a reason to exist? How fast can you pivot, and to what?

**Why it matters.** Platform owners ship features that touch hundreds of millions of users at zero marginal cost to them. If your entire value prop is something they can plausibly absorb into the OS or the chat box, your timeline is their roadmap. The defense is either being too niche to be worth their attention, owning a distribution channel they don't control, or having a workflow moat (Axis 1) deep enough that the native version is a worse experience.

**Rubric.**

| Score | Descriptor | What it looks like |
|---|---|---|
| 1 | **Direct overlap** | Your hero feature is on a hyperscaler's obvious roadmap and they have every reason to ship it. You are competing with a free, pre-installed version. |
| 2 | **In the blast radius** | Adjacent to a likely native feature. They might ship it as a side effect of something else. |
| 3 | **Specialized enough to survive** | A native version would exist but be generic. Your specialization (vertical, workflow, geography, regulation) means committed users still prefer you. |
| 4 | **Not worth their focus** | The market is real but too narrow, regulated, or relationship-driven for a hyperscaler to prioritize. They could build it; the math doesn't justify it. |
| 5 | **Structurally off-limits** | Building this would require something they fundamentally won't or can't do (data residency, regulated industry posture, anti-trust constraint, channel they don't own). |

**Score:** __/5
**Rationale:**
**Evidence:** *(public roadmaps, recent platform releases, regulatory constraints, distribution you control that they don't)*
**Falsifier:** *(e.g. "score drops to 2 if Apple Intelligence ships native local-language assistant in EU by date X")*
**Named attacker:**

---

## Killer memo — three attackers

A single memo flattens the threat picture. Real products are attacked from at least three angles at once. Write a 3-sentence memo from each archetype.

> **Format for each:**
> 1. **Attack** — what they ship.
> 2. **Wedge** — why their version reaches users.
> 3. **Why users switch** — the specific reason a current user of yours leaves.

### Attacker A — The hyperscaler / frontier lab
*(OpenAI, Anthropic, Google. Their move: absorb the workflow into the chat box or the OS. Their advantage: distribution and model quality. Their blind spot: generic by default.)*

1. Attack:
2. Wedge:
3. Why users switch:

### Attacker B — The vertical incumbent
*(The dominant non-AI player in your space who finally adds AI. Their move: bolt a chat layer onto the existing product. Their advantage: existing distribution and trust. Their blind spot: legacy product surface.)*

1. Attack:
2. Wedge:
3. Why users switch:

### Attacker C — The OS / platform owner
*(Apple, Microsoft, the dominant device or browser. Their move: ship the feature natively, free, pre-installed. Their advantage: zero install friction. Their blind spot: lowest-common-denominator quality, slow to specialize.)*

1. Attack:
2. Wedge:
3. Why users switch:

---

## Synthesis

The three axes interact. The dangerous failure mode is not a single low score — it is a *combination*. A weak moat with a strong data advantage may still hold. A weak moat with no data advantage and high platform exposure is structurally cooked.

**Combined score:** __/15

**Pattern.** *(e.g. "low moat + low data + medium platform = wrapper at risk of obsolescence"; "medium moat + high data + low platform = defensible niche"; "high moat + low data + low platform = workflow product, model-agnostic — defensible as long as integrations hold")*

**The honest read.** *(In one paragraph: where does this product actually live in three years, and what is the bet that gets it there?)*

---

## Top vulnerability

The single biggest structural risk, in one sentence. Not a list. The one that, if it plays out, makes the rest of the strategy moot.

---

## Confidence

Confidence is in the *diagnostic*, not the product. Tied to the specific unknowns that would change the read.

**Level:** H / M / L

> **Legend.**
> **H — High.** Scores grounded in shipped-product evidence: real users, measured retention, run evals, observed competitor behavior. The diagnostic could be defended in front of a skeptical board.
> **M — Medium.** Scores grounded in a concrete v1 plan and credible market read. Key unknowns are execution risk (will we ship what we said?) and unobservable competitor moves (what will Apple do?). Defensible, but a year of evidence would sharpen it.
> **L — Low.** Scores are mostly intent. No users, no evals, no committed plan. Useful as a thinking exercise, not yet as a strategic instrument.

**Why this level:** *(name the specific unknowns. e.g. "M — Axis 1 score assumes v1 ships the planned agentic actions; Axis 2 assumes the correction loop is wired up; Axis 3 assumes hyperscalers continue their pattern of skipping tail-dialect markets.")*
**What would raise confidence:** *(specific evidence or experiments that would move the level up)*

---

## Review log

| Date | Reviewer | What changed |
|---|---|---|
| | | |
