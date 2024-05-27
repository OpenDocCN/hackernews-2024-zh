<!--yml
category: 未分类
date: 2024-05-27 13:29:05
-->

# Understanding and avoiding visually ambiguous characters in IDs

> 来源：[https://gajus.com/blog/avoiding-visually-ambiguous-characters-in-ids](https://gajus.com/blog/avoiding-visually-ambiguous-characters-in-ids)

It is not uncommon that I need to write down or communicate IDs when interacting with systems, e.g. reporting a bug, entering a discount code, or tracking a package. It is frustrating when the experience is marred by an easy mistake to avoid - visually ambiguous characters.

*   `O` / `0` - The letter `O` and the number `0` can look very similar, especially in fonts where the number zero isn’t slashed or dotted.
*   `I` / `l` / `1` / `7` - The letter `I` (uppercase `i`), lowercase `l` (`L`), the number `1`, and the number `7` ^([1](#user-content-fn-1)) can be indistinguishable in many types of print and handwriting.
*   `5` / `S` - In some fonts, the number `5` and the letter `S` can appear quite similar.
*   `2` / `Z` - These can also be confused with each other, particularly in handwritten forms.
*   `8` / `B` - These characters might be mixed up when poorly written or in certain stylized fonts.
*   `6` / `G` - These characters can be confused in some fonts.
*   `9` / `q` / `g` - These characters can be confused in some fonts.

They cause confusion and errors in data entry, especially when the characters are handwritten or when the font is not clear. For example, if a user is trying to enter a code that contains the letter 'O' and the number '0', they might not be able to tell which character is which. This leads to frustrating user experiences.

A few examples using different system fonts:

*   9qg6G8B2Z5SIl170O (arial)
*   9qg6G8B2Z5SIl170O (helvetica)
*   9qg6G8B2Z5SIl170O (courier)
*   9qg6G8B2Z5SIl170O (times)
*   9qg6G8B2Z5SIl170O (verdana)
*   9qg6G8B2Z5SIl170O (georgia)
*   9qg6G8B2Z5SIl170O (tahoma)
*   9qg6G8B2Z5SIl170O (impact)
*   9qg6G8B2Z5SIl170O (comic sans)

Some pairs are visually ambiguous in all fonts (e.g. `I` and `l`), while others are much harder to distinguish in handwritten forms, e.g. try writing `9qg` in your own handwriting.

Any time that the ID might need to be communicated verbally or written down, e.g.

*   Customer support
*   Discount codes
*   Tracking codes
*   Error IDs
*   Product IDs
*   etc.

You may also consider whether your IDs should be case-sensitive or not. Is `abc` the same as `ABC`?

Assuming that you are going with case sensitivity, you have 53 characters to choose from (adjusted for visually ambiguous characters). On the other hand, if you decide to make your IDs case-insensitive, you have only 22 characters to choose from.

Assuming an ID length of 5 characters, you have the following number of possible IDs:

*   Case-sensitive: 53^5 = 418,195,493
*   Case-insensitive: 22^5 = 5,153,632

However, as the number of members in the set increases, the number of possible IDs increases exponentially.

*   Case-sensitive: 53^8 = 62,259,690,411,361
*   Case-insensitive: 22^8 = 54,875,873,536

Therefore, the real question is whether what's preferable is a shorter ID with a higher chance of visual ambiguity or a longer ID with a lower chance of visual ambiguity.

As pointed out in a Hacker News comment ^([2](#user-content-fn-2)), if you use both upper and lower case, you are likely to eventually be surprised by some third party system or protocol that is case insensitive.

> I even found a commercial system which allowed users to choose IDs with case sensitivity (iD and id being distinct) but if you query it for one which does not exist they do case insensitive matching and return the wrong data.
> 
> When I reported this bug they said it was for convenience!

As my preference is readability, this is the character set that I use for generating IDs in my projects:

```
[
 "a",
 "b",
 "c",
 "d",
 "e",
 "f",
 "h",
 "i",
 "j",
 "k",
 "m",
 "n",
 "o",
 "p",
 "r",
 "s",
 "t",
 "w",
 "x",
 "y",
 "3",
 "4"
]
```

*   `rn` (looks like `m`) ^([3](#user-content-fn-3))
*   `vv` (looks like `w`) ^([4](#user-content-fn-4))

Personally, I would be wary of excluding characters just because they look like other characters when combined. Avoiding these particular combinations might be a good idea at the ID generation level.

In some cases, you might also want to avoid characters that sound similar when spoken. For example, `b` and `p` can sound similar when spoken out loud. This can be especially important in situations where IDs are communicated verbally.

Crockford’s [Base32](https://www.crockford.com/base32.html) (distinct from the [IETF Base32](https://www.rfc-editor.org/rfc/rfc4648#page-8))

> This takes the approach of allowing ambiguous characters by decoding them to the same value, and also considers the problem of accidental obscenities.

[Open Location Code](https://github.com/google/open-location-code)

> A character set I like to use when I need something like this is the one used by Open Location Code, which is 23456789CFGHJMPQRVWX. It was apparently chosen not only to avoid visually ambiguous characters, but also to [avoid spelling words in common languages](https://github.com/google/open-location-code/wiki/Evaluation-of-Location-Encoding-Systems#open-location-code). It does however include both 6 and G, as well as 9 and Q.