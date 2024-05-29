<!--yml
category: 未分类
date: 2024-05-29 12:32:10
-->

# Jonas Hietala: Let's create a Tree-sitter grammar

> 来源：[https://www.jonashietala.se/blog/2024/03/19/lets_create_a_tree-sitter_grammar/](https://www.jonashietala.se/blog/2024/03/19/lets_create_a_tree-sitter_grammar/)

A Tree-sitter parser is actually a C program. The grammar we’ve seen has been described in JavaScript, but it’s only used as a description to generate the parser in C. If you’re a masochist, you can take a look at it in `src/parser.c` after running `tree-sitter generate`.

An external scanner is just some custom C code that’s inserted into the parser, and it allows us to override the parser precedence, keep track of a context state, or whatever else we might need or want to do.

Let’s take a look.

Let’s start by closing a paragraph early when a `:::` is encountered. This is simpler because we can solve this without storing any state.

When parsing `$.paragraph` we’ll give the parser a choice between ending the paragraph on a newline or on our new `$._close_paragraph` token:

```
paragraph: ($) =>
  seq(repeat1(seq($._inline, "\n")), choice("\n", $._close_paragraph)), 
```

`$._close_paragraph` is handled by the external scanner, which is specified using the `externals` field:

```
externals: ($) => [$._close_paragraph], 
```

Now let’s turn our attention to `src/scanner.c`. The tokens in `externals` gets assigned an incremented number, starting from 0… Just like an enum in C!

```
typedef enum { CLOSE_PARAGRAPH } TokenType; 
```

The five functions we need to implement are these:

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

Because we won’t use any state, we’ll only have to update the `scan` function.

What you’re supposed to do is check `valid_symbols` for the tokens we can return at any point in time, and return `true` if any was found:

```
bool tree_sitter_sdjot_external_scanner_scan(void *payload, TSLexer *lexer,
                                             const bool *valid_symbols)  { if (valid_symbols[CLOSE_PARAGRAPH] && parse_close_paragraph(lexer)) {
    return true;
  }
  return false; } 
```

To decide if we’re going to close the paragraph early, we’ll look ahead for any `:::`, and if so we’ll close it without consuming any characters. This might not be the most efficient solution because we’ll have to parse the `:::` again, but it gets the job done.

The matched token should be stored in `lexer->result_symbol`:

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

Note that the resulting token will mark any symbol we advance over as owned by that token. So `:::` would be marked as `_close_paragraph` (which will be ignored by the output since it begins with an underscore), instead of `div_marker`. To prevent this, we turn `_close_paragraph` into a zero-width token by marking the end before advancing the lexer.

How do we advance the lexer? We call `lexer->advance`:

```
static uint8_t consume_chars(TSLexer *lexer, char c)  { uint8_t count = 0;
  while (lexer->lookahead == c) {
    lexer->advance(lexer, false);
    ++count;
  }
  return count; } 
```

This is almost all we can do with the lexer. We only process one character at a time, cannot look behind, and our only tool to look ahead is to `mark_end` at the correct place. (We can also query the current column position.)

With this we have a working external scanner and div tags now close paragraphs:

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

To automatically close other open blocks we need to add some context to our parser, which means we’ll need state management.

The small subset we’re implementing is only concerned with closing divs—because it would be a terribly long post otherwise—but I’ll try to implement this in a general manner, to be more indicative of a real-world parser.

Our strategy is this:

1.  A div can have a varying number of `:` that must match.

    Therefore we’ll parse colons in an external scanner and store it on a stack.

2.  When we find a div marker, we’ll need to decide if it should start a new div, or close an existing one.

    We’ll look at the stack of open blocks and see if we find a match.

3.  If we have need to close a nested div, that is if we want to close a div further down the stack, we need to close the nested div(s) first.

    Thus we’ll introduce a `block_close` marker that ends a div, and leave the ending div marker as optional.

First we’ll ask the grammar to let the external scanner manage the begin and end tokens. We’ll use a `_block_close` marker to end the div, and leave the end marker optional. (You could probably use a `choice()` between the two, but this made more sense to me when I was implementing it.)

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

And remember to update the list of external tokens in the scanner (order matters):

```
typedef enum {
  CLOSE_PARAGRAPH,
  BLOCK_CLOSE,
  DIV_MARKER_BEGIN,
  DIV_MARKER_END,
 IGNORED } TokenType; 
```

Then to our stack of blocks.

I used a `Block` type to keep track of the type and number of colons:

```
typedef enum { DIV } BlockType;

typedef struct {
  BlockType type;
  uint8_t level;
} Block; 
```

I know that `level` isn’t the best name, but I couldn’t find a very good general name for the number of colons, indentation level, etc. With sum types you could model it in a clearer way, like this:

```
enum Block {
    Div { colons: u32 },
    Footnote { indent: u32 }, } 
```

> I will, in fact, claim that the difference between a bad programmer and a good one is whether he considers his code or his data structures more important. Bad programmers worry about the code. Good programmers worry about data structures and their relationships.

But I digress, I’ll go with `level` like a bad programmer.

Another joy of programming C is that you’ll get to re-implement standard data structures such as a growable stack. It’s not truly difficult, but it’s annoying and bug-prone.

Luckily, during the time I’m writing this blog post, [tree-sitter 0.22.1](https://github.com/tree-sitter/tree-sitter/releases/tag/v0.22.1) was released with an array implementation. So now I don’t have to show you my shoddy stack implementation, and we can use their array for our stack instead.

We’ll shove our `Array` of `Block*` into a `Scanner` struct, because we’ll need to track more data later:

```
#include "tree_sitter/array.h" 
typedef struct {
  Array(Block *) * open_blocks;
} Scanner; 
```

When you manage state in tree-sitter, you need to do some data management in the `tree_sitter_` functions we defined earlier.

Allocations are managed in the `_create` and `_destroy` functions. Also new for 0.22.1 is the recommendation to use `ts_` functions for allocations, to allow consumers to override the default allocator:

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

I allocate the blocks in a `push_block` helper:

```
static void push_block(Scanner *s, BlockType type, uint8_t level)  { Block *b = ts_malloc(sizeof(Block));
  b->type = type;
  b->level = level;

    array_push(s->open_blocks, b); } 
```

You also need to define the serialize functions. These store and retrieve the managed state, to allow tree-sitter to backtrack.

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

And that’s the (initial) state management taken care of!

Of course, we haven’t used our state yet. Let’s change that.

First, let’s add the `parse_div` entry point to our scan function:

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

Because advancing the lexer is primitive, and we cannot “go back a char”, it’s important to only advance it if we really need to. Therefore we always need to check `valid_symbols` before we continue:

```
static bool parse_div(Scanner *s, TSLexer *lexer, const bool *valid_symbols)  { if (!valid_symbols[DIV_MARKER_BEGIN] && !valid_symbols[DIV_MARKER_END]) {
    return false;
  } } 
```

Next we’ll need to consume all colons we’re at, and only continue if we see at least three:

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

Opening a new div is simple; we push the block and register the number of colons:

```
push_block(s, DIV, colons);
lexer->result_symbol = DIV_MARKER_BEGIN;
return true; 
```

But to the decide if we should open or close a div, we need a way to search through the stack. This function does that, while also returning how many blocks deep into the stack we found the div (which we’ll use shortly):

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

But we have a problem: when we want to close the div, we want to be able to output multiple tokens.

For example, with this type of input:

```
:::
:::::
:::::::
text
::: 
```

We’ll have a stack of 3 divs when we see the closing `:::` marker:

```
7 (top)
5
3 (the one we want to close) 
```

In the code above, `from_top` will be `3` and we need to output 4 tokens: 3 `BLOCK_CLOSE` (one for each div) and 1 `DIV_MARKER_END` (for the last `:::`). But the scanner can only output a single token at a time.

The way I solved this is by introducing more state to the Scanner. Specifically, I introduced a `blocks_to_close` variable that we’ll use to output `BLOCK_CLOSE`, and some variables to output (and consume) the `DIV_MARKER_END`.

```
typedef struct {
  Array(Block *) * open_blocks;

    uint8_t blocks_to_close;

    TokenType delayed_token;
    uint8_t delayed_token_width;
} Scanner; 
```

We need to remember to update the create and serialize functions too.

Serialize:

```
buffer[size++] = (char)s->blocks_to_close;
buffer[size++] = (char)s->delayed_token;
buffer[size++] = (char)s->delayed_token_width; 
```

Deserialize:

```
s->blocks_to_close = (uint8_t)buffer[size++];
s->delayed_token = (TokenType)buffer[size++];
s->delayed_token_width = (uint8_t)buffer[size++]; 
```

We’ll use `IGNORED` as the unused token, so we’ll need to reset it when we create the scanner:

```
s->blocks_to_close = 0;
s->delayed_token = IGNORED; 
```

Now when we scan we should first check `blocks_to_close` and then `delayed_token`, before we scan other things:

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

When we see `blocks_to_close > 0`, we should output a `BLOCK_CLOSE` and remove the top block (with some sanity checks for good measure):

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

With this we can output multiple `BLOCK_CLOSE`, and now to handle delayed tokens:

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

Another way to design this is to have a stack of delayed tokens and then just pop that. It’s certainly more powerful, I just happened to choose this way when I was playing around with it because it’s more explicit and it felt a little easier to follow what was happening.

Either way, we can now implement the div end handling. In `parse_div`:

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

`close_blocks_with_final_token` is a general helper that sets up the number of blocks to close and the final token:

```
static void close_blocks_with_final_token(Scanner *s, TSLexer *lexer,
                                          size_t count, TokenType final,
                                          uint8_t final_token_width)  { remove_block(s);
  s->blocks_to_close = s->blocks_to_close + count - 1;
  lexer->result_symbol = BLOCK_CLOSE;
  s->delayed_token = final;
  s->delayed_token_width = final_token_width; } 
```

Now we can finally try to close divs:

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

We can see that it parses without error, the last marker closes the *second* div correctly, and the last marker captures the final `:::`.

While I’m jumping to a working implementation directly in this post, when I first did this that was of course not the case. I found the `-d` argument useful to see what characters are consumed and what token is output in each step.

Here’s a part of the output (when scanning the final `:::`), with some comments to point out some interesting things:

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

While the output seems confusing, when you know what to look for it’s very useful. I’ve found that a deliberate process, where I look at a single character at a time, helps me get through the problems I’ve encountered so far.