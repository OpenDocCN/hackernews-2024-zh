<!--yml

category: 未分类

date: 2024-05-29 12:43:52

-->

# Chris's Wiki :: blog/sysadmin/PrometheusAutomatingDNSChecks

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/sysadmin/PrometheusAutomatingDNSChecks](https://utcc.utoronto.ca/~cks/space/blog/sysadmin/PrometheusAutomatingDNSChecks)

最近我写了一篇关于[使用基本的Prometheus监控DNS查询结果的问题](/~cks/space/blog/sysadmin/PrometheusDNSMonitoringProblem)，这主要是因为[Blackbox导出器](https://github.com/prometheus/blackbox_exporter)需要为每个您要进行的DNS查询生成一个配置段落（一个[*模块*](/~cks/space/blog/sysadmin/PrometheusBlackboxNotes)），并且不会暴露任何关于查询类型和名称的标签。在一条评论中，Mike Kohne问我是否考虑过使用脚本来生成所需的各种配置，以便您可以检查N个DNS查询在M个不同的DNS服务器上的情况。我实际上没有考虑过这个问题，而且我们不太可能这样做，但是如果我们这样做了，这就是我会怎么做。

生成系统的输入是我们想要确认工作的DNS查询列表，这至少包括一个名称和一个DNS查询类型（A、MX、SOA等），可能还包括预期结果以及我们想要执行这些查询的DNS服务器列表。一个完整的系统将允许多组查询和DNS服务器，这样您就可以查询内部DNS服务器以及您希望始终可解析的外部名称。

首先，我会为此目的运行一个完全独立的Blackbox实例，以便其配置完全可以由脚本生成。对于要进行的每个DNS查询，脚本将计算出黑匣子模块的名称，然后组合成公式化的段落，例如：

> ```
> autodns_a_utoronto_something:
>   prober: dns
>   dns:
>     query_name: "utoronto.example.com"
>     query_type: "A"
>     validate_answer_rrs:
>       fail_if_none_matches_regexp:
>         - ".*\t[0-9]*\tIN\tA\t.*"
> 
> ```

然后，您的生成程序将所有这些段落与一些固定的开头内容结合在一起，就得到了这个Blackbox实例的配置文件。只有在添加新的DNS名称要查询时，它才需要更改。

脚本生成的另一项是一个格式化为Prometheus [文件发现](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#file_sd_config)所需格式的抓取目标和其标签的列表。由于我们自动生成这个文件，我们可以把所有的智能内容都放入标签中，包括指定黑匣子模块。这将为每个模块提供一个块，列出将查询的所有DNS服务器及其必需的标签。这可以是JSON或YAML格式，在YAML格式下，它看起来像（对于一个模块）：

> ```
> - labels:
>     # Hopefully you can directly set __param_module in
>     # a label like this.
>     __param_module: autodns_a_utoronto_something
>     query_name: utoronto.example.com
>     query_type: A
>     [... additional labels based on local needs ...]
>   targets:
>   - dns1.example.org:53
>   - dns2.example.org:53
>   - 8.8.8.8:53
>   - 1.1.1.1:53
>   [...]
> 
> ```

(如果我们从程序中的数据开始，最好生成JSON。现在几乎所有的编程语言都可以创建JSON，而且它比尝试自动生成YAML更宽容。但如果我要把结果放入版本控制存储库，我会生成YAML。)

更详细的细分是可能的，例如将外部 DNS 服务器与内部 DNS 服务器分开，并将他人的 DNS 名称与您的 DNS 名称分开。你会得到很多带有各种标签混合的段落，但整个过程都是自动生成的，你不必看它。在我们的本地配置中，我们最终会得到至少几个额外的标签和更复杂的组合。

我们需要查询名称和查询类型作为标签可用，因为我们将为所有这些 Blackbox 模块编写一个通用的警报规则，类似于：

> ```
> - alert: DNSGeneric
>   expr: probe_success{probe=~"autodns_.*"} == 0
>   for: 5m
>   annotations:
>     summary: "We cannot get the {{$labels.query_type}} record for {{$labels.query_name}} from the DNS server ..."
> 
> ```

（如果 Blackbox 把这些标签放在 DNS 探针指标中，我们可以跳过在抓取目标配置中添加它们的步骤。我们还能将许多现有的 DNS 警报合并为更通用的警报。）

如果你愿意付出额外的努力，使一些 DNS 查询需要特定的结果（而不仅仅是'某个 A 记录'或'某个 MX 记录'），那么你可能需要额外的标签来编写更具体的警报记录。

对于我们来说，这两个生成的文件都是相对静态的。实际上，我们很少添加额外的名称进行检查或者经常测试的 DNS 服务器。

我们当然可以编写这样的配置文件生成系统，并且比我们当前拥有的更全面地覆盖我们的 DNS 区域和各种名称服务器。然而，我目前的看法是，额外的复杂性几乎肯定不值得用于问题检测和系统维护。如果更容易，如果使用这样的生成系统，我们会对更多的 DNS 服务器进行更多的查询，但几乎永远不会检测到我们不知道的任何问题。
