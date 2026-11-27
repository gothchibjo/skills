---
name: merge-request
description: "Create a merge/pull request from the current branch into the remote default branch, or into the branch given as an argument, with a Conventional Commits-style title and a description covering the branch's changes. Use only when the user explicitly runs /mr or asks to open a merge request — never automatically. Arguments: no args — default branch; <branch> — that branch. In plan mode only the draft is produced."
disable-model-invocation: true
metadata:
  protected: "true"
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

## Which convention to follow

The format above is the default. Project instructions take precedence, and you
do not need to go looking for them: your harness has already loaded the
repository's rules, whatever mechanism it uses for that — read them from your
context instead of hunting for a particular file name. If they say anything
about merge request titles or descriptions, follow that.

Next comes a description template. If the host's conventional template exists
(GitHub: `.github/pull_request_template.md` or a
`.github/PULL_REQUEST_TEMPLATE/` directory; GitLab:
`.gitlab/merge_request_templates/`), read it and follow its headings for the
description, keeping the title in the convention above.

The third source is the repository itself: `git log --oneline -10`. If recent
titles use another style — ticket prefixes, other types — match it.

Precedence: project instructions, then the template, then recent history, then
the default above.

## Modes

- **Plan mode (read-only).** Draft the title and description, show the commit
  list and the exact commands that would run, then stop. Do not fetch, push, or
  run a CLI command that creates or updates anything, and do not touch the index
  or history. `git fetch` writes remote-tracking refs, so plan mode skips it and
  uses the existing `origin/<target>`; if that ref is missing, stop and say the
  target has never been fetched.
- **Build mode.** After the confirmation, push the branch and create or update
  the request.
- **Unclear mode.** Default to plan behaviour and say so.

## Steps

1. **Preconditions.** Do all of this before asking for confirmation, so the user
   is never asked to approve a plan that cannot run.
   - Confirm a git work tree: `git rev-parse --is-inside-work-tree`.
   - Get the current branch: `git branch --show-current`. If it is empty
     (detached HEAD), stop and ask the user to check out a branch.
   - Get the remote URL: `git remote get-url origin`. Stop if there is no
     `origin` remote.
   - Parse that URL into `host`, `owner` and `repo`:
     - drop the scheme (`https://`, `ssh://`, `git://`);
     - drop `user@`;
     - split on the first `:` when no `/` comes before it (scp form
       `git@host:owner/repo.git`), otherwise on the first `/`;
     - strip a trailing `.git`. Everything after the host is the repository
       path: `owner/repo`, or `group/subgroup/project` on GitLab. The web
       fallback below runs without a CLI, so this parsing is the only source of
       the URL it prints.
   - Resolve the target branch:
     - With an argument, use it as-is.
     - Without an argument, read the remote default:
       `git symbolic-ref --short refs/remotes/origin/HEAD`. If that fails, run
       `git ls-remote --symref origin HEAD` and take the name from the line
       ending in `HEAD` (`ref: refs/heads/<name>`).
     - Normalise once: strip `refs/heads/`, then `origin/`. Stop if the result
       is empty.
   - Stop if the current branch is the target branch.
   - Build mode: `git fetch origin <target>` and verify `origin/<target>`
     exists. Stop with the fetch error if it does not. Plan mode: skip the fetch
     and use the existing ref, as described in [Modes](#modes).
   - Run `git merge-base origin/<target> HEAD`. If it fails, stop and tell the
     user the histories are unrelated — a three-dot diff means nothing there.
   - If `git rev-list --count origin/<target>..HEAD` is `0`, stop: there are no
     commits to put in a merge request.
   - If the working tree is dirty (`git status --porcelain` is non-empty), warn
     that uncommitted changes will not be part of the merge request, but do not
     stage or commit anything.
   - Choose the publishing path by the authenticated CLI, not by the host name:
     - `command -v gh` and `gh auth status --hostname "$host"` succeed — GitHub;
     - otherwise `command -v glab` and `glab auth status --hostname "$host"`
       succeed — GitLab;
     - otherwise the web fallback. `gh auth status` exits non-zero when the host
       is not authenticated, which is what makes this work for self-hosted
       instances whose hostname says nothing about the product. Remember the
       chosen path for the confirmation.

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
   - Write the title and description following the format above, or the template
     structure when "Which convention to follow" says so. Keep bullets factual
     and specific; do not pad with formatting-only or lockfile changes unless
     that is the entire change.
   - Cover the whole branch: map the substantive commits to bullets. If one
     logical change spans several commits, merge them into one bullet.

4. **Duplicate check and confirmation.**
   - Only when the branch is already on the remote
     (`git ls-remote --exit-code --heads origin "$branch"`): a branch that was
     never pushed cannot have a request, so skip the check otherwise.

     ```bash
     gh pr list --head "$branch" --base "$target" --state open \
       --json number,url,title
     ```

     ```bash
     glab mr list --source-branch "$branch" --target-branch "$target" \
       --output json
     ```

   - Show the user:
     - source and target: `<current> -> origin/<target>`;
     - the list of commits going into the merge request;
     - the prepared title and description in a code block;
     - the publishing path chosen in step 1;
     - the existing request and its URL, if one was found.
   - Ask whether to push the branch and create (or update) the merge request.
     **Do not push and do not create anything until the user explicitly
     confirms.** On no, stop and leave the work tree untouched.
   - If an open request already exists for this source/target pair, offer to
     update its title and description instead of creating a duplicate. Only ever
     update those two fields, and never force-push.

5. **Push the branch** (build mode, after confirmation only).
   - If the branch has no upstream
     (`git rev-parse --abbrev-ref --symbolic-full-name '@{u}'` fails), push and
     set it: `git push -u origin HEAD`. Otherwise use `git push origin HEAD` and
     leave the existing upstream alone.
   - A non-fast-forward rejection means the remote branch has diverged: stop and
     report it. **Never force-push.**

6. **Create or update the request** (build mode). Use the commands in
   [Commands](#commands) for the path chosen in step 1.
   - Create path: run the create command and print the URL it outputs.
   - Update path: `gh pr edit` and `glab mr update` print nothing, so print the
     URL found in step 4. `$number` is the `number` field on GitHub and the
     `iid` field on GitLab.
   - **Web fallback** (no CLI is installed or authenticated for the host): do
     not invent an API call. Print the prepared title/description and the host's
     compare page — GitHub
     `https://<host>/<owner>/<repo>/compare/<target>...<branch>`, GitLab
     `https://<host>/<owner>/<repo>/-/compare/<target>...<branch>` — then name
     the CLI for that host (`gh` for GitHub, `glab` for GitLab) and, if it is
     installed but not authenticated, the login command with `--hostname`.
   - Do not open the browser.

## Commands

GitHub:

```bash
gh pr create --base "$target" --head "$branch" --title "$title" --body-file - <<'EOF'
<description>
EOF
```

```bash
gh pr edit "$number" --title "$title" --body-file - <<'EOF'
<description>
EOF
```

GitLab:

```bash
glab mr create --source-branch "$branch" --target-branch "$target" \
  --title "$title" --description-file - --yes <<'EOF'
<description>
EOF
```

```bash
glab mr update "$number" --title "$title" --description-file - --yes <<'EOF'
<description>
EOF
```

The quoted `'EOF'` is what stops the shell from interpolating backticks, `$` and
`!` in the description — the same reason the [commit] skill uses a heredoc.
Replace `<description>` with the drafted body and keep its blank lines. `--yes`
only skips the CLI's own prompt; the user's confirmation has already happened.

## Failure handling

- Push error or non-fast-forward rejection: report it, stop, do not retry.
- Not authenticated, or the CLI is missing: caught in step 1, before the push.
  Fall back to the web path rather than pushing and failing afterwards.
- Create or update fails after the push: report that the branch was pushed, the
  exact CLI error and the compare URL, then stop. Do not retry blindly, do not
  create a second request, do not switch to `--fill`.
- Never `--amend`, never `--no-verify`, never change the index or history.

## Notes

- **The merge request includes committed branch changes only.** Never run
  `git add`, `git commit`, `git stash`, `git reset`, `git restore`,
  `git restore --staged`, `git rm --cached` or `git checkout -- <path>`.
  Uncommitted work stays out of the request and is only mentioned as a warning.
- A fork workflow, where `origin` is your fork, is out of scope: this skill
  targets the repository `origin` points at. `--head <user>:<branch>` is the
  escape hatch if the user needs it.
- Never push to the target branch, never force-push, never delete branches.
- Never create a merge request that has no commits behind it, or one whose
  source and target are the same branch.
- Use the remote default branch by default; the argument only changes the
  target, never the source.
- If the branch contains merge commits, summarise what they actually bring in,
  not just "merge branch X".
- Do not run `gh pr merge` / `glab mr merge`; opening or updating the request is
  the end of this skill.

<!-- refs -->

[commit]: ../commit/SKILL.md
