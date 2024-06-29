<!--yml

category: 未分类

date: 2024-05-29 12:29:58

-->

# 在 Y 分钟内尝试 ripgrep

> 来源：[https://codapi.org/try/ripgrep/](https://codapi.org/try/ripgrep/)

[ripgrep](https://github.com/BurntSushi/ripgrep) 是一个命令行工具，可以搜索你的文件，查找你给定的模式。它类似于 grep，但提供更好的用户体验，并且（通常）更快。

[基础知识](#basics) · [递归搜索](#recursive-search) · [有用的选项](#useful-options) · [替换](#replacements) · [配置](#configuration) · [进一步阅读](#further-reading)

✨ 这是一个开源指南。随时[改进它](https://github.com/nalgeon/tryxinyminutes/blob/main/try/ripgrep/index.md)！

## 基础知识

ripgrep 表现得就像它逐行读取每个文件一样：

+   如果一行与提供给 ripgrep 的模式匹配，则打印该行。

+   如果一行不匹配模式，则不打印该行。

最好的方法是通过一个例子来看看这是如何工作的。

我们将尝试搜索[httpurr](https://github.com/rednafi/httpurr)源代码，我已经下载到 `/opt/httpurr` 目录中，像这样：

```
cd /opt curl -OL https://github.com/rednafi/httpurr/archive/refs/tags/v0.1.2.tar.gz tar xvzf v0.1.2.tar.gz mv httpurr-0.1.2 httpurr cd httpurr 
```

[在文件中搜索](#search-in-file) · [部分匹配](#partial-matches) · [正则表达式](#regular-expressions) · [固定字符串](#fixed-strings) · [多个模式](#multiple-patterns)

### 在文件中搜索

让我们在 `README.md` 中找到所有 `codes` 一词的出现：

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
3:    <strong><i> >> HTTP status codes on speed dial << </i></strong>
30:* List the HTTP status codes:
54:* Filter the status codes by categories:
124:		  Print HTTP status codes by category with --list;
131:		  Print HTTP status codes 
```

那么这里发生了什么？ripgrep 读取了 `README.md` 的内容，并对每一行包含 `codes` 的行进行了打印。

ripgrep 默认包含每行的行号（使用 `-n`/`--line-number` 强制此行为）并高亮匹配项（使用 `--color=always` 强制此行为）。

### 部分匹配

ripgrep 默认支持部分匹配：

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
81:* Display the description of a status code:
127:		  Print the description of an HTTP status code 
```

单词 `description` 匹配 `descr` 搜索模式。

若要搜索整个单词而非部分匹配，使用 `-w`（`--word-regexp`）选项：

```
rg --word-regexp code README.md 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
81:* Display the description of a status code:
84:	httpurr --code 410
94:	The HyperText Transfer Protocol (HTTP) 410 Gone client error response code
99:	code should be used instead.
126:	  -c, --code [status code]
127:		  Print the description of an HTTP status code 
```

ripgrep 发现包含单词 `code` 的字符串，但没有 `codes`。尝试移除 `--word-regexp` 看看结果如何变化。

### 正则表达式

默认情况下，ripgrep 将搜索模式视为*正则表达式*。让我们找到所有包含 `res` 以及后跟其他字母的单词的行：

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
94:	The HyperText Transfer Protocol (HTTP) 410 Gone client error response code
95:	indicates that access to the target resource is no longer available at the
152:of the rest. 
```

`\w+` 表示“一个或多个类似单词的字符”（例如字母如 `p` 或 `o`，但不包括标点符号如 `.` 或 `!`），因此 `response`、`resource` 和 `rest` 都匹配。

假设我们只对以 `res` 开头的 4 个字母单词感兴趣：

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
152:of the rest. 
```

`\b`意味着“单词边界”（例如空格、标点符号或行尾），因此`rest`匹配，但`response`和`resource`不匹配。

最后，让我们搜索3位数（显示前10个匹配项使用`head`）：

```
rg '\d\d\d' README.md | head 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
45:	100    Continue
46:	101    Switching Protocols
47:	102    Processing
48:	103    Early Hints
69:	200    OK
70:	201    Created
71:	202    Accepted
72:	203    Non-Authoritative Information
73:	204    No Content
74:	205    Reset Content 
```

关于正则表达式的完整教程超出了本指南的范围，但ripgrep的特定语法在[regex](https://docs.rs/regex/*/regex/#syntax)包中有文档记录。

### 固定字符串

如果我们想要搜索一个*字面*字符串而不是正则表达式？假设我们对一个后跟点的单词`code`感兴趣：

```
rg 'code.' src/data.go | head 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
8:The HTTP 100 Continue informational status response code indicates that
14:status code in response before sending the body.
31:The HTTP 101 Switching Protocols response code indicates a protocol to which the
53:Deprecated: This status code is deprecated. When used, clients may still accept
56:The HTTP 102 Processing informational status response code indicates to client
59:This status code is only sent if the server expects the request to take
112:The HTTP 200 OK success status response code indicates that the request has
141:The HTTP 201 Created success status response code indicates that the request has
149:The common use case of this status code is as the result of a POST request.
165:The HTTP 202 Accepted response status code indicates that the request has been 
```

由于`.`在正则表达式中表示“任何字符”，我们的模式也匹配`code`、`codes`及其他我们不感兴趣的情况。

要将模式视为字面字符串，请使用`-F`（`--fixed-strings`）选项：

```
rg --fixed-strings 'code.' src/data.go 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
197:to responses with any status code.
283:Browsers accessing web pages will never encounter this status code.
695:to an error code.
1027:erroneous cases it happens, they will handle it as a generic 400 status code.
1051:Regular web servers will normally not return this status code. But some other
1418:then the server responds with the 510 status code. 
```

好多了！

### 多个模式

要搜索多个模式，请使用`-e`（`--regexp`）选项列出它们。ripgrep将输出至少匹配指定模式之一的行。

例如，搜索`make`或`run`：

```
rg -e 'make' -e 'run' README.md 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
139:* Go to the root directory and run:
141:	make init
145:	make lint
149:	make test 
```

如果有许多模式，将它们放在文件中并使用`-f`（`--file`）选项指向ripgrep可能更容易：

```
echo 'install' > /tmp/patterns.txt echo 'make' >> /tmp/patterns.txt echo 'run' >> /tmp/patterns.txt   rg --file=/tmp/patterns.txt README.md 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
13:* On MacOS, brew install:
17:	    && brew install httpurr
20:* Or elsewhere, go install:
23:	go install github.com/rednafi/httpurr/cmd/httpurr
139:* Go to the root directory and run:
141:	make init
145:	make lint
149:	make test 
```

## 递归搜索

之前，我们使用ripgrep在单个文件中搜索，但ripgrep可以完全递归搜索目录。

[目录中搜索](#search-in-directory) · [自动过滤](#automatic-filtering) · [文件通配符](#file-globs) · [文件类型](#file-types)

### 在目录中搜索

让我们找到所有未导出的函数（它们以小写字母开头）：

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
./cmd/httpurr/main.go:12:func main() {
./src/cli.go:16:func formatStatusText(text string) string {
./src/cli.go:21:func printHeader(w *tabwriter.Writer) {
./src/cli.go:35:func printStatusCodes(w *tabwriter.Writer, category string) error {
./src/cli.go:105:func printStatusText(w *tabwriter.Writer, code string) error { 
```

此搜索返回了来自`cmd`和`src`目录的匹配项。如果只对`cmd`感兴趣，请指定它而不是`.`：

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
cmd/httpurr/main.go:12:func main() { 
```

要搜索多个目录，请像这样列出它们：

```
rg 'func [a-z]\w+' cmd src 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
cmd/httpurr/main.go:12:func main() {
src/cli.go:16:func formatStatusText(text string) string {
src/cli.go:21:func printHeader(w *tabwriter.Writer) {
src/cli.go:35:func printStatusCodes(w *tabwriter.Writer, category string) error {
src/cli.go:105:func printStatusText(w *tabwriter.Writer, code string) error { 
```

### 自动过滤

ripgrep足够聪明，能够在搜索时忽略一些路径：

+   `.gitignore`中的模式，

+   隐藏文件和目录，

+   二进制文件，

+   符号链接。

例如，让我们搜索GitHub操作任务：

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
exit status 1 
```

因为GitHub的东西在一个隐藏的`.github`目录中，ripgrep找不到它。但是通过`-.` (`--hidden`)选项可以找到它：

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
./.github/workflows/automerge.yml:8:jobs:
./.github/workflows/lint.yml:11:jobs:
./.github/workflows/release.yml:10:jobs:
./.github/workflows/test.yml:11:jobs: 
```

类似地，还有选项可以启用其他路径：

+   `--no-ignore`来搜索`.gitignore`中的模式；

+   `-a` (`--text`)来搜索二进制文件；

+   `-L` (`--follow`)用于跟随符号链接。

ripgrep允许你用`.ignore`文件覆盖`.gitignore`中忽略的路径。详细信息请参见[官方指南](https://github.com/BurntSushi/ripgrep/blob/master/GUIDE.md#automatic-filtering)。

### 文件glob

让我们搜索`httpurr`：

```
rg --max-count=5 httpurr . 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
./README.md:2:    <h1>ᗢ httpurr</h1>
./README.md:16:	brew tap rednafi/httpurr https://github.com/rednafi/httpurr \
./README.md:17:	    && brew install httpurr
./README.md:23:	go install github.com/rednafi/httpurr/cmd/httpurr
./README.md:33:	httpurr --list
./cmd/httpurr/main.go:4:	"github.com/rednafi/httpurr/src"
./go.mod:1:module github.com/rednafi/httpurr
./httpurr.rb:7:  homepage "https://github.com/rednafi/httpurr"
./httpurr.rb:12:      url "https://github.com/rednafi/httpurr/releases/download/v0.1.1/httpurr_Darwin_x86_64.tar.gz"
./httpurr.rb:16:        bin.install "httpurr"
./httpurr.rb:20:      url "https://github.com/rednafi/httpurr/releases/download/v0.1.1/httpurr_Darwin_arm64.tar.gz"
./httpurr.rb:24:        bin.install "httpurr"
./src/cli.go:24:	fmt.Fprintf(w, "\nᗢ httpurr\n")
./src/cli_test.go:64:	want := "\nᗢ httpurr\n==========\n\n" 
```

请注意，我通过`-m` (`--max-count`)选项将每个文件的结果数限制为5，以保持结果的可读性（如果有许多匹配项）。

结果相当多。让我们通过仅在`.go`文件中搜索来缩小范围：

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
./cmd/httpurr/main.go:4:	"github.com/rednafi/httpurr/src"
./src/cli.go:24:	fmt.Fprintf(w, "\nᗢ httpurr\n")
./src/cli_test.go:64:	want := "\nᗢ httpurr\n==========\n\n" 
```

`-g` (`--glob`)选项接受一个*glob*（文件名模式），通常包含固定部分（在我们的例子中是`.go`）和通配符`*`（“除了路径分隔符外的任何内容”）。

另一个例子 — 搜索名为`http`的文件：

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
./httpurr.rb:7:  homepage "https://github.com/rednafi/httpurr"
./httpurr.rb:12:      url "https://github.com/rednafi/httpurr/releases/download/v0.1.1/httpurr_Darwin_x86_64.tar.gz"
./httpurr.rb:16:        bin.install "httpurr"
./httpurr.rb:20:      url "https://github.com/rednafi/httpurr/releases/download/v0.1.1/httpurr_Darwin_arm64.tar.gz"
./httpurr.rb:24:        bin.install "httpurr"
./httpurr.rb:31:      url "https://github.com/rednafi/httpurr/releases/download/v0.1.1/httpurr_Linux_arm64.tar.gz"
./httpurr.rb:35:        bin.install "httpurr"
./httpurr.rb:39:      url "https://github.com/rednafi/httpurr/releases/download/v0.1.1/httpurr_Linux_x86_64.tar.gz"
./httpurr.rb:43:        bin.install "httpurr" 
```

要*取反*glob，用`!`前缀。例如，除了`.go`、`.md`和`.rb`文件外的所有地方搜索：

```
rg -g '!*.{go,md,rb}' httpurr . 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
./go.mod:1:module github.com/rednafi/httpurr 
```

剩下的就是`go.mod`。

要应用多个过滤器，请指定多个glob选项。例如，查找所有函数，除了测试文件中的函数：

```
rg -g '*.go' -g '!*_test.go' 'func ' . 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
./cmd/httpurr/main.go:12:func main() {
./src/cli.go:16:func formatStatusText(text string) string {
./src/cli.go:21:func printHeader(w *tabwriter.Writer) {
./src/cli.go:35:func printStatusCodes(w *tabwriter.Writer, category string) error {
./src/cli.go:105:func printStatusText(w *tabwriter.Writer, code string) error {
./src/cli.go:123:func Cli(w *tabwriter.Writer, version string, exitFunc func(int)) { 
```

### 文件类型

而不是使用glob按扩展名过滤，你可以使用ripgrep对文件类型的支持。让我们在`.go`文件中搜索`httpurr`：

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
./cmd/httpurr/main.go:4:	"github.com/rednafi/httpurr/src"
./src/cli.go:24:	fmt.Fprintf(w, "\nᗢ httpurr\n")
./src/cli_test.go:64:	want := "\nᗢ httpurr\n==========\n\n" 
```

`-t` (`--type`)选项将搜索结果限制为特定类型的文件（在我们的例子中是Go源文件）。

要排除特定类型的文件，请使用`-T` (`--not-type`)选项：

```
rg -T go -T md -T ruby httpurr . 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
./go.mod:1:module github.com/rednafi/httpurr 
```

我们已经排除了Go、Markdown和Ruby文件，所以剩下的只有`go.mod`（就我个人而言，我也认为它是Go文件，但ripgrep不这么认为）。

## 有用的选项

ripgrep支持多种额外的搜索和输出选项，这些选项可能对你很有用。

[忽略大小写](#ignore-case) · [反向匹配](#inverse-matching) · [计数匹配项](#count-matches) · [仅显示匹配项](#show-matches-only) · [仅显示文件](#show-files-only) · [显示上下文](#show-context) · [多行搜索](#multiline-search)

### 忽略大小写

还记得我们在README中搜索`codes`的情况吗？

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
3:    <strong><i> >> HTTP status codes on speed dial << </i></strong>
30:* List the HTTP status codes:
54:* Filter the status codes by categories:
124:		  Print HTTP status codes by category with --list;
131:		  Print HTTP status codes 
```

它返回`codes`的匹配项，但不返回`Codes`，因为ripgrep默认区分大小写。要更改此设置，请使用`-i`（`--ignore-case`）：

```
rg --ignore-case codes README.md 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
3:    <strong><i> >> HTTP status codes on speed dial << </i></strong>
30:* List the HTTP status codes:
40:	Status Codes
54:* Filter the status codes by categories:
64:	Status Codes
124:		  Print HTTP status codes by category with --list;
131:		  Print HTTP status codes 
```

也可以使用`-S`（`--smart-case`），它的行为类似于`--ignore-case`，除非搜索模式全为大写：

```
rg --smart-case HTTP README.md 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
3:    <strong><i> >> HTTP status codes on speed dial << </i></strong>
30:* List the HTTP status codes:
94:	The HyperText Transfer Protocol (HTTP) 410 Gone client error response code
109:	https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/410
124:		  Print HTTP status codes by category with --list;
127:		  Print the description of an HTTP status code
131:		  Print HTTP status codes 
```

搜索`HTTP`会匹配`HTTP`，但不会匹配`https`或`httpurr`。

### 反向匹配

要查找*不包含*模式的行，请使用`-v`（`--invert-match`）。例如，查找不含`@`符号的非空行：

```
rg --invert-match -e '@' -e '^/codapi-snippet> Makefile 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
1:.PHONY: lint
2:lint:
8:.PHONY: lint-check
9:lint-check:
14:.PHONY: test
15:test:
20:.PHONY: clean
21:clean:
27:.PHONY: init
28:init: 
```

### 计数匹配项

要计算每个`.go`文件中匹配行的数量，请使用`-c`（`--count`）。例如，计算每个`.go`文件中的函数数量：

```
rg --count -t go 'func ' . 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
./cmd/httpurr/main.go:1
./src/cli.go:5
./src/cli_test.go:10
./src/data_test.go:2 
```

请注意，`--count`计算的是*行数*，而不是匹配数。例如，`src/cli.go`中有6个`string`单词，但其中两个在同一行，所以`--count`报告5：

```
rg -w --count 'string' src/cli.go 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
5 
```

要计算*匹配项*的数量，请使用`--count-matches`：

```
rg -w --count-matches 'string' src/cli.go 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
6 
```

### 仅显示匹配项

默认情况下，ripgrep会打印包含匹配项的整行内容。要仅显示匹配的部分，请使用`-o`（`--only-matching`）选项。

假设我们要查找名称为`print`-something的函数：

```
rg --only-matching -t go 'func print\w+' . 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
./src/cli.go:21:func printHeader
./src/cli.go:35:func printStatusCodes
./src/cli.go:105:func printStatusText 
```

结果要比没有使用`--only-matching`（尝试在上述命令中删除该选项并自行查看）时更清晰。

### 仅显示文件

如果匹配项过多，可能更喜欢仅显示出现匹配项的文件。使用`-l`（`--files-with-matches`）即可实现此功能：

```
rg --files-with-matches 'httpurr' . 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
./README.md
./cmd/httpurr/main.go
./go.mod
./httpurr.rb
./src/cli.go
./src/cli_test.go 
```

### 显示上下文

还记得我们搜索GitHub职位时的情景吗？

```
rg 'jobs:' .github/workflows 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
.github/workflows/automerge.yml:8:jobs:
.github/workflows/lint.yml:11:jobs:
.github/workflows/release.yml:10:jobs:
.github/workflows/test.yml:11:jobs: 
```

这些结果有点无用，因为它们不返回实际的工作名称（该名称位于 `jobs` 后的下一行）。为了解决这个问题，让我们使用 `-C` (`--context`)，显示每个匹配周围的 `N` 行：

```
rg --context=1 'jobs:' .github/workflows 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
.github/workflows/automerge.yml-7-
.github/workflows/automerge.yml:8:jobs:
.github/workflows/automerge.yml-9-  dependabot:
--
.github/workflows/lint.yml-10-
.github/workflows/lint.yml:11:jobs:
.github/workflows/lint.yml-12-  golangci:
--
.github/workflows/release.yml-9-
.github/workflows/release.yml:10:jobs:
.github/workflows/release.yml-11-  goreleaser:
--
.github/workflows/test.yml-10-
.github/workflows/test.yml:11:jobs:
.github/workflows/test.yml-12-  test: 
```

也许只显示匹配后的 *下一行* 会更好，因为我们不关心前一行。使用 `-A` (`--after-context`) 来实现这一点：

```
rg --after-context=1 'jobs:' .github/workflows 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
.github/workflows/automerge.yml:8:jobs:
.github/workflows/automerge.yml-9-  dependabot:
--
.github/workflows/lint.yml:11:jobs:
.github/workflows/lint.yml-12-  golangci:
--
.github/workflows/release.yml:10:jobs:
.github/workflows/release.yml-11-  goreleaser:
--
.github/workflows/test.yml:11:jobs:
.github/workflows/test.yml-12-  test: 
```

不错！

### 多行搜索

我有另一个搜索 GitHub 职位的想法。由于工作名称总是在字面上 `jobs:` 后的下一行，让我们启用带有 `-U` (`--multiline`) 的多行搜索：

```
rg --multiline 'jobs:\n\s+\w+' .github/workflows 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
.github/workflows/automerge.yml:8:jobs:
.github/workflows/automerge.yml:9:  dependabot:
.github/workflows/lint.yml:11:jobs:
.github/workflows/lint.yml:12:  golangci:
.github/workflows/release.yml:10:jobs:
.github/workflows/release.yml:11:  goreleaser:
.github/workflows/test.yml:11:jobs:
.github/workflows/test.yml:12:  test: 
```

现在即使不使用 `--context`，我们也可以看到工作名称。

## 替换

ripgrep 提供了有限的能力来用其他文本替换匹配的文本。

[替换匹配项](#replace-matches) · [替换整行](#replace-entire-line) · [替换分组](#replace-groups)

### 替换匹配项

还记得我们在 README 中搜索 `codes` 的例子吗？

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
3:    <strong><i> >> HTTP status codes on speed dial << </i></strong>
30:* List the HTTP status codes:
54:* Filter the status codes by categories:
124:		  Print HTTP status codes by category with --list;
131:		  Print HTTP status codes 
```

现在让我们使用 `-r` (`--replace`) 选项将所有 `codes` 替换为 `ids`：

```
rg codes -r ids README.md 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
3:    <strong><i> >> HTTP status ids on speed dial << </i></strong>
30:* List the HTTP status ids:
54:* Filter the status ids by categories:
124:		  Print HTTP status ids by category with --list;
131:		  Print HTTP status ids 
```

### 替换整行

替换仅适用于匹配文本的部分。要替换整行文本，包括整行在内，像这样匹配：

```
rg '^.*codes.*/codapi-snippet> -r 'REDACTED' README.md 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
3:REDACTED
30:REDACTED
54:REDACTED
124:REDACTED
131:REDACTED 
```

或者，您可以结合 `-o` (`--only-matching`) 选项和 `--replace` 来达到相同的结果：

```
rg codes -or 'REDACTED' README.md 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
3:REDACTED
30:REDACTED
54:REDACTED
124:REDACTED
131:REDACTED 
```

### 替换分组

替换可以包括捕获组。假设我们想要找到所有 `status` 后面跟着另一个单词的出现，并用破折号将它们连接起来。我们可能使用的模式是 `status\s+(\w+)`：

+   字面字符串 `status`，

+   后跟任意数量的空白字符，

+   后跟任意数量的 "word" 字符（例如字母）。

我们将 `\w+` 放在 "捕获组" 中（由括号表示），以便稍后在我们的替换字符串中引用它。例如：

```
rg 'status\s+(\w+)' -r 'status-$1' README.md 
```

<codapi-snippet sandbox="shell" command="aha" editor="basic" template="httpurr.sh" output-mode="html" output="">```
3:    <strong><i> >> HTTP status-codes on speed dial << </i></strong>
30:* List the HTTP status-codes:
54:* Filter the status-codes by categories:
81:* Display the description of a status-code:
124:		  Print HTTP status-codes by category with --list;
126:	  -c, --code [status-code]
127:		  Print the description of an HTTP status-code
131:		  Print HTTP status-codes 
```

我们的替换字符串 (`status-$1`) 包括字面上的 `status-`，后跟捕获组索引 `1` 的内容。

> 捕获组实际上从索引 `0` 开始，但第 `0` 个捕获组始终对应于整个匹配。索引 `1` 处的捕获组始终对应于正则表达式模式中找到的第一个显式捕获组。

ripgrep **永远不会修改您的文件**。`--replace` 标志仅控制 ripgrep 的输出。（并且没有标志可以让您在文件中进行替换。）

## 配置

ripgrep 默认具有合理的默认设置，但您可以通过配置文件进行更改。

ripgrep 不会自动在预定义目录中查找配置文件。要使用配置文件，请将 `RIPGREP_CONFIG_PATH` 环境变量设置为其路径。

这是一个配置文件的示例：

<codapi-snippet sandbox="shell" editor="basic" template="config.sh" output="">```
# Trim really long lines and show a preview
--max-columns=40
--max-columns-preview

# Search hidden files / directories (e.g. dotfiles)
--hidden

# Do not search git files
--glob
!.git/*

# Ignore case unless all caps
--smart-case 
```

当指定一个带有值的标志时，可以将标志和值放在同一行，并用 `=` 符号连接（例如 `--max-columns=40`），或者将标志和值放在两行（例如 `--glob`）。不要在同一行上放置它们而不带等号（例如不要 `--max-columns 40`）。

让我们使用这个配置进行搜索：

```
export RIPGREP_CONFIG_PATH=/tmp/.ripgreprc rg httpurr . | head 
```

<codapi-snippet sandbox="shell" editor="basic" template="config.sh" output="">```
./.goreleaser.yml:14:    main: ./cmd/httpurr
./.goreleaser.yml:45:  - name: httpurr
./.goreleaser.yml:46:    homepage: "https://github.com/rednaf [... omitted end of long line]
./.goreleaser.yml:50:      name: httpurr
./README.md:2:    <h1>ᗢ httpurr</h1>
./README.md:16:	brew tap rednafi/httpurr https://github [... omitted end of long line]
./README.md:17:	    && brew install httpurr
./README.md:23:	go install github.com/rednafi/httpurr/c [... omitted end of long line]
./README.md:33:	httpurr --list
./README.md:37:	ᗢ httpurr 
```

## 进一步阅读

ripgrep 支持更多功能，例如显式处理编码或搜索二进制数据。有关详细信息，请参阅[官方指南](https://github.com/BurntSushi/ripgrep/blob/master/GUIDE.md)。

使用 `rg --help` 查看所有支持的选项（我们在本指南中仅覆盖了不到一半）。

[Andrew Gallant](https://blog.burntsushi.net) + [1 others](#contributors) · [original](https://github.com/BurntSushi/ripgrep/blob/master/GUIDE.md) · CC-BY-SA-4.0 · 2024-03-19

[Andrew Gallant](https://blog.burntsushi.net)，[Anton Zhiyanov](https://antonz.org)</codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet>
