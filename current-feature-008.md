# Current Feature

## Feature NN, `meta.status` is the developer's check, not the 4.02 review

> Set NN from the last entry in `CHANGELOG.md` before starting.

## Goal

video-tool stops claiming that a licensed CPA signs a lesson off inside this
repo. No refusal changes its condition, order, or exit code. What changes is
the paragraph the tooling cites, the record it points at, the word it uses, and
`CLAUDE.md`'s standing rule that states the claim — so a reader of this repo
can tell where the 4.02 content review actually happens, which is superCPE.

This is a correction of stated compliance meaning. It ships no new capability.

## Why

`export.ts` refuses a lesson whose `meta.status` is not `"reviewed"` with a
message citing **4.01.1, 4.02** and directing the reader to
`drafts/<code>-review.md`. `CLAUDE.md` rule 4 goes further and says the flag
"evidences the 4.02 independent content review only," with 4.01.1 explicitly
excluded from it.

Read literally, that tells a licensed CPA to open a markdown file in a git repo
and then hand-edit a TypeScript string in a second file. No CPA will do that,
and they should not: superCPE already built the 4.02 review, and its records
are the ones a NASBA reviewer would ask for.

From superCPE's `COMPLIANCE.md`, the 4.02 rows — authoritative here, because
this repo has no review model at all:

- `course_reviews` holds the review, with `content_updated_at_reviewed`
  snapshotted; `current_review` in `backend/app/services/development.py`
  derives whether the approved review still reviews the current content.
- Reviewers submit at `POST /api/v1/review/courses/{code}/reviews` behind
  `require_role("reviewer", "admin")`. The reviewer has a login, not a repo.
- `review_missing`, `reviewer_is_developer`, and `cpa_participation` are block
  findings in `backend/app/services/readiness.py`; `publish` refuses on any
  block finding.
- Reviewer identity and qualification live in `subject_matter_experts` — name,
  credentials, jurisdiction, license number, status (the 9.02.2(4) row).
- `backend/app/constants/review_attestation.py` holds the versioned statements
  the reviewer's approval puts their name to.

The review is **course-level, against ingested package versions**. It cannot
happen in video-tool, because the thing being reviewed does not exist until
superCPE has it.

`meta.status` is still a real attestation, under a different paragraph. From
the 4.01.1 row: `developer_used_technology` defaults true because superCPE
content is drafted with a language model in video-tool, and the developer of
record is the human who directed and checked that draft. 4.01.1 requires that
developer to review technology-assisted content for accuracy. Flipping the flag
is that check, performed by the developer, before export. That is worth gating
on and the gate stays.

Consequences, all of them wording:

1. `CLAUDE.md` rule 4 asserts the inversion as a standing rule, and the
   Boundary section lists `meta.status` among the things a *package* carries.
   It is not in either branch's manifest — it is an export gate.
2. Export's refusal cites the wrong paragraph and names the wrong record.
3. `drafts/` is documented as 4.02 sign-off evidence. It is not — it is the
   developer's accuracy record under 4.01.1, and supporting documentation under
   9.02.2(2)(ii). It is still preserved by `retire`, for that reason instead.
4. The value `"reviewed"` collides with 4.02's "content reviewer" and is the
   specific word that produced the confusion this feature corrects.

## In scope

- `CLAUDE.md`: rule 4, the Boundary bullet, and the drift listed in task 6
- The refusal messages in both branches of `scripts/export.ts`
- The status comment and the `author` TODO in `scripts/new-lesson.ts`
- The status warning in `scripts/check-lessons.ts`
- Renaming the passing status value (see the decision below)
- `README.md`, `LESSON-RUNBOOK.md` wherever they describe `drafts/`,
  `meta.status`, or the reviewer
- A `CHANGELOG.md` entry recording the correction, and a second short entry
  correcting entry 06's citation (task 7)

## Out of scope

- Any change to what export refuses, or to the order of its refusals
- Any change to `retire`'s preservation of `drafts/` and `sources/` — the
  behavior is right, only its stated reason is wrong
- Any change to the package contract or the manifest
- Anything in superCPE. If superCPE's review flow turns out to disagree with
  the 4.02 rows of its own `COMPLIANCE.md`, report it and stop.
- Building a reviewer surface in video-tool. There is deliberately none.
- Nano learning. `CLAUDE.md`'s rule against inventing a kind for it stands.

## Decision to confirm before starting

**Rename the passing value from `"reviewed"` to `"checked"`.**

The value is internal to video-tool — it gates export and never enters the
manifest. Renaming removes the word that invites the 4.02 reading. It is wider
than one file: `LessonStatus` in `src/types.ts` serves both `meta.status` and
`CourseLesson.status` in `src/course.ts`, `new-lesson.ts` writes it into both,
and `check-lessons.ts` warns when the two disagree.

The alternative is keeping `"reviewed"` and fixing only the prose, which is
lower risk and leaves the collision in place.

The spec below assumes the rename. To back it out, drop task 4 and keep
`"reviewed"` everywhere; nothing else changes.

## Read first

- `CLAUDE.md`, rule 4 and the Boundary section
- `scripts/export.ts`, both status refusals (the video branch's step 2 and the
  identical one at the head of `exportTextLesson`)
- `scripts/new-lesson.ts`, the scaffolded `meta` block and the course entry it
  writes
- `src/course.ts` and `src/types.ts`, for `LessonStatus` and `CourseLesson`
- superCPE's `COMPLIANCE.md`: the 4.02 rows, the 4.01.1 row, the 9.02.2(4) row
- 4.01.1, 4.02, 9.02.2(2)(ii), and 9.02.2(7) in the 2026 Statement, before
  citing any of them

## Tasks

### 1. Find every reader of `meta.status`

`rg 'status' src/ scripts/ *.md` and list them before editing anything. Known:
`export.ts` (two refusals), `check-lessons.ts` (the `[draft]` header label, the
status warning, and the module/course-record mismatch warning),
`new-lesson.ts` (writes `"draft"` into both files), `retire.ts` (the
reviewed-but-unexported warning), `src/course.ts` (`CourseLesson.status`),
`src/types.ts` (`LessonStatus`), and `LessonMeta` in `src/slides.tsx`, which
carries `status: string` — **so a sheet component may render it**. Check
`src/Sheet.tsx` and `src/slides.tsx` for what is drawn from it. If the string
reaches the rendered frame, the rename changes the video, and that must be
stated in the changelog rather than discovered later.

Confirm `meta.status` appears in neither branch's manifest object in
`export.ts`. Task 3 and the rename both rest on this. If it does appear, stop
and report before changing anything.

### 2. `CLAUDE.md` rule 4

Rewrite the second paragraph of rule 4. It currently reads that `meta.status`
evidences 4.02 only and that 4.01.1 is not recorded by the flag. Both halves
are backwards. The replacement says, in substance:

`meta.status` is the developer's own check under 4.01.1: when technology is
used in development, the content developer is responsible for reviewing the
content for accuracy, and generated narration is exactly that case.
`drafts/<code>-review.md` is where that check is recorded. The 4.02 independent
content review is superCPE's — a licensed CPA with a reviewer login, against an
ingested package version — and nothing in this repo evidences it.

Keep the rest of rule 4 unchanged. The two-places instruction is correct:
`CourseLesson.status` mirrors the module's `meta.status`, the module is
authoritative, and `check-lessons.ts` warns when they disagree. Rename the rule
from "Review is a human's signature" to something that does not read as 4.02 —
"The status flag is the developer's signature" or similar.

### 3. `CLAUDE.md` Boundary section

The five-bullet list of what a package carries that a bare MP4 cannot includes
`meta.status: "reviewed"` — 4.02 and 4.02.1. Export refuses anything else.

`meta.status` is not in the manifest (task 1 confirms this). Remove the bullet
from that list. If the export gate is worth stating in Boundary at all, state
it as a gate on what may be built, cited to 4.01.1, and not as package content.

Leave the other four bullets alone.

### 4. Rename `"reviewed"` to `"checked"`

`LessonStatus` in `src/types.ts`; the two comparisons in `export.ts`;
`new-lesson.ts`'s scaffold, including the `status` it writes into the
`CourseLesson` entry; `retire.ts`'s warning; `check-lessons.ts` including the
mismatch warning; `CLAUDE.md` rule 4; and the runbook. `"draft"` is unchanged.

### 5. Correct the refusal, scaffold, and `drafts/` text

Both refusals in `export.ts` become one message, cited to 4.01.1 only, saying
in substance: the lesson's status is *X*; only a checked lesson exports; 4.01.1
makes the content developer responsible for reviewing technology-assisted
content for accuracy, and `drafts/<code>-review.md` is where that check is
recorded; nothing in the tooling sets the flag; the 4.02 content review is
superCPE's, performed by a licensed CPA against the ingested package, and is
not what this flag represents.

Do not cite 4.02 as something this repo satisfies. Naming it as superCPE's is
correct and is the point.

`new-lesson.ts`: the same correction in the scaffolded status comment, and
`author.name`'s TODO changes from "the reviewing CPA's name" to the
author/developer of record — that block becomes `manifest.author` under
9.02.2(4), and superCPE holds the reviewer separately in
`subject_matter_experts`. Correct the same phrase in the scaffolded module
header comment if it appears there.

`check-lessons.ts`: the status warning drops the `drafts/` review framing and
matches the new refusal.

`README.md`, `CLAUDE.md`'s layout table (`drafts/ … the reviewer's surface`),
`LESSON-RUNBOOK.md`, and `retire.ts`'s own printed line each describe the
review document as the reviewer's surface or as sign-off evidence. It is the
developer's accuracy record under 4.01.1 and supporting documentation under
9.02.2(2)(ii) — the judgment list behind the numbers that reach the credit
formula. `retire` still preserves it. Say that reason instead.

`LESSON-RUNBOOK.md`'s "Mark it reviewed" step becomes the developer's check,
and the runbook gains one line after upload: the 4.02 review happens in
superCPE, by a licensed CPA with a reviewer login, against the ingested
package — not here.

### 6. `CLAUDE.md` drift, unrelated to the above

Four corrections while the file is open:

- **`check-lessons.ts` contradicts itself.** "Maintained duplicates" says every
  rule in it is decidable from one lesson's module and its questions file;
  "Commands" says duplicate-stem detection is cross-lesson. The second is true
  — it is why `Finding.lessons` is a list. Reword the first to say the rules are
  decidable from the registered lessons without leaving the repo, which is what
  makes export able to gate on them.
- **The Layout scripts list is incomplete**: `render.ts`, `registry.ts`, and
  `text-preview.ts` are missing.
- **`src/course.ts`'s header comment cites "superCPE feature 004"**, which
  `CLAUDE.md`'s own changelog rule forbids. Replace with the file path, or drop
  the sentence.
- **`src/course.ts` says it is "written by `npm run new` and `npm run retire`,
  not by hand"**, while rule 4 requires a hand edit of a `CourseLesson.status`.
  Both are true of different fields. Add a clause: the commands own the
  structure, the human owns the status, and `check` warns when the two files
  disagree.

### 7. Changelog

Two entries, per `CLAUDE.md`'s format.

**The feature entry.** Under **Standards touched**: 4.01.1 — what `meta.status`
attests and who makes it; 4.02 — named as superCPE's, with the records that
hold it, and explicitly not satisfied by anything in this repo; 9.02.2(2)(ii) —
`drafts/` as supporting documentation. Under **Decisions**: that the gate was
kept rather than removed, and why; and the rename, or its rejection.

**A correction entry.** Entry 06 cites 9.02.1 for what `retire` preserves, and
`retire.ts`'s reviewed-but-unexported warning cites 9.02.1(8). `CLAUDE.md` is
right that 9.02.1 is group programs: self study is 9.02.2, whose element list
ends at item 7, so the transcript-of-record citation is **9.02.2(7)**. Fix the
citation in `retire.ts` and write a new entry saying entry 06 was wrong — the
changelog is append-only and entry 06 is not edited.

Cite each paragraph only after reading it in the 2026 Statement.

## Acceptance

1. `npm run typecheck` and `npm run check` both clean.
2. `rg '4\.02' src/ scripts/ *.md` returns only text naming 4.02 as superCPE's.
3. `rg 'reviewing CPA|reviewer.s surface|signed .* off' src/ scripts/ *.md`
   returns nothing placing the 4.02 review in this repo.
4. `rg '9\.02\.1' src/ scripts/` returns nothing.
5. `CLAUDE.md` rule 4 and the new export refusal say the same thing. An agent
   reading `CLAUDE.md` and then this spec finds no contradiction between them.
6. A scaffolded throwaway lesson exports refused on status, with the new
   message, creating nothing under `dist/`; setting both status fields to
   `"checked"` clears that refusal and the next one fires normally.
7. If task 1 found the status string rendered on a sheet, a render before and
   after the rename differs only in that string, and the changelog says so.

## Do not

- Run `npm run generate` without `--dry-run`
- Change any refusal's condition, order, or exit code
- Add a review model, a sign-off record, or a reviewer command to this repo
- Edit `CHANGELOG.md`'s existing entries
- Edit anything under `../supercpe`

## When done

Append both changelog entries and stop.

## Note for the author

superCPE's `COMPLIANCE.md` has no row for video-tool's `meta.status`, because
from over there it is invisible. Once this lands, the 4.01.1 row's parenthetical
about the developer of record is the only place either repo explains what the
flag is. A one-line addition to that row naming it would close the loop, but it
is a superCPE edit and belongs to a superCPE feature.
