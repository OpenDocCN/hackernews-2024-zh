<!--yml
category: 未分类
date: 2024-05-27 14:36:48
-->

# Things to avoid building as an early startup | Numeric

> 来源：[https://www.numeric.io/blog/avoid-building-as-an-early-startup](https://www.numeric.io/blog/avoid-building-as-an-early-startup)

Product development in a new company is defined by scarcity and exploration.  You've got a small team (perhaps no team beyond the founders) and you're building an offering which is still in the process of being defined and discovered.

At Numeric, we made clear, intentional decisions of what not to build which allowed us to make great progress with a small team and ultimately gain traction with our audience. Our velocity was driven substantially by the things we did not do. It's important as product builders to identify those asks and ideas which might slow progress on core value to users. To that end, here are a few of those things which we avoided at Numeric and have been generally happy to have done so.

For context, we are a B2B SaaS company. Our users are finance & accounting teams who use our platform to automate & analyze their financial data as well as to collaborate on closing their books each month.

## Dashboards

People love a dashboard. Or rather, people love the idea of a dashboard.

A splashy top-down view with charts, progress, indicators, statuses, and more looks great in a sales demo. It helps paint the picture of what success looks after engaging on your product. The reality, however, is that these dashboards can be of limited value and difficult to maintain in the early days.

To be clear: I'm referring to dashboards of your own, first-party data. For Numeric, that looks like this:

But we held off on building this for a long time, and had dozens of companies using our platform before we finally put the overview page in place.

This is because for a while user engagement wasn't excellent. We hadn't provided enough core value to get people consistently engaged and coming back. Any dashboards would have been fairly empty. And while a dashboard is good for a sales demo, it won't drive usage.

In short, a dashboard of your own app's data is what I'd call a "2nd degree feature"–its value is derived and dependent on other features of your product being used. It's not a good place to dedicate your early efforts. Put simply, if feature B depends on successful feature A, make A successful first.

## Other 2nd-degree features

### Sharing functionality

Is there something to share yet? Instagram started first by being a great app for taking and modifying photos. The sharing of photos drove its growth, but only after they had built a reason for people to be taking photos there in the first place.

In B2B software, this can look like sharing reports, views, or exports to other people or parts of the company.

As with dashboards, ask: is the content already there? Is there something worth sharing yet?

### Preferences and configuration

When do you want your notifications? Through email, slack, daily, in-time, …? Do you want a dark mode?

Beware user input on this front: in research calls, users will happily surface "blockers" or "needs" along these lines. What they're doing is imagining that they're *already onboarded to your product*, and contriving what problems they might run into at some point. They're doing this both to be helpful and to make sure they don't go through the effort to switch and learn to use your product only to later run into deal-breakers.

It's important to realize that while these proposals may be accurate, they're still second-degree features. Make sure your users know that you understand their perspective, and meticulously keep track of what you're hearing. But don't put the cart before the horse. These features don't drive adoption.

You won't win users by eliminating reasons *not* to use your product–you have to have win them by giving them a good reason *for* using it. You'll even be surprised how many of these objections can be delayed into the future or forgotten entirely if the core value is substantial.

## Table-stakes: billing, user management, etc.

There are a lot of things that may seem to fall into a category of “table-stakes”–things that users just expect a software product to have. Examples include:

*   User management (invite & remove users, avatars)
*   Notification preferences
*   Billing settings (change credit card)
*   Plan / tier selection
*   UI themes, dark mode

Psychologically, these feel like safe things to do. They're easy to design, hard to argue with, and feel like progress. But these can be built later as they're needed. Instead, I recommend you handle these things manually (e.g. changing a credit card) until the burden to do so becomes significant.

Until then, stay focused on things that help you learn about your users.

Coding always takes longer than you expect so don't spend precious time on any tables stakes if you can get away with not including them. And don't assume what your audience will need, as it may be less than you think.

## Permissions

Permissions are messy. They define rules based on the attributes of domain objects, the roles of users, and relationships among these. [This article](https://www.osohq.com/post/ten-types-of-authorization#:~:text=At%20Oso%2C%20we're%20constantly,all%20of%20them%20are%20related.) does a great job of outlining this complexity, and rightly points out that most real applications will have a model which uses multiple types of authorization to meet their needs.

When your product is still early, things are in flux. Features are being built, re-built, removed, and refactored in response to new learnings all the time. This is not ideal when it comes to defining your permissions rules.

At Numeric, we avoided authorization for this exact reason. We knew the changes in the data model were too frequent, and introducing permissions too early would be a disproportionate headache to build, evolve, and maintain.

By the time we did add a permissions model, the model was almost obvious. Our platform's design and data model had coalesced around a few primitives like workspaces, reports, and tasks. With the help of [Oso](https://www.osohq.com/) (both their software and their team), we were able to define a permissions model which was straight-forward enough that non-engineers can understand and discuss our user needs, and what we've built for authorization has been stable and long-lasting.

But I want to reiterate: it was *not obvious* what these primitives were going to be prior. In retrospect they seem obvious, but at times it looked like abstractions were going to develop in a very different way. Had we introduced permissions too early, it might have artificially tied our plans and our minds to a worse model for the platform.

## Mileage may vary

For all of these points, you need to understand and evaluate what makes sense for your business. Focus on what is distinct about your product and your audience. If something I’ve listed above is necessary or part of the core value proposition to your users, don’t skip it.

All of these examples illustrate the same general theme: in a startup, learn about your users and your domain deeply, and stay aggressively focused on learning more and progressing toward product-market fit. It sounds simple, but in the fog of actually building a company it can be surprisingly hard and subtle to apply the idea. It's difficult, but alongside a tight-knit team and a fast iteration cycle this allows you to accomplish great things.