<!--yml

category: 未分类

date: 2024-05-27 14:53:34

-->

# systemd：启用无限服务重启 - Michael Stapelberg

> 来源：[`michael.stapelberg.ch/posts/2024-01-17-systemd-indefinite-service-restarts/`](https://michael.stapelberg.ch/posts/2024-01-17-systemd-indefinite-service-restarts/)

当一个服务连续启动失败了足够多次时，systemd 就放弃了它。

在服务器上，这不是我想要的 —— 一般来说，如果守护进程无限地重新启动，对于自动恢复是有帮助的。只要你的服务之间没有循环依赖，所有的服务在瞬时故障后最终都会启动起来，而不需要指定依赖关系。

这特别有用，因为在 systemd 级别上指定依赖关系会引入一些问题：当交互式停止单个服务时，systemd 也会停止依赖的服务。然后你需要记得稍后重新启动依赖的服务，这很容易被忘记。

## 启用服务的无限重新启动

要使 systemd 无限重新启动一个服务，我首先喜欢创建一个如下的 drop-in 配置文件：

```
cat > /etc/systemd/system/restart-drop-in.conf <<'EOT'
[Unit]
StartLimitIntervalSec=0

[Service]
Restart=always
RestartSec=1s
EOT 
```

然后，我可以为单个服务（如 `prometheus-node-exporter`）启用重新启动行为，而无需修改它们的 `.service` 文件（在更新时需要手动操作）：

```
cd /etc/systemd/system
mkdir prometheus-node-exporter.service.d
cd prometheus-node-exporter.service.d
ln -s ../restart-drop-in.conf
systemctl daemon-reload 
```

## 改变所有服务的默认设置

如果你的大部分服务设置了 `Restart=always` 或 `Restart=on-failure`，你可以像这样更改系统范围内的默认值：

```
mkdir /etc/systemd/system.conf.d
cat > /etc/systemd/system.conf.d/restartdefaults.conf <<'EOT'
[Manager]
DefaultRestartSec=1s
DefaultStartLimitIntervalSec=0
EOT
systemctl daemon-reload 
```

## 默认设置是做什么的？

那么我们为什么需要改变这些设置呢？

默认的 systemd 设置（截至 systemd 255）是：

```
DefaultRestartSec=100ms
DefaultStartLimitIntervalSec=10s
DefaultStartLimitBurst=5 
```

这意味着，指定了 `Restart=always` 的服务会在崩溃后的 100ms 后重新启动，如果服务在 10 秒内崩溃超过 5 次，systemd 将不再尝试重新启动该服务。

很容易看出，对于一个服务，比如说，需要花费 100ms 来崩溃，例如，因为它无法绑定到它的监听 IP 地址，这意味着：

| 时间 | 事件 |
| --- | --- |
| T+0 | 第一次启动 |
| T+100ms | 第一次崩溃 |
| T+200ms | 第二次启动 |
| T+300ms | 第二次崩溃 |
| T+400ms | 第三次启动 |
| T+500ms | 第三次崩溃 |
| T+600ms | 第四次启动 |
| T+700ms | 第四次崩溃 |
| T+800ms | 第五次启动 |
| T+900ms | 10 秒内的第五次崩溃 |
| T+1s | systemd 放弃 |

## systemd 为什么默认放弃？

我不确定。如果我必须猜测，我会猜测开发人员想要防止笔记本电脑因为一个 CPU 核心永久忙于重新启动一些在紧密循环中崩溃的服务而过快地耗尽电池。

使用更宽松的 `DefaultRestartSec=` 值也可以达到同样的目的：例如，使用 `DefaultRestartSec=5s`，我们可以足够地将这些崩溃在时间上分开。

有一些[最近的讨论](https://github.com/systemd/systemd/issues/30804)关于改变默认值。让我们看看讨论的结果。

我从 2005 年开始经营博客，传播知识和经验已经将近 20 年啦！ :)

如果您想支持我的工作，您可以[给我买杯咖啡](https://www.buymeacoffee.com/stapelberg)。

感谢您的支持！ ❤️
