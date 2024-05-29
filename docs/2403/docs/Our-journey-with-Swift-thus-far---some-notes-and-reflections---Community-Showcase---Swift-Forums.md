<!--yml
category: 未分类
date: 2024-05-27 14:47:21
-->

# Our journey with Swift thus far - some notes and reflections - Community Showcase - Swift Forums

> 来源：[https://forums.swift.org/t/our-journey-with-swift-thus-far-some-notes-and-reflections/70510](https://forums.swift.org/t/our-journey-with-swift-thus-far-some-notes-and-reflections/70510)

Hello!

# Background

Our team is quietly working on developing a new electronic trading system using Swift on macOS and Linux, building on our experience from two previous projects that have been done in C, Objective-C, C++, Java, C# and even for a while, homegrown DSL. All running on multiple OS:s at various points; NeXTSTEP, OpenStep, Windows, HP-UX, Solaris, Linux. These past projects have been fairly sizeable, with over 25 million lines of code and several hundred components between them over a period of over 25 years with teams ranging between 40 and to 100+ engineers over time.

These are very high performance, distributed cluster systems on the server side, combined with sophisticated frontends with visualisation and monitoring. The server side provides APIs for customers to hook in custom plugins that perform e.g. trading activities or pricing, which need to run in-process typically (i.e. dynamically loaded). This has been done with both C++, Java, C, and custom DSL. When we talk about "server side" here, it is in the more traditional sense with high performance IPC etc, so no HTTP, JSON or similar in general.

Delving into Swift's ecosystem has provided us with valuable insights into both its advantages and areas where there's room for growth. The feedback we're sharing is based on our journey so far, and the hurdles we've encountered, with the intention of offering constructive thoughts for Swift's continued refinement. There's still some way to go before we're launching properly, but we wanted to take this time to share a few of our reflections and musings.

# Why Swift?

We have a vision that our next-generation system would not only use an expressive, memory safe and highly performant language, but also that we should be able to use a single technology stack from top-to-bottom. That enables great code re-use, engineers can more easily pick up work in various layer of the stack and we can benefit from the unified implementation of various types between both client and server pieces more easily. We also have a commercial necessity to be able to deploy on Linux on the server side in addition to macOS for visualisation (and possibly other Apple platforms going forward).

Swift ticks most of those boxes while also providing the ability to create native, good user interfaces using SwiftUI. There are a few other options, but for various reasons they didn't meet our criteria (and not relevant to this post).

# Good stuff

### Overall design

*   Swift's design of being "safe by default" - the strict type checking felt a bit frustrating at the beginning (ok, still sometimes ;-) but after coming over the initial (small) hurdle, we could really see that it paid off handsomely - code *much more* often will work immediately as expected (especially if compared with C-style languages) after going through the compiler. Major refactoring work is a lot faster and more robust with Swift in general, something that has significant value to us. The number of runtime errors is just a fraction of what's common with more traditional tooling. We're really happy with that aspect. The static data-race safety guarantees coming up is just the icing on the cake.

*   The language expressiveness is overall very good and we think the overall philosophy has a set of good tradeoffs. We're a little bit worried that the complexity of the language continues to grow for newcomers, but hope the philosophy of progressive disclosure will make it manageable over time.

*   The pace of development of major language features during the last few years has been great, with several things being right up our alley and being really useful to us, e.g. Structured concurrency/Async+Await/Actors, Distributed Actors, Memory ownership (super important and great it is being worked on!), and overall improvements to generics and Macros.

*   The open evolution process is overall really good (modulo some hiccups like result builders). To have visibility in the roadmap really helps and is appreciated - e.g. we saw that Distributed Actors and the new Predicate type was in the pipeline, and could choose to just stub out trivial implementations internally and then move over as they shipped - very good.

*   ABI stability is really a key feature - we do want to use library-evolution for our customer-facing SDK:s and this is just a great feature - not just for Apple as a system provider, but also for us as a third party. We need to be able to ship shared libraries that customers can link against so we can load plugins dynamically with resiliency and type compatbility.

### Ecosystem

*   The ecosystem of packages usable for server-side development [was more extensive than expected](https://forums.swift.org/t/the-current-state-of-swift-for-server-and-linux/47732/9) - we've over time also have helped contribute to some of the pieces that we've missed, like e.g. [HDR Histogram](https://github.com/HdrHistogram/hdrhistogram-swift), [swift-kafka-client](https://github.com/swift-server/swift-kafka-client) and [performance benchmarking](https://github.com/ordo-one/package-benchmark)

*   We also are very happy with the native rewrite of Foundation (and are starting to use certain new features like Predicates heavily already) and are *really* looking forward to its integration as the built-in implementation for Linux - it moved us from having a no-Foundation policy to now instead adopt it.

*   SwiftPM plugins (and Macros) are enabling a wider range of workflows.

In general we are very happy with the language itself and see significant progress over time.

# Major challenges

Much of issues are centered around richer dynamic library support, API packaging and Linux platform support, something we need for packaging products properly.

*   [Dynamic Library support on Linux with library evolution](https://github.com/apple/swift-package-manager/issues/5714) - even a limited officially supported package with clear limitations would be useful (as our customers will have a completely controlled environment, it's possible to mandate specific library versions and patch levels for OS etc).

*   [Library support for artifact bundles](https://github.com/apple/swift-package-manager/pull/6967) (we are currently running with a custom toolchain that supports xcframeworks on Linux to work around it, but that's not long term sustainable).

*   Concurrency runtime analytics tools could be richer, especially for Tasks - while we have some of the tooling in Instruments, it would be fantastic to also have support for accessing similar task information (async task stack backtraces etc, for creation/running/sleeping) in LLDB (especially import on Linux that lacks Instruments...), including inspecting Task locals.

*   [Compile and runtime availability checking](https://github.com/apple/swift/issues/60458) support for third party APIs is not available currently.

*   Fundamental support for dynamic libraries overall as a first class citizen, e.g. [targets can't be linked dynamically](https://github.com/apple/swift-package-manager/issues/4951) .

# Minor challenges

### Concurrency and testing

### Tooling and platform support

*   Linux support has been lagging at times (e.g. Macro support), which has postponed adoption of newer releases for us and it hasn't always been transparent .

*   Build times on Linux when using the new Foundation are quite painful (we need to not only build swift-foundation, but also swift-syntax due to use of macros - it's a well-known problem of course, but just wanted to mention it as it causes long CI turnaround for us).

*   Build Stability and Performance - we *very* often need to nuke `.build` or SwiftPM caches or Xcodes derived data due to weird issues (like missing Packages in Xcode), e.g. things seem to rebuild more than expected (e.g. [this example](https://github.com/apple/swift-package-manager/issues/7210) for Benchmark).

*   SwiftPMs model with products/targets is a bit problematic for larger projects; targets are exposed to downstream projects, targets can't depend products in the same package, target name conflicts (aliasing doesn't always work out there) with module names (perhaps not SwiftPM exactly).

*   Local development of dependencies is a bit clunky, Swift package edit only solves this partially. We end up with a sometimes convoluted and non-ideal project structure with some tools / workaround on top of it.

*   LLDB support has been spotty with many crashes on Linux especially (and [little updates on issues reported](https://github.com/issues?q=is%3Aopen+is%3Aissue+author%3Ahassila+archived%3Afalse+SR-)), it is improving over time though.

*   Libdispatch as the underpinning for thread management (on Linux) continues to lag the macOS implementation (assuming primarily due to the difficulty of merging newer implementations as libdispatch is heavily using Mach:isms). Perhaps the approach to integration with thread management could/should be reconsidered at some point, but understandably there are a lot of historical considerations to take here as there's a lot of code using Dispatch.

### Performance

*   Performance issues are sometimes a bit hard to diagnose and address (e.g. unnecessary copying, we can get tons of metadata lookups when using existensial/generics if not getting the incantations right, ARC traffic, etc) - and [need workarounds](https://forums.swift.org/t/modern-way-to-vend-specializations-of-a-generic-type/65657/5) that allow the compiler to specialize properly. It feels like some sort of instrument for analyzing anti-patterns in Swift could help, would be happy to discuss details.

*   Generics, while having improved over time, still is not without friction and it is fairly complex at times to get the desired behavior. From a performance point of view it's a bit of hit-or-miss. It's challenging to guarantee that things will get specialised, and it is easy to make a nice generic refactoring that just runs over a performance cliff that you don't expect. It would be great with improvements in this area, e.g. there have been discussions about various annotations (e.g. to guarantee specialization).

*   [KeyPath performance is problematic](https://forums.swift.org/t/pitch-swift-predicates/62000/35) - unfortunately the performance of KeyPaths has forced us to do workarounds even in fairly simple cases like sort comparators as they would take over samples completely - it'd be nice if they could be used without fear.

*   Deeper documentation and hints on best practices for performance when using generics and protocol oriented programming (related to the previous point). Basically a follow up to the older performance best practices, but with a bit of more focus on cross-module considerations.

*   Analyzing program behaviour is [missing some runtime hooks](https://github.com/apple/swift/issues/64636) to better understand runtime behaviour and perform better benchmarks.

Overall, we'd welcome further investment in runtime performance, compile times and overall robustness (SwiftPM / Toolchain / Linux support on par).

# Closing notes

In conclusion, our experience with Swift, while largely positive due to its modern features and robust ecosystem, also highlights areas where enhancements could further refine and expand its capabilities, especially for slightly more complex and advanced development scenarios. We hope this feedback serves as a constructive contribution to the ongoing development and improvement of Swift, reflecting our commitment to help pushing the ecosystem in a positive direction. We'd be happy to elaborate and engage in discussion on any of the above topics.

Many thanks to everyone involved in making Swift with related packages available - we think it's on a great trajectory, looking forward to continue on our journey!

A special thanks to everyone who participates and shares their knowledge on these forums, it's an invaluable community resource, thanks!

Joakim