# Current Feature

## Feature 11, SEC-01 body prose, drafted from `sources/sec/`

> 14 is the last entry in `CHANGELOG.md`. Confirm before starting.

## Goal

The eleven `body` sections of SEC-01 are written, from the source set in
`sources/sec/`, with a per-section traceability record in
`drafts/SEC-01-review.md`. Front matter, glossary and appendix are written
too. No questions are written — that is feature 16, and it depends on this
prose existing.

The lesson does not become clean in this feature. Six rule-1 ERRORs
(objectives with no assessment question) are expected to remain, and
`meta.status` stays `"draft"`.

## Why

This is the first real course, for the NASBA Registry initial application.
Field of study Information Technology, knowledge level Basic, audience
licensed CPAs in small firms.

The course targets 3.0 credits:

    30 review + 15 assessment = 45 questions x 1.85 = 83.25
    ~30 min of clips                                = 30.00
    6,615 counted words / 180                       = 36.75
                                                     ------
                                                     150.00  /50 = 3.0

Only the third line is this feature's. Aim slightly over each budget: 7.02.6
rounds down, and a section trimmed in review should not drop the course a
fifth of a credit.

**Sourcing discipline is the point of this feature, not a side condition.**
ASC842-PCX-01 was drafted first and sourced afterward. It shipped with 13
UNSOURCED flags across 8 blocks, five questions inheriting them, and took two
addenda to close — including two rewrites where no paragraph supported what
had been written, and a source file discovered to be mislabeled. The sources
exist this time. Use them while drafting, not after.

`sources/sec/INDEX.md` states each file's standing and is authoritative on
which source governs what. Read it before the first sentence.

## Read first

- `sources/sec/INDEX.md` — every entry, including `## Not yet sourced`
- `src/lesson-01.ts` — sections, objectives, glossary terms, sources
- `docs/course-package.md` — the 7.02.5 role rules, the `front_matter`
  template at the end of the document, and the media rules
- `LESSON-RUNBOOK.md` steps 3 through 7
- `CHANGELOG.md` entries 02 and its two addenda — how the sourcing went wrong
  the first time and what closing it cost

## The sourcing rule

For every sentence in a `body` section, one of three things is true:

1. A file in `sources/sec/` supports it. Record the file and the section or
   technique id in `drafts/SEC-01-review.md`.
2. No file supports it, and it is rewritten into something that is supported.
3. No file supports it and it is kept anyway. Then it is flagged `UNSOURCED`
   in the review document with a one-line note on what a human must verify.

Option 3 is for judgment and framing, not for facts. A number, a mechanism,
or a requirement takes option 1 or option 2.

Four traps recorded in `INDEX.md`, repeated here because they are the ones
that will actually happen:

- **NIST SP 800-63B is withdrawn.** 800-63B-4 supersedes it and 800-63Bsup1
  in their entirety. Rev 4 changed the password guidance substantively. Any
  password sentence written from working knowledge is likely wrong. Grep the
  file and quote the real requirement.
- **WebAuthn Level 3 is for mechanism only**, §1.2 and §1.3. Section 5 onward
  is API surface. If a draft sentence names a `PublicKeyCredential` member,
  the section has drifted out of scope for a Basic course.
- **The CISA fact sheet predates 800-63B-4 by three years.** Where they
  differ on authenticator requirements, 800-63B-4 governs.
- **No statistics.** None is sourced, and a percentage dated to one breach
  report is what will make this course read as stale at the two-year 4.01
  review. Prefer durable mechanism claims.

## Tasks

### 1. Body sections

Word budgets are counted words as `npm run check` reports them. Each entry
gives the argument the section makes, the objective it serves, and where the
support comes from.

**`sec-01` — Why credentials are the target · 600 words · lo-1**
An attacker wants the account, not the machine: a professional's mailbox and
document store are the target, and credentials are what stands in front of
them. Name the four techniques this course covers — phishing, credential
stuffing, infostealers, and attacks on the second factor — and say plainly
that the last three all work on someone who already has MFA. Sets up the
whole guide. Sources: 800-63B-4 for the vocabulary (claimant, verifier,
authenticator, relying party); CISA/NSA/FBI phishing guidance for framing.

**`sec-02` — Phishing: anatomy of a credential harvest · 650 words · lo-1, lo-2**
Walk one credential harvest end to end: lure, landing page, capture, use.
At each step, what is observable to the person being phished. This is where
lo-2's indicators are taught, so be concrete about what can and cannot be
checked — the origin in the address bar can; a padlock and a plausible logo
cannot. Source: CISA/NSA/FBI phishing guidance. Clip `vid-01` lands here.

**`sec-03` — Credential stuffing and password reuse · 600 words · lo-1**
Breach corpora plus reuse equals automated login attempts against services
never themselves breached. Then what actually helps, from 800-63B-4: length
and blocklist screening rather than composition rules. **Grep the real
requirement and quote it by section number — do not write this from memory.**
Sources: 800-63B-4; MITRE ATT&CK T1110 and T1110.004. Clip `vid-02`.

**`sec-04` — Infostealers and the browser credential store · 600 words · lo-1, lo-3**
Malware on the device takes stored passwords *and* session cookies. The
second half is the hinge of the whole course: it introduces the token, which
sections 06 and 07 build on. Sources: ATT&CK T1555 and T1539. **`INDEX.md`
flags this section as under-sourced** — T1555 names the technique but does not
describe the artifact. Either find a published technical analysis and record
its standing, or write at the level the ATT&CK pages actually support.

**`sec-05` — What MFA does and does not stop · 650 words · lo-1, lo-4**
MFA defeats an attacker holding only a password. It does not defeat one who
obtains the ceremony in real time, or the session that follows it. Frame the
next three sections as the three ways that happens. Do not let this become a
case against MFA — the conclusion is that method choice matters, which is
lo-4. Sources: 800-63B-4; CISA fact sheet.

**`sec-06` — Session cookies and token theft · 650 words · lo-3**
Authentication happens once; the session persists. A stolen token is
post-authentication, which is why no second factor is asked for again. This
section carries lo-3 alone and section 11's response ordering depends on it —
make the mechanism land. Source: ATT&CK T1539. Clip `vid-04`.

**`sec-07` — Real-time proxy phishing · 600 words · lo-3, lo-4**
The attacker relays the whole ceremony to the real service as it happens, so
a one-time code is captured and used inside its validity window. This is why
"MFA" as a category does not answer the question. Sources: CISA fact sheet;
800-63B-4 on verifier impersonation resistance. **`INDEX.md` flags this as
under-sourced** for the claim that this is deployed rather than theoretical.
Clip `vid-05`.

**`sec-08` — MFA fatigue and social engineering the second factor · 550 words · lo-4**
Push bombing, and the phone call that asks for the code. Number matching
mitigates push fatigue and is not phishing-resistant — CISA is explicit on
that and the distinction is the point of the section. Source: CISA fact
sheet.

**`sec-09` — Phishing-resistant authentication · 600 words · lo-4**
Origin binding: the assertion is scoped to the relying party, so one produced
at an attacker's site is not valid at the real one. That single property is
what makes this categorically different from everything in 05 through 08, and
it answers lo-4. FIDO/WebAuthn and PKI are the two approaches CISA identifies.
Sources: 800-63B-4; WebAuthn §1.2 and §1.3 for the ceremony; CISA fact sheet.
Clip `vid-06`.

**`sec-10` — Detecting an account takeover · 550 words · lo-5**
What a firm without a security operations team can actually observe: sign-in
locations, unfamiliar sessions, mail rules created without the user, and what
the account's own security page shows. **`INDEX.md` flags this section as
having no source at all.** Expect the heaviest UNSOURCED density here. Write
what the sources support and flag the rest rather than filling the budget.

**`sec-11` — Responding to a suspected takeover · 565 words · lo-6**
Order of operations, and the order is the lesson: revoke sessions first, then
change the password, then re-enroll authenticators. Say why — a password
change alone leaves a stolen session alive, which is section 06's mechanism
paying off. Source: CISA/NSA/FBI phishing guidance.

### 2. Front matter, glossary, appendix

Excluded from the word count (7.02.5), so write them at whatever length is
useful.

`00-front-matter.md` — the "How this course works" block, from the template at
the end of `docs/course-package.md`. This is 4.05.3 item 4 and superCPE
refuses to publish a text lesson without it.

`90-glossary.md` — renders the ten terms in `meta.glossaryTerms`. Those
definitions are drafted and unverified; where 800-63B-4 defines a term, its
wording governs. Check each one and correct both the section file and the
module.

`91-appendix-a.md` — supplementary reference only: the source list with full
citations, and pointers into 800-63B-4 by section. Nothing critical to a
learning objective belongs here, since none of it is counted.

**Every section file must be non-blank.** `check` tolerates an empty file;
`validate-package.ts` does not.

### 3. `drafts/SEC-01-review.md`

Per body section: the text as drafted, the source file and section or
technique id relied on, and every unsupported sentence flagged `UNSOURCED`
with what a human must verify. Then a `## Still needs judgment` list, numbered
J1, J2, and so on.

This is the 4.01.1 accuracy record and the 9.02.2(2)(ii) supporting
documentation — the judgment list behind the numbers that reach the credit
formula. It is what the author works through at runbook step 7.

### 4. Check

    npm run typecheck && npm run check

Run it after each section, not at the end. Report the final per-section table
and the credit estimate.

## Do not

- Write any question. Feature 16.
- Set `meta.status`. That is the author's hand edit after step 7.
- Invent an author name, credentials, jurisdiction, or license number. Those
  fields stay `TODO:`.
- Cite NIST SP 800-63B or SP 800-63Bsup1. Both are withdrawn.
- Cite WebAuthn Level 3 sections 5 and later.
- Write a statistic, prevalence claim, or percentage.
- Pad a section to hit its word budget. An under-budget section that is
  sourced beats an on-budget one that is not; the shortfall is recoverable
  from questions or clips.
- Run `generate` or `render`. This is a text lesson; it has neither.

## Acceptance

1. Fourteen sections on disk, all non-blank, all listed in `meta.sections`.
2. `npm run typecheck` clean.
3. `npm run check`: eleven rule-4 WARNs and six rule-1 ERRORs remain (no
   questions yet); the `[draft]` status WARN remains; no other finding.
4. Counted words at or above 6,615, reported per section.
5. `drafts/SEC-01-review.md` covers all eleven body sections, with sources
   per section, every UNSOURCED sentence flagged, and a non-empty
   `## Still needs judgment` list.
6. Every UNSOURCED flag counted in the changelog entry.

## When done

Append entry 15. Under **Standards touched**: 3.01 (objectives as observable
outcomes), 4.01.1 (drafted with technology from the sources in `sources/sec/`,
with a traceability record; not yet checked by the developer), 4.05.3 (front
matter and glossary), 7.02.5 (roles and what is counted). Under **Known
gaps**: the UNSOURCED count per section, every claim `INDEX.md` listed as not
yet sourced that is still not sourced, and that the lesson is unchecked and
carries no questions. Then stop.
