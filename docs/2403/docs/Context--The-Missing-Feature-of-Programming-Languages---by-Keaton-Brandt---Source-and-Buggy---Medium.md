<!--yml
category: 未分类
date: 2024-05-29 12:40:24
-->

# Context: The Missing Feature of Programming Languages | by Keaton Brandt | Source and Buggy | Medium

> 来源：[https://medium.com/source-and-buggy/context-the-missing-feature-of-programming-languages-7c1095fe8d32](https://medium.com/source-and-buggy/context-the-missing-feature-of-programming-languages-7c1095fe8d32)

Modern computer hardware is so capable that we can often simply ignore its limitations when writing code. Conceptually though, any function that allocates memory is inherently context-dependent: its outcome depends not only on the arguments passed in but also on the amount of contiguous RAM the system is able to grant.

> **Context [Noun]:** Any state that can affect a function’s outcome or performance without being passed in as an argument.

Other basic examples of context include: Global variables, calls to external services, information about the current thread (eg. whether it’s the UI thread), and information about the current stack (eg. to create a stack trace for an exception).

The problem with context is that its assumptions and mutations are happening in the musty back-alleys of application code — the low-level implementation details, not the brightly-lit interface definitions. Compilers simply don’t check what context a function relies on (whether it reads from `stdin`, for example), and functions don’t report it. It’s not a black market but it is a [dangerously unregulated one](https://en.wikipedia.org/wiki/Gun_law_in_the_United_States).

When I call `getTimestamp(user.registeredAt, TimeZone.PST)` I can *assume* that it’s just doing some simple math and returning a result. It *probably* doesn’t make network calls or hold locks or mine bitcoin, so I *probably* don’t have to optimize how often I call it. But actually, it *might* have to look up this year’s timeline for Daylight Savings Time, either from a database or from the network. As programmers we have to rely on intuition for cases like this, which is not exactly a strong foundation for stability.

If programmers were structural engineers

## Coralling Context

Context is only dangerous because it is absent from a function’s arguments and so is opaque to the caller. The caller, for example, has no way to know if a function could cause a deadlock because it cannot know if the implementation of that function relies on locking. The simplest way to deal with context is to simply get rid of it, and pass *everything* as function arguments.

For example, we could require that all locks in our codebase must only be grabbed using a passed-in `LockGrabber` object, and have all `synchronized` code blocks adamantly refuse to pass that LockGrabber to any of their child functions, thereby cutting off their ability to grab nested locks. Conversely, services that perform computationally-expensive operations could be constructed on a background thread and only passed to functions called from within that thread, preventing those services from blocking work on the UI thread. Problem solved!

Crap, I did it again. *Actually this still kind of sucks.*

Setting aside the fact that this will quickly result in functions with *dozens* of arguments, which are just plain ugly and bad — this approach is still putting all the onus of context-management onto programmers. It would be so easy to accidentally pass an argument where it doesn’t belong, and it wouldn’t necessarily cause a compiler error or a runtime error or a test failure. All it takes is one person to accidentally pass a `LockGrabber` to another function while holding a lock and suddenly all the carefully-crafted architectural guarantees go out the window.

This idea of moving away from implicit context and towards explicit arguments for everything is a good start, but it’s untenable without help from the compiler. Here’s an example of how a context-aware compiler could work:

The `with` and `without` functions here change the context, allowing the compiler to identify a deadlock. The syntax is inspired by Kotlin’s Context Receivers (more on that later).

Behind the scenes the compiler is basically just passing extra arguments around, but with constraints automatically enforced and without cluttering up the code. A polite compiler would even give us a helpful error message, like:

```
ERROR: Cannot call `myFunction2` without required context: `LockGrabber`

Problematic trace:
 - myFunction1 [main.kt:17]                     << has LockGrabber
 - LockGrabber::withLock [main.kt:2]
 - without(LockGrabber) [main.kt:6]             << LockGrabber removed here
 - myFunction1::$lambda1@withLock [main.kt:19]
 - call to myFunction1 [main.kt:22]             << needs LockGrabber
```

It’s not always wise to integrate new features directly into a programming language, since third-party libraries are more nimble and can compete with each-other in a sort of free market. In this case though, I believe that managing context is foundational to good software and so must be foundational to good programming languages as well — not tacked on top. The standard library should be context-aware, as should IDEs and debuggers and build tools. The complexity introduced by managing context is bad, but the complexity introduced by ignoring it is so much worse.

# Kotlin: A (partial) case study

Kotlin is arguably one of the world’s most underrated programming languages. It’s often described, by me, as “Java, but good” — which isn’t exactly a gripping headline. But looking past the long shadow cast by Java and the JVM, Kotlin is a language packed with novel ideas. In fact, though Kotlin is sometimes seen as Android’s clone of Swift, it actually *predates* Swift. (This is the part where I disclose that I work on the Android team at Google so this is a bit of a sore spot for me personally)

There are two features in Kotlin relating to context:

[**CoroutineContext**](https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html) is a runtime mechanism that allows [coroutines](https://kotlinlang.org/docs/coroutines-overview.html) to access contextual data such as their async stack trace or the current thread. It can technically be used as an arbitrary key-value store to pass other data around, but that usage cannot be checked at compile time and so is generally discouraged. I have nothing against it, but it’s not what this article is about.

**The experimental** [**Context Receivers**](https://github.com/Kotlin/KEEP/blob/master/proposals/context-receivers.md) **API** is a compile-time language feature that allows function arguments to be supplied from context rather than being passed in manually. Could this be exactly what I’ve been waiting for? It looks pretty close!

This working example prints “LOG: Hi Keaton!”

While this does make the pass-everything-as-an-argument approach more feasible, it has two major drawbacks:

*   **There is no** `**without**` **function.** I can add the user and the logger to the context with `with`, but `sayHi` has no option to call other functions without that same context being present. You win this time, deadlocks.
*   **Context is explicit.** Any function that calls `sayHi` needs to specify `UserAccount` and `Logger` as context for itself in order to forward that context to `sayHi`. The compiler isn’t smart enough to do that itself. As an example, this won’t work:

Here, `processOnAppStartup` would need to include in its context `UserAccount` and `Logger` and `Database` and `SyncApi` and anything that `someOtherHeavyComputation` needs and anything that `somethingElseAsWell` needs *and anything that any of their child functions need*. Adding a new piece of context to a function would require updating all of functions that call it, transitively. So we haven’t avoided the inevitable function-with-a-dozen-arguments problem, only glazed it with some syntactic sugar.

There’s still a big gap between my ideal solution and the reality of what’s possible with Context Receivers. I understand the hesitation. A line of code that reads “this function gets access to a Logger from *somewhere* up the call stack” is unsettlingly vague without the proper tools in place to scrutinize it. In that way, though, Kotlin is a perfect testing ground for this kind of feature. Kotlin is maintained by JetBrains, makers of IntelliJ and other IDEs. It is very much designed to be used within an IDE that can decorate its code with information that would be explicitly stated in other languages (Kotlin elides most data types, some `return` statements, and doesn’t even *have* `await` statements). Context would be just another thing to decorate.

# Addendum: Other case studies in Context

I don’t usually edit posts after I publish them, but I got a lot of interesting feedback about academic work in this area and I wanted to briefly share some of it. I’m not a programming-language researcher so I won’t attempt anything more than a high-level overview, but I will link out to some great in-depth sources.

First, the bad news: Nobody I’ve talked to knows of any real-world production-ready programming language that does what I want. The only language I’ve encountered that supports something like `without` is [Flix](https://flix.dev/), a neat little research language. Functional languages like Haskell, OCaml, and Scala have strongly-typed Effects that can do everything else, just not `without`.

In Programming Language jargon an Effect is anything a function *does* beyond just returning a value. Modifying state, eg. printing to the console or making a network call, is an Effect. Knowing when a function modifies state is half the battle, the other half is knowing when a function *reads* state. This is not an Effect, but rather a [Coeffect](https://tomasp.net/coeffects/) — although sometimes it all gets lumped together under the banner of ‘Effects’ or ‘Effect Systems’.

Examples of research languages that have explored different approaches to Effects include: [Eff](https://www.eff-lang.org/try/), [Effekt](https://effekt-lang.org/docs/concepts/effect-safety), [Flix](https://flix.dev/), and [Koka](https://koka-lang.github.io/koka/doc/book.html). There’s a gigantic list of work [here](https://github.com/yallop/effects-bibliography), although much of it takes the form of runtime libraries which I don’t find particularly helpful. After all, if “the best integration test is a good compiler”, a useful corollary would be “the worst integration test is crashing the production build and hoping somebody emails you about it”.

There are plenty of academic papers on this subject, my favorites are this [Coeffect](https://tomasp.net/coeffects/) website based on a PhD thesis and [Effects as Capabilities](https://dl.acm.org/doi/10.1145/3428194).

## Exception Handling is Context

One interesting insight I got from the research is that exception handling is basically a special case of context. A thrown Exception propagates up the call stack until it finds a `catch` block that accepts it. We could hack this system to accomplish something that looks an awful lot like context:

This example prints: “Hi!” then “HI!”

This would work for `sayHi` and any functions it calls internally, without having to pass a `Logger` around as an argument! The obvious problem is that any code after the `throw LogText()` line will never run. The other problem is that this still isn’t actually checked at compile-time. Calling `sayHi` without a catch block for `LogText` will just … crash the entire program. Oof.

Neither of those is a fundamental problem, just a language design choice. Exception handlers could have a `continue` keyword that resumes the throwing function as though it had never thrown an exception. Compilers could ensure that every thrown exception is caught somewhere — Java does this already, although it ignores RuntimeExceptions.

Personally I don’t think hacking try/catch blocks is a great way to implement context management, but it does go to show that context is not a wild new paradigm that will totally break every existing language feature and slow compilers down to a crawl. It’s just exception handling on steroids.

# Conclusion

This post is a spiritual sequel to [“Zoom Out”: The missing feature of IDEs](/source-and-buggy/zoom-out-the-missing-feature-of-ides-f32d0f36f392). Both are about the same problem: that we can see the individual pieces of our codebase but not how they fit together. Code is not just a collection of text files but also an interconnected graph of logic that is often too vast to fit within our feeble human minds. It is hubris to think we can catch all deadlocks or optimize thread utilization without some computational help.

As with everything, there are tradeoffs. My ideal solution here adds complexity to programming languages and slows down compilers. It would require developing a new set of Best Practices, which can only be identified once people have failed in every conceivable way.

The advantage is that it enables programmers to have more expressive power, in particular allowing us to constrain what future programmers are allowed to do to our codebases. *In this house, we don’t make network calls from the UI thread*. I could write that in code review guidelines but even the smartest engineers make mistakes sometimes. The only real guarantee is a compile-time guarantee.

Of course, writing a blog post is very different from solving a problem. The best case scenario is that you read this and start to think about how context applies to your work. Maybe you share it with somebody else who shares it with somebody else who writes a proposal that ends up getting incorporated into a production language. It’s not an easy path from Point A to Point B, there’s a lot of odd and improbable steps in between. The path from an idea to a product is, like so many things, a Rube Goldberg Machine.