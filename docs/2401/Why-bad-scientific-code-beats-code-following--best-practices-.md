<!--yml
category: 未分类
date: 2024-05-27 14:33:44
-->

# Why bad scientific code beats code following "best practices"

> 来源：[https://yosefk.com/blog/why-bad-scientific-code-beats-code-following-best-practices.html](https://yosefk.com/blog/why-bad-scientific-code-beats-code-following-best-practices.html)

I've just read "[The Low Quality of Scientific Code](http://techblog.bozho.net/?p=1423)", which claims that code written by scientists comes out worse than it would if "software engineers" were involved.

I've been working, for more than a decade, in an environment dominated by people with a background in math or physics who often have sparse knowledge of "software engineering".

Invariably, the biggest messes are made by the minority of people who do define themselves as programmers. I will confess to having made at least a couple of large messes myself that are still not cleaned up. There were also a couple of other big messes where the code luckily went down the drain, meaning that the damage to my employer was limited to the money wasted on my own salary, without negative impact on the productivity of others.

I claim to have repented, mostly. I try rather hard to keep things boringly simple and I don't think I've done, in the last 5-6 years, something that causes a lot of people to look at me funny having spent the better part of the day dealing with the products of my misguided cleverness.

And I know a few programmers who have explicitly *not* repented. And people look at them funny and they think they're right and it's everyone else who is crazy.

In the meanwhile, people who "aren't" programmers but are more of a mathematician, physicist, algorithm developer, scientist, you name it commit sins mostly of the following kinds:

*   Long functions
*   Bad names (m, k, longWindedNameThatYouCantReallyReadBTWProgrammersDoThatALotToo)
*   Access all over the place – globals/singletons, "god objects" etc.
*   Crashes (null pointers, bounds errors), largely mitigated by valgrind/massive testing
*   Complete lack of interest in parallelism bugs (almost fully mitigated by tools)
*   Insufficient reluctance to use libraries written by clever programmers, with overloaded operators and templates and stuff

This I can deal with, you see. I somehow rarely have a problem, if anyone wants me to help debug something, to figure out what these guys were trying to do. I mean in the software sense. Algorithmically maybe I don't get them fully. But what variable they want to pass to what function I usually know.

Not so with software engineers, whose sins fall into entirely different categories:

*   Multiple/virtual/high-on-crack inheritance
*   7 to 14 stack frames composed principally of thin wrappers, some of them function pointers/virtual functions, possibly inside interrupt handlers or what-not
*   Files spread in umpteen directories
*   Lookup using dynamic structures from hell – dictionaries of names where the names are concatenated from various pieces at runtime, etc.
*   Dynamic loading and other grep-defeating techniques
*   A forest of near-identical names along the lines of DriverController, ControllerManager, DriverManager, ManagerController, controlDriver ad infinitum – all calling each other
*   Templates calling overloaded functions with declarations hopefully visible where the template is defined, maybe not
*   Decorators, metaclasses, code generation, etc. etc.

The result is that you don't know who calls what or why, debuggers are of moderate use at best, IDEs & grep die a slow, horrible death, etc. You literally have to give up on ever figuring this thing out before tears start flowing freely from your eyes.

Of course this is a gross caricature, not everybody is a sinner at all times, and, like, I'm principally a "programmer" rather than "scientist" and I sincerely believe to have a net positive productivity after all – but you get the idea.

Can scientific code benefit from better "software engineering"? Perhaps, but I wouldn't trust software engineers to deliver those benefits!

Simple-minded, care-free near-incompetence can be better than industrial-strength good intentions paving a superhighway to hell. The "real world" outside the computer is full of such examples.

Oh, and one really mean observation that I'm afraid is too true to be omitted: idleness is the source of much trouble. A scientist has his science to worry about so he doesn't have time to complexify the code needlessly. Many programmers have no real substance in their work – the job is trivial – so they have too much time on their hands, which they use to dwell on "API design" and thus monstrosities are born.

(In fact, when the job is far from trivial technically and/or socially, programmers' horrible training shifts their focus away from their immediate duty – is the goddamn thing actually working, nice to use, efficient/cheap, etc.? – and instead they declare themselves as responsible for nothing but the sacred APIs which they proceed to complexify beyond belief. Meanwhile, functionally the thing barely works.)