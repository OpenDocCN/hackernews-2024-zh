<!--yml
category: 未分类
date: 2024-05-27 14:35:09
-->

# Git Worktrees and GitButler

> 来源：[https://blog.gitbutler.com/git-worktrees/](https://blog.gitbutler.com/git-worktrees/)

Last week, I gave a talk at DevWorld with more "[So You Think You Know Git](https://blog.gitbutler.com/git-tips-and-tricks/)" style tips. One of the things I covered in this new talk was `git worktree` and since I've heard it mentioned a lot lately, I thought it was time to do a little writeup explaining it.

The "[worktree](https://git-scm.com/docs/git-worktree?ref=blog.gitbutler.com)" tool in Git helps you work on multiple branches at the same time.

Since GitButler solves the same basic problem, I'll take a minute to explain what `git worktree` does for those who haven't heard of it, and then also to compare it to how GitButler solves the same core issue.

## Simultaneous Branches

So, what is the core issue here? Basically, it's trying to do work on two topics in your code at the same time.

Let's take an example. Let's say you are working on a feature. Your boss interrupts you (🙄, bosses, right?) and says "we need this bug fixed". Or maybe you just notice the bug as you're working. Whatever it is, you need to switch contexts while in the middle of something.

Now you have a few options.

*   You can fix the bug and commit it into your feature branch and try to get them both deployed together.
*   You can stash everything, create a new branch, switch to it, fix the bug, commit and push it, then switch back to what you were working on.
*   You *could* locally clone the repository to a new directory and work on the other branch there, then push it back into the main one and then push *from there* back upstream. 🙄

Surely there's a better way!

## Worktrees

This is the basic use case for `git worktree`. Worktrees allow you to have a separate working directory (and staging area) for each branch you're actively working on.

Let's say we're working on our project in the `my-project` directory and we decide that we want to work on a new branch called `bugfix-123`. We can run the `git worktree add` command and give it a directory to create and checkout that branch into.

```
❯ git worktree add -b bugfix-123 ../my-project-branches/bugfix-123
Preparing worktree (new branch 'bugfix-123')
HEAD is now at 5b67eb2 first commit
```

Now we have two directories that are linked. When we edit files in the `my-project-branches/bugfix-123` directory, it's on the `bugfix-123` branch.

Now, the cool thing is that both checkouts share the same core Git database. So if we commit in either of them, those commits on those branches are visible by *all* of them.

```
❯ pushd ../my-project-branches/bugfix-123/
~/my-project-branches/bugfix-123 ~/my-project

❯ echo 'new commit content' >> README.md; git commit -am 'new commit'
[bugfix-123 2ff5902] new commit
 1 file changed, 1 insertion(+)

❯ popd
~/my-project

❯ git log --oneline bugfix-123
2ff5902 (bugfix-123) new commit
5b67eb2 (HEAD -> main) first commit
```

This can work for lots of branches, and `git worktree` remembers where they all are for your project and what branches they're all currently on:

```
❯ git worktree list
~/my-project                              5b67eb2 [main]
~/my-project-branches/bugfix-123          2ff5902 [bugfix-123]
~/my-project-branches/feature-foobar-baz  b9b9280 [foobar]
~/my-project-branches/feature-new-widget  daf62e4 [new-widget]
```

You can remove ones you're not using anymore with `git worktree remove` or you can also just delete the directory and run `git worktree prune` to remove it from the list.

```
❯ rm -Rf ../my-project-branches/feature-foobar-baz

❯ git worktree list
~/my-project                              5b67eb2 [main]
~/my-project-branches/bugfix-123          2ff5902 [bugfix-123]
~/my-project-branches/feature-foobar-baz  5b67eb2 [foobar] prunable
~/my-project-branches/feature-new-widget  5b67eb2 [new-widget]

❯ git worktree prune

❯ git worktree list
~/my-project                              5b67eb2 [main]
~/my-project-branches/bugfix-123          2ff5902 [bugfix-123]
~/my-project-branches/feature-new-widget  5b67eb2 [new-widget]
```

Now, there are some downsides to this approach too.

*   If you're using an editor like VS Code, you need a new window for each directory/branch.
*   If you have any ignored files like build artifacts, you have to rebuild them for each worktree. This is less of an issue when you're just stashing or switching branches in one directory.
*   The worktrees are separate, so you can easily create merge conflicts between them without knowing.

However, in some cases, worktrees can be a nice way to give yourself a new place to work in a new context without messing up the context you're currently in.

## GitButler vs Worktrees

So how is this different from GitButler's virtual branches? They also provide a way for you to work on multiple branches at the same time, right?

Well, the big difference is that virtual branches use a *single* working directory for *all* the branches.

Let's take a very simple example. Let's say we want to add translations of our README into Spanish and French and we want to open a Pull Request for each one to be reviewed by different people at possibly different times.

### How to do this in Worktrees

With worktrees, we could open a new worktree for each one, commit in each and push up.

First, we create the two new worktrees.

```
❯ git worktree add -b sp ../my-project-branches/spanish
Preparing worktree (new branch 'sp')
HEAD is now at 5b67eb2 first commit

❯ git worktree add -b fr ../my-project-branches/french
Preparing worktree (new branch 'fr')
HEAD is now at 5b67eb2 first commit
```

Now, we go into each directory, make the changes and commit them.

```
❯ cd ../my-project-branches/spanish

❯ echo 'Léeme' > README.es.md

❯ git add README.es.md; git commit -am 'spanish readme'
[sp 4d57bfb] spanish readme
 1 file changed, 1 insertion(+)
 create mode 100644 README.es.md

❯ cd ../french/

❯ echo 'Lis-moi' > README.fr.md

❯ git add README.fr.md; git commit -am 'french readme'
[fr 48df685] french readme
 1 file changed, 1 insertion(+)
 create mode 100644 README.fr.md
```

Now if we go back to our main directory, we can see both branches there and can create PRs from them.

```
❯ cd ../../my-project

❯ git log --oneline --graph --all
* 48df685 (fr) french readme
| * 4d57bfb (sp) spanish readme
|/  
* 5b67eb2 (HEAD -> main) first commit
```

### How to do this in GitButler

In GitButler, it's much simpler. You can simply edit both files, then drag them into separate branch lanes, commit and push them at the same time.

Let's say we just edit the files:

```
❯ echo 'Léeme' > README.es.md
❯ echo 'Lis-moi' > README.fr.md
```

Then we go to our GitButler UI and we see that an anonymous virtual branch has been automatically created, because it saw new changes. We can just drag one of the files into a new virtual branch lane, then commit and push both at the same time.

Notice how much less friction this is. You don't have to create new branches or switch directories. It's a much faster and smoother way to handle this type of change management.

How does this work, exactly?

In essence, we do a `git diff` to see everything that has been changed and then have our own mapping of which changes are owned by which branch. Then when you hit the "commit" button, we run a sort of `git apply` of those changes to what you started with.

It's *sort of* like what `git add -p` does, except that we can keep doing it over and over and keep things separate.

For instance, with the normal Git tooling, we could just add one of the READMEs here and commit and push that, but then doing the same for the second file would require us to reset and then add the other and then change the active branch and commit, etc. It's not simple.

## One Working Directory to Rule them All

So, there are advantages and disadvantages to both approaches, depending on what you want to do.

Since GitButler only has a single working directory, you *cannot* have two branches applied that have conflicting work. You can't have conflicts if you have one copy of each file.

This could be seen as an advantage or a limitation, but we find that it's quite nice. Essentially you're starting with the merged product and then extracting branches out of it.

For instance, lets say that both the `french` and the `spanish` branches are eventually merged. It doesn't matter what order they're merged in, we know they will not only merge cleanly, but we know that when they do both get merged, what we'll end up with is the state of the working directory *when we started*.

So, that's how worktrees and virtual branches approach the same basic problem in rather different ways. Hope this has been helpful!