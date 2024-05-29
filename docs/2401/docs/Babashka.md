<!--yml
category: 未分类
date: 2024-05-27 15:08:44
-->

# Babashka

> 来源：[https://babashka.org/](https://babashka.org/)

### Instant startup

Leveraging GraalVM native-image and the Small Clojure Interpreter, babashka is a self-contained and instantly starting scripting environment.

Babashka comes with scripting batteries included: tools.cli, cheshire, babashka.fs, babashka.process, java.time and many more libraries and classes.

### Cross-platform

Babashka scripts work on linux, macOS and Windows.

Besides the built-in libraries, babashka is able to load libraries from source, tapping into the world of already existing Clojure libraries.

### Multi-threaded

Babashka supports real JVM threads and like Clojure, supports futures and dynamic thread-locally bound vars

Babashka features a built-in task runner which covers the most popular use cases of make, just and npm scripts.

Babashka can shell out to other CLI programs like you are used to in bash. It goes one step further and offers seamless integration with other binaries using the pod protocol. Pods can be implemented in any language, including Clojure, Rust and Go.