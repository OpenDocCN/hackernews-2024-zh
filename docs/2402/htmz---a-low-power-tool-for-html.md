<!--yml

类别：未分类

日期：2024-05-27 15:00:10

-->

# htmz - 一个低功率工具用于处理 HTML

> 来源：[https://leanrada.com/htmz/](https://leanrada.com/htmz/)

# =>htmz>

*一个低功率工具用于处理 HTML*

**htmz** 是一个极简的 HTML 微框架，用于创建交互式和模块化的 Web 用户界面，具有**纯 HTML** 的简单易用性。[[GitHub]](https://github.com/Kalabasa/htmz)

## 纯🍦

使用纯 HTML。没有超集。没有 hz- ng- hx- v- w- x-；没有特殊属性。没有 DSL。没有 <custom-elements>。*只是纯 HTML*。

## 轻量级🪶

**总共 166 字节。** 零依赖。零 JS 捆绑包加载。甚至不需要后端。*只需内联 HTML 片段*。

## 无过滤器⚡

没有 preventDefaults。没有隐藏层。真实的 DOM，真实的交互。没有 VDOM，没有点击侦听器。没有 AJAX，没有 fetch。*不重新发明浏览器*。

**简而言之，htmz** 允许你根据请求交换页面片段，使用纯 HTML。

想象点击一个链接，但不是重新加载整个页面，而是*仅更新页面的相关部分*。

htmz 是一个*实验*，灵感来自于 [htmx](https://htmx.org/)、[Comet](https://en.wikipedia.org/wiki/Comet_(programming))、‘HTML As The Engine Of Application State’[[1]](https://en.wikipedia.org/wiki/HATEOAS)[[2]](https://htmx.org/essays/hateoas/) 等类似的 Web 应用架构。

## 演示

查看这些演示，了解 htmz 的功能！

🐙 从上面选择一个示例！

另请参阅：[扩展 🍱](extensions/)

## 安装

只需将以下片段直接复制到您的页面中：

```
<iframe hidden name=htmz onload="setTimeout(()=>document.querySelector(contentWindow.location.hash||null)?.replaceWith(...contentDocument.body.childNodes))"></iframe>
```

对于 **npm 爱好者**，请使用以下 npm 命令来自动化*复制片段的简单过程*。对于最大的 npm 乐趣，此 npm 包包含 25 个额外依赖项！

```
npm install --save-dev htmz
npx htmzify ./path/to/my/index.html
```

对于 **黑客**，你可以从开发版本（未混淆的）开始：[htmz.dev.html](https://github.com/Kalabasa/htmz/blob/master/htmz.dev.html)

## 基本使用

要调用 htmz，您需要一个具有以下属性的超链接（或表单）：

1.  href（或动作）指向的 **资源 URL** `href="/flower.html⋯`

1.  在 href 中继续：**目标 ID 选择器** `⋯#my-element"`

1.  并带有这个值的 **目标** 属性 `target=**htmz**`

```
 <a href="*/flower.html***#my-element**" target=**htmz**>Flower</a>
```

尽管这看起来像是滥用 URL 片段（确实是），但在这种情况下 URL 片段没有其他用途，因此它被重新用作目标 ID 选择器。而且它已经看起来像是一个 CSS ID 选择器。

**⚠ 重要提示：** 加载的内容**替换**了所选目标。一开始可能不直观，但 htmz 不会将内容插入目标中。理由是替换是一种更强大的操作。通过替换，你可以*替换*、*删除*（替换为空）、*插入到*（替换为与原始容器相同的容器）。

## 它确切地做什么？

htmz 只做一件事。

**在页面上的 *任意元素* 上请求加载 HTML。**

思考选项卡式用户界面（tabbed UIs）、双窗格列表-详细布局、对话框、原地编辑器等。

**此想法并非新颖。** 分割网页并各自加载自 1990 年代中叶以来就是一项技术。它们被称为[**帧**](https://www.w3.org/TR/html401/present/frames.html)，即如果rame s、<frame>s 和 <frameset>s。

**htmz 是 HTML 帧的泛化。** - 在页面内的任意 frame 中加载 HTML 资源。

在 `[如何运作]`这一部分中查看如何运行的详细信息。

## 示例

更多示例应用、组件化方法和不同语言的代码可在 `/examples` 目录中找到。启动示例服务器的步骤如下：

```
cd examples
./run_servers.sh
```

然后加载 `http://localhost:3000/`。

## 进阶使用

理所当然地，只有 `<a>` 和 `<form>`元素可以目标化和调用 htmz（到目前为止，HTML5 提供的）。这很合理；毕竟，这是语义相关的。不过，HTML 还提供了一两个与 htmz 搭配使用的特性。

### 按按钮的行动与目标

如果你希望在每个按钮的基础上更迭表单的行动，可以利用 `<button>` 的 `formaction` 属性。

```
<form action="*/default***#my-target**" target**=htmz>**
  <button>Default form action</button>
  <button formaction="*/button***#my-target**">
    Different button action
  </button>
  <button formaction="*/another-action***#another-target**">
    Another action
  </button>
</form>
```

### 默认目标值

厌倦了在每条链接和形式上添加 `target=htmz`？

使用 [base](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/base) 元素，**设置 htmz 作为所有相对链接的默认目标**。请在页面顶部添加这一配置。

```
<base target**=htmz>**
```

### 清洁目标值

不喜欢 `target=htmz` 这种形式？想要使用实际目标作为值呢？

我们可以实现一项巧妙策略，使你可以在目标属性本身编写目标 ID 选择器！使用这种方式：

```
 <a href="*/flower.html*" target="**#my-element**">Flower</a>
```

关键在于添加具有*匹配名字*的 iframe，并相应地修改 htmz 配置。

```
<iframe hidden name="**#my-element**" onload="htmz(this)"></iframe>
<script>
  function htmz(frame) {
    document.querySelector(frame.name) 
      ?.replaceWith(...frame.contentDocument.body.childNodes);
  }
</script> 
```

你甚至可以通过[自动化生成匹配源 iframe](https://github.com/Kalabasa/htmz/blob/master/examples/cf_clean_target_tabs/worker.js)。

### 支持在新标签页中打开链接

用户如何在新标签页中打开 htmz 链接呢？实际上，他们就是加载自定义设计、格式切除你的页面片段，将其独立加载在单独的页面上！

通过使用[**Sec-Fetch-Dest**](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-Fetch-Dest) 配置，遇到以下情况时，你可以[回退到完整页面](https://github.com/Kalabasa/htmz/blob/master/examples/php_new_tab_detection/content.php)。这条配置让服务器了解到请求的目标，并适当地呈现片段或完整页面。

### 脚本化/互动性

若需超越请求-响应模型的互动性，可以尝试 htmz 的伴生脚本语言 **JavaScript**。抱歉，我指的 **JavaScript** 正好是一种内联脚本语言，旨在增强 HTML 的互动性。

htmz 并不妨碍你使用 JS 或 UI 库来增强交互。例如，你可以只用[vanillaJS](http://vanilla-js.com/)就增强单个表单控件，但仍然可以通过 htmz 正常提交包括单表按钮在内的所有常规 HTTP 表单。

不过，htmz 是高度可扩展的！

## 可扩展性

具备高级选择器？有错误处理？需要多重目标吗？别担心，英雄已准备好拯救你。

这里是[片段的开发版本](https://github.com/Kalabasa/htmz/blob/master/htmz.dev.html)。根据你的需要随意进行修改和扩展。你是程序员，对吧？

```
<script>
  function htmz(frame) {
    **// Write your extensions here**

    setTimeout(() =>
      document
        .querySelector(frame.contentWindow.location.hash || null)
        ?.replaceWith(...frame.contentDocument.body.childNodes)
    );
  }
</script>
<iframe hidden name=htmz onload="htmz(**this**)"></iframe>
```

**现在可以在[扩展 🍱 页面](extensions/)找到预写的扩展。**

## 常见问题

### 它是如何工作的？

htmz是一个名为"htmz"的iframe。你通过target=htmz将一个URL加载到iframe中来调用htmz。通过使用iframe，我们依赖浏览器的原生能力来获取URL并解析HTML。加载HTML资源后，我们通过onload处理程序获取结果的DOM。

htmz本质上是一个**代理目标**。

就像代理服务器将请求转发到某个指定服务器一样，htmz代理将响应转发到某个指定的目标。

```
 <a href="*/flower.html*" target="**#my-element**">Flower</a>

<a href="*/flower.html***#my-element**" target=**htmz**>Flower</a>
```

当你将一个URL加载到htmz iframe中时，onload处理程序会启动。它从URL哈希片段中提取你的目标ID选择器，然后将iframe的内容（现在包含加载的HTML资源）移植到你指定的目标中。

htmz只有在调用时才会运行。它不会持续解析你的DOM并扫描特殊属性或语法，也不会在你的DOM中附加监听器。它是一个代理而不是VPN。

### 所以这只是另一个JavaScript框架？

哦天啊！不是那个脏话！！！

更严肃地说，我会说它不像是一个纯JS的，更像是一个HTML微型f*******k。它确实使用了JS，但只是必要的最小限度。

### htmz是一个库还是一个框架？

htmz只是一个片段。✂️

### htmz是什么意思？

HTMZ代表***H**tml with **T**argeted **M**anipulation **Z**ones*（带有定向操纵区的HTML）。

### 这是一个笑话吗？

最初只是一个*“我真的需要htmx吗？我不能用当前的Web做加载-链接到目标的事情吗？听起来很像框架。”*，结果却变成了这样。

所以，这并不完全是一个笑话，而是对htmx的回应。我想尝试htmx。起初听起来很棒（*为什么你只能替换整个屏幕？*），然后我看到它是16kB的JavaScript。嗯。然后到处都是特殊的语法。嗯。我不想学习一整套新的指令和特定于这些指令的图灵完备DSL。

不管是不是笑话，htmz作为一个库似乎表现不错。尽管其体积小，但感觉功能强大。（但实际上是浏览器在进行大部分工作！）尽管如此，也存在一些限制。

### 有哪些限制？

主要的直接限制是每个响应只有一个目标。不过，这可以通过编写扩展来解决。;)

一个更普遍但经典的限制是请求-响应模型。Web 1.0模型及其带来的负担。比如每次交互都会有一个往返延迟，每次点击都会有一个浏览器历史记录条目等。

Web 1.0模型可能意味着将更多的UI逻辑放在Web服务器中。这可能是好事，也可能是坏事，因为它可能导致UI逻辑的统一化或分散化，分别降低或增加复杂性。这实际上取决于你的目标和风格。

最后，这不是一个模板化库！没有HTML导入或客户端包含。
