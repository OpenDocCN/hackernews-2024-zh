<!--yml

类别：未分类

日期：2024年05月27日 14:29:27

-->

# 深切悼念：软件设计先驱尼古劳斯·维尔特 • The Register

> 来源：[https://www.theregister.com/2024/01/04/niklaus_wirth_obituary/](https://www.theregister.com/2024/01/04/niklaus_wirth_obituary/)

瑞士计算机科学家尼古劳斯·维尔特教授于新年第一天去世，大约六周后他将迎来他的90岁生日。

[维尔特](https://people.inf.ethz.ch/wirth/)因创建 Pascal 编程语言而受到公正的赞誉，但这只是他一系列重要语言和研究项目中的一步。他赢得了计算机科学界最高荣誉图灵奖，[于1984年](https://amturing.acm.org/award_winners/wirth_1025774.cfm)，该页面有一些来自[2018年采访](https://www.youtube.com/watch?v=SUgrS_KbSI8)的简短英语片段。

[Youtube 视频](https://www.youtube.com/embed/SUgrS_KbSI8)

尼古劳斯·埃米尔·维尔特于1934年圣瓦伦丁节的第二天出生在瑞士的温特图尔。1959年，他从[苏黎世联邦理工学院](https://ethz.ch/en.html)获得了学士学位，后来他回到了那里，并在那里进行了许多重要的研究。

他一生中几次改变了国家——1960年他获得了加拿大[拉瓦尔大学](https://www.ulaval.ca/en)的硕士学位，1963年获得了加州大学伯克利分校[UC Berkeley*](https://www.berkeley.edu/)的博士学位——伯克利 Unix（通常称为 BSD）的诞生地。在接下来的四年里，他在[斯坦福大学](https://www.stanford.edu/about/)担任计算机科学助理教授。在这期间，他创作了他的前两种编程语言：[Euler](http://pascal.hansotten.com/niklaus-wirth/euler-2/)于1965年，以及1968年出版的[PL/360](https://dl.acm.org/doi/10.1145/321439.321442)。

一部分是由于这项工作，他被邀请加入筹划下一代 ALGOL 编程语言的工作组，以取代[ALGOL 60](https://www.theregister.com/2020/05/15/algol_60_at_60/)。与[英国计算机科学家托尼·霍尔爵士](https://www.theregister.com/2004/06/14/grand_challenge_compsci/)一起，他提出了一个名为[ALGOL-W](http://pascal.hansotten.com/niklaus-wirth/algol-w/)的提案。然而，这个提案被阿德里安·范·温加登提出的更复杂的提案所取代，后者成为了 ALGOL-68。

正如C H林赛的[ALGOL-68历史](https://dl.acm.org/doi/pdf/10.1145/234286.1057810) [PDF]所述，当ALGOL-W提案被拒绝时，威斯特辞去了委员会的职务，并在1968年11月的[Algol Bulletin 29](https://archive.computerhistory.org/resources/text/algol/algol_bulletin/)上发表了一篇强有力的["结语"](https://archive.computerhistory.org/resources/text/algol/algol_bulletin/A29/P32.HTM)，其中包含了一些珍贵的话语，例如：

相反，威斯特采取了他的建议，使之与ALGOL相比略微不那么兼容，并于1970年以帕斯卡的名义发布了它。

这样，帕斯卡创作的故事产生了两种不同但平行的影响。 当然，它是一场[巨大的成功](https://www.theregister.com/2020/05/29/happy_50th_pascal/)，至今仍在使用。 但与此同时，复杂的ALGOL-68却是一个失败：正如*ALGOL-60 Forever*页面所[描述的](https://www.algol60.org/15algolw.htm)：

这并不夸张，ALGOL-60影响了后来发明的每一种编程语言，但其影响在威斯特离职后发布的版本结束。 他自己的语言在研究和商业上都很成功 - Delphi仍在销售中，而[Free Pascal项目](https://www.freepascal.org/)刚刚发布了跨平台的[Lazarus IDE 3.0版本](https://www.lazarus-ide.org/)。 然而，ALGOL-68的复杂性为C、Simula-67及其衍生物C++等更新、简化的语言以及由它们实现的大量其他语言和操作系统提供了机会。

从Van Wijngaarden在1965年的[国际信息处理联合会](https://www.ifip.org/)大会上向威斯特介绍的方式可以略窥两位竞争对手ALGOL提案的领导者之间的关系。 他开了一个笑话，后来变得很有名：

如果威斯特继续致力于ALGOL的研究，它的巨大影响力可能会继续存在。 类似地，如果他坚持使用著名的帕斯卡品牌，他的后续项目可能会取代它，而不是与其预期的替代品一起存活。

1976年，威斯特命名了他的下一个语言[Modula](https://www.research-collection.ethz.ch/bitstream/20.500.11850/68669/1/eth-3057-01.pdf) [PDF]，但很快就被1977年的Modula-2取代了。 这为语言添加了协作进程，称为*协程*，使用了他的前同事C A R Hoare的[通信顺序进程](https://dl.acm.org/doi/10.1145/359576.359585)模型。（这些特性如今在Erlang、Go和Clojure中出现，并且[被我们自己的Verity Stob所讽刺](https://www.theregister.com/2009/12/10/verity_stob/)。）在1980年代和1990年代，Modula-2是行业内的一种重要语言，就像我们去年在[GCC 13增加对其的支持](https://www.theregister.com/2022/12/16/gcc_13_will_support_modula2/)时讨论的那样。

维尔斯在1976年至1977年和1984年至1985年两次在加利福尼亚的施乐帕克进行了为期一年的休假。受到那里的启发，他回到苏黎世创造了更雄心勃勃的奥伯龙系统。奥伯龙是一种编程语言，一个瓦片式窗口开发环境，以及一个完整的自实现操作系统。[奥伯龙手册](http://www.projectoberon.net/wirth/ProjectOberon/PO.System.pdf) [PDF]的介绍包含了后来被称为*维尔斯法则*的内容，尽管他谦虚地将其归功于[Martin Reiser](https://www.theregister.com/2023/01/11/software_versus_hardware/)。

这个项目在[奥伯龙——被忽视的宝石](https://people.cis.ksu.edu/~danielwang/Investigation/System_Security/download.pdf) [PDF]中被描述得非常易读，仅有13页。奥伯龙激发了多个后继者，包括奥伯龙-2、奥伯龙 07 和[组件帕斯卡](https://blackboxframework.org/index.php?cID=home,en-us)。

奥伯龙系统是软件如何可以非常强大而几乎不可思议地小的一种存在证明：2013年版的`inner`、`outer`和`systools`归档总共约有4,623行代码，占据262kB的文本。这就是整个核心环境。但如果这还不够的话，维尔斯在一篇著名的1995年论文中解释了这个想法，["对于精简软件的呼吁"](https://blog.frantovo.cz/s/1576/Niklaus%20Wirth%20-%20A%20Plea%20for%20Lean%20Software%20-%20OCR.pdf)（原版是PDF，但我在[这里](https://liam-on-linux.dreamwidth.org/88032.html)放了一个文本版本）。

他的母校苏黎世联邦理工学院在2021年对他进行了采访，德语视频带有英文字幕：[第1部分](https://inf.ethz.ch/news-and-events/spotlights/infk-news-channel/2021/11/niklaus-wirth-video-interview.html)，[第2部分](https://inf.ethz.ch/news-and-events/spotlights/infk-news-channel/2021/12/niklaus-wirth-video-interview.html) 和 [第3部分](https://inf.ethz.ch/news-and-events/spotlights/infk-news-channel/2021/12/niklaus-wirth-video-interview-part3.html)。

根据许多描述，他很平易近人、友好且风趣；我们特别喜欢来自谷歌员工[Raph Levien](https://levien.com/)的[这篇致敬](https://mastodon.online/@raph/111693863925852135)。维尔斯于1999年4月退休，尽管在2013年，就在他快80岁时，他重新出现并发布了[奥伯龙计划](http://www.projectoberon.net/)的更新版本。

在他的工作中，他创造的语言和工具，在他对更小、更高效软件的雄辩呼吁中——即使在他辞职的项目中——他对计算机行业的影响几乎无法估量。现代软件行业显然未能从他身上学到。尽管他已经离开我们，但他的工作仍有很多东西可以教给我们。®

*本故事的先前版本将之称为"UCSD" Pascal，该项目的一位成员很好地提醒我们，这是来自瑞士苏黎世联邦理工学院（ETH Zürich）的P2编译器的一个派生版本。
