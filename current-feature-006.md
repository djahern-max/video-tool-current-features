# Retire a lesson, and scaffold the next one

The tool has no way to finish with a lesson. Every lesson ever authored stays
registered forever: `src/lesson-NN.ts`, `questions-NN.json`,
`audio-meta-NN.json`, `public/audio/NN/`, an entry in `lessons.ts`, an entry in
`questions.ts`, an entry in `COURSE.lessons`, a review document in `drafts/`.
Nothing removes any of it, so `npm run check` keeps validating dead lessons,
`Root.tsx` keeps registering dead compositions, and the cross-lesson
duplicate-stem check keeps comparing new work against lessons that shipped
months ago.

It has been done twice by hand already — feature 02 deleted the six ESG fixture
lessons and their 42 committed MP3s file by file, and the registry is empty
again today. Both times the hand edit had to get the same three things right:
the two registries, the course record, and the rule that deleting an MP3 means
resetting the matching `audio-meta-NN.json` to `{}` in the same commit.

This feature makes that a command, and gives the opposite command — scaffolding
a new lesson — the same treatment, because "start fresh" is both halves.

**It is a workspace feature. It changes nothing about what a package contains,
how it is validated, or what it attests.** No change to `export.ts`,
`validate-package.ts`, `word-count.ts`, `docs/course-package.md`, any lesson's
content, or any voice/model setting. No audio generated. No renders run.

## Invariants it must not break

1. **Nothing in the tooling touches `meta.status`** (rule 4; 4.01.1, 4.02).
   Retiring removes a lesson; it never downgrades `"reviewed"` to `"draft"`,
   and scaffolding always writes `"draft"`.
2. **An MP3 and its measured timings die together.** `public/audio/NN/` and
   `src/audio-meta-NN.json` are removed in the same operation, never
   separately — otherwise a lesson renders silent while still claiming measured
   durations (CLAUDE.md, "Costs and secrets"; rule 2).
3. **The MP3s are committed source, not build output.** They cannot be
   regenerated identically and regenerating spends credits. Git history is the
   only archive, so retire refuses to delete audio that git is not already
   tracking.
4. **Retire never touches `drafts/` or `sources/`.** The review document is the
   4.02 evidence that a licensed CPA signed the lesson off, and the source
   extractions are what the narration cites; both are program-development
   records under 9.02.1. Deleting them is a human decision made by hand, not a
   side effect of clearing a workspace. Print where they are and leave them.
5. **The exported package is the record downstream, not this repo.**
   `transcript.md` (9.02.1(8)) leaves here inside the package and is retained
   by superCPE. Retiring a lesson whose package was never exported and ingested
   destroys the only copy of its transcript — see task 1's warning.

## 1. `npm run retire -- --lesson NN`

`scripts/retire.ts`, registered in `package.json` alongside the existing
scripts. Takes `--lesson <id>`, plus `--dry-run` and `--force`.

**Refusals, in this order, each naming what is wrong and creating nothing:**

- unknown lesson id, listing the registered ids as `render.ts` does;
- the working tree has uncommitted changes under any path this command would
  remove (`git status --porcelain` over the removal set) — git history is the
  archive, and an uncommitted file has no history;
- any `public/audio/NN/*.mp3` is untracked by git, named file by file
  (invariant 3). `--force` does not override this one.

**Warning, not a refusal:** if `meta.status` is `"reviewed"` and no
`dist/<courseCode>.zip` exists, print that the lesson was reviewed but has no
exported package on disk, and that the transcript of record leaves this repo
only inside a package (9.02.1(8)). Continue after the confirmation prompt.

**Then confirm** — print the removal set and require a typed `y`, unless
`--force`. `--dry-run` prints the same set and exits 0 having changed nothing.

**The removal set:**

    src/lesson-NN.ts
    src/questions-NN.json
    src/audio-meta-NN.json
    public/audio/NN/
    guide/NN/                     (text lessons)
    out/lesson-NN.mp4             (gitignored, reproducible)
    dist/<courseCode>*            (gitignored, reproducible)

**And the registry edits**, which are the part that is easy to get wrong by
hand:

- `src/lessons.ts` — remove the import and the `LESSONS` entry.
- `src/questions.ts` — remove the import and the entry.
- `src/course.ts` — remove the lesson's entry from `COURSE.lessons` and print
  that the remaining `position` values may now have a gap, without renumbering
  them (position is what superCPE ordered the course by; renumbering silently
  is a content decision, not a cleanup).
- `Root.tsx` needs no edit — it derives compositions from `LESSONS`.

Print a closing summary: what was removed, that git history keeps the audio and
the retired lesson file, and the paths under `drafts/` and `sources/` that were
deliberately left behind.

## 2. `npm run retire -- --all`

The clear-the-workspace command. Same guards applied across every registered
lesson, one confirmation for the whole set. On success the repo is back to the
state it is in right now: `LESSONS` empty, `QUESTIONS` empty, `COURSE.lessons`
empty, no `lesson-NN.ts`, no audio.

Acceptance: with zero lessons registered, `npm run typecheck` is clean,
`npm run check` reports zero lessons and zero errors, and `npm run dev` starts
Studio with no compositions. `LessonId` narrowing to `never` must not break any
file that walks `LESSONS`.

## 3. `npm run new -- --lesson NN --code <lessonId> --title "..."`

`scripts/new-lesson.ts`. Optional `--kind text|video`, defaulting to `video`.
Refuses an id already in `LESSONS`, and refuses a `--code` already used by any
registered lesson (a reused package id re-ingests as a *new version* of that
lesson downstream and marks the course's credit and review stale — that is a
deliberate re-export, not a new lesson).

Writes, from the shape of `src/lesson-02.ts`:

- `src/lesson-NN.ts` — a title sheet and one placeholder narrated block, all
  descriptor fields present with `TODO:` values, `status: "draft"`, course-level
  fields imported from `src/course.ts` rather than repeated;
- `src/questions-NN.json` — `[]`;
- `src/audio-meta-NN.json` — `{}`, so `usingEstimates` is true from the first
  moment, as it must be;
- `guide/NN/` with a front-matter and a body section, for `--kind text`;
- `drafts/<code>-review.md` — the review document's headings only, empty;
- the entries in `lessons.ts` and `questions.ts`.

It does **not** write a `COURSE.lessons` entry: which course a lesson belongs to
and at what position is an authoring decision. Print that the entry is needed
before export, since export refuses a lesson with no course entry.

Acceptance: `npm run new -- --lesson 07 --code TEST-07 --title "T"` followed by
`npm run typecheck` and `npm run check` is clean, `check` shows the `[draft]`
warning, `npm run generate -- --lesson 07 --dry-run` lists the placeholder block
as pending and sends nothing, and `npm run export -- --lesson 07` refuses on
status first. Then `npm run retire -- --lesson 07 --force` returns the tree to
`git status` clean apart from `drafts/TEST-07-review.md`.

## 4. Documentation

- `README.md` — the two commands in build order: `new` before step 1, `retire`
  after step 5.
- `CLAUDE.md` — both in the Commands list; a line under "Costs and secrets"
  that `retire` is the only supported way to delete audio, because it enforces
  the audio-meta invariant.
- `LESSON-RUNBOOK.md` — a step 0 (`new`) and a step 10 (`retire`, after the
  package is uploaded and accepted). While in the file, delete the stale
  `~/projects/abacadaba/video` paths and the SSH/database upload steps in step
  9; the upload is the admin packages page and has been since feature 01.

## Acceptance

1. `retire --lesson NN` removes exactly the set above, edits all three
   registries, leaves `drafts/` and `sources/` intact, and leaves
   `npm run typecheck` and `npm run check` clean.
2. All three refusals fire and create nothing: unknown id; a dirty working tree
   naming the file; an untracked MP3 naming the file, still refusing under
   `--force`.
3. The reviewed-but-unexported warning fires, names 9.02.1(8), and does not
   block.
4. `--dry-run` changes nothing, verified by `git status` before and after.
5. `retire --all` leaves the empty-registry state of task 2, all three checks
   passing.
6. `new` → typecheck → check → dry-run generate → export refusal → `retire`
   round trip of task 3 completes, spending no credits and running no render.
7. `git log` still resolves a retired lesson's MP3s and lesson file at the
   commit before the retire.

## When done

Changelog entry per the format. Under **Standards touched**: 9.02.1 — the
records this command deliberately does not delete (`drafts/`, `sources/`) and
why the exported package, not this repo, is the retention artifact; 4.02 —
nothing in the tooling sets, clears, or downgrades `meta.status`. Under
**Decisions**: that git history is the archive, which is why a dirty working
tree and untracked audio are refusals rather than warnings. Under **Known
gaps**: retiring a lesson leaves a gap in `COURSE.lessons[].position` that a
human must reconcile.
