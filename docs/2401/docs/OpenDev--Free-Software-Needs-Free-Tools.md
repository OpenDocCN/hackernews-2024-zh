<!--yml
category: 未分类
date: 2024-05-27 14:37:41
-->

# OpenDev: Free Software Needs Free Tools

> 来源：[https://opendev.org/](https://opendev.org/)

## What is OpenDev?

OpenDev is a collaboratory for open source software development at scale. Its focus is on code review, continuous integration, and project hosting provided exclusively through open source solutions like Git, Gerrit, Zuul, and Gitea. It also provides a number of peripheral collaboration services (like Mailman mailing lists, IRC bots for notifications and annotating text-based meetings, Jitsi-Meet for videoconference discussions integrated with Etherpad for collective text editing). All of these services are openly operated by the community, and continuously integrated and deployed using OpenDev itself.

## How is development different with OpenDev

OpenDev doesn't use a pull request (or merge request) workflow, like those implemented by GitHub or Gitlab. Instead it follows Gerrit's iterative change proposal workflow, which results in a slightly different experience.

In GitHub or Gitlab, contribution typically starts by forking a personal copy of the original repository, cloning that locally, and pushing one or more commits to that personal fork. Once your code seems ready to merge, you ask the service to create a pull request for your branch into the original repository. The pull request is reviewed, and if accepted your changes get merged into the original repository. If not, you push more commits to your fork and update the request seeking more reviews.

By contrast, contribution with Gerrit starts by cloning the original repository locally. You iterate on development of one or more local commits, and then use git push (or the git-review tool) to propose your commits as a series of changes to the Gerrit code review service. Each change in the series is reviewed, and if it's accepted (and passing tests) then it gets merged into the original repository. If more work is needed, you amend the same commits and push new versions of them for re-review.

The difference is subtle, but significant. In the pull request model, you create a fork of the original repository, push changes to it, and ultimately propose to merge changes back. In the change proposal model, you prepare a change, and propose it. You do not create a complete fork and everyone contributes into the same original repository with what are basically lightweight, ephemeral topic branches. It results in less fragmentation overall, and avoids confusion between the original repository and its numerous forks.

That high-level difference also affects lower-level details. A pull request may contain several commits, and if merged all those commits will appear in the original repository history. In Gerrit, every commit is a separate change (optionally depending on other related changes) for code reviewers to review, so developers squash or amend edits made while developing to represent the desired final state of that change. Each prior revision of a change is still retained by the service, as are all the review comments, so updates can be compared side-by-side for a clear picture of how the change evolved. It generally results in easier code review, but also a cleaner branch history with the commit messages reviewed as an integral part of each change.

In summary, the Gerrit workflow, its user experience and UI are different from the pull request workflow. While it may not be immediately familiar to developers used to pull request workflows, it's worth learning. Long-term benefits outweigh the short-term cost of having to learn a new tool, especially for someone who is going to spend a lot of time developing for that project anyway.

## Integrated Continuous Integration

One key benefit of OpenDev is that it integrates powerful continuous integration features, made possible by the donation of compute resources from our generous infrastructure partners. Test jobs are run when changes are proposed and provide code reviewers with valuable information. Test jobs also run again at merge time, in case recently approved changes introduce an incompatibility, preventing code which doesn't pass tests from merging to the repository.

Advanced Zuul features like speculative execution of tests allow the testing of sequenced changes in parallel, so development velocity is rarely limited by how thorough you want your tests to be. Changes in one Git repository can depend on proposed changes in another repository, allowing integration testing of features actively developed across multiple projects, removing artificial barriers between development teams.

This advanced continuous integration system was developed to sustain the complexity and scale of OpenStack development, one of the three most actively developed open source projects in the world. OpenDev makes this system available to other projects, enabling open development at scale for everyone.

Finally, another key difference between OpenDev and other development infrastructure services like GitHub or Gitlab.com is that it's built purely using open source software. GitHub and Gitlab.com are provided free of charge for open source projects, but they are both implemented using proprietary code. If development of your free and open source software requires interaction with proprietary code, is it truly free (as in freedom)?

It is widely accepted today that using open source technology reduces your reliance on outside parties and enables innovation. It should be obvious that developing software by using open source toolchains has the same effect. Nothing prevents a service provider from changing its terms of service, creating new limitations, blocking access for contributors connecting from specific countries, or even fully removing your project. Proprietary development services create the same form of hard limits, lock-in and dependency that proprietary software does, and prevent open innovation in the development infrastructure space.

OpenDev is entirely built using open source software, but goes one step beyond: it is also openly operated. Even its operation tooling and configuration is open source, lives in Git, and is continuously-deployed. It serves as a clear example of transparent collaboration for systems administration. Like free and open source software, it requires engaging with its community to get the most of it -- OpenDev is not a service you consume, it's a community you join. That can be a bit overwhelming if you just want to focus on development, especially when ready-to-consume services are available. But it's worth it, especially if you want to have a say in what services are provided, or just support the idea of improving open source tools in that space.