<!--yml

category: 未分类

日期：2024-05-27 14:55:02

-->

# 我的存在的噩梦：在Rust中支持异步和同步代码 | NullDeref

> 来源：[https://nullderef.com/blog/rust-async-sync/](https://nullderef.com/blog/rust-async-sync/)

[第三次尝试](https://github.com/ramsayleung/rspotify/pull/129)基于一个叫做[`maybe_async`](https://crates.io/crates/maybe_async)的crate。我记得当我发现它时，愚蠢地认为它是完美的解决方案。

无论如何，这个crate的想法是，在你的代码中自动删除`async`和`.await`的出现，使用过程宏实现，基本上是自动化的复制-粘贴方法。例如：

```
#[maybe_async::maybe_async] async  fn endpoint()  {  /* stuff */  }
```

生成以下代码：

```
#[cfg(not(feature = "is_sync"))] async  fn endpoint()  {  /* stuff */  }   #[cfg(feature = "is_sync")] fn endpoint()  {  /* stuff with `.await` removed */  }
```

当编译创建时，你可以通过切换`maybe_async/is_sync`功能来配置你想要异步还是阻塞的代码。这个宏适用于函数、特性和`impl`块。如果一个转换不像删除`async`和`.await`那么简单，你可以使用`async_impl`和`sync_impl`过程宏来指定自定义实现。它做得非常好，我们已经使用它一段时间了。

实际上，它的效果很好，我使Rspotify成为了*http客户端不可知*，这比*异步/同步不可知*更灵活。这使我们能够独立地支持多个HTTP客户端，像[`reqwest`](https://crates.io/crates/reqwest)和[`ureq`](https://crates.io/crates/ureq)，无论客户端是异步还是同步。

保持*http客户端的不可知性*并不难以实现，如果你有`maybe_async`的话。你只需为[HTTP客户端](https://github.com/ramsayleung/rspotify/blob/89b37219a2230cdcf08c4cfd2ebe46d64902f03d/rspotify-http/src/common.rs#L46)定义一个特性，然后为你想要支持的每个客户端实现它：

```
#[maybe_async] trait  HttpClient  {  async  fn get(&self)  -> String; }   #[sync_impl] impl  HttpClient  for  UreqClient  {  fn get(&self)  -> String {  ureq::get(/* ... */)  } }   #[async_impl] impl  HttpClient  for  ReqwestClient  {  async  fn get(&self)  -> String {  reqwest::get(/* ... */).await  } }   struct SpotifyClient<Http: HttpClient>  {  http: Http }   #[maybe_async] impl<Http: HttpClient>  SpotifyClient<Http>  {  async  fn endpoint(&self)  {  self.http.get(/* ... */)  } }
```

然后，我们可以扩展它，使用户可以通过在他们的`Cargo.toml`中启用特性标志来使用任何他们想要的客户端。例如，如果启用了`client-ureq`，因为`ureq`是同步的，它会启用`maybe_async/is_sync`。这会移除`async`/`.await`和`#[async_impl]`块，并且Rspotify客户端会在内部使用`ureq`的实现。

这个解决方案没有我在之前尝试中列出的任何缺点:

+   没有任何代码重复

+   既没有运行时开销，也没有编译时开销。如果用户想要一个阻塞客户端，他们可以使用`ureq`，它不会拉取`tokio`和其它依赖

+   对用户来说很容易理解；只需在你的`Cargo.toml`中配置一个标志

无论如何，停下来读一会儿，试着想想为什么你不应该这样做。事实上，我给你9个月的时间，就是我想明白这点的时间……

### 问题[#](#_the_problem)

嗯，问题是Rust中的特性必须是**可加的**：“启用一个特性不应该禁用功能，并且通常可以安全地启用任意组合的特性”。Cargo在重复出现在依赖树中的crate时可能会合并crate的特性，以避免多次编译同一个crate。[如果你想了解更多细节，可以参考](https://doc.rust-lang.org/cargo/reference/features.html#feature-unification)。

这个优化意味着互斥的特性可能会破坏依赖树。在我们的情况下，`maybe_async/is_sync`是由`client-ureq`启用的*切换*特性。所以如果你尝试编译时同时启用了`client-reqwest`，它将会失败，因为`maybe_async`将被配置为生成同步函数签名。根据Cargo参考，直接或间接地依赖于同时依赖于sync和async Rspotify的crate是不可能的，而`maybe_async`的整个概念目前都是错误的。

### 特性解析器v2[#](#_the_feature_resolver_v2)

一个常见的误解是这被“特性解析器v2”修复了，[参考也很好地解释了](https://doc.rust-lang.org/cargo/reference/features.html#feature-resolver-version-2)。它自2021版起默认启用，但你可以在之前的版本中在`Cargo.toml`中指定它。这个新版本，除其他外，在某些特殊情况下避免了合并特性，但在我们的情况下没有：

> +   为当前未构建的目标启用的平台特定依赖项上的特性将被忽略。
> +   
> +   构建依赖和过程宏与普通依赖不共享功能。
> +   
> +   开发依赖不会激活特性，除非构建需要它们的目标（例如测试或示例）。

以防万一，我尝试自己重现了这个问题，结果和我预期的一样。[这个存储库](https://github.com/marioortizmanero/resolver-v2-conflict)是冲突特性的一个例子，它会使任何特性解析器失败。

### 其他失败[#](#_other_fails)

还有一些crate也有这个问题：

+   [`arangors`](https://crates.io/crates/arangors)和[`aragog`](https://crates.io/crates/aragog)：ArangoDB的包装器。两者都使用`maybe_async`在异步和同步之间切换（实际上，`arangors`的作者就是同一个人）[[5]](#arangors-error) [[6]](#aragog-error)。

+   [`inkwell`](https://crates.io/crates/inkwell)：LLVM的包装器。它支持多个版本的LLVM，这些版本彼此不兼容[[7]](#inkwell-error)。

+   [`k8s-openapi`](https://crates.io/crates/k8s-openapi)：Kubernetes的包装器，与`inkwell`一样存在相同的问题[[8]](#k8s-error)。

### 修复`maybe_async`[#](#_fixing_maybe_async)

一旦创建开始流行起来，`maybe_async`中就会出现这个问题，它解释了情况并展示了一个解决方案：

`maybe_async` 现在将有两个功能标志：`is_sync` 和 `is_async`。crate 会以相同的方式生成函数，但是会在标识符后加上 `_sync` 或 `_async` 后缀，以避免冲突。例如：

```
#[maybe_async::maybe_async] async  fn endpoint()  {  /* stuff */  }
```

现在将生成以下代码：

```
#[cfg(feature = "is_async")] async  fn endpoint_async()  {  /* stuff */  }   #[cfg(feature = "is_sync")] fn endpoint_sync()  {  /* stuff with `.await` removed */  }
```

然而，这些后缀引入了噪音，所以我想知道是否可能以更符合人体工程学的方式来做。我分叉了 `maybe_async` 并尝试了一下，关于这一点你可以阅读更多[在这系列评论中](https://github.com/fMeow/maybe-async-rs/issues/6#issuecomment-880581551)。总的来说，这太复杂了，我最终放弃了。

修复这种边缘情况的唯一方法是恶化 Rspotify 的可用性，但我认为同时依赖异步和同步的人不太可能；实际上我们还没有人抱怨过。与 `reqwest` 不同，`rspotify` 是一个“高级”库，因此很难想象它在依赖树中出现超过一次的情况。

或许我们可以向 Cargo 的开发者寻求帮助？

### 官方支持[#](#_official_support)

Rspotify 远非第一个遇到此问题的人，因此阅读以前关于此问题的讨论可能会很有趣：

+   [这个现已关闭的 Rust 编译器的 RFC](https://github.com/rust-lang/rfcs/pull/2962) 建议添加 `oneof` 配置谓词（类似 `#[cfg(any(…​))]` 等）以支持排他性功能。这仅仅是为了在存在*没有选择*的情况下更容易地出现冲突功能，但功能仍应严格是增加的。

+   先前的 RFC 在允许 Cargo 本身具有排他功能的背景下 [引发了一些讨论](https://internals.rust-lang.org/t/pre-rfc-cargo-mutually-exclusive-features/13182/27)，尽管它提供了一些有趣的信息，但并没有取得太大进展。

+   [Cargo 中的这个问题](https://github.com/rust-lang/cargo/issues/2980) 解释了与 Windows API 类似的情况。讨论包括更多的示例和解决方案想法，但尚未在 Cargo 中实现。

+   [Cargo 中的另一个问题](https://github.com/rust-lang/cargo/issues/4803) 要求以简单的方式测试和构建组合标志。如果功能严格是增加的，那么 `cargo test --all-features` 将涵盖所有内容。但如果不行，用户必须以多种功能标志组合运行命令，这相当繁琐。这已经可以在非官方情况下通过 [`cargo-hack`](https://github.com/taiki-e/cargo-hack) 实现。

+   完全不同的方法 [基于关键字泛型倡议](https://blog.rust-lang.org/inside-rust/2023/02/23/keyword-generics-progress-report-feb-2023.html)。这似乎是最新解决这个问题的方法，但处于“探索”阶段，并且[截至本文撰写时还没有可用的 RFC](https://blog.rust-lang.org/inside-rust/2022/07/27/keyword-generics.html#q-is-there-an-rfc-available-to-read)。

根据 [这条旧评论](https://github.com/rust-lang/rfcs/pull/2962#issuecomment-664656377)，这不是 Rust 团队已经丢弃的东西； 它仍在讨论中。

尽管不是官方的，但在 Rust 中可以进一步探索的另一种有趣方法是 [“Sans I/O”](https://sans-io.readthedocs.io/)。 这是一个Python协议，可以在我们的情况下抽象出像HTTP这样的网络协议，从而最大限度地提高了可重用性。 Rust 中的一个现有示例是 [`tame-oidc`](https://github.com/EmbarkStudios/tame-oidc)。
