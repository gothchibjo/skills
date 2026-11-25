---
name: req-plan
description:
  "Build the implementation plan for a frozen PCR from its sealed
  03-technical.yaml: per-REQ task breakdown in dependency order, the evidence each
  REQ must produce, and the Plan Approval gate. Phase of req-develop; load only
  via the Skill tool from req-develop."
disable-model-invocation: true
metadata:
  protected: "true"
  copyright: "© 2026 github.com/gothchibjo"
---

# Phase: req-plan

Called by req-develop with the event (`plan` | `plan_approve`) and the target
PCR. Cost rules P1–P6 from req-develop apply.

## Inputs

- `plan`: read the frozen `03-technical.yaml` in full — planning covers every
  REQ, so this is the one event that legitimately needs the whole contract. Its
  `glossary` block, when present, is the authoritative vocabulary: use those
  definitions rather than `requirements/glossary.md`, which may have moved on
  since the freeze. Do not read `01-vision.md` or `02-business.md`.
- `plan_approve`: read `04-plan.md` front matter and confirm the file exists
  with `status: planning`. Do not re-read the plan body.

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
  revisions append-only under a `Revisions` block (never delete history). Append
  the revision; do not rewrite the rest of the document.

Gate rule: the `plan` event alone does NOT flip `plan_approval` — implementation
is impossible until a human approves.

## Event: plan_approve — plan-approval gate (REQ-011)

Allowed only when:

- `03-technical.yaml` is `frozen`;
- `04-plan.md` exists and `status: planning`;
- the user explicitly approves the plan (event came from req-develop's
  `plan_approve`).

On success:

- `04-plan.md`: edit front matter to `status: plan_approved`,
  `approved_by: <who>`, `approved_at: <today>`, and append the approval to
  `Revisions` — history is append-only, the plan body is untouched.
- `03-technical.yaml`: `pipeline_gates.plan_approval.passed: true`. Nothing else
  about the YAML changes — one line, edited in place.
- Report to the user: approved plan, the REQ order (ids only), and that
  implementation is now open (`implement`).

Refuse otherwise, naming the failing condition. Never flip `plan_approval`
without human approval, and never edit REQ texts while doing it.
