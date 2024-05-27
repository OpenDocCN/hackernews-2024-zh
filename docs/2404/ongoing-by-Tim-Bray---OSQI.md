<!--yml
category: 未分类
date: 2024-05-27 12:49:35
-->

# ongoing by Tim Bray · OSQI

> 来源：[https://www.tbray.org/ongoing/When/202x/2024/04/01/OSQI](https://www.tbray.org/ongoing/When/202x/2024/04/01/OSQI)

I propose the formation of one or more “Open Source Quality Institutes”. An OSQI is a public-sector organization that employs software engineers. Its mission would be to improve the quality, and especially safety, of popular Open-Source software.

Why? · The [XZ-Utils backdoor](https://en.wikipedia.org/wiki/XZ_utils_backdoor) (let’s just say **#XZ**) launched the train of thought that led me to this idea. If you read the story, it becomes obvious that the key vulnerability wasn’t technical, it was the fact that a whole lot of Open-Source software is on the undermaintained-to-neglected axis, because there’s no business case for paying people to take care of it. Which is a problem, because there is a *strong* business case for paying people to attack it.

There are other essential human activities that lack a business case, for example tertiary education, potable water quality, and financial regulation. For these, we create non-capitalist constructs such as Universities and Institutes and Agencies, because society needs these things done even if nobody can make money doing them.

I think we need to be paying more attention to the quality generally, and safety especially, of the Open-Source software that has become the underlying platform for, more or less, our civilization. Thus OSQI.

They’re out to get us · For me, the two big lessons from **#XZ** were first, the lack of resources supporting crucial Open-Source infrastructure, but then and especially, the demonstration that the attackers are numerous, skilled *and patient*. We already knew about numerous and skilled but this episode, where the attacker was already well-embedded in the project [by May 2022](https://www.mail-archive.com/xz-devel@tukaani.org/msg00562.html), opened a few eyes, including mine.

The advantage, to various flavors of malefactor, of subverting core pieces of Open-Source infrastructure, is incalculable. **#XZ** was the one we caught; how many have we missed?

What’s OSQI? · It’s an organization created by a national government. Obviously, more nations than one could have an OSQI.

The vast majority of the staff would be relatively-senior software engineers, with a small percentage of paranoid nontechnical security people (see [below](OSQI#p-21)). You could do a lot with as few as 250 people, and the burdened cost would be trivial for a substantial government.

Since it is a matter of obvious fact that every company in the world with revenue of a billion or more is existentially dependent on Open Source, it would be reasonable to impose a levy of, say, 0.1% of revenue on all such companies, to help support this work. The money needn’t be a problem.

Structure · The selection of software packages that would get OSQI attention would be left to the organization, although there would be avenues for anyone to request coverage. The engineering organization could be relatively flat, most people giving individual attention to individual projects, then also ad-hoc teams forming for tool-building or crisis-handling when something like **#XZ** blows up.

Why would anyone work there? · The pay would be OK; less than you’d make at Google or Facebook, but a decent civil-service salary. There would be no suspicion that your employer is trying to enshittify anything; in fact, you’d start work in the morning confident that you’re trying to improve the world. The default work mode would be remote, so you could live somewhere a not-quite-Google salary would support a very comfortable way of life. There would be decent vacations and benefits and (**gasp**) a pension.

And there is a certain class of person who would find everyday joy in peeking and poking and polishing Open-Source packages that are depended on by millions of programmers and (indirectly) billions of humans. A couple of decades ago I would have been one.

I don’t think recruiting would be a problem.

So, what are OSQI’s goals and non-goals?

Goal: Safety · This has to come first. If all OSQI accomplishes is the foiling of a few **#XZ**-flavor attacks, and life becoming harder for people making them, that’s just fine.

Goal: Tool-building · I think it’s now conventional wisdom that Open Source’s biggest attack surfaces are dependency networks and build tools. These are big and complex problems, but let’s be bold and set a high bar:

> Open-Source software should be built deterministically, verifiably, and reproducibly, from signed source-code snapshots. These snapshots should be free of generated artifacts; every item in the snapshot should be human-written and human-readable.

For example: As [Kornel](https://mastodon.social/@kornel) said, [Seriously, in retrospect, #autotools itself is a massive supply-chain security risk.](https://mastodon.social/@kornel/112187783363254917) No kidding! But then everyone says “What are you gonna do, it’s wired into everything.”

There are alternatives; I know of [CMake](https://cmake.org) and [Meson](https://mesonbuild.com). Are they good enough? I don’t know. Obviously, GNU AutoHell can’t be swept out of all of the fœtid crannies where it lurks and festers, but every project from which it is scrubbed will present less danger to the world. I believe OSQI would have the scope to make real progress on this front.

Non-goal: Features · OSQI should never invest engineering resources in adding cool features to Open-Source packages (with the possible exception of build-and-test tools). The Open-Source community is bursting with new-features energy, most coming from people who either want to scratch their own itch or are facing a real blockage at work. They are way better positioned to make those improvements than anyone at OSQI.

Goal: Maintenance · Way too many deep-infra packages grow increasingly unmaintained as people age and become busy and tired and sick and dead. As I was writing this, a [plea for help](https://github.com/libexpat/libexpat/blob/R_2_6_2/expat/Changes) came across my radar from Sebastian Pipping, the excellent but unsupported and unfunded maintainer of [Expat](https://github.com/libexpat/libexpat/tree/R_2_6_2), the world’s most popular XML parser.

And yeah, he’s part of a trend, one that notably included the now-infamous [XZ-Utils](https://en.wikipedia.org/wiki/XZ_Utils) package.

And so I think one useful task for OSQI would be taking over (ideally partial) maintenance duties for a lot of Open-Source projects that have a high ratio of adoption to support. In some cases it would have to take a lower-intensity form, let’s call it “life support”, where OSQI deals with vulnerability reports but flatly refuses to address any requests for features no matter how trivial, and rejects all PRs unless they come from someone who’s willing to take on part of the maintenance load.

One benefit of having paid professionals doing this is that they will blow off the kind of social-engineering harassment that the **#XZ** attacker inflicted on the XZ-Utils maintainer (see [Russ Cox’s excellent timeline](https://research.swtch.com/xz-timeline)) and which is unfortunately too common in the Open-Source world generally.

Goal: Benchmarking · Efficiency is an aspect of quality, and I think it would be perfectly reasonable for OSQI to engage in benchmarking and optimization. There’s a non-obvious reason for this: **#XZ** was unmasked when a Postgres specialist noticed performance problems.

I think that in general, if you’re a bad person trying to backdoor an Open-Source package, it’s going to be hard to do without introducing performance glitches. I’ve [long advocated](/ongoing/When/202x/2021/05/15/Testing-in-2021#p-13) that unit and/or integration tests should include a benchmark or two, just to avert well-intentioned performance regressions; if they handicap bad guys too, that’s a bonus.

Goal: Education and evangelism · OSQI staff will develop a deep shared pool of expertise in making Open-Source software safer and better, and specifically in detecting and repelling multiple attack flavors. They should share it! Blogs, conferences, whatever. It even occurred to me that it might make sense to structure OSQI as an educational institution; standalone or as a grad college of something existing.

But what I’m talking about isn’t refereed JACM papers, but what my Dad, a Professor of Agriculture, called “Extension”: Bringing the results of research directly to practitioners.

Non-goal: Making standards · The world has enough standards organizations. I could see individual OSQI employees pitching in, though, at the IETF or IEEE or W3C or wherever, with work on Infosec standards.

Which brings me to…

Non-goal: Litigation · Or really any other enforcement-related activity. OSQI exists to fix problems, build tools, and share lessons. This is going to be easier if nobody (except attackers) sees them as a threat, and if staff don’t have to think about how their work and findings will play out in court.

And a related non-goal…

Non-goal: Licensing · The intersection between the class of people who’d make good OSQI engineers and those who care about Open-Source licenses is, thankfully, very small. I think OSQI should accept the license landscape that exists and work hard to avoid thinking about its theology.

Non-goal: Certification · Once OSQI exists, the notion of “OSQI-approved” might arise. But it’d be a mistake; OSQI should be an *engineering* organization; the cost (measured by required bureaucracy) to perform certification would be brutal.

Goal: Transparency · OSQI can’t afford to have any secrets, with the sole exception of freshly-discovered but still-undisclosed vulnerabilities. And when those vulnerabilities are disclosed, the story of their discovery and characterization needs to be shared entirely and completely. This feels like a bare-minimum basis for building the level of trust that will be required.

Necessary paranoia · I discussed above why OSQI might be a nice place to work. There will be a downside, though; you’ll lose a certain amount of privacy. Because if OSQI succeeds, it will become a super-high-value target for our adversaries. In the natural course of affairs, many employees would become committers on popular packages, increasing their attractiveness as targets for bribes or blackmail.

I recall once, a very senior security leader at an Internet giant saying to me “We have thousands of engineers, and my job requires me to believe that at least one of them also has another employer.”

So I think OSQI needs to employ a small number of paranoid traditional-security (not Infosec) experts to keep an eye on their colleagues, audit their finances, and just be generally suspicious. These people would also worry about OSQI’s physical and network security. Because attackers gonna attack.

Pronunciation · Rhymes with “bosky”, of course. Also, people who work there are OSQIans. I’ve grabbed “osqi.org” and will cheerfully donate it in the long-shot case that this idea gets traction.

Are you serious? · Yeah. Except for, I no longer speak with the voice of a powerful employer.

Look: For better or for worse, Open Source won. *[Narrator: Obviously, for better.]* That means it has become crucial civilizational infrastucture, which governments should actively support and maintain, just like roads and dams and power grids.

It’s not so much that OSQI, or something like it, is a good idea; it’s that *not* trying to achieve these goals, in 2024, is dangerous and insane.

* * *

* * *