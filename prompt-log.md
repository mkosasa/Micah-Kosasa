---
type: prompt-log
owner: micah-kosasa
started: 2026-08-23
---

# Prompt log

| Date | Tool | What I asked | What I got | What I did with it |
|---|---|---|---|---|
| 2026-08-23 | Claude (web) | Write a 150–200 word bio using a set structure (Introduction, Focus Areas, Achievements, Goals) | A request for missing details (name, role, background, focus, achievements, goals) | Provided the details in a follow-up prompt |
| 2026-08-23 | Claude (web) | Refer to Micah-context and ask for any needed additional details | Clarifying questions on purpose, focus, achievements, and goals, informed by stored context | Answered: economics class assignment, professional + community work, short/long-term goals |
| 2026-08-23 | Claude (web) | Provided purpose (DLEMBA economics class assignment), focus (professional career + community work), and goals | First full 4-part draft bio | Reviewed and moved to edits |
| 2026-08-23 | Claude (web) | Edit achievements: no longer on Judicial History board; use ASUO Senate election and budget oversight/allocation work instead | Achievements section rewritten with ASUO Senate detail | Kept the ASUO addition; flagged Judicial History board for later re-inclusion |
| 2026-08-23 | Claude (web) | Add back prior Judicial History board service; look up official names of the Center and the board | Verified official names (King Kamehameha V Judiciary History Center; Friends of the Judiciary History Center of Hawaiʻi) | Kept both names; re-added board service to Achievements |
| 2026-08-23 | Claude (web) | Remove the em dash | Em dashes replaced with commas/periods throughout | Kept the punctuation change |
| 2026-08-23 | Claude (web) | Simplify the DLEMBA sentence to "pursuing an Executive MBA at the University of Hawaii's Shidler College of Business" | Sentence replaced as specified | Kept as written |
| 2026-08-23 | Claude (web) | Add an AI-attribution line and keep the bio at ≤200 words | Attribution line added; other sections trimmed to fit | Kept the attribution line; accepted the trims |
| 2026-08-23 | Claude (web) | Pasted ASUO resume detail; asked to improve the ASUO section | Achievements expanded with budget figures and committee roles | Kept expanded version, then trimmed other sections to stay under 200 words |
| 2026-08-23 | Claude (web) | Pasted my own edited version with an $11M budget line | Flagged a phrasing/accuracy question (Senate vs. my role) for my confirmation | Confirmed and clarified the correct attribution |
| 2026-08-23 | Claude (web) | Noted that as PFC Vice-Chair I drafted the budget for student programs | Budget line attributed to the PFC Vice-Chair role specifically | Kept the correction |
| 2026-08-23 | Claude (web) | Replace "drafted" with "allocated $5.6 million to over 180 programs, departments and contracted services" | Achievement line updated with the allocation figure | Kept as written |
| 2026-08-23 | Claude (web) | Noted ASUO section shouldn't outweigh current career info; asked to simplify ASUO and beef up career section instead | ASUO trimmed to one line; career section expanded with team size and network scope | Kept the rebalanced version as the near-final draft |
| 2026-08-23 | Claude (web) | Build a 1-page resume from my existing Drive resumes, using a given Penn-style markdown skeleton | Full resume mapped into the skeleton's Education/Experience/Leadership/Honors/Skills sections | Used as the first draft |
| 2026-08-23 | Claude (web) | Add my CHRC certification and always include it going forward | CHRC added to the name line and Honors section; standing preference saved | Kept both; preference now applies to future documents |
| 2026-08-23 | Claude (web) | Add DPH State Convention Vice-Chair role, sourced from articles and the Cvent event page | Bullet with convention theme, venue, and delegate count | Kept initially, later shortened during page trimming |
| 2026-08-23 | Claude (web) | Reformat to match a specific GitHub template's raw markdown structure, copy-paste ready | Resume restructured to match the template's headers/spacing exactly | Adopted as the standing format |
| 2026-08-23 | Claude (web) | Add my Privacy Coordinator role; move Laulima Data Alliance into Professional Experience | Privacy Coordinator entry added with bullets; Laulima moved out of Honors into Experience | Kept both changes, bullets trimmed later |
| 2026-08-23 | Claude (web) | Give trim options before cutting bullets for length; confirmed cutting the DPH Executive Director hiring bullet | List of trim candidates across all sections | Used list to guide subsequent trim decisions |
| 2026-08-23 | Claude (web) | Cut the Privacy Coordinator audits bullet and the ASUO $5.6M bullet | Both bullets removed | Kept |
| 2026-08-23 | Claude (web) | Check whether the resume was actually one page | Rendered to PDF; confirmed it was still 2 pages, identified exact overflow content | Used findings to target further trims |
| 2026-08-23 | Claude (web) | Reduce Privacy Coordinator to title and years only, no bullets | Entry trimmed; re-checked page count (still 2 pages) | Kept |
| 2026-08-23 | Claude (web) | Trim the ASUO bullet and DPH convention bullet; remove the Laulima "collaborates with stakeholders" bullet | Three bullets shortened or removed; near-one-page result confirmed | Kept as final content; remaining gap left to formatting, not further cuts |
| 2026-08-23 | Claude (web) | Finalize and prepare the resume for posting on GitHub | Final RESUME.md delivered in template-matching raw markdown | Delivered as the finished file |
| 2026-08-24 | Claude (web) | Critique my perfect-competition brief: name implicit assumptions, name unsupported claims, ask the three questions a client would ask, assess whether the hypothesis is falsifiable | 8 implicit assumptions, 5 unsupported claims (incl. a mesclun fertilizer-cost figure that doesn't match the spec), 3 client questions, and a falsifiability assessment (falsifiable, with a caveat on integer vs. continuous bed counts) | Logged for revision; brief not yet updated |
| 2026-08-30 | Claude Code (/schedule) | Add a syllabus cross-check to the "MBA team meeting notes" routine — for each action item, use the class syllabus for the official assignment name, naming convention, and accurate due date, in both the notes and the group email | Routine prompt updated with a CLASS SYLLABI section, a per-meeting cross-check step, and matching house-style + email + final-report changes | Kept; live routine `trig_01BxaTzajWi8SkW9bhrUe2RQ` updated |
| 2026-08-30 | Claude Code (/schedule) | Confirm the syllabus locations are correct | First-pass folder IDs were copied from the DLEMBA routines and 404 on the personal Google account this routine uses; located the four actual BUS 6xx syllabus files and the DLEMBA Task Tracker sheet on that account | Kept; routine re-updated to read the syllabi by file ID, with the Task Tracker as a secondary due-date cross-check |
| 2026-08-30 | Claude Code | Document the routine and its edit/update process in the repo per AGENTS.md | New `.claude/skills/mba-team-meeting-notes/` (README + redacted `prompt.md`), skills index entry, this log | Committed on a branch and opened as a PR |
| 2026-09-04 | Claude Code | Add a markdown drafting + review stage to the `mba-class-notes` skill before the docx build, and document the skill in the repo | Skill workflow split into 9 steps (draft `.md` → finalize `.md` → convert to docx), an edge case routing corrections back through the markdown, stale changelog block removed; new `.claude/skills/mba-class-notes/` (README + redacted `prompt.md`), skills index entry, this log | Live synced skill updated; repo record committed on a branch |

## Errors caught
- 2026-08-23 — none identified; all factual claims (Queen's size/scope, official org names) were verified via web search before inclusion.
