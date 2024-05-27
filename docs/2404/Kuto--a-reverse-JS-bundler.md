<!--yml
category: 未分类
date: 2024-05-27 13:02:29
-->

# Kuto, a reverse JS bundler

> 来源：[https://samthor.au/2024/kuto/](https://samthor.au/2024/kuto/)

# Kuto, a reverse JS bundler

[Kuto](https://kuto.dev) is a novel approach to shipping code on the web. It lets you re-use code a client *already has* for shipping updates.

For a 'real-world' site with ~3mb of JS, updating the React dependency resulted in:

*   71% smaller download
*   28% faster start time (on a ~5yo old phone, a [Pixel 3](https://en.wikipedia.org/wiki/Pixel_3)).

…vs a single bundle, or any case where all the code is invalidated.

Note that Kuto works really well on the final ESM bundles of real sites or apps, but probably *not* libraries themselves, even though Kuto's output will be valid. Kuto also works as a predictable 'chunk' generator for large bundles.

If this is interesting to you—do *you* have too much JavaScript?—then do the thing, and [do a Kuto on your code](https://kuto.dev). (Is Kuto a verb? Who knows. I'm trying it out.) ✂️

## How does it work?

Instead of focusing on minifying output or anything idempotent, Kuto takes a different route.

1.  On the first build:

    *   Kuto splits source JS into a 'main' part, and a normally larger 'corpus' of code which has no side effects.
    *   This corpus can be cached [forever](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control#immutable), and a hashed timestamp is included in its output name.

2.  On every further build:

    *   Kuto still splits out the source JS
    *   It identifies code from any existing corpus that can be used to 'satisfy' the source JS
        *   Each corpus will either stay the same or *shrink* as functions, statements etc change
    *   Any code that cannot be satisfied is put into a brand new corpus, which can also be cached forever.

This is a little complicated. Here's a ~~gif~~ video:

 <https://storage.googleapis.com/hwhistlr.appspot.com/assets/kuto-demo.mp4> 

Kuto will build an additional corpus when something changes

After each build, you're likely to generate another corpus with changed code. (This will eventually be an issue with fragmentation, more on this later.)

## Why does it work?

Each corpus, once fetched by a client, can be cached forever. However, on every build, that corpus *may* shrink in size. New clients will get the smaller version. (Kuto's corpuses have a hashed timestamp, and you *do* have to set up your webserver to send the [right headers](https://web.dev/articles/love-your-cache) here).

But…this breaks all we know about caching responses on the web! I've changed an immutable file! 🤯

The result is that older clients will have bigger files, and newer clients will have smaller ones, even for the same filename. But that bigger file simply has now-deprecated code. Kuto's primary thesis is that disk I/O to load a slightly bigger file than you need is faster than compling anew.

(Note that at least [v8-based browsers](https://v8.dev/blog/code-caching-for-devs) cache the bytecode of the source, which provides the speed benefit. If you were compiling anew every time, Kuto wouldn't help.)

Aside the remarkably bonkers way this leverages your browser, Kuto also basically performs code-splitting in a completely predictable, useful and automated fashion. Other bundlers either require you to:

*   (not codesplit an output bundle at all)
*   explicitly mark dependencies to be put into their own bundle
*   put code in bundles on (effectively) random boundaries, just trying to restrict the size of each 'chunk'.

## No really, how does it work?

The above explanation was fairly high-level. At a lower level, Kuto looks for code with no side effects to include in its corpuses, and it uses circular dependencies to ensure they're safe to call.

### No Side Effects

Turns out, defining a function has no side effects. No, really! The *definition* of a function does nothing except creates a variable:

```
function foo() {
  console.info(`Side effect`);
} 
```

Since `foo` isn't *actually called*, and we declare it in a module scope, nothing happens. Until we run `foo`, it might as well be:

```
function foo() {

} 
```

Kuto takes this theory to the extreme, putting classes into the same form, and even arbitrary statements. We *hand wave* can hoist statements to be within a function.

### Circular Dependencies

Each corpus contains silo'ed^ functions which reference back to the main file in order to work out their dependencies. A good way to see this is to check out Kuto itself and run its "./release.sh" script, which builds Kuto itself this way.

For a trivial site that appends a custom element to your page, where the custom element has changed and been rebuilt, this might look like:

```
 import { _1 } from './main.kt-abc.js';
import { _2 } from './main.kt-def.js';
_1();
export { _2 };

import { _2 } from './main.js';
export var _1 = function setupSite() {
  const element = new _2();
  document.body.append(element);
}

export var _2 = class MyElement extends HTMLElement {  }; 
```

This all looks awkward…and it is, but the output isn't really meant for human consumption, but we benefit from ESM's "seen as a bug" feature of circular dependencies.

^Kuto will in future reference *between* corpuses

## Should I use this?

Maybe!

Kuto is new, and while the science says it works, it's a bit weird. And as I've mentioned above, it works really well on:

*   single bundle output sites
*   that have a large JS bundle (maybe >1mb?)
*   …which are made up of lots of top-level ESM code, such as functions and classes

You will need to:

*   use a regular bundler first
*   keep or have access to your old build artifacts—Kuto doesn't know what you *already shipped* by magic 🪄
*   trade off a slightly larger *first load* for a better update experience

The other issue with this is that more and more browsers are moving to a world where cache is regularly evicted. If your visitors don't have your site cached, then Kuto *is* pointless—every load is slightly more expensive for an update that never happens. YMMV.

Regardless, the tool emits a few statistics, including how much overhead the initial load costing you, and what % of code is able to be identifed as having "no side effects" and put into a corpus. And to be clear, running Kuto on tiny codebases has no benefit—the 'cost' of parsing a few kb of JS is trivial, even for potato phones. 🥔

So you can experiment and see if it works for you. My view is that Kuto will help for enterprise apps (because the JS is bloated, and no-one *really* cares) or social media (because power users come *so often*).

### Fragmentation

Kuto generates multiple files over time. Right now, it actually only uses the top 4 (by size) previous bundles, although this is configurable by flag. This is…not very good, and I'm yet to come up with a good automatic metric as to when to 'clean up' vs 're-use'.

Consider this though: if you just don't provide Kuto with historic bundles, it… obviously can't use them. So you can decide whether that typo fix generating a 100-byte file is worth it for the *next build*.

## Why Only 28% Faster?

Ignoring some of the possible downsides, there's a big question for me at play. My test case above showed 71% reduction in size, but only 28% increase in speed. 🤔

I have a suspicion that v8's assembly of a bunch of module code *is still quite costly*. Yes, it's great that huge chunks of static code can be outsourced and cached forever, but in the end, your website still needs to assemble it all together.

## An Ask

I would love to hear from you if Kuto makes a material difference to the way you bundle and update your code, even in just a theoretical test environment. Please feel free to contact me, including filing [a GitHub issue](https://github.com/samthor/kuto/issues/new), if you can give me some sweet, sweet numbers.

## Thanks

Thanks for reading! Kuto has been incredibly interesting to build, yet honestly it's only taken me a few days of full-time engineering to make it work. It's been an idea I've been noodling on for a few years, and I'm really proud to have it come to fruition. 🍇