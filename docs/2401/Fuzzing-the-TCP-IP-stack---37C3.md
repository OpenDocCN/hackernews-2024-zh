<!--yml

category: 未分类

date: 2024-05-27 14:25:13

-->

# Fuzzing the TCP/IP stack - 37C3

> 来源：[https://events.ccc.de/congress/2023/hub/en/event/fuzzing_the_tcp_ip_stack/](https://events.ccc.de/congress/2023/hub/en/event/fuzzing_the_tcp_ip_stack/)

我们的探索始于对之前应用于 TCP/IP 协议栈的传统模糊测试方法（如 ISIC）的诚实评估，揭示了它们固有的局限性，例如，它们无法超越 TCP 的初始状态。认识到需要更进化的方法，我们采取了不同的方法，利用全面的主动网络连接进行模糊测试。在这段旅程中的一个关键发现是有意绕过构建自定义 TCP/IP 协议栈的艰巨任务，这是一种根植于实际考虑的选择。

不愿意构建定制的 TCP/IP 协议栈导致我们采用创新策略，比如在 Linux 内核中嵌入钩子，并利用用户空间的 TCP/IP 协议栈，如 PyTCP、Netstack（Google gVisor 的一部分）和 PicoTCP。PicoTCP 处于核心地位，提供了一个用户空间的 TCP/IP 协议栈，成为我们状态模糊测试方法的重要组成部分。与会者将更深入地了解其架构、API 和文档，欣赏其在加固网络安全方面的关键作用。

随着演示的展开，我们穿越了开发一个强大模糊测试器的过程，这是我们识别 TCP/IP 协议栈中漏洞的方法的核心要素。揭示了驱动流量通过系统的复杂性、模拟真实场景以及利用可重现性和诊断技术的细节。讨论扩展到展示切实结果，包括获得的奖杯、报告的错误以及项目在 GitHub 上的最终发布。本次会议以引人入胜的问答环节结束，鼓励参与者深入探讨 TCP/IP 协议栈模糊测试的复杂性及其对网络安全的深远影响。
