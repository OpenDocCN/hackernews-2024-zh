<!--yml
category: 未分类
date: 2024-05-27 13:41:54
-->

# Solene'% : OpenBSD scripts to convert wg-quick VPN files

> 来源：[https://dataswamp.org/~solene/2024-04-27-openbsd-wg-quick-converter.html](https://dataswamp.org/~solene/2024-04-27-openbsd-wg-quick-converter.html)

# 1\. Introduction [§](#_Introduction)

If you use commercial VPN, you may have noticed they all provide WireGuard configurations in the wg-quick format, this is not suitable for an easy use in OpenBSD.

As I currently work a lot for a VPN provider, I often have to play with configurations and I really needed a script to ease my work.

I made a shell script that turns a wg-quick configuration into a hostname.if compatible file, for a full integration into OpenBSD. This is practical if you always want to connect to a given VPN server, not for temporary connections.

[OpenBSD manual pages: hostname.if](https://man.openbsd.org/hostname.if)

[Sourcehut project: wg-quick-to-hostname-if](https://git.sr.ht/~solene/wg-quick-to-hostname-if)

# 2\. Usage [§](#_Usage)

It is really easy to use, download the script and mark it executable, then run it with your wg-quick configuration as a parameter, it will output the hostname.if file to the standard output.

```
wg-quick-to-hostname-if fr-wg-001.conf | doas tee /etc/hostname.wg0 
```

In the generated file, it uses a trick to dynamically figure the current default route which is required to keep a non-vpn route to the VPN gateway.

# 3\. Short VPN sessions [§](#_Short_VPN_sessions)

When I shared my script on mastodon, Carlos Johnson shared their own script which is pretty cool and complementary to mine.

If you prefer to establish a VPN for a limited session, you may want to take a look at his script.

[Carlos Johnson GitHub: file-wg-sh gist](https://gist.github.com/callemo/aea83a8d0e1e09bb0d94ab85dc809675#file-wg-sh)

# 4\. Prevent leaks [§](#_Prevent_leaks)

If you need your WireGuard VPN to be leakproof (= no network traffic should leave the network interface outside the VPN if it's not toward the VPN gateway), you should absolutely do the following:

*   your WireGuard VPN should be on rdomain 0
*   WireGuard VPN should be established on another rdomain
*   use PF to block traffic on the other rdomain that is not toward the VPN gateway
*   use the VPN provider DNS or a no-log public DNS provider

[Older blog post: WireGuard and rdomains](https://dataswamp.org/~solene/2021-10-09-openbsd-wireguard-exit.html)

# 5\. Conclusion [§](#_Conclusion)

OpenBSD's ability to configure WireGuard VPNs with ifconfig has always been an incredible feature, but it was not always fun to convert from wg-quick files. But now, using a commercial VPN got a lot easier thanks to a few piece of shell.