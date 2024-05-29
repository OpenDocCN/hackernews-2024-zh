<!--yml
category: 未分类
date: 2024-05-27 14:51:43
-->

# Brane Dump: Not all TLDs are Created Equal

> 来源：[https://www.hezmatt.org/~mpalmer/blog/2024/02/13/not-all-tlds-are-created-equal.html](https://www.hezmatt.org/~mpalmer/blog/2024/02/13/not-all-tlds-are-created-equal.html)

Posted: Tue, 13 February 2024 | [permalink](//www.hezmatt.org/~mpalmer/blog/2024/02/13/not-all-tlds-are-created-equal.html) | [No comments](//www.hezmatt.org/~mpalmer/blog/2024/02/13/not-all-tlds-are-created-equal.html#comments)

In light of the recent [cancellation of the `queer.af` domain registration by the Taliban](https://akko.erincandescent.net/notice/AenvYJ0yiHfspKM8uW), the fragile and difficult nature of country-code top-level domains (ccTLDs) has once again been comprehensively demonstrated. Since many people may not be aware of the risks, I thought I’d give a solid explainer of the whole situation, and explain why you should, in general, not have anything to do with domains which are registered under ccTLDs.

# Top-level What-Now?

A top-level domain (TLD) is the last part of a domain name (the collection of words, separated by periods, after the `https://` in your web browser’s location bar). It’s the “com” in `example.com`, or the “af” in `queer.af`.

There are two kinds of TLDs: country-code TLDs (ccTLDs) and generic TLDs (gTLDs). Despite all being TLDs, they’re very different beasts under the hood.

# What’s the Difference?

Generic TLDs are what most organisations and individuals register their domains under: old-school technobabble like “com”, “net”, or “org”, historical oddities like “gov”, and the new-fangled world of words like “tech”, “social”, and “bank”. These gTLDs are all regulated under a set of rules created and administered by ICANN (the “Internet Corporation for Assigned Names and Numbers”), which try to ensure that things aren’t a complete wild-west, limiting things like price hikes (well, sometimes, anyway), and providing means for disputes over names^.

Country-code TLDs, in contrast, are all two letters long^(, and are given out to countries to do with as they please. While ICANN kinda-sorta has something to do with ccTLDs (in the sense that it makes them exist on the Internet), it has no authority to control how a ccTLD is managed. If a country decides to raise prices by 100x, or cancel all registrations that were made on the 12th of the month, there’s nothing anyone can do about it.)

If that sounds bad, that’s because it is. Also, it’s not a theoretical problem – the Taliban deciding to asssert its bigotry over the little corner of the Internet namespace it has taken control of is far from the first time that ccTLDs have caused grief.

# Shifting Sands

The `queer.af` cancellation is interesting because, at the time the domain was reportedly registered, 2018, Afghanistan had what one might describe as, at least, a *different* political climate. Since then, of course, things have changed, and the new bosses have decided to get a bit more active.

Those running `queer.af` seem to have seen the writing on the wall, and were planning on moving to another, less fraught, domain, but hadn’t completed that move when the Taliban came knocking.

# The Curious Case of Brexit

When the United Kingdom decided to leave the European Union, it fell foul of the EU’s rules for the registration of domains under the “eu” ccTLD^(. To register (and maintain) a domain name ending in `.eu`, you have to be a resident of the EU. When the UK ceased to be part of the EU, residents of the UK were no longer EU residents.)

Cue much unhappiness, wailing, and gnashing of teeth when this was pointed out to Britons. Some decided to give up their domains, and move to other parts of the Internet, while others managed to hold onto them by various legal sleight-of-hand (like having an EU company maintain the registration on their behalf).

In any event, all very unpleasant for everyone involved.

# Geopolitics… on the Internet?!?

After Russia invaded Ukraine in February 2022, the Ukranian Vice Prime Minister [asked ICANN to suspend ccTLDs associated with Russia](https://eump.org/media/2022/Goran-Marby.pdf). While ICANN said that it wasn’t going to do that, because it wouldn’t do anything useful, some domain registrars (the companies you pay to register domain names) ceased to deal in Russian ccTLDs, and some websites restricted links to domains with Russian ccTLDs.

Whether or not you agree with the sort of activism implied by these actions, the fact remains that even the actions of a government that *aren’t* directly related to the Internet can have grave consequences for your domain name if it’s registered under a ccTLD. I don’t *think* any gTLD operator will be invading a neighbouring country any time soon.

# Money, Money, Money, Must Be Funny

When you register a domain name, you pay a registration fee to a registrar, who does administrative gubbins and causes you to be able to control the domain name in the DNS. However, you don’t “own” that domain name ^(– you’re only *renting* it. When the registration period comes to an end, you have to renew the domain name, or you’ll cease to be able to control it.)

Given that a domain name is typically your “brand” or “identity” online, the chances are you’d prefer to keep it over time, because moving to a new domain name is a *massive* pain, having to tell all your customers or users that now you’re somewhere else, plus having to accept the risk of someone registering the domain name you used to have and capturing your traffic… it’s all a gigantic hassle.

For gTLDs, ICANN has various rules around price increases and bait-and-switch pricing that tries to keep a lid on the worst excesses of registries. While there are any number of reasonable criticisms of the rules, and the Internet community has to stay on their toes to keep ICANN from [totally succumbing to regulatory capture](https://domainnamewire.com/2019/03/18/icann-proposes-lifting-price-controls-on-org-info-domains/), at least in the gTLD space there’s some degree of control over price gouging.

On the other hand, ccTLDs have no effective controls over their pricing. For example, in 2008 the Seychelles increased the price of `.sc` domain names from [US$25](https://web.archive.org/web/20071113061947/http://www.afilias-grs.info:80/public/policies/sc) to [US$75](https://web.archive.org/web/20080308114828/http://www.afilias-grs.info:80/public/policies/sc). No reason, no warning, just “pay up”.

# Who Is Even Getting That Money?

A closely related concern about ccTLDs is that some of the “cool” ones are assigned to countries that are… not great.

The poster child for this is almost certainly Libya, which has the ccTLD “ly”. While Libya was being run by a [terrorist-supporting extremist](https://en.wikipedia.org/wiki/Muammar_Gaddafi), companies thought it was a great idea to have domain names that ended in `.ly`. These domain registrations weren’t (and aren’t) cheap, and it’s hard to imagine that at least some of that money wasn’t going to benefit the Gaddafi regime.

Similarly, the British Indian Ocean Territory, which has the “io” ccTLD, was created in a [colonialist piece of chicanery that expelled thousands of native Chagossians from Diego Garcia](https://theconversation.com/how-the-us-and-uk-worked-together-to-recolonise-the-chagos-islands-and-evict-chagossians-177636). Money from the registration of `.io` domains doesn’t go to the (former) residents of the Chagos islands, instead [it gets paid to the UK government](https://web.archive.org/web/20200314085443/gigaom.com/2014/06/30/the-dark-side-of-io-how-the-u-k-is-making-web-domain-profits-from-a-shady-cold-war-land-deal/).

Again, I’m not trying to suggest that all gTLD operators are wonderful people, but it’s not particularly likely that the direct beneficiaries of the operation of a gTLD stole an island chain and evicted the residents.

# Are ccTLDs Ever Useful?

The answer to that question is an unqualified “maybe”. I certainly don’t think it’s a good idea to register a domain under a ccTLD for “vanity” purposes: because it makes a word, is the same as a file extension you like, or because it looks cool.

Those ccTLDs that clearly represent and are associated with a particular country are more likely to be OK, because there is less impetus for the registry to try a naked cash grab. Unfortunately, ccTLD registries have a disconcerting habit of changing their minds on whether they serve their geographic locality, such as when auDA decided to declare an open season in the `.au` namespace some years ago. Essentially, while a ccTLD may have geographic connotations *now*, there’s not a lot of guarantee that they won’t fall victim to scope creep in the future.

Finally, it *might* be somewhat safer to register under a ccTLD if you live in the location involved. At least then you might have a better idea of whether your domain is likely to get pulled out from underneath you. Unfortunately, as the `.eu` example shows, living somewhere today is no guarantee you’ll still be living there tomorrow, even if you don’t move house.

In short, I’d suggest sticking to gTLDs. They’re at least *lower* risk than ccTLDs.

# “+1, Helpful”

If you’ve found this post informative, why not [buy me a refreshing beverage](https://ko-fi.com/tobermorytech)? My typing fingers (both of them) thank you in advance for your generosity.

* * *

* * *