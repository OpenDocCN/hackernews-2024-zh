<!--yml

类别：未分类

日期：2024-05-27 14:42:30

-->

# Bitnami 博客：Debian 12 现在是 Bitnami 打包软件的基础操作系统

> 来源：[https://blog.bitnami.com/2024/02/debian-12-is-now-base-operating-system.html](https://blog.bitnami.com/2024/02/debian-12-is-now-base-operating-system.html)

我们很高兴分享，我们已将所有 Bitnami 打包容器和 Helm charts 的社区版的基础操作系统（OS）更新为从 Debian 11（bullseye）到 Debian 12（bookworm）。

我们容器和 Helm charts 的这一更新有助于使系统软件包保持更为更新，并减少漏洞扫描器报告的未修复/未打补丁的漏洞数量。尽管我们定期使用最新的系统软件包更新我们的镜像，某些 CVE 可能会持续存在，直到在操作系统中修补为止。因此，转换到更新的发行版将允许我们加快我们的目录更新速度。您可以在此处了解更多关于我们 CVE 政策的信息[here](https://docs.bitnami.com/kubernetes/open-cve-policy/)。

想要超越 Debian 并使用其他知名 Linux 发行版作为 Bitnami 打包容器和 Helm charts 的基础操作系统的用户可以利用[Tanzu 应用目录](https://app-catalog.vmware.com/catalog)，这是 Bitnami 应用目录的企业版。Tanzu 应用目录提供多种基础镜像，如 Debian 11 和 12、PhotonOS 4、Ubuntu 20.04 和 22.04、RedHat UBI 8 和 9，以及定制的强化金镜像。

# 预期发生哪些变化？

+   **容器镜像标签**

+   根据 Bitnami 的[滚动标签政策](https://docs.bitnami.com/tutorials/understand-rolling-tags-containers/)，一些标签将更改以反映新的发行版版本，例如 *6-debian-11* 将变为 *6-debian-12*。

+   其他滚动标签将指向新的基于 Debian 12 的容器，而不在名称中明确提及，例如 *latest*。

+   不可变标签也将更改以显示新的发行版版本，并重置修订号，例如 *6.4.1-debian-11-r4* 将变为 *6.4.1-debian-12-r0*。

+   不会删除任何镜像。所有 Debian 11 的镜像将保留在注册表中。

+   **bitnami/containers GitHub 仓库**

+   在源代码方面，[GitHub 仓库](https://github.com/bitnami/containers) 将从 *bitnami/ASSET/BRANCH/debian-11* 更改为 *bitnami/ASSET/BRANCH/debian-12* 的目录结构。

+   请注意，这两个目录（debian-11 和 debian-12）可能会共存一段时间。

+   **Helm charts**

+   由于容器使用的发行版版本变更不会影响向后兼容性，不预期 Helm chart 版本的主要版本提升。

+   为每个 Helm chart 的新版本发布更新的容器指向基于 Debian 12 的镜像，即

> > 仓库：bitnami/mariadb
> > 
> > - 标签：11.1.3-debian-11-r0
> > 
> > + 标签：11.1.3-debian-12-r0
> > 
> > > 仓库：bitnami/os-shell
> > > 
> > > 如果您对此更新有任何疑问，请随时通过以下方式与我们的团队联系
> > > 
> > > [此 GitHub 问题](https://github.com/bitnami/charts/issues/21362)
