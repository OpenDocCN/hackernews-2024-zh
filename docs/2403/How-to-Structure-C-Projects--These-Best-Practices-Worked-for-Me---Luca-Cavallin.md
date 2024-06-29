<!--yml

分类：未分类

日期：2024-05-27 14:44:30

-->

# 如何为 C 项目进行结构化：这些最佳实践对我很有效 | Luca Cavallin

> 来源：[https://www.lucavall.in/blog/how-to-structure-c-projects-my-experience-best-practices](https://www.lucavall.in/blog/how-to-structure-c-projects-my-experience-best-practices)

我最近参与了两个不同的 C 项目，我希望以一种易于维护和理解的方式来构建它们。我还希望确保项目易于构建和测试。在本文中，我将分享我的经验和我为结构化 C 项目找到的最佳实践。

你可以在以下的仓库中找到我参与的项目的源代码：

由于我在 C 语言上的经验有限 - 十年来没有写过任何东西 - 我不得不进行大量的研究，以了解当涉及到目录布局时当前的共识是什么。我在 GitHub、Reddit 和 Stack Overflow 上找到了许多有用的信息。我还查看了一些流行的开源 C 项目的源代码，以了解它们的结构。我发现我查看的大多数项目都遵循了类似的布局，并决定以此作为起点。

## [](#what-the-internet-people-say)互联网用户的看法

如果你在谷歌上搜索“C 项目结构最佳实践”，你会得到约 5.83000000 个结果 - 无需担心自己的研究 - 我已经两次阅读了所有这些页面。尽管意见不一，但有一些共同的主题一再出现。有两种方法特别受欢迎：

+   **“模块化”方法**：这是大型项目中最常见的方法。其思想是将项目分割为多个目录，每个目录包含项目的不同模块。每个模块都有自己的头文件、源文件和测试。这种方法使得查找代码变得容易，并且可以轻松地单独测试每个模块。例如，这是 [Linux 内核](https://github.com/torvalds/linux) 的结构方式。

+   **“平面”方法**：这种方法对于小型项目更为常见，它侧重于尽可能简单和良好组织的项目。

因为我参与的项目比较小，我决定采用“平面”方法，接下来我会详细描述。

## [](#my-approach-to-structuring-c-projects)我对 C 项目结构化的方法

经过两次查看了所有的 5.83000000 个结果后，我为我的项目选择了以下的目录布局：

```
├── .devcontainer           configuration for GitHub Codespaces ├── .github                 configuration GitHub Actions and other GitHub features ├── .vscode                 configuration for Visual Studio Code ├── bin                     the executable (created by make) ├── build                   intermediate build files e.g. *.o (created by make) ├── docs                    documentation ├── include                 header files ├── lib                     third-party libraries ├── scripts                 scripts for setup and other tasks ├── src                     C source files │   ├── main.c             (main) Entry point for the CLI │   └── *.c ├── tests                   contains tests ├── .clang-format           configuration for the formatter ├── .clang-tidy             configuration for the linter ├── .gitignore ├── compile_commands.json   compilation database for clang tools ├── LICENSE ├── Makefile └── README.md 
```

让我们深入了解这种布局。我们现在可以忽略 `.devcontainer`、`.github`、`.vscode` 和 `scripts`，因为它们特定于我的开发环境，与项目结构无关。`.clang-format` 和 `.clang-tidy` 是 [Clang](https://clang.llvm.org/) 格式化和静态分析工具的配置文件。这些文件并不是严格必需的，但如果你想在项目中使用 Clang 工具，它们可能会有用。`LICENSE` 和 `README.md` 是不言自明的，`Makefile` 也无需介绍，尽管你可以在 [为 C 项目编写一个干净、可维护和易于理解的 Makefile](/blog/crafting-clean-maintainable-understandable-makefile-for-c-project) 中了解更多关于我如何编写的信息。

在我们深入讨论更重要的细节之前，让我们先了解几个其他目录：

+   `bin` 目录包含运行 `make` 时创建的可执行文件。

+   `build` 目录包含中间构建文件，如 `.o` 文件。

+   `docs` 目录包含项目的文档。

让我们花一些时间了解 `src`、`include`、`lib` 和 `tests` 目录。

### [](#the-src-directory)`src` 目录

`src` 目录包含项目的 C 源文件，你会在这里花费大部分时间。我决定保持简单，使用平面布局。除了作为程序入口文件的 `main.c` 文件外，我根据"关注点"和数据结构分割了其余的代码。例如，在 `gnaro` 项目中：

+   `btree.c`：包含 B 树数据结构的实现。

+   `cursor.c`：包含用于读写数据库的游标的实现。

+   `database.c`：包含数据库的实现。

+   `pager.c`：包含分页器的实现。

+   `row.c`：包含数据库中行的实现。

+   `input.c`，`meta.c` 和 `statement.c`：包含解析和准备用户输入所需的逻辑。

我发现这种简单的布局易于理解和导航。只要你积极努力保持文件的小和专注，就很容易找到你正在寻找的代码。这种方法的缺点是，你需要确保 `Makefile` 更新了你添加到项目中的新文件，以便正确编译和链接它们。鉴于我工作的项目规模较小，我并没有发现这是一个问题，但对其他人可能会是一个问题。

### [](#the-include-directory)`include` 目录

`include` 目录包含项目的头文件。`src` 目录中的大部分（如果不是全部）`.c` 文件都会在 `include` 目录中有对应的 `.h` 文件。头文件应包含模块的公共 API，而源文件应包含实现。这样可以方便地查看模块的功能，而无需查看实现。它还使得能够轻松地在隔离环境中测试模块，因为你可以直接在测试文件中包含头文件。

再次以 `gnaro` 项目为例：

+   `btree.h`：包含在 `src/btree.c` 中，定义了 B 树数据结构的公共 API。

+   `cursor.h`：包含在 `src/cursor.c` 中，定义了用于从数据库读取和写入的游标的公共 API。

+   `database.h`：包含在 `src/database.c` 中，定义了数据库实现的公共 API。

+   `pager.h`：包含在 `src/pager.c` 中，定义了分页逻辑的公共 API。

+   `row.h`：包含在 `src/row.c` 中，定义了数据库行结构的公共 API。

+   `input.h`、`meta.h` 和 `statement.h`：包含在 `src/input.c`、`src/meta.c` 和 `src/statement.c` 中，定义了处理用户输入的公共 API。

就像 `src` 目录一样，此布局的缺点是头文件应在 `Makefile` 中引用，以便它们在编译过程中被包含。

### [](#the-lib-directory)`lib` 目录

`lib` 目录包含项目依赖的第三方库。例如，[lucavallin/gnaro](https://github.com/lucavallin/gnaro) 使用 [argtable](https://www.argtable.org) 和 [log.c](https://github.com/rxi/log.c) 库来分别解析 CLI 参数和记录日志。

关于此目录没有太多可以说的。它只是一个放置依赖项的地方。同样，别忘了在 `Makefile` 中包含它们。

### [](#the-tests-directory)`tests` 目录

`tests` 目录包含项目的测试。我使用 [CUnit](https://cunit.sourceforge.net) 库进行测试，并发现它非常适合我的需求。`tests` 目录包含了 `src` 目录中每个模块的一个测试文件。例如，在 `gnaro` 项目中，`tests` 目录包含了 `gnaro_test.c` 文件，该文件用于测试 `src/gnaro.c` 中定义的逻辑。

此时，在实践中，该文件仅包含了根据 [CUnit](https://cunit.sourceforge.net) 文档推荐设置测试所需的代码。虽然测试可以运行，但我实际上没有继续编写有用的检查功能，因为它们只是副业和兴趣项目。

## [](#conclusion)结论

感谢您阅读至此！希望您觉得这篇文章有用。我知道我描述的项目布局并不是组织C项目的唯一方式，但这是我发现对我最有效的方式。`bin`、`build`、`docs`、`script` 和 ".something" 目录对开发非常有帮助，但真正工作发生在 `src`、`include`、`lib` 和 `tests` 目录中。

+   `src`：包含项目的C源文件。

+   `include`：包含项目的头文件，被 `src` 中的 `.c` 文件包含。

+   `lib`：包含项目依赖的第三方库。

+   `tests`：包含项目的测试。

`Makefile` 是将所有内容粘合在一起的关键，必须更新以包含新文件和依赖关系。虽然现代化的构建系统如 CMake 和 Meson 已经出现，但我发现一个简单的 `Makefile` 对我的需求已经足够。

希望您能从我在这里提出的一些想法中获益，并应用到您自己的项目中。如果您有任何问题或意见，请随时联系我！
