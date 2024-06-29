<!--yml

分类: 未分类

date: 2024-05-27 14:59:50

-->

# 生成字母表 — Amy Goodchild

> 来源：[https://www.amygoodchild.com/blog/generating-the-alphabet](https://www.amygoodchild.com/blog/generating-the-alphabet)

我围绕中心点 mid_x 和 mid_y 定义了我的字母。事后看来，从左下角点开始工作可能会更好，我随时会进行调整，以帮助改善目前不一致的字距。

在一个 Letter 类中，我定义了像 x 高度、大写字母高度等关键位置，它们与字体大小和中点有关。例如，从基线到大写字母顶部的整体高度等于字体大小。从 y_mid 到 y_x 是整体高度的三分之一。

我还定义了一些小的距离，可以根据高度调整一个点。
