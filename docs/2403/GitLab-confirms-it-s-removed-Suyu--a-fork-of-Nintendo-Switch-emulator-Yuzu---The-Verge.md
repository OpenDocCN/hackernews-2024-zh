<!--yml

category: 未分类

date: 2024-05-29 12:35:20

-->

# GitLab 确认已移除了 Suyu，这是任天堂 Switch 模拟器 Yuzu 的一个分支 - The Verge

> 来源：[https://www.theverge.com/2024/3/21/24108191/gitlab-suyu-nintendo-switch-emulator-takedown](https://www.theverge.com/2024/3/21/24108191/gitlab-suyu-nintendo-switch-emulator-takedown)

Nintendo 可能无需[通过单独起诉模拟器以使其从地下深处彻底消失](/24098640/nintendo-emulator-yuzu-lawsuit-switch-aftermath)来将它们逼入更深的地下。今天，[GitLab 切断了对任天堂 Switch 模拟器 Suyu 的访问](https://gitlab.com/suyu-emu/suyu)，并在收到一封看似可怕的 DMCA 撤销请求的电子邮件后，禁用了其开发人员的账户。

“GitLab 收到了来自权利持有人代表的 DMCA 撤销通知，并且按照我们的[标准流程](https://handbook.gitlab.com/handbook/legal/dmca/)进行了处理，”发言人 Kristen Butler 告诉 *The Verge*。

Suyu 是 Yuzu 的一个分支，这是任天堂成功起诉的模拟器，但现在问题并不是任天堂是否拥有 Yuzu 代码的权利 — 或者甚至可能不是任天堂？任天堂在[其和解中并不一定赢得了对 Yuzu 代码的所有权](/2024/3/4/24090357/nintendo-yuzu-emulator-lawsuit-settlement)，而 GitLab 并没有告诉 *The Verge* 说是任天堂在背后发起的撤销请求。

*一位 Suyu 贡献者收到的电子邮件之一。*

相反，正如你可以在上面的电子邮件中看到的那样 — 在 Suyu 的 Discord 中共享的几个之一，以及[由 Overkill.wtf 提前发布的](https://overkill.wtf/suyu-emulator-removed-from-gitlab/) — 发出撤销请求的人似乎试图借鉴 Yuzu 涉嫌违反 DMCA 1201 规避任天堂技术保护措施的方式。哦，也许还在威胁 GitLab 在此期间进行非法交易（也是 DMCA 1201 的一部分）。

我不是律师，但两年前有几位律师告诉我，*有效*的 DMCA 撤销请求[在技术上应该包含](https://www.law.cornell.edu/uscode/text/17/512)“声称已被侵犯的受版权保护作品的标识”，而 DMCA 1201 并不等同于涵盖撤销请求的 DMCA 512。

此外，[Suyu 声称](https://arstechnica.com/gaming/2024/03/heres-how-the-makers-of-the-suyu-switch-emulator-plan-to-avoid-getting-sued/)，它不包含与 Yuzu 相同的规避措施。

但是那些律师还告诉我，无论有效与否，这并不一定很重要，因为像 GitLab 这样的平台并不必须托管任何它不想托管的东西。如果替代方案可能是任天堂向你发起实际诉讼，保护你甚至可能不关心的东西可能不值得推回无效的 DMCA 撤销请求，特别是如果这可能导致任天堂真的提起诉讼。

*现在 Suyu 的 GitLab 页面是什么样子。*

GitLab 尚未立即回答关于是否公司政策在用户有机会删除其项目或提交 DMCA 反通知之前禁用其账户的问题。该公司的在线手册并未说明 GitLab 可能决定封锁或禁止用户使用其平台的原因；只是“在适当的情况下，我们可能会禁用被举报用户的访问权限或终止其账户。”

Suyu 看来已经找到了新家。大约一个小时前，其领导人写道“我肯定会托管代码的副本。” 在那时，另一名成员已经克隆了这个仓库到 [git.suyu.dev.](https://git.suyu.dev/)
