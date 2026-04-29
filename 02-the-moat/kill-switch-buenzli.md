# Kill Switch — Bünzli

> Filled instance of [`kill-switch-template.md`](kill-switch-template.md). Snapshot dated 2026-04-29 against the current `main` branch of `zueribot`.
>
> **Headline finding.** Bünzli is **pre-launch** — no production users today, so this is a *building* audit, not an *exposure* audit. But: the prototype runtime currently defaults to Anthropic Claude Haiku 4.5 (US-hosted hyperscaler), while the sovereignty-anchored strategy (diagnostic Axis 3 = 4/5) and the orchestration plan ("Apertus voice + Mistral tools") point elsewhere. The kill switch is therefore the engineering checklist that **must close before v1 ships** — not a present-tense problem, but a release-gate one. Until that gap closes, the sovereignty story is positioning, not infrastructure.

---

## Connection to the diagnostic

> **Diagnostic reference.** Axis 3 (Platform Exposure) — currently scored **4/5** ("Not worth their focus"). The score assumes Bünzli's stack reflects the sovereignty posture *at launch*. The prototype runtime does not yet — but pre-launch that is acceptable. **The release gate is: Axis 3's promised infrastructure must be in place before v1 goes to users.** If v1 ships on Anthropic-default, the diagnostic score should be revised down at the next quarterly review.

---

## The three components

### Component 1 — Abstraction Layer

**Score: 3 / 5 — Interface.**

**Rationale.**
LLM access is configurable via env vars (`LLM_PROVIDER`, `LLM_MODEL`, `LLM_BASE_URL`, `LLM_API_KEY`) and the rest of the codebase calls a single `get_llm()` factory. Provider-specific concerns live behind LangChain / LangGraph adapters; `agent.py` does not contain direct Anthropic / OpenAI SDK calls. That is solidly an "Interface" pattern.

We do not score 4 because **prompts are strings in code, not versioned data** — `SYSTEM_PROMPT` and `GRADER_PROMPT` live inline in `agent.py`. A real prompt-as-data layer would version them, hash them for cache busting, and surface them per-task config.

**Evidence.** `backend/config/settings.py` (env-driven provider config); `backend/agent.py` (`get_llm().bind_tools(...)` pattern); no direct provider SDK imports outside `settings.py`.

**Falsifier.**
- **Drops to 2** if direct provider SDK calls are found in product code outside the adapter.
- **Rises to 4** once prompts are extracted as versioned data + per-task config + structured logging on every call.

### Component 2 — Multi-Model Routing

**Score: 1 / 5 — Single provider.**

**Rationale.**
A single `LLM_PROVIDER` env var sets the provider globally — every task (voice, tools, grading, summarisation) hits the same model. The "Apertus voice + Mistral tools + browser-local state" decision exists in memory but is not yet implemented. This is the score where strategy and runtime are most clearly out of sync.

**Evidence.** Single `get_llm()` returning one configured provider per process; no per-task routing logic in `agent.py`; no failover mechanism.

**Falsifier.**
- **Rises to 3** once tasks (voice / tools / grading / classification) route to different providers per task fit.
- **Rises to 4** once routing decisions also factor cost/latency/quality.
- **Rises to 5** once automatic failover on provider outage is implemented and tested.

### Component 3 — Eval Harness

**Score: 1 / 5 — No tests.**

**Rationale.**
No automated eval suite exists. Quality assessment is currently vibes-based (manual prototype testing). This is **the single biggest blocker** on this entire audit — without an eval harness, every potential swap is a leap of faith, and you cannot tell silent quality regressions on model deprecation from genuine improvements.

**Evidence.** No `tests/eval/` or `evals/` directory in the repo; no eval-related CI step; no documented golden dataset.

**Falsifier.**
- **Rises to 3** once a golden dataset of ≥50 cases (covering Swiss German voice, agentic tool calls per schema, civic accuracy, safety) exists and can be run on demand.
- **Rises to 4** once that suite runs in CI on every prompt or model change.

---

## The audit table

| Surface | Current provider | Swappable? | Time-to-swap (today) | Target | Notes |
|---|---|---|---|---|---|
| **Voice / answer generation** | Anthropic Claude Haiku 4.5 *(default)* | Y *(env-driven)* | Technically hours; *practically* weeks (no eval to prove quality) | 48h | Strategy says Apertus; runtime says Anthropic. Gap. |
| **Tools / function-calling** | Anthropic via LangChain `bind_tools(...)` | Y but coupled to LangChain's tool-spec abstraction | Days | 48h | LangChain handles the cross-provider format mapping — that abstraction itself is a lock-in surface. |
| **Embeddings** | Google EmbeddingGemma-300M *(self-hosted via HuggingFace)* | N *(re-embed required)* | Weeks (re-embed entire KB; ~41,714 chunks) | Weeks (acceptable) | Self-hosted = no API lock-in, good for sovereignty. The lock-in is the sunk cost of the existing Qdrant index, not a vendor relationship. |
| **Fine-tunes** | None | N/A | — | — | None today. If we add one (e.g. Swiss German voice fine-tune of Apertus), this row becomes critical. |
| **Guardrails** | None *(provider-level only)* | N/A | — | — | We do not run a guardrail layer — we rely on the provider's safety. Worth a future review. |
| **Observability** | LangSmith *(implicit via LangChain)* | Partial — would lose tracing on swap to non-LangChain stack | Days | TBD | LangChain dependency couples observability to the framework. Significant lock-in surface that the slide-level audit misses. |

---

## Sovereignty-preserving swap options

| From | To | Quality preserved? | Sovereignty preserved? | Acceptable as |
|---|---|---|---|---|
| Anthropic Claude *(current default)* | **Apertus** *(Swiss)* | TBD — likely comparable on Swiss German, possibly lower on general reasoning. Without eval, unknown. | **Improved** *(US → CH)* | **Default for v1** — this is the swap the strategy demands |
| Anthropic Claude | Mistral *(EU)* | Likely comparable | Improved *(US → EU but not Swiss)* | **Bridge** — acceptable while Apertus capacity / quality matures |
| Anthropic Claude | OpenAI | Comparable | Same (US → US) | **Emergency only** — defeats the entire sovereignty positioning |
| Anthropic Claude | Self-hosted Llama / Apertus open-weights *(on Infomaniak)* | Lower at Bünzli's scale today | **Maximum** | **Long-game ideal** — the v3 sovereignty endgame |
| Mistral *(target — tools)* | Apertus | Comparable for European tasks | Improved *(EU → CH)* | **Default fallback** if Mistral terms ever conflict |
| Mistral | OpenAI | Higher general | Reduced *(EU → US)* | **Emergency only** |

The matrix says something important: **the only swap that preserves sovereignty without quality risk is the one we have not yet built (Apertus).** Every emergency-only option breaks the brand. The kill switch's job is to make the default option (Apertus) actually work — the rest are insurance.

---

## Stress-test analysis *(against current stack)*

| # | Shock | Days to swap | What breaks | Mitigation |
|---|---|---|---|---|
| 1 | Anthropic pricing 2–10× | Weeks *(no eval to validate Apertus / Mistral as alternatives)* | Margin first, product if unsustainable | **Eval harness** unblocks Apertus / Mistral as real alternatives, dropping swap time to days |
| 2 | Claude Haiku model-version deprecation | 30 days notice typical, but **silent quality regression** if no eval coverage | Quality drifts without anyone noticing until users complain | **Eval harness** — the same mitigation closes shock #1 and #2 |
| 3 | Use-case restriction *(content-policy change)* | 14 days | Specific civic / agentic prompts begin to refuse or over-cautious | Low likelihood for Bünzli's content profile (civic + commercial = benign), but worth monitoring |
| 4 | Anthropic acquisition / terms change | 90 days *(typical)* | **Brand promise** — sovereignty story breaks publicly | Apertus path must be production-ready by then or the brand dies during the migration window |
| 5 | Anthropic multi-day outage | Hours | Product availability — no automatic failover | Multi-model routing with failover (component 2 → 4) closes this |
| 6 | **Geopolitical shock** *(US provider becomes politically uncomfortable for Swiss / EU sovereignty markets)* | varies | **The whole bet** — sovereignty positioning becomes hollow | This is Bünzli's most strategically dangerous shock. The kill switch is the response. **Closing the runtime/strategy gap is the mitigation, full stop.** |

**Biggest blocker: no eval harness.** Of the six shocks, four are mitigated primarily by having an eval harness (1, 2, 4, 6 all turn on "can we prove a swap preserves quality?"). Building the harness first unlocks every other defensive move.

---

## Three actions

| Horizon | Action | Owner | Evidence-of-completion |
|---|---|---|---|
| **This week** | Document the actual stack: every model call, every prompt, every config. Lock in the path from "Apertus voice + Mistral tools" decision to runtime — write the implementation plan with concrete commits. | Benedict | Markdown in `backend/README.md` or repo `ARCHITECTURE.md` listing per-task model usage + a v1.x → v1.y migration plan. |
| **This month** | Ship the eval harness baseline. ≥50 golden cases covering: Swiss German voice quality, agentic tool calls (Züri wie neu, ERZ, Quandoo schemas), civic factual accuracy, refusal / safety. Run against current Anthropic stack to establish baseline. | Benedict + Claude (engineering session) | `evals/` directory + a passing CI step + a baseline-results JSON committed |
| **This quarter** | Wire Apertus + Mistral live behind the existing abstraction. Run the eval against each and publish the per-task quality delta. Run the **first swap drill** — swap Anthropic → Apertus in staging, run the suite, report results. | Benedict | Two new adapters merged + eval-comparison report committed + drill report in `02-the-moat/swap-drill-2026-Q3.md` |

---

## Swap drill

Once a year minimum (target: quarterly during the M0–M12 push). **Last drill: never.** First drill scheduled: end of this quarter, as part of the quarterly action above.

---

## Portability score

**Total: 5 / 15.** Components: 3 + 1 + 1.
**Status: Locked.** *(per template thresholds: ≤6/15 or two components at 1 → Locked.)*

**The honest read.**
The abstraction layer is in good shape; routing and eval are not. The score is "Locked" but read it in pre-launch context: Bünzli has no production users today, so this is a building audit and the locked status is acceptable *now*. It will not be acceptable at launch. The single highest-leverage move is the eval harness — it unblocks the runtime/strategy alignment, the failover work, the model-deprecation defence, and the swap drill. Build it this month. The release gate for v1 is: **Apertus (or another Swiss/EU sovereign provider) is the default in production, and we have a passing eval suite that proves it.** Shipping v1 on an Anthropic default would publicly contradict the sovereignty positioning the rest of the strategy depends on, and would force us to revise Axis 3 of the diagnostic downward. The kill switch is not theatre — for Bünzli, it is the engineering closure of the brand promise *at launch*, not a fire-drill for a stack already in production.

---

## Note for project memory

The memory file `project_orchestration_v1.md` records "code-orchestrated, no LangChain, Apertus voice + Mistral tools." The runtime as of this audit is LangChain / LangGraph + Anthropic Claude default. The decision is preserved; the implementation has not yet caught up. **Memory should be updated to reflect that the orchestration v1 is currently a target architecture, not a shipped one.**

---

## Review log

| Date | Reviewer | What changed |
|---|---|---|
| 2026-04-29 | Benedict + Claude (Opus 4.7) | Initial fill. Discovered strategy/runtime gap — runtime defaults to Anthropic, strategy claims sovereignty. Score 5/15 (Locked). Eval harness named as the single biggest blocker. |
