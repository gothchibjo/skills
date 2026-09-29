---
name: req-technical-spec
description:
  "Generate the machine-readable technical requirements
  (requirements/PCR-NNNN-<slug>/03-technical.yaml) from the PCR's approved (or
  proposed, as structural draft) business requirements: deterministic BR→REQ
  mapping, coverage/traceability report with warnings (not blocks), YAML
  validation, and the freeze action that seals the spec (Requirement Freeze
  gate) once the BR is approved and the Assumptions Ledger is free of unresolved
  items. Phase of the req-workflow state machine; load only via the Skill tool
  from req-workflow."
disable-model-invocation: true
metadata:
  protected: "true"
  copyright: "© 2026 github.com/gothchibjo"
---

# Phase: req-technical-spec

Turns a PCR's business requirements into the machine-ready `03-technical.yaml`.
Called by req-workflow with the event (`technical` | `freeze`) and the target
PCR.

## Inputs

Read `01-vision.md` (V + CL), `02-business.md` (BR document + status), and the
existing `03-technical.yaml` if any.

## Event: technical — generate

Map every business requirement to REQ entries.

### Permissions by BR status

- `approved` → full generation (all fields below).
- `proposed` → structural draft only:
  `id, traces_to, type, title, description, priority, acceptance_criteria`; omit
  `verification`, `guard_conditions`, `size_hint`. Report that a full
  regeneration is required after approval.
- `draft` → do not generate; report that the BR document must reach at least
  `proposed` first.

### Mapping rules

- Deterministic: same BR document ⇒ same YAML. `id`: `REQ-001…`, ordered
  topologically (dependencies first, ties by BR order).
- Every `BR-<n>` maps to ≥1 REQ; every REQ `traces_to` ≥1 BR. A BR without a
  REQ, or a REQ without a BR, is a coverage hole — report it, never drop it
  silently.
- `type` from the BR nature: `functional` (a behavior), `non-functional`
  (performance/security/scalability/usability), `business-rule` (a rule/policy),
  `constraint` (a fixed boundary).
- `priority` = BR priority (`must | should | could`).
- `title`: short. `description`: one unambiguous sentence, EARS-flavored ("WHEN
  <trigger>, THE SYSTEM SHALL <outcome>"), no negation tricks or weasel words.
- `acceptance_criteria` (list): derived from the BR business acceptance
  criteria, made concrete and checkable. If a criterion cannot name any way to
  check it, include it anyway and flag it "not verifiable" in the report.
- `dependencies` (list): REQ ids from the BR's `depends on` plus conservative
  intra-REQ data/order dependencies.
- `verification.method`: `test | manual-review | inspection | simulation`;
  `verification.evidence` (list): named artifacts that prove completion (e.g.
  `test report`, `traceability matrix`, `review sign-off`) — generic, since the
  tech stack is undecided at this stage.
- `guard_conditions` (list): preconditions/invariants that must hold for the REQ
  to be worked on and for completion to be valid. Seed from constraints; every
  `(unresolved)` assumption in the Assumptions Ledger becomes a guard condition
  referencing it.
- `size_hint`: `xs | s | m | l` heuristic from description breadth and
  dependency count.
- `pipeline_gates.plan_approval` / `verification_evidence` / `guard_conditions`:
  leave `passed: false` here — Phase 2 (development pipeline) flips them.

### Verify and report

- Write the YAML, then re-read and parse it
  (`python3 -c "import yaml,sys; yaml.safe_load(open('<path>'))"` or `yq`) — fix
  any parse error before reporting success.
- Coverage report to the user: BR→REQ mapping table, orphans, non-verifiable
  acceptance criteria, unresolved assumptions carried as guards.
- Do not modify `01-vision.md` or `02-business.md`.

## Event: freeze — seal

Allowed only when BR status = `approved` AND the Assumptions Ledger has no
`(unresolved)` items. Otherwise refuse and name the failing gate (approve the
BR, or resolve the ledger). On success:

- Set YAML `status: frozen`, `frozen_by: <who>`, `frozen_at: <today>`,
  `pipeline_gates.requirement_freeze.passed: true`.
- A frozen spec is sealed: later `technical` events are refused — scope change
  at this point requires a new PCR (change control, Phase 2).
- Report `frozen` back to req-workflow.
