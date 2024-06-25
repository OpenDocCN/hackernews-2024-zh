<!--yml

类别：未分类

日期：2024年05月27日 15:16:08

-->

# Porffor：性能的类型注释

> 来源：[https://goose.icu/porffor-types/](https://goose.icu/porffor-types/)

我的JS引擎[Porffor](https://porffor.goose.icu)现在可以解析TypeScript，因为我添加了可插拔的解析器支持，其中包括Babel的解析器（可以解析TS）。这本身并不那么有趣，但我现在可以将这些类型注释用作编译器提示来进行优化！

## 基本示例

```
let a = 1; if (a) {  console.log(typeof a); } 
```

对于上面的JS，Porffor必须对`a`的类型进行几次检查：

1.  `if (a) {`

+   如果是字符串，如果非空，则为真（不是`''`）

+   如果未定义，则为假

+   &mldr;

+   **如果是数字，为真，如果不是零**

1.  `typeof a`

+   如果是字符串，`'string'`

+   如果是对象，`'object'`

+   &mldr;

+   **如果是数字，`'number'`**

我已经优化了文件中完全未使用的类型。例如，如果文件中没有使用字符串，则会删除所有字符串检查。但如果有任何字符串，则所有字符串检查都将保留。Porffor可以尝试自动推断类型，但这很棘手，可能会出错。JIT引擎有时可以通过假定某个东西的类型来推断类型，一旦它以该类型多次运行，等等。但由于我们是AOT编译而不是JIT，如果类型猜测出错，我们无法取消猜测 - 相反，我们将会崩溃。

但我们可以使用类型注释（TS）定义类型！

```
let a: number = 1; if (a) {  console.log(typeof a); } 
```

现在我们不必对`a`的类型进行**任何检查**，因为我们在编译时已经知道了它。这可以实现一些很大的速度提升 :)

## 渐进类型

Porffor本身不进行类型检查。如果你想要的话，你可以使用你自己的（还有你自己的配置）。输入也不必完全类型化，允许“渐进类型”。存在的类型可以帮助加快速度，而未类型化的内容将被正常处理。

## 基准测试

我在[我写的基本的Brainf&mldr;解释器（`bf.js`）](https://github.com/CanadaHonk/porffor/blob/main/bench/bf.js)上使用了加上[一个带类型的版本（`bf.ts`）](https://github.com/CanadaHonk/porffor/blob/main/bench/bf.ts)。这里是运行绘制曼德勃罗集的BF的时间，使用不同的命令（`porf` = Porffor）：

#### `porf bf.js`：272秒

这只是正常运行JS文件，使用默认选项。

#### `porf bf.ts -parse-types`：274秒

这会运行TS文件，但仅解析类型而不使用它们进行优化。与常规JS几乎相同的时间，有一些误差。

#### `porf bf.ts -parse-types -opt-types`：184秒

`-opt-types`告诉Porffor将类型注释用作编译器提示。仅仅通过使用类型就提速了1.5倍！

#### `porf bf.ts -parse-types -opt-types -valtype=i32`：84秒

`-valtype=i32`也告诉Porffor使用`i32`（32位整数）作为`number`，而不是`f64`。在未来，我计划添加一个特殊类型来完成这个任务，而不是为所有内容更改`number`，但编译器还不够智能呢。与上次相比，又提速了2.2倍，总共提速了3.2倍！

#### `porf bf.ts -parse-types -opt-types -valtype=i32 -funsafe-zero-proto-checks=charcodeat`：67秒

`-funsafe-zero-proto-checks=charcodeat` 还告诉编译器不对 `String.prototype.charCodeAt` 进行任何输入超出界限等检查。在未来，这也应该能够被自动检测并执行。与以前相比，轻微的1.3倍加速，导致相对于原始值的4倍。

#### `node bf.js`: 66秒

作为比较，Node 运行相同的 JS 文件。显然，Node 是 JIT 编译的，因此与编译 AOT 到字节码相比具有很大的优势。然而，我们仍然使用正确的选项大致与其匹配！ 🚀

#### `quickjs bf.js`: 1812秒

作为比较，一个传统的解释器运行相同的基准测试。

## 结论

对我来说最好的部分是，[这个功能的主要差异很小，只有 +131 -25](https://github.com/CanadaHonk/porffor/commit/a6a92e01ac1e09383f5c3f24e55f2648ff714b7a)，而且只花了我大约一个晚上的时间来完成；感谢[重写](/porffor-rewrite/)提供的良好基础。请注意，常规 JS 和类型化 JS 很可能在不久的将来会更快！这只是今天。我还没有与许多其他引擎进行比较，因为我把它留到了以后，当我做更多的基准测试和测试时。

为了增加喜剧效果，这使得 Porffor 的完整描述变得更加复杂：

> 一个从零开始的实验性的 AOT 优化 JS/TS -> Wasm/C 引擎/编译器/运行时在 JS 中

感谢阅读！ :)
