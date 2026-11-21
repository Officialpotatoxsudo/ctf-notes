# Git Recovery Cheatsheet

How to undo mistakes without losing work.

## 1. See what happened

```bash
git status
git log --oneline --graph --decorate -20
git reflog
```

`git reflog` is the safety net. It records where `HEAD` and branches pointed.

## 2. Undo unstaged changes

```bash
git restore file.txt
git restore .
```

## 3. Unstage a file

```bash
git restore --staged file.txt
```

## 4. Discard a local commit (keep changes)

```bash
git reset --soft HEAD~1
```

The commit is gone, the changes are staged.

```bash
git reset --mixed HEAD~1
```

The commit is gone, the changes are unstaged.

## 5. Discard a local commit and its changes

```bash
git reset --hard HEAD~1
```

This is destructive. Check `git reflog` first; you can usually recover the commit.

## 6. Undo a pushed commit safely

```bash
git revert <commit>
git push
```

`revert` creates a new commit that undoes the change. It is safe for shared branches.

## 7. Recover a lost commit

```bash
git reflog
git checkout -b rescue <sha>
```

Or:

```bash
git fsck --lost-found
ls .git/lost-found/commit/
```

## 8. Recover a deleted branch

```bash
git reflog --all | grep <branch>
git branch <branch> <sha>
```

If the branch was pushed, you can also fetch it from the remote:

```bash
git fetch origin
git checkout -b <branch> origin/<branch>
```

## 9. Undo a merge

Before pushing:

```bash
git reset --hard ORIG_HEAD
```

After pushing:

```bash
git revert -m 1 <merge-commit>
```

## 10. Stash

```bash
git stash push -m "wip"
git stash list
git stash show -p stash@{0}
git stash pop
git stash apply stash@{0}
git stash drop stash@{0}
```

`pop` applies and drops; `apply` keeps the stash.

## 11. Cherry-pick and interactive rebase

```bash
git cherry-pick <sha>
git rebase -i HEAD~5
```

In the interactive rebase, use `pick`, `reword`, `edit`, `squash`, `fixup`, or `drop`.

## 12. Find the commit that broke it

```bash
git bisect start
git bisect bad
git bisect good <known-good-sha>
# test, then:
git bisect good
git bisect bad
git bisect reset
```

## 13. Restore a file from a commit

```bash
git checkout <sha> -- path/to/file
git restore --source=<sha> path/to/file
```

## 14. Fix the last commit message

```bash
git commit --amend
```

Only amend commits that have not been pushed to a shared branch.

## 15. Emergency rules

- Don't run `git reset --hard` until you have checked `git reflog`.
- Don't force-push a shared branch without coordination.
- Commit or stash before switching branches.
- When in doubt, create a backup branch: `git branch backup-$(date +%s)`.
