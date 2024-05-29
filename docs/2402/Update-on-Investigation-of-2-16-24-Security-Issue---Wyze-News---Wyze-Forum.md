<!--yml
category: 未分类
date: 2024-05-27 15:00:10
-->

# Update on Investigation of 2/16/24 Security Issue - Wyze News - Wyze Forum

> 来源：[https://forums.wyze.com/t/update-on-investigation-of-2-16-24-security-issue/291002](https://forums.wyze.com/t/update-on-investigation-of-2-16-24-security-issue/291002)

**Email to users who had thumbnails made available to them that were not their own, but their thumbnails were not made available to others**

Wyze Friends,

On Friday morning, we had a service outage that led to a security incident affecting your Wyze account.

The outage originated from our partner AWS and took down Wyze devices for several hours early Friday morning. If you tried to view live cameras or events during that time, you likely weren’t able to. We’re very sorry for the frustration and confusion this caused.

As we worked to bring cameras back online, we experienced a security issue. Some users reported seeing the wrong thumbnails and Event Videos in their Events tab. We immediately removed access to the Events tab and started an investigation.

We can now confirm that as cameras were coming back online, about 13,000 Wyze users received thumbnails from cameras that were not their own and 1,504 users tapped on them. Most taps enlarged the thumbnail, but in some cases it could have caused an Event Video to be viewed.

We’ve identified your Wyze account as one that was able to see and tap on thumbnails that were not yours, but your thumbnails and Event Videos were not seen by anyone else.

The incident was caused by a third-party caching client library that was recently integrated into our system. This client library received unprecedented load conditions caused by devices coming back online all at once. As a result of increased demand, it mixed up device ID and user ID mapping and connected some data to incorrect accounts.

To make sure this doesn’t happen again, we have added a new layer of verification before users are connected to Event Videos. We have also modified our system to bypass caching for checks on user-device relationships until we identify new client libraries that are thoroughly stress tested for extreme events like we experienced on Friday.

We know this is very disappointing news. It does not reflect our commitment to protect customers or mirror the other investments and actions we have taken in recent years to make security a top priority at Wyze. We built a security team, implemented multiple processes, created new dashboards, maintained a bug bounty program, and were undergoing multiple 3rd party audits and penetration testing when this event occurred.

We must do more and be better, and we will. We are so sorry for this incident and are dedicated to rebuilding your trust.

If you have questions about your account, please visit [support.wyze.com](http://support.wyze.com).

Wyze Team