<!--yml

类别：未分类

日期：2024-05-27 13:05:14

-->

# 它干了吗？ – 坏格糖果坏格糖果坏格糖果坏格糖果

> 来源：[https://badgerbadgerbadgerbadger.dev/posts/misc/2024-04-08-is-it-dry-yet/](https://badgerbadgerbadgerbadger.dev/posts/misc/2024-04-08-is-it-dry-yet/)
> 
> **澄清**：这确实是关于我的洗衣。我没有使用比喻。我不是在谈论我的感受或秘密。我在谈论我的衣服。

## 介绍

我们的洗衣机是三星的某种型号，它在完成洗涤周期后会通过[SmartThings应用](https://www.samsung.com/be/smartthings/app/)向我发送通知。这很有用，因为我们几乎总是需要在洗涤周期后进行烘干，甚至可能需要再次洗另一批衣物。这台机器不是组合洗衣机烘干机，所以必须有人把湿衣服从我们的智能洗衣机移到我们的笨烘干机。

我们的烘干机没有“智能”功能，这正是我开始自动化之旅的原因。如果它能够连接到Wifi并在完成烘干周期时发送通知，那么我现在就不会写这篇文章了。

由于我已经习惯了知道我的洗涤何时完成（所以我知道何时去把东西移到烘干机），我决定为烘干机做类似的事情。例如，当衣物需要额外的干燥周期或者有更多的湿衣服等待洗衣机时，这尤其有用。在这些时候，知道烘干机是否完成一个周期非常有用。

由于它不是智能设备，所以我必须*某种方式*加入智能。

## 攻击计划

这就是我决定解决问题的方式：

+   我有一个[Shelly智能插座](https://www.shelly.com/en-be/products/shop/shelly-plus-plug-s)，我会把它放在Drew（烘干机名为Drew）和墙壁之间。

+   插头将会将电力读数传输到我的[Home Assistant](https://www.home-assistant.io/)设置中。

+   当电力读数从一个高数值下降到非常低的数值时，我会*某种方式*来检测（我假设它永远不会降到0，因为小LED面板需要电源）。

+   我会*某种方式*通知自己。

这是*某种方式*和*某种方式*的部分我不确定。我的Home Assistant设置尽可能基础。我只是一个多月或两个月前开始自己的家庭自动化之旅，并不熟悉这些可能性。然而，这感觉像是HASS能力范围内的事情。

## 发现自动化

当我进一步了解Home Assistant时，我发现的第一件事是[自动化](https://www.home-assistant.io/docs/automation/)。在Home Assistant中，您通过自动化系统创建自动化。您可以在有人进入房间时通过运动传感器点亮灯光；当湿度达到一定水平时打开通风口；您可以在2月14日当您的爱人下班回家时，让所有灯变成粉红色并在扬声器上播放浪漫音乐。可能性*无限*。

## 自动化设置

弄清楚我想要的自动化设置很简单。家庭助手自动化系统有3个关键组件。

1.  [触发器](https://www.home-assistant.io/docs/automation/trigger/)：引发自动化的事物。这些可以是许多类型，从传感器上的数字变化到传入的MQTT负载。

1.  [条件](https://www.home-assistant.io/docs/automation/condition/)：一个可选的额外检查，以确保当自动化触发时，一切都处于您期望的状态。您可以检查传感器的数值，检查设备的开关状态，甚至可以[检查太阳是否升起](https://www.home-assistant.io/docs/scripts/conditions/#sun-condition)。

1.  [动作](https://www.home-assistant.io/docs/automation/action/)：这是在自动化中*执行*某些操作的地方。这是您想要设置的地方，例如当太阳升起时设置电动公鸡报时（如果您出于某种原因想要制作自己的奇怪闹钟），或者当孩子们使用电力过多时在Webhook上发送消息。

经过我的思考，我决定为我的触发器/条件/动作设置如下：

1.  **触发器**：我希望当德鲁的功耗降至10瓦以下（由智能插座希拉报告）并且保持在10瓦以下超过5分钟时启动自动化。

1.  **条件**：我希望确保此自动化仅在过去5分钟内消耗了超过100瓦的功率时触发。

1.  **动作**：以某种方式通知我发生了这件事。

虽然触发器和条件很简单，但动作则有点模糊。起初，我不太确定如何通知自己。我查阅了[Home Assistant的Restful通知](https://www.home-assistant.io/integrations/notify.rest/)，但是怎么也无法使其工作。然后我决定查看是否可以发送电子邮件。我找到了[Home Assistant的SMTP通知](https://www.home-assistant.io/integrations/smtp/)，决定试试看。同样，我也无法使其工作。它要求我在Home Assistant配置文件中输入我的Gmail密码，但我不敢这样做。我决定寻找另一种方法。

最后，我偶然发现了[Home Assistant iOS应用](https://www.home-assistant.io/integrations/ios/)，我将其安装在我的手机上，并指向了我的Home Assistant服务器。这使我能够向我的手机发送通知（我的手机成为了通知的目标）。

完整的自动化设置如下：

```
alias: Is it dry yet? description: Notifies when Drew (the dryer) has finished a drying cycle. trigger:  - id: Sheila Power Below 10 W platform: numeric_state entity_id: - sensor.sheila_power below: 10 for: minutes: 5 condition:  - condition: numeric_state entity_id: sensor.sheila_power_5_minutes_ago above: 100 action:  - service: notify.mobile_app_sharpie data: message: Drew has finished a cycle. You should unload him. title: Dryer Cycle Finished - service: notify.mobile_app_buttercups_android data: message: Drew has finished a cycle. You should unload him. title: Dryer cycle finished. - service: media_player.play_media target: entity_id: media_player.thuis data: announce: true media_content_type: mp3 media_content_id: https://<static-host>/drew-finished-drying.mp3 mode: single 
```

`notify.mobile_app_sharpie`和`notify.mobile_app_buttercups_android`是由我的手机和女友的手机上的Home Assistant iOS和Android应用公开的实体。`media_player.thuis`是我Google Nest扬声器组上公开的实体。不仅仅满足于向我的手机发送通知，我还想在Google Nest扬声器组上播放声音。这个声音是我托管在静态主机上的mp3文件。

`sensor.sheila_power`是我智能插头暴露的传感器实体，它让我监控烘干机的功率消耗。`sensor.sheila_power_5_minutes_ago`是我使用[SQL集成](https://www.home-assistant.io/integrations/sql)创建的传感器，用于获取5分钟前的功率消耗。

```
select * from states where metadata_id = (select states_meta.metadata_id  from states_meta where states_meta.entity_id = 'sensor.sheila_power') and datetime(last_updated_ts, 'unixepoch') < datetime('now', '-5 minutes') order by last_updated_ts desc limit 1 
```

结合起来，这确保了当我们的功率消耗在至少5分钟内低于10瓦特，并且在此之前我们的功耗超过100瓦特时，此自动化才会触发。这给我一个很好的指示，告诉我烘干机已经完成了一个循环，我应该去卸载它。

## 暴露给世界

我应该提到，此时我的Home Assistant服务器并不对外网开放。所有的集成都是通过本地Wi-Fi进行的，有防火墙规则阻止任何外部访问。我并不需要将我的Home Assistant服务器暴露到互联网，就可以在我的手机上接收通知，因为它们都是[Tailnet](https://tailscale.com/kb/1136/tailnet)的一部分。这意味着只要我在手机上安装了[Tailscale应用](https://tailscale.com/download/ios)，并启用了Tailscale VPN，我就可以在世界任何地方访问我的Home Assistant服务器。

这对我来说已经足够好了，但是我的女朋友也需要在*她的*手机上接收通知，让她安装Tailscale并在需要时打开/关闭，对她来说有点麻烦。

所以，我决定将我的Home Assistant服务器暴露到互联网上。

我的第一次尝试是使用Let's Encrypt和DuckDNS，按照类似于[这篇](https://www.makeuseof.com/access-home-assistant-server-remotely-duckdns-letsencrypt/)的指南（或者说三篇指南）。我不会详细展开，但我可以说那真是一场*痛苦*。最终，在苦苦挣扎了几天后，我放弃了。我放弃了，决定回到我的女朋友身边，低头告诉她这个坏消息。

后来我偶然发现了[Tailscale Funnels](https://tailscale.com/kb/1223/funnel)。因为我已经在使用Tailscale，只需额外一次调用，我就可以与我的Home Assistant服务器建立一个安全的、加密的隧道。

```
➜  ~ tailscale funnel --https=8443 --bg 8123 
```

由于我可以使用我的Tailnet名称，而不需要额外的DNS，并且由于Tailscale为我处理了所有的证书，因此也不需要额外的证书。

这种方法的唯一注意事项是Tailscale目前不支持域名处理。因此，我无法为我的Home Assistant服务器指定一个好看的域名。此外，我只有[3个可用的端口](https://tailscale.com/kb/1223/funnel#limitations)。这意味着如果我希望在该机器上公开其他服务，我最多只能再使用2个端口，并且它们必须通过它们的端口号进行引用。

这是我以后需要回头处理的事情，但目前我和我的女朋友都可以在世界任何地方访问我们的Home Assistant服务器，并且很开心。

当然，我将她的账户权限限制在只能访问通知和部分视图上。我不希望她搞乱我的自动化设置，也不希望第三方乱翻我的智能家居设置。
