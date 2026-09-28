# grilling

> Vendored from [`mattpocock/skills`] · © 2026 Matt Pocock
>
> MIT licensed — see the [full notice] for provenance.

Interview the user relentlessly about a plan, decision, or idea until every
branch of the design tree is resolved.

**Model-invoked.** The agent can reach for this on its own, and it is also the
implementation behind [`grill-me`] — a user-invoked wrapper around it. The split
is deliberate: a user-invoked skill orchestrates, a model-invoked skill holds
the reusable discipline.

## How it works

The topic is mapped as a **design tree**: every decision branches into the
decisions that hang off it. The tree is worked in **rounds**, and each round
asks the whole **frontier** — every decision whose prerequisites are already
settled. Questions are numbered and each comes with a recommended answer.

```
❓ **Q1** - **<question title>**: <question body>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body>

➡️ <your recommended answer>
```

Answers reshape the tree, the frontier is recomputed, and the next round is
asked. The session is done when the frontier is empty.

## The two rules that make it work

- **Facts are the agent's job.** When a frontier question needs a fact from the
  environment, the agent dispatches a sub-agent instead of asking. It does not
  block on the sub-agent: only downstream questions wait.
- **Decisions are the user's.** Every judgement call is put to the user and the
  answer is awaited. Nothing is acted on until the user confirms shared
  understanding.

A question whose answer depends on another question still open in the same round
is deferred to a later round, not guessed at. That is what stops the interview
from collapsing into a wall of questions the user cannot answer in order.

## When to use it

Any time the plan has unexamined branches. "Plan a refactor" and "we should move
to event sourcing" are both better after a grilling round than before.

## Install

```bash
npx skills add gothchibjo/skills --skill grilling -g -a opencode
```

See [docs/installation.md] for other methods.

## Updating from upstream

Vendored, not owned. The body is byte-identical to upstream at the commit
recorded in the frontmatter. To update: re-fetch the upstream file, re-apply the
frontmatter additions, re-run the diff. Do not edit the body. Details in
[THIRD_PARTY_NOTICES.md].

## License

MIT, © 2026 Matt Pocock. See [THIRD_PARTY_NOTICES.md].

<!-- refs -->

[`mattpocock/skills`]: https://github.com/mattpocock/skills
[full notice]: ../../THIRD_PARTY_NOTICES.md
[`grill-me`]: ../grill-me/
[docs/installation.md]: ../../docs/installation.md
[THIRD_PARTY_NOTICES.md]: ../../THIRD_PARTY_NOTICES.md
