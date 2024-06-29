<!--yml

category: 未分类

date: 2024-05-29 12:39:26

-->

# 用数学解决团队战略问题

> 来源：[https://www.alexirpan.com/2024/03/23/crew-battle.html](https://www.alexirpan.com/2024/03/23/crew-battle.html)

在超级Smash兄弟锦标赛中，偶尔会有一种称为团队战的活动。（它们也出现在其他格斗游戏中，但我主要观看Smash。）两支队伍的玩家在一系列的1v1对战中竞争。在第一场比赛中，每个队伍同时选择一名选手。他们进行比赛，输家淘汰，赢家继续留在场上。然后，输掉的队伍挑选下一个选手出战。淘汰赛重复进行，直到一支队伍所有选手都淘汰完毕。

这些活动总是非常激动人心。人们喜欢团队运动！它们通常还以区域性的方式组织（例如美国对日本，西海岸对东海岸），这可以强调和挑起地区间的竞争。决定派遣谁参战的策略可能很复杂，尽管通常是凭直觉做出的，但我一直想知道什么是最优的策略。

这个问题变得复杂的原因在于格斗游戏角色之间的对战并不完全平衡。有些角色会克制其他角色。有时候玩家在特定的对战中异常出色 - 例如狐狸对狐狸的对战在理论上是50-50，但有些玩家因在狐狸对战中的出色表现而享有盛誉。还有普通玩家的实力如何？通常的经验法则是将最强的玩家（锚点）留到最后，因为最后一名选手面对的心理压力最大。但如果我们忽略了这些因素，这样做真的正确吗？何时派遣第二强的选手，或者最弱的选手？**什么是最优的团队战略？**

## 理论

让我们稍微形式化一下这个问题。为了简单起见，我将忽略“Smash”游戏中的股票，并假设每场比赛是完全的胜利或失败。我预计结论仍然会相似。

Let’s call the teams \(A\) and \(B\). There are \(n\) players on each team, denoted as \(\{a_1, a_2, \cdots, a_n\}\) and \(\{b_1, b_2, \cdots, b_n\}\). Each player has a certain chance of beating each other player which can be described as an \(n \times n\) matrix of probabilities, where row \(i\) column \(j\) is the probability \(a_i\) beats \(b_j\). We’ll denote that as \(\Pr(a_i > b_j)\). This matrix doesn’t need to be symmetric, or have its rows or columns sum to 1.

\[\begin{bmatrix} \Pr(a_1 > b_1) & \Pr(a_1 > b_2) & \cdots & \Pr(a_1 > b_n) \\ \Pr(a_2 > b_1) & \Pr(a_2 > b_2) & \cdots & \Pr(a_2 > b_n) \\ \vdots & \vdots & \ddots & \vdots \\ \Pr(a_n > b_1) & \Pr(a_n > b_2) & \cdots & \Pr(a_n > b_n) \end{bmatrix}\]

我们称之为**对战矩阵**。每个团队都知道这个对战矩阵，并且其目标是最大化团队的获胜概率。为了具体化，我们可以举例说明一个石头剪刀布的团队战。每个团队有3名选手：一个出石头、一个出布、一个出剪刀。这将得到以下的对战矩阵。

\[\begin{array}{c|ccc} & \textbf{石头} & \textbf{纸} & \textbf{剪刀} \\ \hline \textbf{石头} & 0.5 & 0 & 1 \\ \textbf{纸} & 1 & 0.5 & 0 \\ \textbf{剪刀} & 0 & 1 & 0.5 \end{array}\]

在 RPS 游戏中，平局会再次进行。在团队战斗中没有平局，我们将随机选择一个获胜者，如果两个玩家都一样。这里有一个示例游戏：

```
A picks rock, B picks scissors
A rock beats B scissors
B sends in paper
A rock loses to B paper
A sends in scissors
A scissors beats B paper
B sends in rock
A scissors loses to B rock
A sends in paper
A paper beats B rock
B is out of players - A wins.

```

这是一个相当愚蠢的团队战斗，因为缺乏戏剧性，但我们稍后会回到这个例子。

首先：根据通用对战矩阵，我们应该期望有一个有效的算法来解决团队战斗吗？

我的怀疑是，可能不是。找到最优策略在某种程度上类似于选择每队 \(N\) 名玩家的最佳顺序，这立即在我的脑海中引发了[旅行推销员问题](https://en.wikipedia.org/wiki/Travelling_salesman_problem)的警报。解决团队战斗似乎是解决常规矩阵收益游戏的严格更难版本，快速搜索显示这些通常被认为是难以一般解决的。（如果感兴趣，见[Daskalakis, Goldberg, Papadimitriou 2009](https://people.csail.mit.edu/costis/simplified.pdf)。）

然而，即使没有高效的算法，肯定是有**一个**算法的。让我们定义一个函数 \(f\)，其中 \(f(p, team, S_A, S_B)\) 是在以下情况下 A 队获胜的概率：

+   刚刚获胜的玩家是 \(p\)。

+   决定谁上场的是 \(team\)（即不包括 \(p\) 的队伍）。

+   \(S_A\) 是队伍 A 中剩余的玩家集合，不考虑玩家 \(p\)。

+   \(S_B\) 是队伍 B 中剩余的玩家集合，不考虑玩家 \(p\)。

这样的 \(f\) 可以递归地定义。以下是基本情况。

+   如果 \(team = A\) 并且 \(S_A\) 为空，则 \(f(p, team, S_A, S_B) = 0\)，因为由于 A 队没有玩家了，A 已经输了。

+   如果 \(team = B\) 并且 \(S_B\) 为空，则 \(f(p, team, S_A, S_B) = 1\)，因为由于 B 队没有玩家了，A 队已经赢了。

（一个澄清：团队战斗并不一定在 \(S_A\) 或 \(S_B\) 为空时结束。当一个队伍只剩下最后一个玩家时，他们将没有剩余玩家可派出，但如果他们表现得足够好，他们的最后一个玩家仍然可以击败整个对方队伍。）

以下是递归情况。

\[\begin{align*} f(a_i, B, S_A, S_B) = \min_{j \in S_B} & \Pr(a_i > b_j) f(a_i, B, S_A, S_B - \{b_j\}) \\ & + (1-\Pr(a_i > b_j)) f(b_j, A, S_A, S_B - \{b_j\}) \end{align*}\]

（现在轮到 B 队了。他们想要派出玩家 \(b_j\)，以最小化 A 队获胜的概率。当前的玩家要么保持为 \(a_i\)，要么更换为 \(b_j\)。）

\[\begin{align*} f(b_j, A, S_A, S_B) = \max_{i \in S_A} & \Pr(a_i > b_j) f(a_i, B, S_A - \{a_i\}, S_B) \\ & + (1-\Pr(a_i > b_j)) f(b_j, A, S_A - \{a_i\}, S_B) \end{align*}\]

（反过来，如果轮到 A 队，并且他们想要最大化 A 队获胜的概率。）

Computing this \(f\) can be done with dynamic programming, in \(O(n^22^{2n})\) time. That’s not going to work at big scales, but I only want to study teams of like, 3-5 players, so this is totally doable.

One neat thing about crew battles is that after the first match, it turns into a turn-based perfect information game. Sure, the outcome of each match is random, but once it’s your turn, the opposing team is locked into their player. That means we don’t have to consider strategies that randomly pick among the remaining players - there will be exactly one reply that’s best. And that means the probability of winning the crew battle assuming optimal play is locked in as soon as the first match is known.

This means we can reduce all the crew battle outcomes down to a single \(n \times n\) matrix, which I’ll call the **crew battle matrix**. Let \(C\) be that matrix. Each entry \(C_{ij}\) in the crew battle matrix is the probability that team A wins the crew battle, if in the very first match A sends in \(a_i\) and B send in \(b_j\).

\[\begin{align*} C_{ij} = &\Pr(a_i > b_j) f(a_i, B, S_A - \{a_i\}, S_B - \{b_j\}) \\ &+ (1-\Pr(a_i, b_j)) f(b_j, A, S_A - \{a_i\}, S_B - \{b_j\}) \end{align*}\] \[C = \begin{bmatrix} C_{11} & C_{12} & \cdots & C_{1n} \\ C_{21} & C_{22} & \cdots & C_{2n} \\ \vdots & \vdots & \ddots & \vdots \\ C_{n1} & C_{n2} & \cdots & C_{nn} \end{bmatrix}\]

This turns our original more complicated problem into exactly a 1 round matrix game like [囚徒困境](https://en.wikipedia.org/wiki/Prisoner%27s_dilemma) or [麋鹿狩猎](https://en.wikipedia.org/wiki/Stag_hunt)…assuming we have \(f\). But computing \(f\) by hand is really annoying, and likely does not have a closed form. There’s no way to get \(f\) computed for arbitrary crew battles unless we use code.

## So I Wrote Some Python Code to Compute \(f\) for Arbitrary Crew Battles

Let’s consider the RPS crew battle again. Here’s the matchup matrix:

\[matchup: \begin{array}{c|ccc} & \textbf{rock} & \textbf{paper} & \textbf{scissors} \\ \hline \textbf{rock} & 0.5 & 0 & 1 \\ \textbf{paper} & 1 & 0.5 & 0 \\ \textbf{scissors} & 0 & 1 & 0.5 \end{array}\]

and here’s what my code outputs for the crew battle matrix, assuming optimal play.

\[crew battle: \begin{array}{c|ccc} & \textbf{rock} & \textbf{paper} & \textbf{scissors} \\ \hline \textbf{rock} & 0.5 & 0 & 1 \\ \textbf{paper} & 1 & 0.5 & 0 \\ \textbf{scissors} & 0 & 1 & 0.5 \end{array}\]

No, that’s not a typo. The two are identical! Let’s consider a crew battle of “noisy RPS”, where rock is only favored to beat scissors, rather than 100% to win, and so on.

\[对决：\begin{array}{c|ccc} & \textbf{石头} & \textbf{纸} & \textbf{剪刀} \\ \hline \textbf{石头} & 0.5 & 0.3 & 0.7 \\ \textbf{纸} & 0.7 & 0.5 & 0.3 \\ \textbf{剪刀} & 0.3 & 0.7 & 0.5 \end{array}\] \[团队战：\begin{array}{c|ccc} & \textbf{石头} & \textbf{纸} & \textbf{剪刀} \\ \hline \textbf{石头} & 0.500 & 0.399 & 0.601 \\ \textbf{纸} & 0.601 & 0.500 & 0.399 \\ \textbf{剪刀} & 0.399 & 0.601 & 0.500 \\ \end{array}\]

直觉上，我觉得这使得对原始RPS矩阵的拉向有意义 - 从某种意义上讲，团队战就像是玩3局RPS而不是1局，只是有更复杂的限制。

这是一场样本游戏的记录。

```
Game start, A = rock B = scissors
Win prob for A is 0.601
A = rock beats B = scissors
If B sends in paper: A wins 0.729
If B sends in rock: A wins 0.776
B sends in paper
A = rock beats B = paper
If B sends in rock: A wins 0.895
B sends in rock
A = rock beats B = rock
B has no more players
A wins

```

球队B在这里遭遇了惨败，被石头完全横扫。

为了进一步验证，这里有一个随机的对决矩阵和相应的游戏矩阵。

\[对决：\begin{array}{c|ccc} & \textbf{b}_1 & \textbf{b}_2 & \textbf{b}_3 \\ \hline \textbf{a}_1 & 0.733 & 0.666 & 0.751 \\ \textbf{a}_2 & 0.946 & 0.325 & 0.076 \\ \textbf{a}_3 & 0.886 & 0.903 & 0.089 \\ \end{array}\] \[团队战：\begin{array}{c|ccc} & \textbf{b}_1 & \textbf{b}_2 & \textbf{b}_3 \\ \hline \textbf{a}_1 & 0.449 & 0.459 & 0.757 \\ \textbf{a}_2 & 0.748 & 0.631 & 0.722 \\ \textbf{a}_3 & 0.610 & 0.746 & 0.535 \\ \end{array}\]

好了。让我们尝试一些新的东西。如果所有玩家之间的对决都是传递的怎么样？假设每个玩家有一个特定的力量水平 \(p\)，如果力量水平为 \(p\) 和 \(q\) 的玩家对决，那么力量水平为 \(p\) 的玩家以 \(p/(p+q)\) 的概率获胜。下面是一个遵循此规则的随机对决矩阵。我已经按照 \(a_1\) 和 \(b_1\) 最弱，\(a_3\) 和 \(b_3\) 最强的顺序对玩家进行了排序。由于对决矩阵是A赢得的概率，所以在读取行时值会下降（与更强的B玩家对战），在读取列时值会上升（与更强的A玩家对战）。

\[对决：\begin{array}{c|ccc} & \textbf{b}_1 & \textbf{b}_2 & \textbf{b}_3 \\ \hline \textbf{a}_1 & 0.289 & 0.199 & 0.142 \\ \textbf{a}_2 & 0.585 & 0.462 & 0.365 \\ \textbf{a}_3 & 0.683 & 0.568 & 0.468 \\ \end{array}\] \[团队战：\begin{array}{c|ccc} & \textbf{b}_1 & \textbf{b}_2 & \textbf{b}_3 \\ \hline \textbf{a}_1 & 0.385 & 0.385 & 0.385 \\ \textbf{a}_2 & 0.385 & 0.385 & 0.385 \\ \textbf{a}_3 & 0.385 & 0.385 & 0.385 \\ \end{array}\]

…哎呀。这表明*无论每支队伍先派出谁*都无关紧要。赢得团队战的概率是相同的。用几个随机的3x3矩阵尝试都显示出相同的模式。

或许这只是因为3人团队战很奇怪？我们试试5人团队战。

\[对阵: \begin{array}{c|ccccc} & \textbf{b}_1 & \textbf{b}_2 & \textbf{b}_3 & \textbf{b}_4 & \textbf{b}_5 \\ \hline \textbf{a}_1 & 0.715 & 0.706 & 0.564 & 0.491 & 0.340 \\ \textbf{a}_2 & 0.732 & 0.723 & 0.584 & 0.511 & 0.358 \\ \textbf{a}_3 & 0.790 & 0.782 & 0.659 & 0.591 & 0.435 \\ \textbf{a}_4 & 0.801 & 0.794 & 0.675 & 0.608 & 0.453 \\ \textbf{a}_5 & 0.807 & 0.800 & 0.682 & 0.616 & 0.461 \\ \end{array}\] \[船员战: \begin{array}{c|ccccc} & \textbf{b}_1 & \textbf{b}_2 & \textbf{b}_3 & \textbf{b}_4 & \textbf{b}_5 \\ \hline \textbf{a}_1 & 0.731 & 0.731 & 0.731 & 0.731 & 0.731 \\ \textbf{a}_2 & 0.731 & 0.731 & 0.731 & 0.731 & 0.731 \\ \textbf{a}_3 & 0.731 & 0.731 & 0.731 & 0.731 & 0.731 \\ \textbf{a}_4 & 0.731 & 0.731 & 0.731 & 0.731 & 0.731 \\ \textbf{a}_5 & 0.731 & 0.731 & 0.731 & 0.731 & 0.731 \\ \end{array}\]

同样的情况发生了！让我们玩一个样本游戏。即使无论谁开始，胜利概率都相同，但未来出战选手的选择显然很重要。

```
Game start, A = 1 B = 2
Win prob for A is 0.731
A = 1 beats B = 2
If B sends in 0: A wins 0.804
If B sends in 1: A wins 0.804
If B sends in 3: A wins 0.804
If B sends in 4: A wins 0.804
B sends in 0
A = 1 beats B = 0
If B sends in 1: A wins 0.836
If B sends in 3: A wins 0.836
If B sends in 4: A wins 0.836
B sends in 1
A = 1 loses B = 1
If A sends in 0: A wins 0.759
If A sends in 2: A wins 0.759
If A sends in 3: A wins 0.759
If A sends in 4: A wins 0.759
A sends in 0
A = 0 beats B = 1
If B sends in 4: A wins 0.800
If B sends in 3: A wins 0.800
B sends in 4
A = 0 beats B = 4
If B sends in 3: A wins 0.969
B sends in 3
A = 0 beats B = 3
B has no more players
A wins

```

虽然A赢得船员战的概率会随着决定支持A或B的比赛而变化，但对于任何选择，胜利概率*不会改变*。

我发现这个结果相当令人惊讶！开始之前，我假设会有一些与技能水平差异相关的经验法则。我绝对没想到完全不相关。这个结果取决于我们如何建模技能，即玩家具有技能 \(p\) 的情况下击败具有技能 \(q\) 的玩家的概率为 \(p/(p+q)\)。然而，这个模型非常普遍。它被称为[布拉德利-特里模型](https://en.wikipedia.org/wiki/Bradley%E2%80%93Terry_model)，是Elo评级的基础。（也是根据人类反馈调整AI聊天机器人响应的基础。）

让我们更正式地描述这个观察结果。

**猜想:** 假设您有一个 \(N\) 人船员战，玩家为 \(A = \{a_1, a_2, \cdots, a_N\}\) 和 \(B = \{b_1, b_2, \cdots, b_N\}\)。进一步假设每个玩家都有非负的技能水平 \(p\)，并且每场比赛遵循 \(\Pr(a_i > b_j) = p_{a_i} / (p_{a_i} + p_{b_j})\)。那么，\(A\) 赢得船员战的概率不依赖于策略。

我自己还没有证明这一点，但感觉很容易证明。把它当作一个练习，自己去证明一下。

## 那么...这是什么意思呢？

假设这个猜想是正确的，我相信这意味着**在选择玩家时完全忽略玩家实力，只应关注与技能无关的因素，如角色配对**。在*超级马里奥兄弟梅尔*中，Fox对Puff更有利，因此如果你面对Hungrybox（有史以来最好的Puff玩家），这表明你应该派出Generic Netplay Fox，尽管他们很可能会被击败。相反，它表明船员应该*避免*在Generic Netplay Fox在场时派遣Hungrybox，尽管Hungrybox可能会赢。

这是一个相当极端的结论，我不会盲目地接受它。从心理学角度来看，如果每个人都能做出贡献，对团队士气是有好处的，而让人们在技术水平上有很大差距的对手比赛则会损害团队士气。但是，这也表明团队战略在首要位置并不那么重要！在顶尖水平上，很少看到非常不平衡的角色对决。职业选手倾向于选择强大的角色，这些角色在大多数对手中有良好的对决。鉴于非常不平衡的对决是通过战略获取优势的唯一途径，数学表明队长们实际上对团队战斗的结果没有太多影响力。

如果你的团队输了，你不能把责任推到糟糕的团队战略上。你的团队只是比较差劲而已。接受现实吧，接受失败。
