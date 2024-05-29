<!--yml
category: 未分类
date: 2024-05-27 17:09:36
-->

# A* Tricks for Videogame Path Finding | Tim Mastny

> 来源：[https://timmastny.com/blog/a-star-tricks-for-videogame-path-finding/](https://timmastny.com/blog/a-star-tricks-for-videogame-path-finding/)

Here’s a few tricks to make the algorithm faster and easier to implement.

### Implicit Graph Data Structure

In textbooks, graphs are represented by a list of nodes and an adjacency matrix or adjacency list. But you can be a little more flexible about representing adjacency nodes.

For example, let’s say our game screen has 256 * 240 pixels. We’ll call the coordinates of each pixel a node. Then we can say there are 8 adjacent pixels: up, down, left, right, and the 4 diagonals. The cardinal directions have a weight of 1, while the diagonals have a weight of the square root of 2 (~1.4).

We don’t need to create a huge adjacency list: we can generate it on the fly. Moreover, not every pixel is a valid position for the monster: it might be on a wall or occupied by another sprite. In that case, we can dynamically exclude that pixel from the adjacency list.

Here’s how this might look:

```
# suppose current is some data structure with x and y coordinates
neighbors = [current + dir for dir in [(-1, 0), (1, 0), (0, -1), (0, 1), (-1, -1), (-1, 1), (1, -1), (1, 1)]]

for neighbor in neighbors:
    if out_of_bounds(neighbor) or occupied(neighbor):
        continue
    # do something with neighbor
```

So we only need to generate the adjacency list for the nodes we are actually going to visit, and we don’t have to manually exclude adjacent nodes with a map editor.

### Geometry Informed Heuristics

You can also manually tune aspects of the algorithm based on the geometry of your map.

#### Step size

Above I talked about using the pixels as nodes, but in a 2d tile based game, you can use the tiles as nodes. This speeds up the search dramatically, since the monster can find a path to the player in fewer iterations.

With this method, the path really isn’t an exact sequence of steps: your monster likely isn’t moving at a speed of 1 tile per frame. Now the path is a series of *directions* the monster should go.

And that’s okay! As I said in the Dijkstra section, our monster doesn’t actually care about the exact path, which is changing each frame as the player moves. It just needs to move in a direction that could possibly reach the player (i.e. not straight into a wall).

#### Iteration depth

One cool feature about A*: once a node comes off the priority queue, we know that that node represents the last step in the best path we’ve seen so far. Therefore, if we stopped the algorithm at some fixed number of iterations, we would know that the resulting path is our best guess for the shortest path to the destination. So we can get reasonable progress without the running the algorithm to completion.

You need to tune this maximum iteration depth to the geometry of the level. For example, if the depth is too small you can still have monsters get stuck behind a wall:

![](img/3e5f1df81a892e847c511804471ebf12.png)

With a fixed iteration depth of 30 tiles, we can position the player so the monster gets traps and can’t make progress towards the player. How does this happen? Remember that A* is recomputed each frame. On the first frame once reaching the wall, the monster calculates that it should move down. But on the next frame, it calculates that it should move up. This casuses the monster to get stuck in a loop. However, as soon as the player moves into it’s range, the monster is able to correctly find a path to the player.

You can see this effect exaggerated with a fixed depth of `1`:

```
 frame 1    frame 2

4
3     | .        | m
2   p | m      p | .
1     |          |
0
```

The monster always returns to pxiel 2 because it’s the shortest (Euclidean) distance to the player.

If you wanted to be really fancy, you could pre-compute the maximum depth A* needs to find a path from any position on the map. Unlike the pre-computed Dijkstra, you only have to save that maximum, since you know A* will find a valid path in real time, given that maximum depth.