<!--yml

category: 未分类

date: 2024-05-27 14:46:29

-->

# 介绍我的知识湖泊

> 来源：[`tabokie.github.io/non-fiction/2024/01/13/log-kb.html`](https://tabokie.github.io/non-fiction/2024/01/13/log-kb.html)

# 介绍我的知识湖泊

# 非小说，2024-01-13

1 只有三个实体：`log`、`list`、`structure`。

1.1 一切都按照时间顺序记录在 `log` 中。这是真相的来源。

1.11 我们将 `log` 中的最小内容单位称为 `entry`。

1.111 `entry` 可以是任何东西：一个想法，一个记忆，一条信息，一个任务的进度更新，一个待办事项，对另一个 `entry` 的引用……

1.12 即时性和不变性很重要。`entry` 在发生时被捕获，永远不会被修改。

1.121 没有审核或遗忘的余地。`entry` 是对现实的真实反映。

1.122 应该努力优化捕获的延迟。

1.1221 有时主要工具不可用。一个“始终在线”的次要工具应该代替它。

1.1222 有时需要“合并”、“持久化”来自不同工具或位置的 `log`。总的原则是减少手动步骤（例如使用自动合并的 Web 应用），并在固定的时间和地点（例如睡前推送到 git 仓库）进行日常操作。

1.123 在这个意义上，`log` 很类似于 [实验笔记](https://sambleckley.com/writing/lab-notebooks.html)（Sam Bleckley，2020）。

1.13 外部上下文（如果有的话）是 `entry` 的一部分。

1.131 至少应引用标题、作者和出版年。

1.132 更好的方法是，使用个人 URI 系统引用本地副本的上下文，例如 [Zotero 快照](https://www.zotero.org/support/adding_items_to_zotero#saving_webpages)，图片，书籍。

1.14 `log` 可以有不同类型。不同的 `log` 应该代表完全不同的领域。这些领域的思想通常发生在不同的时间和地点。

1.142 我目前有三种类型的 `log`：日志、阅读记录、工作日志。

1.2 `list` 和 `structure` 源自于 `log`。

1.21 一个 `list` 引用 `log` 中的多个 `entry`。

1.211 `list` 是 `entry` 的扁平集合，不包含其他内容。

1.212 `list` 中 `entry` 的顺序可能很重要，也可能不重要。这意味着，你可以将 `list` 视为 [`Array`](https://en.wikipedia.org/wiki/Array_(data_structure))，也可以将其视为 [`Set`](https://en.wikipedia.org/wiki/Set_(mathematics))。

1.213 我在这里有一些 `list`，如果你想看的话。

1.22 `structure` 以层次化的方式引用 `log` 中的多个 `entry`。

1.221 `structure` 的构建涉及许多作为“粘合剂”的思想。它们可以存在于 `log` 之外。

1.2211 将“粘合剂”思想想象成编写 [`struct`](https://en.wikipedia.org/wiki/Struct_(C_programming_language)) 时的数据成员名称。一个名称代表着观察 `entry` 的特定角度，`entry` 的一个不完整片段。

1.23 `list`和`structure`不需要详尽收集它们试图收集的内容。与`log`不同，`list`和`structure`的创建和维护是异步进行的。

1.24 使用自动化创建`list`是惯用法。例如，使用搜索和去重脚本创建所有未完成的 TODO 项目的`list`。

1.3 是的，就像一个数据湖。`list`和`structure`是对存储在`log`中的非结构化数据的物化视图。

2 对内部`entry`的引用只是内容和日期的一对。

2.1 内容是原始`entry`的副本或释义。

2.11 重复自己，与[DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)相反。把内容看作是完整`entry`的缓存。

2.2 日期是原始`entry`创建的时间。

2.21 解包引用需要读者检查整天的所有`log`，这有助于重新发现旧的想法。

2.22 通过搜索该日期，可以轻松找到对`entry`的引用。这些引用通常被称为[回链](https://en.wikipedia.org/wiki/Backlink)，或者在网络页面的上下文中称为[webmention](https://www.w3.org/TR/webmention/)。

2.221 对于不可搜索的`log`（例如纸质笔记本），需要创建一个可搜索的`list`来记录其中的所有引用。它包含一组`date1（date2）`，其中`date1`是被引用的日期，`date2`是创建此引用的日期。通过搜索`date1`，我们知道`date2`是它的回链。

3 所有的`log`都要定期和随机地复查。

3.1 通过反复阅读，学会记住过去的抱负和忧郁。学会与之不同意，并在不同意中找到前进的方法。

3.2 为了便于回顾，数字`log`应该打印成实体书籍。

3.21 没有什么比触摸实体书更让人舒服，尤其是当阅读的目的不是为了消费和完成时。
