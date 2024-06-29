<!--yml

category: 未分类

date: 2024-05-27 13:40:28

-->

# ryjo.codes - Tour of CLIPS

> 来源：[https://ryjo.codes/tour-of-clips.html](https://ryjo.codes/tour-of-clips.html)

### 0\. 简介

过去一年半左右，我一直在学习关于 [CLIPS](https://www.clipsrules.net/) 编程语言。[NASA 的一个开发团队](https://clipsrules.net/AboutCLIPS.html) 在 80 年代中期创建了 CLIPS。它于 90 年代中期进入公共领域，并且自那时起一直由原始团队成员之一的 [Gary Riley](http://www.clipsrules.net/AboutTheDeveloper.html) 积极开发。

CLIPS 给人一种神秘感。没有像 npm 这样集中常见 CLIPS “库”的服务。关于这种语言的大部分写作表面上看起来都是学术性或更高层次的。更重要的是，CLIPS 的核心 [Rete 算法](https://en.wikipedia.org/wiki/Rete_algorithm) 对于理解可能有些令人望而却步。

不要被这些事情吓到；CLIPS 绝对值得学习。它的基于规则的方法是学习神经网络和人工智能的良好入门。通过这个教程，你将迈出进入规则引擎和专家系统的第一步。

首先，我们将讨论为什么应该考虑使用 CLIPS。CLIPS 的核心是 [Rete 算法](https://en.wikipedia.org/wiki/Rete_algorithm)。该算法对我们有几个非常好的特性。首先，它确定规则运行的顺序。这使我们可以专注于我们程序的领域业务逻辑，而不是软件如何运行。其次，Rete 网络“缓存”了软件将进行的常见计算。编写过应用内缓存的人都理解这一功能的好处。

CLIPS 的另一个巨大优势是主要开发者 [Gary Riley](http://www.clipsrules.net/AboutTheDeveloper.html) 提供的出色文档和支持。请查看 CLIPS 主页的 [文档部分](https://clipsrules.net/#documentation)，里面链接了各种级别的指南。此外，Riley 先生在各种论坛上活跃，为 CLIPS 提供支持。再次查看 CLIPS 主网站，找到 [各种支持途径的链接](https://clipsrules.net/#support)。

最后，CLIPS 仍在积极开发中；其最新版本 6.40 于 2021 年 5 月发布。相当不可思议！

#### 0.a 这个教程

本教程深受精彩的[A Tour of Go](https://go.dev/tour/welcome/1)的启发。点击代码下方的`(batch*)`按钮运行本教程中的示例。框中的代码可编辑，因此可以调整代码以测试您的好奇心。代码的输出将显示在按钮下方的框中。通过点击屏幕底部的链接在前后章节之间导航。左链接（←）将导航到前一章节，右链接（→）将进入下一章节。这两个链接之间将显示当前章节的名称。点击该链接将显示本书中所有章节的列表。点击其中一个章节将导航到相应的章节。

```
; Welcome to the tutorial! This is known as a "comment."
; It is meant to help developers
; understand a particular line of code better.
;
; Click the (batch*) button to run this code sample.
; You will see CLIPS code written in this area of the webpage
; throughout this tour.
;
; This is a WASM compilation of CLIPS run completely in your browser;
; no server involved!
(println "Hello, world")
```

### 1\. 说Hello

经典的"第一个程序"是"Hello, World"程序。这将向您展示如何从您的程序中输出一些文本。在右侧代码下方点击`(batch*)`按钮。您应该会看到文本出现在按钮下方的框中。

使用CLIPS，我们可以通过几种方式将文本打印到屏幕上。使用`(print)`将输出单词，而`(println)`将输出带有**结尾处的新行**。在右侧的代码示例中，您可以看到我们使用`(print)`来打印单词"Hello"。然后我们使用`(println)`来打印", World!"和一个新行字符。在此文本后打印的内容将出现在下一行。

`(printout t)`类似于`(print)`和`(println)`，但它允许您将文本发送到指定的"路由器"。在本例中，我们将文本发送到默认情况下发送文本的路由器`t`。我们还使用文本`crlf`作为`(printout`的最后一个参数。我们这样做是为了使我们的文本在行末"断开"；我们下一个打印的内容将出现在当前行下方。`crlf`代表"回车换行"，这意味着"将光标移动到下一行的开头"。

`(format t)`允许您打印包含"格式标志"的句子结构。`%s`和`%n`是格式标志，分别指定字符串和换行符。下一个参数（在本例中是"fun"）将替换`%s`的位置。`%n`将被新行字符替换。要了解更多这些"格式标志"，请查看[Basic Programming Guide](https://clipsrules.sourceforge.io/documentation/v640/bpg.pdf) -- 特别是**12.4.6格式化打印**一节 -- 以了解更多。

```
(print Hello)
(println ", World!")
(printout t "Welcome to this tour of CLIPS" crlf)
(format t "I hope that you have %s!%n" fun)
```

### 2\. 学习规则

规则是理解CLIPS工作原理的重要部分。规则由"前提"和"结论"组成。前提也称为"if部分"或"左侧部分"。此部分包含必须满足的条件，以使规则运行。

"后续"也称为"then部分"或"右侧部分"。这部分描述了规则运行时会发生什么。

这两部分由`=>`分隔开。前件在此符号之前，而后件在此符号之后。

在这个例子中，我们定义了一个名为`will-run-no-matter-what`的规则。这个特定规则在左手边没有任何内容，所以它将始终运行。右手边指定了一个`(println)`将会运行。

在规则被定义之后最后，有一个`(run)`。为了使其工作，请点击`(batch*)`按钮。

```
(defrule will-run-no-matter-what
      =>
      (println "Hello from a rule, world!"))
(run)
```

### 3\. Just the Facts

除了CLIPS中的规则，我们还有“Facts”。事实在CLIPS中表示“数据”。为了将数据放入CLIPS中，我们使用`(assert)`。一旦我们`(assert)`了事实，它们就被称为“在工作内存中”。

在右侧的代码中，我们`(assert)`了一个`(name`事实，并在其第二个字段中使用了`ryjo`。

我们接着用我们的老朋友`(println)`打印一行文本。

`(facts)` is a new one. We use this to print out the Facts stored in Working Memory. In this case, we see 1 fact printed with an index `1` as denoted by `f-1`.

最后，我们将`(retract)`刚刚断言的这个事实。我们可以通过其索引`1`引用这个事实。

注意：如果你连续点击`(batch*)`多次，将会出现错误。这是因为我们必须`(reset)`或`(clear)`环境以重置在工作内存中给事实分配的 ID。如果我们不这样做，我们所断言的事实将会有 ID `2`，而不是 `1`。如果你点击`(reset)`或`(clear)`，然后再次点击`(batch*)`，代码将按预期运行。

**奖励**：如果你在完成[第 5 章](#5)后返回此章节，你可能会看到多于 1 个事实返回。这是因为我们在代码顶部没有`(clear)`来首先“清理”环境。你将在[第 18 章](#18)中了解更多关于`(clear)`的信息。

```
(assert (name ryjo))
(println "The current facts: ")
(facts)
(retract 1)
(println "After retracting: ")
(facts)
```

### 4\. Running the Engine

“Working Memory”中存储的“Facts”可用于满足规则的左手边（LHS）。记住：当规则的左手边满足时，规则的右手边（RHS）将运行。

在右侧的代码中，我们使用`(defrule)`创建了一个名为`greet`的规则。当一个匹配`(name ?)`形式的事实输入到工作内存中时，这个规则将“激活”。一旦我们`(run)`规则引擎，如果系统中存在一个`name`事实与第二字段，则该规则的右手边将运行。

接下来，我们在其第二个字段中`assert`了一个`name`事实，并赋予其值`ryjo`。一旦这样做，`greet`规则将被“激活”；当我们下次`(run)`我们的规则引擎时，该规则的右手边将被执行。

**奖励**：如果你在完成[第 5 章](#5)后返回此章节，你可能会看到多于 1 个问候语。这是因为我们在代码顶部没有`(clear)`来首先“清理”环境。你将在[第 18 章](#18)中了解更多关于`(clear)`的信息。

```
(defrule greet
      (name ?name)
      =>
      (println "Hello " ?name "!"))
(assert (name ryjo))
(run)
```

### 5\. Negative, Ghostrider

有时候，你希望在“Working Memory”中有一个事实不匹配某个模式时激活一个规则。波浪符（`~`）字符用于否定给定的模式。这就像是在说“一个事实不在这里”。

请注意，我们无法像在[第四章](#4)中那样在此规则的RHS中使用不匹配的名称。我们将在[第6章](#6)讨论如何使用这些名称。

首先，我们使用`(clear)`清除CLIPS环境中所有之前定义的规则和事实。然后，我们使用`(assert`两个`(name`s: `Zach`和`Zethus`。最后，我们的`non-ryjos-only`规则将在工作内存中进入匹配模式`(name ~ryjo)`的事实时激活。这个模式可能在英语中被解读为“以`name`开头，其第二个字段不含`ryjo`。”最后，这个规则的RHS打印文本“Hello, not ryjo!”

**额外奖励:** 在执行此代码之后`(batch*)`，尝试导航到[第四章](#4); 您将看到先前定义的`greet`规则适用于这两个新名称。这是因为本教程中的示例使用相同的CLIPS环境。在此示例中，我们在顶部使用`(clear)`，它会删除前面示例中定义的所有内容。在后续章节中，您将看到这种惯例更多。

```
(clear)

(assert (name Zach) (name Zethus))

(defrule non-ryjos-only
  (name ~ryjo)
  =>
  (println "Hello, not ryjo!"))

(run)
```

### 6\. 捕获名称

现在，我们将结合两个先前介绍的概念使用`&`。首先，我们将使用`?n`捕获第一个字段的值（[第四章](#4)）。接下来，我们将用`~`指定“不是ryjo”（[第五章](#5)）。`&`符号充当“和”的作用。我们可以将模式`(name ?n&~ryjo)`解读为“一个值，它不是ryjo。”

此外，我们`(assert)`三个事实。只有其中两个将激活我们在其下定义的`non-ryjos-only`规则。有一点需要注意：我们只需调用`(assert)`一次，就能将三个新事实放入工作内存。

在这里，我们定义了第二个规则`ryjo`，以展示`non-ryjos-only`规则的“相反”。当一个事实进入工作内存，其看起来像`(name ryjo)`时，这个规则将被激活。

注意本示例中规则不按照编写顺序执行。这是因为**规则运行的顺序是由算法确定的**。更多关于CLIPS如何确定规则执行顺序的信息，请参阅[基本编程指南](https://clipsrules.sourceforge.io/documentation/v640/bpg.pdf)的第5.2节“规则执行的基本循环”。我们可以使用`(salience`来显式声明规则运行的顺序。我们将在[第15章](#15)讨论这个问题。

```
(clear)

(assert
  (name Gabe)
  (name ryjo)
  (name John))

(defrule non-ryjos-only
  (name ?n&~ryjo)
  =>
  (println "Hello, " ?n "!"))

(defrule ryjo
  (name ryjo)
  =>
  (println "Oh hey there"))

(run)
```

### 7\. 与/或

我们可以利用“连接约束”来更好地细化我们规则的左侧。例如，在代码编辑器中的`ors`规则中，我们匹配第二个字段匹配`Gabe`或`Peter`的事实。这可以解读为“一个`name`事实，其第一个字段我们存储为变量`?name`，并且匹配`Gabe`或`Peter`。”

在`and-nots`规则中，我们将`name`事实的第二个字段存储在变量`?name`中。这一次，我们说“第一个字段的值不能匹配`ryjo`，也不能匹配`Zach`，也不能匹配`Zethus`。”

```
(clear)

(assert
  (name Gabe)
  (name ryjo)
  (name John)
  (name Zach)
  (name Zethus)
  (name Peter))

(defrule ors
  (name ?name&Gabe|Peter)
  =>
  (println "Greetings, " ?name))

(defrule and-nots
  (name ?name&~ryjo&~Zach&~Zethus)
  =>
  (println "Hi there, " ?name))

(run)
```

### 8\. 谓词约束

谓词约束听起来很复杂，但它们非常直观。基本上，它们允许您进一步指定字段所包含的值。例如，根据我们当前的知识，我们可以为诸如 `(name ?n&ryjo)` 的值指定*确切*匹配。谓词条件允许我们对值执行函数以确定是否匹配。

在本章的代码示例中，我们编写了一个描述允许进入酒吧的基本规则。在美国，我们要求一个人年满 21 岁。如果他们未满 21 岁，则不允许他们进入酒吧。

注意 `?variable&` 的熟悉形式。我们在这里使用 `and` 符号 (`&`) 来表示 `?variable` 必须符合特定条件。这使我们可以将其解读为 "一个人的年龄及其大于或等于 21" 在 `allow-entry-to-bar` 规则中。

还请注意，我们利用了工作内存中的两个不同事实来激活我们的规则：`name` 和 `age` 事实。此外，请注意，在规则的 LHS 中，我们使用 `?person` 来指定这些事实的第一个字段必须匹配。由于 `?person` 在这两个条件中都使用了，我们称此规则在这些事实的第一个字段具有相同值时为 "激活"。

例如，`(name ryjo)` 事实和 `(age ryjo 32)` 事实将一起激活 `allow-entry-to-bar` 规则。这些事实的第一个字段匹配的值相同，并且数字 `32` 大于或等于 `21`。事实 `(name ryjo)` 和 `(age someone 20)` 将不会激活 `no-entry-to-bar` 规则。数字 `20` 确实小于 `21`，但这两个事实不会一起激活规则。然而，两个事实 `(name someone)` 和 `(age someone 20)` 将会。如谓词条件 `(>= ?a 21)` 所示，`20` 确实小于 `21`，并且符号 `someone` 在两个事实中匹配。

```
(clear)

(defrule allow-entry-to-bar
  (name ?person)
  (age ?person ?a&:(>= ?a 21))
  =>
  (println ?person " is over 21 years old and can enter the bar!"))

(defrule no-entry-to-bar
  (name ?person)
  (age ?person ?a&:(< ?a 21))
  =>
  (println ?person " is not allowed in (under 21)."))

(assert
  (name ryjo) (age ryjo 32)
  (name someone) (age someone 20))

(run)
```

### 9\. 测试条件

这个例子看起来像前一节。我们不再使用 `&:`，而是使用 `(test` 条件元素来定义我们的约束条件。

该示例的输出应该与上一个示例非常相似。

```
(clear)

(defrule allow-entry-to-bar
  (name ?person)
  (age ?person ?a)
  (test (>= ?a 21))
  =>
  (println ?person " is over 21 years old and can enter the bar!"))

(defrule no-entry-to-bar
  (name ?person)
  (age ?person ?a)
  (test (< ?a 21))
  =>
  (println ?person " is not allowed in (under 21)."))

(assert
  (name ryjo) (age ryjo 32)
  (name someone) (age someone 20))

(run)
```

### 10\. 一个 forall

有时您想知道工作内存中的所有事实是否与给定模式匹配。 `(forall` 提供了这种检查。在我们的例子中，如果每个工作内存中的 `(name` 事实都可以与 `(age` 事实匹配（反之亦然），`(forall` 将匹配。

在我们的例子中，事实 `(name another)` 没有与匹配值 `?n` 的 `(age` 事实。因此，我们的引擎告诉我们 "名单已被拒绝"。

```
(clear)

(assert
  (name ryjo) (age ryjo 32)
  (name someone) (age someone 20)
  (name another))

(defrule accept-roster
  (forall (name ?n) (age ?n ?))
  =>
  (println "Roster has been accepted"))

(defrule decline-roster
  (not (forall (name ?n) (age ?n ?)))
  =>
  (println "Roster has been declined"))

(run)
```

### 11\. 外星人存在

这是一个稍长的例子，但它结合了几个方面。 `(exists` 将在工作内存中的事实与其中的模式匹配一次时匹配。只要条件继续匹配，它将不会再次运行。在我们的例子中，我们匹配 "任何以 `(age` 开头并跟随两个值的事实"。当我们使用 `?` 时，这些值是什么并不重要；重要的是它们是否存在。

尝试修改我们的 `any-age-reported` 规则，捕获 `?name` 并在规则的 RHS 中 `(println ?name)`。

我们还使用 `(forall` 来确定每个用户何时报告了他们的年龄。

放弃控制代码执行顺序可能看起来有点奇怪。请记住，这使我们能够专注于业务逻辑而不是领域控制流。确保阅读 [Basic Programming Guide](https://clipsrules.sourceforge.io/documentation/v640/bpg.pdf) 中的 **性能注意事项** 部分，学习如何与此方法编写代码的和谐性。

```
(clear)

(defrule any-age-reported
  (exists (age ? ?))
  =>
  (println "Users have begun to report their age"))

(defrule no-ages-reported
  (not (age ? ?))
  =>
  (println "No user has reported their age yet..."))

(defrule reported-age
  (name ?n)
  (age ?n ?a)
  =>
  (format t "%s has reported their age: %d%n" ?n ?a))

(defrule has-not-reported-age
  (name ?n)
  (not (age ?n ?))
  =>
  (println ?n " has not yet reported their age"))

(defrule all-ages-reported
  (forall (name ?n) (age ?n ?))
  =>
  (println "Everyone in the system has reported their age"))

(assert
  (name ryjo)
  (name someone)
  (name another))

(println "First run results:")
(println "------------------")
(run)
(println " ")

(assert (age ryjo 32))

(println "Second run results:")
(println "-------------------")
(run)
(println " ")

(assert (age someone 20)
        (age another 52))

(println "Third run results:")
(println "------------------")
(run)
```

### 12\. 你读取了(line)我吗？

有时，您希望从用户那里获取输入，而不是"硬编码"他们的姓名、年龄和其他细节。我们可以使用 `(readline)` 从用户那里获取输入。

在这个例子中，我们从用户那里获取输入，然后告诉他们他们输入了什么。注意：当您运行此示例时，屏幕上将弹出一个提示框。将您的文本输入到字段中，然后点击"OK"按钮。

类似地，`(read)` 将从用户输入中获取数据，但只接受单个单词。请在代码编辑器中将`(readline)`更改为`(read)`。如果您在文本框中输入了"foo bar 123"，然后点击"OK"按钮，它只会回显"foo"。

```
(clear)

; replace (readline) with (read) and note the differences
(println "You entered: " (readline))

(run)
```

### 13\. 说出秘密词

**请注意：** 注意这个例子中有一行被注释掉了。如果您取消注释此行，弹出式提示将继续显示，直到您输入单词 `secret`。

此循环发生的原因是因为我们在 `user-did-not-say-secret` 规则的 LHS 中 `(retract` 捕获的 `(word ?f` 事实。这触发了在 `get-word-from-user` 中的模式 `(not (word ?))`，即工作内存中没有一个带有一个字段的 `(word` 事实的存在。

这个例子有点复杂，但它提供了一个展示如何"验证"用户输入的好例子。请注意，这不是*唯一*的方法。然而，它确实是展示如何使用规则和事实构建程序的"主循环"的好例子。

```
(clear)

(defrule get-word-from-user
  ;uncommenting this line will cause a prompt to popup
  ;until you enter "secret" in the textbox ;)
  ;(not (word ?))
  =>
  (print "Enter your word: ")
  (bind ?word (read))
  (println ?word)
  (assert (word ?word)))

(defrule user-said-secret
  (word secret)
  =>
  (println "You said the secret word! Great job"))

(defrule user-did-not-say-secret
  ?f 
  (retract ?f)
  (println "Try again..."))

(run)
```

### 14\. 这么多词语

`(readline)` 函数将捕获用户输入并返回一个字符串。如果用户的输入在单词之间有任何空格，我们可以使用 `(explode$` 将输入拆分成"多字段值"。这使我们能够在我们的规则中定义模式，以匹配用户输入的单个单词。

查看规则 `first-word-one`。当用户输入的第一个单词是 "**one**" 时，此规则将被激活。同样地，规则 `last-word-last` 在用户输入的最后一个单词是 "**last**" 时将被激活。

**任何地方** 规则将在提示中输入 "**任何地方**" 时触发。`$?` 将匹配任意数量的单词。例如，我们在 `counter` 规则中看到模式 `(exploded-words $?words)`。这意味着"匹配任意数量的值并将其引用为“多字段值” `?words`。就像 `?` 一样，`$?` 允许我们在 RHS 中声明"匹配任意数量的字段"，而不需要在其中引用它。

谈到`counter`规则：看一下`(length$ ?words)`。正如你可能已经猜到的那样，`(length$`将返回传递给它的多域值中的值数量。在这种情况下，它返回用户在提示中输入的单词数（由`?words`引用）。

```
(clear)

(defrule get-words-from-user
  =>
  (assert (words (readline))))

(defrule catcher
  (words ?words)
  =>
  (println "You said: " ?words)
  (assert (exploded-words (explode$ ?words))))

(defrule counter
  (exploded-words $?words)
  =>
  (println "That's " (length$ ?words) " words total."))

(defrule silence
  (exploded-words)
  =>
  (println "Ah, the strong silent type..."))

(defrule first-word-one
  (exploded-words one $?words)
  =>
  (println "Your first word was 'one'"))

(defrule last-word-last
  (exploded-words $?words last)
  =>
  (println "Your last word was 'last'"))

(defrule anywhere
  (exploded-words $? anywhere $?)
  =>
  (println "At some point, you typed 'anywhere'"))

(run)
```

### 15\. 显著突出

通过在我们的规则中使用`(declare (salience`，我们可以指定规则的“优先级”。这允许我们声明**某些规则必须在其他规则之前始终运行**。我们对规则的执行顺序进行了“硬编码”，而Rete算法则围绕它构建。

默认情况下，规则的显著性为`0`。具有大于`0`显著性的规则将在它们之前执行。类似地，具有较低显著性的规则将在它们之后执行。根据[基本编程指南](https://clipsrules.sourceforge.io/documentation/v640/bpg.pdf)，这个功能应该谨慎使用：

> 尽管可能值有很多，但在简单程序中，通过良好设计很少需要超过五个显著值，在复杂程序中十个显著值。

大多数情况下，规则引擎应该是无序的。这使得CLIPS能够“重活”确定规则运行的时机。这是微妙但相当强大的功能。使用规则时，你不会将值“传递”给运行并返回值的“函数”。相反，算法基于工作内存中“匹配”的事实“激活”规则。这种解耦了系统中存储的数据（事实）与业务逻辑（规则）。

这种解耦有助于用更少的引用来表达您正在建模的领域的域。软件越透明，您的系统就越接近您领域的“真理”。

```
(clear)

(defrule third
  (declare (salience 0))
  ?f 
  (println "Last!")
  (retract ?f))

(defrule never-runs
  (declare (salience -1))
  (my-fact)
  =>
  (println ":("))

(defrule first
  (declare (salience 2))
  (my-fact)
  =>
  (println "First!"))

(defrule second
  (declare (salience 1))
  (my-fact)
  =>
  (println "Second!"))

(assert (my-fact))

(run)
```

### 16\. 你的功能是什么？

函数允许您在程序中重复引用您编写的代码块。如果我们在写书，可能会发现把“说出来的话”放在引号中更容易理解，以及对说话者的引用。我们对喊叫也是这样处理。

我们还可以将难以记住的公式存储为函数。就个人而言，我很难记住华氏转摄氏的公式。通过`deffunction`的帮助，我可以轻松地存储并随时引用这个计算。

注意`(*`、`(-`和`(/`函数。这些是基本的数学运算：乘法、减法和除法。在CLIPS中，我们将数学问题`1 + 2`写成`(+ 1 2)`。这是因为CLIPS使用“类似LISP”的语法。

```
(clear)

(deffunction say (?words)
  (println "You say, \"" ?words ".\""))

(say "Hello, world")

(deffunction shout (?words)
  (println "\"" ?words "!\" you shout."))

(shout "Hello, world")'

(deffunction fahrenheit-to-celsius (?f)
  (* (- ?f 32) (/ 5 9) ))

(println "50°F is " (fahrenheit-to-celsius 50) "°C")
```

### 17\. 模板化我

模板用于定义事实的正式结构。通过在我们的例子中使用`(deftemplate person`，我们描述了`(person`事实的结构。我们在前几章中将事实写成值列表。我们将这些事实描述为具有有序“字段”的列表。`(person Sally "Hi")`将描述一个`(person`事实，其第二个字段中有值`Sally`，第三个字段中有值`"Hi"`。

使用模板，我们不再受限于以特定顺序存储人们的信息。相反，我们将值存储在命名为"Slots"的位置。例如，我们可以选择将我们的事实定义为`(person (greeting "Hi") (name Sally))`，首先是问候语。我们还可以为这些命名的插槽定义"默认"值，并在断言的事实中省略它们。

使用模板使我们在定义事实以及在规则左侧手边写匹配模式时具有一定的灵活性。看看规则`people-greet-each-other`。我们的第二个模式匹配“名字不是`?name1`的人”。在这个规则中，我们不需要包括对该事实的`(greeting` Slot的引用，因此我们可以省略它。

您还会注意到我们在模板定义中为`(greeting`定义了默认值。除了`default`之外，还有许多其他描述Slot的方法。请查看[Basic Programming Guide](https://clipsrules.sourceforge.io/documentation/v640/bpg.pdf)中名为“Deftemplate Construct”的部分，了解这些方法。

```
(clear)

(deftemplate person
  (slot name)
  (slot greeting (default Hello)))

(assert
  (person (name Greg))
  (person (name Sally) (greeting Hi))
  (person (name Sam) (greeting Howdy)))

(deffunction greet (?name1 ?name2 ?greeting)
  (println ?name1 " says \"" ?greeting "\" to " ?name2 "."))

(defrule people-greet-each-other
  (person (name ?name1) (greeting ?greeting1))
  (person (name ?name2&~?name1))
  =>
  (greet ?name1 ?name2 ?greeting1))

(run)
```

### 18\. 明确重置

让我们讨论一下`(clear)`和`(reset)`。我们熟悉的`(clear)`使我们的环境再次如新。没有规则，没有事实了。

`(reset)`做得少一点。它撤销系统中的所有事实和规则激活，但保留规则本身的定义。通过这种方式，您可以使用`(reset)`将引擎返回到已知的默认“状态”，然后再次运行它以获取新的输入。

如果我们需要一些事实被断言以返回到“默认”状态会发生什么？介绍`(deffacts`。当您使用`(reset)`时，将会断言您的`(deffacts`中定义的事实。考虑本章中的例子及其输出。在第一次运行时，我们比第二次多`(assert`了2个水果事实。因此，我们看到两个更多的人吃他们喜欢的水果。

```
(clear)

(deffacts fruits
  (fruit apple 1)
  (fruit banana 1)
  (fruit orange 1))

(deftemplate person
  (slot name)
  (slot favorite-fruit))

(deffacts people
  (person (name Tom) (favorite-fruit apple))
  (person (name Jenna) (favorite-fruit orange))
  (person (name Rian) (favorite-fruit apple)))

(defrule people-eat-favorite-fruits
  (person (name ?name) (favorite-fruit ?fruit))
  ?f 
  (println ?name " eats " ?fruit)
  (retract ?f))

(defrule persons-favorite-fruit-missing
  (person (name ?name) (favorite-fruit ?fruit))
  (not (fruit ?fruit ?))
  =>
  (format t "There's no more of %s's favorite fruit %s!%n" ?name ?fruit))

(reset)

(assert (fruit apple 2) (fruit orange 2))

(println "Run 1:")
(println "------")
(run)

(println " ")
(reset)

(println "Run 2:")
(println "------")
(run)
```

### 19\. 谁观察统计数据？

```
(clear)
(watch statistics)

(defrule fibonacci
  ?fact  ?i 1))
  =>
  (print ?fib " ")
  (retract ?fact)
  (assert (fib (+ ?fib ?lastfib) ?fib (- ?i 1))))

(defrule last-fibonacci-number
  ?fact 
  (println ?fib)
  (retract ?fact))

(assert (fib 1 0 10))

(run)
(unwatch statistics)
```

### 20\. 规则：激活！

```
(clear)
(watch activations)

(defrule b-exists-and-a-less-than-1
   (b ?b)
   (a ?a&:(< ?a 1))
   =>
   (println "there is b = " ?b ", AND a " ?a " is less than 1"))

(defrule a-less-than-1
   (a ?a&:(< ?a 1))
   =>
   (println "a " ?a " is less than 1"))

(println "  Step 1  ")
(println "==========")
(println "--- assert")
(assert (a 2))
(println "------ run")
(run)
(println " ")

(println "  Step 2  ")
(println "==========")
(println "--- assert")
(assert (a 0))
(println "------ run")
(run)
(println " ")

(println "  Step 3  ")
(println "==========")
(println "--- assert")
(assert (b foo) (a -1) (a 3))
(println "------ run")
(run)
(unwatch activations)
```

### 21\. 观察函数

```
(clear)
(watch deffunctions)

(deffunction less-than-1 (?a)
   (println "calculating whether " ?a " is less than 1...")
   (< ?a 1))

(defrule b-exists-and-a-less-than-1
   (b ?b)
   (a ?a&:(less-than-1 ?a))
   =>
   (println "there is b = " ?b ", AND a " ?a " is less than 1"))

(defrule a-less-than-1
   (a ?a&:(less-than-1 ?a))
   =>
   (println "a " ?a " is less than 1"))

(println "  Step 1  ")
(println "==========")
(println "--- assert")
(assert (a 2))
(println "------ run")
(run)
(println " ")

(println "  Step 2  ")
(println "==========")
(println "--- assert")
(assert (a 0))
(println "------ run")
(run)
(println " ")

(println "  Step 3  ")
(println "==========")
(println "--- assert")
(assert (b foo) (a -1) (a 3))
(println "------ run")
(run)
(unwatch deffunctions)
```

### 22\. ???

就这些了！感谢您到目前为止坚持使用本教程。我会随着时间的推移继续添加内容，所以请确保收藏此页面并定期查看 :).

这整个页面在可能时都是从头编写的。没有框架，只有经典的过程式Javascript、CSS和HTML。多年前学习Go时，我发现[Go之旅](https://go.dev/tour/welcome/1)非常有帮助，因为不需要安装任何东西。我使用[emscripten](https://emscripten.org/)将[CLIPS](https://www.clipsrules.net/)编译成WASM二进制文件。我从头开始编写了代码编辑器和语法高亮器，并从[Stackoverflow](https://stackoverflow.com/)中获取了代码片段。我承认我对此有些自豪 :).

我在本教程中的目标是传播有关CLIPS的信息。我希望看到它在更多日常应用中的使用，因为它是一种用更少代码表达复杂业务逻辑的非常强大的方式。

```
(clear)
(run)
```
