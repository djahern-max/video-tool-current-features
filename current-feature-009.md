# Current Feature

## Feature NN, Block audio is identified by content, not by block id

> Set NN from the last entry in `CHANGELOG.md` before starting. The last entry
> read at drafting time was 11.

## Goal

`audio-meta-NN.json` stops being trusted on the strength of a block id alone.
A block whose narration has changed, or whose audio was generated under a
different voice or model, is treated as unvoiced — by `render`, by `check`, and
by `usingEstimates` — instead of silently rendering against timings that belong
to different words.

No new capability. This closes a defect that produced two wrong renders in one
authoring session and would have produced a package whose measured durations
did not describe its own narration.

## Why

Two separate holes, found while authoring BALLOON-01.

**1. `durationOf` and `revealsOf` never check that the metadata matches the
block.** From `src/lesson-NN.ts`, the accessors every lesson carries:

```ts
export const hasAudio = (b: Block): boolean => audio[b.id] !== undefined;

export const durationOf = (b: Block): number =>
  audio[b.id]?.durationSeconds ?? b.estimatedSeconds;

export const revealsOf = (b: Block): number[] =>
  audio[b.id]?.reveals ?? b.reveals;
```

`BlockMeta` already carries a `hash`. Nothing reads it. A lookup by `b.id`
succeeds whenever an entry exists under that key, whatever it describes.

Observed: BALLOON-01 was rewritten from 8 narrated blocks to 14. The ids
`block-01` through `block-08` were reused for entirely different narration.
The render composited the old measured durations and the old measured reveal
timestamps onto the new sheets. One sheet sat blank from 0:52 to 1:10 because
it inherited an 18-second reveal written for a different block. Blocks 09–14,
having no entry, rendered silent. Nothing warned.

`usingEstimates` has the same hole, and it is the more serious one: it is
false as soon as an entry exists under every narrated block's id, so a lesson
in this state passes the one gate that exists to stop estimated timings from
reaching a package. `docs/course-package.md` requires
`duration_source: "measured"` and superCPE rejects any other value, on the
strength of this repo attesting the number was measured from the rendered
narration. In the state above that attestation is false. Export's ffprobe
check compares the render against the metadata, so a whole-lesson drift would
likely be caught there — but that is a second line of defence catching a
failure the first line was supposed to make impossible, and it does not fire
on a per-block mismatch that happens to leave the total intact.

**2. The generate cache key does not include voice or model.** `README.md`
records that unchanged blocks are skipped by content hash. Observed: the voice
id in `scripts/generate-audio.ts` was changed from `21m00Tcm4TlvDq8ikWAM` to
`HKFOb9iktHA85uKXydRT`, and the next `npm run generate -- --lesson 01
--dry-run` reported all eight blocks `unchanged, skipped`. The MP3s on disk
were the old voice; the config claimed the new one; `--force` was the only way
to reconcile them, and nothing said so.

`CLAUDE.md` states that voice and model are frozen and that changing them means
regenerating every block of every lesson for consistency. That instruction is
correct and the tooling does not support it: a voice change is invisible to the
cache, so the regeneration it requires can only be triggered by remembering to
pass `--force`.

## In scope

- `hash` becomes the identity check on read, in the accessors every lesson
  module carries and in `scripts/new-lesson.ts`'s scaffold for both kinds
- Voice id and model id become part of what the generate cache hashes
- `check-lessons.ts` reports a block whose audio metadata does not match its
  narration
- `LESSON-RUNBOOK.md` and `CLAUDE.md` updated where they describe the cache

## Out of scope

- Generating any audio. Not one block. Verify against the metadata and the
  hashes; do not spend credits proving a cache miss.
- Any change to voice settings, `speed`, `stability`, or `similarity_boost`.
- Changing what `hash` is computed over beyond adding voice and model — the
  narration normalisation stays as it is.
- The 40–75s sheet window, and the missing first-reveal check. Separate
  features; see Known gaps.

## Read first

- `scripts/generate-audio.ts`, all of it, and in particular what it hashes,
  where it writes the hash, and where it decides to skip.
- `src/blocks.ts` — `BlockMeta` and its `hash` field.
- Any registered lesson's accessor block at the bottom of `src/lesson-NN.ts`.
- `scripts/new-lesson.ts` — both `videoModule` and `textModule`.
- `docs/course-package.md`, the `duration_source` rule.

## Tasks

### 1. Establish what `hash` currently covers

Read it out of `generate-audio.ts` and write it down before changing anything.
The rest of this feature assumes the hash is over the block's speech text; if
it is over something else, stop and report rather than proceeding on the
assumption.

### 2. Voice and model join the hash

Whatever is hashed today, hash it together with the voice id and the model id.
A block generated under a different voice must miss the cache.

This invalidates every existing block of every lesson on its next dry run.
That is the correct behaviour and the point of the change, but it is not a
silent one: the dry-run report must distinguish a miss caused by changed
narration from a miss caused by changed voice or model, so an author reading
the report knows whether they are about to pay for an edit or for a
configuration change.

Do not regenerate anything to prove this. A dry run reporting the misses is
the evidence.

### 3. The accessors check the hash

`hasAudio` becomes true only when an entry exists under the block's id **and**
its `hash` matches the hash of that block's current speech text. `durationOf`
and `revealsOf` fall back to `estimatedSeconds` and `reveals` on a mismatch,
exactly as they do on a missing entry.

The hash function must be the one `generate-audio.ts` uses, not a second
implementation. If that means extracting it to a module both can import, do
that — a hash computed two ways is a defect waiting for a whitespace change.

`usingEstimates` inherits the fix through `hasAudio` and needs no separate
edit. Confirm that it does.

Update `scripts/new-lesson.ts` so both scaffolds emit the corrected accessors.
A scaffold that emits the defect is how this spreads.

### 4. `check-lessons.ts` reports the mismatch

A narrated block with an `audio-meta` entry whose hash does not match its
current narration is a finding.

Level: **ERROR**. It meets the file's own stated bar — decidable from the
lesson's own module and its metadata, and it produces a defective render, which
is the rule the missing-`figure.src` check was set by. It also reaches a
package: this is the condition under which `duration_source: "measured"` would
be untrue.

The message should say what to do, which is regenerate that block, and should
name the block.

Orphaned entries — an `audio-meta` key matching no block id, which is what
retiring or renaming a block leaves behind — are a WARN. They render nothing
and reach no package; they are litter, not a defect.

### 5. Documentation

- `README.md`'s build-order section says unchanged blocks are skipped by
  content hash. Extend it: voice and model are part of that hash, so changing
  either invalidates every block.
- `CLAUDE.md`'s costs-and-secrets section says changing voice or model means
  regenerating every block of every lesson. Note that the cache now enforces
  this rather than relying on the author remembering `--force`.
- `LESSON-RUNBOOK.md`: a block whose narration you edit loses its audio and its
  measured timings until you regenerate it, and `check` will tell you so.

## Acceptance

Run against a scratch lesson created with `npm run new` and removed afterwards,
as features 09 and 10 did. `npm run generate` is not run without `--dry-run` at
any point in this feature.

1. `npm run typecheck` clean.
2. `npm run check` on the repo as it stands: report the result. If any
   registered lesson now errors on a hash mismatch, that is this feature
   finding a real one — do not fix it here; record it in Known gaps with the
   lesson and block named.
3. Hand-edit one narrated block's `narration` in a scratch lesson without
   regenerating. `check` errors on that block, naming it. `usingEstimates` is
   true. Restore byte-identical; the error clears.
4. Hand-edit the `hash` value of one entry in a scratch `audio-meta-NN.json`.
   Same error. Restore byte-identical.
5. Add an `audio-meta` entry under an id no block uses. `check` warns, does not
   error, and exit code is unchanged.
6. Change the voice id in `scripts/generate-audio.ts`, run
   `npm run generate -- --lesson <scratch> --dry-run`, and confirm every block
   reports as a miss attributed to the configuration change rather than to
   changed narration. Nothing is sent, nothing written. Restore the voice id.
7. Confirm a scaffold produced by `npm run new` for both `--kind video` and
   `--kind text` carries the corrected accessors.

## Do not

- Run `npm run generate` without `--dry-run`.
- Delete or regenerate any committed MP3 to make a test pass. `npm run retire`
  is the only supported way to remove audio, and no lesson needs retiring here.
- Change the voice, the model, or any voice setting as part of this feature.
- Reimplement the hash. One function, imported by both call sites.

## When done

Append the NN entry. Under **What changed**, the accessors, the cache key, the
two new findings, the scaffold, and the three documentation files. Under
**Standards touched**, 9.02.2(2)(ii) — the A/V duration retained as supporting
documentation for the word count formula is the number these accessors return,
and returning one measured against different narration makes that
documentation wrong.

Under **Decisions**, record why the mismatch is an ERROR rather than a WARN,
and why an orphaned entry is not.

Under **Known gaps**, record plainly that this defect shipped: block audio was
identified by id alone from the pipeline's first commit, `BlockMeta.hash` was
written but never read, and the condition was reachable by ordinary authoring —
renumbering blocks in a lesson under revision. Name any registered lesson the
new check errors on. Note the two features this one deliberately does not
touch: the 40–75s sheet window, which is wrong for image-heavy lessons, and the
absence of a first-reveal-too-late check, the symmetric case of the
last-reveal warning that already exists.

Then stop.
