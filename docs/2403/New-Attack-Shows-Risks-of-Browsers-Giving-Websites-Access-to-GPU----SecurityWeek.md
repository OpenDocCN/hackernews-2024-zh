<!--yml

category: 未分类

日期：2024-05-27 15:04:17

-->

# 浏览器授予网站GPU访问权限的风险的新攻击 - SecurityWeek

> 来源：[https://www.securityweek.com/new-attack-shows-risks-of-browsers-giving-websites-access-to-gpu/](https://www.securityweek.com/new-attack-shows-risks-of-browsers-giving-websites-access-to-gpu/)

**来自奥地利格拉茨技术大学和法国雷恩大学的研究团队展示了一种影响多个流行浏览器和显卡的新图形处理单元（GPU）攻击。**

研究集中在WebGPU上，这是一个API，允许Web开发人员在Web浏览器中使用底层系统的GPU进行高性能计算。通过利用这个API，他们展示了一种完全依赖于JavaScript的从Web浏览器进行的攻击。这使得远程执行变得更加容易，但与需要访问本机GPU API的先前攻击相比，潜在影响也受到了限制。

学术研究人员描述他们的工作为浏览器内部的首次GPU缓存侧信道攻击之一。展示了该方法如何利用于远程攻击，方法是让目标用户访问一个托管恶意WebGPU代码的网站，并在执行利用期间使其停留数分钟。

例如，利用漏洞可以在受害者阅读恶意网站上的文章时执行。进行攻击不需要任何其他类型的用户交互。

专家们展示了这种新方法可以用于间击键定时攻击，这可以根据击键定时数据推断敏感信息，如密码。它还可以用于在几分钟内获取基于GPU的AES加密密钥，以及具有高达10 Kb/s传输速率的隐蔽数据外泄通道。

“我们的工作强调，浏览器供应商需要像对待其他安全和隐私相关资源一样对待对GPU的访问权限”，研究人员指出。

参与该项目的研究人员之一Lukas Giner告诉*SecurityWeek*，尽管他们展示的攻击并非“极其强大”，但确实展示了浏览器允许任何网站访问主机系统显卡的潜在风险，而无需专门请求权限。

“这可能导致像我们这样的隐秘攻击（或者未来可能更糟的攻击），或者网站仅仅使用GPU进行诸如加密货币挖掘之类的事情，而用户完全不知情”，Giner解释道。

广告。继续阅读。

研究针对11种桌面显卡进行了测试：AMD的两款RX系列产品以及NVIDIA的九款GTX、RTX和Quadro系列产品。攻击目标支持WebGPU的浏览器，包括Chrome、Chromium、Edge和Firefox Nightly。

“通过针对 Web 浏览器，我们的威胁模型包括浏览器在处理敏感信息时可能运行的任何场景。由于整个系统通常共享 GPU，这可能包括任何渲染的内容（例如网站或应用程序）和通用计算操作，” 研究人员在详细介绍其工作的一篇[论文](https://ginerlukas.com/publications/papers/WebGPUAttacks.pdf)中写道。

Mozilla、AMD、NVIDIA 和 Chromium 开发者已经收到通知。AMD 发布了一个[公告](https://www.amd.com/en/resources/product-security/bulletin/amd-sb-6011.html)，称“研究人员未展示出对 AMD 产品的任何利用”。

研究人员表示其他公司都没有计划采取任何行动。

Giner 表示他们建议在浏览器中添加一个权限弹出窗口，类似于请求麦克风或摄像头访问权限的窗口。然而，Chromium 团队表示，他们发现要求用户做出他们不理解后果的安全决策会增加摩擦而不会使用户更安全。

已发布一个小的概念验证 ([PoC](https://ginerlukas.com/gpuattacks/))。它显示是否可用 WebGPU 并在浏览器中进行无害攻击。

**更新以重新表达 Chromium 团队的回应。同时更新第三段以澄清这是从浏览器内部进行的第一次 GPU 缓存侧信道攻击，而不是这类攻击的第一个案例。这是在撰写论文时的第一次攻击，但其他人已经进行了[类似的研究](https://arxiv.org/pdf/2401.04349.pdf)并针对不同目标。*

**相关**: [新的 GPU 侧信道攻击允许恶意网站窃取数据](https://www.securityweek.com/new-gpu-side-channel-attack-allows-malicious-websites-to-steal-data/)

**相关**: [AI 数据通过漏洞的 AMD、Apple、Qualcomm GPU 遭受 “LeftoverLocals” 攻击](https://www.securityweek.com/ai-data-exposed-to-leftoverlocals-attack-via-vulnerable-amd-apple-qualcomm-gpus/)
