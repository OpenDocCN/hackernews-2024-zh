<!--yml
category: 未分类
date: 2024-05-27 14:49:23
-->

# Implementing scoped defer in C - by Sir Whinesalot

> 来源：[https://btmc.substack.com/p/implementing-scoped-defer-in-c](https://btmc.substack.com/p/implementing-scoped-defer-in-c)

*Note: Before reading this post, you should read my post on [implementing generators in C](https://btmc.substack.com/p/implementing-generators-yield-in) because I’ll be relying on similar techniques here.*

Error handling and resource management in C can be a total pain in the butt. Consider for example a program that needs to split a log file into three separate files for “info”, “warning” and “error” logs respectively:

```
FILE* log_file = fopen("log.txt", "r");
if (!log_file) {
  return -1;
}

FILE* info_log_file = fopen("info_log.txt", "w");
if (!info_log_file) {
  fclose(log_file);
  return -1;
}

FILE* warning_log_file = fopen("warning_log.txt", "w");
if (!warning_log_file) {
  fclose(info_log_file);
  fclose(log_file);
  return -1;
}

FILE* error_log_file = fopen("error_log.txt", "w");
if (!error_log_file) {
  fclose(warning_log_file);
  fclose(info_log_file);
  fclose(log_file);
  return -1;
}

// ...
// code to actually read and write the files goes here.
// if any operation fails we also need to "cleanup" there.
// ...

fclose(error_log_file);
fclose(warning_log_file);
fclose(info_log_file);
fclose(log_file);
return 0;
```

Copy-pasting those `fclose` calls around everywhere is not only annoying but extremely error prone. The above is the *simplest* example. Imagine we only want to create the various output files if one of the corresponding log type lines appears in the original file, or only create and write to one of the files if a certain configuration option is set. There might be memory management (malloc/free) happening in between all of this for whatever reason.

In this post I’m going to show you a really neat macro trick that makes dealing with *resource* management as pleasant as it can get in *standard* C.

Most programming languages provide a built-in mechanism for resource management.

C++ and Rust have [scope-based resource management](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization), where each type may have an associated “destructor” (called `drop` in rust) that is implicitly inserted by the compiler at the end of the scope where the variable storing the resource resides. When the scope ends resources are cleaned up, no matter if it happens “naturally”, due to an exception, or a return statement.

This solution is particularly nice because there is nothing for the consumer of the API to worry about, it just works, but it requires that object lifetimes be tied to scopes.

Languages like C#, Java and Python have special resource handling blocks: `using` statements in C#, `try-with-resource` statements in Java, and `with` statements in python, which implicitly call a cleanup function at the end of their scope. In C# it’s also possible to have `using` statements not introduce a new scope, tying the resource to the *current* scope instead.

Instead of calling the destructor, a specific method from the language’s “cleanup protocol” is called: `Dispose()` in C#, `close()` in Java and `__exit__()` in Python.

This solution has the disadvantage that the consumer of the API must remember to use the resource handling statement for cleanup to occur, but it allows regular objects to be managed in a non-scope oriented manner.

Languages like Go, Zig, Odin, and C3 support a `defer` statement that allows to specify the cleanup action, rather than implicitly invoking some standard cleanup protocol method or destructors:

```
// Go
log_file, err := os.Open("log.txt")
if err != nil {
  return nil, err
}
defer log_file.Close()

info_log_file, err := os.Open("info_log.txt", "w")
if err != nil {
  return err
}
defer info_log_file.Close()

warning_log_file, err := os.Open("warning_log.txt", "w")
if err != nil {
  return err
}
defer warning_log_file.Close()

error_log_file, err := os.Open("error_log.txt", "w")
if err != nil {
  return err
}
defer error_log_file.Close()

// ...

return nil
```

This solution has the disadvantage that the user of the API must not only remember to use defer, but also to know which specific function to call.

* * *

An important thing to note is that Go’s defer works differently from the other languages mentioned. The defer of the other languages is very simple: it can be understood as “copy-pasting” the cleanup code right before the scope ends (including if it is exited by a return statement).

Go’s defer is more of an imperative command. The block of code it will execute works like a closure, capturing the values of the referenced variables at the time of the defer statement’s execution (not at cleanup!). It also runs the cleanup code at function end rather than scope end, so it requires extra storage. For example, a defer statement executed in a loop 10 times will add 10 cleanup instructions to the “cleanup stack”.

Go’s style of defer is less efficient and its behavior can be surprising.

* * *

The advantage of defer is that it is fully explicit and gives the user the most control.

Fully explicit? Most control? Sounds like C to me. There are already some existing implementations of defer for C but I’m not happy with any of them:

*   The [reference implementation](https://gustedt.gitlabpages.inria.fr/defer/) of the [N2895 defer proposal](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2895.htm) is needlessly complicated, to the point of requiring an extra preprocessor. It relies on GCC extensions and setjmp + longjmp. It needs all this complexity to support features nobody asked for, IMHO. It is “Go style”.

*   ceraii’s [implementation](https://github.com/seleznevae/ceraii/blob/master/src/ceraii.h) is also quite complicated (though not to the same extent) and relies on setjmp + longjmp. It is “Go style”.

*   moon-chilled’s [Defer macro](https://github.com/moon-chilled/Defer/blob/master/defer.h) is the simplest but also requires either GCC extensions or setjmp + longjmp, plus a 32 element buffer at the start of each function to store the defers. It is “Go style”.

I don’t want any of this, I want a simple and efficient scoped defer like Zig, Odin, C3 and D have, but in pure standard C. I want the scoped defer of the [N3199 proposal](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3199.htm).

So until that gets accepted (i.e. never), it is time to get our hands dirty.

The classic solution to do resource management in C is to (ab)use goto statements to organize things a little:

```
`int result = 0;
FILE* log_file = fopen("log.txt", "r");
if (!log_file) {
  result = -1;
  goto DEFER_0;
}

FILE* info_log_file = fopen("info_log.txt", "w");
if (!info_log_file) {
  result = -1;
  goto DEFER_1;
}

FILE* warning_log_file = fopen("warning_log.txt", "w");
if (!warning_log_file) {
  result = -1;
  goto DEFER_2;
}

FILE* error_log_file = fopen("error_log.txt", "w");
if (!error_log_file) {
  result = -1;
  goto DEFER_3;
}

// ...

DEFER_4:
  fclose(error_log_file);
DEFER_3:
  fclose(warning_log_file);
DEFER_2:
  fclose(info_log_file);
DEFER_1:
  fclose(log_file);
DEFER_0:
  return result;`
```

We avoid the copy-paste problem, but this has its own issues. The resource cleanup has to be explicitly written in last-in-first-out order at the end of the function. Instead of using return, it’s now necessary to use goto to jump to the correct cleanup point. Things gets even more complicated when certain resources are allocated and released within inner scopes. This technique is also partially responsible for [goto fail](https://www.imperialviolet.org/2014/02/22/applebug.html).

Having only one resource cleanup call and the ability to jump around opens up some options. Lets reorganize the code a little:

```
`int result = 0;
FILE* log_file = fopen("log.txt", "r");
if (!log_file) {
  result = -1;
  goto DEFER_0;
}
if (0) {
  DEFER_1:
    fclose(log_file);
    goto DEFER_0;
}

FILE* info_log_file = fopen("info_log.txt", "w");
if (!info_log_file) {
  result = -1;
  goto DEFER_1;
}
if (0) {
  DEFER_2:
    fclose(info_log_file);
    goto DEFER_1;
}

FILE* warning_log_file = fopen("warning_log.txt", "w");
if (!warning_log_file) {
  result = -1;
  goto DEFER_2;
}
if (0) {
  DEFER_3:
    fclose(warning_log_file);
    goto DEFER_2;
}

FILE* error_log_file = fopen("error_log.txt", "w");
if (!error_log_file) {
  result = -1;
  goto DEFER_3;
}
if (0) {
  DEFER_4:
    fclose(error_log_file);
    goto DEFER_3;
}

goto DEFER_4;
DEFER_0:
  return result;`
```

Yikes that looks even worse! But this was an important change, now the cleanup code is right next to the initialization code, right where defer would be.

Lets try making some macros:

```
`#define CAT(A, B) A##B
#define defer_return(I, V) do {goto CAT(DEFER_, I)} while(0)
#define defer(I, F, P) if(0){CAT(DEFER_, I): F; goto CAT(DEFER_, P)}
#define run_deferred(I) goto CAT(DEFER_, I); DEFER_0: return result;

int result = 0;
FILE* log_file = fopen("log.txt", "r");
if (!log_file) {
  defer_return(0, -1);
}
defer(1, fclose(log_file), 0);

FILE* info_log_file = fopen("info_log.txt", "w");
if (!info_log_file) {
  defer_return(1, -1);
}
defer(2, fclose(info_log_file), 1);

FILE* warning_log_file = fopen("warning_log.txt", "w");
if (!warning_log_file) {
  defer_return(2, -1);
}
defer(3, fclose(warning_log_file), 2);

FILE* error_log_file = fopen("error_log.txt", "w");
if (!error_log_file) {
  defer_return(3, -1);
}
defer(4, fclose(error_log_file), 3);

run_deferred();`
```

Better, but not good enough. Having to manually count the defers like this is really annoying. We need to get fancier.

There’s no good way to “count” in the C preprocessor to avoid the issue above. The closest thing we have is the __LINE__ macro which stores the current line, but there’s no way to refer to the “last line” in which we deferred.

Rather than trying to do everything at the preprocessor level, we’ll shift some of the work to runtime. Let’s rework the example above to use a duff’s device-like switch statement instead of straight gotos:

```
`#define defer_return do {goto _d_start;} while(0)
#define defer(I, F) {_d=I; if(0){case I: _d--; F; goto _d_start;}}
#define defer_block int _d = -1; _d_start: switch(_d){case -1:
#define run_deferred() goto _d_start;}

int result = 0;
defer_block {
  FILE* log_file = fopen("log.txt", "r");
  if (!log_file) {
    result = -1; defer_return;
  }
  defer(1, fclose(log_file));

  FILE* info_log_file = fopen("info_log.txt", "w");
  if (!info_log_file) {
    result = -1; defer_return;
  }
  defer(2, fclose(info_log_file));

  FILE* warning_log_file = fopen("warning_log.txt", "w");
  if (!warning_log_file) {
    result = -1; defer_return;
  }
  defer(3, fclose(warning_log_file));

  FILE* error_log_file = fopen("error_log.txt", "w");
  if (!error_log_file) {
    result = -1; defer_return;
  }
  defer(4, fclose(error_log_file));
} run_deferred();
return result;`
```

Much better, now we only need 1 count in the defers. The basic idea is that each call to defer sets the defer count (`_d`) equal to I. It also adds an `if(0)` block that has the corresponding switch case label in its body. Inside the body it executes the action, decrements `_d`, and jumps to the switch again.

So once the first jump back to the switch occurs, the defer blocks get executed in reverse order, until `_d` becomes 0 which isn’t a valid case and the switch stops.

The syntax:

```
defer_block {...} run_deferred();
```

is modeled after:

```
do {...} while(...);
```

Which I personally find makes it quite easy to understand. But this is still not good enough! If a user writes the wrong value in the defer count, all hell breaks loose. We’re also missing inner scope support for proper scoped defer.

Rather than track the current defer by incrementing and decrementing, we’ll track where the defers occur by storing their `__LINE__`, and we’ll track the previous defer through an auxiliary variable called `_d_p`:

```
#define defer(F) {int _d_p = _d; \ 
  _d = __LINE__; if(0){case __LINE__: _d = _d_p; F; goto _d_start;}}
```

The `_d_p` (defer previous) variable is declared inside a new scope, such that it shadows any previous usage of the variable. It remembers the last value of `_d`, before `_d` is set to the current line. Now we can use the line as the case constant, avoiding explicit defer counting. Instead of decrementing, we set `_d` back to its previous value through `_d_p`. The last thing we need to ensure is that when the first `_d_p` is set, it points to “nowhere” so the switch ends, we do this by setting `_d` right after the switch starts to an impossible line:

```
#define defer_block int _d = 0; \ 
  _d_start: switch(_d){case 0: _d = -1;
```

We’ve now gotten a single level of defer working with no manual counting.

The last thing we want to support are nested scopes, where you can execute a subset of defers for a particular scope without running the whole thing. We also need `defer_return` to run every defer from the current scope all the way to the top level defer block.

The first thing to do is introduce a way to open a new defer scope. We’ll do this with the most complicated trick so far:

```
`#define _DEFER_SNAPSHOT(A) {int _d_p = _d; \ 
  _d = __LINE__; if(0){case __LINE__: _d = _d_p; A;}}
#define defer(F) _DEFER_SNAPSHOT(F; goto _d_start)
#define defer_scope
  do { _DEFER_SNAPSHOT(if(_d_r){goto _d_start;} else {continue;})`
```

We factor out the logic of defer into a `_DEFER_SNAPSHOT` macro which is shared with `defer_scope`. The idea is that `defer_scope` behaves like defer, in that it creates a “jump point”, but rather than executing an action, it makes a decision based on a new `_d_r` (defer return) variable:

*   If `_d_r` is true, it continues the “unwinding” process.

*   If `_d_r` is false, it jumps out of its `do {} while(0)` loop with `continue`.

This variable is set by the updated `defer_return`. We also add a `defer_break` alternative which is used to end only the current scope:

```
`#define defer_break { goto _d_start; }
#define defer_return { _d_r = 1; goto _d_start; }`
```

We make a small change to run_deferred, such that it works for both the top level defer block and the inner defer scopes:

```
`#define run_deferred() goto _d_start; } while (0)`
```

For the inner scopes, the `while(0)` ends the `do` loop. For the top level block, it’s just an extra `while(0)` that does nothing.

Lastly, we need to initialize the `_d_r` variable when we start the defer_block:

```
`#define defer_block int _d_r = 0; int _d = 0; \ 
  _d_start: switch(_d){case 0: _d = -1;`
```

And we’re done, scoped defer:

```
`int result = 0;
defer_block {
  FILE* log_file = fopen("log.txt", "r");
  if (!log_file) {
    result = -1; defer_return;
  }
  defer(fclose(log_file));

  defer_scope {
    FILE* info_log_file = fopen("info_log.txt", "w");
    if (!info_log_file) {
      result = -1; defer_return;
    }
    defer(fclose(info_log_file));

    FILE* warning_log_file = fopen("warning_log.txt", "w");
    if (!warning_log_file) {
      result = -1; defer_return;
    }
    defer(fclose(warning_log_file));

  } run_deferred();

  FILE* error_log_file = fopen("error_log.txt", "w");
  if (!error_log_file) {
    result = -1; defer_return;
  }
  defer(fclose(error_log_file));
} run_deferred();
return result;`
```

Unlike the little macro I used to implement generators in C, this one is a bit… much. But it’s also substantially simpler and more efficient than the longjmp based approaches, so if you want to have defer in C, try this out.

If you are ok with sticking to GCC (or clang), then [__attribute__((cleanup))](https://gcc.gnu.org/onlinedocs/gcc/Common-Variable-Attributes.html#index-cleanup-variable-attribute) is the way to go, it just works, even if it only allows calling a function with a specific prototype rather than an arbitrary block of code.