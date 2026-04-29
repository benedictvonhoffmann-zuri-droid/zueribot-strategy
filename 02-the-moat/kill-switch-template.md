# Kill Switch — Provider Portability Audit

> **Purpose.** Make the AI stack swappable. The diagnostic asks *"are you exposed to the platform?"*; this doc asks the operational follow-up: *"if your primary provider doubles pricing, deprecates a model, or ships terms you cannot accept — how long until you are running on someone else, with the same quality bar?"*
>
> **Default benchmark.** *If you can't swap in 48 hours, that is not a strategy — it is a dependency.* Adjust the benchmark only with a stated reason. For some products 48h is unrealistic (heavy fine-tunes, regulated stacks); for others it is lazy (a chat wrapper should swap in an afternoon).
>
> **How to use it.** Score every component honestly. Run the stress test as a thought experiment. Define three concrete actions. Re-audit every quarter — and ideally run a swap drill at least once a year (a real fire-drill where you swap to the alternate provider in staging and verify the eval suite passes).

---

## Connection to the diagnostic

Axis 3 of the diagnostic (Platform exposure) is the *strategic* question. This doc is the *engineering* one. Sovereignty as a strategic posture is meaningless if the runtime is irreversibly wired to a single hyperscaler.

> **Diagnostic reference.** Axis 3 (Platform Exposure) — current __ / 5. The kill switch is the operational expression of Axis 3 — without it, any sovereignty claim is positioning, not infrastructure.

---

## The three components

A swap-ready stack rests on three things. Each is scored 1–5 against an anchored rubric. Pick the descriptor that matches; no half-points; if between two, pick the lower.

Every component requires:
- **Rationale** — why this descriptor.
- **Evidence** — concrete artefact (a file, a config, a runbook, a passing CI step). If you can't name evidence, the score is a 1.
- **Falsifier** — what would move the score up or down.

---

### Component 1 — Abstraction Layer

**The question.** Are provider-specific calls quarantined behind a typed interface, or scattered through product code?

**Rubric.**

| Score | Descriptor | What it looks like |
|---|---|---|
| 1 | **Hardcoded** | Direct provider SDK calls scattered through product code. Provider-specific request/response shapes leak into application logic. |
| 2 | **Wrapped** | A single helper function or two; some abstraction, but provider-specific concerns still leak (model names, prompt format, response handling). |
| 3 | **Interface** | All model calls flow through a typed interface. Provider-specific code lives in adapters. New provider = new adapter, not surgery. |
| 4 | **Prompt-as-data** | Interface + prompts versioned as data (not strings in code) + per-task config + structured logging of every call. |
| 5 | **Hot-swappable** | Interface + prompt-as-data + retries + circuit-breakers + per-environment config + traffic-shifting at runtime. Swapping providers is a config change. |

**Score: __/5** **Rationale: ___** **Evidence: ___** **Falsifier: ___**

---

### Component 2 — Multi-Model Routing

**The question.** Does the stack route per-task by cost / latency / quality? Does it fail over automatically if a provider goes dark?

**Rubric.**

| Score | Descriptor | What it looks like |
|---|---|---|
| 1 | **Single provider** | One provider hardcoded for everything. Outage = outage. |
| 2 | **Two providers, manual** | More than one provider in the stack, but routing decisions are made by humans, not code. |
| 3 | **Task-aware routing** | Different tasks (voice / tools / embeddings / classification) routed to different providers based on task fit. |
| 4 | **Cost / latency / quality routing** | Per-call routing decisions on cost, latency, or measured quality. Tasks find the cheapest provider that meets the SLA. |
| 5 | **Automatic failover** | All of the above + automatic failover on provider failure. Outage on the primary causes traffic to shift; users do not notice. |

**Score: __/5** **Rationale: ___** **Evidence: ___** **Falsifier: ___**

---

### Component 3 — Eval Harness

**The question.** Can you switch providers with confidence — or is "the new one feels worse" the only signal you have?

**Rubric.**

| Score | Descriptor | What it looks like |
|---|---|---|
| 1 | **No tests** | No automated evals. Quality assessment is vibes-based. |
| 2 | **Spot-check golden cases** | A handful of canonical examples reviewed manually before a release. |
| 3 | **Golden dataset, manual run** | A defined eval set (≥50 cases covering core tasks) that can be run on demand. Pass/fail criteria documented. |
| 4 | **Automated regression suite** | Eval suite runs in CI on every prompt or model change. Regression on any case fails the build. |
| 5 | **Continuous + adversarial** | All the above + adversarial cases generated from production failures + dashboards tracking eval drift over time. |

**Score: __/5** **Rationale: ___** **Evidence: ___** **Falsifier: ___**

---

## The stress test

Provider risk is not just pricing. The doc that only thinks about pricing has optimised the wrong defence. Six realistic shocks, in order of decreasing frequency:

| # | Shock | Time-to-swap target | What breaks first |
|---|---|---|---|
| 1 | **Pricing 2–10× overnight** | 48h | Margin. Then product if pricing is unsustainable. |
| 2 | **Model-version deprecation** | 30 days *(typical notice period)* | Silent quality regression on the new version if no eval coverage. |
| 3 | **Use-case restriction** *(new content / safety policy)* | 14 days | Specific prompts begin to refuse or degrade. Hard to detect without monitoring. |
| 4 | **Provider acquisition / terms change** | 90 days *(typical migration window)* | Strategic posture if the acquirer is misaligned (US-hosted, surveillance-adjacent, etc.). |
| 5 | **Multi-day outage** | hours | Product availability. Users see broken responses or timeouts. |
| 6 | **Geopolitical shock** *(provider becomes politically uncomfortable)* | varies | The brand promise, especially for sovereignty-positioned products. |

For each shock, name the impact on **margin**, **product**, **brand**, and **time-to-swap** in the audit table below.

---

## The forgotten lock-in surfaces

Most kill-switch audits stop at "which LLM provider." Real lock-in lives in five places that look like infrastructure, not vendors:

1. **Embeddings.** Your vector DB is tied to whatever model embedded it. Swapping embedding providers means re-embedding the entire knowledge base — possibly weeks of compute and a quality regression to manage.
2. **Fine-tunes.** Provider-specific fine-tunes do not port. A fine-tune you depend on can become a multi-month rebuild.
3. **Tool / function-calling format.** OpenAI, Anthropic, Mistral, Apertus all differ subtly in tool schemas and parsing. A function-call-heavy product has more lock-in than a chat product.
4. **Guardrail / safety layers** baked into the provider's API (moderation endpoints, prompt-injection mitigations).
5. **Observability / tracing stacks** tied to specific platforms (LangSmith, Helicone, etc.).

Audit these explicitly. A kill switch that ignores them is theatrical.

---

## The audit table

| Surface | Current provider | Swappable? | Time-to-swap (today) | Time-to-swap (target) | Notes |
|---|---|---|---|---|---|
| **Voice / answer generation** | | Y / N | | | |
| **Tools / function-calling** | | Y / N | | | |
| **Embeddings** | | Y / N | | | |
| **Fine-tunes** | | Y / N | | | |
| **Guardrails** | | Y / N | | | |
| **Observability** | | Y / N | | | |

---

## Sovereignty-preserving swap options *(use if your strategic posture is sovereignty-, regulatory-, or values-positioned)*

Most products treat swap options as a quality / cost question. For a sovereignty-positioned product, swap options have a *second axis*: does the swap preserve the strategic posture, or break it?

| From | To | Quality preserved? | Sovereignty preserved? | Acceptable as: |
|---|---|---|---|---|
| | | Y / partial / N | Y / partial / N | *(default / bridge / emergency only / never)* |
| | | | | |
| | | | | |

A swap that preserves quality but breaks sovereignty (e.g. running on a US hyperscaler) is acceptable in an emergency but kills the strategy if it becomes default. Name the acceptable tier per option.

---

## Stress-test analysis

For each shock, against your current stack:

| # | Shock | Days to swap | What breaks (margin / product / brand) | Mitigation |
|---|---|---|---|---|
| 1 | Pricing 2–10× | | | |
| 2 | Model deprecation | | | |
| 3 | Use-case restriction | | | |
| 4 | Acquisition / terms change | | | |
| 5 | Multi-day outage | | | |
| 6 | Geopolitical shock | | | |

**Biggest blocker.** *(One sentence: the single thing that, if fixed, would most reduce time-to-swap across multiple shocks.)*

---

## Three actions

Each with a named owner and an evidence-of-completion (a file, a passing test, a runbook link). "We will work on it" does not count.

| Horizon | Action | Owner | Evidence-of-completion |
|---|---|---|---|
| **This week** | | | |
| **This month** | | | |
| **This quarter** | | | |

---

## Swap drill

Once a year, run a real fire-drill: swap to the alternate provider in staging, run the eval suite, report results. Audits prove paper readiness; drills prove operational readiness. **Last drill date: ___**

---

## Portability score

**Total: __/15** *(component scores summed)*. **Status:** *(Ready / Partial / Locked)*.

> **Ready** = ≥12/15 + green stress test on all six shocks + drill within last 12 months.
> **Partial** = 7–11/15 or any single component at 1.
> **Locked** = ≤6/15 or two components at 1.

**The honest read.** *(One paragraph: where are you, where do you need to be, what is the work?)*

---

## Review log

| Date | Reviewer | What changed |
|---|---|---|
| | | |
