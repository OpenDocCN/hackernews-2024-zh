<!--yml
category: 未分类
date: 2024-05-27 14:45:07
-->

# python - How to solve the "Mastermind" guessing game? - Stack Overflow

> 来源：[https://stackoverflow.com/questions/1185634/how-to-solve-the-mastermind-guessing-game](https://stackoverflow.com/questions/1185634/how-to-solve-the-mastermind-guessing-game)

Key tools: entropy, greediness, branch-and-bound; Python, generators, itertools, decorate-undecorate pattern

In answering this question, I wanted to build up a language of useful functions to explore the problem. I will go through these functions, describing them and their intent. Originally, these had extensive docs, with small embedded unit tests tested using doctest; I can't praise this methodology highly enough as a brilliant way to implement test-driven-development. However, it does not translate well to StackOverflow, so I will not present it this way.

Firstly, I will be needing several standard modules and **future** imports (I work with Python 2.6).

```
from __future__ import division # No need to cast to float when dividing
import collections, itertools, math 
```

I will need a scoring function. Originally, this returned a tuple (blacks, whites), but I found output a little clearer if I used a namedtuple:

```
Pegs = collections.namedtuple('Pegs', 'black white')
def mastermindScore(g1,g2):
  matching = len(set(g1) & set(g2))
  blacks = sum(1 for v1, v2 in itertools.izip(g1,g2) if v1 == v2)
  return Pegs(blacks, matching-blacks) 
```

To make my solution general, I pass in anything specific to the Mastermind problem as keyword arguments. I have therefore made a function that creates these arguments once, and use the **kwargs syntax to pass it around. This also allows me to easily add new attributes if I need them later. Note that I allow guesses to contain repeats, but constrain the opponent to pick distinct colours; to change this, I only need change G below. (If I wanted to allow repeats in the opponent's secret, I would need to change the scoring function as well.)

```
def mastermind(colours, holes):
  return dict(
    G           = set(itertools.product(colours,repeat=holes)),
    V           = set(itertools.permutations(colours, holes)),
    score       = mastermindScore,
    endstates   = (Pegs(holes, 0),))

def mediumGame():
    return mastermind(("Yellow", "Blue", "Green", "Red", "Orange", "Purple"), 4) 
```

Sometimes I will need to *partition* a set based on the result of applying a function to each element in the set. For instance, the numbers 1..10 can be partitioned into even and odd numbers by the function n % 2 (odds give 1, evens give 0). The following function returns such a partition, implemented as a map from the result of the function call to the set of elements that gave that result (e.g. { 0: evens, 1: odds }).

```
def partition(S, func, *args, **kwargs):
  partition = collections.defaultdict(set)
  for v in S: partition[func(v, *args, **kwargs)].add(v)
  return partition 
```

I decided to explore a solver that uses a *greedy entropic approach*. At each step, it calculates the information that could be obtained from each possible guess, and selects the most informative guess. As the numbers of possibilities grow, this will scale badly (quadratically), but let's give it a try! First, I need a method to calculate the entropy (information) of a set of probabilities. This is just -∑p log p. For convenience, however, I will allow input that are not normalized, i.e. do not add up to 1:

```
def entropy(P):
  total = sum(P)
  return -sum(p*math.log(p, 2) for p in (v/total for v in P if v)) 
```

So how am I going to use this function? Well, for a given set of possibilities, V, and a given guess, g, the information we get from that guess can only come from the scoring function: more specifically, how that scoring function partitions our set of possibilities. We want to make a guess that distinguishes best among the remaining possibilites — divides them into the largest number of small sets — because that means we are much closer to the answer. This is exactly what the entropy function above is putting a number to: a large number of small sets will score higher than a small number of large sets. All we need to do is plumb it in.

```
def decisionEntropy(V, g, score):
  return entropy(collections.Counter(score(gi, g) for gi in V).values()) 
```

Of course, at any given step what we will actually have is a set of remaining possibilities, V, and a set of possible guesses we could make, G, and we will need to pick the guess which maximizes the entropy. Additionally, if several guesses have the same entropy, prefer to pick one which could also be a valid solution; this guarantees the approach will terminate. I use the standard python decorate-undecorate pattern together with the built-in max method to do this:

```
def bestDecision(V, G, score):
  return max((decisionEntropy(V, g, score), g in V, g) for g in G)[2] 
```

Now all I need to do is repeatedly call this function until the right result is guessed. I went through a number of implementations of this algorithm until I found one that seemed right. Several of my functions will want to approach this in different ways: some enumerate all possible sequences of decisions (one per guess the opponent may have made), while others are only interested in a single path through the tree (if the opponent has already chosen a secret, and we are just trying to reach the solution). My solution is a "lazy tree", where each part of the tree is a generator that can be evaluated or not, allowing the user to avoid costly calculations they won't need. I also ended up using two more namedtuples, again for clarity of code.

```
Node = collections.namedtuple('Node', 'decision branches')
Branch = collections.namedtuple('Branch', 'result subtree')
def lazySolutionTree(G, V, score, endstates, **kwargs):
  decision = bestDecision(V, G, score)
  branches = (Branch(result, None if result in endstates else
                   lazySolutionTree(G, pV, score=score, endstates=endstates))
              for (result, pV) in partition(V, score, decision).iteritems())
  yield Node(decision, branches) # Lazy evaluation 
```

The following function evaluates a single path through this tree, based on a supplied scoring function:

```
def solver(scorer, **kwargs):
  lazyTree = lazySolutionTree(**kwargs)
  steps = []
  while lazyTree is not None:
    t = lazyTree.next() # Evaluate node
    result = scorer(t.decision)
    steps.append((t.decision, result))
    subtrees = [b.subtree for b in t.branches if b.result == result]
    if len(subtrees) == 0:
      raise Exception("No solution possible for given scores")
    lazyTree = subtrees[0]
  assert(result in endstates)
  return steps 
```

This can now be used to build an interactive game of Mastermind where the user scores the computer's guesses. Playing around with this reveals some interesting things. For example, the most informative first guess is of the form (yellow, yellow, blue, green), not (yellow, blue, green, red). Extra information is gained by using exactly half the available colours. This also holds for 6-colour 3-hole Mastermind — (yellow, blue, green) — and 8-colour 5-hole Mastermind — (yellow, yellow, blue, green, red).

But there are still many questions that are not easily answered with an interactive solver. For instance, what is the most number of steps needed by the greedy entropic approach? And how many inputs take this many steps? To make answering these questions easier, I first produce a simple function that turns the lazy tree of above into a set of paths through this tree, i.e. for each possible secret, a list of guesses and scores.

```
def allSolutions(**kwargs):
  def solutions(lazyTree):
    return ((((t.decision, b.result),) + solution
             for t in lazyTree for b in t.branches
             for solution in solutions(b.subtree))
            if lazyTree else ((),))
  return solutions(lazySolutionTree(**kwargs)) 
```

Finding the worst case is a simple matter of finding the longest solution:

```
def worstCaseSolution(**kwargs):
  return max((len(s), s) for s in allSolutions(**kwargs)) [1] 
```

It turns out that this solver will always complete in 5 steps or fewer. Five steps! I know that when I played Mastermind as a child, I often took longer than this. However, since creating this solver and playing around with it, I have greatly improved my technique, and 5 steps is indeed an achievable goal even when you don't have time to calculate the entropically ideal guess at each step ;)

How likely is it that the solver will take 5 steps? Will it ever finish in 1, or 2, steps? To find that out, I created another simple little function that calculates the solution length distribution:

```
def solutionLengthDistribution(**kwargs):
  return collections.Counter(len(s) for s in allSolutions(**kwargs)) 
```

For the greedy entropic approach, with repeats allowed: 7 cases take 2 steps; 55 cases take 3 steps; 229 cases take 4 steps; and 69 cases take the maximum of 5 steps.

Of course, there's no guarantee that the greedy entropic approach minimizes the worst-case number of steps. The final part of my general-purpose language is an algorithm that decides whether or not there are *any* solutions for a given worst-case bound. This will tell us whether greedy entropic is ideal or not. To do this, I adopt a branch-and-bound strategy:

```
def solutionExists(maxsteps, G, V, score, **kwargs):
  if len(V) == 1: return True
  partitions = [partition(V, score, g).values() for g in G]
  maxSize = max(len(P) for P in partitions) ** (maxsteps - 2)
  partitions = (P for P in partitions if max(len(s) for s in P) <= maxSize)
  return any(all(solutionExists(maxsteps-1,G,s,score) for l,s in
                 sorted((-len(s), s) for s in P)) for i,P in
             sorted((-entropy(len(s) for s in P), P) for P in partitions)) 
```

This is definitely a complex function, so a bit more explanation is in order. The first step is to partition the remaining solutions based on their score after a guess, as before, but this time we don't know what guess we're going to make, so we store all partitions. Now we *could* just recurse into every one of these, effectively enumerating the entire universe of possible decision trees, but this would take a horrifically long time. Instead I observe that, if at this point there is no partition that divides the remaining solutions into more than n sets, then there can be no such partition at any future step either. If we have k steps left, that means we can distinguish between at most n^(k-1) solutions before we run out of guesses (on the last step, we must always guess correctly). Thus we can discard any partitions that contain a score mapped to more than this many solutions. This is the next two lines of code.

The final line of code does the recursion, using Python's any and all functions for clarity, and trying the highest-entropy decisions first to hopefully minimize runtime in the positive case. It also recurses into the largest part of the partition first, as this is the most likely to fail quickly if the decision was wrong. Once again, I use the standard decorate-undecorate pattern, this time to wrap Python's *sorted* function.

```
def lowerBoundOnWorstCaseSolution(**kwargs):
  for steps in itertools.count(1):
    if solutionExists(maxsteps=steps, **kwargs):
      return steps 
```

By calling solutionExists repeatedly with an increasing number of steps, we get a strict lower bound on the number of steps needed in the worst case for a Mastermind solution: 5 steps. The greedy entropic approach is indeed optimal.

Out of curiosity, I invented another guessing game, which I nicknamed "twoD". In this, you try to guess a pair of numbers; at each step, you get told if your answer is correct, if the numbers you guessed are no less than the corresponding ones in the secret, and if the numbers are no greater.

```
Comparison = collections.namedtuple('Comparison', 'less greater equal')
def twoDScorer(x, y):
  return Comparison(all(r[0] <= r[1] for r in zip(x, y)),
                    all(r[0] >= r[1] for r in zip(x, y)),
                    x == y)
def twoD():
  G = set(itertools.product(xrange(5), repeat=2))
  return dict(G = G, V = G, score = twoDScorer,
              endstates = set(Comparison(True, True, True))) 
```

For this game, the greedy entropic approach has a worst case of five steps, but there is a better solution possible with a worst case of four steps, confirming my intuition that myopic greediness is only coincidentally ideal for Mastermind. More importantly, this has shown how flexible my language is: all the same methods work for this new guessing game as did for Mastermind, letting me explore other games with a minimum of extra coding.

What about performance? Obviously, being implemented in Python, this code is not going to be blazingly fast. I've also dropped some possible optimizations in favour of clear code.

One cheap optimization is to observe that, on the first move, most guesses are basically identical: (yellow, blue, green, red) is really no different from (blue, red, green, yellow), or (orange, yellow, red, purple). This greatly reduces the number of guesses we need consider on the first step — otherwise the most costly decision in the game.

However, because of the large runtime growth rate of this problem, I was not able to solve the 8-colour, 5-hole Mastermind problem, even with this optimization. Instead, I ported the algorithms to C++, keeping the general structure the same and employing bitwise operations to boost performance in the critical inner loops, for a speedup of many orders of magnitude. I leave this as an exercise to the reader :)

**Addendum, 2018:** It turns out the greedy entropic approach is not optimal for the 8-colour, 4-hole Mastermind problem either, with a worst-case length of 7 steps when an algorithm exists that takes at most 6!