<!--yml
category: 未分类
date: 2024-05-27 14:28:04
-->

# Harmful complexity: using React for static content | Go Make Things

> 来源：[https://gomakethings.com/harmful-complexity-using-react-for-static-content/](https://gomakethings.com/harmful-complexity-using-react-for-static-content/)

# Harmful complexity: using React for static content

Yesterday, I wrote about [the two types of harmful complexity](https://gomakethings.com/the-two-types-of-harmful-complexity-in-a-web-project/) I typically see when working with [consulting clients](https://gomakethings.com/consulting)…

1.  Using over-engineered tooling (like React) for simple, mostly static websites.
2.  Using too little tooling for large, interactive, data-driven UI.

Today, I want to talk about the first type, using over-engineered tooling. Let’s dig in!

## React is bad

I feel very unambiguous about this. React is bad software.

The idea *behind* React is pretty neat. Update some data, and have the UI *react* to it automatically. But React isn’t the only tool to do that, and they didn’t originate the idea, either. They just helped popularize it.

If your site is mostly static content, React is actively harmful to both the people who use it and the developers who have to work on an maintain it.

React is comically, absurdly big. [It’s slow](https://gomakethings.com/react-is-still-absolutely-terrible-for-web-performance/) ([at it’s very core](https://gomakethings.com/react-is-slow-at-its-very-core/), and won’t ever not be). Rendering HTML with JavaScript means your very resilient HTML is now super fragile.

All of this is terrible for users. But React is also horrible for your developer team.

React requires a build step, and that build step is often centered around Webpack. Webpack is very powerful, but also very complicated, and a bit rigid in what it can and can’t do.

Once you’re in the React ecosystem, you’re incentivized to use it for more and more stuff.

That means your HTML, CSS, and JavaScript all end up getting rendered with JavaScript. This is, again, terrible for your users. But it also means that the only developers who can work on your site have to specialize in JavaScript, and often don’t know as much about HTML and CSS.

This results in sites that, frankly, have terrible HTML and CSS, that actively harms end users, is often inaccessible, and becomes increasingly hard to maintain over time.

React is like a black hole. It has gravity, and slowly sucks in your entire site.

## Static sites should be (mostly) static

If your site is mostly content that doesn’t change, or that doesn’t change based on user interactions, it should be served as static HTML files from the server.

That might mean using a static site generator, or it could mean server-side rendering.

In either case, the end result will be a site that’s …

*   Faster and more resilient for your users.
*   Easier to maintain for your developers.
*   Can be worked on by people without JS expertise.

What about stuff on a mostly static site that *is* interactive?

I recommend using *islands* of interaction for that. I personally think [Web Components are the best way to handle that](https://gomakethings.com/the-elevator-pitch-for-web-components/) in 2024, but there are other valid approaches as well.

Tomorrow, we’ll look at the benefits of static site generators versus server-side rendering, and how to choose which approach to use.

And if you have a project that could use the help of an expert, [I have one consulting spot open right now](https://gomakethings.com/consulting). I’d love to work with you!