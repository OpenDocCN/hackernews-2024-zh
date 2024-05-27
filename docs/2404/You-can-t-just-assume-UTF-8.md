<!--yml
category: 未分类
date: 2024-05-27 13:37:36
-->

# You can't just assume UTF-8

> 来源：[https://csvbase.com/blog/9](https://csvbase.com/blog/9)

Humans speak countless different languages. Not only are these languages incompatible, but runtime transpilation is a real pain. Sadly, every [standardisation](https://en.wikipedia.org/wiki/Esperanto) [initiative](https://en.wikipedia.org/wiki/Lojban) has failed.

At least there is someone to blame for this state-of-affairs: God. It was him, after-all, who cursed humanity to speak different languages, in an [early dispute over a controversial property development](https://www.kingjamesbibleonline.org/Genesis-11-7/).

However, mankind can only blame itself for the fact that *computers* struggle to talk to each other.

And one of the biggest problems is the most simple: computers do not agree on how to write letters in binary.

## How to write letters in binary

How are letters encoded into binary?

Let's take the character "A" for example. It was assigned the number 65 in the American Standard Code for Information Interchange, or [ASCII](https://man7.org/linux/man-pages/man7/ascii.7.html). This numbering was [grandfathered into](https://codepoints.net/U+0041) Unicode, except that they the Unicode people write the number 65 in hexadecimal, as `U+0041`. And they call it a "codepoint".

Easy enough - at least the number for "A" enjoys wide consensus. But computers can't just store decimal numbers, they can only store binary.

In the most popular character encoding, UTF-8, character number 65 ("A") is written:

`01000001`

Only the second and final bits are `1`, or "on". The second bit is worth 64 and the last bit is worth 1\. Those two sum up to 65\. Easy peasy.

Another popular encoding is UTF-16, mainly in the world of Windows, Java and JavaScript. 65 is represented in UTF-16 this way:

`01000001 00000000`

That's much the same, except the UTF-16 uses (at least) two full bytes for each character, but doesn't need any more bits to represent 65, so the second byte is blank.

What about the other encodings? Here are just a few of the other popular ones:

*   [Win-1252](https://en.wikipedia.org/wiki/Windows-1252), a non-Unicode encoding, used where European languages are spoken
*   [KOI8](https://en.wikipedia.org/wiki/KOI8-RU) - non-Unicode, used where the [Cyrillic alphabet](https://en.wikipedia.org/wiki/Cyrillic_script) is
*   [GB18030](https://en.wikipedia.org/wiki/GB_18030) - is Unicode, but mainly used in mainland China
*   [Big5](https://en.wikipedia.org/wiki/Big5) - non-Unicode, widely used where Traditional Chinese is
*   [Shift_JIS](https://en.wikipedia.org/wiki/Shift_JIS) - non-Unicode, used in Japan

Well, all these other encodings also grandfathered in the ASCII letters, so they all represent `A` as:

`01000001`

Just like UTF-8 does.

So happy days. This is why the basic western alphabet usually works even when the rest of the document is [a garbled mess](https://en.wikipedia.org/wiki/Mojibake). Many popular encodings - with the notable exception of UTF-16 - follow ASCII at least for the Latin alphabet.

Well, so far, so good. But now let's look at a trickier character. The euro sign, €. It is numbered [8364](https://codepoints.net/U+20AC) (`U+20AC`) by the Unicode consortium.

In UTF-8, 8364 is represented by:

`11100010 10000010 10101100`

Notice that it takes three bytes in UTF-8\. UTF-8 is a "variable-length" character encoding - the higher the Unicode number goes, the more bytes are required. (This is actually true for UTF-16 as well, but more rarely.)

In UTF-16 however, 8364 is encoded completely differently:

`10101100 00100000`

Win-1252, however, doesn't follow Unicode. Instead, it numbers the euro sign as 128\. Then it represents 128 this way:

`10000000`

So just the first bit is on, worth 128.

And this is where the trouble starts. Once you leave the leafy suburbs of the English alphabet, the representations quickly go to hell in an hand-basket.

`€` is completely unrepresentable in KOI8.

In GB18030, € is encoded:

`10100010 11100011`

In Big5, € is:

`10100011 11100001`

In Shift JIS, it is:

`10000000 00111111`

All completely different and completely incompatible. If you assume UTF-8 you will get absolute nonsense.

## How to tell which encoding is being used?

Some formats dictate the encoding, as JSON mandates UTF-8\. That certainly keeps things simple - as soon as you know it's JSON, you know it must be UTF-8.

Other times, it's possible to transmit the encoding "out-of-band". HTTP allows you to put the encoding in the `Content-Type` header:

```
Content-type:  text/html;  charset=ISO-8859-1 
```

And some formats have an "in-band" way to signal the encoding. For example, some text files have the header:

```
# -*- encoding: utf-16be -*- 
```

Though that's a bit of a mind-bender because you need to be able to read the file already in order to find that bit.

But what if there is no label?

And what if the label is wrong? As I will come on to, this is not uncommon.

CSV files, in particular, have no way to signal, in-band, which encoding is used. You can't put a comment, because there is no space - and csv readers will be unable to parse your csv in most cases. And many popular tools that work with CSV files (MS Excel) do not use UTF-8.

What then?

The answer is: statistics.

## Detecting the encoding with statistics

There are two basic strategies which are used to detect the encoding of an unlabelled text string.

1.  Byte-level
2.  Character-level

Most implementations use byte-level first, then character level next, if necessary.

### Byte level heuristics

The byte level is fairly straight forward. You can just look at the bytes and say whether they are plausible for a given character encoding.

For example, as I said above, UTF-16 (mostly) uses two bytes per character. For Latin text (eg English) that tends to result in a lot of "blank" second bytes. Happily, many markup languages use a lot of Latin text (eg `<`, `>`, `[`, `]` and so on), even if the document is not in a Latin script. If your test string contains lots and lots of blank second bytes - well, then it's likely that it's UTF-16.

There are other giveaways. Imagine you are a web-browser and a link has taken the user to a file whose first two bytes are these:

`00111100 00100001`

If this were UTF-16, these two bytes would be ℼ, a symbol called ["double struck small pi"](https://codepoints.net/U+213C), numbered in Unicode as 8508 (U+213C). Is that a common first character in an HTML file?

Or, perhaps, is it more likely to be the two character sequence `<!` encoded with UTF-8? Perhaps the following bytes are `doctype>`, or the standard boilerplate of any HTML document? Just a hint!

Another giveaway are specific bytes. UTF-16 frustratingly comes in two versions, one where you write the bits in the normal way and one where you write them all backwards. In order for people to tell these two forms apart the standard for UTF-16 includes a ["byte order mark"](https://en.wikipedia.org/wiki/Byte_order_mark) that you can put at the front of a text stream to indicate which version you've used. This pair of bytes doesn't occur very often in other encodings, and certainly not at the start, so it's usually a good hint about what follows.

So bytes can get you pretty far. If you can narrow it down unambiguously to UTF-8 or UTF-16 at this point, your job is finished.

### Character level heuristics

The tricky cases tend to be non-Unicode single byte encodings. For example, telling Win-1252 apart from KOI8, as both use the otherwise blank 1st bit of ASCII to encode different things.

How do you tell these apart? By frequency analysis. You look at the letters that would be present in the document, for example if it were KOI8 and ask "is this a typical distribution of letters for a Cyrillic document?".

Here is a basic algorithm:

*   Exclude all encodings ruled out by prior byte-level heuristics
*   For each remaining possible encoding `X`
    *   Parse the input as though it were encoded with `X`
    *   Compare the frequencies of the characters in the string with those in a known frequency table
    *   Optionally, also compare the *pairs* of letters (eg `qu`) with a known frequency table
    *   If they are a good match, return `X`
*   Otherwise, fail

Often you can also tell which language the document is in as well this way - that's one way that web browsers know to pop up the "do you want to translate this page" dialogue box.

## Does it really work?

Heuristics tend to make people uneasy but the answer is: yes. It works surprisingly well. It certainly works far better than simply assuming that the text is UTF-8 - and ultimately that is the benchmark.

It probably should not be a surprise the statistics work well. They often work well on languages, from the [first effective spam filters](https://paulgraham.com/better.html) to, well, [many things](https://research.google.com/pubs/archive/35179.pdf).

Heuristics are also important because people get the encoding wrong.

It would seem reasonable to assume that if you exporting a Excel sheet as a csv file in the most recent release of MS Excel that you would get UTF-8\. Or maybe UTF-16\. You would be wrong. By default, in most configurations, Excel will output your CSV encoded in Win-1252.

Win-1252 is a non-Unicode, single byte encoding. It is an extension of ASCII that rams enough characters for almost every European language into the unused 8th bit. The typical Excel user has never heard of it, if they have heard of character encoding at all. [Ignorance is bliss](https://www.kingjamesbibleonline.org/Ecclesiastes-1-18/).

## See also

Probably the majority of encoding detection code runs off the principles established by Netscape in the early 2000s. The original authors' paper describing their approach [is in Mozilla's archive](https://www-archive.mozilla.org/projects/intl/universalcharsetdetection).

There is a definite sense that auto-detecting text encoding is an instance of [Postel's Law](https://en.wikipedia.org/wiki/Robustness_principle): "be conservative in what you do, be liberal in what you accept from others". Postel's Law is something I always accepted as true but is now [increasingly controversial](https://datatracker.ietf.org/doc/html/rfc9413). Perhaps there is a case to be made that csvbase's auto-detection should be a more explicit user interface action than a pre-selected combo-box. Answers on [a postcard, to the usual address](mailto:cal@calpaterson.com).