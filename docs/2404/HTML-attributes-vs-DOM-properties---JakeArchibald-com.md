<!--yml
category: 未分类
date: 2024-05-27 13:32:33
-->

# HTML attributes vs DOM properties - JakeArchibald.com

> 来源：[https://jakearchibald.com/2024/attributes-vs-properties/](https://jakearchibald.com/2024/attributes-vs-properties/)

Attributes and properties are *fundamentally* different things. You can have an attribute and property of the same name set to different values. For example:

```
<div foo="bar">…</div>
<script> const div = document.querySelector('div[foo=bar]');

  console.log(div.getAttribute('foo')); 
  console.log(div.foo); 

  div.foo = 'hello world';

  console.log(div.getAttribute('foo')); 
  console.log(div.foo); </script>
```

It seems like fewer and fewer developers know this, partially thanks to frameworks:

```
<input className="…" type="…" aria-label="…" value="…" />
```

If you do the above in a framework's templating language, you're using attribute-like syntax, but under the hood it'll sometimes be setting the property instead, and when it does that differs from framework to framework. In some cases, it'll set a property *and* an attribute as a side-effect, but that isn't the framework's fault.

Most of the time, these distinctions don't matter. I think it's good that developers can have a long and happy career without caring about the differences between properties and attributes. But, if you need to dig down into the DOM at a lower level, it helps to know. Even if you feel you know the difference, maybe I'll touch on a couple of details you hadn't considered. So let's dig in…

Before we get to the interesting stuff, let's get some of the technical differences out of the way:

Attributes serialise to HTML, whereas properties don't:

```
const div = document.createElement('div');

div.setAttribute('foo', 'bar');
div.hello = 'world';

console.log(div.outerHTML); 
```

So when you're looking at the elements panel in browser developer tools, you're only seeing attributes on elements, not properties.

In order to work in the serialised format, attribute values are always strings, whereas properties can be any type:

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

Attribute names are case-insensitive, whereas property names are case-sensitive.

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

However, attribute *values* are case-sensitive.

Ok, here's where things start to get blurry:

Take a look at this:

```
<div id="foo"></div>
<script> const div = document.querySelector('#foo');

  console.log(div.getAttribute('id')); 
  console.log(div.id); 

  div.id = 'bar';

  console.log(div.getAttribute('id')); 
  console.log(div.id); </script>
```

This seems to contradict the first example in the post, but the above only works because `Element` has an `id` getter & setter that 'reflects' the `id` attribute.

When a property reflects an attribute, the *attribute* is the source of the data. When you set the property, it's updating the attribute. When you read from the property, it's reading the attribute.

For convenience, most specs will create a property equivalent for every defined attribute. It didn't work in the example at the start of the article, because `foo` isn't a spec-defined attribute, so there isn't a spec-defined `foo` property that reflects it.

[Here's the spec for `<ol>`](https://html.spec.whatwg.org/multipage/grouping-content.html#the-ol-element). The "Content attributes" section defines the attributes, and the "DOM interface" defines the properties. If you click on `reversed` in the DOM interface, it takes you to this:

> The `reversed` and `type` IDL attributes must [reflect](https://html.spec.whatwg.org/multipage/common-dom-interfaces.html#reflect) the respective content attributes of the same name.

But some reflectors are more complex…

Ok, this is relatively minor, but sometimes the property has a different name to the attribute it reflects.

In some cases it's just to add the kind of casing you'd expect from a property:

*   On `<img>`, `el.crossOrigin` reflects the `crossorigin` attribute.
*   On all elements, `el.ariaLabel` reflects the `aria-label` attribute (the aria reflectors became cross browser in late 2023\. Before that you could only use the attributes).

In some cases, names had to be changed due to old JavaScript reserved words:

*   On all elements, `el.className` reflects the `class` attribute.
*   On `<label>`, `el.htmlFor` reflects the `for` attribute.

Properties come with validation and defaults, whereas attributes don't:

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

In this case, the validation is handled by the `type` getter. The setter allowed the invalid value `'foo'`, but when the getter saw the invalid value, or no value, it returned `'text'`.

Some properties perform type coercion:

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

In this case, the `open` property is a boolean, returning whether the attribute exists. The setter also coerces the type - even though the setter is given `'hello'`, it's turned to a boolean rather than going directly to the attribute.

Properties like `img.height` coerce the attribute value to a number. The setter converts the incoming value to a number, and treats negative values as 0.

`value` is a fun one. There's a `value` property and a `value` attribute. However, the `value` property does not reflect the `value` attribute. Instead, the `defaultValue` property reflects the `value` attribute.

I know, I know.

In fact, the `value` property doesn't reflect *any* attribute. That isn't unusual, there's loads of these (`offsetWidth`, `parentNode`, `indeterminate` on checkboxes for some reason, and many more).

Initially, the `value` property defers to the `defaultValue` property. Then, once the `value` property is set, either via JavaScript or through user interaction, it switches to an internal value. It's as if it's implemented *roughly* like this:

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

So:

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

This would have made way more sense if the `value` attribute was named `defaultvalue`. Too late now.

In my opinion, attributes should be for configuration, whereas properties can contain state. I also believe that the light-DOM tree should have a single owner.

In that sense, I think `<input value>` gets it right (aside from the naming). The `value` attribute configures the default value, whereas the `value` property gives you the current state.

It also makes sense that validation applies when getting/setting properties, but never when getting/setting attributes.

I say 'in my opinion', because a couple of recent HTML elements have done it differently.

The `<details>` and `<dialog>` elements represent their open state via the `open` attribute, and the browser will self add/remove this attribute in response to user interaction.

I think this was a design mistake. It breaks the idea that attributes are for configuration, but more importantly it means that the system in charge of maintaining the DOM (a framework, or vanilla JS) needs to be prepared for the DOM to change itself.

I think it should have been:

```
<details defaultopen>…</details>
```

And a `details.open` property to get/set the current state, along with a CSS pseudo-class for targeting that state.

Update: [Simon Peters](https://twitter.com/zcorpan) unearthed some of the [early design discussion around this](https://lists.whatwg.org/pipermail/whatwg-whatwg.org/2007-August/054532.html).

I guess `contenteditable` also breaks that contract, but… well… it's a opt-in to a lot of breakage.

Back to the example from earlier:

```
<input className="…" type="…" aria-label="…" value="…" />
```

How do frameworks handle this?

Aside from a predefined set of cases where they favour attributes, they'll set the prop as a property if `propName in element`, otherwise they'll set an attribute. Basically, they prefer properties over attributes. Their render-to-string methods do the opposite, and ignore things that are property-only.

React does things the other way around. Aside from a predefined set of cases where they favour properties, they'll set an attribute. This makes their render-to-string method similar in logic.

This explains why custom elements don't seem to work in React. Since they're custom, their properties aren't in React's 'predefined list', so they're set as attributes instead. Anything that's property-only on the custom element simply won't work. This will be fixed in React 19, where they'll switch to the Preact/VueJS model for custom elements.

The funny thing is, React popularised using `className` instead of `class` in what *looks like* an attribute. But, even though you're using the property name rather than the attribute name, [React will set the `class` attribute under the hood](https://github.com/facebook/react/blob/699d03ce1a175442fe3443e1d1bed14f14e9c197/packages/react-dom-bindings/src/client/ReactDOMComponent.js#L388-L389).

Lit does things a little differently:

```
<input type="…" .value="…" />
```

It keeps the distinction between attributes and properties, requiring you to prefix the name with `.` if you want to set the property rather than the attribute.

That's pretty much everything I know about the difference between properties and attributes. If there's something I've missed, or you have a question, let me know in the comments below!

Thanks to my [podcast husband](https://offthemainthread.tech/) [Surma](https://surma.dev/) for his usual reviewing skills.