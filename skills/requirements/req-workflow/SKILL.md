---
name: req-workflow
description:
  "State machine for the requirements process in the active repo (requirements/):
  register a customer vision as a PCR, elicit clarifications, approve the
  business requirements, generate and freeze the technical spec. The single entry
  point — /requirements routes here, as do 'new request', 'here are the answers',
  'customer approved', 'tech spec', 'freeze PCR-0001'. Not for writing
  application code."
metadata:
  protected: "true"
  copyright: "© 2026 github.com/gothchibjo"
---

# Requirements state machine: vision → approved BR → frozen technical spec

One continuous, resumable process. State lives in the record files, never in
memory: any session can pick a PCR back up by reading the files.

## Record layout

- `requirements/README.md` — registry: one row per PCR
  (`| PCR | Title | Status | Updated |`). Its header carries the glossary paths:
  `glossary:` (writable canon, default `requirements/glossary.md`) and optional
  `glossary_ref:` (an existing project glossary elsewhere, read-only).
- `requirements/glossary.md` — project-wide canonical glossary, append-only,
  `G-<nnn>` ids. Written in Phase 1 by req-elicitation; never written in
  Phase 2.
- `requirements/stakeholders.md` — project-wide stakeholder registry, `S-<nnn>`
  ids; rows are immutable except `status`
  (`active | inactive | superseded-by S-<nnn>`). Fixed path: req-register
  creates the table, req-elicitation fills it.
- `requirements/PCR-NNNN-<slug>/01-vision.md` — canonical record `status` in
  front matter + PART A (verbatim vision, immutable) + PART B (Clarification
  Log, append-only).
- `.../02-business.md` — the BR document: `draft → proposed → approved`,
  `revisions[]`, `approved_by/at`, plus the Terms Used in This PCR and
  Stakeholders for This Change slices.
- `.../03-technical.yaml` — machine-readable technical spec: `draft → frozen`;
  carries the 4 pipeline gates.

## States (canonical, in `01-vision.md` front matter `status`)

`registered → elicitation → br_review → br_approved → technical → frozen`

## 1. Resolve the target record

- Message names `PCR-<nnnn>` → that record.
- Message names `BR-<n>` → find the record whose `02-business.md` defines
  `BR-<n>`.
- No id and exactly one record in the registry → it.
- No id and multiple records → show `status` and ask which.

## 2. Read state before acting — metadata only

Classification needs metadata, not documents. Read:

- `01-vision.md` and `02-business.md` front matter (`status`);
- the target PCR's registry row, and the `glossary:` / `glossary_ref:` keys in
  the registry header;
- `03-technical.yaml` front matter and `pipeline_gates` only — never the
  `requirements[]` array.

Pull the few markers a report needs (`(awaiting)`, `(unresolved)`) with grep
instead of reading the log. Document bodies — PART A, PART B, the BR list, the
REQ list — belong to the phase that consumes them. Reading them here charges
every event twice for the same bytes.

## 3. Classify the event (priority order)

| User phrase →                                                        | Event     | Valid from                                              | Result                                     |
| :------------------------------------------------------------------- | :-------- | :------------------------------------------------------ | :----------------------------------------- |
| new request / register this / formalize the vision + vision text     | new       | —                                                       | registered, then straight into elicitation |
| let's continue / resume / back to it                                 | continue  | elicitation, br_review, technical                       | —                                          |
| answers received / here are the answers / customer replied + content | answers   | elicitation                                             | ← br_review when frontier empties          |
| requirements agreed / approved / BR looks good                       | approve   | br_review, technical (while BR = proposed)              | br_approved                                |
| revisions / comments / rework it + content                           | feedback  | br_review, br_approved, technical (while BR = proposed) | br_review (BR → proposed, `revisions[]`)   |
| technical requirements / tech spec / write the spec                  | technical | br_approved (proposed → structural draft only)          | technical                                  |
| freeze / seal it                                                     | freeze    | technical                                               | frozen                                     |
| status / what's next                                                 | status    | any                                                     | —                                          |

Unrecognized message with a known record: infer the most consistent event; if
that transition is illegal, refuse and list the legal events — never silently
coerce. The table is enforced, not advisory.

Invariant: BR-facing events (`approve`, `feedback`) stay legal whenever the BR
document is not yet `approved` — including the `technical` stage reached via a
pre-approval structural draft. `freeze` is never legal while BR ≠ `approved`
(enforced again inside req-technical-spec).

## 4. Dispatch and persist

Call the Skill tool with the phase name, passing context: record path, event,
any pasted answers/revisions/vision text. Do not restate the record's contents
in the dispatch — the phase reads what it needs itself.

- `new` → `req-register`, then immediately `req-elicitation` (round 1); record
  status `elicitation`.
- `continue` / `answers` → `req-elicitation`; if it reports the BR was composed,
  set status `br_review`.
- `approve` → first resolve who is approving against
  `requirements/stakeholders.md`: the named actor must match exactly one active
  row with `authority: decision-maker`. No name, no match, several matches, or a
  non-decision-maker → stop and ask; add or correct the row from the answer,
  never guess. Then set `status: approved`, `approved_by: <actor> (S-<nnn>)`,
  `approved_at` in `02-business.md`; status `br_approved`.
- `feedback` → in `02-business.md` append to `revisions` `{at, by, note}`, set
  `status: proposed`; record status `br_review`; route content to
  `req-elicitation` only if BR wording needs rework.
- `technical` → `req-technical-spec`; status `technical`.
- `freeze` → `req-technical-spec`; on success status `frozen`.
- `status` → skip dispatch, report from §2 metadata and stop. Do not read the
  documents to answer it.

After every transition update the registry row (`status`, `updated: <today>`).

## 5. Report — once per event

The phase returns a compact result to this skill; the phase does not address the
user. You give the user: record path, what just happened, current state, and the
next legal action. Call out any open glossary conflict. Always end with the
record path (e.g. `requirements/PCR-0001-invoice-export/`) so the process
resumes trivially later.

## 6. Cost discipline

R1–R6 bind this skill and every phase it dispatches. Phase skills inherit them
by reference instead of restating them.

- **R1 Sliced reads.** Read the smallest span that answers the question: front
  matter, one named section (grep for the heading or id), one REQ block, one
  table row. Never read a whole artifact to use one part of it.
- **R2 Edit, never rewrite.** `02-business.md`, `03-technical.yaml` and every
  append-only log are patched in place. Do not regenerate a document to change a
  section, and never rewrite content you did not change.
- **R3 No echo.** After writing, do not print or re-read the written content.
  Validate with a command whose output is a single line (a `yaml.safe_load` that
  prints nothing, a `git diff --stat`).
- **R4 One report.** Phases return `outcome`, changed paths and remaining
  blockers. Only this skill writes to the user.
- **R5 Batch.** One pass over the record serves every item in the request. A
  message covering several items is one event, not a loop of events.
- **R6 Facts, cheaply.** Look up facts with the cheapest sufficient tool: grep
  or a sliced read before a subagent. A subagent is for genuinely multi-file
  exploration and answers in a short digest; it never replaces reading a file
  whose path you already know.

## Notes

- After `frozen`, the development contract is executed through Phase 2:
  `req-develop` (plan → plan_approval → guard_conditions →
  verification_evidence), not through this state machine — but a scope change on
  a frozen PCR comes straight back here as `new` (change control).
- Commits are manual: write files, never commit.
- Artifacts (CL questions, BR, YAML) are written in the record's
  `artifact_language`; PART A stays verbatim in the source language; raw
  customer text is never rewritten.
- History is append-only: PART B and `revisions[]` only grow; corrections go
  through revision entries, never destructive edits.
- Glossary conflicts are hard blockers, not warnings: an unresolved conflict
  sits in the Assumptions Ledger and therefore blocks `freeze`; one found in
  Phase 2 fails a guard (`blocked`) and, if the meaning must change, returns
  here as a new PCR. `glossary_ref:` is read-only — the pipeline never writes
  it.
- Stakeholder inconsistencies are warnings, with one exception: ambiguous
  approval authority stops `approve`. A duplicate actor row, an inactive
  participant or conflicting positions are reported and asked, never blocking.
  The registry is a roster, not history: rows change only by `status`.
