<!--yml

类别：未分类

日期：2024-05-27 14:42:07

-->

# Princess：如皇族般编程 — Princess 0.1 文档

> 来源：[https://princess.sh/](https://princess.sh/)

# Princess：如皇族般编程

警告

请注意，这种语言处于早期 alpha 阶段。此文档中提到的并非所有功能都已实现。语法可能随时更改，请谨慎操作！

请注意，此文档不会解释什么是 if 语句或 for 循环，需要一些基本的编程知识。

Princess 中的标准库实质上是以这样一种方式实现的，以至于可以用它来编写编译器。如果你想通过添加更多有用的函数来做贡献，请继续！

## 特性

> +   熟悉且逻辑清晰的语法
> +   
> +   结构化类型的静态类型化
> +   
> +   在编译时运行任何代码
> +   
> +   支持使用引用进行自动内存管理
> +   
> +   拷贝构造函数和析构函数
> +   
> +   运算符重载
> +   
> +   多态函数和类型
> +   
> +   统一函数调用语法
> +   
> +   使用 defer 进行清理
> +   
> +   隐式转换函数
> +   
> +   完整的运行时和编译时反射
> +   
> +   FFI 和与 C 的互操作，访问 C 标准库
> +   
> +   带有 GDB 的调试符号
> +   
> +   LLVM 作为后端
> +   
> +   增量编译（WIP）
> +   
> +   VSCode 插件与自动完成（WIP）

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
