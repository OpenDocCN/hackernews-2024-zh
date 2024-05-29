<!--yml
category: 未分类
date: 2024-05-27 14:48:45
-->

# Chris's Wiki :: blog/programming/GitBranchesSocialConstructs

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/programming/GitBranchesSocialConstructs](https://utcc.utoronto.ca/~cks/space/blog/programming/GitBranchesSocialConstructs)

Over on the Fediverse, [I had a half-baked thesis](https://mastodon.social/@cks/111449110724408233):

> A half-baked thesis: branches in Git are a social construct, somewhat enabled by technical features. We talk about things having been done on a branch or existing on a branch, or what branches are what on an intertwined tree of them, even when this is not something you can find in the Git repository.
> 
> (This is since commits aren't permanently associated with a branch; they are merely currently reachable from one or more branches. What branch a multi-head-reachable commit is on is up to us.)

The background on this is more or less Julia Evans' [git branches: intuition & reality](https://jvns.ca/blog/2023/11/23/branches-intuition-reality/), or more exactly a Fediverse discussion between [Julia Evans](https://social.jvns.ca/@b0rk/111445767832607539) and [Mark Dominus](https://mathstodon.xyz/@mjd/111446192179078717) (and Mark Dominus's [I wish people would stop insisting that Git branches are nothing but refs](https://blog.plover.com/prog/git/branches.html)).

This ties into my long standing view that [modern version control systems are a user interface to their underlying mathematics](/~cks/space/blog/tech/VCSUINotMathematics). Git has an internal, mathematical view of what 'branches' are, but very few people actually use this mathematical view; instead we use a variety of 'user interface' views of what branches are and how we think of them. Git supports these user views of branches with various 'porcelain' features.

Some projects using Git actively work to create branches that have a more concrete and durable existence. For instance, [commits on a Go release branch have the branch name in the commit's title](https://go.googlesource.com/go/+log/refs/heads/release-branch.go1.21), which is something the Go project does for a lot of branches that are used in Go development and release. For development branches specifically, this durably marks commits as having been done on the branch even after the branch is merged to the 'main' development branch.

Certainly, how I normally think of Git branches is different from their technical existence, and it differs from branch to branch. For example, in a typical repository I think of the 'main' branch as running all the way back to the creation of the repository, but other branches as only running back to where they split from 'main', despite this not being technically correct.

(Another sign of Git branches as being a bit socially constructed is [how you can rename them (per the comments)](/~cks/space/blog/programming/GitMasterToMainWithLocalChanges).)

PS: There are other VCSes where branches have a more durable existence in the VCS history. These VCSes are neither wrong nor right; my view is that they've taken a different view of both the UI and the mathematics of what 'branches' are in their mathematical version of version control.