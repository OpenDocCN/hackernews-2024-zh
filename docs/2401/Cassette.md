<!--yml

类别：未分类

日期：2024-05-27 14:29:06

-->

# Cassette

> 来源：[https://cassette-lang.com](https://cassette-lang.com)

Cassette 是一种小型的、类似 Lisp 的编程语言。它看起来像这样：

```
import List
import Math
import Canvas
import System

let width = 800,
 height = 480,
 canvas = Canvas.new(width, height)

canvas.text("Lines!", {200, 2})

System.seed(System.time())

def rand-line(i) do
 let x0 = Math.floor(i * width / 100),
 y0 = Math.rand-int(20, height / 10),
 x1 = Math.rand-int(0, width),
 y1 = Math.rand-int(20, height)
 canvas.line({x0, y0}, {x1, y1})
end

List.map(\i -> rand-line(i), List.range(0, 100))
```

我创建 Cassette 是为了作为一种简单的“嬉戏编程”语言。嬉戏编程是为了写而写。它是制作软件 3D 渲染器或 GIF 读取器，即使已经存在更好的实现。它是制作生成艺术程序并用钢笔绘制它们。Cassette 本身就是嬉戏编程——当然还有其他脚本语言可能更适合这些个人项目，但这个是我的。

以下是 Cassette 的一些设计目标：

+   函数式的

+   不可变类型

+   简单优先于效率

+   实现较小

+   依赖较少

这是 Cassette 的一些未来工作：

+   大数支持

+   分代垃圾回收

+   编译器 & 虚拟机优化

+   支持其他后端（WebAssembly、LLVM）

+   解构赋值（v2？）

+   基于模式的函数调度（v2？）

该项目需要 C 语言的构建工具链和 SDL2。源代码可以在[这里](https://git.sr.ht/~zjm/Cassette)找到。

1.  获取项目的依赖项

    +   在 macOS 上使用 Homebrew，运行 `brew install llvm git sdl2 sdl2_ttf`

    +   在 Debian 上，运行 `apt install build-essential clang git libsdl2-dev libsdl2-ttf-dev libfontconfig-dev`

1.  使用 `git clone https://git.sr.ht/~zjm/Cassette` 克隆存储库。

1.  运行 `make` 来构建项目。这将创建可执行文件 `cassette`。

1.  可选地，运行 `make install` 来安装 Cassette 可执行文件。您可以在 Makefile 中设置安装文件夹。

1.  使用 `./cassette test/test.ct` 尝试示例。

1.  编写一个小脚本，然后用 `./cassette script.ct` 运行它。

Cassette 有两种数字类型，整数和浮点数。整数可以用十进制、十六进制或字符表示。普通的中缀算术运算适用于数字，位运算适用于整数。

```
-1

1                 ; decimal integer
0x1F              ; hex integer
$a                ; => 0x61
1.0               ; float

-4                ; => -4
1 + 2             ; => 3
5 * 5             ; => 25
10 / 2            ; => 5.0
12 % 10           ; => 2

0xF0 >> 4         ; => 0x0F (shift right)
0x55 << 1         ; => 0xAA (shift left)
0x37 & 0x0F       ; => 0x07 (bitwise and)
20 ^ 1            ; => 21   (bitwise or)
~4                ; => 3    (bitwise not)
```

Cassette 有符号，代表任意值。`true` 和 `false` 是符号。在布尔运算中，除了 `false` 和 `nil` 外，所有值都为真。比较运算符仅适用于数字，但相等运算符适用于任何类型。`and` 和 `or` 运算符是短路的。

```
:ok
:not_found

3 > 1             ; => true
3 < 3             ; => false
5 <= 4 + 1        ; => true
5 >= 4 + 1        ; => true
:ok == :ok        ; => true
:ok != :ok        ; => false
3 > 1 and 4 == 5  ; => false
3 > 1 or 4 == 5   ; => true
3 >= 0 and 3 < 5  ; => true
nil and :ok       ; => false
:error and :ok    ; => true
```

字符串是 UTF-8 编码的二进制。您可以获取字符串的长度，连接两个二进制，并测试一个字符串或字节是否存在于另一个字符串中。二进制也表示其他任意字节序列，比如文件的内容。

```
"Hello!"
#"Hello!"         ; => 6
#""               ; => 0

"Hi " <> "there"  ; => "Hi there"
"foo" <> "bar"    ; => "foobar"

"ob" in "foobar"  ; => true
$r in "foobar"    ; => true
65 in "Alpha"     ; => true
1000 in "foobar"  ; error: bytes must be between 0–255
```

Cassette 有 Lisp 风格的 cons 对，形成链表。对通过 `|` 运算符形成的对进行操作，可以轻松地将值添加到列表中。`nil` 是空列表。您可以获取列表的长度，连接两个列表，并测试值是否在列表中。可以使用整数索引访问列表。

```
[1, 2, 3]         ; list
[1, 2, 3].2       ; => 3
[1, 2, 3].8       ; error: out of bounds
[:a, x, 42]
nil == []         ; => true

1 | [2, 3, 4]     ; => [1, 2, 3, 4]
3 | nil           ; => [3]
1 | 2 | 3 | nil   ; => [1, 2, 3]

#[1, 2, 3]        ; => 3
#nil              ; => 0
[1, 2] <> [3, 4]  ; => [1, 2, 3, 4]
:ok in [:ok, 3]   ; => true
```

Cassette 具有元组，它们是值的固定长度数组。元组比列表不够灵活，但使用的内存较少，访问速度略快。你可以找到元组的长度，连接两个元组，并测试一个值是否在元组中。可以使用整数索引访问元组。

```
{1, 2, 3}         ; tuple
{1, 2, 3}.2       ; => 3
{1, 2, 3}.8       ; error: out of bounds
#{1, 2, 3}        ; => 3
#{}               ; => 0
{1, 2} <> {3, 4}  ; => {1, 2, 3, 4}
"x" in {"y", "z"} ; => false
```

Cassette 具有映射（又名字典），可以像使用符号键的元组一样编写。映射字面量只能有符号键，但其他函数可以获取和设置其他类型的键。你可以找到映射中键/值对的数量，合并两个映射，并测试映射是否包含键。

```
{x: 3, y: 4}      ; a map with keys `:x` and `:y`
my_map.x          ; => 3
my_map.z          ; => nil

#{x: 1, y: 2}     ; => 2
{x: 1, z: 4} <> {x: 2, y: 3}
 ; => {x: 2, y: 3, z: 4}
:x in {x: 1, y: 2}  ; => true
```

使用 `let` 定义变量。`do` 块可以引入新的作用域，并且可以将一组表达式组合成一个。Cassette 具有词法作用域。变量必须以字母或下划线开头，但后面可以包含任何字符，除了空白字符和这些保留字符：`;,.:()[]{}`。这意味着，如果你想要编写一个中缀表达式，通常必须在运算符周围包含空格以消除与操作数的歧义。

```
let x = 1, y = 2, x-y = 3

print(x - y)     ; prints "-1"
print(x-y)       ; prints "3"

do
 let y = 3, z = 4

 x       ; => 1 (from the parent scope)
 y       ; => 3 (shadows the parent `y`)
 z       ; => 4
end

x         ; => 1
y         ; => 2
z         ; error: undefined variable
```

Cassette 提供了 `if`/`else` 块和 `cond` 块来进行条件判断。`cond` 块将评估每个条件，直到找到一个为真，然后评估该子句。

```
if x == 0 do
 IO.print("Uh oh!")
 :error
else
 :ok
end

cond do
 x > 10    -> :ten
 x > 1     -> :one
 true      -> :less  ; default
end
```

使用反斜杠、参数列表和箭头可以创建 Lambda 表达式。`def` 语法是定义 Lambda 作为变量的语法糖，不同之处在于 `def` 声明的函数具有块作用域，因此可以递归调用。函数调用时需要用括号。

```
let foo = \x -> x + 1
foo(3)            ; => 4

; equivalent, except for scope:
let foo = \x -> x + 1
def foo(x) x + 1

; these produce an error, since `b` isn't defined when the body of `a` is compiled
let a = \x -> b(x * 3),
 b = \x -> a(x / 2)

; these are ok, since `a` and `b` are in scope from the beginning of the block
def a(x) b(x * 3)
def b(x) a(x / 2)
```

Cassette 程序可以拆分为不同的模块，每个模块一个文件。模块可以直接导入到文件的作用域中，或者别名为一个映射。

```
; file "foo.ct"
module Foo

let pi = 3.14
def bar(x) x + 1
```

```
; file "main.ct"
import Foo        ; imported as a map called `Foo`

Foo.bar(3)        ; => 4
Foo.pi            ; => 3.14
```

```
; alternative "main.ct"
import Foo as F   ; imported as a map called `F`

def bar(x) x + 8

bar(3)            ; => 11
F.bar(3)          ; => 4
```

```
; alternative "main.ct"
import Foo as *   ; imported directly into current scope

bar(3)            ; => 4
pi                ; => 3.14
```

关于 Cassette 的更多信息，请查阅其他一些文档。敬请关注未来的文章。
