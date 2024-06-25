<!--yml

category: 未分类

日期：2024-05-27 15:07:34

-->

# scrapscript.py | Max Bernstein

> 来源：[`bernsteinbear.com/blog/scrapscript/`](https://bernsteinbear.com/blog/scrapscript/)

[Scrapscript](https://scrapscript.org/) 是一种小型、纯粹的、功能型、内容可寻址的、以网络为先的编程语言。它旨在允许创建小型、简单共享的程序。该语言由 [Taylor Troesh](https://taylor.town/) 创建，主要实现由我和 [Chris](https://www.chrisgregory.me/) 创建。

```
fact 5
. fact =
  | 0 -> 1
  | n -> n * fact (n - 1) 
```

我不打算填写 [通常的清单](https://www.mcmillen.dev/language_checklist.html)——这不是本文的重点，而且很多问题在网站上都有答案。本文更多地涉及实现方面的内容。

## 一点历史

2023 年 4 月，我在 Hacker News 上看到了 scrapscript 的发布，并将其发送给了 Chris。我们经常互相发送新的编程语言，他非常喜欢函数式编程，所以我想他会喜欢它。他确实喜欢！

但我们没有看到任何下载或浏览实现的链接，所以我们有点失望。我们喜欢尝试各种东西，了解它们的运作方式。一个月或两个月过去了，仍然没有实现，所以我们决定给 Taylor 发电子邮件问问我们能否提供帮助。

> 主题：Chris 和我想帮你处理 scrapscript
> 
> 嗨 Taylor，
> 
> 我的朋友 Chris（抄送）和我对 scrapscript 感到兴奋。这是我们多年来一直想要构建的项目。他是一个对 Haskell 具有不合理兴趣的 ML 程序员，我是一个 PL/编译器程序员（对 Haskell 不是很感兴趣），很乐意提供帮助，或者至少尝试早期版本。
> 
> Chris：https://www.chrisgregory.me/
> 
> 我：https://bernsteinbear.com/
> 
> 问候，
> 
> Max

Taylor 对整个事情非常慷慨，并分享了他的 [scrapscript 的小型 JavaScript 实现](https://github.com/tekknolagi/scrapscript/blob/71d1afecc32879aed9c80a3ed17cb81fe1c010d6/scrapscript.ts)。对于这样一个小文件，它实现了一个令人印象深刻的功能集。

Chris 和我对 JavaScript 不是特别擅长，我们认为更多的实现不会有害，所以我们决定用相同的功能制作一个并行实现。虽然 Taylor 的主要设计约束是实现大小，但我们的约束是可读性和正确性的某种组合。这意味着我们在编写过程中撰写了大量的测试来确定预期的行为。

在这个小型黑客马拉松进行了两天之后，我们告诉了 Taylor，他对此感到非常高兴，所以我们开始定期交流并继续进行编程。三周后，我们将 [repo](https://github.com/tekknolagi/scrapscript) 公开了。请随意查看！大部分内容都在一个文件中，`scrapscript.py`。这是我们早期的设计决策之一…

## 实现设计决策

虽然我们没有像 Taylor 一样明确地试图控制实现的大小，但我们确实希望将其保持自包含。这导致了一些隐式和显式的设计选择：

**核心功能没有外部依赖。**使核心可理解，而无需去查阅其他库。

作为某种推论，**尽量减少对主机编程语言（Python）不寻常或花哨特性的依赖。**我们希望确保将实现移植到另一种编程语言尽可能容易。

**在实现级别和端到端的测试功能。**编写调用`tokenize`、`parse`和`eval_exp`的函数，以便我们尽可能地捕获错误和其他行为。同时编写完整的集成测试，因为这些测试可以作为功能展示，并且很容易移植到其他实现中。

```
class Tests(unittest.TestCase):
    def test_tokenize_binary_sub_no_spaces(self) -> None:
        self.assertEqual(tokenize("1-2"), [IntLit(1), Operator("-"), IntLit(2)])
    # ...
    def test_parse_binary_sub_returns_binop(self) -> None:
        self.assertEqual(
            parse([IntLit(1), Operator("-"), IntLit(2)]),
            Binop(BinopKind.SUB, Int(1), Int(2)),
        )
    # ...
    def test_eval_with_binop_sub(self) -> None:
        exp = Binop(BinopKind.SUB, Int(1), Int(2))
        self.assertEqual(eval_exp({}, exp), Int(-1))
    # ...
    def test_int_sub_returns_int(self) -> None:
        self.assertEqual(self._run("1 - 2"), Int(-1)) 
```

选择 Python 主要是一个怪癖。Python 一般来说并不特别；它只是恰好是我和克里斯经常使用的一种语言。

### 我们测试策略的后果

确保尽早进行测试并进行彻底的测试带来了一些出色的后果。这意味着，我们可以在将其移植时，记录在`scrapscript.js`中发现的预期行为，并持续检查该记录。这也意味着，当我们[彻底重建它](https://github.com/tekknolagi/scrapscript/commit/082e30375225394f30fd270ffdcee7f5d63173ae)时，我们非常有信心我们没有把一切都搞砸。

```
 def tokenize(x: str) -> list[str]:
-    # TODO: Make this a proper tokenizer that handles strings with blankspace.
-    stripped = re.sub(r" *--[^\n]*", "", x).strip()
-    return re.split(r"[\s\n]+", stripped) +    lexer = Lexer(x)
+    tokens = []
+    while lexer.has_input():
+        tokens.append(lexer.read_one())
+    return tokens 
```

所有的测试都继续通过，我们甚至可以启用一个新的测试！

## 为什么这个解释器与所有其他解释器不同？

不是。这是一个相当标准的树遍历解释器，用于增强的 lambda 演算^(也许我们最终会生成字节码或其他 IR 并将其编译，但我们目前没有任何性能问题。而且 scrapscript 并不像一个“工业强度”的语言；没有人在其中编写大型应用程序，而且该语言明确不是为此设计的。)

对我来说有点不同，因为它具有我以前从未实现过的功能！特别是，scrapscript 支持一些相当广泛的模式匹配，我们不得不学习如何从零开始实现它。

```
def eval_exp(env: Env, exp: Object) -> Object:
    # ...
    if isinstance(exp, Apply):
        callee = eval_exp(env, exp.func)
        arg = eval_exp(env, exp.arg)
        # ...
        if isinstance(callee.func, MatchFunction):
            for case in callee.func.cases:
                m = match(arg, case.pattern)
                if m is None:
                    continue
                return eval_exp({**callee.env, **m}, case.body)
            raise MatchError("no matching cases")
        # ... 
```

它也不同，因为这是我与他人合作的第一个从零开始的语言实现（我想）。克里斯一直是一个出色的共同实现者，考虑到这是他第一次实现编程语言，这是非常令人印象深刻的！

## 一些巧妙的实现特性

如果你不尝试一些新的技巧和技术，为什么要有一个小型编程项目呢？

### REPL

您可能已经看过我最近的博客文章，关于使用 Python 提供的一个很好的库构建一个功能丰富的 REPL。我在学习有关 scrapscript 的所有内容时编写了那篇文章。Scrapscript 的 REPL 实现非常简短，但它具有`readline`支持、制表符补全、行继续等功能。谢谢，Python！

```
>>> $$[^tab]
$$add         $$fetch       $$jsondecode  $$listlength  $$serialize
>>> $$add 
```

### 一个真正可移植的可执行文件

我们使用 Justine Tunney 的[Cosmopolitan](https://justine.lol/cosmopolitan/)将 scrapscript 构建为一个实际的便携可执行文件。这意味着它打包了一个小的 libc 实现和 Python 运行时到一个（相对）小的、自包含的可执行文件中。这个可执行文件理论上可以在所有主要平台上轻松运行。而我们用它构建的 Docker 容器总共只有 25.5MB（~~36MB~~），因为它不需要在文件系统中有一堆操作系统的东西。

```
$ docker images ghcr.io/tekknolagi/scrapscript
REPOSITORY                      TAG    IMAGE ID       CREATED       SIZE
ghcr.io/tekknolagi/scrapscript  trunk  16867189d853   3 hours ago   25.5MB
$ 
```

请查看[build-com](https://github.com/tekknolagi/scrapscript/blob/e38210f7aa8ce375a7e615b301922bd7b9710d37/build-com)和[Dockerfile](https://github.com/tekknolagi/scrapscript/blob/e38210f7aa8ce375a7e615b301922bd7b9710d37/Dockerfile)以获取更多信息。

### 网页 REPL

我们希望有一个像许多其他编程语言一样的交互式 playground。然而，我并不想再次实现 scrapscript。所以我写了一个小的无状态服务器程序，在 Python 中是一个函数`(env, exp) -> (env', result)`，并编写了一个 JS 程序来驱动 web 请求。你得到的是[web REPL](https://scrapscript.fly.dev/repl)。构建这个需要能够序列化对象和环境，以便它们可以作为不透明 blob 存储在客户端上。这基本上已经可以了，但我还没有一个完整的解决方案来处理带有循环的对象。然而，这个功能正在进行中！

## 进度中的功能

正如我之前提到的，带有循环的对象序列化是一个正在进行的工作。这需要在序列化器内部添加对假`ref`类型的支持，并在反序列化器中解析它们。不过，很快就应该准备好了。

我们还没有完全支持主网站上描述的备用功能。这并不特别困难，但还没有人实现它。不过，我们确实让符号起作用了，所以我们在 scrapscript 中实现了`#true`和`#false`。

```
some-sum-type :
  #cowboy
  #ron int
  #favcolor (#green #blue #other)
  #friend int
  #stranger text 
```

我们还在第一次实现 scrapyards。我们还不确定要走哪种设计方向，所以 Taylor 和我各自以不同的方式原型化了它。我的实现使用 Git 作为版本化的 blob 存储，以便对其非常懒惰，并重复使用一堆现有的基础设施。这是我对这个实现的个人目标之一：首先是最小化实现。

Scrapscript 带有平台的概念——主机运行时给 scrap 提供的不同 API 集。目前还没有真正实现，但我确实原型化了一个“web 平台”。这是一个围绕 scrap 的小 Python shell，将 web 请求作为 scrap 记录提供给它。然后 scrap 可以使用模式匹配来路由请求并构建请求。正如网站上所说的，scrapscript 对于 HTML DSL 来说相当不错。

```
handler
. handler =
  | { path = "/" } -> (status 200 <| page "you're on the index")
  | { path = "/about" } -> (status 200 <| page "you're on the about page")
  | x -> (status 404 <| page "not found")
. status = code -> content -> { code = code, content = content }
. page = body -> "<!doctype html><html><body>" ++ body ++ "</body></html>" 
```

在未来，我认为编写一个函数`(db, request) -> (db', response)`并拥有一个无状态的 webserver 内核，即使外部世界将变化应用到数据库上，它仍然可以存储数据，可能会很有趣。

克里斯和我还谈论了构建一个图形 API 或图形平台的事情。我们不确定这是什么样子，但他对细胞自动机很感兴趣，能够直接从 scrap 写像素，而不用经过 PPM 或其他东西，这将是很棒的。

最后，但肯定不是最不重要的，我们正在开发一个 scrapscript 编译器。不过，有趣的是，这个编译器是用*scrapscript*编写的。目标是通过将编译器编译成 JS（在 Python 解释器之上），然后在 web 上运行编译器（现在是 JS 代码），将 scrapscript 移植到浏览器上，而不是通过在 JS 中编写一个新的解释器。

```
compile =
| {type="Int", value=value} -> $$int_as_str value
| {type="Var", name=name} -> name
| {type="String", value=value} -> $$str_as_str value
| {type="Binop", op="++", left=left, right=right} ->
    -- Special case string concat
    (compile left) ++ "+" ++ (compile right)
| {type="Binop", op=op, left=left, right=right} ->
    (compile left) ++ op ++ (compile right)
| {type="List", items=items} ->
    "[" ++ (join ", " (map compile items)) ++ "]"
-- ... 
```

## 感谢阅读。

首先，玩一玩[web REPL](https://scrapscript.fly.dev/repl)。然后查看[repo](https://github.com/tekknolagi/scrapscript)，开始贡献吧！由于我们目前还没有一大堆明确定义范围的项目积压，我建议先在[discourse group](https://scrapscript.discourse.group/)发帖，了解一下对你来说最有用和最有趣的是什么。
