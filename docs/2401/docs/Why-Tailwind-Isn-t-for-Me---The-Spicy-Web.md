<!--yml
category: 未分类
date: 2024-05-27 15:12:43
-->

# Why Tailwind Isn’t for Me | The Spicy Web

> 来源：[https://www.spicyweb.dev/why-tailwind-isnt-for-me/](https://www.spicyweb.dev/why-tailwind-isnt-for-me/)

> *Breaking News!* The brand-new course **[CSS Nouveau](/css-nouveau)** is now available! This is the first release in the **THE SPICY WEB** Courses Series. **[Go take a look around and sign up today!](/css-nouveau)**

**August 2022 Update**: Still working on that design systems course (😅), but in the meantime, I’ve written [The Three Laws of Utility Classes](/the-three-laws-of-utility-classes/) and announced [Vanilla Breeze](https://www.vanillabreeze.dev), a new open source tool which will convert Tailwind “class soup” into clean, portable, vanilla HTML + CSS! Progress…

**March 2022 Update**: Well the JIT is now the default way of managing Tailwind output generation, so that’s cool. Unfortunately, Tailwind’s purview has only grown in directions that are [breathtaking in their weirdness](https://tailwindcss.com/docs/adding-custom-styles#using-arbitrary-values). I remain as dubious of TW as ever, and in fact have started writing a course *specifically designed* to teach people how to switch *away* from Tailwind and use the best of today’s “vanilla” CSS. **Stay tuned.** 😁 –JW

**March 2021 Update**: [the experimental new JIT (Just-In-Time) compiler for Tailwind](https://twitter.com/tailwindcss/status/1374675986965528576?s=21) has the potential to alleviate some of the concerns outlined here and also provides some intriguing new benefits. I haven’t tried it out yet, but once I do I’ll formulate additional thoughts and link to them from here. –JW

* * *

I’ve gotten into more than one heated argument on the interwebs lately over [Tailwind CSS](https://tailwindcss.com). I’m not proud of this. I don’t like being at odds with anybody. I think the folks building Tailwind are talented people. But at a pure technical level, I simply don’t like Tailwind. Whoever it was built for, **it was not built for me**.

And in one sense, that’s fine. There are *tons* of web technologies out there which I’ll never use. Doesn’t mean they’re bad. There are plenty of groovy tech stacks to go around.

The problem I keep running into however is this increasing popular sentiment that Tailwind *is the future* (man). It’s *the way things should be done*. In other words, it’s opinionated and it inspires a cadre of evangelists. Again, on a certain level, that’s fine. Rails is very opinionated, for example, and I love using Rails.

But Tailwind definitely throws down a gauntlet. I’ll quote directly from creator Adam Wathan highlighted right on the Tailwind website:

> “Best practices” don’t actually work.
> 
> I’ve written a few thousand words on why traditional “semantic class names” are the reason CSS is hard to maintain, but the truth is you’re never going to believe me until you actually try it. If you can suppress the urge to retch long enough to give it a chance, I really think you’ll wonder how you ever worked with CSS any other way.

Challenge accepted.

I’ve tried it. And I’ve used it. A lot. A project one of my largest clients has me developing is built on top of React and Tailwind. So whatever you may come at me with, you can’t accuse me of not giving Tailwind the good ol’ college try.

Still not my thing. At all. In fact I have some real concerns about Tailwind, and what I find supremely frustrating is whenever I raise these concerns, I get immediate pushback from die-hard Tailwind fans who accuse me (in so many words) of just being a fucking idiot. As a programmer who has worked full-time in the web industry since the late 90s, that just doesn’t sit right with me.

So since Twitter and Hacker News comments are apparently poor mediums for technical conversations of this magnitude, I will now attempt to outline the very real reasons why Tailwind is not for me.

This first reason is an aesthetic concern, yet it’s intimately related to real technical challenges which I’ll outline shortly. But at the very least, I **hate** the way utility-css-only HTML looks. Hate, hate, hate it. Adam even acknowledges this head on when he begs us to “suppress the urge to retch long enough to give it a chance…”. This is a tacit admission that writing markup this way initially seems ugly and weird—but somehow we’ll eventually just “get over it” because the benefits are so great.

After a year of writing Tailwind, I haven’t gotten over it. Sorry folks! You’ll *never* get me to appreciate this:

```
<div class="min-w-0 flex-auto space-y-0.5">
  <p class="text-lime-600 dark:text-lime-400 text-sm sm:text-base lg:text-sm xl:text-base font-semibold uppercase">
    <abbr title="Episode">Ep.</abbr> 128
  </p>
  <h2 class="text-black dark:text-white text-base sm:text-xl lg:text-base xl:text-xl font-semibold truncate">
    Scaling CSS at Heroku with Utility Classes
  </h2>
  <p class="text-gray-500 dark:text-gray-400 text-base sm:text-lg lg:text-base xl:text-lg font-medium">
    Full Stack Radio
  </p>
</div> 
```

Now I already hear many of you screaming at your computer screens to tell me “dude, just use `@apply` if you want to keep your HTML clean! Problem solved!” Well, that is a potential solution, and in fact that’s what we’ve done on the aforementioned project. Much of our HTML is oriented around component-scoped class names (fairly close to BEM in concept) and thus we use `@apply` extensively. But that brings me to my next concern.

### Reason 2: `@apply` is fundamentally incompatible and non-standard (and largely unnecessary). [#](#reason-2-apply-is-fundamentally-incompatible-and-non-standard-and-largely-unnecessary)

This is where a lot of Tailwind fans get tripped up and keep on arguing with me over and over again, so I’ll try to explain this as clearly and obviously as possible.

1.  `@apply mt-3` in a CSS file *only* works if you use Tailwind. It requires the presence of Tailwind in your build process. If you remove Tailwind from your build process, that statement doesn’t work and your CSS is broken.
2.  While it’s true you can take the generated output CSS of a site and use that without Tailwind, it’s typically a bundled compilation of dozens if not hundreds of small CSS files scattered around a codebase (if you write CSS-per-component files like we do). It’s not something you can count on for source code.
3.  Therefore, it’s simply the truth that CSS files built for Tailwind are non-standard (aka proprietary) and **fundamentally incompatible** with all other CSS frameworks and tooling. Once you go Tailwind, *you can never leave*. (da da dum 😱)
4.  And as an added bonus, writing all your CSS files with `@apply` everywhere basically means you’re not learning and authoring CSS. You’re authoring Tailwind. No matter how many times you write `@apply flex`, that’s **not** the same as writing `display: flex`.

Now I realize most of us aren’t in the habit of trying to swap out CSS frameworks on projects on a regular basis. But believe me, I have done this! I’m on a client project right now where we’re migrating from Foundation to Bulma. While it’s true that it requires updating a bunch of HTML and some of the stylesheets in use, rest assured any custom bits of styling we wrote before will work again without hassle, because when you write plain ol’ CSS (or even Sass), it just works no matter what.

And while `@apply` seems cool on the face of it, it ends up becoming an enormous crutch. For example, I like the way Tailwind makes writing styles using CSS Grid techniques pretty straightforward. Unfortunately, after having done so, I still don’t really understand Grid syntax itself. I remain ignorant of the open CSS standard.

As for why `@apply` in the grand scheme of things is largely unnecessary, that brings me to my third point.

### Reason 3: Tailwind’s focus on design systems and tokens could mostly be replaced by CSS Custom Properties (aka variables)—which IS a standard. [#](#reason-3-tailwinds-focus-on-design-systems-and-tokens-could-mostly-be-replaced-by-css-custom-properties-aka-variableswhich-is-a-standard)

People initially like Tailwind because it comes out-of-the-box with a nice design system and lots of tokens you can tweak (colors, font sizes, spacing, etc.). It’s easy to get good-looking results quickly.

The problem is that all these tokens are defined…in JavaScript. A CSS framework. Using JavaScript for its design tokens. In 2021.

I hate to break it to you, but all modern browsers support this thing called CSS Custom Properties. You can define design tokens once at the `:root` level as variables, and utilize them **everywhere**. You can even modify them in real-time while the site is loaded, or overload them in particular parts of the DOM tree. And they work *great* with web components. More on that in a moment.

So for example, in Tailwind you can write `class="mb-8"` and you get a `margin-bottom: 2rem` style applied. But guess what you could do instead? Define `:root { --spacing-8: 2rem }` in your stylesheet, and then write `margin-bottom: var(--spacing-8)` anywhere you want. As in literally anywhere: a stylesheet, or a JS component, *or even a* `style=` *attribute directly in HTML!*

While the story gets a little murkier once you start looking at how to accommodate responsive breakpoints and so forth, nevertheless the principle here is that Tailwind uses a non-standard JavaScript-based build process for its design system at a time when you can build design systems using syntax that’s *native* to all modern browsers.

Speaking of what’s native in modern web browsers…

### Reason 4: Tailwind forgets that web components exist. [#](#reason-4-tailwind-forgets-that-web-components-exist)

This is perhaps the biggest knock against Tailwind. It seemingly was conceived and promoted in a world where web components don’t exist. Tailwind CSS is completely unusable within the Shadow DOM. Some enterprising developers have come up with solutions where select bits of Tailwind styling can get injected into components through a build process, but it’s definitely a hack.

Meanwhile, there are ways to build web component-based design systems today where global theming and component styling via the Shadow DOM (and exposed Parts) all work together. Again, you can do all this based on technology that’s built-in and native to all modern browsers. And before you shrug your shoulders and go back to your React or your Vue, bear in mind that web components are not only an integral part of the HTML/CSS/JS spec today but are increasingly at the heart of further advancements to browser technology (for example how advanced customization of form controls might work in the future).

Tailwind in this respect is no more helpful to you than Bootstrap or Foundation or any other CSS framework written years/decades ago. (Even my beloved Bulma! 😢)

### Reason 5: Finally, Tailwind encourages div/span-tag soup. [#](#reason-5-finally-tailwind-encourages-divspan-tag-soup)

I almost included this in the previous point, but it really bears its own conversation. I have become convinced by now that using `<div>` and `<span>` tags everywhere in your markup is an anti-pattern. We live in a world where custom elements (aka `<whatever-you-can-dream-of>`) are fully supported and enabled by modern browsers. There’s virtually no reason you’re forced to write `<div class="card"></div>` when you can write `<ui-card></ui-card>`. And in fact it’s quite possible to use custom attributes along with elements to write *extremely expressive markup* that—compared to ye markup of ol’—looks quite futuristic!

Take the [Shoelace](https://shoelace.style) web component library for example. Here’s a button:

```
<sl-button type="default" size="small">
  <sl-icon slot="prefix" name="gear"></sl-icon>
  Settings
</sl-button> 
```

And here’s a modal:

```
<sl-dialog label="Dialog" style="--width: 50vw;">
  Lorem ipsum dolor sit amet, consectetur adipiscing elit.
  <sl-button slot="footer" type="primary">Close</sl-button>
</sl-dialog> 
```

Note that this isn’t JSX. This isn’t XML. This isn’t some kind of fancy-pants template language you have to convert to ordinary HTML.

**This is HTML.** This is what modern markup can look like.

Compare that to an example from Tailwind’s home page:

```
<button class="hover:bg-light-blue-200 hover:text-light-blue-800 group flex items-center rounded-md bg-light-blue-100 text-light-blue-600 text-sm font-medium px-4 py-2">
  New
</button> 
```

Ewwwww. 🤢

There is a future world of HTML/CSS/JS (and in large part it’s here already) where you can write bespoke Grid/Flexbox layouts quickly and easily with vanilla CSS, set up design tokens with CSS variables, utilize a well-architected web component library like Shoelace (or even mix ‘n’ match two or three), and end up with a website/app that **looks amazing** and works quite well—all without needing *any* of the many megabytes of Tailwind utility classes that you then need to purge to get your performance back down to manageable levels.

In other words, Tailwind’s main selling point (besides rapid prototyping via utility classes) is its attractive design system—yet the way it implements that design system really kind of sucks! (Incompatible with web components by default, only minimally leverages CSS variables, doesn’t encourage custom elements/attributes with relevant scoped styling…)

Which begs the question: how does Tailwind enable us to “build modern websites” exactly? On a pure technical level, I honestly don’t see it as being much of an improvement over Bootstrap. And Bootstrap at least provides an open-source component library for free. If you use Tailwind, [they ask you to pay for it](https://tailwindui.com).

### Conclusion: If you like Tailwind, use it! But don’t try to convince me it’s the future. [#](#conclusion-if-you-like-tailwind-use-it-but-dont-try-to-convince-me-its-the-future)

Listen, we can go back and forth on the relative merits or problems with any technology. There are definitely some benefits to choosing Tailwind, most notably how you can go from blank page to fancy-pants design quickly by simply hammering out a bunch of div tags with utility classes.

But after over a year of experience with Tailwind and weighing the pros and cons against other approaches to HTML, styling, and component-based web development in general, I’m thoroughly convinced that Tailwind does not represent the direction I wish to see the web head in as a whole. And apologies to all the Tailwind fans out there, but you just don’t have a compelling argument that will convince me otherwise.

And that’s why Tailwind isn’t for me. YMMV. 🙃