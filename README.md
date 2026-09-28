# Skills

Agent skills for coding agents.

Small, composable instruction sets. Each one gives your agent a repeatable
process for something worth doing the same way twice — opinionated where it
matters, deliberately boring everywhere else.

They are plain `SKILL.md` files, they work with any model, and they are meant to
be adapted rather than obeyed.

## Inspiration

This repository is **inspired by
[`mattpocock/skills`](https://github.com/mattpocock/skills)** — "Skills for Real
Engineers". Its README, its split between user-invoked and model-invoked skills,
its frontmatter conventions and the `npx skills add` installation flow all
follow that work.

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
**[docs/installation.md](docs/installation.md)**.

Skills are plain `SKILL.md` files with YAML frontmatter, so anything that speaks
the [Agent Skills spec](https://agentskills.io) can run them.

## Catalogue

`user` means only you can trigger it (`/skill-name`); `model` means the agent
can pick it up on its own when the task fits.

### Available

| Skill                      | Invocation | What it does                                                                                              |
| -------------------------- | ---------- | --------------------------------------------------------------------------------------------------------- |
| [`commit`](skills/commit/) | model      | Conventional Commits message from the staged diff, matching the project's own commit style, then commits. |

## Commands

OpenCode command wrappers live in [`command/`](command/). The `skills` CLI does
not install these — they are a few lines each and trivially copied by hand.
`docs/installation.md` has the one-liner.

## License

MIT. See [LICENSE](LICENSE).
