<!--yml
category: 未分类
date: 2024-05-27 14:29:33
-->

# How Much Architecture Is “Enough?”: Balancing the MVP and MVA Helps You Make Better Decisions - InfoQ

> 来源：[https://www.infoq.com/articles/mva-enough-architecture/](https://www.infoq.com/articles/mva-enough-architecture/)

### Key Takeaways

*   Avoid over-investing in the MVA: the critical challenge is to solve the MVP's current challenges while anticipating but not actually solving future challenges. 
*   If the MVP is not successful, the MVA is a wasted effort, but if the MVP is successful, the MVA is essential to the healthy evolution of the product.
*   The MVP and the MVA are like two climbers, tied together with a rope. The MVP leads but cannot get too far ahead or the MVA will hold it back.
*   It also makes no sense for the MVA to get too far ahead of the MVP because feedback about the MVP’s value may invalidate decisions made to create the MVA. 
*   During the development of the initial release, the MVA will be based on assumptions that may not prove true so some overinvestment may happen but this will be corrected once the feedback loop is working.
*   When the MVP evolves based on feedback, the MVA must be reconsidered. This is especially true when the architecture created for one application is reused for another. Applications support different MVPs and therefore meet different requirements.

In a series of previous [articles](https://www.infoq.com/articles/minimum-viable-architecture/), we introduced a concept called the *Minimum Viable Architecture,* or *MVA*. The MVA is the architectural complement to a *Minimum Viable Product* or *MVP*. The MVA balances the MVP by making sure that the MVP is technically viable, sustainable, and extensible over time; it is what differentiates the MVP from a throw-away proof of concept. It is what makes the MVP the seed of a new product.

Extending the MVP concept in an agile context, we view every new product release as a new MVP that extends the previous MVP with additional customer outcomes. The MVP’s corresponding MVA evolves just enough to support the incremental improvements to the MVP. In this fashion, both the product and its associated software architecture evolve in a series of increments. This co-evolution is depicted in Figure 1.

![](img/e586a3c7430617c9924e917c23a7d358.png)

**Figure 1: The MVP and its MVA co-evolve in a series of incremental deliveries**

#### Related Sponsored Content

All this sounds well and good until we start asking fundamental questions like what, really, is "minimum", and how do we know? And similarly, what is "viable"?

In many cases, not a lot of time or effort goes into the answers; teams are asked to quickly deliver a "new" product (aka an MVP) meant to stimulate more customer demand, and they do some technical architectural work (the MVA) to make the MVP releasable.

Both the MVP and its associated MVA are defined by time and resource constraints rather than by a thoughtful analysis of underserved customer outcomes.

But what if we were to take the time to understand those outcomes and shape the MVP and MVA accordingly? What would that look like?

## How do you know if the MVP is minimal?

Products are vehicles for delivering outcomes to customers. The smallest possible increment of a product is one that delivers a single improved outcome. Each release has to deliver at least one improved outcome to some set of users of the product. If it does not improve at least one outcome it is not worth releasing. If it delivers more than one improved outcome it may not be "minimum". Hence, the minimum MVP delivers exactly one improved outcome to some set of users.

If a product release improves more than one outcome for different sets of users of the product, the product organization is missing opportunities to release smaller increments more frequently so that it can gather feedback on the effectiveness of the product release.

An observation that we like to keep in mind when asking whether a product increment is minimum is from the author Antoine de Saint-Exupéry:

> "Perfection is achieved, not when there is nothing more to add, but when there is nothing left to take away."

Teams have a tendency to want to add "just one more thing" to a release, but if the "single outcome rule" is respected, teams will deliver value sooner and get feedback faster than if they delay the release to add that one more thing.

This discussion underscores the value of using outcomes to manage scope, not features. If your MVP isn’t expressed in terms of outcomes it’s harder to manage scope because it’s harder to see which features are needed to improve a targeted outcome, and which ones are merely fluff.

Rarely is an organization certain that an MVP will improve the customer outcome that it is targeting, so MVPs are often expressed in terms of an experiment using techniques borrowed from [Lean UX](https://www.interaction-design.org/literature/article/a-simple-introduction-to-lean-ux#:~:text=Creating%20a%20Hypothesis%20in%20Lean%20UX,-The%20hypotheses%20created&text=There's%20a%20simple%20format%20that,level%20of%20sign%20up%20completions.), such as:

> "We believe that enabling people to [perform some task] will result in improving [some desired outcome]. We will have demonstrated this when we observe [some measurement of that outcome]."

## How do you know if the MVP is viable?

The MVP is viable if the premise about the value it delivers proves to be true, while the costs of developing and maintaining its associated MVA do not exceed its financial benefits. There is only one way to prove this: let actual users/customers use the product and provide feedback. The "real users/customers" are important; internal stakeholders are not sufficient and can suffer from confirmation bias because their input shaped the original ideas about the MVP in the first place. Hand-selected focus groups can fail to represent the needs of the customer base at large because they have been selected by the people who shaped the concept of the MVP and also suffer from confirmation bias.

For some products, it may take a long-ish period to determine the viability of the MVP, weeks, and sometimes even months. Early feedback may be able to quickly ascertain that the MVP is valuable even when it cannot fully determine how valuable. When considering what the MVP should be, it’s worth considering how you will know that it is valuable, and how long it will take you to know.

Viability also includes estimates of financial viability. Without getting too far into financial decision-making processes, the net present value of the cost of developing the product cannot exceed the net present value of the future benefits the product will generate. Developing the MVP and MVA helps the organization determine the cost side of this equation, but they also need to find ways to assess the benefits side as well.

For an MVP to be viable it also has to be supportable over the lifespan of the product. A product has no value to customers if, for some reason, it stops working when the customer needs it. This is where the product’s MVA comes in: when a company delivers an MVP to customers, they are implicitly committing to supporting that MVP until they tell customers that they will no longer do so. The purpose of the MVA is to ensure the supportability of the outcomes delivered by an MVP.

## How do you know if the MVA is minimal?

Together, the MVP and MVA incrementally answer the question "Is our product (and its associated software architecture) viable, e.g. supportable and reliable over the lifetime of the product?" Each MVA attempts to satisfy the Quality Attribute Requirements (QARs) associated with the MVP. If it also satisfies other QARs that are not associated with the MVP, the MVA is not minimum. Put another way, the MVA supports only the outcomes included in the MVP, and it must do so over the expected lifetime of the product, anticipating future needs that *should be* expressed in the QARs.

The critical challenge that the MVA must solve is that it must answer the MVP’s *current* challenges while anticipating but *not actually* solving future challenges. In other words, the MVA must not require unacceptable levels of rework to actually solve those future problems. Some rework is okay and expected, but the words "complete rewrite" mean that the architecture has failed and all bets on viability are off.

As a result of this, the MVA hangs in a dynamic balance between solving future problems that may never exist, and letting technical debt pile up to the point where it leads to, metaphorically, architectural bankruptcy. Being able to balance these two forces is where experience comes in handy.

The typical problem is that the development team *thinks* they know a lot about the future architectural challenges implied by the MVP. This can lead to overinvesting in the MVA and then later having to undo a lot of work. For example, they might say:

> "We *know* we are going to need a messaging mechanism, so let’s develop that now while we are waiting for the business to decide what they really need."

Whether or not they need a messaging mechanism depends on the QARs. If they cannot satisfy the MVP over the expected lifespan of the product, they may be right, but the question they should ask is "Do we need it *now*?"

If the MVP is not successful they will have spent a lot of time and effort developing something that has no use. If they *don’t* develop it, they may find that the MVP will, at some point, become unsupportable and so they will have also incurred a large amount of [technical debt](https://www.infoq.com/articles/technical-debt-tells-you/). Making this decision is critical and one where experience in making trade-offs is important. Some early evidence that the MVP will meet its goals also helps, as it gives the team a sense whether their work on the MVA will be useful.

When planning what will go into the MVA, a development team asks "What QARs do we need to satisfy to reliably deliver the MVP over the long term, and what technical work do we need to undertake?". The architecture doesn’t need to be completely developed to meet those future needs, but it does need to anticipate the potential changes that will need to be made to meet those needs.

## How much architecture do you need in the first MVA?

The development team creates the initial MVA based on their initial and often incomplete understanding of the problems the MVA needs to solve. They will not usually have much in the way of QARs, perhaps only broad organizational "standards" that are more aspirational than accurate. These initial statements are often so vague as to be unhelpful, e.g. "the system must support very large numbers of concurrent users", "the system must be easy to support and maintain", "the system must be secure against external threats", etc.

The development team will also have some initial statements from stakeholders about what they think the system needs to do. Project/product sponsors don’t get to where they are by being shy about making grand projections of the success of the MVP. This can lead to the development team overbuilding the MVA before they get feedback

BUT ... sometimes the project sponsors are right about their projections and even an overbuilt MVA can struggle under high demand from users. And there are some absolutes, even for bare-bones MVAs; most organizations can’t afford to release even the simplest products with poor security, for example.

The reality is that the initial MVA, like the MVP, is based on educated guesses guided by experience. You might have to build something that you suspect could be wasted, but you can’t afford not to. The initial MVA will probably be at least somewhat overbuilt in some areas and underbuilt in others, and you can only know where by creating it, releasing it, and gathering feedback. The MVA should be just enough to get feedback, without:

*   Exposing the organization to unacceptable risk to its business or reputation, or
*   Spending so much time developing and perfecting the MVA to delay important feedback

One thing to watch out for is the cost of undoing initial assumptions/decisions that prove to be incorrect. [Technical debt](https://www.infoq.com/articles/technical-debt-tells-you/) is an unavoidable consequence of the initial MVA, just as some aspects of most MVPs will turn out to be at least partially wrong. That's why you want to get feedback quickly.

Development teams are sometimes tempted to copy the architecture from a successful system as a way to get started, including using vendor solutions. This can be a good approach if you can do it in such a way that you’re not locked in when you find that some aspects of the solution don’t work. Just don’t copy more of the architecture than you need or you may be creating a technical debt balance that you can’t afford. Also, resist the temptation to hold on too long to an initial MVA when the MVP evolves.

## How do you know if the MVA is viable?

Before you release the product increment, you don’t know. You can’t know. You have absolutely no feedback on whether the MVP is useful, and without that, you don’t know whether the MVA is simply waste, or whether it may need to support a set of product capabilities over the lifespan of the product. So the first gate the MVA must pass is the assessment of the value of the MVP.

We believe every product increment is, in essence, a new MVP. The reason for keeping it as small as possible is simple: the larger the MVP, the greater the possibility that it contains things that aren’t valuable. If the goal of the MVA is to ensure the long-term viability of the MVP, keeping the MVP small also helps to avoid bloat in the MVA.

The MVP and its MVA are rather like two climbers tied together with a rope (see Figure 2). The MVP is the lead climber, and the MVA can’t lag too far behind or it will hurt the MVP. Because the MVP is like a lead climber, there is also no point in the MVA getting ahead of the MVP. There is one exception to this: the MVA for the initial release of a system has to start somewhere, and the assumptions the development team uses for the initial MVA may lead to some overinvestment. This will correct itself once the feedback loop is working.

Like a climber exploring an unclimbed summit, the MVP may change their route based on feedback. The two climbers can move in tandem, but this isn’t the best solution; you will learn things in building the MVP that help to inform the decisions you will make to build the MVA (including whether to build it at all - metaphorically, to drop out of the ascent).

Here are some rough heuristics from an earlier [article](https://www.infoq.com/articles/minimum-viable-architecture) that we’ve found useful:

*   "If making (or not making) a decision will affect the product’s viability and sustainability, or if changing the decision would be so costly in terms of money or time that doing so would make the product uneconomic, impractical, or impossible, then that decision must be made as part of the MVA."
*   "Making the architectural decisions transparent helps the organization to better understand why certain choices have been made, which helps them make better decisions about how they can adapt the product to changing market conditions and evolving customer needs."

![](img/435af13eddbf49578d8cfe9ae40c4e67.png)

**Figure 2: The MVP and MVA must remain closely linked to continue being both minimal and viable**

Once you know the MVP is valuable and useful, evaluating the MVA will tell you whether the MVP is viable or whether it is simply an empty promise to its users. Unless the MVA is viable, the MVP will fail to deliver valuable outcomes over the long run. Coming back to the "roped climbers" metaphor, if the MVP is successful it also may sow the seeds for its downfall because it often creates a lot of technical debt that needs to be repaid in a hurry if the product is going to be viable in the long run.

A development team can prevent some of this technical debt build-up if it adheres to a strong [Definition of Done](https://www.infoq.com/articles/definition-of-done-mva/). Most of the remaining MVP-created technical debt needs to be resolved as soon as the development team is confident that the MVP is valuable. Extending the "climber" metaphor, the length of the rope between the two climbers represents the amount of technical debt the development team is willing to incur while still keeping the product viable.

## Responding to changes in the MVP

Sometimes the feedback from delivering an MVP is not what the organization had hoped for. When this happens, the development team must significantly rework the MVP to respond to the feedback. Sometimes it even means more or less starting over and creating a new MVP, or even stopping work entirely if the business concept proves insufficiently valuable.

When the MVP changes significantly, the development team *should* re-evaluate the associated MVA, even if it proved useful in the past. Maybe not all of it is needed. Maybe none of it is needed. Maybe something new and different is needed. Holding on to an old MVA, or one borrowed from another application, is a bit like traveling on a road that’s already built: it’s only valuable if it takes you where you want to go.

When an organization finds that an MVP is flawed and needs to be significantly reconsidered, it needs to reassess the MVA as well, even if they have invested substantially in it. This is true even when just a portion of the MVP turns out to be viable; as the MVA may be largely built to support the portions of the MVP that are no longer needed. Significant scope or vision changes need to trigger a reconsideration of the MVA, asking "Is this still minimal", or "Is this still viable?" and "How will we know?"

## Conclusion

Development teams and their organizations have a hard time reducing product releases to their absolute minimum because they feel that the release will be more valuable if they pack more capabilities into it. It *might* be more valuable, but it will also be delayed. Since the only way to know if a release is valuable is to release it and measure the result, delaying the release to add more capabilities may reduce the value of the release if the added capabilities are not valuable. For this reason, we like to limit incremental MVPs to a single outcome for a single group of users/customers.

It’s also important to consider the MVA alongside its associated MVP. If the MVP is not valuable then neither is the MVA. The MVA needs to support the long-term viability of the MVP, but only if that MVP is valuable. The MVA has to *anticipate* the future evolution of the MVPs but it does not need to solve those future problems yet. And it does not necessarily have to even solve them at the time the MVP is released.

The MVP and MVA are linked, and there is often a lag between the two, to avoid over-investing in the MVA when the MVP falls short of its goals. But it can’t fall too far behind or the MVP may not be supportable in the long run. In addition, the MVA for the initial release of a system has to start somewhere, and the assumptions the development team uses for the initial MVA may lead to some overinvestment. This will correct itself once the feedback loop is working.