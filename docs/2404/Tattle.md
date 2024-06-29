<!--yml

类别：未分类

日期：2024年5月27日12:57:33

-->

# Tattle

> 来源：[https://tattle.co.in/blog/2024-04-03-securing-feluda-pt1/](https://tattle.co.in/blog/2024-04-03-securing-feluda-pt1/)

## 保护您的代码库

发表于2024年4月2日Aurora - Feluda

我们最近对[Feluda](https://github.com/tattle-made/feluda)进行了许多改进，遵循安全优先的方法，以支持长期维护和积极贡献。这些方法不仅适用于Feluda，还可以应用于任何软件项目。这些文章是为技术人员撰写的，我们希望它们帮助其他项目学习如何为更安全的数字体验实施这些实践。

在构建软件项目时，每个开发人员都应确保他们编写的代码在被接受到代码库之前是安全的。虽然这个想法令人望而生畏，但通过当前的安全工具，学习曲线大大降低，这些工具也充当教学辅助工具。这种方法确保了维护安全责任不仅仅落在单独的网络安全团队身上，以修复代码已经进入存储库后的漏洞。因此，在开发管线的早期阶段或更向左的地方进行bug修复。当开发周期被视为从左到右开始和结束时，这种方法也被称为[向左转移](https://en.wikipedia.org/wiki/DevOps#DevSecOps,_shifting_security_left)方法。该过程是“DevSecOps”团队旨在实现的一部分。正如名称所示，这包括开发、安全和运营。这些团队有不同的目标，通常是[不对齐的](https://www.youtube.com/watch?v=EB5yT-GS460)。开发人员希望更快的开发，运营人员希望稳定性，而网络安全人员则希望安全代码。DevSecOps遵循[CALMS框架](https://www.atlassian.com/devops/frameworks/calms-framework) - 文化、自动化、精益、测量和分享。该方法倡导文化转变，朝向协作，为可重复和稳定的系统自动化组件，以敏捷的方式实施过程，从实施的过程中衡量改进，并分享知识，打破团队之间的隔离，增加信任、敏捷性和可靠性。

我们现在将讨论如何在开发周期中为Feluda实施自动化，以提高安全性、减少技术债务、增强代码健壮性和稳定性。

我们的目标是确保不安全的代码不进入Feluda代码仓库。为此，我们设置了每当有人向Feluda贡献代码时运行的检查，即每当在Feluda仓库的任何分支中打开Pull Request（PR）时运行检查。这种自动化测试是使用[持续集成](https://en.wikipedia.org/wiki/Continuous_integration)（CI）流水线设置的。如果检查失败，将自动阻止将更改合并到代码库中的选项。在Github中，这些操作通过[Github Actions Workflows](https://docs.github.com/en/actions/using-workflows)完成。您可以在Feluda代码库的[这里](https://github.com/tattle-made/feluda/tree/main/.github/workflows)查看。

将所有测试作为预提交检查的一部分运行在CI流水线中是良好的做法。预提交检查在开发人员尝试在本地提交代码时每次运行。如果检查失败，本地提交将失败，强制开发人员在等待PR失败之前修复错误。然而，预提交检查也有缺点。首先，当使用GUI工具来[提交更改](https://github.com/typicode/husky/issues/390#issuecomment-436966021)时，预提交检查可能会默默失败运行。更重要的是，预提交检查依赖于开发人员不是恶意的并且不会在本地绕过这些检查来提交更改的假设。因此，将所有检查作为CI流水线的一部分运行是至关重要的。

安全重点软件项目的[检查清单](https://www.youtube.com/watch?v=cmWQF2FDlG8)

1.  确保提交的代码没有错误的基本检查是确保在合并新代码后现有功能仍然正常工作。为此，有足够的[单元测试](https://en.wikipedia.org/wiki/Unit_testing)和[集成测试](https://en.wikipedia.org/wiki/Integration_testing)以及足够的[代码覆盖率](%5Bhttps://en.wikipedia.org/wiki/Code_coverage)是至关重要的。我们的团队继续为每个新功能和现有功能添加测试，以便在开发过程中测试我们不会破坏功能。此外，我们设置了一个Github工作流，用于[运行测试](https://github.com/tattle-made/feluda/blob/main/.github/workflows/pr-tests.yml)以验证每个创建的PR。

1.  接下来的步骤是将代码检查作为CI流水线的一部分添加。语言特定的[linter](https://en.wikipedia.org/wiki/Lint_(software))检查基本的语法和格式问题。虽然linting不会指示安全问题，但它确实指出了编写不良代码，这些代码可能目前有错误或最终会变得有错误。因此，早期修复这些问题比积累可能导致未来安全和稳定问题的技术债务更为重要。

1.  大多数现代软件使用软件包存储库来获取外部依赖项。这些存储库包括 Java 的 Gradle 和 Maven，Python 的 PyPi，以及 Node.js 项目的 NPM 注册表。大多数软件包存储库维护着每个项目版本中已知漏洞的数据库。这些漏洞不一定是故意的恶意软件包，而是在产品发布后发现的漏洞。语言特定的工具通常具有审计工具，用于检查您正在使用的软件包与漏洞数据库的匹配情况。例如，您可以使用 `pip audit` 命令检查来自PyPi的依赖项，或者使用 `npm audit` 检查来自NPM注册表的依赖项。漏洞数据库通常会下载并在本地运行检查，以防止您的代码中的安全问题暴露给第三方服务器。如果发现安全问题，修复通常只需将软件包升级到较新版本。审计工具提供自动修复选项或更多信息（如果没有修复）。如果修复不可用，则需要选择更安全的依赖项或等待包含修复的更新版本。无论哪种情况，您现在知道您的代码库存在已知的漏洞。依赖关系审计在代码开发过程中是基本的检查，但在CI系统中变得至关重要，因为提交的新代码可能添加了一个新的已知恶意软件包或将现有软件包降级到一个易受攻击的版本。在Feluda中，我们为每个PR运行依赖关系审计。

1.  尽管依赖关系审计是良好的实践，但始终使用所有软件工具和依赖项的最新版本是最佳实践。这样可以尽快应用程序修复，防止未发现的漏洞和已知的稳定性问题。这可以通过[软件组成分析](https://en.wikipedia.org/wiki/Software_composition_analysis)（SCA）工具实现，这些工具会扫描您的代码库以查找所有依赖项，并可以自动为您创建更新软件包的PR。这些工具还会在发现依赖项中的新漏洞时立即向您发出警报。Github 的 Dependabot 是一个易于设置的工具，专门用于此目的。开源社区中的一个好的替代品是 [Renovate](https://github.com/renovatebot/renovate)。直接通过SCA工具创建的PR进行软件包升级时，确保在CI中运行单元测试和集成测试以确保自动化软件包更新不会破坏软件功能显得至关重要，尤其是在存在主要版本更新（意味着有破坏性更改）时更为重要。

1.  使用的一个必要的安全工具是扫描您的代码库以查找秘密信息。这些可能包括加密密钥或密码，用于您的开发帐户或生产服务器。重要的是尽早设置此工具，以确保秘密信息不会故意或无意地提交到代码库并公开泄露。秘密扫描工具存在是因为这种疏忽很常见。Github 提供了一个[秘密扫描](https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning)工具。一个好的开源选择是[TruffleHog](https://github.com/trufflesecurity/trufflehog)。如果您意外地将秘密信息提交到您的存储库中，最好是[轮换您的密钥](https://cheatsheetseries.owasp.org/cheatsheets/Key_Management_Cheat_Sheet.html)。防止意外提交秘密信息的一个好方法是在开发过程中使用环境文件来存储秘密，并将这些文件添加到您的`.gitignore`文件中，以确保这些文件永远不会上传。

1.  下一步是添加一个[静态分析](https://en.wikipedia.org/wiki/Static_program_analysis)工具，它将检查您的代码中的漏洞。这是自动化"[蓝队战术](https://en.wikipedia.org/wiki/Blue_team_(computer_security))"或加固防御的重要部分。有多个开源的SAST工具可用。一些适用于多种语言，而另一些则专为特定语言设计。这些工具具有不同的误报率和漏报率。它们通常会为发现的漏洞分配一个风险评分，并提供有关漏洞存在于代码中的位置、为何这是不良编码实践、如何评估其是否真正存在风险以及如何减轻风险的信息。这些工具可能会提供CVSS评分[通用漏洞评分系统](https://en.wikipedia.org/wiki/Common_Vulnerability_Scoring_System)或CWE[(通用弱点枚举)](https://en.wikipedia.org/wiki/Common_Weakness_Enumeration)以获取额外的信息以评估和修复问题。这是开始解决安全问题时可能变得艰巨的步骤。您需要能够评估风险，研究最佳方法来应用修复，并确保修复不会为您的代码增加额外的漏洞。然而，重要的是要理解，您可能遇到的许多安全问题可能是低复杂性问题，并且对于具有足够技术知识的大多数开发人员来说很容易修复。在这个阶段最好有经验丰富的人士提供建议并审查您的代码。在Feluda中，我们使用开源的特定于Python的SAST工具[bandit](https://github.com/PyCQA/bandit)，在每个PR上自动化漏洞扫描。

1.  Feluda是一个部署在托管生产机器上运行的服务器端项目。因此，我们的生产部署流程中包含大量的基础设施即代码（IaC）文件。IaC文件可以包括dockerfile和kubernetes配置文件。确保IaC代码本身没有安全问题并且遵循最佳实践非常重要。这些问题可以通过专门用于扫描IaC代码的工具来发现。对于Feluda，我们使用开源的IaC扫描工具[Trivy](https://github.com/aquasecurity/trivy)来扫描生产配置的漏洞。修复IaC漏洞需要掌握加固配置文件的技能，并可能需要深入研究。

注意：SAST工具和IaC漏洞扫描工具不断更新，采用更好的扫描技术和/或更新的漏洞数据库。因此，作为CI的定期cron作业的最佳实践，每周运行这些工具可以检测现有代码中的新漏洞。

1.  我们还使用[OSSF Scorecard](https://github.com/ossf/scorecard)静态供应链安全分析工具来评估Feluda的安全状况。该工具能够建议修复措施以防止供应链攻击，这种攻击越来越普遍。检测到的问题可能需要简单的依赖关系固定修复。

1.  最后 - 软件项目永远不能保证100%安全。核心团队也不可能单独检测项目中的每一个漏洞。因此，提供一种方法让外部研究人员安全提交安全问题而不必担心报复是至关重要的。为此，我们为Feluda撰写了一个安全政策，可以在[这里](https://github.com/tattle-made/feluda/security/policy)访问。撰写安全政策需要基本的网络安全法律理解，以及如何在实践中运作，报告安全问题的最佳实践和漏洞披露流程。

保护代码是一个持续的过程，并不一定是技术性的。人的因素至关重要，因为它需要建立协作环境，避免责备，从错误中学习，并建立信任。
