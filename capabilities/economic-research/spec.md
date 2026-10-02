---
type: spec
capability: economic-research
engagement: research-paper
date: 2026-09-28
status: built            # draft | built | audited
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
| SHPDA Table 16 (licensed long-term care beds not staffed or set up), 2017 to 2024 | Unstaffed long-term care beds on Dec 31 (the staffing limit, reported only) | 2017: https://health.hawaii.gov/shpda/files/2018/10/Table-16-Long-Term-Care-Bed-Not-Staffed-by-County-2017-with-state-total-revised-20181203.pdf ; 2018: https://health.hawaii.gov/shpda/files/2019/11/Table-16-Licensed-Long-Term-Care-Beds-Not-Staffed-or-Set-Up.pdf ; 2019: https://health.hawaii.gov/shpda/files/2025/06/2019UR-Table-16-Licensed-Long-Term-Care-Beds-Not-Staffed-or-Set-Up-Rev-20210520.pdf ; 2020: https://health.hawaii.gov/shpda/files/2021/10/2020UR-Table-16-Licensed-Long-Term-Care-Beds-Not-Staffed-or-Set-Up.pdf ; 2021: https://health.hawaii.gov/shpda/files/2022/10/2021UR-Table-16-Licensed-Long-Term-Care-Beds-Not-Staffed-or-Set-Up.pdf ; 2022: https://health.hawaii.gov/shpda/files/2023/09/2022UR-Table-16-Licensed-Long-Term-Care-Beds-Not-Staffed-or-Set-Up-1.pdf ; 2023: https://health.hawaii.gov/shpda/files/2024/09/2023UR-Table-16-Licensed-Long-Term-Care-Beds-Not-Staffed-or-Set-Up.pdf ; 2024: https://health.hawaii.gov/shpda/files/2025/09/2024UR-Table-16-Licensed-Long-Term-Care-Beds-Not-Staffed-or-Set-Up.pdf ; local `shpda-2024UR-Table-16-Licensed-Long-Term-Care-Beds-Not-Staffed-or-Set-Up.pdf`, `shpda-2024-table16-ltc-not-staffed.csv` |
| SHPDA 2024 Table 28, Glossary | Definition of waitlisted days | https://health.hawaii.gov/shpda/files/2025/09/2024UR-Table-28-Glossary.pdf |
| HCR 161 Complex Patients Work Group report (Med-QUEST, Dec 2017) | Definition of a waitlisted patient; the add-on charge; the objection context | https://humanservices.hawaii.gov/wp-content/uploads/2018/01/HCR-161-2017-Report-Re-Complex-Patients-Work-Group.pdf |
| NHS Borders board paper on delayed discharges (2015) | Reference for the 14-day benchmark | https://www.nhsborders.scot.nhs.uk/media/275132/delayed-discharges.pdf |
| KFF state indicator, hospital expenses per inpatient day by ownership (AHA Annual Survey), Hawaii nonprofit, 2023 | `KFF_COST_DAY`, the base of the avoidable-cost range | https://www.kff.org/health-costs/state-indicator/expenses-per-inpatient-day-by-ownership/ |
| Med-QUEST provider memos, Jan 2016 to Jan 2026 (QI-1521 to QI-2532) | Median nursing-facility rate; hospital waitlisted rate | Memo index: https://medquest.hawaii.gov/content/medquest/en/plans-providers/provider-memo.html ; local `scratch/snf-cost-reports/hi-medicaid-rate-series-2016-2026.csv`, `medquest-QI-2326-rates-jul2023.pdf`, `medquest-QI-2532-rates-2026.pdf` |
| Medicaid State Plan Amendment HI-23-0014, Attachment 4.19-D | Rate-event dates (12% adjustment for private homes, effective Jan 13, 2021; Jan 2024 reset); index-only method since 2024 | https://www.medicaid.gov/sites/default/files/2024-02/HI-23-0014.pdf ; local `medicaid-SPA-HI-23-0014-NF-rate-method.pdf` |
| CMS SNF cost reports, FY2011 to FY2023 (Worksheet A) | Median nursing-home cost per day, all facilities and Medicaid-heavy homes (figure c) | https://data.cms.gov/provider-compliance/cost-reports/skilled-nursing-facility-cost-report ; local `hi-snf-cost-per-day-series-2011-2023.csv`, `extract_hi_snf.py` |
| CMS SNF PPS final rule fact sheets, FY2022 (CMS-1746-F) and FY2025 (CMS-1802-F) | Medicare comparators in the text only (+1.2%, net +4.2%); not model inputs | https://www.cms.gov/newsroom/fact-sheets/fiscal-year-fy-2022-skilled-nursing-facility-snf-prospective-payment-system-pps-final-rule-cms-1746 ; https://www.cms.gov/newsroom/fact-sheets/fiscal-year-2025-skilled-nursing-facility-prospective-payment-system-final-rule-cms-1802-f |
| CMS SNF PPS final rule fact sheets, FY2024 (CMS-1779-F), FY2025 (CMS-1802-F), FY2026 (CMS-1827-F) and FY2027 (CMS-1843-F) | `MB_GROSS`: the SNF market basket before the forecast-error and productivity adjustments | https://www.cms.gov/newsroom/fact-sheets/fiscal-year-fy-2024-skilled-nursing-facility-perspective-payment-system-final-rule-cms-1779-f ; https://www.cms.gov/newsroom/fact-sheets/fiscal-year-2025-skilled-nursing-facility-prospective-payment-system-final-rule-cms-1802-f ; https://www.cms.gov/newsroom/fact-sheets/fy-2026-skilled-nursing-facility-snf-prospective-payment-system-final-rule-cms-1827-f ; https://www.cms.gov/newsroom/fact-sheets/fiscal-year-2027-skilled-nursing-facility-prospective-payment-system-final-rule-cms-1843-f |

## Inputs: the named contract

`(y)` is one value per year, `(c,y)` per county and year, `(fy)` per CMS fiscal year, `(d)` per memo effective month. Each series is a named column in a year table. Every input cell is filled yellow (Conventions).

**Verification columns.** Every input row on `Inputs` carries three yellow manual columns, "Source page", "Verified by" and "Date verified", which I fill in as I verify each value against its source. A cell at the top of `Inputs` counts the rows with a date.

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

Reason columns drop the `REASON_` prefix to fit. In 2017 the reasons sum to 120 against 157 by level of care. Maui Memorial records reasons for 6 of its 47 patients (the other cells are blank, "data not available"), and Queen's Punchbowl's reasons sum to 43 against its total of 39.

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
| `MB_GROSS(fy)` | 3.0% (FY2024), 3.0% (FY2025), 3.3% (FY2026), 3.3% (FY2027) | percent a year | CMS SNF PPS final rule fact sheets, FY2024 to FY2027: market basket increase before the forecast-error and productivity adjustments. A national Medicare index standing in for Hawaii nursing-home cost growth |
| `ADMIT_WINDOW_DAYS` | 14 | days | My judgment (2026-10-01): the time within which a home giving the guarantee must admit. Set equal to the benchmark |
| `MEDIAN_MCD_LOS` | 542.83 | days | CMS SNF cost reports, FY2023 file, "SNF Average Length of Stay Title XIX", each Hawaii home's latest report: median of the 31 homes reporting a value (range 8.26 to 1,680.75). Rounded to 543 in the paper |

### Rates and cost for figure c

| Name | Value | Unit | Source |
|---|---|---|---|
| `NF_RATE_MEDIAN(d)` | Jan 2016 to Jan 2026, 17 memos (2019-07 left blank: QI-1911B, the controlling July 2019 memo, lists but omits its acuity rate table) | USD per day | Med-QUEST memos, `hi-medicaid-rate-series-2016-2026.csv` |
| `NF_COST_ALL(fy)` | FY2011 to FY2023 (295.13 ... 519.16) | USD per patient day | CMS cost reports, median of all Hawaii facilities, `hi-snf-cost-per-day-series-2011-2023.csv` |
| `NF_COST_MCD_HEAVY(fy)` | FY2011 to FY2023 (287.76 ... 461.79) | USD per patient day | Same file; Medicaid-heavy = Medicaid (Title XIX) days at least 50% of total days, excluding complex-care Kulana Malama (CCN 125057) and Islands Skilled Nursing and Rehab (CCN 125067). Cost per day = (Worksheet A salaries + other costs) / total days, each home's latest report in the file |

### Rate events and test settings

| Name | Value | Unit | Source |
|---|---|---|---|
| `DATE_ADJ_2021` | 2021-01-13 | date | SPA HI-23-0014, Attachment 4.19-D p. 38: 12% adjustment for private homes (first seen in the January 2021 memo, QI-2041) |
| `DATE_RESET_2024` | 2024-01-01 | date | SPA HI-23-0014; QI-2342 |
| `T1_MIN_DROP_PTS` | 5 | percentage points | Brief, test 1 |
| `COVID_YEARS` | 2020, 2021 | years | Brief |

## Structure

One workbook, `capabilities/economic-research/model.xlsx`, replacing the old one:

| Sheet | Purpose |
|---|---|
| `Inputs` | Every input above, yellow; scalar inputs in a Name, Value, Unit, Source table; year-indexed inputs in one year table; county, rate and cost series in their own tables |
| `Waitlist` | Year table 2017 to 2024: days per patient, benchmark days, excess days, beds, reason shares, flags |
| `Cost` | 2023, 2024 and the current case: average waitlisted rate, net cost per day (low, high), cost of excess days; days saved and the offer ceiling per placed patient |
| `Tests` | The three tests, each with its inputs, result and verdict; then a side diagnostic (test 1 on counts) that changes no verdict |
| `County` | Days per patient and share of days by county, 2023 and 2024 |
| `Capacity` | Excess beds against unstaffed long-term-care beds, 2017 to 2024: a scale comparison, not a sizing |
| `Conditions` | Condition (a): cost carried forward at the gross CMS SNF market basket against the rate at its post-reset pace; the stay shortfall; the paper figures (rate-cost gap, rate rises, behavior and guardianship share) |
| `FigureData` | The exact series each figure plots |
| `WorkedExample` | A synthetic anchor with its own yellow inputs, not linked to `Inputs`, run through the same formula shapes as the model, with an expected column and a match column |
| `Checks` | Regression anchors, invariants and error scans (Validation rules), with an ALL CHECKS cell and a count of anchors matching |

## Calculation logic

### Waitlist, by year (2017 to 2024)

- `DAYS_PER_PT(y) = WL_DAYS(y) / WL_PATIENTS(y)`
- `BENCH_TOTAL(y) = WL_PATIENTS(y) * BENCH_DAYS`
- `GAP_PER_PT(y) = DAYS_PER_PT(y) - BENCH_DAYS`, the signed gap (negative in years under the benchmark)
- `EXCESS_DAYS(y) = MAX(0, WL_DAYS(y) - BENCH_TOTAL(y))` (floored at 0 in years at or under the benchmark)
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
- Current case: `NET_COST_LO_NOW = AVOID_COST_LO - WL_RATE_2026`, `NET_COST_HI_NOW = AVOID_COST_HI - WL_RATE_2026`; `EXCESS_COST_LO_NOW` and `EXCESS_COST_HI_NOW` value `EXCESS_DAYS(2024)` at those net costs
- Days saved per placed patient: `DAYS_SAVED_PER_PT = MAX(0, DAYS_PER_PT(2024) - ADMIT_WINDOW_DAYS)`
- Offer ceiling per placed patient: `CEIL_PER_PT_LO = NET_COST_LO_NOW * DAYS_SAVED_PER_PT`; `CEIL_PER_PT_HI = NET_COST_HI_NOW * DAYS_SAVED_PER_PT`, the most a hospital gains from placing one waitlisted patient under the guarantee, at the current waitlisted rate. This replaces the September 30 definition, which used the full 2024 average wait

### County (2023, 2024)

- `CTY_DAYS_PER_PT(c,y) = CTY_DAYS(c,y) / CTY_PATIENTS(c,y)`
- `CTY_DAY_SHARE(c,y) = CTY_DAYS(c,y) / WL_DAYS(y)`

### Capacity (2017 to 2024)

- `CAP_RATIO(y) = LTC_UNSTAFFED(y) / EXCESS_BEDS(y)`, unstaffed long-term-care beds per excess bed; blank in years with no excess beds, so no division by zero
- `CAP_RATIO_STAFF_2024 = LTC_UNSTAFFED_STAFF_2024 / EXCESS_BEDS(2024)`, the same against the beds unstaffed for lack of staff; blank if `EXCESS_BEDS(2024)` is 0

### Condition (a)

- `RATE_GROWTH_2024_2026 = (NF_RATE_MEDIAN(Jan 2026) / NF_RATE_MEDIAN(Jan 2024)) ^ (1/2) - 1`, the post-reset pace, compounded over two years
- `COST_FWD(fy)`, FY2024 to FY2027: `NF_COST_MCD_HEAVY(2023)` carried forward, each year times (1 + `MB_GROSS(fy)`)
- `RATE_FWD(2026)` = `NF_RATE_MEDIAN(Jan 2026)`; `RATE_FWD(2027) = RATE_FWD(2026) x (1 + RATE_GROWTH_2024_2026)`
- `COND_A_GAP(fy) = 1 - RATE_FWD(fy) / COST_FWD(fy)`, FY2026 and FY2027
- `STAY_SHORTFALL = (COST_FWD(2026) - NF_RATE_MEDIAN(Jan 2026)) * MEDIAN_MCD_LOS`, projected cost above the rate over a median Medicaid stay

### Paper figures

Numbers the paper states that are computed from the inputs, so each has a cell.

- `RATE_COST_GAP(fy) = 1 - NF_RATE_MEDIAN(July of fy) / NF_COST_MCD_HEAVY(fy)`, FY2016 to FY2023; blank in a year with no July median in the rate series (2019, 2022)
- `RATE_COST_GAP_MIN` and `RATE_COST_GAP_MAX`: the smallest and largest `RATE_COST_GAP` (the paper's "19 to 29% below")
- `RATE_RISE_2021 = NF_RATE_MEDIAN(Jul 2021) / NF_RATE_MEDIAN(Jul 2020) - 1` (the paper's "about 18%")
- `RATE_RISE_2024 = NF_RATE_MEDIAN(Jan 2024) / NF_RATE_MEDIAN(Jul 2023) - 1` (the paper's "30%")
- `BEH_GUARD_SHARE_2024 = SHARE_BEHAVIOR(2024) + SHARE_GUARDIANSHIP(2024)` (the paper's "about a third")

### Tests (definitions in Conventions)

- Test 1, opportunity cost:
  - `T1_DROP_2021 = SHARE_FINANCIAL(2020) - SHARE_FINANCIAL(2021)`
  - `T1_DROP_2024 = SHARE_FINANCIAL(2023) - SHARE_FINANCIAL(2024)`
  - `T1_PASS_2021 = T1_DROP_2021 * 100 >= T1_MIN_DROP_PTS`; `T1_PASS_2024` likewise
  - `T1_VERDICT` = "Falsified" if neither passes; "Not falsified" if either passes. If only `T1_PASS_2021` passes, add "weak (rests on the COVID-confounded 2021 comparison)"
- Test 2, wait for the reset:
  - `T2_SLOPE` = least-squares slope of `DAYS_PER_PT(y)` on y, 2017 to 2023
  - `T2_VERDICT` = "Met" if `T2_SLOPE < 0`, else "Not met"
- Test 3, new capacity:
  - `LARGEST_REASON(y)` = the reason with the highest count; if two or more tie at the top, no reason is largest that year
  - `T3_YEARS = COUNT(y where LARGEST_REASON(y) = no bed)`, 2017 to 2024
  - `T3_VERDICT` = "Met" if `T3_YEARS > 8 / 2` (5 or more of 8), else "Not met"; 2017 carries `FLAG_2017_INCOMPLETE`

### Side diagnostic: test 1 on counts (changes no verdict)

- `T1_FIN_CHANGE_2021 = REASON_FINANCIAL(2021) - REASON_FINANCIAL(2020)`
- `T1_OTHER_CHANGE_2021 = (REASON_SUM(2021) - REASON_FINANCIAL(2021)) - (REASON_SUM(2020) - REASON_FINANCIAL(2020))`
- `T1_FIN_CHANGE_2024` and `T1_OTHER_CHANGE_2024` likewise, 2023 to 2024

## Conventions

- **Definitions set after results.** I locked the three tests on Sep 28, before pulling the 2017 to 2022 data. I set the definitions below on Sep 28, after I had seen the results. The paper says so.
- **Which days count.** The gap and its cost cover the whole acute waitlist, all levels of care, because the yearly figures aren't split by level. The Dec 31 nursing-facility shares (82% in 2023, 69% in 2024) are reported beside the result, not used to scale it.
- **Excess days are a lower bound.** Excess days are measured against the average benchmark (total days minus patients x 14, floored at 0), not patient by patient. Because some patients wait beyond 14 days even in a year whose average is under 14, they are a lower bound on the days patients spent beyond 14, and the paper does not describe them as days patients waited beyond two weeks. `GAP_PER_PT` shows each year's signed margin.
- **Days in the year.** Calendar days: 366 in 2020 and 2024.
- **Reason shares.** The denominator is the sum of the seven recorded reasons (`REASON_SUM`), not the Dec 31 total by level of care. For 2017 that is 120, not 157.
- **Year's waitlisted rate.** The simple average of the January and July rates. Each memo's rate is the waitlisted per diem that Attachment A prints for most hospitals. Hilo and Leahi had their own lower rates in the 2023 and 2024 memos, and Leahi in January 2026 (`WL_RATE_LEAHI_2026`).
- **Rate medians.** Each memo's median is taken over every row of its acuity-based long-term-care rate table, including hospital-based and complex-care homes.
- **Test 1.** Drop = the financial share on the earlier Dec 31 minus the share on the later Dec 31. Shares are compared unrounded. The hypothesis fails only if both drops fall short of 5 points (lenient reading, as in the brief). 2020 and 2021 are flagged for COVID, and the 2021 comparison is confounded.
- **Test 2.** "Falling steadily toward 14" means the linear (least-squares) trend of days per patient over 2017 to 2023 slopes down.
- **Test 3.** "Most years" means more than half of 2017 to 2024 (5 or more of 8). 2017 is counted on its recorded reasons and flagged. A tie for the largest reason counts as "not largest."
- **Side diagnostic.** Test 1 on counts describes the data around test 1. It is not a test: it has no verdict, changes no verdict, and was not part of the tests I locked on Sep 28. I chose it on Sep 29, after the results were known.
- **Verdicts only.** The model reports each test's verdict. It does not rank or cost the options and does not choose between the add-on and the top-up.
- **Capacity comparison.** The `Capacity` sheet compares scale; it does not size or cost an option. Three limits go beside it: (1) excess beds cover the acute waitlist across all levels of care on an average day, while unstaffed beds are long-term-care beds on Dec 31, so this is not a bed-for-bed match; (2) reopening beds idle for lack of staff is a workforce lever outside my scope, while my capacity option is new transitional capacity; (3) excess days are a lower bound, so excess beds are too.
- **Condition (a).** Cost is carried forward at the gross CMS SNF market basket, a national Medicare index standing in for Hawaii nursing-home cost growth, and the rate at its 2024 to 2026 pace. The gap is 1 - rate / cost. Cost is dated July 1 of the fiscal year and the rate January, a half-year mismatch. The condition describes what the model computes under stated assumptions; it is not a forecast, and the model does not rank or cost the options.
- **Offer ceiling.** `CEIL_PER_PT_LO` and `CEIL_PER_PT_HI` describe the ceiling of an offer per placed patient. They are not a test and change no verdict. I chose them on Sep 30, after the results were known, and on Oct 1 restated them on the days a hospital saves under the guarantee: the 2024 average wait minus `ADMIT_WINDOW_DAYS`, a 14-day window that is my judgment. The average hides patients who wait much longer. Displayed to cents.
- **Paper figures.** A number the paper computes, as opposed to one it quotes from an outside source, is calculated in the model. The rate-cost gap pairs each fiscal year's Medicaid-heavy cost with that July's rate memo; the rate median covers all homes. Medicare's 1.2% and 4.2% are quoted from CMS, and the six-month window is my judgment; neither is a model cell.
- **Stay shortfall.** `STAY_SHORTFALL` is a rough comparison, not a finding: it sets the cost of Medicaid-heavy homes against the rate for all homes, over the median of homes' average Medicaid stays.
- **Dates on figure c.** Each fiscal year's cost is plotted at July 1 of that year; each rate at its memo's effective month. Cost lines stop at 2023; the rate line runs to 2026.
- **Cost-report periods.** Each CMS fiscal-year file holds 12-month reports whose periods vary by facility; in every year from FY2011 to FY2023, 5 to 8 reports run into the following calendar year. In FY2023, six reports (857 beds) run past the January 2024 reset. The model uses each file as published.
- **Precision.** No rounding inside calculations. Input medians (rate and cost) are taken on unrounded values and rounded half up to the cent. Display days per patient to 1 decimal, shares to 0.1 point, beds to 1 decimal, dollars per day to cents, totals to whole dollars.
- **Workbook formatting.** Every manual-input cell is filled yellow. Row headers and column headers are set apart by color: dark blue fill with white bold text for column headers, light gray fill with bold text for row headers. Columns and row heights are sized so every value and label is fully visible (no `####`, no clipped text).
- **Names.** Every defined name is absolute (the 2026-09-28 lesson from the old model). No calculation refers to a cell address where a name exists. One exception: an error scan on the `Checks` sheet may refer to a sheet's cell range even where the range contains named cells. The `WorkedExample` sheet has no defined names; its formulas refer to its own cells.

## Validation rules

Structural:

- Every calculated cell is a formula; only `Inputs` (including its verification columns) and the `WorkedExample` anchor inputs hold typed values. Expected values in `Checks` and `WorkedExample` are constants inside formulas.
- No error cells anywhere.
- Every name in this spec exists in the workbook as an absolute defined name, and no other names exist.
- Every input cell (on `Inputs`, including the verification columns, and the `WorkedExample` anchor inputs) is yellow; no calculated cell is yellow.
- The workbook opens in Excel with no errors, the ALL CHECKS cell reads ALL PASS, and, with the draft inputs as built, every regression anchor matches.

The `Checks` sheet has three groups.

**Regression anchors.** The hand checks below, each against a fixed expected value from the draft inputs. They are expected to change when I correct an input during verification; after a correction I re-derive the expected value. They are not part of ALL CHECKS. A cell shows "Anchors matching: n of 50."

**Invariants.** These must pass for any inputs. One row each; a row that covers years passes only if it holds in every year.

| # | Invariant |
|---|---|
| I1 | For each year 2017 to 2024, the seven `SHARE_` values sum to 1 |
| I2 | For each year, `WITHIN_DAYS + EXCESS_DAYS = WL_DAYS` |
| I3 | For each year, `EXCESS_DAYS = MAX(0, GAP_PER_PT x WL_PATIENTS)` (this covers the 2019 and 2022 excess days, which are not anchored) |
| I4 | For 2023 and 2024, the county `CTY_DAYS(c,y)` sum to `WL_DAYS(y)` |
| I5 | For 2023 and 2024, the county `CTY_DAY_SHARE(c,y)` sum to 1 |
| I6 | `T2_SLOPE` equals the least-squares slope written out by hand over 2017 to 2023: the sum of (year minus mean year) x (days per patient minus mean) over the sum of (year minus mean year) squared |
| I7 | `T3_YEARS` equals the count of years in which `REASON_NO_BED` is greater than each of the other six reasons, recomputed from the counts |
| I8 | For each year, `MEETS_BENCH` is TRUE exactly when `GAP_PER_PT <= 0` |
| I9 | Every row of the `WorkedExample` sheet reads "match" |

**Error scans.** One row per sheet counting error values in that sheet's used range: `Inputs`, `Waitlist`, `Cost`, `Tests`, `County`, `Capacity`, `Conditions`, `FigureData`, `WorkedExample`, and `Checks` (the rows above the scan, so the scan does not refer to itself). Each passes at 0.

**ALL CHECKS.** One cell: "ALL PASS" when every invariant and every error scan reads PASS; otherwise "REVIEW". Tolerance for invariants is 1e-6.

Hand checks (regression anchors, from the draft inputs; expected to change when an input is corrected). The two `KFF_COST_DAY` x 0.15 and x 0.40 rows check arithmetic on the input, not the model; they stay as anchors.

| Check | Expected |
|---|---|
| `DAYS_PER_PT(2023)`, `DAYS_PER_PT(2024)` | 23.02, 19.49 |
| `EXCESS_DAYS(2023)`, `EXCESS_DAYS(2024)` | 32,557; 17,154 |
| `EXCESS_BEDS(2023)`, `EXCESS_BEDS(2024)` | 89.2 (365 days), 46.9 (366 days) |
| `EXCESS_DAYS` in 2017, 2018, 2020, 2021 | 0 (days per patient under 14) |
| `GAP_PER_PT(2017)`, `GAP_PER_PT(2024)` | -4.06, +5.49 |
| `WL_RATE_AVG(2023)`, `WL_RATE_AVG(2024)` | 293.245, 460.28 |
| `NET_COST_LO(2024)`, `NET_COST_HI(2024)` | 89.72, 1,009.72 |
| `EXCESS_COST_LO(2024)`, `EXCESS_COST_HI(2024)` | about 1,539,057; about 17,320,737 |
| `NET_COST_LO_NOW`, `NET_COST_HI_NOW` | 63.24, 983.24 |
| `DAYS_SAVED_PER_PT` | 5.49 |
| `CEIL_PER_PT_LO`, `CEIL_PER_PT_HI` | 347.36; 5,400.74 |
| `STAY_SHORTFALL` | $5,873.50 |
| `RATE_RISE_2021`, `RATE_RISE_2024` | 18.3%; 29.5% |
| `RATE_COST_GAP_MIN`, `RATE_COST_GAP_MAX` | 19.45% (FY2023); 28.96% (FY2020) |
| `BEH_GUARD_SHARE_2024` | 34.7% (60 of 173) |
| `WL_DAYS_CHANGE_2024` | -26.7% |
| `T1_DROP_2021`, `T1_DROP_2024` | -1.92 points, -11.12 points (both rises; unrounded, the 2021 rise is 1.9 points; see Errors caught, 2026-09-28) |
| `T2_SLOPE` | about +1.88 days per patient per year |
| `T3_YEARS` | 5 (2017 to 2021) |
| `REASON_SUM(y) = DEC31_TOTAL(y)` | Every year except 2017 (120 vs 157) |
| Sum of `CTY_DAYS(c,y)` = `WL_DAYS(y)` | 2023 and 2024 |
| `KFF_COST_DAY` x 0.15 and x 0.40 | 549.60 and 1,465.60, within rounding of `AVOID_COST_LO` and `AVOID_COST_HI` |
| `CAP_RATIO(2024)`, `CAP_RATIO_STAFF_2024` | 11.69 (548 / 46.87); 7.53 (353 / 46.87) |
| `T1_FIN_CHANGE_2021`, `T1_OTHER_CHANGE_2021` | +13; +32 |
| `T1_FIN_CHANGE_2024`, `T1_OTHER_CHANGE_2024` | +15; -33 |
| `RATE_GROWTH_2024_2026` | +1.38% a year |
| `COST_FWD(2026)`, `COST_FWD(2027)` | $506.08; $522.78 |
| `COND_A_GAP(2026)`, `COND_A_GAP(2027)` | 2.14%; 3.95% |

(The table's current rows become 30 anchor rows on the `Checks` sheet; the two county-sum rows move to invariant I4. The new rows add 11 anchors, one per value: 41 in all. The offer-ceiling row adds 2: 43 in all. The days-saved and stay-shortfall rows add 2: 45 in all. The paper-figure rows add 5: 50 in all.)

**WorkedExample anchor.** The `WorkedExample` sheet has its own yellow inputs (not linked to `Inputs`), including its own benchmark (14) and test 1 threshold (5 points). Each case runs through the same formula shapes as the model and compares every result with the expected value below.

| Case | Inputs | Expected |
|---|---|---|
| A. Rising series | Years 1 to 3; 100 patients each year; days 1,000, 1,400, 2,000 | Days per patient 10, 14, 20; signed gap -4, 0, +6; excess days 0, 0, 600; days within 1,000, 1,400, 1,400; meets benchmark TRUE, TRUE, FALSE (14 exactly meets); slope +5.0; test 2 "Not met" |
| B. Falling series | Same patients; days 2,000, 1,400, 1,000 | Days per patient 20, 14, 10; slope -5.0; test 2 "Met" |
| C. Largest reason, tie | Reason counts: no bed 5, behavior 5, special care 2, financial 3, guardianship 1, PASARR 0, other 1 | "Tie (none largest)" |
| D. Largest reason, no tie | Same, with no bed 6 | "No bed" |
| E. Test 1 truth table | Drops in points (2021 window, 2024 window): (6, 6); (6, -2); (-2, 6); (-2, -2); (5.00, -2) | "Not falsified"; "Not falsified; weak (rests on the COVID-confounded 2021 comparison)"; "Not falsified"; "Falsified"; the "weak" text (a drop of exactly 5.00 points passes) |

Case E feeds the drops straight into the verdict formula; the share-to-drop step is covered by the regression anchors.

**Owner's manual audit.** I run these in Excel, reset the input after each step, and record the results in Audit findings. In every step the ALL CHECKS cell should still read ALL PASS; anchors that depend on the changed input FAIL, as they should.

| Step | Set | Expected |
|---|---|---|
| 1 | `BENCH_DAYS` = 21 | `EXCESS_DAYS(2023)` = 7,287; `EXCESS_DAYS(2024)` = 0; `CAP_RATIO(2024)` and `CAP_RATIO_STAFF_2024` blank; anchors on excess days, beds, gap, cost and capacity FAIL |
| 2 | `T1_MIN_DROP_PTS` = -12 | Both windows pass; `T1_VERDICT` = "Not falsified" (not the "weak" text, since both pass); all anchors match |
| 3 | `REASON_NO_BED(2023)` = 48 | 2023 `LARGEST_REASON` = "Tie (none largest)"; `T3_YEARS` stays 5; `T1_DROP_2024` = -11.84 points; `T1_OTHER_CHANGE_2024` = -39; the reasons-total, `T1_DROP_2024` and `T1_OTHER_CHANGE_2024` anchors FAIL |
| 4 | `AVOID_COST_LO` = 460.28 | `NET_COST_LO(2024)` = 0; `EXCESS_COST_LO(2024)` = 0; `NET_COST_LO_NOW` = -26.48; the low-cost anchors and the `AVOID_COST_LO` rounding anchor FAIL |

## Outputs

- By year, 2017 to 2024: `DAYS_PER_PT`, `GAP_PER_PT`, `WL_DAYS`, `EXCESS_DAYS`, `EXCESS_SHARE`, `EXCESS_BEDS`, `MEETS_BENCH`, the seven `SHARE_` values, `LTC_UNSTAFFED`, COVID flag.
- 2023, 2024 and now: `WL_RATE_AVG`, `NET_COST_LO/HI`, `EXCESS_COST_LO/HI` (and the `_NOW` versions); `NF_SHARE_DEC31`; `WL_DAYS_CHANGE_2024`.
- Offer ceiling: `DAYS_SAVED_PER_PT`, `CEIL_PER_PT_LO`, `CEIL_PER_PT_HI`.
- County row: `CTY_DAYS_PER_PT(c,y)` and `CTY_DAY_SHARE(c,y)`, 2023 and 2024.
- Tests: `T1_DROP_2021`, `T1_DROP_2024`, `T1_VERDICT`; `T2_SLOPE`, `T2_VERDICT`; `LARGEST_REASON(y)`, `T3_YEARS`, `T3_VERDICT`.
- Staffing note: `LTC_UNSTAFFED(2024)` and `LTC_UNSTAFFED_STAFF_2024`.
- Capacity: `CAP_RATIO(y)` for 2019, 2022, 2023 and 2024, `CAP_RATIO_STAFF_2024`, and the three limits.
- Side diagnostic: `T1_FIN_CHANGE_2021`, `T1_OTHER_CHANGE_2021`, `T1_FIN_CHANGE_2024`, `T1_OTHER_CHANGE_2024`.
- Condition (a): `RATE_GROWTH_2024_2026`; `COST_FWD(fy)`, `RATE_FWD(fy)` and `COND_A_GAP(fy)` for FY2026 and FY2027; `STAY_SHORTFALL`.
- Paper figures: `RATE_COST_GAP(fy)`, `RATE_COST_GAP_MIN`, `RATE_COST_GAP_MAX`, `RATE_RISE_2021`, `RATE_RISE_2024`, `BEH_GUARD_SHARE_2024`.

### Figures

Built from `FigureData` into `analysis/figures/` (PNG). File names `fig-a-days-per-patient.png`, `fig-b-reason-shares.png`, `fig-c-cost-vs-rate.png`, `fig-d-excess-days.png`, `fig-g-opportunity-cost.png`. Every figure has a title, labeled axes with units, a source line, and a caption that reads on its own. No repository URL on any figure.

| Fig | Content |
|---|---|
| a | `DAYS_PER_PT`, 2017 to 2024, as a line, with a horizontal line at `BENCH_DAYS` (14). Vertical markers at the rate events (`DATE_ADJ_2021`, `DATE_RESET_2024`). 2020 and 2021 marked COVID. |
| b | The seven `SHARE_` values by year, 2017 to 2024, as 100% stacked bars. The two test 1 windows (Dec 2020 to Dec 2021, Dec 2023 to Dec 2024) shaded. 2017 labeled "recorded reasons only (120 of 157)". Carries tests 1 and 3. |
| c | `NF_RATE_MEDIAN(d)` Jan 2016 to Jan 2026 as connected points at each memo's effective month. The line breaks at every January or July slot not in the rate series, so no value is drawn across a gap: Jan 2018 and Jan 2020 (the memo index lists no acuity rate table), Jul 2019 (QI-1911B omits its table), and Jan 2021 and Jul 2022 (QI-2041 and QI-2210 print tables, not collected in the series). `NF_COST_MCD_HEAVY(fy)` and `NF_COST_ALL(fy)`, FY2011 to FY2023, as lines at July 1 of each year, stopping at 2023. Rate events marked. Note on the chart that the rate median covers all homes and the cost medians the stated groups. |
| d | `WL_DAYS` by year, 2017 to 2024, as stacked bars: `WITHIN_DAYS` and `EXCESS_DAYS`. |
| g | Schematic, no dollar values: one bed's year as one long Medicaid stay, against the same bed turning over a run of short Medicare stays. The caption states that test 1 did not support this mechanism. |

In the paper, figures a, c and b are Figure 1, Figure 2 and Figure 3, in order of first mention, and their captions carry those numbers. Figure b's caption also says that "No bed" has not been the largest reason since 2021. Figures d and g are not used in the paper and keep their letters. File names are unchanged.

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

**Note (2026-10-01).** The paper's recommendation departs from test 3's verdict (Met), so criterion 5 is not met as written. The departure and my reason are stated in the paper and recorded in the brief's Correction of 2026-10-01. No test, definition or verdict changed.

## Audit findings

Build of 2026-09-29, by Claude Code from this spec (main at `3641f5d`), on branch `research-model-build`. Every value is still a draft until I verify it against its source (the SHPDA counts, the OCR rate medians, the 2017 reason totals, the Medicaid-heavy rule, $3,664, the 543 days, the CAH share).

Add to my verification list: the FY2023 cost-report periods (six reports run past the January 2024 reset); the market-basket values in `MB_GROSS`.

### What was checked

| Check | Method | Result |
|---|---|---|
| Hand checks and relational rules | 32 rows on the `Checks` sheet, evaluated by the `formulas` engine, then in Excel | 32 of 32 PASS in the engine; I opened the workbook in Excel: no errors, every row PASS |
| Calculation logic | Independent Python re-implementation from the scratch CSVs, compared output by output with the engine | 195 outputs, 0 mismatches |
| Inputs | Every `Inputs` cell compared with this spec's own tables and values (parsed from this file, not the CSVs); rate and cost series by count, endpoints and the blank 2019-07 median | 154 comparisons, 0 differences |
| Figure c rate slots | 21 January/July slots, Jan 2016 to Jan 2026, against the rate series | 0 mismatches; breaks at Jan 2018, Jul 2019, Jan 2020, Jan 2021, Jul 2022 |
| Formulas only outside `Inputs` | Script | 0 typed numbers outside `Inputs` |
| Yellow fill | Script | Every input value yellow; no formula or cell outside `Inputs` yellow |
| Error cells | Engine | 0 |
| Names | Explicit list from this spec | 76 names, all absolute: the 71 in this spec (with `SHARE_<reason>` expanded to seven names) plus 5 I approved (`YEAR`, `RATE_DATE`, `COST_FY`, `COUNTY`, `COVID_FLAG`); none missing, none extra |
| Clipped text | Script estimate of display width against column width and row height | 0 |

### What was found

- Results: days per patient 23.02 (2023) and 19.49 (2024); excess days 32,557 and 17,154; beds 89.2 and 46.9; signed gap -4.06 (2017) and +5.49 (2024); average waitlisted rate 293.245 and 460.28; 2024 net cost 89.72 to 1,009.72 per day; test 1 drops -1.92 and -11.12 points; test 2 slope +1.88; test 3, 5 years (2017 to 2021). All match the hand checks above.
- Verdicts: test 1 "Falsified"; test 2 "Not met"; test 3 "Met", with 2017 flagged.
- No calculation error found. The first build displayed some values finer than the Precision convention (signed gap and test 2 slope to 2 decimals, the average waitlisted rate to 3 decimals, test 1 shares and drops to 0.01 point); corrected. The first figures used "Health Care Utilization Reports" in the source lines and had figure c's note only in the caption; corrected to this spec's wording and a note on the chart.

### What was done, and build conventions not stated above

- Hand-check expected values are constants inside the `Checks` formulas (no typed values outside `Inputs`); tolerance is half a unit of the precision shown in the hand-check table. My decision.
- Series formulas read names by position (`INDEX(name,k)` or `INDEX(name,MATCH(year,YEAR,0))`), never a bare range name in a cell, so Excel 365 cannot spill or add `@`.
- Figure c's missing slots are empty strings in `FigureData`, not `#N/A`, so the line breaks without error cells.
- The 2017 to 2022 cells of `DEC31_NF`, `WL_RATE_JAN` and `WL_RATE_JUL` are blank and not yellow (no input exists). My decision.
- Index columns on `Inputs` (`YEAR`, `RATE_DATE`, `COST_FY`, `COUNTY`, memo IDs) are gray row headers, not yellow, and the blank 2019-07 `NF_RATE_MEDIAN` cell is yellow (an input left blank). Claude's call, not yet reviewed by me.
- Figure a plots each year at July 1, with the rate events at their dates. Figure g shows six illustrative Medicare stays, with no durations, no dollar values and no `FigureData` block. My decisions.
- Figure c's axis and caption say "nominal" dollars; to verify against the cost-report extract.
- Figure captions are Claude's factual drafts, pending my review.
- Build and check scripts are kept outside the repo, in my Research Paper folder (`model-build-scripts-v2`).

### Lean audit build (2026-09-29)

Rebuild by Claude Code after a model audit (findings kept outside the repo) and my decisions on it, trimmed to the lean set: the Checks split into regression anchors, invariants and error scans; the `WorkedExample` sheet and the owner's manual audit; verification columns on `Inputs`; the capacity ratio; test 1 on counts; condition (a); the cost-report-periods line. Branch `research-model-lean-audit`, from main at `24b0767`. No test, definition or verdict changed.

**What was checked**

| Check | Method | Result |
|---|---|---|
| Checks sheet | Evaluated by the `formulas` engine, then in Excel | ALL CHECKS reads ALL PASS; anchors matching 41 of 41; invariants I1 to I9 PASS; 10 error scans at 0. I opened the workbook in Excel: no errors, ALL PASS |
| WorkedExample | Engine | 29 of 29 rows match (cases A to E, including the tie, the "weak" test 1 text and test 2 "Met") |
| Calculation logic | Independent Python re-implementation from the scratch CSVs and this spec's literal market-basket values | 217 outputs, 0 mismatches |
| Inputs | Every `Inputs` value against this spec's tables and values, including `MB_GROSS` | 158 comparisons, 0 differences |
| Names | Explicit list from this spec | 87 names, all absolute: the 82 in this spec plus the 5 approved extras; none missing, none extra |
| Typed values and fill | Script | Typed numbers only on `Inputs` and the `WorkedExample` inputs; every input yellow; no formula yellow |
| Clipped text | Script estimate | 0 |
| Owner's manual audit | Simulated in the engine on copies of the workbook, one step at a time | Every step as specified, with ALL CHECKS at ALL PASS throughout: step 1, excess days 7,287 (2023) and 0 (2024), capacity ratios blank, 10 anchors FAIL; step 2, "Not falsified", 41 of 41 anchors; step 3, 2023 a tie, `T3_YEARS` 5, `T1_DROP_2024` -11.84 points, `T1_OTHER_CHANGE_2024` -39, 3 anchors FAIL; step 4, `NET_COST_LO(2024)` 0, `EXCESS_COST_LO(2024)` 0, `NET_COST_LO_NOW` -26.48, 4 anchors FAIL. My own run in Excel is still to do |

**What was found**

- New outputs from the draft inputs: `CAP_RATIO(2024)` 11.69 and `CAP_RATIO_STAFF_2024` 7.53; test 1 on counts +13 and +32 (2020 to 2021), +15 and -33 (2023 to 2024); `RATE_GROWTH_2024_2026` 1.38% a year; `COST_FWD` $506.07 (FY2026) and $522.77 (FY2027); `COND_A_GAP` 2.14% and 3.95%. Verdicts unchanged: test 1 "Falsified", test 2 "Not met", test 3 "Met".
- No calculation error found. The formulas engine treats a name that refers to its own range as circular, so `COST_FWD` multiplies the FY2023 base by the market-basket factors directly, and `RATE_FWD(2027)` looks up the January 2026 rate directly; the values are the same as a year-to-year chain.

**What was done, and build conventions not stated above**

- Verification columns sit in columns P to R of `Inputs` on every input row (58 rows), past the widest table, so the long source text stays readable; the count is at the top of `Inputs`.
- Anchor results read FAIL, not an error, when a value is blank (for example the capacity ratios in manual-audit step 1), so a blank never trips an error scan.
- The figures were not rebuilt: `FigureData` is unchanged, and its figure c rate slots were re-checked (0 mismatches).
- Build and check scripts, extended, are in my Research Paper folder (`model-build-scripts-v2`, with `manual_audit.py`).

### Input verification (2026-09-29)

I verified the inputs against their primary sources, with Claude Code finding and opening each source and re-deriving derived values. Four values were wrong by one cent and are corrected: the January 2026 rate median (495.25 to 495.26; the median 495.255 had been rounded down) and three cost medians (FY2014 all facilities 344.18 to 344.19, FY2015 Medicaid-heavy 322.11 to 322.10, FY2023 Medicaid-heavy 461.78 to 461.79; the build had taken medians of values already rounded to the cent). The `COST_FWD` anchors moved to $506.08 and $522.78. No test, definition or verdict changed. My verification entries are on `Inputs`, columns P to R.

### Offer ceiling build (2026-09-30)

Rebuild by Claude Code after I approved two new outputs, `CEIL_PER_PT_LO` and `CEIL_PER_PT_HI`, on the `Cost` sheet. Branch `research-offer-ceiling`, from main at `efb1c77`. No input, test, definition or verdict changed.

| Check | Method | Result |
|---|---|---|
| Checks sheet | Evaluated by the `formulas` engine | ALL CHECKS reads ALL PASS; anchors matching 43 of 43; 62 rows PASS, 0 FAIL. I opened the workbook in Excel on 2026-09-30 and the check was good |
| Calculation logic | Independent Python re-implementation from the scratch CSVs | 219 outputs, 0 mismatches |
| Inputs | Every `Inputs` value against this spec's tables and values | 158 comparisons, 0 differences |
| Names | Explicit list from this spec | 89 names, all absolute: the 84 in this spec plus the 5 approved extras; none missing, none extra |
| Verification entries | Columns P to R of `Inputs` carried from the workbook on main into the rebuild; columns A to O compared cell by cell | 173 cells on 58 rows carried; 0 differences in A to O; the count reads 57 of 58 |
| Owner's manual audit | Simulated in the engine | ALL CHECKS at ALL PASS in every step; step 4 now also fails the `CEIL_PER_PT_LO` anchor (5 anchors FAIL) |

New outputs from the current inputs: `CEIL_PER_PT_LO` $1,232.72 and `CEIL_PER_PT_HI` $19,166.10 per placed patient. Verdicts unchanged: test 1 "Falsified", test 2 "Not met", test 3 "Met". The figures were not rebuilt (`FigureData` is unchanged). The clipped-text estimate flags 40 cells in column P of `Inputs`: my typed source-page entries are longer than the column, as on main.

### Figure numbers (2026-09-30)

Rebuild of figures a, b and c by Claude Code with the paper's numbers in their captions (Figure 1, Figure 3 and Figure 2) and the added phrase in figure b's caption. Branch `research-figure-numbers`, from main at `a67dd3e`. `FigureData` and the charts themselves are unchanged; only caption text changed.

| Check | Method | Result |
|---|---|---|
| Match with the paper | SHA-256 of each rebuilt file against the chart embedded in my Word draft v3 | `fig-a-days-per-patient.png`, `fig-c-cost-vs-rate.png` and `fig-b-reason-shares.png` are byte-identical to the draft's Figure 1, 2 and 3 |
| Unused figures | SHA-256 before and after the rebuild | `fig-d-excess-days.png` and `fig-g-opportunity-cost.png` unchanged |

### Admission window and median stay build (2026-10-01)

Rebuild by Claude Code after my instructor's review (PR #85, items 2 and 3) and my approval of the wording. Two new inputs, `ADMIT_WINDOW_DAYS` (14, my judgment) and `MEDIAN_MCD_LOS` (542.83); new outputs `DAYS_SAVED_PER_PT` and `STAY_SHORTFALL`; `CEIL_PER_PT_LO` and `CEIL_PER_PT_HI` restated on the days saved. At my direction, the numbers the paper computes that had no cell were added as the Paper figures block on `Conditions`: `RATE_COST_GAP`, `RATE_COST_GAP_MIN`, `RATE_COST_GAP_MAX`, `RATE_RISE_2021`, `RATE_RISE_2024` and `BEH_GUARD_SHARE_2024`. Branch `research-test3-departure`. No test, definition or verdict changed.

| Check | Method | Result |
|---|---|---|
| Checks sheet | Evaluated by the `formulas` engine | ALL CHECKS reads ALL PASS; anchors matching 50 of 50; 69 rows PASS, 0 FAIL. I opened the workbook in Excel on 2026-10-01 and it checked out |
| Calculation logic | Independent Python re-implementation from the scratch CSVs and this spec's literal values | 234 outputs, 0 mismatches |
| Inputs | Every `Inputs` value against this spec's tables and values | 160 comparisons, 0 differences |
| Names | Explicit list from this spec | 99 names, all absolute: the 94 in this spec plus the 5 approved extras; none missing, none extra |
| Verification entries | Columns P to R of `Inputs` carried from the workbook on main, matched by each row's content | 173 cells on 58 rows carried, none unmatched; the count reads 57 of 60. The two new input rows are mine to verify |
| `MEDIAN_MCD_LOS` | Re-derived by Claude Code on 2026-10-01 from the CMS FY2023 file on data.cms.gov (37 Hawaii rows, 34 homes, each home's latest report) | Median 542.83 days over the 31 homes reporting a value (the Avalon Care Center report); range 8.26 to 1,680.75; 3 homes blank |

New outputs from the current inputs: `DAYS_SAVED_PER_PT` 5.49 days; `CEIL_PER_PT_LO` $347.36 and `CEIL_PER_PT_HI` $5,400.74 per placed patient (they were $1,232.72 and $19,166.10 on the full average wait); `STAY_SHORTFALL` $5,873.50; `RATE_RISE_2021` 18.3% and `RATE_RISE_2024` 29.5%; `RATE_COST_GAP` from 19.45% (FY2023) to 28.96% (FY2020); `BEH_GUARD_SHARE_2024` 34.7%. Verdicts unchanged: test 1 "Falsified", test 2 "Not met", test 3 "Met". The figures were not rebuilt (`FigureData` is unchanged).
