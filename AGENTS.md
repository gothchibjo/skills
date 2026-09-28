# AGENTS.md

House rules for agents and contributors working in this repository.

## Formatting

This repository is Markdown only, and Prettier is the formatter. Its config is
committed, so formatting is not a matter of taste — run it before every commit:

```bash
npx prettier --write "**/*.md"   # apply
npx prettier --check "**/*.md"   # verify
```

`proseWrap: always` reflows prose at 80 columns. Do not hand-tune line breaks to
fight it.

## Commits

The [`commit`](skills/commit/) skill generates messages in this project's
Conventional Commits style, if you want it.

## Adding a skill

1. `skills/<name>/SKILL.md`, with `name` and `description` in the frontmatter.
2. `skills/<name>/README.md` — what it does, when to use it, install,
   troubleshooting. Documentation for humans, not for the agent.
3. Add it to the catalogue in the root [README.md](README.md).
4. If it needs a slash command, add `command/<name>.md`. The `skills` CLI does
   not install those; users copy them by hand.
5. Run Prettier.
