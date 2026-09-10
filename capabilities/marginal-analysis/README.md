# Marginal analysis

Allocating a constrained resource across competing uses by comparing the
marginal contribution of each additional unit committed, where that contribution
changes as more units are committed.

**Exercised in:** perfect-competition (`docs/briefs/perfect-competition-brief.md`)

## Files

- `spec.md` — the named contract: season and whole-farm constants, the labor
  supply, the per-crop crop table, the calculation logic in named-range
  notation, the conventions, and the validation rules the built model must pass.
- `README.md` — this file.
- `model.xlsx` — the Solver workbook, built from `spec.md`. Not yet built;
  commit it after `spec.md`, never before.

## Engagements

| Engagement | Brief | Status |
|---|---|---|
| perfect-competition | `docs/briefs/perfect-competition-brief.md` | Spec written; model not built |

## Resolved modeling questions

- **Diminishing returns load onto labor hours**, compounding as
  `LABOR_HRS(c, q) = q x LABOR_HRS_WK(c) x SEASON_WEEKS x (1 + DIM_PCT(c)) ^ q` —
  the form given by the case. Two other readings (a summed per-bed geometric
  series; decline applied to price or yield) were considered and rejected; price
  is fixed regardless of quantity, so diminishing returns cannot touch revenue.
- **Labor is hourly, and the P&L uses a blended rate.** The farmer's 720 hours
  are consumed first at $34.72/hr, temporary hours follow at $17.36/hr (billed
  by the hour, not in whole-worker blocks), and each crop's P&L labor cost is
  its hours times the farm-level blended rate.

See the Conventions section of `spec.md` for the full statement. Do not re-open
these in the build without a change to the case or brief.
