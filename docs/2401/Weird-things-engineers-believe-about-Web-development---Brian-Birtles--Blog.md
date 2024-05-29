<!--yml
category: 未分类
date: 2024-05-27 14:34:57
-->

# Weird things engineers believe about Web development - Brian Birtles’ Blog

> 来源：[https://birtles.blog/2024/01/06/weird-things-engineers-believe-about-development/](https://birtles.blog/2024/01/06/weird-things-engineers-believe-about-development/)

*I wrote most of this post sometime in 2022 but I think it holds up alright in 2024 so I decided to publish it for posterity. I don’t really like doing posts like this—I’d much rather share some innocuous learnings or tips but it turns out I have opinions too* 😓 *Sorry!*

***2024-02-21**: I’ve added a few reflections at [the end of the post](#reflections-a-couple-of-months-later).*

***2024-04-27**: Developpez.com has produced a [French translation of this post](https://web.developpez.com/actu/356070/Les-ingenieurs-croient-des-choses-etranges-sur-le-developpement-Web-par-Brian-Birtles/).*

Since I quit Mozilla and went back to full-time Web development, I’ve discovered a few surprises. It turns out Web development is actually pretty hard, Web developers are actually very smart, and some of these frameworks and techniques we mocked as browser engineers aren’t so bad. Oops.

At the same time, it turns out some Web developers have ideas about browsers and the Web that, as a former browser engineer and standards editor, I’m a bit dubious of.

Here are a few of the things that surprised me.

1.  [“Web browser engineers know Web development really well”](#web-browser-engineers-know-web-development-really-well)
2.  [“The people who make Web specifications know Web development really well”](#the-people-who-make-web-specifications-know-web-development-really-well)
3.  [“Web developers know Web development really well”](#web-developers-know-web-development-really-well)
4.  [“Browsers aren’t made to run SPAs”](#browsers-arent-made-to-run-spas)
5.  [”MPAs will replace SPAs”](#mpas-will-replace-spas)
6.  [“All sites should work without JavaScript”](#all-sites-should-work-without-javascript)
7.  [“Web development shouldn’t need a build step”](#web-development-shouldnt-need-a-build-step)
8.  [“My blog is representative of Web development at large”](#my-blog-is-representative-of-web-development-at-large)

It’s easy to imagine that Web browser engineers, who write the code that makes up the Web platform, must know the Web inside-out like no-one else.

The trouble is, writing Web browsers is hard.

Most browser engineers focus on a particular area and know it really well with only a superficial understanding of other areas. Furthermore, browser platform engineers are writing C++ and Rust code all day long with a smattering of very simple JavaScript test cases. On top of that, they contribute to one massive respository where someone else takes care of all the tooling.

As a result, in their day jobs browser engineers are not fighting webpack, not trying to understand full-page TypeScript errors (they’ve got C++ template errors to fill *that* hole in their lives), not trying to get Safari on iOS to behave like other browsers, not struggling with CSS at scale, not trying to evaluate if the latest SSR/SSG/island framework is worth investing in, not refactoring massive JS codebases for more optimal chunking, not filing exasperated issues on GitHub because the latest version of one of their dependencies broke their app, not trying to get all their tools to agree on ESM vs CJS, and not losing sleep over whether or not they chose the right state management approach or if they should rewrite the entire thing for the tenth time over.

In short, they’re not doing Web development day in and day out so they know much less about real-world Web development than Web developers probably expect.

Now the engineers working on browser *developer tools* and the browser *front-end* often *are* using JS day-to-day and certainly have more awareness of the issues, but it’s still a few degrees removed from regular Web development. For example, they only need to target a single browser engine’s platform features, and often only a single browser version (being able to use the new and shiny features without worrying about compatibility is *amazing*), don’t need to worry about bundle size, or servers, or being offline, or so many other issues that make Web development hard.

Obviously some browser engineers have hobby projects, but the constraints on a hobby project are a world apart from being in a startup where you live or die by the success of your Web app.

I started my career in graphic design and Web development and even after I started contributing to Firefox I worked at a Web development company in Osaka for a time and produced some Web apps at Mozilla Japan too. However, after I quit Mozilla and got back into Web development full-time I was shocked at how much it had changed and how little I knew about it.

My experience as a browser engineer has been incredibly useful in Web development not least because I know where to look, who to ask, and where to file bugs, but I’d be kidding myself if I said that being a browser engineer automatically qualified me as a Web developer.

During my time at Mozilla, the Firefox OS days were by far the most enjoyable. We had internal teams building real-world mobile Web apps that we, as the platform team, had to debug and make successful as our highest priority. I saw how [`transitionend` events](https://developer.mozilla.org/docs/Web/API/Element/transitionend_event) could be unreliable and cause Web apps to be buggy and overly complicated and so I proposed and implemented the [`transitioncancel` event](https://developer.mozilla.org/docs/Web/API/Element/transitioncancel_event) and Web Animations’ [`Animation.finished` promise](https://developer.mozilla.org/docs/Web/API/Animation/finished).

But even working side-by-side with Web developers could not prepare me for actually *being* a full-time Web developer again. For the most part browser engineers just operate in a different world from Web developers and might not be the Web developer superheroes we imagine them to be.

Ok, but surely the people working on Web *standards* and *specifications* (who, it turns out, are mostly browser engineers) must be well-versed in Web development, right?

Back in 2012, [Brendan Eich pointed out](https://brendaneich.com/2012/02/community-prioritized-web-standards/) that, “Standards-making like law-making is definitely [sausage-making](https://en.wikiquote.org/wiki/John_Godfrey_Saxe)” referring to the following quote from John Godfrey Saxe:

> Laws, like sausages, cease to inspire respect in proportion as we know how they are made.

As a Web developer, it’s easy to imagine a group of people, infinitely wise about all things related to Web technology, making calm, rational decisions based on the technical merits of each proposal balanced against a thorough understanding of industry needs.

That illusion typically won’t last your first working group meeting. Despite the best of intentions, sometimes decisions get made based, at least in part, on one person’s charisma or forceful personality, on who happens to be in the room at the time, on how tired everyone is, or, dare I suggest, maybe even someone’s need to ship a feature in order to fill out their promotion packet.

That sounds very cynical so let me make two clarifications.

Firstly, the folks on these groups are well-meaning, wonderful people. Furthermore, they are often aware of their limitations and try their best to elicit Web developer feedback. Unfortunately, I’ve yet to see a group do this very successfully. There are Twitter/X polls, for example, but they tend to only be answered by the Web developers on the bleeding edge and are easily skewed by *who* spreads the word about the poll.

Secondly, I haven’t had much experience with WHATWG specs like HTML and DOM where decisions appear to be made asynchronously (“any decisions made during in-person meetings are to be considered non-binding”—[WHATWG meetings](https://whatwg.org/working-mode#meetings)), but my impression is that they seem to make better decisions. Folks like Anne van Kesteren, Simon Pieters, and Domenic Denicola probably know the Web better than anyone else on the planet. But even that is not the same as knowing Web development.

As a browser engineer it’s really satisfying to ship a new platform feature. There are articles about it on Smashing Magazine and CSS tricks and Twitter/X goes abuzz with the news. It’s easy to think that now the whole world now knows about this great new advance in the Web’s capabilities.

At one point a few years ago, a few of us in Mozilla’s Japan team decided to interview local Web developers in Tokyo to learn what developer tools they would benefit from.

The results were surprising.

Many didn’t know about new CSS features that had shipped 10 years ago. What’s more, even when we told them about them, they didn’t seem too excited. They were doing just fine with jQuery and WordPress, thank you.

Instead, they were having trouble with things like, “When I show clients a site in responsive design mode, if the window doesn’t have a mockup of an iPhone around it, clients can’t grasp that they are looking at a preview of how the site will look on a smartphone. I really need that iPhone mockup.”

As a browser engineer involved in developing new Web standards I was a little disappointed but my lasting impression was of the constraints on these folks who are making their living from shipping Web sites and Web apps.

Unlike people working on browsers or well-funded Silicon Valley startups, many of these people are working for little shops with considerable pressure to deliver something quickly and then move onto the next project in order to pay the bills. They don’t have the luxury of spending a morning tinkering with upcoming technologies and instead reach for tried and trusted solutions they have experience with.

Another surprise from moving from browser development back to Web development was some of the assertions about how browsers work.

When I worked on animations, I was surprised at how many people believed that some animations “run on the GPU” (the browser can offload some animations to a separate process or thread that updates animations using the GPU to composite or even paint each frame but it doesn’t offload them to the GPU wholesale) but that was a fairly minor misunderstanding compared to some of the other ones that get thrown around like, “browsers aren’t made to run SPAs (single page applications)“.

I believe the argument here is that originally browsers would load content off the network, progressively laying it out and rendering it and have been heavily optimized for this. Dynamic content came later and so has been less heavily optimized.

Having worked on a browser on and off for nearly two decades I’m not convinced this is the case, or at least not anymore.

After all, the Firefox front-end and developer tools are, in effect, SPAs. The developer tools in particular are written using React and Redux and are in every sense an SPA.

To the argument that browsers are deficient at handling complex and long-lived DOM trees with dynamic changes made via JavaScript, the browser itself stands in contradiction to that claim.

An argument could be made that *on mobile* browsers aren’t optimized to run SPAs. After all, on Android, Firefox switched from an HTML rendered browser chrome to a native browser chrome in order to improve performance. I’m not in a position to comment on what the particular performance constraints were that lead to that change, but [a blog from that time](https://lucasr.org/2011/11/15/native-ui-for-firefox-on-android/) suggests it was related to improving app startup time and panning and scrolling performance, neither of which point to browsers being deficient at handling SPAs compared to other architectures.

”Ok, so maybe browsers can handle complex long-lived DOM trees with frequent dynamic changes, but SPAs tend to have large JS bundles that are slow to download and parse, blocking the initial render.” That’s a fair argument, but it’s an argument for smaller render-blocking initial bundles, which applies to equally to your average WordPress site, not an argument that browsers are somehow unsuited to running SPAs.

While we’re talking about SPAs and “the one true way to write Web apps”, a more recent variation on the “browsers can’t handle SPAs” take is, “MPAs (multi-page applications) will replace SPAs”.

I’m pretty excited about MPAs. More specifically, as someone who is involved with a lot of animation specs, I’m excited by the [view transitions spec](https://drafts.csswg.org/css-view-transitions-1/) spec. It’s something we wanted to do at Mozilla for a while, particularly during the Firefox OS days, and made a [proposal](https://www.chrislord.net/2015/04/24/web-navigation-transitions/) to that end. Kudos to Jake and others for finally making it happen.

View transitions were originally implemented for SPAs but have been adapted to work for MPAs and to the extent that they make multi-page sites more enjoyable they are a very welcome addition.

However, building on the “SPAs are bad” thinking, there seems to be a tendency to assume that MPAs are the future and SPAs are on the way out.

Unlike the earlier points in this post, my surprise at this take is not based on my experience with working on browsers, but on my more recent experience with working on Web apps.

First though, what do we even mean by MPAs?

My understanding is that whereas SPAs are characterized by having a long-lived DOM tree or two that are frequently updated by script, MPAs primarily update content by navigating to different HTML resources served from the network. These don’t have to be topmost navigations—it could be by navigating an `<iframe>`, for example. Similarly, SPAs might use `<iframe>`s as a means of chunking but there’s a difference in how content is typically updated.

By this definition, Google Docs is an SPA since although each document is served as a separate resource, most of the time you’re interacting with the one document that is continually updated via JavaScript. YouTube would probably be considered an MPA but it might actually be implemented as an SPA in order to smooth out changes in content, intercepting navigations and replacing the content via script—that is, until view transitions can help with that.

In either case, my surprise at the idea that MPAs will replace *all* SPAs is simple: How would Figma or Photoshop for Web work as an MPA? Or Slack, or Discord, or Google Maps?

I’m currently working on an offline-first mobile Web app that stores data locally and syncs to the server. Wanting to be on the forefront of Web tech I investigated how we could embrace MPAs.

To keep a long story short, if we’re to retain our desired UX which has independently navigable panels and our offline-first requirement, we *could* introduce some `<iframe>`s to partition out some of the functionality into separate HTML requests (as opposed to separate script chunk requests) and we could probably even pre-render some chunks sometimes. The trouble is it would increase the complexity ten-fold (two-way sync becomes three-way sync for a start) while providing no benefit to customers whatsoever—instead it would almost certainly lead to more bugs, more latency, and shipping features more slowly.

Given that our app is not currently released I realise that’s a fairly weak argument since no-one can look at the app and suggest a better approach so I’m just asking you to trust me on this one. I tried. I really did. It’s just not the right architecture for this particular app and I’m surprised by the suggestion that *everything* should be an MPA.

Continuing on our theme of Web development best practices, building a site that works without JavaScript is an admirable goal. Doing so probably means it degrades gracefully, has no JS blocking the initial render, and works on a wide range of browsers. But I’ve been surprised to see how often this becomes dogmatic: “*all* sites should work without JavaScript”.

I guess it’s easy to come to this conclusion if *your* site is able to work without JavaScript (see [“My blog is representative of Web development at large”](#my-blog-is-representative-of-web-development-at-large)) but it feels a little myopic to me.

I’ve mentioned Figma and Photoshop for Web before; it’s hard to imagine how they could work without JavaScript. Likewise for a browser’s developer tools. Or even [Parapara Animation](/2012/01/27/parapara-animation/)!

Furthermore, although many advocates against JS are also concerned about accessibility, JavaScript is often necessary in order to make an app accessible.

One thing the accessibility folks at Mozilla taught me about keyboard navigation was that `Tab` navigation should be fairly coarse. That is, you use `Tab` to navigate to the control group (e.g. toolbar) and then use the arrow keys to move within that group. That allows you to move around an app more quickly without having to `Tab` through every single control first. WAI calls this a [“roving tabindex”](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/#kbd_roving_tabindex).

However, in order to implement this kind of coarse-grained keyboard navigation you’re going to need JavaScript. If you want to make the arrow-key based navigation two-dimensional, you’re going to need even more JavaScript. Maybe one day we’ll fill that gap in the platform (looking at you, [focusgroup](https://open-ui.org/components/focusgroup.explainer/)) but for now you should feel no shame about using client-side JavaScript to make your app accessible.

Honestly, I think some sites should use *more* JavaScript.

As an example, the [Eleventy documentation](https://www.11ty.dev/docs/) seems to avoid using client-side JavaScript for the most part. As Eleventy supports various templating languages it provides code samples in each of the different languages. Unfortunately, it doesn’t record which language you’ve selected so if your chosen language is not the default one, you are forced to change tabs on *every single* code sample. A little client-side JavaScript here would make the experience so much more pleasant for users.

*Update (2024-01-11): It seem that Eleventy have taken this feedback onboard and [fixed this](https://twitter.com/eleven_ty/status/1745156260649959708). Thank you!* 🙏

*While just about everything in this post is from 2022, while tidying it up I couldn’t resist adding just one more recent idea that surprised me.*

Our final stop on the Web development dogma train is a view that’s come up a few times but still surprises me. The most recent rendition I saw went something like, “we’re so blind and stubborn that we’ve ended up with a hugely complex toolchain but we really should be able to ship Web apps without a build step at all”.

As someone who spent most of his career working with compiled languages the desire to go without a build step is surprising. The things compiler engineers do amaze me. They are geniuses who layer clever optimization on top of clever optimization transforming my very average code into something unrecognizable and insanely fast. If anything, I want more of that compiler magic in my Web development.

Obviously JavaScript presents its own challenges since it can be very hard to statically determine the side effects of a certain operation but I’m sure there’s still more room to explore in optimizing JavaScript at compile time.

Web developers seem to agree on optimizing image assets and pre-generating static HTML pages where it makes sense, why is there resistance to optimizing code assets too? Why defer computation and I/O to runtime that can be done once at build time? If nothing else, I have no desire to ship my megabyte-long comments cursing iOS Safari to every user on every request.

Maybe 2024 will be the year where client-side Rust/WASM frontend frameworks start to get traction and if that’s the case, we’d better get used to having a build step!

A number of the points above could possibly be summarised as “My blog is representative of Web development at large”. That is, coming from browser engineer to Web developer, most of the notions about Web development that have surprised me are the result of people extrapolating their experience of the Web in a way that doesn’t overlap with mine.

Since I left Mozilla over four years ago, I’ve spent most of my time working on a Web app. I’ve also spent *way* too long setting up this blog. Surprisingly, with regards to tooling, architecture, or Web platform features used, I’ve found almost no overlap between the two. It’s almost as if blogs and apps exist in entirely disparate corners of the Web development landscape.

My app is a mass of TypeScript code, my blog uses almost no client-side JavaScript. My app is hugely complicated by two-directional data sync, my blog is read-only. My app uses webpack, Playwright E2E tests, a component framework, and a state management library, my blog uses none of those.

If you’re mostly engaged with one or the other, it’s easy to think that’s what Web development looks like. In truth, Web development is probably more diverse than any of us imagines.

There are other notions I’ve found surprising but the common theme in the above is it’s easy to assume our experience of the Web is representative of Web development in general.

Moving from browser development back to Web development has been humbling because it’s so much broader and deeper than I knew. I have a much greater respect for Web developers, especially those at little Web shops and startups that live or die by the success of their Web apps.

If that’s you, I hope reading this post gives you confidence that even if browser engineers, standards editors, and other Web developers insist on a particular way of doing things, you know your constraints better than anyone else and you’re probably doing things just fine.

Since posting this article I’ve received some very useful feedback which I’ll summarise below:

*   Some people pointed out that the background behind a lot of anti-JavaScript and anti-SPA sentiment is a reaction to seeing static sites being rendered as React SPAs which is a very poor fit. That was helpful context that I hadn’t appreciated.

*   Others clarified that the “no build step” argument is more about local development and not about production assets. That makes a lot of sense and is something I can appreciate.

    My point wasn’t to say you *should* have a build step but simply to say that coming from browser development I was surprised at the anti-build-step sentiment because I’m used to compilers and I’d be interested in seeing what more they can do for us.

*   Some people took issue with some of the later points because I failed to clearly summarise my main contention. Here it is:

    I worked on browsers for over a decade and am still involved in Web standards and browser development. *I love the platform and encourage others to make full use of it.*

    My contention is with the absolute terms with which Web development best practices are sometimes asserted. Web development social media can be a fierce place—something I was sheltered from when working on browsers.

    I’m certainly not trying to establish a new dogma but just to push back and say that using JavaScript, writing an SPA, and having a build step is sometimes the right choice and you shouldn’t feel bad about it.