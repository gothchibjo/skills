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

`.prettierignore` exists for a reason: the vendored skills listed in
[THIRD_PARTY_NOTICES.md] keep their bodies byte-identical to upstream, so
Prettier must not touch them. Never edit those bodies in place — re-fetch
upstream, re-apply only the frontmatter additions, and re-run the diff. If you
add a vendored skill, add its path to `.prettierignore` in the same commit.

## Commits

The [`commit`] skill generates messages in this project's Conventional Commits
style, if you want it.

## Adding a skill

1. `skills/<name>/SKILL.md`, with `name` and `description` in the frontmatter.
2. `skills/<name>/README.md` — what it does, when to use it, install,
   troubleshooting. Documentation for humans, not for the agent.
3. Add it to the catalogue in the root [README.md].
4. If it needs a slash command, add `command/<name>.md`. The `skills` CLI does
   not install those; users copy them by hand.
5. Run Prettier.

Skills that belong to one pipeline are a family: they cross-reference each other
by relative path, and the phase skills carry `disable-model-invocation: true` so
only the entry point can load them. Install them together and document them
together, and give the family its own `README.md`.

## Where a skill may live

`skills/<name>/` for a standalone skill, `skills/<family>/<name>/` for a member
of a family. One grouping level is the limit, and the reason is mechanical
rather than stylistic: the `skills` CLI scans the repository root one level deep
but `skills/` three levels deep, so `skills/<family>/<name>/SKILL.md` is found
by the plain `npx skills add` while a family in a top-level folder is not found
at all. Two grouping levels would need `--full-depth`, which pushes that flag
onto every reader. Do not use a `.claude-plugin` manifest to work around this.

The CLI's other discovery roots — `.agents/skills/`, `.claude/skills/` and the
rest — are install targets, not places to author skills in.

<!-- refs -->

[THIRD_PARTY_NOTICES.md]: THIRD_PARTY_NOTICES.md
[`commit`]: skills/commit/
[README.md]: README.md
