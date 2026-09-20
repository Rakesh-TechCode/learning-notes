# Git — Quick Recall Notes

## 1. Git Basics

### Git

**Git** is a distributed version-control system used to track code changes and collaborate safely.

### Git installation/version check

Check whether Git is installed and its version:

```bash
git --version
```

---

### `git init`

**Definition:** Initializes a new Git repository in the current project directory.

```bash
git init
```

Creates the hidden `.git/` directory containing Git's repository data.

---

### `git status`

**Definition:** Shows the current state of your working directory and staging area.

```bash
git status
```

Shows:

* Modified files
* Untracked files
* Staged files
* Current branch

---

### `git add`

**Definition:** Moves changes from the working directory to the **staging area**.

```bash
git add file.txt
```

All files:

```bash
git add .
```

**Remember:**

```text
Working Directory → Staging Area
```

---

### `git commit`

**Definition:** Saves the staged changes as a permanent snapshot in Git history.

```bash
git commit -m "Add payment service"
```

**Remember:**

```text
Staging Area → Repository
```

---

### `git log`

**Definition:** Shows commit history.

```bash
git log
```

Short version:

```bash
git log --oneline
```

---

### `git diff`

**Definition:** Shows changes that are modified but **not staged**.

```bash
git diff
```

Think:

```text
Working Directory vs Staging Area
```

---

### `git diff --staged`

**Definition:** Shows changes that are staged but not committed.

```bash
git diff --staged
```

Think:

```text
Staging Area vs Last Commit
```

---

### `git restore`

**Definition:** Discards changes from the working directory.

```bash
git restore file.txt
```

⚠️ The uncommitted changes in that file are discarded.

---

### `git restore --staged`

**Definition:** Removes a file from staging without deleting its changes.

```bash
git restore --staged file.txt
```

Think:

```text
Staged → Unstaged
```

The file's changes remain on your computer.

---

### `git rm`

**Definition:** Removes a tracked file from the working directory and stages its deletion.

```bash
git rm file.txt
```

---

### `git mv`

**Definition:** Moves or renames a tracked file and stages the change.

```bash
git mv old.txt new.txt
```

---

# HEAD

### `HEAD`

**Definition:** `HEAD` points to the commit/branch you are currently working on.

Example:

```text
HEAD → main → latest commit
```

Check it with:

```bash
git log --oneline
```

You'll commonly see:

```text
HEAD -> main
```

Meaning you're currently on `main`.

---

# Undoing Commits

There are two major approaches we covered:

```text
reset
revert
```

The key difference is **whether you rewrite history or create a new commit**.

---

## `git reset`

**Definition:** Moves the current branch/HEAD backward to another commit and can change the staging/working area.

### `git reset --soft`

Moves HEAD backward but **keeps changes staged**.

```bash
git reset --soft HEAD~1
```

```text
Commit
  ↓
HEAD moves back
  ↓
Changes remain STAGED
```

**Use when:** You want to undo a commit but keep everything ready to recommit.

---

### `git reset --mixed`

Moves HEAD backward and **unstages the changes**, but keeps them in your working directory.

```bash
git reset --mixed HEAD~1
```

This is the **default reset mode**.

```text
Commit
  ↓
HEAD moves back
  ↓
Changes become UNSTAGED
```

---

### `git reset --hard`

Moves HEAD backward and **discards the changes**.

```bash
git reset --hard HEAD~1
```

```text
Commit
  ↓
HEAD moves back
  ↓
Changes DISCARDED
```

⚠️ Dangerous because uncommitted work can be lost.

---

# `git revert`

### Definition

Creates a **new commit that reverses the changes introduced by an earlier commit**.

```bash
git revert <commit-id>
```

Example:

```text
A → B → C
```

Revert `C`:

```text
A → B → C → Revert-C
```

History is preserved.

### Interview point

Use **revert** when you want to undo a commit safely without rewriting existing history, especially on shared branches.

---

# Reset vs Revert

| `git reset`                    | `git revert`                |
| ------------------------------ | --------------------------- |
| Moves HEAD/branch backward     | Creates a new commit        |
| Can rewrite history            | Preserves history           |
| Common for local/unshared work | Safer for shared branches   |
| `--soft/mixed/hard`            | Reverses an existing commit |

### One-line recall

> **Reset moves history; revert adds a reversal commit.**

---

# 2. Branching & Merging

## Branch

**Definition:** A branch is an independent line of development that allows you to work on changes without directly modifying another branch.

Typical:

```text
main
  |
  └── feature/payment-api
```

---

### `git branch`

**Definition:** Used to view, create, and delete branches.

View branches:

```bash
git branch
```

Create:

```bash
git branch feature/payment-api
```

Delete:

```bash
git branch -d feature/payment-api
```

---

### `git switch`

**Definition:** Changes your current branch.

```bash
git switch main
```

Create + switch:

```bash
git switch -c feature/payment-api
```

This is the modern preferred command for branch switching.

---

### Create/Delete Branches

Create:

```bash
git switch -c feature/order-api
```

Delete:

```bash
git branch -d feature/order-api
```

Force delete:

```bash
git branch -D feature/order-api
```

`-D` can delete an unmerged branch, so use carefully.

---

### `git checkout`

Older/multi-purpose command.

Switch branch:

```bash
git checkout main
```

Create + switch:

```bash
git checkout -b feature/payment-api
```

**Recall:** We prefer `git switch` for branch operations because its purpose is clearer.

---

# `git merge`

### Definition

Combines changes from one branch into the current branch.

Example:

```bash
git switch main
git merge feature/payment-api
```

Meaning:

> Merge `feature/payment-api` into `main`.

---

# Fast-Forward Merge

### Definition

Occurs when the target branch has no new commits after the feature branch was created.

```text
main
 A
  \
   B → C
```

Merge:

```text
A → B → C
```

No separate merge commit is required.

---

# Three-Way Merge

### Definition

Occurs when both branches have developed independently.

Example:

```text
        B → C  feature
      /
A
      \
        D → E  main
```

Git combines the histories and normally creates a merge commit.

---

# Merge Conflict

### Definition

A conflict occurs when Git cannot automatically determine which changes should be kept.

Common example:

Both branches modify the same lines:

```text
<<<<<<< HEAD
Payment processing started
=======
Payment validation started
>>>>>>> feature/order-api
```

Git asks you to decide.

---

# Conflict Resolution

Basic workflow:

```bash
git status
```

Open the conflicted file.

Choose/edit the correct final content.

Then:

```bash
git add <file>
```

Finally:

```bash
git commit
```

### Recall

```text
Conflict
   ↓
Fix file manually
   ↓
git add
   ↓
git commit
```

We practiced this with `PaymentService.java` and resolved the conflict by keeping both required lines.

---

# 3. GitHub / Remote Repository

## GitHub

**Definition:** GitHub is a cloud platform that hosts Git repositories and provides collaboration features such as remote repositories, pull requests, code reviews, and issues.

### Git vs GitHub

**Git:**

> Version-control tool running locally.

**GitHub:**

> Online platform for hosting/collaborating on Git repositories.

Simple:

```text
Git = Version Control
GitHub = Remote Collaboration Platform
```

---

# GitHub Repository

### Definition

A repository on GitHub is the remote copy of your project's Git repository.

Example:

```text
Local Repository
      ↕
GitHub Repository
```

---

# Connect Local Repo to GitHub

Add a remote:

```bash
git remote add origin <github-url>
```

`origin` is simply the conventional name given to the remote repository.

---

# `git remote`

### Definition

Manages connections to remote repositories.

View remote names:

```bash
git remote
```

---

# `git remote -v`

### Definition

Shows the remote repository URLs used for fetching and pushing.

```bash
git remote -v
```

Example:

```text
origin  https://github.com/... (fetch)
origin  https://github.com/... (push)
```

---

# `git push`

### Definition

Uploads your local commits to a remote repository.

```bash
git push
```

Think:

```text
Local Repository → GitHub
```

---

# `git push -u origin main`

### Definition

Pushes the local `main` branch to the remote `origin` and establishes an upstream/tracking relationship.

```bash
git push -u origin main
```

After this, usually:

```bash
git push
```

is enough.

---

# Local `main` → `origin/main`

This is important.

```text
main
```

= your **local branch**

```text
origin/main
```

= your local reference to the **remote main branch on GitHub**

Conceptually:

```text
Your Mac                 GitHub

main  ───────────────→  origin/main
```

We will study this properly in the next section.

---

# Push `.txt` File

We practiced pushing a simple text file:

```text
practice.txt
```

Workflow:

```bash
git add .
git commit -m "Add practice text file"
git push
```

---

# Verify on GitHub

After pushing:

1. Open the GitHub repository.
2. Refresh the repository page.
3. Confirm the file/project is visible.

---

# Push Java/Spring Boot Project

We practiced the complete workflow:

```text
Spring Boot project
       ↓
.gitignore
       ↓
git init
       ↓
git add .
       ↓
git commit
       ↓
git remote add origin
       ↓
git push -u origin main
```

Your `Testing_Practice` project was successfully pushed.

---

# Verify Clean Working Tree

```bash
git status
```

Ideal output:

```text
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

### Meaning

There are no uncommitted changes, and your local branch matches its tracked remote branch.

---

# 4. `.gitignore` ⭐

## `.gitignore`

**Definition:** A file that tells Git which **untracked files/directories should be ignored**.

Example:

```gitignore
target/
*.iml
.idea/
.DS_Store
```

---

# Why use `.gitignore`?

To prevent unnecessary/generated/environment-specific files from entering Git.

For Java projects:

```text
Source code       → commit ✅
pom.xml           → commit ✅
target/           → ignore ❌
IDE files         → ignore ❌
OS files          → ignore ❌
```

---

# `target/`

Maven generates `target/` during builds.

Example:

```text
target/
├── classes/
├── test-classes/
├── surefire-reports/
└── *.jar
```

These are generated from your source code.

Therefore:

```gitignore
target/
```

means:

> Don't track Maven's generated `target` directory.

---

# `*.iml`

`.iml` files are IDE-specific IntelliJ project files.

```gitignore
*.iml
```

means:

> Ignore every file ending in `.iml`.

---

# `.idea/`

IntelliJ IDEA project configuration directory.

```gitignore
.idea/
```

means:

> Ignore the `.idea` directory.

---

# `.DS_Store`

A macOS-generated file containing Finder folder metadata.

```gitignore
.DS_Store
```

means:

> Don't track macOS `.DS_Store` files.

---

# IDE / Generated Files

Common examples:

```gitignore
.idea/
*.iml
.classpath
.project
.settings/
```

These are generally environment/IDE-specific rather than application source code.

---

# `.gitignore` Patterns

Common patterns:

```gitignore
target/
```

→ ignore directory named `target`

```gitignore
*.iml
```

→ ignore all `.iml` files

```gitignore
*.log
```

→ ignore all `.log` files

```gitignore
.idea/
```

→ ignore `.idea` directory

You don't need to memorize every pattern. Understand the concept.

---

# `.gitignore` Does NOT Delete Files

Very important:

```text
.gitignore
     ↓
does NOT delete files
```

If you have:

```text
target/
```

on your Mac, it remains there.

Git simply doesn't track it.

---

# How `git add .` Respects `.gitignore`

When you run:

```bash
git add .
```

Git checks `.gitignore`.

For example:

```text
pom.xml              → stage ✅
src/                 → stage ✅
target/              → ignore ❌
Testing_Practice.iml → ignore ❌
```

That's exactly what happened in our `Testing_Practice` project.

---

# Already-Tracked Files vs `.gitignore`

Important interview point:

`.gitignore` primarily prevents **untracked** files from being added.

If a file is **already tracked**, adding it to `.gitignore` does not automatically untrack it.

You would need:

```bash
git rm --cached <file>
```

For a directory:

```bash
git rm -r --cached <directory>
```

The file remains on your computer but is removed from Git tracking.

---

# Java/Maven `.gitignore` — Quick Template

For your Spring Boot projects, a basic version can be:

```gitignore
target/

.idea/
*.iml

.classpath
.project
.settings/

.DS_Store

*.log
```

---

# 🔥 30-Second Git Mental Model

Remember this flow:

```text
              Git Workflow

Working Directory
       │
       │ git add
       ↓
Staging Area
       │
       │ git commit
       ↓
Local Repository
       │
       │ git push
       ↓
GitHub / Remote Repository
```

And when changes come from GitHub:

```text
GitHub
   │
   │ git pull
   ↓
Local Repository / Working Tree
```

### Most important commands so far

```bash
git init
git status
git add .
git commit -m "message"
git log --oneline

git diff
git diff --staged

git restore <file>
git restore --staged <file>

git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1

git revert <commit>

git branch
git switch -c feature/name
git switch main
git merge feature/name

git remote -v
git push
git push -u origin main

git status
```
