<!--yml

类别：未分类

日期：2024-05-27 15:05:47

-->

# Ansible的三元过滤器的有趣用法

> 来源：[https://www.zufallsheld.de/2024/02/21/interesting-use-of-ansible-ternary-filter/](https://www.zufallsheld.de/2024/02/21/interesting-use-of-ansible-ternary-filter/)

发布于2024年2月21日星期三

一段时间前，我发现了Ansible中[三元过滤器](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/ternary_filter.html)的一个有趣用法。在Ansible中，三元过滤器是一个接受三个参数的过滤器：条件、条件为真时的值以及条件为假时的替代值。

这里有一个简单的例子直接来自Ansible的文档：

```
- name: service-foo, use systemd module unless upstart is present, then use old service module
  service:
    state: restarted
    enabled: yes
    use: "{{ (ansible_service_mgr == 'upstart') | ternary('service', 'systemd') }}" 
```

但是对于这个过滤器还有更多有趣的用例，我决定看看Ansible的集合作者是如何使用它的。

## 只有在详细模式大于0时显示命令输出。

这是最初引起我对过滤器不同用例感兴趣的用法。

根据你在运行playbook时使用的详细程度（例如添加多少个`-v`到命令中），该命令将以安静模式运行（带有`-quiet`标志）或者不运行。

```
- name: Validate configuration
  become: true
  become_user: "{{ consul_user }}"
  ansible.builtin.command: >
    {{ consul_binary }} validate {{ (ansible_verbosity == 0) | ternary("-quiet", "") }}
    {{ consul_config_path }}/config.json {{ consul_configd_path }}
  changed_when: false 
```

（[来源](https://github.com/ansible-collections/ansible-consul/blob/3ee2b43972e0da2f378422d16c672a7c719c4998/tasks/config.yml#L49)）

HN-用户 [raziel2p](https://news.ycombinator.com/user?id=raziel2p) 给出了一个更好的 [示例](https://news.ycombinator.com/item?id=39466080) 来实现这一点：

```
- name: Validate configuration
  become: true
  become_user: "{{ consul_user }}"
  ansible.builtin.command: >
    {{ consul_binary }} validate {{ "-quiet" if ansible_verbosity == 0 }}
    {{ consul_config_path }}/config.json {{ consul_configd_path }}
  changed_when: false 
```

## 在没有分子的情况下在一次运行中测试任务幂等性

这种使用三元过滤器来测试任务幂等性在一次运行中非常有用。

首先，任务文件`test_create_scheduler.yml`被导入时没有设置变量，所以任务将改变某些内容。然后，再次导入任务文件，但这次设置了变量`test_proxysql_scheduler_check_idempotence`为`true`。

```
- name: "{{ role_name }} | test_create_scheduler | test create scheduler"
  import_tasks: test_create_scheduler.yml

- name: "{{ role_name }} | test_create_scheduler | test idempotence of create scheduler"
  import_tasks: test_create_scheduler.yml
  vars:
    test_proxysql_scheduler_check_idempotence: true 
```

当导入测试文件`test_create_scheduler.yml`时没有设置变量`test_proxysql_scheduler_check_idempotence`，断言将检查`status is changed`，因为三元过滤器将变量`test_proxysql_scheduler_check_idempotence`评估为`false`。

```
- name: "{{ role_name }} | {{ current_test }} | check if create scheduler reported a change"
  assert:
    that:
      - "status is {{ test_proxysql_scheduler_check_idempotence|ternary('not changed', 'changed') }}" 
```

当导入测试文件`test_create_scheduler.yml`并设置变量`test_proxysql_scheduler_check_idempotence`时，断言将检查`status is not changed`，因为三元过滤器将变量`test_proxysql_scheduler_check_idempotence`评估为`true`。

（[来源](https://github.com/ansible-collections/community.proxysql/blob/d4ef72ae73dfad8d46ff639dd2ac76e204635d5b/tests/integration/targets/test_proxysql_scheduler/tasks/test_create_scheduler.yml#L13)）

## 基于正则表达式搜索进行操作

在Ansible中，您可以使用管道符号（`|`）链接过滤器。这允许您基于正则表达式搜索进行过滤（事后看来显而易见它起作用了，但我从未想过那样做）。

```
- name: "{{ role_name }} | {{ current_test }} | are we performing a delete"
  set_fact:
    test_delete: "{{ current_test | regex_search('^test_delete') | ternary(true, false) }}" 
```

([来源](https://github.com/ansible-collections/community.proxysql/blob/d4ef72ae73dfad8d46ff639dd2ac76e204635d5b/tests/integration/targets/test_proxysql_mysql_users/tasks/base_test.yml#L5))

## 轻松处理较老的 Python 版本

在下一个任务中，将安装 cassandra-driver。如果使用的（已过时！）Python 版本以 2.7 开头，pip 应该安装 cassandra-driver 的 3.26.* 版本。如果使用的是较新的 Python 版本，pip 将安装 cassandra-driver 的最新版本。

```
- name: Install cassandra-driver
  pip:
    name: "cassandra-driver{{ ansible_python_version.startswith('2.7') | ternary('==3.26.*', '') }}" 
```

这绝对不是最优雅的解决方案，但它有效。（我可能会尝试根据操作系统及其 Python 版本安装正确的 cassandra-driver 版本，它应该被安装在那里）

([来源](https://github.com/ansible-collections/community.cassandra/blob/a35580565c949d7d13bbfd5dca307746e42d3725/tests/integration/targets/setup_cassandra/tasks/cassandra_auth.yml#L130))

如果变量 `zabbix_web_ssl_cipher_suite` 被定义且不为 `none`，此任务将添加以 `ssl_ciphers` 开头的行。否则，它将添加相同的行但是被注释掉。

```
{{ (zabbix_web_ssl_cipher_suite is defined and zabbix_web_ssl_cipher_suite is not none) | ternary('', '# ') }}ssl_ciphers {{ zabbix_web_ssl_cipher_suite | default('') }} 
```

([来源](https://github.com/ansible-collections/community.zabbix/blob/facde86d8e388673d503ebc3b19fd0f9f6037798/roles/zabbix_web/templates/nginx_vhost.conf.j2#L73))

我用另一种方式在模板中添加了被注释掉的行（[查看](https://github.com/dev-sec/ansible-collection-hardening/blob/bdf6d65cfd9d63b7ffe00f67e280f652299283bc/roles/ssh_hardening/templates/opensshd.conf.j2#L48)）。我使用了 [comment-filter](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/comment_filter.html)：

```
{{ "HostKeyAlgorithms " ~ ssh_host_key_algorithms|join(',') if ssh_host_key_algorithms else "HostKeyAlgorithms" | comment }} 
```

现在我看到自己的代码，我可能也可以在这里使用三元过滤器！

```
{{ ssh_host_key_algorithms | ternary("HostKeyAlgorithms " ~ ssh_host_key_algorithms|join(','), "HostKeyAlgorithms" | comment) }} 
```

但我觉得我实际上更喜欢 if-else 语法。

你有其他有趣的使用三元过滤器的例子吗？
