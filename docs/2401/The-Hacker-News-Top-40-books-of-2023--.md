<!--yml

分类：未分类

日期：2024-05-27 14:51:05

-->

# 《黑客新闻》2023年度前40本书籍 -

> 来源：[https://hnreads.com/post/top40_2023/](https://hnreads.com/post/top40_2023/)

嗨，欢迎来到全新的网站 HN 读物。我喜欢阅读 [黑客新闻](https://news.ycombinator.com/)，我喜欢购买书籍（和阅读），我也热爱数据，那么有什么比对书籍的数据进行处理以找到一些有趣的结果更好的呢？！这也给了我机会写一些我觉得有趣的书籍的内容。

这里是黑客新闻读者在2023年推荐的前40本书籍。

我生成这个列表的方法：

+   获取包含“book”一词的故事，但不包括“macbook”、“chromebook”等。这些故事发布于2023年，并且尽量只获取“ask HN”类型的故事，排除其中含有 url 的故事。

+   获取故事的评论

+   对于每条评论，使用 OpenAI gpt3.5 聊天 API 提取书籍详细信息的列表，格式为 json：

```
 [
 	 	{"match":"xxx","title":"xxx", "author":"xxx", "link":"xxx"},
 	 	...
 	 ] 
```

+   规范标题，例如，“哥德尔、艾舍尔、巴赫：永恒的黄金编织” 转换为 “哥德尔、艾舍尔、巴赫”。任何以“the”开头的内容也将被移除。我最终得到了一个标题列和一个原始标题列的数据。如果有相同标题但不同副标题的书籍，例如“Javascript：好的部分”和“Javascript：权威指南”，这并不是太大的问题，因为作者是不同的。

+   聚合得到的信息。为了标准化书名和作者名，进行了一些手动调整，例如 Gödel 有时会被拼写为“Godel”，而 Douglas Hofstadter 有时会被返回为“Douglas R. Hofstadter”。

我还在[我的博客](https://blog.reyem.dev/post/extracting_hn_book_recommendations_with_chatgpt_vol_ii/)分享了一些有关这个过程的详细信息。

很高兴看到我最喜欢的系列之一出现在这个列表上，那就是《超越疆界》系列。我通常不会选择带有恐怖元素的书籍，但故事非常好，科幻元素也很酷。

有一对具有争议性的人物在2023年度前40名中排名不高：埃隆·马斯克的新传记在关于书籍的“Ask HN”中没有被推荐，尽管黑客新闻上有大量关于他和艾萨克森书籍的故事。另外，安·兰德的书在2023年的提及次数不够多，未能入选列表。

毫不拖延，以下是列表：

*有关联属链接的信息请参见脚注*。

| # | 标题 | 作者 | 数量 | 首次提及 |
| --- | --- | --- | --- | --- |
| 1 | [计算机程序的构造和解释](https://www.amazon.com/dp/0262510871?tag=reyemdev0f-20) | 哈罗德·阿贝尔森, 杰拉尔德·杰伊·苏斯曼, 朱莉·苏斯曼 | 26 | [34231811](https://news.ycombinator.com/item?id=34231811) |
| 2 | [设计数据密集型应用程序](https://www.amazon.com/dp/1449373321?tag=reyemdev0f-20) | 马丁·克莱普曼 | 18 | [34231811](https://news.ycombinator.com/item?id=34231811) |
| 3 | [哥德尔，艾舍尔，巴赫](https://www.amazon.com/dp/0465026567?tag=reyemdev0f-20) | 道格拉斯·霍夫斯塔特 | 18 | [34440923](https://news.ycombinator.com/item?id=34440923) |
| 4 | [C程序设计语言](https://www.amazon.com/dp/0131103628?tag=reyemdev0f-20) | 布莱恩·W·克尼根，丹尼斯·M·里奇 | 17 | [35933845](https://news.ycombinator.com/item?id=35933845) |
| 5 | [如何赢得朋友和影响人](https://www.amazon.com/dp/0671027034?tag=reyemdev0f-20) | 戴尔·卡耐基 | 15 | [34310435](https://news.ycombinator.com/item?id=34310435) |
| 6 | [神话般的程序员月度](https://www.amazon.com/dp/B00B8USS14?tag=reyemdev0f-20) | 弗雷德里克·P·布鲁克斯小姐 | 14 | [34333293](https://news.ycombinator.com/item?id=34333293) |
| 7 | [如何解题](https://www.amazon.com/dp/069111966X?tag=reyemdev0f-20) | 乔治·波利亚 | 14 | [34441085](https://news.ycombinator.com/item?id=34441085) |
| 8 | [数学分析原理](https://www.amazon.com/dp/0070856133?tag=reyemdev0f-20) | 沃尔特·鲁丁 | 13 | [34311378](https://news.ycombinator.com/item?id=34311378) |
| 9 | [计算系统的要素](https://www.amazon.com/dp/0262640686?tag=reyemdev0f-20) | 诺姆·尼桑，希蒙·肖肯 | 13 | [34412400](https://news.ycombinator.com/item?id=34412400) |
| 10 | [微积分](https://www.amazon.com/dp/0914098918?tag=reyemdev0f-20) | 迈克尔·斯皮瓦克 | 13 | [34441489](https://news.ycombinator.com/item?id=34441489) |
| 11 | [编码：计算机硬件和软件的隐藏语言](https://www.amazon.com/dp/0735611319?tag=reyemdev0f-20) | 查尔斯·佩兹尔德 | 12 | [34476530](https://news.ycombinator.com/item?id=34476530) |
| 12 | [编写解释器](https://www.amazon.com/dp/0990582930?tag=reyemdev0f-20) | 罗伯特·尼斯特罗姆 | 11 | [34231811](https://news.ycombinator.com/item?id=34231811) |
| 13 | [人件](https://www.amazon.com/dp/0321934113?tag=reyemdev0f-20) | 汤姆·德马科，蒂莫西·利斯特 | 11 | [35930473](https://news.ycombinator.com/item?id=35930473) |
| 14 | [简洁JavaScript](https://www.amazon.com/dp/1593279507?tag=reyemdev0f-20) | 马里恩·哈弗贝克 | 10 | [34412355](https://news.ycombinator.com/item?id=34412355) |
| 15 | [几何原本](https://www.amazon.com/dp/1888009187?tag=reyemdev0f-20) | 欧几里得 | 10 | [34440894](https://news.ycombinator.com/item?id=34440894) |
| 16 | [计算机程序设计艺术](https://www.amazon.com/dp/020103803X?tag=reyemdev0f-20) | 唐纳德·E·克努斯 | 10 | [34442163](https://news.ycombinator.com/item?id=34442163) |
| 17 | [视觉残留](https://www.amazon.com/dp/0765319640?tag=reyemdev0f-20) | 彼得·瓦茨 | 10 | [35054030](https://news.ycombinator.com/item?id=35054030) |
| 18 | [重新思考的电子神话](https://www.amazon.com/dp/0887307280?tag=reyemdev0f-20) | 迈克尔·E·格伯 | 10 | [35169069](https://news.ycombinator.com/item?id=35169069) |
| 19 | [实用编程艺术](https://www.amazon.com/dp/0135957052?tag=reyemdev0f-20) | Andrew Hunt, David Thomas | 10 | [35966675](https://news.ycombinator.com/item?id=35966675) |
| 20 | [编译原理](https://www.amazon.com/dp/8131721019?tag=reyemdev0f-20) | Alfred V. Aho, Monica S. Lam, Ravi Sethi, Jeffrey D. Ullman | 10 | [36146877](https://news.ycombinator.com/item?id=36146877) |
| 21 | [高产管理](https://www.amazon.com/dp/0679762884?tag=reyemdev0f-20) | Andrew S. Grove | 9 | [34262064](https://news.ycombinator.com/item?id=34262064) |
| 22 | [具体数学](https://www.amazon.com/dp/B00NPN0U86?tag=reyemdev0f-20) | Ronald L. Graham, Donald E. Knuth, Oren Patashnik | 9 | [34441399](https://news.ycombinator.com/item?id=34441399) |
| 23 | [永不妥协](https://www.amazon.com/dp/0062407805?tag=reyemdev0f-20) | Chris Voss | 9 | [35171863](https://news.ycombinator.com/item?id=35171863) |
| 24 | [凤凰项目](https://www.amazon.com/dp/1942788290?tag=reyemdev0f-20) | Gene Kim, Kevin Behr, George Spafford | 8 | [34312721](https://news.ycombinator.com/item?id=34312721) |
| 25 | [电子学的艺术](https://www.amazon.com/dp/0521809266?tag=reyemdev0f-20) | Paul Horowitz, Winfield Hill | 8 | [34475069](https://news.ycombinator.com/item?id=34475069) |
| 26 | [沙丘](https://www.amazon.com/dp/0441013597?tag=reyemdev0f-20) | Frank Herbert | 8 | [35053380](https://news.ycombinator.com/item?id=35053380) |
| 27 | [超时空神话](https://www.amazon.com/dp/0553283685?tag=reyemdev0f-20) | Dan Simmons | 8 | [35054030](https://news.ycombinator.com/item?id=35054030) |
| 28 | [银河系漫游指南](https://www.amazon.com/Complete-Hitchhikers-Guide-Galaxy-Boxset/dp/1529044197?tag=reyemdev0f-20) | Douglas Adams | 8 | [35399641](https://news.ycombinator.com/item?id=35399641) |
| 29 | [操作系统：三个易学的部分](https://www.amazon.com/dp/198508659X?tag=reyemdev0f-20) | Remzi H. Arpaci-Dusseau, Andrea C. Arpaci-Dusseau | 7 | [34231811](https://news.ycombinator.com/item?id=34231811) |
| 30 | [用 Python 自动化枯燥的事情](https://www.amazon.com/dp/1593275994?tag=reyemdev0f-20) | Al Sweigart | 7 | [34412667](https://news.ycombinator.com/item?id=34412667) |
| 31 | [每天都在使用的东西的设计](https://www.amazon.com/dp/0465050654?tag=reyemdev0f-20) | Don Norman | 7 | [34439374](https://news.ycombinator.com/item?id=34439374) |
| 32 | [微积分易学](https://www.amazon.com/dp/B09MYWWBDC?tag=reyemdev0f-20) | Silvanus P. Thompson | 7 | [34440860](https://news.ycombinator.com/item?id=34440860) |
| 33 | [线性代数学得对](https://www.amazon.com/dp/3031410254?tag=reyemdev0f-20) | Sheldon Axler | 7 | [34440972](https://news.ycombinator.com/item?id=34440972) |
| 34 | [三体](https://www.amazon.com/dp/0765382032?tag=reyemdev0f-20) | Cixin Liu | 7 | [34606380](https://news.ycombinator.com/item?id=34606380) |
| 35 | [时间的孩子](https://www.amazon.com/dp/B06ZXTHNSJ?tag=reyemdev0f-20) | Adrian Tchaikovsky | 7 | [34611775](https://news.ycombinator.com/item?id=34611775) |
| 36 | [团队的五大功能障碍](https://www.amazon.com/dp/8126522747?tag=reyemdev0f-20) | Patrick Lencioni | 7 | [34627776](https://news.ycombinator.com/item?id=34627776) |
| 37 | [Bobiverse](https://www.amazon.com/dp/1680680595?tag=reyemdev0f-20) | Dennis E. Taylor | 7 | [35053380](https://news.ycombinator.com/item?id=35053380) |
| 38 | [创业者的工作](https://www.amazon.com/dp/B092RC56HW?tag=reyemdev0f-20) | Jessica Livingston | 7 | [35170064](https://news.ycombinator.com/item?id=35170064) |
| 39 | [神经漫游者](https://www.amazon.com/Complete-Collection-William-Neuromancer-Overdrive/dp/1473233968?tag=reyemdev0f-20) | William Gibson | 7 | [35393643](https://news.ycombinator.com/item?id=35393643) |
| 40 | [设计模式](https://www.amazon.com/dp/0201633612?tag=reyemdev0f-20) | Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides | 7 | [35930659](https://news.ycombinator.com/item?id=35930659) |

#### 差一点就入选的名单

最初我想发布前50名，但有太多的并列。我选择在40个项目结束，因为那是被提及7次和被提及6次或更少之间的分界线。以下29本书被提及了5次或6次。

* * *
