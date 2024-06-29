<!--yml

category: 未分类

date: 2024-05-29 12:44:51

-->

# 用接口在Go中表示状态 - Evan Moses

> 来源：[https://www.emoses.org/posts/resolver-resolved-pattern/](https://www.emoses.org/posts/resolver-resolved-pattern/)

# 用接口在Go中表示状态

前几天我在Go中设计了一个巧妙的小模式。这是一种通过在系统中暴露不同状态的API，而仅在单个底层结构中保存状态来表示系统中的状态变化的方法。我确信我不是第一个发明这个的人，也许这已经有了一个名字，所以如果您知道，请告诉我 *[更新：[apg](https://lobste.rs/~apg) 在Lobsters上指出了名字](https://lobste.rs/s/tzgizl/representing_state_as_interfaces_go#c_cvlm2d)，我很喜欢]*。首先我将展示一个模式的实例，然后是动机。

## 这个模式

```
// You start with a Resolver, and incrementally feed it data to be looked up type Resolver interface {  // Collect collects some piece of data and maybe extra information about it that you // need to do resolution Collect(someId string, someData string) // Maybe you also need some global contextual data to do the resolve AddContextualData(data SomeContext) // When you're done, the resolve can send the bulk query to the executor to perform // the lookup that you want to do Execute(context.Context, Executor) (Resolved, error) }   // Resolved is what you get back after you Execute, and it lets you access the resolved data type Resolved interface {  // Resolve returns the result. If the result doesn't have that id, either because it // wasn't looked up successfully during execution or because you never [Collect]ed it, // it will return ("", false) Resolve(id string) (string, bool) }   // Executor is capable of doing the db lookup/cache lookup/service request/http request // that actually gets the data you need to get type Executor interface {  DoTheThing( ctx context.Context, idsToResolve map[string][]string, contextualData SomeContext, ) ([]ResolveResult, error) } 
```

到目前为止，很无聊。你有一个`Resolver`用来批量处理查询数据，一个`Executor`来执行查询，以及一个`Resolve`让你访问结果。这里有意思的是`Resolver`和`Resolved`可以由同一个结构体实现：

```
type idResolver struct {  collected map[string][]string  contextData SomeContext  resolved map[string]string }   func (r *idResolver) Collect(someId string, someCategory string) {  r.collected[someCategory] = append(r.collected[someCategory], someId) }   func (r *idResolver) AddContextualData(data SomeContext) {  r.contextData = data }   func (r *idResolver) Execute(ctx context.Context, executor Executor) (Resolved, error) {  result, err := executor.DoTheThing(ctx, r.collected, r.contextData)  if err != nil {  return nil, err  }    r.resolved = transformResult(result)    /***  * 🪄 THE MAGIC 🪄  * Ooh look I'm just returning this struct as a Resolved!  */  return r, nil }   func (r *idResolver) Resolve(id string) (string, bool) {  res, ok := r.resolved[id]  return res, ok } 
```

## 好的，但是为什么？

我正在构建一个系统，通过ID批量查找我的系统中的名称。但如果你在执行之前调用了`Resolve`，你会得到一个无效的结果。而如果你在执行后调用了`Collect`，你永远不会查找这个ID。所以我在`IdResolver`中添加了一个布尔值`hasExecuted`，这样如果你在调用`Execute`之前调用`Resolve`，或者在执行后调用`Collect`，它会检查该标志并抛出异常。

```
func (r *StatefulIdResolver) Collect(someId string, someCategory string) {  if r.hasExecuted { panic("Collect called after Execute") }  r.collected[someCategory] = append(r.collected[someCategory], someId) }   // Execute executes and changes state but doesn't return anything new func (r *StatefulIdResolver) Execute(ctx context.Context, executor Executor) error {  if r.hasExecuted { panic("Execute called twice") } ... r.hasExecuted = true ... }   func (r *StatefulIdResolver) Resolve(id string) (string, bool) {  if !r.hasExecuted { panic("Resolve called before execute") }  res, ok := r.resolved[id]  return res, ok } 
```

这有点混乱，难以阅读和维护，逻辑混乱起来也容易多了。

## 何时使用它

当然，`Resolver`和`Resolved`*可以*由具有自己方法的不同结构体表示。但在这种情况下，感觉我们真的在处理一个单一对象：它收集数据，对其进行操作，并管理结果。

问题的关键是对象在不同状态下具有不同有效的操作，而Go接口正是公开这一点的完美方式。

## 更新：这只是一个`Builder`吗？

一些网友指出，这与Java/C#中经常见到的Builder模式非常相似（在Go中也有，虽然程度较小），特别是多阶段的[Step Builder](https://www.svlada.com/step-builder-pattern/)。

我认为本质上没有根本的区别，但我熟悉的Builder通常有一个状态改变/执行步骤（`Build()`），生成一个不可变对象供其他地方使用，而不是执行任何形式的有影响的操作。我认为这种模式更加通用（你当然可以将其用作Builder），并且有不同的目的。
