<!--yml

category: 未分类

date: 2024-05-29 12:50:01

-->

# VM 在黑客教育中的不合理有效性 | 安德鲁·奎因的 TILs

> 来源：[https://hiandrewquinn.github.io/til-site/posts/the-unreasonable-effectiveness-of-vms-in-hacker-pedagogy/](https://hiandrewquinn.github.io/til-site/posts/the-unreasonable-effectiveness-of-vms-in-hacker-pedagogy/)

如果你已经安装了[Vagrant](https://www.vagrantup.com/)和[VirtualBox](https://www.virtualbox.org/)，而且你的同事也是，那么你们可以通过运行以下命令，都可以启动一个几乎完全相同的空白 Debian 12 Linux 虚拟机：

|

```
[1](#hl-0-1) [2](#hl-0-2) [3](#hl-0-3) [4](#hl-0-4) [5](#hl-0-5) [6](#hl-0-6) 
```

|

```
mkdir tutorial/ cd tutorial/   vagrant init debian/bookworm64 vagrant up vagrant ssh 
```

|

. 这在你或者他们使用 Linux、Mac^(, BSD 或者甚至 Windows 时都适用。（通过别名的魔法，`mkdir` 和 `cd` 甚至在 PowerShell 中也可以工作。）

如果你接下来要写一篇教程，尤其是你试图展示一些你编写的新命令行工具或程序的教程，那么你可以从*你绝对需要安装和执行的所有事情*开始，来启动教程，直到达到实际使用工具的地步。你可以把这些变成你的同事可以机械地复制和粘贴的命令，因为你知道你们两个都从*完全相同的起点*开始。（直到它停止变得机械，而开始变成知识，几乎无意识地传授给学习者，因为你恰当地[设置了学习的环境](https://www.johnseelybrown.com/Situated%20Cognition%20and%20the%20culture%20of%20learning.pdf)。）

这种事情从学习者的角度来看是金子般宝贵的。这相当于在数学教科书中不跳过步骤，而且还有额外的好处，即教程读者可以尝试去除步骤，看看每个步骤为什么重要，以及在没有它们时会发生什么。

“但是安德鲁，Docker 已经存在了。而且已经有一段时间了。” 是的。但是不是每个人都知道什么是虚拟机。并非每个人都了解 Docker。你想冒险让某人去学习容器化是什么吗，当他们本可以学习*你的东西*？与学生会面的时候，要看他们所处的位置，而不是你希望他们在的位置。

这是我自从几个月前尝试 Laravel 的[Homestead](https://laravel.com/docs/11.x/homestead)以来，在专业和个人领域都在使用的一种技术。当我意识到使用 Vagrant 虚拟机可以显著减少我面对的障碍时，我被吸引住了。本周早些时候，我写了[一篇这样的教程](https://andrew-quinn.me/reposurgeon/)，而且我预计以这种方法将来还会写更多。
