<!--yml
category: 未分类
date: 2024-05-27 14:42:38
-->

# The sinusoidal tetris | andreinc

> 来源：[https://www.andreinc.net/2024/02/06/the-sinusoidal-tetris](https://www.andreinc.net/2024/02/06/the-sinusoidal-tetris)

Let’s play Tetris, but with a twist. No geometrical figures will *fall from the sky*. Instead, you control a [sinusoid](https://en.wikipedia.org/wiki/Sine_wave), defined by: \(f(x)=A*sin(\omega x + \varphi)\):

* * *

^(^([(Source code)](/assets/js/2024-02-06-the-sinusoidal-tetris/tetris.js)))

* * *

Controls

*   To increase the angular frequency, \(\omega\), press: `s`;
*   To decrease the angular frequency, \(\omega\), press: `x`;
*   To increase the amplitude, \(A\), press: `a`;
*   To decrease the amplitude, \(A\), press: `z`;
*   To increase the phase: \(\varphi\), press: `q`;
*   To decrease the phase: \(\varphi\), press: `w`;
*   To *drop* the sinusoid, press `p`;

* * *

To win the game, you need to reduce the signal as close to zero as possible. It’s hard but not impossible. There’s a current threshold of `unit * 0.3`. Surviving is not winning. The *Path of the Alternating Phases* is boredom.

You lose if the original signal spikes outside the game buffer (canvas).

A professional player turns off the suggestions, now enabled by default. If you are a savant, you can compute the [*Fourier Series Coefficients*](https://en.wikipedia.org/wiki/Fourier_series) in your head. Cancel that noise!

To better understand what is happening, check out [this first article of a series](/2024/04/24/from-the-circle-to-epicycles).

* * *

The game was developed using [p5js](https://p5js.org/).

The source code [(here)](/assets/js/2024-02-06-the-sinusoidal-tetris/tetris.js) is not something I am particularly proud of.

* * *

Some discussion from around the web:

* * *

^(This game is a joke I put together during a weekend. I’m sorry for the graphics.)