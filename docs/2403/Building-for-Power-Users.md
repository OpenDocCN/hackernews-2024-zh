<!--yml

类别：未分类

日期：2024-05-27 14:42:59

-->

# 为专业用户设计

> 来源：[https://www.thediff.co/archive/building-for-power-users](https://www.thediff.co/archive/building-for-power-users)

在本期中：

+   为专业用户设计- 一些应用程序被用作一次性目的非常间歇地使用，比如订餐或查看财务转账状态。有些应用程序则始终如此，但作为消除无聊和替代不知所措的工具。然后，有专业用户的应用程序，典型用户不仅在工作时使用它们作为工具，而且将其作为思考的工具。这些是一个重要的类别，其中许多可用性规则必须进行微调甚至颠倒。

+   广告技术和“天桥”- 我们正在重建旧的开放广告生态系统，但仅适用于最大的参与者。

+   超微！- 五年增长 40 倍是一个可靠的回报，但该股票近期的表现存在一些并非完全基本面驱动的因素。

+   年度增长率环比- 定期提醒，年化增长率可能具有误导性。

+   黑匣子- 较少的广告透明度意味着软件漏洞的经济学已经发生了变化。

+   基金会和周期 - 对冲基金可以立即抛弃一项亏损的持仓，但在人员方面却更难以灵活应对。

## 为专业用户设计

我的 iPhone 告诉我现在安装了 102 个应用，听起来差不多。用于在我的 Mac 上找到所有可执行文件的 bash 脚本显示，除了那些受限于 root 用户的文件外，至少有 200,327 个可执行文件。其中大多数我是偶尔使用的（比如在我用来计算可执行文件的脚本中使用了 `find` 和 `wc`），但偶尔会有一些我基本上生活在其中的应用程序：Emacs、[Superhuman](https://www.thediff.co/archive/the-superhuman-formula/)（$，*Diff*）、[Excel](https://www.thediff.co/archive/the-programmingexcel-efficient-frontier/)（$，*Diff*）、Notion、[Bloomberg](https://www.thediff.co/archive/the-bloomberg-terminal-shows-how/) 等。

这些应用程序属于一个特殊的类别。它们是为专业用户设计的。

专业用户的应用程序有一些共同点：

+   它们是用户谋生或思考的工具。

+   它们通常大部分时间都在使用中。拥有多个显示器的用户通常会将其中一个或多个用于这些应用程序。

+   您可以使用它们多年，并继续了解新功能。

+   他们吸引了党派分子：仅仅提到 Emacs 可能已经导致有人启动 Vi 并撰写一篇关于为什么这是个糟糕决定的长文，打字速度达到每分钟 140 个单词。

+   更新一个移动重要功能、改变热键操作方式或破坏插件的更新可能完全毁掉用户的一天，甚至促使他们转向其他替代品牌。

+   它们通常可以在应用程序内扩展到其他应用程序。Excel 有插件；其中一些插件从其他提供者那里获取数据，而一些 Excel 电子表格也向其他地方提供输出。

+   它们具有痛苦的学习曲线；在一端是权力和普遍性之间的严格权衡，而在另一端是可发现性。而且许多这些产品非常古老，不能修改它们的界面，否则会破坏用户的肌肉记忆，所以这些烦恼会永远存在。

这些产品的一个有趣特征是它们在每个规模上都存在。中间的强大应用程序用户可能是那些工作日大部分时间都在Excel中度过的人。但对于强大用户的中间应用程序来说，可能是由单一公司内部使用建立的某些小工具，它完成它的工作并做得很好。（在研究本篇文章时，我们与那些通过其产品流动数十亿美元的人以及其他产品进行了交流，这些产品中那些完全没有任何用处的人数在几十到几百人之间。）

构建这样的工具意味着非常认真地思考用户试图完成什么，并且以详细的方式模拟最复杂用户的行为。这与许多其他应用形成鲜明对比，这些应用的默认目标是可发现性。Spotify并不是为那些想在异国情调的主题下DJ、做马勒特别棘手乐段的比较分析，或者追踪某一特定嘻哈采样的流行程度的人设计的。不，它是一个更简单的工作任务：我要去上班/去锻炼/快要疯掉了，因为我的孩子们要求一首可能实际上并不存在的新奇歌曲，所以给我一些好听的背景音乐。对于更具交易性的应用程序来说，任务更简单，但元任务更难。如果有人登录他们的人力资源门户并打开手机上安装的半打客户忠诚度应用程序之一，他们可能有半打不同的任务在心中，但他们也可能已经几周或几个月没有查看相关应用了，因此应用程序需要重新学习。这类应用程序可能会非常低地重视保持其体验一致，因为如果它发生变化，没有人会注意到。相反，它们必须高度重视可发现性，并可以默认地假设每个用户都是第一次打开该应用程序。它们实际上不会也无法关心可扩展性、可组合性等问题——很少有用例涉及在定期不使用的应用程序中偶尔做一些复杂的事情。

最大化功率的缺点是它必然会牺牲可发现性。对于经验丰富的用户来说，任何事情都能更快地完成——无论是系统管理员在单个快速shell脚本中交叉参考内存消耗和CPU使用情况，还是Bloomberg用户在几次按键中比较铜的隐含波动率和智利比索的实现波动率——构建能够实现这一点的产品意味着不要阻碍用户，这意味着假设他们知道自己在做什么或者可以快速弄清楚。如果一个产品的中位用户小时来自已经使用该产品数千小时的用户，那么“提示”功能突出显示他们多年来使用的内容只会让他们感到恼火。

如果应用假设用户希望以任何格式使用任何功能，那就是理想的。一个特定的应用可能存在为独立的桌面应用程序、基于浏览器的工具以及移动应用程序。这是面向高级用户类别应用的一个强而不是绝对的要求，因为有些工具如果没有外部键盘是没有意义的（例如使用Emacs或Vi时，具有易错输入机制听起来是令人发狂的）。但是，如果一个应用以多种格式存在，以下几点是至关重要的：

1.  状态始终在所有设备之间同步，这样应用可以作为单一真相源工作，以及

1.  桌面版中没有任何你绝对需要做的事情。

点2需要进一步阐述，但它与这样一个观念有关：面向高级用户的应用是用户工作和思考的应用。如果你乘坐Uber去某个地方，你不会把70%的大脑留在办公室；你用来工作的应用也不应该要求你这样做。在多平台应用中，智能同步也变得非常重要。移动应用是启动将在其他地方完成的事情或反之的不成比例的常见场所：快速记录一些要扩展成真正电子邮件的要点，将股票添加到观察列表中，以便在回到桌子前做更深入的研究，使用新变量更新现有计算并获取输出等等。如果产品在不同产品之间没有精确的功能兼容性，你会花费一些心理开销来考虑你正在使用的产品实例，而不是把全部心思都花在手头的工作上。

这带来了“任务时间”和“工具时间”的重要区别。任务时间是当您建立一个复杂的Excel模型时；工具时间是在您按下F9重新计算一切并最终获得结果之间发生的事情。理解面向高级用户的应用程序的一种方式是它们正在针对最小工具时间渐近线：它们希望对那些确切知道自己在做什么的人最大程度地有用。工具时间在学习如何使用某些东西的早期阶段可能很有用；每个弹出窗口突出显示一个新功能都在告诉您一些您可能并不知道的东西，如果您对产品了解不深，工具时间税的边际影响就很低。但是，在某些时候，工具时间会成为完成任务的障碍，因为用户可以比应用程序更快地思考。

所有这些特点使得这些应用成为有趣的产品，并赋予它们一些独特的经济特性。首先，它们往往是那些已经具有相当经济价值的人工作的有价值补充品。在20世纪80年代的一段时间内，Emacs曾是一款专有软件产品，[许可价格从395美元到2,500美元不等](https://en.wikipedia.org/wiki/Gosling_Emacs?ref=thediff.co)，或者说，在低端的时候，[大约相当于湾区程序员典型年收入的1%](https://www.bls.gov/opub/mlr/1984/09/rpt1full.pdf?ref=thediff.co)。Microsoft在Excel的总价值中占据了极小的一部分（而且幸运的是，不用为所有Excel-enabled的价值破坏负责）。

这些应用程序往往具有令人难以置信的定价权力。当有人的认知转移到应用程序时，换一种意思是从他们的大脑中挖掘出一部分，等待几个月或几年它重新长回来。一些公司推动这一点，比如Bloomberg。对于像Microsoft这样的其他公司，存在一些限制：Microsoft过去担心的是Google通过G Suite从低端颠覆它们，但现在的担忧更多是Google通过在某些应用程序上使Sheets变得更好（例如，它是一个奇怪流行的准后端），并且通过集成更多的与搜索相关的功能，添加了诸如[QR码生成](https://www.lovesdata.com/blog/google-sheets-tips?ref=thediff.co#google-sheets-tip-17)等好东西，并成为Google Forms的默认后端。与此同时，在高端，Microsoft不得不担心一些计算变得足够庞大和至关重要，以至于它们毕业并开始在Python中进行。

这些应用面临的另一种颠覆形式是，它们通常是特定任务的通用实现。“我们只在Excel中跟踪它”通常是一个产品想法的种子，可以针对Excel市场的0.1%，但每个座位收费高达Excel所需特定领域的100倍，扩展到处理核心产品无法完成的事务。

因此，尽管在用户层面上，这些产品非常粘性，在组织层面上却总是面临威胁。人们养成难以改变的习惯，而这些习惯从容易进入的点开始。将一个简单的产品复杂化比简化一个复杂的产品更容易，因此，新的高级用户应用程序可以从特定领域的应用程序中出现。

实际上，这是AI将产生影响的领域。使用LLMs编码可以加快软件开发周期，但前提是创建者清楚地知道他们最终想要构建的东西。我们使用如此多的通用应用的原因是许多目的太小规模，无法支持软件工程团队的全职工作。其中更多的目的足够大，可以作为夜间和周末项目逐步发展为全职业务。

至少在这个类别中，这是必须发生的事情：如果创建者不密切关注其使用方式，特别是用户何时转向其他产品，应用程序只能为高级用户工作。如果你观察某人使用你发现的这样一个应用程序，每个alt-tab都应该是身体上的痛苦 —— 它代表了你的产品无法帮助他们完成的用户常常需要做的事情。如果不是自己成为狂热者，很难向痴迷者销售产品，而这些产品极长的客户生命周期意味着任何不必要的流失都很昂贵。但这只是做这种特定类型业务的成本。

软件问题往往随时间演变成哲学问题，这是一些*Diff*文章中提到的一个观点，如[对立面写作](https://www.thediff.co/archive/antithesis-debugging-debugging/)和[Asana的本体论即服务](https://www.thediff.co/archive/asana-ontology-as-a-service/)（$）。在这一特定类别中，这是关于工作和职业的哲学：一个债券交易员、会计师或社交媒体经理并不一定要花费大量时间反思他们工作的本质。但是，如果您正在构建一个使特定工作更有效率的产品，您自然会成为一个兼职人类学家，对工作本质、流状态的维护以及没有完全适合工作的工具的挫败感有深入的看法。

* * *

披露：长期持有MSFT。

感谢[Readwise](https://readwise.io/?ref=thediff.co)的Daniel Doyon和其他几位匿名讨论过此类产品的人士。

## **Diff Jobs**

+   一家CRM摄取创业公司正在为其LLM驱动的销售软件引入客户，并需要一名后端工程师来优化内部流程和与客户的互动。

+   一家致力于建设21世纪新养老金和普遍基本资本的公司正在寻找一位具有金融科技经验的产品设计师。（纽约）

+   一家投资公司利用人工智能加速对神秘资产类别的投资，正在寻找具有Python和Typescript经验的产品工程师；最好是有自己独立开发经验的人才。（旧金山地区，远程也有可能）

+   一家加密专营交易公司正在积极寻找具有加密经验的系统导向交易员，理想情况下是有跨多个交易所和代币经验的人才。（远程）

+   一家数据咨询公司正在寻找在对冲基金、研究公司或银行具有经验的数据科学家。优先考虑具备替代数据经验者。（纽约）

即使现在你找不到完全匹配你技能和兴趣的职位，我们很乐意早期进行交流，以便在好的机会出现时通知你。

如果你在寻找人才的公司，我们应该谈一谈！Diff Jobs 与跨金融科技、硬件技术、消费软件、企业软件以及其他领域的公司合作，任何一个发现异常有效人才是首要任务的公司。

## 其他地方

### 广告技术和“天空桥”

现在公司更难以在平台间程序化地共享详细的客户数据，广告正在发生变化。具体而言：[大型广告商和大型客户正在达成交易以汇集其数据，这意味着旧广告模型仍然主要可用，但只限于最大的赢家](https://live-adexchanger.pantheonsite.io/commerce-roundup/retail-media-has-created-sky-bridges-between-content-fortresses/?ref=thediff.co)。这不仅意味着一个广告位在被更大公司拥有时能更好地实现盈利，而且在这些交易中也存在规模优势：最大的平台可以达成更加倾斜的交易，通常会进行那些进一步增加其数据优势的交易，通过获取合作伙伴的信息而无需分享任何东西。这导致广告技术和零售业的整合，但考虑到正在整合的公司，它不太可能采取并购的形式，而更可能是较小规模参与者的衰退。

### SuperMicro！

有些股票在半个十年里增长了40倍，也有大型市值公司，但二者结合在一起并不常见。然而，SuperMicro在2019年初以20美元的价格交易，上周五收盘价为803美元（即使是在下跌20%之后）。*《金融时报》*有[关于这家企业的概述](https://www.ft.com/content/5b3ff851-acd9-4588-9ed9-edef8164fbcc?ref=thediff.co)（$）：它们生产服务器，特别是用于AI计算的高性能服务器。这个业务增长相当迅速（尽管它们不为超大规模AI部署提供服务；SuperMicro销售给规模较小的客户，其毛利润率属于半商品化业务，而不是具有持久定价能力的业务。当然，在供需极度失衡时，拥有商品化业务可以赚取丰厚的利润。但还有另一种供需失衡需要考虑：[参考36%流通股份的期权在上周五到期，如果零售投资者通常的期权购买模式持续，其中许多期权已进入实值，迫使期权交易商购买更多股份进行对冲](https://www.fabricatedknowledge.com/p/wtf-is-going-on-at-supermicro-smci?ref=thediff.co)。这种情况可能导致剧烈波动，但需要持续流入新的零售资金才能维持由期权需求驱动的股价，更不用说增加它了。

披露：作为一篮子被过度炒作的AI赌注的一部分，少量SMCI空头。这些赌注部分抵消了对应大型科技公司的多头赌注，后者应该会从AI中受益，但短期内这并不构成对冲；那些较弱或更面向零售投资者的AI赌注往往在更主流的AI赌注下跌时上涨，因为做空它们的杠杆投资者通常也有对大型科技公司的杠杆多头头寸。SMCI实际上比它们大多数更好，但我看到的情况更多是低利润公司暂时低收益，而不是这类公司经济学永久重置的情况。

### GDP 同比增长率

很久以前，2020 年 7 月，*The Diff* 发出严厉警告：[不要将“年度” GDP 数字视为年度数字，因为一些国家的报告是通过季度变化进行年化计算的](https://www.thediff.co/archive/spacs-as-a-call-option-on-hype/#q-q-4-y-y)。这再次变得相关，因为[以色列经济去年第三季度到第四季度收缩了 9%，这相当于年化率达到了 20%](https://www.ft.com/content/763bb384-a974-4222-996f-8aecfbc32074?ref=thediff.co)（$, *FT*）。但这种年化只有在数字是干净的长期趋势时才有意义：任何经济体，其中有大量劳动力转向武装部队，都将经历一些经济中断，但至少这一部分是一次性问题。对于 GDP 讨论来说，令人恼火的是，这一数字往往在极端情况下讨论，但极端数字更可能是由一次性转变引起的，不应该在整个年度内进行推断。

### Black Boxes

*The Diff* 曾多次写到广告业务向使用专有数据和模型定位的大型平台发展的过程。这个过程变得不那么简单的一件事是，当模型错误时，广告客户很难清楚地知道错误的程度及其造成的成本。[Meta 和 Google 最近在其广告定位或观看跟踪系统中出现了 bug，导致它们不得不进行退款](https://www.adexchanger.com/daily-news-roundup/how-facebook-profits-from-overspending-errors-and-speaking-of-hi-google/?ref=thediff.co)。当广告有许多可以调整的参数时——如何选择关键词、瞄准哪些人群、一天中何时调整出价以保持竞争力等，广告客户对他们的支出与收入之间的关联有相当的信心。但当只有一个调节按钮时，允许他们花更多的钱来获得更多的客户，但所有细节都由广告平台决定，bug 和平台利润率增加之间的差异是模糊不清的，并且必须靠诚信制度来解决。

披露：长 META。

### Pods 和 Cycles

[豆荚商店](https://www.thediff.co/archive/pod-people/)（$, *Diff*）在过去一两年中引起如此大的关注的一个原因是它们向新的投资组合经理支付了如此巨大的支票。对冲基金行业发展迅速，但也有一些地方限制了它的结构，其中之一是许多员工有长期的竞业禁止协议：一个大额报价部分是为了支付某人辞职，等待一年，然后*再*去其他地方工作。但这也意味着，基金的成本结构今天由一年前的招聘决定决定，而且由于面试过程如此之长，雇佣决定甚至比那更早。因此，如果超额表现的机会波动，基金将经常发现自己要么人手不足，要么为当前表现付出过高的代价。我们现在似乎正处于[这一周期的后半段](https://www.efinancialcareers.com/news/hedge-fund-jobs-2024?ref=thediff.co)。
