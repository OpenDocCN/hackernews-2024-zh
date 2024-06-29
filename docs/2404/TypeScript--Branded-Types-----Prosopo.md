<!--yml

category: 未分类

date: 2024-05-27 13:29:40

-->

# TypeScript: 品牌化类型 🔧 - Prosopo

> 来源：[https://prosopo.io/articles/typescript-branding/](https://prosopo.io/articles/typescript-branding/)

**[第1部分：TypeScript映射类型魔法](/articles/typescript-mapped-type-magic/)**

TypeScript 战士们，你们好！👋 今天我们在[TypeScript映射类型文章](/articles/typescript-mapped-type-magic/)的基础上扩展了我们的工作，引入*品牌化*。上一篇文章讨论了如何在TypeScript映射类型中以名义方式而不是结构方式使用。

```
type A = {
    x: number
}

type B = {
    x: number
}
```

这是一种高级的说法，即TypeScript默认是结构化的，即在处理类型时会将类型`A`和`B`视为相等。使类型`A`和`B`称为名义上的会使TypeScript区分它们，即使它们的结构相同。

在本文中，我们基于上述工作提出一种方法来*品牌化*类型，提供一种自动化且易于使用的方式使类型成为名义化。品牌化专注于仅在类型系统中而不是像之前文章中介绍的引入运行时字段，这是与之前方法相比的一个重大优势。

## 问题出在哪里？

品牌化，也称为不透明类型，在TypeScript中允许区分类型，否则这些类型将被分类为相同类型。例如，

```
 type A = {
    x: number,
    y: boolean,
    z: string,
}

type B = {
    x: number,
    y: boolean,
    z: string,
} 
```

`A`和`B`在结构上是完全相同的，因此TypeScript接受`A`或`B`的任何实例来替代彼此：

```
 const fn = (a: A) => {
    console.log('do something with A')
}

const obj: B = {
    x: 1,
    y: true,
    z: 'hello'
}

fn(obj) 
```

函数期望输入类型为`A`的值，而我们传递的是类型为`B`的值。TypeScript通过结构比较这两种类型，因为它们具有完全相同的结构，所以认为这个操作是可以接受的。

但是，如果我们需要区分`A`和`B`怎么办？概念上讲，它们必须不同吗？如果我们在`A`和`B`上做一些TypeScript不知道但我们需要这些类型不同的花哨事情，那就是我们最近遇到的情况！

我们需要品牌化来正好做到这一点。

## 解决方案

就像在[TypeScript映射类型文章](/articles/typescript-mapped-type-magic/)中一样，关键在于创建一个带有符号名称的字段作为我们的*id*。然而，使用品牌化时，我们只需要在类型级别添加这个字段，而不是在运行时级别。由于类型在编译后被抹除，我们需要在不改变运行时数据的情况下向类型添加这个字段。需要类型转换，对吧？

首先，让我们介绍`brand`字段。

```
 const brand = Symbol('brand') 

type A = {
    x: number,
    y: boolean,
    z: string,
} & {
    [brand]: 'A'
}
```

在这里，我们正在为类型`A`添加`brand`字段。品牌字段名称是一个符号，类似于UUID。我们使用符号来确保`A`的品牌字段永远不会与`A`的任何其他字段冲突，否则我们将覆盖一个字段并引入更糟糕的错误：类型错误 🐛 。目前我们将品牌设置为`'A'`，但这可以是你想要的任何内容。这类似于类型名称。现在让我们再次比较`A`和`B`：

```
 const fn = (a: A) => {
    console.log('do something with A')
}

const obj: B = {
    x: 1,
    y: true,
    z: 'hello'
}

fn(obj) 
```

这里是错误的所在：

```
Argument of type 'B' is not assignable to parameter of type 'A'.
  Property '[brand]' is missing in type 'B' but required in type '{ [brand]: "a"; }'.ts(2345) 
```

TypeScript 不允许我们将 `B` 的实例传递给接受 `A` 的函数，因为它缺少 `brand` 字段 - 太棒了！`A` 和 `B` 现在是不同的类型了。但是如果 `B` 有自己的品牌呢？

```
 type B = {
    x: number,
    y: boolean,
    z: string,
} & {
    [brand]: 'B'
}
```

请注意，我们仍然使用之前的相同 `brand` 变量。保持这个常量非常重要，否则我们将声明具有不同名称的字段！

现在让我们再试试这个函数：

```
 const fn = (a: A) => {
    console.log('do something with A')
}

const obj: B = {
    x: 1,
    y: true,
    z: 'hello'
}

fn(obj) 
```

这里是错误

```
Argument of type 'B' is not assignable to parameter of type 'A'.
  Type 'B' is not assignable to type '{ [brand]: "A"; }'.
    Types of property '[brand]' are incompatible.
      Type '"B"' is not assignable to type '"A"'.ts(2345) 
```

就这样了！错误表明，尽管两种类型都有一个 `brand` 字段，但这两种类型的品牌值不同，即 `'A' != 'B'`！

让我们看看如果 `brand` 是相同的会发生什么：

```
 type A = {
    x: number,
    y: boolean,
    z: string,
} & {
    [brand]: 'foobar'
}

type B = {
    x: number,
    y: boolean,
    z: string,
} & {
    [brand]: 'foobar'
}

const fn = (a: A) => {
    console.log('do something with A')
}

const obj: B = {
    x: 1,
    y: true,
    z: 'hello'
}

fn(obj) 
```

没有错误！`A` 和 `B` 被视为可互换的类型，因为它们在结构上是相同的，具有相同的字段 *和* `'foobar'` 的品牌值。太棒了！

## 让它成为泛型！

太棒了，这样就行了。但这只是一个玩具示例，不适合生产。让我们创建一个可以为任何您希望的类型添加品牌的 `Brand` 类型：

```
const brand = Symbol('brand') 

type Brand<T, U> = T & {
    [brand]: U
}
```

这种类型非常简单，它接受您的类型 `T` 并添加一个带有 `U` 品牌值的 `brand` 字段。这里是如何使用它的：

```
 type A_Unbranded = {
    x: number,
    y: boolean,
    z: string,
}

type A = Brand<A_Unbranded, 'A'> 
```

现在我们可以给任何类型添加品牌了。为了完整起见，这里是同样类型的内容，用于 *移除* 品牌并回到普通的 TypeScript 类型：

```
type RemoveBrand<T> = T[Exclude<keyof T, typeof brand>]
```

这将从任何带品牌的类型中移除 `brand` 字段。还要注意，如果类型没有品牌，它将不会受到影响！

## 现实世界的用法

让我们把这个应用到实践中。我们有一个需要通过[映射类型](/articles/typescript-mapped-type-magic/)来识别其类型的类。

为了简单起见，让我们将这个类简化为一个 `Dog` 类：

```
 class Dog {
    constructor(public name: string) {}
}

type DogBranded = Brand<Dog, 'Dog'>

const dog = new DogBranded('Spot') 
```

TypeScript 不允许我们构造一个带有品牌的狗 😢 。我们需要使用构造函数进行一些类型转换，这样可以给 *构造函数* 而不是 *类* 自身添加品牌。

```
 type Ctor<T> = new (...args: any[]) => T

const addBrand = <T>(ctor: Ctor<T>, name: string) => {
    return ctor as Ctor<Brand<T, typeof name>>
}

const DogBranded = addBrand(Dog, 'Dog')

const dog = new DogBranded('Spot') 
```

`addBrand` 函数接受一个类的构造函数并将其转换为带有品牌的类型。这本质上是为 `Dog` 类创建一个别名，它可以与 `Dog` 类完全相同的方式使用，例如在其上调用 `new`。

我们可以 `export` `DogBranded` 类型以允许外部世界使用我们的类，同时确保它始终是带品牌的：

```
export type DogExported = typeof DogBranded
```

同样，我们可以对去除品牌的操作做同样的处理：

```
 const removeBrand = <T>(value: T) => {
    return value as RemoveBrand<T>
}
```

这只是通过将类型转换为没有 `brand` 字段的类型映射来删除品牌。

然后我们来了：在 TypeScript 中创建和移除类型品牌的绝佳方法 😃

我们已将此工作发布为一个库，您可以通过[NPM](https://www.npmjs.com/package/@prosopo/ts-brand)访问！

在 Prosopo，我们正在使用 TypeScript 的品牌技术来强化我们的类型，并为即将发布的运行时类型验证器进行智能类型映射。敬请期待更新！

**[第一部分：TypeScript 映射类型的魔法](/articles/typescript-mapped-type-magic/)**

* * *
