<!--yml
category: 未分类
date: 2024-05-27 12:53:53
-->

# Xr0 – C But Safe

> 来源：[https://xr0.dev/](https://xr0.dev/)

Xr0 is a verifier for C. It eliminates many stubborn instances of undefined behaviour, like use-after-frees, double frees, null pointer dereferences and the use of uninitialised memory.

Xr0 uses C-like annotations to verify code:

```
void *
alloc() ~ [ return malloc(1); ] /* caller must free */
{
        return malloc(1);
} 
```

<details><summary>More about the annotations</summary>

They’re attached to every function that is potentially unsafe and express what its callers need to know to use it safely:

```
void *
alloc_if(int x) ~ [ if (x) return malloc(1); ] /* caller must free if x != 0 */
{
        if (x) {
                return malloc(1);
        } else {
                return NULL;
        }
} 
```

The really subtle safety bugs creep in through layers of function calls. Xr0 makes this impossible, because everything needed to secure safety is distributed through every function call, so that no subtle mistake can creep in. It “quantum entangles” the safety semantics of every part of the program with every other part. Think of it like a infinitely rich type system that rises to the demands of your program’s structure. You still have to make the code safe; Xr0 just checks your work. Thus Xr0 is magical like the wand, not the magician. The real magic comes from the programmer.</details> 

Xr0 is a work in progress and currently verifies a subset of C89. Its most significant limitation is we haven’t yet implemented verification for loops and recursive functions, so these are being bridged by axiomatic annotations. Xr0 1.0.0 will enable programming in C with no undefined behaviour, but for now it’s useful for verifying sections of programs.

Xr0 is written in pure C and is open source. View it on [GitHub](https://github.com/xr0-org/xr0) or [SourceHut](https://git.sr.ht/~lbnz/xr0).

## Intrigued?

The best way to understand Xr0 is to [try it](/try). If you want to see how Xr0 works, be sure to use the debugger, which you can learn about [here](/debug).

Read the [tutorial](/learn) to learn more, and then if you want to go deeper, engage with [our theses](/theses), which explain how Xr0 will make C safe, and take a look at [our vision and roadmap](/vision-roadmap).

Come and talk to us on [Discord](https://discord.gg/yfx69tbhxQ), or via email [here](mailto:betz@xr0.dev) and [here](mailto:a@xr0.dev).