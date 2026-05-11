# Compounding System — Bünzli

> Filled instance of [`compounding-system-template.md`](compounding-system-template.md). Snapshot dated 2026-05-02.
>
> **Headline.** At v1, Bünzli's three compounding loops are **all aspirational** — one is broken-by-omission (Recursive Learning needs the M4 correction primitive shipped), one is missing (Cross-Domain Transfer requires multi-domain operation that doesn't exist yet at v1), one is structurally capped by product shape (Network Intelligence). The freeze test reveals: *Bünzli would probably still be winning if frozen for 12 months — but for the wrong reason. The advantage at v1 is positional (sovereignty + hyper-local + civic-AI category being slow-moving), not compounding.* That is a strategic finding worth naming honestly, not papering over.

---

## Connection to other modules

- **`02-the-moat/data-flywheel-buenzli.md`** — designed four flywheel loops (Correction, Preference, Domain context, Network). At v1, the scoring was 4/20 (cold start) → projected 14/20 at M24. This M5 doc is the *check* on whether those scores are tracking. They're not at v1; that's the point.
- **`04-the-contract/eval-strategy-buenzli.md`** — the closed-loop "user flag → LLM-as-judge → eval set addition" is what powers the Recursive Learning loop. Without M4 shipping, Recursive Learning stays missing.
- **`01-the-bet/diagnostic-buenzli.md` — Axis 1 (Moat = 2) and Axis 2 (Data = 2)** — both scores are aspirational pending the compounding loops actually running. At the next quarterly review, both axes will be re-scored against whether these loops moved from missing/broken → active.

---

## The three loops — mapped to M2's framing

| M5 framing (this doc) | M2 flywheel framing |
|---|---|
| Recursive Learning | Correction loop |
| Cross-Domain Transfer | Domain Context loop |
| Network Intelligence | Network loop |
| *(no M5 equivalent)* | Preference loop |

The M2 Preference loop has no M5 equivalent because Bünzli is structurally capped on Preference at score 3 (no per-user behavioural inference by values choice). Not relevant to "compounding" framing — same finding documented here.

---

## Loop status — Bünzli at v1

| Loop | Input | Output | Compounds? | Status | Evidence |
|---|---|---|---|---|---|
| **Recursive Learning** | "War das richtig?" user flag → LLM-as-judge → eval-set addition (per M4) | Per-query eval quality improves week-over-week | N at v1 | **MISSING** | Correction primitive not shipped at v1 launch; ETA Q3 |
| **Cross-Domain Transfer** | Signal from civic queries informing commercial queries (and vice-versa) | Quality on domain B improves when only domain A receives correction | N at v1 | **MISSING** | v1 is single-domain (Zürich civic only); commercial integrations (Quandoo, Migros) are v2 roadmap |
| **Network Intelligence** | More users → richer Quartier-Wissen → better answers for everyone | The 1,001st user makes the product measurably better for users 1-1,000 | Partial, structurally capped | **CAPPED at level 3 (indirect benefit via community contributions)** | Bünzli is not a social product; structural cap documented in M2 at score 3/5 |

**The honest read.** All three are aspirational at v1. Recursive Learning is the highest-leverage to ship; Cross-Domain Transfer arrives with v2 commercial integrations; Network Intelligence is permanently capped (and that's a values choice, not a roadmap issue).

---

## Broken loop diagnosis

**Loop: Recursive Learning** (currently missing)

- **Hypothesis on why it's missing**: the "war das richtig?" UI primitive + the closed-loop processing pipeline (M4) weren't built into v1 scope. The eval strategy designed it; the v1 product doesn't yet ship the entry point.
- **Fix plan**: ship the "war das richtig?" thumbs + optional correction text widget at the end of every Bünzli response. Wire the flag → LLM-as-judge processing per the M4 eval strategy. Target: end of v1 Q3 (within 6 months of launch).
- **Re-test criterion**: after 3 months of the flag being live, demonstrate that eval-set rows added from flags have improved per-query quality on those exact patterns by ≥5% week-over-week.

**Loop: Cross-Domain Transfer** (currently missing)

- **Hypothesis on why it's missing**: v1 ships civic queries only. Without a second domain to transfer to/from, the loop literally cannot run.
- **Fix plan**: v2 introduces commercial integrations (Quandoo booking, Migros loyalty if it lands). Cross-domain telemetry instrumented as part of that build.
- **Re-test criterion**: at v2 + 6 months, demonstrate that a correction filed on civic queries measurably improves commercial-query answer quality (or vice versa).

**Loop: Network Intelligence** (structurally capped at level 3)

- Not a "broken" loop in the M2 sense — it's a *deliberately limited* one. Documented in M2 as cap-by-design (not a social product, privacy posture forbids inter-user value transfer).
- The level-3 ceiling is "indirect benefit via community contributions" (more contributors → richer Quartier-Wissen → better answers). That's the best Bünzli can do without changing product shape.
- **No fix planned.** Acceptance, not deficit.

---

## Context connectivity — where Bünzli's knowledge silos

| Silo | Where it lives | Why it matters | Plan to bridge |
|---|---|---|---|
| User-flagged corrections | Will live in eval pipeline once M4 ships | Without the bridge to the KB, the same flag fires repeatedly without the underlying knowledge updating | Wire validated flags → KB update via PR-style review at v2 |
| Quartier-Wissen contributions | Planned for v1+; UI primitive separate from chat | Without indexing into the same retrieval surface as the curated KB, contributions are invisible to the Researcher | Contributions index into the same Qdrant collection with a `source: community` tag at v1.5 |
| Civic outcome data (Züri wie neu resolution times, ERZ adherence) | Public information from Stadt Zürich open-data portal | Bünzli currently doesn't ingest this; ingesting it would let the Critic verify civic-action confidence | Open-data ingestion job at v2 |
| Eval-set rows | Will live in `~/zuribot/evals/golden.jsonl` per M4 | Without versioning + change-review, the dataset drifts silently | Locked to repo at v1; PR review on every change |

---

## The freeze test — Bünzli's answer

*If we froze Bünzli for 12 months — no shipping, no curation, no community contributions — would we still be winning?*

**Honest answer: probably yes, but for the wrong reason.** Bünzli's defensibility at v1 is **positional** (sovereignty + hyper-local civic specialisation + the slow-moving civic-AI category), not **compounding** (none of the three loops are running yet). The freeze test reveals that we're not currently building a compounding advantage; we're occupying a defensible position.

That is **fine for now** but it has a shelf life. Per M1's diagnostic synthesis:

> *"The bet, in one sentence: Hyperscalers will not bother with Swiss-German Zürich for the next 24 months. We will use that window to convert positional advantage (Axis 3) into earned advantage (Axis 1 + Axis 2)."*

The "earned advantage" *is* the compounding loops. Until they're active, the 24-month bet is decaying — slowly, but decaying. The Recursive Learning loop shipping in Q3 is the most important single milestone for converting position into compounding.

**Strategic implication:** the next quarterly review should explicitly re-score Axis 1 and Axis 2 of the M1 diagnostic *downward* if the Recursive Learning loop hasn't shipped. Don't paper over.

---

## Open questions

1. **When exactly does the "war das richtig?" primitive ship?** Currently named as "Q3"; needs a concrete sprint commitment.
2. **What's the threshold of accumulated user flags before Recursive Learning starts producing visible compounding?** Estimate: ~200 valid flags before week-over-week improvement is measurable above noise.
3. **Can civic outcome data (Züri wie neu resolution telemetry) be ingested via official open-data API, or does it require a partnership conversation?** Worth checking before v2 planning.
4. **Should Bünzli accept the Network Intelligence cap permanently, or revisit if a low-friction user-to-user value pattern emerges?** (e.g. "your neighbour's tip on this restaurant" — would require explicit opt-in and privacy treatment.)

---

## Review log

| Date | Reviewer | What changed |
|---|---|---|
| 2026-05-02 | Benedict + Claude (Opus 4.7) | Initial fill. All three loops marked missing or capped at v1. Recursive Learning identified as highest-leverage to ship. Freeze test answered honestly: positional advantage at v1, not compounding. Strategic note: M1 diagnostic Axes 1+2 should be re-scored downward at next quarterly review if Recursive Learning hasn't shipped. |
