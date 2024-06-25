<!--yml

category: 未分类

date: 2024-05-27 14:59:56

-->

# 碰撞检测

> 来源：[`www.jeffreythompson.org/collision-detection/index.php`](https://www.jeffreythompson.org/collision-detection/index.php)

<canvas id="Introduction" data-processing-sources="code/Introduction/web-export/Introduction.pde">

[ 您的浏览器不支持 canvas 标签 ]

</canvas>

# 碰撞检测

#### 杰夫·汤普森

对象的碰撞是大多数游戏体验和用户界面的基础。 棒球棒与球碰撞，僵尸撞到墙壁，马里奥落在平台上并踩踏乌龟。 即使像用鼠标（点）点击按钮（矩形）这样简单的事情也是一种碰撞。

这本书解释了使用基本形状如圆圈、矩形和直线的算法来模拟这些碰撞，以便您可以将它们应用到自己的项目中。

**更新！** 这个网站一直都有很多访问量，这太棒了。 我做了一些改变，使它在移动设备上看起来更好，使导航更容易，并尝试检查所有错误。 如果您有任何问题或建议，请[在项目存储库上发布问题](https://github.com/jeffThompson/CollisionDetection/issues)。 谢谢！

准备好开始了吗？ 快速跳转到目录...

## 这里涵盖了什么？

这本书涵盖了点、圆、矩形、线、多边形和三角形之间的碰撞。 这些示例旨在尽可能易读和易于理解。 肯定有更快、更有效的方法来检测这些碰撞，但本书旨在友好地教授原则，并尽量减少数学内容。

每个部分都包括对碰撞算法的描述以及使用`processing.js`构建的交互式示例。 您可以在 GitHub 上查看[所有示例（以及本书！）的源代码。](https://github.com/jeffThompson/CollisionDetection)

**注意！** 如果您使用的是移动设备，示例可能不会对您非常有效。 它们是为鼠标输入设计的，所以如果您感到沮丧或手指挡住了视线，请尝试在计算机上查看网站。

## 有哪些不包括的？

与任何书籍一样，比这里所涵盖的更多有用的材料。 没有讨论的东西大多被省略了，因为数学变得太复杂了。 没有涉及到三维空间。 看起来应该相当简单的椭圆实际上非常困难。

如果有特定的未涵盖的碰撞对您有帮助，请[提交一个请求的问题](https://github.com/jeffThompson/CollisionDetection/issues) 或者，更好的是，提交一个您构建的可工作的示例！

## 有问题吗？

如果您发现代码运行不正确，算法解释不太准确，或者是拼写错误，请[在此项目的 GitHub 存储库报告它们](https://github.com/jeffThompson/CollisionDetection/issues)。 感谢您的帮助！

## 导航

好的，让我们写一些代码！点击页面底部的链接，或者顶部的箭头，前往下一章。顶部的`Collision Detection`链接将带你回到目录。

下一步：目录
