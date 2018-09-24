---
layout: post
title: "Intersection Types in Java"
date: 2018-09-19 11:29:15 +0200
comments: true
categories: ["java", "solid", "generics"]
---

This blog post aims to explain how we can use intersection types in Java when we expect an object that implements different interfaces.

<!-- more -->

## Interface Segregation Principle

The Interface Segregation Principle (ISP) stipulates that interfaces should contain the least amount of methods as possible. In other terms, a client of an interface should use all the methods of this interface.

For instance, let's take this `File` interface:

```java
interface File {
	Collection<String> readLines();
	void write(String line);
	void deleteFile();
}

class LocalFile implements File {
	// ...
}
```

As a client of this interface, it is highly unlikely that I need all methods. I may want to just read the file and, in that case, I certainely don't want to be able to delete it. If I just want to delete the file, I probably don't want to read all its lines.

In order to avoid that, it is a good idea to split this interface in 3 seperate ones:

```java
interface FileReader {
	Collection<String> readLines();
}

interface FileWriter {
	void write(String line);
}

interface FileDestroyer {
	void deleteFile();
}

class LocalFile implements FileReader, FileWriter, FileDestroyer {
	// The concrete class can implement all 3 interfaces
	// ...
}
```

Now, a client can just require the interface it needs and ignore the rest. 

## Interface Combination

Writing tiny interfaces is good to enforce ISP and lower the coupling of the code. However, what happens when a client wants to read a file *and* write at the same time?

The first two snippets won't compile because one of the interface is not implemented:

```java
void readAndWrite(FileReader reader) {
	reader.readLines();
	reader.write("Hello"); // That won't compile since reader does not implement FileWriter
}

void readAndWrite(FileWriter writer) {
	reader.readLines(); // That won't compile since writer does not implement FileReader
	reader.write("Hello");
}
```

As an alternative, it is possible to pass an instance of `LocalFile` but it introduces a high coupling between the method and the LocalFile concrete class, defeating the whole purpose of interfaces entirely.

```java
void readAndWrite(LocalFile file) {
	file.readLines();
	file.write("Hello");
	// That will compile but it is not recommended
}
```

Since Java 1.5, and the introduction of generics, a feature, known as Intersection Types, allows to combine interfaces in this kind of situation.

## Intersection Types to the Rescue

The following code uses intersection types to solve the issue of needing an object that implements several interfaces:

```java
<T extends FileReader & FileWriter> void readAndWrite(T file) {
	file.readLines();
	file.write("Hello");	
}
```

The `&` symbol means that the method expects a type `T` that implements both the `FileReader` and `FileWriter` interfaces.

## Conclusion

The intersection types is a feature that is not widely used in Java. However, it is very powerful as it allows to write very tiny interfaces and combine them on demand. From now on, there is no excuse to write big fat interfaces that have dozens of totally unrelated methods!