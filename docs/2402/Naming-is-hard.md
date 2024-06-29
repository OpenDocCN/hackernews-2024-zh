<!--yml

category: 未分类

date: 2024-05-27 14:43:06

-->

# 取名很难

> 来源：[https://blog.dureuill.net/articles/too-dangerous-cpp/](https://blog.dureuill.net/articles/too-dangerous-cpp/)

一些模式仅因为Rust的内存安全而变得实用，并且在C++中使用会过于危险。这里有一个具体的例子。

* * *

在一个用Rust编写的内部库上工作时，我有一个我希望能够克隆的解析器错误类型，而不复制其中数据的需求。在Rust中，这需要使用像[Rc](https://doc.rust-lang.org/std/rc/struct.Rc.html)这样的引用计数指针。

所以我写了我的错误类型，并将其用作易错函数的错误变体，然后继续我的生活。

```
struct Error {
  data: Rc<ExpensiveToCloneDirectly>, }   pub type Response = Result<Output, Error>;   fn parse(input: Input) -> Response {
 todo!() } 
```

有时候后来，我们注意到在某些输入上解析需要很长时间来执行，所以我决定通过一个[通道](https://doc.rust-lang.org/std/sync/mpsc/fn.channel.html)将输入发送到另一个线程，然后通过另一个通道获取响应，这样长时间的命令就不会阻塞主线程。

```
enum Command {
 Input(Input), Exit, }   pub enum RequestStatus {
 Completed(Response), Running, }   pub struct Parser {
  command_sender: Sender<Command>,
  response_receiver: Receiver<(Input, Response)>,
  cached_result: HashMap<Input, RequestStatus>, }   impl Parser {
  pub fn new() -> Self {
  let (command_sender, command_receiver) = channel::<Command>();
  let (response_sender, response_receiver) = channel::<(Input, Response)>();   std::thread::spawn(move || loop {
  match command_receiver.recv() {
 Ok(Command::Input(input)) => {  let response = parse(input);
  let _ = response_sender.send((input, response));
 } Ok(Command::Exit) => break,
 Err(_) => break,
 } });    Self {
 command_sender, response_receiver, cached_result: HashMap::default(), } }    pub fn request_parsing(&mut self, input: Input) -> RequestStatus {
  // pump previously received responses
  while let Ok((input, response)) = self.response.receiver.try_recv() {
  self.cached_result
 .insert(input, RequestStatus::Completed(response));
 }    let response = match self.cached_result.entry(input) {
 Entry::Vacant(entry) => {  self.command_sender
 .send(Command::Input(entry.key()))
 .unwrap();
 entry.insert(RequestStatus::Running)
 } Entry::Occupied(entry) => entry.into_mut(),
 }; response.clone()
 } } 
```

然而，在进行这种更改时，我遇到了以下错误消息：

```
error[E0277]: `Rc<String>` cannot be sent between threads safely
 --> src/main.rs:58:32
 | 58 |               std::thread::spawn(move || loop {
 |  _____________------------------_^ | |             | | |             required by a bound introduced by this call 59 | | match command_receiver.recv() { 60 | |                     Ok(Command::Input(input)) => { 61 | | let response = maybe_make(input); ...   | 68 | |                 } 69 | |             });
 | |_____________^ `Rc<String>` cannot be sent between threads safely
 | = help: within `(&'static str, Result<worker::Output, worker::Error>)`, the trait `Send` is not implemented for `Rc<String>` note: required because it appears within the type `Error` --> src/main.rs:17:16
 | 17 | pub struct Error {
 |                ^^^^^ note: required because it appears within the type `Result<Output, Error>`
 --> /home/dureuill/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/result.rs:502:10 | 502 | pub enum Result<T, E> {
 |          ^^^^^^ = note: required because it appears within the type `(&str, Result<Output, Error>)`
 = note: required for `Sender<(&'static str, Result<worker::Output, worker::Error>)>` to implement `Send` note: required because it's used within this closure --> src/main.rs:58:32 | 58  |             std::thread::spawn(move || loop {
 |                                ^^^^^^^ note: required by a bound in `spawn`
 --> /home/dureuill/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/std/src/thread/mod.rs:683:8 | 680 | pub fn spawn<F, T>(f: F) -> JoinHandle<T>
 |        ----- required by a bound in this function ... 683 |     F: Send + 'static,
 |        ^^^^ required by this bound in `spawn` 
```

正如编译器很好地解释的那样，这是因为[`Rc`类型不支持在线程之间发送](https://doc.rust-lang.org/std/rc/struct.Rc.html#impl-Send-for-Rc%3CT,+A%3E)，这样做会导致数据竞争。确实，`Rc`中的引用计数并不是以原子方式处理的，这样使用就不是线程安全的，而是使用常规整数操作。

对于线程安全的引用计数，Rust提供了[另一种类型称为`Arc`](https://doc.rust-lang.org/std/sync/struct.Arc.html)，它使用了**a**tomic **r**eference **c**ounting。将代码修改为使用`Arc`只是一件简单的事情：

```
diff --git a/src/main.rs b/src/main.rs index 04ec0d0..fd4b447 100644 --- a/src/main.rs +++ b/src/main.rs @@ -3,9 +3,9 @@ use std::{io::Write, time::Duration};
 mod parse { use std::{ collections::{hash_map::Entry, HashMap}, -        rc::Rc,
 sync::{ mpsc::{channel, Receiver, Sender}, +            Arc,
 }, time::Duration, }; @@ -15,13 +15,13 @@ mod parse {   #[derive(Clone, Debug)] pub struct Error { -        data: Rc<ExpensiveToCloneDirectly>, +        data: Arc<ExpensiveToCloneDirectly>,
 }   impl Error { fn new(data: ExpensiveToCloneDirectly) -> Self { Self { -                data: Rc::new(data), +                data: Arc::new(data),
 } } } 
```

（[在线测试此代码](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=b1f40129f7a6c9baf77fde13a4156889)）

只要我不需要引用计数是原子的，我就可以使用`Rc`。当我需要线程安全时，编译器就强制我切换到`Arc`和原子引用计数的开销。这说明了“不为你不需要的东西付费”的旧原则。

这个原则对C++开发者也很重要，但与Rust形成鲜明对比的是，C++标准库中只有带有原子引用计数的共享指针，相当于`Arc`，而不是`Rc`。即使你不使用原子，你也要为它付出代价。曾考虑提供2个类，但被拒绝了，尤其是因为[被认为太危险了](https://stackoverflow.com/a/15140227/1614219)（"使用非同步的`shared_ptr`编写的代码可能最终被用于线程化代码，导致难以调试的问题，且没有警告"）。

因为Rust会在编译时捕捉这些问题，所以并不危险。

在某些 C++ 标准库实现中，有试图在一些有限情况下恢复失去性能的尝试（例如，整个程序不是多线程的情况下），[在微基准测试中效果非常滑稽](http://snf.github.io/2019/02/13/shared-ptr-optimization/)。

不幸的是，C++ 采用始终具有原子引用计数的预防措施，仍然不足以使 `shared_ptr` 在多线程环境中安全，因为人们应该注意一些惯用语境中的潜在问题。

这是一个比较微妙的问题，老实说我不认为我曾经遇到过这个问题，但我在这里提及它是为了清晰起见，因为有时人们会将它与第二个问题混淆。

你可以拿一个 `shared_ptr` 并通过调用其拷贝构造函数来安全地进行复制。然而，你不能在多个线程之间共享一个 `shared_ptr` 的单一实例。想象一下，有一个包含共享指针的结构体，在多个线程之间共享，以及在该结构体上有一个重新分配共享指针的方法。如果多个线程未同步地调用该方法，那么这将导致未定义行为。

显然，这个问题足以引起关注，以至于 C++20 添加了对 `std::atomic<std::shared_ptr>` 的[部分模板特化](https://en.cppreference.com/w/cpp/memory/shared_ptr/atomic2)。然而，我的建议是“不要那样做！”。相反，保持你的共享指针在单线程中，并根据需要向其他线程发送副本。

由于赋值要求具有独占引用或拥有的对象，Rust 静态地禁止向多个线程共享的 `Arc` 赋值，从而在编译时避免了此问题。

在 `shared_ptr` 中，只有引用计数是原子的，但指向的对象需要自己的同步机制，以便从不同的线程进行写入和读取。这是一个陷阱，因为把“`shared_ptr` 是一个线程安全的引用计数指针”简化为“`shared_ptr` 是一个线程安全的引用计数指针”是诱人的，而只有前者是真实的。

虽然对经验丰富的开发者来说这可能显而易见，但我在实际情况中经常看到这个问题，很可能是由初级开发者引起的 🙃，而从未被有经验的开发者通过重构代码引入线程 😇。

自然地，Rust 对 `Arc` 的内容[施加了相同的要求](https://doc.rust-lang.org/std/sync/struct.Arc.html#thread-safety)，但[多亏了](https://doc.rust-lang.org/std/sync/struct.Arc.html#impl-Send-for-Arc%3CT,+A%3E) `Send` 和 [`Sync`](https://doc.rust-lang.org/std/marker/trait.Sync.html) 特性，`Arc` 只提供对其内容的共享引用，对指向的对象进行未同步的写入和读取是一个编译时错误。

Rust 完全依靠借用检查器和其类型系统实现这一结果。这是我使用的唯一一种静态[防止数据竞争](https://doc.rust-lang.org/nomicon/races.html)的语言。
