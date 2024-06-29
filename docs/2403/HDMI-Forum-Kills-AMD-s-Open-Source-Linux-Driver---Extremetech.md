<!--yml

category: 未分类

date: 2024-05-27 14:34:57

-->

# HDMI论坛终止了AMD的开源Linux驱动 | Extremetech

> 来源：[https://www.extremetech.com/computing/hdmi-forum-kills-amds-open-source-linux-driver](https://www.extremetech.com/computing/hdmi-forum-kills-amds-open-source-linux-driver)

HDMI标准自推出20多年来已经无处不在。它如此普及，以至于人们很容易忘记它是一个专有标准——这对于自由开源软件的爱好者来说可能是个问题。AMD一直希望向[Linux信徒](https://www.extremetech.com/computing/developer-touts-the-benefits-of-diagonal-mode-linux-desktop)赠送开源HDMI驱动程序，但这一努力似乎即将告终。据报道，HDMI论坛拒绝了AMD的法律理由，迫使他们继续保密驱动程序。

HDMI论坛由索尼、松下和Maxell等公司于2002年成立，目前拥有近100名成员。论坛的职责是监督HDMI标准的开发和许可，并不允许技术的开放访问。这一点一直是最苛刻的Linux桌面用户的眼中钉。如果你想在HDMI上驱动4K或5K显示器以高刷新率，那么你没门了。

作为论坛的长期成员，AMD自去年以来一直在开发[ HDMI 2.1规范](https://www.extremetech.com/extreme/329749-hdmi-2-1-devices-are-not-required-to-support-any-new-hdmi-2-1-features) 的开源版本。那时，AMD的开发人员Alex Deucher宣布，AMD的法律团队正在努力确定公司可以在不违反论坛义务的情况下提供哪些功能。有了这样的驱动程序，AMD的开放FreeSync标准可以通过HDMI实现高分辨率和刷新率。

截至2023年底，AMD已向HDMI论坛提交了请求，并正在等待决定。Deucher在他保持开放三年的问题跟踪器线程上[发布了更新](https://gitlab.freedesktop.org/drm/amd/-/issues/1417#note_2303163)，情况不妙。论坛拒绝了AMD的请求，Deucher认为没有办法发布开源HDMI 2.1驱动程序而不让AMD陷入困境。该帖子吸引了数十位失望的Linux用户的新回复，他们谴责论坛的固执。

如果发布该驱动程序会违反什么规定，目前还不清楚。根据Ars Technica的报道，论坛成员每年支付15,000美元的会费，但所有公开的文件中严格禁止创建规范的开源版本。然而，HDMI论坛的源代码许可非常限制性。一些评论员在问题跟踪器中推测，论坛可能担心盗取受保护的视频流。然而，数字盗版分子几乎不需要开源HDMI驱动程序来盗取视频内容。

这可能是对于 Linux 用户而言 HDMI 的丧钟之钉。DisplayPort 是一种新标准，在 Linux 上具有更好的输出支持。AMD 可以在闭源驱动程序或二进制 blob 中实现新的 HDMI 特性，但 Deucher 并未说明项目将如何推进或是否会推进。
