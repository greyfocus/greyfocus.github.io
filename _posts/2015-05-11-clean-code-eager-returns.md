---
layout: post
title: Clean Code - Pitfalls of Eager Return
date: 2015-05-11 21:28
categories: java, coding
---

When working in a team, it's very important to establish a coding style and some
general principles which may vary depending on the programming language. This
will ensure that the code will be uniform, easier to understand and follow a
standard. At least this is the ideal.

<!-- more -->

But all things in life should be in moderation - if you exaggerate something,
there will be something else, lurking behind the corner, that will exaggerate at
you later. 

Today we will talk about eager returns and what it can cause when it's enforced
blindly.

### Background: What does "eager return" mean?

A very simple way of figuring out the complexity of a method is to count the
number of indentation levels inside it. Assuming that the code is properly
formatted (it is, right?), this will give you a rough idea of how much a method
does and what it's complexity level is. 

> (Note: For the conscious reader, there's a better metric for this - [Cyclomatic complexity](http://en.wikipedia.org/wiki/Cyclomatic_complexity).)

The complexity level can be reduced very often by inverting conditions and
exiting early from the method or code block.

For example, let's take the following Java code:

``` java
  public List<String> retrieveAuthorNames(Book book) {
    List<String> authorNames = null;

    if (book != null) {
      authorNames = new LinkedList<String>();

      List<Author> authors = book.getAuthors();
      for (Author author : authors) {
        authorNames.add(author.getName());
      }
    }

    return authorNames;
  }
```

This can be simplified by inverting the `if` condition:

``` java
  public List<String> retrieveAuthorNames(Book book) {
    if (book == null) {
      return null;
    }

    List<String> authorNames = new LinkedList<String>();

    List<Author> authors = book.getAuthors();
    for (Author author : authors) {
      authorNames.add(author.getName());
    }

    return authorNames;
  }

```

At this point, some of you may start to disagree. That's alright, I understand
your point - while multiple exit points are not a problem for the compiler (as
some myths may suggest), they can become a problem for the reader. 

But for now, let's look at the advantages:

 * by inverting the if condition we're basically writing validation code; you
   can see it as a precondition, before entering the next block, make sure that
   we actually have a book
 * the maximum indentation level has decreased by one - the remaining method is
   easier to understand and follow, since we don't need to keep asking ourselves,
   can `book` be `null``?


### The problem

While all principles are meant to improve the quality, they usually fail 
miserably when they're abused.

Let's take the following example:

``` java
  public long countPublishedBooks(List<Book> books) {
    long count = 0;

    for (Book book : books) {
      if (!book.isPublished()) {
        continue;
      }

      count += 1;
    }

    return count;
  }
```

Does it make sense? What does it do?

It's alright, there's no reason for headscratching as it was meant to make
things more complicated than they should be. We've taken the previous principle
and applied it consciously to the `for` loop. 

The principle said that we should exit the block early, that's why we've used
`continue`. But there's more - we've also inverted the `if` condition, to count,
err... sorry, to discard unpublished books, just for the sake of "reducing"
complexity. Wait... weren't we supposed to write a method that counts
"published" books?

### Conclusion
Principles are good, since they are based on the hard earned experience from the 
past, but they should never be applied blindly.

### Notes
The last method can be written in just a line by taking advantage of the new
features from Java 8:

``` java
   public long countPublishedBooks(List<Book> books) {
     return books.stream().filter(book -> book.isPublished()).count();
   }
``` 
