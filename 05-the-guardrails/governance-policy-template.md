# Governance Policy

> **Purpose.** Codify *how* we operate AI responsibly day-to-day: what's in scope, what the agent can do on its own, when humans must enter the loop, how often we audit, what regulations apply. The output is a policy a non-technical stakeholder (regulator, executive, lawyer) can read and verify against actual product behaviour.
>
> **The structural insight this doc serves.** Governance written after a problem ships is reputation management. Governance written before a problem ships is product. Most AI products skip this doc because *responsible AI* gets framed as something separate from product — it isn't. The autonomy boundaries you set, the escalation triggers you wire, the audit cadence you commit to: all of these are product decisions.
>
> **How to use it.** Fill in honestly with what you actually do, not what you wish you did. An empty cell is honest; a fake commitment you can't operationally honour is dangerous. Re-review quarterly.

---

## Connection to other modules

- **`compounding-system.md`** — the operational sibling. Compounding is "is the product getting better"; governance is "is the product being run responsibly."
- **`04-the-contract/eval-strategy.md`** — the audit cadence and escalation triggers here should match the eval-strategy reliability contract. If they don't, one of them is wrong.
- **`07-the-agent/agent-workflow-spec.md`** — the agent topology section here is the governance lens on what the AW Spec already specified. Same agents; this doc asks "who approved what they're allowed to do?"
- **`01-the-bet/diagnostic.md` — Axis 3 (Platform exposure)** — regulatory exposure influences platform-exposure risk. If EU AI Act classifies you as high-risk, your platform exposure goes up.

---

## Scope

*What this governance policy covers.* Be specific. *"Our AI product"* is too vague; *"the Bünzli civic assistant (web + future mobile), including all agentic actions on user behalf and all KB content surfaced to users"* is right.

- **Products / surfaces:**
- **User-facing AI behaviour included:**
- **Internal-facing AI use (e.g. team uses of LLMs for content generation, code, support):**
- **What this policy explicitly does NOT cover:**

---

## Autonomy boundaries

What the agent is allowed to do without human approval, and what requires human-in-the-loop. Anchored rubric: be explicit per action class.

| Action class | OK solo | Needs human |
|---|---|---|
| **Information retrieval** | ✓ | |
| **Composition / generation** *(natural-language answers)* | ✓ | |
| **Confidence judgement / refusal** | ✓ | |
| **Search across user's own data** *(scoped, no cross-user)* | ✓ | |
| **Read external systems on user's behalf** *(scheduled lookup, public data)* | ✓ | |
| **Write to external systems on user's behalf** *(submit form, book service, send email)* | | ✓ — explicit user confirmation per action |
| **Modify user's own data** *(saved preferences, history)* | varies | varies |
| **Affect other users / cross-user actions** | | ✓ — explicit policy approval required |
| **Make decisions about people** *(eligibility, scoring, classification of persons)* | | ✓ — domain-expert review + regulatory compliance |
| **Refusal of safety-critical scope** *(legal advice, medical referral, financial recommendation)* | ✓ — autonomous refusal | |

**The principle:** the agent acts solo on *reading and responding*; humans approve *writing and deciding*. Asymmetric — false refusals are tolerable; false confident actions are not.

---

## Escalation triggers

*When humans must enter the loop.* For each trigger, name the action.

| Trigger | What happens |
|---|---|
| Agent confidence below threshold | Refuse / route to "go ask authority directly" |
| Out-of-scope query (per Scope Classifier) | Refuse politely with route to appropriate resource |
| User flags an answer as wrong | Routed to eval-pipeline (per M4 closed loop) |
| Agentic action proposed | User confirms before execution |
| Critic / verifier fails twice | Escalate to "send to Legal" / appropriate authority |
| Production telemetry anomaly | On-call PM investigates |
| Adversarial pattern detected | Security review of attack + response |
| Cost ceiling exceeded | Abort + engineering alert |

---

## Audit cadence

How often we look at the system's behaviour with critical eyes.

| Audit type | Frequency | Owner | Output |
|---|---|---|---|
| **Eval suite (full)** | Per release + monthly | PM + Eng | Pass/fail vs. reliability contract |
| **Red-team / adversarial** | Monthly | PM (with rotating security reviewer) | New adversarial rows added to eval set |
| **Production telemetry review** | Weekly | PM | Trends + flagged anomalies |
| **Sample-rate output review** *(5% of production answers)* | Weekly | Domain expert reviewer | Drift signal + calibration data for LLM-as-judge |
| **Privacy / data-handling review** | Quarterly | Privacy Officer (or designated PM at small scale) | Confirmation no PII drift; data-processing register up to date |
| **Regulatory exposure review** | Quarterly + on any law change | Legal (or designated PM) | Risk-tier assessment unchanged or updated |
| **Full governance review** | Annually | Cross-functional | This doc updated; commitments re-affirmed |

---

## Regulatory exposure

### EU AI Act risk classification

The EU AI Act classifies AI systems into four risk tiers. Classify your product honestly.

| Tier | Definition | Examples | Obligations |
|---|---|---|---|
| **Unacceptable** | Social scoring, real-time biometric ID in public, manipulation, exploitation of vulnerable groups | Banned outright | Don't ship |
| **High-risk** | AI making decisions about people (credit, employment, education, healthcare, law enforcement) + safety-critical AI (medical devices, infrastructure, etc.) | Resume screening, credit scoring, medical diagnosis assist | Conformity assessment, registration, robust eval, transparency obligations, post-market monitoring |
| **Limited-risk** | Generative AI, chatbots, AI-generated content surfaced to consumers | Most consumer AI products | Article 50 transparency (label AI-generated content), basic disclosure |
| **Minimal** | Spam filters, video game AI, recommendation systems with no behavioural manipulation | | None specific (general AI Act provisions apply) |

**Classification for this product:**
- **Tier:** *(unacceptable / high-risk / limited / minimal)*
- **Rationale:** *(why this tier; what would push it up a tier)*
- **Article 50 transparency status:** *(complied / in progress / not applicable)*

### Data protection

- **Applicable regime:** *(GDPR / revDSG / sector-specific / multiple)*
- **Data-processing register status:** *(maintained / in progress / not applicable)*
- **Data-subject obligations:** *(access / rectification / deletion / portability — how each is honoured)*
- **PII storage policy:** *(none / scoped / full — consistent with privacy posture)*
- **Cross-border data flows:** *(none / EU-only / global with adequacy decisions)*

### Sector-specific regulation

- **Sector(s) applicable:** *(healthcare / financial / legal / civic / education / none)*
- **Specific obligations:** *(named regulations + compliance status)*

---

## Agent topology — governance lens

*Who approved what each agent is allowed to do?* Cross-reference to the AW Spec (M7).

| Agent role *(from AW Spec)* | What it CAN do | What it CAN'T do | Who approved this scope |
|---|---|---|---|
| | | | |

Common patterns to make explicit:
- No agent reads or writes another user's data
- No agent executes destructive actions without explicit user confirmation
- No agent modifies its own prompts or routing rules at runtime
- No agent calls external APIs without rate-limit + cost-cap
- No agent makes decisions about people (per Autonomy Boundaries)

---

## Shadow AI audit

*Rogue or unauthorized AI tools in use inside the organisation or product surface.* Relevant when: teams are using AI tools without governance (e.g. ChatGPT for confidential work), or when third parties are using AI on your platform without disclosure.

| Tool / pattern | Owner | Risk level | Decision |
|---|---|---|---|
| | | H / M / L | keep / govern / kill |
| | | H / M / L | keep / govern / kill |

**Summary:**
- **Total tools / patterns identified:**
- **Tools after triage:**
- **Estimated hidden spend:** *(if applicable)*
- **Detection mechanism:** *(self-disclosure / network audit / spot-check / not applicable)*

For small teams or pre-launch products, this section may be N/A or hypothetical — but worth thinking through anyway because the issue arrives at scale fast.

---

## Open questions

- 
- 

---

## Review log

| Date | Reviewer | What changed |
|---|---|---|
| | | |
