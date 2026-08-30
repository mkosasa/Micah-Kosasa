# MBA team meeting notes — Friday check

A scheduled cloud routine (Claude Code "routine" / CCR agent) that turns the
team's weekly Zoom meeting into filed notes and a group email. It is managed
at `claude.ai/code/routines`, not run from this repo — this folder is its
version-controlled record.

## What it does

Every Friday it looks for a new transcript of the 5-person MBA study-group
meeting (Zoom recurring series, plus a manual Drive drop folder), writes
structured notes (Summary / Decisions Made / Action Items), files them as a
Google Doc in the team's shared Drive folder, archives the source
transcript, and emails all five members a link plus their action items.

Action-item names and due dates are cross-checked against the relevant DLEMBA
class syllabus (and, secondarily, the DLEMBA Task Tracker sheet) so the
official assignment name and an accurate due date are used even when the
call referred to the work by an informal shorthand.

This is separate from the two DLEMBA class-notes routines, which handle
lecture transcripts and course materials. Those run against the hawaii.edu
Google account; this one runs against the personal gmail.com account (its
output Doc is owned there, and the class syllabus file IDs it reads are on
that same account).

## Live configuration

| Field | Value |
|---|---|
| Trigger ID | `trig_01BxaTzajWi8SkW9bhrUe2RQ` |
| Schedule (cron, UTC) | `0 0 * * 6` — Saturday 00:00 UTC = Friday 2:00 pm Pacific/Honolulu |
| Model | `claude-sonnet-5` |
| Environment | `env_01SxR7dpuMXzMMMnHy6uwp53` (Default, anthropic_cloud) |
| Connectors | Zoom, Google Drive, Gmail |
| Notifications | push on |

The prompt is the substance of the routine; its redacted text is in
[`prompt.md`](prompt.md). Real Drive/Zoom identifiers and the roster emails
live only in the routine (this repo is public).

## Related routines

- DLEMBA transcript → notes daily check — lecture transcripts → study notes
- DLEMBA course materials sweep — Gmail + Drive inbox → filed course materials, deadline tracker, calendar

## Change process

The live routine is the system of record for execution; `prompt.md` here is
the reviewable mirror. When the routine changes:

1. Edit the live routine (`/schedule`, or `claude.ai/code/routines`).
2. Update `prompt.md` to match the new live prompt, keeping the redaction
   key current. Update this README in the **same commit** if the schedule,
   trigger, connectors, or behavior changed.
3. Add a row to `/prompt-log.md` (repo root): date, tool, what was asked,
   what changed, what was done with it.
4. Commit with a message that says what changed and why — not "update".
5. Open a PR using `.github/pull_request_template.md` (Stage: n/a).
