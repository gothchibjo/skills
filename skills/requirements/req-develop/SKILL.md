---
name: req-develop
description:
  "State machine for developing a frozen contract (03-technical.yaml, status:
  frozen): plan from the frozen spec, require plan approval, guard each
  implementation step, gate completion on verification evidence. The single entry
  point for Phase 2 — /develop routes here, as do 'write a plan', 'plan
  approved', 'implement REQ-010', 'REQ-013 is done', 'all REQs closed'. Not for
  requirements shaping."
metadata:
  protected: "true"
  copyright: "© 2026 github.com/gothchibjo"
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
- `.../03-technical.yaml` — the frozen contract: 15+ `requirements[]`,
  `pipeline_gates`, and an optional `glossary` block of load-bearing term
  definitions. **Contract texts are sealed** — including the glossary
  definitions: gate flags inside `pipeline_gates` are execution bookkeeping and
  may flip; REQ `title/description/acceptance_criteria/...` and glossary
  definitions may NOT be edited (change control → REQ-015). When the block is
  present, it outranks `requirements/glossary.md`, which may have moved on.
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

## 2. Read state before acting — metadata only

Read `01-vision.md` `status`, `03-technical.yaml` front matter and
`pipeline_gates`, and `04-plan.md` front matter. Get the open REQ ids by
grepping the evidence matrix rather than reading the plan. The REQ bodies belong
to the phase that works on that REQ.

## 3. Classify the event (priority order)

| User phrase →                                       | Event        | Valid from                | Result                                                        |
| :-------------------------------------------------- | :----------- | :------------------------ | :------------------------------------------------------------ |
| plan / plan the development / write a plan          | plan         | — (record must be frozen) | planning (04-plan.md written from the frozen spec)            |
| plan looks good / plan approved / I approve         | plan_approve | planning                  | plan_approved (`plan_approval` gate passed)                   |
| implement / start / continue / work on REQ-00x (…)  | implement    | plan_approved, blocked    | implementing (guards pre-checked by req-guards)               |
| blockers cleared / conditions met / resume          | resume       | blocked                   | implementing (guards re-checked; stays blocked while failing) |
| REQ-00x is done / here is the evidence / verify (…) | verify       | implementing              | verifying (req-verification builds the evidence matrix)       |
| all REQs closed / contract complete / finish        | complete     | verifying                 | done (`verification_evidence` gate passed)                    |
| status / what's next                                | status       | any                       | —                                                             |

Unrecognized message against a frozen record: infer the most consistent event;
if the transition is illegal, refuse and list the legal events — never silently
coerce. The table is enforced, not advisory.

A message naming several REQs is one event with several REQs, not a loop of
events: the phase checks and records them in one pass over the contract.

Invariants: development never runs while `01-vision.md` is not `frozen`; it
never edits `03-technical.yaml` beyond the `pipeline_gates` flags; any deviation
from the frozen REQ texts is change control → a new PCR via req-workflow, not an
edit.

## 4. Dispatch and persist

Call the Skill tool with the phase name, passing context: record path, event,
the REQ ids in scope, any pasted evidence/plan text. Do not restate the contract
in the dispatch — the phase reads the REQ blocks it needs.

- `plan` → `req-plan`; on success status `planning`. If a plan already exists in
  `planning`, re-enter anyway (fresh plan draft).
- `plan_approve` → `req-plan`; it flips `plan_approval` and sets status
  `plan_approved` — only with human approval.
- `implement` → `req-guards`; it pre-checks guard conditions for the REQs to be
  worked on. If blocked, status `blocked`; else `implementing`.
- `resume` → `req-guards`; it re-checks guards and sets `implementing` again, or
  keeps `blocked`.
- `verify` → `req-verification`; it builds the evidence matrix and closes REQs.
- `complete` → `req-verification`; it flips `verification_evidence` and sets
  `done` only when every REQ has matching evidence.
- `status` → skip dispatch, report from §2 metadata and stop.

Development never writes to `01-vision.md` or `02-business.md`, and never
commits.

## 5. Report — once per event

The phase returns a compact result; you give the user: record path, development
state, the REQs in scope with their status, gate status (`plan_approval`,
`guard_conditions`, `verification_evidence`), and the next legal action. Always
end with the record path (e.g. `requirements/PCR-0001-invoice-export/`).

## 6. Cost discipline

The same rules as Phase 1, and the reason Phase 2 is where token cost explodes
if they are ignored:

- **P1 Sliced reads.** One `implement REQ-010` reads the REQ-010 block and its
  guards, the plan section for it, and the guard log — not the whole contract
  and not the whole plan. Read `04-plan.md` front matter plus the named section.
- **P2 Edit, never rewrite.** Guard-log and matrix rows are appended. Gate flags
  are edited in place. Never regenerate `03-technical.yaml` or `04-plan.md`.
- **P3 No echo.** Do not print what you wrote, and do not re-read the plan to
  confirm a row landed — `git diff --stat` or a grep for the row is enough.
- **P4 One report.** Phases return `outcome`, changed paths and blockers; only
  this skill writes to the user.
- **P5 Batch.** Several REQs in one event share one pass over the contract.
- **P6 Facts, cheaply.** grep before subagent; a subagent answers in a short
  digest and is never a way to read a file whose path is known.

## Notes

- Commits are manual.
- History is append-only in `04-plan.md` (plan revisions, guard log, evidence
  matrix grow; nothing destructive).
