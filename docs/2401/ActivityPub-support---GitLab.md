<!--yml

类别：未分类

日期：2024-05-27 15:23:55

-->

# ActivityPub 支持 | GitLab

> 来源：[https://docs.gitlab.com/ee/architecture/blueprints/activity_pub/](https://docs.gitlab.com/ee/architecture/blueprints/activity_pub/)

此页面包含与即将推出的产品、功能和功能相关的信息。重要的是要注意，所提供的信息仅供参考。请不要依赖此信息进行购买或规划。任何产品、功能或功能的开发、发布和时间可能会受到更改或延迟，并保留由 GitLab Inc. 完全自行决定的权利。

# 活动发布（ActivityPub）支持

## 摘要

此提案的最终目标是在 GitLab 中构建互操作性功能，以便在 GitLab 的一个实例上向托管在另一个实例上的项目提出合并请求，将所有愿意的实例合并为一个全局网络。

为了实现这一目标，我们建议使用 ActivityPub，这是 Fediverse 使用的 w3c 标准。这将使我们能够建立在一个稳健和经过实战检验的协议之上，并将 GitLab 开放给更广泛的社区。

在开始实施跨实例合并请求之前，我们希望从较小的步骤开始，帮助我们建立关于 ActivityPub 的领域知识，并创建支持更高级功能的基础架构。因此，我们建议首先实施社交功能，允许 Fediverse 上的人们订阅 GitLab 上的活动，例如在其社交网络中选择通知他们的最爱项目在 GitLab 上发布新版本时。作为一个额外的好处，这是一个使 GitLab 更具社交性并增加其受众的机会。

如果你已经了解了 ActivityPub 和 Fediverse，可以直接跳转到[动机](#motivation)。

在推动[网络去中心化](https://en.wikipedia.org/wiki/Decentralized_web)的过程中，有几个项目尝试了不同的协议，背后的理念也不尽相同。一些例子：

近来有一个获得了关注：[ActivityPub](https://en.wikipedia.org/wiki/ActivityPub)，更为人所知的是通过像 [Mastodon](https://en.wikipedia.org/wiki/Mastodon_%28social_network%29)（可以描述为某种去中心化的 Facebook）或 [Lemmy](https://en.wikipedia.org/wiki/Lemmy_%28software%29)（可以描述为某种去中心化的 Reddit）等应用构建在其上的俗称 [Fediverse](https://en.wikipedia.org/wiki/Fediverse)。

ActivityPub 具有几个优点，这些优点使其对实施者具有吸引力，并且可以解释其当前的成功：

+   **它建立在 HTTP 之上**。如果你有一个 web 服务器或提供 HTTP API 的应用程序（比如一个 rails 应用程序），你就已经拥有了实现 ActivityPub 所需的一切，不需要安装新软件或调整 TCP/UDP。

+   **它是建立在 JSON 之上的**。所有通信基本上都是 JSON 对象，这是网页开发人员已经习惯的，这简化了采用的过程。

+   **这是一个 W3C 标准，并且已经有多个实现**。由 W3C 主导是对稳定性和质量工作的保证。他们在过去通过对 HTML、CSS 或其他网络标准的工作表明，我们可以在其基础上构建，而不必担心在几年后它就会过时或变得无关紧要。

### 联合宇宙

Mastodon 和 Lemmy 的核心理念被称为联合宇宙（Fediverse）。与完全去中心化不同，这些应用程序依赖于联合，在这种情况下仍然有服务器和客户端。它不像 SSB、Dat 和 IPFS 那样是点对点的，而是一群服务器互相交流，而不是由单一实体控制的中央服务器。

用户注册到其中一个服务器（称为**实例**），然后他们就可以与该实例或其他实例上的用户互动。从用户的角度来看，他们访问的是一个全球网络，而不仅仅是他们的实例。他们可以看到其他实例发布的文章，可以对其发表评论、点赞等。

发生在幕后的事情: 他们的实例知道他们回复的用户托管在哪里。它联系另一个实例来告诉他们有一条消息。类似于 SMTP。类似地，当用户订阅一条反馈时，他们的实例通知托管反馈的实例这个订阅。目标实例创建新活动时，然后返回消息。这允许推送模型，而不是像 RSS 那样的常规轮询模型。当然，刚才描述的是幸福的路径; 一直有适度、验证和容错率的发生。

### ActivityPub

联合宇宙背后是 ActivityPub 协议。这是一个试图尽可能通用的社交网络 API，同时给出扩展的选项。

基本理念是一个`行动者`发送和接收`活动`。活动是结构化的 JSON 消息，具有明确定义的属性，但可以扩展以满足任何需要。行动者由四个端点定义，这些端点通过`application/ld+json; profile="https://www.w3.org/ns/activitystreams"` HTTP 接收头联系:

+   `GET /inbox`: 行动者用于查找发送给他们的新活动。

+   `POST /inbox`: 实例用于推送给行动者的新活动。

+   `GET /outbox`: 任何人都可以用来查看由该行动者创建的活动。

+   `POST /outbox`: 由行动者用来发布新的活动。

其中，Mastodon 和 Lemmy 只使用 `POST /inbox` 和 `GET /outbox`，这是实现联合所需的最低限度：

+   实例将行动者的新活动推送到收件箱。

+   读取 outbox 允许读取行动者的反馈。

此外，Mastodon和Lemmy实现了一个`GET /`端点（带有提及的Accept标头）。该端点会响应有关参与者的一般信息，比如收件箱和发件箱的名称和URL。虽然不受标准要求，但它使发现变得更容易。

虽然人是参与者的主要用例，但参与者不一定对应一个人。任何东西都可以成为参与者：一个话题，一个子版块，一个群组，一个事件。对于GitLab而言，任何具有活动（就像GitLab所说的“活动”那样）都可以成为ActivityPub参与者。这包括项目、群组和发布等项目。在这些更抽象的示例中，可以将参与者视为可操作的反馈。

ActivityPub本身并不能涵盖实现Fediverse所需的一切。特别是，这些留给实现者去解决：

+   找到一种方法来处理垃圾邮件。垃圾邮件通过授权或阻止（“取消联邦”）其他实例来处理。

+   发现新实例。

+   执行全网搜索。

## 动机

社交媒体协议对GitLab有什么用处呢？人们希望有一个单一的全球GitLab网络，可以在各种项目之间进行交互，而不必在每个主机上注册。

已经有几次围绕这个问题的非常受欢迎的讨论：

理想的工作流程应该是：

1.  Alice注册到她最喜欢的GitLab实例，像是`gitlab.example.org`。

1.  她在给定主题上寻找一个项目，并看到Bob的项目，尽管Bob在`gitlab.com`上。

1.  Alice选择**Fork**，`gitlab.com/Bob/project.git`被分叉到`gitlab.example.org/Alice/project.git`。

1.  她进行编辑，并发起一个合并请求，该合并请求出现在Bob的`gitlab.com`项目中。

1.  Alice和Bob在各自的GitLab实例中讨论合并请求。

1.  Bob可以发送额外的提交，这些提交会被Alice的实例接收。

1.  当Bob接受合并请求时，他的实例会从Alice的实例中获取代码。

在这个过程中，ActivityPub会在以下方面帮助：

+   让Bob知道发生了分叉。

+   向Bob发送合并请求。

+   使Alice和Bob讨论合并请求。

+   让Alice知道代码已合并。

在这些情况下它*不*起作用，需要特定的实现：

+   实现全网搜索。

+   实现跨实例分支。（由于Git不需要，因此不是必需的。）

为什么在这里使用ActivityPub而不是以自定义方式实现跨实例合并请求呢？有两个原因：

1.  **在标准的基础上构建有助于超越GitLab**。尽管上面介绍的工作流程只提到了GitLab，但在W3C标准的基础上构建意味着其他熔炉可以跟随GitLab，构建一个庞大的代码共享联邦。

1.  **让 GitLab 更具社交性**。为了准备上述工作流程的架构，可以采取较小的步骤，允许人们从他们的 Fediverse 社交网络订阅活动动态。任何有 RSS 订阅的内容都可以成为 ActivityPub 订阅源。Mastodon 上的人可以关注他们在 GitLab 上喜欢的开发者、项目或主题，并在他们的 Mastodon 动态中看到相关新闻，希望能提高与 GitLab 的互动。

### 目标

+   允许在基于 ActivityPub 的社交媒体上分享有趣的事件

+   允许从一个实例打开问题并在另一个实例中进行讨论

+   允许从一个实例 fork 一个项目到另一个实例

+   允许从一个实例打开合并请求，讨论并合并到另一个实例

+   允许进行全网搜索？

### 非目标

+   私有资源的联邦

+   允许进行全网搜索？

## 提案

这种实施路径的想法不是为了采取增加最大价值功能的最快路径（跨实例合并请求），而是继续进行每次迭代中最小有用的步骤，确保每个步骤都立即带来一些有用的东西。

1.  **实现 ActivityPub 进行社交关注**。之后，Fediverse 可以关注 GitLab 实例上的活动。

    1.  使用 ActivityPub 订阅项目发布。

    1.  使用 ActivityPub 订阅主题中项目创建。

    1.  使用 ActivityPub 订阅项目活动。

    1.  使用 ActivityPub 订阅组活动。

    1.  使用 ActivityPub 订阅用户活动。

1.  **实现跨实例 fork**，以便从另一个实例 fork 一个项目。

1.  **实现 ActivityPub 进行跨实例讨论**，以便从另一个实例讨论问题和合并请求：

    1.  在问题中。

    1.  在合并请求中。

1.  **实现 ActivityPub 提交跨实例合并请求**，以便向其他实例提交合并请求。

1.  **实现跨实例搜索**，以便发现其他实例上的项目。

目前尚不确定是否应该包括这最后一步。当前，在大多数 Fediverse 应用中，当您想显示您的实例不知道的资源（通常是您想关注的用户）时，您会在您的实例的搜索框中粘贴资源的 URL，然后它会获取并显示远程资源，现在可以在您的实例中操作。我们计划首先这样做。

问题是：我们是否就此打住？这种用户体验存在严重的摩擦，尤其是对于不习惯 Fediverse 用户体验模式的用户（这可能是大多数 GitLab 用户）。另一方面，分布式搜索是一个足够复杂的主题，值得有自己的蓝图（尽管它并不像过去那样复杂，现在去中心化协议和应用程序已经在此方面努力了一段时间）。

## 设计和实施细节

首先，熟悉一下我们将要使用的三个标准的规范是个好主意：

+   [ActivityPub](https://www.w3.org/TR/activitypub/)定义了实现联合的HTTP请求。

+   [ActivityStreams](https://www.w3.org/TR/activitystreams-core/)定义了协议用户交换的JSON消息的格式。

+   [活动词汇表](https://www.w3.org/TR/activitystreams-vocabulary/)定义了默认情况下被识别的各种消息。

如果您有问题或觉得文件太难理解，请随时联系[@oelmekki](https://gitlab.com/oelmekki)。

### 生产就绪性

待确定

### 社交跟随部分

此部分正在为GitLab[添加新的ActivityPub角色](../../../development/activitypub/actors/index.html)奠定基础。

我们要实现5个角色：

+   `releases` actor，当给定项目发布新版本时通知

+   `topic` actor，当一个新项目被添加到一个主题时通知

+   `project` actor，涉及来自一个项目的所有活动

+   `group` actor，涉及来自一个群组的所有活动

+   `user` actor，涉及来自一个用户的所有活动

我们现在只处理公共资源。允许私有资源的联合是一个棘手的问题，如果可能的话，这将在以后解决。

#### 端点

每个角色需要3个端点：

+   包含基本信息的配置文件端点，例如名称、描述，还包括指向收件箱和发件箱的链接

+   发件箱端点，允许显示角色的先前活动

+   收件箱端点，用于发布跟随和取消跟随请求（我们现在不会使用的其他一些东西）。

提供这些端点的控制器位于`app/controllers/activity_pub/`。决定使用此命名空间是为了避免将ActivityPub的JSON响应与用于前端的响应混淆，而且因为我们可能需要进一步的命名空间，因为我们格式化活动的方式可能对于一个Fediverse应用程序、另一个应用程序以及我们以后的跨实例功能可能不同。此外，这个命名空间使我们能够轻松地切换我们在所有端点上所需的内容，比如确保不能访问任何私有项目。

#### 序列化器

`app/serializers/activity_pub/`中的序列化器是我们实现的核心部分，它们提供了ActivityStreams对象。抽象类`ActivityPub::ActivityStreamsSerializer`承担了验证开发者提供的数据、设置公共字段和提供分页的繁重工作。

那个分页部分是通过`Gitlab::Serializer::Pagination`完成的，它使用偏移分页。[我们需要允许它进行键集分页](https://gitlab.com/gitlab-org/gitlab/-/issues/424148)。

#### 订阅

通过将[FOLLOW活动](https://www.w3.org/TR/activitystreams-vocabulary/#dfn-follow)发布到角色收件箱来订阅资源。当收到跟随活动时，[我们应该返回生成接受或拒绝活动](https://www.w3.org/TR/activitypub/#follow-activity-inbox)，发送到订阅者的收件箱。

实施的一般工作流程如下：

+   发送一个 POST 请求到收件箱端点，其 Follow 活动以 JSON 编码

+   如果接收到的活动不是支持的类型（例如，有人试图评论该活动），我们会忽略它；否则：

+   我们创建一个带有订阅者个人资料 URL 的 `ActivityPub::Subscription`

+   我们排队一个作业来解析订阅者的收件箱 URL

    +   在其中，我们执行 HTTP 请求到订阅者的个人资料以查找其收件箱 URL（如果有的话还有共享收件箱 URL）

    +   我们将该 URL 存储在订阅记录中

+   我们排队一个作业来接受订阅

    +   在其中，我们执行 HTTP 请求到订阅者的收件箱以发布一个接受活动

    +   我们更新订阅的状态为 `:accepted`

`ActivityPub::Subscription` 是一个新的抽象模型，从中继承了与我们的参与者相关的模型，每个模型都有自己的表：

+   ActivityPub::ReleasesSubscription，表 `activity_pub_releases_subscriptions`

+   ActivityPub::TopicSubscription，表 `activity_pub_topic_subscriptions`

+   ActivityPub::ProjectSubscription，表 `activity_pub_project_subscriptions`

+   ActivityPub::GroupSubscription，表 `activity_pub_group_subscriptions`

+   ActivityPub::UserSubscription，表 `activity_pub_user_subscriptions`

选择多个模型而不是，比如，在 Subscription 模型中使用一个更简单的 `actor` 枚举与单个表一起的原因是因为我们需要为每个模型进行特定的关联和验证（`ActivityPub::ProjectSubscription` 属于项目，而 `ActivityPub::UserSubscription` 不属于）。这也为我们未来的扩展提供了更多的空间。

#### 取关

当接收到[撤销活动](https://www.w3.org/TR/activitypub/#undo-activity-inbox)提到之前的 Follow 时，我们会从数据库中删除该订阅。

我们不需要发送任何活动，所以我们在这里不需要任何工作程序，可以直接从数据库中删除记录。

#### 发送活动

当特定事件（哪些事件？）与我们的参与者相关时，我们应该排队事件以在订阅者的收件箱中发布活动（活动与我们在参与者的发件箱中显示的活动相同）。

我们应该对订阅者列表进行去重，以确保我们不会将活动两次发送给同一个人 - 虽然在接收 Follow 活动时通过模型的唯一性验证可能更好处理。

更重要的是，我们应该对同一主机的请求进行分组：如果十个用户都在 `https://mastodon.social/` 上，我们应该在提供的共享收件箱上发出单个请求，将所有用户添加为收件人，而不是每个用户发送一个请求。

Mastodon [要求实例实现 Webfinger 协议](https://docs.joinmastodon.org/spec/webfinger/)。该协议是关于在一个已知位置添加一个端点，该端点允许查询资源名称并将其映射到我们想要的任何 URL（基本上，它用于发现）。Mastodon 使用这个来查询其他 Fediverse 应用程序的 actor 名称，以便找到它们的配置文件 URL。

实际上，GitLab 已经通过 Doorkeeper 实现了 Webfinger 协议端点（[这是映射到其路由的操作](https://github.com/doorkeeper-gem/doorkeeper-openid_connect/blob/5987683ccc22262beb6e44c76ca4b65288d6067a/app/controllers/doorkeeper/openid_connect/discovery_controller.rb#L14-L16)），在 GitLab 中实现了 [JwksController](https://gitlab.com/gitlab-org/gitlab/-/blob/efa76816bd0603ba3acdb8a0f92f54abfbf5cc02/app/controllers/jwks_controller.rb)。

这里没有不相容，我们可以简单地扩展这个控制器。尽管我们可能需要重命名它，因为它不再仅仅与 Jwks 相关。

我们可能会遇到的一个困难是，与 Mastodon 相反，我们不仅仅处理用户。所以我们需要想出一些方法来区分请求用户和请求项目，例如。一个明显的方法是使用前缀，如`user-<username>`，`project-<project_name>`，等等。当我们实际实施代码并且我还没有深入研究 Webfinger 的规范时，我正在远处思考这个问题，这个评论可能在我们达到实际实施时就过时了。

Mastodon [要求使用 HTTP 签名](https://docs.joinmastodon.org/spec/security/#http)，这是另一个标准，为了确保没有垃圾邮件发送者试图冒充给定服务器。

这是非对称加密，有一个私钥和一个公钥，就像 SSH 或 PGP 一样。我们将需要实现签名请求和验证它们。当我们以后想要各种 GitLab 实例进行通信时，这将非常有助于我们。

### 主机 allowlist 和 denylist

为了让 GitLab 实例所有者控制潜在的垃圾邮件，我们需要允许维护两个互斥的主机列表：

+   allowlist：只有列在此列表中的主机才能进行联邦。

+   denylist：所有主机都可以进行联邦，但在该列表中提到的主机除外。

一个设置应该允许所有者在 allowlist 和 denylist 之间切换。起初，这可以在 rails console 中管理，但最终需要在管理员界面中有一个部分。

### 限制和推出

为了在前几个月发布功能时控制负载，我们将设置 `gitlab.com` 使用 allowlist 并逐步向几个 Fediverse 服务器推出联邦，以便我们可以看到它如何逐步承担负载，最终切换到 denylist（注：关于 `gitlab.com` 是否应该激活联邦正在进行[一些讨论](https://gitlab.com/gitlab-org/gitlab/-/issues/426373#note_1584232842)）。

我们还需要实施限制，以确保联邦系统不被滥用：

+   限制资源可以接收的订阅数量。

+   限制第三方服务器可以生成的订阅数量。

### 跨实例问题和合并请求部分

在设计这部分之前，我们将等待社交关注部分完成，以便在使用ActivityPub时有实际经验。
