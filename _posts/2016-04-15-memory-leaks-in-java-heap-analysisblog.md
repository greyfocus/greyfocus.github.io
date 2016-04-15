---
layout: post
title: Memory Leaks in Java - Heap Analysis
date: 2016-04-15
keywords: java, performance
---

In the [previous post]({% post_url 2016-04-01-memory-leaks-in-java-applications-introduction %}) 
we've went through what are memory leaks in Java and how they can manifest
themselves. In this post, I will go through more details on how to find memory
leaks and how to investigate memory issues.

<!-- more -->

### Tools of Trade
This guide will focus on the following tools:

- [VisualVM](https://visualvm.java.net/) or Eclipse MAT to take a heap dump.
VisualVM should also be available in most JDK distributions. 
- [Eclipse MAT](https://eclipse.org/mat/) for heap analysis

These tools are seen as "traditional" ones in the Java eco-system and should be
avialable in all environments and in wide range of JDKs. There are some other
options available, some of them which may provided extended tools - but they
either require new JDK versions or are not free.

Other tools that you can try,

- [Oracle Mission Control](http://www.oracle.com/technetwork/java/javaseproducts/mission-control/java-mission-control-1998576.html)
- [Your Kit](https://www.yourkit.com/)
- [JProfiler](https://www.ej-technologies.com/products/jprofiler/overview.html)

### How to Take a Heap Dump?
The first step in analysing memory issues is to get a heap dump when the issue
occurs. Unfortunately this may not always be enough and most of the time it is
recommended to also have some steps to reproduce the issue that you're trying to
fix. This will allow you to take incremental heap dumps and see how the memory
evolves across time.

There are multiple ways to take a heap dump, but I will focus on two of them.

#### Using VisualVM
VisualVM is bundled most JDK versions and can be used for monitoring and
profiling. You can take a heap dump from a local process, or from a remote
process through JMX.

> The next instructions assume that you have JMX enabled for your JVM process.
> To learn more about JMX and how to enable it, see
> [Oracle JMX Management Guide](https://docs.oracle.com/javase/1.5.0/docs/guide/management/agent.html)

To take a heap dump from a local JVM process,

1. Under the local Applications section (left side menu), select the JVM process
   that you would like to analyse.
2. Select the "Monitor" tab.
3. Select the "Heap Dump" button.

To take a heap dump from a remote JVM process,

1. Right click on the "Remote" section.
2. Enter the host name of the machine where the JVM process is running and
   select OK. If you're encountering issues, be sure to check also the "Advanced
   Section".
3. Select the newly added host and choose "Add a new JMX connection". It is also
   possible to use jstatd, but it's beyound the scope of this guide to go into
   details.
4. Fill in the required details according to the JMX configuration that you're
   using.
5. Select OK.

From this point on, the process is the same as when you're monitoring a local
JVM process. Select the "Monitor" tab use the "Heap Dump" button.

> **Note 1:** The heap dump will be saved on the remote machine and you will be
> prompted to enter the path.

> **Note 2:** Make sure that you have enough disk space available. The disk
> space needed is slightly the same as the size of the heap that you're trying
> to dump, but in some cases it can also be higher, with around 10-20%.

#### Using Eclipse Memory Analyzer
Eclipse Memory Analyzer can also be used to take heap dumps from local JVM
process. If this is your case, it should probably be more simpler than using
VisualVM (one less tool needed).

#### Take a Heap Dump after an OutOfMemoryError
You can also configure your application to generate a heap dump after an
`OutOfMemoryError` by addding the following parameter when launching the JVM:
`-XX:+HeapDumpOnOutOfMemoryError`.

### Heap Analysis
Once you have a heap dump available, you can open it in an analysing tool. In
this guide we will focus on Eclipse Memory Analyzer, but some instructions may
also be similar for other tools.

The first step would be to open the heap dump in Eclipse MAT. Be warned though,
for larger heap dumps you will need:

- Physical RAM installed on the machine roughly equal or higher than the size of
  the heap dump.
- Increase the maximum heap size available for Eclipse MAT. This can be done by
  editing the `eclipse.ini` file and adding a line similar to `-Xmx2048m`.
- Free disk space on the machine roughly equal to 100-150% of the heap dump
  size.

The first time when you open a heap dump, Eclipse MAT will build various index
files which may take a long time, depending on the heap dump size.

#### Get Familiar with Your Data
I think this is one of the most important steps that you can take when analysing
a heap dump. Sometimes you may be able to find a memory leak easily, just by
looking at the default reports produced by Eclipse, but in the most complicated
cases this will not be possible.

The real issue may lie somewhere deeper - it may not be a single instance that
uses more than 90% of your heap, instead most of the heap usage could be spread
around millions of instances, that you cannot really keep track of.

This is where it helps a lot of you know your data model. In a lot of cases, you
should be able to determine the expected number of instances of a certain class
that should be loaded in memory.

Let's say that you're investigating a memory issue related to the login process.
If you can determine the number of logged in users at the time and also the
number of active sessions, you can map these to the actual number of user
instances from the memory. Do you have a 1 to 1 mapping? If not, maybe that's
where the problem is.

The rule of thumb is to try to understand the number of instances and check if
this is correct.

#### Use the Eclipse MAT Reports
If you don't know what you're looking for, it's usually good idea to go through
the reports suggested by Eclipse MAT, but try to take them with a grain of salt.
As mentined in the previous post, not all memory issues are real memory leaks -
while the leak report may sound dramatic, it doesn't mean that you actually have
a memory leak. You should consider the entities listed there as suspects and
evaluate them one by one.

##### Histogram
As described in the tooltip, the Histogram "lists number of instances per
class". It can be a useful indicator in some cases - for example if you notice
an unusually high number of instances of a certain class.

##### Dominator Tree
As described in the [Dominator Tree](http://help.eclipse.org/mars/index.jsp?topic=%2Forg.eclipse.mat.ui.help%2Fconcepts%2Fdominatortree.html)
section from the Eclipse MAT help,

> An object x dominates an object y if every path in the object graph from the
> start (or the root) node to y must go through x.
> 
> The immediate dominator x of some object y is the dominator closest to the
> object y.
> 
> A dominator tree is built out of the object graph. In the dominator tree each
> object is the immediate dominator of its children, so dependencies between the
> objects are easily identified.

This can be a very good starting point in understanding the memory usage.

The other reports may also provide interesting data, be sure to also check them
out.

#### Do You Know What to Look For?
If you already know what you're looking for, it is quite simple to get the data
that you need. Eclipse MAT supports a query language (OQL - Object Query
Language) that allows you to query the memory details for specific objects.

For example, to get a list of all instances of the User class, you could enter
the following query:

``` sql
select * from com.mypackage.model.User
```

and select the "!" (exclamation mark) symbol.

You will get back a list of all objects of the specific class. For each
instance, you will see the shallow and retained heap used by this instance.
According to the Oracle documentation,

> *Shallow heap* is the memory consumed by one object. An object needs 32 or 64
> bits (depending on the OS architecture) per reference, 4 bytes per Integer, 8
> bytes per Long, etc. Depending on the heap dump format the size may be
> adjusted (e.g.  aligned to 8, etc...) to model better the real consumption of
> the VM.
> 
> *Retained set* of X is the set of objects which would be removed by GC when X
> is garbage collected.
> 
> *Retained heap* of X is the sum of shallow sizes of all objects in the
> retained set of X, i.e. memory kept alive by X.

If you go with this approach, you should be careful not to count the same memory
twice. If object A holds a reference to object B, the retained size of A *may*
also contain the retain size of B. It will not contain the retain size
of B if there are other references to B, except the ones from A.

##### Other options
If you right click on one of the results form the OQL query, you get access to a
context menu with several options. I will try to go through the ones that I find
the most useful:

*List Objects - with their incoming references*
It will show the list of objects that point to the currently selected one.

*List Objects - with their outgoing references*
This one is quite straightforward - it will show you the references that point
outside of the currently selected object (i.e. it's instance fields).

*Path to GC roots*
As the name says, it will show you the path to the GC roots. Most of the time,
you will probably want to also ignore all phantom/soft/weak references.

*Immediate dominators*
It will show the immediate parent object that prevents this object from being
garbage collected.

*Copy - OQL Query*
It will generate an OQL that returns this single object. This is useful if you
dig deeper through the hiearchy.


## Other Resources
* [10 Tips for Using the Eclipse Memory Analyzer](http://eclipsesource.com/blogs/2013/01/21/10-tips-for-using-the-eclipse-memory-analyzer/)
* [Eclipse MAT Help](http://help.eclipse.org/mars/topic/org.eclipse.mat.ui.help/welcome.html)

