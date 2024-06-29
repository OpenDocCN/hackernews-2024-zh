<!--yml

类别：未分类

日期：2024-05-27 13:33:32

-->

# Tailwind CSS 的隐藏宝石 | Rails Designer

> 来源：[https://railsdesigner.com/hidden-gems-tailwind/](https://railsdesigner.com/hidden-gems-tailwind/)

Tailwind CSS 使得设计和创建 Web UI 变得轻而易举。难怪它是我过去几年里每个（基于 Rails 的）SaaS 应用的首选工具。[Rails Designer](https://railsdesigner.com/) 的核心工具也毫不意外。

我想列出一些不太知名或鲜为人知的功能，为未来的我，但更重要的是为了让你能更充分地利用 Tailwind CSS 提供的一切。

## 更改单选框/复选框颜色

使用`accent-*`工具来改变复选框或单选按钮的颜色。

```
<label>
  <input type="checkbox" checked> Default
</label>
<label class="ml-3">
  <input type="checkbox" class="accent-pink-600" checked> The sky is the limit
</label> 
```

## 同级修饰符

`peer`修饰符允许根据**同级状态**样式化一个元素。好吧，现在你在说些无稽之谈！我们来看一个例子？

```
<input type="email" placeholder="support@railsdesigner" value="support@" class="w-full px-3 py-1 text-sm rounded text-slate-700 peer bg-slate-200 ring-1 ring-offset-0 ring-slate-200"/>
<span class="invisible !mt-0.5 text-sm text-rose-600 peer-invalid:visible">
  Please provide a valid email address.
</span> 
```

请提供一个有效的电子邮件地址。

它通过在一个元素上添加`peer`，然后立即使用任何`peer-*`实用程序的下一个元素，可以与每个伪类修饰符一起工作，如`peer-hover`、`peer-required`和`peer-disabled`。一旦了解了这个实用工具，您将发现有很多方法可以使用它，而不是 JavaScript。

## 样式文件输入

当您想要提升文件输入字段的外观时，请使用`file:`修饰符。

```
<input type="file" class="block w-full text-sm text-slate-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-sky-50 file:text-sky-700 hover:file:bg-sky-100 "/> 
```

它需要几个实用工具，但可以为你的文件输入添加大量的样式！

## 更改颜色不透明度

Tailwind CSS 中的每个颜色工具都允许附加`/n`不透明度修饰符。因此，这对于每个文本颜色、背景颜色和环形颜色都适用。

```
<button class="px-2 rounded bg-emerald-500/100">A</button>
<button class="px-2 rounded bg-emerald-500/75">B</button>
<button class="px-2 rounded bg-emerald-500/50">C</button> 
```

我经常使用它来让元素与具有渐变背景的背景更加融合。或者让颜色*刚刚好*。

## 结合字体大小和行高

类似地，您可以结合字体大小和行高。

你可能已经了解字体大小工具（`text-sm`，`text-base`等）和行高工具（相对：`leading-loose`和`leading-tight`，固定：`leading-4`和`leading-6`）。

*注意*：相对行高是基于当前字体大小的。而固定行高则是独立于当前字体大小设置的。

您可以将固定行高与字体大小`text-xl/8`组合使用，设置字体大小为`1.25rem`，行高为`2rem`。

## 在元素之间添加空白

这是我经常使用的一个方法，因为它是在元素和可选元素之间添加空间的绝佳方式。`gap-*`仅在实际元素之间添加空间，而不只是添加`margin-left`或`margin-right`。

我学得太晚了，`gap-*`不仅可以在网格中使用（如其名称所示），还可以在 flex 布局中使用。还有`gap-x-*`和`gap-y-*`，分别用于水平（左/右）或垂直（上/下）方向添加间距。

## 过渡悬停/活动状态

真正提升应用程序“感觉”的一些微过渡可以做到这一点。这是我经常添加的一些东西。

而不是`bg-sky-100 hover:bg-sky-200`。

我做的是：`bg-sky-100 transition hover:bg-sky-200`。

对于更加流畅的效果，可以像这样编写：`bg-sky-100 transition ease-in-out duration-300 hover:bg-sky-200`。

#### 获取最新的 Rails Designer 更新

如果您将其用于较大的卡片元素上，则值得添加 `delay-75`，以便当用户滚动并悬停在元素上时不触发过渡。

## 指针事件

这是我最近在为 [Rails Designer 的 NotificationComponent](https://railsdesigner.com/components/notifications/) 修复错误时偶然发现的一个问题。

如果您有一个元素位于另一个元素的“上方”（例如具有 NotificationComponent 的容器元素，您也见过这种情况）。

指针事件将触发子元素，并传递到“下方”目标的元素。

添加 `pointer-events-none`，您就完成了：我喜欢它！

## 截断多行文本

处理用户生成的内容时，您肯定会遇到破坏 UI 的情况。

`line-clamp-*` 实用工具对此很有帮助。 它在给定的数字处断开句子并添加省略号（`…`，请勿与`...`混淆）。

它将此 CSS 添加到元素中：

```
overflow: hidden;
display: -webkit-box;
-webkit-box-orient: vertical;
-webkit-line-clamp: 1; 
```

您可以使用从 1 到 6 的任何数字或 `none`。

## 将宽度和高度结合为正方形元素

以前，当您想要一个正方形元素时，比如 [AvatarComponent](https://railsdesigner.com/components/avatars/)，您需要同时添加 `w-*` 和 `h-*`。

现在，这些可以与设置宽度和高度相同值的 `size-*` 实用程序结合使用。

## 突出显示文本

通过添加 `selection:` 修饰符，您可以为应用程序增加一些额外的个性。 将其添加到您的 body 元素以在选择文本时调整每个页面。

```
selection:bg-sky-50 selection:text-sky-600 
```

就像在这个网站上一样（尝试选择文本）！

## 命名分组

您可以根据其父元素的状态（悬停、焦点、禁用）样式化元素。 它的工作方式如下：

```
<a href="#" class="group hover:bg-sky-500 hover:ring-sky-500">
  <h3 class="text-sm font-semibold text-slate-900 group-hover:text-white">
    New project
  </h3>
</a> 
```

但我几乎总是喜欢为我的分组命名。 就像这样：

```
<a href="#" class="group/link hover:bg-sky-500 hover:ring-sky-500">
  <h3 class="text-sm font-semibold text-slate-900 group-hover/link:text-white">
    New project
  </h3>
</a> 
```

记住语法花了我一些时间，但在进行更高级的 UI 工作时，它已被证明是一个有价值的默认选择。

## 添加自定义变体

您可以添加自己的变体来针对非常具体的事物。 让我们看看我在所有 Rails 应用程序中使用的变体（所有这些应用程序都使用 `turbo-frame`）。

```
{
  // …
  plugins: [
    // …
    function ({ addVariant }) {
      addVariant("turbo-frame", "turbo-frame[src] &")
    }
  ]
} 
```

现在您可以在*元素位于 `turbo-frame` 内时*以不同的样式显示元素。

```
<div class="px-4 py-2 turbo-frame:shadow-xl">
</div> 
```

想象一下，当作为页面查看时，元素没有投影效果，但当作为 turbo-frame 查看时（例如作为 [Modal](https://railsdesigner.com/components/modals/)），它会添加投影效果。 使元素真正可重复使用。

## 任意值

如果所有的 Tailwind CSS 实用工具都不足以满足您的需求，您可以使用*任何值*！ 我真的把它当作最后的手段，因为这仍然感觉像是违背了框架的初衷（尽管这是一个由框架支持的功能）。

您可以采用任何可用的实用工具并添加您自己的值，比如`text-[0.65rem]`（这是我在所有应用程序中都使用的东西——也许需要将其作为自定义变体？）。

现在我就讲到这里了。 您有没有看到不太常见的功能？ [请与我分享](mailto:support@railsdesigner.com)。
