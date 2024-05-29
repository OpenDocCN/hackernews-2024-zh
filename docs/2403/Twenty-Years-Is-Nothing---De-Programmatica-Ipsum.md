<!--yml
category: 未分类
date: 2024-05-27 14:36:29
-->

# Twenty Years Is Nothing - De Programmatica Ipsum

> 来源：[https://deprogrammaticaipsum.com/twenty-years-is-nothing/](https://deprogrammaticaipsum.com/twenty-years-is-nothing/)

In a [previous edition of this magazine](/the-winner-takes-it-all/), we argued that English was so pervasive in our industry, nobody even questioned its use anymore. The same can be said of [Git](https://git-scm.com/). It is difficult to imagine that merely twenty years ago, the landscape of source control tools was more diverse, and the choice of one such tool was much more complicated than today. Actually, Git was not even on the map yet. Before debating whether the hegemony of Git is good or bad, let us go back in time for a little while.

In [one of the most famous tangos](https://en.wikipedia.org/wiki/Volver_(song)) of all time, [Carlos Gardel](https://en.wikipedia.org/wiki/Carlos_Gardel) famously sings

> To feel… that life is a breath of fresh air, that twenty years is nothing, that, feverish, the gaze wandering in the shadows seeks and names you.

## Twenty Years Ago

The second edition of “Code Complete” by [Steve McConnell](/steve-mcconnell/) was published in 2004\. On page 668 of this massive 900-page volume, we find the *only* reference to the subject of source control in the whole book: about three quarters of a page long. Nothing else. ChatGPT can easily summarize all of it with one phrase: “Version control software is good and brings several big benefits.” Not a lot to phone home about. We are really far from GitOps at this point.

That same year, almost exactly 20 years ago at the time of the publication of this article, [Subversion 1.0](https://en.wikipedia.org/wiki/Apache_Subversion) saw the light of day. What was Subversion? Probably the shortest lived idea-with-good-intentions in the history of computing. See, Subversion (or `svn`) was supposed to be a better CVS (no, not the [pharmacy](https://www.cvs.com/), but this [other thing](https://en.wikipedia.org/wiki/Concurrent_Versions_System)). “Better than CVS” meant, back in those days, to be transactional ([databases](/issue-61-databases/), anyone?) and to have a somewhat better support for branches. We did not have higher ambitions back then, kids.

Linus Torvalds, however, did have higher ambitions. In 2004, the Linux kernel developers got in an increasingly strong disagreement over the use of [BitKeeper](https://en.wikipedia.org/wiki/BitKeeper), the proprietary, distributed version control system used to manage the kernel source code. So, what is a developer to do? Well, Linus has a tradition of writing the software everybody needs and nobody wants to start. He also has a tradition of [naming things after himself](https://www.wordnik.com/words/git). Legend says that the first version of Git was [written in a couple of weeks](https://www.linuxjournal.com/content/git-origin-story).

## CVS

Never heard of CVS? It was a source control system that Joel Spolsky described in September 2000 as *fine* (emphasis his) in the first item of his eponymous [Joel Test](https://www.joelonsoftware.com/2000/08/09/the-joel-test-12-steps-to-better-code/) for better software:

> I’ve used commercial source control packages, and I’ve used CVS, which is free, and let me tell you, CVS is *fine*.

Yes, the first step for better software was (shocker!) using source control software. (As a side note, I started my professional career as a software developer in 1997, and nope, we did not use source control, not even CVS. Yes, you guessed right: we just saved VBScript files locally and uploaded them via FTP. Too bad if we overwrote changes with one another: you only live once. I had to wait until 2002 to use a source control system for the first time, and for the curious among you, it was [Rational ClearCase](https://en.wikipedia.org/wiki/Rational_ClearCase).)

Never heard of Joel Spolsky? Well, he was the co-creator of Stack Overflow, which I guess you *have* used at some point in your career. 24 years ago, Joel was one of the first influencers of the burgeoning field of software engineering. Think Kelsey Hightower, but with more controversial views. Or Steve Yegge, but with less controversial views.

Speaking about Stack Overflow, here is an example of the state-of-the-art of source control in 2008\. One of the first-ever questions asked on the site, dated September 8th, 2008, asks precisely [what version control system to use](https://stackoverflow.com/questions/49601/is-there-a-barebones-windows-version-control-system-thats-suitable-for-only-one) for a single-developer workflow. (Interestingly, that question was asked precisely at the same time while in the real world, the [2008 financial crisis](https://en.wikipedia.org/wiki/2007%E2%80%932008_financial_crisis#2008_(September)) was breaking havoc. Our industry lives in a bubble, no doubt about that. But I am digressing, once again.)

> I’m trying to find a source control for my own personal use that’s as simple as possible. The main feature I need is being able to read/pull a past version of my code. I am the only developer.

The replies to the question consist of a long catalog of pretty much every version control system known to mankind at that point in time.

## Windows’ Version Control Odyssey

But let us return to the year 2000: a few days before Joel Spolsky published his Joel Test, [Mark Lucovsky](https://www.linkedin.com/in/mark-lucovsky-5280034/) gave a talk titled [“Windows: A Software-Engineering Odyssey”](https://www.usenix.org/legacy/events/usenix-win2000/invitedtalks.html) at the [4th USENIX Windows System Symposium](https://www.usenix.org/legacy/events/usenix-win2000/) in Seattle, Washington. Mr. Lucovsky was a member of the original Windows NT team from 1988 to the mid-2000s. The PowerPoint slides of the talk [are still available online](https://www.usenix.org/legacy/events/usenix-win2000/invitedtalks/lucovsky_html/Lucovsky.ppt) at the time of this writing, and I seriously recommend you take a look at them.

Because part of the “odyssey” was, you guessed it, source control. On slide 14 you can learn that Windows NT 3.51 used an “internally developed” system… which was “on life support” by the time of Windows 2000:

> To keep a machine in synch was a huge chore (1 week to setup, 2 hours per-day to synchronize)

Oops. Not a great way to onboard new team members. Now you know why the [Agile Manifesto](https://agilemanifesto.org/), published in 2001, was so revolutionary. Thanks to [Raymond Chen](https://devblogs.microsoft.com/oldnewthing/), arguably the most important lecturer of Windows history, [we learn the name](https://devblogs.microsoft.com/oldnewthing/20180122-00/?p=97855) of said internally developed system:

> In the early days, Microsoft used a homemade source control system formally called Source Library Manager, but which was generally abbreviated SLM, and pronounced *slime*. It was a simple system that did not support branching.

In slide 24 of Mark Lucovsky’s PowerPoint slides, we learn that Microsoft took the decision to migrate the source code of Windows 2000 to something called “Source Depot”. Raymond Chen [agrees](https://devblogs.microsoft.com/oldnewthing/20180122-00/?p=97855):

> Shortly after Windows 2000 shipped, the Windows source code transitioned to a source control system known as Source Depot, which was an authorized fork of Perforce.

Why Perforce? The choice [had to do](https://softwareengineering.stackexchange.com/a/85891) with the gigantic size of Microsoft Windows’ source code base:

> The justification is perhaps less relevant than it once was, but Perforce tends to perform better on large repositories than Subversion. This is one of the reasons Microsoft acquired a source license to Perforce to build Source Depot; NT’s repository is a monster, and not many products, commercial or otherwise, could handle it.

Mark Lucovsky’s summarized the benefits of Source Depot in two bullet points on slide 24 of his presentation:

> *   New machine setup 3 hours vs. 1 week
> *   Normal sync 5 minutes vs. 2 hours

Is the Microsoft Windows team still using Source Depot today? Apparently not. In 2017, we learned that Microsoft migrated all 300 GB of Windows source code to Git [in an article on Ars Technica](https://arstechnica.com/information-technology/2017/02/microsoft-hosts-the-windows-source-in-a-monstrous-300gb-git-repository/), which contains another gem describing the “Microsoft odyssey” in source control systems:

> Long ago, the company had a thing called SourceSafe, which was reputationally the moral equivalent to tossing all your precious source code in a trash can and then setting it on fire thanks to the system’s propensity to corrupt its database.

(I can confirm. Sadly, I should say.)

Microsoft’s adoption of Git, however, [was not without hurdles](https://arstechnica.com/information-technology/2017/05/90-of-windows-devs-now-using-git-creating-1760-windows-builds-per-day/), and led to the creation of the Git Virtual File System (GVFS) project:

> But Git isn’t designed to handle 300GB repositories made up of 3.5 million files. Microsoft had to embark on a project to customize Git to enable it to handle the company’s scale.

## The Age Of Git

The infatuation of Microsoft with Git reached its peak in 2018, when [it swallowed GitHub](https://news.microsoft.com/2018/06/04/microsoft-to-acquire-github-for-7-5-billion/), the platform that arguably made Git mainstream. Three years prior, sensing *l’air du* temps, they had released [Visual Studio Code](https://en.wikipedia.org/wiki/Visual_Studio_Code), with integrated Git support.

GitHub introduced the concept of Pull Requests to the world [as early as February 2008](https://github.blog/2008-02-23-oh-yeah-there-s-pull-requests-now/), a feature later adopted and adapted by [GitLab](https://docs.gitlab.com/ee/user/project/merge_requests/), [Gitea](https://docs.gitea.com/next/usage/pull-request) (and its recent fork [Forgejo](https://forgejo.org/docs/latest/user/pull-requests-and-git-flow/)), and [BitBucket](https://support.atlassian.com/bitbucket-cloud/docs/tutorial-learn-about-bitbucket-pull-requests/), and which became the bread-and-butter for code reviews during the past 15 years. But the matter of fact is that GitHub also created a paradox in the world of Git: suddenly, a distributed source control system… became centralized. Some are understandably [aghast](https://blog.edwardloveall.com/lets-make-sure-github-doesnt-become-the-only-option) by this state of things.

We are in 2024, and Git is everywhere. The long evolution that led to the Git supremacy in the 2010s and, apparently, also the 2020s, can be summarized as a sequence of open-source programs, one replacing the other: [SCCS](https://en.wikipedia.org/wiki/Source_Code_Control_System) in the 1970s, [RCS](https://en.wikipedia.org/wiki/Revision_Control_System) in the 1980s, CVS in the 1990s, and Subversion in the 2000s. To ensure smooth migration paths, Subversion could import CVS repositories, and Git can import Subversion repositories. But most importantly, version control systems migrated from local-only systems like SCCS and RCS, to client-server architectures (CVS and Subversion), to distributed systems, like Git and others, most notably [Mercurial](https://www.mercurial-scm.org/).

(Speaking about Mercurial, did you know that the Firefox developers recently [decided to drop it](https://glandium.org/blog/?p=4346) and use Git instead?)

These days, we are used to *cloning* an entire project on our computer, after which we can safely plug it off the network and continue writing software in a completely disconnected way. This simple paradigm was utterly and completely unthinkable 20 years ago. And guess what: your local repository also contains the full history of every single change ever made to your project. This was a feature that, naturally, client-server systems could never provide (spoiler alert: the server had the full history, while clients had only the `HEAD`, so to speak).

Git (and its cheap branching facilities) had a lasting impact on developer workflows. Vincent Driessen published in January 2010 a seminal article titled [“A successful Git branching model”](https://nvie.com/posts/a-successful-git-branching-model/) introducing the world to the [controversial](https://web.archive.org/web/20111007075151/http://scottchacon.com/2011/08/31/github-flow.html) concept of git-flow. Why controversial? Well, because most opinions in the software industry are such. Now there is a [GitHub flow](https://docs.github.com/en/get-started/using-github/github-flow) and an [Atlassian Gitflow workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) and many more branching workflows available.

Git repositories have become eventful, with every push, merge, or tag operation triggering a workflow somewhere. A whole industry has sprung up, including names such as [Argo CD](https://argoproj.github.io/cd/), [GitHub Actions](https://docs.github.com/en/actions), [GitLab CI/CD pipelines](https://docs.gitlab.com/ee/ci/pipelines/), and [Gitea Runner](https://about.gitea.com/products/runner/), providing a new level of automation and convenience. The influence of Git is so strong in this space that the term [GitOps](https://www.gitops.tech/) now refers to a whole subset of our industry. But you should not be using branches for deployments, [you have been warned](https://codefresh.io/blog/stop-using-branches-deploying-different-gitops-environments/).

The question is simple now: what comes after Git? At this point, it is probably impossible to challenge the immense popularity of Git. I say “*probably*” because in our industry, it is impossible to predict the future. There are two interesting contenders worthy of mention: [Pijul](https://pijul.org/ "https://pijul.org/"), written in Rust (although the project [started with OCaml](https://github.com/8l/pijul "https://github.com/8l/pijul") a decade ago) or [Fossil](https://fossil-scm.org/ "https://fossil-scm.org/"), written by the creators of SQLite. In this last case, the SQLite team provides [a list of reasons](https://sqlite.org/whynotgit.html) why not to use Git:

> Let’s be real. Few people dispute that Git provides a suboptimal user experience. A lot of the underlying implementation shows through into the user interface. The interface is so bad that there is even a parody site that generates [fake git man pages](https://git-man-page-generator.lokaltog.net/).

While we wait for a better alternative, let us read the `man 7 giteveryday` page and call [git-extras](https://github.com/tj/git-extras), [SourceTree](https://www.sourcetreeapp.com/), [TortoiseGit](https://tortoisegit.org/), [Fugitive](https://github.com/tpope/vim-fugitive), [Codeberg](https://codeberg.org/), and [Magit](https://magit.vc/) to the rescue. It seems that, whether we like it or not, we will probably be storing our source code in Git repositories for the next twenty years.

Cover photo by [Praveen Thirumurugan](https://unsplash.com/@praveentcom?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/a-book-and-a-small-figurine-on-a-desk-KPAQpJYzH0Y?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash).

Continue reading "

[Linus Torvalds](https://deprogrammaticaipsum.com/linus-torvalds/)

" or go back to

[Issue 066: Version Control](/issue/issue-066-version-control)

. Did you like this article? Consider

[subscribing](/newsletter/)

to our newsletter or

[contributing](/contribute/)

to the sustainability of this magazine. Thanks!