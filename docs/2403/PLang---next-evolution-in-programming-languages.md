<!--yml

类别：未分类

日期：2024-05-29 12:37:03

-->

# PLang - 编程语言的下一个演变

> 来源：[https://plang.is](https://plang.is)

**注意：构建代码会花费金钱**

每行代码通常需要通过 LLM 支付 $0.03 - $0.15 的费用。回报？异常的效率提升。

**早期采用者注意：PLang 版本为 0.1。**

为断点变化和错误的存在做好准备。加入我们

[Discord 上的社区](https://discord.gg/A8kYUymsDD)

或通过报告来贡献

[GitHub 上的问题](https://github.com/PLangHQ/plang/issues)

。如果你有意帮助 PLang 的成长，请看看我们如何在我们的贡献

[GitHub 页面](https://github.com/PLangHQ)

.

### 你好 PLang 世界

创建你的第一个应用程序很容易。

1.  打开你最喜爱的文本编辑器。

1.  输入。

`Start - write out "Hello PLang world"`

现在构建并运行使用（你必须

[首先安装 PLang](https://github.com/PLangHQ/plang/blob/main/Documentation/Install.md)

)

plang 执行

*在第一次构建时，将要求你预购积分*

#### 快速开发

你从未见过软件建设如此迅速。

#### 简单开发

只需写下你想要发生的事情。`select newest users, 50 per page`，PLang 将从你的数据库中给你最新的50个用户

#### 开发无废话

关注 **应用程序应该做什么**，而不是 *如何*。

#### 没有配置地狱

没有需要设置的配置。只需运行。如果编程语言需要你的帮助，它会问你。

## 变量

当然还有 %变量% `- 将 %name% 设为 "Dwight Shrewd"`

## 函数

他们被称为 Goals，你可以调用任何 Goal 并发送参数 `- call !FooBar %name%`

## 条件和列表

你写下你的 if 语句，像你想要的那样。像从未有过的方式浏览列表。

`- 如果 %products.Count% > 0 then - go through %products% call !ProcessProduct`

C# 开发者？是的，这是 List.Count 属性

## 重试、缓存和错误处理比以往更简单

这是其中一个 WTF！来看看这个！

`- 从 user 表中选择 name where id=%id% 缓存结果 10 分钟，在错误时在 5 分钟内重试 10 次调用 !NotifyAdmin`

它会缓存返回数据的结果，如果有错误它将重试，并且如果失败了，它会调用一个通知管理员的函数

### 数据库支持

在 Plang 中创建表、视图、触发器。支持 SQLite、SqlServer、MySql、Postresql 或任何 IDbConnection 实现

*在你的目录中创建 Setup.goal 文件，内容如下* `Setup - create table comments, columns text(not null), sentiment(not null), created(datetime, default now)`

### LLM 内置

简单地请求 LLM 评估请求，你可以用写入到一个你可以处理的变量中的响应 `Start - [llm] 系统：用户内容是积极还是消极 用户：%userInput% 方案：{sentiment:positive|negative} 写入到 %sentiment% - insert into comments, text=%userInput%, %sentiment%` 现在运行

plang 执行 userInput="这太棒了"

### Web 服务器

在一行中启动你的 Web 服务器 `Start - start webserver` 现在运行

plang 执行

### 支持 REST

想要与 REST API 进行通信，简单，它为您处理一切。*在您的文件夹中创建 RestTest.goal* `RestTest - get http://https://cat-fact.herokuapp.com/facts write to %json% - write out %json.text%`

现在运行

plang exec RestTest

### 内置消息

你可以简单地在用户之间发送消息。全部加密（AES256）`Start - send message to %publicKey% content: 嘿，你今天好吗，试过 PLang 了吗？`

### 内置爬虫

爬取您喜欢的网站内容，以供以后使用。`Start - go to https://example.org/ - click, "More information..." link - extract .help-article into %helpContent% - write out %helpContent%`

### 内置认证

语言内置认证，因此在与其他 Plang 服务通信时不再需要处理密码和电子邮件。`- get http://api.plang.is/api/balance write to %balance% - write out '我的余额是：%balance%'` 终端服务可以验证请求来自您。无需用户名和密码。用户管理和注册变得更简单和更私密。

### UI - 早期开发

描述您想要的布局 `- center content, horizontal and vertical - create login form, #username, #password, TOS (https://plang.is/tos.html) submit to !Login` 这将提供平台的中立性，包括桌面、移动、手表、电视或任何其他设备。

## 具备超级功能的事件

*您可以将自己的事件添加到任何目标或步骤中* `- before each goal, call !LogGoal - before step nr 2 in !Process.Image, call !Preprocess.Image - on error on !Fetch.Data, call !Report.SysAdmin`

## 查看源码 x 100

*代码是可验证的*

你可以查看所有使用 PLang 构建的源代码。

您可以用它来学习和修改目标。

所有构建文件都是一系列指导 PLang 运行时应如何操作的 JSON 文件。

## 测试您的代码

只需编写您想要测试的内容

## 其他一些功能

### 运行 Python 脚本

无缝执行 Python 脚本。`- run processImage.py %imagePath%`

### IDE 支持

下载 VS Code 扩展以帮助开发

[立即下载](https://marketplace.visualstudio.com/items?itemName=PlangHQ.plang-extension)

### 简化缓存

高效的数据缓存机制。需要 Redis 或其他，都不是问题。

### 操作系统独立性

我需要[这个的帮助](https://github.com/PLangHQ)，但底层代码是 c#，完全操作系统无关。

### 构建事件

这对于分析、调试等非常强大。

### 依赖注入

想要使用自己的记录器？`@logger=MyLogger.Logger` 或 llm、缓存、设置、加密等。

### 定时任务

如此简单，简直疯了 `- every other monday, at 11:23, call !ProcessUsers` or sleep `- wait for 2 sec`

### 压缩

需要处理 zip 文件 `- unzip %filepath% to %newPath%` `- compress %filepath% to %compressedPath%`

### 运行命令行应用程序

运行任何终端/命令行应用程序 `- convert video.mp4 to audio.mp3 using ffmpeg`

### 隐私控制

只有目标可以访问它们运行的路径。如果它们需要外部访问，将请求权限

### 应用程序可扩展性

注入您自己的日志记录器、设置、LLM 服务、数据库、缓存、加密模块。它们非常容易编写。

### 坚实的基础

Plang语言是用C#编写的，它在C#运行时上构建和运行。安全、快速且坚实的基础

## 我如何购买？

PLang是开源的，因此软件本身是免费的。

将您的请求翻译成代码使用一个LLM服务，这项服务是不免费的。

当您第一次运行 'plang build' 时，系统会要求您购买您选择金额的代金券。

您还可以使用OpenAI API服务而无需从PLang购买代金券。

然后适用OpenAI价格，而不通过PLang服务

购买服务的代金券支持PLang的开发，请支持这个项目 :)

## 价格表

### 构建代码

输入 $0.02 / 1K 令牌

输出 $0.06 / 1K 令牌

当您运行 'plang build' 时。

### 运行时的LLM请求

gpt-4-turbo（默认）

输入 $0.02 / 1K 令牌

输出 $0.06 / 1K 令牌

当您运行 'plang run' 并使用 [LLM] 模块时

### 运行时的LLM请求

gpt-3.5

输入 $0.0030 / 1K 令牌

输出 $0.004 / 1K 令牌

当您运行 'plang run' 并使用 [LLM] 模块以gpt-4-turbo参数运行时
