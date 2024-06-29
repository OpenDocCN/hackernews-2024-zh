<!--yml

category: 未分类

date: 2024-05-27 14:49:03

-->

# Nvidia 的 Chat with RTX 是一个有前途的AI聊天机器人，可以在您的个人计算机上本地运行 - The Verge

> 来源：[https://www.theverge.com/2024/2/13/24071645/nvidia-ai-chatbot-chat-with-rtx-tech-demo-hands-on](https://www.theverge.com/2024/2/13/24071645/nvidia-ai-chatbot-chat-with-rtx-tech-demo-hands-on)

Nvidia 今天发布了早期版本的 [Chat with RTX](https://go.redirectingat.com/?xs=1&id=1025X1701640&url=https%3A%2F%2Fwww.nvidia.com%2Fen-us%2Fai-on-rtx%2Fchat-with-rtx-generative-ai%2F)，这是一个演示应用程序，允许您在个人计算机上运行个人AI聊天机器人。您可以为它提供YouTube视频和您自己的文档，以便创建摘要并根据您自己的数据获得相关答案。所有操作均在PC上本地运行，您只需使用至少8GB VRAM的RTX 30或40系列GPU。

我已经开始简要测试Chat with RTX了，虽然这款应用程序还有些粗糙，但我已经可以看到它对于新闻工作者或任何需要分析文档集合的人来说，已经是一个有价值的数据研究部分。

Chat with RTX 能够处理YouTube视频，所以您只需输入URL，它就能让您搜索特定提及的剧本或总结整个视频。我发现这对于搜索视频播客非常理想，尤其是在过去一周中，关于微软新的 [Xbox战略转变](/2024/2/12/24067370/microsoft-xbox-playstation-switch-games-future-hardware) 的传闻中寻找特定提及。

不过，用于搜索YouTube视频时并不完美。我尝试通过 *Verge* YouTube视频的剧本进行搜索，但是Chat with RTX下载了完全不同视频的剧本。这甚至不是我之前查询过的视频，所以这个早期演示显然存在缺陷。

*Chat with RTX 擅长搜索您的本地文档。*

Tom Warren / The Verge 的截图

当它正常工作时，我能够在几秒钟内找到视频中的参考信息。我还为Chat with RTX创建了一个 *FTC v. Microsoft* 文件数据集。去年我报道法庭案件时，要快速搜索文档常常让人感到不知所措，但是Chat with RTX帮助我在PC上几乎即时地进行了查询。

例如，聊天机器人很好地总结了微软整个Xbox Game Pass战略的内部文件：

> 根据提供的上下文信息，Xbox Game Pass 是一项游戏内容订阅服务，用户可以通过支付单一月费来访问游戏库。它被设想为一种平台，使玩家能够在任何地方玩游戏，并允许发行商触达全球玩家。该服务由 Project xCloud 和 Xbox Series X 支持，旨在为发行商创造一个赚钱的好机会。Gaming CSA 的主要战略目标是扩展 Xbox Game Pass，在全球范围内有着可观的订阅者机会，预计可能达到7.5亿订阅者。该服务面临三个主要的内容动态，包括需求差异化内容、扩展到主机以外以及有限的内容供应。

我还发现这对于扫描 PDF 和事实核查数据非常有用。Microsoft 自己的 Copilot 系统在 Word 内部无法很好地处理 PDF，但 Nvidia 的 Chat with RTX 没有问题地提取出所有关键信息。响应速度也几乎是即时的，没有像使用基于云的 ChatGPT 或 Copilot 聊天机器人时通常看到的延迟。

Chat with RTX 的一个显著缺点是，它*真的*感觉像是早期开发者演示。Chat with RTX 本质上在您的 PC 上安装了一个 Web 服务器和 Python 实例，然后利用 Mistral 或 Llama 2 模型来查询您提供的数据。然后利用 Nvidia 的 Tensor 核心在 RTX GPU 上加速您的查询。

*虽然 Chat with RTX 不总是准确的。*

Tom Warren / The Verge 提供的截图

在我的装有 Intel Core i9-14900K 处理器和 RTX 4090 GPU 的 PC 上，Chat with RTX 大约花了30分钟来安装。该应用程序的大小接近40GB，Python 实例占用了我系统中可用的64GB内存中的大约3GB。一旦运行起来，您可以从浏览器访问 Chat with RTX，同时命令提示符在后台运行，输出正在处理的内容和任何错误代码。

Nvidia 并不将此作为所有 RTX 用户都应立即下载和安装的完美应用程序。存在许多已知的问题和限制，包括来源归因并不总是准确的。我最初还试图让 Chat with RTX 索引25000份文档，但这似乎导致应用程序崩溃，我不得不清除偏好设置才能继续进行。

Chat with RTX 也不记住上下文，因此后续问题不能基于先前问题的上下文。它还在您要求它索引的文件夹内创建 JSON 文件，因此我不建议在 Windows 的整个文档文件夹上使用它。

我喜欢一个精彩的技术演示，尽管如此，Nvidia 在这里确实交出了一份满意的答卷。它展示了 AI 聊天机器人在未来可以在您的个人电脑上本地运行的潜力，特别是如果您不想订阅像[Copilot Pro](/2024/1/15/24038711/microsoft-copilot-pro-office-ai-apps)或者[ChatGPT Plus](/2023/10/29/23937497/chatgpt-plus-new-beta-all-tools-update-pdf-data-analysis)这样的服务，仅仅是为了分析您的个人文件。
