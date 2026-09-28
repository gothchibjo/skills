---
name: req-workflow
description:
  "State machine for the end-to-end requirements process in the active repo
  (requirements/): register an incoming customer vision as a PCR, elicit
  clarifications, approve the business requirements with the customer, generate
  the machine-readable technical spec, and freeze it. The single entry point.
  Trigger when the user talks about requirements management, PCRs or BRs, a
  customer vision/change request, or an RFI from a stakeholder — e.g. 'new
  request', 'register the requirements', 'continue PCR-0001', 'resume PCR-0001',
  'customer answers received', 'here are the answers', 'PCR-0001 requirements
  agreed', 'customer approved', 'customer revisions', 'write the technical
  spec', 'tech spec', 'freeze PCR-0001', 'PCR-0001 status'. The /requirements
  command also routes here. Not for writing application code; this shapes
  requirements only."
metadata:
  protected: "true"
---

# Requirements state machine: vision → approved BR → frozen technical spec

One continuous, resumable process. State lives in the record files, never in
memory: any session can pick a PCR back up by reading the files.

## Record layout

- `requirements/README.md` — registry: one row per PCR
  (`| PCR | Title | Status | Updated |`).
- `requirements/PCR-NNNN-<slug>/01-vision.md` — canonical record `status` in
  front matter + PART A (verbatim vision, immutable) + PART B (Clarification
  Log, append-only).
- `.../02-business.md` — the BR document: `draft → proposed → approved`,
  `revisions[]`, `approved_by/at`.
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

## 2. Read state before acting

Record status, business status, the last CL marker in PART B, and the unresolved
count in the Assumptions Ledger.

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
any pasted answers/revisions/vision text.

- `new` → `req-register`, then immediately `req-elicitation` (round 1); record
  status `elicitation`.
- `continue` / `answers` → `req-elicitation`; if it reports the BR was composed,
  set status `br_review`.
- `approve` → in `02-business.md` set `status: approved`, `approved_by`,
  `approved_at`; status `br_approved`.
- `feedback` → in `02-business.md` append to `revisions` `{at, by, note}`, set
  `status: proposed`; record status `br_review`; route content to
  `req-elicitation` only if BR wording needs rework.
- `technical` → `req-technical-spec`; status `technical`.
- `freeze` → `req-technical-spec`; on success status `frozen`.
- `status` → skip dispatch, report.

After every transition update the registry row (`status`, `updated: <today>`).

## 5. Report

Give the user: record path, what just happened, current state, and the next
legal action. Always end with the record path (e.g.
`requirements/PCR-0001-invoice-export/`) so the process resumes trivially later.

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
