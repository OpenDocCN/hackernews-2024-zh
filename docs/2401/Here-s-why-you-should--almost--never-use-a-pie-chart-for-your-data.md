<!--yml

分类：未分类

日期：2024-05-27 14:36:26

-->

# 这就是为什么你几乎永远不应该对你的数据使用饼图的原因

> 来源：[https://theconversation.com/heres-why-you-should-almost-never-use-a-pie-chart-for-your-data-214576](https://theconversation.com/heres-why-you-should-almost-never-use-a-pie-chart-for-your-data-214576)

我们的生活越来越依赖数据。我们的手机监视我们的时间和上网情况，在线调查了解我们的观点和喜好。这些数据收集用于告诉我们我们睡得如何或者我们可能想购买什么。

数字对于日常生活变得越来越重要，但人们的数字技能却落后了。例如，澳大利亚高中12年级学生参加高级和中级数学的比例[几十年来一直在下降](https://amsi.org.au/?publications=year-12-participation-in-calculus-based-mathematics-subjects-takes-a-dive-2)。

为了帮助普通人理解大数据和数字，我们经常使用可视化摘要，比如饼图。但是，虽然非数字人士会避开数字，但大多数数字人士会避开饼图。原因在于这里。

## 什么是饼图？

饼图是一种代表数字百分比的圆形图表。圆被分成片，每个片的大小与其代表的类别成比例。它之所以被命名是因为它类似于切片的馅饼，可以以许多不同的方式“上桌”。

下面的示例饼图显示了上次选举前澳大利亚两党倾向选票，工党占55%，联盟党占45%。两个近似半圆显示了相对激烈的竞争——这是一个饼图的有用示例。

一个简单的饼图显示了一次民意调查中澳大利亚两个主要政党的百分比。Victor Oguoma

## 饼图有什么问题？

一旦我们有了超过两个类别，饼图就很容易误传百分比并且难以阅读。

下面的三个图表是一个很好的例子——很难弄清楚哪个区域是最大的。饼图的圆形意味着各区域缺乏共同的参考点。

三个示例饼图，每个饼图有五个相似的类别。你能快速分辨出每个饼图中哪个颜色最大吗？[Schutz/Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Piecharts.svg)，[CC BY](http://creativecommons.org/licenses/by/4.0/)

当有大量类别时，饼图也表现不佳。例如，这张来自一项关于COVID数据可视化使用的研究的图表显示了一个饼图中的数百个类别。

一个带有几十个类别的饼图。并非每个类别都有标签，不清楚总类别数是多少以及未标记的切片是指什么。[Trajkova et al., Informatics (2020)](https://doi.org/10.3390/informatics7030035)，[CC BY](http://creativecommons.org/licenses/by/4.0/)

微小的切片、缺乏清晰的标签和五颜六色的色彩使得任何人都难以解读。

对于色盲患者来说情况甚至更加糟糕。例如，这是一个模拟，展示了对绿色光敏感度降低的人看上去会是什么样子。这是最常见的色盲类型，大约影响[4.6%的人口](https://wearecolorblind.com/articles/a-quick-introduction-to-color-blindness/)。

与上文相同的数据图表，但通过模拟滤镜运行，以展示对普通类型的色盲者看起来会是什么样子。Trajkova 等，Informatics (2020); 修改，[CC BY](http://creativecommons.org/licenses/by/4.0/)

如果我们将饼图变成三维的，情况可能会变得更糟。这可能导致数据的严重误导。

下面，黄色、红色和绿色区域的面积都相同（三分之一），但根据角度和哪个扇形位于饼图底部，它们看起来不同。

一个标准的二维饼图和两个三维饼图。在每个图表中，比例都是三分之一，但在三维版本中似乎存在州之间的差异。Victor Oguoma，[CC BY-ND](http://creativecommons.org/licenses/by-nd/4.0/)

## 那么，为什么饼图无处不在呢？

尽管饼图存在众所周知的问题，但它们无处不在。它们出现在期刊文章、博士论文、政治民意调查、书籍、报纸和政府报告中。它们甚至被澳大利亚统计局使用过。

尽管统计学家们已经批评它们几十年了，但很难反驳这个逻辑：“如果饼图如此糟糕，为什么会有那么多呢？”

可能它们之所以受欢迎是因为它们本身受欢迎，这是一个适合饼图的循环论证。

从各种开放获取来源收集的糟糕饼图集合，包括“爆炸”的饼图和三维饼图。Adrian Barnett 和 Victor Oguoma，[CC BY-ND](http://creativecommons.org/licenses/by-nd/4.0/)

## 什么是饼图的良好替代方案？

有一个简单的解决方案可以在小空间内有效总结大数据，仍然允许创意色彩方案。

它是普通的条形图。还记得上面五个类别的令人头痛的饼图示例吗？这里是使用条形图的相同示例 - 我们现在可以立即看出哪个类别最大。

三个饼图，每个有五个类似的类别，并使用条形图呈现相同的数据。Schutz/Wikimedia Commons，[CC BY](http://creativecommons.org/licenses/by/4.0/)

线性条比饼图的非线性部分更容易被眼睛接受。但是要小心，不要通过添加三维效果使一般的条形图看起来更有趣。正如你已经看到的那样，3D图表会扭曲感知，使寻找参考点变得更加困难。

下面是1992年美国总统选举中按家庭收入分段（从低于15000美元到超过75000美元）的选民人数的标准条形图和3D替代品。使用3D版本，你能告诉最高收入类别的每个候选人的选民人数吗？不容易。

同样的选民数据呈现为标准的二维条形图和一个无益的三维版本。Victor Oguoma，[CC BY-ND](http://creativecommons.org/licenses/by-nd/4.0/)

## 使用饼图永远可以吗？

我们展示了一些最糟糕的饼图示例以说明一个观点。饼图在只有少数几个类别且百分比不同的情况下可能还可以，例如一个大类别和一个小类别。

总的来说，最好是谨慎使用饼图，尤其是在有更“易消化”的替代方案时——条形图。

每当我们看到饼图时，我们会想到两件事：它们的创建者不知道自己在做什么，或者他们知道自己在做什么，并且有意试图误导他人。

图形摘要旨在轻松快速地传达数据。如果你觉得需要将其装饰一番，你很可能是在无意中降低了理解。

* * *

***阅读更多：[下次看到图表或地图时，问问自己这3个问题](https://theconversation.com/3-questions-to-ask-yourself-next-time-you-see-a-graph-chart-or-map-141348)***

* * *