<!--yml

类别：未分类

日期：2024-05-27 14:24:52

-->

# software-development/commit-messages.md at master · aaronjensen/software-development · GitHub

> 来源：[`github.com/aaronjensen/software-development/blob/master/commit-messages.md`](https://github.com/aaronjensen/software-development/blob/master/commit-messages.md)

← 文章

# 以主题为先的提交消息

[](#subject-first-commit-messages)

注意

这篇文章只描述了我们开发过程的一小部分。我们所有做法的目标都是使开发人员在旧项目上和新项目上一样高效。

我建议先阅读这篇文章，它描述了这个目标：

目标：连续性

抽出一点时间，快速扫描而不是阅读以下列表：

```
Fix `OrderedOptions#dig` for array indexes
Handle outdated Marshal payloads in Cache::Entry with 6.1 cache_format
Fix time travel helpers to work when nested using with separate classes
Added rails/railtie requires to AS/i18n_railtie.rb and AS/railtie
Fixed even more missing requires
Fix active_support/*/conversions by requiring missing files
Fix file cache store to split url-encoded keys on encode-sequence boundaries
Use RDoc block quote syntax [ci-skip]
Use custom class for pending migrations connection
Typo fix in BigDecimal job arguments warning
Fix code example in the field_name method
Drop dependency on mutex_m
Include `ActiveModel::API` in `ActiveRecord::Base`
Update classic_to_zeitwerk_howto.md
Add description for error_highlight gem
Fix extra blank line when no error_highlight
Bump psych to 5.1.1.1
Fix failing linter in `guides/source`
[ci skip] Fix shard docs followup
[ci skip] Fix shard docs
Revert "Add psych to the bug report template"
Add psych to the bug report template
Whitespaces
Avoid __method__
We don't use `::` to denote class methods 
```

你是否注意到了关于 `field_name` 方法的那一行？

然后，用同样的方式扫描这些行。再次快速扫描，不要阅读：

```
Snapshot interval build argument overrides any snapshot interval declared by the class macro
EntityStore build method records the specifier argument when given
Category macro is optional on EntityStore when a category is given to the constructor
Tests are clarified
Category can be configured when constructing a store
Category declaration test is moved
Specifier can be configured when constructing a store
Specifier macro is implemented
Instance configure is invoked before the instance is assured
Instance configure template method called from constructor
Store's project method is an alias for fetch
Title is changed
Parenthesis are added to assert_raises and refute_raises
Automated test runner supplies exclude file pattern directly into CLI
Test files are compatible with TestBench 2.0
Unbalanced parenthesis in a log output is corrected
Log tags are standardized
Library-level log tags reduced to 'entity_store'
Snapshot classes must implement assurance
Snapshot implementation assures snapshot
Default batch size is sourced from reader class
Default reader batch size is 1000
Superflous test details are removed
Update of entity in substitute is tested
Updating substitute with new entity is tested 
```

你看到了关于 Store 的消息吗？

第一个列表是来自[Rails](https://github.com/rails/rails)的提交消息样本。第二个是来自[Eventide 中的 `entity-store` 项目](https://github.com/eventide-project/entity-store)。除非你已经在 Eventide 社区的轨道上，否则第一个提交消息样式可能是你所习惯的。很可能是你的团队所使用的，因为这是通常推荐的提交消息样式，被认为是一种"最佳实践"。

第二种样式可能感觉陌生，可能不太舒服。它是被动语态和陈述语气，而不是更常见的主动语态和祈使语气。那么为什么 Eventide 项目和许多使用 Eventide 的团队选择使用这种以主题为先的提交消息样式呢？因为它更易于扫描。它经过了人类认知的优化。人们在查看列表时通常不会阅读，除非他们绝对需要。我们会扫描。当我们扫描时，我们希望首先看到最重要的事情。此外，以主题为先的提交消息样式使提交与变更相关，而不是与做出变更的人相关。它不是关于个人。它关乎代码。

## 检查你自己的仓库

[](#check-your-own-repository)

在你的一个仓库中尝试这个：

```
git log --oneline | grep -v Merge | cut -d' ' -f2 | head -50
```

你看到了什么？想象一下，当你快速扫描时，你只看到了这些内容。这并不是一个完美的模拟，因为当我们扫描时，我们可能会进一步模式匹配字符串，我们可能会读取两个单词左右，我们可能会教自己第一个单词是无关紧要的并试图跳过它，但所有这些都需要额外的努力。我们的工作已经够难了，所以我们要尽一切可能让它变得更容易。

像我们所有的过程和设计决策一样，我们决定使用这种技术是基于先验知识、研究、第一原则和观察。以下是一些与列表处理相关的研究链接（这通常是提交消息摘要的消费方式）。请注意，我们在做出决定时并没有参考这些具体的文章，这些只是例子。

当我第一次遇到 Eventide 项目使用的主题优先命名风格时，我并不喜欢它。我不喜欢它是因为我的个人偏见和我倾向于符合“常规”的偏好。它看起来不像我习惯的样子，最初我反对它。我最终认识到了可扫描性的好处，并决定在我的当前团队中尝试它。习惯用那种风格写信息需要一些时间，每个新的团队成员都需要一定的培训和强化。我们现在已经使用了这种风格三年了，我们的团队现在对它有了强烈的偏好。

我建议在你的项目中尝试这种风格。一旦你克服了最初的反应，并习惯了用这种风格写作和阅读信息，你可能会感到惊讶。

这里是我们遵循的一些指南：

+   如果引入新事物，你并不总是需要一个动词。例如，“小部件测试”优先于“添加小部件测试”。

+   我们更喜欢使用“已更正”而不是“已修复”。例如，“小部件对账已更正”。

+   当某事被重构或以某种方式改进时，我们倾向于使用“已澄清”。例如，“小部件创建已澄清”。

+   在描述重命名时，使用“而不是”。例如，“小部件，而不是齿轮”。

+   描述版本增加时，要明确指出新旧版本。例如，“包版本从 1.1.1 增加到 1.2.0”。

+   不要费心遵循 50 字符主题规则，让第一行尽可能长。简洁是有用的，但在我看来，50 个字符太限制了。

* * *

[评论](https://github.com/aaronjensen/software-development/discussions/4)

[订阅以接收新文章通知](https://github.com/aaronjensen/software-development/discussions/8)

[所有文章](https://github.com/aaronjensen/software-development/blob/master/README.md#articles)

* * *

版权所有 Aaron Jensen 2023-present
