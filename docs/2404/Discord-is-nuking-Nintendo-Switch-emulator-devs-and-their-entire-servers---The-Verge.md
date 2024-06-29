<!--yml

category: 未分类

date: 2024-05-27 13:09:19

-->

# Discord正在铲除任天堂Switch模拟器开发人员及其整个服务器 - The Verge

> 来源：[https://www.theverge.com/2024/4/11/24127545/discord-suyu-sudachi-server-shutdown-account-ban](https://www.theverge.com/2024/4/11/24127545/discord-suyu-sudachi-server-shutdown-account-ban)

Discord已关闭了任天堂Switch模拟器[Suyu](https://suyu.dev/)和[Sudachi](https://github.com/sudachi-emu/sudachi)的Discord服务器，并完全禁用了其主要开发者的账户 —— 公司对为何采取如此举措未回答我们的问题。Suyu和Sudachi最初是在3月4日开始作为[Nintendo起诉消失的模拟器Yuzu的分支](/24098640/nintendo-emulator-yuzu-lawsuit-switch-aftermath)。

"Discord回应并遵守所有合法有效的数字千禧年版权法（DMCA）请求。在这个实例中，还有法院发布的禁令要求撤销这些材料，我们的行动与法院命令一致，" Discord产品传播主任Kellyn Slone在接受*The Verge*采访时的声明部分表示。

根据与*The Verge*分享的图片显示，Suyu和Sudachi的开发者只收到了关于他们共享涉嫌侵犯知识产权的内容的模糊信息。与此同时，Discord告诉我们，它正在按照其正常的DMCA撤销请求流程进行 —— 但目前尚不清楚是否存在有效的DMCA撤销请求，或者这些社区是否实际上违反了知识产权，而且很可能Discord并未遵循其自己的政策而将它们踢出去。

请记住，任天堂让Yuzu达成和解，而不是在法庭上证明其案件，而且和解并未授予任天堂Yuzu自由可复制的[GPL v3](https://www.gnu.org/licenses/gpl-3.0.en.html)代码的权利。Yuzu分支的开发者还声称他们[在改变代码的同时进行其他做法](/24098640/nintendo-emulator-yuzu-lawsuit-switch-aftermath#:~:text=they%E2%80%99re%20treating%20Nintendo%E2%80%99s%20lawsuit%20like%20a%20guidebook%20about%20how%20not%20to%20piss%20off%20the%20company.)，以避免激怒任天堂。而且那些代码在任何情况下都不是托管在Discord上的。

但尽管承诺，仍有可能在这些服务器上分享任天堂的加密密钥、固件，甚至整个盗版游戏。归根结底，大多数寻找任天堂Switch模拟器的人是想在上面玩任天堂游戏。但是随着服务器的消失，很难证明任何一种情况。

即使 Suyu 和 Sudachi 确实存在侵权行为，[Discord 的政策](https://support.discord.com/hc/en-us/articles/4410339349655-Discord-s-Copyright-IP-Policy)也不建议在首次违规行为时永久禁止，更不用说清除整个服务器了。 Discord 没有回答关于这些用户是否是重复侵权者、是否收到过任何先前警告或是否转发过任何下架请求的问题。

Sudachi 的开发者 Jarrod Norwell 告诉我，这似乎是突如其来的：“他们的第一封电子邮件说我的帐户违反了服务条款，没有额外信息。” 他声称 Sudachi 并没有进行任何侵权活动。稍后，他被告知这可能与知识产权有关，但称 Discord 仍未给予他任何详细信息。

DMCA 下架请求传统上是关于*内容*，而不是关于人或人群，而[Discord 的政策](https://support.discord.com/hc/en-us/articles/4410339349655-Discord-s-Copyright-IP-Policy)则是根据这一点编写的。有效的下架请求必须包括侵权内容的描述及其位置；平台然后会撤下内容，如果用户提交了声称内容实际上没有侵权的“反通知”，则可以恢复内容。在这一点上，Discord 已经完成了其工作，如果任天堂想起诉开发者，可以使用反通知追踪他们。

但这似乎并不是这里发生的事情。看起来 Discord 只是通过清除其通讯渠道来取消这些模拟器的平台支持。

而[法庭命令](https://storage.courtlistener.com/recap/gov.uscourts.rid.56980/gov.uscourts.rid.56980.11.0.pdf)中提到的禁止某些第三方“提供、营销、广告、推广...或以其他方式交易 Yuzu 或 Yuzu 的任何源代码或功能”，特别指的是与被告“积极共同行动和参与”的第三方。 Discord 不会告诉我 Yuzu 的任何开发者是否与 Suyu 或 Sudachi 项目有关联。

归根结底，像 Discord 这样的平台没有义务托管任何它们不想托管的内容，正如我们在 [GitLab 通过取消 Suyu 代码的平台支持时讨论过的](/2024/3/21/24108191/gitlab-suyu-nintendo-switch-emulator-takedown)。也许 Discord 确实看到了这些 Discord 中的软件盗版证据。但目前它并没有以此来为这些频道清除行为辩护。

对于一些Suyu的开发者来说，这可能是压垮骆驼的最后一根稻草：内部人士告诉我，经过内斗之后，一组人分裂出去做自己的项目，可能与仿真相关或者不相关；[这里有一个Pastebin](https://pastebin.com/6FYdz9Sr)，一个“真正的Suyu开发者”声称核心开发团队因为Suyu的“放射性”及其据称自大的领导人而离开了项目。（根据内部人士向我展示的内容，该领导人往往会[发号施令](https://www.reddit.com/r/suyu/comments/1c10p18/founder_speaking_current_situation_and_future/)。）

Sudachi的开发者告诉我，他仍在继续他的所有项目。

任天堂此次的最新下架行动不仅瞄准了Switch模拟器，还包括一些协助它们的工具：它向GitHub发送了DMCA下架请求，以移除[27个Sigpatch Updater的分支](https://github.com/github/dmca/commit/f4c2c915c058e01d70d7671668b4c12a4b0e45b5)，以及[Lockpick_RCM, kezplez-nx和Incognito_RCM](https://github.com/github/dmca/commit/ccb374868b46ad19371d9f96cccdd6c8fc689cba)，这些工具帮助Switch的所有者和开发者获取加密密钥。
