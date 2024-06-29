<!--yml

category: 未分类

date: 2024-05-27 15:00:03

-->

# Union、intersection、difference 等功能即将到来到 JavaScript Set | Sonar

> 来源：[https://www.sonarsource.com/blog/union-intersection-difference-javascript-sets/](https://www.sonarsource.com/blog/union-intersection-difference-javascript-sets/)

[JavaScript `Set`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set "JavaScript Set") 在 ES2015 规范中引入了语言，但似乎始终不完整。这即将改变。

集合是一组值的集合，其中每个值只能出现一次。在 ES2015 版本的 `Set` 中，可用的功能围绕着创建、添加、删除和检查 `Set` 的成员身份展开。如果你想要操作或比较多个集合，你必须编写自己的函数。幸运的是，[TC39](https://tc39.es/ "TC39") — 负责制定 ECMAScript 规范的委员会 — 和浏览器一直在努力解决这个问题。我们现在在 JavaScript 实现中看到了像 `union`、`intersection` 和 `difference` 这样的函数。

在我们看新功能之前，让我们回顾一下 JavaScript Set 现在能做什么，然后我们会介绍下面的 [新的 Set 函数](https://www.sonarsource.com/blog/union-intersection-difference-javascript-sets/#what-are-the-new-set-functions "new Set functions") 和 [支持它们的 JavaScript 引擎](https://www.sonarsource.com/blog/union-intersection-difference-javascript-sets/#support "JavaScript engines that support them")。

## ES2015 JavaScript Set 做了什么？[](/blog/union-intersection-difference-javascript-sets/#what-do-es2015-javascript-sets-do "What do ES2015 JavaScript Sets do?")

让我们来看看 JavaScript `Set` 的 ES2015 版本能做什么。最好通过一些示例来说明。

你可以在不提供任何参数的情况下 [构造一个 `Set`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/Set "construct a Set")，这将给你一个空的 `Set`。或者，你可以提供一个可迭代对象，如数组，来初始化 `Set`。

```
const languages = new Set(["JavaScript", "TypeScript", "HTML", "JavaScript"]);
```

`Set` 只能包含唯一值，因此上面的 `Set` 有三个成员。你可以用 [`size` 属性](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/size "size property") 来检查这一点。

```
languages.size;

// => 3
```

你可以使用 [`add` 函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/add "add function") 向 `Set` 中添加更多元素。添加已经存在于 `Set` 中的元素不会产生任何效果。

```
languages.add("JavaScript");

languages.add("CSS");

languages.size;

// => 4
```

你可以用 [`delete`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/delete "delete") 从 `Set` 中删除元素。

```
languages.delete("TypeScript");

languages.size;

// => 3
```

你可以使用[`has`函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/has "has function")来检查一个元素是否是`Set`的成员。`Set`的一个好处是这种检查可以在常数时间内完成（**O(1)**），而检查一个元素是否在`Array`中的时间则因`Array`的长度而异（**O(n)**）。对于这样的任务使用`Set`是[编写意图明确且高效的代码的一种清晰方式](https://www.sonarsource.com/blog/what-is-clean-code/#example-4 "clean way to write intentional efficient code")。

```
languages.has("JavaScript");

// => true

languages.has("TypeScript");

// => false
```

你可以使用[`forEach`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/forEach "loop through a Set's elements using forEach")函数或`for...of`循环来遍历`Set`的元素。元素按照它们被添加到`Set`中的顺序排序。

```
languages.forEach(element => console.log(element));

// "JavaScript"

// "HTML"

// "CSS"
```

你还可以使用[`keys`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/keys "keys")和[`values`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/values "values")函数（[它们实际上是等效的](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/keys#using_keys "which are actually equivalent")）以及[`entries`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/entries "entries")函数从`Set`中获取迭代器。

最后，你可以使用[`clear`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/clear "clear")函数清空一个`Set`。

```
languages.clear();

languages.size;

// => 0
```

这是一个关于ES2015规范版本的`Set`可以做什么的很好的提醒：

+   `Set`提供了处理唯一值集合的方法。

+   向`Set`中添加元素和测试其是否存在于`Set`中是高效的。

+   将`Array`或其他可迭代对象转换为`Set`是过滤重复项的简单方法。

这种实现忽略了`Set`之间的操作。你可能想要创建一个包含来自两个其他`Set`的所有项目的`Set`（两个`Set`的并集），找出两个`Set`共有的项目（交集），或找出一个`Set`中不存在但在另一个`Set`中存在的项目（差集）。直到最近，你可能需要提供自己的函数。

## 新的`Set`函数是什么？[](/blog/union-intersection-difference-javascript-sets/#what-are-the-new-set-functions "What are the new Set functions?")

`Set`方法提案为`Set`实例添加了以下方法：`union`，`intersection`，`difference`，`symmetricDifference`，`isSubsetOf`，`isSupersetOf`和`isDisjointFrom`。

这些方法中有些类似于某些SQL连接，我们将用它们来说明结果与代码并行。让我们看看每个函数做了什么的一些例子。

你可以在Chrome 122+或Safari 17+中尝试下面的任何代码示例。

### `Set.prototype.union(other)`[](/blog/union-intersection-difference-javascript-sets/#setprototypeunionother "Set.prototype.union(other)")

集合的并是一个包含两个集合中所有元素的集合。

```
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);

const backEndLanguages = new Set(["Python", "Java", "JavaScript"]);

const allLanguages = frontEndLanguages.union(backEndLanguages);

// => Set {"JavaScript", "HTML", "CSS", "Python", "Java"}
```

在这个例子中，前两个集合的所有语言都在第三个集合中。与其他添加元素到`Set`的方法一样，重复的元素会被移除。

这相当于两个表之间的 SQL `FULL OUTER JOIN`。

### `Set.prototype.intersection(other)`[](/blog/union-intersection-difference-javascript-sets/#setprototypeintersectionother "Set.prototype.intersection(other)")

交集是包含两个集合中都存在的所有元素的集合。

```
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);

const backEndLanguages = new Set(["Python", "Java", "JavaScript"]);

const frontAndBackEnd = frontEndLanguages.intersection(backEndLanguages);

// => Set {"JavaScript"} 
```

"JavaScript" 是这两个集合中唯一同时存在的元素。

交集类似于`INNER JOIN`。

### `Set.prototype.difference(other)`[](/blog/union-intersection-difference-javascript-sets/#setprototypedifferenceother "Set.prototype.difference(other)")

你正在操作的集合与另一个集合之间的差异是第一个集合中存在但第二个集合中不存在的所有元素。

```
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);

const backEndLanguages = new Set(["Python", "Java", "JavaScript"]);

const onlyFrontEnd = frontEndLanguages.difference(backEndLanguages);

// => Set {"HTML", "CSS"} 

const onlyBackEnd = backEndLanguages.difference(frontEndLanguages);

// => Set {"Python", "Java"}
```

在找到集合之间的差异时，调用函数的集合和参数的集合很重要。在上面的例子中，从前端语言中移除后端语言会导致"JavaScript"被移除，并在结果集中返回"HTML"和"CSS"。而从后端语言中移除前端语言仍然导致"JavaScript"被移除，并返回"Python"和"Java"。

差异类似于执行`LEFT JOIN`。

### `Set.prototype.symmetricDifference(other)`[](/blog/union-intersection-difference-javascript-sets/#setprototypesymmetricdifferenceother "Set.prototype.symmetricDifference(other)")

两个集合之间的对称差集是一个包含两个集合中某一个但不同时出现的所有元素的集合。

```
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);

const backEndLanguages = new Set(["Python", "Java", "JavaScript"]);

const onlyFrontEnd = frontEndLanguages.symmetricDifference(backEndLanguages);

// => Set {"HTML", "CSS", "Python", "Java"} 

const onlyBackEnd = backEndLanguages.symmetricDifference(frontEndLanguages);

// => Set {"Python", "Java", "HTML", "CSS"}
```

在这种情况下，结果集中的元素是相同的，但请注意顺序不同。集合的顺序由添加到集合中的元素的顺序决定，并且函数执行的集合将首先添加其元素。

对称差集类似于`FULL OUTER JOIN`，不包括两个表中都存在的任何元素。

### `Set.prototype.isSubsetOf(other)`[](/blog/union-intersection-difference-javascript-sets/#setprototypeissubsetofother "Set.prototype.isSubsetOf(other)")

如果第一个集合中的所有元素都出现在第二个集合中，则第一个集合是第二个集合的子集。

```
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);

const declarativeLanguages = new Set(["HTML", "CSS"]);

declarativeLanguages.isSubsetOf(frontEndLanguages);

// => true

frontEndLanguages.isSubsetOf(declarativeLanguages);

// => false
```

一个集合也是其自身的子集。

```
frontEndLanguages.isSubsetOf(frontEndLanguages);

// => true
```

### `Set.prototype.isSupersetOf(other)`[](/blog/union-intersection-difference-javascript-sets/#setprototypeissupersetofother "Set.prototype.isSupersetOf(other)")

如果第二个集合中的所有元素都出现在第一个集合中，则第一个集合是第二个集合的超集。这是子集的相反关系。

```
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);

const declarativeLanguages = new Set(["HTML", "CSS"]);

declarativeLanguages.isSupersetOf(frontEndLanguages);

// => false

frontEndLanguages.isSupersetOf(declarativeLanguages);

// => true
```

一个集合也是其自身的超集。

```
frontEndLanguages.isSupersetOf(frontEndLanguages);

// => true
```

### Set.prototype.isDisjointFrom(other)[](/blog/union-intersection-difference-javascript-sets/#setprototypeisdisjointfromother "Set.prototype.isDisjointFrom(other)")

最后，如果两个集合没有共同元素，则一个集合与另一个集合是不相交的。

```
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);

const interpretedLanguages = new Set(["JavaScript", "Ruby", "Python"]);

const compiledLanguages = new Set(["Java", "C++", "TypeScript"]);

interpretedLanguages.isDisjointFrom(compiledLanguages);

// => true

frontEndLanguages.isDisjointFrom(interpretedLanguages);

// => false
```

这些集合中的解释语言和编译语言没有重叠，因此这些集合是不相交的。前端语言和解释语言与元素"JavaScript"重叠，因此它们不是不相交的。

## 支持[](/blog/union-intersection-difference-javascript-sets/#support "支持")

在撰写本文时，[提案](https://github.com/tc39/proposal-set-methods "提案")在[TC39的流程](https://tc39.es/process-document/ "TC39的流程")中处于第3阶段，并且Safari 17（于2023年9月发布）和Chrome 122（于2024年2月发布）已经实现了这些方法。Edge紧随Chrome，Firefox Nightly在标志后也支持，预计这两个浏览器也很快会支持。

[Bun](https://bun.sh/ "Bun")还使用Safari的[JavaScriptCore引擎](https://docs.webkit.org/Deep%20Dive/JSC/JavaScriptCore.html "JavaScriptCore引擎")，因此已经支持这些新功能。Chrome的支持意味着它已添加到[V8 JavaScript引擎](https://v8.dev/ "V8 JavaScript引擎")中，并将很快被[Node.js](https://nodejs.org/ "Node.js")采纳。

希望这意味着该提案将在流程中毕业到第4阶段，或许在ES2024规范最终确定之前就加入其中。

### Polyfills[](/blog/union-intersection-difference-javascript-sets/#polyfills "Polyfills")

虽然您需要旧版JavaScript引擎的支持，但您可以使用polyfill来升级到符合规范的这些函数的实现。它们在[core-js](https://github.com/zloirock/core-js#new-set-methods "core-js")中可用，或作为[es-shims项目](https://github.com/es-shims "es-shims项目")中每个功能的单独包（例如，可以使用[set.prototype.union包](https://www.npmjs.com/package/set.prototype.union "set.prototype.union包")进行联合功能）。

如果您已编写了这些函数的自己的实现，我建议您首先升级到polyfill，然后随着支持变得更加广泛逐步淘汰它们。

## [集合不再感觉不完整](/blog/union-intersection-difference-javascript-sets/#sets-no-longer-feel-incomplete "集合不再感觉不完整")

JavaScript的`Set`长期以来并不完整，但这7个新功能使其实现更加完善。将这样的功能内置到语言中意味着我们可以更少依赖依赖项或我们自己的实现，并且可以[专注](https://www.sonarsource.com/blog/what-is-clean-code/#intentional "专注")于我们正在解决的问题。

现在在TC39之前有一个[完整的第三阶段提案流程](https://github.com/tc39/proposals?tab=readme-ov-file "完整的第三阶段提案流程")。查看列表以了解接下来可能加入JavaScript的其他内容。我特别关注[Temporal](https://github.com/tc39/proposal-temporal "Temporal")和[Decorators](https://github.com/tc39/proposal-decorators "Decorators")，这两者都有可能改变我们编写JavaScript的主要部分的方式。
