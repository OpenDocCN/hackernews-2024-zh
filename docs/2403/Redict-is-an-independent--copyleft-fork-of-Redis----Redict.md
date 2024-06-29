<!--yml

category: 未分类

date: 2024-05-29 12:35:05

-->

# Redict是Redis®的一个独立的copyleft分支 | Redict

> 来源：[https://redict.io/posts/2024-03-22-redict-is-an-independent-fork/](https://redict.io/posts/2024-03-22-redict-is-an-independent-fork/)

##### 2024年3月22日

像你们许多人一样，当我得知Redis®改变为[非自由许可模型](https://github.com/redis/redis/pull/13157)时，我感到失望。这是对自由软件社区的背叛，但或许并不完全令人惊讶。未来几天可能会出现分支，今天，我想向您提供[Redict](/)作为您未来需求的可能选择，并呈现其与您可能选择的其他分支的权衡。

简而言之，Redict 是 Redis® OSS 7.2.4 的一个独立、非商业分支。它基于 Redis® OSS 的 BSD 3-Clause 源代码，并且从此版本开始的所有更改均根据GNU Lesser General Public License，即LGPL-3.0-only授权许可。

## 为什么选择LGPL？ [#](#why-lgpl)

选择LGPL作为Redict的许可证是经过深思熟虑的决定，平衡了多种考虑因素。最重要的是，这是一个不可撤销的承诺，保证了Redict将始终保持自由，这比2018年RedisLabs联合创始人兼前CEO Yiftach所做的承诺更加坚定。通过使用copyleft许可证，要求所有对Redict的更改都必须使用相同的LGPL自由软件许可证进行分发，从而确保软件的修改版本将是自由的。

此外，我们不会使用任何形式的贡献者许可协议，该协议赋予某一实体在版权和Redict许可证方面的特殊特权，类似于Redis®采用的做法。因此，Redict的版权由所有贡献者共同持有，他们必须全体同意未来许可证的变更，这几乎不可能会在Redict的未来发生类似的许可证变更。贡献的来源使用[开发者证书](https://developercertificate.org/)进行验证。

有许多copyleft许可证，LGPL被选择为最适合Redict的许可证。Affero GNU General Public License（AGPL）是此类项目的常见选择，特别是那些由Redis® Ltd这样的管理者运行的项目，他们希望云提供商不销售他们的软件。AGPL是一个很好的许可证，但我们希望尽可能使用户遵守Redict许可证，并且我们不认为有任何理由阻止云提供商使用Redict。EUPL也被考虑过，但出于相同的原因未被选中。

选择LGPL而不是GNU General Public License，是为了减少与Redis®兼容模块或Lua插件集成可能受GNU GPL“传染性”影响的顾虑。

选择LGPL保护了Redict项目的未来，同时尽可能平衡了每一个相关的关切。

<details><summary>等等，你是怎么改变许可证的？你有权限这样做吗？</summary>

好问题！Redis® OSS基于BSD 3-Clause许可证，这是一种宽松的许可证，允许在符合其条款的情况下进行再许可。Redict通过保留原始许可证和版权声明遵守了这些条款。Redict所做的所有更改都使用LGPL许可，并且整体结合作品根据LGPL的条款进行分发。仓库中明确指出了许可情况，使用了[REUSE](https://reuse.software)规范。

顺便说一句，Redis® Ltd *并没有* 持有Redis®代码的版权（尽管他们拥有该名称的商标）。他们使用的许可证是同样宽松的BSD许可证，就像Redict一样，只是他们使用的是非自由的SSPL，而我们使用的是自由的LGPL。</details>

## 关于Redis®的变更[#](#changes-with-respect-to-redis)

目前，Redis® 7.2.4的变更范围有限。主要问题是以向后兼容的方式更改名称，并为独立未来奠定技术基础。迄今为止，用户界面的更改包括以下内容：

+   可执行文件已重命名为redict-*，例如redict-cli。

+   Lua API提供了一个“redict”全局，与Redis® OSS API兼容，通过“redis”全局进行向后兼容。

+   模块API符号已重命名，但Redict保留了与Redis® OSS模块版本7.2.4及以下的ABI兼容性。

我们正在进行重命名过程的完成工作，并且当首个版本7.3.0发布时，将提供迁移指南，这理想情况下将在下周发布。Redict将作为Redis® OSS 7.2.4的插拔替换，但如果愿意，您可以采取步骤更新您的本地配置以适应Redict（例如将您的数据库迁移到/var/lib/redict）。

我们还更新了仓库以符合[REUSE](https://reuse.software)规范，以便简化许可证符合过程，并澄清适用于Redict的各种软件许可证，包括原始Redis® OSS代码的BSD 3-Clause许可证和新的LGPL变更，以及Lua等供应商依赖。

### 未来变更[#](#future-changes)

Redict的意图是为Redis® OSS兼容软件的自由软件发布提供持续开发，并在当前尽量减少不必要的变更。目前正在讨论以下变更：

+   利用此机会移除一些长期弃用的功能，如“redis-trib”

+   消除供应商依赖，并转向上游Lua、jemalloc

+   变得更加下游不可知，删除例如示例systemd或upstart服务

我们还将分叉[Hiredis](https://github.com/redis/hiredis)，因为它是Redict的一个内部依赖。

我们不会努力保持与未来版本的Redis® SAL兼容。

### 基础设施变更 [#](#infrastructure-changes)

这个机会正在被用来建立一个独立于专有基础设施（如GitHub和Slack）的社区。源代码托管在[Codeberg](https://codeberg.org/redict/redict)，这是由德国非营利组织运营的Forgejo实例，对于熟悉Redis® OSS基于GitHub社区的任何人来说，应该提供了一个舒适和熟悉的用户体验。此外，我们还建立了一个IRC频道，[#redict on libera.chat](https://web.libera.chat/?channel=#redict)，正在组织这个蓬勃发展的社区。

### 与其他分支的关系 [#](#relationship-to-other-forks)

在Redis®许可证变更之前，已经建立了许多分支，比如[KeyDB](https://docs.keydb.dev/)。你应该去看看这些——也许你一直在错过！然而，这些分支比Redict的目标更具观点性：Redict将为Redis® OSS代码库提供更为保守的延续。

在接下来的几周内，我们可能会看到几个其他的Redis® OSS分支出现。我们可能会从那些使用宽松许可证或与LGPL兼容的Copyleft许可证的分支拉取变更。

## 招募帮助！ [#](#help-wanted)

请加入项目并提供帮助！已建立的Redis® OSS贡献者只需让自己的存在被知晓，即可获得对我们上游仓库的推送访问权限。如果您有任何针对Redis® OSS的未处理的拉取请求，请花些时间重新基于Redict进行重整。我们也鼓励新贡献者参与开发。

帮助建设Redict文档也是一个简单的办法，因为Redis®文档不使用自由许可证，因此无法为Redict进行调整。我们也对希望用自由软件替代其“redis”包的Linux发行版的参与感兴趣；请过来并告诉我们您的需求、关切和反馈。

加入我们的IRC频道开始：[#redict on libera.chat](https://web.libera.chat/?channel=#redict)。

在可预见的未来，我们将建立额外的社区基础设施，包括安全邮件列表和更新的行为准则。请关注本博客获取更新。
