<!--yml
category: 未分类
date: 2024-05-27 14:57:03
-->

# We Have to Start Over: From Atom to Zed

> 来源：[https://zed.dev/blog/we-have-to-start-over](https://zed.dev/blog/we-have-to-start-over)

# We Have to Start Over: From Atom to Zed

*After the [last conversation](/blog/why-the-big-rewrite) with Nathan, Max, and Antonio—Zed's three co-founders—I still had quite a few questions to ask: why did you make the technological choices you did? How important is Rust to Zed? Did you consciously set out to own the technological stack the way you do now? How do you decide what to polish and build-once-and-for-all-time and what to ship faster?*

*Lucky for me, they sat down with me again and answered my questions again.*

*What follows is an editorialized transcript of an hour long conversation we had. I tried to preserve intent and meaning as much as possible, while getting rid of the uhms, the likes, the you-knows, and the pauses and course-corrections that make up an in-depth conversation.*

*(You can [watch the full conversation on our YouTube channel](https://www.youtube.com/watch?v=w894KLbapLw).)*

**Thorsten: The three of you have been working together for, what? Has it been 10 years?**

**Nathan**: Ungefähr. [German for: "Something like that."]

**Max**: Yeah, something like that.

**Antonio**: I think 10 years sounds about right. 2014, yeah.

**Thorsten: [Atom](https://en.wikipedia.org/wiki/Atom_(text_editor)) — that was 10 years ago. You worked together on Atom and then said "we're building Zed." It's pretty clear that you have a vision for what you want to build. And you also made some really distinct technological choices. You use Rust, it's GPU-accelerated, CRDTs. I'm wondering: how much are these technological choices are tied-in with the vision for Zed? How big of a role does the technology play?**

**Nathan**: I mean, from my perspective, the vision for Zed is just a more refined and fleshed out version of the original vision for Atom, which just fell short due to the technical choices we made in my mind and the level of technical maturity that I think all of us had when we started Atom.

But the goal I've always had is a lightweight editor that is minimal that I love using that feels like a text editor, but has the power of an IDE when needed, without all of the slowness in the experience and kind of heaviness in the UI, but still powerful. That was very early on what I wanted. And for it to be extensible.

You know, I remember using Emacs and thinking it was so cool that you could extend it, but you're sort of operating at the character level. So that was part of the original vision too and now Max has achieved that with [Tree-sitter](https://tree-sitter.github.io/tree-sitter/). You know, basically scriptability. We're not scriptable yet, but we'll get there. But when we are, we'll have access to a richer representation of the text than just looping over characters or whatever I was doing in Emacs. I mean, they probably have that now as well, right? Because Max wrote tree-sitter.

**Max**: They do, I hear.

**Nathan**: So that's cool. But anyway, that was all part of the vision. And it just didn't pan out with Atom. I mean, when Chris hired me to work on Atom, I almost didn't join [meaning: join GitHub to work on Atom] because I was afraid of using web tech and that it wouldn't be able to get there.

But to be honest, I don't know that whatever I had at my disposal at the time would have gotten us any better because Rust didn't exist back then. So it would have been C, C++. I don't know, it would have been really hard to do something native at the time we created Atom. And I don't think my skills were in place on just fundamental algorithms, et cetera, that are critical to text editing. And C++ would have really slowed me down in my learning there. So I think it happened how it needed to happen.

We got to a certain point with Atom. It was 2017 when we'd shipped [Teletype](https://github.com/atom/teletype) and it felt like, okay, it's no longer our own ignorance holding us back, it really is like the platform holding us back at this point. That's really how it was starting to feel. Just the fact that in JavaScript, an array in JavaScript is... You think you have an array of objects, but you really have an array of pointers to objects. So every single time you're walking over that, you're chasing it down. You have no control over memory. There are garbage collector pauses that are just happening that you have no control over. You just look at the profile of the fricking thing.

**Thorsten: When you were working on Atom, was there a specific point at which you said "oh, I wish we would have built it with X"? Or was it an accumulation of paper cuts? Antonio was nodding right now.**

**Antonio**: Speaking of 10 years of working on this thing: I remember one of my first projects in Atom — I don't know if you remember this, Nathan — was speeding up line layout. Basically, we were seeing line layout being very slow. And I remember trying rendering lines in an iframe. And I remember using Canvas, measured text, and all these APIs that were... I don't know, the Canvas API wasn't quite the same as the browser API, so you couldn't really measure text correctly. The iframe stuff had some other weird problem.

My experience in Atom always felt like bending over backwards to try to achieve something that in principle should have been simple. Lay out some lines and read the position of the cursor at this spot in between these two characters. That seems fundamentally doable and yet it always felt like the tools were not at our disposal. They were very far away from what we wanted to do.

**Nathan**: It was a nightmare. I mean, the ironic thing is that we created Electron to create Atom, but I can't imagine a worse application for Electron than a code editor, I don't know. For something simpler, it's probably fine, the memory footprint sucks, but it's fine. But for a code editor you just don't have the level of control I think you need to do these things in a straightforward way at the very least. It's always some... backflip.

So at some point in 2017, I remember sitting there, writing in Atom in my journal — that was the evening when I thought: we have to start over. We have to start over, this isn't working and we're never going to get this where we want it to go. VS Code has done an admirable job of getting it as far as it's gonna go. There'll be incremental improvements in the tech, I'm sure, but I just wanted more.

So at that point it was: okay, what should we do? And I'd been watching Rust, I'd seen some of [Raph Levien](https://raphlinus.github.io/)'s writing about Rust. And at the time this seemed like the only viable path to kind of overcome some of these obstacles. And it started as: what if we just write the core of this thing in Rust and we keep Electron as the presentation layer?

So it was just inch-by-inch that we came to the technological decisions we chose. Even after building the UI framework, giving up on Electron, we were using [Pathfinder](https://github.com/servo/pathfinder), which is like this really cool project that could do arbitrary presentation of SVGs basically, but it was too slow. So then I thought: okay, what if we do our own shaders? And then went and learned about signed distance fields.

It was kind of funny. It was me not wanting to solve some more fundamental problem, but just being forced to do that by the unavailability of any other choice.

I shouldn't say it was only me, it wasn't only me. But this was from my perspective...

**Thorsten: It sounds like the goal was always to be as fast as possible, as lightweight as possible. You tried to get there, but you couldn't with the technology you had. But then Rust came along. And you didn't start saying "we need a GPU-accelerated editor", but you started by saying you want the fastest possible editor and then GPU acceleration was one way to do that. Is that right?**

**Nathan**: Yeah, it was: we have this hardware in the computer and rather than negotiating what DOM nodes are in the DOM at a given moment, or all this nonsense, we could just literally be like, what color should this pixel be? Great. Okay, if we can program that hardware to decide what color every pixel on the screen should be in parallel, or as parallel as possible — we should probably use that if we want to be fast.

But you know, we came there kind of grudgingly because I didn't know how to do any of it.

**Thorsten: Now, this is kind of a loaded question, but I've been on the side hacking with Rust for the last three years and my feelings about it are complicated. But two weeks someone on Hacker News said that find-all-matches in Zed takes a second but in Sublime Text it's really fast, closer to 200ms. So Antonio and I paired on that and he wrote code to optimize for this case of searching all the occurrences in a buffer. But the optimized code wasn't "optimized" code: it didn't use any dirty tricks, no SIMD, or something like that. It was high-level code, with the optimization being that it's assumption was different: instead of doing things in a loop, one by one, its assumption was to find all results. We made a release build of that and it went from 1s down to 4ms.**

**Antonio**: Hahaha

**Thorsten: And I sat there, looking at the 4ms, and... well, I thought: this is going to be a nice lunch break after this. But 4ms? With high-level code like that, that just called other internal APIs? Wow. So what I'm trying to ask is: Rust has these zero-cost abstractions — so you can use high levels of abstraction to build a text editor, but it still gives you this kind of performance. Do you think this is a specific thing about Rust, or do you think if you had just been a better C++ or C programmer, you could've done it in another language?**

**Antonio**: You probably could have done it with C and C++. I don't know. I mean, tree-sitter is written in C, right, Max? So it is possible to write a complex library piece of software in C. Although I have to say anytime I look at the C code in tree sitter, I scream because it's just too much for me. I don't know.

Personally, yes, Rust is in a sweet spot. If it wasn't for the compile time being that slow — that's a thing that I really don't like about the language, but maybe it's our project that's too big, I don't know.

But yes it is pretty cool that you can build on top of these abstractions and just rely on them. I mean, I don't know about zero cost — every abstraction has a cost, I guess.

But the thing with this project in general... With Atom it always felt like we didn't know where to look for performance. With this, with Zed, it's more: we could do this and we could do that and we could improve this. Just last week, Nathan and I were discussing how to improve the subtraction of the SumTree to perform batched insertion a lot faster.

**Nathan**: And then you implemented it, right?

**Antonio**: Yeah. But it's not shipped yet.

**Nathan**: Great.

**Max**: I'll say we did do a lot of C++ on Atom — we did a lot. We tried. And it worked, but it was just that there was a very meaningful distinction — obviously — where the boundary was between the JavaScript application code and the C++ library code. And people would talk about performance and say "just do this on a background thread, just don't do it on the main thread" and we'd say "okay, we can do that" but that means this whole subsystem that is involved here needs to be dropped into C++ in order to share memory. Then build JavaScript APIs around that and figure out how it's going to still look like idiomatic JavaScript code and preserve all these properties that it had from when it was written in JavaScript.

And only then could we actually move this one task to the background thread.

It was such a difference between writing JavaScript code — that was scriptable and pluggable and overridable — and the C++ that had the core capability to have shared memory and multi-threading.

**Nathan**: And Rust is designed to be multi-threaded. I remember when I tried to learn Rust and I wanted to implement this [splay tree](https://en.wikipedia.org/wiki/Splay_tree) because the splay tree was a structure that we used a lot in Atom. Well, for a time — it had its era at least. I mean, it was actually pretty good for our needs, but it had parent pointers, it was very much a mutable structure.

And I tried to build that in Rust and the language fought me. And I thought: can you even build anything real in this language? I had serious doubts, actually.

So I gave up for a while, then I tried again. And this time around I built a copy-on-write B-tree. And when I built it that way, it used `Arc`s and that meant it was inherently multi-threading friendly.

And when I followed the dictates of the language and the borrower checker and did it the way Rust wanted me to do it, it was: oh, cool.

Now we have this way of representing [ropes](https://en.wikipedia.org/wiki/Rope_(data_structure)) — which is the fundamental text storage structure in Zed — in a way where we can spawn a background thread, and it's O(1), it's just bumping an `Arc` to take that to a background thread and look at a snapshot, et cetera.

It's not just about being native. I also think Rust brings to the table innovations. The language is designed to be used the way we're using it on multiple threads and at a low level.

And frankly, I was just a little too much of a script kiddie I think to do well in C++. It always just annoyed me: these freaking files, jumping over here, all these arcane rules. And could a C++ master do what we did in Rust? Probably, but like I wasn't gonna become that person.

**Thorsten: So Antonio, you just mentioned that with JavaScript, you didn't know where to look for performance. Max, you said, when you were using C++ or JavaScript, you felt these boundaries when you want to make something async. And now we heard about Rust — you can suddenly do stuff async on a background thread, you have less restrictions, you can move freely around.**

**That reminds me of something you can actually see in the Zed code base: you own the whole stack. From tree-sitter doing the parsing to GPUI, the GPU-accelerated UI framework — there aren't a lot of third-party dependencies in the code base. Some libraries, but not big building blocks.**

**How important is that for you? We own the full stack, top to bottom, we understand it top to bottom. Is that a conscious choice or did this happen by accident because Max built tree-setter and then you did this and now look at us, we rebuilt the whole thing.**

**Max**: I don't know. I think it has trade-offs, but I so far it's been pretty nice to be able to just... decide how we want things to work. Like right now: we want to have language extensions that use WASM. Tree-sitter didn't have that but we added that to it.

There's a million things like that. We don't want to be beholden to some UI framework that may not render text exactly the way we want, because we're a text editor and that matters a lot. But we can now go change that.

It doesn't feel like anything is sort off limits.

**Nathan**: One thing I'll say: this was very informed by an experience earlier in my career with jQuery. jQuery was the hot thing and I learned jQuery. I remember [Yehuda](https://yehudakatz.com) coming to [Pivotal](https://en.wikipedia.org/wiki/Pivotal_Labs) and presenting on jQuery and I thought this is so cool. I was blown away by it. So early on, all the Atom code, believe it or not, was jQuery.

And the funny thing was though, having learned jQuery, because everybody told me, "oh, the reason you use jQuery is it abstracts over all the differences in the browser APIs", et cetera. I never really questioned that. But then I remember the day that I just sat down and read the freaking DOM APIs. And I thought: you know what, this is actually fine. Maybe there were some missing features or something — and I don't want to shit on jQuery, I think it has its role — but what I came away with was that if I don't have an abstraction that's nailing almost 100% of my needs then I might not want to have that abstraction and go to the level below, understand the level below it fully, and do what I need to do.

That was kind of what happened with GPUI. There were some UI frameworks in flight when we started on a GPUI, in 2019, but none of them did what I knew we would need to do. And I didn't understand them.

But I knew I could come to understand quite easily the fundamental primitives that we're going to be relying on — the language, the graphics framework. I knew we could learn those things and I knew we've written a lot of code. If I can build a system that I can understand and learn from, then I know that I can do what we need to do if it's fundamentally possible on the underlying system. And so really it was — at least for GPUI — a survival strategy: I need to understand this and the best way to understand it is to build it.

**Thorsten: What are the downsides of that? Max, you said there's trade-offs.**

**Nathan**: Takes forever. It's slow.

**Antonio & Max**: [laughing]

**Antonio**: It's also tricky to onboard people. You're not using the X framework out there that everybody knows, you have to teach this code base from scratch — you know, 300,000 lines of code. That's a downside.

But the cool thing is that while this is a downside, at the same time there's somebody else who has written that code and can explain it to the new person.

It might be slower, but again, you're retaining control.

**Nathan**: I think it accumulates. The advantages accumulate, and the downsides depreciate. Someone just built [another app on GPUI](https://github.com/MatthiasGrandl/loungy), so now we have another stakeholder. The cost of having owned it, we're going to gradually write off over time, and the costs and the upsides of owning it are going to start to kick in.

**Thorsten: Sometimes people say: I only build it once and then I never have to touch it again. And the opposite of that might be: you can't predict the future, [worse is better](https://en.wikipedia.org/wiki/Worse_is_better), and so on.**

**And what you just said, that it's slower to build it all yourself, there's a ring of "build it once and do it right for our use case" to it: only the perfect abstraction for what we want to do. At the same time, there's this sense of urgency that you all have. I mean, I joined four weeks ago and I do feel like we're moving fast and we've got so much to do.**

**How do you balance that? How do you have this huge vision for what you want to build, and balance that with saying "I'm gonna write shaders, I'm gonna perfect how we render drop shadows or whatever"?**

**Nathan**: Build what you need, only what you need, no more, no less, and build that as well as you can within reason. And then when it turns out to not be quite what you needed, be willing to revisit it after you've learned. But I think if you're laying down every brush stroke with intentionality and care, and you're not wasting time speculating about what you might need, then... for me, that's always worked out. Sometimes it takes a little while though.

**Antonio**: I would add on top of that: it's a gradient. It's not like everything needs to be built perfectly. Or at least that's how I feel about much of the code we write. If we're writing stuff in GPUI, well, the whole app depends on the GPUI, that better be perfect. Or the sum tree. It's this data structure that's used everywhere in the code base. That one we really wanna nail, because it has to be fast.

It has to work perfectly so that we can build on top of it, right? And that is reflected, also in the testing that we do on those things. The SumTree is randomized-tested because we want to make sure that all those edge cases work perfectly.

Now, as you move towards the edge— that performance improvement that you alluded to, Thorsten, we didn't spend three hours gold plating it, right? It was like: whatever gets the job done, it's pretty much at the edge. I mean, we should feel good about the code, we should always strive to write the code as best as we can, but we don't need to gold plate it.

It's a gradient. The more core something is, the more it deserves thought and quality.

**Nathan**: And my favorite code to write is the code I've earned myself the right to write. Whoa, that's a lot of homophones.

With GPUI I had so much fun writing it that I almost felt like guilty about it. But I have earned the right to write this code because I wrote the first version, I lived with it, I pushed it forward, I made the compromises when I needed to make them to make things right.

But now is the moment where we can make this better. And it makes sense to make this better. And I'm informed. I think a lot of times when people talk about like rewriting— if you're rewriting something someone else wrote, be doubly suspicious of yourself.

But if you wrote it and you've lived with it and you've put in the work... That's also something to be said: don't let perfectionism get in the way of learning.

**Thorsten: If I were to take the three of you and... I don't know what to throw at you that you couldn't build, but say you'd have to build, I don't know what — a PID controller for an airplane. Airplane software, there you go. Something like that.**

**Nathan**: Nuclear reactor control subsystem.

**Thorsten: You don't know the domain and you don't know yet how you're going to build it and you don't yet know which parts you're going to need. Is that where you would say you'd need a different approach? Unlike in the editor, where you have a strong vision for what you want, you built a few before, so now you know which parts count. You know beforehand, GPUI is going to be important. So let's take our time with it and gold-plate it as Antonio said.**

**Max**: I mean, I don't know if a nuclear reactor is a good example, but I do feel like if it was our first code editor, ... We *did* "worse is better". It wasn't intentionally bad, but we took the kind of quicker, dirtier approach once and then kind of identified the things that were real pain points to build on top of, in their sort of worse-is-better form.

But I do think, if for some reason Antonio and me and Nathan had to build a...

**Nathan**: Choose something less mission critical maybe?

**Max**: ... a Shopify clone or something. We would probably have a different mindset about it. We wouldn't know which pieces needed to be really highly honed.

**Nathan**: But if I *were* building a nuclear reactor control system, I would use a Byzantine fault-tolerant consensus algorithm and pit three teams against one another to compromise each other's security and then make them come to consensus on all of it. But I don't know how to do that.

**Antonio**: There's another example that's less mission critical than a nuclear reactor, but where we didn't know the data structure and we kind of took the time to learn it. It wasn't always that the structure powering the buffers in Atom and in Zed today was a CRDT. There was a research period where Nathan and I read... I forget how many papers. A lot of them. And the approach we're using right now with the CRDT — we still rewrote it two or three times — but the approach is more or less the same.

So I think there's a part of it that comes with experience. I feel like you tend to develop a sense of what you need to spend time on and what is more frivolous, less important.

Now, that said, we did rewrite the CRDT two or three times, but the research part, was important. I don't know.

**Nathan**: The funny thing is in the Atom that shipped, the buffer was an array of lines, a JavaScript array of strings. And in Zed, it's a multi-thread-friendly snapshot-able copy-on-write B-tree that indexes everything you can imagine. Or everything we've needed. So we did worse is better.

But starting over, would it be an array of lines again? Probably not because look at the look at the time bounds on that. And put a little more thought into it, but that — again — I learned that by like doing worse-is-better and then having it be really slow or problematic in edge cases that ended up mattering.

**Thorsten: So what are the most gold-plated parts of Zed?**

**Nathan**: GPUI is pretty gold-plated, I think, because we just rewrote the whole fricking thing.

**Antonio**: Good question.

**Max**: I think what's in the [`editor` crate](https://github.com/zed-industries/zed/tree/main/crates/editor), where there's sort of the stack of different transforms that convert the raw text of the buffer into the lines that you see on screen, that expand the tabs and do the soft-wrapping and insert the block decorations and handle folds and stuff. All those layers have this uniform testing strategy where it's randomized-tested with property testing. So I think they're pretty gold-plated.

The multi-buffer too, where we sort of weave together the different excerpts of different buffers into one.

**Nathan**: I would call them, um... I just wanted to suggest like an alternative substance for that part of the code base. I would say it was kind of plated and coated in blood.

**Antonio**: Ha! Blood plated.

**Thorsten: You mean the editor and the multi-buffer?**

**Antonio**: Yeah.

**Nathan**: Yeah. It's randomized tests where we've spent literally the entire day in 2021, many days in a row, just debugging failures in these randomized tests that would find some weird edge case of this ornate— I mean, I wouldn't say it's ornate, but it's complicated — stacking of different layers of transformation required to present things on screen. And so it's just elbow grease, right? Find the edge cases and then figure out why they're happening by reducing the log, which for a long time we just did manually.

**Thorsten: How did it feel when one of these bugs popped up? Did you have moments of panic when property testing threw a bug in our face and you thought "maybe this whole thing doesn't work?" Or was it rather "well, it's just another thing to polish down and if not, we rewrite this"?**

**Antonio**: Never panic, that's the rule of randomized testing, never panic. I have a lot of faith in our capability as engineers, I really do, and maybe it might be that we have to rewrite the whole thing and the randomized test is telling us that, but it's fine, we just learned something, back to the drawing board and redo it.

**Nathan**: What was scary was: how long is this going to take? I think Lee, our seed investor, was also asking us that at certain times. But he stuck with us and was patient because it took a while.

But that piece was written in Rust. If we f this up, the program is panicking. Goodbye, poof. It's not just like a stack trace gets thrown in the corner of the editor or something, no, it's done.

So we knew how hard it was to get those layers right. And we knew that there was no other choice, but to get them right. But yeah, I remember, Antonio, remember working on soft wraps and that problem we came up against where we realized the primitive we needed was this ability to represent a patch and then to be able to compose these patches together — that was one moment where I was sweating a bit, thinking "are we gonna freaking figure this out?" and then Antonio figured it out.

**Antonio**: Yeah. But powering it all — you know, in terms of gold-plating — is again the sum tree. And even with that, there are some ideas on how to make it better, if we were to rewrite it.

**Nathan**: We were talking the other night about how dope it would be. And Antonio, you applied some of the ideas we talked about: being able to construct all these layers in a more streaming friendly way. And you did one optimization, which is gonna land on preview next week.

**Antonio**: Yeah, I need to open the PR for it still...

**Nathan**: To enable more streaming inputs so people don't get zero feedback when they open a big file, but start actually loading things in, more efficiently — does that necessitate a rewrite? Maybe. Maybe not. I don't know.

**Thorsten: Quite interesting how often this idea of rewriting or doing it again comes up. We talked about this [the last time](/blog/why-the-big-rewrite). Learning continuously. It's not: learning and *then* fixing something and patching something. It's more: we learned something, so now let's redo it with that learning in mind, vs. just putting a band aid on. It came up multiple times and you see it in the product.**

**When we talk, Antonio, you say things like: before we did it like *this* and now we do it like that. Just yesterday, we basically rewrote the part that deals with macOS' IME system, because Antonio said: we can't leave it like this, we don't want another person to fall into this rabbit hole, let's put a bridge on top of it.**

**Nathan**: Nice. That's good to hear. I've heard about that.

**Thorsten: I don't know if it makes sense, but my last question is this... With a lot of software, say, SaaS Enterprise Whatever, or the Shopify clone, I think most users do not care what technology is used, as long as it works for them. And I wonder: do you think this is different with developer tools or editors? Does the technology that's used shine through more or do the users care more about it?**

**Max**: I do think it affects the type of contributions we can get. A lot of our users, so many of them are people who would be prepared to contribute something to the code base to fulfill their own needs.

I think it's important that it's easy to contribute to Zed. If we written it all in C++, I think that there would be a lot of people who would like wanna change something about Zed, but would not be as prepared to make the change themselves.

Whereas just from the contributions that we've gotten so far since going open source a few weeks ago, it's a lot. I think people like that it's written in Rust. It's approachable. People can build the project easily. They don't have to go learn how to use CMake or whatever to build the project or Gyp. They can just use cargo.

But also the fact that the compiler has this strictness to it, allows us, as the receiving end of those contributions, to often merge with confidence. I think it's really helpful.

**Nathan**: Rust is an absolutely beautiful tool. It's not perfect, but I love it. But does it matter to the people? I mean, I think people want a fast editor at the end of the day. We could write it in, what is it, brainfuck? They wouldn't care. But the contribution angle is super valid.

**Antonio**: But I still would like to talk about the performance, I just think we are forced to do things a certain way because of performance. Why do we have this GPU accelerated UI framework? It's because the performance needs to be at a certain level. We want our frames to be below three milliseconds. We could rasterize everything on the CPU and we could have used something that did that, but, no.

To some extent, we're positioning ourselves to be a performance editor, because we want a performance editor. I want a performance editor. And so, the choice is almost... we have no choice almost.

**Nathan**: But there's Zig now and I don't know a lot about Zig yet, I haven't had time, honestly, to learn about it, but people I respect are excited about it. It seems like it shares some of the same goals in terms of the output as Rust. I'm unclear what it's sacrificing in terms of safety, or how they handle those things. There may be pragmatic workarounds that are sort of not as strict as Rust, but in practice work, et cetera.

So I'm intrigued by that. I'm intrigued, but then there's a lot to be said for like monolingualism, if that makes sense. The server's in Rust, the frontend's in Rust, but if I could get 99% of the benefits of Rust with a 10th of the compile time or something...

**Thorsten: Well, I can tell you about Zig that a person on Discord was saying they're writing an editor in Zig, but the perfect name for a text editor in Zig is already taken: Zed.**

* * *