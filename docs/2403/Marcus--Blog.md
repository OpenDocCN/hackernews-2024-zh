<!--yml

类别：未分类

日期：2024-05-27 14:59:02

-->

# Marcus的博客

> 来源：[https://mbuffett.com/posts/compressing-chess-moves/](https://mbuffett.com/posts/compressing-chess-moves/)

更新：我写了一篇[新文章](/posts/compressing-chess-moves-even-further/)，介绍了一种比这种方法效率高大约2.5倍的新技术

国际象棋标记法自[描述性标记法](https://en.wikipedia.org/wiki/Descriptive_notation)以来发展了很多，现在我们有了非常好的和可解释的标准代数记号，比如`Qxf7`（后在f7吃子）或`Nf3`（马走到f3）。

如果您尝试存储大量这些文本格式的内容，这是一种绝佳的文本格式，但非常浪费空间。`Qxf7`占用4字节，即32位。让我们简单计算一下实际传输的信息量。这一步影响了6种棋子之一（3位），这一步也是吃子（1位），并且指定了一个目的方格（64种可能性 == 6位）。把它们加起来，你得到10位。远远少于文本表示所需的32位。

我为什么在乎呢？我运营一个存储大量棋局走法的网站，总计大约有1亿个走法。假设每个走法平均6步，那就是6亿步。数据库足够大，查询时受到IO限制。在提取数千个走法时，我希望加快从数据库读取的速度。

## 第一次尝试

首先是一些常规数据：

+   编码一个文件或列（a-h或1-8）需要3位（8种可能性）

+   编码一个棋子（k、q、r、b、n、p）需要3位（6种可能性）

+   编码一个方格需要6位（64种可能性）

所以让我们看看我们用于编码SAN的棋子。

让我们以最简单的方法开始。

+   是哪一种棋子？3位

+   是否是一个吃子动作？1位

+   我们是否需要消除歧义（即`Ngf3`）？最多2位 + 6位（这将在后面进一步解释）

+   它去了哪里？6位

+   是否是一个晋升，并且是哪一种？7 位

+   是否将其当作将军？1位

+   是否将其当作将军？1位

+   是否是易位？短易位还是长易位？2位

这给我们一个总共`3+1+2+6+6+7+1+1+2 = 29`位，或者大约每步3.5字节。虽然不算太好。实际上，很多走法在文本格式中所占的位数要少于这个。

## 变得更聪明

### 最初的几位

在第一轮中，我们只是用3位编码了移动的棋子，但是由于国际象棋只有6种棋子，这样做留下了2个未使用的排列。幸运的是，有两个非常常见的走法恰好填补了这个空缺，分别是短易位（`O-O`）和长易位（`O-O-O`）。

所以我们可以这样编码前3位：

```
match first_bytes {  FirstBytes::Pawn => bits.extend(&[false, false, false]), FirstBytes::Knight => bits.extend(&[false, false, true]), FirstBytes::Bishop => bits.extend(&[false, true, false]), FirstBytes::Rook => bits.extend(&[false, true, true]), FirstBytes::Queen => bits.extend(&[true, false, false]), FirstBytes::King => bits.extend(&[true, false, true]), FirstBytes::ShortCastling => bits.extend(&[true, true, false]), FirstBytes::LongCastling => bits.extend(&[true, true, true]), } 
```

### 目的地方格

在除了易位之外的所有情况下，为了重现标准代数记号（SAN），您需要知道棋子移动到的方格。因此，当易位时我们会完全跳过这一点，但在其他情况下，始终包括6位用于目的地方格。

#### 只是开玩笑，兵的走法！

目的地方格规则还有一个可能不那么明显的例外。我们来看一下 `exf6` 这步棋。我们知道它是从 e 列走的兵步，因此我们不需要用 6 位来编码它所捕捉的文件。毕竟，你永远不会看到 `exa6`。所以在这些情况下，我们只需要 4 位（一个用于方向，一个用于行）而不是 6 位来表示目的地方格。

但我们可以在这里变得更聪明。以 `hxg6` 为例。你一看到 `hx`，就知道文件将会是 g 文件。因此，我们甚至不需要额外的比特来编码方向，我们只需编码文件一次，行一次。

因此，这里是设置：

+   从 b-g 文件的兵捕捉：3 位用于你要捕捉的文件，1 位用于捕捉方向，3 位用于行

    +   总计：10 比特用于移动

+   从 a 和 h 文件的兵捕捉：3 位用于你要捕捉的文件，3 位用于行

    +   总计：6 比特用于移动

### “特殊”走法

对每一步棋进行编码有点浪费，无论是晋升、将军、将死、捕捉等。毕竟，在一场比赛中，绝大多数的棋步都不是这些。

因此，我们将一个比特用于确定一个走法是否是“特殊”的。这意味着我们必须为促进/将军/捕捉的走法使用额外的比特，但这也意味着我们可以为“常规”走法节省大量的比特。

晋升是很好的，因为虽然有 6 种象棋棋子，但只有 4 种有效的棋子可以晋升（毕竟，你不能晋升为国王或兵）。因此，我们只需要 2 比特来编码晋升的棋子。

```
if let Some(promotion) = promotion {  bits.push(true); bits.extend(match promotion { PromotionPiece::Queen => &[false, false], PromotionPiece::Rook => &[false, true], PromotionPiece::Bishop => &[true, false], PromotionPiece::Knight => &[true, true], }); } else {  bits.push(false); } 
```

### 消除歧义

消除歧义有点棘手。你需要编码相当多的信息，才能准确解码接收到的标准代数记法。

以 `Ngf3` 为例。除了通常的内容，你还需要编码消除歧义（文件=g）。有 3 种不同的消除歧义可能性（行、文件或整个方格），因此我们需要 2 位。有一个位会浪费，但是消除歧义的走法并不常见，我想不出一个好办法来利用最后一个排列。

所以我们总是使用 2 位，然后使用 3 位来表示行，3 位来表示列，或者 6 位来表示整个方格（非常罕见）。

```
if let Some(disambiguate) = disambiguate {  bits.push(true); match disambiguate { Disambiguation::File(file) => { bits.extend(&[false, true]); bits.extend(square_component_to_bits(file)); } Disambiguation::Rank(rank) => { bits.extend(&[true, false]); bits.extend(square_component_to_bits(rank)); } Disambiguation::Square(square) => { bits.extend(&[true, true]); bits.extend(square_component_to_bits(square.0)); bits.extend(square_component_to_bits(square.1)); } } } else {  bits.push(false); } 
```

## 这样做有何效果？

| Move | 原始比特 | 编码后比特 | 节省 |
| --- | --- | --- | --- |
| e4 | 16 | 10 | 37.5% |
| exd5 | 32 | 12 | 62.5% |
| Nf3+ | 24 | 10 | 58.33% |
| Qxa5+ | 40 | 16 | 60% |
| cxd8=Q# | 56 | 16 | 71.43% |

不错！我们节省了从 37.5% 到 71.43% 的比特数。

## PGN

你可能会想到这样的问题：“以比特来衡量这些是不是有点作弊？因为你每次只能处理一个字节，需要 10 比特来表示一步棋几乎与 16 比特相同”

好吧，是的，这是真的，这意味着当存储单个标准代数记法时，我们的节省就少了。但它们并不能占据我存储的大部分内容，那些是 PGN。

PGN 是一种存储棋局或棋局线路的方式，看起来是这样的：

```
1.e4 d5 2.exd5 Nf6 3.d4 Bg4 4.Be2 Bxe2 5.Nxe2 Qxd5 6.O-O Nc6 7.c3 O-O-O 8.Qb3 Qh5 9.Nf4 Qh4 10.Qxf7 Ng4 11.h3 Nf6 12.Be3 e6 13.Qxe6+ Kb8 14.Nd2 Bd6 15.g3 Qh6 16.Qf5 Ne7 17.Qa5 b6 18.Qa6 Bxf4 19.Bxf4 Qxh3 20.Nf3 Ned5 21.Be5 Qh5 22.Kg2 Qg6 23.Rae1 Nh5 24.Nh4 Nhf4+ 25.Bxf4 Nxf4+ 26.Kh2 b5 27.Qxg6 hxg6 28.gxf4 Rxh4+ 29.Kg3 Rdh8 30.Re5 Rh3+ 31.Kg4 R8h4+ 32.Kg5 Rh5+ 33.Kxg6 Rh6+ 34.Kf7 Rh7 35.Rxb5+ Kc8 36.Rg5 g6+ 37.Kxg6 R3h6+ 38.Kf5 Rf7+ 39.Kg4 Rhf6 40.f5 Kd7 41.Re1 Kd6 42.Re8 Kd7 43.Ra8 Rh6 44.Rxa7 Rfh7 45.Kf4 Rh4+ 46.Rg4 Rh2 47.f3 Rxb2 48.f6 Rf7 49.Rg7 Ke6 50.Rxf7 Kxf7 51.Rxc7+ Kxf6 52.a4 Ra2 53.Rc4 Ra3 54.d5 Ke7 55.Ke5 Kd7 56.f4 Ra2 57.Kd4 Rd2+ 58.Kc5 Ra2 59.Kb5 Kd6 60.Rd4 Rb2+ 61.Ka6 Ra2 62.a5 Kc5 63.d6 Ra3 64.d7 Kc6 65.d8=Q Rxa5+ 66.Kxa5 Kc5 67.Qc7# 
```

这个 PGN 是 759 字节。然而有大量的浪费空间。每个移动之间有一个字节的空格。然后 `2.` 之间至少有 3 个字节。这有点疯狂，如果我们结合我们的 SAN 编码，我们可以将其压缩到更小。

如果我们使用我们的 SAN 编码对整个 PGN 进行编码，因为我们知道一旦达到移动的结尾，我们就可以将这个特定的 759 字节 PGN 压缩到 195 字节，节省 **74%**。

## 影响

这个还没有部署，我正在处理大量其他性能改进。但是我预计这个以及 EPD 压缩（我可能会另写一篇文章介绍）将使数据库大小减少约 70%。我们几乎完全受到读取限制，这应该意味着我们运行的最昂贵的查询将加速 3 倍。

## 速度

另一个考虑因素是进行此编码/解码的速度；这是否会抵消拥有更小的数据库带来的收益？事实证明，计算机在这方面非常快，从这种编码的转换几乎可以忽略不计。

我正在使用 Rust 和 `bitvec` 库。编码然后解码 1000 个移动大约需要 600,000 纳秒，或者 0.6 毫秒。我还没有完全进行性能优化，而且我知道有几个地方非常低效。我猜测我甚至没有达到最佳状态的 10 倍，但应该足够好。
