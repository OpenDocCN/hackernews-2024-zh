<!--yml

分类：未分类

日期：2024-05-29 12:34:52

-->

# 发布1.0.0 · jgthms/bulma · GitHub

> 来源：[https://github.com/jgthms/bulma/releases/tag/1.0.0](https://github.com/jgthms/bulma/releases/tag/1.0.0)

Bulma v1是使用[Dart Sass](https://sass-lang.com/dart-sass/)的**完全重写**框架，这是Sass的主要实现。虽然这影响了一些开发细节，但已经尽可能地使过渡**尽可能简单**。

## 什么保持不变

**所有HTML片段都是相同的**。这意味着您无需更新标记。**这很重要**，因为这意味着，如果您直接使用Bulma，您无需更改任何内容。

您只需将`bulma@0.9.4/css/bulma.min.css`替换为`bulma@1.0.0/css/bulma.min.css`，一切都会正常工作。外观会略有不同，但仍然可以正常工作。

## 什么改变

+   [**Dart Sass**](https://sass-lang.com/dart-sass/)用于构建Bulma。

    +   如果您使用`sass` npm包，您已经在使用Dart Sass

+   [**CSS变量**](https://bulma.io/documentation/features/css-variables/)代替了文字字面值：`color: var(--bulma-primary);`代替了`color: hsl(171deg, 100%, 41%);`，这意味着您可以仅使用CSS（而不使用Sass）自定义Bulma。

+   通过设置自己的Sass变量值进行自定义工作方式不同。查看[如何使用Sass自定义Bulma](https://bulma.io/documentation/customize/)。

## 新特性（即之前不存在的）

+   引入了[**主题**](https://bulma.io/documentation/features/themes/)的概念：主题是一个上下文中的一组CSS变量，是自定义Bulma的最佳方法。

+   因此，包含了**深色模式**的主题

+   [**颜色调色板**](https://bulma.io/documentation/features/color-palettes/)为7种主要颜色创建。

+   [**骨架加载器**](https://bulma.io/documentation/features/skeletons/)既作为独立组件存在，也作为其他组件的变体存在。

+   您可以给所有Bulma类添加**前缀**，使`.button`变为`.my-prefix-button`。
