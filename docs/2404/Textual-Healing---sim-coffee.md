<!--yml
category: 未分类
date: 2024-05-27 13:06:54
-->

# Textual Healing – sim.coffee

> 来源：[https://sim.coffee/textual-healing/](https://sim.coffee/textual-healing/)

[It’s a Token Issue](#tokenizer)
[The Result](#results)
[Surprise](#surprise)

We’re seeing a lot of activity on [our Discord](https://discord.gg/7fpmbqfXcJ) lately. People are using [Codea](http://itunes.apple.com/app/id439571171?mt=8), and they’re complaining when it doesn’t work! Which is *freaking awesome*

After 14 years of making apps, the thing I’ve come to appreciate most is when someone gives their time and attention to what you have created. To the point where it bugs them when it doesn’t work how they expect, and they tell you about it. You should appreciate every one of these people, even the frustrated ones who are running short on patience

Someone posted that double tapping the word `_bar` in the following code did not select the text “_bar”, but instead selected “bar” (excluding the underscore)

```
function  setup()
 foo._bar  =  5
end
```

I knew immediately why this was happening. Our `UITextInputTokenizer` was why it was happening. I checked out my subclass of `UITextInputStringTokenizer`^([1](#165e6f52-80ea-42ba-9a36-d82931aa4386))

```
//
//  JAMCodeTokenizer.m
//  Jam
//
//  Created by Simeon on 11/11/2013.
//  Copyright (c) 2013 Two Lives Left. All rights reserved.
//
```

*2013*. I started this file eleven years ago

## It’s a Token Issue

`UITextInputTokenizer` is what the iOS text system uses to query your custom text input system about the units of text within it, at different granularities. It has a wild and esoteric API that encompasses the following four methods:

```
func  isPosition(UITextPosition, 
 atBoundary: UITextGranularity, 
 inDirection: UITextDirection) ->  Bool
 func  isPosition(UITextPosition, 
 withinTextUnit: UITextGranularity, 
 inDirection: UITextDirection) ->  Bool
 func  position(from: UITextPosition, 
 toBoundary: UITextGranularity, 
 inDirection: UITextDirection) -> UITextPosition?
 func  rangeEnclosingPosition(UITextPosition, 
 with: UITextGranularity, 
 inDirection: UITextDirection) -> UITextRange?
```

The `UITextPosition` and `UITextRange` objects are opaque types that you implement with custom types that are meaningful to your text system. (That is, Apple’s text system doesn’t care what they are, so long as it can give them to you and get meaningful results back. For example, Codea uses `AVAudioPlayer` for `UITextPosition`s, and `CLLocation2D` for `UITextRange`^([2](#9df35c91-6dc2-4036-9ce0-a9f104e5dc70)))

The methods basically query your text system for *boundaries* with *granularities* in *directions*. Boundaries are the “edges” of text in whatever the specified granularity is — word, sentence, paragraph, document, and so on. “Is this *position* at the *edge* of a *word* if I’m moving *forward*?” and you reply with “Why, yes. It is”, or no

This is very important for keyboard navigation and selection. When you hold option and hit the arrow keys, you jump across by word. This is how the text system understands what a “word” is and where the next one lives. Same thing for double-tapping a word to select, or triple-tapping to get a line

Apple provides `UITextInputStringTokenizer` (note the `String` in there), a concrete subclass of the `UITextInputTokenizer` protocol, so you don’t have to write your own. Being lazy, we used this as the basis for our code tokenizer a long time ago

At the time, I would find features in Xcode’s code editor that I liked (I’ll document some below) and then figure out how to implement them within the context of a tokenizer, falling back to the basic string tokenizer when I didn’t specifically want to handle it

Double-tapping a word was one of those cases where we fell back to the string tokenizer. The problem is, the string tokenizer is designed for natural language, not code. Words in English don’t typically include underscores, and so they are not selected because they form a boundary at the word granularity level, and the `rangeEnclosingPosition` method will not include them as part of a word

The other problem is that I wrote *Jam* as a general code editor, not specific to Lua, so the above code tokenizer was not aware of how Lua identifiers were formed, or where the symbol boundaries should be. I had focused our code tokenizer on how to navigate whitespace and allow for exact caret placement^([3](#51675be5-f78f-475a-9933-fae14827f803))

## The Result

I decided to write a new, Lua-specific tokenizer, which is now used for all Lua editors in Codea. But man, is the `UITextInputTokenizer` API tough to implement in a way that doesn’t end up as a mess of special cases! Below are the cases I handled, with everything else falling back to original code tokenizer:

#### Command back arrow for indented code

Go to the end of a line of indented code in Xcode

Hit ⌘←

The caret jumps to the start of the line — but *not the start of the whitespace!*

Hit ⌘←

The caret jumps to the start of the line, at the start of the whitespace

Here it is in Codea^([4](#acc4a566-1df1-40d3-ae0a-1fe2517dd8c7))

#### Exact caret placement

Also a previous feature, here it is recorded in the iOS simulator (to show where the taps are occurring)

#### Respect for symbol boundaries

Below is navigating with the option key to jump by “word” (symbol). The ugly, `_main_Test` member is related to the original bug report. You can see we now traverse the symbol as a single entity

#### Supplying ranges for symbols as “words”

Demonstrating fixed behaviour around selecting “_” when they form part of an identifier

#### Falling back to natural language tokenization

Of course, because quoted strings are considered symbols in the code editor, double-tapping on one in this system would typically select the entire string. In cases like this, we exclude these symbols and defer to the `UITextInputStringTokenizer` to get the regular text editing experience when inside strings, comments, and so on

## Addendum

As I sat down to write this post, I had finished rewriting the Codea tokenizer last night, Sunday April 7, 2024, publishing a new beta build and promptly falling asleep

And I began this piece by talking about user *complaints*. But as I was half-way through writing it tonight, a very lovely and dear email came into my inbox. It is as follows

> *I suspect you don’t get enough of this kind of beta-feedback, so..:*
> 
> *Unless I’m very much mistaken, you have, at some point, made the cursor-positioning and line-selection in the editor work much better than it used to, at least for me.*
> 
> *I used to have trouble all the time, when selecting lines using the line-number gutter and conversely when trying to place the cursor at the beginning of a line (these two operations would get mixed up in other words), but now it feels much easier / needs less precision or whatever made me fail before.*
> 
> *It is so often these minute quality-of-life things that makes all the difference, especially when you use them all the time (ie. most features in a code-editor I guess, if taken across the whole user-base).*
> 
> *I seem to remember bitching to you about this at least a couple of times (my walls have certainly listened to a LOT of it), so it only stands to reason that I also take the time to dredge the following up from the bottom of my heart (or whatever blackened piece of charcoal is left):*
> 
> *Thank You VERY Much!*

Someone noticed!