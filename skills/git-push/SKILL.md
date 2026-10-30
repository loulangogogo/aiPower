---
name: git-push
description: Use only when the user explicitly runs the /git-push command; never invoke proactively. Lists changed files for the user to tick, then commits, rebases to sync with the remote, and pushes safely; on a sync conflict, rolls back and prints a Chinese report.
disable-model-invocation: true
---

# /git-push Command

## Trigger

**Runs only when the user explicitly invokes `/git-push`.** When the user has not invoked it explicitly, you must not run this skill on your own initiative — even if the context makes it look like "time to commit and push".

Invoking the command is the authorization for this commit and push; step 2's file selection is still required before committing.

## Hard Prohibitions

Under no circumstances execute:

- `git push --force` / `--force-with-lease` / `--force-if-includes`
- A refspec with a `+` prefix (such as `git push origin +main`)
- `git add .` / `git add -A` — the files the user ticked must be staged path by path
- `git merge` or `git pull` (without `--rebase`) — this command syncs with the remote using rebase only
- Editing conflict files yourself after a rebase conflict and running `git rebase --continue`, or using `--skip`
- Running `git add` / `git commit` before step 2's selection is complete
- Deleting a remote branch or remote tag

When the push is aborted, the only permitted action is to roll back, print the report, and stop.

## Execution Flow

### 1. Detect changes

```bash
git status --porcelain
```

No output → print "No code changes on the current branch; nothing to commit." and stop.

### 2. Tick the files to commit

Turn each line of `git status --porcelain` into a multi-select option, each as `<status> <file path>`, with the status rendered as: added / modified / deleted / untracked (will be added) / renamed. Use the question widget's **multi-select** capability to let the user tick them.

- **All options start unticked**, and the user actively ticks the files to commit. That multi-select widget does not support pre-selection — the initial state is always empty — so never claim that some files are selected by default.
- **Untracked files must be flagged individually**: temporary scripts, secrets, and build artifacts are the most likely things to be committed by mistake at this step.
- Add a one-line summary: `N files, +X / −Y lines` (line counts from `git diff --stat HEAD`, excluding untracked files).
- Do not paste the full diff unprompted; show it only when the user asks in the moment.

Branch handling:

- **One or more files ticked** → proceed to step 3, committing only those.
- **Nothing ticked** → abort, print "No files selected; cancelled.", and run no `git add` / `git commit` / `git push`.
- **No response, or multi-select unavailable** → abort (fail closed). **Never degrade into "commit everything by default", and never default to continuing.**

**Unticked files stay in the working tree untouched — never discard them** — and tell the user in step 3's output how many remain uncommitted.

### 3. Commit

Generate a Chinese commit message per the `git-commit` skill's specification, then stage **path by path** according to the selection (separate with `--` and quote each, to tolerate spaces, CJK characters, and shell metacharacters; pass renamed entries by their new path):

```bash
git add -- "<path1>" "<path2>"
git commit -m "<message>"
```

Output "Committed N files; M files remain uncommitted in the working tree." (N is the ticked count, M the unticked count.)

Commit first, sync second — that order is mandatory, because `pull --rebase` requires a clean working tree. If step 2 ticked only a subset, tracked files may still be modified after the commit; step 6 accounts for that.

### 4. Upstream check

```bash
git symbolic-ref --short -q HEAD        # empty output means detached HEAD; abort
git rev-parse --abbrev-ref '@{u}'       # failure means no usable upstream; abort
```

If either check fails → output "The current branch has no usable remote association. Run `git push -u <remote> <branch>` first to establish one." and stop.

Once the checks pass, resolve the remote and branch names dynamically, and **use those values for every subsequent step — never hardcode `origin`**:

```bash
BRANCH=$(git symbolic-ref --short HEAD)
REMOTE=$(git config "branch.$BRANCH.remote")
```

### 5. Sync and compare

```bash
git fetch "$REMOTE"
git rev-list --count "HEAD..@{u}"    # behind: commits the remote is ahead by
git rev-list --count "@{u}..HEAD"    # ahead : commits the local branch is ahead by
```

### 6. Decide

**Scenario A: `behind == 0` (remote is not ahead; fast-forward is possible)**

```bash
git push
```

Output "Push succeeded! The branch including the new commit has been pushed to <remote branch name>."

Whether uncommitted files remain in the working tree does not affect the push and needs no handling.

If `git push` is rejected (non-fast-forward, meaning someone pushed during the sync window) → enter scenario B's abort flow. **Do not switch to `--force`.**

**Scenario B: `behind > 0` (the remote has new commits)**

Rebase requires no modifications to tracked files in the working tree, so check first:

```bash
git status --porcelain --untracked-files=no
```

- **Produces output** (step 2 ticked only a subset, so tracked files remain uncommitted) → **abort**, do not rebase, print the report, and note: uncommitted tracked files will block the rebase — handle them (commit or stash) and re-run this command.
- **No output** → run the automatic rebase sync:

```bash
git pull --rebase "$REMOTE" "$BRANCH"
```

  - **Succeeds** → re-run the step 5 comparison:
    - `behind == 0` → run `git push` and print scenario A's success message.
    - `behind > 0` (someone pushed again during the window) → abort and print the report. **Do not loop and retry.**
    - Untracked files do not block a rebase and are preserved as-is; the rebase rewrites the hashes of local unpushed commits, which is expected behavior.
  - **Fails (conflict)** → handle in exactly this order:
    1. `git diff --name-only --diff-filter=U` to record the conflicted files (**this must run before the abort** — after aborting, the working tree no longer carries the conflict information)
    2. `git rebase --abort` to roll back to the pre-rebase state
    3. Abort and print the report

  **Never** edit conflict files by hand and continue the rebase, **never** use `--skip`, and **never** switch to merge. Conflicts always go back to the user.

## Abort Report Template

```text
⚠️  Push aborted: <reason>

📊 Status:
   - Current branch: <BRANCH>
   - Remote branch: <REMOTE>/<BRANCH>
   - Local ahead of remote: <ahead> commits
   - Remote ahead of local: <behind> commits
   - Committed this run: <N> files
   - Still uncommitted in the working tree: <M> files
   - Auto-sync result: <not run (uncommitted tracked files present) / rebased successfully / conflict rolled back / push rejected>

📝 Conflicted files (listed on rebase conflict):
   <output of git diff --name-only --diff-filter=U>

💡 Suggested actions:
   The push was blocked to avoid overwriting commits others have made on the remote. Your local commits are intact.
   Choose how to proceed:
   [1] Handle the uncommitted tracked files in the working tree, then re-run this command (uncommitted tracked files block the rebase)
   [2] Run git pull --rebase <REMOTE> <BRANCH> manually, resolve the conflicts, then re-run this command
   [3] Cancel (handle it manually)
```

The options in the report are for the user to choose from only; **this command must not execute them on the user's behalf**.

## Edge Cases

| Situation | Handling |
| --- | --- |
| No changes in the working tree | Report and stop; do not push |
| Step 2: not a single file ticked | Abort; run no git write operation |
| Step 2: no response / multi-select unavailable | Abort (fail closed); never default to committing everything |
| Tracked files still uncommitted after the selection, and `behind > 0` | Abort; tell the user to handle it and re-run |
| Only untracked files left uncommitted after the selection, and `behind > 0` | Rebase as usual (untracked files do not block it); untracked files are preserved |
| Uncommitted files still present after the selection, but `behind == 0` | Push as usual |
| detached HEAD | Abort and tell the user to switch branches first |
| No upstream associated | Tell the user to run `git push -u`, then stop |
| Upstream configured but the remote branch was deleted (`@{u}` fails to resolve) | Same as above; treat as no upstream |
| Rebase conflict | Record conflicted files first → `git rebase --abort` → abort report |
| Rebase succeeded but the branch fell behind again before the push | Abort and report; do not loop and retry |
| `git push` rejected (non-fast-forward) | Abort and report; never switch to `--force` |
| A rebase / merge is already in progress | Abort and tell the user to finish or abort the current operation first |

## Danger Signals — Stop and Print the Report the Moment They Appear

- Wanting to use `--force` / `--force-with-lease` to "just overwrite the remote"
- Treating "the user ran /git-push" as a reason to skip step 2's selection
- Using `git add .` instead of staging the ticked files path by path, for convenience
- Describing options as "already selected by default" or "all selected by default" — that widget does not support pre-selection, and the initial state is always empty
- Running `git add` / `git commit` before the user has ticked anything, "to save a round trip"
- Omitting untracked files or shortening the list to make it look tidy
- Working around "uncommitted tracked files block the rebase" by any means other than `--force`, such as stashing or discarding changes on your own
- Wanting to resolve a rebase conflict yourself and run `--continue`, treating "make the command finish" as the goal
- Repeating the rebase until it succeeds, ignoring one interruption after another
- Thinking "the user is in a hurry, push it first and sort it out later"
