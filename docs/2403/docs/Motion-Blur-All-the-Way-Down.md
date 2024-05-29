<!--yml
category: 未分类
date: 2024-05-27 14:36:42
-->

# Motion Blur All the Way Down

> 来源：[https://www.osar.fr/notes/motionblur/](https://www.osar.fr/notes/motionblur/)

# Motion Blur All the Way Down

Pierre Cusa <pierre@osar.fr>

November 2022

[(history)](https://github.com/pac-dev/notes/commits/master/content/motionblur.md)

"Torusphere Accelerator", the animation that motivated this article.

What happens if you take motion blur past its logical extreme? Here are some fun observations and ideas I encountered while trying to answer this question, with an attempt to apply the results in a procedural animation.

## What is motion blur supposed to look like?

Motion blur started out purely as a film artifact, the result of a subject moving while the camera's shutter is open. This artifact turned out to be desirable, especially for videos, because it improves the perceptual similarity between a video and a natural scene, something I'll dive into in this section.

In a 3D and animation context, it's interesting to note that those two goals - looking natural, and simulating a camera, might not be in agreement, and might result in different motion blurs. I'll keep the simulation aspect as a side note, and ask what the most natural possible motion blur should look like. This can be broken down into a few questions:

1.  How do we perceive a natural moving scene?
2.  How do we perceive this scene reproduced in a video?
3.  What is the perceptual difference between those two cases?
4.  How can video motion blur minimize this difference?

### Perception of motion in a natural scene

For the purpose of crafting motion blur, we can start by analyzing the very first steps of human vision, where the light hits our retina and [phototransduction](https://en.wikipedia.org/wiki/Visual_phototransduction) takes place. Under well-lit conditions, this is handled by cone-type cells. Phototransduction is not immediate, and we can model this lag by smoothing out the light stimulus over time.

**1\. Example of temporal integration in goldfish cones**

, taken directly from

[Howlett et al. (2017)](https://journals.plos.org/plosbiology/article?id=10.1371/journal.pbio.2001210)

. Raw stimulus is the number of photons entering the photoreceptor, in this case, a realistic example. The weighting function is derived from the cone's response in different conditions, and can be used to simulate the cone's average temporal integration. The resulting "effective stimulus", while not a measurable response, is a successful first step in modeling the cone's actual photocurrent response.

Combining the above weighting function's shape with known human cone response times, we can create a detailed simulation of a *perceived image* based on any input scene. This concept has been [used before](https://pubmed.ncbi.nlm.nih.gov/26357212/), but without the shape of the time response.

**2\. Motion Smearing**. Left: Example scene, assumed to be natural and continuous. Right: simulated perceived image. This assumes the viewer is looking at a fixed point, and not tracking the object with their eyes.

What this shows is that there already exists a natural blur at the photoreceptor level, a phenomenon often called motion smear. So why do we add artificial motion blur in videos, and what is the link between motion smear and motion blur?

### Perception of a scene on a screen

Let's see what this perceived image looks like when viewing a screen with limited frames per second.

**3\. Perception of a video**. Left: Example scene as it would appear on a screen. Right: perceived image.

This finally gives us a way of visualizing the usefulness of motion blur. When viewing a video without motion blur, the resulting perceived image looks like overlaid frames instead of the expected motion smear. This situation is improved with a motion blurred video, where each frame no longer shows a moment in time, but an average of all the moments within the time interval covered by this frame. This is analogous to a video made with a camera whose shutter is open for the duration of each frame's timespan. The resulting perceived image looks a lot more similar to the natural case.

### Making the screen natural with a shutter function

Something still looks off in the perceived image for traditional motion blur. At some object speeds, artifacts still appear as discontinuities in the motion smear. This can be almost eliminated by applying a shutter function: instead of averaging all the moments within a frame, we weight them by a function so that the start and end moments have less weight than the central moment of a frame. The name "shutter function" comes from the analogy to shutter efficiency in a [diaphragm camera](https://en.wikipedia.org/wiki/Diaphragm_%28optics%29), where the shutter takes time to transition between open and closed states. But instead of simulating cameras, the shutter function can be chosen in a way that minimizes the perceptual difference between the screen and a natural scene. The problem then becomes very similar to crafting [window functions](https://en.wikipedia.org/wiki/Window_function) in signal processing, and indeed the most popular window functions give very good results.

**4\. Applying a shutter function**.

How well does this work? You can get a rough idea in the following demo, which can be switched between motion blur with and without a shutter function. My impression is that the shutter function is not necessary at low speeds, but is noticeably more natural for fast-moving objects^(. This makes it highly relevant to the "past the logical extreme" experiment I'm aiming for. It also looks smoother in still frames, which is incidental but sometimes relevant.)

**5\. Live comparison** of motion blur with and without a shutter function.

I'll emphasize that this perceptual approach to motion blur is not conventional and could be misguided in some way. The common approach is to simulate cameras, which results in zero time overlap between frames, and often entirely discards moments that fall between frames. Meanwhile, the method I'm describing results in overlapping time-ranges for successive frames. With that out of the way, let's try applying this technique.

## Getting irrational with the torusphere

To make things both difficult and interesting, I decided to make this infinite motion blur animation as a realtime [shader](https://thebookofshaders.com/01/). Because I like hardship and misery, yes, but mostly because I'd like the end product to be interactive, and in this case, a shader might be the simplest way.

First, how does one render motion blur in realtime? After ruling out multisampling ^(and analytic ray-traced motion blur^(, I settled on a terrible hack best described as "integrated volume motion blur". Represent the moving object as a function that takes coordinates (including time) and returns density (the inside is 1, the rest is 0). Integrate this density function over time, and the result should give you a "motion-blurred density" over any time interval. The result can be rendered by [volume ray casting](https://en.wikipedia.org/wiki/Volume_ray_casting). This method is not photorealistic, but can handle extremely long trails with realtime performance.))

The intended animation combines an **orbiting sphere** and a **rotating torus**, both of which need to be motion-blurred up to essentially infinite speed.

### Motion-blurred sphere

Taking a 2D slice of the orbiting sphere, the problem is reduced to finding the motion-blurred density for an orbiting circle. Let's assume an orbital radius $R$, and a circle of radius $a$. The circle's center is always at a distance of $R$ from the origin, so it can start at the point $(R, 0)$. This means that initially, all points $(x, y)$ on the circle are defined by:

$$ (x - R)^2 + y^2 = a^2 $$

In order to work with the orbit, this should be expressed in polar coordinates $(r,\theta)$, which can be done by substitution:

$$ r^2 - 2 r R \cos\theta + R^2 = a^2 $$

Finding the density function means taking any point, and answering the question: When does this point enter the orbiting circle? When does it exit? The answer lies in the angle coordinate coordinate $\theta$ of the initial object's surface, with the same radial coordinate $r$ as the given point. Because the object is orbiting, this angle is directly related to the time when the object will hit the point. So let's find $\theta$ based on the above definition of the surface:

$$ \theta = \pm\arccos\frac{R^2 + r^2 - a^2}{2 r R}, \theta\in[-\pi,\pi] $$

The $\pm$ sign comes from the inversion of $\cos$. This $\pm$ is useful, since it determines which half-circle is defined: positive or negative $\theta$. The two halves can be combined to get a polar expression of the density $\rho$ of the corresponding disk:

$$ \rho(r,\theta) = \begin{cases} 1 & \text{if }-h(r)\lt\theta\lt h(r) \cr 0 & \text{otherwise} \end{cases}\\[2ex] \text{where}\ \ h(r) = \arccos\frac{R^2 + r^2 - a^2}{2 r R} $$

From this starting position, the disk is orbiting around the origin. This is equivalent to removing the time $t$ times the speed $v$ from the angle coordinate:

$$ \rho(\colorbox{yellow}{t,}r,\theta) = \begin{cases} 1 & \text{if }-h(r)\lt\theta\colorbox{yellow}{- v t}\lt h(r) \cr 0 & \text{otherwise} \end{cases} $$

We can separate $t$ from the time interval $I$ during which the object is present at a point $(r,\theta)$:

$$ \rho(t, r,\theta) = \begin{cases} 1 & \text{if }t\in I \cr 0 & \text{otherwise} \end{cases}\\[2ex] I=\left[\cfrac{\theta-h(r)}{v}, \cfrac{\theta+h(r)}{v}\right] $$

The motion-blurred density is the integral of the density $\rho$ over the current frame's time interval $F$. This works out to be the length of the intersection between $I$ and $F$. This can also be described intuitively: we're measuring how much of the frame's time is occupied by the object at a given point in space.

$$\int_F\rho(t,r,\theta) d t = \int_{F\cap I}1\ d t = |F\cap I|$$

Let's apply a shutter function $s$. For simplicity, assume $s$ is already centered on the current frame's time span. We can apply it by multiplying the density with $s(t)$ before integrating, replacing the need for any bounds of integration of the density. If $s$ has an antiderivative $S$, then the motion-blurred density becomes:

$$ \int\rho(t,r,\theta) s(t) d t = \int_I s(t) d t = S(\max I)-S(\min I) $$

This can be implemented in a shader and works with any shutter function, however, based on the goals from the first part of this article, shutter functions should have an integral of 1 and should overlap in such a way that the sum of all shutter functions at any timepoint is always 1\. This can be satisfied with a trapezoid function, or with a sinusoid function such as this one, used in the animation:

$$ s(t)=\begin{cases} \cfrac{1-\cos\frac{(t-A)2\pi}{B-A}}{B-A} & \text{if }A\lt t\lt B \cr 0 & \text{otherwise} \end{cases}\\[2ex] A=\min F-\frac{|F|}{2},B=\max F+\frac{|F|}{2} $$

### Motion-blurred torus

The same process can be followed for the torus. A 2D vertical slice of a torus is called a [spiric section](https://en.wikipedia.org/wiki/Spiric_section), or Spiric of Perseus. Aside from sounding like an **epic videogame weapon**, it also has a convenient formulation in polar coordinates. Take a torus of minor radius $a$ and major radius $b$. Take a section at position $c$, and within this section in polar coordinates $(r,\theta)$, all torus points are defined by:

$$ (r^2-a^2+b^2+c^2)^2 = 4b^2(r^2\cos^2\theta+c^2) $$

Solving for $\theta$, assuming $\theta\in[-\pi/2,\pi/2]$, this becomes:

$$ \theta = \pm\arccos\frac{\sqrt{(a^2 - b^2 - c^2 - r^2 - 2 b c) (a^2 - b^2 - c^2 - r^2 + 2 b c)}}{2 b r} $$

Once again, the inside of the torus is enclosed between the positive and negative cases of the $\pm$ sign, giving us a polar expression of the density of the solid torus. The remaining steps to get the motion-blurred rotating torus are exactly the same as for the sphere above.

**6\. Motion-blurred spiric section**.

### Putting it together

All that's left is to "draw the rest of the owl" by combining elements in a convincing way, and by using standard volume ray casting on the result. [Surface normals](https://en.wikipedia.org/wiki/Normal_%28geometry%29) need extra care because there's no such thing as "motion-blurred surface normals", so they're just blended together here.

The animation should run below with basic mouse/touch interaction. It might not work well on all devices, so there's also a pre-rendered video at the top of the page. You can also find [this shader on Shadertoy](https://www.shadertoy.com/view/cdXSRn).

Torusphere accelerator (live)

* * *