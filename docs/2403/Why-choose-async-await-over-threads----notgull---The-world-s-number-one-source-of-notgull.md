<!--yml
category: 未分类
date: 2024-05-29 12:40:25
-->

# Why choose async/await over threads? – notgull – The world's number one source of notgull

> 来源：[https://notgull.net/why-not-threads/](https://notgull.net/why-not-threads/)

A common refrain is that threads can do everything that `async`/`await` can, but simpler. So why would anyone choose `async`/`await`?

This is a common question that I’ve seen a lot in the Rust community. Frankly, I completely understand where it’s coming from.

Rust is a low-level language that doesn’t hide the complexity of coroutines from you. This is in opposition to languages like Go, where `async` happens by default, without the programmer needing to even consider it.

Smart programmers try to avoid complexity. So, they see the extra complexity in `async`/`await` and question why it is needed. This question is especially pertinent when considering that a reasonable alternative exists in OS threads.

Let’s take a mind-journey through `async` and see how it stacks up.

## Background Blitz

Rust is a low-level language. Normally, code is linear; one thing runs after another. It looks like this:

```
fn main() {
    foo();
    bar();
    baz();
} 
```

Nice and simple, right?

However, sometimes you will want to run many things at once. The canonical example for this is a web server. Consider the following written in linear code:

```
fn main() -> io::Result<()> {
    let socket = TcpListener::bind("0.0.0.0:80")?;

    loop {
        let (client, _) = socket.accept()?;
        handle_client(client)?;
    }
} 
```

Imagine if `handle_client` takes a few milliseconds, and two clients try to connect to your webserver at the same time. You’ll run into a serious problem!

*   Client #1 connects to the webserver, and is accepted by the `accept()` function. It starts running `handle_client()`.
*   Client #2 connects to the webserver. However, since `accept()` is not currently running, we have to wait for `handle_client()` for Client #1 to finish running.
*   After waiting a few milliseconds, we get back to `accept()`. Client #2 can connect.

Now imagine that instead of two clients, there are two million simultaneous clients. At the end of the queue, you’ll have to wait several minutes before the web server can help you. It becomes un-scalable very quickly.

Obviously, the embryonic web tried to solve this problem. The original solution was to introduce threading. By saving the value of some registers and the program’s stack into memory, the operating system can stop a program, run another program in its place, then resume running that program later. Essentially, it allows for multiple routines (or “threads”, or “processes”) to run on the same CPU.

Using threads, we can rewrite the above code as follows:

```
fn main() -> io::Result<()> {
    let socket = TcpListener::bind("0.0.0.0:80")?;

    loop {
        let (client, _) = socket.accept()?;
        thread::spawn(move || handle_client(client));
    }
} 
```

Now, the client is being handled by a separate thread than the one handling waiting for new connections. Great! This avoids the problem by allowing concurrent thread access.

*   Client #1 is `accept`ed by the server. The server spawns a thread that calls `handle_client`.
*   Client #2 tries to connect to the server.
*   Eventually, `handle_client` blocks on something. The OS saves the thread handling Client #1 and brings back the main thread.
*   The main thread `accept`s Client #2\. It spawns a separate thread to handle Client #2\. With only a few microseconds of delay, Client #1 and Client #2 are run in parallel.

Threads work especially well when you consider that production-grade web servers have dozens of CPU cores. It’s not just that the OS can give the *illusion* that all of these threads run at the same time; it’s that the OS can *actually* make them all run at once.

Eventually, for reasons I’ll elaborate later, programmers wanted to bring this concurrency out of the OS space and into the user space. There are many different models for userspace concurrency. There is event-driven programming, actors, and coroutines. The one Rust settled on is `async`/`await`.

To oversimplify, you compile the program as a grab-bag of state machines that can all be run independently of another. Rust itself provides a mechanism for creating state machines; the mechanism of `async` and `await`. The above program in terms of `async`/`await` would look like this, written using [`smol`](https://crates.io/crates/smol):

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

*   The main function is preceded with the `async` keyword. This means that it is not a traditional function, but one that returns a state machine. Roughly, the function’s contents correspond to that state machine.
*   `await` includes another state machine as a part of the currently running state machine. For `accept()`, it means that the state machine will include it as a step.
*   Eventually, one of the inner functions will *yield*, or give up control. For example, when `accept()` waits for a new connection. At this point the entire state machine will yield its execution to the higher-level executor. For us, that is `smol::Executor`.
*   Once execution is yielded, the `Executor` will replace the current state machine with another one that is running concurrently, spawned through the `spawn` function.
*   We pass an `async` block to the `spawn` function. This block represents an entire new state machine, independent of the one created by the `main` function. All this state machine does is run the `handle_client` function.
*   Once `main` yields, one of the clients is selected to run in its place. Once that client yields, the cycle repeats.
*   You can now handle millions of simultaneous clients.

Of course, user-space concurrency like this introduces an uptick in complexity. When you’re using threads, you don’t have to deal with executors and tasks and state machines and all.

If you’re a reasonable person, you might be asking “why do we need to do all of this? Threads work well; for 99% of programs, we don’t need to involve any kind of user-space concurrency. Introducing new complexity is technical debt, and technical debt costs us time and money.

“So why wouldn’t we use threads?”

## Timeout Trouble

Perhaps one of Rust’s biggest strengths is *composability*. It provides a set of abstractions that can be nested, built upon, put together, and expanded upon.

I recall that *the* thing that made me stick with Rust is the [`Iterator`](https://doc.rust-lang.org/std/iter/trait.Iterator.html) trait. It blew my mind that you could make something an [`Iterator`](https://doc.rust-lang.org/std/iter/trait.Iterator.html), apply a handful of different combinators, then pass the resulting [`Iterator`](https://doc.rust-lang.org/std/iter/trait.Iterator.html) into any function that took an [`Iterator`](https://doc.rust-lang.org/std/iter/trait.Iterator.html).

It continues to impress me how powerful it is. Let’s say you want to receive a list of integers from another thread, only take the ones that are immediately available, discard any integers that aren’t even, add one to all of them, then push them onto a new list.

That would be fifty lines and a helper function in some other languages. In Rust it can be done in five:

```
let (send, recv) = mpsc::channel();
my_list.extend(
    recv.try_iter()
        .filter(|x| x & 1 == 0)
        .map(|x| x + 1)
); 
```

The best thing about `async`/`await` is that it lets you apply this composability to I/O-bound functions. Let’s say you have a new client requirement; you want to add a timeout to your above function. Assume that our `handle_client` above function looks like this:

```
async fn handle_client(client: TcpStream) -> io::Result<()> {
    let mut data = vec![];
    client.read_to_end(&mut data).await?;

    let response = do_something_with_data(data).await?
    client.write_all(&response).await?;

    Ok(())
} 
```

If we want to add, say, a three-second timeout, we can combine two combinators to do that:

*   The [`race`](https://docs.rs/smol/latest/smol/future/fn.race.html) function takes two futures and runs them at the same time.
*   The [`Timer`](https://docs.rs/smol/latest/smol/struct.Timer.html) future waits for some time before returning.

Here is what the final code looks like:

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

I find this to be a very easy process. All you have to do is wrap your existing code in an `async` block and race it against another future.

An added bonus of this approach is that it works with any kind of stream. Here, we use a `TcpStream`. However we can easily replace it with anything that implements `impl AsyncRead + AsyncWrite`. It could be a GZIP stream on top of the normal stream, or a Unix socket, or a file. `async` just slides into whatever pattern you need from it.

## Thematic Threads

What if we wanted to implement this in our threaded example above?

```
fn handle_client(client: TcpStream) -> io::Result<()> {
    let mut data = vec![];
    client.read_to_end(&mut data)?;

    let response = do_something_with_data(data)?
    client.write_all(&response)?;

    Ok(())
} 
```

Well, it’s not easy. Generally, you can’t interrupt the `read` or `write` system calls in blocking code, without doing something catastrophic like closing the file descriptor (which can’t be done in Rust).

Thankfully, [`TcpStream`](https://doc.rust-lang.org/std/net/struct.TcpStream.html) has two functions [`set_read_timeout`](https://doc.rust-lang.org/std/net/struct.TcpStream.html#method.set_read_timeout) and [`set_write_timeout`](https://doc.rust-lang.org/std/net/struct.TcpStream.html#method.set_write_timeout) that can be used to set the timeouts for reading and writing, respectively. However, we can’t just use it naively. Imagine a client that sends one byte every 2.9 seconds, just to reset the timeout.

So we have to program a little defensively here. Due to the power of Rust combinators, we can write our own type wrapping around the [`TcpStream`](https://doc.rust-lang.org/std/net/struct.TcpStream.html) to program the timeout.

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

On one hand, it could be argued that this is elegant. We used Rust’s capabilities to solve the problem with a relatively simple combinator. I’m sure it would work well enough.

On the other hand, it’s definitely hacky.

*   We’ve locked ourselves into using [`TcpStream`](https://doc.rust-lang.org/std/net/struct.TcpStream.html). There’s no trait in Rust to abstract over using the [`set_read_timeout`](https://doc.rust-lang.org/std/net/struct.TcpStream.html#method.set_read_timeout) and [`set_write_timeout`](https://doc.rust-lang.org/std/net/struct.TcpStream.html#method.set_write_timeout) types. So it would take a lot of additional work to make it use any kind of writer.
*   It involves an extra system call for setting the timeout.
*   I imagine this type is much more unwieldy to use for the kinds of actual logic that web servers demand.

If I saw this code in production, I would ask the author why they avoided using `async`/`await` to solve this problem. This is the phenomenon I was describing in my post “[Why you might actually want async in your project](/why-you-want-async/)”. Quite frequently I encounter a pattern where synchronous code can’t be used without contortion, so I have to rewrite it in `async`.

## Async Success Stories

There’s a reason why the HTTP ecosystem has adopted `async`/`await` as its primary runtime mechanism, even for clients. You can take any function that makes an HTTP call, and make it fit whatever hole or use case you want it to.

[`tower`](https://crates.io/crates/tower) is probably the best example of this phenomenon I can think of, and it’s really *the* thing that made me realize how powerful `async`/`await` can be. If you implement your service as an `async` function, you get timeouts, rate limiting, load balancing, [hedging](https://docs.rs/tower/0.4.13/tower/hedge/index.html) and back-pressure handling. All of that for free.

It doesn’t matter what runtime you used, or what you’re actually doing in your service. You can throw [`tower`](https://crates.io/crates/tower) at it to make it more robust.

[`macroquad`](https://docs.rs/macroquad) is a miniature Rust game engine that aims to make game development as easy as possible. Its main function uses `async`/`await` in order to run its engine. This is because `async`/`await` is really the best way in Rust to express a linear function that needs to be stopped in order to wait for something else.

In practice, this can be extremely powerful. Imagine simultaneously polling a network connection to your game server and your GUI framework, on the same thread. The possibilities are endless.

## Improving Async’s Image

I don’t think the issue is that some people think threads are better than `async`. I think the issue is that the benefits of `async` aren’t widely broadcast. This leads some people to be misinformed about the benefits of `async`.

If this is an educational problem, I think it’s worth taking a look at the educational material. Here’s what the [Rust Async Book](https://rust-lang.github.io/async-book/01_getting_started/02_why_async.html) says when comparing `async`/`await` to operating system threads.

> **OS threads** don’t require any changes to the programming model, which makes it very easy to express concurrency. However, synchronizing between threads can be difficult, and the performance overhead is large. Thread pools can mitigate some of these costs, but not enough to support massive IO-bound workloads.
> 
> *- [Rust Async Book](https://rust-lang.github.io/async-book/01_getting_started/02_why_async.html), various authors*

I think this is a consistent problem throughout the `async` community. When someone asks the question of “why do we want to use this over OS threads”, people have a tendency to kind of wave their hand and say “`async` has less overhead. Other than that, everything’s the same.”

This is the reason why web server authors switched to `async`/`await`. It’s how they solved the [C10k problem](https://en.wikipedia.org/wiki/C10k_problem). But, it’s not going to be the reason why everyone else switches to `async`/`await`.

Performance gains are fickle and can disappear in the wrong circumstances. There are plenty of cases where a threaded workflow can be faster than an equivalent `async` workflow (mostly, in the case of CPU bound tasks). I think that we, as a community, have over-emphasized the ephemeral performance benefits of `async` Rust while downplaying its semantic benefits.

In the worst case, it leads to people shrugging off `async`/`await` as “[a weird thing that you resort to for niche use cases](https://shnatsel.medium.com/smoke-testing-rust-http-clients-b8f2ee5db4e6)”. It should be seen as a powerful programming model that lets you succinctly express patterns that can’t be expressed in synchronous Rust without dozens of threads and channels.

I also think there’s a tendency to try to make `async` Rust “just like sync Rust” in a way that encourages negative comparison. By “tendency”, I mean that it’s [the stated roadmap for the Rust project](https://blog.rust-lang.org/inside-rust/2022/02/03/async-in-2022.html), saying that “that writing async Rust code should be as easy as writing sync code, apart from the occasional `async` and `await` keyword.”.

I reject this framing because it’s fundamentally impossible. It’s like trying to host a pizza party on a ski slope. Sure, you can probably get 99% of the way there, especially if you’re really talented. But there are differences that the average bear *will* notice, no matter how good you are.

We shouldn’t be trying to force our model into unfriendly idioms to appease programmers who refuse to adopt another type of pattern. We should be trying to highlight the strengths of Rust’s `async`/`await` ecosystem; its composability and its power. We should be trying to make it so `async`/`await` is the *default* choice whenever a programmer reaches for concurrency. Rather than trying to make sync Rust and `async` Rust the same, we should embrace the differences.

In short, we shouldn’t be using technical reasons to argue for a semantic model. We should be using semantic reasons.