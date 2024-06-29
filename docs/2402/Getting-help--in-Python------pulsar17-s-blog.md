<!--yml

类别：未分类

日期：2024-05-29 13:24:34

-->

# 获取Python帮助 🧙 - pulsar17的博客

> 来源：[https://pulsar17.me/2024/02/ongettinghelp](https://pulsar17.me/2024/02/ongettinghelp)

## 上下文

当我乘坐德里地铁去[PyDelhi](https://pydelhi.org)的二月聚会时，我在考虑我能做什么演讲。有一段时间，我一直想探索Python中`sleep()`函数的实现方式。所以我想我会谈一下`sleep()`，顺便展示一下`strace`...

我就是记不起`sleep()`函数是从哪里来的。它是内置的吗？我需要导入一个模块吗？我打开了Python REPL来探索：

我尝试了以下事情：

```
>>> sleep(5)
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
NameError: name 'sleep' is not defined
>>> import os
>>> os.sleep(5)
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
AttributeError: module 'os' has no attribute 'sleep' 
```

当然，正确的版本是：

```
>>> import time
>>> time.sleep(5) 
```

但那时，我记不起来了！我知道有一个`sleep()`，因为我过去已经用过它了。（我们都有吧 😉）

## 请求`help()`

Python有一个内置的`help()`函数，我认为这是Python中未被充分利用的部分。我经常首先使用它，而不是搜索网页。

它的使用方式大致是这样的：

你传入你想要获取帮助的对象。如果你已经导入了模块，它也适用于模块：

```
>>> import os
>>> help(os) 
```

或者你可以*meta* 😀：

## 终于明白了`help()`

我绝望地尝试了：

```
>>> help("sleep")
No Python documentation found for 'sleep'.
Use help() to get the interactive help utility.
Use help(str) for help on the str class. 
```

然后只需

```
Welcome to Python 3.12's help utility! If this is your first time using
Python, you should definitely check out the tutorial at
https://docs.python.org/3.12/tutorial/.

Enter the name of any module, keyword, or topic to get help on writing
Python programs and using Python modules.  To get a list of available
modules, keywords, symbols, or topics, enter "modules", "keywords",
"symbols", or "topics".

Each module also comes with a one-line summary of what it does; to list
the modules whose name or summary contain a given string such as "spam",
enter "modules spam".

To quit this help utility and return to the interpreter,
enter "q" or "quit".

help>

```

这就是事情的全貌！当然我忽略了，然后尝试了几种不同的组合。看来我错过了，我的提示符从`>>>`变成了`help>`。

```
help> help() # notice the prompt! 
```

过了一段时间，我意识到提示符的差异，并仔细阅读了上述消息。

现在我对那条消息中的`"topics"`感到好奇了。

```
ASSERTION           DELETION            LOOPING             SHIFTING
ASSIGNMENT          DICTIONARIES        MAPPINGMETHODS      SLICINGS
ATTRIBUTEMETHODS    DICTIONARYLITERALS  MAPPINGS            SPECIALATTRIBUTES
ATTRIBUTES          DYNAMICFEATURES     METHODS             SPECIALIDENTIFIERS
AUGMENTEDASSIGNMENT ELLIPSIS            MODULES             SPECIALMETHODS
BASICMETHODS        EXCEPTIONS          NAMESPACES          STRINGMETHODS
BINARY              EXECUTION           NONE                STRINGS
BITWISE             EXPRESSIONS         NUMBERMETHODS       SUBSCRIPTS
BOOLEAN             FLOAT               NUMBERS             TRACEBACKS
CALLABLEMETHODS     FORMATTING          OBJECTS             TRUTHVALUE
CALLS               FRAMEOBJECTS        OPERATORS           TUPLELITERALS
CLASSES             FRAMES              PACKAGES            TUPLES
CODEOBJECTS         FUNCTIONS           POWER               TYPEOBJECTS
COMPARISON          IDENTIFIERS         PRECEDENCE          TYPES
COMPLEX             IMPORTING           PRIVATENAMES        UNARY
CONDITIONAL         INTEGER             RETURNING           UNICODE
CONTEXTMANAGERS     LISTLITERALS        SCOPING
CONVERSIONS         LISTS               SEQUENCEMETHODS
DEBUGGING           LITERALS            SEQUENCES

```

如果现在你输入一个主题：

你会得到一些有用的东西（*... 表示被裁剪的输出*）

```
Mapping Types — "dict"
**********************

A *mapping* object maps *hashable* values to arbitrary objects.
Mappings are mutable objects.  There is currently only one standard

...

```

太棒了！ 💯

那时我知道我必须谈论的是什么。不是`sleep()`，而是`help()`以及我甚至不知道存在的Python内部的这个美丽的帮助子系统。

## 为什么甚至

后来我意识到这就是可以在[官方Python文档](https://docs.python.org/3/library/stdtypes.html#typesmapping)中找到的相同文档，就在我的终端里！

对我来说，这很重要，因为我当时在地铁上没有互联网访问。如果我在飞行中，情况也是一样的，拥有这些信息就在那里 - 离线 - 这是非常有力的！

我邀请读者尝试这种交互式帮助，还有一些其他有趣的事情，我没有覆盖，同时我也希望这个*发现*对你有用。
