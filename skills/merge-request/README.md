# merge-request

Open a merge request (pull request on GitHub) from the current branch, with a
commit-style title and a description that covers the branch's changes.

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

After showing the draft and the list of commits, it asks for confirmation. Only
then does it push the current branch (when needed) and open the request.

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
branch.

## Behaviour worth knowing

- **Confirmation before publishing.** The title, description and commit list are
  shown first. Nothing is pushed and no request is created until you say yes.
- **Committed work only.** The skill never stages, commits, stashes or resets. A
  dirty working tree produces a warning, not a commit.
- **No force-push, no merging.** If the remote branch has diverged, the push is
  rejected and the skill stops. It never pushes to the target branch and never
  merges the request.
- **Duplicates are avoided.** If an open request already exists for the same
  source and target, the skill offers to update its title and description
  instead of opening a second one.
- **Host-aware.** GitHub uses `gh pr create`; GitLab uses `glab mr create`. On
  an unknown host, or when the CLI is missing, the skill prints the draft and a
  ready-to-open web URL instead of guessing an API call.

## Install

See the [installation guide] for alternative installation methods. To install
just this skill:

```bash
npx skills add gothchibjo/skills --skill merge-request -a opencode
```

The `/mr` wrapper is not installed by the `skills` CLI. Copy
[command/mr.md](../../command/mr.md) to `~/.config/opencode/command/mr.md`.

## Troubleshooting

| Symptom                                | Fix                                                                                                               |
| :------------------------------------- | :---------------------------------------------------------------------------------------------------------------- |
| `/mr` is not found                     | The `command/` wrappers are not installed by the CLI. Copy `command/mr.md` to `~/.config/opencode/command/mr.md`. |
| `gh: command not found`                | Install the GitHub CLI, then run `gh auth login`.                                                                 |
| `glab: command not found`              | Install the GitLab CLI, then run `glab auth login`.                                                               |
| "no commits to put in a merge request" | The branch has no commits ahead of the target. Commit first or pick another target with `/mr <branch>`.           |
| Push rejected (non-fast-forward)       | The remote branch diverged. Rebase or merge manually; the skill will not force-push.                              |
| Uncommitted changes are missing        | Expected: only commits are included. Commit the work first and run `/mr` again.                                   |
| The title or description is vague      | The branch history was ambiguous. State the intent, or squash/clean up the commits and retry.                     |

## License

MIT. See [LICENSE].

<!-- refs -->

[commit]: ../commit/
[installation guide]: ../../docs/installation.md
[LICENSE]: ../../LICENSE
