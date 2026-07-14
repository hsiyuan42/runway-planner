# Runway & Investment Planner

A personal finance tool for people navigating money in a new country — international students, migrant workers, and relocating professionals. It answers two connected questions most tools treat separately:

1. **How long does my money last?** — cash runway modelled against income scenarios, one-off expenses, and hard deadlines like visa expiry.
2. **What can I safely spend or invest?** — a layered money framework (emergency buffer → runway reserve → investable funds) derived from that runway.

**Privacy by design:** all financial data stays in your browser. Nothing is sent to a server.

## Status

🚧 **Phase 0 — Foundations.** This repo currently contains the two original standalone prototypes as the "before" state:

- `cashflow_runway.html` — cash flow & runway dashboard (income scenarios, one-off travel expense, month-by-month table)
- `investing_planner.html` — investment planner (compound growth, runway trade-off, money layers)

These were built for the author's own situation and validated through real use. The project generalises them into one merged, configurable app. See `docs/` for the PRD and project plan.

## Roadmap

| Phase | Goal | Status |
|---|---|---|
| 0 | Repo, shared data model, information architecture | 🔨 in progress |
| 1 | Merge + genericise + deploy static (live URL) | ⬜ |
| 2 | Live currency conversion via serverless API proxy | ⬜ |
| 3 | AI features via Claude API (plain-language summary, NL scenario input) | ⬜ |
| 4 | Published case study | 📝 ongoing log in `CASE-STUDY.md` |

## Disclaimer

This tool is not financial advice. It is for personal, informative use only — figures are illustrative estimates based on historical averages and are not guaranteed.

## Built with

Claude (planning, code, review) + Claude Design (UI) — the development process itself is documented in `CASE-STUDY.md` as part of the project's portfolio goal.
