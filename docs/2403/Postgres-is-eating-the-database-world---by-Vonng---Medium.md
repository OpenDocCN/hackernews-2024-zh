<!--yml
category: 未分类
date: 2024-05-27 14:59:54
-->

# Postgres is eating the database world | by Vonng | Medium

> 来源：[https://medium.com/@fengruohang/postgres-is-eating-the-database-world-157c204dcfc4](https://medium.com/@fengruohang/postgres-is-eating-the-database-world-157c204dcfc4)

# Game Changer in the DB Arena

**The emergence of PostgreSQL has shifted the paradigms in the database domain**: Teams endeavoring to craft a “new database kernel” now face a formidable trial — how to stand out against the open-source, feature-rich Postgres. What’s their unique value proposition?

Until a revolutionary hardware breakthrough occurs, the advent of practical, new, general-purpose database kernels seems unlikely. No singular database can match the overall prowess of PG, bolstered by all its extensions — not even Oracle, given PG’s ace of being open-source and free ;-)

A niche database product might carve out a space for itself if it can outperform PostgreSQL by an order of magnitude in specific aspects (typically performance). However, it usually doesn’t take long before the PostgreSQL ecosystem spawns open-source extension alternatives. Opting to develop a PG extension rather than a whole new database gives teams a crushing speed advantage in playing catch-up!

Following this logic, the PostgreSQL ecosystem is poised to snowball, accruing advantages and inevitably moving towards a monopoly, mirroring the Linux kernel’s status in server OS within a few years. Developer surveys and database trend reports confirm this trajectory.

[***StackOverflow 2023 Survey: PostgreSQL, the Decathlete***](https://survey.stackoverflow.co/2023/#section-most-popular-technologies-databases)

[***StackOverflow’s Database Trends Over the Past 7 Years***](https://demo.pigsty.cc/d/sf-survey)

PostgreSQL has long been the favorite database in HackerNews & StackOverflow. Many new open-source projects default to PostgreSQL as their primary, if not only, database choice. And many new-gen companies are going All in PostgreSQL.

As “[**Radical Simplicity: Just Use Postgres**](https://www.amazingcto.com/postgres-for-everything/)” says, Simplifying tech stacks, reducing components, accelerating development, lowering risks, and adding more features can be achieved by **“Just Use Postgres.”** Postgres can replace many backend technologies, including MySQL, Kafka, RabbitMQ, ElasticSearch, Mongo, and Redis, effortlessly serving millions of users. **Just Use Postgres** is no longer limited to a few elite teams but becoming a mainstream best practice.

# What Else Can Be Done?

The endgame for the database domain seems predictable. But what can we do, and what should we do?

PostgreSQL is already a near-perfect database kernel for the vast majority of scenarios, making the idea of a kernel “bottleneck” absurd. Forks of PostgreSQL and MySQL that tout kernel modifications as selling points are essentially going nowhere.

This is similar to the situation with the Linux OS kernel today; despite the plethora of Linux distros, everyone opts for the same kernel. Forking the Linux kernel is seen as creating unnecessary difficulties, and the industry frowns upon it.

Accordingly, the main conflict is no longer the database kernel itself but two directions— database **extensions** and **services**! The former pertains to internal extensibility, while the latter relates to external composability. Much like the OS ecosystem, the competitive landscape will concentrate on **database distributions**. In the database domain, only those distributions centered around extensions and services stand a chance for ultimate success.

Kernel remains lukewarm, with MariaDB, the fork of MySQL’s parent, nearing delisting, while AWS, profiting from offering services and extensions on top of the free kernel, thrives. Investment has flowed into numerous PG ecosystem extensions and service distributions: Citus, TimescaleDB, Hydra, PostgresML, ParadeDB, FerretDB, StackGres, Aiven, Neon, Supabase, Tembo, PostgresAI, and our own PG distro — — [Pigsty](https://pigsty.io/).