---
type: spec
capability: marginal-analysis
engagement: perfect-competition
date: 2026-09-09
status: draft            # draft | built | audited
built_with: "Claude Code, from this file"
---

# Marginal analysis — model specification

**Sources.** Case scenario and crop table, and the Stage 2 build requirements:
`https://adamwstauffer.github.io/ai-lms/case-perfect-competition-stage2.html`.
Cross-validation reference (Farm Profit Lab):
`https://adamwstauffer.github.io/ai-lms/farmlab.html`.
Every input below traces to the case scenario; the labor formula and the two
labor conventions are stated on the Stage 2 page.

## Purpose
This model supports one decision: how many beds to plant of tomatoes, carrots,
and mesclun for a 36-week season, given fixed per-crop prices, fertilizer costs,
labor requirements, and bed caps, plus a fixed labor supply (the farmer's own
720 field hours and up to four temporary workers). It must report the integer
bed allocation that maximizes season profit, the season profit and its cost
breakdown at that allocation, which constraint stops the allocation where it
does, and — for each crop analyzed on its own — the bed count at which price
meets marginal cost.

## Definitions
- **P&L** — the season profit expression
  `PROFIT = TOTAL_REVENUE - TOTAL_FERT_COST - TOTAL_LABOR_COST - FIXED_COSTS`.
  "Enters the P&L" means it appears in that expression or one of its components.
- **Current allocation** — the live values in the decision cells `q(TOM)`,
  `q(CAR)`, `q(MES)`. Every roll-up formula reads those cells directly.
- **Solved allocation** — the same cells after Solver has run and its result has
  been accepted. The model keeps no frozen copy; "solved" just means "current,
  after Solver."
- **Standalone**, of a crop `c` — `c` modeled as if it were the only crop on the
  farm: the full `FARMER_HRS` (720) is available to it and beds are free up to
  `MAX_BEDS(c)`.
- **TVC(c, q)** — total variable cost of `q` beds of crop `c`:
  `FERT_COST(c, q) + LABOR_COST_SA(c, q)`. Excludes `FIXED_COSTS`.
- **Marginal cost of the q-th bed** — `TVC(c, q) - TVC(c, q - 1)`.
- **BLENDED_RATE** — `TOTAL_LABOR_COST / TOTAL_LABOR_HRS`, a single farm-level
  dollars-per-hour figure at the current allocation.
- **Interior optimum** — no constraint binds: every crop's marginal profit has
  fallen to zero or below before its `MAX_BEDS(c)` cap and before any resource
  limit.
- **Exponent guard** — the `q = 1` hand check confirming `(1 + DIM_PCT(c)) ^ q`
  was built with the exponent, not dropped.
- **Farm Profit Lab** — an interactive web calculator
  (`https://adamwstauffer.github.io/ai-lms/farmlab.html`), a separate
  implementation of this same model: same labor formula
  `q x hrs/wk x 36 x (1 + dim) ^ q`, same permanent-before-temporary split at
  `FARMER_RATE` / `TEMP_WORKER_RATE`, same 64-bed / three-crop structure with the
  carrot cap. It reports revenue, total labor hours, the farmer / temporary
  labor-cost split, fertilizer and fixed costs, total profit, and a
  marginal-cost-vs-price chart per crop. Use it for the cross-check in
  Validation. Note: the Lab explains the tomato marginal-cost dip on screen;
  this spec still defers that explanation to Stage 3.

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
| `FARMER_RATE` | `(FARMER_WAGE / 2) / FARMER_HRS` = 25,000 / 720 | USD per field hour | Derived; displays as 34.72 |
| `FARMER_WAGE` | 50,000 | USD per season | Case scenario — documented input; feeds `FARMER_RATE` only, no other formula |
| `TEMP_WORKER_RATE` | `TEMP_WORKER_WAGE / TEMP_WORKER_HRS` = 25,000 / 1,440 | USD per hour | Derived; displays as 17.36 |
| `TEMP_WORKER_MAX` | 4 | workers | Case scenario |
| `TEMP_WORKER_WAGE` | 25,000 | USD per worker per season | Case scenario — feeds `TEMP_WORKER_RATE` and the hours ceiling only |
| `TEMP_WORKER_HRS` | 1,440 | hours per worker per season | Case scenario |
| `LABOR_HRS_CAP` | 6,480 | hours per season | Derived: `FARMER_HRS + TEMP_WORKER_MAX x TEMP_WORKER_HRS` |

`FARMER_RATE` and `TEMP_WORKER_RATE` are held as the **exact quotients** in their
own cells. The values `34.72` and `17.36` are display only; no formula reads a
rounded rate.

### Per-crop economics (crop table)
`c` ranges over `{TOM, CAR, MES}`. `DIM_PCT(c)` is entered as a **decimal
fraction** (0.10, 0.025, 0.0125) in its own cell; the "%" below is presentation.

| Name | TOM | CAR | MES | Unit | Source |
|---|---|---|---|---|---|
| `DIM_PCT(c)` | 10.00% | 2.50% | 1.25% | per bed (decimal) | Case scenario, crop table |
| `FERT_BED(c)` | 880 | 440 | 880 | USD per bed | Case scenario, crop table |
| `LABOR_HRS_WK(c)` | 2.50 | 0.833 | 1.25 | hours per week per bed | Case scenario, crop table |
| `PRICE(c)` | 8,800 | 2,094 | 2,700 | USD per bed | Case scenario, crop table |
| `MAX_BEDS(c)` | 20 | 20 | 30 | beds | Case scenario, crop table |

`LABOR_HRS_WK(CAR)` is `0.833` exactly as printed — not `5/6`.

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
- **Marginal-cost schedules** — one block per crop, standalone: total variable
  cost and the marginal cost of the `q`-th bed for `q = 1 .. MAX_BEDS(c)`, set
  against `PRICE(c)`, with `XING(c)` and `XING_FIRST(c)` reported.
- **Cost structure** — at the current allocation: total labor hours, the
  farmer / temporary split, total labor cost, the blended labor rate, each
  crop's blended-rate labor charge, total fertilizer cost, fixed costs, total
  cost, and per-crop margin.
- **Optimization (Solver)** — the three decision variables, the constraint set,
  and the `PROFIT` objective cell.
- **Enumeration** — every feasible integer `(q(TOM), q(CAR), q(MES))` and its
  profit; the maximum row. Always built.
- **Checks** — the validation checks below as live cells with PASS / FAIL flags.
- **Worked example** — the fixed reconciliation table below, recomputed by the
  live formulas at `q = (5, 5, 5)`.
- **Summary** — the solved allocation and every named output in one place.

## Calculation logic
Named-range notation, never cell addresses. For crop `c` at that crop's own bed
count `q`:

    REVENUE(c, q)   = q x PRICE(c)
    FERT_COST(c, q) = q x FERT_BED(c)
    LABOR_HRS(c, q) = q x LABOR_HRS_WK(c) x SEASON_WEEKS x (1 + DIM_PCT(c)) ^ q

The `(1 + DIM_PCT(c)) ^ q` factor multiplies the crop's *entire* labor demand,
not just the last bed. Illustration — tomatoes at `q = 3`:
`LABOR_HRS = 3 x 2.50 x 36 x 1.10 ^ 3 = 270 x 1.331 = 359.37` hours; the `1.331`
applies to all three beds.

### Standalone marginal-cost schedule
For crop `c` treated as the only crop on the farm, for `q = 1 .. MAX_BEDS(c)`
(`LABOR_HRS(c, 0) = 0`, `TVC(c, 0) = 0`):

    FARMER_HRS_USED(c, q) = MIN(LABOR_HRS(c, q), FARMER_HRS)
    TEMP_HRS_USED(c, q)   = MAX(0, LABOR_HRS(c, q) - FARMER_HRS)
    LABOR_COST_SA(c, q)   = FARMER_HRS_USED(c, q) x FARMER_RATE
                          + TEMP_HRS_USED(c, q)  x TEMP_WORKER_RATE
    TVC(c, q)             = FERT_COST(c, q) + LABOR_COST_SA(c, q)
    MARG_COST(c, q)       = TVC(c, q) - TVC(c, q - 1)
    MARG_REVENUE(c)       = PRICE(c)                        [constant — price taker]
    SA_PROFIT(c, q)       = q x PRICE(c) - TVC(c, q)
    XING(c)               = the q in 0 .. MAX_BEDS(c) that maximizes SA_PROFIT(c, q)
    XING_FIRST(c)         = smallest q in 1 .. MAX_BEDS(c) with MARG_COST(c, q) >= PRICE(c)
                            (report "none below cap" if there is no such q)

The schedule uses `LABOR_COST_SA` — the farmer / temporary split rates — **not**
`BLENDED_RATE`. The step down from `FARMER_RATE` to the lower `TEMP_WORKER_RATE`
as cumulative hours pass `FARMER_HRS` is deliberate and is what makes the tomato
schedule non-monotonic. For a monotonic schedule `XING(c)` and `XING_FIRST(c)`
coincide; where they differ, both are reported so the dip stays visible.

### Roll-up at the current allocation
    TOTAL_LABOR_HRS   = LABOR_HRS(TOM, q(TOM)) + LABOR_HRS(CAR, q(CAR)) + LABOR_HRS(MES, q(MES))
    FARMER_HRS_USED   = MIN(TOTAL_LABOR_HRS, FARMER_HRS)
    TEMP_LABOR_HRS    = MAX(0, TOTAL_LABOR_HRS - FARMER_HRS)
    TOTAL_LABOR_COST  = FARMER_HRS_USED x FARMER_RATE + TEMP_LABOR_HRS x TEMP_WORKER_RATE
    BLENDED_RATE      = IF(TOTAL_LABOR_HRS = 0, 0, TOTAL_LABOR_COST / TOTAL_LABOR_HRS)
    ALLOC_LABOR_COST(c) = LABOR_HRS(c, q(c)) x BLENDED_RATE
    MARGIN(c)         = REVENUE(c, q(c)) - FERT_COST(c, q(c)) - ALLOC_LABOR_COST(c)
    TOTAL_FERT_COST   = FERT_COST(TOM, q(TOM)) + FERT_COST(CAR, q(CAR)) + FERT_COST(MES, q(MES))
    TOTAL_REVENUE     = REVENUE(TOM, q(TOM)) + REVENUE(CAR, q(CAR)) + REVENUE(MES, q(MES))
    PROFIT            = TOTAL_REVENUE - TOTAL_FERT_COST - TOTAL_LABOR_COST - FIXED_COSTS

`SUM(ALLOC_LABOR_COST(c)) = TOTAL_LABOR_COST` and
`SUM(MARGIN(c)) = PROFIT + FIXED_COSTS`, both by construction — use them as
tie-out checks. `MARGIN(c)` is a season total, variable margin only; it carries
no share of `FIXED_COSTS`.

### Solver setup
    Maximize:      PROFIT
    By changing:   q(TOM), q(CAR), q(MES)
    Subject to (all entered as explicit Solver constraints, not just cell bounds):
      q(TOM) + q(CAR) + q(MES)  <=  TOTAL_BEDS
      q(c) <= MAX_BEDS(c)                            for each crop c
      q(c) >= 0                                      for each crop c
      TEMP_LABOR_HRS  <=  TEMP_WORKER_MAX x TEMP_WORKER_HRS      (5,760)
      q(TOM), q(CAR), q(MES) = integer
    Engine:  GRG Nonlinear. Options: Integer Optimality = 0%.
    Run from two starting points — (0, 0, 0) and (20, 0, 0) — and confirm both
    converge to the same allocation. If they differ, the Enumeration maximum is
    authoritative.

`TEMP_LABOR_HRS <= 5,760` is the same limit as `TOTAL_LABOR_HRS <= LABOR_HRS_CAP`
(6,480) and as "at most 4 temporary workers".

## Conventions

### Model shape
- **Diminishing returns load onto labor hours, via `(1 + DIM_PCT(c)) ^ q`
  applied to the crop's whole season labor demand** — the case's form,
  `hours(q) = q x hrs-per-week-per-bed x 36 weeks x (1 + dim%)^q`.
  - *Rejected reading 1:* per-bed escalation summed as a geometric series,
    `base x (1 + rate) x ((1 + rate) ^ q - 1) / rate`. Not the case's form.
  - *Rejected reading 2:* decline applied to price or yield,
    `PRICE(c) x (1 - rate) ^ (q - 1)`. Price is fixed regardless of quantity
    (price taker), so diminishing returns cannot touch revenue.
- **Revenue and fertilizer cost are linear in `q`.** `PRICE(c)` fixed (perfect
  competition); `FERT_BED(c)` a flat per-bed cost.

### Labor
- **Rule 1 — permanent before temporary.** The farmer's `FARMER_HRS` (720) are
  consumed first at `FARMER_RATE`. Every hour beyond 720 is a temporary worker
  at `TEMP_WORKER_RATE`, up to `TEMP_WORKER_MAX x TEMP_WORKER_HRS` (5,760).
  Temporary labor is billed by the hour, not in whole-worker blocks — a
  partly-used worker costs only the hours used.
- **Rule 2 — the P&L charges labor at the blended rate.**
  `BLENDED_RATE = TOTAL_LABOR_COST / TOTAL_LABOR_HRS` is a single farm-level
  figure; each crop's P&L labor cost is its own hours x `BLENDED_RATE`, never
  its own farmer / temporary split. Both rules must be in the model — building
  only one is the most common structural defect in this model.
- **The farmer's field labor is billable, not salaried.** It enters the P&L at
  `FARMER_RATE x FARMER_HRS_USED`: it scales with hours worked and is $0 at zero
  hours. It reaches at most $25,000 (720 x `FARMER_RATE`). The `$50,000` salary
  never enters a cost formula; the non-field half of the farmer's time is
  outside this decision.
- **Costing order:** variable costs first — fertilizer, then labor at the
  blended rate — then `FIXED_COSTS`.

### Precision and boundaries
- **Rates are the exact quotient**, carried at full precision. `34.72` / `17.36`
  are display values; no formula reads a rounded rate.
- **No intermediate rounding.** Hours, costs, and `BLENDED_RATE` carry full
  precision throughout. Rounding is display-only: whole dollars for currency,
  two decimals for rates and hours.
- **`DIM_PCT(c)` is a decimal fraction** in its own cell (0.10, not 10).
- **Zero-labor state.** When `TOTAL_LABOR_HRS = 0` (the Solver start
  `(0, 0, 0)`), `BLENDED_RATE = 0` and every `ALLOC_LABOR_COST(c) = 0` and
  `MARGIN(c) = REVENUE - FERT_COST`. The `BLENDED_RATE` formula guards the
  division; no cell shows `#DIV/0!`.
- **`q = 0` for a crop.** `LABOR_HRS(c, 0) = REVENUE(c, 0) = FERT_COST(c, 0)
  = TVC(c, 0) = 0`; the compounding term is never evaluated at `q = 0`. The
  marginal-cost schedule starts at `q = 1`.
- **Beds are integers** everywhere — inputs, decision variables, schedules,
  enumeration.

### Scope
- **One crop per bed for the full 36-week season.** No succession planting, no
  bed turnover within the season.
- **Plots are descriptive.** The `16 beds x 4 plots` structure explains where 64
  beds come from; allocation is bed-level and is not constrained to whole plots
  or multiples of 16.
- **Unplanted beds are free.** `q(TOM) + q(CAR) + q(MES)` may be below 64; idle
  beds carry no holding cost.
- **Caps bind even mid-margin.** If a crop's marginal profit is still positive at
  its `MAX_BEDS(c)`, the cap holds; the model never relaxes a cap to chase
  profit.
- **The tomato standalone marginal-cost schedule is non-monotonic** — it dips
  once, then resumes climbing. This is expected; do not "correct" it. Flag the
  dip in the schedule. Its cause is left for Stage 3 — do not annotate it on the
  sheet or in this spec.

## Validation rules
Written as acceptance criteria for the built model. Build the **Checks** region
as live PASS / FAIL cells.

Structural:
- Every calculated cell is a formula referencing named ranges — no hardcoded
  numbers in any computed cell, no error values (`#DIV/0!`, `#REF!`, `#VALUE!`,
  ...) anywhere in the workbook. Spot-check: change a named input and confirm
  every dependent figure moves.
- `q(TOM) + q(CAR) + q(MES) <= 64`; `0 <= q(c) <= MAX_BEDS(c)` for each crop.
- `TEMP_LABOR_HRS <= 5,760` and `TOTAL_LABOR_HRS <= 6,480` at the solved
  allocation.
- `TEMP_WORKER_WAGE / TEMP_WORKER_HRS` equals `TEMP_WORKER_RATE`;
  `(FARMER_WAGE / 2) / FARMER_HRS` equals `FARMER_RATE`.
- `SUM(ALLOC_LABOR_COST(c))` equals `TOTAL_LABOR_COST`.
- `SUM(MARGIN(c))` equals `PROFIT + FIXED_COSTS`.

Hand check — the `q = 1` exponent guard:
- `LABOR_HRS(TOM, 1) = 1 x 2.50 x 36 x 1.10 = 99 hours`, exactly. Repeat for each
  crop with its own `DIM_PCT(c)`. A result of `90` (i.e. `x 1.00`) means the
  `(1 + dim%)^q` exponent was dropped — the most common structural defect.

Worked example — the model must reproduce this table at `q = (5, 5, 5)`, an
arbitrary non-optimal allocation chosen only as a numeric anchor. Figures shown
to the cent; the workbook carries full precision.

| Figure | Value |
|---|---|
| `LABOR_HRS(TOM, 5)` | 724.7295 hrs |
| `LABOR_HRS(CAR, 5)` | 169.6433 hrs |
| `LABOR_HRS(MES, 5)` | 239.4185 hrs |
| `TOTAL_LABOR_HRS` | 1,133.7913 hrs |
| `FARMER_HRS_USED` | 720.0000 hrs |
| `TEMP_LABOR_HRS` | 413.7913 hrs |
| `TOTAL_LABOR_COST` | 32,183.88 |
| `BLENDED_RATE` | 28.386068 USD/hr |
| `ALLOC_LABOR_COST(TOM)` | 20,572.22 |
| `ALLOC_LABOR_COST(CAR)` | 4,815.51 |
| `ALLOC_LABOR_COST(MES)` | 6,796.15 |
| `TOTAL_REVENUE` | 67,970 |
| `TOTAL_FERT_COST` | 11,000 |
| `FIXED_COSTS` | 20,000 |
| `PROFIT` | 4,786.12 |

Cross-checks:
- **Farm Profit Lab.** Reconcile `PROFIT`, `TOTAL_LABOR_HRS`, `BLENDED_RATE`,
  and each crop's `MARG_COST(c, q(c))` against Farm Profit Lab — the reference
  implementation supplied with the case. Tolerance: within $1 on `PROFIT` and
  `TOTAL_LABOR_HRS`, within $5 on each per-crop marginal cost. A larger gap is a
  defect; trace it to the region that produced it.
- **Solver stability.** GRG Nonlinear from `(0, 0, 0)` and from `(20, 0, 0)`
  must land on the same allocation. If they differ, report the Enumeration
  maximum.
- **Solver vs. enumeration.** The solved allocation must equal the maximum-profit
  row of the Enumeration region (`21 x 21 x 31 = 13,671` feasible integer
  combinations, filtered to the constraints).

The Stage 2 case page publishes check figures for the optimal mix, season
profit, and the standalone `PRICE ≈ marginal cost` crossing points. Those are
acceptance targets for the built model and are recorded in Audit findings after
the build — they are intentionally not restated here as acceptance criteria.

## Outputs
- `q(TOM)`, `q(CAR)`, `q(MES)` — the profit-maximizing integer bed allocation.
- `TOTAL_REVENUE`, `TOTAL_FERT_COST`, `TOTAL_LABOR_COST`, `FIXED_COSTS`,
  `PROFIT` — the season cost breakdown.
- `TOTAL_LABOR_HRS`, `FARMER_HRS_USED`, `TEMP_LABOR_HRS`, and the implied
  temporary-worker count `TEMP_LABOR_HRS / TEMP_WORKER_HRS` (reported as the
  fraction).
- `BLENDED_RATE`; `ALLOC_LABOR_COST(c)` and `MARGIN(c)` for each crop.
- `MARG_REVENUE(c)` and `MARG_COST(c, q(c))` at the solved allocation for each
  crop — why the allocation stops where it does.
- `XING(c)` and `XING_FIRST(c)` — each crop's standalone profit-maximizing bed
  count, and the first bed where marginal cost reaches price.
- Binding constraint at the optimum — one of: the 64-bed total, the 5,760
  temporary-hour ceiling, a crop's `MAX_BEDS(c)`, or "interior".
- The tomato marginal-cost-schedule dip — surfaced as a flag, not explained.

## Audit findings
Added after the model is built. For each check: what was checked, what was
found, what was done about it. At least three checks, including the `q = 1`
exponent guard, the `q = (5, 5, 5)` worked-example reconciliation, and the Farm
Profit Lab reconciliation. Compare the built model's optimal mix, season profit,
and standalone crossing points against the Stage 2 case page's published check
figures and record any variance.

Example entry — At `q = 1`, `LABOR_HRS(TOM, 1)` returned 99 hours; hand
calculation `1 x 2.50 x 36 x 1.10` = 99 hours — PASS.

Not yet audited — no model has been built from this spec.
