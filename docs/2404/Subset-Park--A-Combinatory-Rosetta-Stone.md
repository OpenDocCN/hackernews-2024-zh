<!--yml

类别：未分类

日期：2024-05-27 13:24:19

-->

# Subset Park：一个组合式罗塞塔石

> 来源：[https://blog.zdsmith.com/posts/a-combinatory-rosetta-stone.html](https://blog.zdsmith.com/posts/a-combinatory-rosetta-stone.html)

2024-04-14

在[组合式编程](./combinatory-programming.html)中，我们试图为日常程序员提供各种有用的组合器的激励示例。

那篇文章的目的是尽可能从其上下文中提取组合器的基本概念，并将某些特别有用的组合器呈现为几乎任何编程语言中可调用的高阶函数，适用于大多数编程风格。当然，在实践中，隐式形式——我们可以使用组合器函数实现的编程风格——在某些语言和某些环境中比其他情况更合适，并且最能与编程环境的其他特性（如丰富的纯函数、一级函数、部分应用和简洁的组合器语法等）优雅地结合。

在这个补充中，我们可以更深入地探讨其中一些例子；在有用时，用多种风格表达相同的逻辑，并有时评论不同风格或语言的特性及其产生的影响。

## identity

```
const maybeAbs = n => shouldNormalizeNegatives ? Math.abs(n) : n
return nums.map(maybeAbs) 
```

在这里，`maybeAbs` 是一个函数，它根据 `shouldNormalizeNegatives` 的值调用 `Math.abs` 或直接返回 `n`。这是最显式的方式，通过名称直接三次引用 `n`，但也是最容易阅读的方式。

```
const maybeAbs = shouldNormalizeNegatives ? Math.abs : identity 
```

```
(def maybe-abs (if should-normalize-negatives math/abs identity))
(map mayb-abs nums) 
```

JavaScript 和 Janet 版本的行为相同：我们根据条件的值将绝对值函数或 `identity` 绑定到我们的新符号上。在后续示例中，我们将看到 `identity` 的更少琐碎的用法。

### 一个条件组合器

```
 is_between_negative_3_and_0 =: (<&0)*.(>&_4)
    maybe_abs =: [`| @. is_between_negative_3_and_0
    maybe_abs _5 _4 _3 _2 _1 0 1 2 3
_5 _4 3 2 1 0 1 2 3 
```

虽然我们还没有把它放入我们的个人组合器库中，但我们可以想象一个函数`agenda`，它的行为像这样：

```
function agenda(p, f, g) {
    return x => {
        if (p(x)) {
            return f(x)
        } else {
            return g(x)
        }
} 
```

换句话说，我们可以想象一个简单的组合器，让我们隐式地表达条件逻辑。J 的 Agenda `@.` 就是这样一个函数。在上面稍显愚蠢的 J 示例中，我们能够重现更接近 JavaScript 示例的行为：而不是使用条件来选择函数 `f` 或 `g`，然后将其应用于某些值，我们的 J 代码将一个动词 `between_negative_3_and_0` 作为参数传递给 `maybe_abs`，然后对参数的每个元素调用它。因此，我们可以在单次调用中同时使用 *f* 和 *g*。在这个例子中，`identity` 被拼写为`[`。

## 组成

```
const cleanData = datum => removeUndefineds(coalesceNaNs(datum))
const cleanedData = dirtyData.map(cleanData) 
```

对于组合的函数（与上述最小的`identity`示例相反），显式代码很少比隐式代码有更多优势。我们必须两次引用`datum`，所以：

+   我们必须引入一个名字；

+   这并不是很短。

但是，除此之外，显式版本在结构上较为复杂。嵌套的函数调用很难阅读。

```
const cleanData = compose(removeUndefineds, coalesceNaNs) 
```

```
(def clean-data (comp remove-undefineds coalesce-nans)) 
```

在两个隐式版本中，我们避免了不必要的命名，同时获得了更简单的结构，更少的嵌套。

```
clean_data =: remove_undefineds @ coalesce_nans 
```

在 J 中，函数组合是中缀运算符。

### 绝对差异

```
const absoluteDifference = (x, y) => Math.abs(x - y)
return absoluteDifference(9, 13) // => 4 
```

```
const absoluteDifference = compose(Math.abs, _.subtract) 
```

在这种情况下，显式代码要求我们绑定`x`和`y`；在像减法这样直观透明的情况下，这似乎特别不必要。

```
(def absolute-difference (comp math/abs -))
(absolute-difference 11 24) # => 13 
```

Janet 代码还利用了`-`作为普通函数的事实，而不是中缀运算符，而 JavaScript 则必须依赖 Lodash 的`subtract`来获得执行减法的一级函数。

```
absolute_difference =: |@:- 
```

J 代码是函数组合的另一个直接应用。在这种情况下，但是，减法与绝对值的组合被拼写为`|@:-`，这比变量名`absolute_difference`更短。

数组语言的从业者会认为这完全消除了将动词绑定到名称的必要性；为什么在每次重复拼写时需要将其分配给变量名，而不是使用更多字符？

一个可能的答案是：理解`absolute_difference`的语义比`|@:-`更为直接。我并不是一个经验丰富的 J 程序员，所以我不能支持或反对其中任何一种方式。

### 管道

值得一提的是，函数组合传统上是“向左”编写的；也就是说，最内部的函数是最后一个参数，而最后应用的函数是参数列表中的第一个。这与传统的函数调用语法清晰地映射——`f(g(x))`中的`f`和`g`顺序与`(compose f g)`相同——但读起来更难，因为应用的顺序与语法的顺序不同。

许多较新的语言具有语法或标准库函数，可以有效地以反向执行函数组合，以便可以按与函数应用相同的顺序读取对“组合子”的调用。

例如，在 Janet 中，我们可以原生地编写

```
(-> dirty-data (coalesce-nans) (remove-undefineds)) 
```

这不是一个适当的组合子，因为它不是一个高阶函数。这是特定隐式控制流形式的语法；它表示对`dirty-data`这两个函数的实际应用。

尽管如此，我们可以想象一个所谓的`pipe`，它的行为与`compose`完全相同，但顺序相反，这对那些习惯于`->`及其同类的程序员可能更为舒适。

## 应用

```
const basesAndExponents = [[0, 1], [1, 2], [3, 5], [8, 13]]
return basesAndExponents.map(([base, exponent]) => base ** exponent) // => [ 0, 1, 243, 549755813888 ] 
```

如果缺少`apply`，我们需要显式解构参数值的两个元素，然后将它们传回给`Math.pow`。这相对冗长，但仍然可读。如果我们不能使用解构，我们将被迫编写

```
args => {
    const base = args[0]
    const exponent = args[1]
    return base ** exponent
} 
```

这开始感觉非常笨拙。

```
return basesAndExponents.map(apply(Math.pow)) 
```

```
(def bases-and-exponents [[0 1] [1 2] [3 5] [8 13]])
(map (apply math/pow) bases-and-exponents) 
```

在显式示例中，我们利用了 JavaScript 的中缀指数运算符`**`。在含蓄示例中，我们使用它们各自语言的标准库`pow`函数，以便能够将一级参数传递给`apply`组合子。

```
 exponentsAndBases: (0 1;1 2;3 5;8 13)
  pow: */# 
  pow .' exponentsAndBases
1 2 125 815730721 
```

虽然 K 语言并非真正设计为 tacit 编程，但作为数组家族的一员，它仍然具有某些特性，使得某些 tacit 形式成为编写程序的最惯用方式。其中之一是`apply`被拼写为`.`。这里的`.'`意味着“逐个应用”，即它将`pow`应用于`exponentsAndBases`列表中的每一对。

在这里我们使用了反转形式`exponentsAndBases`，因为 tacit 形式`pow`本身没有被反转，它期望首先是指数。

`*/#`本身就是`compose`的一个例子；作为两个参数的函数，它将`#`应用于它们，创建一个`exponent`长度的`base`数组，然后在它们上进行`*`减少。在 K 中，不像 J，函数组合不是一个中缀运算符；它通过简单地并置两个动词`#`和`*/`来实现。

在 JavaScript 和 Janet 中，并置并不是非常有意义的。另一方面，我们将在稍后看到，J 极大地依赖并置，但将更复杂的组合器语义分配给该语法。

## 翻转

```
const exponentsAndBases = [[0, 1], [1, 2], [3, 5], [8, 13]]
return exponentsAndBases.map(([exponent, base]) => base ** exponent) // => [ 1, 2, 125, 815730721 ] 
```

```
return exponentsAndBases.map(apply(flip(Math.pow))) // => [ 1, 2, 125, 815730721 ] 
```

```
(def exponents-and-bases [[0 1] [1 2] [3 5] [8 13]])
(map (apply (flip math/pow)) exponents-and-bases) 
```

### 无`apply`的翻转

`flip`的最小示例包括使用`apply`，因为这是处理由多个有序值组成的数据最自然的方式，其中顺序必须更改。

然而，我们也可以想象到我们的组合器接受变参函数的情况，使得顺序成为问题。例如：

```
function explicit(x, y) {
    return [x ** y, y / x]
}
return explicit(2, 3) // => [ 8, 1.5 ] 
```

```
const tacit = recombine(Array.of, Math.pow, flip(_.divide)) 
```

在这个稍微人为的例子中，我们既想把某个`x`带到`y`，又想将`y`除以`x`。在一个`recombine`调用的`g`和`h`期望不同顺序的参数的情况下，我们可以用`flip`来处理其中一个。

### 局部应用下的翻转

到目前为止，我们已经排除了*部分函数*的讨论；在实践中，部分应用的易用性使得更多的 tacit 编程更方便。

```
(def double (partial * 2)) 
```

在 Janet 中，使用内置的`partial`函数，我们可以轻松地使用部分应用来定义`double`；我们绑定第一个参数并生成一个新函数，该函数接受单个参数并将其乘以 2。

```
(defn square [n] (math/pow n 2)) 
```

如果我们想要根据`math/pow`定义`square`，同样的技术并不天然适用：在这种情况下，我们想要绑定的参数是第二个参数。

```
(def square (partial (flip math/pow) 2)) 
```

在这种情况下，我们可以通过使用`flip`在 tacit 风格中工作；现在我们想要绑定的参数是函数的第一个参数，因此我们可以直接将这个反转的函数传递给`partial`。

### 局部应用下的翻转（2）

另一个示例来自将[bagatto](https://git.sr.ht/~subsetpark/bagatto/commit/1f10e88)重构为 tacit 风格。要重构的是一个高阶函数，它接受某个属性名称并返回一个调用通过获取该属性来调用`sort`的函数。

```
(defn attr-sorter
  [key &opt descending?]
  (defn by [x y] ((if descending? > <) (x key) (y key)))
  (fn [items] (sort items by))) 
```

在 tacit 风格中，这变成了：

```
(defn attr-sorter
  ...
  (partial (flip sort) by)) 
```

通过在`sort`上调用`flip`，我们获得一个函数，可以轻松地将我们的新`by`应用为第一个参数。

## 重组

```
const mean = xs => _.sum(xs) / xs.length
return mean([0, 1, 1, 2, 3, 5, 8, 13]) // => 4.125 
```

```
const mean = recombine(_.divide, _.sum, _.size) 
```

```
(def mean (recombine / sum length)) 
```

```
mean =: +/ % # 
```

在 J 的例子中，我们可能看到该语言向以点式编程倾斜的最明显的例子。J 结合了一些语法特征，使得动词的简单并列，也就是直接将语法动词紧挨在一起而不使用运算符，触发应用组合器的行为。对于 `mean`，我们看到创建了一个所谓的 3-训练，`f g h`，创建了一个行为类似于 `recombine(g, f, h)` 的新动词，J 中称为 *fork*。

### 最小-最大

```
const minMax = xs => [_.min(xs), _.max(xs)]
return minMax([1, 1, 2, 3, 5, 8, 13]) // => [ 1, 13 ] 
```

```
const minMax = recombine(Array.of, _.min, _.max) 
```

```
(def min-max (recombine array min-of max-of)) 
```

```
 min_max =: <./ , >./
   min_max 0 1 1 2 3 5 8 13
0 13 
```

### 加或减

```
const plusOrMinus = (x, y) => [x + y, x - y]
return plusOrMinus(13, 8) // => [ 21, 5 ] 
```

```
const plusOrMinus = recombine(Array.of, _.add, _.subtract) 
```

如上面的绝对差异一样，我们看到两个参数函数在显式处理时会产生相当多的噪音。明确的 `plusOrMinus` 接受两个参数，其中没有一个具有特别有意义的名称（根据 Lodash 文档，添加操作应命名为 `augend` 和 `addend`；减法下应命名为 `minuend` 和 `subtrahend`——但当两种操作同时出现时，我们应该如何称呼它们？），每个参数都需要两次引用。

```
(def plus-or-minus (recombine array + -))
(plus-or-minus 13 8) 
```

```
 plus_minus =: - , +
   1 plus_minus 2
_1 3 
```

在我们的 JavaScript 和 Janet 示例中，值得注意的是，像 `minMax` 这样的一元重新组合函数和像 `plusOrMinus` 这样的可变参数函数之间的区别纯粹是语义上的：当 `g` 和 `h` 期望的参数数量不同时，应用的语法不会改变。

另一方面，在 J 的例子中，我们的应用语法默认是中缀；因此，虽然我们的 `min_max` 是写在其参数之前的，但我们的 `plus_minus` 是写在其两个参数之间的，语法上与其组成的 `g` 和 `h` 完全相同。

### 项目获取器

另一个来自 [bagatto](https://git.sr.ht/~subsetpark/bagatto/commit/1f10e88) 的例子展示了使用 `recombine` 进行现实世界的重构，以及对 `constant` 的非平凡使用。

`item-getter` 是一个接受关键字路径的函数，应返回一个函数，给定两个参数，将检索第二个参数中该路径处的属性。

一个简短的测试以展示预期的行为：

```
(deftest item-getter
  (let [item {:foo {:bar :baz}}
        getter (bagatto/item-getter [:foo :bar])]
    (is (== :baz (getter {} item))))) 
```

明确版本：

```
(defn item-getter
  [path]
  (fn [site item] (get-in item path))) 
```

以点式风格：

```
(defn item-getter
  [path]
  (recombine get-in right (constant path))) 
```

至少在所有名称都拼写出来的情况下，点式版本*并不更短*。如果我们仍然试图记住 `recombine` 的行为，这样阅读起来也不容易。

但是一旦我们稍微内化了 `recombine` 的行为，点式示例就有了不引入任何额外名称的优势。我们可以看到它返回一个函数，在其第二个参数上调用 `get-in` 和 `path`。

### 总体标准偏差

```
function sum(xs) {
    return xs.reduce((n, m) => n + m) 
}
function std(xs) {
    const mean = sum(xs) / xs.length
    const squaredDifferences = xs.map((x - mean) ** 2)
    const meanSquares = sum(squaredDifferences) / xs.length
    return Math.sqrt(meanSquares)
}
return std([0, 1, 1, 2, 3, 5, 8, 13]) // => 4.136348026943574 
```

```
pow: {*/y#x}
mean: %/ (+/;#:) @\:
std: % mean @ pow[;2]' -/ 1 mean\ 
```

在这个例子中，我们展示了偏函数在以点式风格编程中的有效性。

首先，重要的是要注意，在我们的 K 代码中，我们已经从 `apply` 的例子中定义了不同的 `pow`：这里我们选择了一个显式定义，使得 `pow` 的参数是 `base`、`exponent`，这可以说是更自然的顺序。

我们还使用了K翻译的`recombine`来定义`mean`：在那个例子中，我们的J代码利用了*并置*来创建所谓的*3-列*，J将其定义为`recombine`，而K没有这样的内置支持。并置的唯一效果是组合，正如我们在`*/#`的例子中看到的那样。相反，我们使用了短语`@\:`，意思是“对左边的每个函数应用”，带有包含`sum`和`length`的数组，并在其左边组合了`%/`，用除法减少包含两个操作结果的数组。

我们的最终定义，`std`，是一个无声表达式，描述了JavaScript代码中迭代执行的所有操作的组合。特别要强调的是`pow[;2]`的拼写。这是所谓的*投影*，其语义是生成一个将`2`作为函数`pow`的第二个参数固定的函数。K对投影的原始且灵活的语法使得描述部分函数应用非常方便；而在上面我们的`partial`示例中，我们需要包含`flip`来绑定`pow`的第二个参数：K的投影语法允许在任何参数位置进行部分应用。

结果函数是以下函数的并置

+   `1 mean\`: 生成包含 `x` 和 `x` 均值的数组。

+   `-/`: 用`-`连接这两个值。在这一步中隐含的是显式的JavaScript中的`map`；作为一个数组语言，K在从值数组中减去单个值时“做正确的事情”。

+   `pow[;2]'`: 对差值进行平方。

+   `mean @`: 取均值。

+   `%`: 取平方根。

除了显式的 `@`，所有情况下两个动词并置被语法上视为函数组合，生成一个组合的单一函数。

### 每个值都是唯一的。

```
const eachValueIsUnique = xs => xs === _.uniq(xs)
return eachValueIsUnique([0, 1, 1, 2, 3, 5, 8, 13]) // => false
return eachValueIsUnique([0, 1, 2, 3, 5, 8, 13]) // => true 
```

```
const eachValueIsUnique = recombine(_.isEqual, identity, _.uniq) 
```

```
(def each-value-is-unique (recombine deep= identity distinct))
(each-value-is-unique [0 1 2 3 5 8 13]) 
```

```
 each_value_is_unique =: -: ~.
   each_value_is_unique 0 1 1 2 3 5 8 13
0
   each_value_is_unique 0 1 2 3 5 8 13
1 
```

J 还支持另一种专门的做事无声的编程方式；我们已经看到 3-列构成了*叉*，而 2-列（我们通过将`recombine`传入`identity`作为`g`或`h`表达的方法）有其自己的组合意义：它创建了一个*钩*，以便`f g`的行为类似于`x => f(x, g(x))`。

```
 eachValueIsUnique: ~/ 1 ?:\
    eachValueIsUnique 0 1 1 2 3 5 8 13
0
    eachValueIsUnique 0 1 2 3 5 8 13
1 
```

数组家族的其他成员倾向于将2-列保留给简单的函数组合，正如我们在其他K示例中看到的那样。在K中，专门的钩行为可以通过`n-dos`实现，其中`1 g\ x`生成一个 `(x; g x)` 的数组，然后我们可以在其上折叠一些`f`。如果`f`需要传递`g(x), x`，那么我们可以在折叠之前反转数组。

同样地，我们可以使用`eachleft`来编写`mean`的方式：

```
 ~/ (?:;::) @\: 0 1 1 2 3 5 8 13
0
    ~/ (?:;::) @\: 0 1 2 3 5 8 13
1 
```

在这里，我们应用的两个函数中的第二个是`::`，即身份函数。

## under

```
const isAnagram = (x, y) => _.sortBy(x) === _.sortBy(y)
return isAnagram("live", "evil") 
```

```
const isAnagram = under(_.isEqual, _.sortBy) 
```

```
(def is-anagram (under deep= sort))
(is-anagram @"live" @"evil") 
```

```
sort =: /:~
    is_anagram =: -:&:sort
    is_anagram =: -:&:sort
   'evil' is_anagram 'live'
1 
```

在J中，二元`under`被称为Appose，`&:`。

值得注意的是，只有一个参数时，`under`的行为就像`compose`：`x => _.isEqual(_.sortBy(x))`。（因此，单子Appose `&:` 等同于At `@:`，我们在上面已经看到了！）

值得注意的区别在于，当给定两个或更多参数时，我们希望如何运用最内层函数。在`compose(f, g)`中，所有这些参数都传递给单个`g`的应用，然后将结果传递给`f`。在`under`中，`g`被视为单参数函数，并且组合的每个参数都会单独调用`g`。
