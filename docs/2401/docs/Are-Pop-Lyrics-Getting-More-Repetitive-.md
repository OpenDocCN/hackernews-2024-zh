<!--yml
category: 未分类
date: 2024-05-27 14:56:12
-->

# Are Pop Lyrics Getting More Repetitive?

> 来源：[https://pudding.cool/2017/05/song-repetition/](https://pudding.cool/2017/05/song-repetition/)

In 1977, the great computer scientist Donald Knuth published a paper called [The Complexity of Songs](https://en.wikipedia.org/wiki/The_Complexity_of_Songs), which is basically one long joke about the repetitive lyrics of newfangled music (example quote: "the advent of modern drugs has led to demands for still less memory, and the ultimate improvement of Theorem 1 has consequently just been announced").

I'm going to try to test this hypothesis with data. I'll be analyzing the repetitiveness of a dataset of 15,000 songs that charted on the Billboard Hot 100 between 1958 and 2017.

## Measuring repetitiveness?

I know a repetitive song when I hear one, but translating that intuition into a number isn't easy. One thing we might try is looking at the number of unique words in a song, as a fraction of the total number of words. But this metric would call the following lyric excerpts equally repetitive:

> baby I don't need dollar bills to have fun tonight
> I love cheap thrills
> baby I don't need dollar bills to have fun tonight
> I love cheap thrills
> I don't need no money
> as long as I can feel the beat
> I don't need no money
> as long as I keep dancing
> 
> ~ Sia, Cheap Thrills

> tonight I need dollar bills
> I don't keep fun
> cheap thrills long to feel money
> the bills don't need the dancing baby
> fun dollar dancing thrills the baby I need
> don't have fun
> no no don't have dancing fun tonight
> beat the can as I don't feel thrills
> love the dancing money
> 
> ~ Colin Morris, Original composition

These are both 52 words long and use the same 23 word vocabulary, but the first one is clearly more repetitive, because it arranges words in a predictable, repetitive order.

## Repetitiveness ≈ compressibility?

You may not have heard of the [Lempel-Ziv algorithm](https://en.wikipedia.org/wiki/LZ77_and_LZ78), but you probably use it every day. It's a lossless compression algorithm that powers gifs, pngs, and most archive formats (zip, gzip, rar...).

What does this have to do with pop music? The Lempel-Ziv algorithm works by exploiting repeated sequences. How efficiently LZ can compress a text is directly related to the number and length of the repeated sections in that text.

Here's how it works.

You can explore some examples of the algorithm applied to full songs [here](http://colinmorris.github.io/pop-compression/). It turns out, for example, that the entire lyrics of *Cheap Thrills* reduce in size 76% when compressed.

Is 76% a lot? Here's the distribution of size reduction percentages across (almost) all 15,000 songs in my dataset:

## The Repetition of Pop Music

Distribution of compressibility of 15,000 songs from 1958 to 2017, excluding 20 outliers.

You may have noticed the percentages on the x-axis are not evenly spaced. I'm using a logarithmic scale with the property that, for any given song, a song that compresses half as small is a fixed distance away. For example, the distance between 20% and 60% is the same as between 98% and 99%. I'll be using this kind of scale throughout.

I excluded the 20 most repetitive songs in the dataset from the above chart. When you see what it looks like with them, you'll understand why:

What are these uber-repetitive outliers? *Around The World* by Daft Punk gets reduced a whopping **98%**. It goes from 2,610 characters to 61\. Small enough to fit in a tweet - twice! ([Here are the lyrics](http://www.azlyrics.com/lyrics/daftpunk/aroundtheworld.html), if you're not familiar with the song.) Check out some of the others below.

## The Most Repetitive Songs

Of 15,000 songs from the Billboard Hot 100

Intuitively, *Around The World* definitely *feels* like a very repetitive song. Several of the familiar entries on this list are essentially novelty songs (Macarena, Barbara Streisand, The Thong Song, Cotton Eye Joe...) famed for their silly refrains. This inspires some confidence in the metric I'm using.

So, is music getting more repetitive? The current decade is pretty well-represented in the top 10 above, but it's also a bit overrepresented in my dataset (it's easier to find lyrics for recent songs). We'll need to look at a lot more data to be sure...

## Who's responsible for this madness?

Let's look at the average repetitiveness of some prolific artists in the dataset (those that have at least 15 charting songs as solo artists).

## Repetitiveness Per Artist

Genre does seem like a differentiating factor here. In the 00's, our artists actually separate pretty cleanly into two clusters, with country music and hip-hop (and whatever John Mayer does) on the left, and pop and rock on the right.

The variation between artists is considerable. The Backstreet Boys have an average compression ratio of 60%, to Brad Paisley's 38%. In other words, if we asked the Backstreet Boys and Brad Paisley to each write a 400 word song, and compressed them both, we'd expect Brad's compressed song to be 50% bigger than BSB's.

Let's zoom in on a specific artist, say Gwen Stefani. Each circle below represents a song in her discography. The background blob is the histogram of *all songs* in the dataset (the same one as before, but mirrored).

On average, Gwen's music is extremely repetitive, but there's a wide spread between the incessantly repetitive "Hollaback Girl", and the laid-back ballad "Cool", which is far less repetitive than the median song in the dataset.

You can search for your favorite artist in the dropdown above. A few interesting examples (click an artist's name to jump to their chart):

*   People have described *1989* as the album where  fully transitioned from country singer/songwriter to popstar. This data adds some flavour to that claim. 4 of her 5 most repetitive songs are from *1989*.
*   has a long, consistent legacy of highly repetitive music. Interestingly, her two least repetitive songs are arguably the only two that don't belong to the pop genre: they're both from the musical *Evita*, and written by Tim Rice and Andrew Lloyd Weber.
*   Rappers like  and  tend to be consistently non-repetitive.
*   Is it any wonder  won the title of most repetitive artist in the dataset? Over half a dozen of her songs sit on the *far* right of the distribution (including, appropriately, *Pon de Replay*).

<main></main>