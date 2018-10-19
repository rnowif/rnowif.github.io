---
layout: post
title: "C# for the Java Developer: Extension Methods"
date: 2018-10-20 01:20:24 +1300
comments: true
categories: ["C#", "Java"]
---

After spending several years crafting Java code, I recently decided to dive back into C# and share what I learn in the process. In this blog post, I will talk about extensions methods. This concept, which exists in some JVM languages (like [Kotlin](https://kotlinlang.org/docs/reference/extensions.html)) but not in Java, let the developer add methods to a class without touching its code (hence the name _extension_ methods).

<!-- more -->

Let's say you want to create a method that puts the first letter of a string in upper case and returns the new string. You cannot modify (or even inherit) the `String` class in Java to add this method. Since you don't necessarily want to create your own custom flavour of the `String` class for your code base, you would probably create a static function that takes a `String` as an argument and return another `String`:

```java
class StringExtensions {
	static String capitalize(String word) {
		return Character.toUpperCase(word.charAt(0)) + word.substring(1);
	}
}
```

Now, you can simply invoke this function like this:

```java
String word = "hello";
String capitalizedWord = StringExtensions.capitalize(word);
```

Of course, creating a static method is also possible in C#. However, C# has a feature that allows to add new methods to a class without having to modify its code. This feature is called _extension methods_. In order to create an extension method, you have to create a top-level static class and implement a static method. The first argument of this method specifies the type that the method operates on and should have the `this` modifier:

```csharp
static class StringExtensions 
{
  static string Capitalize(this String word)
  {
      // Note that this cannot access any private data in the String class. 
      return char.ToUpper(word[0]) + word.Substring(1);
  }
}
```

In order to use it, the namespace that contains the class must be specified with a `using` directive. Afterwards, the method can be invoked as if it was an instance method of the type:

```csharp
string word = "hello";
string capitalizedWord = word.Capitalize();
```

Extension methods are very powerful. Beside the fact that the code looks "cleaner", it is also possible to seamlessly plug new behaviours in existing types by simply importing a namespace. The extension code can be in separate specific modules that can be imported only when needed. 