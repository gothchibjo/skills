# req-guards

Phase 3 of [req-develop]: the guard-conditions gate (REQ-012). Nothing gets
implemented until the REQ's preconditions actually hold.

**Not directly invocable.** The frontmatter sets
`disable-model-invocation: true` — loaded only through the state machine.

## What it does

Before any implementation step, it takes the REQ's `guard_conditions` from the
frozen YAML and checks each one against the repo **right now** — not against
what the plan claims, but against what is on disk.

Every check is appended to the Guard Log in `04-plan.md`:

```markdown
| date       | REQ     | guard                | check                       | result |
| ---------- | ------- | -------------------- | --------------------------- | ------ |
| 2026-03-14 | REQ-004 | migration applied    | `migrations/004.sql` exists | pass   |
| 2026-03-14 | REQ-004 | contract frozen      | `03-technical.yaml` frozen  | pass   |
```

None failing → the step is announced clean and `implementing` begins. Any
failing → status `blocked`, the step is refused, and every failing guard is
listed with the evidence of its check. **No gate is flipped in either case by
this skill alone** — `guard_conditions.passed` becomes true only once all guards
are clean, and is forced back to false whenever any guard fails.

`resume` re-runs the failed checks: still failing keeps `blocked`, clean returns
to `implementing`.

## Design decisions worth knowing

- **The plan gate comes first.** REQ-011 before REQ-012: an unapproved plan is
  refused before any guard is even looked at.
- **Checks are evidence, not assertions.** The log records what was checked and
  the result, so a later audit can see why a step was allowed.
- **Blocking is the normal outcome, not an error.** A guard that is not
  satisfied is information about the repo, and the refusal names it.
- **Skipping a guard is a scope change.** Any need to alter a frozen REQ text or
  bypass a guard "to get going" is refused: no YAML edit, no plan edit, routed
  to [req-workflow] to register a change-request PCR (REQ-015).

## Install

Not installed on its own; it ships with the development pipeline.

```bash
npx skills add gothchibjo/skills -g -a opencode
```

See [docs/installation.md] for the full matrix.

## Troubleshooting

| Symptom                           | Fix                                                                     |
| :-------------------------------- | :---------------------------------------------------------------------- |
| Stuck `blocked` after a fix       | Run `resume` — guards are re-checked, not remembered.                   |
| Refused before checking any guard | The plan is not `plan_approved`. Approve it via [req-plan] first.       |
| A guard you believe is satisfied  | The check failed. Read the row: it records what was actually looked at. |
| It refuses to skip a guard        | By design — that is change control, not an obstacle. Open a PCR.        |

## License

MIT. See [LICENSE].

<!-- refs -->

[req-develop]: ../req-develop/
[req-workflow]: ../req-workflow/
[docs/installation.md]: ../../../docs/installation.md
[req-plan]: ../req-plan/
[LICENSE]: ../../../LICENSE
