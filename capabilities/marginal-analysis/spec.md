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
This model supports one decision: how many beds to plant of tomatoes, carrots, and
mesclun for a 36-week season, given fixed per-crop prices, fertilizer costs, labor
requirements, and bed caps, plus a fixed labor supply (the farmer's own 720 hours
and up to four temporary workers). It must answer which integer bed allocation
maximizes season profit, whether that allocation runs out of labor hours or bed
capacity first, and — from the marginal figures — why the optimum sits where it
does rather than at a crop's cap.

## Inputs — the named contract

### Season and whole-farm
| Name | Value | Unit | Source |
|---|---|---|---|
| `SEASON_WEEKS` | 36 | weeks | Case scenario / brief |
| `TOTAL_BEDS` | 64 | beds (16 beds x 4 plots) | Case scenario / brief |
| `FIXED_COSTS` | 20,000 | USD per season | Case scenario / brief |

### Labor supply
| Name | Value | Unit | Source |
|---|---|---|---|
| `FARMER_WAGE` | 50,000 | USD per season | Case scenario / brief |
| `FARMER_HRS` | 720 | field hours per season | Case scenario ("half her time in the field") |
| `FARMER_RATE` | 34.72 | USD per field hour, implied | Derived: `(FARMER_WAGE / 2) / FARMER_HRS` — half the salary against the 720 field hours |
| `TEMP_WORKER_MAX` | 4 | workers | Case scenario / brief |
| `TEMP_WORKER_WAGE` | 25,000 | USD per worker per season | Case scenario / brief |
| `TEMP_WORKER_HRS` | 1,440 | hours per worker per season | Case scenario / brief |
| `TEMP_WORKER_RATE` | 17.36 | USD per hour, implied | Derived: `TEMP_WORKER_WAGE / TEMP_WORKER_HRS` |
| `LABOR_HRS_CAP` | 6,480 | hours per season | Derived: `FARMER_HRS + TEMP_WORKER_MAX x TEMP_WORKER_HRS` |

### Per-crop economics (crop table)
`c` ranges over `{TOM, CAR, MES}`.

| Name | TOM | CAR | MES | Unit | Source |
|---|---|---|---|---|---|
| `DIM_PCT(c)` | 10.00% | 2.50% | 1.25% | per bed | Crop table |
| `FERT_BED(c)` | 880 | 440 | 880 | USD per bed | Crop table |
| `LABOR_HRS_WK(c)` | 2.50 | 0.833 | 1.25 | hours per week per bed | Crop table |
| `PRICE(c)` | 8,800 | 2,094 | 2,700 | USD per bed | Crop table |
| `MAX_BEDS(c)` | 20 | 20 | 30 | beds | Crop table |

### Decision variables
| Name | Unit | Source |
|---|---|---|
| `q(TOM)`, `q(CAR)`, `q(MES)` | beds (integer) | Solver output |

## Structure
One workbook. Suggested sheets / regions:

- **Assumptions** — every named input above in its own labeled cell, entered
  once. Everything downstream references these by name; no input value is
  retyped.
- **Crop economics** — one block per crop: revenue, fertilizer cost, and season
  labor-hour demand as a function of that crop's own bed count `q`, plus the
  marginal (per-additional-bed) revenue, labor-hours, and cost figures used to
  sanity-check the solver.
- **Labor & cost roll-up** — sums labor hours across the three crops at the
  current allocation, converts the excess over `FARMER_HRS` into whole temporary
  workers and a labor cost, then adds fertilizer and fixed costs to reach total
  cost.
- **Allocation / Solver** — the three decision variables, the constraint set, and
  the `PROFIT` objective cell, arranged for Excel Solver.
- **Enumeration check** — a full sweep of every feasible `(q(TOM), q(CAR),
  q(MES))` combination and its profit, so the Solver result can be confirmed
  against the true maximum (see Validation).
- **Summary** — the solved allocation and every named output in one place.

## Calculation logic
Named-range notation. For crop `c` at that crop's own bed count `q`:

    REVENUE(c, q)   = q x PRICE(c)
    FERT_COST(c, q) = q x FERT_BED(c)
    LABOR_HRS(c, q) = q x LABOR_HRS_WK(c) x SEASON_WEEKS x (1 + DIM_PCT(c)) ^ q

Roll-up across the current allocation:

    TOTAL_LABOR_HRS = LABOR_HRS(TOM, q(TOM)) + LABOR_HRS(CAR, q(CAR)) + LABOR_HRS(MES, q(MES))
    TEMP_HRS_NEEDED = MAX(0, TOTAL_LABOR_HRS - FARMER_HRS)
    WORKERS_NEEDED  = ROUNDUP(TEMP_HRS_NEEDED / TEMP_WORKER_HRS, 0)
    LABOR_COST      = FARMER_WAGE + WORKERS_NEEDED x TEMP_WORKER_WAGE
    TOTAL_FERT_COST = FERT_COST(TOM, q(TOM)) + FERT_COST(CAR, q(CAR)) + FERT_COST(MES, q(MES))
    TOTAL_REVENUE   = REVENUE(TOM, q(TOM)) + REVENUE(CAR, q(CAR)) + REVENUE(MES, q(MES))
    PROFIT          = TOTAL_REVENUE - TOTAL_FERT_COST - LABOR_COST - FIXED_COSTS

Marginal figures, evaluated at each crop's current `q` (used to check the
optimum, not to drive it):

    MARG_REVENUE(c)   = PRICE(c)                                     [constant — price taker]
    MARG_LABOR_HRS(c) = LABOR_HRS(c, q(c)) - LABOR_HRS(c, q(c) - 1)
    MARG_COST(c)      = FERT_BED(c) + MARG_LABOR_HRS(c) x TEMP_WORKER_RATE

`MARG_COST` prices the marginal bed's extra hours at `TEMP_WORKER_RATE`, because
once `TOTAL_LABOR_HRS > FARMER_HRS` every additional hour is bought from a
temporary worker. `FARMER_RATE` is the shadow price of one of the farmer's own
720 hours: compare it against `MARG_COST(c) / MARG_LABOR_HRS(c)` to see whether a
marginal bed is worth the farmer's own time versus hired time.

Solver setup:

    Maximize:      PROFIT
    By changing:   q(TOM), q(CAR), q(MES)
    Subject to:
      q(TOM) + q(CAR) + q(MES)  <=  TOTAL_BEDS
      0 <= q(c) <= MAX_BEDS(c)                 for each crop c
      TOTAL_LABOR_HRS  <=  LABOR_HRS_CAP
      q(TOM), q(CAR), q(MES) integer
    Engine:  Evolutionary

## Conventions
- **Diminishing returns load onto labor hours, via `(1 + DIM_PCT(c)) ^ q`
  applied to the crop's whole season labor demand.** Adding a bed of a crop
  raises the attributed hours for every bed of that crop, read as
  crowding / disease-pressure / management burden that makes the crop more
  labor-intensive to run at scale. This is the form the engagement brief's
  mechanism assumes (tomatoes' 10%/bed compounding "outpaces the fixed $8,800
  price well before the cap") and the form carried in the case template.
  - *Rejected reading 1:* per-bed escalation summed as a geometric series,
    `base x (1 + rate) x ((1 + rate) ^ q - 1) / rate`. Closer to a literal "each
    marginal bed costs `(1 + rate)` more than the last," but not the form the
    template or brief use.
  - *Rejected reading 2:* decline applied to price or yield,
    `PRICE(c) x (1 - rate) ^ (i - 1)`. The committed brief fixes price
    regardless of quantity (price taker), so diminishing returns cannot touch
    revenue. An earlier abandoned model build used this reading; it is wrong for
    this engagement.
- **Revenue and fertilizer cost are linear in `q`.** `PRICE(c)` is fixed
  regardless of quantity (perfect competition); `FERT_BED(c)` is a flat per-bed
  cost. Diminishing returns touch neither.
- **Labor is costed in whole-worker blocks.** The farmer's 720 hours are applied
  first against `TOTAL_LABOR_HRS`; any remainder is covered by whole temporary
  workers at `TEMP_WORKER_WAGE` each, `ROUNDUP` — a worker costs the full
  $25,000 no matter how few of their 1,440 hours are used. No fractional
  workers.
- **Farmer wage and fixed costs are period costs.** Both enter `PROFIT` once, at
  full value, regardless of the crop mix — `FARMER_WAGE` is paid whether the
  farmer works 100 hours or 720.
- **Costing order:** variable, allocation-dependent costs first (fertilizer, then
  hired labor), then the period costs (`FARMER_WAGE`, `FIXED_COSTS`) last.
- **Beds are integers** everywhere — inputs, decision variables, enumeration.
- **Boundaries:**
  - At `q(c) = 0`: `REVENUE(c, 0) = FERT_COST(c, 0) = LABOR_HRS(c, 0) = 0`. The
    compounding term is never evaluated at `q = 0` for a live figure.
  - The marginal check starts at `q(c) = 1`, where `LABOR_HRS(c, 0) = 0`;
    `MARG_*(c)` at `q(c) = 0` is undefined and not used.
  - If `TOTAL_LABOR_HRS <= FARMER_HRS`, the marginal hour is the farmer's own
    (already paid) and its cash cost is 0, not `TEMP_WORKER_RATE`. At every
    allocation near the brief's hypothesis `TOTAL_LABOR_HRS` far exceeds 720, so
    `MARG_COST` uses `TEMP_WORKER_RATE`; flag it if a solved allocation falls
    below 720 total hours.
- **Caps bind even mid-margin.** If a crop's marginal profit is still positive at
  its `MAX_BEDS(c)`, the cap holds; the model never relaxes a cap to chase
  profit.
- **`WORKERS_NEEDED <= TEMP_WORKER_MAX` is the same constraint as
  `TOTAL_LABOR_HRS <= LABOR_HRS_CAP`** (6,480). Stating both is deliberate; they
  must agree.

## Validation rules
Structural:
- Every calculated cell is a formula referencing named ranges — no hardcoded
  numbers in any computed cell, no error values (`#DIV/0!`, `#REF!`, `#VALUE!`,
  ...) anywhere in the solved workbook.
- `q(TOM) + q(CAR) + q(MES) <= 64`.
- `0 <= q(c) <= MAX_BEDS(c)` for each crop.
- `TOTAL_LABOR_HRS <= 6,480` and `WORKERS_NEEDED <= 4` at the solved allocation;
  if `WORKERS_NEEDED > 4` the labor constraint was set up wrong.
- `TEMP_WORKER_WAGE / TEMP_WORKER_HRS` equals `TEMP_WORKER_RATE` (17.36) and
  `(FARMER_WAGE / 2) / FARMER_HRS` equals `FARMER_RATE` (34.72) — the derived
  rates are self-consistent with their sources.

Hand checks (do before trusting the model):
- `LABOR_HRS(c, 1) = LABOR_HRS_WK(c) x 36 x (1 + DIM_PCT(c))` for each crop.
- `LABOR_HRS(TOM, 20) / (20 x 90) = 1.10 ^ 20 ~= 6.73` — the brief's "~6.7x the
  first bed by bed 20" statement, reproduced from the model.
- Pick any allocation by hand, compute `PROFIT` independently, confirm the
  workbook matches.

Optimum check (acceptance):
- The Solver result must equal the maximum-profit row of the full enumeration of
  all feasible `(q(TOM), q(CAR), q(MES))` — at most `21 x 21 x 31 = 13,671`
  combinations, filtered to those satisfying every constraint. Solver alone is
  not sufficient evidence: `ROUNDUP` plus integer beds plus compounding make
  `PROFIT` non-smooth, so GRG Nonlinear and Simplex LP can stall at a local
  optimum. Use the Evolutionary engine and confirm against the enumeration.
- Compare the solved allocation to the committed brief's hypothesis
  (`10 TOM / 30 MES / 20 CAR`, tolerance ±3 beds per crop). This is a check, not
  a target — a mismatch outside ±3 is not automatically a defect but must be
  explainable from the constraints and marginal figures.
- The brief's three falsification conditions, checked against the solved result:
  1. If `q(TOM)` is at or within ~1 of `MAX_BEDS(TOM)` (20), the brief's core
     mechanism (10%/bed labor compounding outpacing the $8,800 price before the
     cap) is contradicted — report it.
  2. If `q(CAR) < MAX_BEDS(CAR)` (20) while `q(TOM) < MAX_BEDS(TOM)`, the brief's
     claim that carrots' shallow compounding and low fixed costs carry them to
     their cap is undercut — report it.
  3. If `q(MES) < MAX_BEDS(MES)` (30), the labor limits differ from what the
     brief assumed — report it.

## Outputs
- `q(TOM)`, `q(CAR)`, `q(MES)` — the profit-maximizing integer bed allocation.
- `WORKERS_NEEDED` — temporary workers to hire (0–4).
- `TOTAL_REVENUE`, `TOTAL_FERT_COST`, `LABOR_COST`, `PROFIT` — season totals, USD.
- `TOTAL_LABOR_HRS` and `LABOR_HRS_CAP - TOTAL_LABOR_HRS` — labor used and slack.
- Binding constraint at the optimum — one of: the 64-bed limit, `LABOR_HRS_CAP`,
  a crop's `MAX_BEDS(c)`, or "interior" — the answer to the Purpose question.
- `MARG_REVENUE(c)` and `MARG_COST(c)` at the solved `q(c)` for each crop — shows
  why the solver stopped (marginal bed no longer pays, cap reached, or labor
  exhausted).

## Audit findings
Not yet audited — no model has been built from this spec.
