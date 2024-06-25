<!--yml

category: 未分类

date: 2024-05-27 14:26:28

-->

# 在低端 MS-DOS 机器上播放视频 | Scali's OpenBlog™

> 来源：[`scalibq.wordpress.com/2024/01/01/video-playback-on-low-end-ms-dos-machines/`](https://scalibq.wordpress.com/2024/01/01/video-playback-on-low-end-ms-dos-machines/)

在 MS-DOS 上播放视频…在过去，这在类似 [GRASP/GLPro](https://en.wikipedia.org/wiki/Graphics_Animation_System_for_Professionals) 和 [FLIC](https://en.wikipedia.org/wiki/FLIC_(file_format)) 格式的东西中有所体现。但在真正的低端机器上，比如 8088 和 CGA 上却不是。直到 2004 年发布了演示版本 8088 Corruption。然后在 2014 年发布了它的继任者 8088 Domination，[我之前已经讨论过了](https://scalibq.wordpress.com/2014/08/13/8088-domination-by-trixterhornet/)。

它可以在 4.77 MHz 的 8088 上通过（复合）CGA 和 Sound Blaster 2.0 以非常可接受的质量播放视频。

等等，Sound Blaster 2.0？这有点年代错误，不是吗？是的，但有两个很好的理由：

1.  Sound Blaster 是第一个常见的 IBM PC 声音解决方案，可以利用 DMA 控制器进行样本播放，从而将 CPU 负载减少到接近零。

1.  [如此前讨论的](https://scalibq.wordpress.com/2017/03/12/dma-activation/)，早期的 Sound Blaster 存在一些缺点，这意味着它们无法实现无缝播放。2.0 版本是第一个解决这个问题的版本（通过在 DSP 固件中实现新命令来修复此问题。创新公司还出售 SB 1.x 卡的升级，以获取新的 2.0 固件，从而解决此问题）。

好吧，所以 Sound Blaster 2.0 是第一个 *好* 的数字音频播放解决方案。但是，有人可能会说……带有 4.77 MHz 8088 和 CGA 的 IBM PC 也不属于 ‘好’ 类别，对吧？关键是“无论如何让我们看看我们能做些什么”。如果我们也把这个应用到音频上呢？

我已经想了一段时间了。自从我在我的 IBM 5160 PC/XT 上为 PC 扬声器做了 [流媒体音频解决方案](https://scalibq.wordpress.com/2016/12/11/any-real-keeping-lately/)，我也想加入视频。在当时已经写过了结合两者的意义。过去的几天我有点空闲时间，所以我决定试试快速且肮脏的尝试。

[XDC 编码器和解码器的代码](https://x86dc.wordpress.com/) 在 [Github 上可用](https://github.com/MobyGamer/XDC)，所以我只是创建了自己的分支：

[`github.com/Scalibq/XDC`](https://github.com/Scalibq/XDC)

## 首先是 PC 扬声器

正如您从提交中看到的那样，我添加了我的代码，用于 [自动-EOI](https://scalibq.wordpress.com/2015/12/15/pc-compatibility-its-all-relative/)，以在每个中断时节省几个周期。对于 Sound Blaster 和 ‘quiet’ 选项，这是不需要的，因为每帧只有一个中断。但是对于 PC 扬声器和其他不支持 DMA 的各种声音源，您需要为每个单独的样本设置一个中断，因此我在这些情况下启用了自动-EOI。

我随后添加了一些 PC 扬声器例程以启用 PWM 播放。我使用了一个简单的转换表，将 XDV 文件中的无符号 8 位 PCM 样本转换为正确的 PWM 值。请注意，PWM 值取决于采样率，因此当文件的采样率已知时，表会在运行时初始化。

这基本上已经解决了问题。PC 扬声器的声音现在已经能够播放了。当然，这会增加额外的 CPU 开销，所以你要么需要一个更快的机器来播放相同的内容，要么需要用更低的采样率编码你的内容，可能还要降低帧率和/或编码时的 **MAXDISKRATE** 值。我不建议在 4.77 MHz 的机器上使用 PC 扬声器，但在 8 或 10 MHz 的 turbo XT 上，只要你将采样率保持在足够低，比如在 8 到 11 kHz 之间，你应该可以相对良好地使用它，只要你找到适合你具体机器的最佳设置。这可能需要一些试错来找到最适合你特定机器的设置。

## 然后是 Covox…

现在我有了一个能够按位发送样本的解决方案，我觉得我也可以添加一些其他常见的设备。其中之一是 Covox，因为 8 位无符号 PCM 样本可以直接发送到 Covox，所以这很简单。这使得它比 PC 扬声器解决方案略快。它的声音也更好听。质量几乎和声霸卡一样好。主要问题是中断驱动的样本播放比 DMA 传输更容易受到抖动的影响。在足够快的计算机上，这不会成为问题，但在 8088 上可能会听到一些轻微的抖动。

## 以及 Tandy…

Tandy 是另一个有趣的目标。XDC 已经支持 Tandy 视频模式，因此添加 Tandy 音频是有意义的。它当然可以播放 PC 扬声器音频，但 PWM 也会产生一个令人讨厌的载波波形。在 SN76496 芯片上的样本播放不会有这个问题。你可以从声音芯片的音量寄存器中获取 4 位 PCM。但音量是非线性的，因此你不能只是从 8 位源样本中获取高半字节（虽然可以工作，但并不是最佳选择）。相反，你应该使用一个从线性 8 位样本到正确非线性 4 位音量设置的转换表。这很简单。我可以使用与 PC 扬声器相同的方法，只是使用不同的表格。

然而，Tandy 给我带来了一些问题。之前我已经做过 PCjr 的样本播放，所以我以为我可以把那段代码作为起点。但是它给了我一个奇怪的低音，而且非常响亮。奇怪的是，之前没有这个问题？

PCjr 使用的是一个实际的 [Texas Instruments SN76496](https://en.wikipedia.org/wiki/Texas_Instruments_SN76489) 芯片，而 Tandy 则选择了一个克隆芯片：[NCR 8496](https://www.vgmpf.com/Wiki/index.php/NCR_8496)。你可能会认为没什么大不了的。但是有个问题：对于样本播放，你将 SN76496 设置为周期值 0。这是一个未记录的特性，会导致输出为 0 Hz，因此实际上输出始终保持高电平。你可以通过音量寄存器调制它，然后神奇地：我们有了一个 4 位 DAC。

然而，NCR 8496 没有收到那个备忘录。当你向它写入周期 0 时，它将其解释为 400h 的值，这比你通常可以设置的最大周期值 3FFh 大一。因此，这导致音调为 (基频)/1024，而不是 0 Hz。这就解释了我听到的响亮的低音。

但等等，Tandy 上的样本播放可以正常工作，不是吗？罗布·哈伯德在他为 Tandy 游戏制作的一些出色音乐中使用了样本通道，对吗？是的：

[`www.youtube.com/embed/I8_z_CI37JE?version=3&rel=1&showsearch=0&showinfo=1&iv_load_policy=1&fs=1&hl=en&autohide=2&wmode=transparent`](https://www.youtube.com/embed/I8_z_CI37JE?version=3&rel=1&showsearch=0&showinfo=1&iv_load_policy=1&fs=1&hl=en&autohide=2&wmode=transparent)

视频

那么，这是怎么做到的呢？我决定研究代码，看看它到底做了什么。确实，我发现它将周期设置为 1 而不是 0。在 NCR 8496 上，这导致它可以播放的最高频率，而不是周期为 0 时所得到的最低频率。由于这个频率远远超出了可听范围（超过 100 kHz），你实际上听不到它。但你可以通过音量寄存器听到调制效果，从而获得相同的 DAC 效果（顺便说一句，这个技巧也被用在了 [SAA1099](https://en.wikipedia.org/wiki/Philips_SAA1099) 声音芯片上，例如在 [CMS/GameBlaster](https://en.wikipedia.org/wiki/Sound_Blaster#Creative_Music_System_and_Game_Blaster) 上发现的，比如游戏 [BattleTech: The Crescent Hawk’s Revenge](https://www.mobygames.com/game/233/battletech-the-crescent-hawks-revenge/) 中）。

于是，一旦我弄清楚了 NCR 8496 和 SN76496 的区别，修复起来就很容易了。显然，我的所有代码都是正确的，我只是应该将周期设置为 1 而不是 0。

## Sound Blaster..

Sound Baster？那已经支持了，不是吗？是的，2.0 版本支持了。但是正如我[之前讨论过的](https://scalibq.wordpress.com/2017/03/12/dma-activation/)，你也可以在早期的 Sound Blaster 上实现流式播放。它可能不会完全无故障，但你可以获得可接受的结果。

在这种情况下，我只是采取了一种快速而粗糙的方法。问题在于音频缓冲区的大小与帧速率直接相关。也就是说，音频缓冲区的长度正好等于一帧的长度。这就是原始 XDC 播放器中魔术的实现方式：当音频缓冲区完成时，SB 将生成一个中断。然后中断处理程序将显示一个新的帧。这意味着音频缓冲区非常短。

正如我之前所说，这是次优的，因为每次启动新缓冲区时都会出现故障。因此，理想情况下，您希望使用尽可能大的缓冲区，以将故障降至最低。但是，至少目前我只是采取了一种快速而粗糙的解决方案，只是在每次中断时重新启动音频缓冲区。将这添加到现有代码中是微不足道的。这将导致每帧出现故障。但是，这种故障是由于在 DSP 上启动新缓冲区所需的时间。当我写这篇文章时，假设您使用高采样率。正如我在文章中所写的，向 DSP 发送新命令需要大约 316 个 CPU 循环，在 4.77 MHz 的 8088 上，而 22 kHz 的单个样本则需要 216 个 CPU 循环。显然，在较低的采样率下，每个样本所需的时间更长，而 DSP 的时间应该保持不变。所以这意味着在较低的速率下，故障变得不太明显，也许在低到一定程度的速率下，根本就不会注意到。

我可能会在某个时候重新访问这段代码，并以一种可以应用我早期文章中的一些想法的方式修改音频缓冲区。但至少目前 SB 1.x 可以工作。代码会检查 DSP 版本，因此仅在检测到 1.x SB 时才使用此回退。SB 2.0 和更高版本应该仍然像以前一样工作，因此它们不会受到此添加的任何影响。

## 现在该怎么办？

好吧，使用这种方法的一种方式是创建新内容，专门针对更低的采样率，这些采样率在这些新支持的声音设备上更有效。

另一种方法是在更快的机器上运行现有内容。8088 Domination 使用 22 kHz 音频，对于按位设备来说是一个相当糟糕的情况。但是一个相当快的 286 不会有问题。所以你应该能够在足够快的机器上原样播放 8088 Domination 内容。

第三种方法是降级现有内容。音频缓冲区存储在每个帧的末尾。在原地对缓冲区进行降采样是微不足道的。一个 Turbo XT 应该能够在音频从 22 kHz 降采样到 8-11 kHz 范围内时在 PC 扬声器/Covox 上播放视频内容，我想。

而当你无论如何修改音频时，你也可以预先应用翻译表，这样就不必在运行时进行，减少每个样本的宝贵 CPU 循环，这肯定会提高性能。一些额外的特征标志可以引入到 XDV 格式的特征字节中，以标记 PWM 编码或 Tandy 编码音频。

无论如何，随意尝试并享受乐趣。[查看下一篇帖子](https://scalibq.wordpress.com/2024/01/02/some-results-from-the-modified-xdc-movie-player-for-8088-cga/)，了解我的一些成果，以及一个修改版的 8088 Domination 可供玩耍。

[`www.youtube.com/embed/HAmL9UXRWNM?version=3&rel=1&showsearch=0&showinfo=1&iv_load_policy=1&fs=1&hl=en&autohide=2&wmode=transparent`](https://www.youtube.com/embed/HAmL9UXRWNM?version=3&rel=1&showsearch=0&showinfo=1&iv_load_policy=1&fs=1&hl=en&autohide=2&wmode=transparent)

视频
