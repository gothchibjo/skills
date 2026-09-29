---
name: req-plan
description:
  "Build the implementation plan for a frozen PCR strictly from its sealed
  03-technical.yaml (REQ-010): per-REQ task breakdown, topological order by
  dependencies, the evidence each REQ must produce, and the plan-approval gate
  (REQ-011) that human-approves the plan before any implementation starts. Phase
  of the req-develop state machine; load only via the Skill tool from
  req-develop."
disable-model-invocation: true
metadata:
  protected: "true"
  copyright: "© 2026 github.com/gothchibjo"
---

# Phase: req-plan

Called by req-develop with the event (`plan` | `plan_approve`) and the target
PCR.

## Inputs

Read `03-technical.yaml` (must be `status: frozen`) and any existing
`04-plan.md`.

## Event: plan — author `04-plan.md`

Front matter:

```yaml
---
pcr: PCR-NNNN
title: ...
status: planning
basis: 03-technical.yaml (frozen)
revised: <YYYY-MM-DD>
approved_by: null
approved_at: null
---
```

Body:

- **Plan invariants** — one explicit block: the contract is sealed (REQ-010).
  REQ titles/descriptions/acceptance criteria are NOT editable here; anything
  that looks like a scope deviation routes to change control (REQ-015 → new PCR
  via req-workflow), not to a plan edit.
- **Execution order** — topological: `dependencies` first, ties by REQ id; one
  section per REQ:
  - `REQ-<n> — <title>` (`priority`, `size_hint`)
  - task breakdown: concrete, checkable work items for the current tech stack
  - guard_conditions to honor (each is a pre-step check for req-guards)
  - evidence to produce (from `verification.evidence`) and the
    `verification.method`
- **Plan lifecycle** — a new `plan` event re-drafts the plan; keep prior draft
  revisions append-only under a `Revisions` block (never delete history).

Gate rule: the `plan` event alone does NOT flip `plan_approval` — implementation
is impossible until a human approves.

## Event: plan_approve — plan-approval gate (REQ-011)

Allowed only when:

- `03-technical.yaml` is `frozen`;
- `04-plan.md` exists and `status: planning`;
- the user explicitly approves the plan (event came from req-develop's
  `plan_approve`).

On success:

- `04-plan.md`: `status: plan_approved`, `approved_by: <who>`,
  `approved_at: <today>` (keep the approval and prior draft in `Revisions` too —
  history is append-only).
- `03-technical.yaml`: `pipeline_gates.plan_approval.passed: true`. Nothing else
  about the YAML changes.
- Report to the user: approved plan, the REQ order, and that implementation is
  now open (`implement`).

Refuse otherwise, naming the failing condition. Never flip `plan_approval`
without human approval, and never edit REQ texts while doing it.
