<!--yml
category: 未分类
date: 2024-05-27 12:51:34
-->

# A lazy and flippant classification of programming languages | Bryce’s Blog

> 来源：[https://blog.brycekerley.net/2024/04/01/p-langs.html](https://blog.brycekerley.net/2024/04/01/p-langs.html)

Sometimes I post in an online community of computer touchers, as one does. When talking flippantly about programming languages, it’s sometimes useful to glob a bunch of them together based on characteristics, whether superficial or not. This started with jokes about common features of “P languages” like Perl, PHP, Python, JavaScript and Ruby, but you can classify other languages too.

This classification isn’t intended as the final word or to be accepted uncritically. It’s mostly here to get it out of my head, and if it starts a conversation that’s good too?

## P-languages

Perl, PHP, Python, Ruby, Javascript (the last two have an honorary “P”) are all garbage-collected “scripting” ^(languages that let you get stuff done quick, often in web contexts. They often make sense from the beginning but get idiosyncratic at the edges. Often times huge swaths of the standard library will be very thin wrappers over the C standard library, system calls, or other platform libraries.)

## C-languages

A wise man once said that “the C++ spec is a case of regulatory capture by compiler vendors.” There’re enough cases of “undefined behavior” doing something absolutely ridiculous (like running off the end of an infinite loop into the next function) that C-language people have helpfully abbreviated it “UB” to save time while threatening you that maybe they’ll make a structural part of your program UB too.

These also have lots of well-named standard library functions like `printf`, `strcpy`, and `gets` that give control of your program counter and memory to arbitrary input. There are functions that supersede or obsolete these, but the old busted ones have to hang around forever. For example, C++ added safe ways to access `std::vector` and `std::array`, but gave them the name `at` instead of `operator[]` to not break the unsafe version that everyone uses.

## J-languages

Java and C#, these have separate compilation steps that go to a non-native bytecode that requires some kind of separate executable to run. They all subscribe to a “[Kingdom of Nouns](https://steve-yegge.blogspot.com/2006/03/execution-in-kingdom-of-nouns.html)” vision of “object oriented” that tends towards lots of gang of four patterns in codebases, that makes it hard to find the code that actually does stuff. They also both really like UTF-16.

## SQL

SQL

## G-languages

Golang, Rust, Crystal (my beloved), these are chasing after J-language safety while targeting native bytecode like a C-language, with a willingness to make syntax changes (if not the drastic typing system changes Rust and Crystal make) from C- and J-languages.

## R-languages

Erlang and Elixir are the big ones. They have the compilation to bytecode step of J-languages, but instead of calling methods on objects, you have lightweight processes passing messages around, and you get to use functional programming stuff too. Elixir works really hard to be an on-ramp to these for P-language people, but it’s not so much “idiosyncratic” at the edges as a whole new thing altogether.

## F-languages

Haskell, Ocaml, and other languages where concepts get names like “applicative functor” and variables get names like `v` and `m`.

# Conclusion

computers