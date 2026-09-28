# Skills

Agent skills for coding agents.

Small, composable instruction sets. Each one gives your agent a repeatable
process for something worth doing the same way twice — opinionated where it
matters, deliberately boring everywhere else.

They are plain `SKILL.md` files, they work with any model, and they are meant to
be adapted rather than obeyed.

## Inspiration

This repository is **inspired by [`mattpocock/skills`]** — "Skills for Real
Engineers". Its README, its split between user-invoked and model-invoked skills,
its frontmatter conventions and the `npx skills add` installation flow all
follow that work. Some skills here are vendored from it with full MIT
attribution; see [THIRD_PARTY_NOTICES.md].

If you like this, go read his repo — it is a superset, and a better tour of the
ideas.

## Install

```bash
# everything, globally, for OpenCode
npx skills add gothchibjo/skills -g -a opencode

# just one skill
npx skills add gothchibjo/skills --skill commit -g -a opencode

# what is in the box
npx skills add gothchibjo/skills --list
```

The `skills` CLI copies or symlinks the skill files into the agent's skills
directory. Full matrix, manual installation and the OpenCode symlink setup:
**[docs/installation.md]**.

Skills are plain `SKILL.md` files with YAML frontmatter, so anything that speaks
the [Agent Skills spec] can run them.

## Catalogue

How the skill gets invoked:

| Value   | Meaning                                                               |
| ------- | --------------------------------------------------------------------- |
| `user`  | Only you can trigger it.                                              |
| `model` | The agent may pick it up on its own when the task fits.               |
| `both`  | Either.                                                               |
| `phase` | Never invoked directly — loaded only by the skill that dispatches it. |

### Available

| Skill                  | Invocation | What it does                                                                                                                                                 |
| ---------------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [`commit`]             | model      | Conventional Commits message from the staged diff, matching the project's own commit style, then commits.                                                    |
| [`grill-me`]           | user       | Relentless interview to sharpen a plan or design. Vendored from mattpocock/skills.                                                                           |
| [`grilling`]           | model      | The reusable interview primitive behind `grill-me`. Vendored from mattpocock/skills.                                                                         |
| [`req-workflow`]       | both       | Phase 1 entry point. Registers a customer vision as a PCR, elicits clarifications, approves business requirements, generates and freezes the technical spec. |
| [`req-register`]       | phase      | Creates the PCR record: vision verbatim as PART A, empty clarification log, skeletons.                                                                       |
| [`req-elicitation`]    | phase      | Runs the clarification interview, appends every round to PART B, writes the business requirements once the frontier is empty.                                |
| [`req-technical-spec`] | phase      | Deterministic BR to REQ mapping, coverage report, YAML validation, the requirement-freeze gate.                                                              |
| [`req-develop`]        | both       | Phase 2 entry point. Plans from a frozen contract, requires plan approval, guards every step, gates completion on evidence.                                  |
| [`req-plan`]           | phase      | Per-REQ task breakdown, topological order, the evidence each REQ must produce, the plan-approval gate.                                                       |
| [`req-guards`]         | phase      | Checks `guard_conditions` before each implementation step and blocks on violations.                                                                          |
| [`req-verification`]   | phase      | Closes each REQ only with matching evidence; builds the BR to REQ to evidence chain.                                                                         |

## The requirements pipeline

Eight of these skills are one process in two phases: `req-workflow` takes an
incoming vision to a frozen contract, `req-develop` takes that contract to
verified code. They only make sense installed together, and the six `phase`
skills are never invoked on their own — the entry point dispatches them.

States live in the record files, never in the agent's memory, so any session
resumes a PCR by reading the folder. Four gates hold the process together:
`requirement_freeze` → `plan_approval` → `guard_conditions` →
`verification_evidence`.

**[skills/requirements/README.md]** — the state diagram, the record layout,
which skill dispatches which, and the gates.

## Commands

OpenCode command wrappers live in [`command/`]. The `skills` CLI does not
install these — they are a few lines each and trivially copied by hand.
`docs/installation.md` has the one-liner.

| Command         | Skill          | Purpose                               |
| --------------- | -------------- | ------------------------------------- |
| `/commit`       | `commit`       | Generate a commit message and commit. |
| `/requirements` | `req-workflow` | Drive the requirements state machine. |
| `/develop`      | `req-develop`  | Drive the development state machine.  |

## License

MIT. See [LICENSE]. Vendored third-party skills keep their own copyrights in
[THIRD_PARTY_NOTICES.md].

<!-- refs -->

[`mattpocock/skills`]: https://github.com/mattpocock/skills
[THIRD_PARTY_NOTICES.md]: THIRD_PARTY_NOTICES.md
[docs/installation.md]: docs/installation.md
[Agent Skills spec]: https://agentskills.io
[`commit`]: skills/commit/
[`grill-me`]: skills/grill-me/
[`grilling`]: skills/grilling/
[`req-workflow`]: skills/requirements/req-workflow/
[`req-register`]: skills/requirements/req-register/
[`req-elicitation`]: skills/requirements/req-elicitation/
[`req-technical-spec`]: skills/requirements/req-technical-spec/
[`req-develop`]: skills/requirements/req-develop/
[`req-plan`]: skills/requirements/req-plan/
[`req-guards`]: skills/requirements/req-guards/
[`req-verification`]: skills/requirements/req-verification/
[skills/requirements/README.md]: skills/requirements/README.md
[`command/`]: command/
[LICENSE]: LICENSE
