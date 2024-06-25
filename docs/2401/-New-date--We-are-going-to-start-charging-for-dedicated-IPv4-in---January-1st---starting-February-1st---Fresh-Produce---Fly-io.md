<!--yml

类别：未分类

日期：2024-05-27 14:25:25

-->

# [新日期] 我们将从**2 月 1 日**开始收取专用 IPv4 的费用 - 新鲜农产品 - Fly.io

> 来源：[`community.fly.io/t/we-are-going-to-start-charging-for-dedicated-ipv4-in-january-1st/15970`](https://community.fly.io/t/we-are-going-to-start-charging-for-dedicated-ipv4-in-january-1st/15970)

如果你是 [Fly.io](http://fly.io/) 的老客户，你会注意到我们首先发布功能，然后再为其添加计费。我们的定价页面已经有一段时间了，我们每个专用 IPv4 每月收费约 2 美元，但我们从未真正实施过！

专用 IPv4 是一种稀缺资源，但幸运的是，去年 12 月我们有了 [公告：共享 Anycast IPv4](https://community.fly.io/t/announcement-shared-anycast-ipv4/9384) ，这给我们带来了大量时间，直到这成为问题，**共享 IPv4 是免费的**！

我们承诺在向您收费之前会提前通知您，这就是现在，**我们只会在** **2 月 1 日** **开始计费**！

## 未来应用需要做些什么？

新的应用程序在首次部署时已经获取了共享 IPv4，你的新应用程序应该可以正常运行，没有意外的费用。另外，只创建应用程序并不会立即分配 IP 地址。

## 如何避免被计费？

很高兴你问。你所需做的只是删除你的专用 IPv4，并添加一个共享 IPv4。每个应用程序大约需要两个命令（假设每个应用程序只有一个专用 IPv4）。

```
$ fly ips release 213.188.197.238 -a YOUR_APP_NAME
Released 213.188.197.238 from YOUR_APP_NAME

$ fly ips allocate-v4 --shared -a YOUR_APP_NAME
VERSION	IP           	TYPE  	REGION
v4     	66.241.125.86	shared	global 
```

**如果你有证书**，请确保更新你的 DNS 记录！目前你只能拥有共享或专用 IPv4，所以确保有一个专用 IPv6 作为备用。

**如果你正在使用自定义域名**，请确保为你的共享 IP 发行证书，以确保像这样正常工作：[New Shared IPv4 Connection Reset by peer - #2 by pavel](https://community.fly.io/t/new-shared-ipv4-connection-reset-by-peer/17567/2)

[来源于 Jerome 的帖子：](https://community.fly.io/t/we-are-going-to-start-charging-for-dedicated-ipv4-in-january-1st/15970/24)我们现在允许同时拥有共享 IPv4 和专用 IPv4。这一变化应该使得**切换到共享 IPv4 无需停机**成为可能。

产品如下：

+   通过 `flyctl ips allocate-v4 --shared` 添加共享 ipv4

+   （可选：如果您没有使用 CNAME 到 .fly.dev，则将共享 ipv4 添加为主机名的 A 记录）

+   通过共享 IP 确认你的应用程序是否正常工作：`curl -Iv http://<your-hostname> --resolve <your-hostname>:80:<shared ipv4 you got>`（应该会响应一个 301 重定向）

+   等待 DNS 缓存清除... 可能需要 5 分钟，但这变化很大，这应该是你的 DNS 记录 TTL 和我们的 `<your-app>.fly.dev` 记录之间的最大值（我相信是 300）。

+   从 Fly 中删除专用 IPv4（如果您手动设置了 A 记录，则可选地从您的 DNS 中删除）

## 我如何知道哪些应用程序拥有专用 IPv4？

我们对我们的[IP 使用页面](https://fly.io/dashboard/lubien/usage/ip)进行了新的改进，以显示哪些应用有多少专用 IP。它将显示专用 IPv6（免费的）和专用 IPv4（最终会向您收费的）。

## 我可以预览一下这会收费多少吗？

我们还在您的[发票页面](https://fly.io/dashboard/personal/billing)上发布了一个预览，告诉您到目前为止本月专用 IPv4 使用量。**目前这些不会添加到您的发票**，但到了~~1 月 1 日~~ 2 月 1 日它们将会在 3 月的发票中添加。这意味着是的，您从 2022 年 1 月至~~2023 年 11 月底~~ 2023 年 1 月底都享有免费的专用 IPV4 使用，我们不会对其收费。

* * *

如果您有任何其他问题，请告诉我们，我们很乐意听到！
