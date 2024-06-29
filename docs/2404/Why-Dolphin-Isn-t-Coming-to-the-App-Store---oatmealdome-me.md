<!--yml

category: 未分类

date: 2024-05-27 13:22:50

-->

# 为什么Dolphin不能进入App Store - oatmealdome.me

> 来源：[https://oatmealdome.me/blog/why-dolphin-isnt-coming-to-the-app-store/](https://oatmealdome.me/blog/why-dolphin-isnt-coming-to-the-app-store/)

两周前，苹果修改了他们的App Store指南，允许复古游戏模拟器进入App Store。本周，Delta，一个之前只能通过AltStore获取的多系统模拟器，[在App Store上发布了](https://apps.apple.com/us/app/delta-game-emulator/id1048524688)。

自这些事件发生以来，我们被问过很多次是否会将DolphiniOS（我们的Dolphin分支）提交给App Store。

**不幸的是，不行。**

苹果仍然不允许我们使用对于Dolphin运行至关重要的技术：JIT。

## 什么是JIT？

GameCube和Wii内部都有基于PowerPC的CPU。所有现代的苹果设备使用基于ARM的CPU。无法直接在ARM CPU上运行PowerPC代码，反之亦然。因此，如果我们想在iPhone上运行GameCube或Wii游戏，必须将游戏的PowerPC代码转换为ARM，以便CPU能够理解。

Dolphin使用称为即时（JIT）重新编译器来实现这一点。每当模拟控制台想要运行游戏代码时，Dolphin将使用其JIT将PowerPC代码转换为ARM，然后执行结果。

## iOS上的JIT

不幸的是，苹果通常不允许应用在iOS上使用JIT重新编译器。唯一的例外是Safari和欧洲的替代网络浏览器。

我们向苹果提交了关于JIT支持的DMA互操作性请求，但几周前苹果拒绝了该请求。

很难确切地说为什么苹果如此不愿开放JIT支持。他们可能认为这是安全风险。（查看欧洲对JavaScript JIT在替代网络浏览器中施加的各种限制和限制，他们似乎担心其可能被滥用。）

## 没有JIT的Dolphin？

Dolphin在没有其JIT重新编译器的情况下技术上是可能运行的。这样做时，Dolphin使用所谓的“解释器”来运行PowerPC代码。

不幸的是，解释器比JIT重新编译器慢了几倍。

我们已经附上了DolphiniOS运行的两个视频，这样您就可以自行判断性能差异。一个使用解释器，另一个使用JIT。

没有JIT（使用解释器）

有了JIT

如您所见，基本上是无法游玩的。这些剪辑甚至是在iPhone 15 Pro Max上录制的，这是目前最高端的iPhone。

虽然我们可以只使用解释器将DolphiniOS提交到App Store，但我们可能会因用户对性能差的无休止投诉而遭拒。App Review也可能因为应用不可用而拒绝我们。

## 结论

我们很愿意在App Store上发布DolphiniOS，或者与Dolphin模拟器项目合作，让官方版本进入App Store。

很遗憾，目前除非苹果放宽对即时编译的限制，否则不可能实现。
