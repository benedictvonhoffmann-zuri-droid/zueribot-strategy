# The Prototype Bet — Bünzli

> Filled instance of [`prototype-template.md`](prototype-template.md). Built for the Productschool AI Product Strategy course pitch, 2026-04-27.

---

## What I built
A single-page web demo of the Bünzli chat interface, with **five** scripted Swiss-German conversations: the three agentic actions (Züri wie neu civic report, ERZ waste-pickup check, Quandoo restaurant booking), one curated weekend-tips flow, and one **sovereignty + privacy** flow that surfaces where the user's data is stored, who can see it, and how to delete it. A persistent "trust strip" under the header shows the four Bünzli commitments at all times: 🇨🇭 Schwiizer Hosting · 🛡️ Sovereign · 🔒 Verschlüsselet · 🚫 Kei Tracker.

## Tool used
Hand-rolled. Static HTML / CSS / JS, no framework, no build step. Hosted as a static asset under the existing Bünzli landing site (Astro `public/` directory + Caddy + Docker on the Infomaniak VPS in Zürich).

## Prototype link
**https://buenzli.space/prototype-pitch/** *(noindex, nofollow — not for public discovery)*

## What is real / what is faked
**Real.**
- The visual identity (Zürich-blue palette, Inter typography, the Bünzli "B" mark) — matches the production landing site.
- The agentic interaction *pattern*: badged response types, action buttons, confirmation flows, the way the assistant proposes an action and asks for permission before executing.
- The Swiss-German voice — every line was written in the register Bünzli will actually use.
- Hyper-local specificity (Wipkingen, Limmatstrasse, Kronenhalle, Üetliberg).
- The **sovereignty and privacy posture** *as content* — the trust strip and the privacy flow articulate the actual product commitment (Swiss hosting on Infomaniak, no US cloud, no model training on user data, DSG-compliant export and deletion). This is the production stance, surfaced in the demo so reviewers feel it during the pitch instead of being told about it after.

**Faked.**
- All model responses are scripted. There is no LLM call.
- All integrations are scripted. No real Züri wie neu submission, no real Quandoo booking, no real ERZ pickup data.
- The "personal context" (address in Wipkingen, name on the booking) is hardcoded for the demo, not loaded from a real user profile.
- The privacy controls in the demo (the "Mini Date aaluege" and "Alles lösche" buttons) are surfaced visually but do not yet wire to a real export/delete pipeline — that is engineering work for v1.

A skeptic looking at the network tab will see no backend calls. That is intentional — the prototype proves the *concept*, not the engineering.

## AI value archetype
**Orchestrator.** Bünzli's value is not just answering questions (Oracle) or saving time on a single task (Automator) — it is coordinating across multiple Zürich systems (civic, commercial, household) on the user's behalf. The prototype shows this explicitly: each scripted flow ends with the assistant offering to *take an action* against an external system, not just to inform.

## The bet in one sentence
**Zürich residents will trust a Swiss-hosted, Swiss-German AI assistant to take recurring civic and household actions on their behalf — because being narrow, local, and sovereign is exactly what the global chatbots cannot credibly be.**

## What this prototype is meant to prove
1. The agentic value is *legible in 30 seconds* — anyone seeing the demo immediately understands why this is different from "ChatGPT for Switzerland."
2. The Swiss-German voice + hyper-local specificity + sovereignty framing read as a coherent product, not three unrelated features.
3. The pattern of "Bünzli proposes an action → user confirms → action executes" is intuitive and trustworthy *as a UX*, before we have built the real backend integrations.

## What this prototype is *not* meant to prove
- That the integrations are technically feasible (separate engineering work — Züri wie neu and Quandoo APIs are confirmed, ERZ requires scraping).
- That the data flywheel works (correction loop is the next piece, not in this demo).
- That users will pay (a pricing test belongs in Module 3, *The Margin*).
- That the model is good (we are not running an LLM call here — model quality is a Module 4 *The Contract* question).

## How we will measure it
Pitch-room reactions. Specifically:
- **Right signal:** "Oh — it actually *does* the thing for me?" The agentic moment is what the prototype is built around. If reviewers light up on the Züri wie neu flow, the bet reads.
- **Wrong signal:** "Couldn't ChatGPT just do this?" If that question dominates, the prototype has not made the orchestration value legible — the next iteration needs to show something ChatGPT structurally cannot do (e.g. live municipal API call, sovereignty-anchored data residency badge).

## Kill criteria
- The dominant feedback in the pitch session is "this is just a wrapper" → the agentic + sovereign framing is not landing, and the strategy needs a sharper articulation, not a prettier prototype.
- Reviewers cannot articulate, unprompted, *who* this is for after seeing the demo → the user segment is too vague.
- Reviewers ask "why Swiss German specifically?" and the answer requires more than one sentence → the linguistic moat is being over-claimed.

## Next iteration
Wire one real integration end-to-end — most likely Züri wie neu, since the API is open and the civic-action flow is the most distinctive of the three. A single real submission moves the prototype from "concept" to "narrow but working product." That is the artefact for the next pitch, not a polished version of the current scripted demo.
