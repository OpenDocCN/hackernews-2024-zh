<!--yml
category: 未分类
date: 2024-05-27 14:40:31
-->

# The web is mostly links and forms | Go Make Things

> 来源：[https://gomakethings.com/the-web-is-mostly-links-and-forms/](https://gomakethings.com/the-web-is-mostly-links-and-forms/)

# The web is mostly links and forms

A few weeks ago, I read an offhand comment somewhere (I cannot for the life of me remember where) that said…

> The web is still mostly just links and forms.

It’s really stuck with me as I here people argue about how you need all of this heavy, buggy tooling to build for the web because “we’re building apps, not websites.”

**Because here’s the thing: a vast majority of web apps *are* mostly just websites.**

Google Docs? A website. Accounting software like QuickBooks? A website. Discord and Slack? Websites. Apple Maps? A website.

Every single core function of these apps could work—and work well!—without a single drop of JavaScript.

Do they work better with JavaScript? Absolutely! Or well… do they? Sometimes? Sometimes!

**My point is, JavaScript can serve to *enhance* what these apps do, but it’s not the *core* of what they do.**

Discord-like apps have existed since before JavaScript did. Back in the 90’s, chat rooms where literally `form` elements that you typed into. You’d submit to the server, the page would update, and your post would be displayed. If you wanted to see new posts, you’d refresh the browser.

It feels really obvious to me that handling a lot of that with JavaScript-driven Ajax and partial UI updates is a much better UX.

But it’s also obvious to me that having an app that works even when the super fragile JS that powers it fails is *also* a better UX than getting a blank white screen.

Most of what we build is links from one page to another, and `form` submissions that send data from the browser to the server.

Over the next few days, I’m going to write a bit more about *how* to implement a setup like that, and *why* its a superior approach for both the people who use your site *and* the developers who build it.

And if you want help doing this sort of thing in your own projects, I [work with organizations big and small](https://gomakethings.com/consulting) and [teach developers how to build a simpler web](https://leanwebclub.com).