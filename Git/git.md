# Git Interview Questions

## Basics
1. What is the .git directory and what does it contain?
2. Explain the difference between the Git directory, working directory, and staging area.
3. What is .gitignore used for?
4. How do you check if a file is tracked by Git, and how do you start tracking it?
5. What does `git diff` do?
6. What does `git status` show?

## Merge & Rebase
7. What is a Git merge conflict and how do you resolve it?
8. Difference between Git Reset and Git Revert?
9. Difference between Git Rebase and Git Stash?
10. What Git merge strategies are you familiar with?

## Branching
11. What Git branching strategies are you familiar with (Git flow, GitHub flow, Trunk-based, GitLab flow)?
12. How do you check if a branch has already been merged?
13. How do you remove a remote branch?

## Remote & History
14. What is the difference between `git fetch` and `git pull`?
15. How do you remove a file from Git without removing it from the filesystem?
16. How do you squash multiple commits into one?
17. How do you discard local uncommitted changes?
18. How do you discard local commits?

## Behavioral
19. Why did you choose Git as your Source Code Management tool?

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
