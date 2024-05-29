<!--yml
category: 未分类
date: 2024-05-27 14:46:34
-->

# CSS is Logical - Geoff Graham

> 来源：[https://geoffgraham.me/css-is-logical/](https://geoffgraham.me/css-is-logical/)

*   Documents flow in the direction of the writing mode.
*   Elements flow in block and inline directions.
*   The block and inline directions are based on the writing mode.
*   Relatively positioned elements have a reserved space in the document flow while absolutely positioned elements are pulled out of it.
*   Screens are measured in viewport units.
*   Parent elements are measured in container units.
*   Em and rem units evaluate relative to their elemental and root contexts.
*   Setting `z-index` creates a new stacking context for managing the way elements visually overlap one another.
*   The Cascade [organizes selectors](https://css-tricks.com/the-c-in-css-the-cascade/) in prioritized layers that can be re-organized into customized Cascade Layers.
*   Selectors are evaluated by a specificity score.
*   The applied style when two selectors match is the one that appears lowest in the Cascade when they share the same origin.
*   If one property is invalid, the next matched property in a matching selector is used.
*   [`width` looks up the tree while `height` looks down the tree.](https://geoffgraham.wpengine.com/width-looks-outward-height-looks-inward/)
*   Specific properties can be declared based on whether or not a browser `@supports` it.
*   An element can be selected if it `:is()` a certain selector, is `:not()` a certain selector, `:has()` another element, or `:where()` it meets a certain condition.

CSS be weird, but it not be illogical.