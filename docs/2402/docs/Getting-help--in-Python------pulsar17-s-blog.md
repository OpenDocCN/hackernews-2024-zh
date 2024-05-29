<!--yml
category: 未分类
date: 2024-05-29 13:24:34
-->

# Getting help [in Python] 🧙 - pulsar17's blog

> 来源：[https://pulsar17.me/2024/02/ongettinghelp](https://pulsar17.me/2024/02/ongettinghelp)

## Context

While I was travelling in the Delhi Metro for [PyDelhi](https://pydelhi.org)'s February meetup, I was thinking about what I could present. For some time I have had this idea of looking into how the `sleep()` function in Python is implemented. So I thought I would present about `sleep()`, show a bit of `strace` on the side...

I just couldn't remember where `sleep()` function would come from. Was it a built-in? Did I need to import a module? I opened the Python REPL to explore:

I tried the following things:

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

Of course, the correct version is:

```
>>> import time
>>> time.sleep(5) 
```

But at that time, I couldn't remember! I knew there was a `sleep()` because I had already used it in the past. (Haven't we all 😉)

## Calling for `help()`

Python has a built-in `help()` function, which in my opinion is an underused part of Python. I often use it first instead of searching the web.

It is used something like:

You pass in the object you want the help on. It works on modules as well if you've imported them:

```
>>> import os
>>> help(os) 
```

or you could go *meta* 😀 with:

## Finally getting `help()`

I then desperately tried:

```
>>> help("sleep")
No Python documentation found for 'sleep'.
Use help() to get the interactive help utility.
Use help(str) for help on the str class. 
```

then just

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

That is something! I ignored it of course and tried a few different combinations. I seemed to have missed though that my prompt had changed from `>>>` to `help>`.

```
help> help() # notice the prompt! 
```

After some time I realized the difference in the prompt and read the above message more carefully.

I was now curious about the `"topics"` in that message.

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

If you now type a topic:

you get some useful things (*... indicates clipped output*)

```
Mapping Types — "dict"
**********************

A *mapping* object maps *hashable* values to arbitrary objects.
Mappings are mutable objects.  There is currently only one standard

...

```

Amazing! 💯

I knew then what I had to talk about. It was not `sleep()` but `help()` and this beautiful help subsystem inside Python which I didn't even know existed.

## Why even

I later realized that this is the same documentation that can be found on the [official Python docs](https://docs.python.org/3/library/stdtypes.html#typesmapping) right inside my terminal!

This was important for me then as I had no internet access while on the Metro. It would be the same if I was on a flight, and having this information right there - offline - is extremely empowering!

I invite the reader to try this interactive help, there are some other interesting things as well which I did not cover, I also hope that this *discovery* proves useful to you.