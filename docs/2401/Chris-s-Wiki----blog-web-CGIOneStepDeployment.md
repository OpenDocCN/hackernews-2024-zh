<!--yml

category: 未分类

date: 2024-05-27 15:05:55

-->

# Chris's Wiki :: blog/web/CGIOneStepDeployment

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/web/CGIOneStepDeployment](https://utcc.utoronto.ca/~cks/space/blog/web/CGIOneStepDeployment)

当我写关于 [CGI 程序在今天并不特别慢](/~cks/space/blog/web/CGINotSlow) 时，我看到的一种反应是建议您最好使用 [FastCGI](https://en.wikipedia.org/wiki/FastCGI) 系统将您的 'CGI' 作为持久守护程序运行，这样您就可以节省每个请求时启动 CGI 程序的开销。其中一个实际的答案是，FastCGI 并没有像通常提供的 CGI 那样简单的部署模型，[这也是它们的吸引之一](/~cks/space/blog/web/CGIAttractions)。

有许多 CGI 使用和配置模型，安装 CGI、删除 CGI 或更新 CGI 都是一个单步过程；您将程序复制到一个目录中，然后再删除或更新它。Web 服务器注意到可执行文件存在（有时带有特定的扩展名或其他内容）并在响应请求时运行它。这种部署模型肯定可以变得更为复杂，您可以将整个 URL 树定向到一个 CGI，但这并非必须；您可以从非常简单的方式开始并逐步扩展。

理论上可以使 FastCGI 部署几乎与 CGI 模型一样简单，但我不知道是否有任何 FastCGI 服务器和 Web 服务器对此提供良好支持。相反，FastCGI 和一般的 '应用服务器' 模型几乎总是至少需要两步配置，您需要在应用服务器中配置您的应用程序，然后在 Web 服务器中配置您的应用程序的 URL（以便将其转发到您的应用服务器）。在某些情况下，每个应用程序需要一个单独的服务器（FastCGI 或其他机制），这意味着每次添加应用程序时都必须安排启动并可能监视新服务器。

(我将假设 FastCGI 服务器支持在部署更改时可靠且自动的热重载您的应用程序。如果不支持，那么事情会更复杂。)

如果您的应用程序相对静态，这种多步部署过程是完全可以的，因为您不必经常进行。但这更为复杂，通常需要一定程度的集中化（例如，用于 Web 服务器配置更新），尽管可以完全分布式 CGI 部署模型，其中人们只需将合适命名的程序放入自己拥有的目录中（然后通过例如 Apache suexec 以其自身身份运行它们的 CGI）。当然，这也是需要学习的更多内容。

(CGI 并不是 web 语言领域唯一具有简单一步部署模型的东西。PHP 传统上也有，尽管我模糊的理解是现在人们经常使用 PHP 应用服务器。)

PS：至少在 Apache 上，CGI 也有一个简单的调试故事；Web 服务器将记录 CGI 发送到标准错误输出的任何内容，包括由于完全无法运行而生成的任何输出。这在没有经验的人尝试开发和运行他们的第一个 CGI 时非常有用。[其他 Web 服务器有时可能不那么有用](/~cks/space/blog/sysadmin/LighttpdCGIStderr)。
