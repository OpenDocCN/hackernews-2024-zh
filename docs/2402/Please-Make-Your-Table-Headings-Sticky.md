<!--yml

category: 未分类

date: 2024-05-29 13:21:15

-->

# 请使您的表头固定

> 来源：[https://btxx.org/posts/Please_Make_Your_Table_Headings_Sticky/](https://btxx.org/posts/Please_Make_Your_Table_Headings_Sticky/)

我经常在网上遇到大数据集或表格布局。当这些表格包含数百行内容时，一旦开始滚动，问题就开始变得棘手...

<https://btxx.org/posts/Please_Make_Your_Table_Headings_Sticky/not-fixed-header-tables.mp4>

您的浏览器不支持视频标签。

看，表头就消失了！现在，如果我滚动到第300项（例如），我还记得每列数据与什么相关吗？如果这是我第一次查看此表格 - 可能不记得。幸运的是，我们可以通过少量的CSS来修复这个问题（没有刻意的意思！）。

看看这个：

<https://btxx.org/ikiwiki/git/fixed-header-tables.mp4>

您的浏览器不支持视频标签。

相当不错，对吧？它看起来可能像魔术，但实际上实现起来非常容易。你只需要在你的`thead`上添加两个CSS属性：

```
position: sticky;
top: 0; 
```

这就是全部！最重要的是，`sticky`属性全球支持率约为[~96%](https://caniuse.com/?search=sticky)，这意味着这不是一些“尖端”属性，可以安全支持大量浏览器。更不用说为您的最终用户带来的改进体验了！

您可以在[CodePen示例](https://codepen.io/bradleytaunt/pen/bGZyJBj)上查看此表格的实时演示。

如果你觉得这很有趣，请随时查看我另一篇关于表格的帖子：[使用最少的CSS使表格响应式](https://btxx.org/posts/tables/)
