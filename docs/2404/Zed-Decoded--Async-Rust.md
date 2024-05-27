<!--yml
category: 未分类
date: 2024-05-27 13:03:09
-->

# Zed Decoded: Async Rust

> 来源：[https://zed.dev/blog/zed-decoded-async-rust](https://zed.dev/blog/zed-decoded-async-rust)

# Zed Decoded: Async Rust

Welcome to the first article in a new series called **Zed Decoded**. In Zed Decoded I'm going to take a close look at Zed — how it's built, which data structures it uses, which technologies and techniques, what features it has, which bugs we ran into. The best part? I won't do this alone, but get to interview and ask my colleagues here at Zed about everything I want to know.

**Companion Video**: Async Rust

This post comes with a 1hr companion video, in which Thorsten and Antonio explore how Zed uses async Rust — in Zed. It's a loose conversation that focuses on the code and dives a bit deeper into some topics that didn't fit into the post.

Watch the video here: [https://youtu.be/gkU4NGSe21I](https://youtu.be/gkU4NGSe21I)

The first topic that was on my list: async Rust and how it's used in Zed. Over the past few months I've become quite fascinated with async Rust — Zed's the first codebase I've worked in that uses it — so I decided to sit down and ask Antonio, one of Zed's co-founders, about how we use async Rust in Zed.

We won't get into the details of async Rust itself (familiarity with that is to be expected if you want to understand the nitty-gritty of the code we'll see), but instead focus on how Zed uses async Rust to build a high-performance, native application: what async code looks like on the application level, which runtime it uses, why it uses that runtime.

## Writing async Rust with GPUI

Let's jump right into the deep end. Here is a snippet of code that's representative of async code in the Zed codebase:

```
fn show_cursor_names(&mut self, cx: &mut ViewContext<Self>) {
 self.show_cursor_names = true;
 cx.notify();
 cx.spawn(|this, mut cx| async move {
 cx.background_executor().timer(CURSORS_VISIBLE_FOR).await;
 this.update(&mut cx, |this, cx| {
 this.show_cursor_names = false;
 cx.notify()
 })
 .ok()
 })
 .detach();
}
```

It's [a function from our `Editor`](https://github.com/zed-industries/zed/blob/98ddefc8884d0957ab766b3aea09265c8423684e/crates/editor/src/editor.rs#L3935-L3947). When it's called, Zed shows the names of the owners of each cursor: your name or the names of the people you're collaborating with. It's called, for example, when the editor is re-focused, so you can quickly see who's doing what and where.

What `show_cursor_names` does is the following:

*   Toggle on `Editor.show_cursor_names` and trigger a re-render of the editor. When `Editor.show_cursor_names` is true, cursor names will be rendered.
*   Spawn a task that sleeps for `CURSOR_VISIBLE_FOR`, turn the cursors off, and trigger another re-render.

If you've ever written async Rust before, you can spot some familiar elements in the code: there's a `.spawn`, there's an `async move`, there's an `await`. And if you've ever used the `async_task` crate before, this might remind you of code [like this](https://docs.rs/async-task/4.7.0/async_task/struct.Task.html#method.detach):

```
let ex = Executor::new();
ex.spawn(async {
 loop {
 Timer::after(Duration::from_secs(1)).await;
 }
})
.detach();
```

That's because Zed uses `async_task` for its `Task` type. But in this example there's an `Executor` — where is that in the Zed code? And what does `cx.background_executor()` do? Good questions, let's find answers.

## macOS as our async runtime

One remarkable thing about async Rust is that it allows you to choose your own runtime. That's different from a lot of other languages (such as JavaScript) in which you can also write asynchronous code. Runtime isn't a term with very sharp definition, but for our purposes here, we can say that a runtime is the thing that runs your asynchronous code and provides you with utilities such as `.spawn` and something like an `Executor`.

The most popular of these runtimes is probably [tokio](https://github.com/tokio-rs/tokio). But there's also [smol](https://github.com/smol-rs/smol), [embassy](https://github.com/embassy-rs/embassy) and others. Choosing and switching runtimes comes with tradeoffs, they [are only interchangable to a degree](https://corrode.dev/blog/async/), but it is possible.

In Zed for macOS, as it turns out, we don't use any one of these. We also don't use `async_task`'s `Executor`. But there has to be something to execute the asynchronous code, right? Otherwise I wouldn't be typing these lines in Zed.

So what then does `cx.spawn` do and what is the `cx.background_executor()`? Let's take a look. Here are [three relevant methods from GPUI's `AppContext`](https://github.com/zed-industries/zed/blob/dc98b3cfa19d6bd4eae813ce7dfaf9d9e13c232c/crates/gpui/src/app.rs#L818-L836):

```
// crates/gpui/src/app.rs

impl AppContext {
 pub fn background_executor(&self) -> &BackgroundExecutor {
 &self.background_executor
 }

 pub fn foreground_executor(&self) -> &ForegroundExecutor {
 &self.foreground_executor
 }

 /// Spawns the future returned by the given function on the thread pool. The closure will be invoked
 /// with [AsyncAppContext], which allows the application state to be accessed across await points.
 pub fn spawn<Fut, R>(&self, f: impl FnOnce(AsyncAppContext) -> Fut) -> Task<R>
 where
 Fut: Future<Output = R> + 'static,
 R: 'static,
 {
 self.foreground_executor.spawn(f(self.to_async()))
 }

 // [...]
}
```

Alright, two executors, `foreground_executor` and `background_executor`, and both have `.spawn` methods. We already saw `background_executor`'s `.spawn` above in `show_cursor_names` and here, in `AppContext.spawn`, we see the `foreground_executor` counterpart.

One level deeper, we can see what `foreground_executor.spawn` does:

```
// crates/gpui/src/executor.rs

impl ForegroundExecutor {
 /// Enqueues the given Task to run on the main thread at some point in the future.
 pub fn spawn<R>(&self, future: impl Future<Output = R> + 'static) -> Task<R>
 where
 R: 'static,
 {
 let dispatcher = self.dispatcher.clone();
 fn inner<R: 'static>(
 dispatcher: Arc<dyn PlatformDispatcher>,
 future: AnyLocalFuture<R>,
 ) -> Task<R> {
 let (runnable, task) = async_task::spawn_local(future, move |runnable| {
 dispatcher.dispatch_on_main_thread(runnable)
 });
 runnable.schedule();
 Task::Spawned(task)
 }
 inner::<R>(dispatcher, Box::pin(future))
 }

 // [...]
}
```

There's a lot going on here, a lot of syntax, but what happens can be boiled down to this: the `.spawn` method takes in a `future`, turns it into a [`Runnable`](https://docs.rs/async-task/latest/async_task/struct.Runnable.html) and a `Task`, and asks the `dispatcher` to run it on the main thread.

The `dispatcher` here is a `PlatformDispatcher`. That's the GPUI equivalent of `async_task`'s `Executor` from above. It has `Platform` in its name because it has different implementations for macOS, Linux, and Windows. But in this post, we're only going to look at macOS, since that's our best-supported platform at the moment and Linux/Windows implementations are still work-in-progress.

So what does `dispatch_on_main_thread` do? Does *this* now call an async runtime? No, no runtime [there either](https://github.com/zed-industries/zed/blob/dc98b3cfa19d6bd4eae813ce7dfaf9d9e13c232c/crates/gpui/src/platform/mac/dispatcher.rs#L66-L75):

```
// crates/gpui/src/platform/mac/dispatcher.rs

impl PlatformDispatcher for MacDispatcher {
 fn dispatch_on_main_thread(&self, runnable: Runnable) {
 unsafe {
 dispatch_async_f(
 dispatch_get_main_queue(),
 runnable.into_raw().as_ptr() as *mut c_void,
 Some(trampoline),
 );
 }
 }
 // [...]
}

extern "C" fn trampoline(runnable: *mut c_void) {
 let task = unsafe { Runnable::<()>::from_raw(NonNull::new_unchecked(runnable as *mut ())) };
 task.run();
}
```

`dispatch_async_f` is where the call leaves the Zed codebase, because `dispatch_async_f` is actually a compile-time generated binding to the [`dispatch_async_f`](https://developer.apple.com/documentation/dispatch/1452834-dispatch_async_f) function in [macOS' Grand Central Dispatch's (GCD)](https://developer.apple.com/documentation/DISPATCH). `dispatch_get_main_queue()`, too, is such a binding.

That's right: Zed, as a macOS application, uses macOS' GCD to schedule and execute work.

What happens in the snippet above is that Zed turns the `Runnable` — think of it as a handle to a `Task` — into a raw pointer and passes it to `dispatch_async_f` along with a `trampoline`, which puts it on its `main_queue`.

When GCD then decides it's time to run the next item on the `main_queue`, it pops it off the queue, and calls `trampoline`, which takes the raw pointer, turns it back into a `Runnable` and, to poll the `Future` behind its `Task`, calls `.run()` on it.

And, as I learned to my big surprise: that's it. That's essentially all the code necessary to use GCD as a "runtime" for async Rust. Where other applications use tokio or smol, Zed uses thin wrappers around GCD and crates such as `async_task`.

Wait, but what about the `BackgroundExecutor`? It's very, very similar to the `ForegroundExecutor`, with the main difference being that the `BackgroundExecutor` calls this method on `PlatformDispatcher`:

```
impl PlatformDispatcher for MacDispatcher {
 fn dispatch(&self, runnable: Runnable, _: Option<TaskLabel>) {
 unsafe {
 dispatch_async_f(
 dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH.try_into().unwrap(), 0),
 runnable.into_raw().as_ptr() as *mut c_void,
 Some(trampoline),
 );
 }
 }
}
```

The only difference between this `dispatch` method and `dispatch_async_f` from above is the queue. The `BackgroundExecutor` doesn't use the `main_queue`, but [a global queue](https://developer.apple.com/documentation/dispatch/1452927-dispatch_get_global_queue?language=objc).

Like I did when I first read through this code, you now might wonder: why?

Why use GCD? Why have a `ForegroundExecutor` and a `BackgroundExecutor`? What's so special about the `main_queue`?

## Never block the main thread

In a native UI application, the main thread is important. No, the main thread is *holy*. The main thread is where the rendering happens, where user input is handled, where the operating system communicates with the application. The main thread should never, ever block. On the main thread, the responsiveness of your app lives or dies.

That's true for [Cocoa](https://en.wikipedia.org/wiki/Cocoa_(API)) applications on macOS too. Rendering, receiving user input, communication with macOS, and other platform concerns have to happen on the main thread. And since Zed wants perfect cooperation with macOS to ensure high-performance and responsiveness, it does two things.

First, it uses GCD to schedule its work — on and off the main thread — so that macOS can maintain high responsiveness and overall system efficiency.

Second, the importance of the main thread is baked into GPUI, the UI framework, by explicitly making the distinction between the `ForegroundExecutor` and the `BackgroundExecutor`, both of which we saw above.

As a writer of application-level Zed code, you should always be mindful of what happens on the main thread and never put too much blocking work on it. If you were to put, say, a blocking `sleep(10ms)` on the main thread, rendering the UI now has to wait for that `sleep()` to finish, which means that rendering the next frame would take longer than 8ms — the maximum frame time available if you want to achieve [120 FPS](https://zed.dev/blog/120fps). You'd "drop a frame", as they say.

Knowing that, let's take a look at another small snippet of code. This time it's from the built-in terminal in Zed, a function that [searches through the contents of the terminal buffer](https://github.com/zed-industries/zed/blob/dc98b3cfa19d6bd4eae813ce7dfaf9d9e13c232c/crates/terminal/src/terminal.rs#L1346-L1358):

```
// crates/terminal/src/terminal.rs

pub struct Terminal {
 term: Arc<Mutex<alacritty_terminal::Term<ZedListener>>>,

 // [... other fields ...]
}

pub fn find_matches(
 &mut self,
 mut searcher: RegexSearch,
 cx: &mut ModelContext<Self>,
) -> Task<Vec<RangeInclusive<AlacPoint>>> {
 let term = self.term.clone();
 cx.background_executor().spawn(async move {
 let term = term.lock();

 all_search_matches(&term, &mut searcher).collect()
 })
}
```

The first line in `find_matches`, the `self.term.clone()`, happens on the main thread and is quick: `self.term` is an `Arc<Mutex<...>>`, so cloning only bumps the reference count on the `Arc`. The call to `.lock()` then only happens in the background, since `.lock()` might block. It's unlikely that there will be contention for this lock in this particular code path, but if there was contention, it wouldn't freeze the UI, only a single background thread. That's the pattern: if it's quick, you can do it on the main thread, but if it might take a while or even block, put it on a background thread by using `cx.background_executor()`.

Here's another example, the project-wide search in Zed (`⌘-shift-f`). It pushes as much heavy work as possible onto background threads to ensure Zed stays responsive while searching through tens of thousands of files in your project. Here's a simplified and heavily-commented [excerpt from `Project.search_local`](https://github.com/zed-industries/zed/blob/dc98b3cfa19d6bd4eae813ce7dfaf9d9e13c232c/crates/project/src/project.rs#L6485-L6498) that shows the main part of the search:

```
// crates/project/src/project.rs

// Spawn a Task on the background executor. The Task finds all files on disk
// that contain >1 matches for the given `query` and sends them back over
// the `matching_paths_tx` channel.
let (matching_paths_tx, matching_paths_rx) = smol::channel::bounded(1024);
cx.background_executor()
 .spawn(Self::background_search(
 // [... other arguments ... ]
 query.clone(),
 matching_paths_tx,
 ))
 .detach();

// Setup a channel on which we stream results to the UI.
let (result_tx, result_rx) = smol::channel::bounded(1024);

// On the main thread, spawn a Task that first...
cx.spawn(|this, mut cx| async move {
 // ... waits for the background thread to return the filepaths of
 // the maximum number of files that we want to search...
 let mut matching_paths = matching_paths_rx
 .take(MAX_SEARCH_RESULT_FILES + 1)
 .collect::<Vec<_>>()
 .await;

 // ... then loops over the filepaths in chunks of 64...
 for matching_paths_chunk in matching_paths.chunks(64) {
 let mut chunk_results = Vec::new();

 for matching_path in matching_paths_chunk {
 // .... opens each file....
 let buffer = this.update(&mut cx, |this, cx| {
 this.open_buffer((*worktree_id, path.clone()), cx)
 })?;

 // ... and pushes into `chunk_results` a Task that
 // runs on the main thread and ...
 chunk_results.push(cx.spawn(|cx| async move {
 // ... waits for the file to be opened ...
 let buffer = buffer.await?;
 // ... creates a snapshot of its contents ...
 let snapshot = buffer.read_with(&cx, |buffer, _| buffer.snapshot())?;
 // ... and again starts a Task on the background executor,
 // which searches through the snapshot for all results.
 let ranges = cx
 .background_executor()
 .spawn(async move {
 query
 .search(&snapshot, None)
 .await
 .iter()
 .collect::<Vec<_>>()
 })
 .await;

 Ok((buffer, ranges))
 }));
 }

 // On the main thread, non-blocking, wait for all buffers to be searched...
 let chunk_results = futures::future::join_all(chunk_results).await;
 for result in chunk_results {
 if let Some((buffer, ranges)) = result.log_err() {
 // send the results over the results channel
 result_tx
 .send(SearchResult::Buffer { buffer, ranges })
 .await?;
 }
 }
 }
})
.detach();

result_rx
```

It's a lot of code — sorry! — but there's not a lot more going on than the concepts we already talked about. What's noteworthy here and why I wanted to show it is the ping-pong between the main thread and background threads:

*   **main thread**: kicks off the search and hands the `query` over to background thread
*   **background thread**: finds files in project with >1 occurrences of `query` in them, sends results back over channel as they come in
*   **main thread**: waits until background thread has found `MAX+1` results, then drops channel, which causes background thread to exit
*   **main thread**: spawns multiple other main-thread tasks to open each file & create a snapshot.
*   **background threads**: search through buffer snapshot to find all results in a buffer, sends results back over channel
*   **main thread**: waits for background thread to find results in all buffers, then sends them back to the caller of the outer `search_local` method

Even though this method can be optimized and the search made a lot faster (we haven't gotten around to that yet), it can already search thousands of files without blocking the main thread, while still using multiple CPU cores.

## Async-Friendly Data Structures, Testing Executors, and More

I'm pretty sure that the previous code excerpt raised a lot of questions that I haven't answered yet: how exactly is it possible to send a buffer snapshot to a background thread? How efficient is it do that? What if I want to modify such a snapshot on another thread? How do you test all this?

And I'm sorry to say that I couldn't fit all of the answers into this post. But there is a [companion video](https://youtu.be/gkU4NGSe21I) in which Antonio and I did dive into a lot of these areas and talked about async-friendly data structures, copy-on-write buffer snapshots, and other things. Antonio also gave [a fantastic talk about how we do property-testing of async Rust code](https://www.youtube.com/watch?v=ms8zKpS_dZE) in the Zed code base that I highly recommend. I also promise that in the future there will be a post about the data structures underlying the Zed editor.

Until next time!

* * *