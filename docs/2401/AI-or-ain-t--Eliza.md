<!--yml

类别：未分类

日期：2024-05-27 14:34:58

-->

# AI 或不是：Eliza

> 来源：[`zserge.com/posts/ai-eliza/`](https://zserge.com/posts/ai-eliza/)

# AI 或不是：Eliza

在 2023 年，AI 成为媒体关注的焦点，引发了关于它究竟是炒作还是真正进步的讨论。

然而，[非人类智能](https://en.wikipedia.org/wiki/Extraterrestrial_intelligence)的概念并不是最近才出现的迷恋；自古以来，这一直是一个梦想。随着我们对大脑中神经元如何通过电子脉冲进行通信的了解越来越多，用类似的电子电路模拟我们的“智能”似乎是可行的。术语“机器智能”是在 20 世纪 50 年代提出的，大约与[Turing 测试](https://en.wikipedia.org/wiki/Turing_test)同时引入。

这个测试（也称为“模拟游戏”）表明，如果人类无法分辨他们是在与另一个人还是机器人交互，那么 AI 可以被认为是真正智能的。想象一个询问者在一个房间里，通过文本界面进行聊天，提问并试图弄清楚他们的对话伙伴是人还是机器。如果与计算机对话的人认为它是一个人，尽管它是一台机器，那么这就表明这台机器真正具有人工智能。

## Eliza

最早成功通过图灵测试的计算机程序之一就是 Eliza。由[Joseph Weizenbaum](https://web.stanford.edu/class/cs124/p36-weizenabaum.pdf)于 1966 年创造，Eliza 熟练地模拟了其对话中心理医生的语音模式。有趣的是，Eliza 在某些图灵测试变体中仍然[胜过 ChatGPT-3.5](https://arstechnica.com/information-technology/2023/12/real-humans-appeared-human-63-of-the-time-in-recent-turing-test-ai-study/)。

Eliza 演示了即使是最简单的算法也足以表现出智能的足够性。让我们想象一个程序，当它启动时不断打印“你好，用户！”我们几乎无法认为它是智能的，用户很快就会怀疑它只是一种硬编码的行为。

现在，想象一个程序从预定义列表中随机打印一个问候语。要揭示其人工性需要更多尝试，但只要询问者开始提问，这样的机器人将在与人类交互相比显得太不足够。

## Eliza 的工作原理

让我们重新创建 57 年前的 Eliza。我会分享一些[Go 代码](https://github.com/zserge/aint/tree/main/eliza)，但你也可以轻松地将其适应其他编程语言。

我们从一个基本的聊天机器人界面开始：

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

这里，我们逐行读取来自标准输入的文本，并将每一行发送到`response()`函数。这个函数的作用是处理用户说的话，找到一个合适的回复，并将其发送回去。

目前，我们的 Eliza 相当愚蠢；它实际上不理解用户说的任何话，并且无法适当地结束对话。因此，让我们通过引入一些停止词来为 Eliza 增加一点智能，以帮助 Eliza 适当地结束对话。

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

## 知识库

为了增强 Eliza 的对话技能，我们需要向其大脑植入一些知识。在我们的情况下，知识将以非常结构化和组织良好的方式存储 —— 作为一个关键词的排序列表，每个关键词都伴随着一组可能的转换规则。

转换规则是预定义的指令和模式，帮助 Eliza 修改和响应特定的用户输入。每个转换规则包括一个用于匹配的模式（分解）和有关如何重新组装回复的指令。

这是一个简单的规则示例：

```
keyword:  "sorry"  rules:  - match:  "*"   reasmb:   - Please don't apologise.   - Apologies are not necessary.   - I've told you that apologies are not required.   - It did not bother me.  Please continue. 
```

当用户输入包含单词 “sorry” 时，将触发此规则。模式是一个通配符 (`*`)，匹配所有输入文本。重新组装规则只是静态字符串，任何一个都可以作为可能的回复返回：

```
> How do you do.  Please tell me your problem.
: Oh sorry
> Please don't apologise. 
```

更复杂的示例会使用包含用户输入部分的重新组装规则：

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

考虑用户输入 *“我怀疑你是一个机器人”*。Eliza 将检测输入中的关键词 `你` 并开始寻找属于该关键词的匹配模式。

第一个模式 `* 你 * 我` 将不匹配输入，但第二个会。给定输入 *“我怀疑你是一个机器人”* 和模式 `* 你是 *`，第一个通配符会保持 `我怀疑`，第二个通配符会保持 `一个机器人`。现在，重新组装规则将使用第二个通配符的内容替换 `(2)` 的占位符，并回复 *“你为什么认为我是一个机器人？”*。

同样，在输入 *“我想你可能不喜欢我”* 时，回复可能是 *“真的吗，我可能不喜欢你？”*。

# 同义词

为了避免规则的重复，Eliza 在其模式中使用了同义词组。例如，类别 `belief` 可能包括诸如 `belief`、“feel”、“think”、“believe”、“wish” 等词语。然后在模式中，可以引用整个类别而不是单个词语：

```
keyword: "i"
rules:
- match: "* i /belief i *"
  reasmb:
  - "Do you really think so ?",
  - "But you are not sure you (2).",
  - "Do you really doubt you (2) ?"), 
```

在此输入 *“我感觉我在和一个机器人交谈”* 时，回复可能是 *“但你并不确定你在和机器人交谈”*。

你可能会想知道 “我是” 怎么变成了 “你是”。Eliza 有一个预处理和后处理规则列表。预处理有助于将 “I’m” 转换为 “I am” 或修正一些常见的拼写错误。后处理帮助反转代词和动词，即 “am → are”、“your → my”、“myself → yourself” 等。

Eliza 处理同义词的另一个技巧是 “goto” 重新组装规则。对于给定的关键词，一些转换规则可能会将 Eliza 重定向到一个不同的（相关的）关键词：

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

这里关键词 “apologise” 被视为 “sorry”。对于关键词 “can”，第一个回复将是 *“你相信我能…”*，而第二次尝试将引导 Eliza 到一个通用回复。

最后，还有一个 Eliza 用来模拟智能的特性：记忆。一些转换规则会要求 Eliza 将回复存储在内存堆栈中。后来，当找不到合适的回复时，这些先前的回复会从内存中弹出并重新使用：

```
> How do you do.  Please tell me your problem.
: My chatbot is misbehaving
> Lets discuss further why your chatbot is misbehaving.
: It pretends being intelligent while it's not
> Lets discuss further why your chatbot is misbehaving.
: See?
> Please go on. 
```

这里关于聊天机器人行为不端的第一个回复被记忆下来，并在 Eliza 缺少合适的回复时重复使用。

这就是 Eliza 所做的全部。知识库可以直接用 Go 代码表示，但您也可以将 Eliza 的“脚本”存储为 JSON 或 YAML 文件，并在启动时解析这些文件：

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

所有 Eliza 状态都可以存储在两个变量中：一个索引，指向每个规则下一个可用的分解，以及最新存储的回复的内存：

```
var (
  index = map[string]int{}
  mem   []string
) 
```

完整的 Eliza 代码不超过三个函数：一个用于预处理/后处理文本的辅助函数，一个模式匹配算法和一个顶层响应函数，用于处理 Eliza 剩余的逻辑：

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

模式匹配可以用多种方式实现。显而易见的是使用正则表达式，但对于 Eliza 来说，那太高级了。另一个选择是通过实现[一个 35 行的微小匹配器](https://benhoyt.com/writings/rob-pike-regex/)来实现，由[Rob Pike](https://en.wikipedia.org/wiki/Rob_Pike)提供。

但由于我们的文本已经被分成了标记，我们可以构建一个类似的递归匹配器，该匹配器操作单词而不是字符：

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

在将所有知识编码并编写这些实用函数之后，我们现在可以构建 Eliza 的其余大脑了：

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

完整的源代码可以在[GitHub 上](https://github.com/zserge/aint/tree/main/eliza/)找到，你可以在[线上](https://www.masswerk.at/elizabot/)玩弄其中一个 Eliza 实现，自己进行图灵测试。

# AI？

显然，Eliza 并不是人工智能。但它取得了巨大的成功，并导致许多人将其视为人类。对其用户怀有同情心，并将其语言反映回他们，Eliza 似乎非常理解人情。此外，它也不倾向于透露太多关于自己的信息，这使得发现它只是一系列硬编码短语更加困难。沉默确实是金子般的。

下一部分：马尔可夫链

希望您喜欢本文。您可以在[Github](https://github.com/zserge)、[Mastodon](https://mastodon.social/@zserge)、[Twitter](https://twitter.com/zsergo)上关注并贡献，或通过 rss 订阅。

*2024 年 1 月 1 日*
