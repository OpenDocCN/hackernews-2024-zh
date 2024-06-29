<!--yml

类别：未分类

日期：2024-05-27 14:59:26

-->

# [高频、吸收性策略 | Databento 博客](https://databento.com/blog/liquidity-taking-strategy)

> 来源：[https://databento.com/blog/liquidity-taking-strategy](https://databento.com/blog/liquidity-taking-strategy)

在早期的指南中，我们展示了如何使用基于模型（机器学习）的alpha在[Python with Databento and sklearn](https://databento.com/blog/hft-sklearn-python)中构建算法交易策略。

在本指南中，我们将向您介绍如何构建基于规则的算法交易策略。我们还将展示如何使用Databento的实时数据源计算类似PnL的交易指标。这个示例适用于高频交易（HFT）和中频交易场景。

在分解这个策略之前，让我们解释一些术语：

**特征**：任何基础独立变量，被认为具有一定预测价值。这遵循机器学习的命名规范；在统计或计量经济学设置中，其他人可能会将其称为预测变量或回归变量。

**交易规则**：硬编码的交易决策。例如，“如果最佳报价仅剩下一单，提高报价；如果最佳买价仅剩下一单，击中买价。”当特征值超过一定阈值时，交易规则可能是采取的硬编码交易决策。

**基于规则的策略**：一种基于交易规则而不是基于模型的alpha的策略。

**吸收性策略**：一种通过使用侵略性或可交易订单跨越价差以吸收流动性的策略。

**高频策略**：一种以大量交易为特征的策略。公众对此的定义没有一致的共识，但这类策略通常表现出高方向性的换手率，至少每日20个基点的平均每日成交量（ADV），以及小的最大持仓。重要的是，低延迟和短持有期并不是必要条件，但在实践中，大多数这类策略在微秒级的线到线延迟下表现出明显的盈亏（PnL）衰减。

**中频策略**：关于这个术语也没有公众的传统定义，但我们将其用于指称相对于高频策略来说低延迟和方向性换手条件的一种放松；一个中频策略通常会对流动性工具的日内方向性换手具有几小时的顺序。

现在我们已经定义了一些关键术语，让我们深入探讨这个策略。

<astro-island uid="Z9HpX3" component-url="/marketing-assets/PostHeading._ALulch2.js" component-export="PostHeading" renderer-url="/marketing-assets/client.r4Y8F-6p.js" props="{&quot;depth&quot;:[0,3],&quot;class&quot;:[0,&quot;astro-nenrng47&quot;]}" ssr="" client="only" opts="{&quot;name&quot;:&quot;PostHeading&quot;,&quot;value&quot;:&quot;react&quot;}" await-children=""><template data-astro-template=""></template></astro-island>

书籍特征中最简单的一种类型称为书籍偏斜，即书籍顶部的休息买单深度（vb​）和休息卖单深度（va​）之间的不平衡。我们可以将其形式化为vb​和va​之间的某种差异。方便起见，我们通过它们的数量级来缩放这些差异，因此我们取它们的对数差值而不是直接值。

<astro-island uid="Z2wSff2" component-url="/marketing-assets/PostHeading._ALulch2.js" component-export="PostHeading" renderer-url="/marketing-assets/client.r4Y8F-6p.js" props="{&quot;depth&quot;:[0,4],&quot;class&quot;:[0,&quot;astro-nenrng47&quot;]}" ssr="" client="only" opts="{&quot;name&quot;:&quot;PostHeading&quot;,&quot;value&quot;:&quot;react&quot;}" await-children=""><template data-astro-template=""></template></astro-island>
