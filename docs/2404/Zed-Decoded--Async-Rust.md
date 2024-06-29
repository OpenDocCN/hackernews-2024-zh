<!--yml

category: 未分类

date: 2024-05-27 13:03:09

-->

# Zed Decoded: 异步 Rust

> 来源：[https://zed.dev/blog/zed-decoded-async-rust](https://zed.dev/blog/zed-decoded-async-rust)

# Zed Decoded: 异步 Rust

欢迎来到新系列文章的第一篇 **Zed Decoded**。在 Zed Decoded 中，我将仔细研究 Zed — 它是如何构建的，使用了哪些数据结构，哪些技术和技巧，具备了哪些功能，我们遇到了哪些错误。最棒的部分？我不会独自完成，而是有机会采访和询问在 Zed 工作的同事关于我想了解的一切。

**伴随视频**：异步 Rust

本文配有一段 1 小时的视频，在其中 Thorsten 和 Antonio 探讨 Zed 如何在异步 Rust 中使用 — 在 Zed 中。这是一次自由的对话，侧重于代码并深入探讨了一些没有包含在文章中的主题。

在这里观看视频：[https://youtu.be/gkU4NGSe21I](https://youtu.be/gkU4NGSe21I)

我列表上的第一个主题：异步 Rust 及其在 Zed 中的应用。在过去的几个月里，我对异步 Rust 深感兴趣 — Zed 是我工作过的第一个使用它的代码库 — 所以我决定坐下来询问 Zed 的联合创始人之一 Antonio，关于我们如何在 Zed 中使用异步 Rust。

我们不会深入讨论异步 Rust 本身的细节（如果你想理解我们将看到的代码的细微差别，熟悉它是理所当然的），而是专注于 Zed 如何使用异步 Rust 构建高性能本地应用程序：应用层面的异步代码看起来如何，它使用哪个运行时，为什么使用该运行时。

## 用 GPUI 编写异步 Rust

让我们直接进入深水区。这里是一个代码片段，代表了 Zed 代码库中的异步代码：

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

这是来自我们的 `Editor` 的一个函数。调用它时，Zed 会显示每个光标的所有者姓名：你的姓名或者你正在协作的人的姓名。例如，当编辑器重新聚焦时会调用它，这样你可以快速看到谁在做什么。

`show_cursor_names` 的作用是：

+   打开 `Editor.show_cursor_names` 并触发编辑器的重新渲染。当 `Editor.show_cursor_names` 为 true 时，光标名称将被渲染。

+   派生一个任务，它会休眠 `CURSOR_VISIBLE_FOR`，关闭光标，并触发另一个重新渲染。

如果你以前写过异步 Rust，你可以在代码中看到一些熟悉的元素：有一个 `.spawn`，有一个 `async move`，有一个 `await`。如果你以前使用过 `async_task` crate，这可能会让你想起类似这样的代码 [像这样](https://docs.rs/async-task/4.7.0/async_task/struct.Task.html#method.detach)。

```
let ex = Executor::new();
ex.spawn(async {
 loop {
 Timer::after(Duration::from_secs(1)).await;
 }
})
.detach();
```

这是因为 Zed 使用 `async_task` 来实现其 `Task` 类型。但在这个示例中有一个 `Executor` — 在 Zed 代码中它在哪里？`cx.background_executor()` 是做什么的？好问题，让我们找到答案。

## macOS 作为我们的异步运行时

Rust 的异步编程中一个显著的特点是它允许你选择自己的运行时。这与许多其他语言（如 JavaScript）不同，这些语言也可以编写异步代码。运行时并没有非常明确的定义，但在这里，我们可以说运行时是运行您的异步代码并为您提供像 `.spawn` 和类似 `Executor` 这样的实用程序的东西。

这些运行时中最流行的可能是 [tokio](https://github.com/tokio-rs/tokio)。但还有 [smol](https://github.com/smol-rs/smol)，[embassy](https://github.com/embassy-rs/embassy) 等等。选择和切换运行时伴随着权衡，它们 [在一定程度上是可以互换的](https://corrode.dev/blog/async/)，但是确实可能。

在 macOS 的 Zed 中，事实证明我们不使用其中任何一个。我们也不使用 `async_task` 的 `Executor`。但是必须有某些东西来执行异步代码，对吧？否则我就不能在 Zed 中输入这些行了。

那么 `cx.spawn` 是做什么的，`cx.background_executor()` 又是什么？让我们来看看。这里有 [GPUI 的 `AppContext` 中三个相关方法](https://github.com/zed-industries/zed/blob/dc98b3cfa19d6bd4eae813ce7dfaf9d9e13c232c/crates/gpui/src/app.rs#L818-L836)：

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

好的，有两个执行器，`foreground_executor` 和 `background_executor`，它们都有 `.spawn` 方法。我们之前已经在 `show_cursor_names` 中看到了 `background_executor` 的 `.spawn` 方法，在 `AppContext.spawn` 中，我们看到了 `foreground_executor` 的对应部分。

再深入一层，我们可以看到 `foreground_executor.spawn` 做了什么：

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

这里涉及很多内容，有很多语法，但发生的事情可以归结为：`.spawn` 方法接受一个 `future`，将其转换为 [`Runnable`](https://docs.rs/async-task/latest/async_task/struct.Runnable.html) 和一个 `Task`，并请求 `dispatcher` 在主线程上运行它。

这里的 `dispatcher` 是一个 `PlatformDispatcher`。这是 GPUI 中相当于 `async_task` 的 `Executor`。它的名称中带有 `Platform` 是因为它在 macOS、Linux 和 Windows 上有不同的实现。但在本文中，我们只关注 macOS，因为这是目前我们支持最好的平台，而 Linux/Windows 的实现仍在进行中。

那么 `dispatch_on_main_thread` 做什么？现在是调用异步运行时吗？不，也没有运行时 [在这里](https://github.com/zed-industries/zed/blob/dc98b3cfa19d6bd4eae813ce7dfaf9d9e13c232c/crates/gpui/src/platform/mac/dispatcher.rs#L66-L75)：

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

`dispatch_async_f` 是调用离开 Zed 代码库的地方，因为 `dispatch_async_f` 实际上是 [`dispatch_async_f`](https://developer.apple.com/documentation/dispatch/1452834-dispatch_async_f) 函数在 macOS 的 Grand Central Dispatch (GCD) 中的编译时生成的绑定。`dispatch_get_main_queue()` 也是这样的一个绑定。

是的：Zed 作为 macOS 应用程序，使用 macOS 的 GCD 来调度和执行工作。

在上面的片段中，Zed将`Runnable` —— 将其视为`Task`的句柄 —— 转换为原始指针，并将其与`trampoline`一起传递给`dispatch_async_f`，将其放入其`main_queue`中。

当 GCD 决定在`main_queue`上运行下一个项目时，它会将其从队列中弹出，并调用`trampoline`，该函数将原始指针转换回`Runnable`，并轮询其后的`Future`的`.run()`方法。

而正如我惊讶地发现的那样：就是这样。这基本上是使用 GCD 作为异步 Rust 的“运行时”的所有代码。其他应用程序使用 tokio 或 smol，而 Zed 使用 GCD 和诸如`async_task`等库的薄包装。

等等，`BackgroundExecutor`怎么样？与`ForegroundExecutor`非常非常相似，主要区别在于`BackgroundExecutor`在`PlatformDispatcher`上调用此方法。

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

此`dispatch`方法与上述`dispatch_async_f`的唯一区别是队列。`BackgroundExecutor`不使用`main_queue`，而是使用[全局队列](https://developer.apple.com/documentation/dispatch/1452927-dispatch_get_global_queue?language=objc)。

就像我第一次阅读这段代码时所做的那样，现在你可能会想知道：为什么？

为什么使用 GCD？为什么有`ForegroundExecutor`和`BackgroundExecutor`？`main_queue`有何特殊之处？

## 绝不能阻塞主线程。

在本机 UI 应用程序中，主线程非常重要。不，主线程是*神圣*的。主线程是渲染发生的地方，处理用户输入的地方，操作系统与应用程序通信的地方。主线程绝对不能阻塞。在主线程上，您的应用程序的响应速度决定了其生死。

这对 macOS 上的[Cocoa](https://en.wikipedia.org/wiki/Cocoa_(API))应用程序同样适用。渲染、接收用户输入、与 macOS 通信以及其他平台相关的问题都必须在主线程上进行。由于 Zed 希望与 macOS 完美合作，以确保高性能和响应速度，它做了两件事。

首先，它使用 GCD 调度其工作 —— 在主线程和其他线程上 —— 以便 macOS 可以保持高响应性和整体系统效率。

其次，主线程的重要性被GPUI UI 框架内化，通过显式区分`ForegroundExecutor`和`BackgroundExecutor`来体现，正如我们之前所见。

作为 Zed 应用层代码的编写者，你应该始终注意主线程上发生的事情，并且永远不要将太多阻塞工作放在主线程上。例如，如果在主线程上放置一个阻塞的 `sleep(10ms)`，那么 UI 渲染现在必须等待该 `sleep()` 完成，这意味着渲染下一帧将花费超过 8ms — 这是如果想实现 [120 FPS](https://zed.dev/blog/120fps) 的最大帧时间。你会“丢帧”，正如他们所说的那样。

了解了这些，让我们再来看另一个小代码片段。这次来自 Zed 内置终端的函数，它[搜索终端缓冲区内容的函数](https://github.com/zed-industries/zed/blob/dc98b3cfa19d6bd4eae813ce7dfaf9d9e13c232c/crates/terminal/src/terminal.rs#L1346-L1358)：

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

在 `find_matches` 的第一行，`self.term.clone()` 在主线程上执行并且很快：`self.term` 是一个 `Arc<Mutex<...>>`，所以克隆只是增加了 `Arc` 的引用计数。然后调用 `.lock()` 是在后台进行的，因为 `.lock()` 可能会阻塞。在这个特定的代码路径中，不太可能发生对该锁的争用，但如果有争用，它不会冻结 UI，只会影响单个后台线程。这是模式：如果很快，可以在主线程上执行，但如果可能需要一段时间甚至会阻塞，使用 `cx.background_executor()` 将其放到后台线程执行。

这里还有一个例子，在 Zed 中进行的项目范围搜索 (`⌘-shift-f`)。它尽可能将大量繁重的工作推到后台线程，以确保在搜索数万个项目文件时，Zed 仍然保持响应性。这是简化且有大量注释的 [`Project.search_local` 的节选](https://github.com/zed-industries/zed/blob/dc98b3cfa19d6bd4eae813ce7dfaf9d9e13c232c/crates/project/src/project.rs#L6485-L6498)，展示了搜索的主要部分。

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

代码很多 — 抱歉！ — 但实际上并没有比我们已经讨论过的概念更多的东西。这里值得注意的是主线程和后台线程之间的 ping-pong：

+   **主线程**：启动搜索并将 `query` 交给后台线程处理。

+   **后台线程**：在项目中找到包含 `query` 多于 1 次出现的文件，并在收到结果时通过通道发送回来。

+   **主线程**：等待直到后台线程找到 `MAX+1` 个结果，然后关闭通道，导致后台线程退出。

+   **主线程**：启动多个其他主线程任务以打开每个文件并创建快照。

+   **后台线程**：搜索缓冲区快照以找到缓冲区中的所有结果，并通过通道发送结果回来。

+   **主线程**：等待后台线程在所有缓冲区中找到结果，然后将它们发送回调用外部 `search_local` 方法的调用者。

尽管这种方法可以进行优化，并且搜索速度可以大大提高（我们还没有开始做到这一点），它已经可以搜索成千上万个文件而不会阻塞主线程，同时仍然利用多个CPU核心。

## 异步友好的数据结构、测试执行器等更多内容

我非常确定之前的代码摘录引发了许多问题，我还没有回答：如何确切地将缓冲快照发送到后台线程？这样做的效率如何？如果我想在另一个线程修改这样的快照怎么办？如何测试所有这些？

很抱歉我没有办法在这篇文章中涵盖所有的答案。但是有一个[配套视频](https://youtu.be/gkU4NGSe21I)，在这个视频中，安东尼奥和我深入探讨了许多这些领域，并讨论了异步友好的数据结构、写时复制的缓冲快照等内容。安东尼奥还给出了[一个关于我们如何对Zed代码库中的异步Rust代码进行属性测试的精彩演讲](https://www.youtube.com/watch?v=ms8zKpS_dZE)，强烈推荐。我也承诺将来会发表关于Zed编辑器底层数据结构的文章。

期待下次再见！

* * *
