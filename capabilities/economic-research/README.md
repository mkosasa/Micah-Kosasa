# Economic research

Explaining a real economic challenge with course concepts (micro or macro),
analyzing its implications, and recommending a policy or strategy defended
against objections.

**Exercised in:** research paper (`docs/briefs/research-brief.md`,
`capabilities/economic-research/spec.md`, `analysis/research-paper.pdf`)

## Files

- `spec.md`: data sources, models, figures, and success criteria for the
  paper. Committed 2026-09-28; `status: draft` until the model is built.
  Its draft-marked values are to be verified in the model audit.
- `model.xlsx`: the model built from `spec.md` (2026-09-28). It recalculates
  fully on open. The base sheets, Checks and WorkedExample were evaluated
  with a formula engine and match an independent re-implementation of the
  spec; the 16 scenario runs (Scenarios sheet) are not yet evaluated.
- `README.md`: this file.

Figures live in `analysis/figures/` (the assignment page names `figures/`;
kept under `analysis/` to match this repo's layout). Dated drafts live in
`drafts/`; throwaway work goes in `scratch/`, which is gitignored.

## Engagements

| Engagement | Brief | Status |
|---|---|---|
| research-paper | `docs/briefs/research-brief.md` | Brief committed (frozen) 2026-09-28; spec committed 2026-09-28 (`spec.md`, status draft); model built 2026-09-28 (`model.xlsx`), scenario runs pending verification, audit pending |
