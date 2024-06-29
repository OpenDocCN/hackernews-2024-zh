<!--yml

category: 未分类

date: 2024-05-27 14:54:09

-->

# 柠檬水摊位 | 可能错了

> 来源：[https://possiblywrong.wordpress.com/2024/03/12/lemonade-stand/](https://possiblywrong.wordpress.com/2024/03/12/lemonade-stand/)

*此文讨论在 [Hacker News](https://news.ycombinator.com/item?id=39694353) 上。*

**介绍**

这是一次有趣的怀旧之旅。但最终它变成了试图收集和保存一些复古计算机历史的尝试……我还学到了——或者是忘记然后记起了？——那个将近 45 年前编程环境的另一个有趣怪癖。

1981 年，我有一个住在街道尽头的朋友，他有一台 Apple II+。那一年晚些时候，我的父母为我雇了一位编程导师。我觉得我后来的生活轨迹很大程度上归功于那段与“凯西夫人”在一起的时光，她家里有几台计算机：我记得有一台 Apple II+、一台 VIC-20 和一台 IBM PC。

但直到 1983 年我们才有了自己的家用电脑，一台 Apple //e。其中一个随电脑附送的程序是 [柠檬水摊位](https://en.wikipedia.org/wiki/Lemonade_Stand)，如下屏幕截图所示。

游戏很简单：每天早晨，根据当天的天气预报，你决定制作多少杯柠檬水和多少广告牌，以及每杯售价。这些决定和当天的天气影响需求，你卖出一些杯子后，重复这个过程。

这听起来和它一样有趣……但我记得自己被“逆向工程”游戏的前景所迷住。Applesoft BASIC 源代码包含了计算需求和因玩家输入以及天气而导致的利润的公式。原则上，我们可以“解决”出最大化预期利润的输入。本文由我未能成功尝试这样做而激发。

**历史**

所有在这里讨论的代码，无论是旧的还是新的，都可以在 [GitHub](https://github.com/possibly-wrong/lemonade) 上找到。我们从旧代码开始：`lemonade.bas` 是 Charlie Kellner 的 Applesoft BASIC 源代码，来自我原始的 5.25 英寸软盘的 .dsk 镜像。我相信这是游戏的最早 Apple 版本——为了完整性，我还包括了两个后续发布版本的源代码片段（`lemonade_v4.bas` 和 `lemonade_v5.bas`），但在这里它们的行为是相同的，只是由 Drew Lynch、Bruce Tognazzini 和 Jim Gerber 更新了图形和声音效果。

为了比较，我还从明尼苏达教育计算机协会（MECC，以*俄勒冈小径*闻名）的Elementary Volume 3盘中提取了`selll.bas`的源代码。尽管这个版本出现在1980年后期，我认为这个版本更接近于由Bob Jamison于1973年为UNIVAC大型机编写的原始游戏版本。那个最初的1973年源代码似乎已经湮灭在古老的迷雾中... 但我认为`selll.bas`保留了比Kellner版本更多的父母特性，后者去掉了一些解释需求函数术语的有用注释，误译了一些变量名和`GOTO`行号，导致无法访问的代码等。

**利润最大化**

现在看看新的代码：`lemonade_strategy.py`实现了上述的利润最大化策略。它是纯粹的蛮力方法：给定下面的目标函数，

```
def expected_profit(glasses, signs, price, cost, r1=1, p_storm=0):
    n1 = 54 - 2.4 * price if price < 10 else 3000 / (price * price)
    demand = min(int(r1 * n1 * (2 - math.exp(-0.5 * signs))), glasses)
    revenue = demand * price
    expense = glasses * cost + signs * 15
    return ((1 - p_storm) * revenue - expense, revenue - expense,
            glasses, signs, price)

```

其中`r1`和`p_storm`是天气预报的函数，我们只需评估所有可行输入的预期利润，并找到最大值，但可行区域（我们可以制作多少杯和广告牌）的限制也取决于我们当前的资产：

```
max(expected_profit(glasses, signs, price, cost, r1, p_storm)
    for glasses in range(min(1000, assets // cost) + 1)
    for signs in range(min(50, (assets - cost * glasses) // 15) + 1)
    for price in range(101))

```

我像往常一样陷入了这个兔子洞中，这是在与一位学生讨论之后的常规方式。我的目标并不是真的要“解决”这个几十年前的游戏，而只是为了清楚地举例说明计算机当时有多慢：在AppleSoft BASIC中使用Apple //e的1 MHz处理器，仅仅为Lemonsville的第一个晴天计算最优策略就花费了将近4小时。相比之下，上面提到的Python代码运行速度快得多，仅用了几秒钟，在我十年前的2.6 GHz笔记本电脑上运行的时钟速度“只有”快了2600倍。

**改变天气**

你可以在[互联网档案馆](https://archive.org/details/Lemonade_Stand_1979_Apple)中的Apple版本游戏中进行游戏，这个链接也是从游戏的维基百科页面链接过来的。我就是这样做的，同时我的Python脚本在计算最优策略的同时运行... 结果不起作用。

第一天的天气是“炎热而干燥的”。以2.00美元和每杯柠檬水成本2美分的情况下，最优策略应该是制作74杯柠檬水，3个广告牌，每杯售价12美分，预期利润为6.95美元。但当我在游戏中输入这些值时，我只卖出了37杯，仅获得了2.51美元的利润。

到底发生了什么事？原来，*柠檬水摊*有许多不同的副本存在，既在互联网档案馆，也在各种Apple II的[磁盘镜像存档](https://mirrors.apple2.org.za/ftp.apple.asimov.net/)中... 但从维基百科链接的特定版本在所有这些版本中都是独一无二的，因为它包含了一处在其他副本中都没有出现的重要源代码修改，如下所示：

```
400  REM   WEATHER REPORT
410 SC =  RND (1)
420  IF SC < .4 THEN SC = 2: GOTO 460
430  IF SC < .7 THEN SC = 10: GOTO 460
440 SC = 7
460  REM  IF D<3 THEN SC=2
```

（你可以在互联网档案馆的[这里](https://archive.org/details/LEMONADE_STAND)玩原始未修改的版本。）

我不知道这些变化来自哪里。而且我相信这确实是从原始发布中的*更改*（即不是反过来）。但这仅仅是这三个编辑足以使我的最优策略计算无法产生正确的结果。乍一看这很奇怪，因为上述代码部分仅仅计算了随机生成的天气*预报*，这是后续利润计算的*输入*。在上述修改版本中，天气预报有40%的时间是晴天（`SC=2`），30%的时间是多云（`SC=10`），30%的时间是炎热干燥（`SC=7`）。原始的阈值分别为0.6和0.8，对应概率（0.6，0.2，0.2）为（晴天，多云，炎热干燥）。

第460行中的`REM`是“备注”注释，有效地禁用了游戏的前两天（当`D=1`和`D=2`时）强制晴天预报的原始行为。但为什么这很重要呢？这些都是利润计算的输入，与原始版本完全相同，那么是什么导致了行为上的差异呢？

**未声明的变量**

为了更方便地查找这个问题，我写了`lemonade.py`，这是我试图用Python重制原始的`lemonade.bas`的尝试，去掉了图形和声音 - 也就是说，只是一个经典的文本界面，但其他方面是一行一行的直接翻译，行为完全相同。

那个翻译过程是一个有趣的练习，几乎是一个逻辑谜题，将Applesoft BASIC中基于`GOTO`的非结构化流程控制转换为结构化的Python代码 - 尽管所有的`GOSUB`，我最终很惊讶地发现只需要一个`def`定义的函数和额外的临时布尔变量。

那个翻译帮助我理解了我在原始版本中看到的不同行为的原因。问题出在下面的代码部分，有问题的行已经被突出显示：

```
700  REM   AFTER 2 DAYS THINGS CAN HAPPEN
710  IF D > 2 THEN 2000
...
2000  REM   RANDOM EVENTS
2010  IF SC = 10 THEN 2110
2030  IF SC = 7 THEN 2410
...
2410 X4 = 1
2430  PRINT "A HEAT WAVE IS PREDICTED FOR TODAY!"
2440 R1 = 2
2450  GOTO 805
```

变量`R1`是利润函数的一个输入。但尽管上述460行的代码更改允许第一天的*显示预报*是炎热干燥 - 或者其他非晴天的预报 - 第710行仍然阻止在计算利润时实际*设置*`R1`来反映非晴天预报。

其效果是，在头两天，天气预报可能会*显示*“炎热干燥”或“多云”等，但实际上头两天仍然像原始版本一样是晴天。

所有这些导致了这篇文章的实际动机：当我最初试图在Python版本中复制三行更改的效果时，我遇到了“`NameError: name 'D' is not defined`”，因为在上面的BASIC版本中注释掉了第460行：变量`D`在460行之前从未声明或分配值。

Python 无法处理这一点... 但显然 Applesoft BASIC 可以。变量在使用之前无需声明：在评估包含尚未赋值的变量的表达式时，该变量的值默认为零（或空字符串，对于字符串变量）。

这种行为实际上在游戏中的几个其他地方也有所依赖。我承认，这对我来说是新鲜事——在学习编程的早期阶段，我不记得曾经意识到或利用过这个“特性”。而且这并不是达特茅斯基础语言的特点。在我拥有的所有书籍、手册和杂志中都找不到它的文档... 但我在原始的“[蓝书](https://archive.org/details/Applesoft_Reference_Manual_1978-_bluebook/page/n11/mode/2up)”——Applesoft 参考手册中找到了它的记载。从第9页开始：“*另一个重要的事实是，如果在公式中遇到一个变量，在分配任何值之前，它将自动被赋值为零。然后将零作为该变量在特定公式中的值替换。*”
