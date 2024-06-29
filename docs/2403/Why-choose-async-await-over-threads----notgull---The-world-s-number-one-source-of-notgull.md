<!--yml

类别：未分类

日期：2024-05-29 12:40:25

-->

# 为什么选择 async/await 而不是线程？– notgull – notgull 的世界第一来源

> 来源：[https://notgull.net/why-not-threads/](https://notgull.net/why-not-threads/)

一个常见的说法是，线程可以做任何`async`/`await`可以做的事情，但更简单。那么为什么有人会选择`async`/`await`呢？

这是我在 Rust 社区经常看到的一个常见问题。老实说，我完全理解这个问题的出发点。

Rust 是一种低级语言，不会向你隐藏协程的复杂性。这与像 Go 这样的语言相反，后者默认情况下发生`async`，程序员甚至无需考虑它。

聪明的程序员试图避免复杂性。因此，他们看到`async`/`await`中的额外复杂性，质疑为什么需要它。当考虑到操作系统线程中存在一个合理的替代方案时，这个问题尤为重要。

让我们通过`async`来进行一次思维之旅，看看它如何堆叠。

## 背景闪电战

Rust 是一种低级语言。通常，代码是线性的；一件事情接着一件事情发生。看起来像这样：

```
fn main() {
    foo();
    bar();
    baz();
} 
```

简单明了，对吧？

但有时你会希望同时运行许多事情。这个典型的例子是 web 服务器。考虑以下线性代码的例子：

```
fn main() -> io::Result<()> {
    let socket = TcpListener::bind("0.0.0.0:80")?;

    loop {
        let (client, _) = socket.accept()?;
        handle_client(client)?;
    }
} 
```

想象一下，如果`handle_client`需要几毫秒，并且两个客户端同时尝试连接到你的 web 服务器。你会遇到严重的问题！

+   客户端 #1 连接到 web 服务器，并被`accept()`函数接受。它开始运行`handle_client()`。

+   客户端 #2 连接到 web 服务器。然而，由于当前未运行`accept()`，我们必须等待客户端 #1 的`handle_client()`运行完毕。

+   等待几毫秒后，我们回到`accept()`。客户端 #2 可以连接。

现在想象一下，不是两个客户端，而是两百万个同时的客户端。在队列的末尾，你将不得不等待几分钟，然后 web 服务器才能帮助你。这很快就变得不可伸缩。

显然，最初的网络尝试解决这个问题。最初的解决方案是引入线程。通过将一些寄存器的值和程序的堆栈保存到内存中，操作系统可以停止一个程序，运行另一个程序，然后稍后继续运行该程序。本质上，它允许多个例程（或“线程”或“进程”）在同一CPU上运行。

使用线程，我们可以将上述代码重写如下：

```
fn main() -> io::Result<()> {
    let socket = TcpListener::bind("0.0.0.0:80")?;

    loop {
        let (client, _) = socket.accept()?;
        thread::spawn(move || handle_client(client));
    }
} 
```

现在，客户端被一个与等待新连接处理的线程分开处理。太棒了！这通过允许并发线程访问避免了问题。

+   客户端 #1 被服务器`accept`。服务器生成一个调用`handle_client`的线程。

+   客户端 #2 试图连接服务器。

+   最终，`handle_client`在某些事情上阻塞。操作系统保存处理客户端 #1 的线程，并恢复主线程。

+   主线程 `accept` 客户端#2。它启动一个单独的线程来处理客户端#2。只有几微秒的延迟，客户端#1和客户端#2并行运行。

当考虑到生产级别的 Web 服务器拥有数十个 CPU 内核时，线程特别有效。重要的不仅仅是操作系统可以给出这些线程同时运行的*假象*；重要的是操作系统*实际上*可以使它们全部同时运行。

最终，出于稍后我将详细说明的原因，程序员们希望将这种并发性从操作系统空间带入用户空间。有许多不同的用户空间并发模型。有事件驱动编程、actors 和协程。Rust 选择的是 `async`/`await`。

简单来说，您将程序编译为一组可以彼此独立运行的状态机。Rust 本身提供了创建状态机的机制；`async` 和 `await` 的机制。以上程序在 `async`/`await` 方面将如下所示，使用 [`smol`](https://crates.io/crates/smol) 编写：

```
#[apply(smol_macros::main!)]
async fn main(ex: &smol::Executor) -> io::Result<()> {
    let socket = TcpListener::bind("0.0.0.0:80").await?;

    loop {
        let (client, _) = socket.accept().await?;
        ex.spawn(async move {
            handle_client(client).await;
        }).detach();
    }
} 
```

+   主函数前面带有 `async` 关键字。这意味着它不是传统的函数，而是返回一个状态机。大致上，函数内容对应于该状态机。

+   `await` 包含另一个状态机作为当前运行状态机的一部分。对于 `accept()` 来说，这意味着状态机将其作为一个步骤包含在内。

+   最终，其中一个内部函数会*让步*，或放弃控制。例如，当 `accept()` 等待新连接时。此时整个状态机将将执行让给更高级别的执行器。对于我们来说，那就是 `smol::Executor`。

+   一旦执行被让步，`Executor` 将用 `spawn` 函数并发运行另一个当前状态机来替换当前状态机。

+   我们将一个 `async` 块传递给 `spawn` 函数。此块表示一个全新的状态机，独立于 `main` 函数创建的状态机。此状态机的唯一功能是运行 `handle_client` 函数。

+   一旦 `main` 让步，会选择一个客户端来替代它运行。一旦该客户端让步，循环重复。

+   现在，您可以处理数百万个同时客户端。

当然，像这样的用户空间并发引入了复杂性上升。当您使用线程时，您不必处理执行器、任务、状态机等。

如果你是一个合理的人，你可能会问：“为什么我们需要做所有这些呢？线程运行良好；对于99%的程序，我们不需要涉及任何用户空间并发。引入新的复杂性是技术债务，技术债务会浪费我们的时间和金钱。”

“那么为什么我们不使用线程呢？”

## 超时问题

Rust 的一个最大优势之一是*可组合性*。它提供了一组可以嵌套、构建、组合和扩展的抽象。

我记得让我坚持使用 Rust 的*东西*是 [`Iterator`](https://doc.rust-lang.org/std/iter/trait.Iterator.html) trait。让我震惊的是，你可以将某物变成 [`Iterator`](https://doc.rust-lang.org/std/iter/trait.Iterator.html)，应用一些不同的组合器，然后将结果 [`Iterator`](https://doc.rust-lang.org/std/iter/trait.Iterator.html) 传递给任何接受 [`Iterator`](https://doc.rust-lang.org/std/iter/trait.Iterator.html) 的函数。

它继续让我印象深刻。假设你想从另一个线程接收一个整数列表，仅获取即时可用的整数，丢弃任何不是偶数的整数，对所有整数加一，然后将它们推送到一个新列表中。

在其他语言中，这可能需要五十行代码和一个辅助函数。但在 Rust 中，只需五行代码即可完成：

```
let (send, recv) = mpsc::channel();
my_list.extend(
    recv.try_iter()
        .filter(|x| x & 1 == 0)
        .map(|x| x + 1)
); 
```

`async`/`await` 最好的一点是，它让你将这种可组合性应用于 I/O 绑定的函数。假设你有一个新的客户端需求；你想给上面的函数添加超时。假设我们的上面的 `handle_client` 函数看起来像这样：

```
async fn handle_client(client: TcpStream) -> io::Result<()> {
    let mut data = vec![];
    client.read_to_end(&mut data).await?;

    let response = do_something_with_data(data).await?
    client.write_all(&response).await?;

    Ok(())
} 
```

如果我们想添加，比如，三秒的超时，我们可以组合两个组合器来实现：

+   [`race`](https://docs.rs/smol/latest/smol/future/fn.race.html) 函数同时运行两个 futures。

+   [`Timer`](https://docs.rs/smol/latest/smol/struct.Timer.html) future 在返回之前等待一段时间。

最终代码如下所示：

```
async fn handle_client(client: TcpStream) -> io::Result<()> {
    // Future that handles the actual connection.
    let driver = async move {
        let mut data = vec![];
        client.read_to_end(&mut data).await?;

        let response = do_something_with_data(data).await?
        client.write_all(&response).await?;

        Ok(())
    };

    // Future that handles waiting for a timeout.
    let timeout = async {
        Timer::after(Duration::from_secs(3)).await;

        // We just hit a timeout! Return an error.
        Err(io::ErrorKind::TimedOut.into())
    };

    // Run both in parallel.
    driver.race(timeout).await
} 
```

我发现这是一个非常简单的过程。你所需做的就是在 `async` 块中包装你现有的代码，并与另一个 future 竞争。

这种方法的额外好处是它适用于任何类型的流。在这里，我们使用了 `TcpStream`。然而，我们可以轻松地用任何实现了 `impl AsyncRead + AsyncWrite` 的东西替换它。它可以是普通流上面的 GZIP 流，或者是 Unix 套接字，或者是文件。`async` 只是按照你从中需要的任何模式滑入。

## 主题线程

如果我们想在上面的线程示例中实现这个，我们会怎么做呢？

```
fn handle_client(client: TcpStream) -> io::Result<()> {
    let mut data = vec![];
    client.read_to_end(&mut data)?;

    let response = do_something_with_data(data)?
    client.write_all(&response)?;

    Ok(())
} 
```

嗯，这并不容易。通常情况下，在阻塞代码中，你不能中断 `read` 或 `write` 系统调用，而不做一些像关闭文件描述符（在 Rust 中无法做到的事情）那样灾难性的事情。

幸运的是，[`TcpStream`](https://doc.rust-lang.org/std/net/struct.TcpStream.html) 有两个函数 [`set_read_timeout`](https://doc.rust-lang.org/std/net/struct.TcpStream.html#method.set_read_timeout) 和 [`set_write_timeout`](https://doc.rust-lang.org/std/net/struct.TcpStream.html#method.set_write_timeout)，可以用来设置读取和写入的超时时间。然而，我们不能简单地朴素地使用它。想象一个客户端，每 2.9 秒发送一个字节，只是为了重置超时。

因此，我们必须在这里稍微防御性地编程。由于 Rust 组合器的强大功能，我们可以编写自己的类型包装，围绕 [`TcpStream`](https://doc.rust-lang.org/std/net/struct.TcpStream.html) 来编程超时。

```
// Deadline-aware wrapper around `TcpStream.
struct DeadlineStream {
    tcp: TcpStream,
    deadline: Instant
}

impl DeadlineStream {
    /// Create a new `DeadlineStream` that expires after some time.
    fn new(tcp: TcpStream, timeout: Duration) -> Self {
        Self {
            tcp,
            deadline: Instant::now() + timeout,
        }
    }
}

impl io::Read for DeadlineStream {
    fn read(&mut self, buf: &mut [u8]) -> io::Result<usize> {
        // Set the deadline.
        let time_left = self.deadline.saturating_duration_since(Instant::now());
        self.tcp.set_read_timeout(Some(time_left))?;

        // Read from the stream.
        self.tcp.read(buf)
    }
}

impl io::Write for DeadlineStream {
    fn write(&mut self, buf: &[u8]) -> io::Result<usize> {
        // Set the deadline.
        let time_left = self.deadline.saturating_duration_since(Instant::now());
        self.tcp.set_write_timeout(Some(time_left))?;

        // Read from the stream.
        self.tcp.write(buf)
    }
}

// Create the wrapper.
let client = DeadlineStream::new(client, Duration::from_secs(3));

let mut data = vec![];
client.read_to_end(&mut data)?;

let response = do_something_with_data(data)?
client.write_all(&response)?;

Ok(()) 
```

一方面，可以说这很优雅。我们利用 Rust 的能力用一个相对简单的组合器解决了问题。我相信这足够好用了。

另一方面，这显然是一种破解方式。

+   我们已经锁定自己使用 [`TcpStream`](https://doc.rust-lang.org/std/net/struct.TcpStream.html)。在 Rust 中没有抽象使用 [`set_read_timeout`](https://doc.rust-lang.org/std/net/struct.TcpStream.html#method.set_read_timeout) 和 [`set_write_timeout`](https://doc.rust-lang.org/std/net/struct.TcpStream.html#method.set_write_timeout) 类型的特性。因此，如果要使用任何类型的写入器，需要进行大量的额外工作。

+   它涉及一个额外的系统调用来设置超时。

+   我想象这种类型对于网络服务器所需的实际逻辑使用起来更加笨拙。

如果我在生产环境中看到这段代码，我会问作者为什么避免使用 `async`/`await` 来解决这个问题。这就是我在文章 “[为什么你实际上可能想在项目中使用 async](/why-you-want-async/)” 中描述的现象。我经常遇到这样一种模式，同步代码无法在不扭曲的情况下使用，所以我不得不将其重写为 `async`。

## Async 成功故事

HTTP 生态系统之所以将 `async`/`await` 作为其主要运行时机制，甚至用于客户端，这并非没有原因。你可以拿任何进行 HTTP 调用的函数，使其适应你想要的任何空白或使用案例。

[`tower`](https://crates.io/crates/tower) 可能是我能想到的这种现象的最佳例子，它真正地展示了 `async`/`await` 的强大之处。如果你将服务实现为一个 `async` 函数，你将获得超时、速率限制、负载均衡、[hedging](https://docs.rs/tower/0.4.13/tower/hedge/index.html) 和反压处理。所有这些功能都是免费的。

不管你使用什么运行时，或者你的服务实际在做什么，你都可以投入 [`tower`](https://crates.io/crates/tower) 来使其更加健壮。

[`macroquad`](https://docs.rs/macroquad) 是一个迷你的 Rust 游戏引擎，旨在尽可能简化游戏开发。它的主要功能使用 `async`/`await` 来运行引擎。这是因为 `async`/`await` 真的是 Rust 中表达需要停止以等待其他东西的线性函数的最佳方式。

在实践中，这可能非常强大。想象一下同时轮询游戏服务器和 GUI 框架的网络连接，使用同一个线程。这种可能性是无限的。

## 改善 Async 的形象

我认为问题不在于有些人认为线程比 `async` 更好。我认为问题在于 `async` 的好处没有广泛传播。这导致一些人对 `async` 的好处存在误解。

如果这是一个教育问题，我认为值得查看教育资料。这是[Rust Async Book](https://rust-lang.github.io/async-book/01_getting_started/02_why_async.html)在比较`async`/`await`与操作系统线程时的说法。

> **操作系统线程**不需要对编程模型进行任何更改，这使得表达并发非常容易。然而，在线程之间进行同步可能很困难，并且性能开销很大。线程池可以缓解部分成本，但无法完全支持大规模的 IO 密集型工作负载。
> 
> *- [Rust Async Book](https://rust-lang.github.io/async-book/01_getting_started/02_why_async.html)，各位作者*

我认为这是整个`async`社区中的一个一贯问题。当有人问“为什么我们要用这个而不是操作系统线程”时，人们往往会有点挥手说“`async`的开销更小。除此之外，一切都一样”。

这就是为什么 Web 服务器作者转向`async`/`await`的原因。这是他们解决[C10k 问题](https://en.wikipedia.org/wiki/C10k_problem)的方式。但这并不是所有人都转向`async`/`await`的原因。

性能增益是善变的，在错误的情况下可能会消失。有很多情况下，线程化的工作流程可能比等效的`async`工作流程更快（主要是在 CPU 绑定任务的情况下）。我认为作为一个社区，我们过分强调了`async` Rust的短暂性能优势，而淡化了其语义上的好处。

在最坏的情况下，它会导致人们将`async`/`await`视为“[一种奇怪的东西，只用于小众用例](https://shnatsel.medium.com/smoke-testing-rust-http-clients-b8f2ee5db4e6)”而不予理会。它应该被视为一种强大的编程模型，可以让您简洁地表达那些在同步 Rust 中无法用几十个线程和通道表达的模式。

我也认为有一种倾向是试图使`async` Rust在某种程度上“就像同步 Rust 一样”，这种做法促使负面比较。我所说的“倾向”，是指这是[Rust 项目的官方路线图](https://blog.rust-lang.org/inside-rust/2022/02/03/async-in-2022.html)，称“编写`async` Rust 代码应该与编写同步代码一样简单，除了偶尔需要用到的`async`和`await`关键字”。

我反对这种表述，因为这从根本上是不可能的。这就像在滑雪场举办披萨派对一样。当然，如果你真的很有天赋，你可能会完成99%的工作。但平均人会注意到一些差异，无论你多么优秀。

我们不应该试图通过迎合拒绝采纳另一种模式的程序员而将我们的模型强加于不友好的习语中。我们应该试图突显Rust的`async`/`await`生态系统的优势；其可组合性和强大性。我们应该努力使得在程序员追求并发时，`async`/`await`成为*默认*选择。与其试图让同步Rust和`async` Rust变得相同，我们应该拥抱它们的差异。

简而言之，我们不应该使用技术原因来为语义模型辩护。我们应该使用语义原因。
