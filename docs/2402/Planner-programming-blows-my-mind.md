<!--yml

类别：未分类

日期：2024-05-27 15:01:06

-->

# 规划编程让我震惊

> 来源：[https://www.hillelwayne.com/post/picat/](https://www.hillelwayne.com/post/picat/)

[Picat](http://picat-lang.org/) 是一种研究语言，旨在结合逻辑编程、命令式编程和[约束求解](https://www.hillelwayne.com/post/minizinc/)。我最初学习它是为了帮助安排假期，但很快就发现了它的 `planner` 模块，这是我见过的最迷人的编程模型之一。

首先，简要解释一下 <dfn>逻辑编程</dfn> (LP)。在命令式和函数式编程中，我们接受输入并编写算法以产生输出。在逻辑编程和约束求解中，我们提供一组方程，并找到满足这些关系的赋值。例如：

```
main =>
  Arr = [a, b, c, a],
  X = a,
  member(X, Arr),
  member(Y, Arr),
  X != Y,
  println([X, Y]). 
```

以小写字母开头的非函数标识符是“原子”，或者说是唯一标记。以大写字母开头的标识符是变量。所以 `[a, b, c, a]` 是一个包含四个原子的列表，而 `Arr` 和 `X` 是变量。因此 `Member(X, Arr)` 会如预期地返回真。

有趣的是 `Member(Y, Arr)`。Y 还没有定义！所以 Picat 找到一个值让方程成立。Y 可以是 `a`、`b` 或 `c` 中的任意一个。然后，之后的行使得 Y 不可能是 `a`，所以这将打印出 `ab` 或 `ac`。Picat 甚至可以处理像 `member(a, Z)` 这样的表达式，实例化 Z 为一个列表！

<dfn>计划</dfn> 将所有这些推进了一步：与其找到满足方程的变量赋值，我们找到达到某个最终状态的变量 *变异*。这开启了一些非常酷的可能性。

为了展示这一点，我们将使用 Picat 来解决路径问题。

## 问题

我们在网格上放置一个标记，从 *起点* (0, 0) 开始，并选择另一个坐标作为 *目标*。每一步我们可以朝任意基本方向移动一步，但不能超出网格边界。当标记位于目标坐标时，程序成功。举个小例子：

```
+---+
|   |
| G |
|O  |
+---+ 
```

一个解决方案是先移动到 (1, 0)，然后到 (1, 1)。

要用计划解决这个问题，我们需要提供三样东西：

1.  初始状态 `Start`，包含起始和目标坐标。

1.  一组表示状态转换的 <dfn>动作</dfn> 函数。在 Picat 中，这些函数 *必须* 全部命名为 `action`，并接受四个参数：当前状态、下一个状态、动作名称和成本。我们将在下文看到这一切。

1.  一个名为 `final(S)` 的函数，用于确定 S 是否为最终状态。

一旦我们定义了所有这些内容，我们就可以调用内置的 `best_plan(Start, Plan)`，它将把 `Plan` 赋值为达到最终状态所需的最短步骤序列。

### 我们的第一个实现

```
import planner.
import util.

main =>
   Origin = {0, 0}
   , Goal = {2, 2}
   , Start = {Origin, Goal}
   , best_plan(Start, Plan)
   , println(Plan)
   . 
```

<details><summary>解释</summary>

`main`是Picat程序的默认入口点。在这里，我们只是设置初始状态，调用`best_plan`，并打印`Plan`。`{a, b}`是Picat数组的语法，基本上是一个元组。

Picat体中的每个表达式必须后跟逗号，*除了*最后一个子句，必须后跟句号。这使得移动行变得*非常烦人*。将其用“项目符号”样式写会稍微有所帮助。</details>

因为`final`只需要一个参数，所以我们需要将当前位置和目标存储到该参数中。Picat具有强大的模式匹配功能，所以我们可以直接像这样写：

```
final({Pos, Goal}) => Pos = Goal. 
```

<details><summary>解释</summary>

没有模式匹配，我们必须这样写：

```
final(S) =>
  S = {Pos, Goal}
  , Pos = Goal
  . 
```

如果我们写第二个`final`谓词，计划将在*任一*`final`返回true时成功。</details>

最后，我们需要定义规划器可以采取的动作。在这里我们只需要一个动作。

```
action(From, To, Action, Cost) ?=>
  From = {{Fx, Fy}, Goal}
  , Dir = [{-1, 0}, {1, 0}, {0, -1}, {0, 1}] 
  , member({Dx, Dy}, Dir)                    % (a)
  , Tx = Fx + Dx
  , Ty = Fy + Dy
  , member(Tx, 0..10)                        % (b)
  , member(Ty, 0..10)                        % (b)
  , To = {{Tx, Ty}, Goal}
  , Action = {move, To[1]}
  , Cost = 1
  . 
```

<details><summary>解释</summary>

`From`是初始状态，`To`是下一个状态，`Action`是动作的名称——在本例中是`move`。您可以在动作中存储元数据，我们用它来存储新坐标。

使用`action ?=>`而不是`action =>`使`action`具有回溯能力，我承认我*不完全*理解？我*相当肯定*它意味着如果*此*定义的`action`模式匹配但不能导致可行计划，那么Picat可以尝试*其他*动作定义。这对后续版本将更加重要。

就像前面的介绍示例一样，我们使用`member`来*查找*值（在`(a)`行）和*测试*值（在`(b)`行）。Picat还有一个非赋值谓词`membchk`，它只进行测试。如果我不想展示Picat，我可以用`membchk`来代替测试部分，因为它不能赋值。

`Cost`是动作的“成本”。`best_plan`尝试最小化总成本。将其保留为1意味着计划的成本是总步骤数。</details>

就这样，我们的程序完成了。这是输出：

```
> picat planner1.pi

[{move,{1,0}},{move,{2,0}},{move,{2,1}},{move,{2,2}}] 
```

这有点难以阅读，所以我让Picat输出结构化数据，我可以处理成图片。

```
main =>
  Origin = {0, 0}
  , Goal = {2, 2}
  , Start = {Origin, Goal}
  , best_plan(Start, Path)
- , println(Plan)
+ , printf("Origin: %w\n", Origin)
+ , printf("Goal: %w\n", Goal)
+ , printf("Bounds: {10, 10}\n")
+ , printf("Path: ")
+ , println(join([to_string(A[2]): A in Plan], ", "))
  . 
```

我使用了一个[Raku脚本](src/format_path.raku)来将其可视化。这是我们现在得到的：

```
> raku format_path.raku -bf planner1.pi

+-----+
|     |
|     |
|  G  |
|  •  |
|O••  |
+-----+ 
```

为了展示规划器可以绕过“障碍”，我将添加一个规则，即状态*不能*是某个特定值：

```
 , Tx = Fx + Dx
  , Ty = Fy + Dy
+ , {Tx, Ty} != {2, 1} 
```

```
+-----+
|     |
|     |
| •G  |
| •   |
|O•   |
+-----+ 
```

现在让我们将其注释掉，保留这作为我们当前版本的代码：

<details><summary>代码</summary>

```
import planner.
import util.

main =>
  Origin = {0, 0}
  , Goal = {2, 2}
  , Start = {Origin, Goal}
  , best_plan(Start, Plan)
  % , println(Plan) 
  , printf("Origin: %w\n", Origin)
  , printf("Goal: %w\n", Goal)
  , printf("Bounds: {10, 10}\n")
  , printf("Path: ")
  , println(join([to_string(A[2]):  A in Plan], ", "))
  .

final(S) =>
  S = {Pos, Goal},
  Pos = Goal.

action(From, To, Action, Cost) ?=>
  From = {{Fx, Fy}, Goal}
  , Dir = [{-1, 0}, {1, 0}, {0, -1}, {0, 1}]
  , member({Dx, Dy}, Dir)
  , Tx = Fx + Dx
  , Ty = Fy + Dy
  % , {Tx, Ty} != {2, 1}
  , member(Tx, 0..10)
  , member(Ty, 0..10)
  , To = {{Tx, Ty}, Goal}
  , Action = {move, To[1]}
  , Cost = 1
  . 
```</details>

### 添加多个目标

接下来我会添加多个目标。为了成功，计划者需要按顺序达到每一个目标。我们从对`main`的一个改变开始：

```
main =>
  Origin = {0, 0}
- , Goal = {2, 2}
+ , Goal = [{2, 2}, {3, 4}] 
```

现在目标代表一个按顺序达到的“队列”。然后我们添加一个*新*动作，一旦达到目标，就从我们的队列中移除一个目标。

```
action(From, To, Action, Cost) ?=>
  From = {Pos, Goal}
  , Goal = [Pos|Rest]
  , To = {Pos, Rest}
  , Action = {mark, From[1]}
  , Cost = 1
  . 
```

<details><summary>解释</summary>

`[Head|Tail]`将列表分成第一个元素和其余部分。由于`Pos`在前一行中定义，`Goal = [Pos|Rest]`仅在列表中的第一个目标等于`Pos`时为*真*。然后我们通过将新的目标状态声明为`Rest`来从新状态中删除该目标。

（这就是回溯与`?=>`变得重要的地方：如果我们没有使动作可回溯，Picat将首先匹配`move`而不是`mark`。）</details>

由于我们现在在达到目标时从列表中破坏性地删除目标，所以需要调整`final`：

```
final({Pos, Goal}) =>
- Pos = Goal.
+ Goal = []. 
```

就是这样。我们甚至都不需要更新我们的第一个动作！

```
+-----+
|   G |
|   • |
|  G• |
|  •  |
|O••  |
+-----+ 
```

<details><summary>代码</summary>

```
import planner.
import util.

main =>
  Origin = {0, 0}
  , Goal = [{2, 2}, {3, 4}]
  %, Goal = [{9, 2}, {0, 4}, {9, 6}, {0, 9}]
  , Start = {Origin, Goal}
  , best_plan(Start, Plan)
  , printf("Origin: %w\n", Origin)
  , printf("Goal: %w\n", Goal)
  , printf("Bounds: {10, 10}\n")
  , printf("Path: ")
  , println(join([to_string(A[2]):  A in Plan], ", "))
  .

final({Pos, Goal}) =>
  Goal = [].

action(From, To, Action, Cost) ?=>
  From = {{Fx, Fy}, Goal}
  , Dir = [{-1, 0}, {1, 0}, {0, -1}, {0, 1}]
  , member({Dx, Dy}, Dir)
  , Tx = Fx + Dx
  , Ty = Fy + Dy
  , member(Tx, 0..10)
  , member(Ty, 0..10)
  , To = {{Tx, Ty}, Goal}
  , Action = {move, To[1]}
  , Cost = 1
  .

action(From, To, Action, Cost) ?=>
  From = {Pos, Goal}
  , Goal = [Pos|Rest]
  , To = {Pos, Rest}
  , Action = {mark, From[1]}
  , Cost = 1
  . 
```</details>

### 成本最小化

按顺序完成目标并不总是导致*总路径*最短。

```
main =>
  Origin = {0, 0}
- , Goal = [{2, 2}, {3, 4}]
+ , Goal = [{9, 2}, {0, 4}, {9, 6}, {0, 9}] 
```

```
+----------+
|G         |
|•         |
|•         |
|•••••••••G|
|         •|
|G•••••••••|
|•         |
|•••••••••G|
|         •|
|O•••••••••|
+----------+ 
```

如果我们不关心目标的顺序，只想找到最短路径怎么办？那么我们只需要改变两行代码：

```
action(From, T, Action, Cost) ?=>
  From = {Pos, Goal}
- , Goal = [Pos|Rest]
- , T = {Pos, Rest}
+ , member(Pos, Goal)
+ , T = {Pos, delete(Goal, Pos)}
  , Action = {mark, From[1]}
  , Cost = 1
  . 
```

现在规划器可以删除它经过的任何目标，而不管它在`Goal`列表中的位置如何。因此Picat可以“选择”下一个移动到的目标，以使整体路径长度最小化。

```
+----------+
|G•••••••••|
|•        •|
|•        •|
|•        G|
|•        •|
|G        •|
|•        •|
|•        G|
|•         |
|O         |
+----------+ 
```

最终代码：

<details><summary>代码</summary>

```
import planner.
import util.

main =>
  Origin = {0, 0}
  , Goal = [{9, 2}, {0, 4}, {9, 6}, {0, 9}]
  , Start = {Origin, Goal}
  , best_plan(Start, Plan)
  , printf("Origin: %w\n", Origin)
  , printf("Goal: %w\n", Goal)
  , printf("Bounds: {10, 10}\n")
  , printf("Path: ")
  , println(join([to_string(A[2]):  A in Plan], ", "))
  .

final({Pos, Goal}) =>
  Goal = [].

action(From, To, Action, Cost) ?=>
  From = {{Fx, Fy}, Goal}
  , Dir = [{-1, 0}, {1, 0}, {0, -1}, {0, 1}]
  , member({Dx, Dy}, Dir)
  , Tx = Fx + Dx
  , Ty = Fy + Dy
  , member(Tx, 0..10)
  , member(Ty, 0..10)
  , To = {{Tx, Ty}, Goal}
  , Action = {move, To[1]}
  , Cost = 1
  .

action(From, To, Action, Cost) ?=>
  From = {Pos, Goal}
  , member(Pos, Goal)
  , To = {Pos, delete(Goal, Pos)}
  , Action = {mark, From[1]}
  , Cost = 1 
  . 
```</details>

### 其他变体

Picat支持更多的规划变体：

+   `best_plan(S, Limit, Plan)`将最大成本限制为`Limit` —— 有助于早期失败。

+   对于每个`best_plan`，都有一个`best_plan_nondet`来找到每一个可能的最佳计划。

+   `sequence(P, Action)`根据当前的部分计划限制可能的动作，因此我们可以添加像“你必须在转弯前移动两次”这样的限制。

对我来说最酷的事情是，这种规划与Picat的所有其他功能集成。我很快就编写了一个结合了规划和约束求解的演示。<dfn>分区问题</dfn>是一个NP完全问题，其中你需要将一个数字列表分成两个相等的子集。这个程序接受一个数字列表，并找到具有最大可能等分的子列表：

<details><summary>代码</summary>

```
% got help from http://www.hakank.org/picat/set_partition.pi 
import planner.
import util.
import cp.

main =>
Numbers = [32, 122, 77, 86, 59, 47, 154, 141, 172, 49, 5, 62, 99, 109, 17, 30, 977]
  , if final(Numbers) then
      println("Input already has a partition!")
      , explain_solution(Numbers)
    else
        best_plan(Numbers, Plan)
      , printf("Removed: %w%n",[R: $remove(R, _) in Plan])
      , $remove(Last, FinalState) = Plan[Plan.length]
      , printf("Final: %w%n", FinalState)
      , explain_solution(FinalState)
    end
  .

final(Numbers) => get_solutions(Numbers) != [].

get_solutions(Numbers) = S =>
  X = new_list(Numbers.length)
  , X :: 0..1
  , X[1] #= 0 % symmetry breaking
  , sum(Numbers) #= 2*sum([Numbers[I]*X[I]: I in 1..Numbers.length])
  , S = solve_all([$limit(1)], X) 
  .

action(From, To, Action, Cost) =>
   member(Element, From)
   , To = delete(From, Element)
   , Action = $remove(Element, To)
   , Cost = Element
   .

explain_solution(Numbers) =>
  [Sol] = get_solutions(Numbers)
  , Left =  [Numbers[I]: I in 1..Numbers.length, Sol[I] = 0]
  , Right = [Numbers[I]: I in 1..Numbers.length, Sol[I] = 1]
  , printf("%s=%d%n", join([to_string(N): N in Left], "+"), sum(Left))
  , printf("%s=%d%n", join([to_string(N): N in Right], "+"), sum(Right))
  . 
```</details>

```
Removed: [5,17]
Final: [32,122,77,86,59,47,154,141,172,49,62,99,109,30,977]
32+99+977=1108
122+77+86+59+47+154+141+172+49+62+109+30=1108 
```

所有这些对我来说都是非常令人震惊的。这几乎就像是一个元约束求解器，允许我对有效约束*进行约束*。

## 我应该使用Picat吗？

是否？

我不建议在生产中使用Picat。它是一种研究语言，没有很好的文档或清晰的错误信息。当没有解决问题的计划时，这就是你会得到的情况：

```
*** error(failed,main/0) 
```

但它可以在Windows上运行，这比99%的研究语言都要好。

Picat作为一种“工具包”语言似乎更有用，你学会它来解决特定类别的计算问题，而不是期望维护或分享代码。但在这个领域它真的非常出色！有几个问题是我用普通编程语言和约束求解器难以解决的。Picat以相当优雅的方式解决了许多这样的问题。

## 附录：其他规划语言

虽然最初是为机器人和人工智能而开发，但“规划”通常用于视频游戏人工智能，这被称为“目标导向行动规划”（GOAP）。通常它是作为其他语言的库构建，或者实现为 [自定义搜索策略](https://artint.info/3e/html/ArtInt3e.Ch3.html)。您可以在 [这里](https://web.archive.org/web/20140613121607/http://alumni.media.mit.edu/~jorkin/goap.html) 了解更多关于 GOAP 的信息。

这里还有 [PDDL](https://planning.wiki/guide/whatis/pddl)，一个独立规划者作为输入的规划描述语言，就像 DIMACS 是 [SAT](https://www.hillelwayne.com/post/sudoku/) 的描述格式一样。

*感谢 [Predrag Gruevski](https://predr.ag/) 的反馈。我最初在 [我的新闻简报](https://buttondown.email/hillelwayne/) 上分享了对 Picat 的看法。我每周都会写新的简报帖子。*
