<!--yml

类别：未分类

日期：2024-05-27 14:29:51

-->

# r300：对顶点着色器进行更多 NIR 降级（!26816）· 合并请求 · Mesa / mesa · GitLab

> 来源：[https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/26816](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/26816)

这个 MR 将大部分剩余的后端降级移到了 NIR 中。具体来说，ftrunc、fcsel（适用时）和 flrp。已删除了后端降级路径。这是更多后端清理的先决条件，例如我已经准备好了一个 MR 来消除顶点着色器的后端 DCE。

老实说，我已经尝试了五次才做到这一点，最初甚至在 nir_to_rc 之前，我总是失败了。之所以比通常的降级更棘手的原因是所需的传递顺序。事实上，我们降级的所有操作码都来自整数降级（ftrunc）或布尔降级。我们不能在 finalize nir 中的主循环中进行布尔降级（特别是因为 peephole select 需要布尔值进行条件判断），虽然早期整数降级在测试中是可以的，并且似乎可以工作，但 Emma 之前曾对此表示反对。因此，所有这些都是在转换到后端 IR 时发生的。出于同样的原因，有一些自定义代数传递执行相同的降级，我们通常会使用通用的 nir_opt_algebraic。在理想的情况下，我们应该在晚期代数之前进行降级，因此如果我们将 fcsels 一直降级到 add+muls，我们可以最佳地生成 ffmas，但我无法做到这一点，所以当我们降低来自 fcsels 的 flrps 时，我们直接发出 ffmas...

在 R500 上对 NIR 进行 fcsel 降级（我们有一个本地的 CMP，但顶点引擎有一个限制，即它不能从三个不同的 TMP 寄存器中读取）也不幸比在后端中进行要复杂得多，需要查看 fneg 和 fabs，还要检查常量和输入是否会被复制传播。最终这是大量的代码，因此总体上这个系列大部分是平衡的（NIR fcsel 降级 vs TRUNC、LRP 和 CMP 后端删除），但为将来进行更多清理铺平了道路。

Shader-db-wise 这对于 R500 稍微有所胜利（通过在 NIR 中执行 fcsel 降级，我们仍然拥有信息，即这是 fcsel_lt/ge 还是 fcsel，如果必须降级，则可以保存比较），在 R300 上也大多是平衡的，详细统计请参见单个提交。
