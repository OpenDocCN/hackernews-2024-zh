<!--yml
category: 未分类
date: 2024-05-27 15:00:41
-->

# Floats Are Weird

> 来源：[https://a.exozy.me/posts/floats-weird/](https://a.exozy.me/posts/floats-weird/)

Floats are weird. Anyways, here’s a basic calculus exercise:

\[\lim_{x \to 0} \frac{e^x-1}{x} = 1\]

However, I won’t prove that limit here, because this is a post about floats being weird, not a post about calculus. Instead, let’s try to approximate this limit numerically using Python, because what’s the worst that could happen? Let’s write that fraction as a Python function and plug in smaller and smaller $x$.

```
def f(x):  return (math.exp(x)-1)/x 
```

`f(1e-9)` returns 1.000000082740371 and `f(1e-12)` returns 1.000088900582341, which is already weird. Shouldn’t the approximation get better with smaller $x$? However, trouble really strikes with `f(1e-15)`. It returns a shocking 1.1102230246251565, which has a more than 10% error!

But, with this one weird trick&mldr;

```
def g(x):  y = math.exp(x) return (y-1)/math.log(y) 
```

`g(1e-9)` returns 1.0000000005, `g(1e-12)` returns 1.0000000000005, and `g(1e-15)` returns 1.0000000000000004, giving us exactly what we want! 🤯

Wait a minute! This new function is doing redundant computation! Why should we take $\log(e^x)$ when we can simply divide by $x$? How does this reduce the error so effectively?

## An assumption

To explain this phenomenon, we need to make a huge simplifying assumption. For some operation like $x+y$ when done using floating-point arithmetic, the final answer won’t be exactly equal to the exact value of $x+y$. Let’s use $fl(x+y)$ to denote the value of this floating-point calculation. Then we will assume that $fl(x+y) = (x+y)(1+\delta)$ for some small $\delta$. The upper bound on $\delta$ depends on what kind of floating-point numbers we’re using. Python uses IEEE 754 double-precision floats, which have a $\delta$ with magnitude at most $2^{-53}$.

You might be wondering why we didn’t choose a different assumption that $|fl(x+y) - (x+y)| < \epsilon$ for some small $\epsilon$. This is because floating-point numbers become spaced out farther apart as they get larger, so $fl(x+y)$ for large $x, y$ might have a large absolute difference from $x+y$, but the relative difference of $\frac{fl(x+y)}{x+y}$ will still be small. Thus, our original assumption is indeed the correct one to make.

We will also assume that this $1+\delta$ assumption holds for other operations like subtraction, multiplication, exponentiation, and logarithms. Now we have the necessary tools to tackle the problem!

## Analyzing f

First, let’s find $fl\left(\frac{e^x-1}{x}\right)$. Using our $1+\delta$ assumption repeatedly, we have

\[fl\left(\frac{e^x-1}{x}\right) = \frac{(e^x(1+\delta_1)-1)(1+\delta_2)}{x}(1+\delta_3).\]

You might remember from calculus that $e^x = 1+x+\frac{x^2}{2!}+\frac{x^3}{3!}+\dots$, which is a Taylor series. Since we are only plugging in very small values of $x$, $x^2$ and $x^3$ and other higher powers are very, very small numbers so we can simply ignore them. We’ll just say $e^x \approx 1+x$. This is a common trick in math and physics. We now get

\[fl\left(\frac{e^x-1}{x}\right) = \frac{((1+x)(1+\delta_1)-1)(1+\delta_2)}{x}(1+\delta_3) = \frac{(x+\delta_1+x\delta_1)(1+\delta_2)}{x}(1+\delta_3).\]

Again, since $x$ is small, $x\delta_1$ will be absolutely tiny, so we can also ignore it. Thus, the whole thing simplifies to

\[fl\left(\frac{e^x-1}{x}\right) = (1+\frac{\delta_1}{x})(1+\delta_2)(1+\delta_3).\]

If we plug in $x = 10^{-15}, \delta_1 = 2^{-53}$, we get $1+\frac{\delta_1}{x} = 1.1110$ which is pretty similar to the actual error that we got at the beginning! Cool, now we can see the source of the error: $x$ is in the denominator, so as it gets smaller, the error grows.

## Analyzing g

Next, let’s find $fl\left(\frac{e^x-1}{\log(e^x)}\right)$. Using our favorite assumption again, we get

\[fl\left(\frac{e^x-1}{\log(e^x)}\right) = \frac{(e^x(1+\delta_1)-1)(1+\delta_2)}{\log(e^x(1+\delta_1))(1+\delta_4)}(1+\delta_3).\]

This looks pretty scary, but we can use our $e^x \approx 1+x$ and $x\delta_1 \approx 0$ approximations to reduce it down to

\[fl\left(\frac{e^x-1}{\log(e^x)}\right) = \frac{(x+\delta_1)(1+\delta_2)}{\log(1+x+\delta_1)(1+\delta_4)}(1+\delta_3).\]

Now we need one last Taylor series, for $\log(x)$. Since this post isn’t about calculus (although it kind of is), I’m just going to give you the Taylor series that we need: $\log(x) = (x-1) - \frac{(x-1)^2}{2} + \frac{(x-1)^3}{3} - \frac{(x-1)^4}{4} + \dots$ (this series doesn’t converge for all $x$, but whatever). The important thing is that $(x-1)^2$ and the higher powers are going to be very small, since our $x$ that we’re plugging into $\log(x)$ is $1+x+\delta_1$. Thus, we can just ignore them and conclude $\log(x) \approx x-1$.

Something magical happens when we plug that approximation in. The $x+\delta_1$ in the numerator and the denominator cancel out! We’re just left with

\[fl\left(\frac{e^x-1}{\log(e^x)}\right) = \frac{1+\delta_2}{1+\delta_4}(1+\delta_3).\]

Awesome! Now we can clearly see that for small $x$, the resulting error is tiny. You might be wondering why there isn’t an $x$ in this answer. When we plugged in $x = 10^{-6}$, it wasn’t all that close to 1\. This is because with $x = 10^{-6}$, our approximations of $e^x \approx 1+x$ and $\log(x) \approx x-1$ aren’t exactly valid since the squared terms do contribute a bit. Basically, our answer is only true for when $x$ is very small and close to $10^{-15}$.

Intuitively, `g` is more accurate than `f` because the errors of the $e^x$ in the numerator and denominator end up cancelling each other out, but this is actually a pretty misleading explanation. The only way to see why this happens is to crunch through the analysis.

Anyways, floats are weird.