<!--yml
category: 未分类
date: 2024-05-27 14:29:32
-->

# Go Enums Suck

> 来源：[https://www.zarl.dev/articles/enums](https://www.zarl.dev/articles/enums)

# Go Enums Suck

Go doesn’t technially have Enums and it is a missing feature in my book but there is a Go idiomatic way to achieve roughly the same thing. Go does have however the `iota` keyword. This is basically a self incrementing integer - so you end up writing Enums something like this:

```
type Operation int

const (
	Escalated Operation = iota
	Archived
	Deleted
	Completed
) 
```

This is fine however it is nothing but an integer under the covers this means what we actually have is:

```
const (
	Escalated Operation = 0
	Archived  Operation = 1
	Deleted   Operation = 2
	Completed Operation = 3
) 
```

This also means any function that uses these or a struct that contains these can also just take an `int` value. For this reason its best to make sure our default case is correctly handled. So for this we either decide which will be the default case or what I usually do is specify an `Unknown` case and use this as a validation check. So now we have :

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

But what you notice here is we have no string representation of these Enums so we have to build that out next, for this I just use a `map[Operation]string` and instantiate this with the defined string constants. We also now need to handle in inverse scenario of a `string` to an `Operation` for example if we are reading the value off the wire via JSON for example. Adding the string maps:

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

Now that we have these maps set up we should really put them to use, for this I usually make 1 method and 1 function, the `String()` method on the Enum type and a package level function Parse which takes the `any` type and can return an Operation type. This looks like:

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

This lets us use the Enum in JSON for example, it can be represented as an `int` or a `string` it’s up to you, but I almost always go for the `string` representation in JSON. And of course it’s the 2 most used interfaces in the world. The JSON Marshall and Unmarshall interfaces.

```
func (o Operation) MarshalJSON() ([]byte, error) {
    return []byte(`"` + t.String() + `"`), nil
}

func (t *Operation) UnmarshalJSON(b []byte) error {
    *t = Parse(string(b))
    return nil
} 
```

OK so now that we have all the individual Enums defined I like to group them up in a struct and give it the feel of an Enum more akin to a Java/Kotlin Enum when referencing it. Start with making a container wrapper with a `Operation` for each case then define a public variable of the container type with all the correct mappings. This ends up looking like:

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

This allows us to use these Enums like below if for example we are importing from the `cmd` pkg.

```
cmdOp := cmd.Operations.COMPLETED 
```

With this container we can also extend the functionality to return all Enums as a slice for iteration over all the values if required excluding the `Unknown` value.

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

## Job Done. Or is it….

After all this we now have a much nicer more fully featured Enum but the biggest problem remains; Anywhere that accepts an `Operation` Enum type will just as happily accept an `int`. This is a real pain as it almost completely negates the work we have done here. There is also the fact that we have our container but the original definitions are also public and can be accessed from outside the package so we have two things that are functionally the same.

```
opCompleted := cmd.Completed
enumCompleted := cmd.Operations.COMPLETED 
```

To limit this the first thing I did was to make the operation Enum type private so `Operation` became `operation` and made all instances of the operation type private also. I then defined a new public struct called `Operation` which contains this private `operation` Enum.

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

This now makes it impossible to just pass an `int` to any function or struct that uses the `Operation` type as they cannot instantiate a new one with a valid private `operation` as it cannot be used outside the package. This also means even if someone for example uses this as an empty struct `empty := cmd.Operation{}` it will be Invalid and will report as `Unknown`. So now that this has been done we need to change our public container to accommodate our new `Operation` struct but as you can see the container does not need to change but the assignment does.

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

We also now need to change our Parse function to handle the wrapped `operation`

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

With this new structure we still have the same nice syntax for accessing the Enum outside the package but the individual Enums are not public so this encourages the use of the singleton Enum container for getting the appropriate value and it also makes Intellisense in the IDE better as there is less public variables exposed meaning the drop down is cleaner.

Now I know - this is a metric ton of work just to get nice Enums, but to be honest I think it really makes a nicer developer experience when using them and is better than the `iota` approach from a type safety perspective. However I too agree that this is way to much work and it’s repetitive work at that, which can easily require rework with the adding of Enums or the changing of Enums means making sure everything has been done correctly for all the string maps etc.

So ………..

# goenums

This is a a little tool I wrote to accompany this post - its an Enum generator that generates Enums in the same style as I have laid out above. It’s a simple `text/template` based tool that can turn some simple JSON configuration into a formatted go file.

It turns this:

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

Into this:

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

Note that the config is a JSON array this allows specifying many different Enums and have them all get generated in one shot. More about goenums can be found on the github at [https://www.github.com/zarldev/goenums](https://www.github.com/zarldev/goenums)

[](https://www.zarl.dev/articles/enums-take-two)

# [I have done a follow-up to this, it better reflects the current goenums code.](https://www.zarl.dev/articles/enums-take-two)