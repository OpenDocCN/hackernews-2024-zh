<!--yml
category: 未分类
date: 2024-05-27 14:34:21
-->

# A concrete example of why Apple's documentation is terrible | A concrete example of why Apple's documentation is terrible

> 来源：[https://www.amimetic.co.uk/blog/a-concrete-example-of-why-apples-docs-are-terrible/](https://www.amimetic.co.uk/blog/a-concrete-example-of-why-apples-docs-are-terrible/)

Apple's documentations is famously dreadful. In particular it is often missing or incomplete. [No overview available](https://nooverviewavailable.com/) gives you some numbers on this but I think that actually understates the problem. Even when it is available it is often terrible. Here is a small concrete example I recently encountered while doing some MacOS development (an area I'm less familiar with).

I want to prompt the user to open a directory. How do I do that? Obviously I Google it (throwing in `Swift` to avoid ancient `Obj-C` answers).

Seems like I want something called `NSOpenPanel`.

I find [Apple's docs](https://developer.apple.com/documentation/appkit/nsopenpanel) and it seems to be fairly clear. I configure with some straightforward options.

Okay how do I actually use it?

Hmmm. That is far from clear. There is literally nothing on the page that explains even the most basic use case. Sigh. Hit google again finding that I want to call `begin` and that that this accepts a callback which is given a result.

The example is out of date (a perpertual issue with Swift) but eventually sort that out in the editor (we now have a nice enum with the result).

Okay but how do I get the actual directory selected? The success value is `.OK`. Is there some kind of data with that? No. Just a result.

Hmmm...

There is a [delegate](https://developer.apple.com/documentation/appkit/nsopensavepaneldelegate) that I can set. Maybe that gets the result.

Waste a few minutes on that, basically appears to be for configuration.

Ah, it inherits from `NSSavePanel`! That was not at all obvious (and I'm sorry OOP devotees, that makes damn all sense in principle). Oh, look, loads of new functions and properties. Ah, so I'm meant to query the NSOpenPanel object for the directory.

Not really clear how. Xcode offers a `url`, `urls`, `directoryUrl`, `representedUrl` among others (I'm not making any of those up and most didn't seem to be in documentation). Let's be optimistic and chose `url`. Success! It appears to work.

Oh, also missing (before I continue): in recent versions of MacOS need to add a capability for reading (and writing) to user selected directories. Again completely missing from documentation and yet another frustration for someone not already very familiar with the ecosystem.

This can be so much better. Apple developers sneer at Electron, but look at how [clear Electron's open dialog docs](https://electronjs.org/docs/api/dialog#dialogshowopendialogbrowserwindow-options) are.

## What goes wrong with Apple's docs?

1.  Missing and incomplete
2.  Outdated (Obj-C or older Swift versioned docs are common)
3.  Badly written (assumes a level of familiarity that would make reading the docs unnecessary). Lack of explanation.
4.  Lack of examples of even basic usage.
5.  Badly designed, legacy OOP APIs: Swift can sometimes get away without much documentation through good use of types. Sadly a lot of MacOS API's are a blend of old fashioned OOP with surprisingly low level details.

## What if you are doing Apple development?

Avoid Apple's docs (even where they are top Google results). They will likely be pretty bad. Stack Overflow is sometime good (but often an old Swift version). Some third party docs and good (Paul Hudson, Ray Wenderlich).