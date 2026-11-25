---
name: req-verification
description:
  "Verification-evidence gate (REQ-013) and traceability (REQ-014): close a REQ
  only with evidence matching its declared verification.method, keep the
  REQ↔evidence matrix, document every gap, and flip the gate only when all REQs
  are covered. Phase of req-develop; load only via the Skill tool from
  req-develop."
disable-model-invocation: true
metadata:
  protected: "true"
  copyright: "© 2026 github.com/gothchibjo"
---

# Phase: req-verification

Called by req-develop with the event (`verify` | `complete`) and the target PCR
plus the REQ ids in scope. The cost discipline from req-develop applies.

## Inputs

Read only what the event needs:

- `verify`: the REQ blocks of the REQs in scope (id, `verification`) by grep;
  the `## Evidence Matrix` rows for those REQs; and the implementation artifacts
  they name. Not the plan body, not the rest of the contract.
- `complete`: the full REQ id list (`grep 'id: REQ-' 03-technical.yaml`), each
  REQ's `traces_to` (for the chain), the `## Evidence Matrix`, and
  `pipeline_gates`. A REQ id with no `done`/`waived` row is the only thing you
  need to know about its body.

## Event: verify — record evidence per REQ

For each REQ that the user claims done, map the concrete artifact(s) that prove
it against `verification.evidence` and `verification.method` from the frozen
YAML:

- `test` → runnable check / test report
- `inspection` → reviewed file paths + review sign-off
- `manual-review` → review verdict / traceability matrix
- `simulation` → run artifact / logs

Append rows to the `## Evidence Matrix` table in `04-plan.md` (append-only), and
set `04-plan.md` `status: verifying` — it is the event that enters that state.

```markdown
| REQ | verification.method | evidence artifact | status (open/done/waived) | reason (waived only) |
```

Rules:

- A REQ is `done` only when its evidence artifact is present and consistent with
  the declared method. Anything else stays `open`.
- A `waived` REQ requires a documented reason in the `reason` column — a waiver
  is an exception, recorded and visible, never silence.
- A REQ with no plan evidence and no waived entry cannot be claimed complete.

## Event: complete — close the gate

Allowed only when every REQ in `03-technical.yaml` has a `done` (or explicitly
`waived`) matrix row **and** all four `pipeline_gates` already read
`passed: true`. Refuse, naming the failing condition, if any gate is not true.
On success:

- Edit `03-technical.yaml` in place:
  `pipeline_gates.verification_evidence.passed: true`. One key.
- `04-plan.md`: `status: done`; append the traceability chain report: for every
  REQ show `BR-<n> → REQ-<n> → evidence artifact` (REQ-014). One line per REQ,
  ids and artifact names only. Every gap in the chain is listed explicitly —
  never hidden.
- Return to req-develop: the gate status read from the YAML, the traceability
  chain, and that the contract is fully executed (`plan_approval`,
  `guard_conditions`, `verification_evidence`, and `requirement_freeze` all
  passed).

Refuse otherwise, naming the REQs still lacking evidence or the gate that is not
true. Never flip the gate while any REQ is open, and never remedy a gap by
editing a frozen REQ (REQ-015 → change control via req-workflow).
