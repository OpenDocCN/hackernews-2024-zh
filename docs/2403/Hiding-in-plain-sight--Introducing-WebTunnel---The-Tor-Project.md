<!--yml
category: 未分类
date: 2024-05-27 14:51:38
-->

# Hiding in plain sight: Introducing WebTunnel | The Tor Project

> 来源：[https://blog.torproject.org/introducing-webtunnel-evading-censorship-by-hiding-in-plain-sight/](https://blog.torproject.org/introducing-webtunnel-evading-censorship-by-hiding-in-plain-sight/)

Today, March 12th, on the [World Day Against Cyber Censorship](https://en.wikipedia.org/wiki/World_Day_Against_Cyber_Censorship), the Tor Project's Anti-Censorship Team is excited to officially announce the release of WebTunnel, a new type of Tor bridge designed to assist users in heavily censored regions to connect to the Tor network. Available now in the stable version of Tor Browser, WebTunnel joined our collection of censorship circumvention tech developed and maintained by The Tor Project. 

The development of different types of bridges are crucial for making Tor more resilient against censorship and stay ahead of adversaries in the highly dynamic and ever-changing censorship landscape. This is especially true as we're going through the 2024 global election megacycle, [the role of censorship circumvention tech becomes crucial in defending Internet Freedom](https://blog.torproject.org/2024-defend-internet-freedom-during-elections/).

If you've ever considered becoming a Tor bridge operator to help others connect to Tor, now is an excellent time to get started! You can find the requirements and instructions for running a WebTunnel bridge in the [Tor Community portal](https://community.torproject.org/relay/setup/webtunnel/).

## What is WebTunnel and how does it work?

WebTunnel is a censorship-resistant pluggable transport designed to mimic encrypted web traffic (HTTPS) inspired by [HTTPT](https://www.usenix.org/conference/foci20/presentation/frolov). It works by wrapping the payload connection into a WebSocket-like HTTPS connection, appearing to network observers as an ordinary HTTPS (WebSocket) connection. So, for an onlooker without the knowledge of the hidden path, it just looks like a regular HTTP connection to a webpage server giving the impression that the user is simply browsing the web. 

In fact, WebTunnel is so similar to ordinary web traffic that it can coexist with a website on the same network endpoint, meaning the same domain, IP address, and port. This coexistence allows a standard traffic reverse proxy to forward both ordinary web traffic and WebTunnel to their respective application servers. As a result, when someone attempts to visit the website at the shared network address, they will simply perceive the content of that website address and won't notice the existence of a secret bridge (WebTunnel).

## Comparing WebTunnel to obfs4 bridges

WebTunnel can be used as an alternative to obfs4 for most Tor Browser users. While obfs4 and other fully encrypted traffic aim to be entirely distinct and unrecognizable, WebTunnel's approach to mimicking known and typical web traffic makes it more effective in scenarios where there is a protocol allow list and a deny-by-default network environment.

Consider a network traffic censorship mechanism as a coin sorting machine, with coins representing the flowing traffic. Traditionally, such a machine checks if the coin fits a known shape and allows it to pass if it does or discards it if it does not. In the case of fully encrypted, unknown traffic, as demonstrated in the published research [How the Great Firewall of China Detects and Blocks Fully Encrypted Traffic](https://www.usenix.org/conference/usenixsecurity23/presentation/wu-mingshi), which doesn't conform to any specific shape, it would be subject to censorship. In our coin analogy, not only must the coin not fit the shape of any known blocked protocol, it also needs to fit a recognized allowed shape--otherwise, it would be dropped. Obfs4 traffic, being neither a match for any known allowed protocol nor a text protocol, would be rejected. In contrast, WebTunnel traffic resembling HTTPS traffic, a permitted protocol, will pass.

If you want to learn more about bridges, different designs and how they work, [check out our video series.](https://www.youtube.com/watch?v=8mdtSgHWhXY)

## How to use a WebTunnel Bridge? 

### 🌉 Step 1 - Getting a WebTunnel bridge

At the moment, WebTunnel bridges are only distributed via the Tor Project bridges website. We plan to include more distributor methods like Telegram and moat. 

1.  Using your regular web browser, visit the website: [https://bridges.torproject.org/options](https://bridges.torproject.org/options)

2.  In "Advanced Options", select "webtunnel" from the dropdown menu, and click on "Get Bridges".

3.  Solve the captcha.

4.  Copy the bridge line.

### 💻 Step 2 - Download and install Tor Browser for Desktop

Note: WebTunnel bridges will not work on old versions of Tor Browser (12.5.x).

1.  Download and install the latest version of Tor Browser for Desktop.

2.  Open Tor Browser and go to the Connection preferences window (or click on "Configure Connection").

3.  Click on "Add a Bridge Manually" and add the bridge lines provided on Step 1.

4.  Close the bridge dialog and click on "Connect."

5.  Note any issues or unexpected behavior while using WebTunnel.

### 📲 Or Download and install Tor Browser for Android

1.  Download and install the latest version of Tor Browser for Android.

2.  Run Tor Browser and choose the option to configure a bridge.

3.  Select "Provide a Bridge I know" and enter the provided bridge addresses.

4.  Tap "OK" and, if everything works well, it will connect.

### ✍️ Step 3 - Share feedback with us

Your feedback is crucial to help us identify any issues and ensuring the reliability of WebTunnel bridges. For users living in censored regions, we would love to hear how this new bridge's performance compares to other circumvention methods such as obfs4 and Snowflake.

## Thank you to all the volunteers who have contributed to making WebTunnel possible

The more tools we have at our disposal, the better we will be able to target our response, keeping censors at bay and enabling millions of users to access the free and open internet. We first [announced this new bridge type in October 2023 with a call for testers](https://forum.torproject.org/t/call-for-testers-webtunnel-a-new-way-to-bypass-censorship-with-tor-browser/9855) asking Tor users for whom it was safe to use WebTunnel to provide feedback. So many of you sprung into action and we received a lot of feedback, both public and private, that allowed us to make numerous stability improvements to WebTunnel. 

Right now, there are [60 WebTunnel bridges](https://metrics.torproject.org/rs.html#search/transport:webtunnel?fields=transports) hosted all over the world, and [more than 700 daily active users using WebTunnel](https://metrics.torproject.org/userstats-bridge-transport.html?start=2023-12-08&end=2024-03-07&transport=webtunnel) on different platforms. However, while WebTunnel works in regions like China and Russia, it does not currently work in some regions in Iran.

Our goal is to ensure that Tor works for everyone. Amid geopolitical conflicts that put millions of people at risk, the internet has become crucial for us to communicate, to witness and share what is happening around the world, to organize, to defend human rights, and to build solidarity. That is why our community's volunteer contributions are vital. Remember, there are many ways to get engaged: You can run more [bridges](https://community.torproject.org/relay/setup/bridge), [Snowflake proxies](https://snowflake.torproject.org/) and [relays](https://community.torproject.org/relay/) to continue our fight against censorship and for free and open access to the unrestricted internet.