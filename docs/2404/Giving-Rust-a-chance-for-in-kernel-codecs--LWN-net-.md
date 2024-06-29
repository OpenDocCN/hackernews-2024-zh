<!--yml

类别：未分类

日期：2024年5月27日13:33:13

-->

# 给予Rust在内核中编解码器的机会 [LWN.net]

> 来源：[https://lwn.net/Articles/970565/](https://lwn.net/Articles/970565/)

| **LWN订阅者的好处**[订阅LWN](/subscribe/)的主要好处是帮助我们继续发布内容，此外，订阅者可以立即访问所有站点内容，并使用许多额外的站点功能。请今天就注册！ |
| --- |

2024年4月26日

本文由Daniel Almeida贡献

视频播放无疑是现代消费设备中最重要的功能之一。然而，令人惊讶的是，用户们大多不知道视频数据压缩和解压缩背后复杂的工程过程，编解码器的工作被留给了在图像质量、带宽和功耗之间寻找微妙平衡的情况下。面对持续的性能压力，视频编解码器变得越来越复杂，硬件实现现在很普遍，但编程这些设备变得越来越困难，并充满了被利用的机会。我希望传达Rust如何帮助解决这个问题。

不久前，我向Linux媒体社区[提出](/ml/linux-kernel/20230406215615.122099-1-daniel.almeida@collabora.com/)，由于编解码数据特别敏感、复杂且难以解析，我们可以考虑用Rust编写一些编解码驱动程序，以从其安全性保障中获益。当时提出了一些重要的问题，特别是指出，在已经不堪重负的维护者身上增加维护Rust抽象层的成本将是一个很大的负担。因此，我又重新构思并提出了一个新的、更简单的方案；这与[Rust-for-Linux](https://rust-for-linux.com/)社区迄今为止的一般流程有些不同，它认识到我们可以在不编写整个Rust绑定层的情况下转换易出错的驱动程序部分。

#### 无状态解码器的危险

我在Collabora的大部分博客文章都集中讨论有状态和无状态编解码器API之间的差异。在开始阅读本文之前，我建议先快速阅读[这篇文章](https://www.collabora.com/news-and-blog/blog/2021/06/23/adding-vp9-and-mpeg2-stateless-support-in-v4l2codecs-for-gstreamer/)来介绍这一领域。我的同事Nicolas Dufresne的[这次演讲](https://www.youtube.com/watch?v=9wY06rusbMM)也是一个很好的资源。无状态解码器以干净的方式运作，因此它们需要直接从位流中读取的大量元数据，以在解码每一帧之前进行控制。需要注意的是，这些元数据指导解码过程，并用于在编解码器内部做出控制流决策；错误或不正确的元数据可能会轻易使编解码器误入歧途。

用户空间负责解析这些元数据并将其提供给驱动程序，驱动程序在使用前执行尽力而为的验证例程以获取关于如何继续解码过程的指令。内核的责任是浏览和传输这些数据到硬件。解析算法由编解码规范详细说明，通常是数百页长，并像其他技术文档一样存在勘误。

通过上述内容，很容易看出无状态解码器驱动程序的棘手性质。不久前，一些研究人员开发了一个[程序](https://github.com/h26forge/h26forge)，能够生成语法上正确但在语义上不符合的 H.264 流，利用了解码过程中固有的[弱点](https://github.com/h26forge/h26forge/blob/main/docs/MOTIVATION.md)。有兴趣的读者可以参考[实际论文](https://wrv.github.io/h26forge.pdf)。

#### Video4Linux2 编解码库的角色

硬件加速器的一个关键方面是它们在硬件中实现了工作负载的重要部分，但通常不是全部。特别是对于编解码驱动程序，这意味着元数据不仅用于控制设备中的解码过程，还供给在 CPU 中运行的编解码算法。

这些编解码算法在编解码规范中详细说明，每个驱动程序的源代码中都不会有它们的版本，因此这部分会被抽象为内核库。要查看实现这些编解码器的代码，请查看内核源代码中的文件，例如[`drivers/media/v4l2-core/v4l2-vp9.c`](https://elixir.bootlin.com/linux/latest/source/drivers/media/v4l2-core/v4l2-vp9.c)，[`v4l2-h264.c`](https://elixir.bootlin.com/linux/latest/source/drivers/media/v4l2-core/v4l2-h264.c) 和 [`v4l2-jpeg.c`](https://elixir.bootlin.com/linux/latest/source/drivers/media/v4l2-core/v4l2-jpeg.c)。

另外，随着越来越多的 AV1 驱动程序的引入以及建议的 V4L2 无状态编码 API 的提出，编解码库的数量可能会增加。有了无状态编码 API，成功捕获内核中部分元数据的一个新挑战是逐位地解析从设备返回的数据。关于无状态编码倡议的更多信息，请参阅[此讲座](https://www.youtube.com/watch?v=lSg_bwD-Jhw)，由我的同事安德烈·皮耶特拉谢维奇主讲，或者参考[邮件列表讨论](https://lwn.net/ml/linux-media/ZK2NiQd1KnraAr20@aptenodytes/)。去年还提交了一种 H.264 用户空间 API 和 Hantro 设备的驱动程序，尽管讨论仍处于 RFC 阶段。

#### 为什么选择 Rust？

软件开发中安全性和可靠性至关重要；在内核中，旨在改善自动化测试和持续集成的倡议正在取得进展。尽管这是个好消息，但并不能解决因选择 C 作为编程语言而产生的许多困难。Miguel Ojeda 和其他人在 Rust-for-Linux 项目中的工作有望最终解决诸如复杂的锁定、错误处理、边界检查和跨多个领域和子系统的难以跟踪所有权问题，从而带来缓解。

编解码器代码也受到上述许多问题的困扰，我们已经详细讨论了编解码算法和元数据的挑剔和容易出错的性质。正如我们所见，这些算法将使用元数据来动态引导控制流，还将用它们来索引到各种内存位置。这在用户空间堆栈中已经显示出是一个主要问题，在内核级别则更为严重。

Rust 可以通过使一类错误变得不可能来帮助，从而显著减少攻击面。特别是，可以在运行时检查原始指针算术和有问题的 `memcpy()` 调用，简化错误路径。通过使用迭代器、范围、泛型等现代抽象，可以更简洁地表达复杂算法。这些都会导致更安全的驱动程序，从而实现更安全的系统。

#### 逐步将编解码器代码移植到 Rust

如果认为添加 Rust 抽象层存在问题，更清晰的方法可以集中精力，仅在需要时逐步转换为 Rust。这种技术组合良好，通过使用 `extern "C"` 结构指示 Rust 编译器生成遵循 C 调用约定的代码，以便内核中现有的 C 代码可以无缝调用 Rust。还必须关闭名称重整，对计划暴露的任何符号使用 `[repr(C)]` 注解，确保 Rust 编译器将结构体、联合体和数组按照 C 的布局进行排列，以实现互操作性。

一旦符号和机器码进入目标文件，调用这些函数现在就成为在 C 和 Rust 之间匹配签名和声明的问题。在两个层之间保持 ABI 可能具有挑战性，但幸运的是，这个问题可以通过使用 [`cbindgen`](https://github.com/mozilla/cbindgen) 解决，这是 Mozilla 的一个独立工具，能够从 Rust 文件生成一个等效的 C 头文件。

有了这个头文件，链接器将会完成剩下的工作，在运行时无缝地过渡到 Rust。进入 Rust 领域后，可以自由调用其他不需要标注 `#[no_mangle]` 或 `extern "C"` 的 Rust 函数，因此建议仅将 C 入口点作为本机 Rust 代码的外观：

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

在这个例子中，从C中调用`v4l2_vp9_fw_update_probs_rs()`，但立即跳转到`fw_update_probs()`，这是一个本地的Rust函数，其中实现了实际功能。

在C驱动程序中，转换只需简单调用`_rs()`版本而不是C版本。Rust函数需要的参数可以在C端整齐地打包到一个结构体中，使程序员无需为许多类型编写抽象。

#### 把它放到测试中

鉴于现在可以逐个函数将易出错的C代码重写为Rust，我认为现在是时候重写我们的编解码库了，包括直接访问比特流参数的任何驱动程序代码。幸运的是，至少对于解码器来说，测试编解码器驱动程序及其相关库非常容易。

Fluendo的[Fluster工具](https://github.com/fluendo/fluster)可以通过运行解码器并将其结果与规范实现进行比较来自动化一致性测试。这为我们提供了一个关于退化的客观指标，并有效地测试了整个基础设施：从驱动程序到编解码库，甚至V4L2框架。我计划在不久的将来在[KernelCI](https://kernelci.org/)上测试Rust代码，以评估其稳定性并为其上游提供案例。

通过将任何新的Rust代码门控在`KConfig`选项后面，用户可以继续运行C实现，同时持续集成系统测试Rust版本。通过建立这种信任，我希望看到Rust在内核中获得进展。

有意评估这一倡议的读者可以参考我发送到Linux媒体邮件列表的[补丁集](/ml/linux-media/20240227215146.46487-1-daniel.almeida@collabora.com/)。它将由Pietrasiewicz编写的VP9库作为概念验证移植到Rust中，转换了`hantro`和`rkvdec`驱动程序以使用新版本。然后将`rkvdec`本身的容易出错部分转换为Rust，包括所有直接接触VP9比特流参数的代码，展示了Rust和C如何在驱动程序中共存。

到目前为止，只有一个人回复说这些补丁对他们没有引入任何退化。我计划在下一次媒体峰会上进一步讨论这个想法，这是一年一度的核心媒体开发者聚会，今年尚未举行。

在我看来，我们不仅应该努力将现有库转换为Rust，还应该立志直接用Rust编写不可避免需要的新库。如果这证明成功，我希望表明在媒体树中将不再容纳C编解码库的空间。至于驱动程序，我希望看到Rust在真正需要其安全性和改进人机工程学的地方得到应用。

#### 参与其中

有意贡献于此工作的人可以先通过阅读所选择编解码器的规范来了解视频编解码器。一个很好的第二步是参考 GStreamer 或 FFmpeg，了解如何使用无状态编解码器 API 驱动编解码器加速器。特别是对于 GStreamer，可以查找[v4l2codecs](https://gitlab.freedesktop.org/gstreamer/gstreamer/-/tree/main/subprojects/gst-plugins-bad/sys/v4l2codecs?ref_type=heads)插件。通过参考由 Mozilla 提供的[`cbindgen` 文档](https://github.com/mozilla/cbindgen/blob/master/docs.md)，可以更好地掌握 `cbindgen`。最后，阅读诸如 `rkvdec` 和[V4L2 内存到内存无状态视频解码器接口文档](https://www.kernel.org/doc/html/latest/userspace-api/media/v4l/dev-stateless-decoder.html)也会有所帮助。

* * *

（

[登录](https://lwn.net/Login/?target=/Articles/970565/)

发表评论）
