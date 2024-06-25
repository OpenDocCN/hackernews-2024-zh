<!--yml

category: 未分类

date: 2024-05-27 15:21:47

-->

# 一个错误发布的密码是如何暴露了梅赛德斯-奔驰的源代码 | TechCrunch

> 来源：[`techcrunch.com/2024/01/26/mercedez-benz-token-exposed-source-code-github/`](https://techcrunch.com/2024/01/26/mercedez-benz-token-exposed-source-code-github/)

梅赛德斯-奔驰在一次疏忽后意外暴露了大量内部数据，因为将一个私钥在线上留下，给了公司源代码“无限制的访问权限”，这是一家发现这一问题的安全研究公司所说的。

Shubham Mittal，RedHunt Labs 的联合创始人兼首席技术官，向 TechCrunch 发出警告，请求帮助披露给汽车制造商。这家总部位于伦敦的网络安全公司表示，它在一月份的例行网络扫描中发现了一名梅赛德斯员工的认证令牌在一个公开的 GitHub 仓库中。

根据 Mittal 的说法，这个令牌——作为 GitHub 身份验证的替代品——可以授予任何人对梅赛德斯的 GitHub 企业服务器的完全访问权限，从而允许下载公司的私有源代码仓库。

“这个 GitHub 令牌给了整个内部 GitHub 企业服务器‘无限制’和‘无监控’的访问权限，”Mittal 在一份由 TechCrunch 分享的报告中解释道。“这些仓库包括大量的知识产权……连接字符串、云访问密钥、蓝图、设计文档、[单点登录]密码、API 密钥和其他关键的内部信息。”

Mittal 向 TechCrunch 提供了证据，显示暴露的仓库包含 Microsoft Azure 和 Amazon Web Services（AWS）密钥、一个 Postgres 数据库和梅赛德斯的源代码。目前尚不清楚这些仓库中是否包含任何客户数据。

TechCrunch 在周一将安全问题向梅赛德斯披露。周三，梅赛德斯发言人 Katja Liesenfeld 确认公司“撤销了相应的 API 令牌并立即删除了公开的仓库。”

“我们可以确认，由于人为错误，内部源代码已经发布在一个公开的 GitHub 仓库上，”Liesenfeld 在向 TechCrunch 发表的声明中说道。“我们组织、产品和服务的安全是我们的首要任务之一。”

“我们将继续根据我们的正常流程分析这个案例。根据这一情况，我们将实施补救措施，”Liesenfeld 补充道。

目前尚不清楚除了 Mittal 之外是否还有其他人发现了暴露的密钥，这些密钥是在 2023 年 9 月底发布的。

梅赛德斯拒绝透露是否知道任何第三方访问暴露数据的情况，或者公司是否有技术能力（例如访问日志）来确定是否有任何对其数据仓库的不当访问。发言人引用了未指明的安全原因。

上周，[TechCrunch 独家报道，现代汽车印度子公司修复了一个漏洞](https://techcrunch.com/2024/01/11/hyundai-motor-india-data-exposed/)，该漏洞泄露了其客户的个人信息，包括在印度各地现代汽车自营站点维修过车辆的客户的姓名、邮寄地址、电子邮件地址和电话号码。

> [现代汽车印度子公司修复了泄露客户个人数据的漏洞](https://techcrunch.com/2024/01/11/hyundai-motor-india-data-exposed/)。
