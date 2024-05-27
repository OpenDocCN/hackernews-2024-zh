<!--yml
category: 未分类
date: 2024-05-27 13:30:37
-->

# I'm giving up — on open source - Blog

> 来源：[https://nutjs.dev/blog/i-give-up](https://nutjs.dev/blog/i-give-up)

A little over a year ago I wrote a blog post about [open source and why I'm charging money for some of my plugins](/blog/money). Sadly, one year later I've reached a point where I'm just not willing to continue the way I did before.

So this is my letter of resignation.

## Why?

Ever since I first started to use Linux I was fascinated by the idea of open source. Almost everything I built on my own is open source, and I'm still contributing to upstream projects I'm using if I encounter things I can improve.

I sponsored an open source project for the first time over ten years ago when I was still in university, because I always held the belief that if a project is valuable for me, it's worth supporting it. And if I don't have the time to contribute to it myself, I should at least support the people that do.

Of course, there are people that explicitly do not want any kind of sponsorship, but if they do, I'm happy to help. Working on an open source project is still work, and if you're doing a good job, you should be rewarded for it. I also always believed that if you ever started a project that is valuable for companies, they would support you in return, at least that's the reason why my own company is a monthly sponsor of [Verdaccio](https://verdaccio.org/) and why I sponsored maintainers of libraries I'm relying on.

So, due to these naive believes, I started to work on nut.js under Apache-2.0 license, because I thought that if companies and individuals alike are able to permissively use my software, they would also be willing to support me in return. Now, before you start judging me that I'm only in for the money, wouldn't you agree that it sounds awesome to work on an open source project full time, and still be able to pay your bills?

Did it work out? No. All I got was complaints.

In the beginning, people were complaining that the image search plugin was baked into the core of nut.js, and that they were forced to use certain compatible versions of node or Electron. Then they started complaining that the image search plugin was not compatible with Apple Silicon. Which I made clear I would not be able to fix without a machine to test on. So if nobody is willing to lend me a machine or sponsor me, so I could get one myself, it's not going to happen.

You think anyone made a move? Nope.

Once I decided to take the investment myself, but charge for the new plugin, I suddenly turned into the greedy asshole that's not giving away everything for free.

Same goes for companies. Nobody cares about you as long as everything is working smoothly, but as soon as they encounter a problem, guess who comes knocking on my door?

~~This public issue on the nut.js repo~~, where I'm publicly accused of something that's entirely not true was the final nail in the coffin.

-- EDIT --

The previously linked issue caused quite a lot of reactions, some good, some not so good.

Even GitHub reached out to me, asking if it would be okay if they would remove the issue, as it took a turn into some difficult direction.

For reference, here's the condensed version of it, with all names and accounts removed:

```
> Not only did you sell out, you also removed all the old versions that were released under an open source license so that others couldn't continue to use out-of-support versions.

`@nut-tree/plugin-ocr` has never been publicly available, nor was any other of the paid addons that were released after introducing non-free addons **almost 2.5 years ago**.

> You should do a better job updating your documentation so that people do not waste their time like I did. This change to closed source was announced where, exactly? All of your READMEs and documentation sites do not mention this

This is mentioned on the nutjs.dev landing page and e.g. [pricing page](https://nutjs.dev/pricing/pricing). The core readme refers to the website for additional information, so I’d say it’s fair enough to expect people to actually read it.

> tl;dr get off GitHub and npm entirely if you want to do the closed-source thing, kthx.

It was a pleasure to meet you, have a great day! 
```

-- END EDIT --

This has happened several times already. I've been insulted for the things I do with nut.js on Discord, Reddit and now GitHub, but this time, I'm not just sucking it up.

It may seem to you that open source is great because it's free to use. Truth is, it certainly is not free. Someone is paying a price for it, and if it's not the user, it's the maintainer.

Everyone's time is valuable, and you may want to spend it wisely. If it's fun to spend time on something, that's great. But if it becomes a burden, it's not fun anymore.

And if people start insulting you for something you're doing in your free time, it's time to stop.

Open source is great, but it's not sustainable. We self-sabotaged ourselves over decades, and now we're at a point where it's hard to turn back. Publishing source code for the greater good is a noble cause, but to be honest, I think that over the years, using "open source" has become an excuse to avoid paying for software.

And who's to blame if something goes wrong? The maintainers, of course.

I've played this game with nut.js for almost six years, but it's coming to an end now.

## What's next?

All of my packages around nut.js will cease to exist publicly on npm. Ready-to-use packages will only be available through the private nut.js package registry, which requires an active subscription to be used.

The GitHub repo will remain public, so if you want to continue using nut.js on your own, you'll have to take care of building, testing and hosting packages yourself.

If you want to save yourself some time and work, you should grab a license today, because prices will also increase with the release of additional plugins. Existing subscribers will not be affected by this increase.

## Will I stop working on nut.js entirely?

Definitely not.

I'll continue to work on nut.js, but updates to the repo will happen with a delay. New features, patches, bug fixes and security updates will be made available to subscribers first.

As I said, if you want to continue using nut.js, you'll have to take care of building, testing and hosting packages yourself.

All the best

Simon