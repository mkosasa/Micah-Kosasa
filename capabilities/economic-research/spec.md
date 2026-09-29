---
type: spec
capability: economic-research
engagement: research-paper
date: 2026-09-28
status: draft            # draft | built | audited
built_with: "Claude Code, from this file"
---

# Economic research: model specification

This spec replaces the one written for the superseded brief (still in git history). It serves the problem-first brief at `docs/briefs/research-brief.md`. Decisions behind it are recorded in my decision tree v6. Every value below is a draft until I verify it against its source after the build.

## Purpose

The model supports the Healthcare Association of Hawaii's decision on which option, if any, to pursue to shorten the time patients wait in acute hospital beds for a place in a nursing facility: waiting on the 2024 rate reset, a targeted Medicaid add-on, a hospital-paid top-up or new transitional capacity. It must answer how far the statewide waitlist, across all levels of care, runs past an average of 14 days (in days per patient, excess days and beds occupied on an average day), what the excess costs hospitals net of Medicaid's waitlisted payment, and what the three tests show, which narrows the options but cannot separate the add-on from the top-up.

## Sources

| Source | Used for | Where |
|---|---|---|
| SHPDA Healthcare Utilization Reports, Table 18 (waitlisted patients in acute care beds), 2017 to 2024 | Yearly waitlisted patients and days; Dec 31 counts by level of care and by reason; county and hospital rows (2023, 2024) | 2017: https://health.hawaii.gov/shpda/files/2018/10/Table-18-Wait-listed-patients-acute-2017.pdf ; 2018: https://health.hawaii.gov/shpda/files/2019/11/2018UR-Table-18-Wait-Listed-Patients-in-Acute-Care-Beds-1.pdf ; 2019: https://health.hawaii.gov/shpda/files/2025/06/2019UR-Table-18-Wait-Listed-Patients-in-Acute-Care-Beds-Rev-20210525.pdf ; 2020: https://health.hawaii.gov/shpda/files/2021/10/2020UR-Table-18-Wait-Listed-Patients-in-Acute-Care-Beds.pdf ; 2021: https://health.hawaii.gov/shpda/files/2022/10/2021UR-Table-18-Wait-Listed-Patients-in-Acute-Care-Beds.pdf ; 2022: https://health.hawaii.gov/shpda/files/2023/09/2022UR-Table-18-Wait-Listed-Patients-in-Acute-Care-Beds-1.pdf ; 2023: https://health.hawaii.gov/shpda/files/2024/09/2023UR-Table-18-Wait-Listed-Patients-in-Acute-Care-Beds.pdf ; 2024: https://health.hawaii.gov/shpda/files/2025/09/2024UR-Table-18-Wait-Listed-Patients-in-Acute-Care-Beds.pdf ; local `scratch/snf-cost-reports/shpda-waitlist-series-2017-2024.csv`, `shpda-table18-by-hospital-2023-2024.csv`, `shpda-history/` |
| SHPDA Table 16 (licensed long-term care beds not staffed or set up), 2017 to 2024 | Unstaffed long-term care beds on Dec 31 (the staffing limit, reported only) | 2017: https://health.hawaii.gov/shpda/files/2018/10/Table-16-Long-Term-Care-Bed-Not-Staffed-by-County-2017-with-state-total-revised-20181203.pdf ; 2018: https://health.hawaii.gov/shpda/files/2019/11/Table-16-Licensed-Long-Term-Care-Beds-Not-Staffed-or-Set-Up.pdf ; 2019: https://health.hawaii.gov/shpda/files/2025/06/2019UR-Table-16-Licensed-Long-Term-Care-Beds-Not-Staffed-or-Set-Up-Rev-20210520.pdf ; 2020: https://health.hawaii.gov/shpda/files/2021/10/2020UR-Table-16-Licensed-Long-Term-Care-Beds-Not-Staffed-or-Set-Up.pdf ; 2021: https://health.hawaii.gov/shpda/files/2022/10/2021UR-Table-16-Licensed-Long-Term-Care-Beds-Not-Staffed-or-Set-Up.pdf ; 2022: https://health.hawaii.gov/shpda/files/2023/09/2022UR-Table-16-Licensed-Long-Term-Care-Beds-Not-Staffed-or-Set-Up-1.pdf ; 2023: https://health.hawaii.gov/shpda/files/2024/09/2023UR-Table-16-Licensed-Long-Term-Care-Beds-Not-Staffed-or-Set-Up.pdf ; 2024: URL to find; local `shpda-2024UR-Table-16-Licensed-Long-Term-Care-Beds-Not-Staffed-or-Set-Up.pdf`, `shpda-2024-table16-ltc-not-staffed.csv` |
| SHPDA 2024 Table 28, Glossary | Definition of waitlisted days | https://health.hawaii.gov/shpda/files/2025/09/2024UR-Table-28-Glossary.pdf |
| HCR 161 Complex Patients Work Group report (Med-QUEST, Dec 2017) | Definition of a waitlisted patient; the add-on charge; the objection context | https://humanservices.hawaii.gov/wp-content/uploads/2018/01/HCR-161-2017-Report-Re-Complex-Patients-Work-Group.pdf |
| NHS Borders board paper on delayed discharges (2015) | Reference for the 14-day benchmark | https://www.nhsborders.scot.nhs.uk/media/275132/delayed-discharges.pdf |
| KFF state indicator, hospital expenses per inpatient day by ownership (AHA Annual Survey), Hawaii nonprofit, 2023 | `KFF_COST_DAY`, the base of the avoidable-cost range | https://www.kff.org/health-costs/state-indicator/expenses-per-inpatient-day-by-ownership/ |
| Med-QUEST provider memos, Jan 2016 to Jan 2026 (QI-1521 to QI-2532) | Median nursing-facility rate; hospital waitlisted rate | Memo index: https://medquest.hawaii.gov/content/medquest/en/plans-providers/provider-memo.html ; local `scratch/snf-cost-reports/hi-medicaid-rate-series-2016-2026.csv`, `medquest-QI-2326-rates-jul2023.pdf`, `medquest-QI-2532-rates-2026.pdf` |
| Medicaid State Plan Amendment HI-23-0014, Attachment 4.19-D | Rate-event dates (12% adjustment for private homes, effective Jan 13, 2021; Jan 2024 reset); index-only method since 2024 | https://www.medicaid.gov/sites/default/files/2024-02/HI-23-0014.pdf ; local `medicaid-SPA-HI-23-0014-NF-rate-method.pdf` |
| CMS SNF cost reports, FY2011 to FY2023 (Worksheet A) | Median nursing-home cost per day, all facilities and Medicaid-heavy homes (figure c) | https://data.cms.gov/provider-compliance/cost-reports/skilled-nursing-facility-cost-report ; local `hi-snf-cost-per-day-series-2011-2023.csv`, `extract_hi_snf.py` |
| CMS SNF PPS final rule fact sheets, FY2022 (CMS-1746-F) and FY2025 (CMS-1802-F) | Medicare comparators in the text only (+1.2%, net +4.2%); not model inputs | https://www.cms.gov/newsroom/fact-sheets/fiscal-year-fy-2022-skilled-nursing-facility-snf-prospective-payment-system-pps-final-rule-cms-1746 ; https://www.cms.gov/newsroom/fact-sheets/fiscal-year-2025-skilled-nursing-facility-prospective-payment-system-final-rule-cms-1802-f |

## Inputs: the named contract

`(y)` is one value per year, `(c,y)` per county and year, `(fy)` per CMS fiscal year, `(d)` per memo effective month. Each series is a named column in a year table. Every input cell is filled yellow (Conventions).

### Waitlist (SHPDA Table 18)

| Name | Value | Unit | Source |
|---|---|---|---|
| `WL_PATIENTS(y)` | Series table below | patients | Table 18, statewide total, each year |
| `WL_DAYS(y)` | Series table below | days | Table 18, statewide total |
| `REASON_NO_BED(y)` | Series table below | patients, Dec 31 | Table 18, reason F |
| `REASON_BEHAVIOR(y)` | Series table below | patients, Dec 31 | Table 18, reason G |
| `REASON_SPECIAL_CARE(y)` | Series table below | patients, Dec 31 | Table 18, reason H |
| `REASON_FINANCIAL(y)` | Series table below | patients, Dec 31 | Table 18, reason I |
| `REASON_GUARDIANSHIP(y)` | Series table below | patients, Dec 31 | Table 18, reason J |
| `REASON_PASARR(y)` | Series table below | patients, Dec 31 | Table 18, reason K |
| `REASON_OTHER(y)` | Series table below | patients, Dec 31 | Table 18, reason L |
| `DEC31_TOTAL(y)` | Series table below | patients, Dec 31 | Table 18, total by level of care |
| `DEC31_NF(y)` | 157 (2023), 120 (2024) | patients, Dec 31 | Table 18, type A "SNF, ICF, or SNF/ICF" |
| `LTC_UNSTAFFED(y)` | Series table below | beds, Dec 31 | Table 16, statewide |
| `LTC_UNSTAFFED_STAFF_2024` | 353 | beds, Dec 31, 2024 | Table 16, reason text naming staffing (Okutsu Veterans Home's 21 "staff to census" excluded) |

| y | `WL_PATIENTS` | `WL_DAYS` | `NO_BED` | `BEHAVIOR` | `SPECIAL_CARE` | `FINANCIAL` | `GUARDIANSHIP` | `PASARR` | `OTHER` | `DEC31_TOTAL` | `LTC_UNSTAFFED` |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 2017 | 6,187 | 61,470 | 39 | 16 | 23 | 18 | 11 | 2 | 11 | 157 | 128 |
| 2018 | 5,294 | 66,091 | 57 | 29 | 17 | 31 | 18 | 2 | 17 | 171 | 160 |
| 2019 | 4,790 | 78,911 | 72 | 27 | 21 | 32 | 26 | 0 | 6 | 184 | 248 |
| 2020 | 4,553 | 57,297 | 41 | 24 | 22 | 30 | 15 | 0 | 12 | 144 | 462 |
| 2021 | 4,927 | 60,050 | 50 | 36 | 22 | 43 | 20 | 0 | 18 | 189 | 614 |
| 2022 | 3,681 | 78,561 | 57 | 35 | 39 | 75 | 29 | 5 | 10 | 250 | 661 |
| 2023 | 3,610 | 83,097 | 42 | 48 | 26 | 45 | 28 | 1 | 1 | 191 | 510 |
| 2024 | 3,123 | 60,876 | 31 | 29 | 18 | 60 | 31 | 0 | 4 | 173 | 548 |

Reason columns drop the `REASON_` prefix to fit. In 2017 the reasons sum to 120 against 157 by level of care (37 patients have no recorded reason; blank cells in the source).

### County (Table 18, hospital rows summed by county)

| Name | Value | Unit | Source |
|---|---|---|---|
| `CTY_PATIENTS(c,y)` | 2023: Hawaii 869, Honolulu 2,255, Kauai 86, Maui 400. 2024: Hawaii 727, Honolulu 1,979, Kauai 64, Maui 353 | patients | Table 18 by hospital, `shpda-table18-by-hospital-2023-2024.csv` |
| `CTY_DAYS(c,y)` | 2023: Hawaii 23,212, Honolulu 40,327, Kauai 1,720, Maui 17,838. 2024: Hawaii 20,174, Honolulu 27,380, Kauai 1,927, Maui 11,395 | days | Same |

### Benchmark and cost

| Name | Value | Unit | Source |
|---|---|---|---|
| `BENCH_DAYS` | 14 | days per patient (average) | My benchmark; reference: Scotland's 2-week maximum (NHS Borders, 2015) |
| `KFF_COST_DAY` | 3,664 | USD per adjusted inpatient day, 2023 | KFF (AHA Annual Survey), Hawaii nonprofit hospitals ($3,663.56) |
| `AVOID_COST_LO` | 550 | USD per waitlisted day | About 15% of `KFF_COST_DAY`, rounded (brief) |
| `AVOID_COST_HI` | 1,470 | USD per waitlisted day | About 40% of `KFF_COST_DAY`, rounded (brief; unrounded 1,466) |
| `WL_RATE_JAN(y)` | 283.60 (2023), 464.97 (2024) | USD per waitlisted day | Med-QUEST QI-2225 (Jan 2023), QI-2342 (Jan 2024) |
| `WL_RATE_JUL(y)` | 302.89 (2023), 455.59 (2024) | USD per waitlisted day | QI-2326 (Jul 2023), QI-2413 (Jul 2024) |
| `WL_RATE_2026` | 486.76 | USD per waitlisted day | QI-2532 (Jan 2026), the current case |
| `WL_RATE_LEAHI_2026` | 480.19 | USD per waitlisted day | QI-2532; reported as a note, not used in a calculation |

### Rates and cost for figure c

| Name | Value | Unit | Source |
|---|---|---|---|
| `NF_RATE_MEDIAN(d)` | Jan 2016 to Jan 2026, 17 memos (2019-07 has no median; left blank) | USD per day | Med-QUEST memos, `hi-medicaid-rate-series-2016-2026.csv` |
| `NF_COST_ALL(fy)` | FY2011 to FY2023 (295.13 ... 519.16) | USD per patient day | CMS cost reports, median of all Hawaii facilities, `hi-snf-cost-per-day-series-2011-2023.csv` |
| `NF_COST_MCD_HEAVY(fy)` | FY2011 to FY2023 (287.76 ... 461.78) | USD per patient day | Same file; Medicaid-heavy = at least 50% Medicaid days, excluding complex-care Kulana Malama and Islands Skilled (research notes; rule to verify against the extract) |

### Rate events and test settings

| Name | Value | Unit | Source |
|---|---|---|---|
| `DATE_ADJ_2021` | 2021-01-13 | date | SPA HI-23-0014: 12% adjustment for private homes (first seen in the July 2021 memo) |
| `DATE_RESET_2024` | 2024-01-01 | date | SPA HI-23-0014; QI-2342 |
| `T1_MIN_DROP_PTS` | 5 | percentage points | Brief, test 1 |
| `COVID_YEARS` | 2020, 2021 | years | Brief |

## Structure

(proposed: sheet layout derived by Claude from my decisions) One workbook, `capabilities/economic-research/model.xlsx`, replacing the old one:

| Sheet | Purpose |
|---|---|
| `Inputs` | Every input above, yellow; scalar inputs in a Name, Value, Unit, Source table; year-indexed inputs in one year table; county, rate and cost series in their own tables |
| `Waitlist` | Year table 2017 to 2024: days per patient, benchmark days, excess days, beds, reason shares, flags |
| `Cost` | 2023, 2024 and the current case: average waitlisted rate, net cost per day (low, high), cost of excess days |
| `Tests` | The three tests, each with its inputs, result and verdict |
| `County` | Days per patient and share of days by county, 2023 and 2024 |
| `FigureData` | The exact series each figure plots |
| `Checks` | Validation rules below, each PASS or FAIL |

## Calculation logic

(proposed: formulas derived by Claude from my Purpose, conventions and test definitions; the definitions themselves are mine)

### Waitlist, by year (2017 to 2024)

- `DAYS_PER_PT(y) = WL_DAYS(y) / WL_PATIENTS(y)`
- `BENCH_TOTAL(y) = WL_PATIENTS(y) * BENCH_DAYS`
- `EXCESS_DAYS(y) = MAX(0, WL_DAYS(y) - BENCH_TOTAL(y))` (proposed: floored at 0 in years at or under the benchmark)
- `WITHIN_DAYS(y) = WL_DAYS(y) - EXCESS_DAYS(y)` (figure d)
- `EXCESS_SHARE(y) = EXCESS_DAYS(y) / WL_DAYS(y)`
- `DAYS_IN_YEAR(y)` = 366 in leap years (2020, 2024), else 365
- `EXCESS_BEDS(y) = EXCESS_DAYS(y) / DAYS_IN_YEAR(y)`, beds occupied by excess days on an average day
- `MEETS_BENCH(y) = DAYS_PER_PT(y) <= BENCH_DAYS`
- `REASON_SUM(y)` = the sum of the seven `REASON_` inputs
- `SHARE_<reason>(y) = REASON_<reason>(y) / REASON_SUM(y)`, for each of the seven reasons
- `NF_SHARE_DEC31(y) = DEC31_NF(y) / DEC31_TOTAL(y)`, 2023 and 2024, reported beside the result only
- `WL_DAYS_CHANGE_2024 = WL_DAYS(2024) / WL_DAYS(2023) - 1`
- `FLAG_2017_INCOMPLETE` = TRUE when `REASON_SUM(2017) < DEC31_TOTAL(2017)`

### Cost of the gap (2023, 2024, current case)

- `WL_RATE_AVG(y) = (WL_RATE_JAN(y) + WL_RATE_JUL(y)) / 2`
- `NET_COST_LO(y) = AVOID_COST_LO - WL_RATE_AVG(y)`; `NET_COST_HI(y) = AVOID_COST_HI - WL_RATE_AVG(y)`
- `EXCESS_COST_LO(y) = EXCESS_DAYS(y) * NET_COST_LO(y)`; `EXCESS_COST_HI(y) = EXCESS_DAYS(y) * NET_COST_HI(y)`
- Current case: `NET_COST_LO_NOW = AVOID_COST_LO - WL_RATE_2026`, `NET_COST_HI_NOW = AVOID_COST_HI - WL_RATE_2026`; (proposed) `EXCESS_COST_LO_NOW` and `EXCESS_COST_HI_NOW` value `EXCESS_DAYS(2024)` at those net costs

### County (2023, 2024)

- `CTY_DAYS_PER_PT(c,y) = CTY_DAYS(c,y) / CTY_PATIENTS(c,y)`
- `CTY_DAY_SHARE(c,y) = CTY_DAYS(c,y) / WL_DAYS(y)`

### Tests (definitions in Conventions)

- Test 1, opportunity cost:
  - `T1_DROP_2021 = SHARE_FINANCIAL(2020) - SHARE_FINANCIAL(2021)`
  - `T1_DROP_2024 = SHARE_FINANCIAL(2023) - SHARE_FINANCIAL(2024)`
  - `T1_PASS_2021 = T1_DROP_2021 * 100 >= T1_MIN_DROP_PTS`; `T1_PASS_2024` likewise
  - (proposed wording) `T1_VERDICT` = "Falsified" if neither passes; "Not falsified" if either passes. If only `T1_PASS_2021` passes, add "weak (rests on the COVID-confounded 2021 comparison)"
- Test 2, wait for the reset:
  - `T2_SLOPE` = least-squares slope of `DAYS_PER_PT(y)` on y, 2017 to 2023
  - `T2_VERDICT` = "Met" if `T2_SLOPE < 0`, else "Not met"
- Test 3, new capacity:
  - `LARGEST_REASON(y)` = the reason with the highest count; if two or more tie at the top, no reason is largest that year
  - `T3_YEARS = COUNT(y where LARGEST_REASON(y) = no bed)`, 2017 to 2024
  - `T3_VERDICT` = "Met" if `T3_YEARS > 8 / 2` (5 or more of 8), else "Not met"; 2017 carries `FLAG_2017_INCOMPLETE`

## Conventions

- **Definitions set after results.** I locked the three tests on Sep 28, before pulling the 2017 to 2022 data. I set the definitions below on Sep 28, after I had seen the results. The paper says so.
- **Which days count.** The gap and its cost cover the whole acute waitlist, all levels of care, because the yearly figures aren't split by level. The Dec 31 nursing-facility shares (82% in 2023, 69% in 2024) are reported beside the result, not used to scale it.
- **Days in the year.** Calendar days: 366 in 2020 and 2024.
- **Reason shares.** The denominator is the sum of the seven recorded reasons (`REASON_SUM`), not the Dec 31 total by level of care. For 2017 that is 120, not 157.
- **Year's waitlisted rate.** The simple average of the January and July rates.
- **Test 1.** Drop = the financial share on the earlier Dec 31 minus the share on the later Dec 31. Shares are compared unrounded. The hypothesis fails only if both drops fall short of 5 points (lenient reading, as in the brief). 2020 and 2021 are flagged for COVID, and the 2021 comparison is confounded.
- **Test 2.** "Falling steadily toward 14" means the linear (least-squares) trend of days per patient over 2017 to 2023 slopes down.
- **Test 3.** "Most years" means more than half of 2017 to 2024 (5 or more of 8). 2017 is counted on its recorded reasons and flagged. A tie for the largest reason counts as "not largest."
- **Verdicts only.** The model reports each test's verdict. It does not rank or cost the options and does not choose between the add-on and the top-up.
- **Dates on figure c.** Each fiscal year's cost is plotted at July 1 of that year; each rate at its memo's effective month. Cost lines stop at 2023; the rate line runs to 2026.
- **Precision.** No rounding inside calculations. (proposed) Display days per patient to 1 decimal, shares to 0.1 point, beds to 1 decimal, dollars per day to cents, totals to whole dollars.
- **Workbook formatting.** Every manual-input cell is filled yellow. Row headers and column headers are set apart by color: (proposed) dark blue fill with white bold text for column headers, light gray fill with bold text for row headers. Columns and row heights are sized so every value and label is fully visible (no `####`, no clipped text).
- **Names.** Every defined name is absolute (the 2026-09-28 lesson from the old model). No calculation refers to a cell address where a name exists.

## Validation rules

(proposed: rules and hand-check values derived by Claude from the draft inputs)

Structural:

- Every calculated cell is a formula; only `Inputs` holds typed values.
- No error cells anywhere.
- Every name in this spec exists in the workbook as an absolute defined name, and no other names exist.
- Every input cell is yellow; no calculated cell is yellow.
- The workbook opens in Excel with no errors, and every `Checks` row reads PASS.

Hand checks (from the draft inputs):

| Check | Expected |
|---|---|
| `DAYS_PER_PT(2023)`, `DAYS_PER_PT(2024)` | 23.02, 19.49 |
| `EXCESS_DAYS(2023)`, `EXCESS_DAYS(2024)` | 32,557; 17,154 |
| `EXCESS_BEDS(2023)`, `EXCESS_BEDS(2024)` | 89.2 (365 days), 46.9 (366 days) |
| `EXCESS_DAYS` in 2017, 2018, 2020, 2021 | 0 (days per patient under 14) |
| `WL_RATE_AVG(2023)`, `WL_RATE_AVG(2024)` | 293.245, 460.28 |
| `NET_COST_LO(2024)`, `NET_COST_HI(2024)` | 89.72, 1,009.72 |
| `EXCESS_COST_LO(2024)`, `EXCESS_COST_HI(2024)` | about 1,539,057; about 17,320,737 |
| `NET_COST_LO_NOW`, `NET_COST_HI_NOW` | 63.24, 983.24 |
| `WL_DAYS_CHANGE_2024` | -26.7% |
| `T1_DROP_2021`, `T1_DROP_2024` | -1.92 points, -11.12 points (both rises; unrounded, the 2021 rise is 1.9 points, not the 2.0 in decision tree v5 and the handoff, which came from rounded shares) |
| `T2_SLOPE` | about +1.88 days per patient per year |
| `T3_YEARS` | 5 (2017 to 2021) |
| `REASON_SUM(y) = DEC31_TOTAL(y)` | Every year except 2017 (120 vs 157) |
| Sum of `CTY_DAYS(c,y)` = `WL_DAYS(y)` | 2023 and 2024 |
| `KFF_COST_DAY` x 0.15 and x 0.40 | 549.60 and 1,465.60, within rounding of `AVOID_COST_LO` and `AVOID_COST_HI` |

## Outputs

(proposed: list derived by Claude from the Purpose)

- By year, 2017 to 2024: `DAYS_PER_PT`, `WL_DAYS`, `EXCESS_DAYS`, `EXCESS_SHARE`, `EXCESS_BEDS`, `MEETS_BENCH`, the seven `SHARE_` values, `LTC_UNSTAFFED`, COVID flag.
- 2023, 2024 and now: `WL_RATE_AVG`, `NET_COST_LO/HI`, `EXCESS_COST_LO/HI` (and the `_NOW` versions); `NF_SHARE_DEC31`; `WL_DAYS_CHANGE_2024`.
- County row: `CTY_DAYS_PER_PT(c,y)` and `CTY_DAY_SHARE(c,y)`, 2023 and 2024.
- Tests: `T1_DROP_2021`, `T1_DROP_2024`, `T1_VERDICT`; `T2_SLOPE`, `T2_VERDICT`; `LARGEST_REASON(y)`, `T3_YEARS`, `T3_VERDICT`.
- Staffing note: `LTC_UNSTAFFED(2024)` and `LTC_UNSTAFFED_STAFF_2024`.

### Figures

Built from `FigureData` into `analysis/figures/` (PNG). (proposed) File names `fig-a-days-per-patient.png`, `fig-b-reason-shares.png`, `fig-c-cost-vs-rate.png`, `fig-d-excess-days.png`, `fig-g-opportunity-cost.png`. Every figure has a title, labeled axes with units, a source line, and a caption that reads on its own. No repository URL on any figure.

(proposed: chart forms, markers and labels below are Claude's; the choice of figures, figure c's two cost lines and 2023 cutoff, figure g as a schematic and its caption point are mine)

| Fig | Content |
|---|---|
| a | `DAYS_PER_PT`, 2017 to 2024, as a line, with a horizontal line at `BENCH_DAYS` (14). Vertical markers at the rate events (`DATE_ADJ_2021`, `DATE_RESET_2024`). 2020 and 2021 marked COVID. |
| b | The seven `SHARE_` values by year, 2017 to 2024, as 100% stacked bars. The two test 1 windows (Dec 2020 to Dec 2021, Dec 2023 to Dec 2024) shaded. 2017 labeled "recorded reasons only (120 of 157)". Carries tests 1 and 3. |
| c | `NF_RATE_MEDIAN(d)` Jan 2016 to Jan 2026 (OPEN: step line or connected points; the series has no memo for Jan 2018, Jan 2020, Jan 2021 or Jul 2022 and no median for Jul 2019, and the Jan 13, 2021 adjustment first shows in the Jul 2021 memo, so a step line would sit flat past its own event marker). `NF_COST_MCD_HEAVY(fy)` and `NF_COST_ALL(fy)`, FY2011 to FY2023, as lines at July 1 of each year, stopping at 2023. Rate events marked. Note on the chart that the rate median covers all homes and the cost medians the stated groups. |
| d | `WL_DAYS` by year, 2017 to 2024, as stacked bars: `WITHIN_DAYS` and `EXCESS_DAYS`. |
| g | Schematic, no dollar values: one bed's year as one long Medicaid stay, against the same bed turning over a run of short Medicare stays. The caption states that test 1 did not support this mechanism. |

## Success criteria

A finished paper:

1. States the problem with numbers: days per patient against 14 for 2023 and 2024, the gap in excess days and beds, and the net cost range.
2. Names the course concepts doing the work: opportunity cost, economic profit, elasticity, administered prices and price takers, positive vs. normative.
3. Reports all three test verdicts exactly as set, including test 1's failure; discloses that the definitions were fixed after the results; flags the COVID years.
4. Covers what happens next under named conditions (for example, if the index-only rate falls behind cost again, or if no-bed becomes the largest reason again).
5. Has a recommendation that follows from the test results, not from the hypothesis, and is defended against the strongest objection.
6. Names that objection in advance: the financial category can't carry the main test. SHPDA doesn't define it, and it mixes homes turning Medicaid patients away with patients waiting on an eligibility decision or a coverage problem.
7. States the limitations: all levels of care vs. the nursing-facility scope, the one-day snapshot, self-reporting, the payer assumption, staffing.
8. Cites every figure in the text; figure a or b carries evidence the argument needs; every caption reads on its own.
9. Uses only numbers verified against their primary source, with one error caught logged.
10. Meets the format: at most 4 pages, Times New Roman 12, double-spaced, 1-inch margins, one citation style, no repository URL, identifying information only on the title page.

## Audit findings

Added after the build.
