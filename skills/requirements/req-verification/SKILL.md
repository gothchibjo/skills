---
name: req-verification
description:
  "Verification-evidence gate (REQ-013) and traceability (REQ-014) of the
  frozen-contract pipeline: close each REQ only with evidence that matches its
  declared verification.method, build the REQ↔evidence matrix and the
  BR→REQ→evidence chain in 04-plan.md, document every gap (never hide it), and
  flip the gate only when all REQs carry matching evidence. Phase of the
  req-develop state machine; load only via the Skill tool from req-develop."
disable-model-invocation: true
metadata:
  protected: "true"
  copyright: "© 2026 github.com/gothchibjo"
---

# Phase: req-verification

Called by req-develop with the event (`verify` | `complete`) and the target PCR.

## Inputs

Read `03-technical.yaml` (frozen), `04-plan.md` (plan, guard log, evidence
matrix), and the finished implementation work in the repo.

## Event: verify — record evidence per REQ

For each REQ that the user claims done, map the concrete artifact(s) that prove
it against `verification.evidence` and `verification.method` from the frozen
YAML:

- `test` → runnable check / test report
- `inspection` → reviewed file paths + review sign-off
- `manual-review` → review verdict / traceability matrix
- `simulation` → run artifact / logs

Update the Evidence Matrix in `04-plan.md` (append-only rows):

```markdown
| REQ | verification.method | evidence artifact | status (open/done/waived) |
```

Rules:

- A REQ is `done` only when its evidence artifact is present and consistent with
  the declared method. Anything else stays `open`.
- A `waived` REQ requires a documented reason in the matrix — a waiver is an
  exception, recorded and visible, never silence.
- A REQ with no plan evidence and no waived entry cannot be claimed complete.

## Event: complete — close the gate

Allowed only when every REQ in `03-technical.yaml` has a `done` (or explicitly
`waived`) matrix row. On success:

- `03-technical.yaml`: `pipeline_gates.verification_evidence.passed: true`.
- `04-plan.md`: `status: done`; append the traceability chain report: for every
  REQ show `BR-<n> → REQ-<n> → evidence artifact` (REQ-014). Every gap in the
  chain is listed explicitly — never hidden.
- Report to the user: gate status summary, the traceability chain, and that the
  contract is fully executed (`plan_approval`, `guard_conditions`,
  `verification_evidence`, and `requirement_freeze` all passed).

Refuse otherwise, naming the REQs still lacking evidence. Never flip the gate
while any REQ is open, and never remedy a gap by editing a frozen REQ (REQ-015 →
change control via req-workflow).
