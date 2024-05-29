<!--yml
category: 未分类
date: 2024-05-27 14:42:11
-->

# Decentralized Hacker News | Enindu Alahapperuma

> 来源：[https://enindu.com/blog/decentralized-hacker-news](https://enindu.com/blog/decentralized-hacker-news)

While scrolling through [Hacker News](https://news.ycombinator.com), I randomly got this idea! I'm not talking about only Hacker News, but a combination of Hacker News, [Mastodon](https://joinmastodon.org), and [Reddit](https://www.reddit.com). Do you need it? I don't know. Maybe this idea is nothing but rubbish. However, I have a business plan too. So let's get into the idea.

This system has three main components:

1.  Client
2.  API server
3.  Courier server

## The client

The user is the client. There will be two types of clients for this system:

1.  Website
2.  Mobile application

The website is just a front-end that connects to the API server. We can build the website from any technology. I prefer something like [Vue](https://vuejs.org) or [React](https://react.dev). Anyway, I'm not a front-end developer, so any front-end developer can suggest the technology.

The mobile application is just a front-end as well as the website. We can build it using any technology that a front-end developer suggests. I haven't framed into any technologies for both the website and mobile application yet. This is just a high-level explanation of the system that I'm talking about.

From the client-side, users are allowed to register into the system, log in to their accounts, and post anything like in Reddit. So they can post articles or links. When saying articles, I'm not talking about [microblogging](https://en.wikipedia.org/wiki/Microblogging) but real [blogging](https://en.wikipedia.org/wiki/Blog). This is where Reddit-ness comes into play. In Hacker News, we can't do blogging. I have a reason to post articles in this system. Let's talk about that later.

The client, website, or mobile application is joined to the API server through a [REST API](https://en.wikipedia.org/wiki/REST). This is my idea. If you have other alternatives, suggestions are always welcome. Whatever technology we use here, anyone should have the freedom to create clients on their own. So the API server should be publicly accessible. This means, as a decentralized system, anyone should be able to create their own client for any instance. This should be an open network. That's what was in my mind. If the instance owner needs to make their instance private, they should host it in a private network, where outsiders don't have access.

## The API server

This is the engine of this system. In the API server, there will be three primary components:

1.  API
2.  Database
3.  Cache

Again, the API can be written in any technology. I have [Go](https://go.dev), [Python](https://www.python.org), [Node.js](https://nodejs.org/en), [PHP](https://www.php.net), and [Ruby](https://www.ruby-lang.org) in my mind. Since this is the engine of this system, the API must be able to process lots of data. So, not like in the front-end, the technology will be crucial here. We should use maintainable, solid technology for this.

All the user's data will be stored in the database. We can use separate storages, something like an [S3](https://aws.amazon.com/s3) bucket, for storing files. I don't have any specific technology to use as the database. We can use [PostgreSQL](https://www.postgresql.org), [MariaDB](https://mariadb.org), or even a no-SQL database like [MongoDB](https://www.mongodb.com). This is open to suggestions as well.

We need a cache to optimize the user experience as well as the federation across the instances. When an outer instance sends a request to this system, the cache server will serve the data. The internal database is not open to the outer instances. Maybe we can use something like [Redis](https://redis.io) for this. I don't know.

The API server is connected to the courier server via something like a REST API. And again, the mentioned technology is just an example. We can use whatever we like.

## The courier server

We use the courier server to do the federation among the instances. Mastodon uses the [ActivityPub](https://www.w3.org/TR/activitypub) protocol to do this. We also can use that or we can just use a REST API with a standardized structure. As always, I'm open to any suggestion.

The courier server is a two-way server, which means it sends requests and receives requests at the same time. When an external instance requests data from this instance, the request is sent from the courier server of the external instance. That request was received by the courier server of this instance. After that, the internal requests made within the internal courier server, internal API server, cache, and database. After that, the response will be sent to the external courier server. Only courier servers communicate with each other. Endpoints, which are not available for the front-end, will be only opened to the internal courier server. External courier servers can't communicate directly with other API servers.

We might need to use some kind of authentication system across the internal API server and internal courier server. Since courier servers are externally accessible from other courier servers, we also might need to add an authentication layer for courier servers. Courier servers should not be publicly accessible for anyone but other courier servers. I'm still working on what kind of authentication system we should implement for this. Suggestions are always welcome.

## The business idea

I'm not talking about a commercial system here, but an open-source project. So when I say business, I'm not talking about money.

There are several ways for someone to use this system:

*   We can use this for blogging. We already know about [IndieWeb](https://indieweb.org), [Webmention](https://www.w3.org/TR/webmention), and other stuff. We can do things like Webmention in this system. Each instance will act like an independent blog. Just like Mastodon, but for blogging.
*   We can use this as an internal system of a company. The company has to set up instances for each component. Each instance has a certain set of employees, but due to the decentralized nature, all employees can work together.
*   Hacker News stands for one reason and one reason only. We can use this system to make hundreds or thousands of Hacker News sites for various categories.

I have more use cases for this system. I'm just too lazy to mention them all. By the way, you get the idea. But three things are always speaking in my mind.

1.  Am I just talking about Mastodon? Well, maybe.
2.  What is the difference between this system and Mastodon? I'm not so sure.
3.  Do we need this system? I don't know.

However, we can find the correct answers for them, together.

## Disclaimer

This is a pure idea of mine. I didn't copy it from anywhere. If someone has an idea like this, I don't intend to reveal it because I didn't know that they had an idea like this.