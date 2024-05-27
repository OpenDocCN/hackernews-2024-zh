<!--yml
category: 未分类
date: 2024-05-27 13:17:23
-->

# Caching secrets of the HTTP elders, part 1

> 来源：[https://csvbase.com/blog/8](https://csvbase.com/blog/8)

There is no hard limit on table size in csvbase. Within reason, they can be as big as you like. And if you go off and run your own instance you can do whatever you want. Big Data isn't always Better Data, but it's nice to have the option.

That said, wouldn't it be nice to have those tables cached, so that if you read it twice you don't have to fully download it again the second time?

## Cache-control header - with max-age

Perhaps this HTTP server header?:

```
Cache-Control: max-age=86400 
```

The above header says "keep using this file for a day [86400 seconds] before re-downloading it". Great!

Hang on though. If someone changes that table you'd be stuck looking at the old version for up to 24 hours. Being [eventually consistent](https://doi.org/10.1145%2F1435417.1435432) was once very trendy. But it's no longer hip - even Amazon S3 [no longer returns stale data](https://aws.amazon.com/blogs/aws/amazon-s3-update-strong-read-after-write-consistency/). So serving stale data just won't do for csvbase.

## HTTP/1.1 has a lot of caching features

The elden ones [thought deeply about caching](https://doi.org/10.1109/ICDE.2000.839382). DNS [is a cache](https://www.internethalloffame.org/2012/07/23/why-does-net-still-work-christmas-paul-mockapetris/). [Virtual memory](https://www.linuxatemyram.com/) is a cache. And if you've ever wondered why HTTP/1.0 got a patch release (HTTP/1.1) less than a year after it came out: more caching.

HTTP/1.0 had almost nothing in the way of caching. In HTTP/1.1, a whole new top-level section on the subject was added to the standard. This was essential because, back in the day, the repeated re-downloading of unchanged files was a real headache.

This needless re-downloading was especially bad for popular sites, whose pages would be massively re-downloaded over and over - even though they hadn't changed - by many, many people. Enough to start cause problems for the internet as a whole.

One of the *cool new features* in HTTP/1.1 (implemented in Netscape Navigator 4.0, available from all reputable software stockists) is the `ETag` header. It works like this:

The HTTP server generates a unique id for each version of a file. 'Version' in this context means not just revision but also format, so CSV files get different ids from Parquet files even if served from the same URL (csvbase [does this](https://csvbase.com/blog/2)). The HTTP server returns this id in the `ETag` header.

When an HTTP client requests the file again, it provides that same id in the `If-None-Match` header.

If that ETag is up-to-date, the server just sets the status to `304 Not Modified` and returns absolutely nothing. The client then knows that it should use the file it already has.

This is a way to both keep a cache *and* avoid stale data.

## Generating the ETag

How should the ETag be generated? That is up to the server - up to csvbase. csvbase could just md5sum the file and put that in the header. That would be valid and it would work.

Well, except. Except, csvbase is not actually a database of CSV files. The name is a bit of a trick. csvbase doesn't keep the CSV files on some filesystem (or in an [object store](https://calpaterson.com/s3.html) for that matter). Instead, uploaded CSV files are fully parsed and then put into a SQL database. When csvbase returns you a CSV file it's being generated *on the fly*. That's how csvbase returns other formats like Parquet, JSONlines, JSON, etc.

It would be quicker for csvbase not to have to regenerate the CSV file (or Parquet file) each time, just in order to check whether the md5sum of that file matched the `If-None-Match` header.

So instead of hashing the contents of the file, we'll create the ETag by hashing a key:

`(table id, last change time, file format, page number [if any])`

That key represents both the version of the file plus what page is being looked it (the JSON and HTML formats are paginated). And if your table is changed, last change time is different and the ETag changes too. Fab.

## How long to cache for?

At this point we have a caching system that:

1.  Spares the client downloading the file
    *   though they must make a tiny request
2.  Also avoids the server regenerating the file
3.  Prevents any stale results (ie: avoids ["TTL hell"](https://calpaterson.com/ttl-hell.html))

The only question now is how long we tell clients to cache stuff. How long?

The answer is forever. csvbase sets no `max-age`. Clients can cache stuff as long as they like, **providing that they revalidate each time** the use that data.

Here's what csvbase's response looks like

```
HTTP/1.1 content-type: text/csv etag: <some gobbledegook string> cache-control: no-cache, must-revalidate [ then loads of CSV data ] 
```

The operative bit here is the `no-cache` pragma. Despite the misleading name, this tells clients that they can in fact cache this file, *but* that they must revalidate each time they use it. Here's how [MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control#response_directives) describes it (emphasis is mine):

> *"The no-cache response directive indicates that the **response can be stored in caches**, but the response must be validated with the origin server before each reuse, even when the cache is disconnected from the origin server."*
> 
> *"If you want caches to always check for content updates while reusing stored content, no-cache is the directive to use. It does this by requiring caches to revalidate each request with the origin server."*

`must-revalidate` is probably not necessary but the original RFC suggests that maybe caches might choose, in exceptional circumstances, yadda yadda yadda, to actually serve stale data anyway. `must-revalidate` bans that. `must-revalidate`'s name actually makes sense so I probably don't need to quote chapter and verse, but here, anyway, again from MDN:

> *HTTP allows caches to reuse stale responses when they are disconnected from the origin server. must-revalidate is a way to prevent this from happening - either the stored response is revalidated with the origin server or a 504 (Gateway Timeout) response is generated.*

No TTL necessary. And browsers really support this stuff. It all works.

[`csvbase-client`](https://pypi.org/project/csvbase-client/) also supports this. You can test it out this way:

```
import pandas as pd

# read the table
df1 = pd.read_csv("csvbase://calpaterson/opcodes-6502")

# read it again (this came from your local cache)
df2 = pd.read_csv("csvbase://calpaterson/opcodes-6502") 
```

And if you prefer just to use `curl`, here's how to test it:

```
curl \
--etag-compare opcodes-6502-etag.txt \
--etag-save opcodes-6502-etag.txt \
https://csvbase.com/calpaterson/opcodes-6502.parquet
\-O 
```

## What else is possible with ETags?

A bit more. Join me next time when I use ETags as an effective form of concurrency control. Until then:

## See also

I wrote [another article on caching](https://calpaterson.com/ttl-hell.html) a couple of years ago. The subject bit different but the aim is the same: caching but without correctness problems.

I loved Eric's Brewer's [2004 lecture on Inktomi](https://www.youtube.com/watch?v=E91oEn1bnXM). [Inktomi](https://en.wikipedia.org/wiki/Inktomi) was effectively a primordial Google who tried white-labelling instead of running the search engine themselves. They later changed to being a content distribution network and at [around 57 minutes](https://youtu.be/E91oEn1bnXM?si=zt5vj1TPSBOMkKzG&t=3434) Brewer relays the story of the release of the Starr report which vindicated much of the effort put into web caching since HTTP/1.1 was released. He is very emotional throughout as Inktomi had gone spectacularly bust a couple of years before.

One wrinkle I have not discussed is that many caching reverse proxies don't have exactly have spectacular support for cache revalidation. The going advice on Varnish is to use some [quite complicated config](https://developers.cloudflare.com/cache/concepts/default-cache-behavior/). Cloudflare [do the wrong thing with `no-cache`](https://developers.cloudflare.com/cache/concepts/cache-control/#origin-cache-control-behavior) but it's possible to work around it. What can't be worked around, for csvbase, is their [lack of support for `Vary`](https://simonwillison.net/2023/Nov/20/cloudflare-does-not-consider-vary-values-in-caching-decisions/). This is all HTTP/1.1 stuff - from 1997 - and browsers overwhelmingly support it. But reverse proxies don't, for some reason.