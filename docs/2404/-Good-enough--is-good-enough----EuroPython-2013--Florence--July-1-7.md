<!--yml
category: 未分类
date: 2024-05-27 13:40:17
-->

# "Good enough" is good enough! — EuroPython 2013: Florence, July 1–7

> 来源：[https://ep2013.europython.eu/conference/talks/good-enough-is-good-enough](https://ep2013.europython.eu/conference/talks/good-enough-is-good-enough)

Our culture’s default assumption is that everybody should always be striving for perfection – settling for anything less is seen as a regrettable compromise. This is wrong in most software development situations: focus instead on keeping the software simple, just “good enough”, launch it early, and iteratively improve, enhance, and re-factor it. This is how software success is achieved!

n a 1989 keynote speech at a Lisp conference, Richard Gabriel had a “light relief” section where he caricatured a SW development approach he called “worse is better” (AKA “New Jersey approach”) and contrasted it with what he called “the right thing” (AKA “MIT/Stanford approach”)… and despite the caricatural aspects reluctantly concluded that NJ was the most viable approach, identifying several of the actual reasons (speed of development, less monolithic designs, systems more easily adaptable to a variety of uses [including changes in the underlying requirements], ease of gradual incremental improvement over time, …).

The debate hasn’t died down since (Gabriel himself contributing richly to both sides (!), sometimes under the pseudonym “Nickieben Bourbaki”). My favorite Gabriel quote is “The right-thing philosophy is based on letting the experts do their expert thing all the way to the end before users get their hands on it [snip] Worse-is-better takes advantage of the natural advantages of incremental development. Incremental improvement satisﬁes some human needs”.

However, while the debate is still raging, reality has steadily been shifting away from “the right thing” (inherently “Cathedral”-centralized, with “Big Design Up Front” a must, conceived with academia and large firms in mind, and quite unsuited to always-shifting real-world requirements) and towards “the NJ approach” (suited to “Bazaar”-like structures, agile and iterative enhancement, dynamic start-ups and independent developers, in a world of always-shifting specs).

In this talk, I come down strongly on the side of “the NJ approach”, illustrating it and defending it on both philosophical and pragmatical grounds.

I draw technical examples from several areas where the systems that won the “mind-share battles” did so by focusing on pragmatic simplicity (“good enough”) to the expense of theoretical refinement and completeness (the quest for elusive perfection), leading to large ecosystems of developers bent on incremental improvement—the TCP/IP approach to networking contrasted with ISO/OSI, the HTTP/HTML approach to hypertext contrasted with Xanadu, early Unix’s simplistic (but OK) approach to interrupted system calls versus Multic’s and ITS’s perfectionism.

Within Python, I show how metaclasses’ quest for completeness yielded excessive complexity (and 80% of their intended uses can now be obtained via class decorators for 20% of the complexity), and how well incremental improvement worked instead in areas such as sorting, generators, and “guaranteed”-finalization semantics.

The talk is not about lowering expectations: our dreams must stay big, bigger than we can achieve. It’s about the best practical track towards making such dreams reality—think grandiose, act humble. “Rightly traced and well ordered: what of that? // Speak as they please, what does the mountain care? // Ah, but a man’s reach should exceed his grasp // Or what’s a heaven for? All’s silver-grey // Placid and perfect with my art: the worse!”

This talk is probably not perfect, but I do think it’s good enough.