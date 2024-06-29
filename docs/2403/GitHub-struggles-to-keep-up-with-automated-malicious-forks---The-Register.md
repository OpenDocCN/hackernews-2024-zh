<!--yml

category: 未分类

date: 2024-05-27 14:32:12

-->

# GitHub难以跟上自动化恶意分叉 • The Register

> 来源：[https://www.theregister.com/2024/03/01/github_automated_fork_campaign/](https://www.theregister.com/2024/03/01/github_automated_fork_campaign/)

从去年五月开始的恶意软件分发活动，最初上传到Python包索引（PyPI）的几个恶意软件包已扩展到GitHub，并扩展到至少100,000个受感染仓库。

根据安全公司Apiiro的说法，这场污染代码的活动包括克隆合法仓库，感染它们以恶意软件加载程序，并将修改后的文件上传到GitHub上的同名仓库，然后将受感染的代码分叉数千次，并在论坛和社交媒体渠道上推广被篡改的代码。

因此，寻找有用代码的开发人员可能会发现一个被描述为有用的仓库，乍一看似乎合适，但实际上却会通过运行恶意Python代码和二进制可执行文件的隐藏有效载荷来窃取其个人数据。

“这种恶意代码（主要是修改版的[BlackCap-Grabber](https://github.com/Inplex-sys/BlackCap-Grabber-NoDualHook)）随后将从不同应用程序收集登录凭据、浏览器密码和cookie以及其他机密数据，” 安全研究员Matan Giladi和AI负责人Gil David在[报告](https://apiiro.com/blog/malicious-code-campaign-github-repo-confusion-attack/)中说道。“然后将其发送回恶意行为者的C&C（命令与控制）服务器，并执行一系列其他恶意活动。”

趋势微分析报告描述了恶意代码的分析方法，说明其采用了巧妙的技术来掩盖其真实性质。例如，该代码通过称为“exec走私”的技术隐藏了其对exec函数的使用，用于动态执行代码。

此类攻击增加了数百个空格字符（共521个），以将exec函数推至屏幕之外，作为防止手动审查的防御措施。

GitHub表示，他们意识到情况并非都很好。

“GitHub托管着超过1亿开发人员，涵盖超过4.2亿个仓库，致力于为开发人员提供安全的平台，” 发言人告诉*The Register*。

“我们有专门团队负责检测、分析和删除违反我们[可接受使用政策](https://docs.github.com/en/site-policy/acceptable-use-policies/github-acceptable-use-policies#5-site-access-and-safety)的内容和帐户。我们采用手动审核和规模化检测，利用机器学习不断演进和适应对抗性战术。我们还鼓励客户和社区成员[举报滥用和垃圾信息](https://docs.github.com/en/communities/maintaining-your-safety-on-github/reporting-abuse-or-spam)。”

意识和自动扫描都很好，但是Apiiro的Giladi和David观察到，GitHub错过了许多自动存储库的分支，以及手动上传的存储库。

“由于整个攻击链似乎大部分是自动化的大规模操作，即使只有百分之一的恶意存储库幸存下来，也相当于成千上万个。” 作者写道，补充道，如果算上被删除的存储库，这次活动可能涉及数百万个恶意克隆和分支。

他们还指出，攻击规模足够大，可以从网络效应中受益，具体来说，开发者会复制恶意存储库，而不是真的使用该软件，他们没有意识到自己在验证和传播恶意软件。

研究人员称，GitHub由于支持自动生成账户和存储库、友好的API和软速率限制以及其规模，成为妥协软件供应链的有效途径。

拜登政府通过国家标准与技术研究院的[网络安全框架2.0](https://www.theregister.com/2024/02/27/nist_cybersecurity_framework_2/)和努力推动组织公布其[软件材料清单](https://www.theregister.com/2023/03/05/sboms_supply_chain_security/)来推动更强的软件供应链安全性。但显然还有工作要做。
