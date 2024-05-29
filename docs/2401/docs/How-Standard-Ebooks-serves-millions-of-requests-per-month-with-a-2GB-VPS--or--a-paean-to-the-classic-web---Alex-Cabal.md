<!--yml
category: 未分类
date: 2024-05-27 14:32:24
-->

# How Standard Ebooks serves millions of requests per month with a 2GB VPS; or, a paean to the classic web - Alex Cabal

> 来源：[https://alexcabal.com/posts/standard-ebooks-and-classic-web-tech](https://alexcabal.com/posts/standard-ebooks-and-classic-web-tech)

# How Standard Ebooks serves millions of requests per month with a 2GB VPS; or, a paean to the classic web

February 11^(th), 2022

[Standard Ebooks](https://standardebooks.org) is a project that takes transcriptions of public domain literature, like the kind typically available at [Project Gutenberg](https://gutenberg.org), and creates beautiful, modern ebooks out of them using a [detailed style manual](https://standardebooks.org/manual). Volunteers from all over the world work to produce these ebooks, and then we release them free of cost and free of copyright restrictions.

Books that are in the US public domain are typically books that were published at least 95 years ago. (At the time of writing, this means published before 1927.) As you can imagine, we spend a lot of our time working on really, really old books. So why not create a website out of really, really old technology?

In 2021 the Standard Ebooks server served about 1.4 million ebook downloads over 15.5 million page views. That comes out to a little over 1.2 million page views per month, and that *excludes requests for our [OPDS](https://en.wikipedia.org/wiki/Open_Publication_Distribution_System) and RSS feeds*, which also number in the millions per month. It’s been on the front page of Hacker News at [least](https://news.ycombinator.com/item?id=25138534) [three](https://news.ycombinator.com/item?id=20594802) [times](https://news.ycombinator.com/item?id=14570035) in the past several years without breaking a sweat. And we did all this with a tiny VPS serving a website that has no Javascript—or even a database!

The amount of traffic we get isn’t *that* crazy. But it’s a demonstration of how inexpensive, classic-web technology can serve a surprising amount of traffic, without relying on any of the technological crutches that so many of us reflexively reach for nowadays when starting new projects.

## What the SE server does

SE runs its entire operations using one single-core VPS with 2GB of RAM. It’s responsible for:

1.  Hosting all of the Git repositories for all of our ebooks, serving as the source of truth for the ebooks we produce. (While we also host all of our ebooks on GitHub, *GitHub is actually a mirror*, not the origin, for our ebooks. We think keeping local control is important, and self-hosting a GitHub-like solution to allow the public to view ebook sources and submit corrections, instead of relying on GitHub, is on our long-term roadmap.)

2.  Running the ebook build process when an ebook is released or updated.

3.  Serving the SE website, including the OPDS and RSS feeds.

The [Standard Ebooks website](https://github.com/standardebooks/web) runs on a classic LAP stack. What’s that, you say? LAP? That’s right: LAMP without the M. *There isn’t even a traditional database behind the SE website.*

## Why PHP?

I’ve been writing PHP for almost twenty years now. It’s a language and paradigm I’m familiar with. As one gets older, one feels less and less excitement for the new tech hotness—after all, when one must drive a nail, there’s only so many different kinds of hammers to get excited about before one stops caring and just reaches for the go-to. So when I want to start a new website, I reach for PHP.

Fortunately, PHP is [by one measure the server-side language of choice for 78% of the web](https://w3techs.com/technologies/details/pl-php), so hitching your wagon to such a popular language means it’ll be easy to find someone to maintain it for at least the next few decades.

Modern PHP isn’t half bad, and it has at least two major benefit over some of its competitors:

1.  Each request is a totally independent request that rebuilds the world. There’s no shared state (unless you want there to be).

2.  It’s an interpreted language, so there’s no code to recompile and no servers to restart when deploying. You make a change in your script, save your file, refresh your browser, and there it is. That makes development and debugging *really fast* and deployment *really easy*.

What many people also don’t know is that PHP is also a fairly good templating language out of the box! While it’s easy to assume that any PHP web project at the bare minimum requires one of the [big](https://laravel.com/) [frameworks](https://symfony.com/), in truth it’s easy to use basic PHP as its own simple templating system. For many basic classic-web websites, like SE’s, that’s often enough to get the job done; and the big benefit is that you’re not stuck with having to learn and *maintain* a huge bells-and-whistles 3rd-party framework in perpetuity.

I think people really underestimate the burden of *maintaining* a 3rd-party framework even after *development* of the website is complete. If your website is alive for even just five short years, how many times will you have to update the framework, even if only for security updates, and keep up with API updates to make sure there are no breaking changes? Once is too much for what’s essentially scaffolding that I never want to see again.

## How the SE server handles everything without falling over

### Serving a web request

Requests start at Apache, which usually starts by rewriting the URL using `mod_rewrite`—for example, rewriting an ebook path into a PHP script with a query string. This is kind of equivalent to the router in a server-side framework, except that it’s done before PHP even enters the picture.

Once Apache decides what PHP script to run, it forwards it to PHP-FPM. [PHP-FPM](https://php-fpm.org/) is the process manager that replaced the ailing and feeble `mod_php` many years ago, and is one of the major reasons why modern PHP websites can be so fast.

Once a request reaches PHP, we start creating output for the browser using a [very, very small file-based templating system](https://github.com/standardebooks/web/blob/master/lib/Template.php). It uses raw PHP instead of a custom templating syntax. As I mentioned earlier, PHP is already a surprisingly good templating language in its own right. Why reinvent the wheel?

### Dynamic content without a database

At this point in the request, we may have to look up some data before we can present it to the browser. This is where one might query a database—but we don’t have a database. So how do we do it?

You may recall that the SE server also hosts all of the Git repositories for all our ebooks. These ebooks are rich in metadata, including title, author, descriptions, word count, series information, and so on. Because our corpus of ebooks is relatively small (about 630 ebooks at the time of writing), *we’re able to fit all of them in memory*: specifically, using PHP’s built-in [APCu object cache](https://www.php.net/manual/en/book.apcu.php).

This cache survives across PHP processes, so we only have to initialize it once when PHP-FPM first starts, and we only have to update it when an ebook is updated. If the PHP process crashes or the server restarts, the website rebuilds the cache from scratch whenever it starts up again. Building this cache does take a few seconds, but since crashes and restarts are rare it’s not a big deal.

You may be thinking that this is a pretty silly approach to a dynamic website. Why not just use a database?

I don’t entirely disagree. But it’s that way because SE started out as—and in many ways still is—a small hobby project, and small hobby projects want to spend time on the meat (ebooks) and not the potatoes (a website).

To build a website, we would have *already* had to parse the XML in our ebooks, if only to insert it into a database for later lookup. If we’re already creating ebook objects in PHP, why not just skip the database, the ORM layer, the glue code, the data model, and all that ancillary stuff, and simply cache the ebook objects themselves, to save development time on this hobby?

It turned out that for our small data set and very basic data processing requirements, this approach is perfectly performant. Of course, we’ll *eventually* reach a point where we’ll need a real database, but we’ll probably reach that point first because we want our website to do more complex things, not because the size of our ebook corpus calls for one.

And in the meantime, we’re still serving millions of page views just fine.

### Javascript not necessary

If we haven’t shocked you with our deeply-questionable design decisions yet, this one might shock the younger developers out there most of all: our website doesn’t use any Javascript. Zero. (Well, with the exception of a link to a `manifest.json` file, because there’s no way around that.)

In terms of content, the SE website is very much a classic-web website. There’s a homepage, a search page with a paginated list of items, a detail view of those items, some very basic forms here and there, and a few ancillary, mostly static pages. And it just so happens that plain HTML with some clever CSS has just about all of the functionality needed by that kind of website:

*   Styling, animations, transitions, and responsive affordances are handled with plain CSS.

*   Forms are plain HTML that send regular old GET and POST requests via a boring Submit button. We don’t need fancy AJAX or loading spinners—when you click Submit, your web browser probably already shows you a loading spinner somewhere.

*   There’s no client-side tracking code and there are no ads.

*   Everything is rendered server-side before it reaches your browser.

In just a few short years, server-side rendering went from the near-universal default to a quaint relic, and now Javascript is an “obvious” necessity in a land where everything is an API to be rendered client-side by a huge 3rd-party Javascript framework. But this new paradigm is *slow*: serving megabytes of Javascript is slow, waiting for an API response is slow, parsing the response client-side and generating a page client-side is slow.

To add insult to injury, more often than not *you’re doing the work twice:* Once to parse, assemble, and output data on the server, and again to parse, assemble, and render data on the client.

And JS frameworks, like PHP frameworks, are yet another 3rd-party dependency for you to maintain indefinitely, even if your website is no longer actively updated. How often has a once-popular JS framework faded away, overtaken by the latest hotness?

Not only that, but it’s far easier to output [accessible web pages](https://developer.mozilla.org/en-US/docs/Learn/Accessibility/HTML) using carefully-chosen, plain HTML at the source, than it is when cobbling together elements after the fact with client-side JS.

JS frameworks do make sense for some larger, *very complex apps*. But so few websites are legitimately *very complex apps*. More often than not, they’re [just another CRUD app](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete)—the kind of document-based, asynchronous model that the classic web was designed and built for.

And while it may seem faster to develop a new project by starting with a JS framework, it may be just as fast to develop a new project starting from a server-side framework for your language of choice.

I’m not an anti-JS zealot. Judicious use of Javascript can most definitely enhance the user experience. But does the whole thing need to be done in React? Or could a few carefully-placed lines of jQuery—or even vanilla JS—do the job? Does your CRUD app *need* to be a single-page JS app delivered over websockets?

For SE—a very classic-web project—eschewing Javascript, rendering server-side, and relying on the classic web’s model of static documents and basic forms for interactivity not only reduces our maintenance burden and greatly simplifies development, but it also speeds up the user experience and improves accessibility to boot.

### Cloud not necessary, either

These days, it’s very popular to start even a basic website on a cloud service like AWS or Azure. But while the company hosting our VPS is technically a cloud service—admittedly, our desire to be self-sufficient ends just before owning bare metal in a rented rack—we don’t rely on any other 3rd-party cloud service like AWS or Azure.

Part of the reason is because just glancing at the [dizzying list of AWS acronyms and initialisms](https://docs.aws.amazon.com/general/latest/gr/glos-chap.html) makes me light-headed and woozy.

But the more serious reason is that SE, as a kind of classic-web project, aspires to be independent, just like the classic web once was. Our big external dependency is currently GitHub, but our (very) long-term goal is to drop that too in favor of a self-hosted Git interface.

Starting on a cloud provider cedes one’s independence because it often leads to vendor lock-in. For example, once you start with AWS and its eldritch coterie of inscrutible services, you start requiring specialized, proprietary knowledge—how do I set permissions on an S3 bucket again?—and it can be pretty tough to dig an established project out of that. What if you wanted to change your hosting provider, or your storage or backup solution?

The big benefit of running a basic Linux box on our own VPS is that everything is just files on a generic, well-understood platform. For the most part, Ubuntu at one host is going to be the same as Ubuntu on another one, and Ubuntu is Linux so jumping to (say) CentOS isn’t a huge leap. These skills are highly transferable, making it much easier to move a project somewhere else, or upgrade a server if our needs outgrow its muscle.

Lots of people also choose a cloud service because their CDNs can improve request speed. While that’s certainly true, SE’s classic-web approach makes that much less important. Modern web apps have grown to be so bloated in asset size, and spend so much time processing JS, that using a CDN to shave a few milliseconds off of fetching a static asset becomes a lot more valuable to them. But since we carefully control the size and amount of static assets we serve, and just send complete pages back to the client instead of relying on JS to render a page, shaving milliseconds all of a sudden seems much less important. And if the website’s a touch slower at geographic locations far from our host, well, we’re serving free ebooks here—it’s not life-or-death.

Cloud providers have their place for a lot of computing tasks, and start making more sense once a project starts reaching a *very* large scale. But for serving a basic website and the attendant infrastructure that that requires—up to a perhaps-surprising amount of traffic—a VPS is a low-cost, simple, and lock-in-free way to go. Very classic-web.

### It’s just files in a filesystem

For most basic websites, the biggest resource consumer besides the database is usually the server-side language assembling the web page to serve. Web servers are very, very fast when serving plain files on a filesystem that don’t require any server-side processing.

With that in mind, we’ve made it so that most of the files the SE server serves are static files residing on a regular filesystem. Our OPDS and RSS feeds are static files that are only re-generated when a book is updated; ebook downloads are plain files in a folder, and so are their read-online versions; and so on.

It can be tempting to serve downloads via a script that does a database lookup, or regenerate a feed for each request. But that’s a lot of work when plain files on a filesystem are so performant. Keeping the website as a bunch of mostly-static files is not only a really fast, very classic way of organizing a website, but it also gives us the benefit of...

### Easy deployment

Because PHP isn’t precompiled and because we have no Javascript or CSS to transpile or minify, deploying the website is as easy as [`rsync`](https://linux.die.net/man/1/rsync).

A single `rsync` invocation pushes our development filesystem into production, creating, updating, and deleting files to make the two filesystems match, and voila—our changes are live.

This paradigm of a website being just a collection of files is becoming more and more unusual as language and frameworks are accumulating increasingly-esoteric compilation processes and package managers. Is my NPM version up to date? Did we write the correct XML scaffolding to compile the right DLLs? Did we transpile the Typescript before minifying it into a Yarn package? Did Bower clone the right library into Jekyll before crombulating the monorepo? Who cares! All we have to do is copy our filesystem over and it Just Works.

### Building ebooks

The SE server is also responsible for building ebooks when they get released or updated. This is done using our [ebook production command line toolset](https://github.com/standardebooks/tools). Every time an ebook repository is updated, a Git hook kicks off a rebuild of the ebook so that the updated version can be served on the website.

Building an ebook can be fairly slow and resource-heavy for several reasons, and it’s even slower on our low-resource VPS. The VPS can comfortably build one ebook at a time while also serving the website, but more than say, three, ebooks building at once can cause the server to gasp for air and claw frantically at its chest.

To stop this from happening, builds are queued using [Task Spooler, or `ts`](https://manpages.ubuntu.com/manpages/xenial/man1/tsp.1.html), a program that manages a queue of Bash scripts, executes them sequentially, and records their output. `ts` is a small, simple, Unix-y way to make sure that frequent updates to a series of ebooks don’t overtax our limited resouces with a bunch of build tasks all at once, while at the same time avoiding the overkill of a much larger, all-in-one solution like Beanstalk.

### Side note: XHTML

One quirk of the SE website unrelated to performance is that it serves XHTML5.

XHTML is the overlooked sibling of HTML5, and the herald of the fabled semantic web of the mid–early-2000s. It brings with it all of the tedious, groan-inducing baggage of XML, like namespaces and brutal parser errors, but with only minor benefits that go mostly to the rendering engine, and not much else.

It never took off on the web, in part because in a website context so much HTML is generated by templates and libraries that it’s all too easy to introduce a syntax error somewhere along the line; and unlike HTML, where a syntax error would still render *something*, the tiniest syntax error in XHTML means the whole thing gets thrown out by the browser and you get the Yellow Screen of Death.

But even though XHTML isn’t popular in the web, it still lives in on in one very big space: ebooks.

You see, [EPUB](https://en.wikipedia.org/wiki/EPUB), the surprisingly excellent (how often would you describe a standard as “excellent”?) open ebook format used by the vast majority of ebook producers and reading systems, is basically a set of XML/XHTML files bundled up in a zip file. You could unzip an ebook and open its XHTML files in your web browser, and it would look just like a regular web page.

Since our stock in trade is zip files containing XHTML, we figured why not serve the website itself as XHTML too, as a whimsical nod to our ebook roots? So we did.

I wouldn’t really recommend you serve XHTML if you’re starting a new project. It’s unforgiving, with few or no benefits. But for us, it’s a kind of fun experiment, something a project manager wouldn’t let you do in real life—a tribute to our beloved EPUB format, and a raise of the glass to the halcyon dream of the semantic web.

## Why does this all matter?

All of this goes to show that you don’t need a whole lot to build a performant, useful website, capable of serving millions of requests a month, on a tiny server that also handles other resource-intensive tasks.

It’s a fallacy to think that bloated JS frameworks, complex back-end frameworks, a galaxy of package managers, elaborate moving parts like databases or specialized queueing software, and hyper-resourced, expensive cloud instances at proprietary vendors are needed to create a “modern” website. Classic-web techniques can get you serving millions of page views for pennies per month. *And you get speed, accessibility, maintainability, and simplicity for free.*

Next time you’re thinking of adding a JS library to animate an element, ask yourself: Can I do this with plain CSS?

Next time you’re starting a new website with React, ask yourself: What if I just rendered server side?

Next time you’re spinning up a compute instance on a proprietary cloud provider, ask yourself: Would a $5 VPS do?

And next time you’re feeling like serving XHTML5, ask yourself: Hold on a minute, *what*?