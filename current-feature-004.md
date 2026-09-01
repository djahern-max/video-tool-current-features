# Current Feature

## Feature 04, Lessons 2–4 drafted, and lesson 1's coverage fix

## Goal
The ASC842-PCX course has all four lessons drafted from sources, each with
its own review document, ready for the CPA's pass. Lesson 1 gains a fourth
assessment question so the course's assessment covers every objective.
Nothing is voiced; nothing spends credits.

## In scope
- `src/lesson-02.ts`, `-03.ts`, `-04.ts` with questions, audio-meta stubs,
  review documents, silent renders
- A fourth assessment question for lesson 1, re-exported as a new version
- Course-level question rules that superCPE 007 will enforce, applied here
  first so the course does not fail readiness on arrival
- `COURSE.lessons[].status` reconciled with `meta.status`

## Out of scope
- Generating audio for any lesson
- Any change to slides, `export.ts`, or voice settings
- Regenerating lesson 1's audio. The new question changes `questions.json`
  and therefore the content hash, but no narration; export produces
  version 3 with the same video

## Read first
- `drafts/ASC842-PCX-01-review.md` — the format, the sourcing discipline,
  and the second-addendum section showing what the CPA's edits looked like.
  Lessons 2–4 should need fewer of them.
- `sources/asc842/INDEX.md` and every authoritative file. Lesson 2 leans on
  `842-20-30-3.txt` and `ASU_2021-09.txt` (read its Basis for Conclusions:
  the Board's reason for changing from all-or-nothing to by-class is the
  best teaching moment in the course). Lesson 3 on `842-10-15-37.txt`.
  Lesson 4 on `842-10-15-3A.txt`, `842-20-35-12A.txt`, and
  `ASU_2023-01.txt`'s Basis for Conclusions.
- `LESSON-RUNBOOK.md` for mechanics.

## Course-level question rules
superCPE 007 checks these across the whole course. Apply them here:

1. **Assessment questions per lesson: four**, each mapped to a *different*
   one of the lesson's four objectives, so every objective in the course is
   measured (6.01.2 requires 75 percent; four-of-four per lesson makes it
   100 and removes the question from readiness).
2. **No assessment stem may duplicate a review stem** anywhere in the
   course, including across lessons. Compare after lowercasing, collapsing
   whitespace, and stripping trailing punctuation. A question that tests
   the same fact must ask it differently.
3. **Assessment questions have four choices; review questions at least
   three.** Two-choice review questions do not count toward the minimum, so
   do not write any.
4. **Review questions: five per lesson**, each with `after_block` on the
   block it tests, never two on the same block.
5. Feedback on every question: why the right answer is right, which
   misunderstanding each wrong answer reflects, and which block to re-study.

Add these five rules to `LESSON-RUNBOOK.md` as a "Questions" section so
lesson 5 of some future course inherits them.

## Lesson 1 fix
Add `q-09`, kind assessment, four choices, mapped to whichever of lo-1
through lo-4 the existing three assessment questions leave uncovered (read
`questions-01.json` to find it). Source it, add it to the review document
as a third addendum with its own judgment item for the CPA, and re-export.
`meta.status` stays `reviewed`; the narration is unchanged and the CPA
reviews the one new question in the addendum.

## Lesson arcs
Each lesson: title sheet plus seven to nine narrated blocks, 900–1,150
words, same voice as lesson 1 (the controller, the plain-spoken examples,
"the standard says" before any quotation, one quoted sentence per block at
most). Four objectives each, written as observable outcomes. Adjust the
arcs where the source text argues for it.

### Lesson 2 — The risk-free rate election (842-20-30-3, ASU 2021-09)
1. The problem: a private company with no borrowing history has to find an
   incremental borrowing rate for a five-year office lease. What that
   costs and how uncertain it is.
2. What 842-20-30-3 says: implicit rate when readily determinable;
   otherwise incremental borrowing rate; and for a lessee that is not a
   public business entity, the risk-free rate as an accounting policy
   election by class of underlying asset. Quote the election sentence.
3. What the rate is: a risk-free rate for a period comparable to the lease
   term. Treasury yields as the practical source; matching the term.
4. The history, from the Basis for Conclusions: the original election was
   all-or-nothing across every lease, and the Board changed it to by-class
   because entities wanted it for immaterial classes and not for real
   estate. Why that matters.
5. The trade: a lower rate means a larger liability and asset. A worked
   Calc: the same payments at a risk-free rate versus a plausible IBR.
6. The exception inside the election: if the implicit rate is readily
   determinable, use it, even for a class where the election is made.
7. Who cannot use it: public business entities, and the not-for-profit
   conduit bond obligors the Standard names. Keep this short and sourced.
8. Summary: three questions to ask before electing.

### Lesson 3 — Not separating lease and nonlease components (842-10-15-37)
1. The problem: an office lease with common area maintenance, utilities,
   and a service contract bundled into one monthly payment. What is the
   lease and what is not.
2. The default rule: components are separated and consideration allocated
   on relative standalone price. Cite the paragraphs in the 15-28 to 15-36
   range only if they are in `sources/`; otherwise state the default from
   15-37's own framing and flag.
3. The election: 15-37 lets a lessee, by class of underlying asset, elect
   not to separate nonlease components and instead account for the whole
   as a single lease component. Quote it.
4. What that does: the liability grows to include the nonlease payments;
   expense is the same in total. A Facts figure.
5. Why lessees usually elect it: the allocation work versus the
   balance-sheet cost, and when the cost is too high (large service
   components).
6. Lessor side, briefly: a different test (timing and pattern of transfer)
   and a different outcome; the lessee election is not mirrored. Source
   from 15-42A only if present; otherwise one sentence and a flag.
7. Worked Calc: bundled payment, two treatments, side by side.
8. Summary.

### Lesson 4 — Common control arrangements (842-10-15-3A, 842-20-35-12A,
ASU 2023-01)
1. The problem: the owner's LLC leases the building to the operating
   company on a handshake, and the operating company just spent on
   leasehold improvements. Two questions the old guidance answered badly.
2. Issue 1, the practical expedient in 15-3A: a private company may use
   the written terms and conditions to determine whether a lease exists
   and how to classify and account for it, without assessing enforceability.
   Quote it. What "written" requires and what happens if there is no
   written agreement.
3. From the Basis for Conclusions: why the Board did this (legal
   enforceability was costly to assess and rarely meaningful between
   related parties).
4. Issue 2, leasehold improvements in 35-12A: amortized over the useful
   life of the improvements to the common control group, not the lease
   term, as long as the lessee controls the use of the asset. Quote it.
5. What happens when the lessee stops controlling the asset: the
   improvements transfer, and the accounting for that.
6. The disclosure and the transition in the ASU, briefly.
7. Worked example: a five-year handshake lease with a fifteen-year roof.
8. Summary, and the course close: four elections, one theme — the Board
   letting private companies trade precision for cost where the precision
   was not buying anything.

## Tasks
1. `src/course.ts`: lesson 01 status `reviewed`; lessons 02–04 `draft`.
   Add a comment that `COURSE.lessons[].status` mirrors each lesson's
   `meta.status` and `check` should warn on disagreement; implement that
   warning in `check-lessons.ts`.
2. Lesson 1: `q-09` as above; review doc addendum; re-export → v3.
3. For each of lessons 2–4: `lesson-NN.ts` (`status: "draft"`,
   `position` from the course record, author from `COURSE` or copied from
   lesson 1), `questions-NN.json` (5 review, 4 assessment), `audio-meta-
   NN.json` as `{}`, `drafts/ASC842-PCX-NN-review.md` in the same format
   as lesson 1's, with UNSOURCED flags and a `Sources still needed` list.
   Register in `lessons.ts` and `questions.ts`.
4. A course-wide check in `check-lessons.ts`: duplicate stems across all
   lessons' review and assessment questions; assessment coverage of each
   lesson's objectives; the per-lesson counts. Errors, not warnings; this
   is what superCPE will refuse.
5. Silent renders of 02–04; frame-check the end of every block.
6. `LESSON-RUNBOOK.md`: the Questions section.

## Acceptance
- `npm run typecheck`, `npm run check` clean (including the new
  course-wide checks and the status-mirror warning)
- `ls src/`: four lessons, four questions files, four audio-meta files
- Lesson 1 exported as v3 (`dist/ASC842-PCX-01.zip`) with nine questions
- Lessons 2–4: dry run lists every block pending, spends nothing; export
  refuses each on `draft` status
- Three review documents exist, each with a `Sources still needed` list
  (empty is allowed; absent is not)
- Reviewer's summary at the end of the response: for each of lessons 2–4,
  the count of UNSOURCED flags and the judgment items, so the CPA knows
  the size of each pass before starting

## When done
Append the 04 entry. Standards touched: 3.01 (objectives), 6.01.2
(coverage and duplicate rules applied at authoring), 5.01.2.1 (review
placement). Known gaps: three lessons unreviewed and unvoiced; anything in
the `Sources still needed` lists. Then stop.
