<!--yml
category: 未分类
date: 2024-05-29 12:29:50
-->

# Practical Railway-Oriented Pipelines in Ruby - Ismael Celis

> 来源：[https://ismaelcelis.com/posts/practical-railway-oriented-pipelines-in-ruby/](https://ismaelcelis.com/posts/practical-railway-oriented-pipelines-in-ruby/)

In this series:

Some years ago I [explored patterns](/posts/composable-pipelines-in-ruby/) for building composable processing pipelines in Ruby, using a Railway-oriented paradigm.

In this series, I’ll describe a simplified implementation for practical use.

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

# Part 1: The core pattern

I’ve relied on versions of this approach in various projects for some time now, and I’ve found it to be a very effective way to build and maintain complex data processing workflows. The following are the core components of the pattern.

## The result class

A generic `Result` wraps values passed through the pipeline, and can be in one of two states: `Continue` or `Halt`. The values themselves can be anything relevant to the domain, but `Result` offers a consistent interface for handling them in the pipeline, as well as metadata such as user input, errors and arbitrary context (I’ll describe some of this in a separate article).

```
# Initial result
result = Result.new([1, 2, 3, 4])
result.value # [1, 2, 3, 4]
result.continue? # => true 
```

`Result` instances can be *continued* or *halted*. These return new copies with the same or different data.

```
result = result.continue([5, 6, 7, 8])
result.value # [5, 6, 7, 8]
result.continue? # => true

result = result.halt
result.continue? # => false 
```

## The Step interface

```
#call(Result) Result 
```

A step is a simple object that responds to `#call` and takes a `Result` as input, returning a new `Result`.

This is a step:

```
class MyCustomStep
  def call(result)
    # Do something with result.value
    result.continue(new_value)
  end
end 
```

And so is this:

```
MyProcStep = proc do |result|
  # Do something with result.value
  result.continue(new_value)
end 
```

Steps can be instances, classes, Procs, Lambdas or [Method objects](https://ruby-doc.org/3.3.0/Method.html). They can be stateless procedures, or complex objects that manage their own internal state. It doesn’t matter how they were defined or initialised, as long as they respond to `#call`.

## The pipeline

A pipeline is a container for a sequence of steps that process a `Result` and return a new `Result`, in the order they were added to the pipeline.

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

There’s very little to the `Pipeline` class itself.

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

Because it responds to `#call(Result) Result`, a pipeline is itself a step. More on that later.

## The Railway bit

Where this becomes useful is in the ability to “halt” processing at any point in the pipeline.

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

The key to this is the `Pipeline#call` method, expanded here for clarity:

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

Now, any step that returns a *halt* will just skip over further steps downstream. In other words, a step can return a *continue* or a *halt*, but it can only ever receive a *continue* as argument.

```
#call(Result[Continue]) [Result[Continue], Result[Halt]] 
```

> Other implementations of this pattern rely on Sum types or monads to represent the *continue* and *halt* states. See [Dry::Monads](https://dry-rb.org/gems/dry-monads/) for a more functional approach. I also expand on a typed implementation in [this article](/posts/composable-pipelines-in-ruby/).

Lets do some number crunching:

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

In this example, the second `ValidateSetSize` step will halt the pipeline if the set size is greater than 100, preventing `MultiplyBy` from running.

```
initial_result = Result.new((1..101))
result = NumberCruncher.call(initial_result)
result.continue? # => false 
```

*   1\. `Logging`
*   2\. `ValidateSetSize.new(lte: 100)`
*   3\. `MultiplyBy.(2)`
*   4\. `LimitSet.(5)`

However, if all steps return a *continue* result, the pipeline processes all steps and returns the final result.

```
initial_result = Result.new((1..99))
result = MyPipeline.call(initial_result)
result.continue? # => true
# Each number in set was multiplied by 2, then limited to the first 5
result.value # => [2, 4, 6, 8, 10] 
```

*   1\. `Logging`
*   2\. `ValidateSetSize.new(lte: 100)`
*   3\. `MultiplyBy.(2)`
*   4\. `LimitSet.(5)`

## Composing pipelines

Since `Pipeline` itself implements the `#call(Result) Result` interface, it can be used as a step in another pipeline.

```
BigPipeline = Pipeline.new do |pl|
  pl.step Step1 # a regular step
  pl.step NumberCruncher # a nested pipeline
  pl.step Step3 # another regular step
end 
```

*   1\. `Step1`
*   2\. `NumberCruncher`
    *   2.1\. `Logging`
    *   2.2\. `ValidateSetSize.new(lte: 100)`
    *   2.3\. `MultiplyBy.(2)`
    *   2.4\. `LimitSet.(5)`
*   3\. `Step3`

This allows “packaging up” complex processing workflows into reusable components, where each component can be composed of multiple steps if need be. It’s also possible to have factory methods that parameterise the creation of pipelines.

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

Pipelines can also be used internally by custom classes.

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

Which approach to use will depend on each step’s internals. Other than the simple `#call` interface, steps are effective black boxes and refactoring them is straightforward.

## Conclusion

Like anything, this approach has its trade-offs. If the problem can be better thought of as an object graph rather than a sequence, or the processing required can’t be easily broken down into steps, then this approach might not be the best fit.

In general, I’ve found it provides a *simple* mental model to reason about problems (in [the Rick Hickey sense](https://www.youtube.com/watch?v=SxdOUGdseq4)).

Any operation that can be coerced into complying with the [Monoid Laws](https://blog.ploeh.dk/2017/10/06/monoids/) can be a good candidate.

In the next article in this series, I describe how to make pipelines more useful by handling user input, errors and general metadata.

The basic implementation used in this article is [here](https://gist.github.com/ismasan/0bdcc76c2ea48f4259b38fafe131edb8).