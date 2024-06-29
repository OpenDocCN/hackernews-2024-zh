<!--yml

category: 未分类

date: 2024-05-27 14:54:34

-->

# 大O见解 - DeriveIt

> 来源：[https://deriveit.org/coding/cheat-sheets/big-o-insights-106?h=t](https://deriveit.org/coding/cheat-sheets/big-o-insights-106?h=t)

想象一下，你被要求评估算法的速度有多快。考虑一下你可能如何做到这一点。一个想法是仅仅计算它在输入大小<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>n</mi></mrow><annotation encoding="application/x-tex">n</annotation></semantics></math>n时执行的简单操作总数（读取，写入，比较）。例如：

事情是，要像上面那样提出一个公式很难，而且系数会根据你运行程序的处理器而变化。

为了简化这个问题，计算机科学家只讨论算法在<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>n</mi></mrow><annotation encoding="application/x-tex">n</annotation></semantics></math>n变得非常大时速度*增长*的方式。当<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>n</mi></mrow><annotation encoding="application/x-tex">n</annotation></semantics></math>n变得非常大时，最大阶项总是消除其他项（例如当n=1,000,000时）。这种情况不管你有什么系数都会发生。你可以看到，当n=1,000,000时，<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><msup><mi>n</mi><mn>2</mn></msup></mrow><annotation encoding="application/x-tex">n^2</annotation></semantics></math>n2项主导，尽管它有一个较小的系数：

因此，计算机科学家只讨论最大的术语，而没有任何系数。上述算法以<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><msup><mi>n</mi><mn>2</mn></msup></mrow><annotation encoding="application/x-tex">n^2</annotation></semantics></math>n2作为最大的术语，因此我们说：

这被称为大O符号。口头上说，你可以说算法的运行时间“大约为<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><msup><mi>n</mi><mn>2</mn></msup></mrow><annotation encoding="application/x-tex">n^2</annotation></semantics></math>n2次操作”。换句话说，算法运行所需的时间与术语<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><msup><mi>n</mi><mn>2</mn></msup></mrow><annotation encoding="application/x-tex">n^2</annotation></semantics></math>n2成比例。

计算机科学家根据内存如何扩展来描述内存。如果你的程序需要<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mn>200</mn><mi>n</mi><mo>+</mo><mn>5</mn></mrow><annotation encoding="application/x-tex">200n + 5</annotation></semantics></math>200n+5个存储单元，那么计算机科学家说<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>空间复杂度</mtext><mo>=</mo><mi>O</mi><mo stretchy="false">(</mo><mi>n</mi><mo stretchy="false">)</mo></mrow><annotation encoding="application/x-tex">\text{空间复杂度} = O(n)</annotation></semantics></math>空间复杂度=O(n)。

有一个普遍的困惑人们常有。人们直觉地知道，需要<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>n</mi></mrow><annotation encoding="application/x-tex">n</annotation></semantics></math>n次操作是<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>O</mi><mo stretchy="false">(</mo><mi>n</mi><mo stretchy="false">)</mo></mrow><annotation encoding="application/x-tex">O(n)</annotation></semantics></math>O(n)。但很多人会困惑于，比如说，需要<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mn>2</mn><mi>n</mi></mrow><annotation encoding="application/x-tex">2n</annotation></semantics></math>2n次操作仍然是<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>O</mi><mo stretchy="false">(</mo><mi>n</mi><mo stretchy="false">)</mo></mrow><annotation encoding="application/x-tex">O(n)</annotation></semantics></math>O(n)，或者需要26次操作是<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>O</mi><mo stretchy="false">(</mo><mn>1</mn><mo stretchy="false">)</mo></mrow><annotation encoding="application/x-tex">O(1)</annotation></semantics></math>O(1)。

Big O 只是估算你的函数增长的一个大概值。它与<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>n</mi></mrow><annotation encoding="application/x-tex">n</annotation></semantics></math>n的哪个幂增长？如果它根本不增长，那就像数字1增长一样（根本不增长），所以它是<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>O</mi><mo stretchy="false">(</mo><mn>1</mn><mo stretchy="false">)</mo></mrow><annotation encoding="application/x-tex">O(1)</annotation></semantics></math>O(1)，即n的0次幂。

如果它按照某个百分比<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>n</mi></mrow><annotation encoding="application/x-tex">n</annotation></semantics></math>n增长，那么它的增长率为<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>O</mi><mo stretchy="false">(</mo><mi>n</mi><mo stretchy="false">)</mo></mrow><annotation encoding="application/x-tex">O(n)</annotation></semantics></math>O(n)。有些是平方的，它的复杂度是<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>O</mi><mo stretchy="false">(</mo><msup><mi>n</mi><mn>2</mn></msup><mo stretchy="false">)</mo></mrow><annotation encoding="application/x-tex">O(n^2)</annotation></semantics></math>O(n²)。如果它按照<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>log</mi><mo>⁡</mo><mi>n</mi></mrow><annotation encoding="application/x-tex">\log n</annotation></semantics></math>logn增长，那么它的复杂度是<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>O</mi><mo stretchy="false">(</mo><mi>log</mi><mo>⁡</mo><mi>n</mi><mo stretchy="false">)</mo></mrow><annotation encoding="application/x-tex">O(\log n)</annotation></semantics></math>O(logn)。依此类推。这只是一个粗略的估计，试图过于正式化或者过于复杂化是一个大错误。它应该是直观的。

如果你建立了一个大小为10,000的数组，你必须至少执行了10,000次操作。但如果你执行了10,000次操作，你可能没有使用10,000个内存槽位。你的<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>时间复杂度</mtext></mrow><annotation encoding="application/x-tex">\text{Time Complexity}</annotation></semantics></math>时间复杂度将*始终*大于或等于你的<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>空间复杂度</mtext></mrow><annotation encoding="application/x-tex">\text{Space Complexity}</annotation></semantics></math>空间复杂度。

这张图片可能会给你一个直观的感觉。你可以重复使用空间，但不能重复使用时间。这最终是因为在我们的宇宙中，你可以回到空间，但不能回到时间。

它将*始终*成立<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mtext>总时间</mtext><mo>≥</mo><mtext>总空间</mtext></mrow><annotation encoding="application/x-tex">\text{Total Time} \ge \text{Total Space}</annotation></semantics></math>总时间≥总空间，即使你在编写并行算法时（这在面试中不会出现），因为并行算法只是由许多单处理器算法组成。这就是为什么在大多数问题中通常假设内存无限，以及为什么像原地排序这样的有限内存问题可能并不是非常有趣。时间限制空间，因此时间更加有趣。
