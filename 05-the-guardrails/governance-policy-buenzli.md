# Governance Policy — Bünzli

> Filled instance of [`governance-policy-template.md`](governance-policy-template.md). Snapshot dated 2026-05-02.
>
> **Headline.** Bünzli is **EU AI Act limited-risk** (consumer-facing generative AI; no decisions about people; no high-risk classification). **revDSG-primary** for data protection given Swiss hosting. Autonomy boundaries are deliberately tight: agents *answer*, users *act* — every Züri wie neu submission, every Quandoo booking, every ERZ check requires explicit user confirmation before execution. Shadow AI is not currently a problem at team-of-one scale but a plausible community-contribution scenario is worth designing controls against now, before the contribution channel goes live.

---

## Connection to other modules

- **`compounding-system-buenzli.md`** — the operational sibling.
- **`04-the-contract/eval-strategy-buenzli.md`** — audit cadence in this doc must match the eval cadence there (monthly red-team + weekly telemetry + sample-rate review).
- **`07-the-agent/agent-workflow-spec-example-pactly-nda.md`** — the agent topology is a governance lens on what the AW Spec defined.
- **`01-the-bet/diagnostic-buenzli.md` — Axis 3 (Platform exposure = 4)** — limited-risk classification protects Axis 3; high-risk classification would force massive compliance investment and effectively force out of consumer market.

---

## Scope

- **Products / surfaces:** Bünzli web app (buenzli.space + future mobile). Asksaul.ch is out of scope (separate parody / educational product with its own governance disclaimer).
- **User-facing AI behaviour included:** all Bünzli responses, all agentic actions proposed on user behalf (Züri wie neu, ERZ, Quandoo), all KB content surfaced via search.
- **Internal-facing AI use:** Benedict's use of LLMs for spec-writing and prototyping (out of scope of this consumer-facing policy; covered by general professional discretion).
- **What this policy explicitly does NOT cover:** AI used by community contributors for their own purposes (out of scope); AI used by Pactly's NDA helper / asksaul.ch (separate product).

---

## Autonomy boundaries

| Action class | OK solo | Needs human |
|---|---|---|
| Information retrieval from KB | ✓ | |
| Composing plain-Swiss-German answers from retrieved content | ✓ | |
| Refusing out-of-scope queries (legal advice, medical referral, financial recommendation) | ✓ — autonomous refusal | |
| Suggesting agentic actions (drafting a Züri wie neu report, finding ERZ schedule, proposing Quandoo time slot) | ✓ — propose | |
| **Executing agentic actions on user behalf** | | **✓ — explicit user confirmation per action, no exceptions** |
| Modifying user-saved preferences (address, household composition) | | ✓ — user edits directly via settings UI, not via chat |
| Reading another user's data | | **Never.** Architecturally impossible per privacy posture. |
| Cross-user actions / aggregating user behaviour | | **Never** for personally-identifiable cross-user data. Aggregate-only per privacy posture; no per-user attribution. |
| Affecting external services in ways that cost user money | | ✓ — explicit user confirmation + clear cost disclosure (e.g. Quandoo booking commission) |

**The principle made operational:** *Bünzli's agents answer; Bünzli's users act.* Every external-world effect requires user confirmation. Asymmetric design — false refusals are tolerable (user just asks again); false confident actions are not (a Züri wie neu report filed in someone's name with wrong content is a real-world harm).

---

## Escalation triggers

| Trigger | What happens |
|---|---|
| Agent confidence < 70% | Refuse + route to "contact authority directly" (e.g. Stadt Zürich, ERZ, ZVV) with contact info |
| Out-of-scope query (legal / medical / financial / negotiation) | Refused at Scope Classifier; routed to appropriate professional |
| User flags an answer wrong via "war das richtig?" | Routed to LLM-as-judge processing → eval set addition per M4 |
| Agentic action proposed | UI explicitly confirms before execution; user sees exactly what will be submitted |
| Critic fails twice on a draft answer | Drop to "I'm not confident enough — please consult [authority]" |
| Production telemetry anomaly (refusal rate > 25%, p95 latency > 30s, judge-pass rate < 80%) | Benedict investigates same-day at v1; on-call PM at scale |
| Prompt-injection / authority-claim attack detected | Logged + reviewed in monthly red-team session; pattern added to adversarial set |
| Cost ceiling exceeded per query (>CHF 0.10) | Abort + alert (per M3 cost-curve stress tests) |
| EU AI Act / revDSG audit request | Privacy/Legal review within statutory deadline |

---

## Audit cadence

| Audit type | Frequency | Owner at v1 | Owner at scale | Output |
|---|---|---|---|---|
| Eval suite (full) | Per release + monthly | Benedict | PM + Eng | Pass/fail vs. reliability contract per M4 |
| Red-team / adversarial | Monthly | Benedict | PM + rotating security reviewer | New rows in adversarial eval set |
| Production telemetry review | Weekly | Benedict | PM | Trends + flagged anomalies |
| Sample-rate output review (5%) | Weekly post-launch | Benedict | Swiss-German-fluent reviewer (paid hourly post open beta) | Drift signal + LLM-as-judge calibration |
| Privacy / data-handling review | Quarterly | Benedict | Designated Privacy Officer | No PII drift; data-processing register up to date |
| Regulatory exposure review | Quarterly + on any law change | Benedict (with external legal consult as needed) | Legal | Risk-tier assessment unchanged or updated |
| Full governance review | Annually | Cross-functional | Cross-functional | This doc updated; public commitments re-affirmed |

---

## Regulatory exposure

### EU AI Act risk classification

- **Tier:** **Limited-risk**
- **Rationale:**
  - **Not unacceptable** — no social scoring, no real-time biometric ID, no manipulation patterns, no exploitation of vulnerable groups.
  - **Not high-risk** — Bünzli does not make decisions about people (no credit/employment/education/healthcare/law-enforcement decisions). Bünzli is not safety-critical infrastructure. Agentic actions are proposals to the user, not autonomous decisions affecting third parties.
  - **Limited-risk applies** — Bünzli is a consumer-facing chatbot that uses generative AI. Article 50 of the AI Act applies: transparency obligations require labelling AI-generated content and disclosing the use of AI to users.
- **Article 50 transparency status:** **In progress.** Footer disclosure + "AI-generated" labelling on every response is part of v1 scope. Privacy / about page documents the AI usage clearly.
- **What would push it up a tier:** if Bünzli ever auto-files reports without user confirmation, or makes decisions about user eligibility for services, or starts cross-user data flows — all of which we explicitly architect against.

### Data protection

- **Applicable regime:** **revDSG primary** (Swiss hosting, primarily Swiss users), with **GDPR alignment** for any EU users.
- **Data-processing register status:** **In progress at v1.** Minimal because the privacy posture is no-PII-storage: no per-user history, no question logs containing user identifiers, no answer logs containing user identifiers.
- **Data-subject obligations:**
  - **Access / rectification / deletion / portability:** trivially honoured because there is no per-user data to access, rectify, delete, or port. The privacy-as-policy stance is also the compliance-by-design stance.
- **PII storage policy:** **None.** PDFs uploaded for the NDA helper (asksaul.ch — out of scope of this policy) are wiped on session end. Bünzli itself processes Quartier addresses but does not persist them beyond the user's own browser session unless the user explicitly opts into saving preferences locally.
- **Cross-border data flows:** **None outside CH/EU.** Infomaniak hosting is Zürich-based. Apertus runs on Swiss/EU compute. Mistral is European.

### Sector-specific regulation

- **Sector(s) applicable:** Civic-AI / consumer information service. No formal Swiss sector regulation specific to AI civic assistants at time of writing.
- **Adjacent obligations to monitor:**
  - Swiss e-government strategy and any forthcoming federal AI legislation
  - Stadt Zürich's data-use policy if formal procurement / partnership conversation happens
  - Children's privacy / DSG-Kinderschutz if Bünzli usage skews toward minors

---

## Agent topology — governance lens

Cross-reference: `07-the-agent/agent-workflow-spec-example-pactly-nda.md` for the architecture; this section is the governance commitment about what each agent in Bünzli's runtime is allowed to do.

| Agent role | What it CAN do | What it CAN'T do | Who approved |
|---|---|---|---|
| **Senior Counsel (Apertus voice)** | Generate plain-Swiss-German answers from verified retrieved content + Senior-Counsel checklist | Read user data; modify the KB; execute agentic actions; access another user's session | Benedict (PM); reviewed in this doc |
| **Analyst (Mistral tools)** | Triage queries; extract from PDFs; structure tool calls; retrieve from KB | Compose user-facing voice answers (that's Apertus's job); execute external actions | Benedict; reviewed in this doc |
| **(Orchestration layer)** | Route between agents per LangGraph state machine; manage shared scratch within a single run | Persist user state across sessions; cross-user routing; modify its own routing rules at runtime | Benedict; reviewed in this doc |

Common-pattern commitments made explicit:
- **No agent reads or writes another user's data.** Enforced architecturally (no cross-user data structures exist) and at policy level.
- **No agent executes external-world actions without explicit user confirmation.** The orchestrator surfaces a confirmation UI for every Züri wie neu / ERZ / Quandoo action; the action only fires on user click.
- **No agent modifies its own prompts or routing rules at runtime.** Prompt templates are versioned + reviewed in PRs; no self-modification.
- **No agent calls external APIs without rate-limit + cost-cap.** Per M3 cost ceiling: $1.00 hard cap per session.
- **No agent makes decisions about people.** Excluded by design per Autonomy Boundaries.

---

## Shadow AI audit

**Status at v1: not currently applicable.** Bünzli is a team of one (Benedict). There are no rogue Copilot subscriptions, no employees using ChatGPT for confidential work, no third parties using AI on Bünzli's platform surfaces.

**But — the hypothetical scenario worth thinking through now, before it bites:**

| Hypothetical pattern | Owner / source | Risk level | Decision / control |
|---|---|---|---|
| **Community contributors use ChatGPT to generate Quartier-Wissen entries** | Bünzli community contributors (once the contribution channel ships v1.5) | **M** — risks polluting the curated KB with AI-generated content that's wrong, generic, or hallucinated; conflicts with the "community-curated" claim that makes the moat real | **Govern.** Detection: linguistic uniformity check on submissions; submission-velocity flag (one contributor → 50 entries in a day = red flag); periodic human spot-check by domain reviewer. Action: flag suspected AI-generated submissions for review; reject if not authentic; community guidelines explicitly require human-authored contributions. |
| **Third-party developers use Bünzli's API to power their own AI agents without disclosure** | External developers (post-API-launch) | **L** — at v1 there's no public API | **N/A at v1.** Will need API ToS at launch covering AI-disclosure obligations downstream. |
| **Internal use of LLMs for prompt-tuning + spec-writing** | Benedict | **L** — already happening (this very document) | **Keep.** No confidential user data ever pasted into upstream LLMs; documentation of usage in this very policy. |

**Summary:**
- **Total patterns identified:** 3 (1 hypothetical-near-term, 2 lower-priority)
- **Patterns after triage:** 1 active control (community-contribution AI-generation detection) deferred to v1.5 when the contribution channel ships
- **Estimated hidden spend:** N/A
- **Detection mechanism:** ad-hoc spot-check + community guidelines at v1; automated linguistic-uniformity check at v1.5

---

## Open questions

1. **Article 50 transparency labelling — exact wording and placement** on Bünzli responses. Needs review against Article 50 implementation guidance once published.
2. **Privacy Officer designation** — at what scale (active user count or revenue threshold) does Bünzli need a designated Privacy Officer rather than Benedict-handling-it? My estimate: at 10,000 MAU or any B2G procurement contract, whichever comes first.
3. **EU users specifically** — Bünzli is Zürich-focused but some users will access from across the border (Germany / France / Italy / Austria). At what volume do EU-resident user counts trigger GDPR-primary treatment over revDSG?
4. **AI Act enforcement timeline in Switzerland** — Switzerland is not in the EU; how does the AI Act apply to Swiss-hosted AI serving EU users? Needs legal review at v2 scale.
5. **Community-contribution AI-generation policy** — exact wording for community guidelines; threshold for human-spot-check workload.

---

## Review log

| Date | Reviewer | What changed |
|---|---|---|
| 2026-05-02 | Benedict + Claude (Opus 4.7) | Initial fill. EU AI Act classified as limited-risk with Article 50 transparency status in-progress. revDSG primary regime; no-PII-storage makes data-subject obligations trivially honoured. Autonomy boundaries: agents answer, users act — every external action requires explicit confirmation. Shadow AI is hypothetically-relevant (community-contribution AI-generation) but not currently active. |
