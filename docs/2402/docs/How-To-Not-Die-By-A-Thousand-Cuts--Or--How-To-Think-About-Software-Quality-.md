<!--yml
category: 未分类
date: 2024-05-29 13:20:49
-->

# How To Not Die By A Thousand Cuts. Or, How To Think About Software Quality.

> 来源：[https://www.evalapply.org/posts/how-to-not-die-by-a-thousand-cuts/index.html](https://www.evalapply.org/posts/how-to-not-die-by-a-thousand-cuts/index.html)

* * *

First off, what even is Quality?

All things emerge, change, and die. I think *Quality* is the experience of the process. The idea of *Good Quality* essentially boils down to performing the process with grace, and leaving the place better than we found it.

Further, the process of emergence and change—i.e. living—is also the process of dying. It follows that to think clearly about the Quality of the former one must think clearly about the Quality of the latter. The saddest way it can unfold is a slow painful degradation without healing succour meaning or hope. The proverbial death by a thousand cuts. I hope you never witness such a passing, even from afar.

Ok, that got dark fast, and if we're not careful, we will produce a 300 page Zen dialogue on Motorcycle Maintenance. So we will distract ourselves with the much smaller, lighter—and I'd argue, even pleasant—task of contemplating Quality of software products.

None of what follows is novel, but I feel the message *and* its surrounding context bears repeating, because if it is not obvious already, software fails us all the time. Far too often with terrible consequences.

# What is the nature of software products?

See? This is already easier than asking "What is the nature of life?".

Like any other machine, a software product is wrought of the labour of many minds and hands, and it requires maintenance and upkeep throughout its life.

Unlike *all* other machines, it is pure concept, and as such it is infinitely malleable and mutable. And mutate it does, all the time.

Sometimes, "finished" software emerges, only needing minor fixes and patches, but remaining the same in purpose, interface, and behaviour. Many Unix tools fall in this category. Some projects like ZeroMQ make it their explicit goal. Many Clojure programmers value such "finished-ness". Such scattered examples exist.

Most software does not have this luxury. Most software must change indefinitely because the world it must serve changes indefinitely. The Emacs editor is a software product that has evolved non-stop for *nearly half a century* since it emerged in 1976, and it continues to thrive. This post was written in Emacs.

There is a strong reinforcing feedback loop too. Software changes the world fast, forcing software to change faster. The current reincarnation of Machine Learning and AI can be viewed as an expression of this process. We're basically saying it's all accelerating so much that it is getting humanly *impossible* to write and revise software fast enough, to out-OODA the pace of change. So we must instead find algorithms that sense the world and then dynamically generate or revise other algorithms to achieve system objectives (viz. alter the world further in our favour).

We have to wonder, how do we make sure our product continues to thrive and succeed under such unrelenting pressure of constant and sometimes violent change? And who's neck should be on the line for it?

# Whom to hold responsible for Software Quality Assurance?

The Usual Suspects?

*   Those "Quality Assurance" boffins? Developers? UX people? DevOps?

The Less Usual Suspects?

Consider the scenarios below. All of them directly impact customers, making them think "bad quality". Consider who is responsible for the underlying problem (or more likely, problems)?

*   Your app framework is extremely performant and glitch-free. Your app bombs.
*   A feature does exactly what it promises, but people fail to use it right.
*   Your company committed half of itself to ship a second product in record time, but customers never really wanted it.
*   A huge update was pushed out on a do-or-die basis. Naturally it misbehaves, can't be rolled back, costs 5x as much to get right as it took to ship, and the rework effectively adds months to your plan of world domination.
*   Your service fails to scale. You discover there were no benchmarks.
*   A deployment breaks production. You discover a bad configuration.
*   A feature leaks data to unintended users and breaks SLAs / regulations. Your CEO releases a statement blaming a DevOps engineer.
*   A several-hour glitch goes un-monitored, causing serious widespread data corruption.
*   Your production noticeably degrades often. A large sea mammal is your mascot.
*   Your production seldom degrades, but when it does, it takes down half the known Internet along with it.
*   and on and on…

In a quiet moment of honest self-reflection, you may confess to the mirror that the thousand cuts metaphor applies. That any of the above scenarios were likely the product of corner-cuts, often near-invisible to the naked eye in the moment. Corner-cuts that added up—nay, *compounded*—over time; slowly as band-aids, then as stitches and casts, and then suddenly as gangrene. And maybe the whole thing died of those cuts, or continued as a barely alive entity until someone had the heart to pull the plug (or offer a bail out).

You may even confess that maybe, just *maybe*, the job of assuring the goodness of a product belongs to *every function involved in the product's life*.

# Why?

Suppose we model a traditional software production workflow, i.e. Analysis -> Product requirements -> UX/Design -> Development -> "QA" -> Production.

Such a strictly linear model is common in the software industry at large. This is what it translates to in terms of time, complexity, costs, and risks.

```
 ^   Feedback
 Analysis -> Product -> UX/Design -> Dev -> "QA" ->  Prod --./--> arrives
                                                           /      too late
                                                         /-
                                                       /-
                                                     /- ^
                                                  /--   | Price of fixing
                                               /--      | errors and
                                           /---   ^     | corner cuts.
                                       /---       |     |
                                  /----   ^       |     | ~ AND/OR ~
                             /----        |       |     | Compounding of
                     /------   ^          |       |     | software debt.
            /--------          |          |       |     |
  ----------      ^            |          |       |     | ~ AND/OR ~
   ^              |            |          |       |     | Increasing odds
   |              |            |          |       |     | of being wrong.
---+--------------+------------+----------+-------+-----+---------------->
                            Time, Complexity, Sunk costs 
```

Visualising a linear workflow this way suggests some things:

*   All the risk is actually front-loaded at the Analysis stage. If that is wrong, then everything is wrong.
*   The workflow looks linear, but has a compounding growth debt/risk profile.
*   By tasking a single group with "assuring" product quality, we maximize our odds of being too wrong too late, as well as of entirely failing to spot bad news.

What's not obvious from the picture is that the risk is rooted in *feedback delays*. Weak signals die when the deliver pressure is high.

Our death-by-cuts risk profile will look the same, if the workflow is strictly linear as depicted above. It doesn't matter if we do it slowly in big batches over months, or faster as smaller batches over days. Small linearised batches may even worsen the aggregate risk profile, such as when market feedback loops are delayed or discontinuous. The smaller the batch, the more likely it is that feedback about several batches ago gets to us now. Such delayed feedback tends to severely disrupt strictly linear flows.

The above picture is also incomplete. For the full story, we need to talk deeply about systems (a longer conversation, for another day). We can make a small start by doing scenarios. Consider points on a product spectrum, ways to destroy/create quality, and what might help us go from worse to better?

# Is it different for different kinds of products?

Suppose we contrast two typical ends of the product spectrum defined by primary customer. Which one risks death by a thousand cuts?

| Key growth metric | Revenue Growth | User Growth |
| Key sales driver | Referrals + executive credibility | Referrals + Friends-and-family experiences |
| Customer risk | High risk/reward per account | Tiny unit economics per account |
| Contract risk | SLAs with crippling penalties | 1 EULA / ToS that users don't read |
| etc … | … | … |

Well, here's the thing. Not only does all software mutate, we *also* end up performing all kinds of deep surgery on the *organisation* that produces it. The whole thing—product and org—is *simultaneously* flexed, reconfigured, and even totally redesigned in-place with rapidity that is very uncommon in other industries. Why? Because software fundamentally is peoples' thoughts being played on repeat.

So however we break it down, the common theme is this. Every hotfix is a cut. Every complaint is a cut. Every app crash is a cut. Every service outage is a cut. And so on. Each cut heals slowly and destroys Quality and value(ation).

# How to destroy Quality?

It's useful to come up with ways to destroy quality, so that we may contrast those with ways to generate quality. I've seen and heard all of the following in work life so far (hopefully without actively perpetrating them, but memory is a fickle beast).

*   Misconstrue and mislabel Software Testing as Quality Assurance. Testing is *not* "Quality Assurance".
*   Ostensibly make all teams responsible for their "QA", which really means make the least experienced people do it day-to-day.
*   Create a culture where it's normal to say things like this):
    *   "Hey I'm adding this to the sprint. It's a small thing, so let's not slip our deadline."
    *   "Testing is boring."
    *   "We'll fix it if customers complain."
    *   "Who the f*#$ wrote this code?"
    *   "Ah yes, those are known flaky tests. Just re-trigger the build."
    *   "You don't know your job. Ship this." (This one stung. I'll tell you over beer/coffee :).
*   Ensure designers, developers, and testers work on tasks and priorities set by others.
*   Ensure someone catches the blame for mistakes.
*   Set up incentives to make departments compete with each other.
*   Hire a Vogon or a Darth Vader CEO.
*   Further [normalise all kinds of deviance](https://danluu.com/wat/).

This was just a shortlist of things I recalled while writing this post. Think up as many ways as you can. #protip for inspiration: read CIA's now-declassified [Simple Sabotage Field Manual](https://www.gutenberg.org/files/26184/page-images/26184-images.pdf). Pay special attention to part 11: *General Interference with Organisations and Production*.

# How to create Quality?

One clue is to *not* do quality-destroying things. Another is to do the *inverse* of quality-destroying things (e.g. share know-how instead of hoarding it.) A third is to notice whether high-quality product producing organisations have any common traits (they do). Most important, perhaps, is to understand that there is no formula for how to acquire those traits.

To design and build high quality software products, it is imperative to design and build high quality organisation-wide systems and culture. We have many tools, frameworks, fundamental ideas at our disposal. But no "best practices" process or methodology or "one weird trick" style intervention can fix broken systems and broken people.

The "way" has to be co-evolved:

*   by collaborative stakeholders,
*   spread across the org,
*   appropriate to the org's unique context,
*   along with customers, partners, and the immediate ecosystem.

This is universally a very difficult process, with challenges surprisingly similar to what it takes to recover fitness after a year of slacking off. It requires mindset, leadership, and persistent holistic intelligent *eval/apply* behaviour. And all of that derives from *perspective*.

> "*Perspective is worth 80 IQ points.*"
> 
> — Alan Kay

So, if we are to chart a course from Worse Quality to Better Quality, then it must be our first duty to purposely get really uncomfortable by seeking out new-to-us, diverse, status-quo-challenging perspective. And …

# The first skill is to learn to suffer constructively.

We suffer, you and I.

It is inevitable. Yet, it is also why life flourishes. *"Why are we suffering?"* is a great discussion to have, because constructive suffering yields quality outcomes.

OK, back to the real world…

The path to recovering a *previous* fitness peak after a year of slacking off is filled with sore muscles, cursing at the alarm clock, far too many days of being a generally irritable snappy person, and a constant mental battle against mainlining deliciously easy instant gratification. It gets harder before it gets easier. Then we reach the top of the previous S-curve. And we must begin the cycle again, to climb the next one.

We are very fortunate.

Fellow sufferers have been fostering quality-generative conversation and change all around us. We have access to a growing body of top-notch industry research *and* experience reports. Without exaggerating, very many of these lessons have been paid for in tears, blood, lives. Let's augment our intuitions with these power tools. Those hard-won *80 extra IQ points* are ours for the taking.

Some selected resources.

Many inputs have shaped my thinking about Quality (well, all the things, because everything is connected); people, events, books, lectures etc. If you're wondering where to go. These are not prescriptions, but a sort of sampling platter. Triggers for your own searches. Please send me more!

Systems:

Software complexity:

Failure:

Doing Together:

Oneself (heavily biased, because I identify as a software programmer):

"Practical philosophy", for lack of better words:

I recently discovered Gene Kim's podcast, [The Idealcast](https://itrevolution.com/the-idealcast-podcast/). Gene is gathering fantastic people and resources in one place. Definitely have a look-see.

# Caveats, mea culpa, etc.

I am very much a work-in-progress, and this post is my current intuition.

The post is heavily coloured by many witting and uwitting eval/apply loops comprised of personal failures, ignorant mistakes, and occasional wins, over the last about 20 years of professional life. And well, life life. It is also informed by the good fortune of having learned by working with people who understand the world far better than I do. And obviously a lot of reading, thinking, talking, frequently "in anger" after having hit walls and obstacles.

So please take what is useful, and discard the rest.

May the source be with you _\\//