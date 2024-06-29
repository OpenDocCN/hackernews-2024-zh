<!--yml

category: 未分类

date: 2024-05-27 14:53:58

-->

# 引入 FlakeHub Cache

> 来源：[https://determinate.systems/posts/flakehub-cache-beta/](https://determinate.systems/posts/flakehub-cache-beta/)

[二进制缓存](https://zero-to-nix.com/concepts/caching) 确实是 [Nix](https://zero-to-nix.com) 最精彩的核心特性之一。它允许你获取 [导出](https://zero-to-nix.com/concepts/derivations) 的构建结果，而不是在本地构建它们，这使得你使用 Nix 进行的一切工作——[开发环境](https://zero-to-nix.com/concepts/dev-env)、[软件包构建](https://zero-to-nix.com/concepts/packages)、持续集成运行——都变得更加快速，以至于没有缓存的 Nix 本质上是一个不同的、次等的工具。但尽管我们热爱 Nix 缓存，我们认为现有的缓存解决方案不适合安全的生产环境。因此，我们决定打造一些更好的东西。

今天，我们非常高兴地宣布 **FlakeHub Cache**，这是我们在 [Determinate Systems](/) 推出的一个强大的新缓存概念，提供[强大的 flake 级别访问控制](#access-control)。FlakeHub Cache 目前处于 *私有 beta* 阶段，我们正在积极迭代实现，并寻找设计合作伙伴来尝试它。您可以在这里注册 beta 版，并很快我们将告诉您下一步的操作：

注册 FlakeHub Cache 的私有 beta 版

Nix 的官方二进制缓存服务器，[`nix-serve`](https://nixos.org/manual/nix/stable/package-management/binary-cache-substituter)，存在一些主要缺陷，使其不适合用于安全的生产环境。最重要的是，`nix-serve` 将 [Nix 存储](https://zero-to-nix.com/concepts/nix-store) 作为单个整体缓存提供，而缓存中的*所有内容*对任何持有公钥的人都是可用的。对于大多数组织来说，这种松散的访问模型是不可接受的。

然而，FlakeHub Cache 的访问控制模型允许你在 [flake](https://zero-to-nix.com/concepts/flakes) 级别授予或拒绝读取和写入权限，这为你提供了更多的控制能力。那么这是什么样子呢？

让我们从读取权限开始，以从 flake 的缓存中拉取数据。例如，如果你的组织有一个名为 `security-mega-important` 的 flake，你可以只为一小部分信任的用户或组织中的自动代理提供读取权限。另一方面，如果你有一个名为 `shared` 的 flake，旨在供所有人使用，你可以为你的组织中的每个人提供读取权限。

至于写入权限，我们选择了围绕受控、经过身份验证的构建环境为中心的模型。FlakeHub Cache 目前允许将推送到缓存的操作仅作为 [GitHub Actions](https://docs.github.com/en/actions) 运行的一部分（支持 [GitLab](https://gitlab.com) 的支持正在进行中）。这意味着*绝对不允许*通过 shell 脚本或 CLI 命令意外地推送到缓存。

这种模型的粒度和可控性非常适合具有严格安全和其他要求的组织，远远超出了现有解决方案的水平。

当然，为了授予或拒绝访问，FlakeHub 缓存需要确定是谁或什么正在发出请求。FlakeHub 缓存的认证基于[JSON Web Tokens](https://jwt.io) (JWTs)。目前我们使用[GitHub](https://github.com)作为我们的 JWT 认证提供者，但很快将支持[GitLab](https://gitlab.com)。未来，这种灵活的模型将使我们能够支持各种认证解决方案，包括[SAML](https://en.wikipedia.org/wiki/SAML_2.0)和各种[单点登录](https://en.wikipedia.org/wiki/Single_sign-on) (SSO) 提供者。

静态令牌在某些情况下非常有效—FlakeHub 允许您在 UI 中创建这样的令牌—但我们选择超越这种模型。

FlakeHub 缓存能够提供[细粒度访问控制](#access-control)，因为它不是像服务整个[Nix store](https://zero-to-nix.com/concepts/nix-store)那样作为“缓存”，而是应用了**切片**抽象概念，其中切片是所有存储路径的某个子集。启用 FlakeHub 缓存后，在[FlakeHub](https://flakehub.com)上发布的每个 flake 都有其自己的切片，并且在此级别应用读写权限。

当用户通过[认证](#auth)与 FlakeHub 缓存连接时，它会确定您被允许访问的切片，并将这些切片组合成缓存的一个单一**视图**。通过这种视图抽象，FlakeHub 缓存可以做出如下决策：

+   用户`devops_aficionado_123`来自`WidgetsDotCom`组织，*可以*从缓存中拉取`WidgetsDotCom/devops` flake。

+   用户`AnaBooper`来自`WidgetsDotCom`组织，*不得*从`WidgetsDotCom/super-secure` flake 拉取。

尽管这些用户属于同一组织，WidgetsDotCom 可以根据 flake 来做出访问决策，因此可以选择任何访问模式和协作模型。从安全角度来看，这种模型显然更为健壮，但同时也：

1.  **更简单**。在使用 Nix 时，您只需配置 FlakeHub 缓存；无需分发多个公钥。

1.  **提供更好的性能**。仅使用一个平台进行缓存意味着减少网络往返和认证握手次数。此外，我们在多个区域运行 FlakeHub 缓存，并支持 CDN 支持的存储，这显著加速了您的 Nix 构建过程。

除了访问控制外，像垃圾收集等功能也可以在 flake 级别进行配置。

使用 FlakeHub 缓存的两个核心方面使其与其他可用的缓存系统有显著差异：

1.  **不公开缓存**。没错：FlakeHub 缓存不允许您公开像[cache.nixos.org](http://cache.nixos.org)这样的公共缓存。只有在您（a）通过身份验证并且（b）有[权限](#access-control)的情况下，您才能从特定flake的缓存中拉取。这意味着您甚至不能*意外地*使特定flake的缓存真正公开。我们愿意在未来探索公共缓存，但只会谨慎行事。

1.  **只能通过CI进行推送**。目前，在创建 FlakeHub 缓存认证令牌时，这些令牌仅适用于从缓存中*拉取*。您无法通过CLI或其他工具*推送*到缓存，只能从[GitHub Actions](https://docs.github.com/en/actions)，特别是[Magic Nix Cache Action](https://github.com/DeterminateSystems/magic-nix-cache-action)（支持[即将推出的GitLab支持](https://gitlab.com)）。如果您参加测试版，您可以通过在Actions配置中使用以下一行代码来启用推送：

    ```
     permissions:
        contents: read
        id-token: write
      steps:
        - uses: actions/checkout@v3
        - uses: DeterminateSystems/nix-installer-action@main
        - uses: DeterminateSystems/magic-nix-cache-action@main
        - run: nix flake check
    ```

有了这个设置，您通过`nix build`的任何东西都会自动推送到缓存中——如果有授权的话！——而无需单独的推送命令。

我们坚信，FlakeHub 缓存是Nix生态系统中首个企业级准备就绪的缓存。公共缓存和缺乏粒度访问控制的缓存简直不适合广泛的用例和整个行业，特别是那些受到严格合规标准、监管制度和安全需求约束的行业。如果您对Nix的强大感兴趣，但对现有解决方案有顾虑，这可能正是您一直在等待的重大变革。

此外，Determinate Systems是Nix生态系统中唯一符合[SOC2](https://www.imperva.com/learn/data-security/soc-2-compliance/)合规的供应商，这应该会使与高层的讨论变得更加顺利。

您很快就可以自行尝试 FlakeHub 缓存。立即注册，配置其中一个GitHub Actions运行以使用Magic Nix Cache Action，对您的Nix配置进行小幅更新，您将第一手体验这一海量变化。

注册 FlakeHub 缓存的私人测试版
