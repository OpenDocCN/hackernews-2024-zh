<!--yml
category: 未分类
date: 2024-05-27 14:59:50
-->

# Generating The Alphabet — Amy Goodchild

> 来源：[https://www.amygoodchild.com/blog/generating-the-alphabet](https://www.amygoodchild.com/blog/generating-the-alphabet)

I defined my letters around a central point, mid_x and mid_y. In hindsight it would have been better to work from a bottom left point and I’ll be adjusting this at some point, to help improve my kerning, which is currently inconsistent.

Within a Letter class, I defined these key locations like x height, cap height etc, in relation to the font size and the mid point. For example, the full height from base to cap is equal to the font size. From y_mid to y_x is 1/3 of the full height.

I also defined some small distances I could adjust a point by, in relation to the height.