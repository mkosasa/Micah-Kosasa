# Marginal analysis

Allocating a constrained resource across competing uses by comparing the
marginal contribution of each additional unit, where that contribution
declines as more units are committed.

## Files

- `spec.md` — the fixed inputs: constraints, per-crop economics, caps.
- `model.xlsx` — the Excel model, wired for Solver. Two sheets: `Assumptions`
  holds every spec value in its own cell; `Solver Model` holds the decision
  variables, the profit objective, and the Solver configuration to enter.

## Engagements

| Engagement | Brief | Status |
|---|---|---|
| perfect-competition | `docs/briefs/perfect-competition-brief.md` | Model built, not yet solved |

## Open question in the model

`spec.md` gives a per-bed diminishing-returns rate but not the functional
form. `model.xlsx` interprets it as geometric decline — bed *i* yields
`Price × (1−rate)^(i−1)`, so *n* beds total `Price × (1−(1−rate)^n)/rate`.
Linear decline, or decline applied to price rather than yield, are equally
readable from the spec and would give a different optimum. Verify the form
against the source before trusting a solved result.
