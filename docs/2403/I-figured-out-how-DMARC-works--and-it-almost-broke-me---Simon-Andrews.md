<!--yml

category: 未分类

date: 2024-05-27 14:59:35

-->

# 我弄清楚了DMARC的工作原理，它几乎把我搞垮了 | Simon Andrews

> 来源：[https://simonandrews.ca/articles/how-to-set-up-spf-dkim-dmarc](https://simonandrews.ca/articles/how-to-set-up-spf-dkim-dmarc)

# 我弄清楚了DMARC的工作原理，它几乎把我搞垮了

如何在您的域上防止电子邮件欺诈，使用一种愚蠢的标准组合。

最近，我遇到了一个问题。我的域名没有正确实施SPF、DKIM或DMARC。

然后，我遇到了第二个问题：我完全不知道这些是什么，似乎*没有人*以人类可以理解的方式写过有关SPF、DKIM或DMARC的文章，更不用说实施了。我找到的每篇文章要么技术性很强，试图通过SEO来销售东西，要么水平太高以至于没有用处。

因此，我不得不进行大量的辛勤工作和研究来理解这个问题。希望是因为我必须这样做，您就不必了。

这里有两个主要部分：对这些事物是什么的人类解释，以及实施它们的相对简单的方法。

如果您来到这里，这可能不容易，但这可能并非可选。希望这可以帮助到您。

*随着我的学习，我会不断更新本文，因此它将会变化和增长。如果您想了解自上次访问以来发生了什么变化，请[查看GitHub上的差异](https://github.com/simon360/simonandrews.ca/commits/main/pages/articles/how-to-set-up-spf-dkim-dmarc.mdx)。*

## 目录

## 这些奇怪的首字母缩略词是什么？

SPF、DKIM和DMARC是互补的系统。邮件服务器使用SPF和DKIM作为邮件是否为垃圾邮件的指标。DMARC随后执行两件事情：它告诉邮件服务器SPF和DKIM的重要性，以及当电子邮件未能通过测试时该怎么做。

目前这可能还不太容易理解，但没关系。让我们深入一点。

### SPF

`SPF`是声明谁可以从您的域发送电子邮件的一种方式。它代表"发件人策略框架"，但您不需要知道这一点。只需称其为SPF或"欺骗"。它旨在使发送欺骗邮件变得更加困难。

例如，它是一种表明"来自mycompany.com的电子邮件只能通过Google和Postmark发送"的方式。声明SPF使我更难以从您的域发送电子邮件以试图钓鱼。

这是它的工作方式，对于一个*有效的、非钓鱼*电子邮件：

1.  我从`hello@sadl.io`发送了一封电子邮件给你，使用了我的Fastmail SMTP服务器。

1.  Gmail（您的电子邮件服务）接收到了该电子邮件。

1.  电子邮件来自`sadl.io`的某人，所以Gmail获取了`sadl.io`的DNS记录。

1.  `sadl.io`有一个声明其SPF策略的DNS记录。它声明电子邮件可以从Fastmail发送。

1.  这封电子邮件是从Fastmail发送的，因此它通过了SPF测试。

1.  电子邮件到达您的收件箱。

那太棒了！`SPF`没有阻止我向你发送真实的电子邮件。但它看起来相当简单。那么...它到底会阻止什么呢？

众所周知，电子邮件容易被伪造。尽管我多年没有写PHP了，*没有*比这个PHP脚本更简单地证明了这一点：

```
<?php

$to = "employee@example.com"; 
$headers = "From: payroll@example.com"; 

$subject = "Verify your bank details for your paycheque"; 
$txt = "Hi Simon, we're updating our payroll software, and in order to continue receiving your paycheque, we'll need you to enter your details here: http://nefarious-payroll-software.com.";

mail($to, $subject, $txt, $headers); 
```

此脚本向`employee@example.com`发送一封电子邮件，看起来是来自`payroll@example.com`，并要求员工输入其银行详细信息。这是一封相当有说服力的电子邮件，可能会让一些人至少点击链接 - 或更糟。

你现在*可以*运行这个脚本，如果`example.com`没有设置SPF，这封电子邮件可能真的会达到`employee@example.com`的收件箱。

说真的，事情就是这么简单。我们都在某个时候学过这个道理：电子邮件很容易被伪造。这一直是做生意的成本。"你不能阻止伪造者！电子邮件修复起来太复杂了！"这可能是你听过或者说过的话。我们把它放在脑后。我们知道电子邮件不安全，但我们还是在使用它们。结果证明这并非完全正确。SPF实际上可以帮助减少伪造邮件。

如果设置了SPF，当该脚本运行时会发生以下情况：

1.  一封电子邮件被发送到`employee@example.com`。

1.  这封电子邮件被Google接收，他们运行`example.com`的邮件服务器。

1.  由于它是来自`example.com`的电子邮件，Google获取了`example.com`的DNS记录。

1.  `example.com`声明了一个SPF策略，指出只能从Google发送电子邮件。

1.  这封电子邮件并不是从Google发送的；它是从本地邮件服务器发送的。

1.  由于发送域与`example.com`允许的域名不匹配，该电子邮件被标记为垃圾邮件。*也许*。

等等…… *也许*？是的。SPF在原则上听起来很棒。但如果没有DMARC，它通常没有任何效果。我们会讲到这个，但可以说：如果你刚刚设置了SPF，它大部分是信息性的。一些邮件服务器*可能*会使用它，但它们可能不会给予太多重视。DMARC允许你增加其重要性。

#### 关于信封和信函

这一部分变得更加怪异，但请跟着我走。你可能现在想大致看一下这部分内容，然后阅读剩余的文章，再回来再看。一旦理解DMARC对齐，这部分就会更有意义，但我们还没到那一步。好了，让我们开始。

首先：向正确指出这个SPF描述不准确的Liam致敬。SPF *实际上*检查的是"信封发件人"的有效性，而不是"发件人"。

Liam向我指出了一篇文章，解释了两个事物之间的区别：一封电子邮件（将其视为一封信函）和传输该信函的SMTP交互（一个信封）。这篇文章来自XEAMS，你可以（而且应该！）[在这里阅读](https://www.xeams.com/difference-envelope-header.htm)。但我会用自己的话来概括：

**电子邮件** 是你在客户端看到的东西。它包括消息正文和一堆头部信息。这些头部包括 `From` 头部，在我们那个 PHP 脚本中伪造了，以及电子邮件主题、DKIM 头部（我们马上会讲到！）和一堆其他头部。大多数电子邮件头部被用户的电子邮件客户端软隐藏，但你总是可以查看它们 —— 在你的电子邮件客户端中，无论是应用程序还是网站，你可能会找到一个名为 "查看原始" 或 "查看原始消息" 的菜单项。这将展示给你一个电子邮件的 *真实* 内容。但它不会展示给你信封。

**信封** 是在 SMTP 层面发生的事情。我会参考 XEAMS 文章中 SMTP 的样子，但实际上：*信封* 可能包含与邮件的 `From` 头完全不同的 `MAIL FROM` 邮件地址。

你会听到几个术语用来描述 SMTP `MAIL FROM` 命令：信封发件人、回复地址、退信地址。它们都是同一个东西。我喜欢在口语中称之为 "信封发件人"，因为这个类比很巧妙，但我认为知道它来源于用于传递你的电子邮件的 SMTP 协议命令是很有用的。

这就带我们来了：SPF *实际上并不关心你的电子邮件的 From 头部*。它只关心 `MAIL FROM` 的值。

后来，我们将讨论 DMARC 对齐。但这里有个预告：通过 DMARC 对齐，尽管 DKIM 不考虑，你在说“为了使邮件有效，发件人地址必须通过 SPF，且 From 头中的域必须与发件人地址中的域相同”。实际上，它使得 PHP 伪造示例大致运行得如你所期望的那样，但要经过一些绕圈子的方式才能实现。

我在上述部分留下了这个不准确之处。为什么呢？嗯……解释信封发件人和 From 头之间的差异在这个页面上占用了与 SPF 本身解释一样多的空间。如果我先解释了这个，到真正的例子时你可能已经入迷了。抱歉误导了你 —— 但如果有帮助的话，我写这篇文章的第一个版本时也不知道这个区别，而我 *基本上* 从中对 SPF 有了一个不错的理解。

### DKIM

`DKIM` 是声明来自你的域的邮件签名密钥的一种方式。它代表 DomainKeys Identified Mail。再说一遍……这并不重要。它读作 Dee-Kim。你可以将它视为 SPF 的加密表兄弟。

它意味着接收邮件的电子邮件服务器可以检查该邮件是否由知道一个秘密的服务器发送。由于它是公钥加密，有两个密钥：一个是私密的，由你的电子邮件发送服务器（SMTP）持有且其他人不知道，另一个是公开的，设置在你的 DNS 中，任何人都可以看到并用于确定是否使用那个秘密密钥进行签名。

顺便说一句，一个域名可以有多个DKIM密钥。我花了很长时间才弄清楚这一点。很可能，你使用的大多数电子邮件发送服务（Gmail、Office 365、Fastmail、Mailchimp、Postmark、Sendgrid、Mandrill、Postmark或其他）都会提供一个DKIM密钥，你可以添加到你的DNS记录中。如果你添加了该密钥，你就授权这些服务代表你的域名发送邮件 - 这类似于在SPF中添加它们，但不是关于域名，而是关于知道一个秘密，

这是它在顺利路径中的工作方式：

1.  我从`hello@sadl.io`发送电子邮件给`somebody@gmail.com`。

1.  为了发送该电子邮件，这封电子邮件经过我的Fastmail SMTP服务器。

1.  Fastmail SMTP服务器使用秘密密钥生成签名，并将其附加到电子邮件，然后将其发送到`example.com`的接收服务器。

1.  谷歌邮件服务器接收这封邮件。它来自`sadl.io`，所以它获取了DNS记录。

1.  该电子邮件嵌入了一个签名，并且`sadl.io`的DNS记录声明了几个公共DKIM密钥，这些密钥可以用来验证该签名。

1.  其中一个DKIM密钥与用于生成此签名的密钥匹配 - 具体来说，是Fastmail提供的那个。因此，DKIM测试通过。

1.  电子邮件落入`somebody@gmail.com`的收件箱。

这里的不幸路径非常简单。你可以使用来自SPF部分的同样PHP脚本。这是会发生的事情：

1.  我运行这个脚本。邮件发送到`employee@example.com`。

1.  这封电子邮件被谷歌接收，他们运行`example.com`的邮件服务器。

1.  由于这是来自`example.com`的电子邮件，Google获取了`example.com`的DNS记录

1.  `SPF`测试失败了。但是*可能*没关系，因为`example.com`在其DNS记录中声明了一些DKIM密钥。

1.  这封电子邮件没有附加签名。因此，这些DKIM密钥都没有用。（或者：电子邮件*确实*有附加签名，但与`example.com`上的任何公共密钥生成的签名不匹配）。

1.  由于电子邮件未通过SPF测试*和*DKIM测试，它被标记为垃圾邮件。*也许*。

是的。再一次：*也许*。一封电子邮件可能会*同时*失败这两个测试，但仍然会落入用户的收件箱。你已经付出了这么多的努力，*仍然*，你的电子邮件域名可以用5行*非常基础的*PHP进行欺骗。

如果你是我，已经花费数小时达到这一点。你感到沮丧和失去动力。你尝试过的每件事都很难理解，并且需要大量研究。你紧张地更新DNS记录，担心会破坏你组织的电子邮件服务 - 只是发现，相反地，你实际上什么也没做。一切都很糟糕。你短暂地重新考虑了你的职业选择。

在你要实现的事物清单中，还有一个缩写词：DMARC。另一个可怜的电子邮件安全实践要实施，并且 - 你预期 - 又一个导致死路的解决方案。

好吧。好的。见鬼。我们就这么搞吧。但如果这不起作用，我放弃。

### DMARC

我有个好消息告诉你。DMARC *会*使 SPF 和 DKIM 运行得更好。

但也有一些坏消息：它有点复杂，并且需要一些更多的基础设施。

那么，什么是 DMARC？在其核心，它实际上是两件事情，打包在一个 DNS 记录中：

1.  它是一个政策，宣布当电子邮件未通过 SPF 和 DKIM 测试时，邮件服务器应该做什么。

1.  这是一个报告系统，让你可以找出谁在试图伪造你的域名。

你可能更感兴趣于1，但2非常重要。要理解为什么，让我们谈谈为什么 SPF 和 DKIM 并不是你以为的那样。

电子邮件很复杂。如果你到现在还没有得出这个结论，我想和你聊聊。这是一个由几十年来编写的无数标准维持的旧系统。人们对电子邮件做出了意想不到的事情。有些复杂的系统是在90年代设立的，至今仍在电子邮件之上运行，毫无阻碍。

改变邮件工作方式通常会导致一些问题。这就是 SPF 和 DKIM 所做的事情。

SPF 和 DKIM 被用作是否伪造邮件的*指标*。但如果你在你的域上添加了一个 SPF 记录，却忘记为你的邮件系统之一添加 - 比如说，Postmark，你用它向客户发送应用程序的重要通知 - 那么你的客户可能就收不到邮件了。如果你在你的域上添加了 DKIM 密钥，但是你的某个邮件服务不支持 DKIM - 或者你忘记为该服务添加 DKIM 密钥 - 那么你的客户也可能收不到邮件。

但他们*并没有*停止收到邮件。有些邮件将同时失败 SPF *和* DKIM，但仍然会进入用户的收件箱。这完全取决于接收邮件服务器的自由裁量权，它们可能会相当宽松。这对于你忘记添加的服务来说是好事，但对于你试图消灭的伪造者来说是坏事。

你想做两件事：

1.  找出你配置错误的服务，这样你就可以修复它们。

1.  阻止伪造者滥用你的域名。

DMARC 的“报告”部分帮助你，在你的电子邮件域安全工作的早期阶段，找出你配置错误并需要更新的服务。你可以启用报告部分，*而不*强制执行严格的 SPF/DKIM 策略。换句话说：你可以使用 DMARC 找出从你的域发送的邮件，这些邮件*将*失败 SPF/DKIM 测试，*而不*告诉邮件服务器所有失败这些测试的邮件都应标记为垃圾邮件。

一旦你确信已配置好一切，并且所有收到的报告都是关于伪造邮件，你可以更新你的策略，告诉邮件服务器这些测试现在很重要。这给了你一些时间来验证和测试，在你切换到强制执行 SPF/DKIM 规则之前。

实施 DMARC 大致如下：

1.  阅读这篇文章，对着墙头疼，公开哭泣。

1.  在你域名的 DNS 中实施一个宽松的 DMARC 记录，这样你就可以开始收到报告。

1.  仔细阅读接下来几天和几周收到的报告。检查是否有任何有效的电子邮件因 DKIM 或 SPF 失败而被标记。如果有，修复它们。

1.  更新您的 DMARC 记录以减少宽松性。

不幸的是，实施 DMARC 是一个过程。当我开始调查这个问题时，我希望能够复制一些良好的 SPF、DKIM 和 DMARC 记录，更新它们以匹配我的域，并在一两个小时内实施它们。事实并非如此。这需要一些努力，您需要让一些时间过去。

换句话说：电子邮件很复杂。

幸运的是，正如我所说，我在这里已经做了艰苦的工作，因为我认为 *任何人* 都不应该再自己摸索。对您来说，这个过程将在几天或更可能是几周内进行，但在几分钟内，您就可以成功地走出这一步。

#### 让我们谈谈对齐问题

DMARC 的魔力部分在于它使所有这些变得更加完美：它在 SMTP MAIL FROM（也称为信封发件人）和 From 头部之间强制实施 *对齐*，*或者* 在 DKIM 签名域和 From 头部之间实现对齐。在这两种情况下，From 头部可以被视为锚点：用户看到的是它，所以 SPF 或 DKIM 需要保证 From 头部是它应该是的。

最终这样做的 PHP 脚本在顶部现在只会在满足以下两个条件之一时成功地将电子邮件送入收件箱：

1.  PHP 脚本正在一个授权为 `example.com` 发送邮件的 IP 地址上运行，使用 SPF，*并且* SMTP MAIL FROM 和 From 头部都以 `example.com` 结尾，*或者*

1.  PHP 脚本连接到一个可以使用 DKIM 为电子邮件签名的邮件服务器，使用适用于 `example.com` 的密钥，From 头部以 `example.com` 结尾。

（在这两种情况下，假设 `mail()` 函数连接到同一 IP 上的邮件服务器。否则，我需要使用的词汇会变成一团乱。脚本可以连接到外部邮件服务器，只要 *该* 邮件服务器在 SPF 中被允许，或者可以用 DKIM 为该域签名，它就能正常工作。）

现在可能是一个回到[信封发件人](#on-envelopes-vs-letters)部分的好时机——它与本部分完美配对。

## 如何使用它们？

好消息是：您的域名 *可能* 已经设置了 SPF 和 DKIM 记录。如今，大多数电子邮件服务会提供正确的 DNS 记录供您添加，并提供验证工具来确保它们设置正确。

### 实施 SPF

SPF 是设置为您域名的 DNS 中的 `TXT` 记录。这是我的 simonandrews.ca 的样子：

```
v=spf1 include:spf.messagingengine.com include:spf.mandrillapp.com -all 
```

它直接在 `simonandrews.ca` 上声明为 `TXT` 记录——不是在子域上。如果我要从 `newsletter.simonandrews.ca` 发送电子邮件，我需要为该子域 *再* 添加另一个记录。

大多数 SPF 记录会像这样，尽管许多会有多个 `include:` 子句。让我们来详细解释一下：

+   `v=spf1`: 声明这是一个 SPF 记录，与你可能在 TXT 记录中声明的其他内容不同。这是 SPF v1 版本。

+   `include:spf.messagingengine.com`: 允许从任何 `spf.messagingengine.com`（Fastmail 的 SPF 子域）发送邮件。通常情况下，这意味着 `spf.messagingengine.com` 有其自己的 SPF DNS 记录，其中可能会列出一些有效的 IP 地址，可以从这些地址发送邮件。对于 transactional emails，也有一些其他服务，比如 Mandrill。你可以有任意数量的这些声明。如果在你的域上设置了 SPF 后想查看 `include:` 是如何工作的，请参考 DMARC 分析器 [SPF Record Checker](https://www.dmarcanalyzer.com/spf/checker/)。

+   `-all`: 这个很容易混淆，请注意。你可能会认为这是一个像在终端命令中使用的参数。但实际上不是。它实际上表示对“所有其他”的“失败” (`-`)。换句话说：“如果前面的声明都不匹配你从我们域收到的邮件，那么你收到的邮件很可能是伪造的。”

最后一个 `-all`，特别令人困惑，因为许多邮件服务推荐的是 `~all`。那是一个波浪符，不是破折号。很容易忽略。波浪符表示“应该失败，但不执行任何操作”。其他一些服务可能建议使用 `?all`，看起来像是编码错误，但实际上表示“如果邮件不匹配这些域，那么就不要紧”。*如果你使用 `~all` 或 `?all`，你的 SPF 记录实际上没有起到多大作用，即使你设置了 DMARC。* 相反，应该使用 `-all`。**真的，请使用 `-all`。** 总是把它放在你的 SPF 记录的最后。

正如我所说，大多数邮件服务现在在设置服务时提供应该设置的 SPF 记录，并会从他们的端口验证你的 DNS。你应该与每个提供商双重检查你的 SPF 当前配置是否正确。在服务的控制面板或配置中，可能会有一个“域”部分指导你并验证你的设置。如果有需要，他们的支持团队可能会提供帮助。还有一点要记住：如果他们建议使用 `~all` 或 `?all`，请忽略并使用 `-all`。

**N.B.** SPF 之前曾经是其自己的 DNS 记录类型 - 你会在域的 DNS 中声明一个 SPF 记录，而不是 TXT 记录来包含 SPF 信息。如果你的域有 SPF 类型的记录和 TXT 类型的记录，删除 SPF 记录。如果你只有 SPF 类型的记录，将它们移动到 TXT 类型的记录中。这涉及到 DNS，可能会让你感到焦虑，但请相信我。SPF 类型的记录正在逐渐消失，所有遵循 SPF 记录的邮件服务都将从 TXT 类型的记录中读取它们。你应该进行这个操作。

#### 主要要点

你可能已经设置了 SPF，即使当时你并不知道自己在设置它。这可能只是你注册电子邮件平台时做的一百个小任务之一。但是你应该检查它。使用你的电子邮件服务工具来检查它们，但也要进行手动检查。而且永远不要忘记使用 `-all`。

#### 资源

+   dmarcian 提供了 [SPF 记录语法概述](https://dmarcian.com/spf-syntax-table/)。比我描述的要多得多，但根据我的经验，你并不需要那么多细节。无论如何，现在你已经掌握了基础知识（顺便说一句，恭喜你！），这个页面应该很容易理解。

+   DMARC 分析器的 [SPF 记录检查器](https://www.dmarcanalyzer.com/spf/checker/) 是检查你的 SPF 记录的好方法。

### 实施 DKIM

有了 DKIM，你甚至更加受制于你的电子邮件提供商。他们可能会提供设置 DKIM 的说明，就像设置 SPF 一样。如果他们没有提供说明，那可能意味着他们不支持 DKIM。如果他们*确实*提供了说明，你应该*严格按照*它们来执行。

通常，DKIM 公钥存储在你的 DNS 记录中的 `_domainkey.yourdomain.net` 的子域上 - 例如，对于 Fastmail，我在 `fm1._domainkey.simonandrews.ca`、`fm2._domainkey.simonandrews.ca` 和 `fm3._domainkey.simonandrews.ca` 上有按照他们的指示设置的密钥。它可能是一个 `TXT` 记录，其中直接包含密钥，或者是一个 `CNAME` 记录，指向你的电子邮件提供商托管的公钥。

当电子邮件由发送服务器使用 DKIM 签名时，签名将包含要使用的标识符 - 例如，在我的域上，可以是 `fm1`、`fm2` 或 `fm3` 中的一个。

我这里一点也不涉及结构*。*它会因服务提供商而异，但最终将在 `_domainkey.yourdomain.net` 的 DNS 记录下生成。同样，服务的域名面板中可能有相关信息告诉你应该做什么，并验证你的配置。你可能已经设置了 DKIM，并且不知道。

#### 主要要点

如果你的电子邮件提供商支持 DKIM，他们会告诉你如何设置。相信他们，并按照他们的指示操作。如果他们不支持 DKIM，这并不是世界末日 - 但请确保 SPF 设置正确。

#### 资源

+   Postmark 有一个 [非常好的 DKIM 文档](https://postmarkapp.com/guides/dkim)。比我写的任何东西都要好 - 尽管现在你已经理解了基础知识，将帮助你在第一次阅读时理解该文档。

+   再次提醒，DMARC 分析器有一个很好的 [DKIM 记录检查器](https://www.dmarcanalyzer.com/dkim/dkim-checker/)，你可以用来验证你的设置。你需要你的域名和密钥（例如 Fastmail 的 `fm1`），因为 DKIM 是声明在子域上，而不是你的域根目录上。

### 实施 DMARC

哦，天哪，你会讨厌这一部分。

这是我发现最难研究的部分。有一*堆*公司在搞SEO以推广他们的DMARC分析服务，但他们中没有一个公开解释DMARC是什么。他们不提供让您理解正在发生的事情和需要做什么的信息——相反，他们试图向您出售可以为您解决问题的东西。

我不信任搞SEO的公司，所以我也不相信这些文章。我想回到基础。

不过，这里有个坏消息。当我们讨论报告时，我稍微忽略了一些东西。请原谅我。我不想吓跑您。我们现在*如此接近*了。您已经理解了我们正在处理的大多数概念，即使您的电子邮件域名还没有完全锁定，但它们肯定已经好多了。但是... DMARC报告是以XML文件的形式发送的。

我非常非常抱歉。

DMARC报告并不是为了人类阅读的。如果您拥有任何量级的电子邮件域，您可能会收到大量关于欺骗的DMARC报告。这些报告是用机器来解释和汇总的。

换句话说：除非您愿意手动阅读XML报告以了解发生了什么（嘿，在小域名中，这完全可能！），否则您要么将在服务器上设置一些软件，要么使用第三方提供商。那些搞SEO并且对DMARC实际上是什么很神秘的公司？您可能会给其中一家公司一些钱。

不管怎样。让我们从基础知识开始。假装我刚才没说过。虽然手动解释报告不是什么有趣的事情，但拿到一些报告并了解一下是个好主意。我们将会经历三个阶段：

1.  实施宽松的DMARC策略并开始收集报告

1.  等等，观察和分析

1.  提高您的DMARC策略的严格性

这并不太糟糕。让我们从一个非常简单的DMARC策略开始。

#### 宽松策略

设置一个能接收报告的电子邮件地址。我用的是[postmaster@simonandrews.ca](mailto:postmaster@simonandrews.ca)，实际上它只是转发到我的个人电子邮件地址。然后，在`_dmarc.simonandrews.ca`上添加了一个`TXT`类型的DNS记录，看起来像这样——你的将会放在`_dmarc.mydomain.net`上：

```
v=DMARC1;p=none;pct=100;rua=mailto:postmaster@simonandrews.ca;ruf=mailto:postmaster@simonandrews.ca 
```

哇哇哇...那是什么？好吧。让我们分解一下。

这里是*策略*声明：

+   `v=DMARC1`表示“这是DMARC版本1”。

+   `p=none`表示“如果SPF或DKIM失败，您不需要采取任何措施。”还记得我说过我们将实施宽松策略吗？就是这样。

+   `pct=100`表示“将此策略应用于*所有*来自我的域名的电子邮件”

然后是*报告*声明：

```
rua=mailto:postmaster@simonandrews.ca 
```

这告诉邮件服务器向`postmaster@simonandrews.ca`发送**汇总**的欺骗邮件报告。

```
ruf=mailto:postmaster@simonandrews.ca 
```

这告诉邮件服务器向`postmaster@simonandrews.ca`发送**法庭**的欺骗邮件报告。

这个策略… 有效。如果您设置自己的邮件管理员电子邮件地址，并使用此DMARC记录，您将成功迈出第一步。继续前进。几天后，您将开始收到报告。

不相信？尝试使用类似这样的配置设置DMARC，留一点时间让其传播（因为DNS很慢），然后运行之前的PHP脚本，使用您的域名和您拥有的“收件人”电子邮件地址。您不会立即收到报告，并且脚本生成的伪造电子邮件甚至可能出现在您的收件箱中。但在24小时内，您可能会收到第一份DMARC报告。

#### 坐下来观察

开始收集这些报告。如果只是个人域名，您可能会很高兴手动查看它们。如果是您公司的域名，您可能会收到大量报告 - 您可能希望建立像[lafayette](https://github.com/LinkedInAttic/lafayette)这样的DMARC服务器，或者使用第三方服务来处理这些报告。

无论如何，在收到一些报告后，请开始分析这些报告。是否有未通过SPF或DKIM测试的电子邮件，而不应如此？您是否配置错误了某项服务？您或您的公司是否使用了您不知道的服务发送电子邮件？或者，是否有某个伪造者在大量滥用您的域名？

您可以期待什么？好吧，这是我得到的一份报告，我之前运行了那个PHP脚本并发送到我拥有的Gmail地址。我添加了一些评论，概述了它告诉您的内容。

```
<?xml version="1.0" encoding="UTF-8" ?>
<feedback>
  <report_metadata>

    <org_name>google.com</org_name>
    <email>noreply-dmarc-support@google.com</email>
    <extra_contact_info>https://support.google.com/a/answer/2466580</extra_contact_info>
    <report_id>14538673265069095400</report_id>
    <date_range>
      <begin>1628899200</begin>
      <end>1628985599</end>
    </date_range>
  </report_metadata>
  <policy_published>

    <domain>simonandrews.ca</domain>
    <adkim>r</adkim>
    <aspf>r</aspf>
    <p>none</p> 
    <sp>none</sp>
    <pct>100</pct> 
  </policy_published>
  <record>

    <row>
      <source_ip>81.100.0.0</source_ip> 
      <count>2</count> 
      <policy_evaluated>
        <dkim>fail</dkim> 
        <spf>fail</spf> 
        <disposition>none</disposition> 
      </policy_evaluated>
    </row>
    <identifiers>
      <header_from>simonandrews.ca</header_from> 
    </identifiers>
    <auth_results>
      <spf>
        <domain>simons-mbp.lan</domain> 
        <result>none</result>
      </spf>
    </auth_results>
  </record>
</feedback> 
```

如果您运行的是大型域名，您将收到大量此类报告。如果您运行的是小型域名，您可能能够自行处理。

无论如何，稍作等待后，您将对已正确设置策略感到自信。

#### 提高严格性

当我们首次添加DMARC时，我们设置了 `p=none`。这基本上是说，“什么也不做，只是在SPF和DKIM失败时告诉我。”

如果您对收到的报告感到满意 - 即报告中所有电子邮件都来自伪造者，并且您发送的所有电子邮件都通过了测试 - 您可以增加此参数。您有两个选择：

1.  设置 `p=quarantine`。这告诉接收服务器，如果SPF和DKIM都失败，电子邮件应该被隔离。这可能意味着它立即被标记为垃圾邮件并发送到接收者的垃圾邮件文件夹中。也可能意味着它被放入隔离区，接收域的管理员可以批准或拒绝它。无论如何，这个选项都很好：它确保了您的域名安全，但用户遇到问题时还有补救措施。

1.  设置 `p=reject`。这更严格。它告诉接收服务器，如果SPF和DKIM都失败，那么电子邮件应该直接被拒绝。如果您非常确信您已正确配置了一切，并且对未来的服务设置也很有信心，那么这可能是适合您的选项。我会建议您先设置 `p=quarantine`，然后在以后逐步将其提升到 `p=reject`。

现在，这取决于您了。

#### 资源

+   dmarc.org对[DMARC记录的构成](https://dmarc.org/overview/)有一个很好的概述。如果您想直接跳到实质性内容，请阅读标题为“DMARC资源记录的解剖”部分。

+   dmarc.org也有一个关于[代码和库](https://dmarc.org/resources/code-and-libraries/)的列表，您可以使用。这就是我找到lafayette的地方。

+   dmarcian有一个[DMARC记录检查器](https://dmarcian.com/dmarc-inspector/)，将帮助验证您的策略设置是否良好。

+   Postmark有一个关于[DMARC工具的指南](https://postmarkapp.com/blog/best-dmarc-tools)。这个指南是一个很好的、相对不偏不倚的分析。在这里，您可能需要为服务付费，~~Postmark以不偏不倚的视角提供这些信息，因为——在撰写本文时——他们并未提供这样的服务。~~ 但是Postmark提供了一个合理的分析（事实证明，DMARC Digests是他们提供的服务，因此这并不是一个不偏不倚的来源。感谢[Hacker News上的max1cc](https://news.ycombinator.com/item?id=28193601)指出这一点。）

## 故障排除

### 没有收到报告？您可能需要EDV。

*感谢[Nicolas Lenz](https://www.eisfunke.com)联系我！*

如果您已正确设置了一切，并且正在等待未到达的DMARC报告，您可能需要设置EDV。

您不能将DMARC报告发送给*任何人*。例如，我不能在`simonandrews.ca`上设置DMARC并发送报告到`tim@apple.com`。很可能Tim Cook并不希望了解来自我的域名的邮件可达性（他的损失）。为了防止令人讨厌的、未经请求的报告，DMARC实施了一个名为EDV的系统（这根据您与谁交谈，可能是外部域验证或外部目的地验证）。

那么，何时需要它呢？您需要满足这两个主要条件：

1.  您正在将您的DMARC报告发送到不同的域名。（如果我从`simonandrews.ca`发送报告到`postmaster@simonandrews.ca`，EDV不会是必需的，因为这不是外部的。请记住：*外部*域名验证。）

1.  您*没有*使用第三方DMARC报告服务。（这些工具应该自动设置EDV。）

一个需要EDV的示例：我正在为我拥有的`sadl.io`域设置DMARC，并且我想要发送报告到我拥有的另一个域上的邮件地址[postmaster@simonandrews.ca](mailto:postmaster@simonandrews.ca)。为了使报告工作，我需要在`sadl.io`上配置DMARC，并且在`simonandrews.ca`上配置EDV。

一旦您理解了所有这些，就很简单了：在接收域名（在这种情况下为`simonandrews.ca`）的DNS `TXT`记录上，添加一个`sadl.io._report._dmarc`子域的值为`v=DMARC1`。换句话说，类似于...

```
sadl.io._report._dmarc.simonandrews.ca TXT "v=DMARC1" 
```

现在，在提供者将与 `postmaster@simonandrews.ca` 相关的 `sadl.io` 的电子邮件发送之前，它会检查 `sadl.io._report._dmarc.simonandrews.ca`，看看是否愿意接收它们。

您可以在 dmarc.org 文章 [Receiving DMARC Reports Outside Your Domain](https://dmarc.org/2015/08/receiving-dmarc-reports-outside-your-domain/) 找到更多信息。

## 总结

有可能，您应该实施所有这些。但是随着您的学习，您的实施可能会与我的有所不同。

我的主要目标是为您提供一个词汇表，让您了解 SPF、DKIM 和 DMARC 是什么。我不是专家。我在这里可能做错了一些事情。而且明确地说，本文仅介绍基础知识。

但希望现在您在这里，您已经有了基本的了解，也许开始实施了。您可以阅读其他一些文章，不会完全迷失方向。我建议您查看我链接的一些资源 - 它们并不复杂，大多数不会试图向您推销东西。

感谢您陪伴我走过这段疯狂而并不那么令人满意的旅程。我会尽量在学到更多知识时保持这篇文章的更新，并欢迎您提供任何反馈意见。如果我在这里有任何错误（我绝对肯定有），请联系 [hello@sadl.io](mailto:hello@sadl.io)。

### 更正

您可以查看我所做的所有更改 [在 GitHub 上](https://github.com/simon360/simonandrews.ca/commits/main/pages/articles/how-to-set-up-spf-dkim-dmarc.mdx) - 如果您之前阅读过本文，只想查看新内容，我建议您查看差异。

1.  之前的版本声称，这篇文章认为 Postmark 提供了关于 DMARC 分析服务的公正观点，因为他们并未提供这样的服务。实际上，他们提供了一个名为 DMARC Digests 的服务，在他们的指南中有提到。**感谢 [Hacker News 上的 max1cc](https://news.ycombinator.com/item?id=28193601) 纠正了这一点。**

1.  SPF 部分声称 SPF 在发件人头部起作用。实际上，它在 "envelope from" 上起作用，这只存在于 SMTP 传输层。而不是纠正这一点（我确实尝试了，但该部分变得过于复杂），我添加了一个额外的部分来解释区别。**感谢 Liam 通过电子邮件联系我并纠正我。**

最后更新时间：2021年8月17日，晚上7:10 BST
