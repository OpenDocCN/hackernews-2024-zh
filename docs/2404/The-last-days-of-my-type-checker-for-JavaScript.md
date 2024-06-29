<!--yml

category: 未分类

date: 2024-05-27 13:28:39

-->

# 我的JavaScript类型检查器的最后几天

> 来源：[https://jsmonk.github.io/2024-01-29-last-days-of-hegel/](https://jsmonk.github.io/2024-01-29-last-days-of-hegel/)

**TL;DR** 去年，我开始开发使用[Rust](https://www.rust-lang.org/)编写的[JavaScript静态类型检查器的2.0版本](https://hegel.js.org/)，但不幸的是，如今，[TypeScript](https://www.typescriptlang.org/)获胜。严格类型检查JavaScript的想法不如2019年流行。此外，网上还有一种将更严格的类型引入web的方法。因此，我失去了对这样一个工具有观众的信心，并决定关闭项目。

## 简而言之

2019年初，我在一个名为[WookieeLabs](https://github.com/wookieelabs)的小型外包公司工作。在大多数项目中，我们更喜欢使用[Flow](https://flow.org/)而不是[TypeScript](https://www.typescriptlang.org/)。有许多好处，比如[nominal classes](https://flow.org/en/docs/lang/nominal-structural/#toc-classes-are-nominally-typed)，[type variances](https://flow.org/en/docs/lang/variance)，[opaque types](https://flow.org/en/docs/types/opaque-types/)，[与React的平滑集成](https://flow.org/en/docs/react/)。但主要原因是，2019年的Flow只是带有类型的JavaScript。

但是2019年Flow类型推断的工作方式并不理想：

```
const id = x => x;
const a = id(2); // number | string
const b = id("string"); // number | string 
```

您可以在[此处](https://flow.org/try/#1N4Igxg9gdgZglgcxALmAXwDTggEwKYrZQDOALgARw7kC85AHrQHwMDcAOlJCRQIa2UcACgBMASg5doZcgCMBVIexBkATnCgJlEkFgBueVcTjRCegAwA6AIwAOAKw2QaIA)尝试此示例

因此，Flow分析了函数的使用情况，并基于此推断了函数的类型。在上面的示例中，我们向`id`函数提供了一个数字和一个字符串，所以根据使用情况，工具推断出`id`的类型为`(x: number | string) => number | string`。

然而，Flow团队通过在工具知道不会有本地类型上下文的地方要求注释来显著改进了此类行为。您可以在[这里](https://medium.com/flow-type/local-type-inference-for-flow-aaa65d071347)阅读相关信息。

同时，有关新的“语言”（[OCaml](https://ocaml.org/)的语法）称为[ReasonML](https://reasonml.github.io/)（今天也有一个名为[ReScript](https://rescript-lang.org/)的替代项目）也被大肆炒作。

我在想，如果我们把[Hindley-Milner类型推断](https://en.wikipedia.org/wiki/Hindley%E2%80%93Milner_type_system)的强大和简单的Flow类型语法结合起来，会发生什么。因此，我作为大学学士项目创建了[Hegel](https://hegel.js.org/)并开始在一些会议上讨论它。

你可以在[这里](https://hegel.js.org/docs#benefits-and-disadvantages-over-typescript)和[这里](https://hegel.js.org/docs#benefits-and-disadvantages-over-flowjs)查看与 TypeScript 和 Flow 的主要区别。

不过，我最喜欢的一个例子是描述这个工具提供的功能是[这个](https://hegel.js.org/try#GYVwdgxgLglg9mABAEwKYGdUCcYEMA2MAXqgKqZYAU6UOYA5jMDKsudgFLoICUiA3gChEiCAhqIAtrgCeAIzIVEAXkQcAygHkAcgDoADriyZqtGAyYs2FLrwDcwxE0SVHIqDP2o4wKbIXsWCrKqgBEcHIAVqjQoYgAZPFufvKK2IgAhCGIYCD4+AlJIiKhYLiSqHHmKQFKickeXj41aVi6ZRXBYTR09KGOfELFiFioUCBYSNKpgQ4iAL6OUAAWWHAA7jmomwAqnqgAolhrVKEACmsAbjBoyIgUeIQkdyBKMOhOYJcENxmhPA5FoJaDIBI4xGAJMA4HAVCgMNhHsRWpRQgAWABM-0BolwUAgyxcqEGgkEiyAA)。

```
// The type of the function is: (string) => { name: string, ... } throws SyntaxError | TypeError
function deserializeUser(stringifiedUserJson) {
  /* the result type is unknown because there is no `any` type */
  const maybeUser  = JSON.parse(stringifiedUserJson); /* it throws a SyntaxError, so it was added to the type signature */
  if (
    typeof maybeUser === "object" && maybeUser !== null       // a proof that `maybeUser` is an object
    "name" in maybeUser && typeof maybeUser.name === "string" // a proof that there is a property called `name` with `string` type
  ) {
    return maybeUser; // the result type is { name: string, ... }
  }
  throw new TypeError("Provided serialized user is invalid!");
}

try {
  /* the type of `foo` is { name: string, ... } */
  const foo = deserializeUser("42");
} catch (e) {
  // the type of `e` is SyntaxError | TypeError | unknown
} 
```

## 还有一些困难。

因为大部分时间我是核心模块的唯一贡献者，全职工作并继续攻读硕士学位并没有给我足够的时间去做更多的贡献，所以问题和 bug 非常多（至今如此），需要修复，因此开发时间很长。

另外，参加本地会议并没有帮助我找到很多新的贡献者。

> 我要感谢所有支持过我的人，提出了激动人心的想法，并帮助我准备了不同的示例。一些人还帮助我修正了文档中的语法错误，审查了所有的章节，并设置了 CI/CD。你们都是不可思议的。

在某个时刻（大约在2021年3月左右），我因为所有的COVID-19限制和关于乌克兰与俄罗斯即将发生战争的谣言而感到困扰。因此，我和妻子在战争开始前两个月搬到了荷兰。

> 我还要向[JetBrains](https://www.jetbrains.com/)表示衷心的感谢。你们不仅让我有机会参与如此激动人心的项目，而且真正拯救了我和我妻子的生命。

过了一段时间，我开始重新设计核心系统，以支持增量编译（以提高分析速度）和中间表示（以支持包括 TypeScript 语法在内的多种语法；此前该工具直接使用 AST 进行工作）。

但在那段时间里，我感觉自己像是在试图赶超一辆法拉利，而社区对这样一个项目的兴趣已经微乎其微。

## 对项目的反思。

> 到今天为止，我只有[State of JavaScript 2022](https://2022.stateofjs.com/en-US/)的结果，一旦[State of JavaScript 2023](https://stateofjs.com/en-US)的结果出来，我会更新数据。

这些事情让我失去了继续工作的信心：

+   TypeScript 的开发者们正在打造一门优秀的语言，并在不断努力改进，因此它如今是 JavaScript 最受欢迎的替代品（98.9%的[问题](https://2022.stateofjs.com/en-US/other-tools#javascript_flavors)回答者使用 TypeScript，最接近的竞争对手是 Elm，其结果为2.3%）。

+   JavaScript作为一项技术和社区已经非常成熟，因此新工具、框架和库只占据了市场的一小部分。[这里](https://2022.stateofjs.com/en-US/other-tools#runtimes)你可以看到[Deno](https://deno.com/)（2019年其中一个Deno原型的演示[已经呈现](https://www.youtube.com/watch?v=M3BM9TB-8yA)）与[Node.js](https://nodejs.org/en)的使用情况比较，或者[框架使用情况的比较](https://2022.stateofjs.com/en-US/libraries/front-end-frameworks#front_end_frameworks_experience_linechart)，在那里我们可以看到像[Solid.js](https://www.solidjs.com/)或[qwik](https://qwik.builder.io/)这样的新框架分别占有6%和2%的市场份额。

+   基于我与不同开发者的沟通，我意识到TypeScript改变了开发者对JavaScript中类型的看法。在2019年，人们认为类型是限制开发更安全软件（减少错误或意外运行时错误）的规则，但今天，人们更多地考虑使用类型来描述**任何**（甚至不那么安全的）软件。所有这些花哨的[模板文字类型](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html)，[条件类型](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)，[断言函数的未经检查的主体](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#assertion-functions)和[类型断言](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates)，当然还有[任何](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#any)，为人们提供了一个奇妙的工具，可以用类型覆盖现有的代码，但不强制重新设计它以使其更安全。

+   我看到网页上增加语言多样性的其他机会（文章中进一步讨论）

因此，基于这些原因，我决定关闭项目[Hegel](https://hegel.js.org/)。

## 其他项目和其他机会

去年，我意外地意识到作为一项技术的[wasm](https://webassembly.org/)已经完成了一项如此庞大而精彩的工作。像[GC proposal](https://github.com/WebAssembly/gc/blob/main/proposals/gc/Overview.md)和[Exception handling](https://github.com/WebAssembly/exception-handling/blob/main/proposals/exception-handling/Exceptions.md)这样的提案帮助将更多具有严格类型系统的编程语言带到网络上，这些语言已经为人们所熟悉，并且拥有成熟的构建工具、库和框架生态系统，但也有[Component Model](https://github.com/WebAssembly/component-model)提出了解决方案来使它们之间可以互操作，这意味着可以在它们之间共享广泛的生态系统的部分。

因此，这也是我决定加入[Kotlin/Wasm](https://kotlinlang.org/docs/wasm-overview.html)团队，并大部分时间专注于帮助将令人惊叹的[Kotlin](https://kotlinlang.org/)及其生态系统引入网络的原因之一。

假设你仍在寻找 JavaScript 项目的分析工具。那么，你可以关注一个很酷的项目，名为[Ezno](https://github.com/kaleidawave/ezno)，它提供严格的类型检查，并且（因为它是一个完整的编译器）专注于优化你的代码。[Ben](https://twitter.com/kaleidawave)正在做一份令人难以置信的工作。
