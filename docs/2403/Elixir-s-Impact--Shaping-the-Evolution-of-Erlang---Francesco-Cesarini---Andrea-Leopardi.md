<!--yml
category: 未分类
date: 2024-05-29 12:40:51
-->

# Elixir's Impact: Shaping the Evolution of Erlang • Francesco Cesarini & Andrea Leopardi

> 来源：[https://goto.buzzsprout.com/1714721/14674925](https://goto.buzzsprout.com/1714721/14674925)

**Intro**

Andrea Leopardi: Hey, everyone. Welcome to another episode of GOTO Unscripted that we're in Copenhagen at GOTO Copenhagen 2023\. I'm Andrea Leopardi. I'm from Italy. I'm a member of the Elixir team. Elixir is a functional language that runs on the Erlang virtual machine, and I'm here with Francesco Cesarini.

Francesco Cesarini: So, I'm Francesco Cesarini. I'm the founder and technical director at Erlang Solutions. I have been working with Erlang Ecosystem since the mid-'90s. Very fortunate to have seen a programming language become an ecosystem of languages, and I think that's probably what we're gonna talk about today, right?

Andrea Leopardi: Francesco and I live very close to each other.

**The Expansion of the Erlang Ecosystem** 
Andrea Leopardi: We've never met in Italy, only through conferences, various conferences, but yeah. He's been involved with Erlang a lot longer than I have. I came into the Elixir generation, yeah. So, how was it to see this, like, ecosystem grow or, like, this small language I guess grow?

Francesco Cesarini: It was really interesting. I mean, the first language on the BEAM came about...I discovered it had been written, it was called [inaudible 00:01:17]. I heard about it at the Open Source Conference in Portland in 2008\. And so, when we ran our first conference in America, we invited Tony to come in and present it. It was interesting looking at how, you know, he'd taken a Ruby-like syntax and then applied it to Erlang running on the BEAM, which is the prevalent virtual machine on which we run everything today. And then from there, soon after Lisp Flavored Erlang came about with Robert Virding, Efene by Mariano Guerra. And so, there were a lot of Prologue, you know, got ported. So, there were a lot of languages that came about, and some were even being used in production.

But we never kind of realized it had become an ecosystem of languages probably until 2014 when we were in San Francisco, and we had for the first time at the Erlang factory dedicated a whole track to Elixir. It was still in the early days. I think it was the year the first Elixir Comp happened, so I think something like 140 people including Dave Thomas's dog. I was walking with Garrett Smith and we were discussing this new phenomenon called Elixir and excited about what was happening and how Elixir was opening up the door to whole new communities of programmers. And in that conversation, it kind of transpired, you know, Garrett came up, "Well, yeah, Erlang's becoming now an ecosystem of languages," and that's when it clicked. That's exactly what happened with Java. That's exactly what happened. NET. And the same was happening with Erlang, and yeah, it's been amazing to see. And you look at it today, I think there are over 35 languages running on the BEAM, which is impressive. You know, not all of these are used in production, many are experimental.

**The Flavors of Erlang** 
Andrea Leopardi: And it's interesting, I was speaking with Sam Aaron from Sonic Pi earlier and he was talking about differences between Java and Closure being a lot more than differences between, for example, Erlang and Elixir. And I think that's...do you feel that's true that, like, in...and the languages on the BEAM are much...while there are many flavors of languages on the BEAM and some are statically typed, some, like, have different flavored syntaxes like Lisp, or they look like Prologue, they look like Lisp, or they look sort of like Ruby. Do you feel like the difference...? My feeling is that the differences are like...

Francesco Cesarini: I agree. They're not...the differences aren't major, but my view is that Erlang...they got a lot of things right when they invented Erlang when they created Erlang. They spent probably three years prototyping a certain domain of the problem, you know, the problem domain of scalable, fully tolerant, soft real-time systems. You know, first looking at all existing languages, languages being used in the industry back in the '80s and then from there using a virtual machine implementing a prologue. So by the time they started creating the first virtual machine written in C, they'd come so far in these prototyping activities that they streamlined and created the language with the bare bones and bare necessities.

If you ask the different co-inventors of Erlang what they contributed, you know, Mike Williams once told me, "Oh, I spent most of my time convincing Joe and Robert that even though this was a cool language feature, we didn't need it because didn't add any value." And, you know, these design and prototype activities, they looked at different constructs which they tested, and if this construct added value and reduced the code base and made the task of building these telecom systems easier, they kept it. If it didn't, they removed it. And so, as a result, I think Erlang became a very compact language that was fit for purpose. And so as a result, I think all the language is written on top of the BEAM, you know, get all of those features and then add their flavors to it. Accidentally looked at the T-shirt I got.

Andrea Leopardi: Which flavor of Erlang is your favorite? This is very small. I was wondering though, do you think that these similarities between these languages are also due...at least that's also how I feel in part, but also because Erlang is really hard to extend? Like, it's not...like, the basic data structures in Erlang, for example, are very set. Like, it's just those. And if you look at things like, for example, queues in Erlang, they're implemented as, like, on top of existing data structures doubles and lists, right? Or if you look at Elixir, they have to be implemented on top of maps. They're like something that compiled time time-checked, but they have to be implemented on top of maps. Do you think that's...?

Francesco Cesarini: I mean, everything is possible even with Erlang. But when the language inventors go out and create a programming language, they're out to solve a particular problem. And I think, you know, so the problems they're trying to solve dictates the features which they need. I mean, if you look at macros, for example, they exist in Erlang, but they're made with...you know, they're very well hidden and they're not accessible to everyone. So, they're very hard to use and implement. And that was a conscious decision because they didn't want the language to be extendable. I think one of the problems they were trying to solve was the cost of maintenance. They were trying to reduce maintenance and operational costs in systems, which they did very, very successfully with the toolings and the hooks, introspection hooks you've got into the VM. And again, everything, you know, we were being able to all, all the systems we've been able to solve, it's rare we've gone in and said, you know, we wish we had this or that. Maybe some libraries might have been missing, but not language features because keeping it simple meant you could do things with a few lines of code on top of it if you needed it.

Andrea Leopardi: A super interesting thing that comes to mind is that maps, for example, are a recent Erlang feature, right? And they now feel like a really necessary language feature.

Francesco Cesarini: Absolutely. I mean, maps were first presented, I believe, at the Erlang User Conference by Richard O'Keefe, and what eventually became maps in 2000 but it probably took a decade before they made it into the language.

Andrea Leopardi: Because I remember it was around the time Elixir was becoming 1.0 that they made it to OTPR like 17 maybe is like...

Francesco Cesarini: Correct. Yeah. Correct. And again, this is Ericsson, you know, Ericsson is a benevolent dictator and that's the reality is when you're dealing with open source and they're incredibly conservative over new features because they add to the language. After all, if they add something, they need to support and maintain it.

Andrea Leopardi: For decades.

Francesco Cesarini: And not only that, but everything they do needs to be backward compatible. Because there are tens of millions of lines of coding in production. You make something that isn't backward compatible, you break the build and upgrading will take...yeah, can take years. We've worked on projects where they forked the BEAM and to get the code base back on the latest version of the BEAM, you know, 3, 4 years later, in some cases, it took 6 to 12 months of work and we've seen this on multiple occasions.

Andrea Leopardi: The thing I like though is that I have a wish list of things that would go into Erlang. I think maybe the biggest one would be structs because the interoperability would be nice. Like the fact...

Francesco Cesarini: I think structs is exactly what is stopping the ease of calling Elixir libraries from Erlang.

Andrea Leopardi: Yes, because you have to do some esoteric-looking stuff to get them to work.

Francesco Cesarini: Exactly. And it is one of the gating issues. So, you know, there are conversations ongoing, and one of the missing possibilities is to add structure to the language for compatibility issues. I mean, going back to the very first ElixirCon, if I remember Jose saying, oh, we've got access to all of the libraries written in Erlang, which is decades of work of open source libraries, which we can use from Elixir, and the opposite isn't...you know, you don't have the opposite from Erlang. And it's payback time, so yeah, adding structure would be...you know, that would be one of the huge benefits from it.

Andrea Leopardi: I think so. And there is such like a central, like Elixir seems to have developed around structs a lot. It's very...you see them all over the place, extremely idiomatic, and it would be nice to see them in Erlang core because you could probably significantly optimize tagging this struct for example. After all, we have to use this little construct that takes memory and all these structures that we have in the system, and it would be nice to have it in the BEAM because that would probably change.

Francesco Cesarini: Absolutely. Absolutely.

Andrea Leopardi: So, that's very interesting.

Francesco Cesarini: As a side note, Robert Virding added structs to Lisp Flavored Erlang, which is...yeah, one of the many languages is a Lisp implementation running on the BEAM. And so, yeah, that now allows us to call all the Elixir libraries completely seamlessly.

Andrea Leopardi: I'm curious if Gleam has structures. Gleam is a statically typed language that's compiled via Rust though, like the Gleam compiler's reading Rust and it generates Erlang by code or AST, I don't know. But I wonder if it has structs and I wonder if it...I mean, I would bet that it has, like, records. It would be interesting to know if they actually can get away with not having them tagged because it's a static type language that you can probably do the checking.

Francesco Cesarini: It's a good point. I don't know...

Andrea Leopardi: But one thing that I've noticed in Erlang that's very exciting is that I do feel like it sometimes it can be hard to extend Erlang, the VM itself, which is probably for the best, right? Like, it's a very intricate VM with, like, lots of...it's a big VM, right? It's a very smart VM, like, with the scheduling and the process and the garbage collection has lots of features, and so, like, it would be hard, I think, to make it extensible in any significant way when you go and touch, like, the depth of the VM. But recently it's been...like, the releases of ODP have been exciting. They've added a lot of stuff in the last, just even the last two or three years. Being able to call gen servers I think this is OTP 25 or 26, like very recent, like being able to do asynchronous calls to gen server and, like, check for the response later. All this stuff is, like, getting into core and that's awesome because it allows us to build a lot of these, like, lower-level abstractions for ourselves.

Francesco Cesarini: That is correct.

Andrea Leopardi: I'm a big fan.

**Challenges in Erlang Development** 
Francesco Cesarini: If you look at an actual BEAM itself, I think the biggest challenge in extending and adding things to it is ensuring they don't disrupt the soft real-time properties of the scheduler, which is I think their biggest fear.

Andrea Leopardi: Interestingly, you mentioned this because like one of the people that are familiar with the BEAM and Elixir, and Erlang and Elixir, probably know this, but one of the...maybe the only shared functional data structure, like pure functional data structure lives in shared memories is binaries, right? Like bigger, binaries larger than 64 bytes, so they go into, like, a shared memory area, so they didn't have to move them around the process, you just move references. And that's the only data structure that I know of that's completely shared across. Well, now they have Atomics encounters, but...

Francesco Cesarini: ETS tables are another that has destructive operations.

Andrea Leopardi: Right. But ETS tables are sort of like shared memory errors, right? But it's treated as an impure. You can modify its state, right?

Francesco Cesarini: Correct

Andrea Leopardi: And the stuff with the binaries is not like that. You can't modify the selected binary. And it's so interesting to me that even just binaries which are very pragmatic to be in a, like, separate memory space after 64 bytes and all that, they cause many problems because, like, it turns out that like a lot of memory leaks in Erlang applications are from parsers copying binaries or, like, taking big binaries and splitting them up.

Francesco Cesarini: Memory fragmentation.

Andrea Leopardi: Exactly. And, like, creating copies of these, like, smaller banners or having, like, memory leaks where you have reference to the binary that you don't clear up or the process doesn't die or this or that, and then you have a memory leak because, like, the binary never gets garbage collected. And it's interesting because you notice that, like, even one day the structure that it's built-in still bubbles up to problems for users in some specific use cases. After all, like, it's hard to do this. Like, the BEAM has wonderful properties of software priming per process garbage collection and all this stuff, but they're hard to maintain, you know? Maybe some of it is...

Francesco Cesarini: I think yeah, you've answered your question as to why I think Ericsson has to be very conservative and change. I mean, just the JIT compiler, which, you know, that I mean was at least three, four years in the making. Extensive research, but when that came out, it allowed WhatsApp to reduce its server needs by around 30%. That was when WhatsApp went out and tweeted about it thanking Ericsson for it. And that, in turn, you know, translates to huge energy savings, which is something that we're seeing.

Andrea Leopardi: It's interesting that Elixir itself, it's not in a spot where we're not adding much to it either, right? Like, I think arguably more stuff is coming up into Erlang in the past, like, a couple of years maybe because Elixir has been very...Elixir's ecosystem is expanding incredibly. Like, all the machine learning stuff is nuts. Like, the live view stuff is nuts but…

Francesco Cesarini: But this is the positive. A good part of it is also the positive influence Elixir is having back on Erlang.

Andrea Leopardi: Because people are realizing that a lot of things you have to solve in Erlang. Like, for example, Atomics. All these new features that have been added a persistent term, those are, like, fantastic features that have been added in just the last couple or two or three years, I think, in Erlang. And they, you just cannot do them. Like, something like the persistent term, which is like this shared persistent storage gets this copied tool processes, very fast to read, very painful to write, it's awesome, like, for so many use cases, but you just can't do it in...like, you have to do it at the VM level, right? You can't do it at the...So, it's awesome that these features are, like, popping up in Erlang because, like, from Elixir, we just get them for free, and from...you know, in Erlang, you can just use them. You can use them in Gleam, you can use them. That's the benefit, obviously, of adding features to Erlang.

Francesco Cesarini: That's amusing if you say, oh, nothing's happening with Erlang. No, there's a lot of work happening. You know, Ericsson is doing an amazing job at driving things forward, but it's not so much the language, it's the libraries where the effort is being put in and on the virtual machine. And...

Andrea Leopardi: I think that's tricky because, like, the work that the Erlang team is putting into Erlang is low-level. Like, something like persistent term or something like Atomic or counters, these are low-level stuff that usually you need...99% I think of use cases are for, like, library authors or OTP itself.

Francesco Cesarini: Exactly. And the user doesn't see it.

Andrea Leopardi: Are mostly used for gen server calls and, like, they're used by OTP itself, right? So, you don't get to...And the advancements and the news that are happening in Elixir are...like, they seem more exciting because they're, like, much easier. It's like, oh, there's a new machine learning library that lets you, like, pick up models from [inaudible 00:18:47] and run your machine learning models. Like, that's something I can touch very easily that can get easily to users or live new features in live view. They get right into the hands of users. And instead, the work that Erlang is doing is just as important and just as prolific, right? Like, they're doing a lot of stuff. It's just that it's so within maybe the depth...

Francesco Cesarini: It's much more low-level. It's a low level where you need to have...

Andrea Leopardi: So, it's harder to get it to users, you know?

Francesco Cesarini: It's where you need to have full control over what's going on.

Andrea Leopardi: But it does have an impact. I mean, for libraries, I've used counters, I've used Atomics, I've used new features, like, and they've been awesome.

**Match Specifications in Erlang** 
Francesco Cesarini: An amazing new feature, which I discovered reading the manual pages a couple of weeks ago, is the ability to match specifications. So, a match specification, for those of you who don't know, is a program that provides a set of...it's a term, which consists of a set of variables, a set of logical operations you can do on these variables, and then a set of actions. And so, it's a very simple program. It has a very, very small subset of operations and actions you can take, and it was originally used for tracing local and global function calls in runtime. You know, we're having to trace, and compile the code, and you wanted to turn on or turn off the trace or trigger trace messages when certain conditions were met.

So, you know, only if, you know, the first argument to a local call was user Andrea Leopardi and the second argument was to have them call home, for example. At that level of granularity, you could trigger a trace message. If you knew, you know, that was where, you know, you needed to go in and troubleshoot. And, you know, this has been extended to message passing, to pattern matching. It's just been extended to a large variety of things. It also works on ETS tables. It has worked on the ETS table for a while, but, you know. So basically, you know, the report, the power of tracing off, kind of, sequential calls to a lot of other constructs where, again, you know, a process receives a particular message, you trigger, you know, you start tracing, you know?

Andrea Leopardi: It's a very fast way of, like, matching on stuff and extracting information, which is why it's allowed in ETS tables.

Francesco Cesarini: It's not for the faint of heart, especially if you're doing it in a live system. But what is kind of hard to reason and hard to figure out bugs. Again, one of the points of Erlang was to reduce operational costs. And this reduces the time it takes to troubleshoot systems and find that needle in a haystack.

**Elixir's Advances in Compiler Tracing & Machine Learning** 
Andrea Leopardi: I think Elixir is doing a...I'm not teaming up with any of it, that's why I allow myself to say it, but it's doing an amazing job at doing something similar with compiler tracing and, like, code diagnostics. There has been a ton of work really in the last, I don't know, year to, like, improve Elixir's lexer and parser and, like, all the code compilers.

Francesco Cesarini: So, you're putting hooks into the code.

Andrea Leopardi: Everywhere. So, essentially, like, the language servers become very precise in doing stuff. Like, now Elixir's able to, for example, partially tokenize strings that are...like, all Elixir codes that have syntax errors so that, like, we can point you to the syntax error and where it is, you know, like all these tools, and that's really...like, it makes me think of that because, like, that's, like, really precise tracing at the compilation level, right? And like, I'm not involved in any of it, sadly, but I'll benefit from it, you know.

Francesco Cesarini: I imagine when they went in and added line numbers in error messages, you know, which...

Andrea Leopardi: Elixir, especially with this tooling stuff, can have its compiler written mostly in Elixir, right? And a little bit of Erlang, like the compiler for Elixir itself, is very high level but very easy to...I've always, like, felt it's, like, quite easy to contribute to and it's quite easy to, like, do bigger stuff compared to language, like, where you have a lower level compiler that, like, has to take care of, like, optimizations and, like, do multiple more passes. Like, Elixir's compilation passes are less, for example, so you have easier opportunities for doing this sort of stuff like tracing compilation, compilation hooks, and all this stuff for code diagnostics, so it's been very interesting.

Francesco Cesarini: Why don't you tell us, you know, Elixir is making its way into a lot of new verticals. I think one of them is machine learning. Can you share a little bit about what's happening in that space?

Andrea Leopardi: I am not involved in that space at all, so always a pinch of salt, but essentially we have port...like, the Elixir...Sean Moriarty, in particular, has been leading all of this, but the Elixir community has been writing the SLs for Elixir that compile to this, like, GPU frameworks. You know, like, I don't know any of them. Like, I'll embarrass myself, but essentially you can do tons of operations in Elixir now. And from that, sort of, was born a whole ecosystem of things that you can do now in Elixir-related machine learning. The really interesting thing to me is that this is not...it's not necessarily better than Python as far as I understand, for example, because they both compile to, like, GPU instructions. Eventually, they use like this Google compilers for GPU instructions and stuff. So it's not necessarily, like, better performance, but the thing that's been interesting is this seems to be one of the first languages to challenge Python's dominance over this space, right?

Francesco Cesarini: Well, it focuses on user experience and...

Andrea Leopardi: Exactly, yeah. It looks nice.

Francesco Cesarini: ...adoptability as they do for machine learning what they've done for web development in Phoenix, where they've reduced the barrier to entry where you need to know a little, just, you know, a subset of Elixir to be productive using Phoenix and not understand, yeah, anything of what's happening under the hood, right? And, in no time, you'll deploy the website. If they do the same for machine learning, I think it's gonna open up the doors to a whole new set of users.

Andrea Leopardi: There's a lot of focus... There's a lot of focus on user experience again, like we saw with Phoenix, like we saw with LiveView, like all these tools, like we saw with...

Francesco Cesarini: On Nerves as well.

Andrea Leopardi: Nerves, exactly.

Francesco Cesarini: So Nerves is the embedded framework used

Andrea Leopardi: For embedded devices and, like, the same thing, like, very, very usable, very user-friendly, and they're doing the same thing with machine learning. And to me, it's very interesting because, like, you can see that the ergonomics make a big difference, right? Like having...even if, like, maybe performance's not gonna be better than Python because they're gonna compile, like, very similar...I don't know anything. I could be saying, like, total lies here, but like, I think that's how it works. And, you know, it turns out that ergonomics developer experience and usability matter to developers, right? I think of this, like, it's interesting this is one of the...from my very limited experience, it seems to be one of the very few languages that are challenging Python's because Python's like, it's the thing in the machine learning sphere, right? Like, you write Python. And instead, like, Elixir is coming up with like, oh, hey, you know, you can write this in Elixir too. And it's very, very interesting to see what that will bring in the future. I have no clue. I'm not involved sadly because, I don't know, I don't do anything with machine learning but...

Francesco Cesarini: What really kind of excites me is the ability to run everything in the same memory space. So, you know, if you look at Phoenix and you decide to use Python as a backend database, you know, you'd have the web server, you'd have your business logic, and your database all in the same memory space, which makes it extremely fast. I mean, it's faster to create dynamic pages and to read static pages cached in Redis. So, if you can do the same if the same happens with machine learning, then it's gonna be fantastic.

Andrea Leopardi: We use a lot of ETS at my job for caching and for, like, responding very quickly to requests and, you know, like, we have an endpoint for beginning

Francesco Cesarini: ETS tables are the Redis on the BEAM. The Redis on the BEAM.

Andrea Leopardi: Like half a milliseconds and it just queues requests and then we flush them out to...

Francesco Cesarini: You distributed ETS tables?

Andrea Leopardi: There's a lot of tools, very powerful tools.

**Future Vision and Interoperability** 
Francesco Cesarini: What's your view over kind of the future of Erlang, Elixir, Gleam, and the ecosystem as a whole?

Andrea Leopardi: Good question. I have no clue. My dream is just that, like, they become so interoperable that I can write my libraries in...some libraries in Elixir, some libraries in Gleam, some libraries in Erlang, and they just like all work together. That's my dream because, like, I think people are gonna have preferences. Erlang is a simpler language, for example, and I think a bunch of...in that it has fewer features, strictly speaking. Like, there's less feature surface, so it's, like, simpler. Gleam has completely different directories, like, you know, the static type stuff, and I would like to be able to choose which one I want to use for different use cases. Like, I want to write, I don't know, like a compression algorithm, like maybe Gleam is nice because it lets me like do it, like, have static types and check all this stuff. I need to write something related to, I don't know, like mingling with protocol stuff. It's nice to have, like, static types.

I want to write something that, like, is gonna be complex and I want to write in Erlang to keep the feature set simple. I wanna write something that, you know, like, uses protocols. I want to be able to use Elixir. So my dream would be to have...and also to have a single communal. Like, I wrote a bunch of drivers in Elixir, for example, Cassandra already. So, like all this stuff, I would love it if people could just use them, you know? Because, like, the Cassandra

Francesco Cesarini: Absolutely. Absolutely. I think interoperability is key.

Andrea Leopardi: I hope, that's my hope.

Francesco Cesarini: And I completely agree with you. From my point of view, I'm seeing, kind of, Erlang, Elixir, and other BEAM languages making their way to edge networks, and even out on the IOT devices. And that's for example, where I think, you know, Gleam would take off for the simple reason it's statically typed and it makes the language much more secure.

Andrea Leopardi: You can take off a bunch of stuff from round time because

**Outro**

Francesco Cesarini: Exactly. You're reducing the surface of attack and the external surface of attack. But the great thing is that all of these languages are interoperable and dependent on each other. I'm hoping they'll thrive and continue growing together. Thanks, Andrea.

Andrea Leopardi: Thank you.