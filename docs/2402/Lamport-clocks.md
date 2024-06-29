<!--yml

category: 未分类

date: 2024-05-27 14:44:15

-->

# Lamport时钟

> 来源：[https://blog.fponzi.me/2024-02-02-lamport-clocks.html](https://blog.fponzi.me/2024-02-02-lamport-clocks.html)

# Lamport时钟

发布于2024-02-02 | 最后更新：2024-05-23 | 阅读时长12分钟

<details class="toc-container" open=""><summary>目录</summary></details>

上周末，我想找些灵感写一些TLA+规范，于是我重新阅读了Leslie Lamport的论文[《Time, Clocks, and the Ordering of Events in a Distributed System》](http://lamport.azurewebsites.net/pubs/time-clocks.pdf)。

这是Lamport最被引用的论文。它介绍了“先于发生”的概念和**逻辑时钟**，通常被称为*Lamport时钟*，并且提供了一个使用它们来解决分布式互斥问题的示例。

这篇文章是对论文的快速总结，并详细介绍了我基于论文中描述的互斥算法所写的规范。在撰写这篇文章的规范时，我还通过一些文档[贡献](https://github.com/hwayne/learntla-v2/pull/78)给了learntla网站。

## 他们试图解决什么问题？

“事件”可能是在我们的系统中发生重要事件之后创建的，例如执行某些指令。在我们的分布式系统中，假设我们有一个中央管理机构希望知道两个事件中哪一个先发生。这些事件发生在运行在两台不同机器上的进程上。首先的想法可能是让两个进程在将事件转发给中央管理机构之前将当前时间戳附加到事件上。理论上这可能有效，但问题在于时间戳是由运行在两台不同机器上的物理芯片派生的。即使只是使它们同步也很困难；例如使用NTP，它将带有一些误差。假设操作员没有错误地设置错误的时间，它们仍然受到称为“时钟漂移”的影响，这将导致它们随着时间的推移不同步。

首先，论文试图回答这个问题：我们能否找出哪个事件先发生，而不使用物理时钟？正如我们将看到的那样，答案*并非总是*。论文的第二部分通过引入物理时钟加强了第一部分的属性。

## “先于发生”的关系是什么？

如果我们在一个过程中可视化一系列事件，则如果“$a$”在“$b$”*先于发生*，那么“$a$”将在“$b$”之前显示。

让我们用符号"$\rightarrow$"来定义"*先于发生*"关系：

1.  如果$a$和$b$是同一个过程中的事件，并且$a$发生在$b$之前，则$a \rightarrow b$。

1.  如果$a$是一个进程发送的消息$m$，$b$是另一个进程接收的$m$，那么$a \rightarrow b$。

1.  如果$a \rightarrow b$并且$b \rightarrow c$，那么$a \rightarrow c$。两个不同的事件$a$和$b$被称为并发事件，如果$a \nrightarrow b$且$b \nrightarrow a$。

假设对于任何事件 $a$，都有 $a \nrightarrow a$，这个关系定义了事件的非自反偏序。

上一段中的“不总是”的概念源于两个并发事件，我们无法确定哪一个先发生。消息交换是一个同步点。

## 那么逻辑时钟是什么呢？

每个进程在其本地状态中有一个数字，可以被认为是当前时钟。时钟在每个事件后递增，并且每个事件都有其自己的时间戳。这样，如果 $a \rightarrow b$，那么 $a$ 的时间戳将比 $b$ 小。这个概念通过Lamport所称的时钟条件得到了形式化。

在进程 $P_i$ 中，我们有一个函数 $C_i$ 将时间戳编号 $C_i\langle a \rangle$ 分配给事件 $a$。然后时钟条件被定义为：

$$对于\ 任何\ 事件\ a,b:\ 如果\ a \rightarrow b\ 那么\ C\langle a \rangle < C\langle b \rangle$$

同样有趣的是反过来并不一定成立。如果 $C\langle a \rangle < C\langle b\rangle$，可能 $a$ 和 $b$ 是两个并发事件（同时运行）。

总结一下，逻辑时钟可以被认为是一种在事件上添加时间戳而不使用物理时钟的方法。

那么我们如何在实践中应用它到我们的算法中呢？文章提出了两条规则：

1.  每个进程 $P_i$ 在两个连续事件之间递增 $C_i$。

1.  如果事件 $a$ 由进程 $P_i$ 发送了消息 $m$，那么消息 $m$ 包含时间戳 $T_m = C \langle a \rangle$。收到消息 $m$ 后，进程 $P_i$ 将 $C_i$ 设置为不小于其当前值且大于 $T_m$。

## 如何获得一个总排序？

Lamport 提出了一种将 → 关系任意扩展到表示总序的 ⇒ 关系的方法。为了解决冲突，每个进程被分配一个唯一的数字标识。在事件排序时，首先按时间戳排序。如果时间戳相等，则按进程标识排序。这将得到一个总排序。

正如提到的，这是一种任意打破冲突的方式。它引发了用户对时间流逝的感知和系统对时间流逝的感知之间的一些异常。文章解释了用户可能向某个服务发送请求，打电话给他们的朋友并告诉他们发送相同的请求，第二个请求可能具有较低的时间戳。

接下来，我们只有两种方式。或者，在打电话给他们的朋友时，第一个用户提供了附加到其请求的时间戳，第二个用户也附加了它们的请求;或者，我们需要一个能满足以下**强**时钟条件的时钟系统：

$$对于\ 任何\ 事件\ a,b:\ a \rightarrow b\ \equiv C\langle a \rangle < C\langle b \rangle$$

这是说如果$C\langle a \rangle < C\langle b\rangle$，这意味着a发生在b之前。一般来说，逻辑时钟不满足强时钟条件。本文的其余部分致力于使用物理时钟创建能够满足强时钟条件的新时钟系统。

有趣的是，通过使用向量时钟可以实现这种时钟系统的解决方案，而不使用物理时钟。向量时钟使用时钟向量而不是像Lamport时钟那样简单的标量来跟踪时间。在实际系统中，如Google的TrueTime和混合时钟中使用物理时钟。

## 我们如何使用Lamport时钟解决互斥问题？

我想写的最后一部分是关于如何在实践中使用Lamport时钟解决互斥问题的实际例子。该问题的一个示例是当两个进程需要访问共享资源（如磁盘）时。

让我们列出一些规则：

1.  被授予资源的进程必须在被授予给另一个进程之前释放它。

1.  必须按照它们发出的顺序授予资源的不同请求。

1.  如果每个被授予资源的进程最终都释放它，那么每个请求最终都会被授予。

所提出的算法是分散式的（没有一个节点比其他节点更重要），并且不涉及故障处理。

每个进程的本地状态包括：

为简单起见，每个进程都可以向其他每个进程发送消息，消息不会丢失，并按发送顺序到达。

该算法使用六个动作解决互斥问题。每个动作映射到导致本地时间戳增加的事件：

1.  要请求资源，进程$P_i$向每个其他进程发送消息$<<RequestResource,\ P_i,\ T_m>>$，并将该消息放入其自己的请求队列中，其中$T_m$是消息的时间戳。

1.  当进程$P_j$接收到消息$<<RequestResource,\ P_i,\ T_m>>$请求资源时，(i) 它将其放置在其请求队列中，并(ii) 向$P_i$发送一个（带时间戳的）确认消息。

1.  每当$P_i$收到$AckRequestResource$时，它将其附加到其本地的确认集合中。

1.  要释放资源，进程$P_i$从其请求队列中删除任何$<<RequestResource,\ P_i,\ T_m>>$消息，并向每个其他进程发送一个带时间戳的$P_i$ $ReleaseResource$消息。

1.  当进程$P_j$接收到$P_i$ $ReleaseResource$消息时，它从其请求队列中删除任何$<<RequestResource,\ P_i,\ T_m>>$请求资源消息。

1.  进程$P_i$在满足以下两个条件时被授予资源：(i) 在其请求队列中有一个$<<RequestResource,\ P_i,\ T_m>>$，该请求在关系$⇒$下排在其队列中的任何其他请求之前。（为了定义消息的关系"⇒"，

    我们将一个消息与发送它的事件进行了标识。 (ii) $P_i$已接收到每个时间戳晚于Tm的其他进程的消息。

### TLA+规范

基于这份蓝图，我用TLA+和PlusCal编写了一个更正式的规范。我会在这里进行简短的介绍，只涵盖有趣的部分，但你可以在[这里](https://github.com/FedericoPonzi/tla-plus-specs/)找到完整的规范。

```
---- MODULE lamport_clock ----

CONSTANT Processes, ProcessCanFail, MaxTimestamp
ASSUME IsFiniteSet(Processes)
ASSUME Cardinality(Processes) > 1
ASSUME \A p \in Processes : p \in Nat
ASSUME MaxTimestamp \in Nat
ASSUME ProcessCanFail \in BOOLEAN 
```

可用的常量有：

+   `Processes`：像`{0, 1, 2}`这样的一组进程。

+   `ProcessCanFail`：一个布尔值，如果设置为true，将展示活性属性是如何被破坏的。

+   `MaxTimestamp`：为了保持模型的界限，我设置了最大可能的时间戳限制。否则，TLC不知道何时停止模型检查。

假设语句有助于基本类型检查，并确保配置正确。

```
(* --fair algorithm mutual_exclusion {

    variables resource_owner = {},
              mailbox = [p \in Processes |-> EmptyMailbox]; 

    define {
        CanSendMessage(myself) == \A p \in Processes : mailbox[p] = EmptyMailbox
				SortFunction(seq1, seq2) ==
            IF seq1.ts > seq2.ts
            THEN FALSE
            ELSE IF seq1.ts < seq2.ts
            THEN TRUE
            ELSE IF seq1.proc > seq2.proc
            THEN FALSE
            ELSE IF seq1.proc < seq2.proc
            THEN TRUE
            ELSE FALSE
        \* SAFETY: Only one process is allowed in the critical section
        Inv == Cardinality(resource_owner) <= 1 
```

资源所有者是资源当前所有者的集合，或者换句话说，当前处于其临界区的进程。

邮箱用于跨进程进行通信。它一次只允许一个消息；虽然简单，但是有效。另一种选择是在[Linda tuplespaces风格](https://en.wikipedia.org/wiki/Linda_(coordination_language))中保留所有消息。

`define`块包含一些有用的定义。 `SortFunction`用于创建总排序。 `Inv`是用于检查只有一个进程处于临界区的安全不变式。

```
fair process (p \in Processes)
    variables local_timestamp = 0,
              ack_request_resource = {},
              requests_queue = {}; 
```

这些变量是进程的局部状态，如前一节所述。

进程体用于定义上述动作。

```
while (TRUE) {
        either { 
            (*********************
                1\. To request the resource, process Pi sends the message <<RequestsResource, Pi, Tm>> 
                    to every other process, and puts that message on its own request queue, where Tm is the timestamp of the message.
            **********************)
            await /\ CanSendMessage(self) 
                  /\ Cardinality(ack_request_resource) = 0;
            BumpTimestamp();
            requests_queue := requests_queue \union {[ msg |-> RequestResource, proc |-> self, ts |-> local_timestamp]};
            mailbox := [p \in Processes |-> IF p = self THEN EmptyMailbox ELSE [ msg |-> RequestResource, proc |-> self, ts |-> local_timestamp]];
            ack_request_resource := {[ msg |-> AckRequestResource, proc |-> self, ts |-> local_timestamp]}; 
```

进入该块的条件是：

+   所有邮箱都是空的，或者换句话说，`p`可以将消息传递给所有其他进程。

+   `p`尚未发送过资源请求。这是因为在此操作结束时，`p`已将`AckRequestResource, self, local_timestamp`添加到本地集合中。这样，可以使用ack_request_resources的基数来理解`p`是否发送了`RequestResource`，而无需保留额外的状态。避免在未获取资源的情况下发送多个请求只是简化规范的一种方式。

```
(*********************
2\. When process Pj receives the message <<RequestsResource, Pi, Tm>> requests resource, 
    (i) it places it on its request queue and 
    (ii) sends a (timestamped) acknowledgment message to Pi.
**********************)
await mailbox[self].msg = RequestResource;
requests_queue := requests_queue \union {mailbox[self]};
BumpTimestampO(mailbox[self].ts);

\* Respond with an Ack
await mailbox[mailbox[self].proc] = EmptyMailbox; 
mailbox[mailbox[self].proc] := IF ProcessCanFail /\ \A proc \in Processes \ {self}: proc > self \* if process can fail and this is the smallest pid
                                THEN EmptyMailbox
                                ELSE [ msg |-> AckRequestResource, proc |-> self, ts |-> local_timestamp];
EMPTY_MAILBOX:
mailbox[self] := EmptyMailbox;
BumpTimestamp(); 
```

邮箱是一个“全局”映射变量。我可以使用`mailbox[self]`来访问p的分配邮箱。因为消息的形式是`msg, proc, ts`，所以我可以使用`mailbox[mailbox[self].proc]`来访问发送者的邮箱。

如果将`ProcessCanFail`设置为true，最小进程`p`将永远不会发送确认消息，模拟一个故障。这会停止系统并破坏活性属性。

```
(*********************
    5\. When process Pj receives a Pi ReleasesResource message, it removes any <<RequestsResource, Pi, Tm>> 
       requests resource message from its request queue.
**********************)
await self \in resource_owner  /\ CanSendMessage(self);
requests_queue := SeqToSet(Tail(SetToSortSeq(requests_queue, SortFunction))); 
BumpTimestamp();
mailbox := [p \in Processes |-> IF p = self THEN EmptyMailbox ELSE <<ReleaseResource, self, local_timestamp>>];
resource_owner :=  resource_owner \ {self};
ack_request_resource := {}; 
```

要从`requests_queue`中移除`RequestResource`，只需使用排序函数对请求进行排序并丢弃结果的头部即可。这是安全的，这要归功于我们假设没有丢失消息。

```
(*********************
6\. Process Pi is granted the resource when the following two conditions are satisfied: 
    (i) There is a <<RequestsResource, Pi, Tm>> in its request queue which is ordered before any other 
        request in its queue by the relation ⇒. (To define the relation " ⇒ " for messages, we identify a message
        with the event of sending it.).
    (ii) Pi has received a message from every other process timestamped later than Tm.
**********************)
await /\ Cardinality(requests_queue) > 0 
      /\ Head(SetToSortSeq(requests_queue, SortFunction)).proc = self
      /\ Cardinality(ack_request_resource) = Cardinality(Processes)
      /\ self \notin resource_owner;

resource_owner := resource_owner \union {self}; 
```

现在，每个`p`可以在本地检查它们是否被允许访问其临界区域。`self \notin resource_owner`只是为了减少状态更改的数量。由于`resource_owner`只是一个集合，重新进入此操作不会有任何坏处。

```
MaxtimestampConstraint == \A proc \in Processes: local_timestamp[proc] < MaxTimestamp 
```

此定义可用作状态约束，以保持模型的边界性。有了这个，我可以在 cfg 文件中添加：

```
CONSTRAINTS 
    MaxtimestampConstraint 
```

现在，当此约束条件返回TRUE时，TLC将停止。这种方法在这里有详细描述：[https://learntla.com/topics/unbound-models.html#topic-unbound-models](https://learntla.com/topics/unbound-models.html#topic-unbound-models)

```
\* LIVENESS: either we eventually get ownership, or we're at the edge of the timestamp boundary.
ResourceAcquisition == \A proc \in Processes : Cardinality(ack_request_resource[proc]) > 0 ~> \/ proc \in resource_owner 
                                                                                              \/ local_timestamp[proc] = MaxTimestamp 
```

正如我们所见，当`p`发送`RequestResource`消息时，它将在本地队列中附加一个确认。活性属性表明，对于任何请求访问的进程，最终它们将获得资源的所有权。我必须排除我们耗尽时间戳的情况，因为在这种情况下，系统无法进一步进行。这意味着我还必须从配置中禁用`CHECK_DEADLOCK`参数。

这就完成了整个演示，你可以在[GitHub](https://github.com/FedericoPonzi/tla-plus-specs/)上找到完整的规范。有趣的是，在规范符合属性之后，我能够通过删除不必要的状态和检查来重构它，从而导致了更短、更简单的规范；采用了“测试驱动开发”的风格。

一个可能的改进是在单独的规范中指定互斥问题，并使用细化来检查该算法是否解决了该问题。

## 结论

在本文中，我概述了 Lamport 在他的论文*[Time, Clocks, and the Ordering of Events in a Distributed System by Leslie Lamport](http://lamport.azurewebsites.net/pubs/time-clocks.pdf)*中介绍的逻辑时钟。

逻辑时钟可以用于在分布式系统中一组事件之间创建部分排序。关系“发生在之前”能够通过使用因果关系而不是依赖于物理时钟来定义逻辑时间的流逝。物理时钟不准确，并且在不同机器和长距离上保持同步非常困难。

为了打破关系并实现事件的总排序，Lamport 建议按时间戳和进程ID排序。这在某种意义上是任意的，因为我们定义了并发事件的排序，这可能与实际时间排序不同。这可能导致异常情况。为了摆脱这些异常，我们需要一个满足强时钟条件的时钟系统。

我认为实际示例非常有用，可以帮助理解如何在实践中使用 Lamport 时钟。本文介绍了一个用于解决互斥问题的 TLA+ 规范。TLA+ 规范是一种非常好玩的方式，可以在不必担心无关细节的情况下玩耍算法。

在编写本文的规范时，我还通过[贡献](https://github.com/hwayne/learntla-v2/pull/78)一些文档到 learntla 网站。

## 参考文献
