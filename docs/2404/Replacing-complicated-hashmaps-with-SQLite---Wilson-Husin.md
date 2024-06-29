<!--yml

分类: 未分类

日期: 2024-05-27 12:58:47

-->

# 用 SQLite 替换复杂的哈希映射 | Wilson Husin

> 来源：[https://husin.dev/ephemeral-sqlite/](https://husin.dev/ephemeral-sqlite/)

最近的一个项目中，我正在处理一个带有棘手数据操作场景的命令行应用程序。想象一下，如果你正在将邮件列表的成员（例如 Google Groups）移动到群组聊天服务（例如 Slack），目标是让邮件列表的成员在群组聊天频道中找到新的归属地。

对于有黑客精神的工程师来说，说“将其脚本化并运行每个组的命令”是完全合理的，我同意 —— 当然，少量 HTTP 调用和哈希映射可以解决这个问题？

结果证明，这种情况带来了一些曲折：

+   邮件列表可能会有其他邮件列表作为成员（嵌套邮件列表），但群组聊天频道只有成员，因此在迁移时可能需要合并一些。

    +   存在多对一关系的潜力，即多个邮件列表实体最终合并为一个频道。

+   有些邮件列表已经有现有的群组聊天频道对应，但成员资格不一定保持一致。

    +   并非所有请求都是“创建频道”。

    +   用户和成员资格也是如此 — 考虑现有记录。

+   邮件列表的成员通过电子邮件地址识别。群组聊天服务可以通过电子邮件查找用户，但需要用户ID来进行 API 调用以链接频道成员资格。

    +   我们需要通过电子邮件和用户ID来查找群组聊天服务的用户；这是相同值类型的两个查找键（哈希映射）。

现在看起来并不那么简单了，对吧？当我在考虑这个问题时，脑海中有一个悬而未决的想法：“如果我可以在任何字段上使用 `WHERE` 并且 `JOIN` 多个实体，新的成员资格可以在单个 SQL 查询中解决。”

然后我意识到 — 或许我真的可以。所以我确实这样做了。

我最初实施的大部分内容现在都变得简单得多，而不必担心在每个函数中传递状态。初始实现使我的函数签名看起来像这样：

```
// MatchUsers performs best effort matching of users, returning: // - map[string]User of email with their User counterpart // - []string of emails without User counterpart // - error, if any func MatchUsers(  ctx context.Context, emailsToMatch []string, existingUsers map[string]User, ) (map[string]User, []string, error) 
```

几乎所有功能都需要传递状态以避免产生噪音和易于出错（例如，如果一个函数意外修改了哈希映射怎么办？）。

使用 SQLite 作为状态管理，相同的方法现在看起来像这样：

```
// MatchUsers performs best effort matching of users by comparing emails. // Users without any match will have `slack_user_id` as NULL. func MatchUsers(ctx context.Context, conn *sql.DB) error 
```

这样一来，当我想要查找未找到任何匹配项的用户列表时，它最终成为了一种直观的流程：

```
SELECT  *  FROM  mailing_list_users  WHERE  slack_user_id  IS  NULL; 
```

如果你发现自己在传递哈希映射、重建映射、编写太多条件来解决状态管理，并且认为在 SQL 中会轻而易举：考虑使用临时 SQLite 实例！

项目中的其他有趣细节：

+   因为数据库是临时的（使用 `:memory:`），我需要在每次应用程序启动时运行迁移。作为必然结果，我不需要迁移工具来管理向上 / 向下迁移！

+   当调试时，我可以使用 `sqlite3` CLI 来检查数据库，这在调试时是一个很好的奖励。当然，我需要将设置从 `:memory:` 改变为文件系统中存在的文件路径。

+   `MatchUsers` 的最终形式最终仅采用了 `context.Context`，因为这样对项目的其余部分访问数据库连接更为有效。
