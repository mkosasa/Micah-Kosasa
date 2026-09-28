# Economic research

Explaining a real economic challenge with course concepts (micro or macro),
analyzing its implications, and recommending a policy or strategy defended
against objections.

**Exercised in:** research paper (`docs/briefs/research-brief.md`,
`capabilities/economic-research/spec.md`, `analysis/research-paper.pdf`)

## Files

- `spec.md`: data sources, models, figures, and success criteria for the
  paper. Committed 2026-09-28; `status: built` with Audit findings added
  2026-09-28. Its draft-marked values are still to be verified (decision 89)
  before it moves to `audited`.
- `model.xlsx`: the model built from `spec.md` (2026-09-28). It recalculates
  fully on open and carries the engine's computed values. Every sheet,
  including all 16 scenario runs, was evaluated with a formula engine (no
  error cells; 45 of 45 worked-example anchors pass) and matches an
  independent re-implementation of the spec. The spec's audit is still to
  come.
- `README.md`: this file.

Figures live in `analysis/figures/` (the assignment page names `figures/`;
kept under `analysis/` to match this repo's layout). Dated drafts live in
`drafts/`; throwaway work goes in `scratch/`, which is gitignored.

## Engagements

| Engagement | Brief | Status |
|---|---|---|
| research-paper | `docs/briefs/research-brief.md` | Brief committed (frozen) 2026-09-28; spec committed 2026-09-28 (`spec.md`, status built); model built and evaluated 2026-09-28 (`model.xlsx`); audit findings recorded, verification of draft values pending |
