<!--yml
category: 未分类
date: 2024-05-27 14:59:26
-->

# High-frequency, liquidity-taking strategy | Databento Blog

> 来源：[https://databento.com/blog/liquidity-taking-strategy](https://databento.com/blog/liquidity-taking-strategy)

In an earlier guide, we showed you how to build an algorithmic trading strategy with a model-based (machine learning) alpha in [Python with Databento and sklearn](https://databento.com/blog/hft-sklearn-python).

In this guide, we'll walk you through how to build a rule-based algorithmic trading strategy instead. We'll also show you how you can compute trading metrics like PnL online with Databento's real-time feed. This example is adaptable to high-frequency trading (HFT) and mid-frequency trading scenarios.

Before breaking down the strategy, let's explain some terminology:

**Feature**: Any kind of basic independent variable that's thought to have some predictive value. This follows machine learning nomenclature; others may refer to this as a predictor or regressor in a statistical or econometric setting.

**Trading rule**: A hardcoded trading decision. For example, "If there's only one order left at the best offer, lift the offer; if there is only one order left at the best bid, hit the bid." A trading rule may be a hardcoded trading decision taken when a feature value exceeds a certain threshold.

**Rule-based strategy**: A strategy that's based on trading rules instead of model-based alphas.

**Liquidity-taking strategy**: A strategy that takes liquidity by crossing the spread with aggressive or marketable orders.

**High-frequency strategy**: A strategy characterized by a large number of trades. There's no public consensus on what this means, but such strategies will usually show high directional turnover, at least 20 bps ADV, and small maximum position. Importantly, low latency and short holding period are not necessary conditions, but in practice, most such strategies exhibit sharp decay in PnL up to 15 microseconds wire-to-wire.

**Mid-frequency strategy**: There's also no public convention on this term, but we'll use it to refer to a relaxation of the low latency and directional turnover conditions as compared to a high-frequency strategy; a mid-frequency strategy will usually have intraday directional turnover in the order of hours for liquid instruments.

Now that we have a few key terms defined, let's dive into the strategy.

<astro-island uid="Z9HpX3" component-url="/marketing-assets/PostHeading._ALulch2.js" component-export="PostHeading" renderer-url="/marketing-assets/client.r4Y8F-6p.js" props="{&quot;depth&quot;:[0,3],&quot;class&quot;:[0,&quot;astro-nenrng47&quot;]}" ssr="" client="only" opts="{&quot;name&quot;:&quot;PostHeading&quot;,&quot;value&quot;:&quot;react&quot;}" await-children=""><template data-astro-template=""></template></astro-island>

One of the simplest types of book feature is called the book skew, which is the imbalance between resting bid depth (vb​) and resting ask depth (va​) at the top of the book. We can formulate this as some difference between vb​ and va​ . It's convenient to scale these by their order of magnitude, so we take their log differences instead.

<astro-island uid="Z2wSff2" component-url="/marketing-assets/PostHeading._ALulch2.js" component-export="PostHeading" renderer-url="/marketing-assets/client.r4Y8F-6p.js" props="{&quot;depth&quot;:[0,4],&quot;class&quot;:[0,&quot;astro-nenrng47&quot;]}" ssr="" client="only" opts="{&quot;name&quot;:&quot;PostHeading&quot;,&quot;value&quot;:&quot;react&quot;}" await-children=""><template data-astro-template=""></template></astro-island>