<!--yml

category: 未分类

日期：2024-05-27 14:41:11

-->

# Grats: 一个更愉快的构建 TypeScript GraphQL 服务器的方式 / Jordan Eldredge

> 来源：[https://jordaneldredge.com/blog/grats/](https://jordaneldredge.com/blog/grats/)

# Grats: 一个更愉快的构建 TypeScript GraphQL 服务器的方式

2024年3月4日

过去一年来，我一直在构建[Grats](https://grats.capt.dev/)，旨在展示我认为对于在 TypeScript 中构建 GraphQL 服务器来说是一种根本上更好的开发体验。

思想是简单地用文档块注释您现有的 TypeScript 代码，以指示您想要公开的构造，并让 Grats 利用您代码的现有名称和类型来提取完全类型安全的可执行 GraphQL 模式。

让我们看一个已经注释过的简单模型的示例：

```
`/**
 * A user in our system
 * @gqlType
 */
type  User  = {
 /** @gqlField */
 name:  string;
};
 /** @gqlField */
export  function  greeting(user:  User):  string {
 return  `Hello ${user.name}`;
}`
```

从这个例子中，Grats 推导出这个 GraphQL 模式：

```
`"""
A user in our system
"""
type  User {
 name: String!
 greeting: String!
}`
```

[*Playground link*](https://grats.capt.dev/playground/#N4IgJg9gxiBcIHoBUSAEAdAdqtBBVArgM4CmATqgJbYQEVECeRALiQLZY6oACA5gI4AbACoMADiU5IEWZuJKoAqqQoBeVME6pkaPkIBilEoLA4Z2VJgCGbErFQsy1XgG4sAXyxYdPAYMPGptJYJAAeYhBkzKgAZgSYUMyUENi8ZCQkSZi8ABTE5PbK5ACU9o7OGlrpzHTYAAYAEsaCEKgAJMD5ZAB01rbudW6Y7iAANCBQKTGUvHCgmASCglYARoIkAEIMACIkMVaLzHDMZAQk4+kRUaISAMpQTmLMNyQAomRkkUTHpyTu4wA3IwAdzmIFozDEBGYAHknslMHAQEQwIIxsiABYQYEAcTIVmYRG2lHSiUoAJI31g+0EpH+IAAaq8AEq3ACSMIAcnAAIzuIA)

请注意，我们不必显式声明任何我们想在模式中出现的类型或名称。Grats 简单地直接从我们的 TypeScript 代码中推导出它们。已经采用 Grats 的人报告说，它使得开发 GraphQL 服务器的体验更加轻量化，GraphQL 的大部分心智负担似乎都消失了。

## 将先实施的开发引入 TypeScript

这种模式开发方法，我称之为[先实施](https://jordaneldredge.com/blog/implementation-first/)，在 GraphQL 生态系统中并不新鲜。Python 有[Strawberry](https://strawberry.rocks/)，C# 有[Hot Chocolate](https://chillicream.com/docs/hotchocolate/)，Rust 有[Juniper](https://github.com/graphql-rust/juniper)，而 Meta 内部的 GraphQL 服务器（用 Hack/PHP 编写）[操作方式非常相似](https://youtu.be/G_zipR8Y8Ks?t=1159)。然而，这些工具都依赖于运行时类型内省或编译时宏，这在 TypeScript 中是不可能的。

使 TypeScript 的先实施 GraphQL 需要一种新颖的方法。Grats 的创新在于它使用静态分析来实现模式提取。Grats 分析您的代码作为一组[AST](https://en.wikipedia.org/wiki/Abstract_syntax_tree)，从中能够推断出您的 GraphQL 模式。

## 优势

除了改进的开发体验外，Grats 相对于现有的 TypeScript 提供的其他优势还有很多：

+   **自动类型安全** — 您的 TypeScript 类型 *就是* 您的 GraphQL 类型。无需将模式声明与实现同步。

+   **没有运行时开销** — Grats是一个仅用于开发的构建工具，不会引入任何运行时开销或增加包大小。当在边缘或浏览器中使用时，这可能会产生显著影响。

+   **友好的错误消息** — Grats并不依赖于复杂的TypeScript类型，后者通常会报告冗长和难以理解的错误。Grats对每个问题都有准确的理解，并为每个问题提供定制和有帮助的错误消息。

+   **鼓励最佳实践** — Grats将所有字段默认为可空，这是[GraphQL最佳实践](https://graphql.org/learn/best-practices/#nullability)的一部分，有助于实现更强大的响应能力。

## 缺点

没有解决方案是没有取舍的。以下是一些理性人士可能选择不使用Grats的原因：

+   **Grats引入了一个构建步骤** — 构建步骤为您的开发流程引入了摩擦力，必须权衡上述利弊。

+   **没有现有的插件生态系统** — 像[Pothos](https://pothos-graphql.dev/)这样的成熟工具带有插件生态系统，可以提供重要的价值。Grats目前没有（还没有？）类似的生态系统。

+   **模式设计不是强制性的** — 模式优先的解决方案提供了一个强制功能，明确和有意识地设计您的模式。使用Grats时，您必须在实施时在头脑中设计您的模式，然后用生成的模式确认该设计。

## 结论

Grats利用一种新颖的静态分析方法，为使用TypeScript构建GraphQL服务器提供更轻量和更愉悦的开发体验。如果您对此感兴趣，请访问[Grats网站](https://grats.capt.dev/)。一些入门的地方可能包括：

如果您有问题或意见，我很乐意在[Twitter](https://twitter.com/captbaritone/)、[Threads](https://www.threads.net/@captbaritone)，或者[Grats Discord服务器](https://discord.gg/xBf4gxPefu)上听取您的意见。
