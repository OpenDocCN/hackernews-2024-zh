<!--yml
category: 未分类
date: 2024-05-27 14:55:20
-->

# Freenginx: Core Nginx Developer Announces Fork of Popular Web Server - InfoQ

> 来源：[https://www.infoq.com/news/2024/03/freenginx-ngnix-web-server/](https://www.infoq.com/news/2024/03/freenginx-ngnix-web-server/)

Recently, a former employee of F5 and main contributor of the Nginx project announced the fork [Freenginx](http://freenginx.org/) of the popular web server. The new project was started to [address a security dispute](https://mailman.nginx.org/pipermail/nginx-devel/2024-February/K5IC6VYO2PB7N4HRP2FUQIBIBCGP4WAU.html) and wants to be a drop-in replacement of Nginx, run by developers rather than corporate entities. [Maxim Dounin](http://mdounin.ru/), formerly principal software engineer at F5, provides insights into the fork:

> Unfortunately, some new non-technical management at F5 recently decided that they know better how to run open-source projects. In particular, they decided to interfere with the security policy Nginx has used for years, ignoring both the policy and the developers' position.

Originally developed by [Igor Sysoev](http://sysoev.ru/en/) and currently maintained by F5, Nginx is open-source software for web serving, reverse proxying, caching, load balancing, and media streaming. According to the [Web Server Survey](https://www.netcraft.com/blog/january-2024-web-server-survey/), even two decades after its initial launch, Ngnix remains the leading web server serving 23.21% of all sites. In a popular [thread](https://news.ycombinator.com/item?id=39373327) on Hacker News, user *sevg* [points out](https://news.ycombinator.com/item?id=39373804):

> Worth noting that there are only two active "core" devs, Maxim Dounin (the OP) and Roman Arutyunyan. Maxim is the biggest contributor that is still active. Maxim and Roman account for basically 99% of the current development. So this is a pretty impactful fork.

In his announcement on the [nginx-devel mailing list](https://mailman.nginx.org/mailman/listinfo/nginx-devel), Dounin highlights the [initial dispute](https://freenginx.org/pipermail/nginx/2024-February/000007.html) that prompted Nginx to issue a security patch addressing two critical vulnerabilities. He adds:

> I am no longer able to control which changes are made in Nginx within F5, and I no longer see Nginx as a free and open-source project developed and maintained for the public good. As such, starting from today, I will no longer participate in Nginx development as run by F5\. Instead, I am starting an alternative project, which is going to be run by developers, and not corporate.

Freenginx is not the first fork and drop-in replacement for Nginx: [Angie](https://github.com/webserver-llc/angie) was created by other Russian Nginx developers when F5 left Russia in 2020 and it is currently managed by the Russian company [Web Server](https://wbsrv.ru/). Diogo Baeder, lead backend developer at DeepOpinion, [comments](https://www.linkedin.com/posts/activity-7164034851187113984-YnaE/?):

> Nginx is an incredible software and platform, but I was wondering if it wouldn't be time to just bite the bullet and create a more modern solution based on Rust. Having a solution that follows a similar model, "understands" the Nginx configuration language, and reaches similar performance levels, but with the memory safety and wide community adoption of Rust could lead to an amazing new project - perhaps even doing to Nginx what Nginx did to Apache HTTP.

Vincentz Petzholtz, network engineer and architect, is less optimistic and [adds](https://www.linkedin.com/posts/vincentz-berlin_announcing-freenginxorg-activity-7164343754835931136-_GEM?utm_source=share&utm_medium=member_desktop):

> Sometimes a fork is all you can do when a project is taken into a difficult direction. In the end, the users will vote with the adoption and install-base.

The first release version is [Freenginx-1.25.4](https://freenginx.org/en/download.html?utm_source=thenewstack&utm_medium=website&utm_content=inline-mention&utm_campaign=platform) under the same BSD license as Nginx. Dounin has provided access to a read-only [Mercurial repository](http://freenginx.org/hg/nginx), discarding for now the [option to migrate to GitHub](https://mailman.nginx.org/pipermail/nginx-devel/2024-February/YIFSHIYSKDFBYZ2QRA3WF6SRPGIBDBKI.html). The project started a new [developer mailing list](https://freenginx.org/en/support.html).