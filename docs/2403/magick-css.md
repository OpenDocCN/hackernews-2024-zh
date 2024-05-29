<!--yml
category: 未分类
date: 2024-05-29 12:34:17
-->

# magick.css

> 来源：[https://css.winterveil.net](https://css.winterveil.net)

<main>

> In the grand compendium of universal knowledge, it is universally acknowledged that the scribbles of a wizard not only transcend the mundane but are, without a shadow of a doubt, the most exhilarating form of notation known to humanoid kind. For who could resist the allure of a well-placed spell in the margins?
> 
> Albus Greybeard, On the Virtues of Mystical Marginalia, 1132

## Fundamentals

magick.css is a minimalistic, (mostly) classless CSS framework that is designed to be easy to use and easy to understand. It is contained in a single file, and every choice is commented. The goal is to achieve an elegant, but magically playful look, while maximizing readability and the ability to convey information; *somewhat akin to the notes of a wizard*.

The framework retains its beauty and functionality across **all devices** and screen sizes, and works entirely **without JavaScript**. It is inspired by [LaTeX](https://www.latex-project.org/), old school TTRPG rulebooks, CSS frameworks like [concrete.css](https://concrete.style/) and [Tufte CSS](https://edwardtufte.github.io/tufte-css/) and the "usability as design" ethic of brutalism.

## How to use magick.css

The easiest way to check out magick.css for your own project is to add it from a CDN. Just add these two lines to your html `<head>`:

```
<link rel="stylesheet" href="https://unpkg.com/normalize.css">
<link rel="stylesheet" href="https://unpkg.com/magick.css">
```

You could also just download the magick.css file from this repo, and include it into your html `<head>` as you normally would:

```
<link rel="stylesheet" href="path/to/magick.css">
```

You could also add magick.css to your JS project like this:

```
npm install normalize.css magick.css
```

And import them in your code:

```
import 'normalize.css'
import 'magick.css'
```

For anything beyond that: magick.css should work with any HTML5 document, as it is 99% classless. There are some cool features that you can use, though; just scroll down to discover them!

## Layout

To have your page structured as a responsive, readable column, wrap all of your content in `<main>` tags, and then use `<section>` tags to break up long form content content into sections. This page is a good example of how to do this.

(You could of course just ignore this advice and use magick.css for typography and styled components only.)

Here is a simple example of the usual structure:

```
<html>
  <head>
  </head>
  <body>
    <main>
      <header> </header>
      <section> </section>
      <section> </section>
      <section> </section>
      <footer> </footer>
    </main>
  </body>
</html>
```

To add tangential information, magick.css uses the `<aside>` tag. You will find it used from time to time on this page. Should you want to elaborate on a point, or provide additional context, you can use magick.css's sidenotes. Sidenotes are automagically numbered! You can find out more about this in the [Extras](#extras) section below.

## Typography

Here is a showcase of the typography that magick.css offers:

# Heading 1: A title bold and grand, the story's gate.

## Heading 2: A subtitle beneath, where chapters await.

### Heading 3: With secrets whispered, plots thicken here.

#### Heading 4: Fine details shared, the message stands clear.

Paragraphs flow, a river of text,

[Links like portals, where will we go next?](#)

**Strong as stone, our words firmly stand,**

*Emphasized whispers, softly they land.*

Underlined secrets, subtly hinted,

Small, yet mighty, our words are minted.

`Code speaks in logic, crisp and neat,`

Typography's spell, now complete.

Headings below `<h4>` are not styled by magick.css; they are usually not needed for hierarchical structuring, but instead for very specific use cases. So should you need them, you can style them as you see fit.

> [It is] notable that the Feynman lectures [...] write about all of physics in 1800 pages, using only 2 levels of hierarchical headings: chapters and A-level heads in the text.

## Structured Content

### Lists

Lists are kept simple, as to not distract from the content:

*   This is...
*   ... an unordered list.

1.  This is...
2.  ... an ordered list.

And this weird fellow here...

... is a description list.

### Tables

In a similar manner to lists, tables are clear and concise, directing reader focus to the data at hand:

| Potion | Main Ingredient | Catalyst |
| --- | --- | --- |
| Glow Dew | Bioluminescent algae (81mg) | 🜼 |
| Float Leaf | Dandelion puff (61mg) | 🝆 |
| Might Brew | Ginseng root (78mg) | 🝀 |
| Vanish Draft | Mimosa pudica (26mg) | 🜞 |
| Remembrance Tea | Ginkgo biloba (39mg) | 🜇 |

## Code, Quotes and Preformatting

### Code

Displaying code can be done either inline or in a distinct block. To inline code, use the `<code>` tag. For larger pieces of code, use `<pre>` to wrap the `<code>` tag. By the way, you can also use figures to display code snippets!

```
123456789void fib(int x) {
    unsigned long a = 0, b = 1, c;
    for(int i = 0; i < x; i++) {
        printf("%lu ", a);
        c = a + b;
        a = b;
        b = c;
    }
}
```

This C function generates numbers of the fibonacci sequence.

(Line numbering of code requires some extra preprocessing of the content, but it is quite simple to do, even just by hand. Find out how in the [Extras](#extras) section.)

### Quotes

For quotes, use the `<blockquote>` tag. Depending on the content, you might want to use preformatted text or let the browser handle the formatting:

> It was a bright cold day in April, and the clocks were striking thirteen.

Adding a citation is done using a `<footer>` inside the blockquote. Just look at the source code of this page to see how it's done!

Let's now consider a poem, where the formatting is imporant and can't be dynamically determined by the browser. In this case, we wrap the poem contents in a `<pre>` tag:

> ```
> The angels, not half so happy in Heaven,
>    Went envying her and me— 
> Yes!— that was the reason (as all men know,
>    In this kingdom by the sea)
> That the wind came out of a cloud, chilling
>    And killing my Annabel Lee.
> ```
> 
> Excerpt from Annabel Lee by Edgar Allan Poe

## That's it!

I hope I could interest you in using magick.css. If you like what you see, please consider giving it a star on [GitHub](https://github.com/wintermute-cell/magick.css). It costs nothing and shows me that **someone cares**!

Should you be interested in what other things I build, join my [Discord](https://discord.gg/f8nyyK4ngZ)!

</main>