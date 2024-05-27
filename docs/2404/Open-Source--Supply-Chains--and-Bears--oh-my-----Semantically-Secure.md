<!--yml
category: 未分类
date: 2024-05-27 13:10:28
-->

# Open Source, Supply Chains, and Bears (oh my!) – Semantically Secure

> 来源：[https://scottarc.blog/2024/04/04/open-source-supply-chains-and-bears-oh-my/](https://scottarc.blog/2024/04/04/open-source-supply-chains-and-bears-oh-my/)

I didn’t want to add my voice to the cacophony of hot takes about [the xz backdoor incident](https://xeiaso.net/notes/2024/xz-vuln/) because I’m sure many people are already sick of hearing about it.

However, there is something related to it that I’ve been noodling over for a while. As a compromise, I won’t summarize or rehash the xz incident to spare anyone from having to read that for the thousandth time.

Most of the postmortem conversation I’ve witnessed has centered around the burnout of open source developers, corporations’ reliance and profiteering from volunteer labor, and how these factors can create a perfect storm for any nation state or sophisticated cyber-crime group that wants to backdoor infrastructure for their own gain.

## Why Should We Care About Your Opinion?

If you were asking yourself that question: You shouldn’t. I’m just some guy.

But if you’re instead curious, I’ve been developing open source software for over a decade. A [significant percentage of websites on the Internet](https://www.zdnet.com/article/wordpress-finally-gets-the-security-features-a-third-of-the-internet-deserves/) uses open source software I wrote.

Additionally, years ago, I spearheaded [an effort to add code-signing and transparency](https://paragonie.com/blog/2022/01/solving-open-source-supply-chain-security-for-php-ecosystem) to the PHP ecosystem’s package manager, Composer. Though unsuccessful, it provided a lot of insight into the challenges faced by the open source community. This will tie into a later point, but it’s not super important.

## What Other People Are Saying

Tim Bray wrote about [government-funded institutes to improve the quality of open source](https://www.tbray.org/ongoing/When/202x/2024/04/01/OSQI). Dan Lorenc tweeted about his experience trying to pay open source software developers.

Some full-time, unpaid open source developers are [excited to hear about how they’re going to finally get paid for their work](https://hax0rbana.social/@adam/112192930865266770) in the wake of this incident.

A lot of Twitter’s discourse is decidedly not as helpful:

## The Missed Point

There should be, by default, a clear separation between “open source software developer”, which can be a hobby rather than a profession, and “software library package provider for corporations” which is a paid service that some open source developers may choose to partake in.

Trying to get open source developers to do anything is worse than herding cats. You may be able to entice some with a treat, but the rest are going to do their own thing and there’s fuck all you can do about it.

If their software is the best available tool to solve the job, and they don’t want to deal with paperwork or legal crap to get paid for it, nobody can or should try to force them.

If they don’t want to bother with code signing or reproducible builds, don’t make it their problem. (Trust me, I tried to make it happen to PHP.)

Software packaging should be a separate vocation; another hat that some devs may choose to wear sometimes if it suits them.

Providing tarballs and binaries for an open source library is valuable labor. Your value proposition can simply be, “Patches for $library for $platforms; will upstream fixes at the original developer’s own pace,” predicated on the trust you’ve earned to your community. The rest of us can choose which package provider we wish to trust.

In addition to separating the responsibility into distinct roles (development, release management), it also adds a built-in layer of veracity that increases the probability of malicious updates being detected before they’re installed anywhere. This isn’t at all a guarantee, but it could help.

Open source developers that want to maintain control over how their code is used can simply wear both hats. Those that don’t want this responsibility can just focus on the part they enjoy and let packaging and distribution become someone else’s problem. Security vendors can hang out their shingle on, “Trust us to provide this library.” End users can decide whether or not they actually trust those vendors.

To me, this seems like a fairer way to ease some of the pressure off open source software developers whose pain points aren’t corporate funding. But for people who need money, I have another suggestion that’s unrelated (but compatible) with this one.

## Funding the Developers That Want Money

There have been three genres of “how we’ll fund open source development” floating around since I entered the scene.

1.  **Community-driven donations.** Patreon, Ko-Fi, OpenCollective, TideLift, etc. Individuals funnel money to projects they consider valuable, developers for said projects continue to work on them. Since most of us have very little disposable income, this tends to be like $5 per month of support.
2.  **Corporate sponsorships.** Sounds good on paper, but runs into a lot of challenges. Some OSS developers have existing employment agreements and aren’t interested in being a Business Insider hit piece on “overemployment”. Others live in another country, which sometimes makes paperwork challenging.
3.  **Government programs**. This is what Tim Bray argued for in his blog post (which I linked above and is worth a read). This puts programs at the mercy of the government (which, at least in America, is locked in a decades-long tug of war between one party that loves social programs that help people and another that loves tax cuts for the ultra-wealthy at the expense of the rest of us–in between the looming threat of government shutdown if their spending bills aren’t authorized by Congress).

Each of these three ideas faces tremendous challenge. I’d like to suggest a fourth idea, which can be supplemental to existing efforts, but is at least *novel* from what I’ve seen.

**Let’s create a new type of [Cyber Insurance](https://www.ftc.gov/business-guidance/small-businesses/cybersecurity/cyber-insurance).**

Wait wait wait, put away your pitchforks for a moment. I promise it will make sense.

In theory, insurance companies are supposed to work like this:

1.  Collect a small, recurring payment from individuals or companies over time.
2.  If an incident occurs, cover the cost of the incident.

In practice, many of us know that insurance companies do everything they can to avoid paying out (step 2), and the recurring payment costs (step 1) almost always increase. Ask anyone in the USA that’s been to a hospital for any reason in the past few years and you’ll hear all about it. The system is corrupt at every level. You’ll need to provide strong regulations on the new type of Cyber Insurance to prevent that. What this sort of regulation would look like exactly is an exercise for legal and policy experts to figure out; I do not pretend to be one.

That said, insurance companies that want to turn a profit can do so by hedging against risk. In the case of this hypothetical insurance, they could offset their risk by:

1.  Funding open source development, since so much of the Internet is powered by open source software.
2.  Funding open source packaging and distribution (previous section).
3.  Paying security experts to review open source software.
4.  Paying for security audits and penetration tests of proprietary systems for large companies that want lower coverage premiums.

This would provide another funding stream that is resistant to government shutdowns or budget cuts, and isn’t entirely tethered to a specific tech company’s self-interest.

Additionally, it will provide incentives that address one of the more annoying aspects of information security work:

 [https://www.youtube.com/embed/9IG3zqvUqJY?version=3&rel=1&showsearch=0&showinfo=1&iv_load_policy=1&fs=1&hl=en&autohide=2&wmode=transparent](https://www.youtube.com/embed/9IG3zqvUqJY?version=3&rel=1&showsearch=0&showinfo=1&iv_load_policy=1&fs=1&hl=en&autohide=2&wmode=transparent)

VIDEO

If you hire security experts to review your systems, and they find an issue, you can just accept the risk and not fix it. You’re the client. You usually own the work they produce, and can choose to do nothing with it!

In this model, the insurance company is the client. If you want to reduce your spend on insurance, you’ll adhere to their recommendations, or face a steeper premium.

## In Short

Separating the joy of developing OSS from the demands of mega corps, and using the insurance business model to fund developers who want to get paid for their work, is an additional idea to address open source supply chain risks.

Also: This blog post only represents my thoughts on the subject, and does not reflect the opinion of any employer (past, present, or future).