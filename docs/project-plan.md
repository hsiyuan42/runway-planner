# Project Plan — Runway & Investment Planner

Companion to the PRD. Three phases, each ending with something shipped and something learned. Case-study capture points are built in — the write-up is a first-class deliverable, not an afterthought.

**Working rule:** Phase 1 goes live before Phases 2–3 begin. Everything after improves a shipped product.

---

## Phase 0 — Foundations (1–2 evenings)

Decisions and setup before writing product code.

| Step | Task | Output |
|---|---|---|
| 0.1 | ~~Decide the open questions that block build~~ **Done (PRD §8):** English-only v1 (strings centralised for later zh-TW); allocation hardcoded; layer rules adjustable; disclaimer required for launch | Decisions recorded in PRD v0.2 |
| 0.2 | Create a GitHub repository; commit the two existing HTML files as the "before" state | Repo with initial commit — this becomes case-study evidence |
| 0.3 | Sketch the merged information architecture: onboarding → shared data model → View 1 (Runway) / View 2 (Invest & Spend) | One-page IA sketch (Claude Design or paper) |
| 0.4 | Define the data model as a single JSON shape (savings, spend, income scenarios, deadlines, one-off expenses, invest amount, currency) | `state` schema written down — this is the contract everything else builds on |
| 0.5 | Start the case-study log: a running `CASE-STUDY.md` in the repo | Log file with entry #1: the starting point |

**Why 0.4 matters:** a clean serialisable data model is what makes Phase 3 (Claude reads/writes your scenario) cheap later, and localStorage persistence trivial now.

---

## Phase 1 — Merge, genericise, ship static (1–2 weeks)

**Learning goals:** Git-based deployment, DNS, how static hosting works.
**Ship goal:** live public URL.

### 1A. Merge & genericise (the product work)
1. Merge the two HTML files into one app with two views sharing the data model from 0.4. Start from the existing code — it works; refactor, don't rewrite.
2. Strip all personal numbers. Replace with an onboarding flow: currency → savings → monthly spend → income scenario + end date → visa/deadline date. Pre-fill sensible defaults so an empty form still renders something meaningful (US-9).
3. Add the **visa expiry field** (US-2): draw the deadline on the runway chart; compute `effective runway = min(cash runway, months to visa expiry)`; headline states which constraint binds.
4. Generalise one-off expenses (US-4): from the single Scotland-trip control to an add/remove list of `{month, label, amount}` entries.
5. Wire the **layer framework** to the inputs (US-5, R4): buffer = 3 × fixed costs, reserve = 6 months × burn, investable = remainder. Currently these are hardcoded numbers — make them derived, and expose the two multipliers as user-adjustable controls with the defaults pre-set. (The multipliers belong in the state JSON from 0.4.)
6. Persist state to localStorage (R6) + a "your data never leaves this browser" note.
6b. Add the disclaimer (R8): "not financial advice — personal, informative use only" near results and in the footer; label return figures as illustrative. Centralise all UI strings in one module while doing this (keeps future zh-TW localisation cheap).
7. Use Claude Design for the visual pass: consistent look across both views, mobile check. **Case-study capture:** save before/after screenshots and the actual prompts used.

### 1B. Deploy (the learning work)
8. Push to GitHub; connect the repo to **Vercel or Cloudflare Pages**; confirm auto-deploy on push. *Learn: what a build/deploy pipeline is doing.*
9. Buy a domain (~£10/yr) and point it at the deployment. *Learn: DNS records (A/CNAME), propagation, HTTPS certificates.*
10. Small but real: add a favicon, page title, Open Graph tags (how the link previews when shared), and a plausibility check on mobile.

**Phase 1 exit criteria:** live URL on your own domain · a stranger can get their runway in < 3 min · disclaimer visible (R8) · repo README explains the project · case-study log has ≥ 3 entries.

---

## Phase 2 — First real API: live currency (1–2 weeks)

**Learning goals:** calling external APIs, why API keys must be hidden, serverless functions, caching.
**Ship goal:** currency switching with live rates (US-8).

1. Pick a free FX API (e.g., frankfurter.app — no key needed — or exchangerate-api with a key).
2. **Deliberately do it the naive way first:** call the API directly from the browser. If the API needs a key, observe that the key is visible in devtools. *This is the lesson — write it into the case study.*
3. Build a serverless function (Vercel Function / Cloudflare Worker) as a proxy: browser → your function → FX API. Key lives in an environment variable server-side. *Learn: what "backend" minimally means; environment variables; request/response handling.*
4. Add caching (rates change daily — cache for 24h in the function or via headers). *Learn: why you don't hammer third-party APIs.*
5. Product wiring: currency selector; amounts entered in one currency, displayed in another; graceful fallback to a manual rate if the API is down. *Learn: error handling for external dependencies.*
6. Optional stretch: multi-currency inputs ("£7,000 here + NT$150,000 at home") consolidated into the display currency.

**Phase 2 exit criteria:** currency switching works on the live site · no API key visible in browser devtools · you can explain the request path browser→function→API out loud.

---

## Phase 3 — AI feature via Claude API (2–3 weeks)

**Learning goals:** LLM API integration, prompt design for structured output, cost/latency awareness.
**Ship goal:** the feature that makes this a portfolio-grade *AI* demonstration.

1. Choose ONE feature to ship first (both use the same plumbing):
   - **A. Natural-language scenario input** (US-10): user types their situation; Claude returns the JSON state object from step 0.4; the app renders it.
   - **B. Plain-language situation summary** (US-11): app sends the state JSON; Claude returns 3–4 sentences of grounded explanation ("your visa, not your cash, is the binding constraint").
   
   *Recommendation: start with B — output is text (forgiving), input is your own structured data (controlled), and it directly serves the emotional job of the tool. A is the flashier demo; do it second.*
2. Reuse the Phase 2 pattern: browser → serverless function → Claude API. Key in env vars. *The learning compounds — that's why the phases are ordered this way.*
3. Prompt engineering: system prompt that constrains Claude to the provided numbers (no invented advice), with the disclaimer stance baked in. **Case-study capture:** keep prompt versions; show one failure and how you fixed it.
4. Guardrails: rate-limit the endpoint (it's your API bill on a public site), cap input length, handle API errors with a graceful fallback so the core tool never depends on the AI feature.
5. Cost sanity check: estimate tokens/request × plausible traffic; consider Haiku-class model for the summary feature.

**Phase 3 exit criteria:** AI feature live and rate-limited · core calculator fully functional if the AI endpoint is disabled · case study documents the prompt iteration.

---

## Phase 4 — The case study (parallel throughout, finalise ~1 week)

For the portfolio, this artifact matters as much as the app.

1. Structure: Problem → Why I built it → How I worked with AI (the meat) → Architecture (one diagram: static site + two serverless functions) → What I learned → What I'd do differently.
2. The "How I worked with AI" section needs **specifics**: real prompts, real outputs, real corrections (e.g., catching a timeline/calendar bug the AI introduced — concrete episodes like this are exactly what reviewers want).
3. Include the architecture diagram — it doubles as proof of the infrastructure learning goal.
4. Publish: on the portfolio site, or as the project's own `/about` page, plus a shorter LinkedIn-friendly version.

---

## Sequence at a glance

```
Phase 0  Foundations         ▸ repo, data model, IA sketch          (1–2 evenings)
Phase 1  Merge & ship static ▸ LIVE URL, DNS, deploy pipeline       (1–2 wks)
Phase 2  Currency API        ▸ serverless proxy, caching            (1–2 wks)
Phase 3  Claude API feature  ▸ LLM integration, guardrails          (2–3 wks)
Phase 4  Case study          ▸ ongoing log → published write-up     (finalise 1 wk)
```

Roughly 6–8 weeks part-time, with a shippable checkpoint at the end of every phase — if job-search demands interrupt the project, whatever is live still stands on its own in the portfolio.

## Risks & mitigations

- **Scope creep** (country presets, localisation, editable allocations…): parking lot lives in the PRD's P1/P2 lists; nothing enters a phase mid-phase.
- **Rewrite temptation** in 1A: the existing HTML works — refactor incrementally, keep it deployable at every commit.
- **AI endpoint abuse/cost** on a public site: rate limiting + input caps are P0 within Phase 3, not optional.
- **Perfectionism delaying launch:** Phase 1 exit criteria are deliberately modest. Ship, then improve.
