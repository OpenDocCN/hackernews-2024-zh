<!--yml
category: 未分类
date: 2024-05-27 14:55:37
-->

# Forgejo forks its own path forward — Forgejo

> 来源：[https://forgejo.org/2024-02-forking-forward/](https://forgejo.org/2024-02-forking-forward/)

Since its [inception](../2022-12-15-hello-forgejo/), Forgejo (a self-hosted git forge, like GitHub) has been a soft fork of Gitea. Upgrading to it was - and for the time being, remains to be - as simple as [changing the URL from which the release is downloaded](../download/). Over time, the way Forgejo is governed and developed evolved. To be able to provide stable, secure, reliable releases, Forgejo requires a [reasonable effort made at writing tests](https://codeberg.org/forgejo/governance/src/branch/main/PullRequestsAgreement.md) for each change that goes into the code. This has worked out remarkably well, as it caught both regressions in imported code, and mistakes in proposed changes. Furthermore, Forgejo has accepted features and other changes that are not available in Gitea, and has [diverged](#the-hard-forking-process) in other ways already.

Today, Forgejo has a healthy number of people contributing to its main mission:

> 1.  The community is in control, and ensures we develop to address community needs.
> 2.  We will help liberate software development from the shackles of proprietary tools.

To continue living by that statement, a [decision was made](https://codeberg.org/forgejo/governance/issues/58) in early 2024 to become a hard fork. By doing so, Forgejo is no longer bound to Gitea, and can forge its own path going forward, allowing maintainers and contributors to reduce tech debt at a much higher pace, and implement changes - whether they’re new features or bug fixes - that would otherwise have a high risk of conflicting with changes made in Gitea. Simply put, the governance and development models of Gitea and Forgejo diverged over time, and so did their goals. Becoming a hard fork is the culmination of that divergence.

## The hard forking process[](#the-hard-forking-process)

Forgejo has been, since its inception late 2022, a soft fork of Gitea which means it contains all of Gitea, both good and bad, with Forgejo having little control over what it is built on. However, some parts of Gitea were already “hard-forked” before:

Most of these steps were taken to liberate parts of the code base from proprietary solutions, to manage them with free software instead, and in the same process, make it simpler to manage Forgejo-specific changes. All while keeping the impact on the software itself minimal.

## Consequences of becoming a hard fork[](#consequences-of-becoming-a-hard-fork)

As of Forgejo v1.21, Forgejo contains all of Gitea, and that has the benefit of allowing Forgejo to be a drop-in replacement. With the decision to become a hard fork, this will no longer be guaranteed. It will remain possible to upgrade from the latest [Gitea version released](https://github.com/go-gitea/gitea/releases/tag/v1.21.5) at the time of the hard fork, but versions past that will not have such a guarantee.

As such, if you were considering upgrading to Forgejo, we encourage you to do that sooner rather than later, because as the projects naturally diverge further, doing so will become ever harder. It will not happen overnight, it may not even happen soon, but eventually, Forgejo will stop being a drop-in replacement.

The Forgejo API will strive to remain compatible with the Gitea API going forward, after the hard fork. Existing APIs at the time of the fork are public, and changing them is a breaking change, which has to be evaluated very carefully, and not done lightly. Future APIs should similarly be evaluated, and Forgejo will try to remain compatible with Gitea. However, Forgejo contributors shall also use their own judgement whether to implement an API or not, and how - with the previous goals in mind.