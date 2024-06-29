<!--yml

类别：未分类

日期：2024-05-27 14:47:38

-->

# hn-index

> 来源：[https://www.alexmolas.com/2024/02/12/hn-index.html](https://www.alexmolas.com/2024/02/12/hn-index.html)

<main>

# hn-index

*2024年2月12日 · 5分钟 · 1066字

* * *

**TLDR** - 我用 HackerNews 分数实现了一个 h-index 计算器 - [alexmolas.com/hn-index](https://www.alexmolas.com/hn-index)。

* * *

我总是在寻找新的博客来关注，而要找到质量高的博客的好方法之一就是 HackerNews。我已经有一个精选的博客列表，但我确定我漏掉了很多。我的一个问题是事先很难知道哪些个人资料值得关注，哪些不值得。为了解决这个问题，我决定使用 h-index，这是一种在科学中用来衡量出版物影响力的指标。通过这个指标，我可以给 HN 用户打分，并根据分数决定是否关注他们。作为一个快速提醒，根据 [Wikipedia](https://en.wikipedia.org/wiki/H-index)，h-index 被定义为

> h-index 被定义为 h 的最大值，使得给定的作者/期刊至少发表了 h 篇每篇被引用至少 h 次的论文。

幸运的是，HackerNews 提供了一个开放的 API 来获取用户和项目的数据，有了这些信息，计算这个指数就更或多或少容易了。API 的文档可以在他们的 GitHub 上找到（[HN API 文档](https://github.com/HackerNews/API)）。

在这篇文章中，我将解释我如何实现这个解决方案，并展示如何使用它来计算你的指数。

# 用户数据

用户数据可以从 `https://hacker-news.firebaseio.com/v0/user/USER_NAME.json?print=pretty` 获取，你会得到类似以下的信息：

```
{  "about"  :  "This is a test",  "created"  :  1173923446,  "delay"  :  0,  "id"  :  "jl",  "karma"  :  2937,  "submitted"  :  [  8265435,  8168423,  8090946,  8090326,  7699907  ]  } 
```

# 项目数据

用户数据可以从 `https://hacker-news.firebaseio.com/v0/item/ITEM_ID.json?print=pretty` 获取，你会得到类似以下的信息：

```
{  "by"  :  "dhouston",  "descendants"  :  71,  "id"  :  8863,  "kids"  :  [  8952,  9224,  8917,  8884,  8887  ],  "score"  :  111,  "time"  :  1175714200,  "title"  :  "My YC app: Dropbox - Throw away your USB drive",  "type"  :  "story",  "url"  :  "http://www.getdropbox.com/u/2/screencast.html"  } 
```

我在这里面对的问题之一是所有的项目都暴露在同一个端点上。这意味着你不能特别查询故事。所以你事先不知道一个项目是评论还是故事。我们稍后会看到，这会显著减慢计算速度。

# 将所有内容整合在一起

有了这两个端点，我们就有了计算用户 h-index 所需的所有信息。为此，我们只需获取用户提交的所有内容，然后针对每个提交获取其得分。正如我之前所说，我们不能事先知道用户的哪些提交是故事，哪些是评论，所以我们需要先获取数据，然后按类型筛选，这使得过程变慢。

我实现了计算它的代码在一个 [python 脚本](https://github.com/alexmolas/alexmolas.github.io/blob/master/hn-index/script.py) 中。这段代码运行速度相当快，为 [pg](https://news.ycombinator.com/user?id=pg) 计算其有超过 15k 提交，其中大约 700 是故事，大约需要 20 秒。对于我的用户，大约需要 1 秒。

## 使其可访问

然后，我想如果我想让它更易访问，我应该直接在我的网站上实现它。并且在一位知名的LLM的帮助下，我成功地做到了。现在功能已经在[HN-Index](https://www.alexmolas.com/hn-index/)上可用。

由于我不知道的某些原因，网页版本比Python版本慢得多，我还没有成功地加快它。所以如果你知道如何改进，请写信给我，我会改进的。用于获取和计算指数的JavaScript代码在[这里](https://github.com/alexmolas/alexmolas.github.io/blob/master/hn-index/app.js)。我不是JavaScript专家，请不要对代码过于苛刻。

# 一些见解

最后，让我们看看在h指数方面谁是最优秀的HN用户。由于计算所有用户的指数是不可行的，我决定仅检查[领导者](https://news.ycombinator.com/leaders)的指数。在下表中，我们有最高h指数的用户。

```
| username        |   number of submissions |   h index |   karma |
|:----------------|------------------------:|----------:|--------:|
| ingve           |                   14263 |       283 |  190555 |
| tosh            |                   18329 |       255 |  142664 |
| todsacerdoti    |                   10798 |       252 |  134184 |
| pseudolus       |                   15798 |       238 |  152021 |
| lxm             |                    9622 |       237 |  137389 |
| danso           |                    4871 |       235 |  162226 |
| Tomte           |                   22221 |       232 |  142563 |
| zdw             |                    4125 |       214 |  110034 |
| thunderbong     |                    7380 |       208 |   75116 |
| luu             |                    5309 |       200 |  103885 | 
```

另一方面，有趣的是排行榜上一些用户的h指数非常低，这意味着他们几乎所有的声望来自评论。

```
| username        |   number of submissions |   h index |   karma |
|:----------------|------------------------:|----------:|--------:|
| saagarjha       |                       0 |         0 |   53435 |
| Waterluvian     |                       7 |         3 |   44106 |
| Someone1234     |                       3 |         3 |   47266 |
| dragonwriter    |                      12 |         5 |  115744 |
| tyingq          |                      16 |         5 |   58391 |
| derefr          |                      11 |         6 |   52205 |
| Retric          |                      23 |         6 |   53454 |
| hn_throwaway_99 |                      15 |         8 |   62696 |
| paxys           |                      18 |         9 |   58494 |
| WalterBright    |                      22 |         9 |   68943 | 
```* </main>
