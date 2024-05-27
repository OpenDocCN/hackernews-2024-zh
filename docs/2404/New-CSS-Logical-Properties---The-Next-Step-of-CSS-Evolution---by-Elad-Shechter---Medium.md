<!--yml
category: 未分类
date: 2024-05-27 13:00:31
-->

# New CSS Logical Properties!. The Next Step of CSS Evolution | by Elad Shechter | Medium

> 来源：[https://elad.medium.com/new-css-logical-properties-bc6945311ce7](https://elad.medium.com/new-css-logical-properties-bc6945311ce7)

## CSS Floats

Float is very straightforward, there are only two values, `**inline-start**`/`**inline-end,**` that replace the values of `left`/`right` **.**

In English(LTR):
`float: left` = `float: **inline-start**`
`float: right` = `float**: inline-end**`

## Text-align

Even easier than floats, the values `left`/`right` are replaced by `**start**`/`**end**`.

In English(LTR):
`text-align :left` = `text-align: **start**;`
`text-align :right` = `text-align: **end**;`

## More Stuff

**Resize** property: used mostly for `<textarea>`, its values are updated from `horizontal`/`vertical` to `**inline**`/`**block**`;

In English (LTR):
`resize: horizontal` = `resize: **inline**;`
`resize: vertical` = `resize: **block;**`

**background-position**: There isn’t any implementation yet in any browser, but if you look deeply, you can find some references to `**background-position-inline**` & `**background-position-block**` in Mozilla’s MDN website. There is no complete documentation yet, but they’re working on it! :-)

**Other Stuff**: We can assume that properties like `**transform-origin**` will be updated as well, much like any property that has to do with directions.

## CSS Grid & CSS Flexbox

The great news for CSS Grid & CSS Flexbox is that these two features are already built with the new methodology of logical properties, and there is no need to update them.

## Understanding the Workflow with Logical Properties

At first, It looks very complicated, but it’s very easy to use. While writing the styles, there is no need to worry about cross-language support. You just need to use logical properties, instead of the old physical properties, and match them to your preferred language.

# Applying Alignment According to Language

After we learned all the updates of the new logical properties, here are two properties that let us define the **block axis alignment** (flow of the website), and the **inline axis alignment** (the direction in which we read the text).

## Writing-mode property (block axis)

Defines the flow of the website. In most cases, it will be top to bottom, but as mentioned, certain language can flow from right to left (Japanese), or even left to right (Mongolian). In both cases, we will have a horizontal scroll instead of the vertical scroll we are used to.

Note: For now there are 3 main values for `**writing-mode**` . Their names are a little bit confusing. That’s because in their name there is the block-axis direction, plus what will be the alignment of the text in that case (inline-axis). This is obviously very frustrating, as the text alignment is redundant, and it causes only confusion.

To eliminate this confusion, it’s recommended to ignore the inline-axis part of the value, and only refer to the block-axis part of the value.

Examples:

**values:**

*   `writing-mode: horizontal-**tb**;` = **Top to Bottom flow**, like in English (default value)
*   `writing-mode: vertical-**rl**;` = **Right to Left Flow,** for Japanese.
*   `writing-mode: vertical-**lr**;` = **Left to Right Flow**, for Mongolian.

As for my personal opinion, I would have preferred that the values included only: **tb/rl/lr (block-axis part),** to eliminate this potential confusion.

Example definition for **Japanese**:

```
html{
    **writing-mode**: **vertical-rl**;
}
```

## Direction property (inline axis)

Defines whether the text should start from **left-to-right** or **right-to-left**, but only when the default horizontal `writing-mode` property is active. If we change the `writing-mode` to one of the vertical modes, the actual text direction, of left-to-right, will change to top-to-bottom. Or the opposite, with an`rtl`(right-to-left) value, it will change to bottom-to-top.

Example definition for **Arabic**:

```
html{
    **direction**: **rtl**;
}
```

It’s Amazing how easily a top-to-bottom website can be transformed into a right-to-left website with a horizontal scroll.

Here is a demo I made, best viewed in Firefox (which currently supports more features).

[**Live Example**](https://codepen.io/elad2412/pen/oQJmYQ/) **(try out the language selection!):**

# Browsers Support

*   All The Box Model properties `margin`/`padding`/`border` and the new `width`/`height`(`inline-size`, `block-size`) properties work in all main browsers except for Edge.
*   `text-align` new values work in all main browsers except for Edge.
*   `Floats`/`Positions`/`Resize` — values/properties work only in Firefox.

# Issues With the Logical Properties

With these new refactors, we are facing new issues. For example, what if we want to write all properties of margin in the shorthand way:
`margin: 10px 20px 8px 5px;` you can’t predict how it would be interpreted.
If the website is using physical properties, the values will be interpreted as:
`**margin-top**`**/**`**margin-right**`**/**`**margin-bottom**`**/**`**margin-left**`**,**
but if the website is using the new logical properties, the values should be:
`**margin-block-start**`**/**`**margin-inline-end**`**/**`**margin-block-end**`**/**`**margin-inline-start**`**.**

In English websites, the physical properties and the logical properties work in the same way. In other languages, when we use shorthand, like `margin`, the goal is for it to work according to the `**direction**` property or the new property `**writing-mode**`**.**

This is still an open issue. I’ve added a [*suggestion that can solve the issue at github csswg-drafts*](https://github.com/w3c/csswg-drafts/issues/1282#issuecomment-443253091)*.* If you think you have a better solution, you can comment there!

For now, if you want to use logic properties, you have to use the full property names without any shorthand.

**My Optional Solution:**

```
html{
   flow-mode:**physical**; 
       /*or*/
   flow-mode:**logical**;
}.box{
  /***will be** **interpreted** **according to the HTML flow-mode value***/
   margin:10px 5px 6px 3px;
   padding:5px 10px 2px 7px;
}
```

## Issue With Responsive Design

When trying to make a fully working demo, I tried to use the new “max-width” property `**max-inline-size**`in a media query**,** with the understanding that in left-to-right/right-to-left it will be behave like `**max-width**` and for languages like Japanese, it will behave like `**max-height**`. Unfortunately, the browsers are not interpreting this property currently in media queries.

```
/*Not Working*/
[@media](http://twitter.com/media) (**max-inline-size**:1000px){
  .main-content{
    background:red;
    grid-template-columns:auto;
  }
}
```

## Changes to be Considered

When I wrote this post, after learning and understanding the concept of logical properties in-depth, there were still some missing adaptions, that I thought should be incorporated in the future:
- `line-height` can be `“line-size”`
- `border-width` can be `“border-size”`

And it does not seem to be the case, at least with the `border-width`. It just got updated with logical properties and it still has the word ‘width’ in its name.
Example: `border-block-start-width`.

But who knows, maybe the right people at w3c will read this post :-)

# Final Words

That’s all,
I hope you’ve enjoyed this article and learned from my experience.
If you like this post, I would appreciate applause and sharing :-)

I create lots of content about CSS. Be sure to follow me via [**Twitter**](https://twitter.com/eladsc)**,** [**Linkedin**](https://www.linkedin.com/in/eladshechter/)**, and** [**Medium**](https://medium.com/@elad)**.**

You can also access all of my content on my website: [**eladsc.com**](https://eladsc.com/).

**More Recommended posts on typography:**
[CSS Writing Modes](https://24ways.org/2016/css-writing-modes/) — Very Recommended!
[Text Orientations](https://tympanus.net/codrops/css_reference/text-orientation/)

**More of my CSS posts:**
[Why CSS HSL Colors are Better!](https://medium.com/@elad/why-css-hsl-colors-are-better-83b1e0b6eead)
[The New Responsive Design Evolution](https://medium.com/@elad/the-new-responsive-design-evolution-2bfb9b504a4e)
[CSS Position Sticky — How It Really Works!](https://medium.com/@elad/css-position-sticky-how-it-really-works-54cd01dc2d46)

**Who Am I?**
**I am Elad Shechter**, a Web Developer specializing in CSS & HTML design and architecture.

(Me talking at ConFrontJS 2019, Warsaw, Poland)

# My Talk on CSS Logical Properties — New!

This is my full talk on CSS Logical Properties, fill free to share it [🙂](https://emojipedia.org/slightly-smiling-face/)