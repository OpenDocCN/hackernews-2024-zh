<!--yml

category: 未分类

date: 2024-05-29 12:42:47

-->

# 创建交互式绘图艺术装置 | Lostpixels.io

> 来源：[https://lostpixels.io/writings/building-interactive-plotter-art](https://lostpixels.io/writings/building-interactive-plotter-art)

## 渲染与性能

当我制作[绘图艺术](/browse/plotter)时，我使用[P5.JS](https://p5js.org/)和[P5-SVG 插件](https://github.com/zenozeng/p5.js-svg)将我的图形渲染为 SVG。这种工作流允许我将文件下载为矢量图。然后，我可以上传到[Saxi](https://github.com/nornagon/saxi)，这是一个控制我的 Axidraw 画笔绘图机器人的实用程序。在编码艺术与画笔绘图机器人绘制之间的转换几乎是无缝的。

使用这种方法，以每秒 60 次更新 DOM 并响应用户输入开始变得缓慢了。我发现每秒 60 次更新 DOM 添加新的 SVG 元素并不理想。因此，我回到了 P5.JS 使用的传统渲染方法，即利用 HTML 画布。这立即解决了所有帧率问题，并使应用程序在旋转 MIDI 控件时感觉更加流畅。

为了同时具有画布的性能和 SVG 的兼容性，我创建了两个 P5.JS 实例，分别针对每个渲染目标。在将应用程序传输到云作业管理器之前，我只渲染了矢量实例。

我需要构建一个应用程序 UI 来显示帮助提示、介绍页面以及一些提交步骤。为此，我选择了[Svelte](https://svelte.dev/)。尽管我可以在这里使用 React，但我喜欢 Svelte 带来的易用性。在这一层面上，React 感觉过于笨重，因为我不担心组件重用或应用程序渲染性能。尽管如此，React 最终还是成功地进入了技术栈。
