<!--yml
category: 未分类
date: 2024-05-27 15:01:01
-->

# Fuzzing Ladybird with tools from Google Project Zero

> 来源：[https://awesomekling.substack.com/p/fuzzing-ladybird-with-tools-from](https://awesomekling.substack.com/p/fuzzing-ladybird-with-tools-from)

While [Ladybird](https://ladybird.dev/) does an okay job with well-formed web content, I thought it would be useful to throw some security research tools at it and see what kind of issues it might reveal. So today we’ll be using “**[Domato](https://github.com/googleprojectzero/domato)** [🍅](https://github.com/googleprojectzero/domato)”, a DOM fuzzer from [Google Project Zero](https://googleprojectzero.blogspot.com/), to stress test Ladybird and fix some issues found along the way.

The way this works is that Domato generates randomized web pages with lots of mostly-valid but strange HTML, CSS and JavaScript. I then load these pages into a debug build of Ladybird and observe what happens.

Domato generates HTML pages of roughly 500 KiB in size, filled with “interesting” JS, CSS and HTML to surprise and delight your browser engine!

The Domato README boasts a ton of bugs discovered in all major browsers, so I have no doubt it will find some in ours as well. Here we go!

Unsurprisingly, it took less than a second to find the first issue! The output produced by Domato is actually 562 KiB, but I was able to reduce it to the following:

```
<body>
<script>
let mfrac = document.createElement("mfrac");
mfrac.appendChild(document.createElement("th"));
document.body.appendChild(mfrac);
</script>
```

I’ve compiled Ladybird with UBSAN (Undefined Behavior SANitizer) for this test, and I get the following very helpful trace output:

```
**LibWeb/HTML/HTMLTableCellElement.cpp:55:36: runtime error: member call on null pointer of type 'Web::DOM::Node'** #0 0x7e9bfa1610dd in table_containing_cell LibWeb/HTML/HTMLTableCellElement.cpp:55:36
#1 0x7e9bfa1610dd in Web::HTML::HTMLTableCellElement::apply_presentational_hints(Web::CSS::StyleProperties&) const LibWeb/HTML/HTMLTableCellElement.cpp:101:33
#2 0x7e9bf97c22d1 in Web::CSS::StyleComputer::compute_cascaded_values(Web::CSS::StyleProperties&, Web::DOM::Element&, AK::Optional<Web::CSS::Selector::PseudoElement::Type>, bool&, Web::CSS::StyleComputer::ComputeStyleMode) const LibWeb/CSS/StyleComputer.cpp:1448:17
#3 0x7e9bf97e5899 in Web::CSS::StyleComputer::compute_style_impl(Web::DOM::Element&, AK::Optional<Web::CSS::Selector::PseudoElement::Type>, Web::CSS::StyleComputer::ComputeStyleMode) const LibWeb/CSS/StyleComputer.cpp:2231:5
#4 0x7e9bf97e447e in Web::CSS::StyleComputer::compute_style(Web::DOM::Element&, AK::Optional<Web::CSS::Selector::PseudoElement::Type>) const LibWeb/CSS/StyleComputer.cpp:2202:12
#5 0x7e9bf9a7e60c in Web::DOM::Element::recompute_style() LibWeb/DOM/Element.cpp:575:64
```

It’s a good old-fashioned null pointer dereference!

As it turns out, we’ve implemented `<th>` and `<td>` elements with the assumption that there’s always a `<table>` somewhere above it in the DOM tree. We probably believed this because the following is not allowed by the HTML parser:

`<mfrac><th>`

If you load the above markup in a spec-compliant browser, it will create a single `<mfrac>` element with nothing inside it.

However, when creating DOM nodes manually using JavaScript API, you can break some of the rules that the parser has to follow, and indeed put a `<th>` inside an `<mfrac>`!

Here’s the buggy function:

```
HTMLTableElement const& table_containing_cell(HTMLTableCellElement const& node)
{
    auto parent_node = node.parent();
    while (!is<HTML::HTMLTableElement>(parent_node))
        parent_node = parent_node->parent();
    return static_cast<HTML::HTMLTableElement const&>(*parent_node);
}
```

It’s used to implement the ancient feature where `<table border=3>` and `<table padding=5>` applies CSS border and padding to each table cell, and not just the table box itself.

The fix is simply to stop assuming that `<th>` and `<td>` elements always have a containing `<table>` in the ancestor chain. We don’t need the `table_containing_cell` helper but can instead simply replace this:

```
auto const& table_element = table_containing_cell(*this);
```

With this:

```
auto const table_element = first_ancestor_of_type<HTMLTableElement>();
if (!table_element)
    return;
```

And we’re done with issue #1! [(Fix committed here.)](https://github.com/SerenityOS/serenity/commit/b9bacb3ff49ba675f9875434d646fb1b34d2c0df)

We continue executing the fuzzer and once again, within less than 1 second, we hit a new problem. The Domato output is 472 KiB, but it whittles down to this:

```
<script>
    var parser = new DOMParser();
    var doc = parser.parseFromString("", "text/html");
    var body = doc.createElement("body");
    body.onblur = null;
</script>
```

When opened in Ladybird, we fail like so:

```
VERIFICATION FAILED: m_ptr at LibJS/Heap/GCPtr.h:174
#1  JS::GCPtr<Web::HTML::Window>::operator* at LibJS/Heap/GCPtr.h:174
#2  Web::DOM::Document::window at LibWeb/DOM/Document.h:320
#3  Web::HTML::HTMLBodyElement::global_event_handlers_to_event_target at LibWeb/HTML/HTMLBodyElement.cpp:104
#4  Web::HTML::GlobalEventHandlers::set_onblur at LibWeb/HTML/GlobalEventHandlers.cpp:24
#5  Web::Bindings::HTMLElementPrototype::onblur_setter at LibWeb/Bindings/HTMLElementPrototype.cpp:1153
```

The reason things go wrong here is due to the special behavior of `onfoo` event handler attributes on the `<body>` element. For compatibility with ancient web content, assignments to `document.body.onfoo` must forward to `window.onfoo`. However, documents created via `DOMParser`  *do not have a window object!*

It turns out we’ve misunderstood this detail in our implementation, and structured our internal object model as if every document always has a window. Oops!

The fix is to make `Document::window()` return a nullable value, and then handle null in a bajillion places. When assigning `document.body.onblur` in a window-less document, we now simply do nothing, same as other browsers.

[(Fix committed here.)](https://github.com/SerenityOS/serenity/commit/b98a2be96bf03562ec21e80b48d4b3b97ad74cd6)

Modern browsers have to support SVG both inline in HTML, and also as an external image format. This adds a whole host of new edge cases and interesting interactions.

For example, SVG allows declaring gradients that reference other gradients to inherit their colors. As it turns out, we hadn’t considered the case where a gradient references itself:

```
<svg>
    <linearGradient id="oops" href="#oops"/>
    <rect fill="url(#oops)" />
</svg>
```

The SVG above would cause our implementation to loop forever as it attempted to follow the chain of referenced gradients. Oops, indeed!

The obvious fix is to stop following the chain of linked gradients if the current gradient references itself. However, that doesn’t cover reference cycles that span multiple steps like so:

```
<svg>
    <linearGradient id="lol" href="#lmao"/>
    <linearGradient id="lmao" href="#even"/>
    <linearGradient id="even" href="#lol"/>
    <rect fill="url(#lol)" />
</svg>
```

To properly handle cyclical references, we have to keep track of all the gradients we’ve visited, and stop following the chain if we encounter a gradient we’ve already visited before.

Curiously, Firefox actually complains about these kind of gradients in their developer console:

[(Fix committed here.)](https://github.com/SerenityOS/serenity/commit/2e0297d7035a807d4a18c89c8b310ec06bef005d)

This one is interesting:

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

The above test would crash like this:

```
**LibWeb/HTML/WindowProxy.cpp:161:136: runtime error: reference binding to null pointer of type 'const BrowsingContext'** #0 0x77367675cadf in Web::HTML::WindowProxy::internal_get(JS::PropertyKey const&, JS::Value, JS::CacheablePropertyMetadata*) const LibWeb/HTML/WindowProxy.cpp:161:5
#1 0x773670c4fcca in JS::Bytecode::get_by_id(JS::VM&, AK::DeprecatedFlyString const&, JS::Value, JS::Value, JS::Bytecode::PropertyLookupCache&) LibJS/Bytecode/CommonImplementations.h:105:18
#2 0x773670c182d5 in JS::Bytecode::Op::GetById::execute_impl(JS::Bytecode::Interpreter&) const LibJS/Bytecode/Interpreter.cpp:1088:28
#3 0x773670bc8e1f in execute LibJS/Bytecode/Op.h:1898:9
#4 0x773670bc8e1f in JS::Bytecode::Interpreter::run_bytecode() LibJS/Bytecode/Interpreter.cpp:409:38
```

As it turns out, this is actually a bug in the HTML spec! When an iframe is removed from the DOM, its content document is detached from its browsing context. However, when getting or setting a property on a window object, we run a spec algorithm called **"check if an access between two browsing contexts should be reported"** which inspects the browsing context of the “accessor” and “accessed” window. The spec incorrectly assumed that both windows have an associated browsing context at the time of property access.

[I’ve opened an issue against the HTML spec](https://github.com/whatwg/html/issues/10192), and patched this in Ladybird in the meantime by adding a null check.

Finding spec bugs is actually one of my favorite things while working on Ladybird. It allows us to improve the specs for everyone by submitting a bug report or fix.

[(Fix committed here.)](https://github.com/SerenityOS/serenity/commit/ad843b6e4a4404a4897b587711f9ce97a4504ed0)

When opening the next page, it just wouldn’t finish loading. It just sat there chewing 100% CPU.

Here’s the reduction:

```
<div id="one"></div><div id="two"></div>
<script>
    two.before(one);
</script>
```

This was a mistake in the implementation of `before()`, which has to find the first preceding sibling of `<div id=”two”>` that isn’t one of its arguments (i.e `<div id=”one”>` in this case.) We had the following loop mistake in the implementation:

```
while (auto previous_sibling = node->previous_sibling()) {
    // check if previous_sibling is one of the arguments
}
```

The problem here is that we kept fetching `node->previous_sibling`, instead of continuing to follow the sibling chain via `previous_sibling->previous_sibling`.

Here’s how I fixed it:

```
for (auto sibling = node->previous_sibling(); sibling; sibling = sibling->previous_sibling()) {

    // check if previous_sibling is one of the arguments

}
```

[(Fix committed here.)](https://github.com/SerenityOS/serenity/commit/35f359c51c787153cd17c7289664ab87146241ea)

This can probably go on for quite some time, so let’s call it a day. We found five real bugs, one of which was a spec bug, and were able to fix all of them.

Even though things went basically as I expected, it’s still quite interesting just how quickly we fall apart when confronted with strange and unexpected inputs. Fuzzers like Domato are an amazing resource for anyone who wants to make their software more robust.

The next step here will be to get Ladybird to the point where we can handle a sustained onslaught of fuzzed inputs. And once it’s reasonably stable, we can start running it automatically in the cloud somewhere, and hopefully shake out even more issues.