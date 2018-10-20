---
layout: post
title: "C# for the Java Developer: Enums"
date: 2018-10-21 06:54:27 +1300
comments: true
categories: ["C#", "Java"]
---

Enums is a very common concept. It exists, of course, in Java and C# as well. However, Java and C# enums do not have the same capabilities. This blog post aims to show their differences.

<!-- more -->

In Java, enums are very much like regular classes: they can implement interfaces and have methods. However, they cannot inherit other classes or be explicitly instantiated. They can be viewed as `final` classes (or `sealed` classes in C#) that already inherit the virtual "enum" class, have only private constructor(s) and a set of pre-defined instances (the values of the enum).

For instance, let's take the example of the HTTP status codes. In Java, it is possible to write this code:

```java
enum HttpStatus implements Comparable<HttpStatus> {
	OK(200), SERVER_ERROR(500), NOT_FOUND(404);

	private final int code;

	HttpStatus(int code) {
		this.code = code;
	}

	// This is a regular instance method
	int code() {
		return code;
	}

	// This is the implementation of the Comparable interface
	@Override
	int compareTo(HttpStatus o) {
		// Http statuses will be sorted by status code
		return Integer.compare(code, o.code);
	}
}
```

```java
HttpStatus status = HttpStatus.OK;

// The 'code' method can be invoked like any other method
int code = status.code();
```

In C#, enums are just integers in disguise. The previous snippet can be simulated in C# only because the `code` attribute happens to be an `int`. Otherwise, it would be very complex to have the same behaviour:

```csharp
enum HttpStatus {
	// int value of the enum can be forced to a specific value
	OK = 200,
	NOT_FOUND = 404,
	SERVER_ERROR = 500
}
```

```csharp
HttpStatus status = HttpStatus.OK;

// The enum can be casted to an int to get its value
int code = (int) status;
```

To sum up, Java enums are much more powerful than their C# counterparts. I often use these features when I write Java code and I think I would miss them if I had to write C# code on a daily basis.