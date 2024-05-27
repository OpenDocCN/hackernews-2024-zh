<!--yml
category: 未分类
date: 2024-05-27 13:33:13
-->

# Giving Rust a chance for in-kernel codecs [LWN.net]

> 来源：[https://lwn.net/Articles/970565/](https://lwn.net/Articles/970565/)

| **Benefits for LWN subscribers**The primary benefit from [subscribing to LWN](/subscribe/) is helping to keep us publishing, but, beyond that, subscribers get immediate access to all site content and access to a number of extra site features. Please sign up today! |

April 26, 2024

This article was contributed by Daniel Almeida

Video playback is undeniably one of the most important features in modern consumer devices. Yet, surprisingly, users are by and large unaware of the intricate engineering involved in the compression and decompression of video data, with codecs being left to find a delicate balance between image quality, bandwidth, and power consumption. In response to constant performance pressure, video codecs have become complex and hardware implementations are now common, but programming these devices is becoming increasingly difficult and fraught with opportunities for exploitation. I hope to convey how Rust can help fix this problem.

Some time ago, I [proposed](/ml/linux-kernel/20230406215615.122099-1-daniel.almeida@collabora.com/) to the Linux media community that, since codec data is particularly sensitive, complex, and hard to parse, we could write some of the codec drivers in Rust to benefit from its safety guarantees. Some important concerns were raised back then, in particular that having to maintain a Rust abstraction layer would impose a high cost on the already overstretched maintainers. So I went back to the drawing board and came up with a new, simpler proposal; it differs a bit from the general flow of the [Rust-for-Linux](https://rust-for-linux.com/) community so far by realizing that we can convert error-prone driver sections without writing a whole layer of Rust bindings.

#### The dangers of stateless decoders

Most of my blog posts at Collabora have focused on the difference between stateful and stateless codec APIs. I recommend a quick reading of [this one](https://www.collabora.com/news-and-blog/blog/2021/06/23/adding-vp9-and-mpeg2-stateless-support-in-v4l2codecs-for-gstreamer/) for an introduction to the domain before following through with this text. [This talk](https://www.youtube.com/watch?v=9wY06rusbMM) by my colleague Nicolas Dufresne is also a good resource. Stateless decoders operate as a clean slate and, in doing so, they require a lot of metadata that is read directly from the bit stream before decoding each and every frame. Note that this metadata directs the decoding, and is used to make control-flow decisions within the codec; erroneous or incorrect metadata can easily send a codec astray.

User space is responsible for parsing this metadata and feeding it to the drivers, which perform a best-effort validation routine before consuming it to get instructions on how to proceed with the decoding process. It is the kernel's responsibility to comb through and transfer this data to the hardware. The parsing algorithms are laid out by the codec specification, which are usually hundreds of pages long and subject to errata like any other technical document.

Given the above, it is easy to see the finicky nature of stateless decoder drivers. Not long ago, some researchers crafted a [program](https://github.com/h26forge/h26forge) capable of emitting a syntactically correct but semantically non-compliant H.264 stream exploiting the [weaknesses](https://github.com/h26forge/h26forge/blob/main/docs/MOTIVATION.md) that are inherent to the decoding process. Interested readers can refer themselves to the [actual paper](https://wrv.github.io/h26forge.pdf).

#### The role of Video4Linux2 codec libraries

A key aspect of hardware accelerators is that they implement significant parts of a workload in hardware, but often not all of it. For codec drivers in particular, this means that the metadata is not only used to control the decoding process in the device, it is also fed to codec algorithms that run in the CPU.

These codec algorithms are laid out in the codec's specification, and it would not make sense for each driver to have a version of that in their source code, so that part gets abstracted away as kernel libraries. To see the code implementing these codecs, look at files like [`drivers/media/v4l2-core/v4l2-vp9.c`](https://elixir.bootlin.com/linux/latest/source/drivers/media/v4l2-core/v4l2-vp9.c), [`v4l2-h264.c`](https://elixir.bootlin.com/linux/latest/source/drivers/media/v4l2-core/v4l2-h264.c), and [`v4l2-jpeg.c`](https://elixir.bootlin.com/linux/latest/source/drivers/media/v4l2-core/v4l2-jpeg.c) in the kernel sources.

What's more, with the introduction of more AV1 drivers and the proposed V4L2 Stateless Encoding API, the number of codec libraries will probably increase. With the stateless encoding API, a new challenge will be to capture parts of the metadata in the kernel successfully, bit by bit, while parsing data returned from the device. For more information on the stateless encoding initiative, see [this talk](https://www.youtube.com/watch?v=lSg_bwD-Jhw), by my colleague Andrzej Pietrasiewicz or [the mailing-list discussion](https://lwn.net/ml/linux-media/ZK2NiQd1KnraAr20@aptenodytes/). A tentative user-space API for H.264 alongside a driver for Hantro devices was also [submitted](/ml/linux-media/20231116154816.70959-1-andrzej.p@collabora.com/) last year, although the discussion is still on a RFC level.

#### Why Rust?

Security and reliability are paramount in software development; in the kernel, initiatives aimed at improving automated testing and continuous integration are gaining ground. As much as this is excellent news, it does not fix many of the hardships that stem from the use of C as the chosen programming language. The work being done by Miguel Ojeda and others in the Rust-for-Linux project has the potential to finally bring relief to problems such as complex locking, error handling, bounds checking, and hard-to-track ownership that span a large number of domains and subsystems.

Codec code is also plagued by many of the pitfalls listed above and we have discussed at length about the finicky and error-prone nature of codec algorithms and metadata. Said algorithms, as we've seen, will use the metadata to guide the control flow on the fly and also to index into various memory locations. That has been shown to be a major problem in the user-space stack, and the problem is even more critical at the kernel level.

Rust can help by making a whole class of errors impossible, thus significantly reducing the attack surface. In particular, raw pointer arithmetic and problematic `memcpy()` calls can be eliminated, array accesses can be checked at run time, and error paths can be greatly simplified. Complicated algorithms can be expressed more succinctly through the use of more modern abstractions such as iterators, ranges, generics, and the like. These add up to a more secure driver and, thus, a more secure system.

#### Porting codec code to Rust, piece by piece

If adding a layer of Rust abstractions is deemed problematic for some, a cleaner approach can focus on using Rust only where it matters by converting a few functions at a time. This technique composes well, and works by instructing the Rust compiler to generate code that obeys the C calling convention by using the `extern "C"` construct, so that existing C code in the kernel can call into Rust seamlessly. Name-mangling also has to be turned off for whatever symbols the programmer plans to expose, while a `[repr(C)]` annotation ensures that the Rust compiler will lay out structs, unions, and arrays as C would for interoperability.

Once the symbol and machine code are in the object file, calling these functions now becomes a matter of matching signatures and declarations between C and Rust. Maintaining the ABI between both layers can be challenging but, fortunately, this is a problem that is solved by employing [`cbindgen`](https://github.com/mozilla/cbindgen), a standalone tool from Mozilla that is capable of generating an equivalent C header from a Rust file.

With that header in place, the linker will do the rest, and a seamless transition into Rust will take place at run time. Once in Rust land, one can freely call other Rust functions that do not have to be annotated with `#[no_mangle]` or `extern "C"`, which is why it's advisable to use the C entry point only as a facade for the native Rust code:

```
    // The C API for C drivers.
    pub mod c {
        use super::*;

        #[no_mangle]
        pub extern "C" fn v4l2_vp9_fw_update_probs_rs(
            probs: &mut FrameContext,
            deltas: &bindings::v4l2_ctrl_vp9_compressed_hdr,
            dec_params: &bindings::v4l2_ctrl_vp9_frame,
        ) {
            super::fw_update_probs(probs, deltas, dec_params);
        }

```

In this example, `v4l2_vp9_fw_update_probs_rs()` is called from C, but immediately jumps to `fw_update_probs()`, a native Rust function where the actual implementation lives.

In a C driver, the switch is as simple as calling the `_rs()` version instead of the C version. The parameters needed by a Rust function can be neatly packed into a struct on the C side, freeing the programmer from writing abstractions for a lot of types.

#### Putting that to the test

Given the ability to rewrite error-prone C code into Rust one function at a time, I believe it is now time to rewrite our codec libraries, together with any driver code that directly accesses the bit stream parameters. Thankfully, it is easy to test codec drivers and their associated libraries, at least for decoders.

The [Fluster tool](https://github.com/fluendo/fluster) by Fluendo can automate conformance testing by running a decoder and comparing its results against that of the canonical implementation. This gives us an objective metric for regressions and, in effect, tests the whole infrastructure: from drivers, to codec libraries and, even, the V4L2 framework. My plan is to see Rust code being tested on [KernelCI](https://kernelci.org/) in the near future so as to assess its stability and establish a case for its upstreaming.

By gating any new Rust code behind a `KConfig` option, users can keep running the C implementation while the continuous-integration system tests the Rust version. It is by establishing this level of trust that I hope to see Rust gain ground in the kernel.

Readers willing to judge this initiative may refer to the [patch set](/ml/linux-media/20240227215146.46487-1-daniel.almeida@collabora.com/) I sent to the Linux media mailing list. It ports the VP9 library written by Pietrasiewicz into Rust as a proof of concept, converting both the `hantro` and `rkvdec` drivers to use the new version. It then converts error-prone parts of `rkvdec` itself into Rust, which encompasses all code touching the VP9 bit stream parameters directly, showing how Rust and C can both coexist within a driver.

So far, only one person has replied, noting that the patches did not introduce any regressions for them. I plan on discussing this idea further in the next Media Summit, the annual gathering of the kernel media developers that is yet to take place this year.

In my opinion, not only should we strive to convert the existing libraries to Rust, but we should also aim to write the new libraries that will invariably be needed directly in Rust. If this proves successful, I hope to show that there will be no more space for C codec libraries in the media tree. As for drivers, I hope to see Rust used where it matters: in places where its safety and improved ergonomics proves worth the hassle.

#### Getting involved

Those willing to contribute to this effort may start by introducing themselves to video codecs by reading the specification for their codec of choice. A good second step is to refer to GStreamer or FFmpeg to learn how stateless codec APIs can be used to drive a codec accelerator. For GStreamer, in particular, look for the [v4l2codecs](https://gitlab.freedesktop.org/gstreamer/gstreamer/-/tree/main/subprojects/gst-plugins-bad/sys/v4l2codecs?ref_type=heads) plugin. Learning `cbindgen` is better accomplished by referring to the [`cbindgen` documentation](https://github.com/mozilla/cbindgen/blob/master/docs.md) provided by Mozilla. Lastly, reading through a codec driver like `rkvdec` and the [V4L2 memory-to-memory stateless video decoder interface documentation](https://www.kernel.org/doc/html/latest/userspace-api/media/v4l/dev-stateless-decoder.html) can also be helpful.

* * *

(

[Log in](https://lwn.net/Login/?target=/Articles/970565/)

to post comments)