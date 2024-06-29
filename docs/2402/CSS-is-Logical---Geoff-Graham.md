<!--yml

分类：未分类

date: 2024-05-27 14:46:34

-->

# CSS 是逻辑的 - Geoff Graham

> 来源：[https://geoffgraham.me/css-is-logical/](https://geoffgraham.me/css-is-logical/)

+   文档按照书写模式的方向流动。

+   元素以块和内联方向流动。

+   块和内联方向基于书写模式。

+   相对定位元素在文档流中保留了空间，而绝对定位元素则被拉出文档流。

+   屏幕以视口单位测量。

+   父元素以容器单位测量。

+   Em 和 rem 单位相对于它们的元素和根上下文进行评估。

+   设置 `z-index` 会为管理元素视觉重叠方式创建一个新的层叠上下文。

+   层叠 [组织选择器](https://css-tricks.com/the-c-in-css-the-cascade/) 成优先级图层，可以重新组织为自定义的层叠图层。

+   选择器通过特异性评分进行评估。

+   当两个选择器匹配时，应用的样式是它们在层叠中最低的那一个，当它们共享相同的来源时。

+   如果一个属性无效，则在匹配选择器中使用下一个匹配的属性。

+   [`width` 向上查找树，而 `height` 向下查找树。](https://geoffgraham.wpengine.com/width-looks-outward-height-looks-inward/)

+   根据浏览器是否支持 `@supports`，可以声明特定属性。

+   如果元素 `:is()` 特定选择器，不是 `:not()` 特定选择器，`:has()` 另一个元素，或者 `:where()` 满足特定条件，则可以选择该元素。

CSS 可能很奇怪，但它并不是不合逻辑的。
