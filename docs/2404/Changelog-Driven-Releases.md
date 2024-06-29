<!--yml

类别：未分类

日期：2024-05-27 13:28:43

-->

# 变更日志驱动的发布

> 来源：[https://mathieularose.com/changelog-driven-releases](https://mathieularose.com/changelog-driven-releases)

<main>

<hgroup>

# 变更日志驱动的发布

2024年4月

</hgroup>

曾经发布过未经测试的开源项目版本或者忘记更新版本号？我们都有过这样的经历。管理开源项目的发布可以感觉像是一个繁琐、容易出错的过程，特别是当手动和自动化步骤混合进行时。本文介绍一种方法，将您的变更日志转变为发布的最终真相来源，使您项目的发布完全自动化。

## 变更日志作为发布的真相来源

想象一个变更日志不仅作为历史记录，而且作为发布的真相来源。这正是我为 [utt](https://github.com/larose/utt) 采用的方法，这是我维护的一个开源项目。变更日志按照逆时间顺序排列，最新版本位于顶部。这里是一个概述：

```
## 1.30 (2024-01-17)

  * Add 'per-task' CSV report type

## 1.29 (2021-01-16)

  * Show total time in summary section
  * Remove redundant "time" label from summary section
... 
```

### 未发布的变更

新变更在 utt 的变更日志中以特殊的`(unreleased)`标记添加。这允许在一起发布多个变更。

这是它的工作原理：

```
## (unreleased)

  * Fix bug X
  * Enhance feature Y 
```

### 发布一个版本

准备发布时，我只需用所需的版本号和日期替换`(unreleased)`：

```
## 1.31 (2024-04-20)

  * Fix bug X
  * Enhance feature Y 
```

将此更改合并到主分支将触发 utt 中的自动化发布流程，包括运行自动化测试。

## 实现

虽然 utt 中具体实现使用了 Python 和 GitHub Actions（见[utt的工作流](https://github.com/larose/utt/blob/e31c7340cd4c151b7a5bfa004b676ca72149978f/.github/workflows/publish.yml)），但这些概念可以适用于大多数编程语言和 CI/CD 工具。

核心工作流程涉及一个脚本，该脚本在推送事件到主分支时解析变更日志。如果最新条目不是`(unreleased)`，脚本会自动更新相关项目文件中的版本号（例如 pyproject.toml、version.txt）。

最后，如果版本不是`(unreleased)`，脚本将尝试发布新版本到 PyPI，如果还没有发布到 PyPI 的话。如果版本已存在，脚本会跳过发布步骤，以防止再次尝试相同版本。

</main>
