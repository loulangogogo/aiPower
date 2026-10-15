---
name: git-commit
description: Analyze the current working-tree changes, generate an English commit message following the Conventional Commits specification, and complete the commit; triggered by the user running /git-commit
disable-model-invocation: true
---

# Git Commit Command

This skill runs only when the user explicitly invokes `/git-commit`. It is never invoked proactively by the model. Once invoked, execute the steps below in order — do not skip steps, and do not commit from memory.

## Execution Flow

### 1. Sync remote updates

First confirm you are inside a Git repository, then pull upstream updates for the current branch. **If the current branch does not exist on the remote, skip the pull** and go straight to the next step.

```bash
git rev-parse --is-inside-work-tree    # fails → report "not a Git repository" and stop
git fetch --prune                      # fails (no remote / unreachable) → abort immediately

if upstream=$(git rev-parse --abbrev-ref --symbolic-full-name '@{u}' 2>/dev/null); then
  git pull                             # uses the default merge strategy
else
  echo "Branch does not exist on the remote; skipping pull"
fi
```

- Two cases count as "the branch does not exist on the remote": the branch has no upstream configured; or its upstream was deleted on the remote (`git branch -vv` shows `gone`). You must run `git fetch --prune` before making this determination — otherwise stale remote-tracking refs left locally will distort the answer.
- `git pull` uses the default merge strategy, so being behind the remote introduces a merge commit. That is expected behavior.
- Pull failure (unreachable network, local changes would be overwritten, merge conflict) → **abort the entire commit flow immediately**, relay Git's raw output to the user, and perform no automatic recovery. If the failure leaves you mid-merge, only tell the user they may run `git merge --abort` themselves to return to the pre-pull state — do not do it for them.

### 2. Check changes and intermediate state

```bash
git status --porcelain
git diff --check
gitdir=$(git rev-parse --git-dir); ls "$gitdir"/MERGE_HEAD "$gitdir"/CHERRY_PICK_HEAD "$gitdir"/REVERT_HEAD "$gitdir"/rebase-merge "$gitdir"/rebase-apply 2>/dev/null
```

- `git status --porcelain` produces no output → report "No code changes to commit." and stop.
- `git diff --check` reports conflict markers → list the affected files and stop.
- The last command produces output → you are in the middle of a merge / rebase / cherry-pick / revert. Stop immediately and tell the user to finish or abort that operation first. Committing mid-operation disguises the conflict resolution as an ordinary commit.

### 3. Read the diff and determine the commit scope

```bash
git diff           # unstaged changes
git diff --cached  # staged changes
```

Go through the files one by one and decide which represent the logical change being committed, and which are debugging leftovers, temporary files, or changes unrelated to this task.

**The commit scope is the user's decision, not yours**: call `ask_user_question` first so the user can tick the files to commit, and stage only after you have their answer.

- One option per file, `multi_select: true`, with the question text stating the total number of candidate files.
- The `description` gives the change status and a one-line summary, for example "modified · adds a file-confirmation dialog to the commit flow".
- Files directly related to the current task come first; suspected sensitive files (`.env`, `*.pem`, `id_rsa`, `credentials*`, and so on) are still listed, but marked "⚠️ possible secret" in the `description` and placed last.
- When there are more than 15 changed files, group them by top-level directory instead, to keep the dialog from growing too long.
- In a multi-select question, `custom` in the answer is a **supplement**, not an override: merge it with `selected`. If `custom` mentions a file outside the candidate list, verify it actually has changes before including it — otherwise ignore it and say so.
- Empty `selected` with no `custom` → treat it as abandoning this commit; report and stop.

Stage the confirmed files one by one:

```bash
git add <file1> <file2> ...
```

Do not use `git add .`, `git add -A`, or `git commit -a`: they **bypass the confirmation you just obtained** and write unrelated changes, temporary files, and secrets into history.

If the host has no `ask_user_question` tool, or the call fails (it returns `DELEGATED_CALLER` when running inside a subagent) → **do not degrade into "decide for yourself and commit"**. Instead, write the full candidate list and its statuses in your reply, explicitly ask the user to confirm, and continue only after they respond.

### 4. Confirm the staging area is non-empty

```bash
git diff --cached --stat
```

No output means step 3 staged nothing. Stop; do not create an empty commit.

### 5. Generate the commit message

Generate it per the "Commit Message Format" section, and show the final message to the user before committing as the record of this change (the files to be committed were already confirmed in step 3).

### 6. Verify committer identity and commit

Git rejects the commit or records the wrong author when identity is missing, so confirm first:

```bash
git config user.name
git config user.email
```

If either is empty → report the `git config user.name "..."` and `git config user.email "..."` commands needed, then stop.

```bash
# Title only
git commit -m "<type>(<scope>): <subject>"

# For a single-paragraph body, each -m produces a separate paragraph
git commit -m "<type>(<scope>): <subject>" -m "<body>" -m "<footer>"

# For a body containing multi-line lists, use a heredoc to avoid newline escaping problems in -m
git commit -F - <<'EOF'
<type>(<scope>): <subject>

<body>

<footer>
EOF
```

If a hook fails or the commit errors: **leave the staging area exactly as it is**, relay Git's output verbatim, and do not bypass with `--no-verify` or undo the user's staging with `git reset`.

### 7. Report the result

On success, output:

```text
Committed <short hash>: <type>(<scope>): <subject>
Changes: N added / M modified / K deleted
```

## Commit Message Format

```text
<type>(<scope>): <subject>

<body>

<footer>
```

- `scope` is **optional** and denotes the affected area (such as `auth`, `api`, `ui`). Omit the parentheses when there is no clear scope: `feat: add user login module`.
- `subject` is written in English, states in one line what was done, stays under 50 characters, and takes no trailing period.
- `body` explains *why* the change was made and *what* changed; it may be a bulleted list. Omit the entire section when there is no body.
- `footer` carries references such as `Closes #123` or `Refs #45`. Omit the entire section when there is none.
- Breaking changes: add `!` after the type or scope (for example `feat(api)!: remove v1 endpoints`), and write `BREAKING CHANGE: <impact>` in the footer.
- Revert commits: title it `revert: feat(auth): add user login verification`, and write `This reverts commit <hash>` in the body.

Example:

```text
feat(auth): add user login verification

Implement JWT-based user authentication:
- Add login endpoint
- Token generation and validation
- Expiry management

Closes #123
```

## Supported Types

| Type       | Meaning                                                    |
| ---------- | ---------------------------------------------------------- |
| `feat`     | A new feature                                              |
| `fix`      | A bug fix                                                  |
| `docs`     | Documentation changes                                      |
| `style`    | Code formatting (no effect on behavior)                    |
| `refactor` | Refactoring (neither a feature nor a bug fix)              |
| `perf`     | Performance improvement                                    |
| `test`     | Test-related changes                                       |
| `chore`    | Build process or auxiliary tooling changes                 |
| `build`    | Build system or external dependency changes                |
| `ci`       | CI configuration changes                                   |
| `revert`   | Reverting a previous commit                                |

The [Conventional Commits](https://www.conventionalcommits.org/) specification itself defines only the semantics of `feat` (new feature) and `fix` (bug fix); the remaining types follow the Angular convention and may be extended as a project requires.

## Prohibitions

- No bulk staging with `git add .`, `git add -A`, or `git commit -a`
- No bypassing hooks with `--no-verify`, and no rewriting existing commits with `--amend`
- No committing while in a merge / rebase / cherry-pick intermediate state
- No committing unconfirmed sensitive files or unrelated changes
- After a failed commit, do not undo the user's staging with `git reset`
- After a failed pull, perform no automatic recovery (`git merge --abort`, `git reset`, `git stash`, and so on) — only report and suggest the command
