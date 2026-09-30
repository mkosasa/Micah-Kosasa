# Economic research

Explaining a real economic challenge with course concepts (micro or macro),
analyzing its implications, and recommending a policy or strategy defended
against objections.

**Exercised in:** research paper (`docs/briefs/research-brief.md`,
`capabilities/economic-research/spec.md`, `analysis/research-paper.pdf`)

## Files

- `spec.md`: data sources, model, figures, and success criteria for the
  paper. Rewritten 2026-09-28 for the problem-first brief; `status: built`
  with Audit findings from the 2026-09-29 build and the 2026-09-29 lean
  audit build. The spec for the superseded brief is in the repository
  history.
- `model.xlsx`: the model built 2026-09-29 from `spec.md` and rebuilt the
  same day for the lean audit changes (sheets Inputs, Waitlist, Cost,
  Tests, County, Capacity, Conditions, FigureData, WorkedExample, Checks).
  Values are drafts until I verify them against their sources; the
  verification columns on Inputs track that.
- `README.md`: this file.

Figures live in `analysis/figures/` (the assignment page names `figures/`;
kept under `analysis/` to match this repo's layout). Dated drafts live in
`drafts/`; throwaway work goes in `scratch/`, which is gitignored.

## Engagements

| Engagement | Brief | Status |
|---|---|---|
| research-paper | `docs/briefs/research-brief.md` | First brief committed 2026-09-28, superseded 2026-09-29 by a problem-first brief at the same path (status committed); spec rewritten 2026-09-28 for the new brief; model (`model.xlsx`) and figures a, b, c, d, g (`analysis/figures/`) built 2026-09-29, spec `status: built`; model rebuilt 2026-09-29 for the lean audit changes (anchors, invariants and error scans; WorkedExample; capacity ratio; test 1 on counts; condition (a)); verification of values, my manual-audit run and review of figure captions pending |
