# grill-me

> [SKILL.md] is vendored from [mattpocock/skills] — MIT, © 2026 Matt Pocock. See
> [full notice].

A relentless interview to sharpen a plan or design.

## What it does

`grill-me` is a two-line entry point. It calls the [grilling] skill, which is
the actual interview primitive: it maps the thing you are discussing as a design
tree, then works that tree in rounds.

Each round asks the whole **frontier** — every question whose prerequisites are
already settled — numbered, with a recommended answer for each, then waits.

```markdown
❓ **Q1** - **<question title>**: <question body>

➡️ <your recommended answer>
```

Your answers reshape the tree, which produces the next round. The session ends
when the frontier is empty: every branch visited, nothing silently assumed.
Nothing is acted on until you confirm shared understanding.

## Design decisions worth knowing

- **Questions, not chores.** The skill is explicitly told that finding _facts_
  is its job, not yours. It will dispatch a sub-agent rather than ask you for
  something it could look up. A running sub-agent only blocks the questions that
  depend on it — the rest of the frontier still gets asked.
- **Decisions are yours.** Anything that is a judgement call is put to you and
  waits for your answer.
- **Recommended answers are mandatory.** You get a default for every question,
  so an unanswered question is never silently left open.
- **Rounds are honest.** A question whose answer depends on another question
  still open in the same round is deferred to a later round, not guessed at.

## When to use it

Before committing to a design, when the request is one sentence and the
implementation is a week. It is most useful when you already have an instinct
and want it attacked — that is what the design tree does: it finds the branches
of your plan you have not considered yet.

## Install

```bash
# grill-me alone is useless without grilling, so install both
npx skills add gothchibjo/skills --skill grill-me --skill grilling -g -a opencode
```

See [docs/installation.md] for other methods.

## License

MIT, © 2026 Matt Pocock, for the vendored [SKILL.md]. See
[THIRD_PARTY_NOTICES.md].

<!-- refs -->

[SKILL.md]: SKILL.md
[mattpocock/skills]: https://github.com/mattpocock/skills
[full notice]: ../../THIRD_PARTY_NOTICES.md
[grilling]: ../grilling/
[docs/installation.md]: ../../docs/installation.md
[THIRD_PARTY_NOTICES.md]: ../../THIRD_PARTY_NOTICES.md
