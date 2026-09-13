# Git Workflow: Branching, Rebasing, and Day-to-Day Commands

---

## 1. Core Daily Commands

| Command | What it does |
|---|---|
| `git status` | Show staged/unstaged/untracked changes |
| `git diff` | Show unstaged changes |
| `git diff --staged` | Show staged changes |
| `git add <file>` | Stage a file |
| `git add -p` | Stage interactively, chunk by chunk |
| `git commit -m "msg"` | Commit staged changes |
| `git commit --amend` | Edit the last commit (message and/or contents) |
| `git log --oneline --graph --all` | Compact visual history of all branches |
| `git log -p <file>` | Full diff history of a specific file |

---

## 2. Branching

| Command | What it does |
|---|---|
| `git branch` | List local branches |
| `git branch -a` | List local + remote branches |
| `git branch <name>` | Create a branch (doesn't switch to it) |
| `git switch <name>` | Switch to existing branch |
| `git switch -c <name>` | Create + switch in one step |
| `git branch -d <name>` | Delete a branch (safe — refuses if unmerged) |
| `git branch -D <name>` | Force delete |
| `git branch -m <new-name>` | Rename current branch |

`git switch` is the modern replacement for `git checkout` when just moving between branches — clearer intent, fewer footguns.

---

## 3. Merging

```bash
git switch main
git merge feature-branch
```

Creates a merge commit (unless fast-forward is possible). Use when you want to preserve the branch's individual commit history as-is.

Conflict resolution:
```bash
# after a conflict
git status                  # see which files conflict
# edit the files, resolve the <<<<<<< ======= >>>>>>> markers
git add <resolved-file>
git commit                  # finishes the merge
```

Abort a merge in progress:
```bash
git merge --abort
```

---

## 4. Rebasing

Rebase replays your branch's commits on top of another branch, producing linear history (no merge commit).

```bash
git switch feature-branch
git rebase main
```

If conflicts occur:
```bash
# resolve conflicts in the files
git add <resolved-file>
git rebase --continue
```

Abort mid-rebase:
```bash
git rebase --abort
```

Skip a specific problem commit during rebase:
```bash
git rebase --skip
```

### Interactive rebase (rewrite history)
```bash
git rebase -i HEAD~5      # last 5 commits
```
Opens an editor with commands per commit:
```
pick   <hash> commit message
reword <hash> commit message   # edit message only
squash <hash> commit message   # combine into previous commit
fixup  <hash> commit message   # like squash, discard message
drop   <hash> commit message   # remove commit entirely
```

**Rule of thumb:** never rebase commits that have already been pushed and could be in someone else's local history — it rewrites hashes and causes divergence. Safe to rebase freely on a branch only you're working on.

---

## 5. Merge vs Rebase — When to Use Which

| Situation | Use |
|---|---|
| Bringing `main` up to date into your feature branch, still WIP | Rebase (`git rebase main`) — keeps history clean |
| Feature branch is done, merging into `main`/shared branch | Merge (`git merge`) — preserves what actually happened, safer for shared history |
| Cleaning up your own messy WIP commits before opening a PR | Interactive rebase (`git rebase -i`) |
| Branch already pushed and others have pulled it | Avoid rebasing it — merge instead |

---

## 6. Stashing

| Command | What it does |
|---|---|
| `git stash` | Shelve current changes, restore clean working tree |
| `git stash -u` | Also stash untracked files |
| `git stash list` | Show all stashes |
| `git stash pop` | Reapply most recent stash and remove it from the stash list |
| `git stash apply` | Reapply without removing from stash list |
| `git stash drop` | Delete a stash without applying |
| `git stash show -p stash@{0}` | View diff of a specific stash |

---

## 7. Undoing Things

| Situation | Command |
|---|---|
| Unstage a file (keep changes) | `git restore --staged <file>` |
| Discard unstaged changes to a file | `git restore <file>` |
| Undo last commit, keep changes staged | `git reset --soft HEAD~1` |
| Undo last commit, keep changes unstaged | `git reset HEAD~1` |
| Undo last commit, discard changes entirely | `git reset --hard HEAD~1` (destructive) |
| Revert a commit safely (adds a new commit undoing it) | `git revert <hash>` |
| Recover a "lost" commit after a hard reset | `git reflog` → find the hash → `git reset --hard <hash>` |

`git revert` is the safe option for anything already pushed/shared — it doesn't rewrite history, just adds an inverse commit.

---

## 8. Remotes

| Command | What it does |
|---|---|
| `git remote -v` | List remotes and URLs |
| `git fetch` | Download remote changes, don't merge |
| `git pull` | Fetch + merge (or rebase, if `pull.rebase = true`) |
| `git pull --rebase` | Fetch + rebase instead of merge |
| `git push` | Push current branch |
| `git push -u origin <branch>` | Push + set upstream tracking (first push of a new branch) |
| `git push --force-with-lease` | Force push, but safely — fails if remote has commits you don't have locally |

Avoid plain `git push --force` — use `--force-with-lease` so you don't clobber someone else's work you haven't fetched yet.

Set rebase as default pull behavior (recommended for solo/small-team work to avoid merge-commit noise):
```bash
git config --global pull.rebase true
```

---

## 9. Tags

```bash
git tag v1.0.0                     # lightweight tag on current commit
git tag -a v1.0.0 -m "message"     # annotated tag (preferred for releases)
git push origin v1.0.0             # push a single tag
git push origin --tags             # push all tags
```

---

## 10. Useful Aliases (add to `~/.gitconfig` or via `git config --global`)

```bash
git config --global alias.st status
git config --global alias.co switch
git config --global alias.br branch
git config --global alias.lg "log --oneline --graph --all --decorate"
git config --global alias.last "log -1 HEAD"
git config --global alias.amend "commit --amend --no-edit"
```

---

## 11. A Realistic Feature-Branch Workflow

```bash
git switch main
git pull
git switch -c feature/new-thing

# ...work, commit as you go...

git rebase main                    # pick up latest main, keep linear history
# resolve any conflicts, git rebase --continue

git push -u origin feature/new-thing
# open PR

# after review feedback:
git commit --amend                 # or new commits, then rebase -i to squash
git push --force-with-lease

# after PR approved and merged (often via GitHub UI, squash-merge):
git switch main
git pull
git branch -d feature/new-thing
```

---

## 12. Reference

- `git help <command>` — man page for any command, e.g. `git help rebase`
- Pro Git book (free): https://git-scm.com/book
