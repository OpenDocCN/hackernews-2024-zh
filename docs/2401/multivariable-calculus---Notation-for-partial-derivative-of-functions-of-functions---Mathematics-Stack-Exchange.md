<!--yml

category: 未分类

日期：2024-05-27 14:58:13

-->

# 多元微积分 - 函数的偏导数的符号 - 数学堆栈交换

> 来源：[https://math.stackexchange.com/questions/3266639/notation-for-partial-derivative-of-functions-of-functions](https://math.stackexchange.com/questions/3266639/notation-for-partial-derivative-of-functions-of-functions)

这是一个较长的答案，我最终会直接回答你的问题，但首先，有几个需要解决的初步问题。

你需要克服的最大障碍是理解莱布尼茨符号的真正含义。莱布尼茨符号有时非常方便，但必须谨慎使用，因为很多时候，它在分母中引入了无关的字母，而且往往，人们想要写的与实际写的不同。这是因为在评估函数在其定义域的特定点时，很容易混淆函数（将定义域的元素映射到目标空间的规则）与函数值（在目标空间的元素）之间的区别。让我们从一个简单的单变量示例开始。

* * *

首先声明的是，“实值变量的实值函数”应该只用一个字母表示，例如$f$，而我们应该说的是

> "函数$f: \Bbb{R} \to \Bbb{R}$"

然而，我假设你经常被教导/一直使用术语：

> "the (real-valued) function $f(x)$ (of a real variable $x$) "

这在最初学习时听起来可能是个好主意，但你很快就会遇到各种符号和术语相关的困惑。要想知道为什么第二种表达方式是不好的，请问自己：“$x$有什么特殊之处？” $f(x)$是否意味着一个函数，而$f(y)$则意味着完全不同的函数？$f(\xi)$是否意味着不同的函数？$f(\eta)$呢？显然，使用括号$()$内的字母影响一个概念的数学含义的想法是荒谬的... ***函数的概念不应取决于你喜欢的字母***！

所以，正确思考这个问题的方式是，$f: \Bbb{R} \to \Bbb{R}$表示函数，而$f(x)$是**当你在点$x$上评估函数$f$时得到的特定实数**。

通常，如果$f: \Bbb{R} \to \Bbb{R}$是一个可微函数，$x \in \Bbb{R}$，那么按照拉格朗日的表示法，人们会将$f$的导数在点$x$处表示为$f'(x)$。这是概念上最清晰的表示法。在莱布尼茨的表示法中，你经常会看到类似于$\dfrac{df}{dx}$，或者$\dfrac{df}{dx}\bigg|_x$，或者天啊，一个说法是“设$y=f(x)$是一个函数，那么在$x$处的$y$的导数是$\dfrac{dy}{dx}(x)$”或类似的其他说法。这都是不好的（从概念角度来看），因为如果我想在原点$0$处求导，那么我就得写出类似于$\dfrac{df}{dx}\bigg|_{x=0}$，或者$\dfrac{df}{dx}(0)$，或者$\dfrac{dy}{dx}\bigg|_{x=0}$或$\dfrac{df(x)}{dx}\bigg|_{x=0}$或$\dfrac{dy(x)}{dx}\bigg|_{x=0}$或其他类似的奇怪符号。这是不好的表示法（有时很方便，但在教学上非常误导人），因为它**引入了一个无关紧要的字母**$x$在分母中，这个字母根本没有任何意义！人们可能会再次问：“$x$有什么特别之处吗？如果我更喜欢$\xi$怎么办？”然后就可以写出$\dfrac{df}{d\xi}\bigg|_{\xi=0}$。仅仅是因为几百年的传统/约定，人们才将$x$用作“自变量”和$y$用作“因变量”……但显然，字母的选择**不应该**影响数学陈述的含义。在拉格朗日的表示法中，当在原点求导时我们只需写$f'(0)$。这很清晰，不会引入额外的无关字母。

当我们开始讨论链式法则时情况会变得更糟。在拉格朗日的表述中，链式法则会说类似于：

> 如果$f,g: \Bbb{R} \to \Bbb{R}$是可微函数，那么复合函数$f \circ g$也是可微的，并且对于每一个$x \in \text{domain}(g) = \Bbb{R}$，我们有\begin{equation} (f \circ g)'(x)= f'(g(x)) \cdot g'(x) \end{equation}

刚开始学习时这可能看起来很抽象，但实际上这是最清晰的表述，它避免了在处理更复杂的对象时出现很多混乱。

在莱布尼茨的表示法中，你可能会看到这样的陈述：

> 如果$f(y)$和$g(x)$是实值函数的实变量，则\begin{equation} \dfrac{d f(g(x))}{dx}(x) = \dfrac{df}{dy}(g(x)) \cdot \dfrac{dg}{dx}(x) \end{equation}（这仍然不太糟糕，因为我特意包含了评估点以使其更准确；但这仍然不好，因为分母中不必要地使用了$x$和$y$）

最糟糕的情况是这样一种说法：

> 如果$z= z(y)$和$y=y(x)$是实值函数关于实变量的函数，则\begin{equation} \dfrac{d z}{dx} = \dfrac{dz}{dy} \cdot \dfrac{dy}{dx} \end{equation}

这最后一种表达方式最容易记住，因为$dy$会"消除"，一切看起来都很顺利。但从符号/逻辑的角度来看，这完全是无稽之谈，因为它引入了无关的字母$x$和$y$，完全抑制了导数被计算的地方。而且，等号两边的$z$代表的含义不同…因为等号左边的$z$怎么可能是$x$的"函数"，而右边的$z$又是$y$的"函数"呢？这种说法完全是胡说八道。只有在简单情况下，新手才能直接应用这条规则，并得到正确的答案。第三种表达中出现的等号$=$实际上意味着"只有读者正确解释一切时才相等"（随着学习的内容越来越复杂，这变得越来越难做到…所以在你真正知道自己在做什么之前，最好避免这种情况）。

* * *

现在，我已经在单变量情况下对符号做了一番苛刻的批评，让我们来看看当我们有两个变量的实值函数时会发生什么…更明确地说，我们有一个函数$f: \Bbb{R}^2 \to \Bbb{R}$。在单变量情况下，我为使用$f'(\cdot)$符号而不是$\dfrac{df}{dx}$或其他符号进行了辩护。在这里，我会做同样的事情。通常，偏导数用下面的符号表示 \begin{equation} \dfrac{\partial f}{\partial x} \quad \text{和} \quad \dfrac{\partial f}{\partial y} \end{equation} 或 \begin{equation} \dfrac{\partial f(x,y)}{\partial x} \quad \text{和} \quad \dfrac{\partial f(x,y)}{\partial y} \end{equation} 或 \begin{equation} \dfrac{\partial f}{\partial x} \bigg|_{(x,y)} \quad \text{和} \quad \dfrac{\partial f}{\partial y}\bigg|_{(x,y)} \end{equation} 或以上符号的某种组合。再次强调，这是非常糟糕的符号，因为，选择字母$x,y$有什么特别之处？为什么不选择希腊字母$\xi,\eta$或甚至希腊字母和英文字母混合的形式，比如$\alpha,b$？那么为什么不像这样：

\begin{equation} \dfrac{\partial f}{\partial \alpha} \quad \text{和} \quad \dfrac{\partial f}{\partial b}? \end{equation}

我并不是说你应该因为"你能够"就去惹人讨厌，搞乱你的语言/符号。我只是指出了莱布尼兹符号中固有的逻辑缺陷。

更好的表示法应该是 \begin{equation} (\partial_1f)(x,y) \quad \text{和} \quad (\partial_2f)(x,y) \end{equation} 来表示**在特定点$(x,y)$求偏导数$f$的部分导数**。这里的下标$1$和$2$确实有意义，因为它们告诉你你正在变化函数$f$的哪个参数。为了明确起见，$(\partial_1f)(\alpha,y)$的含义是"计算当你变化$f$的第一个参数时的偏导数，然后在点$(\alpha,y)$处求值"。所以就极限而言，它等于 \begin{equation} (\partial_1f)(\alpha,y) = \lim_{h \to 0} \dfrac{f(\alpha+h, y) - f(\alpha,y)}{h} \end{equation}

在这里，$\partial_1f$ 是一个从 $\Bbb{R}^2$ 到 $\Bbb{R}$ 的函数；因此，我们会将其写成 $\partial_1f: \Bbb{R}^2 \to \Bbb{R}$，而 $(\partial_1f)(\alpha, y)$ 则是如果我们计算上面的极限得到的特定实数。

* * *

我真的希望你能够意识到关于函数是什么以及在其定义域的点上求值时函数的值是什么，以及在任一情况下使用正确的符号的重要性。现在，我们可以开始剖析你的问题并正确地表达它。

你一开始说

> 假设我们有一个函数 $f(x(t),t)$...

我明白你想表达什么，但为了明确起见，我想指出一些事情。你不只是有一个单一的函数。实际上，在这一点上，你有 $3$ 个函数。首先，你有一个函数 $f: \Bbb{R}^2 \to \Bbb{R}$，接下来，你有一个函数 $x: \Bbb{R} \to \Bbb{R}$，最后，**你通过复合定义了一个新的函数**，规则如下：你定义了一个函数 $g: \Bbb{R} \to \Bbb{R}$，如下所示： \begin{equation} g(t) := f(x(t),t) \end{equation}

因此，你有 $3$ 个函数 $f,x,g$，你应该清楚它们各自的作用。

接下来，你说

> ... 并且我们对 $f(x(t),t)$ 关于 $t$ 的偏导数进行求导 \begin{equation} \frac{\partial f(x(t),t)}{\partial t} \end{equation} 我们会保持 $x(t)$ 不变吗？

你使用的符号非常糟糕，这可能是你感到困惑的原因。正如你在评论中提到的，你不能同时改变 $t$ 并保持 $x(t)$ 不变（除非 $x$ 是一个常数函数）。符号 \begin{equation} \frac{\partial f(x(t),t)}{\partial t} \end{equation} 实际上是无意义的。（我并不是要不尊重，我只是想指出这样的表达没有意义，因为不清楚你的意思）

你上面写的内容有两种可能的解释。在正确的表示法中，它们分别是：\begin{equation} (\partial_2f)(x(t),t) \quad \text{和} \quad g'(t). \end{equation} 在第一种情况中，这意味着你计算 $f$ 关于第二个参数的偏导数，即计算函数 $\partial_2f: \Bbb{R}^2 \to \Bbb{R}$，然后在数字元组 $(x(t),t)$ 上评估该函数。在第二种情况中，你正在计算 $g$ 的导数，即计算函数 $g': \Bbb{R} \to \Bbb{R}$，然后在特定的实数 $t$ 上评估。**一般来说，这两者不相同！** 如果我们明确地写成极限的形式，我们有：\begin{equation} (\partial_2f)(x(t),t) := \lim_{h \to 0} \dfrac{f(x(t),t+h) - f(x(t),t)}{h} \end{equation} 而 \begin{align} g'(t) &:= \lim_{h \to 0} \dfrac{g(t+h) - g(t)}{h} \\ &:= \lim_{h \to 0} \dfrac{f \left(x(t+h), t+h \right) - f(x(t),t)}{h} \end{align}

这应该清楚地表明一般情况下两者是不同的。

现在，$g'$ 和偏导数 $\partial_1f$ 和 $\partial_2f$ 之间的关系是由链式法则得到的。我们有： \begin{align} g'(t) = \left[ \left( \partial_1f \right)(x(t),t) \right] \cdot x'(t) + \left( \partial_2f \right)(x(t),t) \end{align}

这是写事物的符号精确的方式，因为它避免引入分母中不必要的字母，不使用相同的字母用于两个不同的目的，而且，它明确说明了所有导数的评估位置。一个人可能会看到以下等式：

> \begin{equation} \dfrac{df(x(t),t)}{dt} \bigg|_{t} = \dfrac{\partial f}{\partial x} \bigg|_{(x(t),t)} \cdot \dfrac{dx}{dt} \bigg|_{t} + \dfrac{\partial f}{\partial t} \bigg|_{(x(t),t)} \end{equation} （这仍然不够精确，因为在分母中无关的使用了 $x$ 和 $t$，但这是使用莱布尼兹符号的最精确方法了）

你甚至可能会遇到这样的说法

> \begin{equation} \dfrac{df}{dt} = \dfrac{\partial f}{\partial x} \cdot \dfrac{dx}{dt} + \dfrac{\partial f}{\partial t}, \end{equation}

这种表示方法绝对糟糕，因为左边的 $f$ 和右边的 $f$ 完全意味着不同的事情。（这就是我在开始时所说的“人们意图写的与实际写的不同”的意思。）

现在，让我们来看看你的具体示例；我将用精确的表示方法写出导数。（顺便说一句，我认为你在计算上犯了一些错误）

请记住，正如我上面提到的，不只是一个函数涉及其中。有 $3$ 个不同的函数。在你的特定示例中，它们是：

+   $f: \Bbb{R}^2 \to \Bbb{R}$，定义为 $f(\xi, \eta) = \xi^2 + \eta$

+   $x: \Bbb{R} \to \Bbb{R}$，定义为 $x(t) = t^2$

+   $g: \Bbb{R} \to \Bbb{R}$，定义为 $g(t) = f(x(t),t) = f(t^2,t) = (t^2)^2 + t = t^4 + t$

我故意使用 $\xi,\eta$，而不是写成 $f(x,t) = x^2 + t$，因为这样我们将在两个地方使用具有不同含义的 $x$；作为 $f$ 的第一个参数，还有作为一个函数（严格来说，写成 $f(x,t) = x^2 + t$ 是有效的，但我使用 $\xi,\eta$ 只是为了避免潜在的混淆）。

使用精确的符号表示，导数如下： \begin{align} (\partial_1f)(\xi,\eta) &= 2\xi \qquad (\partial_2f)(\xi,\eta) = 1 \\\\ x'(t)&= 2t \\\\ g'(t) &= 4t^3 + 1 \end{align}

（再次注意函数是什么，以及它在哪里被评估）。

还要注意： \begin{align} \left[ \left( \partial_1f \right)(x(t),t) \right] \cdot x'(t) + \left( \partial_2f \right)(x(t),t) &= [2x(t)] \cdot 2t + 1 \\ &= [2t^2] \cdot 2t + 1 \\ &= 4t^3 + 1 \\ &= g'(t), \end{align} 所以我们已经明确验证了链式法则在这种情况下的工作。

再次，直接回答你的问题，你说：

> 我认为 $\frac{\partial f(x(t),t)}{\partial x(t)}=2x(t)$ 但 $\frac{\partial f(x(t),t)}{\partial t}=4t+1 $ 还是 $1$？

这个表示极度糟糕，将 $x(t)$ 放在分母中会让人错误地认为你在某种程度上对 "函数" $x(t)$ 进行微分，而保持 $t$ 固定... 但是然后你可能会困惑 $t$ 怎么可能是固定的，如果 $x(t)$ 在变化... 或者类似的情况。我的观点是糟糕的表示法会导致误解。所以，在这种情况下，正确的写法应该是：\begin{align} \begin{cases} (\partial_1f)(x(t),t) &= 2 x(t) = 2t^2 \\\\ (\partial_2f)(x(t),t) &= 1 \\\\ g'(t) &= 4t^3 + 1 \qquad \text{(you wrote $4t$ rather than $4t^3$ which is wrong)} \end{cases} \end{align}

* * *

如果你理解了我一直试图强调的内容，你应该能够找到剩下问题的答案，但为了完整起见，这里是：

你问：

> 为了进一步理解，以下陈述是否正确：

$$ \frac{\partial f(x(t),t)}{\partial t}=\frac{df(x(t),t)}{dt}$$

因为 $f(x(t),t)=f(t)$ 在你用 $x(t)$ 代入 $f(x(t),t)$ 之后简化它之后成立吗？

立即的回答是"你写的是模糊的"。正确的关系是我上面写的链式法则（从精确到粗糙的三个版本）\begin{align} g'(t) = \left[ \left( \partial_1f \right)(x(t),t) \right] \cdot x'(t) + \left( \partial_2f \right)(x(t),t) \tag{most precise} \end{align}

\begin{align} \dfrac{df(x(t),t)}{dt} \bigg|_{t} = \dfrac{\partial f}{\partial x} \bigg|_{(x(t),t)} \cdot \dfrac{dx}{dt} \bigg|_{t} + \dfrac{\partial f}{\partial t} \bigg|_{(x(t),t)} \tag{medium} \end{align}

\begin{align} \dfrac{df}{dt}= \dfrac{\partial f}{\partial x} \cdot \dfrac{dx}{dt} + \dfrac{\partial f}{\partial t} \tag{very sloppy} \end{align}

此外，请永远不要说像 "$f(x(t),t) = f(t)...$ " 这样的话，这没有任何意义。我知道你的意思是在将 $x(t)$ 替换为 $f$ 的第一个参数，并将 $t$ 替换为 $f$ 的第二个参数之后，你只剩下了一个仅依赖于 $t$ 的东西。但是在等号两边使用 $f(t)$ 的表示方法是糟糕的表示法。像我上面那样使用一个不同的字母，比如 $g$，来指定组合是什么。否则，这就引出了一个问题，即 "为什么 $f$ 在等式的左边有两个参数，而在右边只有一个参数？"

此外，正是对等号两边都误用字母 $f$ 造成了你的困惑，因此你说

> 但如果这是正确的，在上面的例子中，$\frac{\partial f(x(t),t)}{\partial x(t)}=0$，因为 $f(x(t),t)=f(t)$ 实际上并不依赖于 $x(t)$。

再次强调，如我上面所述，我们有 $(\partial_1f)(x(t),t) = 2x(t) = 2t^2$（当且仅当 $t=0$ 时为 $0$）。

* * *

**最终说明：**

我希望现在你能够理解为什么了解你的符号意味着什么，以及区分像$f,g,x$和$\partial_1f, \partial_2f, g', x'$这样的函数，它们是**将域中的每个元素映射到目标空间的唯一元素的规则**，以及像$f(\xi,\eta),f(x(t),t), g(t), x(t)$这样的函数**值**，它们是函数的目标空间的**元素**。

另外，我觉得有必要说以下内容。我的整个回答都批评了莱布尼茨的导数和偏导数符号，因为它非常粗糙，可能会导致各种混乱（参见任何一本普通的数学/物理书籍，特别是热力学中的所有变体的符号/逻辑荒谬），我已经给出了几个理由说明为什么应该避免使用它。然而，话虽如此，我经常使用莱布尼茨的符号，但这仅仅是因为我（通常）知道如何将不精确的符号“翻译”成完全精确的形式，而且我（通常）理解与符号相关的陷阱。所以，在你能够从精确符号“翻译”为不精确符号之前，我建议你坚持使用精确符号，直到所有基本概念都清楚，然后才滥用符号。虽然这在开始时可能会有点乏味，但为了避免愚蠢的混淆，这绝对是值得的。
