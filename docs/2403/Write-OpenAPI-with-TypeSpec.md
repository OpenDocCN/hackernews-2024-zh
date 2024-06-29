<!--yml

category: 未分类

date: 2024-05-29 12:42:57

-->

# 使用TypeSpec编写OpenAPI

> 来源：[https://blog.trl.sn/blog/typespec-for-openapi/](https://blog.trl.sn/blog/typespec-for-openapi/)

<main id="skip">

# 使用TypeSpec编写OpenAPI

在Microsoft工作的最后几年中，我专注于一种称为[TypeSpec](https://typespec.io)的API定义语言。它本质上是一种超级灵活的协议不可知DSL，用于描述API形状。您可以在[TypeSpec游乐场](https://typespec.io/playground)中尝试它。关于它的许多方面令人兴奋，但我特别想谈一件事：为什么TypeSpec是编写OpenAPI的最佳方式。

## OpenAPI: 优点和不足

OpenAPI非常擅长描述HTTP API的低级细节。它允许软件理解API的形状，从而实现诸如生成客户端和文档、配置API网关或生成测试用例等多种有用功能。事实证明，OpenAPI是描述HTTP API最广泛使用的语言，这充分显示了其优势。

但是，在与Azure内部使用OpenAPI后，我也清楚地看到OpenAPI存在一些关键弱点。人们不认为OpenAPI特别适合编写和审查，无论是在JSON还是YAML中。尽管社区中的许多人做出了巨大的努力，但从OpenAPI生成的代码通常并不理想。在大规模构建API时，其冗长和缺乏可重用组件的特性需要大量投入API审查和治理过程。

我相信API优先开发原则在理论上很棒，但在实践中，使用OpenAPI的API优先方法面临着重大阻力。开发人员更有可能选择以代码优先的方式生成OpenAPI作为构建产物。我认为TypeSpec改变了游戏规则。让我们来看看吧！

## 进入TypeSpec

TypeSpec被设计为协议不可知，但团队在大幅度投资使其支持出色的OpenAPI 3.0输出。您可以使用TypeSpec通过其提供的`http`标准库来定义大多数OpenAPI文档，该库提供了各种类型和装饰器，用于添加HTTP特定的元数据，如头部、查询字符串或状态码。使用TypeSpec编写OpenAPI比直接使用OpenAPI具有许多优势。今天我将讨论的是，TypeSpec更小，更易读，并且可以使用API组件。

### 简洁而富有表现力

TypeSpec的语法和语义大量借鉴于TypeScript，后者是描述REST API形状的最佳语言之一。TypeSpec能够用比OpenAPI更少的打字描述复杂的模型形状和REST端点，最终结果更易读。我甚至听说有些人成功地使用TypeSpec语法与非技术利益相关者讨论API形状！

让我们看看我所说的一个例子。您可以在 [游乐场中查看](https://typespec.io/playground?c=aW1wb3J0ICJAdHlwZXNwZWMvaHR0cCI7DQp1c2luZyBIdHRwOw0KDQovKiogQSBjaGFyYWN0ZXIgaW4gYSBmYW50YXN5IFJQRy1zdHlsZSBnYW1lICovDQptb2RlbCBDyTB7DQogIG5hbWU6IHN0cmluZzvEEWlkOiBzYWZlaW50xRBzdGF0dXM6ICJBbGl2ZSIgfCAiRGVhZOQAkCAgY2xhc3M6IEPEBzsNCn3kAJdlbnVtxhIgeyB3YXJyaW9yOyB3aXphcmQ7IMUjQG1pblZhbHVlKDEpxA5heMYOMjApDQpzY2FsYXLFeMUWIGV4dGVuZHMgdWludDjlAPrmAMpTdGF0c%2BYAxnN0cmVuZ3Ro5ADKxzfmAM1udGVsbGlnZW5j5QDmzBxkxFhyaXR5zhnlAKFvcCBnZXTpASsoQHBhdGjsARspOuoBSTs%3D&e=%40typespec%2Fopenapi3&options=%7B%7D) 中查看生成的 OpenAPI。

```
import "@typespec/http";
using Http;
 /** A character in a fantasy RPG-style game */
model Character {
 name: string;
 id: safeint;
 status: "Alive" | "Dead";
 class: Class;
}
 enum Class { warrior; wizard; }
 @minValue(1)
@maxValue(20)
scalar statValue extends uint8;
 model Stats {
 strength: statValue;
 intelligence: statValue;
 dexterity: statValue;
}
 op getCharacter(@path id: safeint): Character;
```

幻想 RPG 的部分 TypeSpec API 定义

这种 TypeSpec 最终编译成的 OpenAPI 几乎是行数的三倍，字节数的三倍半。TypeSpec 代码非常容易阅读和理解 —— 对于 TypeScript 开发者来说很熟悉，但对任何人来说都足够清晰。

特别是 TypeSpec 联合体展示了 TypeSpec 语法如何显著提升可读性。联合体提供了一种简洁的语法，编译成 OpenAPI 的 `oneOf`、`anyOf` 和 `enum` 结构。考虑以下 TypeSpec 和 OpenAPI 示例：

```
model Character {
 // ... snip other fields ...
 status: "Alive" | "Dead";
}
```

TypeSpec 文字类型的联合体

```
status:
 type: string
 enum:
 - Alive
 - Dead
```

OpenAPI 对象文字类型的输出

编译器确定这是一组文字类型的联合体，因此正确的选择是将其作为枚举发出。但是，如果我们有一组对象类型的联合体，它将编译成 `anyOf`。

```
model Character {
 // ... snip other fields ...
 items: (Weapon | Armor | GenericItem)[]
}
```

TypeSpec 对象的联合体

```
items:
 type: array
 items:
 anyOf:
 - $ref: "#/components/schemas/Weapon"
 - $ref: "#/components/schemas/Armor"
 - $ref: "#/components/schemas/GenericItem"
```

OpenAPI 对象联合输出

总的来说，TypeSpec 的各种语言特性编译成的 OpenAPI 通常更长，更复杂，无论是在语法还是语义上。

### 可组合和模块化

将 API 模式封装成可重用组件的能力是我们开始开发 TypeSpec 的主要原因之一。在规模化时，跨 API 的一致性和符合 API 准则变得非常昂贵。需要 API 作者和 API 审阅者付出巨大的努力来确保质量。

TypeSpec 的 API 组件使一致性变得容易。API 设计者可以将受欢迎的 API 模式封装到组件中，供 API 作者导入和使用。这样，API 设计者可以确保 API 的正确性，只需使用组件，而 API 作者则无需在 JSON 中实现复杂的高级模式。

常见的例子是分页。使用 TypeSpec，我们可以定义一个模板来定义页面的结构，并将其用于每个分页的终点。

```
model Page<T> {
 items: T[];
 size: int32;
 nextLink: url;
 prevLink: url;
}
 @route("/characters")
op listCharacters(): Page<Character>;
 @route("/items")
op listItems(): Page<Item>;
```

分页端点的 TypeSpec

另一个常见的模板用法是定义标准的错误形状。例如，如果每个端点都可能返回 `403` 错误，我们可以用一个模板来表达这一点：

```
alias WithStandardErrors<T> = T | ForbiddenResponse;
 @route("/characters")
op listCharacters(): WithStandardErrors<Page<Character>>;
 @route("/items")
op listItems(): WithStandardErrors<Page<Item>>;
```

标准响应形状的 TypeSpec

在 Azure 中，我们有数十个这些模板，这些模板封装了我们在整个 Azure API 表面上使用的 API 模式。我们将这些模板打包成模块，服务所有者在编写他们的 API 规范时导入这些模块。他们无需知道“长时间运行操作”使用特定的HTTP动词和特定形状的负载，他们只需为其API提供几个特定参数，并完成。API 作者高兴，审阅者满意。

## 结尾

这只是探索 TypeSpec 能力的冰山一角。有许多其他功能结合在一起，使得 TypeSpec 非常高效，生成的 OpenAPI 文档可以是原来的10倍以上。当编写API如此高效时，我认为以API为先的方式不仅变得可行，而且十分吸引人。

在接下来的帖子中，我将分享一些关于 TypeSpec 能做的有趣事情的更多细节。主题可能包括它对其他协议的支持（例如Protobuf，JSON Schema），TypeSpec的可扩展模型以及如何使用TypeScript添加自定义装饰器或生成自定义输出，TypeSpec如何实现更高质量的代码生成，以及关于分页和流API支持的未来方向。如果您对特定主题感兴趣，请随时在 [twitter](https://twitter.com/bterlson) 上表达出来。

</main>
