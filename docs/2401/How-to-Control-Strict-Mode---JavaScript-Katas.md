<!--yml

category: 未分类

日期：2024-05-27 14:30:18

-->

# 如何控制严格模式 | JavaScript Katas

> 来源：[`jskatas.org/blog/2023/10/16-control-strict-mode/`](https://jskatas.org/blog/2023/10/16-control-strict-mode/)

我惊讶地发现，工作线程默认情况下以非严格模式运行。这种情况应该很少见，因为使用 ES 模块（ESMs）、类或`"use strict"`时，引擎会强制使用严格模式。一开始我不知道这一点。我一直在为这个网站使用的卡塔进行工作，这个网站使用[javascript-katas 仓库](https://codeberg.org/wolframkriesing/javascript-katas)，它使用 ESMs 和`type=module`，这是我的默认设置。这使得整个 nodejs 项目，任何`npm ...`命令都会使用严格模式。逃离严格模式并不容易，但以非严格模式运行代码的一种方法是使用工作线程。

我总是喜欢了解事物的起源，所以让我们来读一读严格模式。

## 严格模式 – 何时以及为什么？

严格模式是在[ES5（2009 年）](https://262.ecma-international.org/5.1/#sec-4.2.2)中引入的。规范中说严格模式被添加了

> 为了保证安全性，避免[...]认为是容易出错的功能，获得增强的错误检查

和

> 严格模式还指定了必须通过抛出错误异常报告的额外错误条件

严格模式是为了让 JS 更安全而引入的。

## 严格模式？

实际上，我使用的工作线程只是解决我真正问题的一种方法，即运行`arguments`卡塔的测试。为了做到这一点，我需要以非严格模式运行测试，所以让我们来看看严格模式。

现在严格模式起到什么作用？我一直在深入研究"function API"，深入挖掘最初的[ES1 规范](https://www.ecma-international.org/wp-content/uploads/ECMA-262_1st_edition_june_1997.pdf)和所有后续版本，以找出何时引入或弃用了什么（JavaScript 很少被弃用）。所以我找到了`arguments`，当我写下了以下测试时：

```
const fn = function() {
  const arguments = 'arg';
  return arguments;
}
assert.equal(fn('the argument'), 'arg'); 
```

它以"在严格模式中意外 eval 或 arguments"失败，这不是由于测试代码本身造成的，而是由于我在工作线程中运行的其余代码。上面的测试只是被注入了，只是工作线程中整个代码的一小部分。代码中有一个`import`，它将所有代码都转换为严格模式运行。正如错误所说，在严格模式中不允许使用`arguments`。

所以我想要控制何时开启严格模式。但为此，让我们先更好地了解严格模式。

### 什么会强制使用严格模式？

正如上面提到的，`"use strict"`会开启严格模式，但在这种情况下，工作代码中还没有这个，那么是什么打开了它？在[ES6 规范章节"Strict Mode Code"](https://262.ecma-international.org/6.0/#sec-strict-mode-code)中，新增了以下内容，强制开启了严格模式：

> 模块代码始终是严格模式代码。
> 
> ClassDeclaration 或 ClassExpression 的所有部分都是严格模式代码。

所以`import`把它打开了。

### 如何控制严格模式？

所以我重写了代码，不使用`import`，它在非严格模式下运行。

现在我可以控制严格模式，我只需在代码顶部添加`"use strict"`。更好的是，我可以把`"use strict"`放在函数体的开头，只有这个函数以严格模式运行。它必须是这个函数中的第一条语句，请参见下面的代码进行一些探索。哦，我看到了新的挑战出现了。

```
'use strict'
const fn = function() {
  const arguments = 'arg';
} 
```

上述代码出错了，我们从上面知道的错误消息是"在严格模式下，意外的 eval 或 arguments"。

```
const fn = function() {
  'use strict'     // "use strict" inside the function, test still fails.
  const arguments = 'arg';
} 
```

上面的代码也因为同样的原因失败了。这里只有函数在严格模式下运行，但这也是使用`arguments`的地方，所以它失败了。

```
const fn = function() {
  const arguments = 'arg';
}
'use strict'     // "use strict" NOT at the beginning of a function it gets ignored, test passes! 
```

通过了。

```
const fn = function() {
  if (true) {
    'use strict'     // "use strict" at the start of a scope, but NOT at the beginning of a function it gets ignored, test passes!
    const arguments = 'arg';
  }
} 
```

这个也通过了，因为`"use strict"`字符串不在函数的开头。一开始我很烦恼，因为我期望它在作用域的开头，即在`{`之后，但我错了。[规范也说明了](https://262.ecma-international.org/5.1/#sec-10.1.1)这一点：

> 如果函数代码以包含 Use Strict Directive 的 Directive Prologue 开头。

以人类的语言来说，这意味着函数必须以`"use strict"`字符串开头。作用域没有在那里提及。规范用非常难以理解的词语定义了另外三种情况。这些情况是 1）全局代码，2）eval 代码和 3）`Function` 代码，当它们中的任何一个以`"use strict"`字符串开始时，整个代码就会在严格模式下运行。

有了这个，我不仅了解了严格模式限制了什么，还学会了如何控制它。所有的魔法都消失了。

💡 **要启用严格模式，您可以在四种描述的方式、类或模块中使用`"use strict"`。这就是严格模式可以（手动）打开或强制打开的方式。**
