# Current Feature

## Feature NN, The course-wide question counts come out, and export gates on what remains

Replace `NN` with the next number after the last entry in `CHANGELOG.md`. This
feature produces **two** changelog entries, numbered consecutively.

## Goal

Two things, in this order.

First, remove three constants from `scripts/check-lessons.ts` that were never
derived: the per-lesson review and assessment question counts, and the
four-choice assessment minimum. Question-count minimums are a function of CPE
credit, which superCPE computes and this repo does not. They were a boundary
violation on the day they were written.

Second, make `npm run export` refuse on the rules that remain. Today export
never calls `check-lessons.ts`, so a lesson can fail `npm run check` and export
cleanly anyway. That is the gate that would otherwise have caught a bad package
before it reached a NASBA Registry application.

The order matters. The constants come out first so the extraction does not
harden numbers that are about to be deleted.

## In scope

- `scripts/check-lessons.ts`: delete the constants and the checks that use
  them; narrow rule 1 to coverage; set the choice minimum to 3; rewrite the
  header comment
- `scripts/check-lessons.ts`: extract the per-lesson check into an exported
  pure function
- `scripts/export.ts`: call it, in both the video and text branches
- An ERROR/WARN inventory, produced as a report — **no level changes**
- `LESSON-RUNBOOK.md`, `CLAUDE.md`
- Two `CHANGELOG.md` entries

## Out of scope

- Re-deriving the counts anywhere, in any form. See "Do not."
- Any change to `scripts/validate-package.ts`. It is a maintained duplicate of
  superCPE's `backend/app/services/packages.py` and nothing here touches the
  contract.
- Any change to `src/questions-0[1-4].json`. ASC842-PCX keeps its 5 review + 4
  assessment per lesson. It is a pipeline validator, not shippable content.
- Generating or regenerating any audio. Rendering, if needed, is free.
- Any new course, lesson, or nano-learning support.

## Read first

- `scripts/check-lessons.ts` in full, including `main()`. The extraction
  depends on how findings are currently accumulated, printed, and turned into
  an exit code.
- `scripts/export.ts`, both branches, for the refusal ordering.
- `CLAUDE.md`, "Maintained duplicates."
- `CHANGELOG.md` entry 04, and `current-feature-004.md` if it is still on disk.
  That is where the constants came from and what the first changelog entry
  describes.

---

## 1. Remove the constants

### 1.1 Delete

```ts
const REVIEW_PER_LESSON = 5;
const ASSESSMENT_PER_LESSON = 4;
```

and the two checks that use them, inside `checkCourseQuestions`:

```ts
    if (review.length !== REVIEW_PER_LESSON) {
      err(`${id} questions`, `${review.length} review question(s), rule 4 requires ${REVIEW_PER_LESSON}`);
    }
    if (assessment.length !== ASSESSMENT_PER_LESSON) {
      err(`${id} questions`, `${assessment.length} assessment question(s), rule 1 requires ${ASSESSMENT_PER_LESSON}`);
    }
```

### 1.2 Narrow rule 1 to coverage

Rule 1 currently does two things: every objective is assessed, and no two
assessment questions share an objective. Keep the first. Delete the second — it
only ever held because four questions against four objectives forced a
bijection, so it is the count rule restated. Replace:

```ts
    const coveredBy = new Map<string, string>();
    for (const q of assessment) {
      for (const lo of q.objective_ids ?? []) {
        const already = coveredBy.get(lo);
        if (already) {
          err(`${id} ${q.id}`, `objective ${lo} is already assessed by ${already} — each assessment question maps to a different objective (rule 1)`);
        } else {
          coveredBy.set(lo, q.id);
        }
      }
    }
```

with:

```ts
    // 6.01.2 requires a qualified assessment to measure 75 percent or more of
    // the program's objectives. One question per objective gives 100 and is
    // decidable from the module alone. How MANY questions is a function of
    // credit, which superCPE computes — not checked here.
    const covered = new Set<string>();
    for (const q of assessment) {
      for (const lo of q.objective_ids ?? []) covered.add(lo);
    }
```

The loop below it that reports an objective with no assessment question keeps
its existing message unchanged; adjust it to read from `covered`.

### 1.3 Choice minimum

```ts
      const min = q.kind === "assessment" ? 4 : 3;
```

becomes:

```ts
      // Assessment: 3, matching validate-package.ts's ASSESSMENT_MIN_CHOICES
      // and the Standards, which prohibit forced choice rather than
      // prescribing an option count. The 4 was unrecorded and stricter than
      // this repo's own mirror of packages.py.
      // Review: 3, deliberately stricter than the mirror's 2 — 5.01.2.1 does
      // not count true/false review questions toward the required number, so
      // a two-choice review question is one that does not count.
      const min = 3;
```

Assessment moves 4 → 3. Review stays at 3 and gains a recorded basis. Update
the surrounding error message so it no longer refers to a per-kind minimum.

### 1.4 Rewrite the header comment above `checkCourseQuestions`

It currently reproduces the five rules from `current-feature-004.md`, including
both counts and the four-choice minimum, and asserts that superCPE enforces
them. Replace the whole block with:

```ts
/*
 * video-tool's own authoring discipline. These are not a mirror of any
 * superCPE rule and must not be described as one — an earlier version of this
 * comment claimed superCPE enforced them, sourced only to the feature document
 * that invented them (see CHANGELOG entry NN).
 *
 * Every rule here is decidable from a single lesson's module and questions
 * file. Question-count minimums are not: 5.01.2.1 is three review questions
 * per CPE credit and 6.01.2 is five assessment questions per credit, both
 * functions of credit, which superCPE computes. Adding a question also moves
 * credit by 1.85/50, so the minimum is not even a static function of the
 * content. Do not reintroduce a per-lesson count here.
 *
 *   1. Every objective carries at least one assessment question (6.01.2's 75
 *      percent floor; one-per-objective gives 100).
 *   2. No assessment stem duplicates a review stem anywhere in the course.
 *   3. Assessment and review questions carry at least three choices.
 *   4. Review questions are placed on distinct narrated blocks or sections,
 *      never two on the same one (5.01.2.1's "sufficient intervals").
 *   5. Feedback and an objective mapping on every question.
 */
```

Fill in the real changelog number.

---

## 2. The ERROR/WARN inventory — report only, change nothing

Before the extraction, produce an inventory of every finding
`scripts/check-lessons.ts` can emit, as a table: rule, current level, and
whether that level is still correct once export refuses on it.

The calibration has changed and that is the point of this task. These levels
were set for a script run voluntarily, where WARN means "look at this." As an
export gate, WARN means "we shipped it." Judge each WARN against the new
meaning, not the old one.

Three are already known and should appear in the table with these notes:

- **draft status** — WARN, already an independent export refusal, so the level
  is immaterial at the gate.
- **course-record mirror disagreement** — WARN. `COURSE.lessons[].status`
  mirrors `meta.status`; as a refusal this blocks export on a bookkeeping
  mismatch between a field and its copy.
- **missing rendered clip on a text lesson** — WARN. `out/` is reproducible and
  gitignored, so a WARN may well still be right even as a gate.

The per-block video rules — marker-to-reveal parity, figure-kind pairing,
pacing against words-per-minute — have not been assessed and need reading in
full. Marker parity in particular: `LESSON-RUNBOOK.md` says a mismatch makes a
slide element silently never appear, which is a defect that reaches a rendered
package.

**Change no levels in this feature.** If a WARN looks wrong as a gate, put it
in the table with the reasoning and leave the code alone. Export ships gating on
ERROR only. Promotions are a separate decision.

---

## 3. Extract the check

Extract the per-lesson check into an exported pure function returning
`Finding[]`, the way `validatePackage(dir): string[]` already is. `main()`
becomes a caller: it keeps ownership of printing, the summary line, and the
exit code. Nothing about `npm run check`'s output or exit behaviour changes.

Note the seam. `checkCourseQuestions` runs over every registered lesson because
duplicate stems are cross-lesson; it cannot be made per-lesson. So the exported
function runs the course-wide check as it stands and the caller filters. That
asymmetry is handled in task 4.

No new dependencies.

---

## 4. Call it from export

### 4.1 Placement

Video branch: after the `meta.status` refusal and the `usingEstimates`
refusal, before the `out/lesson-<id>.mp4` existence check. An author should not
be told to render before being told the questions are wrong.

Text branch: immediately after its `meta.status` refusal.

Both: before anything is created under `dist/`, matching every other refusal in
the file.

### 4.2 Scope

Filter to findings naming the lesson being exported. A duplicate-stem collision
involving this lesson names it, so cross-lesson collisions survive the filter
where they should and a different lesson's unrelated breakage does not block
this export.

Verify that a collision between the exported lesson and another lesson produces
a finding labelled with the exported lesson — if the current implementation
labels only one side of a pair, fix the labelling so both sides are named, and
say so in the changelog.

State in the code how the filter identifies a finding's lesson. If the
`Finding.block` labels are not uniform enough to filter on reliably, add a
`lesson` field to `Finding` rather than matching on string prefixes.

### 4.3 The call-site comment

This is required, not optional, and belongs at the call site rather than only
in the commit message:

```ts
  // The course-wide question rules run over every registered lesson —
  // duplicate stems are cross-lesson and cannot be checked from one package.
  // Export acts only on the findings naming this lesson: a collision
  // involving it names it, so it still fires, while an unrelated draft
  // lesson's breakage does not block a finished one. Course-wide check,
  // lesson-scoped action.
```

### 4.4 Refusal behaviour

Refuse on ERROR. Print WARN and continue. Match the existing refusal format in
`export.ts`: print every message, create nothing under `dist/`, exit non-zero.

---

## 5. Documents

- `LESSON-RUNBOOK.md`, "Questions" section: it currently reproduces the same
  five rules including both counts and the four-choice minimum. Rewrite it to
  the five rules in task 1.4, and state that question-count minimums are
  superCPE's because they depend on credit.
- `CLAUDE.md`: record `scripts/check-lessons.ts` as video-tool's own authoring
  discipline, and state that it deliberately mirrors no superCPE code.
  **Do not add it to the "Maintained duplicates" list** — with the counts gone
  it mirrors nothing, and an entry naming an original nobody has read is the
  defect this feature exists to remove.
- `CLAUDE.md` Commands section: if it describes `npm run check` as pre-checking
  superCPE readiness, correct it. Export gates on rules it can evaluate; it
  does not pre-check readiness it cannot see.

---

## 6. Changelog

Two entries, appended in this order, numbered consecutively from the last one
in the file. Use the text below; fill in the numbers, the dates, and the
**Verified** section of the second from real runs.

### Entry one

```markdown
## NN — The course-wide question counts were unsourced
Shipped: YYYY-MM-DD

**What changed**
Nothing in the code. This entry records a defect in an earlier one.

`REVIEW_PER_LESSON = 5` and `ASSESSMENT_PER_LESSON = 4` in
`scripts/check-lessons.ts` entered the repo in feature 04, stated as two of
five "course-level question rules" in `current-feature-004.md` and copied
verbatim into the code comment, `LESSON-RUNBOOK.md`'s Questions section, and
entry 04 of this file. None of the four texts derives either number.

- The 4 has a stated rationale that is not a count: one assessment question per
  objective, because ASC842-PCX lessons each have four objectives. It is that
  course's objective count frozen as a constant. A lesson with a different
  number of objectives fails a rule that was never about counts.
- The 5 has no recorded rationale anywhere. Entry 04 cites 5.01.2.1 under
  Standards touched, but 5.01.2.1 is the placement-and-count paragraph and
  nothing in it produces 5.
- The four-choice assessment minimum has no recorded rationale either, and
  contradicts `validate-package.ts`'s `ASSESSMENT_MIN_CHOICES = 3` — the file
  that is a recorded mirror of superCPE's `packages.py` — and this project's
  policy that three-or-more is a policy choice, the Standards prohibiting
  forced choice rather than prescribing an option count. Two files in this
  repo, both presented as reflecting superCPE, disagreed on one rule, and the
  stricter one was the unrecorded copy.

The claim that superCPE enforces these is circular. "superCPE feature 007
enforces these across the whole course on ingest" appears in the feature
document that invented the rules, in the code comment copied from it, and in
the runbook section copied from that. There is no source outside video-tool.
`check-lessons.ts` was never listed in CLAUDE.md's maintained duplicates, so
unlike `validate-package.ts` it had no recorded original to lose step with.

Downstream consequence, recorded because it is in a shipped package:
ASC842-PCX lesson 01's `q-09` was added in feature 04 to bring that lesson to
four assessment questions. It is a real, sourced question and the objective it
covers (`lo-3`) was genuinely unmeasured before it. But the reason it was
written was a number nobody derived, and it changed the lesson's content hash
and produced version 3 on upload.

**Known gaps**
- Whether superCPE's readiness code enforces anything resembling these counts
  is unverified and was never verified. Nothing in this repo can answer it.
```

### Entry two

```markdown
## NN — The course-wide question counts come out, and export gates on what remains
Shipped: YYYY-MM-DD

**What changed**
- `scripts/check-lessons.ts`: removed `REVIEW_PER_LESSON`,
  `ASSESSMENT_PER_LESSON`, and the two count checks. Question-count minimums
  are 5.01.2.1's three review questions per CPE credit and 6.01.2's five
  assessment questions per credit — both functions of credit, which superCPE
  computes and this repo does not. Adding a question moves credit by 1.85/50,
  so the minimum is not even a static function of the content. Question-count
  minimums and readiness findings are superCPE's; these rules crossed that
  boundary on the day they were written.
- Rule 1 keeps its coverage half (every objective carries at least one
  assessment question) and loses its uniqueness half (no two assessment
  questions on one objective). The uniqueness half existed only because four
  questions against four objectives forced a bijection; it is the count rule
  restated.
- Rule 3: assessment choice minimum 4 → 3, matching `validate-package.ts`.
  Review stays at 3, now with its basis recorded: 5.01.2.1 does not count
  true/false review questions toward the required number.
- Rule 4 (review questions on distinct blocks or sections) is unchanged and is
  what the removed 5 was standing in for. 5.01.2.1 asks for distribution at
  sufficient intervals, which is a placement property, decidable from the
  module alone.
- The header comment no longer claims superCPE enforces these rules.
- `scripts/export.ts` now runs the lesson check and refuses on ERROR, in both
  branches, after the status and `usingEstimates` refusals and before the
  render-exists check, creating nothing under `dist/`. Previously export never
  called `check-lessons.ts`, so a lesson could fail `npm run check` and export
  cleanly — the gate that would otherwise have caught a bad package before a
  Registry application.
- The course-wide question rules still run over every registered lesson;
  export acts only on findings naming the lesson being exported. The asymmetry
  is documented at the call site.
- WARN findings are printed and do not block. Their levels were calibrated for
  a voluntarily run script and are reassessed in this feature's inventory; no
  level was changed here.
- `LESSON-RUNBOOK.md` and `CLAUDE.md` updated. `CLAUDE.md` records
  `check-lessons.ts` as video-tool's own authoring discipline and explicitly
  not a maintained duplicate.

**Standards touched**
- 5.01.2.1, 6.01.2 — the per-credit minimums, named as superCPE's to evaluate
  rather than restated here as per-lesson constants.
- 6.01.2 — objective coverage retained: one assessment question per objective
  satisfies the 75 percent floor from the manifest alone.
- 4.01.1, 4.02 — export's review gate now runs the authoring checks it always
  claimed were run before a package shipped.

**Verified**
- (fill from real runs — see Acceptance)

**Known gaps**
- ASC842-PCX still carries 5 review + 4 assessment per lesson. It is a pipeline
  validator, not shippable content, and its 36-question shape is not a template
  for the QAS course.
- The ERROR/WARN levels are unchanged from when they were set for a voluntary
  script. The inventory produced in this feature lists the ones worth revisiting
  now that ERROR blocks an export.
```

---

## Acceptance

1. `npm run typecheck` clean.
2. `npm run check` on the repo as it stands: 0 errors, and the same warnings as
   before this feature. Removing the counts must not change ASC842-PCX's result
   — it already satisfied 5/4.
3. Negative test, by temporarily editing a questions file and restoring it
   byte-identical afterwards (`git diff` empty), as feature 04 did:
   - an assessment question with 3 choices no longer errors; with 2, it does
   - two assessment questions on the same objective no longer error
   - an objective with no assessment question still errors
   - two review questions on the same `after_block` still error
   - a duplicated stem across two lessons still errors
   - a removed `feedback` still errors
4. `npm run export -- --lesson 01` succeeds and produces a package byte-identical
   to what it produced before this feature, except that nothing about the
   package should have changed at all — confirm the `content_hash` is unchanged.
   If it is not, stop and report; this feature must not alter package contents.
5. Export refusal path: temporarily break one question in `questions-01.json`
   (remove its `feedback`), run `npm run export -- --lesson 01`, confirm it
   refuses with the finding, creates nothing under `dist/`, and exits non-zero.
   Restore byte-identical.
6. Export scope filter: temporarily break a question in a *different* registered
   lesson, confirm `npm run export -- --lesson 01` still succeeds. Then create a
   stem collision between that lesson and lesson 01 and confirm export of
   lesson 01 refuses. Restore byte-identical.
7. `grep -rn "REVIEW_PER_LESSON\|ASSESSMENT_PER_LESSON" .` returns nothing
   outside `CHANGELOG.md` and this feature file.
8. No occurrence of a claim that superCPE enforces the course-wide rules
   survives in `scripts/`, `CLAUDE.md`, or `LESSON-RUNBOOK.md`.

## Do not

- Reintroduce a question count in any form, including as a warning, a suggested
  range, a default, or a comment recommending one. If the extraction seems to
  need a count, it does not; stop and report.
- Add `scripts/check-lessons.ts` to CLAUDE.md's maintained-duplicates list, or
  name a superCPE file as its original. No path to superCPE's readiness module
  is to be guessed, inferred, or recalled.
- Change any finding's ERROR/WARN level.
- Edit `scripts/validate-package.ts`, `docs/course-package.md`, or any
  `src/questions-NN.json`.
- Remove `q-09` from lesson 01. It is a real question covering a real objective
  and removing it would change a shipped package's content hash.
- Run `npm run generate` without `--dry-run`.
- Touch `../supercpe` or `../abacadaba`.

## When done

Append both changelog entries, report the ERROR/WARN inventory table, and stop.
List anything encountered that was out of scope rather than fixing it.
