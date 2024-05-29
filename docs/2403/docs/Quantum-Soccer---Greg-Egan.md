<!--yml
category: 未分类
date: 2024-05-27 14:51:52
-->

# Quantum Soccer — Greg Egan

> 来源：[https://www.gregegan.net/BORDER/Soccer/Soccer.html](https://www.gregegan.net/BORDER/Soccer/Soccer.html)

# Border Guards

## Quantum Soccer

* * *

### Quantum Soccer

In the game of Quantum Soccer, the aim is to shape the wave function of a quantum-mechanical “ball” so that the probability of it being inside one of the goals rises above a set threshold. This is achieved by using the motion of the players to alter the energy spectrum of the wave function: when a player moves across the field, the energy that this action provides (or absorbs) enables transitions between certain modes of the wave function. The pairs of modes involved depend on the player’s velocity; the exact rules are spelt out in the [mathematical details](SoccerNotes.html), but it’s easy to experiment using trial and error.

| Applet controls |
| The **velocity control** sets the *x*- and *y*-velocities of a player in metres per second. Click or drag the mouse on this control to set the velocities, then select a player. If a player has already been selected, the new velocity will be applied immediately. | *v*[*x*] = chosen x-velocity | The **goal control** selects the probability threshold for the ball being inside one of the goals in order to score. | The **resolution control** sets the size of the grid on which the wave function is computed. |
| *v*[*y*] = chosen y-velocity | If the **Auto-reverse** option is checked, players reverse automatically when they encounter an obstacle. Otherwise, they halt. | The **RESTART** button starts the game again from scratch. |
| If the **OK sounds** option is checked, sounds play when goals are scored. | The **demo** buttons play scripted demonstrations, in which the two teams cooperate to show how goals can be achieved. (Real play, of course, would include very different tactics.) |

Players are **selected** by clicking on them; click on an empty spot to de-select the player.

The **green panel** represents the playing field (which is 100 metres by 50 metres). The “ball” is a wave function confined to lie within the borders of the field (it can’t tunnel out, since the energy barrier is assumed to be infinitely high). The **wave function** is drawn as contours of the square of its magnitude, with phase indicated by the colour of the contour lines.

The **probability** of the ball lying within the goals is shown above each one. If the value crosses the threshold, the goal flashes white and the players and wave function are reset to the starting configuration. (If you have checked the **OK sounds** box, then sounds are played when goals are scored.)

If you choose **probabilistic scoring** rather than a percentage threshold in the **goal control**, then whenever the time is a multiple of 50 seconds, a measurement is performed on the ball to determine whether it is inside the left-hand goal, inside the right-hand goal, or neither – but which does not localise it any further than that. The wave function determines the probabilities for these three outcomes, but it cannot guarantee any particular result. If the ball is measured to be inside a goal, the score is counted, but the players are not reset to their starting configuration. The wave function is projected into a subspace that depends on the result of the measurement, such that the ball (at the moment of measurement) definitely lies in the region it was found to be in, to the extent that is possible given the finite number of eigenfunctions available with which to localise the wave.

The **blue panel** displays the spectrum of the wave function: the amplitude of each of the possible modes, with the phase indicated by a dial, and the frequency of the mode (in milliHertz, i.e. cycles per 1000 seconds) shown beneath it. Links are drawn in **grey** between pairs of states for which transitions are enabled by the current motion of the players; also, if you click-and-hold (or click-and-drag) on the velocity control, the links for the velocity you specify will appear **temporarily in red** on the spectrum panel.

You can **preview** a single mode by clicking on it: this displays it on the field, without affecting the wave function of the ball or registering any goals. You can also display the **best possible goal-scoring spectrum**, by holding down the **shift** key and clicking anywhere on the blue panel. This wave function produces peaks of over 50% probability in alternate goals whenever the time is a multiple of 50 seconds. (If more modes were included, these peaks would be even higher.)

#### Keyboard operation

The following functions can be performed with the keyboard, after clicking somewhere on the applet first, so it receives the keystrokes rather than your browser.

| Setting velocities with the keyboard |
| Keys that alter the *x*-velocity (when typed alone), or the *y*-velocity (when combined with the **SHIFT** key): | preceded by the **MINUS** (-) key for a negative velocity | the **digit** keys, 0-9, SELECT a velocity. |
| the **UP** arrow key INCREASES the velocity. |
| the **DOWN** arrow key DECREASES the velocity. |
| **R** REVERSES the velocity. |
| Keys that alter BOTH *x*- and *y*-velocities: | **B** reverses BOTH *x* and *y* velocities. |
| **Z** sets both *x* and *y* velocities to ZERO. |
| **H** HALTS all players. |
| Selecting players with the keyboard |
| **RIGHT ARROW** | selects the NEXT player, in team order |
| **LEFT ARROW** | selects the PREVIOUS player |
| **SHIFT RIGHT ARROW** | selects the next MOVING player |
| **SHIFT LEFT ARROW** | selects the PREVIOUS MOVING player |

* * *

* * *

Border Guards / Quantum Soccer / created Monday, 26 April 1999 / revised Sunday, 8 November 2009*Copyright © [Greg Egan](../../images/GregEgan.htm), 1999-2009\. All rights reserved.*