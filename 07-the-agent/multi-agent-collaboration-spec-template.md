# Multi-Agent Collaboration Spec (MACSpec)

> **Purpose.** The Agent Workflow Spec defines what each agent does individually. The MAC Spec defines how they coordinate. Without it, the protocol layer between agents — handoff schemas, memory rules, conflict resolution, termination — gets reinvented at implementation time, governance is bolted on afterwards, and "the agents collaborate" is a vibe rather than a contract.
>
> **One multi-agent system, one MAC Spec.** If you have several agents that don't actually talk to each other, you have several single-agent systems and you need several AW Specs, not a MAC Spec.
>
> **How to use it.** Fill the seven core fields first — they are the Productschool template and the *minimum viable coordination contract*. Then fill the supporting sections (Memory, Termination, Conflict resolution, Observability). Treat as a living document — these contracts harden as the system gets built.

---

## Identity

- **System name:**
- **Owner:**
- **Spec version:**
- **Last updated:**
- **Status:** *(draft / buildable / shipping / deprecated)*
- **Related AW Specs:** *(link to each agent's individual AW Spec, or to the section of a combined spec where each role is defined)*

---

## The spec table

The seven fields below are the Productschool MAC Spec — the minimum viable coordination contract. Fill *every* row. "TBD" is acceptable on a v0 draft if the open question is named in the *Open questions* section.

| Field | What it answers | Spec |
|---|---|---|
| **Agents involved** | Which agent roles are in this system, and what is each one's single core purpose? | Name each agent + one-line purpose. Refer to the AW Spec for full role definition. *e.g. "Researcher: scours the web for data. Writer: drafts the email. Reviewer: checks tone."* |
| **Orchestration type** | What collaboration pattern do the agents use? | Pick one (rubric below). State the chosen pattern + a one-line *why this pattern fits this system*. |
| **Data format** | What data structure carries information between agents? | Define the handoff schemas. Each handoff between agents has a contract — what fields are required, what's optional, what's the type. *e.g. "List of {source_url, snippet, summary} objects, ≥3 items."* |
| **Communication channel** | What is the technical mechanism for sharing data? | Direct output→input, shared state / blackboard, message queue, event bus, tool-use-as-comms. State *where* and *how* data flows physically (LangFlow connection, in-memory dict, Redis, file). |
| **Trigger logic** | What rule fires the next agent? | Pick a trigger pattern (rubric below). Different agents in the same system can use different triggers. |
| **Fallback logic** | What recovery action runs when an agent fails or produces low-quality output? | Pick a fallback pattern per failing agent (rubric below). System-level fallback — if the whole pipeline cannot recover — also named. |
| **Governance hooks** | Where are the automated eval gates and human checkpoints? | Reference the gate structure from each agent's AW Spec; *here* state the gates that span more than one agent (handoff-boundary checks, cross-agent validation, system-level HITL). |

---

## Orchestration type — pattern rubric

Choosing the orchestration pattern is the most consequential design choice. It determines latency, cost, debuggability, and the failure modes you'll see. Pick the simplest pattern that meets the goal.

| Pattern | When to use | When *not* to use |
|---|---|---|
| **Sequential pipeline** *(A → B → C)* | Linear flow, each step depends on the previous, clear ordering. | When steps are independent — the pipeline serialises work that could parallelise. |
| **Parallel + aggregator** *(A + B + C → Aggregator)* | Independent subtasks that can run concurrently; an aggregator merges results. | When subtasks are interdependent or vary wildly in latency — fastest only as the slowest leg. |
| **Hierarchical / orchestrator** *(Manager → Specialists)* | Complex tasks needing dynamic delegation; the manager decides which specialist to call based on input. | When the routing is deterministic — a sequential pipeline is simpler and faster. |
| **Blackboard / shared scratch** *(agents read/write to common state)* | Agents need to see each other's work product, not just final output; iterative refinement. | When ordering matters strictly — blackboard models tolerate concurrency that may not be safe. |
| **Voting / ensemble** *(N agents → consensus)* | High-stakes outputs where independent verification matters more than speed; open-ended creative tasks where diversity helps. | Throwaway costs — running N agents for one output is N× the cost. Only use when the stakes justify it. |
| **Critic / reflexion loop** *(generator ↔ critic, iterate)* | Output quality benefits from review-and-revise; especially when ground truth is hard but checking is easy (e.g. citation grounding). | When the critic shares the generator's blind spots — a critic from the same model family rarely catches what the generator missed. |

> **Default to the simplest pattern.** Three agents in a sequential pipeline outperform six agents in a hierarchy in 80% of real-world cases at a fraction of the cost.

---

## Trigger logic — pattern rubric

Different agents in the same system can use different trigger patterns. Name the trigger per agent.

| Trigger | When the next agent fires |
|---|---|
| **Sequential complete** | The previous agent finishes successfully (most common). |
| **Conditional output** | The previous agent's output meets a specific condition (e.g. critic returns "pass"). |
| **Time-based** | A schedule fires the agent (cron, interval). |
| **Event-driven** | An external event fires the agent (webhook, queue message, file upload). |
| **HITL approval** | A human approves before the next agent runs. |
| **Confidence threshold** | The previous agent's confidence is above (or below) a threshold (e.g. "if confidence <80%, skip the answer agent and escalate"). |

---

## Fallback logic — pattern rubric

Per failing agent, pick the recovery pattern. System-level fallback (when the whole pipeline cannot recover) is named separately at the bottom.

| Fallback | What happens when the agent fails or produces low-quality output |
|---|---|
| **Retry — same agent** | Run again with same model. Use a max-retry count to avoid loops. |
| **Retry — different model** | Run again with an alternate model (e.g. fall back from Sonnet to GPT-4 Turbo). |
| **Skip — degraded output** | Skip this agent; downstream agents handle the missing piece (e.g. "no critic verification, mark output as 'unverified'"). |
| **HITL escalate** | Hand off to a human with context. The agent stops; the human decides. |
| **Abort workflow** | Cannot recover; terminate the whole run with an error. |
| **Fall back to simpler agent** | Replace a complex multi-agent path with a single-agent ReAct fallback. Useful when the critic / planner is unavailable but a degraded answer is better than no answer. |

---

## Memory architecture

> **Why this section exists.** The PDS template treats memory implicitly. In practice, *the strategic choice of what to remember and what to forget* often determines more about whether a multi-agent system works than the orchestration pattern does. Memory choices also have direct privacy and security implications — they cannot be ignored at v1 and bolted on at v2.

### Memory taxonomy

Multi-agent systems use up to five kinds of memory. Most production systems need at least three.

| Type | What it is | Lifetime | Typical implementation |
|---|---|---|---|
| **Working memory** | The current LLM call's context window — the prompt and the immediate inputs. | Single LLM call | The prompt itself. |
| **Shared scratch / blackboard** | State written and read by multiple agents *within a single run*. | One full run | In-memory dict, LangFlow state, Redis hash. |
| **Session memory** | State that survives across turns *within one user session* (a multi-turn conversation, an iterative refinement). | One session | Conversation history, scoped per session ID. |
| **Persistent / long-term** | State that survives across sessions, scoped per user, per workspace, or globally. | Indefinite (until pruned) | Database, vector store, user profile. |
| **Episodic / semantic KB** | Indexed memory of facts, prior cases, or "what we've seen before" — usually retrievable, not loaded wholesale. | Indefinite | Vector DB, retrieval over historical runs, KB. |

### Strategic choices

| Choice | Trade-off |
|---|---|
| **Per-agent vs. shared memory** | Per-agent isolates concerns and reduces cross-agent contamination, but blocks agents from learning from each other's work product. Shared scratch enables iterative collaboration but introduces ordering and overwrite risks. |
| **Persistence** | Persistent memory creates lock-in (good for switching cost moats) and personalisation, but raises privacy bar and requires GDPR-grade data handling. Some systems are *deliberately stateless* per session as a privacy posture. |
| **Read / write permissions** | Not every agent should write to every memory store. A naive "everyone can write" pattern produces silent corruption. Define per-store read/write per agent. |
| **Pruning / summarisation** | Memory grows. Without a pruning rule (FIFO, summarisation, importance-weighted decay), context windows fill, costs rise, and old data drowns relevant signal. |
| **Privacy posture** | Two questions per memory store: *whose data is it* (user-specific, system-shared, aggregate) and *can the user delete it* (must be yes for personal data under GDPR / DSG). |

### Memory architecture table

Fill one row per memory store in the system.

| Memory store | Type | Lifetime | Who writes | Who reads | Pruning rule | Privacy posture |
|---|---|---|---|---|---|---|
| | *(working / shared / session / persistent / episodic)* | *(call / run / session / forever)* | *(which agent(s))* | *(which agent(s))* | *(FIFO / size cap / summary / importance-weighted)* | *(stateless / per-session / per-user / aggregate; export+delete supported?)* |
| | | | | | | |
| | | | | | | |

### The "what we deliberately don't remember" list

Equally important — name the data that the system *will not* store, and why. Stateless-by-design is a feature for some products. *e.g. "We do not store user-uploaded PDFs after the session ends — privacy posture."*

- 
- 

---

## Termination conditions

When does the workflow stop? A multi-agent system without explicit termination conditions runs forever in pathological inputs. Name them upfront.

| Condition | Trigger | Result |
|---|---|---|
| **Success** | Final agent's output passes governance gates | Return result to user |
| **Hard failure** | Unrecoverable error in any agent | Abort + log + (optionally) escalate to human |
| **Max-loop** | Critic-reflexion loop exceeds N iterations | Return current best output with explicit "not fully verified" flag, or escalate |
| **Time budget exhausted** | Run exceeds wall-clock budget | Return partial output or abort |
| **Cost budget exhausted** | Run exceeds cost ceiling | Abort + alert |
| **HITL refusal** | A human in the loop rejects | Stop and route per the human's instruction |

---

## Conflict resolution

Whenever the system has a critic, a reviewer, a voting ensemble, or any pattern with potential disagreement: name how conflicts are resolved. The default *"the last agent wins"* is rarely the right answer.

| Conflict scenario | Resolution rule |
|---|---|
| *(e.g. "Critic disagrees with Answer Agent")* | *(re-run answer with critic feedback once; if still disagree, escalate to human)* |
| *(e.g. "Two agents in a voting ensemble produce contradictory outputs")* | *(third agent breaks the tie / take the most-cited / abort and escalate)* |
| *(e.g. "Two agents try to write to the same memory key")* | *(last write wins / first write wins / explicit lock / version conflict triggers reconciliation)* |
| | |

---

## Observability / tracing

In a multi-agent system, when something goes wrong you need to know *which* agent broke, *on what input*, and *with what state*. Without tracing, debugging a 5-agent failure is archaeology.

- **What is logged per agent run:** *(input, output, model used, tokens, cost, latency, errors)*
- **What is logged per handoff:** *(from-agent, to-agent, schema, payload size, validation result)*
- **What is logged per memory write:** *(store, key, agent, timestamp)*
- **Where the logs live:** *(LangFlow built-in tracing, LangSmith, Helicone, custom OTel, etc.)*
- **Retention:** *(how long traces are kept; PII handling)*
- **On-call alerting:** *(what conditions trigger a page — e.g. failure rate >5% over 1h, p95 latency >2× target)*

---

## Cost & latency budget — system level

Each agent has its own per-call budget in its AW Spec. The MAC Spec adds the *coordination overhead*: serial latency in pipelines, fan-out cost in parallel patterns, retry costs in critic loops.

- **Total cost target per successful run:** *(sum of per-agent budgets + coordination overhead)*
- **Total cost ceiling per run:** *(hard cap; aborts when exceeded)*
- **End-to-end p50 latency target:** *(user-facing wait — sum of serial steps; min of parallel branches)*
- **End-to-end p95 latency target:**
- **Latency degradation policy:** *(if any agent breaches its individual budget, do we abort, fall back, or accept? Defines the failure-mode under load.)*

---

## Open questions

- 
- 
- 

---

## Review log

| Date | Reviewer | Version | What changed |
|---|---|---|---|
| | | | |
