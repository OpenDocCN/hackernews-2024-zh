<!--yml

category: 未分类

date: 2024-05-29 12:35:38

-->

# CSS | 18F Engineering

> 来源：[https://guides.18f.gov/engineering/languages-runtimes/css/](https://guides.18f.gov/engineering/languages-runtimes/css/)

<main class="usa-layout-docs__main desktop:grid-col-9

                usa-prose measure-5 engineering">

# CSS

CSS编码风格指南的目的是在项目中创建一致的CSS或预处理器CSS代码（如[Sass](http://sass-lang.com)）。应将样式指南视为指南 — 根据项目需求可以修改规则。

## 架构[](#architecture)

网站的架构应基于其目标和目的。这意味着此处的指导应根据不同的站点和情况进行调整。

### 模块化或组件化架构[](#modular-or-component-architecture)

在使用模块化或组件化架构时，每个页面都被分解为一系列模块化组件。这些组件分为两类：`components`和`modules`。该架构从基本的HTML元素规则开始：HTML、p、a、form等标签，然后在其上编写组件和模块。组件是非常基本的结构，如按钮、简介、导航和位置结构（如插页、岛屿和围护）。接下来，使用这些组件构建模块。该架构还试图在文件中保持特异性趋势向上曲线，即向下移动文件时（稍后详细说明）。

+   从所有标签规则（a、h1-h5、p、*、html、body）的元素文件开始。

+   为每个结构元素创建组件文件，例如按钮、导航等。这些主要是基于类的，并使用[BEM](http://getbem.com/introduction/)或其他命名方案。

+   使用模块创建更具体的结构。例如，如果徽标图像和文本需要非常特定的处理方式，请使用模块。

    +   通过混合、扩展和HTML从组件构建模块。

    +   模块可以具有更高的特异性，使用更深的嵌套也是可以的。

+   有一个覆盖文件或文件夹，包含用于覆盖组件和模块的全局规则。

    +   这些可以是通用的工具。

    +   在此处放置特定于断点的规则是一个好方法，例如在小断点下隐藏某些内容。

#### 文件结构[](#file-structure)

```
_elements.scss
_mixins.scss
_typography.scss
_util.scss
_vars.scss
component/_blurb.scss
component/_button.scss
component/_island.scss
component/_sub_nav.scss
module/_logo.scss
module/_progress_bar.scss
lib/bourbon.scss
lib/neat.scss
_overrides.scss
```

对于`util`、`typography`、`elements`和`overrides`文件，一旦它们的大小超过300行，将它们拆分为自己的文件夹和子文件。

```
elements/_all.scss
elements/_p.scss
elements/_h.scss
typography/_body.scss
typography/_links.scss
overrides/_breakpoints.scss
overrides/_util.scss
util/_center.scss
util/_clearfix.scss
```

### 导入[](#importing)

正如您可能知道的那样，文件中后面的CSS规则会覆盖前面的规则。这意味着可以使用Sass导入来控制继承和特异性。

+   从基本元素开始

+   转向单一嵌套类和utils

+   转向更具体的类，通常是嵌套的

+   转向覆盖，可能带有重要的规则！

+   按字母顺序导入

+   只能修改文件组的导入顺序，而不是特定的文件

```
 @import 'module/logo';
@import 'component/mask';
@import 'component/button'; 

@import 'component/button';
@import 'component/mask';
@import 'module/logo';
```

## 文档[](#documentation)

在使用 `//`（静默注释）与 `/* */`（在 CSS 输出中保留）时要有意识。犹豫时，使用 `//`。

### KSS[](#kss)

使用 KSS 进行文档编写。更多关于 KSS 的信息可在[官方网站](http://warpspire.com/kss/)找到。

#### 示例[](#example)

```
 .button {
}

.button-modified {
}
```

#### 理由[](#rationale)

KSS 是迄今为止最常见的 CSS 文档编写方法。虽然它并非完美，但生成的文档可以通过模板进行修改。

## 框架[](#frameworks)

TTS 建议使用 [美国网页设计系统（USWDS）](https://github.com/uswds/uswds)，因为它专为快速、无障碍和移动友好的联邦政府网站构建而设计。

有时，项目可能使用其他 CSS 框架，例如：

1.  [Bourbon](https://www.bourbon.io/)

1.  [BassCSS](https://basscss.com/)

这些框架之所以被选择，是因为它们在设计决策方面相对不持有意见，同时提供了使框架在快速准确的前端工作中不可或缺的帮助程序，例如响应式设计解决方案、网格和常见设计模式。此外，通过模块化设计和优秀的文档，这两个框架使设计师或开发人员能够仅使用他们需要的部分，而不必包含庞大的库。

### 不要使用 Bootstrap[](#do-not-use-bootstrap)

18F 特别不推荐在生产工作中使用 [Bootstrap](http://getbootstrap.com/)，因为难以将其偏见的样式适应定制设计工作。

## 格式化[](#formatting)

我们建议使用 [Prettier](https://prettier.io)，并在默认情况下在您的编辑器中启用它。Prettier 是一个自动代码格式化工具，将使您的代码格式保持一致。这样我们就不必争论如何格式化我们的代码 - 只需让工具强制执行即可！Prettier 可以同时处理普通 CSS 和 SCSS。

除非需要明确设置所有可用值，否则不要使用缩写声明。

```
 margin: inherit 3em;

margin-bottom: 3em;
margin-top: 3em;

margin: 3em 4em 2em 1em;
```

避免重复、链接或依赖代码其他部分的任意数字（也称为“魔法数字”）。

```
 .component {
  top: 0.327em;
}

.component {
  top: 0.327em;
}

$align_top: 100%;
.component {
  top: $align_top;
}
```

### 顺序[](#order)

+   使用以下排序：

    1.  变量

    1.  @extend 指令

    1.  @include 指令

    1.  声明列表（`property: name;`）

    1.  媒体查询

    1.  伪状态（`:checked`、`:target`等）和伪元素（`::after`、`::selection`等）

    1.  嵌套元素

    1.  嵌套类

+   使用字母顺序或类型顺序进行声明。选择一种并在整个项目中保持一致。

+   在嵌套选择器之前放置一个新行，除非它们位于第一个选择器之后。

+   将嵌套包括，例如 Neat 的媒体包括 – `@include media($small-screen)` — 视为标准媒体查询，而不是 Sass @include。因此，它们应直接在声明列表后排序。

+   在嵌套选择器后面用 `@content` 放置混合调用。

+   您可以根据项目需求调整排序顺序，只要在整个项目中保持一致即可。

```
 .module {

  .module-element {
    color: #fff932;
  }
}

.module {
  .module-element {
    color: #fff932;
  }
}

.module {
  $amount = 3;
  @extend .component;
  @include sizing($amount);
  margin-top: $amount * 1em;
  text-align: center;

  @include media($small-screen) {
    margin-top: ($amount + 10em);
  }

  &::before {
    content: "hello";
  }

  .module__ele {
    color: #fff932;
  }
}
```

## 继承[](#inheritance)

### 混合[](#mixins)

+   对于故意一起出现且多次使用的一组属性，使用mixin。

    ```
    @mixin clearfix {
      &:after {
        content: '';
        display: table;
        clear: both;
      }
    }
    ```

+   用mixin来为组件改变大小。

+   当某些东西需要参数时使用mixin。

    ```
    @mixin size($width, $height: $width) {
      width: $width;
      height: $height;
    }
    ```

+   不要为浏览器前缀使用mixin。使用[Autoprefixer](https://github.com/postcss/autoprefixer)。

    ```
     @mixin transform($value) {
      -webkit-transform: $value;
      -moz-transform: $value;
      transform: $value;
    }
    ```

## 扩展[](#extend)

在使用`@extend`时要非常小心。它是一个功能强大的工具，但可能会导致灾难性的副作用。在使用之前，请考虑以下事项：

+   当前选择器会被附加到哪里？

+   我可能会造成不良副作用吗？

+   这个单个扩展生成的CSS有多大？

如果不确定使用`@extend`，请遵循这些规则以避免出现问题：

+   在一个模块内部使用`@extend`，而不要在不同的模块之间使用。

+   仅在[占位符](http://thesassway.com/intermediate/understanding-placeholder-selectors)上使用`@extend`，而不是实际选择器。

+   确保你扩展的占位符在样式表中尽可能少地出现。

你可以在[混入](http://sass-lang.com/guide#mixins)中使用，代替选择器。虽然混入会复制更多代码，但一旦输出文件被gzip压缩，差异通常可以忽略不计。

## 代码检查[](#linting)

使用一个代码检查工具可以确保CSS代码符合一致的规则。像[Stylelint](https://stylelint.io/)这样的检查工具可以在你的代码与已建立规则不符时发出警告。

### 在本地设置Stylelint[](#setting-up-stylelint-locally)

1.  运行`npm install --save-dev stylelint stylelint-config-standard`来下载这个包，并保存到你的`package.json`中。

1.  在项目根目录下创建一个`.stylelintrc.json`配置文件，并包含以下内容：

```
{
  "extends": "stylelint-config-standard"
}
```

1.  在你的项目中的所有CSS文件上运行Stylelint：`npx stylelint "**/*.css"`

这使用了stylelint的标准规则配置来进行CSS代码检查。如果你的项目需要不同的规则或默认设置，或者你想扩展到其他类型的文件，你可以利用自定义语法或编写你自己的规则。

Stylelint有一个[用户指南](https://stylelint.io/user-guide/get-started)，可以详细了解如何配置和扩展你的代码检查规则。

## 命名[](#naming)

+   HTML元素应该是小写的。

    ```
    body,
    div {
    }
    ```

+   类名应该是小写的。

+   避免使用驼峰命名法。

+   清晰地命名事物。

+   语义化地编写类。命名其功能而不是外观。

    ```
     .ClassNAME { }

    .commentForm { }

    .c1-xr { }
    ```

+   避免在名称中使用与呈现或位置相关的词语，因为这会在你（必然）稍后需要更改颜色、宽度或特性时造成问题。

    ```
     .blue
    .text-gray
    .100width-box

    .warning
    .primary
    .lg-box
    ```

+   当基于内容命名组件时要谨慎，因为这会限制类的使用。

    ```
     .product_list

    .item_list
    ```

+   除非是众所周知的缩写，否则不要缩写。

    ```
     .bm-rd

    .block--lg
    ```

+   在类型伪选择器中使用引号。

    ```
     .top_image[type="text"] {
    }
    ```

+   使用单数名为CSS组件和模块命名。

    ```
    .button {
    }
    ```

+   使用形容词为修改器和基于状态的规则命名。

    ```
    .is_hovered {
    }
    ```

+   如果你的CSS必须与其他CSS库进行交互，请考虑对每个类进行命名空间处理。

    ```
    .f18-component
    ```

### 命名方法论[](#naming-methodologies)

在命名时，最重要的是一致性。推荐的做法是使用像[BEM](#bem)这样的现有方法，或者使用一个清晰定义的自定义方法。

#### [BEM](#bem)

[BEM](http://getbem.com/introduction/)（**B**lock，**E**lement，**M**odifier）结构化CSS，使得每个实体都由（您猜对了）块、元素和修饰符组成。来自[Harry Roberts](http://csswizardry.com/2013/01/mindbemding-getting-your-head-round-bem-syntax/)：

> BEM的要点是仅凭其名称就能告诉其他开发人员有关标记块正在执行的更多信息。通过阅读一些带有类的HTML，您可以看到这些块如何（如果有的话）相关联；某些可能仅是一个组件，某些可能是该组件的子组件或元素，某些可能是该组件的变体或修饰符。

TTS通常建议在下一小节中概述的修改版BEM方法。但是，当您需要使用标准BEM时：

+   您需要一个通用CSS开发者已经熟悉的命名方案，或者现有的命名方案不够一致。

+   当您想要使用JavaScript动态修改BEM类名时。

这里是SCSS中BEM的一个示例：

```
 .inset {
  margin-left: 15%;

  .inset__content {
    padding: 3em;
  }
}

.inset--sm {
  margin-left: 10%;

  .inset__content {
    padding: 1em;
  }
}

.inset--lg {
  margin-left: 20%;
}
```

#### 建议的自定义方法[](#suggested-custom-methodology)

TTS对命名方法的推荐是修改版的BEM。它仍然使用块、块内部的部分和修饰符，但使用简化的语法。

```
.accordion
.accordion-item
.accordion-item-selected

.nav_bar
.nav_bar-link
.nav_bar-link-clicked 
```

#### 命名方法资源[](#naming-methodology-resources)

### 带有js-标记的类[](#js-flagged-classes)

不要将样式附加到带有`js-`标志的类。这些类保留用于JavaScript。

```
// Bad
.js-people {
  color: #ff0;
}
```

#### 理由[](#rationale-1)

带有`js-`标记的类需要具有高度的可移植性。给它添加样式会破坏这种可移植性。

### 带有test-标记的类[](#test-flagged-classes)

不要将样式附加到带有`test-`标志的类。这些类保留用于诸如selenium之类的测试钩子。

```
// Bad
.test-people {
  color: #ff0;
}
```

## 预处理器[](#preprocessors)

TTS最受支持的CSS预处理器是[Sass](http://sass-lang.com)。使用这个预处理器意味着您将获得支持的资源，如框架、库、教程和全面的样式指南。

*话虽如此，只要它是一个完善的项目并有社区支持，任何预处理器都是允许的。*

[Dart Sass](https://sass-lang.com/dart-sass)是Sass的主要实现，并建议在项目中使用。

### 命令行安装[](#command-line-installation)

#### 使用npm[](#with-npm)

#### 使用homebrew[](#with-homebrew)

+   运行 `brew install sass/sass/sass`

#### 其他安装[](#other-installations)

您可以在[他们的网站](https://sass-lang.com/install)上了解其他安装方法。

## [特异性](#specificity)

+   ID应保留用于JavaScript。不要为样式使用ID。

    ```
     #component { }

    .component { }
    ```

+   不要嵌套超过3层。

+   不要使用`!important`来解决问题。有目的地使用`!important`。

    ```
     .component {
      width: 37.4% !important;
    }

    .hidden {
      display: none !important
    }
    ```

+   保持低特异性并向下文件中的特异性趋势上升。有关更多信息，请参见[特异性图](#specificity-graph)部分。

+   不要使用不必要的标签选择器。

    ```
     p.body_text { }

    .body_text
    ```

+   如果必须提高特异性，请使用安全的方法：*多类*。

    ```
     .component.component { }
    ```

### 特异性图[](#specificity-graph)

处理特异性时的一个简单规则是从低特异性开始，向底部输出文件的方向逐渐提高特异性。由于 CSS 规则会被文件中更下方的规则替换，因此您将以预期的方式覆盖规则。

有一个可以绘制文件特异性的工具，[CSS 特异性图](http://jonassebastianohlsson.com/specificity-graph/)。请通过此工具运行您的最终输出文件，并努力使曲线向上趋势。

#### 资源[](#resources)

### 理由[](#rationale-2)

具体性带来了极大的责任。广泛的选择器使我们能够高效，但如果不经过测试可能会产生不良后果。特定位置的选择器可以节省时间，但会迅速导致样式表混乱。请慎重选择选择器，以找到在整体 DOM 样式和布局中平衡的最佳方式。

+   当修改现有元素以用于特定用途时，请尝试使用特定的类名。而不是`.listings-layout.bigger`，应使用类似`.listings-layout.listings-bigger`的规则。考虑将来使用 ack/grep 搜索您的代码。

+   在命名选择器时，请使用小写字母并用连字符分隔单词。避免使用驼峰命名法和下划线。使用描述元素样式的可读性选择器。

+   属性选择器应该在值周围使用双引号。避免使用过度限定的选择器；`div.container`可以简单地表示为`.container`。

+   ID 应该保留给 JavaScript 使用。除非有非常好的理由，所有 CSS 都应该附加到类而不是 ID。当犹豫时，请使用类名。这可以防止目标混乱，让 CSS 开发者和 JS 开发者和平共处。如果必须使用 ID 选择器（`#id`），确保在规则声明中不超过一个。

## 单位[](#units)

### 测量[](#measurements)

+   使用**rem**单位设置字体大小，并提供像素回退。可以通过以下 mixin 实现：

    ```
    @mixin font-size($sizeValue: 1.6) {
      font-size: ($sizeValue * 10) + px;
      font-size: $sizeValue + rem;
    }
    ```

+   将 HTML 字体大小设置为`10px`，以确保`0.1rem`等于`1px`。

    ```
    html {
      font-size: 10px;
    }
    ```

+   使用**em**单位进行定位。

+   在布局组件相对于彼此保持关系时（例如占屏幕 75% 的主内容区域和占 25% 的侧边栏），请使用**百分比**。

    ```
     .panel-a {
      width: 25%;
    }

    .panel-b {
      width: 75%;
    }
    ```

+   当测量不应基于用户设置的字体大小或浏览器缩放时，使用**px**单位。

    ```
     selector {
      border-width: 55px;
    }

    selector {
      border-width: 2px;
    }
    ```

+   对于`line-height`，请不要使用带单位的值，因为这将从`font-size`继承值。

+   在 em 单位中使用多达 10 位小数以确保准确性。

    ```
     .body_copy {
      @include rem-font-size(1.4);

      line-height: 1.8;
    }

    .container {
      height: 12em;
      margin-left: 10.6666666667em;
      width: 82.5%;
    }
    ```

+   不要在零值上使用单位。

    ```
     width: 0px;

    width: 0;
    ```

+   在尺寸、边距、边框、填充和排版中始终使用单位值。

    ```
     border-width: 12;

    border-width: 12px;
    ```

### 颜色[](#colors)

+   首先使用**十六进制**表示法，然后是**rgb(a)**或**hsl(a)**。

+   可接受三位数和六位数的十六进制表示法。

+   使用十六进制表示颜色时，全部使用小写字母。

+   使用 HSL 或 RGB 表示法时，逗号后始终添加一个空格，括号和内容之间不加空格。

```
 color: #FFF;
color: rgb( 255, 0, 0 );

$light: #fff;
color: $light;

$primary: #fe9848;
color: $primary;

$secondary: rgba(255, 100, 255, 0.5);
color: $secondary;
```

+   如果使用 rgba 规则，请在十六进制中包含一个回退值。

    ```
     .illustration {
      background-color: #eee; 
      background-color: rgba(221, 221, 221, 0.75);
    }
    ```

## [变量](#variables)

+   在以下情况下创建新变量：

    +   值重复两次。

    +   值可能会至少更新一次。

    +   所有值的出现都与变量相关（例如不是巧合）。

+   在构建将用于多个项目的 scss 时，使用 `!default` 标志允许覆盖。

    ```
    $baseline: 1em !default;
    ```

+   `!global` 标志只能在从本地作用域覆盖全局变量时使用。

+   整个 scss 代码库中的变量应该放在它们自己的文件中。

+   在声明颜色变量时，不要根据颜色内容命名。

    ```
     $light_blue: #18f;
    $dark_green: #383;

    $primary: #18f;
    $secondary: #383;
    $neutral: #ccc;
    ```

+   在根据上下文命名变量时要小心。

    ```
     $background_color: #fff;
    ```

+   不要在变量名中使用维度变量的值。

    ```
     $width_100: 100em;

    $width_lg: 100em;
    ```

+   为所有使用的 z-index 命名变量。

+   对每个使用的 z-index 使用一个 z-index 变量，并可能为 z-index 使用的地方创建一个单独的变量别名。

    ```
    $z_index-neg_1: -100;
    $z_index-neg_2: -200;
    $z_index-1: 100;

    $z_index-hide: $z_index-neg_2;
    $z_index-bg: $z_index-neg_1;
    $z_index-show: $z_index-1;
    ```

### [响应式设计与断点](#responsive-design-and-breakpoints)

+   在样式表的顶部设置断点变量。这个功能已经内置在 Bourbon 中。

    ```
    $sm: new-breakpoint(min-width 0 max-width 40em $sm_cols);
    ```

+   使用变量设置查询，这样如果需要可以轻松调整。

+   将媒体查询放置在它们影响的类最近的位置。

+   在决定断点位置时，不要专注于设备，而要专注于内容；将断点变量名称相对于彼此命名。

    ```
     $iphone: new-breakpoint(min-width 0 max-width 640px 6);

    $small: new-breakpoint(min-width 0 max-width 40em 6);
    $medium: new-breakpoint(min-width 0 max-width 60em 6);
    ```

</main>
