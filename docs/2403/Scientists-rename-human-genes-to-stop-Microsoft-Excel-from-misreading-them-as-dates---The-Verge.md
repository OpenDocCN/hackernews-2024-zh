<!--yml

category: 未分类

date: 2024-05-29 12:42:08

-->

# Scientists rename human genes to stop Microsoft Excel from misreading them as dates - The Verge

> 来源：[https://www.theverge.com/2020/8/6/21355674/human-genes-rename-microsoft-excel-misreading-dates](https://www.theverge.com/2020/8/6/21355674/human-genes-rename-microsoft-excel-misreading-dates)

***2023年10月24日更新，美东时间上午11:25：** 微软* [*已经在Windows和macOS上更新了Excel*](/2023/10/21/23926585/microsoft-excel-misreading-dates-human-genes-conversion-fixed), *增加了一个切换选项，可以* [*关闭自动数据转换*](https://insider.microsoft365.com/en-us/blog/control-data-conversions-in-excel-for-windows-and-mac)*。本文的原始版本继续如下。*

人类基因组中有数万个基因：这些微小的DNA和RNA扭曲结合在一起，表达了构成每个人独特特征和特性的所有特征。每个基因都被赋予一个名字和字母数字代码，称为符号，科学家们用它来协调研究。但在过去的一年左右，由于微软的Excel一直错误地将它们的符号识别为日期，一共有27个人类基因被重新命名。

这个问题并不像听起来那么意外。Excel在电子表格领域是一个庞然大物，科学家们经常用它来跟踪他们的工作甚至进行临床试验。但它的默认设置更多地考虑了更平凡的应用，因此当用户将基因的字母数字符号输入到电子表格中时，比如MARCH1——简称为“[Membrane Associated Ring-CH-Type Finger 1](https://www.genenames.org/data/gene-symbol-report/#!/hgnc_id/HGNC:26077)”——Excel会将其转换为一个日期：3月1日。

研究发现，论文中五分之一的遗传数据受到Excel错误的影响

这是非常令人沮丧甚至危险的，会破坏科学家们不得不手动整理的数据。这个问题也出奇地普遍，甚至影响到同行评审的科学工作。一项[2016年的研究](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-016-1044-7)检查了随同3597篇发表论文分享的遗传数据，发现大约五分之一受到了Excel错误的影响。

“这真的非常让人恼火，”英国Quadram研究所的系统生物学家德兹奥·莫多斯告诉*The Verge*。莫多斯的工作涉及分析新测序的遗传数据，他说Excel错误经常发生，仅仅因为这款软件通常是科学家处理数值数据时首先接触到的工具。“它是一个广泛使用的工具，如果你在计算方面有些文盲，你可能会使用它，”他说。“在我读博士期间我也使用了！”

*举例说明微软Excel中基因符号被识别为日期的情况。*

GIF: The Verge

也没有简便的解决方法。Excel并不提供关闭此自动格式化的选项，唯一避免的方式是针对单个列*更改数据类型*。即使如此，科学家可能会修复他们的数据，但将其导出为CSV文件而没有保存格式。或者，另一位科学家可能会加载没有正确格式的数据，将基因符号更改为日期。结果是，尽管有经验的Excel用户可以避免这个问题，但很容易引入错误。

然而，帮助已经到来，以科学机构负责规范基因命名的形式，即HUGO基因命名委员会（HGNC）。本周，HGNC发布了新的[指导方针](https://www.nature.com/articles/s41588-020-0669-3)，包括“影响数据处理和检索的符号”。从现在开始，他们说，人类基因及其表达的蛋白质将以Excel自动格式化为依据来命名。这意味着符号MARCH1现在变成了MARCHF1，而SEPT1则变成了SEPTIN1，等等。旧符号和名称的记录将由HGNC保存，以避免将来的混淆。

据Elspeth Bruford，HGNC的协调员在接受*The Verge*采访时表示，到目前为止，过去一年中已经有27个基因的名称发生了类似的变化，但这些指导方针本身直到本周才正式宣布。“在实施这些变化时，我们与相关研究社区进行了磋商，并在变更生效时通知了那些专门研究这些基因的研究人员，”布拉福德说道。

正如布拉福德所明确的，命名基因的艺术在很大程度上是通过共识驱动的。就像负责更新字典的词典编纂者一样，基因命名委员会必须对其工作将最受影响的个体的需求保持敏感。

以前并非总是如此。在遗传学的早期边疆时代，基因命名经常成为创造性科学家的游乐场，导致了像“sonic hedgehog”（是的，以*Sonic*命名）和“Indy”（缩写为“I’m not dead yet”，指基因的功能，可以使果蝇突变后的寿命加倍）这样臭名昭著的基因的出现。

现在，然而，HUGO基因命名委员会（HGNC）已经牢牢掌控局面，当前的指导方针不再让任何一点随心所欲或自我占据上风。重点是解决实际问题：我们如何最大程度地减少混淆？因此，委员会表示，基因符号应该是唯一的，基因名称应该简洁而具体。它们不能使用下标或上标；只能包含拉丁字母和阿拉伯数字；不应拼写名称或单词，特别是冒犯性的（这一规则在“理想情况下任何语言中”都应适用）。

基因名称应该“理想情况下在任何语言中都避免冒犯”

尽管重命名基因的决定并非轻率，但Bruford说这并不罕见。许多可被解读为名词的基因符号已经被重命名，以避免搜索期间的误报，例如，CARS已更名为CARS1，WARS变为WARS1，MARS调整为MARS1。其他更改则是为了避免侮辱。

“我们总是不得不想象临床医生如何向父母解释他们的孩子具有某个基因的突变，”Bruford说。“例如，HECA曾经以‘headcase homolog (Drosophila)’的基因名命名，以果蝇中的等价基因命名，但我们将其改为‘hdc homolog, cell cycle regulator’以避免潜在的冒犯。”

但Bruford说，这是第一次专门为了应对软件引起的问题而重写指南。到目前为止，反应似乎非常积极——有些人甚至会说是喜悦的。

遗传学家Janna Hutz在Twitter上分享了HGNC新指南的相关部分后，社区的反应非常高兴。“人类基因命名委员会的这一宣布让人非常兴奋，” Hutz本人在推特上写道。“终于！！！”麻省州布罗德研究所的计算生物学家Mudra Hegde回应道。“今天的最大新闻！”一位化名为ScienceSid1的Twitter用户说。

为什么微软在与人类基因学的斗争中获胜？

Bruford指出，对这一决定存在一些异议，但主要集中在一个问题上：为什么重命名人类基因比改变Excel的工作方式更容易？为什么在微软和整个遗传学界的斗争中，科学家们不得不让步？

微软没有对评论请求作出回应，但Bruford认为这根本不值得费事去改变。“这是Excel软件的一个非常有限的使用案例，”她说。“对于那些极其广泛使用Excel的庞大社区的其他用户来说，改变这些功能并没有太大的激励。”

尽管如此，Bruford似乎并不对这种情况感到苦恼。毕竟，她说，等待假设的Excel更新来解决这些问题并不现实，科学家们可以通过引入长期解决方案来解决。微软Excel可能会逝去，但人类基因将与我们同在。最好给它们起一个实用的名字。

***更正：**故事已经更正，以澄清Excel用户可以保存保留其格式的电子表格，避免基因符号被更改为日期的错误。我们对此表示歉意。*
