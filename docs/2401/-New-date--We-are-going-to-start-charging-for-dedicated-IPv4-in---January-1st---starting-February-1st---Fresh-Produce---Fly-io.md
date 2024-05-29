<!--yml
category: 未分类
date: 2024-05-27 14:25:25
-->

# [New date] We are going to start charging for dedicated IPv4 in ~~January 1st~~ starting February 1st - Fresh Produce - Fly.io

> 来源：[https://community.fly.io/t/we-are-going-to-start-charging-for-dedicated-ipv4-in-january-1st/15970](https://community.fly.io/t/we-are-going-to-start-charging-for-dedicated-ipv4-in-january-1st/15970)

If you’re a long-time [Fly.io](http://fly.io/) customer you will notice we first ship stuff and, later on, we add billing to it. Our pricing page has been for a while we charge ~$2 per dedicated IPv4 per month but we never actually did!

Dedicated IPv4 is a scarce resource but fortunately last December we had [Announcement: Shared Anycast IPv4](https://community.fly.io/t/announcement-shared-anycast-ipv4/9384) and that bought us a ton of time until this was an issue and **shared IPv4s are free**!

We promise we would give a good heads up before we charged you and that’s here, **we will only start billing in ~~January 1st~~ February 1st**!

## Do I need to do something for future apps?

New apps have been getting shared IPv4 when first deployed for a while, your new apps should be good to go with no unexpected charges. Also, just creating an app doesn’t assign IPs immediately.

## How NOT to be billed?

I’m glad you asked. All you need to do is remove your dedicated IPv4 and add a shared IPv4\. That takes roughly two commands per app (assuming you only have one dedicated IPv4 per app).

```
$ fly ips release 213.188.197.238 -a YOUR_APP_NAME
Released 213.188.197.238 from YOUR_APP_NAME

$ fly ips allocate-v4 --shared -a YOUR_APP_NAME
VERSION	IP           	TYPE  	REGION
v4     	66.241.125.86	shared	global 
```

**If you have certificates** make sure to update your DNS records! Currently you can only have either a shared or dedicated IPv4 so make sure to have a dedicated IPv6 as a fallback.

**If you you’re using a custom domain** make sure to issue certificates for your shared IP to work properly like this: [New Shared IPv4 Connection Reset by peer - #2 by pavel](https://community.fly.io/t/new-shared-ipv4-connection-reset-by-peer/17567/2)

[Originally from Jerome’s post:](https://community.fly.io/t/we-are-going-to-start-charging-for-dedicated-ipv4-in-january-1st/15970/24) We now allow having both a shared IPv4 and dedicated IPv4\. This change should make it possible to **switch to a shared IPv4 without downtime**.

The produce looks like this:

*   Add shared ipv4 via `flyctl ips allocate-v4 --shared`
*   (Optional: add the shared ipv4 as an A record for your hostname if you don’t use a CNAME to .fly.dev)
*   Confirm your app works via shared IP: `curl -Iv http://<your-hostname> --resolve <your-hostname>:80:<shared ipv4 you got>` (should respond with a 301 redirect)
*   Wait for DNS caches to clear… probably 5 minutes but that varies wildly, this should be the biggest value between your DNS record’s TTL and our `<your-app>.fly.dev` record (300 I believe).
*   Remove the dedicated IPv4 from Fly (and optionally remove from your DNS if you set it manually as an A record)

## How do I know which apps have dedicated IPv4?

We shipped a new improvement to our [IP usage page](https://fly.io/dashboard/lubien/usage/ip) to show which apps have how dedicated IPs. It will show both dedicated IPv6 (which are free) and dedicated IPv4 (the ones that will eventually charge you).

## Can I preview how much this would charge me?

We also shipped a preview in your [invoice page](https://fly.io/dashboard/personal/billing) that tells you how much dedicated IPv4 usage you have so far in the current month. **These will not be added to your invoice** right now but by ~~January 1st~~ Feburary 1st they will be added for Marchs’s invoice. That means yes, you had free dedicated IPV4 usage from January 2022 until the end of ~~November 2023~~ January 2023 and we will not charge those.

* * *

Let us know if there’s any other questions, we’d love to hear them!