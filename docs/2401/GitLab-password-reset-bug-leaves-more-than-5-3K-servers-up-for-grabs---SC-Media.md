<!--yml

分类：未分类

日期：2024-05-27 15:13:55

-->

# GitLab 密码重置漏洞使得超过 5.3K 台服务器易受攻击 | SC Media

> 来源：[`www.scmagazine.com/news/gitlab-password-reset-bug-leaves-more-than-5-3k-servers-up-for-grabs`](https://www.scmagazine.com/news/gitlab-password-reset-bug-leaves-more-than-5-3k-servers-up-for-grabs)

[重大的 GitLab 漏洞 CVE-2023-7028](https://www.scmagazine.com/news/gitlab-vulnerability-risks-account-takeover-via-simple-password-reset)截至周二还未在超过 5,300 台服务器上修补，可能导致软件开发者账户远程接管。

这个漏洞的 CVSS 最高评分为 10 分，[GitLab 首次披露并修复](https://about.gitlab.com/releases/2024/01/11/critical-security-release-gitlab-16-7-2-released/)是在 1 月 11 日。GitLab 登录系统的漏洞将允许攻击者向其自己的未验证电子邮件地址发送密码重置链接，而不需要受害者的任何用户交互。

“通过构造特殊格式的 HTTP 请求，可以实现接管帐户，该请求能够向未经验证的电子邮件地址发送密码重置电子邮件，从而在未修补的版本中实现账户接管，” GitLab 发言人在 1 月 12 日的电子邮件中告诉 SC Media。

GitLab 版本 16.5.6、16.6.4 和 16.7.2 已发布安全更新，并已回溯到版本 16.1.6、16.2.9、16.3.7 和 16.4.5。

一名研究人员在 GitLab 社区版 16.6.1 版本上测试了该漏洞，并在[AttackerKB 上分享了他们的结果](https://attackerkb.com/topics/VBDvNxhyjr/cve-2023-7028)，表示 CVE-2023-7028“非常有效且易于利用”。

在补丁发布近两周后，Shadowserver Foundation 发现全球有[5,379 个 GitLab 的脆弱实例](https://dashboard.shadowserver.org/statistics/combined/map/?map_type=std&day=2024-01-23&source=http_vulnerable&source=http_vulnerable6&tag=cve-2023-7028%2B&geo=all&data_set=count&scale=log)。这个监控恶意在线活动的非营利组织在 1 月 23 日发布了数据，指出美国和德国的脆弱实例最多，分别为 964 个和 730 个。

[Shadowserver 的仪表板工具](https://dashboard.shadowserver.org/statistics/combined/time-series/?date_range=7&source=http_vulnerable&source=http_vulnerable6&tag=cve-2023-7028%2B&group_by=geo&style=stacked)显示在 1 月 24 日仅有 4,652 个脆弱实例。一位 Shadowserver 发言人向 SC Media 确认，“检测到的脆弱实例数量减少，这是一个积极的发展”，但需要更多时间来确定此减少是否是趋势还是 Shadowserver 扫描中的“波动”。

## GitLab CVE-2023-7028 的威胁指标

受影响产品的自托管实例的 GitLab 客户 —— GitLab Community Edition 和 GitLab Enterprise Edition —— 应按照 GitLab 所述的两种方法审查其日志，以查找对 CVE-2023-7028 的利用。

+   检查 gitlab-rails/production_json.log 中针对 /users/password 路径的 HTTP 请求，其中 params.value.email 由包含多个电子邮件地址的 JSON 数组组成。

+   检查 gitlab-rails/audit_json.log 中具有 PasswordsController#create 的 meta.caller.id 和 target_Details 由包含多个电子邮件地址的 JSON 数组组成的条目。

公司表示，在 GitLab.com 或 GitLab Dedicated 实例上没有检测到对该漏洞的滥用。

GitLab 建议客户还启用两因素认证 (2FA)，以防止通过 CVE-2023-7028 进行帐户接管，尽管未打补丁的实例的用户仍然容易因攻击者利用漏洞重置其密码而被锁定在其帐户之外。
