<!--yml

category: 未分类

date: 2024-05-27 14:50:48

-->

# (如何编写（Lisp）解释器（在Python中）)

> 来源：[https://www.norvig.com/lispy.html](https://www.norvig.com/lispy.html)

# (如何编写（Lisp）解释器（在Python中）)

This page has two purposes: to describe how to implement computer language interpreters in general, and in particular to build an interpreter for most of the [*Scheme*](http://en.wikipedia.org/wiki/Scheme_(programming_language)) dialect of Lisp using [Python 3](http://python.org) as the implementation language. I call my language and interpreter *Lispy* ([**lis.py**](lis.py)). Years ago, I showed how to write a semi-practical Scheme interpreter [Java](jscheme.html) and in [in Common Lisp](http://books.google.com/books?id=QzGuHnDhvZIC&lpg=PA756&vq=scheme%20interpreter&dq=Paradigms%20of%20Artificial%20Intelligence%20Programming&pg=PA753#v=onepage&q&f=false)). This time around the goal is to demonstrate, as concisely and simply as possible, what [Alan Kay called](http://queue.acm.org/detail.cfm?id=1039523) "[*Maxwell's Equations of Software*](http://www.righto.com/2008/07/maxwells-equations-of-software-examined.html)."

为什么这很重要？正如[Steve Yegge所说](http://steve-yegge.blogspot.com/2007/06/rich-programmer-food.html)，*"如果你不知道编译器是如何工作的，那么你不知道计算机是如何工作的。"* Yegge描述了8个可以通过编译器解决的问题（或者同样可以通过解释器解决，或者以Yegge典型的大量讽刺方式解决的问题）。

## Scheme程序的语法和语义

一种语言的*语法*是形成正确语句或表达式的字符排列；*语义*是这些语句或表达式的含义。例如，在数学表达式的语言中（以及许多编程语言中），添加1加2的语法是"1 + 2"，语义是将加法操作应用于这两个数字，得到值3。当我们确定表达式的值时，我们说我们正在*评估*该表达式；我们会说"1 + 2"评估为3，并写为"1 + 2" ⇒ 3。

Scheme语法与大多数其他编程语言不同。考虑：

> | Java |        | Scheme |
> | --- | --- | --- |
> |  |
> 
> | `**if** (x.val() > 0) {   **return** fn(A[i] + 3 * i,
> 
> **new** String[] {"one", "two"});
> 
> }` |   | `(**if** (> (val x) 0)     (fn (+ (aref A i) (* 3 i))
> 
> (**quote** (one two)))` |

Java has a wide variety of syntactic conventions (keywords, infix operators, three kinds of brackets, operator precedence, dot notation, quotes, commas, semicolons), but Scheme syntax is much simpler:

+   Scheme程序仅由*表达式*组成。没有语句/表达式的区别。

+   Numbers (e.g. `1`) and symbols (e.g. `A`) are called *atomic expressions*; they cannot be broken into pieces. These are similar to their Java counterparts, except that in Scheme, operators such as `+` and `>` are symbols too, and are treated the same way as `A` and `fn`.

+   其他都是 *列表表达式*：一个"(", 后面跟着零个或多个表达式，然后是")"。列表的第一个元素决定了其含义：

    +   以关键字开头的列表，如`(if ...)`，是一个 *特殊形式*；其含义取决于关键字。

    +   以非关键字开头的列表，如`(fn ...)`，是一个函数调用。

Scheme的美妙之处在于整个语言只需要5个关键字和8个语法形式。相比之下，Python有33个关键字和[110](https://docs.python.org/3/reference/grammar.html)个语法形式，Java有50个关键字和[133](https://docs.oracle.com/javase/specs/jls/se7/html/jls-18.html)个语法形式。所有这些括号可能看起来令人望而生畏，但Scheme语法具有简单和一致的优点。（有人开玩笑说"Lisp"代表"[***L**ots of **I**rritating **S**illy **P**arentheses*](http://www.google.com/search?q=Lots+of+Irritating+Silly+Parentheses)"; 我认为它代表"[***L**isp **I**s **S**yntactically **P**ure*](http://www.google.com/search?hl=en&as_q=&as_epq=Lisp+Is+Syntactically+Pure)"。)

在本页中，我们将涵盖Scheme语言及其解释的所有重要要点（省略一些细节），但我们将分两步完成，首先定义一个简化的语言，然后再定义接近完整的Scheme语言。

## 语言1：Lisp计算器

*Lispy Calculator* 是Scheme的一个子集，只使用了五个语法形式（两个原子形式，两个特殊形式和过程调用）。Lispy Calculator让你能够执行任何你在典型计算器上进行的计算，只要你习惯前缀表示法。并且你可以做两件典型计算器语言不提供的事情："if"表达式，和定义新变量。以下是一个计算半径为10的圆的面积的示例程序，使用公式 π *r*²：

```
(define r 10)
(* pi (* r r))

```

这里是所有可接受的表达式的表格：

| 表达式 | 语法 | 语义和例子 |
| --- | --- | --- |
| [变量引用](http://www.schemers.org/Documents/Standards/R5RS/HTML/r5rs-Z-H-7.html#%_sec_4.1.1) | *symbol* | 一个符号被解释为一个变量名；它的值是该变量的值。例如：`r` ⇒ `10`（假设`r`之前被定义为10） |
| [常量字面量](http://www.schemers.org/Documents/Standards/R5RS/HTML/r5rs-Z-H-7.html#%_sec_4.1.2) | *number* | 一个数字求值为它自身。例如：`12 ⇒ 12` 或 `-3.45e+6 ⇒ -3.45e+6` |
| [条件](http://www.schemers.org/Documents/Standards/R5RS/HTML/r5rs-Z-H-7.html#%_sec_4.1.5) | `(if` *test conseq alt*`)` | 对*test*进行评估；如果为真，评估并返回*conseq*；否则返回*alt*。例如：`(if (> 10 20) (+ 1 1) (+ 3 3)) ⇒ 6` |
| [定义](http://www.schemers.org/Documents/Standards/R5RS/HTML/r5rs-Z-H-8.html#%_sec_5.2) | `(define` *symbol* *exp*`)` | 定义一个新变量，并将其值设为求值表达式 *exp*。例如：`(define r 10)` |
| [过程调用](http://www.schemers.org/Documents/Standards/R5RS/HTML/r5rs-Z-H-7.html#%_sec_4.1.3) | `(`*proc arg...*`)` | 如果 *proc* 不是 `if, define` 或 `quote` 中的任何一个符号，则将其视为过程。评估 *proc* 和所有 *args*，然后将该过程应用于 *arg* 值列表。例如：`(sqrt (* 2 8)) ⇒ 4.0` |

在此表的语法列中，*symbol* 必须是符号，*number* 必须是整数或浮点数，其他斜体字可以是任何表达式。符号 *arg...* 表示 *arg* 的零个或多个重复。

## 语言解释器的功能

语言解释器有两部分：

1.  **解析：** 解析组件接受以字符序列形式的输入程序，根据语言的*语法规则*进行验证，并将程序转换为内部表示。在简单的解释器中，内部表示是树结构（通常称为*抽象语法树*），它紧密地反映了程序中语句或表达式的嵌套结构。在称为*编译器*的语言翻译器中，通常有一系列内部表示，从抽象语法树开始，进展到一系列可以由计算机直接执行的指令。Lispy 解析器使用函数 `parse` 实现。

1.  **执行：** 然后根据语言的*语义规则*处理内部表示，从而进行计算。Lispy 的执行函数称为 `eval`（请注意，这与 Python 的同名内置函数相冲突）。

这里是解释过程的图片：

> 程序 ➡ `parse` ➡ 抽象语法树 ➡ `eval` ➡ 结果

这里是我们希望 `parse` 和 `eval` 能够执行的简短示例（`begin` 按顺序评估每个表达式并返回最终一个）:

```
>> program = "(begin (define r 10) (* pi (* r r)))"

>>> parse(program)
['begin', ['define', 'r', 10], ['*', 'pi', ['*', 'r', 'r']]]

>>> eval(parse(program))
314.1592653589793

```

## 类型定义

让我们明确我们对 Scheme 对象的表示：

```
Symbol = str              # A Scheme Symbol is implemented as a Python str
Number = (int, float)     # A Scheme Number is implemented as a Python int or float
Atom   = (Symbol, Number) # A Scheme Atom is a Symbol or Number
List   = list             # A Scheme List is implemented as a Python list
Exp    = (Atom, List)     # A Scheme expression is an Atom or List
Env    = dict             # A Scheme environment (defined below) 
                          # is a mapping of {variable: value}

```

## 解析：`parse`，`tokenize` 和 `read_from_tokens`

解析通常分为两部分：*词法分析*，其中输入字符串被分解为一系列*标记*，以及*语法分析*，其中标记被组装成抽象语法树。Lispy 的标记包括括号、符号和数字。有许多词法分析工具（如 Mike Lesk 和 Eric Schmidt 的 [lex](http://dinosaur.compilertools.net/#lex)），但现在我们将使用一个非常简单的工具：Python 的 `str.split` 函数。函数 `tokenize` 的输入是一个字符串；它在每个括号周围添加空格，然后调用 `str.split` 来获取一个标记列表：

```
def tokenize(chars: str) -> list:
    "Convert a string of characters into a list of tokens."
    return chars.replace('(', ' ( ').replace(')', ' ) ').split()

```

我们在这里将 tokenize 应用于我们的示例程序：

```
>>> program = "(begin (define r 10) (* pi (* r r)))"
>>> tokenize(program)
['(', 'begin', '(', 'define', 'r', '10', ')', '(', '*', 'pi', '(', '*', 'r', 'r', ')', ')', ')']

```

我们的函数`parse`将接受一个程序的字符串表示作为输入，调用`tokenize`以获取一个令牌列表，然后调用`read_from_tokens`以组装一个抽象语法树。`read_from_tokens`查看第一个令牌；如果它是`')'`，那么是语法错误。如果是`'('`，则我们开始构建一个子表达式列表，直到找到匹配的`')'`为止。任何非括号令牌必须是符号或数字。我们将让Python来区分它们：对于每个非括号令牌，首先尝试将其解释为整数，然后尝试解释为浮点数，如果既不是整数也不是浮点数，则必须是符号。这是解析器：

```
def parse(program: str) -> Exp:
    "Read a Scheme expression from a string."
    return read_from_tokens(tokenize(program))

def read_from_tokens(tokens: list) -> Exp:
    "Read an expression from a sequence of tokens."
    if len(tokens) == 0:
        raise SyntaxError('unexpected EOF')
    token = tokens.pop(0)
    if token == '(':
        L = []
        while tokens[0] != ')':
            L.append(read_from_tokens(tokens))
        tokens.pop(0) # pop off ')'
        return L
    elif token == ')':
        raise SyntaxError('unexpected )')
    else:
        return atom(token)

def atom(token: str) -> Atom:
    "Numbers become numbers; every other token is a symbol."
    try: return int(token)
    except ValueError:
        try: return float(token)
        except ValueError:
            return Symbol(token)

```

`parse`的工作方式如下：

```
>>> program = "(begin (define r 10) (* pi (* r r)))"

>>> parse(program)
['begin', ['define', 'r', 10], ['*', 'pi', ['*', 'r', 'r']]]

```

我们差不多可以定义`eval`了。但是我们首先需要再介绍一个概念。

## 环境

环境是从变量名到它们的值的映射。默认情况下，`eval`将使用一个全局环境，其中包括许多标准函数（如`sqrt`和`max`，以及像`*`这样的操作符）。此环境可以通过使用表达式`(define *symbol value*)`来扩充用户定义的变量。

```
import math
import operator as op

def standard_env() -> Env:
    "An environment with some Scheme standard procedures."
    env = Env()
    env.update(vars(math)) # sin, cos, sqrt, pi, ...
    env.update({
        '+':op.add, '-':op.sub, '*':op.mul, '/':op.truediv, 
        '>':op.gt, '=':op.ge, '<=':op.le, '=':op.eq, 
        'abs':     abs,
        'append':  op.add,  
        'apply':   lambda proc, args: proc(*args),
        'begin':   lambda *x: x[-1],
        'car':     lambda x: x[0],
        'cdr':     lambda x: x[1:], 
        'cons':    lambda x,y: [x] + y,
        'eq?':     op.is_, 
        'expt':    pow,
        'equal?':  op.eq, 
        'length':  len, 
        'list':    lambda *x: List(x), 
        'list?':   lambda x: isinstance(x, List), 
        'map':     map,
        'max':     max,
        'min':     min,
        'not':     op.not_,
        'null?':   lambda x: x == [], 
        'number?': lambda x: isinstance(x, Number),  
		'print':   print,
        'procedure?': callable,
        'round':   round,
        'symbol?': lambda x: isinstance(x, Symbol),
    })
    return env

global_env = standard_env()

```

## 评估：`eval`

现在我们准备实现`eval`。作为复习，我们重复列出Lispy计算器的形式表格：

| 表达式 | 语法 | 语义和示例 |
| --- | --- | --- |
| [变量引用](http://www.schemers.org/Documents/Standards/R5RS/HTML/r5rs-Z-H-7.html#%_sec_4.1.1) | *symbol* | 符号被解释为变量名；其值是变量的值。例子：`r` ⇒ `10`（假设`r`先前已定义为10） |
| [常量文字](http://www.schemers.org/Documents/Standards/R5RS/HTML/r5rs-Z-H-7.html#%_sec_4.1.2) | *number* | 数字评估为它自己。示例：`12 ⇒ 12` *或* `-3.45e+6 ⇒ -3.45e+6` |
| [条件](http://www.schemers.org/Documents/Standards/R5RS/HTML/r5rs-Z-H-7.html#%_sec_4.1.5) | `(if` *test conseq alt*`)` | 评估*test*；如果为真，评估并返回*conseq*；否则*alt*。示例：`(if (> 10 20) (+ 1 1) (+ 3 3)) ⇒ 6` |
| [定义](http://www.schemers.org/Documents/Standards/R5RS/HTML/r5rs-Z-H-8.html#%_sec_5.2) | `(define` *symbol* *exp*`)` | 定义一个新变量，并将其值设为求值表达式*exp*的结果。示例：`(define r 10)` |
| [过程调用](http://www.schemers.org/Documents/Standards/R5RS/HTML/r5rs-Z-H-7.html#%_sec_4.1.3) | `(`*proc arg...*`)` | 如果*proc*不是`if, define,`或`quote`中的一个符号，则将其视为过程。评估*proc*和所有*args*，然后将该过程应用于*arg*值的列表。示例：`(sqrt (* 2 8)) ⇒ 4.0` |

这里是`eval`的代码，其紧随表格而来：

```
def eval(x: Exp, env=global_env) -> Exp:
    "Evaluate an expression in an environment."
    if isinstance(x, Symbol):        # variable reference
        return env[x]
    elif isinstance(x, Number):      # constant number
        return x                
    elif x[0] == 'if':               # conditional
        (_, test, conseq, alt) = x
        exp = (conseq if eval(test, env) else alt)
        return eval(exp, env)
    elif x[0] == 'define':           # definition
        (_, symbol, exp) = x
        env[symbol] = eval(exp, env)
    else:                            # procedure call
        proc = eval(x[0], env)
        args = [eval(arg, env) for arg in x[1:]]
        return proc(*args)

```

*我们完成了！* 你可以看到它如何运行：

```
>>> eval(parse("(begin (define r 10) (* pi (* r r)))"))
314.1592653589793

```

## 交互：一个REPL

每次输入 `eval(parse("..."))` 都很繁琐。Lisp 的一个伟大遗产之一是交互式读取-评估-打印循环：程序员可以输入一个表达式，并立即看到它被读取、评估和打印，而无需经历冗长的构建/编译/运行循环。因此，让我们定义函数 `repl`（代表读取-评估-打印循环）和函数 `schemestr`，后者返回表示 Scheme 对象的字符串。

```
def repl(prompt='lis.py> '):
    "A prompt-read-eval-print loop."
    while True:
        val = eval(parse(raw_input(prompt)))
        if val is not None: 
            print(schemestr(val))

def schemestr(exp):
    "Convert a Python object back into a Scheme-readable string."
    if isinstance(exp, List):
        return '(' + ' '.join(map(schemestr, exp)) + ')' 
    else:
        return str(exp)

```

`repl` 的实际操作如下：

```
>>> repl()
lis.py> (define r 10)
lis.py> (* pi (* r r))
314.159265359
lis.py> (if (> (* 11 11) 120) (* 7 6) oops)
42
lis.py> (list (+ 1 1) (+ 2 2) (* 2 3) (expt 2 3))
lis.py> 

```

## 语言 2：完全的 Lisp 风格

现在我们将通过引入三种新的特殊形式扩展我们的语言，从而得到一个更接近完整的 Scheme 子集：

| 表达式 | 语法 | 语义和示例 |
| --- | --- | --- |
| [quotation](http://www.schemers.org/Documents/Standards/R5RS/HTML/r5rs-Z-H-7.html#%_sec_4.1.2) | `(quote` *exp*`)` | 直接返回 *exp*；不对其进行评估。例如：`(quote (+ 1 2)) ⇒ (+ 1 2)` |
| [assignment](http://www.schemers.org/Documents/Standards/R5RS/HTML/r5rs-Z-H-7.html#%_sec_4.1.6) | `(set!` *symbol exp*`)` | 评估 *exp* 并将其值分配给之前定义过的 *symbol*（使用 `define` 或作为封闭过程的参数）。例如：`(set! r2 (* r r))` |
| [procedure](http://www.schemers.org/Documents/Standards/R5RS/HTML/r5rs-Z-H-7.html#%_sec_4.1.4) | `(lambda (`*symbol...*`)` *exp*`)` | 创建一个具有参数名为 *symbol...* 和体为 *exp* 的过程。例如：`(lambda (r) (* pi (* r r)))` |

`lambda` 特殊形式（一个晦涩的命名选择，指的是阿隆佐·邱奇的[λ演算](http://en.wikipedia.org/wiki/Lambda_calculus)）创建一个过程。我们希望过程能像这样工作：

```
lis.py> (define circle-area (lambda (r) (* pi (* r r)))
lis.py> (circle-area (+ 5 5))
314.159265359

```

这里有两个步骤。在第一步中，`lambda` 表达式被评估以创建一个过程，该过程引用全局变量 `pi` 和 `*`，接受一个称为 `r` 的单一参数。这个过程被用作新变量 `circle-area` 的值。在第二步中，我们刚刚定义的过程是 `circle-area` 的值，因此它被调用，参数为值为 10。我们希望 `r` 取值为 10，但不能简单地在全局环境中设置 `r` 为 10。如果我们正在使用 `r` 作其他用途，我们不希望调用 `circle-area` 导致该值发生改变。相反，我们希望安排有一个名为 `r` 的*局部*变量，可以将其设置为 10，而不必担心干扰具有相同名称的任何全局变量。调用过程的过程引入了这些新的局部变量，将函数参数列表中的每个符号绑定到函数调用参数列表中的相应值。

## 重新定义 `Env` 为一个类

要处理局部变量，我们将重新定义 `Env` 为 `dict` 的一个子类。当我们评估 `(circle-area (+ 5 5))` 时，我们将获取过程体 `(* pi (* r r))` 并在一个环境中评估它，该环境将 `r` 作为唯一的局部变量（值为10），同时也将全局环境作为“外部”环境；在那里我们将找到 `*` 和 `pi` 的值。换句话说，我们希望一个看起来像这样的环境，局部（蓝色）环境嵌套在外部（红色）全局环境中：

| `pi: 3.141592653589793 *: <内置函数 mul>

...`  |

在这样的嵌套环境中查找变量时，首先查找最内层，如果变量名不在那里找到，则移动到下一个外层。过程和环境是交织在一起的，因此让我们一起定义它们：

```
class Env(dict):
    "An environment: a dict of {'var': val} pairs, with an outer Env."
    def __init__(self, parms=(), args=(), outer=None):
        self.update(zip(parms, args))
        self.outer = outer
    def find(self, var):
        "Find the innermost Env where var appears."
        return self if (var in self) else self.outer.find(var)

class Procedure(object):
    "A user-defined Scheme procedure."
    def __init__(self, parms, body, env):
        self.parms, self.body, self.env = parms, body, env
    def __call__(self, *args): 
        return eval(self.body, Env(self.parms, args, self.env))

global_env = standard_env()

```

我们看到每个过程有三个组成部分：参数名列表、体表达式和一个告诉我们从体表达式中可以访问哪些其他变量的环境。对于在顶层定义的过程来说，这将是全局环境，但过程也可以引用它定义时的环境的局部变量（而不是调用时的环境）。

环境是 `dict` 的一个子类，因此它具有 `dict` 的所有方法。此外，还有两个方法：构造函数 `__init__` 通过取参数名列表和对应的参数值列表，创建一个具有这些 {变量: 值} 对作为内部部分，并且还引用给定的 `outer` 环境的新环境。方法 `find` 用于找到变量的正确环境：内部环境或外部环境。

要看看它们是如何一起工作的，请看 `eval` 的新定义。注意变量引用的条款已经改变：现在我们必须调用 `env.find(x)` 来找到变量 `x` 存在的级别；然后我们可以从那个级别获取 `x` 的值。（`define` 的条款没有改变，因为 `define` 总是在最内层环境中添加一个新变量。）还有两个新的条款：对于 `set!`，我们找到变量存在的环境级别，并将其设置为新值。对于 `lambda`，我们创建一个新的过程对象，其中包括给定的参数列表、体和环境。

```
def eval(x, env=global_env):
    "Evaluate an expression in an environment."
    if isinstance(x, Symbol):    # variable reference
        return env.find(x)[x]
    elif not isinstance(x, List):# constant 
        return x   
    op, *args = x       
    if op == 'quote':            # quotation
        return args[0]
    elif op == 'if':             # conditional
        (test, conseq, alt) = args
        exp = (conseq if eval(test, env) else alt)
        return eval(exp, env)
    elif op == 'define':         # definition
        (symbol, exp) = args
        env[symbol] = eval(exp, env)
    elif op == 'set!':           # assignment
        (symbol, exp) = args
        env.find(symbol)[symbol] = eval(exp, env)
    elif op == 'lambda':         # procedure
        (parms, body) = args
        return Procedure(parms, body, env)
    else:                        # procedure call
        proc = eval(op, env)
        vals = [eval(arg, env) for arg in args]
        return proc(*vals)

```

要理解过程和环境如何协同工作，可以考虑这个程序及其在评估 `(account1 -20.00)` 时形成的环境：

|

&#124; `(define **make-account**

&#124;   (lambda (**balance**)

&#124;     (lambda (**amt**)       (begin **(set! balance (+ balance amt))**

balance)))) &#124;

&#124;

`(define **account1** (make-account 100.00))

(account1 -20.00)`` &#124;

|   |
| --- |

&#124; **`+`**: <内置运算符 add> `**make-account**: <一个Procedure>

``**account1**: <一个Procedure>``` &#124;

 |

Each rectangular box represents an environment, and the color of the box matches the color of the variables that are newly defined in the environment. In the last two lines of the program we define `account1` and call `(account1 -20.00)`; this represents the creation of a bank account with a 100 dollar opening balance, followed by a 20 dollar withdrawal. In the process of evaluating `(account1 -20.00)`, we will eval the expression highlighted in yellow. There are three variables in that expression. `amt` can be found immediately in the innermost (green) environment. But `balance` is not defined there: we have to look at the green environment's outer `env`, the blue one. And finally, the variable `+` is not found in either of those; we need to do one more outer step, to the global (red) environment. This process of looking first in inner environments and then in outer ones is called *lexical scoping*. `Env.find(var)` finds the right environment according to lexical scoping rules.

Let's see what we can do now:

```

>>> repl()

lis.py> (define circle-area (lambda (r) (* pi (* r r))))

lis.py> (circle-area 3)

28.274333877

lis.py> (define fact (lambda (n) (if (<= n 1) 1 (* n (fact (- n 1))))))

lis.py> (fact 10)

3628800

lis.py> (fact 100)

9332621544394415268169923885626670049071596826438162146859296389521759999322991

5608941463976156518286253697920827223758251185210916864000000000000000000000000

lis.py> (circle-area (fact 10))

4.1369087198e+13

lis.py> (define first car)

lis.py> (define rest cdr)

lis.py> (define count (lambda (item L) (if L (+ (equal? item (first L)) (count item (rest L))) 0)))

lis.py> (count 0 (list 0 1 2 3 0 0))

3

lis.py> (count (quote the) (quote (the more the merrier the bigger the better)))

4

lis.py> (define twice (lambda (x) (* 2 x)))

lis.py> (twice 5)

10

lis.py> (define repeat (lambda (f) (lambda (x) (f (f x)))))

lis.py> ((repeat twice) 10)

40

lis.py> ((repeat (repeat twice)) 10)

160

lis.py> ((repeat (repeat (repeat twice))) 10)

2560

lis.py> ((repeat (repeat (repeat (repeat twice)))) 10)

655360

lis.py> (pow 2 16)

65536.0

lis.py> (define fib (lambda (n) (if (< n 2) 1 (+ (fib (- n 1)) (fib (- n 2))))))

lis.py> (define range (lambda (a b) (if (= a b) (quote ()) (cons a (range (+ a 1) b)))))

lis.py> (range 0 10)

(0 1 2 3 4 5 6 7 8 9)

lis.py> (map fib (range 0 10))

(1 1 2 3 5 8 13 21 34 55)

lis.py> (map fib (range 0 20))

(1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597 2584 4181 6765)

```

现在我们有了一个具有过程、变量、条件（`if`）和顺序执行（`begin`过程）的语言。如果您熟悉其他语言，您可能会认为需要`while`或`for`循环，但Scheme语言完全没有这些也能运行良好。Scheme语言报告说：“Scheme展示了形成表达式的非常少的规则，以及这些规则如何组合，足以形成一个实用且高效的编程语言。”在Scheme语言中，您通过定义递归函数进行迭代。

## Lisp语言的大小/速度/完整性/优良性如何？

在这里，我们根据几个标准评判Lisp语言：

## 真实故事

为了支持这个想法——了解解释器如何工作是非常有帮助的——这里有一个故事。早在1984年，我正在写博士论文。当时还没有LaTeX，也没有Windows上的Microsoft Word——我们使用troff排版系统。不幸的是，troff系统不支持向符号标签的前向引用：我想能够写“正如我们将在@theorem-x页面上看到”，然后在适当的地方写上类似“@(set theorem-x \n%)”（其中troff寄存器\n%保存页面号）。我的同事Tony DeRose也有同样的需求，于是我们一起草拟了一个简单的Lisp程序，作为预处理器来处理这个问题。然而，当时的Lisp语言在读取Lisp表达式时效率很高，但在逐字符读取非Lisp表达式时速度非常慢，使得我们的程序使用起来很烦人。

从那时起，托尼和我选择了不同的道路。他认为困难的部分是表达式的解释器；他需要用Lisp来做这件事，但他知道如何写一个小的C程序来读取和回显非Lisp字符，并将其链接到Lisp程序中。我不知道如何进行这种链接，但我推理道，写一个解释器来处理这种简单的语言（它只有设置变量、获取变量和字符串连接的功能）很容易，所以我用C写了一个解释器。所以，具有讽刺意味的是，托尼写了一个Lisp程序（其中有一个小小的C例程），因为他是一个C程序员，而我写了一个C程序，因为我是一个Lisp程序员。

最后，我们俩都完成了我们的论文（[Tony](http://www.eecs.berkeley.edu/Pubs/TechRpts/1985/6081.html)，[Peter](http://www.eecs.berkeley.edu/Pubs/TechRpts/1987/5995.html)）。

## 全部内容

整个程序在这里：[lis.py](lis.py)。

## 进一步阅读

要了解更多关于Scheme的信息，请参考一些优秀的书籍（由[Friedman and Fellesein](http://books.google.com/books?id=xyO-KLexVnMC&lpg=PP1&dq=scheme%20programming%20book&pg=PP1#v=onepage&q&f=false)，[Dybvig](http://books.google.com/books?id=wftS4tj4XFMC&lpg=PA300&dq=scheme%20programming%20book&pg=PP1#v=onepage&q&f=false)，[Queinnec](http://books.google.com/books?id=81mFK8pqh5EC&lpg=PP1&dq=scheme%20programming%20book&pg=PP1#v=onepage&q&f=false)，[Harvey and Wright](http://www.eecs.berkeley.edu/~bh/ss-toc2.html)或者[Sussman and Abelson](http://mitpress.mit.edu/sicp/)撰写），视频（由[Abelson and Sussman](http://groups.csail.mit.edu/mac/classes/6.001/abelson-sussman-lectures/)制作），教程（由[Dorai](http://www.ccs.neu.edu/home/dorai/t-y-scheme/t-y-scheme.html)，[PLT](http://docs.racket-lang.org/quick/index.html)，或者[Neller](http://cs.gettysburg.edu/~tneller/cs341/scheme-intro/index.html)制作），或者[参考手册](http://www.schemers.org/Documents/Standards/R5RS/HTML)。

我还有另一页描述了[Lispy的更高级版本](http://norvig.com/lispy2.html)。

* * *

*[Peter Norvig](http://norvig.com)*

* * *
