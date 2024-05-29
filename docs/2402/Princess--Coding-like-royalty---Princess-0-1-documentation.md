<!--yml
category: 未分类
date: 2024-05-27 14:42:07
-->

# Princess: Coding like royalty — Princess 0.1 documentation

> 来源：[https://princess.sh/](https://princess.sh/)

# Princess: Coding like royalty

Warning

Do note that this language is in the early alpha stages. Not everything mentioned in this documentation is already implemented. The syntax might change at any point, proceed with caution!

Do note that this documentation will not explain what an if statement or a for loop is, some basic knowledge on programming is expected.

The standard library in Princess is essentially implemented in such a way that it was possible to write the compiler in it. If you want to contribute by adding more useful functions please go ahead and do so!

## Features

> *   Familar and logical syntax
>     
>     
> *   Statically typed with structural typing
>     
>     
> *   Run any code at compile time
>     
>     
> *   Support for automatic memory management with references
>     
>     
> *   Copy constructors and destructors
>     
>     
> *   Operator overloading
>     
>     
> *   Polymorphic functions and types
>     
>     
> *   Uniform function call syntax
>     
>     
> *   Cleanup using defer
>     
>     
> *   Implicit converter functions
>     
>     
> *   Full runtime and compile time reflection
>     
>     
> *   FFI and interop with C, Access to the C standard library
>     
>     
> *   Debug symbols with GDB
>     
>     
> *   LLVM as Backend
>     
>     
> *   Incremental compilation (WIP)
>     
>     
> *   VSCode extension with autocompletion (WIP)

```
let  i  =  7

def  mod(a:  int,  base:  int)  ->  int  {
  return  ((a  %  base)  +  base)  %  base
}

// Uniform call syntax
assert  i.mod(3)  ==  1

import  strings
type  Person  =  struct  {
  name:  Str
  age:  uint
}

// Define to_string to be able to print Person
export  def  to_string(person:  &Person)  ->  String  {
  return  person.name  +  ": "  +  person.age
}

let  person  =  [  name  =  "Bob",  age  =  42  ]  !Person
print(person,  "\n")

// Compile time parameter type T
def  my_size_of(type  T)  ->  size_t  {
  // Full reflection support
  return  T.size
}

assert  my_size_of(Person)  ==  32 
```