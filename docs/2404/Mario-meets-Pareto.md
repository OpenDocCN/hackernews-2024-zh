<!--yml

category: 未分类

date: 2024-05-27 12:53:55

-->

# 马里奥遇见帕累托

> 来源：[https://www.mayerowitz.io/blog/mario-meets-pareto](https://www.mayerowitz.io/blog/mario-meets-pareto)

我们在这里有点开心，但你看到模式了吗？我们经常面临类似的权衡。您想要[既便宜又美味的一餐](https://en.wikipedia.org/wiki/Noodle_soup)？一份既高薪、轻松又充实的工作？[一个低风险、高回报的投资组合](https://en.wikipedia.org/wiki/Modern_portfolio_theory)？一个既灵活又强大、又易于生产的材料？[既公平又高效的税收](https://academic.oup.com/restud/article-abstract/38/2/175/1527903)？[既高质量又快速、又成本高效的LLM](https://artificialanalysis.ai/#summary)？在所有这些情况下，您都面临多目标优化问题，并且必须进行权衡。

当然，如果您已经知道要为每个维度分配的确切权重（即，您知道您的效用函数），您将问题简化为单一目标优化。这是因为您可以将权重与维度结合成一个要优化的单一数量（通常称为效用、成本或适应度）。在这种情况下，您根本不需要帕累托。

但通常您面临的情况是，您的效用函数是未知的或不确定的。在这些情况下，帕累托前沿帮助您客观地消除所有次优选项。它不会从一开始就揭示最佳选项，但现在您可以尝试这些高效选项，并选择最适合您的选项。

### 致谢

本文中，我做了一些简化的假设，以便让大众更易于理解。事实上，我呈现的统计数据转化为游戏内的派生统计数据，并不总是与基础统计线性相关。此外，所有齿轮（除驾驶员外）都有4个速度统计和4个处理统计，但我决定简单地对这些进行平均。我还完全隐藏了效用函数的功能形式，这可能起到了重要作用。如果您希望了解本文背后的更多细节，或者如果您喜欢我的工作并希望未来看到更多内容，请考虑[捐赠一些硬币](https://ko-fi.com/antoinemayerowitz)。

### 积分

超级马里奥维基 [马里奥赛车8豪华版游戏内统计](https://www.mariowiki.com/Mario_Kart_8_Deluxe_in-game_statistics)

亨利·H. [马里奥赛车与帕累托前沿](https://hinnefe2.github.io/python/tools/2015/09/21/mario-kart.html)，2015
