<!--yml

类别: 未分类

日期: 2024-05-27 13:17:01

-->

# 我无法使用我的数字键盘输入2FA代码 – Terence Eden's Blog

> 来源：[https://shkspr.mobi/blog/2024/04/i-cant-use-my-number-pad-for-2fa-codes/](https://shkspr.mobi/blog/2024/04/i-cant-use-my-number-pad-for-2fa-codes/)

这肯定是我提交过的最令人恼火的错误报告。

我要在一个网站上输入我的2FA代码 - 但屏幕上没有显示任何数字。显然，我是个笨蛋，忘记按NumLock按钮了。哎呀！我切换了它并重新输入。仍然没有显示数字。我切换到另一个选项卡，当我输入它们时，我的数字出现了。因此，我相当确信我的键盘是正常工作的。

我切换回2FA输入并再次尝试。仍然没有任何反应。然后我尝试使用键盘顶部的数字行输入数字。我的2FA代码出现了。

究竟是什么鬼？！？

开发者经常使用JavaScript来"改进"HTML的标准功能。例如，使用`<input type="number">`会有一些[可访问性问题](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/number#accessibility_concerns)，而使用[`inputmode="numeric"`](https://css-tricks.com/everything-you-ever-wanted-to-know-about-inputmode/#aa-numeric)在移动设备上显示数字键盘非常棒，但用途不大。

因此，开发者希望找到一种可靠的方法，确保用户*只能*输入数字。说得也对。

有两种方法可以做到这一点 - 正确的方法和错误的方法 - 使用[`KeyboardEvent`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent)。

一种方法是[监听从键盘发送的字符](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key) - 称为`key`。

另一种方法是[监听键盘上按下的*按钮*](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/code) - 称为`code`。

这在[keyjs.dev](https://keyjs.dev/)上有一个很好的演示 - 尝试一下，看看你的浏览器能检测到哪些键盘按钮。

当我在键盘顶部行按下7键时，键是7，而代码是**`Digit7`**。

但是当我在我的数字键盘上按下7键时，键是7，但代码是**`Numpad7`**。

网站上的JavaScript在拒绝不是“Digit”的任何键码！

或许我是一个怪人，坚持同时拥有并使用我的数字键盘？也许开发者需要在除了MacBook之外的设备上进行测试？也许JavaScript是个错误，Web没有它会更好？

无论如何，不要像那个网站一样。让用户使用他们喜欢的任何键输入。
