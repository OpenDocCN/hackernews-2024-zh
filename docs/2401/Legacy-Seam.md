<!--yml

类别：未分类

日期：2024-05-27 14:33:31

-->

# **传统接缝**

> 来源：[https://martinfowler.com/bliki/LegacySeam.html](https://martinfowler.com/bliki/LegacySeam.html)

在与传统系统合作时，识别并创建接缝是有价值的：可以在不编辑源代码的情况下更改系统行为的地方。一旦找到接缝，我们就可以使用它来打破依赖关系，以简化测试，插入探针以增加可观察性，并将程序流重定向到新模块，作为传统置换的一部分。

迈克尔·菲瑟斯在他的书[《与传统代码高效工作》](https://www.amazon.com/gp/product/0131177052/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0131177052&linkCode=as2&tag=martinfowlerc-20)中首次提出了“接缝”这个术语。他的定义是：**“接缝是在程序中可以更改行为而不必编辑该位置的地方”**。

下面是一个接缝会很方便的示例。想象一下一些用于计算订单价格的代码。

```
// TypeScript
export async function calculatePrice(order:Order) {
  const itemPrices = order.items.map(i => calculateItemPrice(i))
  const basePrice = itemPrices.reduce((acc, i) => acc + i.price, 0)
  const discount = calculateDiscount(order)
  const shipping = await calculateShipping(order)
  const adjustedShipping = applyShippingDiscounts(order, shipping)
  return basePrice + discount + adjustedShipping
}

```

函数`calculateShipping`会命中一个外部服务，这个服务很慢（而且昂贵），所以我们不希望在测试时调用它。相反，我们想引入一个[存根](/bliki/TestDouble.html)，这样我们就可以为每个测试场景提供一个预先定义的确定性响应。不同的测试可能需要函数的不同响应，但我们不能在测试内部编辑`calculatePrice`的代码。因此，我们需要在对`calculateShipping`的调用周围引入一个接缝，使我们的测试能够将调用重定向到存根。

有一种方法是将`calculateShipping`函数作为参数传递

```
export async function calculatePrice(order:Order, shippingFn: (o:Order) => Promise<number>) {
  const itemPrices = order.items.map(i => calculateItemPrice(i))
  const basePrice = itemPrices.reduce((acc, i) => acc + i.price, 0)
  const discount = calculateDiscount(order)
  const shipping = await shippingFn(order)
  const adjustedShipping = applyShippingDiscounts(order, shipping)
  return basePrice + discount + adjustedShipping
}

```

对该函数的一个单元测试可以替代一个简单的存根。

```
const shippingFn = async (o:Order) => 113
expect(await calculatePrice(sampleOrder, shippingFn)).toStrictEqual(153)
```

每个接缝都有一个**启用点**：“可以在该点决定使用哪种行为或另一种行为的地方”[[WELC]](https://www.amazon.com/gp/product/0131177052/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0131177052&linkCode=as2&tag=martinfowlerc-20)。将函数作为参数传递在`calculateShipping`的调用者中开辟了一个启用点。

现在测试变得更加容易了，我们可以输入不同的运费值，并检查`applyShippingDiscounts`是否能正确响应。尽管我们必须更改原始源代码以引入接缝，但对该函数的任何进一步更改都不需要我们修改该代码，所有更改都发生在启用点，即测试代码中。

将函数作为参数传递并不是我们引入接缝的唯一方式。毕竟，改变`calculateShipping`的签名可能会带来风险，而且我们可能不想在生产代码中将航运函数参数传递给传统调用堆栈。在这种情况下，查找可能是更好的方法，比如使用服务定位器。

```
export async function calculatePrice(order:Order) {
  const itemPrices = order.items.map(i => calculateItemPrice(i))
  const basePrice = itemPrices.reduce((acc, i) => acc + i.price, 0)
  const discount = calculateDiscount(order)
  const shipping = await ShippingServices.calculateShipping(order)
  const adjustedShipping = applyShippingDiscounts(order, shipping)
  return basePrice + discount + adjustedShipping
}

```

```
class ShippingServices {
  static #soleInstance: ShippingServices
  static init(arg?:ShippingServices) {
    this.#soleInstance = arg || new ShippingServices()
  }
  static async calculateShipping(o:Order) {return this.#soleInstance.calculateShipping(o)}
  async calculateShipping(o:Order)  {return legacy_calcuateShipping(o)}
  // ... more services

```

定位器允许我们通过定义一个子类来覆盖行为。

```
class ShippingServicesStub extends ShippingServices {
  calculateShippingFn: typeof ShippingServices.calculateShipping =
     (o) => {throw new Error("no stub provided")}
  async calculateShipping(o:Order) {return this.calculateShippingFn(o)}
  // more services

```

然后我们可以在测试中使用一个启用点

```
const stub = new ShippingServicesStub()
stub.calculateShippingFn = async (o:Order) => 113
ShippingServices.init(stub)
expect(await calculatePrice(sampleOrder)).toStrictEqual(153)
```

这种服务定位器是通过函数查找设置缝隙的经典面向对象的方式，我在这里展示它是为了表明我可能会在其他语言中使用的方法，但我不会在 TypeScript 或 JavaScript 中使用这种方法。相反，我会将类似这样的内容放入一个模块中。

```
export let calculateShipping = legacy_calculateShipping

export function reset_calculateShipping(fn?: typeof legacy_calculateShipping) {
  calculateShipping = fn || legacy_calculateShipping
}

```

我们可以像这样在测试中使用这段代码。

```
const shippingFn = async (o:Order) => 113
reset_calculateShipping(shippingFn)
expect(await calculatePrice(sampleOrder)).toStrictEqual(153)
```

正如最后一个例子所暗示的，用于缝隙的最佳机制在很大程度上取决于语言、可用框架，以及遗留系统的风格。控制遗留系统意味着学习如何在代码中引入各种缝隙，以提供正确类型的启用点，同时将对遗留软件的干扰降到最低。虽然函数调用是引入此类缝隙的一个简单示例，但在实践中它们可能更为复杂。一个团队可能会花费几个月的时间来思考如何将缝隙引入一个熟悉的遗留系统中。向遗留系统添加缝隙的最佳机制可能与我们在绿地上为了类似的灵活性所做的不同。

Feathers 的书主要关注于将遗留系统置于测试之下，因为这通常是能够以理智的方式与其一起工作的关键所在。但是缝隙的用途不止于此。一旦我们有了一个缝隙，我们就可以将探针放入遗留系统中，从而增加系统的可观察性。我们可能想要监视对 `calculateShipping` 的调用，弄清楚我们使用它的频率，并将其结果捕获进行单独分析。

但是，缝隙最有价值的用途可能是它们允许我们将行为迁移到遗留之外。一个缝隙可以将高价值客户重定向到不同的运费计算器。有效的遗留位移基于在遗留系统中引入缝隙，并使用它们逐渐将行为转移到更现代的环境中。

缝隙也是我们编写新软件时要考虑的事情，毕竟每个新系统迟早都会成为遗留系统。我的许多设计建议都是关于使用适当放置的缝隙构建软件，这样我们就可以轻松测试、观察和增强它。如果我们编写软件时考虑测试，我们往往会得到一个很好的缝隙集，这就是为什么 [测试驱动开发](/bliki/TestDrivenDevelopment.html) 是如此有用的技术的原因之一。
