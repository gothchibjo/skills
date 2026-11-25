# The requirements pipeline

Eight skills that are one process in two phases. They do not work apart — the
entry point dispatches the phases, and the phases read what the entry point
wrote. Install the family.

> The `requirements/` folder in this repository is just a container for these
> skills. It is unrelated to the `requirements/` folder the skills create in
> _your_ project.

## The two processes

| Phase            | Entry point                    | Precondition                          | Takes you from                          |
| :--------------- | :----------------------------- | :------------------------------------ | :-------------------------------------- |
| 1 — requirements | [req-workflow] `/requirements` | none — this is where a request enters | an incoming vision to a frozen contract |
| 2 — development  | [req-develop] `/develop`       | `03-technical.yaml` is `frozen`       | a frozen contract to verified code      |

```text
Phase 1
registered → elicitation → br_review → br_approved → technical → frozen

Phase 2
planning → plan_approved → implementing → verifying → done
```

`blocked` is a transient state in Phase 2: entered by `req-guards` when a guard
fails, left through `resume` once the failing guards are clean.

## Minimal scenario

Phase 1, from an incoming request to a sealed contract:

```bash
/requirements new request: <vision text>   # creates PCR-0001, starts the interview
PCR-0001 let's continue                    # pick it back up after a break
PCR-0001 here are the answers: <answers>   # that elicitation round is closed
PCR-0001 requirements agreed                # BR approved
PCR-0001 write the technical spec          # generates the full 03-technical.yaml
PCR-0001 freeze                            # the contract is sealed
PCR-0001 status                            # current state and the next legal step
```

Phase 2, from the sealed contract to verified code:

```bash
/develop PCR-0001 plan          # a plan from the frozen spec -> 04-plan.md
PCR-0001 plan looks good        # plan_approval: a human approved the plan
PCR-0001 implement REQ-010      # implement; guard_conditions checked before the step
PCR-0001 implement REQ-010..013 # several REQs: one pass, one guard check each
PCR-0001 REQ-013 is done        # evidence for that REQ recorded (verification)
PCR-0001 all REQs closed        # verification_evidence: the contract is executed
PCR-0001 status                 # state, and the REQs still open
```

Three things worth knowing before you type any of it:

- Only the first line of each cycle needs the `/`-command. After that, any
  phrase from the event table works.
- The `PCR-0001` prefix is only needed when more than one record is open. With
  exactly one, say it and the agent will find it.
- Both cycles are resumable. State lives in the record files, so any session
  continues by reading the folder.

## Key rules

1. Development does not start without a frozen contract, and no REQ is
   implemented without an approved plan.
2. A failing guard blocks the step (`blocked`) until it is cleared; a REQ does
   not close without evidence.
3. A frozen spec is never edited. A scope change is a new PCR through change
   control, not a fixup to the contract.
4. History is append-only — questions, revisions, guard logs and evidence only
   accumulate.
5. Neither entry point commits anything. They write files; committing stays a
   human decision.
6. A term is defined once, in `requirements/glossary.md`, and snapshotted into
   every PCR that uses it. A conflicting definition stops the process — it is a
   question, never an overwrite, and it blocks `freeze` while open.
7. An actor is identified once, in `requirements/stakeholders.md`, and the PCR
   records only their interest and position. Ambiguous approval authority stops
   `approve`; every other roster inconsistency is a warning.

## How it is wired

```text
Phase 1 — /requirements

  registered → elicitation → br_review → br_approved → technical → frozen
                             ▲           │
                             └───────────┘
                               feedback

Phase 2 — /develop

  frozen ──▶ planning → plan_approved → implementing → verifying → done
                                             ⇅
                                          blocked
```

`feedback` is the only back-edge: it sends `br_approved` or `technical` back to
`br_review` and reopens the BR, as long as the BR is still `proposed`.

The phase skills carry `disable-model-invocation: true`. The agent will never
reach for them on its own; only the entry point loads them.

[req-elicitation] borrows the interview method of the [grilling] skill but
inlines it, so the family has no skill dependency — installing the eight skills
is enough.

## Why the phases read so little

Every skill in the family is under ~9 KB, which is not where the cost is. The
cost is in what the skills make the agent read and write, and it multiplies as a
record grows: a PCR at twenty REQs has a contract and a plan of tens of
kilobytes each, all append-only.

So each entry point carries a short cost discipline that the phases inherit:

- **Sliced reads.** Classification reads front matter and gate flags only; a
  phase reads the one REQ block, table row or section it needs, found by grep.
  The one legitimate whole-document read is the BR document when generating the
  YAML, and the contract when planning.
- **Edit, never rewrite.** Append-only logs are appended, gate flags are edited
  in place, and a document is never regenerated to change one section.
- **No echo.** Validation (a silent `yaml.safe_load`) replaces re-reading what
  was just written.
- **One report.** Phases return a compact result; only the entry point writes to
  the user.
- **Batch.** Several REQs — or several clarification items — in one message are
  one event with one pass over the record.

| Phase skill          | Dispatched by  | On event                     |
| :------------------- | :------------- | :--------------------------- |
| [req-register]       | `req-workflow` | `new`                        |
| [req-elicitation]    | `req-workflow` | `new`, `continue`, `answers` |
| [req-technical-spec] | `req-workflow` | `technical`, `freeze`        |
| [req-plan]           | `req-develop`  | `plan`, `plan_approve`       |
| [req-guards]         | `req-develop`  | `implement`, `resume`        |
| [req-verification]   | `req-develop`  | `verify`, `complete`         |

[req-elicitation] runs the [grilling] interview, so that skill is a dependency
of the family.

## The record

State lives in the record files, never in the agent's memory, so any session
resumes a PCR by reading the folder. The skills expect this to be a normal,
committed part of your project — nothing is global state.

Two artifacts are project-wide, and both have a per-PCR counterpart.
`requirements/glossary.md` is the canon for terminology; each PCR carries a
snapshot of the terms it uses, so a frozen contract never changes meaning. An
existing project glossary found elsewhere is recorded as read-only
`glossary_ref:` in the registry header and is never edited.
`requirements/stakeholders.md` is the roster of actors, roles and authority;
each PCR records only the actors this change concerns, their interest and
position.

```text
requirements/
  README.md        registry: one row per PCR, glossary paths   req-register, then every transition
  glossary.md      project-wide glossary, append-only          req-elicitation
  stakeholders.md  project-wide actor registry                req-register, then req-elicitation
  PCR-NNNN-<slug>/
    01-vision.md   PART A verbatim vision (immutable)  req-register
                   PART B clarification log           req-elicitation, append-only
    02-business.md BR: draft → proposed → approved     req-elicitation, approved at the event
                   terms and stakeholders for this PCR req-elicitation
    03-technical.yaml  machine-readable REQs           req-technical-spec; gates flipped in place
    04-plan.md     plan, guard log, evidence matrix    req-plan, req-guards, req-verification
```

## The four gates

Each gate is flipped by the skill that owns it, never by hand, and only in this
order:

| Gate                    | Flipped by           | Means                                     |
| :---------------------- | :------------------- | :---------------------------------------- |
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
— copy [requirements.md] and [develop.md] into your agent's `command/` directory
by hand. See [docs/installation.md] for alternative installation methods.

## Conventions

- **`disable-model-invocation: true`** on the phase skills. This is what keeps
  the entry points the only way in.
- **`metadata: protected: "true"`** on every skill in the family. It marks them
  as off-limits to maintenance tooling: no archiving, no rewriting, no pruning
  them out of an install.

## License

MIT. See [LICENSE].

<!-- refs -->

[req-workflow]: req-workflow/
[req-develop]: req-develop/
[req-register]: req-register/
[req-elicitation]: req-elicitation/
[req-technical-spec]: req-technical-spec/
[req-plan]: req-plan/
[req-guards]: req-guards/
[req-verification]: req-verification/
[grilling]: ../grilling/
[requirements.md]: ../../command/requirements.md
[develop.md]: ../../command/develop.md
[docs/installation.md]: ../../docs/installation.md
[LICENSE]: ../../LICENSE
