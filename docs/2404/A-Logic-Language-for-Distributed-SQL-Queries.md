<!--yml

category: 未分类

日期：2024-05-27 13:31:00

-->

# 用于分布式 SQL 查询的逻辑语言

> 来源：[https://www.osohq.com/post/logic-language-distributed-sql-queries](https://www.osohq.com/post/logic-language-distributed-sql-queries)

如果您曾经使用过微服务，您就会知道解决共享能力的难度有多大。授权就是这样一种能力。大多数服务都需要某种形式的授权，因此您希望有一个干净的授权实现，每个服务都可以重复使用。但授权作为一个领域却是*混乱的*。授权逻辑与特定于领域的信息混在一起，用于评估授权决策的数据与应用程序数据有很大的重叠。

这意味着当试图将授权提取到自己的服务中时，很难实现清晰的分离。

授权作为一个领域模糊了多个应用程序之间的界限，使得难以将其提取为一个服务。

我们理想中的授权版本作为一个共享能力，强化了这些界限，因此您可以将其与任何给定应用程序集成，而不需要额外的开销。一个关键考虑因素是，您不应该需要将数据同步到不同的服务，只是为了评估授权逻辑，这些逻辑本可以全部在本地处理。

为此，我们构建了一个用于表达授权的逻辑语言称为 Polar。它被设计为允许多个团队构建一个表达共享和特定领域逻辑的授权模型。

然后我们使得可以在多个数据存储中运行 Polar，这样它可以使用集中存储的共享数据和与应用程序并行存储的应用程序数据来进行授权。

在分布式授权中，我们尽可能将尽可能多的授权*提取*到一个共享服务中，但让每个服务都可以自行管理一些授权

我们称这种结果架构为*分布式授权*。这是我们如何构建它的方式。

### 情景：面向服务的 GitHub

想象一下，你正在构建一个像GitHub这样的应用，但是采用面向服务的架构。像“存储库问题”这样的应用特定功能往往会与像“管理组织和用户角色”这样的授权和权限控制解耦。

假设你有两个服务：**AuthzService** 和 **IssuesService**

图表显示我们的两个服务、它们的数据库以及它们管理的数据列表。

AuthzService了解关于组织的所有授权相关信息：谁属于这些组织、他们拥有什么角色、他们创建了什么自定义角色。它还管理有关存储库的授权数据，例如哪个组织拥有存储库、成员是谁以及他们拥有什么角色。

与此同时，IssuesService处理与问题相关的*所有*内容：谁创建了问题、它属于哪个存储库等等。

举个例子：`用户Alice`想要关闭`问题＃42`，它位于存储库`acme/anvil`上。IssuesService想知道Alice是否被允许这样做，并将向AuthzService发出API请求以获取答案。

这是一个简单的问题，但在像GitHub这样成熟的应用中，逻辑变得复杂。例如，GitHub具有组织角色、可自定义的默认存储库角色、自定义角色、问题创建者以及公共/私有存储库。所有这些都影响授权逻辑：

角色、关系和属性会影响Alice是否能关闭问题＃42。

而授权不仅限于像这样的简单的是/否问题。正如[*Hacker News上的wkirby所说*](https://news.ycombinator.com/item?id=40054080)：

> 权限具有三个移动部分，谁想要做什么，他们想要做什么，以及在什么对象上。任何良好的权限系统都必须能够有效地回答这些变量的任何排列。给定此人和此对象，他们可以做什么？给定此对象和此操作，谁可以执行它？给定此人和此操作，他们可以对哪些对象进行操作？

这在所有上述复杂性中起到了一种乘数作用。

综上所述，此应用程序的共享授权功能必须能够：

1.  表达上述跨 Authz 和 Issues 领域的逻辑

1.  使用存储在 AuthzService 和 IssuesService 中的数据

1.  支持是/否问题以及更灵活的“列表”问题

让我们从逻辑开始。从那里开始，我们将解释我们的逻辑方法如何帮助我们解决一些棘手的数据问题。我们将通过应用这些概念到是/否和列表问题的示例来演示这些概念。

## Polar：一个受 Prolog 启发的语言，转译为 SQL

Polar 是我们的声明性授权逻辑，跨多个服务。Polar 受逻辑编程语言如 Prolog、Datalog 和 miniKanren 的启发。在底层，Polar 转译为 SQL，基于自定义数据模型，从而执行查询规划和快速数据访问。

这里有一个相当简单的 Polar 策略示例：

```
allow(user: User, "read", org: Organization) if   has_role(user, "member", org); 	
has_role(User{"alice"}, "member", Organization{"acme"});
```

存在一个 `allow` ”rule”，其内容如下：

1.  一个 `user`（类型为 `User`）被允许 `read` 一个 `org`（类型为 `Organization`）

1.  *如果* `user` 在该 `org` 上有 `member` 角色

还有一个孤立的 `has_role` 规则，没有 *if* 和没有条件。这些称为 *facts* —— 无条件真实信息。在这种情况下，事实是断言具有 `id: alice` 的用户是 `id: acme` 组织的成员。

此 Polar 策略表达了：

1.  一个简单的基于角色的访问控制逻辑：org 成员可以读取该组织

1.  一小段事实数据，将 `alice` 分配为成员角色。

要使用这个策略，你可以*查询*它。其理论基础来自逻辑编程和一阶逻辑。如果想深入了解，建议查看[Prolog 的威力](https://www.metalevel.at/prolog/introduction)。

今天，你需要理解逻辑编程的两个主要方面：

1.  推理逻辑（如何评估 Polar 规则）

1.  统一（Polar 的核心“模式匹配算法”）

### 插曲：逻辑编程

规则表达了关于世界的逻辑属性：“如果这些其他事情也是真的，那么这个声明就是真的”。规则的语法使它们看起来有点像函数，但它们在逻辑上/声明性地解决，而不是命令式地。不过，仍然有帮助思考“调用”规则。

**推理**说：“给定一个用户`alice`，她在`acme`组织拥有`member`角色，因此`allow(alice, "read", acme)`是真的。也就是说，我们根据`has_role`条件的满足推断`allow`规则是真的。

**统一**实际上是一种复杂的模式匹配算法，将变量赋值和比较合并为一个操作符。

例如，以下都是简单统一的示例：

```
1 = 1  # two concrete values, behaves like comparison true  1 = 2  # false  
x = 1  # behaves like assignment, the variable `x` now has value 1  
x = 1  # `x` is now 1  y = x # true, the variable `y` is equal to 1  
y = x # true, the variable `y` is equal to variable `x`  x = 1  # true, `x` is now 1, `y` is also 1
```

统一是一种帮助你*解决*这些方程的过程。给定 x=1 和 y=x，那么 y 是多少？显然是 1。这就是统一在做的事情。

### Polar 中的逻辑编程

将规则推理和统一结合在一起，你将得到一个可以进行简单推断的逻辑引擎：

评估针对 Polar 策略的查询所采取的逻辑步骤

### 查询 Polar 策略

下面是如何使用`oso-cloud`CLI查询策略的示例：

```
> tee -a intro.polar <<EOF allow(user: User, "read", org: Organization) if  has_role(user, "member", org);   has_role(User{"alice"}, "member", Organization{"acme"}); EOF  
> oso-cloud policy intro.polar
Policy successfully loaded.

> oso-cloud query allow User:alice read Organization:_ allow(User:alice, String:read, Organization:acme)
```

`oso-cloud query`的输出显示了逻辑推断的结果：它“填充”了`Organization:acme`的缺失参数。

而且你可以扩展这一点：

```
> tee intro.polar <<EOF allow(user: User, "read", org: Organization) if  has_role(user, "member", org);   has_role(User{"alice"}, "member", Organization{"acme"}); has_role(User{"bob"}, "member", Organization{"megacorp"}); EOF  
> oso-cloud policy intro.polar && oso-cloud query allow User:_ _ Organization:_
Policy successfully loaded.
allow(User:alice, String:read, Organization:acme) allow(User:bob, String:read, Organization:megacorp)
```

这次有一个额外的事实 — 用户`bob`是`megacorp`的`member`。

而且查询更加通用——所有用户在任何组织上都有任何权限。与上述相同的推理过程一样，Oso Cloud确定`alice`可以读取`acme`，而`bob`可以读取`megacorp`。

这里展示了逻辑语言的一个特别好的元素：您可以使用完全相同的声明性配置来回答“Alice是否可以读取Organization:acme”以及“Alice可以访问的所有组织是哪些”。这正是您在常见授权用例中所需的。

### 使用推理来去除冗余

现在假设您通过定义具有以下权限的成员来引入更多角色和更多权限：

```
allow(user: User, "read", org: Organization) if   has_role(user, "member", org); 
allow(user: User, "create_repository", org: Organization) if   has_role(user, "member", org);
```

还有一个带有略微增加权限集的管理员：

```
allow(user: User, "read", org: Organization) if   has_role(user, "admin", org); 
allow(user: User, "create_repository", org: Organization) if   has_role(user, "admin", org); 
allow(user: User, "invite_users", org: Organization) if   has_role(user, "admin", org);
```

您可以看到它开始有点重复了。因此，在任何有意义的地方编写逻辑以消除冗余是有意义的。在上述情况下，管理员可以做任何成员可以做的事情，因此对于管理员权限，您可以写：

```
- allow(user: User, "read", org: Organization) if  -	    has_role(user, "admin", org);  -  - allow(user: User, "create_repository", org: Organization) if  -	    has_role(user, "admin", org);  
+ has_role(user: User, "member", org: Organization) if  +    has_role(user, "admin", org);  
allow(user: User, "invite_users", org: Organization) if
	has_role(user, "admin", org); 
```

也就是说，您使用推理逻辑来表示如果用户具有“管理员”角色，则在组织中用户具有“成员”角色。规则推理逻辑是*递归*的。通过添加这一规则，您现在知道依赖于用户是组织成员的任何规则也将适用于组织管理员。这样一来，您就无需重复所有成员权限的操作，也无需记住每次向成员添加新权限时都要这样做。

顺便说一句，上述用例非常常见，Polar有一些内置的原语来表达它。您可以将其写为：

```
actor User { } 
resource Organization {
 roles = ["admin", "member"];
 permissions = ["read", "create_repository", "invite_users"];

  "read"  if  "member";
  "create_repository"  if  "member";

  "member"  if  "admin";
  "invite_users"  if  "admin"; }
```

这样会少一些噪音，同时还包含验证逻辑，以防止名称拼写错误。

在幕后，那些`"read" if "member"`规则扩展到您之前写的相同内容

```
#                          "read"                              if  #                            \/  has_permission(user: User, "read", organization: Organization) if  #                "member"               ;  #                   \/   has_role(user, "member", organization);
```

这纯粹是语法糖。

### 将所有内容汇总

回到GitHub的例子，之前的完整逻辑大致如下：

```
actor User {} 
resource Organization {
 roles = ["member", "admin"]; }

resource Repository {
 roles = ["reader", "triage", "admin"];
 permissions = ["close_issues"];
    relations = {
  parent: Organization,    };

  "admin"  if  "admin" on "parent";
  "close_issues"  if  "triage";
  "triage"  if  "admin"; }

has_role(user: User, role: String, repository: Repository) if  org matches Organization and  has_relation(repository, "parent", org) and  has_role(user, "member", org) and     default_repository_role(org, role);

resource Issue {
 permissions = ["read", "close"];
    relations = {
  parent: Repository,  creator: User,    };
  "close"  if  "creator"  and  "read" on "parent";
  "close"  if  "close_issues" on "parent";
 }

 has_permission(user: User, "close_issues", repository: Repository) if  org matches Organization and  has_relation(repository, "parent", org) and  role matches CustomRole and  has_relation(role, "parent", org) and  has_role(user, role, repository) and  grants_permission(role, "Repository", "close_issues");

grants_permission(role: Role, resource_type: String, permission: String) if  inherited_role matches String and  inherits_role(role, inherited_role) and     grants_permission(inherited_role, resource_type, permission);
```

在这个策略中，实际上有 *五* 种不同的方式（不包括它们的组合爆炸），用户可能获得关闭问题的权限：

1.  他们是问题的创建者，并且他们对仓库具有阅读访问权限

1.  他们直接在所属仓库上拥有适当的角色（例如“triage”）

1.  他们直接在仓库上拥有自定义角色，并且组织配置了该角色在仓库上具有“close_issues”权限

1.  他们是组织成员，并且组织配置成员在所有仓库上至少具有“triage”级别的访问权限

1.  他们是组织管理员，这使他们在所有仓库上具有管理员级别的访问权限。

哦！

逻辑编程作为授权的强大范式之一在于能够实现此类分支条件逻辑，这通常在授权系统中见到，使用可重用的逻辑谓词（即更多规则）。Polar 的逻辑编程基础使其能够清晰而简洁地表示这种逻辑。

更重要的是，通过将授权逻辑从各个服务中抽出并放入一个共同的位置，多个团队可以共同协作策略。因为策略语言是声明性的，规则易于阅读，逻辑易于理解，进一步促进了协作。

例如，要扩展先前的策略以使“关闭已锁定问题”成为一个独立的权限，您可以进行小的策略更改：

```
 resource Issue {    permissions = ["read", "close"];
    relations = {
 	    parent: Repository,
 	    creator: User,
 	  };
-   "close" if "creator" and "read" on "parent";  -   "close" if "close_issues" on "parent";  +   "close" if "creator" and "read" on "parent" and is_locked(resource, false);  +   "close" if "close_issues" on "parent" and is_locked(resource, false);  +   "close" if "close_locked_issues" on "parent";  }
```

### 从逻辑到数据：将事实作为 SQL

将授权逻辑表达在像 Polar 这样的语言中只是一半的战斗。它为您提供的是一种一致的方法来编写可重用的授权逻辑，多个团队可以共同协作建设。

然而，我们故意跳过数据：它来自何处以及它如何影响性能。

早些时候，我们展示了如何通过直接在策略中写入事实来添加数据：

```
has_role(User{"alice"}, "member", Organization{"acme"});
```

从逻辑编程的角度来看，这没有任何固有的问题。没有真正的需要区分规则和事实 —— 记住，事实只是一个无条件成立的规则。关键在于实现提供方便的 API 来通过添加/删除新事实来修改系统状态，并实现有效的存储/索引/查找事实的方式。

### 存储事实

[Datalog](https://en.wikipedia.org/wiki/Datalog) 很好地说明了这一点。Datalog 是一种逻辑编程语言，将规则与事实区分开来作为其设计的基本元素之一。不同的 Datalog 实现之间的区别在于它们用于评估规则和事实的策略：[https://en.wikipedia.org/wiki/Datalog#Evaluation](https://en.wikipedia.org/wiki/Datalog#Evaluation)。

实际上，我们本可以使用 Datalog 来实现我们的数据目标 —— 但这意味着我们必须构建我们自己的 Datalog 实现、支持数据存储等等。我们不想这样做。

我们*希望*利用现有的经过验证的技术 —— 比如 SQL。更具体地说，是 SQLite。

因此，我们决定：我们应该有一个专为 Polar 设计、由 SQL 支持的自定义数据模型。我们已经经历了几次迭代的数据模型，并最终采用了一个痛苦简单的模型：我们将事实存储为 SQL 表中的行。

最初，我们将所有具有相同元数（参数个数）的事实放在一起放在一个表中。但是我们意识到，使用 SQLite，为特定事实*类型*动态创建表会更好，我们定义事实名（例如“has_role”）和参数类型为事实类型。

在 Polar 中，数值要么是原始类型（字符串、整数、布尔值），要么是类型 + ID 对，例如 `{ type: "Organization", id: "acme" }`。我们也可以用这种格式表示原始类型（例如 `{ type: “String”, id: “member”}`）。

这给我们留下了一个事实，例如：

```
has_role(User{"alice"}, "member", Organization{"acme"});
```

我们在事实索引中创建一个条目，它将递增的ID映射到事实类型，例如。

```
1 -> has_role, [User, String, Organization]
```

然后将事实值（即引用部分）的id存储在fact_1表中：

```
arg0 | arg1   | arg2 ---------------------
alice | member | acme
```

### 检索事实

现在问题变成：Polar应该如何访问这些数据？有两种值得考虑的选择：

1.  使得可以在查询评估的一部分访问后端数据存储。

1.  将评估分为两个阶段：查询评估和数据查找

我们最终选择了（2），因为我们在开始时就知道我们正在朝着可以支持数据分割到多个服务的方向发展，因此我们需要将这两个步骤分开。

工作原理是：当Polar评估类似于`allow User:alice read Organization:_`的查询时，它基本上像普通的逻辑编程语言一样工作。它执行统一并应用推理逻辑来找到输入变量可能的所有可能性 - 唯一的重要区别是。

当Polar看到这样的条件时：

```
has_role(user, "member", organization)
```

它考虑了两种不同的可能性：

1.  条件由某些其他规则满足，因此应找到匹配的规则并评估这些规则

1.  条件由存储在底层数据库中的具体事实满足。在这种情况下，将其跟踪为稍后解决的内部状态。

因此，与其像以前那样Polar返回具体的结果，实际上Polar正在返回一个中间表示，显示数据库中的事实可以如何满足查询的所有方式。

对于上述规则“如果用户具有存储库中的‘triage’角色，则用户可以关闭问题”的示例规则，查询“Alice可以关闭哪些问题”将产生中间表示：

```
Outputs:   issue:  f0.arg0  
Facts:   f0:  has_relation(Issue,  String,  Repository)   f1:  has_role(User,  String,  Repository)  
Conditions:   f0.arg2  =  f1.arg2   f0.arg1  =  "parent"   f1.arg1  =  "triage"   f1.arg0  =  User{"alice"}
```

这看起来很像SQL的`SELECT ... FROM ... WHERE`语法并非巧合。该过程的最后一步是将上述内容转换为SQL，并查询我们嵌入的SQLite数据库。

对于索引为 `f0` 和 `f1` 的事实，我们通过索引查找相应的事实表。为简单起见，我将假设这些表是 `fact_0` 和 `fact_1` 。SQL 输出如下：

```
SELECT f0.arg0 FROM  fact_0 f0,
 fact_1 f1
WHERE  f0.arg2 = f1.arg2 and  f0.arg1 =  'parent'  and  f1.arg1 =  'triage'  and  f1.arg0 =  'alice';
```

### 插曲：性能。

鉴于使用 SQL 的动机之一是性能，这个方法表现如何？

因为我们的事实存储在 SQL 表中，我们可以使用简单的启发式方法为这些表生成大量索引：

```
CREATE  UNIQUE INDEX fact_0_idx_0 on fact_0 ( arg0, arg2, arg1 ); CREATE  UNIQUE INDEX fact_0_idx_1 on fact_0 ( arg2, arg0, arg1 ); 
CREATE  UNIQUE INDEX fact_1_idx_0 on fact_1 ( arg0, arg2, arg1 ); CREATE  UNIQUE INDEX fact_1_idx_1 on fact_1 ( arg2, arg0, arg1 );
```

我们的 SQL 查询最终会是一些连接和对这些索引列的简单过滤。这使得 SQLite 查询规划器非常满意：

```
QUERY PLAN |--SEARCH f0 USING COVERING INDEX fact_0_idx_0 (arg0=?)
`--SEARCH f1 USING COVERING INDEX fact_1_idx_1 (arg2=?)
```

它确实做到了你所期望的：找到 Alice 担任“筛选”角色的所有仓库，然后获取这些仓库中的所有问题。

在我们询问 Alice 是否可以关闭特定问题的情况下呢？SQLite 查询规划器足够聪明，能够意识到问题严格属于一个仓库，而用户可能在多个仓库上拥有角色，并反转连接的顺序：

```
QUERY PLAN |--SEARCH f1 USING COVERING INDEX fact_1_idx_0 (arg0=?)
`--SEARCH f0 USING COVERING INDEX fact_0_idx_0 (arg0=?)
```

### Polar + Facts：一个理想的组合。

到这一点上，我基本上只是在向你解释为什么 SQL 是很酷的... 但很难过分强调我们能够完全专注于向像 SQLite 这样的经过实战检验的 SQL 引擎提供良好的输入，而不是试图构建这个过程。我们可以把创新的筹码花在其他地方。

我们还在相对清晰的界限上划出了一条线。Polar 支持：

1.  常见的授权数据：角色，关系，属性。

1.  表达为事实的自定义数据类型。

1.  任意组合的事实，通过 `and` 和 `or` 表达。

1.  几个比较操作，例如整数比较和基于时间的检查（用于过期）。

1.  递归数据查找（例如，类似谷歌驱动器的文件和文件夹层次结构）。

我们认为这是理想的组合：足够灵活以表示几乎任何东西，同时结构化到足以编写使用 SQLite 编写强大的查询层来进行底层查询规划和数据查找。

回顾过去，SQL在类似Datalog的逻辑编程语言中运行得如此顺畅并不奇怪。SQL本身就受到逻辑编程的启发，而Datalog经常被描述为逻辑编程和关系代数之间的一个过渡步骤。当你越看越多时，所有的东西开始变得模糊不清。

## 分发查询

使用Polar，我们有一种声明性的方法来表达授权逻辑。我们已经看到，我们可以在SQL中使用自定义数据模型来快速评估查询。我们用它来构建我们的中心授权系统（Oso Cloud）。

返回到我们的 IssuesService 示例：如果我们想要使用那个中心授权系统来决定“Alice是否可以关闭问题 #42”，我们需要以某种方式使其访问所需的数据。

通常，集中式授权系统（包括Oso Cloud）已通过要求在做出决定时授权系统中存在任何必要数据来实现这一点。这意味着要么

1.  在应用程序和授权服务之间复制与授权相关的数据

1.  转向事件溯源

这两者听起来都是为了回答一些授权问题而进行了大量额外工作。在数据复制的情况下，您要么构建一个复制过程，要么为每个授权请求通过网络传递额外的数据。如果您转向事件溯源，那么您正在进行主要的应用程序重构。

无论您选择如何将数据引入服务中，某些操作还将从中返回长列表的数据。回想一下我们要求所有Alice可以关闭的问题的情况。如果Alice可以访问数百个问题，那么响应将包含所有这些问题，增加延迟。

这使我们考虑了一个不同的选择：分发授权。我们可以将所有多服务共享的内容集中起来，而将其他所有内容留在应用程序中。

例如，在GitHub的示例中，用户在问题上的大部分权限来自于仓库角色，因此实施的一个路径可能是：

```
// issuesService/controller/closeIssue.ts  
async  function  closeIssue(currentUser: User, issueData: IssueData) {
  // Note: we should still check for read permission on the repo   // even if the user is the creator -- you cannot   // close issues for repositories you don't have access to   if (!canRead) {  throw  new PermissionDeniedError();  }

  if (!canClose) {  // Implement issue-level authorization logic directly in the issue service.   //   // unless you have the repository-level canClose permission,   // you need to be the issue creator   const issueCreator = await db .selectFrom("issue")
 .select("creator_id")
 .where("id", "=", issueData.id)
      .execute();
  if (issueCreator[0] !== currentUser.id) {
  throw  new PermissionDeniedError();    }
  }

  // close the issue  } 
```

那么，坦白地说，这并不算太糟糕。GitHub 的大部分功能依赖于用户在组织/仓库权限上的设置，因此您需要在共享授权服务中管理这些逻辑和数据。但是，像“问题创建者可以关闭自己的问题”这样的领域特定授权逻辑则需要在服务内部（即IssueService）实现，并且利用您已有的应用数据。

不过我们可以做得更好。上述代码有一个明显的缺点：大量授权逻辑已经渗入到IssuesService中。

这很糟糕，因为它在AuthzService和IssuesService之间创建了耦合。

假设您想修改授权逻辑以区分哪些用户可以关闭*锁定*的问题。可能看起来像这样：

1.  在IssuesService中添加代码，用于检查用户是否具有`"close_locked_issues"`权限。在功能标志后台部署这些更改。

1.  在AuthzService中为角色添加新的权限，然后重新部署这些更改。

1.  切换IssuesService的功能标志以启用新功能。

一个服务中的微小变化需要跨多个服务（因此跨多个团队）协调代码更改。

与此同时，通过将领域特定的授权延迟到关联服务，每个团队都可以在其应用程序中“完成画画的剩余部分”。例如，实施“创建者可以关闭问题”时，Issues团队很幸运地记得检查问题创建者仍然对仓库具有读权限（已经离开公司的员工不应能够关闭他们的旧问题）。但是这种逻辑是在AuthzService之外，因此您失去了确保该行为正确实施和测试的大部分工具。

### 清理混乱

正如我们在开头所说的：授权是混乱的。

要决定谁可以对哪些资源做什么，您需要了解这些资源是什么，以及它们如何与其他资源相关联。但是您不希望授权逻辑泄漏到您的应用程序中，也不希望在授权服务中重复应用程序数据。这意味着您需要一种方法在 AuthzService 和 IssueService 之间分发授权决策。

上述代码示例让我们接近目标，但是一个解决方案可以消除泄露的授权逻辑，并保持领域特定数据封装性将更具抗变化逻辑或数据变化的弹性。实际上，这意味着：

1.  将所有授权逻辑集中在授权服务中

1.  在授权服务中集中*共享*的授权数据（例如组织和存储库角色）

1.  将 *领域特定* 的授权数据保存在适当的应用程序数据库中

### 对一个应用程序数据库评估 Polar

我们通过将 Polar 的能力引入存储在多个数据存储中的数据来实现这一点。而且重要的是，这些数据存储在 Oso Cloud *之外*。

我们之前有 [将 Polar 转换为 SQL](https://www.osohq.com/post/authorization-logic-into-sql) 的经验，它可以在应用程序内运行。然而，以前我们依赖 ORM 来构建查询。我们返回了一个受关系代数启发的中间表示，并使用 ORM 将其转换为 SQL。

但是考虑到我们已经在使用一个足够灵活的中间表示来转换为 SQL，当我们再次思考这个问题时，我们看了看我们已经拥有的东西。

我们需要做两件事情：

1.  将我们的中间表示转换为 SQL，以便在应用程序数据库中运行。

1.  将中间表示分割成一个在中心运行的 Oso 部分和一个在应用程序中本地运行的部分。

一个相对简单的做法（1），虽然有些天真，但接近我们的需求，是使用 [CTEs](https://www.metabase.com/learn/sql-questions/sql-cte) 来使应用程序数据库“看起来像”我们自己的内部架构，例如在前面的查询中，用描述如何获取数据的 CTE 替换 `fact_0` 和 `fact_1`：

```
WITH fact_0(arg0, arg1, arg2) as (  SELECT id, 'parent', repository_id FROM    issues
), fact_1(arg0, arg1, arg2) as )  SELECT user_id, "role", repository_id FROM    repository_roles
)
SELECT f0.arg0 FROM  fact_0 f0,
 fact_1 f1
WHERE  f0.arg2 = f1.arg2 and  f0.arg1 =  'parent'  and  f1.arg1 =  'triage'  and  f1.arg0 =  'alice';
```

要实现这一点，我们只需要一个配置文件，该文件在 Oso 数据架构和 SQL 表格之间进行映射，例如：

```
facts:   has_relation(Issue:_,  String:_,  Repository:_):   query:  |-
 SELECT id, 'parent', repository_id FROM  issues   has_role(User:_,  String:_,  Repository:_):   query:  |-
 SELECT user_id, "role", repository_id FROM  repository_roles
```

将应用程序数据库映射到 Oso 事实的简单映射不是偶然的。我们有一个简单的规则来将数据映射到 Oso：

1.  角色：多对多或一对多的表格，它们关联两个表/主键，并具有额外的“角色”属性。

1.  关系：一个主键 + 一个外键

1.  属性：一个主键 + 一个列值

我们告诉我们的客户使用这个简单的映射发送 Oso 数据，因此我们所有的授权规则的常见模式都是基于这些数据表达的。

“问题”表具有主键，父存储库和问题创建者的外键，以及一个列用于标识是否被锁定。这些组合在 Oso 中映射为关系和属性数据。

我们创建的映射是一个“虚拟”的映射。实际上，我们无需以任何方式修改数据库，也不需要持久化任何数据。但是，如果我们要将数据转换为 Polar 所看到的形式，或者将我们生成的 CTEs 展开为实际表格，它会看起来像这样：

“问题”表中的每一行都映射到多个事实。

就像那样，我们有了一个可以在应用程序数据库上运行的策略语言，只需最少的配置。

*注意：我想澄清这里的意图是说 Oso 用户可以很容易做到这一点，并不是要贬低实现这一功能的 Oso 团队的努力 🐻*

### 将查找拆分成两个服务

现在我们有了（1）—— 一种针对应用程序数据库运行 SQL 的方法。

让我们继续（2）—— 将中间表示拆分为 Oso 部分和本地部分。

一旦我们弄清楚了配置以及如何将其编译为SQL，并决定如何“拆分”中间状态，就很清楚了。

我们将正在搜索的事实分成Oso部分和本地部分，并创建虚拟变量来连接这两者，例如，我们知道`issuesService`管理问题数据，因此问题与存储库之间的`has_relation`事实，而Oso管理角色

**拆分之前：**

```
Outputs:   issue:  f0.arg0  
Facts:   f0:  has_relation(Issue,  String,  Repository)   f1:  has_role(User,  String,  Repository)  
Conditions:   f0.arg2  =  f1.arg2   f0.arg1  =  "parent"   f1.arg1  =  "triage"   f1.arg0  =  User{"alice"} 
```

**拆分之后：**

```
### Oso part:  
Outputs:   var0:  f1.arg2  
Facts:   f1:  has_role(User,  String,  Repository)  
Conditions:   f1.arg1  =  "triage"   f1.arg0  =  User{"alice"}  
### Local part  
Outputs:   issue:  f0.arg0  
Facts:   f0:  has_relation(Issue,  String,  Repository)  
Conditions:   f0.arg2  =  inputs.var0   f0.arg1  =  "parent"
```

在Oso的一侧，我们以与之前相同的方式评估Oso部分：我们将其编译为SQL，运行SQL，并获取一组结果。

要在应用程序内解决其余的授权结果，我们需要采取：

1.  中间表示的另一半

1.  模式映射配置

1.  来自Oso的部分评估结果

并将它们转换为我们可以运行的SQL查询。

也就是说，我们结合了：

所需的查找：

```
Outputs:   issue:  f0.arg0  
Facts:   f0:  has_relation(Issue,  String,  Repository)  
Conditions:   f0.arg2  =  inputs.var0   f0.arg1  =  "parent"
```

加上配置：

```
has_relation(Issue:_,  String:_,  Repository:_):   query:  |   select  id,  repository_id  from  issues
```

和来自Oso的输入：

```
inputs:  ('anvil',  ...)
```

计算执行“其余授权”的SQL：

```
WITH fact_0(arg0, arg1, arg2) as (  SELECT id, 'parent', repository_id FROM    issues
)
SELECT f0.arg0 FROM  fact_0 f0
WHERE  f0.arg2 in ('anvil', ...);
```

IssueService可以针对自己的数据库运行此SQL来检查单个实体的授权：

```
 // get "Local Part" from Oso Cloud   const authorizeSql = await oso.authorizeLocal(user, "close", issue);
  const { allowed } = (   // evaluate "Local Part" against application database   await sql.raw<{ allowed: boolean }>(authorizeSql).execute(db) ).rows[0];
```

通过将中间极坐标表示转换为IssueService可以运行的SQL *，我们建立了一种访问分布在多个服务中的授权数据的方法。

有了这一切就位，`closeIssue`方法变得更简短了：

```
async  function  closeIssue(issueData: IssueData) {
  // get "Local Part" from Oso Cloud   const authorizeSql = await oso.authorizeLocal(user, "close", issue);
  const { allowed } = (   // evaluate "Local Part" against application database   await sql.raw<{ allowed: boolean }>(authorizeSql).execute(db) ).rows[0]; 
  if (!allowed) {  throw  new PermissionDeniedError();  }

  // close the issue  } 
```

您还可以使用此方法生成已授权问题的列表。例如，返回当前用户可以关闭的所有问题：

```
// get "Local Part" from Oso Cloud  const listSql = await  await oso.listLocal(user, "close", "Issue", "id") 
// evaluate "Local Part" against application database  const results = await db .selectFrom("issues")
 .where(sql.raw<boolean>(listSql))
  .selectAll()
  .execute();
```

因为您正在针对本地数据库创建列表，所以很容易对结果进行排序和分页。

```
// get "Local Part" from Oso Cloud  const listSql = await  await oso.listLocal(user, "close", "Issue", "id") 
// evaluate "Local Part" against application database  const results = await db .selectFrom("issues")
 .where(sql.raw<boolean>(listSql))  .orderBy("created_at")
 .offset(50)
 .limit(25)
  .selectAll()
  .execute(); 
```

### 结论

回顾我们的初始标准：

共享授权功能必须提供以下机制：

1.  表达上述跨Authz和Issues领域的逻辑

1.  使用存储在AuthzService和IssuesService中的数据

1.  支持是/否问题以及更灵活的“列表”问题

我们已经实现了这三点！我们有 Polar 来表达脱离任何一个服务的声明性逻辑。我们在服务之间分发 Polar，避免了需要在任何地方同步数据，而我们将 Polar 编译成 SQL 的方法使得可以提出*任何*类型的问题，而无需重写任何逻辑。

在这里尝试一下：[https://www.osohq.com/docs/guides/integrate/filter-lists#list-filtering-with-decentralized-data](https://www.osohq.com/docs/guides/integrate/filter-lists#list-filtering-with-decentralized-data)

‍
