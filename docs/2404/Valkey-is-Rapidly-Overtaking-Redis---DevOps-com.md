<!--yml
category: 未分类
date: 2024-05-27 13:19:42
-->

# Valkey is Rapidly Overtaking Redis - DevOps.com

> 来源：[https://devops.com/valkey-is-rapidly-overtaking-redis/](https://devops.com/valkey-is-rapidly-overtaking-redis/)

In March, [Redis announced](https://redis.io/blog/redis-adopts-dual-source-available-licensing/) that it was dumping the open source [BSD 3-clause license](https://opensource.org/license/bsd-3-clause) for its [Redis in-memory key-value database](https://devops.com/redis-labs-extends-reach-of-in-memory-database/), instead adopting a “source-available” [Redis Source Available License](https://redis.com/legal/rsalv2-agreement/) (RSALv2) and [Server Side Public License](https://redis.com/legal/server-side-public-license-sspl/) (SSPLv1). That made both [developers and users unhappy](https://www.theregister.com/2024/03/22/redis_changes_license/). So as open source people do, members of the community [forked the code into](https://www.theregister.com/2024/04/12/linux_foundation_opinion/) [Valkey](https://www.theregister.com/2024/04/12/linux_foundation_opinion/) with the support of The Linux Foundation. This move has proved much more popular than expected.

Madelyn Olson, an Amazon Web Services (AWS) software engineer and a Valkey maintainer, announced at the [Open Source Summit](https://events.linuxfoundation.org/open-source-summit-north-america/) in Seattle the availability of the [first release,](https://valkey.io/blog/update/2024/04/valkey-7-2-5-out/) [Valkey](https://valkey.io/blog/update/2024/04/valkey-7-2-5-out/) [7.2.5 GA](https://valkey.io/blog/update/2024/04/valkey-7-2-5-out/). This release is based on the last open source version of Redis, 7.2.4\. That’s good news, but it’s not big news. That’s what we expect from forks.

**Rather, the important news is the immediate support for Valkey.**

AWS, Ericsson, Google Cloud, Oracle and Verizon had all said they were supporting the fork. Now Alibaba Cloud, Aiven, Heroku and Percona are backing Valkey as well. I’ve seen many forks take off and overcome their parent projects – LibreOffice over OpenOffice comes to mind — but I’ve never seen one where so many Fortune 500 businesses spun on their heels to drop one project for another.

Olson said in her keynote speech, “These companies are planning to continue or even increase their contributions. Their support is needed for the long-term health and viability of the project. The community is thankful for their support, which will help keep the project open and keep our developers moving at the pace that our users demand.”

The Valkey fork’s maintainers have the technical chops needed to fulfill those business needs. As Olson pointed out, its technical leadership committee has 26 years  of experience and over 1,000 commits to the open source project. Indeed, Tencent’s [Binbin Zhu](https://lwn.net/Articles/966631/), a committee member, alone contributed almost a quarter of the open source Redis commits.

**This response is not what Redis had in mind.**

At the time of the fork, Redis CEO Rowan Trollope didn’t seem too worried. “The major cloud service providers have all benefited commercially from the Redis open source project, so it’s not surprising that they are launching a fork within a foundation,” he wrote. And he added, “Innovation has been and always will be the differentiating factor between the success of Redis and any alternative solution.”

With so many top maintainers now at Valkey, that assertion may be hard to pull off. As Heroku’s CTO Gail Frederick said during the conference, “With this group of experienced contributors and broad industry backing, Valkey is continuing the open source model that’s been transformative for developers everywhere.”

So many top Redis (the software) customers shifting to Valkey presents Redis (the company) with serious challenges. After all, [Redis was the most popular database on AWS](https://www.theregister.com/2020/11/23/redis_the_most_popular_db_on_aws/) and by DB-Engine’s latest count, [it’s the most popular key-value DBMS](https://db-engines.com/en/ranking).

**It appears that many of those customers are now on their way to Valkey.**

Kyle Davis, an AWS senior developer advocate and former Redis employee, said, “[AWS is committed to supporting open source](https://aws.amazon.com/blogs/opensource/why-aws-supports-valkey/) [Valkey](https://aws.amazon.com/blogs/opensource/why-aws-supports-valkey/) for the long term. We are adding Valkey support to our ElastiCache and MemoryDB managed database services, which are built on open source Redis.”

Karan Singh, Oracle’s Director of Data and AI Product Management, wrote that “Oracle Cloud Infrastructure (OCI) Cache will continue supporting its current Redis 7.0-based service because the license changes don’t impact it.” However, Oracle is dropping the name Redis from its offering and is “[working on integrating](https://blogs.oracle.com/cloud-infrastructure/post/oracle-supports-valkey) [Valkey](https://blogs.oracle.com/cloud-infrastructure/post/oracle-supports-valkey) [support into the OCI Cache service](https://blogs.oracle.com/cloud-infrastructure/post/oracle-supports-valkey) to ensure our unique in-memory database offerings on OCI continue to benefit from open source advancements, enhancing value for our customers.”

Empirical data suggests users are also moving away from Redis. Peter Zaitsev, Percona CEO, noted that [DB-Engine is already showing a drop of interest in Redis](https://twitter.com/PeterZaitsev/status/1781104932302758170%20target=).

Redis’s move attracted a lot of snarkiness. For example, Corey Quinn, chief cloud economist at The Duckbill Group, an AWS analyst firm, tweeted, “Good morning to everyone except [the thieves at Redis](https://twitter.com/QuinnyPig/status/1778484953661194524). You didn’t build it; you stole it from the community. If you want it from the ones who built it, it’s called Valkey.”

What we’re seeing here is not open source fans copping an attitude. It’s companies and developers alike making a major move away from Redis.

*Photo credit: [Happy Lee](https://unsplash.com/@happy_lee?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/a-sign-post-with-many-different-colored-signs-on-it-bOkC_HLLeGs?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)*