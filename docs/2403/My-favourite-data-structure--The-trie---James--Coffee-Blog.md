<!--yml
category: 未分类
date: 2024-05-27 14:33:33
-->

# My favourite data structure: The trie | James' Coffee Blog

> 来源：[https://jamesg.blog/2024/01/16/trie/](https://jamesg.blog/2024/01/16/trie/)

Suppose you want to build a predictive text engine. Given a few letters, you want to predict the end of a word. Suppose we have the string "co". The next word could be:

*   Cobalt
*   Code
*   Coffee
*   Co-operate

Or many other words.

Given a dictionary of all words in the English language, you could find all the words that start with "co" and pick one to recommend. But how would you pick one to recommend?

A good answer to this is to use word probabilities. You could have a reference text with a range of different texts from which you calculate how likely a word is to be used. You could tune this based on actual usage, too. If someone types "cobalt" a lot (I don't know why they would, but okay :D), you could rank "Cobalt" higher when choosing a word to recommend for the string "co".

We are missing one piece: how do we do this efficiently?

That is where the trie comes in, my favourite data structure.

## What is a trie?

The trie is a deeply nested tree. The structure is commonly used for predictive text engines because you can represent all strings as a tree. Then, you can use word probabilities to decide on what word you should recommend. This could be done by using a reference corpus of millions of words from English publications to identify how common dfifferent words are.

For next word prediction, a trie might be built at the letter level. This means that every letter has a value and also links to every other possible letter that could come next.

You can add items to, search for items, and remove items from a trie.

## A trie, step by step

Let's mock up a trie to see how it works!

Our trie will contain two words:

Consider the following trie:

```
 {
	"c": {
		"*": 0,
		"o": {
			"*": 1.0,
			"b": {
				"*": 1.2,
				"a": {
					"*": 0,
					"l": {
						"*": 0,
						"t": {
							"*": 0.9
						}
					}
				}
			},
			"d": {
				"*": 0.1,
				"e": {
					"*": 7.3
				}
			}
		}
	}
} 
```

Every possible letter is a key in our trie. Above, our trie is represented as a dictionary. In a real implementation, a trie might be represented as a set of classes, as is common in tree-based algorithms.

Every key has two possible properties:

1.  `*`, which is the probability the word should be recommended, and;
2.  A dictionary that contains all possible next letters, or a number if there are no more letters.

Suppose we only want to get words that start with "co". Here is some pseudo code about how we would do this if our Trie was a dictionary:

```
 trie["c"]["o"] 
```

We could then recurse down all of the keys in "o" to calculate the next possible words. In this example, we can make two words:

We could refine our search by adding another character:

```
 trie["c"]["o"]["d"] 
```

The value of this would be:

```
 "*": 0.1,
"e": {
	"*": 7.3
} 
```

The `*` whose value is 0.1 represents "cod" (as in the fish). The "e" tells us there is one more letter that can follow "cod": "e". The value for this is "7.3".

Now, let's say we traverse our whole tree for "co" that we made earlier. We would end up with two values:

We can order these and choose the one with the highest probability for our next word.

## Let's make a trie! (Python edition)

If you want to use a Trie in Python, I recommend the [pygtrie](https://pygtrie.readthedocs.io/en/latest/) library, originally developed by Google. This library has pre-built utilities for accessing items in a trie, traversing the Trie, and more.

First, run:

```
 pip install pygtrie 
```

Then, create a new file with the following code:

```
 import pygtrie

trie = pygtrie.CharTrie()

# add an element
trie["code"] = 7.3
trie["cobalt"] = 0.9
trie["coffee"] = 10

print(trie.keys(prefix="co")) 
```

In this code, we create a trie with three elements:

At the end of the code, we search the trie to find all words that start with "co".

Our code returns:

```
 ['code', 'cobalt', 'coffee'] 
```

We can sort the words by probability using this code:

```
 results = sorted(results, key=lambda x: trie[x]) 
```

Here is the result:

```
 ['coffee', 'code', 'cobalt'] 
```

## Conclusion and resources

The trie is a tree data structure. In a trie, every node has a value for the node itself and a set of nodes that you can traverse. You can represent words as a trie to efficiently find all words that start with a given prefix. Because you can give each node a value, you can attach probabilities to each word in the trie. This is ideal for next word prediction, where you can extract candidates for a next word from the trie using letters someone has already typed and then rank them based on the probabilities associated with each node.

You can also find if a word is not in the trie efficiently. To do so, you would split up a word (i.e. "codep") into its letters and search for each one `trie["c"]["o"]["d"]["e"]["p"]`. If there is no value associated with the result, you know the word is not in the tree.

For a detailed breakdown of the trie data structure, refer to the [Trie Wikipedia page](https://en.wikipedia.org/wiki/Trie).

[Share this post on Hacker News](https://news.ycombinator.com/submitlink?u=https://jamesg.blog//2024/01/16/trie/&t=My%20favourite%20data%20structure:%20The%20trie).

[Share this post on Lobste.rs](https://lobste.rs/stories/new?url=https://jamesg.blog//2024/01/16/trie/&title=My%20favourite%20data%20structure:%20The%20trie).