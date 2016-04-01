---
layout: post
title: Line breaks with Git
date: 2015-05-20 19:16
category: git
keywords: git, line breaks, line endings, white spaces
---

Git is a wonderful version control system, but some of its concepts seem
daunting at first. One of the issues that may seem confusing is how it deals
with line breaks (also called line endings).

<!-- more -->

> Note: If you're having problems that seem to be related to line endings, feel
free to jump to the [Troubleshooting](#troubleshooting) section.

## Background

The story of line breaks starts during the '60s, when printers where used as 
consoles for computer systems. The output streams of computer programs back 
then had to match the capacities of the printer; that is, if the
output was too fast, the printed data would have been corrupted.

The character LF (line feed) was used to separate lines, but this caused the
printer head to move from the end of the line to the beginning of the next
line. If another character was sent to the printer during this time, it would
have been printed somewhere in the middle off the next line, depending on the
exact position of the printer head.

To fix this issue, the CR (carriage return) was used, which in fact was more of
a delay, to let the printer head align correctly at the beginning of a new line.
In fact, some of these printers needed extra characters as well, most likely
extra CRs or NULs.

Operating systems use different ways to signal line endings. Some of the most
popular options are:

 * CR+LF - Microsoft DOS and the Windows family
 * CR - Mac OS up to version 9
 * LF - Unix-type operating systems and newer versions of Mac OS

Unfortunately this is where our story begins. It's sad to see that a 50 year old
hardware limitation is this applicable in our days, albeit in a slightly
different form.

## Why are line breaks important for Git?

Version control systems are focused on text files; they can track the changes,
merge files automatically, or facilitate the process of resolving conflicts.
Because of this, line endings are crucial in understanding the content of the
file and how to work with the changes (most of them do merges on a line by line
basis).

Git supports both CR+LF and LF line endings using several configuration options.

## Configuring line breaks for Git

There are several configuration options in Git that affect line breaks. 

### core.eol

Sets the line ending type to use in the working directory for files that have
the text property set. Alternatives are `lf`, `crlf` and `native`, which uses
the platform's native line ending. The default value is native. 

**Example:**

``` bash
# Get the value of the eol configuration property
git config core.eol

# If the output is empty, it means the default value of "native" is used.

# Sets the value of the eol configuration property to CRLF. During the
# normalization process, all line breaks will be converted to CRLF
git config core.eol CRLF
```

> Note: Git configuration properties can be set per repository (the default), or
globally by passing the `-g` paramter to `git config`.

### core.autocrlf

This setting configures the normalization process related to line breaks. The
normalization proces is applied whenever files are added to the index, but to
keep things simple let's assume that it happens when files are commited to the
repository (local or remote).

Possible values:

 * `true` - transforms the local line break character (configured in `core.eol`)
to LF
 * `false` - disable the normalization process and leave the files as they are
 * `input` - apply the normalization process to newly added files, but never to
files which are already commited

Please note that the core.autocrlf setting is machine specific. If there are
multiple developers working on the same repository, they could have different
settings, which may cause a lot of issues.

### .gitattributes

This file is used to tell newer versions of Git how to deal with line breaks,
depending on the file type. The file format is slightly generic, which means
that it may be used in the future for other options as well.

***Example:***

``` bash
# Handle line endings automatically for files detected as text using Git
# heuristics
* text=auto

# Configure line endings by file type
*.css text
```

The syntax of the file is quite simple. The first column (separated by space)
selects the file type using wildcards, while the second column contains the
attributes for this file type.

For more details, see the documentation for
[.gitattributes](ftp://www.kernel.org/pub/software/scm/git/docs/gitattributes.html)


## <a name="troubleshooting"></a>Troubleshooting

If you've made it this far, most likely you've already ran into issues with line
breaks. These kind of issues usually cause Git to mark files as modified, even
though they do not appear to have any changes; it's also not posible to revert
the changes.

### Check if you really have a line break issue

The simplest way to tell if you're having a line break issue is to run a diff:

``` bash
git diff -R
```

If you see any `^M` characters in the output or anything similar, then the issue
is caused by line breaks.

In most cases, this happens because there are some files already commited in the
repository using the wrong line breaks. For example, it could happen if you have
line break normalization enabled, but there is at least one file not normalized
in the repository (perhaps it was added by someone who had normalization
disabled in Git).

### Fix

There are two ways of fixing line break issues related to Git. 


#### Disable the line break normalization completely

This is perhaps the most non intrusive way to fix the issue, as it doesn't
require updates to all the files in the repository. Unfortunately it also has a
major side effect: if you have developers working from different operating
systems, you may end up with mixed line breaks in the repository, which could
cause various issues with other tools (and also Git merging).

Steps:

1. Add a .gitattributes file at the root of your repository with the following
content:

    ``` bash
    # Disable line break normalization for all files

    * -text

    # You can also customize this based on the file type, by using something like
    # this:
    # *.css text
    # *.js  text
    # *.csv -text
    ```

2. Commit the .gitattributes file and push it to any remotes that you may have.

#### Fix the line brakes by enforcing the normalization process

With this approach, we need to update all files from the repository in order to
use the correct line breaks. 

``` bash
# From the root of your repository remove everything from the index
git rm --cached -r .

# Change the autocrlf setting of the repository (you may want 
#  to use true on windows):
git config core.autocrlf input

# Re-add all the deleted files to the index
# (You should get lots of messages like:
#   warning: CRLF will be replaced by LF in <file>.)
git diff --cached --name-only -z | xargs -0 git add

# Commit
git commit -m "Fixed crlf issue"

# If you're doing this on a Unix/Mac OSX clone then optionally remove
# the working tree and re-check everything out with the correct line endings.
git ls-files -z | xargs -0 rm
git checkout .
```

(Thanks to [Charles Bailey](
https://stackoverflow.com/questions/1510798/trying-to-fix-line-endings-with-git-filter-branch-but-having-no-luck/1511273#1511273) for this solution.)

