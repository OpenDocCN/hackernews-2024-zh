<!--yml
category: 未分类
date: 2024-05-27 15:19:23
-->

# 12 Modern CSS One-Line Upgrades | Modern CSS Solutions

> 来源：[https://moderncss.dev/12-modern-css-one-line-upgrades/](https://moderncss.dev/12-modern-css-one-line-upgrades/)

 Sometimes, improving your application CSS just takes a one-line upgrade or enhancement! Learn about 12 properties to start incorporating into your projects, and enjoy reducing technical debt, removing JavaScript, and scoring easy wins for user experience.

Properties are explored for the following categories:

*   **[Stable Upgrades](#stable-upgrades)**: fix a hack or issue by replacing older techniques
*   **[Stable Enhancements](#stable-enhancements)**: provide an improved experience with well-supported modern properties
*   **[Progressive Enhancements](#progressive-enhancements)**: provide an upgraded experience when these properties are supported without causing harm in unsupporting browsers

The following well-supported properties can help fix a hack or long-standing issue by replacing older techniques.

Have you ever used the “[padding hack](https://css-tricks.com/aspect-ratio-boxes/)” to force an aspect ratio such as 16:9 for video embeds? As of September 2021, the `aspect-ratio` property is stable in evergreen browsers and is the only property needed to define an aspect ratio.

For an HD video, you can just use `aspect-ratio: 16/9`. For a perfect square, only `aspect-ratio: 1` is required since the implied second value is also `1` .

<details open=""><summary>CSS for "Basic use of aspect-ratio"</summary>

```
.aspect-ratio-hd {
  aspect-ratio: 16/9;
}

.aspect-ratio-square {
  aspect-ratio: 1;
} 
```</details> 

Of note, an applied `aspect-ratio` is forgiving and will allow content to take precedence. This means that when content would cause the element to exceed the ratio in at least one dimension, the element will still grow or change shape to accommodate the content. To prevent or control this behavior, you can add additional dimension properties, like `max-width`, which may be necessary to avoid expanding out of a flex or grid container.

<details open=""><summary>CSS for "Forgiving aspect-ratio"</summary>

```
 .aspect-ratio-square {
  aspect-ratio: 1;
} 
```</details> 

Lorem, ipsum dolor sit amet consectetur adipisicing elit. Iste molestias maiores velit quaerat debitis incidunt delectus nulla, quibusdam fugit eos? Fugiat asperiores assumenda nulla corrupti, sit repellendus ducimus necessitatibus voluptates.

Lorem, ipsum dolor sit amet consectetur adipisicing elit.

Iste molestias maiores velit quaerat debitis incidunt delectus nulla, quibusdam fugit eos?

If you are hesitant to fully replace the padding hack and still want to provide some dimension guardrails, review the [Smol Aspect Ratio Gallery](https://smolcss.dev/#smol-aspect-ratio-gallery) for a progressively enhanced solution to `aspect-ratio`.

This is actually the oldest property in this list, but it solves an important issue and definitely fits the sentiment of a one-line upgrade.

The use of `object-fit` causes an `img` or other [replaced element](https://developer.mozilla.org/en-US/docs/Web/CSS/Replaced_element) to act as the container for its contents and have the those contents adopt resizing behavior similar to `background-size`.

While there are a few values available for `object-fit`, the following are the ones you’re most likely to use:

*   `cover` - the image resizes to *cover* the element, and maintains its aspect-ratio so that the content is not distorted
*   `scale-down` - the image resizes (if needed) *within* the element so that it is fully visible without being clipped and maintains its aspect-ratio, which may lead to extra space (”letterboxing”) around the image if the element has a different rendered aspect-ratio

In either case, `object-fit` is an excellent property pairing with `aspect-ratio` to ensure images are not distorted when you apply a custom aspect ratio.

<details open=""><summary>CSS for "Use of object-fit with aspect-ratio"</summary>

```
.image {
  object-fit: cover;
  aspect-ratio: 1;

  max-block-size: 250px;
} 
```</details> 

> Review my [explanation of object-fit](https://egghead.io/lessons/css-apply-aspect-ratio-sizing-to-images-with-css-object-fit?af=2s65ms) in this free egghead video lesson.

One of many logical properties, `margin-inline` functions as a shorthand for setting the inline (left and right in horizontal writing modes) margin.

The replacement here is simple:

```
 margin-left: auto;
margin-right: auto;

margin-inline: auto;
```

Logical properties have been available for a couple of years and now have [support upwards of 98%](https://caniuse.com/css-logical-props) (with occasional prefixing). Review this article from Ahmad Shadeed to learn more about [using logical properties](https://ishadeed.com/article/css-logical-properties/) and their importance for sites with international audiences.

These well-supported modern CSS properties can provide an improved experience, and may also allow replacing older methods or even JavaScript-aided solutions. Fallback solutions are not likely to be needed, although this is dependent on your specific application considerations, and testing is always encouraged.

The use of `text-underline-offset` allows you to control the distance between the text baseline and the underline. This property has become a part of my standard reset, applied as follows:

```
a:not([class]) {
	text-underline-offset: 0.25em;
}
```

You can use this offset to clear descenders as well as (subjectively) improve legibility, particularly when links are grouped in close proximity, such as a bulleted list of links.

This upgrade may replace older hacks like a border or pseudo-element, or even a gradient background, especially when used with its friends:

Have you been using `box-shadow` or perhaps a pseudo-element to supply a custom outline when you wanted distance between the element and outline on focus?

Good news! The long-available `outline-offset` property ([as early as 2006](https://caniuse.com/?search=outline-offset)!) may be one you missed, and it enables pushing the outline away from the element with a positive value or pulling it into the element with a negative value.

In the demo, the gray solid line is the element border, and the blue dashed line is the outline being positioned via `outline-offset`.

<details open=""><summary>CSS for "Positive and negative outline-offset"</summary>

```
.outline-offset {
  outline: 2px dashed blue;
  outline-offset: var(--outline-offset, .5em);
} 
```</details> 

Positive offset Negative offset

**Reminder**: outlines are *not* computed as part of the element’s box size, so increasing the distance will not increase the amount of space an element occupies. This is similar to how `box-shadow` is rendered without impacting the element size as well.

> Learn more about [using outline-offset as an accessibility improvement](https://moderncss.dev/modern-css-upgrades-to-improve-accessibility/#focus-visibility) for focus visibility.

The `scroll-margin` set of properties (and corresponding `scroll-padding`) allows adding an offset to an element in the context of the scroll position. In other words, adding `scroll-padding-top` can increase scroll offset above the element but doesn’t affect its layout position within the document.

Why is this useful? Well, it can alleviate issues caused by a sticky nav element covering content when an anchor link is activated. Using `scroll-margin-top` we can increase the space above the element when it is scrolled to via navigation to account for the space occupied by the sticky nav.

I like to include a generic starting rule in my reset for any element with an `[id]` attribute given it has the potential to become an anchor link.

```
[id] {
	scroll-margin-top: 2rem;
}
```

An alternative selector is explored in the Modern CSS article on [component-based architecture](https://moderncss.dev/modern-css-for-dynamic-component-based-architecture/#css-reset-additions) and is also in use on this site, as can be tested by using the links from the article table of contents sidebar.

For a more robust solution when accounting for the overlap of sticky, fixed, or absolute positioned elements, you may want to use a custom property with a fallback value. Then, with the assistance of JavaScript, measure the real distance needed and [update the custom property value](https://12daysofweb.dev/2021/css-custom-properties/#accessing-and-setting-custom-properties-with-javascript).

```
[id] {

	scroll-margin-top: var(--scroll-margin, 2rem);
}
```

I encourage you to also update this solution further and use the logical property equivalents: `scroll-padding-block-start` and `-block-end`.

You may be familiar with the `prefers-color-scheme` media query to customize dark and light themes. The CSS property `color-scheme` is an opt-in to adapting browser UI elements including form controls, scrollbars, and CSS system colors. The adaptation asks the browser to render those items with either a `light` or `dark` scheme, and the property allows defining a preference order.

If you’re enabling adapting your entire application, set the following on the `:root`, which says to preference a `dark` theme (or flip the order to preference a `light` theme).

```
:root {
	color-scheme: dark light;
}
```

You can also define `color-scheme` on individual elements, such as adjusting form controls within an element with a dark background for improved contrast.

```
.dark-background {
	color-scheme: dark;
}
```

Learn from Sara Joy’s presentation about how to [use color-scheme for easy dark mode](https://www.youtube.com/watch?v=Lye56NHGtLA), and more about incorporating this feature.

If you’ve ever wanted to change the color of checkboxes or radio buttons, you’ve been seeking `accent-color`. With this property, you can modify the `:checked` appearance of radio buttons and checkboxes and the filled-in state for both the progress element and range input. The browser’s default focus “halo” may also be adjusted if you do not have another override.

<details open=""><summary>CSS for "Effect of using accent-color"</summary>

```
:root {
  accent-color: mediumvioletred;
} 
```</details> 

Consider adding both `accent-color` and `color-scheme` to your baseline application styles for a quick win toward custom theme management.

> If you need more comprehensive custom styling for form controls, review the Modern CSS series beginning with [radio buttons](https://moderncss.dev/pure-css-custom-styled-radio-buttons/).

One of my favorite CSS hidden gems is the use of `fit-content` to “shrink wrap” an element to its contents.

Whereas you may have used an inline display value such as `display: inline-block` to reduce an element’s width to the content size, an upgrade to `width: fit-content` will achieve the same effect. The advantage of `width: fit-content` is that it leaves the `display` value available, thereby not changing the position of the element in the layout unless you adjust that as well. The computed box size will adjust to the dimensions created by `fit-content`.

<details open=""><summary>CSS for "Basic usage of fit-content"</summary>

```
.fit-content {
  width: fit-content;
} 
```</details> 

Using fit-content

Without the use of fit-content

The `fit-content` value is one of several [keywords that enable intrinsic sizing](https://moderncss.dev/contextual-spacing-for-intrinsic-web-design/).

Consider the secondary upgrade for this technique to the logical property equivalent of `inline-size: fit-content`.

## **Progressive Enhancements**

[#](#progressive-enhancements)

This last set of properties provides an upgraded experience when they are supported and can be used without fear of causing harm in unsupporting browsers. This means they do not need a fallback method even though they are more recent additions to modern CSS.

The default behavior of contained scroll regions - areas with limited dimensions where overflow is allowed to be scrolled - is that when the scroll runs out in the element, the scroll interaction passes to the background page. This can be jarring at best, and frustrating at worst for your users.

Use of `overscroll-behavior: contain` will isolate the scrolling to the contained region, preventing continuing the scroll by moving it to the parent page once the scroll boundary is reached. This is useful in contexts such as a sidebar of navigation links, which may have an independent scroll from the main page content, which may be a long article or documentation page.

<details open=""><summary>CSS for "Basic usage of overscroll-behavior"</summary>

```
.sidebar, .article {
  overscroll-behavior: contain;
} 
```</details> 

Lorem ipsum dolor sit amet consectetur adipisicing elit. Maxime nobis consectetur earum!

Aliquid a praesentium quis in consequuntur mollitia laboriosam illum nemo commodi aut?

Doloremque sapiente quos dignissimos sequi cupiditate commodi nemo non perspiciatis placeat totam.

Reiciendis, at nulla! Hic nemo eius atque laborum consequuntur iusto exercitationem quasi.

Est, assumenda amet culpa veritatis maxime debitis? Suscipit error amet quas sed?

Magnam exercitationem neque error deleniti consequuntur, dolor repellat quo perferendis dicta sunt.

Ipsum id repellat velit laudantium vel autem eos non aperiam qui nobis?

Ratione maxime neque numquam minima, omnis dolorem temporibus laboriosam dolor atque suscipit?

Dolores assumenda dolore similique eaque, odio voluptatibus. Aliquid iusto nostrum iste? Exercitationem.

Quos adipisci ea ullam amet blanditiis voluptatibus, fuga laborum sed facere quasi!

Odio rerum labore veritatis esse eaque quae debitis possimus ea omnis vitae.

Quasi eum obcaecati laborum eaque nostrum numquam tempore reprehenderit qui beatae debitis?

Optio atque fugit quia reprehenderit dolor rem et delectus praesentium ratione provident.

Sed accusamus at architecto dolore minima error, assumenda amet nam perferendis odit?

Et eius enim est hic doloribus reiciendis qui cupiditate? Autem, iure cupiditate?

One of the newest properties (as of 2023) is `text-wrap`, which has two values that solve the type-setting problem of unbalanced lines. This includes preventing “orphans,” which describes a lonely word sitting by itself in the last text line.

The first available value is `balance`, which has a goal of evening out the number of characters per line of text.

There is a limitation of six lines of wrapped text, so the technique is best used on headlines or other shorter text passages. Limiting the scope of application also helps limit the impact on page rendering speed.

<details open=""><summary>CSS for "Applying text-wrap: balance"</summary>

```
:is(h1, h2, h3, h4, .text-balance) {
  text-wrap: balance;

  max-inline-size: 25ch;
} 
```</details> 

This text has been balanced by text-wrap

This text has not been balanced by text-wrap

The other value of `pretty` specifically addresses preventing orphans and can be more broadly applied. The algorithm behind `pretty` will evaluate the last four lines in a text block to work out adjustments as needed to ensure the last line has two or more words.

<details open=""><summary>CSS for "Applying text-wrap: balance"</summary>

```
p {
  text-wrap: pretty;

  max-inline-size: 35ch;
} 
```</details> 

**With text-wrap: pretty**

Lorem ipsum dolor sit amet consectetur adipisicing elit. Dolore cupiditate aliquid, facere explicabo voluptatibus iure! Saepe nostrum quasi corporis totam accusamus beatae obcaecati rerum fugiat minima perferendis voluptatibus.

**Without**

Lorem ipsum dolor sit amet consectetur adipisicing elit. Dolore cupiditate aliquid, facere explicabo voluptatibus iure! Saepe nostrum quasi corporis totam accusamus beatae obcaecati rerum fugiat minima perferendis voluptatibus.

Use of `text-wrap` is a great progressive enhancement. However, any adjustments do not change an element's computed width, so a side-effect in some layouts may be an increase in unwanted space next to the text.

In some scenarios, the appearance or disappearance of scrollbars can cause an unwanted layout shift. For example, when a dialog overlay is displayed and the background page adds `overflow: hidden` to prevent scrolling, causing a shift from removing the no longer needed scrollbars.

The modern CSS property `scrollbar-gutter` enables a reservation of space for scrollbars in the layout, which prevents that undesirable shift. When there’s no need for a scrollbar, the browser will still paint a gutter as extra space created in addition to any padding on the scroll container.

> **Important**: This property only has an effect when the user’s system settings are *not* for “overlay” scrollbars, as in the default for Mac OS, where the scrollbar only appears as an overlay on content that is actively being scrolled. Do not drop padding in favor of `scrollbar-gutter` since you will lose all intended space when overlay scrollbars are in use.

Since this is visually apparent extra space, you may choose to assign this property using two keywords: `scrollbar-gutter: stable both-edges`. The use of `stable` by itself only adds a gutter where the scrollbar would otherwise be, while the addition of `both-edges` preferences adding the space to the opposite side of the scroll container, too. This ensures visual balance when the layout doesn’t yet need scrollbars to be visible. See [a visual of using scrollbar-gutter](https://defensivecss.dev/tip/scrollbar-gutter/) from Ahmad Shadeed.

* * *

Be sure to review more of the articles here to upgrade your CSS knowledge even more! A great place to start is [the topics list.](https://moderncss.dev/topics/)