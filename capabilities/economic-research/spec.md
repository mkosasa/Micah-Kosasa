---
type: spec
capability: economic-research
engagement: research-paper
date: 2026-09-28
status: built            # draft | built | audited
built_with: "Claude Code, from this file"
---

# Economic research — model specification

**Sources.** Every input below traces to one of these sources, to a formula on other inputs, or to a logged decision (decision numbers refer to `decision tree.md`). Figures carried over from the research notes are drafts until checked against the source. Local files are in the `snf-cost-reports` folder. Values marked **Pull at build** are read from those files or URLs by the session that builds the model, under the rules in "Values the build pulls."

| Source | Used for | Where |
|---|---|---|
| SHPDA Table 18, waitlisted patients in acute-care beds, 2023 and 2024 | Waitlist days and patients for the year, statewide and by hospital; Dec 31 snapshot by care needed and by reason | 2023: https://health.hawaii.gov/shpda/files/2024/09/2023UR-Table-18-Wait-Listed-Patients-in-Acute-Care-Beds.pdf ; 2024: https://health.hawaii.gov/shpda/files/2025/09/2024UR-Table-18-Wait-Listed-Patients-in-Acute-Care-Beds.pdf ; local `shpda-2023-table18-waitlisted-acute.pdf`, `shpda-2024-table18-waitlisted-acute.pdf`, `shpda-table18-statewide-2023-2024.csv` |
| SHPDA Table 18, earlier years | Reasons test, outside the model | 2022: https://health.hawaii.gov/shpda/files/2023/09/2022UR-Table-18-Wait-Listed-Patients-in-Acute-Care-Beds-1.pdf ; 2015 to 2021 to find |
| SHPDA Table 19, waitlisted patients in long-term-care beds, 2022 to 2024 | Onward placement after the transitional stay | 2024: https://health.hawaii.gov/shpda/files/2025/09/2024UR-Table-19-Wait-listed-Patients-in-Long-Term-Care-Beds.pdf ; 2023: https://health.hawaii.gov/shpda/files/2024/09/2023UR-Table-19-Wait-listed-Patients-in-Long-Term-Care-Beds.pdf ; 2022: https://health.hawaii.gov/shpda/files/2023/09/2022UR-Table-19-Wait-listed-Patients-in-Long-Term-Care-Beds-1.pdf ; local `shpda-2023-table19-waitlisted-ltc.pdf`, `shpda-2024-table19-waitlisted-ltc.pdf` |
| SHPDA Table 1, licensed acute care bed capacity, 2023 and 2024 | Paper only (success criteria), not the model: the acute-bed denominator for the beds held by waitlisted patients. Licensed acute beds (medical/surgical, critical care, obstetric, pediatric, neonatal ICU, psychiatric) were 2,479 in both years; the table's total of 2,598 adds 119 acute/long-term swing beds (draft; verify). Licensed, not staffed: the staffed count is probably lower | 2023: https://health.hawaii.gov/shpda/files/2024/09/2023UR-Table-1-Licensed-Acute-Care-Bed-Capacity.pdf ; 2024: https://health.hawaii.gov/shpda/files/2025/09/2024UR-Table-1-Licensed-Acute-Care-Bed-Capacity.pdf |
| CMS SNF cost reports, FY2011 to FY2023 (Worksheet A) | Nursing-home cost per day; capital-related cost per day; Medicaid days; salary share | https://data.cms.gov/provider-compliance/cost-reports/skilled-nursing-facility-cost-report ; local `hi-snf-cost-report-fy2011.csv` to `hi-snf-cost-report-fy2023.csv`, `hi-snf-cost-per-day-series-2011-2023.csv`, `hi-snf-medicaid-rate-vs-cost-2023.csv`, `extract_hi_snf.py` |
| Med-QUEST rate memos, Jan 2016 to Jan 2026 (QI-1521 to QI-2532) | Median nursing-facility rate; waitlisted hospital rate; index updates | Memo index: https://medquest.hawaii.gov/content/medquest/en/plans-providers/provider-memo.html (each memo's URL is in the research notes, section 7); local `hi-medicaid-rate-series-2016-2026.csv`, `rate_summary.json`, `medquest-QI-2532-rates-2026.pdf`, `medquest-QI-2326-rates-jul2023.pdf` |
| State Plan Amendment HI-23-0014, Attachment 4.19-D | Rate components, including the capital component; the index; the update method; the capital-cost proxy (p. 38a: capital component price $22.50, 120% of median) | https://www.medicaid.gov/sites/default/files/2024-02/HI-23-0014.pdf ; local `medicaid-SPA-HI-23-0014-NF-rate-method.pdf` |
| CMS SNF Prospective Payment System final rules, FY2024 to FY2027 (fact sheets) | `G_IDX`: the SNF market basket, standing in for the S&P index | FY2024 (CMS-1779-F): https://www.cms.gov/newsroom/fact-sheets/fiscal-year-fy-2024-skilled-nursing-facility-perspective-payment-system-final-rule-cms-1779-f ; FY2025 (CMS-1802-F): https://www.cms.gov/newsroom/fact-sheets/fiscal-year-2025-skilled-nursing-facility-prospective-payment-system-final-rule-cms-1802-f ; FY2026 (CMS-1827-F): https://www.cms.gov/newsroom/fact-sheets/fy-2026-skilled-nursing-facility-snf-prospective-payment-system-final-rule-cms-1827-f ; FY2027 (CMS-1843-F): https://www.cms.gov/newsroom/fact-sheets/fiscal-year-2027-skilled-nursing-facility-prospective-payment-system-final-rule-cms-1843-f |
| KFF/AHA, hospital expenses per adjusted inpatient day, Hawaii nonprofit hospitals, 2023 | The $3,664 average behind the avoidable-cost range (an upper bound only) | https://www.kff.org/health-costs/state-indicator/expenses-per-inpatient-day/ |
| Roberts et al. (1999), "Distribution of variable vs fixed costs of hospital care," JAMA 281(7): 644 to 649 | Basis for the low end of the avoidable-cost range: 16% of hospital cost is variable in the short run (draft; from a search summary, verify in the paper) | https://pubmed.ncbi.nlm.nih.gov/10029127/ |
| Taheri, Butz and Greenfield (2000), "Length of stay has minimal impact on the cost of hospital admission," Journal of the American College of Surgeons | Support for the low end: one fewer day cuts an admission's total cost by 3% or less (draft; verify) | https://pubmed.ncbi.nlm.nih.gov/10945354/ |
| MedPAC (March 2015), chapter 3 online appendixes, hospital inpatient and outpatient services | Context for the high end: Medicare's outlier payments use a marginal-cost factor of 80% (draft; verify) | https://www.medpac.gov/wp-content/uploads/import_data/scrape_files/docs/default-source/reports/chapter-3-online-only-appendixes-hospital-inpatient-and-outpatient-services-march-2015-report-.pdf |
| Medicare.gov, skilled nursing facility care | The Medicare run: Part A pays days 1 to 20 of a SNF stay in full after a 3-day inpatient stay; days 21 to 100 carry a $217 daily coinsurance (draft; verify) | https://www.medicare.gov/coverage/skilled-nursing-facility-care |
| SHPDA Certificate of Need applications, nursing-facility bed projects, 2008 to 2024 (13 read, 12 used in the medians: 08-08, 09-03, 10-12, 11-01, 13-18A, 14-07, 15-08, 18-07A, 18-08, 20-21, 21-10A, 22-21A, 24-10A; 12-20A not downloadable) | Capital cost per bed, converted space vs. a new building (`K`) | https://health.hawaii.gov/shpda/certificate-of-need-applications-and-decisions/ (each application's URL is in the local file); local `shpda-con-nf-bed-project-costs.csv` |
| BLS Quarterly Census of Employment and Wages, health-care wages, Hawaii and US, 2024 and 2025 | Wage growth for the staffing run | https://www.bls.gov/cew/ ; local `bls-qcew-hi-us-health-wages-2024-2025.csv` |
| Capital Group, "Municipal bonds: 5 investment themes for 2026" | Discount rate: 30-year tax-exempt bonds rated A or better yield about 4.0 to 4.5% (mid-2026) (draft; from a search summary, verify) | https://www.capitalgroup.com/pcs/insights/articles/2026-municipal-bond-themes.html |
| FRED, 10-year breakeven inflation rate (T10YIE) | Discount rate: expected inflation, 2.28% on Sep 25, 2026 (draft; verify) | https://fred.stlouisfed.org/series/T10YIE |
| OMB memorandum M-25-15 (2025), reinstating Circular A-4 (2003) | Discount-rate runs: 3% and 7% real | https://www.whitehouse.gov/wp-content/uploads/2025/03/M-25-15-Recission-and-Reinstatement-of-Circular-A-4.pdf |
| Federal medical assistance percentage (FMAP), Hawaii | Budget view: the federal match | MACPAC, MACStats (Feb 2026), Exhibit 6, FMAPs by state, FYs 2023 to 2026: https://www.macpac.gov/wp-content/uploads/2026/01/EXHIBIT-6.-Federal-Medical-Assistance-Percentages-and-Enhanced-Federal-Medical-Assistance-Percentages-by-State-FYs-2023%E2%80%932026.pdf |
| BLS CPI-U, all items, Urban Hawaii (series CUUSA426SA0), annual averages | The deflator to 2023 dollars (decision 47) | https://fred.stlouisfed.org/series/CUUSA426SA0 |

## Purpose
This model supports one decision: how many short-stay transitional nursing-home beds (about 30-day stays) the State of Hawaii should back through a hospital partnership, in which a consortium of hospitals funds the capital the Medicaid rate does not cover, under a binding contract, and the state enforces the contract, speeds certificate-of-need approval and adopts a statewide rate rule. Beds are added cheapest first while the system-wide saving from the next bed is at least its full resource cost.

It must report:
- the bed count, and what stops it there: the end of the pool of patients a bed can take, the cost of the next rung of capacity, or no bed paying at all;
- the break-even avoidable hospital cost for the first beds and for each rung;
- whether the new beds pass the operating test now and in each later year as Hawaii costs drift from the index, with and without the rate rule;
- the grant per bed-year, how the consortium's cost splits among hospitals, whether each hospital would join, and whether any one hospital would fund beds alone;
- each party's cash change per bed-year: the state, the federal government and the hospitals;
- the cost of the partnership, the rate rule (both forms) and a one-time statewide rate increase, per waitlist day freed and per bed-equivalent (decision 51);
- all of the above for every scenario run, for the 2023 baseline and the 2024 comparison year.

## Definitions
- **Waitlist day:** a day a patient who no longer needs acute care spends in an acute-care hospital bed waiting for placement (SHPDA Table 18).
- **Freed day:** a waitlist day that does not happen because a transitional bed admitted the patient.
- **Bed-year:** one transitional bed run for a year at `OCC` occupancy, which is `OCC_DAYS` occupied days.
- **Pool:** the waitlist days a bed that accepts Medicaid can clear. It is the share of the Dec 31 snapshot waiting because no bed was available or for financial, Medicaid or insurance reasons (`POOL_SHARE`), applied to the year's waitlist days. The other groups (psychiatric, dementia or behavior; special care; family or guardianship; other) need specialized staffing or a legal decision, not just an empty bed.
- **Rung:** a block of beds with one capital cost per bed. The base follows the brief's ladder (decision 61): (1) converted existing space, then (2) a new building. How much convertible space exists is unknown, so the base prices every bed as converted space and the new-building run prices every bed as a new building (decision 62). Beds are added cheapest rung first. (An empty-bed run that put existing empty certified beds in front as rung 0 was cut, decision 77.)
- **Running cost:** the cost per occupied day in the resource view (`C_FULL`), priced at the FY2023 median total cost, which includes the existing homes' capital (decision 34).
- **Resource view:** real costs and savings to Hawaii as a whole. Medicaid payments, the waitlisted rate and the federal match are transfers between parties and drop out. It sets the bed count (decision 21).
- **Budget view:** each party's cash: the state, the federal government, the hospitals, the consortium and the nursing homes. It sets the cost split and never feeds the bed count (decision 21).
- **Avoidable cost:** what a hospital stops spending when a waitlisted patient leaves a day sooner (`C_AVOID`). It is not the average cost per day, which includes overhead that does not go away when one low-acuity patient leaves.
- **Break-even avoidable cost:** the avoidable cost at which a bed's yearly saving equals its full resource cost (`BREAKEVEN(k)` for rung `k`).
- **Capital component:** the part of the Medicaid nursing-facility rate that pays for buildings (depreciation, interest, rent and property costs), paid on every occupied day (`R_CAP`).
- **Grant:** a bed's annualized capital cost minus what the capital component pays on its occupied days, floored at zero (`GRANT(k)`). The consortium pays it (decisions 12 and 23).
- **Operating test:** the shutdown test (P ≥ AVC) with capital taken out of both sides: the new bed's operating cost per occupied day excluding capital, against the rate excluding its capital component.
- **Drift:** the gap between Hawaii nursing-home cost growth (`G_HI`) and growth in the national index the rate follows (`G_IDX`).
- **Rate rule:** a statewide rule that keeps the rate tracking Hawaii cost: a periodic rebase or a Hawaii-specific index (form open, decision 11).
- **Planning share and contract share:** a hospital's share of the consortium's cost before the beds open (its waitlist days) and once they run (the days its discharged patients use, reconciled yearly) (decision 28). The model computes the planning share; the contract share is settled from real use.
- **Constant 2023 dollars:** the model's price year (decision 33). "Nominal" means the dollars of the year stated.

## Inputs — the named contract

### Waitlist (SHPDA Table 18)
`Y` is the waitlist year: 2023 is the baseline and 2024 the after-reset comparison (decision 18). Values are given as 2023; 2024.

| Name | Value | Unit | Source |
|---|---|---|---|
| `WAIT_DAYS(Y)` | 83,097; 60,876 | waitlist days in the year | Sourced: Table 18 (draft) |
| `WAIT_PATIENTS(Y)` | 3,610; 3,123 | waitlisted patients in the year | Sourced: Table 18 (draft) |
| `SNAP_TOTAL(Y)` | 191; 173 | patients waitlisted on Dec 31 | Sourced: Table 18 (draft) |
| `SNAP_NOBED(Y)` | 42; 31 | Dec 31 patients, reason "no bed available" | Sourced: Table 18 (draft) |
| `SNAP_FIN(Y)` | 45; 60 | Dec 31 patients, financial, Medicaid or insurance | Sourced: Table 18 (draft) |
| `SNAP_BEHAV(Y)` | 48; 29 | Dec 31 patients, psychiatric, dementia or behavior | Sourced: Table 18 (draft) |
| `SNAP_SPECIAL(Y)` | 26; 18 | Dec 31 patients, special care | Sourced: Table 18 (draft) |
| `SNAP_FAMILY(Y)` | 28; 31 | Dec 31 patients, family or guardianship | Sourced: Table 18 (draft) |
| `SNAP_OTHER(Y)` | 2; 4 | Dec 31 patients, pending PASARR plus other | Sourced: Table 18 (draft); 2023 is 1 + 1, 2024 is 0 + 4 |
| `SNAP_NF_LEVEL(Y)` | 157; 120 | Dec 31 patients needing SNF/ICF-level care | Sourced: Table 18 (draft) |
| `DAYS_AVOIDED(Y)` | `WAIT_DAYS(Y) / WAIT_PATIENTS(Y)` = 23.0186; 19.4928 | waitlist days avoided per transitional admission | Derived. The brief rounds 2023 to 23 |
| `POOL_SHARE(Y)` | `(SNAP_NOBED(Y) + SNAP_FIN(Y)) / SNAP_TOTAL(Y)` = 87 / 191 = 0.4555; 91 / 173 = 0.5260 | fraction of waitlist days a bed can clear | Derived. Assumes the Dec 31 mix holds all year and the financial group is mostly Medicaid refusals (brief); contested by the 2024 snapshot (decision 19) |
| `WAIT_DAYS_H(i, Y)` | one value per hospital `i`: see Hospital rows below | waitlist days in the year | Sourced: Table 18 hospital rows, 2023 and 2024 (draft; read from the PDF text layer; verify) |
| `STATE_RUN_H(i)` | 1 for Hilo, Kona and Kauai Veterans; 0 for the rest: see Hospital rows below | flag: state-run hospital (Hawaii Health Systems Corporation) | Sourced: HHSC's facility list, https://www.hhsc.org/ (Hilo and Kona listed; Kauai Veterans, in HHSC's Kauai region, to verify). Kahuku is an HHSC affiliate, flagged 0. State-run share of waitlist days: 33.0% (2024), 28.1% (2023); 34.9% in 2024 if Kahuku counts |

**Hospital rows (SHPDA Table 18).** Patients and days in the year; Dec 31 is the number waitlisted that day. Each year's rows sum to `WAIT_PATIENTS(Y)` and `WAIT_DAYS(Y)`. Local file: `shpda-table18-by-hospital-2023-2024.csv`. The 2023 Queen's Punchbowl row reads out of order in the PDF text layer; its values here are Honolulu County's totals less the other Honolulu hospitals. Hospitals with no waitlisted patients in a year are not listed in the table.

| Hospital `i` | County | `STATE_RUN_H` | 2023 patients | 2023 days | 2023 Dec 31 | 2024 patients | 2024 days | 2024 Dec 31 |
|---|---|---|---|---|---|---|---|---|
| Hilo Benioff Medical Center (Hilo Medical Center in 2023) | Hawaii | 1 | 563 | 12,712 | 32 | 490 | 12,235 | 40 |
| Kona Community Hospital | Hawaii | 1 | 300 | 10,017 | 22 | 235 | 7,597 | 25 |
| North Hawaii Community Hospital | Hawaii | 0 | 6 | 483 | 1 | 2 | 342 | 0 |
| Adventist Health Castle | Honolulu | 0 | 156 | 739 | 1 | 100 | 624 | 3 |
| Kahuku Medical Center | Honolulu | 0 | 15 | 718 | 3 | 33 | 1,129 | 2 |
| Kaiser Foundation Hospital | Honolulu | 0 | 342 | 3,341 | 6 | 354 | 3,370 | 14 |
| Kapiolani Medical Center for Women and Children | Honolulu | 0 | 15 | 1,028 | 3 | 16 | 690 | 4 |
| Kuakini Medical Center | Honolulu | 0 | 645 | 4,447 | 13 | 602 | 4,382 | 12 |
| Pali Momi Medical Center | Honolulu | 0 | 237 | 3,391 | 7 | 249 | 2,036 | 8 |
| Straub Benioff Medical Center (Straub Clinic & Hospital in 2023) | Honolulu | 0 | 249 | 3,545 | 4 | 199 | 2,305 | 6 |
| The Queen's Medical Center, Punchbowl | Honolulu | 0 | 519 | 20,280 | 45 | 383 | 12,099 | 25 |
| The Queen's Medical Center, West Oahu | Honolulu | 0 | 77 | 2,838 | 7 | 43 | 745 | 1 |
| Kauai Veterans Memorial Hospital | Kauai | 1 | 20 | 590 | 1 | 4 | 256 | 1 |
| Wilcox Medical Center | Kauai | 0 | 66 | 1,130 | 3 | 60 | 1,671 | 3 |
| Maui Memorial Medical Center | Maui | 0 | 399 | 17,812 | 42 | 349 | 11,267 | 28 |
| Molokai General Hospital | Maui | 0 | 1 | 26 | 1 | 4 | 128 | 1 |
| **Total** | | | **3,610** | **83,097** | **191** | **3,123** | **60,876** | **173** |

### Transitional bed
| Name | Value | Unit | Source |
|---|---|---|---|
| `OCC` | 0.90 | fraction of bed-days occupied | Assumed (brief) |
| `STAY_DAYS` | 30 | days per transitional stay | Assumed (brief); runs at 60 and 90 (decision 20) |
| `OCC_DAYS` | `OCC x 365` = 328.5 | occupied days per bed-year | Derived |
| `B_MAX` | 400 | beds in the bed schedule | Assumed: above the roughly 330 beds that would absorb every 2023 waitlist day |
| `PAST_POOL_PCT` | 0 in the base; runs at 0.25 and 0.5 | freed days of a bed past the pool, as a share of a pool bed's | Assumed (decision 37). 0 is the brief's mechanism taken literally: past the pool, a bed frees nothing. The runs are illustrative |

### Hospital side
| Name | Value | Unit | Source |
|---|---|---|---|
| `HOSP_AVG_COST` | 3,664 | USD per adjusted inpatient day, 2023 | Sourced: KFF/AHA (draft). An upper bound only; never used as the avoidable cost |
| `C_AVOID` | base 1,010; runs at 550 and 1,470 | USD per freed day, 2023 $ | Assumed. Range set in advance at 15 to 40% of `HOSP_AVG_COST` (decision 24); base at the midpoint, the brief's "near the middle of my range" (decision 38). Basis for the ends (decision 39): the low end rests on Roberts et al. (1999), 16% of hospital cost variable in the short run, supported by Taheri et al. (2000). The high end is judgment, between that 16% and the 80% marginal-cost factor Medicare uses for outlier cases (MedPAC 2015), lowered because a waitlisted patient uses few services |
| `C_AVOID_H(i)` | `C_AVOID` for every hospital in the base; stress run: one hospital at 550, the rest at the base | USD per freed day, 2023 $ | Assumed (decision 40). Avoidable cost is private to each hospital and no data separates hospitals. Shares cancel out of the join test, so which hospital is at 550 doesn't matter |
| `ACUTE_MARGIN` | 0 in every run | USD per freed day, 2023 $ | Assumed (decision 56). Conservative: a freed acute bed earns extra margin only when a hospital is full, and the model has no hospital occupancy or margin data, so savings and the count are floors |

### Nursing-home cost and Medicaid rates
| Name | Value | Unit | Source |
|---|---|---|---|
| `NF_COST_2023` | 462 | USD per patient day, FY2023, total including capital | Sourced: CMS cost reports FY2023, median of 18 Medicaid-heavy freestanding homes (at least 50% Medicaid days; Kulana Malama and Islands Skilled excluded as complex care); Worksheet A total expenses / total patient days (draft) |
| `NF_CAP_COST_2023` | 18.75 | USD per patient day, 2023 $ | Derived proxy (Micah's decision): the SPA HI-23-0014 capital component price, $22.50, is 120% of the day-weighted median capital cost per day for the 2023 rate base (p. 38a), so 22.50 / 1.2 = 18.75. It covers all homes in the rate setting, not the 18 Medicaid-heavy homes; the CMS cost-report file has no capital-cost lines (draft; verify) |
| `C_FULL` | `NF_COST_2023 + WAGE_PREMIUM` = 462 in the base | USD per occupied day, 2023 $; running cost in the resource view, including the existing homes' capital | Decision 34: the brief's basis, kept as drafted. The staffing run adds the premium (decision 48) |
| `C_OP` | `NF_COST_2023 - NF_CAP_COST_2023 + WAGE_PREMIUM` = 462 - 18.75 = 443.25 in the base | USD per occupied day, 2023 $, excluding capital; used only by the operating test | Derived |
| `SALARY_SHARE` | 0.43 | fraction of Worksheet A expense | Sourced: FY2023 cost reports, median (draft). Benefits and contract labor sit outside it |
| `G_HI` | `(462 / 288) ^ (1/12) - 1` = 0.0402 | per year | Derived: median cost per day, $288 (FY2011) to $462 (FY2023) (draft). The 15 homes present in both years grew 4.7% a year |
| `R_NF(t)` | 481.83 (Jan 2024); 509.69 (Jan 2025); 495.25 (Jan 2026) | USD per day, nominal; median nursing-facility rate | Sourced: QI-2342, QI-2427, QI-2532 (read by OCR; spot check). The Jan 2026 drop is unexplained, possibly the Jul 2025 acuity change in QI-2528 |
| `R_NF_HIST(t)` | 246.97 (Jan 2016); 282.98 (Jul 2020); 334.75 (Jul 2021); 371.99 (Jul 2023) | USD per day, nominal | Sourced: rate memos (draft). Figure 1 only |
| `R_CAP(t)` | 22.50 (Jan 2024, the 2023 rate base; no index step, since the SPA's prices relate to 2023 and are updated only for later periods); 23.175 (Jan 2025, x 1.030); 23.9398 (Jan 2026, x 1.033); after 2026 see `R_CAP_f` | USD per day, nominal; capital component, before Hawaii general excise tax and the sustainability fee, which are added on top | Sourced: SPA HI-23-0014, p. 38a: 120% of the day-weighted median capital cost per day, prices relating to 1/1/2023 to 12/31/2023 (draft; verify). The rate memos print total per diems only, so later values are derived with the index. The $22.50 is already priced for 2023, so `R_CAP_2023` takes it directly (Price year) |
| `R_WAIT(t)` | 464.97 (Jan 2024); 486.76 (Jan 2026) | USD per waitlisted hospital day, nominal | Sourced: rate memos (draft). History for Figure 1: 234.09 (Jan 2016), 302.89 (Jul 2023) |
| `G_IDX(t)` | 0.030 (FY2024); 0.030 (FY2025); 0.033 (FY2026); 0.033 (FY2027) | per year, by federal fiscal year (October to September) | Sourced proxy (Micah's decision): the CMS SNF market basket, gross, from the SNF PPS final rules (draft; verify). The net Medicare updates (6.4%, 4.2%, 3.2%, 2.4%) are not used: they add Medicare-only forecast-error and productivity adjustments. Stands in for the S&P Global Nursing Home without Capital Market Basket the SPA names, which is proprietary and not printed in the rate memos. The CMS basket includes capital; the SPA's index excludes it. Timing (Micah's decision): rate year `t` takes fiscal year `t`'s basket (the fiscal year that begins the October before), applied once a year at the January rate period, with no step at July. After FY2027 the base holds 0.033 (Micah's decision), with a run at 0.030 |
| `GET_RATE` | 0.04712 | fraction of the price, added on top | Sourced: Hawaii Department of Taxation, maximum pass-on rate of the general excise tax with the county surcharge, 4.712% in all four counties from Jan 1, 2024 (Maui's surcharge began then): https://tax.hawaii.gov/geninfo/countysurcharge/ (draft; verify). Puts the capital component on the same basis as `R_NF`, which includes the tax (Micah's decision) |
| `MCD_NF_DAYS` | 526,866 | Medicaid nursing-facility days per year, FY2023 | Sourced: FY2023 cost reports, freestanding SNFs, of 1,088,608 total days (draft) |

### Capital (by rung `k`)
| Name | Value | Unit | Source |
|---|---|---|---|
| `K(k)` | rung 1: 10,255; rung 2: 244,869 | USD per bed, 2023 $ | Sourced and derived (decisions 44 and 74; draft, read by OCR; verify): the median of project cost / beds added across the Certificate of Need applications of each type, each deflated to 2023 from its filing year with `CPI_HI`. Rung 1, beds in an existing building by conversion or lease (4 applications): 13-18A 5,666 (67 added beds); 18-08 7,142; 18-07A 13,368; 09-03 159,232. Leased projects count capital items only (the rent stays out); the owned conversion counts its purchase of land with existing buildings, since land is included. Rung 2, new building, land included (8 applications): 24-10A 99,235; 15-08 221,250; 11-01 227,766; 10-12 243,055; 14-07 246,683; 20-21 251,295; 08-08 266,658; 22-21A 535,329. 21-10A (acute beds relicensed, no space work, 2,353) is reported but not in the median. Both types have at least 3 applications, so neither is labeled assumed |
| `N_LIFE(k)` | rung 1: 20; rung 2: 40 | years | Assumed (decision 45): typical lives for a renovation and a new building. The AHA guide Medicare uses for asset lives is paywalled; check against it if available |
| `DISC_RATE` | 0.02; runs at 0.03 and 0.07 | per year, real | Derived (decision 46): 30-year A-rated tax-exempt yields of 4.0 to 4.5% less 2.28% expected inflation give 1.7 to 2.2% real. The runs are the federal cost-benefit rates (OMB Circular A-4, 2003, reinstated in 2025). This is the real form of the brief's "long-term borrowing rate": the model is in constant 2023 dollars (decision 33), so the borrowing rate is taken net of expected inflation (decision 76) |
| `RUNG_BEDS(k)` | rung 1: unlimited in the base, 0 in the new-building run; rung 2: unlimited | beds available in the rung | Decisions 62 and 77 |
| `HORIZON` | 20; 40 in the new-building run | years over which the rate rule is priced and the operating test is run | Decision 62: the life of the run's capital, the brief's "same horizon as the capital" |

### Staffing (critique Part 2)
| Name | Value | Unit | Source |
|---|---|---|---|
| `WAGE_PREMIUM` | `SALARY_SHARE x NF_COST_2023 x WAGE_INCREASE_PCT x (1 + SPREAD_FACTOR)` | USD per occupied day, 2023 $ | Derived (the critique's pattern, corrected: the critique multiplies by `SPREAD_FACTOR` alone, which gives no premium at all when only the new beds' staff get the raise). It is an operating cost, so it enters both `C_FULL` and `C_OP`. 0 in the base; 12.91 in the staffing run (decision 48) |
| `WAGE_INCREASE_PCT` | 0 in the base; 0.065 in the staffing run | fraction | Reference: Hawaii nursing-home pay grew 6.5% in 2024 and 4.0% in 2025 (BLS QCEW; draft) |
| `SPREAD_FACTOR` | 0 | existing staff's pay raised to match, per dollar of the new beds' pay: 0 = only the new beds' staff get the raise | Assumed (decision 48) |

### Budget view
| Name | Value | Unit | Source |
|---|---|---|---|
| `MCD_SHARE_FREED` | 1 | fraction of freed days Medicaid would have paid at `R_WAIT` | Assumed (decision 41): the beds take Medicaid patients, as the brief assumes |
| `MCD_SHARE_BED` | 1 in the base; Medicare run: `MAX(0, STAY_DAYS - MEDICARE_FULL_DAYS) / STAY_DAYS` = 0.3333 at 30-day stays | fraction of transitional-bed days Medicaid pays at `R_NF` | Assumed (decision 41) |
| `MEDICARE_FULL_DAYS` | 20 | days of a SNF stay Medicare Part A pays in full after a qualifying 3-day hospital stay | Sourced: Medicare.gov (draft). Medicare run only. Days 21 to 100 are Medicare days too, with a $217 daily coinsurance that Medicaid pays for dual-eligible patients; the run counts them as Medicaid days, so it understates Medicare's share |
| `FMAP` | 0.5856 (FY2024); 0.5908 (FY2025); 0.5968 (FY2026) | fraction, by federal fiscal year | Sourced: MACPAC Exhibit 6 (draft; verify). The FY2024 value is the one in effect January 1 to September 30, 2024 (the first quarter carried a 1.5-point temporary increase). The Jan 2026 rates the base uses fall in FY2026 |

### Rate comparison
| Name | Value | Unit | Source |
|---|---|---|---|
| `DELTA_R` | 10 | USD per day | Assumed unit (brief) |
| `SUPPLY_RESPONSE` | arc, -1.1997, in the base; simple, -0.9056, as a run | % change in waitlist days per % change in rate | Derived from `WAIT_DAYS(2023)`, `WAIT_DAYS(2024)` and the median rate, $371.99 (Jul 2023) to $481.83 (Jan 2024). Arc: `((W24 - W23) / ((W24 + W23) / 2)) / ((481.83 - 371.99) / ((481.83 + 371.99) / 2))`; simple: `((W24 - W23) / W23) / ((481.83 - 371.99) / 371.99)`. A first estimate, confounded by everything else that changed in 2024 (decisions 57 and 62) |
| `REBASE_YEARS` | 3 | years between rebases | Assumed (decision 50) |
| `RULE_START` | 2027 | first rate year under the rule | Assumed: the first rate period after this paper |
| `INFL` | 0.0228 | per year | Sourced: FRED 10-year breakeven inflation, Sep 25, 2026 (draft). Projects the CPI past the latest full year |

### Onward placement (enters through the `STAY_DAYS` runs and the stay-length flag)
| Name | Value | Unit | Source |
|---|---|---|---|
| `T19_WAITING_2024` | 39 on Dec 31, 2024: 34 waiting for a care home; reasons 24 financial or Medicaid, 10 behavior, 5 family, 0 no bed | patients in long-term-care beds ready to move down a level | Sourced: Table 19, 2024 (draft) |
| `T19_DAYS_2024` | 5,578 (understated: three facilities blank) | days | Sourced: Table 19, 2024 (draft) |
| `MEDIAN_MCD_LOS` | 543 | days, median Medicaid stay | Sourced: cost reports (noisy, 8 to 1,680) |

### Price year (decisions 33 and 47)
| Name | Value | Unit | Source |
|---|---|---|---|
| `CPI_HI(t)` | 325.954 (2023); 340.197 (2024); 348.922 (2025) | index, 1982-84 = 100, annual average | Sourced: BLS CPI-U, Urban Hawaii, CUUSA426SA0 (draft; from a search summary of FRED); Pull at build to confirm. Later years grow at `INFL` |
| `DEFLATOR(t)` | `CPI_HI(2023) / CPI_HI(t)` = 1.00000 (2023); 0.95813 (2024); 0.93417 (2025) | multiplier to 2023 dollars | Derived. A value dated January of year t uses year t - 1 |
| `R_NF_2023` | `R_NF(2026) x DEFLATOR(2025)` = 462.65 | USD per day, 2023 $ | Derived (draft) |
| `R_WAIT_2023` | `R_WAIT(2026) x DEFLATOR(2025)` = 454.72 | USD per waitlisted day, 2023 $ | Derived (draft) |
| `R_CAP_2023` | `R_CAP` for the 2023 rate base = 22.50 | USD per day, 2023 $ | Sourced: SPA HI-23-0014, p. 38a. The price relates to 1/1/2023 to 12/31/2023, so it is already in 2023 dollars and takes no deflator. Moving it forward by `G_IDX` to 2026 and deflating back by the CPI would mix the two indexes and misstate it |

### Hypothesis values (read only by Checks)
| Name | Value | Unit | Source |
|---|---|---|---|
| `HYP_BEDS` | 150 | beds | Brief |
| `BAND_LO`, `BAND_HI` | 125, 175 | beds | Brief (decision 9). 125 is derived; 175 is symmetric by choice |
| `HYP_BREAKEVEN` | 900 | USD per freed day, 2023 $ | Brief (decision 25) |
| `RANGE_LO`, `RANGE_HI` | 550, 1,470 | USD per freed day, 2023 $ | Brief (decision 24) |
| `HYP_STAY_FAIL` | 50 | days | Brief ("if average stays run past about 50 days") |

### Decision variables
| Name | Unit | Source |
|---|---|---|
| `BEDS` | beds (integer) | Bed schedule: the count that maximizes cumulative net saving (Calculation logic) |

### Scenario inputs
Every output is reported for the base run and for each other run, changing one input at a time: 15 runs besides the base.

| Name | Base | Other runs | Unit | Source |
|---|---|---|---|---|
| `C_AVOID` | 1,010 | 550; 1,470 | USD per freed day, 2023 $ | Decisions 24 and 38 |
| Capital | every bed in converted space (rung 1) | new-building run: every bed in a new building (rung 2) | rung | Decisions 62 and 77 |
| `C_AVOID_H(i)` | `C_AVOID` for every hospital | stress run: one hospital at 550, the rest at the base | USD per freed day, 2023 $ | Decision 40 |
| Payer | Medicaid pays every bed day and would have paid every freed day | Medicare run: Part A pays days 1 to 20 of each stay | | Decision 41 |
| `WAGE_INCREASE_PCT` | 0 | staffing run: 0.065, new beds' staff only (`SPREAD_FACTOR` 0) | fraction | Decision 48 |
| `SUPPLY_RESPONSE` | -1.1997 (arc) | -0.9056 (simple) | elasticity | Decision 57 |
| `DISC_RATE` | 0.02 | 0.03; 0.07 | per year, real | Decision 46 |
| `G_IDX` after FY2027 | 0.033 | 0.030 | per year | Decision 68 |
| `STAY_DAYS` | 30 | 60; 90 | days | Decision 20 |
| `Y` | 2023 | 2024 | waitlist year | Decision 18 |
| `PAST_POOL_PCT` | 0 | 0.25; 0.5 | fraction | Decision 37 |

### Values the build pulls
The session that builds the model fills each value below, then records it in Audit findings: the file or URL, the rule applied, the value found, and any gap from the draft figure. Draft figures stay in the tables until then.

1. **`NF_CAP_COST_2023`.** Filled: the $18.75 proxy (Inputs), since the CMS cost-report file has no capital-cost lines. Still re-derive the $462 median from the same 18 homes and record it.
2. **`R_CAP(t)`.** Filled: $22.50 for the 2023 rate base (SPA HI-23-0014, p. 38a; Inputs). The rate memos print total per diems only, so later periods are the $22.50 moved by `G_IDX`, labeled derived.
3. **`G_IDX(t)`.** Filled: the CMS SNF market basket, FY2024 to FY2027 (Inputs). The rate memos print total per diems only, not the index. Timing and the value after FY2027 are set in the `G_IDX` row.
4. **`R_NF(t)` and `R_WAIT(t)`.** Re-read the Jan 2024, Jan 2025 and Jan 2026 medians from the memos, since the notes' values were read by OCR. Read QI-2528 for the cause of the Jan 2026 drop.
5. **`WAIT_DAYS_H(i, Y)` and `STATE_RUN_H(i)`.** Filled: Table 18 hospital rows for 2023 and 2024 (patients, days, Dec 31 count) and the state-run flags (Inputs, Hospital rows). Table 18 also reports the Dec 31 reasons by hospital; those are not entered. Verify Kauai Veterans' flag.
6. **`K(1)` and `K(2)`.** Filled: 10,255 and 244,869 (Inputs), from 13 Certificate of Need applications read, 12 in the medians (`shpda-con-nf-bed-project-costs.csv`). Spot-check the OCR figures against the application PDFs.
7. **Cut** with the empty-bed run (decision 77). The empty-bed counts (817.6 statewide, 193.3 in state-run homes, 2026 Q1) stay in `hi-nursing-home-staffing-2026q1.csv` as context for the analysis.
8. **`FMAP`.** Filled: FY2024 to FY2026 (MACPAC; Inputs).
9. **`DISC_RATE` check.** Record the 30-year A-rated tax-exempt yield and the 10-year breakeven inflation rate on the build date, and report the implied real rate against the 2% base (decision 46).
10. **`CPI_HI(t)`.** BLS CPI-U, all items, Urban Hawaii (CUUSA426SA0), annual averages from 2011 to the latest full year (decision 47).
11. **Table 18, other years.** 2022 is online; 2015 to 2021 if published. Patients, days and Dec 31 reasons, for the reasons test outside the model and for Figure 3.
12. **Table 19, 2022.** Extends the onward-placement evidence.

## Structure
One workbook, `model.xlsx` (decision 32). Sheets, in this order:

- **Inputs:** every named input above in its own labeled cell, entered once, with its Source text beside it. Everything downstream references these by name; no input value is retyped.
- **Waitlist:** Table 18 by year (days, patients, Dec 31 reasons), `DAYS_AVOIDED`, `POOL_SHARE`, `POOL_DAYS`, `POOL_BEDS`; hospital rows (`WAIT_DAYS_H`, `STATE_RUN_H`).
- **Ladder:** one row per rung (rung 1, converted space; rung 2, a new building): beds available, `K`, `N_LIFE`, `K_ANN`, `COST_YR`, `BREAKEVEN`, `GRANT`.
- **BedSchedule:** one row per bed `b = 1 .. B_MAX`: its rung, `IN_POOL`, `FREED_B`, `MS` at each `C_AVOID` run, `MC`, `NET`; then `BEDS`, `BEDS_FIRST_FAIL` and `STOP_REASON`.
- **Operating:** the `C_OP` build-up (total cost, capital, wage premium); one row per rate year with cost, `RATE_AT_COST`, and for the status quo and both rule forms the rate, the capital component and `OP_MARGIN`; `SHORTFALL`; `OP_FAIL_YEAR` for each.
- **RateCompare:** the one-time increase (`INC_FREED`, `UNIT_INC`); both rule forms year by year (`RULE_COST`, `RULE_KEPT`) with present values over `HORIZON`; the partnership (`PART_COST`, `UNIT_PART`); each per waitlist day freed and per bed-equivalent.
- **Budget:** each party's cash per bed-year.
- **Consortium:** one row per hospital: `PLAN_SHARE`, `PAY_H`, `SAVE_H`, `JOIN`, `SOLO`.
- **Scenarios:** one column block per run, each recomputing the outputs from its own copy of the inputs that differ.
- **Checks:** the validation rules below as live PASS / FAIL cells.
- **WorkedExample:** the anchor table below, recomputed by the live formulas.
- **Summary:** every named output for the base run in one place.
- **FigureData:** the series behind each figure.

## Calculation logic
Named-range notation, never cell addresses.

### Freed days and the pool
    OCC_DAYS        = OCC x 365
    ADMITS          = OCC_DAYS / STAY_DAYS
    FREED           = ADMITS x DAYS_AVOIDED(Y)                 [freed days per bed-year, inside the pool]
    POOL_DAYS(Y)    = POOL_SHARE(Y) x WAIT_DAYS(Y)
    POOL_BEDS(Y)    = POOL_DAYS(Y) / FREED

`POOL_BEDS(Y)` also equals `POOL_SHARE(Y) x WAIT_PATIENTS(Y) x STAY_DAYS / OCC_DAYS`: days per patient cancel out of the count and matter only for the saving. That is why 2023 and 2024 both give about 150 beds.

### Capital
    ANNUITY(r, n)   = r / (1 - (1 + r) ^ -n)
    K_ANN(k)        = K(k) x ANNUITY(DISC_RATE, N_LIFE(k))     [present value spread over the life of the space]

### Resource view: cost, saving and break-even (sets the count)
    COST_YR(k)      = K_ANN(k) + OCC_DAYS x C_FULL             [C_FULL = 462 in the base; decisions 34 and 48]
    SAVE_YR         = FREED x (C_AVOID + ACUTE_MARGIN)
    BREAKEVEN(k)    = COST_YR(k) / FREED

`C_FULL` is the FY2023 median total cost per day, which includes the existing homes' capital (decision 34, the brief's basis). A new bed's capital therefore enters twice: inside `C_FULL` and again through `K_ANN(k)`. Medicaid payments never enter this block (decision 21).

### Marginal rule and the bed count
For each bed `b = 1 .. B_MAX`:

    IN_POOL(b)      = MIN(1, MAX(0, POOL_BEDS(Y) - (b - 1)))   [share of bed b inside the pool]
    FREED_B(b)      = FREED x (IN_POOL(b) + (1 - IN_POOL(b)) x PAST_POOL_PCT)
    RUNG(b)         = the rung with the lowest COST_YR(k) that still has beds left at b
    MS(b)           = FREED_B(b) x (C_AVOID + ACUTE_MARGIN)    [marginal saving]
    MC(b)           = COST_YR(RUNG(b))                         [marginal cost]
    NET(b)          = SUM over b' = 1 .. b of (MS(b') - MC(b'))
    BEDS            = the b in 0 .. B_MAX that maximizes NET(b); ties go to the larger b
    BEDS_FIRST_FAIL = smallest b with MS(b) < MC(b)            (report "none" if there is none)
    STOP_REASON     = "no bed pays" if BEDS = 0; otherwise "pool", "rung" or "pool and rung",
                      by what changes between bed BEDS and bed BEDS + 1

In the base every bed is converted space, so `MC(b)` is flat and the count stops at the pool or where no bed pays. The new-building run prices every bed at rung 2.


Rungs are ordered by `COST_YR(k)`, not by `K(k)`, because a cheaper space with a shorter life can cost more per year. `MS(b)` never rises and `MC(b)` never falls with `b`, so `BEDS` is the last bed with `MS(b) >= MC(b)` and equals `BEDS_FIRST_FAIL - 1`. A bed whose saving exactly equals its cost is added (the rule is "add while saving is at least cost"). Both are reported; a difference is a formula error.

### Grant and the operating test
    GRANT(k)        = MAX(0, K_ANN(k) - R_CAP_2023 x OCC_DAYS)   [decision 12, option B]

`R_CAP_2023` is the capital component price for the 2023 rate base, $22.50, already in 2023 dollars (Inputs, Price year). The grant uses it before the general excise tax, because the tax passes through to the state and the home keeps the $22.50; the operating test below uses it with the tax, to match the rate.

The operating test and both forms of the rate rule run in nominal dollars, one row per rate year `t` from 2024 to `RULE_START + HORIZON - 1`. The test counts only the years from `RULE_START`, since the rule cannot act before it starts; the 2024 to 2026 rows are kept for the drift and Figure 1 (decision 90). Decision 11 leaves the form to the analysis, so both are priced (decision 50):

    C_OP_NOM(t)     = C_OP x (1 + G_HI) ^ (t - 2023.5)            [FY2023 cost dated mid-2023]
    RATE_IDX(t)     = R_NF(t) for 2024 to 2026; after that
                      RATE_IDX(t - 1) x (1 + G_IDX(t))             [status quo: the national index only]
    RATE_HIX(t)     = RATE_IDX(t) before RULE_START; after that
                      RATE_HIX(t - 1) x (1 + G_HI)                 [rule form 1: a Hawaii index]
    RATE_REB(t)     = RATE_IDX(t) before RULE_START; in a rebase year (RULE_START, then every
                      REBASE_YEARS) R_NF(2024) x (1 + G_HI) ^ (t - 2024); otherwise
                      RATE_REB(t - 1) x (1 + G_IDX(t))             [rule form 2: a periodic rebase]
    CAP_SHARE       = R_CAP(2024) x (1 + GET_RATE) / R_NF(2024)    [the capital component's share of the rate,
                                                                     both with the general excise tax]
    R_CAP_f(t)      = R_CAP(t) x (1 + GET_RATE) for 2024 to 2026; after that RATE_f(t) x CAP_SHARE
    OP_MARGIN(f, t) = RATE_f(t) - R_CAP_f(t) - C_OP_NOM(t)          [f = IDX, HIX, REB]
    OP_FAIL_YEAR(f) = the first t >= RULE_START with OP_MARGIN(f, t) < 0   ("none" if it never fails)

A rebase puts the rate back where the Jan 2024 reset put it relative to cost: the Jan 2024 rate grown at Hawaii cost growth. The Hawaii index only stops further drift from the Jan 2026 level, so a gap already open in 2026 stays open under it; the rebase closes the gap every `REBASE_YEARS`. All components move with the same index, so the capital component keeps its 2024 share of the rate after 2026.

### Drift
    RATE_AT_COST(t) = R_NF(2024) x (1 + G_HI) ^ (t - 2024)          [the rate that keeps pace with cost]
    SHORTFALL(t)    = 1 - RATE_IDX(t) / RATE_AT_COST(t)             [share by which the index-only rate falls
                                                                     short, from the Jan 2024 reset at cost]

With a constant index this is `1 - ((1 + G_IDX) / (1 + G_HI)) ^ (t - 2024)`, the draft's formula; the ratio form also carries the actual 2025 and 2026 rates, including the Jan 2026 drop.

### Budget view (sets the split)
Per bed-year, in 2023 dollars:

    MCD_NF_PAY      = OCC_DAYS x MCD_SHARE_BED x R_NF_2023           [Medicaid pays the NF rate]
    MCD_WAIT_SAVE   = FREED x MCD_SHARE_FREED x R_WAIT_2023          [Medicaid stops paying the waitlisted rate]
    MCD_NET         = MCD_NF_PAY - MCD_WAIT_SAVE                     [all Medicaid, before the match]
    MCD_NET_B(b)    = OCC_DAYS x MCD_SHARE_BED x R_NF_2023
                      - FREED_B(b) x MCD_SHARE_FREED x R_WAIT_2023   [the same, for bed b, past the pool too]
    STATE_NET       = (1 - FMAP) x MCD_NET
    FED_NET         = FMAP x MCD_NET
    HOSP_NET_DAY(i) = C_AVOID_H(i) - MCD_SHARE_FREED x R_WAIT_2023   [a hospital's net saving per freed day]

`R_NF_2023` and `R_WAIT_2023` are the Jan 2026 rates deflated to 2023 dollars (Inputs, Price year). The draft's state figure was `OCC_DAYS x R - MCD_SHARE_FREED x FREED x R_WAIT`, which assumes Medicaid pays every transitional-bed day and leaves out the match. `MCD_SHARE_BED` and `FMAP` make both explicit. Both Medicaid shares are 1 in the base (decision 41); the Medicare run sets `MCD_SHARE_BED = MAX(0, STAY_DAYS - MEDICARE_FULL_DAYS) / STAY_DAYS`. The Medicare days in that run are paid by Medicare and drop out of the state and federal Medicaid figures.

### Consortium shares (decision 28)
Over the beds counted (`b = 1 .. BEDS`):

    TOTAL_GRANT     = SUM over b of GRANT(RUNG(b))
    TOTAL_FREED     = SUM over b of FREED_B(b)
    PLAN_SHARE(i)   = WAIT_DAYS_H(i, Y) / SUM over i of WAIT_DAYS_H(i, Y)
    PAY_H(i)        = PLAN_SHARE(i) x TOTAL_GRANT
    SAVE_H(i)       = PLAN_SHARE(i) x TOTAL_FREED x HOSP_NET_DAY(i)
    JOIN_MIN        = TOTAL_GRANT / TOTAL_FREED + MCD_SHARE_FREED x R_WAIT_2023
    JOIN(i)         = C_AVOID_H(i) >= JOIN_MIN
    SOLO(i, k)      = PLAN_SHARE(i) x FREED x HOSP_NET_DAY(i) >= GRANT(k)   [rungs with GRANT(k) > 0;
                                                                         "n/a" where GRANT(k) = 0]

`PLAN_SHARE(i)` cancels out of the join test, so `JOIN_MIN` is the same for every hospital and only `C_AVOID_H(i)` differs. The system can pass the break-even test while a low-cost hospital fails this one. With one rung and every bed inside the pool, `JOIN_MIN` reduces to the draft's `G / D + m x R_w`.

`SOLO(i, k)` is the single-hospital test in the budget view (decision 53). A hospital that funds one bed alone pays that bed's whole grant, but the bed takes patients from every hospital, so the funder gets only its planning share of the bed's freed days, net of the waitlisted payments it no longer receives. Where a rung's grant is zero, nothing needs funding and the test does not apply. The brief's wording, "the full cost of the beds those days would fill," is not used: it reduces to the system break-even test (conflict 2 in `decision tree.md`).

### Statewide rate increase, the rate rule and the common unit (decisions 50, 51 and 62)
Costs are per year in 2023 dollars and count all cash spent: Medicaid at all funds (before the federal match), plus the consortium's grant for the partnership. Every option is counted net of the waitlisted-day payments Medicaid stops making on the days it frees, so the three are on the same basis (Micah's decision, 2026-09-28). Each option is reported per waitlist day freed and per bed-equivalent, which is one bed's `FREED` days a year (decision 51).

    STATEWIDE_COST(DELTA_R) = DELTA_R x MCD_NF_DAYS                  [per year, before the match]
    PCT_DR          = DELTA_R / R_NF_2023                             [the increase as a share of the rate]
    INC_FREED       = WAIT_DAYS(Y) x ABS(SUPPLY_RESPONSE) x PCT_DR     [waitlist days a year it frees]
    UNIT_INC        = (STATEWIDE_COST(DELTA_R) - INC_FREED x MCD_SHARE_FREED x R_WAIT_2023) / INC_FREED
                                                                      [one-time increase, per day freed, net]

    RULE_COST(f, t) = (RATE_f(t) - RATE_IDX(t)) x MCD_NF_DAYS x DEFLATOR(t)
                      - RULE_KEPT(f, t) x MCD_SHARE_FREED x R_WAIT_2023          [f = HIX, REB; net]
    RULE_KEPT(f, t) = WAIT_DAYS(Y) x ABS(SUPPLY_RESPONSE) x (RATE_f(t) - RATE_IDX(t)) / RATE_IDX(t)
                                                                      [waitlist days the rule keeps away]
    RULE_COST_PV(f) = SUM over t = RULE_START .. RULE_START + HORIZON - 1
                      of RULE_COST(f, t) / (1 + DISC_RATE) ^ (t - RULE_START + 1)
    RULE_KEPT_PV(f) = RULE_KEPT(f, t) discounted the same way
    UNIT_RULE(f)    = RULE_COST_PV(f) / RULE_KEPT_PV(f)

    PART_COST       = SUM over b = 1 .. BEDS of MCD_NET_B(b) + TOTAL_GRANT   [partnership, per year]
    UNIT_PART       = PART_COST / TOTAL_FREED

    PER_BED_EQ(x)   = UNIT_x x FREED                                  [x = INC, RULE, PART]

The rule is priced over `HORIZON`, the life of the run's capital: 20 years in the base, 40 in the new-building run (the brief's "same horizon as the capital"). Future nominal values are deflated with the CPI projected at `INFL` past the latest full year.

### Staffing (critique Part 2)
    MAX_PREMIUM(k)  = (SAVE_YR - COST_YR(k)) / OCC_DAYS    [largest wage premium per occupied day the saving
                                                            can fund, computed with WAGE_PREMIUM = 0; works
                                                            whatever the wage turns out to be]

### Stay length
    FREED_AT(L)     = OCC_DAYS / L x DAYS_AVOIDED(Y)
    STAY_FAIL(k)    = RANGE_HI x OCC_DAYS x DAYS_AVOIDED(Y) / COST_YR(k)   [the stay length at which
                                                                            BREAKEVEN(k) reaches RANGE_HI;
                                                                            computed on Checks only]
    LONG_STAY_FAIL(k) = (STAY_FAIL(k) - STAY_DAYS) / (MEDIAN_MCD_LOS - STAY_DAYS)
                                                  [share of admissions staying long term at which the
                                                   average stay reaches STAY_FAIL(k); Checks only]
    NF_LEVEL_SHARE(Y) = SNAP_NF_LEVEL(Y) / SNAP_TOTAL(Y)   [share of the Dec 31 waitlist needing
                                                           nursing-home-level care]

If a share `q` of admissions stays the median Medicaid stay and the rest leave after `STAY_DAYS`, the average stay is `q x MEDIAN_MCD_LOS + (1 - q) x STAY_DAYS`, which reaches `STAY_FAIL(k)` at `q = LONG_STAY_FAIL(k)` (decision 58). `NF_LEVEL_SHARE` is an upper bound on `q`, since nursing-home-level care includes short-term needs. Table 19's care-home backlog (`T19_WAITING_2024`) is reported beside it: it measures the next step down, from a nursing home to a care home.

## Conventions

### Views, dollars and time
- **Price year: constant 2023 dollars** (decision 33). The waitlist baseline, `NF_COST_2023`, `HOSP_AVG_COST` and the avoidable-cost range are 2023 values already. Later nominal values (the 2024 to 2026 rates, capital costs from later applications) are deflated to 2023 with the CPI for Urban Hawaii (decision 47): `X_2023 = X_t x DEFLATOR(t)`. A value dated January of year t uses year t - 1's annual average, so the Jan 2024 rates, reset from 2023 costs, stay at their 2023 level. Discount rates are therefore real: 2% in the base, with runs at 3% and 7% (decision 46).
- **The operating test and the drift run in nominal dollars**, year by year, with cost inflated at `G_HI` from mid-2023 (the FY2023 cost-report year) to each rate period, so rates are never compared with costs from another year (critique Part 2, point 5).
- **The rate rule starts with the Jan 2027 rate period** (`RULE_START`) and both forms are priced (decision 50). Future nominal values are deflated with the CPI grown at `INFL`.
- **The resource view sets the count and the budget view sets the split** (decision 21). Nothing in the budget view feeds `BEDS`.
- **Running cost includes the existing homes' capital** (decision 34). `C_FULL` is the $462 total cost per day, as in the brief. The operating test uses `C_OP`, which excludes capital, as the brief states the test.
- **The brief's predictions never feed a calculation.** `HYP_BEDS`, `HYP_BREAKEVEN`, `BAND_LO`, `BAND_HI` and `HYP_STAY_FAIL` are read only by Checks. `RANGE_LO` and `RANGE_HI` are also the low and high `C_AVOID` runs, which is intended: the range was set in advance (decision 24).

### Costing order
- Beds are added cheapest rung first, by cost per bed-year (`COST_YR(k)`); within a rung, beds inside the pool come before beds past it.
- A bed-year costs its annualized capital plus the running cost (`C_FULL`) of its occupied days. Consortium administration, contract enforcement and permit work are outside the model.
- The new beds get the same rate as every other home, and the capital component counts toward their capital (decision 12, option B).

### Precision and boundaries
- **No intermediate rounding.** `DAYS_AVOIDED(2023)` is the exact quotient 83,097 / 3,610 = 23.0186, not the brief's 23, so `FREED` is 252.05, not 251.85. Rounding is display only: whole dollars; two decimals for rates, shares and days per patient.
- **The `C_AVOID` runs are the brief's $550 and $1,470**, not the unrounded 15% and 40% of $3,664 ($549.60 and $1,465.60), so the runs and the check thresholds agree.
- **`GRANT(k)` is floored at zero.** Where the capital component pays more than the capital, the consortium pays nothing and nothing is paid back.
- **Beds are integers; `POOL_BEDS` is not.** The bed that straddles the pool's edge gets the in-pool share of its freed days (`IN_POOL`). In 2023 that is bed 151, with 0.1685 of a pool bed's 252.05 days (42.48).
- **Past the pool, a bed frees nothing in the base run** (`PAST_POOL_PCT` = 0, decision 37), so the base `BEDS` cannot pass `POOL_BEDS`. The runs at 0.25 and 0.5 show how far the count moves if some of the other waitlisted patients can be placed.
- **`B_MAX` is 400**, above the roughly 330 beds that would absorb every 2023 waitlist day.
- **Rung sizes are whole beds, rounded down.**

### Scope
- One statewide market (brief assumption; decision 14).
- Onward placement enters through the `STAY_DAYS` runs and the stay-length flag (decision 58); care-home and home-care capacity are not modeled.
- The waitlist is held at the year's level; demand growth from an aging population is not modeled.
- The acute-bed margin is zero in every run (decision 56), so the savings, and with them the count, are floors.
- Medicaid pays for the transitional stays in the base run, as the brief assumes; the Medicare run moves the first 20 days of each stay to Medicare Part A (decision 41). The operating test stays on the Medicaid rate in every run, because Medicaid is the payer whose rate falls short: Medicare fee-for-service SNF margins run about 22% (MedPAC, March 2026; research notes, 3.1). The payer never changes the bed count, since payments are transfers in the resource view.

## Validation rules
Written as acceptance criteria for the built model. Build the **Checks** sheet as live PASS / FAIL cells.

Structural:
- Every calculated cell is a formula referencing named ranges: no hard-coded numbers in a computed cell and no error values anywhere. Change a named input and confirm every dependent figure moves.
- `BEDS` is the maximum of `NET(b)`, computed for every bed to `B_MAX`, and equals `BEDS_FIRST_FAIL - 1` (or `B_MAX` when no bed fails).
- `POOL_BEDS(Y)` equals `POOL_SHARE(Y) x WAIT_PATIENTS(Y) x STAY_DAYS / OCC_DAYS`.
- The Dec 31 reasons sum to `SNAP_TOTAL(Y)` in each year.
- `GRANT(k) >= 0`; `SUM of PLAN_SHARE(i)` is 1; `SUM of PAY_H(i)` equals `TOTAL_GRANT`.
- In the base run (`PAST_POOL_PCT` = 0), `BEDS <= POOL_BEDS`.
- No Pull at build value is blank or still at a placeholder.

Hypothesis tests, from the brief. Each has a required value, a tolerance and a PASS rule; the result goes in Audit findings.

| Test | Required | Tolerance | PASS when | Runs |
|---|---|---|---|---|
| Bed count | `HYP_BEDS` = 150 | ±25 beds (decision 9) | `BAND_LO <= BEDS <= BAND_HI` | Base; reported for every run. In the base (`PAST_POOL_PCT` = 0) a bed past the pool frees nothing, so `BEDS` is either 0 or the last whole bed inside the pool: the test checks whether any bed pays. The count moves only in the 0.25 and 0.5 runs |
| First beds pay | Break-even of the first rung inside the range set in advance; distance from the $900 guess reported | None on the range, and none on the guess (decision 55): every rung's break-even is reported against $900 | `RANGE_LO <= BREAKEVEN(first rung used) <= RANGE_HI` | Base: converted space (rung 1), the brief's first beds. Every rung's break-even is reported against $900 |
| Stay length | Break-even at 60- and 90-day stays; `STAY_FAIL` against the brief's 50 days; `LONG_STAY_FAIL` against `NF_LEVEL_SHARE` | None | FAIL for any stay length where `BREAKEVEN(first rung used) > RANGE_HI`. Flag "at risk" when `NF_LEVEL_SHARE(Y)` exceeds `LONG_STAY_FAIL` of the first rung used (decision 58) | `STAY_DAYS` 30, 60, 90; `Y` 2023 and 2024 |
| Operating test | `OP_MARGIN(f, t) >= 0` in every year from `RULE_START` to the horizon, under a form of the rate rule | $0 | At least one form (f = HIX or REB) holds every year. `OP_FAIL_YEAR` is reported for the status quo and for each form | Base |
| No single hospital funds alone | `SOLO(i, k)` false: one hospital's share of a shared bed's net saving is below that bed's grant | None | False for every hospital at every rung used with a positive grant; a rung with no grant is reported n/a (decision 53) | Base |
| Every hospital joins | `C_AVOID_H(i) >= JOIN_MIN` | None | For every consortium hospital | Base (every hospital at `C_AVOID`) and the stress run (one hospital at 550) |
| Partnership vs. one-time increase | `UNIT_PART` below `UNIT_INC`: the partnership frees a waitlist day for less than a one-time increase | None | `UNIT_PART < UNIT_INC`. `UNIT_RULE` is reported alongside; since the rule costs about what an increase does per day, adding it can't flip the result (decision 54) | Base |

Worked-example anchors. The model must reproduce these from the named inputs:

| Figure | Value | Tolerance |
|---|---|---|
| `OCC_DAYS` | 328.5 | exact |
| `ADMITS` at 30-day stays | 10.95 | exact |
| `DAYS_AVOIDED(2023)` | 23.0186 | ±0.0001 |
| `FREED`, 2023, 30-day stays | 252.05 (brief: about 252) | ±0.01 |
| `FREED` at 60- and 90-day stays, 2023 | 126.03; 84.02 (brief: about 126 and 84) | ±0.01 |
| `POOL_DAYS(2023)` | 37,850.5 (brief: about 37,900) | ±0.1 |
| `POOL_BEDS(2023)` | 150.17 (brief: about 150) | ±0.01 |
| `FREED`, 2024, 30-day stays | 213.45 | ±0.01 |
| `POOL_BEDS(2024)` | 150.02 | ±0.01 |
| `STATEWIDE_COST(10)` | 5,268,660 (brief: about $5.3M a year) | exact |
| `G_HI` | 0.0402 | ±0.0001 |

Worked example: capital, budget and operating side. The model must reproduce these at fixed test values, chosen only as numeric anchors (like the farm spec's (5, 5, 5)); they are not the base run. Test values are entered on the WorkedExample sheet, never on Inputs.

Capital and consortium test values: `K` = 200,000; `DISC_RATE` = 0.05; `N_LIFE` = 30; one bed, inside the pool; `Y` = 2023; `STAY_DAYS` = 30; `C_AVOID` = 1,010. `K` is set high enough that `GRANT` is positive, so a wrong grant formula can't hide behind a zero. The budget rows use the base inputs (`R_NF_2023`, `R_WAIT_2023`, `FMAP` for FY2026), since they have no test value of their own.

| Figure | Value | Tolerance |
|---|---|---|
| `ANNUITY(0.05, 30)` | 0.065051 | ±0.000001 |
| `K_ANN` | 13,010.29 | ±0.01 |
| `COST_YR` | 164,777.29 | ±0.01 |
| `SAVE_YR` | 254,573.76 | ±0.01 |
| `BREAKEVEN` | 653.74 | ±0.01 |
| `GRANT` | 5,619.04 | ±0.01 |
| `MCD_NF_PAY` | 151,980.48 | ±0.01 |
| `MCD_WAIT_SAVE` | 114,613.32 | ±0.01 |
| `MCD_NET` | 37,367.16 | ±0.01 |
| `STATE_NET` | 15,066.44 | ±0.01 |
| `FED_NET` | 22,300.72 | ±0.01 |
| `HOSP_NET_DAY` | 555.28 | ±0.01 |
| `JOIN_MIN` | 477.01 | ±0.01 |

Operating test values, rate year 2027 (the first rule year and a rebase year): `C_OP` = 400 (2023 dollars, dated mid-2023); `G_HI` = 0.04; `R_NF(2024)` = 480; `R_NF(2026)` = 500; `G_IDX(2027)` = 0.03; `CAP_SHARE` = 0.05.

| Figure | Value | Tolerance |
|---|---|---|
| `C_OP_NOM(2027)` | 458.8563 | ±0.0001 |
| `RATE_IDX(2027)` | 515.0000 | ±0.0001 |
| `RATE_HIX(2027)` | 520.0000 | ±0.0001 |
| `RATE_REB(2027)` | 539.9347 | ±0.0001 |
| `R_CAP_IDX(2027)`; `R_CAP_HIX(2027)`; `R_CAP_REB(2027)` | 25.7500; 26.0000; 26.9967 | ±0.0001 |
| `OP_MARGIN(IDX, 2027)` | 30.3937 | ±0.0001 |
| `OP_MARGIN(HIX, 2027)` | 35.1437 | ±0.0001 |
| `OP_MARGIN(REB, 2027)` | 54.0817 | ±0.0001 |
| `RATE_AT_COST(2027)` | 539.9347 (equals `RATE_REB(2027)`, a rebase year) | ±0.0001 |
| `SHORTFALL(2027)` | 0.046181 | ±0.000001 |

Hand checks:
- `ANNUITY(0.05, 30)` = 0.065051; `ANNUITY(0.03, 30)` = 0.051019.
- Running cost alone (no capital), with `WAGE_PREMIUM` = 0 and `ACUTE_MARGIN` = 0: 328.5 x 462 / 252.05 = 602.12, the brief's "about $600 per freed day." With 2024 waitlist days: 328.5 x 462 / 213.45 = 711.03.
- Running cost alone sets a floor under every break-even: 602.12 per freed day in the 2023 base, and higher in every other run. The first-beds test's lower bound (`RANGE_LO`, 550) therefore cannot bind.
- `SHORTFALL` after 10 years at a 1.5-point gap (4.0% vs. 2.5%) = 0.1352, the brief's "about 14%."
- `LONG_STAY_FAIL` at the brief's $227,000 a bed-year: `STAY_FAIL` = 1,470 x 328.5 x 23.0186 / 227,000 = 48.97 days, so (48.97 - 30) / (543 - 30) = 0.0370.
- `NF_LEVEL_SHARE`: 157 / 191 = 0.8220 (2023); 120 / 173 = 0.6936 (2024).

Cross-checks:
- The 2023 and 2024 runs give nearly the same `POOL_BEDS` (150.17 and 150.02) because days per patient cancel out of the count. A large gap between them is a formula error.
- `UNIT_RULE` and `UNIT_INC` are both across-the-board rate increases with the same supply response, so each reduces to about `MCD_NF_DAYS x rate / (WAIT_DAYS x ABS(SUPPLY_RESPONSE)) - R_WAIT_2023`. They differ only through the rate level each is priced at; a large gap is a formula error.
- In every rebase year, `RATE_REB(t)` equals `RATE_AT_COST(t)`.

## Outputs
- `BEDS`, `BEDS_FIRST_FAIL` and `STOP_REASON`, for every run.
- `POOL_BEDS(Y)`, `FREED`, and the `FREED_B(b)` schedule.
- For each rung: `K_ANN(k)`, `COST_YR(k)`, `BREAKEVEN(k)`, `GRANT(k)`.
- Operating: `C_OP`, `OP_MARGIN(f, t)` and `OP_FAIL_YEAR(f)` for the status quo and both rule forms, `SHORTFALL(t)`.
- Staffing: `WAGE_PREMIUM`, `MAX_PREMIUM(k)`.
- Budget view per bed-year: `MCD_NET`, `STATE_NET`, `FED_NET`, and each hospital's net saving per freed day.
- Consortium: `PLAN_SHARE(i)`, `PAY_H(i)`, `SAVE_H(i)`, `JOIN_MIN`, `JOIN(i)`, `SOLO(i, k)`, and the state-run hospitals' part of the cost.
- Rate comparison: `STATEWIDE_COST(DELTA_R)`, `INC_FREED`, `RULE_COST_PV(f)`, `UNIT_INC`, `UNIT_RULE(f)`, `UNIT_PART`, each also per bed-equivalent.
- Stay length: `BREAKEVEN` at 30-, 60- and 90-day stays; `STAY_FAIL(k)`; `LONG_STAY_FAIL(k)`, `NF_LEVEL_SHARE(Y)` and the at-risk flag, with Table 19's care-home backlog alongside.
- The Checks summary: each test's value, required value, tolerance and PASS / FAIL.

### Figures
Each figure is drawn from the FigureData sheet and must read on its own: a title, axis labels with units, a legend and a source note.

1. **Nursing-home cost vs. Medicaid rates, 2011 to 2036.** x: year. y: USD per day, nominal. Series: median cost per day of the Medicaid-heavy homes (FY2011 to FY2023, points); median Medicaid nursing-facility rate (2016 to 2026, steps); waitlisted hospital rate (2016 to 2026, steps); projected cost at `G_HI` (dashed, from FY2023); projected rate under the index alone (dashed, from Jan 2026); projected rate under each rule form, the Hawaii index and the 3-year rebase (dashed, from Jan 2027). Marked: the Jan 2024 reset. Note on the figure: the rate medians cover about 35 homes and the cost medians the 18 Medicaid-heavy homes. The evidence it carries: the price ceiling and the drift.
2. **Marginal saving vs. marginal cost by bed.** x: beds added, 0 to `B_MAX`. y: USD per bed-year, 2023 $. Series: marginal saving `MS(b)` at the low, base and high `C_AVOID`; marginal cost `MC(b)` for the base (converted space), with the new-building run as a dashed line. Marked: the 125 to 175 band (shaded), `POOL_BEDS` (vertical line), `BEDS` for the base run. The evidence it carries: where the count stops, and why.
3. **Waitlist reasons, Dec 31, 2023 vs. Dec 31, 2024** (optional). x: reason (no bed available; financial, Medicaid or insurance; psychiatric, dementia or behavior; special care; family or guardianship; other). y: patients waitlisted on Dec 31. Series: 2023 and 2024, paired bars. Annotated: waitlist days for each year (83,097 and 60,876). The evidence it carries: the pool, and the first test of it.
4. **Cost per waitlist day freed, by option** (decision 59). x: option (the partnership; the rule as a Hawaii index; the rule as a 3-year rebase; a one-time $10 increase). y: USD per waitlist day freed, 2023 $, base run. Series: one bar per option (`UNIT_PART`, `UNIT_RULE` for each form, `UNIT_INC`), with a marker on each bar for the simple supply-response run (-0.91). Labeled: each bar's value, with the per-bed-equivalent figure beneath. The evidence it carries: the head-to-head comparison behind the strongest objection.

## Success criteria
A finished paper succeeds if it shows why the problem matters. It reports the acute beds held by waitlisted patients on an average day in 2023 and 2024, each year computed with its own day count, as a share of Hawaii's licensed acute beds from a cited source. It shows from the Table 18 nursing-home-level share of the Dec 31 waitlist that these patients are ready for a lower level of care. It compares the cost of a waitlisted day in an acute bed, from a confirmed source year, with the model's range for its avoidable part, labeled as a range set in advance. It explains why now by reporting that waitlist days fell after the January 2024 reset, noting that the comparison is confounded, and by reporting the model's rate shortfall against Hawaii costs in 2034, ten years after the reset. It names at least five of the following and uses each to explain a specific number in the same paragraph: price ceiling, shutdown test, marginal analysis, externality, prisoner's dilemma, Coase bargaining, elasticity, present value. It reports the model's bed count against the 125 to 175 band, naming the triggered test if the count falls outside, and the break-even avoidable cost against the $900 guess and the range set in advance. It shows results at 30, 60 and 90-day stays. It reports the operating test, the new beds' operating cost excluding capital against the Medicaid rate excluding its capital component, with and without the rate rule, and states whether the partnership stays a one-time investment or becomes a continuing subsidy. It reports the single-hospital test and the join test, as specified in the model. It confronts the post-reset rise in financial/Medicaid waitlist cases as evidence against its own reading. It recommends a specific design: a bed count or first phase, a consortium paying by freed days, state enforcement, faster certificate-of-need approval, and a named rate-rule form. It compares a one-time statewide increase, the rate rule alone and the partnership in cost per waitlist day freed and per bed-equivalent. It answers the strongest objection, that the rate rule is itself a statewide increase, with those numbers, and addresses staffing in at least one sentence. It includes a figure of nursing-home cost per day against the Medicaid and waitlisted rates from 2011 through the 2036 projection, with history and projection visibly distinguished, labeled axes with units, a source note, a caption stating its point and a numbered reference in the text, so the figure reads on its own and the argument would be weaker without it. It stays within four body pages in 12-point Times New Roman, double-spaced with 1-inch margins, uses one citation style with a bibliography, identifies the author only on the title page, contains no repository URL, and cites or derives in the appendix every number in the body. Where space allows, it also:

- cites a clinical source on the harms of prolonged hospital stays, such as deconditioning, delirium or infection risk;
- shows from the Table 18 county rows where the waitlist is concentrated, stating that the model sets a statewide count, not one by island;
- reports results for converted space versus a new building and at two discount rates;
- keeps the full-cost view for the bed count and the budget view for the split, including the state's net Medicaid cost per bed-year;
- reports both the Medicaid base run and the Medicare run, stating which it assumes and why.

## Audit findings
Added after the model is built. For each check: what was checked, what was found, what was done about it. At least: every Pull at build value (the file or URL, the rule applied, the value found, and any gap from the draft figure); the worked-example anchors; the structural checks; and each hypothesis test's result against its tolerance.

Audit of `model.xlsx` as built on 2026-09-28. Values marked draft in this spec are still unverified: decision 89 holds that verification for this audit, and it is listed under "Not yet done" below.

### Method

- **No Excel on the build machine.** Excel automation hangs on this computer, so `model.xlsx` was generated by a Python script (openpyxl) from this spec: 259 named ranges for the inputs and base outputs; helper columns instead of array formulas; no cell addresses in the Inputs contract. The script, the checks below and the reference model are in Claude's job folder, not in the repo.
- **The formulas themselves were evaluated.** A formula engine (the `formulas` Python package, in an isolated environment) computed every cell from the workbook's own formulas. A full evaluation of all 16 runs ran out of memory, so the runs were evaluated in five batches of three plus the base; every batch file's formulas are the committed file's formulas (the rebuilt full workbook is byte-identical to the committed one, sheet by sheet).
- **An independent re-implementation.** A separate Python program written from this spec's Calculation logic, without reference to the workbook, computed the base and all 15 runs. The engine's results were compared with it output by output.
- **Cached values.** The engine's results are written into the file as cached values (103,569 numbers, 269 text results, 1 true/false; no formula cell left without a value). The workbook recalculates fully on open.
- **Still needs Excel:** opening the file without a repair prompt and recalculating to the same values. Only Micah can run that check.

### Findings

- **No error values.** The engine found no error cells in the base evaluation or in any of the five batches (about 36,000 formula cells each). Where a run adds no beds, cells that would divide by zero report "n/a (no beds)" (build choice). PASS.
- **Engine vs. independent re-implementation.** 39 outputs per run (bed count, stop reason, pool, capital, break-evens, grants, budget, consortium, rate comparison, operating fail years, shortfall) for all 16 runs, compared at a relative tolerance of 1e-6: 0 mismatches. A readback of the cached file matched again. PASS.
- **Worked-example anchors.** All 45 anchors on the WorkedExample sheet pass their tolerances: the 12 anchors from the named inputs (for example `FREED` 252.05, `POOL_BEDS(2023)` 150.17, `STATEWIDE_COST(10)` 5,268,660, `G_HI` 0.0402), the 13 capital, budget and consortium anchors at the test values (`K_ANN` 13,010.29, `BREAKEVEN` 653.74, `GRANT` 5,619.04, `MCD_NET` 37,367.16, `JOIN_MIN` 477.01), the 12 operating anchors at the test values (`OP_MARGIN` 30.3937, 35.1437, 54.0817; `SHORTFALL(2027)` 0.046181) and the 8 hand checks (`ANNUITY(0.03, 30)` 0.051019; running cost alone 602.12 and 711.03; the brief's 14% drift at a 1.5-point gap, 0.1352; `LONG_STAY_FAIL` 0.0370; `NF_LEVEL_SHARE` 0.8220 and 0.6936). PASS.
- **Structural checks** (Checks sheet, all PASS): Dec 31 reasons sum to `SNAP_TOTAL` (191; 173); hospital rows sum to the Table 18 totals for days, patients and Dec 31 in both years (largest gap 0); `POOL_BEDS` equals its identity form (150.1685); `BEDS` is the largest bed at the maximum `NET` and equals `BEDS_FIRST_FAIL` - 1 (150 and 151); `GRANT` >= 0 in both rungs; planning shares sum to 1; `PAY_H` sums to `TOTAL_GRANT` (0); in the base `BEDS` <= `POOL_BEDS`; every rebase year has `RATE_REB` = `RATE_AT_COST` (0 mismatches); no Pull at build value is blank; the Scenarios base block reproduces the base sheets' `BEDS`, `BREAKEVEN`, `UNIT_INC`, `UNIT_PART`, both `UNIT_RULE`s, `JOIN_MIN`, `MCD_NET` and the three `OP_FAIL_YEAR`s.
- **"Change a named input and confirm every dependent figure moves."** Covered by the 15 runs, each of which changes one input and was checked against the re-implementation (for example `C_AVOID` 550 takes `BEDS` from 150 to 0; `STAY_DAYS` 60 moves the break-even from 604.61 to 1,209.22). Not a cell-by-cell dependency trace.
- **Cross-checks.** `POOL_BEDS` 2023 and 2024 differ by 0.15 (150.17 and 150.02). PASS. The spec's rough form of the `UNIT_RULE` vs. `UNIT_INC` cross-check has no tolerance; the build first set ±15% and the base failed it (ratios 1.1747 and 1.1569). The gap is not a formula error: the rule is priced at the index rate, which grows 3.3% a year against 2.28% CPI inflation, so its real price rises over the 20 years. The check was replaced by the exact identity (`UNIT_RULE` equals the discounted-days-weighted `MCD_NF_DAYS x rate x deflator / (WAIT_DAYS x |SUPPLY_RESPONSE|)` less `R_WAIT_2023`), which passes to 1e-6 for both forms (build choice); the ratios are reported beside it.

### Hypothesis tests (base run)

| Test | Value | Required | Result |
|---|---|---|---|
| Bed count | 150 (stops at the pool) | 125 to 175 | PASS. In the base this tests whether any bed pays |
| First beds pay | Break-even 604.61 per freed day (converted space); 637.64 in a new building | 550 to 1,470 | PASS. Distance from the $900 guess: -295.39 (converted space), -262.36 (new building) |
| Stay length, 30 days | 604.61 (2023); 713.97 (2024) | <= 1,470 | PASS |
| Stay length, 60 days | 1,209.22 (2023); 1,427.94 (2024) | <= 1,470 | PASS |
| Stay length, 90 days | 1,813.83 (2023); 2,141.91 (2024) | <= 1,470 | FAIL in both years |
| `STAY_FAIL` | 72.94 days (2023); 61.77 days (2024) | brief: about 50 | Reported |
| Stay length at risk | `NF_LEVEL_SHARE` 0.8220 vs `LONG_STAY_FAIL` 0.0837 (2023); 0.6936 vs 0.0619 (2024) | share <= threshold | At risk in both years. Table 19: 34 of 39 waiting for a care home (Dec 31, 2024) |
| Operating test | First year from 2027 with a negative margin: 2027 for the index only and the Hawaii index; 2029 for the rebase | At least one rule form holds every year from `RULE_START` to the horizon | FAIL. See the note below |
| No single hospital funds alone | Converted space has no grant | False, or n/a with no grant | PASS (n/a) |
| Every hospital joins | `JOIN_MIN` 454.72 (the grant is zero, so it equals `R_WAIT_2023`); every hospital at 1,010 | `C_AVOID_H` >= `JOIN_MIN` | PASS; the stress run (one hospital at 550) also passes |
| Partnership vs. one-time increase | `UNIT_PART` 148.25 vs `UNIT_INC` 1,990.28 per waitlist day freed | `UNIT_PART` < `UNIT_INC` | PASS. `UNIT_RULE`: 2,337.98 (Hawaii index), 2,302.47 (rebase) |

**Operating test note.** The test counts rate years from `RULE_START` (2027; decision 90). As first built, it counted from 2024, and every form failed in 2026, before the rule starts; the model was changed and re-evaluated (all five batches: no error cells, 0 mismatches with the re-implementation). The margin (rate less the capital component with GET, less running cost excluding capital) is +6.20 (2024), +15.20 (2025) and -18.93 (2026) in every form, before the rule. From 2027 the index-only form falls from -22.18 to -59.52 by 2034; the Hawaii index holds near -19 to -25, since it keeps the gap already open in 2026; the rebase is positive in rebase years (+6.98 in 2027) but dips to about -0.11 to -0.14 a day in the last year of each cycle (2029, 2032, 2035). So neither form holds every year.

Other base outputs: `SHORTFALL` in 2034, 10.11%; `MCD_NET` 37,367.16 per bed-year (state 15,066.44, federal 22,300.72); `MAX_PREMIUM` 311.05 per occupied day (converted space); state-run hospitals' part of the consortium cost 0 (no grant); state-run share of 2023 waitlist days 28.1%.

### Results by run

| Run | BEDS | Stop | Break-even, first rung | Grant, first rung | UNIT_PART | UNIT_INC | UNIT_RULE HIX | UNIT_RULE REB | OP fail IDX / HIX / REB | Join all | Single hospital |
|---|---|---|---|---|---|---|---|---|---|---|---|
| BASE | 150 | pool | 604.61 | 0.00 | 148.25 | 1,990.28 | 2,337.98 | 2,302.47 | 2027 / 2027 / 2029 | yes | n/a |
| CAVOID_LO | 0 | no bed pays | 604.61 | 0.00 | n/a (no beds) | 1,990.28 | 2,337.98 | 2,302.47 | 2027 / 2027 / 2029 | n/a (no beds) | n/a |
| CAVOID_HI | 150 | pool | 604.61 | 0.00 | 148.25 | 1,990.28 | 2,337.98 | 2,302.47 | 2027 / 2027 / 2029 | yes | n/a |
| NEWBUILD | 150 | pool | 637.64 | 1,560.12 | 154.44 | 1,990.28 | 2,708.10 | 2,653.18 | 2027 / 2027 / 2029 | yes | yes |
| STRESS | 150 | pool | 604.61 | 0.00 | 148.25 | 1,990.28 | 2,337.98 | 2,302.47 | 2027 / 2027 / 2029 | yes | n/a |
| MEDICARE | 150 | pool | 604.61 | 0.00 | -253.73 | 1,990.28 | 2,337.98 | 2,302.47 | 2027 / 2027 / 2029 | yes | n/a |
| STAFFING | 150 | pool | 621.44 | 0.00 | 148.25 | 1,990.28 | 2,337.98 | 2,302.47 | 2027 / 2027 / 2027 | yes | n/a |
| SR_SIMPLE | 150 | pool | 604.61 | 0.00 | 148.25 | 2,784.34 | 3,244.96 | 3,197.92 | 2027 / 2027 / 2029 | yes | n/a |
| DISC_3 | 150 | pool | 604.86 | 0.00 | 148.25 | 1,990.28 | 2,331.45 | 2,294.32 | 2027 / 2027 / 2029 | yes | n/a |
| DISC_7 | 150 | pool | 605.96 | 0.00 | 148.25 | 1,990.28 | 2,304.82 | 2,261.85 | 2027 / 2027 / 2029 | yes | n/a |
| GIDX_30 | 150 | pool | 604.61 | 0.00 | 148.25 | 1,990.28 | 2,240.95 | 2,221.42 | 2027 / 2027 / 2029 | yes | n/a |
| STAY_60 | 0 | no bed pays | 1,209.22 | 0.00 | n/a (no beds) | 1,990.28 | 2,337.98 | 2,302.47 | 2027 / 2027 / 2029 | n/a (no beds) | n/a |
| STAY_90 | 0 | no bed pays | 1,813.83 | 0.00 | n/a (no beds) | 1,990.28 | 2,337.98 | 2,302.47 | 2027 / 2027 / 2029 | n/a (no beds) | n/a |
| Y2024 | 150 | pool | 713.97 | 0.00 | 257.31 | 2,882.76 | 3,357.37 | 3,308.91 | 2027 / 2027 / 2029 | yes | n/a |
| PPP_25 | 150 | pool | 604.61 | 0.00 | 148.25 | 1,990.28 | 2,337.98 | 2,302.47 | 2027 / 2027 / 2029 | yes | n/a |
| PPP_50 | 150 | pool | 604.61 | 0.00 | 148.25 | 1,990.28 | 2,337.98 | 2,302.47 | 2027 / 2027 / 2029 | yes | n/a |

Dollar figures are per freed or waitlist day, 2023 dollars; grants are per bed-year. Notes: the single-hospital test fails in the new-building run (the largest hospital's planning share of one bed's net saving, 34,158, exceeds that rung's 1,560 grant). The past-pool runs (0.25, 0.5) leave the count at 150: a bed past the pool saves at most about 127,000 a year at 0.5, below its 152,394 cost. In the Medicare run Medicaid saves more on waitlisted days than it pays for the bed days, so `UNIT_PART` is negative.

### Pull at build items

| Item | Status | What was found |
|---|---|---|
| 1. `NF_CAP_COST_2023` | Filled (proxy) | 18.75, from the SPA p. 38a capital price (22.50 / 1.2); the CMS cost-report file has no capital lines. The $462 median re-derived from the same 18 homes in `hi-snf-cost-report-fy2023.csv` (Worksheet A salaries plus non-salary costs / total days): 461.78 (the spec uses 462). |
| 2. `R_CAP(t)` | Filled | 22.50 (Jan 2024; SPA p. 38a, 120% of median); 23.175 and 23.9398 derived for Jan 2025 and Jan 2026. |
| 3. `G_IDX(t)` | Filled (proxy) | CMS SNF market basket, gross, FY2024 to FY2027: 3.0, 3.0, 3.3, 3.3% (CMS final-rule fact sheets); the memos print totals only. |
| 4. `R_NF(t)`, `R_WAIT(t)` re-read | Not done | Deferred by decision 89. The values used match the local rate series (`hi-medicaid-rate-series-2016-2026.csv`), itself read by OCR. QI-2528 (the Jan 2026 drop) could not be downloaded (the link returned a web page). |
| 5. Hospital rows and state-run flags | Filled | Table 18, 2023 and 2024; the rows sum to the statewide totals (checked in the model). Queen's Punchbowl 2023 rebuilt from the county total. Kauai Veterans' flag not verified. |
| 6. `K(1)`, `K(2)` | Filled | 10,255 and 244,869 from 13 Certificate of Need applications read by OCR (12 in the medians); figures not yet checked against the PDFs. |
| 7. Empty beds | Cut | Decision 77. |
| 8. `FMAP` | Filled | MACPAC Exhibit 6: 0.5856, 0.5908, 0.5968 (FY2024 to FY2026). |
| 9. `DISC_RATE` check | Not done | The build-date yield and breakeven inflation were not recorded. |
| 10. `CPI_HI(t)` | Done | FRED CUUSA426SA0, 2011 to 2025, entered on Inputs; 2023 to 2025 match the spec (325.954, 340.197, 348.922). |
| 11. Table 18, other years | Not done | Needed for the reasons test outside the model and the optional Figure 3 comparison. |
| 12. Table 19, 2022 | Not done | Onward-placement evidence. |

`G_HI` uses the spec's 288 for FY2011. The local cost series reads 287.76 for FY2011 and 461.78 for FY2023, which gives 0.040200 instead of 0.040170 (a difference of 0.00003 a year).

### Build choices (conventions the spec left open)

- "Unlimited" rung capacity is entered as `B_MAX` (every bed in the schedule).
- `STOP_REASON` adds two labels the spec does not list: "B_MAX reached" when every bed pays, and "cost" if the next bed is neither past the pool nor on a new rung (not used in any run).
- `HORIZON` is the life of the first rung used (20 years for converted space, 40 in the new-building run). Operating rows run 2024 to 2066 with an in-horizon flag.
- Deflators for rate year `t` use CPI of `t - 1`, projected past 2025 at `INFL`.
- The budget view uses `FMAP` for FY2026, the fiscal year of the Jan 2026 rates it uses.
- In the Scenarios sheet, the single-hospital test uses the hospital with the largest planning share, and the join test uses the lower of `C_AVOID` and the run's `C_AVOID_H` minimum, since `JOIN_MIN` is the same for every hospital.
- Figure 1's history series (cost per day FY2011 to FY2023; every rate memo from 2016 to 2026) comes from the local data files, not from named inputs. The figures themselves are not drawn yet; FigureData holds their series.

### Not yet done

- Micah's verification of the values marked draft in the spec (decision 89): FMAP, the $22.50 capital price, the market basket, GET, the hospital rows and Kauai Veterans' flag, KFF's 2023 figure, the Certificate of Need figures, SHPDA Table 1, and the sources never confirmed (MedPAC's 80%, the 2.28% breakeven inflation, the Capital Group yields, OMB M-25-15).
- Pull at build items 4, 9, 11 and 12.
- Opening the file in Excel (no repair prompt; Checks reads 37 PASS and 5 FAIL).
