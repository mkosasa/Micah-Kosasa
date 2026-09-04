# MBA class notes — transcript → study notes

A Claude skill that turns a raw DLEMBA class-recording transcript into
polished study notes in Micah's house style and files them as a Google Doc in
the matching class folder. The skill is delivered to sessions as a synced
skill (`mba-class-notes`), not run from this repo — this folder is its
version-controlled record.

## What it does

Given a class transcript sitting in the DLEMBA materials inbox on Drive, it
picks out the transcript-shaped files (leaving slide decks, PDFs and readings
alone), identifies which of the four Fall 2026 courses it belongs to, drafts
structured notes organized by concept rather than chronologically, converts
them to a Word file, uploads it to the class folder as a native Google Doc,
and archives the source transcript.

Since 2026-09-04 the drafting stage is **markdown-first**: notes are drafted
and finalized as a `.md` file on disk, and only converted to docx once the
text is settled. Revisions — reordering sections, merging in a second
transcript or slide content, rewording — are plain-text edits to that draft,
not docx rebuilds. Corrections to already-filed notes route back through the
markdown too, except for trivial one-line fixes.

This is separate from the MBA *team meeting* notes routine, which handles the
study group's own working meetings (decisions and action items) rather than
lecture content. This one runs against the hawaii.edu Google account.

## How it runs

Two ways, same skill:

| | |
|---|---|
| **On demand** | Micah asks — "process my class transcript", "turn today's recording into notes" — and the skill triggers manually. |
| **Scheduled** | A cloud routine fires it every morning; see below. |

### Live configuration — scheduled routine

| Field | Value |
|---|---|
| Routine name | DLEMBA transcript → notes daily check |
| Trigger ID | `trig_019ETX5SDhLBgB7btR2vJfZm` |
| Schedule (cron, UTC) | `0 18 * * *` — 18:00 UTC = 8:00 am Pacific/Honolulu, daily |
| Notifications | push on, email off |
| Connectors | Google Drive |

The skill body is the substance; its redacted text is in
[`prompt.md`](prompt.md). Real Drive folder identifiers live only in the live
skill (this repo is public).

## Related

- [mba-team-meeting-notes](../mba-team-meeting-notes/) — the study group's weekly meeting notes + group email
- DLEMBA course materials sweep (`trig_015EQYXTrtp2BY6AMkx1urNf`) — Gmail + Drive inbox → filed course materials, deadline tracker, calendar

## Change process

The live synced skill is the system of record for execution; `prompt.md` here
is the reviewable mirror. When the skill changes:

1. Edit the live skill's `SKILL.md`.
2. Update `prompt.md` to match, keeping the redaction key current. Update this
   README in the **same commit** if the schedule, trigger, or behavior changed.
3. Add a row to `/prompt-log.md` (repo root): date, tool, what was asked, what
   changed, what was done with it.
4. Commit with a message that says what changed and why — not "update".
5. Open a PR using `.github/pull_request_template.md` (Stage: n/a).
