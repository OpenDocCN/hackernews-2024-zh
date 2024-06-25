<!--yml

category: 未分类

date: 2024-05-27 14:33:32

-->

# 在浏览器中的代码游乐场

> 来源：[https://antonz.org/in-browser-code-playgrounds/](https://antonz.org/in-browser-code-playgrounds/)

# 在浏览器中的代码游乐场

我是各种技术写作的交互式代码片段的忠实粉丝，从产品文档到在线课程再到博客文章都是如此。就像这样一个：

```
def greet(name):  print(f"Hello, {name}!")   greet("World") 
```

<codapi-snippet sandbox="python" editor="basic">事实上，我甚至建立了一个名为 Codapi ^(用于嵌入此类片段的开源工具。)

通常，代码游乐场由客户端小部件和执行代码并返回结果的服务器端部分组成：

```
 browser
┌───────────────────────────────┐
│ def greet(name):              │
│   print(f"Hello, {name}!")    │
│                               │
│ greet("World")                │
└───────────────────────────────┘
  Run ►
    ↓
  server
┌───────────────────────────────┐
│ docker run codapi/python      │
│ python main.py                │
└───────────────────────────────┘
    ↓
  browser
┌───────────────────────────────┐
│ Hello, World!                 │
└───────────────────────────────┘ 
```

就我个人而言，我对这个设置相当满意。但是，人们通常更喜欢不依赖服务器并完全在浏览器中运行代码。因此，我决定研究一下，并为 JavaScript、Python、PHP、Ruby、Lua 和 SQLite 实现了可嵌入的浏览器中的代码游乐场。

## 在浏览器中运行语言运行时

在浏览器中运行任意程序的现代方法似乎是 WebAssembly 系统接口（WASI^() ——一种基于 WebAssembly 的可执行二进制格式。使用 WASI，你将一个程序（最初用 C、Rust、Go 或其他语言编写）编译成 WASI 二进制文件，然后使用 WASI 运行时（来自不同供应商的许多运行时）运行它。

正如我们可以将任意程序编译为 WASI 二进制文件一样，我们也可以将像 Lua 或 CPython 这样的语言解释器编译为 WASI，并使用 WASI 运行时运行它，以执行 Lua 或 Python 代码。然而，在实践中，情况并非如此简单，因为 WASI 编译器尚未（尚）实现传统编译器（如 GCC）的所有功能。

幸运的是，VMWare Labs 已经完成了艰苦的工作，并将 PHP^(，Python ^(和 Ruby ^(编译成了 WASI。所以我所要做的就是将 WASI 二进制文件发布为 NPM 包，以便在 CDN 上提供它们。我还编译了 Lua ^(和 SQLite ^(到 WASI。)))))

> 还有 Tokunaga Kohei 的 container2wasm 项目^(，将任意 Docker 映像转换为 WASI 二进制文件。这看起来很有前途，但即使是最小的基于 Alpine 的映像也会生成 100+ MB 的二进制文件。而且，由于下载数百兆字节的内容仅仅是为了阅读交互式文章可能并不是一个好主意，所以这种方法并不是很实用（尚）。)

编译为 WASI 的语言运行时是等式的一部分。另一个部分是 WASI 运行时（运行二进制文件的东西），它能够在浏览器中工作。我选择了由本·泰勒（Ben Taylor）制作的 Runno ^(运行时，因为它简单轻量（27 KB）。)

最后一步是修改 JavaScript 小部件 ^(以支持可插拔引擎（WASI 是其中之一）))

就是这样！

## 陈列

这里有一些按照上述描述实现的交互式代码片段。请注意，单击运行按钮时将下载语言运行时，因此第一次运行可能需要一些时间。后续运行几乎是即时的。

### Python

使用 Python 3.12 WASI 运行时执行代码（26.3 MB）。

```
def greet(name):  print(f"Hello, {name}!")   greet("World") 
```

<codapi-snippet engine="wasi" sandbox="python" editor="basic">### PHP

使用 PHP 8.2 WASI 运行时（13.2 MB）执行代码。

```
function greet($name) {  echo "Hello, $name!"; }   greet("World"); 
```

<codapi-snippet engine="wasi" sandbox="php" editor="basic" template="main.php">### Ruby

使用 Ruby 3.2 WASI 运行时（24.5 MB）执行代码。

```
def greet(name)  puts "Hello, #{name}!" end   greet("World") 
```

<codapi-snippet engine="wasi" sandbox="ruby" editor="basic">### Lua

使用 Lua 5.4 WASI 运行时（330 KB）执行代码。

```
function greet(name)  print("Hello, " .. name .. "!") end   greet("World") 
```

<codapi-snippet engine="wasi" sandbox="lua" editor="basic">### JavaScript

使用 AsyncFunction^ 执行代码。

```
const greet = (name) => {  console.log(`Hello, ${name}!`); };   greet("World"); 
```

<codapi-snippet engine="browser" sandbox="javascript" editor="basic">### Fetch

使用 Fetch API^ 执行代码。

```
POST https://httpbingo.org/dump/request content-type: application/json   { "message": "hello" } 
```

<codapi-snippet engine="browser" sandbox="fetch" editor="basic">### SQLite

使用 SQLite 3.44 WASI 运行时（2.1 MB）执行代码。

```
select id, name, department from employees order by id limit 3; 
```

<codapi-snippet engine="wasi" sandbox="sqlite" editor="basic" template="employees.sql">## 高级功能

因为 WASI 运行时插入了现有的架构，所以由 WASI 动力的代码片段支持高级的 Codapi 功能，例如模板或代码单元。

模板^(允许您将一些代码隐藏在幕后，仅显示相关部分。 例如，在上面的 SQLite 示例中，`employees` 表作为模板的一部分创建，因此片段可以认为它是理所当然的：)

```
select id, name, department from employees order by id limit 3; 
```

<codapi-snippet engine="wasi" sandbox="sqlite" editor="basic" template="employees.sql">代码单元^(允许您使代码片段相互依赖。 例如，第一个片段定义了 `wrap` 函数，而第二个片段使用了它:)

```
import textwrap   def wrap(text, width=20):  """Wraps the text so every line is at most width characters long.""" return textwrap.fill(text, width) 
```

<codapi-snippet id="cell-1" engine="wasi" sandbox="python" editor="basic">```
text = (  "Python is a programming language that lets you work quickly " "and integrate systems more effectively." ) print(wrap(text)) 
```

<codapi-snippet id="cell-2" engine="wasi" sandbox="python" editor="basic" depends-on="cell-1">## 用法

要使用原生浏览器游乐场（例如 JavaScript 或 Fetch），请包含 `snippet.js` 脚本并在静态代码示例旁边添加 `codapi-snippet` 元素。 使用 `browser` 引擎：

```
<pre>console.log("hello")</pre>   <codapi-snippet engine="browser" sandbox="javascript" editor="basic"> </codapi-snippet>   <script src="https://unpkg.com/@antonz/codapi@0.12.0/dist/snippet.js"></script> 
```

要使用 WASI 动力的游乐场（例如 Python 或 SQLite），请包含两个额外的脚本并使用 `wasi` 引擎：

```
<pre>print("hello")</pre>   <codapi-snippet engine="wasi" sandbox="python" editor="basic"></codapi-snippet>   <script src="https://unpkg.com/@antonz/runno@0.6.1/dist/runno.js"></script> <script src="https://unpkg.com/@antonz/codapi@0.12.0/dist/engine/wasi.js"></script> <script src="https://unpkg.com/@antonz/codapi@0.12.0/dist/snippet.js"></script> 
```

要从浏览器端切换到服务器端游乐场（可以运行几乎任何软件），请移除 `engine` 属性：

```
<pre> fn main() {  println!("Hello, World!"); } </pre>   <codapi-snippet sandbox="rust" editor="basic"></codapi-snippet> 
```

请参阅文档^(了解详情。)

## 概要

WASI 动力的沙盒允许代码片段完全在浏览器中运行，不涉及服务器。 它们可能需要一些时间和流量来初始化运行时，但之后它们几乎立即运行。

正如在 Codapi 中实现的那样，它们完美地融入了整体架构，提供了对模板和代码单元等功能的访问。 您还可以轻松地从浏览器端切换到服务器端执行模型。

尝试一下！

[游乐场](https://github.com/nalgeon/codapi-js/blob/main/docs/browser-only.md) • [片段小部件](https://github.com/nalgeon/codapi-js) • [关于 Codapi](https://codapi.org/)

*[* **订阅***](/subscribe/) *以保持对新文章的跟踪。**
