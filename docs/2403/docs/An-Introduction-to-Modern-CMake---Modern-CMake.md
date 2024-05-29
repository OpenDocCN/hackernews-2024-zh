<!--yml
category: 未分类
date: 2024-05-29 12:35:14
-->

# An Introduction to Modern CMake · Modern CMake

> 来源：[https://cliutils.gitlab.io/modern-cmake/](https://cliutils.gitlab.io/modern-cmake/)

# An Introduction to Modern CMake

People love to hate build systems. Just watch the talks from CppCon17 to see examples of developers making the state of build systems the brunt of jokes. This raises the question: Why? Certainly there are no shortage of problems when building. But I think that, in 2023, we have a very good solution to quite a few of those problems. It's CMake. Not CMake 2.8 though; that was released before C++11 even existed! Nor the horrible examples out there for CMake (even those posted on KitWare's own tutorials list). I'm talking about Modern CMake. CMake 3.5+, maybe even CMake 3.29+! It's clean, powerful, and elegant, so you can spend most of your time coding, not adding lines to an unreadable, unmaintainable Make (Or CMake 2) file. And CMake 3.11+ is supposed to be significantly faster, as well!

*Are you interested in using CMake to build Python packages? I'm working on scikit-build-core, [proposal described here](https://iscinumpy.gitlab.io/post/scikit-build-proposal/)! Let me know if you have a use case!*

 *In short, here are the most likely questions in your mind if you are considering Modern CMake:

## Why do I need a good build system?

Do any of the following apply to you?

*   You want to avoid hard-coding paths
*   You need to build a package on more than one computer
*   You want to use CI (continuous integration)
*   You need to support different OSs (maybe even just flavors of Unix)
*   You want to support multiple compilers
*   You want to use an IDE, but maybe not all of the time
*   You want to describe how your program is structured logically, not flags and commands
*   You want to use a library
*   You want to use tools, like Clang-Tidy, to help you code
*   You want to use a debugger

If so, you'll benefit from a CMake-like build system.

## Why must the answer be CMake?

Build systems are a hot topic. Of course there are many options. But even a really good one, or one that re-uses a familiar syntax, can't come close to CMake. Why? Support. Every IDE supports CMake (or CMake supports that IDE). More packages use CMake than any other system. So, if you use a library that is designed to be included in your code, you have a choice: Make your own build system, or use one of the provided ones, and that will almost always include CMake. And that will quickly be the common denominator if you include multiple projects. And, if you need a library that's preinstalled, the chances of it having a find CMake script or config CMake script are excellent.

## Why use a Modern CMake?

Around CMake 2.6-2.8, CMake started taking over. It was in most of the package managers for Linux OS's, and was being used in lots of packages.

Then Python 3 came out.

I know, this should have nothing whatsoever to do with CMake.

But it had a 3. And it followed 2. And it was a hard, ugly, transition that is still ongoing in some places, even today.

I believe that CMake 3 had the bad luck to follow Python 3.^([1](#fn_1)) Even though every version of CMake is insanely backward compatible, the 3 series was treated as if it were something new. And so, you'll find OSs like CentOS7 with GCC 4.8, with almost-complete C++14 support, and CMake 2.8, which came out years before C++11.

You really should *at least* use a version of CMake that came out after your compiler, since it needs to know compiler flags, etc, for that version. And, since CMake will dumb itself down to the minimum required version in your CMake file, installing a new CMake, even system wide, is pretty safe. You should *at least* install it locally. It's easy (1-2 lines in many cases), and you'll find that 5 minutes of work will save you hundreds of lines and hours of `CMakeLists.txt` writing, and will be much easier to maintain in the long run.

This book tries to solve the problem of the poor examples and best practices that you'll find proliferating the web.

## Other sources

Other material from the original author of this book:

There are some other places to find good information on the web. Here are some of them:

## Credits

Modern CMake was originally written by [Henry Schreiner](https://iscinumpy.gitlab.io). Other contributors can be found [listed on GitLab](https://gitlab.com/CLIUtils/modern-cmake/-/network/master).

> ¹. CMake 3.0 also removed several long deprecated features from very old versions of CMake and make one very tiny backwards incompatible change to syntax related to square brackets, so this is not entirely fair; there might be some very, very old CMake files that would stop working with 3\. I've never seen one, though. [↩](#reffn_1 "Jump back to footnote [1] in the text.")*