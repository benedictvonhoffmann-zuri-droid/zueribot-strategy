# Multi-Agent Collaboration Spec — NDA Helper Agent (Pactly)

> **Example fill** of [`multi-agent-collaboration-spec-template.md`](multi-agent-collaboration-spec-template.md). Built for the Productschool Advanced AI Agents course final project — *not* a Bünzli artefact. Companion to [`agent-workflow-spec-example-pactly-nda.md`](agent-workflow-spec-example-pactly-nda.md): the AW Spec defines what each agent does individually; the MAC Spec defines how they coordinate.

---

## Identity

- **System name:** NDA Helper Agent — Pactly Internal Tools
- **Owner:** Senior PM, Pactly Internal Tools division
- **Spec version:** v0.1 (course final-project draft)
- **Last updated:** 2026-04-29
- **Status:** Draft — buildable in LangFlow
- **Related AW Spec:** [`agent-workflow-spec-example-pactly-nda.md`](agent-workflow-spec-example-pactly-nda.md)

---

## The spec table

| Field | Spec |
|---|---|
| **Agents involved** | **Four LLM-driven agents + one deterministic tool**, in this order: (1) **Scope Classifier** (Haiku 4.5) — gates whether the question is "first-pass" answerable. (2) **Planner** (Haiku 4.5) — decomposes the question into clause topics. (3) **Retriever** *(tool, not agent)* — embedding + keyword search over the parsed PDF; called by the orchestrator between Planner and Answer Agent. (4) **Answer Agent** (Sonnet 4.7) — composes plain-English response with verbatim citations. (5) **Critic** (SaulLM-7B) — verifies grounding. Different model family from the Answer Agent for genuinely independent verification. |
| **Orchestration type** | **Sequential pipeline with a terminal critic-reflexion loop.** Linear flow: Scope → Plan → Retrieve *(tool)* → Answer ↔ Critic. The Answer↔Critic step has at most **one** reflexion iteration; if the Critic fails twice, the system escalates rather than continuing to revise. Not blackboard, not hierarchical — explicit deterministic ordering with a single feedback loop at the verification stage. |
| **Data format** | Six typed handoff contracts (full schemas in *Handoff schemas* below). All contracts are schema-validated at the boundary — if validation fails, the receiving agent does not start and the orchestrator escalates. |
| **Communication channel** | **LangFlow node-to-node connections at v1** (the prototype layer the course requires). Per-run shared state lives in LangFlow's run-state object, which functions as a per-run blackboard. Production target (v2): LangGraph state object passed through a deterministic workflow orchestrator with the same schema contracts. |
| **Trigger logic** | Mostly **sequential complete** + one **conditional output** (Scope = in_scope → Planner; Scope = out_of_scope → terminate). The reflexion loop is **conditional + max-iteration-capped** (Critic = fail AND retries < 1 → re-fire Answer). Empty retrieval is also conditional — Retriever returns 0 candidates for a planned topic → orchestrator skips Answer Agent and triggers escalation directly, so the Answer Agent never has the option to fabricate. |
| **Fallback logic** | Per agent — Scope/Planner failure: retry once → abort with generic "couldn't process, contact Legal." Retriever (tool) failure: OCR fallback for parsing; keyword-only fallback for embedding lookup. Answer Agent failure: retry once → fall back to "verbatim relevant clauses with no plain-English synthesis" + escalate. **Critic failure: HITL escalate. Never ship without verification.** System-level: any unrecoverable failure = "Send to Legal" with the user's question + the PDF preserved in the handoff to a human reviewer. |
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
   │  1. Parse + chunk PDF│  PyMuPDF (+ OCR fallback)
   └──────────┬───────────┘
              │
              ▼
   ┌──────────────────────┐
   │  2. Planner          │  Haiku 4.5
   └──────────┬───────────┘
              │ topics
              ▼
   ┌──────────────────────┐
   │  3. Retriever (tool) │  embedding + keyword
   └──────────┬───────────┘
              │ candidate_clauses
              │  (if 0 candidates → escalate)
              ▼
   ┌──────────────────────┐
   │  4. Answer Agent     │  Sonnet 4.7
   └──────────┬───────────┘
              │ draft + citations
              ▼
   ┌──────────────────────┐
   │  5. Critic           │  SaulLM-7B
   └──────────┬───────────┘
              │
       ┌──────┴──────┐
   pass│             │ fail (≤1 retry)
       ▼             ▼
  [Deliver]    [Re-fire 4 with feedback]
                     │ second fail
                     ▼
              [Escalate to Legal]
```

---

## Handoff schemas

| From → To | Schema |
|---|---|
| Scope Classifier → orchestrator | `{ in_scope: bool, refusal_reason?: string, suggested_legal_route?: string }` |
| Orchestrator → Planner | `{ question: string, doc_id: string, doc_summary: string }` |
| Planner → Retriever (tool) | `{ topics: [{ name: string, keywords: string[], priority: "H"|"M"|"L" }] }` |
| Retriever → orchestrator | `{ topic_results: [{ topic: string, candidate_clauses: [{ text: string, page: int, paragraph_anchor: string, score: float }] }], doc_id: string }` |
| Orchestrator → Answer Agent | `{ question: string, topic_results: [...], doc_id: string }` *(empty `topic_results` triggers escalation, not the Answer Agent)* |
| Answer Agent → Critic | `{ draft_answer: string, citations: [{ quote: string, page: int, paragraph_anchor: string }], confidence: float, doc_id: string, question: string }` |
| Critic → orchestrator | `{ verdict: "pass" \| "fail", reasons: string[], suggested_revisions?: { specific_clause_issues: [...], grounding_violations: [...] }, retry_count: int }` |

Every contract is schema-validated at the boundary. A handoff that fails validation does not invoke the receiving agent — the orchestrator routes to the system-level fallback (escalate to Legal).

---

## Memory architecture

> **The posture: stateless-by-design for user data, aggregate-only for system learning.** No per-user PDFs, no per-user question history, no per-user answer logging — and **no Personally Identifiable Information** persisted past the run, full stop. PDFs may contain PII (party names, signatures, addresses, contact information); we do not extract, log, or persist any of it beyond the session run. All in-memory state is wiped on session end. Aggregate patterns (critic failures, refusal patterns) are PII-stripped at write time. This is the privacy story we tell General Counsel directly: *"We don't store your contract, we don't log your questions, and we don't keep any personal information beyond the time it takes to answer."*

### Memory stores

| Store | Type | Lifetime | Who writes | Who reads | Pruning rule | Privacy posture |
|---|---|---|---|---|---|---|
| **Working memory (per LLM call)** | Working | One LLM call | Each agent's prompt | The same agent | N/A — recreated per call | Standard; ephemeral |
| **Run scratch (LangFlow state)** | Shared blackboard | One run | All four agents + Retriever tool | All four agents + Retriever tool | Deleted at run end | Wiped on completion or abort. PII may be present briefly in PDF text but never persisted. |
| **Session: parsed PDF + chunk index** | Session | One browser session | Set at upload (parser + embedding step) | Retriever, Answer Agent (for citation rendering) | Deleted on session end / tab close / new upload | Per-session only; never written to disk; never crossed-user |
| **Eval set (50 NDAs)** | System persistent | Indefinite, monthly refresh | Legal team (curators) | Eval CI; PM; engineering | Refresh / replace cycle; failures from production promoted to eval | **PII-redacted at curation time** by Legal team. Synthetic NDAs preferred; real ones only with redaction. |
| **Critic-failure patterns (aggregate)** | System persistent | Indefinite | Critic (writes after each "fail" verdict) | Planner (informs clause checklist), PM | FIFO with importance-weighted decay over 90 days | **Aggregate only.** Per-user attribution stripped at write time. Only the *clause type* and *failure mode* are stored, never the source PDF or the question. |
| **Refusal patterns (aggregate)** | System persistent | Indefinite | Scope Classifier | Scope Classifier prompt-improvement, PM | Same FIFO + decay rule | **Aggregate only.** Question categories tracked, never the literal question text. |

### What we deliberately do not remember

Equally important — the "stateless-by-design" list, named explicitly so it survives every refactor:

- **User-uploaded PDFs after the session ends.** Wiped from session storage on tab close. Never written to disk.
- **The literal text of user questions.** Aggregate categories of refused questions are kept (for the refusal-pattern store), but never the user's actual phrasing.
- **The literal text of agent answers.** Aggregate metrics (correct-answer rate, false-confidence rate) are kept; the answers themselves are not logged.
- **Any Personally Identifiable Information** (party names, signatures, addresses, email addresses, phone numbers, dates of birth, identification numbers) — never extracted, never logged, never persisted past the run. PDFs are processed as opaque text; PII present in that text is discarded with the rest of the run state.
- **Per-user history.** No "your last 5 NDAs" view, ever. Privacy posture, not a missing feature.

This list is the privacy story. It is also a constraint — no per-user personalisation, no preference learning across sessions, no "we remember you prefer this format." That trade is deliberate and correct for an internal legal-adjacent tool with a privacy-conscious General Counsel.

---

## Termination conditions

| Condition | Trigger | Result |
|---|---|---|
| **Success** | Critic returns `pass` | Render the answer with HITL-by-UX visual citations |
| **Out of scope** | Scope Classifier returns `in_scope: false` | Polite refusal with `suggested_legal_route` |
| **Empty retrieval** | Retriever returns 0 candidates for any high-priority planned topic | Skip Answer Agent → escalate to Legal with "couldn't find clauses about X in this document" |
| **Critic max-loop** | Critic returns `fail` after 1 reflexion retry | Escalate to Legal with the draft + critic reasons |
| **Hard failure** | Unrecoverable error in any agent or tool | Abort + log + escalate to Legal |
| **Time budget exhausted** | Run exceeds 90s wall-clock | Abort with "this question is taking unusually long; routing to Legal" |
| **Cost budget exhausted** | Run exceeds $1.00 hard cap | Abort + alert engineering |

---

## Conflict resolution

| Scenario | Resolution rule |
|---|---|
| **Critic disagrees with Answer Agent (verdict = fail)** | Re-fire Answer Agent with critic's `suggested_revisions` once. If second verdict is also `fail`, escalate to Legal — the Answer Agent never overrides the Critic. **The Critic always wins.** |
| **Retriever returns 0 candidates for a high-priority topic** | Do not invoke the Answer Agent. Orchestrator escalates directly with the topic name. The Answer Agent is structurally prevented from fabricating an answer that isn't grounded in retrieved text. |
| **Two memory writers conflict on the same key in run scratch** | Should not happen by design — each step writes to a distinct namespaced key. If it does, log the violation, abort the run, escalate to engineering. (This is a bug, not a steady-state behaviour.) |
| **Schema validation fails at a handoff** | The receiving agent does not start. Orchestrator routes to the system-level fallback (escalate to Legal). |

The single principle behind every conflict rule: **the system fails toward Legal escalation, never toward shipping an unverified answer.** This is the asymmetric reliability target made operational at the protocol layer.

---

## Observability / tracing

- **Per-agent log:** input, output, model, tokens, cost, latency, errors. Visible in LangFlow's built-in tracing at v1; LangSmith in v2.
- **Per-handoff log:** from-agent, to-agent, schema, payload size, validation result. A failed validation is a P1 alert.
- **Per memory write:** store, key, agent, timestamp, payload size. Critic-failure-pattern and refusal-pattern writes additionally log the PII-redaction step's output (zero personally-identifying tokens detected = pass; otherwise the write is rejected and the run aborts).
- **Where logs live:** LangFlow built-in (v1); LangSmith for prompt/trace history + OTel for system metrics (v2).
- **Retention:** 30 days for full traces; aggregate metrics indefinitely. Any trace that persists past 24 hours is PII-redacted before storage — the redaction itself is logged.
- **On-call alerting:** failure rate >5% over 1h; p95 latency >2× target (i.e. >180s); Critic-fail rate >25% in any 24h window (suggests an upstream quality regression rather than individual flaky cases); any PII redaction violation (zero tolerance — single occurrence pages on-call).

---

## System cost & latency budget

- **Total cost target per successful run:** ≤$0.20 (per AW Spec; coordination overhead is negligible since LangFlow node-passing is in-memory).
- **Total cost ceiling per run:** $1.00 (hard cap; aborts when exceeded).
- **End-to-end p50 latency target:** ≤30s (parse + 4 LLM calls + retrieval + critic, plus reflexion in ~10% of cases).
- **End-to-end p95 latency target:** ≤90s.
- **Latency degradation policy under load:** if any agent breaches its individual budget, fall back per the agent's fallback rule. **If the Critic times out specifically, escalate to Legal — never ship without verification.** Saying *"this needs Legal"* under load is always a safer outcome than saying *"yes/no"* without a Critic pass.

---

## Open questions

1. **Is LangFlow the production runtime, or just the prototyping environment?** Affects observability tooling choice (LangFlow built-in vs. LangGraph + LangSmith + OTel).
2. **Standard Pactly NDA template KB availability.** If Legal has a canonical "standard NDA," wiring it as Critic context lets the Critic flag deviations from standard, not just verify grounding. Materially strengthens defence against the "missed clause" failure mode.
3. **Session memory scope.** Strict per-browser-session, or 24h device-scoped (let the user reload tomorrow without re-uploading)? The 24h option is better UX but raises a privacy bar (encrypted local storage; explicit user opt-in).
4. **SaulLM-7B hosting.** Self-hosted on Pactly infrastructure or via an inference provider? Affects cost, latency, and the security review.
5. **PII-redaction implementation.** Heuristic regex-based (cheap, fast, miss subtle entities) or LLM-driven entity recognition (slower, more accurate, more expensive)? At v1, leaning regex for the persistent stores given they're aggregate-only and the input is already filtered to clause-type metadata.

---

## Review log

| Date | Reviewer | Version | What changed |
|---|---|---|---|
| 2026-04-29 | Benedict (group lead) + Claude (Opus 4.7) | v0.1 | Initial fill. Five sharp design decisions baked in: stateless-by-design with explicit no-PII-storage; session memory only for parsed PDF + index; reflexion loop hard-capped at 1 iteration; Critic always wins over Answer Agent; empty retrieval routes directly to escalation rather than risking fabrication. Retriever is a tool, not an agent. |
