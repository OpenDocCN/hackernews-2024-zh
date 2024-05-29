<!--yml
category: 未分类
date: 2024-05-27 15:02:42
-->

# How to Fold a Julia Fractal — Acko.net

> 来源：[https://acko.net/blog/how-to-fold-a-julia-fractal/](https://acko.net/blog/how-to-fold-a-julia-fractal/)

We'll start with the classic real number line, marked at the integer positions, and poke around.
We imagine the line continues to the left and right indefinitely.

But there's a problem with this visualization: by picturing numbers as points,
it's not clear how they act upon each other.
For example, the two adjacent numbers $ \class{blue}{2} + \class{green}{3} $ sum to $ \class{red}{5} $ …

… but the similarly adjacent pair $ \class{blue}{-2} + \class{green}{-1} = \class{red}{-3} $.
We can't easily spot where the red point is going to be based on the blue and green.

A better solution is to represent our numbers using arrows instead, or *vectors*.
Each arrow represents a number through its length, pointing right/left for positive/negative.

The nice thing about arrows is that you can move them around without changing them.
To add two arrows, just lay them end to end. You can easily spot why $ \class{blue}{-2} + \class{green}{-1} = \class{red}{-3} $ …

… and why $ \class{blue}{2} + \class{green}{3} = \class{red}{5} $, similarly.
As long as we apply positives and negatives correctly, everything still works.

Now let's examine multiplication. We're going to start with $ \class{blue}{1} $ and then we'll multiply it by $ \class{green}{1.5} $ repeatedly.

With every multiplication, the vector gets longer by 50 percent.
These vectors represent the numbers $ \class{red}{1}, \class{red}{1.5}, \class{red}{2.25}, \class{red}{3.375} $, $ \class{red}{5.0625} $, a nice exponential sequence.

Now we're going to do the same, but multiplying by the negative, $ \class{green}{-1.5} $, repeatedly.

The vectors still grow by 50%, but they also flip around, alternating between positive and negative.
These vectors represent the sequence $ \class{red}{1}, \class{red}{-1.5}, \class{red}{2.25}, \class{red}{-3.375}, \class{red}{5.0625} $.

But there's another way of looking at this. What if instead of flipping from positive to negative, passing through zero, we went around instead, by rotating the vector as we're growing it?

We'd get the same numbers, but we've discovered something remarkable: a way to enter and pass through the netherworld around the number line. The question is, is this mathematically sound, or plain non-sense?

The challenge is to come up with a consistent rule for applying these rotations. We start with normal arithmetic. Multiplying by a positive didn't flip the sign, so we say we rotated by $ 0^\circ $. Multiplying by a negative flips the sign, so we rotated by $ \class{green}{180^\circ} $. The lengths are multiplied normally in both cases.

Now suppose we pick one of the in-between nether-numbers, say the vector of length $ 1.5 $, at a $ 90^\circ $ angle. What does that mean? That's what we're trying to find out! We'll write that as $ \class{green}{1.5 \angle 90^\circ} $ (*1.5 at 90 degrees*). It could make sense to say that multiplying by this number should rotate by $ \class{green}{90^\circ} $ while again growing the length by 50%.

This creates the spiral of points: $ \class{red}{1 \angle 0^\circ} $, $ \class{red}{1.5 \angle 90^\circ} $, $ \class{red}{2.25 \angle 180^\circ} $, $ \class{red}{3.375 \angle 270^\circ} $, $ \class{red}{5.0625 \angle 360^\circ} $. Three of those are normal numbers: $ +1 $, $ -2.25 $ and $ +5.0625 $, lying neatly on the real number line. The other two are new numbers conjured up from the void.

Let's examine this rotation more. We can pick $ 1 $ at a $ \class{green}{45^\circ} $ angle. Multiplying by a $ 1 $ probably shouldn't change a vector's length, which means we'd get a pure rotation effect.

By multiplying by $ \class{green}{1 \angle 45^\circ} $, we can rotate in increments of $ 45^\circ $.
It takes 4 multiplications to go from $ +1 $, around the circle of ones, and back to the real number $ -1 $.

And that's actually a remarkable thing, because it means our invented rule has created a square root of $ -1 $.
It's the number $ \class{green}{1 \angle 90^\circ} $.

If we multiply it by itself, we end up at angle $ \class{green}{90} + \class{green}{90} = \class{blue}{180^\circ} $, which is $ \class{blue}{-1} $ on the real line.

But actually, the same goes for $ \class{green}{1 \angle 270^\circ} $.

When we multiply it by itself, we end up at angle $ \class{green}{270} + \class{green}{270} = \class{blue}{540^\circ} $. But because we went around the circle once, that's the same as rotating by $ \class{blue}{180^\circ} $. So that's also equal to $ \class{blue}{-1} $.

Or we could think of $ +270^\circ $ as $ -90^\circ $, and rotate the other way. It works out just the same. This is quite remarkable: our rule is consistent no matter how many times we've looped around the circle.

Either way, $ \class{blue}{-1} $ has two square roots, separated by $ 180^\circ $, namely $ \class{green}{1 \angle 90^\circ} $ and $ \class{green}{1 \angle 270^\circ} $.
This is analogous to how both $ 2 $ and $ -2 $ are square roots of $ 4 $.

Complex multiplication can then be summarized as: *angles add up, lengths multiply*, taking care to preserve clockwise and counterwise angles. Above, we multiply two random complex numbers a and b to get c.

When we start changing the vectors, c turns along, being tugged by both a and b's angles. It wraps around the circle, while its length changes. Hence, complex numbers like to turn, and it's this rule that separates them from ordinary vectors.

We can then picture the complex plane as a grid of concentric circles. There's a circle of ones, a circle of twos, a circle of one-and-a-halfs, etc. Each number comes in many different versions or flavors, one positive, one negative, and infinitely many others in between, at arbitrary angles on both sides of the circle.

Which brings us to our reluctant and elusive friend, $ \class{blue}{i} $. This is the proper name for $ \class{blue}{1 \angle 90^\circ} $, and the way complex numbers are normally introduced: $ i^2 = -1 $. The magic is that we can put a complex number anywhere a real number goes, and the math still works out, oddly enough. We get complex answers about complex inputs.

Complex numbers are then usually written as the sum of their (real) X coordinate, and their (imaginary) Y coordinate, much like ordinary 2D vectors. But this is misleading: the ugly number $ \class{red}{\frac{\sqrt{3}}{2} + \frac{1}{2}i } $ is actually just $ \class{green}{1 \angle 30^\circ} $ in disguise, and it acts more like a $ 1 $ than a $ \frac{1}{2} $ or $ \frac{\sqrt{3}}{2} $. While knowing how to convert between the two is required for any real calculations, you can cheat by doing it visually.

But looking at individual vectors only gets us so far. We study functions of real numbers by looking at a graph that shows us every output for every input. To do the same for complex numbers, we need to understand how these numbers-that-like-to-turn, this field of vectors, change as a whole.
*Note: from now on, I'll put $ +1 $, i.e. $ 0^\circ $ at the 12 o'clock position for simplicity.*

When we apply a square root, each vector shifts. But really, it's the entire fabric of the complex plane that's warping. Each circle has been squeezed into a half-circle, because all the angles have been halved—the opposite of squaring, i.e. doubling the angle. The lengths have had a normal square root applied to them, compressing the grid at the edges and bulging it in the middle.

But remember how every number had two opposite square roots? This comes from the circular nature of complex math. If we take a vector and rotate it $ 360 ^\circ $, we end up in the same place, and the two vectors are equal. But after dividing the angles in half, those two vectors are now separated by only $ 180 ^\circ $ and lie on opposite ends of the circle. In complex math, they can both emerge.

Complex operations are then like folding or unfolding a piece of paper, only it's weird and stretchy and circular. This can be hard to grasp, but is easier to see in motion. To help see what's going on, I've cut the disc and separated the positive from the negative angles in 3D.

When we square our numbers to undo the square root, the angles double, folding the plane in on itself. The lengths are also squared, restoring the grid spacing to normal.

After squaring, each square root has now ended up on top of its identical twin, and we can merge everything back down to a flat plane. Everything matches up perfectly.

Thus the square root actually looks like this. New numbers flow in from the 'far side' as we try and shear the disc apart. The complex plane is stubborn and wants to stay connected, and will fold and unfold to ensure this is always the case. This is one of its most remarkable properties.

There's no limit to this folding or unfolding. If we take every number to the fourth power, angles are multiplied by four, while lengths are taken to the fourth power. This results in 4 copies of the plane being folded into one.

However, things are not always so neat. What happens if we were to take everything to an irrational power, say $ \frac{1}{\sqrt{2}} $? Angles get multiplied by $ 0.707106... $, which means a rotation of $ 360^\circ $ now becomes $ \sim 254.56^\circ $.

Because no multiple of $ 360 $ is divisible by $ \frac{1}{\sqrt{2}} $, the circular grid never matches up with itself again no matter how far we extend it. Hence, this operation splits a single unique complex number into an infinite amount of distinct copies.

For any irrational power $ p $, there are an infinite number of solutions to $ z^p = c $, all lying on a circle. For a hint as to why this is so, we can look at Taylor series: an arbitrary function $ f(z) $ can be written as an infinite sum $ a + bz + cz^2 + dz^3 + ... \,$ When z is complex, such a sum doesn't just represent a finite amount of folds, but a mindboggling infinite origami of complex space.