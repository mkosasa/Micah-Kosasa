# Marginal analysis

Allocating a constrained resource across competing uses by comparing the
marginal contribution of each additional unit committed, where that contribution
changes as more units are committed.

## Files

- `spec.md` — the named contract: season and whole-farm constants, the labor
  supply, the per-crop crop table, the calculation logic in named-range
  notation, the conventions, and the validation rules the built model must pass.
- `README.md` — this file.
- Model file — not yet built.

## Engagements

| Engagement | Brief | Status |
|---|---|---|
| perfect-competition | `docs/briefs/perfect-competition-brief.md` | Spec written; model not built |

## Diminishing-returns form (resolved)

`spec.md` treats each crop's per-bed diminishing-returns rate as compounding on
**labor hours**:

    LABOR_HRS(c, q) = q x LABOR_HRS_WK(c) x SEASON_WEEKS x (1 + DIM_PCT(c)) ^ q

Two other readings were considered and rejected — a summed per-bed geometric
series, and decline applied to price or yield (ruled out because the engagement
fixes price regardless of quantity). See the Conventions section of `spec.md`.
Do not re-open this in the build without a change to the brief.
