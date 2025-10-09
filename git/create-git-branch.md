<!--
# Creating a Local Git Branch

A short, polished guide for creating a local Git branch safely. Use this whenever you need a new feature, fix, or experiment branch.

## Quick context
Goal: create a local branch from a chosen base (current HEAD, a remote branch, or a specific commit) while preserving any uncommitted work.

Prerequisites:
- You’re in the repository root (or a subfolder under git control).
- Git installed and configured.
- A clean working tree or a plan for your uncommitted changes.

---

## 1 — Inspect repository state
Run these first to understand where you are:
```bash
git status
git branch --show-current
git fetch origin
```

---

## 2 — Handle uncommitted changes
Either commit or stash changes before switching branches.

Option A — Commit WIP:
```bash
git add -A
git commit -m "WIP: save changes"
```

Option B — Stash WIP:
```bash
git stash push -m "WIP: describe briefly"
```

Tip: Use meaningful stash messages so you can find them later (`git stash list`).

---

## 3 — Create and switch to the new branch
Choose one of the following (modern command shown first; older Git uses `git checkout -b` as noted).

Create from current HEAD:
```bash
git switch -c <branch-name>
```
(or)
```bash
git checkout -b <branch-name>
```

Create from a remote branch (e.g., origin/main):
```bash
git fetch origin
git switch -c <branch-name> origin/main
```
(or)
```bash
git checkout -b <branch-name> origin/main
```

Create from a specific commit or tag:
```bash
git switch -c <branch-name> <commit-sha-or-tag>
```
(or)
```bash
git checkout -b <branch-name> <commit-sha-or-tag>
```

---

## 4 — Verify you’re on the new branch
```bash
git branch        # current branch marked with *
git status
```

---

## 5 — Push and set upstream (when ready)
To publish and track the branch on origin:
```bash
git push -u origin <branch-name>
```
After this, `git push` and `git pull` will default to origin/<branch-name>.

---

## 6 — Recover stashed work (if you stashed earlier)
Apply and remove stash:
```bash
git stash pop
```
If you want to keep the stash entry:
```bash
git stash apply
```

---

## Branch naming recommendations
- Use readable, consistent names:
  - feature/<short-desc> (feature/add-login)
  - fix/<issue-number>-<short-desc> (fix/123-null-pointer)
  - chore/<tooling-or-maintenance>
- Keep names lowercase, use hyphens, avoid spaces.

---

## Troubleshooting & tips
- “detached HEAD” after creating from a commit: create a branch at that commit:
  ```bash
  git switch -c <branch-name> <commit-sha>
  ```
- If you have local changes preventing checkout, either commit or stash them.
- To list remote branches:
  ```bash
  git branch -r
  ```
- To delete a local branch (once merged or no longer needed):
  ```bash
  git branch -d <branch-name>    # safe delete (refuses if unmerged)
  git branch -D <branch-name>    # force delete
  ```

---

## Quick reference — one-liners
- Create branch from current HEAD:
  ```bash
  git switch -c feature/my-branch
  ```
- Create branch from origin/main:
  ```bash
  git fetch origin && git switch -c feature/my-branch origin/main
  ```
- Push and set upstream:
  ```bash
  git push -u origin feature/my-branch
  ```

---

Keep this file as a template in your repo (e.g., docs/git-branching.md) so your team follows the same safe workflow.

-->

# Create a local Git branch — Minimal checklist

<br/>

### 1. Inspect
```bash
git status
git branch --show-current
git fetch origin
```

<br/>

### 2. Save uncommitted work (choose one)
```bash
git add -A && git commit -m "WIP: save changes"
# or
git stash push -m "WIP: short note"
```

<br/>

### 3. Create & switch (pick the base)
- From current HEAD:
```bash
git switch -c <branch-name>
# (older Git: git checkout -b <branch-name>)
```
- From a remote branch (e.g., origin/main):
```bash
git fetch origin
git switch -c <branch-name> origin/main
```
- From a specific commit or tag:
```bash
git switch -c <branch-name> <commit-sha-or-tag>
```

<br/>

### 4. Verify
```bash
git branch   # * marks current
git status
```

<br/>

### 5. Publish (optional)
```bash
git push -u origin <branch-name>
```

### 6. Restore stash (if used)
```bash
git stash pop   # removes stash entry
# or
git stash apply # keeps stash entry
```

Tips
- Use short, consistent names: feature/..., fix/..., chore/...
- Use `git branch -r` to inspect remote branches.
