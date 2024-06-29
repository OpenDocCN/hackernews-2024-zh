<!--yml

category: 未分类

date: 2024-05-29 12:41:52

-->

# 英国税务和海关总署（HMRC）的Basic PAYE工具的Linux版本在更新后出现故障 • The Register

> 来源：[https://www.theregister.com/2024/03/26/hmrc_linux_paye_tools/](https://www.theregister.com/2024/03/26/hmrc_linux_paye_tools/)

更新：您知道英国税务和海关总署（HMRC）提供免费的Linux工具吗？不幸的是，最近它们停止工作了。

*Reg*读者Pete Donnell向我们指出，英国税务局提供其Basic PAYE工具的Linux版本，[下载页面](https://www.gov.uk/basic-paye-tools)描述为“英国税务和海关总署（HMRC）为少于10名员工的企业提供的免费工资单软件”。

听起来不错。我们对看到，除了Windows和Mac版本外，还有一个支持“Ubuntu Linux 20.04 LTS和23.10版本”的软件版本感到印象深刻。还有一个详细的[用户指南](https://www.gov.uk/government/publications/basic-paye-tools-user-guide/basic-paye-tools-user-guide)，以及多个服务状态页面，从[相当陈旧](https://www.gov.uk/government/publications/basic-paye-tools-service-availability-and-issues)到[极度陈旧](https://customs.hmrc.gov.uk/channelsPortalWebApp/channelsPortalWebApp.portal?_nfpb=true&_pageLabel=pageNoNavigation_ShowContent&id=HMCE_PROD1_031142&propertyType=document)不等。

当然，如果没有任何问题，我们不会写这篇文章的。这个故事已经酝酿了一段时间，但Pete告诉我们，2月5日：

尽管服务状态页面未显示任何问题，但自那时以来一直出现故障。他告诉我们，他的软件设置为自动更新已有约15年，以前从未出过问题。他已经在Ubuntu 20.04和Xubuntu 23.10的干净安装中尝试过。Mac用户也在[报告问题](https://discussions.apple.com/thread/255234259)。

令人担忧的是，唐奈尔指出这个软件似乎是用Python 2编写的。那个版本已经不再受支持并已[超过四年](https://pythonclock.org/)。

我们联系了HMRC并让他们与Donnell联系。随后他与他们交谈，并提供了一些故障排除信息。一个月后，他告诉我们，仍然没有任何变化。

发现政府部门不仅承认Linux的存在，而且支持并提供Linux专用下载看起来像是一个受欢迎的惊喜……但每一朵银色云层都有一朵乌云。

如果还有其他人使用PAYE工具并遇到类似问题，或者确实软件工作良好，或者能帮助Donnell排除看起来是Python 2和Django应用程序的问题，请告诉我们。这是一个免费的下载，他告诉我们，在任何雇主账户形式之前它就失败了。®

### 添加更新

唐奈尔先生又联系我们，告诉我们，《*The Reg*》的读者群帮助他找出了问题并解决了它。他告诉我们：
