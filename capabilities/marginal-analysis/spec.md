---
type: spec
capability: marginal-analysis
engagement: perfect-competition
date: 2026-09-09
status: draft            # draft | built | audited
built_with: "Claude Code, from this file"
---

# Marginal analysis — model specification

## Purpose
This model supports one decision: how many beds to plant of tomatoes, carrots,
and mesclun for a 36-week season, given fixed per-crop prices, fertilizer costs,
labor requirements, and bed caps, plus a fixed labor supply (the farmer's own
720 field hours and up to four temporary workers). It must report the integer
bed allocation that maximizes season profit, the season profit and cost
breakdown at that allocation, which constraint stops the allocation where it
does, and — for each crop analyzed on its own — the bed count at which price
meets marginal cost.

## Inputs — the named contract

### Season and whole-farm
| Name | Value | Unit | Source |
|---|---|---|---|
| `SEASON_WEEKS` | 36 | weeks | Case scenario |
| `TOTAL_BEDS` | 64 | beds (16 beds x 4 plots) | Case scenario |
| `FIXED_COSTS` | 20,000 | USD per season | Case scenario |

### Labor supply
| Name | Value | Unit | Source |
|---|---|---|---|
| `FARMER_HRS` | 720 | field hours per season | Case scenario ("half her time in the field") |
| `FARMER_RATE` | 34.72 | USD per field hour | Derived: `(FARMER_WAGE / 2) / FARMER_HRS` = 25,000 / 720 |
| `FARMER_WAGE` | 50,000 | USD per season | Case scenario — retained only as the derivation source for `FARMER_RATE` |
| `TEMP_WORKER_RATE` | 17.36 | USD per hour | Derived: `TEMP_WORKER_WAGE / TEMP_WORKER_HRS` = 25,000 / 1,440 |
| `TEMP_WORKER_MAX` | 4 | workers | Case scenario |
| `TEMP_WORKER_WAGE` | 25,000 | USD per worker per season | Case scenario — derivation source for `TEMP_WORKER_RATE` |
| `TEMP_WORKER_HRS` | 1,440 | hours per worker per season | Case scenario |
| `LABOR_HRS_CAP` | 6,480 | hours per season | Derived: `FARMER_HRS + TEMP_WORKER_MAX x TEMP_WORKER_HRS` |

The P&L prices labor with `FARMER_RATE` and `TEMP_WORKER_RATE`. `FARMER_WAGE`,
`TEMP_WORKER_WAGE`, and `TEMP_WORKER_MAX` exist only to derive those rates and
the total-hours ceiling; no salary or whole-worker figure enters a cost formula.

### Per-crop economics (crop table)
`c` ranges over `{TOM, CAR, MES}`.

| Name | TOM | CAR | MES | Unit | Source |
|---|---|---|---|---|---|
| `DIM_PCT(c)` | 10.00% | 2.50% | 1.25% | per bed | Case scenario, crop table |
| `FERT_BED(c)` | 880 | 440 | 880 | USD per bed | Case scenario, crop table |
| `LABOR_HRS_WK(c)` | 2.50 | 0.833 | 1.25 | hours per week per bed | Case scenario, crop table |
| `PRICE(c)` | 8,800 | 2,094 | 2,700 | USD per bed | Case scenario, crop table |
| `MAX_BEDS(c)` | 20 | 20 | 30 | beds | Case scenario, crop table |

### Decision variables
| Name | Unit | Source |
|---|---|---|
| `q(TOM)`, `q(CAR)`, `q(MES)` | beds (integer) | Solver output |

## Structure
One workbook. Sheets / regions, in this order:

- **Inputs** — every named input above in its own labeled cell, entered once.
  Everything downstream references these by name; no input value is retyped.
- **Crop economics** — one block per crop: revenue, fertilizer cost, and season
  labor-hour demand as a function of that crop's own bed count `q`.
- **Marginal-cost schedules** — one block per crop, standalone (that crop as the
  only crop on the farm): the total variable cost and the marginal cost of the
  `q`-th bed for `q = 1 .. MAX_BEDS(c)`, set against `PRICE(c)`, with the
  standalone `PRICE ≈ marginal cost` crossing bed count flagged.
- **Cost structure** — at the current allocation: total labor hours, the
  farmer / temporary split, total labor cost, the blended labor rate, each
  crop's blended-rate labor charge, total fertilizer cost, fixed costs, total
  cost.
- **Optimization (Solver)** — the three decision variables, the constraint set,
  and the `PROFIT` objective cell.
- **Checks** — the validation checks below: the `q = 1` hand calculation, the
  Farm Profit Lab reconciliation, the two-starting-point Solver runs, and the
  formula / no-error scan.
- **Summary** — the solved allocation and every named output in one place.

## Calculation logic
Named-range notation, never cell addresses. For crop `c` at that crop's own bed
count `q`:

    REVENUE(c, q)   = q x PRICE(c)
    FERT_COST(c, q) = q x FERT_BED(c)
    LABOR_HRS(c, q) = q x LABOR_HRS_WK(c) x SEASON_WEEKS x (1 + DIM_PCT(c)) ^ q

### Standalone marginal-cost schedule
For crop `c` treated as the only crop on the farm, for `q = 1 .. MAX_BEDS(c)`
(`LABOR_HRS(c, 0) = 0`):

    FARMER_HRS_USED(c, q) = MIN(LABOR_HRS(c, q), FARMER_HRS)
    TEMP_HRS_USED(c, q)   = MAX(0, LABOR_HRS(c, q) - FARMER_HRS)
    LABOR_COST_SA(c, q)   = FARMER_HRS_USED(c, q) x FARMER_RATE
                          + TEMP_HRS_USED(c, q)  x TEMP_WORKER_RATE
    TVC(c, q)             = FERT_COST(c, q) + LABOR_COST_SA(c, q)
    MARG_COST(c, q)       = TVC(c, q) - TVC(c, q - 1)
    MARG_REVENUE(c)       = PRICE(c)                        [constant — price taker]
    XING(c)               = smallest q in 1 .. MAX_BEDS(c) with MARG_COST(c, q) >= PRICE(c)
                            (none in range => report "no crossing below cap")

### Roll-up at the current allocation
    TOTAL_LABOR_HRS   = LABOR_HRS(TOM, q(TOM)) + LABOR_HRS(CAR, q(CAR)) + LABOR_HRS(MES, q(MES))
    FARMER_HRS_USED   = MIN(TOTAL_LABOR_HRS, FARMER_HRS)
    TEMP_LABOR_HRS    = MAX(0, TOTAL_LABOR_HRS - FARMER_HRS)
    TOTAL_LABOR_COST  = FARMER_HRS_USED x FARMER_RATE + TEMP_LABOR_HRS x TEMP_WORKER_RATE
    BLENDED_RATE      = TOTAL_LABOR_COST / TOTAL_LABOR_HRS
    ALLOC_LABOR_COST(c) = LABOR_HRS(c, q(c)) x BLENDED_RATE
    TOTAL_FERT_COST   = FERT_COST(TOM, q(TOM)) + FERT_COST(CAR, q(CAR)) + FERT_COST(MES, q(MES))
    TOTAL_REVENUE     = REVENUE(TOM, q(TOM)) + REVENUE(CAR, q(CAR)) + REVENUE(MES, q(MES))
    PROFIT            = TOTAL_REVENUE - TOTAL_FERT_COST - TOTAL_LABOR_COST - FIXED_COSTS

`SUM(ALLOC_LABOR_COST(c)) = TOTAL_LABOR_COST` by construction; the per-crop
allocation feeds per-crop margin reporting, not the farm-level `PROFIT`.

### Solver setup
    Maximize:      PROFIT
    By changing:   q(TOM), q(CAR), q(MES)
    Subject to:
      q(TOM) + q(CAR) + q(MES)  <=  TOTAL_BEDS
      0 <= q(c) <= MAX_BEDS(c)                       for each crop c
      TEMP_LABOR_HRS  <=  TEMP_WORKER_MAX x TEMP_WORKER_HRS      (5,760)
      q(TOM), q(CAR), q(MES) integer
    Engine:  GRG Nonlinear, with integer constraints on the decision cells.
    Run from two starting points — (0, 0, 0) and (20, 0, 0) — and confirm both
    converge to the same allocation.

`TEMP_LABOR_HRS <= 5,760` is the same limit as `TOTAL_LABOR_HRS <= LABOR_HRS_CAP`
(6,480) and as "at most 4 temporary workers"; the Solver states one and the
others must hold.

## Conventions
- **Diminishing returns load onto labor hours, via `(1 + DIM_PCT(c)) ^ q`
  applied to the crop's whole season labor demand** — the form given by the
  case: `hours(q) = q x hrs-per-week-per-bed x 36 weeks x (1 + dim%)^q`. Adding
  a bed of a crop raises the attributed hours for every bed of that crop.
  - *Rejected reading 1:* per-bed escalation summed as a geometric series,
    `base x (1 + rate) x ((1 + rate) ^ q - 1) / rate`. Not the case's form.
  - *Rejected reading 2:* decline applied to price or yield,
    `PRICE(c) x (1 - rate) ^ (q - 1)`. Price is fixed regardless of quantity
    (price taker), so diminishing returns cannot touch revenue. An earlier
    abandoned build used this reading; it is wrong for this engagement.
- **Revenue and fertilizer cost are linear in `q`.** `PRICE(c)` is fixed
  (perfect competition); `FERT_BED(c)` is a flat per-bed cost. Diminishing
  returns touch neither.
- **Labor rule 1 — permanent before temporary.** The farmer's `FARMER_HRS`
  (720) are consumed first, priced at `FARMER_RATE` ($34.72). Every hour beyond
  720 is a temporary worker at `TEMP_WORKER_RATE` ($17.36), up to
  `TEMP_WORKER_MAX x TEMP_WORKER_HRS` (5,760). Temporary labor is billed by the
  hour, not in whole-worker blocks — a partly-used worker costs only the hours
  used.
- **Labor rule 2 — the P&L charges labor at the blended rate.**
  `BLENDED_RATE = TOTAL_LABOR_COST / TOTAL_LABOR_HRS` is a single farm-level
  figure. Each crop's P&L labor cost is its own labor hours x `BLENDED_RATE` —
  never its own farmer / temporary split. Both labor rules must be in the model;
  building only one of them is the most common structural defect in this model.
- **The farmer's field labor enters the P&L at
  `FARMER_RATE x FARMER_HRS_USED` (at most $25,000), not the $50,000 salary.**
  The non-field half of the farmer's time is outside this decision.
- **Costing order:** variable costs first — fertilizer, then labor at the
  blended rate — then `FIXED_COSTS`.
- **Beds are integers** everywhere — inputs, decision variables, schedules.
- **Boundaries:**
  - `LABOR_HRS(c, 0) = REVENUE(c, 0) = FERT_COST(c, 0) = 0`; the compounding term
    is never evaluated at `q = 0` for a live figure. The marginal-cost schedule
    starts at `q = 1`.
  - The standalone schedule prices each marginal bed's hours by where those
    hours fall against `FARMER_HRS` — the farmer's rate below 720 cumulative
    hours, the temporary rate above. Because `TEMP_WORKER_RATE` is *below*
    `FARMER_RATE`, the tomato standalone marginal-cost schedule is non-monotonic:
    it dips once, then resumes climbing. This is expected — do not "correct" it.
    Flag the dip in the schedule. Its cause is left for Stage 3; do not annotate
    it on the sheet or in this spec.
- **Caps bind even mid-margin.** If a crop's marginal profit is still positive at
  its `MAX_BEDS(c)`, the cap holds; the model never relaxes a cap to chase
  profit.

## Validation rules
Written as acceptance criteria for the built model.

Structural:
- Every calculated cell is a formula referencing named ranges — no hardcoded
  numbers in any computed cell, no error values (`#DIV/0!`, `#REF!`, `#VALUE!`,
  ...) anywhere in the workbook. Spot-check: change a named input and confirm
  every dependent figure moves.
- `q(TOM) + q(CAR) + q(MES) <= 64`; `0 <= q(c) <= MAX_BEDS(c)` for each crop.
- `TEMP_LABOR_HRS <= 5,760` and `TOTAL_LABOR_HRS <= 6,480` at the solved
  allocation.
- `TEMP_WORKER_WAGE / TEMP_WORKER_HRS` equals `TEMP_WORKER_RATE` (17.36);
  `(FARMER_WAGE / 2) / FARMER_HRS` equals `FARMER_RATE` (34.72).
- `SUM(ALLOC_LABOR_COST(c))` equals `TOTAL_LABOR_COST` — the blended-rate
  allocation ties out to the farm total.

Hand check — the `q = 1` exponent guard:
- `LABOR_HRS(TOM, 1) = 1 x 2.50 x 36 x 1.10 = 99 hours`, exactly. Repeat for
  each crop with its own `DIM_PCT(c)`. A result of `90` (i.e. `x 1.00`) means the
  `(1 + dim%)^q` exponent was dropped — the most common structural defect.
- Pick any allocation, compute `PROFIT` by hand, and confirm the workbook
  matches.

Cross-checks:
- **Farm Profit Lab.** Reconcile per-crop marginal cost, `TOTAL_LABOR_HRS`,
  `BLENDED_RATE`, and `PROFIT` against Farm Profit Lab — a separate
  implementation of the same model supplied with the case. Any divergence beyond
  rounding is a defect; trace it to the region that produced it.
- **Solver stability.** GRG Nonlinear from `(0, 0, 0)` and from `(20, 0, 0)`
  must land on the same allocation. Divergence means a local optimum — re-run
  with a tighter feasible start, or sweep the feasible integer combinations
  (`MAX_BEDS` bounds give at most `21 x 21 x 31 = 13,671`) as an independent
  maximum.

The Stage 2 case page publishes check figures for the optimal mix, season
profit, and the standalone `PRICE ≈ marginal cost` crossing points. Those are
acceptance targets for the built model and are recorded in Audit findings, not
restated here.

## Outputs
- `q(TOM)`, `q(CAR)`, `q(MES)` — the profit-maximizing integer bed allocation.
- `TOTAL_REVENUE`, `TOTAL_FERT_COST`, `TOTAL_LABOR_COST`, `PROFIT` — season
  totals, USD.
- `TOTAL_LABOR_HRS`, `FARMER_HRS_USED`, `TEMP_LABOR_HRS`, and the implied
  temporary-worker count `TEMP_LABOR_HRS / TEMP_WORKER_HRS`.
- `BLENDED_RATE`, and `ALLOC_LABOR_COST(c)` plus per-crop margin for each crop.
- `MARG_REVENUE(c)` and `MARG_COST(c, q(c))` at the solved allocation for each
  crop — why the allocation stops where it does.
- `XING(c)` — each crop's standalone `PRICE ≈ marginal cost` crossing bed count,
  from the marginal-cost schedule.
- Binding constraint at the optimum — one of: the 64-bed total, the 5,760
  temporary-hour ceiling, a crop's `MAX_BEDS(c)`, or "interior".
- The tomato marginal-cost-schedule dip — surfaced as a flag, not explained.

## Audit findings
Added after the model is built. For each check: what was checked, what was
found, what was done about it. At least three checks, including the `q = 1`
exponent guard and the Farm Profit Lab reconciliation. Compare the built model's
optimal mix, season profit, and standalone crossing points against the Stage 2
case page's published check figures and record any variance.

Example entry — At `q = 1`, `LABOR_HRS(TOM, 1)` returned 99 hours; hand
calculation `1 x 2.50 x 36 x 1.10` = 99 hours — PASS.

Not yet audited — no model has been built from this spec.
