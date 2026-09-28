# req-develop

Entry point for Phase 2: turning a frozen contract into implemented, verified
code. One state machine, resumable across sessions.

**User-invoked** through the `/develop` command; also model-invoked when you ask
to start or continue work on a frozen PCR.

## What it does

Runs only where a contract is already frozen
(`requirements/PCR-NNNN-<slug>/03-technical.yaml` with `status: frozen`). The
development state lives in `04-plan.md`, not in the agent's memory.

```text
planning → plan_approved → implementing → verifying → done
```

`blocked` is a transient state: set by [req-guards] on a guard violation, left
via `resume`.

## Events

| Say                      | Event          | Valid from                 | Effect                                 |
| :----------------------- | :------------- | :------------------------- | :------------------------------------- |
| plan it, make a plan     | `plan`         | record must be frozen      | `04-plan.md` written from the contract |
| plan approved            | `plan_approve` | `planning`                 | plan-approval gate passes              |
| implement, continue      | `implement`    | `plan_approved`, `blocked` | guards pre-checked, then code          |
| blockers cleared, resume | `resume`       | `blocked`                  | guards re-checked                      |
| REQ-00x is done, verify  | `verify`       | `implementing`             | evidence matrix built, REQs closed     |
| all REQs closed          | `complete`     | `verifying`                | verification gate passes, `done`       |
| status                   | `status`       | any                        | report only                            |

## Design decisions worth knowing

- **The contract is sealed.** REQ titles, descriptions and acceptance criteria
  cannot be edited during development. The `pipeline_gates` flags inside the
  YAML are bookkeeping and may flip; nothing else in it changes.
- **A scope deviation is a new PCR, not an edit.** Anything that looks like
  changing a frozen REQ is refused and routed to [req-workflow] as change
  control.
- **Four gates, in order.** `requirement_freeze` → `plan_approval` →
  `guard_conditions` → `verification_evidence`. Each is flipped by the phase
  that owns it, never by hand.
- **It never writes `01-vision.md` or `02-business.md`,** and never commits.
  Requirements are upstream of development; touching them here would break the
  separation the whole pipeline depends on.
- **It always reports the record path** last, so resuming is a copy-paste.

## Scope

This drives development against a frozen contract. It is not for shaping
requirements — that is [req-workflow]. It refuses to start when no frozen record
exists.

## Install

```bash
# the whole pipeline
npx skills add gothchibjo/skills -g -a opencode

# or this entry point plus the phases it dispatches
npx skills add gothchibjo/skills --skill req-develop --skill req-plan \
  --skill req-guards --skill req-verification -g -a opencode
```

The `/develop` command wrapper is not installed by the CLI — copy
[command/develop.md] by hand. See [docs/installation.md] for the full matrix.

## Troubleshooting

| Symptom                             | Fix                                                                                    |
| :---------------------------------- | :------------------------------------------------------------------------------------- |
| "Development needs a frozen record" | No `03-technical.yaml` is `frozen`. Finish the requirements phase first.               |
| `implement` refused on a plan       | The plan is not approved yet. `plan_approve` needs a human — it will not self-approve. |
| Stuck in `blocked`                  | Read the Guard Log in `04-plan.md`; the failing guard names its own check.             |
| `complete` refused                  | Some REQ has no evidence row, or a `waived` row without a recorded reason.             |

## License

MIT. See [LICENSE].

<!-- refs -->

[req-guards]: ../req-guards/
[req-workflow]: ../req-workflow/
[command/develop.md]: ../../../command/develop.md
[docs/installation.md]: ../../../docs/installation.md
[LICENSE]: ../../../LICENSE
