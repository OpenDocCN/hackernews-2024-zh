<!--yml
category: 未分类
date: 2024-05-27 14:43:06
-->

# Naming is hard

> 来源：[https://blog.dureuill.net/articles/too-dangerous-cpp/](https://blog.dureuill.net/articles/too-dangerous-cpp/)

Some patterns are only made practical thanks to Rust's memory safety, and too dangerous to use in C++. Here's a concrete example.

 * * *

Working on an internal library written in Rust, I had an error type for a parser that I wanted to be [`Clone`able](https://doc.rust-lang.org/std/clone/trait.Clone.html), without duplicating the data inside. In Rust, this calls for a **r**eference-**c**ounted pointer, like [Rc](https://doc.rust-lang.org/std/rc/struct.Rc.html).

So I wrote my error type, used it as the error variant of the fallible functions, and moved on with my life.

```
struct Error {
  data: Rc<ExpensiveToCloneDirectly>, }   pub type Response = Result<Output, Error>;   fn parse(input: Input) -> Response {
 todo!() } 
```

Sometimes later, we noticed that parsing would take a long time to execute on some inputs, so I decided I'd send the input to another thread via a [channel](https://doc.rust-lang.org/std/sync/mpsc/fn.channel.html), and I'd get the response back through another channel, so that long orders wouldn't block the main thread.

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

However, while doing this change, I was greeted with the following error message:

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

As the compiler nicely explained, it is because the [`Rc` type does not support being sent between threads](https://doc.rust-lang.org/std/rc/struct.Rc.html#impl-Send-for-Rc%3CT,+A%3E), as doing so would cause data races. Indeed, the reference count in `Rc` is not manipulated in an atomic manner that would be thread safe, it is using regular integer operations.

For thread-safe reference counting, Rust offers [another type called `Arc`](https://doc.rust-lang.org/std/sync/struct.Arc.html), that uses **a**tomic **r**eference **c**ounting. Modifying the code to use `Arc` is a simple matter of:

```
diff --git a/src/main.rs b/src/main.rs index 04ec0d0..fd4b447 100644 --- a/src/main.rs +++ b/src/main.rs @@ -3,9 +3,9 @@ use std::{io::Write, time::Duration};
 mod parse { use std::{ collections::{hash_map::Entry, HashMap}, -        rc::Rc,
 sync::{ mpsc::{channel, Receiver, Sender}, +            Arc,
 }, time::Duration, }; @@ -15,13 +15,13 @@ mod parse {   #[derive(Clone, Debug)] pub struct Error { -        data: Rc<ExpensiveToCloneDirectly>, +        data: Arc<ExpensiveToCloneDirectly>,
 }   impl Error { fn new(data: ExpensiveToCloneDirectly) -> Self { Self { -                data: Rc::new(data), +                data: Arc::new(data),
 } } } 
```

([Test this code online](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=b1f40129f7a6c9baf77fde13a4156889))

As long as I didn't need reference counting to be atomic, I could use `Rc`. When I needed thread-safety, the compiler forced me to switch to `Arc` and the overhead of atomic reference counting. This is an illustration of the old principle of "don't pay for what you don't use".

This principle is dear to the heart of C++ developers too, yet in stark contrast to Rust, C++ only has shared pointers with atomic reference counting in its standard library, that is the equivalent to `Arc`, not `Rc`. You always pay for the atomic even if you don't use it. Providing 2 classes was considered, but rejected, notably because [it was deemed too dangerous](https://stackoverflow.com/a/15140227/1614219) ("Code written with the unsynchronized `shared_ptr` may end up being used in threaded code down the road, ending up causing difficult to debug problems with no warning").

Because Rust will catch these at compile time, it is not dangerous.

On some C++ standard library implementations, there are attempts to recover the lost performance in some limited situations (e.g. the program as a whole is not multi-threaded), [to hilarious effect on micro-benchmarks](http://snf.github.io/2019/02/13/shared-ptr-optimization/).

Unfortunately the precaution taken by C++ of always having an atomic reference count is still insufficient to make `shared_ptr` safe in a multi-threaded context, as one should pay attention to a couple of the proverbial footguns.

This is a bit of a subtle issue, and honestly I don't think I ever ran into that one, but I include it for clarity because sometimes people mix it with the second one.

You can take a `shared_ptr` and make a copy of it, calling its copy constructor, in a thread-safe way. What you cannot do, however, is share a single instance of a `shared_ptr` between multiple threads. Imagine having a struct containing a shared pointer that is shared between threads, and a method on that struct that reassigns the shared pointer. If that method is called unsynchronized by multiple threads, then this will result in undefined behavior.

Apparently, this is enough of an issue that C++20 added a [partial template specialization to `std::atomic<std::shared_ptr>`](https://en.cppreference.com/w/cpp/memory/shared_ptr/atomic2). My advice, though, would be "don't do that!". Instead, keep your shared pointer in a single thread, and send copies to other threads as needed.

Since assignment requires at an exclusive reference or an owned object, Rust statically forbids assigning to an `Arc` that is shared between multiple threads, avoiding the issue at compile time.

In a `shared_ptr`, only the reference counting is atomic, but the pointed-to object needs its own synchronization for writing and reading from different threads. This is a bit of a pitfall because it is tempting to simplify "`shared_ptr` is a thread-safely-referenced-counted pointer" to "`shared_ptr` is a thread-safe reference-counted pointer", while only the former is true.

While this may seem obvious to seasoned developers, I saw a lot more of this issue in the wild, probably always by junior developers 🙃 never by experienced developers refactoring their code to introduce threads 😇

Naturally, Rust [imposes the same requirement](https://doc.rust-lang.org/std/sync/struct.Arc.html#thread-safety) on the content of `Arc`, but [thanks](https://doc.rust-lang.org/std/sync/struct.Arc.html#impl-Send-for-Arc%3CT,+A%3E) to [the `Send`](https://doc.rust-lang.org/std/marker/trait.Send.html) and [the `Sync`](https://doc.rust-lang.org/std/marker/trait.Sync.html) traits, and `Arc` only providing shared reference to its contents, writing and reading the pointed-to object unsynchronized is a compile-time error.

Rust achieves this result entirely thanks to the borrow checker and its type system. It is the only language I used that can statically [prevent data races](https://doc.rust-lang.org/nomicon/races.html).