<!--yml

类别：未分类

日期：2024-05-29 12:29:50

-->

# 在 Ruby 中实际应用的铁路导向管道 - Ismael Celis

> 来源：[https://ismaelcelis.com/posts/practical-railway-oriented-pipelines-in-ruby/](https://ismaelcelis.com/posts/practical-railway-oriented-pipelines-in-ruby/)

在这个系列中：

几年前，我曾经[探索过](/posts/composable-pipelines-in-ruby/)在 Ruby 中构建可组合处理管道的模式，使用了铁路导向的范式。

在这个系列中，我将描述一个简化的实现以供实际使用。

```
# An illustrative data processing pipeline
DataImporter = Pipeline.new do |pl|
  pl.step ValidateUserInput
  pl.step ReadCSV
  pl.step ValidateData
  pl.step TransformData
  pl.step SaveData
end 
```

# 第一部分：核心模式

我已经在各种项目中使用这种方法的版本很长一段时间了，我发现这是构建和维护复杂数据处理工作流的非常有效的方法。以下是该模式的核心组件。

## 结果类

一个通用的 `Result` 封装了通过管道传递的值，并且可以处于两种状态之一：`Continue` 或 `Halt`。这些值本身可以是与域相关的任何内容，但 `Result` 提供了处理它们的一致接口，以及诸如用户输入、错误和任意上下文的元数据（我将在另一篇文章中描述一些）。

```
# Initial result
result = Result.new([1, 2, 3, 4])
result.value # [1, 2, 3, 4]
result.continue? # => true 
```

`Result` 实例可以被 *继续* 或 *中止*。这些将返回具有相同或不同数据的新副本。

```
result = result.continue([5, 6, 7, 8])
result.value # [5, 6, 7, 8]
result.continue? # => true

result = result.halt
result.continue? # => false 
```

## 步骤接口

```
#call(Result) Result 
```

一个步骤是一个简单的对象，响应于 `#call` 并且接受一个 `Result` 作为输入，返回一个新的 `Result`。

这是一个步骤：

```
class MyCustomStep
  def call(result)
    # Do something with result.value
    result.continue(new_value)
  end
end 
```

并且这也是一个：

```
MyProcStep = proc do |result|
  # Do something with result.value
  result.continue(new_value)
end 
```

步骤可以是实例、类、Procs、Lambdas或[方法对象](https://ruby-doc.org/3.3.0/Method.html)。它们可以是无状态的过程，也可以是管理自己内部状态的复杂对象。无论它们如何定义或初始化，只要它们响应于 `#call` 即可。

## 管道

一个管道是一个包含一系列步骤的容器，这些步骤处理一个 `Result` 并返回一个新的 `Result`，按照它们添加到管道中的顺序执行。

```
MyPipeline = Pipeline.new do |pl|
  # Anything that responds to #call can be a step
  pl.step MyCustomStep.new

  # Or a simple proc. This one limits the set by the first 5 elements
  pl.step do |result|
    set = result.value.first(5)
    result.continue(set)
  end
end

# Usage
initial_result = Result.new((1..100))
result = MyPipeline.call(initial_result)
result.value # => [1, 2, 3, 4, 5] 
```

`Pipeline` 类本身非常简单。

```
class Pipeline
  attr_reader :steps

  def initialize(&config)
    @steps = []
    config.call(self) and @steps.freeze if block_given?
  end

  # Add a step to the pipeline, either in block form or as a callable object.
  # @param callable [Object, nil] a step that responds to `#call(Result) Result`
  # @yield [Result] a step as a block
  def step(callable = nil, &block)
    callable ||= block
    raise ArgumentError, "Step must respond to #call" unless callable.respond_to?(:call)
    steps << callable
    self
  end

  # Reduce over steps, call each one in turn,
  # * [Continue] results are passed on to the next step
  # * [Halt] results are returned unchanged
  # @param result [Result]
  # @return [Result]
  def call(result)
    steps.reduce(result) do |r, step|
      r.continue? ? step.call(r) : r
    end
  end
end 
```

因为它响应于 `#call(Result) Result`，一个管道本身也是一个步骤。稍后详细介绍。

## 铁路部分

这在于在管道的任何点上都能“中止”处理的能力是非常有用的。

```
MyPipeline = Pipeline.new do |pl|
  # This step halts processing if the set size is greater than 100
  pl.step do |result|
    if result.value.size > 100 # value to bit. Halt.
      return result.halt
    else # nothing to do. Continue.
      result
    end
  end

  # Any further steps here will not be executed
  # if the pipeline is halted in the step above
end 
```

关键在于 `Pipeline#call` 方法，在这里进行了详细展开：

```
# @param result [Result]
# @return [Result]
def call(result)
  steps.reduce(result) do |r, step|
    if r.continue? # if the result is a Continue, invoke the next step
      step.call(r)
    else # if the result is a Halt, return it unchanged
      r
    end
  end
end 
```

现在，任何返回 *中止* 的步骤都将跳过进一步的下游步骤。换句话说，步骤可以返回 *继续* 或 *中止*，但它只能接收 *继续* 作为参数。

```
#call(Result[Continue]) [Result[Continue], Result[Halt]] 
```

> 此模式的其他实现依赖于 Sum 类型或单子（monads）来表示 *继续* 和 *中止* 状态。查看 [Dry::Monads](https://dry-rb.org/gems/dry-monads/) 获取更功能化的方法。我还在[这篇文章](/posts/composable-pipelines-in-ruby/)中扩展了一个带类型的实现。

让我们进行一些数字计算：

```
# A portable step to validate set size
class ValidateSetSize
  # @param lte [Integer] the maximum size allowed (Less Than or Equal)
  def initialize(lte:) = @lte = lte

  def call(result)
    return result.halt if result.value.size > @lte
    result
  end
end

# A step to multiply each number in the set by a factor
# This one is a Proc that returns a Proc.
MultiplyBy = proc do |factor|
  proc do |result|
    result.continue(result.value.map { |n| n * factor })
  end
end

# Limit set to first N elements
LimitSet = proc do |limit|
  proc do |result|
    result.continue(result.value.first(limit))
  end
end

# Compose the pipeline
NumberCruncher = Pipeline.new do |pl|
  pl.step { |r| puts 'Logging'; r }
  pl.step ValidateSetSize.new(lte: 100)
  pl.step MultiplyBy.(2)
  pl.step LimitSet.(5)
end 
```

在这个例子中，第二个 `ValidateSetSize` 步骤将在集合大小大于 100 时停止管道，阻止 `MultiplyBy` 的运行。

```
initial_result = Result.new((1..101))
result = NumberCruncher.call(initial_result)
result.continue? # => false 
```

+   1\. `记录`

+   2\. `ValidateSetSize.new(lte: 100)`

+   3\. `乘以.(2)`

+   4\. `限制集合.(5)`

但是，如果所有步骤都返回 *continue* 结果，管道会处理所有步骤并返回最终结果。

```
initial_result = Result.new((1..99))
result = MyPipeline.call(initial_result)
result.continue? # => true
# Each number in set was multiplied by 2, then limited to the first 5
result.value # => [2, 4, 6, 8, 10] 
```

+   1\. `Logging`

+   2\. `ValidateSetSize.new(lte: 100)`

+   3\. `MultiplyBy.(2)`

+   4\. `LimitSet.(5)`

## 组合管道

由于 `Pipeline` 本身实现了 `#call(Result) Result` 接口，它可以作为另一个管道中的步骤使用。

```
BigPipeline = Pipeline.new do |pl|
  pl.step Step1 # a regular step
  pl.step NumberCruncher # a nested pipeline
  pl.step Step3 # another regular step
end 
```

+   1\. `Step1`

+   2\. `NumberCruncher`

    +   2.1\. `Logging`

    +   2.2\. `ValidateSetSize.new(lte: 100)`

    +   2.3\. `MultiplyBy.(2)`

    +   2.4\. `LimitSet.(5)`

+   3\. `Step3`

这允许将复杂的处理工作流打包成可重复使用的组件，每个组件如果需要可以由多个步骤组成。还可以有工厂方法来参数化管道的创建。

```
# A component to validate and coerce a set of numbers
# It returns a 2-step pipeline that can be composed into a larger pipeline
module NumberValidation
  def self.new(lte:)
    Pipeline.new do |pl|
      pl.step ValidateSetSize.new(lte: lte)
      pl.step CoerceToIntegers
    end
  end

  CoerceToIntegers = proc do |result|
    result.continue(result.value.map(&:to_i))
  end
end

# Compose a larger pipeline
BigPipeline = Pipeline.new do |pl|
  pl.step Step1
  pl.step NumberValidation.new(lte: 100) # a nested pipeline
  pl.step Step3
end 
```

管道也可以被自定义类内部使用。

```
class NumberValidation
  def initialize(lte:)
    @pipeline = Pipeline.new do |pl|
      pl.step ValidateSetSize.new(lte: lte)
      # Use a Method object as step
      # https://ruby-doc.org/3.3.0/Method.html
      pl.step method(:coerce_to_integers)
    end
  end

  # Expose the Step interface
  # to make instances of this class behave like a step
  def call(result) = @pipeline.call(result)

  private def coerce_to_integers(result)
    result.continue(result.value.map(&:to_i))
  end
end 
```

使用哪种方法取决于每个步骤的内部。除了简单的 `#call` 接口外，步骤都是有效的黑盒，重构它们很直接。

## 结论

与任何事物一样，这种方法有其权衡之处。如果问题更适合看作是对象图而不是序列，或者所需的处理不能轻松地分解成步骤，那么这种方法可能不是最佳选择。

总体而言，我发现它提供了一个在问题推理中的 *简单* 模型（在 [Rick Hickey 的意义上](https://www.youtube.com/watch?v=SxdOUGdseq4)）。

任何可以强制符合 [Monoid Laws](https://blog.ploeh.dk/2017/10/06/monoids/) 的操作都可能是一个不错的候选。

在这个系列的下一篇文章中，我描述了如何通过处理用户输入、错误和一般元数据来使管道更加有用。

本文中使用的基本实现在 [这里](https://gist.github.com/ismasan/0bdcc76c2ea48f4259b38fafe131edb8)。
