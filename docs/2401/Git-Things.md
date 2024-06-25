<!--yml

category: 未分类

日期：2024-05-27 14:25:12

-->

# Git 事务

> 来源：[`matklad.github.io/2023/12/31/git-things.html#Git-Things`](https://matklad.github.io/2023/12/31/git-things.html#Git-Things)

每个提交都应该通过测试吗？如果应该，那么你的[不是火箭科学规则](https://graydon2.dreamwidth.org/1597.html)实现必须验证这个属性。它可能不是，只测试将特性分支合并到主分支的最终结果。

这就是为什么对于典型项目将拉取请求*合并*到主分支中是有用的——合并提交的线性序列是成功 CI 运行的记录，并且是你想要通过`git bisect`的一组提交。

在特性分支中，并不是每个提交都必须通过测试（甚至构建），这是一个有用的属性！以下是一些利用这一点的方法：

+   修复错误时，首先添加一个失败的测试，作为一个单独的提交。这样任何人都可以轻松验证测试确实失败了，没有后续的修复。

    相关建议：我经常看到人们注释掉当前失败的测试，或者是将来要修复的测试。这是不好的，因为注释掉的代码比当天的 JavaScript 框架更快腐烂。相反，调整断言使其锁定当前（错误的）行为，并添加清晰的 `// TODO:` 注释，解释正确的结果是什么。这样可以防止这些测试腐烂，并且也能捕获到通过其他无关变化修复了行为的情况。

+   要重构一个具有大量使用情况的 API，可以将工作分成两个提交。在第一个提交中，更改 API 本身，但不要触及使用情况。在第二个提交中，机械地调整所有调用点。

    在审查过程中，将有意义的更改与大量但不重要的差异分开变得非常简单。

+   `git mv` 是假的。很长一段时间，我相信 `git mv` 添加了一些特殊的 git 元数据，告诉它文件被移动了，这样就可以被 `diff` 或 `blame` 理解。事实并非如此：`git mv` 本质上是 `mv` 后跟 `git add`。在 git 中没有任何东西来跟踪文件是否被特定移动，"移动" 的幻觉是由 diff 工具创建的，它在启发式地比较两个时间点的存储库状态时产生。

    出于这个原因，如果你想要在 git 中可靠地记录重构期间的文件移动，你应该做两个提交：第一个提交*只是*移动文件而没有任何更改，第二个提交应用所有所需的修复。

    谈到移动文件，请考虑将以下内容添加到你的`gitconfig`中：

    ```
    [diff]
     colormoved = "default"
     colormovedws = "allow-indentation-change"
    ```

    这样，移动的行将在`diff`中以不同的颜色显示，以便代码动作不会与添加和删除混淆，并且更容易审查。我不清楚为什么这不是默认设置，以及为什么这不是 GitHub UI 中的选项。

“合并到主分支，但重置功能分支”可能是一个难以理解的严格规则，如果你是 git 新手的话。幸运的是，可以使用非常规则来强制执行这个属性。历史记录与源代码一样重要。你可以编写一个测试，该测试 shell 到 git 并检查历史记录中的合并提交是否只来自合并机器人。在此过程中，最好也测试一下存储库中是否存在大型文件 —— 存储库的大小只会增加，而且你无法轻易地后来删除大型二进制对象！

让我以最具煽动性的方式来表达这个观点 :)

如果你的项目具有出色的提交信息，具有简短而精确的摘要行和长而详细的正文，这可能意味着你的持续集成和代码审查流程有问题。

并非所有的变更都是相等的。在一个典型的项目中，大多数应该进行的变更都是小而微不足道的 —— 一些重命名、可见性调整、对用户可见功能中的“细节关注”进行打磨。

然而，在一个典型的项目中，落地一个微不足道的变更是缓慢的。你需要多长时间来修复注释中的`it's/its`拼写错误？实际更改可能需要 30 秒，获取 CI 结果可能需要 30 分钟，审查往返可能需要 3 小时。

进行变更的固定成本是巨大的。主分支的门禁制度极大地激励了反对微不足道的变更。结果，这类变更要么不会被执行，要么作为附带的奖励附加到较大的变更中。无论如何，提交和 PR 的总数都会减少。你正在编写一篇长篇大论的提交消息，因为你必须等待之前的 PR 完成才能进行下一个。

有什么可以做得更好的吗？

*第一*，使变更更小更频繁。很可能，这对你来说是可能的。至少，我倾向于在提交方面胜过大多数同事（[示例](https://github.com/intellij-rust/intellij-rust/graphs/contributors)）。这不是因为我更有生产力 —— 我只是以更小的批次进行工作。

*第二*，将持续集成设为异步。在整个工作流程中，你都不应该等待持续集成通过。你应该标记一个变更以进行合并，然后继续下一项任务，只有在持续集成失败时才回头查看。这是 bors-ng 做得对的地方 —— 可以立即在提交时进行`r+`操作。这是 GitHub 合并队列做错的地方 —— 在 PR 本身的检查未通过之前，无法将 PR 添加到队列中。

*第三*，我们的审查过程是反向的。审查是在代码进入主分支之前进行的，但对于大多数非关键任务项目来说，这是低效的。更好的方法是在非关键性变更允许的情况下，尽快乐观地合并大多数变更，然后稍后在主分支中对代码进行审查。而不是在 web ui 中添加评论，而是直接在原地更改代码，发送一个新的 PR 并抄送原作者。

> 1.  维护者不应该对正确的补丁进行价值判断。
> 1.  
> 1.  维护者应该快速地合并其他贡献者的正确补丁。
> 1.  
> …
> 
> 1.  任何对补丁有价值判断的贡献者应该通过他们自己的补丁来表达这些价值判断。

[集体代码建设合同](https://rfc.zeromq.org/spec/42/)

我怀疑这个确切的工作流程是否有可能，但我对[Zed 的](https://zed.dev)关于允许两个人同时在同一编辑器中编码的想法持谨慎乐观的态度。我认为这实现了类似的效果，并且很好地规避了允许暂时未经审核的代码引起的不安。

好了，回到 git！

*首先*，并不是每个项目都需要一个清晰的历史记录。你有没有看过你个人博客或 dotfiles 的 git 历史？如果你没有，可以用`.`作为提交信息。我在[`github.com/matklad/matklad.github.io`](https://github.com/matklad/matklad.github.io)上这样做，到目前为止效果很好。

*其次*，并不是每个变动都需要一个很好的提交信息。如果一个变动真的很小，我会说`minor`是一个可以的提交信息！

*第三*，一些变动确实需要非常详细的提交信息。如果有*上下文*，请尽量将所有内容包含在提交信息中（并在源代码中作为注释洒一些）。在这种情况下，这里有一个小贴士：*先写提交信息！*

当我致力于一个较大的特性时，我从`git commit --allow-empty`开始写下我要做的事情。大多数情况下，在提交信息的第三段，我意识到我的计划中存在一个缺陷并加以完善。因此，当我开始编写代码时，我已经在第二轮迭代了。而当我完成时，我只需用实际的更改修改提交，并且提交信息已经在那里，只需要进行微调。

最后一件关于提交信息的事情：`man git-commit`告诉我，摘要行应该少于 50 个字符。这显然是错误的，那太短了！[内核文档](https://www.kernel.org/doc/html/v4.10/process/submitting-patches.html)建议更合理的 70-75 限制！确实，看看最近的一些内核提交，50 显然不够！

```
<---               50 characters              --->
 get_maintainer: remove stray punctuation when cleaning file emails
get_maintainer: correctly parse UTF-8 encoded names in files
locking/osq_lock: Clarify osq_wait_next()
locking/osq_lock: Clarify osq_wait_next() calling convention
locking/osq_lock: Move the definition of optimistic_spin_node into osq_lock.c
ftrace: Fix modification of direct_function hash while in use
tracing: Fix blocked reader of snapshot buffer
ring-buffer: Fix wake ups when buffer_percent is set to 100
platform/x86/intel/pmc: Move GBE LTR ignore to suspend callback
platform/x86/intel/pmc: Allow reenabling LTRs
platform/x86/intel/pmc: Add suspend callback
platform/x86: p2sb: Allow p2sb_bar() calls during PCI device probe
 <---               50 characters              --->
```

亲爱的读者，新年快乐！
