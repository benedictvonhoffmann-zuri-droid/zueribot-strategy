# Agent Workflow Spec

> **Purpose.** Blueprint a single agent — what it does, for whom, with what tools, and how we know it works. The spec is the bridge from "we should build an agent for X" to a buildable, testable, governable artefact. Without a spec, every agent ends up reinvented at implementation time, governance is bolted on afterwards, and success is judged by vibes.
>
> **How to use it.** Fill the eight core fields first — they are the Productschool template and the *minimum viable spec*. Then fill the supporting sections (Tools, Out of scope, Failure modes, Cost/latency, HITL, Connection to strategy). Treat this as a living document — Pattern and Governance commonly evolve as the design firms up. Re-version on every material change.
>
> **One agent, one spec.** If you find yourself describing two distinct goals, two distinct triggers, or two distinct outputs in one document, split it.

---

## Identity

- **Agent name:**
- **Owner:**
- **Spec version:** *(e.g. v0.1 draft, v1.0 buildable, v1.2 with HITL)*
- **Last updated:**
- **Status:** *(draft / buildable / shipping / deprecated)*

---

## Connection to broader strategy

Optional but recommended — links the agent to the rest of the strategy doc so this spec doesn't drift into a lone artefact.

- **AI value archetype** *(from Module 1)*: Automator / Copilot / Oracle / Creator / **Orchestrator**. Pick one — agents that try to be multiple at once tend to be none.
- **Which moat does this agent reinforce?** *(from Module 2)* Correction loop / Domain-context loop / Workflow embedment / other. Naming this forces the agent to earn its place in the strategy, not just exist as a feature.
- **What's the contract?** *(forward-link to Module 4)* What level of reliability is required (probabilistic-ok vs. must-be-right), and what is the consequence of failure for this specific agent?

---

## The spec table

The eight fields below are the Productschool template — the minimum viable spec. Fill *every* row. "TBD" is acceptable on a v0 draft if you also name the open question.

| Field | What it answers | Spec |
|---|---|---|
| **Business Goal** | What is the specific outcome for the user? Quantified. | Define the value of this agent. State the **baseline today** (current process, time, cost, error rate) and the **target** (specific outcome with units). *e.g. for a Travel Agent: "Reduce time-to-book from 45 mins to 5 mins; baseline = 45m manual flow, target = ≤5m at ≥95% completion."* |
| **Input** | What exactly does the user provide to start the agent? | Define the trigger and its shape. **Format / schema** (text prompt, file upload, API event, scheduled job). **Validation rules** (max size, allowed types, required fields). **What happens if invalid** (reject with message, fall back to default, escalate to human). |
| **Output** | What does the user see at the end? | Describe the final tangible artefact. **Format** (text response, JSON object, email, database row). **Confidence / uncertainty signals surfaced** (e.g. "the agent says it's 80% confident; here's what it's not sure about"). **Where it's delivered** (chat UI, email, file, database write). |
| **Primary Pattern** | Single agent or specialised roles? | Name the agentic pattern + a one-line *why this pattern fits this task*. See the pattern decision rubric below. |
| **Agent Roles** *(optional, for multi-agent)* | Who does what? | For each agent: persona, **main job**, **tools available**, and **what this role does NOT do** (anti-scope). If single-agent: state the one clear role. |
| **Key Steps** | The 3–5 high-level logic steps from input to output. | Map the flow. For each step: what's the input, what tool is called, what's the output, expected duration, what happens on failure. *e.g. "Step 1 (Plan): planner LLM reads input → produces step list. Step 2 (Retrieve): each step's tool is called in parallel. Step 3 (Compose): writer LLM assembles a final answer. Step 4 (Critic): critic LLM checks faithfulness against retrieved sources. Step 5 (Deliver): if critic passes, return; else loop to step 3 with feedback."* |
| **Governance Gates** | The "safety checks" before the answer reaches the user. | See the governance section below — single-line summary in this row, fuller treatment below. |
| **Success Metric** | The one core metric that proves the agent works. | One primary metric + one **counter-metric** (the thing that mustn't go up while the primary goes up — false positives, refusal rate on valid queries, cost-per-call, latency p95). Plus the eval set the metric is measured against. |

---

## Pattern decision rubric

Choosing the Primary Pattern is the most consequential design choice in the spec. It determines complexity, latency, cost, and what kind of failures you'll see. Pick the simplest pattern that meets the goal — over-engineering an agent is the standard mistake.

| Pattern | When to use | When *not* to use |
|---|---|---|
| **Single LLM call** *(no agent loop)* | The task is one prompt → one response. No tools, no multi-turn reasoning. | Anything that requires retrieval, action, or multi-step planning. If you're tempted to call this an agent, it isn't one. |
| **ReAct (single agent)** | Moderate-complexity, well-bounded task. The agent reasons, calls a tool, reasons again, calls another tool, until done. | Long-horizon planning where the plan benefits from being made *before* execution starts. ReAct can drift on long chains. |
| **Planner & Executor** | Multi-step tasks where the plan benefits from one model (often slower / better) and the execution from another (often faster / cheaper). The plan is created once and executed step-by-step. | Tasks where the plan is short and the steps are interdependent — the overhead isn't worth it. |
| **Multi-agent / Hierarchical team** | Tasks with clearly separable concerns (research vs. write vs. critique). Each role has a different prompt, different tools, different evals. | Anything you can do in ReAct. Multi-agent compounds latency, cost, and failure modes — only use it when separation of concerns is *real*. |
| **Pipeline / DAG** | Highly structured tasks with known shape, where the same steps run in the same order every time. The "agent" is mostly orchestration glue around model calls. | Tasks whose shape changes per input. A DAG that branches on every run is a single agent in disguise. |
| **Critic / Reflexion loop** | Tasks where output quality benefits from self-review (writing, code, complex reasoning). | Tasks where the critic doesn't have enough independent signal — e.g. asking the same model to grade itself rarely improves quality. |

> **Default to the simplest.** A ReAct agent with two tools and a tight system prompt outperforms a five-agent hierarchy in 80% of real-world tasks at a tenth of the cost.

---

## Tools available

List every external API, knowledge base, model, or function the agent can call. Be exhaustive — security and cost reviews depend on this list being complete.

| Tool | Type | Used by | Used for | Cost / latency | Failure mode |
|---|---|---|---|---|---|
| | *(API / KB / LLM / function / DB)* | *(which role / step)* | *(what the agent uses it to do)* | *(per-call cost, expected latency)* | *(what happens when this tool fails — retry, fall back, escalate, abort)* |
| | | | | | |
| | | | | | |

---

## Out of scope

Explicitly state what this agent will *not* do. Naming anti-scope upfront prevents the slow drift from "small agent" to "everything agent" — and gives reviewers a way to say "this isn't your agent's job."

- *(Each line: a thing the agent could plausibly be asked to do but won't, and why. e.g. "The agent will not draft new contract clauses — only flag deviations from the standard. Drafting is Legal's job.")*
- 

---

## Failure modes

The 3–5 most likely ways this agent will produce a wrong, harmful, or useless output — and how each is mitigated. A spec without failure modes is optimistic. Real failure modes show up at: input edge cases, tool failures, hallucinations, scope drift, adversarial users.

| Failure mode | Likelihood | Severity | Mitigation |
|---|---|---|---|
| *(e.g. agent hallucinates a clause not in the source PDF)* | *(H/M/L)* | *(H/M/L)* | *(critic agent checks every quoted clause against source text; refuse if not anchored)* |
| | | | |
| | | | |

---

## Governance gates

Where are the safety checks? Most agent specs name only the pre-output check. Production agents need three layers:

1. **Pre-input validation.** Is the input safe / valid to process? *(e.g. file is a PDF and under N MB; user is authenticated; prompt does not contain known injection patterns.)*
2. **Mid-flight checks.** Approval points before destructive or irreversible actions. *(e.g. before sending an email, before submitting a civic report, before writing to a system of record — HITL approval required.)*
3. **Pre-output validation.** The eval / critic / faithfulness check before the user sees the result. *(e.g. critic agent verifies all quoted clauses exist in source text; refusal layer checks for safety violations.)*
4. **Post-output telemetry.** Adversarial probes and red-team cases run continuously against shipped agents. *(e.g. weekly run of 20 known-edge-case inputs; alert if any silently regresses.)*

| Gate | Layer | What it checks | What happens on fail |
|---|---|---|---|
| | *(pre-input / mid-flight / pre-output / post-output)* | | *(reject / retry / HITL / abort)* |
| | | | |

---

## Cost & latency budgets

A production-grade agent has known cost ceilings and latency targets, not "we'll see when it's running."

- **Cost target per successful run:** *(e.g. ≤$0.05; informs model choice and routing)*
- **Cost ceiling per run:** *(hard cap above which the agent aborts and escalates — e.g. $0.50)*
- **p50 latency target:** *(median user wait — e.g. ≤8s)*
- **p95 latency target:** *(tail user wait — e.g. ≤20s)*
- **Latency fallback:** *(what happens when an underlying tool is slow — retry, switch model, return partial, escalate)*

---

## Human-in-the-loop (HITL) points

Where the agent stops and asks a human to confirm before acting. Especially important for agents that *do* things (book, send, submit, write) rather than just *say* things.

| Step | HITL trigger | Who approves | Default if no response |
|---|---|---|---|
| *(e.g. "Before submitting Züri wie neu report")* | *(the step where the agent pauses)* | *(end user / domain reviewer / on-call human)* | *(timeout behaviour — abort, escalate, fall back to drafted-not-sent)* |
| | | | |

---

## Eval set

The dataset against which the Success Metric is measured. Without a defined eval set, "task completion >95%" is a vibe, not a metric.

- **Size:** *(number of cases — start at ≥30, target ≥100 for production)*
- **Composition:** *(% happy-path, % edge cases, % adversarial / red-team)*
- **Source:** *(synthetic, sampled from real users, hand-crafted, mix)*
- **Refresh cadence:** *(monthly / per release / continuous)*
- **Reviewers:** *(how outputs are graded — automated, human, mix)*

---

## Open questions

The questions whose answers are still unknown. Naming them in the spec is much better than papering over with confident prose.

- 
- 
- 

---

## Review log

| Date | Reviewer | Version | What changed |
|---|---|---|---|
| | | | |
