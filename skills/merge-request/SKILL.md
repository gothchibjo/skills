---
name: merge-request
description: "Create a merge/pull request from the current branch into the remote default branch, or into the branch given as an argument, with a Conventional Commits-style title and a description summarising all or the key changes from the branch's commits. Use ONLY when the user explicitly runs /mr or explicitly asks to open a merge request. NEVER invoke automatically, never as part of another task, never on your own initiative. Arguments: no args — target the remote default branch; <branch> — target that branch."
disable-model-invocation: true
metadata:
  protected: "false"
  copyright: "© 2026 github.com/gothchibjo"
---

# Create Merge Request

Opens a merge request (GitHub: pull request) from the current branch into the
remote default branch, or into the branch named by the argument. The title and
description follow the same convention as the [commit] skill and cover all or
the key changes brought by the branch's commits.

## Merge request format

```text
type(scope): imperative summary
<blank line>
- bullet describing key change
- bullet describing key change
- bullet describing key change
<blank line>
Short purpose paragraph explaining why this branch exists.
```

Rules:

- `type` is one of: `feat`, `fix`, `refactor`, `docs`, `test`, `chore` (use the
  closest fit; only introduce another type if none of these apply and say why).
- `scope` is required — short and specific, comma-separated if the branch
  genuinely spans multiple areas (e.g. `licensing,cli`).
- Title: imperative mood ("add", not "added"/"adds"), no trailing period.
- Description body is **always required**: a bullet list of the key changes,
  blank line, then a short paragraph on the purpose/motivation.
- Breaking changes: append `!` after the type/scope (e.g. `feat(api)!:`) and
  mention the break in the description.
- The description summarises the **whole branch diff**, not the last commit.
  Every substantive commit must be reflected in at least one bullet; group
  related commits into one bullet rather than listing commits one by one.

Example:

```text
feat(billing): add invoice export
- Add CSV and PDF invoice export endpoints
- Queue large exports and expose job status
- Cover export permissions with tests

Customers need invoices outside the dashboard for accounting.
```

## Steps

1. **Preconditions.**
   - Confirm a git work tree: `git rev-parse --is-inside-work-tree`.
   - Get the current branch: `git branch --show-current`. If it is empty
     (detached HEAD), stop and ask the user to check out a branch.
   - Get the remote URL: `git remote get-url origin`. Stop if there is no
     `origin` remote.
   - Resolve the target branch:
     - With an argument, use it as-is after stripping an `origin/` prefix.
     - Without an argument, read the remote default:
       `git symbolic-ref --short refs/remotes/origin/HEAD` and strip `origin/`.
       If that fails, fall back to `git ls-remote --symref origin HEAD`.
     - Normalise away `refs/heads/` and `origin/`. Stop if the target cannot be
       resolved.
   - Stop if the current branch is the target branch.
   - Fetch the target: `git fetch origin <target>` and verify `origin/<target>`
     exists. Stop with the fetch error if it does not.
   - If `git rev-list --count origin/<target>..HEAD` is `0`, stop: there are no
     commits to put in a merge request.
   - If the working tree is dirty (`git status --porcelain` is non-empty), warn
     that uncommitted changes will not be part of the merge request, but do not
     stage or commit anything.

2. **Collect the branch changes.**
   - Read the commits with bodies:
     `git log --format='--- %h%n%s%n%b' origin/<target>..HEAD`.
   - Read the diff against the merge base:
     `git diff --stat origin/<target>...HEAD`, then
     `git diff origin/<target>...HEAD`.
   - If the diff is large (roughly >500 lines or >10,000 chars), work from the
     stat and inspect the patch only for the most important/representative
     files.

3. **Draft the title and description.**
   - Determine intent and the single closest `type` from the commits and the
     diff, not just which lines moved.
   - Derive `scope` from the changed paths, reusing existing scope names when a
     clear match exists. If the branch spans distinct areas and the scope is not
     obvious, list the areas and ask the user before proceeding.
   - Write the title and description following the format above. Keep bullets
     factual and specific; do not pad with formatting-only or lockfile changes
     unless that is the entire change.
   - Cover the whole branch: map the substantive commits to bullets. If one
     logical change spans several commits, merge them into one bullet.

4. **Ask for confirmation.**
   - Show the user:
     - source and target: `<current> -> origin/<target>`;
     - the list of commits going into the merge request;
     - the prepared title and description in a code block.
   - Ask whether to push the branch and create the merge request. **Do not push
     and do not create anything until the user explicitly confirms.** On no,
     stop and leave the work tree untouched.
   - If an open merge request already exists for this source/target pair, show
     its URL, ask whether to update it instead of creating a duplicate, and only
     update the title/description — never force-push.

5. **Push the branch** (after confirmation only).
   - Push the current branch and set its upstream: `git push -u origin HEAD`.
   - A non-fast-forward rejection means the remote branch has diverged: stop and
     report it. **Never force-push.**

6. **Create the merge request.**
   - Detect the host from the remote URL:
     - `github.com` or a host containing `github` — GitHub.
     - a host containing `gitlab` — GitLab.
     - Anything else — unknown.
   - **GitHub:** check `gh auth status`, then:

     ```bash
     gh pr create --base "$target" --head "$branch" \
       --title "$title" --body "$body"
     ```

   - **GitLab:** check `glab auth status`, then:

     ```bash
     glab mr create --source-branch "$branch" --target-branch "$target" \
       --title "$title" --description "$body" --yes
     ```

   - **Unknown host or missing CLI:** do not invent an API call. Print the
     prepared title/description and a ready-to-open web URL for the host (e.g.
     GitHub's `https://<host>/<owner>/<repo>/compare/<target>...<branch>`), and
     tell the user which CLI to install.
   - Print the resulting merge request URL. Do not open the browser.

## Notes

- **The merge request includes committed branch changes only.** Never run
  `git add`, `git commit`, `git stash` or `git reset`. Uncommitted work stays
  out of the request and is only mentioned as a warning.
- Never push to the target branch, never force-push, never delete branches.
- Never create a merge request that has no commits behind it, or one whose
  source and target are the same branch.
- Use the remote default branch by default; the argument only changes the
  target, never the source.
- If the branch contains merge commits, summarise what they actually bring in,
  not just "merge branch X".
- Do not run `gh pr merge` / `glab mr merge`; opening the request is the end of
  this skill.

<!-- refs -->

[commit]: ../commit/SKILL.md
