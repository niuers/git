# Table of Contenst

* [Rebase](#rebase)
* [Fetch submodule within a git repository](#fetch-submodule-within-a-git-repository)
* [Create a Tag](#create-a-tag)
* [Remote branch](#remote-branch)
* [git branch](#git-branch)

# Rebase
## What is rebase?
* Rebase is one of two Git utilities that specializes in integrating changes from one branch onto another. The other change integration utility is **git merge**. Merge is always a forward moving change record.
* Rebasing is the process of moving or combining a sequence of commits to a new base commit.
* From a content perspective, rebasing is changing the base of your branch from one commit to another making it appear as if you'd created your branch from a different commit. Internally, Git accomplishes this by creating new commits and applying them to the specified base. It's very important to understand that even though the branch looks the same, it's composed of entirely new commits.


## Why do we want to use rebase?

## When do we want to use rebase?
* Rebasing is most useful in the context of a feature branching workflow.

## A rebase based worflow

## References
1. [Atlassian Git Tutorial](#https://www.atlassian.com/git/tutorials/rewriting-history/git-rebase)

## Rebase remote feature branch

```
git branch -D develop //this will remove your local develp repository
git fetch //update references 
git checkout develop //change to develop branch, but because you deleted, this command will also download the origin/develop
git rebase -p origin/master
```

at this step you can have some conflits, so resolve then and git add FILES THAT HAD CONFLITSand git rebase --continue

Now check if everything steel working after rebase, if yes

```
git push -f origin develop
```

# Fetch submodule within a git repository

```
git submodule init
git submodule update
git submodule foreach git pull origin master
```

If you see modified submodule, update it with: `git submodule update`

# Create a Tag
```
git tag -a new_tag_name -m your_comment
git push --tags
```

# Remote branch

## Create a remote branch
```
git checkout -b local-branch-name
git push <remote-name> <local-branch-name>:<remote-branch-name>
```

or simply:

```
git push <remote-name> <branch-name>
```
where remote-name usually is 'origin'

* After created the remote branch, the other people need to update their branch information by:
```
git fetch origin
git branch --remote  # shows the newly created branch
```
## Delete remote branch
```
git push origin --delete <branchName>
```

## Remote information
* show remote name: `git remote`

* Display following information on a remote: `git remote show origin`
  * fetch/push url
  * HEAD branch
  * remote branches
  * local push/pull configuration

## Track a remote branch
* “Tracking” is essentially a link between a local and remote branch. When working on a local branch that tracks some other branch, you can git pull and git push without any extra arguments and git will know what to do.
* If you have already created the branch and want to make the current branch track a remote branch:
```
git branch --set-upstream <local-branch-name> <remote-name>/<remote-branch-name>, e.g. git branch --set-upstream test origin/test 
git remote show origin: this will display the latest configuration of pull/push for your local branches
```
* Push a branch and automatically set a tracking
```
git push -u origin test
```

* Track a remote branch from someone else
```
git checkout -t origin/feature #creates and checks out "feature" branch that tracks "origin/feature"
```

# git branch
* Create a new branch with specific commit: 
```
git branch new_branch_name commit_hash
```

## Push
* push to a specific remote branch

```
git push remote-name <local-branch-name>:<remote-branch-name>
```

## Pull
```
git pull <remote-name> <remote-branch-name>
#for example
git pull origin test
```

## Log

* Show branches, tags in `git log`

```
git log --oneline --decorate
```

* Inspect the commit at the tip of the `master` branch

```
git log -1 master
```

# GIT Merge

* `git merge feature`
  * Incorporates changes from the named commits (since the time their histories diverged from the current branch) into the current branch. This command is used by `git pull` to incorporate changes from another repository and can be used by hand to merge changes from one branch into another.
  * The command will replay the changes made on the feature branch since it diverged from master until its current commit on top of master, and record the result in a new commit along with the names of the two parent commits and a log message from the user describing the changes.
 
* To create a merge commit even when the merge resolves as a fast-forward, use option: `--no-ff`

## Amend the most recent commit's message
```
git commit --amend
```

## Use wildcard in git command
* Put the file argument in single quote. e.g., 
```
git add 'src/pattern*'
```


# Rebase and Merge

* The first thing to understand about `git rebase` is that it solves the same problem as `git merge`. Both of these commands are designed to integrate changes from one branch into another branch—they just do it in very different ways.

## Example

* Consider what happens when you start working on a new feature in a dedicated branch named `feature`, then another team member updates the master branch with new commits.

### Use merge
* Merge in two steps:

```
git checkout feature
git merge master
```

* Merge in one step:
```
git merge master feature
```

* Merging is nice because it’s a non-destructive operation. The existing branches are not changed in any way. This avoids all of the potential pitfalls of rebasing. 
* On the other hand, this also means that the feature branch will have an extraneous merge commit every time you need to incorporate upstream changes. If master is very active, this can pollute your feature branch’s history quite a bit. While it’s possible to mitigate this issue with advanced git log options, it can make it hard for other developers to understand the history of the project.

### Use rebase

* Rebase feature branch onto master branch

```
git checkout feature # git checkout <branch>
git rebase master  # git rebase <upstream>
```

* All changes made by commits in the current branch that are not in <upstream> are saved to a temporary area. This is the same set of commits that would be shown by `git log <upstream>..HEAD` (or `git log HEAD`, if `--root` is specified). The current branch is reset to <upstream>.

* The commits that were previously saved into the temporary area are then reapplied to the current branch, one by one, in order. Note, the commit hashes changed after applying to the current branch. 
* It is possible that a merge failure will prevent this process from being completely automatic. You will have to resolve any such merge failure and run `git rebase --continue`. 
  * Another option is to bypass the commit that caused the merge failure with `git rebase --skip`. 
* To check out the original <branch> and remove the `.git/rebase-apply` working files, use the command `git rebase --abort` instead.

* This moves the entire feature branch to begin on the tip of the master branch, effectively incorporating all of the new commits in master. But, instead of using a merge commit, rebasing re-writes the project history by creating brand new commits for each commit in the original feature branch. 
* The major benefit of rebasing is that you get a much cleaner project history. 
  * First, it eliminates the unnecessary merge commits required by `git merge`. 
  * Second, as you can see in the above diagram, rebasing also results in a perfectly linear project history—you can follow the tip of feature all the way to the beginning of the project without any forks. This makes it easier to navigate your project with commands like `git log`, `git bisect`, and `gitk`.
  
* But, there are two trade-offs for this pristine commit history: safety and traceability. 
  * If you don’t follow the **Golden Rule of Rebasing**, re-writing project history can be potentially catastrophic for your collaboration workflow. 
  * And, less importantly, rebasing loses the context provided by a merge commit—you can’t see when upstream changes were incorporated into the feature.

* Interactive rebasing: 
  * Typically, this is used to clean up a messy history before merging a feature branch into master
  * begin an interactive rebasing session, pass the `i` option to the `git rebase` command:
  ```
    git checkout feature
    git rebase -i master
  ```
  * Eliminating insignificant commits like this makes your feature’s history much easier to understand. This is something that git merge simply cannot do.

### The Golden Rule of Rebasing

* The golden rule of git rebase is to never use it on public branches. 
  * Since rebasing results in brand new commits, Git will think that your re-based master branch’s history has diverged from everybody else’s. So, before you run git rebase, always ask yourself, “Is anyone else looking at this branch?” If the answer is yes, take your hands off the keyboard and start thinking about a non-destructive way to make your changes (e.g., the `git revert` command). Otherwise, you’re safe to re-write history as much as you like.

#### Force pushing
* If you try to push the rebased master branch back to a remote repository, Git will prevent you from doing so because it conflicts with the remote master branch. But, you can force the push to go through by passing the `--force` flag, like so:
```
# Be very careful with this command!
git push --force
```

* This overwrites the remote master branch to match the rebased one from your repository and makes things very confusing for the rest of your team. So, be very careful to use this command only when you know exactly what you’re doing.

* One of the only times you should be force-pushing is when you’ve performed a local cleanup after you’ve pushed a private feature branch to a remote repository (e.g., for backup purposes). This is like saying, “Oops, I didn’t really want to push that original version of the feature branch. Take the current one instead.” Again, it’s important that nobody is working off of the commits from the original version of the feature branch.

### Walkflow Walkthrough
* The first step in any workflow that leverages `git rebase` is to create a dedicated branch for each feature. This gives you the necessary branch structure to safely utilize rebasing

* Local Cleanup
  * One of the best ways to incorporate rebasing into your workflow is to clean up local, in-progress features. By periodically performing an interactive rebase, you can make sure each commit in your feature is focused and meaningful. This lets you write your code without worrying about breaking it up into isolated commits—you can fix it up after the fact.

  * When calling `git rebase`, you have two options for the new base: The feature’s parent branch (e.g., master), or an earlier commit in your feature. 
    * We saw an example of the first option in the "Interactive Rebasing" section. 
    * The latter option is nice when you only need to fix up the last few commits. For example, the following command begins an interactive rebase of only the last 3 commits.
      ```
      git checkout feature
      git rebase -i HEAD~3
      ```
    * By specifying HEAD~3 as the new base, you’re not actually moving the branch—you’re just interactively re-writing the 3 commits that follow it. Note that this will not incorporate upstream changes into the feature branch.
  
  * If you want to re-write the entire feature using this method, the `git merge-base` command can be useful to find the original base of the feature branch. The following returns the commit ID of the original base, which you can then pass to git rebase:
  `git merge-base feature master`
  
  * This use of interactive rebasing is a great way to introduce git rebase into your workflow, as it only affects local branches. The only thing other developers will see is your finished product, which should be a clean, easy-to-follow feature branch history.
  * There is no git merge alternative for cleaning up local commits with an interactive rebase.

* Incorporating upstream changes into a feature
  * Keep in mind that it’s perfectly legal to rebase onto a remote branch instead of master. This can happen when collaborating on the same feature with another developer and you need to incorporate their changes into your repository.
  * Note that this rebase doesn’t violate the Golden Rule of Rebasing because only your local feature commits are being moved—everything before that is untouched.
  * By default, the `git pull` command performs a merge, but you can force it to integrate the remote branch with a rebase by passing it the `--rebase` option.

* Reviewing a Feature With a Pull Request
  * If you use pull requests as part of your code review process, you need to avoid using `git rebase` after creating the pull request. As soon as you make the pull request, other developers will be looking at your commits, which means that it’s a public branch. Re-writing its history will make it impossible for Git and your teammates to track any follow-up commits added to the feature.
    * Any changes from other developers need to be incorporated with `git merge` instead of `git rebase`.
    * For this reason, it’s usually a good idea to clean up your code with an interactive rebase before submitting your pull request.

* Integrating an approved feature
  * After a feature has been approved by your team, you have the option of rebasing the feature onto the tip of the master branch before using `git merge` to integrate the feature into the main code base.
  * This is a similar situation to incorporating upstream changes into a feature branch, but since you’re not allowed to re-write commits in the master branch, you have to eventually use `git merge` to integrate the feature. However, by performing a rebase before the merge, you’re assured that the merge will be fast-forwarded, resulting in a perfectly linear history. This also gives you the chance to squash any follow-up commits added during a pull request.
   * If you’re not entirely comfortable with `git rebase`, you can always perform the rebase in a temporary branch. That way, if you accidentally mess up your feature’s history, you can check out the original branch and try again. For example:
   
   ```
    git checkout feature
    git checkout -b temporary-branch
    git rebase -i master
    # [Clean up the history]
    git checkout master
    git merge temporary-branch
    ```

* Don’t rebase a branch that multiple people have access to! Only rebase branches in your fork. 
  * As I’ve already mentioned, rebasing is a destructive operation. If you’re doing it in your repository, which only you can access, then there’s no issue. 
  * If you rebase a branch that other people have access to, you’re going to run into trouble. So only rebase your own branches, and **only push those rebased branches to your own fork**.


# Reset, Checkout, and Revert
## Main Components of Git Repository 
* the working directory
* the staged snapshot
* the commit history

## Commit-Level Operations
### `git reset`

* Move a branch backwards for 2 commits
```
git checkout hotfix
git reset HEAD~2
```
  * The two commits that were on the end of hotfix are now dangling commits, which means they will be deleted the next time Git performs a garbage collection. In other words, you’re saying that you want to throw away these commits.
  
* Alter the staged snapshot and/or the working directory
  * `--soft` – The staged snapshot and working directory are not altered in any way.
  * `--mixed` – The staged snapshot is updated to match the specified commit, but the working directory is not affected. This is the default option.
  * `--hard` – The staged snapshot and the working directory are both updated to match the specified commit. 

* It’s easier to think of these modes as defining the scope of a git reset operation
 * These flags are often used with HEAD as the parameter. 
   * For instance, `git reset --mixed HEAD` has the affect of unstaging all changes, but leaves them in the working directory. 
   * On the other hand, if you want to completely throw away all your uncommitted changes, you would use `git reset --hard HEAD`. 
 * These are two of the most common uses of `git reset`.
* Be careful when passing a commit other than HEAD to git reset, since this re-writes the current branch’s history. As discussed in The Golden Rule of Rebasing, this a big problem when working on a public branch.

### git checkout
* Internally, all the above command does is move HEAD to a different branch and update the working directory to match. Since this has the potential to overwrite local changes, Git forces you to commit or stash any changes in the working directory that will be lost during the checkout operation. Unlike git reset, git checkout doesn’t move any branches around.
* You can also check out arbitrary commits by passing in the commit reference instead of a branch. This does the exact same thing as checking out a branch: it moves the HEAD reference to the specified commit. 
  * For example, the following command will check out out the grandparent of the current commit:
  `git checkout HEAD~2`
  
* This is useful for quickly inspecting an old version of your project. However, since there is no branch reference to the current HEAD, this puts you in a detached HEAD state. This can be dangerous if you start adding new commits because there will be no way to get back to them after you switch to another branch. For this reason, you should always create a new branch before adding commits to a detached HEAD.

### git revert
* Reverting undoes a commit by creating a new commit. This is a safe way to undo changes, as it has no chance of re-writing the commit history. 
  * For example, the following command will figure out the changes contained in the 2nd to last commit, create a new commit undoing those changes, and tack the new commit onto the existing project.
  ```
  git checkout hotfix
  git revert HEAD~2
  ```

* Contrast this with git reset, which does alter the existing commit history. For this reason, git revert should be used to undo changes on a public branch, and git reset should be reserved for undoing changes on a private branch. 
* You can also think of git revert as a tool for undoing committed changes, while `git reset HEAD` is for undoing uncommitted changes.

## File-Level Operation
* Instead of operating on entire snapshots, this forces them to limit their operations to a single file.
### git reset

* When invoked with a file path, `git reset` updates the staged snapshot to match the version from the specified commit. For example, this command will fetch the version of foo.py in the 2nd-to-last commit and stage it for the next commit:
```
git reset HEAD~2 foo.py 
```
* As with the commit-level version of git reset, this is more commonly used with HEAD rather than an arbitrary commit. 
  * Running `git reset HEAD foo.py` will unstage `foo.py`. The changes it contains will still be present in the working directory.
* The --soft, --mixed, and --hard flags do not have any effect on the file-level version of git reset, as the staged snapshot is alwaysupdated, and the working directory is never updated. 

### git checkout
* Checking out a file is similar to using git reset with a file path, except it updates the working directory instead of the stage. Unlike the commit-level version of this command, this does not move the HEAD reference, which means that you won’t switch branches.

* For example, the following command makes foo.py in the working directory match the one from the 2nd-to-last commit:
```
git checkout HEAD~2 foo.py 
```

* Just like the commit-level invocation of git checkout, this can be used to inspect old versions of a project—but the scope is limited to the specified file.

* If you stage and commit the checked-out file, this has the effect of “reverting” to the old version of that file. Note that this removes all of the subsequent changes to the file, whereas the git revertcommand undoes only the changes introduced by the specified commit.

* Like git reset, this is commonly used with HEAD as the commit reference. For instance, git checkout HEAD foo.py has the effect of discarding unstaged changes to foo.py. This is similar behavior to git reset HEAD --hard, but it operates only on the specified file.



# Refs and Reflogs
Commit hash (SHA-1)
This acts as the unique ID for each commit. It is the most direct way to reference a commit.
git rev-parse branch-name
resolve a branch, tag, or another indirect reference into the corresponding commit hash
Refs: 
A ref is an indirect way of referring to a commit. You can think of it as a user-friendly alias for a commit hash. This is Git’s internal mechanism of representing branches and tags. Refs are stored as normal text files in the .git/refs directory. To change the location of the master branch, all Git has to do is change the contents of the refs/heads/master file. Similarly, creating a new branch is simply a matter of writing a commit hash to a new file. This is part of the reason why Git branches are so lightweight compared to SVN.
Directories under .git/refs:
heads
remotes
tags
Specify the refs examples:
git show some-feature: using short name
git show refs/heads/some-feature: using full name
Packed Refs
For large repositories, Git will periodically perform a garbage collection to remove unnecessary objects and compress refs into a single file for more efficient performance. You can force this compression with the garbage collection command:
git gc 
This moves all of the individual branch and tag files in the refs folder into a single file called packed-refs located in the top of the .git directory.  On the outside, normal Git functionality won’t be affected in any way. But, if you’re wondering why your .git/refs folder is empty, this is where the refs went.
Special Refs
HEAD – The currently checked-out commit/branch. The HEAD ref can contain either a symbolic ref, which is simply a reference to another ref instead of a commit hash, or a commit hash. This is how Git knows the branch that is currently checked out. 
If you were to switch to another branch, the contents of HEAD would be updated to reflect the new branch. But, if you were to check out a commit instead of a branch, HEAD would contain a commit hash instead of a symbolic ref. This is how Git knows that it’s in a detached HEAD state. 
FETCH_HEAD – The most recently fetched branch from a remote repo.
ORIG_HEAD – A backup reference to HEAD before drastic changes to it.
MERGE_HEAD – The commit(s) that you’re merging into the current branch with git merge.
CHERRY_PICK_HEAD – The commit that you’re cherry-picking.
These refs are all created and updated by Git when necessary. For example, The git pull command first runs git fetch, which updates the FETCH_HEAD reference. Then, it runs git merge FETCH_HEAD to finish pulling the fetched branches into the repository. Of course, you can use all of these like any other ref, as I’m sure you’ve done with HEAD.
Refspecs
A refspec maps a branch in the local repository to a branch in a remote repository. This makes it possible to manage remote branches using local Git commands and to configure some advanced git push and git fetch behavior. Refspecs give you complete control over how various Git commands transfer branches between repositories. They let you rename and delete branches from your local repository, fetch/push to branches with different names, and configure git push and git fetch to work with only the branches that you want. 
A refspec is specified as [+]<src>:<dst>. The <src> parameter is the source branch in the local repository, and the <dst> parameter is the destination branch in the remote repository. The optional + sign is for forcing the remote repository to perform a non-fast-forward update.
Refspecs can be used with the git push command to give a different name to the remote branch.
git push origin master:refs/heads/qa-master: It pushes the master branch to the origin remote repo like an ordinary git push, but it uses qa-master as the name for the branch in the origin repo.
git push origin :some-feature: deleting remote branches some-feature by put empty <src> branch
git push origin --delete some-feature: same as above, deleting remote branches some-feature by put empty <src> branch. New in v1.7
git fetch fetches all of the branches in the remote repository by default. The reason for this is the following section of the .git/config file:
[remote "origin"]
    url = https://git@github.com:mary/example-repo.git
    fetch = +refs/heads/*:refs/remotes/origin/*
 
The fetch line tells git fetch to download all of the branches from the origin repo. To fetch only the master branch, change the fetch line to match the following:
[remote "origin"]
    url = https://git@github.com:mary/example-repo.git
    fetch = +refs/heads/master:refs/remotes/origin/master
 
You can also configure git push in a similar manner. For instance, if you want to always push the master branch to qa-master in the origin remote, you would add following line to config file:
 push = refs/heads/master:refs/heads/qa-master
Relative Refs
You can also refer to commits relative to another commit. The ~ character lets you reach parent commits. 
git show HEAD~2: displays the grandparent of HEAD
However merge commits have more than one parent, there is more than one path that you can follow. For 3-way merges, the first parent is from the branch that you were on when you performed the merge, and the second parent is from the branch that you passed to the git merge command. The ~ character will always follow the first parent of a merge commit. If you want to follow a different parent, you need to specify which one with the ^ character.

git show HEAD^2: returns the second parent of HEAD if HEAD is a merge commit

git show HEAD^2^1: displays the grandparent of HEAD(assuming it’s a merge commit) that rests on the second parent.


Reflog
The reflog is Git’s safety net. It records almost every change you make in your repository, regardless of whether you committed a snapshot or not. You can think of it is a chronological history of everything you’ve done in your local repo.
git reflog: display reflog
The HEAD{<n>} syntax lets you reference commits stored in the reflog. You can use this to revert to a state that would otherwise be lost.For example, things lost with git reset.
git checkout HEAD@{1}




# Summary
|Command|Scope|Common Use Cases
`git reset`	Commit-level	Discard commits in a private branch or throw away uncommited changes
`git reset`	File-level	Unstage a file
`git checkout`	Commit-level	Switch between branches or inspect old snapshots
`git checkout`	File-level	Discard changes in the working directory
`git revert`	Commit-level	Undo commits in a public branch
`git revert`	File-level	(N/A)

