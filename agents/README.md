# reviewer

An independent, read-only reviewer for implementations, architecture,
documentation, requirements and business processes.

## What it does

The reviewer evaluates work that already exists and reports defects,
inconsistencies, omissions and risks before the work is accepted. It is an
evaluator, not an implementation assistant: it never edits files, never runs
commands and never fixes what it finds.

Every finding carries an ID, a severity, a location, an impact and a
recommendation, and the report ends with one of four verdicts.

## Operating mode

| Property   | Value                                           |
| :--------- | :---------------------------------------------- |
| Mode       | `subagent`                                      |
| Access     | read-only — `edit` and `shell` are denied       |
| Steps      | 20                                              |
| Invocation | delegation from a primary agent, or `@reviewer` |
| Definition | [`reviewer.md`](reviewer.md)                    |

The body of `reviewer.md` is the system prompt. `README.md` documents the agent
for humans and is not loaded by OpenCode.

## What it reviews

| Domain                   | Focus                                                                 |
| :----------------------- | :-------------------------------------------------------------------- |
| Software implementation  | Correctness, edge cases, security, data integrity, concurrency, tests |
| Architecture and design  | Boundaries, coupling, resilience, overengineering                     |
| Technical documentation  | Accuracy, completeness, consistency with the implementation           |
| Policies and regulations | Clarity, responsibilities, conflicts, enforceability                  |
| Requirements and specs   | Testability, ambiguity, acceptance criteria, traceability             |
| Business processes       | Roles, sequencing, decision paths, handoffs                           |

Only the criteria of the identified domains are applied. A review that needs
context it does not have ends as **Inconclusive** rather than as a guess.

## Severity levels

| Severity | Meaning                                                                |
| :------- | :--------------------------------------------------------------------- |
| Critical | Fundamental defect or severe operational/security risk                 |
| High     | Significant defect or violation of an important requirement            |
| Medium   | Incorrect behaviour in specific scenarios or substantial quality issue |
| Low      | Minor defect with limited impact                                       |
| Info     | Observation or improvement recommendation                              |

## Final assessment

- **Approved** — no significant defects identified.
- **Approved with observations** — no blocking defects; minor issues remain.
- **Changes required** — significant defects must be addressed.
- **Inconclusive** — insufficient information for a reliable assessment.

## Invoking it

Ask the primary agent to delegate to it:

```text
Use the reviewer subagent to review the changes made in this task.
```

Or address it directly, where your harness supports agent mentions:

```text
@reviewer Review this architectural design and list its weaknesses.
```

The agent cannot run `git` or tests — shell access is denied by design. It sees
the files as they are on disk, not a diff. State the scope explicitly, and paste
the diff or the change summary when the review depends on what changed.

## Install

OpenCode reads agent definitions from `~/.config/opencode/agents/` (global) or
`.opencode/agents/` (project). Copy the single file:

```bash
REPO=https://raw.githubusercontent.com/gothchibjo/skills/main
mkdir -p ~/.config/opencode/agents
curl -fsSL "$REPO/agents/reviewer.md" \
  -o ~/.config/opencode/agents/reviewer.md
```

For a project-local agent, write the same file to `.opencode/agents/reviewer.md`
and commit it with the project.

If you work on the agent itself, symlink the checkout instead:

```bash
REPO=~/Documents/GitHub/gothchibjo/skills
ln -s "$REPO/agents/reviewer.md" ~/.config/opencode/agents/reviewer.md
```

Restart the session afterwards so OpenCode rescans the agents directory. See the
[installation guide] for the other artifacts in this repository.

## Adapting it

The frontmatter is the whole configuration:

- **Model.** Add `model: provider/model#variant` to pin a model. Without it the
  subagent inherits the parent session's model.
- **Budget.** `steps: 20` caps the number of model steps. Raise it for large
  reviews, lower it for quick checks.
- **Permissions.** The `edit` and `shell` deny rules are what make the reviewer
  read-only. To let it inspect diffs, allow a narrow shell rule instead of
  removing the deny:

  ```yaml
  permissions:
    - action: edit
      resource: "*"
      effect: deny
    - action: shell
      resource: "*"
      effect: deny
    - action: shell
      resource: "git diff *"
      effect: allow
  ```

  The last matching rule wins, so the exception must come after the deny.

- **Language.** The last line of the prompt tells the agent to answer in the
  user's language. Remove it to force a single review language.

## Troubleshooting

| Symptom                                | Fix                                                                                                                  |
| :------------------------------------- | :------------------------------------------------------------------------------------------------------------------- |
| The agent is not offered as a subagent | The file must be in `~/.config/opencode/agents/` or `.opencode/agents/`; check the YAML frontmatter parses.          |
| It edits files anyway                  | The `edit` deny rule is missing or comes before a broader allow rule. Last matching rule wins.                       |
| It cannot see what changed             | Shell is denied, so it cannot run `git diff`. Paste the diff or describe the scope in the request.                   |
| Findings are vague                     | The request lacked context or acceptance criteria. Say what the work was supposed to do and which conventions apply. |
| It reports preferences as defects      | Ask for the evidence and impact of each finding; the prompt requires both.                                           |

## License

MIT. See [LICENSE].

<!-- refs -->

[LICENSE]: ../LICENSE
[installation guide]: ../docs/installation.md
