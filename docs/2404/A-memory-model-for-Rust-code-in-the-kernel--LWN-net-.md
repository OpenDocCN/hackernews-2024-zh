<!--yml
category: 未分类
date: 2024-05-27 13:00:15
-->

# A memory model for Rust code in the kernel [LWN.net]

> 来源：[https://lwn.net/SubscriberLink/967049/0ffb9b9ed8940013/](https://lwn.net/SubscriberLink/967049/0ffb9b9ed8940013/)

| **Did you know...?**LWN.net is a subscriber-supported publication; we rely on subscribers to keep the entire operation going. Please help out by [buying a subscription](/subscribe/) and keeping LWN on the net. |

By **Jonathan Corbet**
April 3, 2024

The Rust programming language differs from C in many ways; those differences tend to be what users admire in the language. But those differences can also lead to an impedance mismatch when Rust code is integrated into a C-dominated system, and it can be even worse in the kernel, which is not a typical C program. Memory models are a case in point. A programming language's view of memory is sufficiently fundamental and arcane that many developers never have to learn much about it. It is hard to maintain that sort of blissful ignorance while working in the kernel, though, so a recent discussion of how to choose a memory model for kernel code in Rust is of interest.

#### Memory models

It is convenient to view a system's memory as a simple array of bytes that can be accessed from any CPU. The reality, though, is more complicated. Memory accesses are slow, so a lot of effort goes into minimizing them; modern systems are built with multiple levels of caching for that purpose. Without that caching, performance would slow to a crawl, severely impacting the production, delivery, and consumption of cat videos, phishing spam, and cryptocurrency scams. This prospect is seen as a bad thing.

Multi-level caching speeds computation, but it brings a problem of its own: it is no longer true that every CPU in the system sees the same memory contents. If one CPU has modified some data in a local cache, another CPU reading that data may not see the changes. Operations performed in a carefully arranged order may appear in a different order elsewhere in the system. As is the case in relativistic situations, the ordering of events depends on who is observing them. Needless to say, this kind of uncertainty also has the potential to create disorder within an operating-system kernel.

CPUs have special operations that can ensure that a given memory store is simultaneously visible across the system. These operations, though, are slow and need to be used with care. Modern CPUs provide a range of "barrier" operations that can be used to properly sequence access to data with less overhead; see [this article](/Articles/576486/) for an overview of some of them. Use of these barriers can be somewhat complex (and architecture-specific), so a few generic interfaces have been created to simplify things (to the extent that they *can* be simplified). A memory model combines a specification of how barriers should be used and any interfaces that ease that use, describing how to safely access data in concurrent settings.

"The majority of the _good_ programmers I know run away screaming from this stuff. As was said many, many years ago - understanding

`memory-barriers.txt`

is an -extremely high bar- to set as a basic requirement for being a kernel developer."

—

[Dave Chinner](/ml/linux-fsdevel/20200716014656.GJ2005@dread.disaster.area/)

, 2020

The C11 standard, for example, defines some

[atomic types](https://en.cppreference.com/w/c/language/atomic)

and

[atomic operations](https://en.cppreference.com/w/c/atomic)

. C++ offers

[an atomic type](https://en.cppreference.com/w/cpp/atomic/atomic)

of its own. While the kernel community has occasionally

[discussed](/Articles/691128/)

using the C atomic types, that has never happened for a number of reasons, some of which will become apparent below. Instead, the kernel defines its own memory model, described in the infamous

[memory-barriers.txt](https://www.kernel.org/doc/html/latest/core-api/wrappers/memory-barriers.html)

file (and sometimes referred to as "LKMM"). Few kernel developers understand this model in detail (and many

[find it too subtle](/Articles/827180/)

to understand deeply), but it governs how memory access works at the lowest levels. (See also

[this article series](/Articles/718628/)

for more on the kernel's memory model).

One of the early concerns about incorporating Rust into the kernel is that the Rust language lacked a memory model of its own. That gap has since been filled; the [Rust memory model](https://doc.rust-lang.org/std/sync/atomic/#memory-model-for-atomic-accesses) looks a lot like the C++ model. Boqun Feng, who helps maintain the kernel's memory model, thought that it would be good to formalize a model for Rust code to use in the kernel. Should the Rust model be used, the kernel's model, or some combination of the two? He [posted his conclusion](/ml/linux-kernel/20240322233838.868874-1-boqun.feng@gmail.com/) to the linux-kernel mailing list: Rust code should adhere to the kernel's memory model. He included an initial patch set showing what that would look like.

#### Using the kernel's model

The reasoning behind this conclusion is simple enough: "<q>Because kernel developers are more familiar with LKMM and when Rust code interacts with C code, it has to use the model that C code uses</q>". Learning one memory model is hard enough; requiring developers to learn two models to work with the kernel would not lead to good results. Even worse results are likely when Rust and C code interact, each depending on its respective memory model to ensure proper data ordering. So, as long as the kernel has its own memory model, that is what Rust code will have to use.

Kent Overstreet [pointed out](/ml/linux-kernel/s2jeqq22n5ef5jknaps37mfdjvuqrns4w7i22qp2r7r4bzjqs2@my3eyxoa3pl3/) a disadvantage of this approach: the kernel will remain incompatible with other Rust code and will not be able to incorporate it easily. He suggested that, perhaps, the kernel's memory model could be rebuilt on top of C or C++ atomic operations, at which point supporting the Rust model would be easier. That seems unlikely to happen, though, given the strong opposition from Linus Torvalds to any such change.

One of Torvalds's arguments was that [language-based memory models are insufficiently reliable](/ml/linux-kernel/CAHk-=whY5A=S=bLwCFL=043DoR0TTgSDUmfPDx2rXhkk3KANPQ@mail.gmail.com/) for use in the kernel, and that is not a problem that can be addressed quickly. "<q>The C++ memory model may be reliable in another decade. And then a decade after *that*, we can drop support for the pre-reliable compilers.</q>" Thus, he said, "<q>I do not understand why people think that we wouldn't want to roll our own</q>". Feng [added](/ml/linux-kernel/Zf4fDJNBeRN5HOYo@boqun-archlinux/) that the kernel's memory model encompasses a number of use cases, such as mixed-size operations on the same variable, that are important to the kernel. Those uses are not addressed (or allowed) in the Rust model. He did suggest that, perhaps, a subset of the Rust model could be implemented on top of the kernel's operations.

Another reason for the kernel project to implement its own memory model, Torvalds [said](/ml/linux-kernel/CAHk-=whkQk=zq5XiMcaU3xj4v69+jyoP-y6Sywhq-TvxSSvfEA@mail.gmail.com/), is that kernel developers need to be deeply familiar with the architectures they are supporting anyway. There is no way to create a kernel without a lot of architecture-specific code. Given that, "<q>having the architecture also define things like atomics is just a pretty small (and relatively straightforward) detail</q>".

Torvalds has [another reason](/ml/linux-kernel/CAHk-=wjP1i014DGPKTsAC6TpByC3xeNHDjVA4E4gsnzUgJBYBQ@mail.gmail.com/) for sticking with the kernel's memory model, though: he thinks that the C++ model is fundamentally misdesigned. A key aspect of that model is that data exposed to concurrent access is given a special atomic type, and the compiler automatically inserts the right barriers when that data is accessed. Such a model can ensure that a developer never forgets to use the proper atomic operations, which is an appealing feature. That is, however, not how the kernel's model works.

In the kernel's memory model, it is not the type of the data that determines how it must be accessed, but the context in which that access happens. A simple example is data that is protected by a lock; while the lock is held, that data is not truly shared since the lock holder has exclusive access. So there is no need for expensive atomic operations; instead, a simple barrier when the lock is released is sufficient. In other settings, where a lock is not held, atomic operations may be needed to access the same data.

Torvalds argued that the kernel's approach to shared data makes more sense:

> In fact, I personally will argue that it is fundamentally wrong to think that the underlying data has to be volatile. A variable may be entirely stable in some cases (ie locks held), but not in others.
> 
> So it's not the *variable* (aka "object") that is 'volatile', it's the *context* that makes a particular access volatile.
> 
> That explains why the kernel has basically zero actual volatile objects, and 99% of all volatile accesses are done through accessor functions that use a cast to mark a particular access volatile.

This approach has been taken in the kernel for a long time; it is described in the [volatile-considered-harmful document](https://docs.kernel.org/process/volatile-considered-harmful.html) that was first [added to the kernel](https://git.kernel.org/linus/0faa45480261) for the 2.6.22 release in 2007.

The outcome of this discussion is clear enough: Rust code in the kernel will have to use the kernel's memory model for the foreseeable future. The Rust language brings with it a number of new ways of doing things, many of which have significant advantages over C. But bringing a new language into a code base that is old, large, and subject to special requirements is always going to require some compromises on the new-language side. Using the kernel's memory model may not actually be a compromise, but it is different from what other Rust code will do; it will be one of the many things Rust developers will have to learn to work in the kernel project.

* * *

(

[Log in](https://lwn.net/Login/?target=/Articles/967049/)

to post comments)