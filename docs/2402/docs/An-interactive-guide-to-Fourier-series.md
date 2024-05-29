<!--yml
category: 未分类
date: 2024-05-29 13:23:57
-->

# An interactive guide to Fourier series

> 来源：[https://injuly.in/blog/fourier-series/index.html](https://injuly.in/blog/fourier-series/index.html)

# An interactive guide to Fourier series

Oct 08

*See discussion on [HackerNews](https://news.ycombinator.com/item?id=39509414).*

Try drawing something on the first canvas, and watch two sets of mechanical alien arms retrace your sketch (no touch screen support yet):

<canvas id="draw-canvas">

<canvas id="redraw-canvas"></canvas>

</canvas>

Once you're done with this introduction to Fourier analysis, you'll be capable of making this (and a lot more) yourself.

The satisfying animation is made possible by the subject of this post - an infinite sum called [the Fourier series](https://en.wikipedia.org/wiki/Fourier_series). The formula is short, and with some effort, you can memorize it. However, I implore you to understand where the series comes from, and build deeper intuition for it.

To keep you from clicking off this page, I'll defer the proof and origin of this equation to the second half, and thread some interactive animations through the body of this write-up.

Before we get to tracing sketches however, let's brush up some basic math.

## Adding functions

Surely, you're familiar with the addition of numbers, vectors and matrices. Adding functions is not so different. The addition of two functions \(f\) and \(g\) at input \(x\) is simply \(f(x) + g(x)\).

Put more formally - \((f + g)(x) = f(x) + g(x)\).

Let's visualize this by taking an example. Assume `f` is \(2sin(x)\) and `g` is \(cos(2x)\).

Their sum then, can be given by a function: \(h(x) = 2sin(x) + cos(2x)\).

The graph below plots \(f\) and \(g\) in shades of gray, and their sum, \(h\), in red.

<canvas id="fun-sum"></canvas>

Note how in some places, the values of \(f\) and \(g\) are both positive, and their sum is therefore a larger positive number, while in other places, \(f\) and \(g\) have opposite signs and their values cancel out to a smaller number.

Through the lens of physics, you could look at the two functions as electromagnetic waves, or visible light rays oscillating in the domain of time. When two such waves overlap with each other in space, they're said to be in [superposition](https://www.britannica.com/science/principle-of-superposition-wave-motion). The superposition of two waves results in their sum.

When two points in a wave supplement each other to result in a higher amplitude (the y-value), their interaction is termed "constructive interference". When they cancel each other out, it's called "destructive interference".

Go through the last two paragraphs again, and try to digest this idea. Now, imagine if we had to work our way backwards. Say we are given a list containing the (x, y) coordinates of all points along the curve of \(h\), where \(x\) is time and \(y\) is the corresponding output of \(h\) at that point in time. We have to come up with two simpler periodic functions that sum up to \(h\).

This is exactly what the Fourier series does.

There are several ways to interpret interference in the real world. If \(f\) and \(g\) were sound waves, their constructive interference would make loud noise, while the destructive interference would produce a quieter sound. If they were light waves instead, their constructive interference would reveal bright spots on a reflective surface, and destructive would look like dim patches.

Applications of the Fourier series spill into almost every domain - signal processing, image compression, shape recognition, analog transmission, noise cancellation, studying thermodynamic systems and fitting equations to datasets.

From this wide array of applications, we show our interest in the science of tracing ugly sketches.

## Decomposing periodic functions.

Imagine you had a machine that could scan any food item and display its recipe. Fourier series does exactly that, except for mathematical functions.

The Fourier series of any periodic function \(f(x)\) with a frequency of \(\omega_0\) is described as:

$$ f(x) = a_0/2 + \sum_{n=1}^{\infty}b_n sin(n\omega_0x) + \sum_{n=1}^{\infty}a_n cos(n\omega_0x) $$

Meaning that for every periodic function \(f\), there exists a set of coefficients \(a\) and \(b\), such that \(f(x)\) can be expressed as an infinite sum of sine and cosine terms of increasing frequencies where the \(nth\) sine term has a coefficient of \(b_n\) and the \(nth\) cosine term has a coefficient of \(a_n\). The values of these coefficients are given by the following formulae:

$$ a_n = \int_0^T{f(x)cos(nw_0x)} $$

$$ b_n = \int_0^T{f(x)sin(nw_0x)} $$

The interval of integration, \(T\), is the fundamental period of the function. \(T\) and \(\omega_0\) are related by this equation:

$$ \omega_0 = 2\pi/T $$

If that was too wordy and made little sense to you, that's okay. We'll prove this equation later in the post. Until then, an example will help understand this better.

Consider the square wave - a periodic signal that alternates between 1 and -1 depending on its input. Formally, it is described like so:

$$ f(t) = 4 \lfloor{t}\rfloor - 2\lfloor2t\rfloor + 1 $$

Here's how it looks when graphed out:

<canvas id="square-wave-graph"></canvas>

If we use the first few terms from \(f\)'s Fourier series, we can closely approximate the behavior of this function. In the following graph, the gray curve represents the the square wave and the red curve represents our approximation of it. You can play with the slider to alter the number of terms we take from the series and see how that changes our approximation.

Clearly, our approximation improves as we take more terms from the series. The Fourier series can be proven to [converge](https://en.wikipedia.org/wiki/Convergent_series). This means that if we take an infinite number of terms from the series, we can get the *exact* value of \(f(x)\) for any \(x\).

Of course, it is not possible to add up infinite terms in computers. Instead, we decide upon a fixed number of terms that approximate our function well enough for most practical purposes.

Whenever I say "Fourier series of a function", I mean a series of simple periodic functions that can be added at any given input to approximate the output of the original function at the same input. For the remainder of this post our goal with Fourier series is to **approximate periodic functions with sums of simpler sine/cosine functions**.

## Drawing with the Fourier series

If you wish to understand how the Fourier series works before seeing it in action, you can skip this section and read ahead to [the proof](#proof), then come back here.

So, How do we go from decomposing time domain functions to recreating sketches?

Imagine you're drawing a sketch on a square sheet of paper. You are to draw your sketch, start to finish, without lifting the nib of your pen from the paper's surface. In other words, your sketch must be *continuous* with no "breaks" in between.

Assume also that the bottom-left corner of the sheet is its origin. Once you start drawing, I can delineate the position of the pen's tip using a pair of coordinates \((x, y)\) at any given point in time.

Much like a cartesian plane, the \(x\) coordinate represents the horizontal distance from the origin, and \(y\) the vertical. Both the x and y coordinates change as the pen moves on the sheet's surface. Meaning, the position of the x-coordinate of your pen's tip can be written as a function of time. Say you draw this figure:

<canvas id="rabbit-canvas"></canvas>

If we plot the x and y-coordinates independently as functions of time, they'll form curves that look like this:

<canvas id="rabbit-plot-canvas"></canvas>

The blue curve represents the values of x-coordinates of your sketch. The vertical axis represents the x-value, and the horizontal axis represents time. Similarly, the red curve plots the y-coordinates.

Both these curves can be viewed as functions of time. The blue curve represents a function \(x(t)\) that returns the x-position of the pen's tip at time \(t\), Similarly, the red curve is a function \(y(t)\) which the same for its y-position. For each of these functions, we can find a Fourier series that approximates it.

Let \(f_x(t)\) and \(f_y(t)\) be the Fourier approximations for \(x(t)\) and \(y(t)\) respectively. Then recreating the sketch requires computing the values returned by `f_t` and `y_t` over a range of values of t. then pairing them into `(x, y)` coordinates and connecting the coordinates with lines. Here is some pseudo-typescript code that mimics this logic:

 ````
// The "dt" is our time step.
// In the real world, a line is an infinitely long series of points.
// In computers, we take a "snapshot" of the pen's position
// every dt seconds and join these positions with straight lines to
// trace the curve. Smaller values of dt require more computation,
// and yield better results.
const dt = 0.01;
const f_x = fourier_series(x); // type of x is (t: number) => number
const f_y = fourier_series(y); // type of y is (t: number) => number
let prev_point = [f_x(0), f_y(0)];
for (let t = 0.01; t < 1; t += dt) {
 const current_point = [f_x(t), f_y(t)];
 draw_line(prev_point, current_point);
 prev_point = current_point;
}
```` 

The approximation generated by this method is shown below. Just as before, you can play with the slider to adjust the number of terms used in approximation of the sketch.

Keep in mind that `f_x` and `f_y` are really just sums of simpler sine/cosine functions, calculated using Fourier's formulae.

You may be wondering - the functions \(x(t)\) and \(y(t)\) aren't periodic, how come we can still decompose them into sine/cosine sums? One trick is to set the period to infinity, and compute the series at this limit.

In my code, I just set the period to 1 time unit, and assume that the pen just retraces the drawing again and again. Meaning that \(x(t + 1) = x(0)\). This makes the math a lot easier, and certainly doesn't make a difference in the outcome.

To be more clear, when the sketch starts, the time is assumed to be 0, and when it ends, the time is assumed to be 1 second. Every time point in between is scaled accordingly. This is not necessary of course, you could set the time period to however long it took to draw the first sketch, if that makes things simpler for you.

## Epicycles

The final caveat are the epicycles. It is easy to just plot the values returned by \(f_x\) and \(f_y\) on the cartesian plane. But how do we animate this using revolving circles?

If you've followed the contents of this article so far, you already know how to recreate sketches. To animate them, you need to understand [The polar coordinate system](https://en.wikipedia.org/wiki/Polar_coordinate_system).

You can read the wikipedia article, or [this article](https://www.mathsisfun.com/polar-cartesian-coordinates.html) to build some intuition for conversion between cartesian and polar coordinates.

In the polar coordinate system, a periodic function with period \(T\) is a vector that rotates around the origin, and completes one full rotation around itself every \(T\) time units. Look at the graph of \(sin(t)\) in Polar form, for example:

<canvas id="polar-sine"></canvas>

Note how the y-coordinates of the vector's tip traces out a regular sine wave. You can just as easily plot any periodic function in the polar coordinate system. To add two periodic functions together, take one rotating vector and center it on the tip of the another rotating vector. The end result is shown below. The following animation shows 3 rotating vectors added together, each representing a periodic function:

<canvas id="two-rotating-vectors"></canvas>

To convert a sketch to an epicycle animation then, all we need is to convert a term in the Fourier series from cartesian to polar coordinates. Once we have that, we can add up the terms like in the animation above, and figure out the x and y-coordinates using two sets of epicycles, each representing the Fourier approximation for \(x(t)\) or \(y(t)\).

To do this conversion, we can use the [polar form of the Fourier series](https://en.wikipedia.org/wiki/Fourier_series#Amplitude-phase_form). Precisely, these are the steps you need to follow:

1.  Represent the sketch as a list of points drawn over a period of time.
2.  Convert the list of points into a two separate lists, one containing the x-coordinates of the sketch, and other the y.
3.  Convert each list into a function (I use [this simple helper](https://github.com/srijan-paul/fourier-sketch/blob/eb2be0f646f3097c6725ab621461ba59bfba4b6b/src/math/util.ts#L58)). Now, you have the \(x(t)\) and \(y(t)\).
4.  For each function, find its Fourier series coefficients. [Here](https://github.com/srijan-paul/fourier-sketch/blob/eb2be0f646f3097c6725ab621461ba59bfba4b6b/src/math/fourier.ts#L16) is how I do it.
5.  For each function, [convert the Fourier series coefficients into a set of polar functions](https://github.com/srijan-paul/fourier-sketch/blob/eb2be0f646f3097c6725ab621461ba59bfba4b6b/src/math/util.ts#L93).
6.  Using a time step of `dt`, find the final x and y positions of our approximation, and [draw them on a canvas](https://github.com/srijan-paul/fourier-sketch/blob/eb2be0f646f3097c6725ab621461ba59bfba4b6b/src/components/RedrawCanvas.tsx#L21).

If you do everything correctly, you should get something like this:

<canvas id="rabbit-epicycle"></canvas>

There is a more novel approach to retracing sketches that involves using only one set of epicycles. It uses [the complex Fourier Series](https://en.wikipedia.org/wiki/Fourier_series#Complex-valued_functions), and is also fewer lines of code. When you're new to this concept however, it may throw you off balance, especially if you're not familiar with imaginary numbers and the Argand plane.

## Proof

When I set out to find an "intuitive" proof for the Fourier series, all I saw were proofs that begin by stating the equation, and then proving it by finding the coefficients \(a_n\) and \(b_n\) using integrals. But where did the equation come from?

Did God whisper it to Joseph Fourier in his dreams?

Did he just happen to run into it by chance?

Surprisingly, the answer is "yes". Of course, he had an unparalleled instinct for math that he whetted with years of practice and research. There has to be a certain train of thought that he boarded to arrive at this revelation, that any periodic signal can be represented as a sum of simpler harmonics. But that line of thinking was never publicized, and as you'll see in the next section, there have been people who've thought of this even before Fourier himself did!

The important part is that Fourier asked a question that was mocked as stupid and bizarre until he presented a proof. And that proof does in fact begin by stating the following hypothesis:

$$ f_o(t) = \sum_{n = 0}^\infty{b_nsin(n\omega_0t)} $$

Here, \(f_o\) is an odd function with a fundamental period of \(w_0\). If we can derive a value for \(b_n\) from this equation, we can be convinced that **any odd function can be represented as a sum of sinusoids**.

Now, consider an even function \(f_e\) with a period of \(w_0\):

$$ f_e(t) = \sum_{n=0}^{\infty}a_n cos(n w_0 t) $$

If we can derive a value for \(a_n\) from this equation, we can be convinced that **any even function can be represented as a sum of co-sinusoids**.

When you combine these two equations with the idea that [any periodic function can be represented as a sum of odd and an even function](https://en.wikipedia.org/wiki/Even_and_odd_functions#Even%E2%80%93odd_decomposition), you get:

$$ f_o(t) + f_e(t) = \sum_{n = 0}^\infty{b_nsin(n\omega_0t)} + \sum_{n=0}^{\infty}a_n cos(n w_0 t) $$

We can turn the order of this proof, and first say that given any function \(f(t)\), we can find its odd and even parts using the odd-even decomposition rule. Then, we can represent the odd part as a sum of sinusoids, and the even part as a sum of co-sinusoids.

Now, all that's left is to derive the values for \(a_n\) and \(b_n\) using the two equations stated above. This is where I save myself the trouble of writing more LaTeX, and defer you to [this excellent proof](http://lpsa.swarthmore.edu/Fourier/Series/DerFS.html) by professors from Swarthmore college. I know I said I'd walk you through the proof, but I can't do a better job of it than the electronics professors at Swarthmore did already. I'd hate to repeat their work and not give credit. If you follow the page I linked, you'll realize that the proof only uses basic calculus and trigonometric identities taught in high school.

## Origins

You'll be surprised to learn that the idea behind the series predates Fourier himself.

[Carl Friedrich Gauss](https://en.wikipedia.org/wiki/Carl_Friedrich_Gauss) was one of the many applied mathematicians who wanted to predict the position of Ceres in the night sky. To attain this goal, he created several algorithms to aid his study of astronomy. One of them was the [Fast Fourier Transform](https://en.wikipedia.org/wiki/Fast_Fourier_transform) - a function that is very closely related with the Fourier Series. However, he never published his work because he believed his method to be an unimportant detail in the overall process.

In the 1700s, Euler had found applications for decomposing periodic functions with Fourier Series.

Half a century before Fourier, [Bernoulli](https://en.wikipedia.org/wiki/Daniel_Bernoulli) was studying the motion of a string. He proposed the idea that periodic functions can be represented as sums of harmonics. Nobody at the time believed this to be a general method, and his ideas were left unexplored.

Things changed in 1807, when a French math wizard named Joseph Fourier found himself studying the heat equation in a metal plate. In his search for a solution, he sought to ask a seemingly absurd question:

*Can we represent any periodic function as a sum of simple sine and cosine functions?*

Precisely, he sought to represent any periodic function \(f(x)\) with a frequency of \(\omega_0\) , in the following form:

$$ f(x) = (a_0 + a_1 cos(\omega_0 t) + a_2 cos(2\omega_0 t) + ... + a_n cos(n\omega_0t)) + (b_1 sin(\omega_0 t) + b_1 sin(2\omega_0 t) + ... + b_n sin(n\omega_0t) $$

Revered mathematicians of the time, including Langrange and Laplace, rejected this idea as informal and hand-wavy. The panel evaluating his findings said:

*"The manner in which the author arrives at these equations is not exempt of difficulties and...his analysis to integrate them still leaves something to be desired on the score of generality and even rigour."* Perhaps this was because of a lack of reasoning as to *why* one should even begin to think of periodic functions this way.

It's not unheard of mathematical ideas to sprout into existence out of seemingly ridiculous places. Ramanujan attributed some of his major findings to God, and dipped at the age of 32.

After the Fourier Series was accepted by the scientific populace, it spawned a new field of research, called Fourier analysis. Developments in this field found everyday use in almost every science.

## Applications

By this point, you know enough about Fourier analysis to delve deeper into it yourself. It would be a shame to blunt the edge of theory by not applying it in practice.

Here a few things you could do:

*   Implement noise reduction in sounds.
*   Sharpen images with denoising.
*   Write a [JPEG](https://en.wikipedia.org/wiki/JPEG) encoder/decoder.
*   [Fit an elephant](https://www.johndcook.com/blog/2011/06/21/how-to-fit-an-elephant/)
*   Write basic shape recognizers.

## Resources and further reading