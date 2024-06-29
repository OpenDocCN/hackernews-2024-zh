<!--yml

分类：未分类

日期：2024-05-27 14:42:38

-->

# 正弦波俄罗斯方块 | andreinc

> 来源：[https://www.andreinc.net/2024/02/06/the-sinusoidal-tetris](https://www.andreinc.net/2024/02/06/the-sinusoidal-tetris)

让我们玩俄罗斯方块，但带着一点不同。没有几何图形会*从天而降*。相反，你控制一个[sin波](https://en.wikipedia.org/wiki/Sine_wave)，定义为：\(f(x)=A*sin(\omega x + \varphi)\)：

* * *

^(^[(源代码)](/assets/js/2024-02-06-the-sinusoidal-tetris/tetris.js))

* * *

控制

+   要增加角频率 \(\omega\)，按下：`s`;

+   要减少角频率 \(\omega\)，按下：`x`;

+   要增加振幅 \(A\)，按下：`a`;

+   要减少振幅 \(A\)，按下：`z`;

+   要增加相位 \(\varphi\)，按下：`q`;

+   要减少相位 \(\varphi\)，按下：`w`;

+   要*下落*这个正弦波，按下`p`;

* * *

要赢得游戏，你需要尽可能地将信号减少到接近零。这很困难但不是不可能的。当前的阈值为 `unit * 0.3`。生存并不等于胜利。*交替相位之路*是无聊的。

如果原始信号在游戏缓冲区（画布）外部尖峰，你就输了。

一个专业玩家关闭了默认启用的建议。如果你是一个天才，你可以在脑海中计算*傅里叶级数系数*。取消那些噪音！

要更好地理解发生了什么，请查看[系列的第一篇文章](/2024/04/24/from-the-circle-to-epicycles)。

* * *

这款游戏是使用[p5js](https://p5js.org/)开发的。

源代码[(在这里)](/assets/js/2024-02-06-the-sinusoidal-tetris/tetris.js)并不是我特别引以为豪的东西。

* * *

一些来自网络的讨论：

* * *

^(这款游戏是我在一个周末制作的一个笑话。对于图形我感到抱歉。)
