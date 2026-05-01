# Agent Workflow Spec — NDA Helper Agent (Pactly)

> **Example fill** of [`agent-workflow-spec-template.md`](agent-workflow-spec-template.md). Built for the Productschool Advanced AI Agents course final project — *not* a Bünzli artefact. Hosted here as a worked reference for the template.
>
> **The fictional setting (per course brief).** Pactly's Internal Tools division. BizDev users wait *weeks* for Legal to review NDAs, killing deal momentum. The agent is a "first-pass" PDF reviewer that answers specific BizDev questions in plain English, with verifiable citations and explicit escalation when uncertain. The Head of Partnerships wants speed; the General Counsel demands safety. The architecture must reconcile both.
>
> **Headline architecture (v0.3): Senior Counsel + Analyst + Copywriter.** Each model is matched to the *character* of the work it does. **SaulLM-7B is the Senior Counsel** — plans the clause checklist and critiques the draft for grounding + coverage. **Claude Haiku 4.5 is the Analyst** — fast triage on whether the question is even in scope. **Claude Sonnet 4.7 is the Legal Copywriter** — reads the full NDA and drafts the plain-English answer from the quoted clauses. Plan and critique are *legal-judgement* tasks; drafting from quoted text is a *writing* task. Mapping models to job-character is sharper than a single-model-everywhere split.

---

## Identity

- **Agent name:** NDA Helper Agent
- **Owner:** Senior PM, Pactly Internal Tools division
- **Spec version:** v0.3 (role-character mapping — see *Design revision* below)
- **Last updated:** 2026-05-01
- **Status:** Phase 1 + Phase 2 (Haiku-Planner) validated in LangChain. Saul-as-Planner swap planned for Phase 4 alongside Critic wiring.

---

## Connection to broader strategy

- **AI value archetype:** **Copilot.** The agent does not replace Legal — it gives BizDev a defensible first-pass answer so they can decide whether they need Legal at all. Not Automator (no actions taken on user's behalf), not Oracle (more than facts, requires interpretation), not Orchestrator (single document, no cross-system coordination). Copilot frames the right expectation: it augments human judgement rather than substituting for it.
- **Strategic role:** Internal velocity tool — clears the NDA backlog on routine questions while preserving Legal's role on the questions that genuinely require legal judgement.
- **Reliability contract:** **Asymmetric.** False-confident-wrong is catastrophic (BizDev signs based on a wrong answer); false-escalation-to-Legal is merely annoying. The system is engineered to fail in the safer direction — when uncertain, it refuses and routes to Legal.

---

## The role-character mapping

| Character | Model | Job |
|---|---|---|
| **Senior Counsel** | **SaulLM-7B** | Plans the clause checklist (what to look for) **and** critiques the draft (was every checklist item addressed; is every quote anchored verbatim in source). Legal-judgement work. Appears twice in the workflow — same character, two scenes. |
| **Analyst** | **Claude Haiku 4.5** | Triage gate. Decides whether the question is "first-pass" answerable or out of scope. Cheap, fast — keeps Saul's attention for actual legal work. |
| **Legal Copywriter** | **Claude Sonnet 4.7** | Reads the full NDA + question + Senior Counsel's checklist; drafts the plain-English answer with verbatim citations. Long-context comprehension and writing skill — defers to Senior Counsel on legal judgement. |

This is the spec's strongest design framing. Each model does what its training and capability are *actually* good at. The deck story writes itself.

---

## The spec table

| Field | Spec |
|---|---|
| **Business Goal** | Reduce BizDev's time-to-first-pass-answer on standard NDAs from weeks (waiting on Legal review) to **<2 minutes for ≥80% of common questions**, without ever shipping a confident-wrong answer. Baseline today: median NDA review wait is multiple weeks; BizDev cannot self-serve any question, however basic. |
| **Input** | (1) **PDF contract upload** — required, ≤25 MB, text-extractable preferred; OCR fallback deferred to a later phase. (2) **Specific natural-language question** — required, English, scope-checked before processing. Invalid input rejected upfront with a useful error message. |
| **Output** | Three-part structured response: **(1)** plain-English answer to the question; **(2)** verbatim quoted clauses from the NDA serving as citations *(with page anchors from the PyPDF page-level loader)*; **(3)** explicit confidence signal — *"Confident / Send to Legal / Cannot answer from this document."* Each citation is a clickable anchor that opens the source PDF at the cited page with the relevant paragraph highlighted — see *HITL-by-UX* below. |
| **Primary Pattern** | **Pipeline + Critic, with role-character model mapping.** Deterministic flow (Scope → Plan → Answer → Critic), not a free-form hierarchy. The Answer Agent (Sonnet) and Critic (Saul) use **different model families** so the verification is genuinely independent rather than the same model grading itself. *No embedding-based retrieval — the document fits in context.* |
| **Agent Roles** | See *Agent roles* table below. **Four roles, three model characters**: Analyst (Scope Classifier), Senior Counsel (Planner), Legal Copywriter (Answer Agent), Senior Counsel (Critic). |
| **Key Steps** | See *Key steps* below. Five high-level steps from upload to delivered answer. |
| **Governance Gates** | Four layers: pre-input file + scope validation; pre-output critic grounding + coverage check; HITL-by-UX visual citation; post-output weekly red-team. See *Governance gates* below. |
| **Success Metric** | **Primary:** correct-answer rate **≥95%** on a Legal-graded eval set. **Counter-metric:** false-confidence rate **≤1%** — the rate at which the agent confidently gives a wrong answer (without escalating). False *escalations* are tolerable; false *confident-wrong* are not. |

---

## Agent roles

| Role | Character | Model | Persona / job | Anti-scope |
|---|---|---|---|---|
| **Scope Classifier** | Analyst | **Haiku 4.5** | Step 0. Decides whether the question is "first-pass" answerable or requires Legal. Allowlist: clause existence, term lengths, summary, key obligations. Refuse-list: "should I sign," "negotiate this," "compare to industry standard," "what are the legal risks." | Does NOT attempt to answer the question; only routes. |
| **Planner** | Senior Counsel | **SaulLM-7B** | Generates a **question-relevant clause checklist** the Answer Agent must verify across the document. Items are *direct* (clauses the question literally asks about) + *adjacent* (clauses that could *affect* the answer — e.g. for a non-compete question, also non-solicitation, restraint of trade, IP assignment, residuals). Drives exhaustive coverage of every dimension that matters for *this specific question*, while keeping the list focused. | Does NOT answer the user's question; does NOT pad with manifestly irrelevant items for the sake of length. |
| **Answer Agent** | Legal Copywriter | **Sonnet 4.7** | Reads the full NDA text + the user's question + the Senior Counsel's checklist. Reports on *every* checklist item explicitly (Present / Not present / Unclear, with verbatim quote + page anchor where present). Then composes a plain-English synthesis answering the user's question. Long-context (200k tokens — comfortably holds the whole NDA + checklist + question). | Does NOT make claims unsupported by the document text; does NOT skip checklist items even if the user didn't ask about them — silent omission is itself a failure. Defers to Senior Counsel on questions of legal judgement. |
| **Critic** | Senior Counsel | **SaulLM-7B** | Verifies grounding *and* checklist coverage. Checks that every quoted clause exists *verbatim* in the source PDF, that the plain-English answer is logically supported by the quotes, and that **every checklist item from the Planner was addressed** (no silent omissions). | Does NOT make up evidence to support a flaky answer; does NOT pass an answer it could not verify against the source. |

**Why Sonnet vs. Saul for Answer vs. Critic.** Asking Claude Sonnet to grade Claude Sonnet rarely improves quality — same training corpus, same blind spots, same hallucination patterns. Pairing the long-context generalist (Sonnet 4.7) for the Answer Agent with a legal-domain fine-tune (SaulLM-7B) for the Critic produces *genuinely independent* verification. When models from two different families agree the answer is grounded, the confidence is materially higher than self-critique.

**Why Saul plays both Planner *and* Critic.** Plan and critique are both *legal-judgement* tasks: what should we look for, and was the look-for done well? A model trained on legal text answers both questions better than a general-purpose model. The two appearances are the same *character* in the workflow — same as a real senior counsel might brief a junior on what to look for, then review their work afterwards. The trade-off: Saul-the-Planner's blind spots in clause identification become Saul-the-Critic's blind spots in coverage verification. We accept that — the alternative (Sonnet-as-Critic) breaks the more important Answer-vs-Critic independence.

---

## Key steps

| # | Step | Character / Model | Output | Failure handling |
|---|---|---|---|---|
| 0 | **Validate + scope-check** | Analyst (Haiku 4.5) | Either "in scope, proceed" or "out of scope, escalate to Legal with reason" | If out of scope, return a polite refusal with one-line reason. |
| 1 | **Parse PDF** | PyPDF page-level loader | Full document text + per-page metadata (for citation anchors) | If PDF is unparseable / image-only, abort with "this PDF is not readable, please re-upload." OCR fallback deferred. |
| 2 | **Plan checklist** | Senior Counsel (SaulLM-7B) | Question-relevant clause checklist (direct + adjacent items, JSON) | If checklist generation or JSON parsing fails, retry once; on second failure abort with escalation. *(JSON-reliability mitigation: see Open Questions.)* |
| 3 | **Compose** | Legal Copywriter (Sonnet 4.7) | Per-checklist-item coverage report + verbatim citations with page anchors + plain-English synthesis + initial confidence | If the model refuses or returns low-quality output, retry once; on second failure escalate. |
| 4 | **Verify** | Senior Counsel (SaulLM-7B) | Pass/fail + reasons (grounding + coverage) | If fail: route back to Compose with critic feedback (one Reflexion loop max); if still fail, escalate to "Send to Legal" rather than ship. |
| 5 | **Deliver** | UI rendering layer | Plain-English answer + clickable verbatim citations + confidence label | UI highlights cited paragraphs in the source PDF — see HITL-by-UX. |

---

## Tools available

| Tool | Type | Used by | Used for | Cost / latency | Failure mode |
|---|---|---|---|---|---|
| PyPDF page-level loader | Library | Step 1 | Extract text + per-page metadata from PDF | Free / <1s for typical NDA | If PDF is image-only, abort (OCR fallback deferred) |
| Claude Haiku 4.5 | LLM | Step 0 | Scope check | $0.001–0.005 / 1–3s | If down, fall back to a Mistral-class small model; if both down, abort |
| Claude Sonnet 4.7 | LLM | Step 3 | Compose answer with citations | $0.05–0.15 / 5–15s | If down, fall back to a Gemini 1.5 Pro / GPT-4 Turbo class model *(emergency only — flag clearly in the eval that a fallback model was used)* |
| SaulLM-7B *(Planner)* | Self-hosted / HF Inference API | Step 2 | Generate question-relevant clause checklist | self-hosted compute / 3–10s warm, 30–60s cold | If down, **fall back to Haiku 4.5 as Planner** (validated working in Phase 2) — degrades the Senior Counsel framing but keeps the system shipping. |
| SaulLM-7B *(Critic)* | Self-hosted / HF Inference API | Step 4 | Critic / grounding + coverage verification | self-hosted compute / 3–10s warm | If down, default to "Send to Legal" rather than ship without verification. **Never ship without Critic.** |

*No embedding model, no vector DB. The full NDA fits in Sonnet's context.*

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
| **Hallucination** — agent claims a clause exists when it doesn't | H | H | Critic verifies every quoted clause is verbatim in source; refuse if not anchored. Different model families for Answer (Sonnet) vs Critic (Saul) = independent verification. |
| **Missed clause** — agent says "no non-compete" when a buried non-compete exists | M | H | **Senior Counsel's question-relevant checklist** forces direct + adjacent coverage. Answer Agent must report on every checklist item, including silent omissions ("not present in this document"). Critic verifies checklist coverage. |
| **Misinterpretation** — correct quote, wrong plain-English explanation | M | M | Critic checks logical consistency between quote and explanation. UI surfaces the quote + visual citation prominently so the user can verify. |
| **Adversarial PDF** — malformed file, prompt injection ("ignore previous instructions") embedded in NDA text | L | H | Input sanitisation; system-prompt hardening (instructions in PDF text are treated as data, not instructions); refuse if doc shape is clearly off. |
| **Scope creep** — user asks a non-NDA question or one requiring legal judgement | M | M | Analyst (Step 0) gates this before any document processing. Returns clear refusal with reason. |
| **Saul-Planner / Saul-Critic correlated blind spot** — Senior Counsel misses clause type X in planning, misses absence of X in critique | L | M | Open question; mitigation likely a periodic eval-set sweep that benchmarks Saul against Sonnet on coverage edge cases. Acceptable trade for the verification-independence gain. |

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

- **Cost target per successful run:** ≤$0.12 (Haiku scope + Saul plan + Sonnet answer + Saul critique). Slightly higher than v0.2 because Saul fires twice; Saul self-hosted is cheap per call but adds compute.
- **Cost ceiling per run:** $0.50 (hard cap; aborts and escalates if exceeded).
- **p50 latency target:** ≤30 seconds end-to-end (assumes Saul warm; cold-start adds 30–60s on the first run after idle).
- **p95 latency target:** ≤75 seconds.
- **Latency fallback:** if the Critic times out, default to "Send to Legal" rather than ship an unverified answer. If the Planner times out (Saul cold-start blocking the run), fall back to the Haiku-Planner path validated in Phase 2.

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

## Design revision — role-character mapping (v0.2 → v0.3)

The v0.2 spec used Haiku 4.5 for the Scope Classifier *and* the Planner — a "small fast model for everything cheap" pattern. After Phase 2 validated that the Haiku-driven Planner produced sensible question-relevant checklists, we revisited the model assignments and noticed an architectural mismatch.

**The insight:** plan and critique are both *legal-judgement* tasks. Drafting plain-English from quoted clauses is a *writing* task. Triage is a *fast-classification* task. We were using the same model (Haiku) for two of those — not because it was the right tool for both, but because it was the cheap default.

**The revision:** map each model to the *character* of the work it does best.

- **SaulLM-7B = Senior Counsel.** Plans the checklist (legal judgement: what clauses matter for this question, including adjacency awareness) AND critiques the draft (legal judgement: was every checklist item addressed; is every quote anchored verbatim). Same character, two scenes — same as a real senior counsel might brief a junior on what to look for, then review the work afterwards.
- **Haiku 4.5 = Analyst.** Triage on whether the question is even worth Senior Counsel's time (scope check). Cheap, fast, doesn't pretend to do legal judgement.
- **Sonnet 4.7 = Legal Copywriter.** Reads the full NDA + checklist + question, drafts the plain-English answer with quoted clauses. Has the comprehension and writing chops; defers to Senior Counsel on legal judgement.

**What this revision does *not* change:**
- Architecture is still Sequential pipeline + Critic-Reflexion (deterministic, not hierarchical).
- Number of agents (4) and number of handoffs unchanged.
- Answer-vs-Critic independence preserved (Sonnet ≠ Saul by family — this is the verification independence claim, untouched).

**What it does change:**
- Saul appears twice (Planner + Critic), introducing a small *internal* correlation: Saul-the-Planner's blind spots in clause identification become Saul-the-Critic's blind spots in coverage verification. We accept this — the alternative (Sonnet-as-Critic) would break the more important Answer-vs-Critic independence.
- Cost target slightly higher ($0.10 → $0.12) and latency wider ($p95: 60s → 75s) due to Saul firing twice and HF Inference cold-start exposure on the first call.
- A new fallback path: if Saul's JSON output is unreliable for the Planner role, **invest in structured-output schemas** (e.g. constrained decoding, output validation + retry) rather than reverting to Haiku-Planner — preserving the Senior Counsel framing matters for the architecture.

The deck story is sharper too: *"Each model does what its training and capability are actually good at — Senior Counsel for legal judgement, Analyst for triage, Copywriter for plain-English drafting."*

---

## Open questions

1. **Pactly's actual baseline NDA review time** — brief says "weeks." Need a number to commit the goal metric to a precise reduction (e.g. "from 14 days to 2 minutes").
2. **Standard Pactly NDA template availability.** If Legal has a canonical template, it can directly inform the Planner's checklist content (clause types and order to verify) — meaningfully strengthens the Failure Mode "Missed clause" defence.
3. **Legal's bandwidth to grade eval cases monthly.** The counter-metric (false-confidence ≤1%) is only as honest as the eval set. If Legal can't grade 50 cases / month, we need to either narrow the eval or fund the time.
4. **UI rendering of visual citations.** Clickable anchors that highlight the source paragraph require either an embedded PDF viewer with annotation overlay, or a side-by-side rendering. Which one fits Pactly's existing internal-tools tech stack?
5. **SaulLM-7B hosting.** Self-hosted (Pactly infrastructure) or via the HuggingFace Inference API? Hosting choice affects cost, latency, and the security review.
6. **Checklist content discipline.** Who owns what's on the standard-NDA-clause checklist (and the prompts that generate question-relevant additions)? Legal should curate the v1 anchor list (this is the General Counsel's territory); engineering wires it into the Planner prompt.
7. **Saul JSON output reliability.** SaulLM is fine-tuned for legal *text*, not structured JSON. **Decision (2026-05-01):** if reliability falls below 95% on the Planner role, *invest in structured-output schemas* (constrained decoding, output validation + retry) rather than revert to Haiku-Planner. Preserving the Senior Counsel framing is worth the engineering cost.

---

## Review log

| Date | Reviewer | Version | What changed |
|---|---|---|---|
| 2026-04-29 | Benedict (group lead) + Claude (Opus 4.7) | v0.1 | Initial fill from project brief. Pattern: Pipeline + Critic with model diversity. Final model lineup: Haiku 4.5 (Scope Classifier + Planner) → Sonnet 4.7 (Answer Agent) → SaulLM-7B (Critic). Asymmetric reliability target. HITL-by-UX via visual citation. Scope Classifier at step 0. |
| 2026-05-01 | Benedict (group lead) + Claude (Opus 4.7) | v0.2 | Architecture simplified after Phase 1 validation. Dropped the embedding-based Retriever — full NDA fits in Sonnet 4.7's 200k context. Five agents → four. Planner reframed: now generates the question-relevant clause checklist (direct + adjacent items). Cost target halved ($0.20 → $0.10); latency targets cut. Added *Design revision* section. |
| 2026-05-01 | Benedict (group lead) + Claude (Opus 4.7) | v0.3 | **Role-character mapping.** Saul promoted from Critic-only to Senior Counsel (Planner + Critic). Haiku stays as Analyst (Scope only). Sonnet remains Legal Copywriter (Answer). Each model now does what its training is actually best at. Deck framing baked into spec. New open question on Saul JSON reliability — decision: invest in output schemas if needed, do not revert to Haiku. Cost target $0.10 → $0.12; p95 latency 60s → 75s. |
