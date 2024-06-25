<!--yml

category: 未分类

日期：2024-05-28 17:56:39

-->

# Rust 编写的 Linux 调度器显示出了游戏性能的有希望的结果 - Phoronix

> 来源：[`www.phoronix.com/news/Rust-Linux-Scheduler-Experiment`](https://www.phoronix.com/news/Rust-Linux-Scheduler-Experiment)

一个 Canonical 工程师一直在尝试用 Rust 编程语言实现 Linux 调度器。他早期的结果令人感兴趣，并且对于通过 sched_ext 使用 eBPF 实现调度器的 Rust 版本的潜力充满希望。

Andrea Righi 是 Ubuntu 制造商 Canonical 的 Linux 内核工程师

[tweeted](https://twitter.com/arighi/status/1746938387968254371)

他一直在尝试使用 Rust 调度器：

> “我在圣诞假期期间用 Rust 写了一个 Linux 调度器，只是为了好玩。我很惊讶地发现它不仅可以工作，而且在某些工作负载（即，游戏）下甚至可以优于默认的 Linux 调度器（EEVDF）。"

他分享了一个 YouTube 视频，展示了在后台运行并行内核构建时，“scx_rustland”调度器的游戏性能优于默认的 Linux 内核调度器：

[`www.youtube.com/embed/oCfVbz9jvVQ?si=dZCm3asl3SlytaQR`](https://www.youtube.com/embed/oCfVbz9jvVQ?si=dZCm3asl3SlytaQR)

视频

对于那些感兴趣的人，代码托管在

[GitHub](https://github.com/sched-ext/scx/)

这是一个有趣的圣诞假期冒险，很有意思看看它将通向何方。
