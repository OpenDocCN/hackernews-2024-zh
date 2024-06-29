<!--yml

类别：未分类

日期：2024-05-29 13:20:10

-->

# 对于每个遇到着色器编译“尝试分配渲染资源时内存不足”错误的人 :: 博世夜莺综合讨论

> 来源：[https://steamcommunity.com/app/1928980/discussions/0/7221029098493967556/](https://steamcommunity.com/app/1928980/discussions/0/7221029098493967556/)

对于每个遇到着色器编译“尝试分配渲染资源时内存不足”错误的人

对于每个遇到着色器编译“尝试分配渲染资源时内存不足”错误的人

您的 CPU 故障，并且存在核心错误。不是您的 GPU，而是您的 CPU。

当执行着色器编译时，出现这种错误消息（以及其他消息，包括直接 CTDs）的崩溃，通常是由于故障 CPU 引起的，具体来说是英特尔 13x 和 14x 系列，特别是（但不仅限于）13900k 和 14900k 变种。

受上述 CPU 着色器崩溃影响的任何人都只需打开事件查看器并搜索“WHEA-Logger”警告进行解析。在这些条目中，查找“Translation Look Aside Buffer”和“Internal Parity Error”。会有很多。这是一个坏的 CPU。

此外，基于 PCore 的 CPU 核心错误可以通过在 BIOS 中应用非常轻微的 OC 并设置 SVID=Typical 和 LLC=4 来显现在运行 OCCT 时浮出水面。OCCT 几乎会立即报告核心错误，并持续到其余的运行中。根据 CPU 的损坏程度，即使没有这些 BIOS 调整，您也可能会让 OCCT 报告核心错误。任何时候都要自担风险。我只是重申我是如何进行隔离的。

还需要更多证据吗？这是我在调查自己的确切问题时偶然发现的一个帖子。

[https://www.reddit.com/r/buildapc/comments/11uftum/rtx_4090_i9_13900k_pc_build_crashing_with/](https://www.reddit.com/r/buildapc/comments/11uftum/rtx_4090_i9_13900k_pc_build_crashing_with/)

且这很快就沉浸到所有相同的问题中……

[https://www.reddit.com/r/intel/comments/13o29w5/13900k_will_no_longer_run_dx12_games_crashingctds/](https://www.reddit.com/r/intel/comments/13o29w5/13900k_will_no_longer_run_dx12_games_crashingctds/)

而且列举不胜列，这只是其中的一小部分……

[https://www.reddit.com/r/remnantgame/comments/17lpz8g/13900k_not_stable_on_stock_during_first_shader/](https://www.reddit.com/r/remnantgame/comments/17lpz8g/13900k_not_stable_on_stock_during_first_shader/)[https://www.reddit.com/r/intel/comments/12bybl5/something_wrong_with_13900k/](https://www.reddit.com/r/intel/comments/12bybl5/something_wrong_with_13900k/)[https://www.reddit.com/r/intel/comments/17ku82f/13900k_degradation_i_have_faulty_cpu_and_try_my/](https://www.reddit.com/r/intel/comments/17ku82f/13900k_degradation_i_have_faulty_cpu_and_try_my/)[https://www.reddit.com/r/intel/comments/17aob62/13900k_wheas_19_or_bsod_during_shader_compilation/](https://www.reddit.com/r/intel/comments/17aob62/13900k_wheas_19_or_bsod_during_shader_compilation/)[https://www.reddit.com/r/pcmasterrace/comments/17gb3p0/13900k_out_of_video_memory_or_bsod_during_shader/](https://www.reddit.com/r/pcmasterrace/comments/17gb3p0/13900k_out_of_video_memory_or_bsod_during_shader/)

唯一的解决方案要么降低PCore倍频，我在52x时保持稳定，但可能会有变化（并且仍然会逐渐恶化，因此需要持续更多的努力），要么进行退货换货。你可以使用Intel XTU降低PCore，但使用时自负风险（尽管对于有基本PC技术理解的人来说，这是相当简单的）。

*在不到一年的时间里，我已经用了第四个13900k处理器*，到目前为止它已经工作了几个月。

这是一个CPU硬件缺陷。除了BIOS级别的降频，软件、操作系统或BIOS都无法提供稳定的系统。你可以相信我，在将近6周的自行解决系统问题后，我把问题定位到了CPU上。

----

编辑：只想明确一点；这些错误通常不会立即显现。也就是说，CPU变得受影响之前需要一段时间。在我的情况下，每个CPU在1-3个月后开始出现完全相同的行为。DX12和着色器组件，无论是在初始着色器组件还是在游戏过程中，都会使游戏客户端崩溃。

这些错误经常困扰我的一个是在Fortnite DX12中，所有设置都调至最高或从最高一步下调。

问题开始很小。偶尔的错误，如标题所示，试图加载游戏时，但点击后再试大多数时候都能工作。然后有时会工作。然后永远不工作，表明CPU正在恶化。然后即使在能加载菜单的时候，试图进入分组区域也会崩溃，并且再次出现相同的错误。检查游戏日志，指向着色器组件导致致命错误。

更换CPU解决了所有这些问题。一段时间后。但在这些情况下，游戏加载并进入比赛时，游戏会崩溃。再次检查游戏日志，显示由于遍历，着色器解压缩再次导致致命错误。

每次发生这些错误时，使用我所述的设置在OCCT中，会显示核心错误。

受影响的游戏列表很长。每个基于 DX12 着色器的游戏在初始着色器编译时至少会崩溃，或者像上面说的，在此之后的某个时候会。如果能通过着色器编译，看起来可能没问题，但只要更新 GPU 驱动程序或者游戏补丁，就会再次进行着色器编译，这就是为什么有些人觉得有时好有时坏。

--

第二次编辑，专门给那些坚信这是开发者会修复的游戏代码特定问题的人。

这个问题很普遍。只需谷歌搜索“Out of video memory trying to allocate a rendering resource”，就会有超过两百万个结果。其他游戏开发者在他们的技术支持页面上积极提到了 PCore 多人游戏解决方法，关键是他们没有在代码中修复它……

Fatshark

-

[https://support.fatshark.se/hc/zh-CN/articles/360021425793--PC-How-to-Resolve-Data-Corruption-Errors?fbclid=IwAR3sTt1GpEPV4sMYLu12zWde1ZXYj6PsYN0g3Fxx9hwrDXu8nWTh8Ni-tfw](https://steamcommunity.com/linkfilter/?u=https%3A%2F%2Fsupport.fatshark.se%2Fhc%2Fzh-CN%2Farticles%2F360021425793--PC-How-to-Resolve-Data-Corruption-Errors%3Ffbclid%3DIwAR3sTt1GpEPV4sMYLu12zWde1ZXYj6PsYN0g3Fxx9hwrDXu8nWTh8Ni-tfw)

“Intel i9 13900k/i7 13700k：将‘性能核心’速度降低

已经注意到，装有 Intel i9 13900k/i7 13700k CPU 的玩家容易遇到这些崩溃。玩家已经通过使用 Intel XTU 将‘性能核心’速度从 x55 降低到 x53 来解决这个问题。

余烬2

-

[https://www.remnantgame.com/zh-CN/news/article/11551423?fbclid=IwAR1x74b4uo7dnGLPDEUwyR87WMqiKFJM7WGN7JAFPO0KUfPzS93M_DWQ4_w](https://steamcommunity.com/linkfilter/?u=https%3A%2F%2Fwww.remnantgame.com%2Fzh-CN%2Fnews%2Farticle%2F11551423%3Ffbclid%3DIwAR1x74b4uo7dnGLPDEUwyR87WMqiKFJM7WGN7JAFPO0KUfPzS93M_DWQ4_w)

"--在加载时内存不足（Intel 第13代 CPU）--

我们已经确认在某些 Intel 第13代 CPU 上启动游戏时，会显示关于视频内存不足或崩溃报告者会弹出关于解压缩着色器问题的消息。如果你遇到这个问题，在其他 DX12 游戏中也可能会看到。

如果你的 CPU 超频了，请尝试恢复默认设置。如果你没有超频或者这样做没用，尝试安装 Intel Extreme Tuning Utility"

甚至 Epic 自己也承认了这个问题

-

[https://www.epicgames.com/help/zh-CN/fortnite-c5719335176219/technical-support-c5719372265755/out-of-video-memory-trying-to-allocate-a-rendering-resource-error-a20960841352859](https://steamcommunity.com/linkfilter/?u=https%3A%2F%2Fwww.epicgames.com%2Fhelp%2Fzh-CN%2Ffortnite-c5719335176219%2Ftechnical-support-c5719372265755%2Fout-of-video-memory-trying-to-allocate-a-rendering-resource-error-a20960841352859)

这还只是个开端。我在霍格沃茨论坛上偶然看到这个问题，最近在《自杀小队》这里看到了更多。

[https://steamcommunity.com/app/315210/discussions/0/4204741842669464852/?ctp=6](https://steamcommunity.com/app/315210/discussions/0/4204741842669464852/?ctp=6)

有人辩称 CPU 一开始就有问题，而不是逐渐恶化，但根据我之前提到的经验，情况通常是这样，有时会出错，如果再试一次游戏就会正常运行。然后过了一段时间，出错的频率增加，使得让游戏正常运行变得更加棘手。再过一段时间，根本就无法运行任何 DX12 着色器游戏，完全崩溃。
