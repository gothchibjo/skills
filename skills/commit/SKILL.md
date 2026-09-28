---
name: commit
description: "Generate a Conventional Commits-style commit message (type(scope): summary, body with bullets, purpose paragraph) from the currently staged git changes, matching the project's commit style. Use when the user asks to write/generate/draft a commit message, asks what to write for a commit, says \"commit this\"/\"commit these changes\", or wants help committing staged work. Also use if the user asks to review or fix an existing commit message against the project's conventions. Behaviour: commit only what is already staged and never add extra files; only if nothing is staged at all, auto-stage everything with `git add -A` before proceeding. Arguments: no args — prepare and commit; push — prepare, commit, and push."
metadata:
  protected: "false"
  copyright: "© 2026 gothchibjo"
---

# Generate Commit Message

Generates a commit message for the currently staged changes, following this
project's commit convention, then handles the commit (and optional push)
according to the arguments passed (no args, `push`).

## Commit message format

```
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

## Steps

1. **Inspect staged changes**: run `git diff --staged`. If the output is very
   large (e.g. >500 lines or >10,000 chars), first run
   `git diff --staged --stat` to see a compact file list, then inspect the
   actual patch only for the most important/representative files. **Commit
   exactly what is staged — never add unstaged/untracked files to the staged
   set.** If nothing is staged at all (empty `git diff --staged`), auto-stage
   everything with `git add -A` and proceed — don't ask or stop. Otherwise work
   only with the already-staged changes and leave all other unstaged/untracked
   files untouched.
2. **Determine intent and type**: read the diff to understand what actually
   changed and why, not just which lines moved. Pick the single closest `type`
   from the list above.
3. **Determine scope**:
   - Derive scope from the file paths in the staged diff (top-level directory
     names, package names, etc.). Reuse existing scope names when a clear match
     exists.
   - If changes clearly belong to one area, use that scope.
   - If changes span multiple distinct areas and it is not obvious which scopes
     to use, list the areas you see and ask the user to confirm/clarify the
     scope(s) before generating the final message.
4. **Draft the message** following the format above. Keep bullets factual and
   specific to the diff (no filler bullets). Keep the purpose paragraph to 1–2
   sentences. If the diff is large, write 3–5 concise bullets covering the main
   changes (do not try to list every hunk).
5. **Output the message** to the user in a code block, exactly as it should be
   committed.
6. **Commit** (and push if requested):
   - All bullets go in a **single** `-m` (use actual newlines between them). The
     `-m` flags produce three paragraphs: summary, bullets, purpose. Do **not**
     use one `-m` per bullet — that creates unwanted blank lines between them.
   - Commit command template:
     ```bash
     git commit -m "type(scope): imperative summary" -m "- bullet one
     - bullet two" -m "Purpose paragraph."
     ```
   - Behavior by arguments:
     - **no args**: run the commit immediately.
     - **`push`** (e.g. `/commit push`): run commit and push immediately.
   - Push is on the current branch: `git push origin HEAD`.

## Notes

- **Only staged changes are committed.** Never run `git add` to bring additional
  unstaged or untracked files into the commit. The single exception is the
  explicit auto-stage step above, which applies only when nothing is staged at
  all.
- Don't pad the bullet list with trivial or auto-generated noise
  (formatting-only diffs, lockfile bumps) unless that's the entire change.
- If the diff is large/spans many files, group bullets by logical change rather
  than by file.
- If staged changes look like a merge (`git diff --staged` after a merge, or
  `MERGE_MSG` present), the same body requirements still apply — summarize what
  the merge actually brings in, not just "merge branch X".
