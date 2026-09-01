# Feature 05 — Text-package authoring and export

superCPE shipped feature 023 Parts A and B on 2026-09-01: the contract now
defines a text package (`kind: "text"`), and superCPE ingests, counts,
reads, and gates them. This feature is Part C — video-tool learns to author
and export one. **Prerequisite: superCPE 023a** (manifest.json joins the
content hash) ships first, so the contract mirrored here is final.

The strategy is in supercpe's `docs/decisions/2026-09-01-text-first.md`:
the study guide is the program, clips are supplements. Read it once.

## In scope
- Mirror the contract (completes supercpe 023 acceptance 10)
- A text-lesson authoring shape inside the existing conventions
- Export of a text package; `validate-package.ts` and `check-lessons.ts`
  extended to the new kind
- Export-time word-count preview and credit estimate
- One minimal real text lesson through the full round trip

## Out of scope
- Any supercpe change (023a is its own, prior feature there)
- Generating audio or touching any existing lesson, slide, or voice setting
- Authoring the first full text course (that is content work, not tooling)
- ASC842-PCX

## Read first
- `docs/course-package.md` — after task 1, the mirrored copy: the text
  layout, section roles, the 7.02.5 exclusion list and word-count rules,
  the 7.02.7 test, the front-matter template, the (post-023a) hash
  definition
- supercpe `CHANGELOG.md` entry 023 — what the server does with what you
  export, including the ingest warnings and the three text publish gates

## Tasks

### 1. Mirror the contract
Copy supercpe's `docs/course-package.md` here byte-identically. Acceptance:
`diff` is empty. Record the supercpe commit copied from in the changelog.

### 2. Text-lesson authoring shape
Stay inside the existing per-lesson-module convention — one `meta` export,
registered in `lessons.ts`/`questions.ts`; do not invent a second config
system:

- `src/lesson-NN.ts` exports `meta` with `kind: "text"` and a `sections`
  array: `{id, file, role, title}` in order, files living under
  `guide/<lessonId>/` as plain markdown. Roles per the contract
  (`front_matter | body | glossary | appendix`).
- `glossaryTerms` on `meta`: term/definition pairs, exported to the
  manifest's `glossary_terms`.
- The front-matter file starts from the contract's "How this course works"
  template; keep it a file the author edits, not a generated string.
- Clips are optional. When present: produced by the existing pipeline,
  listed on `meta.media` with `placement.after_section` and
  `avIsAdditionalLearning: true` per item. A text lesson with no clips
  never touches Remotion, ElevenLabs, or ffprobe-of-a-render.
- `src/questions-NN.json` unchanged in shape except review questions carry
  `after_section` (a real section id) instead of `after_block`.
- `meta.status` keeps its authority: export refuses a lesson whose status
  is not the cleared value, same as today.

### 3. Export
Extend `scripts/export.ts` with the text branch:

- Build `dist/<lessonId>/` per the contract: `manifest.json`,
  `guide/*.md` (copied as authored), optional `media/*`, `questions.json`.
- `content_hash` per the post-023a definition: manifest.json (with the
  `content_hash` key absent) first, then sections in manifest order,
  questions.json, media in manifest order.
- ffprobe each clip for `duration_seconds`, as the video branch does.
- Refuse, each with a clear message:
  - any media item not claiming `av_is_additional_learning: true`, quoting
    7.02.7's test (a clip that narrates the text does not belong in a text
    package)
  - a `word_count` key in the manifest (the contract forbids it; superCPE
    computes)
  - no `body` section, or no `front_matter` section
  - a question placed `after_section` on a section id that does not exist
- Zip and print, as today.

### 4. validate-package.ts and check-lessons.ts
- `validate-package.ts` gains the text-package rules with the contract's
  rule numbers, keeping its header note that supercpe's `packages.py` is
  authoritative and this is a maintained duplicate.
- `check-lessons.ts` extends its course-wide question rules to text
  lessons (`after_section` placement on distinct real sections; the
  duplicate-stem check is already cross-lesson and picks these up).

### 5. Word-count preview and credit estimate
At export (and under `npm run check` for a text lesson), print a
per-section table — section, role, words, **counted or excluded (7.02.5)**
— using exactly the counting rules the contract spells out (fences, HTML,
images, link URLs stripped; headings, link text, inline code kept; a token
counts if it holds a letter or digit). Then one estimate line:
`(counted words ÷ 180 + clip minutes + questions × 1.85) ÷ 50`, labelled
an estimate — superCPE's computation is authoritative and rounding is
course-level (7.01), not lesson-level.

Acceptance: hand-count one short section and match; a word added to the
appendix moves "shipped" and not "counted"; the counted total for the
round-trip lesson (task 6) equals the number superCPE's package summary
shows for it, exactly.

### 6. One real lesson, full round trip
Author a minimal but genuine text lesson — a few short body sections on a
real topic, a small glossary, front matter from the template, one clip if
convenient, 5 review + 4 assessment questions — not a copy of supercpe's
test fixture. Then: export → upload to a local supercpe → package summary
matches the export preview section by section → open the reader preview →
delete the draft.

Record in the changelog the round-trip time from "edit one sentence in a
section file" to "re-exported and re-ingested." That number was the
2026-09-01 walkthrough's Stage 1 question; for the format the catalog will
be built in, it is the authoring loop's speed limit, and 023a means the
re-upload auto-versions with no manual bookkeeping.

## Acceptance
1. Contract diff empty against supercpe's copy; commit recorded.
2. `npm run typecheck` and `npm run check` clean over the new lesson;
   existing lessons and their exports untouched (no audio spent, no
   renders re-run).
3. Export refusals fire for: missing `av_is_additional_learning`, a
   manifest `word_count`, no body section, no front matter, a bad
   `after_section` — nothing created under `dist/` in any refusal case.
4. Task 5's three word-count checks pass.
5. Task 6's round trip completes; superCPE accepts with 201, the summary
   matches the preview, reader renders, draft deleted; round-trip time
   recorded.

## When done
Changelog entry per the format. Under Standards touched: 7.02.5 (the
counting rules and exclusions implemented at authoring time, matching the
server), 7.02.7 (the additional-learning claim required per clip at
export), 5.01.2.1 (`after_section` placement). Note explicitly that
`validate-package.ts` picked up new duplicated rules that must track
`packages.py` by hand — the standing known gap, now larger.
