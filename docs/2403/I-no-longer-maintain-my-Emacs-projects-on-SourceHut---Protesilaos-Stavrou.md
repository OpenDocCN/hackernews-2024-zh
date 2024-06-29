<!--yml

category: 未分类

日期：2024-05-27 14:52:31

-->

# 我不再在 SourceHut 上维护我的 Emacs 项目 | Protesilaos Stavrou

> 来源：[https://protesilaos.com/codelog/2024-01-27-sourcehut-no-more/](https://protesilaos.com/codelog/2024-01-27-sourcehut-no-more/)

在 2022-04-07，我将所有的 Emacs 项目从 GitLab 迁移到了 SourceHut：[https://protesilaos.com/codelog/2022-04-07-all-emacs-projects-sourcehut/](https://protesilaos.com/codelog/2022-04-07-all-emacs-projects-sourcehut/)。现在我正在撤销这个决定。我的代码在 GitLab 和 GitHub 上，后者是事实上的主要来源。

为什么要进行这些改变：

+   SourceHut 的 Web 界面没有显示消息是否有附件的任何指示。上次我尝试时，我不得不下载一个 mbox 文件并从中提取补丁。虽然我知道我在找什么，但体验仍然不愉快。

+   许多用户不愿意订阅我的项目邮件列表，而是直接给我发电子邮件。这没关系，因为我最终能完成工作，但如果我是唯一一个读这些消息的人，那么公共收件箱的意义就不存在了。

+   个人邮件用于包的维护使我难以应用过滤器。我无法轻松地从“个人”切换到“包”，因此难以优先处理任务。

+   回复邮件列表线程的用户经常不选择“回复所有”，因此我为 SourceHut 列表设置的过滤器不起作用，我的收件箱再次杂乱无章。

+   协调我“官方”基于 SourceHut 的仓库和事实上的 GitHub 仓库之间的工作是个问题，因为一个平台上的用户可能不了解另一个平台上的用户在做什么。

还有其他问题，但我理解 SourceHut 目前仍处于“alpha”阶段，所以我不会在这里列出它们。

## Git 仓库在 SourceHut 上会怎样处理？

我不再更新它们，并计划在不久的将来将它们删除。

## 邮件列表怎么样？

我将继续回复发送到那里的消息，但最终会要求人们使用其他媒体。如果你不想使用 Git 集成来报告问题或发送补丁，请通过我的个人邮箱联系：[https://protesilaos.com/contact](https://protesilaos.com/contact)。

## 你会自己托管 Git 服务器吗？

我希望在某个时候这样做，因为我意识到专有 Git 集成的问题。但这是需要知识和资源的任务，所以这不会很快发生。
