<!--yml
category: 未分类
date: 2024-05-29 12:48:53
-->

# Why Has Figma Reinvented the Wheel With PostgreSQL? | by Denis Magda | Mar, 2024 | Medium

> 来源：[https://medium.com/@magda7817/why-has-figma-reinveted-the-wheel-with-postgresql-3a1cb2e9297c](https://medium.com/@magda7817/why-has-figma-reinveted-the-wheel-with-postgresql-3a1cb2e9297c)

# Why Has Figma Reinvented the Wheel With PostgreSQL?

A few weeks ago, Figma published an [article](https://www.figma.com/blog/how-figmas-databases-team-lived-to-tell-the-scale/) introducing their own sharding solution for PostgreSQL. The Figma engineering team was aware of, and evaluated, several existing options that could have made their database layer horizontally scalable. But still, they decided to build their own sharding solution from scratch.

In this article, I’ll share my thoughts (and sometimes speculate) on the following:

*   First, why a knowledgeable and experienced Figma engineering team decided to reinvent the wheel instead of selecting one of the existing options from a broad PostgreSQL ecosystem?
*   Second, how does the future look for Figma’s custom solution? Will the team double down on their sharding solution or eventually migrate to one of the existing and more mature options?
*   Lastly, if Figma eventually migrates to another option from the PostgreSQL ecosystem, then what that option will be?

# Quick Solution Overview

I do understand that not everyone has time to read [Figma’s article](https://www.figma.com/blog/how-figmas-databases-team-lived-to-tell-the-scale/) from cover to cover, that’s why I’d like to give a brief overview of their custom solution.

Sharding is a technique that distributes data and load across several standalone database instances. It allows you to split the original dataset into shards, which are then distributed across multiple database instances.

To get sharding working, you need to come up with a sharding key and a hash function. Then, you write the logic that uses the hash function to map a key with an associated record to a shard, and then the shard to a database node.

The Figma team solved this first step easily by selecting the most suitable sharding keys for their dataset and the hash function. Also, they implemented a [data colocation approach](/@magda7817/colocated-and-interleaved-tables-in-distributed-sql-databases-tradeoffs-f2e7b1b9d68e) that allows storing related data together on the same shards/nodes. For instance, all the comments that were left for a specific Figma file are stored together on the same node with the file record.

However, this first step in any sharding implementation is the most straightforward. The most challenging parts come next. Once you’ve distributed your data across several database servers, you need to decide on:

*   How to maintain data integrity by supporting global unique indexes and foreign keys?
*   How to coordinate, commit, and roll back cross-shard/node transactions?
*   How to roll out schema changes atomically across all database nodes?
*   How to reshard and rebalance the data when necessary?
*   How to route the queries from the application layer? Should there be a coordinator, or does this become the responsibility of app developers?
*   How to make the overall solution highly available (with standbys, replicas, failover solutions)?
*   And many other considerations…

The Figma team acknowledged most of these challenges and made efforts to address several of them. For instance, they introduced a coordinator component, named DBProxy, which significantly eases the work for application developers.

The coordinator plays a pivotal role in any sharding solution, provided it’s fully aware of the data distribution across nodes. This component might be referred to by various names, such as coordinator, router, director, or proxy. The client or application connects to the coordinator, which then either routes requests to the specific database nodes or broadcasts them across the entire fleet.

# Why Custom Sharding Solution?

Overall, as an engineer, you will never regret taking part in the development of a sharding solution. It’s a complex engineering problem with many non-trivial tasks to solve. But it’s also a big investment for a company that sponsors the development — someone needs to build and then support/maintain the solution in production.

The Figma team certainly was aware of the trade-offs between the custom vs. existing solution and evaluated several existing options:

> There are many popular open source and managed solutions for horizontally sharded databases that are compatible with Postgres or MySQL. During our evaluation, we explored CockroachDB, TiDB, Spanner, and Vitess.

But still, they decided to go ahead and create their own sharding solution from scratch. Why would they do that?

They justify their custom implementation by saying that:

> However, switching to any of these alternative databases would have required a complex data migration to ensure consistency and reliability across two different database stores… Given our very aggressive growth rate, we had only months of runway remaining. De-risking an entirely new storage layer and completing an end-to-end-migration of our most business-critical use cases would have been extremely risky on the necessary timeline.

In short, it sounds like some of the evaluated options would have worked for them, but they didn’t have enough time to migrate to one of those existing solutions. This does sound reasonable if you need to transition to distributed databases such as TiDB or Spanner. However, this doesn’t explain why [CitusData](https://www.citusdata.com), the default go-to sharding extension for PostgreSQL, wouldn’t work for them. They haven’t even added CitusData to their evaluation list.

However, if you continue reading the article carefully, you’ll notice the team mentioning the following:

> over the past few years, we’ve developed a lot of expertise on how to reliably and performantly run **RDS Postgres** in-house. While migrating, we would have had to rebuild our domain expertise from scratch.

Figma doesn’t use the open-source distribution of PostgreSQL. Instead, they utilize PostgreSQL as a service by subscribing to [Amazon RDS](https://aws.amazon.com/rds/). There’s an interesting, often overlooked fact about PostgreSQL managed services provided by large cloud providers and smaller vendors. While these services usually offer all the core PostgreSQL capabilities, the list of supported extensions is at the mercy of the service provider.

Now, we have CitusData, a mature PostgreSQL extension for sharding, and we know that Figma uses RDS, a fully-managed PostgreSQL service by Amazon. However, if you check [the list of PostgreSQL extensions](https://docs.aws.amazon.com/AmazonRDS/latest/PostgreSQLReleaseNotes/postgresql-extensions.html) supported by RDS, CitusData isn’t included:

So, now, let me speculate. The real reason why Figma reinvented the wheel by creating their own custom solution for sharding might be as straightforward as this — *Figma wanted to stay on RDS, and since Amazon had decided not to support the CitusData extension in the past, the Figma team had no choice but to develop their own sharding solution from scratch.*

# What Does the Future Look Like?

What I truly like about the Figma engineering team is their transparency. They have been very open about the choices and trade-offs they had to confront. I deeply respect their decision and believe that the path they chose was the most sensible at the time.

However, does this imply that the Figma team will continue to use and invest in their own sharding solution? What does the future hold?

The team gives us the following hint:

> Once we’ve bought ourselves sufficient runway, **we will also reassess our original approach** of in-house RDS horizontal sharding. We started this journey 18 months ago with extremely tight timeline pressure. NewSQL stores have continued to evolve and mature. We **will finally have bandwidth to reevaluate the tradeoffs** of continuing down our current path versus switching to an open source or managed solution.

Overall, the future might look as follows:

*   Option #1: Figma continues investing in its own custom solution for sharding and makes it open source once it becomes mature enough. However, they would then need to find a way to differentiate from the CitusData extension.
*   Option #2: Considering that Figma is a customer of Amazon RDS, they might migrate to [Amazon Aurora Limitless](https://aws.amazon.com/blogs/aws/join-the-preview-amazon-aurora-limitless-database/) once this service is blessed for production by Amazon. Aurora Limitless is a sharding solution for Postgres for AWS users. However, they would need to assess the TCO (Total Cost of Ownership) carefully, as many who have evaluated this new service suggest that “Limitless” might also imply limitless costs.
*   Option #3: Figma migrates to [one of the existing solutions from the PostgreSQL ecosystem](https://www.yugabyte.com/postgresql/distributed-postgresql/) that enables horizontal scalability or full distribution. This could be the CitusData extension or even [YugabyteDB](https://www.yugabyte.com), a distributed shared-nothing database built on Postgres. YugabyteDB might be a likely option, considering that the team previously explored TiDB and CockroachDB.

Alright, I hope you enjoyed this “investigation.” Let’s wish the Figma team luck and keep an eye on how the story of their sharding solution evolves!