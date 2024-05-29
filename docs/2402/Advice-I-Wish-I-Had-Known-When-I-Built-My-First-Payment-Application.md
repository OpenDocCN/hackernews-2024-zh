<!--yml
category: 未分类
date: 2024-05-27 15:03:44
-->

# Advice I Wish I Had Known When I Built My First Payment Application

> 来源：[https://news.alvaroduran.com/p/advice-i-wish-i-had-known-when-i](https://news.alvaroduran.com/p/advice-i-wish-i-had-known-when-i)

If the world were static, our confidence about what we believe would only grow.

If the world were static, our beliefs would be exposed to a wider and wider arrange of experiences. In time, we would be able to tell if those beliefs were false, because only those likely to be true would survive.

But we don’t live in a static world.

That’s fantastic news if you’re building a startup. If the world were static, startups don’t have a chance. The world is dynamic: technology comes and goes, and companies rise and fall.

Yet, most people tend to believe that the world is static, at least when it comes to their opinions. And they’ve got a point! Opinions about human nature, for example, belong in a static world, because **human nature never changes**. But you can’t trust your opinions about things that change. That includes pretty much everything else.

> When experts are wrong, it’s often because they’re experts on an earlier version of the world.
> 
> — Paul Graham, [How to Be an Expert in a Changing World](https://paulgraham.com/ecw.html)

A changing world is an interesting thing. A startup, though, is a changing company in a changing world. And that’s more interesting. **When a startup changes in a good way, we say that it scales**.

Early on, startups grow by [doing things that don’t scale](https://paulgraham.com/ds.html?ref=creatorscience.com). Startups feed on opportunities that incumbents can’t see, because incumbents [don’t know how to count that low](https://www.lesswrong.com/posts/koGbEwgbfst2wCbzG/i-don-t-know-how-to-count-that-low).

But over time, startups must grow. Scalability is about setting up the conditions for business success.

All startups start small, but only successful startups scale.

* * *

*Welcome to Money In Transit, the newsletter bridging the gap between payments strategy and execution. I’m [Alvaro Duran](https://www.linkedin.com/in/alvaroduranbarata/).*

*Recently, we’ve looked at [the limitations of building money software on top of relational databases](https://news.alvaroduran.com/p/classic-databases-arent-paying-off), [why choosing a programming language is about business requirements](https://news.alvaroduran.com/p/nobody-gets-fired-for-choosing-java), and [a primer on the domain of payment applications](https://news.alvaroduran.com/p/pizza-place-payments), among others. They’re all free to read.*

*Want to be notified when there’s a new post? Click on the magic button below.*

* * *

When it comes to software, a belief that used to be right, but now isn’t, is that CPU power is a huge constraint. When you can replicate your system on many machines, how much can a single machine sustain is no longer relevant.

Engineers can scale their systems with redundant copies of their data. But if they do that, they lose access to a single source of truth. Data, not CPU, becomes the biggest constraint.

We now live in the age of the cloud. **The best indicator of software scalability is the capacity to mix and match the tools for storing data.**

At [EuroPython last year](https://www.youtube.com/watch?v=nLQoxqENbgg), I gave a talk on why tools like Django and Ruby on Rails are great for building simple projects, but terrible to scale. The problem is that they help engineers build a system fast, but put huge obstacles to changing the way to store data.

Why is that a problem? Because engineers are oblivious to the design decisions that are baked into these tools. They think “Instagram was started on Django, and Github on Ruby on Rails, so it’s good for my purposes”.

Yes, these companies were early on based on these tools. However, both Instagram and Github figured out ways to move beyond them. Their current designs look nothing like their humble beginnings. **When Meta engineers built Threads**, **they** [](https://engineering.fb.com/2023/09/07/culture/threads-inside-story-metas-newest-social-app/) **[piggy-backed on Instagram’s backend](https://engineering.fb.com/2023/09/07/culture/threads-inside-story-metas-newest-social-app/).**

They knew better.

Let me make the problem more concrete. In [Timelines at Scale](https://www.infoq.com/presentations/Twitter-Timeline-Scalability/), Raffi Krikorian, former VP of Platform Engineering at Twitter, showed how **Twitter’s scalability problems came**, not from an increasing amount of new tweets, but **from serving the home page to an increasing amount of people**.

A Data problem. Not a CPU problem.

Yet, if you look for videos on how to build a social media app online, **there is nobody warning you about this**. Youtube is plagued with videos showing you how to build a social media app that cannot, and will not scale.

Sure, those are pet projects, and the goal is to learn the tool, not to build a product. But if I’m trying to build, not to learn, the only resources I have access to **will prevent me from building scalable software systems**.

Startups always face fearful odds. But **if they’re given incorrect advice, they’re doomed**.

How do you design scalable payment applications? I tried to give a partial answer to that question on [What Makes Payments System Different?](https://news.alvaroduran.com/p/what-makes-payment-systems-different):

> Payment applications are one of those pieces of software that appear easy on the surface—who hasn’t paid with a credit card?—but that is deceiving. For that reason, when confronting such systems for the first time, software engineers [...] make false assumptions that end up causing a lot of harm, and losing a lot of money.
> 
> — [What Makes Payments Systems Different?](https://news.alvaroduran.com/p/what-makes-payment-systems-different)

Building software is an iterative process. But making a terrible architecture decision can put your startup years behind its competitors, diminishing your chances of user growth.

[Michelle Bu](https://www.linkedin.com/in/michellebu?miniProfileUrn=urn%3Ali%3Afs_miniProfile%3AACoAAAdYEJwBWvGF311EleaArWHmpw3AqPOkj_g&lipi=urn%3Ali%3Apage%3Ad_flagship3_search_srp_all%3BvXyTaednRdCVL64FuZziUA%3D%3D) echoed this sentiment. When Stripe was building their Payments API, she experienced the same problem. Namely, how to redesign away from a card-centered architecture.

> We built support for new payment methods on top of a set of abstractions that were designed for the simplest payment method of them all: cards. Naturally, abstractions designed for cards were not going to be great at representing these more complex payment flows.
> 
> Introducing additional states and expanding on the definition of resources that were created for a specific, narrow use case resulted in a confusing integration and an overloaded set of API abstractions. It’s as if we were trying to build a spaceship by adding parts to a car until it had the functionality of a spaceship: a difficult *and* likely doomed proposition.
> 
> — Michelle Bu, [Stripe’s payments APIs: The first 10 years](https://stripe.com/blog/payment-api-design)

In Youtube, [every](https://www.youtube.com/watch?v=C6rNeMnSi2o&list=PLd1Ex3znuJZvgxTPbgCy6c0CCa-Kub6XD&index=3&t=95s&pp=gAQBiAQB) [single](https://www.youtube.com/watch?v=olfaBgJrUBI&list=PLd1Ex3znuJZvgxTPbgCy6c0CCa-Kub6XD&index=4&t=663s&pp=gAQBiAQB) [video](https://www.youtube.com/watch?v=zsD4R_aQctw&list=PLd1Ex3znuJZvgxTPbgCy6c0CCa-Kub6XD&index=5&pp=gAQBiAQB) [I’ve](https://www.youtube.com/watch?v=NxjGFIgFCbg&list=PLd1Ex3znuJZvgxTPbgCy6c0CCa-Kub6XD&index=7&pp=gAQBiAQB) [found](https://www.youtube.com/watch?v=shipSEFMzHs&list=PLd1Ex3znuJZvgxTPbgCy6c0CCa-Kub6XD&index=8&t=224s&pp=gAQBiAQB) on how to design a payment system ends up supporting only credit cards. Even [System Design Interview Volume 2](https://www.amazon.com/System-Design-Interview-Insiders-Guide/dp/1736049119), which includes a chapter on designing a Payment System, says the following:

Again, I know that these are “for educational purposes only”. But who else is bridging the gap between ["we need to build our own payment application in-house"](https://news.alvaroduran.com/p/must-there-be-a-payment-platform) and actually showing engineers how to design payment applications for scale?

Well, I am. And if you want to learn how to do it, how to *really* do it, I suggest that you click subscribe.