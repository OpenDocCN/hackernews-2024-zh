<!--yml
category: 未分类
date: 2024-05-27 15:01:06
-->

# Planner programming blows my mind

> 来源：[https://www.hillelwayne.com/post/picat/](https://www.hillelwayne.com/post/picat/)

[Picat](http://picat-lang.org/) is a research language intended to combine logic programming, imperative programming, and [constraint solving](https://www.hillelwayne.com/post/minizinc/). I originally learned it to help with vacation scheduling but soon discovered its `planner` module, which is one of the most fascinating programming models I’ve ever seen.

First, a brief explanation of <dfn>logic programming</dfn> (LP). In imperative and functional programming, we take inputs and write algorithms that produce outputs. In LP and constraint solving, we instead provide a set of equations and find assignments that satisfy those relationships. For example:

```
main =>
  Arr = [a, b, c, a],
  X = a,
  member(X, Arr),
  member(Y, Arr),
  X != Y,
  println([X, Y]). 
```

Non-function identifiers that start with lowercase letters are “atoms”, or unique tokens. Identifiers that start with uppercase letters are variables. So `[a, b, c, a]` is a list of four atoms, while `Arr` and `X` are variables. So `Member(X, Arr)` returns true as you’d expect.

The interesting thing is `Member(Y, Arr)`. Y wasn’t defined yet! So Picat finds a value for Y that *makes* the equation true. Y could be any of `a`, `b`, or `c`. Then the line after that makes it impossible for `Y` to be `a`, so this prints either `ab` or `ac`. Picat can even handle expressions like `member(a, Z)`, instantiating Z as a list!

<dfn>Planning</dfn> pushes this all one step further: instead of finding variable assignments that satisfy equations, we find variable *mutations* that reach a certain end state. And this opens up some really cool possibilities.

To showcase this, we’ll use Picat to solve a pathing problem.

## The problem

We place a marker on the grid, starting at the *origin* (0, 0), and pick another coordinate as the *goal*. At each step we can move one step in any cardinal direction, but cannot go off the boundaries of the grid. The program is successful when the marker is at the goal coordinate. As a small example:

```
+---+
|   |
| G |
|O  |
+---+ 
```

One solution would be to move to (1, 0) and then to (1, 1).

To solve this with planning, we need to provide three things:

1.  A starting state `Start`, which contains both the origin and goal coordinates.
2.  A set of <dfn>action</dfn> functions that represent state transitions. In Picat these functions *must* all be named `action` and take four parameters: a current state, a next state, an action name, and a cost. We’ll see that all below.
3.  A function named `final(S)` that determines if S is a final state.

Once we define all of these, we can call the builtin `best_plan(Start, Plan)` which will assign `Plan` to the shortest sequence of steps needed to reach a final state.

### Our first implementation

```
import planner.
import util.

main =>
   Origin = {0, 0}
   , Goal = {2, 2}
   , Start = {Origin, Goal}
   , best_plan(Start, Plan)
   , println(Plan)
   . 
```

<details><summary>Explanation</summary>

`main` is the default entry point into a Picat program. Here we’re just setting up the initial state, calling `best_plan`, and printing `Plan`. `{a, b}` is the syntax for a Picat array, which is basically a tuple.

Every expression in a Picat body must be followed by a comma *except* the last clause, which must be followed with a period. This makes moving lines around *really annoying*. Writing it in that “bullet point” style helps a little.</details> 

Since `final` takes just one argument, we’ll need to store both the current position and the goal into said argument. Picat has great pattern matching so we can just write it like this:

```
final({Pos, Goal}) => Pos = Goal. 
```

<details><summary>Explanation</summary>

Without the pattern matching, we’d have to write it like this:

```
final(S) =>
  S = {Pos, Goal}
  , Pos = Goal
  . 
```

If we write a second `final` predicate, the plan succeeds if *either* `final` returns true.</details> 

Finally, we need to define the actions which the planner can take. We only need one action here.

```
action(From, To, Action, Cost) ?=>
  From = {{Fx, Fy}, Goal}
  , Dir = [{-1, 0}, {1, 0}, {0, -1}, {0, 1}] 
  , member({Dx, Dy}, Dir)                    % (a)
  , Tx = Fx + Dx
  , Ty = Fy + Dy
  , member(Tx, 0..10)                        % (b)
  , member(Ty, 0..10)                        % (b)
  , To = {{Tx, Ty}, Goal}
  , Action = {move, To[1]}
  , Cost = 1
  . 
```

<details><summary>Explanation</summary>

`From` is the initial state, `To` is the next state, `Action` is the name of the action— in this case, `move`. You can store metadata in the action, which we use to store the new coordinates.

Writing `action ?=>` instead of `action =>` makes `action` backtrackable, which I’ll admit I don’t *fully* understand? I’m *pretty sure* it means that if *this* definition of `action` pattern-matches but doesn’t lead to a viable plan, then Picat can try *other* definitions of action. This’ll matter more for later versions.

As with the introductory example up top, we’re using `member` to both *find* values (on line `(a)`) and *test values* (on lines `(b)`). Picat also has a non-assigning predicate, `membchk`, which just does testing. If I wasn’t trying to showcase Picat I could instead have use `membchk` for the testing part, which cannot assign.

`Cost` is the “cost” of the action. `best_plan` tries to minimize the total cost. Leaving it at 1 means the cost of a plan is the total number of steps.</details> 

And that’s it, we’re done with the program. Here’s the output:

```
> picat planner1.pi

[{move,{1,0}},{move,{2,0}},{move,{2,1}},{move,{2,2}}] 
```

That’s a little tough to read, so I had Picat output structured data that I could process into a picture.

```
main =>
  Origin = {0, 0}
  , Goal = {2, 2}
  , Start = {Origin, Goal}
  , best_plan(Start, Path)
- , println(Plan)
+ , printf("Origin: %w\n", Origin)
+ , printf("Goal: %w\n", Goal)
+ , printf("Bounds: {10, 10}\n")
+ , printf("Path: ")
+ , println(join([to_string(A[2]): A in Plan], ", "))
  . 
```

I used a [Raku script](src/format_path.raku) to visualize it. Here’s what we now get:

```
> raku format_path.raku -bf planner1.pi

+-----+
|     |
|     |
|  G  |
|  •  |
|O••  |
+-----+ 
```

To show that the planner can route around an “obstacle”, I’ll add a rule that the state *cannot* be a certain value:

```
 , Tx = Fx + Dx
  , Ty = Fy + Dy
+ , {Tx, Ty} != {2, 1} 
```

```
+-----+
|     |
|     |
| •G  |
| •   |
|O•   |
+-----+ 
```

Let’s comment that out for now, leaving this as our current version of the code:

<details><summary>Code</summary>

```
import planner.
import util.

main =>
  Origin = {0, 0}
  , Goal = {2, 2}
  , Start = {Origin, Goal}
  , best_plan(Start, Plan)
  % , println(Plan) 
  , printf("Origin: %w\n", Origin)
  , printf("Goal: %w\n", Goal)
  , printf("Bounds: {10, 10}\n")
  , printf("Path: ")
  , println(join([to_string(A[2]):  A in Plan], ", "))
  .

final(S) =>
  S = {Pos, Goal},
  Pos = Goal.

action(From, To, Action, Cost) ?=>
  From = {{Fx, Fy}, Goal}
  , Dir = [{-1, 0}, {1, 0}, {0, -1}, {0, 1}]
  , member({Dx, Dy}, Dir)
  , Tx = Fx + Dx
  , Ty = Fy + Dy
  % , {Tx, Ty} != {2, 1}
  , member(Tx, 0..10)
  , member(Ty, 0..10)
  , To = {{Tx, Ty}, Goal}
  , Action = {move, To[1]}
  , Cost = 1
  . 
```</details> 

### Adding multiple goals

Next I’ll add multiple goals. In order to succeed, the planner needs to reach every single goal in order. We start with one change to `main`:

```
main =>
  Origin = {0, 0}
- , Goal = {2, 2}
+ , Goal = [{2, 2}, {3, 4}] 
```

Goal now represents a “queue” of goals to reach, in order. Then we add a *new* action which removes a goal from our queue once we’ve reached it.

```
action(From, To, Action, Cost) ?=>
  From = {Pos, Goal}
  , Goal = [Pos|Rest]
  , To = {Pos, Rest}
  , Action = {mark, From[1]}
  , Cost = 1
  . 
```

<details><summary>Explanation</summary>

`[Head|Tail]` splits a list into the first element and the rest. Since `Pos` was defined in the line before, `Goal = [Pos|Rest]` is *only* true if the first goal on the list is equal to `Pos`. Then we drop that goal from our new state by declaring the new goal state to just be `Rest`.

(This is where backtracking with `?=>` becomes important: if we didn’t make the actions backtrackable, Picat would match on the `move` first and never `mark`.)</details> 

Since we’re now destructively removing goals from our list when we reach them, `final` needs to be adjusted:

```
final({Pos, Goal}) =>
- Pos = Goal.
+ Goal = []. 
```

And that’s it. We didn’t even have to update our first action!

```
+-----+
|   G |
|   • |
|  G• |
|  •  |
|O••  |
+-----+ 
```

<details><summary>Code</summary>

```
import planner.
import util.

main =>
  Origin = {0, 0}
  , Goal = [{2, 2}, {3, 4}]
  %, Goal = [{9, 2}, {0, 4}, {9, 6}, {0, 9}]
  , Start = {Origin, Goal}
  , best_plan(Start, Plan)
  , printf("Origin: %w\n", Origin)
  , printf("Goal: %w\n", Goal)
  , printf("Bounds: {10, 10}\n")
  , printf("Path: ")
  , println(join([to_string(A[2]):  A in Plan], ", "))
  .

final({Pos, Goal}) =>
  Goal = [].

action(From, To, Action, Cost) ?=>
  From = {{Fx, Fy}, Goal}
  , Dir = [{-1, 0}, {1, 0}, {0, -1}, {0, 1}]
  , member({Dx, Dy}, Dir)
  , Tx = Fx + Dx
  , Ty = Fy + Dy
  , member(Tx, 0..10)
  , member(Ty, 0..10)
  , To = {{Tx, Ty}, Goal}
  , Action = {move, To[1]}
  , Cost = 1
  .

action(From, To, Action, Cost) ?=>
  From = {Pos, Goal}
  , Goal = [Pos|Rest]
  , To = {Pos, Rest}
  , Action = {mark, From[1]}
  , Cost = 1
  . 
```</details> 

### Cost minimization

Going through the goals in order doesn’t always lead to the shortest *total* path.

```
main =>
  Origin = {0, 0}
- , Goal = [{2, 2}, {3, 4}]
+ , Goal = [{9, 2}, {0, 4}, {9, 6}, {0, 9}] 
```

```
+----------+
|G         |
|•         |
|•         |
|•••••••••G|
|         •|
|G•••••••••|
|•         |
|•••••••••G|
|         •|
|O•••••••••|
+----------+ 
```

What if we didn’t care about the order of the goals and just wanted to find the shortest path? Then we only need to change two lines:

```
action(From, T, Action, Cost) ?=>
  From = {Pos, Goal}
- , Goal = [Pos|Rest]
- , T = {Pos, Rest}
+ , member(Pos, Goal)
+ , T = {Pos, delete(Goal, Pos)}
  , Action = {mark, From[1]}
  , Cost = 1
  . 
```

Now the planner can delete any goal it’s passing over regardless of where it is in the `Goal` list. So Picat can “choose” which goal it moves to next so as to minimize the overall path length.

```
+----------+
|G•••••••••|
|•        •|
|•        •|
|•        G|
|•        •|
|G        •|
|•        •|
|•        G|
|•         |
|O         |
+----------+ 
```

Final code:

<details><summary>Code</summary>

```
import planner.
import util.

main =>
  Origin = {0, 0}
  , Goal = [{9, 2}, {0, 4}, {9, 6}, {0, 9}]
  , Start = {Origin, Goal}
  , best_plan(Start, Plan)
  , printf("Origin: %w\n", Origin)
  , printf("Goal: %w\n", Goal)
  , printf("Bounds: {10, 10}\n")
  , printf("Path: ")
  , println(join([to_string(A[2]):  A in Plan], ", "))
  .

final({Pos, Goal}) =>
  Goal = [].

action(From, To, Action, Cost) ?=>
  From = {{Fx, Fy}, Goal}
  , Dir = [{-1, 0}, {1, 0}, {0, -1}, {0, 1}]
  , member({Dx, Dy}, Dir)
  , Tx = Fx + Dx
  , Ty = Fy + Dy
  , member(Tx, 0..10)
  , member(Ty, 0..10)
  , To = {{Tx, Ty}, Goal}
  , Action = {move, To[1]}
  , Cost = 1
  .

action(From, To, Action, Cost) ?=>
  From = {Pos, Goal}
  , member(Pos, Goal)
  , To = {Pos, delete(Goal, Pos)}
  , Action = {mark, From[1]}
  , Cost = 1 
  . 
```</details> 

### Other variations

Picat supports a lot more variations on planning:

*   `best_plan(S, Limit, Plan)` caps the maximum cost at `Limit`— good for failing early.
*   For each `best_plan`, there’s a `best_plan_nondet` that finds every possible best plan.
*   `sequence(P, Action)` restricts the possible actions based on the current partial plan, so we can add restrictions like “you have to move twice before you turn”.

The coolest thing to me is that the planning integrates with all the other Picat features. I whipped up a quick demo that combines planning and constraint solving. The <dfn>partition problem</dfn> is an NP-complete problem where you partition a list of numbers into two equal sums. This program takes a list of numbers and finds the sublist with the largest possible equal partitioning:

<details><summary>Code</summary>

```
% got help from http://www.hakank.org/picat/set_partition.pi 
import planner.
import util.
import cp.

main =>
Numbers = [32, 122, 77, 86, 59, 47, 154, 141, 172, 49, 5, 62, 99, 109, 17, 30, 977]
  , if final(Numbers) then
      println("Input already has a partition!")
      , explain_solution(Numbers)
    else
        best_plan(Numbers, Plan)
      , printf("Removed: %w%n",[R: $remove(R, _) in Plan])
      , $remove(Last, FinalState) = Plan[Plan.length]
      , printf("Final: %w%n", FinalState)
      , explain_solution(FinalState)
    end
  .

final(Numbers) => get_solutions(Numbers) != [].

get_solutions(Numbers) = S =>
  X = new_list(Numbers.length)
  , X :: 0..1
  , X[1] #= 0 % symmetry breaking
  , sum(Numbers) #= 2*sum([Numbers[I]*X[I]: I in 1..Numbers.length])
  , S = solve_all([$limit(1)], X) 
  .

action(From, To, Action, Cost) =>
   member(Element, From)
   , To = delete(From, Element)
   , Action = $remove(Element, To)
   , Cost = Element
   .

explain_solution(Numbers) =>
  [Sol] = get_solutions(Numbers)
  , Left =  [Numbers[I]: I in 1..Numbers.length, Sol[I] = 0]
  , Right = [Numbers[I]: I in 1..Numbers.length, Sol[I] = 1]
  , printf("%s=%d%n", join([to_string(N): N in Left], "+"), sum(Left))
  , printf("%s=%d%n", join([to_string(N): N in Right], "+"), sum(Right))
  . 
```</details> 

```
Removed: [5,17]
Final: [32,122,77,86,59,47,154,141,172,49,62,99,109,30,977]
32+99+977=1108
122+77+86+59+47+154+141+172+49+62+109+30=1108 
```

This is all so mindblowing to me. It’s almost like a metaconstraint solver, allowing me to express constraints *on the valid constraints*.

## Should I use Picat?

Depends?

I would not recommend using Picat in production. It’s a research language and doesn’t have a lot of affordances, like good documentation or clear error messages. Here’s what you get when there’s no plan that solves the problem:

```
*** error(failed,main/0) 
```

But hey it runs on Windows, which is better than 99% of research languages.

Picat seems more useful as a “toolkit” language, one you learn to solve a specific class of computational problems, and where you’re not expecting to maintain or share the code afterwards. But it’s really good in that niche! There’s a handful of problems I struggled to do with regular programming languages and constraint solvers. Picat solves a lot of them quite elegantly.

## Appendix: Other planning languages

While originally pioneered for robotics and AI, “planning” is most-often used for video game AIs, where it’s called “Goal Oriented Action Planning” (GOAP). Usually it’s built as libraries on top of other languages, or implemented as a [custom search strategy](https://artint.info/3e/html/ArtInt3e.Ch3.html). You can read more about GOAP [here](https://web.archive.org/web/20140613121607/http://alumni.media.mit.edu/~jorkin/goap.html).

There is also [PDDL](https://planning.wiki/guide/whatis/pddl), a planning description language that independent planners take as input, in the same way that DIMACS is a description format for [SAT](https://www.hillelwayne.com/post/sudoku/).

*Thanks to [Predrag Gruevski](https://predr.ag/) for feedback. I first shared [my thoughts on Picat](https://buttondown.email/hillelwayne/archive/picat-is-my-favorite-new-toolbox-language/) on [my newsletter](https://buttondown.email/hillelwayne/). I write new newsletter posts weekly.*