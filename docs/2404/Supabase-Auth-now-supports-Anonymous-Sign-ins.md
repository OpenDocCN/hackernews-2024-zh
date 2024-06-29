<!--yml

category: 未分类

date: 2024-05-27 13:16:32

-->

# Supabase Auth 现在支持匿名登录

> 来源：[https://supabase.com/blog/anonymous-sign-ins](https://supabase.com/blog/anonymous-sign-ins)

Supabase Auth 现在支持[匿名登录](/docs/guides/auth/auth-anonymous)，这是我们社区[最多请求的功能之一](https://github.com/supabase/auth/issues/68)。

匿名登录可用于创建尚未注册应用程序的**临时用户**。这降低了新用户尝试您产品的障碍，因为他们无需提供任何注册凭据。

您可以今天从仪表板为您的项目[启用匿名登录](/dashboard/project/_/settings/auth)：

对于本地开发，请升级您的 Supabase CLI 并将配置添加到`config.toml`文件中：

`_10

enable_anonymous_sign_ins = true`

您可以通过[Javascript](/docs/reference/javascript/auth-signinanonymously)，[Flutter](/docs/reference/dart/auth-signinanonymously)或[Swift](/docs/reference/swift/auth-signinanonymously) SDK今天创建匿名用户。以下是使用`supabase-js`创建匿名用户的方法。

`_10

const { data, error } = await supabase`

创建使用匿名登录的配置文件也是`已认证`！

一旦调用`.signInAnonymously()`，您就已将用户移入认证流程，并且我们会像已登录用户一样对待他们：

像永久用户一样，匿名用户在`auth.users`表中持久化：

| id | role | email | is_anonymous |
| --- | --- | --- | --- |
| e053e470-afa1-4625-8963-37adb862fd11 | 已认证 | NULL | true |
| 5563108e-ac81-4063-9288-4f3db068efa1 | 已认证 | [[email protected]](/cdn-cgi/l/email-protection#0e627b656b4e7d7a6f7c796f7c7d206d6163) | false |

可以通过用户的JWT中返回的`is_anonymous`声明来识别匿名用户，这在您的行级安全性策略（RLS）中是可访问的。如果您想要限制应用程序中某些功能的访问权限，这非常有帮助。

例如，假设我们有一个在线论坛，用户可以创建和阅读帖子。

鉴于这张表来存储帖子：

`_10

创建表 public.posts (

_10

id serial 主键,`

如果我们只想允许永久用户创建帖子，我们可以通过检查JWT来检查用户是否匿名`select auth.jwt() ->> 'is_anonymous'`。

使用此函数在RLS策略中：

`_10

创建策略“只允许永久用户创建帖子”

_10

到已认证 -- 注意：用户仍然已认证

_10

(select auth.jwt() ->> 'is_anonymous')::boolean 为 false`

RLS 为我们提供了创建各种规则的完全灵活性。

例如，为了允许永久用户阅读所有帖子并将匿名用户限制为今天创建的帖子：

`_12

创建策略“限制匿名用户访问”

_12

到已认证 -- 注意：用户仍然已认证

_12

当(select (auth.jwt() ->> 'is_anonymous'))::boolean 为 true

_12

then (created_at >= current_date)`

在某些时候，匿名用户可能会决定要创建一个帖子。这时，我们会提示他们注册一个帐户，从而将其转换为永久用户。

##### 当匿名用户有关联身份时，匿名用户被视为永久用户。

他们转换后，用户 ID 保持不变，这意味着与用户 ID 关联的任何数据都将被继承。

Supabase Auth 提供了2种实现方式：

1.  链接电子邮件或电话身份

1.  链接 OAuth 身份

要链接电子邮件或电话身份：

`_10

const { data, error } = await supabase

_10

.updateUser({ email })`

要将 OAuth 身份链接到匿名用户，您需要为您的项目 [启用手动链接](/dashboard/project/_/settings/auth)。了解 [身份链接](/docs/guides/auth/auth-identity-linking) 如何在 Supabase Auth 中工作。

一旦启用，您可以调用 `linkIdentity()` 方法：

`_10

const { data, error } = await supabase

_10

.linkIdentity({ provider: 'google' })`

当创建 RLS 策略以区分匿名用户的访问权限时，您可以在 SQL 编辑器中利用 [用户模拟功能](/blog/studio-introducing-assistant) 来测试您的策略：

在 SQL 编辑器中的数据库角色设置。您可以通过从下拉菜单中选择用户来模拟匿名用户。

[用户管理屏幕](/dashboard/project/_/auth/users) 提供了按匿名用户筛选的选项，这有助于了解已创建多少匿名用户。

在“用户”页面上按匿名用户进行筛选。

管理匿名用户可能会很棘手，特别是当您的网站有大量访问者时。我们正在开发一个“自动清理”选项，用于删除超过30天未活跃的匿名用户。与此同时，由于匿名用户存储在数据库的认证模式中，您可以通过运行以下查询清理孤立的匿名用户：

`_10

-- 删除30天前创建的匿名用户

_10

delete from auth.users

_10

其中 is_anonymous 为 true 并且 created_at < now() - interval '30 days';`

我们还在开发一个 [linter](https://github.com/supabase/splinter/pull/28) 来检查您的 RLS 策略，并突出显示允许匿名用户访问的策略 - 敬请关注本月稍后的更新！
