<!--yml

分类：未分类

日期：2024-05-29 12:30:14

-->

# GNOME开发者的愚蠢和傲慢 | 费利佩·康特雷拉斯

> 来源：[https://felipec.wordpress.com/2024/03/18/stupid-gnome-developers/](https://felipec.wordpress.com/2024/03/18/stupid-gnome-developers/)

大约一年前，我写了一篇博文，解释了GNOME项目中人们可怕的开发实践导致了 vte 中的一个 bug，尽管我不使用 GNOME，它仍然困扰着我：[GNOME可怕的编码实践](https://felipec.wordpress.com/2023/02/24/gnomes-horrid-coding-practices/)。

这篇文章非常技术性，但如果你能看懂，很明显他们犯了一个错误，并且他们拒绝承认自己的错误，尽管多位用户多年来都抱怨过这个 bug（不只是我）。

但真正的问题不是他们拒绝修复 bug，而是我给了他们一个完全有效的修复方案，他们却因为自负而连看都不愿看一眼。

今天有一个更新：他们提出了一个[他们自己的修复方案](https://gitlab.gnome.org/GNOME/vte/-/commit/1bcf4bca)。然而，讽刺的是，他们的修复版本是我自己修复方案的一个模糊和较差的版本。

那么问题是什么？很多人会认为这是某种“我告诉过你”的报复，来提升我的自尊心。不，问题是我的修复方案**三年前**就已经可用了。他们忽视和审查我，唯一达到的结果就是伤害了他们自己的用户。

他们本可以很久以前应用我的修复。

现在，在深入复杂细节之前，我认为澄清一下很有必要：这个 bug 是因为在事件发生时，他们没有发送事件信号。但即使问题复杂，修复却**非常简单**：事件发生后立即发送事件信号。就是这样简单。这就是我的修复所做的，最终他们也是这样做的。

三年来，他们争辩说延迟信号是好事，结果他们提出了一个不延迟信号的补丁，这正是我最初主张的情况。

如果他们犯了一个错误，并几年后意识到了，那是可以理解的人性，但事情并非如此。我认为他们愚蠢的原因不是因为他们犯了错误，而是因为他们仍然没有意识到他们所犯的错误。因此，他们很可能会再次犯同样的错误。

这是他们处理这个回归问题时所犯的错误的总结：

+   没有检测到回归的引入

+   拒绝撤销引入回归的更改

+   解决了这个回归问题

+   拒绝一个完全有效的修复方案

+   拒绝考虑有效修复的优点

+   拒绝考虑其他减轻回归问题的替代方案

+   锁定并审查了他们的问题跟踪器中的问题

+   尽管有多次报告，拒绝承认存在错误

+   最终仍然应用了有效的修复方案

+   从未承认有效的修复方案已经可用

## 问题

要理解这个问题，我们不需要理解虚拟终端的所有内容，我们只需要理解**两个信号**：`子进程退出`和`流结束`。

### 子进程退出

当你运行一个终端（比如gnome-terminal），总会有一个驱动一切的进程，比如`bash`。这就是“子”进程。

当子进程退出时，通常希望终端也退出（尽管可以配置为保持打开）。最终还是由最终用户决定如何处理，但终端库 — 在这种情况下是vte — 的工作是向终端（如gnome-terminal）提供相关信息，以便按照最终用户的意愿执行操作。

这是一种复杂的方式来表达，当bash退出时，需要将这一信息传达给gnome-terminal，以便它可以立即退出 — 因为99.9%的用户都希望如此。

### 流结束

大多数情况下，子进程退出时流就结束了。所谓的“流”指的是标准输出。因此，当bash退出时，就不会再有输出了 — 因为程序无法再执行任何命令：它已经完成了。一旦程序完成，所有的文件描述符都会关闭 — 包括`stdout`。

```
bash -c 'echo "bye dad"' 
```

但并不总是这样。一个进程总是可以启动子进程。而每一个子进程都会继承父进程的标准输出。

```
bash -c '(sleep 10; echo "bye grandpa") & exit' 
```

在上面的例子中，bash立即退出，但它产生了一个子进程，在10秒后会打印“bye grandpa”。这意味着`stdout`文件描述符不能立即关闭（直到bash的所有子进程退出）。

因此，`流结束`信号**不会**在子进程退出时发送，而是在孙子进程退出时发送，因为当bash退出时，`stdout`文件描述符仍然会保持打开状态。

### 直截了当地说

这在实际应用中意味着什么？

在实际应用中，这意味着当bash退出时，`stdout`文件描述符可能还没有关闭。换句话说：`流结束`信号可能会在`子进程退出`信号**之后**才到来。

[2019年引入的bug](https://gitlab.gnome.org/GNOME/vte/-/issues/204)是这样的：如果“流结束”信号还没来，那么“子进程退出”信号就会延迟。这个延迟可能是**无限**的。

那么gnome-terminal这样的终端究竟何时知道退出呢？它没有知道。这就是退步。

要立即退出，终端需要立即接收到`子进程退出`信号。但在[7888602c提交](https://gitlab.gnome.org/GNOME/vte/-/commit/7888602c)之后（lib: Rework child exit and EOF handling），这种情况不再发生：它被*无限期地*延迟了。

首先是无限延迟，然后被限制为[5秒](https://gitlab.gnome.org/GNOME/vte/-/commit/058adf5f)，最终被限制为[2秒](https://gitlab.gnome.org/GNOME/vte/-/commit/12a54279)。但延迟依然存在，这才是问题的所在。

## 解决方法

如果用户希望终端立即退出，则需要立即发送 `child-exited` 信号。就是这么简单。

### 愚蠢而显而易见

上述问题的解释复杂性可能使一些人困惑，因此我想确保人们理解这个修复。

修复就是在事件发生时发送事件的信号。所以修复就是在子进程退出时发送 `child-exited` 信号。

我不知道修复能更简单些了。

## 我不是为了让你思考而付钱。

如果你告诉某人 — 比如你的助手 — “请监控 NVDA 股票价格，当价格低于$850时通知我”。然后如果你得知股票价格是$825，你可能会对这个家伙感到不满，但别担心，你的助手有解释。

他可能会平静地对你解释：“我知道你的目标是赚钱，所以我推测赚钱的最佳方法是在股价为$800时买入，现在价格只有$825，所以还不是买入的时机”。

但你没告诉他“当你认为是买入时通知我”，对吧？你从未授权他做出那个决定。

这个 bug 非常相似。终端告诉 vte：“请在子进程退出时通知我”，而 vte 回答说：“现在还不是关闭终端的时候”。但这不是终端所要求的。

你看，GNOME 开发人员不仅认为他们比用户更懂，而且 GNOME 库的开发人员认为他们比使用这些库的用户更懂：vte 的开发人员认为他们比 xfce4-terminal 的开发人员（以及无数其他开发者）更懂。

他们声称，当他们认为终端应该关闭时发送 `child-exited` 信号是可取的，而不是在子进程退出时发送。

### 无序库

[我已经向他们解释过](https://gitlab.gnome.org/GNOME/vte/-/issues/204#note_1703057)，`child-exited` 信号应在子进程退出时发送。

> 要做出如此先进的心理运动来辩称 `child-exited` 信号在子进程退出时不实用，确实需要不少努力。
> 
> 费利佩·康特雷拉斯

[他们辩称](https://gitlab.gnome.org/GNOME/vte/-/issues/204#note_1702760)，是他们应该决定何时关闭终端：

> 总结到目前为止我们所获得的经验，既包括原始 bug，也包括对该 bug 的不太理想的“修复”（导致其他问题），期望的行为是**所有这些同时发生**：
> 
> 1.  处理来自直接子进程的所有输出。
> 1.  
> 1.  然后尽快退出（或重新启动，或其他）。
> 1.  
> 1.  无需处理所有来自孙子辈、他们的后代或其他以某种方式访问 tty 行的进程的输出。
> 1.  
> 埃格蒙特·科布林格

显然Egmont不理解分支上标准流的工作方式，因为无法区分子进程的输出和孙子进程的输出。但即使可能，决定何时退出应由终端决定：如果终端希望在处理完子进程的所有输出之前退出，它应该能够这样做。

最终，我 — 最终用户 — 应该决定，而不是终端，当然也不是终端库。库的工作是帮助实现最终用户想要的结果，而不是为最终用户做决定。

如果最终用户希望终端立即关闭并且不显示任何输出，vte应该保持安静并做到这一点。

最终，他的解决方案并没有比我的解决方案多做什么，我们很快就会看到。

## 证明

此时，您可能会认为GNOME开发人员绝对不可能对他们自己的代码如此无视，以至于像我这样的外部人员能够指出如此明显的错误。

嗯，让我们看看。

这是在应用最新修复之前的问题核心，在`Terminal::child_watch_done`方法中：

```
/* If we still have a PTY, or data to process, defer emitting the signals
 * until we have EOF on the PTY, so that we can process all pending data.
 */
if (pty() || !m_incoming_queue.empty()) {
        m_child_exit_status = status;
        m_child_exited_after_eos_pending = true;

        m_child_exited_eos_wait_timer.schedule_seconds(2); // FIXME: better value?
} else {
        m_child_exited_after_eos_pending = false;

        if (widget())
                widget()->emit_child_exited(status);
} 
```

你不需要理解那些在做什么，你只需要知道我的修复把它改成了：

```
if (widget())
        widget()->emit_child_exited(status); 
```

他们的修复做了什么？

```
/* If we still have a PTY, or data to process, defer emitting the signals
 * until we have EOF on the PTY, so that we can process all pending data.
 */
if (pty()) {
        /* Read and process about 64k synchronously, up to EOF or EAGAIN
         * or other error, to make sure we consume the child's output.
         * See https://gitlab.gnome.org/GNOME/vte/-/issues/2627 */
        pty_io_read(pty()->fd(), G_IO_IN, 65536);
        if (!m_incoming_queue.empty()) {
                process_incoming();
        }

        /* Stop processing data. Optional. Keeping processing data from grandchildren and
         * other writers would also be a reasonable choice. It makes a difference if the
         * terminal is held open after the child exits. */
        unset_pty();
}

if (widget())
        widget()->emit_child_exited(status); 
```

还有更多代码，但最后他们调用了`Widget::emit_child_exited`方法，这与我所做的完全一样。

所以现在他们在子进程退出时发送`child-exited`信号。**绝妙的主意！**

你可能认为也许信号可以在那个时刻发送的理由来自上面的额外代码，我会深入讨论这一点。但重要的是，完成后信号就发送了。就这样。

### 区别

或许他们添加的代码块改变了一切？

这是一个展示他们修复情况的视频：

这里有一个展示我的修复效果的视频：

你能发现区别吗？我怀疑你能，因为**没有区别**。

所以[阅读那64k数据](https://gitlab.gnome.org/GNOME/vte/-/blob/aeb663ae/src/vte.cc#L3699-3715)是不必要的，他们本可以简单应用我的补丁。现在，我可以详细解释为什么没有任何区别，但这并不必要，因为这甚至不是重要的用例。重要的用例 — 如下所示 — 是终端保持开放状态，所以有更多时间接收数据。

他们推迟了修复，**毫无收益**。

## 辩护

也许我有些不公平，他们可能有充分的理由。让我们探索一下他们的理由。

开发人员创建了另一个问题票以讨论他们的“正确”修复方案（[在存在子孙进程的情况下退出延迟](https://gitlab.gnome.org/GNOME/vte/-/issues/2627)）。在那里，Egmont用了4102个字来开始为他的提议辩护。但最终他的提议很简单：

> 从两个不同的方向来看，基于流量的安全帽对我来说似乎是最好的选择。此外，同步方法看起来比基于主循环的方法更安全，也更简单。因此，这就是我的补丁所做的事情。当我们注意到子进程退出时，我们会同步读取直到`EOF`、`EAGAIN`或任何其他错误条件，或者是64k数据，以先到者为准，并在发出略微延迟的`child-exited`信号之前处理这些数据。
> 
> Egmont Koblinger

这解释了我们之前看到的代码片段：它试图在发送`child-exited`信号之前同步读取64k数据，所以现在只是“稍微延迟”。

我已经证明了那块代码没有任何影响，但让我们假设他们是正确的，它确实产生了影响。为什么读取64k数据很重要呢？

### 起源故事

回归问题可以追溯到尝试修复问题[2366](https://gitlab.gnome.org/GNOME/vte/-/issues/2366)（用“保持终端打开”丢弃cmd输出的结尾）。但要记住的是，只有当你**保持**终端打开时，这才相关。

假设你启动一个终端来运行单个命令：

```
xfce4-terminal -e fortune 
```

自然而然，你想看到那个命令的输出。但是如果终端在命令完成后退出，那么你将什么也看不到。为了在子进程退出后看到输出，你需要保持终端打开：

```
xfce4-terminal --hold -e fortune 
```

报告在[2366](https://gitlab.gnome.org/GNOME/vte/-/issues/2366)中的问题是，如果发生了两件事，不会显示所有输出：

+   输出超过8k的数据

+   终端保持打开

所以这只是一个非常边缘的问题，可能只影响不到1%的用户（更像是0.01%）。

### 比疾病更糟糕

导致更多问题而不是解决问题的“修复”（[lib: 重写子进程退出和EOF处理](https://gitlab.gnome.org/GNOME/vte/-/commit/7888602c)）在单个提交中进行了几十次更改，这几乎不可能理解哪一项解决了问题。但在我花费大量时间将600多行代码拆分成[25个逻辑上原子的提交](https://github.com/GNOME/vte/compare/master...felipec:vte:eos-rewrite)之后，问题解决的原因就清楚了：

```
--- a/src/vte.cc
+++ b/src/vte.cc
@@ -3265,9 +3265,6 @@ Terminal::child_watch_done(pid_t pid,

         m_pty_pid = -1;

-        /* Close out the PTY. */
-        unset_pty();
-
         /* Tell observers what's happened. */
         if (m_real_widget)
                 m_real_widget->emit_child_exited(status); 
```

需要时间来解释PTY是什么，但你不需要知道那个：你只需要知道它是主要的通信渠道。因此，如果PTY关闭了，终端与子进程之间的链接就中断了。这意味着流被强制关闭，因此在此之后来自子进程的任何输出都将丢失。

如果PTY没有关闭，那么终端能够在`child-exited`信号触发之后接收来自子进程的输出。

本质上，修复措施将流的关闭与`child-exited`信号解耦。

那么接下来的 24 个更改做了什么？没有任何用处。事实上，最后两个更改重新连接了两个信号。之前：当发送 `child-exited` 时，流被关闭，之后：当流关闭时发送 `child-exited` 信号。

因此，一个问题被另一个问题替代，只不过新问题**更糟**。

### 神话

vte 开发者从未能克服的误解是，避免[问题 2366](https://gitlab.gnome.org/GNOME/vte/-/issues/2366)报告的唯一所需就是不关闭 PTY，从而分离两个事件：流从子进程退出后关闭。

他们无法设身处地地理解终端的立场，不明白他们发送 `child-exited` 信号并不会自动导致终端关闭。终端是自主的，它可以决定**忽略** `child-exited` 信号。我甚至写了一个简单的 vte 客户端终端来精确测试所有用例，并用它证明了当终端保持打开状态时，我的修复效果非常好，我在[问题报告中解释过](https://gitlab.gnome.org/GNOME/vte/-/issues/204#note_1703121)。在[问题 2366](https://gitlab.gnome.org/GNOME/vte/-/issues/2366)中明确指出，此问题只在选择了“保持终端打开”选项时才相关。

当选择了“保持终端打开”选项时，`child-exited` 不会终止终端。

终端可以在发送 `child-exited` 后继续显示来自子进程的输出，只要文件描述符未关闭。

因此，他们并不需要延迟 `child-exited`：他们所需做的只是不关闭 PTY。

他们从未真正理解修补程序为何解决了问题：他们错误地将其归因于延迟信号。

### 64 k 数据块

我们回到他们的修补：

```
/* If we still have a PTY, or data to process, defer emitting the signals
 * until we have EOF on the PTY, so that we can process all pending data.
 */
if (pty()) {
        /* Read and process about 64k synchronously, up to EOF or EAGAIN
         * or other error, to make sure we consume the child's output.
         * See https://gitlab.gnome.org/GNOME/vte/-/issues/2627 */
        pty_io_read(pty()->fd(), G_IO_IN, 65536);
        if (!m_incoming_queue.empty()) {
                process_incoming();
        }

        /* Stop processing data. Optional. Keeping processing data from grandchildren and
         * other writers would also be a reasonable choice. It makes a difference if the
         * terminal is held open after the child exits. */
        unset_pty();
}

if (widget())
        widget()->emit_child_exited(status); 
```

总会有一个 PTY，所以可以忽略 `if (pty())` 条件，那么逻辑就包括 3 步：

1.  读取 64 k 数据块

1.  关闭 PTY

1.  发出 `child-exited`

任何要读取的数据都必须在关闭 PTY 之前。通常情况下发送 `child-exited` 会关闭 PTY（实际上终止整个程序），但在选择了“保持终端打开”选项时不会如此。因此，在这种情况下，可以在**之后**读取数据。

因此，我们不需要执行步骤 2（实际上有一条注释说这是**可选的**，“在子进程退出后保持终端打开确实有所不同”，并且终端确实保持打开状态，因此确实有所不同），如果不执行步骤 2，则也不需要执行步骤 1：因为在发出 `child-exited` 后可以读取数据。

如果跳过步骤 2 和步骤 1（这些步骤是不必要的），那么代码与我的修复**完全相同**。

### 结论

他们无法自圆其说。他们的代码**没有任何区别**。

## 旅程

在 2019 年 11 月，`Terminal::child_watch_done` 中相关的代码在引入回归之前是这样的。

```
/* Close out the PTY. */
unset_pty();

/* Tell observers what's happened. */
if (widget())
        widget()->emit_child_exited(status); 
```

在此时，报告的 [2366](https://gitlab.gnome.org/GNOME/vte/-/issues/2366) bug 仍然存在：当子进程退出时显示超过 8k 数据并不起作用。

然而，修复方法很简单，只需移除 `unset_pty()` 行而不关闭 PTY。相反，他们选择增加了几层复杂性：

```
/* If we still have a PTY, or data to process, defer emitting the signals
 * until we have EOF on the PTY, so that we can process all pending data.
 */
if (pty() || !m_incoming_queue.empty()) {
        m_child_exit_status = status;
        m_child_exited_after_eos_pending = true;

        m_child_exited_eos_wait_timer.schedule_seconds(2); // FIXME: better value?
} else {
        m_child_exited_after_eos_pending = false;

        if (widget())
                widget()->emit_child_exited(status);
} 
```

修复问题 [2366](https://gitlab.gnome.org/GNOME/vte/-/issues/2366) 的过程中遇到了一个非常边缘的问题 — 只有在数据超过 8k 且终端配置为保持打开状态，并且只影响不到 0.1% 的用户 — 他们引入了一个导致所有用户都受影响的回归。

值得注意的是，`m_child_exited_after_eos_pending` 最终会通过强制 `end-of-stream` 信号触发 `child-exited` 信号，但这可能需要长达 2 秒的时间。

多年来，他们争辩称这是必要的，以便解决 [2366](https://gitlab.gnome.org/GNOME/vte/-/issues/2366) 中描述的用例。但事实并非如此：他们从未意识到解决问题的关键是移除 `unset_pty()` 的调用。

最终在 2023 年 8 月 — 大约 4 年后 — 他们终于通过以下修复了回归：

```
/* If we still have a PTY, or data to process, defer emitting the signals
 * until we have EOF on the PTY, so that we can process all pending data.
 */
if (pty()) {
        /* Read and process about 64k synchronously, up to EOF or EAGAIN
         * or other error, to make sure we consume the child's output.
         * See https://gitlab.gnome.org/GNOME/vte/-/issues/2627 */
        pty_io_read(pty()->fd(), G_IO_IN, 65536);
        if (!m_incoming_queue.empty()) {
                process_incoming();
        }

        /* Stop processing data. Optional. Keeping processing data from grandchildren and
         * other writers would also be a reasonable choice. It makes a difference if the
         * terminal is held open after the child exits. */
        unset_pty();
}

if (widget())
        widget()->emit_child_exited(status); 
```

但是他们重新引入了 `unset_pty()`，尽管注释本身说是“**可选的**”。如果去掉这段代码中读取 64k 数据的无用操作，我们最终得到的是：

```
if (widget())
        widget()->emit_child_exited(status); 
```

这正是 [我的修复](https://github.com/felipec/vte/commit/8158d9c3#diff-5a3d8ae32aded628803bb443f43eac61814650c78132f6e128d2c7be4712538fR3220-R3221)。

因此，让我们清楚地表达，我们从 2019 年的这段代码：

```
/* Close out the PTY. */
unset_pty();

/* Tell observers what's happened. */
if (widget())
        widget()->emit_child_exited(status); 
```

到 2023 年的这段代码：

```
/* A chunk of code that makes no difference. */

if (widget())
        widget()->emit_child_exited(status); 
```

它与我被无故拒绝的修复方案完全相同。

所有这些年来，我一直认为这两个信号应该解耦，并最终他们确实解耦了。不要提及我花在分析代码、尝试不同方法并与这些愚钝人争论的无数小时；我们（开源社区）通过这场折磨得到了什么？**完全没有**。他们最终添加的代码与我的修复毫无**区别**。

## 但等等…

当我说没有区别时，那是谎言，实际上情况更加**糟糕**。

如果你运行以下命令，你会期望在 10 秒后看到“bye grandpa”的输出。

```
xfce4-terminal --hold -e "bash -i -c '(sleep 10; echo bye grandpa) & exit'" 
```

但这并没有发生… 因为他们决定 [关闭 PTY](https://gitlab.gnome.org/GNOME/vte/-/blob/aeb663ae/src/vte.cc#L3714)。

在我的修复中，该命令可以正常工作。

我为此新开了一个 [问题票](https://gitlab.gnome.org/GNOME/vte/-/issues/2762)，并且我已经发送了一个包含修复的 [补丁](https://gitlab.gnome.org/felipec/vte/-/commit/16500861)，它只是移除了对 `unset_pty()` 的不必要调用。如果他们采纳我的修复，那么读取那些 64k 的操作显然是完全不必要的。但我不指望他们会这么做。

## 缺乏注意力。

在理解这个问题的过程中，我还原了我认为可能不必要的很多更改，而在这样做的同时，我发现这些更改确实是不必要的。我非常小心地做到了不漏掉任何东西，尤其是因为这些代码对我来说都是新的。

这就是我意识到 vte 开发者根本不仔细的方式：他们经常删除方法的调用却不删除方法本身。

例如，考虑这个片段：

```
@@ -10283,19 +10287,12 @@ Terminal::emit_pending_signals()
                 m_bell_pending = false;
         }

-        auto const eos = m_eos_pending;
         if (m_eos_pending) {
                 queue_eof();
                 m_eos_pending = false;

                 unset_pty();
         }
-
-        if (m_child_exited_after_eos_pending && eos) {
-                /* The signal handler could destroy the terminal, so send the signal on idle */
-                queue_child_exited();
-                m_child_exited_after_eos_pending = false;
-        }
 }

 void 
```

调用 `queue_child_exited()` 被移除了，但在此之后没有任何对该方法的调用，那么为什么他们不删除这个方法呢？因为他们没有**注意**自己的代码。

`Terminal::queue_child_exited` 排队调用 `emit_child_exited_idle_cb`（可能本来可以直接调用），然后调用 `Terminal::emit_child_exited`（可能本来不需要是一个单独的函数），最终调用：

```
if (widget())
        widget()->emit_child_exited(status); 
```

现在直接调用了这个，所以显然不再需要了。

所有这些方法都可以移除，因此 `m_child_exit_status` 也可以移除。

在 [回归补丁](https://gitlab.gnome.org/GNOME/vte/-/commit/7888602c) 中也发生了同样的事情，他们删除了 `Terminal::pty_channel_eof` 的定义，但没有删除声明。

我的 [修复版本](https://github.com/felipec/vte/commit/8158d9c3) 正确地删除了所有不再使用的代码。这就是为什么我说他们的版本更差。

我已经 [发送了一个补丁](https://gitlab.gnome.org/GNOME/vte/-/issues/2761) 来修复所有这些疏忽，但我想我们都能猜到可能会发生什么。

## 真的吗？

在一切都说完做完之后，Egmont 在 luit 命令中找到了一个 [有趣的评论](https://gitlab.gnome.org/GNOME/vte/-/issues/2627#note_1753270)：

> > -x 当子进程结束时立即退出。这可能导致 luit 在子进程输出结束时丢失数据。
> > 
> > luit(1)
> > 
> …
> 
> 他们遇到了同样的困境，并把决定权交给了用户。
> 
> Egmont Koblinger

没有衬衫，谢尔洛克。最终用户应该决定软件如何行为？谁会想到呢？

但你错了，他们并没有把决定权“交给”最终用户，这从一开始就是用户的决定。

### 软件的第一原则

我确信大多数开发者甚至不了解软件是什么，很显然包括 GNOME 的开发者在内。

> 任何程序 - 如内核或任何其他项目 - 任何时候破坏用户体验，对我来说这都是软件项目可能做出的最糟糕的失败。绝对不允许破坏用户空间 - 或者其他项目 - 永远不能破坏用户依赖的功能。
> 
> 因此，每当程序（如内核或任何其他项目）破坏用户体验时，对我来说，这是软件项目可能出现的最严重的失败。绝对不允许破坏用户空间 - 或者对其他项目来说 - 永远不能破坏用户依赖的功能。
> 
> 因为没有任何项目比项目的用户更重要。
> 
> Linus Torvalds

软件的整个意义在于对用户**有用**。就是这样。

你的工作 Egmont — 开发者，就是让我 — 最终用户，的事情变得更容易。而不是更难。

决定软件如何运行应该由最终用户决定。如果你故意使软件对最终用户**更不实用**，那你作为软件开发者是**失败**的。这是毋庸置疑的事实。

### 他们的罪行

一旦你理解了软件的本质，他们所犯的灾难性错误清楚无误：

1.  他们没有撤销引入**回归**的补丁。理解软件目的的项目会这样做，我在我的[另一篇博客文章](https://felipec.wordpress.com/2023/02/24/gnomes-horrid-coding-practices/)中提供了Linux项目的例子。

1.  他们没有撤销导致回归的行为。即使他们不想撤销整个补丁——因为这是一个有600行代码的大补丁（本来就是一个糟糕的做法）——他们也可以改变一行代码来恢复先前的行为。

1.  他们没有考虑减轻回归的方法。即使他们不想回到问题[2366](https://gitlab.gnome.org/GNOME/vte/-/issues/2366)仍然存在的状态，他们至少可以考虑一种折中办法，直到找到一个适当的修复方案。

1.  他们并没有允许终端用户的挫败感被听到。即使他们认为他们的解决方案是正确的，锁定问题并威胁那些批评这种方法的人永久封禁，并不是一个关心对用户有用性的项目应该做的事情。

1.  他们没有分析外部提出的任何修复方案。即使保留问题[2366](https://gitlab.gnome.org/GNOME/vte/-/issues/2366)的解决方案至关重要，他们也可以分析那些声称不会破坏问题[2366](https://gitlab.gnome.org/GNOME/vte/-/issues/2366)的修复方案，就像我的修复方案一样，但他们没有这样做。

即使你给他们以怀疑的余地，并认为他们在2023年最终引入的代码——[读取64K数据](https://gitlab.gnome.org/GNOME/vte/-/blob/aeb663ae/src/vte.cc#L3699-3715)——在某种程度上是必要的（尽管这并没有任何区别），他们在拥有那个“正确”的代码之前不应该启用新行为。

如果这意味着直到2023年才能正确修复问题[2366](https://gitlab.gnome.org/GNOME/vte/-/issues/2366)，那就这样吧。

让0.01%的用户在未来4年中受苦比引入回归并让100%的用户在未来4年中受苦更好。有谁会反驳这一点呢？

但这甚至不是事实，因为我的修复方案早在多年前就使一切正常运行了。

## 结论

这些人毫无理由地让他们的用户遭受了4年的痛苦。

当我说他们是“愚蠢”的时候，这并不是出于恶意，而是客观事实。谁会争辩说`child-exited`信号在子进程退出时不应该发送，最终却写了一个确实这样做的补丁呢？

当我说他们“傲慢”时，并不是因为他们拒绝了我的解决方案，而是因为他们甚至无法考虑到一些外部人可能比他们更正确，这就是为什么他们甚至没有看我的补丁。

这只是一个我们可以把它抛在过去的问题，但真正的问题是他们糟糕的开发实践和他们的糟糕态度。这不仅限于 vte 项目，而是 GNOME 整体，我有其他组件的类似恐怖故事来证明这一点。

这不是关于我或我的修复，而是关于详细记录 GNOME 开发者的操作方式，我认为任何客观的观察者都会得出这样一个结论：这根本不妙。

## 附言

因为这不是我第一次经历，我已经能预见到本文的批评：人们会关注我的个人形象和语气，而不是真正重要的事情：**GNOME** 的糟糕实践。

让我们明确一件事：即使是撒旦提供了一个补丁，只要这个补丁有效并修复了真实的问题，你都应该应用这个补丁，没有其他选择。为什么呢？因为这是对用户最好的选择。

我做的只是尝试修复他们糟糕的代码。问题不在于我。我甚至不关心他们会做什么，我已经转向了一个非 vte 终端：kitty（这个终端有一个选项可以立即退出：[close_on_child_death](https://sw.kovidgoyal.net/kitty/conf/#opt-kitty.close_on_child_death)）。

人们应该忽略我，专注于 GNOME 项目未来如何处理类似情况，尽管我知道许多人会做完全相反的事情。

## 附赠内容

有些人不相信我的修复有效，所以我制作了一个视频，逐步展示它是如何完美地运作的，与Egmont声称的相反。
