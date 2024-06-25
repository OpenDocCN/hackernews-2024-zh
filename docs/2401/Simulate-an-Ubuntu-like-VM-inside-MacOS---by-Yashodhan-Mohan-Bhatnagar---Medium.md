<!--yml

类别：未分类

日期：2024年05月27日 14:51:00

-->

# 在MacOS中模拟类似Ubuntu的VM | 作者：Yashodhan Mohan Bhatnagar | Medium

> 来源：[https://yashodhanmohan.medium.com/simulate-an-ubuntu-like-vm-inside-macos-2a20332e02e8](https://yashodhanmohan.medium.com/simulate-an-ubuntu-like-vm-inside-macos-2a20332e02e8)

# 在MacOS中模拟类似Ubuntu的VM

有时，当您正在处理像Ubuntu、RHEL等Unix系统的远程系统时。您想开发脚本，检查一些软件包是否存在，原型命令片段，或者可能测试一些兼容性。

每当我需要这样做时，我都必须启动一个来自AWS免费套餐的t2虚拟机进行测试，然后关闭它。这总是看起来很麻烦，特别是因为我需要互联网来模拟简单的命令。这不像Ubuntu这样的系统离MacOS已经很远了。但有时会有非常微妙的差异和棘手的错误，绝对需要您使用原始操作系统。有时简单的命令像**date**会有不同的标志行为。

最近，我开始使用Ubuntu的Docker容器来替代t2 VM。对于任何测试，我都会启动容器，设置依赖包，测试命令、脚本等，然后让它运行。有时它会被终止，有些日子我不知不觉地清理了所有的容器，有一两次我不得不删除Docker运行时本身（出于无关的原因）。如果我能像对待个人VM一样对待这个Docker容器，它可以保持一定程度的状态，那将是非常好的。我不想每次都安装基本包，像wget、curl等，对我来说，在Dockerfile中正式化每个小改变实际上是一个维护噩梦。如果我在操作系统文件系统中创建了一个符号链接，我不想将其编码到Dockerfile中。

为此，我设置了一批可以模拟VM、维护状态并让我将容器视为VM的脚本。这就是“Friday”的故事。

# 状态

我想要维护两种状态——

1.  系统状态：这是当我安装软件包、进行系统更改、添加用户、更改权限以及执行各种随机管理员操作时所修改的状态。

1.  卷状态：这通常是一些重型的东西，如文件、CSV、密钥等，我不一定想要存储在容器镜像中。它里面的文件可以是小到大到非常大。将其放入容器镜像中将会很不方便，至少可以这么说。

处理系统状态时，我使用了**“docker export”**和**“docker import”**的组合来定期和按需备份容器的状态。

为了处理卷状态，我简单地将MacOS文件系统中的一个目录挂载到容器中的“**/root**”。它还有一个额外的副作用，就是将文件从我的Mac传输到“Ubuntu VM”就像从一个目录复制到另一个目录那样简单。最简单的SFTP。

# 星期五

Friday由4个简单的脚本组成：

1.  ***start-friday.sh***

    这个脚本检查是否已经运行了星期五，如果没有，就启动星期五，并立即备份其状态。

1.  ***freeze-friday.sh***

    这个脚本被所有其他脚本使用，只需通过运行 “**docker export**” 对 Friday 的状态进行备份，生成一个 TGZ 文件，然后立即通过 “**docker import**” 将其带回到 docker 镜像系统中。

1.  ***login-into-friday.sh***

    这个脚本运行 docker exec 命令进入 Friday。特殊的魔法是，当你干净地退出 shell 时，它会立即冻结 Friday。所以这确保了如果你在 shell 会话期间进行了任何更改，它们不会丢失。有点像应用程序在退出前会问你：“你想保存吗？”

1.  ***stop-friday.sh*** 这个脚本会对 Friday 进行备份并安全停止它。

Friday 的核心关键是 freeze-friday 逻辑，确保对 Friday 的每一次微小修改都进行了备份，类似于 AWS 如何维护你的 EBS 备份。

另一种保护 Friday 的方法是设置 **crontab**，通过运行 freeze-friday 每小时/每半小时备份 Friday。

我已经放置了下面的脚本，只需要你修改“ROOT_PATH”变量为一个星期五可以存储其 TGZ 备份和卷状态的目录。设置好之后，只需要一次，你需要从 DockerHub 或你的注册表中拉取你喜欢的操作系统的基础镜像到你的本地系统，并将其标记为“***friday:latest***”。

***这些脚本可以在*** [***这里***](https://gist.github.com/yashodhanmohan/2c1b712883dec6dfba8ba2586df1b0cc) ***作为 Gist 获取***。

例如，我想让我的 VM 是 Ubuntu 口味的，所以：

setup

start-friday.sh

login-into-friday.sh

freeze-friday.sh

stop-friday.sh

你也可以像这样将 freeze-friday 添加到 crontab 中：
