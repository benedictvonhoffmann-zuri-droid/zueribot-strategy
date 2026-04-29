# Data Flywheel Map

> **Purpose.** Engineer the moat. Module 1's diagnostic asks *"is the data axis defensible?"*; this doc asks *"by what specific compounding mechanisms does that defensibility get built, where is the engine weakest, and how does it get stronger over time?"*
>
> **How to use it.** Score honestly at month 0. Re-score at every quarterly review. A flywheel that does not move is decaying — even if the score has not yet fallen.

---

## What is a data flywheel?

A data flywheel is a self-reinforcing loop in which using the product produces signal, the signal makes the system better, the better system produces better outputs, and the better outputs drive more usage — which produces more signal. Each turn of the loop adds energy. The longer the wheel spins, the harder it is for a competitor starting from zero to catch up.

```
                Users interact
                      │
                      ▼
                Capture signal
                      │
                      ▼
                System improves
                      │
                      ▼
                Better outputs
                      │
                      ▼
                  More usage  ──┐
                      │         │
                      └─────────┘
```

A static dataset is not a flywheel. A flywheel requires *all four arrows to be live*. If any one of them breaks, the wheel stalls — and the moat decays.

But a flywheel needs a *substrate to compound on*. Generic logs don't compound; specific kinds of data do. We score the four substrates that build durable AI moats.

---

## The four compounding mechanisms

| Mechanism | What it measures |
|---|---|
| **Correction** | Do users fix AI outputs? Is that signal captured and reused? |
| **Preference** | Does the product learn individual or team preferences over time? |
| **Domain context** | Does usage in one area improve quality in adjacent areas? |
| **Network** | Does each new user make the product better for everyone? |

Score each 1–5. **The weakest loop is where competitors attack** — it is the part of your moat that is not yet compounding.

---

## Timeframe

**Operational — 12 to 24 months.** This is a build plan: what compounds today, what we are constructing, what the system looks like when it works. Each loop is scored **today**, with **trajectory at M12 and M24** under the current plan. A flywheel that scores 4/20 today and 14/20 in 18 months is a *plan*; one that stays at 4/20 indefinitely is a wrapper.

---

## Connection to the diagnostic

This document is where Axis 2 of the diagnostic gets engineered. If the diagnostic says Axis 2 is the lever to pull, the flywheel score is what tells us we are pulling it. The two should track together — Axis 2 cannot move without the flywheel total moving.

> **Diagnostic reference:** Axis 2 (Data) — current __ / 5. Target M24: __ / 5. The flywheel total below should grow in step.

---

## Scoring rubric

Each loop is scored 1–5 against an anchored rubric. **Pick the descriptor that matches, then write the score.** No half-points. If between two, pick the lower — optimism is the default failure mode.

Every loop requires:
- **Rationale** — why this descriptor.
- **Evidence** — concrete artefact, number, or instrumented metric. If you can't name evidence, the score is a 1.
- **Falsifier** — the specific signal that would move this score up or down.
- **Trajectory** — current / month 12 / month 24, under the current plan.

---

### Loop 1 — Correction

**The question.** Do users fix AI outputs? Is that signal captured and reused to improve the system?

**Why it matters.** Corrections are the highest-quality signal a product can collect — the user has shown you exactly what was wrong and what would have been right. A correction loop is what turns a static system into a learning one.

**Rubric.**

| Score | Descriptor | What it looks like |
|---|---|---|
| 1 | **No capture** | No mechanism for users to flag or fix outputs. Wrong answers are invisible. |
| 2 | **Captured, unused** | Feedback widget exists; signal sits in a database with no consumer. |
| 3 | **Manual cadence** | Corrections drive periodic manual updates (KB edits, prompt tweaks, retrieval rule changes) on a defined cadence. |
| 4 | **Instrumented pipeline** | Corrections feed a structured improvement loop (evals, retrieval refresh, fine-tune cycle) running automatically on a regular schedule. |
| 5 | **Automated retraining** | Corrections directly train or tune the model in production. The system measurably improves with every batch, no human in the loop. |

**Score: __/5**
**Rationale:**
**Evidence:**
**Falsifier:**
**Trajectory:** *Now: __/5 · M12: __/5 · M24: __/5*

---

### Loop 2 — Preference

**The question.** Does the product learn individual or team preferences over time, and does that learning shape future outputs in a way the user feels?

**Why it matters.** A product that knows you is a product that is hard to leave. Preference is where the moat moves from "the product is good" to "the product is good *for me specifically*."

**Rubric.**

| Score | Descriptor | What it looks like |
|---|---|---|
| 1 | **Stateless** | No memory between sessions. Every user gets the same product. |
| 2 | **Surface profile** | Basic profile fields stored (name, address) but they don't meaningfully shape outputs. |
| 3 | **Stated preference** | The product remembers preferences the user has explicitly set ("prefer short answers", "vegetarian"), and applies them. |
| 4 | **Behavioural inference** | The product infers preferences from behaviour and adapts proactively. The user notices the system getting more aligned. |
| 5 | **Deep personalization** | Every user gets a meaningfully different product. Personalization is itself a primary differentiator and a switching cost. |

**Score: __/5**
**Rationale:**
**Evidence:**
**Falsifier:**
**Trajectory:** *Now: __/5 · M12: __/5 · M24: __/5*

---

### Loop 3 — Domain context

**The question.** Does usage in one area of the product improve quality in adjacent areas? Does signal cross-pollinate, or does each domain stand alone?

**Why it matters.** Cross-domain transfer is what turns a multi-product into a *system*. A generalist competitor can replicate any single domain; the competitive defense is that *all of them together* form a context no individual domain could produce alone.

**Rubric.**

| Score | Descriptor | What it looks like |
|---|---|---|
| 1 | **Siloed** | Each domain is an island. Improvements don't transfer. The product is a bundle of disconnected features. |
| 2 | **Shared stack** | Common infrastructure but no cross-pollination of signal between domains. |
| 3 | **Adjacent transfer** | Improvements in one domain (e.g. waste pickup) measurably inform adjacent ones (e.g. recycling guidance). |
| 4 | **Multi-domain compounding** | Signal from any one domain improves quality across multiple others. The product is meaningfully more than the sum of its features. |
| 5 | **Cross-domain transfer** | The product's quality in any domain depends on signal from all others. Domains are mutually reinforcing — and that interdependency is itself the moat. |

**Score: __/5**
**Rationale:**
**Evidence:**
**Falsifier:**
**Trajectory:** *Now: __/5 · M12: __/5 · M24: __/5*

---

### Loop 4 — Network

**The question.** Does each new user make the product better for *everyone*? Or do users sit in isolation?

**Why it matters.** A network effect is the strongest moat there is — value compounds across the user base, not just within an individual account. But it is also the rarest, and most products that claim network effects actually have something else (usually Correction at scale, relabelled).

**Rubric.**

| Score | Descriptor | What it looks like |
|---|---|---|
| 1 | **Isolated** | Users do not benefit each other in any way. The product is one-to-one. |
| 2 | **Aggregate visibility** | Basic shared statistics visible ("popular this week") but no real value transfer between users. |
| 3 | **Indirect benefit** | More users → more corrections / contributions / data → better product for everyone. (Note: this is mostly Correction at scale, not a true network effect.) |
| 4 | **Direct user-to-user value** | Explicit value flows between users — shared content, collective recommendations, community Q&A surfaced inside the product. |
| 5 | **Strong network effects** | Value to any one user increases meaningfully as more users join. The product would not work with a small user base; it gets better as it grows. |

**Score: __/5**
**Rationale:**
**Evidence:**
**Falsifier:**
**Trajectory:** *Now: __/5 · M12: __/5 · M24: __/5*

---

## Synthesis

**Total flywheel score: __/20** *(now)* · **__/20** *(M12 target)* · **__/20** *(M24 target)*

**Weakest loop.** *(Name the loop, not the score. The weakest loop is where competitors attack.)*

**Bottleneck logic.** *(In one paragraph: why is this the weakest, and what flow-on effect does it have on the others? A weak Correction loop starves the system of improvement signal; a weak Network loop caps how much the moat can compound; a weak Preference loop means switching cost stays low; a weak Domain Context loop means each feature has to defend itself alone.)*

**Fix for the weakest loop.** *(The one specific operational move that addresses it — not "we'll work on it," but "we will instrument X by date Y, measured by Z.")*

**The honest read.** *(In one paragraph: is this a flywheel, or a curated dataset with marketing? A real flywheel must show movement on every quarterly re-score. If two consecutive quarters pass without movement, the flywheel isn't real — re-evaluate the loop design. Also be honest about which loops are *structurally capped* by your product values or market shape — some products will never have a 5 on Network, and that is fine if the other loops carry.)*

---

## Competitive positioning

Most positioning maps fail because the team picks axes that flatter them. Pick the axes *consciously* — they are the ground on which you claim defensibility.

**Recommended axis pairs for AI products** (pick one, or design your own):

| Pair | When to use |
|---|---|
| **Specialization × Trust** *(vertical–horizontal × verified–probabilistic)* | When the product's value is "we are deeper than a generalist" or "we are more reliable than a chatbot." |
| **Geography × Action depth** *(local–global × informational–agentic)* | When local presence and ability to take action are the primary differentiators. |
| **Sovereignty × Generality** *(sovereign–global × focused–broad)* | When data residency and regulatory posture matter. |
| **Sovereignty × Agency** *(sovereign–global × informational–agentic)* | When sovereignty AND multi-domain action depth are jointly the moat. |
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
