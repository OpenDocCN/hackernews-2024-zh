<!--yml
category: 未分类
date: 2024-05-27 13:21:32
-->

# Natural Language Processing in Bash

> 来源：[https://massimo-nazaria.github.io/nlp.html](https://massimo-nazaria.github.io/nlp.html)

Let’s implement a Bash toolchain to generate random prose that resembles the text corpus by using the *n*-gram language model!

Basically, [NLP](https://en.wikipedia.org/wiki/Natural_language_processing) aims at teaching computers to understand and work with human language by using different techniques.

In what follows, we’re going to have a glimpse at the toolchain by training our model using the novel [*Moby-Dick*](https://gutenberg.org/ebooks/15).

First of all, let’s get the `bash-textgen/` folder from the repository and enter it:

```
$ git clone https://github.com/massimo-nazaria/bash-textgen.git
$ cd bash-textgen/ 
```

### Preprocessing

In a typical NLP process, a text corpus (i.e. input data) is preprocessed by organizing it into structured data before training mathematical models.

This involves tasks like tokenization (i.e. breaking the text into words) or data cleaning (i.e. removing any non-relevant text).

For our example, let’s extract the single words from `moby-dick.txt` by removing unnecessary characters and put the result in `words.txt`:

```
$ cat moby-dick.txt | ./words.sh > words.txt 
```

Initial 10 extracted words:

```
$ cat words.txt | head -n 10
moby
dick
by
herman
melville
chapter
loomings
call
me
ishmael 
```

As you can see, the initial words include: the title, the author full name, the 1st chapter title, and the incipit *Call me Ishmael*.

Precisely, `words.sh` performs the following text transformations:

1.  Make all input text lowercase;
2.  Remove non-alphabetical characters except for the periods `.`;
3.  Remove multiple whitespaces and period characters;
4.  Output one word (or period) per line.

### Training

During the training process, NLP models learn patterns and relationships in the language from the preprocessed training data.

This way, the resulting trained models can provide a variety of services like sentiment analysis and text generation.

#### The *N*-Gram Language Model

*N*-gram language models are easy to understand and implement.

While [considered state-of-the-art](https://en.wikipedia.org/wiki/Natural_language_processing#Neural_NLP_(present)) in the early days of NLP, nowadays more advanced NLP techniques outperform *n*-grams for text generation.

In spite of that, they still work very well for next-word suggestions or auto-completion in text editors.

Training data preparation consists in organizing the preprocessed words from the text corpus into *n*-tuples of consecutive words, namely *n*-grams.

Let’s do this by computing [bigrams](https://en.wikipedia.org/wiki/Word_n-gram_language_model#Bigram_model) (i.e. 2-grams) out of the extracted words:

```
$ cat words.txt | ./ngrams.sh 2 > bigrams.txt 
```

Initial 10 bigrams computed:

```
cat bigrams.txt | head -n 10
moby dick
dick by
by herman
herman melville
melville chapter
chapter loomings
loomings call
call me
me ishmael
ishmael . 
```

During the training process, the *n*-gram language model learns to predict the next word in a sentence based on the previous *n*-1 words.

Thus, with bigrams they learn to predict next word based on just the previous word in the sentence.

### Text Generation

Let’s generate text from the computed bigrams starting from the initial word “the”.

```
$ ./textgen.sh bigrams.txt "the" 
```

*Output:*

> the spare poles and more certain wild creatures to his retired whaleman as he never tell the pier heads to this inclined for the imposed and i lay them endless sculptures.

Not bad at all! The generated prose actually mimics the style of Herman Melville from his novel we initially used as our text corpus.

Let’s try it again with the initial word “man”:

```
$ ./textgen.sh bigrams.txt "man" 
```

*Output:*

> man from the reminiscence even for the thames tunnel then tow line not have to the whale and selecting our hemisphere.

It’s kind of surprising (and funny!) to see how in a few lines of Bash our tool can emulate the poetry of such a literary giant.

In particular, in the example above `textgen.sh` starts generating a sentence from a given initial word as follows:

Let “man” be the initial given word.

#### *Step 1: Get all the bigrams starting with “man”*

```
cat bigrams.txt | grep -e "^man " 
```

The result of the command above is a list of bigrams, e.g.:

```
man on
man receives
man enter
man distracted
man travelled
... 
```

Please note that the list of bigrams generally contains a lot of duplicates. Among them, some bigrams will show up much more frequently than the others.

And the most common bigrams will be the most likely to be selected in the next steps in order to extract our next word.

#### *Step 2: Shuffle all such bigrams*

```
cat bigrams.txt | grep -e "^man " | shuf 
```

We randomly rearrange the list of bigrams so as to avoid extracting always the same next word. Example of shuffled list:

```
man from
man who
man prefers
man interested
man slipped
... 
```

```
cat bigrams.txt | grep -e "^man " | shuf | head -n 1 
```

From the shuffled list above, we pick the 1st bigram:

In our example, “from” is our next word in the generated sentence!

Note that we could have chosen any bigram from the shuffled list, without loss of generality.

In fact, the most common bigrams will always be the more likely to be extracted, regardless of their position in the shuffled list.

#### *Final Step: Get the 2nd word from the bigram*

```
cat bigrams.txt | grep -e "^man " | shuf | head -n 1 | cut -d ' ' -f2` 
```

Clearly, the final command in the pipeline `cut -d ' ' -f2` extracts the 2nd word in the bigram:

The above generation steps repeat iteratively word-by-word until either the next word is a period character or no next-word is present.

Let’s play a little bit more with the toolchain by using [trigrams](https://en.wikipedia.org/wiki/Word_n-gram_language_model#Trigram_model), namely *n*-grams with *n*=3:

```
cat words.txt | ./ngrams.sh 3 > trigrams.txt 
```

In general, `ngrams.sh` accepts as its argument any natural integer starting from 2.

Let’s generate a random sentence starting from the initial 2 words “a man”.

```
$ ./textgen.sh trigrams.txt "a man" 
```

*Output:*

> a man gives himself out of this blubbering now we are about to begin from the hearts of whales is always under great and extraordinary difficulties that every one knows the fine carnation of their yet suspended boats.

Let’s now try the initial 2 words “by falling”:

```
$ ./textgen.sh trigrams.txt "by falling" 
```

*Output:*

> by falling in the best harpooneers of nantucket both messmates of mine tis not me all loveliness is anguish to me we are to credit the old craft s cockpits especially of such a whale ship which like a hot sun s pilot yesterday i wrecked thee and wrong not captain ahab if it might light upon for all deficiencies of that is a new born sight.

Let’s try with “one moment”:

```
$ ./textgen.sh trigrams.txt "one moment" 
```

*Output:*

> one moment to watch the same no the reason of numerous rude scratches altogether of an enormous practical resolution in facing death this old whale hunter of the word why don t you all the men swung in the deep yet is there any reason possessed the most ancient extant portrait anyways purporting to be tried out without being taken out of all voyages now or never for a time when they come from a drooping orchard twig.

What a fun!

## The Implementation

[***Get the toolchain usage and code!***](https://github.com/massimo-nazaria/bash-textgen "GitHub Repository")

Please read also *[“Unix Philosophy with an Example”](/unix-philosophy.html)* to learn how to compose multiple commands together as seen in this tutorial.