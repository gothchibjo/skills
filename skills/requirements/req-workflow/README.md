# req-workflow

Entry point for the requirements process: an incoming customer vision becomes a
frozen, machine-readable technical contract. One state machine, resumable across
sessions.

**User-invoked** through the `/requirements` command; also model-invoked when
you talk about requirements, PCRs or a stakeholder request.

## What it does

The process is a sequence of states, and the state lives in the record files
rather than in the agent's memory — so any session picks a PCR back up by
reading them.

```text
registered → elicitation → br_review → br_approved → technical → frozen
```

It resolves which record you mean, reads its current state, classifies your
message into an event, and dispatches the matching phase skill.

| Record file         | Holds                                                                       |
| :------------------ | :-------------------------------------------------------------------------- |
| `01-vision.md`      | PART A verbatim vision (immutable) + PART B Clarification Log (append-only) |
| `02-business.md`    | The BR document: `draft → proposed → approved`, with `revisions[]`          |
| `03-technical.yaml` | Machine-readable REQs and the four pipeline gates                           |
| `04-plan.md`        | Phase 2: the development record (written by [req-develop])                  |

## Events

The event table is **enforced, not advisory**. An illegal transition is refused
with the list of legal events, never silently coerced.

| Say                        | Event       | Valid from                        |
| :------------------------- | :---------- | :-------------------------------- |
| new request, register this | `new`       | —                                 |
| let's continue, resume     | `continue`  | elicitation, br_review, technical |
| here are the answers       | `answers`   | elicitation                       |
| requirements approved      | `approve`   | br_review, technical              |
| changes, rework this       | `feedback`  | br_review, br_approved, technical |
| technical requirements     | `technical` | br_approved                       |
| freeze it                  | `freeze`    | technical                         |
| status                     | `status`    | any                               |

## Design decisions worth knowing

- **The vision is never rewritten.** PART A is stored as received, defects and
  contradictions included. Artifacts we author go into the record's
  `artifact_language`; raw customer text stays in its source language.
- **History is append-only.** PART B and `revisions[]` only grow. A correction
  is a new revision entry, never a destructive edit.
- **The registry is kept in step.** After every transition the `requirements/`
  registry row is updated with the new status and date.
- **Freezing has two conditions**, not one: the BR must be `approved` _and_ the
  Assumptions Ledger must have no unresolved items. Both are re-checked inside
  the phase skill.
- **It writes files, it never commits.** Version control stays a human decision.
- **It always reports the record path** as the last thing, so resuming later is
  a copy-paste.

## Scope

This shapes requirements; it does not write application code. Once a PCR is
frozen, development runs through [req-develop], and a scope change on a frozen
PCR comes back here as a new PCR — change control, not an edit.

## Install

```bash
# the whole pipeline
npx skills add gothchibjo/skills -g -a opencode

# or this entry point plus the phases it dispatches
npx skills add gothchibjo/skills --skill req-workflow --skill req-register \
  --skill req-elicitation --skill req-technical-spec -g -a opencode
```

The `/requirements` command wrapper is not installed by the CLI — copy
[command/requirements.md] by hand. See [docs/installation.md] for the full
matrix.

## Troubleshooting

| Symptom                         | Fix                                                                                        |
| :------------------------------ | :----------------------------------------------------------------------------------------- |
| It asks which PCR you mean      | Several records are open and your message named none. Name the id.                         |
| "Illegal transition" on approve | The BR is not at `proposed` yet, or is already `approved`. Check `02-business.md`.         |
| Freeze refused                  | BR not approved, or the Assumptions Ledger has `(unresolved)` items. Both are named.       |
| A phase skill seems unavailable | Correct: the phases are `disable-model-invocation` and are only loaded through this skill. |

## License

MIT. See [LICENSE].

<!-- refs -->

[req-develop]: ../req-develop/
[command/requirements.md]: ../../../command/requirements.md
[docs/installation.md]: ../../../docs/installation.md
[LICENSE]: ../../../LICENSE
