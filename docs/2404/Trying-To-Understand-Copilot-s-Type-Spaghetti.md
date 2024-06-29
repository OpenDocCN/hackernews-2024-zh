<!--yml

category: 未分类

date: 2024-05-27 13:13:38

-->

# 尝试理解Copilot的类型混乱

> 来源：[https://rtpg.co/2024/03/07/parsing-copilots-type-spaghetti/](https://rtpg.co/2024/03/07/parsing-copilots-type-spaghetti/)

几天前，[这段代码片段](https://twitter.com/ataiiam/status/1765089261374914957)由Github Copilot生成的Typescript正在流传：

```
`type  MappedParameterTypes<T  extends  Parameter[]>  =  {
  // Check if the parameter has an 'enum' defined
  [P  in  T[number]  as  P["name"]]:  P  extends  {enum:  Array<infer  E>  }
  ?  E  extends  string  // Ensure the enum values are strings
  ?  P["required"]  extends  false  // Check if the parameter is optional
  ?  E  |  undefined  // If so, include 'undefined' in the type
  :  E  // Othewise use the enum type directly
  :  never  // This case should not occur since 'enum' implies string values
  :  // Handle parameters defined as 'object' with specified attributes
  P  extends  {type:  "object";  attributes:  infer  Attributes  }
  ?  Attributes  extends  Parameter[]
  ?  MappedParameterTypes<Attributes>  // Recursively map the attributes of the object
  :  never  // If 'attributes' is not an array of Parameters, this is invalid
  :  // Handle parameters defined as 'object[]' without specified attributes
  P  extends  {  type:  "object[]";  attributes?:  never  }
  ?  any[]  // Default to 'any[]' for arrays of objects without specific attributes
  :
  P  extends  {type:  "object[]";  attributes:  infer  Attributes  }
  ?  Attributes  extends  Parameter[]
  ?  MappedParameterTypes<Attributes>[]  // Recursively map each object in the array
  :  any[]  // Default to 'any[]' if attributes are not properly defined
  :  // Handle all other parameter types
  P["required"]  extends  false
  ?  // Include 'undefined' for optional parameters
  TypeMap[P["type"]  extends  keyof  TypeMap  ?  P["type"]  :  "string"]  |  undefined
  :  // Use the direct mapping from 'TypeMap' for the parameter's type
  TypeMap[P["type"]  extends  keyof  TypeMap  ?  P["type"]  :  "string"];
};` 
```

这是一些可以将字段定义数组转换为类型的类型级编程。上面是原始代码，我能够用以下内容填补`TypeMap`和`Parameter`的空白：

```
`interface  Parameter  {
  name:  string,
  required?:  false,
  type?:  keyof  TypeMap
}

interface  TypeMap  {
  number:  number,
  boolean:  boolean,
  string:  string,
}` 
```

首先的问题是，你如何使用类似`MappedParameterTypes`这样的东西？我肯定在许多项目中看到过这段代码，特别是那些拥有定制ORM的项目。基本思想是将对象配置映射到某种类型。

```
`const  parameters  =  [
  {
  name:  "foo",
  type:  "number"
  }  as  const,
  {
  name:  "bar",
  required:  false,
  type:  "string"
  }  as  const,
  {
  name:  "baz",
  enum:  ["b",  "c"]  as  const
  }  as  const,
];

const  mappedObjs:  MappedParameterTypes<typeof  parameters>[]  =  [
  {
  foo:  1,
  bar:  "hi",
  baz:  "b"
  },
  {
  foo:  2,
  bar:  undefined,
  baz:  "c"
  }
]` 
```

我有一个对象定义，其中`foo`是一个数字，`bar`是一个字符串，`baz`是一个枚举。我已经到处添加了`const`来“确保”Typescript不会丢失`parameters`中的对象字面量（在Typescript的类型级编程中是一个固定的事情要注意）。

但最终结果是，`mappedObjs`的“类型”是`{foo: number, bar: string, baz: "b" | "c"}[]`。我们能够将我们的参数结构转换为一种类型。

这是Typescript中非常常见的任务，可以捕捉到很多错误！如果在对象中没有包含`foo`或`bar`，那么就会出错。但按照现有的代码，仍然可能很脆弱。

例如，在上面的例子中，我可以将`baz`设置为`"a"`或`"d"`，并不会得到错误，尽管我已经创建了`baz`作为一个枚举类型！我不能设置一个字符串，但在类型统一过程中，Typescript决定`["b", "c"] as const`不是`("b" | "c")[]`，而是只是`string[]`。

然而，如果我写了：

```
`{
  name:  "baz",
  enum:  ["b",  "c"]  as  ("b"  |  "c")[]
}` 
```

那么这个问题就消失了。类型编程总的来说是一个不断调整的过程。

## 这段代码有更简洁的版本吗？

当前的类型，坦率地说，并不是最糟糕的类型级编程。有注释，并且缩进被用来相对良好地遵循三元流。但错误有点难以理解，因为一切都是对象字面量。

最简单的做法就是将一些类型逻辑分解到自己的中间类型中。

```
`interface  EnumParameter<E>  extends  Parameter  {
  enum:  Array<E>
}

type  EnumParameterValue<P  extends  EnumParameter<E>,  E>  =
  E  extends  string  //  Ensure  the  enum  values  are  strings
  ?  P["required"]  extends  false  //  Check  if  the  parameter  is  optional
  ?  E  |  undefined  //  If  so,  include  'undefined'  in  the  type
  :  E  //  Othewise  use  the  enum  type  directly
  :  never  //  This  case  should  not  occur  since  'enum'  implies  string  values

type  MappedParameterTypes<T  extends  Parameter[]>  =  {
  //  Check  if  the  parameter  has  an  'enum'  defined
  [P  in  T[number]  as  P["name"]]:
  P  extends  EnumParameter<infer  E>  
  ?  EnumParameterValue<P,  E>  
  :  //  Handle  parameters  defined  as  'object'  with  specified  attributes
  ...
}` 
```

在这里，我分离了枚举参数形状，以及提取数组的逻辑。

另一种选择涉及分离出“必需”的检查，并直接在更高层次上包含`string`检查，避免需要`never`。

```
`interface  EnumParameter<E  extends  string>  extends  Parameter  {
  enum:  Array<E>
}

type  MaybeRequired<V,  P  extends  Parameter>  =  
  P["required"]  extends  false  ?  V  |  undefined  :  V

type  MappedParameterTypes<T  extends  Parameter[]>  =  {
  //  Check  if  the  parameter  has  an  'enum'  defined
  [P  in  T[number]  as  P["name"]]:
  P  extends  EnumParameter<infer  E>  
  ?  MaybeRequired<E,  P>
  :  ...
}` 
```

这种代码的好处在于，稍后当你编写参数值时，你可以通过实际标注类型来确保没有犯错。而不是在映射类型上看到错误，你会看到（正确地）显示在参数上的错误。

```
`const  ep:  EnumParameter<"b"  |  "c">  =  {
  name:  "bar",
  enum:  ["b",  "c"]
}` 
```

这在像这样较小的常量上可能看起来很愚蠢，但在编写代码时非常宝贵，可以帮助你跟踪你所处的位置。

最终，这种事情就像任何一个单行器，从可读性角度来说极其主观。我能够浏览这个东西并理解它是如何工作的。但是当构建这些单行时，如果不把事情拆分出来，你很容易把森林看错。

例如：

```
 `P  extends  {type:  "object[]";  attributes:  infer  Attributes  }
  ?  Attributes  extends  Parameter[]
  ?  MappedParameterTypes<Attributes>  //  Recursively  map  each  object  in  the  array
  :  any[]  //  Default  to  'any[]'  if  attributes  are  not  properly  defined` 
```

为什么要费力写一个类型映射系统，只是为了你的代码能够引入`any[]`，因为你的属性被轻微地拼写错误了？这种代码将导致类型错误报告在意料之外的地方（或者根本不报告），而不是错误实际发生的地方。

我同样发现，放置在有些地方有些令人困惑的检查`P["requires"] extends false`，会导致奇怪的可用性问题。我可以将枚举或字符串或数字字段设置为可选的，但我不能将对象字段设置为可选的？

## 在类型级编程中失去信息

我在上面有点撒谎。`EnumParameter< "b" | "c">` 在下游会引起问题，因为通过注释对象，我们失去了元素是否必需的信息。我们还丢失了参数的名称。

Typescript足够强大，你可以通过注释大量的常量字典映射来应付*很多*事情。但真正迂腐的做法最终会变成以下这样：

```
`interface  EnumParameter<
  E  extends  string,
  Name  extends  string,
  Required  extends  boolean
>  extends  Parameter<Name,Required>  {
  ...
}` 
```

如果你想访问类型的信息，一般来说你会希望它在你的通用签名中的*某个地方*存在。否则，类型注解将影响你的结果。最好的情况是，你“仅仅”有一些模糊的类型错误。但最坏的情况是，你最终会得到一些隐式的`any`在四处飘荡。

最初的单行美丽之处在于，在Typescript中，这种意大利面代码对良好的UX是友好的！这个世界是一个残酷的地方，注释和试图使事物显式化可能会让你的生活变得更加困难。美丽可能不是Copilot的“量子叠加的一百万个代码仓库”的一个适合的词。但在许多情况下，它似乎工作得足够好。

但是，如果你想要一个易于维护的东西，很不幸，通常需要大量的迂腐工作并将事物放入通用类型中。

在类型级编程中的一个核心经验教训，至少在Typescript中是如此：如果你希望某些信息被使用，将其作为通用类型签名的一部分可避免因类型推断系统而“意外”工作的代码。

## 如果可能，改变你的问题的形状

如果你打算做这种类型级别的事情，倾向于简单易行的方式可以节省大量时间。与其使用参数数组，不如使用参数映射来减少迂腐类型的成本。

```
`const  parameters  =  {
  "foo":  stringParam,
  "bar":  optionalNumberParam,
  "baz":  myEnumParam
}` 
```

当你有了这种结构之后，你可以更容易地重复使用其他地方定义的常量。这意味着，如果你最终得到了一些迂腐的签名，你不需要为它们付出巨大的代价。如果你有了迂腐的类型签名，你的错误可能也很迂腐，但更可能出现在问题所在，而不是在下游。

很容易接近一些基本可行的解决方案。但通常情况下，你可能只需稍作调整，就能找到更为直接的解决方案。例如，在这里，将命名和类型开发中的必要性解耦可能会导致更多行的代码，但更易于理解。

但我必须诚实地说：我尝试了几次“简单胜利”的重构，经常会遇到其他问题。有时，清晰的答案需要一些良好的灵感。

如果你对深入研究这种类型级编程感兴趣，我强烈推荐[Execute Program 的高级 TypeScript 课程](https://www.executeprogram.com/courses/advanced-typescript)。它提供了非常详细的探索，展示了如何实现非常强大的功能，比你仅仅查看 TypeScript 手册要详细得多。
