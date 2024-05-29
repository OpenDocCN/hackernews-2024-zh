<!--yml
category: 未分类
date: 2024-05-27 14:55:02
-->

# The bane of my existence: Supporting both async and sync code in Rust | NullDeref

> 来源：[https://nullderef.com/blog/rust-async-sync/](https://nullderef.com/blog/rust-async-sync/)

[The third attempt](https://github.com/ramsayleung/rspotify/pull/129) is based on a crate called [`maybe_async`](https://crates.io/crates/maybe_async) . I remember foolishly thinking it was the perfect solution back when I discovered it.

Anyway, the idea is that with this crate you can automatically remove the `async` and `.await` occurrences in your code with a procedural macro, essentially automating the copy-pasting approach. For example:

```
#[maybe_async::maybe_async] async  fn endpoint()  {  /* stuff */  }
```

Generates the following code:

```
#[cfg(not(feature = "is_sync"))] async  fn endpoint()  {  /* stuff */  }   #[cfg(feature = "is_sync")] fn endpoint()  {  /* stuff with `.await` removed */  }
```

You can configure whether you want asynchronous or blocking code by toggling the `maybe_async/is_sync` feature when compiling the crate. The macro works for functions, traits and `impl` blocks. If one conversion isn’t as easy as removing `async` and `.await`, you can specify custom implementations with the `async_impl` and `sync_impl` procedural macros. It does this wonderfully, and we’ve already been using it for Rspotify for a while now.

In fact, it worked so well that I made Rspotify *http-client agnostic*, which is even more flexible than being *async/sync agnostic*. This allows us to support multiple HTTP clients like [`reqwest`](https://crates.io/crates/reqwest) and [`ureq`](https://crates.io/crates/ureq) , independently of whether the client is asynchronous or synchronous.

Being *http-client agnostic* is not that hard to implement if you have `maybe_async` around. You just need to define a trait for the [HTTP client](https://github.com/ramsayleung/rspotify/blob/89b37219a2230cdcf08c4cfd2ebe46d64902f03d/rspotify-http/src/common.rs#L46), and then implement it for each of the clients you want to support:

```
#[maybe_async] trait  HttpClient  {  async  fn get(&self)  -> String; }   #[sync_impl] impl  HttpClient  for  UreqClient  {  fn get(&self)  -> String {  ureq::get(/* ... */)  } }   #[async_impl] impl  HttpClient  for  ReqwestClient  {  async  fn get(&self)  -> String {  reqwest::get(/* ... */).await  } }   struct SpotifyClient<Http: HttpClient>  {  http: Http }   #[maybe_async] impl<Http: HttpClient>  SpotifyClient<Http>  {  async  fn endpoint(&self)  {  self.http.get(/* ... */)  } }
```

Then, we could extend it so that whichever client they want to use can be enabled with feature flags in their `Cargo.toml`. For example, if `client-ureq` is enabled, since `ureq` is synchronous, it would enable `maybe_async/is_sync`. In turn, this would remove the `async`/`.await` and the `#[async_impl]` blocks, and the Rspotify client would use `ureq`'s implementation internally.

This solution has none of the downsides I listed in previous attempts:

*   No code duplication at all

*   No overhead neither at runtime nor at compile time. If the user wants a blocking client, they can use `ureq`, which doesn’t pull `tokio` and friends

*   Quite easy to understand for the user; just configure a flag in you `Cargo.toml`

However, stop reading for a couple of minutes and try to figure out why you shouldn’t do this. In fact, I’ll give you 9 months, which is how long it took me to do so…​

### The problem[#](#_the_problem)

Well, the thing is that features in Rust must be **additive**: “enabling a feature should not disable functionality, and it should usually be safe to enable any combination of features”. Cargo may merge features of a crate when it’s duplicated in the dependency tree in order to avoid compiling the same crate multiple times. [The reference explains this quite well, if you want more details](https://doc.rust-lang.org/cargo/reference/features.html#feature-unification).

This optimization means that mutually exclusive features may break a dependency tree. In our case, `maybe_async/is_sync` is a *toggle* feature enabled by `client-ureq`. So if you try to compile it with `client-reqwest` also enabled, it will fail because `maybe_async` will be configured to generate synchronous function signatures instead. It’s impossible to have a crate that depends on both sync and async Rspotify either directly or indirectly, and the whole concept of `maybe_async` is currently wrong according to the Cargo reference.

### The feature resolver v2[#](#_the_feature_resolver_v2)

A common misconception is that this is fixed by the “feature resolver v2”, which [the reference also explains quite well](https://doc.rust-lang.org/cargo/reference/features.html#feature-resolver-version-2). It has been enabled by default since the 2021 edition, but you can specify it inside your `Cargo.toml` in previous ones. This new version, among other things, avoids unifying features in some special cases, but not in ours:

> *   Features enabled on platform-specific dependencies for targets not currently being built are ignored.
>     
>     
> *   Build-dependencies and proc-macros do not share features with normal dependencies.
>     
>     
> *   Dev-dependencies do not activate features unless building a target that needs them (like tests or examples).

Just in case, I tried to reproduce this myself, and it did work as I expected. [This repository](https://github.com/marioortizmanero/resolver-v2-conflict) is an example of conflicting features, which breaks with any feature resolver.

### Other fails[#](#_other_fails)

There were a few crates that also had this problem:

*   [`arangors`](https://crates.io/crates/arangors) and [`aragog`](https://crates.io/crates/aragog) : wrappers for ArangoDB. Both use `maybe_async` to switch between async and sync (`arangors`'s author is the same person, in fact) [[5]](#arangors-error) [[6]](#aragog-error).

*   [`inkwell`](https://crates.io/crates/inkwell) : a wrapper for LLVM. It supports multiple versions of LLVM, which are not compatible with eachother [[7]](#inkwell-error).

*   [`k8s-openapi`](https://crates.io/crates/k8s-openapi) : a wrapper for Kubernetes, with the same issue as `inkwell` [[8]](#k8s-error).

### Fixing `maybe_async`[#](#_fixing_maybe_async)

Once the crate started to gain popularity, this issue was opened in `maybe_async`, which explains the situation and showcases a fix:

`maybe_async` would now have two feature flags: `is_sync` and `is_async`. The crate would generate the functions in the same way, but with a `_sync` or `_async` suffix appended to the identifier so that they wouldn’t be conflicting. For example:

```
#[maybe_async::maybe_async] async  fn endpoint()  {  /* stuff */  }
```

Would now generate the following code:

```
#[cfg(feature = "is_async")] async  fn endpoint_async()  {  /* stuff */  }   #[cfg(feature = "is_sync")] fn endpoint_sync()  {  /* stuff with `.await` removed */  }
```

However, these suffixes introduce noise, so I wondered if it would be possible to do it in a more ergonomic way. I forked `maybe_async` and gave it a try, about which you can read more [in this series of comments](https://github.com/fMeow/maybe-async-rs/issues/6#issuecomment-880581551). In summary, it was too complicated, and I ultimately gave up.

The only way to fix this edge case would be to worsen the usability of Rspotify for everyone. But I’d argue that someone who depends on both async and sync is unlikely; we haven’t actually had anyone complaining yet. Unlike `reqwest`, `rspotify` is a “high level” library, so it’s hard to imagine a scenario where it appears more than once in a dependency tree in the first place.

Perhaps we could ask the Cargo devs for help?

### Official Support[#](#_official_support)

Rspotify is far from being the first who has been through this problem, so it might be interesting to read previous discussions about it:

*   [This now-closed RFC for the Rust compiler](https://github.com/rust-lang/rfcs/pull/2962) suggested adding the `oneof` configuration predicate (think `#[cfg(any(…​))]` and similars) to support exclusive features. This only makes it easier to have conflicting features for cases where there’s *no choice*, but features should still be strictly additive.

*   The previous RFC started [some discussion](https://internals.rust-lang.org/t/pre-rfc-cargo-mutually-exclusive-features/13182/27) in the context of allowing exclusive features in Cargo itself, and although it has some interesting info, it didn’t go too far.

*   [This issue in Cargo](https://github.com/rust-lang/cargo/issues/2980) explains a similar case with the Windows API. The discussion includes more examples and solution ideas, but none have made it to Cargo yet.

*   [Another issue in Cargo](https://github.com/rust-lang/cargo/issues/4803) asks for a way to test and build with combinations of flags easily. If features are strictly additive, then `cargo test --all-features` will cover everything. But in case it doesn’t, the user has to run the command with multiple combinations of feature flags, which is quite cumbersome. This is already possible unofficially thanks to [`cargo-hack`](https://github.com/taiki-e/cargo-hack).

*   A completely different approach [based on the Keyword Generics Initiative](https://blog.rust-lang.org/inside-rust/2023/02/23/keyword-generics-progress-report-feb-2023.html). It seems to be the most recent take on solving this, but it’s in an “exploration” phase, and [no RFCs are available as of this writing](https://blog.rust-lang.org/inside-rust/2022/07/27/keyword-generics.html#q-is-there-an-rfc-available-to-read).

According to [this old comment](https://github.com/rust-lang/rfcs/pull/2962#issuecomment-664656377), it’s not something the Rust team has already discarded; it’s still being discussed.

Although unofficial, another interesting approach that could be explored further in Rust is [“Sans I/O”](https://sans-io.readthedocs.io/). This is a Python protocol that abstracts away the use of network protocols like HTTP in our case, thus maximizing reusability. An existing example in Rust would be [`tame-oidc`](https://github.com/EmbarkStudios/tame-oidc).