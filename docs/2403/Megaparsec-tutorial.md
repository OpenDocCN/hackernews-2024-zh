<!--yml

类别：未分类

日期：2024-05-27 14:34:24

-->

# Megaparsec 教程

> 来源：[https://markkarpov.com/tutorial/megaparsec.html](https://markkarpov.com/tutorial/megaparsec.html)

# Megaparsec 教程

*发布于2019年2月23日，最后更新于2021年10月30日*

*这是最初作为《Intermediate Haskell》书中的一章写成的Megaparsec教程。由于过去一年该书没有进展，其他作者同意我将这部分文本发布为独立教程，以便人们至少可以从我们的这部分工作中受益。*

[日语翻译](https://haskell.e-bigmoon.com/posts/2019/07-14-megaparsec-tutorial.html)，[中文翻译](https://blog.yzyzsun.me/megaparsec/)。

在章节“一个例子：编写自己的解析器组合子”中开发的玩具解析器组合子不适合实际使用，所以让我们继续看看Haskell生态系统中解决同一问题的库，并注意它们的各种权衡：

+   [`parsec`](https://hackage.haskell.org/package/parsec) 长期以来一直是Haskell中的“默认”解析库。据说该库专注于错误消息的质量。然而，它没有良好的测试覆盖率，目前处于维护模式。

+   [`attoparsec`](https://hackage.haskell.org/package/attoparsec) 是一个强大且快速的解析库，专注于性能。它是这个列表中唯一支持增量解析的库。其缺点是错误消息质量较差，无法用作单子变换器，并且输入流类型受限。

+   [`trifecta`](https://hackage.haskell.org/package/trifecta) 具有良好的错误消息特性，但文档不足且难以理解。它可以直接解析`String`和`ByteString`，但不能解析`Text`。

+   [`megaparsec`](https://hackage.haskell.org/package/megaparsec) 是`parsec`的一个分支，最近几年一直在积极开发中。当前版本尝试在速度、灵活性和解析错误质量之间找到一个良好的平衡点。作为`parsec`的非官方继承者，它保持传统且对于那些曾经使用过该库或阅读过`parsec`教程的用户来说是立即熟悉的。

尝试覆盖所有这些库是不切实际的，因此我们将专注于`megaparsec`。更准确地说，我们将覆盖版本9，在本书出版时，这个版本可能已经几乎在所有地方取代了旧版本。

## `ParsecT` 和 `Parsec` 单子

`ParsecT` 是主要的解析器单子变换器，也是`megaparsec`中的核心数据类型。`ParsecT e s m a` 的参数化如下：

+   `e` 是自定义错误消息组件的类型。如果我们暂时不需要任何自定义（现在确实不需要），我们只需使用`Data.Void`模块中的`Void`。

+   `s` 是输入流的类型。`megaparsec` 可以直接使用 `String`、严格和惰性的 `Text`，以及严格和惰性的 `ByteString`。也可以使用自定义输入流。

+   `m` 是 `ParsecT` 单子变换器的内部单子。

+   `a` 是解析结果的单子值。

由于大多数情况下 `m` 只是 `Identity`，`Parsec` 类型同义词非常有用：

```
type  Parsec  e  s  a  =  ParsecT  e  s  Identity  a 
```

`Parsec` 简单地是 `ParsecT` 的非变换器版本。

我们还可以在 `megaparsec` 和 MTL 单子变换器及类之间进行类比。事实上，还有 `MonadParsec` 类型类，其目的类似于 `MonadState` 和 `MonadReader` 等类型类。我们将稍后回到 `MonadParsec`，在[后文](#the-monadparsec-type-class)中更详细地讨论它。

谈到类型同义词，使用 `megaparsec` 开始编写解析器的最佳方法是为您的解析器定义一个自定义类型同义词。有两个原因支持这个想法：

+   添加顶级签名（如 `Parser Int`）将更容易。在没有签名的情况下，像 `e` 这样的事物通常是模棱两可的——这是库的多态 API 的反面。

+   使用所有类型变量固定的具体类型可以帮助 GHC 进行更好的优化。如果您的解析器保持多态，GHC 将无法进行太多优化。虽然 `megaparsec` 的 API 是多态的，但预期最终用户将坚持使用具体类型的解析单子，因此内联和大多数函数将其定义倒入所谓的*接口文件*，使 GHC 能够生成非常有效的非多态代码。

让我们定义一个类型同义词（通常称为 `Parser`），如下所示：

```
type  Parser  =  Parsec  Void  Text  --                   ^    ^  --                   |    |  -- Custom error component Type of input stream 
```

在我们开始处理自定义解析错误之前，当您在本章中看到 `Parser` 时，请假定这个类型。

## 字符和二进制流

有人说 `megaparsec` 可以与五种类型的输入流直接配合使用：`String`、严格和惰性的 `Text`，以及严格和惰性的 `ByteString`。这是可能的，因为该库将这些类型作为 `Stream` 类型类的实例，该类型类抽象了每个数据类型应支持以用作 `megaparsec` 解析器输入的功能。

`Stream` 的简化版本可能如下所示：

```
class  Stream  s  where  type  Token  s  ::  *  type  Tokens  s  ::  *  take1_  ::  s  ->  Maybe  (Token  s,  s)  -- aka uncons  tokensToChunk  ::  Proxy  s  ->  [Token  s]  ->  Tokens  s 
```

`Stream` 的真正定义有更多方法，但了解它们对使用该库并不是必要的。

注意该类型类有两个与之关联的类型函数：

+   对于流 `s` 的 `Token s` 是单个令牌的类型。常见的例子有 `Char` 和 `Word8`，尽管对于自定义流可能会有其他类型。

+   `Tokens s` 是流 `s` 的“块”的类型。 *Chunk* 的概念仅出于性能原因而引入。实际上，往往可以对流的部分进行更有效的表示，该表示与令牌列表 `[Token s]` 是同构的。例如，类型为 `Text` 的输入流具有 `Tokens s ~ Text`：`Text` 的块就是 `Text`。尽管类型等式 `Tokens s ~ s` 经常成立，但对于自定义流，`Tokens s` 和 `s` 可能不同，因此我们在 `megaparsec` 中分开这些类型。

我们可以将所有默认输入流放入一个表格中，如下所示：

| `s` | `Token s` | `Tokens s` |
| --- | --- | --- |
| `String` | `Char` | `String` |
| 严格 `Text` | `Char` | 严格 `Text` |
| 惰性 `Text` | `Char` | 惰性 `Text` |
| 严格 `ByteString` | `Word8` | 严格 `ByteString` |
| 惰性 `ByteString` | `Text.Megaparsec.Char` | `Text.Megaparsec.Byte` |

重要的是要习惯 `Token` 和 `Tokens` 类型函数，因为它们在 `megaparsec` API 的类型中随处可见。

如果我们按令牌类型对所有默认输入流进行分组，您可能已经注意到，我们将得到两组：

+   字符流，其中 `Token s ~ Char`：`String` 和严格/惰性 `Text`，

+   二进制流，其中 `Token s ~ Word8`：严格和惰性的 `ByteString`。

结果表明，使用 `megaparsec` 不需要为每种类型的输入流编写相同的解析器（例如，在 `attoparsec` 库中是这种情况），但我们仍然必须为不同的令牌类型编写不同的代码：

+   要获得字符流的通用组合器，请导入 `Text.Megaparsec.Char` 模块；

+   要获得相同的二进制流，请导入 `Text.Megaparsec.Byte`。

这些模块包含两组类似的辅助解析器，例如：

| 名称 | `Word8` | 惰性 `ByteString` |
| --- | --- | --- |
| `newline` | `(MonadParsec e s m, Token s ~ Char) => m (Token s)` | `(MonadParsec e s m, Token s ~ Word8) => m (Token s)` |
| `eol` | `(MonadParsec e s m, Token s ~ Char) => m (Tokens s)` | `(MonadParsec e s m, Token s ~ Word8) => m (Tokens s)` |

让我们介绍一些构建模块的原语，以便了解我们即将使用的工具。

第一个原语称为 `token`，因此它允许我们解析 `Token s`：

```
token  ::  MonadParsec  e  s  m  =>  (Token  s  ->  Maybe  a)  -- ^ Matching function for the token to parse  ->  Set  (ErrorItem  (Token  s))  -- ^ Expected items (in case of an error)  ->  m  a 
```

`token` 的第一个参数是要解析的令牌的匹配函数。如果函数返回 `Just` 中的值，则该值成为解析的结果。`Nothing` 表示解析器不接受此令牌，因此原语失败。

第二个参数是一个 `Set`（来自 `containers` 包），其中包含所有预期的 `ErrorItem`，以便在失败时向用户显示。我们将在讨论解析错误时详细探讨 `ErrorItem` 类型。

为了更好地理解 `token` 的工作原理，让我们看看 `Text.Megaparsec` 模块中的一些定义，该模块包含了与所有输入流类型一起使用的一些组合子。`satisfy` 是一个相当常见的组合子，我们给它一个返回我们想要匹配的标记的谓词，并且它会给我们一个解析器：

```
satisfy  ::  MonadParsec  e  s  m  =>  (Token  s  ->  Bool)  -- ^ Predicate to apply  ->  m  (Token  s)  satisfy  f  =  token  testToken  Set.empty  where  testToken  x  =  if  f  x  then  Just  x  else  Nothing 
```

`testToken` 的工作是将返回 `Bool` 的函数 `f` 转换为返回 `Maybe (Token s)` 的函数，这是 `token` 所期望的。对于 `satisfy`，我们不知道我们期望匹配的确切标记序列，因此我们将 `Set.empty` 作为第二个参数传递。

`satisfy` 应该看起来是可以理解的，让我们看看它是如何工作的。为了使用解析器，我们需要一个运行它的辅助函数。对于在 GHCi 中进行测试，`megaparsec` 提供了 `parseTest`。

首先，让我们启动 GHCi 并导入一些模块：

```
λ> import Text.Megaparsec
λ> import Text.Megaparsec.Char
λ> import Data.Text (Text)
λ> import Data.Void 
```

我们添加了 `Parser` 类型同义词，用于在解析器类型中消除歧义：

```
λ> type Parser = Parsec Void Text 
```

我们还需要启用 `OverloadedStrings` 语言扩展，以便我们可以将字符串字面量用作 `Text` 值：

```
λ> :set -XOverloadedStrings

λ> parseTest (satisfy (== 'a') :: Parser Char) ""
1:1:
  |
1 | <empty line>
  | ^
unexpected end of input

λ> parseTest (satisfy (== 'a') :: Parser Char) "a"
'a'

λ> parseTest (satisfy (== 'a') :: Parser Char) "b"
1:1:
  |
1 | b
  | ^
unexpected 'b'

λ> parseTest (satisfy (> 'c') :: Parser Char) "a"
1:1:
  |
1 | a
  | ^
unexpected 'a'

λ> parseTest (satisfy (> 'c') :: Parser Char) "d"
'd' 
```

*`:: Parser Char` 注解是必要的，因为 `satisfy` 本身是多态的，所以 `parseTest` 无法知道在 `MonadParsec e s m` 中使用什么来替换 `e` 和 `s`（假定 `m` 为 `Identity` 与这些帮助程序）。如果我们使用一个已存在的带有类型签名的解析器来工作，明确指定解析器类型的解释就是不必要的。*

看起来一切都挺好的。`satisfy` 的问题在于当它失败时没有说明期望的内容，因为我们不能分析 `satisfy` 的调用者提供的函数。有其他一些组合子不那么通用，但它们可以生成更有帮助的错误消息。例如 `single`（在 `Text.Megaparsec.Byte` 和 `Text.Megaparsec.Char` 中称为类型约束同义词 `char`），它匹配特定的标记值：

```
single  ::  MonadParsec  e  s  m  =>  Token  s  -- ^ Token to match  ->  m  (Token  s)  single  t  =  token  testToken  expected  where  testToken  x  =  if  x  ==  t  then  Just  x  else  Nothing  expected  =  Set.singleton  (Tokens  (t:|[])) 
```

`Tokens` 数据类型构造函数与我们之前讨论过的类型函数 `Tokens` 没有任何共同点。实际上，`Tokens` 是 `ErrorItem` 的构造函数之一，用于指定我们期望匹配的具体标记序列。

```
λ> parseTest (char 'a' :: Parser Char) "b"
1:1:
  |
1 | b
  | ^
unexpected 'b'
expecting 'a'

λ> parseTest (char 'a' :: Parser Char) "a"
'a' 
```

现在我们可以根据上表定义 `newline`：

```
newline  ::  (MonadParsec  e  s  m,  Token  s  ~  Char)  =>  m  (Token  s)  newline  =  single  '\n' 
```

第二个基本函数称为 `tokens`，它允许我们解析 `Tokens s`，也就是说，它可以用于匹配输入的固定块：

```
tokens  ::  MonadParsec  e  s  m  =>  (Tokens  s  ->  Tokens  s  ->  Bool)  -- ^ Predicate to check equality of chunks  ->  Tokens  s  -- ^ Chunk of input to match against  ->  m  (Tokens  s) 
```

还有两个基于 `tokens` 定义的解析器：

```
-- from "Text.Megaparsec":  chunk  ::  MonadParsec  e  s  m  =>  Tokens  s  ->  m  (Tokens  s)  chunk  =  tokens  (==)  -- from "Text.Megaparsec.Char" and "Text.Megaparsec.Byte":  string'  ::  (MonadParsec  e  s  m,  Data.CaseInsensitive.FoldCase  (Tokens  s))  =>  Tokens  s  ->  m  (Tokens  s)  string'  =  tokens  ((==)  `on`  Data.CaseInsensitive.mk) 
```

它们匹配输入的固定块，`chunk`（在 `Text.Megaparsec.Byte` 和 `Text.Megaparsec.Char` 中称为类型约束同义词 `string`），区分大小写，而 `string'` 则不区分大小写。对于不区分大小写的匹配，使用了 `case-insensitive` 包，因此有 `FoldCase` 约束。

让我们尝试使用新的组合子：

```
λ> parseTest (string "foo" :: Parser Text) "foo"
"foo"

λ> parseTest (string "foo" :: Parser Text) "bar"
1:1:
  |
1 | bar
  | ^
unexpected "bar"
expecting "foo"

λ> parseTest (string' "foo" :: Parser Text) "FOO"
"FOO"

λ> parseTest (string' "foo" :: Parser Text) "FoO"
"FoO"

λ> parseTest (string' "foo" :: Parser Text) "FoZ"
1:1:
  |
1 | FoZ
  | ^
unexpected "FoZ"
expecting "foo" 
```

好的，我们可以匹配单个标记和输入块。下一步是学习如何组合这些构建块，以编写更有趣的解析器。

## 单子和适用语法

最简单的组合解析器的方式是依次执行它们。`ParsecT` 和 `Parsec` 都是单子，而单子绑定正是我们用来顺序执行解析器的方法：

```
mySequence  ::  Parser  (Char,  Char,  Char)  mySequence  =  do  a  <-  char  'a'  b  <-  char  'b'  c  <-  char  'c'  return  (a,  b,  c) 
```

我们可以运行它来检查一切是否按预期工作：

```
λ> parseTest mySequence "abc"
('a','b','c')

λ> parseTest mySequence "bcd"
1:1:
  |
1 | bcd
  | ^
unexpected 'b'
expecting 'a'

λ> parseTest mySequence "adc"
1:2:
  |
1 | adc
  |  ^
unexpected 'd'
expecting 'b' 
```

如果我们记得每个单子也是一个应用函子，那么我们可以使用应用语法来实现一种可能的顺序执行：

```
mySequence  ::  Parser  (Char,  Char,  Char)  mySequence  =  (,,)  <$>  char  'a'  <*>  char  'b'  <*>  char  'c' 
```

第二个版本的工作方式与第一个版本完全相同。使用哪种风格通常是一种品味问题。单子风格可能更冗长但有时更清晰，而应用风格通常更简洁。话虽如此，单子风格当然更强大，因为单子比应用函子更强大。

## 使用 `eof` 强制消耗输入。

`Applicative` 往往足够强大，可以做一些非常有趣的事情。配备有具有单位元的结合运算符后，我们通过 `Alternative` 类型类在 Haskell 中得到了应用函子的幺半群。[`parser-combinators`](https://hackage.haskell.org/package/parser-combinators) 包基于 `Applicative` 和 `Alternative` 的概念提供了相当多的抽象组合子。`Text.Megaparsec` 模块从 `Control.Applicative.Combinators` 中重新导出它们。

最常见的组合子之一称为 `many`。它允许我们运行给定的解析器*零*次或更多次：

```
λ>  parseTest  (many  (char  'a')  ::  Parser  [Char])  "aaa"  "aaa"  λ>  parseTest  (many  (char  'a')  ::  Parser  [Char])  "aabbb"  "aa" 
```

第二个结果可能有点令人惊讶。解析器消耗了匹配的 `a`，但之后停止了。好吧，我们并没有说明在 `many (char 'a')` 之后要做什么！

大多数情况下，我们实际上希望强制解析器消耗整个输入，并报告解析错误，而不是悄悄地停止。这通过要求达到输入的结尾来完成。幸运的是，虽然输入的结尾只是一个概念，但有一个叫做 `eof :: MonadParsec e s m => m ()` 的原始操作，它不消耗任何东西，只在输入的结尾成功。让我们把它加入我们的解析器并再试一次：

```
λ> parseTest (many (char 'a') <* eof :: Parser [Char]) "aabbb"
1:3:
  |
1 | aabbb
  |   ^
unexpected 'b'
expecting 'a' or end of input 
```

我们的解析器中并未提及 `b`，而它们显然是意料之外的。

## 使用替代方案

从现在开始，我们将开发一个真正有用的解析器，可以解析以下形式的 URI：

```
scheme:[//[user:password@]host[:port]][/]path[?query][#fragment] 
```

我们应该记住方括号 `[]` 中的内容是可选的，它们可能出现也可能不出现在有效的 URI 中。`[]` 甚至可以嵌套以表达另一种可能性。我们将处理所有这些 [¹](#fn1)。

让我们从 `scheme` 开始。我们只接受我们已知的 scheme，例如：`data`、`file`、`ftp`、`http`、`https`、`irc` 和 `mailto`。

要匹配固定的字符序列，我们使用 `string`。要表达选择，我们使用 `Alternative` 类型类中的 `(<|>)` 方法。所以我们可以这样写：

```
{-# LANGUAGE OverloadedStrings #-}  {-# LANGUAGE RecordWildCards   #-}  module  Main  (main)  where  import  Control.Applicative  import  Control.Monad  import  Data.Text  (Text)  import  Data.Void  import  Text.Megaparsec  hiding  (State)  import  Text.Megaparsec.Char  import  qualified  Data.Text  as  T  import  qualified  Text.Megaparsec.Char.Lexer  as  L  type  Parser  =  Parsec  Void  Text  pScheme  ::  Parser  Text  pScheme  =  string  "data"  <|>  string  "file"  <|>  string  "ftp"  <|>  string  "http"  <|>  string  "https"  <|>  string  "irc"  <|>  string  "mailto" 
```

让我们试试它：

```
λ> parseTest pScheme ""
1:1:
  |
1 | <empty line>
  | ^
unexpected end of input
expecting "data", "file", "ftp", "http", "https", "irc", or "mailto"

λ> parseTest pScheme "dat"
1:1:
  |
1 | dat
  | ^
unexpected "dat"
expecting "data", "file", "ftp", "http", "https", "irc", or "mailto"

λ> parseTest pScheme "file"
"file"

λ> parseTest pScheme "irc"
"irc" 
```

看起来不错，但 `pScheme` 的定义有点重复。有一种方法可以使用 `choice` 组合子来编写 `pScheme`：

```
pScheme  ::  Parser  Text  pScheme  =  choice  [  string  "data"  ,  string  "file"  ,  string  "ftp"  ,  string  "http"  ,  string  "https"  ,  string  "irc"  ,  string  "mailto"  ] 
```

*`choice`只是`asum`的一个同义词——一个将`(<|>)`放在其元素之间折叠列表的操作，因此使用`choice`的这两个`pScheme`定义实际上是相同的，尽管使用`choice`的看起来可能更好一些。*

在方案之后，应该有一个冒号`:`。请记住，要求在某事物之后还要有另一件事物，我们使用单子绑定或`do`-notation：

```
data  Uri  =  Uri  {  uriScheme  ::  Text  }  deriving  (Eq,  Show)  pUri  ::  Parser  Uri  pUri  =  do  r  <-  pScheme  _  <-  char  ':'  return  (Uri  r) 
```

如果我们尝试运行`pUri`，我们会看到现在它需要跟在方案名称后面的`:`：

```
λ> parseTest pUri "irc"
1:4:
  |
1 | irc
  |    ^
unexpected end of input
expecting ':'

λ> parseTest pUri "irc:"
Uri {uriScheme = "irc"} 
```

尽管我们还没有完成方案解析。一个优秀的 Haskell 程序员会尝试以这样的方式定义类型，以确保不会表示不正确的数据。并非每个`Text`值都是有效的方案。让我们定义一个数据类型来表示方案，并让我们的`pScheme`解析器返回该类型的值：

```
data  Scheme  =  SchemeData  |  SchemeFile  |  SchemeFtp  |  SchemeHttp  |  SchemeHttps  |  SchemeIrc  |  SchemeMailto  deriving  (Eq,  Show)  pScheme  ::  Parser  Scheme  pScheme  =  choice  [  SchemeData  <$  string  "data"  ,  SchemeFile  <$  string  "file"  ,  SchemeFtp  <$  string  "ftp"  ,  SchemeHttp  <$  string  "http"  ,  SchemeHttps  <$  string  "https"  ,  SchemeIrc  <$  string  "irc"  ,  SchemeMailto  <$  string  "mailto"  ]  data  Uri  =  Uri  {  uriScheme  ::  Scheme  }  deriving  (Eq,  Show) 
```

*`(<$)`操作符只是将其左侧的值放入一个函子上下文中，替换当前的任何值。`a <$ f`与`const a <$> f`相同，但对于某些函子可能更有效率。*

让我们继续玩耍我们的解析器：

```
λ> parseTest pUri "https:"
1:5:
  |
1 | https:
  |     ^
unexpected 's'
expecting ':' 
```

嗯，`https`应该是一个有效的方案。你能找出问题出在哪吗？解析器逐个尝试备选项，而`http`匹配成功，因此它不再继续尝试`https`。解决方案是在`SchemeHttp <$ string "http"`行之前放置`SchemeHttps <$ string "https"`行。记住：*在备选项中，顺序很重要！*

现在`pUri`正常工作了：

```
λ> parseTest pUri "http:"
Uri {uriScheme = SchemeHttp}

λ> parseTest pUri "https:"
Uri {uriScheme = SchemeHttps}

λ> parseTest pUri "mailto:"
Uri {uriScheme = SchemeMailto}

λ> parseTest pUri "foo:"
1:1:
  |
1 | foo:
  | ^
unexpected "foo:"
expecting "data", "file", "ftp", "http", "https", "irc", or "mailto" 
```

## 使用`try`控制回溯

接下来处理的是`//[user:password@]host[:port]`——权威部分。这里我们有嵌套的可选部分，让我们更新`Uri`类型以反映这一点：

```
data  Uri  =  Uri  {  uriScheme  ::  Scheme  ,  uriAuthority  ::  Maybe  Authority  }  deriving  (Eq,  Show)  data  Authority  =  Authority  {  authUser  ::  Maybe  (Text,  Text)  -- (user, password)  ,  authHost  ::  Text  ,  authPort  ::  Maybe  Int  }  deriving  (Eq,  Show) 
```

现在我们需要讨论一个重要的概念，叫做*回溯*。回溯是一种在过程中“取消消耗”输入并返回的方式。这在分支中尤为重要。以下是一个例子：

```
alternatives  ::  Parser  (Char,  Char)  alternatives  =  foo  <|>  bar  where  foo  =  (,)  <$>  char  'a'  <*>  char  'b'  bar  =  (,)  <$>  char  'a'  <*>  char  'c' 
```

看起来合理，让我们试一试：

```
λ> parseTest alternatives "ab"
('a','b')

λ> parseTest alternatives "ac"
1:2:
  |
1 | ac
  |  ^
unexpected 'c'
expecting 'b' 
```

这里发生的是，`foo`的`char 'a'`部分（首先尝试）成功匹配并从输入流中消耗了`a`。然后`char 'b'`未能匹配`'c'`，所以我们得到了这个错误。这里的一个重要细节是，`(<|>)`甚至没有尝试`bar`，因为`foo`已经消耗了一些输入！

这是出于性能原因并且因为从`foo`的剩余部分运行`bar`是毫无意义的原因。`bar`希望从与`foo`相同的输入流位置运行。与例如`attoparsec`或前一章中的玩具组合子不同，`megaparsec`不会自动回溯，因此我们必须使用一个称为`try`的原语来显式表达我们回溯的意愿。如果`p`在消耗输入后失败，`try p`会失败，就好像没有消耗任何输入一样（实际上，它会回溯整个解析器状态）。这允许`(<|>)`尝试其右侧的备用选项：

```
alternatives  ::  Parser  (Char,  Char)  alternatives  =  try  foo  <|>  bar  where  foo  =  (,)  <$>  char  'a'  <*>  char  'b'  bar  =  (,)  <$>  char  'a'  <*>  char  'c' 
```

```
λ>  parseTest  alternatives  "ac"  ('a','c') 
```

所有真正消耗输入的原语（还有那些改变现有解析器行为的原语，比如 `try` 本身）在输入消耗方面都是“原子”的。这意味着如果它们失败，它们会自动回溯，因此它们无法在半路失败后消耗部分输入。这就是为什么 `pScheme` 与其替代项列表能够正常工作的原因：`string` 定义在 `tokens` 之上，而 `tokens` 是一个原语。我们要么用 `string` 完全匹配字符串，要么完全失败而根本不消耗输入流。

回到解析 URI，可以使用 `<|>` 来构建一个方便的组合器称为 `optional`：

```
optional  ::  Alternative  f  =>  f  a  ->  f  (Maybe  a)  optional  p  =  (Just  <$>  p)  <|>  pure  Nothing 
```

如果 `optional p` 中的 `p` 匹配成功，我们将得到它的结果为 `Just`，否则将返回 `Nothing`。正是我们想要的！不需要定义 `optional`，`Text.Megaparsec` 为我们重新导出了这个组合器。现在我们可以在 `pUri` 中使用它：

```
pUri  ::  Parser  Uri  pUri  =  do  uriScheme  <-  pScheme  void  (char  ':')  uriAuthority  <-  optional  .  try  $  do  -- (1)  void  (string  "//")  authUser  <-  optional  .  try  $  do  -- (2)  user  <-  T.pack  <$>  some  alphaNumChar  -- (3)  void  (char  ':')  password  <-  T.pack  <$>  some  alphaNumChar  void  (char  '@')  return  (user,  password)  authHost  <-  T.pack  <$>  some  (alphaNumChar  <|>  char  '.')  authPort  <-  optional  (char  ':'  *>  L.decimal)  -- (4)  return  Authority  {..}  -- (5)  return  Uri  {..}  -- (6) 
```

*我擅自接受了任何字母数字字符序列作为用户名和密码，并在主机格式上做了类似的任意简化。*

这里有几个重要的点：

+   在 (1) 和 (2) 中，我们需要用 `try` 封装 `optional` 的参数，因为它是一个复合解析器，不是原语。

+   (3) `some` 就像 `many` 一样，但要求其参数解析器至少匹配一次：`some p = (:) <$> p <*> many p`。

+   (4) 除非必要，不要使用 `try`！在这里，如果 `char ':'` 成功（它本身就是建立在 `token` 之上的，所以不需要 `try`），我们可以确定端口号必须在其后，因此我们只需用 `L.decimal` 要求一个十进制数。在匹配 `:` 后，我们已经做出了承诺，不需要回头了。

+   在 (5) 和 (6) 中，我们使用 `RecordWildCards` 语言扩展组装 `Authority` 和 `Uri` 值。

+   `void :: Functor f => f a -> f ()` 用于明确丢弃解析结果，否则我们会收到 GHC 的未使用值警告。

在 GHCi 中玩耍 `pUri` 并自己看看它的工作原理：

```
λ> parseTest (pUri <* eof) "https://mark:secret@example.com"
Uri
  { uriScheme = SchemeHttps
  , uriAuthority = Just (Authority
    { authUser = Just ("mark","secret")
    , authHost = "example.com"
    , authPort = Nothing } ) }

λ> parseTest (pUri <* eof) "https://mark:secret@example.com:123"
Uri
  { uriScheme = SchemeHttps
  , uriAuthority = Just (Authority
    { authUser = Just ("mark","secret")
    , authHost = "example.com"
    , authPort = Just 123 } ) }

λ> parseTest (pUri <* eof) "https://example.com:123"
Uri
  { uriScheme = SchemeHttps
  , uriAuthority = Just (Authority
    { authUser = Nothing
    , authHost = "example.com"
    , authPort = Just 123 } ) }

λ> parseTest (pUri <* eof) "https://mark@example.com:123"
1:13:
  |
1 | https://mark@example.com:123
  |             ^
unexpected '@'
expecting '.', ':', alphanumeric character, or end of input 
```

## 解析器调试

但是，你可能会发现有一个问题：

```
λ> parseTest (pUri <* eof) "https://mark:@example.com"
1:7:
  |
1 | https://mark:@example.com
  |       ^
unexpected '/'
expecting end of input 
```

解析错误可能会更好！怎么办？弄清楚发生了什么的最简单方法是使用 `Text.Megaparsec.Debug` 模块中的内置 `dbg` 辅助工具：

```
dbg  ::  (VisualStream  s,  ShowToken  (Token  s),  ShowErrorComponent  e,  Show  a)  =>  String  -- ^ Debugging label  ->  ParsecT  e  s  m  a  -- ^ Parser to debug  ->  ParsecT  e  s  m  a  -- ^ Parser that prints debugging messages 
```

`VisualStream` 类型类为可在屏幕上以可读形式打印的输入流定义。我们在这里不详细讨论它。

让我们在 `pUri` 中使用它：

```
pUri  ::  Parser  Uri  pUri  =  do  uriScheme  <-  dbg  "scheme"  pScheme  void  (char  ':')  uriAuthority  <-  dbg  "auth"  .  optional  .  try  $  do  void  (string  "//")  authUser  <-  dbg  "user"  .  optional  .  try  $  do  user  <-  T.pack  <$>  some  alphaNumChar  void  (char  ':')  password  <-  T.pack  <$>  some  alphaNumChar  void  (char  '@')  return  (user,  password)  authHost  <-  T.pack  <$>  dbg  "host"  (some  (alphaNumChar  <|>  char  '.'))  authPort  <-  dbg  "port"  $  optional  (char  ':'  *>  L.decimal)  return  Authority  {..}  return  Uri  {..} 
```

然后让我们再次尝试在不幸的输入上运行 `pUri`：

```
λ> parseTest (pUri <* eof) "https://mark:@example.com"
scheme> IN: "https://mark:@example.com"
scheme> MATCH (COK): "https"
scheme> VALUE: SchemeHttps

user> IN: "mark:@example.com"
user> MATCH (EOK): <EMPTY>
user> VALUE: Nothing

host> IN: "mark:@example.com"
host> MATCH (COK): "mark"
host> VALUE: "mark"

port> IN: ":@example.com"
port> MATCH (CERR): ':'
port> ERROR:
port> 1:14:
port> unexpected '@'
port> expecting integer

auth> IN: "//mark:@example.com"
auth> MATCH (EOK): <EMPTY>
auth> VALUE: Nothing

1:7:
  |
1 | https://mark:@example.com
  |       ^
unexpected '/'
expecting end of input 
```

现在我们可以看到 `megaparsec` 内部到底发生了什么：

+   `scheme` 成功匹配。

+   `user` 失败了：虽然有一个用户名 `mark`，但在冒号 `:` 后面没有密码（我们要求密码不为空）。我们失败了，多亏了 `try`，回溯。

+   `host` 从与 `user` 相同的点开始，并试图现在将输入解释为主机名。我们可以看到它成功了，并将 `mark` 作为主机名返回。

+   主机后可能有一个端口号，所以现在轮到 `port` 了。它看到 `:`，但在那之后没有整数，所以 `port` 也失败了。

+   整个 `auth` 解析器因此失败了（`port` 在 `auth` 内部，而且它失败了）。

+   `auth` 解析器返回 `Nothing`，因为它无法解析任何内容。现在 `eof` 要求我们已经达到了输入的结尾，但事实并非如此，所以我们得到了最终的错误消息。

怎么办？这是一个使用 `try` 封装大量代码可能会使解析错误变得更糟的情况的例子。让我们再看一下我们想要解析的语法：

```
scheme:[//[user:password@]host[:port]][/]path[?query][#fragment] 
```

我们在寻找什么？某种允许我们承诺特定解析分支的东西。就像端口，在我们看到冒号 `:` 时，我们可以确定端口号必须跟随。如果你仔细观察，你会看到双斜杠 `//` 是我们 URI 中授权部分的标志。由于我们用“原子”解析器 (`string`) 匹配 `//`，一旦我们匹配上 `//`，我们可以确保要求授权部分。让我们从 `pUri` 中移除第一个 `try`：

```
pUri  ::  Parser  Uri  pUri  =  do  uriScheme  <-  pScheme  void  (char  ':')  uriAuthority  <-  optional  $  do  -- removed 'try' on this line  void  (string  "//")  authUser  <-  optional  .  try  $  do  user  <-  T.pack  <$>  some  alphaNumChar  void  (char  ':')  password  <-  T.pack  <$>  some  alphaNumChar  void  (char  '@')  return  (user,  password)  authHost  <-  T.pack  <$>  some  (alphaNumChar  <|>  char  '.')  authPort  <-  optional  (char  ':'  *>  L.decimal)  return  Authority  {..}  return  Uri  {..} 
```

现在我们得到了一个更好的解析错误：

```
λ> parseTest (pUri <* eof) "https://mark:@example.com"
1:14:
  |
1 | https://mark:@example.com
  |              ^
unexpected '@'
expecting integer 
```

虽然这仍然有点误导，但好吧，这是我选择的一个棘手的例子。有很多 `optional`。

## 标记和隐藏事物

有时候预期项的列表可能会变得相当长。记住，当我们尝试使用一个未被识别的方案时会得到什么？

```
λ> parseTest (pUri <* eof) "foo://example.com"
1:1:
  |
1 | foo://example.com
  | ^
unexpected "foo://"
expecting "data", "file", "ftp", "http", "https", "irc", or "mailto" 
```

`megaparsec` 提供了一种用自定义内容覆盖预期项的方式，通常称为 *label*。这是通过 `label` 原语来完成的（它在 `(<?>)` 操作符的形式中有一个同义词）：

```
pUri  ::  Parser  Uri  pUri  =  do  uriScheme  <-  pScheme  <?>  "valid scheme"  -- the rest stays the same 
```

```
λ> parseTest (pUri <* eof) "foo://example.com"
1:1:
  |
1 | foo://example.com
  | ^
unexpected "foo://"
expecting valid scheme 
```

我们可以继续添加更多标签，以使错误消息更容易理解：

```
pUri  ::  Parser  Uri  pUri  =  do  uriScheme  <-  pScheme  <?>  "valid scheme"  void  (char  ':')  uriAuthority  <-  optional  $  do  void  (string  "//")  authUser  <-  optional  .  try  $  do  user  <-  T.pack  <$>  some  alphaNumChar  <?>  "username"  void  (char  ':')  password  <-  T.pack  <$>  some  alphaNumChar  <?>  "password"  void  (char  '@')  return  (user,  password)  authHost  <-  T.pack  <$>  some  (alphaNumChar  <|>  char  '.')  <?>  "hostname"  authPort  <-  optional  (char  ':'  *>  label  "port number"  L.decimal)  return  Authority  {..}  return  Uri  {..} 
```

例如：

```
λ> parseTest (pUri <* eof) "https://mark:@example.com"
1:14:
  |
1 | https://mark:@example.com
  |              ^
unexpected '@'
expecting port number 
```

另一个原语被称为 `hidden`。如果 `label` 重命名事物，`hidden` 只是彻底移除它们。比较一下：

```
λ> parseTest (many (char 'a') >> many (char 'b') >> eof :: Parser ()) "d"
1:1:
  |
1 | d
  | ^
unexpected 'd'
expecting 'a', 'b', or end of input

λ> parseTest (many (char 'a') >> hidden (many (char 'b')) >> eof :: Parser ()) "d"
1:1:
  |
1 | d
  | ^
unexpected 'd'
expecting 'a' or end of input 
```

当希望减少错误消息的噪音时，`hidden` 是很有用的。例如，在解析编程语言时，删除“期望空白”消息是个好主意，因为通常每个标记后面可能都有空白。

*练习：完成 `pUri` 解析器留给读者作为练习，现在已经解释了所有必要的工具。*

## 运行解析器

我们详细探讨了如何构建解析器，但还没有检查允许我们运行它们的函数，除了 `parseTest`。

传统上，从您的程序运行解析器的“默认”函数是 `parse`。但是 `parse` 实际上是 `runParser` 的同义词：

```
runParser  ::  Parsec  e  s  a  -- ^ Parser to run  ->  String  -- ^ Name of source file  ->  s  -- ^ Input for parser  ->  Either  (ParseErrorBundle  s  e)  a 
```

第二个参数只是一个文件名，在生成的解析错误中将包含这个文件名，`megaparsec` 不会从该文件中读取任何内容，因为实际输入作为函数的第三个参数。

`runParser` 允许我们运行 `Parsec` 单子，正如我们已经知道的那样，这是 `ParsecT` 的非变换器版本：

```
type  Parsec  e  s  =  ParsecT  e  s  Identity 
```

`runParser` 有三个兄弟函数：`runParser'`、`runParserT` 和 `runParserT'`。带有 `T` 后缀的版本运行 `ParsecT` 单子变换器，而“prime”版本则接受并返回解析器状态。让我们把所有这些函数放到一个表格中：

| 参数 | 运行 `Parsec` | 运行 `ParsecT` |
| --- | --- | --- |
| 输入和文件名 | `runParser` | `runParserT` |
| 自定义初始状态 | `runParser'` | `runParserT'` |

如果你需要自定义初始状态，比如想将制表符宽度设置为非标准值（默认值为 `8`），可能就需要自定义初始状态。例如，这里是 `runParser'` 的类型签名：

```
runParser'  ::  Parsec  e  s  a  -- ^ Parser to run  ->  State  s  -- ^ Initial state  ->  (State  s,  Either  (ParseErrorBundle  s  e)  a) 
```

手动修改 `State` 是库的高级用法，我们这里不打算详细描述它。

如果你想知道 `ParseErrorBundle` 是什么，我们将在 [以下章节之一](#parse-errors) 进行讨论。

## `MonadParsec` 类型类

`megaparsec` 中的所有工具都适用于 `MonadParsec` 类型类的任何实例。该类型类抽象出*原始组合子*——所有 `megaparsec` 解析器的基本构建块，这些组合子无法通过其他组合子表达。

在类型类中拥有原始组合子允许 `megaparsec` 的主要具体单子变换器 `ParsecT` 被包装在 MTL 家族的熟悉变换器中，从而实现单子栈层之间的不同交互。为了更好地理解动机，回想一下单子栈中层的顺序很重要。如果我们像这样结合 `ReaderT` 和 `State`：

```
type  MyStack  a  =  ReaderT  MyContext  (State  MyState)  a 
```

外层的 `ReaderT` 不能检查底层 `m` 层的内部结构。`ReaderT` 的 `Monad` 实例描述了绑定策略：

```
newtype  ReaderT  r  m  a  =  ReaderT  {  runReaderT  ::  r  ->  m  a  }  instance  Monad  m  =>  Monad  (ReaderT  r  m)  where  m  >>=  k  =  ReaderT  $  \r  ->  do  a  <-  runReaderT  m  r  runReaderT  (k  a)  r 
```

实际上，我们唯一知道关于 `m` 的是它是 `Monad` 的一个实例，所以 `m` 的状态只能通过单子绑定传递给 `k`。这通常是我们从 `ReaderT` 的 `(>>=)` 中期望的。

`Alternative` 类型类的 `(<|>)` 方法工作方式不同——它“分割”状态，两个解析的分支不再联系，因此在第一个分支被丢弃时，其状态的更改也被丢弃，并且不能影响第二个分支（我们在状态失败时“回溯”状态）。

为了说明，让我们看一下 `ReaderT` 的 `Alternative` 定义：

```
instance  Alternative  m  =>  Alternative  (ReaderT  r  m)  where  empty  =  liftReaderT  empty  ReaderT  m  <|>  ReaderT  n  =  ReaderT  $  \r  ->  m  r  <|>  n  r 
```

这一切都非常好，因为 `ReaderT` 是一个“无状态”的单子变换器，可以轻松地将实际工作委托给内部单子（这里 `m` 的 `Alternative` 实例非常方便），而不需要结合与 `ReaderT` 本身相关的单子状态（它没有状态）。

现在让我们来看看 `State`。由于 `State s a` 只是 `StateT s Identity a` 的类型同义词，我们应该查看 `StateT s m` 本身的 `Alternative` 实例：

```
instance  (Functor  m,  Alternative  m)  =>  Alternative  (StateT  s  m)  where  empty  =  StateT  $  \_  ->  empty  StateT  m  <|>  StateT  n  =  StateT  $  \s  ->  m  s  <|>  n  s 
```

在这里我们可以看到状态 `s` 的分割，就像我们看到读取器上下文 `r` 的共享一样。然而，有一个区别，因为表达式 `m s` 和 `n s` 产生有状态的结果：连同单子值一起，它们以元组的形式返回新状态。在这里我们要么选择 `m s`，要么选择 `n s`，自然地实现回溯。

`ParsecT` 是什么？现在我们考虑将 `State` 放入 `ParsecT` 中，像这样：

```
type  MyStack  a  =  ParsecT  Void  Text  (State  MyState)  a 
```

`ParsecT`比`ReaderT`更复杂，其`(<|>)`的实现需要做更多的工作：

+   管理解析器本身的状态；

+   合并解析错误（如果适用），如果它们发生。

因此，`ParsecT`在`Alternative`的实例中的`(<|>)`的实现不能将其工作委托给底层单子`State MyState`的`Alternative`实例，因此不会发生`MyState`的分割 - 我们没有回溯。

让我们用一个例子来演示这一点：

```
{-# LANGUAGE OverloadedStrings #-}  module  Main  (main)  where  import  Control.Applicative  import  Control.Monad.State.Strict  import  Data.Text  (Text)  import  Data.Void  import  Text.Megaparsec  hiding  (State)  type  Parser  =  ParsecT  Void  Text  (State  String)  parser0  ::  Parser  String  parser0  =  a  <|>  b  where  a  =  "foo"  <$  put  "branch A"  b  =  get  <*  put  "branch B"  parser1  ::  Parser  String  parser1  =  a  <|>  b  where  a  =  "foo"  <$  put  "branch A"  <*  empty  b  =  get  <*  put  "branch B"  main  ::  IO  ()  main  =  do  let  run  p  =  runState  (runParserT  p  ""  "")  "initial"  (Right  a0,  s0)  =  run  parser0  (Right  a1,  s1)  =  run  parser1  putStrLn  "Parser 0"  putStrLn  ("Result:      "  ++  show  a0)  putStrLn  ("Final state: "  ++  show  s0)  putStrLn  "Parser 1"  putStrLn  ("Result:      "  ++  show  a1)  putStrLn  ("Final state: "  ++  show  s1) 
```

这是运行程序的结果：

```
Parser 0
Result:      "foo"
Final state: "branch A"
Parser 1
Result:      "branch A"
Final state: "branch B" 
```

使用 `parser0` 我们可以看到分支 `b` 没有被尝试。然而，使用 `parser1` 可以明显看到最终的结果 —— 由 `get` 返回的值 —— 来自分支 `a`，尽管由于 `empty` 而失败，并且是分支 `b` 成功了（在解析的上下文中，`empty` 意味着“立即失败，并且没有任何关于发生了什么的信息”）。没有发生回溯。

如果我们想要在我们的解析器中提供回溯自定义状态，我们可以允许将`ParsecT`包装在`StateT`*内部*：

```
type  MyStack  a  =  StateT  MyState  (ParsecT  Void  Text  Identity)  a 
```

现在如果我们在`MyStack`中使用`(<|>)`，那么所使用的实例就是`StateT`：

```
StateT  m  <|>  StateT  n  =  StateT  $  \s  ->  m  s  <|>  n  s 
```

这样就给我们带来了回溯状态，然后将其余工作委托给其内部单子 `ParsecT` 的 `Alternative` 实例 —— 这种行为正是我们想要的：

```
{-# LANGUAGE OverloadedStrings #-}  module  Main  (main)  where  import  Control.Applicative  import  Control.Monad.Identity  import  Control.Monad.State.Strict  import  Data.Text  (Text)  import  Data.Void  import  Text.Megaparsec  hiding  (State)  type  Parser  =  StateT  String  (ParsecT  Void  Text  Identity)  parser  ::  Parser  String  parser  =  a  <|>  b  where  a  =  "foo"  <$  put  "branch A"  <*  empty  b  =  get  <*  put  "branch B"  main  ::  IO  ()  main  =  do  let  p  =  runStateT  parser  "initial"  Right  (a,  s)  =  runParser  p  ""  ""  putStrLn  ("Result:      "  ++  show  a)  putStrLn  ("Final state: "  ++  show  s) 
```

程序输出：

```
Result:      "initial"
Final state: "branch B" 
```

要使这种方法可行，`StateT`应支持整套原始解析器，这样我们就可以像使用`ParsecT`一样使用它。换句话说，它应该是`MonadParsec`的一个实例，就像它不仅是`MonadState`的实例，而且例如它的内部单子是`MonadWriter`的实例（在MTL中）：

```
instance  MonadWriter  w  m  =>  MonadWriter  w  (StateT  s  m)  where  … 
```

的确，我们可以将内部`MonadParsec`实例的基本操作提升到`StateT`中：

```
instance  MonadParsec  e  s  m  =>  MonadParsec  e  s  (StateT  st  m)  where  … 
```

`megaparsec`为所有MTL单子变换器定义了`MonadParsec`的实例，以便用户可以自由地将这些变换器插入`ParsecT`中或将`ParsecT`包装在这些变换器中，在单子栈的不同层之间实现不同类型的交互。

## 词法分析

*词法分析* 是将输入流转换为标记流的过程：整数、关键字、符号等。这比直接解析原始输入更容易，或者这些标记作为生成器创建的解析器的输入期望。词法分析可以在单独的通过中用外部工具（如 `alex`）执行，但是 `megaparsec` 也提供了函数，应该简化以无缝方式编写词法分析器作为解析器的一部分。

有两个词法分析器模块 `Text.Megaparsec.Char.Lexer` 用于字符流和 `Text.Megaparsec.Byte.Lexer` 用于字节流。我们将使用 `Text.Megaparsec.Char.Lexer`，因为我们处理的是严格的 `Text` 作为输入流，但是如果你希望使用 `ByteString`，大多数函数也在 `Text.Megaparsec.Byte.Lexer` 中有镜像。

### 空格

我们需要讨论的第一个主题是如何处理空白。在每个标记之前或之后以一致的方式消耗空白非常有帮助。Megaparsec 的词法分析器模块遵循的策略是“假定标记之前没有空白，并在标记之后消耗所有空白”。

要消耗空白，我们需要一个特殊的解析器，我们将其称为*空白消耗器*。`Text.Megaparsec.Char.Lexer` 模块提供了一个辅助函数，允许构建一个通用的空白消耗器：

```
space  ::  MonadParsec  e  s  m  =>  m  ()  -- ^ A parser for space characters which does not accept empty
          -- input (e.g. 'space1')  ->  m  ()  -- ^ A parser for a line comment (e.g. 'skipLineComment')  ->  m  ()  -- ^ A parser for a block comment (e.g. 'skipBlockComment')  ->  m  () 
```

`space` 函数的文档本身已经非常详尽，但让我们用一个例子来补充说明：

```
{-# LANGUAGE OverloadedStrings #-}  module  Main  (main)  where  import  Data.Text  (Text)  import  Data.Void  import  Text.Megaparsec  import  Text.Megaparsec.Char  import  qualified  Text.Megaparsec.Char.Lexer  as  L  -- (1)  type  Parser  =  Parsec  Void  Text  sc  ::  Parser  ()  sc  =  L.space  space1  -- (2)  (L.skipLineComment  "//")  -- (3)  (L.skipBlockComment  "/*"  "*/")  -- (4) 
```

一些注意事项：

+   应当以限定方式导入 `Text.Megaparsec.Char.Lexer`，因为它包含与 `Text.Megaparsec.Char` 等模块中的名称冲突的名称，例如 `space`。

+   `L.space` 的第一个参数应该是一个解析器，用于提取空白。一个重要的细节是，它不应接受空输入，否则 `L.space` 将陷入无限循环。`space1` 是 `Text.Megaparsec.Char` 中满足要求的解析器。

+   `L.space` 的第二个参数定义了如何跳过行注释，即以特定序列开始并以行结束的注释。`skipLineComment` 辅助函数使我们能够轻松地创建一个用于行注释的辅助解析器。

+   `L.space` 的第三个参数定义了如何处理块注释：即在起始和结束标记之间的所有内容。`skipBlockComment` 辅助函数使我们能够处理非嵌套块注释。如果需要支持嵌套块注释，则应改用 `skipBlockCommentNested`。

在操作上，`L.space` 依次尝试这三个解析器，直到无法应用为止，这意味着我们已经消耗了所有的空白。了解这一点后，应该明白，如果您的语法不包括块或行注释，可以将 `empty` 作为 `L.space` 的第二个和/或第三个参数传递。`empty` 是 `(<|>)` 的身份元素，将导致 `L.space` 尝试下一个空白组件的解析器——这正是所需的行为。

有了空白消耗器 `sc`，我们可以定义各种与空白相关的辅助函数：

```
lexeme  ::  Parser  a  ->  Parser  a  lexeme  =  L.lexeme  sc  symbol  ::  Text  ->  Parser  Text  symbol  =  L.symbol  sc 
```

+   `lexeme` 是一个包装器，用于提取所有尾随空白的词元，使用提供的空白消耗器。

+   `symbol` 是一个解析器，使用 `string` 内部匹配给定文本，然后类似地提取所有尾随空白。

我们将很快看到它们如何一起运行，但首先需要介绍几个来自 `Text.Megaparsec.Char.Lexer` 的辅助函数。

### 字符和字符串字面量

解析字符和字符串字面量可能会很棘手，因为涉及到各种转义规则。为了简化生活，`megaparsec` 提供了 `charLiteral` 解析器：

```
charLiteral  ::  (MonadParsec  e  s  m,  Token  s  ~  Char)  =>  m  Char 
```

`charLiteral` 的工作是解析单个字符，根据 Haskell 报告中字符文字的语法可能进行转义。请注意，它并不解析文字周围的引号，有两个原因：

+   因此用户可以控制如何引用字符文字，

+   所以 `charLiteral` 也可以用来解析字符串文字。

下面是一些构建在 `charLiteral` 之上的示例解析器：

```
charLiteral  ::  Parser  Char  charLiteral  =  between  (char  '\'')  (char  '\'')  L.charLiteral  stringLiteral  ::  Parser  String  stringLiteral  =  char  '\"'  *>  manyTill  L.charLiteral  (char  '\"') 
```

+   要将 `L.charLiteral` 转换为用于解析字符文字的解析器，我们只需添加包围的引号。在这里，我们遵循 Haskell 语法并使用单引号。`between` 组合子简单定义为：`between open close p = open *> p <* close`。

+   `stringLiteral` 使用 `L.charLiteral` 解析双引号括起来的字符串文字中的各个字符。

第二个函数也很有趣，因为它使用了 `manyTill` 组合子：

```
manyTill  ::  Alternative  m  =>  m  a  ->  m  end  ->  m  [a]  manyTill  p  end  =  go  where  go  =  ([]  <$  end)  <|>  ((:)  <$>  p  <*>  go) 
```

`manyTill` 尝试在每次迭代中应用 `end` 解析器，如果失败，则运行 `p` 解析器并在列表中累积 `p` 的结果。

当你希望至少有一个项目存在时，还有 `someTill`。

### 数字

最后，一个非常常见的需求是解析数字。对于整数，有三个辅助函数可以解析十进制、八进制和十六进制表示的值：

```
decimal,  octal,  hexadecimal  ::  (MonadParsec  e  s  m,  Token  s  ~  Char,  Num  a)  =>  m  a 
```

使用它们很容易：

```
integer  ::  Parser  Integer  integer  =  lexeme  L.decimal 
```

```
λ> parseTest (integer <* eof) "123  "
123

λ> parseTest (integer <* eof) "12a  "
1:3:
  |
1 | 12a
  |   ^
unexpected 'a'
expecting end of input or the rest of integer 
```

`scientific` 接受整数和分数语法，而 `float` 仅接受分数语法。`scientific` 返回 `scientific` 包的 `Scientific` 类型，而 `float` 在其结果类型中是多态的，可以返回 `RealFloat` 的任何实例：

```
scientific  ::  (MonadParsec  e  s  m,  Token  s  ~  Char)  =>  m  Scientific  float  ::  (MonadParsec  e  s  m,  Token  s  ~  Char,  RealFloat  a)  =>  m  a 
```

例如：

```
float  ::  Parser  Double  float  =  lexeme  L.float 
```

```
λ> parseTest (float <* eof) "123"
1:4:
  |
1 | 123
  |    ^
unexpected end of input
expecting '.', 'E', 'e', or digit

λ> parseTest (float <* eof) "123.45"
123.45

λ> parseTest (float <* eof) "123d"
1:4:
  |
1 | 123d
  |    ^
unexpected 'd'
expecting '.', 'E', 'e', or digit 
```

注意，所有这些解析器都不解析带符号的数字。要制作带符号数字的解析器，我们需要用 `signed` 组合子包装现有解析器：

```
signedInteger  ::  Parser  Integer  signedInteger  =  L.signed  sc  integer  signedFloat  ::  Parser  Double  signedFloat  =  L.signed  sc  float 
```

`signed` 的第一个参数——空格消耗器——控制符号和实际数字之间的空白如何消耗。如果你不想在其中允许空格，只需传递 `return ()`。

## `notFollowedBy` 和 `lookAhead`。

还有两个原语（除了 `try`）可以在输入流中进行前瞻，而不实际推进解析位置。

第一个被称为 `notFollowedBy`：

```
notFollowedBy  ::  MonadParsec  e  s  m  =>  m  a  ->  m  () 
```

它仅在其参数解析器失败时成功，并且从不消耗任何输入或修改解析器状态。

举例来说，当你可能希望使用 `notFollowedBy` 时，请考虑关键字的解析：

```
pKeyword  ::  Text  ->  Parser  Text  pKeyword  keyword  =  lexeme  (string  keyword) 
```

这个解析器有一个问题：如果我们匹配的关键字只是标识符的前缀呢？在这种情况下，它绝对不是关键字。因此，我们必须通过使用 `notFollowedBy` 来消除这种情况：

```
pKeyword  ::  Text  ->  Parser  Text  pKeyword  keyword  =  lexeme  (string  keyword  <*  notFollowedBy  alphaNumChar) 
```

另一个原语是 `lookAhead`：

```
lookAhead  ::  MonadParsec  e  s  m  =>  m  a  ->  m  a 
```

如果 `lookAhead` 的参数 `p` 成功，那么整个结构 `lookAhead p` 也成功，但输入流（以及整个解析器状态）保持不变，即不消耗任何内容。

一个例子是在已解析的值上执行检查，然后根据情况失败或继续成功。这种习惯用法可以通过如下代码表达：

```
withPredicate1  ::  (a  ->  Bool)  -- ^ The check to perform on parsed input  ->  String  -- ^ Message to print when the check fails  ->  Parser  a  -- ^ Parser to run  ->  Parser  a  -- ^ Resulting parser that performs the check  withPredicate1  f  msg  p  =  do  r  <-  lookAhead  p  if  f  r  then  p  else  fail  msg 
```

这演示了 `lookAhead` 的使用，但我们还应该注意，当检查成功时，我们会执行两次解析，这是不好的。这里是使用 `getOffset` 函数的另一种解决方案：

```
withPredicate2  ::  (a  ->  Bool)  -- ^ The check to perform on parsed input  ->  String  -- ^ Message to print when the check fails  ->  Parser  a  -- ^ Parser to run  ->  Parser  a  -- ^ Resulting parser that performs the check  withPredicate2  f  msg  p  =  do  o  <-  getOffset  r  <-  p  if  f  r  then  return  r  else  do  setOffset  o  fail  msg 
```

这样我们只需将输入流中的偏移设置为运行 `p` 之前的偏移，然后失败。现在剩余未消耗与偏移位置之间存在不匹配，但在这种情况下并不重要，因为我们通过调用 `fail` 立即结束解析。在其他情况下可能会有所不同。我们将在本章后面看到如何在这种情况下做得更好。

## 解析表达式

“表达式”是指由项和应用于这些项的运算符形成的结构。运算符可以是前缀、中缀和后缀的，左结合和右结合的，具有不同的优先级。这样的构造的一个例子就是从学校熟悉的算术表达式：

```
a * (b + 2) 
```

在这里我们可以看到两种类型的项：变量（`a` 和 `b`）和整数（`2`）。还有两个运算符：`*` 和 `+`。

编写表达式解析器可能需要一段时间来正确完成。为了帮助解决这个问题，[`parser-combinators`](https://hackage.haskell.org/package/parser-combinators) 包提供了 `Control.Monad.Combinators.Expr` 模块，该模块仅导出两个东西：`Operator` 数据类型和 `makeExprParser` 辅助函数。两者都有很好的文档说明，因此在本节中我们不会重复文档，而是将编写一个简单但完全功能的表达式解析器。

让我们从定义一个表示表达式的数据类型开始，作为 [AST](https://zh.wikipedia.org/wiki/%E6%8A%BD%E8%B1%A1%E8%AF%AD%E6%B3%95%E6%A0%91)：

```
data  Expr  =  Var  String  |  Int  Int  |  Negation  Expr  |  Sum  Expr  Expr  |  Subtr  Expr  Expr  |  Product  Expr  Expr  |  Division  Expr  Expr  deriving  (Eq,  Ord,  Show) 
```

要使用 `makeExprParser`，我们需要提供一个项解析器和一个运算符表：

```
makeExprParser  ::  MonadParsec  e  s  m  =>  m  a  -- ^ Term parser  ->  [[Operator  m  a]]  -- ^ Operator table, see 'Operator'  ->  m  a  -- ^ Resulting expression parser 
```

让我们从项解析器开始。当处理诸如结合性和优先级等事物时，将项视为一个不可分割整体的盒子是很有帮助的。在我们的情况下，有三件事情属于这个类别：变量、整数和括号中的整个表达式。使用前面章节的定义，我们可以将项解析器定义为：

```
pVariable  ::  Parser  Expr  pVariable  =  Var  <$>  lexeme  ((:)  <$>  letterChar  <*>  many  alphaNumChar  <?>  "variable")  pInteger  ::  Parser  Expr  pInteger  =  Int  <$>  lexeme  L.decimal  parens  ::  Parser  a  ->  Parser  a  parens  =  between  (symbol  "(")  (symbol  ")")  pTerm  ::  Parser  Expr  pTerm  =  choice  [  parens  pExpr  ,  pVariable  ,  pInteger  ]  pExpr  ::  Parser  Expr  pExpr  =  makeExprParser  pTerm  operatorTable  operatorTable  ::  [[Operator  Parser  Expr]]  operatorTable  =  undefined  -- TODO 
```

`pVariable`、`pInteger` 和 `parens` 的定义现在应该已经没有疑问了。在这里我们也很幸运，因为在 `pTerm` 中不需要使用 `try`，因为语法没有重叠：

+   如果我们看到一个开括号 `(`，我们知道接下来会有一个括号中的表达式，因此我们致力于该分支；

+   如果我们看到一个字母，我们知道它是标识符的起始；

+   如果我们看到一个数字，我们知道它是整数的起始。

最后，为了完成 `pExpr`，我们需要定义 `operatorTable`。从类型上我们可以看出它是一个嵌套列表。每个内部列表是我们想要支持的运算符列表，它们都具有相等的优先级。外部列表按降序排序，因此我们在其中放置运算符组时，它们绑定得更紧：

```
data  Operator  m  a  -- N.B.  =  InfixN  (m  (a  ->  a  ->  a))  -- ^ Non-associative infix  |  InfixL  (m  (a  ->  a  ->  a))  -- ^ Left-associative infix  |  InfixR  (m  (a  ->  a  ->  a))  -- ^ Right-associative infix  |  Prefix  (m  (a  ->  a))  -- ^ Prefix  |  Postfix  (m  (a  ->  a))  -- ^ Postfix  operatorTable  ::  [[Operator  Parser  Expr]]  operatorTable  =  [  [  prefix  "-"  Negation  ,  prefix  "+"  id  ]  ,  [  binary  "*"  Product  ,  binary  "/"  Division  ]  ,  [  binary  "+"  Sum  ,  binary  "-"  Subtr  ]  ]  binary  ::  Text  ->  (Expr  ->  Expr  ->  Expr)  ->  Operator  Parser  Expr  binary  name  f  =  InfixL  (f  <$  symbol  name)  prefix,  postfix  ::  Text  ->  (Expr  ->  Expr)  ->  Operator  Parser  Expr  prefix  name  f  =  Prefix  (f  <$  symbol  name)  postfix  name  f  =  Postfix  (f  <$  symbol  name) 
```

注意我们如何在 `binary` 的 `InfixL` 中放置 `Parser (Expr -> Expr -> Expr)`，以及类似地在 `prefix` 和 `postfix` 中放置 `Parser (Expr -> Expr)`。也就是说，我们运行 `symbol name` 并返回一个函数，以便将其应用于术语，以获得类型 `Expr` 的最终结果。

现在我们可以尝试我们的解析器，它已经准备好了！

```
λ> parseTest (pExpr <* eof) "a * (b + 2)"
Product (Var "a") (Sum (Var "b") (Int 2))

λ> parseTest (pExpr <* eof) "a * b + 2"
Sum (Product (Var "a") (Var "b")) (Int 2)

λ> parseTest (pExpr <* eof) "a * b / 2"
Division (Product (Var "a") (Var "b")) (Int 2)

λ> parseTest (pExpr <* eof) "a * (b $ 2)"
1:8:
  |
1 | a * (b $ 2)
  |        ^
unexpected '$'
expecting ')' or operator 
```

`Control.Monad.Combinators.Expr` 模块的文档包含了一些在某些非标准情况下有用的提示，因此阅读它也是一个好主意。

## 缩进敏感解析

`Text.Megaparsec.Char.Lexer` 模块包含在解析缩进敏感语法时应该有用的工具。我们首先将回顾可用的组合子，然后通过编写一个缩进敏感的解析器来使用它们。

### `nonIndented` 和 `indentBlock`

让我们从最简单的东西开始——`nonIndented`：

```
nonIndented  ::  (TraversableStream  s,  MonadParsec  e  s  m)  =>  m  ()  -- ^ How to consume indentation (white space)  ->  m  a  -- ^ Inner parser  ->  m  a 
```

它允许我们确保其内部解析器消耗的输入*不*是缩进的。这是高级缩进敏感输入解析背后的模型的一部分。我们声明有一些顶层项目是不缩进的，并且所有缩进的标记都是这些顶层定义的直接或间接子项。在 `megaparsec` 中，我们不需要额外的状态来表达这一点。由于缩进总是相对的，我们的想法是通过纯解析器的组合显式地关联参考标记和缩进标记的解析器，从而定义缩进敏感语法。

那么，我们如何为缩进块定义解析器？让我们看一下 `indentBlock` 的签名：

```
indentBlock  ::  (TraversableStream  s,  MonadParsec  e  s  m,  Token  s  ~  Char)  =>  m  ()  -- ^ How to consume indentation (white space)  ->  m  (IndentOpt  m  a  b)  -- ^ How to parse “reference” token  ->  m  a 
```

首先，我们指定如何处理缩进。这里需要注意的重要一点是，这个占用空间的解析器 *必须* 同时消耗换行符，而标记（参考标记和缩进标记）通常不应在其后消耗换行符。

正如你所看到的，第二个参数允许我们解析参考标记，并返回一个数据结构，告诉 `indentBlock` 下一步该做什么。有几个选项：

```
data  IndentOpt  m  a  b  =  IndentNone  a  -- ^ Parse no indented tokens, just return the value  |  IndentMany  (Maybe  Pos)  ([b]  ->  m  a)  (m  b)  -- ^ Parse many indented tokens (possibly zero), use given indentation
    -- level (if 'Nothing', use level of the first indented token); the
    -- second argument tells how to get the final result, and the third
    -- argument describes how to parse an indented token  |  IndentSome  (Maybe  Pos)  ([b]  ->  m  a)  (m  b)  -- ^ Just like 'IndentMany', but requires at least one indented token to
    -- be present 
```

我们可以改变主意并不解析缩进标记，我们可以解析 *多个*（也就是可能是零个）缩进标记或要求 *至少一个* 这样的标记。我们可以允许 `indentBlock` 检测第一个缩进标记的缩进级别并使用它，或者手动指定缩进级别。

### 解析一个简单的缩进列表

让我们解析一些项目的简单缩进列表。我们从导入部分开始：

```
{-# LANGUAGE OverloadedStrings #-}  {-# LANGUAGE TupleSections     #-}  module  Main  (main)  where  import  Control.Applicative  hiding  (some)  import  Control.Monad  (void)  import  Data.Text  (Text)  import  Data.Void  import  Text.Megaparsec  import  Text.Megaparsec.Char  import  qualified  Text.Megaparsec.Char.Lexer  as  L  type  Parser  =  Parsec  Void  Text 
```

我们将需要两种空间消耗者：一种是消耗换行符 `scn`，另一种是不消耗 `sc`（实际上只解析空格和制表符）：

```
lineComment  ::  Parser  ()  lineComment  =  L.skipLineComment  "#"  scn  ::  Parser  ()  scn  =  L.space  space1  lineComment  empty  sc  ::  Parser  ()  sc  =  L.space  (void  $  some  (char  ' '  <|>  char  '\t'))  lineComment  empty  lexeme  ::  Parser  a  ->  Parser  a  lexeme  =  L.lexeme  sc 
```

仅供娱乐，我们允许以 `#` 开头的行注释。

`pItemList` 是一个顶层表单，它本身是参考标记（列表标题）和缩进标记（列表项）的组合，所以：

```
pItemList  ::  Parser  (String,  [String])  -- header and list items  pItemList  =  L.nonIndented  scn  (L.indentBlock  scn  p)  where  p  =  do  header  <-  pItem  return  (L.IndentMany  Nothing  (return  .  (header,  ))  pItem) 
```

对于我们的目的，一个项目是由字母数字字符和破折号组成的序列：

```
pItem  ::  Parser  String  pItem  =  lexeme  (some  (alphaNumChar  <|>  char  '-'))  <?>  "list item" 
```

让我们将代码加载到 GHCi 中，并借助内置的 `parseTest` 进行尝试：

```
λ> parseTest (pItemList <* eof) ""
1:1:
  |
1 | <empty line>
  | ^
unexpected end of input
expecting list item

λ> parseTest (pItemList <* eof) "something"
("something",[])

λ> parseTest (pItemList <* eof) "  something"
1:3:
  |
1 |   something
  |   ^
incorrect indentation (got 3, should be equal to 1)

λ> parseTest (pItemList <* eof) "something\none\ntwo\nthree"
2:1:
  |
2 | one
  | ^
unexpected 'o'
expecting end of input 
```

请记住，我们使用了`IndentMany`选项，因此空列表是可以的，另一方面，内置组合器`space`已经从错误消息中隐藏了短语“期望更多空间”，因此这个错误消息是完全合理的。

让我们继续：

```
λ> parseTest (pItemList <* eof) "something\n  one\n    two\n  three"
3:5:
  |
3 |     two
  |     ^
incorrect indentation (got 5, should be equal to 3)

λ> parseTest (pItemList <* eof) "something\n  one\n  two\n three"
4:2:
  |
4 |  three
  |  ^
incorrect indentation (got 2, should be equal to 3)

λ> parseTest (pItemList <* eof) "something\n  one\n  two\n  three"
("something",["one","two","three"]) 
```

让我们用`IndentSome`替换`IndentMany`，用`Just (mkPos 5)`替换`Nothing`（缩进级别从1开始计算，因此在缩进项之前需要4个空格）：

```
pItemList  ::  Parser  (String,  [String])  pItemList  =  L.nonIndented  scn  (L.indentBlock  scn  p)  where  p  =  do  header  <-  pItem  return  (L.IndentSome  (Just  (mkPos  5))  (return  .  (header,  ))  pItem) 
```

现在：

```
λ> parseTest (pItemList <* eof) "something\n"
2:1:
  |
2 | <empty line>
  | ^
incorrect indentation (got 1, should be greater than 1)

λ> parseTest (pItemList <* eof) "something\n  one"
2:3:
  |
2 |   one
  |   ^
incorrect indentation (got 3, should be equal to 5)

λ> parseTest (pItemList <* eof) "something\n    one"
("something",["one"]) 
```

第一条消息可能有些令人惊讶，但是`megaparsec`知道列表中必须至少有一项，因此它检查缩进级别，发现是1，这是不正确的，因此报告了这一点。

### 嵌套缩进列表

让我们允许列表项具有子项。为此，我们将需要一个新的解析器，`pComplexItem`：

```
pComplexItem  ::  Parser  (String,  [String])  pComplexItem  =  L.indentBlock  scn  p  where  p  =  do  header  <-  pItem  return  (L.IndentMany  Nothing  (return  .  (header,  ))  pItem)  pItemList  ::  Parser  (String,  [(String,  [String])])  pItemList  =  L.nonIndented  scn  (L.indentBlock  scn  p)  where  p  =  do  header  <-  pItem  return  (L.IndentSome  Nothing  (return  .  (header,  ))  pComplexItem) 
```

如果我们输入这样的内容：

```
first-chapter
  paragraph-one
      note-A # an important note here!
      note-B
  paragraph-two
    note-1
    note-2
  paragraph-three 
```

到我们的解析器中，我们会得到：

```
Right
  ( "first-chapter"
  , [ ("paragraph-one",   ["note-A","note-B"])
    , ("paragraph-two",   ["note-1","note-2"])
    , ("paragraph-three", [])
    ]
  ) 
```

这说明了这种方法如何在不需要额外状态的情况下扩展到嵌套缩进结构。

### 添加行折叠

*行折叠*由几个元素组成，这些元素可以放在同一行上，也可以放在几行上，只要后续项的缩进级别大于第一项的缩进级别。

让我们使用另一个名为`lineFold`的辅助程序：

```
pComplexItem  ::  Parser  (String,  [String])  pComplexItem  =  L.indentBlock  scn  p  where  p  =  do  header  <-  pItem  return  (L.IndentMany  Nothing  (return  .  (header,  ))  pLineFold)  pLineFold  ::  Parser  String  pLineFold  =  L.lineFold  scn  $  \sc'  ->  let  ps  =  some  (alphaNumChar  <|>  char  '-')  `sepBy1`  try  sc'  in  unwords  <$>  ps  <*  scn  -- (1) 
```

`lineFold`的工作方式如下：我们给它一个接受换行符的空白消耗器`scn`，它返回一个特殊的空白消耗器`sc'`，我们可以在回调中使用它来消耗行折叠中元素之间的空白。

为什么在第一行上使用`try sc'`和`scn`？情况如下：

+   行折叠的组件只能比其开始时更缩进。

+   `sc'`使用换行符消耗了空白，这样在消耗空白后列号大于初始列号。

+   要停止，`sc'`应该遇到相反的情况，也就是说，消耗后的列号应该小于或等于初始列号。此时它在不消耗输入的情况下失败（由于`try`），并且在那个新事物开始的列之前使用`scn`来提取空白。

+   先前使用的`sc'`已经使用空格消耗器探测了带有换行符的空白。因此，在提取尾部空白时也消耗换行符是合理的。这就是为什么在第一行（1）上使用`scn`而不是`sc`的逻辑所在。

*练习：留给读者的练习是玩转我们解析器的最终版本。你可以创建由多个单词组成的“项”，只要它们进行行折叠，它们就会被解析并用单个空格连接在一起。*

## 编写高效的解析器

让我们讨论如何尝试提高`megaparsec`解析器的性能。需要立即指出的是，我们应该始终通过分析和基准测试来检查是否有改进。这是了解在调整性能时是否在正确的路径上的唯一方法。

常见的建议：

+   如果你的解析器使用的是Monad堆栈而不是普通的`Parsec`单子（请记住，它是`ParsecT`单子变换器在`Identity`上，这相当轻量），请确保你至少使用`transformers`库的0.5版本和`megaparsec`的7.0版本。这两个库在这些版本中有关键的性能改进，因此你可以免费获得更好的性能。

+   `Parsec`单子始终比基于`ParsecT`的单子变换器更快。除非绝对必要，否则避免使用`StateT`、`WriterT`和其他单子变换器。您添加到单子堆栈中的越多，解析器的速度就会越慢。

+   回溯是一种昂贵的操作。避免构建长链条的备选项，其中每个备选项在失败之前可能会深入输入。

+   除非确实有理由如此，否则不要保持解析器的多态性。最好为解析器的类型固定指定具体类型，例如每个顶层定义的`type Parser = Parsec Void Text`。这样GHC就能够更好地优化。

+   在合适的情况下要慷慨地进行内联。当您看到内联可以做多大的差异时，您可能会不相信自己的眼睛，尤其是对于短函数来说。对于在一个模块中定义并在另一个模块中使用的解析器来说，这一点尤为重要，因为`INLINE`和`INLINEABLE`指令使GHC将函数定义转储到接口文件中，从而便于专门化。

+   尽可能使用快速的原语，如`takeWhileP`、`takeWhile1P`和`takeP`。[这篇博文](https://markkarpov.com/post/megaparsec-more-speed-more-power#there-is-hope)解释了它们为什么如此快速。

+   避免使用`oneOf`和`noneOf`，优先使用`satisfy`和`anySingleBut`。

尽管上述大部分点不需要额外的评论，我认为养成使用更新的快速原语的习惯将会很有益处：`takeWhileP`、`takeWhile1P`和`takeP`。前两者尤为常见，因为它们允许我们替换基于`many`和`some`的构造，使它们更快，并将返回数据类型更改为输入流的块，即我们之前讨论过的`Tokens s`类型。

例如，回想一下当我们解析URI时，我们在解析权限组件中解析用户名时有这段代码：

```
 user  <-  T.pack  <$>  some  alphaNumChar 
```

我们可以用`takeWhile1P`来替换它：

```
 user  <-  takeWhile1P  (Just  "alpha num character")  isAlphaNum  --                  ^                            ^  --                  |                            |  -- label for tokens we match against         predicate 
```

当我们解析`ByteString`和`Text`时，这比原始方法快得多。还要注意，现在我们直接从`takeWhile1P`获取`Text`，因此不再需要`T.pack`。

这些方程可能有助于理解`takeWhileP`和`takeWhile1P`的`Maybe String`参数的含义：

```
takeWhileP  (Just  "foo")  f  =  many  (satisfy  f  <?>  "foo")  takeWhileP  Nothing  f  =  many  (satisfy  f)  takeWhile1P  (Just  "foo")  f  =  some  (satisfy  f  <?>  "foo")  takeWhile1P  Nothing  f  =  some  (satisfy  f) 
```

## 解析错误

现在我们已经探讨了如何使用`megaparsec`的大多数功能，是时候了解更多关于解析错误的知识：它们如何定义、如何信号化以及如何在运行中的解析器内部处理它们了。

### 解析错误定义

`ParseError`类型定义如下：

```
data  ParseError  s  e  =  TrivialError  Int  (Maybe  (ErrorItem  (Token  s)))  (Set  (ErrorItem  (Token  s)))  -- ^ Trivial errors, generated by Megaparsec's machinery. The data
    -- constructor includes the offset of error, unexpected token (if any),
    -- and expected tokens.  |  FancyError  Int  (Set  (ErrorFancy  e))  -- ^ Fancy, custom errors. 
```

在英语中：`ParseError` 是一个 `TrivialError`，最多包含一个意外项和一个（可能为空的）预期项集合，或者是一个 `FancyError`。

`ParseError s e` 是针对两个类型变量进行参数化的：

+   `s` 是输入流的类型。

+   `e` 是解析错误的自定义组件的类型。

`ErrorItem` 定义如下：

```
data  ErrorItem  t  =  Tokens  (NonEmpty  t)  -- ^ Non-empty stream of tokens  |  Label  (NonEmpty  Char)  -- ^ Label (cannot be empty)  |  EndOfInput  -- ^ End of input 
```

`NonEmpty` 是非空列表的类型，它来自于 `Data.List.NonEmpty`。这里还有 `ErrorFancy`：

```
data  ErrorFancy  e  =  ErrorFail  String  -- ^ 'fail' has been used in parser monad  |  ErrorIndentation  Ordering  Pos  Pos  -- ^ Incorrect indentation error: desired ordering between reference
    -- level and actual level, reference indentation level, actual
    -- indentation level  |  ErrorCustom  e  -- ^ Custom error data, can be conveniently disabled by indexing
    -- 'ErrorFancy' by 'Void' 
```

`ErrorFancy` 包括两种常见情况的数据构造函数，`megaparsec` 默认支持这些情况：

+   使用 `fail` 函数会导致解析器失败并报告任意的 `String`。

+   缩进相关的问题我们在前面的章节中已经见过。因为我们提供了用于处理缩进敏感语法的工具，我们需要一种方式来存储关于缩进问题的良好类型信息。

最后，`ErrorCustom` 是一种“扩展插槽”，允许将任意数据嵌入 `ErrorFancy` 类型中。当我们在解析错误中不需要任何自定义数据时，我们通过 `Void` 参数化 `ErrorFancy`。由于 `Void` 不包含任何非底部值，`ErrorCustom` 就被“抵消”，或者如果我们遵循代数数据类型和数字之间的类比，“乘以零”。

在库的旧版本中，`ParseError` 直接由 `parse` 等函数返回，但版本 7 延迟计算每个错误的行和列，以及在错误情况下获取输入中相关行的信息。这样做是为了加快解析速度，因为所有这些信息通常只在解析器失败时才有用。库的旧版本的另一个问题是，一次显示多个解析错误需要每次重新遍历输入以获取正确的行。

这个问题可以用 `ParseErrorBundle` 数据类型来解决：

```
-- | A non-empty collection of 'ParseError's equipped with 'PosState' that
-- allows to pretty-print the errors efficiently and correctly.  data  ParseErrorBundle  s  e  =  ParseErrorBundle  {  bundleErrors  ::  NonEmpty  (ParseError  s  e)  -- ^ A collection of 'ParseError's that is sorted by parse error offsets  ,  bundlePosState  ::  PosState  s  -- ^ State that is used for line\/column calculation  } 
```

所有运行解析器的函数都返回 `ParseErrorBundle`，其中 `bundlePosState` 被正确设置，并包含一组 `ParseError`。

### 如何信号解析错误

让我们讨论一下信号解析错误的不同方法。最简单的函数是 `fail`：

```
λ> parseTest (fail "I'm failing, help me!" :: Parser ()) ""
1:1:
  |
1 | <empty line>
  | ^
I'm failing, help me! 
```

对于许多熟悉简单解析库（如 `parsec`）的人来说，这通常已经足够了。然而，仅仅显示解析错误给用户并不是一切，我们可能需要分析和/或操作它。这就是 `String` 并不是非常方便的地方。

琐碎的解析错误通常由 `megaparsec` 生成，但我们可以使用 `failure` 组合子自行信号任何此类错误：

```
failure  ::  MonadParsec  e  s  m  =>  Maybe  (ErrorItem  (Token  s))  -- ^ Unexpected item (if any)  ->  Set  (ErrorItem  (Token  s))  -- ^ Expected items  ->  m  a 
```

```
unfortunateParser  ::  Parser  ()  unfortunateParser  =  failure  (Just  EndOfInput)  (Set.fromList  es)  where  es  =  [Tokens  (NE.fromList  "a"),  Tokens  (NE.fromList  "b")] 
```

```
λ> parseTest unfortunateParser ""
1:1:
  |
1 | <empty line>
  | ^
unexpected end of input
expecting 'a' or 'b' 
```

与基于 `fail` 的方法不同，琐碎的解析错误易于模式匹配、检查和修改。

对于复杂的错误，我们相应地有 `fancyFailure` 组合子：

```
fancyFailure  ::  MonadParsec  e  s  m  =>  Set  (ErrorFancy  e)  -- ^ Fancy error components  ->  m  a 
```

对于 `fancyFailure`，通常最好定义一个类似于我们在词法分析器模块中拥有的帮助器，而不是直接调用 `fancyFailure`：

```
incorrectIndent  ::  MonadParsec  e  s  m  =>  Ordering  -- ^ Desired ordering between reference level and actual level  ->  Pos  -- ^ Reference indentation level  ->  Pos  -- ^ Actual indentation level  ->  m  a  incorrectIndent  ord  ref  actual  =  fancyFailure  .  Set.singleton  $  ErrorIndentation  ord  ref  actual 
```

作为向您的解析器添加自定义解析错误组件的示例，让我们通过定义一个特殊的解析错误来说明，即给定的 `Text` 值不是关键字。

首先，我们需要定义数据类型，其中构造函数表示我们想要支持的场景：

```
data  Custom  =  NotKeyword  Text  deriving  (Eq,  Show,  Ord) 
```

并告诉`megaparsec`如何在解析错误中显示它：

```
instance  ShowErrorComponent  Custom  where  showErrorComponent  (NotKeyword  txt)  =  T.unpack  txt  ++  " is not a keyword" 
```

接下来我们更新我们的`Parser`类型同义词：

```
type  Parser  =  Parsec  Custom  Text 
```

之后，我们可以定义`notKeyword`辅助函数：

```
notKeyword  ::  Text  ->  Parser  a  notKeyword  =  customFailure  .  NotKeyword 
```

`customFailure`是一个有用的辅助函数，来自于`Text.Megaparsec`模块：

```
customFailure  ::  MonadParsec  e  s  m  =>  e  ->  m  a  customFailure  =  fancyFailure  .  E.singleton  .  ErrorCustom 
```

最后，让我们试一试：

```
λ> parseTest (notKeyword "foo" :: Parser ()) ""
1:1:
  |
1 | <empty line>
  | ^
foo is not a keyword 
```

### 显示解析错误

使用`errorBundlePretty`函数来显示`ParseErrorBundle`：

```
-- | Pretty-print a 'ParseErrorBundle'. All 'ParseError's in the bundle will
-- be pretty-printed in order together with the corresponding offending
-- lines by doing a single efficient pass over the input stream. The
-- rendered 'String' always ends with a newline.  errorBundlePretty  ::  (  VisualStream  s  ,  TraversableStream  s  ,  ShowErrorComponent  e  )  =>  ParseErrorBundle  s  e  -- ^ Parse error bundle to display  ->  String  -- ^ Textual rendition of the bundle 
```

在99%的情况下，你只需要这一个函数。

### 在运行解析器时捕获解析错误

`megaparsec`的另一个有用特性是可以“捕获”解析错误，以某种方式修改它，然后重新抛出，[就像异常一样](/tutorial/exceptions)。这是通过`observing`原语实现的：

```
-- | @'observing' p@ allows to “observe” failure of the @p@ parser, should
-- it happen, without actually ending parsing, but instead getting the
-- 'ParseError' in 'Left'. On success parsed value is returned in 'Right'
-- as usual. Note that this primitive just allows you to observe parse
-- errors as they happen, it does not backtrack or change how the @p@
-- parser works in any way.  observing  ::  MonadParsec  e  s  m  =>  m  a  -- ^ The parser to run  ->  m  (Either  (ParseError  (Token  s)  e)  a) 
```

这里是一个演示`observing`典型用法的完整程序：

```
{-# LANGUAGE OverloadedStrings #-}  {-# LANGUAGE TypeApplications  #-}  module  Main  (main)  where  import  Control.Applicative  hiding  (some)  import  Data.List  (intercalate)  import  Data.Set  (Set)  import  Data.Text  (Text)  import  Data.Void  import  Text.Megaparsec  import  Text.Megaparsec.Char  import  qualified  Data.Set  as  Set  data  Custom  =  TrivialWithLocation  [String]  -- position stack  (Maybe  (ErrorItem  Char))  (Set  (ErrorItem  Char))  |  FancyWithLocation  [String]  -- position stack  (ErrorFancy  Void)  -- Void, because we do not want to allow to nest Customs  deriving  (Eq,  Ord,  Show)  instance  ShowErrorComponent  Custom  where  showErrorComponent  (TrivialWithLocation  stack  us  es)  =  parseErrorTextPretty  (TrivialError  @Text  @Void  undefined  us  es)  ++  showPosStack  stack  showErrorComponent  (FancyWithLocation  stack  cs)  =  parseErrorTextPretty  (FancyError  @Text  @Void  undefined  (Set.singleton  cs))  ++  showPosStack  stack  showPosStack  ::  [String]  ->  String  showPosStack  =  intercalate  ", "  .  fmap  ("in "  ++)  type  Parser  =  Parsec  Custom  Text  inside  ::  String  ->  Parser  a  ->  Parser  a  inside  location  p  =  do  r  <-  observing  p  case  r  of  Left  (TrivialError  _  us  es)  ->  fancyFailure  .  Set.singleton  .  ErrorCustom  $  TrivialWithLocation  [location]  us  es  Left  (FancyError  _  xs)  ->  do  let  f  (ErrorFail  msg)  =  ErrorCustom  $  FancyWithLocation  [location]  (ErrorFail  msg)  f  (ErrorIndentation  ord  rlvl  alvl)  =  ErrorCustom  $  FancyWithLocation  [location]  (ErrorIndentation  ord  rlvl  alvl)  f  (ErrorCustom  (TrivialWithLocation  ps  us  es))  =  ErrorCustom  $  TrivialWithLocation  (location:ps)  us  es  f  (ErrorCustom  (FancyWithLocation  ps  cs))  =  ErrorCustom  $  FancyWithLocation  (location:ps)  cs  fancyFailure  (Set.map  f  xs)  Right  x  ->  return  x  myParser  ::  Parser  String  myParser  =  some  (char  'a')  *>  some  (char  'b')  main  ::  IO  ()  main  =  do  parseTest  (inside  "foo"  myParser)  "aaacc"  parseTest  (inside  "foo"  $  inside  "bar"  myParser)  "aaacc" 
```

*练习：详细理解这个程序如何工作。*

如果我运行这个程序，我会看到以下输出：

```
1:4:
  |
1 | aaacc
  |    ^
unexpected 'c'
expecting 'a' or 'b'
in foo
1:4:
  |
1 | aaacc
  |    ^
unexpected 'c'
expecting 'a' or 'b'
in foo, in bar 
```

因此，这个特性可以用来附加位置标签到解析错误中，或者确实定义*区域*，在这些区域中以某种方式处理解析错误。这种习惯用法非常有用，所以甚至有一个非原语的辅助函数叫做`region`，也是基于`observing`原语定义的：

```
-- | Specify how to process 'ParseError's that happen inside of this
-- wrapper. This applies to both normal and delayed 'ParseError's.
--
-- As a side-effect of the implementation the inner computation will start
-- with empty collection of delayed errors and they will be updated and
-- “restored” on the way out of 'region'.  region  ::  MonadParsec  e  s  m  =>  (ParseError  s  e  ->  ParseError  s  e)  -- ^ How to process 'ParseError's  ->  m  a  -- ^ The “region” that the processing applies to  ->  m  a  region  f  m  =  do  r  <-  observing  m  case  r  of  Left  err  ->  parseError  (f  err)  -- see the next section  Right  x  ->  return  x 
```

*练习：使用`region`重新编写上面程序中的`inside`函数。*

### 控制解析错误的位置

`region`的定义使用了`parseError`原语：

```
parseError  ::  MonadParsec  e  s  m  =>  ParseError  s  e  ->  m  a 
```

这是错误报告的基本原语，到目前为止我们看到的所有其他函数都是基于`parseError`定义的：

```
failure  ::  MonadParsec  e  s  m  =>  Maybe  (ErrorItem  (Token  s))  -- ^ Unexpected item (if any)  ->  Set  (ErrorItem  (Token  s))  -- ^ Expected items  ->  m  a  failure  us  ps  =  do  o  <-  getOffset  parseError  (TrivialError  o  us  ps)  fancyFailure  ::  MonadParsec  e  s  m  =>  Set  (ErrorFancy  e)  -- ^ Fancy error components  ->  m  a  fancyFailure  xs  =  do  o  <-  getOffset  parseError  (FancyError  o  xs) 
```

`parseError`允许你做的一件事是将错误偏移量（即位置）设置为输入流中当前位置之外的其他位置。让我们回到拒绝解析结果的示例：

```
withPredicate2  ::  (a  ->  Bool)  -- ^ The check to perform on parsed input  ->  String  -- ^ Message to print when the check fails  ->  Parser  a  -- ^ Parser to run  ->  Parser  a  -- ^ Resulting parser that performs the check  withPredicate2  f  msg  p  =  do  o  <-  getOffset  r  <-  p  if  f  r  then  return  r  else  do  setOffset  o  fail  msg 
```

我们注意到，`setOffset o`会使错误定位正确，但会作为副作用使解析器状态无效——偏移量将不再反映实际情况。这在更复杂的解析器中可能是一个真正的问题。例如，想象一下，你用`observing`包裹`withPredicate2`，这样在`fail`后会运行一些代码。

使用`parseError`和`region`，我们最终得到了解决问题的合适方案——要么使用`region`重置解析错误位置，要么一开始就使用`parseError`：

```
withPredicate3  ::  (a  ->  Bool)  -- ^ The check to perform on parsed input  ->  String  -- ^ Message to print when the check fails  ->  Parser  a  -- ^ Parser to run  ->  Parser  a  -- ^ Resulting parser that performs the check  withPredicate3  f  msg  p  =  do  o  <-  getOffset  r  <-  p  if  f  r  then  return  r  else  region  (setErrorOffset  o)  (fail  msg)  withPredicate4  ::  (a  ->  Bool)  -- ^ The check to perform on parsed input  ->  String  -- ^ Message to print when the check fails  ->  Parser  a  -- ^ Parser to run  ->  Parser  a  -- ^ Resulting parser that performs the check  withPredicate4  f  msg  p  =  do  o  <-  getOffset  r  <-  p  if  f  r  then  return  r  else  parseError  (FancyError  o  (Set.singleton  (ErrorFail  msg))) 
```

### 报告多个解析错误

最后，`megaparsec`允许我们在单次运行中发出多个解析错误信号。这对最终用户可能有所帮助，因为他们将能够一次性修复多个问题，从而减少运行解析器的次数。

具有多错误解析器的一个先决条件是应该能够跳过输入的有问题部分，并从已知良好的位置恢复解析。这一部分通过使用`withRecovery`原语实现：

```
-- | @'withRecovery' r p@ allows continue parsing even if parser @p@
-- fails. In this case @r@ is called with the actual 'ParseError' as its
-- argument. Typical usage is to return a value signifying failure to
-- parse this particular object and to consume some part of the input up
-- to the point where the next object starts.
--
-- Note that if @r@ fails, original error message is reported as if
-- without 'withRecovery'. In no way recovering parser @r@ can influence
-- error messages.  withRecovery  ::  (ParseError  s  e  ->  m  a)  -- ^ How to recover from failure  ->  m  a  -- ^ Original parser  ->  m  a  -- ^ Parser that can recover from failures 
```

在Megaparsec 8之前，用户必须选择类型`a`为包括成功和失败可能性的总和类型。例如，它可以是`Either (ParseError s e) Result`。必须手动收集解析错误，然后在显示之前将其添加到`ParseErrorBundle`中。不用说，所有这些都是高级用法的例子，对用户不友好。

Megaparsec 8支持*延迟解析错误*：

```
-- | Register a 'ParseError' for later reporting. This action does not end
-- parsing and has no effect except for adding the given 'ParseError' to the
-- collection of “delayed” 'ParseError's which will be taken into
-- consideration at the end of parsing. Only if this collection is empty
-- parser will succeed. This is the main way to report several parse errors
-- at once.  registerParseError  ::  MonadParsec  e  s  m  =>  ParseError  s  e  ->  m  ()  -- | Like 'failure', but for delayed 'ParseError's.  registerFailure  ::  MonadParsec  e  s  m  =>  Maybe  (ErrorItem  (Token  s))  -- ^ Unexpected item (if any)  ->  Set  (ErrorItem  (Token  s))  -- ^ Expected items  ->  m  ()  -- | Like 'fancyFailure', but for delayed 'ParseError's.  registerFancyFailure  ::  MonadParsec  e  s  m  =>  Set  (ErrorFancy  e)  -- ^ Fancy error components  ->  m  () 
```

这些错误可以在`withRecovery`的错误处理回调中注册，使得最终的类型为`Maybe Result`。这样可以处理包含延迟错误的最终`ParseErrorBundle`，并且在延迟错误集合不为空时使解析器最终失败。

希望用户能更普遍地使用编写多错误解析器的实践。

## 测试Megaparsec解析器

测试解析器是大多数人迟早会面对的实际任务，因此我们有责任覆盖它。测试`megaparsec`解析器的推荐方法是使用[`hspec-megaparsec`](https://hackage.haskell.org/package/hspec-megaparsec)包。该包添加了与`hspec`测试框架配合使用的实用期望，如`shouldParse`、`parseSatisfies`等。

让我们从一个例子开始：

```
{-# LANGUAGE OverloadedStrings #-}  module  Main  (main)  where  import  Control.Applicative  hiding  (some)  import  Data.Text  (Text)  import  Data.Void  import  Test.Hspec  import  Test.Hspec.Megaparsec  import  Text.Megaparsec  import  Text.Megaparsec.Char  type  Parser  =  Parsec  Void  Text  myParser  ::  Parser  String  myParser  =  some  (char  'a')  main  ::  IO  ()  main  =  hspec  $  describe  "myParser"  $  do  it  "returns correct result"  $  parse  myParser  ""  "aaa"  `shouldParse`  "aaa"  it  "result of parsing satisfies what it should"  $  parse  myParser  ""  "aaaa"  `parseSatisfies`  ((==  4)  .  length) 
```

`shouldParse`接受`Either (ParseErrorBundle s e) a`，即解析的结果和要比较的类型`a`。这可能是最常见的辅助函数。`parseSatisfies`非常类似，但不是通过预期结果的相等性来检查结果，而是允许通过应用任意谓词来检查结果。

其他简单的期望包括`shouldSucceedOn`和`shouldFailOn`（虽然它们很少使用）：

```
 it  "should parse 'a's all right"  $  parse  myParser  ""  `shouldSucceedOn`  "aaaa"  it  "should fail on 'b's"  $  parse  myParser  ""  `shouldFailOn`  "bbb" 
```

使用`megaparsec`时，我们希望准确地处理解析器产生的解析错误。用于测试解析错误的方法是`shouldFailWith`，可以像这样使用：

```
 it  "fails on 'b's producing correct error message"  $  parse  myParser  ""  "bbb"  `shouldFailWith`  TrivialError  0  (Just  (Tokens  ('b'  :|  [])))  (Set.singleton  (Tokens  ('a'  :|  []))) 
```

像这样写出`TrivialError`是令人厌烦的。`ParseError`的定义包含“不方便”的类型，如`Set`和`NonEmpty`，直接输入它们并不方便，正如我们刚刚看到的。幸运的是，`Test.Hspec.Megaparsec`还重新导出了`Text.Megaparsec.Error.Builder`模块，该模块提供了一个API，用于更轻松地构造`ParserError`。让我们改用`err`助手：

```
 it  "fails on 'b's producing correct error message"  $  parse  myParser  ""  "bbb"  `shouldFailWith`  err  0  (utok  'b'  <>  etok  'a') 
```

+   `err`的第一个参数是解析错误的偏移量（在获取错误之前消耗的标记数）。在这个例子中，它简单地是0。

+   `utok`代表“意外的标记”，类似地，`etok`表示“期望的标记”。

*练习：熟悉`errFancy`，它用于构建精美的解析错误。*

最后，可以使用`failsLeaving`和`succeedsLeaving`测试解析后剩余的输入部分：

```
 it  "consumes all 'a's but does not touch 'b's"  $  runParser'  myParser  (initialState  "aaabbb")  `succeedsLeaving`  "bbb"  it  "fails without consuming anything"  $  runParser'  myParser  (initialState  "bbbccc")  `failsLeaving`  "bbbccc" 
```

这些应该与`runParser'`或`runParserT'`一起使用，它们接受解析器的自定义初始状态并返回其最终状态（这就是允许我们在解析后检查输入流剩余部分的方法）：

```
runParser'  ::  Parsec  e  s  a  -- ^ Parser to run  ->  State  s  -- ^ Initial state  ->  (State  s,  Either  (ParseError  (Token  s)  e)  a)  runParserT'  ::  Monad  m  =>  ParsecT  e  s  m  a  -- ^ Parser to run  ->  State  s  -- ^ Initial state  ->  m  (State  s,  Either  (ParseError  (Token  s)  e)  a) 
```

`initialState` 函数接受输入流并返回带有该输入流及其他记录字段填充为其默认值的初始状态。

使用 `hspec-megaparsec` 的其他启发源包括：

## 使用自定义输入流

`megaparsec` 可以用来解析任何是 `Stream` 类型类实例的输入。这意味着它可以与词法分析工具（如 `alex`）结合使用。

为了不偏离我们的主题，通过展示如何使用 `alex` 生成令牌流，我们将在以下形式中假设它。

```
{-# LANGUAGE LambdaCase        #-}  {-# LANGUAGE OverloadedStrings #-}  {-# LANGUAGE RecordWildCards   #-}  {-# LANGUAGE TypeFamilies      #-}  module  Main  (main)  where  import  Data.List.NonEmpty  (NonEmpty  (..))  import  Data.Proxy  import  Data.Void  import  Text.Megaparsec  import  qualified  Data.List  as  DL  import  qualified  Data.List.NonEmpty  as  NE  import  qualified  Data.Set  as  Set  data  MyToken  =  Int  Int  |  Plus  |  Mul  |  Div  |  OpenParen  |  CloseParen  deriving  (Eq,  Ord,  Show) 
```

虽然报告解析错误，我们需要一种方法来知道标记的起始位置、结束位置和长度，所以让我们添加 `WithPos`：

```
data  WithPos  a  =  WithPos  {  startPos  ::  SourcePos  ,  endPos  ::  SourcePos  ,  tokenLength  ::  Int  ,  tokenVal  ::  a  }  deriving  (Eq,  Ord,  Show) 
```

然后我们可以为我们的流定义一个数据类型：

```
data  MyStream  =  MyStream  {  myStreamInput  ::  String  -- for showing offending lines  ,  unMyStream  ::  [WithPos  MyToken]  } 
```

接下来，我们需要使 `MyStream` 成为 `Stream` 类型类的一个实例。这需要 `TypeFamilies` 语言扩展，因为我们想要定义关联类型函数 `Token` 和 `Tokens`：

```
instance  Stream  MyStream  where  type  Token  MyStream  =  WithPos  MyToken  type  Tokens  MyStream  =  [WithPos  MyToken]  -- … 
```

`Stream`、`VisualStream` 和 `TraversableStream` 在 `Text.Megaparsec.Stream` 模块中有文档记录。在这里，我们直接定义这些方法：

```
instance  Stream  MyStream  where  type  Token  MyStream  =  WithPos  MyToken  type  Tokens  MyStream  =  [WithPos  MyToken]  tokenToChunk  Proxy  x  =  [x]  tokensToChunk  Proxy  xs  =  xs  chunkToTokens  Proxy  =  id  chunkLength  Proxy  =  length  chunkEmpty  Proxy  =  null  take1_  (MyStream  _  [])  =  Nothing  take1_  (MyStream  str  (t:ts))  =  Just  (  t  ,  MyStream  (drop  (tokensLength  pxy  (t:|[]))  str)  ts  )  takeN_  n  (MyStream  str  s)  |  n  <=  0  =  Just  ([],  MyStream  str  s)  |  null  s  =  Nothing  |  otherwise  =  let  (x,  s')  =  splitAt  n  s  in  case  NE.nonEmpty  x  of  Nothing  ->  Just  (x,  MyStream  str  s')  Just  nex  ->  Just  (x,  MyStream  (drop  (tokensLength  pxy  nex)  str)  s')  takeWhile_  f  (MyStream  str  s)  =  let  (x,  s')  =  DL.span  f  s  in  case  NE.nonEmpty  x  of  Nothing  ->  (x,  MyStream  str  s')  Just  nex  ->  (x,  MyStream  (drop  (tokensLength  pxy  nex)  str)  s')  instance  VisualStream  MyStream  where  showTokens  Proxy  =  DL.intercalate  " "  .  NE.toList  .  fmap  (showMyToken  .  tokenVal)  tokensLength  Proxy  xs  =  sum  (tokenLength  <$>  xs)  instance  TraversableStream  MyStream  where  reachOffset  o  PosState  {..}  =  (  Just  (prefix  ++  restOfLine)  ,  PosState  {  pstateInput  =  MyStream  {  myStreamInput  =  postStr  ,  unMyStream  =  post  }  ,  pstateOffset  =  max  pstateOffset  o  ,  pstateSourcePos  =  newSourcePos  ,  pstateTabWidth  =  pstateTabWidth  ,  pstateLinePrefix  =  prefix  }  )  where  prefix  =  if  sameLine  then  pstateLinePrefix  ++  preLine  else  preLine  sameLine  =  sourceLine  newSourcePos  ==  sourceLine  pstateSourcePos  newSourcePos  =  case  post  of  []  ->  case  unMyStream  pstateInput  of  []  ->  pstateSourcePos  xs  ->  endPos  (last  xs)  (x:_)  ->  startPos  x  (pre,  post)  =  splitAt  (o  -  pstateOffset)  (unMyStream  pstateInput)  (preStr,  postStr)  =  splitAt  tokensConsumed  (myStreamInput  pstateInput)  preLine  =  reverse  .  takeWhile  (/=  '\n')  .  reverse  $  preStr  tokensConsumed  =  case  NE.nonEmpty  pre  of  Nothing  ->  0  Just  nePre  ->  tokensLength  pxy  nePre  restOfLine  =  takeWhile  (/=  '\n')  postStr  pxy  ::  Proxy  MyStream  pxy  =  Proxy  showMyToken  ::  MyToken  ->  String  showMyToken  =  \case  (Int  n)  ->  show  n  Plus  ->  "+"  Mul  ->  "*"  Div  ->  "/"  OpenParen  ->  "("  CloseParen  ->  ")" 
```

`Stream` 类型类的更多背景信息（以及它为什么是这个样子）可以在[这篇博文](https://markkarpov.com/post/megaparsec-more-speed-more-power)中找到。请注意，在 `megaparsec` 的第 9 版中，`Stream` 的某些方法已移动到 `VisualStream` 和 `TraversableStream` 类中，以便更轻松地为某些自定义输入流定义 `Stream` 的实例。

现在我们可以为我们的自定义流定义 `Parser`：

```
type  Parser  =  Parsec  Void  MyStream 
```

下一步是在 `token` 和（如果有意义的话）`tokens` 原语的基础上定义基本解析器。对于支持的即插即用流，我们有 `Text.Megaparsec.Byte` 和 `Text.Megaparsec.Char` 模块，但如果我们要处理自定义标记，就需要自定义辅助程序。

```
liftMyToken  ::  MyToken  ->  WithPos  MyToken  liftMyToken  myToken  =  WithPos  pos  pos  0  myToken  where  pos  =  initialPos  ""  pToken  ::  MyToken  ->  Parser  MyToken  pToken  c  =  token  test  (Set.singleton  .  Tokens  .  nes  .  liftMyToken  $  c)  where  test  (WithPos  _  _  _  x)  =  if  x  ==  c  then  Just  x  else  Nothing  nes  x  =  x  :|  []  pInt  ::  Parser  Int  pInt  =  token  test  Set.empty  <?>  "integer"  where  test  (WithPos  _  _  _  (Int  n))  =  Just  n  test  _  =  Nothing 
```

最后，让我们来看一个解析求和的测试解析器：

```
pSum  ::  Parser  (Int,  Int)  pSum  =  do  a  <-  pInt  _  <-  pToken  Plus  b  <-  pInt  return  (a,  b) 
```

以及它的一个示例输入：

```
exampleStream  ::  MyStream  exampleStream  =  MyStream  "5 + 6"  [  at  1  1  (Int  5)  ,  at  1  3  Plus  -- (1)  ,  at  1  5  (Int  6)  ]  where  at  l  c  =  WithPos  (at'  l  c)  (at'  l  (c  +  1))  2  at'  l  c  =  SourcePos  ""  (mkPos  l)  (mkPos  c) 
```

让我们试一下：

```
λ> parseTest (pSum <* eof) exampleStream
(5,6) 
```

如果我们将第 (1) 行上的 `Plus` 更改为 `Div`，我们将得到正确的解析错误：

```
λ> parseTest (pSum <* eof) exampleStream
1:3:
  |
1 | 5 + 6
  |   ^^
unexpected /
expecting + 
```

换句话说，现在我们有一个完全功能的解析器，可以解析自定义流。

* * *

1.  实际上，[`modern-uri`](https://hackage.haskell.org/package/modern-uri) 包含一个真实的 Megaparsec 解析器，可以按照 RFC 3986 解析 URI。尽管包中的解析器比我们这里描述的要复杂得多。 [↩](#fnref1)
