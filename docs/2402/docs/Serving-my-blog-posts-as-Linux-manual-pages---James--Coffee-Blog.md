<!--yml
category: 未分类
date: 2024-05-29 13:28:43
-->

# Serving my blog posts as Linux manual pages | James' Coffee Blog

> 来源：[https://jamesg.blog/2024/02/29/linux-manual-pages/](https://jamesg.blog/2024/02/29/linux-manual-pages/)

*Intended audience: You are likely to enjoy this post the most if you are interested in Linux and/or Linux manual pages, or if you enjoy reading about esoteric programming projects.*

Linux computers come with pre-installed manual pages that describe how to use specific commands. These pages are readable by typing `man <command>` into your terminal. For example, you can get the manual for the `tac` command, which prints out a file in from bottom-to-top by using the command `man tac`. Some command line software you install adds manual pages, too.

Linux manual pages are formatted using the `roff` syntax, which you can use to mark up documents. `roff` was the first typesetting command line software for Unix, developed at Bell Labs. Earlier this week, with a spark for building but no particular idea in mind, I started to think about the Linux manual page. Could I serve my blog posts as Linux manual pages? Herein lay an adventure.

TL;DR: You can request a Linux manual page version of a blog post with the following HTTP request:

```
 curl -sL -H "Accept: text/roff" {{ site.root_url }}/2024/02/28/programming-projects/ > post.page && man ./post.page 
```

## Devising the system: Content negotiation

I had an idea for how I wanted this to work in mind. I wanted a user to be able to request a `roff` version of a blog post using content negotiation, part of HTTP that lets you specify ixn what format you want a file. For example, you could request an image with an `Accept: image/png`. This tells a server that, if possible, it should send a PNG file. There are lots of intricacies to content negotiation. You can provide a list of types of content you can accept, and the order in which a server should try to return them

But! That's a rabbit hole for another day. What's important here is that an application can ask a server for content in a specific format using a HTTP header.

With content negotiation, I can route requests if a user sends an `Accept` header. If a user asks for an `text/roff` document, I could return a manual page that can be opened with the `man` command.

## Writing the manual pages

Manual pages use the `roff` syntax, so I would need to have versions of my blog posts in that format. To do this, I updated my site to generate `man` pages for each blog post. The template I used to generate the manual page was as follows:

```
 .TH jamesg.blog 1 "" "jamesg.blog"
.SH TITLE
...
.SH AUTHOR
James' Coffee Blog ({{ site.root_url }})
.SH PUBLISHED
...
.SH POST
...
.SH URL
... 
```

Here, I set a header with my domain name and create five sections: title, author, published date, the post content, and the URL of the post. The raw content is markdown. This doesn't always turn out well in a manual as spacing can sometimes be off. But, markdown was more readable than HTML and resulted in less information loss (i.e. titles having no distinction to paragraphs other than being on their own line) than using plain text.

I now had:

1.  Properly formatted manual pages, and;
2.  The knowledge that content negotiation could allow someone to request a manual page.

Now came the final piece of the puzzle: using content negotiation to facilitate the request for manual pages.

## Requesting a manual page

You can use the following command to request the `roff` format of blog posts on this website:

```
 curl -sL -H "Accept: text/roff" {{ site.root_url }}/2024/02/28/programming-projects/ > post.page 
```

You can then open the result as a Linux manual page:

```
 man ./post.page 
```

Let's talk about how this works!

When a browser makes a request to `{{ site.root_url }}/2024/02/19/personal-website-ideas/`, it asks for the HTML version of the page. In the `curl` command above, the command asks for the `text/roff` version. I added a few lines of text in my NGINX configuration to change how the server responds when `text/roff` is requested for a blog post.

First, I declared a few variables in my `/etc/nginx/nginx.conf` file that let me raise a flag when a specific content type was identified:

```
 map $uri $redirect_suffix {
    ~^/(.*)/$   $1;
    default     "";
}

map $http_accept $redirect_location {
    default "";
    "~^text/roff" 1;
} 
```

You can add multiple different redirect locations, but I only need two: the default, and my custom `text/roff` rule.

In my site NGINX configuration (the file in the `/etc/nginx/sites-enabled` folder), I used the following code to handle requests differently if a `roff` page is requested:

```
 server {
				...
        location / {
          if ($redirect_location = 1) {
              rewrite ^/(.*)/$ /$1.man last;
          }
        	...
        }
} 
```

Here, I say: take a URL, and add `.man` to the end, removing the trailing slash, as long as the `Accept: text/roff` header is set. This tells NGINX to read from the `.man` file instead of the `index.html` file associated with each post on my site.

That is to say you can now read blog posts on this website as a Linux manual page. This was a fun investigation into using content negotiation in NGINX and a reminder of how far we have come with typesetting technology from the command line interfaces to modern-day typesetting software and HTML.

*Thank you to [Todd](https://toddpresta.com/) for providing guidance on setting up my NGINX configuration. Todd's help was sincerely appreciated!*

[Share this post on Hacker News](https://news.ycombinator.com/submitlink?u=/https://jamesg.blog/2024/02/29/linux-manual-pages/&t=Serving%20my%20blog%20posts%20as%20Linux%20manual%20pages).

[Share this post on Lobste.rs](https://lobste.rs/stories/new?url=/https://jamesg.blog/2024/02/29/linux-manual-pages/&title=Serving%20my%20blog%20posts%20as%20Linux%20manual%20pages).