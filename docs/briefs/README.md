# Briefs

Documents written before work begins on a capability or initiative.

## Frontmatter schema

Every brief opens with YAML frontmatter:

```yaml
---
type: brief
engagement: perfect-competition   # the slug; matches the decision doc
capability: marginal-analysis     # folder under capabilities/
date: 2026-08-24                  # date the brief was committed
status: committed                 # draft | committed | superseded
hypothesis: "one sentence naming a specific, testable answer"
---
```

- `status: draft` — still being written; not yet frozen.
- `status: committed` — frozen. The hypothesis is not edited after this,
  even if the model later contradicts it.
- `status: superseded` — replaced by a later brief; the frontmatter should
  name which one.
