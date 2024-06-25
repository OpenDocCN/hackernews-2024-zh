<!--yml

类别：未分类

日期：2024-05-27 14:30:23

-->

# [Chris的维基](https://utcc.utoronto.ca/~cks/space/blog/sysadmin/TenYearsNotLongEnough) :: 博客/sysadmin/十年远不够

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/sysadmin/TenYearsNotLongEnough](https://utcc.utoronto.ca/~cks/space/blog/sysadmin/TenYearsNotLongEnough)

最近的时限新闻是，[每个Keybase客户端内置的十年证书在2023年底到期](https://mstdn.social/@jschauma/111688103150154714)。我对此[有点反应](https://mastodon.social/@cks/111689032364322410)：

> 十年默认值给许多人的固定证书或私有CA部署带来了如此多的悄悄的破坏。太多了。
> 
> （我们遇到了这个问题，所以我们的私有OpenVPN CA根是要长得多得多的。）

如果你打算让某个内部事物保持良好状态长达十年，不要这样做（无论是TLS证书还是，例如，设置某个数据库的保留期限）。十年要么太短，要么太长。如果你在写一个例子并使用十年的有效期，请不要这样做（人们一定会复制它）。当你设置某些东西时，十年听起来可能是一个不太可能的长时间，但众所周知，没有什么比一个快速的黑客更加永久，十年后你就会遇到一些问题。 [我们在OpenVPN方面遇到了这个问题](/~cks/space/blog/sysadmin/FailingAtTLSRootRollover)，并且[我们未能找到解决方案](/~cks/space/blog/sysadmin/OpenVPNTLSRootExpirySolution)，给我们带来了痛苦。

如果你没有一个计划要么更新和更新它，要么肯定将其停用，十年实在是太短了。如果你想保持某个东西活跃，直到你改变主意，将你的过期日期，保留期限或任何其他内容设置得尽可能高。十年对此来说是一个可怕的值；它看起来和感觉起来是足够长的，但实际上远远不够。

（在某些情况下，现在距离[2038年问题](https://en.wikipedia.org/wiki/Year_2038_problem)仅剩十四年可能会有点问题。但如果你有这方面的问题，最好早点发现。）

如果你想在一段时间后肯定将你的东西停用，十年实在是太长了。如果你的东西在使用中仍然存在十年，几乎可以肯定它已经渗透到各种地方，以至于停用它会导致爆炸，并且可能会有人出现要求将其寿命延长（假设人们甚至还记得它在哪里使用以及依赖它的是什么）。

如果您计划滚动、延长或更新持续时间，十年也太长了。正如每个人通过与TLS证书的痛苦经历发现的那样，每隔几年才做一次事情会导致问题（而且可能会忘记需要做这件事情）。为了确保您保持实践和流程仍然有效，您需要比每十年一次更频繁地执行此操作。在这一点上，我可能会尝试每年两次。

我并非无罪，因为目前我们[Prometheus数据库](/~cks/space/blog/sysadmin/PrometheusGrafanaSetup-2019)的保留时间是从2018年11月底开始的“3650天”。也许在那之前我们会耗尽磁盘空间，当我们到达那里时，我们可能会决定我们并不需要十年（或十年以上）的时间，但无论如何我应该提前调整，因为我们的意图是“尽可能保留尽可能多的数据，并且如果我们的磁盘空间不够时可能会增加更多的磁盘空间”。我应该在日历中留下某种提醒，以防万一。

（我们现在检查所有长期有效的内部TLS证书，并在它们的到期时间接近时发出警报。这可能有些过度，但不会有害。而且其中一些距离十年还不到。）

（这与[一个更古老的发现有关，即“999天”并不永远](/~cks/space/blog/web/NotForever)，事实上，9999天也不是，尽管这已经是27年多了，所以它更接近。99999天对几乎所有人来说已经足够长了，不用担心了。）