# Agent Workflow Spec — NDA Helper Agent (Pactly)

> **Example fill** of [`agent-workflow-spec-template.md`](agent-workflow-spec-template.md). Built for the Productschool Advanced AI Agents course final project — *not* a Bünzli artefact. Hosted here as a worked reference for the template.
>
> **The fictional setting (per course brief).** Pactly's Internal Tools division. BizDev users wait *weeks* for Legal to review NDAs, killing deal momentum. The agent is a "first-pass" PDF reviewer that answers specific BizDev questions in plain English, with verifiable citations and explicit escalation when uncertain. The Head of Partnerships wants speed; the General Counsel demands safety. The architecture must reconcile both.
>
> **v0.2 design revision (2026-05-01).** Dropped the embedding-based Retriever after Phase 1 plumbing validation. The full NDA fits comfortably in Sonnet 4.7's context window; retrieval was solving a problem we did not have. The Planner is preserved with a different job — generating a standard-NDA-clause checklist for the Answer Agent. See *Design revision* below for the full rationale.

---

## Identity

- **Agent name:** NDA Helper Agent
- **Owner:** Senior PM, Pactly Internal Tools division
- **Spec version:** v0.2 (architecture simplified — see *Design revision*)
- **Last updated:** 2026-05-01
- **Status:** Phase 1 validated in LangChain (single-LLM PDF reader); Phase 2 (Planner + checklist) in progress.

---

## Connection to broader strategy

- **AI value archetype:** **Copilot.** The agent does not replace Legal — it gives BizDev a defensible first-pass answer so they can decide whether they need Legal at all. Not Automator (no actions taken on user's behalf), not Oracle (more than facts, requires interpretation), not Orchestrator (single document, no cross-system coordination). Copilot frames the right expectation: it augments human judgement rather than substituting for it.
- **Strategic role:** Internal velocity tool — clears the NDA backlog on routine questions while preserving Legal's role on the questions that genuinely require legal judgement.
- **Reliability contract:** **Asymmetric.** False-confident-wrong is catastrophic (BizDev signs based on a wrong answer); false-escalation-to-Legal is merely annoying. The system is engineered to fail in the safer direction — when uncertain, it refuses and routes to Legal.

---

## The spec table

| Field | Spec |
|---|---|
| **Business Goal** | Reduce BizDev's time-to-first-pass-answer on standard NDAs from weeks (waiting on Legal review) to **<2 minutes for ≥80% of common questions**, without ever shipping a confident-wrong answer. Baseline today: median NDA review wait is multiple weeks; BizDev cannot self-serve any question, however basic. |
| **Input** | (1) **PDF contract upload** — required, ≤25 MB, text-extractable preferred; OCR fallback for scanned PDFs deferred to a later phase. (2) **Specific natural-language question** — required, English, scope-checked before processing. Invalid input rejected upfront with a useful error message. |
| **Output** | Three-part structured response: **(1)** plain-English answer to the question; **(2)** verbatim quoted clauses from the NDA serving as citations *(with page anchors from the PyPDF page-level loader)*; **(3)** explicit confidence signal — *"Confident / Send to Legal / Cannot answer from this document."* Each citation is a clickable anchor that opens the source PDF at the cited page with the relevant paragraph highlighted — see *HITL-by-UX* below. |
| **Primary Pattern** | **Pipeline + Critic, with model diversity.** Deterministic flow (Scope → Plan → Answer → Critic), not a free-form hierarchy. The Answer Agent and Critic use **different model families** so the verification is genuinely independent rather than the same model grading itself. *No embedding-based retrieval — the document fits in context. See Design revision.* |
| **Agent Roles** | See *Agent roles* table below. **Four roles**: Scope Classifier, Planner, Answer Agent, Critic. Each has a defined anti-scope. |
| **Key Steps** | See *Key steps* below. Five high-level steps from upload to delivered answer. |
| **Governance Gates** | Four layers: pre-input file + scope validation; pre-output critic grounding check; HITL-by-UX visual citation; post-output weekly red-team. See *Governance gates* below. |
| **Success Metric** | **Primary:** correct-answer rate **≥95%** on a Legal-graded eval set. **Counter-metric:** false-confidence rate **≤1%** — the rate at which the agent confidently gives a wrong answer (without escalating). False *escalations* are tolerable; false *confident-wrong* are not. |

---

## Agent roles

| Role | Persona / job | Tools | Anti-scope |
|---|---|---|---|
| **Scope Classifier** | Step 0. Decides whether the question is "first-pass" answerable or requires Legal. Allowlist: clause existence, term lengths, summary, key obligations. Refuse-list: "should I sign," "negotiate this," "compare to industry standard," "what are the legal risks." | **Claude Haiku 4.5** | Does NOT attempt to answer the question; only routes. |
| **Planner** | Generates the **standard-NDA-clause checklist** the Answer Agent must verify across the document, *independent of the user's specific question*. The checklist drives **exhaustive coverage** — even a narrow question ("is there a non-compete?") triggers checks for related restraint clauses, IP assignment, residuals, jurisdiction, etc. This is the *missed-clause defence* — exhaustive review rather than question-driven analysis. | **Claude Haiku 4.5** | Does NOT answer the user's question; does NOT decide what's important — produces the standard checklist + question-specific augmentations. |
| **Answer Agent** | Reads the **full NDA text** + the user's question + the Planner's checklist. Composes a plain-English response, quoting relevant clauses verbatim with page anchors. Outputs the (1) answer + (2) citations + draft (3) confidence signal. | **Claude Sonnet 4.7** | Does NOT make claims unsupported by the document text; does NOT skip checklist items even if the user didn't ask about them — flags omissions if anything material is silent. |
| **Critic** | Verifies grounding. Checks that every quoted clause exists *verbatim* in the source PDF, that the plain-English answer is logically supported by the quotes, and that **the Planner's checklist was covered** (no silent omissions). **Different model family from the Answer Agent** for independent verification. | **SaulLM-7B** (open-source legal-domain fine-tune) | Does NOT make up evidence to support a flaky answer; does NOT pass an answer it could not verify against the source. |

**Why two different model families.** Asking Claude Sonnet 4.7 to grade Claude Sonnet 4.7 rarely improves quality — same training, same blind spots, same hallucination patterns. Pairing the long-context generalist (Sonnet 4.7) for the Answer Agent with a legal-domain fine-tune (SaulLM-7B) for the Critic produces *genuinely independent* verification. When both models agree the answer is grounded, the confidence is materially higher than self-critique. This is the spec's strongest single design decision.

**Final model lineup.**

| Role | Model | Why |
|---|---|---|
| Scope Classifier + Planner | **Claude Haiku 4.5** | Cheap, fast, good enough for classification and checklist generation. Cost-efficient on the high-volume steps. |
| Answer Agent | **Claude Sonnet 4.7** | Long-context (200k tokens — comfortably holds the whole NDA + checklist + question), strong reasoning, reliable citation behaviour. |
| Critic | **SaulLM-7B** | Different model family + legal-domain fine-tune = genuinely independent verification. Open-source so it's self-hostable, which keeps the verification path off any single hyperscaler. |

---

## Key steps

| # | Step | Tool / role | Output | Failure handling |
|---|---|---|---|---|
| 0 | **Validate + scope-check** | Scope Classifier (Haiku 4.5) | Either "in scope, proceed" or "out of scope, escalate to Legal with reason" | If out of scope, return a polite refusal with one-line reason. |
| 1 | **Parse PDF** | PyPDF page-level loader | Full document text + per-page metadata (for citation anchors) | If PDF is unparseable / image-only, abort with "this PDF is not readable, please re-upload." OCR fallback deferred. |
| 2 | **Plan checklist** | Planner (Haiku 4.5) | Standard-NDA-clause checklist + question-specific augmentations | If checklist generation fails, retry once with same model; on second failure abort with escalation. |
| 3 | **Compose** | Answer Agent (Sonnet 4.7) | Draft answer + verbatim citations with page anchors + initial confidence + checklist coverage report | If the model refuses or returns low-quality output, escalate. |
| 4 | **Verify** | Critic (SaulLM-7B) | Pass/fail + reason (grounding + checklist coverage) | If fail: route back to Compose with critic feedback (one Reflexion loop max); if still fail, escalate to "Send to Legal" rather than ship. |
| 5 | **Deliver** | UI rendering layer | Plain-English answer + clickable verbatim citations + confidence label | UI highlights cited paragraphs in the source PDF — see HITL-by-UX. |

---

## Tools available

| Tool | Type | Used by | Used for | Cost / latency | Failure mode |
|---|---|---|---|---|---|
| PyPDF page-level loader | Library | Step 1 | Extract text + per-page metadata from PDF | Free / <1s for typical NDA | If PDF is image-only, abort (OCR fallback deferred to later phase) |
| Claude Haiku 4.5 | LLM | Steps 0, 2 | Scope check + checklist generation | $0.001–0.005 / 1–3s | If down, fall back to a Mistral-class small model; if both down, abort |
| Claude Sonnet 4.7 | LLM | Step 3 | Compose answer with citations | $0.05–0.15 / 5–15s | If down, fall back to a Gemini 1.5 Pro / GPT-4 Turbo class model *(emergency only — flag clearly in the eval that a fallback model was used)* |
| SaulLM-7B | Self-hosted / HF Inference API | Step 4 | Critic / grounding + coverage verification | self-hosted compute / 3–10s | If down, default to "Send to Legal" rather than ship without verification |

*No embedding model, no vector DB. The full NDA fits in Sonnet's context — see Design revision.*

---

## Out of scope

- **Drafting new clauses.** Legal does that.
- **Negotiation advice.** "How should I respond to this clause?" → escalate.
- **Signing or counter-signing.** Legal / executive responsibility.
- **Cross-document analysis.** One PDF per session; no comparing this NDA to others.
- **General legal Q&A** outside the uploaded document.
- **Risk-judgement questions.** "What are the legal consequences if X happens?" → escalate.
- **PII processing beyond the session.** Documents are not stored after the session ends without explicit user consent.
- **Multi-language NDAs at v1.** English only at launch; non-English NDAs trigger refusal with "We support English NDAs at this time."
- **Long NDAs that exceed Sonnet's context window** *(>~150k tokens, roughly 300+ pages — extraordinarily rare for an NDA).* If we ever encounter one, the architecture would re-introduce retrieval. Until then, the simpler design holds.

---

## Failure modes

| Failure mode | Likelihood | Severity | Mitigation |
|---|---|---|---|
| **Hallucination** — agent claims a clause exists when it doesn't | H | H | Critic verifies every quoted clause is verbatim in source; refuse if not anchored. Different model families for Answer vs Critic = independent verification. |
| **Missed clause** — agent says "no non-compete" when a buried non-compete exists | M | H | **Planner's standard-NDA checklist** forces exhaustive coverage independent of the user's question. Answer Agent must report on every checklist item, including silent omissions ("no clause found about X"). Critic verifies checklist coverage as part of the grounding check. |
| **Misinterpretation** — correct quote, wrong plain-English explanation | M | M | Critic checks logical consistency between quote and explanation. UI surfaces the quote + visual citation prominently so the user can verify. |
| **Adversarial PDF** — malformed file, prompt injection ("ignore previous instructions") embedded in NDA text | L | H | Input sanitisation; system-prompt hardening (instructions in PDF text are treated as data, not instructions); refuse if doc shape is clearly off. |
| **Scope creep** — user asks a non-NDA question or one requiring legal judgement | M | M | Scope Classifier (Step 0) gates this before any document processing. Returns clear refusal with reason. |

---

## Governance gates

| Gate | Layer | What it checks | What happens on fail |
|---|---|---|---|
| File validation | pre-input | PDF format, ≤25 MB, text-extractable | Reject with helpful error |
| Scope check | pre-input | Question is in the "first-pass" allowlist; not in the refuse-list | Return refusal with one-line reason and "this requires Legal" |
| Critic grounding + coverage check | pre-output | Every quoted clause exists verbatim in source; answer is logically supported by quotes; **all checklist items addressed** (no silent omissions) | Reflexion loop once; if still fail, escalate to "Send to Legal" |
| **HITL-by-UX (visual citation)** | pre-output | Every claim in the answer is rendered with a clickable anchor that opens the source PDF at the cited page with the relevant paragraph highlighted | The user verifies by virtue of reading the highlighted clause — no explicit approval click required, but the UX makes the verification effectively unavoidable. The user is in the loop because the UI makes it so. |
| Post-output red-team | post-output | Weekly run of 20 known-edge-case NDAs (buried non-competes, non-standard numbering, foreign governing law, scanned PDFs, prompt-injection cases) against the deployed agent | Alert on any silent regression; gate next deploy on green. |

---

## Cost & latency budgets

- **Cost target per successful run:** ≤$0.10 (3 LLM calls + parsing — cheaper than v0.1 because no embedding step).
- **Cost ceiling per run:** $0.50 (hard cap; aborts and escalates if exceeded).
- **p50 latency target:** ≤20 seconds end-to-end.
- **p95 latency target:** ≤60 seconds.
- **Latency fallback:** if the Critic times out, default to "Send to Legal" rather than ship an unverified answer. Saying *"this needs Legal"* is always a safer outcome than saying *"yes/no"* with no verification.

---

## Human-in-the-loop (HITL)

This agent has no destructive actions (no email sent, no system of record updated), so there are no traditional HITL approval gates. Two HITL patterns are in play:

1. **Self-escalation to Legal** — when the agent is uncertain (Critic fails, scope check fails, checklist coverage incomplete), it voluntarily routes the question to Legal rather than guessing. This is a HITL pattern *driven by the agent itself* — the most reliable form because it doesn't require the user to know they should escalate.
2. **HITL-by-UX (visual citation)** — every claim renders with a clickable, highlighted anchor in the source PDF. The user verifies by reading the highlighted clause. The verification is structural to the UX, not a separate approval step. The user *cannot* trust a claim without seeing what it was based on, because the UI makes the basis prominent.

Optional v2: **sample-rate Legal review.** 5% of agent answers reviewed by Legal post-hoc to calibrate the Critic. Drives eval-set refresh and surfaces silent failure modes.

---

## Eval set

- **Size:** 50 NDAs at v1 (course-deliverable scope), target 200 for production.
- **Composition:**
  - **40% — Standard NDAs / happy path.** Common clauses, common questions. The agent should cruise these.
  - **30% — Edge cases.** Buried non-competes, non-standard clause numbering, foreign governing law, unusually long/short terms, mutual vs unilateral asymmetries.
  - **20% — Adversarial.** Malformed PDFs, prompt-injection attempts, scope-creep questions.
  - **10% — Long NDAs** *(>30 pages — to validate the no-retrieval design holds for longer documents).*
- **Source:** Synthetic NDAs hand-crafted by Legal team + sampled real Pactly NDAs with PII redacted.
- **Refresh cadence:** Monthly. Failures from production reviewed every release; new edge cases promoted to permanent eval.
- **Reviewers:** Legal grades correctness; PM grades UX clarity; automated grounding-check (every cited clause verbatim in source) runs in CI on every prompt or model change.

---

## Design revision — why we dropped retrieval (v0.1 → v0.2)

The v0.1 spec included an embedding-based Retriever between the Planner and the Answer Agent. After Phase 1 plumbing validation, we dropped it. Four reasons, in order of weight:

1. **Document size fits comfortably in context.** A typical NDA is 2–10 pages (~5–15k tokens). Sonnet 4.7's 200k-token window holds the whole document with room for the question, the system prompt, the checklist, and Sonnet's own reasoning. Phase 1 demonstrated this empirically — full-PDF-in-context produced sharp, well-cited answers with no quality issue. Retrieval was solving a problem we did not have.
2. **Privacy posture conflicts with persistent embeddings.** The MAC Spec's stateless-by-design commitment means an embedding index built per session would be infrastructure with a session-length lifetime — built, used once, thrown away within seconds. The cost-benefit was always weak; saying so explicitly hardens the privacy story.
3. **Citation grounding is achievable without retrieval.** Per-page metadata from PyPDF gives Sonnet enough anchor information to cite by page; the Critic verifies the cited text exists verbatim in the same full text the Answer Agent saw. No vector search required.
4. **Simpler architecture is better defended.** Four roles we deeply understand beats five roles where one is decorative. The deck story is sharper too: *"We removed a component the architecture initially called for once we saw it was infrastructure for an infrastructure problem we didn't have."*

The Planner is preserved — but with a different job. Originally it decomposed the question for retrieval. Now it generates the **standard-NDA-clause checklist** that the Answer Agent must verify across the document, independent of the question. This is the **missed-clause defence** — exhaustive coverage rather than question-driven analysis. A user asking "is there a non-compete?" still gets coverage of related restraint clauses, IP assignment, residuals, jurisdiction. The Critic verifies the checklist was addressed, catching silent omissions.

**One scenario that would re-introduce retrieval:** an extraordinarily long NDA (>~150k tokens, ~300+ pages). In that case the architecture re-introduces retrieval as a Phase 2.5 component. Until that case appears in the eval set, the simpler design holds.

---

## Open questions

1. **Pactly's actual baseline NDA review time** — brief says "weeks." Need a number to commit the goal metric to a precise reduction (e.g. "from 14 days to 2 minutes").
2. **Standard Pactly NDA template availability.** If Legal has a canonical template, it can directly inform the Planner's checklist content (clause types and order to verify) — meaningfully strengthens the Failure Mode "Missed clause" defence.
3. **Legal's bandwidth to grade eval cases monthly.** The counter-metric (false-confidence ≤1%) is only as honest as the eval set. If Legal can't grade 50 cases / month, we need to either narrow the eval or fund the time.
4. **UI rendering of visual citations.** Clickable anchors that highlight the source paragraph require either an embedded PDF viewer with annotation overlay, or a side-by-side rendering. Which one fits Pactly's existing internal-tools tech stack?
5. **SaulLM-7B hosting.** Self-hosted (Pactly infrastructure) or via the HuggingFace Inference API? Hosting choice affects cost, latency, and the security review.
6. **Checklist content discipline.** Who owns what's on the standard-NDA-clause checklist? Legal should curate the v1 list (this is exactly the kind of thing the General Counsel would have strong views on); engineering wires it into the Planner prompt. Without curated checklist content, the missed-clause defence is theoretical.

---

## Review log

| Date | Reviewer | Version | What changed |
|---|---|---|---|
| 2026-04-29 | Benedict (group lead) + Claude (Opus 4.7) | v0.1 | Initial fill from project brief. Pattern: Pipeline + Critic with model diversity. **Final model lineup: Haiku 4.5 (Scope Classifier + Planner) → Sonnet 4.7 (Answer Agent) → SaulLM-7B (Critic).** Asymmetric reliability target (false-confidence ≤1% is the metric that matters). HITL-by-UX via visual citation. Scope Classifier at step 0. Question-complexity guardrail enforced at scope check. |
| 2026-05-01 | Benedict (group lead) + Claude (Opus 4.7) | v0.2 | **Architecture simplified after Phase 1 validation.** Dropped the embedding-based Retriever — full NDA fits in Sonnet 4.7's 200k context, retrieval was solving a non-problem. **Five agents → Four agents.** Planner reframed: now generates the standard-NDA-clause checklist (missed-clause defence by exhaustive coverage), no longer decomposes for retrieval. Cost target dropped from $0.20 → $0.10 per run; p50 latency from 30s → 20s. Added *Design revision* section with full rationale. |
