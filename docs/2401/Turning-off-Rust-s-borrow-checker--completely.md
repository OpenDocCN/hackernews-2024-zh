<!--yml

category: 未分类

日期：2024年5月27日15:03:39

-->

# 完全关闭Rust的借用检查器

> 来源：[https://iter.ca/post/rust-skip-borrowck/](https://iter.ca/post/rust-skip-borrowck/)

我最近遇到了[`#[you_can::turn_off_the_borrow_checker]`](https://docs.rs/you-can/latest/you_can/attr.turn_off_the_borrow_checker.html)，这是一个Rust宏，使得借用检查器对于一个函数“关闭”。它将函数的代码转换为不使用借用检查器的不安全代码来进行引用操作。

当然，这不是您想要在生产中使用的东西，或者真的只能用于教育和实验。然而，这让我好奇是否可以在更低的级别实现它：我们可以修补编译器以删除借用检查器吗？可以。

显而易见的方法是不运行进行借用检查的传递。编译器的`analysis`传递（在技术上，它是一个[查询](https://rustc-dev-guide.rust-lang.org/query.html)，而不是一个传递，但这在这里不相关）做了很多事情，包括一些借用检查。让我们不运行`analysis`传递！直接返回，以便不运行任何分析：

```
fn analysis(tcx: TyCtxt<'_>, (): ()) -> Result<()> {  return Ok(()); // don't actually do any analysis! ... 
```

这导致函数的其余部分不会运行，并在编译编译器时引发`unreachable statement`警告。

这有点奏效。大多数有效的Rust代码仍然可以使用这个修补过的编译器。偶尔会出现内部编译器错误（ICEs），有时你会因为编译器的某些部分假设分析部分已经执行而得到一些额外的✨*奖励错误*✨。但遗憾的是，移除了分析传递仍然会导致一些借用检查。我们需要使用核心选项：完全忽略错误。在编译器发出错误的一部分，有一个`HandlerInner::emit_diagnostic`，它会发出一个错误诊断。该函数的一部分会检查是否正在发出错误，如果是，则递增一个计数器：

```
if matches!(diagnostic.level, Level::Error { lint: true }) {  self.bump_lint_err_count(); } 
```

在启动代码生成之前，编译器会检查错误计数是否为非零，如果是，则会中止。但是，如果我们只是去掉对`bump_lint_err_count`的调用，错误数量将始终为零。错误仍然会显示，但不会阻止编译成功。通常，编译器在出现错误后不会停止，而是继续进行，然后仅在下一阶段编译之前中止。这样，如果某些代码存在多个错误，你将一次性看到所有错误。但是通过不增加错误计数，我们将欺骗编译器认为没有错误！如果你想一起跟踪，这里是[补丁](https://github.com/rust-lang/rust/commit/709edf2581ad86d02897ba169eb6d92aeda5c11d)。

这意味着会生成和显示错误，但修补的编译器无视了错误的存在，并尝试生成代码。

这不能捕获一些立即停止编译的致命错误，但幸运的是大多数错误会让编译器继续进行，直到某个未来的停止点（但由于错误计数始终为0，所以现在不会发生）。

此时，我相当确信这实际上不会起作用。但它确实可以。这里是[一个借用检查器不喜欢的Rust代码示例](https://blog.logrocket.com/introducing-the-rust-borrow-checker/#inpractice)：

```
fn hold_my_vec<T>(_: Vec<T>) {}   fn main() {  let v = vec![2, 3, 5, 7, 11, 13, 17]; hold_my_vec(v); let element = v.get(3);   println!("I got this element from the vector: {:?}", element); } 
```

这是当我们通过我们的修补后的编译器运行时发生的情况：

```
error[E0382]: borrow of moved value: `v`  --> src/main.rs:6:19 | 4    |     let v = vec![2, 3, 5, 7, 11, 13, 17];  |         - move occurs because `v` has type `Vec<i32>`, which does not implement the `Copy` trait 5    |     hold_my_vec(v);  |                 - value moved here 6    |     let element = v.get(3);  |                   ^^^^^^^^ value borrowed here after move | = note: borrow occurs due to deref coercion to `[i32]` note: deref defined here  --> /home/smit/rustc-dev/rust/library/alloc/src/vec/mod.rs:2434:5 | 2434 |     type Target = [T];  |     ^^^^^^^^^^^^^^^^^^   For more information about this error, try `rustc --explain E0382`.  Finished dev [unoptimized + debuginfo] target(s) in 0.00s Running `target/debug/rustnoerror` I got this element from the vector: Some(-501713657) 
```

有一个错误，但由于我们的补丁，编译器只会继续进行。结果，编译后的程序产生垃圾数据：`Some(-501713657)`不是列表中的元素。

这是它如何处理来自[`#[you_can::turn_off_the_borrow_checker]`](https://docs.rs/you-can/latest/you_can/attr.turn_off_the_borrow_checker.html#expanded)的示例：

```
error[E0499]: cannot borrow `owned` as mutable more than once at a time  --> src/main.rs:6:22 | 5  |     let mut_1 = &mut owned[0];  |                      ----- first mutable borrow occurs here 6  |     let mut_2 = &mut owned[1];  |                      ^^^^^ second mutable borrow occurs here ... 10 |     let undefined = *mut_1 + *mut_2;  |                     ------ first borrow later used here   error[E0505]: cannot move out of `owned` because it is borrowed  --> src/main.rs:9:10 | 5  |     let mut_1 = &mut owned[0];  |                      ----- borrow of `owned` occurs here ... 9  |     drop(owned);  |          ^^^^^ move out of `owned` occurs here 10 |     let undefined = *mut_1 + *mut_2;  |                     ------ borrow later used here   Some errors have detailed explanations: E0499, E0505. For more information about an error, try `rustc --explain E0499`.  Finished dev [unoptimized + debuginfo] target(s) in 0.00s Running `target/debug/rustnoerror` 1511396695 
```

再次，它只会输出垃圾数据。

以下是我们的修补后编译器处理各种错误条件的*有趣*方法：

+   许多结果会导致内部编译器错误，因为代码生成器假定编译器的其他部分已经完成了它们的工作。

+   如果你有非穷尽模式，编译器会尝试执行非法指令并核心转储。

+   有时引用受错误影响的变量的格式字符串会导致内部编译器错误，因此你实际上必须始终执行`std::io::stdout().write_all(...)`而不是`println!`。

请记住不要将其用于除测试之外的任何其他用途！

* * *
