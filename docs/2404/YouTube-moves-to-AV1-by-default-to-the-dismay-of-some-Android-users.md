<!--yml

类别：未分类

日期：2024-05-27 13:19:48

-->

# YouTube默认采用AV1编码器令部分Android用户感到不满

> 来源：[https://www.androidpolice.com/youtube-google-av1-codec-android-video/](https://www.androidpolice.com/youtube-google-av1-codec-android-video/)

### 摘要

+   随着4K视频的兴起，AV1编解码器在Android设备上因其更好的质量正变得流行起来。

+   Google已向许多Android设备推出了新的AV1解码器支持，但应用必须明确声明他们希望使用这一新功能。

+   YouTube选择默认使用新的解码器，一些用户对于缺乏硬件加速支持可能带来的电池影响感到担忧。

随着视频技术的不断发展，处理其复杂性的需求也在增长。例如，随着4K视频的出现，开发人员迅速意识到现有的视频编解码器如H264已经不再适用。因此，AV1开始成为设备上解码和编码的流行标准。现在，Google准备适应时代的变化，很快这将在[YouTube](https://www.androidpolice.com/youtube-1080p-enhanced-bitrate-web-rollout/)上得以体现。

[](/youtube-1080p-enhanced-bitrate-web-rollout/)相关的Android设备所有者将不得不再等待一段时间

[Arif Dikici](https://www.linkedin.com/feed/update/urn:li:activity:7186235577493544960/)，他是Google Android视频和图像编解码团队的一员，最近在LinkedIn上[宣布](https://www.linkedin.com/feed/update/urn:li:activity:7186235577493544960)，Android现在将使用一个名为“libdav1d”的AV1解码器，这是由VLC背后的团队创建的。如规定，一旦接收到2024年3月的[Google Play系统更新](https://www.androidpolice.com/check-for-google-play-system-updates-manually-again/)，所有从Android 12开始的设备将具备该编解码器的本地软件支持。

## YouTube默认使用AV1编码器给推出增加了难度

虽然此更新将为大量Android手机带来改进的AV1编解码器支持，但不会默认激活新编解码器 —— 相反，会继续使用现有的AV1解码器libgav1，除非特定应用明确要求使用更新的dav1d解码器来处理视频。这正是YouTube在此次推出后立即开始采用的做法，引发了部分Android用户的争议。

视频编解码器的一个关键优势是在压缩和解码过程中帮助保持视频质量，因此许多用户对更高质量的AV1视频潜力感到兴奋。然而，并非所有人都对这一变化感到满意，特别是那些使用较老设备和中端手机的用户。目前，只有较新的高端手机支持AV1的硬件解码，因此那些使用中端设备和较旧旗舰机型的用户现在可能不得不依赖软件解码，因为YouTube选择采用了libdav1d解码器。许多这些安卓用户[表达了关注](http://www.reddit.com/r/Android/comments/1c7i24z/youtube_is_now_forcing_av1_on_every_device/)，担心这一变化是否会影响其电池寿命，因为这可能意味着远离硬件加速解码。

尽管有理由担忧，但值得注意的是，谷歌并非第一个在像YouTube这样广泛使用的服务中引入这种变化的公司。早在2020年，Netflix就为其安卓应用用户做了类似的事情，[将AV1作为其新的标准](https://www.androidpolice.com/2020/02/05/netflix-for-android-adopts-av1-video-codec-currently-limited-to-save-data-mode/)。在这一变化之前，Netflix确认其安卓应用一直在使用VP6编解码器。随着升级，Netflix开始允许订阅者流式传输内容，并启用“节省数据”功能，同时利用移动服务。与安卓用户目前所说的情况相比，似乎没有像谷歌最近的新闻那样引起如此大的骚动 — 或许时间会缓解随谷歌最近消息而产生的担忧。

*感谢*: **阿曼多**
