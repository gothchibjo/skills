---
name: req-technical-spec
description:
  "Generate the machine-readable 03-technical.yaml from a PCR's business
  requirements: deterministic BR→REQ mapping, coverage report, glossary
  validation, YAML validation, and the Requirement Freeze gate. Phase of
  req-workflow; load only via the Skill tool from req-workflow."
disable-model-invocation: true
metadata:
  protected: "true"
  copyright: "© 2026 github.com/gothchibjo"
---

# Phase: req-technical-spec

Turns a PCR's business requirements into the machine-ready `03-technical.yaml`.
Called by req-workflow with the event (`technical` | `freeze`) and the target
PCR. Cost rules R1–R6 from req-workflow apply.

## Inputs

- `technical`: read `02-business.md` in full — it is the single source of the
  mapping, and this is the one event that legitimately needs all of it. Do not
  read `01-vision.md`: sources already reach you through the BRs' `traces to`
  lines.
- `freeze`: read front matter of `03-technical.yaml` and `02-business.md`, the
  YAML `glossary` block (it is small), then grep the ledger for `(unresolved)`
  and the terms slice for the disputed term. Do not re-read the generated REQ
  set.
- Glossary: grep the canon named by the registry header (`glossary:`, default
  `requirements/glossary.md`, plus read-only `glossary_ref:`) for each term in
  play. Never read it whole.

## Event: technical — generate

Map every business requirement to REQ entries.

### Permissions by BR status

- `approved` → full generation (all fields below).
- `proposed` → structural draft only:
  `id, traces_to, type, title, description, priority, acceptance_criteria`; omit
  `verification`, `guard_conditions`, `size_hint`, `glossary`. Report that a
  full regeneration is required after approval.
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

### Glossary

- Every term that changes how a REQ is implemented — in its title, description,
  acceptance criteria, guards, or dependencies — must be defined in the PCR's
  Terms Used in This PCR slice. A term used but not defined is a coverage hole:
  report it and ask; never invent a definition.
- A definition that contradicts the canon stops generation: report the conflict,
  ask the question, and write no YAML. This is a deliberate exception to
  "warnings, not blocks" — a REQ built on a wrong definition passes review while
  being wrong.
- Emit the load-bearing terms into the optional top-level `glossary` block,
  copying definitions from `02-business.md` without re-wording:

  ```yaml
  glossary:
    - id: G-001
      term: заказ
      definition: <copied from 02-business.md>
  ```

  Omit the block when no term is load-bearing.

- A conflict already recorded as `(unresolved)` in the Assumptions Ledger is a
  freeze blocker; do not duplicate it as a guard condition.

### Write and verify

- Write the YAML once. If it already exists and this is a regeneration, replace
  the file in one write — partial patching across a changed BR set is what
  produces drift.
- Validate without emitting the file:
  `python3 -c "import yaml; yaml.safe_load(open('<path>'))"` must exit 0 and
  print nothing. Never `cat` the result, never re-read it to "check" — a parse
  error is the only signal you need.
- Coverage report to the user: BR→REQ mapping as a two-column table of ids, plus
  orphans, non-verifiable acceptance criteria, unresolved assumptions carried as
  guards, glossary terms included and any open conflict. Ids and counts only —
  no restating descriptions.
- Do not modify `01-vision.md` or `02-business.md`.

## Event: freeze — seal

Allowed only when all three hold:

- BR status = `approved`;
- the Assumptions Ledger has no `(unresolved)` items;
- the terms slice has no open conflict with the canon.

Otherwise refuse and name the failing gate (approve the BR, resolve the ledger,
or resolve the term conflict). Before sealing, check that every `glossary` entry
in the YAML matches the definition in `02-business.md`; a mismatch is drift —
refuse and regenerate. On success:

- Edit the YAML in place: `status: frozen`, `frozen_by: <who>`,
  `frozen_at: <today>`, `pipeline_gates.requirement_freeze.passed: true`. Change
  those keys only.
- A frozen spec is sealed: later `technical` events are refused — scope change
  at this point requires a new PCR (change control, Phase 2).
- Report `frozen` back to req-workflow.
