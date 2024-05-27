<!--yml
category: 未分类
date: 2024-05-27 13:23:42
-->

# Exploring Indra’s Pearls with WebGPU | by Nicolas Belmonte | Apr, 2024 | Medium

> 来源：[https://medium.com/@philogb/exploring-indras-pearls-with-webgpu-e0f4a745c2f6](https://medium.com/@philogb/exploring-indras-pearls-with-webgpu-e0f4a745c2f6)

## Generalizing Limit Sets: Beyond Circles

Instead of carefully selecting circles and Mobius transformations mapping the outside of one into the inside of another, we can generalize the work by directly computing limit sets from the Mobius generators themselves. This yields limit sets that are not necessarily built from circles.

Playing with parameters to get different limit sets

A parametrization of Mobius transformations that can give these “cusped” limit sets is known as the Maskit parametrization. A Maskit slice is the area under which the groups become non discrete (and we render nonsense).

µ values (x=Re(µ), y=Im(µ)) and the Maskit slice.

If we stay at the exact boundary, we get to groups that are at the limit of non-discreetness. The visualization allows to explore these cusped groups. The book goes into detail on the properties of these groups.

Moving through the boundary of the Maskit slice

# Implementation

I used this project to get immersed into WebGPU and WGSL for shading programming.

The initial visualizations are created via a `KleinGroup` class. The class renders each iteration into a texture that is fed back into the class for an ulterior iteration. Programmatically, this mimics well the action of the Mobius transformations. The resulting texture is fed into a `Quad` class which renders the quad to the screen.

We then include an `RSphere` class which will render the Riemann Sphere. `KleinGroup` feeds the texture both to `Quad` and `RSphere`. The `quad` is then moved around with the sphere placed on top to describe how conjugations work.

Animation showing the RSphere and Quad classes in action.

## Limit Sets

To visualize Limit Sets we implement Jos Ley’s: [*A fast algorithm for limit sets of Kleinian groups with the Maskit parametrisation*](https://www.josleys.com/articles/Kleinian%20escape-time_3.pdf). The algorithm renders a limit set via a fragment shader directly. The inputs to the fragment shader are the Mobius transformations and a texture that describes well the division across each side of the set. For each point in the screen if it falls in the area under or over the curve, we apply the Mobius transformation `a`or its inverse `A`.

The limit set and the underlying texture.

For sets far enough from the Maskit boundary, the line can be implemented as a smooth curve.

## Cusps

For Limit Sets that are at the boundary of a Maskit slice this is more complex. We need to find the combination of Mobius products for which the group is parabolic (we call this the parabolic “word”). For each two Mobius transformations and their inverses, `a`, `A`, `b`, `B`, the word for it to be parabolic could be something like: `aaaaBaaaaB`. Finding the fixed point for this matrix product gives us a point that is tangent to two circles in the set.

We then need to cycle through these words in order to obtain all other fixed points that will be tangent to each circle in the set. My implementation to get the word and its cyclic permutations can be found in [this notebook](https://observablehq.com/d/cde177e22e1a9a6e#getWord).

Texture computed for a cusped group at the boundary of a Maskit slice.

The texture above is using green and blue colors to indicate the algorithm to perform a horizontal shift before evaluating and applying the Mobius transformations (this gave me quite a headache).

## Computing the Maskit Slice

Finally, to trace the Maskit Boundary we need to implement a Newton solver for [parabolic generators](https://en.wikipedia.org/wiki/M%C3%B6bius_transformation#Fixed_points) (where the `Trace=2`). We walk the x-axis by obtaining a [Farey sequence](https://en.wikipedia.org/wiki/Farey_sequence) of fractions, for each of them we generate a Trace polygon, and we solve for it using Newton’s method, which will find the µ-value we need. Then the cusped limit set is fed the numerator of the Farey sequence `p`, its denominator `q`, and the complex value µ.