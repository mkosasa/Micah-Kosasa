# Marginal analysis

Allocating a constrained resource across competing uses by comparing the
marginal contribution of each additional unit committed, where that contribution
changes as more units are committed.

**Exercised in:** perfect-competition (`docs/briefs/perfect-competition-brief.md`,
`analysis/perfect-competition-analysis.md`)

## Files

- `spec.md` — the named contract: season and whole-farm constants, the labor
  supply, the per-crop crop table, the calculation logic in named-range
  notation, the conventions, and the validation rules the built model must pass.
- `README.md` — this file.
- `model.xlsx` — the Solver workbook, built from `spec.md` (12 sheets: Inputs,
  CropEconomics, MCSchedules, CostStructure, Enumeration, Optimization,
  WorkedExample, Checks, Summary, Unconstrained, FertScenario, FertEnum). Every input is a named range; every calculated
  cell is a formula; the Checks sheet computes the validation rules as live
  PASS/FAIL flags. Built by Claude Code without Excel available. A grading
  review found that the committed file had 67,180 formula cells with no cached
  value (mostly in the `FertEnum` and `Enumeration` grids) and that
  `fullCalcOnLoad` was not actually set, despite an earlier version of this
  sentence claiming both; see the last entry in `spec.md` Audit findings for
  what was found and how it was fixed. As of that fix, every formula cell has a
  cached value and `fullCalcOnLoad` is set, so the file opens complete for a
  reader who does not force a recalculation. The manual audits (the checks that
  needed Excel) were completed by Micah; findings are in `spec.md`. `Summary`
  rows 32–45 hold the Stage 3 shadow-price block (named outputs
  `SHADOW_PRICE_TOM/CAR/MES`). `MCSchedules` rows 105–152 continue the carrot and
  mesclun standalone schedules past their caps to the profit peak (named outputs
  `XING_UNCAPPED_CAR/MES`) — a what-if display that changes no result. Both
  additions came after the manual audits; an Excel save of the workbook with the
  new sheets recalculated with no differences (see the last entry in `spec.md`
  Audit findings). The workbook also carries the Solver settings from that Excel
  save, but they are partial (one changing cell, no constraints), so the full
  integer Solver check is still open.
  `Unconstrained` shows the farm with every limit released (beds and temp
  workers): stop bed per crop, beds and labor against today's limits, profit.
  `FertScenario` (with its grid `FertEnum`) tests a 30% fertilizer discount after
  40 beds farm-wide, under today's limits and with limits released. Both are
  what-ifs that change no result; `model.xlsx` is now about 9.9 MB, mostly the grid.

## Engagements

| Engagement | Brief | Status |
|---|---|---|
| perfect-competition | `docs/briefs/perfect-competition-brief.md` | Spec written; model built (`model.xlsx`); audit findings recorded |

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
