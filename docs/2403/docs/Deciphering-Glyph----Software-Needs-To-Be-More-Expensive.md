<!--yml
category: 未分类
date: 2024-05-29 12:50:24
-->

# Deciphering Glyph :: Software Needs To Be More Expensive

> 来源：[https://blog.glyph.im/2024/03/software-needs-to-be-more-expensive.html](https://blog.glyph.im/2024/03/software-needs-to-be-more-expensive.html)

## The Cost of Coffee

One of the ideas that [James Hoffmann](https://www.youtube.com/@jameshoffmann) — probably the most influential… influencer in the coffee industry — works hard to popularize is that “coffee needs to be more expensive”.

The coffee industry is famously exploitative. Despite relatively thin margins for independent café owners^(, there are no shortage of horrific stories about [labor exploitation](https://www.theguardian.com/business/2020/mar/01/children-work-for-pittance-to-pick-coffee-beans-used-by-starbucks-and-nespresso) and even [slavery](https://www.dol.gov/agencies/ilab/addressing-child-labor-and-forced-labor-coffee-supply-chain-honduras) in the coffee supply chain.)

To summarize a point that Mr. Hoffman has made over a quite long series of videos and interviews^(, some of this can be fixed by regulatory efforts. Enforcement of supply chain policies both by manufacturers and governments can help spot and avoid this type of exploitation. Some of it can be fixed by discernment on the part of consumers. You can try to buy fair-trade coffee, avoid brands that you know have problematic supply-chain histories.)

Ultimately, though, even if there is perfect, universal, zero-cost enforcement of supply chain integrity… consumers still have to be willing to, you know, *pay more for the coffee*. It costs more to pay wages than to have slaves.

## The Price of Software

The problem with the coffee supply chain deserves your attention in its own right. I don’t mean to claim that the problems of open source maintainers are as severe as those of literal child slaves. But the principle is the same.

Every tech company uses huge amounts of open source software, which they get for free.

I do not want to argue that this is straightforwardly exploitation. There is a complex bargain here for the open source maintainers: if you create open source software, you can [get a job more easily](https://opensource.com/article/19/5/how-get-job-doing-open-source). If you create open source infrastructure, you can make choices about the architecture of your projects which are more long-term sustainable from a technology perspective, but would be harder to justify on a shorter-term commercial development schedule. You can collaborate with a wider group across the industry. You can build your personal brand.

But, in light of the recent [xz Utils / SSH backdoor scandal](https://boehs.org/node/everything-i-know-about-the-xz-backdoor), it is clear that while the bargain may not be entirely one-sided, it is *not* symmetrical, and significant bad consequences may result, both for the maintainers themselves and for society.

To fix this problem, open source software needs to get more expensive.

A *big* part of the appeal of open source is its implicit permission structure, which derives both from its zero up-front cost and its zero marginal cost.

The zero up-front cost means that you can just get it to try it out. In many companies, individual software developers do not have the authority to write a purchase order, or even a corporate credit card for small expenses.

If you are a software engineer and you need a new development tool or a new library that you want to purchase for work, it can be a maze of bureaucratic confusion in order to get that approved. It might be *possible*, but you are likely to get strange looks, and someone, probably your manager, is quite likely to say “isn’t there a free option for this?” At worst, it might just be impossible.

This makes sense. Dealing with purchase orders and reimbursement requests is annoying, and it only feels worth the overhead if you’re dealing with a large enough block of functionality that it is worth it for an entire team, or better yet an org, to adopt. This means that most of the purchasing is done by management types or “architects”, who are empowered to make decisions for larger groups.

When individual engineers need to solve a problem, they look at open source libraries and tools specifically *because* it’s quick and easy to incorporate them in a pull request, where a proprietary solution might be tedious and expensive.

That’s assuming that a proprietary solution to your problem even exists. In the infrastructure sector of the software economy, free options from your operating system provider (Apple, Microsoft, maybe Amazon if you’re in the cloud) and open source developers, small commercial options have been marginalized or outright destroyed by zero-cost options, for this reason.

If the zero up-front cost is a paperwork-reduction benefit, then the zero *marginal* cost is almost a requirement. One of the perennial complaints of open source maintainers is that companies take our stuff, build it into a product, and then make a zillion dollars and give us nothing. It seems fair that they’d give us some kind of royalty, right? Some tiny fraction of that windfall? But once you realize that individual developers don’t have the authority to put $50 on a corporate card to buy a tool, they *super* don’t have the authority to make a technical decision that *encumbers the intellectual property of their entire core product* to give some fraction of the company’s revenue away to a third party. Structurally, there’s no way that this will ever happen.

Despite these impediments, keeping those dependencies maintained does cost money.

## Some Solutions Already Exist

There are various official channels developing to help support the maintenance of critical infrastructure. If you work at a big company, you should probably have a corporate [Tidelift](https://tidelift.com) subscription. Maybe ask your employer about that.

But, [as they will readily admit](https://social.coop/@luis_in_brief/112185864683054685) there are a *LOT* of projects that even Tidelift cannot cover, with no official commercial support, and no practical way to offer it in the short term. Individual maintainers, like yours truly, trying to figure out how to maintain their projects, either by making a living from them or incorporating them into our jobs somehow. People with a Ko-Fi or a Patreon, or maybe just an Amazon wish-list to let you say “thanks” for occasional maintenance work.

Most importantly, there’s no path for them to *transition* to actually making a living from their maintenance work. For most maintainers, Tidelift pays a sub-hobbyist amount of money, and even setting it up (and GitHub Sponsors, etc) is a huge hassle. So even making the transition from “no income” to “a little bit of side-hustle income” may be prohibitively bureaucratic.

Let’s take myself as an example. If you’re a developer who is nominally profiting from [my infrastructure work](https://pypi.org/user/glyph/) in your own career, there is a very strong chance that you are *also* a contributor to the open source commons, and perhaps you’ve even contributed more to that commons than I have, contributed more to my own career success than I have to yours. I can ask you to pay me^(, but really *you* shouldn’t be paying me, your employer should.)

## What To Do Now: Make It Easy To Just Pay Money

So if we just need to give open source maintainers more money, and it’s really the employers who ought to be giving it, then what can we do?

Let’s not make it complicated. Employers should *just give maintainers money*. Let’s call it the “JGMM” benefit.

Specifically, every employer of software engineers should immediately institute the following benefits program: each software engineer should have a monthly discretionary budget of $50 to distribute to *whatever open source dependency developers they want*, in whatever way they see fit. Venmo, Patreon, PayPal, Kickstarter, GitHub Sponsors, whatever, it doesn’t matter. Put it on a corp card, put the project name on the line item, and forget about it. It’s only for open source maintenance, but it’s a small enough amount that you don’t need intense levels of approval-gating process. You can do it on the honor system.

This preserves zero up-front cost. To start using a dependency, you still just use it^(. It also preserves zero marginal cost: your developers choose which dependencies to support based on perceived need and popularity. It’s a fixed overhead which doesn’t scale with revenue or with profit, just headcount.)

Because the whole point here is to *match* the easy, implicit, no-process, no-controls way in which *dependencies can be added* in most companies. It should be easier to pay these small tips than it is to use the software in the first place.

This sub-1% overhead to your staffing costs will massively de-risk the open source projects you use. By leaving the discretion up to your engineers, you will end up supporting those projects which are really struggling and which your executives won’t even hear about until they end up on the news. Some of it will go to projects that you don’t use, things that your engineers find fascinating and want to use one day but don’t yet depend upon, but that’s fine too. Consider it an extremely cheap, efficient R&D expense.

A lot of the options for developers to support open source infrastructure are already tax-deductible, so if they contribute to something like one of [the PSF’s fiscal sponsorees](https://www.python.org/psf/fiscal-sponsorees/), it’s probably even more tax-advantaged than a regular business expense.

I also strongly suspect that if you’re one of the first employers to do this, you can get a round of really positive PR out of the tech press, and attract engineers, so, the race is on. I don’t really count as the “tech press” but nevertheless [drop me a line](mailto:oss-support-471610@glyph.im) to let me know if your company ends up doing this so I can shout you out.

## Acknowledgments

Thank you to [my patrons](/pages/patrons.html) who are supporting my writing on this blog. If you like what you’ve read here and you’d like to read more of it, or you’d like to support my [various open-source endeavors](https://github.com/glyph/), you can [support my work as a sponsor](/pages/patrons.html)! I am also [available for consulting work](mailto:consulting@glyph.im) if you think your organization could benefit from expertise on topics such as “How do I figure out which open source projects to give money to?”.