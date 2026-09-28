# req-plan

Phase 2 of [`req-develop`]: write the execution plan from the frozen contract,
and hold the human plan-approval gate (REQ-011).

**Not directly invocable.** The frontmatter sets
`disable-model-invocation: true` — loaded only through the state machine.

## What it does

**`plan` — author `04-plan.md`.** Strictly from the frozen `03-technical.yaml`;
the REQ order is topological (`dependencies` first, ties by REQ id). One section
per REQ carrying its `priority` and `size_hint`, a concrete task breakdown for
the current stack, the `guard_conditions` to honor, and the
`verification.evidence` it must produce with its method.

The file opens with a **Plan invariants** block stating plainly that the
contract is sealed and that REQ titles, descriptions and acceptance criteria are
not editable from here.

**`plan_approve` — the gate.** Requires a frozen spec, a `04-plan.md` in
`planning`, and your explicit approval. On success it sets the file to
`plan_approved` with `approved_by` / `approved_at` and flips
`pipeline_gates.plan_approval.passed: true`. Nothing else in the YAML changes.

## Design decisions worth knowing

- **Writing the plan is not approving it.** The `plan` event alone never flips
  `plan_approval` — implementation stays impossible until a human says so. This
  is the single most important property of the gate.
- **The plan cannot bend the contract.** A task that needs a different REQ text
  is a scope deviation: it is refused here and routed to [`req-workflow`] as
  change control (REQ-015), not absorbed into the plan.
- **Re-planning is allowed and additive.** A second `plan` event drafts afresh;
  prior drafts are kept under a `Revisions` block. History is append-only.
- **Guard conditions are named per REQ** so [`req-guards`] can check them
  without re-deriving intent from the plan prose.

## Install

Not installed on its own; it ships with the development pipeline.

```bash
npx skills add gothchibjo/skills -g -a opencode
```

See [docs/installation.md] for the full matrix.

## Troubleshooting

| Symptom                              | Fix                                                                       |
| ------------------------------------ | ------------------------------------------------------------------------- |
| `plan_approve` refused               | The spec is not frozen, or the plan is not in `planning`. Both are named. |
| It refuses to plan a task you expect | The task needs a contract change. Open a change-request PCR instead.      |
| Plan order looks wrong               | It is topological on `dependencies`; check the YAML for a missing edge.   |
| Revisions piling up in the file      | Expected — every re-plan appends. Nothing is ever deleted.                |

## License

MIT. See [LICENSE].

<!-- refs -->

[`req-develop`]: ../req-develop/
[`req-workflow`]: ../req-workflow/
[`req-guards`]: ../req-guards/
[docs/installation.md]: ../../../docs/installation.md
[LICENSE]: ../../../LICENSE
