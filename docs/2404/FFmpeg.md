<!--yml

类别：未分类

日期：2024年5月27日 12:57:30

-->

# FFmpeg

> 来源：[https://ffmpeg.org//index.html#pr7.0](https://ffmpeg.org//index.html#pr7.0)

## 一个完整的跨平台解决方案，用于录制、转换和流传输音频和视频。

### 转换**视频**和**音频**从未如此简单。

```
$ ffmpeg -i input.mp4 output.avi
```

# [](main.rss)**  [](https://twitter.com/FFmpeg)**  [](https://www.facebook.com/ffmpeg)*********新闻*****

*****### 2024年5月13日，Sovereign Tech Fund

FFmpeg社区高兴地宣布，德国的[Sovereign Tech Fund](https://www.sovereigntechfund.de/tech/ffmpeg)已成为其首个政府赞助商。他们的支持将有助于维护FFmpeg项目，这是一个关键的开源软件多媒体组件，每天为全球数十亿用户带来音频和视频。

### 2024年4月5日，FFmpeg 7.0 "Dijkstra"

新的主要版本，[FFmpeg 7.0 "Dijkstra"](download.html#release_7.0)，现已提供下载。对大多数用户而言，最显著的变化是一个[原生VVC解码器](#vvcdec)（目前实验性，需进一步模糊测试），[IAMF支持](#iamf)，以及一个[多线程`ffmpeg` CLI工具](#cli_threading)。

此版本*不*向后兼容，删除了6.0之前弃用的API。对大多数库调用者而言，最大的变化是删除了基于旧位掩码的通道布局API，改为`AVChannelLayout` API，支持自定义通道顺序或环绕声等功能。还删除了某些弃用的`ffmpeg` CLI选项，并且现在需要使用符合C11标准的编译器来构建代码。

与6.1相比，git仓库包含了近2000个新提交，由约100名作者完成，触及了大约2000个文件中的>100000行代码 — 感谢所有贡献者。更详尽的变更列表，请查看[Changelog](https://git.videolan.org/?p=ffmpeg.git;a=blob_plain;f=Changelog;hb=n7.0)，[APIchanges](https://git.videolan.org/?p=ffmpeg.git;a=blob_plain;f=doc/APIchanges;hb=n7.0)，以及git日志。

### 2024年1月3日，原生VVC解码器

`libavcodec`库现在包含一个原生VVC（通用视频编码）解码器，支持该编解码器功能的大部分子集。即将推出更多的优化和更多功能的支持。代码由Nuo Mi、Xu Mu、Frank Plowman、Shaun Loo和吴建华编写。

### 2023年12月18日，IAMF支持

`libavformat`库现在可以读取和写入[IAMF](https://aomediacodec.github.io/iamf/)（沉浸式音频）文件。`ffmpeg` CLI工具可以使用新的`-stream_group`选项配置IAMF结构。IAMF支持由James Almer编写。

### 2023年12月12日，多线程`ffmpeg` CLI工具

由于对 `ffmpeg` 命令行工具进行了重大重构，转码管道的所有主要组件（解复用器、解码器、滤镜、编码器、复用器）现在可以并行运行。这应该提高吞吐量和 CPU 利用率，减少延迟，并为其他新功能铺平道路。

请注意，如果几乎所有计算时间都花费在单个组件（通常是视频编码）上，则不应期望显著的性能改进。

### 2023 年 11 月 10 日，FFmpeg 6.1 "Heaviside"

[FFmpeg 6.1 "Heaviside"](download.html#release_6.1)，一个新的主要版本，现在已经发布！一些亮点包括：

+   libaribcaption 解码器

+   Playdate 视频解码器和解混流器

+   在 Windows 上扩展了对 libva-win32 的 VAAPI 支持

+   afireqsrc 音频源滤镜

+   arls 滤镜

+   ffmpeg 命令行新选项：-readrate_initial_burst

+   zoneplate 视频源滤镜

+   setpts 和 asetpts 滤镜中的命令支持

+   Vulkan 解码硬件加速器，支持 H264、HEVC 和 AV1

+   color_vulkan 滤镜

+   bwdif_vulkan 滤镜

+   nlmeans_vulkan 滤镜

+   RivaTuner 视频解码器

+   xfade_vulkan 滤镜

+   vMix 视频解码器

+   Essential Video Coding 解析器、混流器和解混流器

+   Essential Video Coding 帧合并比特流过滤器

+   bwdif_cuda 滤镜

+   Microsoft RLE 视频编码器

+   Raw AC-4 混流器和解混器

+   Raw VVC 比特流解析器、混流器和解混流器

+   位流滤镜，用于编辑 VVC 流中的元数据

+   位流滤镜，用于将 VVC 从 MP4 转换为 Annex B

+   videotoolbox 的 scale_vt 滤镜

+   videotoolbox 的 transpose_vt 滤镜

+   支持 P_SKIP 暗示以加速 libx264 编码

+   在增强的 flv 格式中支持 HEVC、VP9、AV1 编解码器

+   apsnr 和 asisdr 音频滤镜

+   OSQ 解混器和解码器

+   在增强的 rtmp 协议中支持 HEVC、VP9、AV1 编解码器 fourcclist

+   CRI USM 解混器

+   ffmpeg CLI 中的 '-top' 选项已弃用，推荐使用 setfield 滤镜

+   VAAPI AV1 编码器

+   ffprobe XML 输出模式已更改，以适应同一父元素内多个可变字段元素的情况

+   添加了 ffprobe 的 -output_format 选项，作为 -of 的别名

这个发布至少拖延了半年，但由于存储库中的持续活动，不得不延迟，最近我们终于能够分支发布，在计划的 7.0 大变更合并之前完成了发布。

内部，我们也进行了一些变更。用于编解码器和滤镜的 FFT、MDCT、DCT 和 DST 实现已完全替换为更快的 libavutil/tx（有关详细信息的完整文章即将发布）。

这也导致编译二进制文件的大小减小，这在小型构建中可能是显著的。

每帧视频解码器中的总分配量大幅减少，从而降低了开销。

RISC-V 优化已合并到我们的 DSP 代码的许多部分，主要留下了大型解码器。

致力于改进每个包的时间戳和帧持续时间的准确性，增加了可变帧率视频的准确性。

下一个主要版本将是版本 7.0，计划于二月发布。我们将尝试更好地坚持我们在年初宣布的新发布计划。

我们强烈建议用户、分发商和系统集成商升级，除非他们使用当前的 git 主分支。

### 2023年5月31日，Vulkan 解码

几天前，Vulkan 引擎驱动的解码硬件加速代码被合并到了代码库中。这是第一个支持多个平台、厂商通用的解码加速 API，可以在多个平台上使用相同的代码，且开销非常小。这也是第一个多线程硬件解码 API，我们的代码充分利用了这一点，使得硬件暴露的所有解码引擎都被充分利用。

想要测试该代码的人可以查阅我们的[文档页面](https://trac.ffmpeg.org/wiki/HWAccelIntro#Vulkan)。对于那些希望将 FFmpeg 的 Vulkan 代码集成到分离、解析、解码并获取 VkImage 进行呈现或操作的人来说，我们的源代码树中提供了文档和示例。目前，需要使用我们的[代码库](https://git.videolan.org/?p=ffmpeg.git;a=summary)的最新可用 git 检出版本。这些功能将会随着版本 6.1 的发布很快包含在稳定分支中。

由于这也是规范的首次实际实现，可能存在 bug，特别是在驱动程序以及实现本身经过验证后。Khronos 组织正在进行标准化的新编解码器和编码支持，并且我们也在实施并提供改进反馈。

### 2023年2月28日，FFmpeg 6.0 "Von Neumann"

新的主要版本发布了，[FFmpeg 6.0 "Von Neumann"](download.html#release_6.0)，现在可以下载。这个版本包含了许多新的编码器和解码器、过滤器、ffmpeg 命令行工具的改进，还改变了发布方式。所有主要版本现在都会升级 ABI 版本。我们计划每年发布一个新的主要版本。另一个与版本发布相关的改变是，废弃的 API 将在3个版本后的下一个主要升级中移除。这意味着发布将更加频繁且更加有组织。

新增的解码器包括Bonk、RKA、Radiance、SC-4、APAC、VQC、WavArc和几种ADPCM格式。QSV和NVenc现在支持AV1编码。FFmpeg CLI（通常称为ffmpeg.c以避免混淆）由于多线程而有速度提升，还新增了统计选项以及从文件传递滤镜选项值的能力。此版本包含许多幕后改进，包括用于编解码器的新FFT和MDCT实现（预计很快会发布有关此的博客文章），大量的bug修复，更好的ICC配置文件处理和色彩空间信号改进，引入了一些RISC-V向量和标量汇编优化例程，以及一些新的改进API，详见我们的doc/APIchanges文件。一些提交的功能，如Vulkan改进和更多的FFT优化，将包含在下一个小版本6.1中，我们计划按照新的发布时间表很快发布。一些亮点包括：

+   Radiance HDR图像支持。

+   ddagrab（桌面复制）视频捕获滤镜。

+   ffmpeg的-shortest_buf_duration选项。

+   ffmpeg现在要求构建时使用多线程。

+   ffmpeg现在在单独线程中运行每个复用器。

+   添加新模式到cropdetect滤镜，以基于运动矢量和边缘检测裁剪区域。

+   VAAPI解码和编码支持10/12位422、10/12位444 HEVC和VP9。

+   WBMP（无线应用协议位图）图像格式。

+   a3dscope滤镜。

+   bonk解码器和解封装器。

+   Micronas SC-4音频解码器。

+   LAF解封装器。

+   APAC解码器和解封装器。

+   Media 100i解码器。

+   DTS到PTS重新排序比特流格式。

+   ViewQuest VQC解码器。

+   backgroundkey滤镜。

+   nvenc AV1编码支持。

+   通过NDKMediaCodec的MediaCodec解码器。

+   MediaCodec编码器。

+   oneVPL对QSV的支持。

+   QSV AV1编码器。

+   QSV解码和编码支持10/12位422、10/12位444 HEVC和VP9。

+   showcwt多媒体滤镜。

+   corr视频滤镜。

+   adrc音频滤镜。

+   afdelaysrc音频滤镜。

+   WADY DPCM解码器和解封装器。

+   CBD2 DPCM解码器。

+   ssim360视频滤镜。

+   ffmpeg CLI新选项：-stats_enc_pre[_fmt]、-stats_enc_post[_fmt]、-stats_mux_pre[_fmt]。

+   hstack_vaapi、vstack_vaapi和xstack_vaapi滤镜。

+   XMD ADPCM解码器和解封装器。

+   media100到mjpegb比特流格式。

+   ffmpeg CLI新选项：-fix_sub_duration_heartbeat。

+   WavArc解码器和解封装器。

+   CrystalHD解码器已弃用。

+   SDNS解封装器。

+   RKA解码器和解封装器。

+   ffmpeg CLI中的filtergraph语法现在支持将文件内容作为选项值传递。

+   hstack_qsv、vstack_qsv和xstack_qsv滤镜。

我们强烈建议用户、分销商和系统集成商升级，除非他们使用当前的git主分支。

### 2022年7月22日，FFmpeg 5.1 "Riemann"。

[FFmpeg 5.1 "Riemann"](download.html#release_5.1)，一个新的主要版本，现在已经发布！一些亮点包括：

+   添加了IPFS/IPNS协议支持。

+   dialogue enhance音频滤镜。

+   放弃了过时的XvMC硬件加速器。

+   pcm-bluray编码器。

+   DFPWM音频编码器/解码器和原始复用器/解封装器。

+   SITI滤镜。

+   Vizrt 二进制图像编解码器

+   avsynctest 源滤波器

+   feedback 视频滤镜

+   pixelize 视频滤镜

+   colormap 视频滤镜

+   色卡 视频源滤镜

+   multiply 视频滤镜

+   PGS 字幕帧合并比特流滤镜

+   blurdetect 滤镜

+   tiltshelf 音频滤波器

+   QOI 图像格式支持

+   ffprobe 的 -o 选项

+   virtualbass 音频滤波器

+   VDPAU AV1 硬解加速

+   PHM 图像格式支持

+   remap_opencl 滤波器

+   添加了 chromakey_cuda 滤波器

我们强烈建议用户、发行版和系统集成商升级，除非他们使用当前的 git 主分支。

### 2022年1月17日，FFmpeg 5.0 "Lorentz"

[FFmpeg 5.0 "Lorentz"](download.html#release_5.0)，这是一次重要的新版本发布！在这个期待已久的版本中，进行了大量工作以删除旧的编码/解码 API，并用 N:M 基础的 API 替换它们，整个 libavresample 库已被移除，libswscale 现在有一个新的、更易于使用的基于 AVframe 的 API，Vulkan 代码得到了大幅改进，添加了许多新的滤镜，包括 libplacebo 集成，最后，增加了 DoVi 支持，包括色调映射和重混。默认的 AAC 编码器设置也进行了更改以提高质量。以下是部分更新日志亮点：

+   ADPCM IMA Westwood 编码器

+   Westwood AUD 复用器

+   ADPCM IMA Acorn Replay 解码器

+   Argonaut Games CVG 解复用器

+   Argonaut Games CVG 复用器

+   Concatf 协议

+   afwtdn 音频滤波器

+   音频和视频段滤波器

+   苹果图形（SMC）编码器

+   hsvkey 和 hsvhold 视频滤镜

+   adecorrelate 音频滤波器

+   atilt 音频滤波器

+   灰世界 视频滤镜

+   AV1 低开销比特流格式复用器

+   swscale 切片线程

+   MSN Siren 解码器

+   scharr 视频滤镜

+   apsyclip 音频滤波器

+   morpho 视频滤镜

+   amr 解析器

+   (a)latency 滤波器

+   GEM 光栅图像解码器

+   asdr 音频滤波器

+   speex 解码器

+   limitdiff 视频滤镜

+   xcorrelate 视频滤镜

+   varblur 视频滤镜

+   huesaturation 视频滤镜

+   色谱源视频滤镜

+   RTP 无压缩视频打包器（RFC 4175）

+   bitpacked 编码器

+   VideoToolbox VP9 硬解加速

+   VideoToolbox ProRes 硬解加速

+   支持 loongarch。

+   aspectralstats 音频滤波器

+   adynamicsmooth 音频滤波器

+   libplacebo 滤镜

+   vflip_vulkan、hflip_vulkan 和 flip_vulkan 滤波器

+   adynamicequalizer 音频滤波器

+   yadif_videotoolbox 滤波器

+   VideoToolbox ProRes 编码器

+   anlmf 音频滤波器

我们强烈建议用户、发行版和系统集成商升级，除非他们使用当前的 git 主分支。

### 2021年6月19日，IRC

我们现在在 Libera Chat 上有一个新的 IRC 主页！欢迎加入我们在 #ffmpeg 和 #ffmpeg-devel 的讨论。更多信息请见 [联系#IRCChannels](https://ffmpeg.org/contact.html#IRCChannels)

### 2021年4月8日，FFmpeg 4.4 "Rao"

[FFmpeg 4.4 "Rao"](download.html#release_4.4)，这是一次重要的新版本发布！以下是一些亮点：

+   AudioToolbox 输出设备

+   MacCaption 解复用器

+   PGX 解码器

+   chromanr 视频滤镜

+   VDPAU 加速 HEVC 10/12bit 解码

+   ADPCM IMA Ubisoft APM 编码器

+   Rayman 2 APM 复用器

+   AV1 编码支持 SVT-AV1

+   Cineform HD 编码器

+   ADPCM Argonaut Games 编码器

+   Argonaut Games ASF 复用器

+   AV1 低开销比特流格式解复用器

+   RPZA 视频编码器

+   ADPCM IMA MOFLEX 解码器

+   MobiClip FastAudio 解码器

+   MobiClip 视频解码器

+   MOFLEX 解复用器

+   MODS 解复用器

+   PhotoCD 解码器

+   MCA 解复用器

+   AV1 解码器（仅使用硬件加速）

+   SVS 解复用器

+   Argonaut Games BRP 解复用器

+   DAT 解复用器

+   aax 解复用器

+   IPU 解码器、解析器和解复用器

+   Intel QSV 加速的 AV1 解码

+   Argonaut Games 视频解码器

+   移除 libwavpack 编码器

+   ACE 解复用器

+   AVS3 解复用器

+   通过 libuavs3d 支持的 AVS3 视频解码器

+   Cintel RAW 解码器

+   VDPAU 加速的 VP9 10/12bit 解码

+   afreqshift 和 aphaseshift 滤镜

+   High Voltage Software ADPCM 编码器

+   LEGO Racers ALP (.tun & .pcm) 复用器

+   AV1 VAAPI 解码器

+   adenorm 滤镜

+   ADPCM IMA AMV 编码器

+   AMV 复用器

+   NVDEC AV1 硬件加速

+   DXVA2/D3D11VA 硬件加速的 AV1 解码

+   speechnorm 滤镜

+   SpeedHQ 编码器

+   asupercut 滤镜

+   asubcut 滤镜

+   Microsoft Paint (MSP) 版本 2 解码器

+   Microsoft Paint (MSP) 解复用器

+   AV1 单色编码支持，通过 libaom >= 2.0.1

+   asuperpass 和 asuperstop 滤镜

+   shufflepixels 滤镜

+   tmidequalizer 滤镜

+   estdif 滤镜

+   epx 滤镜

+   Dolby E 解析器

+   剪切滤镜

+   kirsch 滤镜

+   色温滤镜

+   色彩对比度滤镜

+   PFM 编码器

+   色彩校正滤镜

+   binka 解复用器

+   XBM 解析器

+   xbm_pipe 解复用器

+   色彩化滤镜

+   CRI 解析器

+   aexciter 音频滤镜

+   曝光视频滤镜

+   单色视频滤镜

+   setts 比特流滤镜

+   vif 视频滤镜

+   OpenEXR 图像编码器

+   Simbiosis IMX 解码器

+   Simbiosis IMX 解复用器

+   Digital Pictures SGA 解复用器和解码器

+   TTML 字幕编码器和复用器

+   identity 视频滤镜

+   msad 视频滤镜

+   gophers 协议

+   通过 librist 的 RIST 协议

我们强烈建议用户、分销商和系统集成商升级，除非他们使用当前的 git 主分支。

### 2020 年 6 月 15 日，FFmpeg 4.3 "4:3"

[FFmpeg 4.3 "4:3"](download.html#release_4.3)，一个新的主要版本，现已发布！其中一些亮点：

+   v360 滤镜

+   Intel QSV 加速的 MJPEG 解码

+   Intel QSV 加速的 VP9 解码

+   mp4 中的 TrueHD 支持

+   在 Linux 上支持 AMD AMF 编码器（通过 Vulkan）

+   IMM5 视频解码器

+   ZeroMQ 协议

+   支持 Sipro ACELP.KELVIN 解码

+   streamhash 复用器

+   sierpinski 视频源

+   滚动视频滤镜

+   光敏感滤镜

+   ANLMS 滤镜

+   arnndn 滤镜

+   双边滤镜

+   maskedmin 和 maskedmax 滤镜

+   VDPAU 加速的 VP9 解码

+   中值滤镜

+   QSV 加速的 VP9 编码

+   AV1 编码支持，通过 librav1e

+   AV1 帧合并比特流滤镜

+   AV1 Annex B 解复用器

+   axcorrelate 滤镜

+   mvdv 解码器

+   mvha 解码器

+   mp4 中的 MPEG-H 3D 音频支持

+   直方图滤镜

+   freezeframes 滤镜

+   Argonaut Games ADPCM 解码器

+   Argonaut Games ASF 解复用器

+   xfade 视频滤镜

+   xfade_opencl 滤镜

+   afirsrc 音频滤镜源

+   pad_opencl 滤镜

+   Simon & Schuster Interactive ADPCM 解码器

+   Real War KVAG 解复用器

+   CDToons 视频解码器

+   siren 音频解码器

+   Rayman 2 ADPCM 解码器

+   Rayman 2 APM 解复用器

+   cas 视频滤镜

+   High Voltage Software ADPCM 解码器

+   LEGO Racers ALP (.tun & .pcm) 解复用器

+   AMQP 0-9-1 协议（RabbitMQ）

+   Vulkan 支持

+   avgblur_vulkan、overlay_vulkan、scale_vulkan 和 chromaber_vulkan 滤镜

+   ADPCM IMA MTF 解码器

+   FWSE 解复用器

+   DERF DPCM 解码器

+   DERF 解复用器

+   CRI HCA 解码器

+   CRI HCA 解复用器

+   overlay_cuda 滤镜

+   在 Linux 上从 AvxSynth 切换到 AviSynth+

+   mv30 解码器

+   扩展了对于 3GPP 定时文本字幕（movtext）的样式支持

+   WebP 解析器

+   tmedian 滤镜

+   maskedthreshold 滤镜

+   在 m2ts 中复用 pcm 和 pgs 的支持

+   Cunning Developments ADPCM 解码器

+   asubboost 滤镜

+   Pro Pinball 系列声音库解复用器

+   pcm_rechunk 比特流滤镜

+   scdet 滤镜

+   NotchLC 解码器

+   gradients 源视频滤镜

+   MediaFoundation 编码器封装器

+   untile 滤镜

+   Simon & Schuster Interactive ADPCM 编码器

+   PFM 解码器

+   dblur 视频滤镜

+   Real War KVAG 复用器

我们强烈建议用户、分销商和系统集成商升级，除非他们使用当前的 git 主分支。

### 2019年10月5日，Bright Lights

FFmpeg 已经在 libavfilter 中添加了一个实时亮度闪烁去除滤镜。

请注意，此滤镜未经 FDA 批准，我们也不是医疗专业人士。此滤镜也未经过与光敏感性癫痫患者的测试。FFmpeg 及其光敏感滤镜不作任何医学声明。

尽管如此，这是一个新的视频滤镜，可能有助于光敏感人士观看电视、玩视频游戏，甚至可以与 VR 头盔一起使用，以阻止如外出时的阳光滤过等癫痫触发器。或者您可以用它来对付电视屏幕上那些恼人的白色闪光。该滤镜对一些输入失败，例如[超人总动员2的屏幕奴隶](https://www.youtube.com/watch?v=8L_9hXnUzRk)场景。它并不完美。如果您有其他希望这个滤镜能够更好工作的视频片段，请在我们的[trac](http://trac.ffmpeg.org)上向我们报告。

[亲自看看](http://ffmpeg.org/~compn/output20p8.mp4)。例子是用 -vf photosensitivity=20:0.8 制作的。

我们不是专业人士。请在医学研究中使用这个以促进癫痫研究。如果您决定在医学环境中使用这个，或者制作一个硬件 HDMI 输入输出实时电视滤镜，或者找到其他用途，请[告诉我](mailto:compn@ffmpeg.org)。这个滤镜是我[2013年](https://trac.ffmpeg.org/ticket/2104)提出的一个功能请求。

### 2019年8月5日，FFmpeg 4.2 "Ada"

[FFmpeg 4.2 "Ada"](download.html#release_4.2)，一个新的重要版本，现已发布！以下是一些亮点：

+   tpad 滤镜

+   通过 libdav1d 支持 AV1 解码

+   dedot 滤镜

+   chromashift 和 rgbashift 滤镜

+   freezedetect 滤镜

+   truehd_core 比特流滤镜

+   dhav 解复用器

+   PCM-DVD 编码器

+   GIF 解析器

+   vividas 解复用器

+   hymt 解码器

+   anlmdn 滤镜

+   maskfun 滤镜

+   hcom 解复用器和解码器

+   ARBC 解码器

+   基于 libaribb24 的 ARIB STD-B24 字幕支持（A 和 C 配置文件）

+   支持在 nvdec 和 cuviddec 中解码 HEVC 4:4:4 内容

+   移除了 libndi-newtek

+   agm 解码器

+   KUX 解复用器

+   AV1 帧分割比特流滤镜

+   lscr 解码器

+   lagfun 滤镜

+   asoftclip 滤镜

+   支持在 vdpau 中解码 HEVC 4:4:4 内容

+   colorhold 滤镜

+   xmedian 滤镜

+   asr 滤镜

+   showspatial 多媒体滤镜

+   VP4 视频解码器

+   IFV 解复用器

+   derain 滤镜

+   deesser 滤镜

+   mov 复用器默认将轨道写入未指定语言而不是英语

+   增加了使用 clang 编译 CUDA 内核的支持

我们强烈建议用户、分发商和系统集成商升级，除非他们使用当前的 git 主分支。

### 2018 年 11 月 6 日，FFmpeg 4.1 "al-Khwarizmi"

[FFmpeg 4.1 "al-Khwarizmi"](download.html#release_4.1)，一个新的主要版本，现已发布！一些亮点包括：

+   deblock 滤镜

+   tmix 滤镜

+   amplify 滤镜

+   fftdnoiz 滤镜

+   aderivative 和 aintegral 音频滤镜

+   pal75bars 和 pal100bars 视频滤镜源

+   基于 mbedTLS 的 TLS 支持

+   adeclick 和 adeclip 滤镜

+   libtensorflow 后端用于基于 DNN 的滤镜如 srcnn

+   VC1 解码器现在是比特准确的

+   ATRAC9 解码器

+   lensfun 包装滤镜

+   colorconstancy 滤镜

+   AVS2 视频解码器通过 libdavs2

+   IMM4 视频解码器

+   Brooktree ProSumer 视频解码器

+   MatchWare Screen Capture Codec 解码器

+   WinCam Motion Video 解码器

+   1D LUT 滤镜（lut1d）

+   RemotelyAnywhere Screen Capture 解码器

+   cue 和 acue 滤镜

+   支持将 AV1 编码到 MP4 和 Matroska/WebM 中

+   transpose_npp 滤镜

+   AVS2 视频编码器通过 libxavs2

+   amultiply 滤镜

+   块匹配 3D（bm3d）降噪滤镜

+   acrossover 滤镜

+   ilbc 解码器

+   afftdn 作为音频降噪器的 afftdn 滤镜

+   AV1 解析器

+   sinc 音频滤镜源

+   chromahold 滤镜

+   setparams 滤镜

+   vibrance 滤镜

+   h264 中的 S12M 时间码解码

+   xstack 滤镜

+   (a)graphmonitor 滤镜

+   yadif_cuda 滤镜

我们强烈建议用户、分发商和系统集成商升级，除非他们使用当前的 git 主分支。

### 2018 年 4 月 20 日，FFmpeg 4.0 "Wu"

[FFmpeg 4.0 "Wu"](download.html#release_4.0)，一个新的主要版本，现已发布！一些亮点包括：

+   用于编辑 H.264、HEVC 和 MPEG-2 流中元数据的比特流滤镜

+   实验性 MagicYUV 编码器

+   TiVo ty/ty+ 分离器

+   Intel QSV 加速的 MJPEG 编码

+   本地 aptX 和 aptX HD 编解码器

+   NVIDIA NVDEC 加速的 H.264、HEVC、MJPEG、MPEG-1/2/4、VC1、VP8/9 硬件加速解码

+   Intel QSV 加速的叠加滤镜

+   mcompand 音频滤波器

+   acontrast 音频滤镜

+   OpenCL 叠加滤镜

+   video mix 滤镜

+   video normalize 滤镜

+   audio lv2 包装滤镜

+   VAAPI MJPEG 和 VP8 解码

+   AMD AMF H.264 和 HEVC 编码器

+   video fillborders 滤镜

+   video setrange 滤镜

+   支持 LibreSSL（通过 libtls）

+   不再支持构建 Windows XP。最低支持的 Windows 版本是 Windows Vista。

+   deconvolve 视频滤镜

+   entropy 视频滤镜

+   hilbert 音频滤镜源

+   aiir 音频滤镜

+   移除了 ffserver 程序

+   移除了 ffmenc 和 ffmdec 复用器和解复用器

+   VideoToolbox HEVC 编码器和硬件加速

+   VAAPI 加速的 ProcAmp（色彩平衡）、降噪和锐化滤镜

+   添加了 android_camera 输入设备

+   通过 libcodec2 进行 codec2 编解码

+   本地 SBC 编解码器

+   drmeter 音频滤镜

+   hapqa_extract 比特流过滤器

+   filter_units 比特流滤镜

+   通过 libaom 支持 AV1

+   E-AC-3 依赖帧支持

+   用于提取 E-AC-3 核心的比特流滤镜

+   Haivision SRT 协议通过 libsrt

+   vfrdet 滤镜

我们强烈建议用户、分发商和系统集成商升级，除非他们使用当前的 git 主分支。

### 2017 年 10 月 15 日，FFmpeg 3.4 "Cantor"

[FFmpeg 3.4 "Cantor"](download.html#release_3.4)，一个新的重要版本，现已发布！一些亮点包括：

+   去闪烁视频滤镜

+   双编织视频滤镜

+   lumakey 视频滤镜

+   pixscope 视频滤镜

+   示波器视频滤镜

+   更新 cuvid/nvenc 头文件至 Video Codec SDK 8.0.14

+   afir 音频滤镜

+   scale_cuda 基于 CUDA 的视频缩放滤镜

+   librsvg 支持 svg 光栅化

+   crossfeed 音频滤镜

+   MP4 中符合规范的 VP9 混合支持

+   surround 音频滤镜

+   sofalizer 滤镜切换到 libmysofa

+   Gremlin 数字视频分离器和解码器

+   headphone 音频滤镜

+   superequalizer 音频滤镜

+   roberts 视频滤镜

+   Interplay MVE 电影的额外帧格式支持

+   支持通过 D3D11VA 在 ffmpeg 中解码

+   限制器视频滤镜

+   libvmaf 视频滤镜

+   Dolby E 解码器和 SMPTE 337M 分离器

+   反预乘视频滤镜

+   tlut2 视频滤镜

+   泛洪填充视频滤镜

+   伪彩色视频滤镜

+   原生 G.726 复用器和解复用器，左右对齐

+   NewTek NDI 输入/输出设备

+   FITS 分离器和解码器

+   FITS 复用器和编码器

+   抗溢出视频滤镜

+   haas 音频滤镜

+   SUP/PGS 字幕复用器

+   卷积视频滤镜

+   VP9 平铺线程支持

+   KMS 屏幕抓取器

+   CUDA 缩略图过滤器

+   V4L2 mem2mem 硬件辅助编解码器

+   Rockchip MPP 硬件解码

+   vmafmotion 视频滤镜

我们强烈建议用户、分发商和系统集成商升级，除非他们使用当前的 git 主分支。

### 2017 年 4 月 13 日，FFmpeg 3.3 "Hilbert"

[FFmpeg 3.3 "Hilbert"](download.html#release_3.3)，一个新的重要版本，现已发布！一些亮点包括：

+   Apple Pixlet 解码器

+   NewTek SpeedHQ 解码器

+   QDMC 音频解码器

+   PSD（Photoshop 文档）解码器

+   FM 屏幕捕获解码器

+   ScreenPressor 解码器

+   XPM 解码器

+   DNxHR 解码器修复了 HQX 和高分辨率视频

+   ClearVideo 解码器（部分）

+   16.8 和 24.0 浮点 PCM 解码器

+   Intel QSV 加速的 VP8 视频解码

+   本地 Opus 编码器

+   DNxHR 444 和 HQX 编码

+   (M)JPEG 编码器的质量改进

+   VAAPI 加速的 MPEG-2 和 VP8 编码

+   预乘视频滤镜

+   abitscope 多媒体滤镜

+   readeia608 滤波器

+   threshold 滤波器

+   midequalizer 滤波器

+   MPEG-7 视频签名滤镜

+   添加内部 ebur128 库，移除外部 libebur128 依赖

+   Intel QSV 视频缩放和去隔行滤镜

+   Sample Dump eXchange 分离器

+   MIDI Sample Dump Standard 分离器

+   Scenarist 闭式字幕分离器和复用器

+   支持带有多个样本描述表的 MOV

+   Pro-MPEG CoP #3-R2 FEC 协议

+   支持球形视频

+   CrystalHD 解码器迁移到新的解码 API

+   现在如果请求但未找到自动检测库，配置将失败

我们强烈建议用户、分发商和系统集成商升级，除非他们使用当前的 git 主分支。

### 2016 年夏季代码结果

这一刻终于到来了，我们希望能够妥善结束我们在这一轮项目中的参与，这需要时间。有时候只是为了整理每个项目的最终报告，而有时则是在项目结束时仍在进行的工作的最后完成：需要合并最终补丁，稳定待办事项列表，达成未来计划；等等，应有尽有。

毫不拖泥带水，这里是我们在本次编程之夏中力求完成的每个项目的收尾工作的闪亮之处：

#### FFv1（导师：迈克尔·尼德迈尔）

斯坦尼斯拉夫·多尔加诺夫设计并实现了无损 FFV1 编解码器中的运动估计和补偿的实验支持。设计和实现基于雪花视频编解码器，使用了OBMC。斯坦尼斯拉夫的工作证明了通过帧间压缩可以实现显著的压缩增益。FFmpeg 欢迎斯坦尼斯拉夫继续在此概念验证的基础上工作，并将其进展带入IETF的官方FFV1规范中。

#### 自测试覆盖率（导师：迈克尔·尼德迈尔）

彼特鲁·拉雷斯·辛克拉安在FFmpeg中添加了多个自测试，并成功通过了繁琐的过程来微调测试参数，以避免已知且难以避免的问题，比如由于各种平台的舍入误差而导致的校验和不匹配。他的工作显著提高了我们自测试的代码覆盖率。

#### MPEG-4 ALS 编码器实现（导师：蒂洛·博尔格曼）

乌梅尔·汗更新并整合了 ALS 编码器，使其适应当前的 FFmpeg 代码库。他还为 ALS 解码器实现了一个缺失的功能，即启用浮点样本解码。乌梅尔的工作显著改善了 FFmpeg 对 MPEG-4 ALS 的支持。我们欢迎他继续保持其改进，并期待他未来的贡献。

#### Tee 复用器改进（导师：马尔顿·巴林特）

扬·塞贝赫列布斯基的通用目标是改进 Tee 复用器，使其能够容忍阻塞 IO 并允许透明的错误恢复。在设计阶段，发现这种功能需要一个单独的复用器，因此扬在整个夏天都在处理所谓的 FIFO 复用器，逐步修复了代码库中的各种问题。他成功完成了任务，FIFO 复用器现在是主代码库的一部分，还在此过程中进行了其他几项改进。

#### TrueHD 编码器（导师：罗斯蒂斯拉夫·佩利瓦诺夫）

贾伊·卢斯拉的目标是更新libavcodec中已经被弃用且基本被抛弃的MLP（Meridian Lossless Packing）编码器，并改进它以支持TrueHD格式的编码。在资格期间，编码器已更新为可用状态，并在整个夏季成功改进，增加了对多声道音频和TrueHD编码的支持。贾伊的代码现已合并到主代码库中。虽然在LFE通道和32位样本处理方面仍存在一些问题，但正在处理中，以便最终能够专注于改进编码器的速度和效率。

#### 运动插值滤波器（导师：保罗·B·马霍尔）

达温德尔·辛格从现有文献和我们自己的以及迈克尔·尼德迈尔的先前工作中调查了现有的运动估计和插值方法，并基于这些研究实现了滤波器。这些滤波器允许将运动插值帧率转换应用于视频，例如创建慢动作效果或在平滑插值视频的同时改变帧率沿着运动向量。考虑到所有事情，仍然有待完成这些滤波器，这相当困难，但我们对它们的未来持乐观态度。

就是这样。我们对该程序的结果感到满意，并对与如此优秀的一群学生共事的机会深表感谢。我们可能是一个苛刻的群体，但我们的导师在帮助实习生度过他们的旅程中做得非常出色。感谢Google提供这个精彩的项目，以及所有在忙碌生活中抽出时间帮助使GSoC2016成为成功的人们。我们明年见！

### 2016年9月24日，SDL1支持已停止。

由于SDL1自2012年1月起不再维护，并且被SDL2取代，因此不再支持SDL1库。因此，SDL1输出设备已被删除，并替换为SDL2实现。ffplay和opengl输出设备已更新以支持SDL2。

### 2016年8月9日，FFmpeg 3.1.2 "Laplace"

[FFmpeg 3.1.2](download.html#release_3.1)，来自3.1版本分支的新的点版本！修复了几个错误。

我们建议用户、分销商和系统集成商升级，除非他们使用当前的git主分支。

### 2016年7月10日，ffserver程序已停用。

经过深思熟虑，我们宣布我们将在下一个版本中删除项目中的 ffserver 程序。ffserver 由于使用内部 API 而难以维护，这些 API 复杂化了 libavformat 库的最近整理，并阻碍了进一步的清理和改进，这是 API 用户所期望的，并且更易于维护。此外，由于可靠性问题、缺乏知识人员帮助和令人困惑的配置文件语法，该程序对用户部署和运行也很困难。我们邀请当前用户和社区成员编写一个替代程序，使用新的 API 来填补 ffserver 的空缺，并与我们联系，以便我们指导用户测试并为其发展做出贡献。

### 2016 年 7 月 1 日，FFmpeg 3.1.1 "Laplace"

[FFmpeg 3.1.1](download.html#release_3.1)，来自 3.1 发布分支的新点发布现已推出！主要处理了上一个版本引入的几个 ABI 问题。

我们强烈建议用户、分发者和系统集成商，尤其是那些从 3.0 版本升级时遇到问题的用户，升级，除非他们使用当前的 git 主分支。

### 2016 年 6 月 27 日，FFmpeg 3.1 "Laplace"

[FFmpeg 3.1 "Laplace"](download.html#release_3.1)，一个新的主要版本现已发布！一些亮点包括：

+   DXVA2 加速的 HEVC Main10 解码

+   fieldhint 过滤器

+   loop 视频过滤器和 aloop 音频过滤器

+   Bob Weaver 逐行扫描去隔行滤波器

+   firequalizer 过滤器

+   数据范围过滤器

+   bench 和 abench 过滤器

+   ciescope 过滤器

+   协议黑名单 API

+   MediaCodec H264 解码

+   VC-2 HQ RTP 负载格式（草案 v1）解包器和打包器

+   VP9 RTP 负载格式（草案 v2）分组器

+   AudioToolbox 音频解码器

+   AudioToolbox 音频编码器

+   coreimage 过滤器（基于 GPU 的 macOS 图像过滤）

+   移除了 libdcadec

+   用于提取 DTS 核心的比特流过滤器

+   ADPCM IMA DAT4 解码器

+   musx 分离器

+   aix 分离器

+   remap 过滤器

+   hash 和 framehash 复用器

+   colorspace 过滤器

+   hdcd 过滤器

+   readvitc 过滤器

+   VAAPI 加速的格式转换和缩放

+   使用 libnpp/CUDA 加速的格式转换和缩放

+   Duck TrueMotion 2.0 实时解码器

+   宽带单比特数据（WSD）分离器

+   VAAPI 加速的 H.264/HEVC/MJPEG 编码

+   DTS Express (LBR) 解码器

+   通用 OpenMAX IL 编码器，支持树莓派

+   IFF ANIM 分离器和解码器

+   直接流传输（DST）解码器

+   loudnorm 过滤器

+   MTAF 分离器和解码器

+   MagicYUV 解码器

+   OpenExr 改进（瓷砖数据和 B44/B44A 支持）

+   BitJazz SheerVideo 解码器

+   CUDA CUVID H264/HEVC 解码器

+   本地 utvideo 解码器支持 10 位深度

+   移除了 libutvideo 包装器

+   YUY2 无损编解码器解码器

+   VideoToolbox H.264 编码器

我们强烈建议用户、分发者和系统集成商升级，除非他们使用当前的 git 主分支。

### 2016 年 3 月 16 日，Google 夏季编程活动

FFmpeg 已被接受为[Google夏季编程活动](https://summerofcode.withgoogle.com/)的开源组织。如果您希望作为学生参与，请参阅我们的[项目想法页面](https://trac.ffmpeg.org/wiki/SponsoringPrograms/GSoC/2016)。您现在可以与导师联系，开始完成资格任务，并在谷歌注册并提交您的项目提案草案。祝您好运！

### 2016年2月15日，FFmpeg 3.0 "爱因斯坦"

[FFmpeg 3.0 "爱因斯坦"](download.html#release_3.0)，一个新的主要版本，现已推出！一些亮点包括：

我们强烈建议用户、分销商和系统集成商升级，除非他们正在使用当前的 git 主分支。

### 2016年1月30日，移除对两个外部AAC编码器的支持

我们刚刚在FFmpeg主分支中移除了对VisualOn AAC编码器（libvo-aacenc）和libaacplus的支持。

即使在将我们的内部AAC编码器标记为[稳定](#aac_encoder_stable)之前，已经知道libvo-aacenc在大多数样本中与我们的本机编码器相比质量较低。然而，VisualOn编码器广泛被Android开源项目使用，我们希望在我们的代码库中有一个经过测试和验证的稳定选项。

当2011年首次提交时，libaacplus填补了高效AAC格式（HE-AAC和HE-AACv2）的编码空白，这些格式当时FFmpeg的编码器都不支持。

现在，情况已经改变。在罗斯蒂斯拉夫·佩希利万诺夫和克劳迪奥·弗雷雷的主导下，现在稳定的FFmpeg本机AAC编码器已经准备好与更成熟的编码器竞争。弗劳恩霍夫FDK AAC编解码器库于2012年被添加为第四个支持的外部AAC编码器，并且具有最佳的质量和支持最多功能，包括HE-AAC和HE-AACv2。

因此，我们决定是时候移除 libvo-aacenc 和 libaacplus 了。如果您目前正在使用 libvo-aacenc，请准备在更新到下一个版本的 FFmpeg 时过渡到本机编码器（`aac`）。在大多数情况下，只需简单地更改编码器名称即可。如果您目前正在使用 libaacplus，请开始使用 FDK AAC（`libfdk_aac`），并选择适当的`profile`选项以选择符合您需求的精确AAC配置文件。在这两种情况下，您将享受到听觉质量的提升，以及更少的许可证问题。

祝您好运！

### 2016年1月16日，FFmpeg 2.8.5、2.7.5、2.6.7、2.5.10

我们发布了几个新的点版本（**[2.8.5](download.html#release_2.8), [2.7.5](download.html#release_2.7), [2.6.7](download.html#release_2.6), [2.5.10](download.html#release_2.5)**）。它们修复了各种错误，包括 CVE-2016-1897 和 CVE-2016-1898。请查看每个版本的变更日志以获取更多详细信息。

我们建议用户、分销商和系统集成商升级，除非他们正在使用当前的 git 主分支。

### 2015年12月5日，本机FFmpeg AAC编码器现已稳定！

暂后不再使用的FFmpeg AAC编码器在七年后已被去掉了实验性标志，被宣告已准备好供大众使用。该编码器在大多数测试样品中的128kbps中透明，仅在极端情况下会出现副作用。旨在主观质量测试中，编码器与公众可获取的其他编码器相比，验证其质量至少相等或更优。

编码AAC音频的许可一直是一个问题，因为大多数编码器都有许可证，使得FFmpeg如果使用它们的支持向公众重新分发则无法自由分发。现在项目中存在一个完全开放和真正免费的AAC编码器，这对于希望使用被广泛接受的标准的人们意义重大。

在此期间，编码器质量提升的大部分工作由开发者Claudio Freire和Rostislav Pehlivanov在今年的GSoC期间开始并持续进行。后者在其加入时加入了作为开发者和主要维护者行列，并且也在执行项目中的其他部分。此外，感谢[Kamedo2](http://d.hatena.ne.jp/kamedo2/)所作的比较和测试，原始编码器的作者以及所有过去的和当前的编码器贡献者。用户被建议并鼓励使用编码器并向我们[错误追踪器](https://trac.ffmpeg.org/)提供反馈或故障报告。

在此向我们最新支持者：MediaHub和Telepoint表示感谢。这两家公司捐赠了一台专门的服务器，提供了免费的互联网连接。以下为他们自己的陈述：

+   Telepoint（http://www.telepoint.bg/en/）是保加利亚最大的中立数据中心。坐落在索菲亚的中心地带，网络流量主轴的道路交汇处，该设施是一家提供全面IT服务的三级数据中心，提供灵活的定制级客户托管解决方案（从一台服务器到私人托管室），并提供高标准的安全保障。

+   MediaHub Ltd.是一家保加利亚的IPTV平台和服务提供商，自运营以来大量使用FFmpeg。她们表示："捐赠以资助FFmpeg的运营，是我们向社区回馈的一种方式"。

感谢Telepoint和MediaHub的鼎力支持！

### 2015年9月29日，2015年GSoC结果

FFmpeg参与了最新的[Google Summer of Code](http://www.google-melange.com/gsoc/homepage/google/gsoc2015)项目计划。FFmpeg共分配了8个项目，其中有7个成功完成。

我们要感谢[Google](https://www.google.com)、参与的学生们，特别是加入这一努力的所有导师。我们期待着参与下一届GSoC计划！

以下是每个单独项目最终成果的简要描述。

#### 网络协议的基本服务器，实习者：Stephan Holljes，导师：Nicolas George

Stephan Holljes在此Google Summer of Code会话中的项目是为libavformat实现基本的HTTP服务器功能，以补充已有的HTTP客户端和RTMP及RTSP服务器代码。

项目的第一部分是使HTTP代码能够接受单个客户端；这在资格期间的一部分和暑期第一周的一部分完成。由于这项工作，现在可以使用以下命令创建简单的HTTP流：

```
    ffmpeg -i /dev/video0 -listen 1 -f matroska \
    -c:v libx264 -preset fast -tune zerolatency http://:8080
    ffplay http://localhost:8080/

```

项目的下一部分是扩展代码，使其能够同时或依次接受多个客户端。由于libavformat没有这种任务类型的API，因此需要设计一个。这部分在期中考试前大部分完成，并在之后不久应用。由于ffmpeg命令行工具尚未准备好为多个客户端提供服务，因此测试新API的测试程序是提供硬编码内容的示例程序。

项目的最后和最雄心勃勃的部分是更新ffserver以利用新API。这将证明该API可用于实现真正的HTTP服务器，并暴露需要更多控制的地方。夏季结束时，第一个可工作的补丁系列正在进行代码审查。

#### 在服务器上浏览内容，受训者：Mariusz Szczepańczyk，导师：Lukasz Marek

Mariusz完成了FFmpeg社区准备的API，并实现了作为资格任务的Samba目录列表。

在节目期间，他扩展了API，使其能够在远程服务器上删除和重命名文件。他完成了这些功能的实现，适用于文件、Samba、SFTP和FTP协议。

在项目结束时，Mariusz提供了一个HTTP目录监听的草图实现。

#### Directshow数字视频捕获，受训者：Mate Sebok，导师：Roger Pack

Mate正在从数字视频源获取directshow输入。他已从ATSC输入源获得工作输入，可以指定调谐器。

代码尚未提交，但其中的一个补丁已发送到ffmpeg-devel邮件列表，供将来使用。

导师计划清理并提交代码，至少用于ATSC方面的事务。Mate和导师仍在努力最终找出如何使DVB工作。

#### 实现对3GPP定时文本字幕的全面支持，受训者：Niklesh Lalwani，导师：Philip Langdale

Niklesh的项目是扩展我们对3GPP定时文本字幕的支持。这是mp4容器的本地字幕格式，因为通常是iOS和Android设备上的原生播放应用唯一支持的字幕格式，因此非常有趣。

ffmpeg已经基本支持这些字幕，忽略了所有格式信息 - 它只提供基本的纯文本支持。

Niklesh进行了工作，为文本格式化功能（如字体大小/颜色和加粗/斜体、高亮等效果）的编码和解码两方面添加了支持。

这里的主要挑战是，定时文本处理格式与大多数常见字幕格式非常不同。它使用二进制编码（自然基于mp4盒），并且将信息存储在文本本身之外。这需要额外的工作来跟踪文本格式应用的部分，并明确处理重叠的格式（其他格式支持但定时文本不支持），因此需要将重叠部分分解为具有不同格式的非重叠部分。

最后，Niklesh必须小心，不要相信字幕中的任何大小信息 - 这可不是开玩笑：现在臭名昭著的Android舞台恶魔漏洞就存在于解析定时文本字幕的代码中。

Niklesh的所有工作都已经提交，并且在ffmpeg 2.8中发布。

#### libswscale重构，实习生：Pedro Arthur，导师：Michael Niedermayer，Ramiro Polla

Pedro Arthur已将垂直和水平缩放器模块化。为此，他设计并实现了一个通用的滤波器框架，并将现有的缩放器代码移入其中。这些改变现在允许轻松添加、移除、分割或合并处理步骤。该实现已进行基准测试，并尝试了多种替代方法以避免速度损失。

他还添加了伽马校正的缩放支持。使用伽马校正的缩放的一个示例是：

```
    ffmpeg -i input -vf scale=512:384:gamma=1 output

```

Pedro在有限的时间内做了令人印象深刻的工作，现在是一个FFmpeg的提交者。他继续为FFmpeg做贡献，并在GSoC结束后修复了libswscale中的一些错误。

#### AAC编码器改进，实习生：Rostislav Pehlivanov，导师：Claudio Freire

Rostislav Pehlivanov在本地AAC编码器上实现了PNS、TNS、I/S编码和主要预测。所有这些扩展中，只有TNS仍处于不太可用的状态，但由于它为进一步改进提供了良好的基础，实现已经推进（已禁用）。

PNS用单一的标度因子替换带有噪声的频段，极大提高了编码效率，而在低比特率下的质量改进令人印象深刻，对于这样一个简单的功能来说。

TNS仍需要一些润色，但它有潜力通过在时间域中应用噪声整形来减少编码伪影（这是低熵频段上令人恼人的明显失真的来源）。

强度立体声编码（I/S）可以通过利用立体声通道之间的强相关性来使编码效率翻倍，对于使用了平移混音的流行风格曲目效果最佳。然而，在经典的X-Y录音上，这种技术效果不佳。

最后，主要预测通过利用连续帧之间的相关性来提高编码效率。虽然目前收益并不巨大，但Rostislav在GSoC结束后仍然活跃，并且正在优化TNS和主要预测，以及寻找进一步的改进。

在此过程中，编码器的MIPS端口有时会出现问题，他也在努力解决这个问题。

#### 动画可便携网络图形（APNG），学员：Donny Yang，导师：Paul B Mahol。

Donny Yang实现了基本的关键帧APNG编码器作为资格任务。后来他通过各种混合模式实现了帧间压缩。当前的实现尝试所有混合模式并选择占用内存最少的模式。

特别注意确保解码器能正确播放所有在野外找到的文件，并且编码器生成的文件可以在支持APNG的浏览器中播放。

在他的工作期间，他被指定修复解码器中遇到的任何bug，因为它不符合APNG规范。由于这项工作，PNG解码器中的长期存在的bug已经修复。

对于后续的工作，他计划继续在编码器上工作，使得可以选择在编码过程中使用哪些混合模式。这可以加快APNG文件的编码速度。

### 2015年9月9日，FFmpeg 2.8。

我们发布了新的主要版本**[2.8](download.html#release_2.8)**。它包含了从9月8日git主分支的所有功能和错误修复。请查看**[更新日志](https://raw.githubusercontent.com/FFmpeg/FFmpeg/release/2.8/Changelog)**以获取最重要的更改列表。

我们建议用户、分发商和系统集成商升级，除非他们使用当前的git主分支。

### 2015年8月1日，来自FFmpeg项目的一封信。

亲爱的多媒体社区，请注意

昨天，Michael Niedermayer辞去了FFmpeg项目领导人职务，这令人意外。他多年来在FFmpeg项目上不知疲倦地工作，我们必须感谢他所做的工作。我们希望他将来能继续为项目做贡献。在未来几周内，FFmpeg项目将由活跃的贡献者管理。

过去四年对我们的多媒体社区来说并不容易——包括贡献者和用户。现在我们应该展望未来，努力找到这些问题的解决方案，并在分裂社区长期以来进行和解。

不幸的是，到目前为止，许多分歧发生在不合适的场合，这使得找到共同点和解决方案变得困难。我们打算在接下来的几周内在我们的在线社区中讨论这个问题，并且在9月的[VideoLAN开发者日](https://www.videolan.org/videolan/events/vdd15/)上，我们将在巴黎进行面对面的讨论：这是整个开源多媒体社区的中立场所。

[FFmpeg项目](http://www.ffmpeg.org)。

### 2015年7月4日，FFmpeg需要一个新的主机。

**更新：** 我们收到了超过7个提供主机和服务器的提议，非常感谢大家！

在慷慨地为我们的项目（[FFmpeg](http://www.ffmpeg.org)、[MPlayer](http://www.mplayerhq.hu)和[rtmpdump](http://rtmpdump.mplayerhq.hu)）提供了4年后，我们的主办方Arpi通知我们，我们必须立即在其他地方找到新的主机。

如果你想托管一个开源项目，请在[ffmpeg-devel](http://ffmpeg.org/mailman/listinfo/ffmpeg-devel)邮件列表或irc.freenode.net #ffmpeg-devel上告诉我们。

我们使用约4TB的存储和至少4TB的带宽/每月用于各种邮件列表，[trac](http://trac.ffmpeg.org)，[样本库](http://samples.ffmpeg.org)，svn等。

### 2015年3月16日，FFmpeg 2.6.1。

我们发布了一个新的主要版本（**[2.6](download.html#release_2.6)**），一周后发布了2.6.1。它包含从3月6日的git master分支的所有特性和错误修复。请参见**[发行说明](http://git.videolan.org/?p=ffmpeg.git;a=blob;f=RELEASE_NOTES;hb=release/2.6)**，查看值得注意的更改列表。

我们建议用户、分发者和系统集成商升级，除非他们使用当前的git master。

### 2015年3月4日，Google Summer of Code。

FFmpeg已被接受为[Google Summer of Code](http://www.google-melange.com/gsoc/homepage/google/gsoc2015)项目。如果你希望作为学生参与，请查看我们的[项目想法页面](https://trac.ffmpeg.org/wiki/SponsoringPrograms/GSoC/2015)。你可以联系导师并开始从事资格任务。学生在Google上的注册将于3月16日开启。祝你好运！

### 2015年3月1日，Chemnitzer Linux-Tage。

我们高兴地宣布，FFmpeg将在德国Chemnitz的Chemnitzer Linux-Tage（CLT）上展示。活动将于3月21日和22日举行。

更多信息可以在[这里](https://chemnitzer.linux-tage.de/2015/en/)找到。

我们演示了FFmpeg的使用，回答你的问题，并倾听你的问题和愿望。**如果你有无法正确处理的媒体文件，请确保带上样本，以便我们查看！**

在我们的CLT历史上，首次将举办**FFmpeg研讨会**！你可以在[这里](https://chemnitzer.linux-tage.de/2015/de/programm/beitrag/209)阅读详细信息。研讨会面向FFmpeg初学者，首先将介绍多媒体基础知识，之后你将学习如何利用这些知识和FFmpeg的CLI工具分析和处理媒体文件。研讨会仅用德语进行，需提前注册。研讨会将于周六上午10点开始。

我们期待再次见到你！

### 2014年12月5日，FFmpeg 2.5。

我们发布了一个新的主要版本（**[2.5](download.html#release_2.5)**），包含从12月4日的git master分支的所有特性和错误修复。请参见**[发行说明](http://git.videolan.org/?p=ffmpeg.git;a=blob;f=RELEASE_NOTES;hb=release/2.5)**，查看值得注意的更改列表。

我们建议用户、分发者和系统集成商升级，除非他们使用当前的git master。

### 2014年10月10日，FFmpeg再次在Debian不稳定版中。

我们想让您知道，在Debian不稳定版中再次有[FFmpeg软件包](https://packages.debian.org/search?keywords=ffmpeg&searchon=sourcenames&suite=unstable&section=main)。**特别感谢安德烈亚斯·卡达尔普和所有使这一切成为可能的人。**这一切都绝非易事。

不幸的是，这已经是这则消息中较为简单的部分了。坏消息是软件包可能不会迁移到Debian测试版以及即将发布的代号为Jessie的版本。请[在Debian上阅读讨论](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=763148)。

**无论最终如何，我们都希望您继续给予非凡的支持！**

### 2014年10月8日，FFmpeg确保了在OPW中的一个名额！

多亏三星（开源组）慷慨捐赠了6000美元，FFmpeg将至少欢迎1名“女性外展计划”实习生从2014年12月至2015年3月与我们的社区合作。

我们都知道FFmpeg被行业广泛使用，但尽管有无数产品建立在我们的代码基础之上，但在需要帮助时，公司很少会站出来支持我们。因此，特别感谢三星和OPW计划委员会！

如果您考虑作为实习生参与OPW，请查看我们的[OPW维基页面](https://trac.ffmpeg.org/wiki/SponsoringPrograms/OPW/2014-12)，获取一些初步指南。该页面仍在不断完善中，但应该有足够的信息帮助您入门。另一方面，如果您考虑通过OPW计划赞助FFmpeg的工作，请通过opw@ffmpeg.org与我们联系。在您的帮助下，我们可能能够为本轮增加一些额外的实习名额！

### 2014年9月15日，FFmpeg 2.4发布。

我们发布了一个新的主要版本（**[2.4](download.html#release_2.4)**）。它包含了从2014年9月14日开始的git主分支的所有功能和错误修复。请查看**[发布说明](http://git.videolan.org/?p=ffmpeg.git;a=blob;f=RELEASE_NOTES;hb=release/2.4)**，了解重要变更列表。

我们建议用户、分销商和系统集成商升级，除非他们使用当前的git主分支。

### 2014年8月20日，FFmpeg 2.3.3、2.2.7、1.2.8发布。

我们发布了几个新的小版本（**[2.3.3](download.html#release_2.3), [2.2.7](download.html#release_2.2), [1.2.8](download.html#release_1.2)**）。它们修复了各种错误，包括CVE-2014-5271和CVE-2014-5272。请查看更新日志获取更多详情。

我们建议用户、分销商和系统集成商升级，除非他们使用当前的git主分支。

### 2014年7月29日，帮助我们确保在OPW中的位置。

根据我们之前的帖子关于我们参与今年的OPW（女性外展计划）的内容，我们现在正在联系我们的用户（个人和公司），帮助我们筹集所需的资金来确保我们在该计划中的位置。

我们需要至少筹集6000美元，但如果能获得更多资金，将有助于我们招募更多实习生。

您可以通过[Click&Pledge](https://co.clickandpledge.com/advanced/default.aspx?wid=56226)使用信用卡捐款并选择"OPW"选项。如果您想通过汇款或支票捐款，请通过[e-mail](mailto:opw@ffmpeg.org)联系我们，我们将回复并提供说明。

谢谢！

### 2014 年 7 月 20 日，全新网站

FFmpeg 项目自豪地宣布由[db0](http://db0.fr)制作的全新网站版本。尽管最初是出于需要更大的菜单，但整个网站最终都进行了重新设计，并且大多数页面都进行了重新工作以方便导航。我们希望您能够愉快地浏览它。

### 2014 年 7 月 17 日，FFmpeg 2.3

我们发布了一个新的主要版本（**[2.3](download.html#release_2.3)**）。它包含了 7 月 16 日 git master 分支的所有功能和 bug 修复。请参阅**[发布说明](http://git.videolan.org/?p=ffmpeg.git;a=blob;f=RELEASE_NOTES;hb=489d066)**以了解值得注意的变化列表。

我们建议用户、分发商和系统集成商升级，除非他们使用当前的 git master。

### 2014 年 7 月 3 日，FFmpeg 和妇女才能扩展项目

FFmpeg 已经开始成为下一轮项目的 OPW 包含组织，并且实习将于12月9日开始。[OPW](https://gnome.org/opw/)的目标是"帮助妇女（原生和跨性别）和性别酷儿参与自由开源软件"。该过程的一部分需要筹集资金以支持至少一个实习生（6K 美元），因此，如果您一直对您对 FFmpeg 的捐款持观望态度，那么这是一个非常好的机会，希望您能前来联系并帮助该项目和这个伟大的倡议！

我们已设立一个[电子邮件地址](mailto:opw@ffmpeg.org)，您可用于与我们联系有关捐赠和我们参与该项目的一般咨询。希望能尽快收到您的来信！

### 2014 年 6 月 29 日，FFmpeg 2.2.4, 2.1.5, 2.0.5, 1.2.7, 1.1.12, 0.10.14

我们发布了几个新的点版本（**[2.2.4](download.html#release_2.2), [2.1.5](download.html#release_2.1), [2.0.5](download.html#release_2.0), [1.2.7](download.html#release_1.2), [1.1.12](download.html#release_1.1), [0.10.14](download.html#release_0.10)**）。它们修复了[LZO实现中的安全问题](http://blog.securitymouse.com/2014/06/raising-lazarus-20-year-old-bug-that.html)，以及其他一些 bug。详情请参见 git 日志。

我们建议用户、分发商和系统集成商升级，除非他们使用当前的 git master。

### 2014 年 5 月 1 日，LinuxTag

FFmpeg 将再次在德国柏林的 LinuxTag 上展示。该活动将于 5 月 8 日至 10 日举行。请注意，今年的 LinuxTag 将在离市中心更近的不同地点举行。

我们将与 XBMC 和 VideoLAN 共享展位。**如果您有媒体文件无法被 FFmpeg 正确处理，请务必携带样本让我们来查看！**

更多关于 LinuxTag 的信息可以在[这里](http://www.linuxtag.org/2014/)找到

我们期待在柏林见到您！

### 2014 年 4 月 18 日，OpenSSL Heartbeat 漏洞

我们托管 Trac 问题跟踪器的服务器对 "heartbleed" 攻击存在漏洞。OpenSSL 软件库在 4 月 7 日漏洞公开披露后不久进行了更新。我们已更改了所有 FFmpeg 服务器的私钥（和证书）。项目服务器团队成员 Alexander Strasser 已通过邮件列表发送了详细信息。这里是用户邮件列表的[存档链接](https://lists.ffmpeg.org/pipermail/ffmpeg-user/2014-April/020968.html)。

我们鼓励您了解["OpenSSL heartbleed"](https://www.schneier.com/blog/archives/2014/04/heartbleed.html)。**有可能登录问题跟踪器的数据已被利用这个安全漏洞的人暴露。您可能希望在问题跟踪器和其他地方使用了相同密码的地方更改您的密码。**

### 2014 年 4 月 11 日，FFmpeg 2.2.1

我们发布了一个新的点版本（**[2.2.1](download.html#release_2.2)**）。它包含了对 Tickets #2893, #3432, #3469, #3486, #3495 和 #3540 的错误修复，以及其他几个修复。详细信息请参阅 git 日志。

### 2014 年 3 月 24 日，FFmpeg 2.2

我们发布了一个新的主要版本（**[2.2](download.html#release_2.2)**）。它包含自 3 月 1 日起 git 主分支的所有功能和错误修复。以下是一些新内容的部分列表：

```
    - HNM version 4 demuxer and video decoder
    - Live HDS muxer
    - setsar/setdar filters now support variables in ratio expressions
    - elbg filter
    - string validation in ffprobe
    - support for decoding through VDPAU in ffmpeg (the -hwaccel option)
    - complete Voxware MetaSound decoder
    - remove mp3_header_compress bitstream filter
    - Windows resource files for shared libraries
    - aeval filter
    - stereoscopic 3d metadata handling
    - WebP encoding via libwebp
    - ATRAC3+ decoder
    - VP8 in Ogg demuxing
    - side & metadata support in NUT
    - framepack filter
    - XYZ12 rawvideo support in NUT
    - Exif metadata support in WebP decoder
    - OpenGL device
    - Use metadata_header_padding to control padding in ID3 tags (currently used in
    MP3, AIFF, and OMA files), FLAC header, and the AVI "junk" block.
    - Mirillis FIC video decoder
    - Support DNx444
    - libx265 encoder
    - dejudder filter
    - Autodetect VDA like all other hardware accelerations

```

我们建议用户、发行商和系统集成商升级，除非他们使用当前的 git 主分支。

### 2014 年 2 月 3 日，Chemnitzer Linux-Tage

我们高兴地宣布，FFmpeg 将在德国Chemnitz举办的“Chemnitzer Linux-Tage”上亮相。活动将于 3 月 15 日和 16 日举行。

更多信息可以在[这里](http://chemnitzer.linux-tage.de/2014/en/info/)找到。

我们邀请您访问我们位于 Linux-Live 区的展位！在那里，我们将演示如何使用 FFmpeg，解答您的问题，并倾听您的问题和建议。

**如果您有无法用 FFmpeg 正确处理的媒体文件，请务必带上样本让我们查看！**

我们期待与您见面（再次见面）！

### 2014 年 2 月 9 日，trac.ffmpeg.org / trac.mplayerhq.hu 安全漏洞

FFmpeg 和 MPlayer 的 Trac 问题跟踪器安装在的服务器遭到了 compromise。受影响的服务器已被下线并更换，所有软件已重新安装。FFmpeg Git、发布版、FATE、网站和邮件列表位于其他服务器上，并未受到影响。我们认为最初的 compromise 发生在几个月前与 FFmpeg 和 MPlayer 无关的一台服务器上。该服务器被用作克隆最近将 Trac 迁移到的虚拟机的源。目前不清楚是否有人利用了发现的后门。

我们建议所有用户更改他们的密码。**特别是那些在 Trac 上使用与其他地方相同密码的用户，至少应该在其他地方更改密码。**

### 2013 年 11 月 12 日，Debian 中的 FFmpeg RFP

自Libav分离以来，Debian/Ubuntu的维护者一直跟随Libav分支。许多人已请求在Debian中打包ffmpeg，因为它更加功能完整，在许多情况下也更少出现错误。

[罗杰里奥·布里托](http://cynic.cc/blog/)，一位Debian开发者，在Debian的Bug跟踪系统中提出了一个软件包请求（RFP）。

请让Debian和Ubuntu开发者知道您支持打包真正的FFmpeg！详细信息请参见Debian的[ticket #729203](http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=729203)。

### 2013年10月28日，FFmpeg 2.1

我们已经发布了一个新的重大版本（**[2.1](download.html#release_2.1)**），包含了自10月28日以来git主分支的所有功能和错误修复。以下是一部分新功能的列表：

```
    - aecho filter
    - perspective filter ported from libmpcodecs
    - ffprobe -show_programs option
    - compand filter
    - RTMP seek support
    - when transcoding with ffmpeg (i.e. not streamcopying), -ss is now accurate
    even when used as an input option. Previous behavior can be restored with
    the -noaccurate_seek option.
    - ffmpeg -t option can now be used for inputs, to limit the duration of
    data read from an input file
    - incomplete Voxware MetaSound decoder
    - read EXIF metadata from JPEG
    - DVB teletext decoder
    - phase filter ported from libmpcodecs
    - w3fdif filter
    - Opus support in Matroska
    - FFV1 version 1.3 is stable and no longer experimental
    - FFV1: YUVA(444,422,420) 9, 10 and 16 bit support
    - changed DTS stream id in lavf mpeg ps muxer from 0x8a to 0x88, to be
    more consistent with other muxers.
    - adelay filter
    - pullup filter ported from libmpcodecs
    - ffprobe -read_intervals option
    - Lossless and alpha support for WebP decoder
    - Error Resilient AAC syntax (ER AAC LC) decoding
    - Low Delay AAC (ER AAC LD) decoding
    - mux chapters in ASF files
    - SFTP protocol (via libssh)
    - libx264: add ability to encode in YUVJ422P and YUVJ444P
    - Fraps: use BT.709 colorspace by default for yuv, as reference fraps decoder does
    - make decoding alpha optional for prores, ffv1 and vp6 by setting
    the skip_alpha flag.
    - ladspa wrapper filter
    - native VP9 decoder
    - dpx parser
    - max_error_rate parameter in ffmpeg
    - PulseAudio output device
    - ReplayGain scanner
    - Enhanced Low Delay AAC (ER AAC ELD) decoding (no LD SBR support)
    - Linux framebuffer output device
    - HEVC decoder, raw HEVC demuxer, HEVC demuxing in TS, Matroska and MP4
    - mergeplanes filter

```

我们建议用户、分销商和系统集成商升级，除非他们正在使用当前的git主分支。*****
