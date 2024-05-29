<!--yml
category: 未分类
date: 2024-05-29 13:21:06
-->

# Quality is a hard sell in big tech

> 来源：[https://www.pcloadletter.dev/blog/big-tech-quality/](https://www.pcloadletter.dev/blog/big-tech-quality/)

<main id="skip">

# Quality is a hard sell in big tech

I have noticed a trend in a handful of products I've worked on at big tech companies. I have friends at other big tech companies that have noticed a similar trend: The products are kind of crummy.

Here are some experiences that I have often encountered:

*   the UI is flakey and/or unintuitive
*   there is a lot of cruft in the codebase that has never been cleaned up
*   bugs that have "acceptable" workarounds that never get fixed
*   packages/dependencies are badly out of date
*   the developer experience is crummy (bad build times, easily breakable processes)

One of the reasons I have found for these issues is that we simply aren't investing enough time to increase product quality: we have poorly or nonexistent quality metrics, invest minimally in testing infrastructure (and actually writing tests), and don't invest in improving the inner loop. But why is this?

My experience has been that quality is simply a hard sell in bigh tech.

Let's first talk about something that's an *easy* sell right now: AI everything. Why is this an easy sell? Well, Microsoft could announce they put ChatGPT in a toaster and their stock price would jump $5/share. The sad truth is that big tech is hyper-focused on doing the things that make their stock prices go up in the short-term.

It's hard to make this connection with quality initiatives. If your software is slightly less shitty, the stock price won't jump next week. So instead of being able to sell the obvious benefit of shiny new features, you need to have an Engineering Manager willing to risk having lower [impact](../impact-based-performance-evaluation) for the sake of having a better product. Even if there is broad consensus in your team, group, org that these quality improvements are necessary, there's a point up the corporate hierarchy where it simply doesn't matter to them. Certainly not as much as shipping some feature to great fanfare.

## Part of a bigger strategy?

Cory Doctorow has [said some interesting things](https://pluralistic.net/2023/11/22/who-wins-the-argument/#corporations-are-people-my-friend) about *enshittification* in big tech:

*"enshittification is a three-stage process: first, surpluses are allocated to users until they are locked in. Then they are withdrawn and given to business-customers until they are locked in. Then all the value is harvested for the company's shareholders, leaving just enough residual value in the service to keep both end-users and business-customers glued to the platform."*

At a macro level, it's possible this is the strategy: hook users initially, make them dependent on your product, and then cram in superficial features that make the stock go up but don't offer real value, and keep the customers simply because they really have no choice but to use your product (an enterprise Office 365 customer probably isn't switching anytime soon).

This does seem to have been a good strategy in the *short-term*: look at Microsoft's stock ever since they started cranking out AI everything. But how can the quality corner-cutting work long-term?

## I hope the hubris will backfire

Something will have to give. Big tech products can't just keep getting shittier—can they? I'd like to think some smaller competitors will come eat their lunch, but I'm not sure. Hopefully we're not all too entrenched in the big tech ecosystem for this to happen.

</main>