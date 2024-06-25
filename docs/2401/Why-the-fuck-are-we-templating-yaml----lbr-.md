<!--yml

类别：未分类

日期：2024-05-27 15:03:35

-->

# 为什么我们要对 yaml 进行模板化？| lbr.

> 来源：[https://leebriggs.co.uk/blog/2019/02/07/why-are-we-templating-yaml](https://leebriggs.co.uk/blog/2019/02/07/why-are-we-templating-yaml)

我曾在 2019 年的[cfgmgmtcamp](https://cfgmgmtcamp.eu) 中在根特，做了一个[演讲](https://www.youtube.com/watch?v=eBm3oyUmoAo&feature=youtu.be&t=18586)，我认为这个演讲受到了良好的反响，关于需要一些 Kubernetes 配置管理以及我们在 $work 中为此构建的解决方案，[kr8](https://kr8.rocks)。

在演讲中，我发表了一项声明，引发了一些相当激烈的讨论，无论是在线还是在会议上：

以我的话来说：

> 在某个时候，我们决定对我们进行 yaml 模板化是可以接受的。这是什么时候发生的？这是可以接受的吗？

经过一些讨论，我想最好还是以某种方式支持我的观点。本博客文章将尝试做到这一点。

## 配置问题

一旦您要管理的应用程序和基础设施增长到一定规模，您就不可避免地会陷入某种形式的配置复杂性地狱中。如果您只部署 1 或可能 2 个东西，您可以编写一个 yaml 配置文件并完成。然而，一旦您超越了这一点，您就需要弄清楚如何管理这种复杂性。非常可能的是，您拥有多个配置文件的原因是使用该配置的 $thing 与其伙伴略有不同。例如：

+   部署在不同环境（如 dev、stg 和 prod）中的应用程序

+   部署在不同地区（如欧洲或北美）的应用程序

显然，并非*所有*配置都不同，但很可能配置有足够的差异，您希望能够区分两者。

这种配置复杂性多年来一直为运营商（系统管理员、DevOps 工程师，无论您想如何称呼他们）所熟知。一个完整的学科围绕这个问题发展起来了，在配置管理中，每个工具都以自己的方式解决了这个问题，但最终，它们都使用 YAML 完成了任务。

我一直最喜欢的方法是[hiera](https://puppet.com/docs/puppet/6.2/hiera_intro.html)，它与 Puppet 捆绑在一起。具有按层级查找特定配置需求变量的能力非常强大和灵活，并且通常意味着您实际上不需要对 yaml 进行任何模板化，除非是将 Puppet 事实嵌入到 yaml 中。

## 我们退步了吗？

然后，随着我们行业的需求从操作系统上升到云计算，我们有了一个全新的数据平面需要配置。用于配置这些的工具发生了变化，出现了像[CloudFormation](https://aws.amazon.com/cloudformation/)和[Helm](https://helm.sh/)这样的工具。这些工具是出色的配置工具，但我坚信我们（作为一个行业）在设计它们时真的、真的犯了一个很大的错误。为了审视这一点，让我们来看一个helm chart接受自定义参数的示例

### Helm Charts

Helm chart可以接受由`values.yaml`文件定义的外部参数，你在渲染图表时会指定这些参数。一个简单的例子可能看起来像这样：

假设我的外部参数很简单——它是一个字符串。看起来会是这样的：

```
image: "{{ .Values.image }}" 
```

这并不那么糟糕，对吧？你只需在你的values.yaml中为`image`指定一个值，然后就可以继续进行了。

当你想要做更复杂和复杂的事情时，真正的问题开始凸显出来。在这个特定的例子中，你做得还不错，因为你*知道*你必须为Kubernetes部署指定一个镜像。然而，如果你要处理的是像是一个可选字段呢？那么，情况就会变得有些不方便了：

```
{{- with .resourceGroup  }}
    resourceGroup: {{ .  }}
{{- end }} 
```

在模板语言中，可选值只会让事情变得混乱，你不能只是将值留空，所以你不得不求助于丑陋的循环和条件语句，而这些循环和条件语句可能会在以后给你带来麻烦。

假设你需要更进一步，并且你需要将一个数组或映射推送到配置中。使用Helm，你会这样做。

```
{{- with .Values.podAnnotations  }}
      annotations:
{{ toYaml . | indent 8  }}
{{- end  }} 
```

首先，让我们忽略有一个模板函数`toYaml`将yaml转换为yaml的疯狂之处，更多地关注这里的空白问题。

YAML有严格的要求和空白实现规则。例如，以下内容既不是有效的也不是完整的yaml：

```
something: nothing
  hello: goodbye 
```

通常情况下，如果你手写一些东西，这并不一定是个问题，因为你只需按两次退格键就能解决。然而，如果你正在使用模板系统生成YAML，你就不能这样做了——如果你要处理超过5或10个配置文件，你可能想要*生成*你的配置而不是手写。

所以，在上面的例子中，你想要将`.Values.podAnnotations`的值嵌入到已缩进的annotations字段下。所以你不仅要缩进你的值，还要正确缩进它们。

使这一切*更加混乱*的是，go解析器实际上根本不知道YAML，所以如果你试图保持语法清晰并像这样缩进模板：

```
{{- with .Values.podAnnotations }}
      annotations:
      {{ toYaml . | indent 6 }}
{{- end  }} 
```

事实上，你不能这样做，因为模板系统会混淆。这是一个关于在生成YAML配置数据时所面临的复杂性和困难的特例，但当你真正开始做更复杂的工作时，你会发现这并不是正确的方法。

毋庸置疑，这不是我想要花费 *我的* 时间去做的事情。如果在一个并不真正为此设计的模板系统中摆弄空白要求是适合你的，那么我不会阻止你。我也不想花时间在没有注释的 JSON 配置中写配置，而且不小心漏掉逗号到处都是。我们（作为一个行业）很早就决定这样做是行不通的，这就是为什么有 YAML 的原因。

那么，我们应该怎么做呢？这就是 [jsonnet](https://jsonnet.org) 的用武之地。

## JSON、Jsonnet 和 YAML

在我们真正讨论 Jsonnet 之前，值得提醒大家一个非常重要（但常常被忽视的）观点。[YAML 是 JSON 的一个超集](https://yaml.org/spec/1.2/spec.html#id2759572)，在两者之间进行转换是微不足道的。许多应用程序和编程语言可以原生地解析 JSON 和 YAML，并且许多可以在两者之间进行简单的转换。例如，在 Python 中：

```
python -c 'import json, sys, yaml ; y=yaml.safe_load(sys.stdin.read()) ; print(json.dumps(y))' 
```

那么，考虑到这一点，让我们来谈谈 Jsonnet。

### 欢迎来到 Jsonnet 教堂

Jsonnet 是一种相对较新的、鲜为人知的（在 Kubernetes 社区之外？）语言，自称为*数据模板语言*。阅读并消化 Jsonnet [设计原理](https://jsonnet.org/articles/design.html) 页面绝对是一个不错的练习，以了解它存在的原因，但如果我要简要地定义它的目的 - 它是用来生成 JSON 配置的。

那么，它到底如何帮助呢？

好吧，让我们来看看之前的例子 - 我们想要生成一些 JSON 配置来指定一个参数（即，图像字符串）。我们可以使用 Jsonnet 并通过外部变量来非常非常容易地做到这一点。

首先，让我们定义一些 Jsonnet：

```
{

  image: std.extVar('image'),

} 
```

然后，我们可以使用 Jsonnet 命令行工具来生成它，根据需要传入外部变量：

```
jsonnet image.jsonnet -V image="my-image"
{
   "image": "my-image"
} 
```

简单吧！

### 可选字段

之前，我提到如果你想要定义一个可选字段，使用 YAML 模板，你必须为每个字段定义 if 语句。但是，使用 Jsonnet，你只需定义代码！

```
// define a variable - yes, jsonnet also has comments
local rg = null;
{

  image: std.extVar('image'),
  // if the variable is null, this will be blank
  [if rg != null then 'resourceGroup']: rg,

} 
```

这里的输出，因为我们的变量是 null，意味着我们实际上从未填充 resourceGroup。如果你指定一个值，它将出现：

```
jsonnet image.jsonnet -V image="my-image" 
{
   "image": "my-image"
} 
```

### 映射和参数

好了，现在让我们看看我们之前的注解示例。我们想要定义一些 pod 注解，它以 YAML 映射作为输入。你希望这个映射可以通过指定外部数据来配置，并且显然在命令行上这样做很麻烦（例如，你很少会在命令行上使用 Helm 来指定这个）所以通常你会使用 Jsonnet 导入来完成。我将把这个配置指定为一个变量，然后将该变量加载到注解中：

```
local annotations = {
  'nginx.ingress.kubernetes.io/app-root': '/',
  'nginx.ingress.kubernetes.io/enable-cors': true,
};

{
  metadata: { // annotations are nested under the metadata of a pod
    annotations: annotations,
  },

} 
```

这可能只是我对 Jsonnet 的偏见在说话，但这比纠结缩进简直容易得多，我甚至无法描述它有多简单。

### 附加好处

最后我想快速探讨的是，这是我认为不能真正用Helm和其他yaml模板工具完成的概念，那就是操作配置中现有对象的概念。

让我们以带有注释的上面的示例为例，来看看结果文件：

```
{  "metadata":  {  "annotations":  {  "nginx.ingress.kubernetes.io/app-root":  "/",  "nginx.ingress.kubernetes.io/enable-cors":  true  }  }  } 
```

现在，假设例如我想要向这个注释映射中添加一组注释。在任何模板系统中，我可能都必须重新编写整个映射。

Jsonnet使这个*变得微不足道*。我可以简单地使用`+`运算符将某些内容添加到其中。这里是一个（糟糕的）示例：

```
local annotations = {
  'nginx.ingress.kubernetes.io/app-root': '/',
  'nginx.ingress.kubernetes.io/enable-cors': true,
};

{
  metadata: {
    annotations: annotations,
  },
} + { // this adds another JSON object
  metadata+: { // I'm using the + operator, so we'll append to the existing metadata
    annotations+: { // same as above
      something: 'nothing',
    },
  },
} 
```

最终结果是这样的：

```
{  "metadata":  {  "annotations":  {  "nginx.ingress.kubernetes.io/app-root":  "/",  "nginx.ingress.kubernetes.io/enable-cors":  true,  "something":  "nothing"  }  }  } 
```

显然，在这种情况下，需要更多的代码，但是当你的示例变得更加复杂时，能够以这种方式操作对象变得非常有用。

## Kr8

我们在 [kr8](https://kr8.rocks) 中使用所有这些方法，使得为多个 Kubernetes 集群创建和操作配置变得简单易行。如果您在这里找到的任何概念使您点头，我强烈推荐您去了解一下。
