# pattern-observer

Read the git history for what you keep doing by hand, and turn the repeats into
skills.

## What it does

The skill looks at recently added files, groups them into clusters of similar
work, and reports the patterns:

```text
Found pattern: Creating employee profile documents

Detected in:
- employees/ivan-ivanov/README.md (3 days ago)
- employees/petr-petrov/README.md (1 week ago)
- employees/maria-smirnova/README.md (2 weeks ago)

Proposed action: Create new skill 'employee-profile-creator'
```

For every cluster it then checks the existing skills and picks one of four
outcomes: propose a new skill, propose an edit to a skill that half-covers the
pattern, do nothing because a skill already covers it, or propose a refactor of
a skill that grew by accretion.

## When to use

Model-invokable and user-invocable. The agent triggers it on `/observer`, "find
patterns in my activity", "suggest a skill for this", "this skill has become a
mess".

Two thresholds keep it honest: nothing is proposed for a one-off, and a single
repetition is not a pattern. It waits for 2–3 similar actions first.

## Usage

Invoked by name as a skill, or through the `/observer` command in [command]:

| Invocation  | Behaviour                                          |
| :---------- | :------------------------------------------------- |
| `/observer` | analyse the current repository and propose changes |

## Behaviour worth knowing

- **Analysis scope is created files, not edits.**
  `git log --diff-filter=A --name-only` over the last 50 commits, widened to 100
  if the result is thin. Patterns in _how_ you write do not show up there;
  repeated _creation_ does.
- **Accretion check runs alongside.** A skill with 5+ incremental commits since
  creation is a candidate for consolidation, and grab-bag "Notes" sections,
  rules scattered across sections, or templates that no real file in the repo
  matches are the signals it looks for.
- **Proposals are shown, not applied.** The full `SKILL.md`, or a diff for an
  existing skill, is presented for confirmation before anything is written.
- **Usage tracking.** On completion it stamps today's date into
  `.opencode/.skill-usage.json`, which `skill-gardener` reads and which is
  gitignored by contract.

## Install

```bash
# with the skills CLI
npx skills add gothchibjo/skills --skill pattern-observer -a opencode

# or manually
mkdir -p ~/.config/opencode/skills/pattern-observer
curl -fsSL https://raw.githubusercontent.com/gothchibjo/skills/main/skills/pattern-observer/SKILL.md \
  -o ~/.config/opencode/skills/pattern-observer/SKILL.md
```

See [docs/installation.md] for the full matrix.

## Troubleshooting

| Symptom                                      | Fix                                                                                                                     |
| :------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------- |
| `/observer` is not found                     | The `skills` CLI installs skills only. Copy `command/observer.md` to `~/.config/opencode/command/observer.md` yourself. |
| "No patterns found" on a busy repository     | The scope is added files, and those commits may predate the window. Ask for a longer window.                            |
| It proposes a skill for something done twice | Correct behaviour for a real repeat — but check the pattern is work, not coincidence.                                   |
| The proposal writes into the wrong directory | Patterns are project-specific and land in `.opencode/skills/`, not the global directory.                                |

## Related

[skill-gardener] is the other half: it ages skills out. The observer makes them,
the gardener prunes them.

## License

MIT. See [LICENSE].

<!-- refs -->

[command]: ../../command/observer.md
[skill-gardener]: ../skill-gardener/
[docs/installation.md]: ../../docs/installation.md
[LICENSE]: ../../LICENSE
