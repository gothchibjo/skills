---
name: commit
description: "Commit staged changes with a Conventional Commits message: type(scope): summary, bullet body, purpose paragraph. Use when the user asks to commit, to write or review a commit message, or says \"commit this\". Arguments: no args — commit; push — commit and push. In plan mode only the message is produced."
metadata:
  protected: "false"
  copyright: "© 2026 github.com/gothchibjo"
---

# Generate Commit Message

Generates a commit message for the current changes, then commits (and optionally
pushes) according to the session mode and the arguments.

## Commit message format

```text
type(scope): imperative summary
<blank line>
- bullet describing key change
- bullet describing key change
- bullet describing key change
<blank line>
Short purpose paragraph explaining why this change was made.
```

Rules:

- `type` is one of: `feat`, `fix`, `refactor`, `docs`, `test`, `chore` (use the
  closest fit; only introduce another type if none of these apply and say why).
- `scope` is required — short and specific, comma-separated if the change
  genuinely spans multiple areas (e.g. `licensing,cli`).
- Subject line: imperative mood ("add", not "added"/"adds"), no trailing period.
- Body is **always required**, even for small or merge commits: a bullet list of
  the key changes, blank line, then a short paragraph on the purpose/motivation.
- Breaking changes: append `!` after the type/scope (e.g. `feat(api)!:`) and/or
  add a footer `BREAKING CHANGE: <description>`.

Example:

```text
refactor(licensing,cli): add periodic checks
- Add licensing runner with hourly checks
- Terminate serve after prolonged verification failures
- Remove duplicate rag entrypoint

This keeps license enforcement active and simplifies binaries.
```

## Which convention to follow

The format above is the default. Project instructions take precedence, and you
do not need to go looking for them: your harness has already loaded the
repository's rules, whatever mechanism it uses for that — read them from your
context instead of hunting for a particular file name. If they say anything
about commit messages, follow that.

The second source is the repository itself: `git log --oneline -10`. If recent
subjects use another style — ticket prefixes, other types, no body — match it.
Precedence: project instructions, then recent history, then the default above.

## Modes

- **Plan mode (read-only).** Produce the message and stop. Do not stage, commit
  or push — including when the argument is `push`. The message is the
  deliverable: the user reviews it, switches to build mode and commits there.
- **Build mode.** Produce the message, print it, then commit (and push, if
  asked). The printed message records what was committed; it is not an approval
  gate.
- **Unclear mode.** Default to plan behaviour and say so.

## Steps

1. **Make sure there is something to describe.**
   - Build mode: if `git diff --staged` is empty, run `git add -A` and continue
     — do not ask. If the index is still empty, stop: there is nothing to
     commit.
   - Plan mode: never stage. Use `git diff --staged` when it is non-empty;
     otherwise `git diff` plus `git status --short` for untracked files, and
     state in the output that nothing is staged.
2. **Inspect the changes.** Run `git diff --staged` (or the working-tree diff in
   plan mode). If the output is very large (e.g. >500 lines or >10,000 chars),
   run `git diff --staged --stat` first, then inspect the actual patch only for
   the most important/representative files. **Commit exactly what is staged —
   never add unstaged or untracked files once the index has content.**
3. **Determine intent and type**: read the diff to understand what actually
   changed and why, not just which lines moved. Pick the single closest `type`
   from the list above.
4. **Determine scope**:
   - Derive scope from the file paths in the diff (top-level directory names,
     package names, etc.). Reuse existing scope names when a clear match exists.
   - If changes clearly belong to one area, use that scope.
   - If changes span multiple distinct areas and it is not obvious which scopes
     to use, list the areas you see and ask the user to confirm/clarify the
     scope(s) before generating the final message.
5. **Draft the message** following the format above. Keep bullets factual and
   specific to the diff (no filler bullets). Keep the purpose paragraph to 1–2
   sentences. If the diff is large, write 3–5 concise bullets covering the main
   changes (do not try to list every hunk).
6. **Output the message** to the user in a code block, exactly as it should be
   committed.
7. **Stop in plan mode.** Nothing is staged, committed or pushed. Everything
   below runs in build mode only.
8. **Commit** using the command in [Commit command](#commit-command).
9. **Push** if the argument was `push`, following [Push](#push).

## Commit command

```bash
git commit -F - <<'EOF'
type(scope): imperative summary

- bullet one
- bullet two

Purpose paragraph.
EOF
```

The heredoc produces the required structure: summary, one blank line, the
bullets with no blank lines between them, one blank line, the purpose paragraph.
Quoting the delimiter as `'EOF'` stops the shell from interpolating backticks
and `$`. Do not start a bullet with `#` — git's message cleanup may treat it as
a comment.

## Push

Push is on the current branch:

- Detached HEAD (`git branch --show-current` prints nothing): stop and ask the
  user to check out a branch.
- No `origin` remote (`git remote get-url origin` fails): stop and report it.
- First push: `git push -u origin HEAD`. Afterwards `git push origin HEAD`.
- Non-fast-forward rejection: stop and report it. Never force-push.

## Failure handling

- A hook rejects the commit: show its output, fix the problem, create a **new**
  commit. Never `--amend`, never `--no-verify`.
- Nothing left to commit after staging: stop instead of creating an empty
  commit.

## Notes

- **Only staged changes are committed.** Once the index has content, never run
  `git add` to bring additional unstaged or untracked files into the commit. The
  single exception is the build-mode step above, which applies only when nothing
  is staged at all.
- Don't pad the bullet list with trivial or auto-generated noise
  (formatting-only diffs, lockfile bumps) unless that's the entire change.
- If the diff is large/spans many files, group bullets by logical change rather
  than by file.
- If staged changes look like a merge (`git diff --staged` after a merge, or
  `MERGE_MSG` present), the same body requirements still apply — summarize what
  the merge actually brings in, not just "merge branch X".
