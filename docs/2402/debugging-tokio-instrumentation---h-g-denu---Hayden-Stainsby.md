<!--yml

category: 未分类

date: 2024-05-27 14:44:00

-->

# 调试tokio仪表化 - hēg denu - 海登·斯坦斯比

> 来源：[https://hegdenu.net/posts/debugging-tokio-instrumentation/](https://hegdenu.net/posts/debugging-tokio-instrumentation/)

我为[`tokio-console`](https://github.com/tokio-rs/console)做贡献。我经常发现自己在与来自Tokio的“原始”[`tracing`](https://docs.rs/tracing)输出中显示的内容匹配。然而，这很难阅读，实际上并不包含我需要的所有信息。

有几件事情我想要。首先（也是最重要的），我需要看到Tracing的内部[`span::Id`](https://docs.rs/tracing/0.1.40/tracing/span/struct.Id.html)用于发出的spans。这是Tokio中的仪表化中非常重要的一部分，用于调试，我确实需要它可用。

通常，要获取这些信息，我使用`tracing-subscriber` crate的修补版本。但这不是可以提交到控制台项目的东西，每次设置都有点繁琐。

其次，我希望能够视觉上区分Tokio仪表化中使用的特定spans和events。与内部span ID不同，这完全是领域特定的，在这个特定用例之外没有用途。

这里是从[`fmt::Subscriber`](https://docs.rs/tracing-subscriber/0.3.18/tracing_subscriber/fmt/struct.Subscriber.html)输出的一小段Tokio的仪表输出。

```
2024-01-31T09:23:11.247690Z  INFO **runtime.spawn{***kind*=task *task.name*= *task.id*=18 *loc.file*="src/bin/initial-example.rs" *loc.line*=11 *loc.col*=5**}**: initial_example: pre-yield *fun*=true
2024-01-31T09:23:11.247703Z TRACE **runtime.spawn{***kind*=task *task.name*= *task.id*=18 *loc.file*="src/bin/initial-example.rs" *loc.line*=11 *loc.col*=5**}**: tokio::task::waker: *op*="waker.clone" *task.id*=2
2024-01-31T09:23:11.247717Z TRACE **runtime.spawn{***kind*=task *task.name*= *task.id*=18 *loc.file*="src/bin/initial-example.rs" *loc.line*=11 *loc.col*=5**}**: tokio::task: exit
2024-01-31T09:23:11.247737Z TRACE tokio::task::waker: *op*="waker.wake" *task.id*=2
2024-01-31T09:23:11.247754Z TRACE **runtime.spawn{***kind*=task *task.name*= *task.id*=18 *loc.file*="src/bin/initial-example.rs" *loc.line*=11 *loc.col*=5**}**: tokio::task: enter
2024-01-31T09:23:11.247766Z TRACE **runtime.resource{***concrete_type*="Barrier" *kind*="Sync" *loc.file*="src/bin/initial-example.rs" *loc.line*=9 *loc.col*=39**}**: tokio::sync::barrier: enter
2024-01-31T09:23:11.247800Z TRACE **runtime.resource{***concrete_type*="Barrier" *kind*="Sync" *loc.file*="src/bin/initial-example.rs" *loc.line*=9 *loc.col*=39**}**:**runtime.resource.async_op{***source*="Barrier::wait" *inherits_child_attrs*=false**}**: tokio::util::trace: new 
```

这里有很多信息，区分不同的span类型可能很复杂（特别是当你浏览几十甚至数百行时）。此外，完全缺少[`span::Id`](https://docs.rs/tracing/0.1.40/tracing/span/struct.Id.html)。

与相同部分日志的输出进行比较，带颜色并包括[`span::Id`](https://docs.rs/tracing/0.1.40/tracing/span/struct.Id.html)位于span名称之后。

```
**2024-01-31**T**09:23:39**.136879Z  INFO runtime.spawn[**2**]{kind=task, task.name=, task.id=18, loc.file="src/bin/initial-example.rs", loc.line=18, loc.col=5} **initial_example**: fun=true pre-yield
**2024-01-31**T**09:23:39**.136937Z TRACE runtime.spawn[**2**]{kind=task, task.name=, task.id=18, loc.file="src/bin/initial-example.rs", loc.line=18, loc.col=5} **tokio::task::waker**: op="waker.clone", task.id=2
**2024-01-31**T**09:23:39**.136995Z TRACE runtime.spawn[**2**]{kind=task, task.name=, task.id=18, loc.file="src/bin/initial-example.rs", loc.line=18, loc.col=5} **exit**
**2024-01-31**T**09:23:39**.137059Z TRACE **tokio::task::waker**: op="waker.wake", task.id=2
**2024-01-31**T**09:23:39**.137122Z TRACE runtime.spawn[**2**]{kind=task, task.name=, task.id=18, loc.file="src/bin/initial-example.rs", loc.line=18, loc.col=5} **enter**
**2024-01-31**T**09:23:39**.137212Z TRACE runtime.resource[**1**]{concrete_type="Barrier", kind="Sync", loc.file="src/bin/initial-example.rs", loc.line=16, loc.col=39} **enter**
**2024-01-31**T**09:23:39**.137296Z TRACE runtime.resource[**1**]{concrete_type="Barrier", kind="Sync", loc.file="src/bin/initial-example.rs", loc.line=16, loc.col=39} runtime.resource.async_op[**274877906945**]{source="Barrier::wait", inherits_child_attrs=false} **new** 
```

现在已经证明了我想做的事情，让我们构建自己的定制追踪订阅者！

（实际上，这将主要是一个`Layer`）

## 旁注：追踪订阅者和层

如果您已经熟悉`tracing`，您可能希望跳过本节，直接进入[ari-subscriber](https://hegdenu.net/posts/debugging-tokio-instrumentation/#ari-subscriber)。

在tracing生态系统中，除了将跟踪数据发送到虚空之外，实际上需要一个订阅者来执行其他操作。具体来说，需要实现[`Subscriber`](https://docs.rs/tracing/latest/tracing/trait.Subscriber.html) trait的东西。订阅者可以获取跟踪数据并执行它希望执行的操作。可以将它们写入`stdout`、写入文件、收集并执行聚合操作，或通过Open Telemetry发送到另一个服务。

[`tracing-subscriber`](https://docs.rs/tracing-subscriber) crate提供了许多订阅者实现。从外部看，这主要是将跟踪数据写入文件句柄的不同方式。然而，[`tracing-subscriber`](https://docs.rs/tracing-subscriber)的真正核心是[注册表](https://docs.rs/tracing-subscriber/0.3.18/tracing_subscriber/registry/index.html)。注册表是一个订阅者，实现了一个跨度存储，并允许多个层连接到它。

什么是[`Layer`](https://docs.rs/tracing-subscriber/0.3.18/tracing_subscriber/layer/trait.Layer.html)？长时间以来，我真的很难理解层的概念。从文档中可以看出，层是一个用于构建订阅者的可组合抽象。然而，我很难理解我可能希望如何组合层。这也令人困惑，因为层不像[`tower`](https://docs.rs/tower)层那样相互影响（`tower`层类似中间件，一个层所做的影响下一个层接收到的内容）。

可以将层（layer）看作是迷你订阅者。它们可以对[`Layer`](https://docs.rs/tracing-subscriber/0.3.18/tracing_subscriber/layer/trait.Layer.html) trait的某些方法采取行动，但对于它们不感兴趣的事物可以回退到默认实现。而[`Layer`](https://docs.rs/tracing-subscriber/0.3.18/tracing_subscriber/layer/trait.Layer.html)对于所有事物都有一个默认实现。

大多数层需要存储有关跨度的信息，这就是[注册表](https://docs.rs/tracing-subscriber/0.3.18/tracing_subscriber/registry/index.html)的作用（特别是通过[`LookupSpan`](https://docs.rs/tracing-subscriber/0.3.18/tracing_subscriber/registry/trait.LookupSpan.html) trait）。层可以将自己的数据存储在注册表中，以跨度扩展的形式。

为什么将这些数据存储在注册表中的重要性可能不是立即显而易见的。

[`tracing`](https://docs.rs/tracing)本身**不会**存储这些数据。这使得[`tracing`](https://docs.rs/tracing)不需要为数据分配内存，因此可以在[`no_std`](https://docs.rust-embedded.org/book/intro/no-std.html)环境中使用，也可以在我们习惯的大型服务器和开发机器上使用。

为了更清楚地说明，这里有一个例子。当创建一个跨度时，[`Subscriber`](https://docs.rs/tracing/latest/tracing/trait.Subscriber.html)会接收到对[`new_span()`](https://docs.rs/tracing/0.1.40/tracing/trait.Subscriber.html#tymethod.new_span)的调用。这包括跨度[`Attributes`](https://docs.rs/tracing/0.1.40/tracing/span/struct.Attributes.html)，使订阅者可以访问有关该跨度的所有信息。它的元数据、字段名称，以及在创建时设置的任何字段的值。

这太棒了，这是我们所需要的一切！

现在让我们看看当一个跨度进入（变为活动状态）时调用的方法，这称为[`enter()`]，它所带来的只是...一个[`span::Id`](https://docs.rs/tracing/0.1.40/tracing/span/struct.Id.html)。没有元数据，没有字段名称，当然也没有字段值。这种模式重复出现在调用跨度退出或关闭时的trait方法中。

使用注册表存储层可能稍后需要的任何数据是解决方案。这使得[`fmt::Subscriber`](https://docs.rs/tracing-subscriber/0.3.18/tracing_subscriber/fmt/struct.Subscriber.html)能够打印出事件祖先中每个跨度的完整数据。

现在我们已经了解了订阅者和层是什么，让我们来实现其中的一些内容吧！

## ari-subscriber

为了满足我上述使用案例的需求，我编写了[`ari-subscriber`](https://docs.rs/ari-subscriber/0.0.1/ari_subscriber/index.html)包。当前版本为0.0.1，这表明它可能有点粗糙，但到目前为止，它已经帮助我快速确定了Tokio的版本，之后的`yield_now()`[在Tokio控制台中不被检测为自我唤醒](https://github.com/tokio-rs/console/issues/512)。

“ari-subscriber”中的“ari”代表“异步运行时工具”。

界面简单，你需要将`ari-subscriber`层传递给[注册表](https://docs.rs/tracing-subscriber/0.3.18/tracing_subscriber/registry/index.html)：

```
use tracing_subscriber::prelude::*; tracing_subscriber::registry()
 .with(ari_subscriber::layer())
 .init(); 
```

这将输出到`stdout`（目前不可配置）。输出将具有漂亮的颜色！

让我们看一个简单的例子，展示如何使用`ari-subscriber`。以下是我们将使用的Rust代码：

```
#[tokio::main] async fn main() {
  // Set up subscriber
  use tracing_subscriber::prelude::*;
 tracing_subscriber::registry() .with(ari_subscriber::layer())
 .init();    // Spawn a task and wait for it to complete
 tokio::spawn(async { tracing::info!(fun = true, "pre-yield");
 tokio::task::yield_now().await; }) .await .unwrap(); } 
```

我们在异步上下文中开始（使用`#[tokio::main]`属性）。首先，我们使用注册表设置`ari-subscriber`层。然后，我们生成一个任务并等待其完成。任务发出跟踪事件，然后通过调用来自Tokio的[`yield_now()`](https://docs.rs/tokio/1.35.1/tokio/task/fn.yield_now.html)函数将控制返回给运行时。之后结束。

如果你仔细观察（并且关注我随处散播的所有链接），你可能已经意识到我在看[问题console#512](https://github.com/tokio-rs/console/issues/512)中描述的情况。我们要看的是唤醒操作发生的地方。

我们将修复我们的 Tokio 版本到一个旧版本，在这个版本中，我们知道 Tokio Console 检测到对 [`yield_now()`](https://docs.rs/tokio/1.35.1/tokio/task/fn.yield_now.html) 的等待作为自唤醒。因此，让我们在 `Cargo.toml` 中指定以下内容：

```
[dependencies] ari-subscriber = "0.0.1" tokio = { version = "=1.22.0", features = ["full", "tracing"] } tracing = "0.1.40" tracing-subscriber = "0.3.18" 
```

我们将 Tokio 的版本设置为 `=1.22.0`，这表明我们想要确切这个版本。默认情况下，`cargo` 将使用任何 `1.x` 版本，其中 `x` 大于或等于 22。

现在让我们看一下输出（稍作截断以删除我们不关注的内容）。

```
**2024-01-30**T**15:43:24**.010351Z TRACE runtime.spawn[**1**]{kind=task, task.name=, task.id=18, loc.file="src/main.rs", loc.line=10, loc.col=5} **new**
**2024-01-30**T**15:43:24**.010695Z TRACE runtime.spawn[**1**]{kind=task, task.name=, task.id=18, loc.file="src/main.rs", loc.line=10, loc.col=5} **enter**
**2024-01-30**T**15:43:24**.010778Z  INFO runtime.spawn[**1**]{kind=task, task.name=, task.id=18, loc.file="src/main.rs", loc.line=10, loc.col=5} **debugging_tokio_instrumentation**: fun=true pre-yield
**2024-01-30**T**15:43:24**.010829Z TRACE runtime.spawn[**1**]{kind=task, task.name=, task.id=18, loc.file="src/main.rs", loc.line=10, loc.col=5} **tokio::task::waker**: op="waker.wake_by_ref", task.id=1
**2024-01-30**T**15:43:24**.010878Z TRACE runtime.spawn[**1**]{kind=task, task.name=, task.id=18, loc.file="src/main.rs", loc.line=10, loc.col=5} **exit**
**2024-01-30**T**15:43:24**.010924Z TRACE runtime.spawn[**1**]{kind=task, task.name=, task.id=18, loc.file="src/main.rs", loc.line=10, loc.col=5} **enter**
**2024-01-30**T**15:43:24**.010962Z TRACE runtime.spawn[**1**]{kind=task, task.name=, task.id=18, loc.file="src/main.rs", loc.line=10, loc.col=5} **exit**
**2024-01-30**T**15:43:24**.010997Z TRACE runtime.spawn[**1**]{kind=task, task.name=, task.id=18, loc.file="src/main.rs", loc.line=10, loc.col=5} **enter**
**2024-01-30**T**15:43:24**.011032Z TRACE runtime.spawn[**1**]{kind=task, task.name=, task.id=18, loc.file="src/main.rs", loc.line=10, loc.col=5} **exit**
**2024-01-30**T**15:43:24**.011065Z TRACE runtime.spawn[**1**]{kind=task, task.name=, task.id=18, loc.file="src/main.rs", loc.line=10, loc.col=5} **close** 
```

不幸的是它在这个网站上显示得太宽了。但让我们逐步讲解一下。

日期和时间以及日志级别非常直观。我从 [`fmt::Subscriber`](https://docs.rs/tracing-subscriber/0.3.18/tracing_subscriber/fmt/struct.Subscriber.html) 中获取了日志级别的颜色，所以这些应该很熟悉。

### 跟踪类型

输出中所有行都以名为 `runtime.spawn` 的 span 为前缀。具有此名称的 spans 仪表化任务，`ari-subscriber` 以绿色标记它们。Tokio 中有不同类型的仪表化，它们各自有自己的颜色。

+   runtime.spawn spans（绿色）仪表化任务

+   runtime.resource spans（红色）仪表化资源

+   runtime.resource.async_op spans（蓝色）仪表化异步操作

+   runtime.resource.async_op.poll spans（黄色）仪表化异步操作的单个轮询

+   tokio::task::waker events（紫色）表示离散唤醒操作

+   runtime::resource::poll_op events（橙色）表示轮询状态变化。

+   runtime::resource::state_update events（粉色）表示资源状态变化

+   runtime::resource::async_op::state_update events（青绿色）表示异步操作状态变化

在 spans 的情况下，上面给出的值是 span 名称，对于 events 则是目标。

描述如何在 Tokio 中使用每个跟踪信息以及如何解释它们将填充更多帖子，我在这里不会详细介绍该主题。我已经写了一篇关于任务仪表化的帖子，其中涵盖了 `runtime.spawn` spans 和 `tokio::task::waker` events。去阅读 [tracing tokio tasks](https://hegdenu.net/posts/tracing-tokio-tasks/) 以了解更多！

### span events

现在让我们回到 `ari-subscriber` 对我们的测试程序的输出。第一行以 **new** 结束，这是表示创建新 span 的事件。`enter`、`exit` 和 `close` 都有对应的行，它们是 span 生命周期的组成部分。参见我上面链接的帖子中的 [span lifecycle](https://hegdenu.net/posts/tracing-tokio-tasks/#span-lifecycle) 部分，以便再次了解生命周期。

默认情况下，[`fmt::Subscriber`](https://docs.rs/tracing-subscriber/0.3.18/tracing_subscriber/fmt/struct.Subscriber.html) 不会输出这些 "span events"，但可以通过构建器上的 [`with_span_events()`](https://docs.rs/tracing-subscriber/latest/tracing_subscriber/fmt/struct.SubscriberBuilder.html#method.with_span_events) 方法配置它来输出。目前 `ari-subscriber` 总是发出这些 span 事件，但将来可能希望将其配置为减少输出量。

### 分析唤醒

让我们找到我们的唤醒操作。你会注意到在 INFO 级别上正好有一行。这是我们自己添加到生成任务的那一行。在 `runtime.spawn` span 之后，我们看到文本

```
debugging_tokio_instrumentation: fun=true pre-yield 
```

第一部分（`debugging_tokio_instrumentation`）是目标，其默认情况下与模块路径相同，因此是我们示例应用程序的名称。冒号后是字段（只有一个字段：`fun=true`），最后是消息（`pre-yield`）。一个事件的消息实际上仅是一个特殊处理的字段，名称为 `message`。这个事件没有着色，因为它不是 `ari-subscriber` 知道的 instrumentation 的一部分。

下一行是唤醒操作（它是紫色的！）。我们可以看到其目标是 `tokio::task::waker`，然后它有 2 个字段和没有消息。字段是 `op="waker.wake_by_ref"` 和 `task.id=1`。

让我们从第二个字段开始，`task.id=1`。这给出了被唤醒任务的**instrumentation ID**。任务的 instrumentation ID 不是 Tokio 的 [`task::Id`](https://docs.rs/tokio/1.35.1/tokio/task/struct.Id.html)，而是跟踪的 [`span::Id`](https://docs.rs/tracing/0.1.40/tracing/span/struct.Id.html)，它是仪器化该任务的 span 的标识。该值出现在 span 名称 `runtime.spawn` 后面的括号中（例如 `[1]`）。这有点令人困惑，因为 `runtime.spawn` span 还有一个名为 `task.id` 的字段，但那个是指 Tokio 任务 ID。这里的重要点是我们的 span ID 匹配（都是 1），因此这个操作是从受影响的任务内部执行的。

操作`wake_by_ref`表示使用对 waker 的引用来唤醒任务。这个操作不会消耗 waker - 这在 Tokio Console 计算给定任务的唤醒次数时很重要，以确保它没有失去所有的唤醒者。

有了这些信息，我们现在可以手动确定这是一个自唤醒操作。我们在运行该任务时唤醒了一个任务。

### 接下来会发生什么

让我们将 Tokio 的版本更改为最新版本（在撰写时），1.35.1。

```
tokio = { version = "=1.35.1", features = ["full", "tracing"] } 
```

然后运行完全相同的示例。输出如下（像之前一样被截断）。

```
**2024-01-30**T**16:00:09**.484496Z TRACE runtime.spawn[**1**]{kind=task, task.name=, task.id=18, loc.file="src/main.rs", loc.line=10, loc.col=5} **new**
**2024-01-30**T**16:00:09**.484798Z TRACE runtime.spawn[**1**]{kind=task, task.name=, task.id=18, loc.file="src/main.rs", loc.line=10, loc.col=5} **enter**
**2024-01-30**T**16:00:09**.484867Z  INFO runtime.spawn[**1**]{kind=task, task.name=, task.id=18, loc.file="src/main.rs", loc.line=10, loc.col=5} **debugging_tokio_instrumentation**: fun=true pre-yield
**2024-01-30**T**16:00:09**.484930Z TRACE runtime.spawn[**1**]{kind=task, task.name=, task.id=18, loc.file="src/main.rs", loc.line=10, loc.col=5} **tokio::task::waker**: op="waker.clone", task.id=1
**2024-01-30**T**16:00:09**.484998Z TRACE runtime.spawn[**1**]{kind=task, task.name=, task.id=18, loc.file="src/main.rs", loc.line=10, loc.col=5} **exit**
**2024-01-30**T**16:00:09**.485073Z TRACE **tokio::task::waker**: op="waker.wake", task.id=1
**2024-01-30**T**16:00:09**.485150Z TRACE runtime.spawn[**1**]{kind=task, task.name=, task.id=18, loc.file="src/main.rs", loc.line=10, loc.col=5} **enter**
**2024-01-30**T**16:00:09**.485208Z TRACE runtime.spawn[**1**]{kind=task, task.name=, task.id=18, loc.file="src/main.rs", loc.line=10, loc.col=5} **exit**
**2024-01-30**T**16:00:09**.485261Z TRACE runtime.spawn[**1**]{kind=task, task.name=, task.id=18, loc.file="src/main.rs", loc.line=10, loc.col=5} **enter**
**2024-01-30**T**16:00:09**.485313Z TRACE runtime.spawn[**1**]{kind=task, task.name=, task.id=18, loc.file="src/main.rs", loc.line=10, loc.col=5} **exit**
**2024-01-30**T**16:00:09**.485361Z TRACE runtime.spawn[**1**]{kind=task, task.name=, task.id=18, loc.file="src/main.rs", loc.line=10, loc.col=5} **close** 
```

这可能不是立即明显的，但该输出比以前的输出多出一行。可能首先引起注意的是，现在我们可以看到唤醒操作而无需向右滚动。但首先，让我们检查在那之上发生了什么。

就在我们自己的`fun=true pre-yield`事件行的正下方，我们看到仍然有一个`tokio::task::waker`事件，并且它仍在同一个任务上操作（也是我们当前所在的任务），具有任务仪表 ID 为 1。然而，这不是一个唤醒操作，相反它具有字段值`op=waker.clone`。某处，该任务的唤醒器正在被克隆。

直接之后我们可以看到跨度退出 - 这意味着对该任务的轮询调用已返回。在此之后，任务**被唤醒**。我们看到操作是`waker.wake`而不是`waker.wake_by_ref`，这意味着唤醒器被消耗了（这是有道理的，因为之前它被克隆了）。比所有这些更重要的是，这次唤醒操作不在该任务的`runtime.spawn`跨度内，事实上它根本不在任何跨度内，无论是`runtime.spawn`还是其他的。

这证实了在 Tokio Console 中可以观察到的情况，仪表表明这不是自唤醒！

### 发生了什么变化？

这种改变的原因是 PR [`tokio#5223`](https://github.com/tokio-rs/tokio/pull/5223)（在 Tokio 本身）。这个 PR 改变了 [`yield_now()`](https://docs.rs/tokio/1.35.1/tokio/task/fn.yield_now.html) 的行为以推迟唤醒。当一个任务以这种方式向运行时让步时，它立即准备好再次轮询（这就是全部意义）。任何其他准备好的任务都会优先被轮询（除非涉及 LIFO 插槽的特定条件）。然而调度程序不一定会轮询资源驱动程序，这意味着一个始终准备好的任务可能会使资源驱动程序挨饿，尽管它尽最大努力定期向运行时让步。

这个 PR 改变了行为，推迟唤醒通过 [`yield_now()`](https://docs.rs/tokio/1.35.1/tokio/task/fn.yield_now.html) 调用的任务，直到轮询资源驱动程序后，避免了饥饿问题。

在[console#512](https://github.com/tokio-rs/console/issues/512)的一些讨论之后，我们决定 Tokio Console 无法检测到这种特定的自唤醒情况是可以接受的，因为 Tokio 上的 PR 大大减少了由其他 futures 的自唤醒可能导致的某些性能问题。

这就是我如何使用我非常基本的订阅器 crate 以更快的速度回答问题，多亏了漂亮的颜色。

## 我应该使用`ari-subscriber`吗？

现在我们已经稍微看了一下`ari-subscriber`，最重要的问题是，应该有人在使用它吗？

答案是**不**。

除了缺少一堆有用的功能外，`ari-subscriber`当前以“简单方式”执行许多操作，这并不高效。我知道如何使其更高效，但我承诺在对这个 crate 进行更多工作之前要先写这篇文章。

除非你也在尝试调试内置于Tokio中的工具，否则最好使用[`fmt::Subscriber`](https://docs.rs/tracing-subscriber/0.3.18/tracing_subscriber/fmt/struct.Subscriber.html)来自于`tracing-subscriber`包。

如果**你确实**在调试那个工具，请[过来打个招呼](https://hegdenu.net/about/#contact)! 我非常想听听你在做什么，说不定我还能帮上忙。

### 感谢

感谢[Luca Palmieri](https://lpalmieri.com/)和[One](https://github.com/c-git)对这篇文章的反馈！
