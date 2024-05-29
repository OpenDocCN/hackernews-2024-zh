<!--yml
category: 未分类
date: 2024-05-27 15:05:47
-->

# Interesting Uses of Ansible’s ternary filter

> 来源：[https://www.zufallsheld.de/2024/02/21/interesting-use-of-ansible-ternary-filter/](https://www.zufallsheld.de/2024/02/21/interesting-use-of-ansible-ternary-filter/)

Posted on Wed 21 February 2024 

Some time ago I discovered an interesting use of the [ternary-filter](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/ternary_filter.html) in Ansible. A ternary-filter in Ansible is a filter that takes three arguments: a condition, a value if the condition is true and an alternative value if the condition is false.

Here’s a simple example straight from Ansible’s documentation:

```
- name: service-foo, use systemd module unless upstart is present, then use old service module
  service:
    state: restarted
    enabled: yes
    use: "{{ (ansible_service_mgr == 'upstart') | ternary('service', 'systemd') }}" 
```

But there are many more interesting use cases for this filter and I decided to take a look what Ansible’s collection authors used it for.

## Display command output only if verbosity is greater than 0.

This was the usage that initially got me interested in different use-cases for the filter.

Depending on what verbosity level you use when running a playbook (e.g. how many `-v` you add to the command), the command will run in quiet-mode (with the `-quiet` flag) or not.

```
- name: Validate configuration
  become: true
  become_user: "{{ consul_user }}"
  ansible.builtin.command: >
    {{ consul_binary }} validate {{ (ansible_verbosity == 0) | ternary("-quiet", "") }}
    {{ consul_config_path }}/config.json {{ consul_configd_path }}
  changed_when: false 
```

([Source](https://github.com/ansible-collections/ansible-consul/blob/3ee2b43972e0da2f378422d16c672a7c719c4998/tasks/config.yml#L49))

HN-User [raziel2p](https://news.ycombinator.com/user?id=raziel2p) gave an even better [example](https://news.ycombinator.com/item?id=39466080) to achieve this:

```
- name: Validate configuration
  become: true
  become_user: "{{ consul_user }}"
  ansible.builtin.command: >
    {{ consul_binary }} validate {{ "-quiet" if ansible_verbosity == 0 }}
    {{ consul_config_path }}/config.json {{ consul_configd_path }}
  changed_when: false 
```

## Testing task idempotency in one run without molecule

This use of the ternary-filter is useful for testing task idempotency in one run.

First, the task-file `test_create_scheduler.yml` is imported without a variable set, so the task will change something. Then, the task-file is imported again, however this time with the variable `test_proxysql_scheduler_check_idempotence` set to `true`.

```
- name: "{{ role_name }} | test_create_scheduler | test create scheduler"
  import_tasks: test_create_scheduler.yml

- name: "{{ role_name }} | test_create_scheduler | test idempotence of create scheduler"
  import_tasks: test_create_scheduler.yml
  vars:
    test_proxysql_scheduler_check_idempotence: true 
```

When importing the test file `test_create_scheduler.yml` without the variable `test_proxysql_scheduler_check_idempotence`, the assert will check for `status is changed`, because the ternary-filter evaluated the variable `test_proxysql_scheduler_check_idempotence` as `false`.

```
- name: "{{ role_name }} | {{ current_test }} | check if create scheduler reported a change"
  assert:
    that:
      - "status is {{ test_proxysql_scheduler_check_idempotence|ternary('not changed', 'changed') }}" 
```

When importing the test file `test_create_scheduler.yml` with the variable `test_proxysql_scheduler_check_idempotence`, the assert will check for `status is not changed`, because the ternary-filter evaluated the variable `test_proxysql_scheduler_check_idempotence` as `true`.

([Source](https://github.com/ansible-collections/community.proxysql/blob/d4ef72ae73dfad8d46ff639dd2ac76e204635d5b/tests/integration/targets/test_proxysql_scheduler/tasks/test_create_scheduler.yml#L13))

## Do things based on regex searches

In Ansible you can chain filters using a pipe (`|`). This allows you to filter based on regex searches (which in hindsight is obvious that it works, but I never thought about that).

```
- name: "{{ role_name }} | {{ current_test }} | are we performing a delete"
  set_fact:
    test_delete: "{{ current_test | regex_search('^test_delete') | ternary(true, false) }}" 
```

([Source](https://github.com/ansible-collections/community.proxysql/blob/d4ef72ae73dfad8d46ff639dd2ac76e204635d5b/tests/integration/targets/test_proxysql_mysql_users/tasks/base_test.yml#L5))

## Handle older Python versions easily

In the following task, the cassandra-driver is installed. If the used (obsolete!) Python version starts with 2.7, pip should install the cassandra-driver in version 3.26.*. If a recent Python version is used, pip will install the latest version of the cassandra-driver.

```
- name: Install cassandra-driver
  pip:
    name: "cassandra-driver{{ ansible_python_version.startswith('2.7') | ternary('==3.26.*', '') }}" 
```

That’s definitely not the most elegant solution, but it works. (I’d probably have tried to install the correct cassandra-driver version according to the operating system and its Python version, where it should be installed)

([Source](https://github.com/ansible-collections/community.cassandra/blob/a35580565c949d7d13bbfd5dca307746e42d3725/tests/integration/targets/setup_cassandra/tasks/cassandra_auth.yml#L130))

This task will add a line starting with `ssl_ciphers`, if the variable `zabbix_web_ssl_cipher_suite` is defined and not `none`. Otherwise it will add the same line but commented out.

```
{{ (zabbix_web_ssl_cipher_suite is defined and zabbix_web_ssl_cipher_suite is not none) | ternary('', '# ') }}ssl_ciphers {{ zabbix_web_ssl_cipher_suite | default('') }} 
```

([Source](https://github.com/ansible-collections/community.zabbix/blob/facde86d8e388673d503ebc3b19fd0f9f6037798/roles/zabbix_web/templates/nginx_vhost.conf.j2#L73))

I used another way to add commented out lines in a template ([see](https://github.com/dev-sec/ansible-collection-hardening/blob/bdf6d65cfd9d63b7ffe00f67e280f652299283bc/roles/ssh_hardening/templates/opensshd.conf.j2#L48)). I used the [comment-filter](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/comment_filter.html):

```
{{ "HostKeyAlgorithms " ~ ssh_host_key_algorithms|join(',') if ssh_host_key_algorithms else "HostKeyAlgorithms" | comment }} 
```

Now that I see my own code, I could probably use the ternary-filter here, too!

```
{{ ssh_host_key_algorithms | ternary("HostKeyAlgorithms " ~ ssh_host_key_algorithms|join(','), "HostKeyAlgorithms" | comment) }} 
```

But I think I actually like the if-else syntax more.

Do you have any other interesting uses of the ternary-filter?