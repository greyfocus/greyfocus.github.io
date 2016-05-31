---
layout: post
title: VisualVM CPU Sampling
date: 2016-05-31
keywords: java, performance, sampling, profiling
---

If you need to investigate CPU related issues, sampling provides an easy
mechanism for identifying bottlenecks, with minimal effects on the performance.

<!-- more -->

### Sampling vs Profiling

First of all, let's understand the difference between sampling and profiling,
which is a key prerequisite. 

#### Profiling
Profiling involves instrumenting the entire application code or only some
classes in order to provide runtime performance metrics to the profiler
application. Since this involves changes to the application code, which are
applied automatically by the profiler, it also means that there is a certain
performance impact and risk of affecting the existing functionality.

The actual degree of the performance impact is hard to determine, but it can
become significant if CPU intensive sections are instrumented.

Profiling is usually recommended for optimizing specific algorithms or when
you're interested in measuring the invocation counts.

#### Sampling
Sampling on the other side works by periodically retrieving thread dumps from
the JVM. In this case, the performance impact is minor (and constant since the
thread dumps are retrieved using a fixed frequency) and there's no risk of
introducing side effects. This process is a lot less intrusive and can also be
performed quite reliably on remote applications (i.e. it could even be applied
to production instances).

The main downside of CPU sampling is the accuracy - since the thread dump is
retrieved at fixed intervals, there is a high risk of missing certain method
invocations (especially the very fast ones). This means that the invocation
count of methods is very innacurrate, but the total spent time (and CPU time)
should still provide some relavant metrics.

### When to Use CPU Sampling
Unless you are interested in very precise performance metrics (albeit affected
by the added cost of instrumentation), you should use sampling most of the time.
The main advantage of profiling is its accuracy, but since there's the
performance impact added by instrumentation, most of the performance metrics
will be off by an unknown factor.

### How to Run CPU Sampling

### Local applications

For local applications, launch VisualVM from the JDK binary directory,

1. Select (double click) on the process that you would like to monitor on the
   left hand screen.
2. Click on the "Sampler" tab
3. When you are ready to perform your test, select the button "CPU" next to the
   "Sample" tab.
4. Once the test has finished, press "Stop" and press the "Snapshot" button.

Please keep in mind that the data displayed before taking a snapshot may or may
not be very accurate. You should do your analysis only on snapshots. 

A common error that people do when following this steps is to take an actual
screenshot of the sampling screen. While it's nice they were so thoughtful, the
data is mostly pointless - as most of the time the performance bottlenecks will
be somewhere deeper in the call hierarchy and will not be seen from the
overview.

### Remote applications

For remote applications, the process is very similar but it requires setting up
a JMX connection to the Java process to be monitored.

1. Enable the JMX port on your application. This is outside the scope of this
   article, but you can check the official Oracle documentation for more
   details:
   [Monitoring and Management Using JMX Technology](https://docs.oracle.com/javase/8/docs/technotes/guides/management/agent.html)
2. Right click on the "Remote" tab in the left hand screen.
3. Select "Add Remote Host"
4. Fill in the host name. Most of the times, this will be sufficient but
   depending on how you've enabled JMX on the remote process, you may need to
   also check the "Advanced Settings" tab.
3. Right click on the newly added host and select "Add JMX connection".
4. Fill in the connection details (including port number) and the display name.
   If you're application is deployed using multiple processes, you should enter
   a descriptive display name.
5. Double click on the newly added JMX connection.
6. From this point onwards, the steps are identical with the ones from "Local
   applications".

### How to Interpet the Data
This depends a lot on the actual issues that you're trying to investigate and
the application architecture. For example, if you have a desktop application
with a fixed amount of threads, the call tree with the break down per thread may
be useful.

However, if you're working on a web application with a variable number of
threads, it will probably be hard to figure out what's happening. In this case,
you should probably start from thet "Hot spots" tab and dig deeper from there.

### Difference between "Self Time" and "Self Time (CPU)"
VisualVM reports two metrics related to the duration, but there is a significant
difference between them:

- **self time** - counts the total time spent in that method, including the amount
    of time spent on locks or other blocking behaviour
- **self time (cpu)** - counts the total time spent in that method, excluding the
    amount of time the thread was blocked

From here, you will need to decide on what you want to focus,

- if you want to focus on optimising the multithreaded interactions, then you
should aim for the self time values including the time the threads were blocked
- if you're interested in the overall performace and not care too much about the
multithreaded interactions, then should focus solely on the self time (cpu).

Be careful though on how you interpret your results. If you have a thread that
keeps a connection open, most likely you will see some very large numbers for
the self time. This is normal and it's not issue.

