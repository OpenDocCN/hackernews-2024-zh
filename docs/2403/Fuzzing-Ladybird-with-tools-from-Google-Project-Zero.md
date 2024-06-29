<!--yml

category: 未分类

date: 2024-05-27 15:01:01

-->

# 使用 Google Project Zero 的工具对 Ladybird 进行模糊测试

> 来源：[https://awesomekling.substack.com/p/fuzzing-ladybird-with-tools-from](https://awesomekling.substack.com/p/fuzzing-ladybird-with-tools-from)

虽然 [Ladybird](https://ladybird.dev/) 在处理格式良好的网页内容方面做得不错，但我认为用一些安全研究工具来测试它并查看可能出现的问题也是有用的。因此，今天我们将使用来自 [Google Project Zero](https://googleprojectzero.blogspot.com/) 的“**[Domato](https://github.com/googleprojectzero/domato)** [🍅](https://github.com/googleprojectzero/domato)” DOM 模糊器来对 Ladybird 进行压力测试并修复一些问题。

这个工作原理是，Domato 生成带有大量几乎有效但奇怪的 HTML、CSS 和 JavaScript 的随机化网页。然后我加载这些页面到 Ladybird 的调试版本中，并观察发生了什么。

Domato 生成大约 500 KiB 大小的 HTML 页面，充斥着“有趣”的 JS、CSS 和 HTML，可以惊喜和取悦您的浏览器引擎！

Domato 的 README 中自豪地宣称发现了所有主要浏览器中的大量漏洞，因此我确信它也会在我们的浏览器中找到一些。我们开始吧！

毫不奇怪，不到一秒钟就找到了第一个问题！由 Domato 生成的输出实际上是 562 KiB，但我能将其减少到以下内容：

```
<body>
<script>
let mfrac = document.createElement("mfrac");
mfrac.appendChild(document.createElement("th"));
document.body.appendChild(mfrac);
</script>
```

我为了这次测试编译了带有 UBSAN（未定义行为 SANitizer）的 Ladybird，并获得了以下非常有用的跟踪输出：

```
**LibWeb/HTML/HTMLTableCellElement.cpp:55:36: runtime error: member call on null pointer of type 'Web::DOM::Node'** #0 0x7e9bfa1610dd in table_containing_cell LibWeb/HTML/HTMLTableCellElement.cpp:55:36
#1 0x7e9bfa1610dd in Web::HTML::HTMLTableCellElement::apply_presentational_hints(Web::CSS::StyleProperties&) const LibWeb/HTML/HTMLTableCellElement.cpp:101:33
#2 0x7e9bf97c22d1 in Web::CSS::StyleComputer::compute_cascaded_values(Web::CSS::StyleProperties&, Web::DOM::Element&, AK::Optional<Web::CSS::Selector::PseudoElement::Type>, bool&, Web::CSS::StyleComputer::ComputeStyleMode) const LibWeb/CSS/StyleComputer.cpp:1448:17
#3 0x7e9bf97e5899 in Web::CSS::StyleComputer::compute_style_impl(Web::DOM::Element&, AK::Optional<Web::CSS::Selector::PseudoElement::Type>, Web::CSS::StyleComputer::ComputeStyleMode) const LibWeb/CSS/StyleComputer.cpp:2231:5
#4 0x7e9bf97e447e in Web::CSS::StyleComputer::compute_style(Web::DOM::Element&, AK::Optional<Web::CSS::Selector::PseudoElement::Type>) const LibWeb/CSS/StyleComputer.cpp:2202:12
#5 0x7e9bf9a7e60c in Web::DOM::Element::recompute_style() LibWeb/DOM/Element.cpp:575:64
```

这是一个传统的空指针解引用错误！

结果证明，我们实现了 `<th>` 和 `<td>` 元素，假设在 DOM 树中总是有一个 `<table>` 元素位于其上方。我们可能会这样认为，因为 HTML 解析器不允许以下内容：

`<mfrac><th>`

如果您在符合规范的浏览器中加载上述标记，将创建一个单独的 `<mfrac>` 元素，里面没有任何内容。

但是，当使用 JavaScript API 手动创建 DOM 节点时，您可能会违反解析器必须遵循的一些规则，并确实将 `<th>` 放在 `<mfrac>` 中！

这是有问题的函数：

```
HTMLTableElement const& table_containing_cell(HTMLTableCellElement const& node)
{
    auto parent_node = node.parent();
    while (!is<HTML::HTMLTableElement>(parent_node))
        parent_node = parent_node->parent();
    return static_cast<HTML::HTMLTableElement const&>(*parent_node);
}
```

它用于实现 `<table border=3>` 和 `<table padding=5>` 的古老特性，这些特性将 CSS 边框和填充应用于每个表格单元格，而不仅仅是表格框本身。

解决方法很简单，不再假设 `<th>` 和 `<td>` 元素总是有包含它们的 `<table>` 元素在祖先链中。我们不需要 `table_containing_cell` 辅助函数，而可以简单地替换为：

```
auto const& table_element = table_containing_cell(*this);
```

使用这个：

```
auto const table_element = first_ancestor_of_type<HTMLTableElement>();
if (!table_element)
    return;
```

我们已经解决了问题 #1！[(在此提交了修复。)](https://github.com/SerenityOS/serenity/commit/b9bacb3ff49ba675f9875434d646fb1b34d2c0df)

我们继续执行模糊测试程序，不到 1 秒钟，又遇到了一个新问题。Domato 的输出是 472 KiB，但最终精简到了这样：

```
<script>
    var parser = new DOMParser();
    var doc = parser.parseFromString("", "text/html");
    var body = doc.createElement("body");
    body.onblur = null;
</script>
```

当在 Ladybird 中打开时，我们就会像这样失败：

```
VERIFICATION FAILED: m_ptr at LibJS/Heap/GCPtr.h:174
#1  JS::GCPtr<Web::HTML::Window>::operator* at LibJS/Heap/GCPtr.h:174
#2  Web::DOM::Document::window at LibWeb/DOM/Document.h:320
#3  Web::HTML::HTMLBodyElement::global_event_handlers_to_event_target at LibWeb/HTML/HTMLBodyElement.cpp:104
#4  Web::HTML::GlobalEventHandlers::set_onblur at LibWeb/HTML/GlobalEventHandlers.cpp:24
#5  Web::Bindings::HTMLElementPrototype::onblur_setter at LibWeb/Bindings/HTMLElementPrototype.cpp:1153
```

事情出错的原因是因为`<body>`元素上的`onfoo`事件处理程序属性的特殊行为。为了与古老的网络内容兼容，对`document.body.onfoo`的赋值必须转发到`window.onfoo`。然而，通过`DOMParser`创建的文档*没有window对象！*

结果证明，我们在我们的实现中误解了这个细节，并构造了一个内部对象模型，就好像每个文档总是有一个窗口。糟糕！

修复方法是使`Document::window()`返回一个可空值，然后在几百个地方处理null。在无窗口的文档中分配`document.body.onblur`时，我们现在像其他浏览器一样不执行任何操作。

[(在此提交了修复。)](https://github.com/SerenityOS/serenity/commit/b98a2be96bf03562ec21e80b48d4b3b97ad74cd6)

现代浏览器必须同时支持SVG在HTML中内联和作为外部图像格式。这增加了一系列新的边界情况和有趣的交互。

例如，SVG允许声明引用其他渐变以继承其颜色的渐变。事实证明，我们没有考虑渐变引用自身的情况：

```
<svg>
    <linearGradient id="oops" href="#oops"/>
    <rect fill="url(#oops)" />
</svg>
```

上面的SVG会导致我们的实现永远循环，因为它试图跟随引用渐变的链。真是糟糕！

显而易见的修复方法是：如果当前渐变引用自身，则停止跟随链接渐变的链。然而，这并不包括跨多个步骤的引用循环，就像这样：

```
<svg>
    <linearGradient id="lol" href="#lmao"/>
    <linearGradient id="lmao" href="#even"/>
    <linearGradient id="even" href="#lol"/>
    <rect fill="url(#lol)" />
</svg>
```

为了正确处理循环引用，我们必须追踪我们访问过的所有渐变，并在遇到我们之前访问过的渐变时停止跟随链。

有趣的是，Firefox实际上在他们的开发者控制台中抱怨这种渐变：

[(在此提交了修复。)](https://github.com/SerenityOS/serenity/commit/2e0297d7035a807d4a18c89c8b310ec06bef005d)

这个很有趣：

```
<iframe></iframe>
<script>
    window.onload = function() {
        let iframe = document.querySelector("iframe")
        let iframeWindow = iframe.contentWindow;
        iframe.remove();
        iframeWindow.getSelection();
    }
</script>
```

上面的测试会导致崩溃，如下所示：

```
**LibWeb/HTML/WindowProxy.cpp:161:136: runtime error: reference binding to null pointer of type 'const BrowsingContext'** #0 0x77367675cadf in Web::HTML::WindowProxy::internal_get(JS::PropertyKey const&, JS::Value, JS::CacheablePropertyMetadata*) const LibWeb/HTML/WindowProxy.cpp:161:5
#1 0x773670c4fcca in JS::Bytecode::get_by_id(JS::VM&, AK::DeprecatedFlyString const&, JS::Value, JS::Value, JS::Bytecode::PropertyLookupCache&) LibJS/Bytecode/CommonImplementations.h:105:18
#2 0x773670c182d5 in JS::Bytecode::Op::GetById::execute_impl(JS::Bytecode::Interpreter&) const LibJS/Bytecode/Interpreter.cpp:1088:28
#3 0x773670bc8e1f in execute LibJS/Bytecode/Op.h:1898:9
#4 0x773670bc8e1f in JS::Bytecode::Interpreter::run_bytecode() LibJS/Bytecode/Interpreter.cpp:409:38
```

结果事实证明，这实际上是HTML规范中的一个bug！当iframe从DOM中删除时，其内容文档将与其浏览上下文分离。然而，在获取或设置窗口对象的属性时，我们运行一个名为**“检查是否应报告两个浏览上下文之间的访问”**的规范算法，该算法检查“访问者”和“被访问者”窗口的浏览上下文。规格错误地假定在属性访问时，两个窗口都具有关联的浏览上下文。

[我已经在HTML规范](https://github.com/whatwg/html/issues/10192)上提出了一个问题，并在此期间通过添加了一个空检查来修复了Ladybird。

在Ladybird工作时，发现规范bug实际上是我最喜欢的事情之一。通过提交bug报告或修复，这使我们能够改进规范，造福每个人。

[(在此提交了修复。)](https://github.com/SerenityOS/serenity/commit/ad843b6e4a4404a4897b587711f9ce97a4504ed0)

打开下一页时，它就停在那里，占用100%的CPU，加载不完毕。

这是简化后的问题：

```
<div id="one"></div><div id="two"></div>
<script>
    two.before(one);
</script>
```

在实现`before()`方法时出现了错误，需要找到`<div id=”two”>`的第一个前兄弟节点，而不是它的参数之一（即在这种情况下是`<div id=”one”>`）。在实现中我们有以下循环错误：

```
while (auto previous_sibling = node->previous_sibling()) {
    // check if previous_sibling is one of the arguments
}
```

问题在于我们一直在获取`node->previous_sibling`，而不是继续通过`previous_sibling->previous_sibling`跟随兄弟链。

这是我修复它的方法：

```
for (auto sibling = node->previous_sibling(); sibling; sibling = sibling->previous_sibling()) {

    // check if previous_sibling is one of the arguments

}
```

[(修复提交在此处。)](https://github.com/SerenityOS/serenity/commit/35f359c51c787153cd17c7289664ab87146241ea)

这可能会持续相当长时间，所以我们就这样吧。我们找到了五个真正的错误，其中一个是规范错误，并成功修复了所有问题。

尽管事情基本按我预期的进行，但当面对奇怪和意外的输入时，我们仍然很快就崩溃了。像Domato这样的模糊测试工具对于希望使他们的软件更加健壮的人来说是一个很好的资源。

下一步将是让Ladybird能够处理持续的模糊输入。一旦它相对稳定，我们可以开始在云中自动运行它，希望能发现更多问题。
