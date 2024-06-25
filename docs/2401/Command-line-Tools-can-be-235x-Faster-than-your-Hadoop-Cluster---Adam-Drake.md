<!--yml

类别：未分类

日期：2024-05-27 15:12:48

-->

# 命令行工具可能比你的 Hadoop 集群快 235 倍 - Adam Drake

> 来源：[https://adamdrake.com/command-line-tools-can-be-235x-faster-than-your-hadoop-cluster.html](https://adamdrake.com/command-line-tools-can-be-235x-faster-than-your-hadoop-cluster.html)

# 命令行工具可能比你的 Hadoop 集群快 235 倍

当我浏览网页并定期访问一些站点时，我发现了 [Tom Hayden](https://tomhayden3.com/2013/12/27/chess-mr-job/) 的一篇有趣的文章，他讲述了如何使用 [Amazon Elastic Map Reduce](https://aws.amazon.com/elasticmapreduce/) (EMR) 和 [mrjob](https://github.com/Yelp/mrjob) 计算他从 [millionbase archive](https://www.top-5000.nl/pgn.htm) 下载的国际象棋比赛数据的胜负比率，并在 EMR 上玩得开心。由于数据量只有约 1.75GB，包含约 200 万场国际象棋比赛，我对使用 Hadoop 进行任务处理持怀疑态度，但我能理解他学习和在 mrjob 和 EMR 上玩耍的目标。由于问题基本上只是查看每个文件的结果行并汇总不同的结果，因此使用 shell 命令进行流处理似乎是理想的选择。我尝试了这个方法，对于相同数量的数据，我能够使用我的笔记本电脑在约 12 秒内获得结果（处理速度约为 270MB/秒），而 Hadoop 处理大约需要 26 分钟（处理速度约为 1.14MB/秒）。

在报告了在集群中使用 7 台 c1.medium 机器处理数据所需的时间为 26 分钟后，Tom 补充道

> 这可能比在我的机器上串行运行要好，但可能不如如果我在本地使用某种聪明的多线程应用程序好。

这绝对是正确的，尽管甚至串行处理可能会比 26 分钟要快。尽管 Tom 是出于娱乐而进行这个项目，但人们经常使用 Hadoop 和其他所谓的 *Big Data (tm)* 工具进行可以使用更简单的工具和不同技术更快地完成的真实世界处理和分析工作。

数据处理的一个特别少用的方法是使用标准的 shell 工具和命令。这种方法的好处是巨大的，因为使用 shell 命令创建数据管道意味着所有处理步骤都可以并行进行。这基本上就像在本地机器上拥有自己的 [Storm](https://storm-project.net/) 集群。甚至 Spouts、Bolts 和 Sinks 的概念也可以转移到 shell 管道和它们之间的命令中。你可以相当容易地用基本命令构建一个流处理管道，它将与许多现代 *Big Data (tm)* 工具相比具有极好的性能。

另一个要点是批处理与流处理分析方法的比较。Tom 在文章开头提到，加载了 10000 场比赛并在本地进行分析后，他的内存有点不够用了。这是因为所有的游戏数据都被加载到 RAM 中进行分析。然而，仔细考虑了一下这个问题，它可以很容易地通过流式分析来解决，这几乎不需要任何内存。我们将创建的结果流处理流水线将比 Hadoop 实现快 235 倍，并且几乎不使用任何内存。

首先，在流水线中的第一步是从 PGN 文件中获取数据。由于我不知道这是什么格式，我在[Wikipedia](https://en.wikipedia.org/wiki/Portable_Game_Notation)上查看了一下。

```
[Event "F/S Return Match"]
[Site "Belgrade, Serbia Yugoslavia|JUG"]
[Date "1992.11.04"]
[Round "29"]
[White "Fischer, Robert J."]
[Black "Spassky, Boris V."]
[Result "1/2-1/2"]
(moves from the game follow...) 
```

我们只对游戏的结果感兴趣，这只有 3 种真实结果。1-0 表示白棋赢，0-1 表示黑棋赢，而 1/2-1/2 表示游戏是平局。还有一个 *-* 表示游戏正在进行中或无法计分，但我们对此不感兴趣。

首先要做的是获取大量的游戏数据。这比我想象的要困难得多，但在网上找了一些之后，我发现了来自[rozim](https://github.com/rozim/ChessData)的 GitHub 上的一个 git 仓库，里面有大量的游戏数据。我用这个来编译了一组大小为 3.46GB 的数据，大约是 Tom 在他的测试中所用数据的两倍。下一步是将所有这些数据放入我们的流水线中。

*如果你在跟踪并计时你的处理过程，请不要忘记清除操作系统的页面缓存，否则你将无法获得有效的处理时间。*

Shell 命令非常适合数据处理流水线，因为你可以免费获得并行性。要证明，请在你的终端中尝试一个简单的例子。

```
sleep 3 | echo "Hello world." 
```

直观地看，上述代码会休眠 3 秒，然后打印 `Hello world`，但实际上两个步骤是同时进行的。这个基本事实是为什么对于简单的非 IO 限制的处理系统能够在单个机器上运行时提供如此大的加速。

在开始分析流水线之前，最好先了解一下其处理速度可能有多快，为此我们可以将数据简单地转储到 `/dev/null`。

在这种情况下，处理 3.46GB 大约需要 13 秒，大约是 272MB/sec。由于 IO 约束，这将是系统上处理数据的速度的一种上限。

现在我们可以开始分析流水线了，其第一步是使用 `cat` 生成数据流。

由于文件中只有结果行是有趣的，我们可以简单地扫描所有数据文件，并使用 `grep` 选取包含 ‘Results’ 的行。

```
cat *.pgn | grep "Result" 
```

这将只给我们带来文件中的 `Result` 行。现在如果我们想的话，我们可以简单地使用 `sort` 和 `uniq` 命令来获取文件中所有唯一项目的列表以及它们的计数。

```
cat *.pgn | grep "Result" | sort | uniq -c 
```

这是一个非常简单的分析管道，并在大约70秒内给我们结果。虽然我们肯定可以做得更好，但是假设线性扩展，这将需要Hadoop集群大约52分钟来处理。

为了进一步提高速度，我们可以从管道中去除`sort | uniq`步骤，并用AWK替换它们，这是一个非常适用于事件驱动数据处理的精彩工具/语言。

```
cat *.pgn | grep "Result" | awk '{ split($0, a, "-"); res = substr(a[1], length(a[1]), 1); if (res == 1) white++; if (res == 0) black++; if (res == 2) draw++;} END { print white+black+draw, white, black, draw }' 
```

这将获取每个结果记录，将其拆分为连字符，并取左侧的字符，这将在黑色赢得的情况下为0，在白色赢得的情况下为1，在平局的情况下为2。注意，`$0`是一个代表整个记录的内置变量。

这将运行时间减少到大约65秒，因为我们处理的数据量增加了一倍，所以这是大约47倍的加速。

因此，即使在这一点上，我们已经通过一个简单的本地解决方案实现了大约47倍的加速。此外，由于唯一存储的数据是实际计数，而且在内存空间方面增加3个整数几乎是免费的，因此内存使用量实际上为零。然而，查看`htop`运行时，可以看到`grep`目前是瓶颈，完全使用了一个CPU核心。

这个未使用的核心问题可以通过精彩的`xargs`命令解决，它将允许我们并行化`grep`。由于`xargs`期望以某种方式输入，因此最好使用`find`与`-print0`参数，以确保传递给`xargs`的每个文件名都是空终止的。相应的`-0`告诉`xargs`期望空终止输入。此外，`-n`表示每个进程给多少个输入，`-P`表示要并行运行的进程数。还需要注意的是，这样的并行管道不能保证交付顺序，但如果您习惯于处理分布式处理系统，则这不是问题。`grep`的`-F`表示我们仅匹配固定字符串而不执行任何花哨的正则表达式，并且可能会提供小的加速，我在测试中没有注意到。

```
find . -type f -name '*.pgn' -print0 | xargs -0 -n1 -P4 grep -F "Result" | gawk '{ split($0, a, "-"); res = substr(a[1], length(a[1]), 1); if (res == 1) white++; if (res == 0) black++; if (res == 2) draw++;} END { print NR, white, black, draw }' 
```

这导致运行时间约为38秒，这比在我们的管道中并行化`grep`步骤后的处理时间额外减少了大约40%左右。这使我们的速度大约比Hadoop实现快77倍。

尽管我们通过在我们的管道中并行化`grep`步骤大大改善了性能，但实际上我们可以通过让`awk`过滤输入记录（在这种情况下是行），并且仅对包含字符串“Result”的记录进行操作，从而完全删除它。

```
find . -type f -name '*.pgn' -print0 | xargs -0 -n1 -P4 awk '/Result/ { split($0, a, "-"); res = substr(a[1], length(a[1]), 1); if (res == 1) white++; if (res == 0) black++; if (res == 2) draw++;} END { print white+black+draw, white, black, draw }' 
```

你可能认为这将是正确的解决方案，但这将输出**每个**文件的结果，而我们想要将它们全部聚合在一起。结果的正确实现在概念上与MapReduce实现非常相似。

```
find . -type f -name '*.pgn' -print0 | xargs -0 -n4 -P4 awk '/Result/ { split($0, a, "-"); res = substr(a[1], length(a[1]), 1); if (res == 1) white++; if (res == 0) black++; if (res == 2) draw++ } END { print white+black+draw, white, black, draw }' | awk '{games += $1; white += $2; black += $3; draw += $4; } END { print games, white, black, draw }' 
```

通过在最后添加第二个awk步骤，我们就可以得到所需的汇总游戏信息。

进一步显著提高了速度，达到了约18秒的运行时间，约为Hadoop实现的174倍。

不过，我们可以通过使用[mawk](https://invisible-island.net/mawk/mawk.html)使其运行速度更快一些，它经常可以作为`gawk`的替代品，并提供更好的性能。

```
find . -type f -name '*.pgn' -print0 | xargs -0 -n4 -P4 mawk '/Result/ { split($0, a, "-"); res = substr(a[1], length(a[1]), 1); if (res == 1) white++; if (res == 0) black++; if (res == 2) draw++ } END { print white+black+draw, white, black, draw }' | mawk '{games += $1; white += $2; black += $3; draw += $4; } END { print games, white, black, draw }' 
```

这个`find | xargs mawk | mawk`管道使我们的运行时间缩短到约12秒，约为270MB/sec，比Hadoop实现快约235倍。

希望这些例子能够说明一些关于在数据处理任务中使用和滥用Hadoop等工具的观点，这些任务在单台计算机上使用简单的shell命令和工具就可以更好地完成。如果你有大量数据或确实需要分布式处理，那么可能需要使用Hadoop等工具，但现在我经常看到Hadoop被用在传统的关系型数据库或其他解决方案在性能、实施成本和持续维护方面更好的地方。

<data class="p-bridgy-omit-link" value="false"></data>
