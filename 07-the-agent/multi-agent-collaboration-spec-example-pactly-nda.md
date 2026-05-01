# Multi-Agent Collaboration Spec — NDA Helper Agent (Pactly)

> **Example fill** of [`multi-agent-collaboration-spec-template.md`](multi-agent-collaboration-spec-template.md). Built for the Productschool Advanced AI Agents course final project — *not* a Bünzli artefact. Companion to [`agent-workflow-spec-example-pactly-nda.md`](agent-workflow-spec-example-pactly-nda.md): the AW Spec defines what each agent does individually; the MAC Spec defines how they coordinate.
>
> **v0.3 design revision (2026-05-01).** Role-character mapping introduced. **SaulLM-7B = Senior Counsel** (Planner + Critic). **Haiku 4.5 = Analyst** (Scope). **Sonnet 4.7 = Legal Copywriter** (Answer). Each model does what its training is actually best at. See *Design revision* below.

---

## Identity

- **System name:** NDA Helper Agent — Pactly Internal Tools
- **Owner:** Senior PM, Pactly Internal Tools division
- **Spec version:** v0.3 (role-character mapping — see *Design revision*)
- **Last updated:** 2026-05-01
- **Status:** Phase 1 + Phase 2 (Haiku-Planner) validated in LangChain. Saul-as-Planner swap planned for Phase 4 alongside Critic wiring.
- **Related AW Spec:** [`agent-workflow-spec-example-pactly-nda.md`](agent-workflow-spec-example-pactly-nda.md)

---

## The role-character mapping

| Character | Model | Job | Appears in |
|---|---|---|---|
| **Senior Counsel** | **SaulLM-7B** | Plans the question-relevant clause checklist *and* critiques the Answer Agent's draft for grounding + coverage. Legal-judgement work. | Step 2 (Planner) and Step 4 (Critic) |
| **Analyst** | **Claude Haiku 4.5** | Triage gate — is this question even in scope for the agent? | Step 0 (Scope Classifier) |
| **Legal Copywriter** | **Claude Sonnet 4.7** | Reads the full NDA + question + Senior Counsel's checklist; drafts the plain-English answer with verbatim citations. | Step 3 (Answer Agent) |

This framing is the spec's strongest single claim — each model is matched to the *character* of the work it does. Plan and critique are both legal judgement; drafting is writing; triage is fast classification. The deck story writes itself.

---

## The spec table

| Field | Spec |
|---|---|
| **Agents involved** | **Four LLM-driven agents, three model characters**: (1) **Scope Classifier** *(Analyst — Haiku 4.5)* — gates whether the question is "first-pass" answerable. (2) **Planner** *(Senior Counsel — SaulLM-7B)* — generates the question-relevant clause checklist (direct + adjacent items). (3) **Answer Agent** *(Legal Copywriter — Sonnet 4.7)* — reads full NDA + question + checklist; composes plain-English response with verbatim citations. (4) **Critic** *(Senior Counsel — SaulLM-7B)* — verifies grounding *and* checklist coverage. |
| **Orchestration type** | **Sequential pipeline with a terminal critic-reflexion loop.** Linear flow: Scope → Plan → Answer ↔ Critic. The Answer↔Critic step has at most **one** reflexion iteration; if the Critic fails twice, the system escalates rather than continuing to revise. Not blackboard, not hierarchical — explicit deterministic ordering with a single feedback loop at the verification stage. |
| **Data format** | Five typed handoff contracts (full schemas in *Handoff schemas* below). All contracts are schema-validated at the boundary — if validation fails, the receiving agent does not start and the orchestrator escalates. |
| **Communication channel** | **LangChain LCEL chains + LangGraph state** for the build. The orchestrator is a LangGraph `StateGraph` with named nodes per agent and explicit conditional edges. Per-run shared state lives in the LangGraph state object, which functions as a per-run blackboard. Langflow is the demo-deliverable target — same graph re-laid in visual nodes for Slide 1. |
| **Trigger logic** | Mostly **sequential complete** + one **conditional output** (Scope = in_scope → Planner; Scope = out_of_scope → terminate). The reflexion loop is **conditional + max-iteration-capped** (Critic = fail AND retries < 1 → re-fire Answer). |
| **Fallback logic** | Per agent — Scope/Planner failure: retry once → abort with "couldn't process, contact Legal." PDF parsing failure: abort with "this PDF is not readable, please re-upload" (OCR fallback deferred). Answer Agent failure: retry once → fall back to "verbatim relevant clauses with no plain-English synthesis" + escalate. **Critic failure: HITL escalate. Never ship without verification.** *Senior Counsel JSON-output fallback: if Saul's Planner-role JSON reliability is unworkable, invest in structured-output schemas (constrained decoding / validation + retry) rather than reverting to Haiku-Planner — preserving the Senior Counsel framing matters.* System-level: any unrecoverable failure = "Send to Legal" with the user's question + the PDF preserved in the handoff to a human reviewer. |
| **Governance hooks** | Inherited from the AW Spec (Scope check, Critic verification, HITL-by-UX visual citation, weekly red-team). MAC-Spec-level additions: **schema validation at every handoff** (catches contract drift before it corrupts downstream silently); **PII-redaction check at every persistent write** (described in *Memory architecture*). |

---

## Orchestration detail

Visual flow:

```
[User upload + question]
          │
          ▼
   ┌─────────────────────────┐
   │  0. Scope Classifier    │  Analyst — Haiku 4.5
   └──────────┬──────────────┘
              │ in_scope = true
              ▼
   ┌─────────────────────────┐
   │  1. Parse PDF           │  PyPDF (page-level)
   └──────────┬──────────────┘
              │ full_text + page_metadata
              ▼
   ┌─────────────────────────┐
   │  2. Planner             │  Senior Counsel — SaulLM-7B
   └──────────┬──────────────┘
              │ clause_checklist (direct + adjacent)
              ▼
   ┌─────────────────────────┐
   │  3. Answer Agent        │  Legal Copywriter — Sonnet 4.7
   └──────────┬──────────────┘   (full PDF + question + checklist)
              │ draft + citations + coverage_report
              ▼
   ┌─────────────────────────┐
   │  4. Critic              │  Senior Counsel — SaulLM-7B
   └──────────┬──────────────┘
              │
       ┌──────┴──────┐
   pass│             │ fail (≤1 retry)
       ▼             ▼
  [Deliver]    [Re-fire 3 with feedback]
                     │ second fail
                     ▼
              [Escalate to Legal]
```

Note: Senior Counsel (Saul) appears in two scenes — Step 2 and Step 4. Same character, two appearances.

---

## Handoff schemas

| From → To | Schema |
|---|---|
| Scope Classifier → orchestrator | `{ in_scope: bool, refusal_reason?: string, suggested_legal_route?: string }` |
| PDF parser → run scratch | `{ full_text: string, pages: [{ page_number: int, text: string }], doc_id: string }` |
| Orchestrator → Planner | `{ question: string, doc_summary: string }` |
| Planner → orchestrator | `{ clause_checklist: [{ clause_type: string, what_to_check: string, priority: "H"|"M"|"L", reason: string }], ... }` |
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
| **Session: parsed PDF text + page metadata** | Session | One browser session | Set at upload (PyPDF parser) | Answer Agent (for context), Critic (for grounding verification) | Deleted on session end / tab close / new upload | Per-session only; never written to disk; never crossed-user. *No embedding index.* |
| **Eval set (50 NDAs)** | System persistent | Indefinite, monthly refresh | Legal team (curators) | Eval CI; PM; engineering | Refresh / replace cycle; failures from production promoted to eval | **PII-redacted at curation time** by Legal team. Synthetic NDAs preferred; real ones only with redaction. |
| **Critic-failure patterns (aggregate)** | System persistent | Indefinite | Critic (writes after each "fail" verdict) | Planner prompt-improvement, PM | FIFO with importance-weighted decay over 90 days | **Aggregate only.** Per-user attribution stripped at write time. Only the *clause type* and *failure mode* are stored, never the source PDF or the question. |
| **Refusal patterns (aggregate)** | System persistent | Indefinite | Scope Classifier | Scope Classifier prompt-improvement, PM | Same FIFO + decay rule | **Aggregate only.** Question categories tracked, never the literal question text. |
| **Standard clause anchor list** | System persistent | Indefinite, curated by Legal | Legal team (curators) | Planner (as background context for checklist generation) | Versioned; reviewed quarterly | Not user data — system reference content. |

### What we deliberately do not remember

Equally important — the "stateless-by-design" list, named explicitly so it survives every refactor:

- **User-uploaded PDFs after the session ends.** Wiped from session storage on tab close. Never written to disk.
- **The literal text of user questions.** Aggregate categories of refused questions are kept (for the refusal-pattern store), but never the user's actual phrasing.
- **The literal text of agent answers.** Aggregate metrics (correct-answer rate, false-confidence rate) are kept; the answers themselves are not logged.
- **Any Personally Identifiable Information** (party names, signatures, addresses, email addresses, phone numbers, dates of birth, identification numbers) — never extracted, never logged, never persisted past the run. PDFs are processed as opaque text; PII present in that text is discarded with the rest of the run state.
- **Per-user history.** No "your last 5 NDAs" view, ever. Privacy posture, not a missing feature.
- **Embeddings of any kind.** The architecture deliberately does not build an embedding index — the document fits in context. Beyond being unnecessary, embeddings would be infrastructure that lives only seconds and adds no compounding value.

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
| **Planner JSON-parse failure** | Saul's checklist output cannot be parsed after retry | Either invest in output-schema mitigation (constrained decoding) or — if that fails too — escalate. *(Decision per Open Question 7: don't fall back to Haiku-Planner; that breaks the Senior Counsel framing.)* |
| **Hard failure** | Unrecoverable error in any agent | Abort + log + escalate to Legal |
| **Time budget exhausted** | Run exceeds 75s wall-clock *(loosened from v0.2 — Saul cold-start exposure)* | Abort with "this question is taking unusually long; routing to Legal" |
| **Cost budget exhausted** | Run exceeds $0.50 hard cap | Abort + alert engineering |

---

## Conflict resolution

| Scenario | Resolution rule |
|---|---|
| **Critic disagrees with Answer Agent (verdict = fail)** | Re-fire Answer Agent with critic's `suggested_revisions` once. If second verdict is also `fail`, escalate to Legal — the Answer Agent never overrides the Critic. **The Critic always wins.** |
| **Critic flags coverage gap** *(Planner checklist item not addressed)* | Same path as a grounding failure: re-fire Answer Agent with the gap named, max once. If still missing, escalate. The Answer Agent must *explicitly* report on every checklist item, including "not present in this document" — silent omission is itself a failure. |
| **Senior Counsel (Saul) self-correlation risk** | Same model plays Planner and Critic. If Saul-the-Planner has a blind spot for clause type X, Saul-the-Critic will also miss X's absence. Acceptable trade — the alternative (Sonnet-as-Critic) breaks the more important Answer-vs-Critic family independence. Mitigation: periodic eval-set sweeps that benchmark Saul's coverage against a Sonnet-as-Critic shadow run, flagging any cases where the two disagree. |
| **Two memory writers conflict on the same key in run scratch** | Should not happen by design — each step writes to a distinct namespaced key. If it does, log the violation, abort the run, escalate to engineering. |
| **Schema validation fails at a handoff** | The receiving agent does not start. Orchestrator routes to the system-level fallback (escalate to Legal). |

The single principle behind every conflict rule: **the system fails toward Legal escalation, never toward shipping an unverified answer.** This is the asymmetric reliability target made operational at the protocol layer.

---

## Observability / tracing

- **Per-agent log:** input, output, model, tokens, cost, latency, errors. LangGraph + LangSmith for prompt/trace history.
- **Per-handoff log:** from-agent, to-agent, schema, payload size, validation result. A failed validation is a P1 alert.
- **Per memory write:** store, key, agent, timestamp, payload size. Critic-failure-pattern and refusal-pattern writes additionally log the PII-redaction step's output (zero personally-identifying tokens detected = pass; otherwise the write is rejected and the run aborts).
- **Where logs live:** LangSmith for prompt/trace history + OTel for system metrics. LangFlow's built-in tracing is used only for the demo deliverable (Slide 1).
- **Senior Counsel correlation telemetry:** specific to v0.3 — log per run whether Saul-the-Planner and Saul-the-Critic agreed on coverage. Disagreement (Critic flags a gap the Planner didn't even include) is informative; agreement on a *miss* (both ignore clause X that humans flag) is the failure mode we're watching for.
- **Retention:** 30 days for full traces; aggregate metrics indefinitely. Any trace that persists past 24 hours is PII-redacted before storage — the redaction itself is logged.
- **On-call alerting:** failure rate >5% over 1h; p95 latency >2× target (i.e. >150s); Critic-fail rate >25% in any 24h window (suggests an upstream quality regression rather than individual flaky cases); any PII redaction violation (zero tolerance — single occurrence pages on-call).

---

## System cost & latency budget

- **Total cost target per successful run:** ≤$0.12 (Haiku scope + Saul plan + Sonnet answer + Saul critique). Saul fires twice but per-call cost is low.
- **Total cost ceiling per run:** $0.50 (hard cap; aborts when exceeded).
- **End-to-end p50 latency target:** ≤30s (assumes Saul warm; cold-start adds 30–60s on the first call after idle).
- **End-to-end p95 latency target:** ≤75s.
- **Latency degradation policy under load:** if any agent breaches its individual budget, fall back per the agent's fallback rule. **If the Critic times out specifically, escalate to Legal — never ship without verification.** If the Planner times out (Saul cold-start blocking the run), fall back to the Haiku-Planner path validated in Phase 2 *(operational fallback only; not the architectural plan)*.

---

## Design revision — role-character mapping (v0.2 → v0.3)

The v0.2 spec used Haiku 4.5 for both the Scope Classifier and the Planner — a "small fast model for everything cheap" pattern. Phase 2 validated that the Haiku-driven Planner produced sensible question-relevant checklists, so the build was working. But re-examining the role assignments revealed an architectural mismatch worth correcting before Phase 4.

**The insight.** Plan and critique are both *legal-judgement* tasks. Drafting plain-English from quoted clauses is a *writing* task. Triage is a *fast-classification* task. v0.2 was using Haiku for two of those — not because Haiku was the right tool for both, but because it was the cheap default.

**The revision.** Map each model to the *character* of the work it does best:

- **SaulLM-7B = Senior Counsel.** Plans the checklist (legal judgement: what clauses matter for this question, including adjacency awareness) AND critiques the draft (legal judgement: was every checklist item addressed; is every quote anchored verbatim).
- **Haiku 4.5 = Analyst.** Triage on whether the question is even worth Senior Counsel's time (scope check).
- **Sonnet 4.7 = Legal Copywriter.** Reads the full NDA + checklist + question, drafts the plain-English answer with quoted clauses.

**MAC-Spec-specific implications:**

- **Same agent count (4) and same handoff count (5).** The architecture didn't grow; the *casting* changed.
- **Saul appears twice in the orchestration diagram** (Step 2 + Step 4). Same character, two scenes.
- **Cost target nudges up** ($0.10 → $0.12) because Saul fires twice; latency targets widen (p95 60s → 75s) due to cold-start exposure on the first call.
- **A new conflict-resolution row** for Saul self-correlation risk — Saul-the-Planner and Saul-the-Critic share blind spots. Mitigation: periodic shadow-run benchmarks against a Sonnet-as-Critic comparison.
- **A new observability metric** — Senior Counsel correlation telemetry (Planner-Critic disagreement on coverage).
- **A new fallback-path decision** — if Saul's JSON output reliability is unworkable, invest in structured-output schemas rather than reverting to Haiku-Planner. The Senior Counsel framing is an architectural commitment.

**What this revision does *not* change:** Answer-vs-Critic family independence is preserved (Sonnet ≠ Saul). That's the verification claim; it's untouched. Saul appearing in two roles is *internal* correlation, not the same as self-critique.

The deck story is sharper: *"Each model does what its training and capability are actually good at — Senior Counsel for legal judgement, Analyst for triage, Copywriter for plain-English drafting. The architecture isn't just a pipeline; it's a courtroom."*

---

## Open questions

1. **Standard Pactly NDA template availability.** If Legal has a canonical "standard NDA," it directly seeds the Planner's checklist content. Materially strengthens the missed-clause defence.
2. **Checklist content discipline.** Who owns the standard-NDA-clause anchor list? Legal should curate the v1 list (this is the General Counsel's territory); engineering wires it into the Planner prompt.
3. **Session memory scope.** Strict per-browser-session, or 24h device-scoped (let the user reload tomorrow without re-uploading)? The 24h option is better UX but raises a privacy bar (encrypted local storage; explicit user opt-in).
4. **SaulLM-7B hosting.** Self-hosted on Pactly infrastructure or via the HuggingFace Inference API? Affects cost, latency, and the security review.
5. **PII-redaction implementation.** Heuristic regex-based (cheap, fast, may miss subtle entities) or LLM-driven entity recognition (slower, more accurate, more expensive)? At v1, leaning regex for the persistent stores given they're aggregate-only and the input is already filtered to clause-type metadata.
6. **Saul JSON output reliability.** SaulLM is fine-tuned for legal *text*, not structured JSON. **Decision (2026-05-01):** if reliability falls below 95% on the Planner role, *invest in structured-output schemas* (constrained decoding, output validation + retry) rather than revert to Haiku-Planner. Preserving the Senior Counsel framing is worth the engineering cost.

---

## Review log

| Date | Reviewer | Version | What changed |
|---|---|---|---|
| 2026-04-29 | Benedict (group lead) + Claude (Opus 4.7) | v0.1 | Initial fill. Five sharp design decisions: stateless-by-design with explicit no-PII-storage; session memory only for parsed PDF + index; reflexion loop hard-capped at 1 iteration; Critic always wins over Answer Agent; empty retrieval routes directly to escalation. Retriever as a tool, not an agent. |
| 2026-05-01 | Benedict (group lead) + Claude (Opus 4.7) | v0.2 | Architecture simplified after Phase 1 validation. Retriever removed entirely — full NDA fits in Sonnet's context. Five components → four agents. Planner reframed as question-relevant checklist generator. Communication channel switched to LangChain / LangGraph as the primary runtime. |
| 2026-05-01 | Benedict (group lead) + Claude (Opus 4.7) | v0.3 | **Role-character mapping introduced.** SaulLM-7B promoted to Senior Counsel (Planner + Critic). Haiku 4.5 stays as Analyst (Scope only). Sonnet 4.7 remains Legal Copywriter (Answer). Each model now does what its training is actually best at. New conflict-resolution row for Saul self-correlation risk; new observability metric (Senior Counsel correlation telemetry); new fallback-path decision (invest in JSON output schemas if reliability fails, do not revert to Haiku-Planner). Cost target $0.10 → $0.12; p95 latency 60s → 75s. |
