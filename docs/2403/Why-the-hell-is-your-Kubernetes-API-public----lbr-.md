<!--yml

category: 未分类

date: 2024-05-29 12:35:59

-->

# 为什么你的Kubernetes API要公开？| lbr.

> 来源：[https://leebriggs.co.uk/blog/2024/03/23/why-public-k8s-controlplane](https://leebriggs.co.uk/blog/2024/03/23/why-public-k8s-controlplane)

你有没有真正想过如何访问你的Kubernetes控制平面？无论你使用什么机制来部署你的集群，你都会得到一个`KUBECONFIG`，通常只是简单地过度复杂化你的基础设施。

然而，如果你曾经查看过你的`KUBECONFIG`，你会看到你有一个服务器地址。

您可以通过以下方式检查集群的健康状况：

```
curl -k $(kubectl config view --output jsonpath='{.clusters[*].cluster.server}')/healthz 
```

假设一切正常运行（如果不是，请停止阅读并找出问题所在），它应返回`ok`。

你有没有意识到，从网络角度来看，这*只是工作*？很可能你根本不需要连接VPN，或者SSH到一个堡垒/跳板主机？

我怎么知道的？嗯，因为绝大多数Kubernetes集群都悬挂在公共互联网上，毫不在乎。

如果你在[shodan.io搜索Kubernetes集群](https://www.shodan.io/search?query=kubernetes)，你会看到几乎有1.4百万个集群，可以轻松访问并对可怕的世界敞开大门。

出于我无法完全理解的原因，我们似乎集体决定把我们的Kubernetes控制平面放在公共互联网上。至少，我们似乎已经决定允许它们有一个公共IP地址 - 当然，你可能会添加一些安全组或防火墙规则，限制特定IP地址的访问，但控制平面仍然可以从互联网访问。

从某种角度来看，看到这些内容，你中有多少人在考虑将他们的数据库放在公共互联网上时会感到一阵寒意？或者一个带有RDP的Windows服务器？或者一个带有SSH的Linux服务器？

传统做法普遍认为这不是个好主意，但为了使我们的生活更轻松，我们决定让每个人及其狗都自由访问并尝试将我们的集群变成他们自己的。

## 私有网络

一旦你把云中的某些东西放在私有子网中，你就增加了实际使用它的复杂性。你必须向所有需要访问它的人解释，包括刚加入团队的开发人员，它在哪里以及如何使用它。你可能会使用一个SSH隧道，或者一个堡垒实例，或者上帝原谅那个几年前设置的VPN服务器，没人敢碰的那个。

对于像数据库这样的东西，我们接受这些，因为我们很少需要访问它们，除非紧急情况，我们认为通过其他途径访问它们是可以接受的，因为其中的数据足以保护。

此外，当云提供商开始提供托管 Kubernetes 服务器时，它们大多数甚至*没有*私有控制平面。直到最近几年，他们才开始将其作为一项功能提供，即使如此，这也不是默认设置。

因为这样做*更加*容易，所以将非常重要的 API 放在互联网上的做法正在蔓延。这里真正的关注点是，我们距离每个互联网上的 Kubernetes 集群都安装一个比特币挖矿程序只有一个严重的漏洞。

## 另一个选择

我[之前](/blog/2024/02/26/cheap-kubernetes-loadbalancers.html)曾写过[Tailscale Kubernetes 操作员](https://tailscale.com/kb/1236/kubernetes-operator)及其能够将运行在 Kubernetes 集群内部的服务暴露给您的 Tailnet，但它还有另一个令人惊叹的功能：

它可以作为您集群的 Kubernetes 代理。

## 现在说什么？

好吧，假设您在 AWS 中配置了一个 Kubernetes 集群。您决定为了给自己增加另一层保护，将确保控制平面仅在 VPC 内部可访问。

如果安装了 Tailscale Kubernetes 操作员并设置了 `apiServerProxyConfig` 标志，它将在您的 Tailnet 中创建一个设备，使任何连接到 Tailnet 的人都可以访问。这意味着在您能够使用集群之前，您需要连接到 Tailnet。我之前提到的关于堡垒主机和网络的所有痛苦都将消失得无影无踪。

让我们试一试！

## 安装和使用 Tailscale 操作员

### 先决条件

您需要拥有自己的 Tailnet，并与之连接。您可以[注册 Tailscale](https://login.tailscale.com/start?utm=leebriggs.co.uk)，个人使用免费。

完成后，您需要对 ACL 进行轻微更改。Kubernetes 操作员使用 Tailscale 的标记机制，因此让我们为操作员创建一个标记，并为其向客户端设备注册的标记创建一个标记：

```
"tagOwners":  {  "tag:k8s-operator":  ["autogroup:admin"],  //  allow  anyone  in  admin  group  to  own  the  k8s-operator  tag  "tag:k8s":  ["tag:k8s-operator"],  }, 
```

然后，注册一个 OAuth 客户端，具有`Devices`写入范围和`tag:k8s-operator`标记。

### 步骤 1：获取 Kubernetes 集群

有很多种方法可以实现这一点，选择您喜欢的方式。如果您喜欢 eksctl，[此页面](https://eksctl.io/usage/eks-private-cluster/) 展示了如何创建完全私有的集群。

当然，您可以使用默认设置并尝试在公共集群上进行此操作，但我假设您这样做是因为希望使您的集群变为私有。

您可能需要从实际的 VPC 内部执行此操作，因为请记住，任何与 Kubenrnetes API 服务器交互的后安装步骤都将无法运行。我利用[Tailscale 子网路由器](https://tailscale.com/kb/1019/subnets)使这一过程更加轻松，稍后再详细介绍。

### 步骤 2：安装 Tailscale Kubernetes 操作员

最简单的方法是使用[Helm](https://tailscale.com/kb/1236/kubernetes-operator#installation)。您需要添加 Tailscale Helm 存储库：

```
helm repo add tailscale https://pkgs.tailscale.com/helmcharts
helm repo update 
```

然后，使用你的oauth密钥从前提步骤安装运营商，并启用代理：

```
helm upgrade \
  --install \
  tailscale-operator \
  tailscale/tailscale-operator \
  --namespace=tailscale \
  --create-namespace \
  --set-string oauth.clientId=${OAUTH_CLIENT_ID}> \
  --set-string oauth.clientSecret=${OAUTH_CLIENT_SECRET} \
  --set-string apiServerProxyConfig.mode="true" \
  --wait 
```

现在，如果你查看你的Tailscale仪表板，或者使用`tailscale status`，你应该会看到几个新设备 - 运算符和一个专门用于API服务器代理的服务。

现在，你可以从任何安装了Tailscale客户端的地方访问你的Kubernetes集群，不需要繁琐操作。

### 第三步：使用它

你需要使用以下命令配置你的`KUBECONFIG`：

```
tailscale configure kubeconfig <hostname> 
```

这将设置你的`KUBECONFIG`以使用Tailscale API服务器代理作为你的集群的服务器。如果你检查你的`KUBECONFIG`，你应该会看到类似这样的内容：

```
apiVersion: v1
clusters:
- cluster:
    server: https://eks-operator-eu.tail5626a.ts.net
  name: eks-operator-eu.tail5626a.ts.net
contexts:
- context:
    cluster: eks-operator-eu.tail5626a.ts.net
    user: tailscale-auth
  name: eks-operator-eu.tail5626a.ts.net
current-context: eks-operator-eu.tail5626a.ts.net
kind: Config
users:
- name: tailscale-auth
  user:
    token: unused 
```

## 等一下..

你可能有一些问题。首先，`unused`令牌是什么意思？我们的操作员安装中的`noauth`模式是什么意思？这是如何工作的？

嗯，现在如果你运行一个基本的`kubectl`命令（并假设你已连接到你的Tailnet），你会得到一些返回，但它不会帮助太多：

```
kubectl get nodes
error: You must be logged in to the server (Unauthorized) 
```

这里发生了什么？好消息是，我们已经能够路由到我们的私有Kubernetes控制平面，但我们没有发送关于我们是谁的任何信息。所以让我们在Tailscale控制台的Tailscale ACL中做一个小改动。添加以下内容：

```
 "grants":  [  {  "src":  ["autogroup:admin"],  //  allow  any  tailscale  admin  "dst":  ["tag:k8s-operator"],  //  to  contact  any  device  tagged  with  k8s-operator  "app":  {  "tailscale.com/cap/kubernetes":  [{  "impersonate":  {  "groups":  ["system:masters"],  //  use  the  `system:masters`  group  in  the  cluster  },  }],  },  },  ], 
```

通过添加这个授权，我假设你是你的Tailnet的管理员。

现在，再试一次那个`kubectl`命令。

```
kubectl get nodes
NAME                                              STATUS   ROLES    AGE     VERSION
ip-172-18-183-129.eu-central-1.compute.internal   Ready    <none>   13h     v1.29.0-eks-5e0fdde
ip-172-18-187-244.eu-central-1.compute.internal   Ready    <none>   3d18h   v1.29.0-eks-5e0fdde
ip-172-18-42-205.eu-central-1.compute.internal    Ready    <none>   3d18h   v1.29.0-eks-5e0fdde 
```

就更像是这样。

正如你在这里看到的，我解决了两个不同的问题：

+   我已经通过VPN使我的公共Kubernetes控制平面可访问，而无需担心路由和网络 - Tailscale已经为我处理了这个问题。

+   我还能够利用Tailscale的ACL机制为集群角色绑定提供认证。

### noauth模式

现在，如果你对当前的授权模式感到满意，你仍然可以使用Tailscale的访问机制来解决路由问题。在这种特殊情况下，你可以以`noauth`模式安装Operator，然后使用你的云提供商现有的机制来检索令牌。

修改你的Tailscale Operator安装如下：

```
helm upgrade \
  --install \
  tailscale-operator \
  tailscale/tailscale-operator \
  --namespace=tailscale \
  --create-namespace \
  --set-string oauth.clientId=${OAUTH_CLIENT_ID}> \
  --set-string oauth.clientSecret=${OAUTH_CLIENT_SECRET} \
  --set-string apiServerProxyConfig.mode="noauth" \ # using noauth mode
  --wait 
```

安装完成后，如果您再次运行您的`kubectl`命令，您将看到一个不同的错误消息：

```
kubectl get nodes
Error from server (Forbidden): nodes is forbidden: User "jaxxstorm@github" cannot list resource "nodes" in API group "" at the cluster scope 
```

这个原因有点显而易见，我使用的EKS集群完全不知道`jaxxstorm@github`是谁 - 因为它使用IAM来将我认证到集群。

所以让我们修改我们的`KUBECONFIG`以获取像EKS期望的令牌。我们将修改`user`部分以利用一个`exec`指令 - 它应该看起来有点像这样：

```
apiVersion: v1
clusters:
- cluster:
    server: https://eks-operator-eu.tail5626a.ts.net
  name: eks-operator-eu.tail5626a.ts.net
contexts:
- context:
    cluster: eks-operator-eu.tail5626a.ts.net
    user: tailscale-auth
  name: eks-operator-eu.tail5626a.ts.net
current-context: eks-operator-eu.tail5626a.ts.net
kind: Config
users:
- name: tailscale-auth
  user:
    exec:
      apiVersion: "client.authentication.k8s.io/v1beta1"
      command: "aws"
      args: [
        "eks",
        "get-token",
        "--cluster-name",
        "kc-eu-central-682884c" # replace with your cluster name
      ]
      env:
      - name: KUBERNETES_EXEC_INFO
        value: '{\"apiVersion\":  \"client.authentication.k8s.io/v1beta1\"}' 
```

现在我们能够路由到Kubernetes控制平面，并使用云提供商的授权机制进行身份验证。

## 一些常见问题解答

### 为什么我不只是使用子网路由器？

我经常被问到的一个常见问题是，为什么我不只是在VPC中使用子网路由器将任何东西路由到控制平面的私有地址？我在安装运营商时利用了这个机制，因为我的控制平面最初无法从互联网路由。

这是问题的一个合法解决方案，如果您不想在 `auth` 模式下使用运算符，可以继续过您的生活。但是，通过安装运算符并直接通过您的 `KUBECONFIG` 与其通信，您将获得的一个好处是能够使用 Tailscale 的 ACL 来控制谁实际上可以与运算符通信。

如果您回忆起我们之前的 `grant`，我们可以在 `auth` 模式下指定谁能够冒充用户进入集群，但是通过 Tailscale 的 ACL 系统，我们还可以对连接进行详细规定。考虑一下，如果您删除了默认的宽松 ACL

```
//{"action":  "accept",  "src":  ["*"],  "dst":  ["*:*"]},  #  commented  out  because  hujson 
```

然后定义一个可以访问集群的用户组，并添加更明确的 ACL：

```
"groups":  {  "group:engineers":  ["jaxxstorm@github"],  },  //  some  other  stuff  here  "acls":  [  //  Allow  all  connections.  //{"action":  "accept",  "src":  ["*"],  "dst":  ["*:*"]},  {  "action":  "accept",  "src":  ["group:engineers"],  "dst":  ["tag:k8s-operator:443"],  },  ], 
```

我可以对我对集群的访问更加精细化，并且在 *网络* 级别以及授权级别上拥有一个零信任模型。您的信息安全团队会对此*感到高兴*。

当您在集群中部署运算符时，您可以修改它使用的标签，进一步允许您分段您的 Tailnet，请查看[操作符配置在 Helm 图的 values.yaml](https://github.com/tailscale/tailscale/blob/main/cmd/k8s-operator/deploy/chart/values.yaml#L23)

只需确保您的 oauth 客户端具有管理这些标签的正确权限！

### 我可以在集群端限定访问范围吗？

如果您在 `auth` 模式下安装了运算符，您可以在网络级别（使用前述标签）*以及* Kubernetes RBAC 系统中限定访问。

假设我们想要将我们前面提到的工程师组仅限于名为 `demo` 的单个命名空间的访问权限。

首先，我们将创建 ClusterRole（或 Role），然后创建集群角色绑定：

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: engineers-clusterrole
rules:
- apiGroups: ["", "apps", "batch", "extensions"] 
  resources: ["*"]
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: engineers-rolebinding
  namespace: demo
subjects:
- kind: Group
  name: engineers # note the group name here
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: engineers-clusterrole
  apiGroup: rbac.authorization.k8s.io 
```

然后更新我们的 Tailscale ACL 来修改授权：

```
"grants":  [  {  "src":  ["group:engineers"],  "dst":  ["tag:k8s-operator"],  "app":  {  "tailscale.com/cap/kubernetes":  [{  "impersonate":  {  "groups":  ["engineers"],  },  }],  },  },  ], 
```

现在，如果我尝试访问 `demo` 命名空间，我可以执行我需要做的事情：

```
kubectl get pods -n demo
NAME                                     READY   STATUS    RESTARTS   AGE
demo-streamer-e3a170e4-85f4f7b88-gppcn   1/1     Running   0          16h
demo-streamer-e3a170e4-85f4f7b88-xc7gz   1/1     Running   0          16h 
```

但是不能在 `kube-system` 命名空间中：

```
 kubectl get pods -n kube-system
Error from server (Forbidden): pods is forbidden: User "jaxxstorm@github" cannot list resource "pods" in API group "" in the namespace "kube-system" 
```

## 总结

像我在这里写的帖子一样，我是以个人身份写作的，但显然我是一名 Tailscale 员工，对公司的成功有着切身利益。我希望你注册 Tailscale 并支付费用吗？当然希望。即使你不想注册 Tailscale 并支付费用，我也希望你将你的 Kubernetes 集群从公共互联网上移除。**是的**。

我一直对这些公共集群感到不安，我不禁担心我们离灾难仅一步之遥。

现在你知道将你的 Kubernetes 控制平面从互联网上移除是多么容易，你还在等什么呢？
