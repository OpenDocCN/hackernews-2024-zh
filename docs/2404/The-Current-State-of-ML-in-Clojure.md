<!--yml

category: 未分类

date: 2024-05-27 12:59:35

-->

# Clojure 中的机器学习现状

> 来源：[https://codewithkira.com/2024-04-04-state-of-clojure-ml.html](https://codewithkira.com/2024-04-04-state-of-clojure-ml.html)

<main>

# Clojure 中的机器学习现状

<date>*发布于 2024-04-04*</date>

这周我与 [Daniel Slutsky](https://www.linkedin.com/in/daniel-slutsky-42122b4/) 进行了一次非常启发性的交谈（他是一位杰出的数据科学家、软件工程师和社区组织者，如果你还没有与他见过面，我强烈推荐你去见见），讨论了 Clojure 中机器学习领域的当前状态。这篇文章是我为了社区的利益而尝试将其浓缩为摘要，以便更多人可以了解当前的情况以及这一领域的活跃开发者们正在处理的问题。

我喜欢 Clojure，尤其是在 Clojure 中处理数据，但公平地说，Clojure 在数据科学生态系统方面并不像潜在用户可能期望的那样易于使用或理解。这是我今年关注的主要问题，我们正在大力完善工具，使其更容易为更广泛的受众所使用。

在 Clojure 中已经有人在进行“真实”的机器学习工作，以下是截至 2024 年 4 月我们在该领域的库和工具的当前状态概述。

*更新于 2024-04-08：值得一提的是，深度学习和LLM库故意在本文中被省略，以保持其“合理”的长度。在这个领域已经有足够多的独立工作，值得单独进行概述。*

## 摘要

这篇文章中有很多链接。这个表格尝试对它们进行汇总和总结。在下面有更详细的内容值得阅读，但是如果你没时间，这就是要点。简而言之，当前的努力重点是将所有这些令人惊叹的库整合到一个（或至少是一小部分清晰划分的）易于使用的工具包中，为在 Clojure 中进行机器学习提供一个全面的工具集。

| Category | Library | Description | License |
| --- | --- | --- | --- |
| **Java ML Libraries** | [Tribuo](https://tribuo.org/) | 一个全面的 Java ML 框架，是 ML 工作流的首选库 | Apache 2.0 |
|  | [Smile 2.x](https://github.com/haifengl/smile) | 当前正在逐步淘汰主要的 Clojure ML 库 | LGPL |
|  | [Smile 3.x](https://github.com/haifengl/smile) | 因许可证原因避免使用 | GPL |
|  | [XGBoost for JVM](https://xgboost.readthedocs.io/en/stable/jvm/index.html) | 实现梯度提升算法，可通过 Tribuo 使用 | Apache 2.0 |
| **Clojure Wrappers** | [fastmath 2.4.0+](https://github.com/generateme/fastmath) | Clojure ML/stats 库，依赖于 Smile 2.x | MIT |
|  | [fastmath 3.x](https://github.com/generateme/fastmath) | 无 Smile 依赖的 Fastmath | MIT |
|  | [fastmath-clustering](https://github.com/generateme/fastmath-clustering) | Smile聚类的Fastmath封装，依赖于Smile 2.x | EPL-2.0 |
|  | [scicloj.ml.tribuo](https://github.com/scicloj/scicloj.ml.tribuo) | Tribuo的Clojure包装器，可能成为主要的ML算法来源 | EPL-1.0 |
|  | [scicloj.ml.smile](https://github.com/scicloj/scicloj.ml.smile) | 更多Smile功能的Clojure包装器（比fastmath更多），由于许可问题可能被弃用 | EPL-2.0 |
|  | [scicloj.ml.xgboost](https://github.com/scicloj/scicloj.ml.xgboost) | XGBoost的Clojure包装器 | EPL-1.0 |
|  | [tech.ml.dataset](https://github.com/techascent/tech.ml.dataset) | Clojure中的核心数据框架/数据集库，整合了部分Tribuo功能 | EPL-1.0 |
|  | [tech.ml](https://github.com/techascent/tech.ml) | 早期的Clojure机器学习库，被各种scicloj.ml库取代 | EPL-1.0 |
| **Clojure机器学习流水线** | [metamorph](https://github.com/scicloj/metamorph) | Clojure函数组合库 | EPL-2.0 |
|  | [metamorph.ml](https://github.com/scicloj/metamorph.ml) | 基于metamorph的Clojure机器学习流水线库 | EPL-2.0 |
| **集合/框架** | [scicloj.ml](https://github.com/scicloj/scicloj.ml) | Clojure机器学习库和文档集合，正在被noj取代 | EPL-2.0 |
|  | [noj](https://github.com/scicloj/noj) | **整合的Clojure数据科学工具包，可能成为Clojure数据科学栈的主要统一入口点** | EPL-1.0 |
| **互操作性** | [libpython-clj](https://github.com/cnuernber/libpython-clj) | Clojure的Python绑定，允许直接从Clojure使用Python代码和库 | EPL-2.0 |
|  | [sklearn-clj](https://github.com/scicloj/sklearn-clj) | 利用libpython-clj从Clojure直接访问scikit-learn估算器和模型 | EPL-1.0 |
|  | [clojisr](https://github.com/scicloj/clojisr) | 从Clojure到R的桥接，相比Python互操作性更少 | EPL-2.0 |

除了这些库外，文章还提到了[Clojurians Zulip](https://clojurians.zulipchat.com/)，这是主要的Clojure数据科学社区讨论论坛，主要贡献者每天都在活跃。

## Java机器学习库

现在有两个（有点像四个）流行的Java库，它们实现了今天机器学习中使用的许多主要算法和工具（例如分类、回归、聚类、模型开发等）：[Tribuo](https://tribuo.org/)（包括XGBoost，稍后详述）和[Smile](https://haifengl.github.io/)。我们将Smile视为两个库，因为Smile 2.x是LGPL许可，而Smile 3.x是GPL许可，这对某些终端用户可能造成潜在冲突。社区一致认为由于GPL重新许可的问题，应该远离Smile，而集中精力于Tribuo和手工解决方案。

还有[XGBoost](https://xgboost.readthedocs.io/en/stable/jvm/index.html)适用于JVM，如上所述，是梯度提升的一种实现。XGBoost是一套算法集合，而Tribuo是一个更全面的框架（包括数据管理、模型评估和实验跟踪等）。XGBoost可以从Tribuo中使用，因此我并不完全将其视为一个独立的库，尽管它也可以以这种方式使用。

## Clojure包装器

在Clojure中，有两个主要的“家族”库，用于包装这些Java ML库。

[Fastmath](https://github.com/generateme/fastmath)包括Clojure的统计和机器学习工具。Fastmath 2.4.0+依赖于Smile 2，即将推出的fastmath 3.x将完全不依赖Smile。在fastmath 2.x中依赖Smile的聚类功能已经移到了[fastmath-clustering](https://github.com/generateme/fastmath-clustering)库，后续将继续依赖Smile 2.x。社区强烈倾向于避免将GPL许可证库引入生态系统。

[scicloj.ml.tribuo](https://github.com/scicloj/scicloj.ml.tribuo)将主要提供聚类功能，正如你所预期的那样，它包装了Tribuo Java库，并有可能成为生态系统中ML算法的主要来源之一。这是第二组包装上述Java库的几个库之一。其他（不言而喻的）包括[scicloj.ml.smile](https://github.com/scicloj/scicloj.ml.smile)，它比fastmath包装了更多的Smile，以及[scicloj.ml.xgboost](https://github.com/scicloj/scicloj.ml.xgboost)。

还值得一提的是[tech.ml.dataset](https://github.com/techascent/tech.ml.dataset)（作为tablecloth底层的核心数据框架/数据集库），它[整合了tribuo的一些功能](https://github.com/techascent/tech.ml.dataset/blob/master/src/tech/v3/libs/tribuo.clj)，其API围绕单个数据集展开。还曾经有一个叫做[tech.ml](https://github.com/techascent/tech.ml)的库，实现了一些机器学习工具，但已经废弃，被上述各种库所取代。

将API围绕单个数据集而不是其他内容进行定位的概念，引导我们进入下一组库。

## Clojure ML流水线

[Metamorph](https://github.com/scicloj/metamorph) 是一个实现函数组合机制以用于组合机器学习流水线的库。它源于机器学习中的常见实践，即重复使用相同的函数集合但参数不同。例如，您可以尝试许多不同的测试/训练分割来观察其对结果的影响，或者使用许多不同的算法拟合相同的数据，或者尝试使用不同的特征集来训练模型。这导致了流水线排列的激增，因此有必要有机制将机器学习流水线中的可变组件封装为单个函数。这就是 metamorph.ml 的用武之地。

[Metamorph.ml](https://github.com/scicloj/metamorph.ml) 基于元函数和流水线的概念。目前它是在 Clojure 中编排机器学习流水线的核心库。API 是稳定的，但目前有 [多种方法（10+）](https://clojurians.zulipchat.com/#narrow/stream/321125-noj-dev/topic/ols.20interaction.20tutorial/near/422408507) 可以实现相同的结果。这对于有复杂需求和对 metamorph 心智模型有清晰理解的高级用户来说是很好的，但对于新手来说可能会有点令人望而生畏，因此更难选择一个明确的起点。社区正在积极讨论在兴趣驱动下使 Clojure 的机器学习堆栈更易于访问的最佳方法。

## 集合/框架

社区深知想要使用“一键运行”的工具时很难知道从哪里开始，因此已经做出了一些努力来为希望这样的工具的人们提供更明确的路径。[scicloj.ml](https://github.com/scicloj/scicloj.ml) 就是这样一个项目。它是一组库（主要是上述提到的那些），带有一些轻量级的包装器，并且致力于创建文档。

尽管如此，社区正朝着废弃该库的方向迈进，而更青睐于 [noj](https://github.com/scicloj/noj)，我们希望在不久的将来将其稳定下来。目标是在 Clojure 数据科学堆栈中有一个单一的入口点，集成所有需要处理数据的工具到一个地方，具有类似于 R 的 tidyverse 库的无缝互操作性。

## 互操作性

在不提及 [libpython-clj](https://github.com/clj-python/libpython-clj) 无法完整总结 Clojure 中机器学习现状的情况。这是一个为 Clojure 提供 Python 绑定的库，因此您可以直接从 Clojure 调用 Python 代码，如果必要的话。[sklearn-clj](https://github.com/scicloj/sklearn-clj) 利用此桥梁直接在 Clojure 中提供所有 Python 的估计器和模型 [scikit-learn](https://scikit-learn.org/)，因此对于那些仅在 Python 中有的情况，我们仍然可以访问它们。

这里还值得简要提及[clojisr](https://github.com/scicloj/clojisr)，这是一个类似的桥接工具，从Clojure到R（还有为Julia和Wolframite开发的库），但这些对于ML的特定领域来说不那么相关，目前Python是最受欢迎的工具。

## 更多更新

这些讨论都在[Clojurian's Zulip实例](https://clojurians.zulipchat.com)中公开进行，这已成为Clojure数据科学社区的主要聚集地。`#data-science`和`#noj-dev`频道在这些话题上最活跃。你可以在那里跟进最新动态，或者关注github上关键库的更新（[scicloj.ml.tribuo](https://github.com/scicloj/scicloj.ml.tribuo), [metamorph.ml](https://github.com/scicloj/metamorph.ml), [noj](https://github.com/scicloj/noj)）。我也会在这里和我潜伏的所有其他角落定期发布更新。感谢您的阅读！

**在[Hacker News](https://news.ycombinator.com/item?id=39947045)或[Reddit](https://www.reddit.com/r/Clojure/comments/1bxamkm/clojure_for_ml_update/)上讨论此帖。**

*标签: [clojure](tags/clojure.html) [scicloj](tags/scicloj.html) [机器学习](tags/machine-learning.html)*

</main>
