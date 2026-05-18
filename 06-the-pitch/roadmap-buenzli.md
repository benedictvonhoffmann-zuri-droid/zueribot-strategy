# Roadmap — Bünzli

> Filled instance of [`roadmap-template.md`](roadmap-template.md). Snapshot dated 2026-05-18.
>
> **Headline.** Bünzli's H1 is about *shipping the correction loop* (closing the gap M5 flagged: positional advantage, not yet compounding). H2 is about *proving the loop compounds* and *swapping Sonnet judge → Prometheus 2* to close the sovereignty gap M1 named. H3 is about *deciding monetization at open-beta* and *opening the second domain* (Quandoo) — the precondition for Cross-Domain Transfer to become a real loop instead of an aspirational one.

---

## Connection to other modules

- **M1 diagnostic** — Axis 1 (Moat=2) and Axis 2 (Data=2) are the worst axes; H1 + H2 are designed to move both. Axis 3 (Platform=4) is healthy; no H-effort allocated.
- **M2 data-flywheel** — the Correction loop is the one H1 ships; the Domain Context loop is what H3 unlocks via Quandoo.
- **M4 eval-strategy** — every H1/H2 metric below references this doc as source-of-truth.
- **M5 compounding-system** — the freeze-test verdict ("positional, not yet compounding") is what this roadmap answers. H1 + H2 turn position into compounding.

---

## H1 — Ship (0–4 weeks)

| Initiative | Metric (what proves it shipped) | Owner | Confidence |
|---|---|---|---|
| **"War das richtig?" correction primitive live in chat** | Thumb + optional text widget renders on 100% of answers; flags persist to `flags.jsonl` | Benedict + Claude | H |
| **Property-based eval suite running on CI** | 50 golden rows + 6 property tests run on every push; results posted to PR | Benedict | H |
| **Confidence-UX rendering ("Bünzli sagt / gloubt / isch sich nöd sicher")** | Every answer shows a confidence chip in Züritüütsch; tested across 3 query types | Benedict | H |
| **Open Strategy publication on landing page** | M1–M6 curated extracts live at `/strategie`; linked from footer | Benedict | H |

---

## H2 — Validate (1–3 months)

| Initiative | Hypothesis being tested | Kill criterion | Owner | Confidence |
|---|---|---|---|---|
| **Recursive Learning compounds** | ≥200 valid flags → week-over-week eval-set quality on flagged patterns improves ≥5% | After 12 weeks live, no measurable WoW improvement → loop is broken, revisit primitive design | Benedict | M |
| **Prometheus 2 swap for LLM-as-judge** | Self-hosted Prometheus 2 on rent-a-pod matches Sonnet 4.7 agreement-with-human ≥85% | Agreement <75% after 4 weeks tuning → defer swap, keep Sonnet, name as residual sovereignty gap | Benedict | M |
| **First 100 retained users (≥3 sessions / 30 days)** | Hyper-local civic queries are sticky enough that locals return without prompting | <40 retained users at 90 days → core loop is too weak, re-scope before H3 monetization | Benedict | M |
| **Quandoo integration scoped + dry-run pattern proven** | Restaurant booking agentic action works end-to-end in dry-run; HITL confirmation flow tested | Cannot get sandbox access OR HITL flow has >10% override rate → de-prioritize, pick different second domain | Benedict | M |

---

## H3 — Explore (3–6 months)

| Initiative | Question being answered | Cost to run | Owner | Confidence |
|---|---|---|---|---|
| **B2G procurement conversation with Stadt Zürich** | Is the city interested in Bünzli as official civic assistant, or as supplier of civic-AI primitives? | ~10 hours of meetings; one written proposal | Benedict | L |
| **Cross-Domain Transfer measurable (civic ↔ commercial)** | Does a correction filed on civic queries measurably improve commercial answers (or vice versa)? | Instrumented as part of Quandoo build; ~1 week of analysis | Benedict | L |
| **Monetization decision at open beta** | Which of the three monetization paths (Tagesticket subscription / pay-per-action / B2G licensing) is the primary at v1.5? | One pricing experiment with first 100 users; cost-curve refresh | Benedict | L |
| **Legal KB collection as separate product surface** | Is sovereign Swiss-legal-corpus retrieval a distinct product (B2B / law-firm) or a Bünzli feature? | One discovery week + 3 expert conversations | Benedict | L |

---

## AI-specific board metrics — Bünzli targets

| Metric | M1 target (now) | M3 target | M6 target | Source |
|---|---|---|---|---|
| **Hallucination Rate** | ≤8% on golden set | ≤5% | ≤3% | M4 eval-strategy §reliability contract |
| **Drift Velocity** | Baseline-setting (first measurement) | <2% WoW regression on golden set | <1% WoW | M4 |
| **Confidence Distribution** | ~60% high / 25% medium / 15% refuse on real traffic | 70/20/10 with Hallucination flat-or-down | 75/15/10 calibrated | M4 |
| **HITL Rate** | 100% on agentic actions (per M5 governance) | 100% (unchanged — by policy) | 100% with <5% user-override rate | M5 governance-policy |
| **Inference ROI** | Not applicable — no revenue at v1 | Cost per active user / month <CHF 0.50 | Revenue-per-CHF >1.0 (break-even on inference) | M3 cost-curve |
| **Eval Regression** | <2 regressions per release | 0 silent regressions; all regressions caught in CI | 0 regressions reach production | M4 |

**Reporting:** monthly to Benedict (sole stakeholder at v1); shared publicly via Open Strategy page once a quarter (committed transparency posture).

---

## Open questions

1. **Quandoo sandbox access** — can we get it cleanly, or does H2 need a different second-domain candidate (Migros loyalty? SBB? VBZ?)?
2. **Prometheus 2 deployment shape** — rent-a-pod vs Infomaniak GPU vs federated EU compute? Decision lives in H2 but cost-curve assumptions need refreshing.
3. **What is the right "retention" definition for a civic assistant?** 3 sessions / 30 days might be wrong — locals might use Bünzli twice a year for big questions (Mieterhöhung, Steuererklärung) and that's healthy. Need to interrogate before H2 kill-criterion fires.

---

## Review log

| Date | Reviewer | What changed |
|---|---|---|
| 2026-05-18 | Benedict + Claude (Opus 4.7) | Initial fill. H1 = correction loop + property eval + confidence-UX + Open Strategy publish. H2 = prove compounding, swap to Prometheus 2, first 100 retained, Quandoo. H3 = B2G with Stadt Zürich, Cross-Domain Transfer, monetization at open beta, legal-KB as own product. Six AI board metrics targeted M1/M3/M6. |
