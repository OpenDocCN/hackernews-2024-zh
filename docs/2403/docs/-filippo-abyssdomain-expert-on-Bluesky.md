<!--yml
category: 未分类
date: 2024-05-29 12:47:50
-->

# @filippo.abyssdomain.expert on Bluesky

> 来源：[https://bsky.app/profile/filippo.abyssdomain.expert/post/3kowjkx2njy2b](https://bsky.app/profile/filippo.abyssdomain.expert/post/3kowjkx2njy2b)

This is a heavily interactive web application, and JavaScript is required. Simple HTML interfaces are possible, but that is not what this is.

### Post

Filippo Valsorda

filippo.abyssdomain.expert

did:plc:x2nsupeeo52oznrmplwapppl

I'm watching some folks reverse engineer the xz backdoor, sharing some *preliminary* analysis with permission. The hooked RSA_public_decrypt verifies a signature on the server's host key by a fixed Ed448 key, and then passes a payload to system(). It's RCE, not auth bypass, and gated/unreplayable. [contains quote post or other embedded content]

2024-03-30T17:13:58.474Z