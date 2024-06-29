<!--yml

category: 未分类

date: 2024-05-27 13:26:47

-->

# TheFastest.ai

> 来源：[https://thefastest.ai](https://thefastest.ai)

定义

===========

+   **模型：** 使用的语言模型。

+   **TTFT：** 第一个令牌时间。这决定了模型处理传入请求并开始生成文本的速度，直接影响 UI 开始更新的速度。较低的值 = 较低的延迟/更快的性能。

+   **TPS：** 每秒令牌数。这决定了模型生成文本的速度以及完整响应在 UI 中显示的速度。较高的值 = 更高的吞吐量/更快的性能。

+   **总计：** 从请求开始到响应完成的总时间，即最后一个令牌已生成。总时间 = TTFT + TPS * Tokens。较低的值 = 较低的延迟/更快的性能。

方法论

===========

**分布式足迹：** 我们每天在多个数据中心使用 [Fly.io](https://fly.io/docs/reference/regions/) 运行我们的工具。目前我们在 cdg、iad 和 sea 运行。

**连接预热：** 执行预热连接以消除任何 HTTP 连接设置延迟。

**TTFT 测量：** TTFT 时钟在发出 HTTP 请求时启动，并在响应流中接收到第一个令牌时停止。

**最大令牌数：** 输出令牌数设为 20（~100 个字符），这是典型对话句子的长度。

**尝试 3, 保留 1：** 对每个提供程序，执行三次单独的推断，并保留最佳结果（以消除由于排队等原因的任何离群值）。

来源

======

**原始数据：** 所有数据都在这个公共 [GCS 存储桶](https://storage.googleapis.com/thefastest-data) 中。

**基准测试工具：** 完整的测试套件在 [ai-benchmarks repo](https://github.com/fixie-ai/ai-benchmarks) 中可用。

**网站：** 此网站的完整源代码位于 [GitHub](https://github.com/fixie-ai/fastest.ai) 上。
