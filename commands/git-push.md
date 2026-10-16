---
description: List the changed files for the user to confirm, then commit, rebase to sync with the remote, and push safely; on a sync conflict, roll back and print a Chinese report
---

# /git-push Command

## Trigger

**Runs only when the user explicitly invokes `/git-push`.** When the user has not invoked it explicitly, you must not run this command on your own initiative — even if the context makes it look like "time to commit and push".

Invoking the command is the authorization for this commit and push; the file-list confirmation in step 2 is still required before committing.

## Hard Prohibitions

Under no circumstances execute:

- `git push --force` / `--force-with-lease` / `--force-if-includes`
- A refspec with a `+` prefix (such as `git push origin +main`)
- `git add .` / `git add -A` — the user-confirmed files must be staged path by path
- `git merge` or `git pull` (without `--rebase`) — this command syncs with the remote using rebase only
- Editing conflict files yourself after a rebase conflict and running `git rebase --continue`, or using `--skip`
- Running `git add` / `git commit` before step 2's confirmation is complete
- Deleting a remote branch or remote tag

When the push is aborted, the only permitted action is to roll back, print the report, and stop.

## Execution Flow

### 1. Detect changes

```bash
git status --porcelain
```

No output → print "No code changes on the current branch; nothing to commit." and stop.

### 2. Confirm which files to commit

Turn each line of `git status --porcelain` into an entry in a list written into your reply, each as `<status> <file path>`, with the status rendered as: added / modified / deleted / untracked (will be added) / renamed.

- **Untracked files must be flagged individually**: temporary scripts, secrets, and build artifacts are the most likely things to be committed by mistake at this step.
- Suspected sensitive files (`.env`, `*.pem`, `id_rsa`, `credentials*`, and so on) are still listed, but marked "⚠️ possible secret" and placed last.
- Add a one-line summary: `N files, +X / −Y lines` (line counts from `git diff --stat HEAD`, excluding untracked files).
- Do not paste the full diff unprompted; show it only when the user asks in the moment.

`AskUserQuestion` accepts only **2–4 options** per call, so **never generate one option per file**. Confirm with a two-way choice:

- Question text: "Are these N files the content to be committed?"
- Options: `[Commit all]` / `[Cancel]`

When the user needs to commit only a subset, they will use the tool's built-in Other input. Go by the paths the user provides; if Other mentions a file outside the list, verify it actually has changes before including it — otherwise ignore it and say so.

Branch handling:

- **Commit all** → proceed to step 3.
- **Cancel**, or the user says the list is wrong → **abort the entire command**. Run no `git add` / `git commit` / `git push`, output "Cancelled. Nothing was committed.", and hand control back to the user.
- **No response, or interaction unavailable** → abort (fail closed). **Never default to committing everything, and never default to continuing.**

Files that were not confirmed stay in the working tree untouched — **never discard them** — and tell the user in step 3's output how many remain uncommitted.

If `AskUserQuestion` is unavailable or the call fails → **do not degrade into "decide for yourself and commit"**. Instead, write the full candidate list and its statuses in your reply, explicitly ask the user to confirm, and continue only after they respond.

### 3. Commit

Generate a Chinese commit message per the `/git-commit` command's specification, then stage **path by path** according to the confirmation (separate with `--` and quote each, to tolerate spaces, CJK characters, and shell metacharacters; pass renamed entries by their new path):

```bash
git add -- "<path1>" "<path2>"
git commit -m "<message>"
```

Output "Committed N files; M files remain uncommitted in the working tree." (N is the confirmed count, M the unconfirmed count.)

Commit first, sync second — that order is mandatory, because `pull --rebase` requires a clean working tree. If step 2 confirmed only a subset, tracked files may still be modified after the commit; step 6 accounts for that.

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

- **Produces output** (step 2 confirmed only a subset, so tracked files remain uncommitted) → **abort**, do not rebase, print the report, and note: uncommitted tracked files will block the rebase — handle them (commit or stash) and re-run this command.
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
| Step 2 confirmation cancelled | Abort; run no git write operation |
| Step 2 no response / interaction unavailable | Abort (fail closed); never default to committing everything |
| Only some files confirmed, tracked files remain uncommitted, and `behind > 0` | Abort; tell the user to handle it and re-run |
| Only untracked files remain uncommitted, and `behind > 0` | Rebase as usual (untracked files do not block it); untracked files are preserved |
| Uncommitted files remain, but `behind == 0` | Push as usual |
| detached HEAD | Abort and tell the user to switch branches first |
| No upstream associated | Tell the user to run `git push -u`, then stop |
| Upstream configured but the remote branch was deleted (`@{u}` fails to resolve) | Same as above; treat as no upstream |
| Rebase conflict | Record conflicted files first → `git rebase --abort` → abort report |
| Rebase succeeded but the branch fell behind again before the push | Abort and report; do not loop and retry |
| `git push` rejected (non-fast-forward) | Abort and report; never switch to `--force` |
| A rebase / merge is already in progress | Abort and tell the user to finish or abort the current operation first |

## Danger Signals — Stop and Print the Report the Moment They Appear

- Wanting to use `--force` / `--force-with-lease` to "just overwrite the remote"
- Treating "the user ran /git-push" as a reason to skip step 2's confirmation
- Using `git add .` instead of staging the confirmed files path by path, for convenience
- Changing the confirmation default to "commit all" so the user has to click less
- Running `git add` / `git commit` before the user has confirmed, "to save a round trip"
- Omitting untracked files or shortening the list to make it look tidy
- Working around "uncommitted tracked files block the rebase" by any means other than `--force`, such as stashing or discarding changes on your own
- Wanting to resolve a rebase conflict yourself and run `--continue`, treating "make the command finish" as the goal
- Repeating the rebase until it succeeds, ignoring one interruption after another
- Thinking "the user is in a hurry, push it first and sort it out later"
