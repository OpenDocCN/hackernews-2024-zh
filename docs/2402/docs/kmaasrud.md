<!--yml
category: 未分类
date: 2024-05-27 14:45:21
-->

# kmaasrud

> 来源：[https://kmaasrud.com/blog/opml-is-underrated.html](https://kmaasrud.com/blog/opml-is-underrated.html)

# OPML is underrated

As a response to the general [enshittification](https://doctorow.medium.com/social-quitting-1ce85b67b456) of major platforms, I would say we are seeing a resurgence of the old web’s ethos, with personal blogs gaining traction and the concept of the [small web](https://neustadt.fr/essays/the-small-web/) on the rise. That might be colored by the digital communities I hang around in (which are mostly dominated by programmers) but it does at least empirically feel like a trend. That brings along with it a new interest in open web standards. Among them is RSS, which both I, and I think a lot others, have increasingly integrated into our digital routines to keep track of posts from people and sources we’re following. RSS aligns perfectly with the movement towards more personalized and controlled content consumption. Unlike the algorithm-driven feeds of <q>other platforms</q> — which often prioritize engagement over relevance or quality — RSS allows me to curate my own information stream. This feels important to me, as it gives me a level of autonomy over the content that shapes my views and knowledge, as opposed to handing that power over to advertisers.

There is an issue, though. RSS feeds can be a bit clunky to manage and keep track of. Their decentralized nature also makes discoverability an issue. Enter [OPML](http://opml.org/spec2.opml), which is an outliner format that is most commonly used to store [a list of feed subscriptions](http://opml.org/spec2.opml#subscriptionLists). I promise you; having a single file that stores all the feeds you’re interested in is a gamechanger, as it makes it significantly easier to organize, migrate, and share those feeds across different platforms and devices. Here’s an example:

```
<?xml version="1.0" encoding="utf-8"?>
<opml version="2.0">
 <head>
 <title>A list of feeds I follow</title>
 </head>
 <body>
 <outline text="My favorite blog" xmlUrl="https://a-cool-blog.tld/blog/feed.xml" type="rss"
 htmlUrl="https://a-cool-blog.tld/blog" description="You can also add a description" />
 <!-- more outline elements with feeds... -->
 </body>
</opml>
```

Each outline item must have the type `rss` (that goes for both RSS and Atom feeds) and must include the `xmlUrl` attribute. Optionally, you can specify some more attributes, like adding a title with `text`, a description with `description` and a link to the blog front page with `htmlUrl` — that added metadata can be very useful. Yes, it is XML-based, which I admit isn’t exactly the easiest format to work with, but it has a few advantages, which we’ll get back to.

With OPML, you don’t need separate applications or services to categorize feeds. Categorization can be achieved within a single OPML file through its outlining capabilities or by managing multiple OPML files, each dedicated to a different category or use-case. It is a very viable workflow to have one OPML file for your YouTube subscriptions, another for your favorite Twitter/X and Mastodon users, one more for news sites, and yet another for personal blogs — the world’s your oyster. However, there aren’t many application that support nested OPML outlines or categorizing based on different files, sadly, but there should be! This is a call to action, developers: Perfect side-project!

Beyond personal convenience, OPML has the potential to better the *ecosystem* of the <q>small web.</q> By not only sharing an RSS feed on your personal website, but also your list of subscribed feeds, we’re effectively creating a recommendation system that is based on concious curation, not statistical metrics. Your OPML file is now called a *[blogroll](https://blogroll.org/what-are-blogrolls/)*, and you officially get to call yourself a **90s web developer**. Jokes aside, I believe the simple fact that there is a known person behind each recommendation is advantageous. Yes, this might promote smaller digital social circles, but I personally think the transparency of a known source is the best way to combat [filter bubbles](https://en.wikipedia.org/wiki/Filter_bubble). That part is a whole sociological discussion in itself, so if you would like to discuss it further, I would love chatting about it on [my mailing list](https://lists.sr.ht/~kmaasrud/inbox).

Now, getting back to the fact that OPML is XML-based; I’d like to highlight an often-overlooked feature of this: The ability to use an XSL stylesheet to display the OPML file rendered through a HTML template when loaded in a browser. With this, you can add a short introduction and guide to the format, making the blogroll more accessible to those unfamiliar with it. It also opens the possibility to showcase each feed with added context or descriptions.

<details><summary>Here is an example XSL stylesheet you can use:</summary>

```
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
 <xsl:template match="/opml">
 <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
 <head>
 <title>
 <xsl:value-of select="head/title"/>
 </title>
 <meta name="viewport" content="width=device-width, initial-scale=1"/>
 <style> /* Insert CSS here */ </style>
 </head>
 <body>
 <p>
 This is a list of blogs and news sources I follow. The page
 is itself an <a href="http://opml.org/">OPML</a> file, which
 means you can copy the link into your RSS reader to
 subscribe to all the feeds listed below.
 </p>
 <ul>
 <xsl:apply-templates select="body/outline"/>
 </ul>
 </body>
 </html>
 </xsl:template>
 <xsl:template match="outline" xmlns="http://www.w3.org/1999/xhtml">
 <xsl:choose>
 <xsl:when test="@type">
 <xsl:choose>
 <xsl:when test="@xmlUrl">
 <li>
 <strong>
 <a href="{@htmlUrl}"><xsl:value-of select="@text"/></a>
 (<a class="feed" href="{@xmlUrl}">feed</a>)
 </strong>
 <xsl:choose>
 <xsl:when test="@description != ''">
 <br/><xsl:value-of select="@description"/>
 </xsl:when>
 </xsl:choose>
 </li>
 </xsl:when>
 </xsl:choose>
 </xsl:when>
 </xsl:choose>
 </xsl:template>
</xsl:stylesheet>
```</details> 

You can link to the stylesheet in your OPML file by adding `<?xml-stylesheet type="text/xsl" href="path/to/stylesheet.xsl"?>` at the top. I actually do this on [my own blogroll](https://kmaasrud.com/blogroll.xml), so check that out if you want some inspiration.

While we’re all getting a bit fed up with the big platforms, OPML is like a breath of fresh air from the old web days. It’s all about making life easier when managing feeds, sharing cool finds, and stumbling upon new stuff. So I encourage you to create your own blogroll, slap it on your website, and share what you’re into. It’s a simple move, but it could spark some real connections and bring back a bit of that community vibe we miss.

### Here are some posts from sites I follow

TL;DR: rustc will use rust-lld by default on x86_64-unknown-linux-gnu on nightly to significantly reduce linking times. Some context Linking time is often a big part of compilation time. When rustc needs to build a binary or a shared library, it will usually …

via [Rust Blog](https://blog.rust-lang.org/) May 17, 2024

Over the past few years I’ve been noticing more and more of a discourse about the “small web” in my online communities. It’s cool to see! I mentioned the concept in an earlier post and thought I might write a few more words about it today. As far as I can tel…

via [PKGW](https://newton.cx/~peter/) May 14, 2024

Today, exactly five years ago, I, Marcel, started working on Music Assistant . What began as a quick script, to sync my playlists so I could switch between streaming providers, grew into a beast on its own. Music Assistant is what I’d like to call a “music…

via [Home Assistant](https://www.home-assistant.io/) May 9, 2024