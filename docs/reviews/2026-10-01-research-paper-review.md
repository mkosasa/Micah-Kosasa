<!-- PR TARGET: https://github.com/mkosasa/Micah-Kosasa | Individual Research Paper -->
# Research paper review — pre-deadline read

**What I read.** `docs/briefs/research-brief.md` and `capabilities/economic-research/spec.md` in full · `drafts/2026-09-30-draft.md` in full, appendices included · the first draft, and the second draft's recommendation from the history · `drafts/README.md` · every sheet of `model.xlsx` at your newest push · the Errors caught entries and the input-verification rows in `prompt-log.md` · the commit history for the paper's files.

**What I did not open this pass.** The three figure images · the build scripts and research notes you keep outside the repository · `data/README.md`, `analysis/README.md`, `AGENTS.md`, `CLAUDE.md` · your Case 1 files. If something in those changes an item below, say so and I will look.

---

This is a serious piece of applied economics on a problem that matters here: Hawaiʻi patients stuck in hospital beds, priced in Medicaid's own rates. You set three tests before you pulled most of the data, and when the one you cared about failed, the paper says "Test 1 was falsified" in plain words. Your Errors caught list has the best catch in the file: the assistant set the hospital's $63 a day beside the home's $11 a day, and you saw that they are per different days. That is a list you wrote, not one I found. Every number I checked in the newest draft traces to the workbook.

**The recommendation and test 3 have parted ways.** Your brief says new transitional capacity "wins if 'no bed available' is the largest reason in most years." The model reads Met, 5 of 8. Your first draft followed it and recommended capacity. The newest draft recommends the hospital-paid top-up first, the option your brief says "follows from the mechanism" that test 1 falsified, and sets test 3 aside: "Test 3 passed, but on the wrong years." That observation is a good one. "No bed" has not led since 2021. But it is a reading of the test made after its result, and your spec's own success criterion asks for a recommendation that "follows from the test results, not from the hypothesis." There are two honest ways out. One is to let the verdict stand, recommend capacity, and report the post-2021 shift as a condition that could reverse it. The other is to keep the offer and say in the paper, in one sentence, that it departs from test 3's verdict and why. Then add a dated correction to the brief and spec, as you have done before. Amending is the sanctioned move, and your history will show it. Item 2 only matters if the offer stays.

**The offer still reads per day.** The model now gives a ceiling per placed patient, $1,232.72 to $19,166.10. The draft, though, still says "a per-day payment atop Medicaid's rate," with the cap given as "$63 to $983 a day," and Appendix C repeats it per day. A reader will price that per day of a nursing-home stay, which is the confusion your own log caught. State the payment per placed patient. Then ask: does the guarantee save the whole 19.5-day average wait? If homes admit "within an agreed time," how many days does the hospital actually save, and what does that do to the cap? Name the window and answer it.

**The 543 days sits outside the repository.** It carries the hypothesis paragraph and the $5,900 test of what a refusal means. Your log says you re-derived it (542.83 across 31 homes) and put it in your research notes. Add it to the spec's inputs with that derivation, so it sits beside every other number in the paper.

**In order:**

1. Decide whether test 3's verdict stands. If it does, recommend capacity. If it does not, say in the paper why you depart from it and record the change in the brief and spec.
2. If the offer stays, state it per placed patient, name the admission window and say what the hospital actually saves under it, and so where the cap belongs.
3. Add the 543-day median stay and its derivation to the spec's inputs.
