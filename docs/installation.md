# Installation

Every skill is a plain `SKILL.md` file with YAML frontmatter (`name`,
`description`), so anything implementing the [Agent Skills spec] can run them.
Three setup paths, from least to most control.

## 1. The `skills` CLI (recommended)

```bash
# install everything globally for OpenCode
npx skills add gothchibjo/skills -g -a opencode

# install one skill
npx skills add gothchibjo/skills --skill commit -g -a opencode

# inspect before installing
npx skills add gothchibjo/skills --list

# also install for other agents
npx skills add gothchibjo/skills -g -a claude-code -a codex
```

By default the CLI creates a canonical copy and symlinks the agent directories
to it, so a single edit propagates everywhere. Use `--copy` if your setup does
not support symlinks.

| Option       | Effect                                                          |
| :----------- | :-------------------------------------------------------------- |
| `-g`         | install to the user directory instead of the project            |
| `-a <agent>` | target a specific agent (`opencode`, `claude-code`, `codex`, …) |
| `-s <name>`  | install specific skills by name, `'*'` for all                  |
| `--list`     | list available skills without installing                        |
| `--copy`     | copy files instead of symlinking                                |
| `-y`         | skip all prompts (CI)                                           |

Installation paths per agent:

| Agent       | Global path                  | Project path      |
| :---------- | :--------------------------- | :---------------- |
| OpenCode    | `~/.config/opencode/skills/` | `.agents/skills/` |
| Claude Code | `~/.claude/skills/`          | `.claude/skills/` |
| Codex       | `~/.codex/skills/`           | `.agents/skills/` |

## 2. Manual copy

No tooling, full control. Download the skill directory you want:

```bash
REPO=https://raw.githubusercontent.com/gothchibjo/skills/main

mkdir -p ~/.config/opencode/skills/commit
for f in SKILL.md README.md; do
  curl -fsSL "$REPO/skills/commit/$f" -o ~/.config/opencode/skills/commit/$f
done
```

Or clone and copy:

```bash
git clone https://github.com/gothchibjo/skills.git
cp -r skills/commit ~/.config/opencode/skills/
```

The `README.md` next to each `SKILL.md` is documentation for humans and is
optional — agents only read `SKILL.md`.

## 3. Symlink from a checkout (development)

If you are working on the skills themselves, symlink them into your agent
directory so edits take effect immediately, with no reinstall:

```bash
REPO=~/Documents/GitHub/gothchibjo/skills
ln -s "$REPO/skills/commit" ~/.config/opencode/skills/commit
```

Restart the agent session afterwards so it re-scans the skills directory.

## Commands are not installed by the CLI

The `skills` CLI installs skills only. The OpenCode command wrappers in
[command/] — `/commit`, `/requirements`, `/develop` — have to be copied by hand:

```bash
REPO=https://raw.githubusercontent.com/gothchibjo/skills/main
mkdir -p ~/.config/opencode/command
for c in commit requirements develop; do
  curl -fsSL "$REPO/command/$c.md" -o ~/.config/opencode/command/$c.md
done
```

Each one is a few lines: a description and a pointer to the matching skill.

## Verifying an install

```bash
npx skills list          # what the CLI thinks is installed
npx skills add gothchibjo/skills --list   # what the repo offers
```

In the agent itself, ask for the skill by name. A skill that does not appear is
almost always a wrong directory, a missing `name`/`description` in the
frontmatter, or a session that started before the file was copied.

## Requirements pipeline

`req-workflow` and `req-develop` write into `requirements/` in whichever project
you are working in, and expect that directory to be a normal, committed part of
that project. Nothing is global state. The development phase adds `04-plan.md`
to the same folder. See the catalogue in the root [README] for the family and
its gates.

<!-- refs -->

[Agent Skills spec]: https://agentskills.io
[command/]: ../command/
[README]: ../README.md
