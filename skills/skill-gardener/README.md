# skill-gardener

Archive skills nobody uses, restore the ones that were archived by mistake,
delete what has rotted in the archive.

## What it does

Scans both scopes — the project's `.opencode/skills/` and the global
`~/.config/opencode/skills/` — reads each skill's `last-used` from the scope's
usage file, and builds one table:

| Scope   | Skill | Status   | Last Used  | Protected | Proposed Action    |
| :------ | :---- | :------- | :--------- | :-------- | :----------------- |
| project | …     | active   | 2026-01-01 | false     | Archive            |
| global  | …     | archived | 2025-12-01 | false     | Delete             |
| project | …     | archived | 2026-07-15 | true      | (none — protected) |

Then it waits. Every action is executed only after explicit confirmation.

## When to use

Model-invokable and user-invocable. The agent triggers it on `/gardener`, "clean
up my skills", "which skills do I not use", "archive unused skills".

## Usage

Invoked by name as a skill, or through the `/gardener` command in [command]:

| Invocation  | Behaviour                                      |
| :---------- | :--------------------------------------------- |
| `/gardener` | scan both scopes and propose lifecycle actions |

## Behaviour worth knowing

- **Two scopes, both always.** The project scope archives to
  `.opencode/archived-skills/`; the global scope archives to
  `.config/opencode/skills/.archive/`. Archiving also adds `"<name>": "deny"`
  under `permission.skill` in the matching `opencode.json`, and restoring
  removes it. Both scopes are scanned in one pass — neither is optional.
- **Unknown usage is a weak signal.** A skill with no entry in the usage file is
  treated as unused after a month, which is a warning, not an archive order.
- **Protected skills are untouchable.** `metadata.protected: "true"` excludes a
  skill from every operation, including deletion from the archive. Changing that
  flag needs explicit approval.
- **Thresholds are asymmetric.** Three months unused while active → archive. One
  month since last use while archived → restore, on the assumption it was
  archived by mistake. Six months while archived → delete.
- **Global skills are shared.** They affect every project on the machine, so
  they are managed only with explicit confirmation.
- **Usage files stay out of git.** `.skill-usage.json` is gitignored in both
  scopes; the skill checks that it stays that way.

## Install

```bash
# with the skills CLI
npx skills add gothchibjo/skills --skill skill-gardener -a opencode

# or manually
mkdir -p ~/.config/opencode/skills/skill-gardener
curl -fsSL https://raw.githubusercontent.com/gothchibjo/skills/main/skills/skill-gardener/SKILL.md \
  -o ~/.config/opencode/skills/skill-gardener/SKILL.md
```

See [docs/installation.md] for the full matrix.

## Troubleshooting

| Symptom                                           | Fix                                                                                                                     |
| :------------------------------------------------ | :---------------------------------------------------------------------------------------------------------------------- |
| `/gardener` is not found                          | The `skills` CLI installs skills only. Copy `command/gardener.md` to `~/.config/opencode/command/gardener.md` yourself. |
| Every skill looks unused                          | The usage file has no entries yet. It fills in as skills run; a fresh install always looks like this.                   |
| A skill is archived but still in the agent        | The `deny` entry is missing in `opencode.json`. The skill remains listed but inaccessible until it is added.            |
| A skill I use every day is proposed for archiving | It is probably missing from `.skill-usage.json`, or predates usage tracking. Add the date by hand.                      |

## Related

[pattern-observer] is the other half: it spots the work worth capturing as a
skill. The observer makes them, the gardener prunes them.

## License

MIT. See [LICENSE].

<!-- refs -->

[command]: ../../command/gardener.md
[pattern-observer]: ../pattern-observer/
[docs/installation.md]: ../../docs/installation.md
[LICENSE]: ../../LICENSE
