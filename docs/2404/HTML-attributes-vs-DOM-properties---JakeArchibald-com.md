<!--yml

category: 未分类

date: 2024-05-27 13:32:33

-->

# HTML 属性 vs DOM 属性 - JakeArchibald.com

> 来源：[https://jakearchibald.com/2024/attributes-vs-properties/](https://jakearchibald.com/2024/attributes-vs-properties/)

属性和属性是*根本*不同的东西。您可以将同名的属性和属性设置为不同的值。例如：

```
<div foo="bar">…</div>
<script> const div = document.querySelector('div[foo=bar]');

  console.log(div.getAttribute('foo')); 
  console.log(div.foo); 

  div.foo = 'hello world';

  console.log(div.getAttribute('foo')); 
  console.log(div.foo); </script>
```

看起来越来越少的开发者知道这一点，部分要归功于框架：

```
<input className="…" type="…" aria-label="…" value="…" />
```

如果您在框架的模板语言中执行上述操作，您使用的是类似属性的语法，但在幕后，它有时会设置属性，与框架有关。在某些情况下，它将设置属性*和*属性作为副作用，但这不是框架的错。

大多数时候，这些区别并不重要。我认为开发者可以在不关心属性和属性之间的区别的情况下拥有长久和愉快的职业是件好事。但是，如果您需要在更低的 DOM 层次深入挖掘，了解这一点会有所帮助。即使您认为了解了区别，也许我会触及一些您没有考虑过的细节。因此，让我们深入探讨一下…

在我们深入讨论有趣的内容之前，让我们解决一些技术上的差异：

属性序列化为 HTML，而属性不会：

```
const div = document.createElement('div');

div.setAttribute('foo', 'bar');
div.hello = 'world';

console.log(div.outerHTML); 
```

因此，当您在浏览器开发者工具的元素面板中查看时，您只能看到元素上的属性，而不是属性。

为了在序列化格式中工作，属性值始终是字符串，而属性可以是任何类型：

```
const div = document.createElement('div');
const obj = { foo: 'bar' };

div.setAttribute('foo', obj);
console.log(typeof div.getAttribute('foo')); 
console.log(div.getAttribute('foo')); 

div.hello = obj;
console.log(typeof div.hello); 
console.log(div.hello); 
```

属性名称不区分大小写，而属性名称区分大小写。

```
<div id="test" HeLlO="world"></div>
<script> const div = document.querySelector('#test');

  console.log(div.getAttributeNames()); 

  div.setAttribute('FOO', 'bar');
  console.log(div.getAttributeNames()); 

  div.TeSt = 'value';
  console.log(div.TeSt); 
  console.log(div.test); </script>
```

然而，属性*值*是区分大小写的。

好的，这里开始变得模糊起来：

看看这个：

```
<div id="foo"></div>
<script> const div = document.querySelector('#foo');

  console.log(div.getAttribute('id')); 
  console.log(div.id); 

  div.id = 'bar';

  console.log(div.getAttribute('id')); 
  console.log(div.id); </script>
```

这似乎与文章中的第一个示例相矛盾，但以上之所以有效，仅因为 `Element` 具有一个 `id` 的 getter 和 setter，它“反映”了 `id` 属性。

当属性反映属性时，*属性*是数据的来源。当您设置属性时，它更新属性。当您从属性读取时，它读取属性。

出于方便，大多数规范将为每个定义的属性创建一个相应的属性等效项。在文章开头的示例中它不起作用，因为 `foo` 不是规范定义的属性，因此没有反映它的规范定义的 `foo` 属性。

[这是 `<ol>` 的规范](https://html.spec.whatwg.org/multipage/grouping-content.html#the-ol-element)。"内容属性"部分定义了属性，而"DOM 接口"定义了属性。如果您在 DOM 接口中点击 `reversed`，它会带您到这里：

> `reversed` 和 `type` IDL 属性必须反映相同名称的相应内容属性。[反映](https://html.spec.whatwg.org/multipage/common-dom-interfaces.html#reflect)。

但有些反射器更复杂…

好吧，这相对较小，但有时属性的名称与其反映的属性不同。

在某些情况下，它只是添加了你期望从属性中得到的大小写：

+   在 `<img>` 上，`el.crossOrigin` 反映了 `crossorigin` 属性。

+   在所有元素上，`el.ariaLabel` 反映了 `aria-label` 属性（这些 aria 反射器在2023年底变得跨浏览器。在此之前，你只能使用这些属性）。

在某些情况下，由于旧的 JavaScript 保留字，名称不得不更改：

+   在所有元素上，`el.className` 反映了 `class` 属性。

+   在 `<label>` 上，`el.htmlFor` 反映了 `for` 属性。

属性带有验证和默认值，而属性则没有：

```
const input = document.createElement('input');

console.log(input.getAttribute('type')); 
console.log(input.type); 

input.type = 'number';

console.log(input.getAttribute('type')); 
console.log(input.type); 

input.type = 'foo';

console.log(input.getAttribute('type')); 
console.log(input.type); 
```

在这种情况下，验证由 `type` 获取器处理。设置器允许无效值 `'foo'`，但当获取器看到无效值或无值时，它返回 `'text'`。

一些属性执行类型强制转换：

```
<details open>…</details>
<script> const details = document.querySelector('details');

  console.log(details.getAttribute('open')); 
  console.log(details.open); 

  details.open = false;

  console.log(details.getAttribute('open')); 
  console.log(details.open); 

  details.open = 'hello';

  console.log(details.getAttribute('open')); 
  console.log(details.open); </script>
```

在这种情况下，`open` 属性是一个布尔值，返回属性是否存在。设置器还会强制类型转换 - 即使设置器给出 `'hello'`，它也会转换为布尔值，而不是直接转到属性。

像 `img.height` 这样的属性会将属性值强制转换为数字。设置器将传入的值转换为数字，并将负值视为 0。

`value` 是一个有趣的例子。有一个 `value` 属性和一个 `value` 属性。但是，`value` 属性不反映 `value` 属性。相反，`defaultValue` 属性反映了 `value` 属性。

我知道，我知道。

实际上，`value` 属性不反映 *任何* 属性。这并不罕见，还有很多这样的情况（例如 `offsetWidth`、`parentNode`、复选框上的 `indeterminate` 以及其他许多情况）。

最初，`value` 属性延迟到 `defaultValue` 属性。然后，一旦通过 JavaScript 或用户交互设置了 `value` 属性，它就会切换到内部值。它就像大致这样实现的：

```
class HTMLInputElement extends HTMLElement {
  get defaultValue() {
    return this.getAttribute('value') ?? '';
  }

  set defaultValue(newValue) {
    this.setAttribute('value', String(newValue));
  }

  #value = undefined;

  get value() {
    return this.#value ?? this.defaultValue;
  }

  set value(newValue) {
    this.#value = String(newValue);
  }

  formResetCallback() {
    this.#value = undefined;
  }
}
```

因此：

```
<input type="text" value="default" />
<script> const input = document.querySelector('input');

  console.log(input.getAttribute('value')); 
  console.log(input.value); 
  console.log(input.defaultValue); 

  input.defaultValue = 'new default';

  console.log(input.getAttribute('value')); 
  console.log(input.value); 
  console.log(input.defaultValue); 

  input.value = 'hello!';

  console.log(input.getAttribute('value')); 
  console.log(input.value); 
  console.log(input.defaultValue); 

  input.setAttribute('value', 'another new default');

  console.log(input.getAttribute('value')); 
  console.log(input.value); 
  console.log(input.defaultValue); </script>
```

如果 `value` 属性命名为 `defaultvalue`，这将更合理。现在为时已晚。

在我看来，属性应该用于配置，而状态则可以包含在属性中。我也相信轻量 DOM 树应该有一个单一的所有者。

在这个意义上，我认为 `<input value>` 做得对（除了命名）。`value` 属性配置默认值，而 `value` 属性则提供当前状态。

这也意味着在获取/设置属性时应用验证，但在获取/设置属性时绝不应用验证。

我说'在我看来'，因为最近的一些 HTML 元素已经做出了不同的处理。

`<details>` 和 `<dialog>` 元素通过 `open` 属性表示其打开状态，并且浏览器会根据用户交互自动添加/删除此属性。

我认为这是一个设计错误。它破坏了属性用于配置的理念，但更重要的是，它意味着负责维护DOM的系统（框架，或者原生JS）需要准备好DOM自己改变。

我认为应该是：  

```
<details defaultopen>…</details>
```

和一个`details.open`属性以获取/设置当前状态，以及一个用于定位该状态的CSS伪类。

更新：[西蒙·彼得斯](https://twitter.com/zcorpan)发现了一些关于这个问题的[早期设计讨论](https://lists.whatwg.org/pipermail/whatwg-whatwg.org/2007-August/054532.html)。

我想`contenteditable`也打破了这个约定，但……嗯……它是对很多破坏的选择。

回到之前的例子：

```
<input className="…" type="…" aria-label="…" value="…" />
```

框架如何处理这个问题？

除了预定义的情况外，他们更喜欢属性，如果`propName in element`，他们就将属性设置为属性，否则他们会设置为属性。基本上，他们更喜欢属性而不是属性。他们的渲染为字符串方法则相反，忽略了仅为属性的内容。

React则反其道而行之。除了一组预定义的情况，他们更喜欢属性，他们会设置一个属性。这使得他们的渲染为字符串方法在逻辑上类似。

这解释了为什么自定义元素在React中似乎无法正常工作。因为它们是自定义的，它们的属性不在React的“预定义列表”中，所以它们被设置为属性。在自定义元素上仅为属性的任何内容将简单地无法工作。这将在React 19中修复，他们将切换到Preact/VueJS模型以处理自定义元素。

有趣的是，React推广了使用`className`而不是在*看起来*像属性的`class`。但即使你使用属性名称而不是属性名称，[React在幕后设置了`class`属性](https://github.com/facebook/react/blob/699d03ce1a175442fe3443e1d1bed14f14e9c197/packages/react-dom-bindings/src/client/ReactDOMComponent.js#L388-L389)。

Lit做事情有些不同：

```
<input type="…" .value="…" />
```

它保持了属性和特性之间的区别，如果想设置属性而不是特性，就需要在名称前加上`。`。

这大致是我知道的关于属性和特性之间区别的一切。如果有我忽略的地方，或者你有问题，请在下面的评论中让我知道！

感谢我的[播客伴侣](https://offthemainthread.tech/) [Surma](https://surma.dev/)的常规审阅技能。
