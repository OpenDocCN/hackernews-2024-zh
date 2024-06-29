<!--yml

category: 未分类

date: 2024-05-27 13:37:34

-->

# PowerShell：你不知道自己需要的面向对象 shell | Chris Warrick

> 来源：[https://chriswarrick.com/blog/2024/04/29/powershell-the-object-oriented-shell-you-didnt-know-you-needed/](https://chriswarrick.com/blog/2024/04/29/powershell-the-object-oriented-shell-you-didnt-know-you-needed/)

PowerShell 是来自微软的交互式 shell 和脚本语言。它是面向对象的 — 这不只是一个流行语，而是与标准 Unix shell 工作方式的显著区别。而且它确实可以作为交互式 shell 使用。

## 入门指南

PowerShell 如此优秀，微软做了两次。

具体来说，目前存在两个名为 PowerShell 的产品：

+   Windows PowerShell (5.1) 是 Windows 的内置组件。它是专有的，仅限于 Windows，并且基于同样专有且仅限于 Windows 的 .NET Framework 4.x。它有一个蓝色图标。

+   PowerShell (7.x)，以前称为 PowerShell Core，是一个独立应用程序。它采用 MIT 许可证 [(在 GitHub 上开发)](https://github.com/PowerShell/PowerShell)，适用于 Windows、Linux 和 macOS，基于同样采用 MIT 许可证且同样跨平台的 .NET（以前是 .NET Core）。它有一个黑色图标。

当 PowerShell (Core) 推出时，Windows PowerShell 开发停止了。它确实缺少一些便利性和命令，但对于尝试或在无法在 Windows 系统上安装 PowerShell 但需要使用代码解决问题时，它仍然是一个很好的选择。

本文中的所有示例应该可以在任何操作系统上的 PowerShell 的任一版本中运行（除非另有说明）。

安装现代 PowerShell：[Windows](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.4)，[Linux](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-linux?view=powershell-7.4)，[macOS](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-linux?view=powershell-7.4)。

## 对象？在我的 shell 中？

让我们尝试获取一个目录列表。这里是微软的地盘，所以让我们尝试 DOS 命令来获取目录列表 — `dir`：

```
PS C:\tmp\hello> dir

    Directory: C:\tmp\hello

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d----          2024-04-29    18:00                world
-a---          2024-04-29    18:00             23 example.py
-a---          2024-04-29    18:00              7 foobar.txt
-a---          2024-04-29    18:00             14 helloworld.txt
-a---          2024-04-29    18:00              0 newfile.txt
-a---          2024-04-29    18:00              5 test.txt

```

这看起来像是一个典型（如果稍微冗长一些）的文件列表。

现在，让我们试着做点有用的事情。让我们获取所有 `.txt` 文件的总大小。

在 Unix shell 中，一种选择是 `du -bc *.txt`。参数：`-b`（`--bytes`）显示实际字节大小，`-c`（`--summarize`）生成总计。结果如下：

```
7  foobar.txt
14  helloworld.txt
0  newfile.txt
5  test.txt
26  total

```

但是如何只获取数字？这需要文本操作（获取最后一行的第一个单词）。类似于`du -bc *.txt | tail -n 1 | cut -f 1` 将会做到这一点。还有`wc --total=only --bytes *.txt` — 但这是GNU wc特有的，因此在BSD或macOS上不适用。另一种选择是解析`ls -l`的输出 — 但这可能并不总是容易，并且输出可能包含特定的`ls`版本或用户特定的shell配置添加的意外内容。

让我们在PowerShell中尝试一些东西。如果我们执行`$x = dir`，我们将在`$x`中得到`dir`命令的输出。让我们尝试进一步分析它，第一个字符是换行符吗？

```
PS C:\tmp\hello> $x = dir
PS C:\tmp\hello> $x[0]

    Directory: C:\tmp\hello

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d----          2024-04-29    18:00                world

```

这很有趣，我们没有得到第一个字符或第一行，我们得到了第一个*文件*。如果我们尝试`$x[1]`呢？

```
PS C:\tmp\hello> $x[1]

    Directory: C:\tmp\hello

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---          2024-04-29    18:00             23 example.py

```

如果我们尝试从中获取`Length`属性呢？

```
PS C:\tmp\hello> $x[1].Length
23

```

原来`dir`返回一个对象数组，而PowerShell知道如何将这个数组（以及数组中的单个项目）格式化为一个漂亮的表格。我们能用它做些什么呢？像这样：

```
PS C:\tmp\hello> Get-ChildItem -Filter '*.txt' |
  ForEach-Object { $_.Length } |
  Measure-Object -Sum

Count             : 4
Average           :
Sum               : 26
Maximum           :
Minimum           :
StandardDeviation :
Property          :

PS C:\tmp\hello> (Get-ChildItem -Filter '*.txt' |
  ForEach-Object { $_.Length } |
  Measure-Object -Sum).Sum
26
PS C:\tmp\hello> (Get-ChildItem -Filter '*.txt' |
  Measure-Object -Sum -Property Length).Sum
26
PS C:\tmp\hello> (Get-ChildItem -Recurse -Filter '*.txt' |
  Measure-Object -Sum -Property Length).Sum
30
PS C:\tmp\hello> $measured = (Get-ChildItem -Recurse -Filter '*.txt' |
  Measure-Object -Sum -Property Length)
PS C:\tmp\hello> $measured.Sum / $measured.Count
6

```

我们可以遍历所有文件对象，获取它们的长度（使用`ForEach-Object`和lambda），然后使用`Measure-Object`来计算总和（`Measure-Object`返回一个对象，我们需要获取其`Sum`属性）。我们可以在`Measure-Object`中用`-Property`参数替换`ForEach-Object`调用。如果我们想要查看子目录，可以轻松地在`Get-ChildItem`中添加`-Recurse`选项。这样我们就能得到可以进行数学计算的实际整数。

你可能已经注意到，我在前面的例子中使用了`Get-ChildItem`而不是`dir`。`Get-ChildItem`是该命令的完整名称（*cmdlet*）。`dir`是其别名之一，还有`gci`和`ls`（仅限Windows，以避免遮蔽`/bin/ls`）。许多常见命令都有定义的别名，以便更易于键入和使用 —— 例如，`Copy-Item`可以写作`cp`（与Unix兼容），`copy`（与MS-DOS兼容）和`ci`。在我们的示例中，我们也可以用`measure`代替`Measure-Object`，用`foreach`或`%`代替`ForEach-Object`。这些别名在交互使用时非常方便，但对于脚本，最好使用全名以提高可读性，并避免依赖于这些别名在不同环境下的变化。

## 更多文件系统操作

### 每个文件夹中的文件数

在一个名为`Photos`的文件夹中有一个照片集合，按文件夹分组。目标是查看每个文件夹中有多少个`.jpg`文件。以下是PowerShell解决方案：

```
PS C:\tmp> Get-ChildItem Photos/*/*.jpg |
  Group-Object { $_.Directory.Name } |
  Sort-Object -Property Count -Descending
Count Name                      Group
----- ----                      -----
   10 foo bar                   {C:\tmp\Photos\foo bar\img001.jpg, C:\tmp\Photos\foo bar\img002.jpg, C:\tmp\Photos\foo bar\img003.jpg…}
    2 example                   {C:\tmp\Photos\example\img101.jpg, C:\tmp\Photos\example\img201.jpg}

```

在Unix领域，[StackOverflow上有很多解决方案](https://stackoverflow.com/questions/15216370/how-to-count-number-of-files-in-each-directory)。顶部的解决方案是 `du -a | cut -d/ -f2 | sort | uniq -c | sort -nr` — 将许多工具组合在一起，以检查磁盘使用情况为起点，然后进行大量的字符串操作。第二个解决方案使用了find、read和shell通配符。PowerShell的解决方案对于曾经接触过SQL的任何人来说都非常简单和明显。

上述示例适用于一级嵌套。对于更多级别，给定 `Photos\one\two\three.jpg`，使用 `Get-ChildItem -Filter '*.jpg' -Recurse Photos`，并：

+   Group by `$_.Directory.Name`（与以前相同）以获得 `two`

+   Group by `Split-Path -Parent ([System.IO.Path]::GetRelativePath("$PWD/Photos", $_.FullName))` 以获得 `one/two`

+   Group by `([System.IO.Path]::GetRelativePath("$PWD/Photos", $_.FullName)).Split([System.IO.Path]::DirectorySeparatorChar)[0]` 以获得 `one`

（所有上述示例也适用于单个文件夹。后两个示例在 Windows PowerShell 上不起作用。）

### 重复查找器

让我们构建一个简单的工具来检测逐字节重复的文件。 `Get-FileHash` 是一个 shell 内置命令。我们可以再次使用 `Group-Object`，并使用 `Where-Object` 来仅过滤匹配的对象。计算每个文件的哈希值相当低效，因此我们首先按文件长度分组，然后确保哈希值匹配。这给我们带来了一个漂亮的管道，总共有 6 个命令：

```
# Fully spelled out
Get-ChildItem -Recurse -File |
  Group-Object { $_.Length } |
  Where-Object { $_.Count -gt 1 } |
  ForEach-Object { $_.Group } |
  Group-Object { (Get-FileHash -Algorithm MD5 $_).Hash } |
  Where-Object { $_.Count -gt 1 }

# Using aliases
gci -Recurse -File |
  group { $_.Length } |
  where { $_.Count -gt 1 } |
  foreach { $_.Group } |
  group { (Get-FileHash -Algorithm MD5 $_).Hash } |
  where { $_.Count -gt 1 }

# Using less readable aliases
gci -Recurse -File |
  group { $_.Length } |
  ? { $_.Count -gt 1 } |
  % { $_.Group } |
  group { (Get-FileHash -Algorithm MD5 $_).Hash } |
  ? { $_.Count -gt 1 }

```

## 严肃的脚本编写：软件材料清单

软件材料清单（SBOMs）和供应链安全是当今非常流行的话题。老板希望有类似的东西，即一个 CSV 文件，其中包含软件包和版本的列表，以及仅有的直接生产依赖项。当然，存在像 SPDX 这样的标准，但老板不喜欢那些讨厌的“标准”。后端是用 C# 编写的，前端是用 Node.js 编写的。由于我们只关心生产依赖项，我们可以查看 `.csproj` 和 `package.json` 文件。对于 Node 包，我们还会尝试从 npm API 获取许可证名称（在这个例子中，NuGet 的 API 比较复杂，所以我们将其保留为 `TODO`）。

```
$ErrorActionPreference = "Stop" # stop execution on any error
Set-StrictMode -Version 3.0

function Get-CsprojPackages([string]$Path) {
  return Select-Xml -Path $Path -XPath '//PackageReference' |
    ForEach-Object {
      [PSCustomObject]@{
        Name = $_.Node.GetAttribute("Include")
        Version = $_.Node.GetAttribute("Version")
        Source = 'nuget'
        License = 'TODO'
      }
    }
}

function Get-NodePackages([string]$Path) {
  $nameToVersion = (Get-Content -Raw $Path | ConvertFrom-Json).dependencies
  return $nameToVersion.psobject.Properties | ForEach-Object {
    [PSCustomObject]@{
      Name = $_.Name
      Version = $_.Value
      Source = 'node'
      License = (Get-NodeLicense -Name $_.Name)
    }
  }
}

function Get-NodeLicense([string]$Name) {
  try {
    return (Invoke-RestMethod -TimeoutSec 3
      "https://registry.npmjs.org/$Name").license
  } catch {
    return "???"
  }
}

$csprojData = @(Get-ChildItem -Recurse -Filter '*.csproj' |
  ForEach-Object { Get-CsprojPackages $_.FullName })

$nodeData = @(Get-ChildItem -Recurse -Filter 'package.json' |
  Where-Object { $_.FullName -notlike '*node_modules*' } |
  ForEach-Object { Get-NodePackages $_.FullName })

$allData = $csProjData + $nodeData
$allData | ConvertTo-Csv -NoTypeInformation | Tee-Object sbom.csv

```

就像每个写得很好的 shell 脚本都以 `set -euo pipefail` 开头一样，每个 PowerShell 脚本都应该以 [`$ErrorActionPreference = "Stop"`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/set-strictmode?view=powershell-7.4) 开头，这样一旦出现问题，执行就会停止。请注意，这不会影响本地命令，您仍然需要检查 `$LASTEXITCODE`。另一个有用的早期命令是 [`Set-StrictMode -Version 3.0`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/set-strictmode?view=powershell-7.4)，用于捕获未定义的变量。

对于 `.csproj` 文件，这些是 XML，我们使用 XPath 查找 `PackageReference` 元素，然后从 `PackageReference` 节点中提取适当的属性，构建出一个 PSCustomObject。

对于 `package.json`，我们读取文件，解析 JSON，并提取 `dependencies` 对象的属性（这是一个将包名称映射到版本的映射）。要获取许可证，我们使用 `Invoke-RestMethod`，它负责为我们解析 JSON。

在脚本的主体部分，我们寻找适当的文件（跳过 `node_modules` 下的内容），并调用我们的解析器函数。获取所有数据后，我们连接这两个数组，转换成 CSV，并使用 `Tee-Object` 将结果输出到文件和标准输出。我们得到了这样的结果：

```
"Name","Version","Source","License"
"AWSSDK.S3","3.7.307.24","nuget","TODO"
"Microsoft.AspNetCore.SpaProxy","7.0.17","nuget","TODO"
"@testing-library/jest-dom","^5.17.0","node","MIT"
"@testing-library/react","^13.4.0","node","MIT"
"@testing-library/user-event","^13.5.0","node","MIT"
"@types/jest","^27.5.2","node","MIT"
"@types/node","^16.18.96","node","MIT"
"@types/react","^18.3.1","node","MIT"
"@types/react-dom","^18.3.0","node","MIT"
"react","^18.3.1","node","MIT"
"react-dom","^18.3.1","node","MIT"
"react-scripts","5.0.1","node","MIT"
"typescript","^4.9.5","node","Apache-2.0"
"web-vitals","^2.1.4","node","Apache-2.0"

```

可以用其他语言完成吗？当然可以，但是 PowerShell 非常容易与 CI 集成，例如 [GitHub Actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#example-running-a-command-using-powershell-core) 或 [Azure Pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/powershell-v2?view=azure-pipelines)。在 Linux 上，你可能会选择使用 Python —— 你同样可以简单地完成任务，只要你不介意使用丑陋的 `urllib.request` 库，或者确保安装了 `requests`（然后你就陷入了 Python 包管理的地狱）。

## 使用 .NET 类

PowerShell 建立在 .NET 之上。这不仅仅是实现技术——PowerShell 提供了访问 .NET 标准库所提供一切的途径。例如，我们上面探讨的在多个子目录中组织照片的替代方法涉及调用 .NET `System.IO.Path` 类的静态方法。

其他 .NET 类型也是可用的。需要一个 HashSet 吗？看这里：

```
PS> $set = New-Object System.Collections.Generic.HashSet[string]
PS> $set.Add("hello")
True
PS> $set.Add("hello")
False
PS> $set.Add("world") | Out-Null
PS> $set.Count
2
PS> $set -contains "hello"
True
PS> $set -contains "world"
False

```

还可以将任何 .NET DLL 加载到 PowerShell 中（只要它与 PowerShell 构建的 .NET 版本兼容），并像在 C# 中那样正常使用它（尽管语法可能略显丑陋）。

## 强大的 Windows 技巧

Microsoft 去年据称终结了 Internet Explorer。尝试启动 `iexplore.exe` 会弹出 Microsoft Edge。但是，你要知道，Internet Explorer 是 Windows 的重要组成部分，至少已经是两十多年了。软件供应商构建的软件依赖于 IE 的存在和展示网页内容的能力。有些供应商使用 web 视图，但有些则更喜欢其他东西：COM。

COM，即组件对象模型，是微软用来实现不同应用程序和/或组件之间互操作的技术。COM 基本上是一种让来自不同供应商、可能用不同语言编写的类相互通信的方法。在底层，COM 是 C++ 的 `vtable` 加上标准的引用计数和类加载/发现机制。.NET Framework 及其后续版本 .NET 一直都包括 COM 互操作性。现代的 WinRT 平台则是 COM 的增强版。

回到 Internet Explorer，它公开了一些 COM 类。它们并没有随 `iexplore.exe` 被移除。这意味着你可以在 PowerShell 中仅用两行代码打开一个常规的 Internet Explorer 窗口：

```
$ie = New-Object -ComObject InternetExplorer.Application
$ie.Visible = $true

```

为什么要这么做呢？`InternetExplorer.Application` 对象允许您控制浏览器，例如，您可以使用 `$ie.Navigate("https://example.com/")` 访问网页。2024年为什么还要启动IE？我不知道，我猜你可以用它来嘲笑那些删除了用户可访问快捷方式的Microsoft开发人员吧？但确实存在一些遗留应用程序期望能够控制IE。

我们已经探索了使用.NET中的类的可能性。.NET带有一个名为Windows Forms的GUI框架，[可以从PowerShell中加载并用于构建GUI。](https://learn.microsoft.com/en-us/powershell/scripting/samples/creating-a-custom-input-box?view=powershell-7.4) 没有表单设计器，因此需要手动定义和定位控件，但实际上是可行的。

PowerShell 还可以执行各种Windows管理任务。它可以管理启动设置、BitLocker、Hyper-V、网络和存储… 例如，要获取磁盘剩余空间的百分比：

```
$c = Get-Volume C
"$(($c.SizeRemaining / $c.Size) * 100)%"

```

## 走出PowerShell的领域

作为一个Shell，PowerShell显然可以启动子进程。与Python等不同，运行子进程就像运行其他任何东西一样简单。如果您需要 `git pull`，只需键入即可。或者您可以让PowerShell与非PowerShell命令交互，读取输出并传递参数：

```
$changes = (git status --porcelain --null)
if ($LASTEXITCODE -eq 128) {
  throw "Not a git repository"
} elseif ($LASTEXITCODE -ne 0) {
  throw "Getting changes from git failed"
}

if ($null -eq $changes) {
  Write-Host "No changes found"
} else {
  $untrackedFiles = @(
    $changes.Split("`0") |
    Where-Object { $_.StartsWith('?? ') } |
    ForEach-Object { $_.Remove(0, 3) }
  )

  # Alternate spelling for regex fans:
  $untrackedFilesForRegexFans = @(
    $changes.Split("`0") |
    Where-Object { $_ -match '^\?\? ' } |
    ForEach-Object { $_ -replace '^\?\? ','' }
  )

  if ($untrackedFiles) {
    Write-Host "Opening $($untrackedFiles.Length) untracked files in VS Code"
    code $untrackedFiles
  } else {
    Write-Host "No untracked files"
  }
}

```

我选择使用标准的.NET字符串操作方法来计算未跟踪的文件，但也有正则表达式选项。另外，还有三个内容检查运算符：`-match` 使用正则表达式，`-like` 使用[通配符](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_wildcards?view=powershell-7.4)，而 `-contains` 用于检查集合成员。

## 配置脚本

我使用一个相当小的配置脚本，其中添加了一些我在Unix中习惯的行为，以及使Tab完成显示菜单的功能。以下是最基本的部分：

```
Set-PSReadLineOption -HistorySearchCursorMovesToEnd
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward
Set-PSReadlineKeyHandler -Key ctrl+d -Function DeleteCharOrExit
Set-PSReadlineKeyHandler -Key Tab -Function MenuComplete
Set-PSReadLineOption -AddToHistoryHandler {
  param($command)
  # Commands starting with space are not remembered.
  return -not ($command -like ' *')
}

```

除此之外，我使用了一些别名，并借助[oh-my-posh](https://ohmyposh.dev/)实现了一个漂亮的提示符。

## 那些不寻常且有时令人困惑的部分

PowerShell 可能会有些啰嗦。与其他语言相比，它的一些语法有点古怪，例如相等和逻辑运算符（例如，`-eq`，`-le`，`-and`）。别名通常有助于记住命令，但不能总是依赖于它们 — `ls` 仅在Windows上被定义为别名，而Windows PowerShell 将 `wget` 和 `curl` 别名为 `Invoke-WebRequest`，尽管这三者的命令行参数和输出完全不同（这在PowerShell中已经删除）。

此外，Unix/DOS别名不会改变参数处理方式。`rm -rf foo` 是无效的。`rm -r foo` 是有效的，因为参数名称可以缩写，只要缩写是明确的。`rm -r -f foo` 不是有效的，因为 `-f` 可能是 `-Filter` 或 `-Force` 的缩写（所以 `rm -r -fo foo` 将起作用）。`rm foo bar` 不起作用，需要一个数组：`rm foo,bar`。

`C:\Windows\regedit.exe` 启动注册表编辑器。`"C:\Program Files\Mozilla Firefox\firefox.exe"` 是一个字符串。启动带有空格名称的内容需要使用调用运算符：`& "C:\Program Files\Mozilla Firefox\firefox.exe"`。如果需要，PowerShell 的制表符补全将添加 `&`。

有两种函数调用语法。调用函数/cmdlet 使用带有参数名的 Shell 风格语法：`Some-Function -Arg1 value1 -Arg2 value2`，参数名可以缩写，并且有时可以省略。调用方法需要更传统的语法：`$obj.SomeMethod(value1, value2)`。无论哪种情况，名称都是不区分大小写的。

转义字符是反引号。在Windows中，反斜杠是路径分隔符，因此将其作为转义字符会使Windows上的所有事情变得困难。至少它让编写正则表达式变得容易。

## PowerShell 最丑陋的部分

PowerShell 最丑陋且最不直观的部分是处理单元素数组。PowerShell *确实* 希望将它们解包成标量。命令 `(Get-ChildItem).Length` 将生成当前目录中文件的数量 — *除非* 恰好有一个文件，在这种情况下，它将生成单个文件的字节大小。如果没有任何项，PowerShell 不会产生空数组，而是产生 `$null`。有时，最终事情会解决（因为许多 cmdlet 都乐意接受任何输入），但有时，必须要求 PowerShell 停止这种疯狂并返回一个数组：`@(Get-ChildItem).Length`。

前面带有 `git status` 的示例利用其 `--null` 参数获取零分隔数据，因此我们根据规则期望要么是 `$null` 要么是单个字符串。如果我们不想使用 `--null`，我们将需要使用 `@(git status --porcelain)` 来始终获取一个数组（但我们还需要删除 `git` 添加到包含空格路径的引号）。

## 结论

PowerShell 是一个很好的交互式 Shell 和脚本语言。虽然它有一些缺陷，但比起通常的 Unix Shell，它更强大，并且它的强类型、面向对象的代码击败了 *强类型* 的 `sh` 混乱。
