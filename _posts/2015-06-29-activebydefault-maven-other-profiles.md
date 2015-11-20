---
layout: post
title: Active by default Maven profile and other profiles
date: 2015-06-29 23:29
category: java
keywords: Maven, profile, activeByDefault
---

Let's say that you're using an `activeByDefault` Maven profile in your build
process. At some point, someone decides to add another profile, that's activated
based on some conditons (or manually using the `-P` parameter). 

<!-- more -->

``` xml
<profiles>
  <profile>
    <id>default</id>
    <activation>
        <activeByDefault>true</activeByDefault>
    </activation>
    <!-- ... -->
  </profile>
</profiles>
```

When you run the build for the first time, you may notice that the profile
marked as default is not activated.

## Cause
According to the [Maven
documentation](http://maven.apache.org/guides/introduction/introduction-to-profiles.html)
this is the standard behaviour:

> This profile will automatically be active for all builds unless another
> profile in the same POM is activated using one of the previously described
> methods. All profiles that are active by default are automatically deactivated
> when a profile in the POM is activated on the command line or through its
> activation config.

## Solutions
Unfortunately there's no straight forward way of mixing profiles marked as
`activeByDefault` with other profiles. 

The simplest solution with minimal impact would be to rewrite the
`activeByDefault` attribute with something like this:

``` xml
<profiles>
  <profile>
    <id>default</id>
    <activation>
      <property>
        <name>!skipDefault</name>
      </property>
    </activation>
    <!-- ... -->
  </profile>
</profiles>
```

In this form, the `default` profile is activated only if the `skipDefault`
property is not set. Since there's no reason for us to set it, it basically acts
as if `activeByDefault` was set.

A better solution would be to avoid "default" profiles:
* activate the profile based on a condition
* activate the profile manually from the command line
* use `activeByDefault` only as a fallback
