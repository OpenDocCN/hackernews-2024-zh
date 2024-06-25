<!--yml

category: 未分类

date: 2024-05-27 14:57:55

-->

# Pratt 解析器：轻松解析表达式 – journal.stuffwithstuff.com

> 来源：[`journal.stuffwithstuff.com/2011/03/19/pratt-parsers-expression-parsing-made-easy/`](https://journal.stuffwithstuff.com/2011/03/19/pratt-parsers-expression-parsing-made-easy/)

时不时地，我会偶然发现一些算法或思想，它们非常聪明，对于解决问题来说是如此完美的解决方案，以至于我觉得通过学习它们我变得更聪明或获得了 [一种新的超能力](http://xkcd.com/208/)。堆就是其中之一，几乎是我在截断的计算机科学教育中得到的唯一东西。我最近又发现了另一种：Pratt 或“自顶向下运算符优先级”解析器。

当你编写一个解析器时，[递归下降](http://en.wikipedia.org/wiki/Recursive_descent) 就像涂抹花生酱一样简单。当你可以根据你正在查看的代码的下一部分来确定解析什么时，它就会表现出色。这通常适用于语言语法的声明和语句级别，因为大多数语法从关键字开始——`class`、`if`、`for`、`while` 等等。

解析在处理表达式时变得更加棘手。当涉及到中缀运算符如`+`，后缀运算符如`++`，甚至是混合表达式如`?:`时，直到解析到一半时才能确定正在解析的表达式是什么类型的，这可能会很困难。你 *可以* 用递归下降来实现这一点，但这很繁琐。你必须为每个优先级水平编写单独的函数（例如 JavaScript 有 17 个优先级），手动处理结合性，并且将你的语法分散到一堆解析代码中，直到变得难以看清楚。

## 花生酱和果冻，秘密武器

Pratt 解析解决了这个问题。如果递归下降是花生酱，那么 Pratt 解析就是果冻。当你把两者结合在一起时，你得到了一个简单、简洁、易读的解析器，可以处理任何你投入其中的语法。

Pratt 处理运算符优先级和中缀表达式的技术是如此简单和有效，以至于几乎没有人知道它是为什么。在七十年代之后，自顶向下的运算符优先级解析器似乎已经消失了。Douglas Crockford 的 [JSLint](http://www.jslint.com/) 使用它来 [解析 JavaScript](http://crockford.com/javascript/tdop/tdop.html)，但他的处理方式是 [极少数](http://effbot.org/zone/simple-top-down-parsing.htm) 关于此的现代文章之一。

我认为问题的一部分在于，Pratt 的术语是晦涩的，而 Crockford 的文章本身也相当混沌。Pratt 使用诸如“null denominator”之类的术语，而 Crockford 则混入了额外的东西，比如跟踪词法作用域，这使得核心思想变得模糊。

这就是我的所在之处。我不会做出任何革命性的事情。我只会尝试清楚地阐述自顶向下运算符优先级解析器背后的核心概念。我会更换一些术语以（我希望）澄清事物。希望我不会冒犯任何人的纯粹主义情感。我会用 Java 编写代码，这是编程语言中的俚语拉丁文。我想，如果你能用 Java 写出来，你就能用任何语言写出来。

## 我们将制作什么

我是一个通过实践学习的人，这意味着我也是一个通过实践教学的人。所以为了展示 Pratt 解析器的工作原理，我们将为一个[名为 *Bantam* 的微型玩具语言构建解析器](https://github.com/munificent/bantam)。该语言只有表达式，因为那是 Pratt 解析真正有用的地方，但这应该足以让你相信它的有用性。

即使 Bantam 简单，它也有一整套运算符：前缀 (`+`, `-`, `~`, `!`)，后缀 (`!`)，中缀 (`+`, `-`, `*`, `/`, `^`)，甚至还有混合条件运算符 (`?:`)。它有多个优先级级别和右结合和左结合运算符。它还有赋值，函数调用和用于分组的括号。如果我们能解析这个，我们就能解析任何东西。

## 我们将从什么开始

我们只关心解析，所以我们会忽略标记化阶段。我拼凑了一个[简陋的词法分析器](https://github.com/munificent/bantam/blob/master/src/com/stuffwithstuff/bantam/Lexer.java)，它可以工作，我们假装令牌就像从天上下来一样。

令牌是最小的有意义代码块。它有一个与之关联的类型和字符串。给定 `from + offset(time)`，令牌将是：

```
NAME "from"
PLUS "+"
NAME "offset"
LEFT_PAREN "("
NAME "time"
RIGHT_PAREN ")" 
```

对于这个练习，我们不会*解释*或*编译*这段代码。我们只想将其解析为一个漂亮的数据结构。对于我们的目的，这意味着我们的解析器应该吃掉一堆[Token](https://github.com/munificent/bantam/blob/master/src/com/stuffwithstuff/bantam/Token.java)对象，并输出一个实现[Expression](https://github.com/munificent/bantam/blob/master/src/com/stuffwithstuff/bantam/expressions/Expression.java)的某个类的实例。为了给你一个想法，这里是[条件表达式](https://github.com/munificent/bantam/blob/master/src/com/stuffwithstuff/bantam/expressions/ConditionalExpression.java)类的简化版本：

```
class ConditionalExpression implements Expression {
  public ConditionalExpression(
      Expression condition,
      Expression thenArm,
      Expression elseArm) {
    this.condition = condition;
    this.thenArm   = thenArm;
    this.elseArm   = elseArm;
  }

  public final Expression condition;
  public final Expression thenArm;
  public final Expression elseArm;
} 
```

（你得爱上 Java 的“请四次签名”级别的官僚主义。就像我说的，如果你能容忍 Java 中的这种情况，它在*任何*语言中都可以运行。）

我们将从一个简单的[解析器](https://github.com/munificent/bantam/blob/master/src/com/stuffwithstuff/bantam/Parser.java)类开始。解析器拥有令牌流，处理前瞻并提供你编写自顶向下递归下降解析器所需的基本方法（它是[LL(1)](https://en.wikipedia.org/wiki/LL_parser)）。这足以让我们开始。如果以后需要更多，扩展它很容易。

好的，让我们建立一个解析器吧！

## 先说重要的事情

尽管“完整”的 Pratt 解析器非常小，但我发现它很难解读。有点像[快速排序](http://en.wikipedia.org/wiki/Quicksort)，实现是一小撮深度交织的代码，看起来简单但实则错综复杂。为了解开这一困惑，我们将一步一步地构建它。

最简单的表达式解析是前缀操作符和单令牌表达式。对于这些，当前标记告诉我们我们需要做什么。Bantam 只有一个单令牌表达式：命名变量。它有四个前缀操作符：`+`、`-`、`~`和`!`。解析这些的最简单代码是：

```
Expression parseExpression() {
  if (match(TokenType.NAME))       // Return NameExpression...
  else if (match(TokenType.PLUS))  // Return prefix + operator...
  else if (match(TokenType.MINUS)) // Return prefix - operator...
  else if (match(TokenType.TILDE)) // Return prefix ~ operator...
  else if (match(TokenType.BANG))  // Return prefix ! operator...
  else throw new ParseException();
} 
```

但这有点庞大。正如你所看到的，我们正在从 TokenType 转换为不同的解析行为。让我们直接将其编码，通过将 TokenType 映射到解析代码块的 Map。我们将这些块称为“解析器”，它们将实现这个：

```
interface PrefixParselet {
  Expression parse(Parser parser, Token token);
} 
```

解析变量名的解析器实现简单地是：

```
class NameParselet implements PrefixParselet {
  public Expression parse(Parser parser, Token token) {
    return new NameExpression(token.getText());
  }
} 
```

我们可以使用单个类来表示所有前缀操作符，因为它们在实际的操作符令牌本身上不同：

```
class PrefixOperatorParselet implements PrefixParselet {
  public Expression parse(Parser parser, Token token) {
    Expression operand = parser.parseExpression();
    return new PrefixExpression(token.getType(), operand);
  }
} 
```

你会注意到它回调到`parseExpression()`中以解析运算符之后出现的操作数（例如，解析`-a`中的`a`）。这种递归处理嵌套的运算符，如`-+~!a`。

回到解析器，链式`if`语句被替换为一个映射：

```
class Parser {
  public Expression parseExpression() {
    Token token = consume();
    PrefixParselet prefix = mPrefixParselets.get(token.getType());

    if (prefix == null) throw new ParseException(
        "Could not parse \"" + token.getText() + "\".");

    return prefix.parse(this, token);
  }

  // Other stuff...

  private final Map<TokenType, PrefixParselet> mPrefixParselets =
      new HashMap<TokenType, PrefixParselet>();
} 
```

为了定义到目前为止的语法——变量和四个前缀运算符——我们将添加这些辅助方法：

```
public void register(TokenType token, PrefixParselet parselet) {
  mPrefixParselets.put(token, parselet);
}

public void prefix(TokenType token) {
  register(token, new PrefixOperatorParselet());
} 
```

现在我们可以像这样定义语法：

```
register(TokenType.NAME, new NameParselet());
prefix(TokenType.PLUS);
prefix(TokenType.MINUS);
prefix(TokenType.TILDE);
prefix(TokenType.BANG); 
```

这已经比递归下降解析器有所改进，因为我们的语法现在更具声明性，而不是分散在几个命令式函数中，而且我们可以在一个地方看到实际的语法。更好的是，我们可以通过注册新的解析器来扩展语法。我们不必更改解析器类本身。

如果我们*只有*前缀表达式，现在我们就已经完成了。然而，我们没有。

## 卡在中间了

目前我们所拥有的仅在*第一个*标记告诉我们我们正在解析什么类型的表达式时才有效，但情况并非总是如此。对于表达式`a + b`，我们直到解析完`a`并到达`+`之后才知道我们有一个加法表达式。我们必须扩展解析器以支持这一点。

幸运的是，我们现在有能力这样做。我们当前的`parseExpression()`方法解析一个完整的前缀表达式，包括任何嵌套的前缀表达式，然后停止。因此，如果我们对其进行解析：

它将解析`-a`并让我们停在`+`上。这正是告诉我们需要解析什么中缀表达式的标记。与前缀解析相比，中缀解析的唯一变化是在中缀运算符之前有另一个表达式，该表达式作为参数传递给中缀解析器。让我们定义一个支持此的解析器：

```
interface InfixParselet {
  Expression parse(Parser parser, Expression left, Token token);
} 
```

唯一的区别在于`left`参数，它是我们在到达中缀标记之前解析的表达式。我们通过另一个中缀解析器表将其连接到我们的解析器，通过将中缀解析器作为参数传递给它的解析方法。

为前缀和中缀表达式分别设置单独的表格很重要，因为有时我们对于同一 TokenType 既有前缀解析器又有中缀解析器。例如，`(`的前缀解析器处理像`a * (b + c)`这样的表达式中的分组。同时，`(`的*中缀*解析器处理函数调用，如`a(b)`。

现在，在我们解析前缀表达式之后，我们查找匹配下一个令牌并将前缀表达式包装为操作数的中缀解析器：

```
class Parser {
  public void register(TokenType token, InfixParselet parselet) {
    mInfixParselets.put(token, parselet);
  }

  public Expression parseExpression() {
    Token token = consume();
    PrefixParselet prefix = mPrefixParselets.get(token.getType());

    if (prefix == null) throw new ParseException(
        "Could not parse \"" + token.getText() + "\".");

    Expression left = prefix.parse(this, token);

    token = lookAhead(0);
    InfixParselet infix = mInfixParselets.get(token.getType());

    // No infix expression at this point, so we're done.
    if (infix == null) return left;

    consume();
    return infix.parse(this, left, token);
  }

  // Other stuff...

  private final Map<TokenType, InfixParselet> mInfixParselets =
      new HashMap<TokenType, InfixParselet>();
} 
```

相当直接。我们可以这样为二元算术运算符`+`实现一个中缀解析器：

```
class BinaryOperatorParselet implements InfixParselet {
  public Expression parse(Parser parser,
      Expression left, Token token) {
    Expression right = parser.parseExpression();
    return new OperatorExpression(left, token.getType(), right);
  }
} 
```

中缀解析器也适用于后缀运算符。我称它们为“中缀”，但它们实际上是“除前缀之外的任何东西”。如果在令牌之前有一些前导子表达式，那么该令牌将由中缀解析器处理。这包括后缀表达式和像`?:`这样的混合式表达式。

后缀表达式与单令牌前缀解析器一样简单：它们只是取`left`操作数，并将其包装在另一个表达式中：

```
class PostfixOperatorParselet implements InfixParselet {
  public Expression parse(Parser parser, Expression left,
      Token token) {
    return new PostfixExpression(left, token.getType());
  }
} 
```

Mixfix 也很简单。它类似于递归下降：

```
class ConditionalParselet implements InfixParselet {
  public Expression parse(Parser parser, Expression left,
      Token token) {
    Expression thenArm = parser.parseExpression();
    parser.consume(TokenType.COLON);
    Expression elseArm = parser.parseExpression();

    return new ConditionalExpression(left, thenArm, elseArm);
  }
} 
```

现在我们可以解析前缀、后缀、中缀甚至混合式表达式。凭借相当少量的代码，我们可以解析像`a + (b ? c! : -d)`这样的复杂嵌套表达式。我们完成了，对吧？嗯... 差不多了。

## 劳驾你了，阿姨莎莉

我们的解析器*可以*解析所有这些东西，但它没有以正确的优先级或结合性进行解析。如果你将`a - b - c`传递给解析器，它将解析嵌套表达式，如`a - (b - c)`，这是不正确的。（嗯，实际上它是*正确的*——是结合性的。我们需要它是*左*的。）

而这个*最后*一步，我们解决了这个问题，这就是 Pratt 解析器从相当好到完全激进的地方。我们会做两个简单的更改。我们扩展`parseExpression()`以接受*优先级*——一个数字，表示哪些表达式可以通过该调用进行解析。如果`parseExpression()`遇到一个优先级低于允许的表达式，它将停止解析并返回到目前为止所拥有的内容。

为了进行该检查，我们需要知道任何给定中缀表达式的优先级。我们让解析器指定它：

```
public interface InfixParselet {
  Expression parse(Parser parser, Expression left, Token token);
  int getPrecedence();
} 
```

使用这个，我们的核心表达式解析器看起来像这样：

```
public Expression parseExpression(int precedence) {
  Token token = consume();
  PrefixParselet prefix = mPrefixParselets.get(token.getType());

  if (prefix == null) throw new ParseException(
      "Could not parse \"" + token.getText() + "\".");

  Expression left = prefix.parse(this, token);

  while (precedence < getPrecedence()) {
    token = consume();

    InfixParselet infix = mInfixParselets.get(token.getType());
    left = infix.parse(this, left, token);
  }

  return left;
} 
```

那依赖于一个微小的辅助函数，以获取当前令牌的优先级，或者如果没有中缀解析器，则获取默认值：

```
private int getPrecedence() {
  InfixParselet parser = mInfixParselets.get(
      lookAhead(0).getType());
  if (parser != null) return parser.getPrecedence();

  return 0;
} 
```

就是这样。为了将优先级编入 Bantam 的语法中，我们设置了一个小的优先级表：

```
public class Precedence {
  public static final int ASSIGNMENT  = 1;
  public static final int CONDITIONAL = 2;
  public static final int SUM         = 3;
  public static final int PRODUCT     = 4;
  public static final int EXPONENT    = 5;
  public static final int PREFIX      = 6;
  public static final int POSTFIX     = 7;
  public static final int CALL        = 8;
} 
```

为了使我们的运算符以正确的优先级解析其操作数，它们在递归调用`parseExpression()`时会将适当的值传递回去。例如，处理`+`运算符的[BinaryOperatorParselet](https://github.com/munificent/bantam/blob/master/src/com/stuffwithstuff/bantam/parselets/BinaryOperatorParselet.java)实例在解析其右操作数时传递`Precedence.SUM`。

结合性也很容易。如果中缀解析器在其自身的`getPrecedence()`调用中返回相同的优先级，并使用`parseExpression()`进行调用，你将得到左结合性。要成为右结合，它只需要传入比那个优先级小*一个*。

## 前进并繁殖

我已经使用这种方法重写了[Magpie 的解析器](https://github.com/munificent/magpie/blob/master/src/com/stuffwithstuff/magpie/parser/MagpieParser.java)，它运行得非常完美。我还正在使用这种技术开发一个 JavaScript 解析器，同样效果很好。

我认为 Pratt 解析器应该是简单的、简洁的、可扩展的（例如，Magpie 就使用这个特性在运行时让你扩展自己的语法），并且易于阅读。我已经到了无法想象以其他方式编写解析器的地步。我从未想过会说出这样的话，但是现在解析器感觉很容易。

想要亲自见识，只需查看[完整的程序](https://github.com/munificent/bantam)。
