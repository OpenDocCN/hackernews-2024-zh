<!--yml
category: 未分类
date: 2024-05-27 14:44:34
-->

# Exposed RSS – Chris Coyier

> 来源：[https://chriscoyier.net/2024/01/13/exposed-rss/](https://chriscoyier.net/2024/01/13/exposed-rss/)

January 13, 2024

# Exposed RSS

I get sites not having an “RSS” for “Feed” link on their website while actually having an RSS feed. I don’t like it, but I get it. Maybe they picked an off-the-shelf theme that doesn’t have that. Maybe they just forgot. Maybe they even don’t want to.

I somehow never considered sites that have an RSS feed that don’t expose it in the HTML. [As Robb Knight says](https://rknight.me/blog/please-expose-your-rss/):

> This is called RSS auto-discovery and is a standard way to expose RSS feeds to help [browsers and other software to automatically find a site’s RSS feed](https://www.rssboard.org/rss-autodiscovery).
> 
> Like the standard link, a lot of sites were also missing this. This is (at least as a first step) what feed reeders like [NetNewsWire](https://netnewswire.com/) will use to automatically find a feed when you paste in a URL. If you have an RSS feed, you should have the following in the `head` of your website:

```
`<link rel="alternate" type="application/rss+xml" title="My Cool Website" href="https://example.com/feed.xml" />

<link rel="alternate" type="application/atom+xml" title="My Cool Website" href="https://example.com/atom.xml" />`Code language: HTML, XML (xml)
```

I typically don’t even bother looking for a feed link. If I can smell that your site has a feed and I want to subscribe to it, I just chuck your homepage URL into my feed reader “Add” function and let it find what it needs. If it doesn’t find one I’m like *oh well bummer*.

So yeah — if you’re going to bother having a feed, or get one for free, make sure you’ve got that `<link>` tag in place or anybody like me will assume you don’t even have one.

### *Related*

🤘