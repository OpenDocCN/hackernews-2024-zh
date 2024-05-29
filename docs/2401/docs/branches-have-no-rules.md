<!--yml
category: 未分类
date: 2024-05-27 14:26:12
-->

# branches have no rules

> 来源：[https://wizardzines.com/comics/branches-have-no-rules/](https://wizardzines.com/comics/branches-have-no-rules/)

### you might expect git to enforce some rules about branches

some rules you might imagine:

*   you can’t remove commits from a branch, only add them
*   the `main` branch has to stay more less in sync with `origin/main`

But there are no rules.

git character with demon hat: want to do something horrible to your branch? no problem!

### there are literally no rules

commands that you can use to do weird stuff to a branch:

### instead of rules, we have conventions

for example:

*   run `git pull` often to keep your `main` up to date
*   if you’re working with a big team, don’t commit to `main` directly

Illustration of the git demon talking to a nonplussed stick figure with curly hair.

git demon: you’ve just gotta be really careful to not do the wrong thing and not mess up your branch

person: um… thanks?

### our only saviour: the reflog

`git reflog BRANCHNAME`

will show you the history of every change to the branch, so you can always undo

the reflog is a VERY unfriendly UI, but it’s always there.