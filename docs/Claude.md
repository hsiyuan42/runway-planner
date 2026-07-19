# Project context for Claude Code

Read before any task:
- docs/PRD.md — scope, requirements (P0/P1/P2), decisions in §8
- docs/architecture.md — the IA and state JSON contract. This is
  binding: all code reads/writes the state shape defined there.
- docs/project-plan.md — current phase and step

## Rules
- We are in Phase 1A. Do not build P1/P2 features (currency API,
  AI features) even if convenient.
- Single HTML file (index.html), no framework, no build step.
- Never modify cashflow_runway.html or investing_planner.html —
  they are the preserved "before" state.
- All UI strings go in one STRINGS object (future localisation).
- Money layers: buffer = emergencyMonths × fixedMonthlyCosts;
  reserve = reserveMonths × monthlySpend (v1 simplification).
- Portfolio allocation (65/30/5, ~9.7% blended) is a code constant,
  not state.
- Propose changes as small, coherent chunks — I review and commit
  myself in GitHub Desktop.
- The disclaimer (PRD R8) must appear near results and in the footer.