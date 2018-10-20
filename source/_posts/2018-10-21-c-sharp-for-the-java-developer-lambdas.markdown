---
layout: post
title: "C# for the Java Developer: Lambdas"
date: 2018-10-21 00:37:12 +1300
comments: true
categories: ["C#", "Java"]
---

A lambda is an anonymous function that can be assigned to a variable, passed as an argument of a method and invoked at any time. We can find lambdas in Java and C# and the resulting code is very similar. A Java lambda can be viewed as the implementation of an interface with only one method (called a _functional interface_) whereas a C# lambda can be assigned to a `delegate`, which is a concept that does not exist in Java. This article aims to explain how lambdas work in Java and C# and highlight their differences and similarities.

<!-- more -->

## Java Functional Interfaces

In Java, a lambda can simply be viewed as an anonymous function that implements an interface with only one method. This kind of interface is called a _functional interface_ and can be annotated with the `@FunctionalInterface` annotation that tells the compiler to enforce the only-one-method rule:

```java
@FunctionalInterface
interface Printer {
	void print(String message);
}
```

```java
// The function interface is implemented with a lambda
Printer standardPrinter = message -> System.out.println(message);

// The print method of standardPrinter can be invoked
standardPrinter.print("Hello World!");
```

From the previous snippet, you should note that the way the interface is implemented makes absolutely no difference. It can be a lambda, a concrete or even an anonymous class. In any ways, you can simply invoke the `print` method of the interface.

There are some pre-defined [generic functional interfaces](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html) in the JDK that can be used directly off the shelf. The main ones are described below:

| Name | Method | Role |
| ---  | ---  | --- |
| [`Function<T,R>`](https://docs.oracle.com/javase/8/docs/api/java/util/function/Function.html) | `R apply(T t)` | Takes an argument of a given type T and returns an object of type R |
| [`Consumer<T>`](https://docs.oracle.com/javase/8/docs/api/java/util/function/Consumer.html) | `void accept(T)` | Takes an argument of a given type T and does something useful (typically with side effects) |
| [`Predicate<T>`](https://docs.oracle.com/javase/8/docs/api/java/util/function/Predicate.html) | `boolean test(T)` | Takes an argument of a given type T and returns a boolean (similar to `Function<T, Boolean>`) |
| [`Supplier<T>`](https://docs.oracle.com/javase/8/docs/api/java/util/function/Supplier.html) | `T get()` | Returns an object of type T |

## C# Delegates

In C#, a lambda can be assigned to a delegate which is a type that encapsulates a method:

```csharp
delegate void Print(string message);
```

```
// A lambda is assigned to the delegate
Print print = message => System.Console.WriteLine(message);

// print can be directly invoked as a method
print("Hello World!");
```

Like in Java, some delegates are already defined in the .NET framework, the main ones are described below:

| Method | Role |
| ---  | --- |
| [`TResult Func<in T,out TResult>(T arg)`](https://docs.microsoft.com/en-us/dotnet/api/system.func-2) | Takes an argument of a given type T and returns an object of type TResult |
| [`void Action<in T>(T obj)`](https://docs.microsoft.com/en-us/dotnet/api/system.action-1) | Takes an argument of a given type T and does something useful (typically with side effects) |
| [`bool Predicate<in T>(T obj)`](https://docs.microsoft.com/en-us/dotnet/api/system.predicate-1) | Takes an argument of a given type T and returns a boolean (similar to `Func<T, bool>`) |

## Conclusion

From a Java perspective, using lambdas in APIs (like Linq for instance) is pretty straightforward. However, when digging a little bit deeper, there are some subtle differences to understand. I find the Java approach simpler as it does not introduce another concept but the C# approach is cleaner because the delegate can be invoked directly like a method.