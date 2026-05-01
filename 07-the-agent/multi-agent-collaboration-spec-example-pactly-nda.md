# Multi-Agent Collaboration Spec — NDA Helper Agent (Pactly)

> **Example fill** of [`multi-agent-collaboration-spec-template.md`](multi-agent-collaboration-spec-template.md). Built for the Productschool Advanced AI Agents course final project — *not* a Bünzli artefact. Companion to [`agent-workflow-spec-example-pactly-nda.md`](agent-workflow-spec-example-pactly-nda.md): the AW Spec defines what each agent does individually; the MAC Spec defines how they coordinate.
>
> **v0.2 design revision (2026-05-01).** Aligned with the v0.2 AW Spec. The embedding-based Retriever is removed entirely — see *Design revision* below.

---

## Identity

- **System name:** NDA Helper Agent — Pactly Internal Tools
- **Owner:** Senior PM, Pactly Internal Tools division
- **Spec version:** v0.2 (architecture simplified — see *Design revision*)
- **Last updated:** 2026-05-01
- **Status:** Phase 1 validated in LangChain; Phase 2 (Planner + checklist) in progress.
- **Related AW Spec:** [`agent-workflow-spec-example-pactly-nda.md`](agent-workflow-spec-example-pactly-nda.md)

---

## The spec table

| Field | Spec |
|---|---|
| **Agents involved** | **Four LLM-driven agents**, in this order: (1) **Scope Classifier** (Haiku 4.5) — gates whether the question is "first-pass" answerable. (2) **Planner** (Haiku 4.5) — generates the standard-NDA-clause checklist for exhaustive coverage. (3) **Answer Agent** (Sonnet 4.7) — reads full NDA + question + checklist; composes plain-English response with verbatim citations. (4) **Critic** (SaulLM-7B) — verifies grounding *and* checklist coverage. Different model family from the Answer Agent for genuinely independent verification. *(No Retriever — the document fits in context. See Design revision.)* |
| **Orchestration type** | **Sequential pipeline with a terminal critic-reflexion loop.** Linear flow: Scope → Plan → Answer ↔ Critic. The Answer↔Critic step has at most **one** reflexion iteration; if the Critic fails twice, the system escalates rather than continuing to revise. Not blackboard, not hierarchical — explicit deterministic ordering with a single feedback loop at the verification stage. |
| **Data format** | Five typed handoff contracts (full schemas in *Handoff schemas* below). All contracts are schema-validated at the boundary — if validation fails, the receiving agent does not start and the orchestrator escalates. |
| **Communication channel** | **LangChain LCEL chains + LangGraph state** for the build. The orchestrator is a LangGraph `StateGraph` with named nodes per agent and explicit conditional edges (Scope outcome routes to Planner or to refusal; Critic verdict routes to Deliver, Reflexion, or Escalation). Per-run shared state lives in the LangGraph state object, which functions as a per-run blackboard. *Langflow is the demo-deliverable target — the same graph re-laid in visual nodes for Slide 1.* |
| **Trigger logic** | Mostly **sequential complete** + one **conditional output** (Scope = in_scope → Planner; Scope = out_of_scope → terminate). The reflexion loop is **conditional + max-iteration-capped** (Critic = fail AND retries < 1 → re-fire Answer). |
| **Fallback logic** | Per agent — Scope/Planner failure: retry once → abort with generic "couldn't process, contact Legal." PDF parsing failure: abort with "this PDF is not readable, please re-upload" (OCR fallback deferred). Answer Agent failure: retry once → fall back to "verbatim relevant clauses with no plain-English synthesis" + escalate. **Critic failure: HITL escalate. Never ship without verification.** System-level: any unrecoverable failure = "Send to Legal" with the user's question + the PDF preserved in the handoff to a human reviewer. |
| **Governance hooks** | Inherited from the AW Spec (Scope check, Critic verification, HITL-by-UX visual citation, weekly red-team). MAC-Spec-level additions: **schema validation at every handoff** (catches contract drift before it corrupts downstream silently); **PII-redaction check at every persistent write** (described in *Memory architecture*). |

---

## Orchestration detail

Visual flow:

```
[User upload + question]
          │
          ▼
   ┌──────────────────────┐
   │  0. Scope Classifier │  Haiku 4.5
   └──────────┬───────────┘
              │ in_scope = true
              ▼
   ┌──────────────────────┐
   │  1. Parse PDF        │  PyPDF (page-level)
   └──────────┬───────────┘
              │ full_text + page_metadata
              ▼
   ┌──────────────────────┐
   │  2. Planner          │  Haiku 4.5
   └──────────┬───────────┘
              │ clause_checklist
              ▼
   ┌──────────────────────┐
   │  3. Answer Agent     │  Sonnet 4.7
   └──────────┬───────────┘   (full PDF + question + checklist)
              │ draft + citations + coverage_report
              ▼
   ┌──────────────────────┐
   │  4. Critic           │  SaulLM-7B
   └──────────┬───────────┘
              │
       ┌──────┴──────┐
   pass│             │ fail (≤1 retry)
       ▼             ▼
  [Deliver]    [Re-fire 3 with feedback]
                     │ second fail
                     ▼
              [Escalate to Legal]
```

---

## Handoff schemas

| From → To | Schema |
|---|---|
| Scope Classifier → orchestrator | `{ in_scope: bool, refusal_reason?: string, suggested_legal_route?: string }` |
| PDF parser → run scratch | `{ full_text: string, pages: [{ page_number: int, text: string }], doc_id: string }` |
| Orchestrator → Planner | `{ question: string, doc_summary: string }` |
| Planner → orchestrator | `{ clause_checklist: [{ clause_type: string, what_to_check: string, priority: "H"|"M"|"L" }], question_specific_topics: string[] }` |
| Orchestrator → Answer Agent | `{ question: string, full_text: string, pages: [...], clause_checklist: [...] }` |
| Answer Agent → Critic | `{ draft_answer: string, citations: [{ quote: string, page: int }], confidence: float, coverage_report: [{ checklist_item: string, status: "addressed"|"not_present_in_doc"|"skipped" }], doc_id: string, question: string }` |
| Critic → orchestrator | `{ verdict: "pass" \| "fail", grounding_violations: [...], coverage_gaps: [...], suggested_revisions?: object, retry_count: int }` |

Every contract is schema-validated at the boundary. A handoff that fails validation does not invoke the receiving agent — the orchestrator routes to the system-level fallback (escalate to Legal).

---

## Memory architecture

> **The posture: stateless-by-design for user data, aggregate-only for system learning.** No per-user PDFs, no per-user question history, no per-user answer logging — and **no Personally Identifiable Information** persisted past the run, full stop. PDFs may contain PII (party names, signatures, addresses, contact information); we do not extract, log, or persist any of it beyond the session run. All in-memory state is wiped on session end. Aggregate patterns (critic failures, refusal patterns) are PII-stripped at write time. This is the privacy story we tell General Counsel directly: *"We don't store your contract, we don't log your questions, and we don't keep any personal information beyond the time it takes to answer."*

### Memory stores

| Store | Type | Lifetime | Who writes | Who reads | Pruning rule | Privacy posture |
|---|---|---|---|---|---|---|
| **Working memory (per LLM call)** | Working | One LLM call | Each agent's prompt | The same agent | N/A — recreated per call | Standard; ephemeral |
| **Run scratch (LangGraph state)** | Shared blackboard | One run | All four agents + PDF parser | All four agents | Deleted at run end | Wiped on completion or abort. PII may be present briefly in PDF text but never persisted. |
| **Session: parsed PDF text + page metadata** | Session | One browser session | Set at upload (PyPDF parser) | Answer Agent (for context), Critic (for grounding verification) | Deleted on session end / tab close / new upload | Per-session only; never written to disk; never crossed-user. *No embedding index — see Design revision.* |
| **Eval set (50 NDAs)** | System persistent | Indefinite, monthly refresh | Legal team (curators) | Eval CI; PM; engineering | Refresh / replace cycle; failures from production promoted to eval | **PII-redacted at curation time** by Legal team. Synthetic NDAs preferred; real ones only with redaction. |
| **Critic-failure patterns (aggregate)** | System persistent | Indefinite | Critic (writes after each "fail" verdict) | Planner (informs the standard-clause checklist content), PM | FIFO with importance-weighted decay over 90 days | **Aggregate only.** Per-user attribution stripped at write time. Only the *clause type* and *failure mode* are stored, never the source PDF or the question. |
| **Refusal patterns (aggregate)** | System persistent | Indefinite | Scope Classifier | Scope Classifier prompt-improvement, PM | Same FIFO + decay rule | **Aggregate only.** Question categories tracked, never the literal question text. |
| **Standard clause checklist** | System persistent | Indefinite, curated by Legal | Legal team (curators) | Planner | Versioned; reviewed quarterly | Not user data — system reference content. |

### What we deliberately do not remember

Equally important — the "stateless-by-design" list, named explicitly so it survives every refactor:

- **User-uploaded PDFs after the session ends.** Wiped from session storage on tab close. Never written to disk.
- **The literal text of user questions.** Aggregate categories of refused questions are kept (for the refusal-pattern store), but never the user's actual phrasing.
- **The literal text of agent answers.** Aggregate metrics (correct-answer rate, false-confidence rate) are kept; the answers themselves are not logged.
- **Any Personally Identifiable Information** (party names, signatures, addresses, email addresses, phone numbers, dates of birth, identification numbers) — never extracted, never logged, never persisted past the run. PDFs are processed as opaque text; PII present in that text is discarded with the rest of the run state.
- **Per-user history.** No "your last 5 NDAs" view, ever. Privacy posture, not a missing feature.
- **Embeddings of any kind.** The architecture deliberately does not build an embedding index — see Design revision. Beyond being unnecessary for the use case, it would be infrastructure that lives only seconds and adds no compounding value.

This list is the privacy story. It is also a constraint — no per-user personalisation, no preference learning across sessions, no "we remember you prefer this format." That trade is deliberate and correct for an internal legal-adjacent tool with a privacy-conscious General Counsel.

---

## Termination conditions

| Condition | Trigger | Result |
|---|---|---|
| **Success** | Critic returns `pass` | Render the answer with HITL-by-UX visual citations |
| **Out of scope** | Scope Classifier returns `in_scope: false` | Polite refusal with `suggested_legal_route` |
| **Unparseable PDF** | PDF parser fails / extracts no text | Abort with "this PDF is not readable, please re-upload" |
| **Coverage gap** | Critic flags missing checklist coverage that the Answer Agent did not flag itself | Escalate to Legal with the missing items |
| **Critic max-loop** | Critic returns `fail` after 1 reflexion retry | Escalate to Legal with the draft + critic reasons |
| **Hard failure** | Unrecoverable error in any agent | Abort + log + escalate to Legal |
| **Time budget exhausted** | Run exceeds 60s wall-clock *(reduced from v0.1 — no embedding step)* | Abort with "this question is taking unusually long; routing to Legal" |
| **Cost budget exhausted** | Run exceeds $0.50 hard cap *(reduced from v0.1)* | Abort + alert engineering |

---

## Conflict resolution

| Scenario | Resolution rule |
|---|---|
| **Critic disagrees with Answer Agent (verdict = fail)** | Re-fire Answer Agent with critic's `suggested_revisions` once. If second verdict is also `fail`, escalate to Legal — the Answer Agent never overrides the Critic. **The Critic always wins.** |
| **Critic flags coverage gap** *(Planner checklist item not addressed)* | Same path as a grounding failure: re-fire Answer Agent with the gap named, max once. If still missing, escalate. The Answer Agent must *explicitly* report on every checklist item, including "not present in this document" — silent omission is itself a failure. |
| **Two memory writers conflict on the same key in run scratch** | Should not happen by design — each step writes to a distinct namespaced key. If it does, log the violation, abort the run, escalate to engineering. (This is a bug, not a steady-state behaviour.) |
| **Schema validation fails at a handoff** | The receiving agent does not start. Orchestrator routes to the system-level fallback (escalate to Legal). |

The single principle behind every conflict rule: **the system fails toward Legal escalation, never toward shipping an unverified answer.** This is the asymmetric reliability target made operational at the protocol layer.

---

## Observability / tracing

- **Per-agent log:** input, output, model, tokens, cost, latency, errors. LangGraph + LangSmith for prompt/trace history.
- **Per-handoff log:** from-agent, to-agent, schema, payload size, validation result. A failed validation is a P1 alert.
- **Per memory write:** store, key, agent, timestamp, payload size. Critic-failure-pattern and refusal-pattern writes additionally log the PII-redaction step's output (zero personally-identifying tokens detected = pass; otherwise the write is rejected and the run aborts).
- **Where logs live:** LangSmith for prompt/trace history + OTel for system metrics. LangFlow's built-in tracing is used only for the demo deliverable (Slide 1).
- **Retention:** 30 days for full traces; aggregate metrics indefinitely. Any trace that persists past 24 hours is PII-redacted before storage — the redaction itself is logged.
- **On-call alerting:** failure rate >5% over 1h; p95 latency >2× target (i.e. >120s); Critic-fail rate >25% in any 24h window (suggests an upstream quality regression rather than individual flaky cases); any PII redaction violation (zero tolerance — single occurrence pages on-call).

---

## System cost & latency budget

- **Total cost target per successful run:** ≤$0.10 (3 LLM calls + parsing — cheaper than v0.1 because no embedding step).
- **Total cost ceiling per run:** $0.50 (hard cap; aborts when exceeded — also reduced from v0.1's $1.00).
- **End-to-end p50 latency target:** ≤20s (parse + 3 LLM calls + critic + occasional reflexion).
- **End-to-end p95 latency target:** ≤60s.
- **Latency degradation policy under load:** if any agent breaches its individual budget, fall back per the agent's fallback rule. **If the Critic times out specifically, escalate to Legal — never ship without verification.** Saying *"this needs Legal"* under load is always a safer outcome than saying *"yes/no"* without a Critic pass.

---

## Design revision — why we dropped the Retriever (v0.1 → v0.2)

The v0.1 spec included an embedding-based Retriever between the Planner and the Answer Agent — a classic RAG pattern for grounding LLM outputs in a document. After Phase 1 plumbing validation, we dropped it. Four reasons, in order of weight:

1. **Document size fits comfortably in context.** A typical NDA is 2–10 pages (~5–15k tokens). Sonnet 4.7's 200k-token window holds the whole document with room for the question, the system prompt, the Planner's checklist, and Sonnet's own reasoning. Phase 1 demonstrated this empirically — full-PDF-in-context produced sharp, well-cited answers with no quality issue.
2. **Privacy posture conflicts with persistent embeddings.** The MAC Spec's stateless-by-design commitment means an embedding index built per session would be infrastructure with a session-length lifetime — built, used once, thrown away within seconds. The cost-benefit was always weak; saying so explicitly hardens the privacy story.
3. **Citation grounding is achievable without retrieval.** Per-page metadata from PyPDF gives the Answer Agent enough anchor information to cite by page; the Critic verifies the cited text exists verbatim in the same full text the Answer Agent saw. No vector search required.
4. **Simpler architecture is better defended.** Four roles we deeply understand beats five roles where one is decorative. The deck story is sharper too: *"We removed a component the architecture initially called for once we saw it was infrastructure for an infrastructure problem we didn't have."*

The Planner is preserved — but with a different job. Originally it decomposed the question for retrieval; now it generates the **standard-NDA-clause checklist** that the Answer Agent must verify across the document, independent of the question. This is the **missed-clause defence** — exhaustive coverage rather than question-driven analysis. A user asking "is there a non-compete?" still gets coverage of related restraint clauses, IP assignment, residuals, jurisdiction, etc. The Critic verifies the checklist was addressed, catching silent omissions.

**The MAC-Spec-specific implications of this revision:**

- **One fewer agent + one fewer tool.** Five components → four agents.
- **Two fewer handoffs.** Seven schemas → five schemas.
- **No "empty retrieval" termination condition** — replaced by *coverage gap*, where the Critic catches checklist items the Answer Agent silently omitted.
- **Memory architecture simplified.** No embedding index in session memory; no chunked PDF in run scratch — just the full text + page metadata.
- **Cost target halved** (~$0.20 → $0.10) and latency targets reduced (p50 30s → 20s, p95 90s → 60s).
- **Communication channel switched from LangFlow primary to LangChain / LangGraph primary** — the architecture is sequential enough that LangGraph's state machine is the natural runtime. LangFlow remains the demo-deliverable layer for Slide 1.

**One scenario that would re-introduce retrieval:** an extraordinarily long NDA (>~150k tokens, ~300+ pages). In that case the architecture re-introduces retrieval as a Phase 2.5 component. Until that case appears in the eval set, the simpler design holds.

---

## Open questions

1. **Standard Pactly NDA template availability.** If Legal has a canonical "standard NDA," it directly seeds the Planner's checklist content (clause types and order to verify). Materially strengthens the missed-clause defence.
2. **Checklist content discipline.** Who owns the standard-NDA-clause checklist? Legal should curate the v1 list (this is the General Counsel's territory); engineering wires it into the Planner prompt. Without curated checklist content, the missed-clause defence is theoretical.
3. **Session memory scope.** Strict per-browser-session, or 24h device-scoped (let the user reload tomorrow without re-uploading)? The 24h option is better UX but raises a privacy bar (encrypted local storage; explicit user opt-in).
4. **SaulLM-7B hosting.** Self-hosted on Pactly infrastructure or via the HuggingFace Inference API? Affects cost, latency, and the security review.
5. **PII-redaction implementation.** Heuristic regex-based (cheap, fast, may miss subtle entities) or LLM-driven entity recognition (slower, more accurate, more expensive)? At v1, leaning regex for the persistent stores given they're aggregate-only and the input is already filtered to clause-type metadata.

---

## Review log

| Date | Reviewer | Version | What changed |
|---|---|---|---|
| 2026-04-29 | Benedict (group lead) + Claude (Opus 4.7) | v0.1 | Initial fill. Five sharp design decisions baked in: stateless-by-design with explicit no-PII-storage; session memory only for parsed PDF + index; reflexion loop hard-capped at 1 iteration; Critic always wins over Answer Agent; empty retrieval routes directly to escalation rather than risking fabrication. Retriever is a tool, not an agent. |
| 2026-05-01 | Benedict (group lead) + Claude (Opus 4.7) | v0.2 | **Architecture simplified after Phase 1 validation.** Retriever removed entirely — full NDA fits in Sonnet 4.7's 200k context, retrieval was solving a non-problem. Five components → four agents. Seven handoff schemas → five. Planner reframed: now generates the standard-NDA-clause checklist (missed-clause defence by exhaustive coverage). Cost target halved ($0.20 → $0.10); latency targets cut (p50 30s → 20s, p95 90s → 60s). Communication channel switched to LangChain / LangGraph as the primary runtime; LangFlow becomes demo-deliverable layer only. Added *Design revision* section with full rationale. |
