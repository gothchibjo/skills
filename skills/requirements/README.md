# The requirements pipeline

Eight skills that are one process in two phases. They do not work apart — the
entry point dispatches the phases, and the phases read what the entry point
wrote. Install the family.

> The `requirements/` folder in this repository is just a container for these
> skills. It is unrelated to the `requirements/` folder the skills create in
> _your_ project.

## The two phases

| Phase            | Entry point                      | Takes you from                          |
| ---------------- | -------------------------------- | --------------------------------------- |
| 1 — requirements | [`req-workflow`] `/requirements` | an incoming vision to a frozen contract |
| 2 — development  | [`req-develop`] `/develop`       | a frozen contract to verified code      |

```
vision ──▶ registered ──▶ elicitation ──▶ br_review ──▶ br_approved ──▶ technical ──▶ frozen
                                                                                          │
   04-plan.md ◀── plan_approved ◀── plan ──▶ implementing ◀── guards ◀── implementing
                    │                              │
                    └── verifying ──▶ done ◀─── verification ──▶ verifying
```

## Which skill dispatches which

The six `phase` skills carry `disable-model-invocation: true`. The agent will
never reach for them on its own; only the entry point loads them.

| Phase skill            | Dispatched by  | On event                     |
| ---------------------- | -------------- | ---------------------------- |
| [`req-register`]       | `req-workflow` | `new`                        |
| [`req-elicitation`]    | `req-workflow` | `new`, `continue`, `answers` |
| [`req-technical-spec`] | `req-workflow` | `technical`, `freeze`        |
| [`req-plan`]           | `req-develop`  | `plan`, `plan_approve`       |
| [`req-guards`]         | `req-develop`  | `implement`, `resume`        |
| [`req-verification`]   | `req-develop`  | `verify`, `complete`         |

[`req-elicitation`] runs the [`grilling`] interview, so that skill is a
dependency of the family.

## The record

State lives in the record files, never in the agent's memory, so any session
resumes a PCR by reading the folder. The skills expect this to be a normal,
committed part of your project — nothing is global state.

```
requirements/
  README.md                          registry: one row per PCR
  PCR-NNNN-<slug>/
    01-vision.md                     PART A verbatim vision (immutable)
                                     PART B clarification log (append-only)
    02-business.md                   business requirements: draft → proposed → approved
    03-technical.yaml                machine-readable REQs, status: frozen once sealed
    04-plan.md                       plan, guard log, evidence matrix
```

## The four gates

Each gate is flipped by the skill that owns it, never by hand, and only in this
order:

| Gate                    | Flipped by           | Means                                     |
| ----------------------- | -------------------- | ----------------------------------------- |
| `requirement_freeze`    | `req-technical-spec` | BR approved, ledger resolved, spec sealed |
| `plan_approval`         | `req-plan`           | a human approved the plan                 |
| `guard_conditions`      | `req-guards`         | every guard currently checks out          |
| `verification_evidence` | `req-verification`   | every REQ carries matching evidence       |

A frozen contract's REQ texts are sealed. Changing scope mid-flight is a new PCR
routed back through `req-workflow`, not an edit to the YAML.

## Install

The whole repository, or the family alone:

```bash
# everything
npx skills add gothchibjo/skills -g -a opencode

# just this family
npx skills add gothchibjo/skills -g -a opencode \
  --skill req-workflow --skill req-register --skill req-elicitation \
  --skill req-technical-spec --skill req-develop --skill req-plan \
  --skill req-guards --skill req-verification
```

The `/requirements` and `/develop` command wrappers are not installed by the CLI
— copy [`requirements.md`] and [`develop.md`] into your agent's `command/`
directory by hand. See [docs/installation.md] for the full matrix.

## Conventions

- **`disable-model-invocation: true`** on the six phase skills. This is what
  keeps the entry points the only way in.
- **`metadata: protected: "true"`** on every skill in the family. It marks them
  as off-limits to maintenance tooling: no archiving, no rewriting, no pruning
  them out of an install.

## License

MIT. See [LICENSE].

<!-- refs -->

[`req-workflow`]: req-workflow/
[`req-develop`]: req-develop/
[`req-register`]: req-register/
[`req-elicitation`]: req-elicitation/
[`req-technical-spec`]: req-technical-spec/
[`req-plan`]: req-plan/
[`req-guards`]: req-guards/
[`req-verification`]: req-verification/
[`grilling`]: ../grilling/
[`requirements.md`]: ../../command/requirements.md
[`develop.md`]: ../../command/develop.md
[docs/installation.md]: ../../docs/installation.md
[LICENSE]: ../../LICENSE
