---
layout: post
title: Memory Leaks in Java - Introduction
date: 2016-04-01 23:12
category: java
keywords: java, performance
---

While memory leaks in the Java language are different than in languages with
manual memory allocation, they can still become challenges to developers -
especially when they are discovered in the late phases of the project.

<!-- more -->

# Memory Leaks in Java Applications
According to [Wikipedia](https://en.wikipedia.org/wiki/Memory_leak), a memory
leak is

> (...) a type of resource leak that occurs when a computer program
> incorrectly manages memory allocations in such a way that memory which is no
> longer needed is not released.

While we do not directly allocate memory in Java, we do it indirectly - every
time an object is created, some memory is allocated for it. In order for the
Garbage Collector to free the memory used by an object, it should not be
referenced from anywhere else (from other objects, static context, etc.).

The next sentence from the Wikipedia article, refers to OOP languages:

> In object-oriented programming, a memory leak may happen when an object is
> stored in memory but cannot be accessed by the running code

Unfortunately I cannot agree to it entirely,

- In a lot of cases, the objects that cause the memory leak are accessible -
  but they referenced may be hard to see or follow.
- A memory leak can also happen in a non OOP language, with automatic memory
  management (such as a garbage collector), if a long-lived entity keeps
  references to shorter lived ones

A somewhat better definition of a memory leak in Java, would be:

> A memory leak happens when there is a reference to a short lived object from
> a long-lived context (another object, static context, etc.), which causes
> the memory usage of an application to increase over time, until it runs out
> of memory.

# Difference Between Memory Leaks and Poor Memory Usage
Memory leaks are buzz words. They are keywords that you probably want to use
when you explain the performance problems to management, since they probably
sound familiar (at least to the more technical oriented ones) and they also
sound dangerous. When you want to blame the issue on some legacy code, just
throw it out - *"there was a memory leak there that we've found and fixed".*

The difference between a memory leak and poor memory usage is very subtle.
Just because that you've seen the memory increase, it doesn't mean that you've
found a memory leak - in fact, most of the time you've found one of the
following:

- a normal pattern in the lifecycle of the garbage collector
- poor memory usage due to how the data is modelled

In Java applications, the heap memory is deallocated during a Garbage collector
run. An object is eligible for garbage collection if,

- it is not a GC root
- it is not accessible from GC root references (direct or transitive)

> **What is a GC root?**
> GC roots are special objects that are not garbage collected and keep
> references to the objects used by the application flows.
>
> - **Class** - classes loaded by the system class loader
> - **Thread** - live thread
> - **Stack local** - local variable or parameter of a method
> - **JNI local**
> - **JNI global**
> - **Monitor used** - objects used as monitors for synchronization
> - **Held by JVM** - e.g. the system class loader

There are three types of GC runs,

- minor GC: cleaning from the Young space
- major GC: cleaning from the Tenured space
- full GC: cleaning the entire heap - both Young and Tenured spaces

For more details, see also this article [DZone - Minor GC vs Major GC vs Full
GC](https://dzone.com/articles/minor-gc-vs-major-gc-vs-full). If you proceed
in this direction, things will become quite technical - but in most cases, it
should be enough to know that if a GC run occurred, it doesn't necessary mean
that all memory was freed. Depending on its type, there could be some other
areas of memory that can be reclaimable - and they will be reclaimed at some
later point.

# Signs of a memory leak
While it's by not means exhaustive, the following list illustrates some
common signs of a memory leak:

- the used memory steadily increases over a long period of time (e.g. days,
  weeks, months); manually running the garbage collector (e.g. from external
tools) does not free much memory
- repeating certain steps / flows causes the memory to increase constantly;
  manually running the garbage collector (e.g. from external tools) does not
free much memory
- out of memory errors that happen even after increasing the amount of memory
  available
- the used memory increases suddenly after a release, but it cannot be
  explained by any recent changes; manually running the garbage collector
(e.g. from external tools) does not free much memory

In order to rule out any GC patterns, it is important to run the GC manually
before taking any measurements. This can be done from profiling tools, such as
VisualVM or even jConsole.

For now, this post got a bit longer than I anticipated initially. As time
permits, I will try to write a follow-up with some hints on how to approach
memory leaks.
