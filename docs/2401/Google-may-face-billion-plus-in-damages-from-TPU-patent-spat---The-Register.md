<!--yml

category: 未分类

date: 2024-05-27 14:42:52

-->

# 谷歌可能面临来自 TPU 专利纠纷的数十亿美元的损害 • The Register

> 来源：[`www.theregister.com/2024/01/10/google_tpu_patent_dispute/`](https://www.theregister.com/2024/01/10/google_tpu_patent_dispute/)

这周，由 Singular Computing 对搜索巨头提起的陪审团诉讼开始对谷歌对张张的专利侵权指控进行审理。

谷歌被指控侵犯了由 Singular 持有、由计算机科学家 Joseph Bates 开发的专利，后者是一位学术界转型为初创企业创始人。根据他的领英[资料](https://www.linkedin.com/in/joebatessingular/)，Bates 在 1980 年至 2011 年间在美国的康奈尔大学、麻省理工学院、卡内基梅隆大学和约翰霍普金斯大学担任研究和教学职位。

2005 年，Bates 创立了 Singular Computing，以商业化各种计算架构。根据 Singular 的[网站](https://www.singularcomputing.com/)，这家公司“开发和许可用于高性能节能计算的硬件和软件技术，包括大规模和嵌入式计算”。

Singular 与谷歌的法律纠纷可以追溯到 2019 年底，当时 Bates 在马萨诸塞州的一个联邦法院对云巨头提起了诉讼[[PDF](https://regmedia.co.uk/2024/01/09/singular_google_complaint.pdf)]。根据投诉，Bates 在 2010 年至 2014 年间三次根据保密协议向谷歌披露了他提出的各种技术。在此期间，Singular 表示，Bates 让谷歌知道了涉及的技术是受专利保护的。

所谓的专利据称首次于 2009 年提交，并于 2010 年公开，描述了一种设计用于每个处理器周期执行大量低精度计算的计算机架构。投诉称，虽然这种低精度对于传统的计算工作负载可能不实用，但它非常适合可以适应这种低精度的人工智能软件。

Singular Computing 还强调，这些技术不仅仅停留在纸面上，因为在第一份专利申请提交后不久，就基于这些设计制造了一个原型。涉及的专利包括：一项初始的美国[8,407,273](https://patents.google.com/patent/US8407273B2)，以及相关的后续美国[9,218,156](https://patents.google.com/patent/US9218156B2)和[10,416,961](https://patents.google.com/patent/US10416961B2)。

Singular 称，谷歌故意将贝茨的架构纳入其 TPU v2 和 v3 处理器中，而不经过许可或许可证，因此明知侵犯了相关专利。TPU 是谷歌设计的定制人工智能加速器芯片，[在外部帮助下](https://www.theregister.com/2023/09/22/google_broadcom_tpus/)用于加速其云中神经网络的训练和决策过程。

谷歌当前的首席科学家 Jeff Dean 写信给同事们，讨论了贝茨的设计如何“非常适合”这家网络巨头的工作负载，这是根据诉讼中披露的内部邮件。与此同时，谷歌的法律团队辩称，没有任何参与 TPU 开发的人与贝茨或他的蓝图有任何关联。

谷歌已经多次否认了侵权的指控。在向《The Register》发表的一份声明中，一位发言人表示：“Singular 公司的专利主张是可疑的，并且目前正在上诉。它们不适用于我们独立开发了多年的 Tensor 处理单元。我们期待在法庭上澄清事实。”

通过上诉，公关人员指的是本周正在进行的另一起美国上诉法院案件，谷歌将在其中提出理由，说明为什么 Singular 公司的专利应被视为无效。谷歌实质上试图让这些专利被废止，以粉碎 Singular 公司的侵权诉讼。

谷歌的 [TPU](https://www.theregister.com/2016/05/18/confirmed_google_bakes_custom_data_centre_chips/) 于 2016 年推出，最初是为了支持内置在 Gmail、Google 地图和 YouTube 等产品中的机器学习功能。

在高层次上，这些加速器现如今基本上是一堆称为 MXU 的大脑浮点矩阵数学引擎，由一些高带宽内存和几个 CPU 核心支持，使其可编程化。

谷歌如今已经推出了第五代的 [TPU](https://www.theregister.com/2023/12/06/google_unveils_tpu_v5p_pods/)，将其作为云端 AI 训练和推断工作负载的 GPU 替代品。

主审已于周一开始，预计将至少持续两周。

根据谷歌提交的审前文件 [[PDF](https://regmedia.co.uk/2024/01/09/singular_google_pretrial.pdf)]，Singular 公司正在寻求 16 亿至 51.9 亿美元的赔偿，以一次性付款的形式，如果陪审团裁定公司的专利被侵犯。®
