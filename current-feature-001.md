# Current Feature

## Feature 01, Lift the pipeline and export a package

## Goal
The Remotion + ElevenLabs pipeline from `../abacadaba/video` runs in this repo
unchanged, and `npm run export -- --lesson 01` produces a zip that superCPE's
feature 002 accepts with a 201. After this feature the two repos have exchanged
one real lesson.

This spec names no lesson content. Whatever `../abacadaba/video` contains today
is what gets lifted; the first lesson through the exporter is whichever lesson
is `01` there now. Do not reach into abacadaba's git history for older content.

## In scope
- Copying `../abacadaba/video` into this repo's root as it exists today,
  including every lesson, its audio, its metadata, `scripts/check-lessons.ts`,
  and `LESSON-RUNBOOK.md`
- Lesson metadata extended with whatever the manifest needs that it lacks
- A questions file per lesson
- `scripts/export.ts` with local validation mirroring the contract
- A generic `--lesson` flag on render and export

## Out of scope
- Generating any audio. Nothing here spends ElevenLabs credits.
- Changing any slide component, any narration text, or any voice setting.
- LLM drafting of lessons or questions. Feature 02.
- Adding `courseCode`, `position`, `deliveryMethod`, or `revision` to the
  package contract. They stay in the lesson modules; superCPE feature 004 will
  add them to the contract when it builds courses.

## Read first
- `../abacadaba/video/README.md`
- `../abacadaba/CHANGELOG.md`, the two entries titled "Video pipeline 01" and
  "Video pipeline 02"
- `docs/course-package.md`, all of it. The export must produce exactly this and
  the validation rules in the "Rules superCPE enforces on ingest" and questions
  "Rules" sections are what `export.ts` checks before zipping.

## Tasks

### 1. Copy the pipeline
Copy the contents of `../abacadaba/video/` to the root of this repo: `src/`,
`scripts/`, `public/`, `package.json`, `tsconfig.json`, the Remotion config,
`.gitignore`, `README.md`, `LESSON-RUNBOOK.md`. Copy, do not move; leave
abacadaba untouched. Do not copy `.env`, `node_modules/`, or `out/`. Create
`.env.example` with every var `generate-audio.ts` reads.

Verify before going further: `npm install`; `npm run typecheck` (add the script
if absent) clean; `npx tsx scripts/check-lessons.ts` passes if that is what it
does; `npm run generate -- --lesson 01 --dry-run` reports every block unchanged
and spends nothing. Record in the changelog exactly what was lifted: how many
lessons, which have complete audio, which are drafts. If lesson 01 does not
have audio for every narrated block, stop and report.

Normalize the metadata filenames to `src/audio-meta-NN.json` for every lesson
if they are not already, updating imports only.

### 2. Lesson metadata
Each lesson module exports `meta`, and its shape has grown beyond what the
contract needs (`courseCode`, `position`, `deliveryMethod`, `revision`,
`status`, others). Keep all of it. Add only what the manifest needs and `meta`
lacks, using the contract's field names in camelCase:

`lessonId`, `title`, `learningObjectives: {id, text}[]`, `fieldOfStudy`,
`knowledgeLevel`, `prerequisites`, `advancePreparation`,
`sources: {citation, role}[]`, `author: {name, credentials,
licenseJurisdiction, licenseNumber}`, `wordCount`, `avIsAdditionalLearning`.

Put the type in `src/types.ts` if there is no shared place. If `meta` already
carries an equivalent under a different name, map it in `export.ts` rather than
renaming the field and disturbing the slides that read it.

Fill lesson 01's new fields with real values. Objectives are what its blocks
actually teach, written as observable outcomes. `sources` are the regulations
the narration cites. Author fields are `"TODO: ..."` placeholders; do not invent
a license number. Other lessons get the same placeholders and objectives
written from their blocks; if a lesson is too much of a draft to have
objectives, say so in its `meta` rather than inventing them.

**`meta.status` is the single authority on whether a lesson may ship.** Lesson
01's file header still says `DRAFT — NOT REVIEWED` while its `meta.status` is
cleared. Delete the stale header comment on every lesson; from now on the
comment does not exist and the field does. Export refuses any lesson whose
`status` is not the value the runbook uses for "ready" (read `LESSON-RUNBOOK.md`
and `check-lessons.ts` for the vocabulary; do not add a new value).

### 3. Questions
Create `src/questions-NN.json` per lesson in exactly the contract's
`questions.json` shape.

Lesson 01: `../abacadaba/backend/scripts/` seeds questions for it in SQL. Carry
them over, assigning `kind` (review after the block whose content they test,
with `after_block`; assessment with none) and `objective_ids` from the new
`meta`. Assessment questions need at least three choices; if a seeded one has
two, add a plausible third distractor and flag it in the changelog. If the
seed has fewer than eight questions, write more to reach five review and three
assessment, each with feedback that states why the answer is right and which
block to re-study.

Every other lesson: `[]`. Export refuses them anyway.

Add `src/questions.ts` mapping lesson id to its questions file, alongside
`LESSONS` in `lessons.ts`.

### 4. `scripts/validate-package.ts`
A pure function `validatePackage(dir): string[]` that reads a package directory
and returns every rule violation, using the same numbered rules and the same
messages as the contract. It checks manifest fields, the `duration_source`
attestation, the hash, the question rules, and the objective-id references.
It does not run ffprobe; export does that separately because it needs the
rendered file, not the package.

This duplicates superCPE's validator by design. video-tool must be able to say
"this will be rejected" before the human uploads. Note in the file header that
superCPE's `backend/app/services/packages.py` is authoritative and this must
be kept in step with it.

### 5. `scripts/export.ts`
`npm run export -- --lesson 01`. Steps, in order, each refusing with a clear
message on failure:

1. Load the lesson module and its questions.
2. If `meta.status` is not ready: refuse, naming the status.
3. If `usingEstimates` is true: refuse. Message names the blocks without audio
   and says export requires measured narration under 7.02.7.
4. Require `out/lesson-<id>.mp4` to exist. Run ffprobe on it and require the
   duration to match `totalSeconds` within 1 second; otherwise refuse and say
   the render is stale relative to the audio metadata.
5. Build `dist/<lessonId>/`:
   - `video.mp4`, copied from `out/`
   - `transcript.md`: one `## <block id>` heading per narrated block followed by
     `transcriptOf(block)`, markers stripped
   - `questions.json`: verbatim from the questions file
   - `manifest.json`: every field from the contract. `duration_seconds` is
     ffprobe's reading rounded to the nearest integer; `duration_source` is
     `"measured"`; `measured_at` is the latest mtime across the lesson's mp3s
     in ISO 8601 UTC; `narration_blocks` counts blocks with non-empty
     narration; `content_hash` is sha256 over the raw bytes of transcript.md,
     then questions.json, then video.mp4, concatenated, lowercase hex, matching
     the contract's definition exactly; `tts_*` fields read from the constants
     in `generate-audio.ts`. Check `../supercpe/backend/app/services/packages.py`
     rule 3: if it tolerates unknown manifest keys, also write `course_code`,
     `position`, `delivery_method`, and `revision` from `meta`; if it rejects
     them, omit them and note that in the changelog.
6. Run `validatePackage` on the directory. If it returns anything, print every
   message and delete the directory.
7. Zip to `dist/<lessonId>.zip` with the directory as its single top-level
   entry. Print the path and the duration.

Use only `zlib`/`node:fs`/`node:crypto` and one small zip dependency if Node
has no built-in; justify it in the changelog.

### 6. `package.json`
Replace the per-lesson scripts (`render`, `render:02`, `generate`,
`generate:02`) with:

```json
"dev": "remotion studio",
"generate": "tsx scripts/generate-audio.ts",
"render": "tsx scripts/render.ts",
"export": "tsx scripts/export.ts",
"typecheck": "tsc --noEmit"
```

`scripts/render.ts` takes `--lesson <id>` and runs `remotion render
Lesson<id> out/lesson-<id>.mp4`. `generate-audio.ts` already takes
`--lesson`; leave it.

### 7. `.gitignore` and README
Add `dist/` to `.gitignore`. Rewrite `README.md`'s build-order section as:
generate (human, spends credits) → render → export → upload to superCPE. Keep
the two compliance notes at the bottom. State that every exported package is
unreviewed content until a licensed CPA signs it off inside superCPE.

## Acceptance
1. `npm run typecheck` clean; `check-lessons.ts` passes
2. `npm run generate -- --lesson 01 --dry-run`: all blocks unchanged, nothing
   spent. Same for every other lesson.
3. `npm run render -- --lesson 01` completes. Scrub every sheet in Studio to
   confirm reveals land on the narration; this must be unchanged from
   abacadaba's render.
4. `npm run export` on a lesson with no audio refuses with the `usingEstimates`
   message; on a lesson whose `status` is not ready, refuses naming the status.
   Neither creates anything under `dist/`.
5. `npm run export -- --lesson 01` produces `dist/<lessonId>.zip`.
6. Upload that zip to superCPE at http://localhost:5173/admin/packages: 201,
   version 1, duration matching the render. Upload again: 200, not created.
7. `git status` shows `.env`, `out/`, `dist/`, `node_modules/` ignored and every
   `public/audio/**/*.mp3` tracked.

Step 6 is the point of the feature. If superCPE rejects the package, the fix
is in this repo unless the rejection message is itself wrong, in which case
stop and report the discrepancy rather than editing superCPE.

## Do not
- Run `npm run generate` without `--dry-run`
- Edit any `narration` string, any slide component, or `generate-audio.ts`'s
  voice settings
- Touch `../abacadaba` or `../supercpe`

## When done
Append the 01 entry to `CHANGELOG.md` (create it with the same header block
superCPE uses). Under Standards touched: 7.02.7 (export refuses estimated
durations; measured duration and attestation written to the manifest) and
9.02.1(8) (transcript of record exported). Under Known gaps: author fields are
placeholders; which lessons lack audio or questions; the validator is a
maintained duplicate of superCPE's; `courseCode`/`position` are not yet in
the contract. Then stop.
