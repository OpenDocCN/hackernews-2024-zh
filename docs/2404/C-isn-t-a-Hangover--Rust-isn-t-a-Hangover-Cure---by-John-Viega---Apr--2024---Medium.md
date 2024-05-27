<!--yml
category: 未分类
date: 2024-05-27 13:28:33
-->

# C isn’t a Hangover; Rust isn’t a Hangover Cure | by John Viega | Apr, 2024 | Medium

> 来源：[https://medium.com/@john_25313/c-isnt-a-hangover-rust-isn-t-a-hangover-cure-580c9b35b5ce](https://medium.com/@john_25313/c-isnt-a-hangover-rust-isn-t-a-hangover-cure-580c9b35b5ce)

# C isn’t a Hangover; Rust isn’t a Hangover Cure

A few weeks ago, I got a bit miffed reading yet another article that was too dismissive about memory safety, basically being mostly dismissive about the need for change. The following weekend, I started seeing flippant responses from security luminaries, saying essentially that you’re irresponsible and dangerous unless you drop C and C++ faster than I dropped my 8 am classes my first year in college.

I’m going to start with the tl;dr, but then, I’m going to drill down into the issue in detail. I’m going to start by trying to present all sides at a level where most people in the software industry would be able to understand.

That means, I am definitely going to (intentionally) oversimplify some of the more technical bits. This article is already way too long, and the thesis isn’t about the tech per se, it’s that the economics are incredibly complicated on all sides, and we need to find a way to accept other people making decisions we don’t like, and still help the world to a better place.

There is a lot to consider, so I hope you’ll stick with me, and think through the nuance.

# The tl;dr version

1.  The security problem *is indeed* more significant than many people seem to believe, and many people would absolutely be much better served abandoning C/C++ right away for new projects, and not just from a security perspective. But!
2.  It’s far more costly and risky to get rid of all the C our applications are already using than many people might think; creating replacements to some key pieces of software will take a decade or more to be effective as a replacement, with no clear overall benefit.
3.  There is a lot of hidden complexity just when we consider security that complicates the equation to the point where saying “Rust is safer than C” might be true, but actually isn’t a total slam dunk at all.
4.  The economics of something as simple as choice of programming language are actually very intricate. Not only is security not the only non-functional consideration, no matter what you do, there will always be memory unsafe code somewhere (as long as the underlying architectures themselves are unsafe), and there would be many negative consequences to trying to get rid of C quickly.
5.  Systems languages are overused; C vs Rust is a false choice, because compiled languages like Go are often a much better all-around answer economically. Go in particular has decent enough performance that is sufficient for the vast majority of use cases, will be safe, and has good access to low-level systems APIs.

## Some Security People are Already Bristling

I know from experience, simply suggesting that security might not be an absolute higher priority than anything else will feel like a personal attack to some people in the field, as if I am arguing against the security industry’s very existence.

On the contrary, I think the security industry would be thriving even more than it is if we were more well-rounded, and better understood tradeoffs outside of our domain. Some people might have to be dragged into caring about security, but non-security people are the majority, and many of them have a balanced view, where they care about security, but want to avoid throwing excessive time and money at it.

Once, long ago, I was watching a security person arguing with the business, and as the security person dug in, I kind of got it, and asked:

> If you think security is the primary concern, then why are you using a computer at all?

People are willing to accept risks every day. We know we can “catch” a virus any time we go out in public. We know we can get into an accident and die any time we step in the car.

But it’s well known that humans in general are horrible at modern risk assessment. We have a tendency to either overestimate or underestimate our risk levels *dramatically*.

Generally, the security industry probably assumes the average person greatly underestimates risks. And to some extent, that is true. When I was first doing code audits for security in ’98, it was definitely true that engineers were underestimating risks of memory issues. Back then, if you handed me C or C++ code that wasn’t written by Dan J. Bernstein (often refered to simple as djb), it was safe to assume there were going to be **exploitable** memory errors, period. I even saw them in the code of people I looked up to in the security field.

But the world is a different place now. Back then, most of the industry massively underestimated risks and were happy in a world with network connections that could be easily tampered with and code that could be easily exploited, because they either didn’t think about it at all, or else didn’t think it was going to end up being a big factor.

There was no patch Tuesday, nor was there widespread compartmentalization or other effective compensating controls.

The rest of the tech world was eventually willing to acknowledge they were wrong, due to the hard work of the security industry as a whole. As Greg Hoglund often said back in the day, the combination of connectivity, extensibility and complexity created a perfect storm. Nobody could deny it, and as a result, the security industry has had a huge influence, from hardware architectures to network protocols, to programming language design.

But it wasn’t as easy as it should have been, because we (as an industry) have often been unable to truly understand the perspective of people outside the industry.

If we can keep that in mind, the industry can improve its credibility, and make progress even faster. And we need that, because, as we’ll see, we not only have a lot left to accomplish, but there are many important changes that *simply cannot happen quickly*.

# How Big a Problem Is Memory Safety?

I’m 100% going to start off by acknowledging the problem. Taking the problem seriously early on has been good for my career — almost 25 years ago, I co-authored the first book on security for developers, before even basic web security issues like cross-site scripting (XSS) were a thing. Not too long after, I also co-authored the *Secure Programming Cookbook for C and C++*.

Those books did well and helped put me on the map, and I worked on those because I do take memory safety quite seriously. Back then, even more of the software out there was written in C/C++ than today, and it was almost universally more prone not just to having memory safety issues, but far more exploitable.

I’ve produced research tools and libraries to help mitigate the problem, and seen many other people do the same over the years. But! Generally, developers don’t care about security anywhere as much as security people do. Despite plenty of tools, C programmers have tended not to use them.

Memory safety is often considered the most egregious class of vulnerability, because when such problems are exploitable, they generally enable full execution. Often, problems can be exploited remotely, and sometimes without any authentication at all.

But, the reputation that memory safety problems currently have of being plentiful and trivial for sophisticated attackers to find and exploit is *wrong*.

It was certainly true at the turn of the millennium, but it isn’t any more. The impact of memory unsafe code from a security perspective is definitely still incredibly high, but not so high that, when you consider strong economic reasons why you might NOT move away from memory safe languages, it might be reasonable to conclude that you’re making a good decision, even risk adjusted.

## The Risks Have Changed

I’m going to focus on why memory safety in C and C++ code isn’t likely to be quite as risky as many people believe. Nothing in this section should be interpreted as a reason to choose these languages — we’ll explore the circumstances where the tradeoffs might make sense later in this article. Here, I will firmly acknowledge other languages are inherently safer; I’ll just be questioning “how much safer, especially if we have the right controls and are vigilant?”

There are a lot of ways in which the world has changed that directly impacts the risk level (in both directions), including:

1.  Both the hardware architectures and operating systems we all use have done a good job helping thwart exploitability without sacrificing much on performance, starting with StackGuard in 1998, all the way up to Intel’s recent work on control-flow enforcement and ARM’s memory tagging.
2.  C++ has focused on user interface around its standard libraries, trying to ensure that the average C++ user is not likely to ever use an API where memory safety is a concern. On the other hand, while C has evolved as a language, it has been much more conservative in this regard.
3.  The full disclosure movement happened, and vulnerability research became a career field, leading to the most common C components getting a fair deal of scrutiny, and helping to dramatically raise awareness of these kinds of issues with C programmers.
4.  Academia stopped teaching C++, moving to Java and then to Python.
5.  There’s been a big spike in newer languages that are proper systems languages, but with attention paid to memory safety, most notably Rust, but also Zig, Nim and several others.
6.  The move to the cloud and other niceties of the modern tech stack are great abstractions, but they do tend to increase attack surface. On the other hand, they promote compartmentalization that can limit impact, and oddly enough, since we don’t put as much reversible binary code into people’s hands anymore, proprietary code tends to benefit from what is essentially *security by obscurity*, as much as it pains me to admit that (though the fault tolerant design does often help give attackers the ability to automate as many “at bats” as they wish).

Some consequences of the above the above to consider:

1.  Many memory issues in C and C++ code are reported as if they are exploitable, even though there are definitely cases where issues may be impossible to exploit in practice. Way back when I was doing vulnerability research, if I saw a clear memory error, there was not only a good chance it was exploitable, it was going to be really easy for me to build a working exploit. I agree it’s better for us as an industry to just assume any memory issue found is exploitable, because often one can find other bugs to chain to make it so. However, it has become incredibly **hard** to build working exploits, and has gone from a skill that was easy to develop to one that is pretty rare. So true zero-day exploits tend to be strategically used, most often by governments.
2.  The economics of the vulnerability research world tends to skew risk perceptions. That’s because such errors are the most *valuable* in the economy, as I’ll discuss below. However, that means comparing CVEs across languages is suspect. But it also means that, valuable exploits are either sitting places where they’re not likely to get burned, or they go through a “responsible disclosure” cycle, which means that it makes sense if people conclude that a good patching program can mitigate the risk.
3.  Sometimes, people who do write in C have a very good reason for writing in C. These reasons will be important to understand. Far fewer newbies think they have to learn C to be effective programmers, but there are still places like embedded systems where C is often far and away a more practical choice.

As an example of why comparing CVE numbers by language is generally misleading — the Linux kernel recently officially gained the capacity to issue CVEs for its own code base. But in their view, any bug under the sun could have security implications they don’t understand, so every single bug found in the Linux kernel now gets its own CVE, even though mostly they’re not going to be exploitable memory problems.

## Understanding reduced exploitability

Here, understanding the evolution of memory errors can help understand exploitability. I’ll try to keep away from deeply technical explanations, because I think the people that most need to understand this are the ones that will (quite rightly) never work at a low enough level where they should even need to understand any of those details. This will, again, oversimplify things.

And in some sense, this maybe is gratuitous detail. On one hand, the fact that it’s gotten harder to find good exploits by a LOT doesn’t matter, because if you switch away from C, this whole class of issues become non-issues.

On the other hand, the fact that vulnerability researchers work tremendously harder than 20 years ago, to find far fewer bugs, is a signal that practical risks are lower than they used to be (especially in the face of good compensating controls). I would have assumed 20 years ago that any program written in C had easily exploitable holes somewhere, and probably would have been right.

Today, I’m prone to think there are good odds of issues in many C programs, but, if you do your design right and pay for the right people to review your code, the economic costs of finding the next bug are high enough that it’s no longer a slam dunk to me that any C code will result in you being owned.

Back when I was getting started in the field, it was often very simple. If you could find a local variable that was an array, there was a pretty good chance you could trick the program into writing outside the array. And, memory layout was so predictable, it was relatively easy to figure out how to exploit such conditions.

Specifically, local variables are generally kept on the *program stack*. When you enter a function, data gets pushed onto the stack, and when you exit, data gets popped from it (kind of). This is different from long-term memory allocations that survive function calls (heap storage).

For instance, it wasn’t uncommon to see code like:

```
void
open_tmp_file(char *filename)
{
    char full_path[PATH_MAX] = {0,};

    strcpy(full_path, base_path);
    strcat(full_path, filename);

    // Do something with the file.
}
```

To the uninitiated, the above code might seem innocent. It creates an array that is zero-initialized of the maximum size the OS supports for a path, whatever that is. Then, it copies into that array, some base directory name, then finally the file name gets appended to the end of that.

But even on today’s system, if the attacker can control either the input to this function, or to `base_path`, it is easy to crash this program.

Part of the reason is that C doesn’t keep track of how long things are. In C, `strcpy` copies byte-by-byte, until it encounters a byte that is valued 0 (the so-called `NULL`byte). Similarly, `strcat` works by scanning forward to the first null byte in `full_path,` and copies from `filename` until it finds a `NULL`. In no world do either of these functions check what they’re doing against the length of `fullpath`. So if you can pass in more than `PATH_MAX — len(base_path)`characters, you will write past the end of the buffer.

In the “bad old days”, this was a slam-dunk, and it would be trivial to exploit.

Traditionally, the program stack intermixes its own runtime data with the user’s data, which is what made the traditional *stack overflow* such low hanging fruit.

Every time a new function gets called, the stack would get the memory address for the place in code the program should return. So all you had to do once you found one of these conditions, was make sure to craft a exploit *payload* (the malicious data you send at whatever point) in such a way that it overwrites that return address, replacing it with a pointer back into your own payload… which also generally includes executable instructions that will do whatever. The executable part of the payload is usually called the shell code, though getting an interactive login (i.e., a shell) doesn’t need to be the goal. Either way, when the exploit succeeds, the attacker generally is able to run any code they desire from thereon out.

From a technical perspective, the most complicated thing back then was the shell code itself, as it generally requires at least assembly-level knowledge.

However, you didn’t have to write your own shell code, there have always been plenty of off-the-shelf payloads.

## Why not just always bounds-check?

Great question. One might think we could just have programming languages always generate code to check bounds for any accesses, and be done with it.

I’m personally sympathetic. I would point at the massive success of Python to show that, in most cases, performance is even less important than security is. So it’s no surprise that all languages (attempt) to guarantee no out-of-bounds writes, usually with dynamic code.

But, that checking code, if done everywhere, WILL absolutely have a notable impact on performance, and there are definitely domains where that matters.

For instance, if you’re a CDN and are trying to cost-effectively handle massive connection volumes, it’s easy to believe that the additional hardware costs could make the business not worth being in, unless the running software manages to skip bounds checks that are low risk.

And, individual applications written in Python generally are “fast enough” (even if many would disagree, as evidenced by multiple rewrites and a new experimental JIT compiler in the official Python), but would Python be fast enough if every bit of code possible that it runs upon was fully bounds checked?

Most of the software we use makes heavy use of low-level systems code written in C or C++, even if it’s indirect. Not only will the operating system be written in such a language, but also many of the libraries a typical programming language leverages in its runtime will be low-level.

Sure, you *could* “rewrite it in Rust”. But, even if we should do that (see below), it clearly would be a long, arduous journey to get there.

Note that Rust is able to approach C’s speed in part because the compiler can essentially ‘prove’ when it can skip most bounds checks at compile time.

But it’s not actually all that hard to design APIs on top of C that similarly can avoid memory errors if strictly used, while minimizing the generation of runtime code. For instance, in our example above, it’s not hard to provide a different C API for strings that always tracks the length, and fully checks. We could provide similar APIs around arrays and other data types. We could even provide such austerity to pointers.

That is effectively what C++ has done quite successfully, and why Bjarne Stroustrup seems to have taken great offense to the government telling people not to use his language due to memory safety issues. I don’t write modern C++, but it does seem pretty easy from the API docs to never come anywhere near a memory issue.

But, to the question asked above, the more scale we reach for, the more the little things make a big difference. If your whole OS and every library Python used were fully checked, it’s conceivable that Python would simply be too slow.

But in reality, it probably would just make a lot of software even less cost effective. And really, economics is usually the **most** important consideration when it comes to software, and those economics have to revolve heavily around the experience of the people using software (and, for example, generally only care about performance if things are slower than expected).

So if you’re writing low-level systems code, and you want it to get used, it’s got to be widely applicable, and it needs to be cost-effective at the scale of a CDN or any other big tech company.

There’s little reason why pretty much anyone should actually *want* their compiler to generate bound checks *when they are unnecessary*. Ideally, we’d want to be able to prove when it’s safe to skip, and make sure to produce machine code (whether manually or via compiler) whenever it is safe to skip checks.

Unfortunately, it’s pretty much impossible to get absolute assurance in most cases. We need to decide what level of risk of exploitation we even find acceptable.

And, as I will discuss, anyone who claims the risk should ever be absolute 0 is way too unrealistic… even ignoring economic arguments.

## Out-of-Bounds memory errors: a history of mitigations

The question of how much risk should we be willing to accept, leads us to the question, “how much risk are we currently accepting?”

Because if the answer is, “not much”, then we need to think about whether it’s worth the cost to add bounds checking.

The practical level of risk is hard to quantify precisely. The best way to do it really is to talk to people who have spent their careers doing vulnerability reach where they actually prove exploitation.

Anyone qualified who was there would certainly agree that C/C++ programs around the turn of the millennium were an absolute cornucopia of exploitability. I was there; it was easy if you had a basic low-level skill set.

But, there have been a quarter century of mitigations, and we live in an entirely different world. It’s so tough to prove exploitability, that we often just accept the possibility of exploit without any proof, any time we see a memory error we show can give us enough of a foothold where exploitation *might* be possible. And most of my friends still doing this kind of work would absolutely acknowledge that it is no longer “like shooting fish in a barrel”, it’s much, much harder.

It may not be easy to quantify how much the bar has been raised, but it’s getting pretty darn high.

To start understanding the risk today, we should go back to the beginning. In this case, let’s go back to the simple stack overflow discussed above.

In fact, while the code I showed is **definitely** a memory error, and I definitely could have exploited it a quarter century ago, determining whether I could actually build a practical exploit is so daunting a thought that I wouldn’t bother trying.

The code is still wrong because it doesn’t do bounds checking. And it can still be used to crash the program (which in some sense is a security issue). If you’d like to see, here’s me live-coding the example, and showing it still crashes (no up-charge for my slow typing and silly mistakes).

But, just because it’s a memory error doesn’t mean it’s easy to exploit, or even possible to exploit.

As crappy as the above code is, StackGuard addressed this problem fairly well, starting all the way back in 1998\. The basic idea is that when a program starts up, select a random number of at least 64 bits. Push it onto the stack every time you call the function.

Then, every time you return from a function, check to make sure the random canary is intact. If it isn’t, crash instead of returning.

Whereas one used to be able to figure out deterministically what to write to keep the program working up till the point it started executing your shell code, a naive exploit no longer works, unless the program somehow leaks its canary.

There certainly were situations where you could work around the above issue (especially when you can chain another bug), but StackGuard eliminated some issues completely, and certainly raised the work level of the vulnerability researcher.

The software exploitation community (both “bad guys” and the vulnerability research community) kept having to work harder and harder on bypass techniques, but have tended to at least find cases where they could work around some mitigations, if circumstances are right. For instance, the above mitigation doesn’t work if the memory is dynamically allocated, because that memory is kept separately, in the heap.

And, certainly, the program doesn’t keep function return addresses in the heap. However, many real programs, especially one using dynamic dispatch in C++, will keep pointers to functions in the heap, and use them to dynamically select functions to call.

One of the more effective and well-known defenses has been *Address Space Layout Randomization* (ALSR), which is implemented at the operating system level. Basically, every time your program starts up, the OS will randomize where data lives as much as it can. With ASLR, if there’s enough randomization injected, the attack would succeed with such low probability, it could require as many attempts to succeed as atoms in the universe.

In practice, it’s not *quite* that good — sometimes the distances between particular bits that are valuable to the attacker might not be randomizable, because there can be reasons why they need to be knowable (or, within small ranges). And, if the OS is too aggressive at randomizing, it could break programs. This is particularly true within operating system kernels — it’s hard to get a meaningful amount of randomization inside the kernel.

Still, this was an amazingly effective technique that suddenly made exploitation incredibly hard.

If you’ve never dug under the hood here, but are technical enough, you might ask two important questions:

1.  Shouldn’t the system be keeping the user’s program data far from its internal state?
2.  If the payload has to live in the heap or on the stack (other memory is generally non-writable), can’t we prevent those areas from running code??

For question 1, not only is a single stack per thread easier to implement, it is generally faster, because hardware generally will have direct support for a program stack. And ultimately, while processes have virtual address spaces that provide protection from other processes, within one process, any code in the process can address any memory cell in the process.

Though, it is still valuable to shuffle things around. For instance, in heap overruns, function pointers are juicy targets. Storing all function pointers in tables allocated statically or in a separate heap of memory definitely is better than the typical approach of sprinkling function pointers wherever they happen to go.

As for the second question, it’s 100% possible to prevent code execution out of the stack or the heap at the system level, and that is a mitigation totally worth having. However, some environments, including some entire programming languages, use one or the other to implement their own dynamic capabilities (like `lambda` functions that are closures). Still, for most programs, this mitigation is basically free, and raises the bar even more.

Today, when there’s a memory issue, attackers generally can’t expect to run code directly. But, imagine that you’re attacking a program written in Python, and you can somehow leverage a memory error in the underlying C implementation of Python to write to arbitrary places in memory.

Under the hood, Python implements a virtual machine. There are “instructions” that can live in the heap or stack that are examined by Python’s built-in code, and that code does different things depending on the instructions.

As it turns out, when we talk about memory being “non-executable” really only refers to what executes directly on the underlying system processor, not what happens in an application-level virtual machine.

So even if the program you’re attacking has its executable code segments non-writable, and all the data you can write is non-executable, you can still change data that controls what the executable code does.

As an attacker, if there is no virtual machine, you can kind of create one yourself, using a technique called *Return Oriented Programming* (aka, ROP). Basically, leveraging memory errors, you try to groom the program’s data so that it will jump around the program’s memory, doing bits you want it to do (usually with the goal of having it spawn a login shell, where you then can legitimately run anything you want again).

ROP is intrinsically difficult, as it generally requires the attacker to groom data both on the stack and on the heap, which itself is hard. Add in address layout randomization, and you’ll find that most memory errors involving out-of-bounds writing are incredibly difficult to actually exploit, and doing so will generally require chaining together multiple bugs, and most often also applying ROP.

Recently, Intel introduced *Control Flow Integrity* (CFI) as an option that explicitly is meant to thwart ROP.

Remember when we said it often didn’t make sense to move return addresses off the stack? Intel thought it would be too difficult to get the world to stop doing that in practice, but instead, if **duplicates** the return addresses onto a *shadow stack*. Then, when functions return, it ensures that the return locations are consistent.

That obviously would be effective against stack overflows, but what if attacker skips directly writing to the stack at all? For instance, with ROP one often will just manipulate data in way that causes a jump to misfire somewhere, that will run code they like. When that code hits a ‘return’ statement, it might return to the place that CFI expects.

Except that CFI can also validate call sites. Often, ROP will involve jumping into the middle of functions, which CFI can stop. And, it can ensure functions only get called from places they should have been called from.

CFI wouldn’t stop our attack against Python’s virtual machine. But for programs without virtual machines embedded, it is very likely to make ROP-style attacks even more difficult (and thus less practical), for programs making use of CFI.

In summary, while we may not be able to quantify well what percent of memory vulnerabilities modern mitigations thwarts, it’s likely that plenty such errors are simply not exploitable when all easily available system mitigations are applied (many are on for you by default in most apps, but CFI is new enough, and not widely used, and there are some places where these mitigations are NOT applied; see below).

But the space is complex, and I’ve long marveled at the ingenuity of everyone around the table. While it’s been a long time since most bugs were cookie-cutter, or could be exploited without chaining multiple bugs together, the exploit development community has innovated in similarly impressive ways, so it *is* best to always assume that memory problems are likely exploitable. Yes, the number of people with the proper skill set is incredibly low compared to 20 years ago, but we shouldn’t assume it will ever go to zero.

So one might rightly conclude, “why take the risk? Surely it’s just best to get rid of this whole class of issues.” And on one hand, I’d agree, because the bar is high, but the best people can often find and chain together multiple bugs. what they need. Yes, these things can be a problem in Rust and other languages, but they will always be a big problem in C, as long as it exists.

But then again, most of the qualified people doing it are either supplying exploits to governments, or practicing responsible disclosure, which generally should mean that if you stay on top of patching, the risk can be mitigated pretty well.

And if you’re being targeted by a government, enough that they’d be willing to risk exposing a 0-day to own you, they probably have other methods that will work more reliably. Frankly, `xz` isn’t the only long game we’ve seen; governments plant spies in places that can get them access to what they need, and that includes within security companies.

That’s mostly why plenty of of the most paranoid security people hate security products that are big monoliths — no matter how intentioned, you’re going to end up with a much greater attack surface, and quite likely with far more risk, especially if you’re worried about nation states.

I do expect over time, hardware platforms will continue to raise the bar. If we’re lucky, in another decade, we may even be near the point where such mitigations are just about as effective as full bounds checking, but far cheaper (at least, in the environments where they’re available).

But until then, it’s not completely unreasonable for people to believe that, while these problems are a real concern, that they are making enough of an investment in compensating controls to not incur all of the costs they associate with other options.

## Other Memory Errors

Out-of-bounds accesses are not the only thing that qualifies as a “memory error” in C and C++.

These languages are known for users manually taking on the responsibility of allocating and deallocating memory. These languages do come with libraries for helping with memory management, but, unlike many other languages, you are still left to decide when and how to de-allocate heap memory.

There are plenty of other issues beyond pure array bounds errors, including:

1.  If you de-allocate memory that’s still in use by other objects, that can leak sensitive information (such as pointers to ROP targets), partially because C doesn’t zero out memory or ensure that things are written before they are used. Such *Use-After-Free* bugs can also easily result in arbitrary data writes too.
2.  If you can get the program to release memory that has already been released, you can corrupt the internal memory management data structures, which can also have dire consequences. Simply being able to call `free()` twice on the same memory before it gets allocated can be game over.
3.  If the program does math to figure out how much memory to allocate, and the attacker can force the math to yield a result large enough, the number might “wrap around”, resulting in a smaller allocation thanW needed, and forcing a buffer overflow (*integer overflow* issues).

Such problems have been quite common in code, because it is often very difficult to reason about when to release memory, and C programmers doing it manually, often get it wrong. Automatic memory management (e.g., garbage collection) does generally help address most of these issues… unless you can exploit issues with that memory management.

Memory managers in garbage collected languages do tend to be incredibly complicated, and there have been several major exploits for multiple garbage collectors, including most browser JavaScript engines.

## Modern Mitigation

As mentioned above, C++ has done a lot of work to help programmers sidestep the above issues:

1.  The standard library is written in a way that, when functions return, most resources (particularly stack-allocated memory) will be automatically released, without the user having to do anything.
2.  Similarly, C++’s libraries completely avoid raw pointers, and use wrappers that provide automatic memory management via ‘reference counting’.
3.  The APIs for C++’s standard data structures prevent array bounds errors.
4.  C++ has a deep type system, and it’s often possible to be quite confident in the safety of the program, relative to those types. Meaning, if you stick to their guard-rails, you can be sure that type errors (where data is of an unexpected type) are always caught.
5.  There have also long been Garbage Collectors available for C++. The Boehm Collector came out in 1988, and is still maintained.
6.  There are a wealth of static analysis tools for C++ that can identify when code doesn’t follow the C++ core guidelines.

The great thing about the above, is that many C++ programmers are benefitting from most of the above items.

C programmers, on the other hand, tend NOT to enjoy any of the same benefits. While, much like C++, the C standard is frequently updated, unlike C++, C is vastly more conservative on changes, and does not add niceties the way that C++ does.

C programmers can get some of the same benefits:

1.  There are Garbage Collectors for C. In fact, the Boehm Garbage Collector for C++, works just as well with C.
2.  There are quite good static analysis tools for C, mostly centered around the CLang compiler ecosystem.

But, for the most part, C sees its place in the world more as a portable assembly language. We’ll come back to that shortly.

## Why is the View So Skewed?

To the outsider, the discussion so far might fly in the face of the “evidence” you’ve heard. For instance, I’ve heard many people claim, “most CVEs are due to memory safety issues”. I recently read yet [another article quoting this](https://herbsutter.com/2024/03/11/safety-in-context/), which also gave an interesting statistic: at that time (in March, 2024), there have been 61 CVEs in C/C++ code, but only 6 CVEs in Rust code.

All of this is true, of course. But none of the data is a good proxy for risk:

1.  We’ve already established that counting memory errors is not the same as counting *exploitable* memory errors. The later is far more important, and there are good reasons to at least question this.
2.  C/C++ is like a cockroach infestation, in that, you may not see it day to day, but if you know where to look, you’ll find it **everywhere**. While C/C++ haven’t been the most popular languages by any metric in a long time, old building blocks are still widely used and maintained. And both are still common for any new systems-level code. So, for most software we use, no matter what language it was written in, there will probably be critical hidden dependencies under the hood that were written in C/C++. *So saying Rust has had 10% of the CVE total of C/C++ actually feels like it’s either appallingly high for Rust, or amazingly low for C/C++,* given there is actually very little production Rust code in comparison to C and C++.
3.  Even if all C/C++ errors for which there are CVEs happened to be easily exploitable, the data is absolutely skewed towards highlighting memory vulnerabilities, due to the economics around exploitation.

As I noted above, governments (and some corporations) do pay for exploits that they can use “tactically” in operations. In plenty of cases, both have been known to pay six figure sums per bug, and sometimes pay even into the 7 figures.

Vulnerability researchers who play this game generally make more than a year’s worth of salary for the average tech worker with just one bug. And there are absolutely people who manage to sell more than one such bug every year.

Generally, were I a government buying exploits, some of the most important considerations for me, when figuring out what I should buy:

1.  **Reliability**. Due to lots of things (randomness included), lots of exploits work only once in a blue moon. Some exploits, however, are absolute slam dunks, and work every time.
2.  **Ubiquity**. Particularly, I’d care about ubiquity of software among my target. Meaning, if you want to target a particular foreign actor, what software are you like to find?
3.  **Stealth**. Generally, I’d want missions to not be discovered, and to minimize the risk of losing my tools. I’ve seen exploits that can remotely pop common web server software, then leverage another bug to raise privileges (escaping from a container), and then disable SE Linux, all without ever writing to disk, and with only a single log message — one that simply says SE Linux has stopped running. That’s doing pretty well on the ‘stealth’ front.
4.  **Execution**. Generally, I want to be able to bring tools to help with my activities. So full execution is a must-have. Though, data-driven attacks a pretty big thing even in low-level exploitation these days. But I think a lot of the “big actors” also like data driven attacks in higher-level cloud exploitation because so much can be achieved with creds or data on users. Still, if I have full execution, I can do those things, and far more.
5.  **Longevity.** Ideally, the bug isn’t likely to be found and fixed as a matter of course.

There are classes of bugs, like command injection attacks, that can affect most programming languages, that can score pretty well on many of the above fronts. But a really good memory error generally will score even better.

Due both to the innate value of memory errors, as well as the technical challenges involved in actually finding and exploiting such bugs, memory errors are the most prestigious type of bug in the vulnerability research community.

As a result, they get far more attention and press than more mundane issues. That means that there could easily be plenty of reasonably low hanging fruit in code written in other languages, but it also does mean that you’re much more likely to see this kind of vulnerability show up in code you’re using, putting you at more risk.

On the other hand, as time has gone on, exploitation has become harder and less reliable overall as a way to infiltrate, so it’s no surprise to see nation state actors diversifying to other approaches, which we will discuss later.

# Why don’t people switch?

So far, we’ve seen that C is indeed far more susceptible to memory errors than any higher level language.

And while mitigations are reasonably effective, they are not good enough to be the only reason why people don’t switch (well, in the case of C. In the case of C++, they’ve got a better case).

So what are the key factors that keep people from switching?

For the sake of discussion, we’ll act like Economists, and assume that people are rational (acting irrationality would tend to favor C anyway).

First, it shouldn’t surprise anyone that, even if everyone agreed that C should die, it would take an awfully long time, considering I cannot name a single person who believes it’s a travesty COBOL isn’t a common language anymore. Still, there are plenty of COBOL applications out there that aren’t going away any time soon.

Those COBOL applications are still there, because it’s been impossible to rip them out. For every such application I’ve heard of (most of them long-running services in the financial space), the company who owns the application has, at multiple points, tried to migrate away from COBOL, but ultimately failed.

Such are always large applications that are incredibly important to the business’s core economics, and quite complicated. But they have been working robustly for a very long time.

Most people have, over the years, tried building something brand new from scratch, but then have realized it would cost vastly more money to get both feature parity, and to achieve the same level of robustness. Fifty years of insights and bug fixes are encoded, but usually without adequate documentation.

And then, how do you get confidence to swap things out? For instance, let’s imagine you’ve got a system clearing north of $1 Trillion in transactions a day, as it has been doing for many decades. What would it take before you would be willing to cut more than a tiny sliver of that over to a new system, knowing that, if you mess it up, the entire business will likely die?

The monolithic lift-and-shift is too impractical. But even on those old COBOL systems, people have tried swapping out small pieces over time, too. That can eventually work, but it can also lead to pretty catastrophic failure, and several companies have deemed it not worth the cost and risk. So, even though they pay to keep around good talent willing to maintain a code base in COBOL, the economics are keeping the language around.

C (along w/ C++) is far more ubiquitous than COBOL ever was, and it has been the bedrock of most important software technology for pretty close to 50 years.

You might think that’s hyperbole, but it absolutely is not. Currently, Python has been the world’s most popular programming language for several years. The dominant implementation has always been written in C, and much of the core libraries are backed by C code.

JavaScript is the other popular high level language. Yes, it generally runs in browsers, which themselves are typically incredibly large C++ projects.

Even programming languages that intentionally were NOT written in a C-like language are generally going to be dependent on C code. Typically, fairly low-level services like DNS will be offloaded to C code, often in the “standard” libraries.

But at the end of the day, *every major operating system* is primarily written in C, along with some assembly. Most of the network services we use every day are primarily implemented in C or C++. In embedded systems, it’s STILL almost unheard of to use a language other than C or C++. Even in Rust-land you will find some C.

*There are reasons for that*. Here’s a non-comprehensive list:

1.  A lot of C code has become incredibly widely used and trusted over many years, and the world would have to take on a *lot* of pain to migrate.
2.  Plenty of low-level tasks require significantly more gymnastics than necessary in more safe languages, particularly because they are more defensive.
3.  Many embedded environments are constrained enough that bringing more compiled systems languages to the table with lots of dependencies is not feasible. C is incredibly compact and simple in comparison to Rust.
4.  When it comes to developers in the market qualified to do low-level programming, you will find vastly more C and C++ programmers than any other alternative.
5.  Rust is not particularly easy to read or easy to write, and there is not really any other game in town worth mentioning… yet (Zig is getting there). It particularly makes it a huge lift to ask people who have spent 20–30 years honing their craft in C or C++ to basically throw away that institutional knowledge. Nobody wants to experience the feeling of swimming through molasses. More on this below.

A few weeks ago, I was talking about this topic, and I brought up the tremendous level of effort companies running core services would be incurring by migrating from well-designed C code to Rust. The person I was talking to, indicated that would be easy, because there are IETF RFCs around all the pieces, like SMTP and IMAP.

I wish that were good enough. RFCs leave plenty of “SHOULDs” and “COULDs”, and often take time to catch up to evolutions in technology, instead of always defining things before they get built. About 30 years ago, I wrote the first version of Mailman, which by no means needed to implement a full-fledged SMTP server, and I still have the mental scars from new non-conformant use cases leading to subtle bugs around interoperability, coming out of the woodwork at a steady pace.

People have been writing mail clients and mail servers around these core protocols for more than 50 years, and the world is full of things that really should be supported, but are undefined or non-compliant behavior. And some of those issues will come up incredibly infrequently, and be hard to reproduce.

If you’re Google or Microsoft (who run a very large percentage of email these days), how willing are you going to be to jettison code that’s been robust for decades, and is trusted to cover all of those corner cases from the last few decades, especially KNOWING that there’s not a good knowledge base around it?

Now consider that the software here that we run today has built up some trust. Sure, in the bad-old-days, Sendmail was popular and was peppered with holes, because it wasn’t defensively written.

On the Unix side, Postfix has been in active development for 25 years, and is written in C. And while its security track record isn’t perfect, it has been very good, and was designed from the beginning heavily leveraging the least privilege principal.

And even though Postfix was more security conscious from the beginning, and, frankly, easier to configure and use, it took at least a decade (probably longer) before it had mostly displaced Sendmail, mainly because of interoperability. It took time not just to re-discover important anomalies that might disrupt the business, but to prove to people worried about business disruption that it was mature enough to not be a risk on that dimension.

From that same era, there was a second upstart SMTP server, qmail, from Dan Bernstein, which was written even more defensively (but still in C). Its only known issue was an integer overflow back in 2005, when that was a new class of vulnerabilities (and Dan has argued that it wasn’t even practical to exploit in any real world scenario).

But, qmail was not only much harder to use than Postfix and sendmail, but it was not able to get to the necessary level of interoperability to get people to adopt it in mass. As a result, qmail development stopped 25 years ago (although there are still active descendants; they still are not widely used, and I couldn’t imagine them being used anywhere where a bug could be meaningful economically).

So in order to “rewrite it in Rust”, and get that to the point of successfully dethroning Postfix (never mind Exchange), one would have to:

1.  Build something from scratch that meets all the standards.
2.  Get enough real-world usage to find and implement most of the “gotchas” that mature software already will handle.
3.  Build in enough **usability** that people will consider it in bulk.
4.  Keep up with all the new work that does get done in the mail world (there have been plenty of additions over the years, especially in the security arena, and nobody should expect that to stop).
5.  Slowly convince the world to move over.

In a best-case scenario, that process won’t happen quickly; it’s something where we’ll see results another decade down the road. And while, on one hand I’d love to see it, on the other hand, *It’s hard to get excited about a decade-long project to displace something that isn’t generally considered a big risk anyway.*

Of course, the above argument applies to a whole lot of legacy infrastructure written in C. For instance, it’s great to see the Linux community start to experiment with Rust in the code base. But it’s just an experiment at this stage, and if it becomes more than that, it certainly will not be a quick migration.

# C’s long-term niche

In my view, Rust being in the kernel is an impressive accomplishment. Many people have lobbied over the years for C++ to be allowed into the kernel, and it has never happened. The argument against it has been simple — C++ has **too much** abstraction. The kernel basically lives at the bottom of the software stack, and should not incur any unnecessary costs, so the kernel team needs to be able to reason about performance problems, which means they need to be able to see how C code maps to the generated assembly.

I can say from experience, Rust is indeed closer to C on that front. However, Rust is certainly not **better** than C in that regard, and there are certain tasks where Rust very much gets in the way.

Many of those tasks are low-level enough that they’re true systems-level tasks, like memory management, device drivers, exposing new hardware capabilities, and so on.

And yes, you can do these things in Rust, but it is laborious in comparison, and generally will result in leveraging Rust’s ‘unsafe’ capabilities, in which case, you’re incurring the same risks, and why **not** write it in C?

Additionally, just like the Linux kernel doesn’t include the standard C APIs (because they do not make sense in that context; they provide their own internal APIs where needed), Rust cannot use its own APIs; they have to use the kernel’s.

The hardware architectures we use provide very little inbuilt safety at the instruction level. Certainly, even C’s horrible, basic type system is vastly more help than one gets writing directly to the architecture.

In any realistic software system, if you drill down to the lowest levels, there will always be code that will end up needing to be written to target these unsafe platforms.

Additionally, as we noted above, the vast majority of embedded systems use C exclusively. That’s not only because in some environments with weaker CPUs every cycle might count. It’s also because other resources are limited:

1.  Memory might be at a huge premium, including stack space, disk space, cache, registers, the works. The size of the compiled executable can be an issue, as can be any unnecessary space taken up by runtime cruft or fat abstractions when they’re present.
2.  The tooling for such environments generally isn’t good for any other languages. It might be a huge challenge just to get a compiler that produces code at all for the platform, never mind one that provides tools aimed specifically at the platform.
3.  Due to a number of factors, including space constraints and the difficulty that can be involved with test cycles, such environments often have very few external dependencies. Even standard libraries for these environment may completely lack basics like threading. Newer systems languages these days have been trying to keep their standard libraries fairly small, but they are still too large (and the build times are often way too high, particularly with Rust) to be “good enough” in the embedded arena.

C has thrived in this world, and it’s one of the many reasons the C standard aggressively minimizes what API *must* be present, way, way beyond what any of the other systems languages do.

Unfortunately, the embedded world tends to not support many (or any) of the common mitigations that make C safe. They also have other disadvantages from a security perspective, such as a difficult time finding the entropy needed for basic cryptography. Nor do they have the investment that makes upgrading broken software easier.

HOWEVER, these constraints are more business trade-offs made where cost through the supply chain is a huge concern. Much of the software dealing with such constraints is on low-end hardware for a reason, and the products in question often would not be viable if they carried all of the baggage associated with higher-level platforms.

It’s not always just a “cost” consideration, either. Beefier hardware requires more power, and if you’re looking at wearables or other devices that might need to stay on battery power for long periods, those things might matter to people far more than the security risks do.

Those are business trade-offs, and C is really the only non-assembly language that is willing to properly service such environments (well, to a much smaller extent C++, but that is the only other language that has any real traction in the embedded space at all).

People do like to call C a “portable assembler”. That’s not really true at all; in my view, C is *far* more high-level than assembly. When I’m working on compiler tasks and other system-level tasks, were I to be asked to not use C, but to use assembly, that would seem absolutely insane. I only drop to assembly if it cannot be done right in C (or cannot easily be done right), and that often is just a few instructions. I also lose plenty of time when I have to drop to assembly, because it is so infrequent (and each modern architecture is so complex), I have to spend a lot more time in the documentation.

C lives in a unique space that is higher-level than assembly, but lower-level than any other systems programming language (certainly C++, and Rust too). It’s basically a half-step between them, abstracting away many platform portability issues, but still basic enough that, if you DO know an architecture and compiler, you can reliably predict what code will be generated just by looking at the C source.

There are plenty of tasks that are best done at this level. Moving to Rust for such tasks would be nowhere near as much extra work as dropping down to assembly, but it still would be a lot of extra gymnastics, just to be writing “unsafe” blocks anyway. While this would help bound areas where security bugs are likely to be, the more need for such “unsafe” blocks, the more likely I’d be to expect negative net benefit.

Some people have said to me, “does that mean you don’t consider C a real programming language?” Far from it. I think that *assembly* constitutes a real programming language. However, I think we are best served to think about classifying languages using meaningful axises, as best as we can define them. Generally, people who reach for Rust or C care a lot about performance (whether they should or not is another matter). Less often (but still common), resource consumption is an important issue. The axis in my mind has *performance* on one end, and *user experience* on the other, because that is the primary trade-off as you move along the axis.

I group things relative to that access like so:

*   *Assembly languages*, which are so intrinsically low-level and devoid of useful abstraction that I believe people should only deal with it directly in a few special circumstances (including building compilers that automatically target it, unlocking functionality, and very explicit optimization efforts that have a lot of data suggesting the work be done, where compilers are just not in a position to come anywhere near optimal).
*   *Pre-assembly languages,* that try to keep everything as close to the hardware as possible, while giving the programmer as much control as possible over performance. Here, we might sacrifice a little bit on average on performance to make portability and maintainability much better. But, we should still be writing code at a level where we are providing something that should be fundamental, leveraged at scale by software the author isn’t even considering. OS Kernels definitely fall in that category, and perhaps some of the base low-level library facilities should too, but most tasks probably don’t need to be handled at this level. Realistically, C is the only language in this category. Rust may get there someday, but I don’t feel it is anywhere close today. And okay, the name is horrible, but I’ve got nothing better.
*   *Systems languages.* To me, these are languages that focus on providing as much performance as reasonable, while still providing safety in MOST cases (allowing guarded access to unsafe mechanisms). Generally these languages will bend over backwards to avoid traditional garbage collection (or, selectively opt out of it). *That means you may have to do some manual work around memory management.* This has become a “hot” category for language development, and includes C++, Rust, Zig, D and Nim, among many other lesser known ones. Now, the manual memory management generally is far less work and far less error prone than is typical from C. Although, I’d say Rust makes memory management HARDER than best practices in C, but with the benefit of much better safety guarantees. Go’s performance is now good enough that it might even meet some people’s bar for a systems language, though I’ll keep it out of this category due to the lack of first class support for manual memory management.
*   *Compiled languages*. In these languages, performance is still a concern, but so is the overall developer experience. By this point, nobody should have to worry much about memory management. And, there should be a fairly rich ecosystem. There’s generally a strong emphasis on type safety too, with decent type systems. Go, Java and C# are long-standing stalwarts in this category, and I’d put Typescript here, since it is often fully compiled, and has a big emphasis on safety. I probably stick Javascript more in the latter bucket (even though it is often compiled; the naming here is definitely not perfect; it’s just that the correlation mostly applies).
*   *Scripting languages.* In these languages, performance is pretty low on the priority list. Rapid development is highly valued, leading to very rich languages. Usually people don’t want to wait for results to compile, and might even want to see changes without having to re-run the program. Here, dynamic features have traditionally been more highly valued than type safety, although that has started to change (as evidenced by some of the ghastly bolt-on systems that are nevertheless becoming fairly popular).

On this axis, *security* doesn’t actually vary all that much. Yes, proper compiled languages tend to have given it a lot of attention, and systems languages are better than C. Clearly scripting languages sacrifice some safety, but these days they’re usually just as willing to consider security as the compiled languages.

Here, C and assembly definitely are at a firm disadvantage relative to memory safety, but, as we’ve been discussing, it’s not as big of a gap as one might think because:

1.  In desktop / server worlds where security is more likely to be a high priority, there are often very effective mitigations that mitigate the risks. And if it’s clear that where developers take the highest advantage of those mitigations and put thought into their design and implementation, we might end up pretty reasonably confident (e.g., with Postfix).
2.  Other systems level-languages still expose unsafe features, and end up with issues there.
3.  Most languages have dependencies (often, including the compiler) that will be written in memory-unsafe languages.
4.  Plus, as we will discuss at the very end, these is at least one very important way that other languages tend to be LESS secure than C.

Again, this is not to try to argue that C isn’t distinctly and uniquely risky. We consistently see the big C targets get popped: browsers, kernels, network services. While it’s more rare and harder than it used to be, there’s no shortage exploits flowing. Adding new code quickly to such apps helps add attack surface. And, having to rely on a large patchwork of mitigations all being in place at once isn’t awesome, especially when they’re not in the most worrisome places, like embedded systems that are almost never updated.

Still, the people working on OSes, browsers and so on tend to be smart people who want to do the right thing. They’re clearly interested in better security, and we need to understand that there are a lot of considerations we usually are not considering.

That is, it’s hard to imagine your OS and browser of choice being completely written in Rust without unsafe blocks in the next few years, but it’s definitely reasonable to expect that plenty of effort will go towards slowly shifting that way. We can already see it not only in Linux’s acceptance of Rust, but also in Microsoft’s, who is taking it very seriously, and using it for significant pieces. But the people I’ve talked to there also understand the drawbacks, and are being pragmatic.

## Premature Optimization as a Language

I’ve been arguing that, like it or not (and to be clear, I don’t like it), C has a strong argument for its place in the ecosystem, and it is not going away any time soon, because there is nothing appropriate ready to displace it.

I think it’s more concerning that **many** programmers systemically overestimate how important performance is. My observations:

1.  Python has been so popular for so long despite being 50–80x slower than C for much of that run (when languages in the Systems and Compiled categories often have been in the 2x-5x range, and rarely above 10x).
2.  Most language decisions are made long before there is any relevant performance data to guide a decision.
3.  In fact, gathering performance data at all is quite rare, even though decisions made in the name of performance are frequent.
4.  All this despite the fact that it’s oft-repeated that conventional wisdom around how code interactions with the compiler and/or architecture is typically old and wrong. You hear a lot that “it’s generally better to trust the compiler”, and a lot of us who give that advice then turn around and ignore it.

I think any systems programmer being honest would themselves would say premature optimization is a huge pitfall, one that they have fallen into it many times.

I myself often fall prey to it. Sure, I do try to measure, especially when performance is an explicit goal. For instance, when I was working on lock-free, wait-free hash tables (and other data structures), I did a LOT of comparative testing, for instance producing at least 15 different implementations that varied often in very subtle ways, so that I could do controlled testing in a lot of different workloads.

On the other hand, I have habits built over the decades that I can’t fully shake. I still sometimes am compelled to copy function parameters that are passed by reference into locals when that value is used frequently in a function to ensure that it’s in a register, even though:

1.  Were I to measure, it’d almost certainly be irrelevant in all circumstances, in terms of contributing to noticeable performance problems from a user perspective.
2.  Often, I’m using a language where I can take advantage of link-time optimization (LTO), which involves doing additional optimization once you know every single module going into your executable. LTO didn’t exist for me 30 years ago (modules were all separately compiled under the assumption they might be linked to anything in the future), and no compiler was going to risk copying something behind a pointer across modules in case there were threads. I haven’t dug deeply enough to know if the compilers I use bother, but they certainly *could* do this, and if it makes an impact, they probably DO. Of course, not only does it not likely matter in practice, but it probably didn’t even matter in anything I ever did back then either. No, it was just “conventional wisdom” I absorbed.

People are notoriously bad at estimating *performance* (and *risk)*, and for people who are going to overestimate, they are going to end up choosing systems languages (or C, especially if they are cavalier about the risk side of things) when a plain old compiled language might be fine. In fact, in many, many cases, even Python would be fine. Dropbox, for instance, did very well for themselves with most critical things running in Python.

Personally, I think we, as an industry should be far more concerned about people making bad choices around *performance* than we should about security, because:

1.  If you push people to not overestimate the need for performance, security does tend to get meaningfully better as a matter of course.
2.  The indirect economic hit from overestimating on performance can easily be far more impactful to businesses than the cost of underestimating on security. For instance, both C and Rust programmers tend to spend significant amounts of time on memory management, and find themselves spending more time trying to understand low-level issues when they do arise, compared to the compiled languages, and even our highest-level languages. Do you really need to forego the garbage collector? Because that alone often adversely affects development time significantly, never mind the benefits of having much richer abstractions available in a standard library that can lower costs.

Meaning, I’d be just as inclined to push people away from Rust and toward Go as I would to push them away from C and toward Go. If I take off my “security guy” hat, and try to form an opinion that’s as objective as possible, I think that is far more meaningful than the benefit of going from C to Rust, *even if I ignore all of the hidden costs to using Rust we’ve been discussing*.

That’s not to say there aren’t plenty of cases where systems languages, or even pre-assemblers aren’t the right choice. I just think we should routinely push back on this question, and try to get people to try to objectively examine all aspects of why they’re really making the decision:

1.  Is it really because we can demonstrate a performance need? If so, prove it.
2.  Otherwise, it could be valid to consider the *switching cost,* particularly when we start with access to resources seasoned in a particular language, etc.
3.  And if not, what would most likely help with other goals… getting the functionality built faster? Or maybe quality is important (which would push me to Go or other compiled languages, since scripting languages often are harder to drive up quality due to the dynamic nature, and even in a systems language with a very strong type system like Rust, lower-level code seems to implicitly end up with more hidden “gotchas”.

## We Still Need People Learning C

In most scenarios where people select a systems language, we should be urging them to pick something higher-level, instead of prematurely optimizing without data.

More times than not, that will be the right economic decision for individual products and companies. However, the industry as a whole does need to continue to cultivate C programmers somehow.

As long as COBOL’s legs have been, C’s will be MUCH longer. Today, there are incredibly few people with any practical COBOL experience, but things won’t be great if, 50 years from now, there are as few people with practical C experience, because:

1.  Believe it or not, despite its age, COBOL is much higher level and more accessible than C, so it will be much harder to get good people up and running to be able to maintain critical systems 50 years from now, considering how insane C is compared to other programming languages.
2.  There will, without any doubt at all, be far more critical C still in widespread use (very likely including OS kernels).
3.  It facilitates keeping system-level innovation going.

What I mean by that last point, is that, 50 years from now, we will still need people who can become experts at the underlying architectures, AND help enable software developers with any hardware improvements.

Over the last 30+ years, we’ve benefited tremendously as an industry by a large influx of people into programming, and some of them have gone down that path. But:

1.  We’ve also provided people with much higher-level abstractions at the language level than 30 years ago. Javascript, Python, Go and Java do such a great job of abstracting away the hardware details, that only people who get incredibly interested in learning systems stuff do it.
2.  If AI-assisted development becomes incredibly effective, the gap for most developers from where they are to mastering systems programming could grow significantly (which isn’t all bad).
3.  If we do somehow stop most C development, and most people from learning it without any suitable replacement (of which Rust is not, I assure you), we will be massively widening the gap between systems languages and the actual hardware.

C, as horrid as it is, definitely is a much better stepping stone towards understanding low-level architectures from the programming side, relative to any other current language.

If that stepping stone isn’t there, then the people willing to move down the stack all the way to facilitate hardware improvements in software, will drop dramatically due to the level of effort required to learn the basics and accomplish simple tasks would end up being high enough that more people who are interested will either assume they’re incapable or will not want to go through the pain, and give up. People are very goal-oriented creatures, and we tend to not pursue the goals that we perceive as too unachievable, for our own mental health.

I’d feel much more comfortable if there were a pre-assembly language that corrected some of C’s most egregious mistakes (the biggest in my view being C’s treatment of arrays, but my list of issues is very long), and, during development, always perform as much analysis as possible (not just the sanitizers available via the clang project, but runtime safety tools like Valgrind).

Rust currently is the closest thing to that replacement, but over-emphasizing the functional paradigm currently detracts from illuminating the underlying imperative Von Neumann architecture in my view.

# Why One Might Not Choose Rust

Some of these we’ve covered, but overall, I’ve heard people cite:

1.  “Our applications don’t need to be written in a systems-level language; we are fine with garbage collection.”
2.  “We’re comfortable with the language(s) and ecosystem(s) we know, and do not want to lose so much time to learning a new ecosystem.”
3.  “We feel Rust has a steep learning curve, and is hard to write.”
4.  “We don’t have people skilled enough with it (everyone learning it together seems like a waste of resources).”
5.  “Other people’s code written in Rust tends to be hard to understand.”
6.  “Build times tend to be very high, often with too many external dependencies.”

I realize that Rust has become very popular with the Technorati in a very short period of time, and for very good reasons. Personally, I have a great appreciation of some of Rust’s accomplishments; I too masochistically enjoyed my early fights with the borrow checker, because I could appreciate the raw technical accomplishment, and it made me feel smart to be able to be effective at Rust.

Still, I’ve experienced many of the above listed items first hand. About three years ago, I was reading an academic paper that was poorly written, and decided to see if anyone had implemented it yet. I found two different implementations of the same paper, but only two, and they both happened to be in Rust. I’d done some stuff in Rust, so I thought it’d be fine. But, both implementations were both incredibly terse and challenging to comprehend themselves. And if I hadn’t known they implemented the same algorithm, I never would have guessed, because they used substantially different idioms, and looked nothing alike.

It reminded me of reading other people’s Perl in the mid ‘90’s: Rust sometimes feels like a “write-only” language.

And, I’ve written enough Rust and talked to plenty of Rust developers to say confidently that a significant number of people will find it difficult to adopt, and will take a long time to feel as productive in it relative to their current language of choice.

I say that, even having read Google’s blog post where they attempt to debunk Rust being ‘hard to learn’. I read that, but beyond them not sharing any real data, there are issues with this work:

1.  The sentiment of people who found themselves “as productive in Rust” is very skewed by Googlers who are used to building to their internal C++ tool stack, which has enough complexity and controls built up over the years that simply being freed from those constraints will make them feel productive.
2.  Surveying user sentiment instead of metrics around code itself comes with implicit bias that is hard to correct for (look at how much work is applied to correct for such things in political polling, to still have huge misses). Google employees want don’t want to look weak to the people who are surveying them about Rust.
3.  Even if the data were even remotely useful, there is no direct comparison against other languages. The blog post claims their results are the same as any other language, but they explicitly use the word “anecdotally”.
4.  Google’s workforce also tends to be among the most skilled around. Saying, “we think it’s easy at Google” doesn’t generalize in the slightest.

In fact, from what I’ve seen, Rust primarily has only taken off among the Technorati. I think anyone who has done significant work in both Python and Rust should have an intuition that Python is likely far easier for the masses.

Personally, this fact, that Rust is explicitly for the Technorati, people who come to the table with an intuitive grasp of a mathematical function and recursion, is what keeps me from embracing it.

I want programming to be more egalitarian. I think there are many smart, capable people in the world, outside our industry, and outside sciences, who could make amazing contributions to the world if they could more easily translate ideas to computer execution. Much like I don’t like seeing too high a bar for people to learn systems programming, I care a lot about lowering the bar to entry for programming in general (In my view, Python has done the most for the world in that regard).

But fundamentally, Rust is a great language, and people who are comfortable with it should use it where it makes sense. However, I think people should try to be more honest about the economics:

1.  We should be pushing to think more thoroughly about the economics surrounding their choices, particularly pushing to prefer compiled languages over systems languages whenever possible, because the overall economics are likely to be better much of the time.
2.  When pre-assembly needs are there, we should not be forcing people to avoid C/C++. Challenging the choice is fine, but the world’s needs are never going to be met by a single programming language, even in the systems space.
3.  We should be promoting the development, and the success of additional systems languages like Zig, and even something that actually *could* replace C in its niche, so that someday we might be able to put the nails in C’s coffin in a way we haven’t even been able to do with COBOL (despite a forceful push to do so leading up to the millennium).

Currently, Zig and its ecosystem are well behind Rust’s in terms of readiness for many system programming needs, but it takes a much more egalitarian approach to the problem. Between it and Rust, Zig will be a much more accessible language to the bulk of people who have written in compiled languages and even scripting languages.

Part of it is that Rust’s roots are firmly in the functional programming world, where first principles are around mathematically pure functions. There are some people in the world who intuitively understand those things, but it tends to be people with a significant math background.

On the other hand, Zig is still a regular imperative language. First principles essentially are “giving somebody detailed instructions”. Even kids understand such things (even if they generally don’t want to follow them). And indeed, every successful pre-programming project I’ve ever seen (such as Scratch) is imperative.

The fact that functional programming has been widely acknowledged as non-accessible, and difficult to adopt for the last 65 years indicates to me that every class of programming language should have a strong procedural language.

I remember that MIT used to brag about starting with Scheme as a first language, when everyone else was teaching C++, touting the functional paradigm. But they too, eventually moved away from functional languages as an introductory languages.

But, I mostly feel positive about the functional paradigm, and functional languages too, because they do have tremendous advantages of their own, particularly, better encouraging more reliable code that is easier to analyze (and, my copy of *The Implementation of Functional Languages* signed by Simon Peyton Jones 30 years ago remains among my prized possessions).

I’d say that there’s so much value in the functional paradigm, that I believe it’s worth having a good, popular functional language at every abstraction level down to perhaps pre-assembly (I’m not sure there would be much utility in a functional assembly language any time soon. But I’d be excited to be proved wrong!)

On the other hand, I think the object-oriented programming paradigm has much more limited utility and is better off either going away entirely, or at most, being a de-emphasized feature.

# Where Rust is probably a Bigger Security Risk than C (for now)

I get disappointed when well-known security thought leaders make statements like, “It’s irresponsible to build critical applications in non-memory safe languages”, not simply because they ignore the economic complexities and trivialize a complex decision, but also because, even if we only think about security, as bad as memory safety issues in C are, it isn’t clear cut that the risks are worse than in other languages.

Specifically, C programs generally have a small number of external dependencies, where often those dependencies are among the most used pieces of software out there (such as the C standard library).

Most other languages are much better equipped to support programmers leveraging the work of other programmers. In some sense, that’s a good thing from a business perspective. But from a security perspective, more dependencies not only tends to increase our attack surface, but it leaves us more open to supply chain attacks.

One thing I read in the last year or so that stood out for me was a [comment on Hacker News](https://arc.net/l/quote/roohdlws) about Rust, but in a thread about Swift:

> My current problem with rust is the dependency hell. Hundreds of sub dependencies for every top level one. Yes some of them are super common like serde or rand, or oddly some crate that seems to be just to create directories on the filesystem?! A blessed subset of crates is what I was counting on to save the day, but when something like tonic brings in 100 or so fine grained one-off sub dependencies I don’t think that can work. Right now I am just plugging my ears saying “my code is memory safe and I am fearlessly concurrent!” But I am thinking “what horrible thing is lurking in the depths of my dependency tree and which state actor put it for later?” If that seems paranoid look at the recent issues with pypi malicious packages. I know I can roll my own, but that cost money, and if tokio or tonic didn’t exist, and crates wasn’t so darn easy to use, maybe google would have made a monolithic grpc crate instead?

The `xz` affair is the most recent and most compelling example of such a supply chain attack, but that’s just one that the industry was lucky enough to find.

Rust makes it easy to pull in outside dependencies, and much like in the JavaScript ecosystem, it seems to have encouraged lots of tiny dependencies. That makes it a lot harder to monitor and manage the problem.

But Rust’s situation is even worse than in most languages, in that core Rust libraries (major libraries officially maintained by the Rust project) make heavy use of third party dependencies. The project needs to take ownership and provide oversight for their libraries.

To me, this has long been one of the biggest risks in software. I can write C code that is reasonably defensive, but I have a hard time trusting any single dependency I use, never mind scaling that out.

Properly securing your dependency supply chain is a *much harder problem* than writing safe C code. Personally, I only pull in dependencies beyond standard libraries if the work I’d have to do in order to credibly replace the functionality is so great that, if I didn’t bring in a dependency, I would choose not to do the work.

C is a lot better than Rust in this regard, but it’s not particularly great. Partially, that’s because the C standard libraries (which I am always willing to use; the core language implementation and runtime is a given) are not at all extensive. People who write a lot of C end up building things themselves once and keeping them around and adapting them for decades, including basic data structures like hash tables.

I have personally always been far more concerned about minimizing dependencies than buffer overflows. There are straightforward approaches to minimizing memory safety problems (discussed a bit below), and they’re not too hard to apply in most applications.

But digging into each and every dependency? Even the best efforts in supply-chain security so far aren’t in a great position to help with attacks like the recent `xz` affair (note to my spell checker, this is not the *xyz affair* from the history books, but I am indeed trying to evoke it). Nation states are willing to play the long game, and once a developer builds up trust in some downstream dependency of yours, it’s not too hard to introduce backdoors in a way that’s likely to be considered an accidental bug, if other people find it at all.

With `xz`, the backdoor didn’t directly make it into the source tree, so it was more overtly a backdoor. But, we’ve known for an awfully long time that stealthy back doors are indistinguishable from bugs.

I can remember from the early days of the software security industry (probably about 25 years ago), discussing backdoors with Steve Bellovin, who related a story from his days at Bell Labs, where he found a pretty subtle memory error, that had lingered in the code of a former employee, that his instinct told him was an intentional backdoor. It felt intentionally placed, and if I recall correctly, there had been some bad blood. But being indistinguishable from a bug, how could he ever prove it?

Certainly, that was back in the day where many memory errors generally could result in reliable exploits. But it’s still not too hard, especially considering, even though we have more of a peer review culture than we used to, plenty of code that is “reviewed” isn’t reviewed thoroughly enough by people who are looking for the right things.

Plus, code review is a lot harder to do well than writing code, which is part of the reason I don’t expect to use LLM-based code generation in the near future — it turns programmers into both “product managers” writing requirements, AND code reviewers. Currently, it feels easier to me to “just” be an engineer.

Anyway, the more dependencies you have, the larger your circle of implicit trust is, the larger your attack surface is, and the more supply chain risk you’re taking.

That makes Rust in particular a pretty large risk on the supply chain security side, whereas C scores pretty well. And as far as modern risks go, this seems an incredibly practical and significant risk for any code that might eventually get rolled up into something that is used in organizations that nation states might want to attack.

C’s advantage in terms of lack of dependencies (which can come with a lower attack surface in general) is large, but still doesn’t make it the right economic choice in the first place. It might still be wiser to choose Rust when all economic factors are considered, but the security argument is just not one I find compelling enough.

Generally, I think Rust (and pretty much any programming language) would be served well to take ownership of their standard libraries. Pull in all the dependencies, and be willing to take ownership.

Also, I would typically argue for languages to incorporate *more functionality* into their standard libraries, even though the recent trend is to move to less functionality. Yes, from a security perspective, this technically increases your attack surface. But not really:

1.  If people feel like they may need to pull in a particular bit of common functionality from the outside, they’re very likely going to do it, whether the standard library exports it or not.
2.  With link-time-optimization in particular, it’s not too hard to excise bits of a language’s standard libraries that are not actually getting used, which brings the attack surface down to about the same thing (though, many non-systems languages don’t worry about link-time optimization).
3.  The language maintainers, by taking ownership, can not only better focus on properly vetting the security risks in the solutions people are likely to use, they also are more likely to look for the architectural efficiencies that are going to help with the attack surface.

You may or may not end up with fewer people touching the code, but the languages should be willing to be accountable for the functionality people are likely to need, especially when, like Rust, security is billed as one of the primary motivators for using it at all.

Languages like Go and Python that have extensive standard libraries that the language maintainers take responsibility for are actually the best case scenario in my opinion. Yes, more people touch the code, but the DIY economics are often the wrong choice, and having organizations willing to both be accountable, and provide an environment where people **can** focus on minimizing dependencies if they feel its important, is a good thing.

Yes, Python has become so popular, that plenty of people use outside dependencies, and there are several popular package managers. However, it’s still in a vastly better place from a supply chain perspective than JavaScript, which has become famous among developers for hidden dependencies on trivially small packages.

# Recommendations

Frankly, while I think that today, the supply chain issue might possibly give C an edge over Rust thinking strictly about security, that’s an advantage that Rust can easily make disappear. And when it does, C will be left with concerns about memory management. My intent here isn’t to argue for using C over Rust, it’s to show that decisions around language choice are far more complex than the sound bytes people fling around.

I’ve covered a lot of ground (and have already excised quite a lot, including a lot of example code), but let me try to focus on what’s actionable.

## For teams

1.  **Explore the overall economics of choices.** When selecting technologies, challenge each other to think about the broader economic implications, and try to get numbers to support your hypotheses. This has many facets, but in the context of this discussion, it should often push you toward languages like Go and away from either Rust or C, because too often, people will go with their gut on this, which will generally over-estimate the importance of performance.
2.  **If you don’t want to analyze up front, don’t start with systems languages.** Instead, start with Go, Swift, Java, C#, or any really other good compiled language. Remember here that if your gut is shouting about performance, Python has acceptable performance for all sorts of things (remember Dropbox?) and compiled languages are all still going to be vastly more efficient than that.
3.  **Do consider security.** Just because you’re going with Rust doesn’t mean you’re going to get it for free. Bad things can still happen to you. Please dig past the “thought leader” sound bytes on this.
4.  **Avoid unnecessary dependencies.** I will leave ‘unnecessary’ vaguely defined here; you need to be educated and judge all the economic factors. But note that, there are often other benefits to fewer dependencies, from shorter build times to less surface to test, to less risk from API changes or bugs from downstream dependencies.
5.  **Be aware of the dependencies you do have.** More than just trying to get your arms around the transitive graph that dependency scanning tools provide, it’s worth digging down to understand what those things aren’t finding. There’s probably some C libraries linked into your runtime somewhere, and it helps when the next *xz affair* surfaces to be able to more easily, more honestly assess your risks.
6.  **Try to ensure outside security review.** For enterprise software, this is almost a given these days, because large buyers will demand evidence of it. But everyone should be considering how they can facilitate such review regularly, and treat it as an opportunity, not just a box to tick.
7.  **If you choose C, provide documentation around your decision.** I say this because the security concerns are absolutely real concerns, and people deserve to know you have done a good job. C++ to my view is not in the same category if you stick the with official recommendations, yet plenty of people aren’t really in compliance, so I would err on the side of caution, and do the same with C++.
8.  **If you choose C, proactively address the memory safety concerns effectively.** For instance, if at all tractable, use the Boehm collector. Or else, use best of breed techniques when that isn’t possible.

Note, the primary reason why many C developers fall back on unsafe primitives isn’t a lack of education about the security risks, its that they generally have a few large dependencies (such as OpenSSL or some other crypto library), and it isn’t clear how to, for instance, apply the Boehm Garbage Collector to those third party libraries in an easy, portable way (although in many cases, if you compile the source yourself, it’s actually easy to just redefine `malloc()`and friends to call the Boehm API, and in others, there are some low-level techniques that can address the issue, but they’re too low-level to recommend).

Even for those building their own lightweight memory management (*arena allocation* is becoming popular, which one could consider a very lightweight form of garbage collection), but many of the most common dependencies like OpenSSL provide direct ways to hand the library a memory allocator, if you look for them.

I can cover this in detail if there’s enough interest in it (even though I’d prefer you to adopt some other language instead).

## For The Security Industry

Here, I certainly mean thought leaders, but anyone who cares enough to be making decisions based on security, and talk to other people about it at all.

1.  **Most importantly, try to keep in mind that just because you think it’s a bad security decision, doesn’t make it a bad decision, overall.** Try to listen to non-security people, and learn about how their priorities. You probably rail against “FUD” (fear, uncertainty and doubt), but oversimplifying non-trivial issues to push for security at all costs is essentially you spreading FUD. The use of C is just one example here. But remember, even the security implications are far more complex and subtle than we might be thinking about in the moment.
2.  **Make sure the industry considers the broader economics.** Sure, outside the industry, security is far less important than it is, inside the industry. But I think it goes much deeper — if the security industry tries to push too much work to the rest of the industry, we will at the very least destroy more credibly, but could conceivably do a lot of economic damage, for instance, by driving up the costs and liability risks to small businesses and individuals so high that they just don’t bother.
3.  **Contribute to solving our legacy software problems.** I’d love to get rid of C as well, but the reality is that C will be even more pernicious than COBOL has been, for good reasons. Saying, “rewrite it all in a safe language, and migrate everything ASAP” is not really a solution. It’s aspirational at best, and completely impractical from a risk management perspective. Like with replacing Sendmail with Postifx, we can make progress, but it will need to be incremental, and we’ll have to be more practical.

In short, yes, let’s 100% kill C and C++, but let’s be practical about it, and be good collaborators with the rest of industry. Our goal should 100% be to put ourselves out of jobs (or, since we’re not at risk of that, try to work toward a world where it’s a shockingly small industry… even if in the far distant future).

## For the rest of the industry

In general, the rest of the software industry should work with the security industry to visualize what it takes to address risks. Specifically:

1.  Help figure out how can we can keep a fresh pipeline of people who will be able to understand the boundaries software and hardware. That means, we need to eventually get skilled people effectively doing pre-assembly tasks in a way that’s far less risky than doing them in C today (not, trying to keep those tasks from ever happening). If the answer isn’t “another programming language”, then great, but if it needs to be another language, both sides need to be serious about it to ensure success, because the world is littered with languages that have no traction.
2.  Please push others to show their work on broader economic decisions, and don’t assume the world is as simple as whatever “conventional wisdom” gets passed down. That’s a swiftly moving target.
3.  Language designers, in addition to the ways you already consider security, please also give much more thought and attention to the impact of third party dependencies, realizing that balancing contribution with security is hard. We need better pragmatic solutions.
4.  And in all cases, try to assume you’re wrong about the things we all systemically mis-estimate, like performance and security. Assume you can be wrong in either direction, too! Now, try to collect some hard data to give you a better sense of where the reality actually lies.

# Feedback

I’m happy to discuss the topic or take any feedback. As I’ve said, I’m happy to keep learning and re-evaluating my views. However, this story will be published while I’m on a long vacation, so please do reach out to me, but I won’t be quick to respond.