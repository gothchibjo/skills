---
name: skill-gardener
description: "Manage skill lifecycle: archive unused skills, restore archived-if-needed, clean up stale skills. Use when the user runs /gardener or asks to clean up skills, check skill usage, archive old skills, or run skill maintenance."
metadata:
  protected: "true"
  copyright: "© 2026 github.com/gothchibjo"
---

# Skill Gardener

Manage the lifecycle of skills in the current project's `.opencode/skills/` and
in the global `~/.config/opencode/skills/`. Archive unused skills, restore those
that became relevant again, and clean up stale archived skills.

## Two scopes

| Scope   | Active skills                | Archive                               | Usage tracking                         | Deny config                                              |
| :------ | :--------------------------- | :------------------------------------ | :------------------------------------- | :------------------------------------------------------- |
| Project | `.opencode/skills/`          | `.opencode/archived-skills/`          | `.opencode/.skill-usage.json`          | `permission.skill` in `.opencode/opencode.json`          |
| Global  | `~/.config/opencode/skills/` | `~/.config/opencode/skills/.archive/` | `~/.config/opencode/.skill-usage.json` | `permission.skill` in `~/.config/opencode/opencode.json` |

Both scopes are always scanned; neither is optional.

## Scan all skills

1. For the project scope, read every SKILL.md in `.opencode/skills/` and
   `.opencode/archived-skills/`. For the global scope, read every SKILL.md in
   `~/.config/opencode/skills/` and `~/.config/opencode/skills/.archive/`.
2. For each skill, extract `name` and `metadata.protected` from SKILL.md. Read
   `last-used` from the scope's usage file (`.opencode/.skill-usage.json` or
   `~/.config/opencode/.skill-usage.json`). If a skill is not listed there,
   treat `last-used` as unknown (candidate for archiving after 1 month without
   usage record).

## Check active skills

For each active skill (project or global):

- If `last-used` is older than 3 months (or missing) AND `protected` is not
  `"true"`:
  - Mark as candidate for archiving.
  - Proposed action for project scope:
    `mv .opencode/skills/<name> .opencode/archived-skills/<name>` and add
    `"<name>": "deny"` under `permission.skill` in `.opencode/opencode.json`.
  - Proposed action for global scope:
    `mv ~/.config/opencode/skills/<name> ~/.config/opencode/skills/.archive/<name>`
    and add `"<name>": "deny"` under `permission.skill` in
    `~/.config/opencode/opencode.json`.
- If `last-used` is older than 1 month AND `protected` is not `"true"` AND the
  skill has no usage record in the scope's usage file: warn that the skill may
  never have been used since creation.

## Check archived skills

For each archived skill (project or global):

- If `last-used` is less than 1 month ago (recently used — may have been
  archived by mistake):
  - Mark as candidate for restore.
  - Proposed action for project scope:
    `mv .opencode/archived-skills/<name> .opencode/skills/<name>` and remove
    `"<name>": "deny"` from `.opencode/opencode.json`.
  - Proposed action for global scope:
    `mv ~/.config/opencode/skills/.archive/<name> ~/.config/opencode/skills/<name>`
    and remove `"<name>": "deny"` from `~/.config/opencode/opencode.json`.
- If `last-used` is older than 6 months AND `protected` is not `"true"`:
  - Mark as candidate for permanent deletion.
  - Proposed action for project scope:
    `git rm -r .opencode/archived-skills/<name>`.
  - Proposed action for global scope:
    `rm -r ~/.config/opencode/skills/.archive/<name>`.

## Summary and confirmation

1. Build a summary table:

   | Scope   | Skill | Status   | Last Used  | Protected | Proposed Action    |
   | :------ | :---- | :------- | :--------- | :-------- | :----------------- |
   | project | ...   | active   | 2026-01-01 | false     | Archive            |
   | global  | ...   | archived | 2025-12-01 | false     | Delete             |
   | project | ...   | archived | 2026-07-15 | true      | (none — protected) |

2. Present the table to the user.
3. Ask for explicit confirmation before executing any actions.
4. Execute only the confirmed actions.
5. Update the scope's usage file(s) with today's date for this skill after
   completion.

## Notes

- Usage files (`.opencode/.skill-usage.json`,
  `~/.config/opencode/.skill-usage.json`) are gitignored and must never be
  committed. For the project scope, ensure `.gitignore` contains
  `.opencode/.skill-usage.json`; create the entry if missing.
- Never touch skills with `metadata.protected: "true"` — they are excluded from
  all lifecycle operations.
- Removing or changing `metadata.protected` on an existing skill requires the
  user's explicit approval.
- Skills archived via `deny` in the config still have their `name` +
  `description` visible in the `available_skills` block but are hidden from the
  agent (the skill tool will reject access).
- Global skills (including this one, `pattern-observer` and `commit`) live in
  `~/.config/opencode/skills/` and are shared across projects; manage them only
  with the user's explicit confirmation.
