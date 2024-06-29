<!--yml

category: 未分类

date: 2024-05-27 13:25:18

-->

# 使用Postfix延迟电子邮件传递以获得轻松的周末 | Enrico Bassetti

> 来源：[https://www.enricobassetti.it/2024/04/delay-e-mail-delivery-with-postfix-for-a-relaxing-weekend/](https://www.enricobassetti.it/2024/04/delay-e-mail-delivery-with-postfix-for-a-relaxing-weekend/)

一个很好的建议是，如果您想度过愉快放松的周末，永远不要阅读电子邮件。不幸的是，这并不容易：您为开源项目做贡献，朋友、爱好和新闻通讯等都是不错的理由偶尔读电子邮件。

但是我们都有那位朋友，那个经常发送电子邮件的人，你不想在周末读他们的邮件。让我们在Postfix中实现一些“延迟到周一”的功能！

## Postfix队列

您知道吗，[Postfix有不同的*队列*](https://www.postfix.org/QSHAPE_README.html#queues)吗？实际上，Postfix有：

+   **maildrop**是用`sendmail`发送到本地的电子邮件的队列。即使Postfix未运行，这也是唯一可用的队列；

+   **incoming**由`cleanup` Postfix进程使用，用于存储刚到达的电子邮件。它的存在是因为`active`队列（见下文）只能容纳固定数量的活动电子邮件；

+   **active**是包含预定发送电子邮件的队列。为了避免过载Postfix，此队列具有固定容量（您不希望系统尝试同时发送超过200k封电子邮件，对吧？）；

+   **deferred**用于临时错误（即临时问题），因此电子邮件被排队在此处，并稍后重试；

+   最后，**hold**是一个特殊队列，如果某个管理ACL如此指定，消息将进入该队列，并使用`postsuper`命令移动到其他队列。

从理论上讲，我们可能只需将电子邮件保留在*incoming*队列中，直到合适的时机或将其移至*active*队列并稍后安排投递。然而，这两种方法都不太符合这些队列的描述，据我所知，没有简单的方法可以做到这一点。

我们将使用**hold**队列！

## Postfix查找表

计划如下：我们将使用ACL将传入的电子邮件与`HOLD`操作匹配，以便Postfix将传入的电子邮件移动到*hold*队列。然后，我们可以使用一些bash脚本和crontab在适当时释放它们。

进入[Postfix查找*表*](https://www.postfix.org/DATABASE_README.html)！

Postfix可以通过在*表格*中匹配电子邮件的特定部分（或信封）来查找信息，以获取其他信息，例如操作、别名目的地、允许的主机等等。这些表可以存储在本地文件或某些网络服务中。这非常方便，因为你基本上可以[将Postfix连接到数据库服务器以查询电子邮件等内容](https://www.postfix.org/pgsql_table.5.html)，这基本上是每个（具有大量地址的人）都在做的事情。

在我们的情况下，我们需要使用[一个`access`表](https://www.postfix.org/access.5.html)。`access`表返回匹配的操作。具体来说，我们正在寻找`HOLD`操作。理论上，像这样的文件应该足够了：

```
[[email protected]](/cdn-cgi/l/email-protection)    HOLD 
```

如果你保存一个像这样的文件，并且在Postfix配置中的特定参数中使用`file:`表类型（如我们后面所看到的），那么所有来自`[[email protected]](/cdn-cgi/l/email-protection)`的电子邮件都将被移动到*hold*队列中，太好了！

问题在于似乎没有办法为此规则指定时间段。这意味着无论你是否使用`postsuper`将其从队列中删除，该规则都会始终匹配！效果不是很好。我们还能用什么？

结果证明我们可以使用数据库来实现这一点。如果你已经在使用数据库，那完全没有问题。但是，由于我的Postfix设置非常简单，我没有它。在这种情况下，我们可以简单地将`sqlite`作为数据库，并使用一些SQL魔法查询本地文件。

首先，我们需要一个SQLite数据库。确保Postfix编译时使用了`sqlite`表类型（默认情况下是这样），并且你的路径中有`sqlite3`可执行文件。然后：

```
$ sqlite3 /etc/postfix/time_based_access_sqlite.db SQLite version 3.40.1 2022-12-28 14:03:47 Enter ".help" for usage hints. sqlite> 
```

太好了！现在，让我们创建数据库表，用于存储Postfix表的信息：

```
CREATE TABLE time_access_dow (  email TEXT, day_of_week INTEGER, action TEXT ); 
```

现在我们可以关闭数据库文件，并创建包含“数据库连接信息”的文件。比如说，`/etc/postfix/time_based_access`：

```
dbpath = /etc/postfix/time_based_access_sqlite.db
query = SELECT action FROM time_access_dow WHERE email = '%s' AND day_of_week = strftime('%%w', current_date) 
```

正如你所见，我正在使用“星期几”（0-6，其中0表示星期日）。我们可以通过插入以下内容在星期日暂停来自`[[email protected]](/cdn-cgi/l/email-protection)`的电子邮件：

但我们还没有完成。我们需要告诉Postfix查询这个。我们将修改`/etc/postfix/main.cf`。具体来说，你要修改`smtpd_sender_restrictions`参数以添加你的表。请记住，配置的操作是按照你在配置中指定的确切顺序执行的。因此，你希望你的查询最后执行，或者至少在常规的检查（如黑名单、未知收件人域等）之后执行。

```
smtpd_sender_restrictions = reject_non_fqdn_sender,  permit_sasl_authenticated, ... check_sender_access sqlite:/etc/postfix/time_based_access 
```

重新启动Postfix以应用更改。现在，来自`[[email protected]](/cdn-cgi/l/email-protection)`的邮件在星期日将会暂停！

## 释放暂停的电子邮件

让我们创建一个脚本来释放它们（创建文件，然后+执行权限）：

```
#!/usr/bin/env bash set -eou pipefail   # Get all e-mail addresses that are configured for HOLD for email in $(sqlite3 /etc/postfix/time_based_access_sqlite.db "SELECT DISTINCT email FROM time_access_dow"); do  # Release all e-mails from HOLD for this e-mail for MSGID in $(postqueue -j | jq -r "select(.queue_name == \"hold\" and .sender == \"$email\").queue_id"); do postsuper -H "$MSGID" done done 
```

现在我们可以安排这个命令每天早上5点运行：

```
0 5     * * *   root    /etc/postfix/time_based_access.flush 
```

不幸的是，`postsuper`必须以超级用户身份运行。但作为良好的实践，你应该已经知道如何为用户配置必要权限以使用`sudo`和AppArmor。

这就是全部。你拥有了自己的基于后缀的系统来延迟电子邮件的投递！

## 注意事项

正如你可能注意到的，SQL查询仅检查工作日。因此，如果某人在周一凌晨00:01发送电子邮件，该邮件将被传送到你的账户（还有其他在早上5点发布的邮件之前）。如果你的朋友是夜猫子，你可能需要调整数据库结构来存储一些时间信息。

## 未来的改进

下一个改进将是查询我的日历以获取假期和休息时间。我有一个CalDAV服务器，所以应该不难处理（我看见你在笑，别笑了！）。也许下个周末我可以处理一下...等等，我不是期待一个愉快的周末吗？！？
