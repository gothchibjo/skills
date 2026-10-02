---
name: pattern-observer
description: "Discover repetitive patterns in user activity. Analyse git history, suggest new skills, improve existing ones, or propose refactoring of skills that grew by accretion. Use when the user runs /observer or asks to find patterns, auto-create skills, suggest skill improvements, or clean up an overgrown skill."
metadata:
  protected: "true"
  copyright: "© 2026 github.com/gothchibjo"
---

# Pattern Observer

Analyse user activity in the current repository to discover repetitive patterns
and suggest new skills (or improvements to existing skills) that capture those
patterns.

## Analysis scope

Analyse the git log for recently created files:

```bash
# last 50 commits or last 4 weeks, whichever is more
git log --diff-filter=A --name-only --format="" -50
```

If the output is sparse, widen to the last 100 commits.

## Pattern detection

Look for clusters of 2-3+ similar actions. Group files by:

- **Directory**: multiple files created in the same directory subtree (e.g.
  `regulations/IAM/`, `employees/`).
- **File type**: same extension, same naming pattern (e.g. `IAM-R-*.md`,
  `ECP-F-*.md`).
- **Document category**: same YAML front matter `type` or `code` prefix.
- **Edit operation**: similar content structure, same sections added.

### Examples of detectable patterns

| Pattern                                           | Trigger                                                       | Resulting skill                        |
| :------------------------------------------------ | :------------------------------------------------------------ | :------------------------------------- |
| Creating 3+ regulation docs in `regulations/ECP/` | `ECP-R-001.md`, `ECP-R-002.md`, `ECP-R-003.md` added          | Auto-create/improve regulation-manager |
| Adding 3+ employee profiles in `employees/`       | Repeated `employees/*/README.md` creation                     | Create employee-profile skill          |
| Running 3+ CV reviews                             | Repeated `cv.md` + `review.md` creation under vacancy folders | Already covered by cv-analyzer         |

## Actions

If a clear pattern is found (2-3+ similar actions):

1. **Check existing skills** — search the current project's `.opencode/skills/`
   (and global `~/.config/opencode/skills/`) SKILL.md files for overlapping
   descriptions.
2. **If no existing skill covers it** — propose creating a new skill in the
   current project's `.opencode/skills/` (patterns are project-specific):
   - Name: kebab-case, descriptive (e.g. `employee-profile-creator`)
   - Description: trigger keywords, clear intent
   - Instructions: step-by-step for the repetitive task
   - Front matter with `metadata.protected` = `"false"` (leave usage tracking to
     `.opencode/.skill-usage.json`)
3. **If an existing skill partially covers it** — propose improving it (add the
   new pattern to its instructions, update description to match more triggers).
4. **If an existing skill fully covers it** — do nothing.
5. **Skill health check** — alongside the pattern search, look for skills that
   accumulated too many patches and propose consolidating them:
   - Accretion signal: 5+ incremental commits touching one SKILL.md since its
     creation (`git log --oneline -- .opencode/skills/<name>/SKILL.md`).
   - Structural smells inside SKILL.md: grab-bag "Notes"/"Misc" sections; rules
     for one concern scattered across 3+ sections; content duplicating AGENTS.md
     or other skills; templates/examples diverging from actual files in the repo
     (unused statuses, fields, file names that no real file uses).
   - When found, propose a refactor plan: target outline, what moves where, what
     gets deduplicated, what gets aligned with actual repo practice. Show the
     proposed new structure before writing (same confirmation flow).

## Summary and confirmation

1. Present a summary:

   ```text
   Found pattern: Creating employee profile documents

   Detected in:
   - employees/ivan-ivanov/README.md (3 days ago)
   - employees/petr-petrov/README.md (1 week ago)
   - employees/maria-smirnova/README.md (2 weeks ago)

   Proposed action: Create new skill 'employee-profile-creator'
   ```

2. Ask for explicit confirmation before writing any files.
3. Update `.opencode/.skill-usage.json` with today's date for this skill after
   completion. The file is gitignored and must never be committed — ensure
   `.gitignore` contains `.opencode/.skill-usage.json`.

## Notes

- Do not propose skills for one-off actions; wait for 2-3+ repetitions.
- When improving an existing skill, show a diff of proposed changes.
- When creating a new skill, show the full proposed SKILL.md content before
  writing.
- The same analysis can be triggered manually via the `/observer` command.
