<!--yml
category: 未分类
date: 2024-05-29 12:38:53
-->

# Redis switches licenses, acquires Speedb to go beyond its core in-memory database | TechCrunch

> 来源：[https://techcrunch.com/2024/03/21/redis-switches-licenses-acquires-speedb-to-go-beyond-its-core-in-memory-database/](https://techcrunch.com/2024/03/21/redis-switches-licenses-acquires-speedb-to-go-beyond-its-core-in-memory-database/)

[Redis](https://redis.com/), the popular in-memory data store, is switching away from the open source three-clause BSD license. Instead, in a move that is clearly aimed to prevent the large cloud providers from offering free alternatives to Redis’ own hosted services, Redis will now be dual-licensed under the [Redis Source Available License](https://redis.com/legal/rsalv2-agreement/) (RSALv2) and Server Side Public License (SSPLv1). Under this new license, cloud service providers hosting Redis will need to enter into a commercial agreement with Redis. The first company to do so is Microsoft.

In addition, Redis today announced that it has acquired storage engine [Speedb](https://www.speedb.io/) (pronounced “speedy-bee”) to take it beyond the in-memory space. More about that in a bit.

## Redis license changes

In some way, the licensing move is no surprise. We’ve seen other open source companies like MongoDB, Elastic and Confluent make similar moves. Even Redis — when it was still Redis Labs — went through a series of changes in 2018 and 2019 that changed how it licensed its Redis Modules. That’s when the company introduced the first version of its Redis Source Available License.

“We switched for the same reasons, I think, that everything that has come before us has switched, which is protecting our investment that we make in open source,” Redis CEO Rowan Trollope, who joined the company just over a year ago, told me. “Particularly with Speedb, this is a big investment for us as a startup. If we put that in there and the cloud service providers have the ability to quickly just take and ship it to their customers — essentially without paying anything — that’s problematic for us, as you can imagine.”

The company is quite aware of how this may be perceived by the open source community. Rowan Trollope, who joined the company just over a year ago, told me that he briefed quite a few customers about this change and encountered zero controversy. He is also quite aware that these new licenses mean Redis won’t be considered open source, at least according to [the definition](https://opensource.org/osd) of the Open Source Institute. But he did also stress that Redis plans to continue to work out in the open and allow any company to deploy the open source version of Redis.

“I wouldn’t be surprised if Amazon sponsors a fork,” he added. “Microsoft has already licensed Redis. Our doors are open for business for both Google and Amazon to license the software. It’s not that they can’t continue to ship Redis, they just need to have a commercial arrangement with us.”

With this license change, the company is now also consolidating Redis Stack and the Redis Community Edition into a single distribution. Redis Stack launched in 2022 as a cutting-edge distribution that combines some of the most popular modules, a visualization tool and a client SDK. Because of the BSD license, Redis wasn’t able to put its latest innovations into Redis Core, meaning it was missing features like search and query, for example. This move, Trollope argued, will remove complexity for users who previously had to download multiple pieces to get the most out of Redis.

## Acquiring Speedb

In addition to the licensing change, the company also today announced that it has acquired Speedb.

At its core, Speedb is a RocksDB-compatible key-value storage engine, which may seem like an odd acquisition for the in-memory data store Redis. For the longest time, Redis was all-in on in-memory storage, after all. Using RAM was the only way to get to the performance levels the team was looking for at the time. Spinning hard drives simply weren’t fast enough. But today, with NVMe drives and their high transfer rates, there’s a middle ground to be found that combines fast drives with in-memory storage as something akin to a very large cache.

Data volumes are going up, RAM is expensive and modern solid-state drives are comparably cheap. Meanwhile, enterprises are looking to rein in their spending right now, so having this new option allows for new use cases — including in AI — that would otherwise be out of reach for a lot of companies.

One other interesting move is that, over the last year or so, Redis quietly acquired many of the language-specific open source client libraries. These libraries will remain open source, Trollope stressed. He noted that this, too, will remove some confusion for developers and allow Redis to take a more active hand in steering the development of these tools.

Trollope noted that we may see additional acquisitions from Redis in the future. “There are a lot of data companies out there that have not achieved escape velocity,” he said. “Redis and Databricks, I think, are the two larger ones that are sort of on the pre-IPO track. But there are dozens of smaller one-off companies. I think there’s probably going to be a lot of consolidation in the industry. I won’t comment on our specific plans, but there’s a lot of opportunity for [acquisitions].”

Ahead of the recent downturn, Redis was on a clear path to an IPO. Trollope reiterated that the company is still ready to go once the IPO window opens again (maybe with Databricks leading the way).

As for the immediate future of Speedb, Trollope told me that Redis isn’t going to be in the business of selling a storage engine for long, but for the time being, the company will continue to support Speedb’s customers.