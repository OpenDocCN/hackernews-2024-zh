<!--yml

分类：未分类

日期：2024-05-27 14:56:12

-->

# 流行歌词变得更加重复了吗？

> 来源：[https://pudding.cool/2017/05/song-repetition/](https://pudding.cool/2017/05/song-repetition/)

1977年，伟大的计算机科学家唐纳德·克努斯发表了一篇名为[歌曲的复杂性](https://en.wikipedia.org/wiki/The_Complexity_of_Songs)的论文，这基本上是关于新潮音乐重复歌词的一个长笑话（例如引用：“现代药物的出现导致对内存要求更低，因此定理1的最终改进刚刚宣布”）。

我将尝试用数据来测试这个假设。我将分析在1958年至2017年间登上Billboard Hot 100榜单的15000首歌曲的重复性。

## 如何衡量重复性？

当我听到一首重复的歌曲时，我知道，但将这种直觉转化为数字并不容易。我们可以尝试的一件事是看歌曲中独特词汇的数量，作为总词数的一部分。但这个度量标准会将以下歌词摘录视为同样重复：

> 宝贝，我今晚不需要美元来享受乐趣
> 
> 我爱廉价的刺激
> 
> 宝贝，我今晚不需要美元来享受乐趣
> 
> 我爱廉价的刺激
> 
> 我不需要钱
> 
> 只要我能感受到节奏
> 
> 我不需要钱
> 
> 只要我能保持舞蹈
> 
> ~ 希雅，Cheap Thrills
> 
> 今晚我需要美元
> 
> 我不保留乐趣
> 
> 廉价刺激渴望感受金钱
> 
> 账单不需要跳舞的宝贝
> 
> 好玩的美元舞蹈刺激宝贝我需要
> 
> 不玩好玩
> 
> 不不不今晚不玩舞蹈
> 
> 打败的罐头，我不感觉刺激
> 
> 爱上舞蹈的金钱
> 
> ~ 科林·莫里斯，原创作品

这两个句子都是52个词长，使用相同的23个词汇，但第一个句子明显更加重复，因为它以可预测、重复的顺序排列单词。

## 重复性≈可压缩性？

你可能没听过[Lempel-Ziv算法](https://en.wikipedia.org/wiki/LZ77_and_LZ78)，但你可能每天都在使用它。它是一种无损压缩算法，支持gif、png以及大多数压缩格式（zip、gzip、rar等）。

这与流行音乐有什么关系？Lempel-Ziv算法通过利用重复的序列来工作。LZ能够对文本进行多有效地压缩，直接与文本中重复部分的数量和长度有关。

这是它的工作原理。

你可以在[这里](http://colinmorris.github.io/pop-compression/)探索一些将算法应用于完整歌曲的例子。结果表明，例如，*Cheap Thrills*的整个歌词在压缩时缩小了76%。

76%算是很多吗？这是我数据集中（几乎）所有15000首歌曲的大小减少百分比的分布：

## 流行音乐的重复

从1958年到2017年的15000首歌曲的可压缩性分布，排除20个异常值。

您可能已经注意到 x 轴上的百分比不是均匀分布的。我使用的是一个对数刻度，具有这样的特性，对于任何给定的歌曲，将其压缩一半大小的歌曲与固定距离相隔。例如，20% 和 60% 之间的距离与 98% 和 99% 之间的距离相同。我会在整个过程中使用这种刻度。

我从上面的图表中排除了数据集中最重复的 20 首歌曲。当你看到它们时，你会明白为什么：

这些极端重复的异常是什么？Daft Punk的 *Around The World* 减少了惊人的 **98%** 。它从 2,610 个字符减少到 61 个。足够小到可以放在一条推特里 - 两次！（[这里是歌词](http://www.azlyrics.com/lyrics/daftpunk/aroundtheworld.html)，如果你对这首歌不熟悉的话。）下面是一些其他例子。

## 最重复的歌曲

从美国公告牌百强榜中挑选了 15,000 首歌曲

直觉上，《Around The World》绝对*感觉*像一首非常重复的歌曲。这个列表上的一些熟悉的歌曲实际上是一些滑稽歌曲（麦克雷纳，芭芭拉·斯特赖森德，丁字裤之歌，棉眼乔...）以其傻瓜般的副歌而闻名。这增

那么，音乐是否变得更加重复了吗？当前十年在上述前十名中得到了很好的代表，但它在我的数据集中也稍微过于重视（找到最近歌曲的歌词更容易）。我们需要查看更多的数据才能确定……

## 谁应该为这种疯狂负责？

让我们来看看数据集中一些多产艺术家的平均重复率（那些作为独唱艺术家至少有 15 首排行榜歌曲的艺术家）。

## 每位艺术家的重复性

在这里，音乐类型似乎是一个不同iating因素。在00年代，我们的艺术家实际上分成了两个相当清晰的群体，乡村音乐和嘻哈音乐（还有约翰·梅耶尔的音乐）在左边，流行音乐和摇滚音乐在右边。

艺术家之间的差异是相当大的。男孩帮的平均压缩比率为 60%，布拉德·佩斯利为 38%。换句话说，如果我们让男孩帮和布拉德·佩斯利分别写一首 400 词的歌，并对它们进行压缩，我们预计布拉德的压缩歌曲将比男孩帮的大 50%。

让我们聚焦于特定的艺术家，比如格温·斯蒂法尼。下面的每个圆圈代表她的唱片中的一首歌曲。背景斑点是数据集中*所有歌曲*的直方图（与之前相同，但是镜像的）。

平均而言，格温的音乐极其重复，但是"叫停女孩"这首歌和放松的民谣"酷"之间有很大的差异，后者比数据集中的中位数歌曲要少得多。

你可以在上面的下拉菜单中搜索你最喜欢的艺术家。一些有趣的例子（点击艺术家的名字跳转到他们的排行榜）：

+   人们将 *1989* 描述为她从乡村歌手/词曲作者完全转型为流行歌星的专辑。这些数据为这种说法增添了一些色彩。她的五首最重复的歌曲中有 4 首来自 *1989*。

+   **XX** 有着长期而一贯的高度重复的音乐传统。有趣的是，她最不重复的两首歌曲可以说是唯一两首不属于流行音乐流派的歌曲：它们都来自音乐剧《艾维塔》，由蒂姆·赖斯和安德鲁·劳埃德·韦伯创作。

+   像 **XX** 和 **XX** 这样的说唱歌手往往是始终不重复的。

+   难怪 **XX** 赢得了数据集中最重复艺术家的称号？她的超过半打的歌曲都位于分布的*极*右侧（包括，恰当地，*Pon de Replay*）。

`<main></main>`