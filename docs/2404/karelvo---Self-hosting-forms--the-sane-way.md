<!--yml

分类：未分类

日期：2024-05-27 13:35:27

-->

# karelvo | 自托管表单，理性方式

> 来源：[https://karelvo.com/blog/selfhosting-forms-the-sane-way](https://karelvo.com/blog/selfhosting-forms-the-sane-way)

我运行一些（非常）小的网站，不算严肃：它们不是业务，而是爱好。如果它们崩溃，除了可能会让一些通过搜索引擎找到链接的人失望外，没有任何风险。因为偶尔对它们进行修复是一种有趣的挑战，但与此同时，我不是开发者/系统管理员，我遵循两个规则：

+   尽可能多地自己完成，并寻找挑战。

+   …但当事情变得太令人沮丧/可怕时，请寻找替代方案。毕竟，这只是一种爱好，没有压力。

这两者结合在我的设置中的一些示例：

+   我所有的网站都是在Linux服务器上由自己运行的，**但**我租用了服务器（在家庭网络上开放端口给我带来的焦虑感比我需要的还要多）。

+   我不使用CMS或“任何带GUI的东西”，**但**我使用静态站点生成器（我懒得用原始的HTML、CSS和JS）。我最喜欢的是[Astro](https://astro.build)。

+   我自己托管了一些服务，比如网站分析，以便尽量减少数据经过的中间环节，**但**我使用[Coolify](https://coolify.io/)来部署我的网站（最近，也管理我的服务器）。

换句话说：除非有一个很棒的开源软件工具可以帮助我。我告诉自己的借口是“与做同样事情的95%的人相比，这种方法更*hacky*”。

* * *

## DIY vs 表单服务

* * *

在我的某个网站上，我需要安装一个具有文件上传功能的表单。经过一些研究，我找到了几种解决方案来满足我的需求：

+   **嵌入在其他地方构建的表单**。例如[Tally](https://tally.so/)（顺便说一句，这是一家非常鼓舞人心的公司），[Typeform](https://www.typeform.com/)和[Jotform](https://www.jotform.com/)。我不喜欢嵌入的想法，因为它无法让我控制我的网站上显示的内容。

+   **表单后端**，基本上是可以将表单数据发送到的数据库。例如[Formspree](https://formspree.io/)，[FormSubmit](https://formsubmit.co/)，[getform](https://getform.io/)和[Submit JSON](https://www.submitjson.com/)。如果我的用例更专业，我会选择这个。

+   **使用PHP脚本DIY**，也就是老派方式。简单但相对不安全且易于损坏（对于我这种技能水平的人）。

我不喜欢我找到的东西：我想要一些

+   …在这里我**不需要支付**服务费（或者如果不支付会在表单/提交/样式上受到限制），这意味着选项1和2被排除在外。

+   …那**不允许其他服务处理表单数据**，所以选项1和2再次不合适。

+   …那时我需要一个**安全**且**不会让我头疼**的东西，所以第三个选项也被排除了。

* * *

## 我的解决方案：折中之道

* * *

最终，我决定尽可能地按照上述要点建立自己的系统。总结起来，它看起来是这样的：

大多数阅读此图表的人不需要过多的解释就能理解。在我深入探讨细节之前，这里是这种设置的优缺点：

+   Pro - 我可以随心所欲地处理我的表单：拥有无限的提交、字段，并且可以根据自己的需求进行样式化。唯一的瓶颈是我的服务器的容量。

+   Pro - 故障排除非常轻松；n8n 作为“中央处理器”非常好用。

+   Pro/con - 数据只通过用户的浏览器、我的服务器和（不幸地）一个电子邮件服务器传递。如果您想要非常严格，可以将其仅发送到您自己托管的数据库 - 或者自己托管电子邮件服务器，这通常是一件非常麻烦的事情。

+   Con - 您的服务器是单点故障。如果它宕机，您将会失去除通过电子邮件接收的历史数据以外的所有数据。解决方案：将 n8n 托管在不同的服务器上，或者进行持续的异地备份（无论如何都应该这样做）。

这是逐步的过程。是的，它真的这么简单：

* * *

### 表单被填写

* * *

+   您的网站托管一个看起来像这样的表单：

```
<form action="${webhook}" method="post">
 <label for="name">Name</label>
 <input name="name" type="text">
 <button data-umami-event="buttonname" type="submit">Submit</button>
</form> 
```

```
<form
 action="${webhook}"
 method="post"
 >
 <label for="name">Name</label>
 <input name="name" type="text">
 <button
 data-umami-event="buttonname"
 type="submit"
 >
 Submit
 </button>
</form> 
```

+   用户填写表单，按下“提交”按钮，会发生两件事：

+   表单数据被发送到您在表单中声明的 n8n webhook。

+   （可选）按钮点击事件被发送到您的网站分析数据仓库 [Umami](https://umami.is/docs/track-events) 并在上述示例中事件名称为 `buttonname`。

* * *

### n8n 进程

* * *

+   [n8n](https://n8n.io/) 是一款工作流自动化工具（类似于 [Zapier](https://zapier.com)），[您可以自行托管](https://docs.n8n.io/hosting)。

+   n8n 通过 webhook 接收表单数据。

+   （可选但建议）添加一个 [Respond to Webhook](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.respondtowebhook/) 步骤，其中您可以定义提交表单后应重定向到的页面。

+   （可选）添加数据清理步骤，以便按您需要的方式格式化接收到的数据。

+   添加 2 个后续、独立的步骤：

* * *

### NocoDB 收集

* * *

+   [NocoDB](https://nocodb.com/) 是一个无代码数据库（类似于 [Airtable](https://www.airtable.com/)），[您可以自行托管](https://docs.nocodb.com/getting-started/self-hosted/installation)。

+   如果您创建一个列与表单字段对应的表格，您可以在 n8n 的 NocoDB 步骤中选择“自动映射输入数据到列”。非常顺利！如果不行，或者如果您需要更详细的数据如时间、IP 地址等，您可以为每个列定义它。

+   使用 NocoDB 作为您提交的所有表单的数据仓库。

* * *

### 电子邮件通知

* * *

+   我发现及时收到表单填写的通知非常关键，因为在我的网站上，表单填写相对较少。如果您是一家每天会接收到多个表单填写的企业，那么将其同步到每日检查的 CRM 当然是一个更明显的选择。

+   n8n与许多服务集成，可通知您（考虑专有服务[Discord](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.discord/)或[Slack](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.slack/)，以及像[Pushbullet](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.pushbullet/)或[ntfy.sh](https://github.com/cryingpotat0/n8n-ntfy.sh)这样的服务）。我选择了[email](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.sendemail/)。

+   我的电子邮件是通过[Fastmail](https://fastmail.com/)托管的，它通过其[应用密码](https://www.fastmail.help/hc/en-us/articles/360058752854-App-passwords)与第三方应用无缝安全集成。电子邮件的发送通过SMTP完成。

+   我通过别名向自己发送包含表单数据的电子邮件。

就是这样：一种可以自行托管表单的方法，而不至于让你发疯。
