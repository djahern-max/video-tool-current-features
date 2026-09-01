# Current Feature

## Feature 03, Block timings in the manifest, and a real review gate

## Goal
Two small changes the next superCPE feature depends on. The manifest tells
superCPE where each narrated block starts and ends, measured, so review
questions can pause the video at the right second. And `meta.status` becomes
a real gate: export refuses anything a reviewer has not marked reviewed.

## In scope
- `video.blocks` in the manifest, from measured audio metadata
- `meta.status` values and the export rule
- The contract edit, mirrored in the validator

## Out of scope
- Lessons 2–4. Feature 04.
- Any change to slides, narration, or voice.
- Regenerating any audio.

## Contract edit
In `docs/course-package.md`, add to `manifest.video`:

```json
"blocks": [
  { "id": "block-01", "start_seconds": 8.000, "end_seconds": 65.120 },
  ...
]
```

Rules, added to the ingest rules section:
- One entry per narrated block, in playback order, ids matching the
  `## <block id>` headings in `transcript.md`.
- `start_seconds` of the first entry is the title sheet's duration; each
  subsequent `start_seconds` equals the previous `end_seconds`; the last
  `end_seconds` equals `duration_seconds` within 1 second.
- Values come from measured audio; a package whose `duration_source` is
  measured must not carry estimated block timings.
- `questions.json` `after_block` refers to the 1-based index into this list.

Still `package_version: 1`; no real package has been ingested yet. Copy the
edited contract to `../supercpe/docs/course-package.md` so the two stay
byte-identical, and say so in the changelog. superCPE feature 006 enforces
it on its side.

## Status
`meta.status` currently uses `""` for cleared. Replace with an explicit
vocabulary in `src/types.ts`:

```ts
status: "draft" | "reviewed"
```

Export refuses `draft` with a message naming the review document. The human
sets `reviewed` by hand after working through `drafts/<lesson>-review.md`;
nothing in the tooling sets it. Update `LESSON-RUNBOOK.md` step 6 and
`check-lessons.ts` accordingly. Lesson 01 is set to `reviewed` — its review
document's judgment list is closed and the author signed off on 2026-08-27.

## Tasks
1. `scripts/export.ts`: build `video.blocks` from `audio-meta-<id>.json`
   durations in block order, offset by the title sheet's `estimatedSeconds`
   (the only unnarrated block; its length is a fixed render constant, not an
   estimate of speech, so it is not subject to the 7.02.7 rule — say this in
   a comment). Refuse if any narrated block lacks measured audio; that is the
   existing `usingEstimates` refusal and needs no new message.
2. `scripts/validate-package.ts`: the new rules, same numbering scheme as
   superCPE will use (continue from the last existing rule number).
3. `src/types.ts`, `LESSON-RUNBOOK.md`, `check-lessons.ts`: the status
   vocabulary. `check` warns on `draft`, so the state is visible without
   trying to export.
4. `src/lesson-01.ts`: `status: "reviewed"`.

## Acceptance
- `npm run typecheck`, `npm run check` clean
- With lesson 01 audio generated and rendered: `npm run export -- --lesson
  01` produces a zip whose manifest has eight `blocks` entries, contiguous,
  last `end_seconds` within 1 s of `duration_seconds`
- Temporarily setting `status: "draft"` makes export refuse naming the
  review document; restore `reviewed`
- `diff docs/course-package.md ../supercpe/docs/course-package.md` is empty

## When done
Append the 03 entry. Standards touched: 5.01.2.1 (block timings let review
questions be placed throughout the program at measured points); 4.02
(export now requires a recorded review). Then stop.
