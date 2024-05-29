<!--yml
category: 未分类
date: 2024-05-27 15:04:54
-->

# Bluesky: An Open Social Web - Bluesky

> 来源：[https://bsky.social/about/blog/02-22-2024-open-social-web](https://bsky.social/about/blog/02-22-2024-open-social-web)

Today, we’re excited to announce that the Bluesky network is federating and opening up in a way that allows you to host your own data. What does this mean?

Your data, such as your posts, likes, and follows, needs to be stored somewhere. With traditional social media, your data is stored by the social media company whose services you've signed up for. If you ever want to stop using that company's services, you can do that—but you would have to leave that social network and lose your existing connections.

It doesn't have to be this way! An alternative model is how the internet itself works. Anyone can put up a website on the internet. You can choose from one of many companies to host your site (or even host it yourself), and you can always change your mind about this later. If you move to another hosting provider, your visitors won't even notice. No matter where your site's data is managed and stored, your visitors can find your site simply by typing the name of the website or by clicking a link.

We think social media should work the same way. When you register on Bluesky, by default we'll suggest that Bluesky will store your data. But if you'd like to let another company store it, or even store it yourself, you can do that. You'll also be able to change your mind at any point, moving your data to another provider without losing any of your existing posts, likes, or follows. From your followers' perspective, your profile is always available at your handle—no matter where your information is actually stored, or how many times it has been moved.

Federation lets services be interconnected, so there are a variety of apps and experiences that users can move between as fluidly as they do on the open web. The version of federation that we’re releasing today is intended for self-hosters. There are some guardrails in place to ensure we can keep the network running smoothly for everyone in the ecosystem. After this initial phase, we’ll open up federation to people looking to run larger servers with many users. For a more technical overview of what we’re releasing today and how to participate, check out the [developer blog](https://docs.bsky.app/blog/self-host-federation).

Some of our existing features already follow the federated philosophy, including [usernames](https://bsky.social/about/blog/3-6-2023-domain-names-as-handles-in-bluesky) and [feeds](https://bsky.social/about/blog/7-27-2023-custom-feeds). Today, we’re opening up federation for data hosting. Below, we wanted to answer some common questions about what federated hosting is, what it means for your experience using Bluesky, and why we’re so excited about it.

## How does this affect my experience on Bluesky?

The short answer is: it doesn’t! If you don’t run your own server, Bluesky will stay the same. Even if you do run your own server, you may be surprised by how little things change. In fact, it should feel so similar that you might have to double check that you’ve logged into the right server.

This is all intentional: we've made self-hosting your own data both easy and affordable, so when you do so, you should be able to get just as good (or better) of an experience than letting Bluesky host your data.

## If my experience doesn’t change, why federate?

We set out to build a protocol for the future of social media that returns control to users. From the apps you use, the feeds you browse, or the moderation preferences you prefer, your experience on Bluesky is yours to customize. But the abundance of choice and innovation here will be short-lived if it’s dependent on one company. We think that the future of social media needs to be a public good, mutually owned by the people and companies that participate in it. Social media should be as open and reliable as the internet itself.

The ability to host your own data, just as you might run your own website, provides the fundamental guarantee that social media will never again be controlled by only one company. Even if Bluesky were to disappear, if the data is hosted across different sites, the network can be rebuilt. The fact that it requires no permission to set up a new website is what has made the open web such a dynamic and creative force. Larger services, like search engines, have come and gone (anyone remember [Ask Jeeves](https://en.wikipedia.org/wiki/Ask.com)?). But the underlying foundation of independently hosted sites continues to let the web evolve. Making social open in the same way will create a foundation for better public conversations.

Today, we’re taking another step towards the vision of a self-sustaining social web — social media that isn’t controlled by any single company and is free to evolve on its own terms.

## Does this mean Bluesky is going to be like Mastodon?

Mastodon is another federated social network built on a protocol called ActivityPub. While Bluesky — built on a protocol called the AT Protocol (atproto) — shares the term “federation” with other networks, the way it works is very different.

On Bluesky, server choice doesn’t affect what content you see. Servers are only one piece of the protocol — when you browse Bluesky, you see posts that are pulled together from many different servers. This is why you can change your server after signing up without losing your username, friends, or posts.

A summary of some ways Bluesky differs from Mastodon:

*   **A focus on the global conversation:** On Mastodon, your “instance”, or server, determines your community, so your experience depends on which server you join. An instance can send and receive posts from other instances, but it doesn’t try to offer a global view of the network. Your Mastodon server is part of your username, and becomes part of your identity. On Bluesky, your experience is based on what feeds and accounts you follow, and you can always participate in the global conversation (e.g. breaking news, viral posts, and algorithmic feeds). You can use your own domain name as your username, and continue participating from anywhere your account is hosted.
*   **Composable moderation:** Moderation on Bluesky is not tied to your server, like it is on Mastodon. Defederation, a way of addressing moderation issues in Mastodon by disconnecting servers, is not as relevant on Bluesky because there are other layers to the system. Server operators can set rules for what content they will host, but tools like blocklists and moderation services are what help communities self-organize around moderation preferences. We’ve already integrated block and mute lists, and the tooling for independent moderation services is coming soon.
*   **Composable feeds:** We designed your timeline on Bluesky so that it’s not tied to your server. Anyone can build a feed, and there are currently over 40,000 algorithmic feeds to choose from. Your Mastodon timeline is only made up of posts from accounts you follow, and does not pull together posts from the whole network like Bluesky’s custom feeds.
*   **Account portability:** We designed federated hosting on Bluesky so that you can move servers easily. Moving hosting services should be like changing your cell phone provider — you should be able to keep your identity and data. Changing servers on Bluesky doesn’t disrupt your username, friends, or posts.

## So how can I self-host and join the network?

It will become easier to host your own server over time, but at the moment you’ll need a bit of technical know-how to get up and running. If you’re excited to jump in, checkout the [developer blog](https://docs.bsky.app/blog/self-host-federation), the [PDS repo](https://github.com/bluesky-social/pds) on our Github, and the [PDS Administrators Discord](https://discord.gg/UWS6FFdhMe).