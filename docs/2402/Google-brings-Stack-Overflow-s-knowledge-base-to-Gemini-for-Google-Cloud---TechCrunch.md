<!--yml

category: 未分类

date: 2024-05-29 13:28:33

-->

# 谷歌将 Stack Overflow 的知识库引入 Gemini 用于 Google Cloud | TechCrunch

> 来源：[https://techcrunch.com/2024/02/29/google-brings-stack-overflows-knowledge-base-to-gemini/](https://techcrunch.com/2024/02/29/google-brings-stack-overflows-knowledge-base-to-gemini/)

开发者问答网站 Stack Overflow 今天推出了一个新的计划，通过一个名为 OverflowAPI 的新 API，允许 AI 公司访问其知识库。这个计划的首个合作伙伴是谷歌，谷歌将利用 Stack Overflow 的数据丰富 Gemini for Google Cloud，并在 Google Cloud 控制台中提供经过验证的 Stack Overflow 答案。与此同时，Stack Overflow 将与谷歌合作，为其平台引入更多的 AI 功能，这个过程已经在去年与 [OverflowAI 的推出](https://stackoverflow.blog/2023/07/27/announcing-overflowai/) 中开始。

谷歌和 Stack Overflow 打算在四月份的 Google Cloud Next 大会上展示这些集成。

众所周知，像 Stack Overflow 这样的内容驱动服务（还有 [Reddit](https://arstechnica.com/ai/2024/02/reddit-has-already-booked-203m-in-revenue-licensing-data-for-ai-training/)、[出版社](https://techcrunch.com/2023/12/13/openai-inks-deal-with-axel-springer-on-licensing-news-for-model-training/) 等）希望确保当大型语言模型摄取他们的数据时能得到报酬。虽然谷歌和 Stack Overflow 没有讨论这种合作的财务条款，但值得注意的是，这并不是一种独家合作。

“我们在与谷歌思考的方式非常特定，适合解决我们想为用户解决的所有问题，”Stack Overflow CEO Prashanth Chandrasekar 告诉我。“其他公司——我们收到了各种各样的入站请求，来自各种各样的公司，他们正在尝试用 LLM 训练、AI 产品进行实验——云公司和非云公司正在试图成为云公司——所有试图以非常非常强大的方式利用我们的数据的人。这个程序，这个 OverflowAPI 程序，绝对可以供所有合作伙伴与我们合作。”

所有开发者都可以从 AI 聊天机器人获取答案的世界，然而，也是很少有开发者会去 Stack Overflow 网站提问和回答问题（以及将它们复制粘贴到他们的代码中）的世界。“我们想要出现在开发者所在的任何地方，”当我询问他时，Chandrasekar 告诉我。尽管他承认他相信开发者的工作流将因这些 AI 工具而改变，但他仍然认为需要一个可信的经过验证的答案知识库。他说，这里的最终愿景是“让人类和 AI 联合起来”，确保开发者可以信任这些 AI 工具的答案，因为它们来自于由专业领域专家创建的知识库。

还值得注意的是，这不仅仅是关于人工智能。Google 还将把 Stack Overflow 直接引入 Google Cloud 控制台，让开发者可以从那里查看答案并提问问题。

“你可以设想进入 Google Cloud 控制台，输入一个查询，除了所有与 Google 相关的响应之外，你将看到 Stack Overflow 特定的响应，” Google Cloud 的开发者体验副总裁 Gabe Monroy 解释道。“从开发者体验的角度来看，这两者将开始合并在一起。现在，这真的很重要，因为这意味着开发者会得到一个流畅的体验。他们不必四处寻找和点击不同的网站。他们想要的一切，Stack Overflow 的问题和答案，以及 Google Cloud 特定的问题和答案，都在同一个地方。”

他还指出，Gemini 的答案将包括引用，因此开发者可以检查结果的正确性。

在 Stack Overflow 方面，使用 Google 的 Vertex AI 平台通过 Gemini 的想法。目前，团队正在评估这将会是什么样子，但你可以想象在问题提问和管理过程中的人工智能支持，以及作为可以在网站上回答问题的助手的形式。

Stack Overflow 从拥有庞大的专家用户群和十多年的问题和答案积累中获取其价值（而且尽管 Stack Overflow 还运行一系列类似的关于其他主题的网站，但目前重点是其旗舰面向开发者的网站）。Chandrasekar 指出，确保这种质量保持高水平，并且不会被低质量、由人工智能生成的答案所稀释，非常重要。部分原因在于 Google 正在整合 Stack Overflow 平台的人为因素。

“我们希望保持高质量。它应该是质量和准确性方面的最佳选择，” 他说。他认为，对于许多开发者来说，提问的门槛将会更低，因为他们将与 Gemini 互动，而不是与一个个性强烈的程序员小组互动。“你能得到两个世界中最好的一点，这太棒了，” 他说。

Google 的 Monroy 同样强调了在所有这些中人为因素的重要性。“正如 Stack [Overflow] 团队希望利用 Gemini 推出新功能一样，确保它不会破坏 Stack Overflow 多年来为开发者社区服务的美好和纯粹……这是神圣的。”

在长期来看，Monroy 表示，Google 也可能利用这种合作关系来增强其当前称为 Codey 的代码完成模型。
