# gh stack — Complete Cheatsheet

## 1. Mental Model

A stacked PR means each branch/PR is based on the branch below it.

```text
main
  |
  +-- best        -> PR #5
       |
       +-- best1  -> PR #6
            |
            +-- best2 -> PR #7
```

Think of every branch as one layer.

- PR #5: `best -> main`
- PR #6: `best1 -> best`
- PR #7: `best2 -> best1`

The lower PR is the base/dependency of the PR above it.

---

# 2. Installation and Authentication

Install the extension:

```bash
gh extension install github/gh-stack
```

Check GitHub CLI authentication:

```bash
gh auth status
```

Login:

```bash
gh auth login
```

Check the repository remote:

```bash
git remote -v
```

It should point to GitHub, for example:

```text
origin  https://github.com/<user>/<repo>.git
```

---

# 3. Initialize a Stack from an Existing PR Branch

Suppose you already have:

```text
main
  |
  +-- best
```

and `best -> main` already has PR #5.

Checkout the existing branch:

```bash
git checkout best
git pull origin best
```

Initialize the stack:

```bash
gh stack init best
```

Typical output:

```text
Adopted 1 branch: main <- best
You're on best (top of stack).
Found PRs for 1 of 1 branch.
```

This adopts the existing branch/PR into a stack.

---

# 4. Add a New Layer

From `best`:

```bash
gh stack add best1
```

This creates `best1` on top of `best` and checks it out.

Stack becomes:

```text
main
  |
  +-- best       -> PR #5
       |
       +-- best1 -> new PR
```

Important:

```bash
gh stack add best1
```

is preferred over manually doing:

```bash
git checkout -b best1
```

when using `gh stack`.

---

# 5. Make Changes

After `gh stack add best1`, you are automatically on `best1`.

Make changes:

```bash
git status
```

Then:

```bash
git add .
git commit -m "Add best1 changes"
```

Your stack is now:

```text
main
  |
  +-- best
       |
       +-- best1
            |
            +-- your new commit
```

---

# 6. View the Stack

```bash
gh stack view
```

Example:

```text
Stack #7
Base: main

#6 OPEN
best1
  1 file changed
  1 commit

#5 OPEN
best
  2 files changed
  4 commits

main
```

Interactive controls:

```text
UP/DOWN     Navigate
c           View commits
f           View files
o           Open PR
ENTER       Checkout
q           Quit
```

---

# 7. Create / Update PRs

Main command:

```bash
gh stack submit
```

It can:

- Push stack branches
- Create missing PRs
- Update existing PRs
- Link PRs into a GitHub Stack
- Synchronize the stack on GitHub

Example:

```text
main
  |
  +-- best       -> PR #5
       |
       +-- best1 -> PR #6
```

If `best` already has PR #5, `gh stack submit` can create only the new PR for `best1`.

---

# 8. Submit Screen

`gh stack submit` may open an interactive screen.

You may see:

```text
Creating 1 PR

[x] best1   NEW
[ ] best    open · #5
```

Meaning:

- `best1` needs a new PR.
- `best` already has PR #5.
- Only `best1` will be submitted as a new PR.

Useful controls:

```text
UP/DOWN       Select branch
TAB           Move between fields
CTRL+X        Skip/include branch
CTRL+S        Submit PRs
ESC           Quit
CTRL+H        Help
CTRL+P        Preview
CTRL+E        Open editor
```

---

# 9. Ready vs Draft PR

When creating a PR:

```text
CREATE AS [ Ready | Draft ]
```

## Ready

Normal PR.

Use when the PR is ready for review.

```text
PR #6
OPEN
Ready for review
```

## Draft

PR is still being worked on.

Use when you want the PR to exist but don't want to request review/merge yet.

```text
PR #6
DRAFT
```

A Draft PR can later be changed to Ready for review in GitHub.

---

# 10. Switch Between Stack Branches

Interactive:

```bash
gh stack switch
```

Example:

```text
1. best
2. best1
```

Select a branch.

You can also use stack navigation commands such as:

```bash
gh stack up
gh stack down
```

Go to the top:

```bash
gh stack top
```

Go to the bottom:

```bash
gh stack bottom
```

Go to the trunk:

```bash
gh stack trunk
```

---

# 11. Synchronize the Entire Stack

Normal command:

```bash
gh stack sync
```

Conceptually:

```text
main
  |
  +-- best
       |
       +-- best1
```

If `main` has changed, `gh stack sync` can:

```text
Fetch main
   |
Update trunk
   |
Rebase best onto main
   |
Rebase best1 onto updated best
   |
Push branches
   |
Sync PRs
```

So:

```text
main
  |
  +-- best
       |
       +-- best1
```

is processed bottom-to-top.

The important messages may look like:

```text
Fetched latest main
Trunk main is up to date

Rebasing stack...
Rebased best onto main
Rebased best1 onto best

Pushing branches...
Syncing PRs...
Stack on GitHub is up to date
```

---

# 12. If Main Has NOT Changed

You may still see:

```text
Rebased best onto main
Rebased best1 onto best
```

This does not necessarily mean new commits were added to main.

It means `gh stack` checked/rebased the stack according to its dependency structure.

---

# 13. If You Run `gh stack sync` from main

If `main` is the trunk of multiple stacks, you may see:

```text
Branch "main" is the trunk of multiple stacks

(main) <- new <- new1
(main) <- best <- best1
```

GitHub CLI is asking which stack you want.

For the best stack, select:

```text
(main) <- best <- best1
```

Better approach:

```bash
git checkout best1
gh stack sync
```

When you are on a stack branch, `gh stack` can identify the relevant stack more directly.

---

# 14. Rebase the Entire Stack

```bash
gh stack rebase
```

For:

```text
main
  |
  +-- best
       |
       +-- best1
```

it performs:

```text
best  <- main
best1 <- updated best
```

This is a cascading rebase from the bottom of the stack upward.

---

# 15. Rebase Only Stack Layers, Without Touching Main

If you specifically want:

```text
best1 <- best
```

without first rebasing `best` onto `main`, use:

```bash
gh stack rebase --no-trunk
```

Normal:

```bash
gh stack rebase
```

Conceptually:

```text
main
  |
  +-- best        <- main
       |
       +-- best1  <- best
```

No-trunk:

```bash
gh stack rebase --no-trunk
```

Conceptually:

```text
best
  |
  +-- best1
```

---

# 16. Conflict Handling

If:

```bash
gh stack sync
```

or:

```bash
gh stack rebase
```

detects a conflict, you may see:

```text
Conflict detected rebasing best onto main

Conflicted files:
  C new1.md
```

Git may put you into a detached HEAD state temporarily during the rebase.

Do NOT create a new branch because of that.

---

# 17. Resolve a Conflict

Open the conflicted file.

You may see:

```text
<<<<<<< HEAD
This is a change in the main
=======
This is the PR2
>>>>>>> commit
```

Choose the content you want.

For example, to keep both:

```text
This is the PR2

This is a change in the main
```

Remove:

```text
<<<<<<<
=======
>>>>>>>
```

Then stage the file:

```bash
git add .
```

Continue:

```bash
gh stack rebase --continue
```

If another conflict appears:

```bash
# resolve files
git add .
gh stack rebase --continue
```

Repeat until complete.

---

# 18. Abort a Rebase

If you want to completely cancel the rebase:

```bash
gh stack rebase --abort
```

This restores the branches to their state before the rebase.

---

# 19. After a Rebase

When you see:

```text
All branches in stack rebased locally with main
```

your LOCAL stack has been rewritten.

Now update GitHub:

```bash
gh stack submit
```

Do NOT immediately use a normal:

```bash
git pull
```

after a stack rebase.

Why?

Rebase changes commit IDs, so local and remote branches can temporarily look divergent.

Use:

```bash
gh stack submit
```

to push/update the rebased stack.

---

# 20. Why `git pull` Can Fail After a Stack Rebase

You may see:

```text
You have divergent branches and need to specify
how to reconcile them.
```

This can happen because:

```text
Local best1
   |
   +-- rebased commits

Remote best1
   |
   +-- old commits
```

After:

```bash
gh stack submit
```

you may see:

```text
+ oldHash...newHash best1 -> origin/best1 (forced update)
```

That is expected after a rebase.

Prefer the stack workflow:

```bash
gh stack rebase
gh stack submit
```

or:

```bash
gh stack sync
```

rather than manually pulling rebased stack branches.

---

# 21. Typical Conflict Workflow

```text
gh stack sync
      |
      v
Conflict
      |
      v
gh stack rebase
      |
      v
Fix files
      |
      v
git add .
      |
      v
gh stack rebase --continue
      |
      v
Rebase complete
      |
      v
gh stack submit
```

---

# 22. Add Changes to a Parent Branch

Suppose:

```text
main
  |
  +-- best       -> PR #5
       |
       +-- best1 -> PR #6
```

You want to add a new commit to `best`.

Switch:

```bash
git checkout best
```

Make changes:

```bash
git add .
git commit -m "Additional best changes"
```

Now:

```text
main
  |
  +-- best       <- NEW COMMIT
       |
       +-- best1
```

To synchronize the stack:

```bash
gh stack sync
```

This makes `best1` track the updated `best`.

Then:

```bash
gh stack submit
```

to push/update the PRs.

---

# 23. Important PR Diff Concept

With:

```text
main
  |
  +-- best       -> PR #5
       |
       +-- best1 -> PR #6
```

PR #5:

```text
best -> main
```

contains the changes introduced by `best`.

PR #6:

```text
best1 -> best
```

contains the changes unique to `best1`.

That is the main benefit of stacked PRs:

```text
PR #5
Parent/base work
       |
       v
PR #6
Only additional work
```

---

# 24. Link Existing PRs

If PRs already exist and you want to connect them into a stack, use:

```bash
gh stack link
```

The exact arguments depend on the existing branches/PRs.

For example:

```bash
gh stack link best best1
```

The order is bottom -> top.

---

# 25. Modify / Restructure a Stack

Interactive:

```bash
gh stack modify
```

It can be used to restructure a stack, including operations such as:

- Move branch up
- Move branch down
- Insert branch
- Rename branch
- Drop branch
- Fold changes
- Undo changes

Use this when the stack structure itself needs to change.

---

# 26. Unstack

If PRs are still open, you can remove them from the stack using:

```bash
gh stack unstack
```

Unstacking removes the stack relationship.

It does NOT necessarily mean:

```text
delete branch
delete PR
```

Those are separate operations.

---

# 27. Merged PRs Cannot Be Removed from the Stack

If PRs are already merged, you may get:

```text
Unstacking not allowed:
Pull requests #5, #6 cannot be removed from this stack
```

This is expected.

Merged PRs remain part of the historical stack relationship.

You generally do not need to remove them.

Think of it as:

```text
ACTIVE STACK

main
 |
 +-- best
      |
      +-- best1

after both are merged:

main
 |
 +-- merged history
```

The PRs remain in GitHub history.

---

# 28. Deleting Local Branches

If a branch has definitely been merged:

```bash
git branch -d best
```

If Git says it is not fully merged because the branch was rebased:

```bash
git branch -D best1
```

Use `-D` only when you are sure the work is already merged or otherwise backed up.

---

# 29. Deleting Remote Branches

Delete a remote branch:

```bash
git push origin --delete best
```

Delete multiple:

```bash
git push origin --delete best
git push origin --delete best1
```

Then clean stale remote-tracking references:

```bash
git fetch --prune
```

Check:

```bash
git branch -r
```

---

# 30. Local vs Remote vs Stack

These are three different things.

## Local branch

```bash
git branch -D best
```

Deletes your local branch.

## Remote branch

```bash
git push origin --delete best
```

Deletes the branch on GitHub.

## Stack relationship

```bash
gh stack unstack
```

Removes the stack relationship when allowed.

Deleting one does not automatically mean deleting the others.

---

# 31. `git branch -r` and Stale References

If:

```bash
git branch -r
```

shows:

```text
origin/best
origin/best1
origin/new
origin/new1
```

but those branches were already deleted on GitHub, run:

```bash
git fetch --prune
```

Then:

```bash
git branch -r
```

Stale `origin/...` references should disappear.

---

# 32. `git push` vs Stack Submit

Normal:

```bash
git push
```

only pushes your Git branch.

Stack:

```bash
gh stack submit
```

pushes the stack and manages its PR state.

For stacked work, prefer:

```bash
gh stack submit
```

after rebasing.

---

# 33. Merge the Stack

You can merge the stack through the CLI:

```bash
gh stack merge
```

You can also target a PR/layer depending on the command options:

```bash
gh stack merge <PR>
```

This is useful when you want to merge the stack in dependency order.

---

# 34. Recommended Daily Workflow

For a stack:

```text
main
  |
  +-- feature1
       |
       +-- feature2
```

Start:

```bash
git checkout feature1
gh stack init feature1
gh stack add feature2
```

Work:

```bash
git add .
git commit -m "..."
```

Submit:

```bash
gh stack submit
```

Main changes:

```bash
gh stack sync
```

Conflict:

```bash
gh stack rebase

# resolve conflict
git add .
gh stack rebase --continue
```

Submit rewritten branches:

```bash
gh stack submit
```

Inspect:

```bash
gh stack view
```

Switch:

```bash
gh stack switch
```

---

# 35. The Most Important Commands

If you only remember these:

```bash
# See stack
gh stack view

# Initialize existing branch
gh stack init <branch>

# Add layer
gh stack add <branch>

# Switch layer
gh stack switch

# Sync entire stack
gh stack sync

# Rebase entire stack
gh stack rebase

# Rebase stack without touching trunk
gh stack rebase --no-trunk

# Continue conflict resolution
gh stack rebase --continue

# Abort rebase
gh stack rebase --abort

# Push + create/update PRs
gh stack submit

# Restructure stack
gh stack modify

# Link existing PRs/branches
gh stack link

# Unstack open PRs
gh stack unstack

# Merge stack
gh stack merge
```

---

# 36. Golden Rules

## Rule 1

Each PR should have its own branch.

```text
main
 ↓
branch1
 ↓
branch2
 ↓
branch3
```

## Rule 2

A higher branch is based on the branch below it.

```text
branch2 → branch1
```

not:

```text
branch2 → main
```

## Rule 3

After rebase, use:

```bash
gh stack submit
```

to push the rewritten stack.

## Rule 4

If main changes:

```bash
gh stack sync
```

## Rule 5

If you want only the parent-child rebase without updating trunk:

```bash
gh stack rebase --no-trunk
```

## Rule 6

When conflicts happen:

```bash
resolve
git add .
gh stack rebase --continue
```

## Rule 7

Do not use normal `git pull` as your primary stack synchronization mechanism after rebases.

---

# 37. One-Line Mental Model

```text
gh stack = Git branches + dependent PRs + cascading rebases + PR synchronization
```

The basic lifecycle is:

```text
INIT
  ↓
ADD
  ↓
COMMIT
  ↓
SUBMIT
  ↓
SYNC
  ↓
REBASE if needed
  ↓
RESOLVE conflicts if needed
  ↓
CONTINUE
  ↓
SUBMIT
  ↓
MERGE
```
