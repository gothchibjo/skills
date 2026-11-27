# commit

Generate a Conventional Commits-style commit message from the current changes
and commit it — or, in plan mode, just produce the message.

## What it does

The skill reads the diff, works out what actually changed and why, and produces
a message with a summary line, a bullet list of key changes and a short purpose
paragraph. What happens next depends on the session mode:

- **Plan mode** is read-only. The skill prints the message and stops: nothing is
  staged, committed or pushed, even with `push`. You review the message, switch
  to build mode and commit there.
- **Build mode** prints the message and commits it in one go. The printed
  message is the record of what was committed, not an approval gate.

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
- The format is the default. Project instructions the harness has loaded win
  over it, and so does the style of the last ~10 subjects in `git log`.

## When to use

Model-invokable and user-invocable. The agent triggers it on "commit this",
"write a commit message", "generate a commit message for these changes", and on
being asked to review or fix an existing message against the project's
conventions.

## Usage

Installed as a skill it can be invoked directly by name, or through the
`/commit` command in [opencode]:

| Invocation              | Behaviour                                      |
| :---------------------- | :--------------------------------------------- |
| `/commit` in plan mode  | prints the message, changes nothing            |
| `/commit` in build mode | prints the message and commits                 |
| `/commit push` in build | prints the message, commits, pushes            |
| `/commit push` in plan  | prints the message, commits and pushes nothing |

## Behaviour worth knowing

- **Plan mode touches nothing.** No `git add`, no `git commit`, no `git push`,
  and no index changes of any kind. The message is the whole deliverable.
- **Only staged changes are committed.** Once the index has content, the skill
  never runs `git add` to pull unstaged or untracked files into the commit. The
  single exception is build mode with a completely empty staging area: there it
  runs `git add -A` rather than stopping to ask.
- **Large diffs are summarised, not transcribed.** Above roughly 500 lines the
  skill switches to `git diff --staged --stat` first and writes 3–5 bullets
  grouped by logical change, not one bullet per file.
- **Ambiguous scope is escalated.** If the diff spans genuinely distinct areas
  and the scope is not obvious, the skill lists the areas and asks before
  committing.
- **Hooks are respected.** A rejected commit is fixed and retried as a new
  commit; the skill never reaches for `--amend` or `--no-verify`.

## Install

See the [installation guide] for alternative installation methods. To install
just this skill:

```bash
npx skills add gothchibjo/skills --skill commit -a opencode
```

## Troubleshooting

| Symptom                                    | Fix                                                                                                              |
| :----------------------------------------- | :--------------------------------------------------------------------------------------------------------------- |
| `/commit` is not found                     | The `command/` wrappers are not installed by the `skills` CLI. Copy `command/commit.md` to your config yourself. |
| The commit has files I did not stage       | The index was empty, so build mode staged everything. Stage files explicitly next time.                          |
| `/commit push` in plan mode did nothing    | Intended: plan mode changes nothing. Switch to build mode and run it again.                                      |
| Scope does not match what you want         | Say the scope explicitly when asking, or rename the top-level directory.                                         |
| The message misses why the change was made | The skill reads the diff, not your reasoning. State the intent in the request.                                   |
| The message is too generic                 | The diff was ambiguous. Say what the change is for.                                                              |

## License

MIT. See [LICENSE].

<!-- refs -->

[opencode]: ../../command/commit.md
[installation guide]: ../../docs/installation.md
[LICENSE]: ../../LICENSE
