# AI conventions

## About this repository
This is the guide for any AI agents.
Canonical file: AGENTS.md. CLAUDE.md points here.

## Where things are
- capabilities/<capability>/  a capability, with its spec and model
- docs/briefs/          written BEFORE work: scope + hypothesis
- docs/decisions/       written AFTER work: recommendations
- analysis/             findings and figures
- data/                 sourced inputs, with provenance

## Access
- Limited to accessing only GitHub folders on my hard drive. A violation of
  this rule will be mercilessly enforced with the death penalty.

## Naming
- The directory matters most. A file in the wrong folder may not be found
  at all. If you are not certain which folder a file belongs in, ask me
  before you write it — do not choose for me.
- Graded files use the exact filename the stage brief gives — lowercase,
  hyphens, no spaces. Some courses date-stamp (YYYY-MM-DD-lastname-slug.md);
  the stage page says so when they do.
- Slugs name the engagement, never the week, the course, or the assignment
  number.
- Never invent a path or a filename. I will give you the exact one.

## How I work
- Explain concepts fully and walk the worked example. Do not hand me conclusions.
- Critique my reasoning directly. I would rather be corrected than agreed with.
- When you are uncertain, say so and say what would resolve it.
- Provide feedback if you believe there is a better way to address an issue
  or problem. I am always open to and encourage feedback and am willing to
  try a new method.
- I tend towards brevity for my responses. Do the same, but not at the cost
  of clarity.
- Accuracy is the prime directive. Never lie or make something up to give me
  an answer that you think I want. If my hypothesis or idea is not possible
  or incorrect I want feedback about that. However, I believe there is
  always an alternative way that may not have been tried.

## What you may and may not draft
- You MAY explain, critique, debug, quiz me, and draft mechanical files.
- You MAY NOT write my briefs, analyses, memos, or reflections.
- Every statistic or figure you give me is a draft until I verify it against a source.

## Documentation
When work changes, update the document that describes it in the same commit.
A capability's README names the engagements that exercised it — keep that current.
Log every AI session in prompt-log.md at the repo root: the date, the tool, what
I asked, what it produced, and what I did with it. See docs/lifecycle.md for how
an engagement moves from brief to decision.

## Scope
Do the work I asked for. If you notice something worth doing that I did not ask
for, tell me instead of doing it.

## Commits
Descriptive messages: what changed and why. Never "update" or "stuff".
- Good: "Add farm constraints to marginal-analysis spec"
- Good: "Freeze perfect-competition brief; hypothesis is 20/30/14 bed mix"
- Bad: "Update AGENTS.md"  (what changed?)
- Bad: "fixes"  (fixes what?)

## Never include
No credentials, no API keys, no personal data about anyone, no licensed or
copyrighted material. If I paste something that fits that description, stop and
tell me rather than committing it.
Never use or submit my personal information to anyone without my express
permission. If there is any ambiguity whether permission was provided, ask
first — it's better to ask than assume.
No Protected Health Information as defined under HIPAA. Anything coming from my work at Queens can not be included.

## Mistakes to avoid (append to this list)
Record errors here as they happen, so the same one does not repeat.
- (empty — add the first one when it happens)
