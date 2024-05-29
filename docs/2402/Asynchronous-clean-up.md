<!--yml
category: 未分类
date: 2024-05-29 13:22:49
-->

# Asynchronous clean-up

> 来源：[https://without.boats/blog/asynchronous-clean-up/](https://without.boats/blog/asynchronous-clean-up/)

One problem with the design of async Rust is what do about async clean-up code. Consider that you have a type representing some object or operation (like an async IO handle) and it runs clean up code when you are done using it, but that clean up code itself is also non-blocking and could yield control. Async Rust has no good way to handle this pattern today.

The nicest solution seems to be to just use the mechanism that already exists: destructors. If only you could `await` inside a destructor, everything would seem to be solved. Alas, this would present several problems, and I personally do not believe it is realistic to imagine Rust gaining this feature in the same way that destructors work.

The first problem is this: what happens if you drop the the value in a non-async scope? It’s not possible to `await` there! There are two options: either the async destructor doesn’t run (considered too easy a mistake to make), or there is a type-checking rule that prevents users from dropping values with async destructors in non-async scopes. The second solution reduces to undroppable types, which I will discuss later in this post: this rule is just undroppable types with an exception to allow them to be dropped in an async scope. What I can say with certainty is that undroppable types, even with an exception, would be very difficult to add to Rust.

The second problem is the way that the state of the async destructor would impact the state of any future any containing it. This is actually a re-emergence of the problems with async methods, but now applied to any generic type (because you don’t know of a generic type `T` has an async destructor). The first problem is that you have any trait object, when it drops, what happens if it has an async destructor? This introduces the same object safety issues as async methods: you have nowhere to store the future returned by the async destructor of a trait object. The second problem is that you want to send a value to a different thread, that state of its async destructor also needs to be `Send`. This is the same problem that motivated RTN, except that now its a problem for *every* generic type being moved to another thread, not only types on which you explicitly call an async method. I wrote about this problem years and years ago, but it seems to have been misunderstood and ignored since then.

The third problem is that users are concerned about having implicit await points added to their future without them realizing it. Therefore there would need to be some restriction that not only doesn’t allow these types to be dropped in a non-async scope, but also makes it so that they are destructed at an already explicit `await` point. This would make the rules around when their async destructors run very different from other destructors, if its even possible to make them coherent.

The fourth problem, I believe maybe never raised before, is that it is not the ideal code generation to run async destructors sequentially no matter what. For example, if I have two values that I am asynchronously dropping, possibly I want to `join` the destructors so they run concurrently. But doing this implicitly would be very risky, because maybe I actually carefully expect one to run before the other.

All of these problems hint at a different way to frame the problem of asynchronous clean-up: the problem is not that there is no async drop, but that destructors really only work when you can write a destructor function that returns `()`. Async clean-up is just a special case of clean-up which does not return `()`. In this case it returns a future, but there are also scenarios in which the issue is a lack of destructors that can return `Result`, for example.

I want to explore the design space for asynchronous clean up and clean up code that returns values in general, without a focus on destructors specifically. The proposal I’ve fleshed out here, based heavily on the work of others (especially Eric Holk and Tyler Mandry), combines two distinct features - async future cancellation and a `do` &mldr; `final` construct - to enable users to write asynchronous clean up code that is consistently called. I will also show how these constructs are required for any sort of “linear type” mechanism in Rust, so rather than seeing them as alternative to type-based async clean up code, they should be seen as prerequisites that can be implemented in the nearer term.

# Cancellation

There are two reasons a future could need to clean up its state. The first would be that the future is ready, and is returning its final value. The second is that while the future was pending, the caller lost interest in it and canceled it. This section is about introducing an asynchronous cancellation mechanism, so that asynchronous clean-up can occur during cancellation.

Canceling work is one of the big issues of concurrent programming, and different concurrency systems have different approaches. One way to frame the design space is to position designs for cancellation on a spectrum of *cooperativeness*. At one end you would have *non-cooperative cancellation*, in which a unit of work can be canceled without any handling within the work unit itself. At the other end you would have *cooperative cancellation*, in which a unit of work cannot be canceled and will be run until it finishes (you can implement cancellation in such a system by the work unit returning an `Option` or a nullable type, depending on your language, and having a mechanism for your work unit to receive a message to cancel it).

Go’s goroutines are cooperatively canceled; they can’t be canceled unless they explicitly opt into cancellation. POSIX threads can be non-cooperatively canceled by sending them `SIGKILL` with `pthread_kill(3)` (EDIT: this is wrong, because sending a POSIX thread `SIGKILL` will kill your whole process; I don’t have an example of fully non-cooperative cancellation available off the top of my head.)

The problem with non-cooperative cancellation is that it might leave the program in a bad state, as that unit of work could have held locks or owned heap memory, which now will all be leaked. The problem with cooperative cancellation is that if a unit of work doesn’t opt into being canceled, it will run to completion even if its work is no longer necessary.

Between these extremes there exists a range of what you might call *semi-cooperative cancellation* mechanisms. With these mechanisms, a unit of work might be moved into a control-flow path for cancellation, without its cooperation, but it can execute some specifically designated code to “clean up” its state as it is canceled. This is intended to find a middle ground between cooperative and non-cooperative cancellation.

Rust’s async model currently adopts a semi-cooperative cancellation model in practice: during any `await` point in a future, the future might be canceled. A correctly implemented runtime will run the destructor on that future, allowing it to clean up its state. However, that cancellation code must be entirely synchronous; to support asynchronous clean-up code, what is required is some form of *asynchronous semi-cooperative cancellation*, which I will just refer to as “async cancellation.”

Some additional nuance should be noted, if this isn’t already too complex. First, async Rust only supports cancellation at `await` points: except at an `await` point, the user can be assured the future will not be canceled. This is an implication of the fact that async Rust uses *cooperative scheduling*, in which units of work cannot be preempted and only yield control back to the scheduler when they choose to. This has its own pros and cons. Second, though cancellation in async Rust is semi-cooperative in practice, code authors cannot rely on cancellation for soundness: it is not undefined behavior to cancel a future without running its clean up code. This will we return to in a later section of the post.

## `poll_cancel`

To support async cancellation, the traits for async units of work need to gain an API for canceling them asynchronously. This is `poll_cancel`, as in this change to the definition of `Future`:

```
trait  Future  {  type Output;   fn poll(self: Pin<&mut  Self>,  cx: &mut  Context<'_>)  -> Poll<Self::Output>;   fn poll_cancel(self: Pin<&mut  Self>,  cx: &mut  Context<'_>)  -> Poll<()>  { // By default, futures do not have any async cancellation code Poll::Ready(()) } } 
```

Eric Holk discussed this new API at length in a great [post](https://theincredibleholk.org/blog/2023/11/14/a-mechanism-for-async-cancellation/) last year. Some notes on this from my perspective follow.

First, I would not immediately support an `on_cancel` extension to `Future`: I think that it does not make sense for ordinary async Rust (not written in the lower level poll register) to have clean-up that *only* runs on cancellation, and not whenever it finishes. I think there’s a risk of bugs here, because if a user writes a loop that never terminates in their cancellation code, the task will never actually be canceled. Then anything which cancels them non-concurrently, like `select!`, would also never terminate. Because this code only runs on cancellation, it would not be as likely to be tested for bugs like this. I would be interested in examples of code that users believe require cancellation-specific async code, though. Later in this post I’ll show a way to support async clean-up code that runs on both cancellation and normal completion.

This also alleviates the need for considering any sort of question about “what happens if you cancel the cancellation future,” and whether that is recursive or idempotent: once a future begins canceling, “canceling” it again is idempotent, because its already canceling; there is no second future to cancel.

Second, a critical addition would be to add `poll_cancel` to `AsyncIterator` as well. This would allow `AsyncIterator`s to also support async clean up on cancellation (including async generators). As a result of this, the code generation of `poll_cancel` on an async block or function would be more complex. First, it would call `poll_cancel` on whatever future was being awaited, if it was awaiting a future. Then, it would walk backwards to call `poll_cancel` on every `AsyncIterator` it was looping over with `for await`, canceling them as well.

Once these APIs are stabilized in the minimum supported Rust version of the frameworks, those frameworks should begin calling `poll_cancel` on any futures that they cancel. In this way, async Rust will come to support async cancellation on any runtime updated to the version which supports this mechanism. Though it won’t be an iron-clad guaranteed, in the same way that destructors are not an iron-clad guarantee, if you’re using a supportive version of any good runtime, your futures will consistently run their cancellation code.

## Async unwinding

Eric Holk touches on unwinding in his blog post, but I think there are some mistakes in his remarks. Here is my attempt to describe async unwinding behavior.

The unwinding path for a future would need to also call `poll_cancel`. The only complication that arises is what to do about calling `Pending`.

A new API, an async version of `catch_unwind`, would be added to the standard library. When this catches an unwinding that returned `Pending`, it would itself return `Pending` and unset the panic state. When it is polled again, it sets the panic state and polls itself, proceeding further through the `poll_cancel` method of the future it wraps. Eventually, `poll_cancel` returns ready, at which point the async `catch_unwind` returns with a panic error.

If `poll_cancel` returns `Pending` while unwinding not inside of the async `catch_unwind`, one of two choices can be made: either the entire process aborts, or control jumps immediately to the innermost `catch_unwind` without executing any further unwinding code. Probably the former is the safer choice.

In addition to upgrading to handle cancellation, runtimes would need to upgrade to use the asynchronous version of `catch_unwind`.

# `do` &mldr; `final`

The second step to make async clean up work is totally separate from supporting async cancellation: it’s a way to do clean up other than destructors. This will solve several problems: first, it will solve the general “clean up with meaningful return values” problem (whether they return `Result` or `Future`) discussed earlier. Second, it will provide a way to do ad hoc clean up without the rigamarole of creating a custom guard type with a destructor, which code that needs to do one-off ad hoc clean up currently needs to do.

The basic concept is to add a new kind of block construct, which has a way to specify code that runs whenever that block exits. I’ve chosen the syntax `do` &mldr; `final`, because both of these are already reserved words without meaning in Rust. You can easily imagine other syntaxes: Go’s `defer` blocks are basically the same feature, but with less of the block structuring of this syntax.

For example:

```
do  {  if  fallible_call()?  { println!("successful and true") }  else  { panic!("successful but true") } }  final  {  println!("exiting block") } 
```

This code could exit the `do` block in 3 different ways: it could exit normally, `fallible_call` could return an error, or the block could panic. In all cases, the final block will be run before proceeding, so it will print the message `"exiting block"` whatever happens.

The `final` block must always evaluate to `()`, and the entire construct will evaluate to the type that the `do` block evaluates to.

This reduces the pattern of “ad hoc guards” to relatively simple constructs. Consider this example:

```
foo.bar(); closure(&mut  foo)?; foo.unbar(); 
```

If the user’s intent is to call `unbar` every time the closure exits, they will fail: if the closure exits by panicking or by returning an error, `unbar` will not be called. For this reason, ad hoc guard types are used in that scenario:

```
struct Guard<'a>(&'a  mut  Foo);   impl  Drop  for  Guard<'_>  {  fn drop(&mut  self)  { self.0.unbar(); } }   let  mut  guard  =  Guard(&mut  foo); guard.0.bar(); closure(&mut  guard.0)?; drop(guard); 
```

This is quite a bit of code to have to add for this case. `do` &mldr; `final` blocks will make the whole thing simpler:

```
foo.bar(); do  {  closure(&mut  foo)?; }  final  {  foo.unbar(); } 
```

They’re also helpful for handling errors in fallible clean up. Consider the case of closing a file:

```
let  mut  file  =  File::open(path)?; operate_on_file(&mut  file); drop(file); 
```

On a UNIX system, the file is closed in the call to `drop` by calling `close(2)`. What happens if that call returns an error? The standard library ignores it. This is partly because there’s not much you can do to respond to `close(2)` erroring, as a comment in the standard library elucidates:

```
// Note that errors are ignored when closing a file descriptor. The // reason for this is that if an error occurs we don't actually know if // the file descriptor was closed or not, and if we retried (for // something like EINTR), we might close another valid file descriptor // opened after we closed ours. 
```

“Worse is better.”

However, a user may still want some level of control over what happens in this case, even if just to log that an error occurred (which can be used to assist in debugging any incident that may arise). Toward that end, `File` could also have a close method which returns a `Result`. Users would then be able to use `final` blocks to call that method and handle the error however they like, should it arise:

```
let  mut  file  =  File::open(path)?; do  {  operate_on_file(&mut  file)?; }  final  {  if  let  Err(err)  =  file.close()  { info!("Error occurred closing file at {path}: {err}"); } } 
```

## Awaiting in `final` blocks

Both of these use cases are completely separate from async clean up, but the feature is also useful for async clean up, because `final` blocks would be able to contain `await` expressions. For example:

```
do  {  process_messages(&mut  socket).await?; }  final  {  socket.shutdown_graceful().await; } 
```

The `poll_cancel` implementation for async scopes would also execute the `final` blocks for any `do` blocks that the future is currently within, in addition to the `poll_cancel` for the future being awaited and any async iterators being looped over.

Thus, between async cancellation and `do` &mldr; `final`, a method of performing async clean up is achieved. The limitation of this, in contrast to async destructors, is that it is not tied to specific types: a socket type cannot guarantee that it will always be shutdown gracefully. This will be addressed in the later section on linear types.

Before moving on to linear types, I have some notes on other control flow operators and their relationship to `final` blocks. There may be errors of reasoning in this section, I’m not confident I’ve correctly thought through the control flow possibilities.

## Early exit in `final` blocks

One particular quirk of `final` blocks will be what to do if they early exit with a value, because the user might *already* be early exiting with a value. Consider:

```
do  {  read(&mut  file,  &mut  buffer)?; }  final  {  close(&mut  file)?; } 
```

If `read` returns an error and we enter the `final` block, what happens then if close also returns an error? We can’t return *both* errors. There are only two options that I can imagine:

1.  Disallow early exit in `final` blocks. This is the most restrictive, but leads to no confusing scenarios.
2.  Allow early exit in `final` blocks, but drop the value if we are already early exiting. Only the original error will be returned.

In the latter scenario, you would continue to execute any other `final` blocks further down even if this one early exited; just the remainder of this `final` block would not be executed.

## Yielding from `final` blocks

If early return from `final` blocks is permitted, note that `yield` could also be permitted. Consider:

```
gen  {  do  { for  elem  in  iter  { if  elem.has_foo()?  { yield  "elem has foo"; } } }  final  { yield  "final yield"; } } 
```

If `has_foo` never fails, this will yield `"elem has foo"` as many times as an `elem` in `iter` “has foo” and then one `"final yield"`. If it fails, it will return `"elem has foo"` every time before that, then enter the `final` block and yield the `"final yield"` value.

## Unwinding through `final` blocks

Some special attention is needed for figuring out what happens when unwinding through `final` blocks that include control flow other than the ordinary control flow.

For `await`, this is already covered by the discussion of async unwinding: if you are wrapped by an async `catch_unwind`, it will proceed as normal, otherwise it will either abort or stop unwinding.

If `yield` and early `return` operators are permitted in `final` blocks, these would need to escape this `final` block but drop the item. We would proceed with unwinding, skipping the remaining code inside of this particular `final` block. If it was a `yield`, we would not yield that value, instead we would exit the loop and continue unwinding.

# Linear types

With async cancellation and `do` &mldr; `final` blocks, asynchronous clean-up is possible, but it can not be guaranteed that any particular asynchronous clean-up code will be run when a type goes out of scope. It’s already the case today that you can’t guarantee clean-up code runs when a type goes out of scope, asynchronous or not. There are two possible solutions to this problem, though they often are conflated under the single term “linear types,” so I’m going to refer to them with two distinct names.

The first solution would be to allow types to express as part of their contract that their destructor (remember, their normal, non-asynchronous, non-value-returning destructor) runs whenever they go out of scope. I call these *unforgettable types*. It would require adding a `Leak` auto trait and bounding any API which can cause a leak (like `mem::forget`) with that auto trait.

The second solution would be to allow types that cannot be dropped at all. This is actually what “linear types” means outside of the context of Rust, but I am going to call these *undroppable types* to avoid any confusion. Niko Matsakis has [written](https://smallcultfollowing.com/babysteps/blog/2023/03/16/must-move-types/) about adding this functionality under the term “must move types,” because such types must be moved, they cannot be dropped. I’m going look briefly at each of these as a solution to the problem, without considering the challenges of adding them to Rust or how they would impact other use cases.

## Unforgettable types

This weaker limitation would not allow defining types with functionality like async destructors, but it would allow solving the [scoped task trilemma](/blog/the-scoped-task-trilemma): the scoped task API would return a future which is `!Leak`, guaranteeing that its destructor will run.

In practice, the API would ensure that it is asynchronously cleaned up almost all of the time, to a similar degree that even without unforgettable types, right now destructors almost always run. The async `scope` function would look something like this:

```
pub  fn async  scope<'env,  F,  T>(f: F)  -> T where  F: for<'scope>  async  FnOnce(&'scope  Scope<'scope,  'env>)  -> T {  let  scope  =  ...; do  { f(&scope).await }  final  { scope.await_all_tasks().await; } } 
```

Because the `Scope` type would be `!Leak`, the future returned by this function would also be `!Leak`. If `poll_cancel` on that future is not called, the synchronous destructor will run: that will act as an “emergency backstop,” blocking this thread and ensuring memory safety. But as long as this is executed with a correctly implemented runtime, instead the asynchronous clean up code will run, ensuring that all the child tasks are awaited without blocking the thread.

## Undroppable types

Undroppable types allow expressing a wider variety of contracts in the types, but require more code to use correctly. This would involve adding the ability to define a type that does not implement `Drop`, and so cannot be dropped (this would be different from today where types that don’t implement `Drop` can still be dropped; remember we’re avoiding discussing the migration story in this post).

Whereas adding `Leak` impacts only a handful of APIs, `!Drop` would impact a large many more: any time you drop any generic value in any code path of that function, that type would need to be bound by `Drop`.

Because undroppable types cannot be dropped, the only way to get rid of them is to destructure them. If they have private fields, they can only be destructured by calling a public method which destructures them internally; this lets the author of an undroppable type provide one or more clean up methods which *must* be called: users cannot forget to call it, because it will result in an error.

For example, an undroppable `File` type could expose a `close` method, which is the only way to get rid of that `File`. Users would get an error if they did not remember to close the `File` explicitly in their code.

Niko Matsakis proposes this as a way to implement async destructors: there would be some sort of `AsyncDrop` trait; types which are meant to have an async destructor would implement this trait but not `Drop`. Then there would be an `async_drop` function which you are meant to call explicitly on these types to run the clean up code. This is the only proposal for “async destructors” that I believe avoids all of the problems I listed earlier: instead of implicit async destructors, you use undroppable types that require you to explicitly call their async clean-up code.

In the scoped task API shown before, the `await_all_tasks` method would be the *only* way to destroy the `Scope` type, which would not implement `Drop`. This ensures that this method is ultimately called, solving the scoped task trilemma.

## Async cancellation and `do` &mldr; `final` are prerequisite

There was one issue with Niko Matsakis’s proposal about how undroppable types could be used for asynchronous clean up. Consider his description:

> The simplest way to achieve “async drop” then would to define a trait `trait AsyncDrop { async fn async_drop(self); }` and then make the type “must move”. This will force callers to eventually invoke `async_drop(x).await`. We might want some syntactic sugar to handle `?` more easily, but that could come later.

What Matsakis fails to mention in this comment is the problem of the future returned by `async_drop` itself: what if you drop *that future*? Now the async destructor isn’t run to completion, violating the contract. The implication is that the future returned by `async_drop` must itself not implement `Drop`.

But how would you cancel a future which is not `Drop`? You would need an async cancellation path which evaluates the destructor. If a runtime wants to support tasks that have async destructors, it would have to guarantee that it either polls all tasks to completion, or if they are canceled it runs their async cancellation path to completion.

In other words, `poll_cancel` is a prerequisite of making undroppable types work. And when Matsakis alludes to “syntactic sugar to handle `?` more easily,” that is what `do` &mldr; `final` is meant to do: it allows you to hold an undroppable type across any code block that has an early return or a panic in it.

These are also prerequisites for unforgettable types, because you need a combination of async cancellation and a `do` &mldr; `final` block to make the scoped task API asynchronously await all of the scoped tasks by default, without resorting to the blocking emergency backstop.

What this means is that the other features in this post would also be prerequisites for either undroppable or unforgettable types. Because either definition of “linear” types would be a massive change to Rust, and one that seems unlikely to happen before 2027, the immediately actionable thing to do would be to add async cancellation and `do` &mldr; `final`, which will allow users to perform some sort of async clean up in the near term even without linear types.