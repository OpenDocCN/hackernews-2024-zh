<!--yml

category: 未分类

date: 2024-05-29 13:27:37

-->

# 通用领域类型

> 来源：[https://mmapped.blog/posts/25-domain-types.html](https://mmapped.blog/posts/25-domain-types.html)

许多语言提供了简化领域类型定义的语法。这些定义创建了一个新的独特类型，与所选的基础类型共享表示（例如，64位整数）。这些定义的语义在各种语言中有所不同，但通常分为两类：*newtypes* 和 *typedefs*。

Newtypes 包装现有类型，允许程序员继承一些来自底层类型的操作，并添加新操作。Newtypes 灵活，但可能需要样板代码来实现在真实世界应用中所需的所有功能。

在 Rust 中使用 newtype 习惯用法的示例。继承基本操作，如比较和哈希，很容易，但算术操作需要大量样板代码。一些第三方包，如 [`derive_more`](https://crates.io/crates/derive_more)，使这项任务更加简单。

```
*/// The number of standard SI apples.*
#[derive(Clone, Copy, PartialEq, Eq, PartialOrd, Ord, Hash)]
struct MetricApples(i64);

impl std::ops::Add for MetricApples {
  type Output = Self;
  fn add(self, other: Self) -> Self {
    MetricApples(self.0 + other.0)
  }
} 
```

[Haskell](https://wiki.haskell.org/Newtype) 和 [Rust](https://doc.rust-lang.org/book/ch19-04-advanced-types.html#using-the-newtype-pattern-for-type-safety-and-abstraction) 是支持 newtypes 的语言的示例。

Typedefs 为现有类型引入一个新名称，继承所有底层类型的操作。

在 Go 中使用 typedefs 的示例。Typedefs 继承了从底层类型继承的所有操作，即使对于新类型来说这些操作毫无意义。

```
*/// MetricApples hold the number of standard SI apples.*
type MetricApples int64

func main() {
  a, b := MetricApples(2), MetricApples(3)
  // Go allows us to add, multiply, and divide MetricApples.
  // Note that all these operations give us MetricApples back, which doesn’t always make sense.
  // Apples times apples should give apples squared.
  // Dividing apples should give a dimensionless number.
  fmt.Printf("%[1]T %[1]d, %[2]T %[2]d, %[3]T %[3]d\n", a+b, a*b, b/a)
} 
```

[Go](https://go.dev/ref/spec#Type_definitions)，[D](https://dlang.org/library/std/typecons/typedef.html) 和 [Ada](https://en.wikibooks.org/wiki/Ada_Programming/Type_System#Derived_types) 提供了 typedefs（Ada 将 typedefs 称为 *derived types*）。[Boost](https://www.boost.org/) 项目为 C++ 实现了 [typedefs](https://www.boost.org/doc/libs/1_61_0/libs/serialization/doc/strong_typedef.html) 作为库（C 的 [typedef declarations](https://en.cppreference.com/w/c/language/typedef) 是 <q>weak typedefs</q>：它们为现有类型引入了一个别名，而不是一个新类型）。

Newtypes 和 typedefs 非常灵活且实用，但它们处理问题的方式过于简单和机械化。有一种更系统的方式来思考领域类型。

多年来，我发现在我参与的大多数应用程序中，特定类别的领域类型会反复出现。本节是这些类别的概述。

我使用伪 Rust 语法来说明这些概念，但这些思想应该很容易转换到任何静态类型语言。

这篇文章中所有通用领域类型共享的接口。

```
trait DomainType {
  */// The primitive type representing the domain value.*
  type Representation; */// Creates a domain value from its representation value.*
  fn from_repr(repr: Representation) -> Self;

  */// Extracts the representation value from the domain value.*
  fn to_repr(self) -> Representation;
} 
```

代码片段呈现了每个类型类的 *minimal* 接口。实际问题通常需要添加更多操作。例如，在字典中使用标识符作为键需要暴露一个哈希函数（用于哈希映射），或者强加一个顺序（用于搜索树），并且序列化值需要访问它们的内部表示。

域类型的最常见用途之一是作为现实世界中实体或资产的透明句柄，例如在线商店中的客户标识符或工资应用程序中的员工编号。我将这些类型称为*标识符*。

标识符没有结构，即我们不关心它们的内部表示。唯一的基本要求是能够比较这些类型的值是否相等。这种缺乏结构建议了这类类型的适当数学模型：一个*集合*，一个包含不同对象的集合。

标识符的最小接口。

```
trait Eq {
  */// Returns true if two values are equal.*
  fn eq(&self, other: &Self) -> bool;
}

trait IdentifierLike: DomainType + Eq {} 
```

[Newtypes](#newtypes)非常适合作为标识符，因为它们能够隐藏结构。另一方面，[Typedefs](#typdefs)则过于结构化，允许程序员意外地添加和减去数值标识符。但是如果选择的话，typedefs比原始整数或字符串更安全。

域类型的另一个典型用途是表示数量，例如银行账户中以美元计的金额或字节中的文件大小。能够比较、添加和减去金额是至关重要的。

通常情况下，我们不能将两个兼容的金额相乘或相除，然后期望得到相同类型的金额。

然而，通过无量纲数乘以金额是有意义的。银行应用程序将金额增加十个百分点或磁盘工具通过文件计数来除以分配的总字节数都没有问题。

适当的数学抽象化为金额的是[向量空间](https://en.wikipedia.org/wiki/Vector_space)。向量空间是一个集合，附加了定义在此集合元素上的额外操作：加法、减法和标量乘法，这些操作的行为满足一些自然的[公理](https://en.wikipedia.org/wiki/Vector_space#Definition_and_basic_properties)。

金额的最小接口。

```
trait Ord: Eq {
  */// Compares two values.*
  fn cmp(&self, other: &Self) -> Ordering;
}

trait VectorSpace {
  */// The scalar type is usually the same as the [Representation](#representation-type) type.*
  type Scalar;

  */// Returns the additive inverse of the value.*
  fn neg(self) -> Self;

  */// Adds two vectors.*
  fn add(self, other: Self) -> Self;

  */// Subtracts the other vector from self.*
  fn sub(self, other: Self) -> Self;

  */// Multiplies the vector by a scalar.*
  fn mul(self, factor: Scalar) -> Self;

  */// Divides the vector by a scalar.*
  fn div(self, factor: Scalar) -> Self;
}

trait AmountLike: IdentifierLike + VectorSpace + Ord {} 
```

[Newtypes](#newtypes)允许我们实现金额，但可能需要一些繁琐的代码才能正确进行乘法和除法。[Typedefs](#typdefs)很方便，但在乘法和除法方面会出错，混淆美元和美元的平方。

处理类似空间的结构，如时间和空间，提出了一个有趣的挑战。空间有两种类型的值：绝对位置和相对距离。

位置指的是空间中的点，例如时间戳或地理坐标。距离表示两个这样的点之间的差异。

一些自然语言承认这种区别，并为这些概念提供不同的词汇，例如英语中的<q>o’clock</q>与<q>hours</q>或德语中的<q>Uhr</q>与<q>Stunden</q>。

虽然距离与[量](#amounts)相同，但位置更加棘手。我们可以比较、排序和减去它们以计算两点之间的距离。例如，从星期五凌晨5点减去星期六凌晨3点得到22小时。然而，增加或乘以这些日期没有意义。这种语义要求一种新的类型类别，*loci*（*locus*的复数形式）。

系统编程中的位置/距离二分法示例之一来自内存地址算术。低级编程语言区分*指针*（内存地址）和*偏移量*（地址之间的距离）。在C语言中，`void*`类型表示内存地址，`ptrdiff_t`类型表示偏移量。减去两个指针得到一个偏移量，但增加或乘以指针没有意义。

我们可以将每个位置视为距离固定起始点的距离。改变起始点或距离类型需要新的位置类型。

loci的最小接口。

```
trait LocusLike: IdentifierLike + Ord {
  */// The type representing the distance between two positions.*
  type Distance: AmountLike;

  */// The origin for the absolute coordinate system.*
  const ORIGIN: Self;

  */// Moves the point away from the origin by the specified distance.*
  fn add(self, other: Distance) -> Self;

  */// Returns the distance between two points.*
  fn sub(self, other: Self) -> Distance;
} 
```

时间戳提供了<q>距离类型 + 起始点</q>概念的极好演示。Go语言和Rust使用纳秒数表示自UNIX纪元（1970年1月1日午夜）以来的时间。C语言定义了[`time_t`](https://en.cppreference.com/w/c/chrono/time_t)类型，通常表示自UNIX纪元以来的秒数。[q编程语言](https://en.wikipedia.org/wiki/Q_(programming_language_from_Kx_Systems))同样使用纳秒，但选择了[千年纪](https://code.kx.com/q4m3/2_Basic_Data_Types_Atoms/#253-date-time-types)（2000年1月1日午夜）作为其起始点。改变距离类型（如秒到纳秒）或起始点（如UNIX纪元到千年纪）需要不同的时间戳类型。

Go标准库在其[`time`](https://pkg.go.dev/time)包中采用了位置类型设计，区分时间点（[`time.Time`](https://pkg.go.dev/time#Time))和时间段（[`time.Duration`](https://pkg.go.dev/time#Duration))。

Rust标准模块[`std::time`](https://doc.rust-lang.org/std/time/index.html)是一个更进化的示例。它定义了[`SystemTime`](https://doc.rust-lang.org/std/time/struct.SystemTime.html)类型用于墙上钟时间（起始点是[UNIX纪元](https://doc.rust-lang.org/std/time/struct.SystemTime.html#associatedconstant.UNIX_EPOCH)），[`Instant`](https://doc.rust-lang.org/std/time/struct.Instant.html)用于单调时钟（起始点是<q>过去某个未指定的时间点</q>，通常是系统启动时间），以及[`Duration`](https://doc.rust-lang.org/std/time/struct.Duration.html)类型用于两个时钟测量之间的距离。

到目前为止，我们考虑了域类型几乎不相互作用的应用。许多应用需要在单个表达式中结合不同域类型的值。物理仿真可能需要将时间间隔乘以速度以计算行进距离。金融应用可能需要将美元金额乘以转换率以获取欧元金额。

我们可以使用 [量纲分析](https://en.wikipedia.org/wiki/Dimensional_analysis) 方法来建模复杂的类型交互。如果我们将 [amounts](#amounts) 视为带有附加标签的值，标识其单位，那么我们的新类型就是对更结构化标签的自然扩展，等效于基本单位的向量提升到有理幂。例如，加速度将具有标签 (distance × time^(-2))，而 usd/eur [pair](https://en.wikipedia.org/wiki/Currency_pair) 汇率将具有标签 (eur × usd^(-1))。我将具有这种丰富标签结构的类型称为*量*。

量是量的正确扩展：加法、减法和标量乘法的工作方式相同，保持标签结构不变。额外的标签结构赋予了乘法和除法以意义。

乘法的结果将具有基本单位向量，其组分为单位因子的幂向量的分量*和*。例如，汽车燃油消耗计算可以使用类似 2 (km) × 0.05 (liter × km^(-1)) = 0.1 (liter) 的表达式。

除法产生一个标签，它是被除数和除数幂向量之间的组分*差异*。例如，跑步配速计算可以使用类似 10 (min) / 2 (km) = 5 (min × km^(-1)) 的表达式。

数量的最小接口。

```
trait QuantityLike<DimA>: AmountLike {
  */// Multiplies two quantities.*
  fn mul<O: QuantityLike<DimB>>(self, other: O)
    -> impl QuantityLike<AddUnitPowers<DimA, DimB>>;

  */// Divides self by the specified quantity.*
  fn div<O: QuantityLike<DimB>>(self, other: O)
    -> impl QuantityLike<SubUnitPowers<DimA, DimB>>;
} 
```

数量需要复杂的类型级机制，这使得它们在大多数语言中难以实现。[Boost.Units](https://www.boost.org/doc/libs/1_65_0/doc/html/boost_units.html) 是首批在 C++ 中提供完整数量类型实现的库之一。Rust 生态系统提供了 [dimensioned](https://crates.io/crates/dimensioned) 包。在 Haskell 生态系统中，[units](https://hackage.haskell.org/package/units) 包是一个流行的选择。

如果您的语言不支持高级类型级编程或者在您的应用中使用严格类型不切实际，您可以在运行时进行单位检查。Python 的 [quantities](https://github.com/python-quantities/python-quantities) 包就是一个这样的例子。
