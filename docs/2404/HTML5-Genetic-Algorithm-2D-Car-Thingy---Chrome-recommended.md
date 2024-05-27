<!--yml
category: 未分类
date: 2024-05-27 13:36:43
-->

# HTML5 Genetic Algorithm 2D Car Thingy - Chrome recommended

> 来源：[https://rednuht.org/genetic_cars_2/](https://rednuht.org/genetic_cars_2/)

### But what is it?

The program uses a simple genetic algorithm to evolve random two-wheeled shapes into cars over generations. Loosely based on [BoxCar2D](http://boxcar2d.com/), but written from scratch, only using the same physics engine ([box2d](http://box2d.org/)).
seedrandom.js written by [David Bau](http://davidbau.com/). (thanks!)

### Controls

| Save Population | Saves current population locally. |
| Restore Saved Population | Restore a previously saved population. |
| Suprise | Toggles drawing, makes the simulation faster. |
| New Population | Keeps the generated track and restarts the whole car population. |
| Create new world with seed | The same seed always creates the same track, so you can agree on a seed with your friends and compete. :) |
| Mutation rate | The chance that each gene in each individual will mutate to a random value when a new generation is born. |
| Mutation size | The range each gene can mutate into. Lower numbers mean the gene will have values closer to the original. |
| Elite clones | The top n cars that will be copied over to the next generation. |
| View top replay | Pause the current simulation and show the top performing car. Click a second time to resume simulation. |

### Graph

| Red | Top score in each generation |
| Green | Average of the top 10 cars in each generation |
| Blue | Average of the entire generation |

### Genome

The genome consists of:

*   Shape (8 genes, 1 per vertex)
*   Wheel size (2 genes, 1 per wheel)
*   Wheel position (2 genes, 1 per wheel)
*   Wheel density (2 genes, 1 per wheel) darker wheels mean denser wheels
*   Chassis density (1 gene) darker body means denser chassis

### Blurb

This is not as deterministic as it should be, so your best car may not perform as well as it once did. The terrain gets more complex with distance.
I'm not in the mood to deal with checking if all scripts have loaded before running, so refresh the page if things seem whack.

### GitHub

The code is now on a [GitHub repository](https://github.com/red42/HTML5_Genetic_Cars). Feel free to contribute!

Originally written by [this guy](http://rednuht.org), now with contributions from patient people at GitHub.