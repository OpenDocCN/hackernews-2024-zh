<!--yml

category: 未分类

日期：2024-05-27 13:13:32

-->

# 使用 Tailscale 旅行 | Karan Sharma

> 来源：[https://mrkaran.dev/posts/travel-tailscale/](https://mrkaran.dev/posts/travel-tailscale/)

<main>

我即将前往欧洲进行一次旅行，我非常期待。我希望设置一个 Tailscale 出口节点，以确保我依赖的关键应用程序（如银行门户）在国外继续正常工作。Tailscale 提供了一个名为“出口节点”的功能。这些节点可以设置为路由所有流量（0.0.0.0/0，::/0）通过它们。

我在`BLR`地区部署了一个微型 DigitalOcean droplet，并设置了 Tailscale 作为出口节点。这些步骤非常简单，可以在[这里](https://tailscale.com/kb/1103/exit-nodes)找到。

```
$ echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf $ echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf $ sudo sysctl -p /etc/sysctl.d/99-tailscale.conf $ sudo tailscale up --advertise-exit-node 
```

现在该节点已被宣传为出口节点，我们可以通过`tailscale status`的输出确认：

```
$ sudo tailscale status 100.78.212.33 pop-os               mr-karan@    linux   - 100.75.180.88 homelab              mr-karan@    linux   - 100.100.191.57 iphone               mr-karan@    iOS     offline 100.123.189.14 karans-macbook-pro   mr-karan@    macOS   offline 100.104.67.7 lab                  mr-karan@    linux   offline 100.108.220.87 tailscale-exit       mr-karan@    linux   active; exit node; direct 167.71.236.222:41641, tx 21540 rx 17356 
```

在客户端上，我能够启动 Tailscale 并配置它将所有流量发送到出口节点：

```
sudo tailscale up --exit-node=100.108.220.87 
```

我们可以通过检查此设备的公共 IP 来确认流量是否通过出口节点传输：

```
➜ curl  https://ipinfo.io {
  "ip": "167.x.x.222",
  "city": "Doddaballapura",
  "region": "Karnataka",
  "country": "IN",
  "loc": "13.2257,77.5750",
  "org": "AS14061 DigitalOcean, LLC",
  "postal": "560100",
  "timezone": "Asia/Kolkata",
  "readme": "https://ipinfo.io/missingauth" } 
```

然而，在我需要带上工作笔记本进行应急值班以响应旅行期间可能发生的关键生产事故时，我遇到了一个小问题。在我们的组织中，我们使用 Netbird 作为 VPN，它与 Tailscale 类似，可以在不同设备之间创建点对点的覆盖网络。

问题在于所有 0.0.0.0 流量都被路由到出口节点，这意味着用于 Netbird 访问私有 AWS VPC 网络内部站点的内部流量不再通过 Netbird 接口路由。

当连接到系统时，Netbird 自动传播了一堆 IP 路由规则。这些路由是到我们内部 AWS VPC 基础设施的。例如：

```
10.0.0.0/16 via 100.107.12.215 dev wt0 
```

在这里，`wt0` 是 Netbird 接口。例如，任何像`10.0.1.100`的 IP 都将通过这个接口。为了验证这一点：

```
$ ip route get 10.0.1.100 10.0.1.100 dev wt0 src 100.107.12.215 uid 1000 
```

然而，连接到 Tailscale 出口节点后，情况就不再如此了。现在，即使是本应通过 Netbird 路由的私有 IP，也通过 Tailscale 进行路由：

```
$ ip route get 10.0.1.100 10.0.1.100 dev tailscale0 table 52 src 100.78.212.33 uid 1000 
```

尽管 Tailscale 节点允许选择性地为 CIDR 列表添加白名单，以仅通过它们路由指定的网络数据包，但我的场景有所不同。我需要选择性地绕过某些 CIDR 并通过出口节点路由所有其他流量。我找到了一个相关的[GitHub 问题](https://github.com/tailscale/tailscale/issues/1916)，但不幸的是，由于需求有限，它已经关闭了。

这促使我进一步深入了解 Tailscale 如何传播 IP 路由，看看是否有办法添加优先级更高的自定义路由。

起初，我检查了 Tailscale 的 IP 路由。通常，可以使用`ip route`查看路由表列表，该命令显示`default`和`main`表中的路由。但是，Tailscale 使用路由表 52 来存储其路由，而不是默认或主要表。

```
$ ip route show table 52   default dev tailscale0 100.75.180.88 dev tailscale0 ... others ... throw 127.0.0.0/8 192.168.29.0/24 dev tailscale0 
```

路由表的几个注意事项：

+   `default dev tailscale0` 是此表的默认路由。不匹配此表中任何其他路由的流量将通过 `tailscale0` 接口发送。这确保了任何未定向到更具体路由的流量都会经过 Tailscale 网络。

+   `throw 127.0.0.0/8`：这是一个特殊路由，告诉系统在抵达此表时丢弃目标为 127.0.0.0/8（本地主机地址）的流量，有效地在其到达本地路由表之前丢弃它。

我们可以使用 `ip rule show` 命令查看这些 IP 规则的优先级是如何评估的：

```
➜ ip rule show 0: from all lookup local 5210: from all fwmark 0x80000/0xff0000 lookup main 5230: from all fwmark 0x80000/0xff0000 lookup default 5250: from all fwmark 0x80000/0xff0000 unreachable 5270: from all lookup 52 32766: from all lookup main 32767: from all lookup default 
```

此命令列出所有当前的策略路由规则，包括它们的优先级（查找 `pref` 或 `priority` 值）。每个规则都与一个优先级相关联，数字较小的优先级较高。

默认情况下，Linux 使用三个主要的路由表：

+   本地（优先级 0）

+   主要（优先级 32766）

+   默认（优先级 32767）

由于 Netbird 已经传播了主路由表中的 IP 路由，我们只需要在 Tailscale 接管之前向 `main` 表添加更高优先级的查找规则。

```
$ sudo ip rule add to 10.0.0.0/16 pref 5000 lookup main 
```

现在，我们的 `ip rule` 看起来是这样的：

```
$ ip rule show 0: from all lookup local 5000: from all to 10.0.0.0/16 lookup main 5210: from all fwmark 0x80000/0xff0000 lookup main 5230: from all fwmark 0x80000/0xff0000 lookup default 5250: from all fwmark 0x80000/0xff0000 unreachable 5270: from all lookup 52 32766: from all lookup main 32767: from all lookup default 
```

要确认目的地为 `10.0.0.0/16` 的数据包是否通过 `wt0` 而不是 `tailscale0` 路由，我们可以使用经典的 `ip route get` 命令：

```
$ ip route get 10.0.1.100 10.0.1.100 dev wt0 src 100.107.12.215 uid 1000 
```

完美！此设置使我们能够通过出口节点路由所有公共流量，只有用于内部 AWS VPC 的内部流量会通过 Netbird VPN 路由。

由于这些规则是临时的，并且我想添加一堆类似的网络路由，我创建了一个小型的 shell 脚本来自动化添加/删除规则的过程：

```
#!/bin/bash   # Function to add IP rules for specified CIDRs add() {
  echo "Adding IP rules..."
  sudo ip rule add to 10.0.0.0/16 pref 5000 lookup main  # ... others ... }   # Function to remove IP rules based on preference numbers remove() {
  echo "Removing IP rules..."
  sudo ip rule del pref 5000  # ... others .... }   # Check the first argument to determine which function to call case $1 in
 add)
  add
 ;; remove)
  remove
 ;;  *)
  echo "Invalid argument: $1"
  echo "Usage: $0 add|remove"  exit 1 ;; esac 
```

完成！

</main>
