# 🚀 Git Interview Questions & Answers
### Complete Preparation Guide — Top 30 Questions

---

## 🟢 BASIC LEVEL

---

### Q1. What is Git? How is it different from other version control systems?

**Answer:**
Git is a **distributed version control system (DVCS)** that tracks changes in source code during software development. Unlike centralized VCS (like SVN), every developer has a full copy of the entire repository history on their local machine.

| Feature | Git (DVCS) | SVN (CVCS) |
|---|---|---|
| Repository | Full local copy | Central server only |
| Speed | Fast (local ops) | Slow (network calls) |
| Offline Work | ✅ Yes | ❌ Limited |
| Branching | Lightweight & fast | Expensive |

---

### Q2. What is the difference between `git fetch` and `git pull`?

**Answer:**

- **`git fetch`** — Downloads changes from the remote but does **NOT** merge them into your working branch. It's safe and non-destructive.
- **`git pull`** — Is `git fetch` + `git merge`. It downloads AND merges changes into your current branch.

```bash
# Fetch only (inspect before merging)
git fetch origin

# Pull and merge immediately
git pull origin main
```

> 💡 **Best Practice:** Prefer `git fetch` then review, then `git merge` — gives you more control.

---

### Q3. What is a Git repository? What is the difference between a local and remote repository?

**Answer:**

- A **repository** (repo) is a directory containing all project files and the entire revision history stored in a `.git` folder.
- **Local repository** — exists on your machine. All commits, branches, and history are stored locally.
- **Remote repository** — hosted on a server (GitHub, GitLab, Bitbucket). Used for collaboration and backup.

```bash
# Initialize a local repo
git init

# Link to a remote
git remote add origin https://github.com/user/repo.git
```

---

### Q4. Explain the three stages/areas in Git.

**Answer:**

Git has three main areas:

1. **Working Directory** — Where you edit files. Changes here are "untracked" or "modified."
2. **Staging Area (Index)** — A buffer zone. You `git add` files here before committing.
3. **Repository (.git)** — The permanent history. Created when you `git commit`.

```
Working Dir  →  git add  →  Staging Area  →  git commit  →  Repository
```

---

### Q5. What is the difference between `git merge` and `git rebase`?

**Answer:**

- **`git merge`** — Combines two branches by creating a new "merge commit." Preserves full history.
- **`git rebase`** — Moves or replays commits from one branch onto another. Creates a **linear, cleaner history** but rewrites commit SHAs.

```bash
# Merge (preserves history, adds merge commit)
git checkout main
git merge feature-branch

# Rebase (linear history, rewrites commits)
git checkout feature-branch
git rebase main
```

> ⚠️ **Golden Rule:** Never rebase commits that have been pushed to a shared/public branch.

---

### Q6. What is `git stash`? When would you use it?

**Answer:**

`git stash` temporarily saves your uncommitted changes (both staged and unstaged) so you can work on something else, then come back and re-apply them.

```bash
# Save changes to stash
git stash

# List all stashes
git stash list

# Apply the most recent stash
git stash pop

# Apply a specific stash
git stash apply stash@{2}

# Drop a stash
git stash drop stash@{0}
```

> 💡 **Use case:** You're in the middle of a feature, an urgent bug fix comes in — stash your work, fix the bug, then pop your stash.

---

### Q7. What is HEAD in Git?

**Answer:**

`HEAD` is a special pointer that always points to the **current commit** you're on — usually the tip of your current branch. It's how Git knows "where you are" in the repo history.

- **Attached HEAD** — Points to a branch name (normal state).
- **Detached HEAD** — Points directly to a commit hash (happens when you checkout a specific commit).

```bash
# See where HEAD points
cat .git/HEAD

# Detached HEAD example
git checkout abc1234  # → HEAD is now at abc1234
```

---

### Q8. What is the difference between `git reset`, `git revert`, and `git checkout`?

**Answer:**

| Command | What it does | Safe for shared branches? |
|---|---|---|
| `git reset` | Moves HEAD (can delete commits) | ❌ No |
| `git revert` | Creates a new "undo" commit | ✅ Yes |
| `git checkout` | Switches branch or restores file | ✅ Yes |

```bash
# Reset (3 modes)
git reset --soft HEAD~1   # Undo commit, keep changes staged
git reset --mixed HEAD~1  # Undo commit, keep changes unstaged (default)
git reset --hard HEAD~1   # Undo commit, DISCARD all changes ⚠️

# Revert (safe undo — creates new commit)
git revert abc1234

# Checkout a file (discard working dir changes)
git checkout -- filename.txt
```

---

### Q9. What is a `.gitignore` file?

**Answer:**

`.gitignore` is a configuration file that tells Git which files/directories to **ignore and not track**. Commonly used for:

- Build artifacts (`node_modules/`, `dist/`, `*.class`)
- Environment files (`.env`, secrets)
- OS files (`.DS_Store`, `Thumbs.db`)
- IDE config files (`.idea/`, `.vscode/`)

```gitignore
# Example .gitignore
node_modules/
.env
*.log
dist/
.DS_Store
```

> 💡 If you've already committed a file, use `git rm --cached filename` to untrack it.

---

### Q10. What is `git clone`? How is it different from `git init`?

**Answer:**

- **`git clone`** — Copies an **existing remote repository** to your local machine. Sets up the remote origin automatically.
- **`git init`** — Creates a **brand new, empty** Git repository in the current directory.

```bash
# Clone an existing repo
git clone https://github.com/user/repo.git

# Initialize a new empty repo
git init my-project
```

---

## 🟡 INTERMEDIATE LEVEL

---

### Q11. Explain the Git branching strategy. What is Git Flow?

**Answer:**

**Git Flow** is a popular branching model with the following branches:

| Branch | Purpose |
|---|---|
| `main` / `master` | Production-ready code only |
| `develop` | Integration branch for features |
| `feature/*` | New features (branch off develop) |
| `release/*` | Pre-release prep (branch off develop) |
| `hotfix/*` | Urgent production fixes (branch off main) |

```bash
# Typical Git Flow
git checkout develop
git checkout -b feature/login-page
# ... work ...
git commit -m "Add login page"
git checkout develop
git merge feature/login-page
```

---

### Q12. What is a merge conflict? How do you resolve it?

**Answer:**

A merge conflict occurs when two branches modify the **same part of a file differently** and Git can't automatically decide which change to keep.

**Resolution Steps:**
```bash
# 1. Attempt merge
git merge feature-branch

# 2. Git marks conflicts in file:
# <<<<<<< HEAD
# your changes
# =======
# their changes
# >>>>>>> feature-branch

# 3. Edit the file manually to keep desired content

# 4. Mark as resolved
git add conflicted-file.txt

# 5. Complete the merge
git commit
```

> 💡 Use `git mergetool` or IDE tools (VS Code, IntelliJ) for a visual conflict resolver.

---

### Q13. What is `git cherry-pick`?

**Answer:**

`git cherry-pick` applies the changes from a **specific commit** onto your current branch — without merging the entire branch.

```bash
# Apply a single commit from another branch
git cherry-pick abc1234

# Apply multiple commits
git cherry-pick abc1234 def5678

# Cherry-pick without auto-committing
git cherry-pick --no-commit abc1234
```

> 💡 **Use case:** A bug fix was committed on `develop` and you need it on a `hotfix` branch without merging all of develop.

---

### Q14. What is `git bisect`?

**Answer:**

`git bisect` uses **binary search** to find the commit that introduced a bug. You mark a known good and bad commit, and Git checks out commits in between until the culprit is found.

```bash
# Start bisect
git bisect start

# Mark current commit as bad (has the bug)
git bisect bad

# Mark a known good commit
git bisect good v1.0.0

# Git checks out a middle commit — test it, then:
git bisect good   # or
git bisect bad

# Git narrows it down until the bad commit is found
# When done:
git bisect reset
```

---

### Q15. What is the difference between `git diff` and `git status`?

**Answer:**

- **`git status`** — Shows which files are modified, staged, or untracked (high-level overview).
- **`git diff`** — Shows the **exact line-by-line changes** in files.

```bash
# Status overview
git status

# Diff in working directory (unstaged changes)
git diff

# Diff of staged changes
git diff --staged

# Diff between two branches
git diff main..feature-branch

# Diff between two commits
git diff abc1234 def5678
```

---

### Q16. What is a detached HEAD state? How do you recover from it?

**Answer:**

A **detached HEAD** occurs when HEAD points to a specific commit instead of a branch. Any commits made here are "orphaned" and can be lost if you switch branches.

```bash
# How it happens
git checkout abc1234   # → You are in 'detached HEAD' state

# Recovery Option 1: Create a new branch from here
git checkout -b new-branch-name

# Recovery Option 2: Discard and go back
git checkout main

# If you made commits in detached HEAD and want to save them:
git branch temp-save    # create branch at current position
git checkout main
git merge temp-save
```

---

### Q17. Explain `git log` and its useful flags.

**Answer:**

`git log` shows the commit history. Useful flags:

```bash
# Basic log
git log

# One line per commit
git log --oneline

# Graph view (great for visualizing branches)
git log --oneline --graph --all

# Filter by author
git log --author="John"

# Filter by date
git log --after="2024-01-01" --before="2024-12-31"

# Show changes in each commit
git log -p

# Limit number of commits shown
git log -n 5

# Search commit messages
git log --grep="fix login"
```

---

### Q18. What is `git tag`? What are the types of tags?

**Answer:**

Tags mark specific points in history — typically **release versions**.

**Two types:**
1. **Lightweight tag** — Just a pointer to a commit (no extra info).
2. **Annotated tag** — Full object with tagger name, date, message.

```bash
# Lightweight tag
git tag v1.0.0

# Annotated tag (recommended for releases)
git tag -a v1.0.0 -m "Release version 1.0.0"

# Push tags to remote
git push origin v1.0.0
git push origin --tags   # push all tags

# List tags
git tag

# Delete a tag
git tag -d v1.0.0
git push origin --delete v1.0.0   # remote delete
```

---

### Q19. What is `git remote`? Explain common remote commands.

**Answer:**

`git remote` manages connections to remote repositories.

```bash
# List all remotes
git remote -v

# Add a remote
git remote add origin https://github.com/user/repo.git

# Add a second remote (common in open source forks)
git remote add upstream https://github.com/original/repo.git

# Remove a remote
git remote remove origin

# Rename a remote
git remote rename origin new-name

# Sync fork with upstream
git fetch upstream
git merge upstream/main
```

---

### Q20. What is `git reflog`? When is it useful?

**Answer:**

`git reflog` records every movement of HEAD — even commits that were "lost" after a reset or branch deletion. It's your **safety net** in Git.

```bash
# View reflog
git reflog

# Output example:
# abc1234 HEAD@{0}: commit: Add feature
# def5678 HEAD@{1}: reset: moving to HEAD~1

# Recover a "lost" commit
git checkout abc1234            # or
git branch recover-branch abc1234
```

> 💡 **Use case:** You ran `git reset --hard` and lost commits — `git reflog` can save you.

---

## 🔴 ADVANCED LEVEL

---

### Q21. What is the difference between `git merge --squash` and a regular merge?

**Answer:**

- **Regular merge** — Brings all individual commits from the feature branch into the target branch.
- **`--squash`** — Combines all commits from the feature branch into a **single staged change**, which you then commit manually.

```bash
git checkout main
git merge --squash feature-branch
git commit -m "Add entire feature X"
```

> 💡 Great for keeping `main` history clean when a feature branch had many small/messy commits.

---

### Q22. Explain `git rebase -i` (interactive rebase).

**Answer:**

Interactive rebase lets you **rewrite commit history** before pushing. You can squash, reorder, edit, or drop commits.

```bash
# Rebase last 3 commits interactively
git rebase -i HEAD~3

# Editor opens with:
# pick abc1234 First commit
# pick def5678 Second commit
# pick ghi9012 Third commit

# Change 'pick' to:
# squash (s) — combine with previous commit
# reword (r) — edit commit message
# edit (e)   — pause to amend the commit
# drop (d)   — delete the commit
```

> ⚠️ Only do this on local/un-pushed commits.

---

### Q23. How do you undo the last commit without losing changes?

**Answer:**

```bash
# Undo last commit, keep changes STAGED
git reset --soft HEAD~1

# Undo last commit, keep changes UNSTAGED (default)
git reset --mixed HEAD~1

# Undo last commit AND discard all changes ⚠️ DANGEROUS
git reset --hard HEAD~1

# If already pushed — use revert instead
git revert HEAD
git push origin main
```

---

### Q24. What is a Git submodule?

**Answer:**

A **submodule** is a Git repository embedded inside another Git repository. Used to include external dependencies while keeping separate version control.

```bash
# Add a submodule
git submodule add https://github.com/user/lib.git libs/mylib

# Clone repo with submodules
git clone --recurse-submodules https://github.com/user/project.git

# Initialize and update submodules in existing clone
git submodule init
git submodule update

# Update all submodules to latest
git submodule update --remote
```

---

### Q25. What is `git blame`?

**Answer:**

`git blame` shows who last modified each line of a file and in which commit. Useful for tracking down when and by whom a specific change was made.

```bash
# Blame a file
git blame filename.txt

# Output format:
# abc1234 (John Doe 2024-05-01 10:00:00) const x = 5;

# Blame specific lines
git blame -L 10,20 filename.txt

# Ignore whitespace changes
git blame -w filename.txt
```

---

### Q26. How do you rewrite the author of past commits?

**Answer:**

```bash
# Change author of the last commit
git commit --amend --author="New Name <new@email.com>"

# Change author across multiple commits using rebase
git rebase -i HEAD~5
# Mark commits with 'edit', then for each:
git commit --amend --author="New Name <email>" --no-edit
git rebase --continue

# Bulk rewrite (use with care — rewrites all history)
git filter-branch --env-filter '
  OLD_EMAIL="old@email.com"
  NEW_NAME="New Name"
  NEW_EMAIL="new@email.com"
  if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]; then
    export GIT_COMMITTER_NAME="$NEW_NAME"
    export GIT_COMMITTER_EMAIL="$NEW_EMAIL"
  fi
' --tag-name-filter cat -- --branches --tags
```

---

### Q27. What is a fast-forward merge vs a three-way merge?

**Answer:**

- **Fast-forward merge** — When the target branch hasn't diverged. Git simply moves the pointer forward. No merge commit is created.
- **Three-way merge** — When both branches have diverged. Git finds the common ancestor and creates a new merge commit combining both histories.

```bash
# Force a merge commit even when fast-forward is possible
git merge --no-ff feature-branch

# Allow fast-forward (default)
git merge feature-branch
```

---

### Q28. How do you find a deleted file in Git history and restore it?

**Answer:**

```bash
# Find when the file was deleted
git log --all --full-history -- path/to/deleted-file.txt

# Restore the file from the commit before it was deleted
git checkout <commit-before-deletion>^ -- path/to/deleted-file.txt

# Or using the commit hash directly
git show abc1234:path/to/deleted-file.txt > restored-file.txt
```

---

### Q29. What is `git worktree`?

**Answer:**

`git worktree` lets you check out **multiple branches simultaneously** into different directories from a single repository. No need to stash or switch branches.

```bash
# Add a new worktree for a branch
git worktree add ../hotfix-dir hotfix/urgent-fix

# List worktrees
git worktree list

# Remove a worktree
git worktree remove ../hotfix-dir
```

> 💡 **Use case:** Work on a hotfix in a separate directory while keeping your feature branch untouched.

---

### Q30. Explain the difference between `origin/main` and `main`.

**Answer:**

- **`main`** — Your **local** branch. This is what you commit to directly.
- **`origin/main`** — A **remote-tracking branch**. It's Git's local snapshot of what `main` looked like on the remote the last time you fetched/pulled.

```bash
# See the difference
git log main..origin/main    # commits on remote not yet in local
git log origin/main..main    # commits in local not yet pushed

# Sync origin/main reference
git fetch origin

# Update local main to match remote
git pull origin main   # = git fetch + git merge origin/main
```

---

## 🎯 QUICK REVISION CHEATSHEET

| Command | Purpose |
|---|---|
| `git init` | Initialize new repo |
| `git clone <url>` | Copy remote repo |
| `git add .` | Stage all changes |
| `git commit -m "msg"` | Commit staged changes |
| `git push origin main` | Push to remote |
| `git pull origin main` | Fetch + Merge from remote |
| `git branch -b feature` | Create & switch branch |
| `git merge <branch>` | Merge branch into current |
| `git rebase <branch>` | Rebase current onto branch |
| `git stash` | Temporarily save changes |
| `git cherry-pick <sha>` | Apply specific commit |
| `git reset --soft HEAD~1` | Undo last commit (keep staged) |
| `git revert <sha>` | Safe undo (new commit) |
| `git log --oneline --graph` | Visual commit history |
| `git reflog` | Full HEAD movement history |
| `git bisect` | Binary search for bug |
| `git blame <file>` | Line-by-line authorship |
| `git tag -a v1.0 -m "msg"` | Create annotated tag |
| `git submodule add <url>` | Embed another repo |
| `git worktree add <path>` | Multiple branch checkouts |

---

> 💪 **Good luck with your interview tomorrow!**  
> Remember: **Understanding WHY** a command works is more impressive than just memorizing syntax.
