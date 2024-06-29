<!--yml

category: 未分类

date: 2024-05-27 12:53:53

-->

# Xr0 – C But Safe

> 来源：[https://xr0.dev/](https://xr0.dev/)

Xr0 是 C 语言的验证器。它消除了许多顽固的未定义行为实例，如释放后使用、双重释放、空指针解引用和未初始化内存的使用。

Xr0 使用类似 C 的注解来验证代码：

```
void *
alloc() ~ [ return malloc(1); ] /* caller must free */
{
        return malloc(1);
} 
```

<details><summary>关于注解的更多信息</summary>

它们被附加到每个可能不安全的函数上，并表达其调用者需要安全使用它所需的内容：

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

完全微妙的安全性 bug 会潜伏在多层函数调用中。Xr0 使这种情况不可能发生，因为确保安全所需的一切都分布在每个函数调用中，这样任何微妙的错误都无法潜入。它将程序的每个部分的安全语义与其他部分量子纠缠在一起。想象它就像一个无穷丰富的类型系统，能够适应程序结构的需求。你仍然需要使代码安全；Xr0 只是检查你的工作。因此，Xr0 就像魔杖一样神奇，而不是魔术师。真正的魔法来自程序员。</details>

Xr0 目前正在开发中，当前只能验证 C89 的一个子集。它最显著的限制是尚未实现对循环和递归函数的验证，因此这些部分正在通过公理注解进行桥接。Xr0 1.0.0 将能够在 C 中进行无未定义行为的编程，但目前对于验证程序的部分区域非常有用。

Xr0 是用纯 C 语言编写的开源项目。你可以在 [GitHub](https://github.com/xr0-org/xr0) 或者 [SourceHut](https://git.sr.ht/~lbnz/xr0) 上查看它。

## 你感兴趣吗？

要了解 Xr0 最好的方法是[尝试一下](/try)。如果你想看看 Xr0 的工作原理，请务必使用调试器，你可以在[这里](/debug)了解更多。

阅读 [教程](/learn) 以了解更多信息，然后如果你想深入了解，请参阅[我们的论文](/theses)，这些论文解释了 Xr0 如何使 C 语言变得更安全，并查看[我们的愿景和路线图](/vision-roadmap)。

欢迎加入我们的 [Discord](https://discord.gg/yfx69tbhxQ)，或通过电子邮件 [这里](mailto:betz@xr0.dev) 和 [这里](mailto:a@xr0.dev) 与我们联系。
