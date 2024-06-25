<!--yml

分类：未分类

日期：2024-05-27 14:46:23

-->

# Crystal 1.11.0 发布了！- Crystal 编程语言

> 来源：[`crystal-lang.org/2024/01/08/1.11.0-released/`](https://crystal-lang.org/2024/01/08/1.11.0-released/)

我们宣布一个新的 Crystal 发布，其中包含几个新功能和错误修复。

预构建的软件包可在 [GitHub 发布页面](https://github.com/crystal-lang/crystal/releases/tag/1.11.0) 和我们的官方发行渠道上获取。请参阅 [crystal-lang.org/install](https://crystal-lang.org/install/) 获取安装说明。

## 统计信息

本次发布包含自 1.10.1 版以来的 [178 个更改](https://github.com/crystal-lang/crystal/pulls?q=is%3Apr+milestone%3A1.11.0)，由 28 位贡献者贡献。我们感谢所有为改进语言而付出努力的贡献者！❤️

## 更改

下面我们列出了语言、编译器和标准库中最显着的更改。这是一个非常重要的发布版本，有很多事情正在发生，所以请紧紧抓住 🚀

详细信息，请访问 [完整更新日志](https://github.com/crystal-lang/crystal/releases/tag/1.11.0)。

### LLVM 18

最重要的进步之一是对即将发布的 LLVM 18 的支持，这使得在 Windows 上动态链接 LLVM 成为可能（[#14101](https://github.com/crystal-lang/crystal/pull/14101)）。此外，LLVM 18 现在提供了我们在上游 C API 中所需的一切，消除了我们对包装扩展 `llvm_ext` 的需要。对于较旧的 LLVM 版本，仍然是必需的，因此我们将继续保留它一段时间。但未来的工具链正在变得简化。在 [#13946](https://github.com/crystal-lang/crystal/issues/13946) 中阅读更多内容。

*感谢 [@HertzDevil](https://github.com/HertzDevil)*

### 编译器优化级别

编译器获得了四个不同的优化级别：

+   `-O0`：无优化

+   `-O1`：低优化

+   `-O2`：中等优化

+   `-O3`：高度优化

每个级别激活相应的 LLVM `RunPasses` 和 `CodeGenOptLevel` 优化。

`-O3` 对应于现有的发布模式，`-O0` 对应于默认的非发布模式。`-O0` 保持默认，`--release` 等效于 `-O3 --single-module`。

实际上，这引入了两个优化选择，介于之前的完全或无任何优化之间。现在可以在没有 `--single-module` 的情况下使用高度优化。

在 [#13464](https://github.com/crystal-lang/crystal/pull/13464) 中阅读更多内容。

*感谢 [@kostya](https://github.com/kostya)*

### 对齐原语

该语言有两个新的反射原语：[`alignof`](https://crystal-lang.org/reference/1.11/syntax_and_semantics/alignof.html) 和 [`instance_alignof`](https://crystal-lang.org/reference/1.11/syntax_and_semantics/instance_alignof.html) 返回类型的内存对齐方式（[#14087](https://github.com/crystal-lang/crystal/pull/14087)）。这使得在本机 Crystal 中实现类型感知的分配器成为可能，使用正确对齐的指针。它们与 `sizeof` 和 `instance_sizeof` 是同级的，可以以相同的方式使用。

```
class Foo
  def initialize(@x : Int8, @y : Int64, @z : Int16)
  end
end

Foo.new(1, 2, 3)

instance_alignof(Foo) # => 8 
```

对现有代码的影响

引入这些原语使得无法定义同名的方法。所以`def alignof`或`def instance_alignof`现在是无效语法。我们不认为这会在实践中产生很大影响。

*感谢[@HertzDevil](https://github.com/HertzDevil)*

### `Link`注解中的`dll`参数

[`Link`](https://crystal-lang.org/api/1.11.0/Link.html)注解有一个新参数`dll`，用于在 Windows 上指定动态链接库 ([#14131](https://github.com/crystal-lang/crystal/pull/14131)).

```
@[Link(dll: "foo.dll")]
lib LibFoo
end 
```

*感谢[@HertzDevil](https://github.com/HertzDevil)*

### 宏`@caller`上下文

现在宏通过特殊实例变量[`@caller`](https://crystal-lang.org/reference/1.11/syntax_and_semantics/macros/index.html#call-information)引用其调用上下文 ([#14055](https://github.com/crystal-lang/crystal/issues/14055)).

```
macro foo
  {{- @caller.line_number }}
end

foo # => 5 
```

*感谢[@Blacksmoke16](https://github.com/Blacksmoke16)*

### 新的集合方法

[`Enumerable#present?`](https://crystal-lang.org/api/1.11.0/Enumerable.html#present?:Bool-instance-method) 是`#empty?`的直接反转，避免了与相似但不完全相同的`#any?`的一些怪癖 ([#13847](https://github.com/crystal-lang/crystal/issues/13847)).

*感谢[@straight-shoota](https://github.com/straight-shoota)*

[`Enumerable#each_step`](https://crystal-lang.org/api/1.11.0/Enumerable.html#each_step(n:Int,*,offset:Int=0,&:T-%3E):Nil-instance-method) 和 [`Iterable#each_step`](https://crystal-lang.org/api/1.11.0/Iterable.html#each_step(n:Int)-instance-method) 是创建步进迭代器的直接方法 ([#13610](https://github.com/crystal-lang/crystal/pull/13610)).

*感谢[@baseballlover723](https://github.com/baseballlover723)*

[`Enumerable(T)#to_set(& : T -> U) : Set(U) forall U`](https://crystal-lang.org/api/1.11.0/Enumerable.html#to_set(&block:T-%3EU):Set(U)forallU-instance-method) 和 [`#to_a(& : T -> U) forall U`](https://crystal-lang.org/api/1.11.0/Enumerable.html#to_a%28%26%3AT-%3EU%29%3AArray%28U%29forallU-instance-method) 允许将`Enumerable`转换为预定义集合，比标准的`#to_set`和`#to_a`方法更灵活 ([#12654](https://github.com/crystal-lang/crystal/pull/12654), [#12653](https://github.com/crystal-lang/crystal/pull/12653)).

*感谢[@caspiano](https://github.com/caspiano)*

### 数字增强

[`BigFloat#**`](https://crystal-lang.org/api/1.11.0/BigFloat.html#%2A%2A%28other%3ABigInt%29%3ABigFloat-instance-method) 现在适用于所有`Int::Primitive`参数，并支持`BitInt`参数的完整指数范围 ([#13971](https://github.com/crystal-lang/crystal/pull/13971), [#13881](https://github.com/crystal-lang/crystal/pull/13881))

`printf`中的浮点到字符串转换使用了 Ryu 算法 ([#8441](https://github.com/crystal-lang/crystal/issues/8441)).

新方法 [`Float::Primitive.to_hexfloat`](https://crystal-lang.org/api/1.11.0/Float64.html#to_hexfloat:String-instance-method)、[`.parse_hexfloat`](https://crystal-lang.org/api/1.11.0/Float64.html#parse_hexfloat(str:String):self-class-method) 和 [`.parse_hexfloat?`](https://crystal-lang.org/api/1.11.0/Float64.html#parse_hexfloat(str:String):self-class-method) 允许在十六进制浮点格式之间进行转换 ([#14027](https://github.com/crystal-lang/crystal/pull/14027))。

更多数学特性：

*感谢 [@HertzDevil](https://github.com/HertzDevil)*

### `crystal spec` 的增强

`crystal spec` 获取了两个用于内省的新命令：

`crystal spec --dry-run` 打印出所有活动的规范，而不实际执行任何规范代码 ([#13804](https://github.com/crystal-lang/crystal/pull/13804))。

*感谢 [@nobodywasishere](https://github.com/nobodywasishere)*

`crystal spec --list-tags` 列出了规范套件中定义的所有标签 ([#13616](https://github.com/crystal-lang/crystal/pull/13616))。

*感谢 [@baseballlover723](https://github.com/baseballlover723)*

`Crystal 1.10` 中 `crystal tool unreachable` 的基本实现得到了一些有用的增强。

+   `--tallies` 选项打印出所有方法及其调用总数。那些计数为零的方法是无法访问的 ([#13969](https://github.com/crystal-lang/crystal/pull/13969))。

+   `--check` 标志如果存在任何无法访问的代码，则以失败状态退出 ([#13930](https://github.com/crystal-lang/crystal/pull/13930))。

+   注释显示在输出中 ([#13927](https://github.com/crystal-lang/crystal/pull/13927))。

+   新的输出格式：CSV ([#13926](https://github.com/crystal-lang/crystal/pull/13926))。

+   输出中的路径被相对化，使其更加简洁 ([#13929](https://github.com/crystal-lang/crystal/pull/13929))。

*感谢 [@straight-shoota](https://github.com/straight-shoota)*

### API 文档中的继承宏

继承的宏现在在 API 文档中暴露出来了。它们以前是隐藏的，与继承的定义相反 ([#13810](https://github.com/crystal-lang/crystal/pull/13810))。

*感谢 [@Blacksmoke16](https://github.com/Blacksmoke16)*

### 文本

### 其他

+   `String::Buffer` 和 `IO::Memory` 的容量无意间被限制为 1GB。它们现在支持完整范围，最多可达到 `Int32::MAX`，即 2GB ([#13989](https://github.com/crystal-lang/crystal/pull/13989))。 *感谢 [@straight-shoota](https://github.com/straight-shoota)*

+   `Number#format` 中有一个讨厌的 bug 可能会影响整数部分。现在在 [#14061](https://github.com/crystal-lang/crystal/pull/14061) 中已经修复了。 *感谢 [@HertzDevil](https://github.com/HertzDevil)*

+   Vendored shards `markd` 和 `reply` 不再通过相对于编译器源树的路径引用。这意味着当将编译器作为库使用时，它们可以是本地依赖项（即在 `lib` 中） ([#13992](https://github.com/crystal-lang/crystal/pull/13992))。 *感谢 [@nobodywasishere](https://github.com/nobodywasishere)*

+   有两个新常量提供有关编译器主机和目标的信息：`Crystal::HOST_TRIPLE`和`TARGET_TRIPLE`（[#13823](https://github.com/crystal-lang/crystal/pull/13823)）。*感谢[@HertzDevil](https://github.com/HertzDevil)*

### Shards 0.17.4

打包的碎片发布已更新至[0.17.4](https://github.com/crystal-lang/shards/releases/tag/v0.17.4)，带来了一些次要的错误修复。（[#14133](https://github.com/crystal-lang/crystal/pull/14133)）。

*感谢[@straight-shoota](https://github.com/straight-shoota)*

### 实验性内容：`ReferenceStorage`和`.pre_initialize`

我们已经开始努力使在 Crystal 中使用自定义分配机制变得更加容易，并将分配与初始化解耦。主要工具是[`Reference.pre_initialize`](https://crystal-lang.org/api/1.11.0/Reference.html#pre_initialize(address:Pointer)-class-method)，它在实际调用`#initialize`之前执行基本的对象初始化。

[`Reference.unsafe_construct`](https://crystal-lang.org/api/1.11.0/Reference.html#unsafe_construct%28address%3APointer%2C%2Aargs%2C%2A%2Aopts%29%3Aself-class-method)是在其上的一个更高级别的 API。[`ReferenceStorage`](https://crystal-lang.org/api/1.11.0/ReferenceStorage.html)代表了引用分配的静态缓冲区。

这些 API 是实验性的，可能会发生变化。我们期待在未来的版本中有更多相关功能。参与有关自定义引用分配的讨论，请访问[#13481](https://github.com/crystal-lang/crystal/issues/13481)。

> *注意：*`ReferenceStorage`由于与旧版本标准库的兼容性问题，在 1.11.1 中再次移除（[#14207](https://github.com/crystal-lang/crystal/pull/14207)）。它将以改进的实现方式回归。

*感谢[@HertzDevil](https://github.com/HertzDevil)*

## 废弃内容

+   在宏表达式中使用展开运算符已经不推荐使用。请使用`.splat`代替（[#13939](https://github.com/crystal-lang/crystal/pull/13939)）

+   `LLVM.start_multithreaded`和`.stop_multithreaded`。它们没有影响（[#13949](https://github.com/crystal-lang/crystal/pull/13949)）

+   `LLVMExtSetCurrentDebugLocation`来自`llvm_ext.cc`，适用于 LLVM 9+（[#13965](https://github.com/crystal-lang/crystal/pull/13965)）

+   `Char::Reader#@end`（[#13920](https://github.com/crystal-lang/crystal/pull/13920)）

感谢

我们之所以能够做到这一切，要感谢[84codes](https://www.84codes.com/)和其他每一个赞助商的持续支持。为了保持和提高开发速度，捐赠和赞助至关重要。可通过[OpenCollective](https://opencollective.com/crystal-lang)进行捐赠。

如果您想成为直接赞助商或找到其他支持 Crystal 的方式，请联系 crystal@manas.tech。我们在此先行致谢！

贡献
