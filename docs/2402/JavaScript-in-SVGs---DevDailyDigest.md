<!--yml

类别：未分类

日期：2024-05-27 14:40:06

-->

# JavaScript 在 SVG 中的使用 - DevDailyDigest

> 来源：[https://www.devdailydigest.tech/javascript-in-svgs/](https://www.devdailydigest.tech/javascript-in-svgs/)

最近，我在 Web 开发中发现了一个隐藏的宝藏——**JavaScript** 与 **可伸缩矢量图形（SVG）** 的结合。这里是 JavaScript 在 SVG 中的魔法一瞥。

是的，这是一个 SVG，它从 API 获取加密货币价格并实时更新 SVG。有时可能无法正常工作，因为 API 的速率限制。但你明白了吧。

再来一个，Chuck Norris 笑话生成器。

如果你想在时间耗尽之前获得笑话，点击 SVG。老实说，笑话不怎么样，但 SVG 很酷。

## 与 CDATA 的邂逅

如果你打开开发工具检查，你可能已经注意到脚本中的 `<![CDATA[...]]>` 标签。

这是 CDATA（字符数据）——一种在 XML 文档中包含文本块（如 JavaScript 代码）而无需转义特殊字符的方法。它确保内容不会被误解为 XML，非常适合在 SVG 中嵌入 JavaScript。

注意它是不适用于 HTML，而是用于 XML。那么为什么它在 SVG 中使用？答案很简单——SVG 是基于 XML 的格式。

在独立的 SVG 中，您需要使用 `xmlns="http://www.w3.org/2000/svg"` 定义命名空间，而在这里，您需要使用 CDATA 来防止 JavaScript 中的各种字符，例如 `<` 被解释为标签的开始，以及 `&`。如果您嵌入具有 JavaScript 的 HTML 中的 SVG，您不需要 CDATA。否则，如果它是一个独立的 SVG，在浏览器中打开，为了安全起见，您需要 CDATA。

```
<script type="text/ecmascript">
  <![CDATA[
    // Your JavaScript code here
  ]]>
</script>
```

## 为什么要在 SVG 中使用 JavaScript？

我不确定这种用例能够满足什么需求，而不能简单地使用 JavaScript 和 HTML 与 SVG 完成。但了解和实验它是很酷的事情。

这在创建动态图形如图表、图表方面可能会有所帮助。它还可以用来制作交互式 SVG，响应用户操作，获取外部数据，甚至动画效果。

这些只是我对 JavaScript 在 SVG 中潜力的一些想法。可能还有很多我不知道的东西。包括一些安全问题？？

关于[MDN 网页文档](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/script)的更多信息。
