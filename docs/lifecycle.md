# Engagement lifecycle

Every engagement moves through the same stages. Each stage produces one
artifact, in one place. Nothing here overrides a stage brief: when a stage
gives an exact filename or path, use it.

```
brief  ─▶  spec  ─▶  model  ─▶  analysis  ─▶  decision
(before)   (fixed     (built     (findings     (after —
           inputs)    artifact)  + figures)    recommendation)
```

| Stage | Lives in | What it is |
|---|---|---|
| Brief | `docs/briefs/<engagement>.md` | Written before any work. Problem in my own words, assumptions, a falsifiable hypothesis, and how I would know I was wrong. Frontmatter `status: committed` freezes it — see `docs/briefs/README.md` for the schema. |
| Spec | `capabilities/<capability>/spec.md` | The fixed inputs and constraints the work runs on. |
| Model | `capabilities/<capability>/model.*` | The built artifact (spreadsheet, notebook) that computes the answer. |
| Data | `data/` | Any sourced inputs the model or analysis consumes, with provenance. |
| Analysis | `analysis/`, `analysis/figures/` | Findings produced from running the model, and the charts that go with them. |
| Decision | `docs/decisions/<engagement>.md` | Written after the work. Recommendation, options considered, and what would change it. Links back to the brief — the gap between the hypothesis and the result is the point. |

## Rules that span the stages

- **The hypothesis is not revised to match the model.** If the model
  contradicts the brief, that gap is a finding for the decision doc, not an
  error to edit away.
- **A capability's README names the engagements that exercised it.** Keep it
  current in the same commit as the work.
- **The engagement index in `docs/README.md` lists every engagement** with
  links to its brief and decision. Update it when either is added.
- **Slugs name the engagement** — never the week, the course, or the
  assignment number.
