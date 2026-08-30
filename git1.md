# Git — Top 70 Interview Questions
## DevOps Interview Preparation

> **Focus:** Git basics → branching → merge/rebase → remote repositories → conflicts → reset/revert → tags → stash → Git internals → troubleshooting → real-world scenarios.

---

# 1. Git Fundamentals

### 1. What is Git?
**Answer:** Git is a distributed version control system used to track changes in source code and collaborate with multiple developers.

### 2. What is the difference between Git and GitHub?
**Answer:**
- **Git:** Version control software that runs locally.
- **GitHub:** A cloud platform for hosting Git repositories and providing collaboration features such as pull requests, issues, and Actions.

### 3. Why do we use Git?
**Answer:** Git helps us:
- Track code changes
- Maintain history
- Work with branches
- Collaborate with other developers
- Revert changes
- Resolve conflicts
- Maintain different versions of software

### 4. What is a Git repository?
**Answer:** A Git repository is a directory whose contents and history are tracked by Git. It contains the Git metadata inside the `.git` directory.

### 5. What is the `.git` directory?
**Answer:** `.git` contains Git's internal repository data, including commits, branches, references, configuration, and objects.

### 6. What is a working directory?
**Answer:** The working directory is the actual set of files you are currently working on in your local filesystem.

### 7. What is the staging area?
**Answer:** The staging area is an intermediate area where we select changes that should be included in the next commit.

Command:
```bash
git add file.txt
```

### 8. What is a commit?
**Answer:** A commit is a recorded snapshot of changes in the Git repository. It contains information such as the author, timestamp, parent commit, message, and references to the repository state.

### 9. What is the basic Git workflow?
**Answer:**

```text
Working Directory
       |
       | git add
       v
Staging Area
       |
       | git commit
       v
Local Repository
       |
       | git push
       v
Remote Repository
```

### 10. What is `git init`?
**Answer:** `git init` initializes a new Git repository in the current directory.

```bash
git init
```

---

# 2. Basic Git Commands

### 11. What does `git clone` do?
**Answer:** It creates a local copy of an existing remote repository, including its Git history.

```bash
git clone https://example.com/repository.git
```

### 12. What does `git status` do?
**Answer:** It shows the current repository state, including:
- Modified files
- Staged files
- Untracked files
- Current branch
- Other relevant working-tree information

### 13. What does `git add` do?
**Answer:** It moves changes from the working directory into the staging area.

```bash
git add file.txt
```

### 14. What does `git commit` do?
**Answer:** It records the staged changes as a new commit.

```bash
git commit -m "Add login feature"
```

### 15. What does `git push` do?
**Answer:** It sends local commits to a remote repository.

```bash
git push origin main
```

### 16. What does `git fetch` do?
**Answer:** It downloads new commits and references from a remote repository without merging them into your current branch.

```bash
git fetch origin
```

### 17. What does `git pull` do?
**Answer:** `git pull` generally performs a fetch followed by integration of the fetched changes into the current branch. Depending on configuration, that integration may use merge or rebase.

### 18. What is `origin`?
**Answer:** `origin` is the conventional default name given to the remote repository when you clone a repository.

Check it with:

```bash
git remote -v
```

### 19. What is `HEAD` in Git?
**Answer:** `HEAD` is a reference representing the currently checked-out commit/branch position.

For example:

```text
HEAD -> main -> commit
```

### 20. What is `git log`?
**Answer:** It displays commit history.

```bash
git log
```

A compact version:

```bash
git log --oneline
```

---

# 3. Branches

### 21. What is a Git branch?
**Answer:** A branch is a movable reference to a commit that allows developers to work on changes independently.

### 22. Why do we use branches?
**Answer:** Branches allow developers to develop features, fixes, and experiments independently without directly modifying the main development line.

### 23. How do you create a branch?
**Answer:**

```bash
git branch feature/login
```

Or create and switch immediately:

```bash
git switch -c feature/login
```

### 24. How do you switch branches?
**Answer:**

```bash
git switch feature/login
```

Older syntax:

```bash
git checkout feature/login
```

### 25. How do you list branches?
**Answer:**

```bash
git branch
```

Remote branches:

```bash
git branch -r
```

All branches:

```bash
git branch -a
```

### 26. How do you delete a local branch?
**Answer:**

```bash
git branch -d feature/login
```

If the branch has not been merged and you intentionally want to force deletion:

```bash
git branch -D feature/login
```

### 27. How do you delete a remote branch?
**Answer:**

```bash
git push origin --delete feature/login
```

### 28. What is a remote branch?
**Answer:** A remote branch is a reference to a branch on a remote repository, such as:

```text
origin/main
origin/develop
```

### 29. What is a tracking branch?
**Answer:** A local branch can track a remote branch. This establishes a default upstream relationship for commands such as `git pull` and `git push`.

Example:

```bash
git push -u origin feature/login
```

### 30. What is the difference between `main` and `origin/main`?
**Answer:**
- `main` is a local branch.
- `origin/main` is your local remote-tracking reference representing the state of the remote's `main` branch as last fetched.

---

# 4. Merge and Rebase

### 31. What is `git merge`?
**Answer:** Merge combines changes from one branch into another.

Example:

```bash
git switch main
git merge feature/login
```

### 32. What is a fast-forward merge?
**Answer:** A fast-forward merge occurs when the target branch has no divergent commits and Git can simply move its branch pointer forward.

### 33. What is a merge commit?
**Answer:** A merge commit is a commit with multiple parents created when Git combines divergent histories and cannot perform a fast-forward merge.

### 34. What is `git rebase`?
**Answer:** Rebase moves or reapplies commits from one branch onto another base commit, creating a rewritten linear history.

Example:

```bash
git switch feature/login
git rebase main
```

### 35. What is the difference between merge and rebase?
**Answer:**

**Merge:**
- Preserves the existing branch history
- Can create a merge commit
- Does not rewrite existing commits

**Rebase:**
- Creates a cleaner/linear history
- Rewrites commit identities
- Should be used carefully on commits already shared with others

### 36. When would you use merge instead of rebase?
**Answer:** Use merge when preserving the actual branch topology is important or when rewriting shared history would create problems.

### 37. When would you use rebase?
**Answer:** Rebase is useful for updating a private feature branch with the latest target-branch changes and keeping the history linear before integration.

### 38. Why should you avoid rebasing shared public branches?
**Answer:** Rebase rewrites commit history. If other developers already based work on those commits, rewriting them can cause duplicated commits, confusing history, and difficult synchronization.

### 39. What does `git rebase --continue` do?
**Answer:** After resolving a conflict during rebase, it continues applying the remaining commits.

```bash
git add <resolved-file>
git rebase --continue
```

### 40. How do you cancel a rebase?
**Answer:**

```bash
git rebase --abort
```

This attempts to return the branch to its state before the rebase started.

---

# 5. Git Conflicts

### 41. What is a merge conflict?
**Answer:** A merge conflict occurs when Git cannot automatically reconcile competing changes, commonly because different branches modified overlapping parts of the same file.

### 42. How do you resolve a merge conflict?
**Answer:**

```text
1. Run merge/rebase
2. Identify conflicted files
3. Open the files
4. Decide the correct content
5. Remove conflict markers
6. git add <file>
7. Complete merge/rebase
8. Test the application
```

Typical conflict markers are:

```text
<<<<<<< HEAD
Current branch changes
=======
Incoming branch changes
>>>>>>> feature
```

### 43. How do you find conflicted files?
**Answer:**

```bash
git status
```

It lists files that are unmerged.

### 44. What is the difference between a merge conflict and a GitHub pull request conflict?
**Answer:** The underlying Git problem is similar: Git cannot automatically combine changes. A GitHub pull request presents the conflict in a collaboration interface, but the actual resolution still involves Git history and file contents.

### 45. What should you do after resolving a conflict?
**Answer:**
1. Review the resolved file.
2. Stage it.
3. Complete the merge or rebase.
4. Run tests.
5. Push the resulting commits when appropriate.

---

# 6. Reset, Revert, Restore, and Checkout

### 46. What is `git reset`?
**Answer:** `git reset` moves the current branch reference and can also modify the staging area and working tree depending on the option used.

Common modes:

```bash
git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1
```

### 47. What is the difference between `--soft`, `--mixed`, and `--hard` reset?
**Answer:**

**Soft:**
- Moves HEAD
- Keeps changes staged

**Mixed:**
- Moves HEAD
- Keeps changes in working directory
- Unstages them

**Hard:**
- Moves HEAD
- Resets staging area
- Resets working files to the target commit

Be careful with `--hard` because uncommitted changes can be lost.

### 48. What is `git revert`?
**Answer:** `git revert` creates a new commit that reverses the effect of an earlier commit.

```bash
git revert <commit-id>
```

It is generally safer than resetting a shared branch because it does not rewrite existing public history.

### 49. What is the difference between reset and revert?
**Answer:**

**Reset:**
- Moves branch history
- Can rewrite history
- Often suitable for local/private work

**Revert:**
- Creates a new inverse commit
- Preserves existing history
- Better suited to undoing changes on shared branches

### 50. What is `git restore`?
**Answer:** `git restore` is used to restore files in the working tree or staging area.

Example:

```bash
git restore file.txt
```

Unstage a file:

```bash
git restore --staged file.txt
```

### 51. What is the difference between `git checkout` and `git switch`?
**Answer:** `git switch` is specifically designed for changing branches, while `git restore` handles file restoration. Older `git checkout` performs both kinds of operations and more.

For clarity, modern Git commonly uses:

```bash
git switch
git restore
```

---

# 7. Stash, Tags, and Files

### 52. What is `git stash`?
**Answer:** `git stash` temporarily saves uncommitted changes so you can switch context without committing incomplete work.

```bash
git stash
```

### 53. How do you apply stashed changes?
**Answer:**

```bash
git stash apply
```

This applies the stash while keeping it in the stash list.

Or:

```bash
git stash pop
```

This applies the stash and removes it if the application succeeds.

### 54. How do you list stashes?
**Answer:**

```bash
git stash list
```

### 55. What is a Git tag?
**Answer:** A tag is a reference used to mark a specific point in Git history, commonly for releases.

Example:

```bash
git tag v1.0.0
```

### 56. What is the difference between lightweight and annotated tags?
**Answer:**

**Lightweight tag:** A simple reference to a commit.

**Annotated tag:** A Git object containing metadata such as tagger, date, message, and potentially a signature.

For releases, annotated tags are often preferred.

### 57. How do you push a tag to a remote?
**Answer:**

```bash
git push origin v1.0.0
```

Or all tags:

```bash
git push origin --tags
```

### 58. How do you ignore files in Git?
**Answer:** Add patterns to `.gitignore`.

Example:

```gitignore
node_modules/
.env
*.log
```

Important: `.gitignore` does not automatically stop tracking a file that is already committed.

---

# 8. Remote Repositories

### 59. What is the difference between `git fetch` and `git pull`?
**Answer:**

**Fetch:**
- Downloads remote updates
- Does not integrate them into your current branch

**Pull:**
- Fetches remote updates
- Then integrates them into the current branch according to your Git configuration

A safe inspection pattern is:

```bash
git fetch origin
git log --oneline main..origin/main
```

### 60. What is `git push -u origin main`?
**Answer:** It pushes the local `main` branch to the `origin` remote and sets `origin/main` as its upstream tracking branch.

After that, plain:

```bash
git push
```

can use the configured upstream.

### 61. What happens when `git push` is rejected?
**Answer:** A common reason is that the remote branch contains commits your local branch does not have. First inspect the remote changes:

```bash
git fetch origin
```

Then integrate them using an appropriate merge or rebase strategy, resolve conflicts if necessary, test, and push again.

### 62. What is a non-fast-forward push rejection?
**Answer:** It means the remote branch has advanced in a way that would require rewriting or discarding remote history if Git allowed the push. Git rejects the push by default to prevent accidental history loss.

### 63. What is `git remote`?
**Answer:** It manages connections to remote repositories.

Examples:

```bash
git remote -v
git remote add origin <repository-url>
git remote remove origin
```

---

# 9. Git Internals and Advanced Basics

### 64. How does Git store data internally?
**Answer:** Git stores content as objects. The important object types are:
- Blob — file content
- Tree — directory structure
- Commit — snapshot metadata and parent references
- Tag — annotated tag metadata

Objects are identified using cryptographic object IDs.

### 65. What is a Git commit hash?
**Answer:** A commit hash is an identifier derived from the commit's contents and metadata. It allows Git to reference a particular commit.

Example:

```text
a83f91c...
```

Modern Git repositories may use SHA-1 or SHA-256 depending on repository configuration.

### 66. What is `git diff`?
**Answer:** `git diff` shows changes between Git states.

Examples:

```bash
git diff
git diff --staged
git diff main..feature/login
```

### 67. What is `git cherry-pick`?
**Answer:** `git cherry-pick` applies the changes introduced by a specific commit onto the current branch.

Example:

```bash
git cherry-pick abc1234
```

It is useful when you need a particular fix without merging an entire branch.

---

# 10. Real-World DevOps Interview Scenarios

### 68. You accidentally committed a password or API key. What would you do?
**Answer:**
1. Assume the credential is compromised.
2. Immediately revoke/rotate the credential.
3. Remove it from the source.
4. Remove it from Git history if necessary using an appropriate history-rewriting tool/process.
5. Force-push only when the repository policy and coordination allow it.
6. Check whether the secret was exposed in clones, forks, logs, CI systems, or artifacts.
7. Move future secrets to a secure secret-management solution.

**Important:** Simply deleting the secret in a new commit does not remove it from old Git history.

### 69. Your production branch has a bad commit. How would you roll it back?
**Answer:** If the branch is shared, prefer:

```bash
git revert <bad-commit>
git push origin main
```

This creates a new commit that reverses the bad change without rewriting shared history.

For a private/local branch where history has not been shared, reset may be appropriate.

### 70. A developer says, "My branch is up to date," but the pull request says it is behind `main`. How would you troubleshoot it?
**Answer:**

First fetch the latest remote state:

```bash
git fetch origin
```

Then compare:

```bash
git log --oneline feature-branch..origin/main
```

If `origin/main` contains commits missing from the feature branch, update the feature branch using the team's chosen strategy:

```bash
git switch feature-branch
git rebase origin/main
```

or:

```bash
git merge origin/main
```

Resolve conflicts if necessary, run tests, and push the updated branch.

If rebase rewrote commits already pushed to the remote, use:

```bash
git push --force-with-lease
```

only when appropriate and allowed by team policy.

---

# High-Priority Questions to Master First

If your interview is in the next few days, prioritize these:

1. Git vs GitHub
2. Working directory vs staging area vs repository
3. `git init`
4. `git clone`
5. `git add`
6. `git commit`
7. `git push`
8. `git pull`
9. `git fetch`
10. `git status`
11. Branches
12. `git switch`
13. Local vs remote branches
14. `origin` and `origin/main`
15. Merge
16. Fast-forward merge
17. Rebase
18. Merge vs rebase
19. Conflict resolution
20. `git reset`
21. `git revert`
22. Reset vs revert
23. `git restore`
24. `git stash`
25. Tags
26. `.gitignore`
27. Non-fast-forward rejection
28. `git cherry-pick`
29. Commit hash
30. Accidental secret committed
31. Production rollback
32. Branch behind main troubleshooting

---

# Essential Git Commands Cheat Sheet

```bash
# Repository
git init
git clone <url>

# Status and history
git status
git log
git log --oneline
git diff

# Stage and commit
git add .
git add <file>
git commit -m "message"

# Branches
git branch
git branch <branch>
git switch <branch>
git switch -c <branch>
git branch -d <branch>

# Remote
git remote -v
git fetch origin
git pull
git push
git push -u origin <branch>

# Merge
git merge <branch>

# Rebase
git rebase <branch>
git rebase --continue
git rebase --abort

# Undo
git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1
git revert <commit>
git restore <file>
git restore --staged <file>

# Stash
git stash
git stash list
git stash apply
git stash pop

# Tags
git tag
git tag v1.0.0
git push origin v1.0.0

# Cherry-pick
git cherry-pick <commit>

# Remote branch deletion
git push origin --delete <branch>
```

---

# Interview Answer Framework

For Git scenario questions, do not randomly list commands. Explain your reasoning:

```text
1. Identify the current Git state
2. Inspect status/history
3. Fetch the latest remote information if needed
4. Determine whether history is shared
5. Choose merge/rebase/reset/revert based on the situation
6. Resolve conflicts if necessary
7. Test the changes
8. Push using the safest appropriate method
9. Verify the remote state
```

## Critical Rule

Before using a history-rewriting command such as:

```bash
git reset
git rebase
git push --force
```

ask yourself:

> **Has this history already been shared with other people?**

If yes, be extremely careful. For shared branches, prefer history-preserving approaches such as `git revert` where appropriate.

---

# Final Preparation Strategy

For each important question, practice answering in this order:

```text
What is it?
      ↓
Why do we use it?
      ↓
How does it work?
      ↓
Give a simple command/example
      ↓
What can go wrong?
      ↓
How would you troubleshoot it?
```

For example, for **merge vs rebase**, don't stop at the definition. Be ready to explain:
- What each does
- How history differs
- When you would use each
- Why rebasing shared history is risky
- How you resolve conflicts
- What commands you would run

That is the difference between a candidate who has memorized Git commands and a candidate who can actually operate Git in a DevOps environment.
