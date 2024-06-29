<!--yml

类别：未分类

日期：2024-05-29 12:29:45

-->

# Apt 仓库镜像使用 Git-LFS – Simon Josefsson's blog

> 来源：[https://blog.josefsson.org/2024/03/18/apt-archive-mirrors-in-git-lfs/](https://blog.josefsson.org/2024/03/18/apt-archive-mirrors-in-git-lfs/)

我努力提高公共 apt 存档的透明度和信任度。我开始在 “[Apt Archive Transparency](https://blog.josefsson.org/2023/02/01/apt-archive-transparency-debdistdiff-apt-canary/)” 中工作，在其中提到了 [debdistget](https://gitlab.com/debdistutils/debdistget/) 项目。我意识到，拥有公开可审计和保留的 apt 仓库镜像对于能够进行 apt 透明度工作至关重要，因此 debdistget 项目比我想象的更加核心。目前我跟踪 [Trisquel](https://trisquel.info/)、[PureOS](https://pureos.net/)、[Gnuinos](https://www.gnuinos.org/) 及其上游 [Ubuntu](https://ubuntu.com/)、[Debian](https://www.debian.org/) 和 [Devuan](https://www.devuan.org/)。

**Debdistget** 下载 **Release/Package/Sources** 文件并将其存储在发布在 [GitLab](https://about.gitlab.com/) 上的 Git 仓库中。由于大小限制，它使用了两个仓库：一个用于 **Release/InRelease** 文件（较小），另一个也包括 **Package/Sources** 文件（较大）。例如，可以查看 [Trisquel 发布文件的仓库](https://gitlab.com/debdistutils/archives/trisquel/releases) 和 [Trisquel 包/源文件的仓库](https://gitlab.com/debdistutils/archives/trisquel/packages)。所有发行版的仓库可以在 [debdistutils 的归档 GitLab 子组](https://gitlab.com/debdistutils/archives) 中找到。

分成两个仓库的原因是，合并文件的 Git 仓库变得很大，并且我的一些用例只需要发布文件。当前包含包（现在包含几个月的数据）的仓库为[Ubuntu](https://gitlab.com/debdistutils/archives/ubuntu/packages)为 9GB，[Trisquel](https://gitlab.com/debdistutils/archives/trisquel/packages)/[Debian](https://gitlab.com/debdistutils/archives/debian/packages)/[PureOS](https://gitlab.com/debdistutils/archives/pureos/packages)为 2.5GB，[Devuan](https://gitlab.com/debdistutils/archives/devuan/packages)为 970MB，[Gnuinos](https://gitlab.com/debdistutils/archives/gnuinos/packages)为 450MB。仓库大小与存档的大小（用于初始导入）以及更新的频率和大小相关联。Ubuntu 使用 [Apt Phased Updates](https://wiki.ubuntu.com/PhasedUpdates)（触发更高频率的 Packages 文件修改）似乎是其较大大小的主要原因。

处理大型 Git 存储库效率低下，GitLab CI/CD 基准下产生大量的网络流量，反复下载 Git 存储库。其中最明显的用户是 [debdistdiff](https://gitlab.com/debdistutils/debdistdiff) 项目，它下载所有分布包存储库以对不同分布中的包清单进行差异操作。每日任务运行时间大约为 **80 分钟**，其中大部分时间都用于下载存档。诚然，我知道我可以研究运行者侧缓存，但我不喜欢由缓存引发的复杂性。

幸运的是，并非所有用例都需要包文件。[debdistcanary](https://gitlab.com/debdistutils/debdistcanary) 项目只需 **Release/InRelease** 文件，以便向 [Sigstore](https://docs.sigstore.dev/) 和 [Sigsum](https://www.sigsum.org/) 透明日志添加签名。这些工作仍然相当快速，但观察存储库大小的增长让我感到担忧。目前这些存储库的大小分别为：Debian 对应的 [Canary](https://gitlab.com/debdistutils/canary/debian) 为 440MB、PureOS 为 130MB、Ubuntu/Devuan 为 90MB、Trisquel 为 12MB、Gnuinos 为 2MB。我确信主要的大小相关因素是更新频率，Debian 较大是因为我追踪了不稳定的 volatile。

这样我的第一种方法在扩展性上遇到了瓶颈。几个月前，通过删除和重置这些归档存储库，我“解决了”这个问题。GitLab CI/CD 基准再次快速运行而且一切都很顺利。然而，这意味着丢弃宝贵的历史信息。几天前，我再次接近了实际使用范围的极限，开始探索解决方法。我喜欢将数据存储在 git 中（它允许集成到如 GnuPG 和 Sigstore 这样的软件完整性工具中，并且 git 日志提供了数据的某种程度的时间排序），所以我觉得放弃美观的属性而使用传统的在磁盘上的数据库似乎是错误的。因此，我开始学习 [Git-LFS](https://git-lfs.com/) 并理解它能够处理多达数 GB 的数据，这看起来很有希望。

我很快编写了一个 [GitLab CI/CD 作业](https://gitlab.com/debdistutils/debdistget/-/blob/main/ci-debdistget-dists.yml)，用于逐步更新**Release/Package/Sources**文件，这些文件存储在一个使用 Git-LFS 存储所有文件的 Git 仓库中。现在仓库的大小分别为 [Ubuntu 650kb](https://gitlab.com/debdistutils/dists/ubuntu)，[Debian 300kb](https://gitlab.com/debdistutils/dists/debian)，[Trisquel 50kb](https://gitlab.com/debdistutils/dists/trisquel)，[Devuan 250kb](https://gitlab.com/debdistutils/dists/devuan)，[PureOS 172kb](https://gitlab.com/debdistutils/dists/pureos) 和 [Gnuinos 17kb](https://gitlab.com/debdistutils/dists/gnuinos)。可以预期的是，作业快速克隆 Git 存档：[debdistdiff 管道](https://gitlab.com/debdistutils/debdistdiff/-/pipelines)的运行时间从 80 分钟缩短到了 10 分钟，这与存档大小和 CPU 运行时间更为合理地对应。

这些仓库的 LFS 存储大小分别为 [Ubuntu 15GB](https://gitlab.com/debdistutils/dists/ubuntu)，[Debian 8GB](https://gitlab.com/debdistutils/dists/debian)，[Trisquel 1.7GB](https://gitlab.com/debdistutils/dists/trisquel)，[Devuan 1.1GB](https://gitlab.com/debdistutils/dists/devuan)，[PureOS](https://gitlab.com/debdistutils/dists/pureos)/[Gnuinos](https://gitlab.com/debdistutils/dists/gnuinos) 420MB。这些数据仅为几天的量。似乎本地 Git 在压缩/去重数据方面优于 Git-LFS：Ubuntu 的综合大小已经达到了 15GB，而纯 Git 的几个月数据仅为 8GB。这可能是 GitLab 中 Git-LFS 的一个次优实现，但我担心这种新方法在扩展性上可能存在困难。在某种程度上，这种差异是可以理解的，Git-LFS 可能会将两个不同的**Packages**文件（比如 Trisquel 中每个约为 90MB）存储为两个 90MB 文件，而本地 Git 则会将其存储为一个压缩版本的 90MB 文件和一个相对较小的补丁文件以转换旧文件为新文件。因此，总体存储大小而言，Git-LFS 的方法似乎不那么优秀。尽管如此，原始仓库要小得多，而且通常不必拉取所有 LFS 文件。因此，这仍然是一个优势。

在整个工作过程中，我一直在思考我的方法如何与[Debian快照服务](https://snapshot.debian.org/)相关联。最终，我希望能够结合这两种服务。为了在透明工作方面有一个良好的基础，我希望拥有所有曾经发布的**Release/Packages/Sources**文件的集合，最终还包括源代码和二进制文件。虽然从最新的稳定发行版开始是有意义的，但这一努力也应该向过去扩展。为了从源代码重现二进制文件，我需要能够安全地找到用于重建的早期版本的二进制包。因此，我需要将来自快照的所有**Release/Packages/Sources**包导入我的存储库。从该服务器检索文件的延迟很慢，因此我还没有找到有效/并行下载文件的方法。如果我能完成这个任务，我对基于新的Git-LFS存储这些文件的方法能够持续多年感到有信心。这还有待验证。也许存储库必须按发行版或架构等方式分割。

另一个因素是存储成本。虽然具有多年文件的基于Git-LFS的存储库的Git存储库大小可能是可以接受的，但Git-LFS存储大小肯定不会。看起来GitLab对存储库和Git-LFS中的文件收费相同，大约是每年**每100GB约$500**。也许可以设置一个独立的Git-LFS后端，不是由GitLab托管来提供LFS文件。有没有人知道一个适合此用途的服务器实现？我快速查看了[Git-LFS实现列表](https://github.com/git-lfs/git-lfs/wiki/Implementations)，似乎最接近的合理方法是设置Gitea-clone [Forgejo](https://forgejo.org/)作为自托管服务器。也许类似S3的云存储方法是正确的选择？在GitLab上托管这个的费用对于**~1TB（每年$5000）**还能接受，但将其扩展到存储约**500TB**的数据将意味着每年**$2.5M**的费用，这似乎物有所值较差。

我意识到，最终我会想要在本地拥有一个包含所有apt存档内容（包括其二进制和源码包）的git仓库。像快照（约300TB的数据？）这样的服务现在不是 prohibitly 昂贵：20TB的硬盘每个$500，所以一个包含36个硬盘的存储机架将大约花费**$18,000 来存储720TB**，使用RAID1意味着360TB，这是一个很好的起点。虽然我听说过大小约为TB级别的Git-LFS仓库，但Git-LFS是否能扩展到1PB？也许一个包含数百万个Git-LFS指针文件的git仓库将变得难以管理？为了开始这种方法，我决定将**Debian的bookworm for amd64**的镜像导入到Git-LFS仓库中。这大约是**175GB**，因此即使在GitLab上也很便宜托管（每年$1000用于200GB）。使此仓库公开可用将使编写使用此方法的软件变得可能（例如，移植[debdistreproduce](https://gitlab.com/debdistutils/debdistreproduce)），以找出这是否有用以及是否可以扩展。通过Git-LFS分发apt仓库还将使其他有趣的数据保护想法变为可能。考虑配置apt使用本地的**file://** URL访问此git仓库，并使用类似于[Guix的信任git内容的方法](https://archive.fosdem.org/2023/schedule/event/security_where_does_that_code_come_from/)或[Sigstore的gitsign](https://github.com/sigstore/gitsign)来验证git checkout。

一次性提交**175GB**存档的天真尝试遇到了打包大小限制：

`remote: fatal: pack exceeds maximum allowed size (4.88 GiB)`

然而，将提交分解为存档的部分小提交使得能够推送整个存档。以下是创建此仓库的命令：

`git init`

`git lfs install`

`git lfs track 'dists/**' 'pool/**'`

`git add .gitattributes`

`git commit -m"Add Git-LFS track attributes." .gitattributes`

`time debmirror --method=rsync --host ftp.se.debian.org --root :debian --arch=amd64 --source --dist=bookworm,bookworm-updates --section=main --verbose --diff=none --keyring /usr/share/keyrings/debian-archive-keyring.gpg --ignore .git .`

`git add dists project`

`git commit -m"Add." -a`

`git remote add origin git@gitlab.com:debdistutils/archives/debian/mirror.git`

`git push --set-upstream origin --all`

for d in pool/*/*; do

`echo $d;`

`time git add $d;`

`git commit -m"Add $d." -a`

`git push`

done`

结果的[存储库](https://gitlab.com/debdistutils/archives/debian/mirror)大小约为27MB，Git LFS对象存储约为174GB。我认为这种方法能够扩展以处理一个发布的所有架构，但使用一个单一的git存储库来处理所有版本的所有架构可能会导致一个过大的git存储库（>1GB）。所以也许每个发布一个存储库？这些存储库还可以根据**pool/**文件的子集进行拆分，或者可以每个版本每个架构或源一个存储库。

最后，我对使用SHA1来标识对象有所顾虑。目前Git和Debian的快照服务都在使用SHA1。对于Git来说，有关[SHA-256过渡](https://git-scm.com/docs/hash-function-transition)，而GitLab正在开发支持基于SHA256的存储库。对于这些概念的长期严肃部署，直接采用SHA256标识符将是不错的选择。Git-LFS已经在使用SHA256，但Git内部和Debian的快照服务仍使用SHA1。

你怎么看？愉快编程！
