<!--yml
category: 未分类
date: 2024-05-29 12:32:08
-->

# How I Improved My Rust Compile Times by 75%

> 来源：[https://benw.is/posts/how-i-improved-my-rust-compile-times-by-seventy-five-percent](https://benw.is/posts/how-i-improved-my-rust-compile-times-by-seventy-five-percent)

> There's now a Part 2, where I cover a couple more options. Check it out [here](https://benw.is/posts/how-i-improved-my-rust-compile-times-part2)

One of Rust's often mentioned pain points is slow compile times. In order to have nice things like the borrow checker, safety, and zero cost abstractions, we pay in time spent compiling. Web developers, especially those that come from one of the Javascript frameworks, really, really like fast reload times. I was fairly active in the Remix discord early on, and the number of people who asked about HMR(Hot Module Reloading) [was insane](https://github.com/remix-run/remix/discussions/2384). Hot Module Reloading is when the web framework will replaces javascript, html, and css modules in an existing page as they are changed without losing page state. The alternative, Live Page Reloading, will trigger a page reload when a change is detected. For Remix, the time difference between the two is signifigantly less than it would be in Rust, where we'd have to recompile.

You can imagine, as a Leptos team member, we hear a lot of the same discussion. Leptos, as a frontend Rust web framework, sits at the intersection of these two worlds We see it on the [Leptos Discord](https://discord.gg/x8NhWWYTV2) all the time.

> "Now, if only rebuild was faster… 🫣"

> "Anyone have some tips to improve compile times? Leptos is amazing but having to wait 5-10 seconds to render an extra div is very annoying"

So Leptos basically has two approaches to solve this, improve Rust's compile time itself, and/or come up with a way to output changes to parts of the page without needing to recompile first. In this piece, we'll explore both.

In order to test different changes, we need to establish a baseline. In default Rust, on my machine, how long will it take to compile my web app?

I have a quite beefy test machine on which I usually compile. On my machine I'll be compiling a simpler project, and my friend Alex kindly agreed to run test his startup Houski's [site](https://www.houski.ca/),

*   AMD 5950x processor,
*   72GB RAM
*   SATA SSD system drive.
*   7200RPM spinning disk storage drives
*   NVME drives
*   NixOS linux distro
*   Rust 1.75 nightly

*   AMD 7900x processor
*   128GB RAM
*   NVME drive
*   Ubuntu linux distro
*   Rust 1.75 nightly

A Leptos web app typicallys undergoes a two stage build process.

1.  Build Webassembly frontend module using cargo. Optimize and package it with warm-bindgen.
2.  Build the server binary with cargo.

I chose to benchmark the whole process for most of these demos, but since it's essentially two cargo steps, any relative changes would affect any cargo command.

> ~~Note: Discord User @PaulH(and a couple others) let me know about this [bug](https://github.com/leptos-rs/cargo-leptos/pull/203) that prevents the server build from using the dependencies built for the previous compilation of the server/webassembly frontend build due to changes in the RUSTFLAGS. You can fix this by having cargo-leptos > 0.2.1 and adding this to you Cargo.toml under your Leptos settings block.!~~ I did not test with this enabled, but this is now the default behavior of cargo-leptos. The feature is deprecated.
> 
> TOML markup
> 
> ```
> *[**package**.**metadata**.**leptos**]*
> *separate-front-target-dir* *=* *true*
> 
> ```

This advice comes from Bevy, which recommends setting the opt-level for development higher in order to possibly reduce dev compile times and improve performance. By default, the Rust compiler sets an opt-level of 0 for development builds. We're going to give it an opt-level of 1 for our code, and an opt-level of 3 for all the dependencies of our code.

This does come with the drawback of much less useful error messages if they come from our dependencies. So you might have to adjust the levels if you run into tricky bugs.

Since optimization takes additional time, I expect the clean compile times to increase, but we may see the increased optimizations make the code easier to compile in incremental builds. We also had some positive anecdotes on our Discord.

> "fwiw @gbj @Alex Wilkinson i dropped my compile time from between 5-30 minutes to 6 seconds by putting this in my workspace Cargo.toml

We can enable adding these blocks to our `Cargo.toml` file. Nice and easy.

TOML markup

```
*[**profile**.**dev**]*
*opt-level* *=* 1
*[**profile**.**dev**.**package**.**"*"**]*
*opt-level* *=* 3

```

For those unfamiliar with how what goes on inside the Rust compiler, it typically follows a few basic steps. It reads in source code, which is converted to a variety of types of IR(Intermediate Representation), and optimizations are performed during that conversion. Then that IR is fed to a code generator, provided by LLVM, which converts the IR into object files, and then the linker links together those object files and other system libs into one executable binary. Way more details about how that works can be found [here](https://rustc-dev-guide.rust-lang.org/overview.html). Fascinating reading, but a bit too deep for our discussion here.

[Mold](https://github.com/rui314/mold) is a new linker developed by Rui Ueyama, with the goal of increasing linker performance by parallelizing the load as much as possible, and benchmarks have shown it to be signifigantly faster than Rust's default linker.

For Linux and Mac, the default linker is ld, run by cc. Windows is a different story, using Microsoft's MVC link.exe. If you're running Linux, you can use mold directly. If you're on Mac, a paid version of mold called Sold is available for Mac. If Mold generates benefits for you, I encourage you to buy Sold(very affordable) or sponsor Rui on his Github Sponsors [page](https://github.com/sponsors/rui314). Windows users, unforunately, are not supported at this time. Support for Windows in Sold is in development.

A fairly signifigant amount of time is spent linking your Rust binary, especially during incremental builds, so this could provide some real benefits.

On Linux, it's actually really easy to use, just [install Mold](https://github.com/rui314/mold) and then prepend you cargo commands with `mold -run`. For example, `mold -run cargo build`. It can also be enabled in `.cargo/config.toml`, like this:

TOML markup

```
*[**target**.**x86_64-unknown-linux-gnu**]*
*linker* *=* *"clang"*
*rustflags* *=* *[**"-C"**,* *"link-arg=-fuse-ld=/path/to/mold"**]*

```

where `/path/to/mold` is an absolute path to the mold executable. This is also how you enable Sold, just replace the mold path with the sold path and the target to the one for your Mac.

In the previous optimization, we replaced the linker the Rust compiler uses. Let's try replacing the code generator, Cranelift is an alternative code generator, used instead of LLVM in the build step. While it's not good at doing as many optimizations as LLVM, it is good at spitting out code fast. It was recently integrated as an option for code generation in Rust 1.73 nightly for x86_64 linux targets. Other platforms will need to setup cranelift seperately, see their [README](https://github.com/rust-lang/rustc_codegen_cranelift).

bash

```
rustup update nightly #install nightly if you haven't already
rustup component add rustc-codegen-cranelift-preview --toolchain nightly

```

To use it with Cargo, it can be enabled by enabling the unstable `codegen-backend` feature, and then setting the `codegen-backend= "cranelift"` value for a profile. That can be done in `.cargo/config.toml` like so:

TOML markup

```
*[**unstable**]*
*codegen-backend* *=* *true*
*[**profile**.**server-dev**]*
*codegen-backend* *=* *"cranelift"*

```

If you don't need to worry about multi-target builds, you can also enable it below with a target. This does conflict with the desire to not use cranelift during production builds, so I wouldn't recommend it. Rust doesn't support per target profile builds, so creating your own profile is a lot more flexible.

TOML markup

```
*[**target**.**x86_64-unknown-linux-gnu**]*
*rustflags* *=* *[**"-Zcodegen-backend=cranelift"**]*

```

Keep in mind that Cranelift is still in development, and may have some issues with missing intrinsics. So your crate may or may not work in it natively. If you do find missing intrinsics, I encourage you to create an issue in their repo [here](https://github.com/rust-lang/rustc_codegen_cranelift), there may be a workaround available.

I'm primarily interested in optimizing build times for Rust projects in development. This means I'm not concerned about filesize, optimizations, or run speed, as long as they are not signifigantly affected. We're interested in the time it takes to build our Rust Leptos apps, both from a clean state, and with incremental compilation.

To do that we're going to use [hyperfine](https://github.com/sharkdp/hyperfine) a command line benchmarking tool, to run 'cargo leptos build', which as described earlier does Leptos's two step compilation. For clean builds, we will run `cargo clean` before each build, in order to delete any files cached by Rust. Incremental builds are a bit harder.

For incremental builds, we'll simulate a developer modifying a single HTML tag in a Leptos component. To do that, we'll run our handy unix friend `sed` to insert the current date and time into an HTML tag. For this purpose, I've chosen the `<dfn>` tag, because I've never seen it used, and thus I probably don't have to worry about multiple replacements tags. After some finagling, I ended up with this sed command:

bash

```
sed -i -e "s|<dfn>[^<]*</dfn>|<dfn>$(date +%m%s)</dfn>|g" src/routes/index.rs

```

Each configuration is tested six times, with two warmup runs to fill any caches and reduce variability in the output. Without the warmup runs variance was a bit wider, but the results were still consistent. Six runs is probably overkill for this test, but what can I say, I got CPU time for days, or more accurately nights.

We'll measure the time it takes to complete six runs, with two warmup runs to setup filesystem caches, and hopefully provide consistent results.

I will be compiling my [site](https://benwis), and Alex will be compiling [Houski's site](https://houski.ca). Houski's is a lot more complicated than this blog, with heavy usage of Polars, Serialization/Deserialization with Serde, and plenty more routes. Neither of us has tried to reduce these times much or done any special kinds of optimization.

For both of us, these will be the times to beat.

| Clean Compilation |  |  |
| --- | --- | --- |
| Site | Mean(s) | Standard Deviation(s) |
| benw.is 7200RPM | 88.387 | 1.817 |
| benw.is NVME | 80.057 | 0.268 |
| houski.ca | 124.993 | 0.483 |

| Incremental Compilation |  |  |
| --- | --- | --- |
| Site | Mean(s) | Standard Deviation(s) |
| benw.is 7200RPM | 20.461 | 0.801 |
| benw.is NVME | 19.097 | 0.078 |
| houski.ca | 40.818 | 0.252 |

Not a bad baseline. Interesting to me was the variation in the 7200RPM build time for both clean and incremental builds, which might suggest that the drive was struggling to provide the data at the same rate consistently, or perhaps something else was contending with the benchmark for IO access.

Twenty or forty seconds is a pretty substantial time to rebuild the site in the dev profile, hopefully we can improve that.

Let's enable the Mold linker as described earlier, and see how that changes things!

| Clean Compilation |  |  |  |  |
| --- | --- | --- | --- | --- |
| Site | Mean(s) | Standard Deviation(s) | Delta from Baseline(s) | % Delta from Baseline |
| benw.is 7200RPM | 74.35 | 0.96 | 14.037 | 15.88129476 |
| benw.is NVME | 67.206 | 0.386 | 12.851 | 16.05231273 |
| houski.ca | 102.892 | 0.416 | 22.101 | 17.68179018 |

| Incremental Compilation |  |  |  |  |
| --- | --- | --- | --- | --- |
| Site | Mean(s) | Standard Deviation(s) | Delta from Baseline(s) | % Delta from Baseline |
| benw.is 7200RPM | 5.842 | 0.268 | 14.619 | 71.44812082 |
| benw.is NVME | 5.615 | 0.051 | 13.482 | 70.59747604 |
| houski.ca | 19.6 | 0.067 | 21.218 | 51.98196874 |

Woah! That's quite a bit faster. For my site, clean compilation time decreased by 12.04s(13.6%) on spinning disk and 12.80s(16.0%) for NVME. Houski dropped 22.10s(17.7s). Nice boost there.

Incremental compilation times changed signifigantly as well. My site on spinning disk dropped 14.62s(71.4%) and 13.5s(70.6%). That's a radical reduction, which makes some sense. Incremental compilation spends a majority of it's time in the linking step, so any upgrades there will disproportionally affect it.

As far as Leptos users go, mold only supports compiling for x84_64_linux, so the webassembly compile time remained unchanged. Most of these optimizations only affect the server build, but I will be careful to note which ones don't.

With few downsides, and such a huge upside, Mold/Sold seems like a no-brainer, especially if you are doing a lot of incremental builds!

What will adding more optimization do to compile times?

| Clean Compilation |  |  |  |  |
| --- | --- | --- | --- | --- |
| Site | Mean(s) | Standard Deviation(s) | Delta from Baseline(s) | % Delta from Baseline |
| benw.is 7200RPM | 174.399 | 0.488 | -86.012 | -97.31295326 |
| benw.is NVME | 166.847 | 0.41 | -86.79 | -108.4102577 |
| houski.ca | 288.562 | 0.674 | -163.569 | -130.8625283 |

| Incremental Compilation |  |  |  |  |
| --- | --- | --- | --- | --- |
| Site | Mean(s) | Standard Deviation(s) | Delta from Baseline(s) | % Delta from Baseline |
| benw.is 7200RPM | 16.201 | 1.303 | 4.26 | 20.82009677 |
| benw.is NVME | 14.334 | 0.147 | 4.763 | 24.94109022 |
| houski.ca | 32.489 | 0.348 | 8.329 | 20.40521339 |

I did expect a clean compile to be slower, but this is quite a bit slower. Compared to baseline, it takes twice as long for clean builds. Incremental builds do net a twenty percent improvement, which is nice to see. Contained within the listed build time is a doubling of both the webassembly and server builds, which makes sense but should be noted. For me, I don't think the benefits of setting this are worth the delays in clean compile times.

| Clean Compilation |  |  |  |  |
| --- | --- | --- | --- | --- |
| Site | Mean(s) | Standard Deviation(s) | Delta from Baseline(s) | % Delta from Baseline |
| benw.is 7200RPM | 67.989 | 0.763 | 20.398 | 23.07805447 |
| benw.is NVME | 63.645 | 0.247 | 16.412 | 20.50039347 |
| houski.ca |  |  |  |  |

| Incremental Compilation |  |  |  |  |
| --- | --- | --- | --- | --- |
| Site | Mean(s) | Standard Deviation(s) | Delta from Baseline(s) | % Delta from Baseline |
| benw.is 7200RPM | 9.201 | 0.092 | 11.26 | 55.03152339 |
| benw.is NVME | 9.345 | 0.212 | 9.752 | 51.0656124 |
| houski.ca |  |  |  |  |

Cranelift, I had high hopes for you, and you delivered! It improved both clean and incremental compilations compared to the baseline. You may notice that the houski app lines here are blank, which are due to a failure to compile due to missing llvm intrinsics.

After my testing completed, they merged this [PR](%5B#1417%5D(https://github.com/rust-lang/rustc_codegen_cranelift/pull/1417)) which added the missing `llvm.x86.avx.ldu.dq.256` instrinsic, and @bjorn3 mentioned that I can avoid the missing aes intrinsics with a rust flag `RUSTFLAGS="--cfg aes_force_soft"`. Either that, or this [PR](https://github.com/rust-lang/rustc_codegen_cranelift/pull/1425) gets merged into rustup nightly. I hope to update this section soon.

If you are missing intrinsics, you can hook up a debugger to see what is causing the trap, or you can use `cargo vendor` and do a search for the the intrinsic name or part of the intrinsic name. I'd be sure to mention anything you're missing in the [rustc cranelift repo](https://github.com/rust-lang/rustc_codegen_cranelift/issues/171#issuecomment-1803136179) and occasionally check in to see if there's a fix coming. @bjorn3 is on a roll there.

| Clean Compilation |  |  |  |  |
| --- | --- | --- | --- | --- |
| Site | Mean(s) | Standard Deviation(s) | Delta from Baseline(s) | % Delta from Baseline |
| benw.is 7200RPM | 170.241 | 1.016 | -81.854 | -92.60864154 |
| benw.is NVME | 168.047 | 1.018 | -87.99 | -109.9091897 |
| houski.ca | 271.949 | 0.735 | -146.956 | -117.571384 |

| Incremental Compilation |  |  |  |  |
| --- | --- | --- | --- | --- |
| Site | Mean(s) | Standard Deviation(s) | Delta from Baseline(s) | % Delta from Baseline |
| benw.is 7200RPM | 5.789 | 0.051 | 14.672 | 71.70715019 |
| benw.is NVME | 5.727 | 0.041 | 13.37 | 70.01099649 |
| houski.ca | 18.15 | 0.202 | 22.668 | 55.53432309 |

Our first combination of features. Unforunately, this seems to have the slow clean compile times of O3, without decreasing compile times more than mold alone.

| Clean Compilation |  |  |  |  |
| --- | --- | --- | --- | --- |
| Site | Mean(s) | Standard Deviation(s) | Delta from Baseline(s) | % Delta from Baseline |
| benw.is 7200RPM | 107.401 | 1.05 | -19.014 | -21.51221333 |
| benw.is NVME | 103.544 | 0.88 | -23.487 | -29.33784678 |
| houski.ca |  |  |  |  |

| Incremental Compilation |  |  |  |  |
| --- | --- | --- | --- | --- |
| Site | Mean(s) | Standard Deviation(s) | Delta from Baseline(s) | % Delta from Baseline |
| benw.is 7200RPM | 4.992 | 0.322 | 15.469 | 75.60236548 |
| benw.is NVME | 4.767 | 0.073 | 14.33 | 75.03796408 |
| houski.ca |  |  |  |  |

Let's enable all the things! The incremental compiles are faster than any of the previously tested options, but the clean compiles are still suffering a penalty from O3\. Cranelift seems to reduce the penalty from O3, but time will tell whether it is better than Mold + Cranelift alone. I suspect this is because Cranelift optimizes less than llvm, which is in line with their documentation.

| Clean Compilation |  |  |  |  |
| --- | --- | --- | --- | --- |
| Site | Mean(s) | Standard Deviation(s) | Delta from Baseline(s) | % Delta from Baseline |
| benw.is 7200RPM | 110.251 | 0.687 | -21.864 | -24.737 |
| benw.is NVME | 106.732 | 0.526 | -26.675 | -33.323 |
| houski.ca |  |  |  |  |

| Incremental Compilation |  |  |  |  |
| --- | --- | --- | --- | --- |
| Site | Mean(s) | Standard Deviation(s) | Delta from Baseline(s) | % Delta from Baseline |
| benw.is 7200RPM | 9.096 | 0.351 | 11.365 | 55.54469479 |
| benw.is NVME | 8.73 | 0.076 | 10.367 | 54.28601351 |
| houski.ca |  |  |  |  |

Adding further support to the Cranelift optimization hypothesis, we see similiar performance in the clean compilation, but slower performance in incremental without Mold enabled. It is interesting that this ran faster than the previous test with Mold enabled, although I am not sure the difference is signifigant. Perhaps Mold and Cranelift or Mold and O3 have some negative interaction I don't recognize.

| Clean Compilation |  |  |  |  |
| --- | --- | --- | --- | --- |
| Site | Mean(s) | Standard Deviation(s) | Delta from Baseline(s) | % Delta from Baseline |
| benw.is 7200RPM | 65.343 | 1.243 | 23.044 | 26.07170738 |
| benw.is NVME | 59.777 | 0.191 | 20.28 | 25.33195098 |
| houski.ca |  |  |  |  |

| Incremental Compilation |  |  |  |  |
| --- | --- | --- | --- | --- |
| Site | Mean(s) | Standard Deviation(s) | Delta from Baseline(s) | % Delta from Baseline |
| benw.is 7200RPM | 4.831 | 0.251 | 15.63 | 76.38922829 |
| benw.is NVME | 4.654 | 0.067 | 14.443 | 75.62968005 |
| houski.ca |  |  |  |  |

Here's the big money, Cranelift and Mold together. This offered the fastest clean compilation times, with a roughly 25% reduction, and incremental, sitting pretty at around 75% reduction. Interestingly enough, there seemed to be little difference between the NVME and the 7200RPM drives here, suggesting that these two might be better at using disk I/O than some of the other configurations.

For Rust web frameworks, we can try to simulate the performance of HMR, but we won't be able to substitute JS modules. Our attempt to do this is in cargo-leptos, and can be used with the `--hot-reload` flag. It will attempt to send html and css patches to the browser over a web socket connection when a change inside a `view` macro is detected that does not involve a change to a rust code block, and compile in the background. For me it works fairly well.

I didn't try to instrument and test it, as hot reload times are nearly instant. Any changes to the rust code will require a incremental recompilation though, which is why it's so useful to improve that time. JS frameworks will probably have an edge there for the foreseeable future.

Enabling Mold and Cranelift netted me a 75% incremental and 25% clean compilation time reduction, which is quite signifigant. Using Cranelift requires nightly, or setting it up outside rustup, which some projects might find undesirable. Since Mold requires Linux, and Sold requires Mac(and paying), for me this further increases the argument that Rust web development should be done on some for of Linux or Mac, and not on Windows. WSL2 may help improve things, but may [be slower](https://medium.com/for-linux-users/wsl-2-why-you-should-use-real-linux-instead-4ee14364c18). Since I have a Mac laptop and a Linux workstation, I will be buying Sold for my Mac and using Cranelift for my projects that can.

If you feel like running some of these tests on your hardware or with your project, I have automated the process somewhat and put that repo on Github [here](https://github.com/benwis/customs).

Let me know what your experience was, and whether this helped you with your project by reaching out on mastodon at `@benwis@hachyderm.io`.