# Git Interview Questions & Answers

## 📘 Basics

**Q: What is the difference between git fork and git clone?**  
- **git fork**: Creates a copy of a repository on your GitHub (or GitLab, etc.) account—ideal for contributing without write access.  
- **git clone**: Downloads a local copy of any remote repository to your machine for development.

**Q: Describe a scenario where you used git fork instead of git clone.**  
In my previous project, I didn’t have write access to the modernization branch. I forked the repo, cloned my fork locally, created a feature branch, made changes, pushed to my fork, and raised a pull request upstream.

## 🔁 Fetch vs Pull

**Q: What is the difference between git fetch and git pull?**  
- **git fetch**: Downloads new commits into `.git/refs` but does not alter your working directory—useful for reviewing changes first.  
- **git pull**: Runs fetch + merge (or rebase), updating both metadata and working files in one go.

**Q: Which do you generally prefer, git fetch or git pull?**  
I prefer git pull because our CI/CD pipeline runs automated checks on each commit. Pulling and merging in one step fits our workflow better.

## 🔀 Merge vs Rebase

**Q: What is the difference between git merge and git rebase?**  
- **git rebase origin/main**: Replays your commits on top of main, producing a clean, linear history.  
- **git merge origin/main**: Merges histories and adds a merge commit, preserving the actual sequence of development.

## 🌿 Branching Strategy

**Q: Explain your organization's Git branching strategy.**  
1. `modernization` is our core development branch.  
2. Each feature/bug branch stems from it.  
3. After pull request and review, changes are merged back into `modernization`.  
4. A release branch is created for QA/testing.  
5. On deployment success, it goes live; if hotfixes are needed, we branch off release and merge back.

## 💥 Challenges & Problem-Solving

**Q: Share three Git-related challenges faced in your previous organization.**  
1. No branching strategy – introduced trunk-based development with feature, release, and hotfix branches.  
2. Lack of access controls – implemented RBAC with roles: Reader, Developer, Reviewer, Maintainer, Admin.  
3. Unwanted files in the repo – created a shared `.gitignore` and trained the team to exclude lock, config, and secret files.

**Q: Describe a recent Git challenge and how you overcame it.**  
Everyone could create branches, causing CI pipeline overload. I introduced RBAC to restrict branch creation, significantly cutting costs and improving repo governance.

## ⚠️ Merge Conflicts & Strategies

**Q: How do you resolve merge conflicts?**  
1. Identify both authors of conflicting changes.  
2. Discuss resolution with them.  
3. One developer fixes the conflict and pushes.  
4. Validate via tests before merging.

**Q: Explain merge strategies: ours vs theirs.**  
- **ours**: Keeps changes from the current branch, discarding incoming changes.  
- **theirs**: Prefers incoming branch changes, discarding current branch edits.  
Useful in bulk override scenarios (e.g., hotfixes, branch syncs).

## 🔖 Tags & Squashing

**Q: Have you used Git tags? If yes, why?**  
Yes—as part of deployment. Each tagged commit corresponds to a deployed build. Tags help in easy rollbacks and audits.

**Q: How do you combine multiple commits into one?**  
```bash
git rebase -i HEAD~10
```
Then mark the first commit `pick`, the rest `squash (s)`, save, edit the commit message, then:
```bash
git push -f
```

## 🛠️ Daily Commands

**Q: List 10 Git commands you use daily.**  
1. `git clone`  
2. `git add`  
3. `git commit -m "msg"`  
4. `git pull`  
5. `git push`  
6. `git status`  
7. `git log`  
8. `git merge`  
9. `git diff`  
10. `git stash`  
*(Bonus: `git checkout` / `git switch`)*

## ⚙️ Internals & Recovery

**Q: How do you ignore a file from being pushed?**  
Add the path (e.g. `.env`) to `.gitignore`.

**Q: What is the .git folder used for?**  
Contains all metadata: history, refs, staging info, config, hooks, object database—Git’s internal state storage.

**Q: Can you restore a deleted .git folder?**  
Yes—by restoring from backup or recloning the repository. Any unpushed local commits will be lost.

## 🔐 Security & Sensitive Files

**Q: A teammate accidentally pushed a Kubernetes secret—how do you handle it?**  
1. Remove it in a new commit.  
2. Use BFG Repo-Cleaner or `git filter-branch` to purge history.  
3. Rotate the secret immediately.  
4. Add pre-commit hooks (`git-secrets`, `gitleaks`) and update `.gitignore`.

## 🧠 Advanced

**Q: What is git bisect used for?**  
A binary-search tool to identify a commit that introduced a bug—marks commits as “good” or “bad”.

**Q: What does git cherry-pick do?**  
Applies specific commit(s) from one branch to another:  
```bash
git cherry-pick <hash>
```
Useful for grabbing bug fixes without merging entire branches.

**Q: How and why do you use git stash?**  
Temporarily saves uncommitted changes when switching tasks:  
```bash
git stash
git checkout bugfix
git stash pop
```
Handy for WIP preservation.

**Q: What is git reflog?**  
Shows HEAD history (checkouts, commits, rebases)—vital for recovering lost commits.

**Q: What’s the difference between git reset, git revert, and git checkout?**  
- `git reset`: Changes HEAD and optionally staging/workdir (`--soft`, `--mixed`, `--hard`).  
- `git revert`: Creates a new commit that undoes changes—keeps history intact.  
- `git checkout` (or `switch`): Changes branches or restores files, without altering commit history.

## 🧩 Bonus Topics

**Q: Large files**  
Use Git LFS for binary/large file handling.

**Q: Submodules**  
Keep external dependencies separate and update via submodule commands.

**Q: Force-push with lease**  
```bash
git push --force-with-lease
```
Safer than `--force`—avoids overwriting others’ work.
