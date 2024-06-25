<!--yml

类别：未分类

日期：2024-05-27 15:22:17

-->

# 彩色函数其实挺好的

> 来源：[`www.danmailloux.com/blog/colored-functions-are-good-actually`](https://www.danmailloux.com/blog/colored-functions-are-good-actually)

# 彩色函数其实挺好的

在 Bob Nystrom 的优秀文章 [What Color is Your Function?](https://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/) 中，他认为“彩色”（异步）函数不好，因为它们创建了两个分离的代码宇宙，规定了何时何地可以调用什么。有点。

我要把这个断言颠倒过来，我要争论说，在实践中，这种限制通常是一件好事。我承认我在这里有点故意制造一个草人，因为 Bob 比我聪明得多，他的文章也包含了很多技术细节，可能会超过我即将说出的好处。让我自由发挥我的 *有点* 正确的开头，我保证接下来的内容至少会有点有趣。

## Haskell 中的 IO

如果你在网络编程圈子里待过一段时间，你可能听说过 JavaScript 的 promises 是单子（monads）或者它们是单子的。虽然快速的谷歌搜索会告诉你这并不完全正确，但这 *有点* 真实，无论如何这只是为了引入单子的一个失败的过渡。

别担心，这不是又一个关于单子的文章。我不会“说出这句话。”（开玩笑 - 单子其实只是函子范畴中的单子！）实际上，我只想讨论一个特定的单子 - Haskell 的 `IO` 单子。对于未接触过的人来说，Haskell 是一种 [*纯* 函数式编程语言](https://en.wikipedia.org/wiki/Functional_programming)，强调纯粹。在 Haskell 中，为了做像从数据库中读取、写入日志或发出 HTTP 请求这样的事情，你必须将那段代码包装在 `IO` 单子中。

"单子是什么？" 你可能会问。这是一个真正的兔子洞，老实说，现在并不重要。如果你感兴趣，我可以推荐 Arialdo Martini 的 [Monads For The Rest Of Us](https://arialdomartini.github.io/monads-for-the-rest-of-us) 系列，但你真的不需要它来继续，再次强调，单子是一个 *真正* 的兔子洞。 **就本文而言，将 `IO` 视为简单类型，它包装了函数的返回类型。** 就像 JavaScript 中的函数可以返回 `promise<string>` 一样，Haskell 中的函数可以返回 `IO String`。

让我们看一个例子！

```
If we were to try and call `someUnexpectedIOThing` from inside the top `add` function, our program would not compile. The type signature of a function with side effects is different from the type signature of a similar function without side effects. Furthermore, functions with side effects can call other functions with or without side effects, but functions without side effects can only call other functions without side effects. You can probably see where I'm going with this.
It's easy to imagine the benefits of such an approach already. If you would think that working with `IO` functions is relatively more awkward than working with equivalent non-`IO` functions, you'd be correct. The effect of this is that the `IO` portion of your program naturally tends towards being only as large as it has to be, and the non-`IO` parts tend towards being as much of the program as is possible. This is a huge win because pure functions are easy to read, easy to reason about, and easy to change.
Async/Await in C#
Let's see a similar example in a language with colored functions like C#.

```

再次，我们看到一个程序，其中的副作用是由类型的存在暗示的。在这种情况下，该类型是 `Task`，在 JavaScript 中，它可能是 `Promise`，正如我们上面看到的 - 在 Haskell 中，该类型是 `IO`。当然，`Task<int>` 和 `IO Int` 之间有重要的技术区别，但它们共享一个重要的好处。**它向程序员发出信号，表明代码内可能正在发生某种效果。**

当然，在 C# 中，编译器并不强制执行这种区别，甚至并不是所有有副作用的操作都是`async`，比如写入控制台或生成随机数，但实际上，这证明是一个相当有用的约定。

致谢

我想指出我并不是第一个注意到这一点的人。Mark Seemann 写了一篇优秀的文章 [Async as surrogate IO](https://blog.ploeh.dk/2016/04/11/async-as-surrogate-io/)，在这篇文章中，他更加严谨地比较了`IO`和 F# 的`Async`。他的文章很棒，很技术性，很发人深省，每个程序员都应该阅读它。

但是，本文并不是要这样。它更像是对这个主题的一种随意探讨，问道，“即使我们不能依赖相似性，是否仍然有价值可言？”

作为程序员，我们经常会固执于边缘情况和技术细节，但我认为承认有用的经验法则也是可以的，即使它们并不总是百分之百准确。在我们日常工作中，我们中的许多人通常更像是技工而不是科学家。如果注意到一个函数是`async`让你想到，“嘿，里面可能有副作用” - 我觉得那挺好的。 :)

```

```
