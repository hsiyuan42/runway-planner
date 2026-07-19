# Case Study Log — Building the Runway & Investment Planner with AI

A running log of how this project is built in collaboration with Claude. Raw material for the final portfolio write-up. Each entry: what happened, what the AI did, what I did, what I learned.

---

## Entry 1 — The starting point (July 2026)

**Context.** Two standalone HTML tools existed before this project: a cash flow/runway dashboard and an investment planner, both built with Claude for my own London job-search situation. They went through several rounds of real use and targeted fixes — including a calendar-anchoring bug in the timeline logic and making the portfolio projection independent per scenario — and genuinely improved my sense of financial security.

**Decision.** Turn them into one public tool for people in the same situation: international students, migrant workers, relocating professionals. One app, not two — the layered money framework (buffer → reserve → investable) is the bridge between "how long does my money last" and "what can I safely invest".

**AI collaboration so far.** Claude produced a lightweight PRD and a phased project plan from a rough brief; I made the scope calls (adjustable layer rules in v1, English only, hardcoded illustrative portfolio, disclaimer required). The phasing is deliberately structured around my learning goals: static deploy → serverless API proxy → Claude API integration.

**This commit.** The two original HTML files, unmodified, as the honest "before" state.

## Entry 2 — Designing the data model before touching the code (July 2026)

**What I was trying to do.** Complete Phase 0.3–0.4: the information
architecture of the merged app, and the single state JSON everything
reads from.

**What I asked Claude.** To propose the IA and state shape based on the
PRD and the two prototypes.

**What came back.** Three views (Setup / Runway / Plan) over one shared
state object with localStorage auto-save; a persistent header showing
effective runway and which constraint binds (cash vs visa). The biggest
structural change from the prototypes: replacing my hardcoded income
scenario dropdown with a generic array of income periods, with the old
scenarios becoming editable presets. Also: a schemaVersion field for
future localStorage migrations, and fixed costs as a separate input so
the emergency buffer formula (3 × fixed costs) works for any user.

**What I decided.** Confirmed all three proposals: income modelled as
editable periods with the old scenarios as presets; fixed monthly costs
as a seventh input defaulting to 60% of spend; and the v1 reserve
simplification (reserveMonths × monthlySpend, ignoring expected income
during the search — conservative, refine later).

**What I learned.** The state JSON is a contract, not just storage — the
same object gets persisted to localStorage in Phase 1 and sent to the
Claude API in Phase 3. Designing it once, upfront, is what makes both of
those cheap later.

<!-- Template for future entries:

## Entry N — <title> (<date>)

**What I was trying to do.**
**What I asked Claude / Claude Design.** (paste the actual prompt or a screenshot)
**What came back.**
**What I changed or caught.** (bugs, wrong assumptions, design corrections)
**What I learned.**
-->
