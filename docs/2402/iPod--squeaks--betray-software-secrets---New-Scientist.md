<!--yml

category: 未分类

date: 2024-05-27 14:34:16

-->

# iPod 'squeaks' betray software secrets | New Scientist

> 来源：[https://www.newscientist.com/article/dn7085-ipod-squeaks-betray-software-secrets/](https://www.newscientist.com/article/dn7085-ipod-squeaks-betray-software-secrets/)

计算机爱好者们想出了一种巧妙的声学技巧，用自己的代码重新编程苹果的iPod音乐播放器。

当用户在屏幕菜单上滚动时，他们改进了生成点击声音（或“吱吱声”）的组件，以提取设备最新一代的关键信息。这使他们能够安装另一种操作系统，并让他们的iPod运行游戏和其他新程序。

这个项目始于来自德国的17岁计算机科学学生Nils Schneider在圣诞节收到iPod。与大多数新iPod所有者不同，他决定在设备上安装Linux——一种在iPod标准中不常用的免费可用计算机操作系统。

然而，由于播放器的最新一代具有新的硬件，现有的iPod Linux版本不容易安装。尽管如此，Schneider决定自己弄清楚这些组件是如何工作的。他发现自己能够控制设备的一些部分，但不能控制包含设备启动细节的部分，这对于安装Linux至关重要。

## 压电元件

Schneider意识到，与其通过通常的试错过程来解决代码，不如直接倾听它来节省时间。英国软件工程师Bernard Leach帮助建立所谓的iPod Linux项目，他已经找出如何控制iPod内产生点击声音的压电元件。

为了解密引导加载程序代码——这是允许iPod启动的程序——Schneider决定使用Leach的系统将引导加载程序代码播放为声音。

“我试图将单个字节编码为不同点击声音之间的空格，” Schneider告诉**New Scientist**。“看起来行得通，但速度很慢，所以我尝试了一些代码，并发现了如何加快点击速度。”

在编码引导加载程序数据后，他将结果录制的声音传输到另一台电脑上，该电脑被程序化以将其转换回计算机代码。整个过程超过20小时，Schneider不得不为录音制作一个隔音箱。但最终，他成功地提取了信息。

## 严肃目的

这使得在设备上运行Linux成为可能，还能够安装各种兼容软件程序，如简单游戏和音频记录器。

“提取了引导加载程序后，只需几天的工作时间就可以让iPod Linux启动起来了，” Leach告诉**New Scientist**。“否则可能需要几个月。”

Leach解释说，iPod Linux项目在某种程度上是为了乐趣，但也有更严肃的目的。他说：“它将iPod从一个由制造商设定规则的消费设备转变为通用设备。” “许多人感兴趣的是开发各种游戏，但诸如简单计算器、绘图程序，甚至GPS/地图界面等功能也是可能的。”
