<!--yml

category: 未分类

date: 2024-05-27 15:05:49

-->

# Release v1.1.0 · ory/kratos · GitHub

> 来源：[https://github.com/ory/kratos/releases/tag/v1.1.0](https://github.com/ory/kratos/releases/tag/v1.1.0)

以下功能专门针对此版本的 Ory Network 发布：

Ory Kratos 1.1 是我们旅程中的重要里程碑。

我们衷心希望您能发现 Ory Kratos 1.1 中的新功能和改进对您的项目非常有价值。为了体验最新版本的强大功能，我们鼓励您获取 Ory Kratos 的最新版本 [here](https://github.com/ory/kratos)，或在 [Ory Network](https://www.ory.sh/network/) 利用 Ory Kratos —— 运行 Ory 的最简单、最经济高效的方式。

你是否热衷于安全，并希望在最大的开源社区之一发挥有意义的影响？加入 [Ory 社区](https://slack.ory.sh/)，成为新一代 ID 解决方案的一部分。我们正在共同构建下一代能有效保护组织和个人身份的 IAM 解决方案。

想亲自查看 Ory Kratos 吗？使用以下命令在 Ory Network 上运行您的 Ory Kratos 项目：

```
brew install ory/tap/cli

scoop bucket add ory <https://github.com/ory/scoop.git>
scoop install ory

bash <(curl <https://raw.githubusercontent.com/ory/meta/master/install.sh>) -b . ory
sudo mv ./ory /usr/local/bin/

ory auth login

ory create project --name "My first Kratos project"

ory open account-experience registration

ory patch identity-config \
  --replace '/identity/default_schema_id="preset://username"' \
  --replace '/identity/schemas=[{"id":"preset://username","url":"preset://username"}]' \
  --format yaml

ory open account-experience registration 
```

```
- kratos list identities 1 100
+ kratos list identities --page-size 100 --page-token ... 
```

+   `oidc` 在负载中不需要方法 ([#3564](https://github.com/ory/kratos/issues/3564)) ([b299abc](https://github.com/ory/kratos/commit/b299abcfa1ebdb8bbb6bb9339f61873d5c77c44f)):

    +   fix: `oidc` 在负载中不需要方法

    +   refactor: 仅在测试中更新策略顺序

    +   chore: 更新审计消息和注释

+   在 courier 中接受所有 200 响应作为 OK ([#3401](https://github.com/ory/kratos/issues/3401)) ([88237e2](https://github.com/ory/kratos/commit/88237e25b080a9643f6cbf7eedbf23988ba9ba7c)), closes [#3399](https://github.com/ory/kratos/issues/3399):

    +   fix: 在 courier 中接受所有 200 响应作为 OK

+   在验证后接受 login_challenge ([#3427](https://github.com/ory/kratos/issues/3427)) ([6b02350](https://github.com/ory/kratos/commit/6b02350c21aa65decd1bb16e559e1cc7dae42d55))：

    部分来自 [ory/network#320](https://github.com/ory/network/issues/320)

+   在会话 JWT 令牌化期间对 Jsonnet 片段添加缓存 ([#3699](https://github.com/ory/kratos/issues/3699)) ([1da8180](https://github.com/ory/kratos/commit/1da818072154baa5c0921134919afde595031e94))

+   添加一致性标志 ([#3733](https://github.com/ory/kratos/issues/3733)) ([fd79950](https://github.com/ory/kratos/commit/fd7995077307cc101550eda5d7724ea1f68fa98a))

+   在默认的 cors 头部添加 max-age ([#3584](https://github.com/ory/kratos/issues/3584)) ([c5b4aaa](https://github.com/ory/kratos/commit/c5b4aaa2df5d010b62a99ccf45850583daad3a66))

+   在 oidc 策略中添加缺失的追踪和属性 ([#3429](https://github.com/ory/kratos/issues/3429)) ([09bcb71](https://github.com/ory/kratos/commit/09bcb71f1f0b3238e2d0f4376a1a2290d062c6c1))

+   在 createRecoveryLinkForIdentity 的 API 规范中添加 return_to 参数 ([#3711](https://github.com/ory/kratos/issues/3711)) ([757a5e4](https://github.com/ory/kratos/commit/757a5e43257e9ff28a16bfe76f8e737b656d3696))

+   将值代码添加到认证方法枚举中 ([#3546](https://github.com/ory/kratos/issues/3546)) ([95dc7a2](https://github.com/ory/kratos/commit/95dc7a20f49aa682f324b70e507ec56c20159ebb)):

+   配置模式中的 additional_id_token_audiences 键 ([#3622](https://github.com/ory/kratos/issues/3622)) ([9396bb0](https://github.com/ory/kratos/commit/9396bb0b586d1d1e74a85c0ae3bcf9de81214f1b))

+   调整跟踪详细程度 ([976cd0d](https://github.com/ory/kratos/commit/976cd0dc3dd95c2c1992bfa82394e9fad39f34f2))

+   允许后期恢复钩子中断流程 ([#3393](https://github.com/ory/kratos/issues/3393)) ([6c1d2f1](https://github.com/ory/kratos/commit/6c1d2f1e4173cfb9a7abe2bfe4f20e47b7568d3b))

+   允许从 Webhook 响应中更新管理员元数据 ([#3569](https://github.com/ory/kratos/issues/3569)) ([22f61f0](https://github.com/ory/kratos/commit/22f61f015495c55e58db4f31ee6882444b9a3caf))

+   分页链接头部始终返回相对 URL ([fb229c9](https://github.com/ory/kratos/commit/fb229c982c6f7d7a4f5f0f84ffc971a576906160))

+   自动迁移旧账户以使用代码凭证 ([#3581](https://github.com/ory/kratos/issues/3581)) ([569b14a](https://github.com/ory/kratos/commit/569b14aba864761236bd3d5a48e4e69f10ea6c86))

+   将 `oauth2_login_challenge` 传递到注册流程 ([#3419](https://github.com/ory/kratos/issues/3419)) ([76241be](https://github.com/ory/kratos/commit/76241bee3dc7fec4690346ee85bc4b9f897fdd34)):

    修复 [#3321](https://github.com/ory/kratos/issues/3321)

+   将 ListIdentities 更改为键集分页 ([e16fed1](https://github.com/ory/kratos/commit/e16fed1f8563509aac30886386668bb85e6dc797))

+   将 shebang 和 makefile 从 /bin/bash 更改为 /usr/bin/env bash ([#3597](https://github.com/ory/kratos/issues/3597)) ([1343bbb](https://github.com/ory/kratos/commit/1343bbbfa11ff3e7fcbc0f233b858d13fd40c66d)):

    签名：nxy7 [lolnoxy@gmail.com](mailto:lolnoxy@gmail.com)

+   在接受 hydra 登录请求之前检查 whoami aal ([#3669](https://github.com/ory/kratos/issues/3669)) ([a2f79c3](https://github.com/ory/kratos/commit/a2f79c31f3208b88024897fc8bf1307ccac6f895))

+   注册和两步验证的代码方法（[#3481](https://github.com/ory/kratos/issues/3481)) ([7aa2e29](https://github.com/ory/kratos/commit/7aa2e293175d0f4b6c13552cc3781f54f8caf3a0))

+   考虑 OIDC 注册流程错误地使用重复凭证，由策略完成 ([#3525](https://github.com/ory/kratos/issues/3525)) ([3e3c789](https://github.com/ory/kratos/commit/3e3c78967523676cbce9a227d574c2f7f4ea314d)):

    在这里返回其他任何内容可能会导致 Kratos 响应两个连接的 JSON 对象：新的登录流程，实际错误消息作为第一个对象，以及一个非常令人困惑的 '500, 中止注册钩子执行' 作为第二个对象。

+   浏览器流程中的 CSRF 令牌重新生成 ([#3706](https://github.com/ory/kratos/issues/3706)) ([e4908db](https://github.com/ory/kratos/commit/e4908dbe4a42fad5a80c4d46004e1e3710cabeb7)), 关闭了 [#3705](https://github.com/ory/kratos/issues/3705)

+   测试中的数据竞争 ([ab6dc31](https://github.com/ory/kratos/commit/ab6dc3121535d27668fed58804a218b17b17ae43))

+   不要在多个地方编码完整配置 ([#3500](https://github.com/ory/kratos/issues/3500)) ([57a3273](https://github.com/ory/kratos/commit/57a3273055c6e8627dd0b736e881dba3fb0fe75d))

+   不要为 API 流程生成 CSRF 令牌 ([#3704](https://github.com/ory/kratos/issues/3704)) ([d93570d](https://github.com/ory/kratos/commit/d93570d330155c27a9315d1f530a0002a459910a))

+   不要并行初始化注册表的部分 ([#3534](https://github.com/ory/kratos/issues/3534)) ([ff177db](https://github.com/ory/kratos/commit/ff177db8a97f27abc3e883e79832685348602334))

+   不要在设置中列出 org SSOs ([#3637](https://github.com/ory/kratos/issues/3637)) ([6c7068c](https://github.com/ory/kratos/commit/6c7068cf41df51cde5fe9fc79cca84ec6124d38a))

+   MFA 流程不需要代码凭据 ([#3753](https://github.com/ory/kratos/issues/3753)) ([40ed809](https://github.com/ory/kratos/commit/40ed809db631149874864f216a106c43ea5df670))

+   OIDC 验证不需要会话 ([#3443](https://github.com/ory/kratos/issues/3443)) ([e08f831](https://github.com/ory/kratos/commit/e08f831c2715e515bf58dc2dbb47fc3576421a5c))

+   对于 POST /admin/identities 不要为冲突返回 500 ([#3437](https://github.com/ory/kratos/issues/3437)) ([1429949](https://github.com/ory/kratos/commit/142994932e449d9948148804502c98ef73daafff))

+   如果代码无效，不要返回 nil ([#3662](https://github.com/ory/kratos/issues/3662)) ([df8ec2b](https://github.com/ory/kratos/commit/df8ec2b9b77a53beb32e3f94a8fccb711896d8e7)):

+   身份导入时的错误处理 ([#3520](https://github.com/ory/kratos/issues/3520)) ([83bfb2d](https://github.com/ory/kratos/commit/83bfb2d2a9c69bf3a3442500b9484c1a69f8c794)):

    在导入身份时，如果没有特征或特征格式错误，将返回 500。这提高了错误处理和消息传递。

+   在更新时不需要重新验证 ([#3421](https://github.com/ory/kratos/issues/3421)) ([ce8139f](https://github.com/ory/kratos/commit/ce8139f2325a8317388cbcaaa98f3f83d626657b))

+   使用小写 JSON 的 Http 传递应该使用小写 ([#3740](https://github.com/ory/kratos/issues/3740)) ([84149c4](https://github.com/ory/kratos/commit/84149c4b420ea89f0a16a579c017a8e7e1670204))

+   CLI 命令和 SDK 中的身份列表分页 ([#3482](https://github.com/ory/kratos/issues/3482)) ([1e8b1ae](https://github.com/ory/kratos/commit/1e8b1aeb4bf866892788986f62a31255372de999)):

    向 SDK 方法添加正确的分页参数，用于列出身份和会话。

+   在 Apple OIDC 回调中忽略 CSRF 中间件 ([309c506](https://github.com/ory/kratos/commit/309c50694c11162cad070337f9b1d4e0fcdf444b))

+   忽略更多的 Cloudflare cookie ([#3499](https://github.com/ory/kratos/issues/3499)) ([f124ab5](https://github.com/ory/kratos/commit/f124ab5586781cdbfc0a0cfd11b4355bfc8a115c))

+   改进的 SSRF 保护 ([#3629](https://github.com/ory/kratos/issues/3629)) ([6d08576](https://github.com/ory/kratos/commit/6d08576bbc2c06014192f05e0129b95eb6c9fd80)):

    这也改进了 OIDC 策略中的跟踪。

+   登录接受挑战错误 ([#3658](https://github.com/ory/kratos/issues/3658)) ([b5dede3](https://github.com/ory/kratos/commit/b5dede329247d0962688b15872a6caf027cf910f))

+   SDK 生成器路径不正确 ([#3488](https://github.com/ory/kratos/issues/3488)) ([ed996c0](https://github.com/ory/kratos/commit/ed996c0d25e68e8a2c7de861c546f0b0e42e9e6e))

+   SMTP 错误处理不正确 ([#3636](https://github.com/ory/kratos/issues/3636)) ([ee138ec](https://github.com/ory/kratos/commit/ee138ec4e1ba55ef077858653220db9e6b0c7254))

+   过滤参数的 Swagger 规范不正确 ([#3684](https://github.com/ory/kratos/issues/3684)) ([2c1470a](https://github.com/ory/kratos/commit/2c1470ab3556e639f06a01ac1646a6b90c7ecac7)), 关闭 [#3676](https://github.com/ory/kratos/issues/3676) [#3675](https://github.com/ory/kratos/issues/3675)

+   增加连接级超时和关闭超时 ([#3570](https://github.com/ory/kratos/issues/3570)) ([200b413](https://github.com/ory/kratos/commit/200b4138a429d113ee045d16031bb0a6312c1c01)):

    管理 API 通常预期需要更长的超时，例如在批量标识导入期间。

+   在使用 OIDC SSO 注册后验证后，会话问题 ([#3467](https://github.com/ory/kratos/issues/3467)) ([a28b523](https://github.com/ory/kratos/commit/a28b523238743f3873b51479eea3b86d684092f9))

+   Lint ([e8740c3](https://github.com/ory/kratos/commit/e8740c3498446dcaeab2990604a317e61dc170df))

+   导入时将恢复和验证电子邮件转换为小写 ([#3571](https://github.com/ory/kratos/issues/3571)) ([e2ac9ff](https://github.com/ory/kratos/commit/e2ac9ff4e2101788f1fca1b8c83f8791cce446e2)):

    因为在那里所有的电子邮件都被转换为小写，所以包含大写字符的电子邮件会被身份模式扩展运行程序覆盖。

+   将会话结构中的标识标记为可选项 ([#3463](https://github.com/ory/kratos/issues/3463)) ([7ae02ba](https://github.com/ory/kratos/commit/7ae02ba697f68c9cfae5fe8f696b2c55a3ba9ddc)), 关闭 [#3461](https://github.com/ory/kratos/issues/3461):

    在会话结构中，标识有时不可用，例如在需要 AAL2 时。

+   在强制刷新登录流中省略不相关的 OIDC 提供程序 ([#3608](https://github.com/ory/kratos/issues/3608)) ([912dccd](https://github.com/ory/kratos/commit/912dccdf04a550604c5bfeb53ccf79c5f1133ef2)):

    每当用户被要求重新验证身份（例如因为他们希望执行涉及其凭据的设置流程，而其会话不再特权时），他们被要求再次提供他们的凭据。为此类情况生成的强制刷新登录流程已排除了一些在 Kratos 中启用但无法用于验证当前身份的策略，例如，如果身份没有密码凭据，则向用户呈现的表单将不包含密码字段。

    然而，目前不适用于 OIDC 提供程序；用户始终会看到完整集合，即使其中一些不能用于作为当前身份登录。此更改导致生成的强制刷新登录流程也会在生成的表单中省略不相关的 OIDC 提供程序，以避免混淆用户哪些策略/提供程序是有效的，实际上可以用于重新验证。

+   注册后需要验证时，保留 return_to（[#3589](https://github.com/ory/kratos/issues/3589)）（[6a0a914](https://github.com/ory/kratos/commit/6a0a9149b9828ba994bec9b48a43f9d70245f43f)）：

    +   修复：注册后需要验证时，保留 return_to

    +   测试：verification 流程中的 return_to

    +   任务：重构

+   恢复中的 panic（[#3639](https://github.com/ory/kratos/issues/3639)）（[c25ddff](https://github.com/ory/kratos/commit/c25ddffd2270a8d0861e2fc78cd0ba26e63af4eb)）

+   传递上下文（[#3452](https://github.com/ory/kratos/issues/3452)）（[c492bdc](https://github.com/ory/kratos/commit/c492bdcd0c5dbdf527ae523d879a6c1eeb9c4cdf)）

+   正确规范化 OIDC 已验证的电子邮件（[#3450](https://github.com/ory/kratos/issues/3450)）（[703b910](https://github.com/ory/kratos/commit/703b910927d879558bfeb0fd2c3339b1d301fac8)）

+   即使设置了 login_challenge，也重定向到验证 URL（[#3412](https://github.com/ory/kratos/issues/3412)）（[cd9e6a0](https://github.com/ory/kratos/commit/cd9e6a0e1e4cb4957d2a50ae3d288ebb0591e42d)）：

    修复 [ory/network#320](https://github.com/ory/network/issues/320)

+   减少 whoami 中用于 aal 检查的数据库查询（[#3372](https://github.com/ory/kratos/issues/3372)）（[d814a48](https://github.com/ory/kratos/commit/d814a4864d5c25c4f320daca733873577d517331)）：

    通过减少检查不同 AAL 等级时需要执行的查询数量，显著提高性能。

+   注册代码 UI 节点组（[#3505](https://github.com/ory/kratos/issues/3505)）（[6220184](https://github.com/ory/kratos/commit/622018459ddb16c182da49dfd91fd1c6ef8c6b73)）：

+   注册应接受 hydra 登录（[#3592](https://github.com/ory/kratos/issues/3592)）（[7a47827](https://github.com/ory/kratos/commit/7a47827cfd58ef68ebfbbeaf5ed86c394ba2bd5e)）：

    +   修复：注册应接受 hydra 登录

    +   修复：oauth2 注册流程与会话

    +   wip：注册 oauth 流程测试

    +   wip: 重构 oauth 流程测试

    +   wip: 重构 op_registration_test

    +   wip: oauth 提供者注册测试

    +   wip: 重构 oauth 流程测试

    +   修复（测试）：oauth 提供者登录

    +   样式：格式化

+   带有验证的注册（[#3451](https://github.com/ory/kratos/issues/3451)）（[77c3196](https://github.com/ory/kratos/commit/77c3196fd60c5927b84e9a7f6546f80ac2d78ee5)）

+   从courier中拒绝明显无效的电子邮件地址（[#3572](https://github.com/ory/kratos/issues/3572)）（[df18c09](https://github.com/ory/kratos/commit/df18c09e0089743e8aee17540d277b9572252e06)）：

+   从模式中删除`earliest_possible_extend`默认值（[#3464](https://github.com/ory/kratos/issues/3464)）（[7e05b7d](https://github.com/ory/kratos/commit/7e05b7db3c01efc96185ac18042e971e33da37c8)）

+   删除重复的消息ID使用（[#3468](https://github.com/ory/kratos/issues/3468)）（[dfcbe22](https://github.com/ory/kratos/commit/dfcbe226bc53b91f3a6c9837496a159b85c2e68a)）

+   删除对smtp部分的要求（[#3405](https://github.com/ory/kratos/issues/3405)）（[59a3f14](https://github.com/ory/kratos/commit/59a3f1469b8412e49846a500493cb02fc6eb34b1)）

+   从更新身份中删除慢查询（[#3553](https://github.com/ory/kratos/issues/3553)）（[d138abb](https://github.com/ory/kratos/commit/d138abb6278ebb232e120bee0fb956a0f2816b8d)）

+   将“phone”快递通道重命名为“sms”（[#3680](https://github.com/ory/kratos/issues/3680)）（[eb8d1b9](https://github.com/ory/kratos/commit/eb8d1b9abd6d2b3eb86ab11d48d9ebd059586b67)）

+   在邮件队列中尊重gomail.SendError（[#3600](https://github.com/ory/kratos/issues/3600)）（[9c608b9](https://github.com/ory/kratos/commit/9c608b991874d839782d9219f2fc27d0d4a398af)）

+   当SPA身份需要AAL2时返回422（[#3572](https://github.com/ory/kratos/issues/3572)）（[df18c09](https://github.com/ory/kratos/commit/df18c09e0089743e8aee17540d277b9572252e06)）：

    如果您提交带有`Accept`标头为`application/json`的浏览器登录流，但登录流需要AAL2，则代码无法知道需要重定向用户到2FA页面。在这种情况下，而不是响应`Session`，此PR更改行为以响应`browser_location_change_required`错误（状态`422`），以指示浏览器需要打开特定的URL，/self-service/login/browser?aal=aal2。

+   对于无效的登录挑战返回400错误请求（[#3404](https://github.com/ory/kratos/issues/3404)）（[ca34e9b](https://github.com/ory/kratos/commit/ca34e9b744482b41d65082f3bed52e9c4ebd7ba4)）

+   如果密钥解组失败则返回HTTP 400（[#3594](https://github.com/ory/kratos/issues/3594)）（[fdf4956](https://github.com/ory/kratos/commit/fdf4956d9218cfa1d2227c4880e48f9bbdaeb95d)）

    +   修复：如果密钥解组失败，则返回HTTP 400

    +   修复：应用审阅者的建议，准备升级

    +   修复：按照ory/x的审阅建议进行跟进。

    +   务必升级ory/x

+   模式测试错误（[#3528](https://github.com/ory/kratos/issues/3528)）（[bee0341](https://github.com/ory/kratos/commit/bee0341c5bf5708a2210146fc59f050a1b9df663)）

+   如果缺少iss的话从用户信息声明设置iss（[#3744](https://github.com/ory/kratos/issues/3744)）（[241a911](https://github.com/ory/kratos/commit/241a911af74e8ad7353d6e3cab86db20758b86fc)）

+   在migratest中指定正确的最小版本（[18b89ea](https://github.com/ory/kratos/commit/18b89ea588d129fa88379f7b0d7f4fd00ec6023d)）

+   在/sessions/whoami中传递追踪上下文（[1254bf5](https://github.com/ory/kratos/commit/1254bf5a38dbe2c0e2798e07dd0ee5e4b2f63d6e)）

+   追踪改进（[c804cb2](https://github.com/ory/kratos/commit/c804cb2bebbefc97073cf3b8fa250c3eefc58894)）

+   类型断言所有WebHook实现的接口（[ffda1a0](https://github.com/ory/kratos/commit/ffda1a0dab661c5f11ad849b9287094313561b79)）

+   Ui节点输入属性键已添加（[#3561](https://github.com/ory/kratos/issues/3561)）（[9eff0f3](https://github.com/ory/kratos/commit/9eff0f3a611f32af7aa7f27587b3d3f4448ce915)）：

    +   修复：ui节点InputAttributes.Key已添加

    +   修复：selfservice恢复流程添加React唯一键和数字模式

    +   修复：删除与React相关的键添加

    +   测试：更新快照

+   在登录时使用多个标识符添加ID标签（[#3657](https://github.com/ory/kratos/issues/3657)）（[be907db](https://github.com/ory/kratos/commit/be907dbbd841025fd854344b77d3368b2ff8089f)）

+   如果在登录流程中可用，请使用会话中的org ID（[#3545](https://github.com/ory/kratos/issues/3545)）（[1b3647c](https://github.com/ory/kratos/commit/1b3647c2acdad966f920c2b9e6e657c52aa50c6e)）

+   使用提供者标签在链接消息中（[#3661](https://github.com/ory/kratos/issues/3661)）（[fa5ec93](https://github.com/ory/kratos/commit/fa5ec93e8ae7d971d07f0e9b3acaa0840b9ac7de)）

+   使用注册表客户端加载模式（[#3471](https://github.com/ory/kratos/issues/3471)）（[3a57726](https://github.com/ory/kratos/commit/3a577269980213e4415fd5fa713882990e2e7640)）

+   使用名字作为姓氏（[#3556](https://github.com/ory/kratos/issues/3556)）（[df80377](https://github.com/ory/kratos/commit/df80377f5fe6180fba5904baa5be1ba1d68eb2aa)）

+   错误的continue_with枚举声明（[#3522](https://github.com/ory/kratos/issues/3522)）（[4c34c24](https://github.com/ory/kratos/commit/4c34c2417db0cb1f79b42db5f33544c90b38ad87)）

+   在调用whoami时添加将会话转换为JWT的能力（[#3472](https://github.com/ory/kratos/issues/3472)）（[57b7bb8](https://github.com/ory/kratos/commit/57b7bb846c8072f786ea6b80cd688fdee75805da)），关闭[#2487](https://github.com/ory/kratos/issues/2487)：

    此补丁在`/session/whoami`中添加了查询参数`tokenize_as`，它将会话编码为JWT。可以通过使用JsonNet模板定制JWT声明，并进一步更改令牌的过期时间。

    tokenize功能支持多个模板，使得在各种用例中使用生成的JWT变得简单。

+   添加事件（[#3524](https://github.com/ory/kratos/issues/3524)）（[75031e6](https://github.com/ory/kratos/commit/75031e67bc82a820a6aba134115e8d5f93303638)）

+   向RecoveryAddress和Credentials添加GetID成员函数（[#3474](https://github.com/ory/kratos/issues/3474)）（[085d500](https://github.com/ory/kratos/commit/085d5002df27d455057d33bd2d93dfbca0de4872)）

+   添加使用 Google Android/iOS SDK 登录的 ID Token （[#3515](https://github.com/ory/kratos/issues/3515)）（[055ed92](https://github.com/ory/kratos/commit/055ed9226d9d12f5142542be2e18438ff708c2e2)）

+   为密码哈希比较添加 OpenTelemetry 跨度（[#3383](https://github.com/ory/kratos/issues/3383)）（[e3fcf0c](https://github.com/ory/kratos/commit/e3fcf0c31db9742ed61bcf783e37ee119ed19d42)）

+   将请求 URL 添加到电子邮件和短信模板中（[#3526](https://github.com/ory/kratos/issues/3526)）（[bf5f8c3](https://github.com/ory/kratos/commit/bf5f8c3cfb2eb523a77239addb8249adf9f8b31d)）

+   为电话号码添加短信验证（[#3649](https://github.com/ory/kratos/issues/3649)）（[e3a3c4f](https://github.com/ory/kratos/commit/e3a3c4fe0d6697f6864283daf4be8a8f8971c7b4)）

+   添加原生流程的恢复支持（[#3273](https://github.com/ory/kratos/issues/3273)）（[e363889](https://github.com/ory/kratos/commit/e363889732c0a1cb801fd12b2e0e8546006e9714))

+   添加 WebhookSucceeded 事件（[aa8c936](https://github.com/ory/kratos/commit/aa8c93677a8f682f7693afe69f1baf1887355e0a)）

+   添加了各种新的文本消息（[ea91483](https://github.com/ory/kratos/commit/ea914834e6bb626de2977e228af2b40935ccc980)）：

    为了改进国际化和消息定制化，我们添加了一堆新消息。定制消息的集成可能需要处理这些新的消息代码：

    +   1010014

    +   1010015

    +   1040005

    +   1040006

    +   1070012

    +   1070013

    +   4000028

    +   4000029

    +   4000030

    +   4000031

    +   4000032

    +   4000033

    +   4000034

    +   4000035

    +   4000036

    +   4010007

    +   4010008

    +   4040002

    +   4040003

    另外，这些消息增加了更多上下文：

    +   1050014

    +   [`1050018`](https://github.com/ory/kratos/commit/1050018653251a7509c46e47e2478e96de510665)

    +   1070002

    +   4000001

    +   4000003

    +   4000004

    +   4000017

    +   4000018

    +   4000019

    +   4000020

    +   4000021

    +   4000022

    +   4000023

    +   4000024

    +   4000025

    +   4000026

    +   4010001

    +   4040001

    +   4050001

    +   4060005

    +   4070005

    +   5000001

+   允许额外的 ID Token 受众（[#3616](https://github.com/ory/kratos/issues/3616)）（[0fa648d](https://github.com/ory/kratos/commit/0fa648d9f7b837a35de9b230a05b5951e95d5874)）

+   在 NewPersister 中允许额外的迁移（[96c1ff7](https://github.com/ory/kratos/commit/96c1ff7747ea38e23a3892f74b75ee555ed49c88)）

+   允许在凭证标识符上进行模糊搜索（[#3526](https://github.com/ory/kratos/issues/3526)）（[2cb3ea2](https://github.com/ory/kratos/commit/2cb3ea2eaff909ac936611d5653f69e713f41b64)）：

    该 PR 添加了在凭证标识符中搜索子字符串和类似字符串的能力。

    请注意，**postgres** 和 **CRDB** 迁移创建了用于此功能的特殊索引。要使用 [在线模式更改](https://www.cockroachlabs.com/docs/v23.1/online-schema-changes) 与 Cockroach，建议在应用迁移之前手动复制索引定义并运行它。然后迁移将不会执行任何操作。

    如果你在 **mysql**（或 **sqlite**）上运行，不会创建特殊索引。如果需要，你可以手动创建这样的索引，如果能贡献其定义将不胜感激。

    此功能是预览版，并且将改变行为！相似性搜索不应返回确定性结果，但对人类有用。

+   允许导入 HMAC 散列密码（[#3544](https://github.com/ory/kratos/issues/3544)）（[0a0e1f7](https://github.com/ory/kratos/commit/0a0e1f7200e226ef24de062811a05bcdd02b6acd)），关闭 [#2422](https://github.com/ory/kratos/issues/2422)：

    基本格式为 `$hmac-<hashfunction>$<base64 编码的哈希>$<base64 编码的密钥>`：

    ```
    # password = test; key=key; hash function=sha
    $hmac-sha1$NjcxZjU0Y2UwYzU0MGY3OGZmZTFlMjZkY2Y5YzJhMDQ3YWVhNGZkYQ==$a2V5 
    ```

+   允许在注册过程中将 OIDC 提供者验证的地址标记为已验证（[#3448](https://github.com/ory/kratos/issues/3448)）（[e7b33a1](https://github.com/ory/kratos/commit/e7b33a168bf0c0fe0492901abd3df8b6d6a08a68)），关闭 [#3445](https://github.com/ory/kratos/issues/3445) [#3424](https://github.com/ory/kratos/issues/3424) [#1057](https://github.com/ory/kratos/issues/1057)：

    此功能允许将由社交登录提供者提供的电子邮件标记为已验证。

+   批量列出身份信息（[#3598](https://github.com/ory/kratos/issues/3598)）（[8ad54f1](https://github.com/ory/kratos/commit/8ad54f1be53b30fdb24b616be0c52fd66829f201)），关闭 [#2448](https://github.com/ory/kratos/issues/2448)：

    此更改允许通过以下语法来筛选 `GET /admin/identities` 的 ID：

    ```
    /admin/identities?ids=id1&ids=id2&ids=id3 
    ```

+   **变更日志：** 添加对本地恢复的支持（[#3624](https://github.com/ory/kratos/issues/3624)）（[492808c](https://github.com/ory/kratos/commit/492808cae0e804793aef9a02a902fce988f9fc6d)）：

    添加了在 API 流程上正确完成恢复流程的能力。此 PR 还简化了 SPA 流程的行为，不再返回 422 错误。要启用此新行为，请在配置中设置 features.use_continue_with_transitions 标志为 `true`。

    另请参见 [#3273](https://github.com/ory/kratos/pull/3273)

+   来自 userinfo 端点的声明（[#3718](https://github.com/ory/kratos/issues/3718)）（[90bdc61](https://github.com/ory/kratos/commit/90bdc61d28466f10e4e609df014b220afbee0478)）：

+   在 API 流程中发现冗余的 cookie 时，通过发出错误详情来发出错误提示（[#3496](https://github.com/ory/kratos/issues/3496)）（[df74339](https://github.com/ory/kratos/commit/df74339802d98a292abb32806eca35fb2554960b)）

+   最终一致性 API 控制（[#3558](https://github.com/ory/kratos/issues/3558)）（[00cf11c](https://github.com/ory/kratos/commit/00cf11c071344103c603c078f07196401d091780)）：

    添加了在 Ory Network 中使用的功能，该功能使得可以通过略有过时的数据来交易更快的读取。

    此功能依赖于 Cockroach 的功能和配置，并且对于 MySQL 或 PostgreSQL 是不可能的。

+   扩展 Microsoft Graph API 的能力（[#3609](https://github.com/ory/kratos/issues/3609)）（[4a7bcc9](https://github.com/ory/kratos/commit/4a7bcc9322be37e6fd141e411bd65e3977eeb692)）：

    此更改查询所有可用于 `User.Read` 范围的用户信息。

    在 OIDC 过程中，并填充 `RawClaims` 字段。

+   从默认身份架构中提取登录的标识符标签 ([#3645](https://github.com/ory/kratos/issues/3645)) ([180828e](https://github.com/ory/kratos/commit/180828eb507ab239a9c6589f747a6816b6e50074))

+   所有可用流程方法的精细化钩子 ([#3519](https://github.com/ory/kratos/issues/3519)) ([a37f6bd](https://github.com/ory/kratos/commit/a37f6bddc48443b2fc464699fa5c2922f64d81f6)):

    为 totp、webauthn、lookup_secret 方法在设置后流程和 totp、lookup_secret、code 在登录后流程添加了精细化的钩子配置。

+   在密码更改后钩子以撤销会话 ([#3514](https://github.com/ory/kratos/issues/3514)) ([e6af6db](https://github.com/ory/kratos/commit/e6af6db37ff5de33a656ce7804c813451395459d)), 关闭 [#3513](https://github.com/ory/kratos/issues/3513):

    目前，Kratos 系统在用户更改密码时不会自动注销或使其他活动会话失效。这带来了重大安全风险，因为它允许潜在的未授权个体即使在密码更新后仍然保持对账户的访问权限。

    此 PR 提供了在自助服务设置的操作部分中添加`revoke_active_sessions`钩子的选项。

+   热重载 CORS 来源 ([#3423](https://github.com/ory/kratos/issues/3423)) ([157d934](https://github.com/ory/kratos/commit/157d9345aeb04f371f9d85b70c89e8646e781333))

+   改进消息以便更易国际化 ([#3457](https://github.com/ory/kratos/issues/3457)) ([37f1657](https://github.com/ory/kratos/commit/37f16577d92ba88869bf15fb1ea54e819b062724))

+   通过在验证时计算密码哈希来提高性能 ([#3508](https://github.com/ory/kratos/issues/3508)) ([a9786c5](https://github.com/ory/kratos/commit/a9786c599d09f61e2e07df5066ce94feb2d99bac))

+   改进了 Webhook 的跟踪功能 ([#3746](https://github.com/ory/kratos/issues/3746)) ([9d7021d](https://github.com/ory/kratos/commit/9d7021d87f47690c2c1f8000e87b425e49bc9496))

+   为 OIDC 声明映射器、Webhook、JWT 会话令牌器提供 Jsonnet 缓存 ([#3701](https://github.com/ory/kratos/issues/3701)) ([1d26e09](https://github.com/ory/kratos/commit/1d26e097b273aeda36f73637765da5bdb2aa4a66))

+   在登录时链接 oidc 凭据 ([#3563](https://github.com/ory/kratos/issues/3563)) ([b784949](https://github.com/ory/kratos/commit/b784949d03b849d9d1d594977f75f5843b7b5da8)), 关闭 [#2727](https://github.com/ory/kratos/issues/2727) [#3222](https://github.com/ory/kratos/issues/3222):

    当用户首次尝试使用 OIDC 登录但已经先前注册过电子邮件/密码时，Kratos 可能会检测到凭据标识符冲突。在这种情况下，用户需要先用电子邮件/密码登录，然后在设置屏幕上链接 OIDC 凭据。

    此 PR 简化了用户体验，并允许用户在登录流程中直接将 OIDC 凭据链接到现有账户。

    切换至设置流程。

+   通过 OIDC 凭据列出 ([#3721](https://github.com/ory/kratos/issues/3721)) ([bff9c61](https://github.com/ory/kratos/commit/bff9c61b147648ab139e7e86cda4336b5d1cfd39))

+   通过任何凭据类型使用代码登录 ([#3549](https://github.com/ory/kratos/issues/3549)) ([ceed7d5](https://github.com/ory/kratos/commit/ceed7d5478c5cca894587698c57f676dda100b27)):

    用户即使未在`code`凭据上注册，也应能够使用`code`凭据登录。

    仅执行`identifier`匹配并基于标识模式进行验证。

+   一次性代码本地流程 ([#3516](https://github.com/ory/kratos/issues/3516)) ([9b0fee3](https://github.com/ory/kratos/commit/9b0fee30f980d860fd548e7589fa6a06e593537a))

+   按创建时间排序会话 ([#3696](https://github.com/ory/kratos/issues/3696)) ([688111c](https://github.com/ory/kratos/commit/688111c9a6bf9872657cf6aada77f55fa2520e00))

+   参数化快递工作者 ([#3601](https://github.com/ory/kratos/issues/3601)) ([0e4be57](https://github.com/ory/kratos/commit/0e4be57e41e1152f4be22f490541c2c099cfe3fe)):

    允许参数化快递将获取多少条消息以及多久获取一次消息。

+   通过电子邮件进行无密码浏览器登录和注册代码 ([#3378](https://github.com/ory/kratos/issues/3378)) ([eaaf375](https://github.com/ory/kratos/commit/eaaf37519917612671238412a633847386d7c613)), closes [#2029](https://github.com/ory/kratos/issues/2029) [ory-corp/cloud#3573](https://github.com/ory-corp/cloud/issues/3573):

    此功能添加了无密码的电子邮件代码登录。当用户注册或登录时，将向其电子邮件地址发送代码，用户可使用该代码完成认证过程。

    当前此功能仅适用于面向浏览器的API。

+   池化的进程隔离 Jsonnet VM ([9a52ddf](https://github.com/ory/kratos/commit/9a52ddfbe7c24c41b6aa3ddc3c79c6fcbfb8db02))

+   在注册失败时由于重复凭据/地址提供登录提示 ([#3430](https://github.com/ory/kratos/issues/3430)) ([8b28469](https://github.com/ory/kratos/commit/8b284697e4a26fb01ad57d2e9ebd8f714be49f33)):

    +   feat: 在由于重复凭据或标识符导致注册失败时提供登录提示

    +   feat: 识别边缘情况并编写测试

    +   chore: 同步工作空间

    +   feat: 使登录提示可配置

    +   chore: 同步工作空间

    +   chore: 同步工作空间

    +   chore: 同步工作空间

    +   chore: 同步工作空间

+   支持 auth_type 参数 ([#3487](https://github.com/ory/kratos/issues/3487)) ([fc30304](https://github.com/ory/kratos/commit/fc303040b71139f512fd1491ce30f80837b940b9)):

    Facebook OIDC 提供者支持 auth_type 参数，该参数

    当设置为 "重新认证" 时将强制用户

    重新认证（类似于其他提供者的 `prompt=login`）。

+   支持 B2B SSO ([#3489](https://github.com/ory/kratos/issues/3489)) ([0ec037a](https://github.com/ory/kratos/commit/0ec037ab298ed28fb0ac84db6a4d2b14b81e57df))

+   支持通过短信进行多因素认证 ([#3682](https://github.com/ory/kratos/issues/3682)) ([1516cf6](https://github.com/ory/kratos/commit/1516cf64e346819dccace1cc25aaccac38b9e47c))

+   支持多个 WebAuthN 来源 ([#3380](https://github.com/ory/kratos/issues/3380)) ([013f335](https://github.com/ory/kratos/commit/013f335881831bbf90ac31b219b57118fc089fe6)):

    现在用户可以在配置中提供 WebAuthn 的一系列来源。

+   支持使用苹果 SDK 进行本地社交登录 ([#3476](https://github.com/ory/kratos/issues/3476)) ([f561013](https://github.com/ory/kratos/commit/f561013dd737dadcc82c4ec049fde12861e91e43))

+   在接受登录时将当前会话 ID 传输给 Hydra ([#3426](https://github.com/ory/kratos/issues/3426)) ([610c76d](https://github.com/ory/kratos/commit/610c76d9140f2f43217ac55094051a994ea83ecc)):

    +   chore: 将 React Native 端口更改为 19006

    +   feat: 在接受登录时传输当前会话 ID

    +   fix: 在测试中升级 Hydra

+   Webhook 分析事件 ([9c8a25e](https://github.com/ory/kratos/commit/9c8a25eb0d3e06df182565d3d959d57e5dccfed8))
