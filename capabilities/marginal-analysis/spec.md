---
type: spec
capability: marginal-analysis
engagement: perfect-competition
date: 2026-08-24
status: draft            # draft | built | audited
---

# Marginal analysis — model specification

## Purpose
This model supports one decision: how many beds to plant of tomatoes, carrots, and
mesclun for a 36-week season, given fixed per-crop prices, fertilizer costs, labor
requirements, and caps, plus a fixed labor supply (the farmer's own hours and up to
four temporary workers). It must be able to answer: which bed allocation maximizes
season profit, and does that allocation exhaust the labor supply or the bed count
first?

## Inputs — the named contract
| Name | Value | Unit | Source |
|---|---|---|---|
| `SEASON_WEEKS` | 36 | weeks | Case scenario |
| `TOTAL_BEDS` | 64 | beds (16 beds x 4 plots) | Case scenario |
| `FIXED_COSTS` | 20,000 | USD per season | Case scenario |
| `FARMER_WAGE` | 50,000 | USD per season | Case scenario |
| `FARMER_HRS` | 720 | hours per season | Case scenario (half the farmer's time in the field) |
| `FARMER_RATE` | 34.72 | USD per hour, implied | Derived: `FARMER_WAGE / FARMER_HRS` |
| `TEMP_WORKER_MAX` | 4 | workers | Case scenario |
| `TEMP_WORKER_WAGE` | 25,000 | USD per season per worker | Case scenario |
| `TEMP_WORKER_HRS` | 1,440 | hours per season per worker | Case scenario |
| `TEMP_WORKER_RATE` | 17.36 | USD per hour, implied | Derived: `TEMP_WORKER_WAGE / TEMP_WORKER_HRS` |
| `DIM_PCT(TOM)` | 10.00% | per bed | Case scenario, crop table |
| `DIM_PCT(CAR)` | 2.50% | per bed | Case scenario, crop table |
| `DIM_PCT(MES)` | 1.25% | per bed | Case scenario, crop table |
| `FERT_COST(TOM)` | 880 | USD per bed | Case scenario, crop table |
| `FERT_COST(CAR)` | 440 | USD per bed | Case scenario, crop table |
| `FERT_COST(MES)` | 880 | USD per bed | Case scenario, crop table |
| `LABOR_HRS_WK(TOM)` | 2.50 | hours per week per bed | Case scenario, crop table |
| `LABOR_HRS_WK(CAR)` | 0.833 | hours per week per bed | Case scenario, crop table |
| `LABOR_HRS_WK(MES)` | 1.25 | hours per week per bed | Case scenario, crop table |
| `PRICE(TOM)` | 8,800 | USD per bed | Case scenario, crop table |
| `PRICE(CAR)` | 2,094 | USD per bed | Case scenario, crop table |
| `PRICE(MES)` | 2,700 | USD per bed | Case scenario, crop table |
| `MAX_BEDS(TOM)` | 20 | beds | Case scenario, crop table |
| `MAX_BEDS(CAR)` | 20 | beds | Case scenario, crop table |
| `MAX_BEDS(MES)` | 30 | beds | Case scenario, crop table |
| `q(TOM)`, `q(CAR)`, `q(MES)` | decision variables | beds | Solver output |

## Structure
- **Inputs** — the named contract above, entered once, referenced everywhere else by name.
- **Crop economics** — one block per crop: revenue, fertilizer cost, and labor-hour
  demand as a function of that crop's own bed count `q`, plus per-bed and marginal
  figures used to sanity-check the solver.
- **Labor & cost roll-up** — sums labor hours across the three crops at the current
  allocation, converts that into workers-needed and total labor cost, and adds
  fertilizer cost and fixed costs to get total cost.
- **Allocation / solver** — the three decision variables (`q(TOM)`, `q(CAR)`,
  `q(MES)`), the constraint set, and the objective cell, set up for Excel Solver.
- **Summary** — the solved allocation and the outputs listed below, in one place for
  reporting.

## Calculation logic
For crop `c` in {TOM, CAR, MES}, at that crop's own bed count `q`:

    REVENUE(c, q)   = q x PRICE(c)
    FERT_COST(c, q) = q x FERT_COST(c)
    LABOR_HRS(c, q) = q x LABOR_HRS_WK(c) x SEASON_WEEKS x (1 + DIM_PCT(c))^q

Roll-up across the solved allocation:

    TOTAL_LABOR_HRS = LABOR_HRS(TOM, q(TOM)) + LABOR_HRS(CAR, q(CAR)) + LABOR_HRS(MES, q(MES))
    WORKERS_NEEDED  = MAX(0, ROUNDUP((TOTAL_LABOR_HRS - FARMER_HRS) / TEMP_WORKER_HRS, 0))
    LABOR_COST      = FARMER_WAGE + WORKERS_NEEDED x TEMP_WORKER_WAGE
    TOTAL_FERT_COST = FERT_COST(TOM, q(TOM)) + FERT_COST(CAR, q(CAR)) + FERT_COST(MES, q(MES))
    TOTAL_REVENUE   = REVENUE(TOM, q(TOM)) + REVENUE(CAR, q(CAR)) + REVENUE(MES, q(MES))
    PROFIT          = TOTAL_REVENUE - TOTAL_FERT_COST - LABOR_COST - FIXED_COSTS

Solver setup:

    Maximize: PROFIT
    By changing: q(TOM), q(CAR), q(MES)
    Subject to:
      q(TOM) + q(CAR) + q(MES) <= TOTAL_BEDS
      0 <= q(c) <= MAX_BEDS(c)                for each crop c
      TOTAL_LABOR_HRS <= FARMER_HRS + TEMP_WORKER_MAX x TEMP_WORKER_HRS
      q(TOM), q(CAR), q(MES) are integers

## Conventions
- **Revenue and fertilizer cost are linear in `q`.** Price is fixed regardless of
  quantity (the perfect-competition, price-taking assumption from the engagement
  brief), and fertilizer cost is a constant per bed. Diminishing returns do not
  touch either of these.
- **Diminishing returns load onto labor hours, not price or yield.** Each
  additional bed of the same crop raises that crop's own labor-hour requirement,
  via the `(1 + DIM_PCT(c))^q` compounding term — read as crowding/disease-pressure
  effects that make a crop more labor-intensive to manage as more of it is planted,
  not as a change in what it sells for. This is the interpretation most consistent
  with the brief's assumption that "fertilizer and labor costs are fixed per
  bed/hour regardless of volume" (it's the *rate*, not the *hours*, that's fixed).
  This is also the least directly observable set of inputs per the brief, and is
  the first thing to test if the solved allocation looks wrong.
- **Labor is costed in whole-worker blocks, not by the hour.** The farmer's 720
  hours are used first against total demand; any remaining hours are covered by
  whole temporary workers at $25,000 each, regardless of how many of that worker's
  1,440 hours actually get used. Round up, never allocate a fractional worker.
- **Farmer wage and fixed costs are sunk, not allocation-dependent.** They enter
  `PROFIT` once, at the end, regardless of the crop mix chosen.
- **Beds are integers.** No fractional beds anywhere, in inputs or in the solver.
- **Costing order:** fertilizer cost and labor cost first (variable, allocation-
  dependent), then fixed costs and farmer wage last (period costs, allocation-
  independent).
- **Boundary:** at `q(c) = 0`, `LABOR_HRS(c, 0)` and `FERT_COST(c, 0)` are both
  zero regardless of `DIM_PCT(c)` — the compounding term never needs to be
  evaluated at zero beds to get the right answer.
- **Caps bind even mid-margin.** If a crop's marginal profit is still positive at
  its `MAX_BEDS(c)` cap, the cap wins; the model does not relax caps to chase
  additional profit.

## Validation rules
- Total allocated beds (`q(TOM) + q(CAR) + q(MES)`) does not exceed `TOTAL_BEDS` (64).
- Each `q(c)` is between 0 and `MAX_BEDS(c)`, inclusive.
- `TOTAL_LABOR_HRS` does not exceed `FARMER_HRS + TEMP_WORKER_MAX x TEMP_WORKER_HRS` (6,480).
- `WORKERS_NEEDED` never exceeds `TEMP_WORKER_MAX` (4); if it does, the solver's
  labor constraint was set up wrong.
- Hand check: at `q = 1` for any crop, `LABOR_HRS(c, 1) = LABOR_HRS_WK(c) x SEASON_WEEKS x (1 + DIM_PCT(c))`
  — confirm this by hand before trusting the model at larger `q`.
- Every calculated cell contains a formula referencing named inputs; no hardcoded
  numbers, no error cells (`#DIV/0!`, `#REF!`, etc.) anywhere in the solved
  workbook.
- The solved allocation should be compared against the engagement brief's stated
  hypothesis (20 tomato / 30 mesclun / 14 carrot beds) as a check, not a target —
  a mismatch is not itself a defect, but it should be explainable from the
  constraints and marginal figures, not shrugged off.

## Outputs
- `q(TOM)`, `q(CAR)`, `q(MES)` — the profit-maximizing bed allocation.
- `WORKERS_NEEDED` — temporary workers to hire (0-4).
- `TOTAL_REVENUE`, `TOTAL_FERT_COST`, `LABOR_COST`, `PROFIT` — season totals.
- `TOTAL_LABOR_HRS` versus available labor hours (6,480) — utilization and slack.
- Marginal profit per crop at the solved allocation, for interpreting *why* the
  solver stopped where it did (binding cap vs. binding labor supply vs. an
  interior optimum).

## Audit findings
Not yet audited — no model has been built from this spec yet.
