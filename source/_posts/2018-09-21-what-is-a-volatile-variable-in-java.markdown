---
layout: post
title: "What is a Volatile Variable in Java?"
date: 2018-09-21 12:13:33 +0200
comments: true
categories: ["java", "concurrency", "jvm"]
---

The `volatile` keyword is one of the less known and less understood keyword of the Java language. The goal of this article is to explain what it is and when to use it.

<!-- more -->

## Memory Architecture

In order to understand the value of `volatile`, one must first understand the memory architecture of a computer:

{% img /images/memory_architecture.png %}

Each CPU contains its own registers, which are basically in-CPU memory. Accessing these registers and performing operations on variables here is very fast.
Each CPU also has cache memory layers. It can access this cache memory fast, but not as fast as the registers.
Finally, there is the main memory (also called RAM). All CPUs can access this memory. The main memory is much bigger than the cache memories of the CPU.

Typically, when a CPU needs to read something from the memory, it will read it into its CPU cache memory and perform operations on it. It can even read it into its registers. When the CPU needs to write it back to the memory, it will flush its registers to the cache memory and eventually this cache memory will be flushed back to the main memory.

In general, the flush is performed when the CPU needs to make room for other information. Thus, it seems clear that one cannot make any assumption about _when_ this flush will occur.

## Visibility of Shared Variables

If a computer has 2 CPUs or more, it will be able to run several threads at the same time. It means that, if each thread want to access the same variable, each CPU will have a "copy" of this variable into its cache memory and this could lead to a major synchronization issue.

Take this code for instance:

```java
public class MyServer implements Runnable {
    private boolean running = true;

    @Override
    public void run() {
        while (running) {
            // Do some very interesting stuff...
        }
    }

    public void stop() {
        running = false;
    }
}
```

It is very likely that the `stop` method will be called from a different thread than the one executing the `run` method. If these threads are executed on 2 different CPUs, the `run` method will not stop until the CPU has flushed its cache memory to the main memory. And we saw that there is no way to predict when this will happen.

In Java, the `volatile` keyword explicitely ask for a variable to be directly wrote to the main memory, and thus becoming instantely available for all CPUs.

The code then become:

```java
public class MyServer implements Runnable {
    private volatile boolean running = true;

    @Override
    public void run() {
        while (running) {
            // Do some very interesting stuff...
        }
    }

    public void stop() {
        running = false;
    }
}
```

Note that a volatile variable can be a primitive or an object and can be `null`.

## When to Use `volatile`

 A volatile variable is guaranteed to have its last version always available from every CPUs. It is typically used for flags that are modified by a thread and read by another one (like the `running` boolean in the previous snippet). Globally, you may use a `volatile` variable if the variable may be accessed by several threads and you don't need to perform a sequence of operations in an atomic manner (you should not increment a `volatile` variable for instance). If you need to do so, you should consider using a synchronization mechanism.

## Conclusion

The `volatile` keyword is seldomly used, maybe because messing with threads is frowned upon in a JEE (or Spring) application, but it can be very useful to know it and understand its usage. Indeed, it could make your code safer and prevent you from writing algorithms that rely on locks and synchronization where it's not really needed.