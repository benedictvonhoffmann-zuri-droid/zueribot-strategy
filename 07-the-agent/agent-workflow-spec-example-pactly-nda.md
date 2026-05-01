# Agent Workflow Spec — NDA Helper Agent (Pactly)

> **Example fill** of [`agent-workflow-spec-template.md`](agent-workflow-spec-template.md). Built for the Productschool Advanced AI Agents course final project — *not* a Bünzli artefact. Hosted here as a worked reference for the template.
>
> **The fictional setting (per course brief).** Pactly's Internal Tools division. BizDev users wait *weeks* for Legal to review NDAs, killing deal momentum. The agent is a "first-pass" PDF reviewer that answers specific BizDev questions in plain English, with verifiable citations and explicit escalation when uncertain. The Head of Partnerships wants speed; the General Counsel demands safety. The architecture must reconcile both.

---

## Identity

- **Agent name:** NDA Helper Agent
- **Owner:** Senior PM, Pactly Internal Tools division
- **Spec version:** v0.1 (course final-project draft)
- **Last updated:** 2026-04-29
- **Status:** Draft — buildable in LangFlow

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
| **Input** | (1) **PDF contract upload** — required, ≤25 MB, text-extractable preferred; OCR fallback for scanned PDFs. (2) **Specific natural-language question** — required, English, scope-checked before processing. Invalid input rejected upfront with a useful error message. |
| **Output** | Three-part structured response: **(1)** plain-English answer to the question; **(2)** verbatim quoted clauses from the NDA serving as citations; **(3)** explicit confidence signal — *"Confident / Send to Legal / Cannot answer from this document."* Each citation is a clickable anchor that opens the source PDF at the cited page with the relevant paragraph highlighted — see *HITL-by-UX* below. |
| **Primary Pattern** | **Pipeline + Critic, with model diversity.** Deterministic flow (Scope → Plan → Retrieve → Answer → Critic), not a free-form hierarchy. The Answer Agent and Critic use **different model families** so the verification is genuinely independent rather than the same model grading itself. |
| **Agent Roles** | See *Agent roles* table below. Five roles: Scope Classifier, Planner, Retriever, Answer Agent, Critic. Each has a defined anti-scope. |
| **Key Steps** | See *Key steps* below. Five high-level steps from upload to delivered answer. |
| **Governance Gates** | Four layers: pre-input file + scope validation; pre-output critic grounding check; HITL-by-UX visual citation; post-output weekly red-team. See *Governance gates* below. |
| **Success Metric** | **Primary:** correct-answer rate **≥95%** on a Legal-graded eval set. **Counter-metric:** false-confidence rate **≤1%** — the rate at which the agent confidently gives a wrong answer (without escalating). False *escalations* are tolerable; false *confident-wrong* are not. |

---

## Agent roles

| Role | Persona / job | Tools | Anti-scope |
|---|---|---|---|
| **Scope Classifier** | Step 0. Decides whether the question is "first-pass" answerable or requires Legal. Allowlist: clause existence, term lengths, summary, key obligations. Refuse-list: "should I sign," "negotiate this," "compare to industry standard," "what are the legal risks." | **Claude Haiku 4.5** | Does NOT attempt to answer the question; only routes. |
| **Planner** | Decomposes the user question into specific NDA clause topics to look for (e.g. "non-compete," "term length," "governing law," "confidentiality scope"). | **Claude Haiku 4.5** | Does NOT answer the question; does NOT retrieve text itself. |
| **Retriever** | For each topic in the plan, searches the PDF (chunked + embedded) for matching clauses. Returns candidate clauses with page + paragraph anchors. | PDF text extraction (PyMuPDF / pdfplumber); OCR fallback (Tesseract / vision model); embedding model for chunk-level retrieval; keyword search as parallel signal | Does NOT interpret what was found; does NOT compose the answer. |
| **Answer Agent** | Composes plain-English response from retrieved clauses. Quotes verbatim. Outputs the (1) answer + (2) citations + draft (3) confidence signal. | **Claude Sonnet 4.7** | Does NOT cite clauses outside the retrieved set; does NOT make claims unsupported by quoted text. |
| **Critic** | Verifies grounding. Checks that every quoted clause exists *verbatim* in the source PDF, that the plain-English answer is logically supported by the quotes, and that nothing material was missed. **Different model family from the Answer Agent** for independent verification. | **SaulLM-7B** (open-source legal-domain fine-tune) | Does NOT make up evidence to support a flaky answer; does NOT pass an answer it could not verify against the source. |

**Why two different model families.** Asking Claude Sonnet 4.7 to grade Claude Sonnet 4.7 rarely improves quality — same training, same blind spots, same hallucination patterns. Pairing the long-context generalist (Sonnet 4.7) for the Answer Agent with a legal-domain fine-tune (SaulLM-7B) for the Critic produces *genuinely independent* verification. When both models agree the answer is grounded, the confidence is materially higher than self-critique. This is the spec's strongest single design decision.

**Final model lineup.**

| Role | Model | Why |
|---|---|---|
| Scope Classifier + Planner | **Claude Haiku 4.5** | Cheap, fast, good enough for classification and decomposition. Cost-efficient on the high-volume steps. |
| Answer Agent | **Claude Sonnet 4.7** | Long-context, strong reasoning, reliable citation behaviour. The model that has to read the whole NDA + retrieved context and write a defensible plain-English answer. |
| Critic | **SaulLM-7B** | Different model family + legal-domain fine-tune = genuinely independent verification. Open-source so it's self-hostable, which keeps the verification path off any single hyperscaler. |

---

## Key steps

| # | Step | Tool / role | Output | Failure handling |
|---|---|---|---|---|
| 0 | **Validate + scope-check** | Scope Classifier (Haiku 4.5) | Either "in scope, proceed" or "out of scope, escalate to Legal with reason" | If out of scope, return a polite refusal with one-line reason. |
| 1 | **Parse + chunk PDF** | PDF extraction + embedding | Indexed clause chunks with page + paragraph anchors | If PDF is scanned + unreadable, fall back to OCR; if OCR fails, abort with "this PDF is not readable, please re-upload." |
| 2 | **Plan** | Planner (Haiku 4.5) | List of clause topics to search for | If question can't be decomposed (vague), return clarification request. |
| 3 | **Retrieve** | Retriever (embedding + keyword) | Candidate clauses per topic, with anchors | If no candidates found for a topic, mark that topic as "not found in document" — Answer Agent reports honestly. |
| 4 | **Compose** | Answer Agent (Sonnet 4.x) | Draft answer + citations + initial confidence | If the model refuses or returns low-quality output, escalate. |
| 5 | **Verify** | Critic (SaulLM-7B) | Pass/fail + reason | If fail: route back to Compose with critic feedback (one Reflexion loop max); if still fail, escalate to "Send to Legal" rather than ship. |
| 6 | **Deliver** | UI rendering layer | Plain-English answer + clickable verbatim citations + confidence label | UI highlights cited paragraphs in the source PDF — see HITL-by-UX. |

---

## Tools available

| Tool | Type | Used by | Used for | Cost / latency | Failure mode |
|---|---|---|---|---|---|
| PDF text extraction (PyMuPDF or pdfplumber) | Library | Step 1 | Extract text + structure from PDF | Free / <1s for typical NDA | If PDF is image-only, fall back to OCR |
| OCR (Tesseract or a vision LLM) | Library / API | Step 1 fallback | Extract text from scanned PDFs | $0.01–0.05 / 5–20s | If OCR confidence is low, abort with "PDF not readable" |
| Embedding model (e.g. text-embedding-3-small) | API | Step 1 + 3 | Index NDA chunks + retrieve relevant clauses | $0.001 / <1s | If API down, fall back to keyword search alone |
| Claude Haiku 4.5 | LLM | Steps 0, 2 | Scope check + planning | $0.001–0.005 / 1–3s | If down, fall back to a Mistral-class small model; if both down, abort |
| Claude Sonnet 4.7 | LLM | Step 4 | Compose answer with citations | $0.05–0.15 / 5–15s | If down, fall back to a Gemini 1.5 Pro / GPT-4 Turbo class model *(emergency only — flag clearly in the eval that a fallback model was used)* |
| SaulLM-7B | Self-hosted LLM | Step 5 | Critic / grounding verification | self-hosted compute / 3–10s | If down, default to "Send to Legal" rather than ship without verification |
| Standard NDA template KB *(optional, future)* | Internal KB | Step 4–5 | Compare retrieved clauses against Pactly-standard clauses to flag deviations | TBD | Optional — if KB unavailable, agent reports "deviation flag unavailable" |

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

---

## Failure modes

| Failure mode | Likelihood | Severity | Mitigation |
|---|---|---|---|
| **Hallucination** — agent claims a clause exists when it doesn't | H | H | Critic verifies every quoted clause is verbatim in source; refuse if not anchored. Different model families for Answer vs Critic = independent verification. |
| **Missed clause** — agent says "no non-compete" when a buried non-compete exists | M | H | Planner uses a checklist of standard NDA clause types so retrieval is exhaustive, not question-driven alone. Retriever uses both embedding + keyword search. On critical questions, default to "Send to Legal" if any retrieval signal is weak. |
| **Misinterpretation** — correct quote, wrong plain-English explanation | M | M | Critic checks logical consistency between quote and explanation. UI surfaces the quote + visual citation prominently so the user can verify. |
| **Adversarial PDF** — malformed file, prompt injection ("ignore previous instructions") embedded in NDA text | L | H | Input sanitisation; system-prompt hardening (instructions in PDF text are treated as data, not instructions); refuse if doc shape is clearly off. |
| **Scope creep** — user asks a non-NDA question or one requiring legal judgement | M | M | Scope Classifier (Step 0) gates this before any document processing. Returns clear refusal with reason. |

---

## Governance gates

| Gate | Layer | What it checks | What happens on fail |
|---|---|---|---|
| File validation | pre-input | PDF format, ≤25 MB, text-extractable (OCR fallback) | Reject with helpful error |
| Scope check | pre-input | Question is in the "first-pass" allowlist; not in the refuse-list | Return refusal with one-line reason and "this requires Legal" |
| Critic grounding check | pre-output | Every quoted clause exists verbatim in source; answer is logically supported by quotes | Reflexion loop once; if still fail, escalate to "Send to Legal" |
| **HITL-by-UX (visual citation)** | pre-output | Every claim in the answer is rendered with a clickable anchor that opens the source PDF at the cited page with the relevant paragraph highlighted | The user verifies by virtue of reading the highlighted clause — no explicit approval click required, but the UX makes the verification effectively unavoidable. The user is in the loop because the UI makes it so. |
| Post-output red-team | post-output | Weekly run of 20 known-edge-case NDAs (buried non-competes, non-standard numbering, foreign governing law, scanned PDFs, prompt-injection cases) against the deployed agent | Alert on any silent regression; gate next deploy on green. |

---

## Cost & latency budgets

- **Cost target per successful run:** ≤$0.20 (5 LLM calls + embedding + parsing).
- **Cost ceiling per run:** $1.00 (hard cap; aborts and escalates if exceeded).
- **p50 latency target:** ≤30 seconds end-to-end.
- **p95 latency target:** ≤90 seconds.
- **Latency fallback:** if the Critic times out, default to "Send to Legal" rather than ship an unverified answer. Saying *"this needs Legal"* is always a safer outcome than saying *"yes/no"* with no verification.

---

## Human-in-the-loop (HITL)

This agent has no destructive actions (no email sent, no system of record updated), so there are no traditional HITL approval gates. Two HITL patterns are in play:

1. **Self-escalation to Legal** — when the agent is uncertain (Critic fails, scope check fails, retrieval is weak), it voluntarily routes the question to Legal rather than guessing. This is a HITL pattern *driven by the agent itself* — the most reliable form because it doesn't require the user to know they should escalate.
2. **HITL-by-UX (visual citation)** — every claim renders with a clickable, highlighted anchor in the source PDF. The user verifies by reading the highlighted clause. The verification is structural to the UX, not a separate approval step. The user *cannot* trust a claim without seeing what it was based on, because the UI makes the basis prominent.

Optional v2: **sample-rate Legal review.** 5% of agent answers reviewed by Legal post-hoc to calibrate the Critic. Drives eval-set refresh and surfaces silent failure modes.

---

## Eval set

- **Size:** 50 NDAs at v1 (course-deliverable scope), target 200 for production.
- **Composition:**
  - **40% — Standard NDAs / happy path.** Common clauses, common questions. The agent should cruise these.
  - **30% — Edge cases.** Buried non-competes, non-standard clause numbering, foreign governing law, unusually long/short terms, mutual vs unilateral asymmetries.
  - **20% — Adversarial.** Malformed PDFs, prompt-injection attempts, scope-creep questions.
  - **10% — Scanned / OCR-required PDFs.**
- **Source:** Synthetic NDAs hand-crafted by Legal team + sampled real Pactly NDAs with PII redacted.
- **Refresh cadence:** Monthly. Failures from production reviewed every release; new edge cases promoted to permanent eval.
- **Reviewers:** Legal grades correctness; PM grades UX clarity; automated grounding-check (every cited clause verbatim in source) runs in CI on every prompt or model change.

---

## Open questions

1. **Pactly's actual baseline NDA review time** — brief says "weeks." Need a number to commit the goal metric to a precise reduction (e.g. "from 14 days to 2 minutes").
2. **Standard Pactly NDA template availability.** If Legal has a canonical template, it becomes a powerful reference KB for clause-deviation flagging — meaningfully strengthens Failure Mode "Missed clause" defence.
3. **Legal's bandwidth to grade eval cases monthly.** The counter-metric (false-confidence ≤1%) is only as honest as the eval set. If Legal can't grade 50 cases / month, we need to either narrow the eval or fund the time.
4. **UI rendering of visual citations.** Clickable anchors that highlight the source paragraph require either an embedded PDF viewer with annotation overlay, or a side-by-side rendering. Which one fits Pactly's existing internal-tools tech stack?
5. **SaulLM-7B hosting.** Self-hosted (Pactly infrastructure) or via an inference provider? Hosting choice affects cost, latency, and the security review.

---

## Review log

| Date | Reviewer | Version | What changed |
|---|---|---|---|
| 2026-04-29 | Benedict (group lead) + Claude (Opus 4.7) | v0.1 | Initial fill from project brief. Pattern: Pipeline + Critic with model diversity. **Final model lineup: Haiku 4.5 (Scope Classifier + Planner) → Sonnet 4.7 (Answer Agent) → SaulLM-7B (Critic).** Asymmetric reliability target (false-confidence ≤1% is the metric that matters). HITL-by-UX via visual citation. Scope Classifier at step 0. Question-complexity guardrail enforced at scope check. |
