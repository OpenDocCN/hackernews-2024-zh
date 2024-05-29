<!--yml
category: 未分类
date: 2024-05-27 15:05:07
-->

# .well-known/avatar – Terence Eden’s Blog

> 来源：[https://shkspr.mobi/blog/2024/03/well-known-avatar/](https://shkspr.mobi/blog/2024/03/well-known-avatar/)

Hot on the heels of [a post I wrote 4 years ago](https://shkspr.mobi/blog/2020/03/one-avatar-to-rule-them-all/), wouldn't it be useful to have a [well-known URl](https://en.wikipedia.org/wiki/Well-known_URI) for user avatar images?

When I sign up to a web service, I don't want to faff around uploading an image to use as my avatar. I want that service to look at my email address or social-sign-in and automatically pick up my preferred graphic.

Here's how I see it working.

1.  A user signs in to a service with the email address `username@example.com`
2.  In a similar way to [WebFinger](https://www.rfc-editor.org/rfc/rfc7033), the service makes a request to:
    *   `example.com/.well-known/avatar?resource=acct:username@example.com`
3.  If the request's `Accept` header has a MIME type of `image/*`, then the server immediately returns an image.
4.  If the request's `Accept` header has a MIME type of `application/json`, then the server can return a WebFinger-style document with [`"rel":"http://webfinger.net/rel/avatar"`](https://webfinger.net/rel/#avatar) and, perhaps, a list of different images, formats, and sizes.

This makes it incredibly simple for people to use the same avatar *everywhere*.

It also means that if you're designing a service which publicly shows usernames, you can make avatars available without an expensive API call. For example, Twitter could make user's avatars available at:
`twitter.com/.well-known/avatars?resource=acct:edent`

This is a sketch of an idea. I'd like to know if people think it is useful before I take it any further.

I don't think it breaches privacy - a user's image is public on all services anyway.

Users should still be given the option of changing their avatar if they want.

A service shouldn't expose the user's email address - they should proxy the image.

Anything else I should have thought of?

To stave off some common points raised.

*   No this isn't like [Gravatar](https://shkspr.mobi/blog/2020/03/one-avatar-to-rule-them-all/). That works by being a 3rd party service and using the MD5 of your email address.
*   No this isn't like [Libravatar](https://www.libravatar.org/). See above.
*   No this isn't like WebFinger. That only returns JSON.
*   No this isn't like h-card. That requires a server to parse HTML in order to find an image.
*   No this isn't like [BIMI](https://shkspr.mobi/blog/2022/08/dns-esoterica-bimi-svg-in-dns-txt-wtf/). That's expensive and only supports SVG.

* * *