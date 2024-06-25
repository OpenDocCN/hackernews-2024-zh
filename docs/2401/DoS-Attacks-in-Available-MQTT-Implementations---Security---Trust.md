<!--yml

类别：未分类

日期：2024-05-27 15:11:34

-->

# 可用 MQTT 实现中的 DoS 攻击 - 安全与信任

> 来源：[`st.fbk.eu/complementary/IOTSECFOR2021.html`](https://st.fbk.eu/complementary/IOTSECFOR2021.html)

# 补充材料

## 幻灯片、测试结果和配置可在[此处](https://drive.google.com/drive/folders/1YOGKlGiSuHcViDgV3p-7D_QEu2qNY1ck?usp=sharing)获得。

## 演示在 *ARES & CD-MAKE 会议*的 [Youtube 频道](https://www.youtube.com/watch?v=5xhcJVmdJSI)上。

## 摘要

物联网是一种被广泛采用的技术，但也是最方便攻击的技术之一，因为共享数据的量大，同时易受攻击的廉价但不安全的产品也很多。在大多数情况下，如果攻击者无法利用安全漏洞和隐私问题来外泄数据，他们可以通过向后端、连接的客户端或外部服务执行拒绝服务（DoS）攻击来损害服务。

在这项工作中，我们调查了针对 MQTT 消息队列处理的两类 DoS 攻击，MQTT 是最广泛使用的物联网协议之一；有关 MQTT 及其功能的介绍，请参阅[此处](https://www.hivemq.com/mqtt-protocol/)。

+   第一种攻击尝试通过在不同主题上发送许多重消息来饱和 MQTT 代理资源，无论是否有一组已订阅的客户端。这是因为代理为每个已订阅的客户端存储所有消息（最多达到队列限制），并且会损害服务，因为代理是单点故障。如果队列的使用（与已订阅的客户端相关联）影响代理资源（磁盘/RAM/CPU），或者如果托管代理的机器达到 CPU 或网络带宽的 100％ 使用率，则攻击将成功。

+   第二种攻击可以描述为 MQTT 上下文中的一种[放大攻击](https://www.radware.com/security/ddos-knowledge-center/ddospedia/amplification-attack)，它规定定期发送重型消息以向连接到代理的资源受限客户端发起 DoS 攻击。如果客户端被阻止接收其他/所有消息并使用 CPU 资源继续尝试接收重型消息，则攻击将成功。值得一提的是，如果具有持久会话的客户端被强制断开连接，则代理将队列所有新消息（最多达到配置的限制），并在重新连接时再次尝试转发；包括影响客户端 CPU 的重型消息。

我们使用两个测试平台和三个开源 MQTT 代理实现来研究攻击的有效性：[Mosquitto](https://www.mosquitto.org) v.1.6.9、[VerneMQ](https://vernemq.com) v.1.11.0 和 [EMQ X](https://www.emqx.io/products/broker) v.4.1.5。为了模拟客户端，我们使用了 MQTT Python 库 [Paho](https://www.eclipse.org/paho)。

### 对测试中的 MQTT 代理的 DoS 攻击结果

+   所有代理的 RAM 使用都受到消息发布的影响：

    +   在 Mosquitto 中，只需一个恶意连接的客户端即可使 RAM 和交换空间饱和（并被系统杀死）； 在 VerneMQ 和 EMQ X 中，需要使用多个并发客户端。 但是，由于有效载荷大小（默认情况下）的限制，攻击 EMQ X 需要更长的时间。

+   在 VerneMQ 中，可以攻击磁盘，因为它（默认情况下）在磁盘上存储接收的消息。

+   为了攻击 CPU 使用率，我们使用了多达 400 个并发客户端（400 个发布者，在第二次测试运行中为 400 个订阅者）。 当还使用订阅者（它们以一定的服务质量连接，并在发布开始之前断开连接）时，仅在 EMQ X 中攻击成功：从 250 个并发发布者开始，CPU 使用率平均超过 80%（峰值使用率为 93%）。 在最糟糕的情况下，Mosquitto 增加了 7.75%，而 VerneMQ 增加了 14.4%。

    +   在测试期间，Mosquitto 将只使用一个 CPU 核心进行单线程处理； VerneMQ 和 EMQ X 将在同一 CPU 级别同时使用所有核心，变化极小。

### 放大攻击的结果

+   使用 TLS 强烈影响最大有效载荷大小和处理时间：具有 255MB 有效载荷（最大支持）的消息仅在明文配置下接收，并且使用 TLS 时的处理时间约为十倍。

+   连接的客户端无法同时接收其他消息，并且它会长时间使用一个 CPU 核心。 这使得攻击者可以通过定期发送一个最大支持大小的消息来阻止客户端接收（可能重要的）消息或使其过热。

+   在性能方面，TLS 1.2 和 1.3（在 Mosquitto 的情况下）、MQTT v.3.1.1 和 v.5.0，或者如果使用 ACL 作为授权机制，没有任何差异。

+   订阅者接收排队消息所需的平均时间呈线性增加。

为了提高基于 MQTT 的部署的安全意识，我们将攻击和缓解措施集成到 [MQTTSA](https://github.com/stfbk/mqttsa) 中，这是一个检测 MQTT（错误）配置并提供面向安全的建议和配置片段的工具。

最后，我们确定了测试中 MQTT 代理的设置，这些设置将阻止或支持（如果配置错误）DoS 攻击。

### 与 Mosquitto、VerneMQ 和 EMQ X 中的拒绝服务攻击相关的设置

#### Eclipse Mosquitto

限制消息大小的设置：

+   *max_inflight_bytes*，默认情况下不受限制，设置正在传输的消息的总字节限制。

+   *max_packet_size*，协议规范为 250MB（尽管实际限制取决于客户端的硬件能力），设置最大包大小（头部和有效载荷）。 在 MQTT v.5 的情况下，客户端在被代理断开连接时被通知限制（带有原因代码）。

+   *message_size_limit*，默认情况下不受限制，设置 PUBLISH 消息中支持的最大有效载荷大小。

限制消息速率的设置：

+   *max_inflight_messages*，默认值为 20，设置正在传输中的最大出站 QoS 1/2 消息数量。

限制活跃连接的设置：

+   *max_connections*，默认情况下未限制但受主机操作系统限制，限制了已连接客户端的总数。

+   *persistent_client_expiration*，作为 OASIS 规范中的非标准项，默认设置为不过期，将允许在持久客户端在一定时间内不重新连接时到期其会话。

与消息队列相关的设置：

+   *max_queued_messages*，从版本 2.0 开始为 1000（在最新的 Mosquitto 1.6.x 中为 100），设置了除了正在传输中的消息之外队列中要保留的 QoS 1/2 消息的最大数量（每个客户端）。

+   *max_queued_bytes*，也是无限的，而是设置了要发送的排队 QoS 1/2 消息的阈值。

+   *queue_qos0_messages*，作为 OASIS 规范中的非标准项，默认情况下已禁用，允许将支持 QoS 1 和 2 的 QoS 0 消息排入队列。

+   *upgrade_outgoing_qos*，默认情况下禁用，将允许在订阅客户端使用更高 QoS 值时提升已发布消息的 QoS。

影响经纪人磁盘 I/O 的设置：

+   *persistence*，如果启用，则允许将内存数据库存储到磁盘上（连接、订阅和消息数据）；重新启动经纪人时，它将使用所有信息初始化。值*autosave_interval* 指定了经纪人在保存内存数据库之前等待的时间段（以秒为单位）。类似地，*autosave_on_changes* 允许在消息（接收和排队）数量和订阅更改数量超过阈值时保存。错误配置数据库的存储或设置低阈值可能会影响经纪人的性能。

+   *log_dest* 允许将日志条目存储在磁盘上，*log_type* 设置要记录的信息量：目标默认设置为*stdout*，而类型设置为*error*、*warning*、*notice*和*information*。

影响经纪人内存使用的设置：

+   *memory_limit*，默认情况下未限制，设置了经纪人使用的内存的硬限制。

防止慢速拒绝服务（Slow-DoS）攻击的设置：

+   *max_keepalive* 允许在 MQTT5 客户端的情况下覆盖客户端提供的保持活动值；从而防止了慢速拒绝服务（slow-dos）攻击的发生。

可以限制或支持 DoS 攻击的其他设置：

+   *check_retain_source*，默认启用，控制保留消息在重新发布之前的访问权限来源。这可以防止恶意客户端发布消息，其访问权限已被撤销；以及，例如，将大型保留消息重新发布给不断尝试接收并因消息大小而崩溃的客户端。

+   *retain_available* 允许禁用保留消息。

#### VerneMQ

限制消息大小的设置：

+   *max_message_size* 作为 Mosquitto 的*message_size_limit*。

+   *tcp.buffer_sizes* 允许设置内核的*发送*和*接收*缓冲区，以及基于 TCP 的连接的用户级缓冲区。

限制消息速率的设置：

+   *max_inflight_messages* 如同 Mosquitto。

+   *max_message_rate*，默认情况下不受限制，指定每个会话每秒的最大传入发布速率。

限制活动连接的设置：

+   *max_connections*，与 Mosquitto 相同，但默认值为 10,000。

+   *persistent_client_expiration* 如同 Mosquitto。

+   *allow_multiple_sessions*，默认情况下禁用，默认情况下不符合 OASIS 规范且将被废弃，允许使用相同客户端 ID 连接多个客户端。这将允许攻击者在盗用凭据的情况下连接和（误）使用服务而不触发任何警报。

与消息队列相关的设置：

+   *max_online_messages* 和 *max_offline_messages*（默认均为 1,000）设置为连接或突然断开连接的客户端队列的最大数量。

+   *upgrade_outgoing_qos* 如同 Mosquitto。

影响经纪人磁盘 I/O 的设定：

+   *log.console*和*log.console.level*等效于 Mosquitto 的*log_dest*和*log_type*。然而前者设置为存储条目在文件中。

影响经纪人内存使用的设定：

+   *nr_of_acceptors*，默认值为 10，设置等待新连接的并发进程数。该设置负责测试中所有 CPU 核心的使用。

+   Erlang 特定的设置，例如*async_threads*（设置 Erlang VM 中的异步线程数），修改运行经纪人的 Erlang 运行时系统的行为。

+   *maximum_memory.percent*，默认配置文件为 70，设置内存数据库分配的最大百分比（用于存储持久会话的连接、订阅和消息数据）。然而在我们的测试中，如果经纪人在并发发布者的重负载下，则该阈值可能被暂时超过（最多达到 85%）。

可限制或支持 DoS 攻击的其他设置：

+   *retry_interval*，默认为 20 秒，设置未确认 QoS 1/2 消息的重试间隔。这可能会影响经纪人的性能（如果阈值太低），或可能是资源受限的客户端（例如，如果经纪人每分钟尝试重发一条重消息，而客户端设置为以相同的时间连接）。

+   *max_last_will_delay* 允许指定最后遗嘱消息的最大延迟。

对于*suppress_lwt_on_session_takeover*和*receive_max_client*参数，进一步的调查可能与连接的 DoS 攻击有关，但不在文档中描述。

#### EMQ X 设置

考虑到用于根据 MQTT 协议和 Erlang 运行时环境自定义代理的参数数量，以下我们提供了 EMQ X 特有功能的高级描述。请参阅此处=misconfigurable.xlsx)以获取详尽的列表。

+   保留消息配置（默认启用）允许指定目标、到期时间（如果指示，则由 PUBLISH 数据包中的到期时间替换）以及消息或负载大小的限制。

+   速率限制允许设置配额并限制连接数及其速率、发布消息的数量以及通常的 TCP 包数量。

+   监控和报警功能支持在触发的警报数量、CPU 和内存使用率（百分比）、进程数量和默认行为（记录事件或将其报告给特定主题）上配置阈值。此外，还可以检测频繁的断开连接（参考文档中的“抖动检测”）并设置禁止间隔；通过“心跳”信号监视代理状态。

+   最后，可以设置低级设置，如线程数量、并发进程和内存限制、RPC 配置和 TCP 缓冲区。然后，可以禁用订阅中的通配符或保留消息的使用，设置 MQTT QoS 2 确认（PUBREL）的限制和超时，并配置主题优先级。

+   记录（默认为“警告”）、*max_packet_size*（默认为 1MB）、*upgrade_qos*（已禁用）、*max_inflight*（32 条消息）、*retry_interval*（30 秒）、*session_expiry_interval*（2 小时）、*max_mqueue_len*（1,000）、*mqueue_store_qos0*（**true**）、连接接受者（在 EMQ X 中特定于连接类型，并且如果在同一系统或外部，则会起作用）和 *max_connections*（1,024,000）的所有功能都与 Verne MQ 中的功能相同。
