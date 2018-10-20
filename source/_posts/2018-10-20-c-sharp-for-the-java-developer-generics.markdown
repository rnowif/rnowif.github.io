---
layout: post
title: "C# for the Java Developer: Generics"
date: 2018-10-20 05:57:36 +1300
comments: true
categories: ["C#", ".NET", "Java", "JVM"]
---

In my journey into the C# world, I wanted to talk about generics. Generics exist in both Java and C# languages but their implementation is *very* different. This blog post aims to explain the differences and the similarities between the two.  
TL;DR Java generics is a lie, C# generics is not.

<!-- more -->

## Java Generics is a Lie

Generics have been introduced in Java 5. Before that, you had to manipulate `Objects` and cast them to the desired type:

```java
List apples = new ArrayList();
apples.add(new Apple());
// This is really a list of objects, so the cast is required
Apple firstApple = (Apple) apples.get(0);
```

In the previous snippet, you should note that there is absolutely nothing that prevents you from adding a `Banana` into the list and make the program crash at runtime. Also, this code is still valid in the latest version of Java (which is Java 11 as we speak). To say the least, this approach is not very safe and do not leverage the type system as much as we could expect.

Since Java 5, it is then possible to use the generic version of the `List` class:

```java
List<Apple> apples = new ArrayList<Apple>();
apples.add(new Apple());
// The cast is not required any more thanks to the generics
Apple firstApple = apples.get(0);
```

The thing is that, at runtime, the two previous snippets are strictly equivalent. Indeed, Java generics are removed during compilation and the resulting bytecode only manipulate `Objects` and casts. This is called _type erasure_. As a consequence, it is not possible, in Java, to write code like this:

```java
/**
 * Filter objects of the given type
 */
<T> List<T> filterObjectsOfType(List<Object> objects) {
	List<T> filteredObjects = new ArrayList<>();
	for (Object o : objects) {
		if (o instanceof T) {
			filteredObjects.add((T) o);
		}
	}
	return filteredObjects;
}
```

Indeed, the type of `T` is erased at runtime and the `instanceof` operation cannot be performed. That is why a class object is often passed as an argument of the method:

```java
/**
 * Filter objects of the given type
 */
<T> List<T> filterObjectsOfType(List<Object> objects, Class<T> clazz) {
	List<T> filteredObjects = new ArrayList<>();
	for (Object o : objects) {
		if (clazz.isInstance(o)) {
			filteredObjects.add((T) o);
		}
	}
	return filteredObjects;
}
```

This method can be invoked like this:

```java
List<String> stringsOnly = filterObjectsOfType(Arrays.asList("hello", 2), String.class);
// stringsOnly contains only "hello"
```

## C# Generics is a Runtime Feature

C# generics, on the other hand, is a totally different beast. Indeed, the real type is kept at runtime and it is possible to use this type to write this kind of code:

```csharp
/**
 * Filter objects of the given type
 */
public IEnumerable<T> FilterObjectsOfType<T>(IEnumerable<object> objects)
{
    List<T> filteredObjects = new List<T>();
    foreach (var obj in objects)
    {
        if (obj is T)
        {
            filteredObjects.Add((T) obj);    
        }
    }
    return filteredObjects;
} 
```

This method can be invoked like this:

```csharp
IEnumerable<String> stringsOnly = FilterObjectsOfType<String>(new List<object>(new object[] { "hello", 2 }));
// stringsOnly contains only "hello"
```


## Conclusion

In Java, generics are used to write type-safe code but this feature is limited and the code can be awkward sometimes due to the type erasure mechanism. In C#, it is a runtime feature and its usage is much more straightforward.

_PS: I realize that the code I wrote in the snippets is not really idiomatic but I wanted to have Java and C# code as similar as possible to be able to focus only on the usage of generics._