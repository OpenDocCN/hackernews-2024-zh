<!--yml
category: 未分类
date: 2024-05-27 13:22:52
-->

# Announcing git bisect-find - Kevin Cox

> 来源：[https://kevincox.ca/2024/05/19/git-bisect-find/](https://kevincox.ca/2024/05/19/git-bisect-find/)

<main class="kevincox-content">

Posted on 2024-04-19

This is a small utility that I wrote to compliment [`git bisect`](https://git-scm.com/docs/git-bisect). `git bisect` is a fantastic tool. In the most basic usage you give it one “good” commit (a commit that doesn’t yet include some property) and at least one “bad” commit (one that does have it). `git bisect` will then guide you through the search, picking commits to test that cut the search space in half. This allows you to efficiently identify the first commit with a particular feature.

However one of the premises of `git bisect` is that you know a good commit. Often times you notice a bug but don’t yet know a good version. Was that broken in the last release? Or is it new? Maybe it was broken unnoticed in the past couple of releases? Sure, you could start checking out various revisions to see if the bug is there, but why do this manually?

This is where [`git bitsect-find`](https://gitlab.com/kevincox/git-bisect-find) comes in. It will take just a bad commit, and step back looking for the first good commit. Basic usage looks like this:

First check out your bad commit:

Then start bisecting with `git bisect-find bad`.

You will be asked to start a regular `git bisect` run, type `y`.

Then a new candidate commit will be checked out. If the candidate commit is bad run `git bisect-find bad` and a new candidate will be checked out. Repeat the test and mark process until a good commit is found.

Once you have a good commit found you are done with `git bisect-find`. Just run `git bisect good` and continue with the regular bisection process.

There are more complete docs [in the README](https://gitlab.com/kevincox/git-bisect-find/-/blob/master/README.md). There are also more features to add (like automatic bisection). However the basics have already been quite useful to me so I figured I would share.

There really isn’t anything too special about the way it works. It just jumps back twice as far each time (following first parents). All state is logged via regular `git bisect` so once you find a good commit all of your `git bisect-find bad` (and `git bisect-find skip`) commands are remembered as regular `git bisect bad` and `git bisect skip` would be.

I do wonder if jumping back by twice as much is optimal. Obviously jumping right to the first commit would reduce the number of `git bisect-find` steps. But would result in more `git bisect`. But `git bisect` is quite efficient. (Not to mention that the first commit is likely not particularly interesting.) At some point I should probably work out which growth pattern results in the shortest number of total steps. However I think the optimal number also depends on the distribution of the age of the target commit. But either way doubling works well enough for this tool to be very helpful for me.

* * *

</main>