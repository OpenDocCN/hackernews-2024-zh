<!--yml

类别：未分类

日期：2024-05-29 13:20:59

-->

# Herr Bischoff的屏蔽列表

> 来源：[https://ipbl.herrbischoff.com/](https://ipbl.herrbischoff.com/)

# Herr Bischoff的屏蔽列表

欢迎自由使用以下列表，不受任何限制和任何形式的保证或担保。我会尽力保持更新，但仅此而已。它们在给定间隔内进行更新，因此比这更频繁地获取资源只是在浪费。过多的查询最终会导致您被阻止，但通常不会被添加到列表本身。只需友善和考虑周到，其他一切自然会随之而来。

所有列表都包含IPv4和IPv6地址的CIDR表示法，去掉单个地址的/32和/128后缀。

* * *

## IPBL

它包含那些被捕获在进行不应该进行的请求的IP地址。这包括，除其他信号外：

+   企图积极探测服务器

+   企图利用已知漏洞

+   成为恶意软件引导源

+   进行重复自动登录尝试

+   反复无视robots.txt指令

+   运行未知的爬虫

+   运行脑残扫描脚本

+   进行端口扫描

+   发送垃圾邮件

```
https://ipbl.herrbischoff.com/list.txt (Updated hourly)
```

* * *

## DROP

所有Spamhaus DROP列表汇总为IP地址。对于没有内置ASN到IP范围转换器的脚本或防火墙非常有用，您只需使用它现成的或自行进行过滤。它包含IPv4和IPv6地址，因此您要么原样使用，要么自行过滤。

参见[https://www.spamhaus.org/faq/section/Spamhaus%C2%A0DROP#218](https://www.spamhaus.org/faq/section/Spamhaus%C2%A0DROP#218)获取有关更新的原始来源信息。过多的查询最终会导致您被阻止。只需友善和考虑周到，其他一切自然会随之而来。

```
https://ipbl.herrbischoff.com/drop.txt (Updated daily)
```

* * *

## BADASN

这些IP所属的ASN已经无法挽救。它们存在着滥用流量和内容的显著问题。有些还积极参与联系信息抓取。运营者要么无法管理，要么根本不在乎。很可能，他们有某种激励不去关心，或者他们直接就是垃圾邮件运营者。有些甚至可能是可悲材料和行为的防弹主机提供商。无论如何，它们都需要默认屏蔽，并且我对我负责的基础设施做到了这一点。

```
AS29119     ServiHosting
AS36352     ColoCrossing
AS43097     JSC
AS48666     ASN allocation status is Reserved (Potentially a Bogon)
AS50360     Tamatiya EOOD
AS51088     A2B IP B.V.
AS58110     IP Volume LTD
AS62904     Eonix Corporation
AS64496     ASN allocation status is Reserved (Potentially a Bogon)
AS202425    IP Volume inc
AS202685    Aggros Operations Ltd.
AS203168    Constant MOULIN
AS204007    Kleissner Investments s.r.o.
AS206092    IPXO LIMITED
AS207959    XSServer GmbH
AS209298    Online Marketing Sources Kft.
AS210558    1337 Services GmbH
AS211632    Internet Solutions & Innovations LTD.
AS212370    PEENQ.NL
AS213371    ABC Consultancy (Squitter Networks)
AS397630    Blazing SEO, LLC
AS397702    1776 Solutions
AS400328    Intelligence Hosting LLC
```

这是一个主观的列表。再说一遍，我对垃圾邮件、CSAM、种族主义、至上主义妄想、性别歧视、纳粹主义以及许多其他"主义"都是零容忍的。

```
https://ipbl.herrbischoff.com/badasn.txt (Updated daily)
```

* * *

## TOR

这是已知的Tor节点列表。它由多个来源编制、清理和整合而成。

```
https://ipbl.herrbischoff.com/tor.txt (Updated every 6 hours)
```

* * *

## AI

一份已知由通过机器学习创建LLM（大型语言模型）的公司使用的IP范围列表。他们的抓取通常完全不考虑版权和其他既定规范，实际上是在以你自己的内容卖给你。

```
https://ipbl.herrbischoff.com/ai.txt (Updated sporadically)
```

* * *

## 地理位置信息

这些东西有千千万万的用途，但是批量获取却出奇地难。

+   IPv4和IPv6范围在同一个文件中。

+   编译自替代数据源，旨在提高准确性。

+   去重和合并子网以提高效率。

+   根据国家顶级域名简化和排序。

+   CIDR 表示法。

```
https://ipbl.herrbischoff.com/geoip/ (Updated daily)
```

* * *

## 注释

没有任何列表的去除流程。如果您的 IP 在列表中，请调整服务器的行为，最终它将消失。

这些列表仅通过 HTTPS 提供。服务器支持 TLSv1.2+，请确保您的基础设施是最新的。如果您发现任何问题，请告知：

```
printf "%s@%s.%s\n" marcel herrbischoff com
```

* * *

自 2021 年起运营。
