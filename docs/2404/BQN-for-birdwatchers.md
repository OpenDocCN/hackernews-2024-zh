<!--yml

category: 未分类  

date: 2024-05-27 13:23:26

-->

# 针对观鸟爱好者的BQN

> 来源：[https://mlochbaum.github.io/BQN/doc/birds.html](https://mlochbaum.github.io/BQN/doc/birds.html)

*现在有一篇[论文](https://dl.acm.org/doi/10.1145/3520306.3534504)（[下载](https://raw.githubusercontent.com/codereport/Content/main/Publications/Combinatory_Logic_and_Combinators_in_Array_Languages.pdf)）涉及到这个主题！ 真是太棒了！*

有些人认为以鸟类命名[组合子](primitive.html#modifiers)是合理的。他们制作类似于[Ornithodex](https://wiki.xxiivv.com/site/logic.html#ref)或Lähteenmäki的综合性的[组合子鸟](https://blog.lahteenmaki.net/combinator-birds.html)列表（自然地，这些列表还有一份[清单](https://combinatorylogic.com/links.html)）。 这些人有些是错的。有些鸟甚至根本就不是真实的。“空想的鸟”？你没听说过鹌鹑吗？尽管如此，我不会公开评判这样受影响的人，而是提供了这个翻译表，以便用他们可以理解的术语解释BQN。  

| BQN | 鸟1 | 1 | 鸟2 | 2 |
| :-: | --- | --- | --- | --- |
| `⊣` | 同一性 | `I` | 凤凰 | `K` |
| `⊢` | 同一性 | `I` | 风筝 | `KI` |
| `∘` | 蓝鸟 | `B` | 黑鸟 | `B₁` |
| `○` | 蓝鸟 | `B` | 表征 | `ψ` |
| `˙` | 凤凰 | `K` |  | `KK` |
| `⊸` |  | `B₁CBSC` | 奇异 | `Q` |
| `⟜` | 翠鸟 | `S` | ~~鸽子 | `D`-like: `labcd.ac(bd)` |
| `˜` | 莺 | `W` | 鹃 | `C` |
| `k G H` | 鸽子 | `D` | 鹰 | `E` |
| `F G H` | 凤凰 | `Φ` | 雉鸡 | `Φ₁` |

Lambda演算在一个或两个参数上没有BQN的多态性，所以每个BQN组合器对应于两个Lambda演算形式，取决于参数的数量，这就给上面两列的鸟。  

输入根据顺序`𝔽𝔾𝕨𝕩`和`GFH`映射到Lambda演算参数，对于3-train `F G H`，映射为`GFH`。例如，当我写到组合`𝕨 𝔽˜ 𝕩`对应于`C`或`labc.acb`的调用时，`a`是`𝔽`，`bc`是`𝕨𝕩`。

鸟类爱好者Conor Hoekstra现在声称他最初误以为是“[金鹰](https://twitter.com/code_report/status/1440208242529882112)”的东西实际上是一只雉鸡。如同在开头提到的那篇[论文](https://github.com/codereport/Content/blob/main/Publications/Combinatory_Logic_and_Combinators_in_Array_Languages.pdf)中宣布的那样,这个新的鉴定是基于 Haskell Curry 在一篇[1931年的论文](https://www.jstor.org/stable/1968422)中使用 `Φ₁` 来表示这种组合器。
