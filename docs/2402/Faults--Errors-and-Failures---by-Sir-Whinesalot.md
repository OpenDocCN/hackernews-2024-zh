<!--yml
category: 未分类
date: 2024-05-27 14:56:47
-->

# Faults, Errors and Failures - by Sir Whinesalot

> 来源：[https://btmc.substack.com/p/you-cant-handle-errors](https://btmc.substack.com/p/you-cant-handle-errors)

Having somewhat recently become a father (one of the reasons my writing has slowed down to a crawl), I’ve had to deal with a very particular kind of error output: *crying*.

From a computational perspective, it’s about equivalent to the following:

Loud and unspecific. Well, at least with a baby there’s a limited set of reasons for crying, so fixing the issue amounts to tackling each possible cause until something works. Good luck debugging the error message above though.

That got me thinking, how does one end up with a useless error message like the above? What facilities should programming languages provide to make it easy for developers to handle errors properly and conveniently?

Originally this post was going to be a survey on error handling approaches, similar to my post on memory management approaches, but I decided against it in the end because I think looking at this problem from “outside the box” is important. Maybe someone can come up with a better way to “handle errors”.

Lets starts from the beginning. What do we mean by *error*? Unfortunately the term “error” is not used consistently in the literature, but we can use 3 related, commonly used terms from the study of fault tolerance to distinguish different meanings for the word error**:**

*   A **fault** (known colloquially as a *bug*), is static defect in software, meaning there’s some line (or lines) of code that are incorrect and will likely result in…

*   An **error**, which is an *unobserved*, incorrect internal state that will likely result in…

*   A **failure**, which is *observed*, incorrect behavior with respect to the software’s expected behavior.

For example, consider the following piece of C# code:

```
var a = new int[]{1,2,3,4,5};
for (int i = 0; i <= 5; i++) {
  Console.WriteLine(a[i]);
}
```

There’s a *fault* in the loop stop condition (it should be `i < 5`), which will result in an *error* state of `i = 5`, which will trigger a *failure* when used to index the vector (throwing an IndexOutOfRange exception).

**The error is not the exception (failure) nor is it the incorrect loop condition (fault).**

It is critical to understand this distinction for the rest of the article to make sense. Before tackling error *handling*, I’ll need to talk about error *prevention*, as the two often get mixed up, and it’ll help clarify what I mean.

Testing, static type systems, model checking, sanitizers, fuzzers, and even certain language features like `foreach` loops (which would trivially avoid the problem above) are ways to indirectly *prevent* errors by either detecting or preventing *faults*.

I’m 100% pro all of the above, they’re all great. If you’re not using them and you could, then you should. Your users will thank you. Your colleagues will thank you. Your future self will thank you.

Errors are almost always the result of faults. Barring cosmic rays, hardware issues or really unusual race conditions between the application and the operating system, if an error occurs it is because *the programmer screwed up* and introduced a bug.

Some languages like Rust and Haskell have a reputation that “if the code compiles, it works”. They get this reputation because they excel at preventing common faults through a combination of a powerful static type system plus a culture of modeling function domains and codomains as accurately as possible.

Consider the following function declaration (in Rust syntax):

```
fn head(v: Vec<i64>) -> i64
```

The head function takes as input a vector of signed 64-bit integers and returns the first integer in that vector.

This function’s domain is the set of all vectors of signed 64-bit integers and its codomain is the set of signed 64-bit integers. But this function declaration is “lying”, either about its domain, or about its codomain, depending on the point of view.

There is a hidden pre-condition that `v` is not empty. If `v` is empty, the function will panic.

A “truthful” head function would have a different domain or codomain:

```
fn head(v: NonEmptyVec<i64>) -> i64
fn head(v: Vec<i64>) -> Option<i64>
```

In the first case, we tighten the domain, in the second case we loosen the codomain. Either way, we’ve made the function less likely to result in faults — by reminding the programmer of special cases they must take into account — at the cost of making it more annoying to use.

In the first case the caller must prove they have a NonEmptyVec by calling some explicit conversion function, while in the latter case the caller must always handle the “None” case even if they know for a fact that the vector is not empty.

If multiple properties are desired at the same time (e.g., expecting a non-empty even-length vector) the “truthful domain” approach quickly collapses without access to much more powerful type system features like [dependent types](https://en.wikipedia.org/wiki/Dependent_type) or [refinement types](https://en.wikipedia.org/wiki/Refinement_type), which in turn add a massive amount of complexity to the language and are, IMO, not worth it.

The codomain can always be loosened with a single additional “invalid input” case, and various language facilities can be added to conveniently deal with this extra case, making it the better solution most of the time. For example, in C# nullable types have a great deal of features to make them as convenient to use as possible:

*   flow typing, which propagates the result of null checks (i.e. an `int?` variable becomes `int` for as long as the result of the check is valid)

*   null-coalescing (`??` and `??=`) operators that make it convenient to replace a null value with an alternative value from the non-nullable type.

*   null-conditional (`?.` and `?[]`) operators that allow “[flat-mapping](https://en.wikipedia.org/wiki/Monad_(functional_programming)#Definition)” operations from the non-nullable type into the nullable type.

The idea is to keep the advantage of truthful codomains (reminding the programmer to handle special cases) while mitigating the disadvantages (inconvenience).

Even with as many fault prevention measures in place as possible, errors will always happen. Fault prevention amounts to ensuring known pre-and-post-conditions are properly handled, it can’t help with unexpected logic errors or lacking requirements.

Unfortunately once an error occurs, it can’t be handled directly. Remember, an error is an *unobserved* incorrect state. The moment you *observe* an error, it has already turned into a *failure*.

Consider for example a set of additions and subtractions applied to a variable where intermediate computations result in overflow but the final result does not. The intermediate overflowed values are *errors*, but there is no resulting *failure*.

If you actually checked for overflow each operation, you’d detect the error, triggering a *failure* and causing a panic or equivalent. And if, instead, you checked all the inputs to make sure they wouldn’t ever overflow, you’d be preventing a *fault*. Conflating faults and failures, I believe, has led to some really shitty language features that cause more faults and failures than they solve (e.g., **exceptions**, more on those in a bit)

So errors cannot be handled, only failures can, and a failure is an observed incorrect behavior of the program. If the program is behaving incorrectly, what can you do about it?

First, how does the program know it is behaving incorrectly? If the program can know it is behaving incorrectly, couldn’t the fault be prevented in the first place? ***Yes***. But for various reasons the cost of doing so may be too high.

An example would be needing to check after every arithmetic operation for overflow. I don’t mean the compiler inserting checks and triggering a failure, I mean the programmer explicitly checking for overflow after every arithmetic operation and handling that “special case” each and every time. Extremely annoying.

Alternatively, the overflow-related faults could be prevented by using [arbitrary-precision arithmetic](https://en.wikipedia.org/wiki/Arbitrary-precision_arithmetic), which would instead have a computational cost and may result in a different kind of failure (out-of-memory).

Compiler (or programmer) inserted checks are a means of computationally observing failures caused by unlikely (but otherwise expected) errors, in turn caused by faults that would be excessively costly to prevent. The most common example is array bounds checking.

Note that these checks are **not** failure handling, they are a necessary step to detect the failure but actually handling it comes afterwards.

The most common response to observing a failure is to throw an exception. Exceptions unwind the stack until they hit a programmer-specified handler (i.e., a `try/catch` block) or a default handler that crashes the program and usually prints out a [stack trace](https://en.wikipedia.org/wiki/Stack_trace) to help debug the problem.

The second most common approach is to abort the program, by sending it a signal that more often than not is caught by a default signal-handler, which will terminate the program and output a “useful” message like [Segmentation Fault](https://en.wikipedia.org/wiki/Segmentation_fault) (which is really a type of failure as defined above, don’t you hate inconsistent naming conventions?)

Rust panics can be set to use an exception-like mechanism, to terminate the current thread, or to abort the program (sending it a signal as above).

You’ll notice that all of the possibilities just crash the program by default. Before I discuss why, I need to point out that the following is *not* failure handling:

```
try {
  File f = new File("filename.txt");
  ...
}
catch (FileNotFoundException e) {
  ...
}
```

In my opinion this is just a weird looking conditional for one particular “output value” of the File constructor. The “truthful” codomain of the File constructor includes additional cases that are “returned” as exceptions.

Would you ever write code like the above to handle an array out of bounds situation? Right after you tried to index an array? What about division by zero? No, right?

Forgetting to handle a missing file is a fault, and catching that “output” is *not having that fault*. Using exceptions instead of properly modeling the function’s codomain has increased the likelihood of a fault and its corresponding failure by not reminding the programmer to handle a common failure case.

I really dislike exceptions for mixing up these unrelated concerns.

If you allow encoding different “error types” into your failure handling feature you’ve probably already screwed up. Let me explain.

Let me put it nice and clear:

Handling a failure means returning the program to a known, correct state.

Remember the pipeline: Fault → Error → Failure. A fault is a bug in the program’s source code which causes the program to enter an incorrect unobserved state, which can then lead to a failure, which is observed incorrect behavior.

The job of a failure handler is to *get rid of the error*. Note that it’s not handling the error, what is being handled is the failure, the error that caused it is unknown. But the goal is, nonetheless, to get rid of the error, somehow.

Consider an IndexOutOfBounds exception. If one happens, it’s because there is a bug in the program that resulted in some variable being set to an incorrect value, which then resulted in the observed incorrect behavior of trying to index an array out of bounds. What should the failure handler do?

First, a true failure handler won’t be anywhere near the actual index out of bounds situation, because if it was, you were just handling one of the possible “return values” of the indexing operation (preventing a fault), not actually handling a failure.

In a language with [algebraic effect handlers](https://en.wikipedia.org/wiki/Effect_system), the handler for the index out of bounds situation could “fix” the failure by resuming the program with a made up value for that index. Terrible idea, essentially replacing a failure with a new error. With exceptions you can’t even do that.

No, a true failure handler is a piece of code that a programmer hopes never actually has to run! It’s the last line of defense. All that the failure handler knows is that some variable (no idea which) got set to a wrong value at some point (no idea where) that was then used to wrongly index an array (knows where, but can’t do anything about it).

Given the above, IndexOutOfBounds might as well have been PoopExplosion9000 as far as the failure handler is concerned. The information is useful for the programmer, as is the stack trace, but it could just as well have been a text string in the assertion error message. The actual type of the exception is utterly useless for the purpose of failure handling (not for ghetto codomains, but you shouldn’t use exceptions for that in the first place).

While you can only handle failures (because only the failure is observed) getting rid of the failure by itself doesn’t do much good, there’s still the unobserved invalid state that led to it in the first place. But you can’t do anything about that invalid state directly, since you have no idea what it is or where it originated from.

This is why the default handler for any sort of panic mechanism (exceptions, signals, etc.) is a full-on crash that spits out as much information as it can for the programmer to debug with. There’s nothing else it can do.

The only thing you can do is turn it off and on again.

Every failure handler that’s any better than the default is ultimately just decreasing the scope of what’s getting restarted or improving the usefulness of the info-dump.

The choice of panic mechanism (exceptions, signals, terminating threads) essentially dictates the scope of what can be restarted. If restarting wasn’t the goal, then there would be no need for any mechanism beyond terminating the program outright.

Exceptions let you restart at an arbitrary point in a function. This is less useful than it sounds because the error state may have occurred outside the handler’s restart point. You can only safely restart pure functions or those that work like [transactions](https://en.wikipedia.org/wiki/Transaction_processing).

Restarting threads (particularly worker threads) is one of the best approaches if the threads don’t share mutable state. Erlang (and by extension Elixir) is built entirely around this idea, using [supervision trees](https://erlang.org/documentation/doc-4.9.1/doc/design_principles/sup_princ.html). Because Erlang was designed with failure handling in mind, it is an excellent fit for high-reliability systems.

The last case, sending a signal to a program, may sound like it doesn’t leave much room for “restarting” but that depends heavily on how the software works. If the program backs-up the users work every second to disk, then you can go back to a “known, correct state” by restarting the whole program and instructing it to load the user’s backed up work. You can also use this approach in a multi-process architecture.

Another software architecture that works well for restarting is the [Elm Architecture](https://guide.elm-lang.org/architecture/), since each “update” step can be cancelled. Alas, Elm itself lacks a nice failure handling mechanism to take advantage of this. Trying to avoid failures at all costs results in [this sort of nonsense](https://github.com/elm/core/issues/1072).

In all cases the hope is that the triggering of the fault is an uncommon occurrence, otherwise the program will end up in a pointless restart loop.

Unfortunately errors due to logic bugs may not result in a computationally observable failure. Think glitched out physics simulations for example. Just gotta wait for the user complaints to show up.

Faults are bugs in the source code that lead to errors (unobserved invalid states) that wreck havoc until they trigger a failure (observed invalid behavior).

Many faults are easily preventable mistakes while others are too costly or annoying to prevent. Languages and tools that avoid or detect such mistakes are good.

You can’t handle errors directly, you can only handle their corresponding failures.

Handling a failure always means restarting (part of) the program to get rid of the error, otherwise it’s not really handling a failure, it’s just working with an extra known “return value” of a function.

You should architect your software in a way that allows for proper failure handling. Some languages (like Erlang) have excellent support for this. Others like Elm think it isn’t necessary (I disagree).

Exceptions are used as both a way to model extra “return values” of functions and as a failure handling mechanism, leading them to be lousy at both. Exceptions suck.

* * *

**Side-Note:** I didn’t mention it within the text because it’s already too big and an incoherent mess, but there’s another nice approach to preventing faults than expanding a codomain with a “must handle invalid result” case to remind the programmer of the potential fault.

You can also expand the codomain with an extra “harmless” result. A terrible idea for a library since it can’t possible know what such a harmless result would look like for the caller, but a perfectly valid approach for an application.

For example, lets say your program loads up a bunch of textures at the start. Instead of the texture loading function returning an `Option<Texture>` that you then need to deal with everywhere, it could return just `Texture`, but use a special “Missing Texture” for textures that failed to load.

The function would log somewhere which textures were missing, and just let the program proceed as normal, still letting the user do some work (if they didn’t need those textures). The program can check that errors were logged at some point and display them to the user in a separate codepath, while fixing the missing textures can be done by the user some other time. Check [this post by Ryan Fleury](https://www.rfleury.com/p/the-easiest-way-to-handle-errors) on the subject.