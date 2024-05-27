<!--yml
category: 未分类
date: 2024-05-27 13:00:27
-->

# Home · thmsmlr

> 来源：[https://thmsmlr.com/cheap-infra](https://thmsmlr.com/cheap-infra)

One thing is certain in life: if you’re getting something for a steal, someone is getting screwed, and it’s probably you elsewhere. This isn’t an anti-capitalist rant, it’s just the truth. Companies don’t exist if they don’t make money. Unless there’s some innovation, rest assured that you’re overpaying somewhere else.

Last week, I was reminded of this when I saw something on Amazon for cheaper than it was listed at Costco. Costco has the lowest cost structure in the industry. It doesn’t make sense. However, this post isn’t about $3 mosquito repellent and loss leaders. This is a post about AWS and the cloud.

It’s well known that Amazon makes the majority of its profits from AWS, and yet so many people talk about how the cloud is so cheap. That cannot be so. You are getting screwed.

**Here’s some math.**

### From First Principles

Suppose you’re relatively ambitious. You want to create a top 1000 website on the internet, say [businessinsider.com](https://www.businessinsider.com/) for example. According to [SimilarWeb](https://www.similarweb.com/website/businessinsider.com) at the time of writing they are a global top 1000 website (587 to be exact) and get around 200M visitors a month. Each visitor does on average 2 pages per visit, so 400M HTML documents are needed to be served every month. Sampling a few stories on Business Insider it looks like the standard HTML document for a story is on the order of 75KB compressed. Multiply those together and you 30TB of bandwidth just for the HTML.

That sounds like a lot, but consider that we’re not assuming a CDN for the HTML, because your website might not be a news website. Your HTML might be dynamically served for each request. Consider still that Business Insider articles have a bunch of inline javascript that can probably be shipped off to a CDN in your superior implementation. 75KB of compressed HTML is A LOT. Desipte that, 30TB/month is only 11MB/sec, and in Business insider’s case that’s only about 150 request/sec.

Reduce the HTML size, increase the number of requests, either way, with a CDN serving your JS, CSS, and images, you can have a top 1000 website if your app code can produce 11MB/sec of HTML. That’s an incredibly low bar. Even the slowest interpreted language can do 10x that on modern hardware.

The latest AMD server processors have 64 cores and 128 theads. Their newest Zen 5, Turin server processors are rumored to have 192 cores. On a dual socket server you’ll be up near the 400 core count range with 768 threads. **You can serve the world from a single box.** Why do we need Docker, serverless, horizontal scalability again?

You might be thinking, that’s some clever bit of math there, but what about latency? You can’t deny the speed of light, Thomas, the fundamental physics of the universe! That’s what the marketing department of these cloud vendors are going on about these days right? You need to be on the edge, they say. Be close to your users. Minimize latency. It’s true, and we believe in physics in this household, but the specifics are a bit more interesting.

### Edging feels good, but doesn’t deliver the goods

Let’s get a grounding on the lower limits of latency. At the speed of light it would require 200ms to do a roundtrip across the globe. However, in practice this generally ends up being about 300ms to a good data center on the other side of the planet.

Now once again, let’s assume you have your JS, CSS, and media being delivered from a CDN, then **the real question is if you can shave 300ms off your server processing in the initial render, you’ve effectively moved your server across the world next to your user.**

Given this framing, the edge technologies start to not look so good. While yes there has been huge strides in second generation serverless technology to solve the coldboot problem which used to easily eat up that 300ms latency budget, there is still the problem of databases.

No matter how you design your site, SPA, SSR, some hybrid in between you can’t get around that if there is at least one database query involved in rendering your page, you have to go back to your database in us-east-1\. Thereby shifting the latency from user to origin server, to edge server to origin server.

Some might argue that inter-data center connections are faster than user-data center connections, and that may or may not be true, but doesn’t mean you can’t put your website behind something like Cloudfront and get all those benefits without “running on the edge”.

Worse still, most sufficiently complicated pages require 5+ database queries to render. Most web frameworks are single threaded where each of those queries are running in series back and forth to us-east-1 making latency worse than just going there once and having all the database queries be data-center local.

You notice how all of Vercel’s edge demos only display the time? It’s because that’s about the only useful information on the edge.

*Nobody ever thinks about the database.*

A good rule of thumb is that inter-datacenter communication is 10x slower than intra-datacenter communication which is 10x slower than on-device communication. Given those 192 core AMD servers, SQlite is looking pretty nice. Makes up for a good bit of those 300ms.

### And we haven’t even talked about cost

[Hetzner.com](https://www.hetzner.com/cloud). Just look at this site and tell me you aren’t being robbed.

For a 16 core server with 64GB of RAM and an NVM drive, Hetzner charges $0.34/hr. An equivalent x86 server on AWS EC2 is the m5a.4xlarge at $0.68/hr. Not only that, for bandwidth Hetzner gives you 20TB of data for free then charges $1.5/TB there after. AWS only gives you 100GB for free and charges a ridiculous $90/TB.

If you’re using one of the edge cloud vendors, prices are even more ridiculous. Vercel for instance charges $200/TB over the first free TB of bandwidth. These cloud vendors give you tons of stuff for free to lock you in, just to kill you when you scale as if you need it, but as we established before, you probably don’t.

### The reality of the situation

Unless you have some specific usecase for the cloud. Maybe you’re transcoding video, running your own AI models, or doing something that legitimately stresses a system, your website or SaaS can probably run off a single server. It’ll be cheaper and simpler to maintain. Plop one in Virginia and you can get the English speaking world in under 100ms of latency.

Use SQLite locally on the box, you don’t need a managed database. Use [Litestream](https://github.com/benbjohnson/litestream) to continuously back it up. Get a CDN to cache your CSS, JS and images. Server render your page close to your SQLite to minimize back and forths, increasing performance.

Your CI can just SCP your code to the box, NGINX supports zero downtime deploys. Don’t bother with docker and virtualization and all that nonsense, it just slows down your code and slows down your CI/CD.

They say you’ll need it to scale. Most likely you don’t. Tales of horizontal scalability is the most anti-moore’s law thing our industry has ever espoused.

The reality is servers are getting more capable faster than the internet is growing. If you really care about latency, throw a server in Germany, and one in California, route your writes to your primary, use your local read replica for reads.

It’ll scale just fine, be way less complex to manage, and considerably cheaper. 11MB/sec doesn’t have to be so hard.