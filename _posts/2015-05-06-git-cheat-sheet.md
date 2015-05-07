---
layout: post
title: "Git Cheat Sheet"
date: 2015-05-06 23:40:15
categories: git
---

Below are some basic commands for working with GIT repositories using the
command line client. They are useful if you're just starting out with Git, or if
you keep forgetting the exact parameters.
<!-break->
* [Basic Workflow](#basic-workflow)
* [Repository History](#repository-history)
* [Branching](#branching)

### <a name="basic-workflow"></a>Basic Workflow

#### Clone a repository

``` bash
git clone <git-repo-url>
```

#### Repository status (working copy status)

``` bash
git status
```

#### Commit a change
``` bash
git commit -m "Message"
```

Note: Make sure you have added the files to the index first:

``` bash
git add -a .
```

#### Push chnages to the remote repository
``` bash
git push origin master
```

#### Pull changes from the remote repository
``` bash
git pull --rebase origin master
```

Note: Be sure to replace `master` with the branch that you want to update.

#### Switch to a different remote repository
``` bash
git remote remove origin
git remote add origin <new-origin-url>
```



### <a name="repository-history"></a>Repository history

#### View the log (simple case)
``` bash
git log
```

#### View the log in pretty format
``` bash
git log --graph --oneline --all --decorate
```

#### View changes in a commit
``` bash
git diff-tree --no-commit-id --name-only -r bd61ad98
```

#### Reverting changes
``` bash
git checkout -- <name-of-file>
```

Note: The above command brings the specified file to the current index state (if any). To completely revert all changes, you will need to either run 
``` bash
git reset <name-of-file>
git clean -f
```

You can also reset the entire tree to the last commit
``` bash
git reset --hard
```



### <a name="branching"></a>Branching

#### Create a branch
``` bash
git checkout -b feature1
git push -u origin feature1
```

#### View branches
``` bash
git branch -a
```

#### Merge changes from a branch
``` bash
git merge --no-ff <branchname>
```

#### Delete a branch
``` bash
git branch origin :<branch-name>           # deletes the branch from the remote
git branch -d <branch-name>                # deletes the local branch
```


