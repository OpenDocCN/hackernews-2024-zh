<!--yml
category: 未分类
date: 2024-05-27 14:32:28
-->

# LWN's guide to 2024 [LWN.net]

> 来源：[https://lwn.net/SubscriberLink/954544/666b8433d343147e/](https://lwn.net/SubscriberLink/954544/666b8433d343147e/)

| **This article brought to you by LWN subscribers**Subscribers to LWN.net made this article — and everything that surrounds it — possible. If you appreciate our content, please [buy a subscription](/subscribe/) and make the next set of articles possible. |

By **Jonathan Corbet**
January 2, 2024

The calendar has flipped over into 2024 — another year has begun. Here at LWN, we do not have a better idea of what this year will bring than anybody else does, but that doesn't keep us from going out on a shaky limb and making predictions anyway. Here, for the curious, are a few things that we think may be in store for 2024.

**The kernel community will begin to move away from email** as the core of its development process. Progress will be slow, and many kernel developers will prove strongly resistant to any alternative workflow, often for good reasons, but there will be at least a few developers who are able to get work reviewed, updated, and merged without having to touch a mail client. In a world where even Linus Torvalds is [saying](/Articles/952034/) that the time has come to make a change, the unthinkable stands a good chance of actually coming to pass.

**The next long-term stable kernel will be 6.12**, which will be released on December 1, 2024 (unless Linus Torvalds declines to make a release immediately after the US Thanksgiving holiday, in which case it will be one week later).

**The first user-visible Rust code** will be merged into the kernel, perhaps as soon as the 6.8 release (which will happen in March). That code may not be used on many systems initially, but it still marks an important transition: once Rust is used for user-visible features, the kernel community will no longer have the option of easily dropping support for the language. Merging user-visible Rust code into the kernel will, in other words, be a declaration that the Rust experiment is a success.

As Rust becomes necessary to build a Linux kernel, **the lack of a GCC-based Rust compiler will become a bigger problem**. The [gccrs project](/Articles/954787/) is working to fill that void, but the task is large, the target is moving quickly, and the project has progressed slowly with relatively little support. Somebody is going to have to put some resources into that project if it is to succeed; it is not clear where those resources might come from in 2024, though.

**The enterprise distribution market will be shaken up** in 2024\. Last year, vendors working in this space all apparently came to the conclusion that there was little to be done other than offering clones of Red Hat Enterprise Linux (RHEL), essentially ceding control over that part of the market to one company. As Red Hat makes it harder to compete in that space, though, both vendors and users will increasingly ask themselves why they are bothering. Stable Linux does not need to be a copy of RHEL, and there are a number of interesting approaches that could draw parts of the market away from RHEL in the future.

**Firefox will reverse its longtime decrease in browser share** after gaining a strong impetus from Google's [plan to switch to "Manifest V3"](https://developer.chrome.com/blog/resuming-the-transition-to-mv3/) in the Chrome browser. That transition will make ad blockers harder, if not impossible, to implement in Chrome. Ad blockers, once [called](https://doc.searls.com/2015/09/28/beyond-ad-blocking-the-biggest-boycott-in-human-history/) "<q>the biggest boycott in world history</q>" by Doc Searls, are the only thing that makes the World Wide Web tolerable for many users. Switching browsers is not harder than installing the ad blocker in the first place; many people will find the motivation to make that switch when the alternative is a constant stream of advertisements, trackers, and malware.

Of course, this will all go better if Mozilla positions itself well to be the defender of a friendlier web. The recent [statement from Mozilla CEO Mitchell Baker](https://stateof.mozilla.org/#build-better) suggests that the organization sees AI as a more interesting place to focus its efforts on than saving the world (again) from a browser monopoly. If Firefox languishes while Mozilla pursues shinier projects, an opportunity — perhaps the final one — for Firefox to regain relevance may be lost.

Speaking of AI, **open-source generative AI will see a lot of attention this year**. Partly that is because some of the open-source projects have proved to be surprisingly competitive with the proprietary efforts, but there is more going on than that. Those proprietary platforms are going to spend the year tangled up in high-profile copyright lawsuits and could run into serious trouble. A system based on free software, running on one's own servers, may continue to be useful even if the large, proprietary solutions find themselves restricted or shut down. We will also certainly see signs of open-source models being used in ways (such as the generation of hate speech, for example) that the commercial models, often with good reason, prohibit. The results seem unlikely to be pleasant.

**It will be a big year for BPF**, which is not surprising since the last few years have been as well. Projects like [the extensible scheduler class](/Articles/922405/) seem unlikely to go away; increasing pressure from users may eventually cause its rejection to be reconsidered. Meanwhile, the recently announced [acquisition of Isovalent](https://isovalent.com/blog/post/cisco-acquires-isovalent/) by Cisco may bring new resources to BPF development — or it may, in the way of many corporate acquisitions, succeed in messing up an important BPF-development group.

**The first release of a "free threading" Python** (without the global interpreter lock or GIL) in October will be a qualified success. There will be rough spots and bumpy patches due to the need for two different binary distributions of the language (with and without the GIL) and its extensions, but the overall direction will be looking good. The free-threaded version will not be the default in 2024, but progress in that direction will be visible by the end of the year.

[Goodhart's law](https://en.wikipedia.org/wiki/Goodhart%27s_law), paraphrased, says "when a measure becomes a target, it ceases to be a good measure". **Abuse of metrics will become a bigger problem** in the coming year, continuing a trend that has been underway for some time. Whether the metric is CVE numbers obtained, regression reports filed, commit counts, patches "reviewed", toots boosted, discussion-forum badges obtained, or any of a number of other things, the pursuit of "more" will lead to trouble.

Consider, for example, [this post from Daniel Stenberg](https://daniel.haxx.se/blog/2024/01/02/the-i-in-llm-stands-for-intelligence/) on problems with AI-generated security-bug reports in the curl project. As it becomes easier to crank out such reports (or "Tested-by" tags, etc.), and as people perceive value in increasing their personal scores, the amount of abuse will certainly increase. We will have to develop better defenses to avoid being overwhelmed by this type of spam.

Finally, **the ongoing maintainer crisis will intensify** in 2024. There are many projects in the free-software community that are heavily and widely depended upon, but which receive little support. Those projects, as a result, tend to exhibit slow progress, large amounts of technical debt, security problems, and more. This problem is not new; it is also not hidden to anybody who has been paying attention.

Free software has been a major boon to the corporate world. Any company out there can take advantage of massive amounts of free software with no obligations beyond adhering to the license — and many companies do not even bother with that. Companies can adapt the software to their needs, and they can pool their efforts to minimize the need to reimplement functionality that has been created elsewhere.

This freedom is a good thing; it has created a great deal of benefit for almost everybody involved. But free software does not develop itself; it needs care and support. Often, companies have provided those resources, with the result that we all have access to far more free software than we might have once thought possible. We have all benefited from this massive contribution of resources to our community from the commercial realm.

But companies can be severely short-sighted. It is easy for a manager to justify contributing a driver to the kernel to enable their company's hardware. It is harder to justify contributing resources to the framework within the kernel that makes the driver easy to write, to the rest of the kernel, or to user-space support for that hardware. And it is nearly impossible to get resources to support hardware that the company is no longer selling — even though many users still depend on that hardware.

So companies tend to contribute in ways that create other problems and fail to support the platform as a whole. They hire maintainers in the hope of gaining both skills and some influence over a project, then do not give those maintainers time to do their maintenance work. Companies will often ignore pressing needs in parts of the free-software ecosystem, seeing them as somebody else's problem. As a result, much of the work of keeping our software healthy falls onto the shoulders of the relatively few developers who are able to put some time (perhaps at significant personal cost) into it.

By all appearances, 2024 does not look like a year in which companies have decided to improve support for the software that they depend so heavily upon. That needs to change; free software is not some magical, infinite resource that companies can use to reduce their own development costs without a care for its future. It is a way to share maintenance costs; if it is used as a way to shed those costs entirely, the results will continue to be unpleasant, for both the software and the people who work so hard to keep it going.

Hopefully that vision is too dark, and we will see some progress toward improving the situation. Regardless of how it goes, LWN will be there throughout 2024 to keep you apprised of the state of our community. We hope it is a great year for all of our readers, and thank you for supporting us into our 26th year of operation. It has been a fascinating trip, and we are not done yet.

* * *

(

[Log in](https://lwn.net/Login/?target=/Articles/954544/)

to post comments)