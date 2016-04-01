---
layout: post
title: Mac OS Setup for Java Developers
date: 2015-05-30 22:50
category: java
keywords: git, os x, java 
---

In this post I will go through the steps for setting up a very basic Java development
environment on Mac OS, based on homebrew, cask and jenv.

<!-- more -->

# 1. Tools of trade
* [Homebrew](http://brew.sh) - Command line package manager for Mac OS X.
* [Cask](http://caskroom.io/) - Extends `brew` by adding support for multiple
  applications, including commercial ones.  
* [jEnv](http://www.jenv.be/) - Manage multiple Java versions easily.

# 2. Basics

## 2.1. A Better Terminal

The Terminal application distributed with Mac OS X is great - customizable UI,
supports multiple tabs, great OS integration (easy copy & paste), etc. 

However it does have a major downside - by default, the support for colors in
`ls` is disabled. You can enable it by adding the following lines in your
`.bash_profile`: 

``` bash
export CLICOLOR=1
export LSCOLORS=gxBxhxDxfxhxhxhxhxcxcx
```

This configuration assumes you're using a black background for your terminal. If
you prefer to use a different color scheme, feel free to use a [generator like
this one](http://geoff.greer.fm/lscolors/).

## 2.2 Install Homebrew 

``` bash 
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

## 2.3 Install Cask 
Another one liner, 

``` bash
brew install caskroom/cask/brew-cask
```

## 2.4 Install jEnv 
jEnv is a command line tool to help you forget how to set the JAVA_HOME
environment variable. It can be installed through `brew`

``` bash 
brew install jenv
```

You will also need to update your `.bash_profile` with the following lines (you
can read more details in the output of `brew install jenv`): 

``` bash 
export JENV_ROOT=/usr/local/opt/jenv
eval "$(jenv init -)"
```

## 2.5 Pick Your Java Version
You can view the available Java versions by querying Cask: 

``` bash 
brew cask search java7 
```

You can view more details about a Java version using `brew cask info`: 

``` bash
brew cask info java 
```

To install a Java version, 
``` bash 
brew cask install java7 
```

## 2.6 Use jEnv to switch between Java versions
First, you will need to tell jEnv which Java versions you would like to manage:

``` bash 
jenv add /System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home/ 
```

You can view the managed Java versions using:

``` bash 
jenv versions 
```

You can switch globally to anoter Java version using 

``` bash 
jenv global 1.6 
```

For more details, be sure to check the [Official jEnv
Documentation](http://www.jenv.be/).
