# Current Feature

## Feature 02, First accounting lesson, drafted from sources

## Goal
`src/lesson-01.ts` is the first lesson of superCPE's first course, drafted from
the authoritative text in `sources/asc842/` rather than from memory, with a
review document that lets a licensed CPA verify every claim against its source
before a single ElevenLabs credit is spent. The ESG fixtures are gone.

After this feature the lesson renders silently, `npm run check` passes, and the
human's next step is to read the review document, edit the narration, and run
`npm run generate -- --lesson 01`.

## The course
Course code `ASC842-PCX`, "ASC 842 for Private Companies: The Practical
Expedients." Four lessons; this feature builds lesson 1 and records the outline
of 2–4 in `src/course.ts` so later features and superCPE feature 004 have it.

| # | Lesson | Primary source |
|---|---|---|
| 1 | The short-term lease exception | 842-20-25-2 |
| 2 | The risk-free rate election | 842-20-30-3, ASU 2021-09 |
| 3 | Not separating lease and nonlease components | 842-10-15-37 |
| 4 | Common control arrangements | 842-10-15-3A, 842-20-35-12A, ASU 2023-01 |

Course-level metadata, the same on every lesson:
- `nasbaFieldOfStudy`: "Accounting" (technical)
- `knowledgeLevel`: "intermediate"
- `prerequisites`: "Basic familiarity with ASC 842: identifying a lease,
  classifying it, and recognizing a right-of-use asset and lease liability."
- `advancePreparation`: "None"
- `deliveryMethod`: whatever value the existing lessons use for self-study
- Author fields: keep the `TODO:` placeholders. The human fills these.

## In scope
- Deleting the six ESG fixture lessons, their audio, metadata, and questions
- `src/course.ts` with the course record and four-lesson outline
- `src/lesson-01.ts`, `src/questions-01.json`, `src/audio-meta-01.json` (`{}`)
- Text extraction of the two ASU PDFs into `sources/asc842/*.txt`
- `drafts/ASC842-PCX-01-review.md`, the reviewer's document
- Silent render passing `npm run check`

## Out of scope
- Generating audio. Not one block. The human does this after review.
- Lessons 2–4 beyond their one-line outline entries. Feature 03.
- Any change to slide components, `generate-audio.ts`, `export.ts`, or
  `validate-package.ts`. If the lesson needs a figure kind that does not
  exist, use the closest existing one and note the wish in the changelog.
- Removing `HAZWASTE-01` from superCPE's database. superCPE has no delete yet;
  note it for superCPE feature 004.

## Read first
- `LESSON-RUNBOOK.md`. It is the authoring process for this pipeline: block
  shape, marker placement, `reveals` fallback sizing, `estimatedSeconds` at
  the documented wpm, the sheet-window warning from `check-lessons.ts`. Follow
  it exactly; this feature adds a sourcing discipline on top, it does not
  replace the runbook.
- Every file in `sources/asc842/`. The `.txt` files are Codification
  paragraphs copied verbatim. The PDFs are ASU 2021-09 and ASU 2023-01 (each
  with a Basis for Conclusions), a FASB project summary, and a PCC meeting
  handout. Only the Codification paragraphs and the ASUs are authoritative;
  the other two are context.

## Tasks

### 1. Remove the fixtures
Delete `src/lesson-0[1-6].ts`, `src/questions-0[1-6].json`,
`src/audio-meta-0[1-6].json`, and `public/audio/0[1-6]/`. Remove their entries
from `lessons.ts`, `questions.ts`, and `Root.tsx`. Delete any slide component
in `slides.tsx` that only a fixture lesson used, keeping every generic
`figure`-driven component. Remove the fixture note from README.md. The
changelog records what was deleted; git history keeps it.

### 2. Extract the ASU text
Produce `sources/asc842/ASU_2021-09.txt` and `sources/asc842/ASU_2023-01.txt`
from the PDFs. Use `pdftotext -layout` if present, otherwise a short Python
script with `pypdf`; either way commit the `.txt` and leave the PDFs in place.
Check the extraction by finding the Basis for Conclusions heading and at least
one BC-numbered paragraph in each `.txt`. Add a `sources/asc842/INDEX.md`
listing every file, what it is, and whether it is authoritative.

### 3. `src/course.ts`
```ts
export const COURSE = {
  courseCode: "ASC842-PCX",
  title: "ASC 842 for Private Companies: The Practical Expedients",
  nasbaFieldOfStudy: "Accounting",
  knowledgeLevel: "intermediate",
  prerequisites: "...",
  advancePreparation: "None",
  lessons: [
    { position: 1, lessonId: "ASC842-PCX-01", title: "...", status: "draft" },
    { position: 2, lessonId: "ASC842-PCX-02", title: "...", status: "planned" },
    ...
  ],
} as const;
```
Lesson modules import the course-level fields from here rather than repeating
them. Adjust `PackageLessonMeta` only if that import needs it.

### 4. Draft `src/lesson-01.ts`
Target seven to nine narrated blocks, 900–1,150 words of narration in total
(roughly seven to nine minutes at the runbook's wpm). Suggested arc; adjust if
the source text argues for a different one:

1. The problem: a two-year copier lease and a month-to-month storage unit both
   look like "short" leases to a controller, and only one of them is.
2. What 842-20-25-2 actually says: twelve months or less at commencement, no
   purchase option reasonably certain of exercise. Quote the operative
   sentence.
3. The trap: lease term includes renewal options reasonably certain of
   exercise. A one-year lease with four one-year renewals the lessee expects
   to take is a five-year lease. Where "reasonably certain" comes from.
4. What the election gets you: no ROU asset, no liability, straight-line
   expense. And what it does not get you: the disclosure still exists.
5. Election by class of underlying asset. What "class" means, that it is a
   policy election, and that it is all-or-nothing within the class.
6. When the facts change: a short-term lease that gets extended or a purchase
   option that becomes reasonably certain. Where the guidance sends you.
7. A worked example with numbers (use a `Calc` figure): the same lease
   accounted for both ways, side by side.
8. Summary: three things to check before calling a lease short-term.

Narration style: plain spoken English, numbers written as words (runbook
convention), no bullet-reading. Paraphrase the source; quote at most one
sentence per block, and when quoting, say so in the narration ("the standard
says..."). Never state a rule the sources do not support without flagging it
(task 6).

Block 6 is the one most likely to need a paragraph you do not have
(842-20-25-3 on reassessment). Draft it from general knowledge, flag it, and
let the reviewer fetch the paragraph. Do not silently skip the topic.

### 5. `src/questions-01.json`
Five review questions and three assessment questions, contract shape.
Review questions sit after the block whose content they test; every question
maps to at least one objective id. Assessment questions have four choices and
no true/false framing. Feedback is principles-based (5.01.2.2): why the right
answer is right, what misunderstanding the wrong answers reflect, and which
block to re-study. Each question's stem or feedback should be traceable to a
source file; list the file in a `_source` comment key on each question if the
contract's validator tolerates unknown keys (it does, per feature 01), else in
the review document.

### 6. `drafts/ASC842-PCX-01-review.md`
The document a licensed CPA reads before spending credits. For every block:
- The narration as drafted
- The source file(s) and the specific paragraph or BC number relied on
- Any sentence that is not traceable to a source file, marked `UNSOURCED` with
  a one-line note on what the reviewer should verify or fetch
- The reveal markers and what each reveals

Then for every question: the same, plus which objective it tests.

End with a `## Sources still needed` list. Expect at least 842-20-25-3 and the
short-term lease disclosure paragraph in 842-20-50 to appear there.

### 7. Render and check
`npm run typecheck`, `npm run check`, `npm run render -- --lesson 01` (silent).
Fix every checker error; warnings about sheet-window boundaries are acceptable
only if the block genuinely needs the length and the changelog says so.

## Acceptance
1. `ls src/` shows exactly one lesson, one questions file, one audio-meta file;
   `public/audio/` is empty or absent
2. `sources/asc842/` has both ASU `.txt` files and `INDEX.md`
3. `npm run typecheck` and `npm run check` clean
4. Silent render of lesson 01 plays every sheet with every figure element
   visible by the end of its block
5. `npm run export -- --lesson 01` refuses with the `usingEstimates` message
   naming every narrated block — the correct state until the human generates
6. `drafts/ASC842-PCX-01-review.md` exists, covers every block and question,
   and has a non-empty `Sources still needed` section
7. `npm run generate -- --lesson 01 --dry-run` lists every narrated block as
   pending and spends nothing

## Do not
- Run `npm run generate` without `--dry-run`
- Invent a license number, an author name, or a Codification paragraph number
  that is not in `sources/`
- Reproduce more than one sentence of any source per block

## When done
Append the 02 entry. Under Standards touched: 3.01 (objectives written as
observable outcomes), 3.02.1 (intermediate level with stated prerequisites),
4.01/4.01.1 (drafted from authoritative sources with a traceability record
for the reviewer; not yet reviewed). Under Known gaps: the lesson is
unreviewed and unvoiced; list every `UNSOURCED` flag count and every missing
source. Then stop.
