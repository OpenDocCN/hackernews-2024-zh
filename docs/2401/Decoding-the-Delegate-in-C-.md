<!--yml
category: 未分类
date: 2024-05-27 14:44:59
-->

# Decoding the Delegate in C#

> 来源：[https://blog.upperdine.dev/decoding-the-delegate-in-csharp](https://blog.upperdine.dev/decoding-the-delegate-in-csharp)

Delegates are a language feature of C# that have been around for a few years now, and by utilising them we can make our codebases more composable and adaptable. You may or may not have encountered these in the codebases you work in, but they are more common in general-purpose code such as library or framework code.

So, what exactly do I mean when I talk about delegates? Let me start to explain by talking about Functional Programming.

## Functional Programming

Functional programming is a **programming paradigm**, or programming philosophy that emphasises the use of functions as the primary building blocks of your application. Another paradigm you are probably familiar with is **Object-oriented programming**, which alternatively emphasises the use of objects as the primary building blocks of an application.

For example, when designing a system using OOP we generally think in terms of various objects interacting with one another. Using a Functional approach, we think in terms of functions.

In functional programming, there is a concept known as **First Class Functions**. This means that functions are a type just like any other and as such can be passed around as values.

Functions can be passed in as arguments to other functions, or even returned from other functions. Functions that either take functions as a parameter, or return functions are called **Higher Order Functions**.

Because C# is a strongly-typed language, we need a name for the type representing a first class function- and in this case that name is a **Delegate.**

This may sound strange if the concept is new to you, but if you've been using C# professionally you have more than likely used these before without even knowing. For example, when you use LINQ to query collections, you are usually providing a lambda expression for filtering or transformation:

```
IEnumerable<int> FindEvenNumbers(IEnumerable<int> numbers)
{
    return numbers.Where(n => n % 2 == 0);
} 
```

Remember that lambda expressions are just functions without a name, so the `Where` method in LINQ is an example of a higher order function.

So what if we wanted to write our own Higher Order Functions?

## Func

The **Func** type is the most common delegate type that you will encounter and is for functions that return a value. For example, if we wanted to take a function as an argument that took an argument of type *string* and return an *integer* (like **int.Parse**), we would declare it as such:

```
int StringToInt (string toParse, Func<string, int> parser)
{
    return parser(toParse);
}

StringToInt("42", int.Parse); 
```

A **Func** type doesn't require any input parameters, but it *does* require a return type, so as a minimum you'll need to provide a single type parameter.

## Action

In the cases where you require a function that doesn't return a value, you can use the **Action** type.

As an example, imagine that we are writing a method and we want to be able to pass in **Console.WriteLine** as an argument. We can use the Action type as such:

```
void PrintMyMessage(string message, Action<string> printer)
{
    printer(message);
}

PrintMyMessage("Hello, World!", Console.WriteLine); 
```

The beauty of this implementation is that we can swap out **Console.WriteLine** for any void method that takes a string as an argument. In production, we may want to replace this with a logger call, and because we **programmed to an interface instead of a concrete implementation** - we won't need to modify the method signature.

What about functions that neither return a value or take any arguments? These are commonly referred to as **Thunks**, and can be modelled simply by omitting the type parameters from **Action** :

```
Action printHelloWorld = () => Console.WriteLine("Hello, World!"); 
```

There are a few other types of Delegate provided in .NET, such as **EventHandler**, **Predicate, MethodInvoker** and **ThreadStart** .These are either for niche use cases or mostly internal use, so I won't spend time going into them in this blog post.

## Returning Functions

As I touched upon earlier, a higher order function not only can take a function as an argument, but also return a function.

Now that we know the main types of Delegates, returning a function is as simple as changing the return type of a function to be a delegate type:

```
Func<int, int> MultiplyBy(int multiplier)
{
    return number => number * multiplier;
}

Func<int, int> MultiplyByThree = MultiplyBy(3);

var result = MultiplyByThree(10); 
```

It's a fairly simple example, but our **MultiplyBy** method allows us to create a new function that takes an integer and multiplies it by the specified value.

## Creating your own Delegate types

If you would like to create common classes of Delegate to use throughout a codebase without having to repeat type signatures over and over again, there is a built in **delegate** keyword in C#.

For an example, lets imagine that we want to add some uniformity to the object mapping code in our codebase. We need our Product type to be mappable to any type provided using a supplied function, but we want a centralised definition of what a mapping function should look like:

```
public record Product(Guid SKU, decimal MSRP)
{
    public delegate T ProductMapper<out T>(Product product);

    public T MapTo<T>(ProductMapper<T> mapper)
    {
        return mapper(this);
    }
} 
```

You *could* use a **Func** here instead- which actually uses the delegate keyword itself under the hood, but I find it nicer to call our delegate class something related to the domain.

## Conclusion

By utilising delegates, we are able to treat functions just like any other type and write code that is adaptable.

Object-oriented programming utilises overriding and overloading to accomplish polymorphism, but a similar effect can be achieved by composing an object or function to have the behaviour we want via injecting a function in.

An important thing to remember with delegates is that while they are very powerful, overusing them will add unnecessary complexity to your codebase so use them sparingly.