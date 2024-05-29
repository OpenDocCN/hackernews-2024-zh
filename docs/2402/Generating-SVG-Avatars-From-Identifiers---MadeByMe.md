<!--yml
category: 未分类
date: 2024-05-29 13:23:02
-->

# Generating SVG Avatars From Identifiers - MadeByMe

> 来源：[https://madebyme.today/projects/generating-svg-avatars-from-identifiers/](https://madebyme.today/projects/generating-svg-avatars-from-identifiers/)

<main class="main">

When you create a new social service on the internet, one of the things you need to think about is how to make a user’s space feel like their own. Services like [GitHub](https://github.blog/2013-08-14-identicons/), Roll20, or [Reddit](https://www.reddit.com/r/reddit/comments/vtkmni/introducing_collectible_avatars/) generate — or allow you to generate — a custom avatar (i.e. [Identicons](https://en.wikipedia.org/wiki/Identicon)) for your account. Avatar auto-generation is especially neat, as it does not require any engagement from the user. Let’s try to figure that out on our own.

I think this blog post is one of those things which are easier to understand if you’d see the end result first. So, for those interested here’s a spoiler.

This is the end result:

Avatar for identifier "Foo".

The avatar has 24 segments and consists of 24 [`<path>`](https://www.w3.org/TR/SVG2/paths.html#PathElement) tags. The coordinates describing paths are static, but their colors are generated from [SHA-256](https://en.wikipedia.org/wiki/SHA-2) of the identifier.

```
[1](#hl-0-1)SHA256("Foo") == [ 44,  38, 180, 107, 104, 255, 198, 143, [2](#hl-0-2) 249, 155,  69,  60,  29,  48,  65,  52, [3](#hl-0-3) 19,  66,  45, 112, 100, 131, 191, 160, [4](#hl-0-4) 249, 138,  94, 136,  98, 102, 231, 174] [5](#hl-0-5) [6](#hl-0-6)Base64(SHA256("Foo")) == LCa0a2j/xo/5m0U8HTBBNBNCLXBkg7+g+YpeiGJm564= 
```

The rest of this blog post focuses on translating the byte array into an avatar that looks *nice*.

## [#](#svg-scaffolding) SVG scaffolding

Let’s think how to prepare the SVG’s structure. We know that each segment must be its own [`<path>`](https://www.w3.org/TR/SVG2/paths.html#PathElement) element, so we can color them individually with the [fill](https://www.w3.org/TR/SVG2/painting.html#SpecifyingFillPaint) property. Generating annuluses sectors can be challenging — it would require using two(!) [arcs](https://www.w3.org/TR/SVG2/paths.html#PathDataEllipticalArcCommands), which is way too much — instead, we can cheat a little and create overlapping circle sectors. It will make paths simpler and requires only one(!) arc.

Starting with the `<svg>` tag we get:

```
[1](#hl-0-1)<svg viewBox="-1 -1 2 2" xmlns="http://www.w3.org/2000/svg"> [2](#hl-0-2) <!-- ... --> [3](#hl-0-3)</svg> 
```

Paths generation gets much easier if the center of “circles” is in $(0, 0)$. We can do that by setting the `viewBox` property as shown above: top-left corner is $(-1, -1)$, width is $2$, and height is $2$.

OK. Now we need to create circle sectors. SVG has some tags for constructing [basic shapes](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Basic_Shapes) (like circles, rectangles, or ellipses). Sadly, pizza slices are *not* considered as “basic”, so there’s no `<pizza-slice>` to help us here. Instead, we need to use the [`<path>`](https://www.w3.org/TR/SVG2/paths.html#PathElement) tag.

### [#](#making-pizza-slices-pizza) Making pizza slices 🍕

The [`<path>`](https://www.w3.org/TR/SVG2/paths.html#PathElement) tag is nice, because it allows you to construct any complex shape. For it to work we need to describe how we want our shape to look like with its special [path data](https://www.w3.org/TR/SVG2/paths.html#PathData) syntax. It’s fairly complex, but we only need to grasp 4 of its commands:

*   the [absolute **“moveto”**](https://www.w3.org/TR/SVG2/paths.html#PathDataMovetoCommands) command — which establishes a new initial point,
*   the [absolute **“lineto”**](https://www.w3.org/TR/SVG2/paths.html#PathDataLinetoCommands) command — which draws a line from the current point to the given $(x, y)$,
*   the [absolute elliptical arc curve](https://www.w3.org/TR/SVG2/paths.html#PathDataEllipticalArcCommands) command — which draws an elliptical arc from the current point to the given $(x, y)$,
*   the [**“closepath”**](https://www.w3.org/TR/SVG2/paths.html#PathDataClosePathCommand) command — which ends the current path by connecting it back to its initial point.

To start easy, let’s consider an example with 4 pizza slices:

```
[1](#hl-1-1)<svg overflow="visible" viewBox="-1 -1 2 2" xmlns="http://www.w3.org/2000/svg"> [2](#hl-1-2) <path d="M 0,0 L 0,-1 A 1,1,0,0,1,1,0 z" fill="IndianRed" stroke="black" stroke-width="0.01" /> [3](#hl-1-3) <path d="M 0,0 L 1,0 A 1,1,0,0,1,0,1 z" fill="LightCoral" stroke="black" stroke-width="0.01" /> [4](#hl-1-4) <path d="M 0,0 L 0,1 A 1,1,0,0,1,-1,0 z" fill="Salmon" stroke="black" stroke-width="0.01" /> [5](#hl-1-5) <path d="M 0,0 L -1,0 A 1,1,0,0,1,0,-1 z" fill="DarkSalmon" stroke="black" stroke-width="0.01" /> [6](#hl-1-6)</svg> 
```

It renders to:

$ r_1 = 1 $

###### *Note*

*I added `overflow="visible"` to the SVG tag, because — with a non-zero stroke — the image is *slightly* bigger than its `viewBox` and the stroke lines get cut off at $(-1, 0)$, $(1, 0)$, $(0, 1)$, and $(-1, 0)$.*

*We can see an interesting patter here: coordinates of the [**“lineto”**](https://www.w3.org/TR/SVG2/paths.html#PathDataLinetoCommands) command are the same, as the coordinates of the [elliptical arc curve](https://www.w3.org/TR/SVG2/paths.html#PathDataEllipticalArcCommands) command of the *previous* element.*

**Ok, but how to divide these slices into halves?* — you may ask.*

*To do that we need to find the points in the middle of the arcs.*

*<svg xmlns="http://www.w3.org/2000/svg" id="slices-with-points" overflow="visible" viewBox="-1 -1 2 2"><text x=".70710678118" y="-.70710678118" text-anchor="start" dominant-baseline="text-top">(ax, ay)</text><text x=".70710678118" y=".70710678118" text-anchor="start" dominant-baseline="hanging">(bx, by)</text><text x="-.70710678118" y="-.70710678118" text-anchor="end" dominant-baseline="text-top">(cx, cy)</text><text x="-.70710678118" y=".70710678118" text-anchor="end" dominant-baseline="hanging">(dx, dy)</text></svg>*

*$ r_1 = 1 $*

*Fortunately — since the circle’s middle point is $(0, 0)$ and the arc points are in the middle of their arcs — we know that the absolute value of their coordinates is the same. So, our diagram simplifies to:*

*<svg xmlns="http://www.w3.org/2000/svg" id="slices-with-same-points" overflow="visible" viewBox="-1 -1 2 2"><text x=".70710678118" y="-.70710678118" text-anchor="start" dominant-baseline="text-top">(a, -a)</text><text x=".70710678118" y=".70710678118" text-anchor="start" dominant-baseline="hanging">(a, a)</text><text x="-.70710678118" y="-.70710678118" text-anchor="end" dominant-baseline="text-top">(-a, -a)</text><text x="-.70710678118" y=".70710678118" text-anchor="end" dominant-baseline="hanging">(-a, a)</text></svg>*

*$ r_1 = 1 $*

*Great. 👌*

*Now we can use the equation of a circle to find the middle points.*

*$ \begin{aligned} (x - x_0)^2 + (y - y_0)^2 &= r^2 \\[0.25em] (a - 0)^2 + (a - 0)^2 &= 1^2 \\[0.25em] 2a^2 &= 1 \\[0.25em] a^2 &= \frac{1}{2} \\[0.25em] a &= \sqrt{\frac{1}{2}} \end{aligned} $*

*By adding four more pizza slices we get:*

*$ r_1 = 1 $*

### *[#](#layers-of-pizza) Layers of pizza*

*The next step is to create additional (smaller) layers of circle sectors. Let’s say we want three circles; there’re two obvious ways of selecting division points: equal radii, or equal areas. Equal radii is simpler, so let’s try this one first.*

*If the outermost circle has a radius of $1$, then the middle circle will have a radius of $0.\overline{6}$, and the smallest one will have $0.\overline{3}$.*

*$ r_1 = 1 \quad\text{and}\quad r_2 = 0.\overline{6} \quad\text{and}\quad r_3 = 0.\overline{3} $*

*Not that it matters, but the smallest circle is a little *too* small for my liking. But, before we decide, we need to see circles with equal areas first.*

*$ \begin{aligned} \Pi r_1^2 - \Pi r_2^2 = \Pi r_2^2 - \Pi r_3^2 \quad&\text{and}\quad \Pi r_2^2 - \Pi r_3^2 = \Pi r_3^2 \\[0.25em] 1 - r_2^2 = r_2^2 - r_3^2 \quad&\text{and}\quad r_2^2 - r_3^2 = r_3^2 \\[0.25em] 1 = 2r_2^2 - r_3^2 \quad&\text{and}\quad r_2^2 = 2r_3^2 \\[0.25em] 1 = 3r_3^2 \quad&\text{and}\quad r_2^2 = 2r_3^2 \\[0.25em] r_3 = \sqrt{\frac{1}{3}} \quad&\text{and}\quad r_2 = \sqrt{\frac{2}{3}} \end{aligned} $*

*Which gives us this avatar:*

*$ r_1 = 1 \quad\text{and}\quad r_2 = \sqrt{\frac{2}{3}} \quad\text{and}\quad r_3 = \sqrt{\frac{1}{3}} $*

*Which is also not ideal, but the other way around&mldr; I was tinkering with the radii a bit and I think I found a [middle ground](https://en.wikipedia.org/wiki/Arithmetic_mean) with which I’m happy.*

*$ r_1 = 1 \quad\text{and}\quad r_2 = \frac{0.\overline{6} + \sqrt{\frac{2}{3}}}{2} \quad\text{and}\quad r_3 = \frac{0.\overline{3} + \sqrt{\frac{1}{3}}}{2} $*

*Radii in the circles above are the arithmetic average of the equal radii variant and the equal areas variant.*

## *[#](#working-with-colors) Working with colors*

*With the SVG structure out of the way we can focus on selecting quasi-random colors for the circle segments based on the identifier’s hash. [SHA-256](https://en.wikipedia.org/wiki/SHA-2) hashes have 32 bytes; it’s more than we need — assuming we need a single byte per circle sector — which gives us another benefit: we can slide in the 4^(th) circle, if we want to.*

*For now though, let’s talk about the colors. If we’d use [HSL](https://en.wikipedia.org/wiki/HSL_and_HSV), we can split each byte into 4 pieces: 4 bits for hue (since it’s the most dominant), 2 bits for saturation, and 2 bits for lightness. Path’s [fill](https://www.w3.org/TR/SVG2/painting.html#SpecifyingFillPaint) property accepts any [paint](https://www.w3.org/TR/SVG2/painting.html#SpecifyingPaint) value, which basically means we need to format our color to a standard CSS’s hue.*

```
 *`[1](#hl-2-1)fn to_color(byte: u8)  -> String { [2](#hl-2-2) let  h  =  byte  >>  4; [3](#hl-2-3) let  s  =  (byte  >>  2)  &  0x03; [4](#hl-2-4) let  l  =  byte  &  0x03; [5](#hl-2-5) [6](#hl-2-6) let  normalized_h  =  (h  as  f32)  /  16.0; [7](#hl-2-7) let  normalized_s  =  (s  as  f32)  /  4.0; [8](#hl-2-8) let  normalized_l  =  (l  as  f32)  /  4.0; [9](#hl-2-9) [10](#hl-2-10) let  h  =  360.0  *  normalized_h; [11](#hl-2-11) let  s  =  20.0  +  80.0  *  normalized_s; [12](#hl-2-12) let  l  =  40.0  +  50.0  *  normalized_l; [13](#hl-2-13) [14](#hl-2-14) format!("hsl({h}, {s}%, {l}%)") [15](#hl-2-15)}`* 
```

*Alright, let’s generate an avatar for `"foo"` and see how it looks like.*

 *Avatar for "foo".* 

*Well, it does not look terrible, but it’s not eye-catching either. To be perfectly honest, we should expect something like this; [SHA-256](https://en.wikipedia.org/wiki/SHA-2) returns (and for that matter all other hash functions too) a quasi-random values. If we convert those *raw* bytes to fill colors, we’ll get quasi-random colors. To make it nicer we need to modify our algorithm slightly; we need to calculate a global theme, if you will, for the avatar, so that a part of circle segments' colors come from that theme. We can do that by folding all hash bytes into a single one with XOR.*

```
 *`[1](#hl-3-1)fn to_color(normalized_theme: f32,  byte: u8)  -> String { [2](#hl-3-2) let  h  =  byte  >>  4; [3](#hl-3-3) let  s  =  (byte  >>  2)  &  0x03; [4](#hl-3-4) let  l  =  byte  &  0x03; [5](#hl-3-5) [6](#hl-3-6) let  normalized_h  =  (h  as  f32)  /  16.0; [7](#hl-3-7) let  normalized_s  =  (s  as  f32)  /  4.0; [8](#hl-3-8) let  normalized_l  =  (l  as  f32)  /  4.0; [9](#hl-3-9) [10](#hl-3-10) let  h  =  360.0  *  normalized_theme  +  120.0  *  normalized_h; [11](#hl-3-11) let  s  =  20.0  +  80.0  *  normalized_s; [12](#hl-3-12) let  l  =  40.0  +  50.0  *  normalized_l; [13](#hl-3-13) [14](#hl-3-14) format!("hsl({h}, {s}%, {l}%)") [15](#hl-3-15)} [16](#hl-3-16) [17](#hl-3-17)fn calculate_theme(bytes: &[u8])  -> f32 { [18](#hl-3-18) let  theme  =  bytes.iter() [19](#hl-3-19) .fold(0u8,  |acc,  byte|  acc  ^  byte); [20](#hl-3-20) (theme  as  f32)  /  (u8::MAX  as  f32) [21](#hl-3-21)} [22](#hl-3-22) [23](#hl-3-23)fn generate_paths(hash: [u8;  32])  { [24](#hl-3-24) let  theme  =  calculate_theme(&hash); [25](#hl-3-25) [26](#hl-3-26) let  colors  =  hash.iter() [27](#hl-3-27) .cloned() [28](#hl-3-28) .map(|byte|  to_color(theme,  byte)) [29](#hl-3-29) .collect::<Vec<_>>(); [30](#hl-3-30) [31](#hl-3-31) unimplemented!("Generate SVG paths."); [32](#hl-3-32)}`* 
```

*This code renders:*

 *Avatar for "foo" with a theme.* 

*It looks rather good, if I say so myself. The solution works as it’s suppose to: it *most probably* produces different avatars for different identifiers ([hash collisions](https://en.wikipedia.org/wiki/Hash_collision) do happen) and they look acceptable. However, the rings seem to *phase* into each other — their colors are too similar. We could experiment with another theme: a ring theme — which would be a XOR-fold of all bytes that represent a single ring — and check the results.*

```
 *`[1](#hl-4-1)fn to_color(normalized_theme: f32,  normalized_ring_theme: f32,  byte: u8)  -> String { [2](#hl-4-2) let  h  =  byte  >>  4; [3](#hl-4-3) let  s  =  (byte  >>  2)  &  0x03; [4](#hl-4-4) let  l  =  byte  &  0x03; [5](#hl-4-5) [6](#hl-4-6) let  normalized_h  =  (h  as  f32)  /  16.0; [7](#hl-4-7) let  normalized_s  =  (s  as  f32)  /  4.0; [8](#hl-4-8) let  normalized_l  =  (l  as  f32)  /  4.0; [9](#hl-4-9) [10](#hl-4-10) let  h  =  360.0  *  normalized_theme [11](#hl-4-11) +  120.0  *  normalized_ring_theme [12](#hl-4-12) +  30.0  *  normalized_h; [13](#hl-4-13) let  s  =  20.0  +  80.0  *  normalized_s; [14](#hl-4-14) let  l  =  40.0  +  50.0  *  normalized_l; [15](#hl-4-15) [16](#hl-4-16) format!("hsl({h}, {s}%, {l}%)") [17](#hl-4-17)} [18](#hl-4-18) [19](#hl-4-19)fn calculate_theme(bytes: &[u8])  -> f32 { [20](#hl-4-20) let  theme  =  bytes.iter().fold(0u8,  |acc,  byte|  acc  ^  byte); [21](#hl-4-21) (theme  as  f32)  /  (u8::MAX  as  f32) [22](#hl-4-22)} [23](#hl-4-23) [24](#hl-4-24)fn generate_paths(hash: [u8;  32])  { [25](#hl-4-25) let  theme  =  calculate_theme(&hash); [26](#hl-4-26) let  ring_themes  =  hash [27](#hl-4-27) .windows(8) [28](#hl-4-28) .map(calculate_theme) [29](#hl-4-29) .collect::<Vec<_>>(); [30](#hl-4-30) [31](#hl-4-31) let  colors  =  hash [32](#hl-4-32) .iter() [33](#hl-4-33) .cloned() [34](#hl-4-34) .enumerate() [35](#hl-4-35) .map(|(idx,  byte)|  to_color(theme,  ring_themes[idx  %  8],  byte)) [36](#hl-4-36) .collect::<Vec<_>>(); [37](#hl-4-37) [38](#hl-4-38) unimplemented!("Generate SVG paths."); [39](#hl-4-39)}`* 
```

*With the final touch we get:*

 *Avatar for "foo" with a global theme and a ring theme.* 

*Which is exactly the thing that hides in the spoiler block at the top. If you restrained yourself and didn’t check it out: congratulations. Now feel free to do so. 😃*

## *[#](#conclusions) Conclusions*

*Working on this blog post has been an effort I wanted to make to better understand how SVGs work from the inside, as well as, a challenge to reverse-engineer ideas from [Representing SHA-256 Hashes As Avatars](https://francoisbest.com/posts/2021/hashvatars).*

*I’m not working on any service which would benefit from these avatars. If I happen to work on one in the future I’ll for sure like to try them out. To make this solution easier to re-use in other places, I’ve made a Rust crate called [`svg_avatars`](https://crates.io/crates/svg_avatars), which implements this exact solution. The crate is fairly minimal, so if anyone has any idea on how to enhance it, I’d love to hear from you/check out your PRs.*

*As a closing thought, one of the parameters of the [absolute elliptical arc curve](https://www.w3.org/TR/SVG2/paths.html#PathDataEllipticalArcCommands) command is the `sweep-flag` parameter. It allows you to determine, whether an arc should be a smiley face or a sad face. If the parameter is `1` — as it is in our case — then the arc is a sad face. However, if you’d flip the flags to be `0`, then all arcs become smiley faces&mldr;*

 **Smiley-face* avatar for "foo".* 

*&mldr;and the avatar [emerges](https://en.wikipedia.org/wiki/Emergence) as a spiderweb. Who does not love a single-line-change feature for Spooktober.*

</main>