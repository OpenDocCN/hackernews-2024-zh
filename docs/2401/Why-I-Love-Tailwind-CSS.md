<!--yml

category: 未分类

date: 2024-05-27 15:02:37

-->

# 为什么我喜欢 Tailwind CSS

> 来源：[https://ben.page/tailwind](https://ben.page/tailwind)

我是[Tailwind CSS](https://tailwindcss.com)的铁杆粉丝。

我试过几次才开始使用 Tailwind。我之前见过它，但是大概两年前我再次尝试并且坚持了下来。

对我来说，最让我感到困惑的一件事情始终是记住类的名称。一般来说，我认为记忆是最大的学习曲线。CSS 属性的名称已经在我的脑海中了，所以需要记住另一种语法看起来有点傻。但是这种语法最终也进入了我的脑海，现在我喜欢它了。（我后来意识到有一个官方的 VS Code 扩展可以为类添加自动完成，这有助于记忆，非常方便。）

好吧，我喜欢它的一些原因：

## 我可以非常快地为事物添加样式

由于我可以在同一个视图中紧挨着编写 HTML 和 CSS，所以我可以非常快速地为事物添加样式。我不需要在 HTML 文件和 CSS 文件之间切换，所以我可以边写边样式，这对我来说感觉非常快。

## 不给类命名

或许我不擅长命名，但有时为类命名确实很痛苦。不需要给类命名然后分开样式消除了这个问题。

## 无用的 CSS 会自动删除

有时我删除 HTML 时并没有意识到我留下了曾经为 HTML 添加样式的无用 CSS。由于 Tailwind 样式封装在我删除的元素的类中，删除 HTML 也会将 CSS 一并删除。

## 颜色已经预先选择

Tailwind 的[颜色调色板](https://tailwindcss.com/docs/colors)已经装载了许多颜色。我知道直接使用预配置的颜色是很懒的，但这样做也加快了速度。

以前我试图避免使用新的颜色，因为我知道我得挑选它们，然后甚至可能选择它们的变体。现在，如果我想为确认图标使用绿色，那么多个绿色的不同色调都是可用的，而不需要额外的工作。

## 项目之间有很强的一致性

我每个 Tailwind 项目都使用相同的 API，所以我不需要记住我是否将类命名为 `no-scroll` 还是 `lock-scroll` 还是 `no-scrolling`。项目之间的一致性使得在它们之间切换和进行更改变得更加容易，而不需要不断地参考过去的命名。

## 响应式样式设计

Tailwind 的 `sm:`/`md:`/等前缀使得响应式设计变得非常容易，它将移动端样式与桌面端样式放在一起。在一个大的 CSS 文件中，我经常会在底部添加媒体查询，将相同元素的样式相距很远。

另外，即使你创建了自己的自定义实用程序类，要复制这个响应式前缀功能也很困难，你最终还是得在媒体查询中编写自定义 CSS。

## Tailwind 排版

Tailwind的[排版插件](https://tailwindcss.com/docs/typography-plugin) *非常好*。我以前总是试图在博客文章的样式上摸索，直到看起来好看，然后不可避免地我总是漏掉了一些东西（比如我会添加引用，但看起来很糟糕），所以我不得不再添加更多的样式。

排版问题一劳永逸地解决了，现在花时间为博客的内容编写 CSS 似乎有点愚蠢，当我只需使用这个插件，必要时稍作调整即可。

* * *

我明白 —— 这似乎不像是编写真正的 CSS，而且学习曲线有点陡峭。但对*我来说*，这让我在网站样式设计方面变得更快。肌肉记忆非常强大，我发现这是无价的。