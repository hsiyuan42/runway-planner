# PRD — Runway & Investment Planner (working title)

**Version:** 0.2 (lightweight) · **Owner:** Hsiyuan · **Date:** July 2026
**Status:** v1 scope decided — ready for Phase 0

---

## 1. Problem Statement

People who move to a new country — international students, migrant workers, relocating professionals — face a uniquely stressful financial question: *"How long can I survive here, and is it safe to spend or invest anything?"* Their situation differs from locals in three ways: income is often temporary or visa-dependent, they lack local financial context (accounts, products, norms), and a hard deadline (visa expiry, course end, contract end) sits underneath every plan.

Existing tools don't serve them. Runway calculators are built for startups; budgeting apps assume stable salaried income; investment calculators ignore that the user might need every pound back in six months. The result is either financial anxiety or false confidence — and no tool answers the connected question: *given my runway, what can I actually afford to spend on life or put into investments?*

## 2. Goals

1. A user can go from landing on the page to seeing their runway in **under 3 minutes**, with no account and no data leaving their browser.
2. The tool answers the *connected* question — runway **and** safe-to-invest/spend amount — through the layered money framework (emergency buffer → runway reserve → investable funds), not two disconnected calculators.
3. Visa/deadline-aware planning: runway is always presented against the user's hard time constraints, not just cash depletion.
4. **Portfolio goal (builder):** the project demonstrates end-to-end AI-assisted product development — shipped app + public case study documenting the Claude workflow.
5. **Learning goal (builder):** the build touches static hosting/DNS, a third-party API behind a serverless function, and an LLM API integration.

## 3. Non-Goals (v1)

- **No accounts, no server-side data storage.** All financial data stays client-side (localStorage). This is a privacy feature and removes auth/database/GDPR scope.
- **No bank connections / open banking.** Manual input only. Aggregation is a different product with heavy compliance burden.
- **No personalised financial advice.** The tool models scenarios; it does not recommend specific products. Illustrative return assumptions carry clear disclaimers.
- **No multi-user / sharing features.** Single-user planning only.
- **No native mobile app.** Responsive web only.

## 4. Personas

| Persona | Situation | Core anxiety |
|---|---|---|
| **The recent graduate** | Finished studies abroad, on a post-study visa, job hunting | "How many months do I have before I'm forced to leave or go home?" |
| **The migrant worker** | Employed but on temporary/renewable contracts, saving actively | "How much can I safely enjoy (travel) or grow (invest) without endangering my base?" |
| **The relocating professional** | Just arrived with savings, new job or job search ahead | "How do I structure my money in a country whose system I don't know yet?" |

## 5. User Stories

Ordered by priority. Grouped by theme.

### Runway (core)
- **US-1.** As a recent graduate on a post-study visa, I want to enter my savings, monthly spend, and income timeline so that I can see exactly how many months of runway I have.
- **US-2.** As a visa holder, I want to enter my visa expiry date so that my runway is calculated against my real deadline — and I can see whether my money or my visa runs out first.
- **US-3.** As a job seeker, I want to model multiple income scenarios (no income / part-time / part-time then full-time / permanent) so that I can see best- and worst-case runway side by side.
- **US-4.** As someone with irregular costs, I want to add one-off expenses (a trip, a visa renewal fee, a deposit) in specific months so that my runway reflects reality, not just an average.

### Spend & invest (the layer framework)
- **US-5.** As a migrant worker, I want the tool to split my savings into emergency buffer, runway reserve, and investable funds so that I know how much I can spend on travel and investing without endangering my base.
- **US-6.** As a cautious first-time investor, I want to see how a monthly investment amount affects both my future portfolio value and my runway so that I can judge the trade-off in days-of-runway, not abstract percentages.
- **US-7.** As someone new to the local system, I want return assumptions and layer sizes explained in plain language so that I understand *why* the tool recommends what it does.

### New-country context
- **US-8.** As someone whose savings are split across countries, I want to switch the display currency and enter amounts in multiple currencies so that I see one consolidated picture. *(v1: manual rate or single API rate; multi-currency inputs may be P1.)*
- **US-9.** As a newcomer, I want sensible default values (typical spend for my situation) so that I get a meaningful first result before I've gathered all my exact numbers.

### AI-powered (Phase 3)
- **US-10.** As a user who doesn't think in spreadsheet terms, I want to describe my situation in plain language ("my contract ends in June, I might get a 3-month gig after, visa expires next April") and have the tool set up the scenario for me.
- **US-11.** As an anxious planner, I want a plain-language summary of my situation ("You're okay until October; your visa is the binding constraint, not your cash") so that I get reassurance and clarity, not just charts.

## 6. Requirements

### P0 — Must have (v1 cannot ship without)
| # | Requirement | Acceptance criteria (abridged) |
|---|---|---|
| R1 | Single merged app with two views (Runway / Invest & Spend) sharing one data model | Changing savings or spend in one view updates the other instantly |
| R2 | Generic onboarding inputs: savings, monthly spend, income scenario(s), income end date, currency | New user reaches a rendered runway chart with ≤ 6 inputs; defaults pre-filled |
| R3 | Visa/deadline field | Runway chart marks the deadline; headline states which constraint binds (cash vs. visa) |
| R4 | Layered framework output, with **adjustable rules** | Tool derives buffer / reserve / investable split from opinionated defaults (buffer = 3 × fixed costs, reserve = 6 months burn); user can adjust both multipliers and the split recalculates live |
| R5 | One-off expense entries (multiple) | User can add ≥ 3 one-off costs, each visible in table + chart, each affecting runway |
| R6 | Client-side persistence | Refreshing the page restores the user's inputs (localStorage); a visible "your data never leaves this browser" note |
| R7 | Deployed on a public URL with custom domain | Live on Vercel or Cloudflare Pages; loads in < 2s on mobile |
| R8 | Disclaimer | Visible statement that the tool is **not financial advice** — for personal, informative use only; shown near results and in the footer; return figures labelled as illustrative historical averages |

### P1 — Nice to have (fast follows)
- Currency conversion via live FX API (serverless proxy) rather than manual rate — *Phase 2 learning milestone*
- Categorised expense input (rent / food / transport / other) feeding the monthly spend figure
- Shareable read-only snapshot (URL-encoded state, still no server storage)
- User-editable portfolio allocation (v1 hardcodes the illustrative 65/30/5 mix)

### P2 — Future considerations (design for, don't build)
- Claude API features (US-10, US-11) — *Phase 3; keep the data model serialisable to JSON so an LLM can read/write it*
- Localisation (zh-TW first) — keep all UI strings in one place from day 1
- Country presets (UK / AU / NZ typical costs, tax-free wrappers like ISA equivalents)

## 7. Success Metrics

Because this is a portfolio + learning project, metrics are honest about that:

**Product signals (lightweight):** time-to-first-runway < 3 min for a test user; ≥ 3 people outside the builder complete onboarding and report the output "useful"; zero data sent to any server (verifiable in devtools — this is a demo talking point).

**Portfolio signals (primary):** live URL + published case study; case study documents ≥ 3 concrete AI-collaboration episodes (prompt → output → correction); project referenced in job applications/interviews.

**Learning signals:** builder can explain, unaided, how DNS resolution, a serverless function, and an API key proxy work for this app.

## 8. Decisions (formerly Open Questions — resolved July 2026)

- **Layer sizing rules: adjustable in v1.** Ship with opinionated defaults (buffer = 3 × fixed costs, reserve = 6 months burn) but let users edit the multipliers. → Promoted into P0 requirement R4.
- **Language strategy: English only for v1.** Localisation (zh-TW first) remains P2. Keep all UI strings centralised in one module from day 1 so this stays cheap later.
- **Return assumptions: hardcoded in v1.** The 65/30/5 illustrative portfolio (~9.7% blended) stays fixed; user-editable allocation moves to P1.
- **Disclaimer: required for launch.** Wording along the lines of *"This tool is not financial advice. It is for personal, informative use only — figures are illustrative estimates based on historical averages and are not guaranteed."* → New P0 requirement R8.

## 9. Timeline & Phasing

No hard external deadline. Phasing follows the learning goals — see the companion **Project Plan** document. Suggested cadence: Phase 1 live within 2–3 weeks of starting, so a portfolio link exists early and everything after improves a shipped thing rather than delaying a launch.
