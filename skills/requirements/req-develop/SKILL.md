---
name: req-develop
description:
  "State machine for developing a frozen contract (03-technical.yaml, status:
  frozen) in the active repo: plan strictly from the frozen spec, require plan
  approval, guard every implementation step, and gate completion on verification
  evidence (Phase 2 development pipeline). The single entry point for starting
  development on a frozen PCR. Trigger when the user wants to start or continue
  development work on a frozen PCR or its REQs — e.g. 'start work on PCR-0001',
  'plan the development', 'write a plan', 'plan looks good', 'plan approved',
  'implement REQ-010', 'continue the implementation', 'REQ-013 is done',
  'evidence', 'close REQ', 'verify', 'all REQs closed', 'contract complete',
  'PCR-0001 status'. The /develop command also routes here. Not for requirements
  shaping; this drives development."
metadata:
  protected: "true"
---

# Development state machine: frozen spec → plan → guarded code → evidence

One continuous, resumable process. Runs only in a repo whose
`requirements/PCR-NNNN-<slug>/03-technical.yaml` is `frozen`. State lives in the
record files, never in memory; any session resumes by reading them.

## Record layout (Phase 2 additions)

- `requirements/README.md` — registry (unchanged: it tracks the requirements
  process; the development status does not rewrite its rows).
- `.../01-vision.md` — canonical requirements `status`; must be `frozen` before
  development starts.
- `.../03-technical.yaml` — the frozen contract: 15+ `requirements[]` and
  `pipeline_gates`. **Contract texts are sealed**: gate flags inside
  `pipeline_gates` are execution bookkeeping and may flip; REQ
  `title/description/acceptance_criteria/...` may NOT be edited (change control
  → REQ-015).
- `.../04-plan.md` — the development execution record. Front matter `status` is
  the canonical development state; body holds the plan, the guard log, and the
  evidence matrix.

## States (canonical, in `04-plan.md` front matter `status`)

`planning → plan_approved → implementing → verifying → done` (`blocked` is a
transient state set by req-guards on a guard violation and left via `resume`).

## 1. Resolve the target record

- Message names `PCR-<nnnn>` → that record.
- Message names `REQ-<n>` → the record whose frozen `03-technical.yaml` defines
  it.
- No id and exactly one frozen record → it. No id and several → list them, ask
  which.
- No frozen record at all → refuse: development needs a frozen contract (see
  req-workflow).

## 2. Read state before acting

`01-vision.md` status (must be `frozen`), `03-technical.yaml` (status,
`pipeline_gates`, the REQ list), and `04-plan.md` if present.

## 3. Classify the event (priority order)

| User phrase →                                   | Event        | Valid from                | Result                                                        |
| :---------------------------------------------- | :----------- | :------------------------ | :------------------------------------------------------------ |
| plan / plan the development / write a plan      | plan         | — (record must be frozen) | planning (04-plan.md written from the frozen spec)            |
| plan looks good / plan approved / I approve     | plan_approve | planning                  | plan_approved (`plan_approval` gate passed)                   |
| implement / start / continue / work on REQ-00x  | implement    | plan_approved, blocked    | implementing (guards pre-checked by req-guards)               |
| blockers cleared / conditions met / resume      | resume       | blocked                   | implementing (guards re-checked; stays blocked while failing) |
| REQ-00x is done / here is the evidence / verify | verify       | implementing              | verifying (req-verification builds the evidence matrix)       |
| all REQs closed / contract complete / finish    | complete     | verifying                 | done (`verification_evidence` gate passed)                    |
| status / what's next                            | status       | any                       | —                                                             |

Unrecognized message against a frozen record: infer the most consistent event;
if the transition is illegal, refuse and list the legal events — never silently
coerce. The table is enforced, not advisory.

Invariants: development never runs while `01-vision.md` is not `frozen`; it
never edits `03-technical.yaml` beyond the `pipeline_gates` flags; any deviation
from the frozen REQ texts is change control → a new PCR via req-workflow, not an
edit.

## 4. Dispatch and persist

Call the Skill tool with the phase name, passing context: record path, event,
any pasted evidence/plan text.

- `plan` → `req-plan`; on success status `planning`. If a plan already exists in
  `planning`, re-enter anyway (fresh plan draft).
- `plan_approve` → `req-plan`; it flips `plan_approval` and sets status
  `plan_approved` — only with human approval.
- `implement` → `req-guards`; it pre-checks guard conditions for the REQ to be
  worked on. If blocked, status `blocked`; else `implementing`.
- `resume` → `req-guards`; it re-checks guards and sets `implementing` again, or
  keeps `blocked`.
- `verify` → `req-verification`; it builds the evidence matrix and closes REQs.
- `complete` → `req-verification`; it flips `verification_evidence` and sets
  `done` only when every REQ has matching evidence.
- `status` → skip dispatch, report.

Development never writes to `01-vision.md` or `02-business.md`, and never
commits.

## 5. Report

Give the user: record path, development state, currently open REQs, gate status
(`plan_approval`, `guard_conditions`, `verification_evidence`), and the next
legal action. Always end with the record path (e.g.
`requirements/PCR-0001-invoice-export/`).

## Notes

- Commits are manual.
- History is append-only in `04-plan.md` (plan revisions, guard log, evidence
  matrix grow; nothing destructive).
