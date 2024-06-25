<!--yml

类别：未分类

日期：2024-05-27 14:26:03

-->

# JavaScript 真的是在 10 天内完成的吗？ • Buttondown

> 来源：[`buttondown.email/hillelwayne/archive/did-brendan-eich-really-make-javascript-in-10-days/`](https://buttondown.email/hillelwayne/archive/did-brendan-eich-really-make-javascript-in-10-days/)

<日期>2023 年 9 月 28 日</日期>

# JavaScript 真的是在 10 天内完成的吗？

我曾听说 JavaScript 之所以有那么多缺陷，是因为第一个版本只花了十天时间。我很好奇 1）这是真的吗？2）这是否可以解释该语言的缺陷。

经过一些研究，我可以不自信地说：这很复杂。

JavaScript 的“第一个版本”确实只花了十天时间。确切的日期尚未确认，但 Brendan Eich 回忆说是 [1995 年 5 月 6 日至 15 日](https://www.quora.com/In-which-10-days-of-May-did-Brendan-Eich-write-JavaScript-Mocha-in-1995)。但这只是一个供内部演示的最小原型（“Mocha”）。JavaScript 1.0 是在 1996 年 3 月公开发布的（[第 10 页](http://www.wirfs-brock.com/allen/jshopl.pdf)），第一个“完整”的版本在 1996 年 8 月发布（同上）。即使在那之后，Netscape 团队仍经常调整 JS 的设计；Eich 在 1996 年秋季 [回忆道](https://brendaneich.com/2011/06/)：“比尔·盖茨一直抱怨我们一直在改 JS”。

Eich 在语言设计和编译器开发方面拥有约十年的经验，并且 Netscape 明确聘请他在浏览器中引入一种编程语言（第 7 页）。最初应该是 Scheme，但后来 Netscape 与 Sun 签署了协议，同意将其更改为更“类似 Java”的语言。

### 这解释了缺陷吗？

JavaScript 现代的大部分缺陷可以说 *并不是* 由于开发时间短导致的：

+   Mocha 最初并没有隐式类型转换，但用户要求 Eich 在 1.0 中添加了这个功能（[视频链接](https://youtu.be/krB0enBeSiE?si=s9_oUf9Tp9Nxz-qh&t=2503)）。他对此深感后悔。

+   JS 1.0 添加了 `null` 以更兼容 Java（[第 13 页](http://www.wirfs-brock.com/allen/jshopl.pdf)）。Java 兼容性也是 `typeof null = object` 的原因。

+   任何 JavaScript API 的缺陷必须是后来才出现的，因为 *所有* 的 API 工作都是在 Mocha 之后进行的。Mocha 是一种相当简单的语言！

+   “所有数字都是浮点数”问题最初出现在 Mocha 中，但我认为这始终是预期的行为。[JavaScript 1.0 手册](https://web.archive.org/web/19970613234917/http://home.netscape.com/eng/mozilla/2.0/handbook/javascript/index.html) 引用了 HyperTalk 作为主要灵感。我从未使用过 HyperTalk，但浏览手册给我一种它执行相同操作的感觉（[第 102 页](https://cancel.fm/stuff/share/HyperCard_Script_Language_Guide_1.pdf)，517）。

我发现 10 天的开发确实对 JavaScript 造成了伤害：Brendan Eich 没有时间添加垃圾回收器，后来的尝试添加垃圾回收器引入了一堆安全漏洞（[43:04](https://youtu.be/krB0enBeSiE?si=dHTMzc0DXbj-1ELQ&t=2584)）。

这份通讯创下了“每字研究时间最长”的新纪录。

*如果您在网上阅读本文，可以在此处订阅/hillelwayne。更新频率为每周一次。我的主要网站在[这里](https://www.hillelwayne.com)。*
