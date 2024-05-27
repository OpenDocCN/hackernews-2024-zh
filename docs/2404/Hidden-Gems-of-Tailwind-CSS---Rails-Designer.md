<!--yml
category: 未分类
date: 2024-05-27 13:33:32
-->

# Hidden Gems of Tailwind CSS | Rails Designer

> 来源：[https://railsdesigner.com/hidden-gems-tailwind/](https://railsdesigner.com/hidden-gems-tailwind/)

Tailwind CSS makes designing and creating web UI’s a breeze. No wonder it’s my number one choice for every (Rails-based) SaaS app I’ve launched over the last few years. It is no surprise it is also a core tool of [Rails Designer](https://railsdesigner.com/).

I wanted to list some of the lesser known or obscure features, for future me, but most importantly for you to squeeze even more out of all that Tailwind CSS offers.

## Change radio/checkbox colors

Use the `accent-*` utilities to change the color of a checkbox or radio button.

```
<label>
  <input type="checkbox" checked> Default
</label>
<label class="ml-3">
  <input type="checkbox" class="accent-pink-600" checked> The sky is the limit
</label> 
```

## Peer-modifier

The `peer`-modifier allows to style an element based on a **sibling state**. Alright, now you’re talking nonsense! Let’s look at an example?

```
<input type="email" placeholder="support@railsdesigner" value="support@" class="w-full px-3 py-1 text-sm rounded text-slate-700 peer bg-slate-200 ring-1 ring-offset-0 ring-slate-200"/>
<span class="invisible !mt-0.5 text-sm text-rose-600 peer-invalid:visible">
  Please provide a valid email address.
</span> 
```

  Please provide a valid email address.

It works by adding `peer` to one element and then the immediate next element can use any `peer-*` utility. It works with every pseudo-class modifier, like `peer-hover`, `peer-required` and `peer-disabled`. Once you know this utility, you will find plenty of ways to use it **instead of JavaScript**.

## Style file-inputs

When you want to spruce up your file-input field, use the `file:` modifier.

```
<input type="file" class="block w-full text-sm text-slate-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-sky-50 file:text-sky-700 hover:file:bg-sky-100 "/> 
```

It takes a few utilities, but packs a lot of style to your file-input!

## Change color opacity

Every color-utility in Tailwind CSS allows to append a `/n` opacity-modifier. So this works for every text-color, background-color and ring-color.

```
<button class="px-2 rounded bg-emerald-500/100">A</button>
<button class="px-2 rounded bg-emerald-500/75">B</button>
<button class="px-2 rounded bg-emerald-500/50">C</button> 
```

I find I use this often to let elements blend more with backgrounds that have gradients to them. Or to get the color *just right*.

## Combining font-size and line-height

In a similar way you can combine font-size and line-height.

You likely know about the font-size utilites (`text-sm`, `text-base` and so on) and the line-height utilties (relative: `leading-loose` and `leading-tight`, fixed: `leading-4` and `leading-6`).

*Note*: Relative line-height is based on its current font-size. While a fixed line-height, sets it irrespective of the current font-size.

You can combine the fixed line-height with the font-size `text-xl/8` to set a font size of `1.25rem` with a line-height of `2rem`.

## Add space between elements

This one I use all the time as it’s a great way to add space between an element and an optional element. `gap-*` only adds the space between actual elements, it not just adds `margin-left` or `margin-right`.

I learned too late that `gap-*` can also be used in flex layouts, not just in a grid (as the name implies). There’s also `gap-x-*` and `gap-y-*` if you want to add space respectively horizontal (left/right) or vertical (up/down).

## Transition hover/active states

Something that can really up the “feel” of your app is micro transitions. It’s something I add a lot.

Instead of `bg-sky-100 hover:bg-sky-200`.

I do: `bg-sky-100 transition hover:bg-sky-200`.

For something even smoother, write it like this: `bg-sky-100 transition ease-in-out duration-300 hover:bg-sky-200`.

#### Get the latest Rails Designer updates

If you use this on bigger card-elements, it’s worth to add a `delay-75` so the transition is not triggered when the user hovers over the element when scrolling.

## Pointer events

This is one I stumbled upon recently when fixing a bug for the [Rails Designer’s NotificationComponent](https://railsdesigner.com/components/notifications/).

If you have an element that is “above” another element (as is the case with container-element with NotificationComponent, you have witnessed this as well.

Pointer events will trigger on child elements and pass-through to elements that are “below” the target.

Add `pointer-events-none` and you’re done: love it!

## Truncating multi-line text

When dealing with user-generated content, you 100% come into situations it will break your UI.

`line-clamp-*` utility helps with this. It breaks off the sentence at the given number and adds ellipsis (`…`, not to be confused with: `...`).

It adds this CSS to the element:

```
overflow: hidden;
display: -webkit-box;
-webkit-box-orient: vertical;
-webkit-line-clamp: 1; 
```

You can use any number from 1 to 6 or `none`.

## Combine width and height for square element

Before, when you wanted a square element, like an [AvatarComponent](https://railsdesigner.com/components/avatars/), you needed to add both the `w-*` and the `h-*`.

Now these can be combined with the `size-*` utility that set the width and height at the same value.

## Highlight text

A little extra personality to your app can be added by adding the `selection:` modifier. Add it your body-element to tweak every page when text is selected.

```
selection:bg-sky-50 selection:text-sky-600 
```

Just like on this site (try selecting text)!

## Named groups

You can style elements based on the state (hover, focus, disabled) of its parent element. It works like this:

```
<a href="#" class="group hover:bg-sky-500 hover:ring-sky-500">
  <h3 class="text-sm font-semibold text-slate-900 group-hover:text-white">
    New project
  </h3>
</a> 
```

But I prefer to, almost always, name my groups. Like so:

```
<a href="#" class="group/link hover:bg-sky-500 hover:ring-sky-500">
  <h3 class="text-sm font-semibold text-slate-900 group-hover/link:text-white">
    New project
  </h3>
</a> 
```

It took me a bit of time to remember the syntax, but it has proved a valuable default when doing more advanced UI work.

## Add custom variants

You can add your own variants to target really specific things. Let’s take the one I use in all my Rails apps (which all use `turbo-frame`s).

```
{
  // …
  plugins: [
    // …
    function ({ addVariant }) {
      addVariant("turbo-frame", "turbo-frame[src] &")
    }
  ]
} 
```

Now you can style elements differently *when they are within a `turbo-frame`*.

```
<div class="px-4 py-2 turbo-frame:shadow-xl">
</div> 
```

Imagine the element when viewed as a page doesn’t has a dropshadow, but when viewed in turbo-frame (eg. as [Modal](https://railsdesigner.com/components/modals/)) it does add a dropshadow. Making elements truly reusable.

## Arbitrary values

If all of the many Tailwind CSS utilities are not enough, you can use *any value* you want! I use this really as a last resort, as it stills feels like going against the framework (even though it is a framework-supported feature).

You take any of the available utility and add your own value, like this `text-[0.65rem]` (really something I use in all my apps—maybe need to make it a custom variant?).

That’s all I have for now. Is there any feature you don’t see too often? [Do share it with me](mailto:support@railsdesigner.com).