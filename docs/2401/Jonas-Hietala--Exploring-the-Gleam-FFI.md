<!--yml

category: 未分类

date: 2024-05-27 14:43:44

-->

# Jonas Hietala: 探索 Gleam FFI

> 来源：[https://www.jonashietala.se/blog/2024/01/11/exploring_the_gleam_ffi/](https://www.jonashietala.se/blog/2024/01/11/exploring_the_gleam_ffi/)

我的大脑是一个好奇的东西。我现在正在出差，本来应该抽出时间完成一些我想要并且需要完成的重要待办事项。但是我没有专注于它们，而是开始玩弄一个年轻而有趣的编程语言——[Gleam](https://gleam.run/ "Gleam is a friendly language for building type-safe systems that scale!")。

我（目前）最喜欢的编程语言是 [Rust](https://www.rust-lang.org/ "Rust programming language") 和 [Elixir](https://elixir-lang.org/ "Elixir programming language")。它们彼此之间非常不同，但它们在一起工作得很好，因为 Rust 可以很容易地通过 Erlang NIFs [嵌入](https://github.com/rusterlium/rustler)。这很有用，因为 Elixir（或者说 Elixir 运行的 Erlang 虚拟机）的一个缺点是原始的计算性能，而 Rust 擅长这方面。

Elixir 的另一个缺点是缺乏类型（虽然[正在进行改进](https://nitter.net/josevalim/status/1744395345872683471)）。这就是我被 Gleam 吸引的原因，它感觉像 Elixir，但在上面加了 Rust 类型——一个非常棒的销售点。

年轻语言的一个很大的缺点是没有太多的库，所以大部分时间你都得自己动手解决。但是，针对现有平台的美妙之处在于你也可以利用它们的库，而在 Gleam 的情况下，这意味着所有 Erlang 和 Elixir 库（或者 JavaScript，取决于你的编译目标）。这是通过 Gleam 的 FFI 实现的，相当方便。

让我们来探索一下。

如果你想跟着进行，确保你已经安装了相应的编译器。我将以一个干净的新存储库为例：

正如[《Gleam 书籍》](https://gleam.run/book/tour/external-functions.html)中所示，调用标准的 Erlang 函数非常简单。你用 `@external` 关键字声明外部函数，然后像正常一样调用它们：

```
import gleam/io

@external(erlang, "rand", "uniform")
pub fn random_float() -> Float

pub fn main() {
  io.debug(random_float())
} 
```

如果运行，这将调用 `rand` Erlang 模块中的 `uniform` 函数：

```
$ gleam run
0.43487935467166317 
```

这适用于标准库，但你可以通过在 `gleam.toml` 中将它们添加为依赖项来访问 [Hex](https://hex.pm/) 上的其他 Erlang 库：

```
[dependencies]
base32 = "~> 0.1.0" 
```

声明并像以前一样调用它：

```
@external(erlang, "base32", "encode")
pub fn encode_base32(x: String) -> String

pub fn main() {
  io.debug(encode_base32("superhidden"))
} 
```

而且 Gleam 会下载并编译 Erlang 依赖：

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

如果你想自己编写 Erlang 代码并调用它，这也非常容易。Gleam 编译器会自动编译和包含 `.erl` 文件。

比如这个文件 `src/erlib.erl`：

```
-module(erlib).
-export([ping/0]).

ping() ->
    io:fwrite("ping~n", []). 
```

声明并像以前一样调用 gleam 中的函数：

```
@external(erlang, "erlib", "ping")
pub fn ping() -> a

pub fn main() {
  ping()
} 
```

你也可以从 Erlang 调用 Gleam 函数。比如说，在 `src/mypong.gleam` 中有这样一个 pong 函数：

```
import gleam/io

pub fn pong() {
  io.println("pong")
} 
```

你可以用 `module:function()` 调用 Gleam 函数：

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

如果你想在现有的 Elixir 项目中包含 Gleam 代码，请查看[MixGleam](https://github.com/gleam-lang/mix_gleam)，了解如何教`mix`如何处理 Gleam 代码和依赖关系。但如果你想在 Gleam 项目中包含 Elixir 代码，你可以像使用 Erlang 一样轻松地做到这一点。

声明外部 Elixir 函数使用相同的`@external`关键字：

```
import gleam/io

@external(erlang, "Elixir.RandomColor", "hex")
pub fn random_color() -> String

pub fn main() {
  io.println(random_color())
} 
```

请注意，Elixir 模块会得到一个`Elixir`前缀，并且我们仍然将外部 Erlang 代码称为 Elixir，因为 Elixir 被编译为 Erlang。

依赖项是从[Hex](https://hex.pm/)添加的，与 Erlang 依赖项完全相同：

```
$ gleam add random_color
$ gleam run
#3724C9 
```

调用我们自己的 Elixir 代码和调用 Erlang 一样容易。只需将其包含在您的源目录中，像这样`src/exlib.ex`：

```
defmodule Exlib do
 def ping() do
    IO.puts("ping from Elixir")
    :mypong.pong()
  end
end 
```

并将模块称为`Elixir.Exlib`：

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

请记住：

1.  你不能直接调用 Elixir 宏（它们在编译时发生）。

1.  如果你包含了 Elixir，你也会包含 Elixir 标准库。如果你想要更轻量级的东西，Erlang 可能是一个更好的选择。

开始时我写道，你可以通过[rustler](https://github.com/rusterlium/rustler)很容易地从 Elixir 中调用 Rust 代码。你也可以通过 Erlang 或 Elixir 从 Gleam 调用 Rust。[Rustler](https://github.com/rusterlium/rustler)建议使用 Elixir 的`mix`，但我们可以通过 Erlang 来做到这一点，并继续使用 Gleam 工具链。

首先，我们需要在某处创建一个 Rust 项目。我们可以将它创建在 Gleam 仓库的顶层，或者在一个`native/`文件夹中，或者任何其他地方。对于这个示例，我将它留在顶层：

在 Rust 方面，我将使用[rustler](https://github.com/rusterlium/rustler)来创建 NIF：

```
$ cd rslib/
$ cargo add rustler 
```

而在`rslib/src/lib.rs`中，我们将要暴露的函数标记为[rustler](https://github.com/rusterlium/rustler)：

```
#[rustler::nif]
pub fn truly_random() -> i64 {
    4 }

rustler::init!("librs", [truly_random]); 
```

我们希望将其构建为一个动态库，所以我们需要更新`Cargo.toml`：

```
[lib]
crate-type = ["dylib"] 
```

然后以发布模式构建它：

```
$ cargo build --release
  ... 
```

这应该会在`target/release/librslib.so`生成库。是的，我这里的命名选择很糟糕，但我们继续吧。

为了让 Gleam 能够使用这个文件，我们需要将它复制到`priv/`（相对于 Gleam 根目录，而不是 Rust 项目）：

```
$ mkdir ../priv
$ cp target/release/librslib.so ../priv/ 
```

文件布局应该看起来像这样：

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

要包含库，我们将使用一些 Erlang。我们将使用`src/rslib.erl`，类似于之前，但使用[Erlang NIFs](https://www.erlang.org/doc/tutorial/nif.html)：

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

我们现在加载`librslib`库，将`truly_random`声明为一个 NIF，并添加一个函数占位符，在库加载时将被替换。librs

在放置了 Erlang 代码后，剩下的就是从 Gleam 调用 Erlang 函数：

```
import gleam/io

@external(erlang, "librs", "truly_random")
pub fn truly_random() -> String

pub fn main() {
  io.debug(truly_random())
} 
```

现在我们可以调用我们的[真正随机](https://xkcd.com/221/)的 Rust 函数了！

虽然 Rust 很棒，我们可以从 Gleam 调用它真是太好了，但也有一些需要注意的缺点：

1.  由于我们需要多次复制函数名称，所以粘合剂很冗长。

1.  构建库并将其移动到 `priv/` 目录不是 Gleam 工具链处理的事情。我见过一些人使用 Makefile 来构建库，然后将其复制到 `priv/` 目录。

1.  NIF 可以使整个运行时崩溃。虽然 Rust 比 C 更安全，但它仍然可能使整个运行时崩溃，而不仅仅是一个 Erlang 进程。

从 Rust 调用 Gleam 代码似乎更困难。我找到了一个[从 Rust 调用 Elixir 或 Erlang 函数的示例](https://github.com/Qqwy/elixir-rustler_elixir_fun "Calling Elixir code from Rust")，但我还没有仔细研究它。

Gleam 也可以编译为 JavaScript，而且没有一个简短的示例，这篇博文就不完整了。

要编译为 JavaScript，我们可以将 `target = javascript` 添加到我们的 `gleam.toml` 文件中，或者使用目标标志：

```
$ gleam run --target javascript 
```

就像以前调用 Erlang 和 Elixir 文件一样，如果你在 `src/` 目录下添加一个 JavaScript 源文件，它将自动包含进来。例如这个 `src/jslib.mjs`：

```
import * as gleam from "./mypong.mjs";

export function ping() {
  console.log("ping from JavaScript");
  gleam.pong();
} 
```

而且为了声明外部函数，我们现在使用相对文件而不是库名：

```
@external(javascript, "./jslib.mjs", "ping")
pub fn ping() -> a

pub fn main() {
  ping()
} 
```

运行时，你会发现我们可以调用 JavaScript 的 `ping` 函数，它会调用我们的 Gleam 的 `pong` 函数：

```
$ gleam run --target javascript
ping from JavaScript
pong 
```

你可以检查位于 `build/dev/javascript/myapp/` 中的输出 JavaScript 文件，在调试或尝试理解 Gleam 生成的代码时非常有用。

我希望这篇文章能帮助你了解如何开始使用 Gleam 的 FFI 系统。我对 Gleam 的了解不超过几天，但考虑到我可以轻松访问到其他我喜欢的语言及其生态系统，如果我要用 Gleam 创造一些真实的东西，我不必担心会错过一些关键的功能。
