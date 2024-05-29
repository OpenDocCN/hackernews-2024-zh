<!--yml
category: 未分类
date: 2024-05-27 14:25:49
-->

# Jeremy Mikkola - Moving git commits between repos

> 来源：[https://jeremymikkola.com/posts/2017_07_15_move_commits_between_git_repos.html](https://jeremymikkola.com/posts/2017_07_15_move_commits_between_git_repos.html)

# Moving git commits between repos

Posted on July 15, 2017

Did you know it is possible to move commits between two unrelated git repositories?

Doing so relies on an interesting property of Git. It doesn’t have a concept of two repositories being the “same,” meaning it allows pushing and pulling between any two repositories, regardless of whether they hold the same project.

(To be fair, git will tell you when the two repositories have no commits in common, but that is a warning you can ignore here.)

Generally just moving the commits to another repo is not enough. It is often important to move the files to a different path inside that repo. Fortunately, git has a command for that. `git filter-branch` allows moving files around in git history (and rewriting history in a variety of other ways).

I’ll demonstrate two use cases you might have: extracting part of one repository out to its own repo, and the reverse, pulling commits in from another repo.

Let’s say you have some subdirectory of a large project that really should be a project on its own. Sure, you could create a new repository and copy over the files, but that doesn’t maintain their commit history. To keep the commit history, we need to do a little bit more work.

Start by creating another copy of your project to work in:

```
git clone existing_project/ new_project/
cd new_project/
```

This clones the existing project from the `existing_project/` directory into the `new_project/` directory. If you don’t have a local copy of `existing_project`, substitute the project’s clone URL.

For safety, we’ll remove the git remote that points to the original project. This will prevent accidentally pushing the modified version back to the original repo.

```
git remote rm origin
```

Now, it’s time to filter the repository to just the subdirectory you want.

```
git filter-branch --prune-empty --subdirectory-filter my_sub_directory/
```

And you are done! Check the results with a quick `ls` and `git log`.

## Pulling commits in from another repo

Doing the reverse and combining two repos is a little more complicated. For this example, the commits will move from a project called `source_repo` and into one called `dest_repo`.

Start by adding a git remote to the destination repository that points to the source:

```
git remote add source ../source_repo
git config remote.source.pushurl "don't push"
```

Setting `remote.source.pushurl` to some random value make pushing fail, effectively making the remote read-only. This prevents accidentally messing up the source repo.

Next, we’ll pull the commits from the source repo into a branch in the destination repo.

```
git fetch source
git checkout source/master
git checkout -b source_master
```

This creates a branch called `source_master` that contains an exact copy of the history of the master branch in the source repository.

Let’s say you want the files from the source repository in a directory called `source_files/`. Like in the other example, `git filter-branch` allows moving the files to the new location.

```
git filter-branch --tree-filter "mkdir -p source_files; git ls-tree --name-only \$GIT_COMMIT | tr '\n' '\0' | xargs -0 -I FILE mv FILE source_files"
```

At this point, I like to use `git log --stat` to sanity check the location of the files in the commits.

Finally, it’s time to combine this history with the original history of this project.

```
git checkout -b source-commits
git rebase master
```

If you prefer, you could merge instead of rebasing.

The `source-commits` branch is ready to be merged into master.