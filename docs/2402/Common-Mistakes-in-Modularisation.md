<!--yml
category: 未分类
date: 2024-05-29 13:29:26
-->

# Common Mistakes in Modularisation

> 来源：[https://two-wrongs.com/software-design-tree-and-program-families.html](https://two-wrongs.com/software-design-tree-and-program-families.html)

When we are new to product development, we often dig into the concrete details right away. This means we make decisions near the leaves before we tackle higher level decisions. Halfway through the development work (i.e. when significant amounts of code has been written already), we may have a design tree that looks like this:

What product does that correspond to? None at all. These decisions are on completely different branches, meaning they are mutually exclusive, and there is no way to construct a product that satisfies them all. We generally don’t notice this until someone forces us to make a decision on e.g. question *b*, and then we realise that whatever we choose, we are locking ourselves out of either *k* or *g*.

The silver lining on the dark clouds is that this is usually discovered rather late in the development process, so at this point the stakeholders may actually be receptive to throwing out most of the decisions we had agreed to in order to make the project finish sooner. It is a little embarrassing none the less.

Sometimes we are lucky and by accident make coherent leaf-level choices, e.g. like this:

But then when we get around to discussing question *a*, we realise we would much rather make the choice that unlocks *c* as an alternative. Unfortunately, that’s incompatible with all decisions we’ve made so far.

This is a devious situation that occurs surprisingly often: our early leaf-near decisions imply a set of higher level decisions that we disagree with. Again, we usually don’t discover this until we are trying to integrate everything and get it out the door. Result: lots of expensive rework, or settling for a suboptimal solution.

This is another reason to focus on high-level decisions first: we lower the risk of spending time on decisions that we’ll have to tear up later because they are incompatible with the overall shape we want the product to have.