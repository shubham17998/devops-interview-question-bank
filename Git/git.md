# Git Interview Questions

## Basics
Q: What is the .git directory and what does it contain?
Ans: The hidden directory at the root of a Git repository that stores everything Git needs: the object database (commits, trees, blobs), refs (branches, tags), the index (staging area), HEAD, config, and hooks. Deleting it removes all Git history — the working files remain, but it stops being a Git repo.

Q: Explain the difference between the Git directory, working directory, and staging area.
Ans: The **Git directory** (`.git/`) is the actual repository — committed history and metadata. The **working directory** is the checked-out files on disk that you actually edit. The **staging area** (index) is an intermediate snapshot — changes you've `git add`ed are staged there, ready to be included in the next commit, distinct from both the working directory (unstaged edits) and the last commit.

Q: What is .gitignore used for?
Ans: A file listing patterns for files/directories Git should never track or show as untracked (build artifacts, dependency folders, secrets, OS/editor cruft) — keeps `git status` clean and prevents accidentally committing generated or sensitive files.

Q: How do you check if a file is tracked by Git, and how do you start tracking it?
Ans: `git ls-files <file>` (or check `git status` — untracked files appear under "Untracked files"). To start tracking it: `git add <file>` followed by a commit.

Q: What does `git diff` do?
Ans: Shows line-by-line differences. With no arguments, it shows unstaged changes (working directory vs. index). `git diff --staged`/`--cached` shows staged changes vs. the last commit. `git diff commit1 commit2` compares two commits.

Q: What does `git status` show?
Ans: The current state of the working directory and staging area relative to the last commit: which files are staged, unstaged, or untracked, and which branch you're on (including ahead/behind info relative to its upstream).

## Merge & Rebase
Q: What is a Git merge conflict and how do you resolve it?
Ans: A conflict occurs when Git can't automatically reconcile changes to the same lines/file made in two branches being merged. Git marks the conflicting sections in the file with `<<<<<<<`, `=======`, `>>>>>>>` markers — you manually edit the file to the desired final content, remove the markers, `git add` the resolved file, then complete the merge with `git commit` (or `git rebase --continue` if mid-rebase).

Q: Difference between Git Reset and Git Revert?
Ans: `git reset` moves the branch pointer backward (optionally altering the working directory/index too), effectively rewriting history — safe only on local/unpushed commits. `git revert` creates a *new* commit that undoes the changes of a previous commit, preserving history — safe to use on already-pushed/shared branches since it doesn't rewrite anything.

Q: Difference between Git Rebase and Git Stash?
Ans: `git rebase` replays commits from one branch onto another, rewriting commit history to create a linear sequence. `git stash` temporarily shelves uncommitted working-directory changes (without committing them) so you can switch context (e.g., branches) cleanly, then reapply them later with `git stash pop` — entirely unrelated operations serving different purposes.

Q: What Git merge strategies are you familiar with?
Ans: **Fast-forward** (no divergent history, branch pointer just moves forward), **Recursive/ort** (the default 3-way merge strategy for divergent branches, creates a merge commit), **Squash merge** (combines all commits from a branch into a single new commit on the target branch), and **Rebase-then-merge** (replay commits onto the target first, resulting in a fast-forward merge with linear history).

## Branching
Q: What Git branching strategies are you familiar with (Git flow, GitHub flow, Trunk-based, GitLab flow)?
Ans: **Git Flow** — long-lived `develop`/`main` branches plus feature/release/hotfix branches; structured but heavier, suits scheduled releases. **GitHub Flow** — simple: branch off `main`, PR, merge, deploy; suits continuous deployment. **Trunk-Based Development** — everyone commits to `main` (or very short-lived branches) frequently, using feature flags to hide incomplete work; suits high-velocity CI/CD teams. **GitLab Flow** — adds environment branches (`staging`, `production`) or upstream/release branches on top of GitHub Flow for more explicit deployment tracking.

Q: How do you check if a branch has already been merged?
Ans: `git branch --merged <target-branch>` lists branches already fully merged into `<target-branch>` (safe to delete); `git branch --no-merged` lists ones that aren't.

Q: How do you remove a remote branch?
Ans: `git push origin --delete <branch-name>` (or the older syntax `git push origin :<branch-name>`).

## Remote & History
Q: What is the difference between `git fetch` and `git pull`?
Ans: `git fetch` downloads new commits/refs from the remote into your local remote-tracking branches, without touching your working branch. `git pull` does a fetch immediately followed by a merge (or rebase, with `--rebase`) into your current branch — it actually updates your working branch, `fetch` alone does not.

Q: How do you remove a file from Git without removing it from the filesystem?
Ans: `git rm --cached <file>` — unstages/untracks the file in Git while leaving it on disk (commonly followed by adding it to `.gitignore` so it isn't re-added accidentally).

Q: How do you squash multiple commits into one?
Ans: Interactive rebase: `git rebase -i HEAD~N` (N = number of commits to squash), then change `pick` to `squash` (or `s`) on all but the first of the commits you want combined, save, and provide the combined commit message when prompted.

Q: How do you discard local uncommitted changes?
Ans: `git checkout -- <file>` (or `git restore <file>` in newer Git) for specific files; `git checkout .` / `git restore .` for all tracked files; add `-u`/`git clean -fd` to also remove untracked files if needed.

Q: How do you discard local commits?
Ans: `git reset --hard <commit>` (e.g., `git reset --hard HEAD~1` to drop the last commit) — moves the branch pointer back and discards the commits and their changes entirely. Only safe for commits that haven't been pushed/shared.

## Behavioral
Q: Why did you choose Git as your Source Code Management tool?
Ans: Representative answer: Git is distributed (every clone is a full history, no single point of failure, works offline), fast for branching/merging, the de facto industry standard (huge tooling/ecosystem/hosting support — GitHub/GitLab/Bitbucket), and its staging area and branching model support flexible workflows (feature branches, trunk-based dev, GitOps) that fit modern CI/CD practices well.

## Hands-on Exercises

### Exercise 1: My First Commit
**Objective:** Learn how to commit changes in a Git repository.
**Steps:**
1. Create a new directory and initialize it as a Git repository.
2. Create a file called `file` with the content "hello commit".
3. Commit the file.
4. Verify the commit was recorded.

**Solution:**
```
mkdir my_repo && cd my_repo
git init
echo "hello commit" > file
git add file
git commit -a -m "It's my first commit. Exciting!"
git log
```

### Exercise 2: Branching
**Objective:** Learn how to work with Git branches.
**Steps:**
1. In an existing repository, create a new branch called `dev`.
2. Modify a file and commit the change on `dev`.
3. Verify the new commit exists only on `dev`, not on `master`.

**Solution:**
```
echo "master branch" > file1
git add file1
git commit -a -m "added file1"
git checkout -b dev
echo "dev branch" > file2
git add file2
git commit -a -m "added file2"

# Verify
git log             # shows two commits on dev
git checkout master
git log             # shows only one commit
```

### Exercise 3: Squashing Commits
**Objective:** Learn how to squash multiple commits into one.
**Steps:**
1. Create a file with the content "Mario" and commit it.
2. Change the content to "Mario & Luigi" and commit again.
3. Verify there are two separate commits.
4. Squash the two commits into one.

**Solution:**
```
echo "Mario" > new_file
git add new_file
git commit -m "New file"

echo "Mario & Luigi" > new_file
git commit -a -m "Added Luigi"

git log             # verify two commits

git rebase -i HEAD~2
# change "pick" to "squash" on the second commit line, save,
# then provide a commit message for the squashed commit
```
> If `git rebase -i HEAD~2` fails with "invalid upstream", the second commit is the repo's root commit — use `git rebase -i --root` instead.
