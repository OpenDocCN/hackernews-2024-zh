<!--yml
category: 未分类
date: 2024-05-27 14:41:10
-->

# Dynamic Printf Debugging with GDB – Abstract Expression

> 来源：[https://abstractexpr.com/2024/03/03/dynamic-printf-debugging-with-gdb/](https://abstractexpr.com/2024/03/03/dynamic-printf-debugging-with-gdb/)

When we debug a program with `printf()` we have to recompile it whenever we add new `printf()` statements! Right!?

Wrong!

With gdb we can add `printf()` statements without recompiling a program and even add them while it is running.

## Adding Printfs with GDB

Let’s say we have the following example program that computes the sum of all the elements of an array:

```
#include <stdio.h>

int main(int argc, char **argv)
{
    int numbers[] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    int n = sizeof(numbers) / sizeof(numbers[0]);
    int sum = 0;

    for (int i = 0; i < n; i++) {
        sum += numbers[i];
    }

    printf("Sum of array: %d\n", sum);

    return 0;
}

```

 Save this program to `sumarray.c` and compile it with the following command:

```
$ gcc -ggdb -o sumarray sumarray.c

```

When we run it the output will look like this:

```
$ ./sumarray
Sum of array: 55

```

What if we want to add a debug print before the for loop to print out the number of elements in the array, and a debug print inside the loop that prints the current loop index and the value of the current array element?

Since we already compiled our program with debug information we can just open it with gdb:

If you want to see the source code within gdb for orientation you can just use the list command. Or you can use the enable tui command to get a semi-graphical UI where you can see the source code all the time.

We add our first dynamic printf at line 8 to print out the total number of elements in the array:

```
(gdb) dprintf sumarray.c:8, "Num elements: %d\n", n

```

The first part of the syntax is like adding a breakpoint in gdb. After that, there is a comma and a C format string followed by the parameters to the format string. Of course, we can print multiple variables at once, as we see when adding the second dynamic print:

```
(gdb) dprintf sumarray.c:10, "Index: %d; value: %d\n", i, numbers[i]

```

The debugger has added a breakpoint to our program for each of our dynamic printfs. But instead of interrupting the program and dropping us into the debugger those breakpoints execute `printf()` with the given format string and parameters and continue the program.

We can now run our program from within gdb with:

The output should now look similar to that:

```
Num elements: 10
Index: 0; value: 1
Index: 1; value: 2
Index: 2; value: 3
Index: 3; value: 4
Index: 4; value: 5
Index: 5; value: 6
Index: 6; value: 7
Index: 7; value: 8
Index: 8; value: 9
Index: 9; value: 10
Sum of array: 55
[Inferior 1 (process 8368) exited normally]
(gdb)

```

If we use the `info break` command to list all breakpoints we will see that our dynamic printfs are indeed represented as breakpoints by gdb:

```
(gdb) info break
Num     Type           Disp Enb Address            What
1       dprintf        keep y   0x00005555555551df in main at sumarray.c:9
	breakpoint already hit 1 time
        printf "Num elements: %d\n", n
2       dprintf        keep y   0x00005555555551e8 in main at sumarray.c:10
	breakpoint already hit 10 times
        printf "Index: %d; value: %d\n", i, numbers[i]
(gdb)

```

To delete a dynamic printf you can use the `delete` command, followed by the number of the breakpoint (e.g. `delete 2` to delete the second dynamic print).

## Conditional Dynamic Printfs

When code runs very often (e.g. in a loop) there are situations where you don’t want your `printf()` calls to execute every time but only when a variable has a certain value or some other special condition is met. Otherwise, there would be so much output that it would be very hard to find the relevant lines in all of the irrelevant debug output.

When you add calls to `printf()` to your source code, you can just put the `printf()` inside an `if` block and your problem is solved.

With dprintf you first add your dynamic print. Then you make the resulting breakpoint conditional by using the gdb command `condition`.

In our last example, gdb created the breakpoint number 2 for the second dynamic printf statement (the one that prints the index and the value in the loop). If we wanted to make it print its output only for even loop indexes, we could achieve this with the following command:

```
(gdb) condition 2 (i % 2 == 0)

```

With this modification, the output of our program will now look like this in gdb:

```
(gdb) run
Num elements: 10
Index: 0; value: 1
Index: 2; value: 3
Index: 4; value: 5
Index: 6; value: 7
Index: 8; value: 9
Sum of array: 55
[Inferior 1 (process 5896) exited normally]
(gdb) 

```

## Saving and Loading Dynamic Printfs

Since dynamic printfs are breakpoints, we can save them like any other breakpoints with the *save breakpoints* command:

```
(gdb) save breakpoints [filename]

```

And to load them we can use the *source* command:

## Printing Discarded Function Return Values

Sometimes you want to print the return value of a function. This is easy to accomplish if the function’s return value is stored in a variable when it is called. But what if the return value is ignored by the code that calls it?

Let’s consider this program:

```
#include <stdio.h>

int func()
{
    printf("func called!");
    return 42;
}

int main(int argc, char **argv)
{
    func();

    return 0;
}

```

Here the function `func()` is called and it returns an int value. But the code that calls it ignores this fact and discards the return value.

Save it to a file named `return.c` and compile it with:

```
$ gcc -ggdb -o return return.c

```

If we are on an x86-64 machine and we know that the calling convention for this architecture mandates that return values (of integer and pointer types) are returned in the register `RAX`, this problem is easy to solve. We just put a `dprintf` in the line after the function call and print the value of the `RAX` register:

```
(gdb) dprintf return.c:12, "return value: %d\n", $rax

```

On ARM64 (aka AArch64) we have to print the `x0` register instead:

```
(gdb) dprintf return.c:12, "return value: %d\n", $x0

```

When we run the program with

the output will look similar to this:

```
return value: 42
func called![Inferior 1 (process 36549) exited normally
```

## Custom Printf Function

Instead of using `printf()` to output our debug messages gdb also allows us to configure our own custom function to be called for every `dprintf` we add to our code.

Such a custom function might look like this:

```
int log_msg(const char *format, ...)
{
    int n;
    va_list args;
    // init the variable length argument list
    va_start(args, format);

    n = vfprintf(logfile, format, args);

    // cleanup the variable length argument list
    va_end(args);

    return n;
}

```

This function returns an `int` and expects a format string and a variable number of additional arguments. Therefore, it has the same prototype as `printf()`.

It first initializes the variable argument list and then calls `vfprintf()` (this function is a variant of `fprintf()` that accepts the variable argument list in the form of a `va_list` variable) with a `FILE` pointer stored in a global variable, the format string, and the variable argument list.

To use this function we need to open the logfile at the start of the main function. We must not put a `dprintf` at any point in the program before the file has been opened or we get a segmentation fault due to the global variable `logfile` still being `NULL`.

The full code of our modified example program looks like this:

```
#include <stdio.h>
#include <stdarg.h>

FILE *logfile = NULL;

int log_msg(const char *format, ...)
{
    int n;
    va_list args;
    // init the variable length argument list
    va_start(args, format);

    n = vfprintf(logfile, format, args);

    // cleanup the variable length argument list
    va_end(args);

    return n;
}

int main(int argc, char **argv)
{
    int numbers[] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    int n = sizeof(numbers) / sizeof(numbers[0]);
    int sum = 0;

    const char *filename = "logfile.txt";
    logfile = fopen(filename, "w");
    if (logfile == NULL) {
        fprintf(stderr, "Failed to open logfile: %s\n", filename);
        return 1;
    }

    for (int i = 0; i < n; i++) {
        sum += numbers[i];
    }

    printf("Sum of array: %d\n", sum);
    fclose(logfile);

    return 0;
}

```

Let’s save this to a file named `custom_printf.c` and compile it with:

```
$ gcc -ggdb -o custom_printf custom_printf.c

```

After loading the resulting `custom_printf` program with gdb we first have to set the calling style for `dprintf` to `call` (otherwise, gdb uses a builtin printf instead of looking it up in our program), and set the function used by `dprintf` to our `log_msg` function:

```
(gdb) set dprintf-style call
(gdb) set dprintf-function log_msg

```

Now we can set our dprintfs with new line numbers:

```
(gdb) dprintf custom_printf.c:33, "Num elements: %d\n", n
(gdb) dprintf custom_printf.c:35, "Index: %d; value: %d\n", i, numbers[i]

```

Once you execute the program with

the output of the `dprintfs` should now appear in the file `logfile.txt` instead of on the screen.

## Conclusion

The dynamic printf feature of gdb is a very powerful alternative to adding throw-away `printf()` statements to your code while debugging. You can add new dynamic printfs without recompiling your program, and you can even add additional print statements while your program is running.

The last advantage is especially useful when debugging long-running programs like servers. Imagine you add some initial dynamic printfs. Then you start your server and send a request. As you learn more and dig deeper into the code you add more dynamic printfs from the comfort of your debugger and start a new request every time. Once you analyze the output you add more dynamic printfs until you find your bug.

Add to this the feature to save your dynamic printfs to a file for later debugging sessions, as well as the ability to print discarded function return values and you will see why gdb pushes printf debugging to the next level.