<!--yml

类别：未分类

日期：2024-05-27 14:32:09

-->

# 并非所有自动化测试工具都支持 Web 组件中的 Shadow DOM - Manuel Matuzovic

> 来源：[`matuzo.at/blog/2024/automated-testing-tools-and-web-components`](https://matuzo.at/blog/2024/automated-testing-tools-and-web-components)

# 并非所有自动化测试工具都支持 Web 组件中的 Shadow DOM

发布于 02.01.2024

没有更多要说的了；这一切都在标题里。

许多自动化测试工具是你在页面上运行的 JavaScript 函数的集合。其中大多数依赖于查询 DOM。如果一个工具不考虑影子树，它只能捕获 Light DOM 中的无障碍错误，这可能会给您一种错误的安全感，并可能影响到您的用户。

这是另一个 不仅依赖自动化测试的原因。

这并不意味着如果你的网站包含 Web 组件就不应该使用自动化测试。你只需确保你正在使用的工具支持它们。一个好的方法是创建一个简单的组件，其中包含一些由连接到组件的 Shadow DOM 的节点引起的无障碍错误，就像下面的组件一样。

```
<main>
  <h1 id="h1">Testing a11y bugs in web components</h1>

  <a href="#el">Jump to elem in shadow</a> <!-- Broken skip link -->

  <some-bugs></some-bugs>
</main>

<script>
class SomeBugs extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({mode: 'open'});

    this.shadowRoot.innerHTML = `
    <style>:host { color: #efefef }</style> <!-- Low contrast. -->
    <h4 id="el">Heading</h4> <!-- Skipped heading. -->
    <img src="#test"> <!-- Missing alt. -->

    <button aria-labelledby="h1"></button> <!-- Broken ARIA reference/missing accessible name. -->
    <input type="text" id="input"> <!-- Missing label. -->
      `
  }
}

customElements.define('some-bugs', SomeBugs);
</script>
```

CodePen：[测试包含无障碍错误的 Web 组件](https://codepen.io/matuzo/pen/yLwYNYY?editors=1010)

如果你将这个组件添加到一个页面中，测试工具应该至少找到五个问题。我选择了五个流行的工具：[axe DevTools](https://www.deque.com/axe/)、[Google Lighthouse](https://developer.chrome.com/docs/lighthouse/overview/)、[ARC Toolkit](https://www.tpgi.com/arc-platform/arc-toolkit/)、[WAVE](https://wave.webaim.org/extension/) 和 [IBM Equal Access (IBM EAAC)](https://www.ibm.com/able/toolkit/verify/automated/)，并对这个组件运行了它们。

基于 axe-core 的 axe DevTools 和 Lighthouse 都找到了全部五个问题。

WAVE 在 Shadow DOM 中没有发现问题，但在 Light DOM 中报告了一个损坏的跳转链接。WAVE 和 ARC 是唯一检查损坏锚点链接的工具。可以说，因为这不是一个明显的无障碍问题或 WCAG 违规，但根据 Web Aim 的 100 万报告，被测试站点中包含跳转链接的站点中有 [六分之一的跳转链接](https://webaim.org/projects/million/#skip) 是损坏的。他们应该检查锚链接的目标是否存在。

IBM EA 发现了除了被跳过的标题之外的所有问题，这可以被视为一种不良实践，而不是一种违规，无论 DOM 的类型如何，IBM EA 都不会报告它，无论 DOM 的类型如何。

ARC Toolkit 找到了除了标题低对比度文本颜色之外的所有问题。

不同测试工具报告的问题的比较

| Bug | axe | WAVE | ARC | IBM EAAC |
| --- | --- | --- | --- | --- |
| 检测到的问题 | 5 | 1 | 5 | 4 |
| 损坏的标签/输入 | 是 | 否 | 是 | 是 |
| 损坏的 ARIA | 是 | 否 | 是 | 是 |
| 缺失的 alt | 是 | 否 | 是 | 是 |
| 被跳过的标题 | 是 | 否 | 是 | 否 |
| 低对比度 | 是 | 否 | 否 | 是 |
| 损坏的锚点链接 | 否* | 是 | 是 | 否* |

*不是直接的无障碍问题，但应该进行检查。

当我发布这篇文章时，ARC 不支持 Shadow DOM，但自 2024 年 5 月初以来，使用版本 5.7.0 开始支持了。我不确定测试 Shadow DOM 是否在 WAVE 的路线图上，但在此期间，我建议不要将该工具用于包含 Web 组件的站点，或者使用其他工具进行双重检查。

同样适用于 HTML 验证器。例如，官方的 [W3C 验证器](https://validator.w3.org/) 不支持 Shadow DOM。

## 更新 04.01.24

添加了有关 ARC 计划在 2024 年初支持 Shadow DOM 的信息。

## 更新 08.01.24

添加有关损坏的跳转链接错误的说明。

## 更新 17.05.24

从版本 5.7.0 开始，[Arc Toolkit 现在支持 Shadow DOM](https://www.tpgi.com/arc-toolkit-5-7-0/)。
