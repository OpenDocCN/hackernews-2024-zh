<!--yml
category: 未分类
date: 2024-05-27 13:00:57
-->

# After AI beat them, professional go players got better and more creative

> 来源：[https://www.henrikkarlsson.xyz/p/go](https://www.henrikkarlsson.xyz/p/go)

###### A game of the board game Go in Japan 1876

* * *

For many decades, it seemed professional Go players had reached a hard limit on how well it is possible to play. They were not getting better. Decision quality was largely plateaued from 1950 to the mid-2010s:

Then, in May 2016, DeepMind demonstrated AlphaGo, an AI that could beat the best human Go players. This is how the humans reacted:

After a few years, the weakest professional players were better than the strongest players before AI. The strongest players pushed beyond what had been thought possible. EDIT: as Gwern points out, I’m misreading the graph. The graph shows the average move quality across the whole Go player population. The lines are not the population, but simply the uncertainty around the mean. So moves were getting better, but we do not know how that was distributed in the population.

Were the moves getting better because some players cheated by using the AI? No. They really were getting better.

And it wasn’t simply that they imitated the AI, in a mechanical way. They got more creative, too. There was an uptick in historically novel moves and sequences. [Shin et al](https://www.pnas.org/doi/epdf/10.1073/pnas.2214840120) calculate about 40 percent of the improvement came from moves that could have been memorized by studying the AI. But moves that deviated from what the AI would do also improved, and these “human moves” accounted for 60 percent of the improvement.

My guess is that AlphaGo’s success forced the humans to reevaluate certain moves and abandon weak heuristics. This let them see possibilities that had been missed before.

Something is considered impossible. Then somebody does it. Soon it is standard. This is a common pattern. Until Roger Bannister ran the 4-minute mile, the best runners clustered just above 4 minutes for decades. A few months later Bannister was no longer the only runner to do a 4-minute mile. These days, high schoolers do it. The same story can be told about the French composer Pierre Boulez. His music was considered unplayable until recordings started circulating on YouTube and elsewhere. Now it is standard repertoire at concert houses.

The recent development in Go suggests that superhuman AI systems can have this effect, too. They can prove something is possible and lift people up. This doesn’t mean that AI systems will not displace humans at many tasks, and it doesn’t mean that humans can always adapt to keep up with the systems—in fact, the human Go players are not keeping up. But the flourishing of creativity and skills tells us something about what might happen at the tail end of the human skill distribution when more AI systems come online. As humans learn from AIs, they might push through blockages that have kept them stalled and reach higher.

Another interesting detail about the flourishing in Go, which is teased out in [this paper by Shin, Kim, and Kim](https://escholarship.org/content/qt6q05n7pz/qt6q05n7pz.pdf), is that the trend shift actually happened 18 months *after*AlphaGo. This coincides with the release of [Leela Zero](https://en.wikipedia.org/wiki/Leela_Zero), an open source Go engine. Being open source Leela Zero allowed Go players to build tools, like Lizzie, that show the AI’s reasoning when picking moves. Also, by giving people direct access, it made it possible to do massive input learning. This is likely what caused the machine-mediated unleash of human creativity.

This is not the first time this kind of machine-mediated flourishing has happened. When DeepBlue beat the chess world champion Kasparov in 1997, it was assumed this would be a blow to human chess players. It wasn’t. Chess became more popular than ever. And the games did not become machine-like and predictable. Instead, top players like Magnus Carlsen became [more inventive than ever](https://www.wsj.com/articles/magnus-carlsen-ian-nepomniachtchi-world-chess-championship-computer-analysis-11639003641).

Our potential is greater than we realize. Even in highly competitive domains, like chess and GO, performance can be operating far below the limit of what is possible. *Perhaps* AI will give us a way to push through these limits in more domains.

Warmly
Henrik

Several of the points here build on comments made on a Twitter thread I made about this yesterday. Nabeel S. Qureshi ([Twitter](https://twitter.com/nabeelqu), [blog](https://nabeelqu.co)) read a draft and gave useful pointers.

* * *