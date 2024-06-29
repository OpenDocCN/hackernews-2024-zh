<!--yml

category: 未分类

date: 2024-05-27 14:55:20

-->

# [Freenginx: Nginx 核心开发者宣布分叉流行的 Web 服务器 - InfoQ](https://www.infoq.com/news/2024/03/freenginx-ngnix-web-server/)

> 来源：[https://www.infoq.com/news/2024/03/freenginx-ngnix-web-server/](https://www.infoq.com/news/2024/03/freenginx-ngnix-web-server/)

最近，一位曾是 F5 员工并且是 Nginx 项目的主要贡献者宣布了流行的 Web 服务器的分叉 [Freenginx](http://freenginx.org/)。新项目旨在 [解决安全争议](https://mailman.nginx.org/pipermail/nginx-devel/2024-February/K5IC6VYO2PB7N4HRP2FUQIBIBCGP4WAU.html) 并且希望成为 Nginx 的替代品，由开发者而非企业实体运营。曾在 F5 担任首席软件工程师的 [Maxim Dounin](http://mdounin.ru/) 为分叉提供了深入见解：

> 不幸的是，F5 的一些新非技术管理人员最近决定他们更懂如何运作开源项目。特别是，他们决定干涉 Nginx 多年来使用的安全策略，无视该策略和开发者的立场。

最初由 [Igor Sysoev](http://sysoev.ru/en/) 开发，目前由 F5 维护，Nginx 是用于 Web 服务、反向代理、缓存、负载平衡和媒体流的开源软件。根据 [Web Server Survey](https://www.netcraft.com/blog/january-2024-web-server-survey/)，即使在其首次发布两十年后，Nginx 仍然是领先的 Web 服务器，占据了所有站点的 23.21%。在 Hacker News 的一个流行 [帖子](https://news.ycombinator.com/item?id=39373327) 中，用户 *sevg* [指出](https://news.ycombinator.com/item?id=39373804)：

> 值得注意的是，目前只有两名“核心”开发者，Maxim Dounin（OP）和 Roman Arutyunyan。Maxim 是仍在活跃的最大贡献者。Maxim 和 Roman 实际上占了目前开发工作的 99%。因此，这是一个相当有影响力的分叉。

在他在 [nginx-devel 邮件列表](https://mailman.nginx.org/mailman/listinfo/nginx-devel) 上的公告中，Dounin 强调了导致 Nginx 发布修复两个关键漏洞的安全补丁的 [最初争议](https://freenginx.org/pipermail/nginx/2024-February/000007.html)。他补充道：

> 我再也不能控制 F5 在 Nginx 中做出哪些变更，我也不再把 Nginx 视为一个为了公众利益而开发和维护的自由开源项目。因此，从今天起，我将不再参与由 F5 运作的 Nginx 开发。相反，我正在启动一个由开发者而非企业运营的替代项目。

Freenginx 并非 Nginx 的第一个分叉和可替代品：[Angie](https://github.com/webserver-llc/angie) 是其他俄罗斯 Nginx 开发者在 F5 于 2020 年离开俄罗斯时创建的，并由俄罗斯公司 [Web Server](https://wbsrv.ru/) 管理。DeepOpinion 的首席后端开发人员 Diogo Baeder [评论](https://www.linkedin.com/posts/activity-7164034851187113984-YnaE/?):

> Nginx 是一款令人难以置信的软件和平台，但我在想，现在是不是该认真考虑一下，基于 Rust 创建一个更现代化的解决方案了。有一个类似的模型，能够"理解" Nginx 的配置语言，并且达到类似的性能水平，但同时又拥有 Rust 的内存安全性和广泛的社区采用，可能会带来一个令人惊艳的新项目 — 或许甚至可以像 Nginx 对 Apache HTTP 做的那样，给 Nginx 带来改变。

网络工程师和架构师文森特·佩特霍尔兹（Vincentz Petzholtz）则持比较悲观的态度，并[补充道](https://www.linkedin.com/posts/vincentz-berlin_announcing-freenginxorg-activity-7164343754835931136-_GEM?utm_source=share&utm_medium=member_desktop)：

> 有时候在项目走向困境时，你只能选择分叉。最终，用户将通过采纳和安装基数来进行投票。

第一个发布版本是[Freenginx-1.25.4](https://freenginx.org/en/download.html?utm_source=thenewstack&utm_medium=website&utm_content=inline-mention&utm_campaign=platform)，采用与 Nginx 相同的 BSD 许可证。Dounin 提供了对只读 [Mercurial 代码库](http://freenginx.org/hg/nginx) 的访问权限，目前放弃了[迁移到 GitHub 的选择](https://mailman.nginx.org/pipermail/nginx-devel/2024-February/YIFSHIYSKDFBYZ2QRA3WF6SRPGIBDBKI.html)。项目启动了新的[开发者邮件列表](https://freenginx.org/en/support.html)。
