<!--yml
category: 未分类
date: 2024-05-27 15:00:32
-->

# Backlog size is inversely proportional to how often you talk to customers

> 来源：[https://bitbytebit.substack.com/p/the-size-of-your-backlog-is-inversely](https://bitbytebit.substack.com/p/the-size-of-your-backlog-is-inversely)

I haven’t written much in the past year because every spare hour has been poured into a [venture I started](https://www.jumpcomedy.com/) providing event and ticket management services for comics. It’s been relatively successful with tens of thousands of tickets sold, and happy customers who are referring other customers. This post is a collection of some of the learnings from this time. They are by no means exhaustive but just something that’s on my mind this lazy Sunday morning. This is me breaking the writing cobwebs after a year in hibernation. Bear with me.

As with anything, these are not absolute truths but born out of my experience.

Instead of spending time planning and concocting roadmaps, replace that activity by talking to current or potential customers on how their lives can be improved, and letting that determine your next feature. Injecting the actual customer who uses the software into the development process is key to creating value. The more proxies you have between the developer and the customer, the less the product will meet customer needs.

There is no point to having a large backlog because the bigger the backlog, the higher the unvalidated assumptions, and the lower the chance that it creates any customer value. I have made too many mistakes assuming that something is valuable, when nobody cares about it. A large backlog should be looked at with an extremely high degree of skepticism, as *the size of your backlog is inversely proportional to how often you talk to customers*.

Planning time is best used focusing on how to build the feature, not what to build. The what should come directly from the customer, with the software development process needing a direct physical line to customer - no proxies.

Don’t spend too much time designing UIs because much like backlogs, they are infested with assumptions. Get a basic UI out the door which uses your current understanding of how a customer might use the app. By [following the basics](https://www.uxtoast.com/design-tips/gestalt-principles-in-ui), it’ll be 60% right and that’s all you need. You can hone the rest through customer feedback. If you think you can’t design UIs, you can. We all use consumer apps and are much more in tune with what a UI should look like than we were 20 years ago.

Implementing that feedback depends on how well-designed your UI code is, and focusing on good component design instead of wireframe/visual designs is time well spent. A product’s capacity to implement UI customer feedback is more important than a product’s initial UI, yet we tend to focus far too much on the latter with heavy design up front. Small components and low-level reusability are key in UIs that are easy to change.

Keeping technical debt low allows you to make the changes your customers need you to make. *Speed is a function of technical debt.*

Make it a point to observe the customer when they are using your app. I use [PostHog](https://posthog.com/)’s session replay feature and am amazed watching videos of how customers use the product compared to how I think they use it. You can track all the metrics you want, but there’s something surreal about seeing a user scroll up and down trying to find something, hitting the back button, waiting, trying to click something that isn’t clickable etc. These qualitative measures can complement the quantitative measures that most people track.

I see this as a tangent of [Hyrum’s Law](https://www.hyrumslaw.com/), and if I may be so bold as to suggest a corollary: *What you intend to be not clickable will be clicked.* The question then becomes, what was the user’s intention when they clicked it?

There will be a case where you will wonder what a specific customer is seeing in production. There is no point even trying to recreate this in your test environment; instead invest heavily in account spoofing, i.e., the ability for an admin user to use the app as if they were a specific production user. This makes testing easy, allows you to diagnose issues effectively and reduces the need on test data, which always has a degree of unreliability in it.

You may not be able to do all write operations while spoofing (e.g., updating payment methods), but most operations are read, and even the write ones are reversible. Don’t let this scare you, embrace it.

The first thing a customer seems when they come to your app should have the most relevant information for that customer, and be sparse in content. The tendency to jam too many things on page one increases cognitive load and can waste a potentially great jumping off point. Running various types of experiments on what the engagement numbers here are is well worth the time. *If the user is reaching for the menu on page one, there’s something wrong with your UI.*

Even though [NPS has its criticisms](https://hbr.org/2019/10/where-net-promoter-score-goes-wrong), I’ve found it a good metric to indicate whether customers feel strongly enough about your product to stake their own reputation on it. We all may need to spend ad dollars at some point, but that’s not where we should start. Getting each of your customers happy enough to recommend your product to other customers is a perfectly solid marketing strategy. This also gives us a higher chance of creating sticky customers, as [Airbnbs Brian Chesky put it](https://www.inc.com/salvador-rodriguez/brian-chesky-ges-entrepreneurship.html):

> “It's better to have 100 people love you than a million people that sort of like you, so if you can find 100 people that love your product -- as long as there are more people like them in the world”

I strongly believe in [social proof](https://buffer.com/library/social-proof/) being able to create sticky customers, so focusing on keeping a smaller set of customers happy enough to recommend your product, over casting the net for a larger base of customers is turning out to be a sound strategic decision. It’s also given us better focus as instead of catering and prioritizing a wide array of customer requests, we’ve focused on making sure a smaller set of customers love our product.

[This diagram](https://www.pmi.org/disciplined-agile/process/product-management/mvps-and-mbis) to me was the end of MVPs. The agility and feedback loop that this concept was supposed to encourage died for me when it got contextualized as part of four other constructs that nobody could agree on.

The MVP is not an excuse to deliver a poor customer experience by cutting corners in the face of date pressures, which is what most orgs used them as. It is there to answer specific questions about whether another iteration is needed, and if so, what it could be? This question is rarely asked let alone answered, resulting in Melissa Perri’s [feature factory](https://www.drift.com/blog/build-trap-with-melissa-perri/).

I have found it helpful to define an MVP like this:

An MVP is enough of a feature that can tell me:

*   Does this have enough customer value to continue investing in?

*   What about the technical implementation needs to be improved if we want to scale this feature?

*   Does this feature make a dent in our key goals, or is it a distraction?

These questions cannot be answered without shipping something to production. I could plan and strategize for months on end and get no closer to answering them. They have to be in production in front of users, and “release one” is rarely ever the final state.