<!--yml
category: 未分类
date: 2024-05-27 14:58:13
-->

# multivariable calculus - Notation for partial derivative of functions of functions - Mathematics Stack Exchange

> 来源：[https://math.stackexchange.com/questions/3266639/notation-for-partial-derivative-of-functions-of-functions](https://math.stackexchange.com/questions/3266639/notation-for-partial-derivative-of-functions-of-functions)

This is a long answer, and I do eventually answer your question directly, but first, there are several preliminary matters which need to be addressed.

The biggest obstacle you need to overcome is unraveling the true meaning of Leibniz's notation. Leibniz's notation is at times very convenient, but it has to be used with caution, because very very very very often, it introduces irrelevant letters in the denominator, and often times, what people intend to write differs from what they actually write. This is because it is very easy to confuse a function (which is a rule for mapping elements of the domain to the target space), with function values(which are elements of the target space) when the function is evaluated at a particular point of its domain. Let's begin with a simple one variable example.

* * *

The first disclaimer is that a "real valued function of a real variable" should just be denoted by a single letter such as $f$, and we should speak of

> "the function $f: \Bbb{R} \to \Bbb{R}$"

However, I'm assuming you've often been taught/you've been using the terminology:

> "the (real-valued) function $f(x)$ (of a real variable $x$) "

This may sound like a good idea when first learning, but you can quickly run into all sorts of notation and terminology related confusions. To see why the second way of phrasing things is bad, ask yourself: "what's so special about $x$?" Does $f(x)$ mean one function, whereas $f(y)$ means a completely different function? Does $f(\xi)$ mean a different function? What about $f(\eta)$? Clearly the idea that the letter we use inside the brackets $()$ affects the mathematical meaning of a concept is nonsense... ***the concept of a function shouldn't depend on what your favourite letter is***!

So, the proper way to think about this is $f: \Bbb{R} \to \Bbb{R}$ denotes the function, while $f(x)$ is the **particular real number you get, when you evaluate the function $f$ on the point** $x$.

Usually, if $f: \Bbb{R} \to \Bbb{R}$ is a differentiable function, and $x \in \Bbb{R}$, then by Lagrange’s notation, one would write $f'(x)$ for the derivative of $f$ evaluated AT the point $x$. This is the conceptually clearest notation. In Leibniz's notation you often see something like $\dfrac{df}{dx}$, or $\dfrac{df}{dx}\bigg|_x$ or heaven forbid, a statement like "let $y=f(x)$ be a function, then the derivative of $y$ at $x$ is $\dfrac{dy}{dx}(x)$" or any other similar sounding statement. This is all bad (from a conceptual point of view) because if I wanted to evaluate the derivative at the origin $0$, then I would have to write something like $\dfrac{df}{dx}\bigg|_{x=0}$, or $\dfrac{df}{dx}(0)$, or $\dfrac{dy}{dx}\bigg|_{x=0}$ or $\dfrac{df(x)}{dx}\bigg|_{x=0}$ or $\dfrac{dy(x)}{dx}\bigg|_{x=0}$ or some other weird symbol like this. This is bad notation (convenient at times, but pedagogically very misleading), because it **introduces an irrelevant letter** $x$ in the denominator, which has no meaning at all! One could ask again: "what's so special about $x$? what if I like $\xi$ better?" and then one could write $\dfrac{df}{d\xi}\bigg|_{\xi=0}$. It is only because of hundreds of years of tradition/convention that people use $x$ as the "independent variable" and $y$ as the "dependent variable"... but clearly, the choice of letters **SHOULD NOT** affect the meaning of a mathematical statement. In Lagrange's notation, when evaluating the derivative at the origin we'd just write $f'(0)$. This is clear, and doesn't introduce extra irrelevant letters.

Things get much worse when we start talking about the chain rule. In Lagrange's formulation, the chain rule would say something like:

> If $f,g: \Bbb{R} \to \Bbb{R}$ are differentiable functions, then the composition $f \circ g$ is differentiable as well, and for every $x \in \text{domain}(g) = \Bbb{R}$, we have \begin{equation} (f \circ g)'(x)= f'(g(x)) \cdot g'(x) \end{equation}

This may seem abstract when first learning, but it's really the clearest formulation, and it avoids a whole lot of confusions when dealing with more complicated objects.

In Leibniz's notation, one might see a statement like:

> If $f(y)$ and $g(x)$ are real valued functions of a real variable, then \begin{equation} \dfrac{d f(g(x))}{dx}(x) = \dfrac{df}{dy}(g(x)) \cdot \dfrac{dg}{dx}(x) \end{equation} (this is still not too bad, because I purposely included the points of evaluation to make it more accurate; but this is still bad because of the unnecessary use of $x$ and $y$ in the denominators)

The worst is a statement like:

> If $z= z(y)$ and $y=y(x)$ are real valued functions of a real variable, then \begin{equation} \dfrac{d z}{dx} = \dfrac{dz}{dy} \cdot \dfrac{dy}{dx} \end{equation}

This last formulation is the easiest to remember because the $dy$'s "cancel", and everything seems to be nice and dandy. But from a notational/logical point of view, it is utter nonsense, because it introduces irrelevant letters $x$ and $y$, and completely suppresses where the derivatives are being evaluated. Also, the $z$'s mean different things on both sides of the equal sign... because how can $z$ be a "function of $x$" on the LHS, whereas $z$ is a "function of $y$" on the RHS? This kind of statement is just nonsense. It is only in simple cases whereby novices can directly apply this rule, and get correct answers. The equal sign $=$ appearing in the third formulation really means "it is equal if the reader correctly interprets everything" (and this gets harder and harder to do as you learn more complicated stuff... so until you really know what you're doing, just avoid this).

* * *

Now that I have been pedantic about notation in the single variable case, let's go to the case when we have a real-valued function of two variables..., more explicitly, we have a function $f: \Bbb{R}^2 \to \Bbb{R}$. In the single variable case, I made an argument for the use of the $f'(\cdot)$ notation, as opposed to $\dfrac{df}{dx}$ or something else. Here, I'll do the same thing. Often, the partial derivatives are denoted by \begin{equation} \dfrac{\partial f}{\partial x} \quad \text{and} \quad \dfrac{\partial f}{\partial y} \end{equation} or \begin{equation} \dfrac{\partial f(x,y)}{\partial x} \quad \text{and} \quad \dfrac{\partial f(x,y)}{\partial y} \end{equation} or \begin{equation} \dfrac{\partial f}{\partial x} \bigg|_{(x,y)} \quad \text{and} \quad \dfrac{\partial f}{\partial y}\bigg|_{(x,y)} \end{equation} or some combination of the above notations. Once again, this is very bad notation, because, what's so special about the choice of letters $x,y$? Why not Greek letters $\xi,\eta$ or even a mix of Greek and English like $\alpha,b$? So why not something like

\begin{equation} \dfrac{\partial f}{\partial \alpha} \quad \text{and} \quad \dfrac{\partial f}{\partial b}? \end{equation}

I'm not saying that you should annoy people and mix up your languages/notation just because "you CAN". Rather I'm just pointing out the logical flaws which are inherent in Leibniz's notation.

A better notation would be \begin{equation} (\partial_1f)(x,y) \quad \text{and} \quad (\partial_2f)(x,y) \end{equation} to mean the partial derivatives **of the function $f$ evaluated at a particular point $(x,y)$**. The subscripts $1$ and $2$ in $\partial_1f$ and $\partial_2f$ are indeed meaningful here because they tell you which argument of the function $f$ you are varying. Just to be explicit, $(\partial_1f)(\alpha,y)$ means "compute the partial derivative when you vary the first argument of $f$, then evaluate this on the point $(\alpha,y)$". So in terms of limits it equals \begin{equation} (\partial_1f)(\alpha,y) = \lim_{h \to 0} \dfrac{f(\alpha+h, y) - f(\alpha,y)}{h} \end{equation}

Once again, here, $\partial_1f$ is a function from $\Bbb{R}^2$ into $\Bbb{R}$; so we'd write this as $\partial_1f: \Bbb{R}^2 \to \Bbb{R}$, while $(\partial_1f)(\alpha, y)$ is the particular real number we get if we compute the limit above.

* * *

I really hope you appreciate the importance of being precise about what a function is vs. what is the value of the function when evaluated on a point of its domain, and the proper notation to use in either case. Now, we can begin to dissect your question and phrase it properly.

You started by saying

> Say we have a function $f(x(t),t)$...

I understand what you want to say, but to make things explicit, I'll point out some things. You don't just have a single function. You actually have $3$ functions in the game at this point. First, you have a function $f: \Bbb{R}^2 \to \Bbb{R}$, next, you have a function $x: \Bbb{R} \to \Bbb{R}$, and lastly, **you're defining a new function via composition** as follows: you're defining a function $g: \Bbb{R} \to \Bbb{R}$ by the rule \begin{equation} g(t) := f(x(t),t) \end{equation}

Hence, you have $3$ functions $f,x,g$, and you should keep in mind which is which.

Next, you say

> ... and we take the partial derivative of $f(x(t),t)$ with respect to $t$ \begin{equation} \frac{\partial f(x(t),t)}{\partial t} \end{equation} do we hold $x(t)$ constant?

The notation you used is very bad, and this is presumably why you're getting confused. As you mentioned in your comment, you can't vary $t$, and simultaneously keep $x(t)$ fixed (unless $x$ is a constant function). The notation \begin{equation} \frac{\partial f(x(t),t)}{\partial t} \end{equation} is really just nonsense. (I don't mean to be disrespectful, I just want to point out that such an expression makes no sense, because it is unclear what you mean)

There are two possible interpretations of what you intended to write above. In proper notation, these are: \begin{equation} (\partial_2f)(x(t),t) \quad \text{and} \quad g'(t). \end{equation} In the first case, this means you compute the partial derivative wrt the second argument of $f$, i.e compute what the **function** $\partial_2f: \Bbb{R}^2 \to \Bbb{R}$ is, and evaluate that function on the tuple of numbers $(x(t),t)$. In the second case, you are computing the derivative of $g$, i.e compute the function $g': \Bbb{R} \to \Bbb{R}$, and then evaluate on the particular real number $t$. **These two are NOT the same in general!** If we write out explicitly in terms of limits, we have: \begin{equation} (\partial_2f)(x(t),t) := \lim_{h \to 0} \dfrac{f(x(t),t+h) - f(x(t),t)}{h} \end{equation} whereas \begin{align} g'(t) &:= \lim_{h \to 0} \dfrac{g(t+h) - g(t)}{h} \\ &:= \lim_{h \to 0} \dfrac{f \left(x(t+h), t+h \right) - f(x(t),t)}{h} \end{align}

This should make it clear that in general the two are different.

Now, the relationship between $g'$ and the partial derivatives $\partial_1f$ and $\partial_2f$ is obtained by the chain rule. We have: \begin{align} g'(t) = \left[ \left( \partial_1f \right)(x(t),t) \right] \cdot x'(t) + \left( \partial_2f \right)(x(t),t) \end{align}

This is the notationally precise way of writing things, because it avoids introducing unnecessary letters in the denominator, and it doesn't use the same letter for two different purposes, and also, it explicitly mentions where all the derivatives are being evaluated. One might see the following equation instead:

> \begin{equation} \dfrac{df(x(t),t)}{dt} \bigg|_{t} = \dfrac{\partial f}{\partial x} \bigg|_{(x(t),t)} \cdot \dfrac{dx}{dt} \bigg|_{t} + \dfrac{\partial f}{\partial t} \bigg|_{(x(t),t)} \end{equation} (this is still imprecise because of the irrelevant usage of $x$ and $t$ in the denominators, but this is as precise as you can get using Leibniz's notation)

You may even encounter a statement like

> \begin{equation} \dfrac{df}{dt} = \dfrac{\partial f}{\partial x} \cdot \dfrac{dx}{dt} + \dfrac{\partial f}{\partial t}, \end{equation}

which is absolutely terrible notation, because the $f$ on the LHS and the $f$ on the RHS mean completely different things. (This is what I meant in the beginning when I said "what people intend to write differs from what they actually write.")

Now, let's get to your specific example; I'll write out the derivatives in precise notation. (By the way, I believe you made some computational mistakes)

Recall that as I mentioned above, there isn't just one function involved. There are $3$ different functions. In your particular example, they are:

*   $f: \Bbb{R}^2 \to \Bbb{R}$ defined by $f(\xi, \eta) = \xi^2 + \eta$
*   $x: \Bbb{R} \to \Bbb{R}$ defined by $x(t) = t^2$
*   $g: \Bbb{R} \to \Bbb{R}$ defined by $g(t) = f(x(t),t) = f(t^2,t) = (t^2)^2 + t = t^4 + t$

I purposely used $\xi,\eta$, rather than writing $f(x,t) = x^2 + t$, because then we would be using $x$ in two places with different meanings; as the first argument of $f$, and also as a function (technically it is valid to write $f(x,t) = x^2 + t$, but I used $\xi,\eta$ simply to avoid potential confusion).

Using precise notation, the derivatives are as follows: \begin{align} (\partial_1f)(\xi,\eta) &= 2\xi \qquad (\partial_2f)(\xi,\eta) = 1 \\\\ x'(t)&= 2t \\\\ g'(t) &= 4t^3 + 1 \end{align}

(again pay close attention to what is the function, and where it is being evaluated).

Notice also that: \begin{align} \left[ \left( \partial_1f \right)(x(t),t) \right] \cdot x'(t) + \left( \partial_2f \right)(x(t),t) &= [2x(t)] \cdot 2t + 1 \\ &= [2t^2] \cdot 2t + 1 \\ &= 4t^3 + 1 \\ &= g'(t), \end{align} so we have explicitly verified that the chain rule works in this case.

Once again, to directly address your question, you said:

> I believe $\frac{\partial f(x(t),t)}{\partial x(t)}=2x(t)$ but does $\frac{\partial f(x(t),t)}{\partial t}=4t+1 $ or $1$?

this is extremely bad notation, putting $x(t)$ in the denominator gives the false idea that you are somehow differentiating $f$ with respect to the "function" $x(t)$, while keeping $t$ fixed... but then you might be confused how can $t$ be fixed if $x(t)$ is varying... or something like this. My point is that bad notation leads to misconceptions. So, in this case, the proper way to write these computations would be: \begin{align} \begin{cases} (\partial_1f)(x(t),t) &= 2 x(t) = 2t^2 \\\\ (\partial_2f)(x(t),t) &= 1 \\\\ g'(t) &= 4t^3 + 1 \qquad \text{(you wrote $4t$ rather than $4t^3$ which is wrong)} \end{cases} \end{align}

* * *

You should be able to figure out the answers to your remaining questions if you understood what I've been trying to emphasise, but for the sake of completeness, here it is:

You asked:

> To further my understanding, is the following true:

$$ \frac{\partial f(x(t),t)}{\partial t}=\frac{df(x(t),t)}{dt}$$

because $f(x(t),t)=f(t)$ after you simplify it by substituting in $x(t)$ into $f(x(t),t)$?

The immediate answer is "what you have written is ambiguous". The correct relationship is the chain rule which I wrote above (the three versions which go from precise to sloppy) \begin{align} g'(t) = \left[ \left( \partial_1f \right)(x(t),t) \right] \cdot x'(t) + \left( \partial_2f \right)(x(t),t) \tag{most precise} \end{align}

\begin{align} \dfrac{df(x(t),t)}{dt} \bigg|_{t} = \dfrac{\partial f}{\partial x} \bigg|_{(x(t),t)} \cdot \dfrac{dx}{dt} \bigg|_{t} + \dfrac{\partial f}{\partial t} \bigg|_{(x(t),t)} \tag{medium} \end{align}

\begin{align} \dfrac{df}{dt}= \dfrac{\partial f}{\partial x} \cdot \dfrac{dx}{dt} + \dfrac{\partial f}{\partial t} \tag{very sloppy} \end{align}

Also, please don't ever say things like "$f(x(t),t) = f(t)...$ " this makes no sense. I know that you mean to say after substituting $x(t)$ in the first argument for $f$, and $t$ in the second argument for $f$, you are left with something which only depends on $t$. But writing $f(t)$ for this new function of $t$ is bad notation. Use a different letter like $g$, as I have done above, to specify what the composition is. Otherwise, this raises the question "why does $f$ have two arguments on the LHS where as it only has one argument on the RHS?"

Also, it is this misuse of the letter $f$ on the two sides of the equal sign which confused you, and hence you said

> But if this is true, in the above example, $\frac{\partial f(x(t),t)}{\partial x(t)}=0$ because $f(x(t),t)=f(t)$ doesn't actually depend on $x(t)$.

Once again, as I mentioned above, we have $(\partial_1f)(x(t),t) = 2x(t) = 2t^2$ (this is $0$ if and only if $t=0$).

* * *

**FINAL REMARKS:**

I hope now you can appreciate why it is absolutely essential to understand what your notation means, and to distinguish between functions, like $f,g,x$, and $\partial_1f, \partial_2f, g', x'$, which are **RULES that map each element in the domain to a unique element of the target space**, and function **VALUES** like $f(\xi,\eta),f(x(t),t), g(t), x(t)$, which are **elements of the target space** of the function.

Also, I feel that it is mandatory to say the following. My entire answer has criticized Leibniz's notation for derivatives and partial derivatives, because it is very sloppy, and can cause all kinds of confusion (see any average math/physics book, in particular thermodynamics for all variants of notational/logical nonsense) and I have given several reasons why it should be avoided. However, having said that I do use Leibniz's notation often, but it is only because I (usually) know how to "translate" an imprecise notation into a completely precise form, and I (usually) understand the pitfalls associated with the notation. So, until you are able to perform this "translation" from precise to imprecise notation, I suggest you stick to the precise notation, until all the fundamental concepts are clear, and only then abuse notation. This may be tedious in the beginning, but it is well worth it to avoid silly confusions.