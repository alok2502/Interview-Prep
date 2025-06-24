```
Question : What do you prefer generally git fetch vs git pull
Answer : I generally prefer git pull over git fetch because, in our organization, commits typically trigger a CI/CD pipeline. This pipeline runs automated checks, including vulnerability scans and other validations, which gives us confidence that the incoming code is safe and ready to merge. Since git pull fetches and integrates the changes in one step, it's more streamlined for our day-to-day development.
```

```
Question : What is the difference between git merge and git rebase
Answer : Let me explain the difference between git merge and git rebase with an example. Suppose we have a repository where the main branch has 100 commits. A feature branch is then created from main, and 70 new commits are added to it. Meanwhile, other developers continue working on main, adding another 100 commits. > > Now, if we use git rebase origin/main on the feature branch, Git will reapply the 70 commits from the feature branch on top of the updated 200-commit main history—making the timeline linear and clean. > > On the other hand, if we use git merge origin/main, Git will combine the two histories, preserving the actual sequence of events. So, you'll see the first 100 commits from main, then the 70 feature commits, followed by the new 100 commits from main, along with a merge commit.
```

```
Question : Explain the git branching strategy that you follow in your organization.
Answer : In our organization, we follow a structured branching strategy to support continuous development and stable releases. The core branch we use is called modernization, which acts as the base for all feature and bug-fix work. > > Whenever a new feature needs to be implemented or a bug needs fixing, we create a dedicated branch from modernization. Once the development is complete, a pull request is raised. After code review and validations, our team lead merges the changes back into the modernization branch. > > Periodically, once we've accumulated a stable set of changes, we create a release branch from modernization. This branch is handed over to the QA team for testing and validation. If everything checks out, the code is deployed to production. > > If any issues are found in production, a hotfix branch is created from the release branch. After the fix is applied and tested, it’s merged back into both the modernization branch and any other relevant feature branches to keep everything in sync.

```
```
Question : Explain three challanges that you faced related to git in your previous orgnozation
Answer : > 1. Lack of a Branching Strategy > When I joined the organization, there was no structured Git branching strategy. I introduced a custom version of trunk-based development to streamline our workflow. We designated a main branch for continuous development. For every new feature or bug, a separate branch was created from main, and after completion, merged back via pull requests. Once features were ready for release, we’d create a release branch from main for QA testing. If everything passed QA, the feature would be deployed to production. In case of production issues, we’d create a hotfix branch, then merge the fix into both main and the release branches to maintain consistency.

> 2. No Access Controls in Place > Initially, our Git repositories lacked proper access control. I helped implement RBAC (Role-Based Access Control) by introducing structured roles such as Admin, Reviewer, Maintainer, and Developer. We also enforced policies—for instance, only DevOps engineers could create branches, and only Reviewers could be assigned to approve pull requests. This brought order, improved security, and reduced the chances of unauthorized changes.

> 3. Unnecessary Files Being Committed > I noticed that some developers were pushing unwanted files to version control, like .lock.hcl files. To address this, I introduced and educated the team about using .gitignore. We created a common .gitignore file and documented best practices to ensure only necessary files were tracked, improving code cleanliness and reducing merge conflicts.

```
```
Question : Explain the recent challange that you faced with git and how did you overcome this challange
Answer : One of the key Git-related challenges I faced was the lack of access control in our repositories. Initially, all developers had full permissions, including the ability to create branches. This led to unnecessary branch proliferation and caused our CI/CD pipelines to trigger multiple times unnecessarily, which increased infrastructure costs. > > To address this, I proposed and implemented a Role-Based Access Control (RBAC) system. We defined roles such as Reader, Developer, Reviewer, Maintainer, and Administrator. Access permissions were granted based on these roles—only authorized personnel could create or delete branches, and reviewers were assigned to ensure quality checks before merging. This drastically reduced redundant pipeline runs, improved repository governance, and helped optimize costs.
```

```
Question : How do you resolve merge conflicts
Answer : > As a DevOps engineer, if I encounter a merge conflict—especially when two developers have modified the same block of code—I first identify the contributors involved. Since I may not have the full context of the business logic or intended functionality, I reach out to both developers so they can discuss the conflicting changes directly. Once they agree on the correct version of the code, one of them resolves the conflict locally and pushes the updated changes. After that, we proceed with testing to ensure everything works as expected before merging into the main branch.
```
```
Question : Explain about merge strategies
Answer : > When resolving merge conflicts in Git, two common strategies are ours and theirs. These determine which version of the conflicting code gets preserved. > > - ours strategy: Keeps the changes from the current branch (the one you're on when running the merge), discarding the incoming branch’s changes in the conflict. > - theirs strategy: Does the opposite—it favors the incoming branch’s changes and discards those from the current branch. > > These strategies are particularly useful when you want to override changes in bulk—like during hotfixes or when syncing long-lived branches—without manually resolving each conflict.
```

```
Question : Have you ever used git tags? if yes why?
Answer : Yes, I’ve used Git tags in my current organization. We primarily use them to mark special commits—especially during manual deployments. For example, once we perform a build or promote a version to another environment, we create a tag on that specific commit. This helps us track exactly which code was deployed where, and makes it easy to refer back or roll back if needed.
```

```
Question : How do you combine mutiple commits in a single commit
Answer : Let’s say I’ve made 10 commits in a feature branch, and I want to combine them into a single commit. I use the command: > git rebase -i HEAD~10 > This opens an interactive editor where I mark the first commit as pick and change the rest to squash or s. After saving, Git prompts me to write a new commit message that represents the combined changes. > > Once the rebase is complete, I use: > git push -f > to force-push the rewritten history to the remote branch.
```

```
Question: Can you tell me 10 Git commands you use on a day-to-day basis?
Answer : Here are 10 essential Git commands commonly used in day-to-day DevOps workflows:
- git clone – Downloads a repository from a remote (like GitHub) to your local machine.
- git add – Stages modified files to be committed.
- git commit -m "message" – Saves the staged changes with a descriptive message.
- git pull – Fetches and merges changes from the remote repository to the current branch.
- git push – Uploads local commits to a remote repository.
- git status – Displays the current state of the working directory and staging area.
- git log – Shows the commit history.
- git merge – Integrates changes from one branch into another.
- git diff – Shows the differences between file versions.
- git stash – Temporarily saves changes that are not ready to be committed.
(Optional Advanced Pick):
git checkout or git switch – Useful for switching branches or restoring files
```

```
Question : I want to ignore pushing changes to a file to git, how you can do it
Answer : To prevent Git from tracking specific files or folders, we use a special file called .gitignore. This file lists the paths that Git should ignore during staging and commits.
For example, if I don’t want Git to track a .env file that contains sensitive information, I simply add this line to .gitignore
```
```
Question : What is the use of .git folder
Answer : The .git folder is a hidden directory located at the root of every Git repository. It contains all the metadata and internal data Git needs to manage the project. This includes:
- Commit history
- Branches and tags
- Staging area information
- Repository configuration settings
- Hooks and logs
- The object database, which stores all content as snapshots
In essence, the .git folder is the brain of the repository—without it, the project is no longer recognized as a Git repo.
You can think of it like the Terraform state file, which maintains the current state of your infrastructure—similarly, the .git folder maintains the current and historical state of your codebase.
```

```
Question : Can you restore a deleted .git folder?
Answer : If the .git folder of a repository gets deleted, the project is no longer recognized as a Git repository. You can only restore it if:
- You have a backup of the original .git folder — in that case, simply placing it back in the root restores full Git functionality, including commit history, branches, tags, etc.
- You have the repository cloned elsewhere or pushed to a remote like GitHub — in this case, you can simply clone the repo again
However, you’ll lose any local-only commits, staged files, or uncommitted changes that weren’t pushed or backed up elsewhere.
```

```
Question : If a teammate accidentally commits a Kubernetes secret to the Git repository how would you tackle this setuation and how will you make sure that it doesn't happen in future
Answer : If a teammate accidentally commits a Kubernetes secret to the Git repository, I would take the following steps:
- Immediately remove the secret from the repository with a new commit.
- Since Git tracks history, I would use a tool like BFG Repo-Cleaner or git filter-branch to permanently remove the secret from the commit history.
- I’d assume the secret may have been exposed, so I’d ask the development team to rotate the secret immediately to prevent any potential misuse.
- To prevent similar incidents in the future:
- Educate the team on best practices for handling secrets.
- Set up pre-commit hooks (e.g. with tools like git-secrets or gitleaks) to scan for sensitive data before commits.
- Add sensitive files like secrets.yml or .env to the .gitignore file so they aren’t tracked accidentally.
```





