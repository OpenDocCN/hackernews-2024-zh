<!--yml
category: 未分类
date: 2024-05-27 14:58:21
-->

# FuturesUnordered and the order of futures

> 来源：[https://without.boats/blog/futures-unordered/](https://without.boats/blog/futures-unordered/)

In my [previous post](/blog/let-futures-be-futures), I wrote about the distinction between “multi-task” and “intra-task” concurrency in async Rust. I want to open this post by considering a common pattern that users encounter, and how they might implement a solution using each technique.

Let’s call this “sub-tasking.” You have a unit of work that you need to perform, and you want to divide that unit into many smaller units of work, each of which can be run concurrently. This is intentionally extremely abstract: basically every program of any significance contains an instance of this pattern at least once (often many times), and the best solution will depend on the kind of work being done, how much work there is, the arity of concurrency, and so on.

*   Using **multi-task concurrency**, each smaller of work would be its own task. The user would spawn each of these tasks onto an executor. The results of the task would be collected with a synchronization primitive like a [channel](https://tokio.rs/tokio/tutorial/channels), or the tasks would be awaited together with a [JoinSet](https://docs.rs/tokio/latest/tokio/task/struct.JoinSet.html).

*   Using **intra-task concurrency**, each smaller unit will be a future run concurrently within the same task. The user would construct all of the futures and then use a concurrency primitive like [join!](https://docs.rs/tokio/latest/tokio/macro.join.html) or [select!](https://docs.rs/tokio/latest/tokio/macro.select.html) to combine them into a single future, depending on the exact access pattern.

Each of these approaches has its advantages and disadvantages. Spawning multiple tasks requires that each task be `'static`, which means they cannot borrow data from their parent task. This is often a very annoying limitation, not only because it might be costly to use shared ownership (meaning `Arc` and possibly `Mutex`), but also because even if it isn’t going to be problematic in this context to use shared ownership,   (I’d love to see this change! Cheap shared ownership constructs like `Arc` and `Rc` should have non-affine semantics so you don’t have to call clone on them.)

When you join multiple futures, they *can* borrow from state outside of them within the same task, but as I wrote in the previous post, you can only join a static number of futures. Users that don’t want to deal with shared ownership but have a dynamic number of sub-tasks they need to execute are left searching for another solution. Enter [FuturesUnordered](https://docs.rs/futures/latest/futures/stream/struct.FuturesUnordered.html).

`FuturesUnordered` is an odd duck of an abstraction from the futures library, which represents a collection of futures as a `Stream` (in std parlance, an `AsyncIterator`). This makes it a lot like tokio’s `JoinSet` in surface appearance, but unlike `JoinSet` the futures you push into it are not spawned separately onto the executor, but are polled as the `FuturesUnordered` is polled. Much like spawning a task, every future pushed into `FuturesUnordered` is separately allocated, so representationally its very similar to multi-task concurrency. But because the `FuturesUnordered` is what polls each of these futures, they don’t execute independently and they don’t need to be `'static`. They can borrow surrounding state as long as the `FuturesUnordered` doesn’t outlive that state.

In a sense, `FuturesUnordered` is a sort of hybrid between intra-task concurrency and multi-task concurrency: you can borrow state from the same task like intra-task, but you can execute arbitrarily many concurrent futures like multi-task. So it seems like a natural fit for the use case I was just describing when the user wants that exact combination of features. But `FuturesUnordered` has also been a culprit in some of the more frustrating bugs that users encounter when writing async Rust. In the rest of this post, I want to investigate the reasons why that is.

# Two patterns with `FuturesUnordered`

The kind of bugs you can get with `FuturesUnordered` depend on how you use it.

The simplest way to use `FuturesUnordered` is to fill the collection with futures and then collect all of their results and process them then. This is usually not a problematic usage. However, often you want to be able to do one of two things while your awaiting all of those futures: on the one hand you want to be be able to push more work into the `FuturesUnordered`, and on the other hand you want to process each result as it comes in rather than waiting for all of them to finish.

There are two main patterns for providing those additional affordances on top of `FuturesUnordered`:

*   **The “buffered stream” pattern:** Here, you represent your work to be done as a stream of futures, and then buffer them using something like the [buffered](https://docs.rs/futures/latest/futures/stream/trait.StreamExt.html#method.buffered) adapter on `StreamExt`. More work can get pushed into the `FuturesUnordered` as there is more work to be done and as space becomes available, and each result can be processed as it becomes ready.

*   **The “scoped task” pattern:** Here, you represent your work to be done as tasks “spawned” onto the `FuturesUnordered` using some kind of handle; an example of this sort of API is Niko Matsakis’s [moro](https://crates.io/crates/moro) library. You are free to spawn more tasks from any of those spawned tasks. If you want to process the result of any of those tasks, you await its join handle or transfer that data to another task using a synchronization primitive.

There are several differences between these two approaches. One is how each of them approach backpressure. It is important when designing systems to have some way of applying backpressure if another component of the system produces more work than this component can handle. Otherwise, work will continue to build until this component becomes hopeless behind or becomes overwhelmed and crashes.

The buffered stream API enables backpressure by encouraging the user to limiting the number of futures that will be buffered at once; this applies pressure on the underlying stream generating the futures, as it will no longer be polled when the maximum number of concurrent futures are running. In contrast, the scoped task pattern as implemented in moro does not have an affordance for applying backpressure: the user is permitted to spawn as many tasks as they want. One could imagine a version of the scoped task pattern which *did* apply backpressure in this way: the `spawn` method would itself be async, and only return ready when it was able to spawn the task. But I don’t know of any library that does this; when using a spawn-like API users are expected to introduce backpressure via some other mechanism, such as bounding the length of a channel.

Another difference is how each pattern handles processing the results of the concurrent sub-tasks. Buffered streams transform a stream of futures into a stream of those futures’ outputs: they are processed by looping over the stream. Scoped tasks on the other hand have no particular organization in how the results of the sub-tasks are processed: the user is required to “link” sub-tasks together by awaiting their `JoinHandle`s.

The result of this is that the buffered stream approach imposes a particular kind of sequential order of events. The underlying stream is only polled when the `Buffered` stream has space for more work & and the `Buffered` stream is only polled when the loop which is processing the buffered stream is ready to process another result. The point of the [poll_progress](/blog/poll-progress) API change is to enable the `Buffered` stream to be executed concurrently with the loop; this would eliminate the second sequencing point. But nothing can be done about the first sequencing point: the entire purpose of that sequencing point is to apply backpressure. Without the `poll_progress` change, the sequencing of buffered streams looks like this:

```
 ┌ WAITS FOR SPACE IN ───┐    ┌ WAITS TO BE PROCESSED BY ─┐ ╔═══════════════╗           ╔════▼═════════╗           ╔══════════▼════╗ ║               ║▐▌         ║              ║▐▌         ║               ║▐▌ ║   STREAM OF   ║▐▌         ║   BUFFERED   ║▐▌         ║   FOR AWAIT   ║▐▌ ║    FUTURES    ║▐▌         ║    STREAM    ║▐▌         ║      LOOP     ║▐▌ ║               ║▐▌         ║              ║▐▌         ║               ║▐▌ ╚════════▲══════╝▐▌         ╚═════════▲════╝▐▌         ╚═══════════════╝▐▌ ▀▀▀▀▀▀▀▀│▀▀▀▀▀▀▀▀▘          ▀▀▀▀│▀▀▀▀│▀▀▀▀▀▀▘          ▀▀▀▀▀▀▀▀▀▀│▀▀▀▀▀▀▘ └─ BUFFERS FUTURES FROM ┘    └─── PROCESSES RESULTS FROM ┘ 
```

Each phase of this process is interspersed with the others, but they are not fully concurrent. When one phase is executing, even if it has no more work to do, the other phases cannot proceed.

The scoped task approach on the other hand does not in itself introduce any sequencing points. All of the futures spawned as “scoped tasks” are executed concurrently. The user must apply sequencing themselves by awaiting the results of other tasks. For example, imagine a scenario in which a user using the scope task pattern spawns two sub-tasks and joins them, one of which also spawns another sub-task and awaits it. That would look like this:

```
 ╔═══════════════════════════════════╗ ║          SCOPED TASK SET          ║▐▌ ║                                   ║▐▌ ║           ╔════════╗              ║▐▌ ║           ║  TASK  ║▐▌            ║▐▌ ║           ╚════════╝▐▌            ║▐▌ ║            ▀▀▀│▀▀▀▀▀▀▘            ║▐▌ ║       ┌─── JOINING ───┐           ║▐▌ ║       │               │           ║▐▌ ║   ╔═══▼════╗       ╔══▼═════╗     ║▐▌ ║   ║  TASK  ║▐▌     ║  TASK  ║▐▌   ║▐▌ ║   ╚════════╝▐▌     ╚════════╝▐▌   ║▐▌ ║    ▀▀▀▀│▀▀▀▀▀▘      ▀▀▀▀▀▀▀▀▀▀▘   ║▐▌ ║     AWAITING                      ║▐▌ ║        │                          ║▐▌ ║    ╔═══▼════╗                     ║▐▌ ║    ║  TASK  ║▐▌                   ║▐▌ ║    ╚════════╝▐▌                   ║▐▌ ║     ▀▀▀▀▀▀▀▀▀▀▘                   ║▐▌ ║                                   ║▐▌ ╚═══════════════════════════════════╝▐▌ ▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▘ 
```

Tasks within the scoped task set can proceed independently by default. The ordering is imposed by the user in how they relate them to one another using the same concurrency primitives they would use with other parts of async Rust. Another usage pattern would have a completely different set of relationships between the tasks.

Some users report experiencing achieving less concurrency than they expected when using buffered streams because they don’t realize the sequencing that occurs: they conceptualize these phases as occurring concurrently, even when they aren’t. The worst sort of issue that can arise as a result of this sequencing is a deadlock, in which no future within the task can make progress because they are all waiting on one another.

# Deadlocks

Deadlocks can occur only when a set of four conditions are met. These are known as the “Coffman conditions,” after Edward G. Coffman, who wrote a definitive [article](https://people.cs.umass.edu/~mcorner/courses/691J/papers/TS/coffman_deadlocks/coffman_deadlocks.pdf) on deadlock in 1971. The original paper expressed the requirements more crisply than I ever could (emphasis is mine):

> 1.  Tasks claim exclusive control of the resources they require (**“mutual exclusion”** condition).
> 2.  Tasks hold resources already allocated to them while waiting for additional resources (**“wait for”** condition).
> 3.  Resources cannot be forcibly removed from the tasks holding them until the resources are used to completion (**“no preemption”** condition).
> 4.  A circular chain of tasks exists, such that each task holds one or more resources that are being requested by the next task in the chain (**“circular wait”** condition).

To further explore this topic, I want to demonstrate a deadlocks using a much simpler example of an async iterator, which doesn’t use buffered streams or `FuturesUnordered` at all. I’m going to use async generator syntax to define my iterator, but this is only for convenience and has no bearing on the deadlock.

We shall have one unit of work (the async generator) which yields two values `x` and `y` while holding a lock on a shared resource. Another unit of work (the for await loop) will consume those values and print them, also taking the lock for each value. In this psuedo-code, holding the lock is unnecessary; imagine a more complicated example in which the algorithm for the production and consumption of these values both require exclusive access to that resource protected by the `Mutex`.

```
// yield two values with an async generator: let  iter  =  async  gen  {  let  mut  guard  =  mutex.lock().await; yield  x; yield  y; drop(guard); };   // consume two values with a for await loop: for  await  elem  in  iter  {  let  mut  guard  =  mutex.lock().await; println!("{}",  elem); drop(guard); } 
```

This code will result in a deadlock. First, the generator will take a lock on the mutex, then it will yield its first value while holding that lock. The `for` loop when then attempt to take the lock while processing that value, but it can’t take it until the generator gives it up. But the generator does not give up the lock until it yields its second value, which it cannot do because the `for` loop isn’t ready to process the second value until it has processed the first. And note that `poll_progress` is no help here: the generator cannot make progress except by yielding the second value, which it cannot do until the `for` loop is done consuming the first.

This is a classic “circular wait” scenario in which the iterator is waiting on the `for` loop and the `for` loop is waiting on the iterator. What’s perhaps interesting about this scenario is that there is only one lock. We are left to ask, what is the second shared resource in this deadlock scenario? The iterator holds the `Mutex`, but what does the `for` loop hold?

The second resource is control of the execution of code itself. In a single task, only one part of the task can be executing at any one time. That control itself is an implicit shared resource that different sub-units of the task need exclusive access to to make progress. In our example, the iterator is holding the lock on the `Mutex`, whereas the `for` loop is holding the lock on the task.

```
 ┌ WAITING TO BE POLLED ───┐ ╔════════════════╗        ╔══════════▼═════╗ ║                ║▐▌      ║                ║▐▌ ║ ASYNC ITERATOR ║▐▌      ║ FOR AWAIT LOOP ║▐▌ ║  (holds lock)  ║▐▌      ║  (holds task)  ║▐▌ ║                ║▐▌      ║                ║▐▌ ╚══════▲═════════╝▐▌      ╚════════════════╝▐▌ ▀▀▀▀▀▀│▀▀▀▀▀▀▀▀▀▀▀▘       ▀▀▀▀▀│▀▀▀▀▀▀▀▀▀▀▀▀▘ └── WAITING TO TAKE LOCK ┘ 
```

A note before we proceed: this problem is not the result of anything particular to async Rust, but is inherent in any imperative procedural language. If you were to write the same code with a blocking `Mutex` and eliminate all the async and await keywords, you’ll get the same behavior: the iterator will take the lock and yield its first item, then the `for` loop will block waiting to take the lock to process that item, and your system will deadlock. It doesn’t matter if you represent a slice of compute time with futures or threads, if code execution proceeds “one damn thing after another,” this algorithm will deadlock.

The only way to prevent this deadlock would be able to preempt the exclusive access to one of the two resources involved: either to preempt the `Mutex` or preempt the code holding control of the compute. The second is not possible in an imperative language when you have explicitly declared a sequential order between the two components. Avoiding this problem was one of the main impetuses for research into languages with less strict ordering semantics like Haskell. In a language with strict ordering, the only solution is explicitly declaring that the order between these operations doesn’t matter; in other words, that they are concurrent. Async Rust provides several facilities for doing that, but this code does not take advantage of them.

On the other hand, it is also possible to preempt the `Mutex` with a lock that is “re-entrant.” With re-entrant lock, either because the `for` loop is in the same thread or same task, it is given access to the lock even while the iterator holds it. This doesn’t cause a data race *because* of the sequencing already provided by the fact that the loop holds control over the order of computation. In other words, a re-entrant `Mutex` breaks the deadlock by piggy-backing on the ordering already provided by the underlying sequential programming model. However, using a re-entrant `Mutex` in this context is risky: if your program logic depends on a guarantee that the state of that shared resource doesn’t change between yields of the iterator, using a re-entrant `Mutex` would upgrade that deadlock into a subtle logic bug with potentially disastrous results. That is to say, this is in most cases no solution at all.

# Buffer data, not code?

The code sample above didn’t involve `FuturesUnordered`, but the deadlocks and performance issues that result from using the buffered stream pattern arise from the same underlying dynamics. Without `poll_progress`, if a future being buffered takes hold of a lock and then the `for` loop tries to take that same lock, the whole thing will deadlock. `poll_progress` eliminates that sequencing point, but the other sequencing point that is used to apply backpressure remains: if the underlying stream of futures took a lock, and then the futures being buffered also try to take that lock (or the `for` loop does), a similar deadlock will occur.

One user reported just such a [bug](https://blog.polybdenum.com/2022/07/24/fixing-the-next-thousand-deadlocks-why-buffered-streams-are-broken-and-how-to-make-them-safer.html) in their program in 2022\. This user proposes changing the signature of `FuturesUnordered` so that it can only await join handles of independently executing tasks as a way to prevent this deadlock. I do not think that is a recommendable approach, as it makes it semantically equivalent to `JoinSet` with a less optimal implementation. Remember the reason to use `FuturesUnordered` is to concurrently execute a dynamic number of futures that borrow from the surrounding scope. The functionally equivalent recommendation would be to avoid `FuturesUnordered` entirely, but I don’t think that’s necessary or sufficient to prevent deadlocks.

I want to consider another deadlock, this time using the scoped task pattern instead of the buffered stream pattern. This deadlock could also be achieved with ordinary tasks on an executor - it doesn’t depend on anything particular to the semantics of `FuturesUnordered`. We’ll implement basically the same algorithm as before, and have a deadlock for basically the same reason.

This code will spawn two sub-tasks which communicate via a channel. The first sub task will send `x` and `y` through the channel while holding onto a lock. The second will receive all values from the channel, await to take the lock, and print each one. For the purposes of making my point, this channel will be a bounded channel with length `0`. This means that sending is not complete until it is actually received on the other side of the channel, as in std’s [sync_channel](https://doc.rust-lang.org/std/sync/mpsc/fn.sync_channel.html) API. (tokio channels don’t support this behavior because they consider it to have risks around cancellation).

```
let  (tx,  rx)  =  channel(0);   // send two values through a channel: scope.spawn(async  {  let  mut  guard  =  mutex.lock().await; tx.send(x).await; tx.send(y).await; drop(guard); });   // process all values sent through the channel: scope.spawn(async  {  for  await  elem  in  rx  { let  mut  guard  =  mutex.lock().await; println!("{}",  elem); drop(guard); } }); 
```

Here, exactly the same deadlock emerges again because the channel does not have enough capacity to hold the data being sent into the channel while waiting for the other task to process it. It is the same circular wait pattern, but control of the task has been replaced with space in the channel:

```
 ┌ WAITING TO SEND DATA ───┐ ╔════════════════╗        ╔══════════▼══════╗ ║                ║▐▌      ║                 ║▐▌ ║ FIRST SUB-TASK ║▐▌      ║ SECOND SUB-TASK ║▐▌ ║  (holds lock)  ║▐▌      ║  (holds rx)     ║▐▌ ║                ║▐▌      ║                 ║▐▌ ╚══════▲═════════╝▐▌      ╚═════════════════╝▐▌ ▀▀▀▀▀▀│▀▀▀▀▀▀▀▀▀▀▀▘       ▀▀▀▀▀│▀▀▀▀▀▀▀▀▀▀▀▀▀▘ └── WAITING TO TAKE LOCK ┘ 
```

There are two conclusions we could reach from this consideration.

The first, and more appealing, would be to say that circular dependencies with locks and channels are easier to spot. Users *know* they’re sharing resources when they use synchronization primitives. Therefore, spawning tasks (whether scoped tasks or independent tasks) or using intra-task concurrency primitives that don’t introduce subtle sequencing points is less likely to result in deadlock, because users can analyze the code more easily. This is because there is no implicit shared resource among the concurrent units: all dependencies are made more obvious by way of the use of synchronization primitives.

In other words, **buffer data, not code**. The practical implication of this would be to reject the buffered stream pattern as a risky and confusing practice, and prefer the scoped task pattern. As I said this is appealing: it results in a straightforward practical rule that users can apply to reduce their risk of deadlock.

The alternative conclusion is not as nice, but should be taken seriously. Consider that ultimately in both cases the deadlock results from a limit on the number of units that can be buffered. Users either limit the number of concurrent futures in a buffered stream or limit the number of objects in a channel. This is necessary to achieve backpressure in your system. But if you have a cyclic dependency which is bounded in both directions, deadlock will occur if all of those queues in the cycle fill up completely.

If its *impossible* for a queue to fill up, deadlock can be prevented. For example, in the code above just setting the channel to a length above `0` would have prevented the deadlock. But if the amount of items that the sender could produce to put into the queue is larger than the bound on the queue and a cyclic dependency exists, it is possible to cause deadlock whatever the queue size. It becomes a question of how frequently that will occur: if it is very rare for the queue to be full, deadlocks will be very improbable but not impossible.

In my experience, users usually set the bound on their channel to a relatively large but ultimately arbitrary figure. A value in the hundreds or low thousands is something I commonly see, for reasons that are not usually very good. On the other hand, I suspect the limit on buffered streams is usually set to a much lower figure, a single digit limit is common I think. In other words, the way that channels are used might be making deadlocks much less *probable* without making them any less *possible*, so users are not encountering them in testing, though they still lurk in your code waiting to be encountered in production. Maybe there really is no difference between buffered streams and channels except the queue size users tend to select?

For some use cases, highly improbably deadlocks are fine; the service degradation and data loss that results from a deadlock that occurs very infrequently may be tolerable for your system. If it isn’t, though, by setting queues to arbitrary large sizes you may be hiding potential deadlocks, especially if the flow among components in your system them is not obvious to programmers because of the size and complexity of your application. One way to mitigate this is to test with unbuffered channels (to ensure that even if your performance is degraded, the system still continues to make progress) while using buffered channels in production as a performance enhancement to enable to you to do more work the same amount of compute time.

# Synchronization & the borrow checker

One important fact to consider is that deadlocks like the above are only possible if the user is using a synchronization primitive. Buffered streams alone without something like locks or channels cannot result in a circular wait condition. This is because of ownership & borrowing prevent the “wait for” condition: since the mutual exclusive property is statically determined, no code ever has to wait for exclusive access to a shared resource. The only “resource” code can wait for is control of the current task, and without multiple resources it is not possible to enter into a “circular wait.” Instead of a deadlock, you’ll get a borrow checker error.

This doesn’t mean users should avoid synchronization primitives entirely, just that they should be alert that they carry additional risks. When using these primitives, users must be aware of the shape of their synchronization dependency graph to recognize potential deadlocks. But users must also be aware that control flow is also a shared resource, and alert to multiple uses of a synchronization primitive in non-concurrent components of a single task. If buffered streams makes that non-concurrent control flow too opaque, that particular pattern should be discouraged.

Enabling users to often avoid synchronization and shared ownership - and therefore potential deadlocks - is the entire reason users reach for `FuturesUnordered`. That the Rust type system can enable this is one of the most interesting and important things about Rust. Ultimately, patterns that use ownership and borrowing this way should be leaned into, because they play to Rust’s strengths. It would be ideal if instead of using `FuturesUnordered`, the scoped task pattern could be integrated into the runtime you use, rather than requiring a separate abstraction - that is, if `JoinSet` could spawn tasks that are not `'static`, but instead allow borrowing from the surrounding context. To achieve that would require some resolution to the [scoped task trilemma](/blog/the-scoped-task-trilemma).