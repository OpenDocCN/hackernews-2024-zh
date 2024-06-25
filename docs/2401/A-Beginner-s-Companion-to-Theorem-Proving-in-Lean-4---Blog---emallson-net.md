<!--yml

分类：未分类

日期：2024-05-27 14:33:25

-->

# 一个 Lean 4 中的定理证明的初学者同伴 | 博客 | emallson.net

> 来源：[`emallson.net/blog/a-beginners-companion-to-theorem-proving-in-lean/`](https://emallson.net/blog/a-beginners-companion-to-theorem-proving-in-lean/)

今年，我的一个业余项目之一是在[Lean](https://lean-lang.org/)中实现一些基本属性和[Matroids](https://en.wikipedia.org/wiki/Matroid)之间的转换。

我进行此操作的主要资源是[*Lean 4 中的定理证明*](https://lean-lang.org/theorem_proving_in_lean4/title_page.html)（TPiL），这是一个非常详细的关于如何使用 Lean 作为证明助手的指南。虽然很有用，但它也有一些缺陷——正如主要的 Lean 文档一样。这些缺陷似乎大部分被[Zulip 聊天](https://leanprover-community.github.io/archive/)填补了。

这篇文章的标题有点双关：我仍然是一个相对初学者，写的是我打算作为 TPiL 的陪伴；而这也是为 Lean 的初学者准备的，涵盖了我作为一个初学者希望我自己知道的内容。

最后的买方请注意：我并没有真正深入研究 Zulip，也不太了解 Lean 中的良好代码组织和风格。这些可能不是最佳实践，但它们对我有帮助。

* * *

这被组织成小节，涵盖了我在我的业余项目上艰难学到的一些东西。

<details><summary>运行片段</summary>

你可以在 Lean 或 [Lean 网站](https://live.lean-lang.org/) 上运行下面的代码片段，如果你将此前导语粘贴在顶部：

```
import Mathlib.Data.Finset.Basic import Mathlib.Data.Finset.Card 
```</details>

```
example (S T : Finset α) : S ⊆ T -> T ⊆ S -> T = S := by 
 exact? 
```

在我的 Lean 学习早期，最大的痛点之一是找到要使用的定理。[`mathlib`](https://leanprover-community.github.io/mathlib4_docs/foundational_types.html) 文档很棒，但缺乏 [hoogle](https://hoogle.haskell.org/)-风格的搜索。

编辑：显然不是！`LeahNeukirchen` 在[lobste.rs](https://lobste.rs/s/hwjv0f/beginner_s_companion_theorem_proving#c_7rypt5)上指出了我忽略的两个不同的 hoogle 风格搜索引擎：

`rw?`、`simp?`、`apply?` 和 `exact?` 策略在 Lean 环境中为你提供了专门的 hoogle 风格访问，找到满足特定目标的候选定理，并在编辑器 UI 中列出它们。如果使用 LSP，你可以通过代码操作选择其中一个解决方案。

默认情况下，所有目标都针对当前目标，但 `rw?` 和 `simp?` 可以有 `at` 形式：

```
example (S T : Finset α) : S ⊆ T -> T ⊆ S -> T = S := by 
 intro S_subs T_subs rw? at S_subs 
```

TPiL 在技术术语中解释了这些策略的每一个做什么，但我花了一些时间才理解每个在实践中是怎么做的。

重要的是要理解，这些策略都在根本上执行*模式重写*。每个的模式为：

**输入目标:** `a`

**假设：** `a = b` 或 `a ↔ b`

**输出目标:** `b`

`rw` 是这三个中最直接的。它将一个类似 `a = b` 或 `a <-> b` 的定理或假设应用到一个类似 `a` 的目标上，并用 `b` 替换它。

`rw` 与 `apply` 和 `exact` 不同，可以应用于非目标以转换它们。这对将来自不同来源的定理调整到与你的目标一致以证明它们非常有用。

```
example (S T X : Finset α) : S = X -> X = T -> S = T := by
 intro S_eq X_eq rw [S_eq, <-X_eq] 
```

**输入目标：** `a`

**假设：** `a`

**输出目标：** *无目标*

`exact` 接受一个假设，以及任何需要的参数，并用它来解决当前的目标。它简单而有效。

```
example (S T : Finset α) : S ⊆ T -> T ⊆ S -> S = T := by
 intro S_subs T_subs  /- Finset.Subset.antisymm looks like: S ⊆ T -> T ⊆ S -> S = T -/
  /- the S_subs and T_subs parameters leave us with S = T, which matches the goal. -/
 exact Finset.Subset.antisymm S_subs T_subs 
```

**输入目标：** `b`

**假设：** `a → b`

**输出目标：** `a`

`apply` 使用蕴含转换你的目标为不同的目标。如果在当前范围内存在 `a` 的证明，那么它被应用（在这种情况下，你基本上得到 `exact`）。否则，你会得到一个新的目标 `a`。

```
example (S T : Finset α) : S ⊆ T ∧ T ⊆ S := by
  /- Finset.Subset.antisymm_iff.mp looks like S = T -> S ⊆ T ∧ T ⊆ S -/
 apply Finset.Subset.antisymm_iff.mp  /- new goal: S = T -/ 
```

重要的是（非常重要，我将在稍后再次提到）：一个 `a ↔ b` 或 `a = b` 的假设可以转换为 `a → b`（用 `.mp`）或 `b → a`（用 `.mpr`）。

TPiL 在几个地方提到了子目标，并简要讨论了`have`，但我觉得这个区域被忽略了。向你的大目标添加子目标是使更大的证明可处理的极其有用的方法。

第一种（也似乎是最常见的）引入子目标的方法是通过 `have`，它通常有两个目的。首先：你可以引入更小的目标，这些目标更容易证明 —— 有时甚至 Lean 可以自动证明。

```
/- While lean cannot auto-solve the outer statement, it can trivially solve the both steps with `exact?` -/ example [DecidableEq α] (S : Finset α) (e1 e2 : α) : e2 ∈ S → insert e2 (insert e1 S) = (insert e1 S) := by
 intro e2_mem  have : e2 ∈ (insert e1 S) := by exact? exact? 
```

其次，你可以用它来实例化其他定理以便重用：

```
/- (not runnable) -/ have T_card := ind_rank_eq_largest_ind_subset_card ind (insert e2 S) T T_subs T_ind T_max 
```

相比之下，`if` / `else` 通常在需要引入依赖于[排中律](https://lean-lang.org/theorem_proving_in_lean4/axioms_and_computation.html?highlight=propext#the-law-of-the-excluded-middle)的目标时非常有用。

```
/- there is a theorem in Mathlib for this, but we use `have` to create smaller goals to bypass it -/ example [DecidableEq α] (S : Finset α) (e : α) : (insert e S).card ≤ S.card + 1 := by
  if h : e ∈ S then
  have : insert e S = S := Finset.insert_eq_self.mpr h rw [this]
 exact Nat.le_add_right (Finset.card S) 1
  else
  have : S.card + 1 = (insert e S).card := (Finset.card_insert_of_not_mem h).symm rw [this] 
```

注意，这个与像 `not_not` 这样有用的定理依赖于经典逻辑，因此在严格的构造性意义上（据我所知）不是严格的，但`if` / `else` 特别是在证明的实现中组织思想方面是如此有用，以至于我选择大量使用它。

编辑：看起来 Lean 只有在模块中使用了`open Classical`才允许`if` / `else`的非构造性用法。[更多信息](https://lobste.rs/s/hwjv0f/beginner_s_companion_theorem_proving#c_jc4wbc)

TLiP 讨论了处理带点和 `cases _ with` 以及 `match _ with` 的情况，但我不清楚如何处理其他工具（如 `apply Iff.intro`）生成的（带标签的）子目标。

你可以使用 `case <label> => <proof>` 来处理特定命名的证明。

这与`refine`很匹配，它类似于`exact`但会生成（命名或匿名）占位符的子目标：

```
example (S T : Finset α) : S ⊆ T -> T ⊆ S -> S = T := by
 intros refine Finset.Subset.antisymm ?left ?right case left => assumption case right => assumption 
```

正如上文所述，你可以使用点访问符来转换推理规则：

此外，许多操作符公开了有用的实用程序。以 `a > b` 为例：

+   `le` 产生 `b < a`

+   `not_le` 产生 `¬ a <= b`

+   `asymm` 产生 `¬ a < b`

+   `trans_le` 产生 `a <= c -> b < c`

还有许多其他用法。在你已经证明了`a > b`并且需要证明`¬ a <= b`的情况下，这些用法非常方便，这种情况在逆否命题中经常出现。

这些策略都与否定有关。与`rw`/`apply`/`exact`类似，它们做着不同但相似的事情。

**输入状态：**一个假设`h`，你可以从其他假设中证明`¬ h`

**输出状态：**目标被替换为`¬ h`

这在你构建了一个矛盾的假设以证明矛盾时非常有用，但这个矛盾对于 Lean 来说并不明显的情况下。

```
example (a b : ℕ) : a > b -> b > a -> True := by
 intro left right absurd right exact left.asymm 
```

**输入状态：**一对假设`h`和`h'`，它们*明显*地相互矛盾

**输出状态：** *无目标*

`contradiction`在 Lean 明显地将矛盾切除掉时省去了`absurd`的额外步骤。通常，这意味着你有假设`h：a`和`h'：¬ a`。

```
example (a b : ℕ) : a > b -> ¬ a > b -> True := by
 intros contradiction 
```

**输入状态：**一个假设`h`

**输出状态：**目标被替换为`¬ h`，并且`h`被替换为`¬ goal`

与`absurd`/`contradiction`不同，这实际上是在做一些不同的事情：应用[contraposition](https://en.wikipedia.org/wiki/Contraposition)。我包含它主要是因为在 TLiP 中没有提到这种策略，我经常发现`contrapose`在简化涉及否定与`rw [not_not]`组合的目标时很有用。

```
/- a theorem for this exists in Mathlib, but again we're ignoring it -/ example (S : Finset α) : ¬ S.card = 0 -> S ≠ ∅ := by
 intro neq_zero contrapose neq_zero rw [not_ne_iff] at neq_zero rw [not_not, neq_zero] rfl 
```

在传统的具有代数数据类型的语言中，类型大致分为两组：

在 Rust 等语言中，这些是`struct`或元组：

```
struct Foo {
  a: usize;
 b: usize; }   struct Bar(usize, usize); 
```

在 Lean 中，`And`类型，虽然通常写成`A ∧ B`，但实际上是一个看起来像这样的结构：

```
structure And where
  left : α
 right : β 
```

由于这是一个结构，这意味着你可以在假设中解构它：

```
have : a ∧ b := sorry  have 〈 a, b 〉 := this 
```

你可以用两种方式来处理这个目标：

+   实例化结构：`exact 〈 a, b 〉`（关闭目标）

+   转换为子目标：`apply And.intro`（为证明左侧和右侧创建新的目标）

在 Rust 等语言中，这些是`enum`：

```
enum Foo {
 A(usize),
 B(usize) } 
```

在 Lean 中，`Or`类型是一个求和类型。虽然写成`A ∨ B`，但实际上会是这样的形式：

```
inductive Or where | inl : α | inr : β 
```

作为一个求和类型，你可以通过在假设中进行*模式匹配*来处理它：

```
have : a ∨ b := sorry  cases this with | inl a => sorry  | inr b => sorry 
```

但是，在目标中，你通常会做两件事之一：

+   实例化结构的一侧：`exact Or.inr b`（关闭目标）。这与`have`很搭配，用于为`Or`的一侧创建一个子目标。

+   转换为蕴含式：例如`rw [or_iff_not_imp_left]`（将`a ∨ b`转换为`¬ a -> b`，并且给定一个`¬a`的假设将目标改为`b`）

在我的业余项目中给我带来最多麻烦的单一事物是证明涉及*存在量词*的定理。了解 Lean 的构造元素以充分理解存在量词的设计是很重要的，我认为 TPiL 在解释这一点方面做得相当不错。然而，这种解释并没有真正告诉你如何在更复杂的情况下证明存在量词。

首先：存在量化器的类型是`Exist α β`。这是一个乘积类型，类似于`And`，并且可以以类似的方式解构：

```
have : ∃ x : N, x < 4 := sorry  have 〈x, x_lt〉 := this /- note that `x` is actually an N that exists satisfying x_lt! -/ 
```

这也意味着你 *证明* 存在性的选项很像 `And`。主要是：用 `exact 〈x, h〉` 构造它，`x` 是满足 `h` 的某个值。

那么问题来了：对于复杂类型，你怎么得到一个 `x`？答案是：通过方法得到它。`bex` 似乎是它的常见名字。例如，要获取有限集 `S` 的一个元素：

```
example (S : Finset α) : S ≠ ∅ -> ∃ x : α, x ∈ S := by
 intro h  /- before getting an element from S, we have to construct the Nonempty type -/
  have : S.Nonempty := Finset.nonempty_of_ne_empty h exact this.bex 
```

一旦你习惯了这个，它开始变得相当自然。从集合中检索元素通常需要证明它们是非空的（否则可能没有元素可检索！），但这是一个完全不同的类型（在我的情况下是 `Finset.Nonempty`）是不透明的，我很难发现。

上面，我讨论了 `have` 作为引入子目标来解决的手段。在它旁边存在一个语法上类似的工具：`let`。具体地说：

```
have <name> : <assumption> := <proof of assumption> 
```

可以重新表述为：

```
have <name> : <type> := <construction> 
```

这几乎等同于：

```
let <name> : <type> := <construction> 
```

有一个关键区别：`let` 定义是 *透明的*，而 `have` 定义是 *不透明的*。换句话说：如果你希望重写器能够操作术语的构造，请使用 `let`。否则，使用 `have`。

要看到区别，考虑这个例子：

```
example [DecidableEq α] (a b : Finset α) (e : α) : ¬ e ∈ a -> insert e a = b -> b.card = a.card + 1 := by
 intro mem h  let x := insert e a  have : x.card = a.card + 1 := by
 rw [Finset.card_insert_of_not_mem mem] rw [← this, ← h] 
```

如果 `let x` 被替换为 `have x`，那么内部子目标就不再可证明了——`x` 的定义变得不透明。

还有另一个区别：哪些字段是可选的。`have` 只需要 `construction` / `proof` 字段，而 `let` 至少需要 `name` 和 `construction` 字段（类型可以推断）。

我的最后一条注记——也是关于代码组织的唯一注记——是使用小定理。如果你看一下 mathlib 的大部分内容，你会发现小证明：最多几行。当你熟悉库的内部结构时，这显然是有帮助的，但对于读者来说，不太清楚证明是如何构建的。

但不要因此而让你不专注于小证明！有两个 *强有力* 的原因超出模糊的 "代码架构" 考虑范围：

1.  通过 Lean，小证明通常可以通过 `exact?`、`rw?` 或 `simp?`（如果没有更高级的工具像 [`aesop`](https://github.com/JLimperg/aesop)）自动构造，这意味着你需要做更少的工作。

1.  定理查找策略开始在大型证明状态下表现得更差，这使得交互体验变得更糟。

这不是 *太* 大的问题——我的 Matroid 秩函数的次模性证明源自独立谓词，大约有 100 行，并且仍然至少是 *可用的*——但肯定是用户体验的一个因素，与良好实践相一致：小定理，像小函数一样，通常更容易处理，而这与 Lean 交互系统的实际性能相一致。

总的来说，通过 Lean 中的这些证明对我来说是一次非常有教益的经验。相对来说，我是一个证明助手的初学者（在研究生期间只轻微地尝试过 Emacs 的[证明通用工具](https://proofgeneral.github.io/) + [Coq](https://coq.inria.fr/)），LSP 和其他工具的质量是巨大的优势。我希望在我更积极地研究理论时存在这样的工具，因为它确实帮助我发现了我做的纸上证明中存在的隐藏假设。

话虽如此：对我来说，它绝对不是纸上工作的替代品。Lean 鼓励小的转换程度使我以一种非常直接的方式错过了森林。它在理论验证中很有用，但不能替代坐下来用笔、纸和茶来证明的过程。

希望这些笔记对其他人想要将 Lean 作为证明助手的人有所帮助（也许，对于不那么致力于建构理论并且更愿意接受诸如`if`/`else`和`not_not`等实际细节的人也是如此）。
