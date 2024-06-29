<!--yml

category: 未分类

date: 2024-05-27 14:49:17

-->

# CSS Mixins & Functions Explainer

> 来源：[https://css.oddbird.net/sasslike/mixins-functions/](https://css.oddbird.net/sasslike/mixins-functions/)

<main><details data-alert="note" open=""><summary>注意：</summary></details>

## 作者

Miriam Suzanne

（基于 Tab Atkins 提出的自定义函数提案）

## 简介

为了减少代码重复，确保项目中的一致性，并鼓励最佳实践，作者们通常转向第三方 CSS 预处理器（如 Sass、Less、PostCSS、Stylus 等）来定义自定义可重用的“宏”。这些通常分为两个类别：

+   **Functions** 返回 CSS *值* - 如字符串、颜色或长度。它们只能在 CSS 属性的*值空间内*使用。

+   **Mixins** 返回 CSS *声明* 或整个 *规则块*。它们只能用于 CSS 属性的*值空间之外*。

CSS 已经提供了广泛的内置函数，例如 `calc()`、`minmax()` 等等。理想情况下，自定义函数将以类似的方式工作，但会使用破折号标识以避免未来的兼容性问题。举个简单的例子：

```
@function --negative (--value) {
  result: calc(-1 * var(--value));
}

html { padding: --negative(var(--gap)); }
```

CSS 目前尚未内置混合，尽管在讨论此功能时提出了几个建议。一个简单的混合可能看起来像这样：

```
@mixin --button (--face, --text, --radius) {
  --background: var(--face, teal);
  --color: color-mix(in lch, var(--text, white) 85%, var(--background));
  --border-color: color-mix(in lch, var(--text, white) 80%, var(--background));

  @result {
    background: var(--background);
    border: medium double var(--border-color);
    border-radius: var(--radius, 3px);
    color: var(--color);
    padding: 0.25lh 2ch;
  }
}

button[type='submit'] { @apply --button(rebeccaPurple); }
button.danger { @apply --button(maroon); }
```

## 讨论

在 CSS 工作组中还有几个与此提案前置的讨论：

（如果我还没有找到更多，请 [告诉我](https://github.com/oddbird/css-sandbox/issues)。）

## 总结与目标

随着它们从（一般为命令式的）预处理器进入 CSS，功能通常会发生变化 - 采用适合声明式、客户端语言的不同功能和约束：

+   CSS 原生混合和函数与预处理器有何不同？

+   在浏览器中提供这些功能会带来什么额外的功能或限制？

从语言/实现的角度来看，混合和函数是完全不同的功能 - 它们存在于语法的不同级别，并带有不同的复杂性。如果我们同时追求这两者，我们可能希望在规范的不同级别定义它们，甚至在不同的规范中。

消除对预处理器的依赖将进一步简化 CSS 作者的维护工作，同时提供新的客户端功能：

+   将级联自定义属性传递作为参数。

+   添加媒体/支持和其他客户端条件。

我在这里的目标是探索每个功能的可能性，我们可以在它们之间重复使用语法，并且我们如何推进它们的实现。

我并不期望这对任一功能来说是最终形态，但我希望捕捉对话的状态，并推动其进展。如果这些功能被工作组正式采纳，进一步的开发可以拆分为单独的规范和问题。

## 作者兴趣

来自HTTP存档项目的一些（不完整）数据可以帮助我们了解作者当前如何使用Sass：

我还在Mastodon上进行了一项小的[调查](https://front-end.social/@mia/110833306689188274)：

> “在CSS预处理器中，您定义/使用的最常见的自定义函数或mixin是什么？”

答案包括：

+   (Functions) 将像素转换为 `rem` 单位

+   (Functions) 随机数生成器

+   (Functions) 颜色对比度

+   (Mixins) 命名常见媒体查询的简写

+   (Mixins) 从对象的forEach循环生成输出（如Sass Maps）

+   (Mixins) 可重用的组件样式

+   (Mixins) 复杂的解决方案，如滚动阴影或渐变文本

+   (Both) 流体排版设置

+   (Both) 复杂的 `calc()` 简写，适用于各种情况

其中一些可以通过CSS的声明语法来实现，无需额外的新功能。其他一些（如循环）则需要命令式的控制结构。

虽然其中一些（如 `random()` ）已经在讨论内置函数中讨论，但其他一些（如 `color-contrast()` ）可能更简单地在用户空间中解决。对于整个平台，CSSWG很难达成长期解决方案，而单个团队则更能够随着时间逐渐改变其方法。通过在单一位置（如自定义函数）中捕捉该逻辑，可以在无需对代码基础进行任何侵入性重写的情况下进行许多更改。

能够在CSS中声明这种逻辑，而不是在预处理器中，将带来几个好处：

+   *减少生成代码所需的外部依赖项* 和构建步骤

+   *减少从服务器传送的文件大小*（尽管在压缩和增加客户端处理后可能是可以忽略的）

+   *使用自定义属性作为参数*，使mixin或函数能够响应级联变化

+   *将媒体/容器/支持条件*作为内部逻辑的一部分

## 定义参数

函数和mixin都依赖于 `<parameter-list>` 语法。 `<parameter-list>` 中的每个 `<parameter>` 都由三个部分组成：

+   `<name>`（必需），是`dashed-ident`

+   `<syntax>`（默认：`*`），类似于`@property`中的`syntax`描述符

+   `<default-value>`（默认：`guaranteed invalid`），即匹配语法的任何值

在函数开头定义这三个方面（名称、类型和默认值）可能会使语法过于复杂化。我的最初提案包括特殊的类似于`@property`的描述块，以实现这一点。

自那时以来，讨论已经转向了使用逗号分隔列表的更简洁的方法。

作者只能提供名称：

```
@function --my-function(--param-a, --another-param) { … }
```

可选地，它们还可以提供一个默认值：

```
@function --my-function(
  --param-a: 1em,
  --another-param: 'this is a string'
) { … }
```

<details data-alert="note" open=""><summary>注：</summary></details>

最后，作者可以定义任何参数的语法，使用 `type()` 函数并与名称一起。这将适用于有或无默认值的情况：

```
@function --my-function(
  --param-a type(string),
  --another-param type(length): 1em
) { … }
```

## 定义函数：`@function` 规则

要定义一个自定义函数，我们需要几个信息：

+   `function-name`（必需）

+   `parameter-list`（可选 - 参见上文）

+   使用`function-rules`的一些内部逻辑

+   返回的`result`值

提议的语法（稍作调整）可能看起来像这样：

```
@function <function-name> [( <parameter-list> )]? {
  <function-rules>

  result: <result>;
} 
```

`function-name`是一个虚线标识符。如果多个函数具有相同的名称，则较高层级中的函数具有优先权，并且在给定级联层内后定义的函数具有优先权。这与其他定义名称的at-rules的行为相匹配。

定义一个预期的“返回类型”（例如`color`或`length`）可能也会很有用，这样可以在解析时进行验证。与自定义属性类似，函数的输出在计算值时仍然可能是*无效的*，但我们至少可以确保函数意图返回适合调用上下文的合适语法。

扩展上述语法，我想象在序文中重复使用`type()`函数：

```
@function <function-name> [( <parameter-list> )]? [returns type(<syntax>)]? {
  <function-rules>

  result: <result>;
} 
```

我期望`<syntax>`允许与`@property`规则的`syntax`描述符提供的CSS类型的相同子集。也许在这种情况下可以去掉语法周围的引号要求？

### 返回值

讨论了几种语法选项以返回`<result>`值，但我觉得最简单和最熟悉的可能是类似`result`或`output`的描述符。这将有助于加强函数的声明性质，因为它可以像其他声明一样处理：如果有多个声明，则使用最后一个`result`。

与自定义属性类似：

+   `<result>`可以包含任何有效的CSS值空间语法

+   此值在计算值时间为`invalid at computed value time`

由于函数存在于值空间中，`<function-rules>`将不包含任何其他（非自定义）CSS属性，因此单个`result`描述符应该很显眼。如果遇到多个结果，则最后一个结果优先（与其他描述符和属性一致）。这在下文中详细讨论。

<details data-alert="note" open=""><summary>注意：</summary></details>

### 函数规则

`<function-rules>`可以包括自定义属性声明（这些属性的作用域限定为函数内部），以及条件性at-rules（可能包含进一步嵌套的自定义属性和`result`）。元素特定的条件（如容器查询）将为调用该函数的每个元素解析。

我的假设是，在函数内定义的自定义属性在调用该函数的元素上不可用。然而，作者们显然期望从函数内部引用外部自定义属性 - 使用某种动态范围的变体和“阴影”行为。

据我所知，仅在函数定义内部有用的是自定义属性、参数/变量和条件规则。函数除了返回值外没有输出，因此嵌套选择器、内置属性和命名规则都是不必要或无意义的。我认为这些东西没有必要使整个函数无效，但应该被忽略和丢弃。

使用条件规则返回多个值的示例函数：

```
@function --sizes(
  --s type(length),
  --m type(length),
  --l type(length),
) returns type(length) {
  --min: 16px;

  @media (inline-size < 20em) {
    result: max(var(--min), var(--s, 1em));
  }
  @media (20em < inline-size < 50em) {
    result: max(var(--min), var(--m, 1em + 0.5vw));
  }
  @media (50em < inline-size) {
    result: max(var(--min), var(--l, 1.2em + 1vw));
  }
}
```

一些函数还希望从调用元素的上下文变量中获取访问权限。为了避免完全动态作用域的自定义属性，Tab建议在函数中提供第二个属性列表，这些属性应该在函数中可用：

```
@function --my-function (--arg1, --arg2) using (--var1, --var2) {

}
```

<details data-alert="note" open=""><summary>注意：</summary></details>

### 调用函数

可以从任何属性的值空间调用自定义函数，函数的名称后跟括号和逗号分隔的参数列表：

```
button {
  background: --contrast(pink, 0.7);
}
```

如果我们（最终）希望支持命名参数，最好使用熟悉的声明语法：

```
button {
  background: --contrast(--color: pink; --ratio: 0.7);
}
```

如果允许在同一函数调用中使用位置和命名参数，则常见约定是要求所有位置值位于任何命名值之前，以避免混淆：

```
button {
  background: --contrast(pink; --ratio: 0.7;);
}
```

我们需要允许广泛的参数值语法 - 包括包含逗号的值。关于如何更普遍地处理这个问题，有一个关于最佳处理方式的活跃讨论 [issue #9539](https://github.com/w3c/csswg-drafts/issues/9539)。自定义函数应使用那里商定的任何解决方案。

### 将所有内容组合在一起

将上述流体比率函数适应建议的语法：

```
@function --fluid-ratio(
  --min-width type(length),
  --max-width type(length),
) returns type(percentage) {
  --min: var(--min-width, 300px);
  --max: var(--max-width, 2000px)l
  --scale: calc(var(--max) - var(--min));
  --position: calc(100vw - var(--min));
  --fraction: calc(var(--position) / var(--scale));

  @return clamp(
    0%,
    100% * var(--fraction),
    100%
  );
}

p {
  font-size: calc-mix(--fluid-ratio(375px; 1920px), 1rem, 1.25rem);
  padding: calc-mix(--fluid-ratio(375px; 700px), 1rem, 2rem);
}
```

我们也可以考虑将`mix()`逻辑移入函数中：

```
@function --fluid-mix(
  --min-value type(length),
  --max-value type(length),
  --from-width type(length),
  --to-width type(length)
) returns type(length) {
  --from: var(--from-width, var(--fluid-min, 375px));
  --to: var(--to-width, var(--fluid-max, 1920px));
  --scale: calc(var(--to) - var(--from));
  --position: calc(100vw - var(--from));
  --fraction: calc(var(--position) / var(--scale));
  --progress: clamp(0%, 100% * var(--fraction), 100%);

  @return calc-mix(var(--progress), var(--min-value), var(--max-value));
}

p {
  font-size: --fluid-mix(1rem, 1.25rem);
  padding: --fluid-mix(1rem, 2rem, 375px, 700px);
}
```

## 定义一个mixin：`@mixin`规则

mixins不仅返回单个值，还返回整个声明和潜在的整个嵌套规则块。虽然可以重用大部分函数语法，但我们需要一种额外的方式来管理属性作用域 - 明确标记哪些规则块是内部的，哪些应该是输出的一部分。

```
@mixin <mixin-name> [( <parameter-list> )]? {
  <mixin-rules>
} 
```

再次强调，当存在多个使用相同名称的mixin时，具有该名称的最后一个mixin占优势。

### Mixin规则和输出

处理嵌套规则和输出的最简单方法是将mixin定义内部视为任何规则块嵌套上下文。我们可以放置在规则块中的任何东西都可以放置在mixin中，并且将在调用mixin时输出（首先替换任何参数）。这对于许多更简单的情况都适用：

```
@mixin --center-content {
  display: grid;
  place-content: center;
}

.page {
  @apply --center-content;

}
```

```
@mixin --clearfix {
  &::after {
    display: block;
    content: "";
    clear: both;
  }

  @supports (display: flow-root) {
    display: flow-root;

    &::after { display: none; }
  }
}

.float-container {
  @apply --clearfix;

}
```

这种方法不允许mixin包含任何仅作用于mixin本身的内部逻辑。Mixin应能够使用内部作用域的自定义属性，并且作为返回的规则块的一部分可选地*输出*自定义属性。在目前的情况下，这似乎与除自定义属性外的任何内容都无关。内置属性、选择器和at-rules仅对其*输出*有用。

鉴于这个问题特定于自定义属性，我们可以考虑一个标志，比如`!private`。该标志对于其他上下文中的自定义属性可能很有趣，但除非有兴趣，我不会沿着这条路走。或者，我们可以明确地用`@output`或`@private`规则标记内容块。

### 应用mixin：（新的）`@apply`规则

为了应用一个mixin，我们使用一个`@apply`规则：

```
@apply <mixin-name> [(<argument-list>)]?
```

`<argument-list>`语法应该理想地匹配函数参数的表示法。

当mixin解析时，mixin的输出将插入到调用apply规则的位置：

```
 .float-container {
  @apply --clearfix;
}

.float-container {
  &::after {
    display: block;
    content: "";
    clear: both;
  }

  @supports (display: flow-root) {
    display: flow-root;

    &::after { display: none; }
  }
}
```

还有一个额外的问题，关于如何处理文档顶层（不是嵌套在选择器内部）的mixin输出：

```
@apply --center-content;
```

只要有一个包装输出的选择器，这不应该是一个问题。即使该选择器只是父级引用`&`，在文档的顶层具有明确定义的行为——指的是当前`:scope`。然而，如果结果是裸声明而没有任何选择器，它们应该被丢弃和忽略。

另一个例子，来自我偶尔使用的一个Sass mixin：

```
@mixin --gradient-text(
  --from-color type(color),
  --to-color type(color),
  --at-angle type(angle),
) {
  --to: var(--to-color, teal);
  --from: var(--from-color, mediumvioletred);
  --angle: var(--at-angle, to bottom right);
  color: var(--from, var(--to));

  @supports (background-clip: text) or (-webkit-background-clip: text) {
    --gradient: linear-gradient(var(--angle), var(--from), var(--to));
    background: var(--gradient, var(--from));
    color: transparent;
    -webkit-background-clip: text;
    background-clip: text;
  }
}

h1 {
  @apply --gradient-text(pink, powderblue);
}
```

## 复杂度的层次

流行的Sass函数和mixin展示了一系列不同的输入需求，从相对静态的速记法到完全命令控制结构。

### 简单的速记法

`clearfix` mixin通常没有公开的‘参数’和内部逻辑。当调用mixin时，它每次都会输出相同的代码。这对于保持DRY代码（不要重复你自己）非常有用，

这种静态mixin最终与‘实用类’如`.clearfix`非常相似。然而，mixin仍然具有在CSS中应用而不是HTML中的优势。当与`@media`/`@container`和其他条件逻辑结合时，CSS控制的需求变得尤为重要。目前在CSS中没有办法写这段代码而不是两次定义所有自定义属性：

```
.dark-mode {
  --background: black;
  --text: white;

}

@media (prefers-color-scheme: dark) {
  html:not(.light-mode) {
    --background: black;
    --text: white;

  }
}
```

关于这种用例的大多数现有提案将条件逻辑与选择器逻辑结合在一起，以便同时定义两者。在Sass中，我们可以通过提供一个`dark-mode` mixin来修复这个问题，该mixin可以多次使用以仅仅最小重复输出相同的声明：

```
@mixin dark-mode {
  --background: black;
  --text: white;

}

.dark-mode {
  @include dark-mode;
}

@media (prefers-color-scheme: dark) {
  html:not(.light-mode) {
    @include dark-mode;
  }
}
```

使用容器样式查询在这里也可能是一个选择。它们可以有点像*mixin*，但也带有所有容器查询的限制。如果我们在根`html`元素上设置自定义属性`--mode`，我们必须在不同的元素上分配属性，而不是我们查询的元素：

```
.dark-mode {
  --mode: dark;
}

@media (prefers-color-scheme: dark) {
  html:not(.light-mode) {
    --mode: dark;
  }
}

@container style(--mode: dark) {

  body {
    --background: black;
    --text: white;

  }
}
```

这可能会导致几个问题：

+   有一些优化和特性特定于根，无法在其他元素上复制。

+   在其他组件上下文中，可能需要额外的标记。

虽然这些无参数的mixin有些常见，但是没有参数的函数要少得多，因为简单值可以捕获在变量或自定义属性中。

### 内置条件

提供没有作者可见参数但仍包含内部逻辑和条件语句的混合物也可能非常有用，例如使用`@supports`、`@media`或`@container`：

```
@mixin gradient-text {
  color: teal;

  @supports (background-clip: text) or (-webkit-background-clip: text;) {
    background: linear-gradient(to bottom right, teal, mediumvioletred);
    -webkit-text-fill-color: transparent;
    -webkit-background-clip: text;
    background-clip: text;
  }
}
```

类似这样的混合物甚至可以依赖于自定义属性而不接受显式的覆盖参数引用外部值：

```
@mixin gradient-text {
  --gradient-text-start: var(--color-primary, teal);
  --gradient-text-end: var(--color-complement, mediumvioletred);
  color: var(--gradient-text-start);

  @supports (background-clip: text) or (-webkit-background-clip: text;) {
    background: linear-gradient(
      to bottom right,
      var(--gradient-text-start),
      var(--gradient-text-end)
    );
    -webkit-text-fill-color: transparent;
    -webkit-background-clip: text;
    background-clip: text;
  }
}
```

### 接受参数

使用函数或混合物的最常见原因是能够定义根据不同输入改变输出的参数。例如，一个`darken()`函数将接受两个参数：一个颜色和一个要加深该颜色的量。

在许多情况下（如`darken()`），内部函数逻辑可以通过使用现有的CSS特性进行内联计算来表示。在这些情况下，自定义函数仍然可以提供更简洁易用的快捷方式，用于更复杂的`calc()`或相对颜色调整。

### 参数条件

一旦我们允许使用参数和条件逻辑，下一步将是允许在条件本身中使用参数。例如：

```
@mixin button(--style: outline, --shape: pill) {
  @when arg(--style: outline) {
    border: medium solid;
    color: teal;
  } @else {
    background: teal;
    color: white;
  }

  @when arg(--shape: pill) {
    border-radius: 50%;
  }
}
```

### 命令式控制流

一些用例需要更复杂的“流控制”，如循环。例如，混合使用可能根据单个原始颜色生成完整的色板。在Sass中，可能看起来像这样：

```
@use 'sass:color';
@use 'sass:math';

@mixin tint-shade($color, $name, $steps: 2) {
  --#{$name}: #{$color};

  $step: math.div(100%, ($steps + 1));

  @for $i from 1 through $steps {
    $amount: $step * $i;
    --#{$name}-t#{$i}: #{color.mix(white, $color, $amount)};
    --#{$name}-s#{$i}: #{color.mix(black, $color, $amount)};
  }
}

@mixin theme($color, $type: 'complement') {

  @include tint-shade($color, 'primary');

  @if $type == 'complement' {
    $complement: color.adjust($color, $hue: 180deg);
    @include tint-shade($complement, 'complement');
  } @else if $type == 'triad' {

  }

}

html {
  @include theme(blue);
}
```

结果输出的CSS将是：

```
html {

  --primary: blue;
  --primary-t1: #5555ff;
  --primary-s1: #0000aa;
  --primary-t2: #aaaaff;
  --primary-s2: #000055;
  --complement: yellow;
  --complement-t1: #ffff55;
  --complement-s1: #aaaa00;
  --complement-t2: #ffffaa;
  --complement-s2: #555500;

}
```

我认为在这里设定一个边界是合理的，因为CSS是一种声明性语言。添加命令式流程可能会导致执行模型上的混乱。

## 详细讨论和开放问题

### 函数的其他结果语法

我和Lea都注意到，如果作者可以依赖级联的“出现顺序”来提供“回退”返回值，这将非常有用。然而，遗憾的是，对于像自定义属性或函数这样的动态计算值时特性，这种解析时的回退是不可能的。

我最初提议使用一个at-rule语法（`@return`），认为：

+   它有助于区分最终返回的值与像自定义属性和嵌套规则这样的内部逻辑

+   结果不是一个属性，但看起来非常像一个

然而，`result`在很多方面确实像一个属性，并且有助于强化我们对声明性执行的熟悉感。虽然许多命令式语言允许“急切”地优先返回函数，CSS和其他声明性语言通常采用“后者优先”的方法。出于同样的原因，我们应该避免像`return`这样暗示评估是线性且可以被中断的积极词语。

François Remy建议设置一个与函数同名的自定义属性，并将该属性视为结果值。Lea Verou建议在序言中使属性名可定制化。

我更喜欢一种更一致和可靠的语法。我没有看到任何允许每个函数中的此功能重命名或要求作者确定该名称的实用性，或将其放入作者自定义标识名称空间中的效益。这些对我来说似乎都是邀请拼写错误和混淆的方法，没有明显的收益。

在我看来，匹配函数名称似乎过于脆弱——因为您永远无法重命名一个而不更新另一个。尽管如此，任一方法都可能有效，并提供相同的基本行为。我们可以继续对细节进行讨论。

### 将嵌套内容传递给混合器

Sass混合器的另一个常见特性是能够将嵌套内容块传递到混合器中，并使混合器将该内容放置在特定的上下文中。这似乎是CSS中可以支持的功能，但需要另一个特定于混合器的at-rule（或类似的占位符）。我暂时称之为`@nested`：

```
@mixin --media-medium {
  @media screen and (env(--small) < inline-size < env(--large)) {
    @nested;
  }
}

.grid {
  @apply --media-medium {
    padding: var(--padding-medium, 1em);
  }
}
```

预期的行为应与以下写法相同：

```
.grid {
  @media screen and (env(--small) < inline-size < env(--large)) {
    padding: var(--padding-medium, 1em);
  }
}
```

如果需要，这似乎是可以稍后添加的东西。

### 无效函数回退

遗憾的是，在这里，最后起作用的`@return`行为未能提供与级联相同的好处——在解析时可以丢弃无效声明，回退到先前声明的值。为了实现这一点，我们需要限制函数，使其成为属性中唯一的值。我认为这种权衡在我看到的用例中没有意义。

我也不确定在提供具有无效语法的参数时提供函数定义的回退值是否有意义。理想情况下，函数回退应该模仿变量回退——在调用函数的地方建立，而不是在定义函数的地方。很难看出这将适合提议的语法中的哪个位置。

一个选项是类似于`var()`的包装函数：

```
button {
  background: fallback(--contrast(pink; 0.7), black);
}
```

我们甚至可以使用现有的`var()`，但这将导致函数和自定义属性共享单一命名空间，这可能不是理想的情况。也许`first-supported()`的提议函数也是一个具有更广泛用途的选项？这可能需要更多的讨论。

### 在条件规则中使用参数

上面，我使用了一个在函数内部使用媒体查询进行条件输出的示例。作者可能希望进一步使用参数来定义媒体查询本身：

```
@function --media(
  --breakpoint: length,
  --below: length,
  --above: length
) {
  @media screen and (width < var(--breakpoint)) {
    result: var(--below);
  }
  @media screen and (width >= var(--breakpoint)) {
    result: var(--above);
  }
}
```

这是预处理器混合器的一个非常常见的用法，也是提议的内联`if()`和`media()`函数的常见用例。

据我所理解，出于与当前不允许在媒体查询条件中使用`var()`相同的原因，按上述方式实现将不可能。然而，问题特定于需要在计算值时间解决的级联值。从参数传递静态参数不应造成相同的问题。

如果我们有一种新的访问传入值的方式 - 我将使用`arg()`作为论点 - 简单的值替换应该是可能的：

```
@function --media(
  --breakpoint: length,
  --below: length,
  --above: length
) {
  @media screen and (width < arg(--breakpoint)) {
    result: var(--below);
  }
  @media screen and (width >= arg(--breakpoint)) {
    result: var(--above);
  }
}

html {

  padding: --media(40em, 0, var(--padding, 1em));

  margin: --media(var(--break, 40em), 0, 1em);
}
```

在上面的例子中，`padding`声明将是有效的，因为可以将静态值传递给媒体查询`arg()` - 但是`margin`声明将会失败，因为它向媒体查询条件提供了自定义属性。

我不清楚参数是否需要提前明确标记任何原因？如本文所述，这将由函数作者来记录和传达哪些参数可以接受级联变量，哪些不能。

### 参数条件和循环

对于混合物和函数，基于传入的参数进行条件判断可能很有用。例如，我们可能希望传入几个已建立的关键字之一，并根据使用的关键字返回不同的值：

```
@function --link(
  --theme: *;
) {
  @when (arg(--theme): light) {
    result: env(--link-light);
  } @else {
    result: env(--link-dark);
  }
}
```

我不清楚提议的`@when`/`@else`功能是否可以适应这种用例，或者是否需要一个类似流控制的独立集合。

同样地，正如我们在前面的色调示例中看到的那样，循环遍历一组重复次数（for循环）或一组项目列表（each循环）可能会很有用。

虽然这些对作者来说是有用的功能，但它们并不是混合物或函数的初始实施所必需的（或依赖的）。它们感觉像是可以很好地组合在一起的独特功能。

### 我们可以允许`<calc-sum>`语法吗？

此问题由[Brandon McConnell](https://github.com/w3c/csswg-drafts/issues/7490#issuecomment-1256880496)在‘声明性自定义函数’问题中提出（参见第5点，尽管它与递归无关）。目标是提供可以接受原始calc表达式的自定义函数，而不必显式地包装在嵌套的`calc()`函数中，类似于其他数学函数的工作方式：

```
.item {
  width: min(100% - 1em, 30em);
}
```

一方面，自定义属性替换使得捕获表达式变得微不足道，并且稍后可以在`calc()`函数内调用它们。这已经可以工作：

```
html {
  --l: 100% - 50%;
  background: hsl(0deg 100% calc(var(--l)));
}
```

要进一步发展，我们需要将`<calc-sum>`语法公开为作者可以使用的有效语法。

也许考虑到对参数特别有用的其他语法/类型也是值得的，或者更广泛地用于属性注册。如果这些列表能保持对齐，那对我来说似乎是理想的。

### `@extend`怎么样？

在Sass中，没有参数的混合物也与`@extend`功能重叠，用于组合相关类 - 其中一个作为另一个的‘扩展’。在大多数情况下，这与没有参数的混合物具有相同的预期结果：

```
 .error {
  border: thin solid maroon;

  &:hover {
    background-color: #fee;
  }
}

.error--serious {
  @extend .error;
  border-width: thick;
}

@mixin error {
  border: thin solid maroon;

  &:hover {
    background-color: #fee;
  }
}

.error--serious {
  @include error;
  border-width: thick;
}
```

不同之处在于类定义可以从不同样式表中的多个规则块编译，而混合物通常有一个集中的定义。这是Sass中扩展变得不太常见的部分原因 - 很难理解它们的影响。目前来看，我认为混合物可以提供类似的功能而不会带来同样的复杂性。

如果我们有兴趣在某个时候探索`@extend`，Tab已经撰写了一个[非官方草案规范](http://tabatkins.github.io/specs/css-extend-rule/)，我们可以依此构建。

### 函数可以链式调用，或者调用自身吗？

我期望应该可以将函数/混合物调用链在一起。一个生成主题的混合物应该能够在内部引用一个单色生成混合物或函数。

对我来说，递归函数调用是否可能或必要不太清楚。循环的形式可能需要使用递归，但我不确定它们有多核心。在级别1中，这似乎不是一个功能要求。

### 基于关键帧的混合物用于插值数值？

最近围绕[在断点之间插值数值](https://github.com/w3c/csswg-drafts/issues/6245#issuecomment-1715416464)进行了大量讨论，例如响应式排版。在概念上，动画关键帧很适合定义涉及的步骤 - 但在这种情况下，结果在技术上并不是动画的，而且插值数值理想情况下不应该移动到动画的起点。

为了避免这种情况，最新的提案涉及一个新属性（暂定为`interpolate`），该属性将接受关键帧名称和时间线，然后‘原地扩展’以表示引用的`@keyframes`规则中的声明。

```
@keyframes typography {
  from {
    font-size: 1.2em;
    line-height: 1.4;
  }
  to {
    font-size: 3em;
    line-height: 1.2;
  }
}

h2 {

  interpolate: typography --container-size ease-in;

  font-size: ;
  line-height: ;
}
```

Alan Stearns在对话中指出，这是一种非常类似混合物的行为，并建议将关键帧视为混合物的一种现有形式，而不是一种新的属性。鉴于上述关键帧，我们可以考虑类似以下的语法：

```
h2 {

  @apply typography(--container-size; ease-in);

  font-size: ;
  line-height: ;
}
```

如果这样会使混合物命名空间混乱，另一种方法可能是要求使用破折号标识混合物名称，并提供一些内置混合物，比如：

```
h2 {

  @apply keyframes(typography; --container-size; ease-in);

  font-size: ;
  line-height: ;
}
```

## 先前的艺术

### `@apply`规则（已废弃）

<details data-alert="note" open=""><summary>链接：</summary></details>

曾经有一个计划，即使用`@apply`规则，使自定义属性作为混合物的一种形式。由于几个相关的原因，该提案被放弃了：

+   自定义属性是值级别的语法，而混合物是声明级别的

+   混合物定义在级联中传递是没有意义的

避免这些问题并不困难。我是从以下前提出发工作的：

+   函数和混合物都应该在全局范围内*定义*，而不依赖于级联中的任何元素感知方面。

+   与例如`@keyframes`类似，函数和混合物的定义仍然会使用全局级联特性如*层次*和*出现顺序*来解决名称冲突。

+   函数应用于*值*空间，而混合物应用于*声明*空间。

### 容器样式查询（部分实现）

<details data-alert="note" open=""><summary>链接:</summary></details>

`@container`的`style()`功能有时可以用于近似混合行为。近期有几篇关于这种方法的[帖子](https://front-end.social/@chriscoyier/110821892737745155)和[文章](https://chriskirknielsen.com/blog/future-themes-with-container-style-queries/)。然而，样式查询与其他容器查询共享限制：*我们不能样式化正在查询的容器*。

容器查询被设计为一种*条件选择器*机制，用于响应上下文变化。祖先/后代限制对于浏览器分离在给定元素上的选择器匹配与值解析是必需的。

然而，*混合物不会改变选择*，它们只是‘打包’现有的CSS规则和声明以便重复使用。理想情况下，这两个功能应该能够很好地配合，以便上下文条件可以改变传递给特定混合物的参数。

### 自定义属性（已实现）

<details data-alert="note" open=""><summary>链接:</summary></details>

我们还可以使用自定义属性来近似一些基本的混合物和函数。尽管这些技巧可能很有用，但它们涉及到显著的复杂性、注意事项和限制：

+   每个‘函数/混合物’和‘参数’都是一个自定义属性，每个元素只能有一个解析后的值。

+   在计算值继承之前，函数/混合物中的参数被替换，因此逻辑必须在每个需要重新计算结果的元素上定义。

### 预处理器中的混合物和函数

<details data-alert="note" open=""><summary>链接:</summary></details>

除了参数外，Sass混合物还可以接受*内容块*。来自[文档](https://sass-lang.com/documentation/at-rules/mixin/#content-blocks)的一个例子：

```
@mixin hover {
  &:not([disabled]):hover {
    @content;
  }
}

.button {
  border: 1px solid black;
  @include hover {
    border-width: 2px;
  }
}
```

这可能也是CSS混合物的一个有用特性。这对于创建命名条件的用例是必需的。该用例也可能通过建议的`@when`规则和‘自定义媒体查询’功能来解决。

Sass提供了一些内置的核心函数，但（到目前为止）并未提供核心混合物。因此，HTTP归档报告列出了几个常用的内置函数（`if()`和`darken()`），但只列出了最常用的自定义混合物名称（`clearfix`）。

### 现有的自定义函数提案

2022年7月，Johannes Odland在CSS工作组问题跟踪器中提出了‘[声明性自定义函数](https://github.com/w3c/csswg-drafts/issues/7490)’。自那时以来，该提案经历了几次修订和更新。

该线程中当前（2023-08-08）的提案建议如下：

+   函数将在变量替换的同时解析。

+   使用CSSOM‘语法’定义的函数参数可以在解析时进行验证（类似于`@property`注册的变量）。

+   这将是更完整功能的Houdini API特性的声明版本

还有几个示例用例，比如这个用于流体排版的函数：

```
@custom-function --fluid-ratio(
  --min-width,
  --max-width
) {
  result: clamp(
    0%,
    100% * (100vw - var(--min-width)) / (var(--max-width) - var(--min-width)),
    100%
  );
}

p {
  font-size: mix(--fluid-ratio(375px, 1920px), 1rem, 1.25rem);
  padding: mix(--fluid-ratio(375px, 700px), 1rem, 2rem);
}
```

<details data-alert="warn"><summary>数学函数中的单位划分：</summary></details>

或用于生成棋盘背景图像的函数：

```
@custom-function --checkerboard(--size) {
   result: linear-gradient(
        45deg,
        silver 25%,
        transparent 25%,
        transparent 75%,
        silver 75%
      )
      0px 0px / var(--size) var(--size),
    linear-gradient(
        45deg,
        silver 25%,
        transparent 25%,
        transparent 75%,
        silver 75%
      )
      calc(var(--size) / 2) calc(var(--size) / 2) / var(--size) var(--size);
}

.used {
  background: --checkerboard(32px);
}
```

对于这些用例，自定义函数可以是将参数插入现有函数（如`calc()`）的简单包装器。Tab Atkins建议，这种仅数学版本可能是最简单实现的方式。虽然这可能是一个有用的第一步，但很快就无法满足我所见到的用例。我更愿意从更完整功能的方法开始，然后逐步朝着可实现的一级实现方向前进。

除了语法上的一些琐碎问题外，线程中还有几个更为开放的问题：

+   作者是否可以为无效参数提供回退输出？

+   在函数定义中包含默认参数值是否有帮助？

+   函数作者是否可以定义内部作用域的自定义属性？

+   作者是否可以在函数逻辑内使用条件性at-rules？

+   函数能否公开接受裸算术计算（不使用`calc()`语法）的参数，类似于`clamp()`等？

+   函数能否执行递归函数调用？

+   函数能否使用命名（而非位置）参数进行调用？

我希望在这个提议的基础上进行扩展，并探索其中的一些问题。

## 致谢

这个提议基于一个[现有讨论](https://github.com/w3c/csswg-drafts/issues/7490)，得到了以下几位的参与：

+   Johannes Odland

+   David Baron

+   Brian Kardell

+   Tab Atkins-Bittner

+   @jimmyfrasche

+   Brandon McConnell

+   Lea Verou

在路上，我还整合了以下几位的反馈：

+   Nicole Sullivan

+   Anders Hartvoll Ruud

+   Rune Lillesveen

+   Alan Stearns

+   Yehonatan Daniv

+   Emilio Cobos Álvarez

+   François Remy

+   Steinar H Gunderson

+   Matt Giuca

## 待办事项

</main>
