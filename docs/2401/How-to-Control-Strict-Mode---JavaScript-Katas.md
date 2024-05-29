<!--yml
category: 未分类
date: 2024-05-27 14:30:18
-->

# How to Control Strict Mode | JavaScript Katas

> 来源：[https://jskatas.org/blog/2023/10/16-control-strict-mode/](https://jskatas.org/blog/2023/10/16-control-strict-mode/)

I was surprised to see that the worker threads run in non-strict mode by default. This should rarely happen, because when using ES modules (ESMs), classes or `"use strict"` then the engine forces strict mode. I didn't know this in the beginning. I have been working on the katas I use for this site in the [javascript-katas repo](https://codeberg.org/wolframkriesing/javascript-katas) which uses ESMs and `type=module`, my default. This makes the entire nodejs project, any `npm ...` command run use strict mode. Escaping from strict mode is not easily possible, but one way to run code in non-strict mode is by using a worker thread.

I always like to understand where things originate, so let's read about strict mode.

## Strict Mode – When and Why?

Strict mode was introduced [in ES5 (in 2009)](https://262.ecma-international.org/5.1/#sec-4.2.2). In the specification it says strict mode was added

> in the interests of security, to avoid what [...] consider to be error-prone features, to get enhanced error checking

and

> The strict variant also specifies additional error conditions that must be reported by throwing error exceptions

So strict mode was introduced for making JS safer.

## Strict Mode?

Actually the worker I used was just a mean to solve my actual problem, which was running tests for the `arguments` kata. In order to do that I needed to run the tests in non-strict mode, so let's have a look at the strict mode.

Where does the strict mode play a role now? I had been diving deep into the "function API", digging in the very first [ES1 spec](https://www.ecma-international.org/wp-content/uploads/ECMA-262_1st_edition_june_1997.pdf) and all versions beyond to find out when was what introduced or deprecated (deprecation happens rarely in JavaScript). So I came across `arguments` and when I wrote the following test:

```
const fn = function() {
  const arguments = 'arg';
  return arguments;
}
assert.equal(fn('the argument'), 'arg'); 
```

It failed with "Uncaught SyntaxError: Unexpected eval or arguments in strict mode", which was not due to the test code itself, but the rest of the code I ran in the worker thread. The above test only got injected and makes only a small part of the entire code run in the worker. The code has an `import` in it, which turned all the code to be run in strict mode. And as the error says `arguments` is not allowed in strict mode.

So I want to control when to turn on strict mode. But for that let's understand strict mode a bit better first.

### What forces Strict Mode?

As mentioned above `"use strict"` turns on strict mode, but in this case there was none in the worker code yet, so what turned it on? In the [ES6 spec chapter "Strict Mode Code"](https://262.ecma-international.org/6.0/#sec-strict-mode-code) the following lines were newly added which force strict mode to turn on:

> Module code is always strict mode code.
> All parts of a ClassDeclaration or a ClassExpression are strict mode code.

So the `import` turned it on.

### How to Control Strict Mode?

So I rewrote the code to not use `import` and it ran in non-strict mode.

Now I can control strict mode, I just have to add `"use strict"` at the top of the code. Even better, I can put `"use strict"` at the beginning of a function body, and it makes only this function run in strict mode. It must be the first statement in this function, see the code below for some exploration. Oooh, I see new katas arising.

```
'use strict'
const fn = function() {
  const arguments = 'arg';
} 
```

The above fails with the error message we know from above "Unexpected eval or arguments in strict mode".

```
const fn = function() {
  'use strict'     // "use strict" inside the function, test still fails.
  const arguments = 'arg';
} 
```

The above fails too, for the same reason. Here only the function is run in strict mode, but that's also where `arguments` is used, so it fails.

```
const fn = function() {
  const arguments = 'arg';
}
'use strict'     // "use strict" NOT at the beginning of a function it gets ignored, test passes! 
```

Passes.

```
const fn = function() {
  if (true) {
    'use strict'     // "use strict" at the start of a scope, but NOT at the beginning of a function it gets ignored, test passes!
    const arguments = 'arg';
  }
} 
```

And this one passes too, because the `"use strict"` strings are not at the beginning of a function. I was irritated in the beginning, because I expected it to be at the beginning of a scope, so after the `{`, but I was wrong. The [spec also states](https://262.ecma-international.org/5.1/#sec-10.1.1) this explicitly:

> if the function code begins with a Directive Prologue that contains a Use Strict Directive.

in human words, this means the function must start with a `"use strict"` string. The scope is not mentioned there. The spec defines three more cases in very hard to read words. These are 1) the global code, 2) eval code and 3) `Function` code, when either starts with a `"use strict"` string, then the entire code is run in strict mode.

With this I did not only learn what strict mode restricts, but also how to control it. And all the magic is gone.

💡 **To turn on strict mode, you can use `"use strict"` in the four described ways, classes or modules. That's how strict mode can be (manually) turned or forced to be on.**