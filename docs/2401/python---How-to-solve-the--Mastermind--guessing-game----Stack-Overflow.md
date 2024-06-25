<!--yml

类别：未分类

日期：2024-05-27 14:45:07

-->

# python - 如何解决 "Mastermind" 猜谜游戏？ - Stack Overflow

> 来源：[`stackoverflow.com/questions/1185634/how-to-solve-the-mastermind-guessing-game`](https://stackoverflow.com/questions/1185634/how-to-solve-the-mastermind-guessing-game)

关键工具：熵、贪婪度、分支和界限；Python、生成器、itertools、装饰-取消装饰模式

在回答这个问题时，我想要建立一个有用的函数语言来探索这个问题。我将会逐个介绍这些函数，描述它们和它们的意图。最初，这些函数有着详尽的文档，带有小的内嵌单元测试，使用 doctest 进行测试；我非常赞赏这种作为实现测试驱动开发的绝佳方式。然而，它在 StackOverflow 上表现不佳，所以我不会以这种方式呈现它。

首先，我将需要几个标准模块和**未来**导入（我使用 Python 2.6）。

```
from __future__ import division # No need to cast to float when dividing
import collections, itertools, math 
```

我需要一个评分函数。最初，这个函数返回一个元组（blacks, whites），但我发现如果使用一个命名元组输出会更清晰：

```
Pegs = collections.namedtuple('Pegs', 'black white')
def mastermindScore(g1,g2):
  matching = len(set(g1) & set(g2))
  blacks = sum(1 for v1, v2 in itertools.izip(g1,g2) if v1 == v2)
  return Pegs(blacks, matching-blacks) 
```

为了使我的解决方案更通用，我将特定于 Mastermind 问题的任何东西作为关键字参数传递。因此，我编写了一个函数来创建这些参数一次，并使用**kwargs 语法来传递它。这也使我可以很容易地在以后需要时添加新的属性。请注意，我允许猜测中包含重复项，但限制对手选择不同的颜色；要更改这一点，我只需更改下面的 G。（如果我想允许对手的秘密中有重复项，我还需要更改评分函数。）

```
def mastermind(colours, holes):
  return dict(
    G           = set(itertools.product(colours,repeat=holes)),
    V           = set(itertools.permutations(colours, holes)),
    score       = mastermindScore,
    endstates   = (Pegs(holes, 0),))

def mediumGame():
    return mastermind(("Yellow", "Blue", "Green", "Red", "Orange", "Purple"), 4) 
```

有时我会需要根据应用于集合中每个元素的函数的结果来*分割*集合。例如，数字 1..10 可以通过函数 n % 2（奇数返回 1，偶数返回 0）分割为奇数和偶数。以下函数返回这样一个分割，实现为从函数调用结果到给出该结果的元素集的映射（例如{ 0: 偶数, 1: 奇数 }）。

```
def partition(S, func, *args, **kwargs):
  partition = collections.defaultdict(set)
  for v in S: partition[func(v, *args, **kwargs)].add(v)
  return partition 
```

我决定探索一种使用*贪婪熵方法*的求解器。在每一步，它计算每个可能的猜测可能获得的信息，并选择最具信息量的猜测。随着可能性的增加，这将不利地扩展（二次），但让我们试试吧！首先，我需要一种方法来计算一组概率的熵（信息）。这只是 -∑p log p。然而，为了方便起见，我将允许未标准化的输入，即不加总到 1：

```
def entropy(P):
  total = sum(P)
  return -sum(p*math.log(p, 2) for p in (v/total for v in P if v)) 
```

那么我要如何使用这个函数呢？嗯，对于一组给定的可能性 V 和一个给定的猜测 g，我们从那个猜测中得到的信息只能来自于评分函数：更具体地说，评分函数如何将我们的可能性集合划分。我们希望做出一个最能区分剩余可能性的猜测——将它们分成最多的小集合——因为这意味着我们离答案更近了。这正是上面的熵函数要给出的一个数字：一个大量的小集合得分比一个小量的大集合高。我们需要做的就是将其插入。

```
def decisionEntropy(V, g, score):
  return entropy(collections.Counter(score(gi, g) for gi in V).values()) 
```

当然，在任何给定的步骤中，我们实际上会有一组剩余的可能性 V 和我们可以做出的可能猜测集合 G，并且我们需要选择最大化熵的猜测。此外，如果有几个猜测具有相同的熵，则首选那些也可能是有效解决方案的猜测；这保证了该方法将终止。我使用了标准的 Python 装饰-取消装饰模式以及内置的 max 方法来实现这一点：

```
def bestDecision(V, G, score):
  return max((decisionEntropy(V, g, score), g in V, g) for g in G)[2] 
```

现在我所要做的就是重复调用此函数，直到猜测出正确的结果。我尝试了几种这种算法的实现，直到找到一种看起来合适的方法。我的几个函数会以不同的方式处理这个问题：有些会列举所有可能的决策序列（每个猜测对手可能做出的一个），而其他一些则只对树中的一个路径感兴趣（如果对手已经选择了一个秘密，我们只是尝试达到解决方案）。我的解决方案是一个“懒树”，树的每个部分都是一个可以评估或不评估的生成器，允许用户避免他们不需要的昂贵计算。我还使用了另外两个命名元组，以便代码更清晰。

```
Node = collections.namedtuple('Node', 'decision branches')
Branch = collections.namedtuple('Branch', 'result subtree')
def lazySolutionTree(G, V, score, endstates, **kwargs):
  decision = bestDecision(V, G, score)
  branches = (Branch(result, None if result in endstates else
                   lazySolutionTree(G, pV, score=score, endstates=endstates))
              for (result, pV) in partition(V, score, decision).iteritems())
  yield Node(decision, branches) # Lazy evaluation 
```

以下函数基于提供的评分函数评估此树的单个路径：

```
def solver(scorer, **kwargs):
  lazyTree = lazySolutionTree(**kwargs)
  steps = []
  while lazyTree is not None:
    t = lazyTree.next() # Evaluate node
    result = scorer(t.decision)
    steps.append((t.decision, result))
    subtrees = [b.subtree for b in t.branches if b.result == result]
    if len(subtrees) == 0:
      raise Exception("No solution possible for given scores")
    lazyTree = subtrees[0]
  assert(result in endstates)
  return steps 
```

现在可以使用这个构建一个交互式的 Mastermind 游戏，用户评分计算机的猜测。通过这样的方式玩耍会发现一些有趣的事情。例如，最具信息量的第一个猜测的形式是（黄色，黄色，蓝色，绿色），而不是（黄色，蓝色，绿色，红色）。使用恰好一半的可用颜色可以获得额外的信息。这也适用于 6 色 3 孔 Mastermind——（黄色，蓝色，绿色）——和 8 色 5 孔 Mastermind——（黄色，黄色，蓝色，绿色，红色）。

但是，仍然有许多问题是交互式求解器难以回答的。例如，贪婪熵法需要的步骤最多是多少？以及有多少个输入需要这么多步骤？为了更容易地回答这些问题，我首先制作了一个简单的函数，将上述懒惰树转换为该树中的路径集，即对于每个可能的秘密，一个猜测和得分列表。

```
def allSolutions(**kwargs):
  def solutions(lazyTree):
    return ((((t.decision, b.result),) + solution
             for t in lazyTree for b in t.branches
             for solution in solutions(b.subtree))
            if lazyTree else ((),))
  return solutions(lazySolutionTree(**kwargs)) 
```

找到最坏情况只是找到最长的解决方案：

```
def worstCaseSolution(**kwargs):
  return max((len(s), s) for s in allSolutions(**kwargs)) [1] 
```

结果表明，这个求解器始终可以在 5 步或更少的步骤中完成。五步！我知道当我小时候玩 Mastermind 时，我经常花的时间比这还长。然而，自从创建了这个求解器并试玩了一下之后，我已经大大提高了我的技术水平，即使在没有时间在每一步计算熵理想猜测的情况下，5 步也是一个可以实现的目标 ;)

求解器需要 5 步的概率有多大？它会在 1 步或 2 步内完成吗？为了找出答案，我创建了另一个简单的小函数来计算解决方案长度分布：

```
def solutionLengthDistribution(**kwargs):
  return collections.Counter(len(s) for s in allSolutions(**kwargs)) 
```

对于允许重复的贪婪熵方法：7 种情况需要 2 步；55 种情况需要 3 步；229 种情况需要 4 步；而 69 种情况需要最多的 5 步。

当然，并不保证贪婪的熵方法最小化了最坏情况下的步数。我通用语言的最后一部分是一个算法，用于决定是否存在*任何*给定最坏情况下的解决方案。这将告诉我们贪婪的熵方法是否是理想的。为此，我采用了分支和界限策略：

```
def solutionExists(maxsteps, G, V, score, **kwargs):
  if len(V) == 1: return True
  partitions = [partition(V, score, g).values() for g in G]
  maxSize = max(len(P) for P in partitions) ** (maxsteps - 2)
  partitions = (P for P in partitions if max(len(s) for s in P) <= maxSize)
  return any(all(solutionExists(maxsteps-1,G,s,score) for l,s in
                 sorted((-len(s), s) for s in P)) for i,P in
             sorted((-entropy(len(s) for s in P), P) for P in partitions)) 
```

这绝对是一个复杂的函数，因此需要更多的解释。第一步是根据猜测后的分数对剩余的解决方案进行分区，就像以前一样，但这次我们不知道我们将要做出什么猜测，所以我们存储所有的分区。现在我们*可能*只是递归进入其中的每一个，有效地枚举可能的决策树的整个宇宙，但这将花费非常长的时间。相反，我观察到，如果在这一点上没有将剩余的解决方案分成多于 n 组的分区，那么在将来的任何一步中也不会有这样的分区。如果我们还剩 k 步，这意味着在我们用尽猜测之前，我们最多可以区分 n^(k-1)个解决方案（在最后一步，我们必须始终猜测正确）。因此，我们可以丢弃任何包含将分数映射到超过这么多解决方案的分区。这是代码的接下来两行。

代码的最后一行使用递归，清晰地使用 Python 的`any`和`all`函数，并首先尝试最高熵的决策，希望在正面案例中最小化运行时间。它还首先递归到分区的最大部分，因为如果决策错误，这部分很可能会迅速失败。再次，我使用标准的装饰-去装饰模式，这次是为了包装 Python 的*sorted*函数。

```
def lowerBoundOnWorstCaseSolution(**kwargs):
  for steps in itertools.count(1):
    if solutionExists(maxsteps=steps, **kwargs):
      return steps 
```

通过反复调用`solutionExists`，并逐步增加步数，我们可以得到在最坏情况下破解**Mastermind**问题所需步数的严格下界：5 步。这种贪婪的熵方法确实是最优的。

出于好奇，我发明了另一个猜谜游戏，我称之为“twoD”。在这个游戏中，你尝试猜测一对数字；在每一步中，你会得到告知你的答案是否正确，你猜测的数字是否至少不少于秘密数字对应的数字，以及数字是否至多不超过。

```
Comparison = collections.namedtuple('Comparison', 'less greater equal')
def twoDScorer(x, y):
  return Comparison(all(r[0] <= r[1] for r in zip(x, y)),
                    all(r[0] >= r[1] for r in zip(x, y)),
                    x == y)
def twoD():
  G = set(itertools.product(xrange(5), repeat=2))
  return dict(G = G, V = G, score = twoDScorer,
              endstates = set(Comparison(True, True, True))) 
```

对于这个游戏，贪婪熵方法的最坏情况是五步，但有一种更好的解决方案可以达到最坏情况下的四步，这证实了我的直觉，即短视的贪婪只是对猜谜游戏偶然理想的。更重要的是，这表明了我的语言有多灵活：所有相同的方法都适用于这个新的猜测游戏，就像对猜谜游戏一样，让我在最小的额外编码下探索其他游戏。

那么性能如何？显然，由于是用 Python 实现的，这段代码不会运行得非常快。我也放弃了一些可能的优化，选择了清晰的代码。

一个简单的优化是观察到，在第一步中，大多数猜测基本上是相同的：（黄，蓝，绿，红）与（蓝，红，绿，黄）或（橙，黄，红，紫）没有什么区别。这极大地减少了我们在第一步需要考虑的猜测数量 — 否则这是游戏中最昂贵的决定。

但是，由于这个问题的运行时间增长率很大，即使进行了这种优化，我也无法解决 8 种颜色、5 个孔洞的猜谜问题。相反，我将算法移植到了 C++，保持了一般结构的不变，并采用了位运算来提高关键内部循环的性能，从而实现了几个数量级的加速。我把这留给读者作为练习 :)

**2018 年补充：** 结果表明，贪婪熵方法对于 8 种颜色、4 个孔洞的猜谜问题也不是最优的，最坏情况下需要 7 步，而存在一种算法最多只需要 6 步！
