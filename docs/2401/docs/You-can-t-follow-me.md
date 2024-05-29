<!--yml
category: 未分类
date: 2024-05-27 14:42:04
-->

# You can’t follow me

> 来源：[https://so.nwalsh.com/2024/01/06-mastodon](https://so.nwalsh.com/2024/01/06-mastodon)

A few days ago, I stumbled over a couple of weblog posts that talked about adding [Mastodon](https://en.wikipedia.org/wiki/Mastodon_(social_network)) to (mostly) static sites. I thought that sounded kind of“Kind of cool.” Always a bad sign! cool.

So, I poked at things a bit. If I was serious about implementing a full-fleged [ActivityPub](https://en.wikipedia.org/wiki/ActivityPub) client or server, I’d dig in and work out what other implementors use for testing environments, I’d read the specifications, etc.

But this looked like low(er) hanging fruit, and I thought I might get it working with just a bit of fun hacking. I think the problem can be broken down into just a few steps:

1.  Publish a “[WebFinger](https://en.wikipedia.org/wiki/WebFinger)” endpoint (easy).
2.  Support a user, with an inbox and an outbox.
3.  Handle “follow” and “unfollow” messages on the inbox.
4.  Publish posts to followers in the outbox.
5.  I assume that replies get sent back to the inbox as posts, though I can’t actually prove that to myself with a few minutes of casual web searching.

The first hurdle is that all of the communcations have to be HTTPS, which [is inconvenient](/2023/12/31-https) for testing purposes. Then there was a fair bit of effort necessary to get [better isolation](/2024/01/06-isolation) for the test rig.

Once I had that up and running, I could fire up [a Mastodon server](https://github.com/martinheinz/mastodon-local) on my laptop. But could it follow my test weblog?

Pointing my Mastodon instance at `@so@sotest.nwalsh.com` was fine. It hit the webfinger endpoint, hit the user endpoint, and posted something to the inbox. Fantastic! Next, I edited a few things to get some more debugging details about the messages being sent to my endpoints, deployed those changes, went back to the Mastodon instance and tried again.

Nothing happened. I mean, the Mastodon instance was happy to show me the user to follow, but it didn’t actually hit the endpoints again.

I spent *a long time* poking about at the various places in the Mastodon server were caching could be performed. I ran the “cache clear” tasks, I fussed with the `nginx` configuration to turn off proxy caching, I did everything I could think of. To no avail.

I was frustrated, but I discovered that I could make something happen by clicking “Follow” so I dug in there a bit.

Next up: we’ve got all these digital signatures to deal with. There are examples on the web of the sorts of things that need to be done, but all the ones I could find were in [TypeScript](https://en.wikipedia.org/wiki/TypeScript). That’s a hurdle I didn’t feel like trying to overcome today.

And then I realized that to validate the signature, I’d have to hit the endpoint of the user attempting to follow me, e.g., `https://localhost/user/admin`, to get the user’s public key.

Well 🤬! 🤬! 🤬! 🤬!

There’s no way that’s going to work from my `sotest` server on a different host. So to test this, I’d need *both* ends of the communcation to be on the public internet with proper certificates. And I’d have to work out how to disable caching. And I’d have to reverse-engineer some TypeScript (or work out how to use TypeScript).

I gave myself a day to see if I could make progress. I did not. (At least, not the kind of progress I was hoping to make.)

I officially give up. (At least for now.)