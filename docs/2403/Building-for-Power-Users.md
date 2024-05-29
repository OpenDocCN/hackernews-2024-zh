<!--yml
category: 未分类
date: 2024-05-27 14:42:59
-->

# Building for Power Users

> 来源：[https://www.thediff.co/archive/building-for-power-users](https://www.thediff.co/archive/building-for-power-users)

In this issue:

*   Building for Power Users—Some apps are used very intermittently for a one-off purpose, like ordering food or checking on the status of a financial transfer. Some are used all the time, but as a cure for boredom and a substitute for not knowing what to do. And then there are apps for power users, where the typical user is constantly engaged with the product and uses it not just as a tool for working but as a tool of thought. These are an important category, where many of the rules of usability have to be tweaked or even reversed.
*   Adtech and "Sky Bridges"—We're rebuilding the old open ad ecosystem, but only for the biggest participants.
*   SuperMicro!—40x in five years is a solid return, but the stock's recent performance has some not-quite-fundamental drivers.
*   GPD YoY—A periodic reminder that annualized growth rates can be misleading.
*   Black Boxes—Less advertising transparency means that the economics of software bugs have changed.
*   Pods and Cycles—Hedge funds can dump a money-losing position instantly, but it's harder for them to be as agile with personnel.

## Building for Power Users

My iPhone tells me I have 102 apps installed right now, which sounds about right. A bash script for finding all of the executable files on my Mac indicates that, excluding files that are restricted to the root user, there are no fewer than 200,327 executable files. Most of them I use incidentally (like `find` and `wc` in the script I used to count the executables), but from time to time, there's an application I basically live in: Emacs, [Superhuman](https://www.thediff.co/archive/the-superhuman-formula/) ($, *Diff*), [Excel](https://www.thediff.co/archive/the-programmingexcel-efficient-frontier/) ($, *Diff*), Notion, [Bloomberg](https://www.thediff.co/archive/the-bloomberg-terminal-shows-how/), etc.

These apps belong in a special category. They're built for power users.

Apps for power users have a few things in common:

*   They're how users make their living or do their thinking.
*   They're often in use most of the day. Users with multiple monitors will typcially devote one or more to one of these applications.
*   You can use them for years and continue to learn about new capabilities.
*   They attract partisans: the mere mention of Emacs has probably led someone to fire up Vi and compose a missive about why that's a terrible decision at 140 wpm.
*   An update that moves a key feature around, changes how hotkeys work, or breaks a plugin can absolutely ruin a user's day, and even prompt them to switch their loyalty to an alternative.
*   They're often extensible within the app and to other apps. Excel has plugins; some of those plugins take in data feeds from other providers, and some Excel spreadsheets feed outputs elsewhere, too.
*   They have a painful learning curve; there is a harsh tradeoff between power and generality on one end and discoverability on the other. And many of these products are quite old, and can't revise their interface without ruining users' muscle memory, so the annoyances persist forever.

A fun feature of these products is that they exist at every scale. The median power app user is probably someone who spends a lot of their workday in Excel. But the median app for power users is probably some tiny tool built for in-house use by a single company, which does its one job and does it well. (In researching this piece, we talked to people who have built products through which billions of dollars flow, and other products where the total number of people who have any use for them at all is in the dozens to hundreds.)

Building a tool like this means thinking very seriously about what users are trying to accomplish, and modeling the behavior of the most sophisticated user in a detailed way. This is in contrast to many other applications where the default goal is discoverability. Spotify is not designed for someone who wants to DJ a one-of-a-kind event with an esoteric theme, do a comparative analysis of how various conductors handle a particularly tricky passage by Mahler, or track the popularity over time of a particular hip-hop sample. No, it's a simpler job-to-be-done: I'm about to head to work / go work out / go crazy because my kids are demanding a novelty song that may not actually exist, so give me some good background listening. For more transactional apps, the task is simpler, but the meta-task is harder. If someone is logging into their HR portal and opening one of the half-dozen customer loyalty apps they have installed on their phone, they might have any of half a dozen tasks in mind, but they also haven't looked at the app in question in weeks or months, so the app needs to be re-learned. Apps like this can put a very low premium on keeping their experience consistent, simply because no one will notice if it changes. Instead, they have to put a high premium on discoverability, and can default to pretending that every user is opening the app for the first time. They don't and can't really care about extensibility, composability, and the like—few use cases involve intermittently doing something complicated within an app that isn't regularly used.

The downside to maximizing power is that it necessarily trades off discoverability. The faster it is for an experienced user to do anything—whether it's a sysadmin cross-referencing memory consumption and CPU usage over time check in a single quick shell script, or a Bloomberg user whipping up a comparison between implied volatility in copper and realized volatility in the Chilean peso in a handful of keystrokes—building a product that enables it means getting out of the user's way, and that means assuming they know what they're doing or can quickly figure it out. If the median user-hour for a product is from someone who has used the product for thousands of hours before, a "tips" feature highlighting things they’ve used for years is just going to annoy them.

It's ideal if the app assumes that users want to use any feature in any format. A given app might exist as a standalone desktop application, a browser-based tool, and a mobile app. This is a strong but not definite requirement for the apps-for-power-users category, since some tools just don't make sense without an external keyboard (using Emacs or Vi with a typo-prone input mechanism sounds maddening, for example). But if an app exists in multiple formats, it's essential that:

1.  The state is synced across devices at all times, so the app can function as a single source of truth, and
2.  There isn't anything you absolutely need to do in the desktop version.

Point 2 needs unpacking, but it's tied to the idea that apps for power users are apps people work in and think in. If you're taking an Uber ride somewhere, you don't leave 70% of your brain behind at the office; the apps you do your work in shouldn't require this of you. Syncing, and syncing intelligently, also becomes important with multi-platform apps. Mobile apps are a disproportionately common place to start things that will be finished elsewhere or vice-versa: write down some quick bullet points that you'll expand into a real email later, add a stock to a watchlist so you remember to do deeper research once you're back to your desk, update an existing calculation with a new variable and grab the output, etc. If there's not precise feature compatibility across products, you spend some of your mental overhead thinking about which instance of the product you're using rather than spending all of it on the job at hand.

This brings up an important distinction between "task time" and "tool time." Task time is when you build a complicated Excel model; tool time is what happens in between when you hit F9 to recalculate everything and when you finally get your result. One way to understand apps for power users is that they're targeting the minimum tool time asymptote: they want to be maximally useful for someone who knows exactly what they're doing. Tool time can be useful in the early stages of learning how to use something; every popup highlighting a new feature is telling you something you didn't necessarily know, and if you don't understand the product well, the marginal impact of a tool time tax is low. But at some point, tool time becomes a barrier to getting things done because the user can think faster than the application can react.

All these characteristics make such apps interesting products, and give them some unique economics. For one thing, they tend to be valuable complements to the work of people who are already quite economically valuable. For a while in the 1980s, Emacs was a proprietary software product [priced from $395 to $2,500 per license](https://en.wikipedia.org/wiki/Gosling_Emacs?ref=thediff.co), or, at the low end, [about 1% of what programmers in the Bay Area made in a typical year](https://www.bls.gov/opub/mlr/1984/09/rpt1full.pdf?ref=thediff.co). Microsoft captures a tiny fraction of the total value created in Excel (and, thankfully, is not on the hook for all Excel-enabled value destruction).

These applications tend to have incredible pricing power. The more of someone's cognition that gets offloaded to the app, the more switching means scooping out a portion of their brain and waiting a few months or years for it to grow back. Some companies push this, like Bloomberg. For others, like Microsoft, there are limits: what Microsoft used to worry about was Google disrupting them from the low end through G Suite, but now the concern is more Google disrupting them sideways, by making Sheets strictly better for some applications (it's a weirdly popular quasi-backend, for example), and by incorporating more search-related features, adding nice extras like [QR code generation](https://www.lovesdata.com/blog/google-sheets-tips?ref=thediff.co#google-sheets-tip-17), and being a default backend for Google Forms. Meanwhile, at the high end, Microsoft has to worry that some calculations get big enough and mission-critical enough that they graduate to being done in Python instead.

The other form of disruption these apps face is that they're often a general-purpose implementation of a special-purpose task. "We just keep track of it in Excel" is often the germ of a product idea that can target 0.1% of Excel's market but charge 100x as much per seat for something that re-implements whatever tiny fraction of Excel's features are needed in a specific domain, and extends it to handle whatever the core product can't do.

So even though at a user level the products are incredibly sticky, at an organizational one they're always threatened. People form habits that are hard to dislodge, and those habits start with easy entry points. It's easier to complicate a simple product than to simplify a complex one, so new power user apps can emerge from domain-specific ones.

In fact, this is the kind of area where AI will have an impact. Using LLMs to code speeds up software development cycle times, but only for products where the creator has a good idea of what they're ultimately trying to build. The reason we use so many general-purpose apps is that many purposes are too small-scale to support the full-time effort of a team of software engineers. Many more of them are big enough for a nights-and-weekends project that evolves into a full-time business.

And, at least in this category, that's something that has to happen: an app can only work for power users if whoever creates it pays close attention to how it's being used, with a special focus on when users switch to something else. If you watch someone use such an application that you discovered, every alt-tab should be physically painful—it represents something your users regularly need to do that your product doesn't help them with. It's hard to sell to obsessives without being obsessive yourself, and the incredibly long customer lifetime for these products means that any unnecessary churn is expensive. But that's just a cost of doing this particular kind of business.

Software problems tend to evolve over time into philosophy problems, a point that's come up in a few *Diff* pieces, like [the Antithesis writeup](https://www.thediff.co/archive/antithesis-debugging-debugging/) and [Asana's ontology-as-a-service](https://www.thediff.co/archive/asana-ontology-as-a-service/) ($). In this particular category, it's the philosophy of work and vocation: a bond trader, accountant, or social media manager doesn't necessarily have to spend much time navel-gazing about the nature of their job. But if you're building a product that makes that specific job 5% more productive, you'll naturally become a part-time anthropologist with elaborate views on the nature of work, the maintenance of flow states, and the frustration of having not-quite-the-right-tool-for-the-job.

* * *

Disclosure: Long MSFT.

Thanks to Daniel Doyon of [Readwise](https://readwise.io/?ref=thediff.co) and several other people who spoke off-the-record about building these kinds of products.

## **Diff Jobs**

*   A CRM-ingesting startup is on-boarding customers to its LLM-powered sales software, and is in need of a backend engineer to optimize internal processes and interactions with customers.
*   A company building the new pension of the 21st century and building universal basic capital is looking for a product designer with fintech experience. (NYC)
*   An investment company using AI to accelerate investment in esoteric asset classes is looking for a product engineer with Python and Typescript experience; preferably someone with a track record of building on their own (Bay Area, remote also a possibility).
*   A crypto proprietary trading firm is actively seeking systematic-oriented traders with crypto experience—ideally someone with experience across a variety of exchanges and tokens. (Remote)
*   A data consultancy is looking for data scientists with prior experience at hedge funds, research firms, or banks. Alt data experience is preferred. (NYC)

Even if you don't see an exact match for your skills and interests right now, we're happy to talk early so we can let you know if a good opportunity comes up.

If you’re at a company that's looking for talent, we should talk! Diff Jobs works with companies across fintech, hard tech, consumer software, enterprise software, and other areas—any company where finding unusually effective people is a top priority.

## Elsewhere

### Adtech and "Sky Bridges"

Now that it's harder for companies to programmatically share detailed customer data across platforms, ads are changing. Specifically: [big advertisers and big customers are striking deals to pool their data, meaning that the old ad model is still mostly available, but only to the biggest winners](https://live-adexchanger.pantheonsite.io/commerce-roundup/retail-media-has-created-sky-bridges-between-content-fortresses/?ref=thediff.co). Not only does this mean that a given piece of ad inventory monetizes better when it's owned by a bigger company, but even within these deals there are scale advantages: the biggest platforms can strike more lopsided deals, and tend to do deals that increase their own data advantage even further, by gaining access to a partner's information without sharing anything in return. The net result of this is consolidation in both adtech and retail, but given who's doing the consolidating, it's less likely to take the form of M&A than of attrition from sub-scale participants.

### SuperMicro!

There are stocks that go up 40x in half a decade, and there are large-cap companies, but the combination is uncommon. SuperMicro, though, traded at $20 in early 2019 and closed Friday at $803 (and that was after a 20% drop). The *Financial Times* has [an overview of the business](https://www.ft.com/content/5b3ff851-acd9-4588-9ed9-edef8164fbcc?ref=thediff.co) ($): they make servers, particularly high-performance servers for AI computing. This business has been growing quite fast (though they don't serve the really big AI deployments by the hyperscalers; SuperMicro sells to smaller customers, and its gross margins are those of a semi-commoditized business, not one with enduring pricing power. There is, of course, plenty of money to be made owning commoditized businesses at times of extreme supply/demand imbalance. But there's another supply/demand imbalance to think about: [options referencing 36% of their shares outstanding expired on Friday, and if the usual option-buying pattern of retail investors holds, many of these options moved into the money, forcing options dealers to buy more shares to hedge](https://www.fabricatedknowledge.com/p/wtf-is-going-on-at-supermicro-smci?ref=thediff.co). This is the kind of thing that can lead to wild volatility, but it requires a continuous flow of new retail money just to maintain a share price driven by options demand, not to mention increase it.

Disclosure: Short a very small amount of SMCI, as part of a larger basket of overhyped-AI bets (these bets are partly offsetting long bets on big tech companies that should benefit from AI, but in the short term it's not a hedge at all; the weaker or more retail investor-oriented AI bets tend to go up when more mainstream AI bets go down, because the levered investors who are shorting them also tend to have levered long exposure to big tech). SMCI is actually a much better business than most of them, but I've seen more cases where a low-margin company temporarily under-earns than cases where such a company's economics permanently reset.

### GDP YoY

Long ago, in July of 2020, *The Diff* came out with a stern warning: [do not treat "annual" GDP numbers as if they are annual numbers, because some countries report them by taking a quarter-over-quarter change and annualizing it](https://www.thediff.co/archive/spacs-as-a-call-option-on-hype/#q-q-4-y-y). This is relevant once again, as [Israel's economy shrank 9% from the third to fourth quarter last year, which is an annualized pace of 20%](https://www.ft.com/content/763bb384-a974-4222-996f-8aecfbc32074?ref=thediff.co) ($, *FT*). But that annualizing only makes sense if the number is a clean long-term trend: any economy where a substantial chunk of the labor force switches to the armed forces will experience some economic disruption, but that part, at least, is a one-time issue. Annoyingly for GDP discourse, the number tends to get discussed when it's extreme, but extreme numbers are more likely to be caused by one-time shifts that shouldn't be extrapolated over an entire year.

### Black Boxes

*The Diff* has written a few time about the evolution of ad businesses towards large platforms that use proprietary data and models to target, giving less flexibility to advertisers. One thing that makes this process less straightforward is that when the model is wrong, it's hard for advertisers to have a clear idea of how wrong it was and what it cost them. [Both Meta and Google have recently had bugs in their ad targeting or view tracking systems, leading them to give refunds](https://www.adexchanger.com/daily-news-roundup/how-facebook-profits-from-overspending-errors-and-speaking-of-hi-google/?ref=thediff.co). When there are many dials to turn for ads—which keywords to target, which demographics to aim for, what time of day to raise or lower bids to stay competitive, etc., the advertiser has a fair amount of confidence in the connection between their spending and revenue. When there's just one dial, allowing them to spend more money to get more customers but leaving all of the details up to the ad platform, the difference between a bug and an increase in the platform's take rate is ambiguous, and has to happen on the honor system.

Disclosure: Long META.

### Pods and Cycles

One reason [pod shops](https://www.thediff.co/archive/pod-people/) ($, *Diff*) got so much attention in the last year or two is that they were writing such big checks to new portfolio managers. The hedge fund industry moves fast, but there are some places where its structure slows things down, one of which is that many employees have long noncompetes: a big offer is partly a way to pay someone to quit their job, wait a year, and only *then* start somewhere else. But that also means that the funds' cost structure today is determined by their hiring decisions a year ago, and since the interview process is so extensive, the decision to make the hire is even earlier than that. So if opportunities for outperformance fluctuate, funds will chronically find themselves either understaffed relative to opportunities or overpaying for their current performance. We now seem to be [in the latter half of that cycle](https://www.efinancialcareers.com/news/hedge-fund-jobs-2024?ref=thediff.co).