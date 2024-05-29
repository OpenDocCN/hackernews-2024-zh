<!--yml
category: 未分类
date: 2024-05-27 14:33:32
-->

# In-browser code playgrounds

> 来源：[https://antonz.org/in-browser-code-playgrounds/](https://antonz.org/in-browser-code-playgrounds/)

# In-browser code playgrounds

I'm a big fan of interactive code snippets in all kinds of technical writing, from product docs to online courses to blog posts. Like this one:

```
def greet(name):  print(f"Hello, {name}!")   greet("World") 
```

 <codapi-snippet sandbox="python" editor="basic">In fact, I even built an open source tool called Codapi ^(for embedding such snippets.)

Typically, a code playground consists of a client-side widget and a server-side part that executes the code and returns the result:

```
 browser
┌───────────────────────────────┐
│ def greet(name):              │
│   print(f"Hello, {name}!")    │
│                               │
│ greet("World")                │
└───────────────────────────────┘
  Run ►
    ↓
  server
┌───────────────────────────────┐
│ docker run codapi/python      │
│ python main.py                │
└───────────────────────────────┘
    ↓
  browser
┌───────────────────────────────┐
│ Hello, World!                 │
└───────────────────────────────┘ 
```

Personally, I'm quite happy with this setup. But often people prefer not to depend on a server and run the code entirely in the browser. So I decided to look into it and implemented embeddable in-browser code playgrounds for JavaScript, Python, PHP, Ruby, Lua, and SQLite.

## Running language runtimes in the browser

The modern way to run arbitrary programs in the browser seems to be WebAssembly System Interface (WASI^() — an executable binary format based on WebAssembly. With WASI, you compile a program (originally written in C, Rust, Go, or some other language) into a WASI binary and then run it with a WASI runtime (there are a number of these runtimes from different vendors).)

Just as we can compile an arbitrary program into WASI binary, we can take a language interpreter like Lua or CPython, compile it into WASI, and run it with the WASI runtime to execute Lua or Python code. In practice, however, it's not that easy, because WASI compilers do not (yet) implement all the features of traditional compilers like GCC.

Fortunately, VMWare Labs has already done the hard part and compiled PHP^(, Python ^(and Ruby ^(into WASI. So all I had to do was publish the WASI binaries as NPM packages to make them available on the CDN. I've also compiled Lua ^(and SQLite ^(to WASI.)))))

> There is also Kohei Tokunaga's container2wasm initiative^(, which converts arbitrary Docker images into WASI binaries. It looks promising, but it generates 100+ MB binaries for even the smallest Alpine-based images. And since downloading hundreds of megabytes just to read an interactive article is probably not the best idea, this approach is not very practical (yet).)

Language runtimes compiled into WASI are one part of the equation. The other one is the WASI runtime (the thing that runs the binaries) capable of working in the browser. I chose the Runno ^(runtime by Ben Taylor because it's simple and lightweight (27 KB).)

The last step was to modify the JavaScript widget ^(to support pluggable engines (WASI is one of them).)

And that was it!

## Showcase

Here are some interactive code snippets implemented as described above. Note that the language runtime is downloaded when you click the Run button, so the first run may take some time. Subsequent runs are almost instantaneous.

### Python

Executes the code using the Python 3.12 WASI runtime (26.3 MB).

```
def greet(name):  print(f"Hello, {name}!")   greet("World") 
```

 <codapi-snippet engine="wasi" sandbox="python" editor="basic">### PHP

Executes the code using the PHP 8.2 WASI runtime (13.2 MB).

```
function greet($name) {  echo "Hello, $name!"; }   greet("World"); 
```

 <codapi-snippet engine="wasi" sandbox="php" editor="basic" template="main.php">### Ruby

Executes the code using the Ruby 3.2 WASI runtime (24.5 MB).

```
def greet(name)  puts "Hello, #{name}!" end   greet("World") 
```

 <codapi-snippet engine="wasi" sandbox="ruby" editor="basic">### Lua

Executes the code using the Lua 5.4 WASI runtime (330 KB).

```
function greet(name)  print("Hello, " .. name .. "!") end   greet("World") 
```

 <codapi-snippet engine="wasi" sandbox="lua" editor="basic">### JavaScript

Executes the code using the AsyncFunction^.

```
const greet = (name) => {  console.log(`Hello, ${name}!`); };   greet("World"); 
```

 <codapi-snippet engine="browser" sandbox="javascript" editor="basic">### Fetch

Executes the code using the Fetch API^.

```
POST https://httpbingo.org/dump/request content-type: application/json   { "message": "hello" } 
```

 <codapi-snippet engine="browser" sandbox="fetch" editor="basic">### SQLite

Executes the code using the SQLite 3.44 WASI runtime (2.1 MB).

```
select id, name, department from employees order by id limit 3; 
```

 <codapi-snippet engine="wasi" sandbox="sqlite" editor="basic" template="employees.sql">## Advanced features

Because the WASI runtime plugs into the existing architecture, WASI-powered code snippets support advanced Codapi features such as templates or code cells.

Templates ^(allow you to hide some code behind the scenes and show only the relevant part. For example, in the SQLite example above, the `employees` table is created as part of the template, so the snippet can take it for granted:)

```
select id, name, department from employees order by id limit 3; 
```

 <codapi-snippet engine="wasi" sandbox="sqlite" editor="basic" template="employees.sql">Code cells ^(allow you to make code snippets depend on each other. For example, the first snippet defines the `wrap` function, while the second snippet uses it:)

```
import textwrap   def wrap(text, width=20):  """Wraps the text so every line is at most width characters long.""" return textwrap.fill(text, width) 
```

 <codapi-snippet id="cell-1" engine="wasi" sandbox="python" editor="basic">```
text = (  "Python is a programming language that lets you work quickly " "and integrate systems more effectively." ) print(wrap(text)) 
```

 <codapi-snippet id="cell-2" engine="wasi" sandbox="python" editor="basic" depends-on="cell-1">## Usage

To use native browser playgrounds (e.g. JavaScript or Fetch), include the `snippet.js` script and add the `codapi-snippet` element next to the static code example. Use the `browser` engine:

```
<pre>console.log("hello")</pre>   <codapi-snippet engine="browser" sandbox="javascript" editor="basic"> </codapi-snippet>   <script src="https://unpkg.com/@antonz/codapi@0.12.0/dist/snippet.js"></script> 
```

To use WASI-powered playgrounds (e.g. Python or SQLite), include two additional scripts and use the `wasi` engine:

```
<pre>print("hello")</pre>   <codapi-snippet engine="wasi" sandbox="python" editor="basic"></codapi-snippet>   <script src="https://unpkg.com/@antonz/runno@0.6.1/dist/runno.js"></script> <script src="https://unpkg.com/@antonz/codapi@0.12.0/dist/engine/wasi.js"></script> <script src="https://unpkg.com/@antonz/codapi@0.12.0/dist/snippet.js"></script> 
```

To switch from in-browser to server-side playgrounds (which can run virtually any software), remove the `engine` attribute:

```
<pre> fn main() {  println!("Hello, World!"); } </pre>   <codapi-snippet sandbox="rust" editor="basic"></codapi-snippet> 
```

See the documentation ^(for details.)

## Summary

WASI-powered sandboxes allow code snippets to run completely in-browser, with no server involved. They may take some time and traffic to initialize the runtime, but after that they run almost instantly.

As implemented in Codapi, they fit nicely into the overall architecture, providing access to features like templates and code cells. You can also easily switch from a browser-side to a server-side execution model.

Give them a try!

[Playgrounds](https://github.com/nalgeon/codapi-js/blob/main/docs/browser-only.md) • [Snippet widget](https://github.com/nalgeon/codapi-js) • [About Codapi](https://codapi.org/)

*[* **Subscribe***](/subscribe/) *to keep up with new posts.**</codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet></codapi-snippet>