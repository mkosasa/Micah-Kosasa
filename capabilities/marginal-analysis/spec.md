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
labor conventions are stated on the Stage 2 page. Shadow prices (added after the
Stage 2 build) trace to the Stage 3 page:
`https://adamwstauffer.github.io/ai-lms/case-perfect-competition-stage3.html`.

## Purpose
This model supports one decision: how many beds to plant of tomatoes, carrots,
and mesclun for a 36-week season, given fixed per-crop prices, fertilizer costs,
labor requirements, and bed caps, plus a fixed labor supply (the farmer's own
720 field hours and up to four temporary workers). It must report the integer
bed allocation that maximizes season profit, the season profit and its cost
breakdown at that allocation, which constraint stops the allocation where it
does, what one more bed of a capped crop would add to profit (its shadow price),
and — for each crop analyzed on its own — the bed count at which price meets
marginal cost. For the two crops that sit at their caps, it also shows where each
one's own standalone profit would peak if the cap were lifted (a what-if display;
the cap still binds). Two further what-ifs each get their own tabs: the farm with
every limit released (beds, temporary workers) to see the maximum-profit point
and how it compares with today's limits, and a fertilizer discount after a set
number of beds farm-wide, under today's limits and with limits released.

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
- **Farm-level marginal cost of the next bed** — for crop `c` at the current
  allocation, the fertilizer plus the labor cost of adding bed `q(c) + 1` to the
  whole farm, with the extra hours priced on top of `TOTAL_LABOR_HRS`
  (farmer hours first, then temporary). It differs from the standalone marginal
  cost above, which prices `c`'s hours as if `c` were the only crop.
- **Shadow price** — of a binding constraint, the increase in `PROFIT` from
  relaxing it by one unit with everything else held. Here it is a discrete
  one-bed step, not a continuous LP dual. For a crop's `MAX_BEDS(c)` cap it is
  `PRICE(c)` minus the farm-level marginal cost of the next bed. A non-binding
  constraint has shadow price 0.
- **Beyond-cap continuation** — the standalone marginal-cost schedule of a crop
  that sits at its cap (carrots, mesclun) carried past `MAX_BEDS(c)`, through the
  bed where `SA_PROFIT` peaks and at least 3 beds after it. A what-if display: it
  never changes `XING(c)`, the Solver, the Enumeration, or a shadow price.
- **XING_UNCAPPED(c)** — the bed count that maximizes `SA_PROFIT(c, q)` over the
  capped schedule and its continuation together: the crop's standalone
  profit-maximizing bed count if `MAX_BEDS(c)` did not apply.
- **Today's limits** — the constraints the model is built with: `MAX_BEDS(c)` per
  crop, `TOTAL_BEDS` on the farm, and `TEMP_WORKER_MAX` temporary workers (the
  temporary-hour ceiling).
- **Limits released** — none of those three. Extra temporary hours cost
  `TEMP_WORKER_RATE`, the same rate as today (no premium); `FARMER_HRS` still come
  first at `FARMER_RATE`. Prices, fertilizer costs, labor requirements,
  diminishing returns, and `FIXED_COSTS` are unchanged.
- **Stop bed** of crop `c`, limits released — the last bed whose farm-level
  marginal cost is below `PRICE(c)`. Beyond it every extra bed loses money. This
  is the maximum-*profit* rule, not maximum output.
- **Fertilizer discount** — fertilizer on each bed after the first
  `FERT_DISC_BEDS` beds farm-wide costs `(1 - FERT_DISC_PCT)` as much. Beds are
  counted across all crops together, and the discount lands on the
  highest-fertilizer-cost beds first.
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
| `LABOR_HRS_WK(c)` | 2.50 | 5/6 | 1.25 | hours per week per bed | Case scenario, crop table |
| `PRICE(c)` | 8,800 | 2,094 | 2,700 | USD per bed | Case scenario, crop table |
| `MAX_BEDS(c)` | 20 | 20 | 30 | beds | Case scenario, crop table |

`LABOR_HRS_WK(CAR)` is `5/6` (0.8333…), held at full precision in its own
cell — this is the value behind the Stage 2 published check figures. The crop
table prints it as `0.833`; an earlier draft of this spec took that literally,
which put season profit `$6.49` above the published check. See Audit findings.

### Decision variables
| Name | Unit | Source |
|---|---|---|
| `q(TOM)`, `q(CAR)`, `q(MES)` | beds (integer) | Solver output |

### Scenario inputs (fertilizer what-if)
| Name | Value | Unit | Source |
|---|---|---|---|
| `FERT_DISC_BEDS` | 40 | beds, farm-wide | Owner's what-if — the beds that pay full fertilizer price |
| `FERT_DISC_PCT` | 0.30 | fraction | Owner's what-if — discount on the fertilizer of each bed after that |

These two are entered on the `FertScenario` sheet, not `Inputs`, because only the
scenario sheets read them; the base model never does.

## Structure
One workbook. Sheets / regions, in this order:

- **Inputs** — every named input above in its own labeled cell, entered once.
  Everything downstream references these by name; no input value is retyped.
- **Crop economics** — one block per crop: revenue, fertilizer cost, and season
  labor-hour demand as a function of that crop's own bed count `q`.
- **Marginal-cost schedules** — one block per crop, standalone: total variable
  cost and the marginal cost of the `q`-th bed for `q = 1 .. MAX_BEDS(c)`, set
  against `PRICE(c)`, with `XING(c)` and `XING_FIRST(c)` reported. Below the
  three blocks, a **beyond-cap continuation** for carrots and mesclun: each
  block repeats its cap row, then adds beds past `MAX_BEDS(c)` through the
  `SA_PROFIT` peak and at least 3 beds after it (the schedule's columns plus a
  `marker` column reading `PEAK` / `after peak`), followed by `XING_UNCAPPED(c)`,
  the profit at the peak and at the cap, the gain, the beds shown after the peak,
  and a live PASS / FAIL that at least 3 are shown. Appended below so no
  existing row moves.
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
  Below the solved-allocation figures, a **shadow-price block**:
  - one row per crop — `q(c)`, `MAX_BEDS(c)`, whether the cap binds, the extra
    labor hours and extra labor cost of bed `q(c) + 1`, the farm-level marginal
    cost of that bed, `PRICE(c)`, `PRICE(c)` minus that cost, whether one more
    bed is feasible, and `SHADOW_PRICE(c)`;
  - one row each for the slack on the 64-bed total, the temporary-hour ceiling,
    and the implied temporary-worker count (limit, used, slack, binding?,
    shadow price);
  - one tie-out row (farmer-first labor cost against `TOTAL_LABOR_COST`).
  The three `SHADOW_PRICE(c)` cells are named ranges (`SHADOW_PRICE_TOM`,
  `SHADOW_PRICE_CAR`, `SHADOW_PRICE_MES`).
- **Unconstrained** — the farm with every limit released: a summary (stop bed per
  crop; beds against `TOTAL_BEDS`, against beds planted today, and against the sum
  of the caps; labor hours against `LABOR_HRS_CAP` and against today's plan; temp
  workers needed; profit against today's), a neighbor check, live PASS / FAIL
  checks, and a farm-level marginal-cost schedule for beds 0 to 55 per crop with
  the stop bed marked.
- **FertScenario** — the fertilizer what-if: the two scenario inputs; results with
  and without the discount under today's limits and with limits released (mix,
  profit, labor hours, temp workers, over / under against the labor and bed
  limits, discount applied); the marginal cost of the last and next bed before and
  after the discount; a sensitivity row for a carrots-first discount order;
  PASS / FAIL checks; and discounted marginal-cost schedules for beds 0 to 55.
- **FertEnum** — the enumeration grid behind FertScenario: every
  `(q(TOM), q(CAR), q(MES))` with `q(TOM)` 0–20, `q(CAR)` 0–36, `q(MES)` 0–52
  (41,181 rows), profit with and without the discount, a flag for combinations
  inside today's limits, and the four readouts (today's limits / limits released,
  each with no discount / discount).

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

### Beyond-cap continuation (carrots and mesclun)
For `c` in `{CAR, MES}` — the two crops that sit at their caps at the solved
allocation — continue the standalone schedule above for
`q = MAX_BEDS(c) + 1, MAX_BEDS(c) + 2, ...` with the same formulas (`LABOR_HRS`,
`FARMER_HRS_USED`, `TEMP_HRS_USED`, `LABOR_COST_SA`, `TVC`, `MARG_COST`,
`SA_PROFIT`, and the `q if MC >= PRICE` and `DIP` flags). The block's first row
repeats the cap row `q = MAX_BEDS(c)` by reference, so
`MARG_COST(c, MAX_BEDS(c) + 1) = TVC(c, MAX_BEDS(c) + 1) - TVC(c, MAX_BEDS(c))`
reads from it. `MAX_BEDS(c)` is not a bound on this block.

    XING_UNCAPPED(c)   = the q that maximizes SA_PROFIT(c, q) over the capped schedule
                         (q = 0 .. MAX_BEDS(c)) and the continuation together
    PEAK_PROFIT(c)     = MAX of SA_PROFIT(c, q) over that same range
    CAP_PROFIT(c)      = SA_PROFIT(c, MAX_BEDS(c))
    GAIN(c)            = PEAK_PROFIT(c) - CAP_PROFIT(c)
    BEDS_AFTER_PEAK(c) = (last q shown) - XING_UNCAPPED(c)
    marker(q)          = "PEAK" at q = XING_UNCAPPED(c); "after peak" for q above it;
                         the cap row is labelled "cap"

Extent: build the block out to at least `XING_UNCAPPED(c) + 3`. The built model
carries 5 beds after each peak, and a live cell reads PASS when
`BEDS_AFTER_PEAK(c) >= 3`. At the case inputs the crop's own hours already exceed
`FARMER_HRS` at the cap, so every extra hour costs `TEMP_WORKER_RATE`,
`MARG_COST(c, q)` keeps rising, and `SA_PROFIT` rises to one peak and then falls;
`XING_UNCAPPED(c)` is then the last bed with `MARG_COST(c, q) < PRICE(c)`.

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

### Shadow prices at the current allocation
For crop `c` at its current `q(c)`. `LABOR_HRS(c, q(c) + 1)` is computed inline
from the labor formula above — the per-crop tables stop at `MAX_BEDS(c)`, so a
table lookup would fail for a crop sitting at its cap.

    LABOR_COST_AT(H)    = MIN(H, FARMER_HRS) x FARMER_RATE
                        + MAX(0, H - FARMER_HRS) x TEMP_WORKER_RATE
    NEXT_LABOR_HRS(c)   = LABOR_HRS(c, q(c) + 1) - LABOR_HRS(c, q(c))
    NEXT_LABOR_COST(c)  = LABOR_COST_AT(TOTAL_LABOR_HRS + NEXT_LABOR_HRS(c))
                        - LABOR_COST_AT(TOTAL_LABOR_HRS)
    NEXT_MC(c)          = FERT_BED(c) + NEXT_LABOR_COST(c)
    NEXT_MARGIN(c)      = PRICE(c) - NEXT_MC(c)
    CAP_BINDING(c)      = q(c) >= MAX_BEDS(c)
    NEXT_FEASIBLE(c)    = q(TOM) + q(CAR) + q(MES) + 1 <= TOTAL_BEDS
                          AND TOTAL_LABOR_HRS + NEXT_LABOR_HRS(c) <= LABOR_HRS_CAP
    SHADOW_PRICE(c)     = IF(CAP_BINDING(c) AND NEXT_FEASIBLE(c), MAX(0, NEXT_MARGIN(c)), 0)

`NEXT_MARGIN(c)` is reported for every crop, including one whose cap does not
bind — it can be negative, which is how a crop that stops short of its cap shows
why. Only a crop at its cap can have a non-zero `SHADOW_PRICE(c)`.

Slack rows, at the current allocation (limit, used, slack = limit - used,
binding = slack <= 0):

    TOTAL_BEDS        limit TOTAL_BEDS                          used q(TOM) + q(CAR) + q(MES)
    Temp-labor hours  limit TEMP_WORKER_MAX x TEMP_WORKER_HRS   used TEMP_LABOR_HRS
    Temp workers      limit TEMP_WORKER_MAX                     used TEMP_LABOR_HRS / TEMP_WORKER_HRS

The temp-labor-hours and temp-workers rows are the same constraint stated in
hours and in workers. Shadow price is `0` when slack is positive; when a row is
binding the cell reports the text `binding - re-solve`, since valuing it needs a
re-optimization, not a one-bed step.

Tie-out: `LABOR_COST_AT(TOTAL_LABOR_HRS) = TOTAL_LABOR_COST`.

### Limits released (Unconstrained sheet)
For each crop `c`, at every bed `q = 1 .. 55`:

    MC_FARM(c, q)  = FERT_BED(c) + (LABOR_HRS(c, q) - LABOR_HRS(c, q - 1)) x TEMP_WORKER_RATE
    STOP(c)        = the largest q in 1 .. 55 with MC_FARM(c, q) < PRICE(c)      (0 if none)
    UNCON_Q(c)     = STOP(c)

    UNCON_HRS      = SUM over c of LABOR_HRS(c, STOP(c))
    UNCON_PROFIT   = SUM(STOP(c) x PRICE(c)) - SUM(STOP(c) x FERT_BED(c))
                     - LABOR_COST_AT(UNCON_HRS) - FIXED_COSTS

Why each crop can be solved on its own: when `UNCON_HRS >= FARMER_HRS`,
`LABOR_COST_AT(H)` equals `FARMER_HRS x FARMER_RATE + (H - FARMER_HRS) x
TEMP_WORKER_RATE`, which is affine in `H`. Profit is then a sum of one term per
crop, and every extra labor hour costs `TEMP_WORKER_RATE`. `MC_FARM` differs from
the standalone `MARG_COST` on `MCSchedules` (which gives each crop the farmer's
hours first) at early beds and agrees at and past the stop bed, so the stop beds
equal the standalone peaks: `STOP(TOM) = XING(TOM)` (when below its cap),
`STOP(CAR) = XING_UNCAPPED(CAR)`, `STOP(MES) = XING_UNCAPPED(MES)`. Do not add the
three standalone `SA_PROFIT` values: each charges its own first `FARMER_HRS` at
`FARMER_RATE`, while the farm pays that once.

Comparisons reported:

    beds:    SUM(UNCON_Q) vs TOTAL_BEDS, vs q(TOM)+q(CAR)+q(MES), vs SUM of MAX_BEDS(c)
    labor:   UNCON_HRS vs LABOR_HRS_CAP                              (over (+) / under (-))
             MAX(0, UNCON_HRS - FARMER_HRS) vs TEMP_WORKER_MAX x TEMP_WORKER_HRS
             temp workers = MAX(0, UNCON_HRS - FARMER_HRS) / TEMP_WORKER_HRS  vs TEMP_WORKER_MAX
             UNCON_HRS vs TOTAL_LABOR_HRS                            (today's plan)
    profit:  UNCON_PROFIT vs PROFIT

Neighbor check: recompute profit from the full cost formulas at `UNCON_Q` and at
each single-bed neighbor (one crop +1 or -1); no neighbor may beat the optimum.

### Fertilizer discount (FertScenario and FertEnum sheets)
For an allocation with `N = q(TOM) + q(CAR) + q(MES)`:

    OVER         = MAX(0, N - FERT_DISC_BEDS)
    FERT_DISC    = FERT_DISC_PCT x ( FERT_BED(TOM) x MIN(q(TOM), OVER)
                                   + FERT_BED(MES) x MIN(q(MES), MAX(0, OVER - q(TOM)))
                                   + FERT_BED(CAR) x MIN(q(CAR), MAX(0, OVER - q(TOM) - q(MES))) )
    PROFIT_DISC  = PROFIT + FERT_DISC

The order TOM, MES, CAR puts the discount on the highest-fertilizer-cost beds
first; it is exact only while `FERT_BED(TOM) >= FERT_BED(MES) >= FERT_BED(CAR)`
(a live check).

Enumeration (FertEnum): every integer allocation on `q(TOM)` 0–20, `q(CAR)` 0–36,
`q(MES)` 0–52. Per row: hours, `LABOR_COST_AT(hours)`, revenue minus fertilizer,
`FERT_DISC`, `PROFIT` and `PROFIT_DISC`, and a flag that is 1 when the row is inside
today's limits (`q(TOM) <= MAX_BEDS(TOM)`, `q(CAR) <= MAX_BEDS(CAR)`,
`q(MES) <= MAX_BEDS(MES)`, `N <= TOTAL_BEDS`, temporary hours at or below the
ceiling). Four readouts: the row maximizing `PROFIT` and `PROFIT_DISC` over (a) the
flagged rows (today's limits) and (b) all rows (limits released). The grid's axes
extend past every released optimum, and a live check requires each released
optimum to sit strictly inside the grid.

Marginal cost by bed comes from profit differences at each view's discounted
optimum, with and without the discount (`e_c` is one bed of crop `c`):

    MC_LAST(c) = PRICE(c) - ( PROFIT(q) - PROFIT(q - e_c) )
    MC_NEXT(c) = PRICE(c) - ( PROFIT(q + e_c) - PROFIT(q) )

Schedules with the discount hold the other two crops at the view's solved
allocation:

    MC_DISC(c, q) = MC_FARM(c, q) - ( FERT_DISC(c at q) - FERT_DISC(c at q - 1) )

and agree with the profit-difference values at the solved allocation.

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

### Shadow prices
- **One extra bed, everything else held.** A shadow price here is the profit
  change from adding one bed of the capped crop, with the other crops' beds
  fixed. It is not a continuous LP dual, and it does not hold for a second extra
  bed — each further bed costs more because the `(1 + DIM_PCT(c)) ^ q` factor
  compounds. Beds are not reallocated from other crops.
- **Farm-level marginal costing.** The extra hours are priced at the rate of the
  marginal source: farmer hours first while any remain, then
  `TEMP_WORKER_RATE`. Not `BLENDED_RATE` (an average, not a marginal cost) and
  not the standalone `MARG_COST(c, q)` (it prices the crop as if it were alone,
  charging its hours against the farmer's 720 that the farm has already used).
  For the case inputs the farmer's hours are fully used at the optimum
  (`TOTAL_LABOR_HRS > FARMER_HRS`), so every extra hour costs `TEMP_WORKER_RATE`;
  the formula still handles allocations where they are not.
- **Not the `Summary` "MARG_COST at q(c)" column.** That column is the
  standalone marginal cost of the last bed already planted. `PRICE(c)` minus it
  is the margin on that last bed, not the value of relaxing the cap.
- **Floors and gates.** `SHADOW_PRICE(c)` is floored at 0 and is 0 unless the cap
  binds and one more bed is feasible under the 64-bed total and the
  labor-hour ceiling. No error values; no lookup into a schedule.
- **Binding total-bed or temp-hour constraint** — reported as the text
  `binding - re-solve`, not computed.

### Beyond-cap continuation
- **A what-if, and standalone.** Each crop is modeled as the only crop: all of
  `FARMER_HRS` to itself, beds free, no 64-bed total, no temporary-hour ceiling.
  The block answers "where would this crop stop on its own if the cap did not
  apply," not "plant this many." At the standalone peaks (26 carrot beds, 37
  mesclun beds) alongside 10 tomato beds the farm would need 73 beds against 64,
  and the 720 farmer hours cannot serve both crops.
- **Display only.** Nothing reads the continuation: `XING(c)` and `XING_FIRST(c)`
  still cover `q = 0 .. MAX_BEDS(c)`, and the Solver, the Enumeration, the
  shadow-price block and the Checks are unchanged. "Caps bind even mid-margin"
  (Scope) still holds — the block shows what a cap costs; it does not lift it.
- **Relation to the shadow price.** Where the crop's own hours already exceed
  `FARMER_HRS` at the cap (true at the case inputs), the standalone
  `MARG_COST(c, MAX_BEDS(c) + 1)` equals `NEXT_MC(c)` in the shadow-price block,
  so `PRICE(c)` minus it equals `SHADOW_PRICE(c)`. The shadow price is the value
  of the first extra bed; the continuation shows the whole path to the peak.
- **Placement.** Appended below the existing schedule blocks, so no existing row
  moves and every existing reference, named range and check is unchanged.

### Limits released and the fertilizer scenario
- **What-ifs, display only.** Nothing in the base model reads the three new
  sheets. `Enumeration`, the Solver setup, `Summary`, `Checks`, and the shadow
  prices are unchanged.
- **Released means released everywhere** — per-crop caps, the farm bed total, and
  the temporary-worker limit — with extra temporary hours at the same
  `TEMP_WORKER_RATE`. Hiring past four workers may cost more in practice; a
  premium would lower the released result.
- **Maximum profit, not maximum output.** Past the stop bed each extra bed raises
  output and loses money.
- **"After 40 beds" counts the whole farm** (the owner's choice), not each crop's
  own bed count. The per-crop reading was not built: no crop's best bed count
  reaches 40 (10 / 26 / 37 released), so it would change nothing.
- **Discount order and its side effect.** The discount sits on the
  highest-fertilizer-cost beds, so while `OVER <= q(TOM) + q(MES)` one more bed of
  *any* crop pulls another top-cost bed into the discounted range: every crop's
  marginal cost falls by `FERT_DISC_PCT x FERT_BED(TOM)` (30% of 880 = 264), even
  carrots, whose own fertilizer is 440. A different order gives different
  numbers; the `FertScenario` sensitivity row shows the mix under a carrots-first
  order.
- **Grid bounds are not a constraint.** The FertEnum axes are wide enough that
  every released optimum sits inside them; the live interior check fails if a
  future input change pushes one to an edge, and the axes must then be widened.
- **Labor.** The released mixes are compared with `LABOR_HRS_CAP` (today's labor
  limit) and with the labor today's plan uses; a positive over / under figure means
  the released plan needs more labor than today's limit allows.

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
- `LABOR_COST_AT(TOTAL_LABOR_HRS)` equals `TOTAL_LABOR_COST` (the tie-out row in
  the shadow-price block — it validates the farmer-first cost function the
  shadow prices use).
- `SHADOW_PRICE(c) >= 0` for every crop, and `> 0` only where `CAP_BINDING(c)`.
- No error values in the shadow-price block.
- Beyond-cap continuation: its first row equals the cap row of the schedule above;
  `MARG_COST` and `SA_PROFIT` at every bed match a from-scratch calculation;
  `SA_PROFIT` is highest at `XING_UNCAPPED(c)` and lower at the next bed; at least
  3 beds are shown after the peak (a live PASS / FAIL cell); and `XING(c)`,
  `XING_FIRST(c)`, `Summary` and `Checks` are unchanged by it.
- Limits released (`Unconstrained`), as live PASS / FAIL cells: total hours
  `>= FARMER_HRS`; at least 3 beds shown after each stop bed; the next bed's
  marginal cost at or above price for every crop; no single-bed neighbor beats the
  optimum and both profit routes agree; the stop beds agree with `XING(TOM)`,
  `XING_UNCAPPED(CAR)`, `XING_UNCAPPED(MES)`; and the result equals the FertEnum
  limits-released, no-discount readout.
- Fertilizer scenario (`FertScenario`, `FertEnum`), as live PASS / FAIL cells: the
  today's-limits, no-discount readout reproduces the model (mix, `PROFIT`, the
  `Enumeration` maximum); the limits-released, no-discount readout reproduces
  `Unconstrained`; the released optima sit strictly inside the grid; the grid has
  every combination; the fertilizer cost order holds; the discounted schedules
  agree with the profit-difference marginal costs; and discounted profit is never
  below undiscounted profit at the same mix.
- The three new sheets change nothing that existed: the original sheets are
  byte-identical, `Checks` still reads 31 PASS, `XING`, `PROFIT`, and the shadow
  prices are unchanged.

Hand check — the `q = 1` exponent guard:
- `LABOR_HRS(TOM, 1) = 1 x 2.50 x 36 x 1.10 = 99 hours`, exactly. Repeat for each
  crop with its own `DIM_PCT(c)`. A result of `90` (i.e. `x 1.00`) means the
  `(1 + dim%)^q` exponent was dropped — the most common structural defect.

Hand check — the shadow price. For a crop at its cap, recompute `PROFIT` from the
Inputs formulas with `q(c) + 1` beds of that crop and every other bed count
unchanged. `PROFIT(q(c) + 1) - PROFIT(q(c))` must equal `NEXT_MARGIN(c)` and, when
the cap binds and the extra bed is feasible, `SHADOW_PRICE(c)`. This uses the
profit expression directly, independent of the block's own formulas.

Worked example — the model must reproduce this table at `q = (5, 5, 5)`, an
arbitrary non-optimal allocation chosen only as a numeric anchor. Figures shown
to the cent; the workbook carries full precision.

| Figure | Value |
|---|---|
| `LABOR_HRS(TOM, 5)` | 724.7295 hrs |
| `LABOR_HRS(CAR, 5)` | 169.7112 hrs |
| `LABOR_HRS(MES, 5)` | 239.4185 hrs |
| `TOTAL_LABOR_HRS` | 1,133.8592 hrs |
| `FARMER_HRS_USED` | 720.0000 hrs |
| `TEMP_LABOR_HRS` | 413.8592 hrs |
| `TOTAL_LABOR_COST` | 32,185.06 |
| `BLENDED_RATE` | 28.385407 USD/hr |
| `ALLOC_LABOR_COST(TOM)` | 20,571.74 |
| `ALLOC_LABOR_COST(CAR)` | 4,817.32 |
| `ALLOC_LABOR_COST(MES)` | 6,795.99 |
| `TOTAL_REVENUE` | 67,970 |
| `TOTAL_FERT_COST` | 11,000 |
| `FIXED_COSTS` | 20,000 |
| `PROFIT` | 4,784.94 |

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
- Shadow prices (added for Stage 3; named `SHADOW_PRICE_TOM`,
  `SHADOW_PRICE_CAR`, `SHADOW_PRICE_MES`) — `SHADOW_PRICE(c)` for each crop, with
  `NEXT_LABOR_HRS(c)`, `NEXT_MC(c)`, and `NEXT_MARGIN(c)` alongside; the slack and
  binding status of the 64-bed total, the temporary-hour ceiling, and the implied
  temporary-worker count; and the labor-cost tie-out. Valid for one extra bed
  only.
- Beyond-cap continuation (added; carrots and mesclun; named `XING_UNCAPPED_CAR`,
  `XING_UNCAPPED_MES`) — `XING_UNCAPPED(c)`, the standalone `SA_PROFIT` at the peak
  and at the cap, the gain from lifting the cap, and the beds shown after the
  peak. A standalone what-if, not a recommendation to exceed a cap.
- Limits released (`Unconstrained`; named `UNCON_Q_TOM`, `UNCON_Q_CAR`,
  `UNCON_Q_MES`, `UNCON_PROFIT`) — the stop bed per crop and in total; the
  comparisons with `TOTAL_BEDS`, beds planted today, and the sum of the caps;
  labor hours required against `LABOR_HRS_CAP` (over / under), against the
  temporary-hour ceiling, against `TEMP_WORKER_MAX`, and against today's plan;
  profit against today's.
- Fertilizer scenario (`FertScenario`; inputs `FERT_DISC_BEDS`, `FERT_DISC_PCT`) —
  the four optima (today's limits and limits released, each with and without the
  discount) with mix, profit, labor hours, temp workers, and over / under against
  the labor and bed limits; the discount applied; the marginal cost of the last and
  next bed before and after the discount; and a carrots-first sensitivity.

## Audit findings
Added after the model is built. For each check: what was checked, what was
found, what was done about it. At least three checks, including the `q = 1`
exponent guard, the `q = (5, 5, 5)` worked-example reconciliation, and the Farm
Profit Lab reconciliation. Compare the built model's optimal mix, season profit,
and standalone crossing points against the Stage 2 case page's published check
figures and record any variance.

Example entry — At `q = 1`, `LABOR_HRS(TOM, 1)` returned 99 hours; hand
calculation `1 x 2.50 x 36 x 1.10` = 99 hours — PASS.

### Method

No Excel was available on the build machine, so `model.xlsx` was hand-authored
as OOXML and the checks below were run outside Excel: a PowerShell
reimplementation of this spec's formulas, plus direct inspection of every
worksheet cell (formula text and cached value). Cached values carry
`fullCalcOnLoad`, so Excel recomputes on open. What still needs Excel — opening
without a repair prompt, recalc reproducing the cached figures, and running
Solver — is flagged per check.

### Findings

- **`q = 1` exponent guard.** Checked `LABOR_HRS(c, 1)` for all three crops
  against the hand calculation `1 x LABOR_HRS_WK(c) x 36 x (1 + DIM_PCT(c))^1`,
  confirmed each differs from the dropped-exponent value `1 x LABOR_HRS_WK(c) x
  36`, and confirmed the live formula text contains `^`.

  | crop | hand = with `^1` | dropped-exponent | workbook (`CropEconomics`) |
  |---|---|---|---|
  | TOM | 99.0000 | 90.0000 | 99.0000 (`D6`) |
  | CAR | 30.7500 | 30.0000 | 30.7500 (`D30`) |
  | MES | 45.5625 | 45.0000 | 45.5625 (`D54`) |

  The `Checks` sheet's own guard cells (`D15:D20`) read PASS. — PASS.
- **`q = (5, 5, 5)` worked example.** The workbook reproduces the reconciliation
  table by live formula at the anchor allocation; both tie-outs
  (`SUM(ALLOC_LABOR_COST) = TOTAL_LABOR_COST`, `SUM(MARGIN) = PROFIT +
  FIXED_COSTS`) close to zero. Every figure matches the anchor table to the
  cent; `PROFIT` at the anchor is `4,784.94` (was `4,786.12` before the `5/6`
  change). — PASS.
- **Independent-implementation cross-check.** The Farm Profit Lab web calculator
  itself was not consulted (built from `spec.md` alone, by decision); its role
  as a second, independently-built implementation was filled by the PowerShell
  reimplementation. It agrees with the workbook's cached cells not only on the
  endpoints (`PROFIT` `42,761.66`, `TOTAL_LABOR_HRS` `5,277.2161`,
  `TOTAL_LABOR_COST` `104,118.3353` at the optimum) but on intermediate
  marginal costs: `MARG_COST(TOM, 10) = 8,248.59`, `MARG_COST(CAR, 20) =
  1,688.95`, `MARG_COST(MES, 30) = 2,420.10`, and `MARG_COST(TOM, 6) =
  4,906.28` (the tomato dip). Largest disagreement `1.5e-11`. — PASS for
  self-consistency and spec-conformance; the check against the Farm Profit Lab
  itself is recorded below.
- **Farm Profit Lab reconciliation (owner's manual audit).** Micah entered a
  mix into the Farm Profit Lab web calculator (linked from the Stage 2 case
  page) and compared its output to this model. Test point: `q(TOM)=7,
  q(CAR)=9, q(MES)=4` — a spot check away from the optimum, not `(10,20,30)`.

  | Figure | Farm Profit Lab | This model, recomputed from the spec's formulas |
  |---|---|---|
  | PROFIT | 14,653.66 | 14,653.66 |
  | TOTAL_LABOR_COST | 42,952 | 42,952.34 |

  `(7,9,4)` isn't a combination the built workbook stores directly, so the
  right-hand column was computed independently here from `LABOR_HRS`, the
  farmer-then-temp blended rate, and `FIXED_COSTS`, not read off a cell.
  Matches to the cent on profit; labor cost matches to the precision Micah
  reported. PASS. Closes the "Farm Profit Lab itself" gap left open by the
  independent-implementation cross-check above.
- **Two Solver starting points.** Excel Solver (GRG Nonlinear) was not run.
  Proxy: an integer hill-climb (violation-descent, then profit-ascent over
  feasible ±1 neighbours) from the two spec-mandated starts. `(0, 0, 0)` →
  `(10, 20, 30)` at `42,761.66`; `(20, 0, 0)` → `(10, 20, 30)` at `42,761.66`
  (note `(20, 0, 0)` is itself infeasible — 20 tomato beds alone blow the
  5,760 temporary-hour ceiling — so a real Solver would repair it first). Both
  starts agree with each other and with the Enumeration maximum over all 9,726
  feasible integer combinations, which is what populates the decision cells and
  is authoritative per this spec. — PASS (proxy); the Solver run itself is
  outstanding.
- **Stage 2 published check figures.** Season profit: model `42,761.66` vs
  published `42,762` — reconciles within `$0.34` after the `LABOR_HRS_WK(CAR) =
  5/6` change (see below). Optimal mix: model `q(TOM)=10`, `q(CAR)=20`,
  `q(MES)=30` — matches the brief hypothesis and the Enumeration maximum.
  Standalone `PRICE = MC` crossings, as the model computes them:
  `XING_FIRST` (first bed with `MC >= PRICE`) `= 11 / 11 / 7` for TOM / CAR /
  MES; `XING` (profit-maximizing bed count) `= 10 / 20 / 30`.

  **Head-to-head against the Stage 2 case page's own published figures**
  (`case-perfect-competition-stage2.html`, retrieved as raw page text, not a
  summary): optimal mix "Tomatoes 10 · Carrots 20 · Mesclun 30 (60 beds)" —
  exact match. Season profit "$42,762" — matches within $0.34 (above).
  Standalone P ≈ MC points: "Tomatoes ~10 · Carrots ~10 · Mesclun ~6 beds" —
  each one less than this model's `XING_FIRST` (11 / 11 / 7), consistent with
  the page describing the last bed still below price rather than the first
  bed at or above it; same crossing, opposite side.

  Carrot's and mesclun's single published number hides something the model's
  full schedule shows and the page's summary doesn't: both standalone curves
  cross price **twice**, not once. Checked directly against `MCSchedules`:
  carrot's MC rises above its $2,094 price from bed 11 ($2,140.11) through
  bed 16 ($2,552.10), then the farmer-to-temp wage switch drops it back under
  price at bed 17 ($1,670.90), where it stays through the bed-20 cap
  ($1,688.95) and beyond. Mesclun does the same: above its $2,700 price from
  bed 7 ($2,710.71) through bed 13 ($2,988.40), then back under at bed 14
  ($2,522.58) through bed 19+ ($2,089.05). `XING_FIRST` reports only the
  first of these two crossings — bed 11 for carrots, bed 7 for mesclun — which
  is not the point where either crop actually stops being planted (the
  bed-20/bed-30 caps still sit on the second, profitable side of the dip).
  The published check's "~10 / ~6" is correct as far as it goes, but a reader
  who took it as "carrots and mesclun stop paying off around there" would be
  wrong; that misses the dip entirely, and the dip is why the caps still bind
  on the profitable side. — PASS on all three published figures; this
  two-crossing behavior generalizes the tomato dip Stage 3 already covers to
  carrots and mesclun as well, previously unnoted in this spec.
- **Season profit vs. published check figure (root cause).** The first build,
  reading `LABOR_HRS_WK(CAR)` as the printed `0.833`, computed `42,768.49` —
  `$6.49` above the published `42,762`. Traced: recomputing with
  `LABOR_HRS_WK(CAR) = 5/6` (0.8333…) gives `42,761.66`, which rounds to the
  published figure; the published figures use `5/6`. Resolved by changing the
  spec's `LABOR_HRS_WK(CAR)` to `5/6` (held exact) and rebuilding.
- **Live formulas, not pasted values.** Rebuilt the workbook with `DIM_PCT(TOM)`
  perturbed `0.10 → 0.12` and diffed cached values: every probed downstream
  figure moved — `LABOR_HRS(TOM, 1)` `99 → 100.8`, `MARG_COST(TOM, 10)`
  `8,248.59 → 10,412.46`, `TOTAL_LABOR_HRS` `5,277.22 → 5,738.11`, `PROFIT`
  (`CostStructure`, `Summary`) `42,761.66 → 34,760.01`, `WorkedExample` `PROFIT`
  `4,784.94 → 3,598.76`, and the Enumeration optimum shifted `(10, 20, 30) →
  (8, 20, 30)`. A separate sweep of all 197 distinct formula shapes confirmed
  every computed cell is a live formula referencing named ranges, with no
  hard-coded parameter value in any computed cell. — PASS; the in-Excel
  "change an input, watch it propagate" spot-check is still worth doing on open.
- **File integrity.** The Excel file required repair when first opened. Cause:
  on the MCSchedules sheet the `PRICE` and `MARG_REVENUE` columns (I, J) were
  written after the later columns K–M, and OOXML requires cells in ascending
  column order within a row, so Excel discarded those 146 cells on load. The
  rest of the workbook loaded intact. Fixed by emitting the columns in order;
  the generator now also validates column order on every sheet. Corrected
  `model.xlsx` rebuilt.
- **Shadow-price block (Stage 3 addition).** Added to `Summary` (rows 32–45) by
  direct OOXML edit after the audit above; no Excel was available. Checked
  outside Excel:
  1. *From-scratch profit differential.* `PROFIT` recomputed straight from the
     spec's inputs with one more bed of a crop (others fixed), minus `PROFIT` at
     the optimum `(10, 20, 30)`: TOM `-590.72`, CAR `352.49`, MES `246.47` —
     equal to the block's `PRICE - MC(q+1)` column. `SHADOW_PRICE`: TOM `0` (cap
     not binding), CAR `352.49`, MES `246.47`.
  2. *Stored formulas re-evaluated.* The formula text of all 48 new formula cells,
     read back from the saved file and evaluated against the workbook's cached
     inputs, reproduces every cached value.
  3. *Input perturbation.* `PRICE(CAR)` +100 → `SHADOW_PRICE(CAR)` +100;
     `FERT_BED(MES)` +50 → `SHADOW_PRICE(MES)` −50; `MAX_BEDS(CAR)` 20 → 25 → cap
     no longer binds, `SHADOW_PRICE(CAR)` 0; `PRICE(MES)` 2,700 → 2,400 →
     `SHADOW_PRICE(MES)` 0 (marginal cost above price); `TOTAL_BEDS` 64 → 60 →
     bed slack 0, binding, the extra bed reads BLOCKED, shadow prices 0. Direct
     inputs only — upstream cached values were held.
  4. *Nothing else moved.* Every pre-existing cell: 0 formula diffs, 0 value
     diffs, 0 dropped; named ranges 47 → 50.
  5. *Stage 3 page.* Shadow prices `$352` (carrots) and `$246` (mesclun), and the
     tomato marginal cost of bed 11 (`$9,391`), reproduce: `352.49`, `246.47`,
     `9,390.72`.

  Finding: the existing `Summary` "MARG_COST at q(c)" column (`8,248.59` /
  `1,688.95` / `2,420.10`) is the standalone marginal cost of the last planted
  bed. `PRICE` minus it (`551.41` / `405.05` / `279.90`) is that bed's margin, not
  a shadow price — the shadow price uses the farm-level cost of the *next* bed.
  — PASS outside Excel. Outstanding: open in Excel (the workbook is set to
  recalculate on open; the stale `calcChain` was removed so Excel rebuilds it)
  and confirm the block recalculates to the same figures without a repair prompt.
- **Beyond-cap continuation (carrots and mesclun).** Appended below the existing
  schedules on `MCSchedules` (rows 105–152) by direct OOXML edit; no Excel was
  available. Checked outside Excel:
  1. *Nothing else moved.* Every pre-existing cell in all 9 sheets, including the
     capped `XING` rows, `Summary` and `Checks`: 0 formula diffs, 0 value diffs, 0
     dropped. `XING` still `10 / 20 / 30`, the shadow prices still `352.49` /
     `246.47`, `Checks` still 31 PASS. Named ranges `50 → 52`
     (`XING_UNCAPPED_CAR`, `XING_UNCAPPED_MES`). Only `sheet3.xml` and
     `workbook.xml` differ from the prior file.
  2. *From-scratch cross-check.* `TVC`, `MARG_COST` and `SA_PROFIT` at every new
     bed, recomputed straight from the spec's inputs, agree with the workbook
     (largest gap `2.9e-11`). The stored text of all 339 new formulas, read back
     from the saved file, re-evaluates to its cached value.
  3. *Result.* Carrots: `SA_PROFIT` peaks at bed `26` (`4,770.87`, against
     `3,511.08` at the cap — a standalone gain of `1,259.79`); marginal cost first
     reaches price at bed `27` (`2,097.81` against `2,094`); profit then falls
     every bed, `4,767.06` at 27 to `4,060.35` at 31. Mesclun: peak at bed `37`
     (`9,067.15` against `8,077.81` at the cap, gain `989.34`); marginal cost first
     reaches price at bed `38` (`2,704.73` against `2,700`); profit falls to
     `8,652.03` at bed 42. Five beds are shown after each peak; the required
     minimum is 3.
  4. *Tie to the shadow prices.* Standalone `MARG_COST` of bed 21 (carrots) and
     bed 31 (mesclun) is `1,741.51` and `2,453.53`, equal to `NEXT_MC` in the
     shadow-price block — as expected where the crop's own hours already exceed
     `FARMER_HRS`.

  These are standalone figures: they ignore the 64-bed total and the other
  crops, so they are not a recommendation to exceed a cap. — PASS outside Excel.
  Outstanding: open in Excel and confirm the block recalculates to the same
  figures without a repair prompt.
- **Limits-released and fertilizer-scenario sheets.** Added `Unconstrained`,
  `FertScenario`, and `FertEnum` by direct OOXML edit; no Excel was available.
  Checked outside Excel:
  1. *Nothing else moved.* All 9 original sheet parts, `sharedStrings.xml`, and
     `styles.xml` are byte-identical to the prior file; only `workbook.xml`, the
     content types, the relationships, and `app.xml` changed to register the new
     sheets and six new names (52 → 58, alphabetical).
  2. *Independent recompute.* Every one of the 41,181 grid rows (hours, profit,
     discount, profit with discount, feasibility flag) matches from-scratch code
     (largest gap 0). The four readouts equal an independent brute force, and a
     wider search (`q(TOM)` < 26, `q(CAR)` < 48, `q(MES)` < 70) returns the same
     released optima. The 2,016 stored formulas on the two small sheets and the
     readouts re-evaluate to their cached values; the 12 last / next-bed
     marginal-cost cells and the 330 discounted schedule cells match profit
     differences.
  3. *Results.*

     | | Today's limits | Limits released |
     |---|---|---|
     | No discount | 10 / 20 / 30, 60 beds, `42,761.66` (reproduces the model) | 10 / 26 / 37, 73 beds, `45,010.80` |
     | Discount, 30% after 40 beds | 10 / 20 / 30, `48,041.66` (`+5,280`) | 10 / 30 / 44, 84 beds, `55,336.48` (`+10,325.68`) |

  4. *Labor and beds, limits released, no discount.* 6,453.11 hours against 6,480
     available: `26.89` hours **under** (3.981 of 4 temporary workers), and
     `+1,175.90` hours over today's plan; `+9` beds over `TOTAL_BEDS`, `+13` over the
     60 planted, `+3` over the sum of the caps; `+2,249.13` profit. Labor was not
     what stopped today's plan (`MAX_BEDS` caps on carrots and mesclun did). With
     the discount the released mix needs 7,642.34 hours — `1,162.34` **over** the
     limit (4.81 workers) — and 84 beds, `+20` over `TOTAL_BEDS`.
  5. *Sensitivity.* With the discount on carrot beds first instead, the same mixes
     earn `45,401.66` (today's limits) and `51,376.48` (released), against
     `48,041.66` and `55,336.48`.

  Outstanding: open in Excel and confirm the workbook recalculates (including the
  41,181-row grid) to the same figures without a repair prompt. The workbook is
  now about 9.9 MB, most of it the grid.
- **Excel save and saved Solver settings (owner's manual audit).** Micah opened
  the workbook in Excel and saved it; the save is recorded as its own commit.
  1. *Excel recalculation.* The saved file has the same 12 sheets and 58 names as
     the model built here, and a cell-by-cell comparison found 0 formula / text
     differences and 0 cached-value differences across all 12 sheets (about 1.0
     million cells, including the 535,000-cell `FertEnum` grid); every check cell
     still reads PASS. The workbook was set to recalculate on open, so this settles
     the "recalculates to the same figures" part of the *Outstanding* notes above
     for the shadow-price block, the beyond-cap rows, and the three new sheets.
     Whether Excel showed a repair prompt was not recorded.
  2. *Saved Solver settings.* Micah's earlier Excel save (PR #15) carries Solver
     settings, now copied into this workbook exactly as saved (15 hidden names).
     On `Optimization`: maximize the objective `Optimization!$B$10` (`PROFIT`),
     GRG Nonlinear, changing cell `Optimization!$B$4` only, **no constraints**,
     variables assumed non-negative. On `MCSchedules`: an objective of
     `MCSchedules!$R$7` (an empty cell), same engine, no constraints. The decision
     cells read `10 / 20 / 30` and `Optimization!B30` reads "solved = enumeration
     max".
  3. *What that does and does not show.* These settings are not the setup in
     "Solver setup" above (three changing cells, nine explicit constraints,
     integer, two starting points), so they do not complete the two-start Solver
     check, which remains open. The `MCSchedules`-scoped settings look like a stray
     Solver open. They are kept as saved; delete or complete them when the Solver
     check is rerun.

  Outstanding: run Solver with the full setup above from both starting points, and
  note whether Excel showed a repair prompt.
- **Missing cached values (grading review finding).** A Stage 1.2 grading review
  found that the committed workbook had 67,180 formula cells with no cached
  value (`MCSchedules` 160, `Enumeration` 3,945, `Unconstrained` 165, `FertEnum`
  62,910), and that `calcPr` did not carry `fullCalcOnLoad` despite the README
  claiming both: the workbook opened blank in those regions for anyone who had
  not built it. Confirmed independently: 67,180 missing, exact match.
  1. *Fix.* Outside Excel, recomputed every one of the workbook's 593,258
     formula cells from the stored formula text (a general evaluator plus a
     compiled fast path for the two large grids), cross-checked against the
     526,078 cells that already had a cached value (0 mismatches, 0 evaluation
     errors, tolerance 1e-6 relative), then patched only the 67,180 missing
     cells directly in the XML. Every one of them resolves to text, not a
     number, and Excel had already written the correct `t="str"` attribute and
     a self-closed empty `<v/>` on each, so the patch simply replaced `<v/>`
     with the computed text, touching nothing else: no `<f>` tag (explicit or shared),
     no already-cached value, `sharedStrings.xml`, and `styles.xml` are all
     byte-identical to the pre-fix file; only `workbook.xml` (for `calcPr`) and
     the 4 affected sheet parts changed. `calcPr` now carries
     `fullCalcOnLoad="1"`.
  2. *Verification.* Re-audited the rebuilt file from scratch: 0 missing values
     remain (all 12 sheets); package integrity (zip clean, every XML part
     well-formed, 12 sheets and 58 named ranges unchanged); every formula-cell
     count per sheet unchanged; every `<f>` tag identical to the pre-fix file,
     cell by cell. Re-read, fresh, from the written file: `Checks!B36` = "ALL
     PASS", `C36` = 31; `Summary!K36/K37/K38` = 0 / 352.4948 / 246.4738;
     `MCSchedules!E124/E147` (`XING_UNCAPPED`) = 26 / 37; `Unconstrained!E20` =
     45,010.7958; `CostStructure!B21` (`PROFIT`) = 42,761.664682745;
     `FertEnum!S3` = 55,336.4796: the model's own published figures and every
     figure the grading review cited, reproduced from the file as committed,
     not from a build script's memory. PASS. `README.md` corrected in the
     same commit.
  3. *Left alone.* The Solver setup (previous entry) is unchanged by this fix
     and remains partial; the two-start run is still outstanding, in Excel, by
     the owner. The `Checks` sheet's `SUMPRODUCT(--ISERROR(...))` ranges
     (`MCSchedules!A1:M110`, `Summary!A1:H30`) predate the beyond-cap rows
     (`MCSchedules` 111–152) and the shadow-price block (`Summary` 32–45), so
     those additions sit outside every error sweep (flagged, not widened; that
     would be a scope change the owner has not asked for).
