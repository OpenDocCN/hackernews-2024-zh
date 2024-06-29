<!--yml

类别：未分类

日期：2024-05-27 14:38:31

-->

# _无尽工艺_的代数结构 – Arthur O'Dwyer – 主要关于C++

> 来源：[https://quuxplusone.github.io/blog/2024/03/03/infinite-craft-theory/](https://quuxplusone.github.io/blog/2024/03/03/infinite-craft-theory/)

上个月我写了一篇关于Neal Agarwal的网页游戏[*无尽工艺*](https://neal.fun/infinite-craft/)的文章。Tom Fang写信告诉我他创建了[*无尽工艺*](https://szdytom.github.io/infinite-craft-dictionary/)元素、用途和配方的字典。这让我开始思考这款游戏的数学结构。

通过“数学结构”，我指的是我们如何制作配方以及我们可能用来比较一种配方与另一种配方的度量标准。例如，如果我们的目标是制作三明治，我们可以这样做：

```
Wave = Water + Wind
Steam = Fire + Water
Plant = Earth + Water
Sand = Earth + Wave
Tea = Plant + Steam
Sandwich = Sand + Tea 
```

或者像这样：

```
Wave = Water + Wind
Sand = Earth + Wave
Glass = Fire + Sand
Wine = Glass + Water
Sandwich = Sand + Wine 
```

如果我们将这些配方进行图表化，我们会发现第一种配方在某种意义上更*浅显*，因为我们仅组合了基本元素、由基本元素（波浪、蒸汽、植物）制造的元素以及由*这些*元素制造的元素（沙子、茶）。但第二种配方更*简洁*，因为它只有五行而不是六行。

乍一看，*无尽工艺*的世界形成一个[有向超图](https://en.wikipedia.org/wiki/Hypergraph)，具有很强的结构性：每个有向超边连接了一个或两个顶点到正好一个顶点的集合。但配方不仅仅是超图上的“路径”！数学家定义了超图的“路径”的类似物，即一组相交的超边 — “相交”意味着边至少共享*一个*顶点。在这个意义上，从起始元素到三明治存在一个仅由一个超边组成的“路径”；即，

```
Sandwich = Wind + Grilled Cheese 
```

那个“路径”的定义对我们没有帮助。

[我向MathOverflow提问](https://mathoverflow.net/questions/466176/what-is-the-proper-name-for-this-tersest-path-problem-in-infinite-craft)，他们指向了另一个相同问题的例子：[加法链](https://en.wikipedia.org/wiki/Addition_chain)。对于整数\(n\)的“加法链”是从1开始以\(n\)结束的序列，使得序列中的每个元素都是前两个元素的和。例如，我们可以用以下三种方法之一制作31：

```
2 = 1 + 1              2 = 1 + 1              2 = 1 + 1
3 = 1 + 2              3 = 1 + 2              3 = 1 + 2
6 = 3 + 3              4 = 2 + 2              6 = 3 + 3
7 = 1 + 6              8 = 4 + 4              12 = 6 + 6
14 = 7 + 7             12 = 8 + 4             14 = 2 + 12
15 = 1 + 14            16 = 8 + 8             17 = 3 + 14
30 = 15 + 15           28 = 12 + 16           31 = 14 + 17
31 = 1 + 30            31 = 3 + 28 
```

中间链是*最浅显*的，但右侧链是*最简洁*的。

加法链惯例上只写成一个递增的整数序列：(1 2 3 6 12 14 17 31)。我们不需要指明每个整数（比如17）是如何由前面的元素构成的，因为这是显而易见的。我们可以同样简洁地表示*无尽工艺*的配方 — (波浪 沙 玻璃 酒 三明治) — 但这不够友好，因为不明显*哪*两个前置元素组合成了酒。

* * *

寻找最简洁的加法链直接关系到计算机编程领域。假设我们想要计算未知数的31次方并存储在寄存器 `A` 中，只使用乘法。那么我们可以选择以下任意一种方法：

```
mul A, A, B  # A^2     mul A, A, B  # A^2     mul A, A, B  # A^2
mul A, B, C  # A^3     mul A, B, C  # A^3     mul A, B, C  # A^3
mul C, C, D  # A^6     mul B, B, D  # A^4     mul C, C, D  # A^6
mul A, C, E  # A^7     mul D, D, E  # A^8     mul D, D, E  # A^12
mul E, E, F  # A^14    mul D, E, F  # A^12    mul B, E, F  # A^14
mul A, F, G  # A^15    mul E, E, G  # A^16    mul C, F, G  # A^17
mul G, G, H  # A^30    mul F, G, H  # A^28    mul F, G, R  # A^31
mul A, H, R  # A^31    mul C, H, R  # A^31 
```

我们的“浅度”度量转化为衡量这些计算中所涉及的[数据依赖关系](https://en.wikipedia.org/wiki/Data_dependency)的度量。在至少拥有两个乘法器的任何机器上，中间的程序因为最浅，所以速度也是最*快*的。

另一个实际相关的链的“好度量”是其*宽度*：它在其最优[着色](https://en.wikipedia.org/wiki/Register_allocation#Graph-coloring_allocation)中使用的寄存器数量。上面的左侧示例是*最窄*的，宽度为2，而其他示例的宽度为3：

```
mul A, A, B  # A^2     mul A, A, B  # A^2     mul A, A, B  # A^2
mul A, B, B  # A^3     mul A, B, A  # A^3     mul A, B, A  # A^3
mul B, B, B  # A^6     mul B, B, C  # A^4     mul A, A, C  # A^6
mul A, B, B  # A^7     mul C, C, B  # A^8     mul C, C, C  # A^12
mul B, B, B  # A^14    mul C, B, C  # A^12    mul B, C, C  # A^14
mul A, B, B  # A^15    mul B, B, B  # A^16    mul A, C, A  # A^17
mul B, B, B  # A^30    mul C, B, B  # A^28    mul C, A, R  # A^31
mul A, B, R  # A^31    mul A, B, R  # A^31 
```

左侧的示例对应于[俄罗斯农民乘法](https://en.wikipedia.org/wiki/Ancient_Egyptian_multiplication)，它总是生成宽度为2的加法链。有关生成加法链的各种算法的详尽内容，请参见 [Knuth Volume 2](https://amzn.to/49zv6Gs) §4.6.3 “Evaluation of Powers”。

* * *

令人惊讶的是，“最简链”问题在*无限工艺*和加法链中都是非平凡的。Knuth写道：

> 几位作者已经发表了关于二进制方法（即俄罗斯农民乘法）实际上给出了最小可能的乘法次数的声明（未经证明）。但这是不正确的。最小的反例是 \(n = 15\)，当二进制方法需要六次乘法时，我们可以用两次乘法计算 \(y = x^3\)，然后再用三次乘法计算 \(x^{15} = y^5\)，只用五次乘法就能达到所需的结果。

这表明Knuth称之为“因子方法”的算法；但仍然可以找到最优链既不是二进制方法也不是因子方法的数字！看来没有一种快速（非指数时间）的算法可以为*每个*输入生成最优加法链。

要对困难有直观感受 — 特别是为什么没有贪婪算法有所帮助 — 再看看我们到三明治的最简路径：

```
Wave = Water + Wind
Sand = Earth + Wave
Glass = Fire + Sand
Wine = Glass + Water
Sandwich = Sand + Wine 
```

这条到三明治的路径在第四步经过了葡萄酒。现在，到葡萄酒本身的最简路径只有三步：

```
Plant = Earth + Water
Dandelion = Plant + Wind
Wine = Dandelion + Water 
```

但如果你通过这条路径制作葡萄酒，你将永远无法以最优步骤到达三明治！

类似地，我们到31的最简路径是 (1 2 3 6 12 14 17 31)，在第六步经过了17。有两条路径只需五步就可以达到17：

```
(1 2 4 8 9 17)
(1 2 4 8 16 17) 
```

但如果你通过任何这些路径计算出17，你永远无法以最优步骤达到31！

> [AdditionChains.com](http://additionchains.com/)的Neill Clift为我提供了这个例子 —— 我对他表示最衷心的感谢！根据Neill的说法，31的最优链中确实包含数字17的有五条链：其中没有一条链包含 (1 2 4 8)。同时，还有72条不包含17的其他最优链。

* * *

更新，2024年4月：第三种具有这种形状的代数结构来自[StackOverflow](https://stackoverflow.com/questions/78228861/choosing-a-sequence-of-bitwise-operations/78229017)。我们有一个原始元素`A`，代表一个公平硬币翻转，以0.5十进制概率（0.1二进制）落在正面。我们可以使用两种二进制运算之一构造新元素：`&`（表示硬币在两个输入均为正面时落在正面）和`|`（表示硬币在两个输入均为反面时落在反面）。我们试图达到一个代表以概率\(p\)（对于某个\(p = a/2^b\)，介于0和1之间）出现正面的硬币的目标状态。

例如，我们可以以以下两种方式之一制作一个以0.5675十进制概率（0.1001二进制）落在正面的硬币：

```
B = A & A = 0.01          B = A | A = 0.11
C = B & A = 0.001         R = B & B = 0.1001
R = C | A = 0.1001 
```

我对这种特定结构没有太多思考，但它感觉与加法链一样不平凡。

* * *

尽管知道*Infinite Craft*和加法链是相同超图结构的两个例子，但这并没有告诉我是否有一个公认的名字*用于*这种特定的超图结构。如果您有任何线索，请访问[MathOverflow](https://mathoverflow.net/questions/466176/what-is-the-proper-name-for-this-tersest-path-problem-in-infinite-craft)或发送电子邮件给我！

注意加法链结构是可交换的*和可结合的*，而*Infinite Craft*结构是可交换的但不可结合：

```
Plant = Water + Earth
Lava = Earth + Fire
Smoke = (Water + Earth) + Fire
Stone = Water + (Earth + Fire) 
```

这使得Knuth的讨论大部分（特别是“图形表示”和生成等效*对偶*加法链）在*Infinite Craft*中不适用。

* * *

要离线探索*Infinite Craft*超图，无需压力Neal的后端或需要规避其Cloudflare机器人检测过滤器，您可以从Tom Fang的GitHub下载一个包含约30,000个元素的压缩数据库，[szdytom/infinite-craft-dictionary](https://github.com/szdytom/infinite-craft-dictionary/)。为每个元素计算最简洁的配方，并发明一种在数据库中表示这样的配方的紧凑方式，留给读者做为练习！
