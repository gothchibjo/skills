# commit

Generate a Conventional Commits-style commit message from the currently staged
changes and commit it.

## What it does

The skill reads `git diff --staged`, works out what actually changed and why,
and produces a message with a summary line, a bullet list of key changes and a
short purpose paragraph — then commits it.

```text
type(scope): imperative summary
<blank line>
- bullet describing key change
- bullet describing key change
<blank line>
Short purpose paragraph explaining why this change was made.
```

Message rules:

- `type` is one of `feat`, `fix`, `refactor`, `docs`, `test`, `chore`. Another
  type is only introduced when none of these fit, and the skill says why.
- `scope` is required, derived from the paths in the diff (e.g.
  `licensing,cli`).
- Subject is imperative mood ("add", not "added"), no trailing period.
- The body is **always** required, even for small or merge commits.
- Breaking changes get `!` after the type/scope and/or a `BREAKING CHANGE:`
  footer.

## When to use

Model-invoked. Triggers on "commit this", "write a commit message", "generate a
commit message for these changes", and on being asked to review or fix an
existing message against the project's conventions.

## Usage

Installed as a skill it can be invoked directly by name, or through the
`/commit` command in [opencode]:

| Invocation     | Behaviour                          |
| :------------- | :--------------------------------- |
| `/commit`      | generate the message and commit    |
| `/commit push` | generate the message, commit, push |

## Behaviour worth knowing

- **Only staged changes are committed.** The skill never runs `git add` to pull
  unstaged or untracked files into the commit. The single exception: if the
  staging area is completely empty, it runs `git add -A` rather than stopping to
  ask.
- **Large diffs are summarised, not transcribed.** Above roughly 500 lines the
  skill switches to `git diff --staged --stat` first and writes 3–5 bullets
  grouped by logical change, not one bullet per file.
- **Ambiguous scope is escalated.** If the diff spans genuinely distinct areas
  and the scope is not obvious, the skill lists the areas and asks before
  committing.
- **The message is shown before it is used** in a code block, so you can see
  what is being committed.

## Install

```bash
# with the skills CLI
npx skills add gothchibjo/skills --skill commit -a opencode

# or manually
mkdir -p ~/.config/opencode/skills/commit
curl -fsSL https://raw.githubusercontent.com/gothchibjo/skills/main/skills/commit/SKILL.md \
  -o ~/.config/opencode/skills/commit/SKILL.md
```

See [docs/installation.md] for the full matrix.

## Troubleshooting

| Symptom                                      | Fix                                                                                                                                         |
| :------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------ |
| `/commit` is not found                       | The `command/` wrappers are not installed by the `skills` CLI. Copy `command/commit.md` to `~/.config/opencode/command/commit.md` yourself. |
| Scope does not match what you want           | Say the scope explicitly when asking, or rename the top-level directory.                                                                    |
| The message is too generic                   | The diff was ambiguous, so was the message. State the intent in the request.                                                                |
| A staged lockfile bump dominates the bullets | It does not, unless it is the only change. Ask for a re-draft if it looks wrong.                                                            |

## License

MIT. See [LICENSE].

<!-- refs -->

[opencode]: ../../command/commit.md
[docs/installation.md]: ../../docs/installation.md
[LICENSE]: ../../LICENSE
