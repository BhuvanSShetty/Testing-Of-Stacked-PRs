# Stacked PRs Without `gh stack` — Complete Cheatsheet

## 1. Mental Model

A stacked PR can be managed using normal Git commands and GitHub PR base branches.

Example:

```text
main
  |
  +-- best        -> PR #5
       |
       +-- best1  -> PR #6
            |
            +-- best2 -> PR #7
```

Each PR depends on the branch below it:

```text
PR #5: best  -> main
PR #6: best1 -> best
PR #7: best2 -> best1
```

---

# 2. Create the First Feature Branch

Start from the latest main:

```bash
git checkout main
git pull origin main
```

Create your first branch:

```bash
git checkout -b best
```

Make changes:

```bash
git add .
git commit -m "Add initial best changes"
```

Push:

```bash
git push -u origin best
```

Create PR on GitHub:

```text
base:    main
compare: best
```

Result:

```text
main
  |
  +-- best -> PR #5
```

---

# 3. Create the Second Layer

IMPORTANT:

Checkout the first feature branch:

```bash
git checkout best
```

Create the second branch FROM `best`:

```bash
git checkout -b best1
```

Now:

```text
main
  |
  +-- best
       |
       +-- best1
```

Make only the second layer's changes:

```bash
git add .
git commit -m "Add best1 changes"
```

Push:

```bash
git push -u origin best1
```

---

# 4. Create PR #2 as a Stacked PR

On GitHub create a new PR.

Set:

```text
base:    best
compare: best1
```

NOT:

```text
base:    main
compare: best1
```

Correct:

```text
PR #5
best -> main

PR #6
best1 -> best
```

GitHub will recognize the dependency/stack.

---

# 5. What the PR Diffs Mean

PR #5:

```text
best -> main
```

shows:

```text
Changes introduced by best
```

PR #6:

```text
best1 -> best
```

shows:

```text
Changes introduced only by best1
```

This prevents PR #6 from showing all the changes from PR #5.

---

# 6. Add Another Layer

Start from the current top branch:

```bash
git checkout best1
```

Create:

```bash
git checkout -b best2
```

Make changes:

```bash
git add .
git commit -m "Add best2 changes"
```

Push:

```bash
git push -u origin best2
```

Create PR:

```text
base:    best1
compare: best2
```

Now:

```text
main
  |
  +-- best       -> PR #5
       |
       +-- best1 -> PR #6
            |
            +-- best2 -> PR #7
```

---

# 7. If Main Changes

Suppose someone adds new commits to main:

```text
main
  |
  +-- NEW MAIN COMMIT
  |
  +-- best
       |
       +-- best1
```

You need to update the stack manually.

The dependency order is:

```text
main
 ↓
best
 ↓
best1
 ↓
best2
```

Rebase from bottom to top.

---

# 8. Rebase the First Branch

First fetch the latest remote state:

```bash
git fetch origin
```

Checkout `best`:

```bash
git checkout best
```

Rebase onto main:

```bash
git rebase origin/main
```

If there is a conflict:

```bash
# edit conflicted files

git add .
git rebase --continue
```

Repeat if more conflicts appear.

Abort if necessary:

```bash
git rebase --abort
```

---

# 9. Rebase the Second Branch

After `best` is successfully rebased:

```bash
git checkout best1
```

Rebase `best1` onto the NEW `best`:

```bash
git rebase best
```

If conflict:

```bash
# resolve files

git add .
git rebase --continue
```

Abort:

```bash
git rebase --abort
```

---

# 10. Rebase the Third Branch

If you have `best2`:

```bash
git checkout best2
git rebase best1
```

Resolve conflicts if necessary:

```bash
git add .
git rebase --continue
```

So the complete cascade is:

```text
main
 ↓
best        ← rebase onto main
 ↓
best1       ← rebase onto updated best
 ↓
best2       ← rebase onto updated best1
```

---

# 11. Push After Rebase

A rebase changes commit history.

Therefore normal:

```bash
git push
```

may fail.

Use:

```bash
git push --force-with-lease origin best
```

Then:

```bash
git push --force-with-lease origin best1
```

Then:

```bash
git push --force-with-lease origin best2
```

IMPORTANT:

Prefer:

```bash
git push --force-with-lease
```

over:

```bash
git push --force
```

because `--force-with-lease` provides protection against overwriting unexpected remote changes.

---

# 12. After Rebasing, PR Bases Usually Stay the Same

Your PR relationships remain:

```text
PR #5
best -> main

PR #6
best1 -> best

PR #7
best2 -> best1
```

You normally do NOT need to recreate the PRs.

The branch histories changed, but the PRs continue tracking those branches.

---

# 13. If PR #1 Gets Merged

Suppose:

```text
PR #5
best -> main
```

is merged.

Before:

```text
main
 ↓
best       <- PR #5
 ↓
best1      <- PR #6
```

After PR #5 is merged, GitHub may automatically update PR #6's base from:

```text
best
```

to:

```text
main
```

The resulting conceptual structure becomes:

```text
main
 ↓
best1       <- PR #6
```

At this point, update your local branch.

---

# 14. Update the Remaining Branch After Parent PR Merges

Fetch:

```bash
git fetch origin
```

Checkout `best1`:

```bash
git checkout best1
```

Rebase it onto main:

```bash
git rebase origin/main
```

Resolve conflicts if needed:

```bash
git add .
git rebase --continue
```

Then push:

```bash
git push --force-with-lease origin best1
```

Now PR #6 becomes:

```text
best1 -> main
```

---

# 15. If You Want to Keep the Original Parent Branch

If PR #5 is still open and you make more changes in `best`:

```bash
git checkout best
```

Make changes:

```bash
git add .
git commit -m "Additional best changes"
```

Push:

```bash
git push origin best
```

PR #5 automatically gets the new commit.

Because `best1` is based on `best`, update `best1` when needed:

```bash
git checkout best1
git rebase best
```

Then:

```bash
git push --force-with-lease origin best1
```

---

# 16. Only Rebase `best1` onto `best`

If you do NOT want to update `best` from `main`, and only want:

```text
best
 ↓
best1
```

do:

```bash
git checkout best1
git rebase best
```

Then:

```bash
git push --force-with-lease origin best1
```

This is the manual equivalent of a "parent-only" rebase.

---

# 17. Create a Stack Entirely Manually

Example:

```bash
# Start
git checkout main
git pull origin main

# PR 1
git checkout -b best
# changes
git add .
git commit -m "PR1 changes"
git push -u origin best
```

GitHub:

```text
base: main
compare: best
```

Then:

```bash
# PR 2
git checkout best
git checkout -b best1
# changes
git add .
git commit -m "PR2 changes"
git push -u origin best1
```

GitHub:

```text
base: best
compare: best1
```

Then:

```bash
# PR 3
git checkout best1
git checkout -b best2
# changes
git add .
git commit -m "PR3 changes"
git push -u origin best2
```

GitHub:

```text
base: best1
compare: best2
```

---

# 18. Useful Git Commands

Check current branch:

```bash
git branch --show-current
```

See all local branches:

```bash
git branch
```

See remote branches:

```bash
git branch -r
```

See local + remote:

```bash
git branch -a
```

Check status:

```bash
git status
```

See commits:

```bash
git log --oneline --graph --all
```

Fetch remote changes:

```bash
git fetch origin
```

Update main:

```bash
git checkout main
git pull origin main
```

---

# 19. Check the Branch Relationship

Use:

```bash
git log --oneline --graph --decorate --all
```

For example:

```text
* best1
|
* best
|
* main
```

This lets you visually understand the stack.

---

# 20. Compare a Layer Against Its Parent

For `best1`:

```bash
git diff best..best1
```

This shows changes unique to `best1`.

For `best`:

```bash
git diff main..best
```

This shows changes unique to `best`.

---

# 21. Check Commits Unique to a Branch

For best:

```bash
git log main..best --oneline
```

For best1:

```bash
git log best..best1 --oneline
```

For best2:

```bash
git log best1..best2 --oneline
```

This is useful for understanding exactly what each PR contributes.

---

# 22. Remove an Open PR from the Stack

If PR #6 is still OPEN and you no longer want it stacked on PR #5:

On GitHub, change the PR base from:

```text
best
```

to:

```text
main
```

Then PR #6 becomes:

```text
best1 -> main
```

You can also close the PR/delete the branch if you no longer need the work.

IMPORTANT:

Changing the PR base does not automatically change the Git branch's commit ancestry.

The branch was still originally created from `best`.

If you want the branch itself to be independent from `best`, rebase it onto main:

```bash
git checkout best1
git rebase origin/main
git push --force-with-lease origin best1
```

---

# 23. Delete a Local Branch

If merged:

```bash
git branch -d best
```

If Git says it is not fully merged but you have verified the work is safely merged:

```bash
git branch -D best1
```

Use `-D` carefully.

---

# 24. Delete a Remote Branch

```bash
git push origin --delete best
```

or:

```bash
git push origin --delete best1
```

Clean stale remote references:

```bash
git fetch --prune
```

---

# 25. Important Difference: PR vs Branch vs Stack

These are separate:

```text
Git branch
    |
    +-- exists locally
    |
    +-- exists remotely

GitHub Pull Request
    |
    +-- compares two branches
    |
    +-- has a base branch

Stack
    |
    +-- describes dependent PRs
```

Deleting a local branch:

```bash
git branch -D best
```

does not delete the remote branch.

Deleting a remote branch:

```bash
git push origin --delete best
```

does not delete the PR history.

Closing/merging a PR does not necessarily delete the branch unless GitHub/repository settings do so.

---

# 26. Common Mistake: Creating PR #2 Against Main

Wrong:

```text
PR #5: best  -> main
PR #6: best1 -> main
```

This makes PR #6 contain all the changes from `best` as well.

Correct:

```text
PR #5: best  -> main
PR #6: best1 -> best
```

This makes PR #6 focused on its own layer.

---

# 27. Common Mistake: Creating the Branch from Main

Wrong:

```bash
git checkout main
git checkout -b best1
```

if `best1` is supposed to depend on `best`.

Correct:

```bash
git checkout best
git checkout -b best1
```

---

# 28. Common Mistake: Rebasing in the Wrong Order

Wrong:

```text
best1 -> main
best  -> main
```

Correct cascade:

```text
main
 ↓
best        <- rebase onto main first
 ↓
best1       <- then rebase onto updated best
 ↓
best2       <- then rebase onto updated best1
```

Always go bottom-to-top.

---

# 29. Common Mistake: Normal Pull After Rebase

After:

```bash
git rebase
```

avoid blindly doing:

```bash
git pull
```

because the local and remote histories may have diverged.

Instead:

```bash
git push --force-with-lease origin <branch>
```

If you need to update the branch from its parent, explicitly rebase:

```bash
git rebase <parent-branch>
```

---

# 30. Complete Manual Workflow

Start:

```bash
git checkout main
git pull origin main
```

Create PR1:

```bash
git checkout -b best
git add .
git commit -m "PR1"
git push -u origin best
```

GitHub:

```text
best -> main
```

Create PR2:

```bash
git checkout best
git checkout -b best1
git add .
git commit -m "PR2"
git push -u origin best1
```

GitHub:

```text
best1 -> best
```

Create PR3:

```bash
git checkout best1
git checkout -b best2
git add .
git commit -m "PR3"
git push -u origin best2
```

GitHub:

```text
best2 -> best1
```

Stack:

```text
main
 ↓
best       -> PR1
 ↓
best1      -> PR2
 ↓
best2      -> PR3
```

---

# 31. Main Changes While Stack Is Open

Fetch:

```bash
git fetch origin
```

Update best:

```bash
git checkout best
git rebase origin/main
```

Resolve:

```bash
git add .
git rebase --continue
```

Push:

```bash
git push --force-with-lease origin best
```

Update best1:

```bash
git checkout best1
git rebase best
git push --force-with-lease origin best1
```

Update best2:

```bash
git checkout best2
git rebase best1
git push --force-with-lease origin best2
```

Final:

```text
main
 ↓
best       <- updated
 ↓
best1      <- updated
 ↓
best2      <- updated
```

---

# 32. If a Conflict Happens in Best

```bash
git checkout best
git rebase origin/main
```

Conflict:

```text
<<<<<<< HEAD
main changes
=======
best changes
>>>>>>> commit
```

Resolve the file.

Then:

```bash
git add <file>
git rebase --continue
```

Repeat.

After completion:

```bash
git push --force-with-lease origin best
```

Then move upward.

---

# 33. If a Conflict Happens in Best1

```bash
git checkout best1
git rebase best
```

Resolve:

```bash
git add <file>
git rebase --continue
```

Then:

```bash
git push --force-with-lease origin best1
```

---

# 34. If You Want to Abort

For any current rebase:

```bash
git rebase --abort
```

This returns the branch to its state before the rebase.

---

# 35. Merge Order

For a stack:

```text
main
 ↓
best       -> PR #5
 ↓
best1      -> PR #6
 ↓
best2      -> PR #7
```

Normally merge bottom-up:

```text
PR #5
 ↓
PR #6
 ↓
PR #7
```

After PR #5 merges, PR #6 may automatically have its base changed from:

```text
best
```

to:

```text
main
```

Then update/rebase the remaining branch if necessary.

---

# 36. Best Practices

1. Keep each PR small.
2. One logical change per layer.
3. Create each branch from the previous branch.
4. Set each PR's base to the previous branch.
5. Keep the stack shallow when possible.
6. Rebase from bottom to top.
7. Use `--force-with-lease` after rebasing.
8. Resolve conflicts before pushing.
9. Merge from bottom to top.
10. Delete branches after they are merged if no longer needed.

---

# 37. Quick Cheatsheet

## Create PR1

```bash
git checkout main
git pull origin main

git checkout -b best
git add .
git commit -m "PR1"
git push -u origin best
```

GitHub:

```text
best -> main
```

## Create PR2

```bash
git checkout best
git checkout -b best1

git add .
git commit -m "PR2"
git push -u origin best1
```

GitHub:

```text
best1 -> best
```

## Create PR3

```bash
git checkout best1
git checkout -b best2

git add .
git commit -m "PR3"
git push -u origin best2
```

GitHub:

```text
best2 -> best1
```

## Main changed

```bash
git fetch origin

git checkout best
git rebase origin/main
git push --force-with-lease origin best

git checkout best1
git rebase best
git push --force-with-lease origin best1

git checkout best2
git rebase best1
git push --force-with-lease origin best2
```

## Conflict

```bash
# resolve file
git add .
git rebase --continue
```

Abort:

```bash
git rebase --abort
```

## Check history

```bash
git log --oneline --graph --decorate --all
```

## Check branch-specific commits

```bash
git log main..best --oneline
git log best..best1 --oneline
git log best1..best2 --oneline
```

## Delete local

```bash
git branch -d best
git branch -d best1
```

Force if already verified merged:

```bash
git branch -D best1
```

## Delete remote

```bash
git push origin --delete best
git push origin --delete best1
```

Clean stale refs:

```bash
git fetch --prune
```

---

# 38. One-Line Mental Model

Without `gh stack`, stacked PRs are simply:

```text
Create branch from previous branch
        ↓
Create PR against previous branch
        ↓
Rebase bottom-to-top when main changes
        ↓
Force-with-lease push after rebase
        ↓
Merge bottom-to-top
```

The essential commands are:

```bash
git checkout <parent>
git checkout -b <child>

git add .
git commit -m "..."

git push -u origin <child>
```

and for updates:

```bash
git fetch origin
git checkout <branch>
git rebase <parent>
git push --force-with-lease origin <branch>
```
