<!--yml
category: 未分类
date: 2024-05-27 14:43:44
-->

# Jonas Hietala: Exploring the Gleam FFI

> 来源：[https://www.jonashietala.se/blog/2024/01/11/exploring_the_gleam_ffi/](https://www.jonashietala.se/blog/2024/01/11/exploring_the_gleam_ffi/)

My brain is a curious thing. I’m on a business trip right now and I’ve set aside time to finish some important todos I want and need to get done. But instead of focusing on them, I started playing around with [Gleam](https://gleam.run/ "Gleam is a friendly language for building type-safe systems that scale!")—a young and interesting programming language.

My (current) favorite programming languages are [Rust](https://www.rust-lang.org/ "Rust programming language") and [Elixir](https://elixir-lang.org/ "Elixir programming language"). They’re really different from each other, but they work well together as Rust can easily be [embedded via Erlang NIFs](https://github.com/rusterlium/rustler). This is useful because one of the drawbacks with Elixir (or rather the Erlang VM that Elixir runs on) is raw computational performance, something that Rust excels at.

Another drawback with Elixir is the lack of typing (although [improvements are underway](https://nitter.net/josevalim/status/1744395345872683471)). This is what drew me to Gleam, which feels like Elixir but with Rust types on top—a fantastic sales pitch.

A big drawback with young languages is that there aren’t a lot of libraries out there, so you’ll have to roll your own solutions most of the time. But the beauty of targeting existing platforms is that you can also leverage their libraries, which in Gleam’s case means all Erlang and Elixir libraries (or JavaScript, depending on your compilation target). This is accomplished via Gleam’s FFI, which is quite convenient.

Let’s explore.

If you want to follow along, make sure you have the respective compilers installed. I’ll base the examples from a clean new repo:

As shown in [the Gleam Book](https://gleam.run/book/tour/external-functions.html) calling standard Erlang functions is straightforward. You declare foreign functions with the `@external` keyword and then call them as normal:

```
import gleam/io

@external(erlang, "rand", "uniform")
pub fn random_float() -> Float

pub fn main() {
  io.debug(random_float())
} 
```

That if run will call the `uniform` function in the `rand` Erlang module:

```
$ gleam run
0.43487935467166317 
```

This works for standard libraries, but you can access other Erlang libraries on [Hex](https://hex.pm/) by adding them as dependencies in `gleam.toml`:

```
[dependencies]
base32 = "~> 0.1.0" 
```

Declare and call it like before:

```
@external(erlang, "base32", "encode")
pub fn encode_base32(x: String) -> String

pub fn main() {
  io.debug(encode_base32("superhidden"))
} 
```

And Gleam will download and compile the Erlang dependency:

```
$ gleam run
  Compiling base32
===> Fetching rebar3_hex v7.0.7
===> Fetching hex_core v0.8.4
===> Fetching verl v1.1.1
===> Analyzing applications...
===> Compiling hex_core
===> Compiling verl
===> Compiling rebar3_hex
===> Analyzing applications...
===> Compiling base32
"ON2XAZLSNBUWIZDFNY======" 
```

If you want to write Erlang code yourself and call that, it’s also very easy. The gleam compiler will compile and include `.erl` files automatically.

For example this file `src/erlib.erl`:

```
-module(erlib).
-export([ping/0]).

ping() ->
    io:fwrite("ping~n", []). 
```

Declare and call the function in gleam like before:

```
@external(erlang, "erlib", "ping")
pub fn ping() -> a

pub fn main() {
  ping()
} 
```

You can also call Gleam functions from Erlang. With for example this pong function in `src/mypong.gleam`:

```
import gleam/io

pub fn pong() {
  io.println("pong")
} 
```

You can call the Gleam function with `module:function()`:

```
ping() ->
    io:fwrite("ping from Erlang~n", []),
    mypong:pong(). 
```

```
$ gleam run
ping from Erlang
pong 
```

If you want to include Gleam code into an existing Elixir project, look at [MixGleam](https://github.com/gleam-lang/mix_gleam) on how to teach `mix` how to work with Gleam code and dependencies. But if you want to include Elixir code into your Gleam project, you can do that just as easily as with Erlang.

Declaring foreign Elixir functions uses the same `@external` keyword:

```
import gleam/io

@external(erlang, "Elixir.RandomColor", "hex")
pub fn random_color() -> String

pub fn main() {
  io.println(random_color())
} 
```

Note that Elixir modules gets an `Elixir` prefix and that we’re still calling external Erlang code as Elixir gets compiled to Erlang.

Dependencies are added from [Hex](https://hex.pm/), exactly the same as with Erlang dependencies:

```
$ gleam add random_color
$ gleam run
#3724C9 
```

Calling our own Elixir code is just as easy as with Erlang. Just include it in your source directory, like this `src/exlib.ex`:

```
defmodule Exlib do
 def ping() do
    IO.puts("ping from Elixir")
    :mypong.pong()
  end
end 
```

And refer to the module as `Elixir.Exlib`:

```
@external(erlang, "Elixir.Exlib", "ping")
pub fn ping() -> a

pub fn main() {
  ping()
} 
```

```
$ gleam run
ping from Elixir
pong 
```

Keep in mind that:

1.  You can’t directly call Elixir macros (they happen on compile time).
2.  If you include Elixir you’ll also include the Elixir standard library. Erlang might be a better choice if you want something more lightweight.

In the beginning I wrote that you could easily call Rust code from Elixir via [rustler](https://github.com/rusterlium/rustler). You can call Rust from Gleam as well, either via Erlang or Elixir. [Rustler](https://github.com/rusterlium/rustler) recommends to use `mix` with Elixir, but we can do this with Erlang and keep using the Gleam toolchain.

First we need to create a Rust project somewhere. We can create it in the top level of the Gleam repo or in a `native/` folder or wherever. For this example I’ll just leave it at the top:

On the Rust side I’ll use [rustler](https://github.com/rusterlium/rustler) to create the NIF:

```
$ cd rslib/
$ cargo add rustler 
```

And in `rslib/src/lib.rs` we’ll tag the function we want to expose with [rustler](https://github.com/rusterlium/rustler):

```
#[rustler::nif]
pub fn truly_random() -> i64 {
    4 }

rustler::init!("librs", [truly_random]); 
```

We want to build this as a dynamic library, so we’ll need to update `Cargo.toml`:

```
[lib]
crate-type = ["dylib"] 
```

Then build it in release mode:

```
$ cargo build --release
  ... 
```

This should generate the library at `target/release/librslib.so`. Yes, my naming choice here sucked, but let’s roll with it.

For Gleam to be able to use this file we need to copy it to `priv/` (relative to the Gleam root, not the Rust project):

```
$ mkdir ../priv
$ cp target/release/librslib.so ../priv/ 
```

The file layout should look something like this:

```
├── README.md
├── build
├── gleam.toml
├── manifest.toml
├── priv
│   └── librslib.so     # The important part
├── rslib
│   ├── Cargo.lock
│   ├── Cargo.toml
│   ├── src
│   │   └── lib.rs
│   └── target
└── src
    └── myapp.gleam 
```

To include the library we’ll use some Erlang. We’ll use `src/rslib.erl` similar to before, but with [Erlang NIFs](https://www.erlang.org/doc/tutorial/nif.html):

```
-module(librs).
-export([truly_random/0]).
-nifs([truly_random/0]).
-on_load(init/0).

init() ->
    ok = erlang:load_nif("priv/librslib", 0).

truly_random() ->
    exit(nif_library_not_loaded). 
```

We now load the `librslib` library, declare `truly_random` as a NIF with `-nifs(..)` and add a function placeholder that will be replaced when the library loads. librs

With the Erlang code in place what’s left is to call the Erlang function from Gleam:

```
import gleam/io

@external(erlang, "librs", "truly_random")
pub fn truly_random() -> String

pub fn main() {
  io.debug(truly_random())
} 
```

And now we can call our [truly random](https://xkcd.com/221/) Rust function!

While Rust is amazing, and it’s great that we can call it from Gleam, there are some drawbacks to be aware of:

1.  The glue is verbose as we need to duplicate the function name a bunch of times.

2.  Building the library and moving it to `priv/` isn’t handled by the Gleam toolchain. I’ve seen some people use a Makefile to build the library and then copy it to `priv/`.

3.  NIFs can crash the whole runtime. While Rust is safer than C, it can still bring the whole runtime down and not just a single Erlang process.

Calling Gleam code from Rust seems harder. I found an example of [calling Elixir or Erlang function from Rust](https://github.com/Qqwy/elixir-rustler_elixir_fun "Calling Elixir code from Rust"), but I haven’t looked at in more detail.

Gleam can also compile to JavaScript, and this blog post wouldn’t be complete without a short example.

To compile for JavaScript we can add `target = javascript` to our `gleam.toml` or use the target flag:

```
$ gleam run --target javascript 
```

Like before when we called Erlang and Elixir files, if you add a JavaScript source file in `src/` it will automatically get included. For example this `src/jslib.mjs`:

```
import * as gleam from "./mypong.mjs";

export function ping() {
  console.log("ping from JavaScript");
  gleam.pong();
} 
```

And to declare the foreign function, we now use the relative file instead of the library name:

```
@external(javascript, "./jslib.mjs", "ping")
pub fn ping() -> a

pub fn main() {
  ping()
} 
```

And when run you’ll see that we can call the JavaScript `ping` function which in turns calls our Gleam `pong` function:

```
$ gleam run --target javascript
ping from JavaScript
pong 
```

You can inspect the output JavaScript files in `build/dev/javascript/myapp/`, which is very useful when debugging or trying to understand the code Gleam generates.

I hope this post gives helps you understand how to get started with Gleam’s FFI system. I haven’t looked into Gleam more than a few days, but given how easy it is to get access to my other favorite languages and their ecosystems, I don’t have to worry that I’ll be missing out on some crucial functionality if I’m going to create something real with Gleam.