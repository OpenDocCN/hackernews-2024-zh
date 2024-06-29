<!--yml

category: 未分类

date: 2024-05-29 12:32:10

-->

# Jonas Hietala：让我们创建一个Tree-sitter语法

> 来源：[https://www.jonashietala.se/blog/2024/03/19/lets_create_a_tree-sitter_grammar/](https://www.jonashietala.se/blog/2024/03/19/lets_create_a_tree-sitter_grammar/)

Tree-sitter解析器实际上是一个C程序。我们在JavaScript中看到的语法只是用作描述以生成C中的解析器。如果你是个虐待狂，你可以在运行`tree-sitter generate`后查看`src/parser.c`。

外部扫描器只是一些自定义的C代码，插入到解析器中，它允许我们覆盖解析器的优先级，跟踪上下文状态或者其他我们可能需要或想要做的事情。

让我们看看。

让我们从遇到`:::`时提前关闭段落开始。这更简单，因为我们可以在不存储任何状态的情况下解决这个问题。

在解析`$.paragraph`时，我们将让解析器在换行符或我们新的`$._close_paragraph`标记上选择结束段落：

```
paragraph: ($) =>
  seq(repeat1(seq($._inline, "\n")), choice("\n", $._close_paragraph)), 
```

`$._close_paragraph`由外部扫描器处理，其使用`externals`字段指定：

```
externals: ($) => [$._close_paragraph], 
```

现在让我们转向`src/scanner.c`。`externals`中的标记会被分配一个递增的编号，从0开始……就像C中的枚举一样！

```
typedef enum { CLOSE_PARAGRAPH } TokenType; 
```

我们需要实现的五个函数如下：

```
bool tree_sitter_sdjot_external_scanner_scan(void *payload, TSLexer *lexer,
                                             const bool *valid_symbols)  { return false; }

void *tree_sitter_sdjot_external_scanner_create()  { return NULL; }
void tree_sitter_sdjot_external_scanner_destroy(void *payload)  {}

unsigned tree_sitter_sdjot_external_scanner_serialize(void *payload,
                                                      char *buffer)  { return 0; }
void tree_sitter_sdjot_external_scanner_deserialize(void *payload, char *buffer,
                                                    unsigned length)  {} 
```

因为我们不会使用任何状态，所以我们只需要更新`scan`函数。

你应该做的是检查`valid_symbols`中我们可以在任何时候返回的标记，并且如果找到任何标记，则返回`true`：

```
bool tree_sitter_sdjot_external_scanner_scan(void *payload, TSLexer *lexer,
                                             const bool *valid_symbols)  { if (valid_symbols[CLOSE_PARAGRAPH] && parse_close_paragraph(lexer)) {
    return true;
  }
  return false; } 
```

要决定是否提前关闭段落，我们将向前查找任何`:::`，如果找到，则在不消耗任何字符的情况下关闭它。这可能不是最有效的解决方案，因为我们将不得不再次解析`:::`，但它完成了工作。

匹配的标记应存储在`lexer->result_symbol`中：

```
static bool parse_close_paragraph(TSLexer *lexer)  { lexer->mark_end(lexer);

  uint8_t colons = consume_chars(lexer, ':');
  if (colons >= 3) {
    lexer->result_symbol = CLOSE_PARAGRAPH;
    return true;
  } else {
    return false;
  } } 
```

注意，结果标记将标记我们跨越的任何符号为该标记所拥有。所以`:::`会被标记为`_close_paragraph`（由于以下划线开头，输出将忽略它），而不是`div_marker`。为了防止这种情况，我们在推进词法分析器之前将`_close_paragraph`转换为零宽标记。

我们如何推进词法分析器？我们调用`lexer->advance`：

```
static uint8_t consume_chars(TSLexer *lexer, char c)  { uint8_t count = 0;
  while (lexer->lookahead == c) {
    lexer->advance(lexer, false);
    ++count;
  }
  return count; } 
```

这几乎是我们可以用词法分析器做的所有事情。我们一次只处理一个字符，不能向后查看，而向前查看的唯一工具是在正确的位置`mark_end`（我们也可以查询当前列位置）。

有了这个，我们现在有一个工作的外部扫描器和div标签现在可以关闭段落：

```
:::
A paragraph inside a div
::: 
```

```
$ tree-sitter parse example-file
(document [0, 0] - [4, 0]
  (div [0, 0] - [3, 0]
    (div_marker [0, 0] - [0, 3])
    (paragraph [1, 0] - [2, 0])
    (div_marker [2, 0] - [2, 3]))) 
```

要自动关闭其他打开的块，我们需要向解析器添加一些上下文，这意味着我们需要状态管理。

我们正在实现的这个小子集只关注关闭divs——否则这篇文章会太长了——但我会尝试以一种更一般的方式来实现，以更好地说明真实世界解析器的工作方式。

我们的策略是这样的：

1.  一个div可以有一个变化的数量的`:`，必须匹配。

    因此，我们将在外部扫描器中解析冒号，并将其存储在堆栈上。

1.  当我们找到一个div标记时，我们需要决定它是否应该开始一个新的div，或者关闭一个现有的div。

    我们将查看打开块的堆栈，并查找是否有匹配项。

1.  如果我们需要关闭嵌套的div，也就是说，如果我们想要关闭堆栈中更深处的div，我们需要先关闭嵌套的div（们）。

    因此，我们会引入一个`block_close`标记来结束一个div，并将结束div标记为可选的。

首先，我们会要求语法让外部扫描器管理开始和结束令牌。我们将使用`_block_close`标记来结束div，并将结束标记设为可选的。（在我实现时，我觉得这样更合理，而不是在两者之间使用`choice()`。）

```
div: ($) =>
  prec.left( seq( alias($._div_marker_begin, $.div_marker),
      "\n",
      repeat($._block),
      $._block_close,
      optional(alias($._div_marker_end, $.div_marker))
    )
  ),

externals: ($) => [
  $._close_paragraph,
  $._block_close,
  $._div_marker_begin,
  $._div_marker_end,

      $._ignored,
], 
```

并记得更新扫描器中外部令牌的列表（顺序很重要）：

```
typedef enum {
  CLOSE_PARAGRAPH,
  BLOCK_CLOSE,
  DIV_MARKER_BEGIN,
  DIV_MARKER_END,
 IGNORED } TokenType; 
```

然后添加到我们的块的堆栈中。

我使用了`Block`类型来追踪冒号的类型和数量：

```
typedef enum { DIV } BlockType;

typedef struct {
  BlockType type;
  uint8_t level;
} Block; 
```

我知道`level`不是最好的名称，但我找不到一个非常好的一般性名称来表示冒号的数量、缩进级别等。使用总和类型，您可以以更清晰的方式对其建模，例如：

```
enum Block {
    Div { colons: u32 },
    Footnote { indent: u32 }, } 
```

> 实际上，我要说的是，一个糟糕的程序员和一个好的程序员之间的区别在于他是否认为他的代码或数据结构更重要。糟糕的程序员担心代码。好的程序员担心数据结构及其关系。

但我跑题了，我会像一个糟糕的程序员一样，使用`level`。

C语言编程的另一个乐趣是，您将不得不重新实现标准数据结构，如可增长栈。这并不真正困难，但很烦人且容易出错。

幸运的是，在我撰写这篇博客文章的时候，[tree-sitter 0.22.1](https://github.com/tree-sitter/tree-sitter/releases/tag/v0.22.1)已发布了一个数组实现。因此，我不必展示我笨拙的堆栈实现，我们可以使用它们的数组作为我们的堆栈。

我们将我们的`Array`的`Block*`塞进一个`Scanner`结构体中，因为我们以后需要跟踪更多的数据：

```
#include "tree_sitter/array.h" 
typedef struct {
  Array(Block *) * open_blocks;
} Scanner; 
```

当您在tree-sitter中管理状态时，需要在我们之前定义的`tree_sitter_`函数中进行一些数据管理。

分配由`_create`和`_destroy`函数管理。0.22.1的另一个新特性是建议使用`ts_`函数进行分配，以允许消费者覆盖默认分配器：

```
#include "tree_sitter/alloc.h" 
void *tree_sitter_sdjot_external_scanner_create()  { Scanner *s = (Scanner *)ts_malloc(sizeof(Scanner));

    s->open_blocks = ts_malloc(sizeof(Array(Block *)));
  array_init(s->open_blocks);

  return s; }

void tree_sitter_sdjot_external_scanner_destroy(void *payload)  { Scanner *s = (Scanner *)payload;

        for (size_t i = 0; i < s->open_blocks->size; ++i) {
        ts_free(array_get(s->open_blocks, i));
  }

      array_delete(s->open_blocks);

  ts_free(s); } 
```

我在`push_block`辅助函数中分配这些块：

```
static void push_block(Scanner *s, BlockType type, uint8_t level)  { Block *b = ts_malloc(sizeof(Block));
  b->type = type;
  b->level = level;

    array_push(s->open_blocks, b); } 
```

您还需要定义序列化函数。这些函数存储和检索管理状态，以允许tree-sitter进行回溯。

```
unsigned tree_sitter_sdjot_external_scanner_serialize(void *payload,
                                                      char *buffer)  { Scanner *s = (Scanner *)payload;
  unsigned size = 0;
  for (size_t i = 0; i < s->open_blocks->size; ++i) {
    Block *b = *array_get(s->open_blocks, i);
    buffer[size++] = (char)b->type;
    buffer[size++] = (char)b->level;
  }
  return size; }

void tree_sitter_sdjot_external_scanner_deserialize(void *payload, char *buffer,
                                                    unsigned length)  { Scanner *s = (Scanner *)payload;
  array_init(s->open_blocks);
  size_t size = 0;
  while (size < length) {
    BlockType type = (BlockType)buffer[size++];
    uint8_t level = (uint8_t)buffer[size++];
    push_block(s, type, level);
  } } 
```

这样，（初始的）状态管理问题就解决了！

当然，我们还没有使用我们的状态。让我们改变这一点。

首先，让我们在我们的扫描函数中添加`parse_div`入口点：

```
bool tree_sitter_sdjot_external_scanner_scan(void *payload, TSLexer *lexer,
                                             const bool *valid_symbols)  { Scanner *s = (Scanner *)payload;

    if (valid_symbols[CLOSE_PARAGRAPH] && parse_close_paragraph(lexer)) {
    return true;
  }

    if (parse_div(s, lexer, valid_symbols)) {
    return true;
  }

  return false; } 
```

因为推进词法分析器是原始的，并且我们不能“返回一个字符”，所以重要的是只有在真正需要推进时才这样做。因此，在继续之前我们始终需要检查`valid_symbols`：

```
static bool parse_div(Scanner *s, TSLexer *lexer, const bool *valid_symbols)  { if (!valid_symbols[DIV_MARKER_BEGIN] && !valid_symbols[DIV_MARKER_END]) {
    return false;
  } } 
```

接下来，我们需要消耗我们所处的所有冒号，并且只有在看到至少三个冒号时才继续：

```
static uint8_t consume_chars(TSLexer *lexer, char c)  { uint8_t count = 0;
  while (lexer->lookahead == c) {
    lexer->advance(lexer, false);
    ++count;
  }
  return count; }

static bool parse_div(Scanner *s, TSLexer *lexer, const bool *valid_symbols)  { uint8_t colons = consume_chars(lexer, ':');
  if (colons < 3) {
    return false;
  } } 
```

打开一个新的 div 很简单；我们推入块并注册冒号的数量：

```
push_block(s, DIV, colons);
lexer->result_symbol = DIV_MARKER_BEGIN;
return true; 
```

但是，要决定是否应该打开或关闭 div，我们需要一种方法来搜索堆栈。这个函数就是做这个的，同时还会返回我们在堆栈中找到的 div 的深度（我们很快会用到）：

```
static size_t number_of_blocks_from_top(Scanner *s, BlockType type,
                                        uint8_t level)  { for (int i = s->open_blocks->size - 1; i >= 0; --i) {
    Block *b = *array_get(s->open_blocks, i);
    if (b->type == type && b->level == level) {
      return s->open_blocks->size - i;
    }
  }
  return 0; }

static bool parse_div(Scanner *s, TSLexer *lexer, const bool *valid_symbols)  { size_t from_top = number_of_blocks_from_top(s, DIV, colons);

      if (from_top > 0) {
      } else {
        lexer->mark_end(lexer);
    push_block(s, DIV, colons);
    lexer->result_symbol = DIV_MARKER_BEGIN;
    return true;
  } } 
```

但是我们有一个问题：当我们想关闭 div 时，我们希望能够输出多个标记。

例如，使用这种类型的输入：

```
:::
:::::
:::::::
text
::: 
```

当我们看到关闭的`:::`标记时，我们将有一个包含 3 个 div 的堆栈：

```
7 (top)
5
3 (the one we want to close) 
```

在上面的代码中，`from_top`将是`3`，我们需要输出 4 个标记：3 个`BLOCK_CLOSE`（每个 div 一个）和 1 个`DIV_MARKER_END`（用于最后的`:::`）。但是扫描器一次只能输出一个标记。

我解决这个问题的方式是向扫描器引入更多状态。具体来说，我引入了一个名为`blocks_to_close`的变量，我们将用它来输出`BLOCK_CLOSE`，以及一些用于输出（和消耗）`DIV_MARKER_END`的变量。

```
typedef struct {
  Array(Block *) * open_blocks;

    uint8_t blocks_to_close;

    TokenType delayed_token;
    uint8_t delayed_token_width;
} Scanner; 
```

我们需要记住更新创建和序列化函数。

序列化：

```
buffer[size++] = (char)s->blocks_to_close;
buffer[size++] = (char)s->delayed_token;
buffer[size++] = (char)s->delayed_token_width; 
```

反序列化：

```
s->blocks_to_close = (uint8_t)buffer[size++];
s->delayed_token = (TokenType)buffer[size++];
s->delayed_token_width = (uint8_t)buffer[size++]; 
```

我们将使用`IGNORED`作为未使用的标记，因此在创建扫描器时需要重置它：

```
s->blocks_to_close = 0;
s->delayed_token = IGNORED; 
```

现在当我们扫描时，我们应该先检查`blocks_to_close`，然后再检查`delayed_token`，然后再扫描其他内容：

```
bool tree_sitter_sdjot_external_scanner_scan(void *payload, TSLexer *lexer,
                                             const bool *valid_symbols)  { Scanner *s = (Scanner *)payload;

  if (valid_symbols[BLOCK_CLOSE] && handle_blocks_to_close(s, lexer)) {
    return true;
  }

  if (output_delayed_token(s, lexer, valid_symbols)) {
    return true;
  } } 
```

当我们看到`blocks_to_close > 0`时，我们应该输出一个`BLOCK_CLOSE`并删除顶部块（一些健全性检查以确保安全）：

```
static void remove_block(Scanner *s)  { if (s->open_blocks->size > 0) {
    ts_free(array_pop(s->open_blocks));
    if (s->blocks_to_close > 0) {
      --s->blocks_to_close;
    }
  } }

static bool handle_blocks_to_close(Scanner *s, TSLexer *lexer)  { if (s->open_blocks->size == 0) {
    return false;
  }

    if (lexer->eof(lexer) || s->blocks_to_close > 0) {
    lexer->result_symbol = BLOCK_CLOSE;
    remove_block(s);
    return true;
  }
  return false; } 
```

有了这个，我们可以输出多个`BLOCK_CLOSE`，现在来处理延迟的标记：

```
static bool output_delayed_token(Scanner *s, TSLexer *lexer,
                          const bool *valid_symbols)  { if (s->delayed_token == IGNORED || !valid_symbols[s->delayed_token]) {
    return false;
  }

  lexer->result_symbol = s->delayed_token;
  s->delayed_token = IGNORED;
    while (s->delayed_token_width--) {
    lexer->advance(lexer, false);
  }
  lexer->mark_end(lexer);
  return true; } 
```

另一种设计方式是拥有一个延迟标记的堆栈，然后只需弹出它。这当然更强大，我只是在玩耍时选择了这种方式，因为它更加明确，感觉更容易跟踪发生了什么。

无论哪种方式，我们现在都可以实现 div 结束处理。在`parse_div`中：

```
size_t from_top = number_of_blocks_from_top(s, DIV, colons);

if (from_top > 0) {
    close_blocks_with_final_token(s, lexer, from_top, DIV_MARKER_END, colons);
  return true;
} else {
  lexer->mark_end(lexer);
  push_block(s, DIV, colons);
  lexer->result_symbol = DIV_MARKER_BEGIN;
  return true;
} 
```

`close_blocks_with_final_token`是一个通用的辅助函数，设置要关闭的块数和最终标记：

```
static void close_blocks_with_final_token(Scanner *s, TSLexer *lexer,
                                          size_t count, TokenType final,
                                          uint8_t final_token_width)  { remove_block(s);
  s->blocks_to_close = s->blocks_to_close + count - 1;
  lexer->result_symbol = BLOCK_CLOSE;
  s->delayed_token = final;
  s->delayed_token_width = final_token_width; } 
```

现在我们终于可以尝试关闭 div 了：

```
:::::
:::
:::::::
Divception
::: 
```

```
$ tree-sitter parse example-file
(document [0, 0] - [6, 0]
  (div [0, 0] - [6, 0]
    (div_marker [0, 0] - [0, 5])
    (div [1, 0] - [4, 3]
      (div_marker [1, 0] - [1, 3])
      (div [2, 0] - [4, 0]
        (div_marker [2, 0] - [2, 7])
        (paragraph [3, 0] - [4, 0]))
      (div_marker [4, 0] - [4, 3])))) 
```

我们可以看到它在解析时没有错误，最后一个标记正确关闭了第二个 div，并且最后一个标记捕获了最后的`:::`。

尽管我在这篇文章中直接跳到了一个工作实现，但当我第一次尝试时当然不是这样。我发现`-d`参数很有用，可以看到在每一步中消耗了哪些字符并输出了什么标记。

这是输出的一部分（扫描最后的`:::`时），并附有一些注释指出一些有趣的事情：

```
$ tree-sitter parse example-file -d
...
process version:0, version_count:1, state:34, row:4, col:0
lex_external state:4, row:4, column:0
  consume character:':'                         // Scan `:::`
  consume character:':'
  consume character:':'
lexed_lookahead sym:_close_paragraph, size:0    // Output _close_paragraph
reduce sym:paragraph_repeat1, child_count:2
shift state:17
process version:0, version_count:1, state:17, row:4, col:0
lex_external state:3, row:4, column:0           // Still on first `:`
  consume character:':'                         // Scan `:::` again
  consume character:':'
  consume character:':'
lexed_lookahead sym:_block_close, size:0        // Close div with _block_close
reduce sym:paragraph, child_count:2
shift state:12
process version:0, version_count:1, state:12, row:4, col:0
lex_external state:5, row:4, column:0           // Still on first `:`
lexed_lookahead sym:_block_close, size:0        // Close second div with _block_close
reduce sym:div, child_count:4
shift state:12
process version:0, version_count:1, state:12, row:4, col:0
lex_external state:5, row:4, column:0           // Still on first `:`
  consume character:':'                         // Consume `:::`
  consume character:':'
  consume character:':'
lexed_lookahead sym:div_marker, size:3          // div_marker is size 3, marks `:::`
shift state:23 
```

虽然输出看起来令人困惑，但是当你知道要找什么时，它是非常有用的。我发现，采用一个深思熟虑的过程，逐个字符查看，有助于我解决目前遇到的问题。
