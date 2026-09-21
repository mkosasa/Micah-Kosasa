# Skill body — mba-class-notes

Redacted mirror of the live skill's `SKILL.md`. The live copy is the synced
skill `mba-class-notes` (delivered to sessions at
`~/.claude/skills/synced/<id>/mba-class-notes/SKILL.md`) and is the system of
record for execution; this file exists so the wording is under version control
and its edits are reviewable.

This is a mirror, not a loadable skill — it is deliberately **not** named
`SKILL.md`, so Claude Code does not pick it up as a second, placeholder-valued
copy of the skill when working in this repo.

## Redaction key

The repo is public, so Drive identifiers are replaced with placeholders here.
The live skill holds the real values. Placeholder names match those already
used in [`../mba-team-meeting-notes/prompt.md`](../mba-team-meeting-notes/prompt.md)
for the same folders.

| Placeholder | What it is |
|---|---|
| `<DLEMBA_ROOT_ID>` | Drive folder "2026 UH DLEMBA" (root) |
| `<BUS626_FOLDER_ID>` | Drive folder "1. Org Behavior BUS 626" |
| `<BUS620_FOLDER_ID>` | Drive folder "2. Economics BUS 620" |
| `<BUS624_FOLDER_ID>` | Drive folder "3. Accounting BUS 624" |
| `<BUS619_FOLDER_ID>` | Drive folder "4. Stats BUS 619" |
| `<MATERIALS_INBOX_ID>` | Drive folder where raw transcripts and other class materials are dropped |
| `<MATERIALS_PROCESSED_ID>` | "Processed" archive subfolder of the materials inbox |

Instructor names and course numbers are left intact — they are public course
information.

---

## Frontmatter

```yaml
name: mba-class-notes
description: "Turns a raw class-recording transcript for Micah's UH DLEMBA MBA program into polished, structured study notes and files them into the matching class folder in Micah's Google Drive. Trigger this whenever Micah says things like 'process my class transcript', 'turn today's recording into notes', 'make notes from the transcript', or 'file my class notes'. Note: a scheduled task ('DLEMBA transcript → notes daily check') already runs this automatically every morning at 8am Hawaii time — this skill is for when Micah wants to trigger it manually instead of waiting, e.g. right after class. Do NOT use for general note-taking unrelated to the DLEMBA program."
```

---

# MBA class transcript → study notes → filed (Google Drive)

Converts a raw Zoom transcript into clean, structured study notes in Micah's established house style, then saves the finished doc directly into the correct class folder in his Google Drive. Use the `mcp__Google_Drive__*` connector tools (search for them via ToolSearch with query "google drive" if not already loaded).

**This whole pipeline also runs automatically once a day** (a scheduled task named "DLEMBA transcript → notes daily check", 8am Hawaii time, checks the materials inbox and processes anything new). This skill exists so Micah can trigger the same thing on demand — e.g. "process today's transcript now" — without waiting for the next scheduled run.

## Google Drive structure (fixed IDs — don't re-search, just use these)

- Root "2026 UH DLEMBA": `<DLEMBA_ROOT_ID>`
- Org Behavior folder: `<BUS626_FOLDER_ID>` — folder "1. Org Behavior BUS 626", BUS 626 Leadership and Organizational Behavior (Sonia Ghumman)
- Economics folder: `<BUS620_FOLDER_ID>` — folder "2. Economics BUS 620", BUS 620 Micro/Macro-Economic Foundations for Managers (Adam Stauffer)
- Accounting folder: `<BUS624_FOLDER_ID>` — folder "3. Accounting BUS 624", BUS 624 Accounting for Decision Making (Rachel Antal)
- Stats folder: `<BUS619_FOLDER_ID>` — folder "4. Stats BUS 619", BUS 619 Business Statistics & Data Analytics (Eddie Merc)
- Materials inbox: `<MATERIALS_INBOX_ID>` — Micah drops raw class transcripts here after class, alongside other class materials (slide decks, PDFs, readings). It is NOT transcript-only, so filter (see step 1).
- Processed archive: `<MATERIALS_PROCESSED_ID>` (a subfolder of the materials inbox) — move a transcript here once it's been turned into notes, so you can tell what's already been handled.

Finished notes are filed directly into the matching class folder above — there is no separate "Notes" folder. If a transcript belongs to a class not in this table (new term, new course), ask Micah for the folder name and create it under the Root "2026 UH DLEMBA" folder rather than guessing. If a class-folder id above ever returns "not found", search_files for a folder under the root whose title starts with the class number + name (e.g. `title contains 'Accounting BUS 624' and mimeType = 'application/vnd.google-apps.folder'`) and use that.

## Class calendar (Fall 2026, Hawaii time)

- **Org Behavior** (Tue): Sep 1, 8, 15, 22, 29; Oct 6 — then Manoa Saturday Oct 10 (in-person, no transcript expected)
- **Economics** (Thu): Sep 3, 10, 17, 24; Oct 8 — then Manoa Saturday Oct 10 (in-person, no transcript expected)
- **Accounting** (Tue): Oct 13, 20, 27; Nov 3, 10, 17, 24; Dec 1, 8 — then Manoa Saturday Dec 12 (in-person, no transcript expected)
- **Stats** (Thu): Oct 15, 22, 29; Nov 5, 12, 19; Dec 3, 10 — then Manoa Saturday Dec 12 (in-person, no transcript expected)
- No class on holidays: Sep 7, Nov 11, Nov 26, Dec 25.

Use this only to sanity-check which course a transcript belongs to — always process whatever's sitting in the inbox regardless of date.

## Step-by-step workflow

1. **Find the transcript.** List the materials inbox folder. If Micah named a specific file, match it. Otherwise, only pick up files that look like a class-recording transcript — plain text / VTT / SRT / a doc of dialogue, with `HH:MM:SS --> HH:MM:SS` (or `[HH:MM:SS]`) timestamp blocks and `Speaker Name: dialogue` lines. Leave slide decks, PDFs, readings, spreadsheets and other non-transcript materials where they are (don't process or archive them); mention any you skipped in the report.
2. **Read it.** `mcp__Google_Drive__download_file_content` on the transcript (it's plain text, blocks of `HH:MM:SS --> HH:MM:SS` then `Speaker Name: dialogue`). Read for substance only — this raw material must not leak into the final notes as timestamps/speaker names/chit-chat.
3. **Identify the class** from content/keywords, cross-checked against the calendar above. If ambiguous, ask Micah.
4. **Draft the notes** in the house style below, as a **markdown (`.md`) file — not docx**. Save it to a scratch/working location on disk (not Drive). The draft is the working artifact; docx comes later.
5. **Review and finalize the markdown draft.** All revisions happen here, while it's still plain text: rewording, reordering or merging sections, filling gaps, and folding in any additional source material for the same session (a second transcript, slide content, readings). Edits at this stage are plain-text changes to the `.md` file — never a docx rebuild. Move on only once the draft is final.
6. **Build the doc.** Only once the markdown draft is finalized, convert it to docx. Read the `docx` skill's SKILL.md for current gotchas (US Letter page size, no literal `•`/`\n`, table width rules, etc.) before writing the `docx`-npm script. Verify visually (PDF render → images → look at it) before filing.
7. **Upload to Drive.** `mcp__Google_Drive__create_file` with `parentId` = the matching class folder, `contentMimeType` = `application/vnd.openxmlformats-officedocument.wordprocessingml.document`, `base64Content` = the file bytes. Do NOT set `disableConversionToGoogleType` — leave it unset so Drive auto-converts it to a native Google Doc (viewable/editable from any device, which is the whole point of moving this to Drive). Title: `<Session/Topic> Notes - YYYY-MM-DD`.
8. **Archive the source.** `mcp__Google_Drive__update_file` on the transcript, changing its parent to the Processed folder id above.
9. **Report back**: the class, topic covered, and confirmation of where it was filed.

## House style for the notes (match Micah's existing notes exactly)

- First line: plain-text title (course name or session topic, no markdown heading).
- Second line: one *italicized* line summarizing the session's core topic(s).
- Body organized into numbered top-level sections (`# 1. Topic Name`), with `##` sub-sections as needed — organize BY CONCEPT/TOPIC, not chronologically through the transcript.
- Key terms, definitions, and formulas in **bold**, often on their own line.
- Bullet lists for enumerated concepts/criteria; numbered lists for sequential steps (e.g. solving an equation).
- Tables for weight/grading breakdowns, variable comparisons, or end-of-topic summaries.
- Bolded one-line callouts for rules of thumb, e.g. **Key calculation rule:** …
- Logistics/announcements (deadlines, homework, exam notes) go in their own section, separate from concept sections.
- Omit roll call, AV troubleshooting, and off-topic tangents entirely.
- If something is unclear/garbled in the recording, flag it inline with a short bracketed note like `[unclear from recording]` rather than guessing silently.

## Edge cases

- **Multiple unprocessed transcripts:** process one at a time; confirm the class for each before filing.
- **Very long transcript:** still produce one cohesive document, just with more sections.
- **New course/term:** ask for the folder name before creating a new subfolder.
- **Change to notes already filed as a Doc:** route the correction back through the markdown draft rather than patching the Google Doc — locate the working `.md` if it still exists (or re-derive it from the archived transcript), edit it, then re-convert and re-upload per steps 6–7. Exception: a trivial one-line fix (typo, date, a single wrong figure) is fine to edit directly in the Doc.
