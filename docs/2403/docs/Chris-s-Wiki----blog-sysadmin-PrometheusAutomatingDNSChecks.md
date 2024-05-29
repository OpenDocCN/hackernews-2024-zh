<!--yml
category: 未分类
date: 2024-05-29 12:43:52
-->

# Chris's Wiki :: blog/sysadmin/PrometheusAutomatingDNSChecks

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/sysadmin/PrometheusAutomatingDNSChecks](https://utcc.utoronto.ca/~cks/space/blog/sysadmin/PrometheusAutomatingDNSChecks)

Recently I wrote about [the problem of using basic Prometheus to monitor DNS query results](/~cks/space/blog/sysadmin/PrometheusDNSMonitoringProblem), which comes about primarily because the [Blackbox exporter](https://github.com/prometheus/blackbox_exporter) requires a configuration stanza (a [*module*](/~cks/space/blog/sysadmin/PrometheusBlackboxNotes)) for every DNS query you want to make and doesn't expose any labels for what the query type and name are. In a comment, Mike Kohne asked if I'd considered using a script to generate the various configurations needed for this, where you want to check N DNS queries across M different DNS servers. I hadn't really thought about it and we're unlikely to do it, but here is how I would if we did.

The input for the generation system is a list of DNS queries we want to confirm work, which is at least a name and a DNS query type (A, MX, SOA, etc), possibly along with an expected result, and a list of the DNS servers that we want to make these queries against. A full blown system would allow multiple groups of queries and DNS servers, so that you can query your internal DNS servers for internal names as well as external names you want to always be resolvable.

First, I'd run a completely separate Blackbox instance for this purpose, so that its configuration can be entirely script-generated. For each DNS query to be made, the script will work out the Blackbox module's name and then put together the formulaic stanza, for example:

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

Then your generation program combines all of these stanzas together with some stock front matter and you have this Blackbox instance's configuration file. It only needs to change if you add a new DNS name to query.

The other thing the script generates is a list of scrape targets and labels for them in the format that Prometheus [file discovery](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#file_sd_config) expects. Since we're automatically generating this file we might as well put all of the smart stuff into labels, including specifying the Blackbox module. This would give us one block for each module that lists all of the DNS servers that will be queried for that module, and the labels necessary. This could be JSON or YAML, and in YAML form it would look like (for one module):

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

(If we're starting with data in a program it's probably better to generate JSON. Pretty much every language can create JSON by now, and it's a more forgiving format than trying to auto-generate YAML even if the result is less readable. But if I was going to put the result in a version control repository, I'd generate YAML.)

More elaborate breakdowns are possible, for example to separate external DNS servers from internal ones, and other people's DNS names from your DNS names. You'll get an awful lot of stanzas with various mixes of labels, but the whole thing is being generated automatically and you don't have to look at it. In our local configuration we'd wind up with at least a few extra labels and a more complicated set of combinations.

We need the query name and query type available as labels because we're going to write one generic alert rule for all of these Blackbox modules, something like:

> ```
> - alert: DNSGeneric
>   expr: probe_success{probe=~"autodns_.*"} == 0
>   for: 5m
>   annotations:
>     summary: "We cannot get the {{$labels.query_type}} record for {{$labels.query_name}} from the DNS server ..."
> 
> ```

(If Blackbox put these labels in DNS probe metrics we could skip adding them in the scrape target configuration. We'd also be able to fold a number of our existing DNS alerts into more generic ones.)

If you go the extra distance to have some DNS lookups require specific results (instead of just 'some A record' or 'some MX record'), then you might need additional labels to let you write a more specific alert record.

For us, both generated files would be relatively static. As a practical matter we don't add extra names to check or DNS servers to test against very often.

We could certainly write such a configuration file generation system and get more comprehensive coverage of our DNS zones and various nameservers than we currently have. However, my current view is that the extra complexity almost certainly wouldn't be worth it in terms of detecting problems and maintaining the system. We'd make more queries against more DNS servers if it was easier, if it would be with such a generation system, but those queries would almost never detect anything we didn't already know.