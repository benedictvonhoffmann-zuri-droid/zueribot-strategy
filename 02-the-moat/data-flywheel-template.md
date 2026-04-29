# Data Flywheel Map

> **Purpose.** Engineer the moat. Module 1's diagnostic asks *"is the data axis defensible?"*; this doc asks *"what does the loop that creates that defensibility actually look like, where is it weakest, and how does it get stronger over time?"*
>
> A data flywheel is four loops chained end-to-end: usage produces signal, signal improves the system, the better system produces a better experience, the better experience drives more usage. The flywheel is only as strong as its weakest link — diagnose the weakest loop, and you have your roadmap.
>
> **How to use it.** Score honestly at month 0. Re-score at every quarterly review. A flywheel that does not move is decaying — even if the score has not yet fallen.

---

## Timeframe

**Operational — 12 to 24 months.** Unlike the structural diagnostic, this is a build plan: what loops exist now, what loops we are constructing, what the system looks like when it works.

For each loop we record the **score today**, plus the **trajectory at month 12 and month 24** under the current plan. A flywheel that scores 4/20 today and 14/20 in 18 months is a *plan*; one that stays at 4/20 indefinitely is a wrapper.

---

## Connection to the diagnostic

This document is where Axis 2 of the diagnostic gets engineered. If the diagnostic says Axis 2 is the lever to pull, the flywheel score is what tells us we are pulling it. The two should track together — Axis 2 cannot move without the flywheel total moving.

> **Diagnostic reference:** Axis 2 (Data) — current score __ / 5. Target at month 24: __ / 5. The flywheel total below should grow in step.

---

## Scoring rubric

Each loop is scored 1–5 against an anchored rubric. **Pick the descriptor that matches, then write the score.** No half-points. If between two, pick the lower — optimism is the default failure mode.

Every loop requires:
- **Rationale** — why this descriptor.
- **Evidence** — concrete artefact, number, or instrumented metric. If you can't name evidence, the score is a 1.
- **Falsifier** — the specific signal that would move this score up or down.
- **Trajectory** — current / month 12 / month 24, under the current plan.

---

### Loop 1 — Usage → Signal

**The question.** Do users produce a structured signal we can capture, beyond raw logs?

**Why it matters.** No signal means nothing to compound on. A flywheel cannot spin without input.

**Rubric.**

| Score | Descriptor | What it looks like |
|---|---|---|
| 1 | **Logs only** | No structured proprietary signal. Raw request/response logs at most. |
| 2 | **Sparse signal** | Some structured signal collected (ratings, completions, click-through) but shallow and inconsistent. |
| 3 | **Reliable signal** | Specific signal per interaction — corrections, action outcomes, structured ratings — captured reliably across a defined set of touchpoints. |
| 4 | **Rich signal** | Multiple signal types (text + behavioural + outcome) at meaningful volume, structured for downstream use. |
| 5 | **Proprietary by design** | Signal that no third party can collect — private context, regulated data, in-the-loop user corrections — captured in a form that directly drives improvement. |

**Score: __/5**
**Rationale:**
**Evidence:**
**Falsifier:**
**Trajectory:** *Now: __/5 · M12: __/5 · M24: __/5*

---

### Loop 2 — Signal → Model / System

**The question.** Does that signal actually improve the model or system, on a defined cadence?

**Why it matters.** Signal that sits in a database is not a flywheel. The loop only spins if signal → improvement is wired up.

**Rubric.**

| Score | Descriptor | What it looks like |
|---|---|---|
| 1 | **Inert** | Signal is collected but not used. No improvement mechanism exists. |
| 2 | **Manual ad-hoc** | Signal drives occasional manual updates (KB edits, prompt tweaks). No cadence. |
| 3 | **Defined cadence** | Signal feeds a structured improvement process (evals, retrieval refresh, fine-tune cycle) on a known cadence. |
| 4 | **Instrumented pipeline** | Improvement runs automatically. New signal produces a measurable system delta within days. |
| 5 | **Live training** | Signal directly trains/tunes the model in production. The system measurably improves with every batch, no human in the loop required. |

**Score: __/5**
**Rationale:**
**Evidence:**
**Falsifier:**
**Trajectory:** *Now: __/5 · M12: __/5 · M24: __/5*

---

### Loop 3 — Model / System → Experience

**The question.** Does a better system produce a noticeably better user experience?

**Why it matters.** A model that improves silently does nothing for retention. The improvement has to land where the user can feel it.

**Rubric.**

| Score | Descriptor | What it looks like |
|---|---|---|
| 1 | **Invisible** | Improvements don't translate into user-visible quality. Internal-only metric movement. |
| 2 | **Subtle** | Improvements show up only to power users or only on edge cases. Most users would not notice. |
| 3 | **Felt** | Better system produces noticeably better answers on real tasks. A returning user would describe the product as having "got smarter." |
| 4 | **Capability-unlocking** | Improvements unlock new capabilities or new use cases not previously possible. |
| 5 | **Primary differentiator** | The system's quality is the primary reason users return. Strip it out and the product no longer has a reason to exist. |

**Score: __/5**
**Rationale:**
**Evidence:**
**Falsifier:**
**Trajectory:** *Now: __/5 · M12: __/5 · M24: __/5*

---

### Loop 4 — Experience → Usage

**The question.** Does better UX drive more / returning usage?

**Why it matters.** This is what closes the loop. Without it, you have a product that gets better but doesn't grow — a hobby, not a flywheel.

**Rubric.**

| Score | Descriptor | What it looks like |
|---|---|---|
| 1 | **No retention** | Users try and leave. Cohort curves go to zero. |
| 2 | **Curiosity-driven** | Some return usage, mostly novelty-driven. No defined value-moment for return. |
| 3 | **Recurring value** | Recurring usage tied to specific value moments. Users come back for defined jobs. |
| 4 | **Compounding** | Usage compounds over time. Users use the product more as they accumulate context. They refer others. |
| 5 | **Network effects** | Value to one user increases as more users use the product. Self-reinforcing growth. |

**Score: __/5**
**Rationale:**
**Evidence:**
**Falsifier:**
**Trajectory:** *Now: __/5 · M12: __/5 · M24: __/5*

---

## Synthesis

**Total flywheel score: __/20** *(now)* · **__/20** *(M12 target)* · **__/20** *(M24 target)*

**Weakest loop.** *(name the loop, not the score)*

**Bottleneck logic.** *(in one paragraph: why is this the weakest, and what flow-on effect does it have on the other loops? A weak Loop 1 starves everything downstream; a weak Loop 4 means the system gets better for nobody.)*

**Fix for the weakest loop.** *(the one specific operational move that addresses it — not "we'll work on it," but "we will instrument X by date Y, measured by Z.")*

**The honest read.** *(In one paragraph: is this a flywheel, or a curated dataset with marketing? A real flywheel must show movement on every quarterly re-score. If two consecutive quarters pass without movement, the flywheel isn't real — re-evaluate the loop design.)*

---

## Competitive positioning

Most positioning maps fail because the team picks axes that flatter them. Pick the axes *consciously* — they are the ground on which you claim defensibility.

**Recommended axis pairs for AI products** (pick one, or design your own):

| Pair | When to use |
|---|---|
| **Specialization × Trust** *(vertical–horizontal × verified–probabilistic)* | When the product's value is "we are deeper than a generalist" or "we are more reliable than a chatbot." |
| **Geography × Action depth** *(local–global × informational–agentic)* | When local presence and ability to take action are the primary differentiators. |
| **Sovereignty × Generality** *(sovereign–global × focused–broad)* | When data residency and regulatory posture matter. |
| **Speed × Depth** *(real-time–batch × workflow–answer)* | When the product competes on response latency and integration depth. |

**Selected axes.**

- **Axis X:**
- **Axis Y:**
- **Why these two:** *(one sentence — what claim does this map make about where you compete?)*

**Map — current position and target moat.**

We plot the product **twice**: where we are today, and where the moat needs to be in 24 months. The arrow between the two is the strategy. A map with only one dot is a snapshot; a map with two dots is a plan.

| Competitor | X position | Y position | Notes |
|---|---|---|---|
| **Your product — today** | | | *(honest read of current position, not aspirational)* |
| **Your product — target moat (M24)** | | | *(where the moat must be after 24 months of work)* |
| | | | |
| | | | |
| | | | |

**The move.** *(One sentence describing the trajectory from today's dot to the M24 dot. What is changing on each axis, and why? "We are moving from X1 to X2 on the X-axis because [reason], and from Y1 to Y2 on the Y-axis because [reason]." If the move can't be stated in one sentence, the strategy is not yet sharp.)*

**The honest read.** *(One paragraph: at the M24 position, which competitors do you beat — at the cost of which? Which competitors beat you — and is that intentional? Is the move from today to M24 ambitious-but-credible, or wishful? Name the operational work that justifies the arrow.)*

---

## 90-day encroachment plan

The diagnostic asked "could a hyperscaler crush you?" The flywheel asks the operational follow-up: **if a serious attack starts on Monday, what are you doing on Tuesday?**

**Named attacker.**
*(One specific competitor, one specific announced or plausible move. Not "Big Tech" — "OpenAI ships X by Y." If you can't name it specifically, you can't plan against it.)*

**Attack timeline.**
- **Days 1–30:** *(announcement, press, influencer adoption)*
- **Days 31–60:** *(GA shipping, first wave of users defect)*
- **Days 61–90:** *(integration with adjacent platforms, default-status entrenchment)*

**Tripwire.**
*(The observable signal that tells you the attack has started. A specific event you would see — not a vibe. Without a tripwire, the plan triggers too late.)*

**Response by horizon.**

| Horizon | Move | Owner | Evidence of completion |
|---|---|---|---|
| **Week 1 — reactive** | *(public messaging, customer comms, sharpen the why-we-still-exist sentence)* | | |
| **Month 1 — product** | *(one feature shipped that anchors the difference — not pretty, but real)* | | |
| **Quarter 1 — structural** | *(a structural move that changes the competitive ground — partnership, exclusive integration, regulatory positioning, distribution lock)* | | |

**The honest read.** *(One paragraph: if the attack lands and the plan executes, do you survive at the same scale, a smaller scale, or as a different product? "Survive at scale" is rare and should be earned in the answer, not assumed.)*

---

## Review log

| Date | Reviewer | What changed |
|---|---|---|
| | | |
