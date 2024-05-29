<!--yml
category: 未分类
date: 2024-05-29 12:34:52
-->

# Release 1.0.0 · jgthms/bulma · GitHub

> 来源：[https://github.com/jgthms/bulma/releases/tag/1.0.0](https://github.com/jgthms/bulma/releases/tag/1.0.0)

Bulma v1 is a **full rewrite** of the framework using [**Dart Sass**](https://sass-lang.com/dart-sass/), which is the the primary implementation of Sass. While this affects a few development details, everything has been done to make the transition **as easy as possible**.

## What remains the same

**All HTML snippets are the same**. This means you don't need to update your markup. **This is important** because it means, if you're using Bulma straight "out of the box", you don't need to change anything.

You can just swap `bulma@0.9.4/css/bulma.min.css` with `bulma@1.0.0/css/bulma.min.css` and everything will work. Things will look slightly different, but they will still work.

## What changes

*   [**Dart Sass**](https://sass-lang.com/dart-sass/) is used to build Bulma
    *   if you use the `sass` npm package, you're already using Dart Sass
*   [**CSS Variables**](https://bulma.io/documentation/features/css-variables/) are used instead of literals: `color: var(--bulma-primary);` instead of `color: hsl(171deg, 100%, 41%);`, which means you can customize Bulma with CSS only (without using Sass)
*   Customization by setting your own value for Sass variables works differently. See [how to customize Bulma with Sass](https://bulma.io/documentation/customize/).

## What's new (i.e. did not exist before)

*   The notion of [**Themes**](https://bulma.io/documentation/features/themes/) is introduced: a theme is a collection of CSS variables within a context, and is the best approach to customize Bulma
*   As a result, a Theme for [**Dark Mode**](https://bulma.io/documentation/features/dark-mode/) is included
*   [**Color Palettes**](https://bulma.io/documentation/features/color-palettes/) are created for each of the 7 primary colors
*   [**Skeleton loaders**](https://bulma.io/documentation/features/skeletons/) exist as standalone components but also as variants of other components
*   You can add a **prefix** to all your Bulma classes so that `.button` becomes `.my-prefix-button`