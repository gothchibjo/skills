# merge-request

Open a merge request (pull request on GitHub) from the current branch, with a
commit-style title and a description that covers the branch's changes — or, in
plan mode, just prepare the draft.

## What it does

The skill compares the current branch with the target branch, reads the commits
and the diff, and drafts a title and description using the same convention as
the [commit] skill:

```text
type(scope): imperative summary
<blank line>
- bullet describing key change
- bullet describing key change
<blank line>
Short purpose paragraph explaining why this branch exists.
```

Unlike a commit message, the description covers the **whole branch**: every
substantive commit is reflected in at least one bullet, with related commits
grouped into a single logical change.

What happens next depends on the session mode:

- **Plan mode** is read-only. The skill prints the draft, the commit list and
  the exact commands it would run, then stops. Nothing is fetched, pushed or
  created, and the index is untouched.
- **Build mode** asks for confirmation, pushes the current branch when needed
  and creates or updates the request.

## When to use

User-invocable only. The skill carries `disable-model-invocation: true`, so the
agent never picks it up on its own — use `/mr` or ask for a merge request
explicitly.

## Usage

| Invocation      | Behaviour                                                  |
| :-------------- | :--------------------------------------------------------- |
| `/mr`           | target the remote default branch (`main`, `master`, …)     |
| `/mr develop`   | target `develop`                                           |
| `/mr origin/qa` | target `qa`; the `origin/` prefix is accepted and stripped |
| `/mr release/2` | target a branch whose name contains slashes                |

The argument changes the **target** only; the source is always the current
branch. In plan mode every invocation only produces the draft.

## Behaviour worth knowing

- **Confirmation before publishing.** The draft, the commit list and the chosen
  publishing path are shown first. Nothing is pushed and no request is created
  until you say yes.
- **Committed work only.** The skill never stages, commits, stashes, resets or
  restores. A dirty working tree produces a warning, not a commit.
- **No force-push, no merging.** If the remote branch has diverged, the push is
  rejected and the skill stops. It never pushes to the target branch and never
  merges the request.
- **Duplicates are detected, not created.** When the branch is already on the
  remote, the skill lists open requests for the same source and target and
  offers to update the title and description instead of opening a second one.
- **Host-aware, by authentication.** The skill asks `gh` and `glab` whether they
  are authenticated for the remote host, so self-hosted instances are detected
  by their credentials rather than by their hostname. Without either CLI it
  prints the draft and a compare page instead of guessing an API call.
- **Templates are respected.** If the repository has a PR/MR template, its
  headings shape the description; the title keeps the convention above.

## Install

See the [installation guide] for alternative installation methods. To install
just this skill:

```bash
npx skills add gothchibjo/skills --skill merge-request -a opencode
```

The `/mr` wrapper is not installed by the `skills` CLI. Copy
[command/mr.md](../../command/mr.md) to `~/.config/opencode/command/mr.md`.

## Troubleshooting

| Symptom                                   | Fix                                                                                                                  |
| :---------------------------------------- | :------------------------------------------------------------------------------------------------------------------- |
| `/mr` is not found                        | The `command/` wrappers are not installed by the CLI. Copy `command/mr.md` to `~/.config/opencode/command/mr.md`.    |
| `gh: command not found`                   | Install the GitHub CLI, then run `gh auth login`.                                                                    |
| `glab: command not found`                 | Install the GitLab CLI, then run `glab auth login`.                                                                  |
| The branch has no commits ahead of target | Commit the work first, or pick another target with `/mr <branch>`.                                                   |
| Push rejected (non-fast-forward)          | The remote branch diverged. Rebase or merge manually; the skill will not force-push.                                 |
| Uncommitted changes are missing           | Expected: only commits are included. Commit the work first and run `/mr` again.                                      |
| `/mr` in plan mode changed nothing        | Intended: plan mode only prepares the draft. Switch to build mode and run it again.                                  |
| Not authenticated for a self-hosted host  | `gh auth login --hostname <host>` or `glab auth login --hostname <host>`; until then the skill prints a compare URL. |
| The description ignores our format        | A project template or convention takes precedence by design. Delete the template to get the default format.          |
| The title or description is vague         | The branch history was ambiguous. State the intent, or squash/clean up the commits and retry.                        |

## License

MIT. See [LICENSE].

<!-- refs -->

[commit]: ../commit/
[installation guide]: ../../docs/installation.md
[LICENSE]: ../../LICENSE
