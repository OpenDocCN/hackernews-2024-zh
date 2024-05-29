<!--yml
category: 未分类
date: 2024-05-27 14:55:51
-->

# There is no EU cookie banner law - Bite code!

> 来源：[https://www.bitecode.dev/p/there-is-no-eu-cookie-banner-law](https://www.bitecode.dev/p/there-is-no-eu-cookie-banner-law)

*You know those modal screens that interrupt your groove when you are surfing?*

*There are no laws forcing websites to use them.*

*They use them because they choose to.*

I've had multiple discussions online with Americans feeling angry that EU forced them to click through a wall every time they go to a new website.

To avoid redundancy, I'll just write once about it here, even it's not usually the topic of this Python-oriented blog.

Even if they were such a thing as a cookie banner law, and there is none, companies in the USA would not have to comply in their country.

It would be only for Europe.

It goes without saying, yet the law says so, just in case:

> **Member States** shall ensure that the storing of information, or the gaining of access to information already stored, in the terminal equipment of a subscriber or user...

The USA are not a member state of the EU, by definition.

So companies could show their annoying banners only to the EU residents.

In fact, some do.

*amazon.**com*** doesn't show any cookie banners, *amazon.**fr*** shows one.

You absolutely don't have to suffer through this, it is a decision made by the companies to inflict it on you.

And you know what's worse? It's unnecessary. **A cookie banner is not required at all, even in the EU.**

Let's follow up on the text:

> ...[gaining of access to information] is only allowed on condition that the subscriber or user concerned **has given his or her consent**, having been provided with clear and comprehensive information..."

So basically, we tell companies the same thing we tell young boys: if it's getting private, you need consent.

You'll notice the words "cookie" or "banner" appear nowhere in this paragraph. That's because they are not the focus of the law.

The only thing that matters is that if an entity wants to track people, they have to let them know in a way that is clear and request their approval.

I find that personally very sane.

What is not sane are thousands of websites stealing my data away without me knowing it, and having to work very hard for it not to happen.

I think anger towards the EU is misdirected, and should be instead refocused on the entities that decided that:

Because **even if you decide on tracking, cookies banners are still not required!**

It's the worse way possible to meet the legal requirements for the users.

There has been for years a proposal for a standard, designed in 2009 (!), still available in all the popular web browsers (except safari) that can make for a seamless experience: [the DNT header](https://en.wikipedia.org/wiki/Do_Not_Track).

The user ticks a box in the browser settings, and all websites receive either:

Then the server can easily check that. In django it's basically one line:

```
`reject_tracking = request.META.get('HTTP_DNT') == '1'`
```

You can just serve your tracking depending of that, no nagging necessary.

Almost no website have implemented it, because companies WANT to nag you, hopping to trick you into being tracked. They know nobody would click yes on those settings.

So now [it's deprecated](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/DNT).

Companies are making your life hard by choice. They got told by the EU they could not be secret abusers anymore, so now they decided to be irritating on top.

This situation is superbly paradoxical. And not just because Americans complaining about EU having too much influence outside of its border is peak irony.

But also because, while people think terrible banners are legal requirements enforced from the EU, most cookie banners are actually... illegal according to the EU law.

Let's look at the law again:

> "Consent should be given by a clear affirmative act establishing a freely given, **specific, informed and unambiguous indication of the data subject's agreement t**o the processing of personal data relating to him or her, such as by a written statement, including by electronic means, or an oral statement. This could include ticking a box when visiting an internet website, choosing technical settings for information society services or another statement or conduct which clearly indicates in this context the data subject's acceptance of the proposed processing of his or her personal data. **Silence, pre-ticked boxes or inactivity should not therefore constitute consent**."

So when you see all those dark patterns trying to trick you into accepting tracking, it's actually completely the opposite of meeting the legal requirement.

In fact, if the button to accept tracking is easy to act on, it should specifically comes with an equally easy way to reject it:

> "The data subject shall have the right to withdraw his or her consent at any time. The withdrawal of consent shall not affect the lawfulness of processing based on consent before its withdrawal. Prior to giving consent, the data subject shall be informed thereof. **It shall be as easy to withdraw as to give consent.**"

Really, Europe is not the enemy, here. The companies behave like high school bullies.

But can you use cookie at all? Yes, yes of course you can.

> This shall not prevent any technical storage or access for the sole purpose of carrying out the transmission of a communication over an electronic communications network, or as strictly **necessary in order for the provider of an information society service explicitly requested by the subscriber or user to provide the service**.

Session cookies are OK. You can also setup anonymous stats on the server side as long as you don't tie it to users personal data. Spam prevention, moderation and reputation all require to get and store data from users, after all.

[Hacker news](https://news.ycombinator.com/) have all that and more, but don't have a cookie banner. No problem.

I have read people arguing that when you make a law, you should have known the consequences of it, and therefore the EU is responsible, not the companies.

It’s a dangerous line of thinking, that would absolve anybody making immoral but legal calls, and put the state as an omniscient dictatorial regulator.

Not to mention there is such thing as the spirit of the law, and luckily judges take that in consideration.

Laws of men are not the laws of physics. We assume people are not sociopath and that most of them wants to be good citizen.

First, spread the message, so that we, as a society, keep fair scores. We want misbehaving entities to get the reputation their deserve.

Second, install [an extension](https://addons.mozilla.org/en-US/firefox/addon/i-dont-care-about-cookies/) that blocks those aggressive banners, and [one to block tracking](https://addons.mozilla.org/en-US/firefox/addon/ublock-origin/).

Finally use Firefox, because on mobile, it's the only browser with a good enough add-on ecosystem to block all this.