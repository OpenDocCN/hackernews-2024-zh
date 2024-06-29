<!--yml

category: 未分类

date: 2024-05-27 14:29:32

-->

# Go 枚举糟糕透了

> 来源：[https://www.zarl.dev/articles/enums](https://www.zarl.dev/articles/enums)

# Go 枚举糟糕透了

Go 在技术上没有枚举，在我看来这是一个缺失的特性，但是 Go 有一种符合习惯的方法来实现大致相同的功能。不过，Go 确实有 `iota` 关键字。这基本上是一个自增的整数 - 因此，你最终会写出像这样的枚举：

```
type Operation int

const (
	Escalated Operation = iota
	Archived
	Deleted
	Completed
) 
```

这没问题，但它在底层只是一个整数，这意味着实际上我们拥有的是：

```
const (
	Escalated Operation = 0
	Archived  Operation = 1
	Deleted   Operation = 2
	Completed Operation = 3
) 
```

这也意味着使用这些或包含这些的结构体的任何函数也可以接受一个 `int` 值。因此，最好确保我们的默认情况得到正确处理。因此，我们要么决定默认情况是什么，要么像我通常做的那样指定一个 `Unknown` 情况并将其用作验证检查。所以现在我们有：

```
type Operation int

const (
	Unknown Operation = iota
	Escalated 
	Archived
	Deleted
	Completed
)

func (o Operation) IsValid() bool {
	if o == Unknown {
		return false
	}
	return true
} 
```

但是你在这里注意到的是我们没有这些枚举的字符串表示，因此我们接下来必须构建它，为此我只使用一个 `map[Operation]string` 并用定义的字符串常量进行实例化。我们现在还需要处理一个从 `string` 到 `Operation` 的反向场景，例如，如果我们通过 JSON 读取值。添加字符串映射：

```
 const (
	unknownStr = "Unknown"
	escalatedStr = "Escalated"
	archivedStr = "Archived"
	deletedStr = "Deleted"
	completedStr = "Completed"
)

var (
    opsToStrings = map[Operation]string{
        Escalated: escalatedStr,
        Archived:  archivedStr,
        Deleted:   deletedStr,
        Completed: completedStr,
    }
    stringsToOps = map[string]Operation{
        escalatedStr: Escalated,
        archivedStr:  Archived,
        deletedStr:   Deleted,
        completedStr: Completed,
    }
) 
```

现在我们设置了这些映射，我们真的应该将它们投入使用，为此，我通常会创建一个方法和一个函数，即枚举类型上的 `String()` 方法和一个接受任意类型并返回 `Operation` 类型的包级别函数 Parse。具体如下：

```
func (o Operation) String() string {
    if s, ok := opsToStrings[o]; ok {
        return s
    }
    return unknownStr
}

func ParseOperation(o any) Operation {
    switch o := o.(type) {
    case Operation:
        return o
    case int:
        return Operation(o)
    case string:
        return stringToOp(o)
    case fmt.Stringer:
        return stringToOp(o.String())
    }
    return Unknown
}

  func (o Operation) IsValid() bool {
    _, ok := opsToStrings[o]
	return ok
}

  func stringToOp(o string) Operation {
    if op, ok := stringsToOps[o]; ok {
        return op
    }
    return Unknown
} 
```

这使我们可以在 JSON 中使用枚举，例如，它可以表示为 `int` 或 `string`，这取决于你，但我几乎总是选择在 JSON 中使用 `string` 表示。当然，这是世界上最常用的两个接口。JSON 的编组和解组接口。

```
func (o Operation) MarshalJSON() ([]byte, error) {
    return []byte(`"` + t.String() + `"`), nil
}

func (t *Operation) UnmarshalJSON(b []byte) error {
    *t = Parse(string(b))
    return nil
} 
```

OK，现在我们已经定义了所有单独的枚举类型，我喜欢将它们分组到一个结构体中，并使其在引用时更像是 Java/Kotlin 的枚举。首先创建一个带有每种情况的 `Operation` 的容器包装，然后定义一个公共变量，类型为容器类型，带有所有正确的映射。最终看起来是这样的：

```
type operationContainer struct {
	UNKNOWN:   Operation
	ESCALATED: Operation
	ARCHIVED:  Operation
	DELETED:   Operation
	COMPLETED: Operation
 }

var Operations = operationContainer {
	UNKNOWN: Unknown,
	ESCALATED: Escalated,
	ARCHIVED: Archived,
	DELETED: Deleted,
	COMPLETED: Completed,
} 
```

这使得我们可以像下面这样使用这些枚举，例如，如果我们从 `cmd` 包导入。

```
cmdOp := cmd.Operations.COMPLETED 
```

有了这个容器，我们还可以扩展功能，以便返回所有枚举作为一个切片，以便在需要时遍历所有值，不包括 `Unknown` 值。

```
func (c operationsContainer) All() []Operation {
    return []Operation{
        c.ESCALATED,
        c.ARCHIVED,
        c.DELETED,
        c.COMPLETED,
    }
} 
```

## 完成了。或者说，完成了吗……

经过所有这些步骤，我们现在有了一个更好、更全面的枚举，但最大的问题仍然存在；任何接受 `Operation` 枚举类型的地方都可以轻松地接受一个 `int`。这真是个大麻烦，因为这几乎完全抵消了我们在这里所做的工作。还有一点是，虽然我们有了我们的容器，但原始定义也是公开的，并且可以从包外访问，因此我们有两个功能上相同的东西。

```
opCompleted := cmd.Completed
enumCompleted := cmd.Operations.COMPLETED 
```

为了限制这一点，我做的第一件事是将operation Enum类型私有化，所以`Operation`变成了`operation`，并且还使所有operation类型的实例都是私有的。然后我定义了一个名为`Operation`的新公共结构体，其中包含了这个私有的`operation` Enum。

```
type Operation struct {
    operation
}

type operation int

const (
    unknown operation = iota
    escalated
    archived
    deleted
    completed
) 
```

现在这样做使得不可能只是将`int`传递给使用`Operation`类型的任何函数或结构体，因为它们无法实例化一个带有有效私有`operation`的新对象，因为它不能在包外使用。这也意味着，即使有人例如使用这个空结构体`empty := cmd.Operation{}`，它将是无效的，并报告为`Unknown`。所以现在这件事情已经完成，我们需要改变我们的公共容器来适应我们的新`Operation`结构，但是如你所见，容器不需要改变，但是赋值需要改变。

```
type operationsContainer struct {
    UNKNOWN   Operation
    ESCALATED Operation
    ARCHIVED  Operation
    DELETED   Operation
    COMPLETED Operation
}

var Operations = operationsContainer{
    UNKNOWN:   Operation{unknown},
    ESCALATED: Operation{escalated},
    ARCHIVED:  Operation{archived},
    DELETED:   Operation{deleted},
    COMPLETED: Operation{completed},
} 
```

我们现在也需要更改我们的解析函数来处理包装的`operation`。

```
func Parse(a any) Operation {
    switch v := a.(type) {
    case Operation:
        return v
    case string:
        return Operation{stringToOperation(v)}
    case fmt.Stringer:
        return Operation{stringToOperation(v.String())}
    case int:
        return Operation{operation(v)}
    case int64:
        return Operation{operation(int(v))}
    case int32:
        return Operation{operation(int(v))}
    }
    return Operation{unknown}
} 
```

使用这种新结构，我们仍然可以在包外部访问Enum时拥有相同的漂亮语法，但个别的Enums不是公开的，因此这鼓励使用单例Enum容器来获取适当的值，而且它还使IDE中的智能感知更好，因为公开的变量更少，这意味着下拉菜单更干净。

现在我知道 - 这是大量工作，只是为了获得漂亮的枚举，但老实说，我认为这确实在使用它们时使开发者体验更好，并且从类型安全性的角度来看，比`iota`方法更好。然而，我也同意这是太多的工作，而且是重复的工作，这很容易需要在添加枚举或更改枚举时进行重新工作，这意味着必须确保所有字符串映射等都已经正确完成。

So ………..

# goenums

这是我为了配合本文编写的一个小工具 - 它是一个枚举生成器，以与我上面所述的相同风格生成Enums。它是一个基于简单`text/template`的工具，可以将一些简单的JSON配置转换为格式化的go文件。

它变成了这样：

```
{
  "enums": [
     {
      "package": "cmd",
      "type": "operation",
      "values": [
        "Escalated",
        "Archived",
        "Deleted",
        "Completed"
      ]
    }
  ]
} 
```

转到这里：

```
package cmd

import "fmt"

type Operation struct {
    operation
}

type operation int

const (
    unknown operation = iota
    escalated
    archived
    deleted
    completed
)

var (
    strOperationMap = map[operation]string{
        escalated: "ESCALATED",
        archived:  "ARCHIVED",
        deleted:   "DELETED",
        completed: "COMPLETED",
    }

    typeOperationMap = map[string]operation{
        "ESCALATED": escalated,
        "ARCHIVED":  archived,
        "DELETED":   deleted,
        "COMPLETED": completed,
    }
)

func (t operation) String() string {
    return strOperationMap[t]
}

func Parse(a any) Operation {
    switch v := a.(type) {
    case Operation:
        return v
    case string:
        return Operation{stringToOperation(v)}
    case fmt.Stringer:
        return Operation{stringToOperation(v.String())}
    case int:
        return Operation{operation(v)}
    case int64:
        return Operation{operation(int(v))}
    case int32:
        return Operation{operation(int(v))}
    }
    return Operation{unknown}
}

func stringToOperation(s string) operation {
    if v, ok := typeOperationMap[s]; ok {
        return v
    }
    return unknown
}

func (t operation) IsValid() bool {
    _, ok := strOperationMap[s]
    return ok
}

type operationsContainer struct {
    UNKNOWN   Operation
    ESCALATED Operation
    ARCHIVED  Operation
    DELETED   Operation
    COMPLETED Operation
}

var Operations = operationsContainer{
    UNKNOWN:   Operation{unknown},
    ESCALATED: Operation{escalated},
    ARCHIVED:  Operation{archived},
    DELETED:   Operation{deleted},
    COMPLETED: Operation{completed},
}

func (c operationsContainer) All() []Operation {
    return []Operation{
        c.ESCALATED,
        c.ARCHIVED,
        c.DELETED,
        c.COMPLETED,
    }
}

func (t Operation) MarshalJSON() ([]byte, error) {
    return []byte(`"` + t.String() + `"`), nil
}

func (t *Operation) UnmarshalJSON(b []byte) error {
    *t = Parse(string(b))
    return nil
} 
```

注意，配置是一个JSON数组，这允许指定许多不同的Enums，并在一次生成中生成它们。有关goenums的更多信息，请访问github上的[https://www.github.com/zarldev/goenums](https://www.github.com/zarldev/goenums)。

[](https://www.zarl.dev/articles/enums-take-two)

# [我对此进行了后续，这更好地反映了当前的goenums代码。](https://www.zarl.dev/articles/enums-take-two)
