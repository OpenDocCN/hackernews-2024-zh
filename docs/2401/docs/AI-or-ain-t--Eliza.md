<!--yml
category: 未分类
date: 2024-05-27 14:34:58
-->

# AI or ain't: Eliza

> 来源：[https://zserge.com/posts/ai-eliza/](https://zserge.com/posts/ai-eliza/)

# AI or ain't: Eliza

In the year 2023, AI took center stage in the media, stirring up discussions about whether it was mere hype or real progress.

However, the concept of [non-human intelligence](https://en.wikipedia.org/wiki/Extraterrestrial_intelligence) isn’t a recent fascination; it has been a dream since ancient times. As we learned more about how neurons communicate through electronic pulses in our brains, it seemed plausible to simulate our “intelligence” with similar electronic circuits. The term “machine intelligence” was coined in the 1950s, around the same time the [Turing test](https://en.wikipedia.org/wiki/Turing_test) was introduced.

This test (also known as the “imitation game”) suggests that AI can be considered genuinely intelligent if a human can’t tell whether they’re interacting with another human or a machine. Picture an interrogator in a room, chatting through a text interface, asking questions, and trying to figure out if their conversational partner is human or not. If the person talking to the computer believes it’s a human, even though it’s a machine, that signifies the machine is genuinely artificially intelligent.

## Eliza

One of the first computer programs that successfully passed the Turing test was Eliza. Created in 1966 by [Joseph Weizenbaum](https://web.stanford.edu/class/cs124/p36-weizenabaum.pdf), Eliza skillfully emulated the speech patterns of a psychotherapist in its conversations. Interestingly, Eliza still [outperforms ChatGPT-3.5](https://arstechnica.com/information-technology/2023/12/real-humans-appeared-human-63-of-the-time-in-recent-turing-test-ai-study/) in certain Turing test variations.

Eliza demonstrates how even the simplest algorithm can be just sufficient to appear intelligent. Let’s imagine a program that constantly prints “Hello, user!” when it starts. We can hardly consider it intelligent and a user will quickly start to suspect it’s just a hardcoded behaviour.

Now, imagine a program printing a random greeting from a predefined list. It will take more attempts to unveil its artificiality, but as soon as the interrogator starts asking questions, such a bot would appear too inadequate in comparison to human interaction.

## How Eliza works

Let’s recreate Eliza just the way it worked 57 years ago. I’ll share some [Go code](https://github.com/zserge/aint/tree/main/eliza), but you can easily adapt it to the other programming languages.

We start with a basic chatbot interface:

```
func response(input string) (output string) {
    return "I'm not sure I understand you fully"
}
func main() {
    fmt.Println("How do you do.  Please tell me your problem.")
    defer fmt.Println("Goodbye.  It was nice talking to you.")
    scanner := bufio.NewScanner(os.Stdin)
    for scanner.Scan() {
        reply := respond(scanner.Text())
        if reply == "" { // no reply = end of conversation
 break
        }
        fmt.Println(reply)
    }
} 
```

Here, we’re reading text from stdin line by line and sending each line to the `response()` function. This function’s job is to handle what the user says, find a suitable reply, and send it back.

Currently, our Eliza is pretty brainless; it doesn’t really understand anything the user says and can’t wrap up a conversation properly. So, let’s give it a bit more intelligence by introducing some stop words to help Eliza bid a proper goodbye.

```
var quit = []string{"bye", "goodbye", "done", "exit", "quit"}

func respond(input string) string {
  q := strings.ToLower(strings.TrimSpace(input))
  if contains(q, quit) {
    return ""
  }
  ...
}

// > How do you do.  Please tell me your problem. // : Bye // > Goodbye.  It was nice talking to you. 
```

## Knowledge base

To enhance Eliza’s conversational skills, we need to implant some knowledge into its brain. In our case knowledge will be stored in a very structured and oranised manner – as a sorted list of keywords, each accompanied with a set of possible transformation rules.

Transformation rules are predefined instructions and patterns to help Eliza modify and respond to specific user inputs. Each transformation rule consist of a pattern to match (decomposition) and instructions on how to reassemble the reply.

Here’s a simple rule example:

```
keyword:  "sorry"  rules:  - match:  "*"   reasmb:   - Please don't apologise.   - Apologies are not necessary.   - I've told you that apologies are not required.   - It did not bother me.  Please continue. 
```

This rule is triggered when the user input contains the word “sorry”. Pattern is a wildcard (`*`) that matches all input text. Reassembly rules are just static strings and any of them can be returned as a possible reply:

```
> How do you do.  Please tell me your problem.
: Oh sorry
> Please don't apologise. 
```

More complicated examples would use reassembly rules that include parts of the user input:

```
keyword: "you"
rules:
- match: "* you * me"
  reasmb:
  - "What makes you think I (2) you ?"
  - "You like to think I (2) you -- don't you ?"
  - "Really, I (2) you ?"
- match: "* you are *"
  - "What makes you think I am (2) ?" 
```

Consider user input *“I suspect you are a bot”*. Eliza will detect the keyword `you` in the input and will start looking for matching patterns belonging to that keyword.

The first pattern `* you * me` will not match the input, but the second one will. Given the input *“I suspect you are a bot”* and pattern `* you are *` the first wildcard would hold `I suspect` and the second wildcard would hold `a bot`. Now, the reassembly rule would replace `(2)` placehodler with the contents of the second wildcard replying with *“What makes you think I am a bot?"*.

Similarly for the input *“I think you might dislike me”* the reply could be *“Really, I might dislike you?"*.

# Synonyms

To avoid duplication of the rules, Eliza uses synonym groups in its patterns. For example, category `belief` may include words like “belief”, “feel”, “think”, “believe”, “wish” etc. Then in a pattern it becomes possible to refer to the whole category instead of individual words:

```
keyword: "i"
rules:
- match: "* i /belief i *"
  reasmb:
  - "Do you really think so ?",
  - "But you are not sure you (2).",
  - "Do you really doubt you (2) ?"), 
```

Here for the input *“I feel I am talking to a bot”* the reply could be *“But you are not sure you are talking to the bot”*.

You might wonder how “I am” turned into “you are”. Eliza has a list of preprocessing and postprocessing rules. Preprocessing helps to convert “I’m” into “I am” or fix some common typos. Postprocessing helps to invert the pronouns and verbs, i.e. “am → are”, “your → my”, “myself → yourself” etc.

Another trick Eliza uses to handle synonyms is a “goto” reassembly rule. For the given keyword some transformation rules might redirect Eliza to a different (related) keyword:

```
keyword: "apologise"
rules:
- match: "*"
  reasmb:
  - "=sorry"
keyword: "can"
rules:
- match: "* can you *"
  reasmb:
  - "You believe I can (2) don't you ?"
  - "=what"
keyword: "what"
rules:
- match: "*"
  reamb:
  - "Why do you ask ?"
  - "Does that question interest you ?" 
```

Here keyword “apologise” is treated just as if it was “sorry”. For the keyword “can” the first reply would be *“You believe I can&mldr;"* and the second attempt would navigate Eliza to a generic reply.

Finally, there is one last feature that Eliza employed to simulate intelligence: memorisation. Some transformation rules would ask Eliza to store replies in a memory stack. Later, when no suitable reply could be found these previous replies are popped from memory and reused:

```
> How do you do.  Please tell me your problem.
: My chatbot is misbehaving
> Lets discuss further why your chatbot is misbehaving.
: It pretends being intelligent while it's not
> Lets discuss further why your chatbot is misbehaving.
: See?
> Please go on. 
```

Here the first reply about chatbot misbehaving is memorised and repeated when Eliza is out of suitable replies.

And that’s all Eliza does. Knowledge base can be represented directly with Go code, but you can also store Eliza “scripts” as JSON or YAML files and parse those on startup:

```
type Keyword struct {
  Word   string
  Rank   int
  Decomp []Decomp
}

type Decomp struct {
  Match  string
  Save   bool
  Reasmb []string
}

var keywords = []Keyword{
  {
    "sorry", // keyword
 0,       // rank to sort the keywords
 []Decomp{
      Decomp{"*", false,
        []string{
          "Please don't apologise.",
          "Apologies are not necessary.",
          ...
        },
      },
    },
  },
  ...
}

var (
  pre = map[string]string{
    "dont":       "don't",
     ...
  }
    post = map[string]string{
    "am":       "are",
    "your":     "my",
    ...
  }
  quit = []string{"bye", "goodbye", "done", "exit", "quit"}
  syn  = map[string][]string{
    "be":       []string{"be", "am", "is", "are", "was"},
    "belief":   []string{"belief", "feel", "think", "believe", "wish"},
    ...
  }
  fallback = []string{
    "I'm not sure I understand you fully.",
    "Please go on.",
    ...
  }
) 
```

All of Eliza state can be stored in two variables: an index, pointing to the next available decomposition for each rule, and a memory of the latest stored replies:

```
var (
  index = map[string]int{}
  mem   []string
) 
```

Complete Eliza code is no more than three functions: a helper to pre/postprocess text, a pattern matching algorithm and a top-level response function to handle the rest of Eliza’s logic:

```
func replace(words []string, mapping map[string]string) (res []string) {
  for _, w := range words {
    if s, ok := mapping[w]; ok {
      res = append(res, strings.Fields(s)...)
    } else {
      res = append(res, w)
    }
  }
  return res
}

replace(strings.Fields("i think you're a machine"}, pre)
// -> ["i", "think", "you", "are", "a", "computer"] replace(replace(strings.Fields("i think you're a machine"}, pre), post)
// -> ["you", "think", "i", "am", "a", "computer"] 
```

Pattern matching can be implemented in a number of ways. The obvious one would be to use regular expressions, but that’s too high-level for Eliza. Another option would be to implement [a 35-line tiny matcher](https://benhoyt.com/writings/rob-pike-regex/) by [Rob Pike](https://en.wikipedia.org/wiki/Rob_Pike).

But since our text is already split into tokens we can build a similar recursive matcher that operates on words rather than characters:

```
func match(pat, words []string) ([]string, bool) {
  if len(pat) == 0 && len(words) == 0 {
    // Both, pattern and text are empty: match!
 return nil, true
  } else if len(pat) == 0 || (len(words) == 0 && (len(pat) != 1 || pat[0] != "*")) {
    // Pattern is empty but some words left, or
 // Words are empty but pattern has some text left: no match!
 return nil, false
  } else if pat[0] == "*" {
    // Wildcard: be greedy, as long as the "tails" match!
 for i := len(words); i >= 0; i-- {
      if m, ok := match(pat[1:], words[i:]); ok {
        g := strings.Join(replace(words[:i], post), " ")
        return append([]string{g}, m...), true
      }
    }
    return nil, false
  } else if strings.HasPrefix(pat[0], "/") {
    // Synonym: compare to one of the words in that category
 if slices.Contains(syn[pat[0][1:]], words[0]) {
      if m, ok := match(pat[1:], words[1:]); ok {
        return append([]string{words[0]}, m...), true
      }
    }
    return nil, false
  } else if pat[0] != words[0] {
    // Literal word in pattern does not match the text: no match!
 return nil, false
  }
  // Seems to match so far, compare the rest!
 return match(pat[1:], words[1:])
}

// // func split(s) []string { ... } // // match(split(""), split("")) -> true // match(split("foo"), split("foo")) -> true // match(split("foo"), split("bar")) -> false // match(split("foo"), split("bar")) -> false // match(split("a * b"), split("a b c d e b")) -> true (1)="b c d e" // match(split(* a * b *), split("a b c")) -> true (1)="", (2)="", (3)="c" 
```

After having all the knowledge encoded and these utility functions written we can now build the rest of Eliza’s brain:

```
func respond(q string) string {
  q = strings.ToLower(strings.TrimSpace(q))
  // Handle stop words
 if slices.Contains(quit, q) {
    return ""
  }
  // Split into words and preprocess
 words := replace(strings.Fields(q), pre)
  // Find a keyword
 for _, k := range keywords {
    if slices.Contains(words, k.Word) {
    nextKey:
      // Find matching transformation rule
 for i, d := range k.Decomp {
        if m, ok := match(strings.Fields(d.Match), words); ok {
          // Choose the next reassembly
 id := fmt.Sprintf("%s:%d", k.Word, i)
          reply := d.Reasmb[index[id]]
          index[id] = (index[id] + 1) % len(d.Reasmb)
          // Handle "goto" rules
 if strings.HasPrefix(reply, "=") {
            for _, nextk := range keywords {
              if nextk.Word == reply[1:] {
                k = nextk
                goto nextKey
              }
            }
          }
          // Replace placeholders with phrases from user input
 for i, s := range m {
            reply = strings.ReplaceAll(reply, fmt.Sprintf("(%d)", i+1), s)
          }
          // Memorise the reply, if needed
 if d.Save {
            mem = append(mem, reply)
          }
          return reply
        }
      }
    }
  }
  // Try to use memory before admitting that we don't have a clue what a user is talking about
 if len(mem) > 0 {
    reply := mem[len(mem)-1]
    mem = mem[:len(mem)-1]
    return reply
  }
  index["fallback"] = (index["fallback"] + 1) % len(fallback)
  return fallback[index["fallback"]]
} 
```

Full sources are available [on GitHub](https://github.com/zserge/aint/tree/main/eliza/) and you can play around with one of Eliza implementations [online](https://www.masswerk.at/elizabot/) to take a Turing test yourself.

# AI?

Clearly, Eliza is not an AI. But it’s been a huge success and caused many people treat it as a human. Being empathetic to its users and reflecting their language back to them, Eliza seems to be very understanding. Also it doesn’t tend to reveal much about itself, making it harder to discover that it’s merely a short list of hardcoded phrases. Silence is golden, indeed.

Next part: [Markov Chains](/posts/ai-markov/)

I hope you’ve enjoyed this article. You can follow – and contribute to – on [Github](https://github.com/zserge), [Mastodon](https://mastodon.social/@zserge), [Twitter](https://twitter.com/zsergo) or subscribe via [rss](/rss.xml).

*Jan 01, 2024*