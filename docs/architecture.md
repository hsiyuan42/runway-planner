# Architecture — Information Architecture & State Model

Phase 0.3–0.4 output. This is the contract the merged app is built against.

## Information architecture (0.3)

Three views over one shared state object. Views never communicate with each other — they all read and write the same state, which auto-saves to localStorage on every change.

```
                    ┌─────────────┐
                    │ Setup view  │  6 core inputs, defaults prefilled
                    └──────┬──────┘
                           ▼
 ┌────────────┐    ┌───────────────┐    ┌──────────────┐
 │ Claude API │◂──▸│   App state   │◂──▸│ localStorage │
 │ (Phase 3)  │    │  (one JSON)   │    │  auto-save   │
 └────────────┘    └───┬───────┬───┘    └──────────────┘
                       ▼       ▼
            ┌─────────────┐ ┌─────────────┐
            │ Runway view │ │  Plan view  │
            └─────────────┘ └─────────────┘
```

### Views and ownership

| Element | Contents | Notes |
|---|---|---|
| **Persistent header** | Effective runway + binding constraint ("Cash runs out Mar 2027" / "Visa is the limit: Apr 2027") | Visible above all tabs — the headline answer is always on screen |
| **Setup view** | Currency, savings, monthly spend, fixed monthly costs, income (via presets), visa expiry | First visit lands here; returning visits (saved state found) land on Runway |
| **Runway view** | Cash flow chart, month-by-month table, income period editor, one-off expense list, horizon selector | The "how long does my money last" question |
| **Plan view** | Layer breakdown (buffer/reserve/investable) with adjustable multipliers, monthly invest amount, compound growth projection, runway trade-off | The "what can I safely spend/invest" question |

Decisions:
- **Setup is a tab, not a wizard.** Users continually tweak inputs; a modal wizard would obstruct that. Defaults make the empty state meaningful.
- **`invest.monthlyAmount` is editable in both Runway and Plan views** — one state field, two surfaces.
- **Single HTML file, no framework, no build step** for v1. Keeps deploys trivial and the source readable.
- All UI strings live in one `STRINGS` object at the top of the file (future localisation; PRD §8).

## State model (0.4)

One serialisable JSON object. Everything derived (month table, layer split, effective runway, portfolio projection) is computed from state on render — never stored.

```json
{
  "schemaVersion": 1,
  "profile": {
    "currency": "GBP",
    "startMonth": "2026-08",
    "savings": 10000,
    "monthlySpend": 1400,
    "fixedMonthlyCosts": 800,
    "visaExpiry": "2027-04"
  },
  "income": {
    "activePreset": "custom",
    "periods": [
      { "id": "p1", "label": "Part-time contract", "monthlyAmount": 1020, "from": "2026-08", "to": "2026-12" },
      { "id": "p2", "label": "Expected full-time", "monthlyAmount": 2200, "from": "2027-01", "to": null }
    ]
  },
  "oneOffExpenses": [
    { "id": "e1", "month": "2026-10", "label": "Visa renewal fee", "amount": 1500 }
  ],
  "layerRules": {
    "emergencyMonths": 3,
    "reserveMonths": 6
  },
  "invest": {
    "monthlyAmount": 100,
    "existingPortfolio": 0
  },
  "view": {
    "horizonMonths": 18
  }
}
```

### Field notes

- **`income.periods`** — replaces the prototype's hardcoded scenario dropdown. Each period: amount + date range; `"to": null` means ongoing; months not covered by any period have zero income. The old scenarios become *presets* — buttons that generate a periods array from the user's dates, which remains fully editable. Overlapping periods sum (allows "part-time + side gig").
- **`fixedMonthlyCosts`** — needed because buffer = emergencyMonths × *fixed* costs, not total spend. Defaults to 60% of `monthlySpend`, user-adjustable.
- **Dates are `"YYYY-MM"` strings.** The model is month-granular; avoids Date/timezone parsing bugs.
- **`schemaVersion`** — bump on any breaking shape change; `load()` runs migrations on older versions found in localStorage.
- **`visaExpiry` may be null** (not everyone has one). Effective runway = min(cash-depletion month, visaExpiry). Header states which binds.
- **`existingPortfolio`** — seed value for the projection (the prototype hardcoded £1,920).
- **Not in state:** portfolio allocation (65/30/5) and blended return (~9.7%) are code constants per PRD §8; UI strings; anything derivable.

### Derived values (computed, never stored)

| Derived | Formula |
|---|---|
| Monthly income for month *m* | Sum of all periods covering *m* |
| Net for month *m* | income − monthlySpend − invest.monthlyAmount − oneOffs in *m* |
| Cash balance | Running sum from savings; floor at 0 = depletion month |
| Effective runway | min(depletion month, visaExpiry) + which constraint binds |
| Emergency buffer | emergencyMonths × fixedMonthlyCosts |
| Runway reserve | reserveMonths × (monthlySpend − expected income during search) — v1 simplification: reserveMonths × monthlySpend |
| Investable now | max(0, savings − buffer − reserve) |
| Portfolio at month *m* | (previous + monthlyAmount) × (1 + monthlyRate), seeded with existingPortfolio |

### Persistence & Phase 3 contract

- `save()` writes the object to `localStorage["runway-planner-state-v1"]` on every mutation (debounced).
- `load()` validates `schemaVersion`, migrates if needed, falls back to defaults if absent/corrupt.
- Phase 3 sends exactly this object to the Claude API (via the serverless proxy) for the plain-language summary, and receives this shape back from natural-language scenario input. The schema doubles as the AI contract — changes here are API changes.
