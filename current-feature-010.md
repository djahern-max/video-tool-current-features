# Current Feature

## Feature NN, rule 4 becomes coverage, not exclusivity

> Set NN from the last entry in `CHANGELOG.md` before starting.

## Goal

`check-lessons.ts` stops refusing two review questions placed on the same
narrated block or guide section. In its place, a lesson whose `body` sections
do not all carry a review question is reported. No other rule changes, and no
new capability ships.

This is a correction of an authoring rule that reads its own citation too
strictly. It exists so a course can be authored at MasterCPE-style density —
roughly three review questions per chapter — without splitting chapters into
pieces that exist only to hold questions.

## Why

Rule 4 currently errors when two review questions carry the same
`after_block` or `after_section`, citing 5.01.2.1's "sufficient intervals".

The paragraph reads:

> Review questions or other content reinforcement tools must be placed
> throughout the program in sufficient intervals to allow the participant the
> opportunity to evaluate the material that needs to be re-studied.

That is a floor on spacing — do not stack every question at the end — not a
ceiling on density. Nothing in 5.01.2.1 forbids two questions on one section,
and the paragraph's stated purpose (letting the participant find what needs
re-studying) is served *better* by a section that is checked twice than by one
that is not checked at all.

The practical cost of the current reading: a 3.0-credit text course carrying 30
review questions needs 30 distinct `body` sections, which at ~6,600 counted
words is ~220 words a section. The content gets shaped by where questions can
be hung rather than by what a chapter is. That is the tooling driving the
course, which is backwards.

This is the same species of finding as feature 08's `REVIEW_PER_LESSON` /
`ASSESSMENT_PER_LESSON` removal, but not identical, and the difference is worth
recording: those constants had no citation at all. Rule 4 has a real one. The
citation simply supports a different rule than the one that was written.

**What this feature does not do.** It does not touch
`av_is_additional_learning`, which is correctly hardcoded `true` in
`export.ts`'s text branch — `docs/course-package.md` requires it on every media
item of a text package, because a text package has no narration branch and a
clip that reads the guide aloud does not belong in one. It does not touch the
reader's section gate, which is superCPE's and is part of 6.01 completion
verification.

## Read first

- `scripts/check-lessons.ts`, `checkCourseQuestions` — the header comment's
  five-rule list and rule 4's implementation
- `CLAUDE.md` and `LESSON-RUNBOOK.md`, for any prose stating rule 4
- `CHANGELOG.md` entry 08, for how the last rule removal was recorded
- 5.01.2.1 and 5.01.2.2 in the 2026 Statement, before citing either

## Tasks

### 1. Inventory rule 4's readers

`rg -n 'after_block|after_section|sufficient interval|rule 4' scripts/ src/ *.md`
and list every site before editing. Rule 4's ERROR is in
`checkCourseQuestions`; its wording also appears in the header comment's
numbered list. Report anything else found.

Do not touch `export.ts`'s placement refusals. Those check that an
`after_section` names a real section and that assessment questions carry no
placement. They are a different rule and both stay.

### 2. Remove the exclusivity ERROR

Delete the check that refuses two review questions sharing a placement, for
both kinds. A block or section may now carry any number.

### 3. Add coverage as a WARN

For text lessons only: every section whose `role` is `body` should carry at
least one review question. Report each one that does not.

**This is a WARN, not an ERROR, and the reason is recorded here so it is not
"tightened" later without the same argument being made again.** The shipped
`ASC842-GDE` lesson has 8 sections and 5 review questions, so a coverage ERROR
would refuse a lesson that is already in a package superCPE has ingested. An
ERROR is also a stronger claim than 5.01.2.1 supports: the paragraph requires
sufficient intervals across the program, and prescribes counts per credit — it
does not require one question per section.

Do not add a coverage rule for video lessons. A narrated block is not a
chapter; a 14-block lesson at 3 review questions per credit would fail a
per-block rule that no paragraph asks for.

### 4. Rewrite the header comment's rule list

Rule 4's entry currently reads as the exclusivity rule. Replace it with the
coverage rule and its citation, and state in the comment that the exclusivity
reading was removed in changelog entry NN because 5.01.2.1 constrains spacing,
not density. Leave rules 1, 2, 3, and 5 as they are.

Keep the paragraph above the list that says these are video-tool's own
authoring discipline and not a mirror of any superCPE rule. It is still true
of the replacement.

### 5. Correct the prose

Any statement of rule 4 in `CLAUDE.md` or `LESSON-RUNBOOK.md` gets the same
correction. If the runbook's text-lesson section implies one question per
section, fix it there too.

### 6. Verify

- `npm run typecheck && npm run check` clean across all lessons.
- A scratch lesson with two review questions on one section passes.
- A scratch lesson with a `body` section carrying no review question WARNs and
  still exports (WARN is not a gate).
- `ASC842-GDE` still exports, and its export preview's word counts and credit
  estimate are unchanged. Rule 4 has no input to the formula; if any number
  moves, stop and report.

## Changelog

Record under **Standards touched**: 5.01.2.1, quoted, with the spacing-versus-
density distinction stated plainly. Record under **Decisions**: that coverage
ships as a WARN rather than an ERROR, and that `ASC842-GDE`'s 8 sections and 5
review questions is the concrete reason.
