<!--yml

分类: 未分类

日期: 2024-05-27 12:55:13

-->

# 单一Helm命令如何关闭生产环境 :: ./techtipsy

> 来源：[https://ounapuu.ee/posts/2024/04/04/helm-rollbljat/](https://ounapuu.ee/posts/2024/04/04/helm-rollbljat/)

你是一名Cletus Kubernetus：一名软件开发者，同时也是自豪的Fedora Linux用户。

你知道Kubernetes，尤其是你之前将一些服务迁移到它之后。

[一切都平静。](https://www.youtube.com/watch?v=ia8Q51ouA_s&pp=ygUGa3JhemFt)

你的pod正在运行。你的服务已上线，一切如常。

你对生产环境进行了一些次要的变更。一切还在工作。太好了！

但接着你收到了同事的消息。天啊，某个功能出现了问题！

没问题。你正在使用Helm。你可以安全地回滚这个更改。你询问了同事。“哦，没错，`helm rollback`应该有效。”

就是`helm rollback`。

稳住，稳住，新pod正在启动。看起来确实是正常工作。

**等一下，所有pod都去哪了？**

经过团队紧张的故障排查后，你重新部署了服务并开始调查。同事在副本环境里进行了`helm rollback`，结果正常，上一个版本的服务成功被部署。

你检查日志。`helm rollback`的调用按预期工作，并开始删除与部署相关的所有实体。pod、秘密、 ingress、与服务相关的*所有*内容，你的名字出现在了每个删除操作中。

暂时还在等待调查的几天里，因为你没有更多的线索，以及必须处理其他工作。但这场问题却让你在心理上难以释怀，对吧？

有一天你继续调查，打开了Helm GitHub仓库，查看打开的问题，并加入了一些可能相关的关键词，比如"rollback"。

[操，怎么回事。](https://github.com/helm/helm/issues/12681)

这不是Helm或者是你的运行方式的问题。显然，Fedora Linux打包的Helm版本里包含了引发这个问题的补丁。然后你使用了副本环境来重现这个问题。这一次是在一个更安全的环境里，一切又消失了。

你立刻运行了`dnf remove -y helm`。

在经历了这个以及[xz后门](https://openwall.com/lists/oss-security/2024/03/29/4)之后，住在农村并学习养蜂听起来不再那么糟糕，对吗？
