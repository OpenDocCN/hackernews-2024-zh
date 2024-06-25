<!--yml

类别：未分类

日期：2024-05-27 14:29:17

-->

# 西部最快的自动微分

> 来源：[`arogozhnikov.github.io/2023/12/28/fastest-autograd.html`](https://arogozhnikov.github.io/2023/12/28/fastest-autograd.html)

谁需要快速自动微分？看起来现在每个人都需要！

从前，我需要一个**实际快速**的自动微分。抛开项目细节，以下是要求：

+   我们测试许多计算图（图形不断变化）

+   每个图中有大量数量的标量操作，大约有**10k—100k 个节点**

+   每个图形应该在前向和后向方向上编译和运行约**10k 次**

+   这应该**非常快**，并且具有方便的 python 接口

我们前方的道路：

1.  torch 中的自动微分

1.  jax 中的自动微分

1.  python 中的自动微分

1.  rust 中的自动微分

1.  C 中的自动微分

1.  汇编中的自动微分

再加上相当数量的草率代码和 M1 macbook 上的定时。

### 让我们在 pytorch 中使用自动微分

我们从 pytorch 开始我们的旅程——这是研究中默认的自动微分引擎。我们将创建一个具有许多节点的图形，为了保持简单，我们的基准测试只有几种操作：一元（softplus），二元（乘法），n 元（求和）和 n 到 n（softmax）。

这使得只使用几个操作，但类似于真实负载。本文中的所有基准测试都将重新实现与下文相同的逻辑。

```
def run_graph(initial_variables, n_operations: int):
    nodes = [*initial_variables]

    for op in range(n_operations):
        match op % 4:
            case 0:
                # softplus
                nodes.append(F.softplus(nodes[-10]))
            case 1:
                # sum
                nodes.append(sum(nodes[-30:-10:5]))
            case 2:
                # prod
                nodes.append(nodes[-20] * nodes[-10])
            case 3:
                # softmax
                softmaxes = F.softmax(torch.stack(nodes[-4:], dim=0), dim=0)
                nodes.extend(softmaxes)

    return nodes

def run_benchmark_pytorch(n_iterations, n_operations):
    init_vars = torch.arange(100, dtype=torch.float32, requires_grad=True)
    for _ in range(n_iterations):
        nodes = run_graph(
            initial_variables=init_vars,
            n_operations=n_operations,
        )
        nodes[-1].backward() 
```

运行 10k 个操作 x 100 次迭代的运行时间：11.3 秒

运行 10k 个操作 x 10k 次迭代的运行时间：**1130 秒**（估计）

鉴于我们创建了 100M 个 python 对象，这实际上是相当快的。是的，这不会提供交互式体验。

让我们也讨论一下`torch.compile`，这是 pytorch 2.0 中的一个重大创新。

对于 100 次操作，torch.compile 需要 4.5 秒。执行变得更快：对于 100 次操作和 10k 次迭代，使用 torch.compile 需要 4.52 秒，而不使用需要 10.4 秒。编译 + 执行仍在同一个范围内。对于更大的图形（1k 操作），`torch.compile` 会崩溃。

### 让我们在 jax 中使用自动微分

Jax 是新的潮流... 嗯，不那么新了。但在某些方面，它非常有趣。Jax 关注即时编译静态图，非常适合手头的问题。

基准测试的实现类似于 pytorch：

```
import jax
import numpy as np

def run_graph_jax(initial_variables):
    nodes = [*initial_variables]
    for op in range(n_operations):
        match op % 4:
            case 0:
                # softplus
                nodes.append(jax.nn.softplus(nodes[-10]))
            case 1: 
                # sum
                nodes.append(sum(nodes[-30:-10:5]))
            case 2: 
                # prod 
                nodes.append(nodes[-20] * nodes[-10])
            case 3: 
                # softmax
                softmaxes = jax.nn.softmax(jax.numpy.stack(nodes[-4:]), axis=0)
                nodes.extend(softmaxes)

    return nodes[-1]

run_graph_and_grad = jax.value_and_grad(run_graph_jax)
# or run_graph_and_grad = jax.jit(jax.value_and_grad(run_graph_jax)) 
```

没有即时编译计算速度极慢：

1k 个操作 x 10 次迭代 => 15.9 秒

10k 个操作 x 10k 次迭代 => 159,000 秒（估计）

这比永远都要长一点！但是 jax 的整个重点是即时编译。所以让我们开始吧。

即时编译：1k 个操作的编译 = 47 秒

即时编译：运行 1k 个操作 x 10k 次迭代 = 0.66 秒

即时编译：10k 个操作 x 10k 次迭代（编译 + 运行时间）=> **470 秒**（估计）

执行时间的加速超乎想象，但我们花费了 >99% 的时间在编译上。

#### Tensorflow

有人总会提及 TF。我将把这留给 TF 的粉丝作为练习。

### 让我们在 python 中使用自动微分

基准测试完成，是时候看看我们是否可以加速了。

让我们创建一个简单的伪框架，并看看它与之前的候选者相比如何。我们将实现一个类似磁带的自动求导，其中操作顺序在磁带中被明确跟踪。

<details><summary class="code-summary">展示纯 Python 中的自动求导引擎</summary>

```
class NaiveVar:
    def __init__(self, val):
        self.val = val
        self.grad = 0.

class NaiveTape:
    def __init__(self, input_values):
        self.ops = []

    def sum(self, *vars):
        res = NaiveVar(sum(v.val for v in vars))
        self.ops.append(('sum', vars, res))
        return res

    def prod(self, var1, var2):
        res = NaiveVar(var1.val * var2.val)
        self.ops.append(('prod', [var1, var2], res))
        return res

    def softmax(self, *vars):
        vals = [v.val for v in vars]
        maxval = max(vals)
        vals = [v - maxval for v in vals]
        denom = sum(math.exp(v) for v in vals)
        res = [NaiveVar(math.exp(v) / denom) for v in vals]
        self.ops.append(('softmax', vars, denom))
        return res

    def softplus(self, var):
        res = NaiveVar(math.log1p(math.exp(var.val)))
        self.ops.append(('splus', var, res))
        return res

    def backward(self, var):
        assert var.grad == 0
        var.grad += 1
        for op, inputs, outputs in self.ops[::-1]:
            match op:
                case 'sum':
                    out = outputs
                    for v in inputs:
                        v.grad += out.grad
                case 'prod':
                    out = outputs
                    in1, in2 = inputs
                    in1.grad += in2.val * out.grad
                    in2.grad += in1.val * out.grad
                case 'splus':
                    inputs.grad += out.grad / (1 + math.exp(-inputs.val))
                case 'softmax':
                    pass # skip for now
                case _:
                    raise NotImplementedError() 
```</details>

并使用我们的新伪框架重新实现参考任务：

<details><summary class="code-summary">展示基准测试代码</summary>

```
def run_graph_python_and_backward(initial_variables, n_operations):
    nodes = [NaiveVar(x) for x in initial_variables]
    tape = NaiveTape(nodes)
    for op in range(n_operations):
        match op % 4:
            case 0: 
                # softplus
                nodes.append(tape.softplus(nodes[-10]))
            case 1: 
                # sum
                nodes.append(tape.sum(*nodes[-30:-10:5]))
            case 2: 
                # prod 
                nodes.append(tape.prod(nodes[-20], nodes[-10]))
            case 3: 
                # softmax
                nodes.extend(tape.softmax(*nodes[-4:]))

    tape.backward(nodes[-1])
    return tape 
```</details>

10k 次操作和 10k 次迭代的运行时间：**312 秒**。

预料之中不快。但与之前的候选相比，这实际上是相当有竞争力的！

### 让我们再次在 Python 中进行自动求导。

这次我们将所有值都移动到磁带中，而不是保留在变量中。此外，磁带将通过记录每次操作中参与的变量的索引来保留计算的‘静态图’。

<details><summary class="code-summary">展示纯 Python 中自动求导的代码</summary>

```
import numba
import math

class VarInd:
    def __init__(self, index):
        self.index = index # variable is just a unique index in tape

class TapeInd:
    def __init__(self):
        self.ops = []
        self.vals = []  # flat memory with values
        self.grads = [] # flat memory with gradients 
    def make_var(self, value):
        self.vals.append(value)
        self.grads.append(0.)
        return VarInd(len(self.vals) - 1)

    def val(self, v: VarInd):
        return self.vals[v.index]

    def add_op(self, kls, input_vars, output_vars):
	    # translate variable to indices. self.ops keeps only indices
        self.ops.append((kls, [x.index for x in input_vars], [x.index for x in output_vars]))        

    def sum(self, *vars):
        res = self.make_var(sum(self.val(v) for v in vars))
        self.add_op('sum', vars, [res])
        return res

    def prod(self, var1, var2):
        res = self.make_var(self.val(var1) * self.val(var2))
        self.add_op('prod', [var1, var2], [res])
        return res

    def softmax(self, *vars):
        vals = [self.val(v) for v in vars]
        maxval = max(vals)
        vals = [v - maxval for v in vals]
        denom = sum(math.exp(v) for v in vals)
        res = [self.make_var(math.exp(v) / denom ) for v in vals]
        self.add_op('softmax', vars, res)
        return res

    def softplus(self, var):
        res = self.make_var(math.log1p( math.exp(self.val(var)) ))
        self.add_op('splus', [var], [res])
        return res

    def forward_backward_external(self, grad_var: VarInd):
        return forward_backward_optimal(self.vals, self.grads, self.ops, grad_var_index=grad_var.index)

def forward_backward_external(
	vals: list[float], 
	grads: list[float], 
	ops: list[tuple[str, list[int], list[int]]],
	grad_var_index: int
):
    v: list[float] = vals
    g: list[float] = grads
    # forward pass
    for op, ins, outs in ops:
        match op:
            case 'sum':
                v[outs[0]] = sum(v[i] for i in ins)
            case 'prod':
                v[outs[0]] = v[ins[0]] * v[ins[1]]
            case 'splus':
                v[outs[0]] = math.log1p(math.exp( v[ins[0]] ))
            case 'softmax':
                maximal = max(v[i] for i in ins)
                exps = [math.exp(v[i] - maximal) for i in ins]
                denom = sum(outs)
                for i, exp in zip(outs, exps):
                    v[i] = exp / denom

    g[grad_var_index] += 1

	# backward pass
    for op, ins, outs in ops[::-1]:
        match op:
            case 'sum':
                for i in ins:
                    g[i] += g[outs[0]]
            case 'prod':
                out: int = outs[0]
                in1, in2 = ins
                g[in1] += v[in2] * g[out]
                g[in2] += v[in1] * g[out]
            case 'splus':
                g[ins[0]] += g[outs[0]] / (1 + math.exp(-v[ins[0]]))
            case 'softmax':
				avg_grad = sum(v[j] * g[j] for j in outs)
				for i, j in zip(ins, outs):
					g[i] += v[j] * (g[j] - avg_grad) 
```

和相应的启动代码

```
def run_graph_python_and_backward(n_operations, n_iterations):
    tape = TapeInd()
    nodes = [tape.make_var(float(x)) for x in range(100)]

    for op in range(n_operations):
        match op % 4:
            case 0: 
                # softplus
                nodes.append(tape.softplus(nodes[-10]))
            case 1: 
                # sum
                nodes.append(tape.sum(*nodes[-30:-10:5]))
            case 2: 
                # prod 
                nodes.append(tape.prod(nodes[-20], nodes[-10]))
            case 3: 
                # softmax
                softmaxes = tape.softmax(*nodes[-4:])
                nodes.extend(softmaxes)

    for _ in range(n_iterations):
        tape.forward_backward(nodes[-1]) 
```</details>

10k 次操作 x 10k 次迭代的运行时间：**94 秒**

如我们所见，将所有值移动到磁带中并切换到操作索引的策略相当有效。我们仍然使用 Python，但现在比 `pytorch` 或 `jax` 快约 5-10 倍。

在这一点上，我想提到另一个实验：上面的代码是为了使其友好于 `numba` 而组织的。[Numba](https://numba.readthedocs.io/en/stable/) 以提供即时编译，通过提供即时编译，以最小的更改加快 Python 中的数值计算而闻名。最近添加的 `numba.typed.List` 使得可以有效地处理列表的列表。

使用 numba 的运行时间，10k 次操作 x 10k 次迭代：**41 秒**。

在这一点上，我们比 jax/pytorch 快了超过 10 倍（仍然在 Python 中编写代码）。

### 让我们在 Rust 中进行自动求导。

一旦我们将图跟踪移动到磁带中，我们现在可以使用一些快速的方法来运行计算。例如，Rust。对于 Rust ↔ Python 的互操作性，我使用了一个围绕 [rustimport](https://github.com/mityax/rustimport) 的小包装器。`Rustimport` 允许方便地“导入”一个单独的 Rust 文件，而不创建一个完整的 Rust 项目。

一些优化说明：

+   `softmax` 成为了一个瓶颈，所以我转而在堆栈上创建临时数组，而不是 Vecs，这需要对输入大小进行特化。

+   我遵循了 Rust 风格的迭代器方法来减少边界检查的数量。

+   我想知道一次检查多个选项的匹配是否慢。在合成测试中，它似乎相对快速，但我希望在这里实现跳转表优化（例如，对于 Rust 中的 [enums](https://users.rust-lang.org/t/match-statement-efficiency/4488) 支持此优化，并且 clang [在 C 中使用](https://stackoverflow.com/questions/60109992/why-is-a-switch-not-optimized-the-same-way-as-chained-if-else-in-c-c) 此优化来进行 switch-case）。

<details><summary class="code-summary">展示最小自动求导的 Rust 代码</summary>

```
// rustimport:pyo3
use pyo3::prelude::*;

// slower softmax version for larger number of inputs
fn softmax_varlength(vals: &mut Vec<f32>, ins: &[usize], outs: &[usize]) {
    let mut max = -1e20_f32;
    let loc_vals: Vec<f32> = ins.into_iter().map(|i| { let x = vals[*i]; max = max.max(x); x} ).collect();
    let mut sum: f32 = 0.0_f32;
    let exps: Vec<f32> = loc_vals.iter().map(|v| {let _exp = f32::exp(*v - max); sum += _exp; _exp}).collect();
    outs.iter().zip(exps.iter()).for_each(|(j, exp)| vals[*j] = exp / sum );
}

// vecs are slow! so allocate slices on stack, and explicit grouping of computations also helps
fn softmax<const N: usize>(vals: &mut Vec<f32>, ins: &[usize], outs: &[usize]) {
    let mut loc_vals: [f32; N] = [0_f32; N];
    let mut exps: [f32; N] = [0_f32; N];
    let mut max = -1e20_f32;
    let mut sum: f32 = 0.;
    for (n, i) in ins.into_iter().enumerate() {
        let v = vals[*i];
        loc_vals[n] = v;
        max = max.max(v);
    }
    for (n, _i) in ins.into_iter().enumerate() {
        let exp = f32::exp(loc_vals[n] - max);
        exps[n] = exp;
        sum += exp;
    }
    let invsum = 1.0_f32 / sum;
    for (n, j) in outs.into_iter().enumerate() {
        vals[*j] = exps[n] * invsum;
    }
}

fn sigmoid(x: f32) -> f32 {
    1.0 / (1.0 + (-x).exp())
}

#[pyfunction]
unsafe fn autograd(
    vals_input: Vec<f32>,
    ops: Vec<i32>,
    input_ids: Vec<Vec<usize>>, 
    output_ids: Vec<Vec<usize>>,
    backward_node_id: usize,
    n_iteration: i32,
) -> (Vec<f32>, Vec<f32>) {
    let mut vals: Vec<f32> = vals_input.iter().map(|x| *x).collect();
    let mut grad: Vec<f32> = vals_input.into_iter().map(|_| 0.0_f32).collect();

    for _ in 0..n_iteration {
        for (i_op, op) in ops.iter().enumerate(){
            let ins: &Vec<usize> = &input_ids[i_op];
            let outs: &Vec<usize> = &output_ids[i_op];

            match op {
                0 => {
                    // softplus
                    let x = vals[ins[0]];
                    let max = f32::max(0., x);
                    let min = f32::min(0., x);
                    vals[outs[0]] = max + f32::ln_1p(f32::exp(min - max));
                }
                1 => {
                    // sum
                    vals[outs[0]] = ins.iter().map(|i| vals.get_unchecked(*i)).sum();
                }
                2 => {
                    // prod
                    vals[outs[0]] = vals[ins[0]] * vals[ins[1]];
                }
                3 => {
                    // softmax. we will need switch-case resolution here for most common cases
                    match ins.len() {
                        1 => {softmax::<1>(&mut vals, &ins, &outs)}
                        2 => {softmax::<2>(&mut vals, &ins, &outs)}
                        3 => {softmax::<3>(&mut vals, &ins, &outs)}
                        4 => {softmax::<4>(&mut vals, &ins, &outs)}
                        5 => {softmax::<5>(&mut vals, &ins, &outs)}
                        _ => {softmax_varlength(&mut vals, &ins, &outs)}
                    }
                }
                _ => { panic!(""); }
           }
        }
        grad[backward_node_id] = 1.;

        for (i_op, op) in ops.iter().enumerate(){
            let ins: &Vec<usize> = &input_ids[i_op];
            let outs: &Vec<usize> = &output_ids[i_op];

            match op {
                0 => {
                    // softplus
                    grad[ins[0]] += grad[outs[0]] * sigmoid(vals[ins[0]]);
                }
                1 => {
                    // sum
                    ins.iter().for_each(|i| grad[*i] += grad[outs[0]]);
                }
                2 => {
                    // prod
                    grad[ins[0]] += grad[outs[0]] * vals[ins[1]];
                    grad[ins[1]] += grad[outs[0]] * vals[ins[0]];
                }
                3 => {
	                // softmax
                    let avg_grad: f32 = outs.iter().map(|j| grad[*j] * vals[*j] ).sum();
                    for (i, j) in ins.iter().zip(outs.iter()) {
                        grad[*i] += vals[*j] * (grad[*j] - avg_grad);
                    }
                }
                _ => { panic!(""); }
           }
        }        
    }
    (vals, grad)
} 
```</details>

10k 次操作 x 10k 次迭代的运行时间：**1.4 秒**

成功：我们处于交互式体验的领域。

记得我们开始时超过了 1000 秒。但是我们应该在这里停下来吗？

### 让我们在 C 中进行自动微分。

在 C 中实现自动微分逻辑的时间到了。为了与 Python 进行交互，我使用 [python-cffi](https://cffi.readthedocs.io/en/stable/index.html)。

我对优化进行了疯狂的尝试：

+   我利用了输出节点在内存中是连续放置的事实，因此我们只传递第一个输出的索引。

+   输入数量限制为 8，并且这些输入作为 `int[8]` 嵌入到结构体中，而不是 `int *`，以避免内存跳转

+   动态堆栈分配可变大小（与 Rust 相比，在 C 中这些很简单）

+   `-O3`，以及不安全的数学运算：`-ffast-math`。甚至尝试了内存对齐和限制指针，但没有运气

<details><summary class="code-summary">给我看一些 C 代码</summary>

```
#include <math.h>  
typedef struct { 
    int opcode;
    size_t n_arguments; // used for softmax and sum
    int ins[8];         // at most 8 inputs
    int out;            // points to the first output variable
} MyOperation;

MyOperation * allocate_memory(int n_elements) {
    return (MyOperation *) malloc(sizeof(MyOperation) * n_elements);
}

// stable implementation
double logaddexp(double x, double y) {
    if (x > y) { return x + log1p(exp(y - x)); }
    else       { return y + log1p(exp(x - y)); }
}

double sigmoid(double x) { return 1.0 / (1.0 + exp(-x)); }

void run_multiple_passes(
    int n_operations,
    MyOperation *ops,
    double *values,
    double *grads,
    int n_iterations
) {
    for(int iteration = 0; iteration < n_iterations; iteration++) {
        for(int operation = 0; operation < n_operations; operation++) {
            MyOperation op = ops[operation];
            switch(op.opcode) {
                case 1: 
                    values[op.out] = logaddexp(0., values[op.ins[0]]);
                    break;
                case 2: 
                    {
                        double out = 0.;
                        for(size_t i=0; i < op.n_arguments; i++) {
                            out += values[op.ins[i]];
                        }
                        values[op.out] = out;
                    }
                    break;
                case 3:
                    values[op.out] = values[op.ins[0]] * values[op.ins[1]];
                    break;
                case 4:
                    {
                        double maximal = -1e20;
                        size_t n_arg = (size_t) op.n_arguments;
                        for(size_t i = 0; i < n_arg; i++) {
                            maximal = fmax(maximal, values[op.ins[i]]);
                        }
                        double exps[n_arg];
                        double sum = 0;
                        for(size_t i = 0; i < n_arg; i++) {
                            exps[i] = exp(op.ins[i] - maximal);
                            sum += exps[i];
                        }
                        for(size_t i = 0; i < n_arg; i++) {
                            values[op.out + i] = exps[i] / sum;
                        }
                    }
                    break;
            }
        }  // end forward

        // TODO set grad for target variable.

        for(int operation = 0; operation < n_operations; operation++) {
            MyOperation op = ops[n_operations - 1 - operation];
            switch(op.opcode) {
                case 1: 
                    grads[op.ins[0]] += grads[op.out] * sigmoid(values[op.ins[0]]);
                    break;
                case 2: 
                    {
                        for(size_t i=0; i < op.n_arguments; i++) { grads[op.ins[i]] += grads[op.out]; }
                    }
                    break;
                case 3:
                    grads[op.ins[0]] += grads[op.out] * values[op.ins[1]];
                    grads[op.ins[1]] += grads[op.out] * values[op.ins[0]];
                    break;
                case 4:
                    {
                        size_t n_arg = (size_t) op.n_arguments;
                        double avg_grad = 0.0;
                        for(size_t i = 0; i < n_arg; i++) {
                            avg_grad += values[op.out + i] * grads[op.out + i];
                        }
                        for(size_t i = 0; i < n_arg; i++) {
                            grads[op.ins[i]] += values[op.out + i] * (grads[op.out + i] - avg_grad);
                        }
                    }
                    break;
            }
        }  // end backward
    }
} 
```</details>

运行 10k 次操作 x 10k 次迭代的运行时间：**0.99 秒**

我更喜欢 Rust 的人体工程学，但在 C 中实现高速度要容易得多。Rust 与 Python 的交互也更方便。

### 让我们再次在 C 中进行自动微分。

我采取的另一种方法是将跟踪的图形“编译”成 C。因此，Python 生成一个长长的 C 文件，其中操作被一个接一个地调用，具有显式索引，类似于

```
...
vals[215] = vals[195] * vals[205];
vals[216] = vals[196] + vals[201] + vals[204];
... // etcetc, and then backward steps are also written the same way 
```

源代码很长，输出很多，为了加快编译速度，我们可以在 clang 中设置 `-O0`。使用 `-O0` 会产生较慢的二进制文件，但有趣的是 *并没有* 加速编译速度。我得到的最佳结果是大约 1 分钟的编译时间和 1 秒的完整运行时间。令人惊讶的是，消除参数的 switch/case 和内存查找并没有导致更快的执行。

鉴于每次图形变化都需要重新编译，用户体验的实时性为 1 分钟。这是不可接受的。

### 汇编

在追求最大速度的努力中，我决定降到汇编层面。否则，感觉就像是一次不完整的旅程。我们可以将计算图映射到一组低级指令，并避免“昂贵”的编译。如今，x86/64 已经不再是王者，但 armv7/armv8 也不是 —— 为几种架构编写汇编是完全不合理的。

所以 …… 使用 WebAssembly 如何？它是低级别的，编译速度快，而且跨平台。像 `wasmer`/`wasmtime` 这样的项目允许从其他语言与 wasm 代码进行交互。这是我第一次接触 WASM，我对它的印象相当积极：WASM 结合了 lisp 风格的语法（用于高效的流式解析）和堆栈机的执行模型。与经典的堆栈机不同，也不同于经典的汇编，WASM 允许对表达式进行分组，例如

```
;; canonical stack-machine way to compute a * b + c
(local.get $a)
(local.get $b)
f32.mul
(local.get $c)
f32.add

;; another way to say write the same, also perfectly legal in wasm
(f32.add 
    (f32.mul (local.get $a) (local.get $b))  
    (local.get $c) 
) 
```

这种便利性使得在 WASM 中编写的代码比传统的汇编更易读。抽象级别看起来对我来说刚刚好 —— 低级指令，但无需管理寄存器分配。

从指令上看，Webassembly 仍然与汇编语言非常接近，即没有`exp`，`log`，更不用说`log1p`之类的函数了。幸运的是，Peter Knight 提供了一个 WASM 的 [实现](https://gist.github.com/going-digital/02e46c44d89237c07bc99cd440ebfa43)，包括了`exp2`/`log2`。

我的主要问题是指数运算的速度是否足够，因为在 C 实现中，`exp`消耗了大量时间。遗憾的是，在一个简单的基准测试中，仅在 wasm 中计算指数就需要约 1.9 秒的时间，比 rust/C 慢。作为参考，javascript 在 0.7 秒内计算了相同数量的指数。因此，至少在数字处理的背景下，我对 WASM 的“接近本机速度”的品牌持保留态度。希望这方面会有所改善，但目前来看，WASM 处于失去竞争力的状态。

## 概要

因此，与主流库相比，我们实现了**1000 倍的加速**。

我觉得这并不奇怪 — 自动微分系统的主要用途是操作大型的 ndarrays。内存管理，消除拷贝，设备同步，计算并行化 — 这些都是主要关注点，每秒 1 百万次操作的吞吐量对于绝大多数场景和用户来说都是完全合理的。

但对我来说并非如此。我的场景在数字和设置方面完全不同，而且以张量为重点的自动微分速度太慢了。对于手头的问题，偏离常规的自动微分系统是正确且唯一可能的选择。探索不同的选择非常有趣，我的期望在这个探索过程中多次受到挑战。

👋
