<!--yml

category: 未分类

date: 2024-05-27 14:35:09

-->

# 通过 DNS 提供有用的实用程序和玩具

> 来源：[https://www.dns.toys/](https://www.dns.toys/)

# 有用的实用程序和服务通过 DNS

dns.toys 是一个 DNS 服务器，它在 DNS 协议中采用创意手段，提供便捷的实用程序和服务，通过命令行轻松访问。

复制并运行下面的命令来尝试它。

## 世界时间

`dig mumbai.time @dns.toys`

`dig newyork.time @dns.toys`

`dig paris/fr.time @dns.toys`

通过不带空格的城市名后缀添加`.time`传递。用斜线隔开的两个字母国家代码是可选的。

### 时区转换

传递 YYYY-MM-DD**T**HH:MM-$fromCity-$toCity（两个字母国家代码是可选的，用斜线隔开）。

`dig 2023-05-28T14:00-mumbai-paris/fr.time @dns.toys`

## 天气

`dig mumbai.weather @dns.toys

`dig newyork.weather @dns.toys`

`dig amsterdam/nl.weather @dns.toys`

通过不带空格的城市名后缀添加`.weather`传递。两个字母的国家代码是可选的。此服务由 [yr.no](https://www.yr.no/en) 提供支持

## 单位转换

`dig 42km-mi.unit @dns.toys`

`dig 32GB-MB.unit @dns.toys`

$Value$FromUnit-$ToUnit。要查看所有 70 个可用单位，请使用 `dig unit @dns.toys`

## 货币转换（外汇）

`dig 100USD-INR.fx @dns.toys

`dig 50CAD-AUD.fx @dns.toys`

$Value$FromCurrency-$ToCurrency。每日汇率来自 [exchangerate.host](https://exchangerate.host)。

## IP 回显

`dig -4 ip @dns.toys`

回显您的 IPv4 地址。

`dig -6 ip @dns.toys`

回显您的 IPv6 地址。

## 数字转单词

`dig 987654321.words @dns.toys`

将数字转换为英文单词。

## 可用的 CIDR 范围

`dig 10.0.0.0/24.cidr @dns.toys`

`dig 2001:db8::/108.cidr @dns.toys`

解析 CIDR 表示法以找出子网中第一个和最后一个可用的 IP 地址。

## 数字进制转换

`dig 100dec-hex.base @dns.toys`

`dig 755oct-bin.base @dns.toys`

将数字从一种基数转换为另一种基数。支持的基数是十六进制、十进制、八进制和二进制。

## 圆周率

`dig pi @dns.toys`

`dig pi -t txt @dns.toys`

`dig pi -t aaaa @dns.toys`

打印圆周率的数字。是的。

## 英语词典

`dig fun.dict @dns.toys`

dig big-time.dict @dns.toys`

获取英文单词的字典定义。由 [WordNet®](https://wordnet.princeton.edu) 提供支持。用连字符替换空格。

## 掷骰子

`dig 1d6.dice @dns.toys

`dig 3d20/2.dice @dns.toys`

要掷的骰子数，后跟`d`，后跟每个骰子的面数，就像在桌面角色扮演游戏中一样。

选择性地加上`/$number`作为总和。例如，一个像 2d20+3 的 DnD 掷骰子可以写作 2d20/3。

## 抛硬币

`dig coin @dns.toys

`dig 2.coin @dns.toys`

要抛的硬币数。

## 随机数生成

`dig 1-100.rand @dns.toys

`dig 30-200.rand @dns.toys`

生成指定范围内的随机数（包括范围值）。

## 纪元/Unix 时间戳转换

`dig 784783800.epoch @dns.toys`

将纪元/Unix 时间戳转换为易读日期。支持 s、ms、µs 和 ns 中的 Unix 时间戳。

## 计算空中距离

`dig A12.9352,77.6245/12.9698,77.7500.aerial @dns.toys`

计算经纬度对之间的空中距离

## 生成UUIDs

`dig 5.uuid @dns.toys`

生成N个UUID（v4）。

## 帮助

`dig help @dns.toys`

列出可用的服务。

# 快捷方式函数

### Bash

将这个bash别名添加到你的`~/.bashrc`文件中。`+`参数会从dig中显示更清晰的输出。

`alias dy="dig +short @dns.toys"`

### 鱼

将此内容添加到你的fish配置文件中。

`alias dy="dig +noall +answer +additional $argv @dns.toys"`

### Zsh

将这个zsh别名添加到你的`~/.zshrc`文件中。`+`参数会从dig中显示更清晰的输出。

`alias dy="dig +short @dns.toys"`

然后，使用dy命令作为快捷方式。

`dy berlin.time`

`dy mumbai.weather`

`dy 100USD-INR.fx`

# 为什么？

为什么不呢？为了好玩。我花了很多时间在终端上，进行快速单位转换、天气检查等，而不必打开笨重的搜索页面很有用。
